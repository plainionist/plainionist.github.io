---
layout: post
title: Implementing Clean Architecture - What is a use case?
description: Clean Architecture is very much focusing on business by emphasizing use cases. But what is a use case? How big should it be? How does it interact with its environment?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: JL0003
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.UseCase.png" class="dynimg" title="Deep diving the use cases circle." alt="From Clean Architectures circles lets take out the 'use cases' one and deep dive into it."/>

Now that [my architecture is screaming](/Implementing-Clean-Architecture-Scream) the business capabilities 
of [my system](/Implementing-Clean-Architecture) let's look at those with more detail.

In the Clean Architecture all the application specific business rules go into the use cases circle.

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

OK, these definitions are rather high-level and nothing concrete. I kinda expected that ;-)

Enough of theory - let's look at something more practical ...

## The example

To recap, *[Athena](/Implementing-Clean-Architecture)* is an "agile backlog visualizer" implemented in F# using Asp.Net MVC.
It provides various reports to improve the overall transparency in a software project. One major report is showing the
ranked backlog with cut-lines:

<img src="{{ site.url }}/assets/clean-architecture/backlog.png" class="dynimg" title="Ranked backlog with cut lines" alt="Our backlog is ranked and has multiple cut lines due to usage of confidence levels. The cut lines are visualized with the background color of the row. The darker the color gets the less likely the feature can be delivered for this release."/>

Is providing this report a use case? On the most highest and abstract level: yes.
The simplified "list of actions" is:

1. Get all workitems from TFS (Team Foundation Server)
2. Rank all workitems
3. Get the team capacity 
4. Determine the cut-lines
5. Generate the report (including all further details like TFS links and hints for missing information)

This would probably be the level on which I would specify an expected system behavior. 

But should I implement this use case in a single use case interactor?

## How big should a use case be?

There is no ideal size of a use case. Use cases can be long or short, high-level or more concrete. It all depends on the 
level of abstraction we want to consider.

Let me rephrase the question: How big should a use case INTERACTOR be?

So what would be a driving principle for deciding about "size"? Correct: Single Responsibility Pattern (SRP).

Let's look closer at the list of actions from above ...

### Action: Get all workitems from TFS

This is clearly data access, isn't it? 

Yes and no. Obviously fetching the (raw) data from TFS is data access. But there are also business rules. 
The TFS itself is rather generic. 
We have put quite some custom conventions on top of it like: fruits, confidence levels, product lines and many more.

Interpreting plain text fields and generic tags and giving these information business semantics is about business rules.
So I want to have this logic in the use cases circle. Guided by the SRP I would say

&#8680; First use case interactor found

<img src="{{ site.url }}/assets/clean-architecture/WorkitemParserInteractor.png" class="dynimg" title="API of the WorkitemParserInteractor" alt="The WorkitemParserInteractor provides methods to parse plain text from TFS fields to get domain object like fruits, confidence levels and product lines."/>

### Action: Rank all workitems

Ranking the backlog is definitively about business rules as those define how the ranking should happen. Ideally this is 
a very simple task which just sorts workitems by stack rank. 
But reality is rarely ideal so I could imagine additional rules, e.g.:

