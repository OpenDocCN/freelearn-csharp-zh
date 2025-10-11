# 前言

作为世界上最广泛使用的游戏引擎之一，Unity提供了易于使用且功能强大的游戏开发工具，这无疑吸引了众多开发者选择它来开发自己的游戏。然而，现代游戏开发所需工具不仅限于游戏引擎；其他工具和服务，如云服务，在游戏开发中的应用也越来越广泛。在本书中，我们将探讨如何使用Unity游戏引擎和Microsoft Game Dev，包括Microsoft Azure云和Microsoft Azure PlayFab服务，来创建游戏。

从理解Unity游戏引擎的基本原理开始，你将逐渐熟悉Unity编辑器和用C#编写Unity脚本的关键概念，这将为你制作自己的游戏做好准备。

然后，你将学习如何使用Unity的内置模块，例如UI系统、动画系统、物理系统，以及如何在游戏中集成视频和音频，使你的游戏更加有趣。

随着你逐步阅读各章节，我将带你深入了解高级主题，例如计算机图形学中涉及的数学知识、如何在Unity中使用新的可脚本化渲染管线创建后处理效果、如何使用Unity的C#作业系统实现多线程，以及如何使用Unity的**实体组件系统（ECS**）以数据导向的方式编写游戏逻辑代码，从而提高游戏性能。

在阅读过程中，你还将了解Microsoft Game Dev、Azure云服务、Azure PlayFab，以及如何使用Unity3D PlayFab SDK访问PlayFab API以从云中保存和加载数据。

在阅读完本书后，你将熟悉Unity游戏引擎，对Azure云有高层次的理解，并准备好开发自己的游戏。

# 本书面向对象

本书面向具有中级.NET和C#编程经验的开发者，他们希望学习使用Unity进行游戏开发。假设读者具备基本的C#编程经验。

# 本书涵盖内容

