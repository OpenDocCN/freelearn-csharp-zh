# 前言

.NET Core 为所有 Microsoft 生态系统消费者带来了一股清新的空气。旧的 ASP.NET 和 .NET 框架都有着悠久的历史。多年来，.NET 框架的增长和长期支持限制导致了奇特的实现和难以维护的 Web 应用程序和 Web 服务。

此外，.NET 框架与 Windows 操作系统之间的紧密依赖导致了在云计算技术世界中存在重大限制。

.NET Core 和 ASP.NET Core 都旨在不断发展。它们使用包括开源、社区导向的方法以及持续改进概念在内的抽象软件开发实践。如今，ASP.NET Core 比以往任何时候都更加灵活、快速和强大。开发者面临的 .NET 框架的所有限制都已消除，并从头开始重写。因此，现在可以在每个平台上无限制地运行 .NET Core。该框架直接与 .NET Core 应用程序一起发货。因此，可以使用容器化方法运行它。文档、问题和路线图都可在 GitHub 上找到。现在，Microsoft 默认采用开源的工作方式。

自从旧 DNX 运行时的首次发布以来，我就对 .NET Core 感兴趣。这本书将向您介绍 ASP.NET Core 的强大功能以及如何利用其力量和灵活性来运行 Web 服务。此外，本书旨在消除人们对 .NET 生态系统的偏见，并寻求提高 .NET Core 的采用率。

# 这本书面向谁

本书旨在为那些想学习如何使用 ASP.NET Core 构建 RESTful Web 服务的人。为了最大限度地利用书中包含的代码示例，您应该具备基本的 C# 知识。

# 这本书涵盖的内容

第一章，*REST 101 和 ASP.NET Core 入门*，解释了 RESTful API 的基本原理以及它们在构建应用程序时如何有用。

第二章，*ASP.NET Core 概述*，展示了 `.csproj` 文件的基本组件。它说明了项目的主要组件：`Startup` 类和 `Program.cs` 文件。

第三章，*与中间件管道一起工作*，探讨了中间件，这是 ASP.NET Core 的核心部分。本章将带您了解中间件管道，并解释它如何处理请求并根据它们初始化不同的服务。此外，本章还涵盖了 ASP.NET Core 提供的不同内置中间件以及如何构建自定义中间件。

第四章，*依赖注入系统*，介绍了依赖注入原理以及依赖注入背后的概念。它展示了如何使用依赖注入初始化应用程序内的组件和选项，以及如何在控制器内使用它们。

第五章，*ASP.NET Core 中的 Web 服务堆栈*，描述了如何在 ASP.NET Core 中创建 Web 服务堆栈。它深入探讨了控制器、操作方法、操作结果、模型绑定和模型验证等问题。

第六章，*路由系统*，深入探讨了处理 HTTP 请求的路由系统。本章展示了如何处理 ASP.NET Core 的默认路由系统。

第七章，*过滤器管道*，涵盖了 ASP.NET Core 中的另一个重要主题：过滤器。过滤器是我们服务中实现横切实现的关键组件。本章介绍了它们；展示了如何实现我们的过滤器，并探讨了某些具体用例。

第八章，*构建数据访问层*，介绍了领域模型部分。主要话题涉及如何构建领域模型以及如何使用**对象关系映射**（**ORM**）来访问数据。

第九章，*实现领域逻辑*，描述了将逻辑与其他应用程序组件隔离的调解器模式方法。调解器模式是处理和管理我们逻辑的一种方式。

第十章，*实现 RESTful HTTP 层*，解释了如何从中介中检索数据并在我们的控制器中使用它。

第十一章，*构建 API 的高级概念*，介绍了 ASP.NET Core 中构建 API 的一些高级概念。本章将涵盖关于资源软删除的话题，并介绍了在 ASP.NET Core 中处理异步代码的一些良好实践。

第十二章，*服务的容器化*，为您快速介绍了容器以及它们在本地沙盒环境中运行应用程序时的有用性。

第十三章，*服务生态系统模式*，专注于当多个服务属于同一生态系统时的相关模式。

第十四章，*使用.NET Core 实现工作服务*，专注于.NET Core 的新工作模板。工作服务提供了一种实现小型服务或守护进程的方式，这些服务或守护进程可以用于执行后台操作。

第十五章，*保护您的服务*，讨论了保护服务或 API 的方法。除此之外，它还涵盖了包括**安全套接字层**（**SSL**）、**跨源资源共享**（**CORS**）和身份验证在内的概念。

第十六章，*缓存 Web 服务响应*，涵盖了 ASP.NET Core 提供的所有缓存选项。

第十七章，*日志记录、监控和健康检查*，展示了日志记录和监控应用程序的一些最佳实践。

第十八章，*在 Azure 上部署服务*，展示了如何在云中托管 Web 服务的示例。

第十九章，*使用 Swagger 记录您的 API*，介绍了 OpenAPI 标准以及如何在 ASP.NET Core 应用程序中实现它。

第二十章，*使用 Postman 测试服务*，展示了如何使用 Postman 测试 Web 服务。

# 要充分利用这本书

本书假设您对 C#（或类似面向对象的编程语言，如 Java）有一定的了解，并且您在构建 Web 应用程序方面有一些经验。

这本书不需要任何特定的工具，但您需要在您的机器上安装.NET Core（[`dotnet.microsoft.com/download`](https://dotnet.microsoft.com/download)）。我强烈建议您至少安装一个代码编辑器，例如 Visual Studio Code，可能还带有 OmniSharp，或者一个**集成开发环境**（**IDE**）如 Visual Studio（在 Windows 或 Mac 上），或者 Rider IDE（在 Windows、Mac 或 Linux 上）。

我使用 Rider IDE 和 Bash 在 macOS X 上编写了所有示例。

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

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3`](https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包可供在 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)** 上获取。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`static.packt-cdn.com/downloads/9781789537611_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781789537611_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个例子：“头部部分告诉客户端应该使用特定的 `content-type` 处理响应；在这种情况下，`application/json`。”

代码块设置如下：

```cs
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFrameworks>netcoreapp3.0;net461</TargetFrameworks>
  </PropertyGroup>
</Project>

```

当我们希望您注意代码块中的特定部分时，相关的行或项目会设置为粗体：

```cs
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
 <TargetFrameworks>netcoreapp3.0;net461</TargetFrameworks>
  </PropertyGroup>
</Project>

```

任何命令行输入或输出都按照以下方式编写：

```cs
dotnet build
dotnet run
```

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

请留下评论。一旦您阅读并使用了这本书，为什么不在这本书购买的网站上留下评论呢？潜在读者可以看到并使用您的客观意见来做出购买决定，Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问 [packt.com](http://www.packt.com/).
