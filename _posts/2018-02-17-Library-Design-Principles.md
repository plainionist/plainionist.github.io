---
layout: post
title: Simple things should be simple - complex things possible
description: When designing a library - open source or commercial - we need to focus on multiple levels of abstraction to make simple things simple and complex things possible.
tags: [design]
excerpt_separator: <!--more-->
---

Just recently I found this great article about 
[Multiple levels of abstraction](http://tomasp.net/blog/2015/library-layers/) by Tomas Petricek.
I know the article is already a bit older but everything it talks about is very valid still.

The key message is this: 

- A library should expose its functionality as multiple layers of abstraction. 
- At the highest level, 80% of the scenarios can be handled with a single function call. 
- The next 15% can be implemented with a little more work and some lower level functions.
- The next 4% are the rare cases which require usage of the lowest level APIs.
- For the last 1%, you have to send a pull request!

Tomas continues giving three great examples to illustrate his point.

The article is very worth reading and we should keep the presented principle in mind when designing libraries.

<!--more-->

