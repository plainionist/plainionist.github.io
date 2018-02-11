---
layout: post
title: Implementing Clean Architecture - What is a use case?
description: Clean Architecture is very much focusing on business by emphasizing use cases. But what is a use case? How big should it be? How does it interact with its environment?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
draft: true
excerpt_separator: <!--more-->
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.UseCase.png" class="dynimg"/>

Now that my architecture is screaming the business capabilities of my system let's look at those with more detail.

In the Clean Architecture all the application specific business rules and business logic goes into the "use cases" circle.

But what is a use case? How big should it be? How does it interact with its environment?

Read on!

<!--more-->

## Definitions

As a starting point for answering these questions I like to fish for find some definitions ...

[Wikipedia](https://en.wikipedia.org/wiki/Use_case):

> In software and systems engineering, a use case is a list of actions or event steps typically defining the interactions 
> between a role (known in the Unified Modeling Language as an actor) and a system to achieve a goal.

[Clean Architecture book](/Clean-Architecture):

> These use cases orchestrate the flow of data to and from the entities, and direct those entities to use their 
> Critical Business Rules to achieve the goals of the use case.

Ok, these definitions are rather high-level and nothing concrete. I kinda expected that ;-)

Enough of theory - let's something more practical ...

## The example

To recap, *[Athena](/Implementing-Clean-Architecture)* is an "agile backlog visualizer" implemented in F# using Asp.Net MVC.
It provides various reports to improve the overall transparency in a software project. One major report is showing the
ranked backlog with cut-lines:

<img src="{{ site.url }}/assets/clean-architecture/backlog.png" class="dynimg"/>

Is providing this page a use case? On the most highest and abstract level: yes.
The simplified "list of actions" is:

- Get all workitems from TFS (Team Foundation Server)
- Rank all workitems
- Get the team capacity 
- Determine the cut-lines
- Generate the report (including all further details like TFS links and hints for missing information)

This would probably be the level on which I would specify an expected system behavior. 
I might add more details or draw a UML use case diagram.

But should I implement this use case in a single use case interactor?

## How big should a use case be?

There is no ideal size of a use case. Use cases can be long or short, high level or more concrete. It all depends on the 
level of abstraction we want to consider.

Let me rephrase the question: How big should a use case INTERACTOR be?

So what would be a driving priciple for deciding about "size"? Correct: Single Responsibility Pattern (SRP).

Let's look again at the list of actions from above and explore some more details.

### Action: Get all workitems from TFS

This is clearly data access, isn't it? 

Yes and no. Obviously fetching the (raw) data from TFS is data access. But there are also business rules. 
The TFS itself is rather generic. 
We have put quite some custom conventions on top of it like: fruits, confidence levels, product lines and many more.

Interpreting plain text fields and generic tags and giving these information business semantics is about business rules.
So I want to have this logic in the "use case circle". Guided by the SRP I would say

&#10142; First use case interactor found


<picture of class WorkitemParserInteractor with "ParseFruit", "ParseConfidenceLevel">

### Action: Rank all workitems

Ranking the backlog is definitively about business rules as those define how the ranking should happen. Ideally this is 
a very simple task which just sorts workitems by stack rank. 
But reality is rarely ideal so I could imagine additional rules, e.g.:

