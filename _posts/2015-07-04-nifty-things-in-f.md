---
layout: post
title: "Nifty things in F#"
author: "Robin Ridderholt"
categories: journal
tags: [F#]
---

For a couple of years I have had an on and off again relationship with F# I also did a "Introduction to F#" [presentation at a conference](https://www.youtube.com/watch?v=XhPX3zYCMLA). Lately I have been digging more deeply into F# in my spare time and I thought I share two small yet powerful things that is very "nifty" in F#.

## No more NullReferenceExceptions
Anyone who has written any sort of application is likely to have gotten a NullReferenceException. We have all been there, using methods without really thinking about what it will return when given certain values and BOOM you forgot to check for null.

In F# there is a concept called [`option`](https://msdn.microsoft.com/en-us/library/dd233245.aspx) which can be compared with [`Nullable<T>`](https://msdn.microsoft.com/en-us/library/b3h38hb0.aspx) in C#. This concept or pattern is very common in F# and it lets us to write functions that return a value or no value. You may be thinking that that's exactly the same as returning a value or null from a C# method but the smart thing about `option` is that the F# compiler will warn you if you don't write code for managing both the value or the `None` value.

When working with `option` there are two keywords you need to be familiar with and they are `Some` and `None`. When returning a value from a functions that returns an option you would write `Some(10)` which would return an option of the type `int` with the value of 10. When you don't want to return anything you simply return `None`.

A very simple example of where this can be useful is when reading input from the console. Lets say that you're asking the user to enter a number and your function `readline` reads the value from the console and tries to parse it as an integer. If the value that the user entered was not a valid integer then you want to return `None` from that function otherwise you return `Some(value)` where value is the parsed integer that the user entered.

```fsharp
let readLine =
  let input = Console.ReadLine()
  match Int32.TryParse input with
  | true, value -> Some(value)
  | _ -> None

```
In this example we take advantage of `Int32.TryParse` which in F# will return a tuple with the first value being a boolean indicating if the value could be parsed and the second item is the parsed value. If the value could not be parsed the second value will be 0. So with some simple [pattern matching](https://msdn.microsoft.com/en-us/library/dd547125.aspx) we check for a valid value and then returns `Some(value)` or when the value is invalid we return `None`.

When using the return value from the `readline` function we can use pattern matching to check for the parsed value and as I mentioned earlier the F# compiler will complain if we don't handle the `None` path of the code. This is what a the code may look like when handling both cases:

```fsharp
let integerValue = readLine

match integerValue with
| Some(iv) -> printf "You entered %d" iv
| None -> printf "Incorrect value"
```

As I have mentioned this is a very common pattern in F# and a very easy way to avoid getting those NullReferenceExceptions.

## Unit of measure
Another nifty feature of F# is the built in way to manage units of measure with types. In some cases when writing an application you may what to handle different units of measure. There is a famous example of the [Mars Climate Orbiter](https://en.wikipedia.org/wiki/Mars_Climate_Orbiter) which unexpectedly went out of its orbit and disintegrated. Later when NASA found out what happened it was reveled that the two teams of programmers working on the software for the Orbiter was measuring speed and other inputs using different units of measure, one team measured things using the metric system and the other team used the imperial system.

To make things like this easy F# has a way to declare types as a `Measure` like this:

```fsharp
[<Measure>] type kmh
[<Measure>] type mph
```

This enables you to declare variables and specifying in which unit of measure this variable should be used with. So if we want to declare a speed in Km/h we simply write:

```fsharp
let swedishSpeed = 130.0<kmh>
let britishSpeed = 70.0<mph>
```

When we want to make sure that only one type of unit of measure can be passed to a function we can set this as the type of the input:

```fsharp
let toFast (speed:float<kmh>) = speed > 120.0<kmh>
```

And if we want to convert between unit of measures we have to write a function that takes one measure as the parameter and returns the measure we want to covert to:

```fsharp
let convertToKmh (speed:float<mph>) = speed * 1.6<kmh/mph>
```

If you specify the type of your parameters the compiler will warn you if you are trying to pass a parameter of the wrong unit of measure to it.

## Summary
Using these two simple and powerful features of F# you can write applications which are less likely to encounter a `NullReferenceException` using `Some` and `None` when a function could return a value or nothing and the compiler will help you to make sure you handle cases when a function may return `None`.

We have also looked at the feature `Measure` in F# which lets you define units of measure in your code to make sure there is no logical conversion error.

If you have any questions or comments regarding this post please leave a comment below. Thanks for reading!



