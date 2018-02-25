---
layout: post
title: Implementing Clean Architecture - Of controllers and presenters
description: What is the role of the controller in Clean Architecture? What is the role of the presenter in the Clean Architecture? Do I need to separate both?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.Presenters.png" class="dynimg"/>

 [Last time](/Implementing-Clean-Architecture-UseCases/) we discussed about use cases and interactors and 
 stopped with the question: "Which role is than actually left to the controller and presenter?"

In this post I will take this question up and dive deeper into the world of controllers and presenters
in the context of the Clean Architecture.

Read on!

<!--more-->

## Definitions

Let me again first look for some definitions ... 

... and what would be a better place to lookup a definition for 'controller' than the Model-View-Controller (MVC) pattern.

[MVC (Wikipedia)](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller):

> The controller is responsible for responding to the user input and perform interactions on the data model objects. 
> The controller receives the input, it validates the input and then performs the business operation that modifies the state 
> of the data model.

.. and the counterpart for the 'presenter' would be of course the Model-View-Presenter (MVP) pattern.

[MVP (Wikipedia)](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter):

> The presenter acts upon the model and the view. It retrieves data from repositories (the model), and formats it for display 
> in the view.

Well, this is about MVC and MVP - but how do the responsibilities change in the Clean Architecture?

## Clean Architecture 

In the MVC/MVP patterns the controller/presenter contains most of the business logic. In the Clean Architecture all of the 
business logic goes either into an use case interactor or an entity (we will talk about entities later).

I earlier showed you this picture:

<img src="{{ site.url }}/assets/clean-architecture/Interactor.Controller.Presenter.png" class="dynimg"/>

From that we can already see that controllers and presenters live in the "interface adapters" circle (green) 
which gives a strong hint: they are adapters only - and we don't want to have adapters much of logic right? 

Let me zoom a little bit out of the picture to get the full context.

<img src="{{ site.url }}/assets/clean-architecture/User.Interactor.Flow.png" class="dynimg"/>

What do we see here? Following the purple arrow we can see the control flow of a user interacting with 
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

Now that we know how the controllers and presenters fit into the complete picture of the Clean Architecture 
let's deep dive into their responsibilities.

## What is the role of the controller? 

The controller takes user input, converts it into the request model defined by the use case interactor and passes 
this to the same.

The request object accepted by the controller is defined by the controller. We do NOT want the controller to depend 
on the view or types defined in the framework circle. 

Such request objects are usually simple data transfer objects (DTO). Depending on the view technology a
request object may contain typed information (e.g. WPF) or just strings (e.g. HTML). It is the role of the controller
to convert the given information into a format which is most convenient for and defined by the use case interactor.
For that the controller may have some simple if-then-else or parser logic but we do not want to have any processing logic
inside the controller.

Finally the controller simply calls an API on the use case interactor to trigger the processing.

## What is the role of the presenter? 

[Uncle Bob says](/Clean-Architecture/):
> The job of the Presenter is to repackage the OutputData into viewable form as the ViewModel, 
> which is yet another plain old Java object. The ViewModel contains mostly Strings and flags that the 
> View uses to display the data. Whereas the OutputData may contain Date objects, the Presenter will load the 
> ViewModel with corresponding Strings already formatted properly for the user. The same is true of Currency objects or 
> any other business-related data. Button and MenuItem names are placed in the ViewModel, as are flags that tell the 
> View whether those Buttons and MenuItems should be gray.

Is there anything to add? ;-)

Maybe one short word on testing: No matter which view technology you use (WPF, HTML, etc) the view will always be hard
to test; at least harder than the code not depending on it. The job of the presenter is to prepare the view model that well
that the view becomes that dumb that testing the view becomes unnecessary.

Finally, after this longer part of theory you probably want to see some code, right? ;-)

## How do I implement controllers and presenters?

As you probably remember from the [previous post](/Implementing-Clean-Architecture-UseCases/), I decided for the *Athena* 
project to have the controller and the presenter in one class as this is a pragmatic solution in 
the context of Asp.Net MVC. Let me first describe how I have implemented controllers and presenters in this context and then 
how it could be done more according to what we have discussed so far.

You probably remember the "Show the ranked backlog with cut-lines" use case from the 
[previous post](/Implementing-Clean-Architecture-UseCases/):

<img src="{{ site.url }}/assets/clean-architecture/backlog.png" class="dynimg"/>

