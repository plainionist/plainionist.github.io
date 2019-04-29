---
layout: post
title: Implementing Clean Architecture - Frameworks vs. Libraries
description: |
  In Clean Architecture the Dependency Rule forbids usage of frameworks outside of the outermost circle.
  Does this mean that the usage of any third party library in the implementation of a gateway or repository
  is a violation to the Dependency Rule?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.Frameworks.png" class="dynimg" title="Frameworks and libraries in the context of Clean Architecture." alt="In Clean Architecture the usage of frameworks is restricted to the outermost circle. But what is a framework? Is every third party library a framework? How to implement gateways without using third party libraries?"/>

It has been a while since [my last post](/Implementing-Clean-Architecture-AspNet/) on "Implementing Clean Architecture"
but I wasn't lazy ;-) In fact I was - apart from my day job - working on getting *[Athena](/Implementing-Clean-Architecture)*
closer to the Clean Architecture. And there was one thing which puzzled me for a while when implementing "repositories":

How do I implement a repository in the interface adapter circle which accesses the TFS which lives in the frameworks 
circle using the official Microsoft TFS APIs? Isn't that a violation to the Dependency Rule?

In this post I am trying to answer this question.

<!--more-->

## What is a framework? 

Uncle Bob says:

> Don't marry the framework! Oh, you can use the framework - just don't couple to it. 
> Keep it at arm's length. Treat the framework as a detail that belongs in one of the outer circles of the 
> architecture. Don’t let it into the inner circles.

Hhmm ... does this mean that any third party library is a framework which has to banned to the outermost circle?
Is really every framework equal or are some frameworks more equal then others?

Let's have a look at some examples to get some idea ...


## What about the .NET framework?

Let's start with an obvious one: *[Athena](/Implementing-Clean-Architecture)* is built with [F#](http://www.fsharp.org)
on top of the .NET framework. Of course that does not mean that my whole project has to live in the frameworks circle ;-)

Uncle Bob says:

> There are some frameworks that you simply must marry. If you are using C++, for example, you will likely
> have to marry STL—it’s hard to avoid. If you are using Java, you will almost certainly have to marry the
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
and let it into the inner circles it means we will never get rid of it again and have to live with it until the projects
life ends.

For *Athena* I have decided to marry only the .NET framework, F# and the FSharp.Core library.


## What about the UI?

UI frameworks dictate quite some decisions to the app - maybe even patterns like MVVM or MVC we have to take.
UI frameworks we definitively do not want to marry because
- UI technologies change frequently
- testing through UI is challenging
- want to support different UIs

we discussed already in previous post: we want to keep it away from business logic. we definitively do not want
it in the usecase/entities layers and we want to avoid mixing it with controllers/presenters as migration to 
new UI technologies is painful ... u know how often UI framework changes!

same is true for: WPF. Swing. Ruby on rails. AngularJS and all these nice UI frameworks.
try to keep ans independent as possible!


but then the UI stuff has to be in the frameworks circle, right?

we could do it like with asp.net post: minimal stuff in frameworks layer
which calls adapters layer


## What about Dependency Injection frameworks?

but we want to use a DI container in a modern software project to wire up all those classes we carefully crafted
following SRP, right? We do not do such things by hand in 2019, right?
sure ... but a DI framework we want to marry? 

we do not want framework specific annotations everywhere in our business lines code. we want to be able to 
replace DI container as easily as possible.

Uncle Bob says:
> It is in this Main component that dependencies should be injected by a Dependency Injection framework. Once they are injected into Main, 
> Main should distribute those dependencies normally, without using the framework.

==> another post

but what about DI frameworks without any annotation? like Asp.Net CORE? 
then we still use it in MAIN component only and probably its impl is simpler as we dont have to pass many deps manually.
but still details belong to MAIN only.



## What about data access libraries?

so the database itself is a detail living in the outermost circle. but what about the repository accessing it?

"
Similarly, data is converted, in this layer [interface adapters layer], from the form most convenient for entities
and use cases, to the form most convenient for whatever persistence framework is being used (i.e., the database). 
No code inward of this circle should know anything at all about the database. If the database is a SQL database, 
then all SQL should be restricted to this layer and in particular to the parts of this layer that have to do with
the database. Also in this layer is any other adapter necessary to convert data from some external form, such as
an external service, to the internal form used by the use cases and entities.
"

but in order to access the DB i usually use "libraries" like

- System.Data.Sqlite (local caching for burn down)
- Newtonsoft.Json (local stored data)
- Microsoft.TeamFoundation.WorkitemTracking.Client (to access work items from TFS)
 
Are these frameworks which have to be banned to the outermost circle?

(IMAGE from fw vs lib)

## Again: What is a framework and what is a library?

From my perspective the difference between a framework and a library is "who is controlling whom?" 
A framework requires you to implement plugins. u write a controller for asp.net and by that plugin ur logic
into the asp.net framework. my code implments SPIs.

a library like newtonsoft.json just provides APIs. It is passive. it does not control anything. it is the 
application which is controlling when and how to call these APIs.

With this definition all the three examples above are just "libraries" so i would NOT put them into 
outermost circle but use those in the repository implementations.

BUT: always take care that types of the third party libraries become part of any of my public APIs.
i want to keep it truly encapsulated so that i can easily replace one implementation with another one if needed.


Alternative: define interface in adapters layer and create a minimal
wrapper implementation in frameworks layer. by that actual repository
implementation can be in adapters layer without dependencies to any 
"third partly" libraries or other "details"

==> which alternative to choose?

personally i dont see much benefit in this additional interface 
in adapers layer and adapter in frameworks layer AS LONG AS i can 
full encapsulate the detail in the adapters layer impl itself.

do i "marry the framework" if i use a library in my repository implementation?
==> no (as long as it is truely encapsulated (interface is free from any third party types)

(SHOW CODE?)

## Libraries in use case interactors?

Uncle bob would probably shout: "NO - NEVER" ...

in Athena i use a third party "numerics library": https://numerics.mathdotnet.com/ for linear interpolation. i use it 
to calc the ramp up. is this a violation to the dependency rule?

(SHOW CODE)

surely i could make a "service" out of it. i could create an interface in the usecase layer and
put the imple in the frameworks layer ... but why should i do it? again "do i marry the framework"? is it controlling me?
no. i am controlling it and if i ever want to replace it is very easy

I have encapsulated it --> ok
i am in control


## Conclusion

i finally mailed Uncle bob and he responded
"
"

i tend to be more pragmatic:
- carefully decide which framework to marry. tend to marry as less as possible
- keep true frameworks (which control your application) in the outermost circle
- use libraries (APIs u control) in adapters and even interactors but  never let any third party types into 
  "out" of ur encapsultation: use it in gateways and interactors but never make it part of ur public API.



## How do others think about usage of frameworks and libraries?

during my research ....

- https://stackoverflow.com/questions/4909301
- http://stackoverflow.com/q/48589192	
- https://stackoverflow.com/questions/50017576/how-to-separate-business-logic-from-rx
- https://stackoverflow.com/questions/54986932/using-bitmap-in-a-java-library


{% include series.html %}
