# 前言

欢迎来到*Unity 2018 增强现实项目*。本书是为那些希望走出舒适区，跃入学习增强现实（**AR**）及其如何在 Unity 中创建项目的人设计的。我们将重点关注 Android、iOS 和 Windows 设备，并深入理解开始使用这一令人惊叹且不断发展的技术的根本。

# 本书面向的对象

本书面向任何希望扩展其 Unity 现有知识的人。他们应该有一些 C#和 Unity 的经验，并且愿意接触一点 C++、Java 和 Objective-C 语言代码。

# 本书涵盖的内容

第一章，*什么是 AR 以及如何设置*，解释了安装各种 SDK 和包以启用 AR 的过程，以及使用 AR 构建 Hello World 示例。

第二章，*GIS 基础 - 地图的力量*，探讨了 GIS 的历史、GIS 在应用和游戏中的影响，以及 GIS 在教育中的应用。

第三章，*审查 - 各种传感器数据和插件*，探讨了如何用 C#为 Unity 编写插件，如何用 C++为 Unity 编写插件，如何用 Objective-C 为 Unity 编写插件，以及如何用 Java 为 Unity 编写插件。

第四章，*花团锦簇的文风之声*，详细介绍了设计应用程序的步骤，探讨了项目的概念化，并探索了如何基于声音感知创建 AR 应用程序。

第五章，*图片拼图 - AR 体验*，帮助您设计教育应用程序，学习使用 Vuforia，并使用 Vuforia 开发一个教育 AR 应用程序。

第六章，*健身乐趣 - 旅游业和随机漫步*，介绍了 Mapbox，如何将 Mapbox 集成到 Unity 中，以及构建一个随机漫步到位置的 App 原型。

第七章，*拍下它！为图片添加滤镜*，帮助您了解 OpenCV，将 OpenCV 集成到 Unity 中，从源代码构建 OpenCV，并使用 OpenCV 构建一个面部检测应用程序原型。

第八章，*通往 HoloLens 及更远之处*，为您揭示了增强现实（AR）与**混合现实**（**MR**）之间的区别，教授您如何使用 HoloLens 模拟器，并指导您使用 HoloLens 模拟器构建一个基本的 MR 原型。

# 为了充分利用本书

要充分利用本书，您应该对 Unity 编辑器、UI 和构建过程有所了解。此外，强烈建议您具备高于初学者水平的 C#技能，因为本书不会介绍如何编写 C#代码。最后，建议您至少浏览过其他编程语言，如 Swift、Objective-C、C、C++和 Java，并能一眼看出本书中遇到的代码的要点。

仅需具备 Unity 游戏引擎和 C#的基本知识，因为它们是本书的主要关注点。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择 SUPPORT 标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   WinRAR/7-Zip（Windows）

+   Zipeg/iZip/UnRarX（Mac）

+   7-Zip/PeaZip（Linux）

该书的代码包也托管在 GitHub 上，网址为**[`github.com/PacktPublishing/Unity-2018-Augmented-Reality-Projects`](https://github.com/PacktPublishing/Unity-2018-Augmented-Reality-Projects)**。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的图书和视频目录的代码包可供选择，网址为**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。查看它们吧！

# 下载彩色图片

我们还提供了一份包含本书中使用的截图/图表的彩色图片的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/Unity2018AugmentedRealityProjects_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/Unity2018AugmentedRealityProjects_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“创建一个全新的 Unity 项目。我将称它为`Snap`。”

代码块设置如下：

```cs
struct Circle
{
Circle(int x, int y, int radius) : X(x), Y(y), Radius(radius) {}
int X, Y, Radius;
};
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```cs
extern "C" void __declspec(dllexport) __stdcall  Close()
{
_capture.release();
}
```

**粗体**: 表示新术语、重要词汇或屏幕上看到的词汇。例如，菜单或对话框中的文字会以这种方式显示。以下是一个例子：“从管理面板中选择系统信息。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请通过 `feedback@packtpub.com` 发送电子邮件，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过 `questions@packtpub.com` 发送电子邮件给我们。

**勘误表**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，我们将不胜感激，如果您能提供位置地址或网站名称，我们将不胜感激。请通过 `copyright@packtpub.com` 联系我们，并附上材料的链接。

**如果您想成为一名作者**：如果您在某个领域有专业知识，并且对撰写或为书籍做出贡献感兴趣，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了这本书，为什么不在此处购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/)。
