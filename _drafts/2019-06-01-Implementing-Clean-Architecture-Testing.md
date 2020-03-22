---
layout: post
title: Implementing Clean Architecture - Testing
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---

i have never been a fan of TDD. to me it always felt like doing things twice.
writing tests was usually complex, took a lot of effort and delivered far too less benefit.
only recently i noticied that this was because i wrote the wrong tests wrong ;-)

after having followed uncle bobs recommendation of a testAPI and developed it further (TDK)
writing test became fun again ;-)
my tests are now acceptance tests which are very simple due to following rules
- always talk to SUT through testAPI which is bridging between data types convenient for test and 
  the actual architecture of SUT. by that make tests very independent from SUT
- only use entities in tests - those are - in DDD terms - objects of the domain and rarely change and
  are also known by domain experts when reviewing tests
- write a TDK which makes tests even more expressive
- only work with data in tests (no mocks of interfaces - no komplex mocking frameworks - this is what
  the TestAPI is for)


how to develop TestAPI and TDK?
- iterative process
- i tend to start with the test because those should be easy to understand from domain expert, easy to maintain and expressive
- the tests drive the TDK - the test dictates which functions and data types are needed (besides entities)
- the TDK wants to talk to the product which demands TestAPIs 
- the TestAPI will never expose any SUT API or data type unless it is entity
- so the TestAPI will create new response models
- which may lead to some complexity in TDK
- finally there will be multiple cycles mostly between TDK and TestAPI unless a good compromise is found

- i am lazy and i always struggle to find a good balance between DRY and testing
- "what is simple enough that writing a test would violate DRY"?
- with this strange idea in mind i currently try to put - as i also think i read it in uncle bobs post - all code in the usecases/entities and test it
- i want to have all gateways that dump that testing feels like violating DRY (the code already clearly documents itself and is too simple that a test adds value - it is obvious)


==> you can test without the web server
==> you can test without many mocks!

- testing interactors !!
- if want to test close to user test controller and presenter (which still does not require a web server)
- the presenter does everything possible to make the view so dump that we never need to test it.
  (humble object)

==> so no testing of html needed

- but what about all the smart JavaScript in a modern SPA?
- maybe u can use frameworks to keep also the JavaScript as simple as possible - i dont want to test it :)
- what about vue or react and binding?

==> in the end u probably just need a very simple integration test which is running thrugh UI to check
    that everything is wired up correctly. but NO deep logic testing through UI!

Where do I do acceptance testing in clean arch?
==> On US?
==> On controller / presenter?
==> interactor … then simple UT for presenter and controller sufficient (because only doing data convertion)


- bdd is a natural fit
  - Bdd only works for inner two circles (this is where the business is and we don’t want technical terms in bdd spec)
  - The rest is technical an can be best tested with unit tests …

"humble object pattern"
- abstract the difficult things ways
- we do not want to test the humble object
- the view is a humble object
- so the viewmodel contains all data ready for the view to just display - no logic in the view to be tested


"
Yes, that’s right: The tests are part of the system, and they participate in the architecture just like every other
part of the system does. In some ways, that participation is pretty normal. In other ways, it can be pretty unique.

In fact, you can think of the tests as the outermost circle in the architecture. Nothing within the system depends 
on the tests, and the tests always depend inward on the components of the system.

Tests that are not well integrated into the design of the system tend to be fragile, and they make the system rigid
and difficult to change. The issue, of course, is coupling. Tests that are strongly coupled to the system must change
along with the system.

Fragile tests often have the perverse effect of making the system rigid. When developers realize that simple changes
to the system can cause massive test failures, they may resist making those changes. For example, imagine the 
conversation between the development team and a marketing team that requests a simple change to the page navigation
structure that will cause 1000 tests to break. The solution is to design for testability. The first rule of software
design—whether for testability or for any other reason—is always the same: Don’t depend on volatile things. GUIs are
volatile. Test suites that operate the system through the GUI must be fragile. Therefore design the system, and the
tests, so that business rules can be tested without using the GUI.

The role of the testing API is to hide the structure of the application from the tests. This allows the production code
to be refactored and evolved in ways that don’t affect the tests. It also allows the tests to be refactored and evolved
in ways that don’t affect the production code. This separation of evolution is necessary because as time passes, the 
tests tend to become increasingly more concrete and specific. In contrast, the production code tends to become increasingly
more abstract and general. Strong structural coupling prevents—or at least impedes—this necessary evolution, and prevents
the production code from being as general, and flexible, as it could be.
"


i still have a "smoke test" starting the whole app and checking for 
exceptions

in f# we focus on making "invalid states" impossible
everything else will be an exception


{% include series.html %}
