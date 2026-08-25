---
layout: post
title: "Book Review: Harness Engineering: Designing the Systems That Make AI Coding Agents Reliable"
description:
tags: [book]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002, JL0005
---

Models are getting better and better. But they are still non-deterministic.
The harness is what turns them into mostly reliable AI coding agents.
So this is where we, as software engineers, need to focus now.

![Book: Harness Engineering: Designing the Systems That Make AI Coding Agents Reliable]({{ site.url }}/assets/harness-engineering.jpg "Harness Engineering: Designing the Systems That Make AI Coding Agents Reliable")

[Harness Engineering: Designing the Systems That Make AI Coding Agents Reliable](https://www.amazon.com/Harness-Engineering-Designing-Systems-Reliable/dp/B0GY9CVQLF/ref=sr_1_17)

<!--more-->

The book starts with the fundamentals: what exactly is harness engineering, and why is it important?
From there, it covers a broad range of topics such as context and state, workflows, verification, team behavior, handoffs, recovery, and governance.

One theme runs through the whole book: **Make knowledge, rules, and expected behavior explicit.**
The goal is to structure the context, processes, and environment around coding agents well enough
that more and more of the work can be automated reliably.

Interestingly, many of the suggested practices also make sense for human software teams:
clear responsibilities, explicit rules, defined processes, and automated verification.
With agents, however, these things need to be even more explicit so they can actually follow them.

This broad coverage is also the biggest value of the book for me.
It makes you aware of how many different aspects influence whether coding agents work reliably
and provides plenty of patterns and ideas to think about.

The downside is that it is not an easy read.
It is well structured, but often feels more like an instruction manual.
Examples are included throughout the book, but they often stay on an abstract, meta level.
I frequently understood *why* something was important, but missed the concrete example that creates the real “aha” moment.

I think the book would be much stronger with simpler explanations and one realistic sample project used consistently throughout.

So, important content and definitely worth knowing about - but probably not the best book on harness engineering. 🤷‍♂️

