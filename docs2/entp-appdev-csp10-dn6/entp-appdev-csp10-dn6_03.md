# *第二章*: 介绍 .NET 6 核心和标准

.NET 是一个开发者平台，它提供了用于构建多种不同类型应用程序的库和工具，例如 Web、桌面、移动、游戏、**物联网**（**IoT**）和云应用程序。使用 .NET，我们可以开发针对许多操作系统的应用程序，包括 Windows、macOS、Linux、Android、iOS 等，并且它支持 x86、x64、ARM32 和 ARM64 等处理器架构。

.NET 还支持使用多种编程语言进行应用程序开发，例如 C#、Visual Basic 和 F#，使用流行的 **集成开发环境**（**IDE**）如 Visual Studio、Visual Studio Code 和 Visual Studio for Mac。

在 .NET 5 之后，.NET 6 现在是一个主要版本，包括 C# 10 和 F# 6，添加了许多语言特性，并包含了许多性能改进。

本章涵盖了以下主题：

+   介绍 .NET 6

+   理解 .NET 6 的核心组件

+   设置开发环境

+   理解 CLI

+   什么是 .NET 标准？

+   理解 .NET 6 的跨平台和云应用程序支持

本章将帮助我们了解包含在 .NET 中用于开发应用程序的一些核心组件、库和工具。

# 技术要求

