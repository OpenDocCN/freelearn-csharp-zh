# 前言

WinUI 3 是 Windows 应用程序开发的最新桌面 UI 框架。它是微软的 Windows App SDK 的一部分，为开发者提供了使用 Fluent 设计系统构建美观应用程序的工具。本书将迅速帮助您熟悉 WinUI，以构建新的 Windows 应用程序，并使用 Blazor 和 Uno Platform 等技术构建跨平台的应用程序。

本书首先探讨了 Windows UI 开发框架的历史，以了解早期框架如何影响了今天存在的 WinUI。它涵盖了基于 XAML 的 UI 开发的基本知识，并探讨了 WinUI 中可用的控件，然后转向对 WinUI 开发者的模式和最佳实践的考察。为了帮助巩固这些概念，本书的前几章通过创建一个用于组织书籍、音乐和电影集合的应用程序来培养实际技能。每一章都会增强应用程序，讨论新的控件和概念。

书中的后几章探讨了开发者如何利用他们的 WinUI 知识来利用开源工具包，在 Windows 应用程序中集成网络内容，并将 WinUI 应用程序迁移到 Android、iOS 和 Web。本书最后教授了一些 Visual Studio 调试技术，并讨论了应用程序部署选项，以便将您的应用程序带给消费者和企业用户。在每一章的结尾，我都包含了一系列问题，供您自己尝试，以便评估您的理解程度。了解 WinUI 如何帮助您构建和部署现代、健壮的应用程序！

# 本书面向对象

本书适用于任何希望使用现代**用户体验**（**UX**）开发 Windows 应用程序的人。如果您熟悉 Windows Forms、UWP 或 WPF，并希望更新您的 Windows 开发知识或现代化现有应用程序，这本书就是为您准备的。如果您刚开始学习.NET 开发，您可以利用这本书在学习 C#和.NET 的过程中学习 XAML 开发的基础知识。

# 本书涵盖内容

*第一章*, *WinUI 简介*，考察了 Windows 中 UI 框架的历史和 WinUI 的起源，您将在 Visual Studio 中创建您的第一个 WinUI 3 项目。

*第二章*, *配置开发环境和创建项目*，解释了如何安装和配置 Visual Studio 以进行 WinUI 开发，XAML 和 C#的基本知识，并通过一个将在本书中增强的项目开始了动手实践。

*第三章*, *可维护性和可测试性的 MVVM 模式*，介绍了模型-视图-视图模型（MVVM）模式的基本知识，这是构建基于 XAML 的应用程序时最重要的设计模式之一。

*第四章*, *高级 MVVM 概念*，在 WinUI 应用程序中学习到的 MVVM 模式基础知识的基础上，介绍了更高级的技术。您将学习如何在项目中添加新的依赖项时保持组件松散耦合和可测试性。

*第五章*, *探索 WinUI 控件*，探讨了 WinUI 为构建 Windows 应用程序的开发者提供的许多控件和 API。本章探讨了 WinUI 2 和 UWP 中之前可用的全新控件和更新控件。

*第六章*, *利用数据和服务的优势*，探讨了管理数据，这是软件开发的核心部分。本章涵盖了数据管理的一些关键概念，包括状态管理和服务定位器模式。

*第七章*, *Windows 应用程序的 Fluent 设计系统*，解释了微软的 Fluent 设计系统的原则以及如何在 WinUI 应用程序中实现它们。

*第八章*, *将 Windows 通知添加到 WinUI 应用程序中*，介绍了如何利用 Windows App SDK 在 WinUI 应用程序中支持推送通知和应用程序通知。

*第九章*, *使用* *Windows* *社区工具包增强应用程序*，介绍了 Windows 社区工具包和.NET 社区工具包——为 Windows 开发者提供的开源库集合。您将学习如何在 WinUI 项目中利用工具包中的控件和辅助工具。

*第十章*, *使用模板工作室加速应用程序开发*，展示了如何利用模板工作室创建一个新的 WinUI 项目，这是一个可能令人望而却步的任务，但基于最佳的 Windows 开发模式和最佳实践。

*第十一章*, *使用 Visual Studio 调试 WinUI 应用程序*，展示了如何利用 Visual Studio 中的 XAML 调试工具追踪 WinUI 项目中的讨厌的 bug——良好的调试技能对于开发者至关重要。

*第十二章*, *在 WinUI 中托管 Blazor 应用程序*，探讨了 WinUI 中的 WebView2 控件，以及如何使用它来托管部署到云中的 Blazor 应用程序，从您的 Windows 应用程序内部进行托管。

*第十三章*, *使用 Uno Platform 将您的应用程序扩展到跨平台*，解释了如何将 WinUI 项目迁移到 Uno Platform，这允许开发者在一个代码库中编写 XAML 和 C#代码，并在任何平台上运行。

*第十四章*, *打包和部署 WinUI 应用程序*，探讨了 WinUI 开发者用于打包和部署 WinUI 应用程序的多种选项，包括通过 Microsoft Store、WinGet 和侧载应用程序进行部署。

