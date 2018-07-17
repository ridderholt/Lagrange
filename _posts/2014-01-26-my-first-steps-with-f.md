---
layout: post
title: "My first steps with F#"
author: "Robin Ridderholt"
categories: journal
tags: [F#,functional]
---

In the last year or so functional programmign has gained in popularity. I was curious so I thought I would start with a functional language that was closley releated to my everyday language C#.

## Why functional?
When I started to look at functional programming it turns out that there is a lot of great concepts that could be applied to my every day C# OO programming. 

### Immuatable variables and parallell code
In most functional languages all variables are immuatable wich means that once you assinged a value to them the value can't be changed. This was a concept that was kind of hard for me to grasp at the beginning because I'm very used to maintain and change state using variables in my OO programs.

So why is this a good concept? One of the basic concepts of functional programming is that you easily should be able to parallelize your code. When you don't have variables in memory that can be changed at any given moment you don't have to worry about syncronize your data between threads. This will really helps if you want to run code in parallell.

#### What if I really need to have a mutable variable?
No worries, in F# you can have mutable variables. But you will have to declare them as mutable explicitly:
```fsharp
let mutable x = 0 //Mutable variable
let y = 1 //Immutable variable
```
So you can have mutable variables but it's recommended that you stick to the immutable concept as far as possible.

### Small reusable functions
The next concept that I really like with the functional way of thinking is the concept with small reusable functions. The idea is that you write these functions that makes small changes to the data by sorting, mapping or filtering it and then returns it. In this way you can combine your functions in different way to suit your needs in different situations. The concept is called [MapReduce](http://en.wikipedia.org/wiki/MapReduce) and is widely used in distributed and parallel systems.

Here is a example of how you can combine small functions to sort, filter and print an array:
```fsharp
let result = [0..100] |> mySortingFunc 
                      |> myFilterFunc 
                      |> myPrintFunc
```
In this short example I'm creating an array with numbers from 0 to 100 and then using the <code>|></code> sign to pipe my data from function to function.

## My first steps
I have found that whenever I try to learn the basics of a new language that doing small [Katas](http://en.wikipedia.org/wiki/Kata_(programming)) really helps me get going. All to often I have wanted to learn something new but got stuck thinking about what problem to solve using this new thing.

### FizzBuzz
A classic Kata is [FizzBuzz](http://en.wikipedia.org/wiki/FizzBuzz) and that's the problem I started with when I was going to learn some F#.

Since I'm a C# developer I bought this book [F# for C# developers](http://shop.oreilly.com/product/0790145366641.do) and my C# skillz also influenced my first solution to the FizzBuzz problem:
```fsharp
member this.Calculate (n:int) =  
    if n%3 = 0 && n%5 = 0 then
        "FizzBuzz"
    elif n%3 = 0 then
        "Fizz"
    elif n%5 = 0 then
        "Buzz"
    else
        n.ToString()
```
I think most developers should be familiar with this implementation and it's a very basic if-else implementation but it's also very C#-ish implementation.

My next try was a little more F#-ish:
```fsharp
member this.Calculate n =
    match n with
    | i when i%3 = 0 && i%5 = 0 -> "FizzBuzz"
    | i when i%3 = 0 -> "Fizz"
    | i when i%5 = 0 -> "Buzz"
    | _ -> n.ToString()
```
In this example I'm using [Pattern matching](http://en.wikipedia.org/wiki/Pattern_matching) insted of if-else statements. To clearify: `i` in this example is acting like a alias for the number `n` that is passed to this function. I also have removed the type definition `(n:int)` from the previous example. This is another cool feature of F# that is called [Type Inference](http://msdn.microsoft.com/en-us/library/dd233180.aspx) wich means that the enviroment can infere the type. Think about it like `var i = 0` in C# where the compiler understand that `i` is an integer. 

I was pretty happy with this implementation at first but after a while learning more of F# as a language it started to look ugly to me and after some pair programming with a fellow [developer](https://github.com/lette) I ended up with this:
```fsharp
member this.Calculate n = 
    match (n%3,n%5) with
    | (0,0) -> "FizzBuzz"
    | (0,_) -> "Fizz"
    | (_,0) -> "Buzz"
    | (_,_) -> string n
```
As you can see I'm still using pattern matching and type inference but the code (in my humble opinion) is much more readable. For starters I'm creating a [F# Tuple](http://msdn.microsoft.com/en-us/library/dd233200.aspx) that holds the calculated modolus of <code>n%3</code> and <code>n%5</code>. I then  match the tuple with the FizzBuzz numbers. The <code>_</code> sign is used to declare that any number can be present here. At last i removed to <code>n.ToString()</code> wich is (to me) a C#-ish way to convert a number to a string and replaced it with the more F# style conversion <code>string n</code>

### Take aways
The more I lear about F# and functional style programming the more I love it. Concepts like writing small and reuseable functions is something that I'm trying to have in mind when programming C# and OO programs. Rembember that we also have ways to write functional style code in C# today! Linq is a great example of how functional thinking has helped us to simplify our code and programs. 

### Examples
If you want to play around with my example of the FizzBuzz problem or if you want to view the code I wrote for solving two other F# Katas (Backwards word and converting decimal number to roman numbers) please visit my [GitHub page](https://github.com/robinridderholt/F--Katas).


