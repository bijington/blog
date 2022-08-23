---
title: "Playing audio in .NET MAUI applications"
date: 2022-08-23  00:00:00 +0000
keywords: "C#,maui,audio,.NET"
tags:
    - "C#"
    - ".NET"
    - "maui"
    - "audio"
---

This post covers how we can add the ability to play audio files in our .NET MAUI applications. In order to achieve this we will be making use of the [`Plugin.Maui.Audio`](https://www.nuget.org/packages/Plugin.Maui.Audio/) package that my good friend Gerald and I have recently put together.

## Initializing the package

In order to use the package with the built-in dependency injection provided then we need to register the `AudioManager.Current` instance with the `MauiAppBuilder`.

```csharp
builder.Services.AddSingleton(AudioManager.Current);
```

## Including our audio files

There are two main options when wanting to play an audio file:

1. Embedded the file in the application.
1. Browsing on the device and running a local file.

This post currently focuses on item 1 in the list above however it is possible to call `IAudioManager.CreatePlayer` and pass in the `fileName` rather than the `audioStream`.

In order to include our audio file in our .NET MAUI project we simply need to drop it in the *\Resources\Raw* folder. As you can see in the screenshot below

![result](/images/2022-08-23-playing-audio-in-dotnet-maui/raw-resource.png)

## Using the package

Now that we have initialized the package and included our files to be used, we can interact with the `IAudioManager` implementation. This allows us to create players that will ultimately play the audio for us.

For this we will need to gain an instance of the `IAudioManager` and then call `CreatePlayer` to give us an `IAudioPlayer` instance that can control thr playback of our audio file.

Let's take a look at a very brief example.

```csharp
public class MusicPlayerPageViewModel
{
    readonly IAudioManager audioManager;
    IAudioPlayer audioPlayer;

    public MusicPlayerPageViewModel(IAudioManager audioManager)
    {
        this.audioManager = audioManager;
    }

    public async Task Load()
    {
        // Load the audio stream from the application
        Stream audioStream = await FileSystem.OpenAppPackageFileAsync(musicItem.Filename);

        // Create the player
        audioPlayer = audioManager.CreatePlayer(audioStream);
    }
}
```

Now that we have created our `audioPlayer` we can call `Play`, `Pause`, `Stop`, `Seek` on it as well as controlling things like the `Volume` and even the `Balance`.

## Tidying up after ourselves

The `IAudioPlayer` instance that we are provided implements `IDisposable` because it holds onto the resource being used, therefore it is our responsibility to call `Dispose` on it when we have finished using it.

## Summary

I wish I could show this working but that isn't exactly the easiest concept with something that is audible :). I hope this article at least shows how straightforward this package makes it to play audio in our applications.

Please do take the package for a spin and provide feedback on your usage.

## Links

- [Sample](https://github.com/jfversluis/Plugin.Maui.Audio/tree/main/samples)
- [GitHub repository](https://github.com/jfversluis/Plugin.Maui.Audio)
- [NuGet package](https://www.nuget.org/packages/Plugin.Maui.Audio/)