# 前言

现在，每个人都想制作游戏——由于游戏产业的民主化和用于设计和开发游戏的工具，这比以往任何时候都更容易实现。本书的编写有几个目的。在编写本书的过程中，Unity 经历了多次重大更新和发布。在本书编写时，Unity 处于 2018.1.1f1 版本，新版本为游戏引擎添加了许多优秀功能。简单来说，在一本书中涵盖所有内容是不可能的！

这本书旨在作为 Unity 学习者的参考指南，帮助他们将技能应用于创建**角色扮演游戏**（RPG）。

# 本书面向的对象

这本书是为那些想要学习和应用他们的 Unity 技能来创建 RPG 的个人而编写的。假设读者对编程概念有基本的理解，并且对 Unity 的 IDE 基础感到舒适。本书为构建您自己的游戏体验提供了强大而坚实的基础，涵盖了核心概念和主题。

# 本书涵盖的内容

第一章，*什么是 RPG？*，提供了关于 RPG 的良好背景信息。它涵盖了历史方面，并给出了现有 RPG 的例子。它讨论了 RPG 的主要方面，涵盖了相关术语，并为本书的其余部分做好了准备。

第二章，*游戏规划*，是我们查看角色定义、角色类属性、角色状态以及如何设置和绑定角色模型的地方，这包括对运动、控制器和逆运动学的探索。

第三章，*RPG 角色设计*，继续扩展玩家角色自定义，并探讨了保存角色状态、**非玩家角色**（NPC）的设置以及 NPC**人工智能**（AI）和交互。

第四章，*游戏机制*，是我们开始规划游戏的地方。我们讨论了在游戏创建过程中我们将需要的不同类型资源和资产，引入了第三人称角色控制器，并创建了我们的初始关卡和脚本。我们还探讨了用于地形生成的地形工具包。

第五章，*游戏大师和游戏机制*，探讨了增强游戏大师脚本，介绍了等级控制器和音频控制器脚本，讨论了角色数据的存储和角色自定义状态，并探讨了主菜单初始用户界面的选项。

第六章, *库存系统*，涵盖了通用库存系统的创建；创建表示库存物品所需的脚本、资源和预制体；设计库存用户界面；以及如何表示库存系统及其物品。

第七章, *用户界面和系统反馈*，讨论了抬头显示、玩家角色信息面板和活动库存物品面板的设计与实现。特别库存物品面板的设计和实现，以及非玩家角色生命条和 UI。

第八章，*多人设置*，讨论了使用 Unity 的 Unet 架构进行多人编程。本章使用两个示例项目来说明这些概念。初始项目是一个坦克游戏，它说明了服务器客户端和数据同步的概念。第二个项目将我们所学应用到创建支持我们的角色模型的场景。

# 要充分利用本书

您需要具备良好的 C#语言理解能力。您还需要对 Unity IDE 的基础知识有良好的理解。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了此书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择支持选项卡。

1.  点击代码下载与勘误。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本解压缩或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Building-an-RPG-with-Unity-2018-Second-Edition`](https://github.com/PacktPublishing/Building-an-RPG-with-Unity-2018-Second-Edition)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还提供来自我们丰富的图书和视频目录中的其他代码包。这些代码包可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。您可以查看它们！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/BuildinganRPGwithUnity2018SecondEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/BuildinganRPGwithUnity2018SecondEdition_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“将下载的 `WebStorm-10*.dmg` 磁盘映像文件挂载为系统中的另一个磁盘。”

代码块设置如下：

```cs
using UnityEngine; 
using UnityEngine.SceneManagement; 

namespace com.noorcon.rpg2e 
{ 
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请发送电子邮件至 `feedback@packtpub.com`，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过 `questions@packtpub.com` 发送电子邮件给我们。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何形式的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过 `copyright@packtpub.com` 联系我们，并提供材料的链接。

**如果您想成为一名作者**：如果您在某个领域有专业知识，并且对撰写或参与一本书籍感兴趣，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用过这本书，为何不在您购买它的网站上留下评论？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解有关 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/).
