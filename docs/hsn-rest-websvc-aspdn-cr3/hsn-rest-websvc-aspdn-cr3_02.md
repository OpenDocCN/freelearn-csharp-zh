# 第一章：REST 101 和 ASP.NET Core 入门

现在，几乎所有的应用程序都依赖于网络服务。其中许多使用 RESTful 方法操作。以资源为中心的方法和 REST 风格的简单性已成为行业标准。因此，了解 REST 工作方式背后的理论以及为什么它很重要是至关重要的。本章将向你介绍**表征状态转移**（**REST**）方法。我们将看到 REST 的定义以及如何识别符合 REST 的网络服务。我们还将介绍.NET Core 3.1 和 ASP.NET Core，这是由微软提供的开源、跨平台框架的最新版本。

总结来说，本章涵盖了以下主题：

+   REST 架构元素的概述

+   .NET 生态系统的简要介绍

+   为什么你应该选择.NET 来构建 RESTful 网络服务

到本章结束时，你将了解一些有用的工具和 IDE，你可以使用它们开始开发.NET Core。

本章将涵盖.NET Core 3.1 和 ASP.NET Core 的一些基本概念。你需要安装 Windows、Linux 或 macOS。设置过程将取决于你使用的操作系统。我们将探讨可以用于在.NET Core 中开发应用程序和网络服务的不同工具。

# REST

什么是 REST？**表征状态转移**（**REST**）通常被定义为一种软件架构风格，它对网络服务施加了一些约束。它确定了一组以资源为中心的规则，这些规则描述了分布式超媒体系统中约束的角色和交互，而不是关注组件的实现。尽管找到不使用 HTTP 的 REST 服务相当罕见，但定义并未提及这些主题，而是将 REST 描述为媒体和协议无关的。

以下定义可以通过一个示例进一步解释。考虑一个电子商务网站。当你浏览产品列表并点击其中一个时，浏览器将你的点击解释为对特定资源的请求；在这种情况下，你点击的产品详情。浏览器通过 HTTP 调用 URI，该 URI 对应于产品的详情，并使用 URI 请求特定的资源。这个过程在以下图中显示：

![图片](img/45c0c1b9-0d12-48e2-8f11-351aa8738e9a.png)

REST 的概念相当相似：客户端向服务器请求特定的资源，这使得它们可以导航并获得关于服务器上存储的其他资源的其他信息。

# 严格遵守 REST 的重要性

在我们讨论 REST 之前，我们应该了解在现代应用程序开发世界中网络服务的重要性。

典型的现代应用程序使用网络服务来获取和查询数据。任何开发产品或解决方案的公司都使用网络服务来交付和跟踪信息。这是因为很难复制所有客户使用您的应用程序所需的所有数据和每个行为。网络服务对于提供第三方客户端和服务的访问也非常有用。例如，考虑 Google Maps 或 Instagram 的 **应用程序编程接口**（**APIs**）：这些平台通过 HTTP 暴露信息，与其他公司和服务的共享。

由于其简单明了，REST 对网络服务架构的方法越来越受欢迎。与一些旧方法（如 **简单对象访问协议**（**SOAP**）或 **Windows Communication Foundation**（**WCF**））不同，REST 服务提供了一种查询数据的方式，无需在请求中添加任何开销。不同的信息可以通过不同的 URI 获取。

现在，比以往任何时候，我们的服务都需要准备好扩展，以便能够适应消费者。在一个充满不同技术和全球分布式的团队的世界里，我们应该能够调整我们的架构来解决复杂的商业挑战。HTTP 和 REST 帮助我们应对这些挑战。

# REST 要求

Roy Thomas Fielding 是 HTTP 协议和 REST 风格正式化的主要人物：他在自己的论文《架构风格和网络软件架构设计》（*Architectural Styles and the Design of Network-based Software Architectures*）中描述了它们（[`www.ics.uci.edu/~fielding/pubs/dissertation/top.htm`](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)）。此外，Fielding 的论文明确定义了 REST 系统的约束，如下所示：

+   统一接口

+   无状态

+   可缓存

+   客户端-服务器

+   分层系统

+   需求代码

