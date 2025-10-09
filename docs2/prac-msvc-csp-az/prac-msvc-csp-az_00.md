# 前言

**.NET Aspire** 是一个提供工具和库的新框架，用于使用 .NET 创建微服务，无论它们是否应在本地、Microsoft Azure 或任何其他云环境中运行。在本书中，你将学习如何在构建解决方案时充分利用 .NET Aspire。

创建 ASP.NET Core 最小 API（创建 REST 服务的一个简单快捷的选项）只是使用基于微服务架构创建应用程序的一部分。本书涵盖了构建成功解决方案所需的所有不同方面。访问数据库，包括关系型数据库和 NoSQL 数据库；使用 Docker 和部署 Docker 镜像；使用 GitHub Actions 进行自动部署；通过日志、指标数据和分布式跟踪来监控解决方案；创建单元测试、集成测试和负载测试；自动将解决方案发布到不同的环境；以及使用二进制、实时和异步通信——本书都涵盖了这些内容。

通过本书提供的代码，你将开发一个运行酷游戏的后端解决方案。从 *第二章* 开始，你将拥有可用和可测试的功能，并且将逐章增强，涵盖与微服务相关的重要方面。如果你不想按顺序逐章工作，我们为每一章都提供了可以开始的代码。

应用程序可以部署到 Microsoft Azure，并使用多个 Azure 服务，如 Azure 容器应用、容器注册库、Cosmos DB、应用配置、密钥保管库、Redis 和 SignalR 服务。它也可以在本地环境中运行在 Kubernetes 集群上，使用 Kafka、Redis 和其他资源。

到本书结束时，你将能够自信地实现一个稳定、性能良好且可扩展的解决方案，并使用各种非常适合托管此类基于服务的解决方案的 Azure 服务。虽然本书的解决方案是一个游戏，但学到的知识将帮助你创建任何与业务相关的服务架构。

# 本书面向的对象

本书面向熟悉 C# 和 .NET、对 Microsoft Azure 有基本了解，并希望了解创建现代 .NET 和 Microsoft Azure 微服务所需所有方面的开发人员和软件架构师。

# 本书涵盖的内容

*第一章*，*.NET Aspire 和微服务简介*，为你介绍了 **.NET Aspire**，以及创建微服务时可能非常有用的工具和库。你将开始一个初始的 .NET Aspire 项目，我们将查看它包含的内容以及你可以利用的 .NET Aspire 的部分。你将看到 Codebreaker 应用程序（这是我们将在本书的所有章节中构建的解决方案）由哪些服务组成，并了解与 Microsoft Azure 一起使用的服务。

*第二章*，*最小 API – 创建 REST 服务*，是创建 Codebreaker 应用程序的起点。你将学习如何使用**ASP.NET Core 最小 API**技术高效地创建 REST 服务，使用 OpenAPI 描述服务，以及使用 HTTP 文件测试服务。

*第三章*，*将数据写入关系型和非关系型数据库*，仅使用*第二章*中的内存存储，并添加使用**Azure SQL**和**Azure Cosmos DB**的数据库存储，比较关系型数据库和非关系型数据库，并使用**EF Core**处理两种变体。

*第四章*，*为客户端应用程序创建库*，添加了客户端库以访问服务，其中一个变体使用 HTTP 客户端工厂，另一个变体则利用**Kiota**自动生成客户端代码。

*第五章*，*微服务的容器化*，深入探讨了**Docker**的所有重要概念以及如何从迄今为止创建的服务中创建 Docker 镜像。在使用.NET CLI 创建 Docker 镜像之前，你将学习 Docker 的概念。将创建一个**.NET 原生 AOT**版本的服务，允许在不使用.NET 运行时的情况下仅使用原生代码创建 Docker 镜像。

*第六章*，*Microsoft Azure 托管应用程序*，由于上一章已创建 Docker 镜像，现在将介绍如何将应用程序发布到**Azure Container Apps**环境。在此之前，将介绍 Azure 的重要概念。然后，将使用 Azure Developer CLI 和.NET Aspire 创建 Azure 资源。

*第七章*，*灵活的配置*，深入探讨了.NET 配置。你将了解.NET 中的配置提供程序，学习配置如何应用于.NET Aspire 的应用模型，使用 Azure Container Apps 添加配置和秘密，并集成**Azure App Configuration**和**Azure Key Vault**。为了便于访问而无需存储秘密，这里还涵盖了**Azure 托管标识**。

*第八章*，*CI/CD – 使用 GitHub Actions 发布*，基于持续集成和持续交付是微服务解决方案的重要方面的事实。本章介绍了如何使用**GitHub actions**自动构建和测试应用程序，以及如何自动更新在 Microsoft Azure 上运行的应用程序。为了支持现代部署模式，本章集成了 Azure App Configuration 中可用的功能标志。

*第九章*，*服务和客户端的认证和授权*，涵盖了两种认证和授权应用程序和用户的方法：Codebreaker 解决方案的云版本集成**Azure Active Directory B2C**和本地解决方案的**ASP.NET Core identities**。为了避免在每次服务中处理认证，创建了一个使用**YARP**的网关。

