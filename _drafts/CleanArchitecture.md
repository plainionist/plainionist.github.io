
Next Steps:
- structure remaining content
- reference stackoverflow questions
- write next articles
  - update stackoverflow questions with links once blogged









==========================================================================================================================================


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


==========================================================================================================================================

---
layout: post
title: Implementing Clean Architecture - About the database and other details
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---


(IMAGE)

so now that we have clarified (LINK) what difference between framework and library is and which principles we want 
to follow, lets discuss now how we want to integrate with libraries and frameworks

<!--more-->

one key aspect here is that the interface belongs to the use case interactor. 

uncle bob says:
"
Between the use case interactors and the database are the database gateways. These gateways are polymorphic
interfaces that contain methods for every create, read, update, or delete operation that can be performed by
the application on the database. For example, if the application needs to know the last names of all the 
users who logged in yesterday, then the UserGateway interface will have a method named 
getLastNamesOfUsersWhoLoggedInAfter that takes a Date as its argument and returns a list of last names.
"
(summarize with own words)

## Corner stones

- that reminds me of unit of work
- still pure data access logic only! and data conversion from "sql rows" to "entities"!
- we probably have some "filtering" logic" as well
  (also filtering can be done close to DB more efficiently)
- but we do not want to have any complex logic which goes beyond filtering and converting rows to entities
- explicitly NOT one repository per entity (http://stackoverflow.com/q/52292717)

- i followed this approach in athena and first blindly created a new interface for each interactor. then
  i refactored the code and consolidated a few repositories

let me show u how this looks like

## Accessing the TFS (database)

- show that we have multiple repositories (UML?)
- show that the interfaces are defined by the use cases
- show impl of one repository (code really relevant or just UML?)
- show shared code to query TFS work items always in the right way
- show how i handle the parsing (repository calls interactor directly - which i try to avoid but in this case it is pragmatic again)

repository returns entities
and we do not want to let any DB specific types out

http://stackoverflow.com/q/47903739 (which objects to return)


### What about ORM?

i dont need ORM as MS has API ... which is kind of ORM actually

we do not want any ORM "framework" code in our most inner cirlces.

we want to bann it to outermost circle or truely encapsulate in adapter.

in athena i use s.th. similar: F# type providers. (Excel) - luckily i can truely encapsulate it.

EF allows "code first" which does not require any dependencies from entities to EF.
we can use it as implmenation detail of the repository.

### How do i handle transactions?

luckily i dont have transactions but ...

how do i do it across multiple interactors?

- transactions are necessary because we store state
- what if we only store events (+snapshots)
- CRUD goes to CR
- its like source code control action

recommendation: keep it as impl detail. follow unit of work and create a repository which does the whole transaction 
at once. remember: we do not want to have one repository per entity.
https://www.martinfowler.com/eaaCatalog/unitOfWork.html

## Communicating with external systems

- i have some excel file on sharepoint
- i handle it with repository pattern (download + parsing)
- again interface defined by use cases

## Defering performance optimizations - The BurnDownRepository

for the burndown i have to query TFS history for each day in the past 
that would create quite some load on TFS
so added a local cache DB (sqlite)
later i even cached in RAM to make it faster

- decorator pattern

==> focus on getting interactors implemented (easy with mocks in UT) and defer decision about persistency later 

## How to integrate a framework which requires passing tokens around?

how do i do it with TeamProjectCOllection class?

quite common pattern for authentication: u need to keep track of a "session" or "token" across the interactors and
repository accesses

recommendation like here
- http://stackoverflow.com/q/54695384
- https://softwareengineering.stackexchange.com/questions/366586/clean-architecture-google-facebook-login-and-data-layer
- http://stackoverflow.com/q/47278858
 

## How do others think about data access?

during my research ....

- http://stackoverflow.com/q/47446180
- https://stackoverflow.com/questions/47903739/android-app-clean-architecture-should-data-layer-have-its-own-model-classes
- https://stackoverflow.com/questions/51912841/golang-transactional-api-design
- http://stackoverflow.com/q/50871171


==========================================================================================================================================

---
layout: post
title: Implementing Clean Architecture - Critical Business Rules
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---


(IMAGE - entities circle)

After circling around the center of the clean architecture image for a long time we finally reached the center ;-)

now lets look what is in the "entities" circle ...

<!--more-->

First lets remember what Uncle Bob sees in that circle:

"
We shall call these rules Critical Business Rules, because they are critical to the business itself, and would exist 
even if there were no system to automate them. Critical Business Rules usually require some data to work with. 
For example, our loan requires a loan balance, an interest rate, and a payment schedule. We shall call this data 
Critical Business Data. This is the data that would exist even if the system were not automated. The critical rules 
and critical data are inextricably bound, so they are a good candidate for an object. We’ll call this kind of object an Entity.

