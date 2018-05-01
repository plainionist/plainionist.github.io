---
layout: post
title: Implementing Clean Architecture - Frameworks vs. Libraries
description: |
  the dependency rule forbids dependencies from frameworks to adapters
  but what is a framework which is restricted by that rule? how to implement a repository without 
  referencing third party code?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.Presenters.AspNet.png" class="dynimg" title="Asp.Net in the context of Clean Architecture." alt="How do Asp.Net Controllers fit into the context of Clean Architecture? Do they belong to the interface adapter layer?"/>

have been some while since last post as i was busy with refactoring *[Athena](/Implementing-Clean-Architecture)* towards
[recent insights](/Implementing-Clean-Architecture-AspNet/)

what is a framework? what is a library? 
how to implement a repository? with jsondatacontractserializer? is that ok? what about newtownsoft.json? is that third party?

In this post ...

<!--more-->


check in the book: is "gateway" the reserved word for adapters to db and external services only?

how do we handle frameworks in the sense of libraries? everywhere i am allowed to use .net fw.
but what is the diff between .net datacontract serializer and newtonsoft.json?
should "framework" be seen only in that sense that app is extending it, is pluged in?
are libraries in general OK if they are encapsulated?
where is the border

"
Frameworks and Drivers.

The outermost layer is generally composed of frameworks and tools such as the Database, the Web Framework, etc. 
Generally you don’t write much code in this layer other than glue code that communicates to the next circle inwards.

This layer is where all the details go. The Web is a detail. The database is a detail. We keep these things on the 
outside where they can do little harm.
"


"
from Uncle Bob’s book I understood that dependency injection frameworks should be handled as any other framework?—?in the framework layer only:

from the book

'
It is in this Main component that dependencies should be injected by a Dependency Injection framework. Once they are injected into Main, 
Main should distribute those dependencies normally, without using the framework.
'

(i also found other places in the book where he points similar direction: no DI framework outside frameworks layer)

i would tend to agree?—?at least if the framework uses some kind of annotations in the code?—?because then once introduced and it got 
distributed all over the code base it is hard to get rid of it again.
"

https://medium.com/@stephanhoekstra/thanks-thats-cool-i-ll-make-sure-to-read-your-stuff-258dc57adbbe


maybe it is also a question of risk? how risky is a bad design with allowing certain third party references on a project?


i would also use 3rd party libs in usecases - encapsulated - if it is about business rules. examples numerics library for calculation.
never use third party types in public apis - also not in gateways

should be ok to have assembly which contains shared "framework" code which can then be used by multiple repository implementations
as these encapsulate the lib again


i would interpret it this way:
the DB itself in in framework and driver layer. with ISqlCommand and SQL we do not depend on the conrete DB - the concrete DB is injected into
the repository (ISqlCommand vs SqliteCommand)
And what if u need specific commands from SqliteCommand? - then we could consider creating an interface in adapters layer implemented by an 
adapter in framework layer - is it worth another level of indirection? in small projects probably not. having the repository depend on 
sqlitecommand is not great but ok-ish. in bigger projects where u could have tens of units of work the dependency to sqlite starts spreading
across the code base - still only in adapter layer but easily in multiple assemblies - in that case i would consider one more abstraction in the 
single assembly which plugs in sqlite into the app
(refering to: single DB. if u go for microserivces and every MS goes with own logical db but again first impl is choosen sqlite again then 
it is again a "local" decision which is then ok to use directly in repo - if we let details grow into our code base - even though only gateway impls 
then we are in trouble)
how do i handle it for athena? i have central DB "TFS" - i depend on TFS apis in multiple repositories.
for me this is fine as long as not public API uses tfs data types. it is very unlikely that the athena will ever
be used for any other "backlog database". i also use other data sources like sqlite for burndown data cache. these
data sources are local decisions and only one repo each exists ... i am also fine using libraries to access these
data sources (fsharp.data, sqlite pacakge) in the gateway directly



04.03.2018	Does R count as an android dependency?	https://stackoverflow.com/questions/4909301	add link once posted about gateway to framework dependencies

02.02.2018 18:56	Dependency from Gateway to Framework in Clean Architecture	http://stackoverflow.com/q/48589192	add link to post once blogged





{% include series.html %}
