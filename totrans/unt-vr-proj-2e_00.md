# 前言

今天，我们见证了**虚拟现实**（**VR**）的兴起，这是一项令人兴奋的新技术和创意媒介，它承诺将从根本上改变我们与信息、朋友和整个世界互动的方式。

穿戴 VR**头戴式显示器**（**HMD**），你可以观看立体 3D 场景。你可以通过移动头部来环顾四周，使用空间规模跟踪在空间中行走，并使用位置手控制器与虚拟对象交互。通过 VR，你可以参与完全沉浸式的体验。就像你真的身处另一个虚拟世界一样。

本书采用实践、基于项目的教学方法，教你如何使用 Unity 3D 游戏引擎的具体细节进行虚拟现实开发。我们通过一系列动手项目、逐步教程和深入讨论，使用 Unity 2018 和其他免费或开源软件。虽然 VR 技术正在快速发展，但我们将尝试捕捉到你可以用来制作自己的 VR 游戏和应用的基本原理和技术，使它们沉浸感强且舒适。

你将学习如何使用 Unity 开发可以在 Oculus Rift、Google Daydream、HTC VIVE 等设备上体验的 VR 应用。我们将涵盖对 VR 特别重要且可能独特的技术考虑因素。到本书结束时，你将能够开发丰富和交互式的虚拟现实体验。

**关于作者和第二版**

几年前，我在大学里学习了 3D 计算机图形学，在研究生阶段学习了用户界面设计，然后成立了一家小型软件公司，开发用于管理 AutoCAD 工程图纸的 3D 图形引擎。我们将业务出售给了 Autodesk。在随后的几年里，我专注于 2D 网络应用开发，撰写关于我的技术冒险的博客，并追求了几个新的创业项目。然后，在 2014 年 3 月，我了解到 Facebook 以 20 亿美元收购了 Oculus；这无疑激起了我的兴趣。我立即订购了我第一台 VR 头盔，即 Oculus DK2 开发者套件，并开始在 Unity 中开发小型 VR 项目。

2015 年 2 月，我有了写一本关于 Unity VR 开发的书的想法。Packt 立即接受了我的提案，我突然意识到，“哦不！我必须这么做！”在 6 个月内，2015 年 8 月，这本书的第一版出版了。从提案到提纲，再到章节草案、审阅、最终草案和出版，这是一个很短的时间。我着迷了。当时，我告诉我妻子，我觉得这本书有它自己的生命，“它在我体内挣扎着要出来，我必须让它出来。”她回答说，“听起来你就像怀孕了一样。”

在写作那个时期，谷歌纸盒是一个热门产品，但当时还没有消费级的 VR 设备。DK2 没有手柄控制器，只有 XBox 游戏控制器。在本书发布后的几个月，即 2015 年 11 月，HTC Vive 带着房间规模和位置跟踪手柄控制器进入市场。2016 年 3 月，Oculus Rift 的消费版发布。直到 2016 年 12 月，也就是本书出版后近一年半，Oculus 才发布了其位置跟踪的 Touch 手柄控制器。

自本书第一版以来，许多新的 VR 设备进入市场，硬件和软件功能得到了提升，Unity 游戏引擎也继续添加原生 VR SDK 集成和新功能来支持它们。Oculus、Google、Steam、Samsung、PlayStation、Microsoft 以及许多其他公司都加入了这一领域，随着行业的加速发展和繁荣。

同时，在 2016 年，我与 Packt 合作编写了另一本书，名为《Cardboard VR Projects for Android》，这是一本非 Unity VR 书籍，使用 Java 和 Android Studio 构建 Google Daydream 和纸盒应用。（在那本书中，你将构建并使用自己开发的移动设备 3D 图形引擎）。然后在 2017 年，我又与 Packt 合作编写了第三本书，名为《Augmented Reality for Developers》，这是一本关于 iOS、Android 和 HoloLens 设备上 AR 应用的激动人心且及时的基于 Unity 的项目书。

