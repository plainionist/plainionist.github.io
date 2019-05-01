---
layout: post
title: Implementing Clean Architecture - Frameworks vs. Libraries
description: |
  In Clean Architecture the Dependency Rule forbids usage of frameworks outside of the outermost circle.
  Does this mean that the usage of any third party API in the implementation of a gateway or repository
  is a violation to the Dependency Rule?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: JL0003
---

<img src="{{ site.url }}/assets/clean-architecture/Circle.Frameworks.png" class="dynimg" title="Frameworks and libraries in the context of Clean Architecture." alt="In Clean Architecture the usage of frameworks is restricted to the outermost circle. But what is a framework? Is every third party library a framework? How to implement gateways without using third party libraries?"/>

It has been a while since [my last post](/Implementing-Clean-Architecture-AspNet/) on "Implementing Clean Architecture"
but I wasn't lazy ;-) In fact I was - apart from my day job - working on getting 
*[Athena](/Implementing-Clean-Architecture)* closer to the Clean Architecture. But there was one thing which puzzled me 
for a while when implementing [repositories](https://deviq.com/repository-pattern/):

In Clean Architecture
- all details are restricted to the frameworks layer. The database is a detail.
- all data conversion from the format most convenient for what ever persistence framework to the format
  most convenient for entities and use cases, happens in the interface adapters layer. 

How do I implement a repository in the interface adapter circle which accesses the TFS database using the 
Microsoft TFS framework APIs? Isn't that a violation to the Dependency Rule? In fact, isn't any usage of a third party
API outside the frameworks circle a violation to the Dependency Rule?

In this post I am trying to answer this question.