- Always rank [MMP](https://agilesoftwareengineer.com/2013/08/28/minimum-viable-product-vs-minimum-marketable-product/) higher than "best effort"
- Rank workitems without any given rank lowest
- In case of duplicate ranks, rank one product line over the other product line.

&#8680; As this has nothing to do with backlog conventions and parsing I will add a second interactor.

<picture RankingInteractor with "Rank" as method>

### Action: Get the team capacity 

This is again mostly about data access. We have a non-TFS data source where all teams are modeled with 
their ramp up and ramp down in head count for a given time frame. Getting this (raw) data is pure data access.

In a second step I need to calculate the team capacity from the given data. This step is again about business rules:

- Substract corridors like hotfixes for the installed base from the modeled head count
- Convert head count into person days by applying an "availability factor" (e.g. remove holidays)
- Apply correct time frame (e.g. duration of an iteration or software version)

&#10145; As this involves much different entities than the previous two interactors I will create a third one.

<picture TeamCapacityInteractor with "GetTotalCapacity(teams,from,to)", "GetTeamCapacity(team,from,to)">

### Action: Determine the cut-lines

Now that I have a ranked backlog and the capacity I can determine the cut-lines.

> *Side note*: You may wonder why I talk about multiple cut-lines. This comes from the confidence levels. Each confidence level
> defines a certain percentage by which the actual estimation is correct, e.g. confidence level "low" could be defined  as "+/- 50%" 
> which means that the actual estimation could be "50% too expensive" or "50% too cheap". This approach basically gives three 
> numbers for each estimation: an optimistic, a pessimistic and a realistic one. Hence, we have three cut-lines.

Calculating the cut-lines is a simple algorithm which walks the backlog from top to bottom, sums up estimations and 
matches these against the capacity. It is clearly about business rules so I will add it to a use case interactor.

&U21D2; Considering the existing interactors and my favor of pragmatic decisions I will put this logic into the RankingInteractor

<picture RankingInteractor with "GetCutLines(rankedBacklog, capacity)">

### Action: Generate the report

On the first glance this sounds like "UI work", but that's not all. Here are further business rules to consider:

- Create hyperlinks for each workitem to the TFS
- Indicating missing information (missing confidence level, missing estimation)
- add total capacity for reference
- add total effort for reference
- aggregate involved team membmers for the filters
- Create e-mail links

Eventhough this sounds like UI logic I see it as business logic. The business rules are the ones deciding how
we compile this report together. The business rules define what is important in that report and what not.

so lets have an interactor which prepares all that so that the presenter has an easy job.

&U21D2; Even with my pragmatic view this logic doesn't fit nicely into any existing interactor without violating SRP. So let me create a new one.

<picture BacklogInteractor with "GenerateReport(filter)">

## Can I reference use cases from use cases?

We have created three interactors so far. But how do we assembly them together to get the use case implemented?

Basically there are two possibilities:

1. Combine the interactors in the controller, so in the interface adapters circle
2. Combine the interactors in another interactor, so in the use cases circle

I could easily find pros and cons for each alternative. The driving question for me would be, how much (business) logic
is involved in combining the other interactors?

I want the interface adapters (controllers, presenters) rather dumb. So as soon as some logic is involved in combining 
interactors I prefer having another interactor realizing the combination.

For the use case discussed here I tend to make again a pragmatic decision: I will make the BacklogInteractor the "aggregate interactor"
which calls the other interactors. Which leaves us with that picture

<picture of interaction: backloginteractor to other interactors>

## How do I access the database then?

Uncle Bob writes in his book:

> Between the use case interactors and the database are the database gateways. 
> These gateways are polymorphic interfaces that contain methods for every create, read, update, 
> or delete operation that can be performed by the application on the database. For example, if the 
> application needs to know the last names of all the users who logged in yesterday, then the UserGateway 
> interface will have a method named getLastNamesOfUsersWhoLoggedInAfter that takes a Date as its argument and returns a list of last names.

For our use case I will define two interfaces. One to get the workitems from TFS and one to get the team capacity information from
the external service.

<picture of the two interfaces with methods>

The details about accessing the "outer world" like database, TFS and other services I will discuss in one of the next posts

## How to interact with controller/presenter?

According to the Dependency Rule a use case interactor must not depend on a controller or presenter.

Instead the use case interactor defines "input and output ports" to invert the dependencies.

<picture from use cases from uncle bob>

In our use case the setup is more simple. As we use Asp.Net MVC the controller and the presenter are the same class: the Asp.Net MVC conroller.

All methods we have defined on the interactors so far are simple functions which return results.

This would give us this picture:

<picture like page 207 with our use case .. dependencies>


### What will be passed to and returned from a use case? 


UseCases define input DTOs (Data transfer objects) and output DTOs which are most convenient for the use case. 
in his book uncle bob writes that entities should not be passed to use cases or returned from use cases

"
We don't want to cheat and pass Entity objects or database rows. 
We don't want the data structures to have any kind of dependency that violates the Dependency Rule.
"

"
Thus, when we pass data across a boundary, it is always in the form that is most convenient for the inner circle.
"

Therefore we dont need to define input or output ports as interfaces - we can have simple DTOs for input and output.


what is then actually the role of the controller and presenter?

==> separate post

## How do others think about use cases?

During research for this post I found many discussions about the "right cut" of use cases. 
Here is a list of some well crafted thoughts:

- [How big or small should a Use Case Interactor be in Clean Architecture?](https://stackoverflow.com/questions/47934312/how-big-or-small-should-a-use-case-interactor-be-in-clean-architecture)
- [Do Interactors in clean architecture violate the Single Responsibility Principle?](https://softwareengineering.stackexchange.com/questions/364725/do-interactors-in-clean-architecture-violate-the-single-responsibility-princip) 
- [Who should handle combining interactors?](https://stackoverflow.com/questions/47868684/in-clean-mvp-who-should-handle-combining-interactors)
