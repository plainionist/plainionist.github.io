---
layout: post
title: Implementing Clean Architecture - About the database and other details
description: |
    In Clean Architecture we focus a lot on the inner circles like use cases circle and entities.
    But to make the application useful it has to interact with the outer world - but how?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---

(IMAGE)

Last time (LINK) we talked a lot about how to integrate "details" like frameworks into the clean architecture
in different very clean but also pragmatic ways.

No i want to become more practical again and show u i concretely did it for *Athena*.

<!--more-->

To remember the basic idea is to create an interface in an inner circle and an implementation in an outer circle.
whether that should be adapters layer or frameworks layer - pls refer to previous post.

One  key aspect here is - compared to traditional architecture - that the interface "belongs" to the more inner circle.
by that i dont mean only the locatio no the interface - i also mean its design.

the interface "needed" by a use case and implmented by an adapter will be primarily shaped by the interactor.
the interactor defines which methods it should have and which not. we do NOT want to have component driven interfaces
which are primarily shaped by the capabilities of its implmentation.

- we do not want dump CRUD interfaces
- we want "unit of work" interfaces

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

what different types of "adapters"?
- repository
- service
- ...

- that reminds me of unit of work
- still pure data access logic only! and data conversion from "sql rows" to "entities"!
- we probably have some "filtering" logic" as well
  (also filtering can be done close to DB more efficiently)
- but we do not want to have any complex logic which goes beyond filtering and converting rows to entities
- explicitly NOT one repository per entity (http://stackoverflow.com/q/52292717)

## Method

- i followed this approach in athena and first blindly created a new interface for each interactor. then
  i refactored the code and consolidated a few repositories
- i went for Usecase specific first
- then consolidated a lot
- then realized that a few interfaces go together most of the time so i crated facades again

i tend to prefer "repositories" for any kind of data access. for me that makes the intend pretty clear.
however for *athena* situation is different. handling workitems is no CRUD
"
- Does it really make sense to strictly separate between service and repository if we have no “Create-update-delete” but just read?
  - Maybe we should merge iworkitemrepository and iworkitemservice
"
==> i finally decided to keep it separate as the repository is used by interactors while the service
is "only" used by the controllers/presenters

let me show u how this looks like

### REPOSITORIES/Services/Gateways

- Must be rather dump
  - Must not have any logic
  - Must not call any interactor
- Still provide APIs convenient for the interactor (unit of work)
- If logic is required then interactor calls interactor
  - This way the logic remains in interactors only and we don’t need to search multiple indirections to understand a usecase
  - ==> document in “design decision” and on “plainionist blog”

==> We should also be carefull with injecting interactor into another interactor through generic interface – semantic might be lost but important
- Check with some examples
- Question is probably whether the “abstraction” is losing imporant info/context or not



## HOW to cache results of an interactor?
(does this fit here?)

- the cache impl is definitively a detail (e.g. HttpRuntime.Cache in my case for Asp.Net MVC)
- two possibilities
  - a) i pass cache through an interface to interactors
  - b) i hide caching behind a service and ask interactors to use service instead of interactor directly
- finally decided to go for b) (favor inversion of control)

## Accessing the TFS (database)

- show that we have multiple repositories (UML?)
- show that the interfaces are defined by the use cases
- show impl of one repository (code really relevant or just UML?)
- show shared code to query TFS work items always in the right way
- show how i handle the parsing (repository calls interactor directly - which i try to avoid but in this case it is pragmatic again)

repository returns entities
and we do not want to let any DB specific types out

http://stackoverflow.com/q/47903739 (which objects to return)


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


{% include series.html %}
