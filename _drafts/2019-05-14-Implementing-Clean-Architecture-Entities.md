---
layout: post
title: Implementing Clean Architecture - Critical Business Rules
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---


domain could be layered
- iworkitem
- improvement


(IMAGE - entities circle)

After circling around the center of the clean architecture image for a long time we finally reached the center ;-)

now lets look what is in the "entities" circle ...

<!--more-->

First lets remember what Uncle Bob sees in that circle:

"
We shall call these rules Critical Business Rules, because they are critical to the business itself, and would exist 
even if there were no system to automate them. Critical Business Rules usually require some data to work with. 
For example, our loan requires a loan balance, an interest rate, and a payment schedule. We shall call this data 
Critical Business Data. This is the data that would exist even if the system were not automated. The critical rules 
and critical data are inextricably bound, so they are a good candidate for an object. We’ll call this kind of object an Entity.

The Entity is pure business and nothing else.
"

that means: entities are NOT just dump data objects. there is logic!

## But i dont have an "enterprise"

Entities encapsulate enterprise-wide Critical Business Rules. An entity can be an object with methods, or it can be a set 
of data structures and functions. It doesn’t matter so long as the entities can be used by many different applications in 
the enterprise. If you don’t have an enterprise and are writing just a single application, then these entities are the 
business objects of the application. They encapsulate the most general and high-level rules. They are the least likely 
to change when something external changes. For example, you would not expect these objects to be affected by a change 
to page navigation or security. No operational change to any particular application should affect the entity layer.

## Shouldnt i have started the series with this post?

if domain is clear and we start a green field project probably we should have started with entities.

==> ddd happens here!
==> u can also have ddd in application specific business objects (interactors)


if we would have done DDD then we would have started probably with this circle.
why didnt i do?

i tend to start with fewer entities - the entities i am sure about.
i tend to start with less methods in the entities.
then i implement use cases - the more i implmenent the more i learn about the domain the more code/logic
goes into the entities. of course i could have done more domain analysis and modeling up front ... but i dont 
like it much. i like to work incrementally. i like to have the first usecases up and running fast. 
but i also like to deferre heavy weight decisions. and so i keep my logic "private to the interactors"
until i learned enough to realize that certain knowledge is actually "application independenty business rules"

## The obvious entities

lets start with the obvious ones:
- work items
- team
- milestones
- fruit
- ...

## ... Would exist even if there were no system to automate them

- calculation of availability
- ...


## Can entities access repositories?

"
The Entity is pure business and nothing else.
"

https://stackoverflow.com/questions/47896909/should-a-domain-entity-call-a-repository
https://stackoverflow.com/questions/50943510/should-entities-in-clean-architecture-know-of-persistence-mechanisms


{% include series.html %}
