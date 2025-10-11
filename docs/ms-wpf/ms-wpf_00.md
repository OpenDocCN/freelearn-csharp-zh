# 前言

微软 **Windows Presentation Foundation** (**WPF**) 为开发者提供了多个库和 API，用于创建引人入胜的用户体验。本书包含了一系列从简单到复杂的示例，展示了如何使用 WPF 开发适用于 Windows 桌面的企业级应用程序。

《精通 Windows Presentation Foundation》的第二版更新版首先介绍了使用 WPF 的**模型-视图-视图模型**（**MVVM**）软件架构模式的好处，然后指导你调试你的 WPF 应用程序。接下来，本书将带你了解应用程序架构和为你的应用程序构建基础层。随着你的进步，你将掌握数据绑定，探索各种内置的 WPF 控件，并根据你的需求进行定制。

本书充满了引人入胜且实用的示例，阐述了帮助你将 WPF 技能提升到新水平的概念。你将学习如何发现 MVVM 以及它是如何帮助使用 WPF 进行开发的，实现你自己的自定义应用程序框架，了解如何适应内置控件，熟练使用动画，实现响应式数据验证，创建视觉上吸引人的用户界面，并提高应用程序的性能，以及学习如何部署你的应用程序。

此外，你还将发现使用 MVVM 软件架构模式与 WPF 一起工作的更智能的工作方式，学习如何创建自己的轻量级应用程序框架来构建未来的应用程序，了解数据绑定，并学习如何在应用程序中使用它。

当内置功能不足以满足你的需求时，你将学习如何创建自定义控件。你还将了解如何通过实用的动画、令人惊叹的视觉效果和响应式数据验证来增强你的应用程序。为了确保你的应用程序不仅具有交互性，而且效率高，你将专注于提高应用程序性能，并最终发现部署应用程序的不同方法。到本书结束时，你将熟练使用 WPF 开发既高效又健壮的用户界面。

# 本书面向的对象

这本 Windows 书籍适合对 Windows Presentation Foundation 有基础到中级知识的开发者，以及那些希望仅仅提升自己 WPF 技能的人。如果你想要学习更多关于应用程序架构以及以视觉吸引力的方式设计用户界面，你会发现这本书很有用。

# 本书涵盖的内容

第一章，《使用 WPF 的更智能的工作方式》，介绍了 **模型-视图-视图模型** (**MVVM**) 软件架构模式和它在 WPF 中使用的好处。

第二章，《调试 WPF 应用程序》，提供了关于调试 WPF 应用程序的各种方法的必要技巧，确保能够解决可能出现的任何问题。

第三章，*编写自定义应用程序框架*，介绍了不可或缺的应用程序框架概念，并提供了早期示例，这些示例将在本书的后续内容中构建。您将拥有一个完全功能化的框架，用于构建您的应用程序。

第四章，*精通数据绑定*，揭示了数据绑定的神秘面纱，并清晰地展示了如何在实际应用中使用它。大量的示例将帮助您了解在任何特定情况下应使用哪种绑定语法，并使您有信心您的绑定将按预期工作。

第五章，*使用适合工作的控件*，解释了在特定情况下应使用哪些控件，并描述了在需要时如何修改它们。它清楚地概述了如何自定义现有控件，以及在需要时如何创建自定义控件。

第六章，*适配内置控件*，专注于通过扩展更改控件行为。它首先调查了内置控件如何通过派生类使我们能够操作它们，然后描述了我们可以如何在我们自己的控件中启用这种技术。它以一个扩展示例结束，展示了如何通过控件扩展来满足我们的需求。

第七章，*精通实用动画*，解释了 WPF 动画的方方面面，详细介绍了这一不太为人所知的功能。它以一系列实用动画的想法结束，并继续构建自定义应用程序框架。

第八章，*创建视觉吸引人的用户界面*，提供了充分利用 WPF 视觉效果的技巧，同时保持实用性，并提供了使应用程序脱颖而出的小贴士。

