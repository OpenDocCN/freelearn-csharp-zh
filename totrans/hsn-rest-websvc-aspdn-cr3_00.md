# 前言

.NET Core 为所有 Microsoft 生态系统消费者带来了一股清新的空气。旧的 ASP.NET 和 .NET 框架都有着悠久的历史。多年来，.NET 框架的增长和长期支持限制导致了奇特的实现和难以维护的 Web 应用程序和 Web 服务。

此外，.NET 框架与 Windows 操作系统之间的紧密依赖导致了在云计算技术世界中存在重大限制。

.NET Core 和 ASP.NET Core 都旨在不断发展。它们使用包括开源、社区导向的方法以及持续改进概念在内的抽象软件开发实践。如今，ASP.NET Core 比以往任何时候都更加灵活、快速和强大。开发者面临的 .NET 框架的所有限制都已消除，并从头开始重写。因此，现在可以在每个平台上无限制地运行 .NET Core。该框架直接与 .NET Core 应用程序一起发货。因此，可以使用容器化方法运行它。文档、问题和路线图都可在 GitHub 上找到。现在，Microsoft 默认采用开源的工作方式。

自从旧 DNX 运行时的首次发布以来，我就对 .NET Core 感兴趣。这本书将向您介绍 ASP.NET Core 的强大功能以及如何利用其力量和灵活性来运行 Web 服务。此外，本书旨在消除人们对 .NET 生态系统的偏见，并寻求提高 .NET Core 的采用率。

# 这本书面向谁

本书旨在为那些想学习如何使用 ASP.NET Core 构建 RESTful Web 服务的人。为了最大限度地利用书中包含的代码示例，您应该具备基本的 C# 知识。

# 这本书涵盖的内容

[第 1 章](b3e95a60-c4fb-491e-ad7e-a2213f70a63b.xhtml)，*REST 101 和 ASP.NET Core 入门*，解释了 RESTful API 的基本原理以及它们在构建应用程序时如何有用。

[第 2 章](6127f023-703b-42e3-a76e-b70a7b110b90.xhtml)，*ASP.NET Core 概述*，展示了 `.csproj` 文件的基本组件。它说明了项目的主要组件：`Startup` 类和 `Program.cs` 文件。

[第 3 章](77d18c37-0c9d-4b2b-82f5-74fd874c0e0f.xhtml)，*与中间件管道一起工作*，探讨了中间件，这是 ASP.NET Core 的核心部分。本章将带您了解中间件管道，并解释它如何处理请求并根据它们初始化不同的服务。此外，本章还涵盖了 ASP.NET Core 提供的不同内置中间件以及如何构建自定义中间件。

[第4章](54bd7784-d757-4cbc-91d4-5362ca3a60de.xhtml)，*依赖注入系统*，介绍了依赖注入原理以及依赖注入背后的概念。它展示了如何使用依赖注入初始化应用程序内的组件和选项，以及如何在控制器内使用它们。

[第5章](deede298-fc20-4523-afa6-02ed2c0592fd.xhtml)，*ASP.NET Core中的Web服务堆栈*，描述了如何在ASP.NET Core中创建Web服务堆栈。它深入探讨了控制器、操作方法、操作结果、模型绑定和模型验证等问题。

[第6章](88a48d0b-32c6-4adb-b907-ecd8365a3659.xhtml)，*路由系统*，深入探讨了处理HTTP请求的路由系统。本章展示了如何处理ASP.NET Core的默认路由系统。

[第7章](13fd7d18-3ebe-4f60-89ff-4666d1c9671a.xhtml)，*过滤器管道*，涵盖了ASP.NET Core中的另一个重要主题：过滤器。过滤器是我们服务中实现横切实现的关键组件。本章介绍了它们；展示了如何实现我们的过滤器，并探讨了某些具体用例。

[第8章](84b281bd-11a2-4703-81ae-ca080e2a267a.xhtml)，*构建数据访问层*，介绍了领域模型部分。主要话题涉及如何构建领域模型以及如何使用**对象关系映射**（**ORM**）来访问数据。

[第9章](f8ac60e1-e948-4435-b804-e3e4ff305510.xhtml)，*实现领域逻辑*，描述了将逻辑与其他应用程序组件隔离的调解器模式方法。调解器模式是处理和管理我们逻辑的一种方式。

[第10章](266d98e8-df54-4024-aefb-b5871b3b35fa.xhtml)，*实现RESTful HTTP层*，解释了如何从中介中检索数据并在我们的控制器中使用它。

