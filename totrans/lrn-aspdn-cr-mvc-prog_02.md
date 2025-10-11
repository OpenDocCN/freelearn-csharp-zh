# 第 2 章. 设置环境

在任何开发项目中，设置正确类型的开发环境至关重要，这样你就可以专注于开发解决方案，而不是解决环境问题或配置问题。对于 .NET 来说，Visual Studio 是构建 .NET 网络应用程序的默认标准 **IDE**（**集成开发环境**）。

在本章中，你将学习以下主题：

+   IDE 的用途

+   Visual Studio 的不同版本

+   Visual Studio Community 2015 的安装

+   创建您的第一个 ASP.NET MVC 5 项目和项目结构

# IDE 的用途

首先，让我们看看为什么我们需要 IDE，当你可以在记事本中输入代码、编译并执行它时。

当你开发网络应用程序时，你可能需要以下东西以提高生产力：

+   **代码编辑器**：这是你输入代码的文本编辑器。你的代码编辑器应该能够识别不同的结构，例如你的编程语言的 `if` 条件、`for` 循环。在 Visual Studio 中，所有关键字都会以蓝色突出显示。

+   **Intellisense**：Intellisense 是一种上下文感知的代码补全功能，在大多数现代 IDE（包括 Visual Studio）中都可用。例如，当你在对象后面输入一个点时，这个 *Intellisense* 功能会列出该对象上所有可用的方法。这有助于开发者更快、更轻松地编写代码。

+   **构建/发布**：如果你能够通过单次点击或单条命令构建或发布应用程序，那将非常有帮助。Visual Studio 提供了多种选项，可以一键构建单独的项目或构建整个解决方案。这使得应用程序的构建和部署更加容易。

+   **模板**：根据应用程序的类型，你可能需要创建不同的文件夹和文件，以及样板代码。因此，如果你的 IDE 支持创建不同类型的模板，那将非常有帮助。Visual Studio 生成不同类型的模板，包括 ASP.NET Web Forms、MVC 和 Web API 的代码，以便你快速开始。

+   **易于添加项目**：你的 IDE 应该允许你轻松地添加不同类型的项。例如，你应该能够无问题地添加一个 XML 文件。如果 XML 文件的结构有任何问题，它应该能够突出显示问题并提供帮助信息，以帮助你解决问题。

# 读累了记得休息一会哦~

**公众号：古德猫宁李**

+   电子书搜索下载

+   书单分享

+   书友学习交流

