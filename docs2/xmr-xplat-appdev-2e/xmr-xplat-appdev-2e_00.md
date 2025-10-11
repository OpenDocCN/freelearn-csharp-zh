# 前言

Xamarin 为开发 iOS 和 Android 应用程序的 C#构建了三个核心产品：Xamarin Studio、Xamarin.iOS 和 Xamarin.Android。Xamarin 让您直接访问每个平台的本地 API，并能够在平台之间共享 C#代码。与 Java 或 Objective-C 相比，使用 Xamarin 和 C#，您可以获得更高的生产力，同时与 HTML 或 JavaScript 解决方案相比，仍能保持出色的性能。

在本书中，我们将开发一个真实世界的示例应用程序，以展示您可以使用 Xamarin 技术做什么，并基于 iOS 和 Android 的核心平台概念进行构建。我们还将涵盖高级主题，如推送通知、检索联系人、使用相机和 GPS 定位。随着 Xamarin 3 的推出，引入了一个名为 Xamarin.Forms 的新框架。我们将介绍 Xamarin.Forms 的基础知识以及如何将其应用于跨平台开发。最后，我们将指导您了解将应用程序提交到 Apple App Store 和 Google Play 所需的内容。

# 本书涵盖的内容

第一章，*设置 Xamarin*，介绍了安装适当的 Xamarin 软件和进行跨平台开发所需的本地 SDK 的过程。

第二章，*嘿，平台！*，引导您在 iOS 和 Android 上创建第一个“Hello World”应用程序，同时也涵盖了每个平台的一些基本概念。

第三章，*iOS 和 Android 之间的代码共享*，提供了可以与 Xamarin 一起使用的代码共享技术和项目设置策略。

第四章，*XamChat – 一个跨平台应用*，介绍了一个我们将全书构建的示例应用程序。在本章中，我们将编写应用程序的所有共享代码，包括单元测试。

第五章，*XamChat for iOS*，涵盖了实现 XamChat 的 iOS 用户界面以及各种 iOS 开发概念的技术。

第六章，*XamChat for Android*，涵盖了实现 XamChat 的 Android 版本的技术，并介绍了 Android 特定的开发概念。

第七章，*在设备上部署和测试*，引导您通过将第一个应用程序部署到设备的痛苦过程。我们还讨论了为什么始终在真实设备上测试您的应用程序很重要。

第八章，*带有推送通知的 Web 服务*，解释了使用 Azure Mobile Services 实现 XamChat 真实后端 Web 服务的技巧。

第九章, *第三方库*，涵盖了使用 Xamarin 的各种第三方库选项，以及您甚至可以利用原生 Java 和 Objective-C 库。

第十章, *联系人、相机和位置*，介绍了 Xamarin.Mobile 库作为跨平台访问用户联系人、相机和 GPS 位置的方法。

第十一章, *Xamarin.Forms*，介绍了 Xamarin 的最新框架，Xamarin.Forms，以及您如何利用它来构建跨平台应用程序。

第十二章, *App Store 提交*，解释了将您的应用程序提交到苹果 App Store 和谷歌 Play 的过程。

# 您需要为本书准备的内容

为了使用本书，您需要一个至少运行 OS X 10.7 Lion 的 Mac 计算机。苹果要求 iOS 应用程序必须在 Mac 上编译，因此 Xamarin 也是如此。您还需要拥有 Xamarin.Android 和 Xamarin.iOS 商业版的许可证。还提供免费 30 天试用版。您还可以尝试免费的 Xamarin 入门版，但某些更高级的示例可能无法使用此版本。您可以通过访问 [`xamarin.com/download`](http://xamarin.com/download) 下载适当的软件。

# 本书面向的对象

本书是为已经熟悉 C# 并想开始使用 Xamarin 进行移动开发的开发者编写的。如果您在 ASP.NET、WPF、WinRT 或 Windows Phone 上工作过，那么使用本书开发原生 iOS 和 Android 应用程序将得心应手。

# 约定

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和推特用户名应如下所示："用户将在 `UITextField` 文本框中输入一些文本，然后点击 `UIButton` 按钮以开始搜索。"

代码块如下所示：

```cs
public override void ViewDidLoad() 
{
  base.ViewDidLoad();
  int count = 0;
  button.TouchUpInside += (sender, e) => 
    label.Text = string.Format("Count: {0}", ++count);
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```cs
public override void ViewDidLoad() 
{
  base.ViewDidLoad();
  int count = 0;
  button.TouchUpInside += (sender, e) => 
    label.Text = string.Format("Count: {0}", ++count);
}
```

任何命令行输入或输出都应如下所示：

```cs
keytool -genkey -v -keystore <filename>.keystore -alias <key-name> -keyalg RSA -keysize 2048 -validity 10000

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中如下所示："点击 **下一步** 按钮将您带到下一屏幕。"

### 注意

警告或重要注意事项如下所示。

### 小贴士

小技巧和技巧看起来像这样。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对本书的看法——您喜欢或不喜欢的地方。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要向我们发送一般反馈，请简单地发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书籍的标题。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)下载您购买的所有 Packt 出版物的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表的颜色图像的 PDF 文件。这些颜色图像将帮助您更好地理解输出的变化。您可以从以下链接下载此文件：[`www.packtpub.com/sites/default/files/downloads/7883OS_ImageBundle.pdf`](https://www.packtpub.com/sites/default/files/downloads/7883OS_ImageBundle.pdf)。

## 勘误

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然会发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误****提交****表**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的**勘误**部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您的帮助，保护我们的作者和我们为您提供有价值内容的能力。

## 问题

如果您对本书的任何方面有问题，您可以联系我们的邮箱 `<questions@packtpub.com>`，我们将尽力解决问题。
