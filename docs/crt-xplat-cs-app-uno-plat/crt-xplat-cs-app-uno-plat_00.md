# 前言

开发人员越来越多地被要求构建在多个操作系统和浏览器上运行的本机应用程序。过去，这意味着学习新技术并制作多个应用程序副本。但是 Uno 平台允许您使用已经熟悉的用于构建 Windows 应用程序的工具、语言和 API 来开发也可以在其他平台上运行的应用程序。本书将帮助您创建面向客户以及业务线的应用程序，这些应用程序可以在您选择的设备、浏览器或操作系统上使用。

这本实用指南使开发人员能够利用他们的 C#和 XAML 知识，使用 Uno 平台编写跨平台应用程序。本书充满了技巧和实际示例，将帮助您构建常见场景的应用程序。您将首先通过逐步解释基本概念来了解 Uno 平台，然后开始为不同的业务线创建跨平台应用程序。在本书中，您将使用示例来教您如何结合您现有的知识来管理常见的开发环境并实现经常需要的功能。

通过本 Uno 平台开发书的学习，您将学会如何使用 Uno 平台编写自己的跨平台应用程序，并使用其他工具和库来加快应用程序开发过程。

# 本书适合的读者

本书适用于熟悉 Windows 应用程序开发并希望利用其现有技能构建跨平台应用程序的开发人员。要开始阅读本书，需要具备 C#和 XAML 的基本知识。任何具有使用 WPF、UWP 或 WinUI 进行应用程序开发的基本经验的人都可以学会如何使用 Uno 平台创建跨平台应用程序。

# 本书涵盖内容

第一章《介绍 Uno 平台》介绍了 Uno 平台，解释了它的设计目的以及何时使用它。之后，本章将介绍如何设置开发机器并安装必要的工具。

第二章《编写您的第一个 Uno 平台应用程序》介绍了创建您的第一个 Uno 平台应用程序，并涵盖了应用程序的结构。通过本章结束时，您将已经编写了一个可以在不同平台上运行并根据应用程序运行的操作系统显示内容的小型 Uno 平台应用程序。

第三章《使用表单和数据》将带您开发一个以数据为重点的虚构公司 UnoBookRail 的业务线应用程序。本章涵盖了显示数据，对表单进行输入验证以及将数据导出为 PDF。

第四章《使您的应用程序移动化》介绍了使用 Uno 平台开发移动应用程序。除此之外，本章还涵盖了在具有不稳定互联网连接的设备上使用远程数据，根据应用程序运行的平台对应用程序进行样式设置，以及使用设备功能，如相机。

第五章《使您的应用程序准备好迎接现实世界》涵盖了编写面向外部客户的移动应用程序。作为其中的一部分，它涵盖了在设备上本地持久化数据，本地化您的应用程序，并使用 Uno 平台编写可访问的应用程序。

第六章《在图表和自定义 2D 图形中显示数据》探讨了在 Uno 平台应用程序中显示图形和图表。本章涵盖了使用诸如 SyncFusion 之类的库以及使用 SkiaSharp 创建自定义图形。最后，本章介绍了编写响应屏幕尺寸变化的用户界面。

*第七章*，*测试您的应用程序*，向您介绍了使用 Uno.UITest 进行 UI 测试。此外，本章还涵盖了使用 WinAppDriver 编写自动化 UI 测试，为应用程序的 Windows 10 版本编写单元测试，以及测试应用程序的可访问性。

*第八章*，*部署您的应用程序并进一步*，将指导您将 Xamarin.Forms 应用程序带到 Uno 平台上，并将 WASM Uno 平台应用程序部署到 Azure。之后，本章将介绍部署 Uno 平台应用程序并加入 Uno 平台社区。

# 充分利用本书

在本书中，我们将使用 Windows 10 上的 Visual Studio 2019 和.NET CLI 来开发 Uno 平台应用程序。我们将介绍安装必要的扩展和 CLI 工具；但是，安装 Visual Studio 和.NET CLI 将不在范围之内。要安装所需的软件，您需要一个正常的互联网连接。

![](img/B17132_Preface_Table_1.jpg)

**如果您使用本书的数字版本，我们建议您自己输入代码或从书的 GitHub 存储库中访问代码（下一节中提供了链接）。这样做将帮助您避免与复制和粘贴代码相关的任何潜在错误。**

# 下载示例代码文件

您可以从 GitHub 上的[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform)下载本书的示例代码文件。如果代码有更新，将在 GitHub 存储库中更新。

我们还提供了来自我们丰富书籍和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。快来看看吧！

# 代码实战

本书的“代码实战”视频可在[`bit.ly/3yHTfYL`](https://bit.ly/3yHTfYL)上观看

# 下载彩色图像

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图和图表的彩色图像。您可以在此处下载：[`static.packt-cdn.com/downloads/9781801078498_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781801078498_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`文本中的代码`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“在`UnoAutomatedTestsApp`文件夹内，创建一个名为`UnoAutomatedTestsApp.UITests`的文件夹。”

代码块设置如下：

```cs
private void ChangeTextButton_Click(object sender,
                                    RoutedEventArgs e)
{
    helloTextBlock.Text = "Hello from code behind!";
}
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```cs
<skia:SKXamlCanvas 
xmlns:skia="using:SkiaSharp.Views.UWP" 
PaintSurface="OnPaintSurface" />
```

任何命令行输入或输出都以以下方式编写：

```cs
dotnet new unoapp -o MyApp
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词以**粗体**显示。这是一个例子：“单击菜单栏中的**View**，然后单击**Test Explorer**，打开**Test Explorer**。”

提示或重要说明

以这种方式出现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请发送电子邮件至[customercare@packtpub.com](https://customercare@packtpub.com)，并在邮件主题中提及书名。

**勘误**：尽管我们已经非常注意确保内容的准确性，但错误是难免的。如果您在本书中发现错误，我们将不胜感激地接受您的报告。请访问[www.packtpub.com/support/errata](https://www.packtpub.com/support/errata)并填写表格。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，请提供给我们位置地址或网站名称，我们将不胜感激。请通过链接[ copyright@packt.com ](https://copyright@packt.com)与我们联系。

**如果您有兴趣成为作者**：如果您在某个专题上有专业知识，并且有兴趣撰写或为一本书做出贡献，请访问[authors.packtpub.com](https://authors.packtpub.com)。

# 分享您的想法

阅读完《使用 Uno 平台创建跨平台 C#应用程序》后，我们很想听听您的想法！请[点击这里直接进入亚马逊评论页面](https://packt.link/r/1801078491)并分享您的反馈。

您的评论对我们和技术社区至关重要，将帮助我们确保我们提供的内容质量卓越。
