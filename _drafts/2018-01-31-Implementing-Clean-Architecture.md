---
layout: post
title: How to implement the Clean Architecture?
description: Uncle Bobs recent book Clean Architecture explains nicely how we should setup the architecture of our projects and which guidelines should drive our decisions. In theory this all sounds logical and easy but what happens when theory meets reality?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
---

Did you enjoy Uncle Bobs [Clean Architecture](/Clean-Architecture) as much as I did?
Did you also felt motivated to start right away applying it to your running projects?

And did you also got stucked with basic questions after the first few changes?

"do u also run into the situation that u read a book and it is all cool and u want it try out and get stucked with questions? ..."
"here is how i currently try to "implement clean architecture""
"this ll be a series of posts sharing the experiences i have"


![](https://8thlight.com/blog/assets/posts/2012-08-13-the-clean-architecture/CleanArchitecture.jpg =200x)

share my experiences with applying clean architecture to practice

<!--more-->

i have currently two projects where i experiment with clean arch. one is kind of "agile backlog visualizer" ...

lets call it [Athena](https://en.wikipedia.org/wiki/Athena) ;-)


## Athena - Feature Overview

its purpose is basically taking all the planning stuff from tfs and visualize it in the web
to get maximum transparency on the backlog aspects we are interested in

core features
- show ranked backlog with filters & scoping
  - we explain later how finegranular we develop the usecases
- burn down
- team work distribution
- Governance
  - without parents
  - midding tagging (feature, arch, defect, ...)


  u may wonder why there are fruits in our backlog ... but this is a different story :)


{% include series.html %}
