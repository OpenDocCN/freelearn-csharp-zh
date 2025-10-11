# 前言

*《Unity 2018 着色器和效果食谱》* 是您熟悉在 Unity 2018 中创建着色器和后处理效果的指南。您将从起点开始，探索后处理堆栈，看看您可以使用着色器以不编写任何脚本的方式影响您所看到的内容的几种可能方法。之后，我们将学习如何从头开始创建着色器，从创建最基本的着色器开始，了解着色器代码的结构。这些基础知识将为您在每一章中进一步学习提供手段，例如学习体积爆炸和毛发着色等高级技术。我们还探讨了新添加的着色器图编辑器，看看您如何可以通过拖放界面创建着色器！本书的这一版专为 Unity 2018 编写，将帮助您掌握基于物理的渲染和全局照明，尽可能接近照片级真实感。

每章结束时，您将获得新的技能，这将提高您着色的质量，甚至使您的着色器编写过程更加高效。这些章节已经定制，以便您可以跳入每个部分，从初学者到专家学习特定的技能。对于那些刚开始在 Unity 中编写着色器的人来说，您可以逐章学习，一次学习一章，以构建您的知识。无论哪种方式，您都将学习使现代游戏看起来如此之美的技术。

完成本书后，您将拥有一套可以在您的 Unity 3D 游戏中使用，并了解如何添加到它们中，实现新效果，并解决性能需求的着色器。那么，让我们开始吧！

# 本书面向的对象

*《Unity 着色器和效果食谱》* 是为想要在 Unity 2018 中创建第一个着色器或通过添加专业的后处理效果将游戏提升到全新水平的开发者编写的。需要具备扎实的 Unity 理解能力。

# 本书涵盖的内容

第一章，*后处理堆栈*，向读者介绍了后处理堆栈，这将使用户能够在不编写任何额外脚本的情况下调整游戏的外观。

第二章，*创建您的第一个着色器*，向您介绍了 Unity 中着色器编码的世界。您将构建一些基本的着色器，并学习如何在您的着色器中引入可调整的属性，使它们更具交互性。

第三章，*表面着色器和纹理映射*，涵盖了您可以使用表面着色器实现的最常见和有用的技术，包括如何使用纹理和法线图为您模型建模。

第四章, *理解光照模型*，深入解释了着色器如何模拟光的行为。本章教你如何创建用于模拟特殊效果的自定义光照模型，例如卡通着色。

第五章, *Unity 5 中的基于物理的渲染*，展示了基于物理的渲染是 Unity 5 用于将现实感带入游戏的标准技术。本章解释了如何通过掌握透明度、反射表面和全局照明来充分利用它。

第六章, *顶点函数*，教你如何使用着色器来改变物体的几何形状。本章介绍了顶点修改器，并使用它们将体积爆炸、雪着色器和其他效果栩栩如生地呈现出来。

第七章, *片段着色器和抓取通道*，解释了如何使用抓取通道来制作模拟半透明材料产生的变形的材料。

第八章, *移动着色器调整*，帮助你优化着色器，以充分利用任何设备。

第九章, *使用 Unity 渲染纹理的屏幕效果*，展示了如何创建特殊效果和视觉，这些效果在其他情况下是无法实现的。

第十章, *游戏玩法和屏幕效果*，介绍了如何使用后处理效果来补充你的游戏玩法，例如模拟夜视效果。

第十一章, *高级着色技术*，介绍了本书中最先进的技术，如毛发着色和热图渲染。

第十二章, *着色器图*，解释了如何设置项目以使用 Unity 新添加的着色器图编辑器。我们涵盖了如何创建简单的着色器图，如何公开属性，以及如何通过代码使用发光高亮系统与着色器图交互。

# 为了充分利用本书

预期读者具备使用 Unity 的经验以及一些脚本编写经验（C#或 JavaScript 均可）。本书以 Unity 2018.1.0f2 编写，但应可通过一些小调整与未来版本的引擎兼容。

# 下载示例代码文件

你可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果你在其他地方购买了这本书，你可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择 SUPPORT 选项卡。

1.  点击代码下载与勘误。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载完成后，请确保您使用最新版本解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

书籍的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Unity-2018-Shaders-and-Effects-Cookbook-Third-Edition`](https://github.com/PacktPublishing/Unity-2018-Shaders-and-Effects-Cookbook-Third-Edition)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有来自我们丰富的书籍和视频目录中的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表的彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/Unity2018ShadersandEffectsCookbookThirdEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/Unity2018ShadersandEffectsCookbookThirdEdition_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“Unity 包是一个包含各种`Assets`的单个文件，这些`Assets`可以在 Unity 中以类似`.zip`文件的方式使用。”

代码块设置为如下：

```cs
Properties 
{
  _MainTex("Texture", 2D) = "white" 
}
```

当我们希望将您的注意力引向代码块中的特定部分时，相关的行或项目将以粗体显示：

```cs
void surf (Input IN, inout SurfaceOutputStandard o) {
  // Use the tint provided as the base color for the material
  o.Albedo = _MainTint;

  // Get the normal data out of the normal map texture 
  // using the UnpackNormal function 
  float3 normalMap = UnpackNormal(tex2D(_NormalTex, 
    IN.uv_NormalTex)); 

 normalMap.x *= _NormalMapIntensity; 
 normalMap.y *= _NormalMapIntensity; 

 // Apply the new normal to the lighting model 
 o.Normal = normalize(normalMap.rgb); 
}
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“要最终烘焙灯光，请通过转到 Window | Lighting | Settings 打开 Lighting 窗口。一旦到达那里，选择 Global Maps 选项卡。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 部分

在本书中，您将找到一些频繁出现的标题（*准备就绪*、*如何操作...*、*如何操作...*、*还有更多...*和*相关内容*）。

为了清楚地说明如何完成食谱，请按以下方式使用这些部分：

# 准备就绪

本节将向您介绍在食谱中可以期待的内容，并描述如何设置任何软件或任何为食谱所需的初步设置。

# 如何操作...

本节包含遵循食谱所需的步骤。

# 如何操作...

本节通常包含对上一节发生情况的详细说明。

# 还有更多…

本节包含有关食谱的附加信息，以便您对食谱有更深入的了解。

# 相关内容

本节提供了链接到其他对食谱有用的信息。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请发送邮件至 `feedback@packtpub.com` 并在邮件主题中提及书名。如果您对本书的任何方面有疑问，请通过 `questions@packtpub.com` 发送邮件给我们。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在本书中发现错误，我们将不胜感激，如果您能向我们报告这一错误。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现任何形式的我们作品的非法副本，我们将不胜感激，如果您能提供位置地址或网站名称。请通过 `copyright@packtpub.com` 联系我们，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

留下评论。一旦您阅读并使用过本书，为何不在您购买书籍的网站上留下评论呢？潜在读者可以看到并使用您的客观意见来做出购买决定，Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
