

# The Web and the Database and other Details

the web is a detail!
the db is a details

we want those details to be plugins into our application architecture.

## But what about the database?

but all meaningful app once does persist s.th. right?
we want to store data efficiently.
access it fast
and model it well to avoid redundancy - we want normalization?

but is it really still like that? in 2000 the database seemt to be the god of the applicaiton architecture.
then there was the time about ORM - abstract the database away
today is about big data ... we dont care about the kind of db! could be even a simple json based document store?
we talk about micro services and eventual consistency?

when did u heards s.o. talking about "normalization of the schema" the last time?

maybe DB has become less important ...

## How to apply EntityFramework or other ORM?

page 214

- irepository is in usecase layer and impl by datalayer
  (in layered arch it is usually the other way round)
- https://softwareengineering.stackexchange.com/questions/315558/where-to-put-peripheral-use-cases-in-android-while-using-clean-architecture

## How do i handle transactions?

how do i do it across multiple interactors?

- transactions are necessary because we store state
- what if we only store events (+snapshots)
- CRUD goes to CR
- its like source code control action

==> no transactions!!

## how to communicate with other systems?

or IO devices? or serivces? 

page 215

## other details

ANY KIND of framework!
- dependency injection (MEF)
- ...



# Entities

application independent business objects

the "interactor controls the dance of the entities"

that means: entities are NOT just dump data objects. there is logic!

==> ddd happens here!
==> u can also have ddd in application specific business objects (interactors)

i tend to start with fewer entities - the entities i am sure about.
i tend to start with less methods in the entities.
then i implement use cases - the more i implmenent the more i learn about the domain the more code/logic
goes into the entities. of course i could have done more domain analysis and modeling up front ... but i dont 
like it much. i like to work incrementally. i like to have the first usecases up and running fast. 
but i also like to deferre heavy weight decisions. and so i keep my logic "private to the interactors"
until i learned enough to realize that certain knowledge is actually "application independenty business rules"



## Can entities access repositories?

https://stackoverflow.com/questions/47896909/should-a-domain-entity-call-a-repository




# The Main module

how do i wire up all the things?
who instanciates interactors?
https://softwareengineering.stackexchange.com/questions/364424/crossing-boundaries-in-clean-architecture?answertab=votes#tab-top

who injects the presenter through the output port into the interacotr?

## Dependency Injection?

- doesn't this mean that there is something wrong with using @Inject constructor for classes in the higher levels?
  - Strictly speaking, yes: DI frameworks should also not be used in use case or entities circle. (That includes attributes and annotations)






# Where is what? - Example

## input validation?

- https://softwareengineering.stackexchange.com/questions/351419/clean-architecture-validation-in-domain-vs-data-persistence-layer


# partial boundaries

- do i really have to create all those DTOs?
- what about performance?
- is it worth the effort?

## pagination

https://stackoverflow.com/questions/48356193/clean-architecture-where-to-implement-pagination-logic


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

