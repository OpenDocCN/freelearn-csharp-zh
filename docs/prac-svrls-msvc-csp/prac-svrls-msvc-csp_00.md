# 前言

当我们开始编写这本书时，我们的主要目标是提供关于开发云原生解决方案的主要方法：分布式应用的实战经验。我们决定描述从无服务器实现到 Kubernetes 编排的各种构建微服务架构的选项。

由于我们的主要技术背景是.NET 和 Azure，我们决定专注于这些领域，为开发者提供了解如何以及何时无服务器和微服务是快速且一致地创建企业解决方案的最佳方式的机会，从而使.NET 开发者能够通过进入现代云原生和分布式应用的世界来实现职业跃迁。通过这本书，您将完成以下内容：

+   学习如何在 Azure 中创建无服务器环境进行开发和调试

+   实现可靠的微服务通信和计算

+   使用 Kubernetes 等编排器优化微服务应用

+   深入探索 Azure Functions 及其触发器，包括用于物联网和后台活动的触发器

+   使用 Azure Container Apps 简化创建和管理容器

+   学习如何正确地保护微服务应用

+   严肃对待成本和使用限制，并正确计算它们

我们相信，通过阅读这本书，您将找到许多宝贵的技巧和实用的示例，这些将帮助您编写自己的应用程序。我们希望这份专注的材料能够帮助您充分利用关于这个重要软件开发主题的知识。

# 这本书面向的对象

这本书是为那些渴望向现代云开发和分布式应用迈进、并希望提升他们对微服务和无服务器知识以充分利用这些架构模型的工程师和高级软件开发人员所写的。

# 这本书涵盖的内容

*第一章*，*揭秘无服务器应用*，介绍了无服务器应用，讨论了它们的优缺点和基础理论。

*第二章*，*揭秘微服务应用*，介绍了微服务应用，讨论了它们的优缺点、基本原理、定义和设计技术。

*第三章*，*设置和理论：Docker 和洋葱架构*，介绍了实现现代分布式应用所需的前提技术，例如 Docker 和洋葱架构。

*第四章*，*可用的 Azure Functions 和触发器*，讨论了与 Azure Functions 相关的可能设置以及可用于创建无服务器应用的触发器。

*第五章*，*实践中的后台函数*，实现了 Azure Functions 触发器，这些触发器能够实现后台处理。详细介绍了定时器、Blob 和队列触发器，包括它们的优缺点和使用机会。

*第六章*, *实践中的 IoT 函数*，讨论了 Azure Functions 在 IoT 解决方案中的重要性。

*第七章*, *实践中的微服务*，详细描述了使用.NET 实现微服务的方法。

*第八章*, *使用 Kubernetes 的实用微服务组织*，详细介绍了 Kubernetes 及其如何用于编排微服务应用程序。

*第九章*, *简化容器和 Kubernetes：Azure Container Apps 和其他工具*，描述了简化 Kubernetes 使用的工具，并介绍了 Azure Container Apps 作为简化微服务编排的选项，讨论了其成本、优势和劣势。

*第十章*, *无服务器和微服务应用程序的安全性和可观察性*，讨论了微服务场景中的安全性和可观察性，介绍了现代软件开发这两个重要方面的主要选项和技术。

*第十一章*, *拼车应用*，展示了本书的示例应用，使用无服务器和微服务应用来理解事件驱动应用程序的工作方式。

*第十二章*, *使用.NET Aspire 简化微服务*，将 Microsoft Aspire 描述为在微服务开发过程中进行测试的一个好选择。

# 要充分利用这本书

为了充分利用这本书，需要具备 C#/.NET 和 Microsoft 堆栈（Entity Framework 和 ASP.NET Core）的先验经验。

## 下载示例代码文件

本书代码包托管在 GitHub 上，网址为[`github.com/PacktPublishing/Practical-Serverless-and-Microservices-with-Csharp`](https://github.com/PacktPublishing/Practical-Serverless-and-Microservices-with-Csharp)。我们还有其他丰富的图书和视频的代码包，可在[`github.com/PacktPublishing`](https://github.com/PacktPublishing)找到。查看它们吧！

## 下载彩色图像

我们还提供了一个包含本书中使用的截图/图表的彩色 PDF 文件。您可以从这里下载：[`packt.link/gbp/9781836642015`](https://packt.link/gbp/9781836642015)。

## 使用的约定

本书使用了几个文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 X/Twitter 用户名。例如：“执行`docker build`命令。”

代码块设置如下：

```cs
public class TownBasicInfoMessage
{
    public Guid Id { get; set; }
    public string? Name { get; set; }
    public GeoLocalizationMessage? Location { get; set; }
} 
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```cs
FROM eclipse-temurin:11   
COPY . /var/www/java   
WORKDIR /var/www/java   
RUN javac Hello.java   
CMD ["java", "Hello"] 
```

任何命令行输入或输出都按照以下方式编写：

```cs
docker run --name myfirstcontainer simpleexample 
```

**粗体**：表示新术语、重要单词或屏幕上出现的单词。例如，菜单或对话框中的单词在文本中显示如下。例如：“从**管理**面板中选择**系统信息**。”

警告或重要提示如下所示。

技巧和窍门如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：通过`feedback@packtpub.com`发送电子邮件，并在邮件主题中提及书籍标题。如果您对此书的任何方面有疑问，请通过`questions@packtpub.com`与我们联系。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在此书中发现错误，我们将不胜感激，如果您能向我们报告，我们将不胜感激。请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，点击**提交勘误**，并填写表格。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packtpub.com`与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[`authors.packtpub.com/`](http://authors.packtpub.com/)。

# 分享您的想法

您已经完成《使用 C#的实用无服务器和微服务》一书，我们非常乐意听到您的想法！如果您从亚马逊购买此书，请[点击此处直接转到此书的亚马逊评论页面](https://packt.link/r/1836642016)并分享您的反馈或在该网站上留下评论。

您的评论对我们和科技社区非常重要，并将帮助我们确保我们提供高质量的内容。

# 下载此书的免费 PDF 副本

感谢您购买此书！

您喜欢在路上阅读，但无法携带您的印刷书籍到处走？

您的电子书购买是否与您选择的设备不兼容？

别担心，现在，每本 Packt 书籍都免费提供该书的 DRM 免费 PDF 版本，无需额外费用。

在任何地方、任何地方、任何设备上阅读。直接从您最喜欢的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

优惠不会就此停止，您还可以获得独家折扣、新闻通讯以及每天收件箱中的优质免费内容

按照以下简单步骤获取好处：

1.  扫描二维码或访问以下链接

![图片](img/B31916_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781836642015`](https://packt.link/free-ebook/9781836642015)

2. 提交您的购买证明

3. 就这样！我们将直接将您的免费 PDF 和其他优惠发送到您的电子邮件
