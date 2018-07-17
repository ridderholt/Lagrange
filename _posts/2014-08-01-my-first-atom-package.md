---
layout: post
title: "My first atom package"
author: "Robin Ridderholt"
categories: journal
tags: [JavaScript,Package,Atom,CoffeScript,GitHub]
---

A few weeks ago the GitHub team released a Windows version of their editor. Since I always love to try out new editors it was a no brainer to download Atom.

### What is Atom?
Atom is an open sourced, JavaScript based editor from the GitHub team. It's a editor which can be customized in pretty much every way. It lets you integrate with Node.js and with its modular design it can be easily extended. If you want to read more please visit [atom.io](http://www.atom.io). Since the editor i web-based you can bring up the debugging window (press Ctrl+Alt+I) which you may recognize from the Chrome web browser.

### CoffeScript
The way you write packages for Atom is with Coffeescript and a folder structure which you can read more about on the [atom documentation page](https://atom.io/docs/latest).

CoffeeScript is a language which is compiled down to JavaScript. As they say on the [CoffeeScript page](http://coffeescript.org/) "CoffeeScript is an attempt to expose the good parts of JavaScript in a simple way."

### My package
 For about a year now I have been using a [VIM](http://vimdoc.sourceforge.net/) plugin for my editors
 and when working with VIM I have found that it's very useful to have relative line numbers when coding. Relative line numbers means that the row number which you are currently on is always 0 and the numbers goes from 1 and upwards and downwards. This is very useful when for example deleting rows by entering the VIM command d+5+j which would delete five lines down. Please view my
 [readme file](https://github.com/robinridderholt/atom-relative-linenumbers/blob/master/README.md) for a image preview.

## Using your web skillz to create a package
The code for my package is very simple so I thought it would be a greate example to show how to create a custom package to extend Atom.

### Creating the plumbing
 On Atoms [documentation page](https://atom.io/docs/v0.120.0/creating-a-package) there is a good description of how the folder structure should be. But if you're lazy like me you can let Atom do all the work for you by from the Package menu selecting "Package Generator" and "Generate Atom Package". Atom will propt you for a location and a package name and then create all the plumbing for you.

### jQuery like development
Since Atom is a web based editor you can use a jQuery like development style by simple requireing
atom core library:
```javascript
{$} = require 'atom'
```
This way you can write jQuery style selectors to select and manipulate elements inside the editor.

Atom is using the [React framework](http://facebook.github.io/react/) from Facebook. I won't go into this riht now but I'm planning to write a blog post about React in the near future.

### My approach
So my idea was to find the selected row and change the line number of that row to 0 and then calculate the row number above and below. As the user changes row I would listen to the move-up and move-down events of the editor and recalculate row numbers as the current row changes.

So I started with declaring the events I wanted to subscribe to:
```javascript
events = [ 'core:move-up', 'vim-mode:move-up', 'core:move-down', 'vim-mode:move-down' ]
```
You will notice that in addition to listing the core event I also listed the vim-mode events, this is because I use the [vim-mode plugin](https://atom.io/packages/vim-mode) for Atom.

Next there is the declaration of the module and one important function declaration:
```javascript
module.exports =
  activate: (state) ->
```
The activate function is the only function that is required by Atom for a package. It's obviously used as a entry point for the whole package.

To start with I need to get a hold of the editor and start subscribing to the movement events:
```javascript
atom.workspaceView.eachEditorView (editor) =>
```

To subscribe to all the events at the same time we can use the on-method which you may recognize if you have worked with jQuery before.
```javascript
editor.on events.join(' '), =>
```
For clarification the events variable is the events array I declare earlier.

Next I wrote two small and simple functions, the first one is for getting a row number element and the second one is a function for settings the html for the row number element:
```javascript
_getRowElementByLineNumber: (lineNumber) ->
  $('.line-number[data-screen-row="' + lineNumber + '"]')

_setNewRowNumber: (rowElement, newNumber) ->
  $(rowElement).html('&nbsp;' + newNumber + '<div class="icon-right"></div>')
```
Now that we can get a row number element and set its value the next step is to set correct values for the rows above the current row:
```javascript
_recalculateBeforeCurrentRow: (currentLineNumber) ->
  counter = 1
  start = currentLineNumber - 1
  for i in [start...-1]
    row = @_getRowElementByLineNumber(i)
    @_setNewRowNumber(row, counter++)
```
And pretty much the same code is used for setting the row numbers for the rows after the current line:
```javascript
_recalculateAfterCurrentRow: (currentLineNumber, totalLine) ->
  counter = 1
  start = currentLineNumber + 1
  for i in [start...totalLine]
    row = @_getRowElementByLineNumber(i)
    @_setNewRowNumber(row, counter++)
```
So now all the logic is in place it is just the matter of putting it all together:
```javascript
_recalculateLineNumbers: (currentLineNumber, totalLines) ->
  currentRow = @_getRowElementByLineNumber(currentLineNumber)
  @_setNewRowNumber(currentRow, 0)
  @_recalculateBeforeCurrentRow(currentLineNumber)
  @_recalculateAfterCurrentRow(currentLineNumber, totalLines)
```
This function is then used in the activation method I showed in the beginning, and now the activation functions looks like this:
```javascript
activate: (state) ->
  atom.workspaceView.eachEditorView (editor) =>
    @_recalculateLineNumbers(editor.getEditor().getCursorScreenRow(), editor.getEditor().getLineCount())
    editor.on events.join(' '), =>
      atom.workspaceView.eachEditorView (editorView) =>
        @_recalculateLineNumbers(editorView.getEditor().getCursorScreenRow(), editorView.getEditor().getLineCount())
```
### Publishing
When you feel that you are done with your package you can publish it by simply running the command:
```bash
apm publish minor
```
But before you do this there is some configuration to be done and you have to publish your package to [GitHub](https://github.com/) before it can be published to Atom. Please read the ["creating a package"](https://atom.io/docs/v0.120.0/creating-a-package) for more information about configuration and more stuff that you can do with your package.

## Summary
I hope that this post have shown how easy it is to create a custom package for Atom and that it may inspire you to write your own. I think Atom has a greate potential to be a great editor, for now it's under development and the GitHub team are doing a great job and they are rolling out new releases every week.

## Source code
If you want to view the complete source code you can visit my package repository at GitHub [https://github.com/robinridderholt/atom-relative-linenumbers](https://github.com/robinridderholt/atom-relative-linenumbers). If you have any comments or questions please write a few lines below. Thanks for reading!

