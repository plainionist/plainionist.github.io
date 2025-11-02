---
layout: post
title: "The Dark Side of Pull Requests for Managers"
description: |
    Pull Requests (PRs) were invented for Open Source projects.
    Work in commercial teams is fundamentally different.
    PRs were not made for this.
    They optimize for control and gatekeeping, while commercial teams optimize for flow and delivery speed without sacrificing quality.
tags: [book]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002, JL0005
---

## Abstract

Pull Requests (PRs) were invented for Open Source projects.
They allow "untrusted contributors" to suggest changes and ask "trusted committers" to pull them in.
Committers review these requests on their own schedule, leave comments, demand changes, or simply reject the "pull request" altogether.
For this reason, PRs work extremely well in Open Source.

Work in commercial teams is fundamentally different.
Team members collaborate long-term, share goals, and trust each other.
Changes are not "suggested" but planned - it’s rarely a question of whether but how something gets done.

PRs were not made for this.
They optimize for control and gatekeeping, while commercial teams optimize for flow and delivery speed without sacrificing quality.


> Using pull requests for code changes by your own team members is like having your family members go through an airport security checkpoint to enter your home.
> It’s a costly solution to a different problem.
>
> Kief Morris, Why your team doesn’t need to use pull requests


<!--more-->

## Motivation

Many great articles and YouTube videos already discuss this topic - see the references -
including well-known voices like Dave Farley, author of "Continuous Delivery".

This article provides a concise overview of the real-world consequences of PRs for commercial teams,
written for managers and team leads.


## Setting the Stage

Before getting into the arguments, let’s clarify a few facts.

### Git ≠ PR

Git doesn’t require Pull Requests.
It’s a version control system that works perfectly well without them.

### Code Review ≠ PR

Code reviews are valuable for code quality, knowledge sharing, and finding bugs.
PRs are not required to practice code reviews.

### Working incrementally

Developers rarely complete a feature in a single commit.
Bug fixes might fit in one, but most features need multiple commits to be completed.


## How Pull Requests work

Pull Requests always require a branch.
A PR can contain one or many commits.
It is a "gated submission": changes are integrated into mainline only after all gates have passed - usually automated tests and code review.
A developer cannot start a second PR on the same branch; further changes will update the existing PR.


## The Arguments

![]({{ site.url }}/assets/flow.drawio.png)

### Passing gates takes time

Running build pipelines takes time. The larger the codebase, the longer it takes.
Reviews take time as well. A PR may wait hours or even days for review.
If reviews are expected immediately, context switching creates new problems (see "reviewing code immediately").

### Continuing on the same task during PR verification is difficult

Feature work usually requires several changes.
While one PR is under review, continuing the same story requires creating a new branch from the first one.
This adds effort and becomes messy if the first PR fails validation.

### Bigger PRs

To avoid this overhead, developers tend to group several changes together and open one PR when the story is finished.
As a result, a single PR often contains multiple logical changes.

### Longer-living branches

Producing more changes takes more time, which means feature branches live longer.

### Delayed Feedback

The longer branches live, the longer feedback gets delayed - from tests, from teammates reviewing the design,
and from product owners or users who can only verify that the right feature was built after integration.

### Higher reluctance to apply global refactorings

The longer branches live, the higher the reluctance to apply refactorings that span multiple files
or affect public APIs because developers fear the effort of complex merges.
Skipping refactoring means not paying back technical debt, which means lowering code quality.

### Bigger integration chunks

The bigger the PR, the bigger the chunk of code changes that is integrated at once into mainline.

### Higher merge complexity

The bigger the chunks that get integrated, the higher the risk of complex merges.
Complex merges cause more effort.

### Higher risk of bugs

The more complex the merge, the higher the risk of mistakes and regressions, reducing product quality.

### Harder to analyze build failures

The bigger the chunks that get integrated, the harder it is to figure out which change actually caused the issue
when the build fails which increases the effort required to finish the feature.

### Higher reluctance to reject changes

Bigger PRs also raise the mental barrier for reviewers to reject changes and demand rework.
In the end, it’s one team aiming for the same goal - and it’s hard to tell a teammate to throw away a week’s work.

### Reduced need to work incrementally

It is common sense in the software industry that working incrementally leads to less waste and better results.

But when accepting bigger PRs, developers lose both the incentive and the skill to work in small increments.
Over time, this leads to even larger PRs.

### Harder reviews

Reviewing large, multi-purpose changes takes longer and increases cognitive load.
Feedback becomes shallow, resulting in a higher risk of technical debt or even new bugs.

### Higher barrier for minor cleanups (Boy Scout Rule)

When integrating small improvements requires the overhead of a PR, developers skip them.
Over time, this lowers code quality.


## But what about ...

### ... working in isolation?

Feature branches may feel productive because they isolate work, but that’s a local optimization.
It benefits an individual’s focus at the expense of team flow.

### ... running CI builds on feature branches?

Running pipelines on branches is possible if the build farm scales, but it provides partial feedback at best.
It shows whether a branch works in isolation - not whether it integrates with everyone else’s changes.
By definition, that’s not Continuous Integration.

### ... reviewing code immediately?

