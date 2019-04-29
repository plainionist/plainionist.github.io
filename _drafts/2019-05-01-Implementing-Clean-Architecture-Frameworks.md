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
> architecture. Don't let it into the inner circles.

Hhmm ... does this mean that every third party code is a framework which has to exiled to the outermost circle?
Is really every framework equal or are some frameworks more equal then others?

Let's have a look at some examples to get some idea ...


## What about the .NET framework?

Let's start with an obvious one: *[Athena](/Implementing-Clean-Architecture)* is built with [F#](http://www.fsharp.org)
on top of the .NET framework. Of course that does not mean that my whole project has to live in the frameworks circle ;-)

Uncle Bob says:

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
and let it into the inner circles it means we will never get rid of it again and have to live with it until the projects
life ends.

For *Athena* I have decided to marry only the .NET framework, F# and the FSharp.Core library.


## What about the UI?

As we have seen [earlier](/Implementing-Clean-Architecture-AspNet/) with the example of Asp.Net MVC,
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

Clearly we want to *use* UI frameworks but we definitively do not want to *marry* them!

So how do we keep these frameworks "at arm's length"?

In most UI frameworks the View completely depends on the framework, e.g.

- The Asp.Net MVC ".cshtml" Razor template cannot live outside the Asp.Net MVC framework
- The WPF XAML window or control completely consists of WPF framework elements
- the VueJS component completely depends to the VueJS framework to call its hooks

So we have to keep the View in the frameworks circle completely.

The Controller and/or ViewModel on the other hand is usually less coupled to the framework. So we can follow
the approach of the previous post and extract those code parts which depend on the UI framework, keep them in 
the frameworks circle and let it call the framework independent parts which will live in the interface adapters
circle.

<img src="{{ site.url }}/assets/clean-architecture/AspNet.Controllers.Clean.png" class="dynimg" title="Separating framework dependent and framework independent code of the controller" alt="The BacklogController does not derive from Asp.Net Controller any longer and contains most of the data conversion logic. BacklogAspNetController derives from Asp.Net Controller, converts data between Asp.Net and application and calls the BacklogController. Asp.Net dependencies are factored out of ReleaseBacklogViewModel into ReleaseBacklogAspNetViewModel"/>

In case we need to communicate from the adapters circle back to the frameworks circle we can simply provide
an interface in the adapters layer which will be implemented by the frameworks circle code and so we maintain 
the Dependency Rule. All arrows crossing the border are pointing "inwards".

<img src="{{ site.url }}/assets/clean-architecture/Adapter.Framework.Inversion.png" class="dynimg" title="Separating framework dependent and framework independent code of the controller" alt="The BacklogController does not derive from Asp.Net Controller any longer and contains most of the data conversion logic. BacklogAspNetController derives from Asp.Net Controller, converts data between Asp.Net and application and calls the BacklogController. Asp.Net dependencies are factored out of ReleaseBacklogViewModel into ReleaseBacklogAspNetViewModel"/>

Is it worth the effort? It depends on how much of your Controller's and ViewModel's code you want to keep independent
from the specific UI framework. If there isn't much such code you could also decide to move the Controller and ViewModel
to the frameworks circle completely. However I would not recommend to let the UI framework into your interface adapters
circle. Maintain a strict border remain the framework and your adapters otherwise the framework will "take over" this
circle as well over time!


## What about Dependency Injection frameworks?

