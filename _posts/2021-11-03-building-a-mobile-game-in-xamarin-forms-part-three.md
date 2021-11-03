---
title: Building a mobile game in Xamarin Forms - Styling / resources
date: 2021-11-03  00:00:00 +0000
keywords: "C#,xamarin,xamarin.forms"
tags:
    - "C#"
    - "xamarin"
    - "xamarin.forms"
categories: [C#, xamarin, xamarin.forms]
---
First off we need to choose a color scheme for our app! This can be a real challenge especially for the less creative types. I am certainly in that category, along with being red-green color blind I have been known to make some questionable decisions around matching colors together :).

Thankfully there are many talented people out in the world that are willing to share their schemes/designs/etc. for this app I have chosen to pick a trending color scheme from [Coolors](https://coolors.co/palettes/trending) and specifically [this one](https://coolors.co/264653-2a9d8f-e9c46a-f4a261-e76f51).

From this color scheme we are going apply the following colors accordingly:

### `#264653`
![Background](/images/2021-11-03-building-a-mobile-game-in-xamarin-forms-part-three/background.png) 

*This will be the background of our app.*

### `#2A9D8F`
![Foreground](/images/2021-11-03-building-a-mobile-game-in-xamarin-forms-part-three/foreground.png) 

*This will apply to most parts of the foreground (e.g. text color, shape colors, etc.).*

## Setup colors 
Let's go over to Visual Studio and add these colors ready for use throughout the application.

First we are going to want to open the `App.xaml` file and add a `ResourceDictionary` in to the `App.Resources` element and then our colors inside of that:
```xml
<ResourceDictionary>
    <Color x:Key="Color01">#264653</Color>
    <Color x:Key="Color02">#2A9D8F</Color>
</ResourceDictionary>
```

### What does this mean?

> A ResourceDictionary is a repository for resources that are used by a Xamarin.Forms application. Typical resources that are stored in a ResourceDictionary include styles, control templates, data templates, colors, and converters. [More details](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/xaml/resource-dictionaries)

Next for anything within the `ResourceDictionary` we need to be able to give it a unique key which allows us to refer back to it when we wish to use it later on.

### Why names like `Color01`?

This comes entirely down to preference, I used to think that giving resources like this names relating to the context in how they are used was a good idea (e.g. `ForegroundColor`). In practise naming things like this can sometimes be painful especially if naming sometimes is a struggle or if you find that odd edge case where you want to use `ForegroundColor` for something other than a foreground.

### Result

Your resulting `App.xaml` file should now look as follows:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Application xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="Pairs.App">
    <Application.Resources>
        <ResourceDictionary>
            <Color x:Key="Color01">#264653</Color>
            <Color x:Key="Color02">#2A9D8F</Color>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

## Use the colors

This will have no effect on our application until we tell it how to use our newly defined colors. Let's jump over to our `MainPage.xaml` file and do that.

This file should have a `StackLayout` filled with a `Frame` and a number of `Label`s. For simplicity let's delete all of that so we have a blank ~~canvas~~ `ContentPage`. This should look like:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="Pairs.MainPage">

</ContentPage>
```

First let's set the `BackgroundColor` of our `ContentPage` to our Color01:

```xml
BackgroundColor="{StaticResource Color01}"
```

We are only using a `StaticResource` here because we are not expecting the color to ever change, if you wanted this to be possible then a `DynamicResource` would be a good option.

Now let's go ahead and just add a `StackLayout` with 2 `Label`s inside to tell the user that the game is coming real soon:

```xml
<StackLayout HorizontalOptions="Center"
            VerticalOptions="CenterAndExpand">

    <Label Text="Coming real soon"
           TextColor="{StaticResource Color02}" />

    <Label Text="Honestly it really is!" />

</StackLayout>
```

You may have noticed we only set the `TextColor` of one of the `Label`s here. This is not a mistake this is a build up to `Style`s and how they can be helpful. First lets run the app and see how it looks:

![app-labels-inconsistent](/images/2021-11-03-building-a-mobile-game-in-xamarin-forms-part-three/app-labels-inconsistent.png)

As you can see the `Label`s display but they are not consistent. A quick fix here could be to set the `BackgroundColor` of the second `Label` and this is fine in simple scenarios but imagine you have 100 `Label`s in your application that all need to have similar styling.

## Styles

In order to solve this inconsistency issue with our `Label`s we are going to create what is called an Implicit Style:

> An implicit style is one that's used by all controls of the same TargetType, without requiring each control to reference the style. [More details](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/styles/xaml/implicit)

*This only really touches the surface of what is possible with `Style`s. I would thoroughly recommend looking over this set of [Microsoft documentation on styling](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/styles/xaml/) to appreciate the possibilities.*

### Slight tidy up

First lets quickly delete the `BackgroundColor` setter from the `Label` in our `MainPage.xaml` file leaving us with:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="Pairs.MainPage"
             BackgroundColor="{StaticResource Color01}">

    <StackLayout HorizontalOptions="Center"
                 VerticalOptions="CenterAndExpand">

        <Label Text="Coming real soon" />

        <Label Text="Honestly it really is!" />

    </StackLayout>

</ContentPage>
```

### Create the style

We are going to jump over to our `App.xaml` file to locate our `Style` definition. Placing them in the `App.Resources` like this is considered creating a global style as it is declared at the application level. A `Style` can be declared at any level of the UI tree allowing you to limit their effect.

Let's go ahead and add the following below our `Color` definitions in the `App.Resources` element:

```xml
<Style TargetType="Label">
    <Setter Property="FontSize" Value="Large" />
    <Setter Property="HorizontalTextAlignment" Value="Center" />
    <Setter Property="TextColor" Value="{StaticResource Color02}" />
</Style>
```

We have added some extra bits here to make the text display a little nicer; centrally aligning the text horizontally and just making it a little bigger so it is easier to read.

We don't need to do anything else because of the Implicit Style approach mentioned earlier. So let's run the app and see the result:

![app-labels-consistent](/images/2021-11-03-building-a-mobile-game-in-xamarin-forms-part-three/app-labels-consistent.png)

## Conclusion

This concludes our journey for today. Sadly given the possibilities available to us from the framework we haven't been able to cover all of it but I hope I have helped to show how powerful styling within Xamarin.Forms really can be.

## Links

The source for the end of this stage can be found at:

[https://github.com/bijington/mobile-game-xamarin-forms/tree/part-three-styling-resources](https://github.com/bijington/mobile-game-xamarin-forms/tree/part-three-styling-resources)



Previous             |  Next
:-------------------------|-------------------------:
[Data layer]({% post_url 2021-10-13-building-a-mobile-game-in-xamarin-forms-part-two %}) | MVVM setup - COMING SOON