[*第1章*](B17146_01_Final_ASB_ePub.xhtml#_idTextAnchor010)，*Hello Unity*，介绍了Unity游戏引擎的基本原理。从Unity的安装过程开始，然后探索编辑器，你还将了解Unity提供的.NET配置文件和脚本后端，最后，你将全面了解Unity。

[*第2章*](B17146_02_Final_ASB_ePub.xhtml#_idTextAnchor025)，*Unity中的脚本概念*，从上一章继续，详细介绍了Unity中的脚本。它首先介绍了Unity脚本中最常用的类，然后解释了脚本的生命周期。它还涵盖了如何在Unity中创建新的脚本，并将脚本作为组件附加到GameObject上，并通过Unity包管理器演示了如何添加或删除包。

[*第3章*](B17146_03_Final_ASB_ePub.xhtml#_idTextAnchor046), *使用Unity UI系统开发UI*，介绍了在Unity中常用的不同类型的UI元素。此外，本章还讨论了如何通过使用**模型-视图-视图模型**（**MVVM**）架构模式在Unity中开发UI。最后，探讨了针对Unity UI的优化技巧。

[*第4章*](B17146_04_Final_ASB_ePub.xhtml#_idTextAnchor062), *使用Unity动画系统创建动画*，涵盖了Unity动画系统最重要的概念，如动画剪辑、Animator控制器、Avatar和Animator组件。在这里，你将使用动画系统实现3D和2D动画。最后，探讨了针对Unity动画系统的优化技巧。

[*第5章*](B17146_05_Final_ASB_ePub.xhtml#_idTextAnchor078), *与Unity物理系统协同工作*，概述了Unity提供的物理解决方案，包括两个内置物理解决方案，即NVIDIA PhysX引擎和Box2D引擎。它还涵盖了Unity物理系统中的关键概念，如碰撞体和刚体。在这里，你将实现一个基于物理的乒乓球游戏。最后，探讨了针对Unity物理系统的优化技巧。

[*第6章*](B17146_06_Final_ASB_ePub.xhtml#_idTextAnchor095), *在Unity项目中集成音频和视频*，涵盖了Unity音频系统和视频系统中的关键概念，如音频剪辑资产、Audio Source组件、Audio Listener组件和Video Player组件。最后，探讨了针对Unity音频系统的优化技巧。

[*第7章*](B17146_07_Final_ASB_ePub.xhtml#_idTextAnchor121), *在Unity中理解计算机图形学的数学原理*，涵盖了与计算机图形学相关的数学，例如坐标系、向量、矩阵和四元数。

[*第8章*](B17146_08_Final_ASB_ePub.xhtml#_idTextAnchor143), *Unity中的可脚本渲染管线概述*，介绍了在Unity中选择的三种现成的渲染管线，即传统的内置渲染管线和基于可脚本渲染管线的两个预制渲染管线，分别是通用渲染管线和高清晰度渲染管线。它还涵盖了如何使用通用渲染管线资产来配置渲染管线，以及如何使用体积框架将后处理效果应用于游戏。最后，探讨了针对通用渲染管线的优化技巧。

[*第9章*](B17146_09_Final_ASB_ePub.xhtml#_idTextAnchor165), *在Unity中使用面向数据的技术堆栈*，介绍了面向数据设计的概念以及面向数据设计与传统面向对象设计的区别。它还探讨了Unity中的**面向数据的技术堆栈**（**DOTS**）及其构成的三个技术模块——即C#作业系统、ECS和Burst编译器。

[*第10章*](B17146_10_Final_ASB_ePub.xhtml#_idTextAnchor181), *Unity和Azure中的序列化系统与资产管理*，讨论了Unity中的二进制序列化、YAML序列化和JSON序列化。它还涵盖了Unity中的资产工作流程，并以探索如何在Azure云中创建Azure Blob存储服务以及如何将Azure中的可寻址内容加载到Unity项目中结束。

[*第11章*](B17146_11_Final_ASB_ePub.xhtml#_idTextAnchor202), *使用Microsoft Game Dev、Azure云、PlayFab和Unity进行工作*，讨论了Microsoft Game Dev、Microsoft Azure云和Azure PlayFab是什么，以及为什么您应该考虑在游戏开发中使用它们。在这里，您将通过Azure PlayFab的API在Unity项目中实现注册、登录和排行榜功能。

# 要充分利用本书

本书假设您对.NET和C#有一定了解。本书涵盖了基本概念、Unity游戏引擎的高级主题，以及其他技术，如Microsoft Azure云和Azure PlayFab。

您还需要在您的计算机上安装**长期支持**（**LTS**）版本的Unity – 推荐使用2020或更高版本。您可以在[*第1章*](B17146_01_Final_ASB_ePub.xhtml#_idTextAnchor010)*，Hello Unity*中找到如何在您的计算机上安装Unity的说明。所有代码示例都在Windows操作系统上的Unity 2020.3.24上进行了测试。然而，它们也应该适用于未来的版本发布。

您还需要一个Microsoft Azure云订阅，您可以在以下链接申请免费Azure账户：[https://azure.microsoft.com/en-in/free/](https://azure.microsoft.com/en-in/free/)。

![](img/B17146_Preface_Table.jpg)

如果您希望从我们的GitHub存储库下载示例项目，您将需要一个Git客户端；我们推荐GitHub Desktop，因为它是最容易使用的。您可以从以下链接下载：[https://desktop.github.com](https://desktop.github.com)。

如果您使用的是Windows操作系统，您还可以考虑使用Git for Windows。您可以从以下链接下载：[https://git-scm.com/download/win](https://git-scm.com/download/win)。

**如果您使用的是本书的数字版，我们建议您自己输入代码或从本书的GitHub存储库（下一节中有一个链接）访问代码。这样做将帮助您避免与代码复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

本书代码包托管在GitHub上，网址为[https://github.com/PacktPublishing/Game-Development-with-Unity-for-.NET-Developers](https://github.com/PacktPublishing/Game-Development-with-Unity-for-.NET-Developers)。如果代码有更新，它将在现有的GitHub存储库中更新。

我们还有其他来自我们丰富的图书和视频目录的代码包，可在[https://github.com/PacktPublishing/](https://github.com/PacktPublishing/)找到。查看它们！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图和图表彩色图像的PDF文件。您可以从这里下载：[https://static.packt-cdn.com/downloads/9781801078078_ColorImages.pdf](https://static.packt-cdn.com/downloads/9781801078078_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

**文本中的代码**: 表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟URL、用户输入和Twitter昵称。以下是一个示例：“如果某些内容是通过`OnCollisionEnter`在对象碰撞开始时生成的，并且您想在对象碰撞结束时销毁它们，那么您应该考虑使用`OnCollisionExit`。”

代码块设置如下：

[PRE0]

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

[PRE1]

**粗体**: 表示新术语、重要词汇或您在屏幕上看到的词汇。例如，菜单或对话框中的文字以**粗体**显示。以下是一个示例：“选择**3D对象** | **平面**以在编辑器中创建一个新的**平面**对象。”

小贴士或重要注意事项

它看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 如果您对本书的任何方面有疑问，请通过[mailto:customercare@packtpub.com](mailto:customercare@packtpub.com)给我们发邮件，并在邮件主题中提及书名。

**勘误**: 尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，如果您能向我们报告，我们将不胜感激。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**: 如果您在互联网上以任何形式遇到我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过[mailto:copyright@packt.com](mailto:copyright@packt.com)与我们联系，并在邮件中附上材料的链接。

**如果您有兴趣成为作者**: 如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了《面向.NET开发者的Unity游戏开发》，我们很乐意听听您的想法！请[点击此处直接进入此书的亚马逊评论页面](https://packt.link/r/1801078076)并分享您的反馈。

您的评论对我们和科技社区都至关重要，并将帮助我们确保我们提供高质量的内容。
