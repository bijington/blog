---
title: "Building a mobile game in Xamarin Forms - Lottie"
date: 2022-03-10  00:00:00 +0000
keywords: "C#,xamarin,xamarin.forms,animations,lottie"
tags:
    - "C#"
    - "xamarin"
    - "xamarin.forms"
    - "animations"
    - "lottie"
---
Lottie is one of the best discoveries that I have made in this past year. It has been around much longer, but it took a friend and a side project to help me discover it.

## What is Lottie?

Lottie is an animation library that was built and open sourced by Airbnb. It is a mobile library for Android and iOS that parses Adobe After Effects animations exported to JSON files with Bodymovin and renders them natively on mobile!

## Not the creative type?

Don't worry neither am I! Don’t get me wrong I think I have these great ideas, but they never translate well on to paper or whatever design tool I am using. We don’t need to worry though because there are great designers out there that build animations and share them on [Lottie Files](https://lottiefiles.com). This site curates a load of free and paid for animations that we can download and embed in our applications.

For our mobile game we are going to make use of the following pre-built animation from Lottie Files:

-	Background
https://lottiefiles.com/75758-background-sparkles

You will notice that the background animation does not match our colour scheme in the app. Thankfully Lottie Files also provides an editor that lets you change the colours and other elements. I have gone ahead and already edited this in Lottie Files. You can do the same or feel free to grab it from the repo here.

Now that we have the files lets jump over to Visual Studio and get them running in our app.

## Using the animation

We need to add the files to both our Android and iOS projects.

### Android

The steps you need to follow are:
1.	Right click on `Pairs.Android`
2.	Select **Add -> Existing Files…**
3.	Navigate to the `background.json` file and add it.
4.	Right click the file
5.	Select **Build Action -> AndroidAsset**

### iOS

The steps you need to follow are:
1.	Right click on `Pairs.iOS`
2.	Select **Add -> Existing Files…**
3.	Navigate to the `background.json` file and add it.
4.	Right click on the file
5.	Select **Build Action -> BundleResource**

### NuGet package

Next we need to go ahead and grab the NuGet package ready to show off our new fancy animation.

1.	Right click on the `Pairs` **solution**
2.	Select **Manage NuGet Packages…**
3.	Search for `Com.Airbnb.Xamarin.Forms.Lottie`
4.	Select the package with the same name
5.	Click **Add Package**
6.	Select all projects
7.	Click **Add**

You should notice that a ReadMe.txt file is opened automatically, this gives a helpful way of seeing the possibilities that come with the package. We certainly won’t need to use it all at least for this scenario but I thoroughly recommend checking out the repository here https://github.com/Baseflow/LottieXamarin

### Render our background animation

Let’s jump over to our `MainPage.xaml` file and add some code changes.

First add the lottie namespace at the top:

```xml
clr-namespace:Lottie.Forms;assembly=Lottie.Forms
```

Next let’s add an `AnimationView` to render the `background.json` file. We want to place this inside our `<Grid>` as the first item. We add it as the first item to make sure it renders behind any other items.

```xml
<lottie:AnimationView Animation="background.json"
                      Grid.RowSpan="2"
                      Opacity="0.5"
                      AnimationSource="AssetOrBundle"
                      RepeatMode="Infinite" />
```

Hopefully most of the changes are self explanatory but here are some key details:

- `Animation="background.json"` - this points to the file we included in our projects.
- `AnimationSource="AssetOrBundle"` - this relates to how we included the file in our projects. Remember the **Build Action** we set when adding the files.
- `RepeatMode="Infinite"` - this will cause the animation to continually play.

Now let's run the app and see the result.

![result](/images/2022-03-10-building-a-mobile-game-in-xamarin-forms-part-seven/background-animation.gif)

## Summary

I hope this shows how we can add some really cool animations to our applications so that it provides the feeling that things are alive.

## Links

The source for the end of this stage can be found at:

[https://github.com/bijington/mobile-game-xamarin-forms/tree/part-seven-lottie](https://github.com/bijington/mobile-game-xamarin-forms/tree/part-seven-lottie)




Previous             |  Next
:-------------------------|-------------------------:
[Shapes/Paths + Converters]({% post_url 2022-01-20-building-a-mobile-game-in-xamarin-forms-part-six %}) | Behaviors - COMING SOON