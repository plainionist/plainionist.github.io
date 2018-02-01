---
layout: post
title: Clean Architecture
description: Brief review and summary of uncle bobs new book clean architecture.
tags: [book, clean-architecture]
excerpt_separator: <!--more-->
---

![]({{ site.url }}/assets/CleanArchitecture.jpg)

[Clean Architecture](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164/ref=sr_1_1?ie=UTF8&qid=1513596353&sr=8-1&keywords=clean+architecture)

As with every book by Uncle Bob it was fun to read it ... so I finished it in 2 days ;-)

As the title states and in contrast to his other recent books this book is really focusing on architecture. 
He clearly describes what architecture is about and what good architecture is, from his perspective.

Having read his [Clean Architecture article](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html) already 3 times for me 
there were no fundamental news in the book, but still gave me some more insights here and there.
<!--more-->
So if you haven't studied "The Clean Architecture" already in detail the book is a must read! 
(what else? - it is by Uncle Bob! ;-) )

## Update 2018-01-02

I felt I should add some more detailed summary about the book - for me later and maybe for others:

Part 1 gives a general introduction into design and architecture - a nice warm up :)

Part 2 gives a nice overview on the different programming paradigms - warm up second round :)

Part 3 discusses the SOLID principles briefly - not as deeply as in [Agile Principles, Patterns, and Practices in C#](https://www.amazon.com/Agile-Principles-Patterns-Practices-C/dp/0131857258/ref=sr_1_1?ie=UTF8&qid=1514894364&sr=8-1&keywords=agile+patterns+and+practices)
but clearly deep enough to understand them. If you have read  [Agile Principles, Patterns, and Practices in C#](https://www.amazon.com/Agile-Principles-Patterns-Practices-C/dp/0131857258/ref=sr_1_1?ie=UTF8&qid=1514894364&sr=8-1&keywords=agile+patterns+and+practices)
you can rush through that part.

Part 4 discusses component principles which are also like the SOLID principles deeply discussed already in 
[Agile Principles, Patterns, and Practices in C#](https://www.amazon.com/Agile-Principles-Patterns-Practices-C/dp/0131857258/ref=sr_1_1?ie=UTF8&qid=1514894364&sr=8-1&keywords=agile+patterns+and+practices).
So again, if you have read that book already you can rush through that part as well.

Part 5 finally is - I fell - the core part of the book. The first chapters prepare the path to the "Clean Architeture" by highlighting
the importance of keeping options open by decoupling independent aspects and drawing borders between them - e.g. design a system device independent".
It continues motivating "framework independence" as frameworks are also just details, instead the business and its domaian should shape 
the landscape of an architecture.
Finally chapter 22 and the subsequent chapters describes the "Clean Architecture" and how it should be put into context.

Part 6 completes the picture on the "Clean Architecture" by looking at "the details" like the database and the web.


And now it is time to realize the "Clean Architecture" in the next project ;-)

