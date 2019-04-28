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


{% include series.html %}
