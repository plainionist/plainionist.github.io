



# Update: Make your architecture scream

lets write it as "update" and keep finally decision is open (until athena is fully refactored and we can give clear recommendation)

https://softwareengineering.stackexchange.com/questions/366930/android-clean-architecture-best-way-to-structure-packages/366945?noredirect=1#comment800927_366945

==> update the "scream" post with discussion about "composable UI"


- "gateway" restricted to adapters to repository and services? should we rename
  assemblies also holding controller and presetner to "adapters"? and should we 
  recommend to separate these assemblies?
  ==> check in the book first

consider also conclusion from asp.net discussion

"
The discussion in this post has shown how important borders are in an architecture. With that in mind a simplified project 
structure should draw a border between framework independent code and framework dependent code. That would give us

- one project per business aspect containing framework depending code
- one project per business aspect containing interface adapters and use cases
- single project containing entities

This approach clearly draws the two most important boarders between frameworks and application and between application
specific logic and enterprise business rules (entities).
"





# DataAccess

the database is a detail!

we want those details to be plugins into our application architecture.

How do we cut “external” services and entities?
==> Unit of work
==> started with rather interactor specific interfaces and then checked whether I could meaningfully consolidate things. 
    If I were in doubt I got them separated - PlanningService
    (depends how clear ur understanding of your units of work already is - personally i prefer to start locally and small and then refactor)

consider:
- http://stackoverflow.com/q/48141142
- http://stackoverflow.com/q/48139897
- http://stackoverflow.com/q/47903739
- http://stackoverflow.com/q/47446180


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


http://stackoverflow.com/q/47446180




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

==> When blogging about entities – update details on what is in the interactor implementations
 ("availability calculation" becomes entity!)


## Can entities access repositories?

https://stackoverflow.com/questions/47896909/should-a-domain-entity-call-a-repository




# The Main module

how do i wire up all the things?
who instanciates interactors?
https://softwareengineering.stackexchange.com/questions/364424/crossing-boundaries-in-clean-architecture?answertab=votes#tab-top

who injects the presenter through the output port into the interacotr?

consider:
- https://stackoverflow.com/questions/49066847

## Dependency Injection?

- doesn't this mean that there is something wrong with using @Inject constructor for classes in the higher levels?
  - Strictly speaking, yes: DI frameworks should also not be used in use case or entities circle. (That includes attributes and annotations)


how to handle too long constructor parameter lists on the use cases?
- common topic - not specific to clean arch
- split US or use objects to "bundle" dependencies
- think of facades
- if a use cases just does "one thing well" it should not have two many dependnecies
- if u use "unit of work" and have interfaces to the outer services and repositories also the number
  of dependencies should be low
- https://stackoverflow.com/questions/49066847/how-to-inject-into-dynamically-created-use-cases-android-clean-architecture-d/49081113#49081113
- http://stackoverflow.com/q/48141142



# Cheat Sheet

now that we discussed all layers including the main module here is the cheat sheet

- first put all logic in the use cases
- make the controllers and presenters dump data converters  
  - request to request model, response model to response
- no logic in the views at all
- main is about composition only
- repositories and services focus on accessing details only
- entities get these rules which do not change across use cases
- 


# Where is what? - Example

## Is the presenter view specific?

i think so. if u have an html view the presenter can know about html. see print example from uncle bob.
or the other way round: if not the presenter is preparing html snippets it means that u have logic in the view
what we want to avoid - also because of testability.

## input validation?

- https://softwareengineering.stackexchange.com/questions/351419/clean-architecture-validation-in-domain-vs-data-persistence-layer
- http://stackoverflow.com/q/47860684


- where to do threading? in gateways (service adapters)


# partial boundaries

- do i really have to create all those DTOs?
- what about performance?
- is it worth the effort?

## pagination

https://stackoverflow.com/questions/48356193/clean-architecture-where-to-implement-pagination-logic


## Must every interactor return a response model?

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

Where do I do acceptance testing in clean arch?
==> On US?
==> On controller / presenter?
==> interactor … then simple UT for presenter and controller sufficient (because only doing data convertion)

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

- after having refactored quite some code from non clean arch to clean arch i can say: "it is unbelievable how the separation of
  interactor and presenter simplifies the design", "how magically entities occur - interactors have interactor specific requests and response. 
  everthing generic to these is entities!?"

- clean architecture supports greatly to defer decisions about UI, DB, cache, etc 
  we could just start with bdd tests as "driver" for use cases


# how to apply clean architecture to command line apps

- Plainion.JekyllLint


# Athena

## overview

draw a scetch of Athenas architecture
separate domain objects for backlog and burndown and governance

## One complete example

show how one scenario would be designed in clean-arch.
what is usecase, what is gateway, ...




# other blogs

- https://www.codingblocks.net/tag/clean-architecture/


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

