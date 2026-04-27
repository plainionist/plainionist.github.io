---
layout: post
title: "Book Review: Thinking Fast and Slow"
description:
tags: [book]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002, JL0005
---


If you believe humans are mostly rational and think logically,  
then this book is for you 😉

![Book: Thinking Fast and Slow]({{ site.url }}/assets/thinking-fast-and-slow.jpg "Thinking Fast and Slow")

[Thinking Fast and Slow](https://www.amazon.com/Thinking-Fast-Slow-Daniel-Kahneman/dp/0374533555/ref=sr_1_1)

<!--more-->

## The key idea

The core thesis of the book is that our thinking can be described as two systems.

**System 1** is fast, automatic, intuitive, and effortless. It helps us recognize patterns, react quickly, read emotions, and make instant judgments.
But it also jumps to conclusions.

**System 2** is slow, deliberate, analytical, and effortful. It helps us calculate, reason, compare options, and question assumptions.
But it is also lazy.

Most of the time, System 1 creates an answer first, and System 2 only checks it if there is enough reason or energy to do so.
That means we often feel rational while we are actually just justifying our first impression.
And because System 1 is very good at creating coherent stories from incomplete information, wrong conclusions can still feel obviously true.

This is not just interesting psychology; it has very practical consequences for how we explain ideas,
make decisions, give feedback, and collaborate as software engineers.


## 5 practical takeaways

### 1. Make the missing context visible

One of the strongest ideas in the book is: **What you see is all there is.**

People reason based on the information available to them, not based on all the information that would be relevant.
And definitely not based on all the context you have in your head.

This matters a lot in collaboration. When people disagree with your proposal, they may not be irrational.
They may simply see a different part of the problem.

So before arguing for a solution, make the context visible.
What problem are we solving? What constraints do we have? What alternatives did we consider? What information influenced our conclusion?

Better context often creates better alignment.

### 2. Do not trust confidence too much

The book shows again and again that confidence can be misleading.

A conclusion can feel correct simply because the story in our head is coherent.
But a coherent story is not the same as a true story.

This is important in discussions, planning, and architecture decisions.
Someone may sound very confident and still be wrong. And that someone may be you.

A useful habit is to ask: **What evidence would change my mind?**

This moves the discussion away from confidence and toward verification.

### 3. Use framing consciously

Kahneman shows that the way information is framed influences how people judge it.

The same facts can lead to different reactions depending on how they are presented.
That does not mean we should manipulate people. It means we should be responsible with framing.

In collaboration, this means we should be careful how we present problems, risks, and proposals.

“Refactoring this module will take two weeks.”

creates a different reaction than:

“Every new feature in this area currently takes longer because this module is hard to change.
Spending two weeks on refactoring should reduce the cost of all future changes in this area.”

Same topic. Different frame. Different understanding.

Good communication is not only about being logically correct. It is also about helping others see the point clearly.

### 4. Make comparison easy

System 2 is slow, effortful, and lazy.

So if we want people to think carefully, we should not make the thinking unnecessarily hard.

This matters when we present options. Long verbal explanations are difficult to compare.
People will often anchor on the first option, remember the most vivid argument, or simply follow the most confident person in the room.

A small decision table is often better:

- option
- benefit
- cost
- risk
- when it works
- when it fails

This does not make the decision automatic. But it makes careful thinking easier.

### 5. Slow down important decisions

Fast thinking is useful. We need intuition, experience, and quick judgment.

But the book is also a warning: intuition is not reliable everywhere.

For important decisions, we should deliberately slow down. Especially when the decision is expensive, risky, hard to reverse, or emotionally loaded.

Write the decision down. List alternatives. Separate facts from assumptions. Ask what could go wrong. Use an ADR when the decision matters.

Not because documents are magic.

But because writing forces System 2 to participate.

## Conclusion

**Do not believe everything you think.** 😉
