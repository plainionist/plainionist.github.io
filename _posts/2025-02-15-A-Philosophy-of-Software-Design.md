---
layout: post
title: "Book Review: A Philosophy of Software Design"
description:
    What is software complexity, its drivers and consequences?
    This book provides an analysis from an interesting perspective and of course suggestions to reduce complexity.
tags: [book]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002, JL0005
---

Have you read "Clean Code" by Robert C. Martin?
Then "A Philosophy of Software Design" by John Ousterhout should be next on your list.
It shares the same goal—writing better, maintainable software—but sometimes takes a very different path to get there.

![Book: A Philosophy of Software Design]({{ site.url }}/assets/A-Philosophy-of-Software-Design.jpg "A Philosophy of Software Design")

[A Philosophy of Software Design](https://www.amazon.com/Philosophy-Software-Design-2nd/dp/173210221X/ref=sr_1_1)

<!--more-->

This book focuses on the core challenge of software development: complexity.
It explores its causes, consequences, and how we can actively work to reduce it.

One of the key issues is that developers often focus too much on tactical programming — just getting features done —
while neglecting strategic programming, which is about designing the best possible structure for long-term maintainability.

Here are some of the book's key insights on reducing complexity:

- "Deep Modules": Encapsulating complexity inside well-designed modules with well designed, abstract interfaces helps keeping the rest of the system simple.
- Avoid unnecessary specialization: General-purpose code tends to be simpler. If specialized code is necessary,
  keep it separated from general-purpose code.
- "Define errors out of existence": Exceptions are special cases which make the code less obvious
- Comments are'nt a sign of failure: Good documentation is part of the definition of an abstraction,
  helping developers understand and manage complexity.
- Invest in good names: good names define abstractions which make the code more obvious
- Consistency matters more than we usually expect: following a consistent structure and style reduces cognitive load,
  making software easier to work with.

Here are a few of my favorite insights from the book:

> Developing incrementally is generally a good idea, but the increments of development should be abstractions, not features.

> The problem with test-driven development is that it focuses attention on getting specific features working, rather than finding the best design.

> To make code obvious, you must ensure that readers always have the information they need to understand it.
> You can do this in three ways.
> 
> The best way is to reduce the amount of information that is needed,
> using design techniques such as abstraction and eliminating special cases.
>
> Second, you can take advantage of information that readers have already acquired in other contexts
> (for example, by following conventions and conforming to expectations) so readers don't have to learn new information for your code.
>
> Third, you can present the important information to them in the code, 
> using techniques such as good names and strategic comments."

