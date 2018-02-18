---
layout: post
title: Implementing Clean Architecture - Of controllers and presenters
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.Presenters.png" class="dynimg"/>

Last time we discussed about use cases and interactors and stopped with the question 
"Which role is than actually left to the controller and presenter?".

In this post I will take this question up and dive deeper into the world of controllers and presenters
in the context of the Clean Architecture.

Read on!

<!--more-->

## Definitions

Let me again first look for some definitions. 

What would be more common for the controller to look it up in the Model-View-Controller (MVC) pattern.

[Wikipedia](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller):

> The controller is responsible for responding to the user input and perform interactions on the data model objects. 
> The controller receives the input, it validates the input and then performs the business operation that modifies the state of the data model.

And the presenter in MVP:

[Wikipedia](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter):

> The presenter acts upon the model and the view. It retrieves data from repositories (the model), and formats it for display in the view.

Well, this is about MVC and MVP ... how do the responsibilities change in Clean Architecture?


## Clean Architecture

in MVC and MVP the controller/presenters are the "hearts of logic".

in clean arch as much logic as possible goes into the use case interactors.

controllers and presenters live in "interface adapters" circle - which gives already a hint: they are adpaters only!




https://softwareengineering.stackexchange.com/questions/357052/clean-architecture-use-case-containing-the-presenter-or-returning-data
nice description of controlflow: https://stackoverflow.com/questions/45921928/use-case-containing-the-presenter-or-returning-data
https://stackoverflow.com/questions/46510550/clean-architecture-what-are-the-jobs-of-presenter


## What is the role of the controller? 


the controller takes user input, prepares it for the interactor and pass it to it
could also take the response from interactor and pass it to the presenter
"controller coordinates"

## What is the role of the presenter? 

if the use case has all the logic ...
ideally a presenter is converting data only. It converts data which is most convenient for one layer into data which is most convenient for the other layer.


"
The job of the Presenter is to repackage the OutputData into viewable form as the ViewModel, 
which is yet another plain old Java object. The ViewModel contains mostly Strings and flags that the 
View uses to display the data. Whereas the OutputData may contain Date objects, the Presenter will load the 
ViewModel with corresponding Strings already formatted properly for the user. The same is true of Currency objects or 
any other business-related data. Button and MenuItem names are placed in the ViewModel, as are flags that tell the 
View whether those Buttons and MenuItems should be gray.
"


nice description of controlflow: https://stackoverflow.com/questions/45921928/use-case-containing-the-presenter-or-returning-data

##  Athena

as said: simplified view throught asp.net mvc

patterns allways have to match into context

so i am fine with using simplified view in my simple example in combination with aps.net

nevertheless my controller/presenter classes will focus on the responsiblities from Clean architecture

## implementation overview

Controllers and presenters interact with the interactors respecting the dependency rule.
so how is the general control flow:
- user
- view
- controller
- interactor
- presenter
- view
- user

<img src="{{ site.url }}/assets/clean-architecture/Interactor.Controller.Presenter.png" class="dynimg"/>

how would i implement a presenter if i would like to separate my Asp.Net controller?

## how to impl an input port?

nice description of controlflow: https://stackoverflow.com/questions/45921928/use-case-containing-the-presenter-or-returning-data

==> method call

## how would i implement an output port?

nice description of controlflow: https://stackoverflow.com/questions/45921928/use-case-containing-the-presenter-or-returning-data

==> interface + callback (more details in the book?)




## Which data is passed between controller, interactor and presenter? 

DOUBLE check with the book: this is actually about BOUNDARIES! 
i think i can pass entities from one internal interactor to another one.
once this internal interactor becomes public we should again check what is passing a boundary


Interactors define input DTOs (Data transfer objects) and output DTOs which are most convenient for the use case. 
in his book uncle bob writes that entities should not be passed to use cases or returned from use cases

Uncle Bob:
> We don't want to cheat and pass Entity objects or database rows. 
> We don't want the data structures to have any kind of dependency that violates the Dependency Rule.
>
> [...]
>
> Thus, when we pass data across a boundary, it is always in the form that is most convenient for the inner circle.

All methods we have defined on the interactors so far are simple functions which return results.
Therefore we dont need to define input or output ports as interfaces - we can have simple DTOs for input and output.












## How do others think about controllers and presenters

During research for this post I found many discussions about the "right cut" of use cases. 
Here is a list of some well crafted thoughts:

- https://stackoverflow.com/questions/45921928/use-case-containing-the-presenter-or-returning-data
- https://softwareengineering.stackexchange.com/questions/357052/clean-architecture-use-case-containing-the-presenter-or-returning-data
- https://stackoverflow.com/questions/46510550/clean-architecture-what-are-the-jobs-of-presenter

{% include series.html %}
