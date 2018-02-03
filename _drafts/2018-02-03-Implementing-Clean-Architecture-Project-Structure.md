---
layout: post
title: Implementing Clean Architecture - Project structure
description: When starting with Clean Architecture the project structure like folders, DLLs and namespaces is one of the first questions.
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
---

Let's start with making our architecture SCREEEEEEAM ;-)

<!--more-->

One of the important aspects in Uncle Bob's Clean Architecture is 

*[Athena](/implementing-clean-architecture)* is a web application which uses ASP.NET.

but that we dont want to yell out ... we want to 

<img src="{{ site.url }}/assets/clean-architecture/circles.png" class="dynimg"/>

## What is a pragmatic project structure?

Probably one of the first questions is: How do I organize my code?

Most recommendations I found proposed to have at least one project per "circle". That would be one for entities, one for 
use cases, one for gateways and one for frameworks. If you have a rather "big" project with boundaries and sub-systems
you probably want to have even the use cases and the gateways projects separated ... just to reflect the architectural
boundaries also in code. Entities, as defined by Uncle Bob, are central for the entire enterprise (or eco-system), so
probably one project is sufficient. And as there is usually rather few code in the frameworks circle probably one
project is sufficient as well.

Lets have an example. Imagine we would code s.th. like GitHub. This (simplified) system would have

- a Source Control System (integration)
- an issue tracker 
- a wiki

It would use Git as souce control system. It would have a database for the issue tracking and one for the wiki pages.
The UI would be a web site.

A system of such a size I would probably organize like this:

![]({{ site.url }}/assets/Github.CleanArch.png)

I would probably have three "use case" layer assemblies:

- GitHub.SourceControl.UseCases
- GitHub.Wiki.UseCases
- GitHub.IssueTracker.UseCases

These would be pure "use case" assemblies. No dependencies to any framework or device.

I would then probably continue this separation for the "gateways" layer:

- GitHub.SourceControl.Gateways: hosts adapters to the actual source control system Git.
- GitHub.Wiki.Gateways: hosts adapters to render wiki pages as HTML
- GitHub.IssueTracker.Gateways: hosts adapters to render issues to HTML

With this approach I would put most focus on the logical subsystems and still keep the "circles" separated.

I may have Git, database or web "extensions" - but probably only few so I would try to start with just one "frameworks" project.

- GitHub.IO

If I later would feel uncomfortable with the decision I may split this project into at least database and web, I think.

And what about the entities? To be honest I have not thought about enough about this sample project yet to 
give a clear recommendation here. I would tend to decide carefully "bottom up" which business logic and which domain object
is really worth sharing across subsystem boundaries.
From my past experience a little code duplication is easier to "fix" than a "to early" reuse.


And what about smaller projects? What about projects which are just a single application which "does only one thing and does it well"?

I also have such projects and I tend to handle it in a pragmatic way. 
I just create two projects. One which has framework dependencies and one which has not. The first one obviously contains all 
the adapters then while the second contains all use cases and entities. With this simple approach I keep it simple but my 
(core) business logic independent from "details".

## Simplification in F#

due to the fact that in f# order in project is important we can easily keep one assembly in smaller projects and just 
organize the files according to the circles in the arch

## How to name things?

In the context of Clean Architecture you can find may names which refer to the same thing. 
When implementing one concrete project we surely want to decide for a clear terminology.

For me I decidied as follows:

- Entities
  - Projects (DLL) and namespaces would end with "Entities"
  - Classes would not have any specific convention
- Use cases
  - Projects and namespaces would end with "UseCases"
  - Classes would end with "Interactor"
- Gateways
  - Projects and namespaces would end with "Gateways"
  - Classes would end with role typical postfixes like "Adapter" or "Controller"
- Frameworks
  - Projects and namespaces would end with role typical postfixes like "Web", "IO" or "UI".
  - Classes would not have any specific convention
