# 前言

*Xamarin.Forms 项目*是一本实践性的书，您将从头开始创建七个应用程序。您将获得设置环境所需的基本技能，我们将解释 Xamarin 是什么，然后过渡到 Xamarin.Forms，真正利用真正的本地跨平台代码。

阅读本书后，您将真正了解创建一个可以持续发展并经得起时间考验的应用程序需要什么。

我们将涵盖动画，增强现实，消费 REST 接口，使用 SignalR 进行实时聊天，以及使用设备 GPS 进行位置跟踪等内容。还有机器学习和必备的待办事项清单。

愉快的编码！

# 这本书适合谁

这本书适合熟悉 C#和 Visual Studio 的开发人员。您不必是专业程序员，但应该具备使用.NET 和 C#进行面向对象编程的基本知识。典型的读者可能是想探索如何使用 Xamarin，特别是 Xamarin.Forms，来使用.NET 和 C#创建应用程序的人。

不需要预先了解 Xamarin 的知识，但如果您曾在传统的 Xamarin 中工作并希望迈出向 Xamarin.Forms 的步伐，那将是一个很大的帮助。

# 本书涵盖内容

第一章，*Xamarin 简介*，解释了 Xamarin 和 Xamarin.Forms 的基本概念。它帮助您了解如何创建真正的跨平台应用程序的构建模块。这是本书唯一的理论章节，它将帮助您入门并设置开发环境。

第二章，*构建我们的第一个 Xamarin.Forms 应用程序*，指导您了解 Model-View-ViewModel 的概念，并解释如何使用控制反转简化视图和视图模型的创建。我们将创建一个支持导航、过滤和向列表添加待办事项的待办事项应用程序，并渲染一个利用 Xamarin.Forms 强大数据绑定机制的用户界面。

第三章，*使用动画创建丰富用户体验的匹配应用程序*，让您深入了解如何使用动画和内容放置定义更丰富的用户界面。它还涵盖了自定义控件的概念，将用户界面封装成自包含的组件。

第四章，*使用 GPS 和地图构建位置跟踪应用程序*，涉及使用设备 GPS 的地理位置数据以及如何在地图上绘制这些数据的图层。它还解释了如何使用后台服务长时间跟踪位置，以创建您花费时间的热图。

第五章，*为多种形式因素构建天气应用程序*，涉及消费第三方 REST 接口，并以用户友好的方式显示数据。我们将连接到天气服务，获取当前位置的预报，并在列表中显示结果。

第六章，*使用 Azure 服务为聊天应用程序设置后端*，是两部分章节中的第一部分，我们将设置一个聊天应用程序。本章解释了如何使用 Azure 服务创建一个后端，通过 SignalR 公开功能，以建立应用程序之间的实时通信渠道。

第七章，*构建实时聊天应用程序*，延续了前一章的内容，涵盖了应用程序的前端，即 Xamarin.Forms 应用程序，它连接到中继消息的后端，实现用户之间的消息传递。本章重点介绍了如何在客户端设置 SignalR，并解释了如何创建一个服务模型，通过消息和事件抽象化这种通信。

第八章，*创建增强现实游戏*，将两种不同的 AR API 绑定到一个 UrhoSharp 解决方案中。Android 使用 ARCore 处理增强现实，iOS 使用 ARKit 执行相同的操作。我们将通过自定义渲染器下降到特定于平台的 API，并将结果公开为 Xamarin.Forms 应用程序消耗的通用 API。

第九章，*使用机器学习识别热狗或非热狗*，涵盖了创建一个应用程序，该应用程序使用机器学习来识别图像是否包含热狗。

# 从本书中获得最大收益

我们建议您阅读第一章，以确保您对 Xamarin 的基本概念有所了解。之后，您可以选择任何您喜欢的章节来学习更多。每一章都是独立的，但章节按复杂性排序；您在书中的位置越深，应用程序就越复杂。

这些应用程序适用于实际应用，但某些部分被省略，例如适当的错误处理和分析，因为它们超出了本书的范围。然而，您应该对如何创建应用程序的基本构建块有所了解。

话虽如此，如果您是 C#和.NET 开发人员已经有一段时间了，那么会有所帮助，因为许多概念实际上并不是特定于应用程序，而是一般的良好实践，例如 Model-View-ViewModel 和控制反转。

但最重要的是，这是一本可以帮助您通过专注于最感兴趣的章节来启动 Xamarin.Forms 开发学习曲线的书籍。

# 下载示例代码文件

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Xamarin.Forms-Projects`](https://github.com/PacktPublishing/Xamarin.Forms-Projects)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有其他代码包来自我们丰富的图书和视频目录，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。去看看吧！

# 下载彩色图像

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/9781789537505_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781789537505_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“打开`DescriptionGenerator.cs`文件，并添加一个构造函数，如下面的代码所示。”

代码块设置如下：

```cs
public class DescriptionGenerator
{
  private string[] _adjectives = { "nice", "horrible", "great",
                                   "terribly old", "brand new" };
  private string[] _other = { "picture of grandpa", "car", "photo
                               of a forest", "duck" };
  private static Random random = new Random();
  public string Generate()
{
  var a = _adjectives[random.Next(_adjectives.Count())];
  var b = _other[random.Next(_other.Count())];
  return $"A {a} {b}";
}
}
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```cs
{
    TabLayoutResource = Resource.Layout.Tabbar;
    ToolbarResource = Resource.Layout.Toolbar;

    base.OnCreate(savedInstanceState);

    global::Xamarin.Forms.Forms.Init(this, savedInstanceState);
    Xamarin.Essentials.Platform.Init(this, savedInstanceState);
    LoadApplication(new App());
}
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会以这种形式出现在文本中。这是一个例子：“从管理面板中选择系统信息。”

警告或重要说明会以这种形式出现。

提示和技巧会显示在这样的形式下。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并发送电子邮件至`customercare@packtpub.com`。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误确实会发生。如果您在本书中发现错误，我们将不胜感激。请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书，点击勘误提交表格链接，并输入详细信息。

盗版：如果您在互联网上发现我们作品的任何形式的非法副本，我们将不胜感激，如果您能向我们提供位置地址或网站名称。请通过`copyright@packt.com`与我们联系，并附上材料链接。

如果您有兴趣成为作者：如果您在某个专业领域有专长，并且有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。当您阅读并使用了这本书之后，为什么不在购买它的网站上留下评论呢？潜在的读者可以看到并使用您的客观意见来做出购买决定，我们在 Packt 可以了解您对我们产品的看法，我们的作者也可以看到您对他们书籍的反馈。谢谢！

有关 Packt 的更多信息，请访问[packt.com](http://www.packt.com/)。
