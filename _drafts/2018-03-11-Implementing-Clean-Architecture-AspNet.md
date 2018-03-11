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



# Clean Architecture and Asp.Net

- https://stackoverflow.com/questions/48589192/dependency-from-gateway-to-framework-in-clean-architecture

- how to do it 100% correct
- how to decide? size of project?
- separation helps to migrate from classic "asp.net mvc" to "asp.net core mvc" and to "asp.net webapi" 
  or asp.net core webapi

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
