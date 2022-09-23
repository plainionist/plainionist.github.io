---
layout: post
title: "Automating dependency governance in .NET"
description: |
    Every software project requires some structure to manage complexity. Every structure implies rules 
    which have to be followed to not break the structure. Every rule requires automatic governance to 
    be effective. How can such automatic governance be set up in .Net projects?
tags: [design]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

Every software project of some reasonable size [needs some structure](https://youtu.be/1IE8RC-IOSE)
to manage complexity and to ensure that it [fits into every developers head](/Code-that-fits-in-your-head/).

Every structure also implies some rules on which code should go where and which dependencies are allowed,
for example

- dependencies between architectural layers are only allowed in one direction
- avoid coupling between individual components or features

But how to ensure that those rules are followed in the daily business of a software project?

<!--more-->

Naive answer: The project has an architecture specification which clearly describes these rules and
the rational behind those. Every developer in the team reads this architecture specification when 
joining the team and follows those rules during feature development.

It is certainly a good starting point to have those rules clearly documented and trained to every
new developer in the team ...

BUT

Even if we assume the most disciplined team there are reasons why having good documentation is not enough,
for example:

- Mistakes happen, that's why writing down functional requirements is not enough - we write tests to 
  verify our software.
- Time pressure tends to cause compromises which result in technical debt, unless such compromises are 
  not accepted by the CI/CD pipeline.
- Developers joining the team have to learn a lot: domain, architectural rules, coding guidelines,
  processes of the project and organization and many more. 
  It is unrealistic - and maybe even unfair - to expect that all these rules, guidelines and processes 
  can be learned, memorized and followed by day one.
  Governance simply reduces load from a developers brain and memory.

## how to set it up?

- not enough that some architect sporadically does an analysis
- fixes are cheaper the earlier found - might especially true for arch relevant issues
- it has to be automated 
- i do not recommend any "scheduled" run at weekend because that blame after checking
- best is to have it as early as possible in CI/CD pipeline and actually in IDE of developers
  so that we get immediate feedback and not get blamed later on

## options?

### compiler 

the compiler can provide such governance regarding "internal" key word and 
cyclic references only if we separate individual components into separate assemblies. 

so as code grows and we want to ensure that deps only go according dependency rule we may split
one assembly into multiple to make use of compiler features.

## tools 

- in .net one common one: ndepend
- thx to roslyn: custom tools
  - show the ut to ut rule from coding bot
- but avoid reinvent the wheel: NsDepCop

## NsDepCop

NsDepCop (https://github.com/realvizu/NsDepCop/issues/28)

goes on namespaces
- independent from assembly structure which may have other drivers as performance or deployment
- whilelisting or blacklisting
- config inheritance

- sample of a project
- sample of inheritance (if u have clear conventions in your project)

how to implement it exactly ==> YT vidoe on my chanel  soon