Immediate reviews reduce PR delay but force frequent context switches for reviewers.
Each context switch costs time (~ 15 minutes) and mental focus.
More PRs mean more context switches, lowering overall team productivity.

### ... short-lived branches?

Short-lived branches reduce some problems but don’t resolve the fundamental ones.
Each PR still acts as a gatekeeper, breaking the flow.
More PRs simply multiply the overhead.


## Continuous Integration

Continuous Integration (CI) is the alternative that optimizes for flow and productivity.

By default, there are no feature branches.
All developers commit directly to mainline and every commit triggers the build pipeline.
Tests run after each commit, and code reviews happen asynchronously. No gatekeeping!

The key point is that integration does not mean release.
That’s why changes are integrated even before a feature is completed.
Still, every change getting integrated must meet production quality.

Builds can fail - that’s normal.
When mainline breaks, fixing it becomes the top priority ("green-to-green" principle).
Because integrations are small, fixes are quick and rollbacks low-risk.

Even if the latest commit is broken, the last green build is always releasable.
Release branches can be created from there if needed.

CI enforces small, incremental changes, solving all the problems introduced by PR-based workflows.
It aligns developer behavior with team performance rather than individual convenience.


## Are branches and PRs entirely bad?

Of course not - they have legitimate uses.

### Prototypes

When exploring a problem using a prototype, it’s fine to work on a branch.
The branch helps manage work-in-progress, but its purpose is learning, not integration.
Once a solution is known, development restarts cleanly on mainline.

### Integrating Dependencies

External dependencies come from outside the team and are therefore trusted less.
Using a gated submission for validation before integration is often appropriate.


## But why does everyone else do PRs then?

Many teams adopted PRs because they are the default in popular tools like GitHub and GitLab.
Others assume it’s the standard way to use Git.
Few stop to question whether a model designed for untrusted collaboration fits a trusted commercial team.


## Conclusion

The key question is: what do you want to optimize for?

If your goal is control, use PRs.
If your goal is flow and productivity, adopt Continuous Integration.


## References

- [Are Pull Requests Holding Back Your Team?](https://medium.com/better-programming/are-pull-requests-holding-back-your-team-e8aec48986c2)
- [You Might Be Better Off Without Pull Requests](https://hamvocke.com/blog/better-off-without-pull-requests/)
- [On the Evilness of Feature Branching](https://thinkinglabs.io/articles/2021/04/26/on-the-evilness-of-feature-branching.html)
- [Why Pull Requests Are A BAD IDEA](https://www.youtube.com/watch?v=ASOSEiJCyEM)
- [Why CI is BETTER Than Feature Branching](https://www.youtube.com/watch?v=lXQEi1O5IOI)
- [I’ve Found Something BETTER Than Pull Requests](https://www.youtube.com/watch?v=WmVe1QrWxYU)
- [Git Flow Is A Bad Idea](https://www.youtube.com/watch?v=_w6TwnLCFwA)
- [The Dark Side of Pull Requests](https://youtu.be/aVcfv9uZPpQ)
- [The Death of Continuous Integration](https://www.infoq.com/presentations/death-continuous-integration/)
- [Continuous Integration - Flow Beats Control!](https://youtu.be/GfGEtCcWUi4)
- [DORA](https://dora.dev/research/2024/dora-report/)
- [Accelerate](https://www.amazon.com/Accelerate-Software-Performing-Technology-Organizations/dp/1942788339/ref=sr_1_1?dib=eyJ2IjoiMSJ9.jemhPhEOvE34eI5T6fVSy6wFBpNPjjBfe3kwxZx76OaYiUzDk_wk895lrRWwY6HIQ0je12lhE6TcAVUA4586GZaPmZt1olHYZ0MTmw6C2EgF2NkPfWx1r3i2oWWk2BDQXDQfRCoHxWIV8ML8Z2eNNORbMWi2JzJFTyit6xsxFEoeivek_tLAVhtXMr6kga89XiSNYVZ9VMTwV9D2cVpnfHyUkibjWIVuL5WxTtEcJdM.c_xcXYbe2_zgsL6vOgYSOY0pQTCNTX5wma1-lzBYNww&dib_tag=se&keywords=accelerate&qid=1762072349&sr=8-1)
- [Continuous Delivery](https://www.amazon.com/Continuous-Delivery-Deployment-Automation-Addison-Wesley/dp/0321601912/ref=sr_1_1?crid=30S5B45LGZKU6&dib=eyJ2IjoiMSJ9.itY9oApzOE7dyjDO0Qqy4ck16gK5272nG-QCzwli1F01vVhqFAJZpqOHz0ikilr40Rl1tvv-BlCC0bx5uQ26S_vTsxdSnbObYYe9WlumcKefqH345eo1ZIlGDtxTk3UhbcGFZN9-taZWlswTg0gan4PhqL94XQOOUfGMflLcAdzXuhcs7lUCpme0W0d4vbISWRO3Vl6pCq7SAL6oIEj4unsHsWFvtSQm7168hINMMZw.0dhWm5CfAo2sTBfHGUs8XfbNKd2E1JWVeWLaofbluD8&dib_tag=se&keywords=continuous+delivery&qid=1762072356&sprefix=continuous+deliver%2Caps%2C190&sr=8-1)
