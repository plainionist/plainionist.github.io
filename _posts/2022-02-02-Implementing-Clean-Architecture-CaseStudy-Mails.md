---
layout: post
title: "Implementing Clean Architecture - Case Study: Sending e-mails"
description: |
  This case study shows how a small feature could be designed according to the rules of 
  Clean Architecture. We will explore which code should be located in which layer and how
  the different components will collaborate.
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

During my day job we recently did a code review of a small feature of an application which aims
to follow the principles of [Clean Architecture](/Implementing-Clean-Architecture). 
At a first glance separation of concerns as well as basic principles of Clean Architecture were
followed. But a closer look revealed that some dependencies did not follow the Dependency Rule, 
specifically there were dependencies from adapters layer to the frameworks layer.

During our design discussions which followed this observation I realized that this example
would perfectly serve for a case study on how to design a feature in Clean Architecture in detail. 

So here we go ...

<!--more-->

## Setting the context

The application, this feature is designed for, analyzes test failures from our CI/CD 
pipeline (stored in some test database) and creates defects in our defect database. 
The application also has a simple configuration file containing e.g. properties to fill
certain defect fields like "Assigned To".

The feature we were reviewing is supposed to send e-mails to our development teams in case some
test failures could not be processed properly because of e.g.:

- Some network failure broke the whole CI/CD pipeline execution
- An internal error in our application happened (e.g. due to corruption of the configuration file 
  or a programming error)
- The database storing the test failures from our CI/CD pipeline is down

<img src="{{ site.url }}/assets/clean-architecture/sending-emails/Context.drawio.png" class="dynimg" title="Context of the application" alt="CI/CD pipeline stores test results in a test database, application analysis the test failures and creates defect in defect database based on configuration. Additionally the application can send emails."/>

## Iteration 1 - Getting started

<img src="{{ site.url }}/assets/clean-architecture/Circles.png" width="50%" class="dynimg" title="Layers of the Clean Architecture with Dependency Rule" alt="The Clean Architecture consists of multiple layers organized as circles while dependencies are only allowed from outer circles to inner circles. The inner circles contain the business logic. All details, devices and frameworks are in the outer circles."/>

In line with agile principles let's develop the design incrementally starting with the "core" of any 
application: the business logic.

As all application specific business logic is organized in [Use case interactors](/Implementing-Clean-Architecture-UseCases)
in Clean Architecture we create a new one called ```MailNotificationInteractor``` which has the
responsibility to detect the scenarios mentioned above. The ```MailNotificationInteractor``` could be triggered
through an API call from the actual test failure processing interactor or it could receive a notification through
some messaging mechanism inside the application. For the scope of this post we will not elaborate this aspect any further.

Instead, let's explore the responsibility of the ```MailNotificationInteractor```. It

- Detects what exactly happened
- Decides which concrete information should be notified to whom
- Fetches all relevant information e.g. from test database, application log and configuration file
- Composes a respective e-mail
- Triggers sending the e-mail by an e-mail server

Of course our interactor would not talk to any of these "external devices" (test database, configuration file,
e-mail server) directly as this would create a dependency from the use case layer to some outer layer which would
violate the Dependency Rule. Instead, we create interfaces for all these collaborations in the use case layer and design
those to be most convenient for the ```MailNotificationInteractor```, e.g. we create specific APIs on the configuration
repository to read all relevant information for creating and sending e-mails.

Furthermore, we decide that the ```MailNotificationInteractor``` will create different e-mail response objects
representing the different scenarios detected in the application and containing specific detailed information about those.
For instance, in case of a network issue the IT experts should be informed about the concrete impacted test agents and 
in case of some internal application error the exception details have to be part of the e-mail.

This is the design we end up with after the first iteration:

<img src="{{ site.url }}/assets/clean-architecture/sending-emails/Step1.drawio.png" class="dynimg" title="Separating business logic from mail client" alt="The interactor is the center of the design in the use case layer. It interacts with its environment through interfaces which are designed to be most convenient for the interactor itself. The result of the interactor are multiple type-safe response objects."/>

## Iteration 2 - Communicating to the outer world

In Clean Architecture the test database, the configuration file as well as the e-mail server are considered as
"external devices" which are located in the most outer circle, the [Frameworks layer](/Implementing-Clean-Architecture-Frameworks).
In order to allow collaboration of the ```MailNotificationInteractor``` with these external devices 
we use the [Dependency inversion principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle).
As we created the required interfaces already in the previous iteration we only need to provide
one implementation each inside the frameworks layer which then talks to the respective external device.
The ```MailClient``` for example would take the different e-mail response objects, builds up an
```System.Net.Mail.MailMessage``` and passes it to ```System.Net.Mail.SmtpClient``` which passes it to 
the actual e-mail server to deliver the e-mail.

