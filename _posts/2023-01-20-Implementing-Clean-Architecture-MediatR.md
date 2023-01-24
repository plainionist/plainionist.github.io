---
layout: post
title: "Implementing Clean Architecture - To use or not to use MediatR?"
description: |
  MediatR is a popular library used in .NET to decouple components. In Clean Architecture
  the core of the application should not depend on third-party libraries and frameworks. 
  Can we still use MediatR in a Clean Architecture project or is this violating the 
  Dependency Rule? 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

<img src="{{ site.url }}/assets/clean-architecture/mediatr/header.drawio.png" class="dynimg" title="Clean Architecture and MediatR" alt="Circles of Clean Architecture and logo of MediatR"/>

[MediatR](https://github.com/jbogard/MediatR) is a popular library in .NET used to decouple components.
In Clean Architecture we aim to keep the core of the application as independent as possible from 
such "details" like third-party libraries and frameworks.

Nevertheless, I have recently read quite some 
[articles](https://binodmahto.medium.com/clean-code-architecture-with-mediator-cqrs-pattern-in-net-core-7cec4ee51fc3)
and watched some great
[videos](https://www.youtube.com/watch?v=BimfDeDV4yU)
about using MediatR in Clean Architecture based projects.

But doesn't the usage of MediatR in Clean Architecture break the Dependency Rule?

<!--more-->

To answer this question let's analyze some concrete examples.

## Dispatching commands

In the first example the MediatR library is used to decouple the processing of a web request
from the application logic computing some weather forecast. 

```csharp
[ApiController]
[Route("[controller]")]
class ForecastController : ControllerBase
{
    private readonly IMediator myMediator;

    public ForecastController(IMediator mediator)
    {
        myMediator = mediator;
    }

    [HttpGet(Name = "GetWeatherForecast")]
    public async Task<IActionResult> Get()
    {
        var weather = await myMediator.Send(new ForecastRequest());

        return Ok(weather);
    }
}

class ForecastRequest : IRequest<IReadOnlyCollection<Forecast>> { }
```

```csharp
class ForecastHandler : IRequestHandler<ForecastRequest, IReadOnlyCollection<Forecast>>
{
    private static readonly string[] Summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild", 
        "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };

    public Task<IReadOnlyCollection<Forecast>> Handle(
        ForecastRequest request, CancellationToken cancellationToken)
    {
        var result = Enumerable.Range(1, 5)
            .Select(index => CreateWeatherForecast(DateTime.Now.AddDays(index)))
            .ToReadOnlyCollection();

        return Task.FromResult(result);
    }

    private Forecast CreateWeatherForecast(DateTime date) =>
        new()
        {
            Date = date,
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = Summaries[Random.Shared.Next(Summaries.Length)]
        };
}

```

The Asp.Net controller sends a specific ```ForecastRequest``` to the mediator
which then locates the respective request handler in the application layer.
Once the weather forecast is computed and returned by the ```ForecastHandler```
the mediator transfers it back to the controller which then returns it to 
the web client.

When we visualize the dependencies between these classes we get this picture:

<img src="{{ site.url }}/assets/clean-architecture/mediatr/dependencies.1.png" class="dynimg" title="Class dependencies" alt="Class dependencies of the first example"/>

The Asp.Net controller obviously depends on MediatR due to the usage of the ```IMediator```
interface to send the request. As the Asp.Net controller is located in the frameworks layer
- due to its dependency to the Asp.Net framework - this dependency to MediatR is valid according
to the Dependency Rule.

But we also see that the ```ForecastHandler``` as well as the ```ForecastRequest``` - both
located in the application layer (use cases layer) - having dependencies to MediatR due to the
MediatR interfaces these classes implement. These dependencies clearly violate the Dependency Rule.

## Publishing domain events

Let's look at another example I have found. In this example MediatR is used inside the 
```RegistrationApplicationService``` - an application or domain service located in the 
use cases layer - to publish a domain event.

```csharp
class RegistrationApplicationService
{
    private readonly IMediator myMediator;

    public RegistrationApplicationService(IMediator mediator)
    {
        myMediator = mediator;
    }

    public void Process(RegistrationRequest request)
    {
        if (!IsValid(request))
        {
            // TODO: report failure
        }

        // TODO: process further, e.g. store to database
        
        myMediator.Publish(new RegistrationSucceededDomainEvent(request.User));
    }
}

record RegistrationRequest(string User, string EMail);
```

```csharp
record RegistrationSucceededDomainEvent(string UserId) : INotification;
```

The ```RegistrationSucceededDomainEvent``` can be consumed by multiple receivers in different layers
or even different parts of the software system (bounded contexts) and so allows communication between
independent features. Examples or such receivers could be a service sending a welcome email or 
a service collecting data for some "business analytics".

Visualizing the dependencies between these classes results in the following picture:

<img src="{{ site.url }}/assets/clean-architecture/mediatr/dependencies.2.png" class="dynimg" title="Class dependencies" alt="Class dependencies of the second example"/>

The ```RegistrationApplicationService``` clearly depends on MediatR due to its usage of the ```IMediator```
interface. But also the ```RegistrationRequest``` and even the ```RegistrationSucceededDomainEvent``` 
- located in the domain layer - depends on MediatR due to the interfaces both classes implement.
 Again, these dependencies clearly violate the Dependency Rule!

## Shouldn't we be pragmatic?

Now many software engineers would probably argue: "Hey, that's not a big deal. MediatR does't do any
harm to the application layer or the domain layer. Let's be pragmatic and do not over-engineer it!"

And I definitively see this point. It is important to be pragmatic and an over-engineered 
solution rarely is a good solution. In the end, software design is all about trade-off decisions
and for small and medium sized projects the pragmatic approach of accepting MediatR library
to be part of the application logic might be perfectly fine.

However, there are reasons why I personally would decide differently 
(probably because of being a software engineer in large and partially legacy software systems for years):

First, MediatR is a third-party library which we neither can influence how it will develop in future,
e.g. regarding breaking changes nor whether it will even reach end-of-service or end-of-life soon.
If you think this is very unlikely to happen, I would like to remind you of a few examples from a
not so long back past where most of us probably have thought similar:

- [Prism Library](https://prismlibrary.com/) experienced quite some breaking changes after it
  was released to open source by Microsoft
- Do you even remember [Silverlight](https://www.microsoft.com/silverlight/)?
- And what about all the libraries Microsoft "discontinued" on their journey from
  .NET framework to .NET 6? If you ever "owned" a large component which heavily depends on 
  [Windows Workflow Foundation](https://learn.microsoft.com/de-de/dotnet/framework/windows-workflow-foundation/)
  you know what I am talking about ...

Second, suppose the requirements of our project change and we have to replace MediatR by some
other library or technology. At the time of writing MediatR only supports in-process communication.
What if we require out-of-process communication in some point of time?

Now you might argue that this wouldn't be any problem at all because we could easily replace all 
MediatR interfaces by the interfaces of another library and adapt the APIs of the implementations
easily. This is certainly true if we talk about 10 or even 20 "handlers" but what about 50 or 500?
What about a code base with some hundreds of thousands or even millions of lines of code developed
by many teams? In such a case this "simple replacement of interfaces" becomes a coordination nightmare 
easily ...

Ok, assuming I could convince you that there are cases where the pragmatic approach is not 
the best option: Which options do we have to avoid violating the Dependency Rule?

## Adapters

The "classic" approach in Clean Architecture to integrate third-party libraries or external 
services without violating the Dependency Rule is the adapter pattern. The basic idea is:
we define own interfaces inside the appropriate layer (e.g. application layer) and 
provide an implementation in the frameworks layer which "adapts" the third-party library to 
the project specific interface(s).

In case of MediatR there are three aspects for which we would have to provide an adapter:

- The ```IMediator``` (and related interfaces) which provides APIs for the caller to interact with MediatR.
  Providing own interfaces and an implementation which adapts MediatR these interfaces is pretty
  straight forward.

- The markup interfaces like ```INotification``` and ```IRequest``` used by MediatR to identify messages.
  It turns out, this would not be that easy. As these interfaces are used as markup interfaces only and do 
  not provide any APIs, a compromise could be to derive the custom interfaces from the MediatR interfaces. 
  Strictly speaking this would still violate the Dependency Rule but if we ever would have to exchange 
  these interfaces, this would only be necessary at a single place.

- The handler interfaces like ```IRequestHandler``` used by MediatR to locate the handlers of requests
  and notifications. Providing adapters for these interfaces seems to be the most difficult part as
  MediatR depends on those to locate the message handlers.

From these thoughts we can conclude: the "classic approach" of providing adapters is not that easy to realize
in this case.
Nevertheless, if you are interested in how such adapters could be implemented without 
violating the Dependency Rule then check out this video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/JGgeoB-tXJw" 
  title="YouTube video player" frameborder="0" 
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen>
</iframe>

In the meantime, let's explore other options ...

## MediatR alternative without marker interfaces?

A convenient solution would be an alternative MediatR-like library which does not require 
such markup interfaces for messages and handlers.

Here are some interesting projects I have found:

- [Mediator](https://github.com/martinothamar/Mediator) uses source generators instead of reflection to
  locate handlers.
- [Brighter](https://www.goparamore.io/) is an interesting "Command Processor and Dispatcher implementation"
  with support for task queues.

But these and other projects I have found still require the application logic to implement certain 
library interfaces to locate message handlers.

I found oen [MediatR fork](https://github.com/user040F019F/Mediation) which does not require markup interfaces
for notifications and requests but also this library still requires interfaces for the request handlers.

Do you know a MediatR-like library which does not force the application logic to implement any library interfaces?
Please share your discovery with the community and leave a comment below the article.

## Reinventing the wheel?

Finally, there is always one last option: solving the problem without MediatR at all.
And immediately I hear you objecting: "Come on! Reinventing the wheel is not only a waste of
time, it's also adding unnecessary complexity to my project. That's not a solution at all!"

Well, if I would suggest to completely reimplement MediatR in your project then I would 
certainly agree to your objection, but do you really need MediatR? Do the benefits outbalance 
the disadvantages? 

As to my experience, often only a subset of the full functionality of a library or a framework
is used. And in some cases a solution without such a third-party library would have even
reduced the overall complexity of the software.

Remember the first example? An Asp.Net controller used MediatR to dispatch a request to some 
request handler. A simple alternative would be a request specific interface which would be 
implemented by the request handler and injected into the controller, e.g. via a DI container.
This approach is easy to understand, makes it much easier to follow the control flow compared to 
the MediatR based solution and is still flexible with respect to exchanging the implementation.

In the second example MediatR was used to publish a domain event to an unknown number of handlers.
One alternative could be again to create a domain event specific interface which is then used
by the service to publish the event. The service could import a collection of objects implementing 
the interface or the composite pattern could be used to support one-to-many event notifications.

In cases where a many-to-many communication pattern is needed, a simple "event bus" implementation
or adapter could be created with minimal effort and minimal complexity as shown in
[this video](https://youtu.be/S4aevuQWZvI).

"And what about MediatR behaviors to handle cross-cutting concerns?" you may ask. Well, the 
"classic" design pattern to address cross-cutting concerns is the decorator pattern as 
demonstrated in [this video](https://youtu.be/e_-bf93vx10). All we need are interfaces and a 
design following the Dependency Inversion principle which is exactly what I have sketched in 
this section.

So as you can see "reinventing the wheel" indeed is an option, especially
if we do not really reinvent it but rather choose a pragmatic alternative design.

## Conclusion

That was probably quite a controversial discussion. To summarize my perspective: 
the bigger the code base the more reluctant I would be violating the Dependency Rule.

What is your view on that topic? Pragmatism over following the Dependency Rule? Or vice versa?
I would appreciate your contribution to this discussion.


{% include series.html %}
