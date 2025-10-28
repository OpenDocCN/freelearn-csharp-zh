# 前言

在这个不断发展的 IT 世界中，我们有众多不同的平台、框架和语言，这些平台、框架和语言基于业务需求、要求和开发者的兴趣来构建应用程序。为了消除不同平台之间的障碍，微软推出了最快、最新、最优秀的 .NET 版本，即 .NET Core 跨平台开源框架。使用这个框架，初学者级别的开发者可以在不同的平台上工作，而经验丰富的开发者和架构师可以跨平台消费他们的 API 和库。本书涵盖了使用 .NET Core 构建现代、互联网连接和基于云的应用程序的简单示例。本书将帮助我们开发简单而有趣的应用程序，为读者提供工作代码示例。我们将从一个简单的井字棋游戏开始，然后构建一个实时聊天应用程序，Let's Chat（用于与我们的在线 Facebook 朋友聊天），一个示例聊天机器人，以及一个虚拟电影预订应用程序。

# 本书面向对象

本书面向希望学习如何使用 Microsoft .NET Core 构建跨平台解决方案的开发者和架构师。假设您对 .NET Framework、面向对象编程和 C#（或类似编程语言）有一定的了解。本书对希望开发支持现有库或在不同平台上创建的库的跨平台应用程序的开发者也很有用。本书涵盖了广泛的主题，并试图解释构建 .NET Core 应用程序所需的所有基础知识。本书还介绍了 SignalR Core、Entity Framework Core、容器、F# 函数式编程以及 .NET Core 开发技巧和窍门。

# 本书涵盖内容

第一章，*入门*，讨论了本书所有示例所需的所有先决条件，例如，使用 VirtualBox 和 Windows 上的 Hyper-V 设置 Ubuntu 虚拟机，以及安装 .NET Core 2.0 和工具。我们还将简要介绍 F# 及其特性，了解 F# 与其他面向对象编程语言的不同之处，以及 C# 与 F# 之间的差异。我们还将创建一个简单的示例应用程序，使用 F#，以便熟悉 F# 语法。

第二章，*.NET Core 中的本地库*，展示了 ncurses 库是什么以及如何通过在 Linux 上实现 ncurses 本地库来扩展 .NET Core 的控制台功能，以及如何与现有的本地和 Mono 库进行交互。我们将构建一个示例本地库和应用，该应用实现了这个新的示例库。

第三章，*构建我们的第一个.NET Core 游戏 - 菲尼克斯游戏*，通过一个示例游戏应用，即井字游戏，展示了.NET Core。我们将介绍 SignalR Core 并学习它如何用于开发实时 Web 应用。玩家可以使用自己的图像而不是单色的 X 和 O 来玩游戏。通过这个示例游戏应用，我们将了解代码（类、接口、模型等）的概览，并学习关于编译、构建和测试应用程序的知识，这些知识适用于.NET Core。

第四章，*Let's Chat Web 应用*，介绍了使用 ASP.NET Core 进行 Web 开发。这是通过 Windows 上的简单聊天应用 Let's Chat 来实现的。我们还涵盖了项目设置、应用程序架构及其描述、SignalR Core、依赖注入、配置、日志记录等内容。我们还将熟悉在开发应用程序组件时引入的 ASP.NET Core 功能的基本原理。

第五章，*开发 Let's Chat Web 应用*，通过大量的示例和代码片段来展示 ASP.NET Core 的基本原理和功能。到本章结束时，Let's Chat 应用将准备就绪供使用。我们还将熟悉测试、托管、安全性和 Web 开发的性能方面。

第六章，*测试和部署 - Let's Chat Web 应用*，解释了.NET Core 的部署模型。我们将部署 Let's Chat 应用。我们将介绍 Docker 容器。我们还将开发并部署一个基于 ASP.NET Core 的聊天机器人，并将其集成到 Let's Chat 应用中。

