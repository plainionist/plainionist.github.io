re read the chapters !!
get all the details !!



# What is a UseCase?

a lot of theory - where to fold in an example?

- picture from uncle bob from "a typical scenario" (page 207)

maybe we start with describing one "aspect" of athena?

https://softwareengineering.stackexchange.com/questions/346847/clean-architecture-how-to-split-up-use-cases-dealing-with-use-case-dependenci


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


# The Main module

https://softwareengineering.stackexchange.com/questions/364424/crossing-boundaries-in-clean-architecture?answertab=votes#tab-top

## Dependency Injection?

- doesn't this mean that there is something wrong with using @Inject constructor for classes in the higher levels?
  - Strictly speaking, yes: DI frameworks should also not be used in use case or entities circle. (That includes attributes and annotations)


# Tests in Clean Architecture

- i am lazy and i always struggle to find a good balance between DRY and testing
- "what is simple enough that writing a test would violate DRY"?
- with this strange idea in mind i currently try to put - as i also think i read it in uncle bobs post - all code in the usecases/entities and test it
- i want to have all gateways that dump that testing feels like violating DRY (the code already clearly documents itself and is too simple that a test adds value - it is obvious)

# What to put in Entities in a micro services architecture?

- uncle bob: enterprise wide
- dont have that - just one app
- then central things
- but i want to keep MS as independent as usefull?
- so i decided to go bottom up: i handle each MS as separate app and put local entities
- if i find entities which are valuable to be shared across multiple microservices i put them in shared lib

# How to related DDD & Clean Architecture

- i first read clean architecture and then Erics DDD
- when reading DDD my brain immediately started comparing both books
- my conclusion
- entities and usecases map to entitis
- ddd says more about how to model entities and usecases
- ddd also talks about iterative and framework independence .

# what is really the benefit/difference to clean architecture?

- is it the independency to frameworks?
- is it the focus on the usecases? (not driven by UI or DB - driven by the usecase of the domain)
- is it the "screaming" architecture?


# Athena

## overview

draw a scetch of Athenas architecture
separate domain objects for backlog and burndown and governance

## One complete example

show how one scenario would be designed in clean-arch.
what is usecase, what is gateway, ...



# Further Learning

- https://www.codingblocks.net/tag/clean-architecture/

# Open Questions

- https://stackoverflow.com/questions/48589192/dependency-from-gateway-to-framework-in-clean-architecture

