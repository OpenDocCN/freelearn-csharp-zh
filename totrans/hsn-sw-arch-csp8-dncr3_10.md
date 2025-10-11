# 使用 Azure Functions

正如我们在[第 4 章](049a0a4b-74b6-41a1-92db-87a4f8af9fd1.xhtml)，“选择最佳云解决方案”中提到的，无服务器架构是提供灵活软件解决方案的最新方式之一。为此，Microsoft Azure 提供了 Azure Functions，这是一种事件驱动的、无服务器的、可扩展的技术，可以加速您的项目开发。本章的主要目标是向您介绍 Azure Functions 以及您在使用它时可以实施的最佳实践。

在本章中，我们将涵盖以下主题：

+   理解 Azure Functions 应用程序

+   使用 C# 编程 Azure Functions

+   维护 Azure Functions

+   用例 - 实现使用 Azure Functions 发送电子邮件

到本章结束时，您将了解如何使用 C# 的 Azure Functions。

# 技术要求

本章要求您具备以下条件：

+   安装了所有数据库工具的 Visual Studio 2017 或 2019 免费社区版或更高版本。

+   一个免费的 Azure 账户。[第 1 章](14b5c5da-4042-439e-9e5a-2e19ba4c4930.xhtml)，“理解软件架构的重要性”中的“创建 Azure 账户”部分解释了如何创建一个。

