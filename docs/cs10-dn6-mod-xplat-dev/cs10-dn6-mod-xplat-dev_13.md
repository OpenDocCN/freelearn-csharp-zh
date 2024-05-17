# 第十三章：介绍 C#和.NET 的实际应用

本书的第三部分也是最后一部分是关于 C#和.NET 的实际应用。你将学习如何构建跨平台项目，如网站、服务以及移动和桌面应用。

微软将构建应用的平台称为**应用模型**或**工作负载**。

在*第一章*至*第十八章*和*第二十章*中，你可以使用特定操作系统的 Visual Studio 或跨平台的 Visual Studio Code 和 JetBrains Rider 来构建所有应用。在*第十九章*，*使用.NET MAUI 构建移动和桌面应用*中，尽管你可以使用 Visual Studio Code 来构建移动和桌面应用，但这并不容易。Windows 上的 Visual Studio 2022 对.NET MAUI 的支持比 Visual Studio Code 更好（目前）。

我建议你按顺序阅读本章及后续章节，因为后续章节会引用早期章节中的项目，并且你将积累足够的知识和技能来解决后续章节中更棘手的问题。

本章我们将涵盖以下主题：

+   理解 C#和.NET 的应用模型

+   ASP.NET Core 中的新特性

+   项目结构

+   使用其他项目模板

+   构建 Northwind 实体数据模型

# 理解 C#和.NET 的应用模型

由于本书是关于 C# 10 和.NET 6 的，我们将学习使用它们构建实际应用的应用模型，这些应用将在本书剩余章节中遇到。

