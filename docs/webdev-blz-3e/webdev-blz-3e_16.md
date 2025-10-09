

# 第十六章：深入了解 WebAssembly

在本章中，我们将更深入地探讨仅与 Blazor **WebAssembly** 相关的技术。

Blazor 中的大多数内容都可以应用于 Blazor Server 和 Blazor WebAssembly。然而，由于 Blazor WebAssembly 在浏览器中运行，我们可以做一些事情来优化代码并使用其他我们无法在服务器端使用的库。

我们还将探讨一些常见问题及其解决方法。

在本章中，我们将涵盖以下内容：

+   探索 WebAssembly 模板

+   .NET WebAssembly 构建工具

+   AOT 编译

+   WebAssembly **单指令多数据**（**SIMD**）

+   压缩

+   懒加载

+   渐进式网络应用

+   原生依赖

+   常见问题

本章的一些部分是很好的跟随学习机会，而其他部分则是供参考的，以便你在需要时能够找到正确的信息。

# 技术要求

本章是一个参考章节，并且与本书的其他章节没有关联。您可以在 [`github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter16`](https://github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter16) 找到本章结果的源代码。

# 探索 WebAssembly 模板

WebAssembly 模板看起来与我们在 *第二章* 中看到的模板略有不同，即创建您的第一个 Blazor 应用。在 Blazor Web App 模板中，我们的入口点是 `app.razor` 文件。它包含我们开始所需的 HTML 标签。WebAssembly 模板有一个 `Index.html` 文件。让我们创建一个项目，以便我们可以查看：

1.  创建一个新的项目并使用 `Blazor WebAssembly Standalone App` 模板。

1.  将项目命名为 `BlazorWebAssembly`。

1.  保持默认设置并按 **创建**。

首先，在 `wwwroot` 文件夹中，我们有一个包含所有 CSS、JavaScript 等内容的 `Index.html` 文件。这与 Blazor Web App 模板中的 `App.razor` 文件内容相同。WebAssembly 项目中也有一个 `app.razor` 文件，但它包含的内容与 `Routes.razor` 文件相同。因此，如果我们同时使用这两个模板，可能会有些令人困惑。

让我们查看每个文件，但只关注特定于 WebAssembly 的内容。在 `Index.html` 中，我们有以下有趣的代码：

```cs
<div id="app">
    <svg class="loading-progress">
        <circle r="40%" cx="50%" cy="50%" />
        <circle r="40%" cx="50%" cy="50%" />
    </svg>
    <div class="loading-progress-text"></div>
</div> 
```

这是一个 `div`，内容是一个显示 WebAssembly 加载进度的进度条。在 `css/app.css` 文件中，我们有以下内容：

```cs
.loading-progress circle:last-child {
    stroke: #1b6ec2;
    stroke-dasharray: calc(3.141 * var(--blazor-load-percentage, 0%) * 0.8), 500%;
    transition: stroke-dasharray 0.05s ease-in-out;
}
.loading-progress-text:after {
    content: var(--blazor-load-percentage-text, "Loading");
} 
```

这些只是加载进度的一些 CSS 类，但有趣的是，Blazor 将给我们两个 CSS 值，`--blazor-load-percentage-text` 和 `--blazor-load-percentage`。这为我们提供了关于加载我们的 WebAssembly 应用剩余时间的指示。这是一种自定义我们的进度指示器的好方法。一旦加载完成，`div` 的内容将被 WebAssembly 应用替换。

如果我们查看 `Program.cs` 文件，当运行独立的 WebAssembly 时，它包含的内容会更多一些：

```cs
var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("#app");
builder.RootComponents.Add<HeadOutlet>("head::after");
builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });
await builder.Build().RunAsync(); 
```

在这里，我们设置我们的 WebAssembly 项目，并告诉 .NET 我们想在 `HTML` 标签中使用 `id` app 渲染 `app.razor` 组件。我们还告诉 .NET 将 `HeadOutlet` 作为 `head` 标签的最后一个子元素渲染。

运行项目并探索一下。项目中的组件与我们已经在 *第四章* 中查看过的相同，即 *理解基本 Blazor 组件*，所以那里没有新的内容。

当我们第一次启动项目时，加载我们的应用程序需要几秒钟。这是所有内容下载并启动的时候。下次我们的用户访问我们的网站时，大部分文件将被缓存，我们不需要再次下载它们。

# .NET WebAssembly 构建工具

当涉及到更“高级”的场景时，我们需要安装额外的工具。有两种安装工具的方法。

在安装 Visual Studio（或使用 Visual Studio 安装程序添加）时，我们可以选择**.NET WebAssembly 构建工具**选项，或者在命令提示符（以管理员身份）中运行以下命令：

```cs
dotnet workload install wasm-tools 
```

.NET WebAssembly 构建工具基于 **Emscripten**，这是一个用于 Web 平台的编译工具链。

# AOT 编译

默认情况下，在 Blazor WebAssembly 应用程序中，作为 WebAssembly 运行的只有运行时。其他所有内容都是普通的 .NET 程序集，通过浏览器使用 WebAssembly 实现的 .NET **中间语言**（**IL**）解释器运行。

当我开始尝试 Blazor 时，我对这一点并不太喜欢；使用 IL 而不是浏览器能够原生理解的代码运行一切感觉是浪费的。然后，我认为浏览器正在运行与我服务器上相同的代码。浏览器中的相同代码！这真是太令人惊讶了！

然而，我们有直接编译到 WebAssembly 的选项；这被称为**即时编译**（**AOT**）。它有一个缺点：应用程序的下载大小会增加，但它的运行和加载速度会更快。

AOT 编译的应用程序通常比 IL 编译的应用程序大两倍。AOT 会将 .NET 代码直接编译成 WebAssembly。

AOT 不会修剪托管程序集，当使用原生 WebAssembly 时，需要更多的代码来表示高级 .NET IL 指令。这就是为什么其大小要大得多，并且它也难以通过 HTTP 压缩。

AOT 并非适用于所有人；大多数不使用 AOT 运行的应用程序都能正常工作。然而，对于计算密集型应用程序，使用 AOT 可以获得很多好处。

我的 ZX Spectrum 模拟器就是这样一款应用程序；它每秒运行多次迭代，对于这些应用程序使用 AOT 的性能提升是显著的。

要使用 AOT 编译我们的 Blazor WebAssembly 项目，我们在 `csproj` 文件中添加以下属性：

```cs
<PropertyGroup>
  <RunAOTCompilation>true</RunAOTCompilation>
</PropertyGroup> 
```

AOT 编译仅在应用程序发布时执行。编译可能需要很长时间（ZX Spectrum 模拟器需要七分钟），所以每次编译应用程序时不必等待这一点是非常好的。

然而，在发布模式下运行可能会出现问题，所以如果您想在发布模式下快速测试，请暂时禁用前面的设置。

不要忘记再次启用它；我在这个领域有一些经验。

# WebAssembly 单指令多数据（SIMD）

.NET7 中的一个新特性是 SIMD，这是一种最近添加到 WebAssembly 的并行处理类型。

SIMD 是一种计算机架构，允许 CPU 同时对多个数据点执行相同的操作，从而提高某些类型任务的性能。SIMD 指令通常用于执行向量运算，其中单个指令同时应用于向量的多个元素。SIMD 对于图像和视频处理等需要快速处理大量数据的任务可能有益。

SIMD 默认启用。要禁用 SIMD，我们需要在项目文件中将其禁用，如下所示：

```cs
<PropertyGroup>
  <WasmEnableSIMD>false</WasmEnableSIMD>
</PropertyGroup> 
```

为了使 SIMD 工作，我们需要使用 AOT 编译。

这超出了本书的范围，但我想要提到它，以防这符合您的项目需求。

# 剪枝

默认情况下，发布 Blazor WebAssembly 应用程序时，会执行剪枝。这将删除不必要的文件，并通过这样做，减小应用程序的大小。

如果我们的应用程序使用反射，剪枝器可能难以确定可以删除和不能删除的内容。

对于大多数应用程序，剪枝是自动的，并且会正常工作。要了解更多关于剪枝选项的信息，您可以查看这里：[`learn.microsoft.com/en-us/dotnet/core/deploying/trimming/trimming-options?pivots=dotnet-8-0`](https://learn.microsoft.com/en-us/dotnet/core/deploying/trimming/trimming-options?pivots=dotnet-8-0)。

# 延迟加载

当使用 Blazor WebAssembly 时，一个挑战是下载大小。尽管这并不是一个大问题，在我看来，我们可以做一些事情来处理下载和加载时间。我们将在本章后面的*常见问题*部分回到这个问题。

当导航到 Blazor WebAssembly 应用程序时，会下载我们应用程序和来自.NET Framework 的所有 DLL。启动一切需要一些时间。我们可以通过使用**延迟加载**来按需加载一些 DLL，以解决这个问题。

假设我们的应用程序很大，其中有一个报告部分。报告可能不是每天都会使用，也不是每个人都会使用，因此从初始下载中删除该部分并在需要时加载它是有意义的。

要实现这一点，我们想要延迟加载的部分必须在一个单独的项目/DLL 中。在 Blazor WebAssembly 客户端项目的`csproj`文件中，通过添加以下代码来添加对 DLL 的引用：

```cs
<ItemGroup>
  <BlazorWebAssemblyLazyLoad Include="{ASSEMBLY NAME}.dll" />
</ItemGroup> 
LazyAssemblyLoader.
```

`LazyAssemblyLoader`服务将进行 JS 互操作调用以下载程序集并将其加载到运行时。

我们确保在路由器（`App.razor`）中下载必要的程序集/DLL，这样我们就可以确保在导航到使用它们的组件之前下载它们：

```cs
@using Microsoft.AspNetCore.Components.Routing
@using Microsoft.AspNetCore.Components.WebAssembly.Services
@using Microsoft.Extensions.Logging
@inject LazyAssemblyLoader AssemblyLoader
@inject ILogger<App> Logger
<Router AppAssembly="@typeof(App).Assembly" 
    OnNavigateAsync="@OnNavigateAsync">
    ...
</Router>
@code {
    private async Task OnNavigateAsync(NavigationContext args)
    {
        try
           {
               if (args.Path == "{PATH}")
               {
                   var assemblies = await AssemblyLoader.LoadAssembliesAsync(
                       new[] { {LIST OF ASSEMBLIES} });
               }
           }
           catch (Exception ex)
           {
               Logger.LogError("Error: {Message}", ex.Message);
           }
    }
} 
```

我们需要注入 `LazyAssemblyLoader`；在 Blazor WebAssembly 项目中，它默认注册为单例。

您需要设置一个 `OnNavigateAsync` 事件，并在该方法中检查路径，确保加载所需的程序集。

此事件也可以通过执行类似以下操作来用于可路由组件：

```cs
@using System.Reflection
@using Microsoft.AspNetCore.Components.Routing
@using Microsoft.AspNetCore.Components.WebAssembly.Services
@using Microsoft.Extensions.Logging
@inject LazyAssemblyLoader AssemblyLoader
@inject ILogger<App> Logger
<Router AppAssembly="@typeof(App).Assembly" 
    AdditionalAssemblies="@lazyLoadedAssemblies" 
    OnNavigateAsync="@OnNavigateAsync">
    ...
</Router>
@code {
    private List<Assembly> lazyLoadedAssemblies = new();
    private async Task OnNavigateAsync(NavigationContext args)
    {
        try
           {
               if (args.Path == "{PATH}")
               {
                   var assemblies = await AssemblyLoader.LoadAssembliesAsync(
                       new[] { {LIST OF ASSEMBLIES} });
                   lazyLoadedAssemblies.AddRange(assemblies);
               }
           }
           catch (Exception ex)
           {
               Logger.LogError("Error: {Message}", ex.Message);
           }
    }
} 
"/fetchdata". The {LIST OF ASSEMBLIES}, which contains a list of assemblies we wish to load, can be "sample.dll".
```

这使得对于没有访问权限的用户不加载管理界面成为可能，例如。当然，当有需要时，我们可以触发下载额外的程序集（而不是等待用户点击应用程序的特定部分然后下载程序集）。

# 渐进式 Web 应用

Blazor 服务器和 Blazor WebAssembly 都可以创建 **渐进式 Web 应用**（**PWA**），但 Blazor WebAssembly 更常见。PWA 使得我们可以下载我们的 Web 应用并将其作为手机或计算机上的应用程序运行。它们将使我们能够添加漂亮的图标，并在没有 URL 输入字段的情况下在网页浏览器中启动我们的网站，这样它就会更像一个应用程序。

在创建项目时，我们选择 **渐进式 Web 应用**。通过这样做，我们将获得一些配置和 JavaScript 来设置一切。

PWA 超出了本书的范围，但有一些很好的资源可以帮助我们入门。您可以在以下链接中找到更多信息：[`learn.microsoft.com/en-us/aspnet/core/blazor/progressive-web-app?view=aspnetcore-8.0&tabs=visual-studio`](https://learn.microsoft.com/en-us/aspnet/core/blazor/progressive-web-app?view=aspnetcore-8.0&tabs=visual-studio)。

# 原生依赖项

由于我们在运行 WebAssembly，我们可以在项目中使用用其他语言编写的 WebAssembly 程序集。这意味着我们可以在项目中直接使用任何原生依赖项。

一种方法是将 C 文件直接添加到我们的项目中。在存储库中的 `Chapter16` 文件夹中，你可以找到一个示例。

我添加了一个名为 `Test.c` 的文件，内容如下：

```cs
int fact(int n)
{
    if (n == 0) return 1;
    return n * fact(n - 1);
} 
```

在项目文件中，我添加了对该文件的引用：

```cs
<ItemGroup>
    <NativeFileReference Include="Test.c" />
</ItemGroup> 
```

在 `Home.razor` 文件中，我添加了以下代码：

```cs
@page "/"
@using System.Runtime.InteropServices
<PageTitle>Native C</PageTitle>
<h1>Native C Test</h1>
<p>
    @@fact(3) result: @fact(3)
</p>
@code {
    [DllImport("Test")]
    static extern int fact(int n);
} 
```

在我们的 C# 项目中，我们现在有一个可以从 Blazor 项目中调用的 C 文件。它被编译成 WebAssembly，然后我们可以引用那个 WebAssembly 文件（这会自动发生）。我们可以通过使用一个使用 C++ 库的库来更进一步。Skia 是一个用 C++ 编写的开源图形引擎。

更多信息请参阅：[`github.com/mono/SkiaSharp`](https://github.com/mono/SkiaSharp)。我们可以通过添加 NuGet 包 `SkiaSharp.Views.Blazor` 将该库添加到 Blazor WebAssembly 应用程序中。

在存储库中的 `Chapter16` 文件夹中，您可以探索一个名为 `SkiaSharpDemo` 的项目。

在 `Home.razor` 文件中，我添加了以下代码：

```cs
<SKCanvasView OnPaintSurface="@OnPaintSurface" />
@code {
    private void OnPaintSurface(SKPaintSurfaceEventArgs e)
    {
        var canvas = e.Surface.Canvas;
        canvas.Clear(SKColors.White);
        using var paint = new SKPaint
        {
            Color = SKColors.Black,
            IsAntialias = true,
            TextSize = 24
        };
        canvas.DrawText("Raccoons are awesome!", 0, 24, paint);
    }
} 
```

页面将在画布上绘制 `"Raccoons are awesome"`。

在这种情况下，我们正在使用一个使用 C++ 库的 C# 库。

我们甚至可以直接通过添加 **对象文件**（`.o`）、**归档文件**（`.a`）、**位码**（`.bc`）和**独立 WebAssembly 模块**（`.wasm`）来引用已经使用 Emscripten 构建的库。如果我们找到一个用另一种语言编写的库，我们可以将其编译成 WebAssembly，然后从我们的 Blazor 应用程序中使用它。这打开了无数的大门！

接下来，我们将探讨一些我遇到的一些常见问题。

# 常见问题

让我们从一开始就深入探讨这个问题。

关于 Blazor WebAssembly 最常见的评论是下载大小和加载时间。一个小项目的大小大约是 1 MB，但我相信问题是加载时间，而不是下载大小/时间，因为所有内容都已缓存，而且在世界上的大多数地方，我们都能访问高速互联网。

有几种解决这个问题的方法。

## 进度指示器

当谈到**用户体验**（**UX**）时，我们可以给用户一种感知速度的感觉。

默认的 Blazor WebAssembly 模板有一个加载进度指示器，它给用户一些可以看的东西，而不是一个空白页面。它被构建得很容易使用 CSS 变量进行自定义。我们可以使用 `--blazor-load-percentage` 和 `--blazor-load-percentage-text` 变量来自定义并创建我们的进度条。

它甚至不需要指示正在发生什么；Dragons Mania Legends 有像“缝纫迷你维京人”这样的评论，这显然不是正在发生的事情。所以根据我们正在构建的应用程序，显示某些内容比显示无内容更重要。

## 服务器端预渲染

在 Blazor 的早期版本中，我们必须自己做一些魔法才能使预渲染工作。但有了新的 Blazor Web App 模板，我们就可以直接获得这个功能。到目前为止，我们一直在使用 Blazor WebAssembly Standalone 模板讨论 Blazor WebAssembly 的功能；在本节中，我们使用的是 Blazor Web App 模板。更好的解决方案是将其作为 `InteractiveAuto` 运行；这样，我们就可以获得服务器快速加载的能力，然后无需等待即可获得 WebAssembly。

这是一种简单而有效的方法，可以为我们的网站添加 SEO。

有一个问题：在服务器上渲染时，它将加载数据，然后在 WebAssembly 加载时再次加载。

有一种方法可以绕过这个问题，我们将在下一节中探讨。

## 预加载和持久化状态

如果我们可以避免，我们不希望我们的组件调用数据库两次。

如果你运行 `BlazorPrerender` 示例并转到 **天气** 页面，你应该能够看到它加载两次，因为数据是随机的，每次我们请求它时都会生成。

这是在使用 `InteractiveServer`、`InteractiveWebAssembly` 和 `InteractiveAuto` 时看到的行为。页面首先在服务器上渲染。然后，SignalR 或 WebAssembly 被连接并再次加载页面。

这个示例的源代码是 `BlazorPrerender` 项目。

当我们通过 WebAssembly 传递有关登录用户的信息时，我们使用了这种技术。

在 Blazor 的早期版本中，我们必须添加一个名为`persist-component-state`的组件，但在.NET 8 中，这个组件默认添加了。这个组件将在服务器上渲染时渲染组件的保存状态，并且当 SignalR 或 WebAssembly 接管时，状态已经存在。

在客户端项目和我们要持久化的组件中（例如示例中的`Weather.razor`），我们注入一个`PersistanceComponentState`，并使组件实现`Idisposable`：

```cs
@inject PersistentComponentState ApplicationState
@implements IDisposable 
```

我们添加了一个`PersistingComponentStateSubscription`组件，该组件将数据保存到应用程序状态中：

```cs
private PersistingComponentStateSubscription _subscription; 
```

在`OnInitializedAsync`中，我们注册监听组件想要持久化数据时运行的代码：

```cs
_subscription = ApplicationState.RegisterOnPersisting(PersistState); 
```

当我们加载数据时，我们首先确保检查应用程序状态。如果数据不可用，我们可以继续并发出一个`HTTP`请求：

```cs
if (ApplicationState.TryTakeFromJson<WeatherForecast[]>("weatherdata", out var stored))
{
    forecasts = stored;
}
else
{
    await Task.Delay(500);
    var startDate = DateOnly.FromDateTime(DateTime.Now);
    var summaries = new[] { "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching" };
    forecasts = Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = startDate.AddDays(index),
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = summaries[Random.Shared.Next(summaries.Length)]
        }).ToArray();
} 
```

它指的是一个将数据持久化到应用程序状态的方法：

```cs
private Task PersistState()
{
    ApplicationState.PersistAsJson("weatherdata", forecasts);
    return Task.CompletedTask;
}
public void Dispose()
{
    _subscription.Dispose();
} 
```

服务器将首先渲染内容，当服务器完成时，它将响应整个页面，包括一个`Base64`编码的 JSON 字符串，数据看起来像这样：

```cs
<!--Blazor:{"type":"server","prerenderId":"d0b382c5fa7d4b65a8002157a8b6a1 a2","key":{"locationHash":"F2AAEE86A5A9C5406A2EF4551C02A263059448AC:0","formattedComponentKey":""},"sequence":0,"descriptor":"CfDJ8EzYgDK6\u002BdZLqM2gwGUPDtNbwNLH7VoJxc6/d6CZ4gHE0LtdIMqSoBfSh8OHGynUVW5DKNVBSG4cZBgETzOixExgSkzmqvPY7I58TMjl4XliAJ ae5d2fmVTS7\u002ByDOooQOqVN41jgj\u002BthTcmHEkBng1MukO5/28AsARyCKVXGxlw3cu9ohFo6b38BprF63EPjo7zQqNYRQT2k xkxn9TiFzTga//RyoyQKIwvEkb044SW\u002Bj9tHP1bBt3B8rpE5EATAvbtKEu7yjwUFGb3xsDHvJ6jGAtQOKOXQhKoWM5pp8 z0RMKkxMfeyuQUubu7i48qPSPvvWCnoym79o64FsTlataWG9JeO8V1X9ihTQppyw/
jkc0RHp9Si49UgCVlEuPWMXTjVSVj7gBizQRc7eT0t2v30NwpBrYHvQS0t\u002BgssPyT\u002BTQWCfEcEc7iMboA/oCSqcAJRTWCcGbWroCIKchU1mdTJj48vAuMKKu5tw6Yqo61V\u002BM4wTR7XJ1ffk0KCQ7lKCqNr2ffNRz1RxjbQX8oVU4s="}--> 
```

由于我们将所有放入应用程序状态的内容都存储为 JSON，因此非常重要，不要包含任何我们没有考虑显示的敏感数据。当然，这对于所有调用都是正确的，因为我们正在使用 JSON 发送数据。

我们还可以在`InteractiveServer`、`InteractiveWebAssembly`和`InteractiveAuto`上使用`PersistentComponentState`。我通常在我的网站上关闭预渲染，但如果你的网站需要 SEO，这会非常棒。

现在，我们了解了一些常见问题和它们的解决方法。

# 摘要

在本章中，我们探讨了 Blazor WebAssembly 中的一些特定内容。大部分情况下，我们可以在 Blazor Server 和 Blazor WebAssembly 中重用组件，并且我们可以通过使用本章学到的知识来加速 WebAssembly。

我们还研究了原生依赖项，这为重用其他库和混合语言打开了可能性。如果我们不需要支持这两种场景，我们可以充分利用 WebAssembly。

在下一章中，我们将探讨*源生成器*。
