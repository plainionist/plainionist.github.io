---
layout: post
title: Implementing Clean Architecture - What is a use case?
description: Clean Architecture is very much focusing on business by emphasizing use cases. But what is a use case? How big should it be? How does it interact with its environment?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
draft: true
excerpt_separator: <!--more-->
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.UseCase.png" class="dynimg"/>

last time we discussed about use cases and stop a little with how use cases integrate with database and controller and presenter

in my simplified sample project athena where controller and presenter are same class - as discussed in this 
stackoverflow questn this might not be ideal solution but a pragmatic one for asp.net mvc frmework :)

in this post i would now liek to dive deeper in to controlers and presenters in clean arch world

Read on!

<!--more-->

## Definitions

what is a controller and presenter in general

## Athena view?

as said: simplified view throught asp.net mvc
patterns allways have to match into context
so i am fine with using simplified view in my simple example in combination with aps.net




nice description of controlflow: https://stackoverflow.com/questions/45921928/use-case-containing-the-presenter-or-returning-data

## Which data is passed to and returned from a use case interactor? 

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

## What is the role of the presenter then? 

ideally a presenter is converting data only. It converts data which is most convenient for one layer into data which is most convenient for the other layer.


"
The job of the Presenter is to repackage the OutputData into viewable form as the ViewModel, 
which is yet another plain old Java object. The ViewModel contains mostly Strings and flags that the 
View uses to display the data. Whereas the OutputData may contain Date objects, the Presenter will load the 
ViewModel with corresponding Strings already formatted properly for the user. The same is true of Currency objects or 
any other business-related data. Button and MenuItem names are placed in the ViewModel, as are flags that tell the 
View whether those Buttons and MenuItems should be gray.
"

## how to impl an input port?

nice description of controlflow: https://stackoverflow.com/questions/45921928/use-case-containing-the-presenter-or-returning-data

==> method call

## how would i implement an output port?

nice description of controlflow: https://stackoverflow.com/questions/45921928/use-case-containing-the-presenter-or-returning-data

==> interface + callback (more details in the book?)











## How do others think about controllers and presenters

During research for this post I found many discussions about the "right cut" of use cases. 
Here is a list of some well crafted thoughts:

-

{% include series.html %}