The Entity is pure business and nothing else.
"

that means: entities are NOT just dump data objects. there is logic!

## But i dont have an "enterprise"

Entities encapsulate enterprise-wide Critical Business Rules. An entity can be an object with methods, or it can be a set 
of data structures and functions. It doesn’t matter so long as the entities can be used by many different applications in 
the enterprise. If you don’t have an enterprise and are writing just a single application, then these entities are the 
business objects of the application. They encapsulate the most general and high-level rules. They are the least likely 
to change when something external changes. For example, you would not expect these objects to be affected by a change 
to page navigation or security. No operational change to any particular application should affect the entity layer.

## Shouldnt i have started the series with this post?

if domain is clear and we start a green field project probably we should have started with entities.

==> ddd happens here!
==> u can also have ddd in application specific business objects (interactors)


if we would have done DDD then we would have started probably with this circle.
why didnt i do?

i tend to start with fewer entities - the entities i am sure about.
i tend to start with less methods in the entities.
then i implement use cases - the more i implmenent the more i learn about the domain the more code/logic
goes into the entities. of course i could have done more domain analysis and modeling up front ... but i dont 
like it much. i like to work incrementally. i like to have the first usecases up and running fast. 
but i also like to deferre heavy weight decisions. and so i keep my logic "private to the interactors"
until i learned enough to realize that certain knowledge is actually "application independenty business rules"

## The obvious entities

lets start with the obvious ones:
- work items
- team
- milestones
- fruit
- ...

## ... Would exist even if there were no system to automate them

- calculation of availability
- ...


## Can entities access repositories?

"
The Entity is pure business and nothing else.
"

https://stackoverflow.com/questions/47896909/should-a-domain-entity-call-a-repository
https://stackoverflow.com/questions/50943510/should-entities-in-clean-architecture-know-of-persistence-mechanisms



==========================================================================================================================================

---
layout: post
title: Implementing Clean Architecture - The dirty MAIN
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---

(IMAGE of a cristal ball)

now that we have covered all layers - how do i wire up all the things? after all the abstractions and interfaces - the magic
has to happen somewhere

<!--more-->

## The "Clean" approach

- the main component is the place where all details are wired up

"
In every system, there is at least one component that creates, coordinates, and oversees the others. I call this component Main.

The Main component is the ultimate detail—the lowest-level policy. It is the initial entry point of the system. 
Nothing, other than the operating system, depends on it. Its job is to create all the Factories, Strategies, and
other global facilities, and then hand control over to the high-level abstract portions of the system. It is in
this Main component that dependencies should be injected by a Dependency Injection framework. Once they are
injected into Main, Main should distribute those dependencies normally, without using the framework. Think
of Main as the dirtiest of all the dirty components.

The point is that Main is a dirty low-level module in the outermost circle of the clean architecture. It loads everything 
up for the high level system, and then hands control over to it.

For example, you could have a Main plugin for Dev, another for Test, and yet another for Production. You could also have a
Main plugin for each country you deploy to, or each jurisdiction, or each customer.

Instead, you can use Spring to inject dependencies into your Main component. 
It’s OK for Main to know about Spring since Main is the dirtiest, lowest-level component in the architecture.
"

## The "pragmatic" approach

i tend to be pragmatic: as i am in F# and everything is just a function call i also just call the functions of interactors
everywhere.

i just created interfaces (actually records of functions ;-)) for the adapters
that means i can test entities and use case interactors easily and isolated.

testing controllers and presenters and any other kind of adapter is more difficult.
current strategy: as less logic as possible to avoid testing at all.

Asp.Net MVC does not support dependency injection out of the box.
so i just sticked to "Service locator pattern" ... which i personally actually dont like as this allows every code to access everything
and dependencies are not clearly visible.

(CODE of IoC)

## Next steps

I ll once migrate to Asp.Net Core and use dependency injection. then i ll register the interactors as well as "services" and "repositories"
on the container and let DI do the work.


==========================================================================================================================================

---
layout: post
title: Implementing Clean Architecture - Cheat Sheet
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---


now that we discussed all layers including the main module here is the cheat sheet
this is how I would start applying clean architecture to a project (existing or green field)
(short compact - single page)

- first put all logic in the use cases
- make the controllers and presenters dump data converters  
  - request to request model, response model to response
- no logic in the views at all
- main is about composition only
- repositories and services focus on accessing details only
- entities get these rules which do not change across use cases



==========================================================================================================================================

---
layout: post
title: Implementing Clean Architecture - Testing
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---


- i am lazy and i always struggle to find a good balance between DRY and testing
- "what is simple enough that writing a test would violate DRY"?
- with this strange idea in mind i currently try to put - as i also think i read it in uncle bobs post - all code in the usecases/entities and test it
- i want to have all gateways that dump that testing feels like violating DRY (the code already clearly documents itself and is too simple that a test adds value - it is obvious)


