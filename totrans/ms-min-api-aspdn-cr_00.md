# 前言

代码的简化是每位开发者的梦想。最小化API是.NET 6中的一个新特性，旨在简化代码。它们用于在ASP.NET Core中构建具有最小依赖关系的API。最小化API通过使用更紧凑的代码语法来简化API开发。

使用最小化API的开发者将能够在某些情况下利用这种语法，以更少的代码和更少的文件维护来更快地工作。在这里，您将了解.NET 6的主要新特性，并理解最小化API的基本主题，这些在.NET 5和先前版本中是不可用的。您将了解如何启用Swagger进行API文档，以及如何处理CORS和应用程序错误。您将学习如何使用微软的新.NET框架——依赖注入来更好地组织代码。最后，您将看到最小化API在.NET 6中引入的性能和基准测试改进。

在本书结束时，您将能够利用最小化API，并理解它们与经典Web API开发的相关性。

# 本书面向对象

本书面向希望构建.NET和.NET Core API并希望研究.NET 6新特性的.NET开发者。假设您对C#、.NET、Visual Studio和REST API有基本了解。

# 本书涵盖的内容

[*第一章*](B17902_01.xhtml#_idTextAnchor014)，*最小化API简介*，向您介绍在.NET 6中引入最小化API的动机。我们将解释.NET 6的主要新特性以及.NET团队正在进行的这项最新版本的工作。您将了解我们决定编写本书的原因。

[*第二章*](B17902_02.xhtml#_idTextAnchor023)，*探索最小化API及其优势*，向您介绍最小化API与.NET 5及所有先前版本的基本区别方式。我们将详细探讨使用System.Text.JSON进行路由和序列化。最后，我们将讨论一些与编写第一个REST API相关的概念。

[*第三章*](B17902_03.xhtml#_idTextAnchor038)，*使用最小化API*，向您介绍最小化API与.NET 5及所有先前版本的高级区别方式。我们将详细探讨如何启用Swagger进行API文档。我们将看到如何启用CORS以及如何处理应用程序错误。

[*第四章*](B17902_04.xhtml#_idTextAnchor061)，*在最小化API项目中使用依赖注入*，向您介绍依赖注入，并介绍如何与最小化API一起使用它。

[*第五章*](B17902_05.xhtml#_idTextAnchor068)，*使用日志记录来识别错误*，向您介绍.NET提供的日志工具。记录器是开发者用来调试应用程序或理解其生产中失败原因的工具之一。日志库已经内置到ASP.NET中，并启用了设计中的几个功能。

[*第6章*](B17902_06.xhtml#_idTextAnchor082)，*探索验证和映射*，将教会您如何验证API接收到的数据，以及如何返回任何错误或消息。一旦数据得到验证，它就可以映射到一个模型，然后用于处理请求。

[*第7章*](B17902_07.xhtml#_idTextAnchor094)，*与数据访问层的集成*，帮助您了解在最小API中访问和使用数据的最佳实践。

[*第8章*](B17902_08.xhtml#_idTextAnchor109)，*添加身份验证和授权*，探讨了如何通过利用我们自己的数据库或像Azure Active Directory这样的云服务来编写身份验证和授权系统。

[*第9章*](B17902_09.xhtml#_idTextAnchor125)，*利用全球化和本地化*，展示了如何在最小API项目中利用翻译系统，并以客户端相同的语言提供错误。

[*第10章*](B17902_10.xhtml#_idTextAnchor140)，*评估和基准测试最小API的性能*，展示了.NET 6的改进以及将随着最小API引入的特性。

# 为了充分利用本书

您需要在计算机上安装Visual Studio 2022和ASP.NET以及Web开发工作负载，或者安装Visual Studio Code和K6。

所有代码示例都已使用Windows操作系统上的Visual Studio 2022和Visual Studio Code进行测试。

![](img/B17902_Preface_Table1.jpg)

**如果您使用的是本书的数字版，我们建议您亲自输入代码或从书的GitHub仓库（下一节中有一个链接）获取代码。这样做将帮助您避免与代码复制粘贴相关的任何潜在错误。**

要完全理解本书，需要具备Microsoft网络技术的基本开发技能。

# 下载示例代码文件

您可以从GitHub下载本书的示例代码文件：[https://github.com/PacktPublishing/Minimal-APIs-in-ASP.NET-Core-6](https://github.com/PacktPublishing/Minimal-APIs-in-ASP.NET-Core-6)。如果代码有更新，它将在GitHub仓库中更新。

我们在[https://github.com/PacktPublishing/](https://github.com/PacktPublishing/)的丰富图书和视频目录中也有其他代码包可供选择。查看它们吧！

# 下载彩色图像

我们还提供了一份PDF文件，其中包含本书中使用的截图和图表的彩色图像。

您可以在此处下载：[https://packt.link/GmUNL](https://packt.link/GmUNL)

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟URL、用户输入和Twitter昵称。以下是一个示例：“在最小API中，我们使用`WebApplication`对象的`Map*`方法定义路由模式。”

代码块设置如下：

[PRE0]

当我们希望引起您对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

[PRE1]

任何命令行输入或输出都应如下编写：

[PRE2]

**粗体**：表示新术语、重要词汇或您在屏幕上看到的词汇。例如，菜单或对话框中的词汇以**粗体**显示。以下是一个示例：“打开Visual Studio 2022，从主屏幕点击**创建新项目**。”

小贴士或重要注意事项

看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请通过电子邮件发送至[customercare@packtpub.com](mailto:customercare@packtpub.com)，并在邮件主题中提及书名。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在本书中发现错误，我们将非常感激您向我们报告。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，我们将非常感激您提供位置地址或网站名称。请通过[copyright@packt.com](mailto:copyright@packt.com)与我们联系，并提供材料的链接。

**如果您想成为一名作者**：如果您在某个领域有专业知识，并且对撰写或为本书做出贡献感兴趣，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了《精通ASP.NET Core中的最小API》，我们非常乐意听到您的想法！请[点击此处直接访问此书的亚马逊评论页面](https://packt.link/r/1803237821)并分享您的反馈。

您的评论对我们和科技社区都很重要，并将帮助我们确保我们提供高质量的内容。

前言

前言

# 第一部分：简介

在本书的第一部分，我们希望向您介绍本书的背景。我们将解释最小API的基础知识以及它们是如何工作的。我们希望一块一块地添加所需的知识，以便充分利用最小API所能赋予我们的所有功能。

我们在本部分将涵盖以下章节：

+   [*第1章*](B17902_01.xhtml#_idTextAnchor014)，*最小API简介*

+   [*第2章*](B17902_02.xhtml#_idTextAnchor023)，*探索最小API及其优势*

+   [*第3章*](B17902_03.xhtml#_idTextAnchor038)，*使用最小API*
