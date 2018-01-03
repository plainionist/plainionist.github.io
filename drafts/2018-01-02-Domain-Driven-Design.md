---
layout: post
title: Domain-Driven Design
tags: [book]
---

After having it on the reading list for a long time and having been reminded to read it by several other sources in the recent 
weeks I finally took the time get through

![]({{ site.url }}/assets/DomainDrivenDesign.jpg)

[Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215/ref=sr_1_1?s=books&ie=UTF8&qid=1514895832&sr=1-1&keywords=domain+driven+design)

First, I have to admit: not an easy read - at least not for me. Of course the book is well structured and for sure eric knows what he
is talking about but for me his language is more challenging to read as e.g. Uncle Bob - more advanced english, longer sentences ... 
but finally i made it ;-)

Now lets focus on content!

As Domain-Driven design (DDD) and so the book has quite some aspects to look at let me separate the whole into several parts.

## The essence of DDD

DDD has 3 core statements:

1. The domain model and the heart of the system design shape each other.
   Both have to be developed together and in iterative way, giving constant feedback to each other.
2. The domain model is the backbone of a language - the UBIQUITOUS LANGUAGE (UL) - which is used by all team members, technical
   as well as non-technical. The UL is used in diagrams, in documents, in code and in speech. It must not contain any 
   inconsistancies or ambiguity. A change in the UL is results in a change in the domain model.
3. The domain model is distilled knowledged, iteratively developed through knowledge crunching and continuous learning.

## Building blocks of model-driven design

One overall corner stone of DDD is that the domain model/logic has to be independent from technical details like the database or the UI.
Therefore the system should be separated into four basic layers

- UI: interaction with user
- Application layer: defines jobs and tasks the software is supposed to do. It coordinates and has no business logic or rules.
- Domain layer: represents business concepts, logic and rules, contains the domain model
- Infrastructure layer: provides generic technical capabilities

DDD uses several core building blocks to build the domain layer:

ASSOCIATIONS describe the interactions between ENTITIES (objects with identity) and VALUE OBJECTS (describe things with attributes).
Operations which do not naturally fit into the responsiblity of an ENTITY or a VALUE OBJECT	are put into stateless SERVICES. 
MODULES are use to package conceptual areas which can be reasoned about independently.

AGGREGATES group ENTITIES and VALUE OBJECTS together with one root ENTITY to simplify life cylce handling and design of invariants between objects.
FACTORIES encapsulate the logic required for creating consistent and valid objects.
REPOSITORIES are used to aquire references to preexisting domain objects. The allow add, remove and query of domain objects with usecase 
specific interfaces.
sPECIFICATIONS are predicate-like VALUE OBJECTS which specify business rules for validation, selection and creation of domain objejcts.

## Supple Design

After having identified the basic building blocks of DDD the question now is, how to put them nicely together.

  - intention-revealing interfaces
    - usage of component need to be understood from interface
	- use names from UL, clear names which make purpose clear
  - side-effect-free functions
    - improve predicablility of design/code
    - separate commands and query
  - assertions
    - use "design by contract" so that a developer does not need to check the implementation to 
	  nderstand side effects
  - conceptual contours
    - decompose design elements into cohesive units (from concept perspective)
  - standalone classes
  - closure of operations
    - monad, define operation whose return type is the same as the type of argument(s)
  - declarative design

## Maintaining model integrity
  - Total unification of the domain model for a large system will not be feasible or cost-effective.
    how to draw bundaries
  - bounded context
    - context within which a model applies
	- maximum consistency within the context
	- large projects can have many bounded contexts
	- same concept (e.g. customer) can be modeled differently in different contexts
  - continuous integration
    - happen within one bounded context
    - therefore bounded context should not be too big
  - context map
    - describes the points of contact between the models
	- outlining explicit translation for any communication 
	- map the existing terrain - take up transformtions later
  - shared kernel
    - shared model across bounded contexts
	- explicitly defined and all teams agree on being shared
  - customer/supplier development team
    - clear delivery model between teams and software sub-systems
  - conformist
    - completely allign align with other systems design
  - anti corruption layer
    - clear boundary between sub systems
  - separate ways
    - sometimes integraion is to expensive
	- do not integrate sub systems at all
  - open host service
    - defined protocol that is used to integrate many sub systems
  - published language
    - defined language that is used to integrate sub systems
	- all speaking same language which reduces translation overhead

## Distillation
  - how to maintain focus in a large system?
    how to extract the essence in a form that makes it more valuable and useful?
  - Core domain
    - the model's critical core, the real business asset
	- separate from supporting code to set priorities
	- this is the code which has to be "perfect"
  - Generic Subdomains
    - parts of the domain model which are not the core domain, separated into cohesive "sub domains"
	- should be in separate modules
	- can also be provides as of the shelve solutions to the system
  - domain vision statement
    - set focus of development at the beginning of a project
	- one page description about the core domain and the value it will bring
  - highlighted core
    - brief document that describes the core domain (at the begining of the project)
  - cohesive mechanism
    - separate conceptually cohesive mechanism into separate lightweight framework
	- separate "what" from "how" in the domain
  - segregated core
    - extract the core domain in a separate module
	- reduce coupling to supporting code
  - abstract core
    - identify most fundamental concepts in the model
	- factor into distinct abstract classes/interfaces
	- expresses most of the interaction between significant components
	- own module, specialized implementations left in own module (subdomain)

## Large scale structure
  - how to allow "seeing the trees for the forest"
  - give over all structure to the system and that allows some understanding of each part’s place in the whole—even without detailed knowledge of the part’s responsibility.
  - should be applied when a structure can be found that greatly clarifies the system 
    without forcing unnatural constraints on model development
  - less is more
  - evolving order
    - let conceptual large-scale structure evolve with the appliation
	- follow minimalistic approach
	- possibly changing to a completely different type of structure along the way
  - system metaphor
    - concrete analogy to the system
	- can guie development
  - Responsibility layers
    - reflect conceptual dependencies in the model
	- tell a story of the high-level purpose and design
	- examples: "operational", "capability", "decision support", "policy"
  - knowledge level
    - distinct set of objects that can be used to describe and constrain the structure and behavior of the basic model
	- two levels: one very concrete the other reflecting rules and knowledge the user is able to customize
	- "meta model", "roles"
  - pluggable component framework
    - destill abstract core and create framework that allow divers implementations of these interfaces to be
	  freely substituted

 
That was quite some content.


what i like also is:
- do not talk about big up front design - instead
- constantly highlights: continues knowledge crunching and refactoring to constantly get a better software (which better serves business needs)
- agile practices
- object-oriented

- refactoring towards deeper insight
  - refactor the model if new understandings arise
  - refactor to make the model more expressive

