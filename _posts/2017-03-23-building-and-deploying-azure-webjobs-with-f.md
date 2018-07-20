---
layout: post
title: "Building and deploying Azure WebJobs with F#"
author: "Robin Ridderholt"
categories: posts
tags: [F#,Azure,WebJob]
---


If you have read some of my other posts you know that F# is a favorite languge of mine so when I was going to create an Azure WebJob to handle messages from the Azure ServiceBus it felt natural to write it using F#. It all seemed as a great idea but along the way I found some issues both with the building and with the deployment which I tought I would share.

## The building part

The first issue occure right away, I had just installed Visual Studio 2017 and all the Azure tools but there is no template for WebJobs and F# only for C#. But a WebJob is just a console application so I started of by creating a new console application.

My case was a very simple one, I was going to listen for messages on a Azure ServiceBus Topic and then relay that message to a given API endpoint.

### Packages

To create a WebJob you need a Nuget package so I simply installed it:

```
Install-Package Microsoft.Azure.WebJobs
```
Since I also needed to listen for messages on the ServiceBus I also needed the nuget package for that:

```
Install-Package Microsoft.Azure.WebJobs.ServiceBus
```

### Configuration

The configuration of the WebJob is very simple, in my ```Program.cs``` file I created a new instance of the class ```JobHostConfiguration```. The ServiceBus integration is added by an extension method for the ```config``` object by calling ```UseServiceBus()```.

Then you need to create a host and pass in the configuration.


```fsharp
open Microsoft.Azure.WebJobs

[<EntryPoint>]
let main argv = 
    
    let config = new JobHostConfiguration()
    config.UseServiceBus()

    let host = new JobHost(config)
    host.RunAndBlock()

    0
```

### Listening for messages

The simpliest way to listen for messages is to create a function with a parameter that represents the incoming message and decorate that parameter with an attribute called ```ServiceBusTrigger```. The attribute takes two arguments, the first being the name of the [Topic](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions#what-are-service-bus-topics-and-subscriptions) and the other is which [Subscription](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions#what-are-service-bus-topics-and-subscriptions) to use.

I created a new module to contain my function, the function can also take another argument of type ```TextWriter``` which lets you log information which will be avaiable to view in the Azure portal.

```fsharp
open Microsoft.Azure.WebJobs

module Listener = 

    let ProcessMessage ([<ServiceBusTrigger("[topicname]", "[subscription]")>] message : string) (log : TextWriter) = 
        ()
```

As long as you function is public it will automatically be discovered when the program runs as the entry point of the message from the ServiceBus.

I knew that the incoming message was going to be in a JSON format so I wrote a simple message parser that checks the message for information about what type of message is is and the parser returns a [discriminated union](https://fsharpforfunandprofit.com/posts/discriminated-unions/) with typed information about the message so I could use [pattern matching](https://fsharpforfunandprofit.com/posts/match-expression/) to decide what to do with the message:

```fsharp
match MessageParser.parse message with
| UserAdded info -> info |> sendUserAddedInfo 
| Empty -> log.Info "Recieved empty message"
| Unknown msg -> log.Info <| sprinf "Unknown message: %s" msg

```

The ```sendUserAddedInfo``` is just a HTTP call that takes the message and passes it to another API.

## Deploying

Now if we had created a C# console application we could have just right clicked on our project i Visual Studio and selected Publish and published our WebJob from there but for some reason there is no Publish option for F# console applications.

After a lot of googeling I found out that when pulishing a WebJob all you have to do is create a .zip file of the output in your ```bin/Release``` folder. In a production scenario it's recommended to let a deploy system like [Octupus Deploy](https://octopus.com/) take care of this for you but for the purpose of this blog post lets just create a .zip file of the output in your release folder.

The next step is to go to the Azure portal and create a new app service and you will also need a storage account. Once the app service is created you have a menu option called "WebJobs" if you click there and selects "+Add" in the top menu you will get a small menu (see picture).

![](/assets/img/new_webjob-2.png)

Give the WebJob a name and under "File upload" select the .zip file you created of the build output. 

In my case I was going to continously listen for messages on the ServiceBus so my WebJob was of the the type "Continous" but you can also create a WebJob that runs like a [CRON job](https://docs.microsoft.com/en-us/azure/app-service-web/web-sites-create-web-jobs).

If you don't want to store connection strings in a .config file in your code you can set these in the "Application settings" for the app service we created. 

## Summary
As shown it is possible to work with WebJobs using F# but the workflow isn't as polished as it is for C#. One might wonder why but I can't shake of the feeling that Microsoft isn't paying F# as much attention as it deserves, I guess we just have to wait and see if the F# community might fill the gap or we will have to wait for Microsoft to catch up.

