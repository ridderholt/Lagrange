---
layout: post
title: "Copy to clipboard with scriptCS"
author: "Robin Ridderholt"
categories: posts
tags: [C#,scriptcs]
---

A few weeks ago I found my self in the situation where I needed to generate a hash based on some business logic from my project at work. Because this logic is embedded pretty deep in the logic it seemed as a hard task to achieve. Then it occurred to me that I had heard about this thing called [scriptCS](http://www.scriptcs.net) which should be able to help me out.

## What is scriptCS
ScriptCS is essentially a framework for writing C# code i a script like manor. It even got full [Nuget](http://www.nuget.org) support. 

You simple write some code, save it as a .csx file and you can run it from the command line. You don't need to use Visual Studio so if you prefer any other editor that's totally fine. My favorite thing is that you now can use a programming language that you are familiar with instead of learning PowerShell for example.

## Install scriptCS
The easiest way to install scriptCS is to use [Chocolatey](https://chocolatey.org/) and if you don't know what Chocolatey is you have missed the best thing since sliced bread. It's like Nuget for Windows, just run the command "cinst" and the name of the program you wish to install and it will install it for you and the best thing is that it will also install any dependency that program would require.

So to install scriptCS simply run the command:
```bash
cinst scriptcs
```                
Now you should be able to type "scriptcs" into to command line and the environment should spin up and you can actually begin to write C# right there in the command line window.

## The problem at hand
So my problem was that I needed to generate a hash from some business logic and I could have written some new console application which referenced my .dll file but that seemed like to much overhead for a very simple task. I also knew that I would be generating a lot of these hashes and I wanted it to be very easy.

In scriptCS you can import a .dll file simple by typing:
```csharp
#r "myLogic.dll"
```
Before you run your application you need to make sure that the .dll file you wish to use is located in a bin folder in the same directory as you .csx file.

So I started with importing the .dll file containing my business logic for generating my hash:
```csharp
#r "myLogic.dll"

var hashGenerator = new HashGenerator();
var hash = hashGenerator.GetHash();

Console.WriteLine(hash);
```
Great! My hash was generating and was printed in my console window. Now a new problem arose, it's not that easy to copy some output from a program in a console window and paste it somewhere else.

I knew I had written some code years ago that saved some text to the clipboard for me, so I looked it up and found out that the only way to achieve this i C# is to reference, brace yourself, System.Windows.Forms.

Well, that shouldn't be to difficult. So I added a using statement in the top of my file
```csharp
using System.Windows.Forms;
```
In scriptCS you can reference .NET core libraries from the GAC in this way.

Now I had access to the Clipboard so instead of sending the output of my HashGenerator to the console window I added 
```csharp
Clipboard.SetText(hash);
```
So I ran my script and I got this error:
"Current thread must be set to single thread apartment (STA) mode before OLE calls can be made. Ensure that your Main function has STAThreadAttribute marked on it."

Okey, so I just have to mark my main method with the attribute [STAThread]. But wait, I'm using scriptCS and there is no main method to put the attribute on.

After some googeling I found out that this was not going to be as easy as I thought. What you have to do is to start a new thread and setting that thread to be a STA thread.
```csharp
var thread = new Thread(() => Clipboard.SetText(hash));
thread.SetApartmentState(ApartmentState.STA);
thread.IsBackground = false;
thread.Start();
thread.Join();
```
That's it, now you can use scriptCS to send things to your clipboard.

## My pull request
This solution worked very well for my small problem, but if you are building a larger script then creating a new thread for each task you wish to do is pretty redundant and I started looking for a Nuget package that could help me with this.

I found [ScriptCS-GUI](http://hemme.github.io/scriptcs-gui/) which can help you with prompting an OpenFileDialog or SaveFileDialog. In the repository I found that the creator had built a structure for managing this thread issue rather well. Unfortunately there was no clipboard support in his Nuget package. 

Having read trough his repository I could see that it shouldn't be that difficult to add a basic SetText and GetText from clipboard methods into his repository, so that what I did. I also got my pull request accepted a few weeks ago as you can see in the [repository](https://github.com/hemme/scriptcs-gui). This was my first pull request ever and it felt pretty great to be able to help someone to build a cool Nuget package for scriptCS.

## Summary
If you are a programmer and you need to write a script for some task, take a look at [scriptCS](http://www.scriptcs.net). I think your scripts will be more readable and therefore moreeasy to use and maintain.


