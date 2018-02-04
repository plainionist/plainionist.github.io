---
layout: post
title: How to implement the Clean Architecture?
description: Uncle Bobs recent book Clean Architecture explains nicely how we should setup the architecture of our projects and which guidelines should drive our decisions. In theory this all sounds logical and easy but what happens when theory meets reality?
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
---

Did you enjoy reading Uncle Bob's [Clean Architecture](/Clean-Architecture)? 

![]({{ site.url }}/assets/CleanArchitecture.jpg)

I did! As with every book from Uncle Bob's it is motivating and inspiring, right?

So let's take his ideas and realize these in our projects to gain what he is promising!

But how do I start? 
How do I transform an existing code base - following a layered (web) architecture - into the Clean Architecture?

<!--more-->

In this blog series I will share with you the questions I had and the experiences I made when
implementing the Clean Architecture.

When I read about the Clean Architecture one of my projects already had an existing code base and 
has even been released several times. Its quality was basically ok but there was clearly 
improvement potential. So I decided that this would be a perfect pick for my experiment ...

## The project

The project of choise is an "agile backlog visualizer" - I will call it 
[Athena](https://en.wikipedia.org/wiki/Athena) in the context of this series ;-)

*Athena's* mission is to bring maximum transparency into the backlog of our agile teams.
It fetches all work items from our Team Foundation Server (TFS) and visualizes those
nicely on an intranet page to give us the views and reports we need ...

### The Backlog

<img src="{{ site.url }}/assets/clean-architecture/backlog.png" class="dynimg"/>

*Athena* shows our ranked backlog in a simple table with some filters.
It also applies the capacity of our teams to illustrate differnt cut-lines in different colors.

You may wonder why we have fruits in our backlog ... this is a different story which I may tell the other day ;-)

### Work balance

<img src="{{ site.url }}/assets/clean-architecture/work-balance.png" class="dynimg"/>

As we have multiple teams working on that backlog we want to know how the different teams are loaded.

### The Burndown

<img src="{{ site.url }}/assets/clean-architecture/burndown.png" class="dynimg"/>

And of course we want to track our remaining work using a burn down.

### Governance

We also have quite some conventions for our backlog, e.g.:

- each work item must have a parent (except the roots of course)
- each work item must have certain category tagging
- iteration paths within parent-child relationship have to be consistent

So we also have a page reporting violations to these conventions.

{% include series.html %}
