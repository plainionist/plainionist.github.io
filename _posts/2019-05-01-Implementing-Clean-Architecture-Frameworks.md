---
layout: post
title: Implementing Clean Architecture - Frameworks vs. Libraries
description: |
  In Clean Architecture the Dependency Rule forbids usage of frameworks outside of the outermost circle.
  Does this mean that the usage of any third party API in the implementation of a gateway or repository
  is a violation to the Dependency Rule?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: JL0003
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.Frameworks.png" class="dynimg" title="Frameworks and libraries in the context of Clean Architecture." alt="In Clean Architecture the usage of frameworks is restricted to the outermost circle. But what is a framework? Is every third party library a framework? How to implement gateways without using third party libraries?"/>

It has been a while since [my last post](/Implementing-Clean-Architecture-AspNet/) on "Implementing Clean Architecture"
but I wasn't lazy ;-) In fact I was - apart from my day job - working on getting 
*[Athena](/Implementing-Clean-Architecture)* closer to the Clean Architecture. But there was one thing which puzzled me 
for a while when implementing [repositories](https://deviq.com/repository-pattern/):

In Clean Architecture
- all details are restricted to the frameworks layer. The database is a detail.
- all data conversion from the format most convenient for what ever persistence framework to the format
  most convenient for entities and use cases, happens in the interface adapters layer. 

How do I implement a repository in the interface adapter circle which accesses the TFS database using the 
Microsoft TFS framework APIs? Isn't that a violation to the Dependency Rule? In fact, isn't any usage of a third party
API outside the frameworks circle a violation to the Dependency Rule?

In this post I am trying to answer this question.

<!--more-->

## What is a framework? 

[Uncle Bob says](/Clean-Architecture):

> Don't marry the framework! Oh, you can use the framework - just don't couple to it. 
> Keep it at arm's length. Treat the framework as a detail that belongs in one of the outer circles of the 
> architecture. Don't let it into the inner circles.

Hhmm ... Do we have to handle each third party API in that way? 
Is [every framework equal or are some frameworks more equal then others](https://en.wikipedia.org/wiki/Animal_Farm)?

Let's look at some examples to get an idea ...


## What about the .NET framework?

Let's start with an obvious one: *[Athena](/Implementing-Clean-Architecture)* is built with [F#](http://www.fsharp.org)
on top of the .NET framework. Of course that does not mean that my whole project has to live in the frameworks circle ;-)

[Uncle Bob says](/Clean-Architecture):

> There are some frameworks that you simply must marry. If you are using C++, for example, you will likely
> have to marry STL - it's hard to avoid. If you are using Java, you will almost certainly have to marry the
> standard library.

Okay, obviously some frameworks are more equal then others. It doesn't make much sense to write modern C++ code without
STL and it is simply impossible to write .NET code without coupling to the .NET framework.

But what about other "standard" libraries like

- [Boost](https://www.boost.org/) is quite popular for C++ applications
- [FSharp.Data](http://fsharp.github.io/FSharp.Data/) is THE library for data access in F#
- And how do you do JSON in C# without [Json.Net](https://www.newtonsoft.com/json)?
 
It depends.

We have to decide carefully which framework we want to marry and which not. And maybe we should really take the word
"marry" seriously and think in terms of "until death (of the project) does both part": If we decide to marry a framework
and let it into the inner circles, it means we will (almost certainly) never get rid of it again and have to live 
with it until the project's life ends.

For *Athena* I have decided to marry only the .NET framework, F# and the FSharp.Core library.


## What about the UI?

As we have seen [earlier](/Implementing-Clean-Architecture-AspNet/) in the example with Asp.Net MVC,
UI frameworks tend to dictate quite some rules to an application. These frameworks want us to derive from 
their base classes, to implement our business logic as plug-ins to them and even come with patterns like 
[MVC](https://en.wikipedia.org/wiki/Model–view–controller) or [MVVM](https://en.wikipedia.org/wiki/Model–view–viewmodel) 
we have to follow.
Same is true other modern UI frameworks like [WPF](https://docs.microsoft.com/en-us/dotnet/framework/wpf/) or 
[VueJS](https://vuejs.org/).

Modern UI frameworks allow a fast and convenient development of applications but there are clear disadvantages
if we couple our applications to tight to them

- UI technologies change frequently and migrating to a completely different framework is expensive and painful
- Writing reliable UI based tests is challenging
- Quite often multiple UIs are needed (desktop, web, mobile)

Clearly we want to *use* UI frameworks as they provide a lot of benefits but we definitively do not want 
to *marry* them!

So how do we keep these frameworks "at arm's length"?

In most UI frameworks the View completely depends on the framework, e.g.

- The Asp.Net MVC ".cshtml" Razor template cannot live outside the Asp.Net MVC framework
- The WPF XAML window or control completely consists of WPF framework elements
- the VueJS component completely depends to the VueJS framework to call its hooks

So we have to keep the View in the frameworks circle completely.

The Controller and/or ViewModel on the other hand is usually less coupled to the framework. So we can follow
the approach of the [previous post](/Implementing-Clean-Architecture-AspNet/) and extract those code parts which
depend on the UI framework, keep them in the frameworks circle and let it call the framework independent parts 
which will live in the interface adapters circle.

<img src="{{ site.url }}/assets/clean-architecture/AspNet.Controllers.Clean.png" class="dynimg" title="Separating framework dependent and framework independent code of the controller" alt="The BacklogController does not derive from Asp.Net Controller any longer and contains most of the data conversion logic. BacklogAspNetController derives from Asp.Net Controller, converts data between Asp.Net and application and calls the BacklogController. Asp.Net dependencies are factored out of ReleaseBacklogViewModel into ReleaseBacklogAspNetViewModel"/>

In case we need to communicate from the adapters circle back to the frameworks circle we can simply provide
an interface in the adapters layer which will be implemented by the code in the framework circle and so maintain 
the Dependency Rule: All border crossing arrows are pointing "inwards".

<img src="{{ site.url }}/assets/clean-architecture/Adapter.Framework.Inversion.png" class="dynimg" title="Separating framework dependent and framework independent code of the controller" alt="The BacklogController does not derive from Asp.Net Controller any longer and contains most of the data conversion logic. BacklogAspNetController derives from Asp.Net Controller, converts data between Asp.Net and application and calls the BacklogController. Asp.Net dependencies are factored out of ReleaseBacklogViewModel into ReleaseBacklogAspNetViewModel"/>

Is it worth the effort? It depends on how much of your Controller's and ViewModel's code you want to keep independent
from the specific UI framework. If there isn't much such code you could also decide to move the Controller and ViewModel
to the frameworks circle completely. However I would not recommend to let the UI framework into your interface adapters
circle. Maintain a strict border between the framework and your adapters otherwise the framework will "take over" this
circle as well over time!