I think everyone agrees that we want to use a dependency injection (DI) framework in a modern software project to 
wire up all those classes we carefully crafted following [SOLID principles](https://en.wikipedia.org/wiki/SOLID), right?
We do not do such things by hand in 2019, right? Sure, but do we want to marry the DI framework? 

Uncle Bob says:
> It is in this Main component that dependencies should be injected by a Dependency Injection framework.
> Once they are injected into Main, Main should distribute those dependencies normally, without using the framework.

What is the "Main component"? I will leave the answer to this question to another post but we definitively do not
want any DI framework specific code anywhere in the inner circles, not in the entities, not in the interactors
and even not in the adapters!

We do not want to use DI framework specific annotations in the inner circles like 
with [MEF](https://docs.microsoft.com/en-us/dotnet/framework/mef/):

```fsharp
[<Export(typeof<IWorkItemRepository>)>]
type WorkItemRepository() = 
	interface IWorkItemRepository
	...

type BackLogController() =
	[Import]
	member val WorkItemRepository = null with get, set
```

And we do not want to use DI framework specific [ServiceLocator](https://en.wikipedia.org/wiki/Service_locator_pattern)
in the inner circles like this:

```fsharp
type BackLogController(serviceLocator:IServiceLocator) =
	let myRepository = serviceLocator.Resolve<IWorkItemRepository>()
```

Luckily DI framework authors recently have accepted that we do not pollute our business logic with their annotations
and have provided alternatives which work entirely without such annotations. Examples are 
[Asp.Net Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-2.2)
and [AutoFac](https://autofac.org/).


## What about Object-Relational mapping (ORM) frameworks?

Clearly the database itself belongs to the frameworks circle but what about the repository (adapter) accessing it?
We want to use these cool ORM frameworks like [Entity Framework](https://docs.microsoft.com/en-us/ef/) or 
[Hibernate](http://hibernate.org/) for repository implementation, right? I mean: who wants to write plain SQL
queries manually in 2019? Instead, we just put some annotations to our entities, derive our repository implementation
from some framework's base class and the magic happens!

We just got married - to the framework! Assuming that we have more than one entity and more than one repository in
non-trivial project moving to a different framework in some future becomes a real pain easily. Not talking about 
difficulties in testing the repositories ...


### How to implement a repository with ORM?

Okay, so we want to keep dependencies to such ORM frameworks in the frameworks circle. How do I implement a 
repository then? Do I really have to fall back to SQL? Not necessarily. We can use the same trick as we used it for 
the UI frameworks: we invert the dependency. We define interfaces and simple data objects in the adapters circle without 
dependency to any ORM framework and add some code to the frameworks circle which implements these interfaces and 
works with these DTOs.

(IMAGE - example UML about TFS work items with "simple fields")

The drawback of this approach is that it involves quite some effort and requires a bunch of new types to be created:

- We have our domain model entities.
- we need some data objects defined in frameworks circle which are used for mapping by the ORM framework.
- we need some data objects defined in adapters circle to pass data from frameworks circle to adapters circle without
  breaking the Dependency Rule.

Can't we make it simpler? We could think of skipping the last point but that would require the code in the frameworks circle 
to work with our domain entities which would practically mean to put the whole repository implementation in the frameworks circle.
That might be a good pragmatic alternative if most of your repository implementation depends on the ORM framework anyhow and 
there is not much code left to be separated out into the adapters layer.

On the other hand, if you can keep quite some code of your repository implementations independent from ORM framework dependencies
it might be worth the effort as it pays off in terms of testability, portability to another ORM framework and even
complexity.


### What about ORM without "side effects"?

But what about a persistence framework which does not require any annotations or requires any other dependency of our
inner layers to it? What about things like
[Entity Framework Fluent API](https://docs.microsoft.com/en-us/ef/ef6/modeling/code-first/fluent/types-and-properties) or
[Hibernate persistence mapping file](http://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#bootstrap-jpa-xml-files)?

This might be a good alternative but consider that

- Such frameworks often still require you to follow certain conventions, e.g. public setters for all properties or
  existence of parameterless constructors. This might be still some kind of marriage ...
- "External" mapping might become a problem with refactoring if those are not well supported by your IDE

It seems that again not all frameworks are equal. In *Athena* I use the following libraries to access data sources:

- Microsoft.TeamFoundation.WorkitemTracking.Client to access work items from TFS
- F# type providers to read team configurations from an Microsoft Excel sheet
- System.Data.Sqlite as a local cache of burn down data

These libraries just provide simple APIs to read and write data without any "side effect" on any design decision outside
the actual repository implementation. Are these libraries "frameworks" which have to be exiled to the outermost circle?

Lets ask again ...


## What is a framework and what is a library?

(IMAGE from fw vs lib)

From my perspective the difference between a framework and a library is "who is controlling whom?" 
A framework requires you to implement plug-ins. u write a controller for asp.net and by that plug-in your logic
into the asp.net framework. my code implements SPIs.

a library like newtonsoft.json just provides APIs. It is passive. it does not control anything. it is the 
application which is controlling when and how to call these APIs.

With this definition all the three examples above are just "libraries" so i would NOT put them into 
outermost circle but use those in the repository implementations.

do i "marry the framework" if i use a library in my repository implementation?
==> no (as long as it is truly encapsulated and (interface is free from any third party types) there is almost no impact
on the other circles

(SHOW CODE?)

==> but hey why not just put it into frameworks layer and strictly stick to the dependency rule?
(as discussed before)
- pro: logically gateways/repositories belong to the adapters layer (rather weak argument)
- pro: keep the code separated from "true" frameworks 
- con: project hosting such "hybrid" adapters need dependencies to such libraries so there is no strong barrier anymore to
  other - otherwise dependency free - code
- pro: possible to be called by other classes in the adapters layer without further indirections


## Libraries in use case interactors?

Uncle bob would probably shout: "NEVER !!" ... and i would tend to agree in most cases but in Athena there was one 
use case where i needed a [numerics library}( https://numerics.mathdotnet.com/ ) for some linear interpolation.

of course i could have decided to define an interface and put the implementation in the frameworks layer - as usual - but i
decided that this would be "overkill" and went for a more pragmatic solution: i just used the library but completely 
encapsulated it in a function - no third party dependency outside this function.

(SHOW CODE)

in that case i also dont see much issue with testing as i wanted this code to be "part of" my tests anyhow.

Probably not 100% "Clean" ... but "pragmatic" ;-)

## Conclusion

I finally mailed Uncle Bob about my confusion and he responded that

> The trick to this is to add code to the outer circle that implements interfaces defined in the adapters layer.

However I tend to be more pragmatic if possible. My recommendation would be:

- carefully decide which framework to marry. only marry if you really want to keep it until lifetime of your project.
- keep true frameworks (which control your application) in the outermost circle
- use libraries (APIs u control) in adapters and even interactors but  never let any third party types 
  "out" of your encapsulation (no third party type in any public api)


## How do others think about usage of frameworks and libraries?

During my research for this post I found some other interesting discussions about usage of frameworks and libraries which 
I would like to share:

- [How to separate business logic from Rx](https://stackoverflow.com/questions/50017576/how-to-separate-business-logic-from-rx)
- [Using bitmap in a java library](https://stackoverflow.com/questions/54986932/using-bitmap-in-a-java-library)


{% include series.html %}
