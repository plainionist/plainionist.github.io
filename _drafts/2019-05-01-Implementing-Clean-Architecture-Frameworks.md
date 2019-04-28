---
layout: post
title: Implementing Clean Architecture - Frameworks vs. Libraries
description: |
  In Clean Architecture the Dependency Rule forbids usage of frameworks outside of the outermost circle.
  Does this mean that the usage of any third partly library in the implementation of a gateway or repository
  is a violation to the Dependency Rule?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.Frameworks.png" class="dynimg" title="Frameworks and libraries in the context of Clean Architecture." alt="In Clean Architecture the usage of frameworks is restricted to the outermost circle. But what is a framework? Is every third party library a framework? How to implement gateways without using third party libraries?"/>

it has been a while ago since i posted last time (LINK) to the "impl clean arch" series but i
wasnt lazy the whole year ;-) in fact i was - apart from my day job - working on bringing 
Athena (LINK) closer a true "clean architecture" ... and there was one thing which puzzled me
for a while when implementing "repositories".

How do I implement the repository (interface adapter layer) which accesses the TFS (framework layer) 
using APIs provided by Microsoft for that purpose?

Isn't that a violation to the Dependency Rule?

In this post I am trying to answer this question.

<!--more-->

What is a framework? 
According to Uncle bob:

"
The outermost layer is generally composed of frameworks and tools such as the Database, the Web Framework, etc. 
Generally you don’t write much code in this layer other than glue code that communicates to the next circle inwards.

[...]

This layer is where all the details go. The Web is a detail. The database is a detail. We keep these things on the 
outside where they can do little harm.

[...]

Don’t marry the framework! Oh, you can use the framework—just don’t couple to it. 
Keep it at arm’s length. Treat the framework as a detail that belongs in one of the outer circles of the 
architecture. Don’t let it into the inner circles.
"
(summarize with own words)

hhmm ...

does this mean that any third party library is a framework which has to banned to the outermost circle?

Let's go through some examples


## What about the .NET framework?

lets start with a simple one: Athena is build with F# on top of .Net "framework". obviously i dont want to move whole project
into frameworks layer ;-)

uncle bob says:
"
There are some frameworks that you simply must marry. If you are using C++, for example, you will likely
have to marry STL—it’s hard to avoid. If you are using Java, you will almost certainly have to marry the
standard library.
"

ok - so obviously we have to make some compromises - but we have to decide explicitly which ones to make.
there are other basic frameworks like boost and fsharp.data where we may want to decide to marry those as well.
but be careful


## What about the UI?

we discussed already in previous post: we want to keep it away from business logic. we definitively do not want
it in the usecase/entities layers and we want to avoid mixing it with controllers/presenters as migration to 
new UI technologies is painful ... u know how often UI framework changes!

same is true for: WPF. Swing. Ruby on rails. AngularJS and all these nice UI frameworks.
try to keep ans independent as possible!


## What about Dependency Injection frameworks?

"
It is in this Main component that dependencies should be injected by a Dependency Injection framework. Once they are injected into Main, 
Main should distribute those dependencies normally, without using the framework.
"

we do not want framework specific annotations everywhere in our business lines code. we want to be able to 
replace DI container as easily as possible.


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

From my perspective the difference between a framework and a library is "who is controlling whom?" 
A framework requires you to implement plugins. u write a controller for asp.net and by that plugin ur logic
into the asp.net framework. my code implments SPIs.

a library like newtonsoft.json just provides APIs. It is passive. it does not control anything. it is the 
application which is controlling when and how to call these APIs.

With this definition all the three examples above are just "libraries" so i would NOT put them into 
outermost circle but use those in the repository implementations.

BUT: always take care that types of the third party libraries become part of any of my public APIs.
i want to keep it truly encapsulated so that i can easily replace one implementation with another one if needed.


## Third party libraries in use case interactors?

in Athena i use a third party "numerics library": https://numerics.mathdotnet.com/ for linear interpolation. i use it 
to calc the ramp up. is this a violation to the dependency rule?

I have encapsulated it --> ok


## Conclusion

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
