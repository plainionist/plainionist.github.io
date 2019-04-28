---
layout: post
title: Implementing Clean Architecture - Cheat Sheet
description: 
tags: [clean-architecture]
series: "Implementing Clean Architecture"
excerpt_separator: <!--more-->
lint-nowarn: 
---


now that we discussed all layers including the main module here is the cheat sheet
this is how I would start applying clean architecture to a project (existing or green field)
(short compact - single page)

- first put all logic in the use cases
- make the controllers and presenters dump data converters  
  - request to request model, response model to response
- no logic in the views at all
- main is about composition only
- repositories and services focus on accessing details only
- entities get these rules which do not change across use cases


{% include series.html %}
