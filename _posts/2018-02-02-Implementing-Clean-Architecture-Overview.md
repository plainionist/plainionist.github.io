---
layout: post
title: Implementing Clean Architecture - An Overview
description: A brief overview on Uncle Bobs Clean Architecture you will find here.
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: JL0003
---

Let's briefly summarize what the Clean Architecture is ...

<img src="{{ site.url }}/assets/clean-architecture/Circles.png" class="dynimg" title="Layers of the Clean Architecture with Dependency Rule" alt="The Clean Architecture consists of multiple layers organized as circles while dependencies are only allowed from outer circles to inner circles. The inner circles contain the business logic. All details, devices and frameworks are in the outer circles."/>

<!--more-->

Wait! Let me rephrase: Let others briefly summarize what the Clean Architecture is about ...

(I really like the DRY principle so why should I repeat what others have put together nicely already ;-) )

- [Uncle Bob's original article on Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html).
  Of course this is the ultimate source. If you have not read it yet, do it right now!
- [Better Software Design with Clean Architecture](https://fullstackmark.com/post/11/better-software-design-with-clean-architecture).
  Mark nicely introduces into Clean Architecture by showing code right away.
  If you thought Uncle Bob's article was missing code then go through Mark's post.
- [clean-architecture-example](https://github.com/mattia-battiston/clean-architecture-example) is a GitHub
  project which describes Clean Architecture "by example". It has great documentation and of course code :)
- [Clean Architecture is Screaming](http://tidyjava.com/clean-architecture-screaming/) is another brief
  summary on the Clean Architecture. Short and very much focused on the core.
- [Clean Architecture: Standing on the shoulders of giants](https://herbertograca.com/2017/09/28/clean-architecture-standing-on-the-shoulders-of-giants/) 
  explains Clean Architecture by comparing it other approaches like Hexagonal and Onion Architectures. 

Brief enough? 

# Update 2018-04-08

- [a Clean Architecture in .Net](https://medium.com/@stephanhoekstra/clean-architecture-in-net-8eed6c224c50) is a great post about 
  realizing Clean Architecture in .Net in an Asp.Net environment - quite similar to my setup ;-) Definitively
  worth reading if you want to start implementing Clean Architecture.

{% include series.html %}