# 为了充分利用本书

如果您熟悉 Windows Forms、.NET MAUI、UWP 或 WPF，并希望提高您对 Windows 开发的知识或现代化现有应用程序，本书将很有用。预期您具备 C# 和 .NET 的实践经验，但不需要对 WinUI 有先前的了解。

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| WinUI 3 | Windows 10 版本 1809 或更高版本或 Windows 11 |
| C# | Windows、macOS 或 Linux |
| .NET 7 | Windows、macOS 或 Linux |
| Visual Studio 2022 | Windows 10 或 11 |
| Blazor | Windows、macOS 或 Linux |
| Uno Platform | Windows、macOS 或 Linux |

*本书介绍了如何开始使用 WinUI 开发，但你应该安装 Visual Studio 和 .NET。请按照 Microsoft* *Learn:* [`learn.microsoft.com/visualstudio/install/install-visual-studio`](https://learn.microsoft.com/visualstudio/install/install-visual-studio)* 的说明操作。*

**如果您使用的是本书的数字版，我们建议您亲自输入代码或从本书的 GitHub 仓库（下一节中提供链接）获取代码。这样做将帮助您避免与代码复制和粘贴相关的任何潜在错误。**

*阅读本书后，您可以通过深入了解 Microsoft* *Learn:* [`learn.microsoft.com/windows/apps/`](https://learn.microsoft.com/windows/apps/)* 上的文档和示例来继续您的 Windows 开发之旅。*

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件 [`github.com/PacktPublishing/Learn-WinUI-3-Second-Edition`](https://github.com/PacktPublishing/Learn-WinUI-3-Second-Edition)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有来自我们丰富的图书和视频目录的其他代码包，可在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 获取。查看它们！

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`: 表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。以下是一个示例：“在 `INavigationService` 中，您可以更新 `namespace` 为 `UnoMediaCollection.Interfaces` 并删除 `using` `System;` 语句。”

代码块应如下设置：

```cs
using UnoMediaCollection.Enums;
using UnoMediaCollection.Interfaces;
using UnoMediaCollection.Model;
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```cs
using UnoMediaCollection.Enums;
namespace UnoMediaCollection.Model
```

任何命令行输入或输出都应如下编写：

```cs
$ mkdir css
$ cd css
```

**粗体**: 表示新术语、重要单词或屏幕上显示的单词。例如，菜单或对话框中的单词会以粗体显示。以下是一个示例：“右键单击**枚举**文件夹，然后选择**添加** | **现有项**。”

小贴士或重要注意事项

看起来是这样的。

# 联系我们

欢迎读者反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请通过 customercare@packtpub.com 发送电子邮件，并在邮件主题中提及书名。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在此书中发现错误，我们将不胜感激，如果您能向我们报告，我们将不胜感激。请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata) 并填写表格。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过 copyright@packt.com 与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了《Power Apps 自动化测试》，我们非常乐意听到您的想法！请[点击此处直接转到此书的亚马逊评论页面](https://packt.link/r/1803236558)并分享您的反馈。

您的评论对我们和科技社区非常重要，并将帮助我们确保我们提供高质量的内容。

# 下载此书的免费 PDF 副本

感谢您购买此书！

您喜欢在路上阅读，但无法携带您的印刷书籍到处走吗？

您的电子书购买是否与您选择的设备不兼容？

不要担心，现在，每购买一本 Packt 书籍，您都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何设备上阅读。直接从您最喜欢的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

优惠远不止于此，您还可以获得独家折扣、时事通讯和每日免费内容的每日电子邮件。

按照以下简单步骤获取优惠：

1.  扫描下面的二维码或访问以下链接

![](img/B20908_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781805120063`](https://packt.link/free-ebook/9781805120063)

1.  提交您的购买证明

1.  就这些了！我们将直接将您的免费 PDF 和其他优惠发送到您的电子邮件。

# 第一部分：WinUI 和 Windows 应用程序简介

WinUI 3 是微软为 Windows 开发者提供的最新 UI 框架。本节将从探索 XAML 和 Windows UI 框架的近期历史开始，向您介绍 WinUI。在本节的章节中，您将通过从头开始构建一个简单的项目并添加控件和功能，遵循设计模式和最佳实践来学习 WinUI 概念。这些模式和最佳实践包括**模型-视图-视图模型**（**MVVM**）设计模式、构建松散耦合、可测试的 C#类，以及使用**依赖注入**（**DI**）将服务依赖项注入到应用程序组件中。

本部分包含以下章节：

+   *第一章*, *WinUI 简介*

+   *第二章*, *配置开发环境和创建项目*

+   *第三章*, *MVVM 的维护性和可测试性*

+   *第四章*, *高级 MVVM 概念*

+   *第五章*, *探索 WinUI 控件*

+   *第六章*, *利用数据和服务*
