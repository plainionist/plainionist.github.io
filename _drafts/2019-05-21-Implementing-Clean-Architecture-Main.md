---
layout: post
title: Implementing Clean Architecture - The dirty MAIN
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---

(IMAGE of a cristal ball)

now that we have covered all layers - how do i wire up all the things? after all the abstractions and interfaces - the magic
has to happen somewhere

<!--more-->

## The "Clean" approach

- the main component is the place where all details are wired up

"
In every system, there is at least one component that creates, coordinates, and oversees the others. I call this component Main.

The Main component is the ultimate detail—the lowest-level policy. It is the initial entry point of the system. 
Nothing, other than the operating system, depends on it. Its job is to create all the Factories, Strategies, and
other global facilities, and then hand control over to the high-level abstract portions of the system. It is in
this Main component that dependencies should be injected by a Dependency Injection framework. Once they are
injected into Main, Main should distribute those dependencies normally, without using the framework. Think
of Main as the dirtiest of all the dirty components.

The point is that Main is a dirty low-level module in the outermost circle of the clean architecture. It loads everything 
up for the high level system, and then hands control over to it.

For example, you could have a Main plugin for Dev, another for Test, and yet another for Production. You could also have a
Main plugin for each country you deploy to, or each jurisdiction, or each customer.

Instead, you can use Spring to inject dependencies into your Main component. 
It’s OK for Main to know about Spring since Main is the dirtiest, lowest-level component in the architecture.
"

## The "pragmatic" approach

i tend to be pragmatic: as i am in F# and everything is just a function call i also just call the functions of interactors
everywhere.

i just created interfaces (actually records of functions ;-)) for the adapters
that means i can test entities and use case interactors easily and isolated.

testing controllers and presenters and any other kind of adapter is more difficult.
current strategy: as less logic as possible to avoid testing at all.

Asp.Net MVC does not support dependency injection out of the box.
so i just sticked to "Service locator pattern" ... which i personally actually dont like as this allows every code to access everything
and dependencies are not clearly visible.

(CODE of IoC)

HOW do i DI with Asp.Net MVC ?

## Next steps

I ll once migrate to Asp.Net Core and use dependency injection. then i ll register the interactors as well as "services" and "repositories"
on the container and let DI do the work.


{% include series.html %}
