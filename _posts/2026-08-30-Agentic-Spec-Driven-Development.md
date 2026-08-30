---
layout: post
title: "Book Review: Agentic Spec-Driven Development"
description:
tags: [book]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002, JL0005
---

Spec-driven development sounds like writing a big specification before starting to code.

This book is about something much more interesting: how to set up your project and development process so that AI agents
can do most of the actual work — guided by clear intent, context, guardrails, and verification.

![Book: Agentic Spec-Driven Development: A Practical Method for Using AI to Build Complete Specifications for Software, Products, and Knowledge Work]({{ site.url }}/assets/agentic-spec-driven-development.jpg "Agentic Spec-Driven Development: A Practical Method for Using AI to Build Complete Specifications for Software, Products, and Knowledge Work")

[Agentic Spec-Driven Development: A Practical Method for Using AI to Build Complete Specifications for Software, Products, and Knowledge Work](https://www.amazon.com/Agentic-Spec-Driven-Development-Practical-Specifications-ebook/dp/B0GX38ZZYT/ref=sr_1_1)

<!--more-->

## More than spec-driven development

The core idea is that when AI writes the code, engineering judgment does not disappear.
It moves into the specification.
You define the intent, constraints, architecture, and expected behavior. The agent figures out how to implement it.

But this does not mean writing a huge specification upfront. The book acknowledges that you cannot think of everything in advance.
Specifications evolve iteratively as implementation exposes missing decisions, conflicts, and edge cases.

That makes the approach much more pragmatic than it might sound from the title.

## Intent over instruction

A key idea of this book is *intent over instruction*.
Tell the agent what you want to achieve, why you want it, which constraints matter, and where the guardrails are.
Then let it reason about how.
You do not need to dictate the answer. You need to give AI enough context to find it.

The same principle applies to rules. Do not try to prevent every possible failure with more and more natural-language instructions.
Keep rules generic and use deterministic mechanisms such as scripts, tests, and architecture fitness functions when you need deterministic behavior.

## A practical guide to agentic development

What makes the book especially useful is that it goes far beyond specifications.
It covers prompts, durable rules, token efficiency, sessions and memory, project knowledge, artifacts, auditability,
tests, architecture fitness functions, and the overall development process.

It even provides a complete sample setup with ready-to-use rule files.

So you can either try the author's method as it is or take the underlying concepts and integrate them into your own setup.
If you are new to agentic development, this is a great foundation because it explains not only how to write specs,
but how to structure an entire project around AI agents.

And if you already work extensively with coding agents, it is still worth reading because it makes you reflect on your own setup.

Despite the title, I would almost call this a book about harness engineering.
And that is exactly why I found it worth reading.
