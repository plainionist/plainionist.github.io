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


stackoverflow
- dependency rule between framework and gateway?

