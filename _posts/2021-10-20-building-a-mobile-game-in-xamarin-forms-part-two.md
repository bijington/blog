---
title: Building a mobile game in Xamarin Forms - Data layer
date: 2021-10-20  00:00:00 +0000
keywords: "C#,xamarin,xamarin.forms"
tags:
    - "C#"
    - "xamarin"
    - "xamarin.forms"
categories: [C#, xamarin, xamarin.forms]
---
For this installment of the series we will be covering how to add a data layer to our application and some patterns we can apply in order cleanly separate data access from business logic.

Let's start with an explanation of the pattern (Repository pattern) we shall be using and then dive in to some examples.

## Repository pattern

The repository pattern allows us to hide all the logic that deals with Creating, Reading, Updating and Deleting of entities within our application. By using this pattern it allows us to keep all the knowledge around how entities are loaded, saved, etc. in a single place. *This has the added benefit that if you wanted to completely change where your data is loaded from you only need to change the implementation inside the repository. It also allows us to provide mock implementations when wanting to perform things like unit testing and we don't want to have to rely on a database existing.*

## What we need to achieve

That is a little bit about the pattern we wish to use, next I would like to explain what we wish to achieve with that pattern. For our game we are going to want to **load** up a number of **shapes** and present them on the screen as tiles for the user to flip over and match. You can see that I have **emphasised** the key parts to what our implementation will need to do, so lets jump over to Visual Studio and start that process.

Using the result from our previous post [Creating the Application]({% post_url 2021-10-13-building-a-mobile-game-in-xamarin-forms-part-one %}).

## Our implementation

### Models

First we are going to add a new folder called `Models` to the shared project (`Pairs`). This will be where we locate all models (entities from the explanation above) that we wish to interact with. Our first model is `Shape` so let's **create** that class now inside the `Models` folder.

```csharp
namespace Pairs.Models
{
    public class Shape
    {
        public string Path { get; }

        public Shape(string path)
        {
            Path = path;
        }
    }
}
```

You can see that there really isn't much to a `Shape` for the purpose of this application however I still like to keep the class rather than just returning a set of strings as I believe it makes the code easier to read.

### Repositories - interface

Next we are going to add a new folder called `Repositories` to the shared project (`Pairs`). Then we can go ahead and create an interface to define how our repositories will be behave. We will call this interface `IShapeRepository` and create it under the new `Repositories` folder.

```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using Pairs.Models;

namespace Pairs.Repositories
{
    public interface IShapeRepository
    {
        Task<IList<Shape>> ListAsync();
    }
}
```

I am hopeful that the above interface is straightforward to follow but if not `Task` simply allows us to make the method call asynchronous and therefore make our application be able to handle the method taking some time to execute without blocking our users.

### Repositories - basic/mock implementation

When first starting on a project you likely won't have a database let alone any data to load. For this a good way to get up and running quickly can be to create a mock implementation. This can also help to speed up development especially if the load requires loading the data from a web service.

Let's create a `MockShapeRepository` now.

```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using Pairs.Models;

namespace Pairs.Repositories
{
    public class MockShapeRepository : IShapeRepository
    {
        private readonly IList<Shape> speakers;

        public MockShapeRepository()
        {
            speakers = new List<Shape>
            {
                new Shape("M25,0 L50,0 L50,25 C50,38.8071187 38.8071187,50 25,50 L0,50 L0,25 C0,11.1928813 11.1928813,0 25,0 Z"),
                new Shape("M25,0 L50,0 L50,50 L0,50 L0,25 C0,11.1928813 11.1928813,0 25,0 Z"),
                new Shape("M25,0 L0,0 L0,50 L50,50 L50,25 C50,11.1928813 38.8071187,0 25,0 Z"),
                new Shape("M50,0 L50,50 L25,50 C11.1928813,50 0,38.8071187 0,25 C0,11.3309524 10.9701429,0.224119049 24.5865793,0.00334928573 L25,0 L50,0 Z"),
                new Shape("M0,0 L50,0 L50,50 L0,50 L0,0 Z"),
                new Shape("M0,50 L25,0 L50,50 L0,50 Z"),
                new Shape("M25,0 L46.6506351,12.5 L46.6506351,37.5 L25,50 L3.34936491,37.5 L3.34936491, 12.5 Z"),
                new Shape("M8,25 25,0 42,25 25,50 Z"),
                new Shape("M24,0 47.7764129,17.2745751 38.6946313,45.2254249 9.30536869,45.2254249 0.223587093,17.2745751 Z"),
                new Shape("M0,14 12,0 38,0 50,14 25,50 Z"),
                new Shape("M0,36 0,13.5 13.5,0 36,0 50,13.5 50,36 36,50 13.5,50 Z")
            };
        }

        public Task<IList<Shape>> ListAsync() => Task.FromResult(speakers);
    }
}
```

There are some key points to notice here:
- we are implementing the `IShapeRepository` interface which the compiler will force us to implement `Task<IList<Shape>> ListAsync`.
- we are manually creating our data using some custom XAML paths.
- we are then instantly returning our list despite not needing to be asynchronous.

### Using the repository

Sadly we don't have any UI to be able to make this easy to view in our application but I would like to walk throuh consuming our new `MockShapeRepository`.

Let's jump over to the `App.xaml.cs` file and add our code to the `OnStart` method that is currently sat empty.

Once we have finished the method should look like:

```csharp
protected async override void OnStart()
{
    IShapeRepository shapeRepository = new MockShapeRepository();

    var shapes = await shapeRepository.ListAsync();

    await MainPage.DisplayAlert(
        $"Loaded {shapes.Count} shapes",
        shapes.First().Path,
        "OK");
}
```

To dissect these changes:

```csharp
IShapeRepository shapeRepository = new MockShapeRepository();
```
This allows us to provide the implementation of our mock repository that we just created. Of course we could utilise dependency injection here but that is not within the scope of this post.

```csharp
var shapes = await shapeRepository.ListAsync();
```
We are then loading the shapes from the implementation and waiting on the result.

```csharp
await MainPage.DisplayAlert(
    $"Loaded {shapes.Count} shapes",
    shapes.First().Path,
    "OK");
```
Finally we are showing some information about the shapes that have been loaded.

![result of first run](/images/2021-10-20-building-a-mobile-game-in-xamarin-forms-part-two/mock-repository-load.png)

### Repositories - web service

OK I hear you! That wasn't a real world example, let's explore how we could achieve this via something more concrete. Let's go ahead and create a new repository that can pull some data off the web. I have created a [Gist file](https://gist.github.com/bijington/ed15ab140a57e751dc8db6a21b068626) containing the same shapes from our mock only with one missing to prove this works.

In order to do this we will need to add a reference to the [Newtonsoft.Json](https://www.newtonsoft.com/json) NuGet package:
- Right click on the `Pairs` project (yes we only need it on the shared project)
- Select `Manage NuGet Packages...`
- It should be at the top of the list but we want to select `Newtonsoft.Json` (current version is 13.0.1 at time of writing)
- Select `Add Package`

Now we can go ahead and create the repository.

```csharp
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;
using Newtonsoft.Json;
using Pairs.Models;

namespace Pairs.Repositories
{
    public class ShapeRepository : IShapeRepository
    {
        private readonly HttpClient httpClient;

        public ShapeRepository()
        {
            httpClient = new HttpClient();
        }

        public async Task<IList<Shape>> ListAsync()
        {
            var response = await httpClient.GetAsync("https://gist.githubusercontent.com/bijington/ed15ab140a57e751dc8db6a21b068626/raw/576cfa32a44a2a6681f3f9a74b1d0e8527ec7699/shapes.json");

            if (!response.IsSuccessStatusCode)
            {
                return new List<Shape>();
            }

            var content = await response.Content.ReadAsStringAsync();

            return JsonConvert.DeserializeObject<IList<Shape>>(content);
        }
    }
}
```

This implementation will perform a HTTP GET to the gist file, if the request was successful then we will deserialize it using Newtonsoft.Json in to the list of `Shape`s that we are expecting and return it. Don't worry for the purpose of this application I serialized the original data first to make sure it will deserialize safely.

### Using the repository - part two

Now if we jump back over to `App.xaml.cs` we can simply replace this line:
```csharp
IShapeRepository shapeRepository = new MockShapeRepository();
```

with this line:
```csharp
IShapeRepository shapeRepository = new ShapeRepository();
```

![result of first run](/images/2021-10-20-building-a-mobile-game-in-xamarin-forms-part-two/repository-load.png)

## Conclusion

I hope that by building and then swapping between implementations in this post you can see the value in keeping all of the loading logic contained within a repository.


This will now serve as the basis for our next step in this series Styling / resources

And finally the source for the end of this stage can be found at:

[https://github.com/bijington/mobile-game-xamarin-forms/tree/part-one-creating-the-application](https://github.com/bijington/mobile-game-xamarin-forms/tree/part-one-creating-the-application)