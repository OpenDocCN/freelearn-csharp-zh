# 前言

软件开发者日常处理源代码。无论他们工作的技术或编程语言是什么，他们都必须在代码上执行一系列常规任务：

+   将源代码编译成特定运行时的二进制格式

+   分析源代码以识别源代码中的问题

+   编辑源代码以修复问题或重构以提高维护性、可理解性、性能、安全性等

+   导航源代码以搜索模式、引用、定义和关系

+   调试源代码以观察和修复功能、性能、安全性等方面的运行时行为

+   可视化源组件集合（项目）、它们的属性、配置等

.NET 编译器平台（代号*Roslyn*）是.NET 编程语言 C#和 Visual Basic 的平台，旨在构建工具和扩展以执行这些常规编程任务。值得注意的是，这个平台在 Microsoft 的.NET 编译器和.NET 开发 Visual Studio IDE 之间是共享的。

Roslyn 被划分为多个编程层，每一层都公开 API 以编写定制的工具和扩展：

+   **代码分析层**：源代码的核心语法和语义表示层。C#和 Visual Basic 编译器（*csc.exe*和*vbc.exe*）都是基于这一层编写的。

+   **工作空间层**：收集一组逻辑上相关源文件的项目和解决方案层。这些与任何特定宿主无关，例如 Visual Studio。

+   **功能层**：在 CodeAnalysis 和 Workspaces API 之上构建的一组 IDE 功能，如代码修复、重构、IntelliSense、自动完成、查找引用和导航到定义等。这些功能与任何特定宿主无关，例如 Visual Studio。

+   **Visual Studio 层**：将所有编译器和 IDE 功能汇集和激活的 Visual Studio 工作空间和项目系统。

Roslyn 本质上是一堆服务，它遵循两个核心设计原则：**可扩展性**（用于上层和第三方插件）和**可维护性**（它在这些层之间有良好文档和支持的公共 API）。外部开发者或第三方可以在这些服务之上做很多酷的事情：

+   为任何特定编程层编写自己的工具以完成之前提到的任何编程任务。

+   为特定层编写简单的插件（例如诊断分析器、代码修复和重构、自动完成和 IntelliSense 提供者）。

+   在任何特定层执行高级场景，例如实现自己的编译器、工作空间、IDE 或项目系统，以及整个堆栈的其他功能会自动激活。

# 本书涵盖的内容

第一章，*编写诊断分析器*，使开发者能够为 C#编译器和 Visual Studio IDE 编写诊断分析器扩展，以分析源代码并报告警告和错误。最终用户将在从命令行或 Visual Studio 构建项目时看到这些诊断，并在 Visual Studio IDE 中编辑源代码时实时看到它们。

第二章，*在.NET 项目中使用诊断分析器*，使 C#社区的开发者能够利用第三方 Roslyn 诊断分析器为其 C#项目服务。您将学习如何在 Visual Studio 中搜索、安装、查看和配置诊断分析器。

第三章，*编写 IDE 代码修复、重构和 IntelliSense 完成提供者*，使开发者能够为 Visual Studio IDE 编写代码修复和代码重构扩展，以编辑 C#源代码并修复编译器/分析器诊断以及重构源代码。它还使开发者能够为 Visual Studio IDE 中的 C# IntelliSense 编写完成提供者扩展，以增强代码编辑体验。

第四章，*提高 C#代码库的代码维护性*，使 C#社区的开发者能够通过使用集成到 Visual Studio IDE 中的分析器和代码修复，以及一些流行的第三方实现，来提高其源代码的代码维护性和可读性。

第五章，*在 C#代码中捕捉安全漏洞和性能问题*，使 C#社区的开发者能够通过使用流行的第三方分析器，如 PUMA 扫描分析器和 FxCop 分析器，来捕捉其 C#代码库中的安全和性能问题。

第六章，*Visual Studio Enterprise 中的实时单元测试*，使开发者能够使用 Visual Studio 2017 Enterprise 版中基于 Roslyn 的新功能，该功能允许在后台执行智能实时单元测试（LUT）。LUT 在您编辑代码时自动在后台运行受影响的单元测试，并在编辑器中实时可视化结果和代码覆盖率。

第七章，*C#交互式和脚本编程*，使开发者能够在 Visual Studio 中使用 C#交互式和脚本功能。C#脚本是一种使用 REPL（读取-评估-打印循环）快速测试 C#和.NET 片段的工具，无需创建多个单元测试或控制台项目。

