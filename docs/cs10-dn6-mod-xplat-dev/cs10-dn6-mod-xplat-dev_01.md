# 第一章：01

# 你好，C#! 欢迎，.NET！

在这第一章中，目标是设置你的开发环境，理解现代.NET、.NET Core、.NET Framework、Mono、Xamarin 和.NET Standard 之间的相似性和差异，使用各种代码编辑器创建尽可能简单的 C# 10 和.NET 6 应用程序，然后发现寻求帮助的好地方。

这本书的 GitHub 存储库包含了所有代码任务和笔记本的完整应用程序项目的解决方案：

[`github.com/markjprice/cs10dotnet6`](https://github.com/markjprice/cs10dotnet6)

只需按下.（点）键或在上面的链接中将`.com`更改为`.dev`，即可将 GitHub 存储库更改为使用 Visual Studio Code for the Web 进行实时编辑，如*图 1.1*所示：

![图形用户界面，文本，应用程序描述自动生成](img/Image00007.jpg)

图 1.1：Visual Studio Code for the Web 实时编辑该书的 GitHub 存储库

这对于在你选择的代码编辑器旁边运行是很好的，你可以将你的代码与解决方案代码进行比较，如果需要，可以轻松地复制和粘贴部分代码。

在整本书中，我使用术语“现代.NET”来指代.NET 6 及其前身，如来自.NET Core 的.NET 5。我使用术语“传统.NET”来指代.NET Framework、Mono、Xamarin 和.NET Standard。现代.NET 是这些传统平台和标准的统一。

在这第一章之后，这本书可以分为三个部分：第一部分是 C#语言的语法和词汇；第二部分是.NET 中可用的类型，用于构建应用程序功能；第三部分是使用 C#和.NET 构建的常见跨平台应用程序的示例。

大多数人通过模仿和重复学习复杂的主题，而不是阅读详细的理论解释；因此，我不会在整本书中详细解释每一步。想法是让你编写一些代码并看到它运行。

你不需要立即了解所有细节。随着时间的推移，随着你构建自己的应用程序并超越任何书籍所能教授的内容，这些细节会逐渐明朗起来。

用塞缪尔·约翰逊（Samuel Johnson）的话来说，他是 1755 年英语词典的作者，我犯了“一些荒唐的错误和可笑的荒谬之处，这是任何如此多元化的作品所不可避免的。” 我对此负有全部责任，并希望你能欣赏我尝试撰写这本关于快速发展的技术如 C#和.NET 以及你可以用它们构建的应用程序的书籍的挑战。

第一章涵盖以下主题：

+   设置你的开发环境

+   理解.NET

+   使用 Visual Studio 2022 构建控制台应用程序

+   使用 Visual Studio Code 构建控制台应用程序

+   使用.NET 交互式笔记本来探索代码

+   审查项目的文件夹和文件

+   充分利用 GitHub 存储库

+   寻求帮助

# 设置你的开发环境

在开始编程之前，你需要一个 C#的代码编辑器。微软有一系列代码编辑器和集成开发环境（IDE），包括：

+   Windows 的 Visual Studio 2022

+   Mac 的 Visual Studio 2022

+   Windows、Mac 或 Linux 的 Visual Studio Code

+   GitHub Codespaces

第三方已经创建了他们自己的 C#代码编辑器，例如 JetBrains Rider。

## 选择适合学习的工具和应用类型

学习 C#和.NET 的最佳工具和应用类型是什么？

在学习时，最好的工具是能帮助你编写代码和配置，但不会隐藏实际发生的事情。IDE 提供了友好的图形用户界面，但在底层为你做了什么呢？在学习时，更接近行动并提供帮助编写代码的基本代码编辑器更好。

话虽如此，您可以提出这样的论点，即最好的工具是您已经熟悉的工具，或者您或您的团队将作为日常开发工具使用的工具。因此，我希望您可以自由选择任何 C#代码编辑器或 IDE 来完成本书中的编码任务，包括 Visual Studio Code、Visual Studio for Windows、Visual Studio for Mac，甚至 JetBrains Rider。

在本书的第三版中，我为 Windows 的 Visual Studio 和所有编码任务的 Visual Studio Code 提供了详细的逐步说明。不幸的是，情况很快变得混乱和令人困惑。在第六版中，我仅在*第一章*中提供了如何在 Windows 的 Visual Studio 2022 和 Visual Studio Code 中创建多个项目的详细逐步说明。之后，我提供了项目名称和适用于所有工具的一般说明，因此您可以使用您喜欢的任何工具。

学习 C#语言构造和许多.NET 库的最佳应用类型是不会分散注意力的应用程序代码。例如，没有必要创建整个 Windows 桌面应用程序或网站，只是为了学习如何编写`switch`语句。

因此，我认为学习*第 1*至*12*章中的 C#和.NET 主题的最佳方法是构建控制台应用程序。然后，在*第 13*至*19*章以及以后，您将构建网站、服务以及图形桌面和移动应用程序。

### .NET Interactive Notebooks 扩展的优缺点

Visual Studio Code 的另一个好处是.NET Interactive Notebooks 扩展。该扩展提供了一个简单且安全的地方来编写简单的代码片段。它使您能够创建一个混合使用 C#和其他相关语言（如 PowerShell、F#和 SQL（用于数据库））的 Markdown（格式丰富的文本）和代码的“单元格”的单个笔记本文件。

然而，.NET Interactive Notebooks 确实有一些限制：

+   它们无法从用户那里读取输入，例如，您不能使用`ReadLine`或`ReadKey`。

+   它们不能接受传递给它们的参数。

+   它们不允许您定义自己的命名空间。

+   它们没有任何调试工具（但这些将来会推出）。

### 使用 Visual Studio Code 进行跨平台开发

选择最现代和轻量级的代码编辑器，也是微软唯一跨平台的代码编辑器，就是 Microsoft Visual Studio Code。它可以在包括 Windows、macOS 和许多 Linux 变体在内的所有常见操作系统上运行，包括 Red Hat Enterprise Linux（RHEL）和 Ubuntu。

Visual Studio Code 是现代跨平台开发的不错选择，因为它有大量且不断增长的扩展，支持除了 C#之外的许多语言。

作为跨平台和轻量级的工具，它可以安装在您的应用程序将部署到的所有平台上，以进行快速的错误修复等。选择 Visual Studio Code 意味着开发人员可以使用跨平台代码编辑器来开发跨平台应用程序。

Visual Studio Code 对 Web 开发有很好的支持，尽管目前对移动和桌面开发的支持较弱。

Visual Studio Code 支持 ARM 处理器，因此您可以在 Apple Silicon 计算机和 Raspberry Pi 上进行开发。

Visual Studio Code 是迄今为止最受欢迎的集成开发环境，超过 70%的专业开发人员在 Stack Overflow 2021 调查中选择了它。

### 在云中使用 GitHub Codespaces 进行开发

GitHub Codespaces 是基于 Visual Studio Code 的完全配置的开发环境，可以在云中托管的环境中启动，并通过任何 Web 浏览器访问。它支持 Git 存储库、扩展和内置命令行界面，因此您可以从任何设备编辑、运行和测试。

### 使用 Visual Studio for Mac 进行一般开发

Microsoft Visual Studio 2022 for Mac 可以创建大多数类型的应用程序，包括控制台应用程序、网站、Web 服务、桌面和移动应用程序。

要编译运行在 iPhone 和 iPad 等设备上的苹果操作系统的应用程序，您必须拥有只能在 macOS 上运行的 Xcode。

### 在 Windows 上使用 Visual Studio 进行一般开发

Microsoft Visual Studio 2022 for Windows 可以创建大多数类型的应用程序，包括控制台应用程序、网站、Web 服务、桌面和移动应用程序。虽然您可以使用 Windows 版的 Visual Studio 2022 及其 Xamarin 扩展来编写跨平台移动应用程序，但您仍然需要 macOS 和 Xcode 来编译它。

它只能在 Windows 7 SP1 或更高版本上运行。您必须在 Windows 10 或 Windows 11 上运行它，以创建从 Microsoft Store 安装并在沙箱中运行以保护您的计算机的**通用 Windows 平台**（**UWP**）应用程序。

### 我使用的

为了编写和测试本书的代码，我使用了以下硬件：

+   HP Spectre（Intel）笔记本电脑

+   Apple Silicon Mac mini（M1）台式机

+   Raspberry Pi 400（ARM v8）台式机

我使用了以下软件：

+   Visual Studio Code 上：

+   Apple Silicon Mac mini（M1）台式机上的 macOS

+   Windows 10 在 HP Spectre（Intel）笔记本电脑上

+   Raspberry Pi 400 上的 Ubuntu 64

+   Windows 上的 Visual Studio 2022：

+   HP Spectre（Intel）笔记本电脑上的 Windows 10

+   Mac 上的 Visual Studio 2022：

+   Apple Silicon Mac mini（M1）台式机上的 macOS

我希望您也能访问各种硬件和软件，因为了解平台的差异可以加深您对开发挑战的理解，尽管以上任何一种组合都足以学习 C#和.NET 的基础知识以及如何构建实际的应用程序和网站。

**更多信息**：您可以通过阅读我在以下链接写的额外文章，了解如何使用 Raspberry Pi 400 和 Ubuntu 桌面 64 位编写 C#和.NET 代码：[`github.com/markjprice/cs9dotnet5-extras/blob/main/raspberry-pi-ubuntu64/README.md`](https://github.com/markjprice/cs9dotnet5-extras/blob/main/raspberry-pi-ubuntu64/README.md)。

## 跨平台部署

您选择的代码编辑器和操作系统并不限制代码的部署位置。

.NET 6 支持以下部署平台：

+   **Windows**：Windows 7 SP1 或更高版本。Windows 10 版本 1607 或更高版本，包括 Windows 11。Windows Server 2012 R2 SP1 或更高版本。Nano Server 版本 1809 或更高版本。

+   **Mac**：macOS Mojave（版本 10.14）或更高版本。

+   **Linux**：Alpine Linux 3.13 或更高版本。CentOS 7 或更高版本。Debian 10 或更高版本。Fedora 32 或更高版本。openSUSE 15 或更高版本。Red Hat Enterprise Linux（RHEL）7 或更高版本。SUSE Enterprise Linux 12 SP2 或更高版本。Ubuntu 16.04、18.04、20.04 或更高版本。

+   **Android**：API 21 或更高版本。

+   **iOS**：10 或更高版本。

.NET 5 及更高版本中的 Windows ARM64 支持意味着您可以在 Windows ARM 设备上开发和部署，比如微软 Surface Pro X。但是使用 Parallels 和 Windows 10 ARM 虚拟机在 Apple M1 Mac 上进行开发，速度显然是原来的两倍！

## 下载并安装 Windows 版 Visual Studio 2022

许多专业的微软开发人员在日常开发工作中使用 Windows 的 Visual Studio 2022。即使您选择使用 Visual Studio Code 来完成本书中的编码任务，您可能也希望熟悉一下 Windows 的 Visual Studio 2022。

如果您没有 Windows 电脑，那么您可以跳过本节，继续下一节，在那里您将在 macOS 或 Linux 上下载并安装 Visual Studio Code。

自 2014 年 10 月以来，微软已经向学生、开源贡献者和个人免费提供了 Windows 版专业质量的 Visual Studio，称为社区版。任何版本都适用于本书。如果您还没有安装，请立即安装：

1.  从以下链接下载 Windows 版 Microsoft Visual Studio 2022 版本 17.0 或更高版本：[`visualstudio.microsoft.com/downloads/`](https://visualstudio.microsoft.com/downloads/)。

1.  启动安装程序。

1.  在**工作负载**选项卡上，选择以下内容：

+   **ASP.NET 和 Web 开发**

+   **Azure 开发**

+   **.NET 桌面开发**

+   **使用 C++进行桌面开发**

+   **通用 Windows 平台开发**

+   **使用.NET 进行移动开发**

1.  在**单独组件**选项卡上，在**代码工具**部分，选择以下内容：

+   **类设计师**

+   **Windows 的 Git**

+   **PreEmptive Protection - Dotfuscator**

1.  点击**安装**，等待安装程序获取所选软件并安装。

1.  安装完成后，点击**启动**。

1.  第一次运行 Visual Studio 时，您将被提示登录。如果您有 Microsoft 帐户，可以使用该帐户。如果没有，则可以在以下链接注册一个新帐户：[`signup.live.com/`](https://signup.live.com/)。

1.  第一次运行 Visual Studio 时，您将被提示配置您的环境。对于**开发设置**，请选择**Visual C#**。对于颜色主题，我选择了**蓝色**，但您可以选择任何您喜欢的。

1.  如果您想要定制您的键盘快捷键，导航到**工具** | **选项...**，然后选择**键盘**部分。

### Windows 版 Microsoft Visual Studio 的键盘快捷键

在本书中，我将避免显示键盘快捷键，因为它们经常被定制。如果它们在代码编辑器中保持一致并且被广泛使用，我会尽量展示它们。如果您想识别和定制您的键盘快捷键，那么您可以在以下链接中找到相关信息：[`docs.microsoft.com/en-us/visualstudio/ide/identifying-and-customizing-keyboard-shortcuts-in-visual-studio`](https://docs.microsoft.com/en-us/visualstudio/ide/identifying-and-customizing-keyboard-shortcuts-in-visual-studio)。

## 下载并安装 Visual Studio Code

Visual Studio Code 在过去几年里迅速改进，并且以其受欢迎程度令微软惊喜。如果您勇敢并且喜欢尝试最新技术，那么有 Insiders 版本，这是下一个版本的每日构建。

即使您打算仅在 Windows 上使用 Visual Studio 2022 进行开发，我建议您下载并安装 Visual Studio Code，并尝试使用它进行本章的编码任务，然后决定是否要继续只使用 Visual Studio 2022 进行本书的其余部分。

现在让我们下载并安装 Visual Studio Code、.NET SDK 以及 C#和.NET Interactive 笔记本扩展：

1.  从以下链接下载并安装 Visual Studio Code 的稳定版本或 Insiders 版本：[`code.visualstudio.com/`](https://code.visualstudio.com/)。

**更多信息**：如果您需要更多帮助安装 Visual Studio Code，可以阅读以下链接的官方设置指南：[`code.visualstudio.com/docs/setup/setup-overview`](https://code.visualstudio.com/docs/setup/setup-overview)。

1.  从以下链接下载并安装版本 3.1、5.0 和 6.0 的.NET SDK：[`www.microsoft.com/net/download`](https://www.microsoft.com/net/download)。

为了完全学会如何控制.NET SDK，我们需要安装多个版本。.NET Core 3.1、.NET 5.0 和.NET 6.0 是目前支持的三个版本。您可以安全地并排安装多个版本。在本书中，您将学习如何选择您想要的版本。

1.  要安装 C#扩展，您必须首先启动 Visual Studio Code 应用程序。

1.  在 Visual Studio Code 中，点击**扩展**图标或导航到**视图** | **扩展**。

1.  C#是最受欢迎的扩展之一，因此您应该在列表顶部看到它，或者您可以在搜索框中输入`C#`。

1.  点击**安装**，等待支持包下载并安装。

1.  在搜索框中输入`.NET Interactive`以找到**.NET Interactive 笔记本**扩展。

1.  点击**安装**，等待安装完成。

### 安装其他扩展

在本书的后面章节中，您将使用更多的扩展。如果您现在想要安装它们，我们将使用的所有扩展都显示在以下表格中：

| 扩展名和标识符 | 描述 |
| --- | --- |
| C# for Visual Studio Code (powered by OmniSharp)`ms-dotnettools.csharp` | 包括语法高亮、智能感知、跳转到定义、查找所有引用、.NET 的调试支持以及对 Windows、macOS 和 Linux 上的 `csproj` 项目的支持。 |
| .NET Interactive Notebooks`ms-dotnettools.dotnet-interactive-vscode` | 此扩展添加了对在 Visual Studio Code 笔记本中使用 .NET Interactive 的支持。它依赖于 Jupyter 扩展 (`ms-toolsai.jupyter` )。 |
| MSBuild 项目工具`tinytoy.msbuild-project-tools` | 为 MSBuild 项目文件提供智能感知，包括 `<PackageReference>` 元素的自动完成。 |
| REST Client`humao.rest-client` | 直接在 Visual Studio Code 中发送 HTTP 请求并查看响应。 |
| ILSpy .NET 反编译器`icsharpcode.ilspy-vscode` | 反编译 MSIL 程序集 - 支持现代 .NET、.NET Framework、.NET Core 和 .NET Standard。 |
| Azure Functions for Visual Studio Code`ms-azuretools.vscode-azurefunctions` | 直接从 VS Code 创建、调试、管理和部署无服务器应用。它依赖于 Azure Account (`ms-vscode.azure-account` ) 和 Azure Resources (`ms-azuretools.vscode-azureresourcegroups` ) 扩展。 |
| GitHub 仓库`github.remotehub` | 直接在 Visual Studio Code 中浏览、搜索、编辑和提交到任何远程 GitHub 仓库。 |
| SQL Server (mssql) for Visual Studio Code`ms-mssql.mssql` | 在任何地方开发 Microsoft SQL Server、Azure SQL 数据库和 SQL 数据仓库，具有丰富的功能集。 |
| Visual Studio Code 的 Protobuf 3 支持`zxh404.vscode-proto3` | 语法高亮、语法验证、代码片段、代码完成、代码格式化、大括号匹配以及行和块注释。 |

### 了解 Microsoft Visual Studio Code 的版本

微软几乎每个月发布 Visual Studio Code 的新功能版本，更频繁地发布 bug 修复版本。例如：

+   版本 1.59，2021 年 8 月功能版本

+   版本 1.59.1，2021 年 8 月 bug 修复版本

本书使用的版本是 1.59，但 Microsoft Visual Studio Code 的版本不如您安装的 C# for Visual Studio Code 扩展的版本重要。

虽然不需要 C# 扩展，但它提供了您输入时的智能感知、代码导航和调试功能，因此安装并保持更新以支持最新的 C# 语言特性非常方便。

### Microsoft Visual Studio Code 的键盘快捷键

在本书中，我将避免显示用于任务的键盘快捷键，比如创建新文件，因为它们在不同的操作系统上通常是不同的。我将展示键盘快捷键的情况是当您需要重复按键时，例如在调试时。这些情况更有可能在不同操作系统上保持一致。

如果您想要自定义 Visual Studio Code 的键盘快捷键，可以参考以下链接：[`code.visualstudio.com/docs/getstarted/keybindings`](https://code.visualstudio.com/docs/getstarted/keybindings) 。

我建议您从以下列表中下载适用于您操作系统的键盘快捷键的 PDF：

+   **Windows** : [`code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf`](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf)

+   **macOS** : [`code.visualstudio.com/shortcuts/keyboard-shortcuts-macos.pdf`](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-macos.pdf)

+   **Linux** : [`code.visualstudio.com/shortcuts/keyboard-shortcuts-linux.pdf`](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-linux.pdf)

# 了解 .NET

.NET 6、.NET Core、.NET Framework 和 Xamarin 是相关且重叠的开发平台，用于构建应用程序和服务。在本节中，我将向您介绍这些.NET 概念。

## 理解.NET Framework

.NET Framework 是一个开发平台，包括**公共语言运行时**（**CLR**），用于管理代码的执行，以及**基础类库**（**BCL**），提供丰富的类库来构建应用程序。

微软最初设计.NET Framework 具有跨平台的可能性，但微软将他们的实现努力放在使其与 Windows 最好地配合使用。

自.NET Framework 4.5.2 以来，它已成为 Windows 操作系统的官方组件。组件具有与其父产品相同的支持，因此 4.5.2 及更高版本遵循其安装的 Windows 操作系统的生命周期策略。.NET Framework 已安装在超过 10 亿台计算机上，因此必须尽可能少地进行更改。即使是错误修复也可能会导致问题，因此更新频率较低。

对于.NET Framework 4.0 或更高版本，计算机上为.NET Framework 编写的所有应用程序共享**全局程序集缓存**（**GAC**）中存储的 CLR 和库的相同版本，这可能会导致问题，如果其中一些应用程序需要特定版本以实现兼容性。

**良好实践**：从实际角度来看，.NET Framework 仅适用于 Windows，是一个传统的平台。不要使用它创建新的应用程序。

## 理解 Mono、Xamarin 和 Unity 项目

第三方开发了一个名为**Mono**项目的.NET Framework 实现。Mono 是跨平台的，但远远落后于官方的.NET Framework 实现。

Mono 已经成为**Xamarin**移动平台和跨平台游戏开发平台**Unity**的基础。

微软在 2016 年收购了 Xamarin，现在免费提供了曾经昂贵的 Xamarin 扩展，并将其与 Visual Studio 捆绑在一起。微软将只能创建移动应用程序的 Xamarin Studio 开发工具更名为 Visual Studio for Mac，并赋予其创建其他类型项目（如控制台应用程序和 Web 服务）的能力。通过 Visual Studio 2022 for Mac，微软已经用 Visual Studio 2022 for Windows 的部分替换了 Xamarin Studio 编辑器的部分，以提供更接近的体验和性能。Visual Studio 2022 for Mac 也被重写为真正的本地 macOS UI 应用程序，以提高可靠性并与 macOS 内置的辅助技术配合使用。

## 理解.NET Core

今天，我们生活在一个真正跨平台的世界，现代移动和云开发已经使作为操作系统的 Windows 变得不那么重要。因此，微软一直在努力将.NET 与其与 Windows 的紧密联系解耦。在将.NET Framework 重写为真正的跨平台时，他们抓住机会重构和删除不再被认为是核心的重要部分。

这个新产品被命名为.NET Core，并包括一个名为 CoreCLR 的跨平台 CLR 实现和一个名为 CoreFX 的精简 BCL。

微软.NET 合作伙伴总监.NET 项目经理 Scott Hunter 表示：“我们.NET Core 客户中有 40%是全新的开发人员，这正是我们希望.NET Core 达到的效果。我们希望吸引新的开发人员。”

.NET Core 发展迅速，因为它可以与应用程序并行部署，所以它可能会经常发生变化，了解这些变化不会影响同一台机器上的其他.NET Core 应用程序。微软对.NET Core 和现代.NET 所做的大多数改进都无法轻松地添加到.NET Framework 中。

## 理解通往一个.NET 的旅程

在 2020 年 5 月的微软 Build 开发者大会上，.NET 团队宣布他们关于.NET 统一的计划已经延迟。他们表示.NET 5 将于 2020 年 11 月 10 日发布，并将统一除移动外的所有各种.NET 平台。直到 2021 年 11 月的.NET 6 版本，移动端也将被统一的.NET 平台支持。

.NET Core 已更名为.NET，主要版本号跳过了数字四，以避免与.NET Framework 4.x 混淆。微软计划每年 11 月发布一次主要版本，有点类似于苹果每年 9 月发布 iOS 的主要版本号。

以下表格显示了现代.NET 的关键版本发布时间，未来发布计划以及本书各版本所使用的版本：

| 版本 | 发布时间 | 版本 | 发布时间 |
| --- | --- | --- | --- |
| .NET Core RC1 | 2015 年 11 月 | 第一版 | 2016 年 3 月 |
| .NET Core 1.0 | 2016 年 6 月 |  |  |
| .NET Core 1.1 | 2016 年 11 月 |  |  |
| .NET Core 1.0.4 and .NET Core 1.1.1 | 2017 年 3 月 | 第二版 | 2017 年 3 月 |
| .NET Core 2.0 | 2017 年 8 月 |  |  |
| .NET Core for UWP in Windows 10 Fall Creators Update | 2017 年 10 月 | 第三版 | 2017 年 11 月 |
| .NET Core 2.1 (LTS) | 2018 年 5 月 |  |  |
| .NET Core 2.2 (当前) | 2018 年 12 月 |  |  |
| .NET Core 3.0 (当前) | 2019 年 9 月 | 第四版 | 2019 年 10 月 |
| .NET Core 3.1 (LTS) | 2019 年 12 月 |  |  |
| Blazor WebAssembly 3.2 (当前) | 2020 年 5 月 |  |  |
| .NET 5.0 (当前) | 2020 年 11 月 | 第五版 | 2020 年 11 月 |
| .NET 6.0 (LTS) | 2021 年 11 月 | 第六版 | 2021 年 11 月 |
| .NET 7.0 (当前) | 2022 年 11 月 | 第七版 | 2022 年 11 月 |
| .NET 8.0 (LTS) | 2023 年 11 月 | 第八版 | 2023 年 11 月 |

.NET Core 3.1 包括用于构建 Web 组件的 Blazor Server。微软原本计划在该版本中包括 Blazor WebAssembly，但推迟了。Blazor WebAssembly 后来作为.NET Core 3.1 的可选附加组件发布。我在上表中包括它，因为它的版本号为 3.2，以排除它不在.NET Core 3.1 的 LTS 范围内。

## 理解.NET 支持

.NET 版本要么是**长期支持**（**LTS**），要么是**当前**，如下列表所述：

+   **LTS** 版本稳定，寿命内需要较少的更新。这些版本适用于您不打算经常更新的应用程序。LTS 版本将在通用可用性后支持 3 年，或者在下一个 LTS 版本发布后的 1 年内支持，以较长者为准。

+   **当前** 版本包括可能根据反馈而更改的功能。这些版本适用于您正在积极开发的应用程序，因为它们提供了最新改进的访问权限。在 6 个月的维护期后，或者在通用可用性后的 18 个月内，之前的次要版本将不再得到支持。

它们在寿命内都会接收到关于安全性和可靠性的关键修复。您必须及时更新最新的补丁以获得支持。例如，如果系统运行的是 1.0 版本，而 1.0.1 已发布，就需要安装 1.0.1 版本以获得支持。

为了更好地理解您对当前和 LTS 版本的选择，以图形方式呈现是有帮助的，LTS 版本为 3 年长的黑色条，当前版本为可变长度的灰色条，在新的主要或次要版本发布后的 6 个月内保持支持，如*图 1.2*所示：

![Text Description automatically generated with low confidence](img/Image00008.jpg)

图 1.2：各版本支持

例如，如果您使用.NET Core 3.0 创建了一个项目，那么当微软在 2019 年 12 月发布.NET Core 3.1 时，您必须在 2020 年 3 月之前将项目升级到.NET Core 3.1。（在.NET 5 之前，当前版本的维护期仅为三个月。）

如果您需要微软的长期支持，那么今天选择.NET 6.0 并坚持到.NET 8.0，即使微软发布.NET 7.0。这是因为.NET 7.0 将是一个当前版本，因此在.NET 6.0 之前失去支持。只需记住，即使使用 LTS 版本，您也必须升级到 6.0.1 等 bug 修复版本。

除以下列表中显示的版本外，所有版本的.NET Core 和现代.NET 都已经结束生命周期：

+   .NET 5.0 将于 2022 年 5 月结束生命周期。

+   .NET Core 3.1 将于 2022 年 12 月 3 日结束生命周期。

+   .NET 6.0 将于 2024 年 11 月结束生命周期。

### 理解.NET Runtime 和.NET SDK 版本

.NET Runtime 版本遵循语义版本控制，即，主要增量表示破坏性更改，次要增量表示新功能，补丁增量表示 bug 修复。

.NET SDK 版本不遵循语义版本控制。主要和次要版本号与其匹配的运行时版本相关联。补丁号遵循一种约定，表示 SDK 的主要和次要版本。

您可以在以下表格中看到一个例子：

| 变更 | 运行时 | SDK |
| --- | --- | --- |
| 初始发布 | 6.0.0 | 6.0.100 |
| SDK 错误修复 | 6.0.0 | 6.0.101 |
| 运行时和 SDK 错误修复 | 6.0.1 | 6.0.102 |
| SDK 新功能 | 6.0.1 | 6.0.200 |

### 移除旧版本的.NET

.NET Runtime 更新与 6.x 等主要版本兼容，更新的.NET SDK 版本保持了构建针对先前版本的运行时的应用程序的能力，这使得可以安全地移除旧版本。

您可以使用以下命令查看当前安装的 SDK 和运行时：

+   `dotnet --list-sdks`

+   `dotnet --list-runtimes`

在 Windows 上，使用“应用和功能”部分来移除.NET SDK。在 macOS 或 Windows 上，使用`dotnet-core-uninstall`工具。这个工具不是默认安装的。

例如，在撰写第四版时，我每个月都使用以下命令：

```cs

dotnet-core-uninstall remove --all-previews-but-latest --sdk

```

## 现代.NET 有何不同？

与传统的.NET Framework 相比，现代.NET 是模块化的，后者是单片的。它是开源的，微软在开放中做出改进和变化的决定。微软特别努力改进现代.NET 的性能。

由于删除了传统和非跨平台技术，它比.NET Framework 的上一个版本要小。例如，工作负载，如 Windows Forms 和**Windows Presentation Foundation**（**WPF**）可用于构建**图形用户** **界面**（**GUI**）应用程序，但它们与 Windows 生态系统紧密绑定，因此不包括在 macOS 和 Linux 的.NET 中。

### Windows 开发

现代.NET 的一个特性是支持使用包含在 Windows 版本的.NET Core 3.1 或更高版本中的 Windows 桌面包运行旧的 Windows Forms 和 WPF 应用程序，这就是为什么它比 macOS 和 Linux 的 SDK 更大。如果需要，您可以对旧的 Windows 应用程序进行一些小的更改，然后重新构建为.NET 6，以利用新功能和性能改进。

### Web 开发

ASP.NET Web Forms 和 Windows Communication Foundation（WCF）是较少开发人员选择用于新开发项目的旧的 Web 应用程序和服务技术，因此它们也已从现代.NET 中移除。相反，开发人员更喜欢使用 ASP.NET MVC，ASP.NET Web API，SignalR 和 gRPC。这些技术已经被重构并合并到一个在现代.NET 上运行的平台中，名为 ASP.NET Core。您将在*第十四章*，*使用 ASP.NET Core Razor Pages 构建网站*，*第十五章*，*使用模型-视图-控制器模式构建网站*，*第十六章*，*构建和使用 Web 服务*和*第十八章*，*构建和使用专门服务*中了解这些技术（第十八章可在[`github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf`](https://github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf)中找到）。

更多信息：一些.NET Framework 开发人员对 ASP.NET Web Forms，WCF 和 Windows Workflow（WF）缺失在现代.NET 中感到不满，并希望微软改变他们的想法。有一些开源项目可以使 WCF 和 WF 迁移到现代.NET。您可以在以下链接阅读更多信息：[`devblogs.microsoft.com/dotnet/supporting-the-community-with-wf-and-wcf-oss-projects/`](https://devblogs.microsoft.com/dotnet/supporting-the-community-with-wf-and-wcf-oss-projects/)。在以下链接中有一个 Blazor Web Forms 组件的开源项目：[`github.com/FritzAndFriends/BlazorWebFormsComponents`](https://github.com/FritzAndFriends/BlazorWebFormsComponents)。

### 数据库开发

Entity Framework（EF）6 是一种对象关系映射技术，旨在处理存储在关系数据库中的数据，如 Oracle 和 Microsoft SQL Server。多年来，它已经积累了一些问题，因此跨平台 API 已经被精简，已经支持了像 Microsoft Azure Cosmos DB 这样的非关系数据库，并且已经更名为 Entity Framework Core。您将在*第十章*，*使用 Entity Framework Core 处理数据*中了解更多信息。

如果您有现有的使用旧 EF 的应用程序，则版本 6.3 支持.NET Core 3.0 或更高版本。

## 现代.NET 的主题

微软使用 Blazor 创建了一个网站，展示了现代.NET 的主要主题：[`themesof.net/`](https://themesof.net/)。

## 理解.NET 标准

2019 年.NET 的情况是，有三个由微软控制的.NET 分支平台，如下列表所示：

+   .NET Core：用于跨平台和新应用程序

+   .NET Framework：用于传统应用程序

+   Xamarin：用于移动应用程序

每个平台都有其优势和劣势，因为它们都是为不同的场景而设计的。这导致了一个问题，即开发人员必须学习三个平台，每个平台都有令人讨厌的怪癖和限制。

因此，微软定义了.NET 标准-一组所有.NET 平台都可以实现的 API 的规范，以指示它们具有何种级别的兼容性。例如，平台符合.NET 标准 1.4 表示基本支持。

通过.NET 标准 2.0 及更高版本，微软使得所有三个平台都收敛到了一个现代的最低标准，这使得开发人员可以更轻松地在任何.NET 版本之间共享代码。

对于.NET Core 2.0 及更高版本，这增加了大部分开发人员需要将为.NET Framework 编写的旧代码移植到跨平台.NET Core 的缺失 API。但是，一些 API 已经实现，但会抛出异常，以指示开发人员实际上不应该使用它们！这通常是由于您运行.NET 的操作系统的差异。您将在*第二章*，*使用 C#进行交谈*中学习如何处理这些异常。

重要的是要理解，.NET Standard 只是一个标准。你不能像安装 HTML5 一样安装.NET Standard。要使用 HTML5，必须安装一个实现了 HTML5 标准的网络浏览器。

要使用.NET Standard，必须安装实现.NET Standard 规范的.NET 平台。最后一个.NET Standard，版本 2.1，由.NET Core 3.0、Mono 和 Xamarin 实现。C# 8.0 的一些功能需要.NET Standard 2.1。.NET Standard 2.1 不被.NET Framework 4.8 实现，因此我们应该将.NET Framework 视为遗留系统。

随着 2021 年 11 月发布的.NET 6，.NET Standard 的需求显著减少，因为现在所有平台，包括移动平台，都有一个单一的.NET。.NET 6 有一个单一的 BCL 和两个 CLR：CoreCLR 针对服务器或桌面场景进行了优化，如网站和 Windows 桌面应用程序，而 Mono 运行时针对移动和具有有限资源的网络浏览器应用程序进行了优化。

即使现在，为.NET Framework 创建的应用程序和网站仍需要得到支持，因此重要的是要理解，您可以创建与旧版.NET 平台向后兼容的.NET Standard 2.0 类库。

## 书籍版本使用的.NET 平台和工具

对于本书的第一版，写于 2016 年 3 月，我专注于.NET Core 功能，但在.NET Core 1.0 最终发布之前，当重要或有用的功能尚未在.NET Core 中实现时，使用了.NET Framework。大多数示例使用 Visual Studio 2015，只有简短地展示了 Visual Studio Code。

第二版（几乎）完全清除了所有.NET Framework 代码示例，以便读者能够专注于真正可以在跨平台上运行的.NET Core 示例。

第三版完成了转换。它被重写，以便所有代码都是纯.NET Core。但是为所有任务提供了使用 Visual Studio Code 和 Visual Studio 2017 的逐步说明增加了复杂性。

第四版继续了这一趋势，只展示了使用 Visual Studio Code 的编码示例，除了最后两章。在*第二十章*，*构建 Windows 桌面应用程序*中，它使用了在 Windows 10 上运行的 Visual Studio，而在*第二十一章*，*构建跨平台移动应用程序*中，它使用了 Visual Studio for Mac。

在第五版中，*第二十章*，*构建 Windows 桌面应用程序*，被移动到*附录 B*，为新的*第二十章*，*使用 Blazor 构建 Web 用户界面*，腾出空间。可以使用 Visual Studio Code 创建 Blazor 项目。

在第六版中，*第十九章*，*使用.NET MAUI 构建移动和桌面应用程序*，已更新，展示了如何使用 Visual Studio 2022 和**.NET MAUI**（**多平台应用 UI**）创建移动和桌面跨平台应用程序。您可以在[`github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf`](https://github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf)找到它。

到第七版和.NET 7 的发布时，Visual Studio Code 将有一个扩展来支持.NET MAUI。到那时，读者将能够在本书中的所有示例中使用 Visual Studio Code。

## 理解中间语言

`dotnet` CLI 工具使用的 C#编译器（名为**Roslyn**）将您的 C#源代码转换为**中间语言**（**IL**）代码，并将 IL 存储在**程序集**（DLL 或 EXE 文件）中。IL 代码语句类似于汇编语言指令，由.NET 的虚拟机 CoreCLR 执行。

在运行时，CoreCLR 从程序集中加载 IL 代码，**即时**（**JIT**）编译器将其编译成本机 CPU 指令，然后由计算机上的 CPU 执行。

这种两步编译过程的好处是，微软可以为 Linux 和 macOS 创建 CLR，也可以为 Windows 创建 CLR。由于第二个编译步骤生成了针对本机操作系统和 CPU 指令集的代码，所以相同的 IL 代码可以在任何地方运行。

无论源代码使用哪种语言编写，例如 C#、Visual Basic 或 F#，所有.NET 应用程序都使用 IL 代码作为其指令存储在程序集中。微软和其他公司提供了反汇编工具，可以打开程序集并显示这些 IL 代码，例如 ILSpy .NET 反编译器扩展。

## 比较.NET 技术

我们可以总结和比较今天的.NET 技术，如下表所示：

| 技术 | 描述 | 主机操作系统 |
| --- | --- | --- |
| 现代.NET | 现代功能集，完全支持 C# 8、9 和 10，用于移植现有应用程序或创建新的桌面、移动和 Web 应用程序和服务 | Windows，macOS，Linux，Android，iOS |
| .NET Framework | 传统功能集，有限的 C# 8 支持，没有 C# 9 或 10 支持，仅用于维护现有应用程序 | 仅限 Windows |
| Xamarin | 仅限移动和桌面应用程序 | Android，iOS，macOS |

# 使用 Visual Studio 2022 构建控制台应用程序

本节的目标是展示如何使用 Visual Studio 2022 为 Windows 构建控制台应用程序。

如果您没有 Windows 计算机，或者想要使用 Visual Studio Code，那么您可以跳过本节，因为代码将是相同的，只是工具体验不同。

## 使用 Visual Studio 2022 管理多个项目

Visual Studio 2022 有一个名为**解决方案**的概念，允许您同时打开和管理多个项目。我们将使用一个解决方案来管理您在本章中将创建的两个项目。

## 使用 Visual Studio 2022 编写代码

让我们开始编写代码吧！

1.  启动 Visual Studio 2022。

1.  在启动窗口中，点击**创建新项目**。

1.  在**创建新项目**对话框中，在**搜索模板**框中输入`console`，选择**控制台应用程序**，确保选择了 C#项目模板而不是其他语言，如 F#或 Visual Basic，如*图 1.3*所示：![](img/Image00009.jpg)

图 1.3：选择控制台应用程序项目模板

1.  点击**下一步**。

1.  在**配置新项目**对话框中，输入项目名称`HelloCS`，输入位置`C:\Code`，输入解决方案名称`Chapter01`，如*图 1.4*所示：![](img/Image00010.jpg)

图 1.4：为新项目配置名称和位置

1.  点击**下一步**。

我们故意使用较旧的.NET 5.0 项目模板，以查看完整的控制台应用程序是什么样子。在下一节中，您将使用.NET 6.0 创建一个控制台应用程序，并看到发生了什么变化。

1.  在**附加信息**对话框中，在**目标框架**下拉列表中，注意.NET 的当前版本和长期支持版本的选择，然后选择**.NET 5.0（当前）**并点击**创建**。

1.  在**解决方案资源管理器**中，双击打开名为`Program.cs`的文件，并注意**解决方案资源管理器**显示**HelloCS**项目，如*图 1.5*所示：![](img/Image00011.jpg)

图 1.5：在 Visual Studio 2022 中编辑 Program.cs

1.  在`Program.cs`中，修改第 9 行，使得写入控制台的文本为`Hello, C#!`

## 使用 Visual Studio 编译和运行代码

下一个任务是编译和运行代码。

1.  在 Visual Studio 中，导航到**调试** | **开始不带调试**。

1.  控制台窗口中的输出将显示运行应用程序的结果，如*图 1.6*所示：![图形用户界面，文本，应用程序说明自动生成](img/Image00012.jpg)

图 1.6：在 Windows 上运行控制台应用程序

1.  按任意键关闭控制台窗口并返回到 Visual Studio。

1.  选择**HelloCS**项目，然后在**解决方案资源管理器**工具栏中，切换打开**显示所有文件**按钮，并注意到编译器生成的`bin`和`obj`文件夹是可见的，如*图 1.7*所示：![图形用户界面，文本，应用程序，电子邮件描述自动生成](img/Image00013.jpg)

图 1.7：显示编译器生成的文件夹和文件

### 了解编译器生成的文件夹和文件

创建了两个编译器生成的文件夹，名为`obj`和`bin`。您不需要查看这些文件夹或了解它们的文件。只需知道编译器需要创建临时文件夹和文件来完成其工作。您可以删除这些文件夹及其文件，稍后可以重新创建。开发人员经常这样做来“清理”项目。Visual Studio 甚至在**生成**菜单上有一个名为**清理解决方案**的命令，可以为您删除其中一些临时文件。在 Visual Studio Code 中，相应的命令是`dotnet clean`。

+   `obj`文件夹包含每个源代码文件的一个已编译的*对象*文件。这些对象尚未链接到最终的可执行文件中。

+   `bin`文件夹包含应用程序或类库的*二进制*可执行文件。我们将在*第七章*中更详细地讨论这一点，*打包和分发.NET 类型*。

## 编写顶级程序

您可能会认为这是为了输出`Hello, C#!`而编写了大量代码。

虽然项目模板为您编写了样板代码，但是否有更简单的方法？

在 C# 9 或更高版本中，有一种称为**顶级程序**的方法。

让我们比较项目模板创建的控制台应用程序，如下面的代码所示：

```cs

使用

System;

命名空间

HelloCS

{

类

Program

{

静态的

void

Main

(

字符串

[] args

)

{

Console.WriteLine("Hello World!"

);

}

}

}

```

到新的顶级程序最小控制台应用程序，如下面的代码所示：

```cs

使用

系统;

Console.WriteLine("Hello World!"

);

```

这样就简单多了，对吧？如果您必须从空白文件开始编写所有语句，这样做更好。但是它是如何工作的呢？

在编译期间，将生成并包装您编写的语句周围的所有定义命名空间、`Program`类和其`Main`方法的样板代码。

关于顶级程序的要点包括以下列表：

+   任何`using`语句仍然必须放在文件的顶部。

+   在项目中只能有一个这样的文件。

文件顶部的`using System;`语句导入了`System`命名空间。这使得`Console.WriteLine`语句能够工作。您将在下一章中了解更多关于命名空间的知识。

## 使用 Visual Studio 2022 添加第二个项目

让我们向我们的解决方案添加第二个项目，以探索顶级程序：

1.  在 Visual Studio 中，导航到**文件** | **添加** | **新建项目**。

1.  在**添加新项目**对话框中，在**最近的项目模板**中，选择**控制台应用程序[C#]**，然后单击**下一步**。

1.  在**配置新项目**对话框中，对于**项目名称**，输入`TopLevelProgram`，将位置保留为`C:\Code\Chapter01`，然后单击**下一步**。

1.  在**附加信息**对话框中，选择**.NET 6.0（长期支持）**，然后单击**创建**。

1.  在**解决方案资源管理器**中，在`TopLevelProgram`项目中，双击`Program.cs`以打开它。

1.  在`Program.cs`中，请注意代码只包含一个注释和一个语句，因为它使用了 C# 9 中引入的顶级程序特性，如下面的代码所示：

```cs

// 有关更多信息，请参见 https://aka.ms/new-console-template

Console.WriteLine("Hello, World!"

);

```

但是当我之前介绍了顶级程序的概念时，我们需要一个`using System;`语句。为什么我们在这里不需要呢？

### 隐式导入的命名空间

诀窍在于，我们确实需要导入`System`命名空间，但现在我们使用了 C# 10 中引入的一个功能来为我们完成。让我们看看：

1.  在**解决方案资源管理器**中，选择`TopLevelProgram`项目并切换**显示所有文件**按钮，并注意编译器生成的`bin`和`obj`文件夹是可见的。

1.  展开`obj`文件夹，展开`Debug`文件夹，展开`net6.0`文件夹，并打开名为`TopLevelProgram.GlobalUsings.g.cs`的文件。

1.  请注意，此文件是由编译器自动为目标为.NET 6 的项目创建的，并且它使用了 C# 10 中引入的一个名为**全局导入**的功能，用于在所有代码文件中导入一些常用的命名空间，如下面的代码所示：

```cs

// <autogenerated />

全局

使用

全局

::System;

全局

使用

全局

::System.Collections.Generic;

全局

使用

全局

::System.IO;

全局

使用

全局

::System.Linq;

全局

使用

全局

::System.Net.Http;

全局

使用

全局

::System.Threading;

全局

使用

全局

::System.Threading.Tasks;

```

我将在下一章更详细地解释这个功能。现在，只需注意.NET 5 和.NET 6 之间的一个重大变化是，许多项目模板（如控制台应用程序的模板）使用了新的语言特性来隐藏实际发生的事情。

1.  在`TopLevelProgram`项目中，在`Program.cs`中，修改语句以输出不同的消息和操作系统的版本，如下面的代码所示：

```cs

Console.WriteLine("Hello from a Top Level Program!"

);

Console.WriteLine(Environment.OSVersion.VersionString);

```

1.  在**解决方案资源管理器**中，右键单击**Chapter01**解决方案，选择**设置启动项目…**，设置**当前选择**，然后单击**确定**。

1.  在**解决方案资源管理器**中，单击**TopLevelProgram**项目（或其中的任何文件或文件夹），并注意 Visual Studio 指示**TopLevelProgram**现在是启动项目，因为项目名称加粗显示。

1.  导航到**调试** | **开始调试**以运行**TopLevelProgram**项目，并注意结果，如*图 1.8*所示：![](img/Image00014.jpg)

图 1.8：在 Windows 上运行 Visual Studio 解决方案中的两个项目的顶级程序

# 使用 Visual Studio Code 构建控制台应用程序

本节的目标是展示如何使用 Visual Studio Code 构建控制台应用程序。

如果您从不想尝试 Visual Studio Code 或.NET 交互式笔记本，请随意跳过本节和下一节，然后继续*查看项目的文件夹和文件*部分。

本节中的说明和截图都是针对 Windows 的，但在 macOS 和 Linux 变体上使用 Visual Studio Code 时，相同的操作也适用。

主要区别在于本机命令行操作，例如删除文件：在 Windows 或 macOS 和 Linux 上，命令和路径可能不同。幸运的是，`dotnet`命令行工具在所有平台上都是相同的。

## 使用 Visual Studio Code 管理多个项目

Visual Studio Code 有一个名为**工作区**的概念，允许您同时打开和管理多个项目。我们将使用一个工作区来管理您将在本章中创建的两个项目。

## 使用 Visual Studio Code 编写代码

让我们开始编写代码！

1.  启动 Visual Studio Code。

1.  确保您没有打开任何文件、文件夹或工作区。

1.  导航到**文件** | **另存工作区为…**。

1.  在对话框中，导航到 macOS 上的用户文件夹（我的名字是`markjprice`），Windows 上的`Documents`文件夹，或者您想要保存项目的任何目录或驱动器。

1.  单击**新建文件夹**按钮，将文件夹命名为`Code`。（如果您已完成 Visual Studio 2022 的部分，则此文件夹已经存在。）

1.  在`Code`文件夹中，创建一个名为`Chapter01-vscode`的新文件夹。

1.  在`Chapter01-vscode`文件夹中，将工作区保存为`Chapter01.code-workspace`。

1.  导航到**文件** | **添加文件夹到工作区…**或单击**添加文件夹**按钮。

1.  在`Chapter01-vscode`文件夹中，创建一个名为`HelloCS`的新文件夹。

1.  选择`HelloCS`文件夹，然后单击**添加**按钮。

1.  导航到**视图** | **终端**。

我们故意使用较旧的.NET 5.0 项目模板，以查看完整的控制台应用程序的外观。在下一节中，您将使用.NET 6.0 创建控制台应用程序，并查看发生了什么变化。

1.  在**终端**中，确保您在`HelloCS`文件夹中，然后使用`dotnet`命令行工具创建一个新的目标为.NET 5.0 的控制台应用程序，如下命令所示：

```cs

dotnet new console -f net5.0

```

1.  您将看到`dotnet`命令行工具在当前文件夹中为您创建了一个新的**控制台应用程序**项目，**资源管理器**窗口显示了创建的两个文件`HelloCS.csproj`和`Program.cs`，以及`obj`文件夹，如*图 1.9*所示：![](img/Image00015.jpg)

图 1.9：**资源管理器**窗口将显示已创建两个文件和一个文件夹

1.  在**资源管理器**中，单击名为`Program.cs`的文件，以在编辑器窗口中打开它。第一次这样做时，Visual Studio Code 可能需要下载并安装 C#依赖项，如 OmniSharp、.NET Core Debugger 和 Razor Language Server，如果在安装 C#扩展时没有这样做，或者如果它们需要更新。Visual Studio Code 将在**输出**窗口中显示进度，并最终显示消息`完成`，如下输出所示：

```cs

安装 C#依赖项...

平台：win32，x86_64

下载包'OmniSharp for Windows (.NET 4.6 / x64)'(36150 KB)....................完成！

验证下载...

完整性检查成功。

安装包'OmniSharp for Windows (.NET 4.6 / x64)'

下载包'.NET Core Debugger (Windows / x64)'(45048 KB)....................完成！

验证下载...

完整性检查成功。

安装包'.NET Core Debugger (Windows / x64)'

下载包'Razor Language Server (Windows / x64)'(52344 KB)....................完成！

安装包'Razor Language Server (Windows / x64)'

完成

```

上述输出来自 Windows 上的 Visual Studio Code。在 macOS 或 Linux 上运行时，输出看起来可能略有不同，但是您的操作系统的等效组件将被下载和安装。 

1.  将创建名为`obj`和`bin`的文件夹，当您看到通知说缺少所需的资源时，单击**是**，如*图 1.10*所示：![图形用户界面，文本，应用程序，电子邮件描述自动生成](img/Image00016.jpg)

图 1.10：警告消息，添加所需的构建和调试资源

1.  如果通知在您与其交互之前消失，则可以单击状态栏最右侧的铃铛图标，以再次显示它。

1.  几秒钟后，将创建另一个名为`.vscode`的文件夹，其中包含一些由 Visual Studio Code 使用的文件，用于在调试期间提供 IntelliSense 等功能，您将在*第四章*，*编写、调试和测试函数*中了解更多信息。

1.  在`Program.cs`中，修改第 9 行，以便写入控制台的文本为`Hello, C#!`

**良好实践**：导航到**文件** | **自动保存**。此切换将保存在每次重建应用程序之前记得保存的烦恼。

## 使用 dotnet CLI 编译和运行代码

下一个任务是编译和运行代码：

1.  导航到**视图** | **终端**，并输入以下命令：

```cs

dotnet run

```

1.  **终端**窗口中的输出将显示运行应用程序的结果，如*图 1.11*所示：![图形用户界面，文本，应用程序，电子邮件描述自动生成](img/Image00017.jpg)

图 1.11：运行您的第一个控制台应用程序的输出

## 使用 Visual Studio Code 添加第二个项目

让我们向我们的工作区添加第二个项目来探索顶级程序：

1.  在 Visual Studio Code 中，导航到**File** | **Add Folder to Workspace…**。

1.  在`Chapter01-vscode`文件夹中，使用**New Folder**按钮创建一个名为`TopLevelProgram`的新文件夹，选择它，然后单击**Add**。

1.  导航到**Terminal** | **New Terminal**，然后在出现的下拉列表中选择**TopLevelProgram**。或者，在**EXPLORER**中，右键单击`TopLevelProgram`文件夹，然后选择**Open in Integrated Terminal**。

1.  在**TERMINAL**中，确认您在`TopLevelProgram`文件夹中，然后输入创建新控制台应用程序的命令，如下命令所示：

```cs

dotnet new console

```

**良好实践**：在使用工作区时，在**TERMINAL**中输入命令时要小心。在输入可能具有破坏性的命令之前，请确保您在正确的文件夹中！这就是为什么我让您在发出创建新控制台应用程序的命令之前为`TopLevelProgram`创建一个新终端的原因。

1.  导航到**View** | **Command Palette**。

1.  输入`omni`，然后在出现的下拉列表中选择**OmniSharp: Select Project**。

1.  在两个项目的下拉列表中，选择**TopLevelProgram**项目，并在提示时单击**Yes**以添加所需的资产以进行调试。

**良好实践**：为了启用调试和其他有用的功能，如代码格式化和转到定义，您必须告诉 OmniSharp 您正在 Visual Studio Code 中积极工作的项目。您可以通过单击状态栏左侧火焰图标右侧的项目/文件夹快速切换活动项目。

1.  在**EXPLORER**中，在`TopLevelProgram`文件夹中，选择`Program.cs`，然后更改现有语句以输出不同的消息，并输出操作系统版本字符串，如下面的代码所示：

```cs

Console.WriteLine("Hello from a Top Level Program!"

);

Console.WriteLine(Environment.OSVersion.VersionString);

```

1.  在**TERMINAL**中，输入运行程序的命令，如下命令所示：

```cs

dotnet run

```

1.  注意**TERMINAL**窗口中的输出，如*图 1.12*所示：![](img/Image00018.jpg)

图 1.12：在 Windows 上运行 Visual Studio Code 工作区中的两个项目的顶级程序

如果您在 macOS Big Sur 上运行程序，则环境操作系统将不同，如下面的输出所示：

```cs

来自顶级程序的问候！

Unix 11.2.3

```

## 使用 Visual Studio Code 管理多个文件

如果您有多个文件要同时使用，则可以将它们并排放置以进行编辑：

1.  在**EXPLORER**中，展开两个项目。

1.  从两个项目的`Program.cs`文件中打开。

1.  点击，按住并拖动一个打开文件的编辑窗口标签，以便您可以同时看到两个文件。

# 使用.NET 交互式笔记本探索代码

.NET 交互式笔记本使编写代码比顶级程序更容易。它需要 Visual Studio Code，所以如果您之前没有安装它，请立即安装。

## 创建一个笔记本

首先，我们需要创建一个笔记本：

1.  在 Visual Studio Code 中，关闭任何打开的工作区或文件夹。

1.  导航到**View** | **Command Palette**。

1.  输入`.net inter`，然后选择**.NET Interactive: Create new blank notebook**，如*图 1.13*所示：![](img/Image00019.jpg)

图 1.13：创建一个新的空白.NET 笔记本

1.  在提示选择文件扩展名时，选择**Create as '.dib'**。

`.dib`是由 Microsoft 定义的实验性文件格式，旨在避免与 Python 交互式笔记本使用的`.ipynb`格式引起混淆和兼容性问题。该文件扩展名历史上仅用于可以包含笔记本文件（NB）中的数据、Python 代码（PY）和输出的交互式（I）混合的 Jupyter 笔记本。使用.NET Interactive 笔记本，这个概念已经扩展到允许混合 C#、F#、SQL、HTML、JavaScript、Markdown 和其他语言。`.dib`是多语言的，意味着它支持混合语言。支持在`.dib`和`.ipynb`文件格式之间进行转换。

1.  选择**C#**作为笔记本中代码单元格的默认语言。

1.  如果有更新版本的.NET Interactive 可用，您可能需要等待它卸载旧版本并安装新版本。转到**视图** | **输出**，并在下拉列表中选择**.NET Interactive：诊断**。请耐心等待。笔记本可能需要几分钟才能出现，因为它必须启动.NET 的托管环境。如果几分钟后什么都没有发生，那么关闭 Visual Studio Code 并重新启动它。

1.  一旦下载并安装了.NET Interactive 笔记本扩展，**OUTPUT**窗口诊断将显示内核进程已启动（您的进程和端口号将与下面的输出不同），如下面的输出所示，已编辑以节省空间：

```cs

扩展已启动，适用于 VS Code Stable。

...

内核进程 12516 端口 59565 正在使用隧道 uri http://localhost:59565/

```

## 在笔记本中编写和运行代码

接下来，我们可以在笔记本单元格中编写代码：

1.  第一个单元格应该已经设置为**C# (.NET Interactive)**，但如果设置为其他任何内容，则单击代码单元格右下角的语言选择器，然后选择**C# (.NET Interactive)**作为该单元格的语言模式，并注意您的其他代码单元格语言选择，如*图 1.14*所示：![](img/Image00020.jpg)

图 1.14：在.NET Interactive 笔记本中更改代码单元格的语言

1.  在**C# (.NET Interactive)**代码单元格中，输入一条语句以将消息输出到控制台，并注意您不需要像在完整应用程序中那样以分号结束语句，如下面的代码所示：

```cs

Console.WriteLine("Hello, .NET Interactive!"

)

```

1.  单击代码单元格左侧的**执行单元格**按钮，并注意出现在代码单元格下方灰色框中的输出，如*图 1.15*所示：![](img/Image00021.jpg)

图 1.15：在笔记本中运行代码并在下方看到输出

## 保存笔记本

与任何其他文件一样，我们应该在继续之前保存笔记本：

1.  转到**文件** | **另存为…**。

1.  切换到`Chapter01-vscode`文件夹，并将笔记本保存为`Chapter01.dib`。

1.  关闭`Chapter01.dib`编辑器选项卡。

## 向笔记本添加 Markdown 和特殊命令

我们可以混合和匹配包含 Markdown 和特殊命令的单元格：

1.  转到**文件** | **打开文件…**，并选择`Chapter01.dib`文件。

1.  如果提示`Do you` `trust the authors of these files?`，请单击**打开**。

1.  将鼠标悬停在代码块上方，单击**+标记**以添加 Markdown 单元格。

1.  输入一级标题，如下所示的 Markdown：

```cs

# 第一章 - 你好，C#! 欢迎，.NET!

混合*丰富*

**text**

和代码很酷！

```

1.  单击单元格右上角的勾号以停止编辑单元格并查看处理后的 Markdown。

如果您的单元格顺序不正确，则可以拖放以重新排列它们。

1.  将鼠标悬停在 Markdown 单元格和代码单元格之间，然后单击**+代码**。

1.  输入一个特殊命令以输出.NET Interactive 的版本信息，如下所示的代码：

```cs

#!about

```

1.  单击**执行单元格**按钮并注意输出，如*图 1.16*所示：![](img/Image00022.jpg)

图 1.16：在.NET Interactive 笔记本中混合 Markdown、代码和特殊命令

## 在多个单元格中执行代码

当你在笔记本中有多个代码单元格时，你必须在后续的代码单元格中执行前面的代码单元格，才能使用它们的上下文：

1.  在笔记本的底部，添加一个新的代码单元格，然后输入一个语句来声明一个变量并赋予一个整数值，如下面的代码所示：

```cs

int

number = 8

;

```

1.  在笔记本的底部，添加一个新的代码单元格，然后输入一个语句来输出`number`变量，如下面的代码所示：

```cs

Console.WriteLine(number);

```

1.  请注意，第二个代码单元格不知道`number`变量，因为它是在另一个代码单元格中定义和赋值的，也就是上下文，如*图 1.17*所示：![](img/Image00023.jpg)

图 1.17：当前单元格或上下文中不存在 number 变量

1.  在第一个单元格中，点击**执行单元格**按钮来声明并赋值给变量一个值，然后在第二个单元格中，点击**执行单元格**按钮来输出`number`变量，并注意这是有效的。（或者，在第一个单元格中，你可以点击**执行单元格及以下**按钮。）

**良好的实践**：如果你的相关代码分散在两个单元格中，请记得在执行后续单元格之前执行前面的单元格。在笔记本的顶部，有以下按钮 - **清除输出** 和 **运行全部**。这些非常方便，因为你可以依次点击它们，以确保所有代码单元格都按正确的顺序执行。

## 使用本书中的代码的.NET Interactive 笔记本

在接下来的章节中，我不会明确说明使用笔记本，但是本书的 GitHub 存储库在适当的时候会有解决方案笔记本。我希望许多读者会想要运行我预先创建的笔记本，以了解*第 2*至*12*章中涵盖的语言和库特性，并且不必编写一个完整的应用程序，即使只是一个控制台应用程序：

[`github.com/markjprice/cs10dotnet6/tree/main/notebooks`](https://github.com/markjprice/cs10dotnet6/tree/main/notebooks)

# 审查项目的文件夹和文件

在本章中，你创建了两个名为`HelloCS`和`TopLevelProgram`的项目。

Visual Studio Code 使用工作区文件来管理多个项目。Visual Studio 2022 使用解决方案文件来管理多个项目。你还创建了一个.NET Interactive 笔记本。

结果是一个文件夹结构和文件，将在后续章节中重复出现，尽管不仅仅是两个项目，如*图 1.18*所示：

![](img/Image00024.jpg)

图 1.18：本章中两个项目的文件夹结构和文件

## 理解常见的文件夹和文件

尽管`.code-workspace`和`.sln`文件是不同的，但是`HelloCS`和`TopLevelProgram`等项目文件夹和文件对于 Visual Studio 2022 和 Visual Studio Code 是相同的。这意味着你可以在这两个代码编辑器之间进行混合和匹配：

+   在 Visual Studio 2022 中，打开一个解决方案，导航到**文件** | **添加现有项目…** 来添加另一个工具创建的项目文件。

+   在 Visual Studio Code 中，打开一个工作区，导航到**文件** | **添加文件夹到工作区…** 来添加另一个工具创建的项目文件夹。

**良好的实践**：尽管源代码，如`.csproj`和`.cs`文件是相同的，但是编译器自动生成的`bin`和`obj`文件夹可能具有不匹配的文件版本，导致错误。如果你想在 Visual Studio 2022 和 Visual Studio Code 中打开同一个项目，删除在另一个代码编辑器中打开项目之前的临时`bin`和`obj`文件夹。这就是为什么我要求你在本章中为 Visual Studio Code 解决方案创建一个不同的文件夹。

## 理解 GitHub 上的解决方案代码

本书的 GitHub 存储库中的解决方案代码包括 Visual Studio Code、Visual Studio 2022 和.NET Interactive 笔记本文件的单独文件夹，如下所示：

+   Visual Studio 2022 解决方案：[`github.com/markjprice/cs10dotnet6/tree/main/vs4win`](https://github.com/markjprice/cs10dotnet6/tree/main/vs4win)

+   Visual Studio Code 解决方案：[`github.com/markjprice/cs10dotnet6/tree/main/vscode`](https://github.com/markjprice/cs10dotnet6/tree/main/vscode)

+   .NET Interactive 笔记本解决方案：[`github.com/markjprice/cs10dotnet6/tree/main/notebooks`](https://github.com/markjprice/cs10dotnet6/tree/main/notebooks)

**良好实践**：如果需要的话，可以返回本章，以便提醒自己如何在所选的代码编辑器中创建和管理多个项目。GitHub 存储库提供了四种代码编辑器（Windows 的 Visual Studio 2022，Visual Studio Code，Mac 的 Visual Studio 2022 和 JetBrains Rider）的逐步说明，以及额外的屏幕截图：[`github.com/markjprice/cs10dotnet6/blob/main/docs/code-editors/`](https://github.com/markjprice/cs10dotnet6/blob/main/docs/code-editors/)。

# 充分利用本书的 GitHub 存储库

Git 是一种常用的源代码管理系统。GitHub 是一个公司、网站和桌面应用程序，使 Git 的管理变得更加容易。微软于 2018 年收购了 GitHub，因此它将继续与微软工具更紧密地集成。

我为这本书创建了一个 GitHub 存储库，并且我用它来做以下事情：

+   存储书中可以在印刷出版日期后继续维护的解决方案代码。

+   提供扩展书籍的额外材料，如勘误修正、小改进、有用链接列表和无法放入印刷书中的长文章。

+   为读者提供一个与我联系的地方，如果他们在阅读书籍时遇到问题。

## 提出书中的问题

如果您在阅读本书时遇到任何问题，或者发现文本或解决方案中的代码错误，请在 GitHub 存储库中提出问题：

1.  使用您喜欢的浏览器导航到以下链接：[`github.com/markjprice/cs10dotnet6/issues`](https://github.com/markjprice/cs10dotnet6/issues)。

1.  点击**New Issue**。

1.  输入尽可能多的细节，这将帮助我诊断问题。例如：

1.  您的操作系统，例如 Windows 11 64 位，或 macOS Big Sur 版本 11.2.3。

1.  您的硬件，例如 Intel、Apple Silicon 或 ARM CPU。

1.  您的代码编辑器，例如 Visual Studio 2022，Visual Studio Code，或其他任何东西，包括版本号。

1.  您认为相关和必要的代码和配置的大部分内容。

1.  预期行为和实际行为的描述。

1.  屏幕截图（如果可能）。

写这本书对我来说是一种副业。我有一份全职工作，所以我大部分时间都在周末工作。这意味着我不能总是立即回复问题。但我希望所有的读者都能成功阅读我的书，所以如果我可以在不太麻烦的情况下帮助您（和其他人），那么我会很乐意这样做。

## 给我反馈

如果您想就本书给我提供更一般的反馈意见，那么 GitHub 存储库的`README.md`页面上有一些调查链接。您可以匿名提供反馈意见，或者如果您希望得到我的回复，那么您可以提供一个电子邮件地址。我只会使用这个电子邮件地址来回复您的反馈。

我很乐意听取读者对我的书喜欢的内容，以及改进建议以及他们如何使用 C#和.NET，所以不要害羞。请与我联系！

提前感谢您的周到和建设性的反馈。

## 从 GitHub 存储库下载解决方案代码

我使用 GitHub 存储了所有章节中的逐步操作、逐步编码示例的解决方案以及每章末尾的实践练习。您可以在以下链接找到存储库：[`github.com/markjprice/cs10dotnet6`](https://github.com/markjprice/cs10dotnet6)。

如果您只想下载所有解决方案文件而不使用 Git，请单击绿色的**Code**按钮，然后选择**Download ZIP**，如*图 1.19*所示：

![自动生成的表描述](img/Image00025.jpg)

图 1.19：下载 ZIP 文件的存储库

我建议您将上述链接添加到您喜爱的书签中，因为我还使用 GitHub 存储库发布勘误（更正）和其他有用的链接。

## 使用 Visual Studio Code 和命令行的 Git

Visual Studio Code 支持 Git，但它将使用您操作系统的 Git 安装，因此您必须先安装 Git 2.0 或更高版本，然后才能使用这些功能。

您可以从以下链接安装 Git：[`git-scm.com/download`](https://git-scm.com/download)。

如果您喜欢使用 GUI，可以从以下链接下载 GitHub Desktop：[`desktop.github.com`](https://desktop.github.com)。

### 克隆书籍解决方案代码存储库

让我们克隆书籍解决方案代码存储库。在接下来的步骤中，您将使用 Visual Studio Code 终端，但您也可以在任何命令提示符或终端窗口中输入命令：

1.  在您的用户文件夹或`Documents`文件夹中创建一个名为`Repos-vscode`的文件夹，或者您想要存储 Git 存储库的任何位置。

1.  在 Visual Studio Code 中，打开`Repos-vscode`文件夹。

1.  导航到**View** | **Terminal**，并输入以下命令：

```cs

git clone https://github.com/markjprice/cs10dotnet6.git

```

1.  请注意，克隆所有章节的所有解决方案将需要一分钟左右，如*图 1.20*所示：![自动生成的图形用户界面、文本、应用程序、电子邮件描述](img/Image00026.jpg)

图 1.20：使用 Visual Studio Code 克隆书籍解决方案代码

# 寻求帮助

本节介绍如何在网络上找到有关编程的高质量信息。

## 阅读微软文档

获取 Microsoft 开发人员工具和平台帮助的权威资源是 Microsoft Docs，您可以在以下链接找到：[`docs.microsoft.com/`](https://docs.microsoft.com/)。

## 获取`dotnet`工具的帮助

在命令行中，您可以使用`dotnet`工具来获取其命令的帮助：

1.  要在浏览器窗口中打开`dotnet new`命令的官方文档，请在命令行或 Visual Studio Code 终端中输入以下内容：

```cs

dotnet help new

```

1.  要在命令行输出帮助信息，请使用`-h`或`--help`标志，如下命令所示：

```cs

dotnet new console -h

```

1.  您将看到以下部分输出：

```cs

控制台应用程序（C#）

作者：Microsoft

描述：用于在 Windows、Linux 和 macOS 上运行.NET Core 的命令行应用程序的项目

选项：

-f|--framework。项目的目标框架。

net6.0           - 目标 net6.0

net5.0           - 目标 net5.0

netcoreapp3.1\.   - 目标 netcoreapp3.1

netcoreapp3.0\.   - 目标 netcoreapp3.0

默认：net6.0

--langVersion    在创建的项目文件文本中设置 langVersion – 可选

```

## 获取类型及其成员的定义

代码编辑器中最有用的功能之一是**转到定义**。它在 Visual Studio Code 和 Visual Studio 2022 中可用。通过读取编译后的程序集中的元数据，它将显示类型或成员的公共定义是什么样子。

一些工具，如 ILSpy .NET 反编译器，甚至可以从元数据和 IL 代码反向工程到 C#。

让我们看看如何使用**转到定义**功能：

1.  在 Visual Studio 2022 或 Visual Studio Code 中，打开名为`Chapter01`的解决方案/工作区。

1.  在`HelloCS`项目中，在`Program.cs`中，在`Main`中，输入以下语句声明一个名为`z`的整数变量：

```cs

int

z;

```

1.  单击`int`内部，然后右键单击并选择**转到定义**。

1.  在出现的代码窗口中，您可以看到`int`数据类型的定义，如*图 1.21*所示：![图形用户界面，文本，应用程序自动生成的描述](img/Image00027.jpg)

图 1.21：int 数据类型元数据

您可以看到`int`：

+   使用`struct`关键字定义

+   在`System.Runtime`程序集中

+   在`System`命名空间中

+   命名为`Int32`

+   因此是`System.Int32`类型的别名

+   实现诸如`IComparable`之类的接口

+   具有其最大和最小值的常量值

+   具有诸如`Parse`之类的方法

**良好实践**：当您尝试在 Visual Studio Code 中使用**转到定义**时，有时会看到一个错误，说**找不到定义**。这是因为 C#扩展不知道当前项目。要解决此问题，请导航到**视图** | **命令面板**，输入`omni`，选择**OmniSharp: 选择项目**，然后选择要使用的项目。

现在，**转到定义**功能对您来说并不那么有用，因为您还不知道所有这些信息的含义。

在本书的第一部分结束时，该部分由*章节* *2*至*6*组成，教授您有关 C#的知识，您将了解足够多的内容，以便此功能变得非常方便。

1.  在代码编辑器窗口中，向下滚动以找到第 106 行上具有单个`string`参数的`Parse`方法，以及在第 86 到 105 行上记录它的注释，如*图 1.22*所示：![图形用户界面，文本，应用程序自动生成的描述](img/Image00028.jpg)

图 1.22：具有字符串参数的 Parse 方法的注释

在注释中，您将看到 Microsoft 已记录以下内容：

+   描述该方法的摘要。

+   方法可以传递的参数，如`string`值。

+   方法的返回值，包括其数据类型。

+   如果调用此方法可能发生的三种异常，包括`ArgumentNullException`，`FormatException`和`OverflowException`。现在，我们知道我们可以选择在`try`语句中包装对此方法的调用以及要捕获的异常。

希望您已经迫不及待地想要了解所有这些含义！

再等一会儿。您几乎已经到达本章的末尾，在下一章中，您将深入了解 C#语言的细节。但首先，让我们看看您还可以在哪里寻求帮助。

## 在 Stack Overflow 上寻找答案

Stack Overflow 是获取困难编程问题答案的最受欢迎的第三方网站。它非常受欢迎，以至于像 DuckDuckGo 这样的搜索引擎有一种特殊的方法来编写查询以搜索该网站：

1.  启动您喜欢的网络浏览器。

1.  导航到[DuckDuckGo.com](https://duckduckgo.com/)，输入以下查询，并注意搜索结果，也显示在*图 1.23*中：

```cs

！so securestring

```

![图形用户界面，文本，应用程序自动生成的描述](img/Image00029.jpg)

图 1.23：securestring 的 Stack Overflow 搜索结果

## 使用 Google 搜索答案

您可以使用高级搜索选项在 Google 上搜索，以增加找到所需内容的可能性：

1.  导航到 Google。

1.  使用简单的 Google 查询搜索有关`垃圾收集`的信息，并注意在看到计算机科学中垃圾收集的维基百科定义之前，您可能会看到很多关于本地地区垃圾收集服务的广告。

1.  通过将搜索限制为诸如 Stack Overflow 之类的有用网站，并删除我们可能不关心的语言，例如 C++、Rust 和 Python，或者显式添加 C#和.NET，来改进搜索，如下面的搜索查询所示：

```cs

垃圾收集 site:stackoverflow.com +C# -Java

```

## 订阅官方.NET 博客

要了解.NET 的最新信息，可以订阅官方.NET 博客，由.NET 工程团队撰写，链接如下：[`devblogs.microsoft.com/dotnet/`](https://devblogs.microsoft.com/dotnet/)。

## 观看 Scott Hanselman 的视频

来自微软的 Scott Hanselman 拥有一个关于计算机知识的优秀 YouTube 频道：[`computerstufftheydidntteachyou.com/`](http://computerstufftheydidntteachyou.com/)。

我建议每个与计算机工作的人都去看。

# 练习和探索

现在让我们通过尝试回答一些问题、进行一些动手实践，并更详细地了解本章涵盖的主题来测试您的知识和理解。

## 练习 1.1 – 测试你的知识

尝试回答以下问题，记住虽然大多数答案可以在本章中找到，但您应该进行一些在线研究或编写代码来回答其他问题：

1.  Visual Studio 2022 是否比 Visual Studio Code 更好？

1.  .NET 6 是否比.NET Framework 更好？

1.  .NET Standard 是什么，为什么它仍然重要？

1.  为什么程序员可以使用不同的语言，例如 C#和 F#，来编写在.NET 上运行的应用程序？

1.  .NET 控制台应用程序的入口方法的名称是什么，应该如何声明？

1.  什么是顶级程序，如何访问任何命令行参数？

1.  在提示符下输入什么来构建和执行 C#源代码？

1.  使用.NET 交互式笔记本编写 C#代码有哪些好处？

1.  您会在哪里寻找 C#关键字的帮助？

1.  您会在哪里寻找常见编程问题的解决方案？

*附录*，*测试你的知识问题的答案*，可以从 GitHub 存储库的 README 中的链接下载：[`github.com/markjprice/cs10dotnet6`](https://github.com/markjprice/cs10dotnet6)。

## 练习 1.2 – 在任何地方练习 C#

你不需要 Visual Studio Code 甚至 Visual Studio 2022 for Windows 或 Mac 来编写 C#。您可以转到.NET Fiddle – [`dotnetfiddle.net/`](https://dotnetfiddle.net/) – 并开始在线编码。

## 练习 1.3 – 探索主题

一本书是一种精心策划的体验。我尽量找到印刷书中包含的主题的合适平衡。我写的其他内容可以在本书的 GitHub 存储库中找到。

我相信这本书涵盖了 C#和.NET 开发人员应该具备或了解的所有基本知识和技能。一些较长的示例最好包含为链接到 Microsoft 文档或第三方文章作者。

使用以下页面上的链接了解本章涵盖的主题的更多细节：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-1---hello-c-welcome-net`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-1---hello-c-welcome-net)

# 总结

在本章中，我们：

+   设置你的开发环境。

+   讨论了现代.NET、.NET Core、.NET Framework、Xamarin 和.NET Standard 之间的相似性和差异。

+   使用 Visual Studio Code 与.NET SDK 以及 Visual Studio 2022 for Windows 创建一些简单的控制台应用程序。

+   使用.NET 交互式笔记本执行代码片段进行学习。

+   学会如何从 GitHub 存储库下载本书的解决方案代码。

+   最重要的是，学会如何寻求帮助。

在下一章中，您将学习如何“说”C#。
