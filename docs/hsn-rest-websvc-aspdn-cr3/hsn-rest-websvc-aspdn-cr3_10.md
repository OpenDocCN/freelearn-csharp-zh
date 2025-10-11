# 第三部分：构建现实世界的 RESTful API

在本节中，我们将逐步介绍一些具体实现的 RESTful Web 服务的实现。本节提供了在实现和测试 Web 服务方面的具体方法。在第一阶段，我们将重点关注通过区分三个层次来实现 Web 服务的实现：数据访问层、领域层和 HTTP 层。此外，我们将了解如何使用一些容器化技术本地运行服务，并发现一些多个 Web 服务之间通信的具体模式。最后，我们将概述如何通过实现令牌认证来保护服务。

正如我们在第一章，*REST 101 和 ASP.NET Core 入门*中提到的，以下示例需要您在操作系统上安装.NET Core 3.1。还建议您安装支持.NET Core 的 IDE，例如 Visual Studio 2019、Visual Studio for Mac 或 Rider IDE。您也可以使用 Visual Studio Code 作为替代，但请注意，您将不会获得其他编辑器提供的 IntelliSense 和代码分析舒适度。

本节中描述的所有代码均可在 GitHub 上找到，链接为[`github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3`](https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3)。

本节包括以下章节：

+   第八章，*构建数据访问层*

+   第九章，*实现领域逻辑*

+   第十章，*实现 RESTful HTTP 层*

+   第十一章，*构建 API 的高级概念*

+   第十二章，*服务的容器化*

+   第十三章，*服务生态系统模式*

+   第十四章，*使用.NET Core 实现工作服务*

+   第十五章，*保护您的服务*
