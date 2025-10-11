# 7. 使用 ASP.NET 创建现代 Web 应用程序

概览

现在有许多类型的应用程序在使用中，Web 应用程序是其中使用最广泛的一种。在本章中，你将介绍 ASP.NET，这是一个用 C# 和 .NET 运行时构建的 Web 框架，旨在轻松创建 Web 应用程序。你还将了解基本 ASP.NET 应用程序的解剖结构、Web 应用程序开发方法，如服务器端渲染和单页应用程序，以及 C# 如何帮助实现这些方法来构建安全、高效和可扩展的应用程序。

# 简介

在 *第一章*，*你好 C#* 中，你了解到 .NET 是使 C# 生机勃勃的东西，因为它包含用于构建你的代码的软件开发工具包 (SDK) 和执行代码的运行时。在本章中，你将了解 ASP.NET，它是一个嵌入在 .NET 运行时中的开源和跨平台框架。它用于构建 Web、移动和物联网设备的客户端和后端应用程序。

它是这类开发的完整工具箱，因为它提供了几个内置功能，例如轻量级和可定制的 HTTP 管道、依赖注入以及对现代托管技术（如容器、Web UI 页面、路由和 API）的支持。一个著名的例子是 Stack Overflow；其架构完全建立在 ASP.NET 之上。

本章的重点是使你熟悉 ASP.NET 的基础知识，并为你提供对使用 Razor Pages 进行 Web 应用程序开发的介绍和端到端概述，Razor Pages 是 ASP.NET 中内置的工具箱，用于构建 Web 应用程序。

# ASP.NET Web 应用的解剖结构

你将从这个章节开始，使用 ASP.NET 创建一个新的 Razor Pages 应用程序。它只是可以使用 ASP.NET 创建的多种应用程序类型之一，但将是一个有效的起点，因为它与其他可以使用该框架构建的 Web 应用程序类型共享并展示了许多共同点。

1.  要创建一个新的 Razor Pages 应用程序，请在 CLI 中输入以下命令：

    ```cs
    dotnet new razor -n ToDoListApp dotnet new sln -n ToDoList dotnet sln add ./ToDoListApp
    ```

在这里，你正在使用 Razor Pages 创建一个待办事项列表应用程序。一旦执行了前面的命令，你将看到一个具有以下结构的文件夹：

```cs
/ToDoListApp |-- /bin |-- /obj |-- /Pages
|-- /Properties |-- /wwwroot |-- appsettings.json |-- appsettings.Development.json |-- Program.cs
|-- ToDoListApp.csproj
|ToDoList.sln
```

1.  在 Visual Studio Code 中打开根文件夹。

这些文件夹中的一些文件将在接下来的章节中介绍。现在，请考虑以下结构：

+   `bin` 是在应用程序构建后放置最终二进制文件的文件夹。

+   `obj` 是在构建过程中编译器放置中间输出的文件夹。

+   `Pages` 是放置应用程序 Razor Pages 的文件夹。

+   `Properties` 是包含 `launchSettings.json` 文件的文件夹，这是一个放置运行配置的文件。在此文件中，你可以定义一些本地运行的配置，例如环境变量和应用程序端口。

+   `wwwroot` 是放置应用程序所有静态文件的文件夹。

+   `appsettings.json` 是一个配置文件。

+   `appsettings.Development.json` 是开发环境的配置文件。

+   `Program.cs` 是自第一章“Hello C#”以来您所看到的程序类。它是应用程序的入口点。

现在您已经知道在 .NET 6.0 中，位于文件夹根目录的 `Program.cs` 文件使 `WebApplication` 生命周期开始，您可以在下一节中更深入地探索 `Program.cs`。

## Program.cs 和 WebApplication

如前所述，`Program.cs` 是任何 C# 应用程序的入口点。在本节中，您将了解一个典型的 ASP.NET 应用程序的 `Program` 类是如何构建的。以下是一个 `Program.cs` 的示例，它描述了一个非常简单的 ASP.NET 应用程序：

```cs
Program.cs
var builder = WebApplication.CreateBuilder(args);
// Add services to the container.
builder.Services.AddRazorPages();
var app = builder.Build();
// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
app.UseExceptionHandler("/Error");
// The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
The complete code can be found here: https://packt.link/tX9iK.
```

这里首先执行的是创建一个 `WebApplicationBuilder` 对象。该对象包含启动 ASP.NET 基本 Web 应用程序所需的一切——配置、日志记录、依赖注入、服务注册、中间件和其他主机配置。这个主机负责管理 Web 应用程序的生命周期；它们设置一个 Web 服务器和一个基本的 HTTP 管道来处理 HTTP 请求。

如您所见，通过几行代码就能完成许多事情，这使您能够运行一个结构良好的 Web 应用程序，这非常令人印象深刻。ASP.NET 做所有这些是为了让您能够专注于通过您将要构建的功能提供价值。

注意

Bootstrap 是一个用于美化网页内容的 CSS 库。您可以在官方网站上了解更多信息。

## 中间件

将中间件想象成连接在一起的小型应用程序组件，以形成一个处理 HTTP 请求和响应的管道。每个组件都是一个可以在管道中执行其他组件之前或之后执行一些工作的组件。它们还通过 `next()` 调用相互连接，如图 7.1 所示：

![图 7.1：HTTP 管道的中间件](img/B16835_07_01.jpg)

图 7.1：HTTP 管道的中间件

中间件是一个庞大的宇宙。以下列表定义了构建 Web 应用程序的主要特征：

+   中间件的放置顺序很重要。由于它们一个接一个地链接，每个组件的放置都会影响管道的处理方式。

+   如图 7.1 所示，`before` 逻辑会一直执行，直到最终到达端点。一旦到达端点，管道将继续使用 `after` 逻辑处理响应。

+   `next()` 是一个方法调用，它将在执行当前中间件的 `after` 逻辑之前，执行管道中的下一个中间件。

在 ASP.NET 应用程序中，可以在 `WebApplicationBuilder` 使用 `WebApplication?` 对象作为结果调用 `Build` 方法之后，在 `Program.cs` 文件中定义中间件。

在 *Program.cs 和 WebApplication* 部分创建的应用程序中，已经包含了一组为新的样板 Razor Pages 应用程序预置的中间件，当 **HTTP 请求** 到达时，这些中间件将按顺序被调用。

这很容易配置，因为 `WebApplication` 对象包含一个通用的 `UseMiddleware<T>` 方法。此方法允许您创建中间件并将其嵌入到 HTTP 管道中，用于请求和响应。当在 `Configure` 方法中使用时，每次应用程序收到一个传入请求，该请求将按照在 `Configure` 方法中放置请求的顺序通过所有中间件。默认情况下，ASP.NET 提供基本的错误处理、自动重定向到 HTTPS、服务静态文件，以及一些基本的路由和授权。

然而，您可能会在 `Program.cs` 文件中注意到，在 *Program.cs 和 WebApplication* 部分没有 `UseMiddleware<>` 调用。这是因为您可以编写扩展方法来给代码提供一个更简洁的名称和可读性，并且 ASP.NET 框架默认情况下已经为一些内置中间件做了这件事。例如，考虑以下示例：

```cs
using Microsoft.AspNetCore.HttpsPolicy;
public static class HttpsPolicyBuilderExtensions
{
public static IApplicationBuilder UseHttpsRedirection(this WebApplication app)
      { 
           app.UseMiddleware<HttpsRedirectionMiddleware>();
           return app;
}
}
```

在这里，使用了内置的 `UseHttpsRedirection` 扩展方法的一个示例来启用重定向中间件。

## 日志记录

日志记录可能被理解为将应用程序所做的一切都写入输出的简单过程。这个输出可能是控制台应用程序、文件，甚至是第三方日志监控应用程序，例如 ELK Stack 或 Grafana。日志记录在理解应用程序行为方面占有重要位置，尤其是在错误跟踪方面。这使得它成为一个重要的概念需要学习。

使 ASP.NET 成为有效企业应用平台的一个因素是其模块化。由于它是建立在抽象之上的，因此任何新的实现都可以轻松完成，而无需将太多内容加载到框架中。**日志记录**抽象就是这些之一。

默认情况下，`Program.cs` 中创建的 `WebApplication` 对象在这些日志抽象之上添加了一些日志提供程序，它们是 `Console`、`Debug`、`EventSource` 和 `EventLog`。后者——`EventLog`——是仅针对 Windows 操作系统的先进功能。这里的重点将是 `Console` 日志提供程序。正如其名所示，此提供程序将所有记录的信息输出到您的应用程序控制台。您将在本节后面的内容中了解更多关于它的信息。

由于日志基本上记录了应用程序所做的所有事情，你可能会想知道这些日志是否会变得非常大，尤其是对于大型应用程序。它们可能会，但在编写应用程序日志时，一个重要的事情是要掌握日志的**严重性**。可能有一些信息是至关重要的，例如意外的异常。也可能有一些信息你只想记录到开发环境中，以便更好地了解某些行为。话虽如此，.NET 中的日志有七个可能的日志级别，它们是：

