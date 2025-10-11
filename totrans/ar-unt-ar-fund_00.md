# 前言

**增强现实**（**AR**）通过数字化增强内容，使人们能够有意义地与真实世界互动。这本书将帮助你开始使用Unity 3D游戏引擎和Unity Technologies Inc.提供的AR Foundation工具包开发自己的AR应用程序。使用本书中介绍的技术和课程，你将能够为各种目标设备创建自己的AR应用程序和游戏。

AR技术现在在移动消费设备上普遍可用——智能手机和平板电脑，包括iOS（ARKit）和Android（ARCore），以及新一代的可穿戴智能眼镜。在这本书中，我专注于指导你如何为移动设备开发AR应用程序，但技术和项目也可以应用于可穿戴设备。

在本书结束时，你将能够构建和运行自己的AR应用程序，这些应用程序将为现实世界添加信息层，使人们能够与真实和虚拟对象互动，并创新地吸引你的用户。

# 本书面向的对象

如果你想要开发自己的AR应用程序，我建议使用Unity和AR Foundation，因为它是功能最强大和最灵活的平台之一。对创建AR项目感兴趣的开发者可以使用这本书来加速他们在学习曲线上的进步，并通过各种有趣和有趣的项目获得经验。这本书补充了Unity的自身文档和其他资源，并提供了实用的建议和最佳实践，让你能够快速启动并高效工作。

