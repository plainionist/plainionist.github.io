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

we earlier looked at this picture ..

<img src="{{ site.url }}/assets/clean-architecture/Interactor.Controller.Presenter.png" class="dynimg"/>

and we see already : controllers and presenters live in "interface adapters" circle - which gives already a hint: they are adpaters only!

lets add some more details:

<img src="{{ site.url }}/assets/clean-architecture/User.Interactor.Flow.png" class="dynimg"/>

todo: describe what we see here with control flow
- user
- view (part of the framework - e.g. HTML/JavaScript)
- controller (adapter)
- interactor (logic)
- presenter
- view
- user

todo: describe the dependencies (code level dependencies - arrow means "knows about")
(see: dependency rule!) - explain different kind of arrows

todo: describe special "implemented by" arrows

so now that we know how the controllers and presenters fit into the complete picture of 
Clean Architecture lets check out what they are doing there.

## What is the role of the controller? 

the controller takes user input, prepares it for the interactor and pass it to it
could also take the response from interactor and pass it to the presenter
"controller coordinates"

the controller probably receives a "command" from the view with some parameters which trigger the whole scenario.

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

decouple view and interactor. none knows about the data format of the other. the presenter is the "adapter"

takes the response model and turns it into another data strcuture - one which is most convenient for the view.
so that the view can be as dump as possible - because the view is very hard to test.
if we want to test how things are show to the user we want to do it on the presenter
(humble object)

there could be multiple presenter per "use case": one for html, one for print, one for wpf ...

## Back to *Athena*

Controller and Presenter are different objects?

"
We want to protect the Controller from changes in the Presenters.
"


as said: simplified view throught asp.net mvc

is this actually correct according to clean architecture to just return data from interactor and pass this to presenter?
is it ok having the controller know the presenter? just passing data?
if we look at the picture the architecture clearly says that the presenter "implements" the output port ... arrow asks for polymorphism


patterns allways have to match into context

so i am fine with using simplified view in my simple example in combination with aps.net

nevertheless my controller/presenter classes will focus on the responsiblities from Clean architecture


how would i implement a presenter if i would like to separate my Asp.Net controller?


## how to impl an input port?

this is an interface implemented by an use case interactor.
do i need to have a dedicated interface? i dont think so. i am pretty fine with calling a method on the usecase interator directy.
do i intend to replace the interactor implementation later? NO - i would just change it. 
do i have other reasons to hide the interactor implementation? NO - even if i would have further APIs on the interactor for 
testing i would make them internal ... controllers i would anyhow put in other projects than interactors to keep dependencies to third party clean.
maybe i have an interactor providing multiple public methods but not all public methods should be accessible to all controllers?
then we should have separate interfaces - one per "scenario".

"
The FinancialReportRequester interface serves a different purpose. It is there to protect the FinancialReportController from knowing too much about the internals of the Interactor. If that interface were not there, then the Controller would have transitive dependencies on the FinancialEntities. Transitive dependencies are a violation of the general principle that software entities should not depend on things they don’t directly use. We’ll encounter that principle again when we talk about the Interface Segregation Principle and the Common Reuse Principle. So, even though our first priority is to protect the Interactor from changes to the Controller, we also want to protect the Controller from changes to the Interactor by hiding the internals of the Interactor.

Martin, Robert C.. Clean Architecture: A Craftsman's Guide to Software Structure and Design (Robert C. Martin Series) (pp. 74-75). Pearson Education. Kindle Edition. 
"


==> method call

## how would i implement an output port?


==> interface + callback (more details in the book?) interface defined by the use case - most convenient for the use case
    containing the output data defined by the use case

==> again: what about just returning data?


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







But who wires all this up? If the controller and presenter are different classes: who is injecting the
presenter into the interactor? This will be answered in one of the next posts about "The Main".




## How do others think about controllers and presenters

During research for this post I came along many other discussions about controllers and presenters in the Clean Architecture.
Here are some of them:

- [Use case containing the presenter or returning data?](https://softwareengineering.stackexchange.com/questions/357052/clean-architecture-use-case-containing-the-presenter-or-returning-data)
- [What are the jobs of presenter?](https://stackoverflow.com/questions/46510550/clean-architecture-what-are-the-jobs-of-presenter)


 https://stackoverflow.com/questions/45921928/use-case-containing-the-presenter-or-returning-data

{% include series.html %}
