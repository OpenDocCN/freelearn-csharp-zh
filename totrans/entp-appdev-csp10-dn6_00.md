# 前言

.NET 6是一个开源、免费的全栈框架，用于编写针对任何平台的应用程序。该框架为你提供了轻松编写应用程序的机会，包括云应用。作为软件开发者，我们肩负着构建复杂企业应用程序的责任。在这本书中，我们将学习使用C# 10和.NET 6构建企业应用程序的各种高级架构和概念。

这本书可以作为在.NET 6中构建企业应用程序时的参考。它包含了对基本概念的逐步解释、实际示例和自我评估问题，你将深入了解并接触到构建专业企业应用程序所需的.NET 6的每一个重要组件。

# 这本书面向的对象

这本书面向的是已经熟悉.NET经典或.NET Core和C#的中级到高级开发者。

# 这本书涵盖的内容

[*第1章*](B18507_01_Epub.xhtml#_idTextAnchor020)，*设计和架构企业应用程序*，首先讨论了常用的企业架构和设计模式，然后介绍如何将企业应用程序设计为包含UI层、服务层和数据库的三层应用程序。

[*第2章*](B18507_02_Epub.xhtml#_idTextAnchor040)，*介绍.NET 6 Core和Standard*，讨论了与.NET 6一起发布的C# 10的新特性。

[*第3章*](B18507_03_Epub.xhtml#_idTextAnchor175)，*介绍C# 10*，从我们的认知出发，运行时是代码运行的地方。在本章中，你将了解.NET 6运行时组件的核心和高级概念。

[*第4章*](B18507_04_Epub.xhtml#_idTextAnchor205)，*线程和异步操作*，帮助你详细了解线程、线程池、任务以及async/await，以及.NET如何让你构建异步应用程序。

[*第5章*](B18507_05_Epub.xhtml#_idTextAnchor445)，*.NET 6中的依赖注入*，帮助我们理解依赖注入是什么以及为什么每个开发者都在涌向它。我们将学习.NET 6中依赖注入的工作方式，并列出其他可用的选项。

[*第6章*](B18507_06_Epub.xhtml#_idTextAnchor473)，*.NET 6中的配置*，教你如何配置.NET 6并在你的应用程序中使用配置和设置。你还将学习如何扩展.NET 6配置以定义自己的部分、处理程序、提供者等。

[*第7章*](B18507_07_Epub.xhtml#_idTextAnchor596)，*.NET 6中的日志记录*，讨论了.NET 6中的事件和日志API。我们还将深入了解使用Azure和Azure组件进行日志记录，并学习如何进行结构化日志记录。

[*第8章*](B18507_08_Epub.xhtml#_idTextAnchor714)，*关于缓存的全部知识*，讨论了.NET 6中可用的缓存组件以及最佳行业模式和惯例。

[*第9章*](B18507_09_Epub.xhtml#_idTextAnchor860)，*在.NET 6中使用数据*，讨论了两种可能的数据提供程序：SQL和关系数据库管理系统（RDBMS）等数据库。我们还将从高层次上讨论如何使用.NET 6来存储和处理NoSQL数据库。本章将讨论.NET Core与文件、文件夹、驱动器、数据库和内存的接口。

[*第10章*](B18507_10_Epub.xhtml#_idTextAnchor1040)，*创建ASP.NET Core 6 Web API*，通过使用ASP.NET 6 Web API模板来开发我们的企业应用程序的服务层。

[*第11章*](B18507_11_Epub.xhtml#_idTextAnchor1228)，*创建ASP.NET Core 6 Web应用程序*，通过使用ASP.NET 6 MVC Web应用程序模板和Blazor来开发我们的企业应用程序的Web层。

[*第12章*](B18507_12_Epub.xhtml#_idTextAnchor1389)，*理解身份验证*，讨论了行业中最常见的身份验证模式以及您如何使用.NET 6来实现它们。我们还将介绍如何实现自定义身份验证。

[*第13章*](B18507_13_Epub.xhtml#_idTextAnchor1531)，*在.NET 6中实现授权*，讨论了不同的授权方法以及ASP.NET 6如何让您处理它。

[*第14章*](B18507_14_Epub.xhtml#_idTextAnchor1674)，*健康和诊断*，讨论了监控应用程序健康的重要性，为.NET应用程序构建`HealthCheck` API，以及Azure应用程序用于捕获遥测信息和诊断问题。

[*第15章*](B18507_15_Epub.xhtml#_idTextAnchor1803)，*测试*，讨论了测试的重要性。测试是开发的重要组成部分，没有适当的测试，任何应用程序都不能发布，因此我们还将讨论如何对我们的代码进行单元测试。我们还将学习如何衡量应用程序的性能。

[*第16章*](B18507_16_Epub.xhtml#_idTextAnchor1932)，*在Azure中部署应用程序*，讨论了在Azure中部署应用程序。我们将把我们的代码提交到我们选择的源代码控制，然后CI/CD管道将启动并在Azure中部署应用程序。

# 为了充分利用这本书

您需要在您的系统上安装.NET 6 SDK；所有代码示例都已在Windows操作系统上的Visual Studio 2022/Visual Studio Code上进行了测试。建议您拥有一个活动的Azure订阅，以便进一步部署企业应用程序。您可以在[https://azure.microsoft.com/en-in/free/](https://azure.microsoft.com/en-in/free/)创建一个免费账户。

![](img/Preface_Table.jpg)

**如果您使用的是这本书的数字版，我们建议您亲自输入代码或从书的GitHub仓库（下一节中有一个链接）获取代码。这样做将帮助您避免与代码复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从GitHub下载本书的示例代码文件[https://github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition](https://github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition)。如果代码有更新，它将在GitHub仓库中更新。

我们还有其他来自我们丰富的图书和视频目录的代码包可供在[https://github.com/PacktPublishing/](https://github.com/PacktPublishing/)找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图和图表的彩色图像的PDF文件。您可以从这里下载：[https://static.packt-cdn.com/downloads/9781803232973_ColorImages.pdf](https://static.packt-cdn.com/downloads/9781803232973_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`: 表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟URL、用户输入和Twitter昵称。以下是一个示例："`WriteMinimalPlainText`将仅输出健康检查服务的总体状态。"

代码块按以下方式设置：

[PRE0]

[PRE1]

[PRE2]

[PRE3]

[PRE4]

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

[PRE5]

[PRE6]

[PRE7]

[PRE8]

[PRE9]

任何命令行输入或输出都按以下方式编写：

[PRE10]

**粗体**: 表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词以**粗体**显示。以下是一个示例：“让我们在用户点击**产品详情**页面上的**添加到购物车**按钮时添加自定义事件跟踪。”

小贴士或重要提示

看起来像这样。

# 联系我们

欢迎读者反馈

**一般反馈**: 如果您对本书的任何方面有疑问，请通过[mailto:customercare@packtpub.com](mailto:customercare@packtpub.com)给我们发邮件，并在邮件主题中提及书名。

**勘误**: 尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，如果您能向我们报告，我们将不胜感激。请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata) 并填写表格。

**盗版**: 如果您在互联网上以任何形式发现了我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过[mailto:copyright@packt.com](mailto:copyright@packt.com)与我们联系，并提供材料的链接。

**如果您想成为作者**: 如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了《使用 C# 10 和 .NET 6 开发企业级应用程序》，我们非常期待听到您的想法！请[点击此处直接访问此书的亚马逊评论页面](https://packt.link/r/1-803-23297-8)并分享您的反馈。

您的评论对我们和科技社区都非常重要，并将帮助我们确保我们提供高质量的内容。
