前言

在 1985 年，随着 Windows 1.0 的推出，微软引入了图形设备接口（GDI）和 USER 子系统，以构建基于 Windows 的图形用户界面（GUI）。在 1990 年，OpenGL 出现，用于在 Windows 和非 Windows 系统上创建 2D 和 3D 图形。在 1995 年，微软推出了另一种技术，称为 DirectX，用于创建高性能的 2D/3D 图形。后来，GDI+被引入，以在现有的 GDI 之上添加 alpha 混合和渐变画笔支持。

在 2002 年，微软推出了.NET Framework。与此同时，Windows Forms 也被引入，用于使用 C#和 Visual Basic 语言构建 Windows 的用户界面（UI）。它是建立在 GDI+之上的，因此仍然具有 GDI+和 USER 子系统的限制。

多年来，微软决定引入一种新技术来构建基于 Windows 的应用程序的丰富 UI，这不仅帮助用户（开发者和设计师）摆脱了 GDI/GDI+和 USER 子系统的限制，还帮助他们提高构建基于桌面的应用程序时的生产力。

在 2006 年 11 月，随着.NET 3.0 的推出，Windows Presentation Foundation (WPF) 也被引入，为开发者提供了一个统一的编程模型，用于构建动态、数据驱动的 Windows 桌面应用程序。它附带了一系列功能，用于创建图形子系统，使用各种控件、布局、图形、资源等来渲染丰富的 UI，同时考虑到应用程序和数据的安全性。由于它最初作为.NET Framework 3.0 的一部分发布，因此第一个版本被称为 WPF 3.0。

WPF 是一个分辨率无关的框架，它使用基于 XML 的语言 XAML（发音为 Zammel）的矢量渲染引擎，来创建现代的用户体验，为应用程序编程提供了一个声明性模型。使用它，你可以轻松地自定义控件并为其添加皮肤，以获得应用程序 UI 的更好表示。

由于 WPF 与经典的 Windows Forms 不同，因为它使用 XAML、数据绑定、模板、样式、动画、文档等，最初它并没有得到太多的关注。然而，后来它开始获得越来越多的流行和关注。发布了多个更新版本，以增加更多功能，使其更加健壮和强大。

在这本书中，我们将介绍一系列食谱，展示如何使用 WPF 执行常见任务。从 WPF 基础知识开始，我们将涵盖标准控件、布局、面板、数据绑定、自定义控件、用户控件、样式、模板、触发器和动画，然后进一步探讨资源的使用、MVVM 模式、WCF 服务、调试、多线程和 WPF 互操作性，以确保您正确理解基础。

本书中的示例简单易懂，为您提供学习并掌握构建桌面应用程序所需技能所需的知识。到您阅读完本书时，您将足够熟练，对每个章节都有深入的了解。尽管本书已涵盖大多数重要主题，但总会有一些主题是任何书籍都无法完全涵盖的。您一定会喜欢阅读这本书，因为其中包含大量图形和文本步骤，帮助您在处理 Windows Presentation Foundation 时建立信心。

# 第一章：本书面向对象

本书旨在为相对较新接触 Windows Presentation Foundation (WPF)的开发者或那些已经使用 WPF 一段时间但希望更深入理解其基础和概念以获得实用知识的人编写。假设读者具备 C#和 Visual Studio 的基本知识。

# 本书涵盖内容

第一章，*WPF 基础*，专注于 WPF 的架构、应用程序类型以及 XAML 语法、术语，并解释如何使用 Visual Studio 2017 安装 WPF 工作负载以创建针对 Windows Presentation Foundation 的第一个应用程序。它将涵盖导航机制、各种对话框、在多个窗口之间建立所有权，然后继续创建单实例应用程序。本章还将介绍如何向 WPF 应用程序传递参数以及如何处理在 WPF 中抛出的未处理异常。

第二章，*使用 WPF 标准控件*，为您提供深入了解，帮助您了解 WPF 的各种常见控件。本章将从 TextBlock、Label、TextBox 和 Image 控件开始，然后继续介绍 2D 形状、Tooltip、标准菜单、上下文菜单、单选按钮和复选框控件。本章还将涵盖如何使用进度条、滑块、日历、列表框、组合框、状态栏和工具栏面板。

