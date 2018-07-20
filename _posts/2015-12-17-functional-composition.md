---
layout: post
title: "Functional composition"
author: "Robin Ridderholt"
categories: posts
tags: [F#,C#]
---

In functional programming where functions is a first class citizen functional composition is very common. The basic concept is that you pipe the result of one function to the input of another function in order to create a new function. This may seem a little abstract but in this post I will try to show how it can be done in F# and also in C#.

## Filtering persons

First let's create a simple example by creating a person type

```fsharp
type Gender = Female | Male

type Person = {
  Name:string
  Age:int
  Gender:Gender
}
```

Now say that we have a list of persons that we want to be able to filter in different ways so we start by defining a basic function that takes a function and a person as two arguments and then applies that function on the person. Type definition <code>('a -> ('a -> 'b) -> 'b)</code>:

```fsharp
let filterPerson person predicate =
  predicate person
```

We now have a function that can apply any function to our person. So now if we want to filter our persons list by gender we can create a function that takes a gender and a person as parameters and uses our first function to return a bool value. Type definition <code>(Gender -> Person -> bool)</code>

```fsharp
let filterByGender gender person =
  filterPerson person (fun p -> p.Gender = gender)
```

And lastly we can take this even further by creating a function called "filterMales" which may seem a little verbose but the code will be very clear about what its intent is. Type definition: <code>(Person -> Bool)</code>

```fsharp
let filterMales person =
  filterByGender Male person
```

What we have done now is to start with a very general function that we then use to create new functions which does less general stuff. As you can see in the type definitions for the functions we have removed or hidden arguments and defined them more and more concrete. The last function is very clear in what it does both in the type definition and the name and this is the power of functional composition.

So if we want to put these function into use we could create a list of persons and then use the List.filter function to apply or filterMales function.

```fsharp
let persons = [{Name = "Jen"; Age = 34; Gender = Female};
{Name = "Greg"; Age = 27; Gender = Male};
{Name = "Chandler"; Age = 31; Gender = Male}]

persons |> List.filter filterMales
        |> List.iter (fun p -> printf "%s\r\n" p.Name)
```

## C# example
Functional composition is a very common practice in function programming but this is also possible in C# however it may not be as elegant as in a functional language like F# or Haskell.

As in the F# example we start with the most general function, applying a given function to a person:

```csharp
Func<Person, Func<Person, bool>, bool> filterPerson =
  (person, predicate) => predicate(person);
```

As you can see the C# code becomes a bit more verbose because we don't have the same type inference as in F#. Let's implement the filterByGender function in C#:

```csharp
Func<Person, Gender, bool> filterByGender =
  (person, gender) => filterPerson(person, p => p.Gender == gender);
```

And finally the filterMales function:

```csharp
Func<Person, bool> filterMales = 
  (person) => filterByGender(person, Gender.Male);
```

Now we can apply our function in C# using Linq:

```csharp
var persons = new List<Person>
{
    new Person {Name = "Jen", Age = 34, Gender = Gender.Female},
    new Person {Name = "Greg", Age = 27, Gender = Gender.Male},
    new Person {Name = "Chandler", Age = 31, Gender = Gender.Male}
};

persons.Where(filterMales)
       .ForEach(p => Console.WriteLine(p.Name));
```

## Summary
So basically functional composition is all about writing general functions which can be used to create more specific functions which makes up a program with many functions that can be reused in different ways. I found that this approach works as well in C# as in F# although it's a bit more verbose but it makes the code pretty clean an very easy to read.



