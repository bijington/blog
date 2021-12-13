---
title: "Building a mobile game in Xamarin Forms - BindableLayout"
date: 2021-12-09  00:00:00 +0000
keywords: "C#,xamarin,xamarin.forms,bindablelayout"
tags:
    - "C#"
    - "xamarin"
    - "xamarin.forms"
    - "bindablelayout"
categories: [C#, xamarin, xamarin.forms, bindablelayout]
---
In this post we will finally get to building some UI and applying the parts we have learnt in the previous posts. This will involve potentially undoing some bits but hopefully this will help to show how we can achieve yet more parts through MVVM and the approaches we have been focussing on.

## Code tidy up

First things first lets jump over to the `App.xaml.cs` file and remove the code we added to `OnStart` in [Data layer]({% post_url 2021-10-20-building-a-mobile-game-in-xamarin-forms-part-two %}).

This code should look like:

```csharp
protected async override void OnStart()
{
    IShapeRepository shapeRepository = new ShapeRepository();

    var shapes = await shapeRepository.ListAsync();

    await MainPage.DisplayAlert(
        $"Loaded {shapes.Count} shapes",
        shapes.First().Path,
        "OK");
}
```

We can completetly delete this method but we will want to write the first two lines again so feel free to cut ready to paste it elsewhere.

Once we have cut the code let's jump over to the `MainPageViewModel` class and use it there.

## Async/await

For this we are going to add some extra logic to our `OnPlay` method we added earlier on in the series.

Before we change our code we are going to make use of a very valuable NuGet package, the [Xamarin Community Toolkit](https://github.com/xamarin/XamarinCommunityToolkit). This toolkit provides some really great things and I thoroughly recommend that you check it out, for now we really care about the `AsyncCommand` which will allow us to easily follow best practises around asynchronous code. For much more in-depth reading on these best practises I recommend you check out Brandon Minnicks (The Code Traveller) repository on [AsyncAwaitBestPractices](https://github.com/brminnick/AsyncAwaitBestPractices). This provides some great explanations on what the compiler does when it encounters the `async` and `await` keywords and how to use them correctly.

To add a NuGet package right click on the solution and choose **Manage NuGet Packages...**. Then search for `Xamarin.Community.Toolkit` and choose **Add**. Select the shared project to add it to.

## Actually load something

Our `OnPlay` method will get a few updates in this series but for now let's do the groundwork:

```csharp
private async Task OnPlay()
{
    ShowPlay = false;
    ShowGuess = true;

    var shapeRepository = new ShapeRepository();

    var allShapes = await shapeRepository.ListAsync();

    // 1. Generate view model copies

    // 2. Randomly sort them

    // 3. Add to a collection for binding
}
```

First you should notice that we have changed the signature of the method to include `async Task` this follows on from the best practise blog post, if you didn't read it that I strongly recommend that you do. 

By changing the signature of the `OnPlay` method we will also be forced to change how we create the `PlayCommand`.

this will change from:

```csharp
PlayCommand = new Command(() => OnPlay());
```

to:

```csharp
PlayCommand = new AsyncCommand(() => OnPlay(), allowsMultipleExecutions: false);
```

Again this implements the best practises for us. The additional `allowsMultipleExecutions` parameter allows us to prevent from a user tapping on our button multiple times and it calling our load method for each tap.

## Create our view model to represent a Shape

The next changes involve us loading the shapes ready to do something with them. Sadly this is our first diversion...

Another good practise is to completely separate your models from your view. Therefore we will create our own view model to represent our `Shape` model class. I have chosen the name `TileViewModel` for this new view model because I feel it represents it's purpose much more clearly than `ShapeViewModel` but this is purely personal choice.

Let's add the new class

```csharp
using Pairs.Models;

namespace Pairs.ViewModels
{
    public class TileViewModel : BaseViewModel
    {
        private bool isGuessed;
        private bool isSelected;
        private readonly Shape shape;

        public string Path => shape.Path;

        public bool IsGuessed
        {
            get => isGuessed;
            set => SetProperty(ref isGuessed, value);
        }

        public bool IsSelected
        {
            get => isSelected;
            set => SetProperty(ref isSelected, value);
        }

        public TileViewModel(Shape shape)
        {
            this.shape = shape;
        }
    }
}
```

Hopefully there isn't too much that is unclear based on what we have already covered but in case some parts aren't clear here is some extra context:

* `using Pairs.Models` - this was added due to there already being a type called `Xamarin.Forms.Shape` and I wanted to make sure that you get the correct type in your code.
* `readonly` - this allows us to mark a field such that it cannot be changed. We can initialise it either by assigning a value when declaring it or within the constructor but nowhere else. This makes it much more safe and indicates to others that you do not expect this to change.

## OK now we will really do something with the shapes

Now that we have our new view model class we can jump back to `MainPageViewModel` and finish off our loading logic.

First we can now add a collection property to our `MainPageViewModel` class that will store our tiles:

```csharp
public ObservableCollection<TileViewModel> Tiles { get; } = new ObservableCollection<TileViewModel>();
```

And then we can populate it inside `OnPlay`:

```csharp
private async Task OnPlay()
{
    ShowPlay = false;
    ShowGuess = true;

    var shapeRepository = new ShapeRepository();

    var allShapes = await shapeRepository.ListAsync();

    var random = new Random();

    // 1. Generate view model copies
    const int gridSize = 8;
    var requiredSpeakerCount = gridSize / 2;

    var actualTiles = new List<TileViewModel>(gridSize);

    for (int i = 0; i < requiredSpeakerCount; i++)
    {
        var shapeIndex = random.Next(allShapes.Count);
        var shape = allShapes[shapeIndex];
        allShapes.RemoveAt(shapeIndex);

        actualTiles.Add(new TileViewModel(shape));
        actualTiles.Add(new TileViewModel(shape));
    }

    // 2. Randomly sort them
    int n = actualTiles.Count;

    while (n > 1)
    {
        n--;
        int k = random.Next(n + 1);
        var value = actualTiles[k];
        actualTiles[k] = actualTiles[n];
        actualTiles[n] = value;
    }

    // 3. Add to a collection for binding
    foreach (var tile in actualTiles)
    {
        Tiles.Add(tile);
    }
}
```

We have added quite a bit of code in here so let's break it down:

```csharp
// 1. Generate view model copies
const int gridSize = 8;
var requiredShapeCount = gridSize / 2;

var actualTiles = new List<TileViewModel>(gridSize);

for (int i = 0; i < requiredShapeCount; i++)
{
    var shapeIndex = random.Next(allShapes.Count);
    var shape = allShapes[shapeIndex];
    allShapes.RemoveAt(shapeIndex);

    actualTiles.Add(new TileViewModel(shape));
    actualTiles.Add(new TileViewModel(shape));
}
```

We are defining how many tiles we will be expecting, creating a list to hold our new tiles and then we loop, picking a random shape out of the `shapes` we loaded from our `ShapeRepository` and then create 2 instances of `TileViewModel`. The last part allowing us to ensure that we will always have matching pairs.

```csharp
// 2. Randomly sort them
int n = actualTiles.Count;

while (n > 1)
{
    n--;
    int k = random.Next(n + 1);
    var value = actualTiles[k];
    actualTiles[k] = actualTiles[n];
    actualTiles[n] = value;
}
```

This loops through our collection that we have built and randomly orders them to make the game challenging.

```csharp
// 3. Add to a collection for binding
foreach (var tile in actualTiles)
{
    Tiles.Add(tile);
}
```

This finally adds them to our `ObservableCollection` which our UI will bind to and be notified of the changes.

## Finally some UI work!!

We are now ready to jump in to the `MainPage.xaml` file and add show something to the user. We are going to swap the `StackLayout` that we used previously out for a `Grid` as this gives us better control on how things are layed out in terms of sizing, location and also allows for elements to overlap.

We should have this from our previous work:

```xml
<StackLayout HorizontalOptions="Center"
            VerticalOptions="CenterAndExpand">

    <Label Text="{Binding GuessedCount}" />

    <Button Text="Play"
            Command="{Binding PlayCommand}"
            IsVisible="{Binding ShowPlay}" />

    <Button Text="Guess"
            Command="{Binding GuessCommand}"
            IsVisible="{Binding ShowGuess}" />

</StackLayout>
```

We are going to swap it to the following:

```xml
<Grid RowDefinitions="80,*">
    <Label Text="{Binding GuessedCount}"
           VerticalOptions="End"
           IsVisible="{Binding ShowGuess}" />

    <Button Text="Play"
            Grid.Row="1"
            Command="{Binding PlayCommand}"
            IsVisible="{Binding ShowPlay}"
            VerticalOptions="Center" />

    <FlexLayout BindableLayout.ItemsSource="{Binding Tiles}"
                Direction="Row"
                Wrap="Wrap"
                JustifyContent="Center"
                AlignItems="Center"
                AlignContent="Center"
                IsVisible="{Binding ShowGuess}"
                Grid.RowSpan="2">
        <BindableLayout.ItemTemplate>
            <DataTemplate x:DataType="viewmodels:TileViewModel">
                <Grid WidthRequest="80"
                      HeightRequest="80"
                      Margin="5">
                    <Frame IsClippedToBounds="True"
                           Padding="0"
                           HasShadow="False"
                           CornerRadius="40"
                           BackgroundColor="{StaticResource Color02}">
                        <Frame.GestureRecognizers>
                            <TapGestureRecognizer Command="{Binding Path=GuessCommand, Source={RelativeSource AncestorType={x:Type viewmodels:MainPageViewModel}}}" />
                        </Frame.GestureRecognizers>

                        <Label Text="{Binding Path}"
                               TextColor="{StaticResource Color01}"/>
                    </Frame>
                </Grid>
            </DataTemplate>
        </BindableLayout.ItemTemplate>
    </FlexLayout>
</Grid>
```

This is probably the most complex piece of code we have written together so we will need to take some time to cover the details:

### FlexLayout

> `FlexLayout` is similar to the Xamarin.Forms `StackLayout` in that it can arrange its children horizontally and vertically in a stack. However, the `FlexLayout` is also capable of wrapping its children if there are too many to fit in a single row or column, and also has many options for orientation, alignment, and adapting to various screen sizes.

This is the main reason we have chosen to use the `FlexLayout`, we want our tiles to wrap on to the next line when it stops fitting on the current one.

Each of these properties we are setting let us control how the content wraps:

```xml
Direction="Row"
Wrap="Wrap"
JustifyContent="Center"
AlignItems="Center"
AlignContent="Center"
```

Microsoft provides some really useful examples via [their documentation](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/layouts/flex-layout).

### BindableLayout

This is the title of our post so everything we have written to this point was all the preamble...

BindableLayouts are a really powerful extension to existing layout controls.

As per the Microsoft [documentation](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/layouts/bindable-layouts):
> Bindable layouts enable any layout class that derives from the Layout<T> class to generate its content by binding to a collection of items, with the option to set the appearance of each item with a DataTemplate. Bindable layouts are provided by the BindableLayout class, which exposes the following attached properties:
> * ItemsSource – specifies the collection of IEnumerable items to be displayed by the layout.
> * ItemTemplate – specifies the DataTemplate to apply to each item in the collection of items displayed by the layout.
> * ItemTemplateSelector – specifies the DataTemplateSelector that will be used to choose a DataTemplate for an item at runtime.

This allows us to extend the `FlexLayout` control that will handle the wrapping of it's contents and use `BindableLayout` to dynamically populate it's contents.

Our usage can be broken down in to 2 key parts based on the above quote:

### BindableLayout.ItemsSource

```xml
BindableLayout.ItemsSource="{Binding Tiles}"
```

Here we are binding our `Tiles` property from the `MainPageViewModel` class to our `FlexLayout`. The result of this will be for every `TileViewModel` that we add to `Tiles` a new item will be added as a child of our `FlexLayout`. 

Next we need to tell the control how to render our items.

### BindableLayout.ItemTemplate

```xml
<BindableLayout.ItemTemplate>
    <DataTemplate x:DataType="viewmodels:TileViewModel">
        <Grid WidthRequest="80"
              HeightRequest="80"
              Margin="5">
            <Frame IsClippedToBounds="True"
                   Padding="0"
                   HasShadow="False"
                   CornerRadius="40"
                   BackgroundColor="{StaticResource Color02}">
                <Frame.GestureRecognizers>
                    <TapGestureRecognizer Command="{Binding Path=GuessCommand, Source={RelativeSource AncestorType={x:Type viewmodels:MainPageViewModel}}}" />
                </Frame.GestureRecognizers>

                <Label Text="{Binding Path}"
                       TextColor="{StaticResource Color01}"/>
            </Frame>
        </Grid>
    </DataTemplate>
</BindableLayout.ItemTemplate>
```

Here we are defining what UI elements will be created for each item (`TileViewModel`) that appears in the `ItemsSource` property. We are creating a `Frame` which allows for the `CornerRadius` to be set and therefore we can create a circular tile.

We have added a `TapGestureRecognizer` to the `Frame` to allow the user to tap on it, this will eventually serve as the mechanism for flipping the tiles over to view the shape underneath. For now it simply binds to the original `GuessCommand` that increments the number of guesses. It is worth taking a moment to look over this slightly more complex binding as it deals with [`RelativeSource`](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/data-binding/relative-bindings). What this allows us to do is bind our command up to the `GuessCommand` on the `MainPageViewModel` and this is needed because the `BindingContext` of each of the items inside this `DataTemplate` is actually the `TileViewModel` that it represents. Therefore the `RelativeSource` keyword lets the binding look up the visual tree to find an ancestor of the type (`TileViewModel`) that we have defined.

Currently we are just showing the `Path` of the underlying shape in the `Text` property of a `Label` but in our next post we will look in to converting this path over to an actual `Xamarin.Forms.Shape`.

## Summary

We have actually covered a bit more than just `BindableLayout` but I hope this post helps to show how it is possible to use each of these building blocks in the process of building our applications.

## Links

The source for the end of this stage can be found at:

[https://github.com/bijington/mobile-game-xamarin-forms/tree/part-five-bindable-layout](https://github.com/bijington/mobile-game-xamarin-forms/tree/part-five-bindable-layout)



Previous             |  Next
:-------------------------|-------------------------:
[MVVM Setup]({% post_url 2021-11-10-building-a-mobile-game-in-xamarin-forms-part-four %}) | Shapes/Paths + Converters - COMING SOON