第三章，*布局和面板*，为您快速介绍标准布局和面板。本章将涵盖如何使用面板创建适当的布局。它还将简要介绍实现拖放功能。

第四章，*使用数据绑定*，讨论了数据绑定的重要概念以及如何在 WPF 中使用它。它还讨论了 CLR 属性、依赖属性、附加属性、转换器和数据操作（如排序、分组和筛选）。逐步方法将指导您熟练掌握所有类型的数据绑定。

第五章，*使用自定义控件和用户控件*，提供了您创建可重用自定义控件和用户控件所需的基本构建块。您还将学习如何使用自定义控件和用户控件的自定义属性和事件来定制控件模板。

第六章，*使用样式、模板和触发器*，深入探讨了控制器的样式和模板，随后介绍了您可以直接从 XAML 执行某些操作或 UI 更改的各种触发器，而不需要使用任何 C#代码。

第七章，*使用资源和 MVVM 模式*，首先演示了使用和管理二进制资源、逻辑资源和静态资源的各种方法。然后，它将继续使用 Model View ViewModel (MVVM)模式，通过在代码后文件中编写更少的代码来构建 WPF 应用程序。MVVM 模式通过一些示例介绍，展示了您如何构建命令绑定。

第八章，*使用动画*，提供了对 WPF 动画功能的全面介绍，并讨论了如何使用各种转换和动画以及将效果应用于动画。

第九章，*使用 WCF 服务*，使您轻松理解 WCF 服务的*ABC*，并解释了如何在 WPF 应用程序中创建、托管和消费它们。

第十章，*调试和线程*，讨论了 WPF 对使用实时视觉树和实时属性浏览器调试 XAML 应用程序 UI 的支持。本章帮助您创建异步操作，以便应用程序 UI 始终保持响应。

第十一章，*与 Win32 和 WinForm 的互操作性*，重点在于理解 WPF 与 Win32 和 Windows Forms 的互操作性。在本章中，您将学习如何将一个技术（WPF/WinForm）的元素托管到另一个技术（WinForm/WPF）中，然后调用 Win32 API 并在 WPF 应用程序中嵌入 ActiveX 控件。

# 要充分利用本书

本书假设读者具备.NET Framework 和 C#（至少 C#版本 3.0，但 C# 7.0 或更高版本更佳）的知识，并且有 Visual Studio 2015 或更高版本（Visual Studio 2017 更佳）的实际使用经验。已假设具备 WPF 和 XAML 的基本知识。

## 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择支持选项卡。

1.  点击代码下载和勘误表。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本解压缩或提取文件夹：

+   Windows 的 WinRAR/7-Zip

+   Mac 的 Zipeg/iZip/UnRarX

+   Linux 的 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Windows-Presentation-Foundation-Development-Cookbook`](https://github.com/PacktPublishing/Windows-Presentation-Foundation-Development-Cookbook)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有来自我们丰富的书籍和视频目录中的其他代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**找到。查看它们吧！

## 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/WindowsPresentationFoundationDevelopmentCookbook_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/WindowsPresentationFoundationDevelopmentCookbook_ColorImages.pdf)。

## 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“表示媒体集成库的包装的 Presentation Core 层是`presentationcore.dll`的一部分，为您提供了包装。”

代码块设置如下：

```cs
     <Button> 
       <Button.Background> 
         <SolidColorBrush Color="Red" /> 
       </Button.Background> 
    </Button> 
```

任何命令行输入或输出都按以下方式编写：

```cs
svcutil.exe http://localhost:59795/Services/EmployeeService.svc?wsdl
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“要构建针对.NET Framework 的 WPF 应用程序，请选择.NET 桌面开发工作负载。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请将电子邮件发送至`feedback@packtpub.com`，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过电子邮件发送至`questions@packtpub.com`。

**勘误表**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，如果您能向我们报告，我们将不胜感激。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**: 如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过发送链接至`copyright@packtpub.com`与我们联系。

**如果您有兴趣成为作者**: 如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

## 评价

请留下您的评价。一旦您阅读并使用了这本书，为何不在购买它的网站上留下评价呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 公司可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问[packtpub.com](https://www.packtpub.com/)。
