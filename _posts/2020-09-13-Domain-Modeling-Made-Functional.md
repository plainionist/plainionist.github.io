---
layout: post
title: "Book Review: Domain Modeling Made Functional"
description: |
    Are you interested in Domain Driven Design (DDD) and do you want to learn how to effectively use it in your projects?
    Do you want to know why Functional Programming is a great fit for DDD? 
tags: [book]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

Unless you are not completely new to our profession I am pretty sure you already heard
about "Domain Driven Design" (DDD). But maybe you think that DDD requires quite some effort which is only worth it
in big software projects and that it has to be done in the beginning of a project to be effective?

This is not necessarily true ...

Here is a book which nicely demonstrates that DDD and Functional Programming (F#) are two great tools to tackle
software complexity in any software project, even in a small pet project:

![Book: Domain Modeling Made Functional: Tackle Software Complexity with Domain-Driven Design and F#]({{ site.url }}/assets/ddd-made-functional.jpg "Book: Domain Modeling Made Functional: Tackle Software Complexity with Domain-Driven Design and F#")

[Domain Modeling Made Functional: Tackle Software Complexity with Domain-Driven Design and F#](https://www.amazon.com/Domain-Modeling-Made-Functional-Domain-Driven-ebook/dp/B07B44BPFB/ref=sr_1_1?crid=34GG05PMNU3AU&dchild=1&keywords=domain+modeling+made+functional&qid=1599973736&sprefix=domain+modeling+%2Caps%2C680&sr=8-1)

<!--more-->

In this book Scott Wlaschin - the author of [F# for fun and profit](https://fsharpforfunandprofit.com/) - does a great job
to demystify DDD and Functional Programming (FP). He uses simple and practical examples and avoids any kind of complex,
scientific or scary terminology. This way you can easily follow his thoughts even if you have no previous experience
in DDD, FP or F#.

The book starts with a great statement:

> As a developer, you may think that your job is to write code
>
> I disagree. A developer's job is to solve a problem through software, and coding is just one aspect of
> software development. Good design and communication are just as important, if not more so.

Scott then jumps right into DDD explaining the importance of a shared model, how to understand the domain through
business events and the need to partition the domain into subdomains (bounded contexts). He completes his
introduction by explaining the need for an "Ubiquitous Language".
By the end of the first chapter you already learned all the key aspects of DDD.

The next chapter teaches you how to learn about the domain itself which is key to ensure that your software system
will finally solve the right problem in the right way.

Chapter 3, the last chapter without code, talks about the important aspects of a domain and business focused architecture.
You will find similarities to "Clean Architecture" and the "Onion Architecture".

The remaining ten chapters are very practical by focusing on how to model the domain using concepts of FP such as 
types and functions. Each concept is introduced step by step using simple and practical examples. To complete the journey 
Scott shows how to incorporate such "nasty things" as error handling and IO into a DDD focused software system.

The last chapter demonstrates that DDD works well with agile software development by showing how to
"Evolving a Design and Keeping It Clean".

## Conclusion

This book is definitively on my list of "recommended readings" for senior software developers.

