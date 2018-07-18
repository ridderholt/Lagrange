---
layout: post
title: "Actually usable tuples in C#"
author: "Robin Ridderholt"
categories: journal
tags: [C#,Tuples]
---

If you, like me, have missed that C# now have support for actually usable tuples then you're in for a treat.

Previously if you didn't want to create a DTO class and return that from a function you could use the generic `Tuple<T>`. And you would write something like this:

```csharp
static Tuple<int, int, int> Sum(int x, int y)
{
    return Tuple.Create(x, y, x + y);
}
```

This syntax wasn't really the problem. The real issue was when you used this function:

```csharp
var tuple = Sum(1, 2);
Console.WriteLine($"The sum of {tuple.Item1} and {tuple.Item2} is {tuple.Item3}");
```

It is'nt very clear for the reader what `Item1`, `Item2` or `Item3` is more than that it is an integer.

## Tuples in C# 7

In C# 7 there is a new version of the tuple which makes it usable and that's both a better looking syntax and the concept of `named tuples`.

First lets rewrite the `Sum` fuction with the new syntax:

```csharp
static (int x, int y, int sum) Sum(int x, int y)
{
    return (x, y, x + y);
}
```

We can get rid of the `Tuple<T>` type completly and replace it with parentheses and the types along with the names. You will get suggestions in autocomplete for these names.

When you want to use this function the syntax is now more clear with the names from the tuple:

```csharp
var result = Sum(2, 3);
Console.WriteLine($"The sum of {result.x} and {result.y} is {result.sum}");
```

## Deconstruction

Along with the new syntax for tuples there is also syntax for deconstruction. This means that if we rewrite our `Sum` function to look like this:

```csharp
static (int, int, int) Sum(int x, int y)
{
    return (x, y, x + y);
}
```

Now that we aren't returning the names for each value in the tuple we can use deconstruction to assing names:

```csharp
var (x, y, sum) = Sum(1, 2);
Console.WriteLine($"The sum of {x} and {y} is {sum}");
```

Deconstruction, for now, only works for tuples out of the box. If you want to use deconstruction for your classes you can implement a `Deconstruct` function on your class:

```csharp
class Person
{
    public string Firstname { get; set; }
    public string Lastname { get; set; }

    public void Deconstruct(out string firstname, out string lastname)
    {
        firstname = Firstname;
        lastname = Lastname;
    }
}
```

You can then use deconstuction like this:

```csharp
var (firstname, lastname) = new Person { Firstname = "Robin", Lastname = "Ridderholt" };
Console.WriteLine($"{firstname} {lastname}");
```

# Summary

So there you go. This might seem like small changes but I think it's a nice change. It will get rid of many small classes that I write whenever I need to return more than one value from a function, and along with deconstruction we get a clean syntax.
