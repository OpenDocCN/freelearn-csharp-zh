# 前言

本书是构建 .NET Standard 2.0 库及其在各个 .NET 平台上使用的逐步指南。此外，我们将演示如何使用 ASP.NET、ASP.NET Core、ASP.NET MVC、Xamarin iOS 和 Xamarin Android 使用库。最后，我们将指导你如何使用 NuGet 包管理器打包和发送你的库。

# 本书面向的对象

这本书是为需要接触新 .NET Standard 2.0 库的开发者而写的。如果你对 C# 有基本的了解，以及库能对你的代码做什么，那么这足以理解这本书。此外，如果你是一位编写第三方库的开发者，这将给你一个关于 .NET Standard 2.0 库是什么以及如何使用它来满足 Windows、Linux 和 macOS 上的各种 .NET 平台的公平概念。

# 本书涵盖的内容

[第 1 章](c123d601-050b-4a65-bd1b-719915e42c77.xhtml)，*回到基础*，为试图构建第一个基于 .NET 的类库的新手概述了所有开始的地方。它讨论了这种方法的局限性。本章通过一个简单的项目开始，并使用控制台应用程序和经典基于 Windows 的应用程序来使用它，讨论了当前库方法的局限性。它向读者介绍了 .NET Standard 2.0 及其版本。涵盖了如何创建第一个 .NET Standard 2.0 库以及如何使用它与各种 .NET 版本，如 .NET Framework 和 .NET Core 应用程序。

[第 2 章](b314c115-b9be-49b2-9826-8f78b3fcc8a6.xhtml)，*原始类型、集合、LINQ 等*，探讨了 .NET Standard 2.0 的核心。它讨论了原始类型、集合、反射和 LINQ。涵盖了迄今为止支持的区域。它利用了不同版本的 .NET Framework 之间的功能。

[第 3 章](e7257972-4311-48d2-8b08-aa7b1dbfc182.xhtml)，*与文件一起工作*，解释了在 .NET Standard 2.0 中 System.IO 和 System.Security 的用法，以及使用文件系统进行读写操作的使用方法。它还介绍了在 Ubuntu 和 macOS 上的 .NET Core 的跨平台版本中的使用。

[第 4 章](2484ef14-ce0a-463d-8d0b-97d3cd25c3f0.xhtml)，*函数式编程*，介绍了 C# 中的函数式编程功能以及如何在 .NET Standard 2.0 库中使用它们。

[第 5 章](2197acce-cbbc-4850-814b-cc5bb09dacf3.xhtml)，*XML 和数据*，解释了在 .NET Standard 2.0 中使用 System.XML 和 System.Data 创建 XML 文档以及使用数据表的用法。

[第 6 章](e0961b72-8059-4a52-b45e-aee148e0747c.xhtml)，*探索多线程*，讨论了创建多线程 .NET Standard 2.0 库的线程支持。这还包括在库中使用 C# 的异步功能（Tasks）。

[第 7 章](d8dd536a-a2d7-4a1b-9cb2-8bea0d890b06.xhtml)，*网络*，专注于 .NET Standard 2.0 中 System.Net 的用法。它深入探讨了套接字、Http 和邮件以及如何在 .NET Standard Library 2.0 中使用它们。

[第8章](03c5bbfe-3780-4928-9256-c9f13a856479.xhtml)，*使用Xamarin迈向iOS*，概述了使用Visual Studio for Mac移动iOS工具创建一个简单的基于移动的应用程序。本章指导您创建一个.NET Standard 2.0库，并将其与构建的iOS应用程序一起使用。

[第9章](56e32df9-4e89-4121-9c46-f66955717680.xhtml)，*使用Xamarin迈向Android*，将指导您使用Visual Studio for Mac和Android工具创建一个简单的基于移动的应用程序。它展示了如何创建一个.NET Standard 2.0库，并将其与构建的Android应用程序一起使用。