- Always rank [MMP](https://agilesoftwareengineer.com/2013/08/28/minimum-viable-product-vs-minimum-marketable-product/) higher 
  than "best effort"
- Rank workitems without any given rank lowest
- In case of duplicate ranks, rank one product line over the other product line.

&#8680; As this has nothing to do with backlog conventions and parsing I will add a second interactor.

<img src="{{ site.url }}/assets/clean-architecture/RankingInteractor.1.png" class="dynimg" title="API of the RankingInteractor" alt="The RankingInteractor provides methods rank a given list of work items."/>

### Action: Get the team capacity 

This is again mostly about data access. We have a non-TFS data source where all teams are modeled with 
their ramp up and ramp down in head count for one year. Getting this (raw) data is pure data access.

In a second step I need to calculate the team capacity from the given data. This step is again about business rules:

- Subtract corridors like hot fixes for the installed base from the modeled head count
- Convert head count into person days by applying an "availability factor" (e.g. remove holidays)
- Apply correct time frame (e.g. duration of an iteration or software version)

&#8680; As this involves quite different entities than the previous two interactors I will create a third one.

<img src="{{ site.url }}/assets/clean-architecture/TeamCapacityInteractor.png" class="dynimg" title="API of the TeamCapacityInteractor" alt="The TeamCapacityInteractor provides methods to calculate the capacity of one or multiple teams"/>

### Action: Determine the cut-lines

Now that I have a ranked backlog and the capacity I can determine the cut-lines.

> *Note:* You may wonder why I talk about multiple cut-lines. This comes from the confidence levels. Each confidence level
> defines a certain percentage by which the actual estimation is correct, e.g. confidence level "low" could be defined  as "+/- 50%" 
> which means that the actual estimation could be "50% too expensive" or "50% too cheap". This approach basically gives three 
> numbers for each estimation: an optimistic, a pessimistic and a realistic one. Hence, we have three cut-lines.

Calculating the cut-lines is a simple algorithm which walks the backlog from top to bottom, sums up estimations and 
matches these against the capacity. It is clearly about business rules so I will add it to a use case interactor.

&#8680; Considering the existing interactors and my preference of pragmatic decisions I will put this logic into 
the ```RankingInteractor```.

<img src="{{ site.url }}/assets/clean-architecture/RankingInteractor.2.png" class="dynimg" title="API of the RankingInteractor" alt="The RankingInteractor gets another method to calculate the cut-lines based on a given ranked backlog and team capacity"/>

### Action: Generate the report

At first glance this may look like "UI work", but that's not all. Here are further business rules to consider:

- Validate missing information (missing confidence level, missing estimation)
- Provide total capacity for reference
- Provide total effort for reference
- Compute list of involved team members (for the filters)

I want to have these business rules realized in an interactor as well.

&#8680; Even with my preference for pragmatic decisions this logic doesn't fit nicely into any existing interactor without 
violating SRP. So let me create a new one.

<img src="{{ site.url }}/assets/clean-architecture/BacklogInteractor.png" class="dynimg" title="API of the BacklogInteractor" alt="The BacklogInteractor compiles all the information wanted for the backlog page together into one response."/>

## How do I combine use case interactors?

We have created four interactors so far. But how do we "combine" them to realize the high-level use case?

Basically there are two possibilities:

1. Combine the interactors in the controller, so in the interface adapters circle
2. Combine the interactors in another interactor, so in the use cases circle

I could easily find pros and cons for each alternative. The driving question for me would be, how much (business) logic
is involved in combining the other interactors?

I want the interface adapters (controllers, presenters) rather dumb. So as soon as some logic is involved in combining 
interactors I prefer having another interactor realizing the combination.

For the use case discussed here I tend to make again a pragmatic decision: I will make the ```BacklogInteractor``` the 
"combining interactor" which calls the other interactors. 

<img src="{{ site.url }}/assets/clean-architecture/Interactors.Collaboration.png" class="dynimg" title="Dependencies between the interactors" alt="The controller will only talk to the BacklogInteractor which compiles the complete response model by interacting with other interactors"/>

## How do I access the database?

In his [book](/Clean-Architecture/) Uncle Bob writes about database access:

> Between the use case interactors and the database are the database gateways. 
> These gateways are polymorphic interfaces that contain methods for every create, read, update, 
> or delete operation that can be performed by the application on the database. For example, if the 
> application needs to know the last names of all the users who logged in yesterday, then the UserGateway 
> interface will have a method named getLastNamesOfUsersWhoLoggedInAfter that takes a Date as its argument and 
> returns a list of last names.

For our use case I will define two interfaces. One to get the workitems from TFS and one to get the team capacity 
information from the external service.

<img src="{{ site.url }}/assets/clean-architecture/Interactors.DataAccess.png" class="dynimg" title="Dependencies between interactors and data stores" alt="Within the use case layer we define an interface IWorkitemRepository and an interface ITeamService. These are used by the interactors to access TFS and the team planning data. These interfaces are implemented in the interface adapter circle."/>

Details about accessing "IO devices" and external systems I will discuss in one of my next posts.

## How do I interact with controller and presenter?

According to the Dependency Rule a use case interactor must not depend on a controller or presenter.
Instead, the use case interactor defines "input ports" and "output ports" to invert the dependencies.

<img src="{{ site.url }}/assets/clean-architecture/Interactor.Controller.Presenter.png" class="dynimg" title="Control flow from Controller to Presenter" alt="The Controller interacts with the interactor through the input port. The interactor passes its response to the presenter through an output port"/>

In the use case I have described here the setup is simpler.

- As I use Asp.Net MVC, the controller and the presenter are the same class: the Asp.Net MVC controller.
- All methods I have defined on the interactors so far are simple and pure functions which get parameters and 
  return results. So I don't need to define any "ports". I will simply pass DTOs (data transfer objects) around.

So I will just let the controller call ```BacklogInteractor.GetBacklog``` passing some filter DTO which will
return some report DTO as result.

Which role is than actually left to the controller and the presenter? The answer to this question I will leave to another post.

## How do others think about use cases and interactors?

During research for this post I found many discussions about the "right cut" of use cases. 
Here is a list of some well crafted thoughts:

- [How big or small should a Use Case Interactor be in Clean Architecture?](https://stackoverflow.com/questions/47934312/how-big-or-small-should-a-use-case-interactor-be-in-clean-architecture)
- [Do Interactors in clean architecture violate the Single Responsibility Principle?](https://softwareengineering.stackexchange.com/questions/364725/do-interactors-in-clean-architecture-violate-the-single-responsibility-princip) 
- [Who should handle combining interactors?](https://stackoverflow.com/questions/47868684/in-clean-mvp-who-should-handle-combining-interactors)

{% include series.html %}
