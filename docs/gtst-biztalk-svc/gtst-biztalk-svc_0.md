# 前言

所有这一切都始于大约一年前，BizTalk Services 将在几个月后进行预览。我们都兴奋地期待在云中间件时代开辟新的领域。我们必须告诉你们，作为产品组的一员（或成为 MVP），一个好处是你们可以提前很久就获得这些功能。在研究这些功能时，我们心想，“如果我们的客户能有一个指南来构建有效的解决方案，那岂不是很好？”这本书关于 BizTalk Services 的设想并不是为了添加每一个细节而破坏乐趣，而是要涵盖足够的内容，以便理解架构、关键组件，并帮助你探索。

这本书是为初学者编写的，并不假设或期望读者了解 BizTalk Server。这也是该主题的第一本书，我们将详细涵盖所有重要功能，包括 EAI、B2B 和混合部署——所有这些都有代码示例和操作指南。如果你是 EAI 用户，可以从第一章，*欢迎使用 BizTalk Services*开始，然后继续阅读第二章，*消息和转换*，第三章，*桥梁*，以及第四章，*企业应用集成*。另一方面，一个 B2B 开发人员或架构师可以遵循第一章，*欢迎使用 BizTalk Services*，第二章，*消息和转换*，以及第五章，*企业对企业集成*。如果你对支撑服务的 API 感兴趣，需要解决你的解决方案的问题，或者想知道如何迁移到 BizTalk Services，那么第六章，*API*，第七章，*跟踪和故障排除*，以及第八章，*迁移到 BizTalk Services*将为你提供指导。

# 本书涵盖的内容

第一章，*欢迎使用 BizTalk Services*，介绍了 BizTalk Services，其架构以及如何创建服务实例和部署解决方案。

第二章, *消息和转换*，解释了消息处理以及如何将消息转换为不同的格式。它还解释了如何使用映射操作来聚合数据、执行参考数据查找以及在转换中使用自定义代码。

第三章, *桥梁*，详细介绍了桥梁，并解释了如何丰富消息并将消息路由到不同的端点。

第四章，*企业应用集成*，解释了源和目标，以及如何从云中将 BizTalk 服务连接到本地企业应用和系统。

第五章，*企业对企业集成*，讨论了使用行业标准协议（如 EDIFACT、X12 和 AS2）进行 B2B 集成。它还讨论了如何在 BizTalk 服务中创建合作伙伴和协议以连接交易伙伴，以及如何利用消息批处理和归档。

第六章，*API*，讨论了 BizTalk 服务下的丰富 API。它还解释了它能够做什么，以及如何在不同的环境中使用它，包括 REST、PowerShell 和自定义代码。

第七章，*跟踪和故障排除*，讨论了在 BizTalk 服务中消息是如何被跟踪的，以及当问题发生时如何使用 BizTalk 服务提供的工具来查找和解决问题。

第八章，*迁移到 BizTalk 服务*，解释了如何从 BizTalk 服务器迁移到 BizTalk 服务，两个产品的区别以及未来的计划。

# 您需要这本书的内容

要跟随书中提供的代码示例和解决方案，您需要以下先决条件：

+   互联网接入

+   以下操作系统之一：Windows 7 Service Pack 1、Windows 8、Windows 8.1、Windows Server 2008 R2 SP1、Windows Server 2012 或 Windows Server 2012 R2

+   互联网浏览器 9 或 10

+   Visual Studio 2012

+   Windows Azure BizTalk 服务 SDK

    请访问[`msdn.microsoft.com/en-us/library/windowsazure/hh689760.aspx`](http://msdn.microsoft.com/en-us/library/windowsazure/hh689760.aspx)

+   需要的 Windows Azure 订阅和 Windows Azure BizTalk 服务实例

    要创建 BizTalk 服务实例，请访问[`www.windowsazure.com/en-us/pricing/free-trial/`](http://www.windowsazure.com/en-us/pricing/free-trial/)

# 这本书面向的对象

这本书是为希望了解 BizTalk 服务、它能做什么以及如何用它来集成服务、本地应用和企业的软件开发人员、IT 专业人士、架构师和技术经理而编写的。

# 习惯用法

在这本书中，您将找到许多不同风格的文本，以区分不同类型的信息。以下是一些这些风格的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名如下所示："让我们解决`ShippingAddress`节点。"

代码块设置如下：

```cs
public string CreateAddress (string Number, string Street, string City, string State, string Country)
{
  return Number + " " +
    Street + "," +
    City + "," +
    State + "," +
    Country;
}
```

任何命令行输入或输出都如下所示：

```cs
select-azuresubscription –SubscriptionName "Test"

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上、菜单或对话框中看到的单词，例如，在文本中显示为：“点击**添加**以创建地图并将其添加到解决方案。”

### 注意

警告或重要注意事项以这种框的形式出现。

### 小贴士

小技巧和技巧以这种形式出现。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者的反馈对我们开发您真正能从中受益的标题非常重要。

要发送一般反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)上下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然会发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)来报告它们，选择您的书籍，点击**错误清单提交表**链接，并输入您的错误详情。一旦您的错误清单得到验证，您的提交将被接受，错误清单将被上传到我们的网站，或添加到该标题的错误清单部分。任何现有的错误清单都可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)中选择您的标题来查看。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即向我们提供位置地址或网站名称，以便我们可以追究补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供疑似盗版材料的链接。

我们感谢您在保护我们的作者以及为我们提供有价值内容的能力方面提供的帮助。

## 问题

如果你在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
