# 前言

本书旨在帮助您学习 ASP.NET Core MVC 的基础知识，并将这些知识应用于使用 ASP.NET Core 构建应用程序。本书还旨在为想要学习 ASP.NET MVC 的初学者提供坚实的指南。具体来说，本书将涵盖以下主题：

+   ASP.NET Core MVC 的基本和目标

+   ASP.NET Core 的哲学（关注点分离、约定优于配置）

+   ASP.NET Core MVC 的组件——控制器、模型和视图

+   使用 Entity Framework 与数据库交互

+   在客户端和服务器端验证用户的输入

+   使用 Bootstrap 为应用程序提供一次改头换面的机会

+   利用 ASP.NET Core MVC 提供的不同部署选项

# 本书涵盖的内容

第一章，*ASP.NET Core 简介*，涵盖了 ASP.NET MVC 的基础知识以及它在 ASP.NET 生态系统中的位置。本章解释了网络开发的基本知识，包括客户端组件和服务器端组件，以及程序员在每一层可以做什么和不能做什么。

第二章，*设置环境*，向读者展示了如何设置开发环境，包括 Visual Studio 和 ASP.NET Core 的安装。还讨论了设置开发环境的硬件和软件要求，并介绍了 ASP.NET MVC 应用程序的结构。

第三章，*控制器*，解释了构成控制器和动作方法的内容及其角色和责任。在本章中，将创建一个简单的控制器及其动作方法，并从整体 ASP.NET MVC 应用程序的角度向读者解释动作方法和控制器的作用。

第四章，*视图*，介绍了 Razor 视图引擎的功能，并使用 Razor 视图引擎的示例解释了各种基本编程结构（条件语句、循环等）。

第五章，*模型*，介绍了模型在 ASP.NET Core 应用程序中的作用。讨论了 ViewModel 的概念，以及它如何为您的应用程序提供灵活性和数据分区。

第六章，*验证*，解释了使用 JavaScript 和 jQuery 库在客户端和服务器端进行验证。

第七章，*路由*，解释了路由模块如何通过示例从接收到的请求中选择适当的控制器，并展示了路由的各种选项和功能。本章还将指导您根据业务逻辑或 SEO 目的为 ASP.NET MVC 应用程序构建自定义路由。

第八章，*使用 Bootstrap 美化 ASP.NET 应用程序*，教授如何使用 Bootstrap，一个响应式前端框架，来美化你的应用程序。你将指导创建 HTML 表单控件。

第九章，*ASP.NET Core 应用程序的部署*，解释了 project.json 库如何处理 ASP.NET Core 应用程序的所有依赖项以及版本。它还解释了 K 运行时（ASP.NET Core 应用程序中的最新选项），这样 ASP.NET MVC 应用程序就可以在非 Windows 环境中部署。

第十章，*使用 Web API 构建 Web 服务*，解释了基于 HTTP 的服务和如何使用 Web API 来实现它们。它还将介绍 Fiddler，并使用它来组合 HTTP 请求。

第十一章，*提高 ASP.NET Core 应用程序的性能*，解释了分析应用程序性能的方法以及提高性能的各种层级的措施。

第十二章，*ASP.NET Core Identity*，解释了应用程序的安全方面以及如何使用 Entity Framework 实现应用程序的安全身份。

# 你需要这本书什么

要开始编程 ASP.NET MVC 应用程序，你需要 Visual Studio Community 2015 IDE。这是一个功能齐全的 IDE，可用于构建桌面和 Web 应用程序。你还需要各种包和框架，例如 NuGet、Bootstrap 和 project.json，这些的安装和配置将在书中解释。

# 这本书适合谁

这本书是为想要学习如何使用 ASP.NET Core 构建 Web 应用程序的开发者而写的，是为想要使用 Microsoft 技术构建 Web 应用程序的职业开发者而写的，以及为在 Ruby on Rails 或其他 Web 框架中工作的开发者而写的，他们想学习如何使用 ASP.NET Core MVC。

不需要了解 ASP.NET 平台或 .NET 平台。即使你不需要有 C# 的经验，对任何现代编程语言的基本结构（循环、条件、类和对象）的理解也会有所帮助。

# 读累了记得休息一会哦~

**公众号：古德猫宁李**

+   电子书搜索下载

+   书单分享

+   书友学习交流

**网站：**[沉金书屋 https://www.chenjin5.com](https://www.chenjin5.com)

+   电子书搜索下载

+   电子书打包资源分享

+   学习资源分享

# 规范

在这本书中，你会发现许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 标签以如下方式显示：“我们需要在`project.json`框架中将`Kestrel HTTP Server`包作为依赖项添加。”

代码块设置如下：

```cs
public static void Main(string[] args)
```

```cs
{

```

```cs
var host = new WebHostBuilder()

```

```cs
.UseKestrel()

```

```cs
}
```

任何命令行输入或输出都写成如下：

```cs

vi project.json

```

**新术语**和**重要词汇**以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“本书中的快捷键基于`Mac OS X 10.5+`方案。”

### 注意

警告或重要注意事项以如下方式显示在框中。

### 技巧

技巧和窍门看起来像这样。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者的反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大价值的标题。要发送一般反馈，请简单地发送电子邮件至 feedback@packtpub.com，并在邮件主题中提及书的标题。如果您在某个主题领域有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从购买中获得最大价值。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**标签上。

1.  点击**代码下载 & 错误修正**。

1.  在**搜索**框中输入书的名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买这本书的地方。

1.  点击**代码下载**。

文件下载完成后，请确保您使用最新版本解压缩或提取文件夹：

+   适用于 Windows 的 WinRAR / 7-Zip

+   适用于 Mac 的 Zipeg / iZip / UnRarX

+   适用于 Linux 的 7-Zip / PeaZip

本书代码包也托管在 GitHub 上[`github.com/PacktPublishing/learning-asp-dot-net-core-programming`](https://github.com/PacktPublishing/learning-asp-dot-net-core-programming)。我们还有其他来自我们丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表的颜色图像的 PDF 文件。这些颜色图像将帮助您更好地理解输出的变化。您可以从 [`www.packtpub.com/sites/default/files/downloads/LearningAspDotNetCoreProgramming_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/Bookname_ColorImages.pdf) 下载此文件。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata) ，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support) 并在搜索字段中输入书籍名称。所需信息将在**勘误**部分显示。

## 海盗行为

在互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过 copyright@packtpub.com 联系我们，并提供疑似盗版材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您对本书的任何方面有问题，您可以通过 questions@packtpub.com 联系我们，我们将尽力解决问题。