第九章，*实现响应式数据验证*，提出了多种数据验证方法以适应各种情况，并继续构建自定义应用程序框架。它涵盖了完整、部分、即时和延迟验证，以及显示验证错误的各种不同方式。

第十章，*打造卓越的用户体验*，提供了创建具有卓越用户体验的应用程序的技巧。这里介绍的概念，如异步数据访问和确保最终用户充分了解信息，将显著改善现有的自定义应用程序框架。

第十一章，*提高应用程序性能*，列出了从冻结资源到实现虚拟化的多种方法来提高 WPF 应用程序的性能。如果您遵循这些技巧和窍门，您可以放心，您的 WPF 应用程序将尽可能优化地运行。

第十二章，*部署您的杰作应用程序*，涵盖了所有专业应用程序的最终要求——部署。它包括使用 Windows Installer 软件的较老方法，以及使用更常见和更新的 ClickOnce 功能的方法。

第十三章，*接下来是什么？*，总结了您从本书中学到的内容，并建议您可以使用许多新的技能做什么。它为您提供了进一步扩展应用程序框架的想法。

# 要充分利用本书

本书包含大量代码示例，完整源代码可在 GitHub 上下载。为了运行代码，您需要以下先决条件：

+   Visual Studio 2019

+   必须安装.NET Framework 的 4.8 版本

如果您还没有这些，它们都可以免费下载和安装：

+   您可以从[`visualstudio.microsoft.com/downloads/`](https://visualstudio.microsoft.com/downloads/)下载 Visual Studio Community 2019。

+   您可以从[`dotnet.microsoft.com/download/dotnet-framework`](https://dotnet.microsoft.com/download/dotnet-framework)下载.NET Framework 的最新版本。

伴随本书的完整源代码可以从[`github.com/PacktPublishing/Mastering-Windows-Presentation-Foundation-Second-Edition`](https://github.com/PacktPublishing/Mastering-Windows-Presentation-Foundation-Second-Edition)下载。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](https://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)上登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载”。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保您使用最新版本解压缩或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Mastering-Windows-Presentation-Foundation-Second-Edition`](https://github.com/PacktPublishing/Mastering-Windows-Presentation-Foundation-Second-Edition)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的书籍和视频目录的代码包可供在 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)** 上获取。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载： [`static.packt-cdn.com/downloads/9781838643416_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781838643416_ColorImages.pdf)

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“您可以在`DragDropProperties`类中使用`BaseDragDropManager`类进行声明。”

代码块设置如下：

```cs
public string Name
{
  get { return name; }
  set
  {
    if (name != value)
    {
      name = value;
      NotifyPropertyChanged("Name");
    }
  }
}
```

任何命令行输入或输出都按以下方式编写：

```cs
System.Windows.Data Error: 17 : Cannot get 'Item[]' value (type  'ValidationError') from '(Validation.Errors)' (type  'ReadOnlyObservableCollection`1').  BindingExpression:Path=(Validation.Errors)[0].ErrorContent;  DataItem='TextBox' (Name=''); target element is 'TextBox' (Name='');  target property is 'ToolTip' (type 'Object')  ArgumentOutOfRangeException:'System.ArgumentOutOfRangeException:  Specified argument was out of the range of valid values.
```

**粗体**：表示新术语、重要词汇或屏幕上看到的词汇。例如，菜单或对话框中的文字会以这种方式显示。以下是一个示例：“取消按钮已在第二行和第二列中声明。”

警告或重要提示看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并电子邮件我们至 `customercare@packtpub.com`。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问 [www.packtpub.com/support/errata](https://www.packtpub.com/support/errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过链接至材料与我们联系至 `copyright@packt.com`。

**如果您有兴趣成为作者**：如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用过这本书，为何不在您购买它的网站上留下评论？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 公司可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问 [packt.com](http://www.packt.com/)。
