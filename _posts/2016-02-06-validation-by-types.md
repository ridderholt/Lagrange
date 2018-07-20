---
layout: post
title: "Validation by types"
author: "Robin Ridderholt"
categories: posts
tags: [F#,C#,Validation,DDD]
---

First of all, what I'm about to present isn't my own idea. This has been covered by Scott Wlaschin in his article about [A functional approach to Domain Driven Design ](http://fsharpforfunandprofit.com/ddd/). But I recently attended the conference [Swetugg](http://www.swetugg.se) and listened to [Anders Hallberg](https://twitter.com/andhallberg) talking about secure programming in .NET where he talked about some similar ideas.

Their idea is that if we use the DDD idea about creating small classes or value objects to model our domain such issues as validation becomes more easy to do.

Instead of trying to decorate your models with metadata of how a particular property should be validated you model your aggregate with small value objects which can validate them self upon creation.

Consider this simple example where we will model a simple person type with a name and an email address. If we just create two properties Name and Email both of the type string, then we would have to validate the email for the person every time an email should be sent to the user. But if we model the person with only the Name property with the type of string and creates a value object of the email. Then in the constructor of the email type we can validate the email and if it's not a valid email then we throw an exception.

```fsharp
module Email =
  type T =
  | ValidEmail of string

  let create mail =
    match mail with
    | m when m <> "" -> ValidEmail(mail)
    | _ -> invalidArg "mail" "Invalid mail"

module Person =
  type T = { Name : string; Email: Email.T }
  let create name mail = {Name = name; Email = Email.create mail}
```

(or if you prefer C#)

```csharp
public class Email
{
  public readonly string Value;

  private Email(string mail)
  {
    Value = mail;
  }

  public static Email Create(string mail)
  {
    if(string.IsNullOrWhitespace(mail))
      throw new ArgumentException("Invalid email", nameof(mail));

    return new Email(mail);
  }
}

public class Person
{
  public readonly string Name;
  public readonly Email Email;

  public Person(string name, Email email)
  {
    Name = name;
    Email = email;
  }
}
```
In this way the person object can't be created without a proper validation of the email. So where ever you use the person object you can be sure that the person has a valid email. Obviously you shouldn't just check if the mail parameter is empty or not you should do a some additional regex validation but for the case of a simple example I choose to just verify that the string is not empty.

This is how you would create the person:

```fsharp
let person = Person.create "Robin" "email@domain.com"
```
C#
```csharp
var person = new Person("Robin", Email.Create("email@domain.com"));
```

## Next step
We have seen how creating value objects may improve and simplify our domain but Anders also spoke about another use of value objects that might not be so obvious at first, and this is an idea that me and my colleagues have been experimenting with in our day to day work.

Let's say that we have a database which uses integers as primary keys (with auto increment) in our tables. If you have a method that takes a id of a person as a parameter, your method maybe looks like this:

```csharp
public Person GetPersonById(int personId)
{
    return _personRepository.GetById(personId);
}
```
So how do we know that the value passed to our method is actually a valid id of a person? For all we know the value could be -1 or 0.

To solve this problem we can use a value object instead of the raw value of an integer.

```csharp
public class PersonId
{
  public readonly int Value;

  private PersonId(int id)
  {
    Value = id;
  }

  public static PersonId FromInt(int id)
  {
    if(id <= 0)
      throw new ArgumentException("Invalid id", nameof(id));

    return new PersonId(id);
  }
}
```

Now we have a class that represents our input value and in our method we can be sure that the value is not -1 or 0 when we go searching in the database.

```csharp
public Person GetPersonById(PersonId personId)
{
    return _personRepository.GetById(personId.Value);
}
```

Another benefit from using this value object is that the code becomes more readable, it's very clear for a developer what sort of input you are expected to pass this method. And yes, of course you can pass any other integer to the PersonId class but as a developer you get reminded to check what value you are actually passing when you are forced to create an instance of the PersonId class.

A sidenote here, when creating these value objects in C# you may want to implement your own version of IEquatable<T>, Equals, GetHashCode and some operators that will improve how you can work with these value objects (and this comes for free if you're using F#).

## Exceptions, really?
As you have seen in the examples above I have thrown exceptions when the data wasn't valid and that might seem strange to some people but as Anders Hallberg mentioned is his talk, getting a negative value for a id of a person isn't expected and in that case you throw an exception. In F# you could expand the union type to include a InvalidEmail type and then use pattern matching to check the type where ever needed but then you as a developer need to remember that and in C# you could go for a inheritance hierarchy but then again you need to remember to check for the invalid type in each method.

## Immutability
In the examples above F# value objects are immutable by default but in the C# examples I have deliberately made the value objects immutable. This is because after creation and validation of these objects you would like to trust that the value stays valid and if the value object isn't immutable there is no way to ensure that.  