==> you can test without the web server
==> you can test without many mocks!

- testing interactors !!
- if want to test close to user test controller and presenter (which still does not require a web server)
- the presenter does everything possible to make the view so dump that we never need to test it.
  (humble object)

==> so no testing of html needed

- but what about all the smart JavaScript in a modern SPA?
- maybe u can use frameworks to keep also the JavaScript as simple as possible - i dont want to test it :)
- what about vue or react and binding?

==> in the end u probably just need a very simple integration test which is running thrugh UI to check
    that everything is wired up correctly. but NO deep logic testing through UI!

Where do I do acceptance testing in clean arch?
==> On US?
==> On controller / presenter?
==> interactor … then simple UT for presenter and controller sufficient (because only doing data convertion)


- bdd is a natural fit
  - Bdd only works for inner two circles (this is where the business is and we don’t want technical terms in bdd spec)
  - The rest is technical an can be best tested with unit tests …

"humble object pattern"
- abstract the difficult things ways
- we do not want to test the humble object
- the view is a humble object
- so the viewmodel contains all data ready for the view to just display - no logic in the view to be tested


"
Yes, that’s right: The tests are part of the system, and they participate in the architecture just like every other
part of the system does. In some ways, that participation is pretty normal. In other ways, it can be pretty unique.

In fact, you can think of the tests as the outermost circle in the architecture. Nothing within the system depends 
on the tests, and the tests always depend inward on the components of the system.

Tests that are not well integrated into the design of the system tend to be fragile, and they make the system rigid
and difficult to change. The issue, of course, is coupling. Tests that are strongly coupled to the system must change
along with the system.

Fragile tests often have the perverse effect of making the system rigid. When developers realize that simple changes
to the system can cause massive test failures, they may resist making those changes. For example, imagine the 
conversation between the development team and a marketing team that requests a simple change to the page navigation
structure that will cause 1000 tests to break. The solution is to design for testability. The first rule of software
design—whether for testability or for any other reason—is always the same: Don’t depend on volatile things. GUIs are
volatile. Test suites that operate the system through the GUI must be fragile. Therefore design the system, and the
tests, so that business rules can be tested without using the GUI.

The role of the testing API is to hide the structure of the application from the tests. This allows the production code
to be refactored and evolved in ways that don’t affect the tests. It also allows the tests to be refactored and evolved
in ways that don’t affect the production code. This separation of evolution is necessary because as time passes, the 
tests tend to become increasingly more concrete and specific. In contrast, the production code tends to become increasingly
more abstract and general. Strong structural coupling prevents—or at least impedes—this necessary evolution, and prevents
the production code from being as general, and flexible, as it could be.
"


==========================================================================================================================================

---
layout: post
title: Implementing Clean Architecture - Answering the unanswered ...
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---


## Is the presenter view specific?

i think so. if u have an html view the presenter can know about html. see print example from uncle bob.
or the other way round: if not the presenter is preparing html snippets it means that u have logic in the view
what we want to avoid - also because of testability.

## input validation?

- https://softwareengineering.stackexchange.com/questions/351419/clean-architecture-validation-in-domain-vs-data-persistence-layer
- http://stackoverflow.com/q/47860684


- where to do threading? in gateways (service adapters)

## pagination

https://stackoverflow.com/questions/48356193/clean-architecture-where-to-implement-pagination-logic

## Must every interactor return a response model?

see also: https://stackoverflow.com/questions/55780386/why-have-a-request-model

==> if we split interactors according to SRP we could ask whether certain interactors can return entities instead of 
dedicated response models?
i think this is fine as long as interaction with interactor does NOT cross a circle boundary!!
- is such a private interactor still an interactor? i dont know what uncle bob thinks about it but i would like to 
continue reusing this term but it brings nice implications i want to hold true also for "private" interactors.
of course i have to maintain discipline: as soon as i start using such private interactor from conroller i have to ensure that
i do not leak entities.

DOUBLE check with the book: this is actually about BOUNDARIES! 
i think i can pass entities from one internal interactor to another one.
once this internal interactor becomes public we should again check what is passing a boundary


Interactors define input DTOs (Data transfer objects) and output DTOs which are most convenient for the use case. 
in his book uncle bob writes that entities should not be passed to use cases or returned from use cases

Uncle Bob:
> We don't want to cheat and pass Entity objects or database rows. 
> We don't want the data structures to have any kind of dependency that violates the Dependency Rule.
>
> [...]
>
> Thus, when we pass data across a boundary, it is always in the form that is most convenient for the inner circle.

All methods we have defined on the interactors so far are simple functions which return results.
Therefore we dont need to define input or output ports as interfaces - we can have simple DTOs for input and output.

## must every interactor have a request model?

even if i just need to pass one parameter

