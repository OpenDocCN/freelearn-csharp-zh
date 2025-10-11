# 前言

ASP.NET Core 是微软提供的最强大的 Web 框架，它充满了使它更强大和有用的隐藏功能。

不应该让你的应用程序去匹配框架；你的框架应该能够完成你的应用程序真正需要的功能。通过这本书，你将学习如何找到可以旋转以获得框架最大化的隐藏螺丝。

使用 ASP.NET Core 开发的开发者将能够通过这本关于自定义 ASP.NET Core 的实用指南来运用他们的知识。本书提供了一种动手实践的方法，以及与其相关的实现方法和方法论，这将使你迅速投入工作并变得高效。

本书是默认 ASP.NET Core 行为的紧凑集合，这些行为你可能想要更改，以及如何一步步进行更改的详细说明。

到这本书的结尾，你将知道如何根据你个人的需求自定义 ASP.NET Core，从而获得一个优化的应用程序。

# ASP.NET Core 架构概述

要继续阅读下一章，你应该熟悉 ASP.NET Core 的基本架构及其组件。本书几乎涵盖了架构的所有组件。

下图显示了 ASP.NET Core 6.0 的基本架构概述。让我们快速浏览一下从底部到顶部层显示的组件：

![图片](img/Image79467.jpg)

在底部，是 **Host** 层。这是启动 web 服务器以及启动 ASP.NET Core 应用程序所需的所有内容的引导层，包括日志记录、配置和服务提供者。这一层创建了实际请求对象及其依赖项，这些依赖项在上述层中使用。

在 **Host** 层之上的是 **Middleware** 层。这一层与请求对象一起工作或对其进行操作。它将中间件附加到请求对象上。它执行中间件，如错误处理、HSTS 认证、CORS 等。

在那之上，是 **Routing** 层，它根据定义的路由模式将请求路由到端点。端点路由是 ASP.NET Core 3.1 中的新参与者，它将路由从上面的 UI 层中分离出来，以实现对不同端点的路由，包括 Blazor、gRPC 和 SignalR。提醒一下：在 ASP.NET Core 的早期版本中，路由是 MVC 层的一部分，而每个其他的 UI 层都需要实现自己的路由。

实际端点由第四层，即 UI 层提供，该层包含知名的 UI 框架 **Blazor**、**gRPC**、**SignalR** 和 **MVC**。作为 ASP.NET Core 开发者，你将在这里做大部分工作。

最后，在 **MVC** 之上，你会发现 **WebAPI** 和 **Razor Pages**。

# 本书涵盖了哪些内容？

本书不涵盖架构概述中提到的所有主题。本书主要涵盖托管层的主题，因为这是包含最多可能需要定制的功能的层。本书探讨了中间件和路由，以及 MVC 功能和一些额外的 WebAPI 主题，在这些主题中你可以施展一些魔法技巧。

每章开头，我们将指出该主题属于哪个级别。

# 未涵盖的内容及其原因？

本书不涵盖 Razor Pages、SignalR、gRPC 和 Blazor。

原因在于 gRPC 和 SignalR 已经非常专业化，实际上并不需要太多定制。Blazor 是 ASP.NET Core 家族的新成员，目前尚未得到广泛应用。此外，作者对 Blazor 的了解还不够深入，以至于不知道如何调整所有细节来定制它。Razor Pages 建立在 MVC 框架之上，因此对 MVC 的定制也适用于 Razor Pages。

# 本书面向的对象

本书面向的是使用 ASP.NET Core 的网络开发者，他们可能需要更改默认行为以完成任务。读者应具备 ASP.NET Core 和 C# 的基础知识，因为本书不涵盖这些技术的基础知识。读者还应熟悉 Visual Studio、Visual Studio Code 或任何支持 ASP.NET Core 和 C# 的其他代码编辑器。

# 本书涵盖的内容

