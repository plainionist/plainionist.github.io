
## some core questions for the over all architecture

- what is a use-case? how big is it? how do we connect usecase?
  can a usecase really be independent of the outer world?
  (dont mix with usecase uml diagrams)
  in F# is one usecase one public function?
  https://stackoverflow.com/questions/48141142/how-to-handle-usecase-interactor-constructors-that-have-too-many-dependency-para
https://stackoverflow.com/questions/47934312/how-big-or-small-should-a-use-case-interactor-be-in-clean-architecture

 
- how do i access the database then?
  - how do i update it?
  - irepository is in usecase layer and impl by datalayer
    (in layered arch it is usually the other way round)

https://www.codingblocks.net/tag/clean-architecture/

- Can I reference use cases from use cases?
  - refer to stackoverflow questions
  - my conclusion ? yes because of SRP

https://stackoverflow.com/questions/47896909/should-a-domain-entity-call-a-repository
https://stackoverflow.com/questions/47868684/in-clean-mvp-who-should-handle-combining-interactors

https://softwareengineering.stackexchange.com/questions/331479/c-wpf-clean-architecture
https://softwareengineering.stackexchange.com/questions/362071/clean-architecture-too-many-use-case-classes
https://softwareengineering.stackexchange.com/questions/364424/crossing-boundaries-in-clean-architecture?answertab=votes#tab-top
https://softwareengineering.stackexchange.com/questions/303478/uncle-bobs-clean-architecture-an-entity-model-class-for-each-layer

how to access the outer world
https://softwareengineering.stackexchange.com/questions/315558/where-to-put-peripheral-use-cases-in-android-while-using-clean-architecture


https://softwareengineering.stackexchange.com/questions/364725/do-interactors-in-clean-architecture-violate-the-single-responsibility-princip

https://softwareengineering.stackexchange.com/questions/346847/clean-architecture-how-to-split-up-use-cases-dealing-with-use-case-dependenci

https://softwareengineering.stackexchange.com/questions/351419/clean-architecture-validation-in-domain-vs-data-persistence-layer

# Athena

## overview

draw a scetch of Athenas architecture
separate domain objects for backlog and burndown and governance

## by example

show how one scenario would be designed in clean-arch.
what is usecase, what is gateway, ...



## misc

testing

-	i am lazy and i always struggle to find a good balance between DRY and testing
-	"what is simple enough that writing a test would violate DRY"?
-	with this strange idea in mind i currently try to put - as i also think i read it in uncle bobs post - all code in the usecases/entities and test it
-	i want to have all gateways that dump that testing feels like violating DRY (the code already clearly documents itself and is too simple that a test adds value - it is obvious)

What to put in Entities in a micro services architecture?

-	uncle bob: enterprise wide
-	dont have that - just one app
-	then central things
-	but i want to keep MS as independent as usefull?
-	so i decided to go bottom up: i handle each MS as separate app and put local entities
-	if i find entities which are valuable to be shared across multiple microservices i put them in shared lib

How to related DDD & Clean Architecture

-	i first read clean architecture and then Erics DDD
-	when reading DDD my brain immediately started comparing both books
-	my conclusion
-	entities and usecases map to entitis
-	ddd says more about how to model entities and usecases
-	ddd also talks about iterative and framework independence .
-	...

clean architecture and f#
- is a usecase just a function?
- what about depenecy injection. reference scott about dependency injection in f#



- what is really the benefit/difference to clean architecture?
  is it the independency to frameworks?
  is it the focus on the usecases? (not driven by UI or DB - driven by the usecase of the domain)
  is it the "screaming" architecture?


## stackoverflow

https://stackoverflow.com/questions/48563184/dagger-and-business-logic-and-prensenters/48571468#48571468
Q:
I'm already using Dagger2 and everything is working but I have a doubt about the proper way to integrate it into the business logic. What Robert Martin says in "Clean Architecture" is that the DI frameworks, since they are frameworks, are details that should be kept away from the Entity and Use cases and more in general from all the classes that are at a higher level than the frameworks.

What R.M. suggests is to allow only the Main-module to know the DI framework used and to inject the other classes by yourself in such a way that you can replace one DI framework with another one without having to change the BL.

The question is, for example, doesn't this mean that there is something wrong with using @Inject constructor for classes in the higher levels?
 
A:
Strictly speaking, yes: DI frameworks should also not be used in use case or entities circle. (That includes attributes and annotations)

The question would be how strict u want to handle this rule in ur project. Every rule and decision has pros and cons. As u said the pro of keeping DI out of the inner circles would be that u could easily replace it later. U would have to decide how big the benefit is compared to the cons, e.g.: having to pass dependencies to use cases manually.

Personally I currently try to handle it very strict in my projects. But my usecases tend to have only few dependencies ...
 

 https://stackoverflow.com/questions/48356193/clean-architecture-where-to-implement-pagination-logic
 Q:
 There is a REST API where search keyword entered by the user is used to query and get results. Sometimes, too many results are returned. I don't want to put a maximum result limit on the server side so I want to handle it on the application. In the application, I try to follow Clean Architecture. I have a fragment, a presenter, a usecase and an API client. User enters a keyword, presses search button, the keyword passed to related usecase function through presenter. Usecase gets results from API client and pass results to presenter through listener. Presenter notifies fragment so that results are displayed.

I want to show max of ten pages of results. Where should I put this control? Usecase or presenter?

A:
If you will strictly make it ten pages ALWAYS, put it on your usecase because here, application business rules resides. So you don't need to pass it if your just always going to pass ten. 

But, I suggest to make it as a parameter on the presenter, to make it flexible because maybe you will have a scenario in which you want to adjust the max pages on a specific activity/fragment.




https://stackoverflow.com/questions/48119036/clean-architecture-usecases-and-entities/48589325#48589325

Quite a lot questions within a single question. let me try to consolidate what I think I understood are ur key questions
•Can Entities reference each other? the answer would be: YES. Also in Clean Architecture u can create a domain model where entities are interconnected
•What should be returned from a UseCase? Answer: UseCases define input DTOs (Data transfer objects) and output DTOs which are most convenient for the use case. in his book uncle bob writes that entities should not be passed to use cases or returned from use cases
•What is the role of the presenter then? Answer: ideally a presenter is converting data only. It converts data which is most convenient for one layer into data which is most convenient for the other layer.
hope this guidance helps u to answer ur detailed questions






## open questions

- https://stackoverflow.com/questions/48589192/dependency-from-gateway-to-framework-in-clean-architecture







