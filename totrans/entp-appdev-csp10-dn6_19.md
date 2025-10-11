# *第14章*：健康和诊断

现代软件应用程序已经演变成复杂和动态，并且具有分布式特性。对这些应用程序的需求是能够全天候在任何设备上工作。为了实现这一点，重要的是要知道我们的应用程序始终可用并响应请求。客户体验将在服务未来的发展和组织的收入中扮演重要角色。

一旦应用程序上线，监控应用程序的健康状况至关重要。定期监控应用程序健康将帮助我们主动检测任何故障，并在它们造成更多损害之前解决它们。应用程序监控现在已成为日常运营的一部分。为了诊断在线应用程序上的任何故障，我们需要拥有正确的遥测和诊断工具。我们捕获的遥测数据也将帮助我们识别那些用户没有直接看到或报告的问题。

让我们了解应用程序健康监控以及.NET 6提供了哪些功能。

本章我们将学习以下主题：

+   介绍健康检查

+   ASP.NET Core 6中的健康检查API

+   使用Application Insights监控应用程序

+   执行远程调试

到本章结束时，您将能够熟练构建.NET 6应用程序的健康检查API，并使用Azure Application Insights来捕获遥测信息和诊断问题。

# 技术要求

为了完成本章中的任务，您需要以下软件：

+   安装了Azure开发工作负载的Visual Studio 2022 Community Edition（某些部分需要企业版）

+   Azure订阅

预期您对Microsoft .NET有基本了解，以及如何在Azure中创建资源。

