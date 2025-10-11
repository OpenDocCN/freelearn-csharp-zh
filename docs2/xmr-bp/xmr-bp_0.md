# 前言

在我作为移动开发者的旅程中，我与许多不同的开发范式和技术合作过。我在所有移动平台上使用 Java、objective-C、Swift（2 和 3）和 C#构建了移动应用程序。我甚至为我的移动应用程序构建了整个服务器。

我站在这里并不是为了炫耀，也不是为了说我是专家。但我确实相信，我遇到了很多问题，并构建了解决方案，这些方案很多移动开发者都会需要。

我最新的工作是在 Xamarin 中使用原生和 Xamarin.Forms 构建跨平台解决方案。我花费了大量时间来缩小我认为构建任何跨平台移动应用程序的最佳方法。构建良好的架构、结构和流畅的用户体验，同时尽可能多地共享代码。

享受阅读。

# 本书涵盖的内容

第一章, *构建相册应用程序*，通过构建一个从本地相册文件读取并显示到 UITableView 和 ListView 中的 iOS 和 Android 应用程序，为你提供了一个使用 Xamarin 进行原生开发的教程。

第二章, *构建语音播报应用程序*，通过构建一个将使用平台语音服务来朗读文本字段中输入的文本的 iOS、Android 和 Windows Phone 应用程序，为你提供了一个 Xamarin.Forms 开发的教程。

第三章, *构建 GPS 定位器应用程序*，展示了如何构建一个集成了原生 GPS 位置服务和 Google Maps API 的 Xamarin.Forms 应用程序。我们将涵盖更多关于 IoC 容器、Xamarin.Forms.Maps 库以及 C#异步和后台任务的技术。

第四章, *构建音频播放器应用程序*，在本章中，我们将集成 iOS 中的 AVFramework 和 Android 中的 MediaPlayer 框架来处理音频文件的原生音频功能。

第五章, *构建股票列表应用程序*，在本章中，我们将通过使用 CustomRenderers、Styles 和 ControlTemplates 来详细说明我们的 XAML 界面。我们还构建了一个简单的 Web 服务，并为我们的移动应用程序设置了一个 JSON 源。

第六章，*构建聊天应用程序*，在本章中，我们的用户界面将远离 MVVM 设计，并遵循一个新的范式，称为 MVP（模型-视图-表示者）。我们进一步深入后端，设置 SignalR 中心客户端以模拟聊天服务，当消息可用时，数据将即时在服务器和客户端之间发送。另一个关键的关注点是项目架构，花费时间将项目划分为模块，并创建一个分层结构，以最大化跨不同平台的代码共享。

第七章，*构建文件存储应用程序*，在本章中，我们将使用 Xamarin.Forms 进行更多开发。我们探讨了行为及其与用户界面的使用。我们还使用 Layout <View>框架构建了一个自定义布局，并构建了我们的第一个 SQLite 数据库以存储文本文件。

第八章，*构建相机应用程序*，我们的最后一章将介绍效果和触发器。我们学习如何将它们应用于用户界面，并使用样式使用它们。我们还为原生平台相机构建了多个复杂的 CustomRenderers，着色图像并接收触摸事件。

# 您需要这本书的内容

**Xamarin Studio**

要安装 Xamarin Studio 的副本，请访问以下链接：

[`www.xamarin.com/download`](https://www.xamarin.com/download)

**构建 Windows Phone 应用程序**

为了构建 Windows Phone 应用程序，您需要一个安装了 Windows、Microsoft Visual Studio 和 Universal Windows Platform SDK 的计算机。

**运行解决方案**

您还需要一个 iOS、android 和 windows phone 设备进行测试。如果您没有设备，您将不得不为每个平台安装模拟器。

**iOS**

模拟器可以通过 XCode 安装。如果您还没有安装 XCode，您将需要安装一个新的副本。

**Android**

请从以下链接安装**Geny Motion**的副本：

[`www.genymotion.com/`](https://www.genymotion.com/)

**Windows Phone**

UWP SDK 附带 Microsoft Visual Studio 的模拟器。

# 这本书是为谁而写的

如果您是一位希望为不同平台创建有趣且功能齐全的应用程序的移动开发者，那么这本书是您的理想解决方案。假设您对 Xamarin 和 C#编程有基本了解。

# 习惯用法

在这本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示："是的，这是我们的`AppDelegate`文件；注意末尾的`.cs`。"

代码块设置如下：

```cs
 private void handleAssetsLoaded (object sender, EventArgs e) 
        { 
            _source.UpdateGalleryItems (_imageHandler.CreateGalleryItems()); 
            _tableView.ReloadData (); 
        }
```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“要这样做，我们只需选择**文件** | **新建** | **解决方案**并选择一个**iOS 单视图应用**。”

### 注意

警告或重要注意事项以这种方式出现在框中。

### 小贴士

小技巧和技巧以这种方式出现。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。要发送一般反馈，请简单地发送电子邮件至 feedback@packtpub.com，并在邮件主题中提及书的标题。如果您在某个主题上有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您的账户中下载此书的示例代码文件。[`www.packtpub.com`](http://www.packtpub.com)。如果您在其他地方购买了此书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**标签上。

1.  点击**代码下载与勘误表**。

1.  在**搜索**框中输入书的名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书籍的来源。

1.  点击**代码下载**。

文件下载完成后，请确保您使用最新版本的以下软件解压缩或提取文件夹：

+   Windows 上的 WinRAR / 7-Zip

+   Mac 上的 Zipeg / iZip / UnRarX

+   Linux 上的 7-Zip / PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Xamarin-Blueprints`](https://github.com/PacktPublishing/CHANGE THIS)。我们还有其他来自我们丰富图书和视频目录的代码包可供在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

## 勘误表

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 盗版

在互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现我们作品的任何非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 copyright@packtpub.com 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们提供有价值内容的能力方面的帮助。

## 问题

如果您对本书的任何方面有问题，您可以通过 questions@packtpub.com 联系我们，我们将尽力解决问题。
