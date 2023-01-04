---
layout: post
title: "Book Review: BDD in Action"
description: |
  Behavior-Driven Development (BDD) is a well established term in software industry.
  Nevertheless, many developers and project managers think that all what it is about is writing tests with 
  Gherkin (GIVEN-WHEN-THEN) to make the tests more readable for non-technical stakeholders.
  In fact this is not even close to reality. 
tags: [book]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002, JL0005
---

If you have at least some professional software engineering experience you probably have heard of BDD
and probability is high that you just think of Gherkin and GIVEN-WHEN-THEN when you think of BDD.
But actually BDD is much, much more as this book illustrates:

![Book: BDD in Action]({{ site.url }}/assets/BDD-in-action.jpg "Book: BDD in Action")

[BDD in Action](https://www.amazon.com/-/de/dp/161729165X/ref=sr_1_1?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&crid=2IBJHADUUJ2V7&keywords=bdd+in+action&qid=1672747172&sprefix=bdd+in+action%2Caps%2C264&sr=8-1)

<!--more-->

- short overview on BDD

## First Part

- is about how to define business goals and break down those into features
- vision - goals - capabilities - features - stories - acceptance criteria - examples - code
  - goals (why): how to earn money (how to come up with good goals)
  - capabilities (how): what does the system need to provide to achieve those goals
  - features (what): implemented components that deliver capabilities
  - stories: chunks of features 
- focus on communication and collaboration between business/domain experts and the team.
- ensure to "build the right system"
- features illustrated by concrete examples to avoid misunderstandings

## Second part

- from features and examples "acceptance criteria" which are turned into "executable specifications"
- dont write tests - exectuable specifications
- feature files with scenarios
- here we use gherkin (given when then)
- focus on WHAT not on HOW (key difference from executable spec to tests), less is more
- living documentation: always up to date, reports progress, 

## Third part

- third part: how to automate
- there are tools
- design: feature files, steps, test abstractions
- initialization and cleanup
- separate the what from how
- can be on UI layer but most should focus on business rules layer
- also on lower (more technical level, unit tests) focus is on "executable specification" and not on test
- shows tools and libraries how to practically do it

## Fourth part

- last part shows how BDD can even support project management to track progress of the project
- and how to integrate BDD (specs) in the CI/CD pipeline

## Conclusion

in sumary: BDD is not just given-when-then it is really what the name states a "software development method"

if u want to get max out of BDD - this book is worth reading - even if u have used Gherkin or any 
of the BDD tools already