*第十章*，*关于测试解决方案的所有内容*，指出任何更改都不应破坏应用程序，错误应尽早检测。在本章中，您将了解创建单元测试、使用.NET Aspire 的集成测试（这使得测试变得简单得多），以及使用**Playwright**进行端到端测试。

*第十一章*，*日志记录和监控*，深入探讨了 Codebreaker 应用程序中正在发生的事情。内存泄漏应尽早检测。在开发过程中，我们应该了解应用程序是如何进行通信的。本章涵盖了高效高性能的日志记录、编写自定义指标数据以及分布式跟踪——包括**OpenTelemetry**的覆盖范围以及它如何与.NET Aspire 集成。在本章中，我们使用**Prometheus**和**Grafana**进行本地解决方案，以及**Azure 应用程序洞察**和**Azure 日志分析**进行云解决方案。

*第十二章*，*服务扩展*，深入探讨了服务扩展，这是使用微服务架构的重要原因之一。使用**Azure 负载测试**，我们对应用程序的主要服务创建大量负载，找出瓶颈，决定是进行垂直扩展还是水平扩展，并使用 Redis 添加缓存以提高性能。

*第十三章*，*使用 SignalR 进行实时消息传递*，涵盖了使用**SignalR**实时通知客户端。使用 REST API，调用 SignalR 中心，传递关于完成游戏的实时信息，SignalR 中心将此信息传递给一组客户端。**Azure SignalR 服务**用于减少服务的负载。

*第十四章*，*二进制通信的 gRPC*，通过将通信改为服务间的**gRPC**来提高性能。您将学习如何创建协议缓冲区定义，使用这个二进制平台无关的通信平台实现服务和客户端，以及如何将.NET 服务发现和.NET Aspire 与 gRPC 结合使用。

*第十五章*，*使用消息和事件进行异步通信*，处理了请求发送后不需要立即得到答案的事实——消息队列和事件就派上用场。在这里，用于本地环境的**Azure 消息队列**、**Azure 事件中心**和**Kafka**被用于使用。

*第十六章*，*在本地和云端运行应用程序*，讨论了在 Azure 上运行时生产环境所需的内容，生产环境和开发环境之间的区别，以及 Codebreaker 应用如何满足可扩展性、可靠性和安全性方面的要求。到本章为止，应用程序要么在本地开发系统上运行，要么在 Azure 容器应用环境中运行。在本章中，应用程序被部署到 **Azure Kubernetes 服务**，并且可以使用 **Aspir8** 工具以类似的方式部署到本地 Kubernetes 集群。

# 要充分利用本书

您需要了解 C# 和 .NET，并具备一些关于 Microsoft Azure 的基础知识。以下工具和应用程序需要安装：

