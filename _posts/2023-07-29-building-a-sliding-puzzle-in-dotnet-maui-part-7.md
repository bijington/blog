---
title: "Building a sliding puzzle game in .NET MAUI - Part seven"
date: 2023-07-29  00:00:00 +0000
keywords: "C#,maui,games,MauiUiJuly"
tags:
    - "C#"
    - "maui"
    - "games"
    - "MauiUiJuly"
---

*This post is part of the [Building a sliding puzzle game in .NET MAUI]({% post_url 2023-07-20-building-a-sliding-puzzle-in-dotnet-maui-part-0 %}) series that is aimed at guiding you through building a simple sliding puzzle game in .NET MAUI.*

*This post is also part of the [MAUI UI July](https://goforgoldman.com/posts/maui-ui-july-23/) community series of blog posts and videos, hosted by Matt Goldman. Be sure to check out the other posts in this series!*

## What we will be covering in this post

1. Animating our level selection page
1. Animating our tile grid
1. Taking our application for a test drive

## Animating our level selection page

This will be a relatively quick change to our code. The plan is to move our items off screen and the animate them moving into position. We can do this through the `TranslationY` property and the `TranslateTo` method. For more information of other types of built-in animations please refer to the [Microsoft documentation](https://learn.microsoft.com/dotnet/maui/user-interface/animation/basic).

Let's open up our *LevelSelectionPage.xaml.cs* file and add the following method:

```csharp
private async void OnLevelsCollectionViewChildAdded(object sender, ElementEventArgs e)
{
    if (e.Element is VisualElement visualElement)
    {
        // Move the item off screen.
        visualElement.TranslationY = DeviceDisplay.Current.MainDisplayInfo.Height;

        // Trigger a delay to wait for the page to load plus a little extra for each new item to make it feel more fluid.
        await Task.Delay(250 + (childCount++ * 100));

        // Manage our own bounce effect
        await visualElement.TranslateTo(0, -20, length: 650, Easing.SinInOut);
        await visualElement.TranslateTo(0, 10, length: 250, Easing.SinInOut);
        await visualElement.TranslateTo(0, 0, length: 150, Easing.SinInOut);
    }
}
```

Next we need to open our *LevelSelection.xaml* file and modify the following lines:

```xaml
<CollectionView
    x:Name="LevelsCollectionView"
    SelectionMode="Single"
    SelectionChanged="OnLevelsCollectionViewSelectionChanged"
    Margin="50">
```

To match the following:

```xaml
<CollectionView
    x:Name="LevelsCollectionView"
    SelectionMode="Single"
    SelectionChanged="OnLevelsCollectionViewSelectionChanged"
    ChildAdded="OnLevelsCollectionViewChildAdded"
    Margin="50">
```

This now performs our animation when each visual child is added to our `CollectionView`.

If we run up our application and open the level selection page we will see the following result:

![Our application animating the levels loading](/images/2023-07-29-building-a-sliding-puzzle-in-dotnet-maui-part-7/level-selection-animation.gif)

## Animating our tile grid

There are two key animations that we will be adding to our grid:

1. Sliding out the tiles once the level has loaded
1. Sliding a tile when tapping on it

First though we need to lay some groundwork:

### Add the `AreAnimationsEnabled` property

There will be times when we want animations to show and times when we don't, in order to handle both we are going to add in a property that can be set.

Let's open up our *SlidingTileGrid.cs* file and add the following property into the class

```csharp
public bool AreAnimationsEnabled { get; set; }
```

Next we only want the animations to be performed on our `LevelPage` so let's open up our *LevelPage.xaml* file and modify the following lines:

```xaml
<local:SlidingTileGrid
    x:Name="SlidingTiles"
    HorizontalOptions="Center"
    VerticalOptions="Center"
    Grid.Row="2"
    Completed="OnSlidingTilesCompleted" />
```

To include the `AreAnimationsEnabled` property:

```xaml
<local:SlidingTileGrid
    x:Name="SlidingTiles"
    AreAnimationsEnabled="True"
    HorizontalOptions="Center"
    VerticalOptions="Center"
    Grid.Row="2"
    Completed="OnSlidingTilesCompleted" />
```

Now that we have the property in place and already being controlled in the correct page, let's proceed to performing some animations.

### How to animate

Initially it can seem like a complicated option to animate things with .NET MAUI Graphics given that we are responsible for drawing directly onto the canvas however we can still make full use of the [`Animation`](https://learn.microsoft.com/dotnet/api/microsoft.maui.controls.animation?view=net-maui-7.0) class.

The `Animation` class allows us to define a numerical value that will change over a specified period of time, this is easiest seen in action so let's proceed to setting up our first animation.

For further information on using this class please refer to the [Microsoft documentation](https://learn.microsoft.com/dotnet/maui/user-interface/animation/custom).

### Sliding out the tiles once the level has loaded

We need to open our *SlidingTileGrid.cs* file and replace the following code in our `Load` method:

```csharp
foreach (var tile in this.tiles)
{
    tile.Offset = TileSpacing;
}
if (this.IsEnabled)
{
    this.ShuffleTiles(200);
}
```

With the following code:

```csharp
if (AreAnimationsEnabled)
{
    new Animation(
        v =>
        {
            foreach (var tile in this.tiles)
            {
                tile.Offset = (float)v;
            }

            this.Invalidate();
        },
        0,
        TileSpacing,
        Easing.SpringOut)
        .Commit(
            this,
            "Load",
            length: 750,
            finished: (v, d) => ShuffleTiles(200));
}
else
{
    foreach (var tile in this.tiles)
    {
        tile.Offset = TileSpacing;
    }

    this.Invalidate();

    if (this.IsEnabled)
    {
        this.ShuffleTiles(200);
    }
}
```

The above code is now creating a new `Animation` that will change the value of each `Tile`s `Offset` property over a period of `750` milliseconds. This animation is instantly committed which starts the animation, we have to give it an owner and a name - these are used if we wish to cancel the animation, the length and a callback to be performed when the animation finishes, in our scenario it is to then shuffle the board.

### Sliding a tile when tapping on it

Our second use of the `Animation` class is more involved, let's replace our existing `Move` method with the code below and then work to understand what is happening.

```csharp
private void MoveTile(Tile matchingTile, bool checkForCompletion, bool animateMovement)
{
    var distance = this.emptyTile.CurrentPosition.Distance(matchingTile.CurrentPosition);

    if (distance == 1)
    {
        var oldPosition = matchingTile.CurrentPosition;
        var newPosition = this.emptyTile.CurrentPosition;
        this.emptyTile.CurrentPosition = oldPosition;

        if (animateMovement)
        {
            var movementAnimation = new Animation();

            // Animate the X
            movementAnimation.Add(
                0,
                1,
                new Animation(
                    x =>
                    {
                        matchingTile.CurrentPosition = new PointF((float)x, matchingTile.CurrentPosition.Y);
                    },
                    oldPosition.X,
                    newPosition.X));

            // Animate the Y
            movementAnimation.Add(
                0,
                1,
                new Animation(
                    y =>
                    {
                        matchingTile.CurrentPosition = new PointF(matchingTile.CurrentPosition.X, (float)y);
                        this.Invalidate();
                    },
                    oldPosition.Y,
                    newPosition.Y));

            movementAnimation.Commit(
                this,
                "Movement",
                length: 250,
                finished: (v, d) =>
                {
                    if (checkForCompletion &&
                        this.tiles.All(t => t.CurrentPosition == t.DestinationPosition))
                    {
                        this.Completed?.Invoke(this, EventArgs.Empty);
                    }
                });
        }
        else
        {
            matchingTile.CurrentPosition = newPosition;
            this.Invalidate();

            if (checkForCompletion &&
                this.tiles.All(t => t.CurrentPosition == t.DestinationPosition))
            {
                this.Completed?.Invoke(this, EventArgs.Empty);
            }
        }
    }
}
```

If we are animating the movement (`animateMovement`) then we start to create our `Animation`, as I mentioned earlier the `Animation` class provides a mechanism to animate a single numerical value. In our scenario though we need to animate both our X and Y positions of a tile when it is sliding from its `oldPosition` to its `newPosition`. We can cheat a little here, the `Animation` class can contain child `Animation`s, so we are building an `Animation` to change the X property and a second one to change the Y property. We then commit the parent `Animation` which will result in both child animations running concurrently and provide us with a smooth sliding animation.

Note that we will need to update the 2 places where we call `MoveTiles`:

#### Inside the `SlidingTileGrid_EndInteraction` method

We need to change the method call to:

```csharp
MoveTile(matchingTile, true, true);
```

This will account for our extra `animateMovement` parameter.

#### Inside the `ShuffleTiles` method

We need to change the method call to:

```csharp
MoveTile(possibleMovements[randomIndex], false, false);
```

This will account for our extra `animateMovement` parameter.

## Taking our application for a test drive

If we run the application we will see the home page and then tapping on the play button will show following pages based on whether we are running in light or dark mode.

Light mode             |  Dark mode
:-------------------------:|:-------------------------:
![Our application running in light mode](/images/2023-07-29-building-a-sliding-puzzle-in-dotnet-maui-part-7/result-1.gif)  |  ![Our application running in dark mode](/images/2023-07-29-building-a-sliding-puzzle-in-dotnet-maui-part-7/result-2.gif)

## What we have achieved

To summarise what we have achieved in this post:

1. Animated our level selection page
1. Animated our tile grid
1. Taken our application for a test drive

This is the end of our journey for now, there are a number of ways that we could enhance this game and if you do please get in touch and let me know how you have.

## Links

The source for the end of this stage can be found at:

[https://github.com/bijington/sliding-puzzle/tree/part-seven](https://github.com/bijington/sliding-puzzle/tree/part-seven)

Previous             |  Next
:-------------------------|-------------------------:
[Creating our level selection page]({% post_url 2023-07-29-building-a-sliding-puzzle-in-dotnet-maui-part-6 %}) | Series complete
