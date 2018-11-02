---
layout: post
title: "Maybe it's time to stop using null"
author: "Robin Ridderholt"
categories: posts
tags: [C#, F#, null, option, maybe]
---

The usage of null was introduced by Tony Hoare back in 1965 in order to be able to create pointers that point to something that doesn't exists. This ha been called (by Mr. Hoare him self) as the billion dollar mistake.

Today most of us .NET developers doesnt's use pointers, not directly at least. But still we see a lot of `NullReferenceException` and perhaps spend time chasing bugs caused by some variable being null. So shouldn't we stop using nulls? Well the short answer is yes, in the cases where we know that a `NullReferenceException` could occur we should take action and make sure to prevent this.

## What to do instead of null?
In many functional languages the concept of null simply doesn't exist, Haskell has it's [Maybe monad](https://en.wikibooks.org/wiki/Haskell/Understanding_monads/Maybe) and F# has an [option type](https://en.wikipedia.org/wiki/Option_type). But C# doesn't really come with a pragmatic solution out of the box, so I thought that I would show you a simple case of fetching a person from a database using F# and C# and in both cases taking care of the null problem by using an option type.

## Simple example
In F# the option type is built in and is defined as an genric type `Option<T>` and to make use of it's value you often use pattern matching. If there is a value you will get `Some<T>` and if there is no value you will get `None`.

```fsharp
let person = Datalayer.getPersonById 12

match person with
| Some(p) -> printf "%s %s" p.firstName p.lastName
| None -> printf "No person was found"
```
In this simple scenario we ask the datalayer to fetch the person with id 12 from the database. So if the a person with id 12 doesn't exist the function will return `None` but if the person does exist we will get `Some<Person>`. 

We use pattern matching on the option type and match for the case where we find a person and then prints that person's full name, otherwise we just print `No person was found`.

That's all fine but what do we do in C#?
I recently came across a great Nuget package for C# called [Optional](https://github.com/nlkl/Optional) and it will give us more or less the same functionality as the built in option type in F#.

```csharp
public Option<Person> GetPersonById(int id)
{
    ...
}
```
The method in the datalayer is defined to return `Option<Person>` (an option of person). The package supports many ways to check and extract the value and one way to do it is call the `Match` function on the option type and pass two actions, one for the `Some` value and one for the `None` value.

```csharp
var person = _datalayer.GetPersonById(12);

person.Match(
    some: (p) => Console.WriteLine($"{p.FirstName} {p.LastName}"),
    none: () => Console.WriteLine("No person was found")
);
```

In cases where you would like to do some opertion and if it succeeded we should use the result of that operation otherwise we should use some sort of default you could do something like this:

```csharp
var option = Divide(2, 0);

var result = option.Match(
    some: (res) => res,
    none: () => 0
);
```
Here the result variable get assiged the value of a succesfull operation or the default value 0.

## Summary
There is tools available to us to start writing code that doesn't result in `NullReferenceException` and also is readable and forces the user of your function to handle both the case when they get a value and if they don't which is powerful since it all it takes is for some one to forget to write an if-statement to check for null.