---
layout: post
title: "Gulp and Browserify with ASP.NET"
author: "Robin Ridderholt"
categories: posts
tags: [Browserify,JavaScript,Gulp.js,ASP.NET MVC]
---

If you ever worked with [node.js](http://www.nodejs.org) you are probably familiar with the “require” concept of node. But if you haven’t then “require” is an easy way to declare and load dependencies for your script or module.

I was recently introduced to a node module named [Browserify](http://browserify.org/) which lets you use the same notation with “require” when developing modules for the web. This means that you can easily write small JavaScript modules which are dependent on other modules you have written. Then, when running Browserify it will build a new set of JavaScript files for you and make sure that each module have its dependencies loaded in the file with a closure surrounding them.

[Gulp.js](http://gulpjs.com/) is a new way of building your JavaScript it’s pretty similar to [Grunt](http://gruntjs.com/) but it’s quite faster because it works with streams which you can pipe to different tasks and you don’t have to write a large configuration file in JSON you just write code.

Now I’m a .NET developer and I would like to take advantage of things like Gulp.js and Browserify when developing Web apps with ASP.NET MVC. So the question is how and if will these technologies will work together, the short answer is yes.

## Getting started
First of all you need to have node and node package manager installed which you can find on the [nodejs.org webpage](http://www.nodejs.org).

When you have node up and running you first need to install Gulp.js by running <code>npm install –g gulp</code> from the command line. This will install gulp globally and add it to your PATH. Now navigate to the Scripts folder in your ASP.NET MVC solution, here you will need to install a developer version of gulp and [Browserify](https://www.npmjs.org/package/gulp-browserify). You can do this by running the commands: <code>npm install -–save-dev gulp</code> and <code>npm install -–save-dev gulp-browserify</code>.

### Writing code
As an example I wrote three very simple modules in JavaScript to demonstrate how Browserify works.
```javascript
'use strict';
function Common(){
    var self = this;
    self.doStuff = function(){
        console.log('Common doing stuff');
    };
    return self;
}
module.exports = new Common();
```
In this small module I have created a JavaScript object which just logs out “Common doing stuff” when the doStuff method is called. The very last line is the node way to make your module publicly available for anyone to import (using the require function).
```javascript
'use strict';
var common = require('./common.js');
function Start(common){
    var self = this;
    self.doStuff = function(){
        common.doStuff();
        console.log('Start doing stuff');
    };
    return self;
}
var start = new Start(common);
start.doStuff();
```

In this module I’m using the require function to define a dependency of the common module in the previous example. 

## Setting up Gulp.js
As I mention earlier in this post I like Gulp.js mostly because instead of JSON configuration you get to write code as configuration.

This is how a basic Gulp script would look like for building your scripts with Browserify.
```javascript
var gulp = require('gulp'),
    browserify = require('gulp-browserify');

gulp.task('scripts', function(){
    gulp.src('start.js')
        .pipe(browserify({
            insertGlobals: false,
            debug: false
        })).pipe(gulp.dest('./build'));
});

gulp.task('default', ['scripts']);
```
When using Gulp.js you define a collection of tasks and at the end you define your default task with an array of task to be run. The build script file should also be named gulpfile.js.

### Building your scripts
To run your build script just (with the command line) navigate to the folder where your gulpfile.js is stored and type “gulp”. After running Gulp you end up with a new script file called the same as the original file but with all the dependencies loaded in the file, in our example this is how the resulting scrip file should look like
```javascript
  (function e(t,n,r){function s(o,u){if(!n[o]){if(!t[o]){var a=typeof require=="function"&&require;if(!u&&a)return a(o,!0);if(i)return i(o,!0);throw new Error("Cannot find module '"+o+"'")}var f=n[o]={exports:{}};t[o][0].call(f.exports,function(e){var n=t[o][1][e];return s(n?n:e)},f,f.exports,e,t,n,r)}return n[o].exports}var i=typeof require=="function"&&require;for(var o=0;o<r.length;o++)s(r[o]);return s})({1:[function(require,module,exports){
  'use strict';
  function Common(){
      var self = this;
      self.doStuff = function(){
          console.log('Common doing stuff');
      };
      return self;
  }

  module.exports = new Common();

  },{}],2:[function(require,module,exports){
  'use strict';

  var common = require('./common.js');

  function Start(common){
      var self = this;
      self.doStuff = function(){
          common.doStuff();
          console.log('Start doing stuff');
      };
      return self;
  }
  var start = new Start(common);
  start.doStuff();
  },{"./common.js":1}]},{},[2])
```
As you can see some code has been added to the file, this code just makes it possible for us to use the require function to inject dependencies in our modules. This code also provides a closure for our code so there is no naming collisions.

So far we have learned how to build our scripts using the command line and for every change we make to our modules we have to run Gulp again to generate our bundles. That’s perhaps not very productive to have to rebuild every time you add or change some functionality, there is a way to let gulp watch for changes in our modules and automatically build the bundles but we will get to that later in this post.

## Integrating Gulp with Visual Studio
Wouldn’t it be very convenient if you could automatically run the gulpfile when you compiled you ASP.NET site? Luckily there is a very simple solution for this. In Visual Studio right click your project file and select “Properties”, on the left menu select “Build Events”. In this dialog you can add commands that will run either before or after a build. I suggest adding gulp to the “Post-build”. 

You will need to add two lines, the first line telling the command line where the location of your gulpfile is and the second line is just for triggering Gulp itself.
```bash
cd $(ProjectDir)Scripts\
gulp
```
The <code>$(ProjectDir)</code> is a macro which you can insert with the edtior in the properties pages.

After adding these lines every time you build your ASP.NET solution will trigger a build with Gulp which will run the Browserify plugin to generate your modules.

### Minification
When developing web applications it’s very common to minify your JavaScript to minimize the amount of data sent to the client. Fortunately this if very simple when you already have started using Gulp.js. 

At first you will need to install a package called [gulp-uglify](https://www.npmjs.org/package/gulp-uglify) (there are other packages which do the same thing) by in the command line running the command <code>npm install –save-dev gulp-uglify</code> also make sure you are in the directory where your gulpfile.js is located.

Now we can create a new task in our gulpfile.js for minifying our JavaScript modules.
```javascript
var gulp = require('gulp'),
    uglify = require('gulp-uglify'),
    browserify = require('gulp-browserify');
gulp.task('scripts', function(){
    gulp.src('start.js')
        .pipe(browserify({
            insertGlobals: false,
            debug: false
        })).pipe(gulp.dest('./build'));
});
gulp.task('minify', function () {
    gulp.src('./build/start.js')
        .pipe(uglify({
            outSourceMap: true
        })).pipe(gulp.dest('./dist'));
});
gulp.task('default', ['scripts', 'minify']);
```
When building your ASP.NET application now a folder called “dist” will be created in the “Scripts” folder which will contain the exact the scripts as created in the “scripts” task but this time they will be minified.

If you try this out you will see that this minifying task takes a couple of seconds and it’s perhaps nothing you would like to do every time you build your solution after all the minified scripts comes in handy in a production environment. So next I will show you how to take the build configuration in Visual Studio in count when running your gulpfile.

### Debug or Release
First we will add a new row to the post-build instructions in Visual Studio. 
```bash
cd $(ProjectDir)Scripts\
set NODE_ENV=$(ConfigurationName)
gulp
```

This row will let you read which build symbol is used in Visual Studio when compiling your solution. To access this variable you can use <code>process.env.NODE_ENV</code>.

So we can use this piece of code to check if we are compiling in Debug or Release mode and then decide if we should run the minifying task or not.
```javascript
...
var tasksToRun = ['scripts'];
if(process.env.NODE_ENV === 'Release'){
    tasksToRun.push('minify');
}
gulp.task('default', tasksToRun);
```

### Automatically build your scripts
Finally we will take a look at how you can let Gulp automatically notice a change in your script files and run the build task for you. This is something you wouldn’t want to add to you gulpfile.js which Visual Studio builds when compiling because this task will not return and the build will not complete. So I would suggest that you add another if-statement and check for a different environment variable that you choose to pass when you want Gulp to watch your changes and then just manually run the gulpfile.js from the command line during development.

So let’s install yet another node package called [gulp-watch](https://www.npmjs.org/package/gulp-watch) by running the command <code>npm install -–save-dev gulp-watch</code> in your command line and then add the following task to your gulpfile.js
```javascript
var watch = require('gulp-watch');
gulp.task('watch', function(){
    watch({ glob: '*.js' }, function(){
        gulp.start('scripts');
    });
});
```
Now when you run Gulp the build process won’t stop it will listen for changes in your script files and when any of them are changed the script task will be run.

## Summary
In this post I have showed you how to write JavaScript modules which depend on other modules and how to solve that issue using Browserify.

We also used Gulp.js to automate our build process by integrating it into Visual Studio. 

I think using Browserify can be a very nice way to structure you JavaScript when building lager web applications and with Gulp.js you can easily automate minification, bundling and event more (there is a lot of plugins for Gulp such as less and sass).

Thanks for reading this post, if you have any questions or comments please feel free to [tweet me](http://www.twitter.com/ridderholt) or leave a comment below. If you are interested in how the source code for this post looks feel free to clone my repository from [GitHub](https://github.com/robinridderholt/gulp-browserify-aspnet).



