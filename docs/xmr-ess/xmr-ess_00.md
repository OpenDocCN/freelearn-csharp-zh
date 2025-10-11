# 前言

移动应用程序已经彻底改变了我们沟通和相互交往的方式，它们的开发现在正开始达到一定程度的成熟。为了支持移动应用程序的开发，现在有许多工具和环境可用。

Xamarin 是一套近年来越来越成功的工具集，并且越来越受到关注，尤其是那些在 .NET 和 C# 资源上有重大投资的开发店铺。Xamarin 使用 C# 包装器包装每个平台的本地 API，允许开发者以与任何本地开发者几乎相同的方式与环境交互。由于 Xamarin 应用是用 C# 开发的，因此跨平台共享代码的新可能性就出现了，带来了所有相关的优势和挑战。

随着公司寻求采用 Xamarin，将需要新的 Xamarin 开发者；他们从哪里来？在许多情况下，将会有已经熟悉 Android 和 iOS 开发的资深移动开发者。

这就是这本书的用意所在；目的是为已经熟悉 Android 和/或 iOS 开发的开发者提供一条快速通道，以便他们能够掌握 Xamarin 开发。为此，本书不专注于开发 Android 和 iOS 应用的基础知识；相反，我们专注于教授经验丰富的 Android 和 iOS 开发者如何使用 Mono、C# 和 Xamarin 工具套件开发应用程序。具体来说，我们重点关注以下主题：

+   架构：这解释了 Xamarin 产品如何允许使用 Mono 和 C# 开发 Android 和 iOS 应用

+   工具：这描述了提供的工具以支持应用程序的开发

+   代码共享：这解释了可以在 Android 和 iOS 应用之间共享的代码类型以及可能出现的问题

+   分发：这解释了在分发 Xamarin.Android 和 Xamarin.iOS 应用时应考虑的特殊注意事项

应该注意的是，在适当的地方提供了示例应用程序和代码片段。

当我最初开始使用 C# 开发 iOS 应用时，感觉有点奇怪。我并不是 Objective-C 的粉丝，但 C# 何时成为了跨平台工具的首选？我总是非常尊重 Mono 团队所取得的成就，但我通常认为微软最终会禁止 C# 和 .NET 在他们不拥有的任何平台上取得巨大成功。作为一个《星球大战》迷，以及某种程度上的一位极客，我被提醒起《星球大战》第三集中的一段对话。如果你还记得安纳金和帕帕廷之间的一段对话，其中安纳金意识到帕帕廷知道原力的黑暗面；只需将原力的黑暗面替换为 Xamarin，帕帕廷就会转向你说：“Xamarin 是一条通往许多被认为是不自然的能力的道路。”那基本上就是我的感觉；我是不是在为了学习一套最终会完全把我绑定到 Windows 上的跨平台技术而妥协？

两年后，我觉得自己可以比较自信地回答这个问题了，答案是！显然，我们工作在一个动态的行业，事情可能瞬间发生变化，但与 10 年前相比，技术世界已经发生了变化，跨平台的 C#和.NET 似乎现在对微软有利。因此，这种奇怪的感觉随着成功而减弱，看到微软和 Xamarin 之间的关系一直从强到更强，我感到鼓舞。

如果您来自 Objective-C 或 Java 背景，您可能会时不时地有同样的感受，但如果您给这些工具一个机会，我认为您会感到惊讶。

我希望您能将这本书视为您成为使用 Xamarin 产品套件成为高效移动应用开发者的宝贵资源。

# 本书涵盖的内容

第一章, *Xamarin 和 Mono – 通向非自然之路*，提供了 Mono 项目和 Xamarin 提供的基于 Mono 的商业产品套件的概述。

第二章, *揭秘 Xamarin.iOS*，描述了 Mono 和 iOS 平台如何共存，并允许开发者使用 C#构建 iOS 应用。

第三章, *揭秘 Xamarin.Android*，描述了 Mono 和 Android 平台如何共存，并允许开发者使用 C#构建 Android 应用。

第四章, *使用 Xamarin.iOS 开发您的第一个 iOS 应用*，将引导您了解创建、编译、运行和调试一个简单的 iOS 应用的过程。

第五章, *使用 Xamarin.Android 开发您的第一个 Android 应用*，将引导您了解创建、编译、运行和调试一个简单的 Android 应用的流程。

第六章, *分享游戏*，展示了在 Xamarin.iOS 和 Xamarin.Android 应用之间共享代码的各种方法。

第七章, *与 MvvmCross 共享*，引导您了解使用 Xamarin.Mobile 应用，它提供了一个跨平台的 API 来访问位置服务、联系人和设备摄像头。

第八章, *与 Xamarin.Forms 共享*，引导您了解使用 MvvmCross 框架的基本知识，以增加平台间的代码复用。

