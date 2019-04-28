---
layout: post
title: Implementing Clean Architecture - Retrospective
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---


Finally that we have discussed many conrete aspects on how to impl clean arch lets have one complete overview
how Athena finally looks like. where did we made compromises. where do we need imporve

## overview

draw a scetch of Athenas architecture
separate domain objects for backlog and burndown and governance

## One complete example

show how one scenario would be designed in clean-arch.
what is usecase, what is gateway, ...


show how the backend architecture has evolved with VueJS as FE
- dependencies are gone to MVC classes
- also update post "implementing-Clean-architecture-aspnet"
- also helper functions used by razor engine are gone
- also ScopedImprovement class is no longer used by view (no cheating - having proper string view models everywhere)

how would controller/presenter interaction change with async controller functions in Asp.Net Core?


==> draw a picture from Plainion.GraphViz


## Final Words

was it worth all the effort? what did i learn from that journey?

so now that i have to put so much additional effort into to appart from just make it work: why should i do so?

- is it the independency to frameworks?
- is it the focus on the usecases? (not driven by UI or DB - driven by the usecase of the domain)
- is it the "screaming" architecture?

- after having refactored quite some code from non clean arch to clean arch i can say: "it is unbelievable how the separation of
  interactor and presenter simplifies the design", "how magically entities occur - interactors have interactor specific requests and response. 
  everthing generic to these is entities!?"

- clean architecture supports greatly to defer decisions about UI, DB, cache, etc 
  we could just start with bdd tests as "driver" for use cases


PRO: defer decisions

may look like a lot of work but it pays of e.g. when testing (stable tests!) and replacing technology

adopting to changing requirements seems expensive but that is actually not the case. u have a nice robust safety net and all the DTOs and adpaters and data converters are pretty easy to change with less risk of breaking s.th. else


{% include series.html %}
