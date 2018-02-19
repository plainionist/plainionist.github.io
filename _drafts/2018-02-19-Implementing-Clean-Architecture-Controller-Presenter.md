---
layout: post
title: Implementing Clean Architecture - Of controllers and presenters
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.Presenters.png" class="dynimg"/>

Last time we discussed about use cases and interactors and stopped with the question 
"Which role is than actually left to the controller and presenter?".

In this post I will take this question up and dive deeper into the world of controllers and presenters
in the context of the Clean Architecture.

Read on!

<!--more-->

## Definitions

Let me again first look for some definitions ... 

... and what would be a more classical context to lookup a definition for 'controller' than the Model-View-Controller (MVC) pattern.

[MVC (Wikipedia)](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller):

> The controller is responsible for responding to the user input and perform interactions on the data model objects. 
> The controller receives the input, it validates the input and then performs the business operation that modifies the state of the data model.

.. and the counterpart for the 'presenter' would be of course the Model-View-Presenter (MVP) pattern.

[MVP (Wikipedia)](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter):

> The presenter acts upon the model and the view. It retrieves data from repositories (the model), and formats it for display in the view.

Well, this is about MVC and MVP - but how do the responsibilities change in the Clean Architecture?

## Clean Architecture 

In the MVC/MVP patterns the controller/presenter contains most of the business logic. In the Clean Architecture all of the 
business logic goes either into an use case interactor or an entity (we will talk about entities later).

I earlier showed you this picture:

<img src="{{ site.url }}/assets/clean-architecture/Interactor.Controller.Presenter.png" class="dynimg"/>

From that we can see already that controllers and presenters live in the "interface adpaters" circle (green color) 
which gives a strong hint: they are adapters only - and we don't want to have adapers much of logic right? 
Adapters just map from one world to another.

Let me zoom a little bit out of the picture to get the full context.

<img src="{{ site.url }}/assets/clean-architecture/User.Interactor.Flow.png" class="dynimg"/>

So what do we see here? Following the purple arrow we can see the control flow of a user interaction with 
a system:

1. The user interacts with the view.
2. The view creates a request (object) which is passed to the controller.
3. The controller converts the request into a request model and passes it to the use case interactor through
   its input port.
4. The use case interactor processes the request model and creates a response model which is passed through the 
   output port to the presenter.
5. The presenter converts the response model to view model which is then passed to the view.
6. The user sees the result of his interaction in the view.

But there is more to be explored. We see two vertical boundaries separating the picture into three sections:

- The left (blue) section where the frameworks live (e.g. an HTML/Javascript view)
- The middle (green) section where the adapters live (controllers and presenters)
- The right (red) section where the business logic (use case interactors) live

The controller and the presenter are located in the middle of this picture and they appear as a bridge between 
the user's world (the view) and the business logic's world (interactors). 

> **Can you see the Dependency Rule?**
>
> Let's look even closer at the picture for a moment. We can see quite some arrows between the different boxes.
> These arrows represent code level dependencies which means the code where an arrow starts knows about the code 
> the arrow is pointing to. All these arrows fulfill the Dependency Rule as all arrows crossing  the boundaries 
> are always going from left to right - from the framework circle to the interface adapters circle to the 
> use case circle.
>
> You probably have noticed that there are two types of arrows: The open arrow indicates a "uses" relationship,
> the closed one indicates a "implements" or "extends" relationship. We will look into this difference in more detail soon.

Now that we know how the controllers and presenters fit into the complete picture of the Clean Architecture 
let's deep dive into their responsiblities.

## What is the role of the controller? 

The controller takes user input, converts it into the request model defined by the use case interactor and passes 
this to the same.

The request object accepted by the controller is also defined by the controller. We do NOT want the controller to depend 
on the view or types defined in the framework circle. 

Such request objects are usually simple data transfer objects (DTO). Depending on the view technology a
request object may contain typed information (e.g. WPF) or just strings (e.g. HTML). It is the role of the controller
to convert the given information into a format which is most convenient for and defined by the use case interactor.
For that the controller may have some simple if-then-else or parser logic but we do not want to have any processing logic
inside the conroller.

Finally the controller simply calls an API on the use case interactor to trigger the processing.

## What is the role of the presenter? 

[Uncle Bob says](/Clean-Architecture/):
"
The job of the Presenter is to repackage the OutputData into viewable form as the ViewModel, 
which is yet another plain old Java object. The ViewModel contains mostly Strings and flags that the 
View uses to display the data. Whereas the OutputData may contain Date objects, the Presenter will load the 
ViewModel with corresponding Strings already formatted properly for the user. The same is true of Currency objects or 
any other business-related data. Button and MenuItem names are placed in the ViewModel, as are flags that tell the 
View whether those Buttons and MenuItems should be gray.
"