from: "Clean Architecture"

"
Use cases expect input data, and they produce output data. However, a well-formed use case object should have no inkling about the way that data is communicated to the user, or to any other component. We certainly don’t want the code within the use case class to know about HTML or SQL! The use case class accepts simple request data structures for its input, and returns simple response data structures as its output. These data structures are not dependent on anything. They do not derive from standard framework interfaces such as HttpRequest and HttpResponse. They know nothing of the web, nor do they share any of the trappings of whatever user interface might be in place. This lack of dependencies is critical. If the request and response models are not independent, then the use cases that depend on them will be indirectly bound to whatever dependencies the models carry with them. You might be tempted to have these data structures contain references to Entity objects. You might think this makes sense because the Entities and the request/response models share so much data. Avoid this temptation! The purpose of these two objects is very different. Over time they will change for very different reasons, so tying them together in any way violates the Common Closure and Single Responsibility Principles. The result would be lots of tramp data, and lots of conditionals in your code.
"

==> should we update the "use cases" article with these details? or better backward and forward link?


## What to put in Entities in a micro services architecture?

- uncle bob: enterprise wide
- dont have that - just one app
- then central things
- but i want to keep MS as independent as usefull?
- so i decided to go bottom up: i handle each MS as separate app and put local entities
- if i find entities which are valuable to be shared across multiple microservices i put them in shared lib

## How to related DDD & Clean Architecture

- i first read clean architecture and then Erics DDD
- when reading DDD my brain immediately started comparing both books
- my conclusion
- entities and usecases map to entitis
- ddd says more about how to model entities and usecases
- ddd also talks about iterative and framework independence .

==========================================================================================================================================

---
layout: post
title: Implementing Clean Architecture - Retrospective
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---


Finally that we have discussed many conrete aspects on how to impl clean arch lets have one complete overview
how Athena finally looks like. where did we made compromises. where do we need imporve

## overview

draw a scetch of Athenas architecture
separate domain objects for backlog and burndown and governance

## One complete example

show how one scenario would be designed in clean-arch.
what is usecase, what is gateway, ...


show how the backend architecture has evolved with VueJS as FE
- dependencies are gone to MVC classes
- also update post "implementing-Clean-architecture-aspnet"
- also helper functions used by razor engine are gone
- also ScopedImprovement class is no longer used by view (no cheating - having proper string view models everywhere)

how would controller/presenter interaction change with async controller functions in Asp.Net Core?


==> draw a picture from Plainion.GraphViz


## Final Words

was it worth all the effort? what did i learn from that journey?

so now that i have to put so much additional effort into to appart from just make it work: why should i do so?

- is it the independency to frameworks?
- is it the focus on the usecases? (not driven by UI or DB - driven by the usecase of the domain)
- is it the "screaming" architecture?

- after having refactored quite some code from non clean arch to clean arch i can say: "it is unbelievable how the separation of
  interactor and presenter simplifies the design", "how magically entities occur - interactors have interactor specific requests and response. 
  everthing generic to these is entities!?"

- clean architecture supports greatly to defer decisions about UI, DB, cache, etc 
  we could just start with bdd tests as "driver" for use cases


PRO: defer decisions

may look like a lot of work but it pays of e.g. when testing (stable tests!) and replacing technology

adopting to changing requirements seems expensive but that is actually not the case. u have a nice robust safety net and all the DTOs and adpaters and data converters are pretty easy to change with less risk of breaking s.th. else



==========================================================================================================================================


# other blogs

try to fold this in here and there and write a summary page for further reading for everything which does not fit

- https://www.codingblocks.net/tag/clean-architecture/
- https://manuel.kiessling.net/2012/09/28/applying-the-clean-architecture-to-go-applications/

- http://five.agency/android-architecture-part-3-applying-clean-architecture-android/
- http://five.agency/android-architecture-part-4-applying-clean-architecture-on-android-hands-on/
- https://blog.sourced-bvba.be/article/2017/02/14/thoughts-on-clean-architecture/
- https://www.thedroidsonroids.com/blog/android/rosie-lets-dive-into-clean-architecture
- https://www.entropywins.wtf/blog/2016/11/24/implementing-the-clean-architecture/
- http://tech.edreamsodigeo.com/clean-architecture-android/
- [Better Software Design with Clean Architecture](https://fullstackmark.com/post/11/better-software-design-with-clean-architecture).
- [clean-architecture-example](https://github.com/mattia-battiston/clean-architecture-example) 
- [Clean Architecture is Screaming](http://tidyjava.com/clean-architecture-screaming/) 
- https://beberlei.de/2012/08/13/oop_business_applications_entity_boundary_interactor.html
- https://blog.sourced-bvba.be/article/2017/02/14/thoughts-on-clean-architecture/
- https://dzone.com/articles/clean-architecture-is-screaming

