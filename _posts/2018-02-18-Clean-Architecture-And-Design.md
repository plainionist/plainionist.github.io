---
layout: post
title: Clean Architecture and Design
description: Uncle Bobs famous talk about Clean Architecture, its history and how it evolved.
tags: [clean-architecture]
excerpt_separator: <!--more-->
lint-nowarn: JL0003
---

While preparing my next post on [Implementing Clean Architecture](/Implementing-Clean-Architecture) I watched again
Uncle Bobs famous talk on [Clean Architecture and Design](https://www.youtube.com/watch?v=Nsjsiz2A9mg).

I know it is more than three years old but if you are interested in Clean Architecture you should take your time and 
watch it. Uncle Bob explains nicely - and a little bit funny as usual ;-) - what the Clean Architecture is about, 
how the puzzle is built up and why it has to be like this.

And as a summary for this video and as an outlook to my next post here is the key message:

<img src="{{ site.url }}/assets/clean-architecture/User.Interactor.Flow.png" class="dynimg" title="Control flow from user through controller, interactor and presenter." alt="The user interacts with the view. The view passes a request (defined in the interface adapter layer) to the controller which converts it into a request model defined in the use case layer. The interactor takes the request model though a input port and produces a response model which gets passed through an output port to the presenter. The presenter converts the response model into a response object defined in the interface adapters layer to the view. The view renders the response for the user"/>

<!--more-->