**网站**：[沉金书屋 https://www.chenjin5.com](https://www.chenjin5.com)

+   电子书搜索下载

+   电子书打包资源分享

+   学习资源分享

# Visual Studio 的产品

可用的 Visual Studio 2015 版本有多种，以满足开发者和组织机构的不同需求。主要来说，Visual Studio 2015 有以下四种版本：

+   **Visual Studio Community**

+   **Visual Studio Professional**

+   **Visual Studio Enterprise**

+   **Visual Studio Test Professional**

## 系统要求

Visual Studio 可以安装在运行 Windows 7 Service Pack 1 操作系统及更高版本的计算机上。您可以从以下 URL 获取完整的系统要求列表：

[Visual Studio 2015 系统要求](https://www.visualstudio.com/en-us/downloads/visual-studio-2015-system-requirements-vs.aspx)

### Visual Studio Community 2015

这是一个功能齐全的 IDE，可用于构建桌面应用程序、Web 应用程序和云服务。对于个人用户，它是免费提供的。

您可以从以下 URL 下载 Visual Studio Community：

[Visual Studio Community](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx)

在本书中，我们将使用 Visual Studio Community 版本进行开发，因为它对个人开发者免费提供。

### Visual Studio Professional

如其名所示，Visual Studio Professional 面向专业开发者，包含如 **Code Lens** 提高团队生产力的功能。它还具备增强团队协作的功能。

### Visual Studio Enterprise

Visual Studio Enterprise 是 Visual Studio 的完整版本，包含用于协作的完整功能集，包括团队基础服务器、建模和测试。

### Visual Studio Test Professional

Visual Studio Test Professional 主要面向测试团队或参与测试的人员，可能包括开发者。在任何软件开发方法中，无论是瀑布模型还是敏捷开发者，都需要执行他们正在开发的代码的开发套件测试用例。

# 安装 Visual Studio Community

按照以下步骤安装 Visual Studio Community 2015：

1.  访问以下链接下载 Visual Studio Community 2015：

    [Visual Studio Community](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx)

    ![安装 Visual Studio Community](img/Image00006.jpg)

1.  点击 **下载 Community 2015** 按钮。将文件保存在您可以轻松检索的文件夹中：![安装 Visual Studio Community](img/Image00007.jpg)

1.  运行下载的可执行文件：![安装 Visual Studio Community](img/Image00008.jpg)

1.  点击 **运行** 按钮，将出现以下屏幕：![安装 Visual Studio Community](img/Image00009.jpg)

安装有两种类型——默认安装和自定义安装。默认安装安装最常用的功能，这将覆盖大多数开发者的使用场景。自定义安装可以帮助您自定义要安装的组件：

1.  选择安装类型后，点击 **安装** 按钮。

1.  根据您的内存和处理器速度，安装过程可能需要 1 到 2 小时。![安装 Visual Studio Community](img/Image00010.jpg)

1.  一旦所有组件都安装完成，你将看到以下 **设置完成** 界面：![安装 Visual Studio Community](img/Image00011.jpg)

# 安装 ASP.NET 5

当我们安装 Visual Studio Community 2015 版本时，ASP.NET 5 将默认安装。由于 ASP.NET Core 应用程序运行在 ASP.NET 5 之上，我们需要安装 ASP.NET 5。安装 ASP.NET 5 有几种方法：

+   从 [https://get.asp.net/](https://get.asp.net/) 获取 ASP.NET 5 ![安装 ASP.NET 5](img/Image00012.jpg)

+   另一个选项是从 Visual Studio 的 **新建项目** 模板中安装

这个选项稍微简单一些，因为你不需要搜索和安装。

以下为详细步骤：

1.  通过选择 **文件** | **新建 | 项目** 或使用快捷键 *Ctrl* + *Shift* + *N* 创建一个新的项目：![安装 ASP.NET 5](img/Image00013.jpg)

1.  选择 **ASP.NET Web 应用程序** 并输入项目名称，然后点击 **确定**：![安装 ASP.NET 5](img/Image00014.jpg)

1.  将出现以下窗口以选择模板。选择以下截图所示的 **获取 ASP.NET 5 RC** 选项：![安装 ASP.NET 5](img/Image00015.jpg)

1.  当你在前面的屏幕上点击 **确定** 时，将出现以下窗口：![安装 ASP.NET 5](img/Image00016.jpg)

1.  当你在前面的对话框中点击 **运行** 或 **保存** 按钮时，将出现以下屏幕，要求进行 ASP.NET 5 设置。选择复选框，**我同意许可条款和条件** 并点击 **安装** 按钮：![安装 ASP.NET 5](img/Image00017.jpg)

1.  安装 ASP.NET 5 可能需要几个小时。一旦完成，你将看到以下界面：![安装 ASP.NET 5](img/Image00018.jpg)

在安装 **ASP.NET 5 RC1 更新 1** 的过程中，可能会要求你关闭 Visual Studio。如果要求，请照做。

# ASP.NET 5应用程序中的项目结构

一旦成功安装 ASP.NET 5 RC1，打开 Visual Studio，创建一个新的项目，并选择以下截图所示的 ASP.NET 5 **Web 应用程序**：

![ASP.NET 5应用程序中的项目结构](img/Image00019.jpg)

将创建一个新的项目，其结构如下：

![ASP.NET 5应用程序中的项目结构](img/Image00020.jpg)

## 基于文件的项目

无论何时你在文件系统中添加文件或文件夹（在 ASP.NET 5 项目文件夹内），更改将自动反映在你的应用程序中。

### 支持 .NET 和 .NET Core 全功能

你可能已经注意到了前面项目中的几个引用：**DNX 4.5.1** 和 **DNX Core 5.0**。**DNX 4.5.1** 提供了完整 .NET 的功能，而 **DNX Core 5.0** 仅支持核心功能，如果你要在跨平台（如 Apple OS X、Linux）上部署应用程序，则会使用这些功能。在下一章中，将解释在 Linux 机器上开发和部署 ASP.NET Core 应用程序的过程。

### Project.json 包

通常情况下，在一个 ASP.NET 网络应用程序中，我们会将程序集作为引用，并在 C# 项目文件中列出引用列表。但在 ASP.NET 5 应用程序中，我们有一个名为 `Project.json` 的 JSON 文件，它将包含所有必要的配置以及所有以 `NuGet` 包形式的 .NET 依赖项。这使得依赖项管理变得更加容易。`NuGet` 是由微软提供的一个包管理器，它使得包的安装和卸载更加简单。在 `NuGet` 之前，所有依赖项都必须手动安装。依赖项部分标识了应用程序可用的依赖项包列表。框架部分告诉我们应用程序支持哪些框架。脚本部分标识了在应用程序构建过程中要执行的脚本。任何部分都可以使用包含和排除属性来包含或排除任何项。

### 控制器

这个文件夹包含了你所有的控制器文件。控制器负责处理请求、传递模型和生成视图。

### 模型

所有代表领域数据的类都将存在于这个文件夹中。

### 视图

视图是包含前端组件的文件，并展示给应用程序的最终用户。这个文件夹包含了你所有的 **Razor** 视图文件。

### 迁移

任何数据库相关的迁移都将在这个文件夹中可用。数据库迁移是包含通过 **Entity Framework**（一个 ORM 框架）执行的任何数据库更改历史的 C# 文件。这将在稍后的章节中详细解释。

### `wwwroot` 文件夹

这个文件夹充当根文件夹，它是放置所有静态文件（如 CSS 和 JavaScript 文件）的理想容器。所有放置在 `wwwroot` 文件夹中的文件都可以直接通过路径访问，而无需通过控制器。

### 其他文件

`appsettings.json` 文件是配置文件，你可以在这里配置应用程序级别的设置。**Bower**、**npm**（**Node 包管理器**）和 **gulpfile.js** 是 ASP.NET 5 应用程序支持的客户端技术。

# 概述

在本章中，你学习了 Visual Studio 的功能。提供了安装 Visual Studio Community 版本的逐步说明，该版本免费提供给个人开发者。我们还讨论了 ASP.NET 5 应用程序的新项目结构以及与之前版本的差异。

在下一章中，我们将讨论控制器及其角色和功能。我们还将构建一个控制器及其相关操作方法，并查看它们是如何工作的。