[第10章](2a145561-0682-4427-bfc0-a70d1e480307.xhtml)，*让我们微调我们的库*，展示了如何使用调试工具和诊断工具微调.NET Standard 2.0库。它还帮助您捕获异常，并确保最终用户在使用我们的.NET Standard Library 2.0时拥有良好的体验。

[第11章](aa99e5f0-dd44-4deb-9242-6adaac1d565e.xhtml)，*打包和交付*，讨论了如何将完成的.NET Standard 2.0库交付给世界。涵盖了如何使用NuGet包管理器、创建包以及交付的内容。

[第12章](78b4923e-f1af-4d8d-bdfd-a35fe5ec0ad1.xhtml)，*部署*，介绍了如何创建.NET Standard Library 2.0，将其与ASP.NET Core Web应用程序一起使用，并将其部署到Azure云。

# 为了充分利用本书

本书假设读者具备C#的基本知识。此外，它还假设读者具备使用Visual Studio、使用NuGet安装包以及在项目中引用其他项目中的库的基本知识。

基本了解如何在Ubuntu上使用bash和macOS中的终端等命令行工具将是一个额外的优势，但不是必须的。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本解压缩或提取文件夹：

+   Windows上的WinRAR/7-Zip

+   Mac上的Zipeg/iZip/UnRarX

+   Linux上的7-Zip/PeaZip

本书代码包也托管在GitHub上，网址为[https://github.com/PacktPublishing/DotNET-Standard-2-Cookbook](https://github.com/PacktPublishing/DotNET-Standard-2-Cookbook)。我们还有其他丰富的图书和视频的代码包可供选择，网址为**[https://github.com/PacktPublishing/](https://github.com/PacktPublishing/)**。请查看它们！

# 代码实战

访问以下链接查看代码运行的视频：

[https://goo.gl/GwN5Xq](https://goo.gl/GwN5Xq)

# 使用的约定

在本书中，使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟URL、用户输入和Twitter昵称。以下是一个示例：“将下载的`WebStorm-10*.dmg`磁盘映像文件作为系统中的另一个磁盘挂载。”

代码块按照以下方式设置：

[PRE0]

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

[PRE1]

任何命令行输入或输出都按照以下方式编写：

[PRE2]

**粗体**: 表示新术语、重要词汇或屏幕上看到的词汇。例如，菜单或对话框中的文字会以这种方式显示。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要提示看起来像这样。

小贴士和技巧看起来像这样。

# 部分

在本书中，您将找到一些频繁出现的标题（*准备工作*、*如何操作...*、*工作原理...*、*还有更多...* 和 *参见*）。

为了清楚地说明如何完成食谱，请按照以下方式使用这些部分：

# 准备工作

本节告诉您在食谱中可以期待什么，并描述如何设置任何软件或任何为食谱所需的初步设置。

# 如何操作…

本节包含遵循食谱所需的步骤。

# 工作原理…

本节通常包含对上一节发生事件的详细解释。

# 还有更多…

本节包含有关食谱的附加信息，以便您对食谱有更深入的了解。

# 参见

本节提供了对食谱其他有用信息的链接。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请将电子邮件发送至`feedback@packtpub.com`，并在邮件主题中提及书名。如果您对本书的任何方面有疑问，请通过电子邮件与我们联系`questions@packtpub.com`。

**勘误表**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在本书中发现错误，我们将不胜感激，如果您能向我们报告，我们将不胜感激。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误表提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上遇到我们作品的任何形式的非法副本，我们将不胜感激，如果您能提供位置地址或网站名称，我们将不胜感激。请通过电子邮件与我们联系`copyright@packtpub.com`，并提供材料的链接。

**如果您想成为一名作者**：如果您在某个领域有专业知识，并且对撰写或为本书做出贡献感兴趣，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下您的评价。一旦您阅读并使用了这本书，为何不在购买它的网站上留下评价呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，而我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/).
