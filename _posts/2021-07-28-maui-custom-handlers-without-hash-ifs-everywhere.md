---
title: Maui custom handlers without hash ifs everywhere
date: 2021-07-28  00:00:00 +0000
keywords: "C#,maui,dotnet"
tags:
    - "C#"
    - "dotnet"
    - "maui"
---
I don't know about you but I really do not like having `#if` definitions dotted around all over my codebase and I try my best to avoid it where I can. Currently a mechanism to configure new custom handlers within MAUI can require you to make use of the `#if` statement as seen below:

```csharp
appBuilder
    .UseMauiApp<App>()
    .ConfigureMauiHandlers(handlers =>
    {
#if __ANDROID__
        handlers.AddHandler(typeof(CustomEntry), typeof(CustomEntryHandler));
#endif
    });
```

I would like to propose an alternative. There are a few steps:

## 1. Create a common `static partial class` to remove the need for `#if`

```csharp
namespace MyFirstMauiApplication
{
    public static partial class CustomHandlerRegistrar
    {
        public static void Register(IMauiHandlersCollection handlers) =>
            RegisterHandlers(handlers);

        public static partial RegisterHandlers(IMauiHandlersCollection handlers);
    }
}
```

## 2. Implement the required parts in your platform specific code

You will need to create a class in each platform **AND make sure that the namespace matches that of the original** `CustomHandlerRegistrar`.

```csharp
namespace MyFirstMauiApplication
{
    public static partial class CustomHandlerRegistrar
    {
        public static partial RegisterHandlers(IMauiHandlersCollection handlers)
        {
            handlers.AddHandler(typeof(CustomEntry), typeof(CustomEntryHandler));
        }
    }
}
```

## 3. Remove the `#if` from `Startup.cs`

This then results in a much cleaner looking `Startup.cs` file.

```csharp
appBuilder
    .UseMauiApp<App>()
    .ConfigureMauiHandlers(handlers => CustomHandlerRegistrar.Register(handlers));
```

`#maui`, `#dotnet`