Is there anything to add? ;-)

Maybe one short word on testing: No matter which view technology you use (WPF, Html, etc) the view will always be hard
to test; at least harder than the code not depending on it. The job of the presenter is to prepare the view model that well
that the view becomes that dumb that testing the view unnecessary.

Finally, after this longer part of theory you probably want to see some code, right? ;-)

## How do I implement controllers and presenters?

As you probably remember from the [previous post](/Implementing-Clean-Architecture-UseCases/) that for the *Athena* 
project I initially decided to have the controller and the presenter in one class as this is a pragmatic solution in 
the context of Asp.Net MVC. Let me first describe how I have implemented controllers and presenters in this context and then 
how it could be done more according to what we have discussed so far.

You probably remember the "Show the ranked backlog with cut-lines" use case from the 
[previous post](/Implementing-Clean-Architecture-UseCases/):

<img src="{{ site.url }}/assets/clean-architecture/backlog.png" class="dynimg"/>

I showed you how I implemented the business rules with multiple interactors. Now let me show the corresponding
controller/presenter (Even if you are not used to F# I am pretty sure you can still get the key points from the code).

```F#
    member this.Backlog () =
        this.Backlog(new BacklogFilter())

    [<HttpPost>]
    member this.Backlog (filter) =
        let request = { 
            AssignedTo = filter.AssignedTo |> toFilter
            Category = filter.Category |> categoryOptionToFilter
            BacklogSet = filter.BacklogSet |> backlogSetOptionToFilter
            ClinicalResponsible = filter.ClinicalResponsible |> toFilter
            ArchitectureResponsible = filter.ArchitectureResponsible |> toFilter
        }

        let response = request |> BacklogInteractor.GetScopedReleaseBacklog IoC.PlanningSerivce IoC.WorkitemRepository

        let viewModel = {
            Categories = response.Categories |> Seq.map categoryToOption |> Mvc.Selects.Create
            BacklogSets = response.BacklogSets |> Seq.map backlogSetToOption |> Mvc.Selects.Create
            ClinicalResponsibles = response.ClinicalResponsibles |> Mvc.Selects.Create
            ArchitectureResponsibles = response.ArchitectureResponsibles |> Mvc.Selects.Create
            AssignedTo = response.AssignedTo |> Mvc.Selects.Create

            Workitems = response.Workitems

            RemainingEffort = response.RemainingEffort |> formatEffort
            RemainingAvailabilty = response.RemainingAvailabilty |> formatAvailability

            Filter = filter
        }
        
        this.View(viewModel)
```

The ```GetBacklog``` method takes a filter object as parameter which contains the currently selected values in to combo boxes of the page.
I convert this request object first into the request model demanded by the interactor and pass it to the same. 

```F#
/// Request object passed to the controller
type BacklogFilter() =
    member val Category = Mvc.Selects.All with get,set
    member val BacklogSet = Mvc.Selects.All with get,set
    member val ClinicalResponsible = Mvc.Selects.All with get,set
    member val ArchitectureResponsible = Mvc.Selects.All with get,set
    member val AssignedTo = Mvc.Selects.All with get,set

/// Request model demanded by the interactor
type ReleaseBacklogRequest = {
    Category : ImprovementCategory option Filter
    BacklogSet : BacklogSet option Filter
    ClinicalResponsible : string Filter
    ArchitectureResponsible : string Filter
    AssignedTo : string Filter
}
```

The interactor returns a response model which contains all the data needed to display the page. As this data is still typed 
and in a form most convenient for the interactor I convert it into a view model as a last step. The view model contains all the 
data in a format most convenient for the view.

```F#
/// Response model returned by the interactor
type ReleaseBacklogResponse = {
    Categories : ImprovementCategory option list
    BacklogSets : BacklogSet option list
    ClinicalResponsibles : string list
    ArchitectureResponsibles : string list
    AssignedTo : string list

    Workitems : ScopedImprovement list

    RemainingEffort : float<PD>
    RemainingAvailabilty : float<Netto PD>
}

/// View model passed to the view
type ReleaseBacklogViewModel = {
    Categories : SelectListItem list
    BacklogSets : SelectListItem list
    ClinicalResponsibles : SelectListItem list
    ArchitectureResponsibles : SelectListItem list
    AssignedTo : SelectListItem list

    Workitems : ScopedImprovement list

    RemainingEffort : string
    RemainingAvailabilty : string

    Filter : BacklogFilter
}
```

I also have some small helper functions defined - which is much more convenient and concise in F# than in C# ;-) - which I use for the
forward and backward conversion of types:

```F#
    let toFilter value = if value = Mvc.Selects.All then Any else value |> Exactly

    let categoryToOption value = ...

    let categoryOptionToFilter value = ...

    let backlogSetToOption = 
        function
        | Some(x) -> x |> Unions.toString
        | _ -> String.Empty

    let backlogSetOptionToFilter = 
        function
        | Mvc.Selects.All -> Any
        | x -> x |> Unions.fromString |> Exactly

    let formatEffort = sprintf "%.2f"
    
    let formatAvailability = sprintf "%.2f"
```

*Note:* I have kept many details in the code which I have not explained in detail because I think those are not relevant to the discussion
in this post. If you still want to know more about these details please drop my a line.

Quite some code! But in the end my implementation of the controller-presenter-hybrid is very simple. Now here comes the 1000$ question:

&#8680; What would Uncle Bob say to this pragmatism? ;-)

I don't know. As mentioned already, personally I consider my approach as a pragmatic and simple way to combine Asp.Net MVC with the
Clean Architecture. So I am happy for now ...

But let's assume I want to convert *Athena* into a more modern Single Page Application (SPA). The frontend would be pure HTML5 with some 
nice [Javascript framework](http://vuejs.org) and the backend would Asp.Net Core or Suave.IO only knowing about JSON.

How would I implement a separation between controller and presenter then?

## Separating controller and presenter

start simple: controller gets all code before we call the interactor, presenter gets the code after we got data from interactor.

CODE controller

CODE presenter

now still this is  not exactly matchting the picture we started with. lets look at it again

<img src="{{ site.url }}/assets/clean-architecture/User.Interactor.Flow.png" class="dynimg"/>

we have a controller - we have a presenter.
we have request (give the name) and a request model (give the name).
we have a response model (give the name) and a viewmodel (give the name).

where are the ports?

## how to impl an input port?

this is an interface implemented by an use case interactor.
do i need to have a dedicated interface? i dont think so. i am pretty fine with calling a method on the usecase interator directy.
do i intend to replace the interactor implementation later? NO - i would just change it. 
do i have other reasons to hide the interactor implementation? NO - even if i would have further APIs on the interactor for 
testing i would make them internal ... controllers i would anyhow put in other projects than interactors to keep dependencies to third party clean.
maybe i have an interactor providing multiple public methods but not all public methods should be accessible to all controllers?
then we should have separate interfaces - one per "scenario".

"
The FinancialReportRequester interface serves a different purpose. It is there to protect the FinancialReportController from knowing too much about the internals of the Interactor. If that interface were not there, then the Controller would have transitive dependencies on the FinancialEntities. Transitive dependencies are a violation of the general principle that software entities should not depend on things they don’t directly use. We’ll encounter that principle again when we talk about the Interface Segregation Principle and the Common Reuse Principle. So, even though our first priority is to protect the Interactor from changes to the Controller, we also want to protect the Controller from changes to the Interactor by hiding the internals of the Interactor.

Martin, Robert C.. Clean Architecture: A Craftsman's Guide to Software Structure and Design (Robert C. Martin Series) (pp. 74-75). Pearson Education. Kindle Edition. 
"


==> method call

## how would i implement an output port?


==> interface + callback (more details in the book?) interface defined by the use case - most convenient for the use case
    containing the output data defined by the use case

==> again: what about just returning data?


## How to invert to control flow?

now controller injects presenter into interactor. but still controller asks for view model form presenter - due to nature 
of asp.net mvc fremwwork

can we fix that?



for now we just did it in the controller? is this correct? ... next post.

But who wires all this up? If the controller and presenter are different classes: who is injecting the
presenter into the interactor? This will be answered in one of the next posts about "The Main".




## How do others think about controllers and presenters

During research for this post I came along many other discussions about controllers and presenters in the Clean Architecture.
Here are some of them:

- [Use case containing the presenter or returning data?](https://softwareengineering.stackexchange.com/questions/357052/clean-architecture-use-case-containing-the-presenter-or-returning-data)
- [What are the jobs of presenter?](https://stackoverflow.com/questions/46510550/clean-architecture-what-are-the-jobs-of-presenter)


 https://stackoverflow.com/questions/45921928/use-case-containing-the-presenter-or-returning-data

{% include series.html %}
