---
layout: post
title: "Book Review: Domain-Driven Design"
description: Brief review and summary on Eric Evans Domain Driven Design book
tags: [book]
excerpt_separator: <!--more-->
lint-nowarn: JL0003
---

After having it on the reading list for a long time and having been reminded to read it by several other sources
in the recent weeks I finally took the time get through

![Book: Domain-Driven Design: Tackling Complexity in the Heart of Software]({{ site.url }}/assets/DomainDrivenDesign.jpg "Book: Domain-Driven Design: Tackling Complexity in the Heart of Software")

[Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215/ref=sr_1_1?s=books&ie=UTF8&qid=1514895832&sr=1-1&keywords=domain+driven+design)

First, I have to admit: not an easy read - at least not for me. Of course the book is well structured and for
sure Eric knows what he is talking about but for me his language is more challenging to read as e.g. Uncle
Bob - more advanced English, longer sentences ... but finally i made it ;-)

Now lets focus on content!

<!--more-->

As Domain-Driven Design (DDD) and so the book has quite some aspects to look at let me separate the whole into
several parts.

## The essence of DDD

DDD has 3 core statements:

1. The domain model and the heart of the system design shape each other.
   Both have to be developed together and in iterative way, giving constant feedback to each other.
2. The domain model is the backbone of a language - the UBIQUITOUS LANGUAGE (UL) - which is used by all team
   members, technical as well as non-technical. The UL is used in diagrams, in documents, in code and in speech.
   It must not contain any inconsistencies or ambiguity. A change in the UL is results in a change in the
   domain model.
3. The domain model is distilled knowledge, iteratively developed through knowledge crunching and continuous
   learning.

## Building blocks of model-driven design

One overall corner stone of DDD is that the domain model/logic has to be independent from technical details
like the database or the UI. Therefore the system should be separated into four basic layers

- UI: interaction with user
- Application layer: defines jobs and tasks the software is supposed to do. It coordinates and has no
  business logic or rules.
- Domain layer: represents business concepts, logic and rules, contains the domain model
- Infrastructure layer: provides generic technical capabilities

DDD uses several core building blocks to build the domain layer:

ASSOCIATIONS describe the interactions between ENTITIES (objects with identity) and VALUE OBJECTS (describe
things with attributes). Operations which do not naturally fit into the responsibility of an ENTITY or a
VALUE OBJECT are put into stateless SERVICES. MODULES are use to package conceptual areas which can be
reasoned about independently. AGGREGATES group ENTITIES and VALUE OBJECTS together with one root ENTITY to
simplify life cycle handling and design of invariants between objects.

FACTORIES encapsulate the logic required for creating consistent and valid objects. REPOSITORIES are used to
acquire references to preexisting domain objects. The allow add, remove and query of domain objects with use
case specific interfaces. SPECIFICATIONS are predicate-like VALUE OBJECTS which specify business rules for
validation, selection and creation of domain objects.

## Supple Design

After having identified the basic building blocks of DDD the question now is, how to put them nicely together.

INTENTION-REVEALING INTERFACES ensure that clearly expose their purpose, simplifying the reading of design and
code. SIDE-EFFECT-FREE FUNCTIONS improve predictability of design and code and separate the code into commands
and queries. ASSERTIONS are a form of "Design by contract" which make constraints explicit and remove the need
to read the implementations to understand these constraints. CONCEPTUAL CONTOURS decompose design elements into
cohesive conceptual units. STANDALONE CLASSES, CLOSURE OF OPERATIONS and DECLARATIVE DESIGN further help shape
a supple design.

## Maintaining model integrity

Total unification of the domain model for a large system will not be feasible therefore boundaries have to be
drawn.

The BOUNDED CONTEXT defines a context within a model applies. The same concept (e.g. a customer) can be
expressed differently in different BOUNDED CONTEXTS. CONTINUOUS INTEGRATION is an important technique within
BOUNDED CONTEXTS to avoid its fragmentation. The CONTEXT MAP describes the points of contact between models
(BOUNDED CONTEXTS) and outlines explicit translations. A SHARED KERNEL can be explicitly defined by teams to
share certain parts of the over all domain model across BOUNDED CONTEXTS. Sometimes a clear CUSTOMER/SUPPLIER
relationship between teams and so subsystems can help.

The CONFORMIST is used to when completely aligning with the design of another system if translation between
independent designs is just too expensive. Alternatively an ANTI-CORRUPTION layer is used to construct a
clear boundary between two systems which blocks floating concepts across. If integration of systems becomes
two expensive SEPARATE WAYS could be followed to do not integrate the systems at all.

The OPEN HOST SERVICE defines a clear protocol that can be used to integrate multiple systems in a uniform way.
A PUBLISHED LANGUAGE extends this approach and further reduces translation overhead.

## Distillation

Once boundaries have been drawn the question arises, how to maintain focus in a large system?

The CORE DOMAIN defines the model's critical core, the real business asset. It should be separated from
supporting code to make its priority clear. Remaining parts of the domain model are separated into
GENERIC SUBDOMAINS which could even be replaced with off-the-shelve solutions.

A DOMAIN VISION STATEMENT at the beginning of the project will set the focus of the development. It briefly
describes the CORE DOMAIN and the value it will bring. The HIGHLIGHTED CORE continues the DOMAIN VISION STATEMENT
and describes the CORE DOMAIN in a little bit more detail.

The COHESIVE MECHANISM is used to separate the "how" from the "what" by factoring conceptually cohesive
mechanisms into separate modules. The SEGREGATED CORE extracts the CORE DOMAIN into a separate module to reduce
the coupling to supporting code. If even the core is rather huge the ABSTRACT CORE identifies most fundamental
concepts and factors those into distinct abstract classed and interfaces. This allows expressing most of the
interaction between significant components/concepts separated from the specialized implementations.

## Large scale structure

While "distillation" helps to maintain focus it is also necessary to maintain a view which allows
"seeing the trees for the forest". Large scale structures give an over all structure to the system to allows
some understanding of each part’s place in the whole even without detailed knowledge of the part’s
responsibility. To avoid unnatural constraints on the domain model development the guiding principle here
is: less is more.

EVOLVING ORDER states that the conceptual large-scale structure should evolve with the application following
a minimalistic approach. This approach could drive the project to completely different type of structure
along the way.

The SYSTEM METAPHOR tries to apply a concrete, known analogy to the system to guide development.

RESPONSIBILITY LAYERS reflect the conceptual dependencies in the model. They tell a story of the high-level
purpose and design. Examples are: "Operational", "Capability", "Decision support" and "Policy".
The KNOWLEDGE LEVEL consists of a distinct set of objects that can be used to describe and constrain the
structure and behavior of the basic model - a meta model comparable to reflection mechanisms in technical
frameworks. The meta model is provided to the customer for customization of the rules of the actual model.
A PLUGGABLE COMPONENT FRAMEWORK require an ABSTRACT CORE and a concrete framework that allows implementations
of the ABSTRACT CORE to be freely substituted.

## Conclusion

That was quite some content ;-)

What I especially like apart from focusing on the business domain is that DDD nicely fits into the agile
thinking. Eric constantly highlights refactoring towards deeper insight and iterative working models.

I'll definitive continue learning and applying these ideas in my next projects.