本章中使用的代码可以在[https://github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition/tree/main/Enterprise%20Application](https://github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition/tree/main/Enterprise%20Application)找到。

# 介绍健康检查

健康检查是对应用程序的全面审查，帮助我们了解应用程序的当前状态，并使用可见指标采取纠正措施。健康检查由应用程序作为HTTP端点公开。健康检查端点用作某些编排器和负载均衡器的健康探测，以将流量从失败的节点路由出去。健康检查用于监控应用程序依赖项，如数据库、外部服务和缓存服务。

在下一节中，我们将学习如何在ASP.NET Core 6中构建健康检查API的支持。

# ASP.NET Core 6中的健康检查API

ASP.NET Core 6有一个内置的中间件（通过`Microsoft.Extensions.Diagnostics.HealthChecks` NuGet包提供），用于报告作为HTTP端点公开的应用程序组件的健康状态。这个中间件使得集成数据库、外部系统和其他依赖项的健康检查变得非常简单。它也是可扩展的，因此我们可以创建我们自己的自定义健康检查。

在下一节中，我们将向我们的`Ecommerce`门户添加一个健康检查端点。

## 添加健康检查端点

在本节中，我们将向我们的`Packt.Ecommerce.Web`应用程序添加一个健康检查端点：

1.  为了添加一个健康检查端点，我们首先需要将`Microsoft.Extensions.Diagnostics.HealthChecks` NuGet包引用添加到`Packt.Ecommerce.Web`项目中，如下面的截图所示：

![Figure 14.1 – NuGet引用，Microsoft.Extensions.Diagnostics.HealthChecks

![img/Figure_14.1_B18507.jpg](img/Figure_14.1_B18507.jpg)

图14.1 – NuGet引用，Microsoft.Extensions.Diagnostics.HealthChecks

现在，我们需要将`HealthCheckService`注册到依赖容器中。`Microsoft.Extensions.Diagnostics.HealthChecks`包定义了`AddHealthChecks`扩展方法，用于将`HealthCheckService`添加到容器中。我们可以从`Program.cs`文件中调用`AddHealthChecks`方法来添加`DefaultHealthCheckService`模块：

[PRE0]

1.  让我们继续配置健康检查端点到我们的Web应用程序。使用`MapHealthChecks`方法映射健康端点，如下面的代码所示，在`Program.cs`文件中。这将添加健康检查端点路由到应用程序。这将内部配置`HealthCheckResponseWriters.WriteMinimalPlainText`框架方法以输出响应。`WriteMinimalPlainText`将仅输出健康检查服务的总体状态：

    [PRE1]

1.  运行应用程序并浏览到`<<Application URL>>/health` URL。您将看到以下输出：

![Figure 14.2 – 健康检查端点响应

![img/Figure_14.2_B18507.jpg](img/Figure_14.2_B18507.jpg)

图14.2 – 健康检查端点响应

我们添加的健康端点提供了关于服务可用性的基本信息。在下一节中，我们将看到如何监控依赖服务的状态。

## 监控依赖URI

企业应用程序依赖于多个其他组件，例如数据库和包括`KeyVault`在内的Azure组件，以及其他微服务（例如我们的`Ecommerce`网站）依赖于订单服务、产品服务等。这些服务可能属于同一组织内的其他团队，或者在某些情况下，它们可能是外部服务。通常，监控依赖服务是一个好主意。我们可以利用`AspNetCore.HealthChecks.Uris` NuGet包来监控依赖服务的可用性。

让我们继续增强我们的健康端点以监控产品和订单服务：

1.  将NuGet包引用添加到`AspNetCore.HealthChecks.Uris`。现在，修改健康检查注册以注册产品和订单服务，如下面的代码片段所示：

    [PRE2]

健康检查中间件还提供了关于单个健康检查状态的详细信息。

1.  现在我们修改我们的健康检查中间件，以输出如下代码所示的详细信息：

    [PRE3]

在此代码中，健康检查中间件被重写，通过提供`HealthCheckOptions`和`ResponseWriter`来将其状态、健康检查持续时间、组件名称和描述作为其响应写入。

1.  现在，如果我们运行项目并导航到健康检查API，我们应该看到以下输出：

![图14.3 – 健康检查端点响应状态

](img/Figure_14.3_B18507.jpg)

图14.3 – 健康检查端点响应状态

我们已经学习了如何自定义健康检查端点的响应，以及如何利用第三方库来监控依赖URI的状态。如果您希望集成通过Entity Framework Core使用的数据库的检查，可以利用`Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore`库。有关使用此库的更多信息，请参阅[https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-6.0#entity-framework-core-dbcontext-probe](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-6.0#entity-framework-core-dbcontext-probe)。

可以在[https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks)找到针对不同服务的更广泛的健康检查包集合。在下一节中，我们将学习如何构建自定义健康检查。

## 构建自定义健康检查

ASP.NET Core 6中的健康检查中间件是可扩展的，这意味着它允许我们扩展并创建自定义健康检查。我们将通过构建进程监控器来学习如何构建和使用自定义健康检查。在某些场景中，可能需要监控机器上运行的特定进程。如果进程（例如，反恶意软件服务）没有运行，或者如果第三方SaaS服务的许可证即将到期/已到期，我们可能会将它们标记为健康问题。

让我们在`Packt.Ecommerce.Common`项目中开始创建`ProcessMonitor`健康检查：

1.  在`Packt.Ecommerce.Common`中添加一个名为`HealthCheck`的项目文件夹，并添加两个类，`ProcessMonitor`和`ProcessMonitorHealthCheckBuilderExtensions`，如下面的截图所示：

![图14.4 – 添加自定义健康检查后的项目结构

](img/Figure_14.4_B18507.jpg)

图14.4 – 添加自定义健康检查后的项目结构

自定义`HealthCheck`中间件需要NuGet引用为`Microsoft.Extensions.Diagnostics.HealthChecks`。

1.  ASP.NET Core 6中的自定义健康检查应该实现`IHealthCheck`接口。该接口定义了当请求到达健康端点API时将被调用的`CheckHealthAsync`方法。

1.  按照以下代码实现`ProcessMonitorHealthCheck`类：

    [PRE4]

在`CheckHealthAsync`方法中，根据`processName`中指定的名称获取进程列表。如果没有这样的进程，则返回健康检查失败，否则，返回状态为失败。

1.  现在我们有了自定义健康检查中间件，让我们添加一个扩展方法来注册。修改`ProcessMonitorHealthCheckBuilderExtensions`类，如下面的代码片段所示：

    [PRE5]

这是一个针对`IHealthCheckBuilder`的扩展方法。我们可以看到，在代码片段中添加`ProcessMonitorHealthCheck`会将它注册到容器中。

1.  现在我们来使用我们构建的自定义健康检查。在下面的代码中，我们为`notepad`注册了`ProcessMonitorHealthCheck`健康检查：

    [PRE6]

1.  现在，当你运行应用程序并导航到健康检查API时，如果你机器上正在运行`notepad.exe`，你将看到*图14.5*中显示的输出：

![图14.5 – 健康检查端点的响应](img/Figure_14.5_B18507.png)

图14.5 – 健康检查端点的响应

我们可以在我们的健康检查端点上启用**跨源资源共享**（**CORS**）、授权和主机限制。有关详细信息，请参阅[https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-6.0](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-6.0)。

在某些场景中，根据它们探测的应用程序状态，健康检查API被分为两种类型。它们如下：

+   **准备就绪探针**：这些指示应用程序正在正常运行，但尚未准备好接收请求。

+   **存活探针**：这些指示应用程序是否已崩溃并需要重启。

准备就绪探针和存活探针都用于控制应用程序的健康状态。失败的准备就绪探针将阻止应用程序处理流量，而失败的存活探针将重启节点。我们在Kubernetes等托管环境中使用准备就绪和存活探针。

我们已经学习了如何将健康检查API添加到ASP.NET Core 6应用程序中。在下一节中，我们将学习Azure Application Insights以及它是如何帮助监控应用程序的。

# 使用Application Insights监控应用程序

监控应用程序对于为最终用户提供一流体验至关重要。在当前这个超级快速数字市场的时代，应用程序监控对于推动业务投资回报率和保持竞争优势至关重要。我们应该关注的参数包括页面/API 性能、最常使用的页面/API、应用程序错误和系统健康等。当系统出现异常时，应设置警报，以便我们可以纠正它并最小化对用户的影响。

您已经在 [*第 7 章*](B18507_07_Epub.xhtml#_idTextAnchor596)，*在 .NET 6 中进行日志记录* 中介绍了将 Application Insights 集成到应用程序及其关键功能。让我们在 Azure 门户中打开 Application Insights 并了解其不同的功能。在概览仪表板上，除了 Azure 订阅、位置和仪器密钥外，我们还可以看到以下关键指标：

![图 14.6 – Application Insights 仪表板

](img/Figure_14.6_B18507.jpg)

图 14.6 – Application Insights 仪表板

**失败的请求** 图显示了在所选时间段内失败的请求数量。这是我们应关注的指标；许多失败表示系统不稳定。**服务器响应时间** 表示服务器对调用的大致响应时间。如果响应时间过高，更多的用户将看到应用程序响应滞后，这可能会导致用户沮丧，我们可能会因此失去用户。

**服务器请求** 图表示对应用程序的总调用次数；这将给我们展示系统使用模式。**可用性** 图表示应用程序的运行时间。在本章后面配置的可用性测试将显示 **可用性** 图。通过点击每个图表，我们可以获取与相应指标相关的更多详细信息，包括请求和异常详情。我们可以更改持续时间来查看所选间隔的图表。

概览仪表板上的图表显示最近的指标。在希望了解过去特定时间点的系统工作情况时，这可能很有用。

在下一节中，我们将了解 Application Insights 的一些最重要的功能，包括实时指标、遥测事件和远程调试功能。

## 实时指标

默认情况下启用了实时指标。实时指标以 1 秒的延迟捕获，与按时间聚合的分析指标不同。只有当实时指标面板打开时，才会流式传输实时指标的数据。收集的数据仅在图表上存在。在实时指标监控期间，所有事件都从服务器传输，并且不会被采样。如果应用程序部署在 Web 农场中，我们还可以根据服务器过滤事件。

Live Metrics 展示了各种图表，例如入站和出站请求，以及内存和 CPU 利用率的整体健康状况。在右侧面板中，我们可以看到捕获的遥测数据，它将列出请求、依赖调用和异常。Live Metrics 在我们需要通过观察失败率和性能来评估已发布到生产环境的修复方案时被利用。我们也会在运行负载测试时监控这些数据，以查看负载对系统的影响。

对于像我们的 `Ecommerce` 应用这样的应用程序，了解用户如何使用应用程序、最常用的功能和用户如何遍历应用程序非常重要。在下一节中，我们将学习如何在 Application Insights 中进行使用分析。

## 使用 Application Insights 进行使用分析

在 [*第 11 章*](B18507_11_Epub.xhtml#_idTextAnchor1228)，*创建 ASP.NET Core 6 Web 应用程序* 中，你学习了如何将 Application Insights 与视图集成。当 Application Insights 与视图集成时，它可以帮助我们深入了解人们如何使用应用程序。Application Insights 下的 **使用** 部分的 **用户** 刀片提供了有关使用应用程序的用户数量的详细信息。用户通过存储在浏览器 cookie 中的匿名 ID 来识别。请注意，使用不同浏览器和机器的单一个人被计为多个用户。**会话** 和 **事件** 刀片分别表示用户活动的会话以及某些页面或功能被使用的频率。你还可以根据你在 [*第 7 章*](B18507_07_Epub.xhtml#_idTextAnchor596)，*在 .NET 6 中记录日志* 中了解的自定义事件生成关于用户、会话和事件的报告。

在使用分析中，还有一个有趣的工具叫做 **用户流程**。**用户流程** 工具可视化用户如何在不同页面和应用程序功能之间导航。用户流程提供了在用户会话期间给定事件之前和之后发生的事件。*图 14.7* 展示了给定时间点的用户流程。这告诉我们，从主页开始，用户主要导航到 **产品详情** 页面或 **账户登录** 页面：

![图 14.7 – 我们电商应用的用户流程

](img/Figure_14.7_B18507.jpg)

图 14.7 – 我们电商应用的用户流程

让我们添加一些自定义事件，并查看用户流程与这些自定义事件之间的关系。在 `Packt.Ecommerce.Web` 应用程序的 `OrderController` 的 `Create` 动作方法中添加一个自定义事件，如下面的代码片段所示。这将跟踪用户在 **购物车** 页面上点击 **下单** 按钮时发生的自定义事件：

[PRE7]

同样，让我们添加一个自定义事件跟踪，当用户在 **产品详情** 页面上点击 **加入购物车** 按钮时。为此，添加以下代码片段：

[PRE8]

添加自定义事件后，用户流程将显示与这些事件相关的应用程序的不同活动。用户流程是一个方便的工具，可以了解有多少用户正在离开页面以及他们在页面上点击了什么。请参考章节末尾的*进一步阅读*部分提供的Azure应用洞察文档，以了解更多关于使用分析的其他有趣功能，包括群体、漏斗和保留。

当有足够的遥测事件时，你可以使用一个名为**智能检测**的应用洞察功能，它能够自动检测系统中的异常并向我们发出警报。在下一节中，我们将学习关于智能检测的内容。

## 智能检测

智能检测不需要任何配置或代码更改。它基于从系统中捕获的遥测数据工作。警报将在系统的**智能检测**选项卡下显示，并且这些警报将发送给具有**监控读取器**和**监控贡献者**角色的用户。我们可以在**设置**选项下为这些警报配置额外的收件人。一些智能检测规则包括**页面加载时间慢**、**服务器响应时间慢**、**每日数据量异常增加**和**依赖项数量下降**。

对于应用程序，我们需要监控的一个重要方面是可用性。在下一节中，我们将学习如何利用应用洞察来监控应用程序的可用性。

## 应用程序可用性

在应用洞察中，我们可以为任何从互联网可访问的`http`或`https`端点设置可用性测试。这不需要对我们的应用程序代码进行任何更改。我们可以为可用性测试配置健康检查端点`(<App Root URL>/health)`。

要配置可用性测试，请转到Azure门户中的应用洞察资源，并执行以下步骤：

1.  在**调查**菜单下选择**可用性**，如图所示：

![图14.8 – 应用洞察的可用性部分

![img/Figure_14.8_B18507.jpg]

图14.8 – 应用洞察的可用性部分

1.  点击**添加测试**以添加可用性测试，如前一张截图所示。

1.  在`Commerce可用性测试`中，选择`<<App root url>>/health`。将其他选项保留在默认值，然后点击**创建**。

1.  一旦测试配置完成，应用洞察将从所有配置的区域每5分钟调用一次配置的URL。我们可以如下查看可用性测试结果：

![图14.9 – 可用性测试结果

![img/Figure_14.9_B18507.jpg]

图14.9 – 可用性测试结果

1.  在创建测试时选择的默认区域为**巴西南部**、**东亚**、**日本东部**、**东南亚**和**英国南部**。我们可以添加或删除将在其上运行可用性测试的任何区域。建议至少配置五个区域。

1.  如果我们想在以后的时间添加新的区域，我们可以编辑可用性测试并选择新的区域（例如，**西欧**），如下面的屏幕截图所示，然后点击**保存**：

![图14.10 – 编辑可用性测试区域

![图14.10 – 示例图片](img/Figure_14.10_B18507.jpg)

![图14.10 – 编辑可用性测试区域

我们还可以在Application Insights中将多步骤Web测试配置为可用性测试。

备注

您可以使用以下文档来帮助您配置多步骤Web测试：[https://docs.microsoft.com/en-us/azure/azure-monitor/app/availability-multistep](https://docs.microsoft.com/en-us/azure/azure-monitor/app/availability-multistep)。

Application Insights提供了一个非常好的工具来查询捕获的遥测事件。在下一节中，我们将了解Application Insights中的**搜索**功能。

## 搜索

Application Insights中的**搜索**功能有助于探索遥测事件，如请求、页面视图和异常。我们还可以查询我们在应用程序中编码的跟踪。**搜索**可以从**概览**选项卡或从**调查**选项卡的**搜索**选项中打开：

![图14.11 – 搜索结果

![图14.11 – 示例图片](img/Figure_14.11_B18507.jpg)

![图14.11 – 搜索结果

使用**事务搜索**功能，我们可以根据时间和**事件类型**过滤显示的遥测事件。

我们还可以根据它们的属性进行过滤。通过点击特定事件，我们可以查看事件的全部属性以及事件的遥测信息。要查看状态代码为**500**的请求，请根据响应代码过滤事件，如下所示：

![图14.12 – 过滤搜索结果

![图14.12 – 示例图片](img/Figure_14.12_B18507.jpg)

![图14.12 – 过滤搜索结果

一旦我们应用了过滤器，在搜索结果中，我们只会看到响应代码为500的请求，如下面的屏幕截图所示：

![图14.13 – 过滤后的搜索结果

![图14.13 – 过滤后的搜索结果

图14.13 – 过滤后的搜索结果

要了解更多关于导致失败的原因，请点击事件。点击事件将显示相关遥测的详细信息，如下面的屏幕截图所示：

![图14.14 – 端到端事务详情

![图14.14 – 示例图片](img/Figure_14.14_B18507.jpg)

图14.14 – 端到端事务详情

我们甚至可以通过点击异常来进一步深入。这将显示诸如方法名称和堆栈跟踪等详细信息，这将帮助我们确定失败的原因。

使用Application Insights，我们可以对捕获的遥测数据编写自定义查询，以获得更有意义的见解。在下一节中，我们将学习如何编写查询。

## 日志

要对捕获的遥测数据进行查询，请按照以下步骤导航：

1.  前往**Application Insights** | **监控** | **日志**。这将显示**日志**页面，其中包含我们可以运行的示例查询：

![图14.15 – Application Insights日志

![图14.15 – 示例图片](img/Figure_14.15_B18507.jpg)

图14.15 – Application Insights日志

1.  在建议的样本查询中选择**请求计数趋势**。这将为我们生成一个查询并运行它。一旦运行完成，我们将看到结果和图表被填充，如下面的截图所示：

![图14.16 – 日志搜索结果

![图片](img/Figure_14.16_B18507.jpg)

图14.16 – 日志搜索结果

在Application Insights中捕获的遥测数据进入不同的表，涵盖请求、异常、依赖项、跟踪和页面视图。这里生成的查询汇总了请求表中的遥测数据，并渲染了一个时间图表，其中时间轴被分成30分钟。

我们根据需求选择时间范围。我们甚至可以在查询中指定时间范围，而不是从菜单选项中选择。这里创建的查询可以保存并在以后重新运行。这里还有一个配置警报的选项，我们曾在[*第7章*](B18507_07_Epub.xhtml#_idTextAnchor596)中了解到，即.NET 6中的*日志记录*。这里编写查询使用的语言是Kusto。

注意

有关Kusto查询语言的更多信息，请参阅以下文档：[https://docs.microsoft.com/en-us/azure/data-explorer/kusto/concepts/](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/concepts/)。

Kusto基于关系数据库结构。使用Kusto查询语言，我们可以编写复杂的分析查询。Kusto支持按组聚合、计算列和连接函数。

让我们再举一个例子，我们想要识别每个客户城市的95百分位服务响应时间。这个查询将如下编写：

[PRE9]

[PRE10]

[PRE11]

在前面的查询中，我们使用`percentile`函数来识别95百分位并按区域进行汇总。结果以条形图的形式呈现。

对于前面的查询，我们看到以下图表：

![图14.17 – Kusto百分位摘要结果

![图片](img/Figure_14.17_B18507.jpg)

图14.17 – Kusto百分位摘要结果

从渲染的图表中，我们可以推断出来自**钦奈**的请求响应时间比来自**西姆纳巴德**的请求响应时间更快。

现在，让我们找到任何导致请求失败的异常，并按请求和异常类型进行汇总。为了得到这个结果，我们将`requests`表与`exceptions`表连接，并根据请求名称和异常类型进行汇总，如下面的查询所示：

[PRE12]

[PRE13]

[PRE14]

[PRE15]

[PRE16]

[PRE17]

如果我们运行这个查询，我们会得到以下截图所示的按请求名称和异常类型汇总的结果：

![图14.18 – Kusto失败的请求异常

![图片](img/Figure_14.18_B18507.jpg)

图14.18 – Kusto失败的请求异常

搜索是Application Insights的强大功能，用于诊断和修复生产站点的故障。建议点击Application Insights的不同功能并探索它们。

当您创建Application Insights资源时，将创建一个**日志分析工作区**，该工作区将持久保存通过Application Insights捕获的遥测数据。使用**日志分析**工作区以及应用程序度量，我们还可以查询和监控与Azure资源相关的关键指标，例如在Cosmos DB中消耗的RU/s。本节中运行的所有查询都是在**日志分析**工作区中执行的。我们可以在Azure门户中创建仪表板，并将所有与我们要跟踪的应用程序关键指标相关的图表固定在仪表板上。

注意

参考Azure文档以了解更多关于日志分析的信息：[https://docs.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-overview](https://docs.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-overview)。

在本节中，我们学习了如何使用Azure Monitor监控在Azure中部署的应用程序。为了更好地分析和调试任何生产故障，我们可能想了解特定错误发生时应用程序的状态。在下一节中，我们将学习Application Insights的**快照调试器**功能如何使我们实现这一点。

# 配置快照调试器

快照调试器监控我们应用程序的异常遥测。它自动收集应用程序中发生的顶级异常的快照，包括当前源代码的状态和变量。

注意

快照调试器功能仅在Visual Studio的企业版中可用。

现在让我们继续配置我们的`Ecommerce`应用的快照调试器：

1.  将`Microsoft.ApplicationInsights.SnapshotCollector` NuGet包添加到`Packt.Ecommerce.Web`项目中。

1.  将以下`using`语句添加到`Startup.cs`中：

    [PRE18]

1.  通过在`ConfigureServices`方法中添加以下行，将快照收集器添加到您的服务中：

    [PRE19]

1.  要模拟失败，请将以下代码添加到`EcommerceService`类的`GetProductsAsync`方法中。如果存在任何产品，此代码将引发错误：

    [PRE20]

1.  现在，让我们继续运行应用程序。我们在主页上看到一个错误。刷新页面，因为调试快照是为至少发生两次的错误而设计的。

1.  现在，打开Application Insights中的**搜索**选项卡。按**异常**事件类型进行筛选：

![图 14.19 – 异常遥测

![图片](img/Figure_14.19_B18507.jpg)

图 14.19 – 异常遥测

1.  点击异常进入详情页面。在详情页面中，我们看到已经为异常创建了调试快照，如以下截图所示：

![图 14.20 – 调试快照

![图片](img/Figure_14.20_B18507.jpg)

图 14.20 – 调试快照

1.  点击**调试快照**图标。这将带我们到**调试快照**页面：

![图 14.21 – 调试快照窗口

![图片](img/Figure_14.21_B18507.jpg)

图 14.21 – 调试快照窗口

1.  要查看调试快照，需要**应用程序洞察快照调试器角色**。由于调试状态可能包含敏感信息，因此默认情况下不会添加此角色。单击**添加应用程序洞察快照调试器角色**按钮。这将向当前登录用户添加该角色。

1.  一旦角色添加完成，我们就可以在页面上看到调试快照的详细信息，以及一个下载快照的按钮：

![图 14.22 – 下载调试快照

](img/Figure_14.22_B18507.jpg)

图 14.22 – 下载调试快照

1.  单击 `diagsession`。在 Visual Studio 中打开下载的 `diagsession` 文件：

![图 14.23 – Visual Studio 中的调试快照视图

](img/Figure_14.23_B18507.jpg)

图 14.23 – Visual Studio 中的调试快照视图

1.  现在，单击 `InvalidOperationException`：

![图 14.24 – Visual Studio 中的调试快照

](img/Figure_14.24_B18507.jpg)

图 14.24 – Visual Studio 中的调试快照

在此调试会话中，我们可能需要将监视器添加到局部变量和类变量，以了解它们的状态，这将有助于调试。

注意

请参阅以下文档以了解有关快照调试器配置的更多信息：[https://docs.microsoft.com/en-us/azure/azure-monitor/app/snapshot-debugger-vm](https://docs.microsoft.com/en-us/azure/azure-monitor/app/snapshot-debugger-vm)。

随着应用程序的增长并与多个其他服务集成，将面临在生产环境中调试和解决出现的问题的挑战。在某些情况下，我们无法在预生产环境中重现它们。通过我们捕获的遥测数据和 Application Insights 提供的工具，我们将能够分析问题并解决问题。快照调试器是一个强大的工具，用于调试关键问题。Application Insights 收集遥测数据并通过后台进程批量发送。使用 Application Insights 对我们应用程序的影响很小。

可能存在我们需要调试一个运行中的应用程序的情况。使用 Visual Studio，我们可以将调试器附加到远程运行的应用程序以进行调试。在下一节中，我们将学习如何实现这一点。

# 执行远程调试

在本节中，我们将学习如何将调试器附加到我们在 Azure App Service 中部署的应用程序。使用 Visual Studio 提供的工具，远程调试应用程序很容易。在 Azure App Service 中部署应用程序的内容在 [*第 16 章*](B18507_16_Epub.xhtml#_idTextAnchor1932)，*在 Azure 中部署应用程序* 中介绍。我们可以通过执行以下操作将调试器附加到已部署的服务：

1.  通过右键单击**Packt.Ecommerce.Web**项目并从上下文菜单中选择**发布**，或通过设置**构建**|**发布****Packt.Ecommerce.Web**菜单项来启动**发布**窗口。

1.  通过在发布向导中选择 Azure App Service 资源，为 **Packt.Ecommerce.Web** 创建一个 **发布** 配置文件。

1.  创建 **发布** 配置文件后，您可以通过从 **托管** 选项中选择 **附加调试器** 来将调试器附加到在 Azure App Service 中运行的应用程序实例，如下面的屏幕截图所示：

![图 14.25 – Visual Studio 的发布窗口

](img/Figure_14.25_B18507.jpg)

图 14.25 – Visual Studio 的发布窗口

1.  一旦调试器连接，应用程序将在 Azure App Service 中通过浏览器打开。我们可以在 Visual Studio 中添加断点，并像在本地开发环境中一样调试应用程序。为了有效地调试，我们需要将应用程序的调试版本部署到 Azure App Service。

虽然这是远程部署应用程序的强大功能，但在将调试器附加到生产实例时应格外小心，因为我们将会看到实时客户数据。我们可以将调试器附加到 Azure App Service 的预发布槽位以调试和修复问题，然后从那里交换预发布槽位以将修复推广到生产。本章未涵盖 Application Insights 和 Azure Monitor 的许多其他重要功能。强烈建议在 Azure 文档中进一步探索它们。

# 摘要

本章向您介绍了使用 Application Insights 对应用程序进行健康检查和诊断问题的概念。我们学习了如何构建健康检查 API 并将健康检查模块添加到我们的 `Ecommerce` 应用程序中，这将帮助我们监控应用程序的健康状况。本章还介绍了 Azure Application Insights 的关键特性，它是一个强大的工具，用于捕获遥测数据和诊断问题。

我们学习了如何使用智能检测功能检测 Application Insights 中的异常和警报。我们还了解了快照和远程调试，这些有助于在生产环境中运行的实时应用程序中解决问题。

在下一章中，我们将学习不同的测试方法，以确保在部署到生产环境之前应用程序的质量。

# 问题

在阅读本章后，我们应该能够回答以下问题：

1.  一旦应用程序部署到生产环境，定期监控应用程序并不那么重要。对还是错？

a. 正确

b. 错误

**答案：b**

1.  自定义健康检查模块应该实现哪个接口？

a. `IHealth`

b. `IApplicationBuilder`

c. `IHealthCheck`

d. `IWebHostEnvironment`

**答案：c**

1.  在 Application Insights 中显示实时指标数据的延迟是多少？

a. 1 分钟

b. 1 秒

c. 10 秒

d. 5 秒

**答案：b**

1.  在 Application Insights 日志中编写查询所使用的查询语言是什么？

a. SQL

b. C#

c. JavaScript

d. Kusto

**答案：d**

# 进一步阅读

+   Azure 应用洞察文档：[https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
