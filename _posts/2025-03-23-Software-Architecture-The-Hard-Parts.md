---
layout: post
title: "Book Review: Software Architecture - The Hard Parts"
description:
tags: [book]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002, JL0005
---

I got caught by the title — and I’m glad I did.

The subtitle ("Trade-Offs in Distributed Architectures") might scare off some folks who don’t work with distributed systems.
That would be a mistake. This is one of the best books on software architecture I’ve read in the last 10 years.

![Book: Software Architecture - The Hard Parts]({{ site.url }}/assets/software-architecture-the-hard-parts.jpg "Software Architecture - The Hard Parts")

[Software Architecture - The Hard Parts](https://www.amazon.com/Software-Architecture-Trade-Off-Distributed-Architectures/dp/1492086894/ref=sr_1_1)

<!--more-->

What happens when there are no "best practices"?
That’s where the architect’s real job begins: making trade-offs.

The authors promote a structured approach to software architecture, starting as early as page 5 with Architecture Decision Records (ADRs) — arguably the most important design documentation in any serious project.
They don’t just explain what to do, but also why it matters and how to document your reasoning.

Another highlight: the book builds around a consistent example project.
This brings clarity to abstract ideas — and more importantly, to the trade-offs architects face in real-world systems.
It strikes a solid balance between theory and practice.

## What’s inside?

Part 1 focuses on modularity, coupling, and architectural decomposition.
It digs into how components relate, how to break down systems effectively, and how to handle data ownership.
It’s packed with reasoning tools and decomposition patterns that go far beyond distributed systems.

Part 2 tackles the complexities of coordination and communication between services.
It explores reuse strategies, data ownership boundaries, distributed transactions, and how to model workflows using patterns like sagas.
It also covers service contracts and how to manage change across teams and systems.

Even though the book focuses on distributed systems, I highly recommend it for any software architect — or anyone who wants to become one!
