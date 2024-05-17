# 前言

C#是一种非常富有表现力和强大的语言，让您可以专注于应用程序而不是低级样板。在过去的十年中，C#编译器已经发展，包括了来自动态和函数式语言的许多功能，同时保持静态类型。最近，它已经应对了并发硬件的激增，引入了新的异步编程功能。

本书将帮助您快速了解最新版本的 C#。在设置开发环境后，您将了解语言的所有最新功能，包括任务并行框架、动态语言运行时、TPL 数据流，以及使用 async 和 await 进行异步编程。

# 本书涵盖内容

第一章 *开始使用 C#*，简要介绍了 C#的诞生，并设置开发环境以编译 C# 5。我们将涵盖编译器和框架的安装，以及所有主要编辑器，如 Visual Studio 和 MonoDevelop。

第二章 *C#的演变*，展示了 C#语言的成长和成熟。随着每个版本的发布，都引入了新功能，使 C#编程变得更加简单和富有表现力。

第三章 *异步行动*，讨论了异步编程，重点放在 5.0 版本上。从任务并行库（TPL）开始，到在这个版本中新引入的 async 和 await 关键字，您现在可以轻松编写响应迅速且可扩展的应用程序。

第四章 *创建 Windows Store 应用*，介绍了 Windows 8 引入了一种新的应用类型，运行在 WinRT 框架上，可以创建在 x86 和 ARM 架构上运行的应用程序。在本章中，我们将探讨创建一个简单的应用程序，该应用程序连接到互联网，从 Flickr 下载并显示图像。

第五章 *移动 Web 应用*，向您展示了如何使用 ASP.NET MVC 和 HTML 5 为用户创建非常复杂和引人入胜的体验。世界正在走向移动化，支持移动客户端的 Web 变得越来越重要。您将学习如何构建一个使用 HTML 5 地理位置 API 实时连接用户的移动优化 Web 应用程序。

第六章 *跨平台开发*，向您展示了作为 C#开发人员，您可以使用 Mono 框架针对非 Microsoft 平台进行开发。在本章中，您将学习如何使用 MonoMac 和 MonoDevelop 为 Mac OS 创建一个实用程序应用。使用 C#的能力可以转化为一个引人入胜的机会，如果您能够在不同平台之间共享大部分代码。

# 本书所需内容

为了测试和编译本书中的所有示例，您需要安装.NET 4.5 框架，该框架支持从 Windows Vista 及更高版本的所有 Windows 版本，您可以在以下网址找到：

[`www.microsoft.com/en-us/download/details.aspx?id=30653`](http://www.microsoft.com/en-us/download/details.aspx?id=30653)

为了编译和测试 Windows Store 和 ASP.NET MVC 项目（第四章 *创建 Windows Store 应用*和第五章 *移动 Web 应用*），您需要安装 Visual Studio 2012 的一个版本（[`www.microsoft.com/visualstudio`](http://www.microsoft.com/visualstudio)）。此外，Windows Store 项目要求您在 Windows 8 上运行 Visual Studio 2012。

本书的最终项目在第六章中，*跨平台开发*是使用 MonoMac ([`www.mono-project.com/MonoMac`](http://www.mono-project.com/MonoMac))和 MonoDevelop ([`monodevelop.com`](http://monodevelop.com))创建一个 Mac OS 应用程序。您必须在 Mac 上开发这个项目。

# 本书适用对象

本书适用于想了解最新版本 C#的开发人员。假定您具有基本的编程知识。有先前版本 C#或.NET Framework 的经验会有所帮助，但不是必需的。

# 约定

在本书中，您会发现一些文本样式，用于区分不同类型的信息。以下是一些这些样式的示例，以及它们的含义解释。

文本中的代码单词显示如下："默认情况下，`csc`将输出一个可执行文件。"

代码块设置如下：

```cs
using System;

namespace program
{
    class MainClass
    {
        static void Main (string[] args)
        {
            Console.WriteLine("Hello, World");
        }
    }
}
```

任何命令行输入或输出都以以下方式书写：

```cs
PS ~> $env:Path += ";C:\Windows\Microsoft.NET\Framework\v4.0.30319"

```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，比如菜单或对话框中的单词，会以这样的方式出现在文本中："通过单击**文件** | **新建项目…**来创建一个新项目。"。

### 注意

警告或重要提示会以这样的方式出现在一个框中。

### 提示

提示和技巧会以这样的方式出现。
