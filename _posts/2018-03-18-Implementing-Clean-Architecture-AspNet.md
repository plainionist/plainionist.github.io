---
layout: post
title: Implementing Clean Architecture - Are Asp.Net controllers "Clean"?
description: |
  In the Clean Architecture conrollers belong to the "interface adapters" and bridge between UI framework 
  and business rules. In Asp.Net controllers derive from Asp.Net base classes. 
  By that, are controllers in Asp.Net violating the Dependency Rule?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: JL0002
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.Presenters.AspNet.png" class="dynimg" title="Asp.Net in the context of Clean Architecture." alt="How do Asp.Net Controllers fit into the context of Clean Architecture? Do they belong to the interface adapter layer?"/>

In the [previous post](/Implementing-Clean-Architecture-Controller-Presenter/) I have discussed controllers and presenters.
I have shown you how I have implemented my controllers and presenters in the *[Athena](/Implementing-Clean-Architecture)* project.

I was quite happy with my design so far but there was one thing which puzzled me ...

In Asp.Net MVC a controller derives from ```System.Web.Mvc.Controller``` which creates a dependency from my controller to the Asp.Net 
framework. Taking the Dependency Rule strict that either means my design is invalid or my controller actually belongs to the 
"frameworks" circle.

In order to learn what others think about this design I have posted my question at 
[StackOverflow](https://stackoverflow.com/questions/48589192/dependency-from-gateway-to-framework-in-clean-architecture)
and had a discussion with [@herbertograca](https://herbertograca.com/2017/09/28/clean-architecture-standing-on-the-shoulders-of-giants/).

In this post I will share what I have learned and how I solved the puzzle ...

<!--more-->

## Status quo

In the [previous post](/Implementing-Clean-Architecture-Controller-Presenter/) I have shown you this picture:

<img src="{{ site.url }}/assets/clean-architecture/User.Interactor.Flow.png" class="dynimg" title="Control flow from user through controller, interactor and presenter." alt="The user interacts with the view. The view passes a request (defined in the interface adapter layer) to the controller which converts it into a request model defined in the use case layer. The interactor takes the request model though a input port and produces a response model which gets passed through an output port to the presenter. The presenter converts the response model into a response object defined in the interface adapters layer to the view. The view renders the response for the user"/>

It illustrates very clearly how the control flow from the user to the interface adapter and back to the user works
in the Clean Architecture. It also shows how code dependencies should be organized.
 
I have then shown you some F# code which basically implemented this picture. In the end of the post I had request
and response objects, request and response models, a controller and a presenter.

What I have not shown you explicitly were the dependencies the code had to the Asp.Net framework. Let me do this now:

<img src="{{ site.url }}/assets/clean-architecture/AspNet.Controllers.png" class="dynimg" title="Dependencies of the BacklogController" alt="The BacklogController is derived from the Asp.Net Controller class and so creates a dependency to the Asp.Net framework. The ReleaseBacklogViewModel uses SelectListItem from Asp.Net framework to create ComboBoxes with with Asp.Net helpers in the Razor view. By that it also creates a dependency to the Asp.Net framework."/>

*Note:* In this picture the BacklogPresenter is part of the BacklogController as it is still kind of "private" class controlled
by the BacklogController.

This picture clearly shows that on the one hand the control flow and the dependencies between my classes from 
*[Athena](/Implementing-Clean-Architecture)* project are pretty much conform with the Clean Architecture and the Dependency Rule.
On the other hand the color coding makes it obvious that the dependency between my classes and the Asp.Net classes breaks the
Dependency Rule.

I see basically three options to handle this situation ...

## Option 1: Keep the design and accept the violation of the Dependency Rule

Choosing this option means that I would continue implementing my Clean Architecture controllers as derived classes from Asp.Net Controller
and that I would continue using Asp.Net view model classes like ```SelectListItem```. I would continue ensuring that the control flow
and the dependencies between my own classes matches Clean Architecture and Dependency Rule. And of course I would have to accept this
(small) violation of the Dependency Rule.

This approach is a rather simple, pragmatic and convenient one. All the data conversion logic remains in the "interface adapters" layer
and I could continue using the benefits of Asp.Net framework.

But of course there are drawbacks. The more I use the convenience of the Asp.Net framework the more my code of course depends on the 
framework. If I would ever want to migrate to another framework - like Asp.Net Core or [Suave.IO](https://suave.io/) or 
[NancyFx](http://nancyfx.org/) - this would have quite some impact on my controller and presenter implementations. 

Another aspect would be testing. Today I don't do much testing on controller and presenter layer. As all my business logic is in the 
interactors I have focused most of my testing efforts on these interactors. Even if my controllers and presenters do data conversion
only there could be bugs and certainly testing this "logic" as well would be beneficial. But testing can become rather difficult if 
the controller utilizes many APIs from Asp.Net base class (e.g. Request and Session property). How do I fake all these APIs correctly? 

There is another observation I have made in my code base: my presenter logic tends to spread across multiple "helper" classes. As the
Razor engine supports executing C# code from the HTML templates it is quite convenient to add custom types to view models and have
some rendering functions which extract the relevant information and convert it into strings. This convenience made the view far less
"dumb" than it should be (Is the view calling the right function? Has the view model property the correct type to be passed to a 
function?).

## Option 2: Keep the design but move the code into the framework layer

The simplest solution towards a design which is conform with the Dependency Rule would be to move the ```BacklogController``` and the 
```ReleaseBacklogViewModel``` from the "interface adapters" layer into the "frameworks" layer. This would mean that all the data 
conversion logic goes into the "frameworks" layer and the "interface adapters" layer becomes rather empty.

Looking at the current project structure of the *[Athena](/Implementing-Clean-Architecture)* 

<img src="{{ site.url }}/assets/clean-architecture/Athena.Projects.4.png" class="dynimg" title="Clean Architecture conform projects with shared 'infrastructure'" alt="Still Clean Architecture: Two projects per core use case (one for business rules, one for the rest). A single separate project for entities. And one more shared project in the 'interface adapters' circle for shared infrastructure."/>

I would probably add a new layer of assemblies, e.g.: ```Athena.Backlog.AspNet.dll``` to make the design decision explicit in the code.

This approach is also simple, pragmatic and convenient. But having so much "application logic" in the "frameworks" layer makes the 
code base more ugly from my perspective.

And of course the other problems mentioned above about testing and complexity are not solved either.

## Option 3: Change the design and separate the concerns

The last and cleanest option I see would be to separate my controller and presenter logic from the Asp.Net framework.
A first attempt could look like this:

<img src="{{ site.url }}/assets/clean-architecture/AspNet.Controllers.Clean.png" class="dynimg" title="BacklogController separated from the Asp.Net controller" alt="The BacklogController does not derive from Asp.Net Controller any longer and contains most of the data conversion logic. BacklogAspNetController derives from Asp.Net Controller, converts data between Asp.Net and application and calls the BacklogController. Asp.Net dependencies are factored out of ReleaseBacklogViewModel into ReleaseBacklogAspNetViewModel"/>

The ```BacklogController``` would still contain most of the data conversion logic between view and use case interactor. The 
```BacklogAspNetController``` would be a very thin adapter between the Asp.Net framework and my application controller converting between
Asp.Net types and custom types. The ```ReleaseBacklogViewModel``` would not reference any Asp.Net types any longer but only primitive and
custom types. The ```BacklogAspNetController``` would then convert the ```ReleaseBacklogViewModel``` into a 
```ReleaseBacklogAspNetViewModel``` in case I would want to continue using the Asp.Net rendering helper functions in the Razor templates.
In turn, if I would stop using these functions the ```ReleaseBacklogAspNetViewModel``` would not be needed.

This approach adds some more classes to the design in order to make it conform with the Dependency Rule. 
It even does more than just that. It also addresses testability: adding tests for the data conversions is now pretty easy as this logic
has no dependency to the Asp.Net framework anymore. And this approach also addresses portability: If I would want to migrate to Asp.Net 
WebApi with pure HTML/JavaScript UI, my controller and presenter logic would be no longer impacted (ignoring the functions in the Razor
templates for a second).

> *Note:* There is still a small design flaw in the refactored design. The ```ReleaseBacklogViewModel``` returns 
> ```ScopedImprovement``` which is a class defined in the use case layer. Reusing this class in the presenter - something
> which Uncle Bob considers as cheating - looks very convenient initially. The consequence of this design that I need some logic
> in the view to do the job of the presenter and "render" the properties of ```ScopedImprovement``` which makes the view smart 
> again. But, in Clean Architecture, we want the view to be as dumb as possible so that there is no logic left which needs to be
> tested.
> 
> Of course the Asp.Net MVC framework didn't force me to choose this design but such frameworks provide some very convenient 
> "magic" to get things done "fast" which in the longer run can slow you down. That's why frameworks should be used with care.
> 
> I will later fix my design by simply using the helpers in the presenter directly instead of from the views, make the view model
> properties "stings only" and so the view dumb again.


### Separating controller and presenter - Reloaded

And there is even more. 

The [previous post](/Implementing-Clean-Architecture-Controller-Presenter/) I closed with an approach separating controller and presenter
in the context of Asp.Net MVC framework. This is the code I came up with:

```fsharp
type IOutputPort = 
    abstract HandleResponse : ReleaseBacklogResponse -> unit

type BacklogPresenter(callback) =
    interface IOutputPort with 
      member this.HandleResponse response = 
        {
            // data conversion logic skipped
        }
        |> callback

[<HttpPost>]
member this.Backlog (filter) =
    let request = { 
        // data conversion logic skipped
    }

    let mutable vm = null
    let presenter = new BacklogPresenter(fun x -> vm <- x) :> IOutputPort
        
    request 
    |> BacklogInteractor.GetScopedReleaseBacklog IoC.PlanningSerivce IoC.WorkitemRepository presenter

    vm
```

With that approach my controller still knows about the presenter. I still think this is not a big deal but let's see whether the new design
can also improve this aspect. Here is an alternative proposal:

```fsharp
type BacklogAspNetController() =
    inherit Controller()

    [<HttpPost>]
    member this.Backlog (filter) =
        let mutable vm = null
        let presenter = new BacklogPresenter(fun x -> vm <- x) :> IOutputPort
        let interactor = BacklogInteractor.GetScopedReleaseBacklog IoC.PlanningSerivce IoC.WorkitemRepository presenter
        
        filter 
        |> BacklogController.CreateBacklog interactor

        // TODO: convert ReleaseBacklogViewModel to ReleaseBacklogAspNetViewModel
        vm

module BacklogController =
    let CreateBacklog interactor filter =
        let request = { 
            // data conversion logic skipped
        }

        request |> interactor 
```

The ```BacklogPresenter``` is still injected into the use case interactor. This is now done in the ```BacklogAspNetController```.
The ```BacklogInteractor``` is then injected into the ```BacklogController```.

> *Note:* In F# we use [partial function application](https://fsharpforfunandprofit.com/posts/partial-application/) as a kind of 
> dependency injection. 

With this approach the application controller does not know any thing about the presenter and so get's even closer to the
Clean Architecture. The ```BacklogAspNetController``` does not only bridge between Asp.Net framework and application, it also 
takes care of wiring controller, presenter and interactor. Actually this should be the responsibility of the MAIN module but this 
is a topic for an own post.

### Project structure - Reloaded

As discussed in [this post](/Implementing-Clean-Architecture-Scream/) the current project structure of 
*[Athena](/Implementing-Clean-Architecture)* looks like this:

<img src="{{ site.url }}/assets/clean-architecture/Athena.Projects.4.png" class="dynimg" title="Clean Architecture conform projects with shared 'infrastructure'" alt="Still Clean Architecture: Two projects per core use case (one for business rules, one for the rest). A single separate project for entities. And one more shared project in the 'interface adapters' circle for shared infrastructure."/>

I have considered this as a good compromise between the need for borders in the architecture and pragmatism.
Now with the new insights on separating application controllers and Asp.Net framework, does the picture change?

[Previously](/Implementing-Clean-Architecture-Scream/) I haven't created projects in the "frameworks" layer as I haven't seen
much code which would go there. The simplest approach to incorporate the new insight would be to add a new project in the frameworks
layer per business aspect.

<img src="{{ site.url }}/assets/clean-architecture/Athena.Projects.5.png" class="dynimg" title="Clean Architecture conform projects with frameworks layer" alt="Clean Architecture with frameworks layer: Three projects per core use case (one for business rules, one for the adapters and one for Asp.Net depending code). A single separate project for entities. And one more shared project in the 'interface adapters' circle for shared infrastructure."/>

This would result in quite some projects. Can we simplify this?

The discussion in this post has shown how important borders are in an architecture. With that in mind a simplified project 
structure should draw a border between framework independent code and framework dependent code. That would give us

- one project per business aspect containing framework depending code
- one project per business aspect containing interface adapters and use cases
- single project containing entities

This approach clearly draws the two most important boarders between frameworks and application and between application
specific logic and enterprise business rules (entities).

## Which option should I choose?

Now that we have discussed multiple options with pros and cons - what would be the best option? As usual when talking about architecture
and design there is only one correct answer: It depends.

It depends on the concrete project. It depends on the size of the project. Borders are more important in a project with 500 developers 
than in a project with one developer. It depends on the future plans of the project. Portability to new frameworks is not equally important
for all project. It depends on the skills of your teams. More experienced developers see borders in the code even if they are not enforced
with separate projects.

For *[Athena](/Implementing-Clean-Architecture)* I currently still have Option 1 implemented in the code base but I plan to 
migrate to Option 3 as I anyhow want to migrate from Asp.Net MVC to Asp.Net WebApi with pure HTML/JavaScript UI. 

How would you decide?

{% include series.html %}