[第11章](3e096108-790a-40ab-a663-1d11f40361ec.xhtml)，*构建API的高级概念*，介绍了ASP.NET Core中构建API的一些高级概念。本章将涵盖关于资源软删除的话题，并介绍了在ASP.NET Core中处理异步代码的一些良好实践。

[第12章](d3fd0efc-cb05-42bb-8e06-4fd3a7ca9300.xhtml)，*服务的容器化*，为您快速介绍了容器以及它们在本地沙盒环境中运行应用程序时的有用性。

[第13章](8f70e186-50f1-4d55-99e6-026c73d5bb96.xhtml)，*服务生态系统模式*，专注于当多个服务属于同一生态系统时的相关模式。

[第14章](c8a85bf6-b5a8-497e-b1ab-61fbb1d2ae81.xhtml)，*使用.NET Core实现工作服务*，专注于.NET Core的新工作模板。工作服务提供了一种实现小型服务或守护进程的方式，这些服务或守护进程可以用于执行后台操作。

[第15章](3ede9648-ed9f-48cb-b9b2-6db5b4d4e882.xhtml)，*保护您的服务*，讨论了保护服务或API的方法。除此之外，它还涵盖了包括**安全套接字层**（**SSL**）、**跨源资源共享**（**CORS**）和身份验证在内的概念。

[第16章](26d8706c-4d06-4cb5-9a59-bd7b7e699fe2.xhtml)，*缓存Web服务响应*，涵盖了ASP.NET Core提供的所有缓存选项。

[第17章](0a303374-2436-4117-8398-17cba958a52a.xhtml)，*日志记录、监控和健康检查*，展示了日志记录和监控应用程序的一些最佳实践。

[第18章](a5b9222f-010b-4238-a475-c35f0fca913b.xhtml)，*在Azure上部署服务*，展示了如何在云中托管Web服务的示例。

[第19章](b361c850-961b-4b19-b88f-f17605c72742.xhtml)，*使用Swagger记录您的API*，介绍了OpenAPI标准以及如何在ASP.NET Core应用程序中实现它。

[第20章](1ee1c0a2-137b-4aa2-8ce1-91b8ad0e23f8.xhtml)，*使用Postman测试服务*，展示了如何使用Postman测试Web服务。

# 要充分利用这本书

本书假设您对C#（或类似面向对象的编程语言，如Java）有一定的了解，并且您在构建Web应用程序方面有一些经验。

这本书不需要任何特定的工具，但您需要在您的机器上安装.NET Core（[https://dotnet.microsoft.com/download](https://dotnet.microsoft.com/download)）。我强烈建议您至少安装一个代码编辑器，例如Visual Studio Code，可能还带有OmniSharp，或者一个**集成开发环境**（**IDE**）如Visual Studio（在Windows或Mac上），或者Rider IDE（在Windows、Mac或Linux上）。

我使用Rider IDE和Bash在macOS X上编写了所有示例。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的账户下载这本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)上登录或注册。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并按照屏幕上的说明操作。

文件下载完成后，请确保使用最新版本的以下软件解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书代码包也托管在GitHub上，网址为[https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3](https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3)。如果代码有更新，它将在现有的GitHub仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包可供在 **[https://github.com/PacktPublishing/](https://github.com/PacktPublishing/)** 上获取。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的PDF文件。您可以从这里下载：[https://static.packt-cdn.com/downloads/9781789537611_ColorImages.pdf](https://static.packt-cdn.com/downloads/9781789537611_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟URL、用户输入和Twitter昵称。以下是一个例子：“头部部分告诉客户端应该使用特定的 `content-type` 处理响应；在这种情况下，`application/json`。”

代码块设置如下：

[PRE0]

当我们希望您注意代码块中的特定部分时，相关的行或项目会设置为粗体：

[PRE1]

任何命令行输入或输出都按照以下方式编写：

[PRE2]

**粗体**：表示新术语、重要词汇或您在屏幕上看到的词汇。例如，菜单或对话框中的文字会像这样显示。以下是一个例子：“如您所见，它正在运行 **Hello, World!**”

警告或重要注意事项看起来像这样。

小技巧和技巧看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并给我们发送电子邮件至 `customercare@packtpub.com`。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们非常感谢您能向我们报告。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，我们将非常感谢您提供位置地址或网站名称。请通过链接材料与我们联系至 `copyright@packt.com`。

**如果您想成为一名作者**：如果您在某个领域有专业知识，并且对撰写或参与一本书籍感兴趣，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用了这本书，为什么不在这本书购买的网站上留下评论呢？潜在读者可以看到并使用您的客观意见来做出购买决定，Packt可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

想了解更多关于Packt的信息，请访问 [packt.com](http://www.packt.com/).
