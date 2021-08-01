---
title: Animations in Xamarin.Forms
date: 2021-07-29  00:00:00 +0000
keywords: "C#,dotnet,xamarin,xamarin.forms"
tags:
    - "C#"
    - "dotnet"
    - "xamarin"
    - "xamarin.forms"
---
During our time building a mobile game with Xamarin.Forms we discovered just how powerful the [`Animation`](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/animation/custom) class in Xamarin.Forms really is!

We wanted to see how far we could push the framework so we made a plan see if we could bring in some fancy CSS animations from [Animate.css](https://animate.style). Here is just a small set of examples using the `Animation` framework in Xamarin.Forms:

## Tada

This worked really well to show a user that they have guess something correctly.

![tada animation](/images/2021-07-29-animations-in-xamarin-forms/tada.gif)

```csharp
public Animation CreateTadaAnimation(View view)
{
    const double maximumScale = 1.1;
    const double minimumScale = 1.1;
    const double rotationAngle = 3.0;

    var animation = new Animation();

    animation.Add(0, 0.1, new Animation(v => view.Scale = v, 1, minimumScale));
    animation.Add(0.2, 0.3, new Animation(v => view.Scale = v, minimumScale, maximumScale));
    animation.Add(0.9, 1.0, new Animation(v => view.Scale = v, maximumScale, 1));

    animation.Add(0, 0.2, new Animation(v => view.Rotation = v, 0, -rotationAngle));
    animation.Add(0.2, 0.3, new Animation(v => view.Rotation = v, -rotationAngle, rotationAngle));
    animation.Add(0.3, 0.4, new Animation(v => view.Rotation = v, rotationAngle, -rotationAngle));
    animation.Add(0.4, 0.5, new Animation(v => view.Rotation = v, -rotationAngle, rotationAngle));
    animation.Add(0.5, 0.6, new Animation(v => view.Rotation = v, rotationAngle, -rotationAngle));
    animation.Add(0.6, 0.7, new Animation(v => view.Rotation = v, -rotationAngle, rotationAngle));
    animation.Add(0.7, 0.8, new Animation(v => view.Rotation = v, rotationAngle, -rotationAngle));
    animation.Add(0.8, 0.9, new Animation(v => view.Rotation = v, -rotationAngle, rotationAngle));
    animation.Add(0.9, 1.0, new Animation(v => view.Rotation = v, rotationAngle, 0));

    return animation;
}
```

## RubberBand

We haven't actually used this inside our app but I just really like it!

![rubber band animation](/images/2021-07-29-animations-in-xamarin-forms/rubberband.gif)

```csharp
public Animation CreateRubberBandAnimation(View view)
{
    var animation = new Animation();

    animation.Add(0, 0.3, new Animation(v => view.ScaleX = v, 1, 1.25));
    animation.Add(0, 0.3, new Animation(v => view.ScaleY = v, 1, 0.75));

    animation.Add(0.3, 0.4, new Animation(v => view.ScaleX = v, 1.25, 0.75));
    animation.Add(0.3, 0.4, new Animation(v => view.ScaleY = v, 0.75, 1.25));

    animation.Add(0.4, 0.5, new Animation(v => view.ScaleX = v, 0.75, 1.15));
    animation.Add(0.4, 0.5, new Animation(v => view.ScaleY = v, 1.25, 0.85));

    animation.Add(0.5, 0.65, new Animation(v => view.ScaleX = v, 1.15, 0.95));
    animation.Add(0.5, 0.65, new Animation(v => view.ScaleY = v, 0.85, 1.05));

    animation.Add(0.65, 0.75, new Animation(v => view.ScaleX = v, 0.95, 1.05));
    animation.Add(0.65, 0.75, new Animation(v => view.ScaleY = v, 1.05, 0.95));

    animation.Add(0.75, 1, new Animation(v => view.ScaleX = v, 1.05, 1));
    animation.Add(0.75, 1, new Animation(v => view.ScaleY = v, 0.95, 1));

    return animation;
}
```

## You say you want more?

You should definitely watch this space as we are currently in the process of adding in a whole chunk of fancy animations to the already wonderful [Xamarin Community Toolkit](https://github.com/xamarin/XamarinCommunityToolkit) and more specifically under this [feature](https://github.com/xamarin/XamarinCommunityToolkit/issues/1447).