除了需求代码之外，所有这些都是在我们将网络服务定义为 REST 兼容时必不可少的。为了解释这些概念，我们将使用纽约时报提供的 API：[`developer.nytimes.com/`](https://developer.nytimes.com/)。要使用这些 API，您需要通过预认证过程。您可以从以下链接获取 API 密钥：[`developer.nytimes.com/signup`](https://developer.nytimes.com/signup)。

# 统一接口

统一接口指的是客户端和服务器之间的分离。这很有优势，因为它意味着这两个系统是独立的。统一接口的第一个原则是它应该是基于资源的，这意味着我们应该开始以资源导向的方式思考。因此，我们系统中的每个对象或实体都是一个资源，每个资源都有一个 URI 唯一标识。此外，如果我们考虑 HTTP 协议，每个资源都以 XML 或 JSON 的形式呈现给客户端，以解耦客户端和服务器。

让我们使用《纽约时报》的 API 来了解这个主题。考虑以下 URI：

```cs
http://api.nytimes.com/svc/archive/v1/2018/6.json?api-key={your_api_key}
```

它包含了一些关于我们请求的资源的确切信息，包括以下内容：

+   我们从报纸数据的*存档*部分获取信息的事实

+   我们请求的具体月份；在这种情况下，2018 年 6 月

+   事实是资源将使用*JSON 格式*进行序列化

# 通过表示操纵资源

让我们看看《纽约时报》提供的另一个 API 请求：

```cs
https://api.nytimes.com/svc/books/v3/lists.json?api-key={your_api_key}&list=hardcover-fiction
```

这提供了一个基于类别的书籍列表；在这种情况下，这个类别是`精装小说`。让我们分析这个响应：

```cs
{
    "status": "OK",
    "copyright": "Copyright (c) 2018 The New York Times Company. All Rights 
      Reserved.",
    "num_results": 15,
    "last_modified": "2018-06-28T02:38:01-04:00",
    "results": [
        {
            "list_name": "Hardcover Fiction",
            "display_name": "Hardcover Fiction",
            "published_date": "2018-07-08",
            "isbns": [
                {
                    "isbn10": "1780898401",
                    "isbn13": "9780316412698"
                }
            ],
            "book_details": [
                {
                    "title": "THE PRESIDENT IS MISSING",
                    "contributor": "by Bill Clinton and James Patterson",
                    "author": "Bill Clinton and James Patterson",
                    "contributor_note": "",
                    "price": 0,
                    "age_group": "",
                    "publisher": "Little, Brown and Knopf"
                }
            ],
            "reviews": [
                {
                    "book_review_link": "",
                    "first_chapter_link": ""
                }
            ]
        }
        ....
    ]
}
```

如您所见，这是资源的清晰表示。客户端拥有所有必要的信息来使用 API（如果 API 允许）处理和修改数据。

# 自描述消息

让我们看看以下调用的响应：

```cs
https://api.nytimes.com/svc/books/v3/lists.json?api-key={your_api_key}&list=hardcover-fiction
```

如我们之前提到的，响应是数据的表示，无论是存储在数据源中还是从另一个系统获取。在任何情况下，一些信息是缺失的：客户端如何知道响应的格式？这类信息通常写在响应头部。例如，以下是之前请求的所有头部：

```cs
accept-ranges: bytes
access-control-allow-headers:Accept, Content-Type, X-Forwarded-For, X-Prototype-Version, X-Requested-With
access-control-allow-methods: GET, OPTIONS
access-control-allow-origin: *
access-control-expose-headers: Content-Length, X-JSON
age: 0
connection: keep-alive
content-length: 14384
content-type: application/json; charset=UTF-8
date: Tue, 03 Jul 2018 12:47:08 GMT
server: Apache/2.2.15 (CentOS)
vary: Origin
via: kong/0.9.5
x-cache: MISS
x-kong-proxy-latency: 4
x-kong-upstream-latency: 29
x-ratelimit-limit-day: 1000
x-ratelimit-limit-second: 5
x-ratelimit-remaining-day: 988
x-ratelimit-remaining-second: 4
x-varnish: 63737329
```

*头部*部分告知客户端响应应使用特定的`content-type`*；*进行处理，在这种情况下，`application/json`*.* 它还提供了有关编码、缓存和相关元信息的信息，例如包含对象在代理缓存中存在时间的`age`头部。

# 超媒体作为应用状态引擎

服务通常通过正文内容、响应代码和响应头部将状态传递给客户端。最重要的是，*超媒体驱动的服务（HATEOAS）*在其响应中包含其他资源的 URI。以下示例描述了 HATEOAS 的概念：

```cs
{
    "links": {
        "self": { "href": "http://example.com/people" },
        "item": [
            { "href": "http://example.com/people/1", "title": "Kendrick 
             West" },
            { "href": "http://example.com/people/2", "title": "Anderson 
             Rocky" }
        ]
    },
}
```

之前的响应提供了一个人员列表，以及指定每个人员详细信息的 URI。因此，客户端知道正确的请求 URI，以便获取有关每个资源的详细信息。

# 无状态

无状态是 REST 服务的一个关键特性。确实，HTTP 作为一个无状态协议，一旦通信结束，就不会跟踪客户端和服务器之间连接的所有信息。

无状态协议迫使客户端每次需要从服务器获取信息时，都必须提供所有必要的信息。让我们看看之前的一个 URI：

```cs
https://api.nytimes.com/svc/books/v3/lists.json?api-key={your_api_key}&list=hardcover-fiction
```

客户端必须在每个请求中发送*API 密钥*以供服务器进行身份验证。此外，它必须存储 API 密钥信息。

如果我们希望利用 REST 服务，无状态性非常重要。如今，随着高度分布式系统的兴起，处理有状态服务变得困难，因为这需要在不同的服务器上管理和复制状态。无状态方法有助于将状态管理委托给客户端。

# 客户端-服务器分离

REST 服务的首要目标是解耦服务器和客户端。这非常重要，因为它有助于保持每个客户端应用程序独特的业务逻辑和数据存储。应用程序通常分布在众多不同的客户端上，包括 Web、智能手机、智能电视和物联网。REST 方法帮助我们防止在客户端之间复制逻辑。这意味着客户端没有任何业务逻辑或存储，服务器也不处理用户界面或表示层。

# 层次化系统

层次化系统的概念与我们的应用程序基础设施的结构密切相关。RESTful 服务允许松散耦合的方法，因为信息是通过协议传输的——在大多数情况下，是通过 HTTP——并且每个服务器都有一个单一的高级目的。代理服务器、Web 服务器和数据库服务器通常是隔离的，它们在我们的功能中覆盖一个目的，如果你有一个提供所有必需功能的单一服务器，那么维护和扩展通常很困难。

# 理查德森成熟度模型

**理查德森成熟度模型**是由伦纳德·理查德森开发的模型，其目的是通过提供一些一般性标准来衡量 API 的成熟度。该模型有四个分类步骤，从**级别 0**到**级别 3**。最高级别对应于更合规的服务。这个模型不仅仅是为了理论目的；它还帮助我们理解一些推荐的 Web 服务开发方法。让我们来看看这些不同级别的概述。以下图表显示了理查德森成熟度模型中各级别的结构：

![图片](img/0f85cdef-13d5-45cc-a224-12d1f075e2ae.png)

当一个通用服务在表面上使用通用协议（在 Web 服务的情况下，这是 HTTP）时，它处于**级别 0：普通旧 XML 的沼泽**。一个例子是重量级的 SOAP Web 服务。SOAP 实现只使用一个 URI 和一个 HTTP 动词，并且它们在每个请求消息中包裹一个巨大的信封。

正如我们之前提到的，从资源的角度思考是理解和设计 API 的最佳方式。因此，处于**级别 1**的通用服务使用与不同资源相关联的多个 URI。例如，如果我们考虑一个普通商店的 API，我们可以通过调用以下示例 URI 来获取产品类别的完整列表：

```cs
GET https://api.mystore.com/v1/categories 
```

同时，我们可以通过调用以下 URI 来获取单个类别的详细信息：

```cs
GET https://api.mystore.com/v1/categories/{category_id}
```

另一方面，我们可以通过调用以下 URI 来获取与单个类别相关的产品列表：

```cs
GET https://api.mystore.com/v1/categories/{category_id}/products
```

如您所见，我们可以通过调用不同的 URI 来获取不同的信息。没有封装，所有请求的信息都包含在 URI 中。

第二级，与 HTTP 动词相关，介绍了使用 HTTP 动词来增强请求上传输的信息。让我们以前一个请求 URI 为例：

```cs
https://api.mystore.com/v1/categories/{category_id}/products
```

这可以产生不同的结果，具体取决于 HTTP 动词。以下表格显示了各种 HTTP 动词的含义：

| **HTTP 动词** | **执行的操作** | **示例** |
| --- | --- | --- |
| `GET` | 获取有关资源的信息 |

```cs
GET /v1/categories/ HTTP/1.1
Host: api.mystore.com
Content-Type: application/json
```

|

| `POST` | 创建与资源相关的新项目 |
| --- | --- |

```cs
POST /v1/categories/ HTTP/1.1
Host: api.mystore.com
Content-Type: application/json

{
 "categoryId": 1,
 "categoryDescription": "Vegetables"
}
```

|

| `PUT` | 替换与资源相关的项目 |
| --- | --- |

```cs
PUT /v1/categories/1 HTTP/1.1
Host: api.mystore.com
Content-Type: application/json

{
  "categoryId": 1,
  "categoryDescription": "Fruits and Vegetables"
}
```

|

| `PATCH` | 更新与资源相关的项目 |
| --- | --- |

```cs
PUT /v1/categories/1 HTTP/1.1
Host: api.mystore.com
Content-Type: application/json

{
  "categoryDescription": "Fruits and Vegetables"
}
```

|

| `DELETE` | 删除与资源相关的项目 |
| --- | --- |

```cs
DELETE /v1/categories/1 HTTP/1.1
Host: api.mystore.com
Content-Type: application/json
```

|

不同的 HTTP 动词对应着不同的数据操作。与不使用任何 HTTP 规范来传递信息的**0 级**服务相反，**2 级**服务利用 HTTP 规范尽可能多地传递信息。最终，具有**3 级**服务的系统实现了 HATEOAS 的概念。正如我们在上一节中讨论的，HATEOAS 在其响应中提供了资源的 URI。这种方法的明显优势是客户端不需要任何信息就可以导航 Web 服务资源。最重要的是，如果我们的 Web 服务添加了资源的 URI，客户端立即就拥有了他们所需的所有信息。

# 介绍 ASP.NET Core

在撰写本书时，.NET Core 3.1 是由 Microsoft 和社区支持的框架的**LTS**（长期支持）版本。ASP.NET Core 是一个高度模块化的 Web 框架，它运行在 .NET Core 平台上：它可以用来开发各种 Web 解决方案，例如 Web 应用程序、Web 程序集客户端应用程序和 Web API 项目。

要了解 ASP.NET Core 的一些基本概念，我们需要理解 ASP.NET Core 中实现的**模型视图控制器**（MVC）模式。

MVC 通过将实现分组到三个不同的区域来分离我们的 Web 应用程序。在 Web 环境中，起点通常是客户端或通用用户发起的 Web 请求。请求通过中间件管道，然后控制器处理它。控制器还执行一些逻辑操作并填充我们的 *模型*。

模型是应用程序状态的表示。当它与视图相关联时，它被称为**视图模型**。一旦模型被填充，控制器根据请求返回一个特定的视图。视图的目的是通过 HTML 页面展示数据。

在 Web API 堆栈的情况下，这是在 ASP.NET Core 中构建 Web 服务的典型方式，过程与视图部分相同。而不是这个，控制器在响应中将模型序列化。

要了解 ASP.NET Core 如何帮助开发者构建 Web 服务，让我们回顾一下 ASP.NET 框架的历史。

# ASP.NET 的演变

ASP.NET 的第一个版本于 2002 年发布，当时微软决定投资 Web 开发。他们发布了 ASP.NET Web Forms，这是一个我们可以用来构建 Web 界面的 UI 组件集合。这种方法的核心思想是提供一个非常高级的抽象工具，可以生成 Web 的 GUI。提供这种级别的抽象是一个好主意，因为开发者对 Web 不熟悉。然而，ASP.NET Web Forms 带来了很多缺点。首先，开发者对 HTML 的控制有限，组件必须将信息存储在 *视图状态* 中，这些状态在客户端和服务器之间传输和更新。此外，组件没有正确分离，开发者倾向于将表示代码与业务逻辑代码混合。

为了提升开发 Web 应用程序的经验，微软在 2007 年宣布了 ASP.NET MVC 的到来。这个新的开发平台运行在 ASP.NET 框架上，并采用了其他实现了 MVC 模式的开发平台的概念，例如 Ruby on Rails。ASP.NET MVC 框架仍然存在一些弱点。它建立在 ASP.NET 之上，这意味着它必须保持与旧的网络表单和网络服务框架（如 WCF）的向后兼容性。此外，它只能在 Windows 服务器上运行，与 **Internet Information Services**（**IIS**）结合使用。

微软最新开发的 Web 框架是 ASP.NET Core。它运行在 .NET Core 上，这是一个跨平台和开源的平台。通过 ASP.NET Core，微软选择了发布一个没有从前一个版本的 ASP.NET 派生的任何向后兼容组件的新轻量级框架。

# 新的 .NET 生态系统

让我们概述一下 .NET 生态系统，以了解作为 ASP.NET Core 基础的不同框架。这里提供的一些信息可能听起来很显然，但澄清不同运行时和框架之间的差异是至关重要的：

![图片](img/7274e7a2-f816-46bc-a757-727e6661ce84.png)

第一个块与 **桌面包** 相关，它为从 **.NET Core 3.0** 开始的桌面应用程序开发提供了工具。在这本书中，我们不会使用这些工具，因为它们严格与桌面开发相关。

第二部分与.NET Core 的跨平台部分相关。这组工具允许开发者构建**WEB**、**DATA**和**AI/ML**领域的应用程序。**WEB**部分指的是 ASP.NET Core，它是随.NET Core 一起提供的库集合。通常，ASP.NET Core 会与**DATA**访问部分结合使用。在本书的后面部分，我们将看到如何使用**EF Core**来访问数据库层。最后，**AI/ML**部分，本书将不会讨论，在机器学习领域提供了有用的工具。在前面的图例底部，我们有一个共同的层，即**.NET STANDARD**。它允许开发者构建第三方库，这些库可以被**Desktop Packs**以及**WEB**、**DATA**和**AI/ML**部分使用。

总之，新的.NET 生态系统可以被任何开发者使用，包括云开发者、Web 开发者和桌面开发者。正如我们之前提到的，它可以在任何地方和任何平台上运行。本书中的所有示例都将基于.NET Core。

# .NET STANDARD

**.NET STANDARD**是与.NET Core 一起引入的。**.NET STANDARD**的目标是为.NET Core 和.NET Framework 提供一个共同的 API 界面。它作为我们.NET Framework 和.NET Core 应用程序的唯一**基础类库**（**BCL**）工作。.NET Standard 2.0 的发布引入了 32,000 个兼容 API，并支持以下框架版本：

+   .NET Framework 4.6.1 +

+   .NET Core 2.0 +

+   Mono 5.4 +

最近，微软推出了.NET Standard 2.1。这个新版本提供了作为.NET Core 3.0 开源开发一部分引入的新 API。从 3.0 版本开始，.NET Standard 2.1 将成为.NET Core 新版本和其他即将到来的框架版本（如 Mono）的共同点。

您可能出于不同的原因选择.NET Standard：

+   要构建与.NET Core 和.NET Framework 都兼容的第三方库。在这种情况下，您的包将针对.NET Standard 2.0。最终，如果您想使用我们之前描述的新优化的 API，可以使用多目标来同时针对.NET Standard 2.0 和.NET Standard 2.1。

+   通过将逻辑隔离在.NET Standard 项目上来逐步迁移您的.NET Framework 代码库。

例如，考虑一个由不同版本的.NET 使用的类库项目。该库可能运行在.NET Core 或.NET Framework 上。为了避免维护陷阱，库包可以编译为多个.NET Standard 版本，并且可以被*.NET Core*和*.NET Framework*解决方案*使用*。

# 为什么使用 ASP.NET Core 来构建 RESTful Web 服务？

有大量的 Web 框架可供开发者构建 RESTful Web 服务。其中一个这样的框架是建立在 .NET Core 3.1 上的 ASP.NET Core。.NET Core 3.1 提供了一种新的、轻量级、跨平台和开源的方式来构建 Web 应用程序。最重要的是，它被设计为云就绪：与 .NET Framework 不同，该框架不再是服务器的一部分；相反，它与应用程序一起分发。

另一个关键点是 .NET Core 保持 *高度模块化*，遵循 Unix 哲学，并允许你在定制应用程序中仅使用所需的部分。ASP.NET Core 还为 Web 应用程序和 Web 服务引入了两种新的托管解决方案：

+   **Kestrel**：ASP.NET Core 的默认 HTTP 服务器。它支持 HTTPS 和 WebSockets，并在 Windows、Linux 和 macOS 上运行。*Kestrel* 通常与 *反向代理* 结合使用，例如 NGINX、IIS 或 Apache。

+   **HTTP.sys**：一个仅适用于 Windows 的 HTTP 服务器，可以用作 Windows 上 Kestrel 的替代品。

ASP.NET Core 和 .NET Core 是由微软和社区共同开发的，它们是 *开源项目*。在 ASP.NET Core 的情况下，开源不仅仅是一个口号；所有功能都是由社区驱动的，并且 ASP.NET 团队每周在 YouTube 上发布社区站立会议视频，其中讨论路线图、截止日期和问题。所有 .NET Core 代码都可以在以下链接的 GitHub 上找到：

+   **.NET Core**：.NET Core 的存储库 ([`github.com/dotnet/core`](https://github.com/dotnet/core))

+   **ASP.NET Core**：包含所有 ASP.NET Core 项目的引用 ([`github.com/aspnet/AspNetCore`](https://github.com/aspnet/AspNetCore))

所有存储库通常都附带路线图和一些贡献指南。你可以打开问题并为代码库做出贡献。微软还成立了 .NET 基金会，这是一个独立的组织，旨在促进围绕 .NET 生态系统的开放开发和协作。

ASP.NET Core 团队也专注于框架的性能。所有基准测试结果都可以在 GitHub 上找到：[`github.com/aspnet/benchmarks`](https://github.com/aspnet/benchmarks)。

# 准备你的开发环境

在本节中，我们将向您展示如何设置您的开发环境，以便您可以使用 ASP.NET Core 开发 Web 服务。正如我们之前提到的，.NET Core 是跨平台的，因此它可以在最常用的操作系统上运行。我们还将探讨如何与 .NET Core CLI 交互，这是构建、运行、开发和测试我们的服务的起点。

首先，让我们从 [`www.microsoft.com/net/download/`](https://www.microsoft.com/net/download/) 下载 .NET Core 3.1 开始。在我们的案例中，我们将安装 SDK 版本，它包含我们开发环境所需的所有组件，包括 ASP.NET Core。

# .NET Core CLI

与.NET 框架不同，.NET Core 提供了一个易于使用的命令行界面（CLI），它暴露了我们用于构建应用程序和服务的所有必要功能。一旦.NET Core 安装到您的机器上，运行`dotnet --help`命令。您将看到以下结果：

```cs
.NET Core SDK (3.1.100)
Usage: dotnet [runtime-options] [path-to-application] [arguments]

Execute a .NET Core application.

runtime-options:
  --additionalprobingpath <path> Path containing probing policy and
    assemblies to probe for.
  --additional-deps <path> Path to additional deps.json file.
  --fx-version <version> Version of the installed Shared Framework to 
    use to run the application.
  --roll-forward <setting> Roll forward to framework version (LatestPatch, 
    Minor, LatestMinor, Major, LatestMajor, Disable).

path-to-application:
  The path to an application .dll file to execute.

Usage: dotnet [sdk-options] [command] [command-options] [arguments]

Execute a .NET Core SDK command.

sdk-options:
  -d|--diagnostics Enable diagnostic output.
  -h|--help Show command line help.
  --info Display .NET Core information.
  --list-runtimes Display the installed runtimes.
  --list-sdks Display the installed SDKs.
  --version Display .NET Core SDK version in use.

SDK commands:
  add Add a package or reference to a .NET project.
  build Build a .NET project.
  build-server Interact with servers started by a build.
  clean Clean build outputs of a .NET project.
  help Show command line help.
  list List project references of a .NET project.
  migrate Migrate a project.json project to an MSBuild project.
  msbuild Run Microsoft Build Engine (MSBuild) commands.
  new Create a new .NET project or file.
  nuget Provides additional NuGet commands.
  pack Create a NuGet package.
  publish Publish a .NET project for deployment.
  remove Remove a package or reference from a .NET project.
  restore Restore dependencies specified in a .NET project.
  run Build and run a .NET project output.
  sln Modify Visual Studio solution files.
  store Store the specified assemblies in the runtime package store.
  test Run unit tests using the test runner specified in a .NET project.
  tool Install or manage tools that extend the .NET experience.
  vstest Run Microsoft Test Engine (VSTest) commands.

Additional commands from bundled tools:
  dev-certs Create and manage development certificates.
  fsi Start F# Interactive / execute F# scripts.
  sql-cache SQL Server cache command-line tools.
  user-secrets Manage development user secrets.
  watch Start a file watcher that runs a command when files change.

Run 'dotnet [command] --help' for more information on a command.
```

首先要注意的是.NET Core 的版本，即.NET Core 的版本，也就是`.NET Core SDK (3.1.100)`，后面跟着一个**软件开发工具包（SDK**）命令列表。这包含了在开发阶段通常执行的命令，如`dotnet build`、`dotnet restore`和`dotnet run`。这些用于构建我们的项目、恢复 NuGet 依赖项以及运行我们的项目。另一个相关的部分是*附加工具*，它包含了我们将需要的所有第三方命令行界面（CLI）包，例如 EF Core。实际上，.NET Core CLI 允许您通过添加特定工具的 NuGet 包来扩展其功能。

# ASP.NET Core 中的 IDE 和开发工具

.NET Core CLI 是构建在 IDE、代码编辑器和**持续集成（CI**）工具等高级工具之上的基础。尽管.NET Core 是一个跨平台框架，但仍有各种工具可用于在不同平台上构建 Web 应用程序和服务。以下表格总结了可用于构建 ASP.NET Core 的不同 IDE 和编辑器：

| **软件** | **Windows** | **Linux** | **macOS X** |
| --- | --- | --- | --- |
| Visual Studio 2019（社区版、专业版和企业版） | 支持 | 不支持 | 支持 |
| Visual Studio Code 和 OmniSharp | 支持 | 支持 | 支持 |
| Rider | 支持 | 支持 | 支持 |
| Visual Studio for Mac | 不支持 | 不支持 | 支持 |

如您所见，针对不同的平台，可以使用不同的集成开发环境（IDE）和代码编辑器。您所做的选择通常取决于不同的因素。让我们来看看不同编辑器的概述：

+   **Visual Studio 2019（社区版、专业版和企业版）**：对于已经在.NET 生态系统上开发过的人来说，这是一个众所周知的编辑器。该产品的社区版完全免费，您可以在[`visualstudio.microsoft.com/it/downloads/`](https://visualstudio.microsoft.com/it/downloads/)找到它。如果您想在 Windows 上开始构建，Visual Studio 2019 是最舒适的选择。

+   **Visual Studio Code 和 OmniSharp**：由 Microsoft 和社区提供支持的一个流行且开源的编辑器。它是跨平台的，基于 Electron 构建。OmniSharp 是 Visual Studio Code 和其他代码编辑器的一个有用的第三方包，它为.NET Core 项目提供了一些集成。它还提供了一个智能感知（IntelliSense）功能。

+   **Rider**：由 JetBrains 提供支持，基于 IntelliJ 平台和 ReSharper 的新 IDE。它与所有平台兼容，但不是免费的。我在大型项目中尝试过它，它运行良好，主要因为它提供了开箱即用的 ReSharper 集成。

+   **Visual Studio for Mac**：由微软推出的一款新 IDE。它仅兼容 macOS，并提供了一些我们可以用于在.NET Core 生态系统中编写 C#或 F#代码的功能。这个 IDE 仍处于早期阶段，但它拥有许多高级功能。

总之，Visual Studio 2019、Rider 和 Visual Studio for Mac 等工具与.NET Core 结合使用时，提供了极佳的体验。另一方面，Visual Studio Code 是最轻便且最快的编辑器。在接下来的章节和代码演示中，我将使用.NET Core CLI 在不同的操作系统上重现相同的步骤。

# 摘要

在本章中，我们通过考虑一些具体的例子，对 REST 风格进行了概述。我们还了解了一些.NET 生态系统的基本概念，包括其结构以及为什么如果我们想构建网络服务，ASP.NET Core 是一个极佳的选择。我们还概述了.NET Core CLI 以及与.NET 生态系统相关的 IDE 和代码编辑器。

本章中我们讨论的主题为我们提供了对 REST 的含义以及为什么在开发网络服务时遵循此类原则很重要的良好理解。此外，我们还探讨了在本地环境中设置.NET Core 的基本原则。

下一章将专注于 ASP.NET Core 和 ASP.NET Core MVC。你将学习如何使用.NET CLI 设置项目，并探索 ASP.NET Core 的一些基本概念。
