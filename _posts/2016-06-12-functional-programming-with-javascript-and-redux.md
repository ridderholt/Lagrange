---
layout: post
title: "Functional programming with JavaScript and Redux"
author: "Robin Ridderholt"
categories: journal
tags: [JavaScript,Functional Programming,Redux]
---

A couple of months ago I joined the team at the company that I work who is build a new product. They are using React, Redux and ASP.NET Core as their technologies.

After I had spent a couple of days writing code in this new project I started to realize that many of the techniques I was using was really functional techniques which I recognized from developing with F# or Haskell. I wondered why I hadn't realized this right away and I guess that I had never thought of JavasScript as being a functional language and sure it's not a pure functional language like Haskell but there is a lot of functions being thrown around when working with JavaScript.

So I thought I would share some of my experiences with React and Redux which in fact is a form of functional programming.

## Redux
If you haven't checked out [Redux](http://redux.js.org/) you should. Basically it's a library that helps you contain the entire state of your SPA app in a single object. The idea is (and I will come back this later on) that you treat the state as an immutable object and each time you want to change some value you create a new copy of the state with that value changed and use that copy as the new state of you application.

This is handle in a specific way in Redux by using [Actions](http://redux.js.org/docs/basics/Actions.html) and [Reducers](http://redux.js.org/docs/basics/Reducers.html). Actions is objects with information about how the state should change, these actions are dispatched and then handled by the reducers. The reducers receives the action and decide how the change the state depending of what type of action it is. Each action has a type property telling the reducer what type of action it should take.

I won't go any deeper into how Redux works but please check it out it will change the way you write JS apps!

## Functional thinking in JS apps
To be able to show some code in this post I wrote a simple Todo app (cliché I know) and it's available on [Github](https://github.com/robinridderholt/ReactTodo).

#### Immutability
The first thing I noticed when working with Redux is that, as I mentioned earlier, the state of the application is treated as an immutable object. So why is this important then? Well, first of all the state isn't truly immutable when working with Redux out of the box, the state is still a regular JavaScript object and you can change values in the state if you want to but it's not recommended. If you keep to the guidelines and only return a new copy of the state in your reducers then at least we can say that we treat the state as immutable.

The advantages is that the behavior of your application becomes much more predictable and it's easier to follow the flow of data trough your application. This is a huge advantage if you consider the fact that there is probably more developers than you who are going to be working on the project.

The Immutability also brings one quite amazing feature, the ability to undo. When the reducer returns a new copy of the state every time some value changes and if you then save the state as it was before the change in a separate place you can very easily implement a undo (and even a redo) feature.

As I mentioned the state isn't really immutable and you have to trust that every developer sticks to the guidelines and doesn't modify state directly. There is a way around this and that is to use [ImmutableJS](https://facebook.github.io/immutable-js/docs/#/) and implement the whole store using this library.

#### Everything is a function
The one thing that defines a functional language is the fact that (almost) everything is a function.

As in F# what looks like a variable declaration is actually a function declaration which simply takes no arguments and always returns the value 1.
```fsharp
let one = 1
```
This is true also in when using Redux. Every action is a function and every reducer is a function.

This is an example from the Todo app which shows the todo reducer:
```javascript
const todo = (state, action) => {
	switch(action.type){
		case 'ADD_TODO':
			return {
				name: action.name,
				id: action.id
			};
		case 'TOGGLE_TODO':
			if(state.id !== action.id){
				return state;
			}

			return Object.assign({}, state, {
				complete: !state.complete
			});
		default:
			return state;
	}
};
```

In Redux there is also a concept called middleware which lets you add cross cutting concern logic such as logging or general error handling and guess what, yes middleware is also functions. You simply register your functions with Redux and stuff just works. 

#### Higher Order Functions (HOF)
A HOF is a function that takes a functions as an argument, returns a function or both of the above. This is also used in Redux when you want to fetch data from the server, and because the fetching is asynchronous you can't just dispatch your action function and let the reducer modify your state. You have to make the request and when it's done then you dispatch your action with your data from the server to the reducer which then can modify the state.

To make this easy there is a package you can use called [redux-thunk](https://www.npmjs.com/package/redux-thunk). This package makes it possible to dispatch a function which takes the dispatch function itself as an argument and makes the request and when the request is done it will dispatch the action.

```javascript
export const fetchTodos = () => {
	return (dispatch) => {
		return fetch('/api/todos')
				.then(response => response.json())
				.then(json => dispatch(todosFetched(json)))
	};
};
```
Here I'm declaring the fetchTodos function which returns a new function (making it a HOF) and the execution of this function will be taken care of using react-thunk. I'm also using the [isomorphic-fetch](https://www.npmjs.com/package/isomorphic-fetch) package to make the ajax request.

## Summary
I have shown some examples of how functional thinking has made it's way into JavaScript. What inspired me to write this post was the fact that I was surprised that it took some time for myself to realize that what I was actually doing in JavaScript was actually functional programming.

Because of the popularity of both React and Redux in the JavaScript community perhaps we will also se a increase of functional thinking in the backend as well.

As you can see in some of the examples I have used some es2015 features like the spread operator and the arrow function, this is transpiled using [Babel](https://babeljs.io/). With this syntax being more and more widley used I think that it will help functional thinking on it's way due to the terse syntax.


