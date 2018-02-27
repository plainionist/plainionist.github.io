---
layout: post
title: MVVM pattern and dialogs
description: Open dialogs with WPF, MVVM and Prism using InteractionRequests and PopupWindowAction.
tags: [WPF]
excerpt_separator: <!--more-->
lint-nowarn: JL0003
---

Every now and then I come across questions on how to handle dialogs in MVVM pattern with WPF.
Honestly, most of the solution proposals I don't like as they - from my perspective - somehow
"violate" the MVVM pattern.

<!--more-->

## The core of MVVM

The core idea of MVVM is three fold.

First, there is a view which is as dumb as possible. It must not contain any logic which e.g. would require a unit test. 
The view is just about nicely interacting with the user.

Second, there is a (domain)model. It is data centric, represents the entities of the domain and models the business rules.

Third, there is the view model which acts as a bridge between the view and the model. It converts the data of the model into 
a format which is most convenient for the view. It converts the "events" of the view into actions most convenient for the model.

In most MVVM frameworks the view "notifies" the view model using "commands" and the view model updates the view through "data binding".

Very often you can also find "services" which implement use cases - business logic which does not fit into the model.

As you probably have guessed the MVVM pattern fits nicely for data centric applications, applications which load data from a DB, show it
to the user through some nice UI and provide some "commands" from common CRUD operations.

But most "rich UI" applications are more than that. There are animations, gestures, drag & drop and other nice UX features which make
the usage of the application "a pleasure for the user".

## And sometimes there are dialogs ...

The challenge of dialog handling in the context of MVVM pattern is that a dialog needs to be shown driven by 
a state change in the view model while the view model does not want to know anything about views (and a dialog obviously is a view/window).

If you search the Internet - or most probably Stackoverflow - for that question you will find many proposals how to address 
this challenge. The solution proposals range from "just open from code behind" to "have a dialog service opening the dialog".

Personally I don't like any of these proposals. I don't want any non-view code to depend on view code. I don't want the view model or a service
knowing anything about the view. I want to keep the view logic separated.

I also don't like "code-behind" code in MVVM-driven applications because mostly this moves the view model out of the control. I want the view
as declarative as possible. I don't to unit test the view so I don't want to have it any logic.

## InteractionRequests to the rescue

From my perspective the best way to handle dialogs in MVVM pattern is using "interaction triggers" from System.Windows.Interactivity and
"interaction request trigger" from [Prism library](https://github.com/PrismLibrary/Prism). The approach works as follows:

You define a interaction request in the XAML of the view from where you want to open a dialog

```Xaml
<i:Interaction.Triggers>
  <prism:InteractionRequestTrigger SourceObject="{Binding ConfirmationRequest, Mode=OneWay}">
    <prism:PopupWindowAction/>
  </prism:InteractionRequestTrigger>
</i:Interaction.Triggers>
```

You an "InteractionRequest" from view model as "SourceObject" like this:

```C#
public InteractionRequest<IConfirmation> ConfirmationRequest { get; private set; }
```

You can use the interaction request from the view model to open the dialog:

```C#
private void OnShowConfirmation()
{
    var confirmation = new Confirmation();
    confirmation.Title = "Really?";
    confirmation.Content = "Here goes the question, doesn't it?";

    ConfirmationRequest.Raise( confirmation, c => this.Response = c.Confirmed ? "yes" : "no" );
}
```

This example would open a default confirmation dialog with the specified title and content. Once the dialog is closed the result
is passed to the delegate given to the Raise method.

Of course you can also define custom dialog content:

```Xaml
<i:Interaction.Triggers>
  <prism:InteractionRequestTrigger SourceObject="{Binding ConfirmationRequest, Mode=OneWay}">
    <prism:PopupWindowAction>
      <prism:PopupWindowAction.WindowContent>
        <DockPanel LastChildFill="True">
          <StackPanel DockPanel.Dock="Bottom" Orientation="Horizontal" Margin="3" 
                      FlowDirection="RightToLeft">
            <Button Content="Cancel" Margin="3,3,3,3" MinWidth="75" 
                    Command="{Binding CancelCommand}"/>
            <Button Content="No" Margin="3,3,3,3" MinWidth="75" 
                    Command="{Binding NoCommand}"/>
            <Button Content="Yes" Margin="3,3,3,3" MinWidth="75" 
                    Command="{Binding YesCommand}"/>
          </StackPanel>
          <StackPanel Orientation="Horizontal">
            <Image Source="/Images/Warning.png" Width="38" Height="38" Margin="10,0,0,0"/>
            <TextBlock DockPanel.Dock="Top" Text="{Binding Question}" HorizontalAlignment="Left" 
                       VerticalAlignment="Center" Margin="25,0,0,0"/>
          </StackPanel>
        </DockPanel>
      </prism:PopupWindowAction.WindowContent>
    </prism:PopupWindowAction>
  </prism:InteractionRequestTrigger>
</i:Interaction.Triggers>
```

You can even define a Prism region for the PopupWindowAction so that you can compose the dialog content - but this is a different story. For 
a detailed evaluation of the different options you probably want to explore [Plainion.Prism RI](https://github.com/plainionist/Plainion.Prism/tree/master/src/Plainion.RI).

## But what about system dialogs?

System dialogs like 
- open file
- save file
- open directory
- print

are a little bit special if you want to have the "native" look & feel. But also this can be done with MVVM and interaction requests:

```Xaml
<i:Interaction.Triggers>
  <prism:InteractionRequestTrigger SourceObject="{Binding OpenFileRequest, Mode=OneWay}">
    <pn:PopupCommonDialogAction FileDialogType="{x:Type win32:OpenFileDialog}"/>
  </prism:InteractionRequestTrigger>
</i:Interaction.Triggers>
```

For more details on handling these special dialogs please refer to

- [File open/save dialog action](https://github.com/plainionist/Plainion.Prism/blob/master/src/Plainion.Prism/Interactivity/PopupCommonDialogAction.cs)
- [Print dialog action](https://github.com/plainionist/Plainion.Prism/blob/master/src/Plainion.Prism/Interactivity/PopupPrintDialogAction.cs=)
- [File open notification](https://github.com/plainionist/Plainion.Prism/blob/master/src/Plainion.Prism/Interactivity/InteractionRequest/OpenFileDialogNotification.cs)
- [File close notification](https://github.com/plainionist/Plainion.Prism/blob/master/src/Plainion.Prism/Interactivity/InteractionRequest/SaveFileDialogNotification.cs)
- [Select folder notification](https://github.com/plainionist/Plainion.Prism/blob/master/src/Plainion.Prism/Interactivity/InteractionRequest/SelectFolderDialogNotification.cs)

## Conclusion

Interaction triggers and Prisms InteractionRequestTrigger provide a great way to integrate dialogs nicely into the MVVM pattern.


