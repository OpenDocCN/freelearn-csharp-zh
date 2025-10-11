# 前言

*使用 ink 脚本语言进行动态故事脚本编写*教授你一种易于学习的叙事脚本语言。ink 允许作者无需为每个项目构建全新的系统，即可使用专为简单和高级叙事体验设计的强大标记语言来创建以故事驱动的内 容。结合 ink Unity 集成插件，作者可以与开发者合作，使用 ink 语言编写所有故事内容，并访问其变量、调用函数或使用 Unity 中的代码在故事的不同部分之间移动。

在本书中，我们将从 ink 本身开始。前五章将指导你了解 ink 如何理解故事、管理流程、故事部分之间的移动，以及如何在故事中存储和操作不同的值。这将直接过渡到中间四章，这些章节将介绍如何使用 ink Unity 集成插件及其提供的应用程序编程接口，在 ink 故事和 Unity 项目之间进行通信。

最后，最后三章将重点介绍三个常见用例。我们将从创建对话系统开始，并回顾在使用 ink 和 Unity 时处理数据的一些方法。接下来，我们将探讨如何创建一个高级任务跟踪系统，其中每个 ink 故事都包含一个任务，但 Unity 用于跟踪它们之间的值。最后一个用例将回顾 ink 和 Unity 中的一些常见术语和模式，以帮助开发者开始在项目中使用程序化叙事。

# 本书面向对象

本书面向寻求叙事驱动项目解决方案的 Unity 开发者以及希望在 Unity 中创建交互式故事项目的作者。为了充分利用本书，需要具备 Unity 开发和相关概念的基本知识。

# 本书涵盖内容

*第一章*，*文本、流程、选择和交织*，描述了墨水、流程、选择及其相互关系的核心概念。

*第二章*，*节点、分支和循环模式*，涵盖了故事划分、节点以及如何在它们之间移动。

*第三章*，*序列、循环和文本洗牌*，解释了生成动态文本、替代方案及其不同形式的编程方法。

*第四章*，*变量、列表和函数*，介绍了如何在 ink 中进行不同方式的值存储以及高级编程。

*第五章*，*隧道和线程*，解释了如何将节点链接在一起，描述了隧道是什么，以及如何将内容拉入新的配置——线程。

*第六章*，*Adding and Working with the ink-Unity Integration Plugin*，解释了如何定位、安装和验证 ink-Unity 集成插件的安装。

*第七章*，*Unity API – Making Choices and Story Progression*，介绍了如何在 weave 中选择选项，并使用 Unity 中的 Story API 推进 ink 故事。

*第八章*，*Story API – Accessing ink Variables and Functions*，解释了如何使用 Story API 从 Unity 访问和使用 ink 变量和函数。

*第九章*，*Story API – Observing and Reacting to Story Events*，解释了如何使用 Unity 中的 Story API 观察和响应 ink 中的变化。

*第十章*，*使用 ink 的对话系统*，描述了在 ink 中使用标签、语音标签创建对话系统的通用方法，以及 Unity 中选项的视觉表示如何影响 ink 中的代码。

*第十一章*，*任务追踪和分支叙事*，在 ink 中提供了一个任务的一般模板，如何从 Unity 跟踪多个任务，以及如何在故事中同步 ink 变量。

*第十二章*，*Procedural Storytelling with ink*，介绍了程序化叙事的概念，如何在 ink 中开始使用它，以及 ink 中的相同方法如何在 Unity 中工作。

# 要充分利用本书

您至少需要 Unity 2021.1 和 Inky 0.12.0。所有代码示例已在 Windows 操作系统上测试。然而，它们应该与 Unity 和 Inky 的未来版本兼容。

![](img/B17597_Preface_Table01.jpg)

*第十章**、*第十一章**和*第十二章**包含 Unity 项目。当使用来自 GitHub 的 Unity 项目时，请记住在首次打开项目时要耐心，因为 Unity 会重建文件，并且始终在每个项目中打开* `SampleScene` *场景以查看最终代码。*

**如果您使用的是本书的数字版，我们建议您亲自输入代码或从本书的 GitHub 仓库（下一节中提供链接）获取代码。这样做将有助于您避免与代码复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件，网址为 [`github.com/PacktPublishing/Dynamic-Story-Scripting-with-the-ink-Scripting-Language`](https://github.com/PacktPublishing/Dynamic-Story-Scripting-with-the-ink-Scripting-Language)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有其他来自我们丰富的图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图和图表彩色图像的 PDF 文件。你可以从这里下载：

[`static.packt-cdn.com/downloads/9781801819329_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781801819329_ColorImages.pdf).

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个例子：“每次按钮被点击时，Story 方法`ChooseChoiceIndex()`将使用正确的索引被调用，并且`LoadTextAndWeave()`方法将被再次调用，刷新`currentLinesText`的值并更新屏幕上显示的当前按钮...”

代码块设置如下：

```cs
public class InkLoader : MonoBehaviour
{
    public TextAsset InkJSONAsset;
    // Start is called before the first frame update
    void Start()
    {
        Story exampleStory = new Story(InkJSONAsset.text);
    }
}
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目将被设置为粗体：

```cs
void Start()
{
Story exampleStory = new Story(InkJSONAsset.text);
Debug.Log(exampleStory.Continue());
Debug.Log(exampleStory.Continue());
}
```

**粗体**：表示新术语、重要单词或你在屏幕上看到的单词。例如，菜单或对话框中的单词以粗体显示。以下是一个例子：

1.  在项目窗口中选择**预制**按钮。

1.  在**检查器**视图中，点击**标签**下拉菜单，然后点击**添加** **标签…**选项。

    小贴士或重要提示

    看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：如果你对本书的任何方面有疑问，请通过 customercare@packtpub.com 给我们发邮件，并在邮件的主题中提及书名。

**勘误表**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果你在这本书中发现了错误，我们将不胜感激，如果你能向我们报告这个错误。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**: 如果你在互联网上以任何形式遇到我们作品的非法副本，如果你能提供位置地址或网站名称，我们将不胜感激。请通过版权@packt.com 与我们联系，并提供材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专业知识，并且你感兴趣的是撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享你的想法

一旦你阅读了《使用 ink 脚本语言进行动态故事脚本编写》，我们很乐意听听你的想法！请[点击此处直接进入此书的亚马逊评论页面](https://packt.link/r/1-801-81932-7)并分享你的反馈。

你的评论对我们和科技社区都很重要，并将帮助我们确保我们提供高质量的内容。
