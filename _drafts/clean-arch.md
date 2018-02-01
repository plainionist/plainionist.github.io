

## Clean Architecture - More Material

i have blogged about clean architecture already here

i digged deeper and found
- 


- https://hackernoon.com/applying-clean-architecture-on-web-application-with-modular-pattern-7b11f1b89011

- https://medium.com/@stephanhoekstra/clean-architecture-in-net-8eed6c224c50

- http://nklein.com/exploring-clean-architecture

https://fullstackmark.com/post/11/better-software-design-with-clean-architecture

https://github.com/mattia-battiston/clean-architecture-example

https://github.com/mp911de/CleanArchitecture

http://tidyjava.com/clean-architecture-screaming/



stackoverflow
- dependency rule between framework and gateway?


start with a better intro and base it on backlook!
i know the usecases better

"do u also run into the situation that u read a book and it is all cool and u want it try out and get stucked with questions? ..."
"here is how i currently try to "implement clean architecture""
"this ll be a series of posts sharing the experiences i have"





==> describe backlook as an example?


how do i start?
- designing a new project
- with the use cases?
- now intro backlook




- what is a use-case? how big is it? how do we connect usecase?
  can a usecase really be independent of the outer world?
  (dont mix with usecase uml diagrams)
  in F# is one usecase one public function?

- what is really the benefit/difference to clean architecture?
  is it the independency to frameworks?
  is it the focus on the usecases? (not driven by UI or DB - driven by the usecase of the domain)
  is it the "screaming" architecture?
 
- how do i access the database then?

https://www.codingblocks.net/tag/clean-architecture/













referencing use cases?

-	refer to stackoverflow questions
-	my conclusion ? yes because of SRP

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





