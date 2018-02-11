---
layout: post
title: Implementing Clean Architecture - What is a use case?
description: Clean Architecture is very much focusing on business by emphasizing use cases. But what is a use case? How big should it be? How does it interact with its environment?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
---

In Clean architecture there is very much focus on use cases.
So what is a use case?



re read the chapters !!
get all the details !!

a lot of theory - where to fold in an example?

- picture from uncle bob from "a typical scenario" (page 207)

maybe we start with describing one "aspect" of athena?

https://softwareengineering.stackexchange.com/questions/346847/clean-architecture-how-to-split-up-use-cases-dealing-with-use-case-dependenci

<!--more-->

## Definitions

(Wikipedia)[https://en.wikipedia.org/wiki/Use_case]:

> In software and systems engineering, a use case is a list of actions or event steps typically defining the interactions 
> between a role (known in the Unified Modeling Language as an actor) and a system to achieve a goal.




## how big should a usecase be?

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

- irepository is in usecase layer and impl by datalayer
  (in layered arch it is usually the other way round)
- https://softwareengineering.stackexchange.com/questions/315558/where-to-put-peripheral-use-cases-in-android-while-using-clean-architecture

## How to interact with controller/presenter?

- request/response model
- https://softwareengineering.stackexchange.com/questions/331479/c-wpf-clean-architecture

## What should be returned from a UseCase? 

UseCases define input DTOs (Data transfer objects) and output DTOs which are most convenient for the use case. in his book uncle bob writes that entities should not be passed to use cases or returned from use cases

## What is the role of the presenter then? 

ideally a presenter is converting data only. It converts data which is most convenient for one layer into data which is most convenient for the other layer.

## Can entities access repositories?

https://stackoverflow.com/questions/47896909/should-a-domain-entity-call-a-repository

## input validation?

- https://softwareengineering.stackexchange.com/questions/351419/clean-architecture-validation-in-domain-vs-data-persistence-layer