The remaining piece of the puzzle is a component which wires up all implementations with the interfaces.
This component Uncle Bob calls the "Main component":

> The Main component is the ultimate detail - the lowest-level policy. It is the initial entry point of the system. 
> Nothing, other than the operating system, depends on it. Its job is to create all the Factories, Strategies, and 
> other global facilities, and then hand control over to the high-level abstract portion of the system.
> 
>  --- Clean Architecture, Robert C. Martin (aka: "Uncle Bob")

The design now looks like this:

<img src="{{ site.url }}/assets/clean-architecture/sending-emails/Step2.drawio.png" class="dynimg" title="Connecting to outer world" alt="In order to communicate from the interactor to the external devices implementations of the previously defined interfaces are added. The Main component composes the feature by wiring up implementations and interactor through the interfaces."/>

## Iteration 3 - Formatting HTML e-mails 

Ok, so far so good. The application can now send e-mails and all dependencies follow the Dependency Rule
of the Clean Architecture. 

But then it happens - as usual - just as we have finished our nice design the requirements change: instead
of pragmatic but not so pretty ASCII e-mails, the customers demand nicely formatted HTML e-mails ...

So which component in the current design is responsible for creating the e-mail body and should do all the 
HTML formatting? It is clearly not the responsibility of the interactor - "formatting" is clearly not part 
of the business logic, especially not any formatting for a specific presentation technology like HTML.

What about the ```MailClient```? Dependency-wise this could be an option but remember that we want as less
code in frameworks layer as possible in Clean Architecture because we want as less code as possible depending
on external devices and external frameworks and we would have a hard time testing code with such dependencies.

Actually, the best fit for formatting something would be a "presenter" component. Such a component
would be located in the [adapters layer](/Implementing-Clean-Architecture-Controller-Presenter) in 
Clean Architecture. So we create a ```MailClientAdapter``` which creates a view model called ```HtmlMail``` 
which - in contrast to the response objects of the interactor - is pretty generic and is designed to be very
convenient for the ```MailClient``` to be processed (e.g. it as a "Body" property containing a completely 
formatted HTML content as string).

The ```MailClientAdapter``` now implements ```IMailClient``` interface defined in the use case layer and
defines a new ```IMailClient``` interface which consumes the ```HtmlMail``` view model and which is 
implemented by the ```MailClient``` in the frameworks layer. 
(I know it is a poor design choice to use the same name for two interfaces in two different layers 
but I couldn't come up with a better name so far. Do you have one? Leave a comment below ...)

Finally we will change the Main component accordingly.

<img src="{{ site.url }}/assets/clean-architecture/sending-emails/Step3.drawio.png" class="dynimg" title="Adding presenter for HTML formatting" alt="The MailClientAdapter is added to the adapters layer to take over the role of a presenter which builds HTML e-mails from the response objects of the interactor. The dependencies of the Main component is changed accordingly."/>

## Iteration 4 - Adding some tests we should ...

Of course we could have - and maybe even should have? - started the whole feature by writing the tests first.
On the other hand, this post is not about test driven development but about how to design a feature according 
to the Clean Architecture. So we will also not dive into test design or test engineering in this section but
rather explore how we would add testability to this feature.

One of the key questions is: How will tests interact with the application in general and the e-mail feature
specifically? Should the tests interact with the UI? Should the tests interact with the business logic APIs,
the interactors? Not being convinced at first, today I fully agree with Uncle Bob on the need for a "Test API":

> The purpose of the testing API is to decouple the tests from the application. This decoupling encompasses
> more than just detaching the tests from the UI. The goal is to decouple the *structure* of the tests from
> the *structure* of the application.
>
> [...]
>
> This API will be a superset of the suite of interactors and interface adapters that are used by the
> user interface. 
>
>  --- Clean Architecture, Robert C. Martin 

Furthermore, the Test API will provide fake implementations for all components in the frameworks layer so
that the tests do not depend on external devices whose behavior is unpredictable. 

This setup enables simple and fast tests, focusing on testing as well as documenting the scenarios of the
e-mail feature from customer perspective 
(see [BDD](https://automationpanda.com/2017/01/25/bdd-101-introducing-bdd/)).

<img src="{{ site.url }}/assets/clean-architecture/sending-emails/Step4.drawio.png" class="dynimg" title="Adding testability through a TestAPI" alt="A TestAPI is introduced in the adapters layer to serve as an dedicated interface for testing. Tests only interact with the application and the feature through that TestAPI."/>

## Conclusion

Actually pretty straight forward, isn't it? ;-)

The key - from my perspective - is to just follow:

1. Separation of Concerns (decision making, formatting, external devices)
2. Dependency Rule

## Update

In this video I show how an implementation of this design could look like:

<iframe width="560" height="315" src="https://www.youtube.com/embed/6hIg86y9HuE" 
  title="YouTube video player" frameborder="0" 
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


{% include series.html %}