+   `跟踪` = 0

+   `调试` = 1

+   `信息` = 2

+   `警告` = 3

+   `错误` = 4

+   `关键` = 5

+   `无` = 6

输出到提供者的级别是通过设置环境变量或通过 `Logging:LogLevel` 部分中的 `appSettings.json` 文件来定义的，如下例所示：

```cs
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "ToDoListApp": "Warning",
      "ToDoListApp.Pages": "Information"
    }
  }
}
```

在此文件中，有日志类别，它们是 `默认` 类别或想要设置日志的类型命名空间的一部分。这正是这些命名空间存在的原因。例如，你可以在命名空间内部为文件设置不同的日志级别。

在前面的示例配置中，整个 `ToDoListApp` 是一个只使用 `LogLevel` 等于或高于 `Warning` 的 `LogLevel` 来记录日志的命名空间。你还指定了，对于 `ToDoListApp.Pages` 类别/命名空间，应用程序将记录所有日志，其级别等于或高于 `Information`。这意味着在更具体的命名空间上的更改会覆盖在更高级别设置的设置。

这一节向你展示了如何配置应用程序的日志级别。有了这些知识，你现在可以理解以下章节中讨论的 DI 概念。

## 依赖注入

依赖注入（DI）是 ASP.NET 框架原生支持的技巧。它是实现面向对象编程中一个著名概念的一种形式，称为控制反转（IoC）。

任何对象需要以功能正常运作的组件都可以称为依赖项。在类的例子中，这可能会指需要构造的参数。在方法的例子中，这可能是参数执行所需的那个方法。使用 IoC 和依赖项意味着将创建类的责任委托给框架，而不是手动完成所有事情。

在 *第二章*，*构建高质量的面向对象代码* 中，你学习了接口。接口基本上是建立契约的常见形式。它们允许你关注调用结果的输出，而不是执行方式。当你使用 IoC 时，你的依赖项现在可以是接口而不是具体类。这允许你的类或方法专注于这些接口建立的契约，而不是实现细节。这带来了以下优势：

+   你可以轻松地替换实现，而不会影响任何依赖于这些契约的类。

+   它解耦了应用程序边界和模块，因为通常不需要任何硬编码的依赖项。

+   它使测试变得更容易，允许你将这些显式依赖项作为模拟或伪造，并专注于行为而不是真实实现细节。

假设现在为了创建应用程序的中间件，你需要构建它们的每个依赖项，并且有很多中间件在构造函数中相互链接。显然，这将是一个繁琐的过程。此外，测试任何此类中间件都将是一个繁琐的过程，因为你需要依赖每个具体的实现来创建一个对象。

通过注入依赖项，你告诉编译器如何构建一个在其构造函数中声明了依赖项的类。DI 机制在运行时执行此操作。这相当于告诉编译器，每当它找到特定类型的依赖项时，它应该使用适当的类实例来解析它。

ASP.NET 提供了一个本地的依赖注入容器，该容器存储了有关如何解析类型的所有信息。接下来，你将学习如何将此信息存储在容器中。

在 `Program.cs` 文件中，你会看到调用 `builder.Services.AddRazorPages()`。`Services` 属性是 `IServiceCollection` 类型，它包含注入到容器中的所有依赖项——也称为服务。许多 ASP.NET 应用程序运行所需的依赖项已经在 `Program.cs` 文件顶部的 `WebApplication.CreateBuilder(args)` 方法中注入。例如，一些本地的日志依赖项将在下一个练习中看到。

## 练习 7.01：创建自定义日志中间件

在这个练习中，你将创建一个自定义日志中间件，该中间件将输出 HTTP 请求的详细信息及其持续时间到控制台。创建后，你将将其放置在 HTTP 管道中，以便它被应用程序接收到的每个请求调用。其目的是为你提供一个关于中间件、日志和 DI 的第一个实际介绍。

以下步骤将帮助你完成此练习：

1.  创建一个名为 `Middlewares` 的新文件夹。

1.  在此文件夹内，创建一个名为 `RequestLoggingMiddleware` 的新类。

1.  创建一个名为 `RequestDelegate` 的新私有 `readonly` 字段，并在构造函数中初始化此字段：

    ```cs
    private readonly RequestDelegate _next;
    public RequestLoggingMiddleware(RequestDelegate next)
    {
        _next = next; 
    }
    ```

这是 ASP.NET 收集的下一个要在 HTTP 管道中执行的中间件引用。通过初始化此字段，你可以调用已注册的下一个中间件。

1.  在 `System.Diagnostics` 命名空间中添加一个 `using` 语句，以便添加一个名为 `Stopwatch` 的特殊类。它将被用来测量请求时间长度：

    ```cs
    using System.Diagnostics;
    ```

1.  创建一个私有的 `readonly ILogger` 字段。`ILogger` 接口是 .NET 提供的默认接口，用于手动记录信息。

1.  之后，在 `ILoggerFactory` 类型的构造函数中放置第二个参数。此接口是 .NET 提供的另一个接口，允许您创建 `ILogger` 对象。

1.  使用此工厂的 `CreateLogger<T>` 方法创建一个日志记录器对象：

    ```cs
    private readonly ILogger _logger;
    private readonly RequestDelegate _next;
    public RequestLoggingMiddleware(RequestDelegate next, ILoggerFactory loggerFactory)
    {
        _next = next; 
        _logger = loggerFactory.CreateLogger<RequestLoggingMiddleware>();
    }
    ```

在这里，`T` 是一个泛型参数，它指的是一个类型，即日志类别，如 *日志* 部分所示。在这种情况下，该类别将是执行日志记录的类的类型，即 `RequestLoggingMiddleware` 类。

1.  一旦初始化了字段，创建一个新的方法，其签名如下：

    ```cs
    public async Task InvokeAsync(HttpContext context) { }
    ```

1.  在此方法内部，声明一个名为 `Stopwatch` 的变量，并将其赋值为 `Stopwatch.StartNew()`：

    ```cs
    var stopwatch = Stopwatch.StartNew();
    ```

`Stopwatch` 类是一个辅助类，用于从调用 `.StartNew()` 方法的那一刻起测量执行时间。

1.  在此变量之后，编写一个 `try-catch` 块，其中包含调用下一个请求以及从 `stopwatch` 的 `.Stop()` 方法调用以测量 `_next()` 调用所花费的时间：

    ```cs
    using System.Diagnostics;
    namespace ToDoListApp.Middlewares;
    public class RequestLoggingMiddleware
    {
        private readonly ILogger _logger;
        private readonly RequestDelegate _next;
        public RequestLoggingMiddleware(RequestDelegate next, ILoggerFactory loggerFactory)
        {
            _next = next;
            _logger = loggerFactory.CreateLogger<RequestLoggingMiddleware>();
        }
    ```

您还可以在此处理可能的异常。因此，最好将这两个调用包裹在一个 `try-catch` 方法中。

1.  在 `Program.cs` 文件中，通过以下声明调用自定义中间件：

    ```cs
    var app = builder.Build();
    // Configure the HTTP request pipeline.app.UseMiddleware<RequestLoggingMiddleware>();
    ```

在将 `app` 变量赋值的下一行编写它。

1.  最后，在 `Program.cs` 文件中，放置一个 `using` 语句到 `ToDoListApp.Middlewares`：

    ```cs
    Program.cs
    using ToDoListApp.Middlewares;
    var builder = WebApplication.CreateBuilder(args);
    // Add services to the container.
    builder.Services.AddRazorPages();
    var app = builder.Build();
    // Configure the HTTP request pipeline.
    app.UseMiddleware<RequestLoggingMiddleware>();
    if (!app.Environment.IsDevelopment())
    {
        app.UseExceptionHandler("/Error");
    ```

```cs
The complete code can be found here: https://packt.link/tX9iK.
```

1.  要在您的网络浏览器中查看应用程序的运行情况及其在 Visual Studio Code 中的输出，请在地址栏中键入以下命令：

    ```cs
    localhost:####
    ```

这里 `####` 代表端口号。这会因不同的系统而异。

1.  按下回车键后，将显示以下屏幕：

![图 7.2：在浏览器上运行的应用程序](img/B16835_07_02.jpg)

图 7.2：在浏览器上运行的应用程序

1.  每次在 VS Code 中执行练习/活动后，都要执行 *步骤 13*。

1.  在 VS code 终端内部按 `Control+C` 以在执行另一个练习/活动之前中断任务。