需要一台 Windows、Linux 或 Mac 机器，并从 [`dotnet.microsoft.com/download/dotnet/6.0`](https://dotnet.microsoft.com/download/dotnet/6.0) 安装相应的 SDK。

# 介绍 .NET 6

2002 年，Microsoft 发布了第一个版本的 .NET Framework，这是一个用于开发 Web 和桌面应用程序的开发平台。.NET Framework 提供了许多服务，包括托管代码执行、通过基类库提供的大量 API、内存管理、公共类型系统、语言互操作性以及 ADO.NET、ASP.NET、WCF、WinForms 和 **Windows 表现框架**（**WPF**）等开发框架。最初，它作为一个单独的安装程序发布，但后来被集成并随 Windows 操作系统一起发货。.NET Framework 4.8 是 .NET Framework 的最新版本。

2014 年，Microsoft 宣布了一个名为 **.NET Core** 的开源、跨平台的 .NET 实现。.NET Core 是从头开始构建的，使其成为跨平台，目前可在 Linux、macOS 和 Windows 上使用。.NET Core 快速且模块化，提供侧边支持，这样我们就可以在同一台机器上运行不同版本的 .NET Core，而不会影响其他应用程序。

.NET 6 是一个开源、跨平台的 .NET 实现，您可以使用它构建可在 Windows、macOS 和 Linux 操作系统上运行的应用程序。使用 .NET 6，您可以构建 Microsoft 统一的平台，用于开发浏览器、云、桌面、物联网和移动应用程序，以便使用相同的 .NET 库并轻松共享代码。

要了解 .NET 6 中的新功能，请访问 [`docs.microsoft.com/en-us/dotnet/core/whats-new/dotnet-6`](https://docs.microsoft.com/en-us/dotnet/core/whats-new/dotnet-6)。

**注意**

.NET 6 是一个 **长期支持** (**LTS**) 版本；从一般可用日期起支持 3 年。建议迁移，尤其是将 .NET 5 应用迁移到 .NET 6。更多详情，请访问 [`dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core`](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core)。

接下来，让我们了解 .NET 的核心特性。

## 理解核心特性

以下是一些 .NET 的核心特性，我们将更深入地了解：

+   **开源**：.NET 是一个免费（包括商业用途在内无许可费用）且开源的开发者平台，为 Linux、macOS 和 Windows 提供了许多开发工具。其源代码由微软和 .NET 社区在 GitHub 上维护。您可以在 [`github.com/dotnet/core/blob/master/Documentation/core-repos.md`](https://github.com/dotnet/core/blob/master/Documentation/core-repos.md) 访问 .NET 仓库。

+   **跨平台**：.NET 应用程序可以在许多操作系统上运行，包括 Linux、macOS、Android、iOS、tvOS、watchOS 和 Windows。它们也可以在多种处理器架构上保持一致运行，例如 x86、x64、ARM32 和 ARM64。

使用 .NET，我们可以构建以下类型的应用程序：

![Table 2.1 – Application types]

![Table_2.1.jpg]

表 2.1 – 应用程序类型

+   **编程语言**：.NET 支持多种编程语言。用一种语言编写的代码在其他语言中也是可访问的。以下表格显示了支持的语言：

![Table 2.2 – Supported Languages]

![Table_2.2.jpg]

表 2.2 – 支持的语言

+   **IDE**：.NET 支持多个 IDE。让我们了解每一个：

a. **Visual Studio** 是一个功能丰富的 IDE，可在 Windows 平台上用于构建、调试和发布 .NET 应用程序。它有三个版本：社区版、专业版和企业版。Visual Studio 2022 社区版对学生、个人开发者和为开源项目做出贡献的组织是免费的。

b. **Visual Studio for Mac** 是免费的，并且适用于 macOS。它可以用来使用 .NET 开发跨平台的应用程序和游戏，适用于 iOS、Android 和网页。

c. **Visual Studio Code** 是一个免费、开源、轻量级但功能强大的代码编辑器，可在 Windows、macOS 和 Linux 上使用。它内置了对 JavaScript、TypeScript 和 Node.js 的支持，并且通过扩展，您可以添加对许多流行编程语言的支持。

d. **Codespaces** 是一个由 Visual Studio Code 驱动并由 GitHub 托管的云开发环境，用于开发 .NET 应用程序。

+   **部署模型**：.NET 支持两种部署模式：

a. **自包含**：当.NET 应用程序以自包含模式发布时，发布的工件包含.NET 运行时、库以及应用程序及其依赖项。自包含应用程序是特定平台的，目标机器不需要安装.NET 运行时。机器使用与应用程序一起提供的.NET 运行时来运行应用程序。

b. **框架依赖**：当.NET 应用程序以框架依赖模式发布时，发布的工件仅包含应用程序及其依赖项。必须在目标机器上安装.NET 运行时才能运行应用程序。

接下来，让我们了解.NET 提供的应用程序框架。

## 理解应用程序框架

.NET 通过提供许多应用程序框架简化了应用程序开发。每个应用程序框架都包含一组用于开发特定应用程序的库。让我们详细了解每个框架：

+   **ASP.NET Core**：这是一个开源且跨平台的应用程序开发框架，允许您构建现代、基于云、互联网连接的应用程序，例如 Web、IoT 和 API 应用程序。ASP.NET Core 建立在.NET Core 之上，因此您可以在 Linux、macOS 和 Windows 等平台上构建和运行。

+   **Blazor**：这是一个应用程序框架，使用 C#而不是 JavaScript 构建交互式客户端 Web UI。Blazor 应用程序可以重用服务器端的代码和库，并在浏览器中使用 WebAssembly 运行，或者使用 SignalR 在服务器上处理客户端 UI 事件。

+   **WPF**：这是一个 UI 框架，允许您为 Windows 创建桌面应用程序。WPF 使用**可扩展应用程序标记语言**（**XAML**），这是一种用于应用程序开发的声明性模型。

+   **Entity Framework**（**EF**）Core：这是一个开源、跨平台、轻量级的**对象关系映射**（**ORM**）框架，用于使用.NET 对象与数据库交互。它支持 LINQ 查询、更改跟踪和模式迁移。它支持 SQL Server、SQL Azure、SQLite、Azure Cosmos DB、MySQL 等流行数据库。

+   **语言集成查询**（**LINQ**）：这为.NET 编程语言添加了查询功能。LINQ 允许您使用相同的 API 从数据库、XML、内存中的数组以及集合中查询数据。

+   **.NET MAUI**：.NET 多平台应用程序 UI 是一个跨平台框架，使用 C#和 XAML 创建原生移动和桌面应用程序。使用.NET MAUI，您可以使用相同的代码库开发针对 Android、iOS、macOS 和 Windows 的应用程序。

    注意

    .NET MAUI 目前处于预览阶段，不建议用于生产环境。有关更多信息，您可以参考[`docs.microsoft.com/en-us/dotnet/maui/what-is-maui`](https://docs.microsoft.com/en-us/dotnet/maui/what-is-maui)。

在下一节中，让我们了解.NET 的核心组件。

# 理解.NET 的核心组件

.NET 有两个主要组件：运行时和基础类库。运行时包括垃圾回收器（**GC**）和即时编译器（**JIT**），它管理 .NET 应用程序的执行以及基础类库（**BCLs**），也称为运行时库或框架库，它们包含 .NET 应用程序的基本构建块。

.NET SDK 可在 [`dotnet.microsoft.com/download/dotnet/6.0`](https://dotnet.microsoft.com/download/dotnet/6.0) 下载。它包含一组用于开发和运行 .NET 应用程序的库和工具。您可以选择安装 SDK 或 .NET 运行时。要开发 .NET 应用程序，您应该在开发机上安装 SDK，并安装 .NET 运行时来运行 .NET 应用程序。.NET 运行时包含在 .NET SDK 中，因此如果您已经安装了 .NET SDK，则无需单独安装 .NET 运行时。

![图 2.1 – .NET SDK 的可视化![图片 2.1](img/Figure_2.1_B18507.jpg)

图 2.1 – .NET SDK 的可视化

.NET SDK 包含以下组件：

+   **公共语言运行时（CLR**）：CLR 执行代码并管理内存分配。当 .NET 应用程序编译时，它产生一个中间语言（**IL**）。CLR 使用 JIT 编译器将编译后的代码转换为处理器能理解的机器代码。它是一个跨平台的运行时，可在 Windows、Linux 和 macOS 上使用。

+   **内存管理**：GC 管理 .NET 应用程序的内存分配和释放。对于每个新创建的对象，内存都在托管堆中分配，当没有足够的可用空间时，GC 会检查托管堆中的对象，并在它们不再在应用程序中使用时移除它们。有关更多信息，您可以参考 [`docs.microsoft.com/en-us/dotnet/standard/garbage-collection`](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection)。 

+   **JIT**：当 .NET 代码编译时，它被转换为 IL。IL 是平台和语言无关的，因此当运行时运行应用程序时，JIT 将 IL 转换为处理器能理解的机器代码。

+   **公共类型系统**：它定义了在 CLR 中如何定义、使用和管理类型。它使跨语言集成成为可能，并确保类型安全。

+   `System.String` 和 `System.Boolean`，如 `List<T>` 和 `Dictionary<Tkey, Tvalue>` 这样的集合，以及执行 I/O 操作、HTTP、序列化等实用函数。它简化了 .NET 应用程序的开发。

+   **Roslyn 编译器**：Roslyn 是一个开源的 C# 和 Visual Basic 编译器，具有丰富的代码分析 API。它允许使用与 Visual Studio 相同的 API 构建代码分析工具。

+   **MSBuild**：这是一个用于构建 .NET 应用程序的工具。Visual Studio 使用 MSBuild 构建 .NET 应用程序。

+   **NuGet**：这是一个开源的包管理工具，您可以使用它创建、发布和重用代码。NuGet 包包含编译后的代码、其依赖文件以及包含包版本号信息的清单。

在下一节中，让我们了解如何设置开发环境以创建和运行 .NET 应用程序。

# 设置开发环境

设置开发环境非常简单。您需要 .NET SDK 来构建和运行 .NET 应用程序。可选地，您可以选择安装支持 .NET 应用程序开发的 IDE。您需要执行以下步骤来在您的机器上设置 .NET SDK：

注意

.NET 6 由 Visual Studio 2022 和 Visual Studio 2022 for Mac 支持。它不支持在更早版本的 Visual Studio 上运行。Visual Studio Community Edition 对个人开发者、课堂学习和在组织内部署研究或开源项目的不限用户是免费的。它提供了与专业版相同的功能，但为了获得高级功能，如高级调试和诊断工具、测试工具等，您需要拥有企业版。要比较功能，您可以访问 [`visualstudio.microsoft.com/vs/compare`](https://visualstudio.microsoft.com/vs/compare)。

1.  在 Windows 机器上，从 [`visualstudio.microsoft.com`](https://visualstudio.microsoft.com) 下载并安装 Visual Studio 17.0 或更高版本。

1.  在安装选项中，从 **工作负载**，您可以选择 ASP.NET 和 Web 用于 Web/API 应用程序、Azure 开发、使用 .NET 进行 iOS、Android、Windows 的移动开发以及用于 Windows 应用程序的 .NET 桌面开发，如下面的截图所示：

![图 2.2 – Visual Studio 安装，工作负载选择![图片](img/Figure_2.2_B18507.jpg)

图 2.2 – Visual Studio 安装，工作负载选择

1.  确认选择并继续完成安装。这将安装 Visual Studio 和 .NET 6 SDK 到您的机器上。

    注意

    Azure 开发工作负载包括用于开发和支持针对 Azure 服务的应用程序的 SDK 和工具。它包括容器开发、Azure 资源管理器、Azure 云服务、Service Fabric、Azure 数据湖和流分析、快照调试器等工具。

或者，您也可以执行以下步骤来设置它：

1.  从 [`dotnet.microsoft.com/download/dotnet/6.0`](https://dotnet.microsoft.com/download/dotnet/6.0) 下载并安装 .NET 6 SDK for Windows、macOS 和 Linux。.NET Core 支持并行执行，因此我们可以在开发机上安装多个版本的 .NET Core SDK。

1.  从命令提示符运行 `dotnet --version` 命令以验证安装的版本，如下面的截图所示：

![图 2.3 – dotnet 命令的命令行输出![图片](img/Figure_2.3_B18507.jpg)

图 2.3 – dotnet 命令的命令行输出

1.  可选地，您可以从[`code.visualstudio.com`](https://code.visualstudio.com)下载并安装 Visual Studio Code，以使用它来开发.NET 应用程序。

现在我们已经了解了如何设置.NET 的开发环境，在下一节中，让我们了解.NET CLI 是什么以及它是如何帮助从命令行创建、构建和运行.NET 应用程序的。

要设置一个电子商务应用程序，您可以参考[`github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition/blob/main/README.md`](https://github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition/blob/main/README.md)。

# 理解 CLI

.NET CLI 是一个跨平台的命令行界面工具，可用于开发、构建、运行和发布.NET 应用程序。它包含在.NET SDK 中。

CLI 命令结构包含`命令驱动程序`（`dotnet`），`命令`，`命令-参数`和`选项`，这是大多数 CLI 操作的一个常见模式。请参考以下命令模式：

```cs
driver command <command-arguments> <options>
```

例如，以下命令创建一个新的控制台应用程序。`dotnet`是驱动程序，`new`是命令，`console`是一个作为参数的模板名称：

```cs
dotnet new console
```

以下表格说明了几个命令以及 CLI 支持的命令的简要描述：

![表 2.3 – CLI 命令](img/Table_2.3.jpg)

表 2.3 – CLI 命令

让我们创建一个简单的控制台应用程序，并使用.NET CLI 运行它：

注意

要执行以下步骤，作为先决条件，您应该在您的机器上安装.NET SDK。您可以从 https://dotnet.microsoft.com/download/dotnet/6.0 下载并安装它。

1.  在命令提示符中，运行以下命令以创建一个名为`HelloWorld`的项目控制台应用程序：

    ```cs
    dotnet new console --name HelloWorld
    ```

1.  此命令将基于`console`应用程序模板创建一个名为`HelloWorld`的新项目。请参考以下屏幕截图：

![图 2.4 – 新控制台应用程序的命令行输出](img/Figure_2.4_B18507.jpg)

图 2.4 – 新控制台应用程序的命令行输出

1.  运行以下命令来构建和运行应用程序：

    ```cs
    dotnet run --project ./HelloWorld/HelloWorld.csproj
    ```

1.  之前的命令将构建并运行应用程序，并将输出打印到命令窗口，如下所示：

![图 2.5 – 控制台应用程序运行时的命令行输出](img/Figure_2.5_B18507.jpg)

图 2.5 – 控制台应用程序运行时的命令行输出

在前面的步骤中，我们创建了一个新的控制台应用程序，并使用.NET CLI 运行了它。

## global.json 概述

在开发机器上，如果`global.json`文件中安装了多个.NET SDK，您可以定义用于运行.NET CLI 命令的.NET SDK 版本。通常，如果没有定义`global.json`文件，则使用 SDK 的最新版本，但您可以通过定义`global.json`来覆盖此行为。

运行以下命令将在当前目录中创建一个 `global.json` 文件。根据您的需求，您可以选择要配置的版本：

```cs
dotnet new globaljson --sdk-version 2.1.811
```

以下是一个通过运行前面的命令创建的示例 `global.json` 文件：

```cs
{
```

```cs
  "sdk": {
```

```cs
    "version": "2.1.811"
```

```cs
  }
```

```cs
}
```

在这里，`global.json` 已配置为使用 .NET SDK 版本 2.1.8.11。.NET CLI 使用此 SDK 版本来构建和运行应用程序。

更多关于 .NET CLI 的信息，您可以参考 [`docs.microsoft.com/en-us/dotnet/core/tools`](https://docs.microsoft.com/en-us/dotnet/core/tools)。

在下一节中，让我们了解什么是 .NET Standard。

# 什么是 .NET Standard？

.NET Standard 是一组适用于多个 .NET 实现的 API 规范。每个新的 .NET Standard 版本都会添加新的 API。每个 .NET 实现都针对特定的 .NET Standard 版本，并可以访问该 .NET Standard 版本支持的所有 API。

针对 .NET Standard 构建的库可以在使用支持这些 .NET Standard 版本的 .NET 实现构建的应用程序中使用。因此，在构建库时，针对更高版本的 .NET Standard 允许使用更多的 API，但只能用于使用支持它的 .NET 实现版本构建的应用程序。

以下截图列出了支持 .NET Standard 2.0 的各种 .NET 实现版本：

![图 2.6 – 支持 .NET Standard 2.0 的 .NET 实现](img/Figure_2.6_B18507.jpg)

图 2.6 – 支持 .NET Standard 2.0 的 .NET 实现

例如，如果你开发一个针对 .NET Standard 2.0 的库，它可以访问超过 32,000 个 API，但它支持的 .NET 实现版本较少。如果你想让你的库能够被最多的 .NET 实现访问，那么请选择最低可能的 .NET Standard 版本，但这样你可能需要在可用的 API 上做出妥协。

让我们了解何时使用 .NET 6 和 .NET Standard。

## 理解 .NET 6 和 .NET Standard 的使用

.NET Standard 使得在不同 .NET 实现之间共享代码变得容易，但 .NET 6 提供了一种更好的共享代码和运行在多个平台上的方法。.NET 6 统一了 API 以支持桌面、Web、云、移动和跨平台控制台应用程序。

.NET 6 实现了 .NET Standard 2.1，因此你的现有代码，如果针对 .NET Standard，可以在 .NET 6 上运行；除非你想访问新的运行时功能、语言功能或 API，否则你不需要更改**目标框架标记符**（**TFM**）。你可以多目标到 .NET Standard 和 .NET 6，这样你就可以访问新功能并让你的代码可供其他 .NET 实现使用。

如果你正在构建需要与 .NET Framework 一起工作的可重用库，那么请将它们针对 .NET Standard 2.0。如果你不需要支持 .NET Framework，那么你可以针对 .NET Standard 2.1 或 .NET 6。建议针对 .NET 6 以获取对新的 API、运行时和语言特性的访问。

使用 .NET CLI，运行以下命令创建一个新的类库：

```cs
dotnet new classlib -o MyLibrary
```

它创建了一个以 `.net6.0` 或开发机上可用的最新 SDK 为目标框架的类库项目。

如果你检查 `MyLibrary\MyLibrary.csproj` 文件，它应该看起来如下面的片段所示。你会注意到目标框架被设置为 `net6.0`：

```cs
<Project Sdk="Microsoft.NET.Sdk">
```

```cs
  <PropertyGroup>
```

```cs
    <TargetFramework>net6.0</TargetFramework>
```

```cs
     <ImplicitUsings>enable</ImplicitUsings>
```

```cs
     <Nullable>enable</Nullable>
```

```cs
  </PropertyGroup>
```

```cs
</Project>
```

你可以在使用 .NET CLI 创建类库时强制使用特定的目标框架版本。以下命令创建了一个针对 .NET Standard 2.0 的类库：

```cs
dotnet new classlib -o MyLibrary -f netstandard2.0
```

如果你检查 `MyLibrary\MyLibrary.csproj` 文件，它看起来如下面的片段所示，其中目标框架是 `netstandard2.0`：

```cs
<Project Sdk="Microsoft.NET.Sdk">
```

```cs
  <PropertyGroup>
```

```cs
    <TargetFramework>netstandard2.0</TargetFramework>
```

```cs
  </PropertyGroup>
```

```cs
</Project>
```

如果你创建了一个针对 .NET Standard 2.0 的库，它也可以在针对 .NET Core 以及 .NET Framework 构建的应用程序中访问。

可选地，你可以针对多个框架；例如，在以下代码片段中，库项目被配置为针对 .NET 6.0 和 .NET Standard 2.0：

```cs
<Project Sdk="Microsoft.NET.Sdk">
```

```cs
  <PropertyGroup>
```

```cs
    <TargetFrameworks>net6.0;netstandard2.0
```

```cs
      </TargetFrameworks>
```

```cs
  </PropertyGroup>
```

```cs
</Project>
```

当你配置你的应用程序以支持多个框架并构建项目时，你会注意到它为每个目标框架版本创建了工件。请参考以下截图：

![图 2.7 – 针对多个框架的构建工件

![img/Figure_2.7_B18507.jpg]

图 2.7 – 针对多个框架的构建工件

让我们在这里总结一下信息：

+   使用 .NET Standard 2.0 在 .NET Framework 和所有其他平台之间共享代码。

+   使用 .NET Standard 2.1 在 Mono、Xamarin 和 .NET Core 3.x 之间共享代码。

+   从现在开始使用 .NET 6 进行代码共享。

在下一节中，让我们了解 .NET 6 的跨平台能力和云应用程序支持。

# 理解 .NET 6 的跨平台和云应用程序支持

.NET 有许多实现。每个实现包含运行时、库、应用程序框架（可选）和开发工具。有四个 .NET 实现：

+   .NET Framework

+   .NET 6

+   **通用 Windows 平台** (**UWP**)

+   Mono

而所有这些实现共有的 API 规范集合是 .NET Standard。

多个 .NET 实现使你能够创建针对许多操作系统的 .NET 应用程序。你可以为以下操作系统构建 .NET 应用程序：

![表 2.4 – .NET 实现

![img/Table_2.4.jpg]

表 2.4 – .NET 实现

让我们更深入地了解 .NET 实现：

+   **.NET Framework** 是 .NET 的初始实现。使用 .NET Framework，您可以开发针对 Windows、WPF、Web 应用程序以及 Web 和 WCF 服务，目标操作系统为 Windows。.NET Framework 4.5 及以上版本实现了 .NET Standard，因此针对 .NET Standard 构建的库可以在 .NET Framework 应用程序中使用。

    注意事项

    .NET Framework 4.8 是 .NET Framework 的最后一个版本，未来将不再发布新版本。Microsoft 将继续将其包含在 Windows 中，并提供安全性和错误修复支持。对于新开发，建议使用 .NET 6 或更高版本。

+   **UWP** 是 .NET 的一个实现，您可以使用它来构建支持触控的 Windows 应用程序，这些应用程序可以在 PC、平板电脑、Xbox 等设备上运行。

+   **Mono** 是 .NET 的一个实现。它是一个小型运行时，为 Xamarin 提供动力，用于开发原生 Android、macOS、iOS、tvOS 和 watchOS 应用程序。它实现了 .NET Standard，因此针对 .NET Standard 构建的库可以在使用 Mono 构建的应用程序中使用。

+   **.NET 6** 是 .NET 的开源、跨平台实现，您可以使用它来构建控制台、Web、桌面和云应用程序，这些应用程序可以在 Windows、macOS 和 Linux 操作系统上运行。.NET 6 现在是 .NET 的主要实现，它基于单个代码库，具有统一的功能和 API 集合，这些可以由 .NET 应用程序使用。

.NET 6 SDK，包括库和工具，还包含多个运行时，包括 .NET 运行时、ASP.NET Core 运行时和 .NET 桌面运行时。要运行 .NET 6 应用程序，您可以选择安装 .NET 6 SDK 或相应的平台和特定工作负载的运行时：

+   **.NET 运行时** 仅包含运行控制台应用程序所需的组件。要运行 Web 或桌面应用程序，您需要单独安装 ASP.NET Core 运行时和 .NET 桌面运行时。.NET 运行时可在 Linux、macOS 和 Windows 上使用，并支持 x86、x64、ARM32 和 ARM64 处理器架构。

+   **ASP.NET Core 运行时** 允许您运行 Web/服务器应用程序，并且可在 Linux、macOS 和 Windows 上为 x86、x64、ARM32 和 ARM64 处理器架构提供支持。

+   **.NET 桌面运行时** 允许您在 Windows 上运行基于 Windows/WPF 的桌面应用程序。它适用于 x86 和 x64 处理器架构。

.NET 运行时在多个平台上的可用性使得 .NET 6 具有跨平台性。在目标机器上，您只需安装所需的工作负载的运行时，然后运行应用程序即可。

现在，让我们探索 Azure 提供的运行 .NET 6 的服务。

## 云支持

.NET 6 获得了包括 Azure、Google Cloud 和 AWS 在内的主流云服务提供商的支持。让我们了解一些可以在 Azure 中运行 .NET 6 应用程序的服务：

+   **Azure App Service** 支持轻松部署和运行 ASP.NET Core 6 应用程序。Azure App Service 提供了在 Linux 或 Windows 平台上使用 x86 或 x64 处理器架构托管 .NET 6 应用程序的机会。更多信息，请参阅 [`docs.microsoft.com/en-in/azure/app-service/overview`](https://docs.microsoft.com/en-in/azure/app-service/overview)。

+   **Azure Functions** 支持部署和运行基于 .NET 6 的无服务器函数。您可以在 Linux 或 Windows 上托管 Functions 应用程序。更多信息，请参阅 [`docs.microsoft.com/en-us/azure/azure-functions/functions-overview`](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview)。

+   **Docker** .NET 6 应用程序在 Docker 容器上运行。您可以在 Docker 容器上构建独立部署、高度可扩展的微服务。官方 .NET Core Docker 镜像可在 [`hub.docker.com/_/microsoft-dotnet`](https://hub.docker.com/_/microsoft-dotnet) 获取，支持不同组合的 .NET（SDK 或运行时）和操作系统。许多 Azure 服务支持 Docker 容器，包括 Azure App Service、Azure Service Fabric、Azure Batch、Azure Container Instances 和 **Azure Kubernetes Service**（**AKS**）。更多信息，请参阅 https://docs.microsoft.com/en-us/dotnet/core/docker/introduction。

使用 .NET 6，我们可以开发企业级服务器应用程序或高度可扩展的微服务，这些服务可以在云中运行。我们可以为 iOS、Android 和 Windows 操作系统开发移动应用程序。.NET 代码和项目文件看起来很相似，开发者可以重用技能或代码来开发针对不同平台的不同类型的应用程序。

# 摘要

在本章中，我们学习了 .NET 是什么以及其核心功能。我们了解了 .NET 提供的应用程序框架及其支持的不同的部署模型。接下来，我们学习了 .NET 提供的核心组件、工具和库，并学习了如何在机器上设置开发环境。

我们还研究了 .NET CLI，并使用 .NET CLI 创建了一个示例应用程序。接下来，我们学习了 .NET Standard 是什么以及何时使用 .NET 6 和 .NET Standard，然后通过讨论各种 .NET 实现、.NET 6 跨平台支持和云支持来结束本章。

在下一章中，我们将学习 C# 10.0 的新特性。

# 问题

1.  .NET Core 是以下哪个？

a. 开源

b. 跨平台

c. 免费

d. 所有以上选项

**答案：d**

1.  .NET Standard 2.0 库由以下哪个支持？

a. .NET Framework 4.6.1 或更高版本

b. .NET Core 2.0 或更高版本

c. .NET 6

d. Mono 5.4+ 或更高版本

e. 所有以上选项

**答案：e**

1.  运行 CLI 命令所必需的 .NET CLI 驱动程序是以下哪个？

a. `net`

b. `core`

c. `dotnet`

d. `none`

**答案：c**

1.  .NET SDK 包含以下哪些内容？

a. The .NET CLI

b. BCL

c. 运行时

d. 所有以上选项

**答案：d**

# 进一步阅读

要了解更多关于 .NET 6 的信息，您可以参考 https://docs.microsoft.com/en-us/dotnet/core/introduction。
