# *第 6 章*：使用不同的托管模型

在本章中，我们将讨论如何在 ASP.NET Core 中自定义托管。我们将探讨托管选项和不同类型的托管，并简要介绍在 IIS 上的托管。本章只是一个概述。对于每个主题，都有可能进行更深入的探讨，但这将需要一本完整的书来阐述！

在本章中，我们将涵盖以下主题：

+   设置 `WebHostBuilder`

+   设置 Kestrel

+   设置 `HTTP.sys`

+   在 IIS 上托管

+   在 Linux 上使用 Nginx 或 Apache

本章中的主题涉及 ASP.NET Core 架构的托管层：

![图 6.1 – ASP.NET Core 架构

![图 6.1 – ASP.NET Core 架构](img/Figure_6.1_B17996.jpg)

图 6.1 – ASP.NET Core 架构

本章探讨了以下服务器架构主题：

![图 6.2 – ASP.NET 服务器架构

![图 6.2 – ASP.NET 服务器架构](img/Figure_6.2_B17996.jpg)

图 6.2 – ASP.NET 服务器架构

# 技术要求

对于本章，我们只需要设置一个小型的空 Web 应用程序：

[PRE0]

就这些了。用 Visual Studio Code 打开它：

[PRE1]

*Et voilà*！一个简单的项目在 Visual Studio Code 中打开。

本章的代码可以在 GitHub 上找到：[https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/tree/main/Chapter06](https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/tree/main/Chapter06)。

# 设置 WebHostBuilder

与上一章一样，在本节中我们将重点关注 `Program.cs`。`WebHostBuilder` 是我们的朋友。这是配置和创建 Web 服务器的地方。

以下代码片段是使用 .NET CLI 的 `dotnet new` 命令创建的每个新 ASP.NET Core Web 项目的默认配置：

[PRE2]

如我们从之前的章节中已经了解到的，默认构建器已经预先配置了所有必要的功能。为了在 Azure 或本地 IIS 上成功运行应用程序，所有需要配置的内容都已为您配置好。

但是，您可以覆盖几乎所有这些默认配置，包括托管配置。

接下来，让我们设置 Kestrel。

# 设置 Kestrel

在创建 `WebHostBuilder` 之后，我们可以使用各种功能来配置构建器。在这里，我们可以看到其中之一，它指定了应该使用的 `Startup` 类。

注意

如在 [*第 4 章*](B17996_04_ePub.xhtml#_idTextAnchor066) 中讨论的，*使用 Kestrel 配置和自定义 HTTPS*，Kestrel 是托管应用程序的一种可能选择。Kestrel 是内置在 .NET 中并基于 .NET 套接字实现的 Web 服务器。之前，它是基于 **libuv** 构建的，这是 Node.js 使用的相同 Web 服务器。Microsoft 移除了对 **libuv** 的依赖，并基于 .NET 套接字创建了自己的 Web 服务器实现。

在上一章中，我们看到了 `UseKestrel` 方法来配置 Kestrel 选项：

[PRE3]

第一个参数是`WebHostBuilderContext`，用于访问已配置的托管设置或配置本身。第二个参数是一个用于配置Kestrel的对象。这个代码片段显示了我们在上一章中配置套接字端点时所做的工作，主机需要监听这些端点：

[PRE4]

（你可能需要向`System.Net`添加一个`using`语句。）

这将覆盖默认配置，你可以传递URL，例如，使用`launchSettings.json`的`applicationUrl`属性或环境变量。

现在，让我们看看如何设置`HTTP.sys`。

# 设置HTTP.sys

另外还有一个托管选项，一个不同的Web服务器实现。`HTTP.sys`是Windows内部一个相当成熟的库，可以用来托管你的ASP.NET Core应用程序：

[PRE5]

`HTTP.sys`与Kestrel不同。它不能用于IIS，因为它与IIS的ASP.NET Core模块不兼容。

使用`HTTP.sys`而不是Kestrel的主要原因是在不需要IIS的情况下，你需要将你的应用程序暴露于互联网。

注意

IIS已经在`HTTP.sys`上运行多年。这意味着`UseHttpSys()`和IIS使用相同的Web服务器实现。要了解更多关于`HTTP.sys`的信息，请阅读文档，相关链接可以在*进一步阅读*部分找到。

接下来，让我们看看如何使用IIS进行托管。

# 在IIS上托管

ASP.NET Core应用程序不应当直接暴露于互联网，即使它支持Kestrel或`HTTP.sys`。最好在两者之间有一个反向代理，或者至少有一个监控托管进程的服务。对于ASP.NET Core来说，IIS不仅仅是一个反向代理。它还负责托管进程，以防因错误而中断。如果发生这种情况，IIS将重新启动进程。在Linux上，Nginx可以用作反向代理，同时也负责托管进程。

注意

确保你创建了一个新项目或移除了上一节中Kestrel的配置。这不会与IIS一起工作。

要在IIS或Azure上托管ASP.NET Core Web，你需要先发布它。发布不仅编译项目；它还准备项目在IIS、Azure或Linux上的Web服务器（如Nginx）上托管。

以下命令将发布项目：

[PRE6]

在系统浏览器中查看时，它应该如下所示：

![图6.3 – .NET发布文件夹](img/Figure_6.3_B17996.jpg)

图6.3 – .NET发布文件夹

这会产生一个可以在IIS中映射的输出。它还创建了一个`web.config`文件，用于添加IIS或Azure的设置。它包含了一个作为DLL编译的Web应用程序。

如果你发布了一个自包含的应用程序，它也包含了运行时本身。自包含的应用程序会自带.NET Core运行时，但交付的大小会增加很多。

在IIS上？只需创建一个新的Web，并将其映射到放置发布输出的文件夹：

![图6.4 – .NET发布对话框](img/Figure_6.4_B17996.jpg)

图6.4 – .NET发布对话框

如果您需要更改安全性，如果您有一些数据库连接等，事情会变得稍微复杂一些。这可以是一个单独章节的主题。

![图6.5 – 在浏览器中查看的“Hello World!”](img/Figure_6.5_B17996.jpg)

图6.5 – 在浏览器中查看的“Hello World!”

*图6.5*显示了示例项目中`Program.cs`中小的`MapGet`的输出：

[PRE7]

接下来，我们将讨论一些Linux的替代方案。

# 在Linux上使用Nginx或Apache

在Linux上发布ASP.NET Core应用程序看起来与在IIS上看起来非常相似，但为反向代理做准备需要一些额外的步骤。您需要一个像Nginx或Apache这样的反向代理服务器，它将流量转发到Kestrel和ASP.NET Core应用程序：

1.  首先，您需要允许您的应用程序接受两个特定的转发头。为此，打开`Startup.cs`，并在`UseAuthentication`中间件之前将以下行添加到`Configure`方法中：

    [PRE8]

1.  您还需要信任来自反向代理的传入流量。这需要您将以下行添加到`ConfigureServices`方法中：

    [PRE9]

    您可能需要向`Microsoft.AspNetCore.HttpOverrides`添加一个`using`。

1.  在这里添加代理的IP地址。这只是一个示例。

1.  然后，您需要发布应用程序：

    [PRE10]

1.  将构建输出复制到名为`/var/www/yourapplication`的文件夹中。您还应该在Linux上通过调用以下命令进行快速测试：

    [PRE11]

1.  在这里，`yourapplication.dll`是编译后的应用程序，包括路径。如果一切正常，您应该能够在`http://localhost:5000/`上调用您的Web应用程序。

    如果一切正常，应用程序应作为服务运行。这需要您在`/etc/systemd/system/`上创建一个服务文件。将文件命名为`kestrel-yourapplication.service`，并在其中放置以下内容：

    [PRE12]

    确保第5行和第6行中的路径指向您放置构建输出的文件夹。此文件定义了您的应用程序应在默认端口上作为服务运行。它还监视应用程序，并在它崩溃时重新启动它。它还定义了传递给配置应用程序的环境变量。请参阅[*第1章*](B17996_01_ePub.xhtml#_idTextAnchor019)，*自定义日志记录*，了解如何使用环境变量配置您的应用程序。

接下来，我们将看到如何配置Nginx。

## 配置Nginx

现在，您可以使用以下代码告诉Nginx要做什么：

[PRE13]

这告诉Nginx将端口`80`上的调用转发到`example.com`，以及它的子域名到`http://localhost:5000`，这是您应用程序的默认地址。

## 配置Apache

Apache的配置看起来与Nginx方法非常相似，并在最后做同样的事情：

[PRE14]

Nginx和Apache就到这里。现在让我们总结这一章。

# 摘要

ASP.NET Core 和 .NET CLI 已经包含了所有工具，可以将它们部署到各种平台，并设置好以准备在 Azure、IIS 以及 Nginx 上运行。这非常简单，在文档中有详细描述。

目前，我们拥有 `WebHostBuilder`，它可以创建应用程序的托管环境。在3.0版本中，我们有了 `HostBuilder`，它能够创建一个完全独立于任何Web上下文的托管环境。

ASP.NET Core 6.0 具有在应用程序内部运行后台任务的功能。要了解更多信息，请阅读下一章。

# 进一步阅读

更多信息，您可以参考以下链接：

+   **Kestrel 文档**：[https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-6.0](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-6.0)

+   **HTTP.sys 文档**：[https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/httpsys?view=aspnetcore-6.0](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/httpsys?view=aspnetcore-6.0)

+   **ASP.NET Core**：[https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/aspnet-core-module?view=aspnetcore-6.0](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/aspnet-core-module?view=aspnetcore-6.0)
