---
title: "Building a sliding puzzle game in .NET MAUI - Introduction"
date: 2023-07-20  00:00:00 +0000
keywords: "C#,maui,games,MauiUiJuly,MauiGraphics"
tags:
    - "C#"
    - "maui"
    - "games"
    - "MauiUiJuly"
    - MauiGraphics
---

*This post is part of the [MAUI UI July](https://goforgoldman.com/posts/maui-ui-july-23/) community series of blog posts and videos, hosted by Matt Goldman. Be sure to check out the other posts in this series!*

The aim of this blog series is to have a bit of fun playing around with the .NET MAUI Graphics APIs while also showing how we can make a good looking app. We will be building a game that replicates the old physical sliding puzzles that I used to play with loads as a child.

This series will take us:

From this default app             |  To this |  |
:-------------------------:|:-------------------------:|:--:|
![starting point](/images/2023-07-20-building-a-sliding-puzzle-in-dotnet-maui-part-0/app-starting-point.png)  |  ![result](/images/2023-07-20-building-a-sliding-puzzle-in-dotnet-maui-part-0/result-1.png) | ![result](/images/2023-07-20-building-a-sliding-puzzle-in-dotnet-maui-part-0/result-2.png) | ![result](/images/2023-07-20-building-a-sliding-puzzle-in-dotnet-maui-part-0/result-3.png)

The posts in this series are broken down into manageable chunks where we can clearly define our objectives and tick them off as we go.

1. [Creating the application]({% post_url 2023-07-20-building-a-sliding-puzzle-in-dotnet-maui-part-1 %})
2. [Creating our data layer]({% post_url 2023-07-20-building-a-sliding-puzzle-in-dotnet-maui-part-2 %})
3. [Creating our home page]({% post_url 2023-07-20-building-a-sliding-puzzle-in-dotnet-maui-part-3 %})
4. [Creating our tile grid]({% post_url 2023-07-20-building-a-sliding-puzzle-in-dotnet-maui-part-4 %})
5. [Creating our level selection page]({% post_url 2023-07-20-building-a-sliding-puzzle-in-dotnet-maui-part-5 %})
6. Creating our game page - Coming soon
7. Adding animations - Coming soon
8. Finishing it off - Coming soon

## Important points

- The key mechanics of the application will be built on top of **.NET MAUI Graphics**, which we will cover as we progress through the series.
- The application will **not** be built using the MVVM architecture as I do not believe it really adds much value here, plus it is nice to see how you can build things a slightly different way.
- It makes limited use of Bindings.
- The series will be using **Visual Studio for Mac 2022** but most of the approaches used should translate well to your preferred tooling.
- The application has been created against .NET MAUI SDK version **7.0.86**.

## Links

The source code for the whole series can be fount at:

[https://github.com/bijington/sliding-puzzle](https://github.com/bijington/sliding-puzzle)

Note there there is a separate branch containing the result of each part within this blog series.

Previous             |  Next
:-------------------------|-------------------------:
[Introduction]({% post_url 2023-07-20-building-a-sliding-puzzle-in-dotnet-maui-part-0 %}) | [Creating the application]({% post_url 2023-07-20-building-a-sliding-puzzle-in-dotnet-maui-part-1 %})
