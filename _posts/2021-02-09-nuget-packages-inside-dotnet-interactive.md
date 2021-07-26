---
title: Using dotnet interactive inside VS Code
keywords: "C#,dotnet-interactive,visual studio code,dotnet,nuget"
---
# NuGet packages inside dotnet interactive

Following on from a previous post on using dotnet interactive inside vs code we can dig a little deeper in to some more useful functionality. We all need to consume NuGet packages right? I know I certainly do. 

It is entirely possible to pull a package down and starting coding against that package. I use this feature a lot for diagnosing or testing things out with my own OSS library hosted via NuGet.

All you need to do is declare the following inside your C# code block:

```csharp
#r "nuget: PACKAGE_ID, PACKAGE_VERSION"
```

So shameless plug time :), here is how I do it with my own library:

```csharp
 #r "nuget: ExpressiveParser, 2.4.0" 
```

Then you are free to start using that package:

```csharp
var expression = new Expressive.Expression("Pow(2, WeekDay(#today#))");  
expression.RegisterFunction("WeekDay", (p, a) => ((DateTime)p[0].Evaluate(a)).DayOfWeek);  
Console.WriteLine(expression.Evaluate());  
```

Here you can see it in action, simply hit Ctrl+Return and watch the tool pull the package the first time and then execute the code.

![nuget-package-inside-dotnet-interactive.png](/images/nuget-package-inside-dotnet-interactive.png)
