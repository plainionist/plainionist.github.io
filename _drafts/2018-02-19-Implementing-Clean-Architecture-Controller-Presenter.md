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

Let me again first look for some definitions ... 

... and what would be a more classical context to lookup a definition for 'controller' than the Model-View-Controller (MVC) pattern.

[Wikipedia](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller):

> The controller is responsible for responding to the user input and perform interactions on the data model objects. 
> The controller receives the input, it validates the input and then performs the business operation that modifies the state of the data model.

.. and the counterpart for the 'presenter' would be of course the Model-View-Presenter (MVP) pattern.

[Wikipedia](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter):

> The presenter acts upon the model and the view. It retrieves data from repositories (the model), and formats it for display in the view.

Well, this is about MVC and MVP - but how do the responsibilities change in the Clean Architecture?

## Clean Architecture 

In the MVC/MVP patterns the controller/presenter contains most of the business logic. In the Clean Architecture all of the 
business logic goes either into an use case interactor or an entity (we will talk about entities later).

We earlier looked at this picture:

<img src="{{ site.url }}/assets/clean-architecture/Interactor.Controller.Presenter.png" class="dynimg"/>

From that we can see already that controllers and presenters live in the "interface adpaters" circle (green color) 
which gives a strong hint: they are adapters only - and we don't want to have adapers much of logic right? 
Adapters just map from one world to another.

Let me zoom a little bit out of the picture to get the full context.

<img src="{{ site.url }}/assets/clean-architecture/User.Interactor.Flow.png" class="dynimg"/>

So what do we see here? Following the purple arrow we can see the control flow of a user interaction with 
a system:

1. The user interacts with the view.
2. The view creates a request (object) which is passed to the controller.
3. The controller converts the request into a request model and passes it to the use case interactor through
   its input port.
4. The use case interactor processes the request model and creates a response model which is passed through the 
   output port to the presenter.
5. The presenter converts the response model to view model which is then passed to the view.
6. The user sees the result of his interaction in the view.

But there is more to be explored. We see two vertical boundaries separating the picture into three sections:

- The most left (blue) section where the frameworks live (e.g. an HTML/Javascript view)
- The middle (green) section where the adapters live (controllers and presenters)
- The most right (red) section where the business logic (use case interactors) live

the controllers and presenters are sitting in the middle of this picture
briding between both worlds 

If we look closer now at the controller and presenter we see even more clearly that they only
convert data objects into other data objects to "bridge" between the user and the business logic.

**Can you see the Dependency Rule?**

> And we see that the black arrows crossing the boundaries are always (!) going from left to right. 
> These arrows represent code level dependencies which means the code where an arrow starts knows about
> the code the arrow is pointing to. So the arrows perfectly respect the Dependency Rule.
> 
> You probably have noticed that there are two types of arrows: The open arrow indicates a "uses" relationship,
> the closed one indicates a "implements" or "extends" relationship. We will look into this difference in more detail soon.


Now that we know how the controllers and presenters fit into the complete picture of the Clean Architecture 
let's deep dive into their responsiblities.

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

presenters are a form of "humble object" pattern - separates behavior easy to test from behavior hard to test.
is about creating boundaries.

for everything the view needs to know the presenter create a field on the view model in the appropriate format and data type ...
probably mostly strings.

there could be multiple presenter per "use case": one for html, one for print, one for wpf ...

## How do I implement controllers and presenters?

how do i implement a controller? - i mentioned so far in athena controller and presenter are one class ... as motivated by Asp.Net mvc.

lets first describe how that works and then disuss whether this is clean arch conform and how it could be done differently

remember the backlog use case from [last post]()?

<img src="{{ site.url }}/assets/clean-architecture/backlog.png" class="dynimg"/>

we discussed what the interactors are doing. This time lets look at some code to figure out what the 
controller/presenter is doing

==> show code!!
- just the controller then the filter
- then how the controller converts the filter into a request model and calls the interactor
- take response model and convert into view model
  (then show the view model)


is this valid clean arch?

uncle bob in his book says:
"
We want to protect the Controller from changes in the Presenters.
"

i found multiple discussions in the web 

as said: simplified view throught asp.net mvc

is this actually correct according to clean architecture to just return data from interactor and pass this to presenter?
is it ok having the controller know the presenter? just passing data?
if we look at the picture the architecture clearly says that the presenter "implements" the output port ... arrow asks for polymorphism


patterns allways have to match into context

so i am fine with using simplified view in my simple example in combination with aps.net



nevertheless my controller/presenter classes will focus on the responsiblities from Clean architecture


Controller and Presenter are different objects?
how would i implement a presenter if i would like to separate my Asp.Net controller?

## Separating controller and presenter

start simple: controller gets all code before we call the interactor, presenter gets the code after we got data from interactor.

CODE controller

CODE presenter

now still this is  not exactly matchting the picture we started with. lets look at it again

<img src="{{ site.url }}/assets/clean-architecture/User.Interactor.Flow.png" class="dynimg"/>

we have a controller - we have a presenter.
we have request (give the name) and a request model (give the name).
we have a response model (give the name) and a viewmodel (give the name).

where are the ports?

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


## How to invert to control flow?

now controller injects presenter into interactor. but still controller asks for view model form presenter - due to nature 
of asp.net mvc fremwwork

can we fix that?



for now we just did it in the controller? is this correct? ... next post.

But who wires all this up? If the controller and presenter are different classes: who is injecting the
presenter into the interactor? This will be answered in one of the next posts about "The Main".




## How do others think about controllers and presenters

During research for this post I came along many other discussions about controllers and presenters in the Clean Architecture.
Here are some of them:

- [Use case containing the presenter or returning data?](https://softwareengineering.stackexchange.com/questions/357052/clean-architecture-use-case-containing-the-presenter-or-returning-data)
- [What are the jobs of presenter?](https://stackoverflow.com/questions/46510550/clean-architecture-what-are-the-jobs-of-presenter)


 https://stackoverflow.com/questions/45921928/use-case-containing-the-presenter-or-returning-data

{% include series.html %}
