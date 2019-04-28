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


{% include series.html %}
