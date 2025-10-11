# 前言

Cinder 是目前互联网上最强大的开源创意编程框架之一。它基于最受欢迎和最强大的中级编程语言 C++，并依赖于最少的第三方库。它是低级和高级语言特性的结合，这使得对于那些有高级编程语言背景的人来说相对容易掌握。

Cinder 可以被认为是那些熟悉 Processing、ActionScript 或其他类似高级编程语言或框架的人的下一级创意编程框架。Cinder 可能看起来与 openFrameworks 相似，因为它基于 C++，但框架背后的哲学略有不同。

openFrameworks 基于许多第三方库来完成工作，有时可能会使一个简单的应用程序变得很大。Cinder 在某种程度上是不同的，即它试图利用它所运行的操作系统的功能。这并不意味着它比 openFrameworks 更好，但在某些情况下，它可能比另一个更有效率。

编程是一项迅速成为每个人都应该掌握的技能。爱沙尼亚刚刚为所有上学的儿童引入了计算机编程学习，这意味着从一年级到十二年级的所有学生都在学习如何编码。相信未来会有更多国家效仿这一点并不困难。这意味着在未来，计算机技术将比现在无处不在，并且这种情况将需要大量的人来照顾它。另一个事实是，我们很可能会看到编程在更多以前与它几乎没有或完全没有联系的领域中出现。

正是在这里，创意编程发挥了作用。创意编程在当今越来越受欢迎，因为越来越多的人开始欣赏编程能为他们的工作带来的逻辑和交互维度。可以创造出一幅自己绘画的画作，一个活生生的雕塑，或者一个不允许任何人坐上去的互动椅子——可能性是无限的。这仅仅取决于个人的创造力和编程知识。

在当今的创意编程框架中，一个重要的事情是性能和灵活性。通常，一个框架越强大、越灵活，学习它就越困难。由于 Cinder 既灵活又强大，理解如何使用它并不是一件容易的事情。

本书试图使 Cinder 的学习曲线不那么陡峭。通过一系列简单的引导示例，我们将尝试涵盖其他创意编程工具通常提供的基本功能。

# 本书涵盖内容

第一章, *学习 Cinder 基础知识 – 现在就学！*，提供了关于创意编码和 Cinder 的简要介绍。这将帮助您在 Mac OS X 和 Windows 机器上设置和测试 Cinder。

第二章, *了解可能实现的内容 – Cinder 工具集*，介绍了通过编译、运行和讨论一些 Cinder 示例应用程序，可以使用 Cinder 执行的各种基本任务。

第三章, *初始设置 – 创建 BaseApp*，解释了如何使用 Cinder 的集成工具 TinderBox 创建基础项目，以及从头开始创建。

第四章, *准备画笔 – 绘制基本形状*，深入介绍了 Cinder 内置的基本形状绘制函数，这些函数通常在其他创意编码框架中也有。

第五章, *利用图像 – 加载和显示*，解释了图像的加载和显示，这是每个创意编码语言的重要部分。在本章中，我们将学习如何使用 Cinder 从本地和网络存储加载和显示图像。

第六章, *加速 – 创建生成动画*，教授使用 Cinder 进行生成或过程动画的基础知识。在整个章节中，我们将创建一个无限循环的动画应用程序。

第七章, *处理图像 – 实时后处理和效果*，解释了 Cinder 提供的几个功能，让您能够操纵图像，并将移动图像帧降低到像素级别，而不会失去速度。在本章中，我们将学习如何使用 Cinder 为静态图像和实时应用应用和创建基本效果。

第八章, *增加深度 – Cinder 3D 基础知识*，介绍了基本 3D 方面和实用方法，以及可以用 Cinder 绘制的 3D 原语。

第九章, *进入声音 – 添加声音和音频*，解释了如何加载、修改、播放和使用声音文件，以及如何使用实时音频输入进行绘制和动画。

第十章, *与用户交流 – 添加交互性和 UI 事件*，解释了如何在 Cinder 中处理鼠标、键盘和其他事件。

附录 A, *基本 Cinder 功能参考*，帮助您找到本书中使用的某些基本 Cinder 功能，以便日后参考。

*附加章节*，*逃离孤独 – 与其他应用程序通信*，本书中未包含，但可在以下链接下载：[`www.packtpub.com/sites/default/files/downloads/Escaping_Singleness.pdf`](http://www.packtpub.com/sites/default/files/downloads/Escaping_Singleness.pdf)

# 为本书所需条件

你只需要一台 Mac OS X 或 Windows 计算机。本书中的代码示例已在 Mac OS X 10.6+和 Windows 7 机器上测试。每个操作系统的开发软件列表将在以下章节中讨论。在这本书中，我们将使用以下软件：

**Mac OS X**

+   Xcode 3+或 Xcode 4+

+   MadMapper

**Windows**

+   Microsoft Visual C++ 10+

+   Windows 平台 SDK

+   DirectX SDK

+   QuickTime SDK

**两者**

+   Pure Data

# 本书面向对象

这本书是为那些想要从低级创意编码框架或其他类型的编码转向使用 Cinder 进行创意编码的人准备的。在这本书中，你不会找到关于编程的一般性介绍章节，你应该已经了解一些相关知识。

如果你之前有使用基于文本的编程语言（如 Processing、ActionScript、JavaScript）或视觉编程语言（如 Pure Data、Max MSP、VVVV 或 Quartz Composer）的经验，这本书就是为你准备的。

# 约定

在这本书中，你会发现许多不同风格的文本，用于区分不同类型的信息。以下是一些这些风格的示例及其含义的解释。

文本中的代码单词如下所示：“我们可以通过使用`include`指令来包含其他上下文。”

代码块设置如下：

```cs
void TextBoxApp::render()
{
  string txt = "Here is some text that is larger than can fit 
  naturally inside of 100 pixels.\nAnd here is another line after 
  a hard break."; [default]
```

当我们希望将你的注意力引到代码块的一个特定部分时，相关的行或项目将以粗体显示：

```cs
void TextBoxApp::render()
{
 string txt = "Here is some text that is larger than can fit 
  naturally inside of 100 pixels.\nAnd here is another line after 
  a hard break."; [default]
```

**新术语**和**重要词汇**将以粗体显示。你会在屏幕上看到，例如在菜单或对话框中的单词，将以如下方式显示：“点击**下一个**按钮将你带到下一屏幕”。

### 注意

警告或重要提示将以这样的框显示。

### 小贴士

小技巧和技巧如下所示。

# 读者反馈

我们欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢什么或可能不喜欢什么。读者反馈对我们开发你真正能从中获得最大价值的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件到 `<feedback@packtpub.com>`，并在邮件主题中提及书名。如果你在某个领域有专业知识，并且对撰写或参与一本书感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 图书的骄傲拥有者，我们有一些可以帮助你从购买中获得最大价值的事情。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户中下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 下载本书彩色图像

我们还为您提供了一个包含本书中使用的截图/图表的彩色图像的 PDF 文件。这些彩色图像将帮助您更好地理解输出中的变化。

您可以从以下链接下载此文件：

[`www.packtpub.com/sites/default/files/downloads/9564OS_ColoredImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/9564OS_ColoredImages.pdf)

## 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误提交**表单链接，并输入您的错误详细信息来报告它们。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站，或添加到该标题的错误清单部分。您可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看任何现有的错误清单。

## 盗版

互联网上版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过以下链接联系我们 `<copyright@packtpub.com>`，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者以及为我们提供有价值内容方面的帮助。

## 咨询

如果您在本书的任何方面遇到问题，可以通过以下链接 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
