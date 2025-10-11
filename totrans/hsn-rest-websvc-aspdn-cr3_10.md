# 第3节：构建现实世界的RESTful API

在本节中，我们将逐步介绍一些具体实现的RESTful Web服务的实现。本节提供了在实现和测试Web服务方面的具体方法。在第一阶段，我们将重点关注通过区分三个层次来实现Web服务的实现：数据访问层、领域层和HTTP层。此外，我们将了解如何使用一些容器化技术本地运行服务，并发现一些多个Web服务之间通信的具体模式。最后，我们将概述如何通过实现令牌认证来保护服务。

正如我们在[第1章](b3e95a60-c4fb-491e-ad7e-a2213f70a63b.xhtml)，*REST 101和ASP.NET Core入门*中提到的，以下示例需要您在操作系统上安装.NET Core 3.1。还建议您安装支持.NET Core的IDE，例如Visual Studio 2019、Visual Studio for Mac或Rider IDE。您也可以使用Visual Studio Code作为替代，但请注意，您将不会获得其他编辑器提供的IntelliSense和代码分析舒适度。

本节中描述的所有代码均可在GitHub上找到，链接为[https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3](https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3)。

本节包括以下章节：

+   [第8章](84b281bd-11a2-4703-81ae-ca080e2a267a.xhtml)，*构建数据访问层*

+   [第9章](f8ac60e1-e948-4435-b804-e3e4ff305510.xhtml)，*实现领域逻辑*

+   [第10章](266d98e8-df54-4024-aefb-b5871b3b35fa.xhtml)，*实现RESTful HTTP层*

+   [第11章](3e096108-790a-40ab-a663-1d11f40361ec.xhtml)，*构建API的高级概念*

+   [第12章](d3fd0efc-cb05-42bb-8e06-4fd3a7ca9300.xhtml)，*服务的容器化*

+   [第13章](8f70e186-50f1-4d55-99e6-026c73d5bb96.xhtml)，*服务生态系统模式*

+   [第14章](c8a85bf6-b5a8-497e-b1ab-61fbb1d2ae81.xhtml)，*使用.NET Core实现工作服务*

+   [第15章](3ede9648-ed9f-48cb-b9b2-6db5b4d4e882.xhtml)，*保护您的服务*
