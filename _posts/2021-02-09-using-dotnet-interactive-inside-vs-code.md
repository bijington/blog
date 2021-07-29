---
title: Using dotnet interactive inside VS Code
keywords: "C#,dotnet-interactive,visual studio code,dotnet"
tags:
    - "dotnet"
    - "visual studio code"
---
# Using dotnet interactive inside VS Code

I try to be a strict note taker throughout my day-to-day work life. Whether those are minute from a meeting or recording the results of an investigation. I have been using markdown for this purpose as it is lightweight and I can easily follow the syntax to achieve an outcome that I desire.

More recently I have found that I tend to record/calculate results or have to dig quite deep in some code in order to investigate an issue. For this I have ended up writing no end of Console applications. While it works and can be easy to debug for relatively straightforward code investigations it can be a pain to setup. I have always like the concept of Jupiter notebooks but never bothered to set it up.

I have now discovered that it is possible to create a Jupiter style notebook from within Visual Studio Code using [dotnet interactive](https://github.com/dotnet/interactive) and a handy little extension [.NET Interactive Notebooks](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.dotnet-interactive-vscode). At the time of writing this the extension is still in preview however I have not hit any issues so far.

The steps to setting your tooling up are pretty simple:

1. Install the latest [Visual Studio Code](https://code.visualstudio.com/).
1. Install the latest [.NET 5 SDK](https://dotnet.microsoft.com/download/dotnet/5.0)
1. Install the .NET Interactive Notebooks extension from the [marketplace](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.dotnet-interactive-vscode).

Once set up you can easily get going by creating sections of code or markdown.

![using-dotnet-interactive.png](/images/using-dotnet-interactive.png)

Pretty cool right?
