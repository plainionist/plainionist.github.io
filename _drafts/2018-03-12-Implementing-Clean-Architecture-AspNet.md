---
layout: post
title: Implementing Clean Architecture - Are Asp.Net controllers "clean" controllers?
description: |
  Controller and presenter in Uncle Bob's Clean Architecture belong to the "interface adapters" layer and bridge
  between the UI framework and the business rules located in the use case interactors. But in Asp.Net controllers
  derive from Asp.Net classes and so depend on the Asp.Net framework. Respecting the Dependency Rule: Are Asp.Net
  controllers valid contollers in the context of Clean Architecture?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.Presenters.AspNet.png" class="dynimg" title="Asp.Net in the context of Clean Architecture." alt="How do Asp.Net Controllers fit into the context of Clean Architecture? Do they belong to the interface adapter layer?"/>

In the [previous post](/Implementing-Clean-Architecture-Controller-Presenter/) I have discussed controllers and presenters.
I have shown how I have implemented my controllers and presenters in the *[Athena](/Implementing-Clean-Architecture)* project.

In general I was quite happy with my design but there was one thing which puzzled me:

a controller in the asp.net world derives from a asp.net fw class which creates a dependency from my controller to 
asp.net fw. taking the dependency rule strict means
- this is either an invalid design
- or my controller actually belongs to the fw layer

In order to learn what others think about it i have posted a question at 
[StackOverflow](https://stackoverflow.com/questions/48589192/dependency-from-gateway-to-framework-in-clean-architecture)
and had a discussion with [](https://herbertograca.com/2017/09/28/clean-architecture-standing-on-the-shoulders-of-giants/)

In this post i will share what i have learned and how i solved the puzzle ...

<!--more-->

## From control flow ...

In the [previous post](/Implementing-Clean-Architecture-Controller-Presenter/) I have shown you this picture:

<img src="{{ site.url }}/assets/clean-architecture/User.Interactor.Flow.png" class="dynimg" title="Control flow from user through controller, interactor and presenter." alt="The user interacts with the view. The view passes a request (defined in the interface adapter layer) to the controller which converts it into a request model defined in the use case layer. The interactor takes the request model though a input port and produces a response model which gets passed through an output port to the presenter. The presenter converts the response model into a response object defined in the interface adapters layer to the view. The view renders the response for the user"/>

It illustrates very clearly how the control flow from the user to the interface adapter and back to the user works
in the Clean Architecture. It also shows how code dependencies should be organized.
 
I have then shown you some F# code which basically implemented this picture. In the end of the post i had request
and response objects, request and response models, a controller and a presenter.

What I have not shown you explicitly were the dependencies the code had to the Asp.Net framework. Let me do this now


PIC today
(i abstracted backlogpresenter away because it is actually still kind of private class as known and called by controller.
controller still in control flow)

This picture makes it rather obvious: control flow and dependencies between my classes are pretty much conform with
Clean Architecture - the dependencies from my code to the Asp.Net framework are breaking the Dependency Rule.

Now I basically see three options ...

## 1. Keep the design and accept the violation of the Dependency Rule

- implement clean architecture controller as asp.net controller
- fine with control flow and my class dependencies
- accept violation by binding to asp.net in the interface adapter layer
- violation of dependency rule means data mapping logic in controller/presenter is impacted
  when switching to new framework
- might be a cheap pragmatic decision where u would benefit most from framework convenience
- ignore the small dependency to FW as s.th. which can be easily refactored e.g. i would want to migrate to
  Asp.Net Core WebApi 
- may lead to presenter code being elsewhere - ViewAPI!!!

## 2. keep the design but move the code into the framework layer

- accept that interface adapter layer is empty with that aspect
- maybe moving to a different assembly?

PIC packaging athena
- for athena i would at least rename the gateway assembly
- basically same pros and cons as 1. but just make it explicit in the code (namespaces) and package structure

## 3. separate the concerns

PIC separation

- Asp.Net controllers as thin adapter
- my controller and presenter about converting DTOs. this logic not impacted when making even more dramatic switch 
  from Asp.Net MVC to WebApi with Html5/JavaScript FE

CODE even now my controller does not need to know about the presenter
just the asp.net controller just wires things up 

### how to separate assemblies then?

so far i am going for this:

PIC

good balance between drawing borders and pragmatism

now with the new insight on separating controller and asp.net Fw further, how do we continue?

- safe choise: own "frameworks" assembly like "Athena.AspNet.dll"
- then gateways for controller
- then use cases and entities
- to simplify: keep gateways and framework together - requires discipline - in f# simpler


## which way to go?

- how to decide? size of project?
- separation helps to migrate from classic "asp.net mvc" to "asp.net core mvc" and to "asp.net webapi" 
  or asp.net core webapi

==> ensure that logic is in interactors
==> avoid view helpers and have all logic in presenter (only return strings and other primitives)
==> then probably decide based on size and future of the project



{% include series.html %}