第七章，*迈向云端*，教您什么是云以及为什么现代开发者应该熟悉云技术。我们将从 Azure 开始，并了解 Azure 管理门户。我们将学习如何从门户创建虚拟机（VM），并看到它是通过 PowerShell 或其他语言使用 Azure SDK 自动化的。我们将使用 PowerShell 管理 VM，并了解如何启动和停止它。我们还将学习如何从 Visual Studio 2017 本身创建 Azure 中的 Web 应用，并学习如何发布配置文件。最后，我们将概述 App 服务，并快速查看 Azure 存储。

第八章，*电影预订 Web 应用*，讨论了**实体框架**（**EF**）和实体框架核心。我们将了解这两个框架的功能和区别。我们还将了解只有在 EF 无法使用或存在迫切的跨平台需求时才应使用 EF Core。我们将通过创建一个简单的应用来学习如何使用 EF Core 执行 CRUD 操作。然后我们将开发一个简单的电影预订应用，并学习如何使用 Visual Studio 部署它。我们还将看到如何通过启用 Application Insights 来监控我们的 Web 应用。

第九章，*使用.NET Core 进行微服务*，概述了微服务架构以及它如何是 SOA 的扩展并克服了传统单体应用的限制。我们将学习 ASP.NET 和 ASP.NET Core 之间的重要架构差异。我们将探讨在开发 ASP.NET Core 2.0 应用程序时需要注意的提示，因为这些架构差异。然后我们将讨论有助于提高 ASP.NET Core 应用程序性能的实用信息、步骤和提示，以及一些关于 Azure 的提示，然后是 ASP.NET Core 团队的新实验项目，该项目被称为 Blazor。我们将通过讨论.NET Core 2.1 路线图和预期在新版本中出现的功能来结束本章。

第十章，*使用 F#进行函数式编程*，讨论了函数式编程及其特性，如高阶函数、纯度、惰性评估和柯里化。我们将学习 F#的基础知识，例如类、`let`和`do`绑定、泛型类型参数、F#中的属性，如何在 F#中编写函数和 lambda 表达式，以及异常处理。我们还将探讨 F#中的不同类型的数据提供者以及不同类型的数据解析器是如何工作的。我们还将学习如何使用 F#查询 SQL Server vNext。

# 充分利用这本书

本书面向使用不同平台（Windows、Linux、Ubuntu、macOS）的有经验开发者，他们想尝试 Microsoft .NET Core 2.0 的跨平台开发。使用 C#、C 或 C++进行开发的开发者，对扩展他们的函数式编程知识感兴趣，希望了解 F#和函数式编程的初学者，我们假设您对 C#有基本的了解，并且熟悉.NET 框架和 Windows。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载这本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”标签。

1.  点击“代码下载与勘误表”。

1.  在搜索框中输入书名，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本的软件解压缩或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/.NET-Core-2.0-By-Example`](https://github.com/PacktPublishing/.NET-Core-2.0-By-Example)。我们还有其他来自我们丰富图书和视频目录的代码包可供选择，网址为**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。请查看它们！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/.NETCore2.0ByExample_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/.NETCore2.0ByExample_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“通过在 `_Layout.cshtml` 中这样做，我们确保了所有页面都具有此功能。”

代码块设置如下：

```cs
# include<stdio.h>

int hello()
{
  return 15;
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```cs
else
       {
           app.UseExceptionHandler("/Home/Error");
       }
       app.UseStaticFiles();
       app.UseMvc(routes =>
 {
```

任何命令行输入或输出都按以下方式编写：

```cs
mcs -out:helloNative.exe InteropWithNativeSO.cs
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“输入显示名称和联系电子邮件，然后单击“创建应用 ID”按钮。”

警告或重要注意事项如下所示。

小贴士和技巧如下所示。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请将电子邮件发送至 `feedback@packtpub.com`，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过 `questions@packtpub.com` 发送电子邮件给我们。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一错误。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，我们将不胜感激，如果您能提供位置地址或网站名称。请通过 `copyright@packtpub.com` 联系我们，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用过这本书，为何不在您购买它的网站上留下评论？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/).[packtpub.com]
