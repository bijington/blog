---
title: "Creating a sprite editor"
date: 2024-07-31  00:00:00 +0000
keywords: "C#,maui,games,MauiUiJuly,MauiGraphics"
tags:
    - "C#"
    - "maui"
    - "games"
    - "MauiUiJuly"
    - MauiGraphics
---

*This post is part of the [MAUI UI July](https://goforgoldman.com/posts/mauiuijuly-24/) community series of blog posts and videos, hosted by Matt Goldman. Be sure to check out the other posts in this series!*

I love participating in this event! It provides me with a platform to have some fun and flex my creative side. On that note for this installment we will looking at building a sprite editor control to support the [Orbit Game Engine](https://github.com/bijington/orbit) that I am currently working on.

![Sneak preview of the final application](/images/2024-07-31-creating-a-sprite-editor/sneak-preview.gif)

The result of this blog post will be to:

- Create a color picker control
- Make use of .NET MAUI GraphicsView control to provide a design surface
- Make use of the Skia implementation for Maui.Graphics to provide the ability to export our creations

## Creating a color picker control

For this we will be adding a new .NET MAUI ContentView (XAML) to our project - I have called this `ColorPicker` inside a folder called `Controls`.

We will then fill out the contents with the following:

### `ColorPicker.xaml`

```xaml
<?xml version="1.0" encoding="utf-8"?>

<Grid xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
      xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
      x:Class="Orbit.Studio.Controls.ColorPicker"
      RowDefinitions="*,*,*,*,*"
      ColumnDefinitions="*,Auto">
    
    <Slider Minimum="0" Maximum="255" Value="255" x:Name="Red" ValueChanged="OnColorSliderValueChanged" />
    <Entry Text="{Binding Value, Source={x:Reference Red}}" Grid.Column="1" />
    
    <Slider Minimum="0" Maximum="255" Value="0" x:Name="Blue" ValueChanged="OnColorSliderValueChanged" Grid.Row="1" />
    <Entry Text="{Binding Value, Source={x:Reference Blue}}" Grid.Column="1" Grid.Row="1" />
    
    <Slider Minimum="0" Maximum="255" Value="0" x:Name="Green" ValueChanged="OnColorSliderValueChanged" Grid.Row="2" />
    <Entry Text="{Binding Value, Source={x:Reference Green}}" Grid.Column="1" Grid.Row="2" />
    
    <Slider Minimum="0" Maximum="255" Value="255" x:Name="Alpha" ValueChanged="OnColorSliderValueChanged" Grid.Row="3" />
    <Entry Text="{Binding Value, Source={x:Reference Alpha}}" Grid.Column="1" Grid.Row="3" />
    
    <BoxView x:Name="ColorPreview" WidthRequest="50" HeightRequest="50" Grid.Row="4" />
    
</Grid>
```

What we see here is:

- A pairing of a `Slider` and `Entry` control repeated 4 times, one for each color component and one for the alpha component.
- Each pair binds the `Text` property of the `Entry` control to the `Value` property of the `Slider`, this will enable the user to either move the slider or manually enter their desired value.
- A `BoxView` that will provide a preview to show the user what the chosen color is.

### `ColorPicker.xaml.cs`

```csharp
namespace Orbit.Studio.Controls;

public partial class ColorPicker : Grid
{
    public ColorPicker()
    {
        InitializeComponent();
    }
    
    private void OnColorSliderValueChanged(object? sender, ValueChangedEventArgs e)
    {
        SelectedColor = Color.FromRgba(
            Red.Value / 255d,
            Blue.Value / 255d,
            Green.Value / 255d,
            Alpha.Value / 255d);
    }
    
    public static readonly BindableProperty SelectedColorProperty = 
        BindableProperty.Create(
            nameof(SelectedColor),
            typeof(Color),
            typeof(ColorPicker),
            Colors.Black, 
            propertyChanged: OnSelectedColorPropertyChanged);

    private static void OnSelectedColorPropertyChanged(BindableObject sender, object oldValue, object newValue)
    {
        ((ColorPicker)sender).UpdatePreviewColor();
    }

    private void UpdatePreviewColor()
    {
        ColorPreview.BackgroundColor = SelectedColor;
    }

    public Color SelectedColor
    {
        get => (Color)GetValue(SelectedColorProperty);
        set => SetValue(SelectedColorProperty, value);
    }
}
```

What we have added here is:

- A `OnColorSliderValueChanged` method that will be handled for the value change of all slider controls.
- A `BindableProperty` that will update the `ColorPreview` `BoxView` when the value changes. We won't be using the bindable nature of this property in our implementation but it makes it easier for someone that might.

## Creating the design surface

There are a number of requirements that I have set upon us and rather than implementing each one individually I have opted to throw them all at you in one go.

We will allow the user to:

- Change the Width and Height of the image
- Zoom into the image
- Show a chessboard and/or grid lines to aid in designing
- Undo their previous action
- Export their creation

Let's proceed to satisfying the above requirements. For this we will be adding a new .NET MAUI ContentPage (XAML) to our project - I have called this `SpriteEditorPage` inside a folder called `Sprites`.

We will then fill out the contents with the following:

### `SpriteEditorPage.xaml`

```xaml
<?xml version="1.0" encoding="utf-8"?>

<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:controls="clr-namespace:Orbit.Studio.Controls"
             x:Class="Orbit.Studio.Sprites.SpriteEditorPage"
             Title="Sprite Editor">
    
    <Grid ColumnDefinitions="2*,*">
        <VerticalStackLayout Grid.Column="1" Spacing="10">
            <HorizontalStackLayout>
                <Label Text="Width" VerticalOptions="Center"/>
                <Entry Keyboard="Numeric" Text="16" TextChanged="WidthEntry_OnTextChanged" />
            </HorizontalStackLayout>
            <HorizontalStackLayout>
                <Label Text="Height" VerticalOptions="Center"/>
                <Entry Keyboard="Numeric" Text="16" TextChanged="HeightEntry_OnTextChanged" />
            </HorizontalStackLayout>
            
            <Label Text="Zoom" />
            <Slider 
                x:Name="Zoom"
                Minimum="1"
                Maximum="50"
                Value="1"
                ValueChanged="Zoom_OnValueChanged"/>
            
            <HorizontalStackLayout>
                <Label Text="Show grid lines" VerticalOptions="Center"/>
                <CheckBox x:Name="ShowGridLines" CheckedChanged="ShowGridLines_OnCheckedChanged" />
            </HorizontalStackLayout>
            
            <HorizontalStackLayout>
                <Label Text="Show chessboard" VerticalOptions="Center"/>
                <CheckBox x:Name="ShowChessboard" CheckedChanged="ShowChessboard_OnCheckedChanged" />
            </HorizontalStackLayout>
            
            <Button Text="Undo" Clicked="OnUndoClicked" />
        
            <controls:ColorPicker x:Name="ColorPicker" />
            
            <Button Clicked="Button_OnClicked" Text="Export" />
        </VerticalStackLayout>
        
        <GraphicsView 
            x:Name="Canvas"
            Drawable="{Binding}"
            MoveHoverInteraction="Canvas_OnMoveHoverInteraction"
            EndInteraction="GraphicsView_OnEndInteraction"/>
    </Grid>
    
</ContentPage>
```

### `SpriteEditorPage.xaml.cs`

```csharp
using Microsoft.Maui.Graphics.Skia;

using SkiaSharp;

namespace Orbit.Studio.Sprites;

public partial class SpriteEditorPage : ContentPage, IDrawable
{
    public SpriteEditorPage()
    {
        InitializeComponent();
        BindingContext = this;
    }

    private int width = 16;
    private int height = 16;
    private IList<Pixel> pixels = [];
    private float mouseX;
    private float mouseY;

    private void GraphicsView_OnEndInteraction(object? sender, TouchEventArgs e)
    {
        pixels.Add(new Pixel { Color = ColorPicker.SelectedColor, Location = new PointF(mouseX, mouseY) });
    }
        
    public void Draw(ICanvas canvas, RectF dirtyRect)
    {
        Render(canvas, dirtyRect, (float)Zoom.Value, ShowGridLines.IsChecked, ShowChessboard.IsChecked);
    }

    private void Render(ICanvas canvas, RectF bounds, float zoomFactor, bool showGridLines, bool showChessboard)
    {
        var renderedWidth = width * zoomFactor;
        var renderedHeight = height * zoomFactor;

        if (showChessboard)
        {
            for (int x = 0; x < width; x++)
            {
                for (int y = 0; y < height; y++)
                {
                    var number = y % 2 == 0 ? 1 : 0;
                    canvas.FillColor = x % 2 == number ? Colors.White : Color.FromRgb(192 / 255d, 192 / 255d, 192 / 255d);
                    canvas.FillRectangle(x * zoomFactor, y * zoomFactor, 1 * zoomFactor, 1 * zoomFactor);
                }
            }
        }

        if (showGridLines)
        {
            var lineColor = Color.FromRgb(192 / 255d, 192 / 255d, 192 / 255d);
            
            for (int x = 1; x < width; x++)
            {
                canvas.StrokeColor = lineColor;
                canvas.DrawLine(x * zoomFactor, 0, x * zoomFactor, height * zoomFactor);
            }
            
            for (int y = 1; y < height; y++)
            {
                canvas.StrokeColor = lineColor;
                canvas.DrawLine(0, y * zoomFactor, width * zoomFactor, y * zoomFactor);
            }
        }

        foreach (var tile in pixels)
        {
            canvas.FillColor = tile.Color;
            canvas.FillRectangle(tile.Location.X * zoomFactor, tile.Location.Y * zoomFactor, zoomFactor, zoomFactor);    
        }
        
        canvas.FillColor = ColorPicker.SelectedColor;
        canvas.FillRectangle(mouseX * zoomFactor, mouseY * zoomFactor, zoomFactor, zoomFactor);
    }

    private void Zoom_OnValueChanged(object? sender, ValueChangedEventArgs e)
    {
        Canvas.Invalidate();
    }

    private void Canvas_OnMoveHoverInteraction(object? sender, TouchEventArgs e)
    {
        var touch = e.Touches.First();

        var zoomFactor = (float)Zoom.Value;
        
        mouseX = MathF.Floor(touch.X / zoomFactor);
        mouseY = MathF.Floor(touch.Y / zoomFactor);
        
        Canvas.Invalidate();
    }
    
    private void Export()
    {
        // Leave this empty for now
    }

    private void Button_OnClicked(object? sender, EventArgs e)
    {
        Export();
    }

    private void ShowGridLines_OnCheckedChanged(object? sender, CheckedChangedEventArgs e)
    {
        Canvas.Invalidate();
    }

    private void ShowChessboard_OnCheckedChanged(object? sender, CheckedChangedEventArgs e)
    {
        Canvas.Invalidate();
    }

    private void OnUndoClicked(object? sender, EventArgs e)
    {
        pixels.RemoveAt(pixels.Count - 1);
        Canvas.Invalidate();
    }

    private void WidthEntry_OnTextChanged(object? sender, TextChangedEventArgs e)
    {
        int.TryParse(e.NewTextValue, out width);
        Canvas.Invalidate();
    }
    
    private void HeightEntry_OnTextChanged(object? sender, TextChangedEventArgs e)
    {
        int.TryParse(e.NewTextValue, out height);
        Canvas.Invalidate();
    }
}

public class Pixel
{
    public Color Color { get; init; }
        
    public PointF Location { get; init; }
}
```

## Provide the ability to export the creation

This involves a little bit more effort so I have opted to separate this out.

We first need to make use of a nuget package which is `Microsoft.Maui.Graphics.Skia`. You can make use of your favorite package manager interface to add the package however I recommend doing it manually for one key reason that you will see shortly.

If we add the PackageReference as below:

```xml
<PackageReference Include="Microsoft.Maui.Graphics.Skia" Version="$(MauiVersion)" />
```

You may notice the use of `Version="$(MauiVersion)"` this means it will keep the version in sync with your .NET MAUI packages and reduce any headaches.

Now we can fill in our `Export` method with the following:

```csharp
private void Export()
{
    using var canvas = new SkiaCanvas();
    using var bitmap = new SKBitmap(new SKImageInfo(width, height));
    canvas.Canvas = new SKCanvas(bitmap);
    
    Render(canvas, new RectF(0, 0, width, height), 1f, false, false);

    var path = Path.Combine(FileSystem.AppDataDirectory, @"sprite.png");
    using var stream = File.Create(path);
    bitmap.Encode(stream, SKEncodedImageFormat.Png, 100);
}
```

This allows us to:

- Create a `SkiaCanvas`
- Pass it into our `Render` method to draw the same creation observed in the application
- Save the result of the `Render` method out to a file

The save process currently sticks it in a fixed location with a fixed filename but I am sure that you could enhance this to be more dynamic.

## Conclusion

The final result of this can be found up on the [Orbit Game Engine](https://github.com/bijington/orbit) repository and specifically in the [Orbit Studio](https://github.com/bijington/orbit/tree/main/games/Orbit.Studio/Orbit.Studio) part.

I hope this helps to show off what fun and creative things you can do with .NET MAUI. I know that the result for me is something that I can now iterate on to actually design content for upcoming games. If you have a desire to get involve then please feel free to contribute to the repository or let me know what you are working on.