**了解更多**：微软在其.NET 应用架构指南文档中提供了丰富的应用模型实施指导，你可以在以下链接阅读：[`www.microsoft.com/net/learn/architecture`](https://www.microsoft.com/net/learn/architecture)

## 使用 ASP.NET Core 构建网站

网站由多个网页组成，这些网页可以从文件系统静态加载或通过服务器端技术如 ASP.NET Core 动态生成。Web 浏览器使用**唯一资源定位符**（**URLs**）进行`GET`请求，这些 URL 标识每个页面，并可以使用`POST`、`PUT`和`DELETE`请求操作服务器上存储的数据。

在许多网站中，Web 浏览器被视为表示层，几乎所有的处理都在服务器端执行。客户端可能会使用一些 JavaScript 来实现某些表示层功能，如轮播。

ASP.NET Core 提供了多种构建网站的技术：

+   **ASP.NET Core Razor Pages**和**Razor 类库**是动态生成简单网站 HTML 的方法。你将在*第十四章*，*使用 ASP.NET Core Razor Pages 构建网站*中详细学习它们。

+   **ASP.NET Core MVC**是**模型-视图-控制器**（**MVC**）设计模式的一种实现，该模式在开发复杂网站时非常流行。你将在*第十五章*，*使用模型-视图-控制器模式构建网站*中详细学习它。

+   **Blazor** 允许你使用 C#和.NET 构建用户界面组件，而非基于 JavaScript 的 UI 框架如 Angular、React 和 Vue。**Blazor WebAssembly** 在浏览器中运行你的代码，如同 JavaScript 框架一样。**Blazor Server** 在服务器上运行你的代码，并动态更新网页。你将在*第十七章*，*使用 Blazor 构建用户界面*中详细了解 Blazor。Blazor 不仅适用于构建网站，还可用于创建混合移动和桌面应用。

### 使用内容管理系统构建网站

大多数网站内容繁多，如果每次内容变动都需要开发者介入，这将难以扩展。**内容管理系统**（**CMS**）使开发者能够定义内容结构和模板，以保持一致性和良好设计，同时让非技术内容所有者轻松管理实际内容。他们可以创建新页面或内容块，并更新现有内容，确保访客看到的内容美观且维护工作量最小。

针对所有网络平台，有多种 CMS 可供选择，如 PHP 的 WordPress 或 Python 的 Django CMS。支持现代.NET 的 CMS 包括 Optimizely 内容云、Piranha CMS 和 Orchard Core。

使用 CMS 的关键优势在于它提供了一个友好的内容管理用户界面。内容所有者登录网站并自行管理内容。内容随后通过 ASP.NET Core MVC 控制器和视图渲染并返回给访问者，或通过称为**无头 CMS**的网络服务端点，将内容提供给作为移动或桌面应用、店内触点或使用 JavaScript 框架或 Blazor 构建的客户端的“头部”。

本书不涉及.NET CMS，因此我在 GitHub 仓库中提供了链接，供你进一步了解：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#net-content-management-systems`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#net-content-management-systems)

### 使用 SPA 框架构建网页应用

网页应用，又称**单页应用**（**SPAs**），由单一网页构成，采用前端技术如 Blazor WebAssembly、Angular、React、Vue 或专有 JavaScript 库。这些应用在需要时向后台网络服务请求更多数据，并通过 XML 和 JSON 等通用序列化格式发布更新数据。典型例子包括谷歌的 Gmail、地图和文档等网页应用。

在网页应用中，客户端使用 JavaScript 框架或 Blazor WebAssembly 实现复杂的用户交互，但大部分重要处理和数据访问仍在服务器端进行，因为网络浏览器对本地系统资源的访问有限。

JavaScript 是弱类型语言，并非为复杂项目设计，因此当今大多数 JavaScript 库采用微软的 TypeScript，它为 JavaScript 添加了强类型特性，并设计了许多现代语言特性以应对复杂实现。

.NET SDK 提供了基于 JavaScript 和 TypeScript 的 SPA 项目模板，但本书不会花费时间学习如何构建基于 JavaScript 和 TypeScript 的 SPA，尽管这些通常与 ASP.NET Core 作为后端配合使用，因为本书专注于 C#，而非其他语言。

综上所述，C# 和 .NET 可用于服务器端和客户端构建网站，如 *图 13.1* 所示：

![](img/B17442_14_01.png)

*图 13.1*：C# 和 .NET 用于构建服务器端和客户端网站

## 构建 Web 及其他服务

尽管我们不会学习基于 JavaScript 和 TypeScript 的 SPA，但我们将学习如何使用 **ASP.NET Core Web API** 构建 Web 服务，然后从我们的 ASP.NET Core 网站的服务器端代码调用该 Web 服务，之后再从 Blazor WebAssembly 组件以及跨平台移动和桌面应用调用该 Web 服务。

虽然没有正式定义，但服务有时会根据其复杂性进行描述：

+   **服务**：客户端应用所需的所有功能集中在一个单一服务中。

+   **微服务**：专注于较小功能集的多个服务。

+   **Nanoservice**：作为服务提供的单一功能。与全天候运行的服务和微服务不同，纳米服务通常处于非活动状态，直到被调用以减少资源和成本。

除了使用 HTTP 作为底层通信技术的 Web 服务以及 API 设计原则外，我们还将学习如何使用其他技术和设计理念构建服务，包括：

+   **gRPC** 用于构建高效且性能卓越的服务，支持几乎所有平台。

+   **SignalR** 用于构建组件间的实时通信。

+   **OData** 用于将 Entity Framework Core 和其他数据模型通过 Web API 进行封装。

+   **GraphQL** 允许客户端控制跨多个数据源检索哪些数据。

+   **Azure Functions** 用于在云中托管无服务器纳米服务。

## 构建移动和桌面应用

移动平台主要有两大阵营：苹果的 iOS 和谷歌的 Android，各自拥有自己的编程语言和平台 API。桌面平台也有两大主流：苹果的 macOS 和微软的 Windows，同样各自拥有自己的编程语言和平台 API，如下表所示：

+   **iOS**：Objective C 或 Swift 以及 UIkit。

+   **Android**：Java 或 Kotlin 以及 Android API。

+   **macOS**：Objective C 或 Swift 以及 AppKit 或 Catalyst。

+   **Windows**：C、C++ 或多种其他语言，以及 Win32 API 或 Windows App SDK。

由于本书关注的是使用 C# 和 .NET 进行现代跨平台开发，因此不包含使用 **Windows Forms**、**Windows Presentation Foundation** (**WPF**) 或 **Universal Windows Platform** (**UWP**) 构建桌面应用的内容，因为它们仅限于 Windows 平台。

跨平台移动和桌面应用可以为 **.NET 多平台应用用户界面** (**MAUI**) 平台构建一次，然后就能在多种移动和桌面平台上运行。

.NET MAUI 使得通过共享用户界面组件和业务逻辑来开发这些应用变得简单。它们可以针对与控制台应用、网站和 Web 服务相同的 .NET API。应用将在移动设备上的 Mono 运行时和桌面设备上的 CoreCLR 运行时执行。与常规 .NET CoreCLR 运行时相比，Mono 运行时对移动设备的优化更好。Blazor WebAssembly 也使用 Mono 运行时，因为它像移动应用一样，资源受限。

这些应用可以独立存在，但通常会调用服务以提供跨所有计算设备的体验，从服务器和笔记本电脑到手机和游戏系统。

未来对 .NET MAUI 的更新将支持现有的 MVVM 和 XAML 模式，以及类似 **模型-视图-更新** (**MVU**) 的 C# 模式，这与 Apple 的 Swift UI 类似。

第六版的倒数第二章是 *第十九章*，*使用 .NET MAUI 构建跨平台移动和桌面应用*，涵盖了使用 .NET MAUI 构建跨平台移动和桌面应用的内容。

## .NET MAUI 的替代方案

在微软创建 .NET MAUI 之前，第三方已发起开源倡议，让 .NET 开发者能够使用 XAML 构建跨平台应用，名为 **Uno** 和 **Avalonia**。

### 理解 Uno Platform

正如 Uno 在其网站上所述，它是“首个也是唯一一个为 Windows、WebAssembly、iOS、macOS、Android 和 Linux 提供单一代码库应用的 UI 平台”。

开发者可以在原生移动、Web 和桌面平台上重用 99% 的业务逻辑和 UI 层。

Uno Platform 使用 Xamarin 原生平台而非 Xamarin.Forms。对于 WebAssembly，Uno 采用 Mono-WASM 运行时，与 Blazor WebAssembly 类似。在 Linux 上，Uno 利用 Skia 在画布上绘制用户界面。

### 理解 Avalonia

根据 .NET 基金会的网站所述，Avalonia “是一个跨平台的 XAML 基础 UI 框架，提供灵活的样式系统，并支持广泛的如 Windows、通过 Xorg 的 Linux、macOS 等操作系统。Avalonia 已准备好用于通用桌面应用开发。”

你可以将 Avalonia 视为 WPF 的精神继承者。熟悉 WPF、Silverlight 和 UWP 的开发者可以继续利用他们多年积累的知识和技能。

JetBrains 利用它来现代化其基于 WPF 的工具，并使其跨平台。

Avalonia 的 Visual Studio 扩展以及与 JetBrains Rider 的深度集成使得开发更加简便和高效。

# ASP.NET Core 的新特性

过去几年，微软迅速扩展了 ASP.NET Core 的能力。您应注意哪些.NET 平台得到支持，如下表所示：

+   ASP.NET Core 1.0 至 2.2 版本可在.NET Core 或.NET Framework 上运行。

+   ASP.NET Core 3.0 及更高版本仅在.NET Core 3.0 及更高版本上运行。

## ASP.NET Core 1.0

ASP.NET Core 1.0 于 2016 年 6 月发布，重点是实现一个适合构建现代跨平台 Web 应用和服务的最小 API，支持 Windows、macOS 和 Linux。

## ASP.NET Core 1.1

ASP.NET Core 1.1 于 2016 年 11 月发布，主要关注错误修复和功能及性能的常规改进。

## ASP.NET Core 2.0

ASP.NET Core 2.0 于 2017 年 8 月发布，重点增加了新特性，如 Razor Pages、将程序集捆绑到`Microsoft.AspNetCore.All`元包中、面向.NET Standard 2.0、提供新的认证模型以及性能改进。

ASP.NET Core 2.0 引入的最大新特性包括 ASP.NET Core Razor Pages，这在*第十四章*，*使用 ASP.NET Core Razor Pages 构建网站*中有所介绍，以及 ASP.NET Core OData 支持，这在*第十八章*，*构建和消费专业化服务*中有所介绍。

## ASP.NET Core 2.1

ASP.NET Core 2.1 于 2018 年 5 月发布，是一个**长期支持**(**LTS**)版本，意味着它将支持至 2021 年 8 月 21 日（LTS 标识直到 2018 年 8 月版本 2.1.3 才正式分配给它）。

ASP.NET Core 2.2 专注于增加新特性，如**SignalR**用于实时通信，**Razor 类库**用于重用 Web 组件，**ASP.NET Core Identity**用于认证，以及对 HTTPS 和欧盟**通用数据保护条例**(**GDPR**)的更好支持，包括下表中列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| Razor 类库 | 14 | 使用 Razor 类库 |
| GDPR 支持 | 15 | 创建和探索 ASP.NET Core MVC 网站 |
| 身份验证 UI 库和脚手架 | 15 | 探索 ASP.NET Core MVC 网站 |
| 集成测试 | 15 | 测试 ASP.NET Core MVC 网站 |
| `[ApiController]`, `ActionResult<T>` | 16 | 创建 ASP.NET Core Web API 项目 |
| 问题详情 | 16 | 实现 Web API 控制器 |
| `IHttpClientFactory` | 16 | 使用 HttpClientFactory 配置 HTTP 客户端 |
| ASP.NET Core SignalR | 18 | 使用 SignalR 实现实时通信 |

## ASP.NET Core 2.2

ASP.NET Core 2.2 于 2018 年 12 月发布，重点改进 RESTful HTTP API 的构建，更新项目模板至 Bootstrap 4 和 Angular 6，优化 Azure 托管配置，以及性能改进，包括下表中列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| Kestrel 中的 HTTP/2 | 14 | 经典 ASP.NET 与现代 ASP.NET Core 的对比 |
| 进程内托管模型 | 14 | 创建 ASP.NET Core 项目 |
| 端点路由 | 14 | 理解端点路由 |
| 健康检查 API | 16 | 实现健康检查 API |
| Open API 分析器 | 16 | 实现 Open API 分析器和约定 |

## ASP.NET Core 3.0

ASP.NET Core 3.0 于 2019 年 9 月发布，专注于充分利用.NET Core 3.0 和.NET Standard 2.1，这意味着它不支持.NET Framework，并增加了有用的改进，包括下表列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| Razor 类库中的静态资产 | 14 | 使用 Razor 类库 |
| MVC 服务注册的新选项 | 15 | 理解 ASP.NET Core MVC 启动 |
| ASP.NET Core gRPC | 18 | 使用 ASP.NET Core gRPC 构建服务 |
| Blazor Server | 17 | 使用 Blazor Server 构建组件 |

## ASP.NET Core 3.1

ASP.NET Core 3.1 于 2019 年 12 月发布，是一款 LTS 版本，意味着它将得到支持直至 2022 年 12 月 3 日。它专注于改进，如 Razor 组件的局部类支持和新的`<component>`标签助手。

## Blazor WebAssembly 3.2

Blazor WebAssembly 3.2 于 2020 年 5 月发布。它是一个当前版本，意味着项目必须在.NET 5 发布后的三个月内升级到.NET 5 版本，即 2021 年 2 月 10 日前。微软最终实现了使用.NET 进行全栈 Web 开发的承诺，并且 Blazor Server 和 Blazor WebAssembly 都在*第十七章*，*使用 Blazor 构建用户界面*中有所涉及。

## ASP.NET Core 5.0

ASP.NET Core 5.0 于 2020 年 11 月发布，重点在于修复错误、使用缓存提高证书认证性能、Kestrel 中 HTTP/2 响应头的 HPACK 动态压缩、ASP.NET Core 程序集的可空性注解，以及减少容器镜像大小，包括下表列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 允许匿名访问端点的扩展方法 | 16 | 保护 Web 服务 |
| `HttpRequest`和`HttpResponse`的 JSON 扩展方法 | 16 | 在控制器中获取 JSON 格式的客户信息 |

## ASP.NET Core 6.0

ASP.NET Core 6.0 于 2021 年 11 月发布，专注于提高生产力的改进，如最小化实现基本网站和服务的代码、.NET 热重载，以及新的 Blazor 托管选项，如使用.NET MAUI 的混合应用，包括下表列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 新的空 Web 项目模板 | 14 | 理解空 Web 模板 |
| HTTP 日志记录中间件 | 16 | 启用 HTTP 日志记录 |
| 最小 API | 16 | 实现最小 Web API |
| Blazor 错误边界 | 17 | 定义 Blazor 错误边界 |
| Blazor WebAssembly AOT | 17 | 启用 Blazor WebAssembly 预先编译 |
| .NET 热重载 | 17 | 使用.NET 热重载修复代码 |
| .NET MAUI Blazor 应用 | 19 | 在.NET MAUI 应用中托管 Blazor 组件 |

# | 构建仅限 Windows 的桌面应用 |

构建仅限 Windows 的桌面应用的技术包括：

+   **Windows Forms**, 2002.

+   **Windows Presentation Foundation** (**WPF**)，2006 年。

+   **Windows Store** 应用，2012 年。

+   **通用 Windows 平台** (**UWP**) 应用，2015 年。

+   **Windows App SDK**（曾用名 **WinUI 3** 和 **Project Reunion**）应用，2021 年。

## 理解传统 Windows 应用平台

随着 1985 年微软 Windows 1.0 的发布，创建 Windows 应用的唯一方式是使用 C 语言并调用名为 kernel、user 和 GDI 的三个核心 DLL 中的函数。当 Windows 95 成为 32 位系统后，这些 DLL 被附加了 32 后缀，并被称为 **Win32 API**。

1991 年，微软推出了 Visual Basic，为开发者提供了一种通过工具箱中的控件进行拖放操作的可视化方式来构建 Windows 应用程序的用户界面。它极受欢迎，Visual Basic 运行时至今仍是 Windows 10 的一部分。

随着 2002 年 C# 和 .NET Framework 的首个版本发布，微软提供了名为 **Windows Forms** 的技术来构建 Windows 桌面应用。当时，Web 开发的对应技术名为 **Web Forms**，因此名称相辅相成。代码可以用 Visual Basic 或 C# 语言编写。Windows Forms 拥有类似的拖放式可视化设计器，尽管它生成的是 C# 或 Visual Basic 代码来定义用户界面，这可能对人类来说难以直接理解和编辑。

2006 年，微软发布了一种更强大的技术，名为 **Windows Presentation Foundation** (**WPF**)，作为 .NET Framework 3.0 的关键组件，与 **Windows Communication Foundation** (**WCF**) 和 **Windows Workflow** (**WF**) 并列。

尽管可以通过仅编写 C# 语句来创建 WPF 应用，但它也可以使用 **可扩展应用程序标记语言** (**XAML**) 来指定用户界面，这既便于人类理解，也便于代码处理。Windows 版的 Visual Studio 部分基于 WPF 构建。

2012 年，微软发布了 Windows 8，其内置的 Windows Store 应用运行在一个受保护的沙箱环境中。

2015 年，微软发布了 Windows 10，并引入了名为 **通用 Windows 平台** (**UWP**) 的更新版 Windows Store 应用概念。UWP 应用可以使用 C++ 和 DirectX UI、JavaScript 和 HTML 或 C# 结合现代 .NET 的定制分支来构建，该分支虽非跨平台，但能完全访问底层的 WinRT API。

UWP 应用仅能在 Windows 10 平台上运行，不支持早期版本的 Windows，但可在 Xbox 及配备运动控制器的 Windows Mixed Reality 头显上运行。

许多 Windows 开发者因 UWP 应用对底层系统的访问受限而拒绝使用 Windows Store。微软最近推出了 **Project Reunion** 和 **WinUI 3**，两者协同工作，使 Windows 开发者能够将现代 Windows 开发的某些优势引入现有的 WPF 应用，并使其享有与 UWP 应用相同的益处和系统集成。这一举措现称为 **Windows App SDK**。

## 理解现代.NET 对遗留 Windows 平台的支持

.NET SDK 在 Linux 和 macOS 上的磁盘占用约为 330 MB。.NET SDK 在 Windows 上的磁盘占用约为 440 MB。这是因为它包含了 Windows 桌面运行时，该运行时允许遗留的 Windows 应用程序平台 Windows Forms 和 WPF 在现代.NET 上运行。

许多使用 Windows Forms 和 WPF 构建的企业应用程序需要维护或增强新功能，但直到最近它们还停留在.NET Framework 上，这是一个遗留平台。借助现代.NET 及其 Windows 桌面包，这些应用程序现在可以充分利用.NET 的现代功能。

# 项目结构

你应该如何组织你的项目？到目前为止，我们构建了小型独立的控制台应用程序来演示语言或库功能。在本书的其余部分，我们将使用不同的技术构建多个项目，这些技术协同工作以提供单一解决方案。

在大型复杂的解决方案中，要在所有代码中导航可能会很困难。因此，组织项目的主要原因是使其更容易找到组件。为解决方案或工作区提供一个反映应用程序或解决方案的整体名称是很好的。

我们将为一家名为**Northwind**的虚构公司构建多个项目。我们将把解决方案或工作区命名为`PracticalApps`，并使用`Northwind`作为所有项目名称的前缀。

有许多方法来组织和命名项目和解决方案，例如使用文件夹层次结构以及命名约定。如果你在一个团队中工作，请确保你知道你的团队是如何做的。

## 在解决方案或工作区中组织项目

在解决方案或工作区中为项目制定命名约定是很好的做法，这样任何开发人员都能立即了解每个项目的作用。常见的选择是使用项目类型，例如类库、控制台应用程序、网站等，如下表所示：

| 名称 | 描述 |
| --- | --- |
| `Northwind.Common` | 一个类库项目，用于跨多个项目使用的通用类型，如接口、枚举、类、记录和结构。 |
| `Northwind.Common.EntityModels` | 一个类库项目，用于通用的 EF Core 实体模型。实体模型通常在服务器和客户端侧都被使用，因此最好将特定数据库提供程序的依赖关系分开。 |
| `Northwind.Common.DataContext` | 一个类库项目，用于依赖特定数据库提供程序的 EF Core 数据库上下文。 |
| `Northwind.Web` | 一个 ASP.NET Core 项目，用于一个简单的网站，该网站混合使用静态 HTML 文件和动态 Razor 页面。 |
| `Northwind.Razor.Component` | 一个类库项目，用于在多个项目中使用的 Razor 页面。 |
| `Northwind.Mvc` | 一个 ASP.NET Core 项目，用于使用 MVC 模式的复杂网站，可以更容易地进行单元测试。 |
| `Northwind.WebApi` | 一个用于 HTTP API 服务的 ASP.NET Core 项目。与网站集成的好选择，因为它们可以使用任何 JavaScript 库或 Blazor 与服务交互。 |
| `Northwind.OData` | 一个实现 OData 标准以允许客户端控制查询的 HTTP API 服务的 ASP.NET Core 项目。 |
| `Northwind.GraphQL` | 一个实现 GraphQL 标准以允许客户端控制查询的 HTTP API 服务的 ASP.NET Core 项目。 |
| `Northwind.gRPC` | 一个用于 gRPC 服务的 ASP.NET Core 项目。与使用任何语言和平台构建的应用程序集成的好选择，因为 gRPC 支持广泛且高效且性能优越。 |
| `Northwind.SignalR` | 一个用于实时通信的 ASP.NET Core 项目。 |
| `Northwind.AzureFuncs` | 一个用于在 Azure Functions 中托管的无服务器纳米服务的 ASP.NET Core 项目。 |
| `Northwind.BlazorServer` | 一个 ASP.NET Core Blazor 服务器项目。 |
| `Northwind.BlazorWasm.Client` | 一个 ASP.NET Core Blazor WebAssembly 客户端项目。 |
| `Northwind.BlazorWasm.Server` | 一个 ASP.NET Core Blazor WebAssembly 服务器端项目。 |
| `Northwind.Maui` | 一个用于跨平台桌面/移动应用的.NET MAUI 项目。 |
| `Northwind.MauiBlazor` | 一个用于托管 Blazor 组件并具有与操作系统原生集成的.NET MAUI 项目。 |

# 使用其他项目模板

安装.NET SDK 时，包含许多项目模板：

1.  在命令提示符或终端中，输入以下命令：

    ```cs
    dotnet new --list 
    ```

1.  您将看到当前安装的模板列表，如果您在 Windows 上运行，还包括 Windows 桌面开发模板，如图*13.2*所示:![](img/B17442_14_02.png)

    图 13.2：dotnet 项目模板列表

1.  注意与 Web 相关的项目模板，包括使用 Blazor、Angular 和 React 创建 SPA 的模板。但另一个常见的 JavaScript SPA 库缺失了：Vue。

## 安装额外的模板包

开发者可以安装许多额外的模板包：

1.  启动浏览器并导航至[`dotnetnew.azurewebsites.net/`](http://dotnetnew.azurewebsites.net/)。

1.  在文本框中输入`vue`，并注意 Vue.js 可用模板列表，其中包括微软发布的一个模板，如图*13.3*所示:![](img/B17442_14_03.png)

    图 13.3：微软提供的 Vue.js 项目模板

1.  点击**微软提供的 ASP.NET Core with Vue.js**，并注意安装和使用此模板的说明，如下列命令所示：

    ```cs
    dotnet new --install "Microsoft.AspNetCore.SpaTemplates"
    dotnet new vue 
    ```

1.  点击**查看此包中的其他模板**，并注意除了 Vue.js 的项目模板外，它还有 Aurelia 和 Knockout.js 的项目模板。

# 为 Northwind 数据库构建实体数据模型

实际应用通常需要与关系数据库或其他数据存储进行交互。在本章中，我们将为存储在 SQL Server 或 SQLite 中的 Northwind 数据库定义实体数据模型。它将被用于我们后续章节创建的大多数应用中。

`Northwind4SQLServer.sql`和`Northwind4SQLite.sql`脚本文件有所不同。SQL Server 脚本创建了 13 个表以及相关的视图和存储过程。SQLite 脚本是一个简化版本，仅创建 10 个表，因为 SQLite 不支持那么多特性。本书的主要项目仅需要这 10 个表，因此你可以使用任一数据库完成本书中的所有任务。

安装 SQL Server 和 SQLite 的指南可在*第十章*，*使用 Entity Framework Core 处理数据*中找到。该章节还包含了安装`dotnet-ef`工具的说明，该工具用于从现有数据库生成实体模型。

**最佳实践**：应为实体数据模型创建单独的类库项目。这使得在后端 Web 服务器和前端桌面、移动及 Blazor WebAssembly 客户端之间共享数据更为便捷。

## 为实体模型创建使用 SQLite 的类库

现在，你将在类库中定义实体数据模型，以便它们能在包括客户端应用模型在内的其他类型项目中复用。如果你未使用 SQL Server，则需为 SQLite 创建此类库。若使用 SQL Server，则可同时为 SQLite 和 SQL Server 创建类库，并根据需要切换使用。

我们将使用 EF Core 命令行工具自动生成一些实体模型：

1.  使用你偏好的代码编辑器创建一个名为`PracticalApps`的新解决方案/工作区。

1.  添加一个类库项目，如下列表所述：

    1.  项目模板：**类库** / `classlib`

    1.  工作区/解决方案文件和文件夹：`PracticalApps`

    1.  项目文件和文件夹：`Northwind.Common.EntityModels.Sqlite`

1.  在`Northwind.Common.EntityModels.Sqlite`项目中，添加 SQLite 数据库提供程序和 EF Core 设计时支持的包引用，如下所示：

    ```cs
    <ItemGroup>
      <PackageReference
        Include="Microsoft.EntityFrameworkCore.Sqlite" 
        Version="6.0.0" />
      <PackageReference 
        Include="Microsoft.EntityFrameworkCore.Design" 
        Version="6.0.0">
        <PrivateAssets>all</PrivateAssets>
        <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
      </PackageReference>  
    </ItemGroup> 
    ```

1.  删除`Class1.cs`文件。

1.  构建项目。

1.  通过将`Northwind4SQLite.sql`文件复制到`PracticalApps`文件夹，为 SQLite 创建`Northwind.db`文件，然后在命令提示符或终端中输入以下命令：

    ```cs
    sqlite3 Northwind.db -init Northwind4SQLite.sql 
    ```

1.  请耐心等待，因为此命令可能需要一段时间来创建数据库结构，如下所示：

    ```cs
    -- Loading resources from Northwind4SQLite.sql 
    SQLite version 3.35.5 2021-04-19 14:49:49
    Enter ".help" for usage hints.
    sqlite> 
    ```

1.  在 Windows 上按 Ctrl + C 或在 macOS 上按 Cmd + D 以退出 SQLite 命令模式。

1.  打开命令提示符或终端，定位到`Northwind.Common.EntityModels.Sqlite`文件夹。

1.  在命令行中，为所有表生成实体类模型，如下所示：

    ```cs
    dotnet ef dbcontext scaffold "Filename=../Northwind.db" Microsoft.EntityFrameworkCore.Sqlite --namespace Packt.Shared --data-annotations 
    ```

    注意以下事项：

    +   执行的命令：`dbcontext scaffold`

    +   连接字符串：`"Filename=../Northwind.db"`

    +   数据库提供程序：`Microsoft.EntityFrameworkCore.Sqlite`

    +   命名空间：`--namespace Packt.Shared`

    +   同时使用数据注解和 Fluent API：`--data-annotations`

1.  注意构建消息和警告，如下面的输出所示：

    ```cs
    Build started...
    Build succeeded.
    To protect potentially sensitive information in your connection string, you should move it out of source code. You can avoid scaffolding the connection string by using the Name= syntax to read it from configuration - see https://go.microsoft.com/fwlink/?linkid=2131148\. For more guidance on storing connection strings, see http://go.microsoft.com/fwlink/?LinkId=723263. 
    ```

### 改进类到表的映射

`dotnet-ef`命令行工具为 SQL Server 和 SQLite 生成不同的代码，因为它们支持不同级别的功能。

例如，SQL Server 文本列可以有字符数量的限制。SQLite 不支持这一点。因此，`dotnet-ef`将生成验证属性，以确保`string`属性在 SQL Server 上限制为指定数量的字符，而在 SQLite 上则不限制，如下面的代码所示：

```cs
// SQLite database provider-generated code
[Column(TypeName = "nvarchar (15)")] 
public string CategoryName { get; set; } = null!;
// SQL Server database provider-generated code 
[StringLength(15)]
public string CategoryName { get; set; } = null!; 
```

两种数据库提供程序都不会将非可空`string`属性标记为必填：

```cs
// no runtime validation of non-nullable property
public string CategoryName { get; set; } = null!;
// nullable property
public string? Description { get; set; }
// decorate with attribute to perform runtime validation
[Required]
public string CategoryName { get; set; } = null!; 
```

我们将对 SQLite 的实体模型映射和验证规则进行一些小改进：

1.  打开`Customer.cs`文件，并添加一个正则表达式以验证其主键值，仅允许使用大写西文字符，如下面的代码中突出显示所示：

    ```cs
    [Key]
    [Column(TypeName = "nchar (5)")]
    **[****RegularExpression(****"[A-Z]{5}"****)****]**
    public string CustomerId { get; set; } 
    ```

1.  激活代码编辑器的查找和替换功能（在 Visual Studio 2022 中，导航至**编辑** | **查找和替换** | **快速替换**），切换**使用正则表达式**，然后在搜索框中输入一个正则表达式，如下面的表达式所示：

    ```cs
    \[Column\(TypeName = "(nchar|nvarchar) \((.*)\)"\)\] 
    ```

1.  在替换框中，输入一个替换用的正则表达式，如下面的表达式所示：

    ```cs
    $&\n    [StringLength($2)] 
    ```

    在新行字符`\n`之后，我添加了四个空格字符以在我的系统上正确缩进，该系统每级缩进使用两个空格字符。您可以根据需要插入任意数量的空格。

1.  设置查找和替换以搜索当前项目中的文件。

1.  执行搜索和替换以替换所有内容，如图*13.4*所示：![](img/B17442_14_04.png)

    图 13.4：在 Visual Studio 2022 中使用正则表达式搜索并替换所有匹配项

1.  将任何日期/时间属性，例如在`Employee.cs`中，改为使用可空`DateTime`而非字节数组，如下面的代码所示：

    ```cs
    // before
    [Column(TypeName = "datetime")] 
    public byte[] BirthDate { get; set; }
    // after
    [Column(TypeName = "datetime")]
    public DateTime? BirthDate { get; set; } 
    ```

    使用代码编辑器的查找功能搜索`"datetime"`以找到所有需要更改的属性。

1.  将任何`money`属性，例如在`Order.cs`中，改为使用可空`decimal`而非字节数组，如下面的代码所示：

    ```cs
    // before
    [Column(TypeName =  "money")] 
    public byte[] Freight { get; set; }
    // after
    [Column(TypeName = "money")]
    public decimal? Freight { get; set; } 
    ```

    使用代码编辑器的查找功能搜索`"money"`以找到所有需要更改的属性。

1.  将任何`bit`属性，例如在`Product.cs`中，改为使用`bool`而非字节数组，如下面的代码所示：

    ```cs
    // before
    [Column(TypeName = "bit")]
    public byte[] Discontinued { get; set; } = null!;
    // after
    [Column(TypeName = "bit")]
    public bool Discontinued { get; set; } 
    ```

    使用代码编辑器的查找功能搜索`"bit"`以找到所有需要更改的属性。

1.  在`Category.cs`中，将`CategoryId`属性设为`int`类型，如以下代码中突出显示所示：

    ```cs
    [Key]
    public **int** CategoryId { get; set; } 
    ```

1.  在`Category.cs`中，将`CategoryName`属性设为必填项，如以下代码中突出显示所示：

    ```cs
    **[****Required****]**
    [Column(TypeName = "nvarchar (15)")]
    [StringLength(15)]
    public string CategoryName { get; set; } 
    ```

1.  在`Customer.cs`中，将`CompanyName`属性设为必填项，如以下代码中突出显示所示：

    ```cs
    **[****Required****]**
    [Column(TypeName = "nvarchar (40)")]
    [StringLength(40)]
    public string CompanyName { get; set; } 
    ```

1.  在`Employee.cs`中，将`EmployeeId`属性改为`int`而非`long`。

1.  在`Employee.cs`中，使`FirstName`和`LastName`属性成为必填项。

1.  在`Employee.cs`中，将`ReportsTo`属性改为`int?`而非`long?`。

1.  在`EmployeeTerritory.cs`中，将`EmployeeId`属性改为`int`而非`long`。

1.  在`EmployeeTerritory.cs`中，使`TerritoryId`属性成为必填项。

1.  在`Order.cs`中，将`OrderId`属性改为`int`而非`long`。

1.  在`Order.cs`中，用正则表达式装饰`CustomerId`属性，以强制五个大写字符。

1.  在`Order.cs`中，将`EmployeeId`属性改为`int?`而非`long?`。

1.  在`Order.cs`中，将`ShipVia`属性改为`int?`而非`long?`。

1.  在`OrderDetail.cs`中，将`OrderId`属性改为`int`而非`long`。

1.  在`OrderDetail.cs`中，将`ProductId`属性改为`int`而非`long`。

1.  在`OrderDetail.cs`中，将`Quantity`属性改为`short`而非`long`。

1.  在`Product.cs`中，将`ProductId`属性改为`int`而非`long`。

1.  在`Product.cs`中，使`ProductName`属性成为必填项。

1.  在`Product.cs`中，将`SupplierId`和`CategoryId`属性改为`int?`而非`long?`。

1.  在`Product.cs`中，将`UnitsInStock`、`UnitsOnOrder`和`ReorderLevel`属性改为`short?`而非`long?`。

1.  在`Shipper.cs`中，将`ShipperId`属性改为`int`而非`long`。

1.  在`Shipper.cs`中，使`CompanyName`属性成为必填项。

1.  在`Supplier.cs`中，将`SupplierId`属性改为`int`而非`long`。

1.  在`Supplier.cs`中，使`CompanyName`属性成为必填项。

1.  在`Territory.cs`中，将`RegionId`属性改为`int`而非`long`。

1.  在`Territory.cs`中，使`TerritoryId`和`TerritoryDescription`属性成为必填项。

既然我们有了实体类的类库，我们就可以为数据库上下文创建一个类库。

### 为 Northwind 数据库上下文创建一个类库

你现在将定义一个数据库上下文类库：

1.  向解决方案/工作区添加一个类库项目，如以下列表所定义：

    1.  项目模板：**类库** / `classlib`

    1.  工作区/解决方案文件和文件夹：`PracticalApps`

    1.  项目文件和文件夹：`Northwind.Common.DataContext.Sqlite`

1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择。

1.  在 Visual Studio Code 中，选择`Northwind.Common.DataContext.Sqlite`作为活动的 OmniSharp 项目。

1.  在`Northwind.Common.DataContext.Sqlite`项目中，添加对`Northwind.Common.EntityModels.Sqlite`项目的项目引用，并添加对 EF Core SQLite 数据提供程序的包引用，如下所示：

    ```cs
    <ItemGroup>
      <PackageReference 
        Include="Microsoft.EntityFrameworkCore.SQLite" 
        Version="6.0.0" />
    </ItemGroup>
    <ItemGroup>
      <ProjectReference Include=
        "..\Northwind.Common.EntityModels.Sqlite\Northwind.Common
    .EntityModels.Sqlite.csproj" />
    </ItemGroup> 
    ```

    项目引用的路径在你的项目文件中不应有换行。

1.  在`Northwind.Common.DataContext.Sqlite`项目中，删除`Class1.cs`类文件。

1.  构建`Northwind.Common.DataContext.Sqlite`项目。

1.  将 `NorthwindContext.cs` 文件从 `Northwind.Common.EntityModels.Sqlite` 项目/文件夹移动到 `Northwind.Common.DataContext.Sqlite` 项目/文件夹。

    在 Visual Studio **解决方案资源管理器**中，如果你在项目间拖放文件，它将被复制。如果你在拖放时按住 Shift 键，文件将被移动。在 Visual Studio Code **资源管理器**中，如果你在项目间拖放文件，它将被移动。如果你在拖放时按住 Ctrl 键，文件将被复制。

1.  在 `NorthwindContext.cs` 中，在 `OnConfiguring` 方法中，移除关于连接字符串的编译器 `#warning`。

    **良好实践**：我们将在需要与 Northwind 数据库交互的任何项目（如网站）中覆盖默认数据库连接字符串，因此从 `DbContext` 派生的类必须有一个带有 `DbContextOptions` 参数的构造函数才能实现这一点，如下列代码所示：

    ```cs
    public NorthwindContext(DbContextOptions<NorthwindContext> options)
      : base(options)
    {
    } 
    ```

1.  在 `OnModelCreating` 方法中，移除所有调用 `ValueGeneratedNever` 方法的 Fluent API 语句，以配置主键属性如 `SupplierId` 永不自动生成值，或调用 `HasDefaultValueSql` 方法，如下列代码所示：

    ```cs
    modelBuilder.Entity<Supplier>(entity =>
    {
      entity.Property(e => e.SupplierId).ValueGeneratedNever();
    }); 
    ```

    如果我们不移除上述配置语句，那么当我们添加新供应商时，`SupplierId` 值将始终为 0，我们只能添加一个具有该值的供应商，之后所有其他尝试都会抛出异常。

1.  对于 `Product` 实体，告诉 SQLite `UnitPrice` 可以从 `decimal` 转换为 `double`。`OnModelCreating` 方法现在应该简化很多，如下列代码所示：

    ```cs
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
      modelBuilder.Entity<OrderDetail>(entity =>
      {
        entity.HasKey(e => new { e.OrderId, e.ProductId });
        entity.HasOne(d => d.Order)
          .WithMany(p => p.OrderDetails)
          .HasForeignKey(d => d.OrderId)
          .OnDelete(DeleteBehavior.ClientSetNull);
        entity.HasOne(d => d.Product)
          .WithMany(p => p.OrderDetails)
          .HasForeignKey(d => d.ProductId)
          .OnDelete(DeleteBehavior.ClientSetNull);
      });
      modelBuilder.Entity<Product>()
        .Property(product => product.UnitPrice)
        .HasConversion<double>();
      OnModelCreatingPartial(modelBuilder);
    } 
    ```

1.  添加一个名为 `NorthwindContextExtensions.cs` 的类，并修改其内容以定义一个扩展方法，该方法将 Northwind 数据库上下文添加到依赖服务集合中，如下列代码所示：

    ```cs
    using Microsoft.EntityFrameworkCore; // UseSqlite
    using Microsoft.Extensions.DependencyInjection; // IServiceCollection
    namespace Packt.Shared;
    public static class NorthwindContextExtensions
    {
      /// <summary>
      /// Adds NorthwindContext to the specified IServiceCollection. Uses the Sqlite database provider.
      /// </summary>
      /// <param name="services"></param>
      /// <param name="relativePath">Set to override the default of ".."</param>
      /// <returns>An IServiceCollection that can be used to add more services.</returns>
      public static IServiceCollection AddNorthwindContext(
        this IServiceCollection services, string relativePath = "..")
      {
        string databasePath = Path.Combine(relativePath, "Northwind.db");
        services.AddDbContext<NorthwindContext>(options =>
          options.UseSqlite($"Data Source={databasePath}")
        );
        return services;
      }
    } 
    ```

1.  构建两个类库并修复任何编译器错误。

## 使用 SQL Server 创建实体模型的类库

要使用 SQL Server，如果您已经在 *第十章*，*使用 Entity Framework Core 处理数据* 中设置了 Northwind 数据库，则无需执行任何操作。但现在您将使用 `dotnet-ef` 工具创建实体模型：

1.  使用您偏好的代码编辑器创建一个名为 `PracticalApps` 的新解决方案/工作区。

1.  添加一个类库项目，如以下列表所定义：

    1.  项目模板：**类库** / `classlib`

    1.  工作区/解决方案文件和文件夹：`PracticalApps`

    1.  项目文件和文件夹：`Northwind.Common.EntityModels.SqlServer`

1.  在 `Northwind.Common.EntityModels.SqlServer` 项目中，添加 SQL Server 数据库提供程序和 EF Core 设计时支持的包引用，如下列标记所示：

    ```cs
    <ItemGroup>
      <PackageReference
        Include="Microsoft.EntityFrameworkCore.SqlServer" 
        Version="6.0.0" />
      <PackageReference 
        Include="Microsoft.EntityFrameworkCore.Design" 
        Version="6.0.0">
        <PrivateAssets>all</PrivateAssets>
        <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
      </PackageReference>  
    </ItemGroup> 
    ```

1.  删除 `Class1.cs` 文件。

1.  构建项目。

1.  为 `Northwind.Common.EntityModels.SqlServer` 文件夹打开命令提示符或终端。

1.  在命令行中，为所有表生成实体类模型，如下列命令所示：

    ```cs
    dotnet ef dbcontext scaffold "Data Source=.;Initial Catalog=Northwind;Integrated Security=true;" Microsoft.EntityFrameworkCore.SqlServer --namespace Packt.Shared --data-annotations 
    ```

    注意以下事项：

    +   执行的命令：`dbcontext scaffold`

    +   连接字符串：`"Data Source=.;Initial Catalog=Northwind;Integrated Security=true;"`

    +   数据库提供程序：`Microsoft.EntityFrameworkCore.SqlServer`

    +   命名空间：`--namespace Packt.Shared`

    +   同时使用数据注解和 Fluent API：`--data-annotations`

1.  在 `Customer.cs` 中，添加一个正则表达式以验证其主键值，仅允许大写的西文字符，如下列高亮代码所示：

    ```cs
    [Key]
    [StringLength(5)]
    **[****RegularExpression(****"[A-Z]{5}"****)****]** 
    public string CustomerId { get; set; } = null!; 
    ```

1.  在 `Customer.cs` 中，使 `CustomerId` 和 `CompanyName` 属性成为必需。

1.  向解决方案/工作区中添加一个类库项目，如以下列表所定义：

    1.  项目模板：**类库** / `classlib`

    1.  工作区/解决方案文件和文件夹：`PracticalApps`

    1.  项目文件和文件夹：`Northwind.Common.DataContext.SqlServer`

1.  在 Visual Studio Code 中，选择 `Northwind.Common.DataContext.SqlServer` 作为活动的 OmniSharp 项目。

1.  在 `Northwind.Common.DataContext.SqlServer` 项目中，添加对 `Northwind.Common.EntityModels.SqlServer` 项目的项目引用，并添加对 EF Core SQL Server 数据提供程序的包引用，如下列标记所示：

    ```cs
    <ItemGroup>
      <PackageReference 
        Include="Microsoft.EntityFrameworkCore.SqlServer" 
        Version="6.0.0" />
    </ItemGroup>
    <ItemGroup>
      <ProjectReference Include=
        "..\Northwind.Common.EntityModels.SqlServer\Northwind.Common
    .EntityModels.SqlServer.csproj" />
    </ItemGroup> 
    ```

1.  在 `Northwind.Common.DataContext.SqlServer` 项目中，删除 `Class1.cs` 类文件。

1.  构建 `Northwind.Common.DataContext.SqlServer` 项目。

1.  将 `NorthwindContext.cs` 文件从 `Northwind.Common.EntityModels.SqlServer` 项目/文件夹移动到 `Northwind.Common.DataContext.SqlServer` 项目/文件夹。

1.  在 `NorthwindContext.cs` 中，移除关于连接字符串的编译器警告。

1.  添加一个名为 `NorthwindContextExtensions.cs` 的类，并修改其内容以定义一个扩展方法，该方法将 Northwind 数据库上下文添加到依赖服务集合中，如下列代码所示：

    ```cs
    using Microsoft.EntityFrameworkCore; // UseSqlServer
    using Microsoft.Extensions.DependencyInjection; // IServiceCollection
    namespace Packt.Shared;
    public static class NorthwindContextExtensions
    {
      /// <summary>
      /// Adds NorthwindContext to the specified IServiceCollection. Uses the SqlServer database provider.
      /// </summary>
      /// <param name="services"></param>
      /// <param name="connectionString">Set to override the default.</param>
      /// <returns>An IServiceCollection that can be used to add more services.</returns>
      public static IServiceCollection AddNorthwindContext(
        this IServiceCollection services, string connectionString = 
          "Data Source=.;Initial Catalog=Northwind;"
          + "Integrated Security=true;MultipleActiveResultsets=true;")
      {
        services.AddDbContext<NorthwindContext>(options =>
          options.UseSqlServer(connectionString));
        return services;
      }
    } 
    ```

1.  构建这两个类库并修复任何编译器错误。

**最佳实践**：我们为 `AddNorthwindContext` 方法提供了可选参数，以便我们可以覆盖硬编码的 SQLite 数据库文件路径或 SQL Server 数据库连接字符串。这将使我们拥有更多灵活性，例如，从配置文件加载这些值。

# 实践与探索

通过深入研究探索本章主题。

## 练习 13.1 – 测试你的知识

1.  .NET 6 是跨平台的。Windows Forms 和 WPF 应用可以在 .NET 6 上运行。那么这些应用是否能在 macOS 和 Linux 上运行呢？

1.  Windows Forms 应用如何定义其用户界面，以及为什么这可能是一个潜在问题？

1.  WPF 或 UWP 应用如何定义其用户界面，以及为什么这对开发者有益？

## 练习 13.2 – 探索主题

使用以下页面上的链接来了解更多关于本章涵盖主题的详细信息：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-13---introducing-practical-applications-of-c-and-net`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-13---introducing-practical-applications-of-c-and-net)

# 总结

本章中，你已了解到一些可用于使用 C#和.NET 构建实用应用的应用模型和工作负载。

你已创建了两到四个类库，用于定义与 Northwind 数据库交互的实体数据模型，可使用 SQLite、SQL Server 或两者兼用。

在接下来的六章中，你将学习如何构建以下内容的详细信息：

+   使用静态 HTML 页面和动态 Razor 页面的简单网站。

+   采用模型-视图-控制器（MVC）设计模式的复杂网站。

+   可被任何能发起 HTTP 请求的平台调用的 Web 服务，以及调用这些 Web 服务的客户端网站。

+   可托管在 Web 服务器、浏览器或混合 Web-原生移动和桌面应用中的 Blazor 用户界面组件。

+   使用 gRPC 实现远程过程调用的服务。

+   使用 SignalR 实现实时通信的服务。

+   提供简便灵活访问 EF Core 模型的服务。

+   Azure Functions 中托管的无服务器微服务。

+   使用.NET MAUI 构建跨平台的原生移动和桌面应用。
