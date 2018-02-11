---
layout: post
title: Implementing Clean Architecture - What is a use case?
description: Clean Architecture is very much focusing on business by emphasizing use cases. But what is a use case? How big should it be? How does it interact with its environment?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
---

start with a picture?

so now we have a screaming architecture which screams out the business.
that gives most focus on use cases.
but what is a use case?

<!--more-->

## Definitions

a first attempt i could try is looking for some definition:

[Wikipedia](https://en.wikipedia.org/wiki/Use_case):

> In software and systems engineering, a use case is a list of actions or event steps typically defining the interactions 
> between a role (known in the Unified Modeling Language as an actor) and a system to achieve a goal.

[Clean Architecture book](/Clean-Architecture):

> These use cases orchestrate the flow of data to and from the entities, and direct those entities to use their 
> Critical Business Rules to achieve the goals of the use case.


## Examples

ok the definition does not help me much - too generic - lets try with exampled

typical example from uncle blob
- picture from uncle bob from "a typical scenario" (page 207)
- redraw it!
- shortly summarize his scenario

==> what is the "use case" here?
==> i dont know as there is no word about business
==> i just see that interactor is independent from many things

that is a web sample which fits very well to athena sample project
so lets make it more concreate by taking one example from athena

name an athena usecase and draw a concrete highlevel picture
- mention the project setup again: asp.net mvc, web, f#, ...

https://softwareengineering.stackexchange.com/questions/346847/clean-architecture-how-to-split-up-use-cases-dealing-with-use-case-dependenci

## how big should a usecase be?

==> break down athena usecase according to SRP
==> gives many interactors

- https://stackoverflow.com/questions/48141142/how-to-handle-usecase-interactor-constructors-that-have-too-many-dependency-para
- https://stackoverflow.com/questions/47934312/how-big-or-small-should-a-use-case-interactor-be-in-clean-architecture
- can a usecase really be independent of the outer world?
- would a usecase just be one function in F#?
- https://softwareengineering.stackexchange.com/questions/362071/clean-architecture-too-many-use-case-classes
  - A Use Case is not "Add a Customer Record." A Use Case is more along the lines of "Sell an item to a customer" 
    (which involves Customer, Product, and Inventory entities) or "Print an invoice" (which involves the same 
    entities, in addition to Invoice Header and Invoice Line Items).
- https://softwareengineering.stackexchange.com/questions/364725/do-interactors-in-clean-architecture-violate-the-single-responsibility-princip

- usecases can be big or small - depending on the abstraction level u look from
- in clean architecture use case "interactors" tend to be small. u want to follow SRP

## Can I reference use cases from use cases?

- refer to stackoverflow questions
- my conclusion ? yes because of SRP
- https://stackoverflow.com/questions/47868684/in-clean-mvp-who-should-handle-combining-interactors
 
## how do i access the database then?

in general

"
Between the use case interactors and the database are the database gateways. 2 These gateways are polymorphic interfaces that contain methods for every create, read, update, or delete operation that can be performed by the application on the database. For example, if the application needs to know the last names of all the users who logged in yesterday, then the UserGateway interface will have a method named getLastNamesOfUsersWhoLoggedInAfter that takes a Date as its argument and returns a list of last names.

Martin, Robert C.. Clean Architecture: A Craftsman's Guide to Software Structure and Design (Robert C. Martin Series) (p. 214). Pearson Education. Kindle Edition. 
"

==> separate post

## How to interact with controller/presenter?

- request/response model
- https://softwareengineering.stackexchange.com/questions/331479/c-wpf-clean-architecture
- picture shows it clearly
  - from controller to usecase
  - from presenter to usecase
  - all dependencies towards usecase! ==> Dependency Inversion
- if the usecase would have to "call" the presenter define e.g. an interface on usecase level
  most convenient for the usecase. this can be implemented by presenter. so we can "notify" the presenter
- in asp.net controller and presenter are the same class?

page 207


### What should be returned from a UseCase? 

UseCases define input DTOs (Data transfer objects) and output DTOs which are most convenient for the use case. 
in his book uncle bob writes that entities should not be passed to use cases or returned from use cases

"
We don’t want to cheat and pass Entity objects or database rows. 
We don’t want the data structures to have any kind of dependency that violates the Dependency Rule.
"

"
Thus, when we pass data across a boundary, it is always in the form that is most convenient for the inner circle.
"


what is then actually the role of the controller and presenter?

==> separate post