你不需要是Unity专家就能使用这本书，但一些熟悉度将帮助你更快地入门。如果你是初学者，我建议你首先运行Unity Learn上找到的一到两个入门教程([https://learn.unity.com/](https://learn.unity.com/))。对于移动设备（iOS和/或Android）的开发经验也可能有所帮助。

话虽如此，我从一开始就着手，慢慢地引导你沿着学习曲线前进，随着你经验的积累，速度会逐渐加快。如果你想要了解更多内容并深入探索特定主题，我会提供大量的外部资源链接。对于有经验的读者来说，他们可以跳过他们已经知道的指令和解释。

# 本书涵盖的内容

[*第1章*](B15145_01_Final_SB_epub.xhtml#_idTextAnchor013)，*为AR开发做准备*，简要定义了AR后，帮助你为AR开发做好准备，安装Unity 3D游戏引擎和AR Foundation工具包，并确保你的系统为开发Android（ARCore）和/或iOS（ARKit）移动设备做好准备。

[*第2章*](B15145_02_Final_SS_epub.xhtml#_idTextAnchor037)，*你的第一个AR场景*，直接进入构建和运行AR场景，从Unity的AR Foundation样本项目中提供的示例开始，然后逐步过渡到从头开始构建自己的简单场景，了解ARSession组件、预制件，以及一点C#编码。

[*第3章*](B15145_03_Final_SB_epub.xhtml#_idTextAnchor064)，*改进开发者工作流程*，教你关于故障排除、调试、远程测试和Unity MARS的知识，这些可以使你的开发工作流程更加高效。

[*第4章*](B15145_04_Final_SB_epub.xhtml#_idTextAnchor077)，*创建AR用户框架*，你将开发一个用于构建AR应用程序的框架，该框架管理用户交互模式、用户界面面板和AR入门图形，我们将将其保存为模板，以便在本书的其他项目中重复使用。

[*第5章*](B15145_05_Final_SB_epub.xhtml#_idTextAnchor119)，*使用AR用户框架*，你将使用上一章创建的AR用户框架构建一个简单的AR平面放置应用程序，包括主菜单、放置对象模式和UI。本章还讨论了一些高级问题，例如使AR可选、确定设备支持以及为项目添加本地化。

[*第6章*](B15145_06_Final_SB_epub.xhtml#_idTextAnchor136)，*画廊：构建AR应用程序*，是两个章节项目的一部分。在这里，你将开发一个图片画廊应用程序，让你能够在现实世界的墙上挂上虚拟相框照片。在这个过程中，你将了解UX设计、管理数据和对象、菜单按钮和预制件。 

[*第7章*](B15145_07_Final_SB_epub.xhtml#_idTextAnchor170)，*画廊：编辑虚拟对象*，是画廊项目的第二部分，你将学习如何在AR场景中实现与虚拟对象的交互，包括选择和突出显示、移动、调整大小、删除、碰撞检测以及更改相框中的照片。

[*第8章*](B15145_08_Final_SB_epub.xhtml#_idTextAnchor192)，*行星：跟踪图像*，展示了如何构建一个教育AR应用程序，该应用程序使用太阳系“行星卡片”的图像跟踪，实例化虚拟行星在你的桌子上悬浮和旋转。

[*第8章*](B15145_08_Final_SB_epub.xhtml#_idTextAnchor192)，*自拍：制作搞笑表情*，你将学习如何使用设备的正面摄像头制作有趣和娱乐性的面部滤镜，包括3D头像、面部面具（可选材质纹理）以及太阳镜和胡须等配件。它还涵盖了ARCore和ARKit特有的高级功能，这些功能可能不被AR Foundation本身普遍支持。

# 要充分利用这本书

首先，你需要一台能够运行Unity的PC或Mac。最低要求并不困难；几乎任何今天的PC或Mac都足够了（见[https://docs.unity3d.com/Manual/system-requirements.html](https://docs.unity3d.com/Manual/system-requirements.html)）。

如果你正在为iOS开发，你需要一台运行OSX且安装了当前版本XCode的Mac，以及一个Apple开发者账户。如果你正在为Android开发，你可以使用Windows PC或Mac。

没有能够运行您应用程序的设备，开发AR是不切实际的。您应该有一个支持Apple ARKit的iOS设备（在网上搜索；苹果似乎没有发布列表——例如，您可以在这里查看：[https://ioshacker.com/iphone/arkit-compatibility-list-iphone-ipad-ipod-touch](https://ioshacker.com/iphone/arkit-compatibility-list-iphone-ipad-ipod-touch)），或者一个支持ARCore的Android设备（[https://developers.google.com/ar/discover/supported-devices](https://developers.google.com/ar/discover/supported-devices)）。

在[*第1章*](B15145_01_Final_SB_epub.xhtml#_idTextAnchor013)*，设置AR开发环境*中，我向您介绍了安装Unity Hub、Unity编辑器、目标设备的XR插件、AR Foundation工具包和其他软件，以帮助您设置环境。本书中的项目是用Unity 2021.1编写和测试的。

由于技术正在快速发展，我尽量关注现有的稳定工具和技术。关于软件版本和安装说明，自然，事情可能会变化，我建议您将我的说明作为指南，但也查看在线文档（通常提供链接）以获取最新的说明。

**如果您正在使用本书的数字版，我们建议您亲自输入代码或从本书的GitHub仓库（下一节中有一个链接）获取代码。这样做将帮助您避免与代码复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从GitHub下载本书的示例代码文件：[https://github.com/PacktPublishing/Augmented-Reality-with-Unity-AR-Foundation](https://github.com/PacktPublishing/Augmented-Reality-with-Unity-AR-Foundation)。如果代码有更新，它将在GitHub仓库中更新。

我们还从我们丰富的图书和视频目录中提供了其他代码包，可在[https://github.com/PacktPublishing/](https://github.com/PacktPublishing/)找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图和图表彩色图像的PDF文件。您可以从这里下载：[https://static.packt-cdn.com/downloads/9781838982591_ColorImages.pdf](https://static.packt-cdn.com/downloads/9781838982591_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟URL、用户输入和Twitter昵称。以下是一个示例：“利用Unity Input System包，我们将首先添加一个新的`SelectObject`输入动作。”

代码块设置如下：

[PRE0]

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

[PRE1]

**粗体**: 表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词以**粗体**显示。以下是一个示例：“在**新场景**对话框中，选择**ARFramework**模板。”

小贴士或重要注意事项

看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 如果你对本书的任何方面有疑问，请通过[客户关怀](mailto:customercare@packtpub.com)给我们发邮件，并在邮件主题中提及书名。

**勘误**: 尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**: 如果你在互联网上以任何形式遇到我们作品的非法副本，如果你能提供位置地址或网站名称，我们将不胜感激。请通过[版权联系](mailto:copyright@packt.com)我们，并提供材料的链接。

**如果您有兴趣成为作者**: 如果您在某个领域有专业知识，并且您有兴趣撰写或为本书做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。
