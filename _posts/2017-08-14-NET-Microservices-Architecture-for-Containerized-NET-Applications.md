---
layout: post
title: Architecture for Containerized .NET Applications
description: Looking for an introduction into practical microservices architectures in the .Net eco system? Here is where you should start.
tags: [book]
excerpt_separator: <!--more-->
lint-nowarn: JL0003
---

Since a while now I am interested in MicroServices architectures, their benefits and how to implement them.
Recently I came across [.NET Microservices: Architecture for Containerized .NET Applications](https://www.microsoft.com/net/download/thank-you/microservices-architecture-ebook)
in [The week in .Net](https://blogs.msdn.microsoft.com/dotnet/tag/week-in-net/) and immediate read it.

Overall I think it is a really good book about the topic!
Of course it focuses on Microsoft technologies but that's okay as this is anyway my primary playground ;-)
The ~300 pages gave me quite good and "deep enough" overview about the topics below and I really wonder whether I would
need another book at all. And this one is for free!
<!--more-->
- Docker and containers
- .Net Core vs .Net Framework
- Domain Driven Design principles
- Microservices architecture principles, e.g.:
  - Data sovereignty per microservice
  - Communication
  - Resiliency
- Deployment process
- Tacking business complexity using CQRS
- Health monitoring
- Security

If your are interested in Microservices definitively a must read!
And if the book still does not give you enough details you will find many "further readings" at the end
of each chapter.

PS: If you like reading code more than reading books the Microsoft guys have a full working example on 
[GitHub](https://github.com/dotnet-architecture/eShopOnContainers).

