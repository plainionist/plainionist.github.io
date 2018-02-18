
# Interface Adapters


## How to apply EntityFramework or other ORM?

page 214

- irepository is in usecase layer and impl by datalayer
  (in layered arch it is usually the other way round)
- https://softwareengineering.stackexchange.com/questions/315558/where-to-put-peripheral-use-cases-in-android-while-using-clean-architecture

## how to communicate with other systems?

or IO devices? or serivces? 

page 215

# Entities

application independent business objects

the "interactor controls the dance of the entities"

that means: entities are NOT just dump data objects. there is logic!

==> ddd happens here!
==> u can also have ddd in application specific business objects (interactors)

## Can entities access repositories?

https://stackoverflow.com/questions/47896909/should-a-domain-entity-call-a-repository


# Where is what? - Example

## input validation?

- https://softwareengineering.stackexchange.com/questions/351419/clean-architecture-validation-in-domain-vs-data-persistence-layer


# partial boundaries

- do i really have to create all those DTOs?
- what about performance?
- is it worth the effort?


# The Main module

how do i wire up all the things?
who instanciates interactors?
https://softwareengineering.stackexchange.com/questions/364424/crossing-boundaries-in-clean-architecture?answertab=votes#tab-top



## Dependency Injection?

- doesn't this mean that there is something wrong with using @Inject constructor for classes in the higher levels?
  - Strictly speaking, yes: DI frameworks should also not be used in use case or entities circle. (That includes attributes and annotations)


# Tests in Clean Architecture

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

- but what about all the smart javascript in a modern SPA?
- maybe u can use frameworks to keep also the javascript as simple as possible - i dont want to test it :)
- what about vue or react and binding?

==> in the end u probably just need a very simple integration test which is running thrugh UI to check
    that everything is wired up correctly. but NO deep logic testing through UI!

# relation to other patterns

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

