---
layout: post
title: "Book Review: Clean Architecture"
description: Brief review and summary of uncle bobs new book clean architecture.
tags: [book, clean-architecture]
excerpt_separator: <!--more-->
lint-nowarn: JL0003
---

![Book: Clean Architecture]({{ site.url }}/assets/CleanArchitecture.jpg "Book: Clean Architecture")

[Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164/ref=sr_1_1?ie=UTF8&qid=1513596353&sr=8-1&keywords=clean+architecture)

As with every book by Uncle Bob it was fun to read it ... so I finished it in 2 days ;-)

As the title states and in contrast to his other recent books this book is really focusing on architecture. 
He clearly describes what architecture is about and what good architecture is, from his perspective.

Even if you have read his [Clean Architecture article](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)
already I strongly recommend reading the book. It goes much more into details and so brings more insights
relevant when you try realizing Clean Architecture in one of you next projects ...

<!--more-->

## Brief Summary

Part 1 gives a general introduction into design and architecture.

Part 2 gives a nice overview on the different programming paradigms.

Part 3 discusses the SOLID principles briefly - not as deeply as in [Agile Principles, Patterns, and Practices in C#](https://www.amazon.com/Agile-Principles-Patterns-Practices-C/dp/0131857258/ref=sr_1_1?ie=UTF8&qid=1514894364&sr=8-1&keywords=agile+patterns+and+practices)
but clearly deep enough to understand them. If you have read  [Agile Principles, Patterns, and Practices in C#](https://www.amazon.com/Agile-Principles-Patterns-Practices-C/dp/0131857258/ref=sr_1_1?ie=UTF8&qid=1514894364&sr=8-1&keywords=agile+patterns+and+practices)
you can rush through that part.

Part 4 discusses component principles which are also like the SOLID principles deeply discussed already in 
[Agile Principles, Patterns, and Practices in C#](https://www.amazon.com/Agile-Principles-Patterns-Practices-C/dp/0131857258/ref=sr_1_1?ie=UTF8&qid=1514894364&sr=8-1&keywords=agile+patterns+and+practices).
So again, if you have read that book already you can rush through that part as well.

Part 5 finally is - I fell - the core part of the book. The first chapters prepare the path to the "Clean Architecture" by highlighting
the importance of keeping options open by decoupling independent aspects and drawing borders between them - e.g. design a system device 
independent". It continues motivating "framework independence" as frameworks are also just details, instead the business and its domain 
should shape the landscape of an architecture.
Finally chapter 22 and the subsequent chapters describes the "Clean Architecture" and how it should be put into context.

Part 6 completes the picture on the "Clean Architecture" by looking at "the details" like the database and the web.

## How to implement Clean Architecture?

Of course the book contains practical examples but it doesn't go too much into detail. When you try to 
realize the Clean Architecture the first time you probably will have still questions which I try to 
answer in my blog series on [How to implement the Clean Architecture?](/Implementing-Clean-Architecture).