| **工具** | **安装** |
| --- | --- |
| **Visual Studio** **2022 (可选)** | [`visualstudio.microsoft.com/downloads/`](https://visualstudio.microsoft.com/downloads/) |
| **Visual** **Studio Code** | [`code.visualstudio.com/download`](https://code.visualstudio.com/download) |
| **Docker Desktop** | [`docs.docker.com/desktop/install/windows-install/`](https://docs.docker.com/desktop/install/windows-install/)[`docs.docker.com/desktop/install/mac-install/`](https://docs.docker.com/desktop/install/mac-install/)[`docs.docker.com/desktop/install/linux-install/`](https://docs.docker.com/desktop/install/linux-install/) |
| **Microsoft Azure 订阅** | [`azure.microsoft.com/en-us/free/`](https://azure.microsoft.com/en-us/free/) |
| **Azure CLI** | [`learn.microsoft.com/en-us/cli/azure/install-azure-cli`](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) |
| **Azure 开发者 CLI** | [`learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd`](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd) |
| **Azure Cosmos DB 模拟器** | [`learn.microsoft.com/en-us/azure/cosmos-db/how-to-develop-emulator`](https://learn.microsoft.com/en-us/azure/cosmos-db/how-to-develop-emulator) |
| **.NET Aspire** | [`learn.microsoft.com/en-us/dotnet/aspire/fundamentals/setup-tooling`](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/setup-tooling) |

Visual Studio 2022 是本列表中唯一需要 Windows 的软件。所有其他工具都可以在 Windows、macOS 或 Linux 上使用。在除 Windows 以外的平台上，您可以使用 Visual Studio Code 或其他工具来处理 .NET 和 C#。

要使用 .NET Aspire 与 Visual Studio 配合，至少需要安装版本 17.10.0。您可以使用 Visual Studio 2022 社区版。

从*第六章*开始，您需要一个 Microsoft Azure 订阅。您可以在 [`azure.microsoft.com/free`](https://azure.microsoft.com/free) 免费激活 Microsoft Azure，这将为您的 Azure 账户提供约 200 美元的信用额度，可用于前 30 天，以及在此之后可以免费使用的多项服务。如果您拥有 Visual Studio Professional 或 Enterprise 订阅，您每月还可以获得一定数量的 Azure 资源。您只需使用 Visual Studio 订阅激活即可：[`visualstudio.microsoft.com/subscriptions/`](https://visualstudio.microsoft.com/subscriptions/)。

**如果您正在使用本书的数字版，我们建议您亲自输入代码或从本书的 GitHub 仓库（下一节中提供链接）获取代码。这样做将帮助您避免与代码复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件：[`github.com/PacktPublishing/Pragmatic-Microservices-with-CSharp-and-Azure`](https://github.com/PacktPublishing/Pragmatic-Microservices-with-CSharp-and-Azure)。

源代码使用 .NET 8。在 .NET 9 发布后，主分支中的源代码将使用 .NET 9，并在 readme 文件中描述了相应的更改。那时，.NET 8 的源代码将在 dotnet8 分支中可用。在代码更新到新版本期间，您可以检查包含更改的额外分支。

我们还有其他来自我们丰富的书籍和视频目录的代码包，可在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 获取。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。以下是一个示例：“将下载的 `WebStorm-10*.dmg` 磁盘镜像文件挂载为系统中的另一个磁盘。”

代码块设置如下：

```cs
public static class LiveGamesEndpoints
{
  public static void MapLiveGamesEndpoints(this 
    IEndpointRouteBuilder routes, ILogger logger)
  {
    var group = routes.MapGroup("/live")
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```cs
var builder = WebApplication.CreateBuilder(args);
builder.AddServiceDefaults();
builder.AddApplicationServices();
```

任何命令行输入或输出都如下所示：

```cs
dotnet new console -o LiveTestClient
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词以**粗体**显示。以下是一个示例：“点击**启用实时跟踪**复选框，然后点击以收集有关**连接日志**、**消息日志**和**HTTP 请求日志**的信息。”

小贴士或重要注意事项

看起来是这样的。

# 联系我们

我们欢迎读者提供反馈。

**一般反馈**：如果你对这本书的任何方面有疑问，请通过 customercare@packtpub.com 给我们发邮件，并在邮件主题中提及书名。对于你与源代码有关的问题和问题，你可以使用这本书的 GitHub 存储库的**讨论**论坛：[`github.com/PacktPublishing/Pragmatic-Microservices-with-CSharp-and-Azure/discussions`](https://github.com/PacktPublishing/Pragmatic-Microservices-with-CSharp-and-Azure/discussions)

**勘误**: 尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果你在这本书中发现了错误，我们将不胜感激，如果你能向我们报告这一点。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**: 如果你在互联网上以任何形式遇到我们作品的非法副本，如果你能提供位置地址或网站名称，我们将不胜感激。请通过版权@packt.com 与我们联系，并提供材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专业知识，并且你感兴趣的是撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享你的想法

一旦你阅读了《使用 C#和 Azure 的实用微服务》，我们很乐意听听你的想法！请[点击此处直接进入此书的亚马逊评论页面](https://packt.link/r/1835088295)并分享你的反馈。

你的评论对我们和科技社区都很重要，并将帮助我们确保我们提供高质量的内容。

# 下载这本书的免费 PDF 副本

感谢你购买这本书！

你喜欢在路上阅读，但无法携带你的印刷书籍到处走吗？

你的电子书购买是否与你的选择设备不兼容？

别担心，现在每购买一本 Packt 书籍，你都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何地方、任何设备上阅读。直接从你最喜欢的技术书籍中搜索、复制和粘贴代码到你的应用程序中。

优惠不会就此结束，你还可以获得独家折扣、时事通讯和每天收件箱中的优质免费内容。

按照以下简单步骤获取好处：

1.  扫描二维码或访问以下链接

![](img/B21217_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781835088296`](https://packt.link/free-ebook/9781835088296)

1.  提交你的购买证明

1.  就这样！我们将直接将免费 PDF 和其他好处发送到你的电子邮件。

# 第一部分：使用.NET 创建微服务

*第一部分*介绍了微服务应用的基本功能。在深入开发 Codebreaker 应用之前，您将探索.NET Aspire——一种新的适用于服务构建的云就绪堆栈。本节涵盖了该技术的提供内容、基本功能、对 Microsoft Azure 的介绍以及 Codebreaker 应用组成组件的概述。随后，您将使用 ASP.NET Core 最小 API 进行编码，通过**Entity Framework** (**EF**) Core 编写与关系型和非关系型数据库的数据交互代码，利用 Azure Cosmos DB 和 SQL Server，并生成客户端库以访问 REST 服务。一种方法涉及利用 HTTP 客户端工厂，另一种则采用 Microsoft Kioata。

本节中的每一章都提供了一个功能应用，该应用随着后续章节的展开而发展，从而增强学习体验。

本部分包含以下章节：

+   *第一章*，*.NET Aspire 和微服务简介*

+   *第二章*，*最小化 API – 创建 REST 服务*

+   *第三章**，将数据写入关系型和非关系型数据库*

+   *第四章**，为客户端应用程序创建库*