当我开始编写《Unity Virtual Reality Projects》的第二版时，我本以为这只是一个相对简单的任务，即更新到当前版本的 Unity，添加对位置跟踪手柄控制器支持，以及一些微调。但并非如此简单！虽然第一版中的许多基本原理和建议没有改变，但作为行业，我们在短短几年里学到了很多。例如，在 VR 中实现蹦床（我们在这个版本中取消的一个项目）真的不是一个好主意，因为这可能会引起运动病！

对于第二版，本书进行了重大修订和扩展。每一章和项目都进行了更新。我们将一些主题分成了独立的章节，并添加了全新的项目，例如音频火焰球游戏（第八章，*玩转物理与火焰*），动画（第十一章，*动画与 VR 叙事*），以及优化（第十三章，*优化性能与舒适度*）。我真诚地希望您觉得这本书有趣、有教育意义且有帮助，因为我们所有人都致力于创造伟大的新 VR 内容并探索这个令人惊叹的新媒介。

# 这本书面向谁

如果你对虚拟现实感兴趣，想了解它是如何工作的，或者想创建自己的 VR 体验，这本书适合你。无论你是非程序员且不熟悉 3D 计算机图形，还是在这两方面都有经验但新手，你都会从这本书中受益。任何 Unity 经验都是一种优势。如果你是 Unity 新手，你也可以阅读这本书，尽管你可能首先想完成 Unity 网站上的一些入门教程（[`unity3d.com/learn`](https://unity3d.com/learn)）。

游戏开发者可能已经熟悉这本书中关于 VR 项目的概念，同时学习了许多特定于 VR 的其他想法。已经知道如何使用 Unity 的移动和 2D 游戏设计师将发现另一个维度！工程师和 3D 设计师可能理解许多 3D 概念，但学习如何使用 Unity 引擎进行 VR 开发。应用开发者可能会欣赏 VR 在非游戏领域的潜在用途，并可能想要学习实现这一目标的工具。

# 本书涵盖的内容

第一章，*为每个人提供虚拟现实的一切*，是关于游戏和非游戏应用中消费者虚拟现实的新技术和机会的介绍，包括对立体视觉和头部跟踪的解释。

第二章，*内容、对象和规模*，在构建一个简单的场景时介绍了 Unity 游戏引擎，并回顾了使用 Blender、Tilt Brush、Google Poly 和 Unity EditorXR 等工具创建的 3D 内容的导入。

第三章，*VR 构建与运行*，帮助你设置系统并配置 Unity 项目，以便在目标设备（包括 SteamVR、Oculus Rift、Windows MR、GearVR、Oculus Go 和 Google Daydream）上构建和运行。

第四章，*基于注视的控制*，探讨了 VR 摄像机与场景中对象之间的关系，包括 3D 光标和基于注视的射线枪。本章还介绍了使用 C#编程语言的 Unity 脚本。

第五章，*实用交互对象*，探讨了使用控制器按钮和交互对象进行用户输入事件，使用包括轮询、可脚本化对象、Unity 事件和由工具包 SDK 提供的交互组件在内的各种软件模式。

第六章，*世界空间 UI*，展示了使用 Unity 世界空间画布实现 VR 用户界面（UI）的许多示例，包括抬头显示（HUD）、信息气泡、游戏内对象和基于手腕的菜单调色板。

第七章*，移动与舒适*，深入探讨了在 VR 场景中移动自己的技术，仔细研究了 Unity 第一人称角色对象和组件、移动、传送和房间规模 VR。

第八章，*玩转物理和火焰*，探讨了 Unity 物理引擎、物理材质、粒子系统以及更多 C#脚本，我们在构建一个拍球游戏的同时，根据您最喜欢的音乐击打火球。

第九章，*探索交互式空间*，教授如何构建一个交互式艺术画廊，包括关卡设计、艺术品照明、数据管理和在空间中传送。

第十章，*使用所有 360 度*，解释了 360 度媒体，并在各种示例中使用它们，包括地球仪、球体、全景照片和天空盒。

第十一章，*动画和 VR 叙事*，使用导入的 3D 资源和音轨，以及 Unity 时间轴和动画，构建一个完整的 VR 叙事体验。

第十二章，*社交 VR 元宇宙*，探讨了使用 Unity Networking 组件的多玩家实现，以及为 Oculus 平台头像和 VRChat 房间开发。

第十三章，*优化性能和舒适度*，展示了如何使用 Unity Profiler 和 Stats 窗口来减少您的 VR 应用中的延迟，包括优化您的 3D 艺术、静态照明、高效编码和 GPU 渲染。

# 要充分利用本书

在我们开始之前，有一些事情您需要准备。拿些小吃、一瓶水或一杯咖啡。除此之外，您需要一个安装了 Unity 2018 的 PC（Windows 或 Mac）。

您不需要一个超级强大的计算机配置。虽然 Unity 可以渲染复杂的场景，VR 制造商如 Oculus 已经发布了针对 PC 硬件的推荐规格，但实际上您可以用更少的配置来完成任务；即使是笔记本电脑也足以完成本书中的项目。

要获取 Unity，请访问[`store.unity.com`](https://store.unity.com)，选择 Personal，并下载安装程序。免费的 Personal 版本就足够了。

我们还可选地使用 Blender 开源项目进行 3D 建模。本书不是关于 Blender 的，但如果您想使用，我们会使用它。要获取 Blender，请访问[`www.blender.org/download/`](https://www.blender.org/download/)并按照您平台的说明进行操作。

强烈建议您访问虚拟现实**头戴式显示器**（**HMD**）以尝试您的构建并获得本书中开发项目的第一手体验。虽然并非完全必需，但在 Unity 中工作期间，您可以使用模拟模式。根据您的平台，您可能需要安装额外的开发工具。“第三章，VR 构建和运行”详细介绍了每个设备和平台所需的工具，包括 SteamVr、Oculus Rift、Windows MR、GearVR、Oculus Go、Google Daydream 以及其他。

这样就差不多了——一台 PC，Unity 软件，一个 VR 设备，*第三章中描述的其他工具*，我们就准备好了！哦，如果你从 Packt 网站下载了相关的资源，一些项目将会更加完整，如下所示。

# 下载项目资源和示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的项目资源和示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

你可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择 SUPPORT 标签页。

1.  点击代码下载与勘误表。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本的以下软件解压或提取文件夹：

+   Windows 版的 WinRAR/7-Zip

+   Mac 版的 Zipeg/iZip/UnRarX

+   Linux 版的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，链接为[`github.com/PacktPublishing/Unity-Virtual-Reality-Projects-Second-Edition`](https://github.com/PacktPublishing/Unity-Virtual-Reality-Projects-Second-Edition)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的书籍和视频目录的代码包，可在以下链接找到：**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：https://www.packtpub.com/sites/default/files/downloads/B08826_UnityVirtualRealityProjectsSecondEdition_ColorImages.pdf。

# 使用的约定

本书使用了许多文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。”

代码块设置如下：

```cs
void Update () {
Transform camera = Camera.main.transform;
Ray ray;
RaycastHit hit;
GameObject hitObject; 
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```cs
 using UnityEngine;
using UnityEngine.Networking;
public class AvatarMultiplayer : NetworkBehaviour
{
public override void OnStartLocalPlayer()
{
GameObject camera = Camera.main.gameObject;
transform.parent = camera.transform;
transform.localPosition = Vector3.zero;
GetComponent<OvrAvatar>().enabled = false;
}
} 
```

**粗体**：表示新术语、重要词汇或屏幕上看到的词汇。例如，菜单或对话框中的文字会像这样显示。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要提示看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 发送电子邮件至`feedback@packtpub.com`，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过电子邮件发送至`questions@packtpub.com`。

**勘误**: 尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**: 如果你在互联网上以任何形式遇到我们作品的非法副本，如果你能提供位置地址或网站名称，我们将不胜感激。请通过发送链接至`copyright@packtpub.com`与我们联系。

**如果您有兴趣成为作者**: 如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 公司可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
