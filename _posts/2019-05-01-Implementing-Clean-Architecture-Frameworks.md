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
[MVC](https://en.wikipedia.org/wiki/Model-view-controller) or [MVVM](https://en.wikipedia.org/wiki/Model-view-viewmodel) 
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


## What about Dependency Injection frameworks?

I think we all agree that we want to use a dependency injection (DI) framework in a modern software project to 
wire up all those classes we carefully crafted following [SOLID principles](https://en.wikipedia.org/wiki/SOLID), right?
We do not do such things by hand in 2019, right? Sure, but do we want to marry the DI framework? 

[Uncle Bob says](/Clean-Architecture):

> It is in this Main component that dependencies should be injected by a Dependency Injection framework.
> Once they are injected into Main, Main should distribute those dependencies normally, without using the framework.

What is the "Main component"? Let's leave this question to another post but we definitively do not
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

Luckily DI framework authors recently have accepted that we do not want to pollute our business logic with their 
annotations and have provided alternatives which work entirely without such annotations. Examples are 
[Asp.Net Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-2.2)
and [AutoFac](https://autofac.org/).


## What about Object-Relational mapping (ORM) frameworks?

As already stated, the database itself is a detail which belongs to the frameworks circle but what about the
code accessing it?
We want to use these cool ORM frameworks like [Entity Framework](https://docs.microsoft.com/en-us/ef/) or 
[Hibernate](http://hibernate.org/) for data access, right? I mean: who wants to write plain SQL
queries manually in 2019? Instead, we just put some annotations to our entities, derive our repositories
from some framework's base class and the magic happens!

We just got married - to the framework! Assuming that a non-trivial project would have more than one entity and 
more than one repository, moving to a different framework in some future becomes a real pain easily. Not talking about 
difficulties in testing the repositories and any other impact the ORM framework brings to our architecture ...


### How to implement a repository with ORM correctly?

Okay, so we want to keep dependencies to such ORM frameworks in the frameworks circle. How do I implement a 
repository then? Do I really have to fall back to plain SQL? Not necessarily. 

We can use the same trick as we used for the UI frameworks: we invert the dependency. We define interfaces and 
simple data objects in the adapters circle without any dependency to the ORM framework and add some code to the 
frameworks circle which implements these interfaces and works with these data objects.

Let's imagine all the work items we want to process in *Athena* would be stored in an SQL database and we would like
to use the [Entity Framework](https://docs.microsoft.com/en-us/ef/) to access those. A "Clean" design would look 
like this.

<img src="{{ site.url }}/assets/clean-architecture/Frameworks.DataAccess-Clean.png" class="dynimg" title="Separating repository implementation from the ORM framework" alt="ITfsDataMapper interface and TfsWorkItem data object are introduced to separate the repository implementation TfsWorkItemRepository from the ORM framework."/>

On the one hand we have the interactor which is dealing with our domain entities only and which does not care about 
persistence at all. It gets the entities through ```IWorkItemRepository``` which is implemented by the 
```TfsWorkItemRepository``` in the adapters layer.

On the other hand we have the SQL database which stores all the information in a nice, normalized way, optimized
for IO performance and minimal storage space. We use the [Entity Framework](https://docs.microsoft.com/en-us/ef/)
to map the rows of the database tables into objects like ```SqlWorkItem``` and ```SqlAreaPath``` which can use
annotations and any other feature provided by the [Entity Framework](https://docs.microsoft.com/en-us/ef/) take 
care of correct serialization.

In order to maintain the Dependency Rule we define the ```ITfsDataMapper``` interface in the adapters layer which 
will be used by the ```TfsWorkItemRepository``` and implemented by a ```SqlDataMapper``` class in the frameworks
layer. The ```SqlDataMapper``` contains minimal logic to convert the "Entity Framework aware" objects into 
"ORM framework neutral" objects (```TfsWorkItem```). The ```TfsWorkItemRepository``` finally consumes the
```TfsWorkItem``` objects and takes care of proper creation of our domain entities.

With this approach the code depending on the ORM framework contains minimal logic which requires less (if any at all)
expensive testing and makes migration to other data sources or ORM frameworks later on quite easy. The actual 
implementation of the repository remains independent of the ORM framework and so can be tested easily 
and is not effected when the data source needs to be changed.

The major drawback of this approach is that it is more complex and more expensive as additional interfaces and
data objects are involved.

Is there no way to make it simpler? We could think of skipping the concept of ```ITfsDataMapper```, put the 
```TfsWorkItemRepository``` into the frameworks layer and let it create our domain entities from the ORM 
framework data objects directly.

<img src="{{ site.url }}/assets/clean-architecture/Frameworks.DataAccess-NoDataMapper.png" class="dynimg" title="Repository implementation in frameworks layer without DataMapper" alt="In order to avoid the complexity and cost of the ITfsDataMapper, the repository implementation gets moved into the frameworks layer directly"/>

This way the Dependency Rule remains intact at lower cost. Of course we loose the benefits of having a repository
implementation which is independent from the ORM framework which are mainly portability and testability.

The key question here seems to be: How much ORM framework independent code does exist in the repository
implementation?


### What about an ORM framework without "side effects"?

But what about a persistence framework which does not require any annotations or any other dependency
of our inner layers to it? What about e.g.
[Entity Framework Fluent API](https://docs.microsoft.com/en-us/ef/ef6/modeling/code-first/fluent/types-and-properties) or
[Hibernate persistence mapping file](http://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#bootstrap-jpa-xml-files)?

With such approaches we could avoid additional interfaces and data objects and map our entities directly from and to the 
database and still keep the inner circles free from dependencies to the ORM framework. 

<img src="{{ site.url }}/assets/clean-architecture/Frameworks.DataAccess-NoSideEffects.png" class="dynimg" title="Using side effects free ORM mapper" alt="Side effects free ORM mapper would allow us to use domain entities directly for mapping to database tables."/>

The ```TfsWorkItemRepository``` would again live in the frameworks layer as it would access the ORM framework directly 
(in this example it would derive from Entity Framework's ```DbContext```). This would not be any issue as probably
there would be no ORM framework independent logic anyhow as the whole mapping is now done by the ORM framework directly.

This approach might look like a simple and clean alternative but consider that

- Such frameworks often still require you to follow certain conventions, e.g. public setters for all properties or
  existence of parameterless constructors. That might still feel like some kind of marriage ...
- "External" mapping might become a problem with refactoring if those are not well supported by your IDE

It seems that again not all frameworks are equal. In *Athena* I use the following frameworks to access data sources:

- ```Microsoft.TeamFoundation.WorkitemTracking.Client``` to access work items from TFS
- F# type providers to read team configurations from a Microsoft Excel sheet
- ```System.Data.Sqlite``` to access a SQLite database acting as a local cache for burn down data

These frameworks just provide simple APIs to read and write data without influencing the architecture of the
application. Do I have to exile these "frameworks" to the outermost circle as well?

Let's ask again ...


## What is a framework? What is a library?

<img src="{{ site.url }}/assets/clean-architecture/Frameworks.Vs.Libraries.png" class="dynimg" title="Framework vs Library" alt="How does the application interact with a framework and how does it interact with a library?"/>

From my perspective there is a major difference between a "framework" and a "library" which is the direction of control.

A framework requires the application to provide its business logic as plug-ins by implementing 
[SPIs](https://en.wikipedia.org/wiki/Service_provider_interface). We need to derive from an Asp.Net MVC
controller in order to get our business logic executed on a HTTP request. The framework is directing the control flow
within the application. The application can "passively participate" by providing plug-ins. 

A library just provides [APIs](https://en.wikipedia.org/wiki/Application_programming_interface). The application needs
to call these APIs "actively" in order to make use of it. The control flow is clearly directed by the application.

Based on this definition, the three "frameworks" I use in *Athena* for data access are just "libraries".
I do not have to "marry" them in order to benefit from their usage. Instead I could just encapsulate them in a class
or a component, hide them behind an interface and never let any library type pass this border. This way the usage of
the library has no impact on any decision outside that component or class. If I want to replace the library in future
with another one, I simply replace that implementation of the interface with another one.

Could that be a pragmatic way of implementing gateways and repositories in the adapters layer?

<img src="{{ site.url }}/assets/clean-architecture/Frameworks.DataAccess-Libraries.png" class="dynimg" title="Using data access libraries for repository implementation" alt="TfsWorkItemRepository accesses TFS through the TFS data access library without any addition interfaces or data objects"/>

But wait! Isn't that still a violation to the Dependency Rule? Strictly speaking: yes it is. The arrow from 
```TfsWorkItemRepository``` to ```Microsoft.TeamFoundation.WorkitemTracking.Client``` proves it.
In order to fix this violation we would have to chose one of the approaches discussed above.

However, I see some benefits in keeping such implementations in the interface adapters circle which are

- Maintaining a clear border between the "harmful frameworks", which I definitively do not want to marry, from the 
  comparable "harmless libraries"
- Possibly easier reuse of the "encapsulated library" among other gateways in the interface adapters layer 
  (thinking of facades, [Unit of Work](https://www.martinfowler.com/eaaCatalog/unitOfWork.html) and similar patterns)
- Keeping "gateways" (repository) in the interface adapters circle just feels more correct than putting it in 
  the frameworks circle

These arguments might not be strong enough to justify a violation of the Dependency Rule. However, for *Athena* I 
decided to follow this separation between frameworks and libraries and allow usage of third party libraries in the
adapters layer (if those are completely hidden behind some interface). 
Let's see how long this decision will last ;-)


## Libraries in use case interactors?

Let me go even one step further ...

In *Athena* I need a [math library](https://numerics.mathdotnet.com/) to do some linear interpolation for some 
calculation done by a use case interactor. In order to make use of that library in a clean way I would have to define 
some interfaces and maybe even data objects in the use case circle and put the implementation of that interface to 
the frameworks layer to restrict the usage of that library to that layer (as discussed above). 
Maybe I even should have some "adapters" in between?

However, in this particular case, I think this adds unnecessary effort and complexity. Instead, I decided to just use
this library from within the interactor directly by completely encapsulate its usage in a single function.

```fsharp
let GetAvailabilityInRange (getHeads:GetHeads) (fromDate:DateTime, toDate:DateTime) team =
    // xAxis: all months of project lifetime
    // yAxis: available team's head count for each month

    // --> "Interpolate" comes from that math library <--
    let interpolation = Interpolate.Linear (xAxis, yAxis)
    
    // interpolate head count from "fromDate" to "toDate" and calculate total sum
    let totalHeads = 
        [ fromDate .. toDate ]
        |> Seq.map interpolation.Interpolate
        |> Seq.sum
     
    // convert head count into "availability"
    totalHeads * team.AvailabilityFactor
```

## Conclusion

Not all frameworks are equal - some are more equal than others.

Some frameworks we have to marry while others we never want to at any cost.

Keeping frameworks "at arm's lengths" comes with additional cost but also with benefits.

Encapsulating usage of a library in an adapter might be a pragmatic way to reduce the cost without risking an 
unwanted marriage.

Finally, it all comes down to finding the "right" borders in a particular architecture and 
adjusting those if circumstances change.

Your turn ;-)

**Update 2022-11-08**

I have created a YouTube video discussing in detail why most intuitive repository implementations are 
not fully compliant with the rules of the Clean Architecture and how this can be fixed:

<iframe width="560" height="315" src="https://www.youtube.com/embed/pfhDO_hZixw" 
  title="YouTube video player" frameborder="0" 
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen>
</iframe>



{% include series.html %}
