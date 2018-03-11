---
layout: post
title: Implementing Clean Architecture - Where to put Asp.Net controllers?
description: |
  Controller and presenter in Uncle Bob's Clean Architecture belong to the "interface adapters" layer and bridge
  between the UI framework and the business rules located in the use case interactors. But an Asp.Net controller
  depends on the Asp.Net framework. Respecting the Dependency Rule - Is the controller still a valid adapter?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.Presenters.png" class="dynimg" title="Deep diving controllers and presenters." alt="From Clean Architectures circles lets take out the 'interface adapters' one and deep dive into controllers and presenters."/>

[Last time](/Implementing-Clean-Architecture-Controller-Presenter/) we discussed about controllers and presenters.
I have shown how I have implemented controllers and presenters in the context of Asp.Net and F#.

I have left one question open: how to revert the control flow.
but there is one more was one more thing which puzzled me:
a controller in the asp.net world derives from a asp.net fw class. which creates a dependency from my controller to 
asp.net fw. taking the dependency rule strict means:
- this is either an invalid design
- or my controller actually belongs to the fw layer

i posted a question here
- https://stackoverflow.com/questions/48589192/dependency-from-gateway-to-framework-in-clean-architecture
and started a discussion with
https://herbertograca.com/2017/09/28/clean-architecture-standing-on-the-shoulders-of-giants/

in this post i am showing how i solved the puzzle ...

Read on!

<!--more-->

in the previous post i have shown u this picture

<img src="{{ site.url }}/assets/clean-architecture/User.Interactor.Flow.png" class="dynimg" title="Control flow from user through controller, interactor and presenter." alt="The user interacts with the view. The view passes a request (defined in the interface adapter layer) to the controller which converts it into a request model defined in the use case layer. The interactor takes the request model though a input port and produces a response model which gets passed through an output port to the presenter. The presenter converts the response model into a response object defined in the interface adapters layer to the view. The view renders the response for the user"/>

and then a lot of f# code which nicely illustrates how the code is structured and how the controll flow is 
realized in code - what it does not show - also be cause of the conciseness of f# - which dependencies the code has.
look at this picture

[class picture showing all dependencies]


# Clean Architecture and Asp.Net

- https://stackoverflow.com/questions/48589192/dependency-from-gateway-to-framework-in-clean-architecture

- how to do it 100% correct
- how to decide? size of project?
- separation helps to migrate from classic "asp.net mvc" to "asp.net core mvc" and to "asp.net webapi" 
  or asp.net core webapi

PIC today
(i abstracted backlogpresenter away because it is actually still kind of private class as known and called by controller.
controller still in control flow)

PIC separation


CODE even now my controller does not need to know about the presenter
just the asp.net controller just wires things up 



## how to separate assemblies then?

so far i am going for this:

PIC

good balance between drawing borders and pragmatism

now with the new insight on separating controller and asp.net Fw further, how do we continue?

- safe choise: own "frameworks" assembly like "Athena.AspNet.dll"
- then gateways for controller
- then use cases and entities
- to simplify: keep gateways and framework together - requires discipline - in f# simpler






{% include series.html %}
