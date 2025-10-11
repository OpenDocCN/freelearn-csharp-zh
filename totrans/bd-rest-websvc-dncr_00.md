# 前言

本书将引导读者通过 RESTful 网络服务的设计，并利用 ASP.NET Core 框架来实现这些服务。从 REST 背后哲学的基本原理开始，读者将经历设计和实现企业级 RESTful 网络服务的步骤。采用实用方法，每一章都提供了他们可以应用于自己情况的代码示例。它展示了最新 .NET Core 版本的威力，与 MVC 一起工作。然后它超越了框架的使用，探索解决弹性、安全和可伸缩性问题的方法。读者将学习处理 Web API 安全的技术，并了解如何实施单元和集成测试策略。最后，本书通过指导你构建 RESTful 网络服务的 .NET 客户端以及一些扩展技术来结束。

# 本书面向对象

这本书旨在为那些想学习使用最新的 .NET Core 框架构建 RESTful 网络服务的人编写。为了最好地利用书中包含的代码示例，你应该具备基本的 C# 和 .NET Core 知识。

# 本书涵盖内容

[第一章](61135ab9-f2f1-44a9-9016-75f70508b6bd.xhtml)，*入门*，将涵盖规划阶段，解释如何根据我们的需求或问题陈述确定完美的技术堆栈，以及 RESTful 和 RESTless 服务的根本方面。

[第二章](c5995463-034e-4746-8255-e0bdfa248c4a.xhtml)，*构建初始框架 – 应用程序布局基础*，将使你熟悉各种方法的概念，例如 GET、POST、PUT、DELETE 等。

[第三章](09fe6ad3-7061-4ae2-a2cb-c454ba802985.xhtml)，*用户注册和管理*，将使你熟悉使用 ASP.NET Core 2.0、Entity Framework Core、基本身份验证和 OAuth 2.0 进行身份验证。

[第四章](920b925a-af77-4f0d-8055-6313669af320.xhtml)，*商品目录、购物车和结账*，将帮助你理解在构建电子商务应用程序的不同部分时 ASP.NET Core 的复杂组件，包括 .NET Standard 2.0。

[第五章](f3ee816a-8343-4a03-9e1d-1682e4cab467.xhtml)，*集成外部组件和处理*，将帮助你了解中间件、使用中间件实现日志记录、身份验证和资源限制。

[第六章](17c8ef83-d95b-41af-9b61-54f69981bc3a.xhtml)，*测试 RESTful 网络服务*，将使你熟悉测试范式、测试概念、存根和模拟、安全测试和集成测试。

[第七章](f766a64a-f90e-487a-8ba3-30e49a2415f3.xhtml)，*持续集成和持续部署*，将使你熟悉使用 VSTS 和 Azure 的 CI 和 CD 概念。

[第八章](18479f1e-2030-404b-b016-1764984f46ed.xhtml)，*保护 RESTful 网络服务*，将帮助你了解各种安全技术，包括基本身份验证、XSS 攻击和数据加密。

[第9章](dfa68fd5-a510-4446-be5c-fe23d0ca08cd.xhtml)，*扩展RESTful服务（Web服务的性能）*，将解释扩展内、扩展外以及各种扩展模式。

[第10章](9fcac4d2-710a-48a2-98be-ed0034525cee.xhtml)，*构建Web客户端（消费Web服务）*，将教授读者使用ASP.NET Core和Rest Client with RestSharp。

[第11章](f0f2e5ae-96d9-49b7-bdd9-ce2ea9c75dea.xhtml)，*微服务简介*，通过涵盖ASP.NET Core中的微服务生态系统来概述微服务。

# 为了充分利用本书

读者应具备先前的.NET Core和.NET Standard知识，以及C#、RESTful服务、Visual Studio 2017（作为IDE）、Postman、高级Rest客户端和Swagger的基本知识。

为了设置系统，读者应在他们的机器上具备以下内容：

+   Visual Studio 2017更新3或更高版本（有关下载和安装说明，请参阅[https://docs.microsoft.com/en-us/visualstudio/install/install-visual-studio](https://docs.microsoft.com/en-us/visualstudio/install/install-visual-studio)）

+   SQL Server 2008 R2或更高版本（有关下载和安装说明，请参阅[https://blogs.msdn.microsoft.com/bethmassi/2011/02/18/step-by-step-installing-sql-server-management-studio-2008-express-after-visual-studio-2010/](https://blogs.msdn.microsoft.com/bethmassi/2011/02/18/step-by-step-installing-sql-server-management-studio-2008-express-after-visual-studio-2010/)))

+   .NET Core 2.0

    +   下载：[https://www.microsoft.com/net/download/windows](https://www.microsoft.com/net/download/windows)

    +   安装说明：[https://blogs.msdn.microsoft.com/benjaminperkins/2017/09/20/how-to-install-net-core-2-0/](https://blogs.msdn.microsoft.com/benjaminperkins/2017/09/20/how-to-install-net-core-2-0/)

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了此书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择SUPPORT选项卡。

1.  点击代码下载与勘误表。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保您使用最新版本的以下软件解压缩或提取文件夹：

+   Windows上的WinRAR/7-Zip

+   Mac上的Zipeg/iZip/UnRarX

+   Linux上的7-Zip/PeaZip

本书代码包也托管在GitHub上，地址为[https://github.com/PacktPublishing/Building-RESTful-Web-services-with-DOTNET-Core](https://github.com/PacktPublishing/Building-RESTful-Web-services-with-DOTNET-Core)。如果代码有更新，它将在现有的GitHub仓库中更新。

我们还有其他来自我们丰富的书籍和视频目录的代码包可供选择，请访问[https://github.com/PacktPublishing/](https://github.com/PacktPublishing/)。查看它们！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表的彩色图像的PDF文件。您可以从[https://www.packtpub.com/sites/default/files/downloads/BuildingRESTfulWebServiceswithDOTNETCore_ColorImages.pdf](https://www.packtpub.com/sites/default/files/downloads/BuildingRESTfulWebServiceswithDOTNETCore_ColorImages.pdf)下载它。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟URL、用户输入和Twitter昵称。以下是一个示例：“`Header`必须作为信封的第一个子元素出现，在`body`元素之前。”

代码块设置如下：

[PRE0]

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

[PRE1]

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“**简单对象访问协议**（**SOAP**）是一种基于XML的消息协议，用于计算机之间交换信息。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请通过`feedback@packtpub.com`发送电子邮件，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`与我们联系。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packtpub.com`与我们联系，并提供材料的链接。

如果您有兴趣成为作者：如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了这本书，为什么不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt公司可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解Packt的更多信息，请访问 [packtpub.com](https://www.packtpub.com/).
