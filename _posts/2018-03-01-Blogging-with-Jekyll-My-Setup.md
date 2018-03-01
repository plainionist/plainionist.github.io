---
layout: post
title: Blogging with Jekyll - Legal and privacy
description: Interested how others set up an efficient blogging environment with Jekyll? Here is mine!
tags: [jekyll, blogging]
series: "Blogging with Jekyll"
excerpt_separator: <!--more-->
---

After having blogged with Jekyll for some while now I have found some convenient setup which makes me feel quite effective.
Today I want to share my setup with you. Maybe some aspects inspire you, maybe you want to leave a comment about your own 
setup to inspire me?

<!--more-->

Being primarily a .Net developer it was almost obvious for me to take Visual Studio as editor. I have created a Visual Studio 
project which contains all my posts and also all layout files, includes and configuration files. This way I have everything
right at hand in the Solution Explorer.

Now being enabled to write posts I have added a few add-ons to make the setup effective ...

## How to preview the post?

As I am writing from a Windows client, installing Jekyll locally and run it to preview the post is almost not an option ... at least
not unless I switch to Windows 10. I have checked various installation tutorials and concluded that it is for me not worth
the effort. I will give an update on that aspect once I gave the Windows 10 setup a try.

What other alternatives for previewing the posts do I have?

I finally decided to install this [MarkdownEditor](https://marketplace.visualstudio.com/items?itemName=MadsKristensen.MarkdownEditor).
It gives me a nice preview right in Visual Studio. It will not produce exactly the same output as Jekyll on GitHub pages will do but
for me it is sufficient enough.

## How to include diagrams?

I am a big fan of simple and pragmatic solutions, also when it comes to diagrams. I really don't like UML and I hate the 
usability of Enterprise Architect - should I actually even call it "usability"? ...

Luckily I came across [draw.io](http://draw.io/). This is exactly the tool I needed. Simple and pragmatic. I can use it 
from the browser for a quick sketch or as a desktop application for bigger projects. I can just draw boxes and circles - my
preferred way of drawing :) ... but I can also choose some of the endless standard shapes - even UML ;-)

## What about spell checking?

To be honest I wrote my first blog posts without any spell checking support. This was a bad idea as I learned later on when I
spell-checked my older posts. 

Typos happen. Errors happen. Quality matters. Do NOT blog without spell checking support.

I have finally installed the [VSSpellChecker](https://github.com/EWSoftware/VSSpellChecker/releases). When I tried it first it did 
not work for markdown files but in combination with the MarkdownEditor it works perfectly fine.

## Plainion.JekyllLint

After having installed the spell checker I wondered what other "quality assurance" I could use to write high quality posts. 
I did some research and found some "linter" projects but none which really fits my needs. So I have started my
[own one - Plainion.JekyllLint](https://github.com/plainionist/Plainion.JekyllLint). It is still in an early stage but helped
me already to find some issues. I will add more rules to it whenever I learn something new about writing, blogging and SEO.

To install Plainion.JekyllLinter just download the latest release and extract it somewhere. Then add the following snippet
to your Visual Studio project:


```xml
<Target Name="AfterBuild">  
  <Exec Command="D:\bin\Plainion.JekyllLint\Plainion.JekyllLint.exe _posts" />
</Target>  
```

If you now build your project you will see errors and warnings in your Visual Studio "Error" tab!

## Plainion.CI

Last but not least I am using [Plainion.CI](https://plainionist.github.io/Plainion.CI/) to publish my posts. Certainly a simple
Git client with "one click commit and publish" would do the trick as well but I am already used to Plainion.CI from my other 
open source projects and in that setup Plainion.JekyllLint runs before publishing and so ensures that I don't publish posts with
"errors" inside.

## Now it is about you ...

How does your IBE (Integrated Blogging Environment) look like?

{% include series.html %}