1.  在您的浏览器中执行应用程序后，您将在 Visual Studio Code 终端看到类似的输出：

    ```cs
    info: ToDoListApp.Middlewares.RequestLoggingMiddleware[0]
          HTTP GET request for path / with status 200 executed in 301 ms
    info: ToDoListApp.Middlewares.RequestLoggingMiddleware[0]
          HTTP GET request for path /lib/bootstrap/dist/css/bootstrap.min.css with status 200 executed in 18 ms
    info: ToDoListApp.Middlewares.RequestLoggingMiddleware[0]
          HTTP GET request for path /css/site.css with status 200 executed in 1 ms
    info: ToDoListApp.Middlewares.RequestLoggingMiddleware[0]
          HTTP GET request for path /favicon.ico with status 200 executed in 1 ms
    ```

您将观察到控制台日志中记录了带有 HTTP 请求中间件管道中经过的时间的消息。由于您已使用自己的方法声明了它，它应该考虑所有管道链的执行时间。

在这个练习中，您创建了您的第一个中间件——`RequestLoggingMiddleware`。此中间件测量 HTTP 请求的执行时间，在您的 HTTP 管道中。通过将其放置在所有其他中间件之前，您将能够测量整个中间件管道中请求的整个执行时间。

注意

您可以在 [`packt.link/i04Iq`](https://packt.link/i04Iq) 找到此练习使用的代码。

现在想象一下，你有一个 10 到 20 个中间件的 HTTP 管道，每个中间件都有自己的依赖项，你必须手动实例化每个中间件。在这种情况下，IoC 通过将这些类的实例化和它们的依赖项注入委托给 ASP.NET 来派上用场。你已经看到了如何创建使用原生 ASP.NET 日志机制和依赖注入的自定义中间件。

在 ASP.NET 中，日志记录和依赖注入是强大的机制，允许你为应用程序创建非常详细的日志。正如你所看到的，这是通过构造函数中的 `logger` 注入实现的。对于这些日志记录器，你可以通过两种方式创建日志类别的对象：

+   如练习所示，一种方式是注入 `ILoggerFactory`。你可以调用 `CreateLogger(categoryName)` 方法，它接收一个字符串作为参数。你也可以调用 `CreateLogger<CategoryType>()` 方法，它接收一个泛型类型。这种方法更可取，因为它将 `logger` 的类别设置为类型的全名（包括命名空间）。

+   另一种方式是通过注入 `ILogger<CategoryType>`。在这种情况下，类别类型通常是你要注入日志记录器的类的类型，如前一个练习中所示。在前面的练习中，你可以用 `ILogger<RequestLoggingMiddleware>` 替换 `ILoggerFactory` 的注入，并将这个新的注入依赖项直接分配给 `ILogger` 的私有字段，如下所示：

    ```cs
    private readonly ILogger _logger;
    private readonly RequestDelegate _next;
    public RequestLoggingMiddleware(RequestDelegate next, ILogger< RequestLoggingMiddleware> logger)
    {
        _next = next; 
        _logger = logger;
    }
    ```

现在，你知道日志记录和依赖注入是强大的机制，允许你为应用程序创建非常详细的日志。在转向 Razor 页面之前，了解应用程序中对象的生命周期是很重要的。这被称为依赖生命周期。

## 依赖生命周期

在进入本章的下一主题之前，讨论依赖生命周期是很重要的。在前面的练习中使用的所有依赖项都是通过构造函数注入的。但这些依赖项的解析之所以成为可能，仅仅是因为 ASP.NET 在 `Program.cs` 部分之前注册了这些依赖项。在下面的代码中，你可以看到一个 ASP.NET 内置的代码示例，它处理日志依赖项的注册，通过向服务容器添加 `ILoggerFactory` 依赖项：

```cs
LoggingServiceCollectionExtensions.cs
public static IServiceCollection AddLogging(this IServiceCollection services, Action<ILoggingBuilder> configure)
{T
if (services == null)
     {
     throw new ArgumentNullException(nameof(services));
     }
     services.AddOptions();
     services.TryAdd(ServiceDescriptor.Singleton<ILoggerFactory, LoggerFactory>());

services.TryAdd(ServiceDescriptor.Singleton(typeof(ILogger<>), typeof(Logger<>)));
services.TryAddEnumerable(ServiceDescriptor.Singleton<IConfigureOptions<LoggerFilterOptions>>(new DefaultLoggerLevelConfigureOptions(LogLevel.Information)));
configure(new LoggingBuilder(services));
return services;
}
The complete code can be found here: https://packt.link/g4JPp.
```

注意

上述代码是一个来自标准库并内置在 ASP.NET 中的示例，它处理日志依赖项的注册。

这里发生了很多事情，但需要考虑的两个重要事项如下：

+   这里的方法是 `TryAdd`，它将依赖项注册到 DI 容器中。

+   `ServiceDescriptor.Singleton` 方法定义了依赖生命周期。这是本章 *依赖注入* 部分的最后一个重要概念。

依赖生命周期描述了应用程序中对象的生命周期。ASP.NET 有三个默认的生命周期，可以用来注册依赖项：

+   临时（Transient）：具有这种生命周期的对象每次请求时都会创建，并在使用后销毁。这对于无状态依赖项非常有效，即当它们被调用时不需要保持状态。例如，如果你需要连接到 HTTP API 来请求一些信息，你可以注册一个具有这种生命周期的依赖项，因为 HTTP 请求是无状态的。

+   作用域（Scoped）：具有作用域生命周期的对象为每个客户端连接创建一次。例如，在 HTTP 请求中，作用域依赖项在整个请求期间将具有相同的实例，无论它被调用多少次。这种依赖项在一段时间内携带一些状态。在连接结束时，依赖项将被销毁。

+   单例（Singleton）：具有单例生命周期的对象在整个应用程序的生命周期内只创建一次。一旦它们被请求，它们的实例将在应用程序运行期间保持。这种生命周期应该仔细考虑，因为它可能会消耗大量内存。

如前所述，这些依赖项的手动注册可以在 `Startup` 类中的 `ConfigureServices` 方法中完成。每个不是由 ASP.NET 自动提供和注册的新依赖项都应该在那里手动注册，了解这些生命周期很重要，因为它们允许应用程序以不同的方式管理依赖项。

你已经了解到，这些依赖项的解析之所以成为可能，仅仅是因为 ASP.NET 注册了三个默认的生命周期，可以用来注册依赖项。现在你将转向 Razor Pages，它允许使用 ASP.NET 提供和驱动的所有功能来构建基于页面的应用程序。

## Razor Pages

现在你已经了解了与 ASP.NET 应用程序相关的主要方面，你将继续构建本章开头开始的应用程序。这里的目的是构建一个待办事项列表应用程序，你可以在 Kanban 风格的看板上轻松创建和管理任务列表。

早期章节中提到了 Razor Pages，但究竟它是什么呢？Razor Pages 是一个框架，它允许使用 ASP.NET 提供和驱动的所有功能来构建基于页面的应用程序。它是为了能够构建具有清晰关注点分离的动态数据驱动应用程序而创建的，也就是说，每个方法和类都有各自但互补的责任。

### 基本 Razor 语法

Razor Pages 使用由 Microsoft 驱动的 Razor 语法，它允许页面具有静态 HTML/CSS/JS、C# 代码和自定义标签助手，这些是可重用组件，它们能够使页面中渲染 HTML 片段。

如果你查看在第一次练习中运行的 `dotnet new` 命令生成的 `.cshtml` 文件，你会注意到大量的 HTML 代码，以及一些带有 `@` 前缀的方法和变量。在 Razor 中，一旦你写下这个符号，编译器就会检测到将编写一些 C# 代码。你已经知道 HTML 是一种用于构建网页的标记语言。Razor 使用它与 C# 结合来创建强大的标记，并结合服务器端渲染的代码。

如果你想放置一个代码块，它可以在括号内完成，如下所示：

```cs
@{ … }
```

在这个块内部，你可以基本上做所有可以用 C# 语法做的事情，从局部变量声明到循环等。如果你想放置一个 `static @`，你必须通过放置两个 `@` 符号来转义它，以便在 HTML 中渲染。例如，在电子邮件 ID（如 `james@@bond.com`）中就是这样做的。

### 文件结构

Razor 页面以 `.cshtml` 扩展名结尾，可能还包含另一个名为 `.cshtml.cs` 的文件。如果你前往应用程序的根目录并导航到 `Pages` 文件夹，你将看到在创建页面时生成的以下结构：

```cs
|-- /Pages
|---- /Shared |------ _Layout.cshtml |------ _ValidationScriptsPartial.cshtml |---- _ViewImports.cshtml
|---- _ViewStart.cshtml
|---- Error.cshtml
|---- Error.cshtml.cs
|---- Index.cshtml
|---- Index.cshtml.cs
|---- Privacy.cshtml
|---- Privacy.cshtml.cs
```

`Index`、`Privacy` 和 `Error` 页面在项目创建后自动生成。简要查看这里的其他文件。

`/Shared` 文件夹包含一个默认用于应用程序的共享 `Layout` 页面。这个页面包含一些共享部分，如导航栏、页眉、页脚和元数据，这些在几乎每个应用程序页面中都会重复：

```cs
_Layout.cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - ToDoListApp</title>    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/ToDoListApp.styles.css" asp-append-version="true" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" asp-area="" asp-page="/Index">ToDoListApp</a>
The complete code can be found here: https://packt.link/2Hb8r.
```

将这些共享部分保存在单个文件中，使得重用性和可维护性更容易。如果你查看在模板中生成的这个 `Layout` 页面，有一些值得强调的事情：

+   默认情况下，Razor Pages 应用程序使用 Twitter Bootstrap 进行设计——一个用于编写美观、简单和响应式网站的库——以及 jQuery 进行基本脚本编写。这可以根据每个应用程序进行自定义，因为这些都是静态文件。

+   有一个特殊的 `RenderBody()` 方法，它指示应用程序页面生成的 HTML 将放置在哪里。

+   另一种名为 `RenderSection()` 的方法，用于按页面渲染预定义的各个部分。例如，当需要仅对某些页面使用某些静态文件（如图像、脚本或样式表）时，它非常有用。这样，你可以在需要这些文件的页面中特定部分内放置这些文件，并在你希望它们被渲染的 HTML 层级上调用 `RenderSection` 方法。这是在 `_Layout.cshtml` 页面上完成的。

`_ViewImports.cshtml` 文件是另一个重要的文件；它使应用程序页面能够共享公共指令，并通过在每一页上放置这些指令来减少工作量。它定义了所有全局使用命名空间、标签辅助器和全局 `Pages` 命名空间的地方。该文件支持的指令如下：

+   `@namespace`：用于设置 `Pages` 的基本命名空间。

+   `@inject`：用于在页面中放置依赖注入。

+   `@model`：包含 `PageModel` 类，该类将确定页面将处理哪些信息。

+   `@using`：类似于 `.cs` 文件，这个指令允许你在 Razor 页面的顶级完全限定命名空间，以避免在代码中重复这些命名空间。

`_ViewStart.cshtml` 文件用于放置在每个页面调用开始时执行的代码。在这个页面上，你定义 `Layout` 属性并设置 `Layout` 页面。

现在你已经熟悉了 Razor Pages 的基础知识，是时候开始着手你的应用程序并深入研究一些更有趣的话题了。你将从创建待办事项应用程序的基本结构开始。

## 练习 7.02：使用 Razor 创建看板

本练习的目标是使用其第一个组件——看板来开始创建待办事项应用程序。这个看板用于控制工作流程，人们可以将他们的工作分成卡片，并在不同的状态之间移动这些卡片，例如待办、进行中和完成。使用这种功能的流行应用程序是 Trello。在本章中，我们将使用与 *练习 7.01* 中创建的相同的 `ToDoListApp` 项目来学习新概念并逐步发展应用程序，包括本练习。执行以下步骤：

1.  导航到你的应用程序的根目录，并创建一个名为 `Models` 的文件夹。

1.  在 `Models` 文件夹中，创建一个新的 `enum` 类 `ETaskStatus`，包含 `ToDo`、`Doing` 和 `Done` 选项：

    ```cs
    public enum ETaskStatus {
    ToDo,
    Doing,
    Done
    }
    ```

1.  再次，在 `Models` 文件夹中，创建一个新的类 `ToDoTask`，它将被用来为待办事项列表创建一个新的任务，具有以下属性：

    ```cs
    namespace ToDoListApp.Models;
    public class ToDoTask
    {
        public Guid Id { get; set; }
        public DateTime CreatedAt { get; set; }
        public DateTime? DueTo { get; set; }
        public string Title { get; set; }
        public string? Description { get; set; }
        public ETaskStatus Status { get; set; }
    }
    ```

1.  为 `ToDoTask` 类创建两个构造函数，如下所示：

    ```cs
    ToDoTask.cs
    namespace ToDoListApp.Models;
    public class ToDoTask
    {
        public ToDoTask()
        {
            CreatedAt = DateTime.UtcNow;
            Id = Guid.NewGuid();
        }
        public ToDoTask(string title, ETaskStatus status) : this()
        {
            Title = title;
            Status = status;
        }
    ```

```cs
The complete code can be found here: https://packt.link/nFk00.
```

创建一个不带参数的，用于设置 `Id` 和 `CreatedAt` 属性的默认值，另一个带有小写命名的参数，用于初始化前面类的 `Title` 和 `Status` 属性。

`Pages`/ `Index.cshtml` 文件是在你的应用程序样板中自动生成的。这个页面将是你的应用程序的入口点。

1.  现在，通过编辑文件 `Pages`/ `Index.cshtml.cs` 并替换以下代码中的样板代码来自定义它：

    ```cs
    Index.cshtml.cs
    using System.Collections.Generic;
    using Microsoft.AspNetCore.Mvc.RazorPages;
    using ToDoListApp.Models;
    namespace ToDoListApp.Pages;
    public class IndexModel : PageModel
    {
        public IList<ToDoTask> Tasks { get; set; } = new List<ToDoTask>();
        public IndexModel()
        {
        }
    ```

```cs
The complete code can be found here: https://packt.link/h8mni.
```

基本上，这段代码填充了你的模型。在这里，`PageModel` 类的 `OnGet` 方法被用来通知应用程序，当页面加载时，它应该使用分配给 `Task` 的属性填充模型。

1.  将 `Pages`/ `Index.cshtml` 中的代码替换为以下代码，以创建你的看板和任务卡片：

    ```cs
    Index.cshtml
    @page
    @using ToDoListApp.Models
    @model IndexModel
    @{
        ViewData["Title"] = "My To Do List";
    }
    <div class="text-center">
        <h1 class="display-4">@ViewData["Title"]</h1>
        <div class="row">
            <div class="col-4">
                <div class="card bg-light">
                    <div class="card-body">
                        <h6 class="card-title text-uppercase text-truncate py-2">To Do</h6>
                        <div class="border border-light">
    ```

```cs
The complete code can be found here: https://packt.link/IhELU.
```

这个页面是你的视图。它共享 `Pages`/ `Index.cshtml.cs` 类（也称为代码后类）的属性。当你为代码后类中的 `Tasks` 属性赋值时，它对视图可见。使用这个属性，你可以从页面填充 HTML。 

1.  现在，使用 `dotnet run` 命令运行你的应用程序。当应用程序在浏览器中加载时，你将在 `Index` 页面上看到以下内容：

![图 7.3：显示你的第一个应用程序，看板](img/B16835_07_03.jpg)

图 7.3：显示你的第一个应用程序，看板

注意，目前应用程序不包含任何逻辑。你在这里构建的只是一个由 `PageModel` 数据驱动的 UI。

注意

你可以在 [`packt.link/1PRdq`](https://packt.link/1PRdq) 找到用于此练习的代码。

正如你在 *练习 7.02* 中所看到的，为每个创建的页面都有两种主要类型的文件，即 `.cshtml` 和 `.cshtml.cs` 文件。这些文件构成了每个 Razor 页面的基础。下一节将详细介绍文件名后缀的差异以及这两个文件是如何相互补充的。

## PageModel

在 *练习 7.02* 中创建的 `Index.cshtml.cs` 文件中，你可能已经注意到其中的类继承自 `PageModel` 类。拥有这个代码后置类提供了一些优势——例如，客户端和服务器之间关注点的清晰分离——这使得维护和开发更加容易。它还使你能够为放置在服务器上的逻辑创建单元和集成测试。你将在 *第十章*，*自动化测试* 中了解更多关于测试的内容。

`PageModel` 可能包含一些绑定到视图的属性。在 *练习 7.02* 中，`IndexModel` 页面有一个属性是 `List<ToDoTask>`。这个属性在页面加载时的 `OnGet()` 方法中被填充。那么填充是如何发生的呢？下一节将讨论填充属性的生命周期以及在 `PageModel` 中使用它们。

### 带有页面处理器的生命周期

处理程序方法是 Razor Pages 的核心功能。当服务器从页面接收到请求时，这些方法会自动执行。例如，在 *练习 7.02* 中，每次页面接收到 `GET` 请求时，`OnGet` 方法都会被执行。

按照惯例，处理程序方法将根据请求的 HTTP 动词来回答。例如，如果你想在一个 `POST` 请求之后执行某些操作，你应该有一个 `OnPost` 方法。同样，在 `PUT` 请求之后，你应该有一个 `OnPut` 方法。这些方法中的每一个都有一个异步等效方法，它改变了方法的签名；方法名后添加了 `Async` 后缀，并且它返回一个 `Task` 属性而不是 `void`。这也使得 `await` 功能可用于该方法。

然而，有一个棘手的情况，你可能希望表单使用相同的 HTTP 动词执行多个操作。在这种情况下，你可以在后端执行一些令人困惑的逻辑来处理不同的输入。然而，Razor Pages 为你提供了一个开箱即用的功能，称为`asp-page-handler`，允许你指定在服务器上被调用的处理程序名称。标签辅助器将在下一节中讨论，但在此阶段，请考虑以下代码作为示例。该代码包含一个 HTML 表单，包含两个提交按钮，用于执行两个不同的操作——一个用于创建订单，另一个用于取消订单：

```cs
<form method="post">
    <button asp-page-handler="PlaceOrder">Place Order</button>
    <button asp-page-handler="CancelOrder">Cancel Order</button>
</form>
```

在服务器端，你只需要有两个处理程序，每个操作一个，如下面的代码所示：

```cs
public async Task<IActionResult> OnPostPlaceOrderAsync()
{
    // …
}
public async Task<IActionResult> OnPostCancelOrderAsync()
{
    // …
}
```

在这里，页面的后台代码与`.cshtml`文件上的`form`方法和`asp-page-handler`标签的值相匹配，与代码后台文件中的方法名称相匹配。这样，你可以在同一个表单中为相同的 HTTP 动词执行多个操作。

关于这个主题的最后一句话是，在这种情况下，服务器上的方法名称应该写成如下格式：

```cs
On + {VERB} + {HANDLER}
```

这可以带有或不带有`Async`后缀。在前面的例子中，`OnPostPlaceOrderAsync`方法是`PlaceOrder`按钮的`PlaceOrder`处理程序，而`OnPostCancelOrderAsync`是`CancelOrder`按钮的处理程序。

### 使用标签辅助器渲染可重用静态代码

你可能已经注意到，之前编写的 HTML 代码很长。你创建了看板卡片、列表和一个板来包裹所有内容。如果你仔细查看代码，会发现整个代码中都有相同的模式重复出现。这引发了一个主要问题，维护。很难想象需要处理、维护和演变所有这些纯文本。

幸运的是，标签辅助器在这方面可以非常有用。它们基本上是渲染静态 HTML 代码的组件。ASP.NET 有一组内置的标签辅助器，具有自定义服务器端属性，如锚点、表单和图像。标签辅助器是一个核心功能，有助于使高级概念易于处理，例如模型绑定，这将在稍后进一步讨论。

除了为内置 HTML 标签添加渲染功能外，它们还是实现静态和重复代码可重用性的令人印象深刻的方法。在下一个练习中，你将学习如何创建自定义标签辅助器。

## 练习 7.03：使用标签辅助器创建可重用组件

在这个练习中，你将改进上一个练习中的工作。这里的改进是将可以复用的部分代码移动到自定义标签辅助器中，以简化 HTML 代码。

要做到这一点，请执行以下步骤：

1.  打开与你的应用程序一起创建的`_ViewImports.cshtml`文件。

1.  将以下行添加到末尾，以定义自定义标签辅助器的内容`@addTagHelper`指令：

    ```cs
    @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
    @addTagHelper *, ToDoListApp 
    ```

在前面的代码中，你使用星号 (`*`) 添加了此命名空间中存在的所有自定义标签助手。

1.  现在，在项目的根目录（`ToDoApp`）下创建一个名为 `TagHelpers` 的新文件夹。

1.  在此文件夹内创建一个名为 `KanbanListTagHelper.cs` 的新类。

1.  让这个类继承自 `TagHelper` 类：

    ```cs
    namespace ToDoListApp.TagHelpers;
    ```

1.  这种继承使得 ASP.NET 能够识别内置和自定义的标签助手。

1.  现在，在 `Microsoft.AspNetCore.Razor.TagHelpers` 命名空间下放置一个 `using` 语句：

    ```cs
    using Microsoft.AspNetCore.Razor.TagHelpers;
    namespace ToDoListApp.TagHelpers;
    public class KanbanListTagHelper : TagHelper
    {
    }
    ```

1.  对于 `KanbanListTagHelper` 类，创建两个名为 `Name` 和 `Size` 的字符串属性，并具有获取器和设置器：

    ```cs
    using Microsoft.AspNetCore.Razor.TagHelpers;
    namespace ToDoListApp.TagHelpers;
    public class KanbanListTagHelper : TagHelper
    {
        public string? Name { get; set; }
        public string? Size { get; set; }
    }
    ```

1.  使用以下代码重写基类的异步 `ProcessAsync (TagHelperContext context, TagHelperOutput)` 输出方法：

    ```cs
    KanbanListTagHelper.cs
    public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
    {
         output.TagName = "div";
         output.Attributes.SetAttribute("class", $"col-{Size}");
         output.PreContent.SetHtmlContent(
         $"<div class=\"card bg-light\">"
              + "<div class=\"card-body\">"
              + $"<h6 class=\"card-title text-uppercase text-truncate py-     2\">{Name}</h6>"
              + "<div class \"border border-light\">");
         var childContent = await output.GetChildContentAsync();
         output.Content.SetHtmlContent(childContent.GetContent());
    ```

```cs
The complete code can be found here: https://packt.link/bjFIk.
```

每个标签助手都有一个标准的 HTML 标签作为输出。这就是为什么在方法开始时，从 `TagHelperOutput` 对象中调用 `TagName` 属性来指定将用作输出的 HTML 标签。此外，你还可以通过从 `TagHelperOutput` 对象中调用 `Attributes` 属性及其 `SetAttribute` 方法来设置此 HTML 标签的属性。这正是你在指定 HTML 输出标签之后所做的事情。

1.  现在，创建另一个名为 `KanbanCardTagHelper.cs` 的类，具有相同的继承和命名空间使用语句，如前所述：

    ```cs
    namespace ToDoListApp.TagHelpers;
    using Microsoft.AspNetCore.Razor.TagHelpers;
    public class KanbanCardTagHelper: TagHelper
    {
        public string? Task { get; set; }
    }
    ```

对于这个类，创建一个名为 `Task` 的 `string` 属性，并具有公共的获取器和设置器：

1.  在这个新类中，重写基类的同步 `Process(TagHelperContext context, TagHelperOutput output)` 方法。在这个方法中，编写以下代码：

    ```cs
    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
         output.TagName = "div";
         output.Attributes.SetAttribute("class", "card");
         output.PreContent.SetHtmlContent(
         "<div class=\"card-body p-2\">"
              + "<div class=\"card-title\">");
         output.Content.SetContent(Task);
         output.PostContent.SetHtmlContent(
         "</div>"
              + "<button class=\"btn btn-primary btn-sm\">View</button>"
              + "</div>");
    output.TagMode = TagMode.StartTagAndEndTag;
    }
    ```

一个重要的概念是了解 HTML 内容如何在标签助手内放置。正如你所见，代码使用了 `TagHelperOutput` 对象的三个不同属性来放置内容：

+   `PreContent`

+   `Content`

+   `PostContent`

预设和后置属性对于在生成内容之前和之后设置内容非常有用。它们的一个用例是当你想要设置固定内容作为 `div` 容器、标题和页脚时。

你在这里做的另一件事是通过 `Mode` 属性设置标签助手如何被渲染。你使用了 `TagMode.StartTagAndEndTag` 作为值，因为你使用了 `div` 容器作为标签助手输出，而 `div` 元素在 HTML 中既有开始标签也有结束标签。如果输出标签是其他 HTML 元素，例如自闭合的电子邮件，你会使用 `TagMode.SelfClosing` 代替。

1.  最后，转到 Pages 文件夹下的 `Index.cshtml` 文件，并用标签助手替换 *练习 7.02* 中创建的 HTML，以使你的代码更简洁：

    ```cs
    Index.cshtml 
    @page
    @using ToDoListApp.Models
    @model IndexModel
    @{
        ViewData["Title"] = "My To Do List";
    }
    <div class="text-center">
        <h1 class="display-4">@ViewData["Title"]</h1>
        <div class="row">
            <kanban-list name="To Do" size="4">
                @foreach (var task in Model.Tasks.Where(t => t.Status == ETaskStatus.ToDo))
                {
                    <kanban-card task="@task.Description">
                    </kanban-card>
    ```

```cs
The complete code can be found here: https://packt.link/YIgdp.
```

1.  现在用以下命令运行应用程序：

    ```cs
    dotnet run
    ```

1.  在你的浏览器中，导航到 Visual Studio 控制台输出提供的 localhost:#### 地址，就像你在上一个练习中所做的那样：

![图 7.4：浏览器中显示的前端](img/B16835_07_04.jpg)

图 7.4：浏览器中显示的前端

你将在前端看到与之前相同的结果，如*图 7.3*所示。改进之处在于，尽管输出相同，你现在拥有一个更加模块化和简洁的代码来维护和演进。

注意

你可以在[`packt.link/YEdiU`](https://packt.link/YEdiU)找到用于此练习的代码。

在这个练习中，你使用了标签辅助器来创建可重用的组件，这些组件生成静态 HTML 代码。你现在可以看到 HTML 代码变得更加干净和简洁。下一节将详细介绍通过使用模型绑定将代码背后的内容与你的 HTML 视图链接起来创建交互式页面。

## 模型绑定

到目前为止，你已经涵盖了有助于为待办事项应用程序打下基础的概念。作为一个快速回顾，主要点如下：

+   `PageModel` 用于向页面添加数据。

+   标签辅助器为服务器生成的 HTML 添加了自定义的静态渲染。

+   处理程序方法定义了页面与 HTTP 请求交互的方式。

一个至关重要的最终概念，是构建 Razor Pages 应用程序的核心，即模型绑定。在处理程序方法中用作参数的数据以及通过页面模型传递的数据是通过此机制渲染的。它包括从 HTTP 请求中提取键/值对，并根据绑定的方向（即数据是从客户端移动到服务器还是从服务器移动到客户端），将它们放置在客户端 HTML 或服务器端代码中。

这些数据可能放置在路由、表单或查询字符串中，并与 .NET 类型绑定，无论是原始类型还是复杂类型。*练习 7.04* 将有助于阐明模型绑定在从客户端到服务器时的运作方式。

## 练习 7.04：创建一个提交任务的页面

本练习的目标是创建一个新页面。它将被用来创建将在看板中显示的新任务。按照以下步骤完成此练习：

1.  在项目根目录文件夹中，运行以下命令：

    ```cs
    dotnet add package Microsoft.EntityFrameworkCore
    dotnet add package Microsoft.EntityFrameworkCore.Sqlite
    dotnet add package Microsoft.EntityFrameworkCore.Design
    ```

1.  在项目根目录中创建一个名为 `Data` 的新文件夹，并在其中包含一个 `ToDoDbContext` 类。这个类将继承自 Entity Framework 的 `DbContext` 并用于访问数据库。

1.  现在将其中的以下代码添加进去：

    ```cs
    using Microsoft.EntityFrameworkCore;
    using ToDoListApp.Models;
    namespace ToDoListApp.Data;
    public class ToDoDbContext : DbContext
    {
        public ToDoDbContext(DbContextOptions<ToDoDbContext> options) : base(options)
        {
        }
        public DbSet<ToDoTask> Tasks { get; set; } 
    }
    ```

1.  更新你的 `Program.cs` 文件以匹配以下内容：

    ```cs
    Program.cs
    using Microsoft.EntityFrameworkCore;
    using ToDoListApp.Data;
    using ToDoListApp.Middlewares;
    var builder = WebApplication.CreateBuilder(args);
    // Add services to the container.builder.Services.AddRazorPages();
    builder.Services.AddDbContext<ToDoDbContext>(opt => opt.UseSqlite("Data Source=Data/ToDoList.db")); 
    var app = builder.Build();
    // Configure the HTTP request pipeline.app.UseMiddleware<RequestLoggingMiddleware>();
    ```

```cs
The complete code can be found here: https://packt.link/D4M8o.
```

此更改将在 DI 容器中注册 `DbContext` 依赖项，并设置数据库访问。

1.  在终端上运行以下命令来安装 `dotnet ef` 工具。这是一个 CLI 工具，它将帮助你与数据库辅助器进行迭代，例如架构创建和更新：

    ```cs
    dotnet tool install --global dotnet-ef
    ```

1.  现在，构建应用程序并在终端上运行以下命令：

    ```cs
    dotnet ef migrations add 'FirstMigration'
    dotnet ef database update
    ```

这些命令将创建一个新的迁移，该迁移将从你的数据库创建架构并将此迁移应用到数据库中。

1.  迁移运行并数据库更新后，在 `Pages` 文件夹内创建一个名为 `Tasks` 的新文件夹。

1.  将`Index`页面文件——`index.cshtml`和`index.cshtml.cs`——移动到`Tasks`文件夹。

1.  接下来，将`Program.cs`中的`AddRazorPages`调用替换为以下调用：

    ```cs
    builder.Services.AddRazorPages(opt =>{    opt.Conventions.AddPageRoute("/Tasks/Index", ""); });
    ```

这将为页面路由添加一个约定。

1.  用以下代码替换`_Layout.cshtml`文件中的标题标签（位于`Pages/Shared/`）以创建应用的共享`navbar`：

    ```cs
    <header>
            <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
                <div class="container">
                    <a class="navbar-brand" asp-area="" asp-page="/Index">MyToDos</a>
                    <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                            aria-expanded="false" aria-label="Toggle navigation">
                        <span class="navbar-toggler-icon"></span>
                    </button>
                    <div class="navbar-collapse collapse d-sm-inline-flex flex-sm-row-reverse">
                        <ul class="navbar-nav flex-grow-1">
                            <li class="nav-item">
                                <a class="nav-link text-dark" asp-area="" asp-page="/tasks/create">Create Task</a>
                            </li>
                        </ul>
                    </div>
                </div>
            </nav>
        </header>
    ```

这个`navbar`将允许你访问新创建的页面。

1.  创建`Create.cshtml`页面（位于`Pages/Tasks/`），并添加以下代码：

    ```cs
    Create.cshtml
    @page "/tasks/create"
    @model CreateModel
    @{
        ViewData["Title"] = "Task";
    }
    <h2>Create</h2>
    <div>
        <h4>@ViewData["Title"]</h4>
        <hr />
        <dl class="row">
            <form method="post" class="col-6">
                <div class="form-group">
                    <label asp-for="Task.Title"></label>
                    <input asp-for="Task.Title" class="form-control" />
    ```

```cs
The complete code can be found here: https://packt.link/2NjdN.
```

这应该包含一个表单，该表单将使用`PageModel`类来创建新任务。对于每个表单输入字段，在`input`标签辅助器内部使用`asp-for`属性。此属性负责在`name`属性中用适当的值填充 HTML 输入。

由于你正在将页面模型中的复杂属性`Task`进行绑定，名称值使用以下语法生成：

```cs
{PREFIX}_{PROPERTYNAME} pattern
```

这里`PREFIX`是`PageModel`上的复杂对象名称。因此，对于一个任务的 ID，客户端会生成一个带有`name="Task_Id"`的输入，并且输入通过具有来自服务器的`Task.Id`属性值的`value`属性进行填充。在页面的情况下，因为你正在创建一个新任务，字段之前没有被预先填充。这是因为你使用`OnGet`方法将一个新对象分配给了`PageModel`类的`Task`属性。

1.  现在，创建名为`CreateModel.cshtml.cs`的后台代码页面（位于`Pages/Tasks/`）：

    ```cs
    Create.cshtml.cs
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.AspNetCore.Mvc.RazorPages;
    using ToDoListApp.Data;
    using ToDoListApp.Models;
    namespace ToDoListApp.Pages.Tasks;
    public class CreateModel : PageModel {
        private readonly ToDoDbContext _context;
        public CreateModel(ToDoDbContext context)
        {
            _context = context;
        }
    ```

```cs
The complete code can be found here: https://packt.link/06ciR.
```

当提交表单时，表单内的所有值都放置在传入的`HttpRequest`中。`TryUpdateModelAsync`的调用尝试使用来自客户端的请求中的这些值填充一个对象。由于表单是用前面解释过的格式创建的，此方法知道如何提取这些值并将它们绑定到对象上。简单来说，这就是模型绑定的魔法。

1.  现在，用以下代码替换`Index.cshtml`（位于`Pages/Tasks/`）的代码：

    ```cs
    Index.cshtml
    @page
    @using ToDoListApp.Models
    @model IndexModel
    @{
        ViewData["Title"] = "My To Do List";
    }
    <div class="text-center">
        @if (TempData["SuccessMessage"] != null)
        {
            <div class="alert alert-success" role="alert">
                @TempData["SuccessMessage"]
            </div>
        }
        <h1 class="display-4">@ViewData["Title"]</h1>
    ```

```cs
The complete code can be found here: https://packt.link/hNOTx.
```

此代码添加了一个部分，用于显示如果`TempData`字典中存在带有`SuccessMessage`键的条目时将显示的警告。

1.  最后，通过数据注释向`Models/ToDoTask.cs`类的属性添加一些显示和验证规则：

    ```cs
    ToDoTask.cs
    using System.ComponentModel;
    using System.ComponentModel.DataAnnotations;
    namespace ToDoListApp.Models;
    public class ToDoTask
    {
        public ToDoTask()
        {
            CreatedAt = DateTime.UtcNow;
            Id = Guid.NewGuid();
        }
        public ToDoTask(string title, ETaskStatus status) : this()
        {
    ```

```cs
The complete code can be found here: https://packt.link/yau4p.
```

在这里，对属性的`Required`数据注释是为了确保这个属性被设置为一个有效的值。在这个练习中，你添加了使用 Entity Framework Core 和 SQLite 的持久性，并创建了一个新页面，用于为待办应用创建任务，最后将其保存到数据库中。

1.  现在，在 VS Code 中运行代码。

1.  要在网页浏览器上查看输出，请在地址栏中输入以下命令：

    ```cs
    Localhost:####
    ```

这里`####`代表端口号。这会因不同的系统而异。

按下回车后，将显示以下屏幕：

![图 7.5：带有创建任务按钮的导航栏的主页](img/B16835_07_05.jpg)

图 7.5：包含创建任务按钮的导航栏主页

1.  点击`创建任务`按钮，你将看到你刚刚创建的页面，用于将新卡片插入到你的看板（Kanban Board）中：

![图 7.6：创建任务页面](img/B16835_07_06.jpg)

图 7.6：创建任务页面

注意

你可以在[`packt.link/3FPaG`](https://packt.link/3FPaG)找到这个练习所使用的代码。

现在，你将深入了解模型绑定如何将所有这些整合在一起，使你能够在客户端和服务器之间传输数据。你还将了解下一节中关于验证的更多信息。

## 验证

在开发应用程序时，验证数据是经常需要做的事情。验证一个字段可能意味着它是必填字段，或者它应该遵循特定的格式。你可能已经注意到，在上一个练习的最后部分，你在最后一个练习的最后一步中，在模型属性上放置了一些`[Required]`属性。这些属性被称为数据注释，用于创建服务器端验证。此外，你还可以添加一些客户端验证，与这种技术相结合。

注意，在*练习 7.04 的第 10 步*中，前端有一些带有`asp-validation-for`属性的`span`标签辅助器，这些属性指向模型属性。将这一切联系在一起的是`_ValidationScriptsPartial.cshtml`部分页面的包含。部分页面是下一节讨论的主题，但就现在而言，只需知道它们是可以被其他页面重用的页面。刚刚提到的那个包括页面的默认验证。

将这三个放在一起（即，所需的注释、`asp-validation-for`标签辅助器和`ValidationScriptsPartial`页面），客户端将创建验证逻辑，防止表单提交无效值。如果你想将验证放在服务器上，可以使用内置的`TryValidateModel`方法，传递要验证的模型。

## 部分页面的动态行为

到目前为止，你已经创建了一个用于显示任务和创建及编辑任务的方式的看板。然而，对于一个待办事项应用来说，还有一个主要功能需要添加——一种在看板之间移动任务的方式。你可以从最简单的方式开始，即从待办事项到进行中，再从进行中到完成。

到目前为止，你的任务卡片都是使用标签辅助器构建的。然而，标签辅助器被渲染为`静态`组件，不允许在渲染过程中添加任何动态行为。你可以直接将标签辅助器添加到你的页面上，但你必须为每个看板列表重复它。这正是 Razor Pages 的一个主要功能发挥作用的地方，那就是部分页面。它们允许你以更小的片段创建可重用的页面代码片段。这样，你可以共享基础页面的动态工具，同时避免在应用程序中重复代码。

这就结束了本节的理沦部分。在接下来的部分中，你将通过练习将其付诸实践。

## 练习 7.05：将标签助手重构为具有自定义逻辑的部分页面

在这个练习中，你将创建一个部分页面来替换 `KanbanCardTagHelper`，并为你的任务卡片添加一些动态行为，例如根据自定义逻辑更改内容。你将看到部分页面如何帮助减少重复代码并使其更容易重用。执行以下步骤以完成此练习：

1.  在 `Pages/Tasks` 文件夹内，创建一个名为 `_TaskItem.cshtml` 的新文件，内容如下：

    ```cs
    _TaskItem.cshtml
    @model ToDoListApp.Models.ToDoTask
    <form method="post">
        <div class="card">
            <div class="card-body p-2">
                <div class="card-title">
                    @Model.Title
                </div>
                <a class="btn btn-primary btn-sm" href="/tasks/@Model.Id">View</a>
                @if (Model.Status == Models.ETaskStatus.ToDo)
                {
                    <button type="submit" class="btn btn-warning btn-sm" href="@Model.Id" asp-page-handler="StartTask" asp-route-id="@Model.Id">
                        Start 
                    </button>
    ```

```cs
The complete code can be found here: https://packt.link/aUOcj.
```

`_TaskItem.cshtml` 主要是包含看板板卡片 `.cshtml` 代码的部分页面。

1.  现在，将 `Index.cshtml.cs` 文件内的代码替换为以下代码，该代码可以从数据库中读取保存的任务，并将你创建的操作放置在部分页面上：

    ```cs
    Index.cshtml.cs
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.AspNetCore.Mvc.RazorPages;
    using ToDoListApp.Data;
    using ToDoListApp.Models;
    namespace ToDoListApp.Pages
    {
        public class IndexModel : PageModel
        {
            private readonly ToDoDbContext _context;
            public IndexModel(ToDoDbContext context)
    ```

```cs
The complete code can be found here: https://packt.link/Tqgup.
```

此代码为三个 HTTP 请求创建了处理程序方法——一个 GET 请求和两个 POST 请求。它还放置了在这些处理程序上要执行的逻辑。你将使用 GET 从数据库中读取值，并使用 POST 将它们保存回去。

1.  最后，使用以下代码更新 `Index.cshtml` 页面，用你的看板卡片的部分 Razor 页面替换标签助手的使用：

    ```cs
    Index.cshtml
    @page
    @using ToDoListApp.Models
    @model IndexModel
    @{
        ViewData["Title"] = "MyToDos";
    }
    <div class="text-center">

        @if(TempData["SuccessMessage"] != null)
        {
            <div class="alert alert-success" role="alert">
                @TempData["SuccessMessage"]
            </div>
    ```

```cs
The complete code can be found here: https://packt.link/9SRsY.
```

这样做，你会注意到有多少重复代码被消除了。

1.  现在运行以下命令来启动应用程序：

    ```cs
    dotnet run
    ```

1.  接下来点击创建任务按钮并填写表单。创建任务后，你将看到一个确认消息，如图 *7.7* 所示。

![图 7.7：创建任务后的主屏幕](img/B16835_07_07.jpg)

图 7.7：创建任务后的主屏幕

注意

如果你已经在之前的屏幕上创建了一些任务，你的系统上的屏幕显示可能会有所不同。

在这个练习中，你创建了一个几乎完全功能化的待办事项应用程序，你可以创建任务并将它们保存到数据库中，甚至记录你的请求以查看它们花费了多长时间。

注意

你可以在 [`packt.link/VVT4M`](https://packt.link/VVT4M) 找到用于此练习的代码。

现在，是时候通过一个活动来工作在增强功能上了。

## 活动 7.01：创建一个页面来编辑现有任务

现在是时候通过一个新功能和基本功能来增强之前的练习，即移动看板板上的任务。你必须使用本章中涵盖的概念（如模型绑定、标签助手、部分页面和依赖注入）来构建此应用程序。

要完成此活动，你需要添加一个编辑任务的页面。以下步骤将帮助你完成此活动：

1.  创建一个名为 `Edit.cshtml` 的新文件，其表单与 `Create.cshtml` 相同。

1.  将页面指令中的路由更改为接收 `"/tasks/{id}"`。

1.  创建一个后端代码文件，通过 `OnGet ID` 从 `DbContext` 架构中加载数据。如果 ID 没有返回任务，则将其重定向到 `Create` 页面。

1.  在 Post 表单中，从数据库中恢复任务，更新其值，发送成功消息，然后重定向到 Index 视图。

页面的输出如下所示：

![图 7.8：输出到活动的编辑任务页面](img/B16835_07_08.jpg)

图 7.8：输出到活动的编辑任务页面

注意

该活动的解决方案可在[`packt.link/qclbF`](https://packt.link/qclbF)找到。

通过到目前为止的示例和活动，你现在知道如何使用 Razor 开发页面。在下一节中，你将学习如何使用一个具有更小范围隔离和可重用逻辑的工具，即视图组件。

# 视图组件

到目前为止，你已经看到了两种创建可重用组件的方法，以提供更好的维护并减少代码量，那就是标签辅助器和部分页面。虽然标签辅助器主要生成静态 HTML 代码（因为它将自定义标签转换为具有一些内容的现有 HTML 标签），但`部分页面`是另一个 Razor 页面内的一个小型 Razor 页面，它共享页面数据绑定机制并可以执行一些操作，如表单提交。`部分页面`的唯一缺点是它的动态行为依赖于包含它的页面。

本节是关于另一个允许你创建可重用组件的工具，即视图组件。视图组件在某种程度上类似于部分页面，因为它们也允许你提供动态功能并在后端有逻辑。然而，它们更强大，因为它们是自包含的。这种自包含性允许它们独立于页面开发，并且可以单独进行全面测试。

创建视图组件有几个要求，如下所述：

+   自定义组件类必须从`Microsoft.AspNetCore.Mvc.ViewComponent`继承。

+   它必须在类名中具有`ViewComponent`后缀或用`[ViewComponent]`属性装饰。

+   此类必须实现`IViewComponentResult Invoke()`同步方法或`Task<IViewComponentResult> InvokeAsync()`异步方法（当你需要从内部调用异步方法时）。

+   两种先前方法的典型结果是带有视图组件模型的`View(model)`方法。在前端，默认视图文件名应按惯例称为`Default.cshtml`。

+   为了渲染视图，它必须位于`Pages/Components/{MY_COMPONENT_NAME}/Default.cshtml`或`/Views/Shared/Components/{MY_COMPONENT_NAME}/Default.cshtml`之一。

+   如果不在上述任何路径中，视图的位置必须显式地作为参数传递给`Invoke`或`InvokeAsync`方法返回的`View`方法。

这就结束了本节的理沦部分。在下一节中，你将通过练习将其付诸实践。

## 练习 7.06：创建用于显示任务统计信息的视图组件

在这个练习中，你将创建一个视图组件，允许你在应用程序的导航栏上显示有关延迟任务的统计信息。通过完成这个练习，你将学习视图组件的基本语法以及如何在 Razor 页面上放置它们。执行以下步骤来完成此操作：

1.  在 `ToDoListApp` 项目的根目录下，创建一个名为 `ViewComponents` 的新文件夹。

1.  在此文件夹内，创建一个名为 `StatsViewComponent` 的新类：

    ```cs
    namespace ToDoListApp.ViewComponents;
    public class StatsViewComponent
    {
    }
    ```

1.  再次，在 `ViewComponents` 文件夹内，创建一个名为 `StatsViewModel` 的新类，包含两个公共 `int` 属性，分别命名为 `Delayed` 和 `DueToday`：

    ```cs
    namespace ToDoListApp.ViewComponents;
    public class StatsViewModel
    {
        public int Delayed { get; set; }
        public int DueToday { get; set; }
    }
    ```

1.  编辑 `StatsViewComponent` 类，使其继承自 `Microsoft.AspNetCore.Mvc` 命名空间中包含的 `ViewComponent` 类：

    ```cs
    using Microsoft.AspNetCore.Mvc;
    public class StatsViewComponent : ViewComponent
    {
    }
    ```

1.  通过构造函数初始化一个 `private readonly` 字段来注入 `ToDoDbContext`：

    ```cs
    public class StatsViewComponent : ViewComponent
    {
        private readonly ToDoDbContext _context;
        public StatsViewComponent(ToDoDbContext context) => _context = context;
    }
    ```

放置正确的 `using` 命名空间。

1.  创建一个名为 `InvokeAsync` 的方法，具有以下签名和内容：

    ```cs
    StatsViewComponent.cs
    using ToDoListApp.Data;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.EntityFrameworkCore;
    using System.Linq;
    namespace ToDoListApp.ViewComponents;
    public class StatsViewComponent : ViewComponent
    {
        private readonly ToDoDbContext _context;
        public StatsViewComponent(ToDoDbContext context) => _context = context;
        public async Task<IViewComponentResult> InvokeAsync()
        {
            var delayedTasks = await _context.Tasks.Where(t =>
    ```

```cs
The complete code can be found here: https://packt.link/jl2Ue.
```

此方法将使用 `ToDoDbContext` 查询数据库并检索延迟任务，以及当天到期的任务。

1.  现在，在 `Pages` 文件夹下，创建一个名为 `Components` 的新文件夹。

1.  在其下创建一个名为 `Stats` 的文件夹。

1.  然后，在 `Stats` 文件夹内，创建一个名为 `default.cshtml` 的新文件，内容如下：

    ```cs
    @model ToDoListApp.ViewComponents.StatsViewModel
    <form class="form-inline my-2 my-lg-0">
        @{
             var delayedEmoji = Model.Delayed > 0 ? "" : "";
             var delayedClass = Model.Delayed > 0 ? "btn-warning" : "btn-success";
             var dueClass = Model.DueToday > 0 ? "btn-warning" : "btn-success";
         }
        <button type="button" class="btn @delayedClass my-2 my-sm-0">
            <span class="badge badge-light">@Model.Delayed</span> Delayed Tasks @delayedEmoji
        </button>
        &nbsp;
        <button type="button" class="btn @dueClass my-2 my-sm-0">
            <span class="badge badge-light">@Model.DueToday</span> Tasks Due Today 
        </button>
    </form>
    ```

`default.cshtml` 将包含视图组件类的视图部分。在这里，你基本上是基于指定的模型创建一个 `.cshtml` 文件。

1.  最后，在 `_Layout.cshtml`（位于 `Pages/Shared/` 下），通过在导航栏内添加 `<vc:stats></vc:stats>` 标签来调用 `ViewComponent`。用以下代码替换页面代码：

    ```cs
    _Layout.cshtml
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>@ViewData["Title"] - ToDoListApp</title>
        <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
        <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
        <link rel="stylesheet" href="~/ToDoListApp.styles.css" asp-append-version="true" />
    </head>
    <body>
        <header>
            <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
    ```

```cs
The complete code can be found here: https://packt.link/DNUBC.
```

1.  运行应用程序以查看如图 *7.8* 所示的导航栏：

![图 7.9：任务统计视图组件](img/B16835_07_09.jpg)

图 7.9：任务统计视图组件

在这个练习中，你创建了你的第一个视图组件，这是一个直接显示在导航栏上的任务统计信息。正如你可能已经注意到的，视图组件的一个高效之处在于它们与显示在页面上的页面无关。你将前端和后端都封装在组件内部，没有任何外部依赖。

注意

你可以在 [`packt.link/j9eLW`](https://packt.link/j9eLW) 找到本练习使用的代码。

本练习涵盖了视图组件，这些组件允许你在应用程序的导航栏上显示有关延迟任务的统计信息。有了这些知识，你现在将完成一个活动，在这个活动中，你将在视图组件中工作以显示日志历史。

## 活动 7.02：编写一个用于显示任务日志的视图组件

作为本章的最后一步，这个活动将基于现实世界应用中的常见任务——记录用户活动。在这种情况下，你将把用户对字段的每个更改写入数据库并在视图中显示。为此，你需要使用视图组件。

以下步骤将帮助你完成此活动：

1.  在 `Models` 文件夹下创建一个新的类，命名为 `ActivityLog`。该类应具有以下属性：`Guid Id`、`String EntityId`、`DateTime Timestamp`、`String Property`、`String OldValue` 和 `String NewValue`。

1.  在 `ToDoDbContext` 下为该模型创建一个新的 `DbSet<ActivityLog>` 属性。

1.  在你的 `DbContext` 下创建一个方法，使用 Entity Framework 的 `ChangeTracker` 中的 `EntityState.Modified` 为 `Entries` 的修改属性生成活动日志。

1.  在 `DbContext` 中重写 `SaveChangesAsync()` 方法，通过在调用 `base` 方法之前将生成的日志添加到 `DbSet`。

1.  创建一个新的 Entity Framework Core 迁移并更新数据库以支持此迁移。

1.  创建一个 `ViewComponent` 类，该类应加载在调用中传递的给定 `taskId` 的所有日志，并将它们返回给 `ViewComponent`。

1.  创建 `ViewComponent` 视图，该视图应接受一个 `ActivityLog` 集合作为模型，并在 Bootstrap 表中显示它们，如果有的话。如果没有记录日志，显示一个警告，说明没有可用的日志。

1.  将视图组件添加到 `Edit` 页面，传递 `taskId` 属性。

1.  运行应用程序，通过打开任务详情来检查最终输出。你将看到一个右侧的框，其中包含你的活动日志，或者如果没有记录活动日志，将显示一条没有日志的消息，对于该任务还没有记录活动日志。

![图 7.10：没有日志的活动日志显示](img/B16835_07_10.jpg)

图 7.10：没有日志的活动日志显示

在这个活动中，你能够创建一个具有完全新功能且与页面解耦的独立视图组件，使其能够一次处理一个功能。

注意

该活动的解决方案可在 [`packt.link/qclbF`](https://packt.link/qclbF) 找到。

# 摘要

在本章中，你学习了使用 C# 和 Razor Pages 构建现代 Web 应用程序的基础。你专注于章节开头的重要概念，例如中间件、日志记录、DI 和配置。接下来，你使用 Razor Pages 创建了 CRUD 模型，并使用了一些更高级的功能，如自定义标签助手、部分页面和视图组件，这些功能使你能够更轻松地创建可维护的应用程序功能。

最后，你看到了 ASP.NET 模型绑定的工作原理，以便在客户端和服务器之间实现双向数据绑定。到目前为止，你应该已经具备了使用 ASP.NET 和 Razor Pages 独立构建现代 Web 应用程序的有效基础。

在接下来的两章中，你将学习如何构建和与 APIs 通信。
