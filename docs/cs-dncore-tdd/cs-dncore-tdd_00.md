# 前言

您如何验证您的跨平台.NET Core 应用程序在部署到任何地方时都能正常工作？随着业务、团队和技术环境的发展，您的代码能够随之发展吗？通过遵循测试驱动开发的原则，您可以简化代码库，使查找和修复错误变得微不足道，并确保您的代码能够按照您的想法运行。

本书指导开发人员通过建立专业的测试驱动开发流程来创建健壮、可投入生产的 C# 7 和.NET Core 应用程序。为此，您将首先学习 TDD 生命周期的各个阶段、一些最佳实践和一些反模式。

在第一章介绍了 TDD 的基础知识后，您将立即开始创建一个示例 ASP.NET Core MVC 应用程序。您将学习如何使用 SOLID 原则编写可测试的代码，并设置依赖注入。

接下来，您将学习如何使用 xUnit.net 测试框架创建单元测试，以及如何使用其属性和断言。一旦掌握了基础知识，您将学习如何创建数据驱动的单元测试以及如何在代码中模拟依赖关系。

在本书的最后，您将通过使用 GitHub、TeamCity、VSTS 和 Cake 来创建一个健康的持续集成流程。最后，您将修改持续集成构建，以测试、版本化和打包一个示例应用程序。

# 本书适合对象

本书适用于希望通过实施测试驱动开发原则构建质量、灵活、易于维护和高效企业应用程序的.NET 开发人员。

# 本书涵盖内容

第一章，“探索测试驱动开发”，向您介绍了如何通过学习和遵循测试驱动开发的成熟原则来改善编码习惯和代码。

第二章，“使用.NET Core 入门”，向您介绍了.NET Core 和 C# 7 的超酷新跨平台功能。我们将通过实际操作来学习，在 Ubuntu Linux 上使用测试驱动开发原则创建一个 ASP.NET MVC 应用程序。

第三章，“编写可测试的代码”，演示了为了获得测试驱动开发周期的好处，您必须编写可测试的代码。在本章中，我们将讨论创建可测试代码的 SOLID 原则，并学习如何为依赖注入设置我们的.NET Core 应用程序。

第四章，“.NET Core 单元测试”，介绍了.NET Core 和 C#可用的单元测试框架。我们将使用 xUnit 框架创建一个共享的测试上下文，包括设置和清除代码。您还将了解如何创建基本的单元测试，并使用 xUnit 断言来证明单元测试的结果。

第五章，“数据驱动的单元测试”，介绍了允许您通过一系列数据输入来测试代码的概念，可以是内联的，也可以来自数据源。在本章中，我们将创建 xUnit 中的数据驱动单元测试或理论。

第六章，“模拟依赖关系”，解释了模拟对象是模仿真实对象行为的模拟对象。在本章中，您将学习如何使用 Moq 框架，使用 Moq 创建的模拟对象来隔离您正在测试的类与其依赖关系。

第七章，*持续集成和项目托管*，侧重于测试驱动开发周期的目标，即快速提供有关代码质量的反馈。持续集成流程将这种反馈周期延伸到发现代码集成问题。在本章中，您将开始创建一个持续集成流程，该流程可以为开发团队提供有关代码质量和集成问题的快速反馈。

第八章，*创建持续集成构建流程*，解释了一个出色的持续集成流程将许多不同的步骤整合成一个易于重复的流程。在本章中，您将配置 TeamCity 和 VSTS 使用跨平台构建自动化系统 Cake 来清理、构建、恢复软件包依赖关系并测试您的解决方案。

第九章，*测试和打包应用程序*，教您修改 Cake 构建脚本以运行 xUnit 测试套件。您将通过为.NET Core 支持的各种平台版本化和打包应用程序来完成该过程。

# 为了充分利用本书

假定您具有 C#编程和 Microsoft Visual Studio 的工作知识。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

文件下载后，请确保使用最新版本的解压缩或提取文件夹：

+   Windows 的 WinRAR/7-Zip

+   Mac 的 Zipeg/iZip/UnRarX

+   Linux 的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址是[`github.com/PacktPublishing/CSharp-and-.NET-Core-Test-Driven-Development`](https://github.com/PacktPublishing/CSharp-and-.NET-Core-Test-Driven-Development)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有其他代码包，来自我们丰富的图书和视频目录，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。快去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/CSharpanddotNETTestDrivenDevelopment_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/CSharpanddotNETTestDrivenDevelopment_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“为了使测试通过，您必须迭代实现生产代码。当实现以下`IsServerOnline`方法时，预计`Test_IsServerOnline_ShouldReturnTrue`测试方法将通过。”

代码块设置如下：

```cs
[Fact]
 public void Test_IsServerOnline_ShouldReturnTrue() 
 { 
    bool isOnline=IsServerOnline();   

    Assert.True(isOnline);
 }
```

任何命令行输入或输出都是按照以下格式编写的：

```cs
sudo apt-get update
sudo apt-get install dotnet-sdk-2.0.0
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会以这种方式出现在文本中。这是一个例子：“Visual Studio Code 将尝试下载 Linux 平台所需的依赖项，Linux 的 Omnisharp 和.NET Core 调试器。”

警告或重要说明看起来像这样。

技巧和窍门看起来像这样。
