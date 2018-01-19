---
layout: post
title: MVVM pattern and dialogs
tags: [WPF]
excerpt_separator: <!--more-->
---

- every now and then i find discussions how to handle dialogs with MVVM in WPF
- this post i want to show what i think is the option which best fits to the ideas of MMVM

<!--more-->

core idea of MVVM is that u have a view which is dump and completely unknown by the ssyste
u have a view model as a bridge between the actuall (business ) model and the view.
view and viewmodel communicate with binding and commands

and then as model is mostly data centric very often there are services implementing usecases
and other business rules.

this pattern works very well if u have data centric app.

u load data from DB and show it in the UI (data binding)
u react to some commands for CRUD

but rich UI has more than that.

one example: dialogs --> how do those fit into the pattern?

there are many proposals out there
- ranging from "a little bit of code behind is not so bad"
- to: "use a dialog service"

personally i dont like any of these proposals.

i would favor keeping the "bizlogic" independent from UI ... and if a service opens a dialog this is broken.
and i dont like code-behind as ....

==> prism + interaction requests + popupwindow action

==> show sample with dialog in general

==> limitation ==> popupviewaction

==> open/file dialog are speciall - point to plainoin again


happy hacking :)
