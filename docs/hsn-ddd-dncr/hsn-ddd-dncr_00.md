# 前言

这本书将帮助你通过更好地理解用户、找到正确的问题来解决，并构建精益、事件驱动的系统来满足客户真正需求，从而解决复杂的企业问题。你将学习领域驱动设计（**DDD**）的基本原则以及如何使用现代工具如 EventStorming、Event Sourcing 和 CQRS 来应用它。通过这本书，你将了解 DDD 如何直接应用于各种架构风格，如 REST、反应式系统和微服务。你将赋予团队以灵活的方式与改进的服务和解耦的交互工作。

# 本书面向对象

这本书是为那些对 C#有中级水平理解的.NET 开发者和那些寻求提供价值而非仅仅编写代码的人所写。在 UI 章节中，具备中级水平的 JavaScript 技能将有所帮助。

# 本书涵盖内容

第一章，*为何选择领域驱动设计？*，涵盖了问题和解决方案空间、需求规范、复杂性、知识和无知的概念。这些话题对我们如何以及提供什么内容有着重要影响。

第二章，*语言与上下文*，深入探讨了语言的重要性，并解释了通用语言。

第三章，*EventStorming*，探讨了领域建模中最受欢迎的技术之一，并介绍了一些关于如何组织领域专家和开发者之间有用工作坊的实用技巧。

第四章，*设计模型*，深入建模过程，更多地关注可以帮助我们尽快开始编写代码并交付初始原型的工件。

第五章，*实现模型*，构成了我们用代码实现的领域模型的基础。我们将探讨在领域实体中执行行为的不同风格，并编写一些测试。

第六章，*使用命令行动*，展示了如何实现命令，以及命令是如何成为我们的领域模型和外部世界的粘合剂的。我们将学习如何通过让人们与之交互来使我们的模型变得有用。

第七章，*一致性边界*，更仔细地观察实体持久化，其范围将是我们的重点。我们将学习我们需要处理哪些类型的致性以及理解一致性边界的重要性。

第八章，*聚合持久化*，深入探讨了聚合持久化的主题。我们将找到一种方法将我们的领域对象存储在数据库中，并看到我们的应用程序首次运行。

第九章，*CQRS - 读取侧*，涵盖了 CQRS 的读取侧，并解释了读取模型是什么。您将学习如何使用通用语言进行查询，并了解如何使用一个数据库实现 CQRS。

第十章，*事件源*，展示了如何使用事件来持久化对象的状态，而不是使用传统的持久化机制。我们将介绍事件流的概念，并了解流与聚合之间的关系。我们将使用事件存储在流中持久化我们的聚合，并将它们加载回来。

第十一章，*投影和查询*，将带您了解查询事件源系统的挑战，并通过使用单独的读取模型和投影来解决这些挑战。

第十二章，*边界上下文*，使您熟悉边界上下文的概念。我们将识别项目中的上下文，并将系统分割成部分。我们还将了解上下文图，它显示了整个系统的边界上下文及其关系。 

[第十三章](https://www.packtpub.com/sites/default/files/downloads/Splitting_the_System.pdf)，*系统拆分*，提供了关于识别边界上下文和在示例应用程序中实现多个上下文的实用建议。本章作为在线章节可在以下链接找到：[`www.packtpub.com/sites/default/files/downloads/Splitting_the_System.pdf`](https://www.packtpub.com/sites/default/files/downloads/Splitting_the_System.pdf)。

# 为了充分利用本书

为了遵循本书中的说明，您需要具备中级水平的 C#理解。其他要求在各自的章节的相关实例中提及。

本书中的图表使用在线协作工具 Miro 和免费在线服务[draw.io](http://draw.io)创建，故意使用线条样式和字体，以体现所有模型和图表的时间性质和实用性。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误表”。

1.  在搜索框中输入书名，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   Windows 系统使用 WinRAR/7-Zip

+   Mac 系统使用 Zipeg/iZip/UnRarX

+   Linux 系统使用 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Hands-On-Domain-Driven-Design-with-.NET-Core`](https://github.com/PacktPublishing/Hands-On-Domain-Driven-Design-with-.NET-Core)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包可供选择，请访问**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。查看它们吧！

# 下载彩色图像

我们还提供包含本书中使用的截图/图表的彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/9781788834094_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781788834094_ColorImages.pdf)

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“将下载的`WebStorm-10*.dmg`磁盘映像文件作为系统中的另一个磁盘挂载。”

代码块设置如下：

```cs
html, body, #map {
 height: 100%; 
 margin: 0;
 padding: 0
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```cs
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

任何命令行输入或输出都按以下方式编写：

```cs
$ mkdir css
$ cd css
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要注意事项显示如下。

技巧和窍门显示如下。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请发送电子邮件至`feedback@packtpub.com`，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请发送电子邮件至`questions@packtpub.com`。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，如果您能向我们报告，我们将不胜感激。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过链接至材料的方式与我们联系至`copyright@packtpub.com`。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下您的评价。一旦您阅读并使用了这本书，为何不在购买它的网站上留下评价呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 公司可以了解您对我们产品的看法，并且我们的作者可以查看他们对书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
