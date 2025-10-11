# 前言

本书献给 Next-Gen UI 工具包（也称为 NGUI）的初学者。您可能听说过这个 Unity 3D 插件；它因其易于使用和有效的 WYSIWYG 工作流程而受到开发者的欢迎。

NGUI 提供了内置组件和脚本，以创建您项目的精美用户界面，大部分工作都在编辑器内部完成。

通过本书，您将获得创建有趣用户界面的必要知识。本书的七个章节都是实用的，将指导您创建主菜单和 2D 游戏。

# 本书涵盖内容

第一章, *NGUI 入门*，描述了 NGUI 的功能和流程。然后我们将导入插件并创建我们的第一个 UI 系统，并研究其结构。

第二章, *创建小部件*，介绍了我们的第一个小部件并解释了如何配置它。然后解释了如何使用小部件模板创建主菜单。

第三章, *优化您的 UI*，解释了拖放系统以及如何创建可拖动窗口。它还涵盖了使用动画、可滚动文本和 NGUI 的本地化。

第四章, *NGUI 中的 C#*，介绍了 C# 事件方法和将用于创建工具提示、通知和 Tweens 的代码导向组件。

第五章, *构建可滚动视口*，介绍了使用滚动条、键盘箭头和可拖动项目进行交互式全屏滚动的视口。

第六章, *图集和字体自定义*，解释了您如何使用自己的精灵和字体自定义 UI；这将使我们能够修改整个主菜单的外观。

第七章, *使用 NGUI 创建游戏*，涵盖了游戏功能，如生成移动敌人、处理玩家输入以及检测小部件之间的碰撞以创建游戏。

# 本书所需条件

为了跟随本书，您需要可从 [`unity3d.com/unity/download`](http://unity3d.com/unity/download) 获取的 Unity 3D 软件。

您可以使用任何版本的 Unity，但我推荐 4.x 循环。仅仅添加组件按钮和复制粘贴组件功能就能为您节省一些时间。您必须熟悉 Unity 的基本工作流程；GameObject、Layers 和 Components 这些词不应该对您来说是秘密。

所有与编码技能相关的代码都在此处提供，并对每一行代码进行了注释。因此，如果您不熟悉它们，您仍然能够理解它们。

在使用这本书的过程中，我们将创建自己的精灵。如果您不想或不能自己创建这些资产，请不要担心；我为这本书创建的资产将可供下载。

您还需要 Tasharen Entertainment 为 Unity 提供的 NGUI 插件。您可以直接从 Unity Asset Store 购买，或者您可以点击页面底部的“立即购买”按钮[`www.tasharen.com/?page_id=140`](http://www.tasharen.com/?page_id=140)。

# 本书面向对象

无论您是刚开始使用 Unity 3D 的初学者，还是中级开发者，或者是一位寻找有效 UI 解决方案的专业开发者，这本书都是为您准备的。

您已经为 PC、控制台或移动平台上的游戏或应用工作过，但您是否在 Unity 内置 UI 系统中挣扎，以创建您游戏的用户界面和菜单？这就是您应该所在的地方。

一旦您阅读完这本书，您会发现构建用户界面可以变得简单、快速且有趣！

# 习惯用法

在这本书中，您将找到多种文本样式，以区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词如下所示：“声明一个新的`Difficulty`变量来存储当前难度。”

代码块如下设置：

```cs
public void OnDifficultyChange() 
{
  //If Difficulty changes to Normal, set Difficulties.Normal
  if(UIPopupList.current.value == "Normal")
    Difficulty = Difficulties.Normal;
  //Otherwise, set it to Hard
  else Difficulty = Difficulties.Hard;
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```cs
//We will need the Slider
UISlider slider;

void Awake ()
{
  //Get the Slider
  slider = GetComponent<UISlider>();
  //Set the Slider's value to last saved volume
  slider.value = NGUITools.soundVolume;
}
```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“您现在可以点击**创建您的 UI**按钮。”

### 注意

警告或重要注意事项以如下框的形式出现。

### 小贴士

小贴士和技巧看起来像这样。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者的反馈对我们开发您真正能从中获得最大收益的标题非常重要。

要向我们发送一般反馈，只需发送一封电子邮件到`<feedback@packtpub.com>`，并在邮件的主题中提及书名。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为 Packt 书籍的骄傲拥有者，我们有许多事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户下载所有已购买的 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。彩色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/8667OT_ColorGraphics.pdf`](https://www.packtpub.com/sites/default/files/downloads/8667OT_ColorGraphics.pdf)下载此文件。

## 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误清单提交表单**链接，并输入您的错误清单详情来报告它们。一旦您的错误清单得到验证，您的提交将被接受，错误清单将被上传到我们的网站，或添加到该标题的错误清单部分。任何现有的错误清单都可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供疑似盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 询问

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决。