第九章, *准备 Xamarin.iOS 应用以分发*，讨论了各种分发 iOS 应用的方法，并引导您了解准备 Xamarin.iOS 应用以分发的流程。

第十章，*为分发准备 Xamarin.Android 应用*，讨论了分发 Android 应用的多种方法，并指导您准备 Xamarin.Android 应用分发的整个过程。

# 本书所需

本书包含 Android 和 iOS 示例。创建和运行所有示例的最简单配置是拥有一台基于 Intel 的 Mac，运行 OS X 10.8（Mountain Lion）或更高版本的操作系统，并安装 Xcode、iOS SDK 7.*x*和 Xamarin。由于 Xamarin 默认安装 Xamarin.iOS 和 Xamarin.Android，因此可以使用 Xamarin 的 30 天试用版。

以下点提供了基于特定功能和配置的详细要求。

要创建和执行本书中的 iOS 示例，您需要以下内容：

+   运行 OS X 10.8（Mountain Lion）或更高版本的基于 Intel 的 Mac

+   已安装 Xcode 和 iOS SDK 7 或更高版本

+   已安装 Xamarin.iOS；30 天试用版可供使用

+   iPad 或 iPhone 可能有所帮助，但不是必需的

要使用 Xamarin.iOS 的 Visual Studio 插件，您需要以下内容：

+   运行 Windows 7 或更高版本的 PC

+   已安装 Visual Studio 2010、2012 或 2013；任何非 Express 版本

+   已安装 Xamarin.iOS；30 天试用版可供使用

+   连接到 Mac 的网络连接，满足编译和运行应用的要求

要创建和执行本书中的 iOS 示例，您需要以下内容：

+   运行 Windows 7 或更高版本的 PC，或运行 OS X 10.8（Mountain Lion）或更高版本的基于 Intel 的 Mac

+   已安装 Xamarin.Android；30 天试用版可供使用。Xamarin.Android 包括 Android SDK

+   安卓手机或平板电脑可能有所帮助，但不是必需的

要使用 Xamarin.Android 的 Visual Studio 插件，您需要以下内容：

+   运行 Windows 7 或更高版本的 PC

+   Visual Studio 2010、2012 或 2013；任何非 Express 版本

+   已安装 Xamarin.Android；30 天试用版可供使用。Xamarin.Android 包括 Android SDK

# 本书面向的对象

本书是移动开发者宝贵的资源，他们已经熟悉 Android 和/或 iOS 开发，并需要快速掌握 Xamarin 开发。假设您有 Android、iOS 和 C#的背景。本书提供了 Xamarin 架构的概述，指导您创建和运行示例应用的过程，演示了 Xamarin 提供的工具的使用，并讨论了为分发准备应用的特殊注意事项。

# 习惯用法

在本书中，您将找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词如下所示：“在`SetContentView()`语句上设置断点。”

代码块设置如下：

```cs
protected string GetFilename()
{
  return Path.Combine (
       Environment.GetFolderPath(
           Environment.SpecialFolder.MyDocuments),
          "NationalParks.json");
}
```

新术语和重要词汇将以粗体显示。你会在屏幕上、菜单或对话框中看到这些词汇，例如：“从**项目**菜单中选择**发布 Android 项目**。”

### 注意

警告或重要注意事项将以这样的框显示。

### 小贴士

小贴士和技巧看起来像这样。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢什么或可能不喜欢什么。读者反馈对我们开发你真正能从中获得最大收益的标题非常重要。

要发送给我们一般反馈，只需发送一封电子邮件到<feedback@packtpub.com>，并在邮件的主题中提及书名。

如果你有一本书需要我们出版，请通过[www.packtpub.com](http://www.packtpub.com)上的**建议标题**表单发送给我们，或者发送电子邮件到<suggest@packtpub.com>。

如果你在一个你擅长的主题上有专业知识，并且你对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助你从你的购买中获得最大收益。

## 下载示例代码

你可以从你购买的所有 Packt 书籍的账户中下载示例代码文件。[`www.packtpub.com`](http://www.packtpub.com)。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果你在我们的书中发现错误——可能是文本或代码中的错误——如果你能向我们报告这一点，我们将不胜感激。这样做可以帮助其他读者避免挫败感，并帮助我们改进本书的后续版本。如果你发现任何勘误，请通过访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，选择你的书籍，点击**勘误提交表**链接，并输入你的勘误详情来报告它们。一旦你的勘误得到验证，你的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分中。

## 侵权

互联网上版权材料的侵权是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常重视我们版权和许可证的保护。如果你在网上遇到任何我们作品的非法副本，无论形式如何，请立即向我们提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过<copyright@packtpub.com>与我们联系，并提供疑似侵权材料的链接。

我们感谢您在保护我们作者以及为我们提供有价值内容方面的帮助。

## 问题

如果你在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