您可以在[https://github.com/PacktPublishing/Hands-On-Software-Architecture-with-CSharp-8/tree/master/ch08](https://github.com/PacktPublishing/Hands-On-Software-Architecture-with-CSharp-8/tree/master/ch08)找到本章的示例代码。

# 理解 Azure Functions 应用程序

Azure Functions 应用程序是一个 Azure PaaS，您可以在其中构建代码片段（函数），并将它们连接到您的应用程序，并使用触发器来启动它们。这个概念相当简单 - 您可以使用您喜欢的语言构建一个函数，并决定启动它的触发器。您可以在系统中编写尽可能多的函数。有些情况下，系统完全使用函数编写。

创建必要环境的步骤与创建函数本身需要遵循的步骤一样简单。以下截图显示了您在创建环境时必须决定的参数。在您选择在 Azure 中创建资源并按 Function App 过滤后，您将看到以下屏幕：

![图片](img/10f24913-eea6-47e6-801c-18fcc84ee695.png)

在创建环境时，您应该考虑几个关键点。第一个是托管计划，您将在其中运行您的函数。托管计划有两种选择：消费计划和应用程序服务计划。让我们现在来谈谈这些。

# 消费计划

如果您选择消费计划，您的函数只有在执行时才会浪费资源。这意味着您只有在函数运行时才会被收费。可扩展性和内存资源将由 Azure 自动管理。

在此计划中编写函数时，我们需要注意的**超时**问题。默认情况下，5分钟后函数将超时。您可以使用`functionTimeout`参数更改超时值。最大值为10分钟。

当您选择消费计划时，您将被收取的费用将取决于您正在执行的内容、它们的执行时间和它们的内存使用情况。更多信息可以在[https://azure.microsoft.com/en-us/pricing/details/functions/](https://azure.microsoft.com/en-us/pricing/details/functions/)找到。

注意，如果您环境中没有App Service并且正在运行低周期性的函数，这可以是一个不错的选择。另一方面，如果您需要连续处理，您可能需要考虑App Service Plan。

# App Service Plan

当您想要创建Azure Functions应用时，App Service Plan是您可以选择的选项之一。以下是由Microsoft建议的几个原因，说明为什么您应该使用App Service Plan而不是消费计划来维护您的函数：

+   您可以使用未充分利用的现有App Service实例。

+   函数应用可以持续运行或几乎持续运行。

+   您需要的CPU或内存选项比消费计划提供的更多。

+   您的代码需要运行超过10分钟。

+   您需要诸如VNET/VPN连接等特性。

+   您希望将您的函数应用运行在Linux或自定义镜像上。

在App Service Plan的场景中，`functionTimeout`值根据Azure Function运行时版本而变化。然而，该值至少为30分钟。

# 使用C#编程Azure Functions

在本节中，您将学习如何创建Azure Functions。值得一提的是，有几种方法可以使用C#创建它们。第一种是在Azure Portal本身创建函数并开发它们。为此，请按照以下步骤操作：

1.  从主页进入所有资源，搜索`wwtravelclub`应用，并点击它。您将看到以下屏幕：

![图片](img/cc02e250-df6d-42f6-bc94-d4499a72b34e.png)

1.  点击“在门户中创建”选项。在这里，您将被提示选择您想要用于启动执行的触发类型。最常用的触发类型是**HTTP请求**和**定时器触发**，如下面的截图所示：

![图片](img/6e568874-17e7-461a-a28a-62e874894f58.png)

当您决定使用哪个触发器时，您必须为其命名。

1.  根据您选择的触发器，您将必须安装一些扩展并设置其他参数。例如，HTTP触发器要求您设置授权级别。有三个选项可供选择，即函数、匿名和管理员，其中我们选择了函数选项，如下面的截图所示：

![图片](img/df03fc3b-2b75-470a-b324-b7f3fc449d46.png)

值得注意的是，这本书并没有涵盖构建函数时所有可用的选项。作为一名软件架构师，您应该了解 Azure 在函数方面为无服务器架构提供了一个良好的服务。这在几种情况下都可能很有用。这已在[第 4 章](049a0a4b-74b6-41a1-92db-87a4f8af9fd1.xhtml)，*决定最佳云解决方案*中进行了更详细的讨论。

1.  结果如下。请注意，Azure 提供了一个编辑器，允许我们运行代码、检查日志并测试我们创建的函数。这是一个用于测试和编写基本函数的好界面：

![](img/d9e2bd25-f352-4a8c-84f9-17fb2e52eaf3.png)

1.  然而，如果您想创建更复杂的函数，您可能需要一个更复杂的开发环境，以便更有效地进行编码和调试。这就是 Visual Studio Azure Functions 项目能帮到您的所在。

在 Visual Studio 中，您可以通过访问“新建项目”|“云”菜单来创建一个专门用于 Azure Functions 的项目。

1.  一旦您提交了项目，Visual Studio 将询问您使用的触发类型以及函数将运行的 Azure 版本：

![](img/28194bbf-1190-4717-bef8-f46ee96451d8.png)

在撰写本文时，Azure Functions 有两个版本：

+   在第一个版本中，您可以创建在 .NET Framework 上运行的函数。

+   在第二个版本中，您可以创建在 .NET Core 上运行的函数。

作为软件架构师，您始终需要考虑代码的可重用性。在这种情况下，您应该注意您将决定在哪个版本的 Azure Functions 项目中构建函数。

默认情况下，生成的代码与您在 Azure Portal 中创建 Azure Functions 时生成的代码类似。发布方法遵循我们在[第 1 章](14b5c5da-4042-439e-9e5a-2e19ba4c4930.xhtml)，*理解软件架构的重要性*中描述的 Web 应用程序发布过程相同的步骤。

# 列出 Azure Functions 模板

Azure Portal 中有多个模板可供您创建 Azure Functions。您可以选择的模板数量会持续更新。以下只是其中的一些：

+   **Blob 存储**：您可能希望在文件上传到您的 blob 存储后立即处理该文件。这可能是 Azure 函数的一个很好的用例。

+   **Cosmos DB**：您可能希望将到达 Cosmos DB 数据库中的数据与处理方法同步。Cosmos DB 在[第 7 章](77cdecb5-cef4-4b02-80a1-052ad366b9f3.xhtml)，*如何在云中选择您的数据存储*中进行了详细讨论。

+   **事件网格**：这是一种管理 Azure 事件的好方法。函数可以被触发以管理每个事件。

+   **事件中心**：这些可以与 Azure Functions 一起使用来管理每个连接设备到达的数据。

+   **HTTP**：这个触发器对于构建无服务器API和Web应用事件非常有用。

+   **Microsoft Graph Events**：Graph API允许您提供与Office 365相关的功能。例如，使用此触发器，您可以将日历事件连接到函数。

+   **队列存储**：您可以使用函数作为服务解决方案来处理队列处理。

+   **服务总线**：这是另一种可以触发函数的消息服务。Azure服务总线将在第9章中更详细地介绍，*设计模式和.NET Core实现*。

+   **定时器**：这通常与函数一起使用，您在这里指定时间触发器，以便可以持续处理系统中的数据。

+   **WebHooks**：WebHooks是一种允许您的应用程序避免从API中池化数据的技术。您可以将它们连接到函数，以了解您已挂钩的事件是如何被处理的。

# 维护Azure Functions

一旦您创建并编程了函数，您就需要监控和维护它。为此，您可以使用各种工具——所有这些工具您都可以在Azure门户中找到。这些工具将帮助您解决由于您将能够收集的大量信息而引起的问题。

当您需要监控函数时，第一个选项是使用Azure门户中的Azure Functions界面内的监控菜单。在那里，您将能够检查所有函数执行情况，包括成功的结果和失败：

![](img/457f702c-541d-40fe-a83a-06726885644a.png)

等待结果大约需要5分钟。网格中显示的日期是协调世界时（UTC）。

同一个界面允许您连接到应用洞察（Application Insights）。这将带您进入一个几乎无限的选择世界，您可以使用这些选择来分析您的函数数据。应用洞察（Application Insights）是目前可用的最佳**应用性能管理（APM**）系统之一：

![](img/dda9710f-d76d-49dd-b6d7-505cefce5689.png)

除了查询界面之外，您还可以使用Azure门户中的洞察界面检查您函数的所有性能问题。在那里，您可以分析和筛选您的解决方案接收到的所有请求，并检查它们的性能和依赖关系。您还可以在您的端点之一发生异常时触发警报：

![](img/69145b32-05c8-4d37-8782-e2f68c52eebf.png)

作为软件架构师，您将在项目中找到这个工具的真正有用的日常助手。值得一提的是，应用洞察（Application Insights）还适用于其他几个Azure服务，如Web应用和虚拟机。这意味着您可以使用Azure提供的出色功能来监控和维护您系统的健康状态。

# 用例 - 实现Azure Functions发送电子邮件

在这里，我们将使用之前描述的 Azure 组件的子集。WWTravelClub 的用例提出了服务的全球实施，并且有可能这个服务需要不同的架构设计来应对我们在[第 1 章](14b5c5da-4042-439e-9e5a-2e19ba4c4930.xhtml)中描述的所有性能关键点，即*理解软件架构的重要性*。

如果你回顾一下在[第 1 章](14b5c5da-4042-439e-9e5a-2e19ba4c4930.xhtml)中描述的用户故事，即*理解软件架构的重要性*，你会发现许多需求都与通信相关。正因为如此，在解决方案中通过电子邮件提供一些警报是非常常见的。本章的用例将专注于如何发送电子邮件。架构将是完全无服务器的。

以下图表显示了架构的基本结构。为了给用户提供良好的体验，应用程序发送的所有电子邮件都将异步排队，从而避免系统响应中的高延迟：

![](img/3916fea9-5954-43af-926e-4156542b61ef.png)

注意，没有服务器来管理 Azure Functions 以插入消息到队列存储，也没有服务器来从队列存储获取消息。这正是我们所说的无服务器。值得一提的是，这种架构不仅限于发送电子邮件，还可以用于处理任何 HTTP POST 请求。

现在，我们将学习如何在 API 中设置安全设置，以便只有授权的应用程序可以使用给定的解决方案。

# 第一步 – 创建 Azure 队列存储

在 Azure Portal 中创建存储非常简单。让我们来学习如何操作：

1.  首先，你需要创建一个存储账户并设置其名称、安全性和网络，如下面的截图所示：

![](img/44690125-2574-4cd9-a1c7-144b8c084895.png)

1.  一旦你设置了存储账户，你将能够设置一个队列。你只需要提供队列的名称：

![](img/b50cc828-03a4-44db-ac6a-b810319c49ba.png)

1.  创建的队列将为你提供 Azure Portal 的概览。在那里，你可以找到你的队列 URL 并使用 Storage Explorer：

![](img/e1cd3823-bfe2-47a8-a94b-d1198341894c.png)

注意，你还可以使用 Microsoft Azure Storage Explorer ([https://azure.microsoft.com/en-us/features/storage-explorer/](https://azure.microsoft.com/en-us/features/storage-explorer/)) 来连接到此存储：

![](img/1c874dcd-a218-46ba-890b-82167a3f9552.png)

1.  现在，你可以开始你的函数式编程，通知队列有电子邮件等待发送。在这里，我们需要使用 HTTP 触发器。请注意，该函数是一个静态类，它异步运行。以下代码正在收集来自 HTTP 触发器的请求数据并将其插入到稍后处理的队列中：

[PRE0]

1.  你可以使用 Postman 这样的工具通过运行 Azure Functions Emulator 来测试你的函数：

![](img/e39b6278-ff70-4d79-8675-95c855e8f3d8.png)

1.  结果将出现在 Microsoft Azure 存储资源管理器和 Azure 门户中。在 Azure 门户中，你可以管理每条消息，对每条消息进行出队，甚至清除队列存储：

![图片](img/404c9bbe-33ed-48aa-a26b-9e39cd8c13ff.png)

1.  然后，你可以创建第二个函数。这个函数将由进入你的队列的数据触发。值得注意的是，对于 Azure Functions v2，你需要将 `Microsoft.Azure.WebJobs.Extensions.Storage` 库作为 NuGet 引用添加：

![图片](img/62b1db91-f9bd-476b-9703-a4dc9eca5b9f.png)

1.  一旦你在 `local.settings.json` 中设置了连接字符串，你将能够运行这两个函数，并使用 Postman 测试它们。区别在于，当第二个函数正在运行时，如果你在它的开始处设置断点，你会检查消息是否已发送：

![图片](img/5b1b32ef-bf7e-4be0-bafd-aef379ccc879.png)

1.  从这个点开始，发送邮件的方式将取决于你拥有的邮件选项。你可能决定使用代理，或者直接连接到你的电子邮件服务器。

以这种方式创建电子邮件服务有几个优点：

+   一旦你的服务被编码和测试，你就可以使用它从你的任何应用程序中发送电子邮件。这意味着你的代码始终可以重用。

+   使用此服务的应用程序不会因为 HTTP 服务的异步优势而停止发送电子邮件。

+   你不需要对队列进行池化以检查数据是否准备好处理。

最后，队列处理是并发运行的，这在大多数情况下提供了更好的体验。你可以通过在 `host.json` 中设置一些属性来关闭它。所有这些选项都可以在本章末尾的 *进一步阅读* 部分找到。

# 摘要

在本章中，我们探讨了使用无服务器 Azure Functions 开发功能的一些优点。你可以将其用作检查 Azure Functions 中可用的不同类型触发器的指南，以及规划如何监控它们的计划。我们还看到了如何编程和维护 Azure 函数。最后，我们查看了一个连接多个函数以避免数据池化和启用并发处理的架构示例。

在下一章中，我们将分析设计模式的概念，了解为什么它们如此有用，以及一些常见的模式。

# 问题

1.  什么是 Azure Functions？

1.  Azure Functions 的编程选项有哪些？

1.  可以与 Azure Functions 一起使用的计划有哪些？

1.  如何使用 Visual Studio 部署 Azure Functions？

1.  你可以使用哪些触发器来开发 Azure Functions？

1.  Azure Functions v1 和 v2 之间的区别是什么？

1.  Application Insights 如何帮助我们维护和监控 Azure Functions？

# 进一步阅读

如果你想了解更多关于创建 Azure Functions 的信息，请查看以下链接：

+   Azure Functions的扩展和托管：[https://docs.microsoft.com/en-us/azure/azure-functions/functions-scale](https://docs.microsoft.com/en-us/azure/azure-functions/functions-scale)

+   Praveen Kumar Sreeram所著的《Azure Functions - Essentials [Video]》：[https://www.packtpub.com/virtualization-and-cloud/azure-functions-essentials-video](https://www.packtpub.com/virtualization-and-cloud/azure-functions-essentials-video)

+   介绍Azure Functions 2.0：[https://azure.microsoft.com/en-us/blog/introducing-azure-functions-2-0/](https://azure.microsoft.com/en-us/blog/introducing-azure-functions-2-0/)

+   Azure Event Grid概述：[https://azure.microsoft.com/en-us/resources/videos/an-overview-of-azure-event-grid/](https://azure.microsoft.com/en-us/resources/videos/an-overview-of-azure-event-grid/)

+   Azure Functions的定时器触发器：[https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer)

+   Ritesh Modi所著的《Azure for Architects》一书中关于Application Insights的部分：[https://subscription.packtpub.com/book/virtualization_and_cloud/9781788397391/12/ch12lvl1sec95/application-insights](https://subscription.packtpub.com/book/virtualization_and_cloud/9781788397391/12/ch12lvl1sec95/application-insights)

+   从Praveen Kumar Sreeram所著的《Azure Serverless Computing Cookbook》一书中关于使用Application Insights监控Azure Functions的部分：[https://subscription.packtpub.com/book/virtualization_and_cloud/9781788390828/6/06lvl1sec34/monitoring-azure-functions-using-application-insights](https://subscription.packtpub.com/book/virtualization_and_cloud/9781788390828/6/06lvl1sec34/monitoring-azure-functions-using-application-insights)

+   使用.NET开始使用Azure Queue存储：[https://docs.microsoft.com/en-us/azure/storage/queues/storage-dotnet-how-to-use-queues](https://docs.microsoft.com/en-us/azure/storage/queues/storage-dotnet-how-to-use-queues)

+   Azure Functions触发器和绑定概念：[https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings](https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings)

+   Azure Functions的Azure Queue存储绑定：[https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue)
