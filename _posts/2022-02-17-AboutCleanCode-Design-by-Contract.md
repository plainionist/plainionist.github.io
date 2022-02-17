---
layout: post
title: "Assumptions aren't that bad in source code if ... "
description: |
  Assumptions are evil in source code, aren't these? Well, in this video we want to discuss how to turn bad assumptions
  in to good assumption using a simple, pretty old but very effective concept called "Design-by-Contract".
tags: [clean-code]
series: "AboutCleanCode"
excerpt_separator: <!--more-->
lint-nowarn: JL0003
---

Design-by-Contract is not only an easy way to find bugs earlier, it is also a great tactic to improve your code quality.
With Design-by-Contract we can document expectations and assumptions crystal clear in the source code itself and 
so improve changeability and maintainability!

How this works I explain in this video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/-3YfmHfZp-U" title="YouTube video player"
  frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen>
</iframe>

<!--more-->

## Design-by-Contract for C

Once upon a time a long time ago ...

... I worked in a project in the context of embedded systems. The project was developed in C programming language
and I wanted to use Design-by-Contract there as well. I found a project developed in Ruby which was designed a 
pre-processor for C. It used tags embedded in C API comments to generate runtime checks as well as 
[Doxygen](https://www.doxygen.nl/index.html) based documentation.

The pre-processor worked pretty well for my project and I even contributed to it.

I couldn't figure out what happened to the source code since RubyForge was shut down but here you can still download
the ruby gems:

- https://rubygems.org/gems/dbc/versions/2.0.0
- https://www.openhub.net/p/dbc


{% include series.html %}
