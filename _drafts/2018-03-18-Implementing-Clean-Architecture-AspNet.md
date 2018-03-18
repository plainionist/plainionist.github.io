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
I have shown how I have implemented my controllers and presenters in the *[Athena](/Implementing-Clean-Architecture)* project.

I was quite happy with my design so far but there was one thing which puzzled me ...

In Asp.Net MVC a controller derives from ```System.Web.Mvc.Controller``` which creates dependency from my controller to the Asp.Net 
framework. Taking the Dependency Rule strict either means this is an invalid design or my controller actually belongs to the 
"frameworks" circle.

In order to learn what others think about this situation I have posted a question at 
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

(Note: In this picture the BacklogPresenter is part of the BacklogController as it is still kind of "private" class controlled
by the BacklogController.)

This picture clearly shows that on the one hand the control flow and the dependencies between my classes from 
*[Athena](/Implementing-Clean-Architecture)* project are pretty much conform with the Clean Architecture and the Dependency Rule.
On the other hand the color coding makes it obvious that the dependency between my classes and the Asp.Net classes breaks the
Dependency Rule.

I see basically three options to handle this situation ...

## Option 1: Keep the design and accept the violation of the Dependency Rule

Choosing this options means that I would continue implementing my Clean Architecture controllers as derived classes from Asp.Net Controller
and that I would continue using Asp.Net view model classes like ```SelectListItem```. I would continue ensuring that the control flow
and the dependencies between my own classes matches Clean Architecture and Dependency Rule. And of course I would have to accept this
(small) violation of the Dependency Rule.

This approach is a rather simple, pragmatic and convenient one. All the data conversion logic remains in the "interface adapters" layer
and I could continue using the benefits of Asp.Net framework.

But of course there are drawbacks. The more I use the convenience of the Asp.Net framework the more my code of course depends on the 
framework. If I would ever want to migrate to another framework - like Asp.Net Core or [Suave.IO](https://suave.io/) or 
[NancyFx](http://nancyfx.org/) - this would have quite some impact on my controller and presenter implementation. 

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

This approach adds some more classes to the design in order to make it conform with the Dependency Rule again. 
It even does more than just that. It also addresses testability: adding tests for the data conversions is now pretty easy as this logic
has no dependency to the Asp.Net framework anymore. And this approach also addresses portability: If I would want to migrate to Asp.Net 
WebApi with pure HTML/JavaScript UI, my controller and presenter logic would be no longer impacted (ignoring the functions in the Razor
templates for a second).

### Separating controller and presenter - Reloaded

And there is even more. 

The [previous post](/Implementing-Clean-Architecture-Controller-Presenter/) I finished with an approach separating controller and presenter
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
    let CreateBacklog interacotr filter =
        let request = { 
            // data conversion logic skipped
        }

        request |> interactor 
```

describe the code!!


we would then just pass the presenter to the interactor and the interactor to the controller. this should actualy be in main module
but this is a topic for an own post.

rather small cosmetic change but makes the application controller even more Clean Architecture conform

### how to separate assemblies then?

so far i am going for this:

<img src="{{ site.url }}/assets/clean-architecture/Athena.Projects.4.png" class="dynimg" title="Clean Architecture conform projects with shared 'infrastructure'" alt="Still Clean Architecture: Two projects per core use case (one for business rules, one for the rest). A single separate project for entities. And one more shared project in the 'interface adapters' circle for shared infrastructure."/>

good balance between drawing borders and pragmatism

now with the new insight on separating controller and asp.net Fw further, how do we continue?

- safe choise: own "frameworks" assembly like "Athena.AspNet.dll"
- then gateways for controller
- then use cases and entities
- to simplify: keep gateways and framework together - requires discipline - in f# simpler


## Which option should I choose?

- how to decide? size of project?
- separation helps to migrate from classic "asp.net mvc" to "asp.net core mvc" and to "asp.net webapi" 
  or asp.net core webapi

==> ensure that logic is in interactors
==> avoid view helpers and have all logic in presenter (only return strings and other primitives)
==> then probably decide based on size and future of the project

For *[Athena](/Implementing-Clean-Architecture)* i currently still have Option 1 in the code base but i plan to migrate to Option 3 as i anyhow want
to migrate from Asp.Net MVC to Asp.Net WebApi + pure Html/Javascript UI. probably using vue.js.

{% include series.html %}
