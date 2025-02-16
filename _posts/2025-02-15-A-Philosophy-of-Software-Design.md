---
layout: post
title: "Book Review: Clean Craftsmanship: Disciplines, Standards, and Ethics"
description: What is software craftsmanship?
tags: [book]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002, JL0005
---

this book is about software complexity
and it sometimes takes a different path then "clean code"

![Book: Clean Craftsmanship: Disciplines, Standards, and Ethics]({{ site.url }}/assets/clean-craftsmanship.jpg "Clean Craftsmanship: Disciplines, Standards, and Ethics")

[Clean Craftsmanship: Disciplines, Standards, and Ethics](https://www.amazon.com/Clean-Craftsmanship-Disciplines-Standards-Ethics/dp/013691571X/ref=sr_1_1)

<!--more-->

# some key thoughts

- what is complexity, causes and consequences
  "complexity comes from an accumulation of dependencies and obscurities"
- tactical vs strategic programming
  - getting features done vs finding the best design
- one key technique to hide and encapsulate complexity is designing "deep modules"
- unnecessary specialization is a significant contributor to software complexity
  - general purpose code is easier, separate specialized code form general-purpose code
- define errors out of existence
  - exceptions are special cases which makes code harder to understand 
- design it twice, it is unrealistic to assume u get it with the first shot - incremental!
- documentation/comments are not and indicator of bad code (failures), they are part of definition of abstractions,
  and support managing system complexity
  - comments describe what isnt obvious from code, comments help to make code obvious to reader
- invest in good names, create abstractions and so reduce complexity
  (help to make the code more obvious)
- pay more attention to consistency - more important than often thought




"Developing incrementally is generally a good idea, but the increments of development should be abstractions, not features."


"The problem with test-driven development is that it focuses attention on getting specific features working, rather than finding the best design."



"To make code obvious, you must ensure that readers always have the information they need to understand it.
You can do this in three ways.

The best way is to reduce the amount of information that is needed,
using design techniques such as abstraction and eliminating special cases.

Second, you can take advantage of information that readers have already acquired in other contexts
(for example, by following conventions and conforming to expectations) so readers don't have to learn new information for your code.

Third, you can present the important information to them in the code, 
using techniques such as good names and strategic comments."


// TODO: what to take over for complexity analysis
