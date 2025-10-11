# 前言

本书是针对新.NET Core 2.0版本中依赖注入技术实现的方法。.NET Core的设计者实现了许多与良好实践相关的功能，并在本版本的各个领域遵循了Robert C. Martin（SOLID原则）所阐述的原则。

本书的目的是深入探讨那些明确阐述的原则，并通过示例展示它们是如何实现的，以及程序员如何使用它们。

# 本书涵盖的内容

第1章，*软件设计的SOLID原则*，介绍了五个SOLID原则以及它们如何在.NET Core 2.0中找到或轻松实现。

第2章，*依赖注入和IoC容器*，让你了解如何独立使用或借助第三方容器使用依赖注入。

第3章，*在.NET Core 2.0中介绍依赖注入*，从控制台应用程序的角度回顾了.NET Core 2.0内部依赖注入的实际实现。

第4章，*ASP.NET Core中的依赖注入*，提供了对使用ASP.NET Core 2.0的Web应用程序中依赖注入技术实现的更详细研究，并充满了示例。

第5章，*对象组合*，带你了解对象组合概念背后的所有隐藏原则，以及如何在.NET Core 2.0和ASP.NET Core MVC 2.0中应用，从而成为依赖注入的支柱。

第6章，*对象生命周期*，作为下一个依赖注入支柱，深入探讨了由依赖注入管理的对象的生活方式和典型管理策略，这有助于通过优化的配置做出更好的决策。

第7章，*拦截*，作为依赖注入生态系统的最后一根支柱，提供了拦截调用和动态地将代码插入管道的技术。本章还讨论了在.NET Core 2.0和ASP.NET Core 2.0中拦截的应用，并配有适当的插图。

第8章，*模式 – 依赖注入*，将引导你了解SOLID原则中的D，以及如何在.NET Core 2.0应用程序中应用所有重要的依赖注入技术。

第9章，*依赖注入的反模式和误解*，处理了DI模式中的常见不良使用和编码时应该避免的场景，以便从应用程序中的依赖注入中获得良好的结果。

第10章，*在其他JavaScript框架中使用依赖注入*，教你如何使用其他流行的框架，如Angular，来使用依赖注入技术。

[第11章](a437ff1b-c4af-41ac-b502-8718dc132272.xhtml)，*最佳实践和其他相关技术*，涵盖了在应用依赖注入到当前和旧应用程序时应采用的经过验证的编码、架构和重构实践。

# 你需要为本书准备的内容

你将需要Visual Studio 2017 Community Edition、Chrome浏览器和IIS（Internet Information Server Express）来成功测试和执行所有代码文件。

# 本书面向的对象

这本书是为那些对依赖注入（DI）一无所知，但希望了解如何在他们的应用程序中实现它的C#和.NET开发者而写的。

# 规范

在这本书中，您将找到许多不同的文本样式，以区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟URL、用户输入和Twitter用户名如下所示：“对于这个新需求，我们可以创建一个新的并重载的`ReadData()`方法，它接收一个额外的参数。”

代码块设置如下：

[PRE0]

当我们希望将您的注意力引到代码块的一个特定部分时，相关的行或项目将以粗体显示：

[PRE1]

任何命令行输入或输出都应如下编写：

[PRE2]

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中如下所示：“双击添加ArcGIS Server。”

警告或重要注意事项如下所示。

技巧和窍门看起来像这样。

# 读者反馈

我们欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者的反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大价值的标题。要发送一般反馈，请简单地发送电子邮件至`feedback@packtpub.com`，并在邮件主题中提及书籍的标题。如果您在某个主题领域有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是Packt书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大价值。

# 下载示例代码

您可以从您的[http://www.packtpub.com](http://www.packtpub.com)账户下载这本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[http://www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的SUPPORT标签上。

1.  点击代码下载与勘误。

1.  在搜索框中输入书籍名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书籍的地方。

1.  点击代码下载。

下载文件后，请确保您使用最新版本的软件解压缩或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

本书代码包也托管在GitHub上，网址为[https://github.com/PacktPublishing/Dependency-Injection-in-.NET-Core-2.0](https://github.com/PacktPublishing/Dependency-Injection-in-.NET-Core-2.0)。我们还有其他来自我们丰富图书和视频目录的代码包，可在[https://github.com/PacktPublishing/](https://github.com/PacktPublishing/)找到。查看它们吧！

# 下载本书的彩色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的PDF文件。这些彩色图像将帮助您更好地理解输出中的变化。您可以从[https://www.packtpub.com/sites/default/files/downloads/DependencyInjectioninNETCore20_ColorImages.pdf](https://www.packtpub.com/sites/default/files/downloads/DependencyInjectioninNETCore20_ColorImages.pdf)下载此文件。

# 错误更正

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误更正，请通过访问[http://www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击错误提交表单链接，并输入您的错误详细信息来报告它们。一旦您的错误更正得到验证，您的提交将被接受，并将上传到我们的网站或添加到该标题的错误部分下现有的错误列表中。要查看之前提交的错误更正，请转到[https://www.packtpub.com/books/content/support](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在错误部分下。

# 盗版

互联网上版权材料的盗版是一个跨所有媒体持续存在的问题。在Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。请通过发送链接到疑似盗版材料至`copyright@packtpub.com`与我们联系。我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

# 问题

如果您对本书的任何方面有问题，您可以通过`questions@packtpub.com`与我们联系，我们将尽力解决问题。
