---
title: Detecting background audio in an iOS app
keywords: "ios, development, mobile, xamarin, xamarin.forms"
---
# Detecting background audio in an iOS app

Recently we launched our brand new, all `Xamarin.Forms` game: **[Super Wordsearch](https://www.superwordsearch.com/)**. To make the game more enjoyable we've added some background music for you to listen to while you are playing.

Shortly after our initial release a couple of users started reporting an issue about the background music. Our game background music would take precedence over any music they were already listening too. Obviously we don't want to get in the way of our users enjoying their fine taste in music!

We quickly scrambled to fix this issue. Our app is iOS only right now, so the solution we need is limited to iOS.

Thankfully the solution is pretty straight-forward as we will see in a little bit.

Apple provide the ability to detect if audio is already playing through their [`AVAudioSession`](https://developer.apple.com/documentation/avfaudio/avaudiosession) class and more specifically the [`isOtherAudioPlaying`](https://developer.apple.com/documentation/avfaudio/avaudiosession/1616610-isotheraudioplaying) property.

Now lets take a look at this in action:

To allow this to used within our shared code we created an interface:

```csharp
public interface IAudioHelper  
{  
  bool IsOtherAudioPlaying { get; }  
}
```

Implemented the platform specific code to access AVAudioSession.

```csharp
[assembly: Xamarin.Forms.Dependency(typeof(MyNamespace.iOS.AudioHelper))]  
namespace MyNamespace.iOS  
{  
  public class AudioHelper : IAudioHelper  
  {  
    public bool IsOtherAudioPlaying => 
      AVAudioSession.SharedInstance.IsOtherAudioPlaying;  
  }  
}
```

Use the dependency service to get the implementation.

```csharp
var isOtherAudioPlaying =  
  DependencyService.Get<IAudioHelper>()?.IsOtherAudioPlaying ?? false;
```
