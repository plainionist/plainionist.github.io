---
layout: post
title: Implementing Clean Architecture - Make it scream
description: Clean Architecture is screaming by focusing on its core business purpose leaving frameworks and other details aside.
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: JL0003
---

How do I make my architecture "scream"?

According to Uncle Bob an architecture "screams" when it clearly expresses its core business purpose.
The top level folder structure, the project/DLL names and the namespaces should express business aspects rather
than frameworks or other details.

*[Athena](/Implementing-Clean-Architecture)* is a web application implemented in ASP.NET MVC. But is this important?
I could switch to Ruby on Rails or Node.js - it wouldn't make any difference for the business. 
Why do we let such details impact our project structure so often?

<!--more-->

## Screaming project structure

*[Athena](/Implementing-Clean-Architecture)* has three core use cases

- Show the backlog with projected team capacity (work balance we consider to be a different view on the backlog itself)
- Calculate a burn down chart
- Ensure backlog conventions via governance rules

In order to let my architecture scream I could create three projects/DLLs accordingly:

<img src="{{ site.url }}/assets/clean-architecture/Athena.Projects.1.png" class="dynimg" title="Minimal project setup" alt="One project for each core use case. At least this project setup screams out the business."/>

But there is one more thing to consider when setting up the project structure. The Dependency Rule.

<img src="{{ site.url }}/assets/clean-architecture/Circles.png" class="dynimg" title="Layers of the Clean Architecture with Dependency Rule" alt="The Clean Architecture consists of multiple layers organized as circles while dependencies are only allowed from outer circles to inner circles. The inner circles contain the business logic. All details, devices and frameworks are in the outer circles."/>

Dependencies between the four circles are only allowed from outer circles to inner circles.
Creating clear boundaries between these circles is a simple way to support the Dependency Rule.
E.g. when the project hosting the business rules does not reference any third party or framework library
at all, this code obviously is free from such unwanted dependencies.

A minimalistic setup respecting both aspects discussed so far could look like this:

<img src="{{ site.url }}/assets/clean-architecture/Athena.Projects.2.png" class="dynimg" title="Minimal Clean Architecture conform project setup" alt="Towards Clean Architecture: Two projects for each core use case. One containing pure business rules witout any dependency to frameworks or devices. A second project for the code with such dependencies."/>

The "UseCases" projects will contain all the framework-free business rules.
The "Gateways" projects will contain all the different kinds of "interface adapters" like Asp.Net controllers and 
repository implementations accessing the TFS. You could now argue that UI code (Asp.Net controllers) and data access
logic (TFS repositories) should not live in the same DLL but honestly I don't see much benefit
right now in separating these aspects so I will keep them together. If I find a good reason later on this decision can easily 
be reverted.

What about the remaining two circles?

A "Frameworks" would contain extensions to the frameworks I use. As I don't have such code yet I will not create 
a project for that circle.

And what about the entities? According to Uncle Bob there would probably be exactly one DLL hosting the enterprise
wide business rules. When *Athena* wouldn't had any code yet I would probably start with an empty DLL and carefully decide
step by step which domain object is really worth sharing across the different features or sub-systems.
From my past experience a little code duplication is easier to "fix" than a "to early" reuse.

As *Athena* has an existing code base already I can easily identify all domain objects related to teams as central entities.
So I have created one "Entities" DLL which gives me now this structure:

<img src="{{ site.url }}/assets/clean-architecture/Athena.Projects.3.png" class="dynimg" title="Minimal Clean Architecture conform setup with separate project for entities" alt="Nore Clean Architecture: Two projects per core use case (one for business rules, one for the rest). A single separate project for entities to share the 'enterprise wide' business rules."/>

## What about infrastructure and helpers?

Looking into my code base I find code which I want to reuse across the three sub-systems but which are NOT entities, e.g.:

- I have some functions which compile reusable HTML snippets.
- I have some functions to handle dates and times conversions consistently.
- I have some code which makes it easier to get the relevant data from TFS. 

I would even consider such code as "framework extensions". But if I would put it in the outer most circle, 
as per the Dependency Rule, I would not be allowed to use it from the "interface adapters" circle where I would need it.

So I decided to create another central "gateways" assembly for such shared "infrastructural" code, which finally gives my this picture:

<img src="{{ site.url }}/assets/clean-architecture/Athena.Projects.4.png" class="dynimg" title="Clean Architecture conform projects with shared 'infrastructure'" alt="Still Clean Architecture: Two projects per core use case (one for business rules, one for the rest). A single separate project for entities. And one more shared project in the 'interface adapters' circle for shared infrastructure."/>

## Naming conventions

The following summarizes the naming conventions I decided for in the context of the *Athena* project:

- Entities
  - Project (DLL) and namespace end with "Entities"
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

## Does it have to be that big?

Finally I came up with eight projects/DLLs which is quite a number for a comparable small application.

For even smaller applications which "do only one thing and do it well" I would recommend to at least separate 
the business rules without framework dependencies from the code which will have framework dependencies.
Such a setup would be less "screaming" but a "vertical slicing" would probably also be less needed in such small projects.

## Easier dependency management in F#?

Did I told you already that *Athena* is written in F#? It is ;-)

The F# compiler has one nice feature which makes managing dependencies easier. The F# compiler processes code in the 
order it is defined in the project. That means code defined earlier in the project cannot access code defined later.

Using that feature I could put all code of one sub-system in a single project and still ensure the Dependency Rule.
I just have to add the code in the reverse order of the Dependency Rule starting with the Entities.
This way I could minimize the number of assemblies. Of course the risk remains that I use framework classes from 
the referenced libraries in the inner circles.

This would be then a question of discipline ...

## Update 2018-05-01

### Gateways

So far I used the term "gateway" as a general abstraction for controllers, presenters, repositories - any kind of 
"interface adapter". After having dug even deeper into Clean Architecture i think "gateway" is best fitting for 
repositories and adapters to external services only. I am currently considering renaming my "gateway" projects into
something like "adapters".

### Frameworks

As discussed [Are Asp.Net controllers "Clean"?](/Implementing-Clean-Architecture-AspNet/) I learned that it is better 
to draw a rather hard border between frameworks and interface adapters. This is probably best achieved by
just not linking to framework projects in the "interface adapters" project. This extends the project setup as follows:

<img src="{{ site.url }}/assets/clean-architecture/Athena.Projects.5.png" class="dynimg" title="Clean Architecture conform projects with frameworks layer" alt="Clean Architecture with frameworks layer: Three projects per core use case (one for business rules, one for the adapters and one for Asp.Net depending code). A single separate project for entities. And one more shared project in the 'interface adapters' circle for shared infrastructure."/>

### Composable UI

A [question on stackoverflow](https://softwareengineering.stackexchange.com/questions/366930/android-clean-architecture-best-way-to-structure-packages/366945?noredirect=1#comment800927_366945)
reminded me to emphasize that a screaming architecture is not limited to the back-end! Modern desktop UI frameworks
like [Prism](https://prismlibrary.github.io/) as well as modern web frameworks like [AngularJS](https://angularjs.org/), 
[ReactJS](https://reactjs.org/) or [VueJS](https://vuejs.org/) are able compose UIs from various, not linked, components.


{% include series.html %}
