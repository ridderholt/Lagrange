---
layout: post
title: "Wox plugin using F#"
author: "Robin Ridderholt"
categories: posts
tags: [F#,Wox]
---

If you're using Windows and haven't checked out [Wox](http://www.getwox.com) you totally should. It's a program that gets you easy access to all your files, folder and programs without using your mouse. Simply press ALT+Space and a bar will appear and you can search.

Wox is written using .NET and C# and besides being a very useful program it also have a nice plugin story with about 80+ plugins that you can install.

I started looking at writing a plugin for Wox just out of curiosity and it turns out that according to their website that they support plugins written in C# or Python. So I thought that if they support C# then in theory it would be possible to write a plugin using F#.

For my day job I work with a company providing teams and sport clubs with a website and we have clubs mainly in Sweden but also in Germany, USA and UK so we have to manage time zones. To make this easier we store all dates as a Unix timestamp in the database, this means I constantly find my self converting dates to Unix timestamps and vice versa. There is a plugin for Wox that does this but I couldn't get it to work on my machine and it seemed like the perfect problem to try writing a plugin for Wox using F#.

## Writing the plugin
It's a very simple to create a plugin for Wox and it's documented on [their site](http://doc.getwox.com/en/), however some parts isn't quite clear but it's not that hard. You start by installing their Nuget package called [Wox.Plugin](https://www.nuget.org/packages/Wox.Plugin/).

Then you have to create a class and implement one interface ```IPlugin``` with two methods ```Init``` and ```Query```. The ```Init``` function is pretty self explanatory and it's triggered when you plugin get initialized. Here's how my ```Init``` function looks:

```fsharp
type Main() =

  interface IPlugin with
    member this.Init(context) =
      ()
```

The ```Query``` function is the function that handles the query you enter in Wox, it takes a Query object as a parameter and you need to return a list of ```Result```. And here's my implementation of the ```Query``` function:

```fsharp
member this.Query(query) =
    let list = new List<Result>()

    match parseDate query.Search with
    | Some(date) -> list.Add <| dateResult date
    | _ -> match parseUnix query.Search with
           | Some(unix) -> list.Add <| unixResult unix
           | _ -> ()
    list
```

Since Wox is all about searching for stuff and F# is great fit because it's a pretty classic map reduce scenario. However since Wox is written using C# your F# code will have a bit of OO feel to it.

In my plugin I used [NodaTime](http://nodatime.org/) which is another great Nuget package that if you're working with time zones you really should take a look at. The full implementation of my plugin is available at [GitHub](https://github.com/robinridderholt/wox-plugin-unixtime). If you have any questions please leave a comment or ping me on [Twitter](https://twitter.com/Ridderholt).

![Result](https://6pfwbq.bn1304.livefilestore.com/y3mlaGy7pPvU5r2fpePn2GLBUZIYXU5yHJkOZL0NXRgzQlfDKAK-RuTj-0aneh5f5XR5wtm1C-slTWV1Rh6Wm2_amMTki959eAO5SasAGg_LpIUB5c7Gx7vIf_GGwnASD_SszvAnsQvcIqmTzYI6PEl1JjxmHUI8xbra3we9AhrbNQ)

# TL;DR
It's possible to write a plugin for [Wox](http://www.getwox.com) using F#. Yaay!!!



