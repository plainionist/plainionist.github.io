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

The book starts with a quick tour on what BDD is really about and already there it becomes clear that 
it is much more than just writing tests with GIVEN-WHEN-THEN phrases.
The rest of the book is divided into four parts.

The first part is all about the business. It explains how to define the business goals (the "why") and
the capabilities (the "how") of the system to be built. It continues explaining how to come up with
features (the "what") and how those can be broken down into stories and illustrated by concrete examples.
The emphasis is thereby always on communication and collaboration between business/domain experts and the team.
In essence, it is all about "building the right system".

The second part describes how features and examples are turned into acceptance criteria and how those
are turned into "executable specification". The key message is: "Don't write tests, write executable
specifications". To achieve this, this part gives an overview on Gherkin and how to write good BDD
scenarios.

The third part focuses on how to automate the scenarios written in Gherkin. It explains design aspects
as well as how to use different automation frameworks in different programming languages.
If finally shows that the idea of "executable specifications" is not limited to automating (business)
acceptance criteria but that it can also be beneficial when being used for describing and verifying 
lower level technical aspects. 

The last part of the book shows how BDD can even support project management to track the progress of the project
and how it can be integrated into any CI/CD pipeline.

In summary, BDD is not just about writing tests using GIVEN-WHEN-THEN. 
Instead, it is what the name already indicates: a complete software development approach.

If you want to get the maximum benefit out of BDD for your project, this book is worth reading,
even if you have some practical experience with Gherkin and BDD tools already.