[*第 1 章*](B17996_01_ePub.xhtml#_idTextAnchor019)，*自定义日志记录*，教你如何自定义日志行为以及如何添加自定义日志提供程序。

[*第 2 章*](B17996_02_ePub.xhtml#_idTextAnchor031)，*自定义应用程序配置*，帮助你了解如何使用不同的配置源并添加自定义配置提供程序。

[*第 3 章*](B17996_03_ePub.xhtml#_idTextAnchor049)，*自定义依赖注入*，教你了解 **依赖注入** (**DI**) 的工作原理以及如何使用不同的 DI 容器。

[*第 4 章*](B17996_04_ePub.xhtml#_idTextAnchor066)，*使用 Kestrel 配置和自定义 HTTPS*，探讨了不同的 HTTPS 配置方法。

[*第 5 章*](B17996_05_ePub.xhtml#_idTextAnchor077)，*配置 WebHostBuilder*，帮助你了解如何在托管层设置配置。

[*第 6 章*](B17996_06_ePub.xhtml#_idTextAnchor092)，*使用不同的托管模型*，教你了解不同平台上的不同托管类型。

[*第 7 章*](B17996_07_ePub.xhtml#_idTextAnchor111)，*使用 IHostedService 和 BackgroundService*，让你了解如何在后台执行任务。

[*第 8 章*](B17996_08_ePub.xhtml#_idTextAnchor124)，*编写自定义中间件*，处理 HTTP 上下文使用中间件。

[*第 9 章*](B17996_09_ePub.xhtml#_idTextAnchor143)，*使用端点路由*，帮助你了解如何使用新的路由来提供自定义端点。

[*第 10 章*](B17996_10_ePub.xhtml#_idTextAnchor156)，*自定义 ASP.NET Core 身份验证*，解释了如何扩展应用程序的用户属性，并帮助你更改身份 UI。

[*第11章*](B17996_11_ePub.xhtml#_idTextAnchor163), *配置身份管理*，帮助您管理您的用户及其角色。

[*第12章*](B17996_12_ePub.xhtml#_idTextAnchor172), *使用自定义输出格式化程序进行内容协商*，教您如何根据HTTP Accept头输出不同的内容类型。

[*第13章*](B17996_13_ePub.xhtml#_idTextAnchor186), *使用自定义ModelBinder管理输入*，帮助您创建具有不同类型内容的输入模型。

[*第14章*](B17996_14_ePub.xhtml#_idTextAnchor207), *创建自定义ActionFilter*，介绍了使用ActionFilter的面向方面编程。

[*第15章*](B17996_15_ePub.xhtml#_idTextAnchor218), *使用缓存*，帮助您使您的应用程序更快。

[*第16章*](B17996_16_ePub.xhtml#_idTextAnchor231), *创建自定义TagHelper*，使您能够通过创建TagHelper简化UI层。

# 要充分利用本书

读者应具备基本的ASP.NET Core和C#知识，以及Visual Studio、Visual Studio Code或任何支持ASP.NET Core和C#的其他代码编辑器。

![图片](img/B17996_Preface_Table1.jpg)

您应该在您的机器上安装最新的.NET 6.0 SDK。请访问[https://dotnet.microsoft.com/download/dotnet-core/](https://dotnet.microsoft.com/download/dotnet-core/)以获取最新版本。

随意使用您喜欢的任何支持ASP.NET Core和C#的代码编辑器。我们推荐使用Visual Studio Code（[https://code.visualstudio.com/](https://code.visualstudio.com/）），它适用于所有平台，并且本书的作者也在使用它。

本书中的所有项目都将使用控制台、命令提示符、shell或PowerShell创建。请随意使用您感到舒适的任何控制台。作者使用Windows命令提示符，托管在cmder shell中（[https://cmder.net/](https://cmder.net/））。我们不推荐使用Visual Studio创建项目，因为基本配置可能会更改，并且Web项目将启动在本书中描述的端口之外。

您是否卡在.NET Core 3.1或.NET 5.0上？如果您由于任何原因无法在您的机器上使用.NET 6.0，所有示例也都可以使用.NET Core 3.1和.NET 5.0。有些章节在.NET 6.0与.NET 5.0有差异时，会包含与.NET 5.0的比较。

**如果您正在使用本书的数字版，我们建议您亲自输入代码或从本书的GitHub仓库（下一节中有一个链接）获取代码。这样做将帮助您避免与代码复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从GitHub下载本书的示例代码文件[https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition](https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition)。如果代码有更新，它将在GitHub仓库中更新。如果代码有更新，它将在GitHub仓库中更新。

我们还有其他来自我们丰富的书籍和视频目录的代码包可供在[https://github.com/PacktPublishing/](https://github.com/PacktPublishing/)找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图和图表彩色图像的PDF文件。您可以从这里下载：[https://static.packt-cdn.com/downloads/9781803233604_ColorImages.pdf](https://static.packt-cdn.com/downloads/9781803233604_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟URL、用户输入和Twitter昵称。以下是一个示例：“您可以使用`ConfigureAppConfiguration`来配置应用程序配置。”

代码块按以下方式设置：

[PRE0]

当我们希望将您的注意力引到代码块的一个特定部分时，相关的行或项目将以粗体显示：

[PRE1]

任何命令行输入或输出都按以下方式编写：

[PRE2]

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词以**粗体**显示。以下是一个示例：“点击左上角的**注册**，您将看到以下页面。”

小贴士或重要提示

看起来是这样的。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请通过电子邮件发送至[customercare@packtpub.com](mailto:customercare@packtpub.com)，并在邮件主题中提及书名。

**勘误表**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问[www.packtpub.com/support/errata](https://www.packtpub.com/support/errata)并填写表格。

**盗版**：如果您在互联网上遇到我们作品的任何形式的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过[copyright@packt.com](mailto:copyright@packt.com)与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](https://authors.packtpub.com)。

# 分享您的想法

读完*Customizing ASP.NET Core 6.0*后，我们很乐意听听您的想法！请[点击此处直接进入此书的亚马逊评论页面](https://packt.link/r/1803233605)并分享您的反馈。

您的评论对我们和科技社区都很重要，并将帮助我们确保我们提供高质量的内容。
