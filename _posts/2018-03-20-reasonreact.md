---
layout: post
title: "ReasonReact"
author: "Robin Ridderholt"
categories: journal
tags: [Functional programming,JavaScript,ReasonML]
---

Since I started to play with functional programming my go-to language has been [F#](https://www.fsharp.org). As a C# developer it was an easy choice since I did't have to leave the safe environment of .NET. Since then I have played with Haskell, Erlang, Elixir and some other languages and I realized that the syntax felt nice and clean in F#. F# is a [ML](https://en.wikipedia.org/wiki/ML_(programming_language)) language mostly influenced by [OCaml](https://ocaml.org/) and when I looked into other ML language I found that the syntax in general appealed to me.

When Facebook released [ReasonML](https://reasonml.github.io/) it was on my radar to dive into and when Jared Forsyth was on [Functional Geekery](https://www.functionalgeekery.com/episode-119-jared-forsyth/) talking about ReasonML and [ReasonReact](https://reasonml.github.io/reason-react/docs/en/installation.html) I knew it was time.

## Redux built-in
One of my preferred ways of working with [React](https://reactjs.org/) is using [Redux](https://redux.js.org/) where you extract the state from the components and manage it by triggering actions and modify your state through reducers, this and other goodies like router, subscriptions and JSX is baked into ReasonReact.

## Why not Fabel?
[Fabel](http://fable.io/) is a F# to JavaScript compiler and through [Fabel.Elmish](https://fable-elmish.github.io/) you get very similar functionality as in ReasonReact. For me the deal breaker is the JSX support. 

In Fabel.Elmish you need to write these element functions to create the layout for your components:
```fsharp
let view model =
  R.div []
      [ R.div [] [ R.str (sprintf "%A" model) ] ]
```

But when using JSX you can write:
```fsharp
let component = ReasonReact.statelessComponent("MyComponent");

let make = (model, _children) => {
  ...component,
  render: _self =>
    <div> <div> (ReasonReact.stringToElement(model)) </div> </div>
};
```

# Writing a Pomodoro timer using ReasonReact

If you haven't already installed ReasonReact, there is a guide for how to do it [here](https://reasonml.github.io/reason-react/docs/en/installation.html). I'm using Reason Scripts (create-react-app) in this example.

The [Pomodoro techinque](https://en.wikipedia.org/wiki/Pomodoro_Technique) basically means that you work 25 mintues and take 5 mintues break and then start over. So this exampel is a timer that counts down from 25 mintues to zero.

## The state
So for that state I'm going to need to keep track of minutes and seconds, I'm also going to need a timer so my state looks like this:

```fsharp
type state = {
  minutes: int,
  seconds: int,
  timerId: option(Js.Global.intervalId)
};
```
Notice that the `timerId` is an option, since I'm not going to have a timer all the time and `null` or `undefined` isn't a concept in ReasonML we need to tell the compiler that their will either be a value or not.

The type `Js.Global` is build into ReasonReact to provide an interop to JavaScript and has a limitied set of basic JavaScript functionality.

## The actions
There will be a few actions, first of all you need to start and stop the timer and also reset it to start over. The timer will tick every second and we need to react to that tick so we can print the current state of the timer. So my actions looks like this:

```fsharp
type action =
  | Start(Js.Global.intervalId)
  | Stop
  | Reset
  | Tick;
```

The `Start` action has a value containing the `intervalId` for the timer which we will need when stoping the timer and we will save it to the state when recieving the start action.

## Other functions
Whenever we recieve the Tick action we need to update the state with new values for minutes and/or seconds.

```fsharp
let calculateTime = state =>
  switch (state.minutes, state.seconds) {
  | (_, 0) => {...state, minutes: state.minutes - 1, seconds: 59}
  | _ => {...state, seconds: state.seconds - 1}
  };
```
We create a tuple of minutes and seconds and using pattern matching we can see if the seconds has reached zero and we can subtract one from minutes and set seconds to 59 otherwise we just keep subtracting 1 from seconds.

In order to be able to determin if the counter has counted all the way down to zero and stop the timer we can check if both minutes and seconds is zero:

```fsharp
let isFinished = state =>
  switch (state.minutes, state.seconds) {
  | (0, 0) => true
  | _ => false
  };
```
To clear the timer we can use the `Js.Global.clearInterval` function, since we declared the `timerId` to be an option in the state we need to use pattern matching to handle the case when `timerId` has a value and when it doesn't:

```fsharp
let clearTimer = timerId =>
  switch timerId {
  | Some(timer) => Js.Global.clearInterval(timer)
  | _ => ()
  };
```
If we get `Some` value we can clear the interval but if the `timerId` has no value we don't to anything.

## The reducer
Now that we have our state, actions and some helper functions we can write our reducer.

The reducer is just a function that takes the action and the current state and returns a new state, this a simple concept but notice that we don't actually mutate the state we return a new one since the state is immutable. Using the spread operator `...` we do not have to set every property in the state that we don't need to change the spread operator means that all other data in the state is copied and just the `timerId` is changed.

Let's start with handling the `Start` action:
```fsharp
let counterReducer = (action, state) =>
  switch action {
  | Start(timer) => ReasonReact.Update({...state, timerId: Some(timer)})
 
```
Using pattern matching on the action we check if we get a `Start` action with our timer, using the build in `ReasonReact.Update` function we create a new state where the `timerId` property has been set to `Some(timer)`.

Now for the `Stop` action:

```fsharp
let counterReducer = (action, state) =>
  switch action {
  | Start(timer) => ReasonReact.Update({...state, timerId: Some(timer)})
  | Stop =>
    ReasonReact.UpdateWithSideEffects(
      {...state, timerId: None},
      (_self => clearTimer(state.timerId))
    )
```
When we recieve a `Stop` action we want to set the `timerId` to `None` in the state and also clear the interval so we use the `ReasonReact.UpdateWithSideEffects` function to both update the state and call our `clearTimer` function. Note that the side effect function takes `self` as a parameter and since we don't use it in our `clearTimer` function we need to name it with an underscore to tell the compiler that we aren't going to use it.

Next up is the `Reset` action where we want to reset the minutes to 25 and seconds to 0 and if the timer is running we want to make sure that it will be stoped:

```fsharp
let counterReducer = (action, state) =>
  switch action {
  | Start(timer) => ReasonReact.Update({...state, timerId: Some(timer)})
  | Stop =>
    ReasonReact.UpdateWithSideEffects(
      {...state, timerId: None},
      (_self => clearTimer(state.timerId))
    )
  | Reset =>
    ReasonReact.UpdateWithSideEffects(
      {...state, minutes: 25, seconds: 0},
      (self => self.send(Stop))
    )
```
We are using the `ReasonReact.UpdateWithSideEffects` again and updating the state and this time we use the `self` parameter to send the `Stop` action which will handle the stopping the timer.

Last we need to handle the `Tick` action and it's a bit special since we have two cases, either the `Tick` action will just update the state with new values for minutes and seconds or we have reached zero and we need to stop the timer.

```fsharp
let counterReducer = (action, state) =>
  switch action {
  | Start(timer) => ReasonReact.Update({...state, timerId: Some(timer)})
  | Stop =>
    ReasonReact.UpdateWithSideEffects(
      {...state, timerId: None},
      (_self => clearTimer(state.timerId))
    )
  | Reset =>
    ReasonReact.UpdateWithSideEffects(
      {...state, minutes: 25, seconds: 0},
      (self => self.send(Stop))
    )
  | Tick when isFinished(state) =>
    ReasonReact.SideEffects((self => self.send(Stop)))
  | Tick => ReasonReact.Update(calculateTime(state))
  };
```
Using the `when` keyword we can add a condition to our pattering matching to say that when the `Tick` action is sent **and** the timer `isFinished` whe just trigger a side effect by sending the `Stop` action. Otherwise we just use our `calculateTime` function to update the state with the new values for minutes and seconds.

## The component

In ReasonReact you start by declaring what type of component you want, in our case we want a reducer component.

```fsharp
let component = ReasonReact.reducerComponent("counter");
```
Unlike React where you create a class or a function as your component in ReasonReact we crate a `make` function:

```fsharp
let make = _children => {
  ...component,
  initialState: () => {minutes: 25, seconds: 0, timerId: None},
  reducer: counterReducer,
  render: self => <div></div>
};
```
In the `make` function we spread the component variable we created earlier and declare what the initial state should look like by creating an instance of our state type. We also pass our `counterReducer` function to the component and declare a `render` function which we will add some more markup to.

To render a text or in our case a number we can use the `ReasonReact.stringToElement` function so lets add markup for displaying the minutes and seconds:

```fsharp
let make = _children => {
  ...component,
  initialState: () => {minutes: 25, seconds: 0, timerId: None},
  reducer: counterReducer,
  render: self =>
    <div className="counter">
      <span className="counter__minutes">
        (ReasonReact.stringToElement(pad(self.state.minutes)))
      </span>
      <span className="counter__divider">
        (ReasonReact.stringToElement(":"))
      </span>
      <span className="counter__seconds">
        (ReasonReact.stringToElement(pad(self.state.seconds)))
      </span>
    </div>
};
```
The `pad` function is a small function I wrote to display `01` instead of just `1` to make the timer look a little more clocklike and it looks like this:

```fsharp
let pad = n =>
  if (n <= 9) {
    "0" ++ string_of_int(n);
  } else {
    string_of_int(n);
  };
```
The `string_of_int` function is built in and it just converts an integer to a string.

Now lets add some buttons to which the user can use to start, stop and reset the timer:

```fsharp
let make = _children => {
  ...component,
  initialState: () => {minutes: 25, seconds: 0, timerId: None},
  reducer: counterReducer,
  render: self =>
    <div className="counter">
      <span className="counter__minutes">
        (ReasonReact.stringToElement(pad(self.state.minutes)))
      </span>
      <span className="counter__divider">
        (ReasonReact.stringToElement(":"))
      </span>
      <span className="counter__seconds">
        (ReasonReact.stringToElement(pad(self.state.seconds)))
      </span>
      <div className="counter__actions">
        <div className="counter__actions--start">
          <button
            className="waves-effect waves-light btn-large green"
            onClick=(
              _event =>
                self.send(
                  Start(Js.Global.setInterval(() => self.send(Tick), 1000))
                )
            )>
            (ReasonReact.stringToElement("Start"))
          </button>
        </div>
        <div className="counter__actions--stop">
          <button
            className="waves-effect waves-light btn-large red"
            onClick=(_event => self.send(Stop))>
            (ReasonReact.stringToElement("Stop"))
          </button>
        </div>
        <div className="counter__actions--reset">
          <button
            className="waves-effect waves-light btn-large orange"
            onClick=(_event => self.send(Reset))>
            (ReasonReact.stringToElement("Reset"))
          </button>
        </div>
      </div>
    </div>
};
```
In the `onClick` attributes of the buttons we simply send the corresponding action. For the start button we use the `Js.Global.setInterval` function to send the `Start` action. The `setInterval` function will return a `timerId` and it takes a function that will be triggered every 1000 millisecond and in our case we want to send a `Tick` action every second.

## Styling
I have used [Materialize](http://materializecss.com/) as a shortcut to get some nice styling. But you can add some custom CSS by adding a normal stylesheet file to your solution and since this is all powered by [Webpack](https://webpack.js.org/) you can require that CSS file. Requiring a CSS file isn't something that is directly supported by ReasonReact instead you need to interop with Bucklescript by adding this to your file: `[%bs.raw {|require('./Counter.css')|}];` this line will render a require statement in the JavaScript version of your Reason file which then will be processed by Webpack.

# Summary
If you are like me and you like ML styled languages and are into React and Redux you should look into ReasonReact. ReasonML is a typesafe functional language with a lot of promise, they work hard to get compiler errors as good as in [Elm](http://elm-lang.org/). The ReasonReact project is still far from done but they have a great thing going and it will be exciting to see where they end up.

I hope that this post has been to some use for you, the source code is available on my [GitHub](https://github.com/ridderholt/reason-pomodoro).