第八章，*向 Roslyn C#编译器开源代码贡献简单功能*，使开发者能够向开源的 Roslyn C#编译器添加新功能。你将学习如何实现新的 C#编译器错误，为它们添加单元测试，然后将你的代码更改的 pull request 发送到 Roslyn 仓库，以便将其纳入 C#编译器的下一个版本。

第九章，*设计和实现新的 C#语言特性*，使开发者能够在开源的 Roslyn C#编译器中设计新的 C#语言特性，并实现该特性的各种编译阶段。你将学习编译设计和实现的以下方面：语言设计、解析、语义分析和绑定，以及代码生成，并伴有合适的代码示例。

第十章，*基于 Roslyn API 的命令行工具*，使开发者能够使用 Roslyn 编译器和 Workspaces API 编写命令行工具来分析和/或编辑 C#代码。

# 你需要为本书准备

你需要 Visual Studio 2017 Community/Enterprise 版本来执行本书中的食谱。你可以从[`www.visualstudio.com/downloads/`](https://www.visualstudio.com/downloads/)安装 Visual Studio 2017。此外，有些章节需要你从[`desktop.github.com/`](https://desktop.github.com/)安装 GitHub 桌面工具。

# 本书面向对象

.NET 开发者和架构师，他们有兴趣充分利用基于 Roslyn 的扩展和工具来改进开发流程，会发现这本书很有用。Roslyn 贡献者，即生产者和 C#社区开发者，也会发现这本书很有用。

# 习惯用法

在本书中，你将找到多种文本样式，以区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名和用户输入如下所示：“此外，`.editorconfig`文件可以与源文件一起提交到仓库，以便为每个为仓库做出贡献的用户强制执行规则。”

代码块应如下设置：

```cs
 private void Method_PreferBraces(bool flag)
  {
    if (flag)
    {
      Console.WriteLine(flag);
    }
  }    

```

任何命令行输入或输出都应如下编写：

```cs
msbuild ClassLibrary.csproj /v:m

```

新术语和重要词汇以粗体显示。屏幕上看到的词汇，例如在菜单或对话框中，在文本中如下所示：“启动 Visual Studio，点击文件 | 新建 | 项目...，创建一个新的 C#类库项目，并用`ClassLibrary/Class1.cs`中的代码替换`Class1.cs`中的代码。”

警告或重要注意事项以如下框的形式出现。

小贴士和技巧如下所示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢什么或不喜欢什么。读者的反馈对我们来说很重要，因为它帮助我们开发出你真正能从中获得最大收益的标题。

要发送给我们一般性的反馈，只需发送电子邮件至 `feedback@packtpub.com`，并在邮件主题中提及书的标题。

如果你在一个领域有专业知识，并且你对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 书籍的骄傲拥有者，我们有许多事情可以帮助你从购买中获得最大收益。

# 下载示例代码

你可以从你的账户中下载本书的示例代码文件 [`www.packtpub.com`](http://www.packtpub.com)。如果你在其他地方购买了这本书，你可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的“支持”选项卡上。

1.  点击“代码下载 & 错误清单”。

1.  在搜索框中输入书的名称。

1.  选择你想要下载代码文件的书籍。

1.  从下拉菜单中选择你购买这本书的地方。

1.  点击“代码下载”。

文件下载完成后，请确保使用最新版本解压缩或提取文件夹：

+   Windows 版本的 WinRAR / 7-Zip

+   Mac 版本的 Zipeg / iZip / UnRarX

+   Linux 版本的 7-Zip / PeaZip

本书代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Roslyn-Cookbook`](https://github.com/PacktPublishing/Roslyn-Cookbook)。我们还有其他来自我们丰富图书和视频目录的代码包，可在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 找到。去看看吧！

# 下载本书的颜色图片

我们还为你提供了一个包含本书中使用的截图/图表颜色图片的 PDF 文件。这些颜色图片将帮助你更好地理解输出的变化。你可以从 [`www.packtpub.com/sites/default/files/downloads/RoslynCookbook_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/RoslynCookbook_ColorImages.pdf) 下载此文件。

# 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在勘误部分下。

# 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的非法复制我们的作品，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`copyright@packtpub.com`与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

# 问题

如果您对本书的任何方面有问题，可以通过`questions@packtpub.com`与我们联系，我们将尽力解决问题。