I showed you how I implemented the business rules with multiple interactors. Now let me show the corresponding
controller/presenter (Even if you are not an F# expert, I am pretty sure you can still get the key points from the code).

```fsharp
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

    let response = 
        request 
        |> BacklogInteractor.GetScopedReleaseBacklog IoC.PlanningSerivce IoC.WorkitemRepository

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

The ```GetBacklog``` method takes a filter object as parameter which contains the currently selected values in to 
combo boxes of the page. It converts this request object first into the request model demanded by the interactor and 
pass it to the same. 

```fsharp
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
and in a form most convenient for the interactor it will be converted into a view model as a last step. The view model 
contains all the data in a format most convenient for the view.

```fsharp
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

I also have some small helper functions defined - which is much more convenient and concise in F# than in C# ;-) - which I
use for the forward and backward conversion of types:

```fsharp
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

*Note:* I have kept some details in the code which I have not explained in detail so far because think those are not that 
relevant to the discussion in this post. If you still want to know more about these details please drop my a line.

Quite some code! But in the end my implementation of the controller-presenter-hybrid is very simple. Now here comes the most 
important question of this post:

&#8680; What would Uncle Bob say to this pragmatism? ;-)

I don't know. As mentioned already, personally I consider my approach as a pragmatic and simple way to combine Asp.Net MVC 
with the Clean Architecture. So I am happy for now ...

But let's assume I would now want to separate controller and presenter. How would I implement that separation?

## Separating controller and presenter

As a first simple step I would just keep all the code before the call to the interactor in the controller and move everything 
after that call into the presenter.

```fsharp
module BacklogPresenter =
    let CreateViewModel filter response =
      {
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

[<HttpPost>]
member this.Backlog (filter) =
    let request = { 
        AssignedTo = filter.AssignedTo |> toFilter
        Category = filter.Category |> categoryOptionToFilter
        BacklogSet = filter.BacklogSet |> backlogSetOptionToFilter
        ClinicalResponsible = filter.ClinicalResponsible |> toFilter
        ArchitectureResponsible = filter.ArchitectureResponsible |> toFilter
    }

    let response = 
        request 
        |> BacklogInteractor.GetScopedReleaseBacklog IoC.PlanningSerivce IoC.WorkitemRepository

    let viewModel = response |> BacklogPresenter.CreateViewModel filter
        
    this.View(viewModel)
```

At least my design is now closer to Single Responsibility Pattern (SRP) - the controller as well as the presenter has only one
reason to change. But still not matching the picture we started with.

<img src="{{ site.url }}/assets/clean-architecture/User.Interactor.Flow.png" class="dynimg"/>

We have a controller and an independent presenter now. We have a request object, a request model, a response model and a view model.

But where are the ports?

## How do I implement an input port?

You probably have already noticed that there are two types of arrows in the picture above. The open arrow indicates a 
"uses" relationship and the closed one indicates an "implements" or "extends" relationship. Interestingly the two closed arrows
are both pointing to a port ...

Obviously, an input port has to be an interface or an (abstract) class to be implemented by the interactor with at least one 
API called by the controller to pass the request model and to trigger the processing.

So far, I have not used any interfaces on the interactors. I have just called APIs on the interactors directly. Do I really need
an interface - another abstraction? 

I could see two benefits in introducing interfaces:

1. Testing. If I would want to test the controller without the interactor I would need an abstraction to replace the actual 
   implementation with a stub or a mock
2. Interface Segregation. If multiple controllers would use different APIs on the same interactor separate input port interfaces 
   would ensure that each controller only knows the parts about the interactor it needs to know.

I should probably consider a refactoring step ...

## How do I implement an output port?

Now that we know what an input port is we can conclude what an output port will be.

An output port needs to be an interface or an (abstract) class implemented by the presenter with at least one API called by the 
interactor to pass the response model.

Implementing an output port finally means to invert the control flow. Instead of getting a return value from the interactor and 
asking the presenter to convert it into a view model, the interactor now would "actively" pass the response model through the 
output port to the presenter. The interactor is now not only controlling HOW the response looks like, it also controls WHEN it 
will be available.

## How do I invert the controll fow?

Here is my proposal

```fsharp
type IOutputPort = 
    abstract HandleResponse : ReleaseBacklogResponse -> unit

type BacklogPresenter(callback) =
    interface IOutputPort with 
      member this.HandleResponse response = 
        {
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
        |> callback

[<HttpPost>]
member this.Backlog (filter) =
    let request = { 
        AssignedTo = filter.AssignedTo |> toFilter
        Category = filter.Category |> categoryOptionToFilter
        BacklogSet = filter.BacklogSet |> backlogSetOptionToFilter
        ClinicalResponsible = filter.ClinicalResponsible |> toFilter
        ArchitectureResponsible = filter.ArchitectureResponsible |> toFilter
    }

    let presenter = new BacklogPresenter(this.View) :> IOutputPort
        
    request 
    |> BacklogInteractor.GetScopedReleaseBacklog IoC.PlanningSerivce IoC.WorkitemRepository presenter
```

The ```BacklogPresenter``` implements the output port interface and also gets a callback to finally pass the view model 
to the Asp.Net MVC controller to trigger the rendering of the view. The presenter is passed to the interactor so that
the interactor can pass the response to the presenter through the output port as discussed above.

DONE! ... Done? Almost ... 

There is just one tiny issue. The code does not compile. An Asp.Net MVC controller method has
to return an object of type ```ActionResult```. Unfortunately I have currently no idea how I could implement an output port
and still return something from the controller method. Maybe I find a solution in one of the next posts.
Until then I will stick to my pragmatic solution I started with ;-)


## How do others think about controllers and presenters

During research for this post I came along many other discussions about controllers and presenters in the Clean Architecture.
Here are some of them:

- [Use case containing the presenter or returning data?](https://softwareengineering.stackexchange.com/questions/357052/clean-architecture-use-case-containing-the-presenter-or-returning-data)
- [What are the jobs of presenter?](https://stackoverflow.com/questions/46510550/clean-architecture-what-are-the-jobs-of-presenter)

{% include series.html %}
