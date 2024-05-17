# 第十三章：13

# 介绍 C#和.NET 的实际应用

本书的第三部分是关于 C#和.NET 的实际应用。您将学习如何构建跨平台项目，如网站、服务以及移动和桌面应用程序。

微软将构建应用程序的平台称为**应用模型**或**工作负载**。

在*第 1*至*18*和*20*章中，您可以使用特定于操作系统的 Visual Studio 或跨平台的 Visual Studio Code 和 JetBrains Rider 构建所有应用程序。在*第 19*章中，*使用.NET MAUI 构建移动和桌面应用程序*（可在[`github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf`](https://github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf)找到），尽管您可以使用 Visual Studio Code 构建移动和桌面应用程序，但并不容易。目前，Visual Studio 2022 for Windows 对.NET MAUI 的支持要比 Visual Studio Code 更好。

我建议您按顺序阅读本章和后续章节，因为后续章节将引用前面章节中的项目，您将积累足够的知识和技能来解决后续章节中的更棘手的问题。

在本章中，我们将涵盖以下主题：

+   了解 C#和.NET 的应用模型

+   ASP.NET Core 的新功能

+   项目结构

+   使用其他项目模板

+   为 Northwind 构建实体数据模型

# 了解 C#和.NET 的应用模型

由于本书涉及 C# 10 和.NET 6，我们将学习使用它们构建实际应用程序的应用模型，这些应用程序将出现在本书的其余章节中。

**了解更多**：微软在其.NET 应用程序架构指南文档中提供了大量关于实现应用模型的指导，您可以在以下链接阅读：[`www.microsoft.com/net/learn/architecture`](https://www.microsoft.com/net/learn/architecture)

## 使用 ASP.NET Core 构建网站

网站由多个网页组成，这些网页可以通过文件系统静态加载，也可以通过服务器端技术（如 ASP.NET Core）动态生成。Web 浏览器使用**统一资源定位符**（**URL**）进行`GET`请求，标识每个页面，并可以使用`POST`，`PUT`和`DELETE`请求操作服务器上存储的数据。

对于许多网站，Web 浏览器被视为演示层，几乎所有处理都在服务器端执行。一些 JavaScript 可能会在客户端使用，以实现一些演示功能，如轮播图。

ASP.NET Core 提供了多种构建网站的技术：

+   **ASP.NET Core Razor Pages**和**Razor 类库**是用于简单网站动态生成 HTML 的方式。您将在*第十四章*中详细了解它们，*使用 ASP.NET Core Razor Pages 构建网站*。

+   **ASP.NET Core MVC**是**Model-View-Controller**（**MVC**）设计模式的一种实现，用于开发复杂网站。您将在*第十五章*中详细了解它，*使用 Model-View-Controller 模式构建网站*。

+   **Blazor**允许您使用 C#和.NET 构建用户界面组件，而不是使用基于 JavaScript 的 UI 框架，如 Angular、React 和 Vue。**Blazor WebAssembly**在浏览器中运行您的代码，就像基于 JavaScript 的框架一样。**Blazor Server**在服务器上运行您的代码，并动态更新网页。您将在*第十七章*中详细了解 Blazor，*使用 Blazor 构建用户界面*。Blazor 不仅用于构建网站，还可以用于创建混合移动和桌面应用程序。

### 使用内容管理系统构建网站

大多数网站都有大量内容，如果开发人员每次需要更改内容时都必须参与其中，那将无法很好地扩展。**内容管理系统**（**CMS**）使开发人员能够定义内容结构和模板，以提供一致性和良好的设计，同时使非技术内容所有者能够轻松管理实际内容。他们可以创建新页面或内容块，并更新现有内容，知道它将以最小的努力为访问者呈现出色。

有大量的 CMS 可用于所有 Web 平台，如 WordPress 用于 PHP 或 Django CMS 用于 Python。支持现代.NET 的 CMS 包括 Optimizely Content Cloud、Piranha CMS 和 Orchard Core。

使用 CMS 的关键好处是它提供了友好的内容管理用户界面。内容所有者登录网站并自行管理内容。然后，使用 ASP.NET Core MVC 控制器和视图，或通过 Web 服务端点（称为**无头 CMS**）返回渲染的内容，以向作为移动或桌面应用程序、店内触点或使用 JavaScript 框架或 Blazor 构建的客户端提供内容。

本书不涵盖.NET CMS，因此我在 GitHub 存储库中包含了您可以了解更多信息的链接：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#net-content-management-systems`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#net-content-management-systems)

### 使用 SPA 框架构建 Web 应用程序

Web 应用程序，也称为**单页应用程序**（**SPAs**），由使用 Blazor WebAssembly、Angular、React、Vue 或专有 JavaScript 库等前端技术构建的单个 Web 页面组成，可以在需要时向后端 Web 服务发出请求以获取更多数据，并使用常见的序列化格式（如 XML 和 JSON）发布更新的数据。典型的例子是谷歌的 Web 应用程序，如 Gmail、地图和文档。

对于 Web 应用程序，客户端使用 JavaScript 框架或 Blazor WebAssembly 来实现复杂的用户交互，但大部分重要的处理和数据访问仍然发生在服务器端，因为 Web 浏览器对本地系统资源的访问受到限制。

JavaScript 是弱类型的，不适用于复杂项目，因此大多数 JavaScript 库这些天都使用 Microsoft TypeScript，它为 JavaScript 添加了强类型，并设计了许多现代语言功能，用于处理复杂的实现。

.NET SDK 具有基于 JavaScript 和 TypeScript 的 SPA 的项目模板，但我们不会在本书中花时间学习如何构建基于 JavaScript 和 TypeScript 的 SPA，尽管这些通常与 ASP.NET Core 作为后端一起使用，因为本书是关于 C#，而不是其他语言。

总之，C#和.NET 可以在服务器端和客户端用于构建网站，如*图 13.1*所示：

![](img/Image00100.jpg)

图 13.1：使用 C#和.NET 在服务器端和客户端构建网站

## 构建 Web 和其他服务

尽管我们不会学习基于 JavaScript 和 TypeScript 的 SPA，但我们将学习如何使用**ASP.NET Core Web API**构建 Web 服务，然后从我们的 ASP.NET Core 网站的服务器端代码调用该 Web 服务，然后稍后，我们将从 Blazor WebAssembly 组件和跨平台移动和桌面应用程序调用该 Web 服务。

虽然没有正式的定义，但有时根据其复杂性来描述服务：

+   **服务**：客户端应用程序所需的所有功能在一个单片服务中。

+   **微服务**：每个都专注于较小功能集的多个服务。

+   **Nanoservice**：作为服务提供的单个功能。与 24/7/365 托管的服务和微服务不同，纳米服务通常处于非活动状态，直到被调用以减少资源和成本。

除了使用 HTTP 作为底层通信技术和 API 的设计原则的 Web 服务之外，我们还将学习如何使用其他技术和设计理念构建服务，包括：

+   **gRPC**用于构建高效和高性能的服务，并支持几乎任何平台。

+   **SignalR**用于在组件之间构建实时通信。

+   **OData**用于使用 Web API 封装 Entity Framework Core 和其他数据模型。

+   **GraphQL**用于让客户端控制跨多个数据源检索哪些数据。

+   **Azure Functions**用于在云中托管无服务器纳米服务。

## 构建移动和桌面应用程序

有两个主要的移动平台：苹果的 iOS 和谷歌的 Android，每个平台都有自己的编程语言和平台 API。还有两个主要的桌面平台：苹果的 macOS 和微软的 Windows，每个平台都有自己的编程语言和平台 API，如下列表所示：

+   **iOS**：Objective C 或 Swift 和 UIkit。

+   **Android**：Java 或 Kotlin 和 Android API。

+   **macOS**：Objective C 或 Swift 和 AppKit 或 Catalyst。

+   **Windows**：C、C++或许多其他语言和 Win32 API 或 Windows App SDK。

由于本书是关于使用 C#和.NET 进行现代跨平台开发，因此不包括使用**Windows Forms**、**Windows Presentation Foundation**（**WPF**）或**Universal Windows Platform**（**UWP**）应用程序构建桌面应用程序的覆盖范围，因为它们仅适用于 Windows。

跨平台移动和桌面应用程序可以在**.NET 多平台应用程序用户界面**（**MAUI**）平台上构建一次，然后可以在许多移动和桌面平台上运行。

.NET MAUI 通过共享用户界面组件和业务逻辑使开发这些应用程序变得容易。它们可以针对与控制台应用程序、网站和 Web 服务使用的相同.NET API。应用程序将在移动设备上由 Mono 运行时执行，在桌面设备上由 CoreCLR 运行时执行。与普通的.NET CoreCLR 运行时相比，Mono 运行时更好地针对移动设备进行了优化。Blazor WebAssembly 也使用了 Mono 运行时，因为像移动应用程序一样，它受到资源限制。

这些应用程序可以独立存在，但它们通常调用服务来提供跨所有计算设备的体验，从服务器和笔记本电脑到手机和游戏系统。

.NET MAUI 的未来更新将支持现有的 MVVM 和 XAML 模式，以及像 C#这样的**Model-View-Update**（**MVU**）模式，这类似于苹果的 Swift UI。

第六版中的倒数第二章是*第十九章*，*使用.NET MAUI 构建移动和桌面应用程序*（可在[`github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf`](https://github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf)找到），涵盖了使用.NET MAUI 构建跨平台移动和桌面应用程序。

## .NET MAUI 的替代方案

在微软创建.NET MAUI 之前，第三方创建了开源倡议，以使.NET 开发人员能够使用名为**Uno**和**Avalonia**的 XAML 构建跨平台应用程序。

### 了解 Uno 平台

正如 Uno 在他们的网站上所说，它是“用于 Windows、WebAssembly、iOS、macOS、Android 和 Linux 的单一代码库应用程序的第一个和唯一 UI 平台。”

开发人员可以在本机移动、Web 和桌面之间重用 99%的业务逻辑和 UI 层。

Uno 平台使用 Xamarin 本机平台，但不使用 Xamarin.Forms。对于 WebAssembly，Uno 使用与 Blazor WebAssembly 一样的 Mono-WASM 运行时。对于 Linux，Uno 使用 Skia 在画布上绘制用户界面。

### 了解 Avalonia

根据.NET 基金会的网站，Avalonia“是一个跨平台的基于 XAML 的 UI 框架，提供灵活的样式系统，并支持诸如 Windows、通过 Xorg 的 Linux、macOS 等多种操作系统。Avalonia 已准备好用于通用桌面应用程序开发。”

您可以将 Avalonia 视为 WPF 的精神继承者。熟悉 WPF 的 WPF、Silverlight 和 UWP 开发人员可以继续受益于他们多年的现有知识和技能。

JetBrains 使用它来现代化他们基于 WPF 的工具并使其跨平台化。

Visual Studio 的 Avalonia 扩展和与 JetBrains Rider 的深度集成使开发更加简单和高效。

# ASP.NET Core 中的新功能

在过去的几年里，微软迅速扩展了 ASP.NET Core 的功能。您应该注意支持哪些.NET 平台，如下列表所示：

+   ASP.NET Core 1.0 到 2.2 可以在.NET Core 或.NET Framework 上运行。

+   ASP.NET Core 3.0 或更高版本仅在.NET Core 3.0 或更高版本上运行。

## ASP.NET Core 1.0

ASP.NET Core 1.0 于 2016 年 6 月发布，重点是实现适用于构建现代跨平台 Web 应用程序和服务的最小 API，适用于 Windows、macOS 和 Linux。

## ASP.NET Core 1.1

ASP.NET Core 1.1 于 2016 年 11 月发布，重点是修复错误并对功能和性能进行一般改进。

## ASP.NET Core 2.0

ASP.NET Core 2.0 于 2017 年 8 月发布，重点是添加新功能，如 Razor Pages，将程序集捆绑到`Microsoft.AspNetCore.All`元包中，针对.NET Standard 2.0，提供新的身份验证模型和性能改进。

ASP.NET Core 2.0 引入的最重要的新功能是 ASP.NET Core Razor Pages，涵盖在*第十四章*，*使用 ASP.NET Core Razor Pages 构建网站*中，以及 ASP.NET Core OData 支持，涵盖在*第十八章*，*构建和使用专门的服务*中（可在[`github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf`](https://github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf)上找到）。 

## ASP.NET Core 2.1

ASP.NET Core 2.1 于 2018 年 5 月发布，是一个**长期支持**（**LTS**）版本，意味着它在 2018 年 8 月 21 日之前得到支持，持续三年（LTS 指定直到 2018 年 8 月才正式分配给它，版本为 2.1.3）。

它的重点是添加新功能，如**SignalR**用于实时通信，**Razor 类库**用于重用 Web 组件，**ASP.NET Core 身份验证**以及更好地支持 HTTPS 和欧盟的**通用数据保护条例**（**GDPR**），包括以下表中列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| Razor 类库 | 14 | 使用 Razor 类库 |
| GDPR 支持 | 15 | 创建和探索 ASP.NET Core MVC 网站 |
| 身份 UI 库和脚手架 | 15 | 探索 ASP.NET Core MVC 网站 |
| 集成测试 | 15 | 测试 ASP.NET Core MVC 网站 |
| `[ApiController]`，`ActionResult<T>` | 16 | 创建 ASP.NET Core Web API 项目 |
| 问题详情 | 16 | 实现 Web API 控制器 |
| `IHttpClientFactory` | 16 | 使用 HttpClientFactory 配置 HTTP 客户端 |
| ASP.NET Core SignalR | 18 | 使用 SignalR 实现实时通信 |

## ASP.NET Core 2.2

ASP.NET Core 2.2 于 2018 年 12 月发布，重点是改进 RESTful HTTP API 的构建，更新项目模板以适配 Bootstrap 4 和 Angular 6，在 Azure 中进行优化配置，并进行性能改进，包括以下表中列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| Kestrel 中的 HTTP/2 | 14 | 经典 ASP.NET 与现代 ASP.NET Core |
| 进程内托管模型 | 14 | 创建 ASP.NET Core 项目 |
| 端点路由 | 14 | 理解端点路由 |
| 健康检查 API | 16 | 实现健康检查 API |
| Open API 分析器 | 16 | 实现 Open API 分析器和约定 |

## ASP.NET Core 3.0

ASP.NET Core 3.0 在 2019 年 9 月发布，重点是充分利用 .NET Core 3.0 和 .NET Standard 2.1，这意味着它不支持 .NET Framework，并且添加了有用的改进，包括以下表中列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| Razor 类库中的静态资源 | 14 | 使用 Razor 类库 |
| MVC 服务注册的新选项 | 15 | 了解 ASP.NET Core MVC 启动 |
| ASP.NET Core gRPC | 18 | 使用 ASP.NET Core gRPC 构建服务 |
| Blazor Server | 17 | 使用 Blazor Server 构建组件 |

## ASP.NET Core 3.1

ASP.NET Core 3.1 在 2019 年 12 月发布，是一个 LTS 版本，意味着将在 2022 年 12 月 3 日之前得到支持。它专注于改进，如对 Razor 组件的部分类支持和新的 `<component>` 标签助手。

## Blazor WebAssembly 3.2

Blazor WebAssembly 3.2 在 2020 年 5 月发布。这是一个当前版本，意味着项目必须在 .NET 5 版本发布后的三个月内升级，也就是在 2021 年 2 月 10 日之前。微软最终实现了使用 .NET 进行全栈 Web 开发的承诺，*第十七章* *使用 Blazor 构建用户界面* 中涵盖了 Blazor Server 和 Blazor WebAssembly。

## ASP.NET Core 5.0

ASP.NET Core 5.0 在 2020 年 11 月发布，重点是修复错误，使用缓存进行性能改进，Kestrel 中对证书认证进行缓存，HPACK 动态压缩 HTTP/2 响应头，ASP.NET Core 组件的可空注解，以及减少容器镜像大小，包括以下表中列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 扩展方法允许匿名访问端点 | 16 | 保护 Web 服务 |
| `HttpRequest` 和 `HttpResponse` 的 JSON 扩展方法 | 16 | 在控制器中以 JSON 形式获取客户端 |

## ASP.NET Core 6.0

ASP.NET Core 6.0 在 2021 年 11 月发布，重点是提高生产力，例如最小化代码以实现基本网站和服务，.NET Hot Reload，以及 Blazor 的新托管选项，如使用 .NET MAUI 的混合应用，包括以下表中列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 新的空白 Web 项目模板 | 14 | 了解空白 Web 模板 |
| HTTP 日志中间件 | 16 | 启用 HTTP 日志 |
| 最小 API | 16 | 实现最小的 Web API |
| Blazor 错误边界 | 17 | 定义 Blazor 错误边界 |
| Blazor WebAssembly AOT | 17 | 启用 Blazor WebAssembly 预编译 |
| .NET Hot Reload | 17 | 使用 .NET Hot Reload 修复代码 |
| .NET MAUI Blazor 应用 | 19 | 在 .NET MAUI 应用中托管 Blazor 组件 |

# 构建仅限于 Windows 的桌面应用程序

用于构建仅限于 Windows 的桌面应用程序的技术包括：

+   **Windows Forms**，2002 年。

+   **Windows Presentation Foundation**（**WPF**），2006 年。

+   **Windows Store** 应用，2012 年。

+   **Universal Windows Platform**（**UWP**）应用，2015 年。

+   **Windows App SDK**（原名 **WinUI 3** 和 **Project Reunion**）应用程序，2021 年。

## 了解传统的 Windows 应用程序平台

1985 年，微软发布了 Windows 1.0，创建 Windows 应用程序的唯一方法是使用 C 语言并调用三个核心 DLL 中的函数，这些 DLL 名为 kernel、user 和 GDI。一旦 Windows 在 Windows 95 中变为 32 位，这些 DLL 就会以 32 为后缀，并被称为 **Win32 API**。

1991 年，微软推出了 Visual Basic，为开发人员提供了一种可视化、从工具箱中拖放控件的方式来构建 Windows 应用程序的用户界面。它非常受欢迎，而 Visual Basic 运行时仍然作为 Windows 10 的一部分进行分发。

随着 2002 年发布的第一个 C#和.NET Framework 版本，微软提供了用于构建 Windows 桌面应用程序的技术，称为**Windows Forms**。当时用于 Web 开发的等效技术被命名为**Web Forms**，因此有了互补的名称。代码可以用 Visual Basic 或 C#语言编写。Windows Forms 具有类似的拖放可视化设计工具，尽管它生成 C#或 Visual Basic 代码来定义用户界面，这对人类来说可能难以直接理解和编辑。

2006 年，微软发布了用于构建 Windows 桌面应用程序的更强大的技术，称为**Windows Presentation Foundation**（**WPF**），作为.NET Framework 3.0 的关键组件，与**Windows Communication Foundation**（**WCF**）和**Windows Workflow**（**WF**）一起。

尽管可以通过编写仅 C#语句来创建 WPF 应用程序，但它也可以使用**可扩展应用标记语言**（**XAML**）来指定其用户界面，这对人类和代码来说都很容易理解。Windows 的 Visual Studio 部分是用 WPF 构建的。

2012 年，微软发布了 Windows 8 及其在受保护的沙箱中运行的 Windows Store 应用程序。

2015 年，微软发布了带有更新的 Windows Store 应用概念的 Windows 10，名为**Universal Windows Platform**（**UWP**）。UWP 应用程序可以使用 C++和 DirectX UI 构建，也可以使用 JavaScript 和 HTML 构建，或者使用一个不跨平台但提供对底层 WinRT API 完全访问权限的现代.NET 的自定义分支来使用 C#构建。

UWP 应用程序只能在 Windows 10 平台上执行，而不能在早期版本的 Windows 上执行，但 UWP 应用程序可以在 Xbox 和 Windows 混合现实头戴设备上运行，并带有运动控制器。

许多 Windows 开发人员拒绝了 Windows Store 和 UWP 应用程序，因为它们对底层系统的访问受到限制。微软最近创建了**Project Reunion**和**WinUI 3**，它们共同工作，允许 Windows 开发人员将现代 Windows 开发的一些好处带到现有的 WPF 应用程序中，并允许它们具有与 UWP 应用程序相同的好处和系统集成。这个倡议现在被称为**Windows App SDK**。

## 了解现代.NET 对遗留 Windows 平台的支持

Linux 和 macOS 上的.NET SDK 的磁盘大小约为 330 MB。Windows 上的.NET SDK 的磁盘大小约为 440 MB。这是因为它包括 Windows 桌面运行时，允许遗留的 Windows 应用程序平台 Windows Forms 和 WPF 在现代.NET 上运行。

有许多使用 Windows Forms 和 WPF 构建的企业应用程序需要维护或增强新功能，但直到最近它们一直停留在.NET Framework 上，这现在已经是一个遗留平台。使用现代.NET 及其 Windows 桌面包，这些应用程序现在可以使用.NET 的全部现代功能。

# 构建项目

您应该如何构建项目？到目前为止，我们已经构建了小型的个别控制台应用程序，以说明语言或库的特性。在本书的其余部分，我们将使用不同的技术构建多个项目，这些技术可以共同提供一个解决方案。

对于大型复杂的解决方案，要在所有代码中进行导航可能会很困难。因此，构建项目的主要原因是为了更容易地找到组件。最好为解决方案或工作区设置一个反映应用程序或解决方案的整体名称。

我们将为一个名为**Northwind**的虚构公司构建多个项目。我们将命名解决方案或工作区为`PracticalApps`，并在所有项目名称前使用名称`Northwind`作为前缀。

有许多结构和命名项目和解决方案的方法，例如使用文件夹层次结构以及命名约定。如果您在团队中工作，请确保您知道您的团队是如何做的。

## 在解决方案或工作区中构建项目

在解决方案或工作区中为项目制定命名约定是很好的，这样任何开发人员都可以立即知道每个项目的作用。一个常见的选择是使用项目的类型，例如类库、控制台应用程序、网站等，如下表所示：

| 名称 | 描述 |
| --- | --- |
| `Northwind.Common` | 用于跨多个项目使用的接口、枚举、类、记录和结构等常见类型的类库项目。 |
| `Northwind.Common.EntityModels` | 用于常见的 EF Core 实体模型的类库项目。实体模型通常在服务器端和客户端上都使用，因此最好将特定数据库提供程序的依赖项分开。 |
| `Northwind.Common.DataContext` | 一个依赖于特定数据库提供程序的 EF Core 数据库上下文的类库项目。 |
| `Northwind.Web` | 一个使用静态 HTML 文件和动态 Razor 页面混合的简单网站的 ASP.NET Core 项目。 |
| `Northwind.Razor.Component` | 用于在多个项目中使用的 Razor 页面的类库项目。 |
| `Northwind.Mvc` | 用于使用 MVC 模式的复杂网站的 ASP.NET Core 项目，可以更容易地进行单元测试。 |
| `Northwind.WebApi` | 用于 HTTP API 服务的 ASP.NET Core 项目。与网站集成的一个很好的选择，因为它们可以使用任何 JavaScript 库或 Blazor 与服务进行交互。 |
| `Northwind.OData` | 一个实现 OData 标准以启用客户端控制查询的 HTTP API 服务的 ASP.NET Core 项目。 |
| `Northwind.GraphQL` | 一个实现 GraphQL 标准以启用客户端控制查询的 HTTP API 服务的 ASP.NET Core 项目。 |
| `Northwind.gRPC` | 用于 gRPC 服务的 ASP.NET Core 项目。与任何语言和平台构建的应用程序集成的一个很好的选择，因为 gRPC 得到了广泛的支持，而且高效和高性能。 |
| `Northwind.SignalR` | 用于实时通信的 ASP.NET Core 项目。 |
| `Northwind.AzureFuncs` | 用于在 Azure Functions 中托管无服务器纳米服务的 ASP.NET Core 项目。 |
| `Northwind.BlazorServer` | 一个 ASP.NET Core Blazor Server 项目。 |
| `Northwind.BlazorWasm.Client` | 一个 ASP.NET Core Blazor WebAssembly 客户端项目。 |
| `Northwind.BlazorWasm.Server` | 一个 ASP.NET Core Blazor WebAssembly 服务器端项目。 |
| `Northwind.Maui` | 用于跨平台桌面/移动应用的.NET MAUI 项目。 |
| `Northwind.MauiBlazor` | 用于在操作系统中本地集成 Blazor 组件的.NET MAUI 项目。 |

# 使用其他项目模板

当您安装.NET SDK 时，会包含许多项目模板：

1.  在命令提示符或终端中，输入以下命令：

```cs

dotnet new --list

```

1.  您将看到当前安装的模板列表，包括如果您在 Windows 上运行，则包括用于 Windows 桌面开发的模板，如*图 13.2*所示：![](img/Image00101.jpg)

图 13.2：dotnet 项目模板列表

1.  注意与 Web 相关的项目模板，包括用于使用 Blazor、Angular 和 React 创建 SPA 的模板。但是另一个常见的 JavaScript SPA 库缺失：Vue。

## 安装额外的模板包

开发人员可以安装许多额外的模板包：

1.  启动浏览器并导航到[`dotnetnew.azurewebsites.net/`](http://dotnetnew.azurewebsites.net/)。

1.  在文本框中输入`vue`，注意 Vue.js 的可用模板列表，包括 Microsoft 发布的一个，如*图 13.3*所示：![](img/Image00102.jpg)

图 13.3：由 Microsoft 提供的 Vue.js 项目模板

1.  单击 Microsoft 的**ASP.NET Core with Vue.js**，注意安装和使用此模板的说明，如下命令所示：

```cs

dotnet new --install "Microsoft.AspNetCore.SpaTemplates"

dotnet new vue

```

1.  单击**查看此包中的其他模板**，注意除了 Vue.js 的项目模板外，它还有 Aurelia 和 Knockout.js 的项目模板。

# 为 Northwind 数据库构建实体数据模型

实际应用通常需要与关系数据库或其他数据存储中的数据一起工作。在本章中，我们将为存储在 SQL Server 或 SQLite 中的 Northwind 数据库定义一个实体数据模型。它将在我们在后续章节中创建的大多数应用程序中使用。

`Northwind4SQLServer.sql`和`Northwind4SQLite.sql`脚本文件是不同的。SQL Server 的脚本创建了 13 个表以及相关的视图和存储过程。SQLite 的脚本是一个简化版本，只创建了 10 个表，因为 SQLite 不支持那么多功能。本书的主要项目只需要这 10 个表，因此您可以使用任一数据库完成本书中的每个任务。

有关安装 SQL Server 和 SQLite 的说明，请参阅*第十章*，*使用 Entity Framework Core 处理数据*。在该章节中，您还将找到安装`dotnet-ef`工具的说明，您将使用该工具从现有数据库中生成实体模型。

**良好的实践**：您应该为实体数据模型创建一个单独的类库项目。这样可以更容易地在后端 Web 服务器和前端桌面、移动和 Blazor WebAssembly 客户端之间共享。

## 使用 SQLite 创建实体模型的类库

现在，您将在一个类库中定义实体数据模型，以便它们可以在其他类型的项目中重用，包括客户端应用程序模型。如果您不使用 SQL Server，则需要为 SQLite 创建此类库。如果您使用 SQL Server，则可以同时创建一个用于 SQLite 和一个用于 SQL Server 的类库，然后根据需要在它们之间切换。

我们将使用 EF Core 命令行工具自动生成一些实体模型：

1.  使用您喜欢的代码编辑器创建名为`PracticalApps`的新解决方案/工作区。

1.  添加一个类库项目，如下列表所定义的：

1.  项目模板：**类库** / `classlib`

1.  工作区/解决方案文件和文件夹：`PracticalApps`

1.  项目文件和文件夹：`Northwind.Common.EntityModels.Sqlite`

1.  在`Northwind.Common.EntityModels.Sqlite`项目中，添加 SQLite 数据库提供程序和 EF Core 设计时支持的包引用，如下标记所示：

```cs

<ItemGroup>

<PackageReference

包括="Microsoft.EntityFrameworkCore.Sqlite"

Version="6.0.0"

/>

<PackageReference

包括="Microsoft.EntityFrameworkCore.Design"

Version="6.0.0"

<PrivateAssets>all</PrivateAssets>

<IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>

</PackageReference>

</ItemGroup>

```

1.  删除`Class1.cs`文件。

1.  构建项目。

1.  通过将`Northwind4SQLite.sql`文件复制到`PracticalApps`文件夹中，然后在命令提示符或终端中输入以下命令，为 SQLite 创建`Northwind.db`文件：

```cs

sqlite3 Northwind.db -init Northwind4SQLite.sql

```

1.  请耐心等待，因为此命令可能需要一段时间来创建数据库结构，如下输出所示：

```cs

-- 从 Northwind4SQLite.sql 加载资源

SQLite 版本 3.35.5 2021-04-19 14:49:49

输入“.help”以获取使用提示。

sqlite>

```

1.  在 Windows 上按 Ctrl + C 或在 macOS 上按 Cmd + D 退出 SQLite 命令模式。

1.  打开`Northwind.Common.EntityModels.Sqlite`文件夹的命令提示符或终端。

1.  在命令行中，生成所有表的实体类模型，如下命令所示：

```cs

dotnet ef dbcontext scaffold "Filename=../Northwind.db" Microsoft.EntityFrameworkCore.Sqlite --namespace Packt.Shared --data-annotations

```

注意以下内容：

+   执行的命令：`dbcontext scaffold`

+   连接字符串。`"Filename=../Northwind.db"`

+   数据库提供程序：`Microsoft.EntityFrameworkCore.Sqlite`

+   命名空间：`--namespace Packt.Shared`

+   要使用数据注释以及 Fluent API：`--data-annotations`

1.  注意构建消息和警告，如下输出所示：

```cs

构建开始... 

构建成功。

为了保护连接字符串中可能包含的敏感信息，您应该将其移出源代码。您可以通过使用 Name= 语法从配置中读取连接字符串来避免生成连接字符串 - 请参阅 https://go.microsoft.com/fwlink/?linkid=2131148。有关存储连接字符串的更多指导，请参阅 http://go.microsoft.com/fwlink/?LinkId=723263。

```

### 改进类到表的映射

`dotnet-ef`命令行工具生成了不同的代码，因为 SQL Server 和 SQLite 支持不同级别的功能。

例如，SQL Server 文本列可以限制字符数。SQLite 不支持这一点。因此，`dotnet-ef`将生成验证属性，以确保`string`属性在 SQL Server 中限制为指定数量的字符，但在 SQLite 中不会，如下面的代码所示：

```cs

// SQLite 数据库提供程序生成的代码

[Column(TypeName =

"nvarchar (15)"

)

]

public

字符串

CategoryName { get

; set

; } = null

！

// SQL Server 数据库提供程序生成的代码

[StringLength(15)

]

public

字符串

CategoryName { get

; set

; } = null

！

```

数据库提供程序都不会将非空的`string`属性标记为必需的：

```cs

// 不对非空属性进行运行时验证

public

字符串

CategoryName { get

; set

; } = null

！

// 可空属性

public

字符串

? Description { get

; set

; }

// 使用属性进行运行时验证

[Required

]

public

字符串

CategoryName { get

; set

; } = null

！

```

我们将进行一些小的更改，以改进 SQLite 的实体模型映射和验证规则：

1.  打开`Customer.cs`文件，并添加一个正则表达式来验证其主键值，只允许大写西方字符，如下面代码中的高亮部分所示：

```cs

[Key

]

[Column(TypeName =

"nchar (5)"

)

]

**[**

**RegularExpression(**

**"[A-Z]{5}"**

**)** 

**]**

public

字符串

CustomerId { get

; set

; }

```

1.  激活您的代码编辑器的查找和替换功能（在 Visual Studio 2022 中，导航到**编辑** | **查找和替换** | **快速替换**），切换到**使用正则表达式**，然后在搜索框中输入一个正则表达式，如下面的表达式所示：

```cs

\[Column\(TypeName = "(nchar|nvarchar) \((.*)\)"

\)\]

```

1.  在替换框中，输入一个替换正则表达式，如下面的表达式所示：

```cs

$&\n    [StringLength($2

)]

```

在换行符`\n`之后，我已经包含了四个空格字符，以便在我的系统上正确缩进，我的系统每个缩进级别使用两个空格字符。您可以插入任意数量的空格。

1.  设置查找和替换以在当前项目中搜索文件。

1.  执行搜索和替换以替换所有内容，如*图 13.4*所示：![](img/Image00103.jpg)

图 13.4：在 Visual Studio 2022 中使用正则表达式搜索和替换所有匹配项

1.  将任何日期/时间属性更改为使用可空的`DateTime`，例如在`Employee.cs`中，而不是使用字节数组，如下面的代码所示：

```cs

// 之前

[Column(TypeName =

"datetime"

)

]

public

字节

[] BirthDate { get

; set

; }

// 之后

[Column(TypeName =

"datetime"

)

]

public

DateTime? BirthDate { get

; set

; }

```

使用您的代码编辑器的查找功能来搜索`"datetime"`，以查找所有需要更改的属性。

1.  将任何`money`属性更改为使用可空的`decimal`，例如在`Order.cs`中，而不是使用字节数组，如下面的代码所示：

```cs

// 之前

[Column(TypeName =

"money"

)

]

public

字节

[] Freight { get

; set

; }

// 之后

[Column(TypeName =

"money"

)

]

public

decimal

? Freight { get

; set

; }

```

使用您的代码编辑器的查找功能来搜索`"money"`，以查找所有需要更改的属性。

1.  将任何`bit`属性更改为使用`bool`，例如在`Product.cs`中，而不是使用字节数组，如下面的代码所示：

```cs

// 之前

[Column(TypeName =

"bit"

)

]

public

字节

[] Discontinued { get

; set

; } = null

！

// after

[Column(TypeName =

"bit"

)

]

public

bool

Discontinued { get

; set

; }

```

使用您的代码编辑器的查找功能来搜索`"bit"`，以找到所有需要更改的属性。

1.  在`Category.cs`中，将`CategoryId`属性改为`int`，如下所示在以下代码中突出显示：

```cs

[Key

]

public

**int**

CategoryId { get

; set

; }

```

1.  在`Category.cs`中，使`CategoryName`属性为必填项，如下所示在以下代码中突出显示：

```cs

**[**

**Required**

**]**

[Column(TypeName =

"nvarchar (15)"

)

]

[StringLength(15)

]

public

string

CategoryName { get

; set

; }

```

1.  在`Customer.cs`中，使`CompanyName`属性为必填项，如下所示在以下代码中突出显示：

```cs

**[**

**Required**

**]**

[Column(TypeName =

"nvarchar (40)"

)

]

[StringLength(40)

]

public

string

CompanyName { get

; set

; }

```

1.  在`Employee.cs`中，将`EmployeeId`属性改为`int`而不是`long`。

1.  在`Employee.cs`中，使`FirstName`和`LastName`属性为必填项。

1.  在`Employee.cs`中，将`ReportsTo`属性改为`int?`而不是`long?`。

1.  在`EmployeeTerritory.cs`中，将`EmployeeId`属性改为`int`而不是`long`。

1.  在`EmployeeTerritory.cs`中，使`TerritoryId`属性为必填项。

1.  在`Order.cs`中，将`OrderId`属性改为`int`而不是`long`。

1.  在`Order.cs`中，使用正则表达式装饰`CustomerId`属性以强制五个大写字符。

1.  在`Order.cs`中，将`EmployeeId`属性改为`int?`而不是`long?`。

1.  在`Order.cs`中，将`ShipVia`属性改为`int?`而不是`long?`。

1.  在`OrderDetail.cs`中，将`OrderId`属性改为`int`而不是`long`。

1.  在`OrderDetail.cs`中，将`ProductId`属性改为`int`而不是`long`。

1.  在`OrderDetail.cs`中，将`Quantity`属性改为`short`而不是`long`。

1.  在`Product.cs`中，将`ProductId`属性改为`int`而不是`long`。

1.  在`Product.cs`中，使`ProductName`属性为必填项。

1.  在`Product.cs`中，将`SupplierId`和`CategoryId`属性改为`int?`而不是`long?`。

1.  在`Product.cs`中，将`UnitsInStock`，`UnitsOnOrder`和`ReorderLevel`属性改为`short?`而不是`long?`。

1.  在`Shipper.cs`中，将`ShipperId`属性改为`int`而不是`long`。

1.  在`Shipper.cs`中，使`CompanyName`属性为必填项。

1.  在`Supplier.cs`中，将`SupplierId`属性改为`int`而不是`long`。

1.  在`Supplier.cs`中，使`CompanyName`属性为必填项。

1.  在`Territory.cs`中，将`RegionId`属性改为`int`而不是`long`。

1.  在`Territory.cs`中，使`TerritoryId`和`TerritoryDescription`属性为必填项。

现在我们有了一个用于实体类的类库，我们可以为数据库上下文创建一个类库。

### 创建一个 Northwind 数据库上下文的类库

现在您将定义一个数据库上下文类库：

1.  向解决方案/工作空间添加一个类库项目，如下列表中所定义的：

1.  项目模板：**Class Library** / `classlib`

1.  工作空间/解决方案文件和文件夹：`PracticalApps`

1.  项目文件和文件夹：`Northwind.Common.DataContext.Sqlite`

1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择。

1.  在 Visual Studio Code 中，选择`Northwind.Common.DataContext.Sqlite`作为活动的 OmniSharp 项目。

1.  在`Northwind.Common.DataContext.Sqlite`项目中，添加对`Northwind.Common.EntityModels.Sqlite`项目的项目引用，并添加对 SQLite 的 EF Core 数据提供程序的包引用，如下标记所示：

```cs

<ItemGroup>

<PackageReference

Include="Microsoft.EntityFrameworkCore.SQLite"

Version="6.0.0"

/>

</ItemGroup>

<ItemGroup>

<ProjectReference Include=

"..\Northwind.Common.EntityModels.Sqlite\Northwind.Common

.EntityModels.Sqlite.csproj"

/>

</ItemGroup>

```

项目引用的路径不应在项目文件中换行。

1.  在`Northwind.Common.DataContext.Sqlite`项目中，删除`Class1.cs`类文件。

1.  构建`Northwind.Common.DataContext.Sqlite`项目。

1.  将`NorthwindContext.cs`文件从`Northwind.Common.EntityModels.Sqlite`项目/文件夹移动到`Northwind.Common.DataContext.Sqlite`项目/文件夹。

在 Visual Studio **Solution Explorer**中，如果您在项目之间拖放文件，它将被复制。如果您在拖放时按住 Shift 键，它将被移动。在 Visual Studio Code **EXPLORER**中，如果您在项目之间拖放文件，它将被移动。如果您在拖放时按住 Ctrl 键，它将被复制。

1.  在`NorthwindContext.cs`中，在`OnConfiguring`方法中，删除关于连接字符串的编译器`#warning`。

**良好实践**：我们将覆盖默认的数据库连接字符串，以便在需要与 Northwind 数据库一起工作的网站等项目中使用，因此从`DbContext`派生的类必须具有带有`DbContextOptions`参数的构造函数才能正常工作，如下面的代码所示：

```cs

公共

NorthwindContext

(DbContextOptions<NorthwindContext> options)

: base

(options)

{

}

```

1.  在`OnModelCreating`方法中，删除所有调用`ValueGeneratedNever`方法的 Fluent API 语句，以配置主键属性，如`SupplierId`，以便不自动生成值，或调用`HasDefaultValueSql`方法，如下面的代码所示：

```cs

modelBuilder.Entity<Supplier>(entity =>

{

entity.Property(e => e.SupplierId).ValueGeneratedNever();

});

```

如果我们不删除上面的配置语句，那么当我们添加新的供应商时，`SupplierId`的值将始终为 0，我们只能添加一个具有该值的供应商，然后所有其他尝试都会引发异常。

1.  对于`Product`实体，告诉 SQLite`UnitPrice`可以从`decimal`转换为`double`。`OnModelCreating`方法现在应该更简化，如下面的代码所示：

```cs

保护

覆盖

空

OnModelCreating

(

ModelBuilder modelBuilder

)

{

modelBuilder.Entity<OrderDetail>(entity =>

{

entity.HasKey(e => new

{ e.OrderId, e.ProductId });

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

.HasConversion<double

>();

OnModelCreatingPartial(modelBuilder);

}

```

1.  添加名为`NorthwindContextExtensions.cs`的类库，并修改其内容以定义将 Northwind 数据库上下文添加到依赖服务集合的扩展方法，如下面的代码所示：

```cs

使用

Microsoft.EntityFrameworkCore; // UseSqlite

使用

Microsoft.Extensions.DependencyInjection; // IServiceCollection

命名空间

Packt.Shared

;

公共

静态

类

NorthwindContextExtensions

{

///

<摘要>

///

将 NorthwindContext 添加到指定的 IServiceCollection。使用 Sqlite 数据库提供程序。

///

</摘要>

///

<param name="services"></param>

///

<param name="relativePath">

设置以覆盖默认值".."

</param>

///

<返回>

可以用于添加更多服务的 IServiceCollection。

</returns>

公共

静态

IServiceCollection

AddNorthwindContext

(

这

IServiceCollection services,

字符串

relativePath =

".."

)

{

字符串

databasePath = Path.Combine(relativePath, "Northwind.db"

);

services.AddDbContext<NorthwindContext>(options =>

options.UseSqlite($"数据源=

{databasePath}

"

)

);

返回

服务;

}

}

```

1.  构建两个类库并修复任何编译错误。

## 为使用 SQL Server 创建实体模型的类库

如果您已经在*第十章* *使用 Entity Framework Core 处理数据*中设置了 Northwind 数据库，那么使用 SQL Server 时将不需要做任何事情。但是，您现在将使用`dotnet-ef`工具创建实体模型：

1.  使用您喜欢的代码编辑器创建一个名为`PracticalApps`的新解决方案/工作区。

1.  添加一个类库项目，如下列表所示：

1.  项目模板：**类库** / `classlib`

1.  工作区/解决方案文件和文件夹：`PracticalApps`

1.  项目文件和文件夹：`Northwind.Common.EntityModels.SqlServer`

1.  在`Northwind.Common.EntityModels.SqlServer`项目中，添加 SQL Server 数据库提供程序和 EF Core 设计时支持的包引用，如下标记所示：

```cs

<ItemGroup>

<PackageReference

Include="Microsoft.EntityFrameworkCore.SqlServer"

Version="6.0.0"

/>

<PackageReference

Include="Microsoft.EntityFrameworkCore.Design"

Version="6.0.0"

<PrivateAssets>all</PrivateAssets>

<IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>

</PackageReference>

</ItemGroup>

```

1.  删除`Class1.cs`文件。

1.  构建项目。

1.  打开`Northwind.Common.EntityModels.SqlServer`文件夹的命令提示符或终端。

1.  在命令行中，生成所有表的实体类模型，如下命令所示：

```cs

dotnet ef dbcontext scaffold "Data Source=.;Initial Catalog=Northwind;Integrated Security=true;" Microsoft.EntityFrameworkCore.SqlServer --namespace Packt.Shared --data-annotations

```

注意以下内容：

+   执行的命令：`dbcontext scaffold`

+   连接字符串。`"Data Source=.;Initial Catalog=Northwind;Integrated Security=true;"`

+   数据库提供程序：`Microsoft.EntityFrameworkCore.SqlServer`

+   命名空间：`--namespace Packt.Shared`

+   要使用数据注释以及 Fluent API：`--data-annotations`

1.  在`Customer.cs`中，添加一个正则表达式来验证其主键值，只允许大写西方字符，如下所示：

```cs

[Key

]

[StringLength(5)

]

**[**

**RegularExpression(**

**"[A-Z]{5}"**

**)**

**]**

public

string

CustomerId { get

; set

; } = null

!;

```

1.  在`Customer.cs`中，使`CustomerId`和`CompanyName`属性为必填项。

1.  将一个类库项目添加到解决方案/工作区中，如下列表所示：

1.  项目模板：**类库** / `classlib`

1.  工作区/解决方案文件和文件夹：`PracticalApps`

1.  项目文件和文件夹：`Northwind.Common.DataContext.SqlServer`

1.  在 Visual Studio Code 中，将`Northwind.Common.DataContext.SqlServer`选择为活动的 OmniSharp 项目。

1.  在`Northwind.Common.DataContext.SqlServer`项目中，添加对`Northwind.Common.EntityModels.SqlServer`项目的项目引用，并添加对 SQL Server 的 EF Core 数据提供程序的包引用，如下标记所示：

```cs

<ItemGroup>

<PackageReference

Include="Microsoft.EntityFrameworkCore.SqlServer"

Version="6.0.0"

/>

</ItemGroup>

<ItemGroup>

<ProjectReference Include=

"..\Northwind.Common.EntityModels.SqlServer\Northwind.Common

.EntityModels.SqlServer.csproj"

/>

</ItemGroup>

```

1.  在`Northwind.Common.DataContext.SqlServer`项目中，删除`Class1.cs`类文件。

1.  构建`Northwind.Common.DataContext.SqlServer`项目。

1.  将`NorthwindContext.cs`文件从`Northwind.Common.EntityModels.SqlServer`项目/文件夹移动到`Northwind.Common.DataContext.SqlServer`项目/文件夹。

1.  在`NorthwindContext.cs`中，删除有关连接字符串的编译器警告。

1.  添加名为`NorthwindContextExtensions.cs`的类，并修改其内容以定义将 Northwind 数据库上下文添加到依赖服务集合的扩展方法，如下所示：

```cs

using

Microsoft.EntityFrameworkCore; // UseSqlServer

using

Microsoft.Extensions.DependencyInjection; // IServiceCollection

namespace

Packt.Shared

;

public

static

class

NorthwindContextExtensions

{

///

<summary>

///

将 NorthwindContext 添加到指定的 IServiceCollection。使用 SqlServer 数据库提供程序。

///

</summary>

///

<param name="services"></param>

///

<param name="connectionString">

设置为覆盖默认值。

</param>

///

<returns>

可以用来添加更多服务的 IServiceCollection。

</返回>

公共

静态

IServiceCollection

AddNorthwindContext

（

这

IServiceCollection 服务，

字符串

连接字符串=

“数据源=。；初始目录=Northwind；”

+

“集成安全性=true；MultipleActiveResultsets=true；”

）

{

services.AddDbContext<NorthwindContext>(options =>

options.UseSqlServer(connectionString));

返回

服务；

}

}

```

1.  构建两个类库并修复任何编译错误。

**良好实践**：我们为`AddNorthwindContext`方法提供了可选参数，以便我们可以覆盖硬编码的 SQLite 数据库文件名路径或 SQL Server 数据库连接字符串。这将使我们更灵活，例如，可以从配置文件中加载这些值。

# 练习和探索

通过深入研究探索本章的主题。

## 练习 13.1-测试您的知识

1.  .NET 6 是跨平台的。 Windows 窗体和 WPF 应用程序可以在.NET 6 上运行。因此，这些应用程序是否可以在 macOS 和 Linux 上运行？

1.  Windows 窗体应用程序如何定义其用户界面，为什么这可能是一个问题？

1.  WPF 或 UWP 应用程序如何定义其用户界面，为开发人员有何好处？

## 练习 13.2-探索主题

使用以下页面上的链接，了解本章涵盖的主题的更多详细信息：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-13---introducing-practical-applications-of-c-and-net`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-13---introducing-practical-applications-of-c-and-net)

# 摘要

在本章中，您已经了解了一些可以使用 C#和.NET 构建实用应用程序的应用程序模型和工作负载。

您已经创建了两到四个类库，用于定义与 Northwind 数据库一起使用 SQLite 或 SQL Server 或两者都使用的实体数据模型。

在接下来的六章中，您将学习如何构建以下细节：

+   使用静态 HTML 页面和动态 Razor 页面创建简单网站。

+   使用模型-视图-控制器（MVC）设计模式构建复杂的网站。

+   可以由任何可以发出 HTTP 请求的平台调用的 Web 服务，以及调用这些 Web 服务的客户端网站。

+   Blazor 用户界面组件可以托管在 Web 服务器上，在浏览器中，或者在混合 Web 本机移动和桌面应用程序上。

+   使用 gRPC 实现远程过程调用的服务。

+   使用 SignalR 实现实时通信的服务。

+   提供对 EF Core 模型的简单灵活访问的服务。

+   在 Azure Functions 中托管的无服务器纳米服务。

+   使用.NET MAUI 构建跨平台本机移动和桌面应用程序。
