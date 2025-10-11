# *第 3 章*：自定义依赖注入

在本章的第三部分，我们将探讨 ASP.NET Core **依赖注入**（**DI**）以及如何根据需要自定义它以使用不同的 DI 容器。

在本章中，我们将涵盖以下主题：

+   使用不同的 DI 容器

+   探索 `ConfigureServices` 方法

+   使用不同的 `ServiceProvider`

+   介绍 Scrutor

本章中的主题涉及 ASP.NET Core 架构的宿主层：

![图 3.1 – ASP.NET Core 架构](img/Figure_2.1_B17996.jpg)

图 3.1 – ASP.NET Core 架构

# 技术要求

要遵循本章的描述，您需要创建一个 ASP.NET Core MVC 应用程序。打开您的控制台、shell 或 Bash 终端，切换到您的工作目录。使用以下命令创建一个新的 MVC 应用程序：

[PRE0]

现在，通过双击项目文件或在 Visual Studio Code 中在已打开的控制台中输入以下命令来在 Visual Studio 中打开项目：

[PRE1]

本章中的所有代码示例都可以在本书的 GitHub 仓库中找到：[https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/tree/main/Chapter03](https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/tree/main/Chapter03)。

# 使用不同的 DI 容器

在大多数项目中，您实际上并不需要使用不同的 DI 容器。ASP.NET Core 中现有的 DI 实现支持主要的基本功能，并且既有效又快速。然而，一些其他 DI 容器支持您可能在应用程序中想要使用的许多有趣功能：

+   使用 Ninject 创建一个支持模块作为轻量级依赖项的应用程序，例如，您可能希望将其放入特定目录并自动在应用程序中注册的模块。

+   在应用程序外部的配置文件中配置服务，在 XML 或 JSON 文件中而不是仅使用 C#。这是各种 DI 容器中的常见功能，但 ASP.NET Core 中尚未支持。

+   在运行时添加服务，可能是因为您不希望有一个不可变的 DI 容器。这也是一些 DI 容器中的常见功能。

现在我们来看看 `ConfigureServices` 方法如何使您能够使用替代的 DI 容器。

# 探索 ConfigureServices 方法

让我们比较当前的 `ConfigureServices` 方法与之前的长期支持版本，看看有什么变化。如果您使用版本 3.1 创建了新的 ASP.NET Core 项目并打开 `Startup.cs`，您将找到配置服务的函数，如下所示：

[PRE2]

相比之下，在 ASP.NET Core 6.0 中，已经不再有 `Startup.cs`，服务的配置是在 `Program.cs` 中完成的，如下所示：

[PRE3]

在这两种情况下，方法都会获取 `IServiceCollection`，它已经包含了一堆 ASP.NET Core 所需的服务。这个服务是由托管服务和在调用 `ConfigureServices` 方法之前执行的 ASP.NET Core 的部分添加的。

在方法内部，还会添加一些更多的服务。首先，添加一个包含 cookie 策略选项的配置类到 `ServiceCollection`。之后，`AddMvc()` 方法添加了 MVC 框架所需的另一堆服务。到目前为止，我们已经将大约 140 个服务注册到了 `IServiceCollection`。然而，服务集合并不是实际的 DI 容器。

实际的 DI 容器被所谓的 `IServiceCollection` 包装，并注册了一个扩展方法来从服务集合中创建 `IServiceProvider`，如下面的代码片段所示：

[PRE4]

`ServiceProvider` 包含一个不可变的容器，在运行时无法更改。使用默认的 `ConfigureServices` 方法，在调用此方法之后，会在后台创建 `IServiceProvider`。

接下来，我们将学习如何在 DI 自定义过程中应用替代的 `ServiceProvider`。

# 使用不同的 ServiceProvider

如果其他容器已经支持 ASP.NET Core，那么切换到不同的或自定义的 DI 容器相对容易。通常，其他容器会使用 `IServiceCollection` 来填充自己的容器。第三方 DI 容器通过遍历集合将已注册的服务移动到其他容器中：

1.  让我们首先使用 `Autofac` 作为第三方容器。在你的命令行中输入以下命令来加载 NuGet 包：

    [PRE5]

1.  要注册自定义的 IoC 容器，你需要注册不同的 `IServiceProviderFactory`。在这种情况下，如果你使用 `Autofac`，你将想要使用 `AutofacServiceProviderFactory`。`IServiceProviderFactory` 将创建一个 `ServiceProvider` 实例。如果第三方容器支持 ASP.NET Core，它应该提供一个。

    你应该将这个小的扩展方法放在 `Program.cs` 中，以将 `AutofacServiceProviderFactory` 注册到 `IHostBuilder`：

    [PRE6]

    不要忘记添加 `Autofac` 和 `Autofac.Extensions.DependencyInjection` 的 using 语句。

1.  要使用这个扩展方法，你可以在 `Program.cs` 中使用 `AutofacServiceProvider`：

    [PRE7]

这将 `AutofacServiceProviderFactory` 函数添加到 `IHostBuilder` 并启用 `Autofac` IoC 容器。如果你已经设置了它，那么在默认方式下向 `IServiceCollection` 添加服务时，你将使用 `Autofac`。

# 介绍 Scrutor

你并不总是需要替换现有的 .NET Core DI 容器来获取和使用一些酷炫的功能。在本章的开头，我提到了服务的自动注册，这可以通过其他 DI 容器来完成。这也可以通过一个名为 `IServiceCollection` 的不错的 NuGet 包来实现，该包可以自动将服务注册到 .NET Core DI 容器中。

注意

安德鲁·洛克（Andrew Lock）发布了一篇相当详细的关于 Scrutor 的博客文章。与其重复他的话，我建议您直接阅读那篇文章来了解更多信息：*使用 Scrutor 自动将服务注册到 ASP.NET Core DI 容器*，可在[https://andrewlock.net/using-scrutor-to-automatically-register-your-services-with-the-asp-net-core-di-container/](https://andrewlock.net/using-scrutor-to-automatically-register-your-services-with-the-asp-net-core-di-container/)找到。

# 摘要

使用本章中展示的方法，您将能够使用任何 .NET Standard 兼容的 DI 容器来替换现有的容器。如果您选择的容器不包含 `ServiceProvider`，请创建自己的实现 `IServiceProvider` 并在内部使用 DI 容器。如果您选择的容器不提供填充容器中注册的服务的方法，请创建自己的方法。遍历注册的服务并将它们添加到其他容器中。

实际上，最后一步听起来很简单，但可能是一项艰巨的任务，因为您需要将所有可能的 `IServiceCollection` 注册转换为其他容器的注册。这项任务的复杂性取决于其他 DI 容器的实现细节。

无论如何，您可以选择使用任何与 .NET Standard 兼容的 DI 容器。您可以在 ASP.NET Core 中更改很多默认实现。

这也是您可以在 Windows 上默认的 HTTPS 行为中完成的事情，我们将在下一章中了解更多。
