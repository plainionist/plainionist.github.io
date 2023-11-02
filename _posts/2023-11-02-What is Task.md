---
layout: post
title: "What actually is Task<T>?"
description: |
    When seeing Task<T> in code, most developers think of threads. But that is actually not quite correct.
    Task<T> is not coupled to multi-threading at all.
tags: [design]
excerpt_separator: <!--more-->
lint-nowarn: JL0003, JL0002
---

If you see such source code, do you immediately think of multi-threading?

<script src="https://gist.github.com/plainionist/86ab9cbd537231998377be8c4e447df7.js?file=Intro.cs"></script>

What if I would tell you that ```Task<T>``` is nothing but a fancy delegate which is not coupled 
to threads at all?

<!--more-->

Let's start with some very simple component. It provides a single API which accepts a request object,
does some computation and returns a response object.

<script src="https://gist.github.com/plainionist/86ab9cbd537231998377be8c4e447df7.js?file=Step1.cs"></script>

Such an API design obviously forces the caller to "actively" wait for the response to be available
in order to be able to continue the processing. This design couples the control flow of processing a request and
the control flow of handling its response.

Now let's assume you want to decouple these two parts of the control flow e.g. because the processing of the request
is delayed because of some queue or it runs in a different thread or process or even on a different machine or in the cloud.

One common way to achieve this in .NET is using events.

<script src="https://gist.github.com/plainionist/86ab9cbd537231998377be8c4e447df7.js?file=Step2.cs"></script>

(Hint: This sample code is kept as simple as possible, event deregistration is skipped for this reason.)

An alternative to .NET events would be passing a simple delegate which we could also call 
[continuation](https://en.wikipedia.org/wiki/Continuation).

<script src="https://gist.github.com/plainionist/86ab9cbd537231998377be8c4e447df7.js?file=Step3.cs"></script>

In order to improve the readability of this design, let's get it closer to the initial request-response
semantic by inventing a small class called ```Promise``` which is returned from the ```Execute``` API 
and which is then used to hook up the continuation.

<script src="https://gist.github.com/plainionist/86ab9cbd537231998377be8c4e447df7.js?file=Step4.cs"></script>

Let's also add error handling to this design.

<script src="https://gist.github.com/plainionist/86ab9cbd537231998377be8c4e447df7.js?file=Step5.cs"></script>

The implementation of the ```IComponent``` interface would use the ```Promise``` class like this:

<script src="https://gist.github.com/plainionist/86ab9cbd537231998377be8c4e447df7.js?file=Step5-Component.cs"></script>

Now this design is already pretty close to the one of ```Task<T>``` which shows us that ```Task<T>``` is actually
nothing but an implementation of a [promise](https://en.wikipedia.org/wiki/Futures_and_promises) provided by .NET.

But what about ```async/await```? Isn't that what makes ```Task<T>``` so powerful?
Well, actually ```async/await``` is an compiler feature which is completely independent from ```Task<T>``` and 
the Task Parallel Library (TPL). To prove this, let's enable it for our custom ```Promise``` implementation as well.

<script src="https://gist.github.com/plainionist/86ab9cbd537231998377be8c4e447df7.js?file=Step6.cs"></script>

Therefore, we just need to provide an API (e.g. an extension method) called ```GetAwaiter``` which returns a type
or an interface which has the following properties:

- it implements ```INotifyCompletion```
- it provides an API called ```IsCompleted```
- it provides an API called ```GetResult```

And with this the compiler allows us to use ```async/await``` for the ```Execute``` API as usual.

<script src="https://gist.github.com/plainionist/86ab9cbd537231998377be8c4e447df7.js?file=Step6-Client.cs"></script>

In essence, that's exactly how ```Task<T>``` works!

<script src="https://gist.github.com/plainionist/86ab9cbd537231998377be8c4e447df7.js?file=Step7.cs"></script>


Quod erat demonstrandum.

Full source code: https://github.com/plainionist/AboutCleanCode/tree/main/Promise

