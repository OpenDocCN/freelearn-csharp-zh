# *第六章*：画廊：构建 AR 应用

在本章中，我们将开始构建一个完整的**增强现实**（AR）应用，一个 AR**艺术画廊**，让你能在现实世界的墙上挂上虚拟的框架照片。

首先，我们将定义项目的目标，并讨论项目规划和**用户体验**（UX）设计的重要性。当用户在主菜单中按下**添加**按钮时，他们将看到一个**选择图像**菜单。当他们选择一个时，他们将被提示将图像的框架副本放置在他们的现实世界墙上。

要实施项目，我们将从本书早期创建的 AR 用户框架场景模板开始。我们将构建一个选择图像 UI 面板和交互模式，并定义应用使用的图像数据。

在本章中，我们将涵盖以下主题：

+   指定新的项目和 UX 设计

+   使用数据结构和数组，以及在对象之间传递数据

+   创建一个带有按钮网格的详细 UI 菜单面板

+   创建用于在 AR 场景中实例化的预制件

+   基于给定的用户故事实现完整场景

到本章结束时，你将拥有一个实现一个场景的工作原型：在墙上放置图片。然后我们将在下一章继续构建和改进项目。

# 技术要求

要在本章中实现项目，您需要在您的开发计算机上安装 Unity，并将其连接到一个支持 AR 应用程序的移动设备（有关说明，请参阅*第一章*，*为 AR 开发设置*）。我们还假设您已安装了`ARFramework`模板及其先决条件；请参阅*第五章**，使用 AR 用户框架*。完成的项目可以在本书的 GitHub 存储库中找到，[`github.com/PacktPublishing/Augmented-Reality-with-Unity-AR-Foundation`](https://github.com/PacktPublishing/Augmented-Reality-with-Unity-AR-Foundation)。

# 指定艺术画廊项目的 UX

在开始任何新项目之前的一个重要步骤是在事先做一些设计和规范。这通常意味着将其写入文档。对于游戏，这可能被称为**游戏设计文档**（GDD）。对于应用程序，它可能是一个**软件设计文档**（SDD）。无论你称之为什么，其目的是在开发开始之前将项目的蓝图写入文档。一个 Unity AR 项目的详细设计文档可能包括以下细节：

+   *项目概述*：总结项目的概念和目的，确定主要受众，并可能包括一些关于项目存在的原因以及它将如何成功的背景信息。

+   *用例*：确定产品将解决的实际生活问题。通常，定义代表应用程序不同用户类型、他们的主要目标和如何使用应用程序来实现这些目标的单独用户**角色**（可以是真实或虚构的名字）是有效的。

+   *关键特性*：确定为用户提供价值的离散功能区域，可能强调其与其他类似解决方案的区别。

+   *UX 设计*：**用户体验（UX）**设计可能包括各种用户场景，详细说明特定的工作流程，通常以**故事板**的形式呈现，使用抽象的铅笔或线框草图。如果没有绘画技巧，白板会议的照片捕捉和便利贴可能就足够了。

    此外，你还可以包括 UI 图形设计，定义实际的风格指南和图形，例如，色彩方案、字体、按钮图形等。

+   *资产*：收集和分类你预计需要的图形资产，包括概念艺术、3D 模型、效果和音频。

+   *技术计划*：这包括将要使用的软件架构和设计模式，开发工具（例如 Unity、Visual Studio 和 GitHub），Unity 版本，第三方包（例如，通过包管理器），以及 Unity 服务和其它云服务（如广告、网络和数据存储）。

+   *项目计划*：实施计划可能显示预期的项目阶段、生产和发布时间表。这可能涉及使用诸如 Jira 或 Trello 之类的工具。

+   *商业计划*：非技术规划可能包括项目管理、营销、资金、货币化、用户获取和社区建设计划。

对于非常大的项目，这些部分可能是单独的文档。对于小型项目，整个文档可能只有几页长，包含项目符号。记住，主要目的是在编写代码之前思考你的计划。话虽如此，不要过度设计。记住我非常喜欢爱因斯坦的一句话：

"尽可能使一切变得尽可能简单，但不能过于简单。"

假设随着项目的发展，事情可能会并且将会发生变化。快速迭代、利益相关者的频繁反馈和吸引真实用户可能会重申你的计划。或者，它可能会暴露原始设计中的严重缺陷，并将项目引向新的、更好的方向。正如我对我客户和学生的建议：

"你对项目了解最少的时候是在项目开始时！"

在这本书中，我将在每个项目的开始提供一个简化的设计计划，试图捕捉最重要的点，而不涉及太多细节。让我们从这个 AR 画廊项目开始，并具体说明项目目标、用例、一个 UX 设计和一组定义项目关键特性的用户故事。

## 项目目标

我们将构建一个 AR 艺术画廊项目，允许用户使用 AR 将他们最喜欢的照片作为虚拟相框图像放置在他们的家或办公室的墙上。

## 用例

*人物：杰克*。杰克在家工作，没有时间装饰他那单调的公寓。杰克希望通过在墙上添加一些漂亮的图片来装饰墙壁。但他的房东不允许在墙上钉钉子。约翰也希望能够经常更换他挂的照片。杰克每天花费很多时间使用他的手机，所以通过手机看墙壁是令人满意的。

*人物：吉尔*。吉尔有一大堆喜欢的照片。她希望将它们挂在办公室的墙上，但这在办公环境中并不太合适。此外，她有点强迫症，因此希望经常重新排列照片并交换图片。

## UX 设计

此应用程序的用户体验（UX）必须包括以下要求和场景：

+   当用户想要在墙上放置照片时，他们从菜单中选择一张图片，然后轻触屏幕，指示放置照片的位置。

+   当用户想要修改已经放置在墙上的照片时，他们可以轻触照片以启用编辑功能。然后用户可以拖动来移动，捏合来调整大小，选择不同的照片或相框，或者滑动来移除照片。

+   当相框照片渲染时，它会匹配当前的房间照明条件，并在真实世界的表面上投下阴影。

+   当用户退出并重新打开应用时，他们放置在房间中的所有照片都将被保存并恢复到它们的位置。

我请了一位专业的 UX 设计师（也是我的朋友）Kirk Membry ([`kirkmembry.com/`](https://kirkmembry.com/)) 为这本书的项目准备 UX 线框草图。以下图像显示了整个故事板的一些框架：

![图 6.1 – UX 设计线框草图](img/Figure_6.01-ux-design-example.jpg)

图 6.1 – UX 设计线框草图

最左侧的框架显示了当用户选择将新照片添加到场景中时出现的图像库菜单。中间的框架描述了用户选择在墙上挂照片的位置。最右侧的框架显示了用户正在编辑现有的图片，包括用于移动和调整大小的手势，以及屏幕底部的其他编辑选项菜单。

这样的故事板可以用来与图形设计师、编码人员和利益相关者沟通设计意图。它可以作为讨论的基础，以消除用户工作流程中的瑕疵和用户界面的不一致性。它可以大大提高项目管理效率，通过在成本最高的阶段——在功能实现之后——防止不必要的返工。

在设计草案足够的情况下，我们现在可以选出一些在构建项目时将使用的资产。

## 用户故事

将功能分解成一系列“用户故事”或小块功能，可以逐步实现，一次构建项目的一部分，这是很有用的。在一个敏捷管理的项目中，团队可能会选择在一到两周的*冲刺*中完成的一组特定的故事。这些故事可以在共享的项目板上进行管理和跟踪，例如 Trello ([`trello.com/`](https://trello.com/)) 或 Jira ([`www.atlassian.com/software/jira`](https://www.atlassian.com/software/jira))。以下是本项目的几个故事：

+   当应用启动时，我被提示扫描房间，同时设备在环境中检测和跟踪垂直墙壁。

+   在跟踪建立后，我可以看到一个主菜单，其中有一个**添加**按钮。

+   当我按下**添加**按钮时，我可以看到一组照片的选择。

+   当我从选择中选择一张照片时，我可以看到跟踪的垂直平面，并被提示在墙上挂一个带框的图片（图片）。

+   当图片实例化时，它垂直地紧贴墙面。

+   当我点击现有的虚拟图片以开始编辑图片时。

+   当编辑图片时，我可以看到一个编辑菜单，可以选择更改照片、更改框架或删除带框的图片。

+   当编辑图片时，我可以将图片拖动到新位置。

+   当编辑图片时，我可以使用捏合（使用两个手指）来调整大小。

这看起来像是一组很好的功能。我们将尝试在本章中完成它们的前半部分，并在下一章中完成它们。让我们开始吧。

# 开始

要开始，我们将使用`ARFramework`场景模板创建一个名为`ARGallery`的新场景，以下是步骤：

1.  选择**文件 | 新场景**。

1.  在**新场景**对话框中，选择**ARFramework**模板。

1.  选择**创建**。

1.  在你的项目`Assets`文件夹中的`Scenes/`文件夹中，将其命名为`ARGallery`，并选择**保存**。

新的 AR 场景已经包含以下对象：

+   一个**AR 会话**游戏对象。

+   一个带有射线管理器和平面管理器组件的**AR 会话起源**装置。

+   **UI Canvas**是一个屏幕空间画布，包含子面板启动 UI、扫描 UI、主 UI 和非 AR UI。它具有我们编写的 UI 控制器组件脚本。

+   **交互控制器**是一个具有我们编写的交互控制器组件脚本的游戏对象，它帮助应用在交互模式之间切换，包括启动、扫描、主和非 AR 模式。它还配置了**玩家输入**组件，该组件使用我们之前创建的**AR 输入动作**资产。

+   来自 AR Foundation Demos 项目的**OnboardingUX**预制件，它提供 AR 会话状态和特征检测状态消息，以及动画引导图形提示。

我们现在为 AR 画廊项目制定了计划，包括目标声明、用例和带有一些用户故事的 UX 设计以实现。有了这个场景，我们就准备好了。让我们找到我们可以工作的照片集合并将它们添加到项目中。

# 收集图像数据

在 Unity 中，图像可以导入用于各种目的。纹理是可以用于渲染 3D 对象表面的材质的图像。UI 使用图像作为按钮和面板图形的精灵。对于我们的相框照片，我们将使用图像作为…图像。

在您的应用程序中使用图片的最基本方法是将它们导入到您的`Assets`文件夹中，并将它们作为 Unity 纹理进行引用。一个更高级的解决方案是在运行时动态查找和加载它们。在本章中，我们将使用前者技术并将图像列表构建到应用程序中。让我们首先导入您想要使用的照片。

## 导入照片以使用

请从您的收藏夹中选择一些图像用于您的画廊。或者，您可以使用本书 GitHub 仓库中包含的图像，这些图像是从 Unsplash.com（[`unsplash.com/`](https://unsplash.com/））找到的免费可用的自然照片集合，以及我自己的一张名为`WinterBarn.jpg`的照片。

要将图像导入到您的项目中，请按照以下步骤操作：

1.  在`Photos`中，通过右键单击，然后选择**创建 | 文件夹**。

1.  从您的 Windows 资源管理器或 OSX Finder 中找到您想要使用的图像。然后将图像文件拖入 Unity，将其放入`Photos/`文件夹中。

1.  在**检查器**窗口中，您可以检查导入的图像的大小。因为我们将在 AR 和相对低分辨率的移动设备上使用它，所以让我们将最大尺寸限制为 1,024 像素。请注意，Unity 要求将纹理导入到 2 的幂次大小，以获得最佳压缩和运行时优化。在**检查器**中，确保选择了**默认**选项卡，并选择**最大尺寸 | 1024**。

现在我们将添加一种在场景中引用您的图像的方法。

## 将图像数据添加到场景

要将图像数据添加到场景中，我们将创建一个包含图像列表的`ImagesData`脚本的空 GameObject。首先，在您的项目`Scripts/`文件夹中创建一个新的 C#脚本，命名为`ImagesData`，并编写如下：

```cs
using UnityEngine;
[System.Serializable]
public struct ImageInfo
{
    public Texture texture;
    public int width;
    public int height;
}
public class ImagesData : MonoBehaviour
{
    public ImageInfo[] images;
}
```

脚本首先定义一个包含图像`Texture`和图像像素尺寸的`ImageInfo`数据结构。它是`public`的，因此可以从其他脚本中引用。然后`ImagesData`类在`images`变量中声明了这个数据结构的数组。`ImageInfo`结构需要`[System.Serializable]`指令，以便在 Unity 检查器中显示。

现在我们可以通过以下步骤将图像数据添加到场景中：

1.  从主菜单中选择`Images Data`（使用三点上下文菜单和**重置**来整理其**变换**）。

1.  将 **ImagesData** 脚本拖放到 **Images Data** 对象上，使其成为一个组件。

1.  要填充 `images` 数组，在 **检查器** 中输入你计划使用的图像数量，或者简单地按右下角的 **+** 按钮以递增方式向数组中添加元素。

1.  通过展开 **Images** 列表中的一个 **Element**，逐个添加导入的图像文件，然后从 **项目** 窗口将图像文件拖放到元素的 **Texture** 槽中。请务必输入每个图像的 **Width** 和 **Height** 像素值。

    我的 **Images Data** 在 **检查器** 中的样子如下：

![图 6.2 – 带有图像列表的图像数据组件![图像 6.02-图像数据检查器.jpg](img/Figure_6.02-ImagesData-inspector.jpg)

图 6.2 – 带有图像列表的图像数据组件

使用 ScriptableObjects

提供图像列表的另一种，可能更好的方法，是使用 ScriptableObjects 而不是 GameObjects。ScriptableObjects 是存在于你的 `Assets/` 文件夹中的数据容器对象，而不是在场景层次结构中。你可以在 [`docs.unity3d.com/Manual/class-ScriptableObject.html`](https://docs.unity3d.com/Manual/class-ScriptableObject.html) 和 [`learn.unity.com/tutorial/introduction-to-scriptable-objects`](https://learn.unity.com/tutorial/introduction-to-scriptable-objects) 上了解更多关于 ScriptableObjects 的信息。

手动输入每张图像的像素尺寸有点繁琐。如果能有一种更好的方法那就太好了，因为这样做并不容易。

## 获取图像的像素尺寸

不幸的是，当 Unity 将图像导入为纹理时，它会将其缩放到 2 的幂以优化运行时性能和压缩，并且原始尺寸数据不会被保留。有几种方法可以解决这个问题，但没有一种是特别美观的：

+   要求开发者手动为每个图像指定像素尺寸。这是我们在这里采取的方法。

+   告诉 Unity 在导入图像时不进行缩放。为此，选择一个图像资产，并在其 `Texture.width` 和 `Texture.height`。

+   采用第一种方法，但使用编辑器脚本来自动确定像素大小。Unity 允许你编写仅在编辑器中运行的脚本，而不是在运行时。编辑器在将原始图像文件导入为纹理之前，可以访问你的 **Assets** 文件夹中的原始图像文件。因此，可以使用系统 I/O 函数或可能是（未记录的）Unity API（见 [`forum.unity.com/threads/getting-original-size-of-texture-asset-in-pixels.165295/`](https://forum.unity.com/threads/getting-original-size-of-texture-asset-in-pixels.165295/)）来读取和查询这些信息。

既然如此，我们将在本章中坚持手动方法，你可以自己探索其他选项。

也许你还在想，如果我不想将图像构建到我的项目中，想在运行时查找和加载它们怎么办？

## 在运行时加载图片列表

在运行时从构建外部加载资源是一个高级主题，并且超出了本章的范围。我将简要描述几种不同的方法，并将提供更多信息的链接：

+   **在资源包中包含图像**：在 Unity 中，你可以选择将资源打包到资源包中，应用程序在用户安装应用程序后可以下载这些资源，作为**可下载内容**（**DLC**）。请参阅[`docs.unity3d.com/Manual/AssetBundlesIntro.html`](https://docs.unity3d.com/Manual/AssetBundlesIntro.html)。

+   **从网络 URL 下载图像**：如果你有图像文件的网址，你可以使用网络请求在运行时下载图像，并将其用作应用程序中的纹理。请参阅[`docs.unity3d.com/ScriptReference/Networking.UnityWebRequestTexture.GetTexture.html`](https://docs.unity3d.com/ScriptReference/Networking.UnityWebRequestTexture.GetTexture.html)。

+   **从设备的照片应用获取图像**：对于像我们的图库这样的应用程序，从用户的照片应用中获取照片是很自然的。要访问移动设备上其他应用程序的数据，你需要一个具有本地访问权限的库。这也可能需要你的应用程序从用户那里获得额外的权限。在 Unity Asset Store 中搜索包。

如果你想实现这些功能，那就由你自己来决定。

我们现在已经导入了我们计划使用的照片，创建了一个包含每个图像像素尺寸的 C# `ImageInfo`数据结构，并在场景中填充了这些图像数据。让我们创建一个包含默认图像和可以放置在墙面上的照片框架的装框照片预制体。

# 创建一个装框照片预制体

用户将在他们的墙上放置一个装框的照片。因此，我们需要创建一个预制游戏对象，它将被实例化。我们希望使其易于更改图像和框架，以及根据不同的方向（横向与纵向）和图像宽高比进行缩放。对于默认框架，我们将从一个压扁的 3D 立方体创建一个简单的块，并将照片安装在其表面上。对于默认图像，你可以选择自己的，或者使用 GitHub 仓库中本章文件包含的一个。

## 创建预制体层次结构

首先，在项目文件夹的`Assets/`中创建一个名为`FramedPhoto`的空预制体。按照以下步骤操作：

1.  在`Prefabs/`文件夹中（如果需要，请创建一个）。然后在该文件夹中*右键单击*并选择**创建 | 预制体**。

1.  将新预制体重命名为`FramedPhoto`。

1.  *双击* **FramedPhoto** 资产（或在**检查器**窗口中点击其**打开预制体**按钮）。

    我们现在正在编辑一个空预制体。

1.  添加一个子`AspectScaler`。

1.  让我们使用一个压扁的立方体来创建一个看起来现代的黑色矩形框架。使用`Frame`。

1.  给框架一些厚度。在框架的`0.05`（这是以米为单位）。

1.  同样，通过设置`-0.025`将其从墙上偏移。

1.  要给这个框架一个黑色表面，创建并添加一个新的材质，如下所示。

    在 `Materials/` 文件夹中（如果需要，请创建一个）。然后 *右键单击* 文件夹并选择 `Black Frame Material`。

1.  将其 **基础地图** 颜色设置为炭黑色。

1.  然后，在 **层次结构** 中，选择 **Default Frame** 对象，并将 **Black Frame Material** 拖放到它上面。

以下截图显示了当前的框架属性：

![图 6.3 – FramedPhoto 的框架属性](img/Figure_6.03-FrameProperties.jpg)

图 6.3 – FramedPhoto 的框架属性

接下来，我们将向包含在此书文件中的 `WinterBarn.jpg` 添加一个默认图像。使用以下步骤创建一个使用此照片作为纹理图像的材质图像对象：

1.  使用 `Image`。一个 **quad** 是最简单的 Unity 3D 基本对象，是一个只有四个边且朝一个方向的面板。

1.  要将你的图像作为 quad 上的纹理添加，我们需要创建一个材质。在 `Materials/` 文件夹中，*右键单击* 文件夹并选择 `Image Material`。

1.  将你的图像文件（`WinterBarn.jpg`）从 **项目** 窗口拖动到 **检查器** 窗口，将其放置在 **基础地图** 属性左侧的小正方形“芯片”槽中。

1.  将 **Image Material** 拖动到 **Image** 游戏对象上。

1.  将图像 quad 偏移，使其略微位于框架立方体的平面之前。设置其 `-0.06`。

1.  你现在应该能看到图像。但是框架是隐藏的，因为图像 quad 被缩放到了与框架相同的大小。通过设置其 `0.9` 来缩小图像。

预制件层次结构现在看起来如下截图所示，其中图像当前被选中并在 **检查器** 中可见：

![图 6.4 – FramedPhoto 预制件](img/Figure_6.04-FramedPhoto-prefab.jpg)

图 6.4 – FramedPhoto 预制件

接下来，让我们添加一个简单的脚本，它将帮助我们的其他代码设置 **FramedPhoto** 对象的图像。

## 编写 FramedPhoto 脚本

我们将需要为 `SetImage` 函数的每个实例设置各种属性，该函数获取此图片的图像数据。

创建一个名为 `FramedPhoto` 的新 C#脚本，打开它进行编辑，并编写以下脚本：

```cs
using UnityEngine;
public class FramedPhoto : MonoBehaviour
{
    [SerializeField] Transform scalerObject;
    [SerializeField] GameObject imageObject;
    ImageInfo imageInfo;
    public void SetImage(ImageInfo image)
    {
        imageInfo = image;
        Renderer renderer =             imageObject.GetComponent<Renderer>();
        Material material = renderer.material;
        material.SetTexture("_BaseMap", imageInfo.texture);
    }
}
```

在 `FramedPhoto` 类的顶部，我们声明了两个属性。`imageObject` 是对子 `scalerObject` 的引用，`scalerObject` 是对 **AspectScaler** 的引用，当脚本需要更改其纵横比时（我们在本章末尾这样做）。

当将 `SetImage` 用于更改 `Renderer`，然后获取其 `Material`，接着设置其基础纹理。

我们现在可以将此脚本按以下方式添加到预制件中：

1.  在编辑 **FramedPhoto** 预制件时，将 **FramedPhoto** 脚本拖动到 **FramedPhoto** 根对象上，使其成为一个组件。

1.  从 **层次结构** 中，将 **AspectScaler** 对象拖动到 **检查器** 并将其放置在 **Framed Photo | Scaler Object** 槽中。

1.  从 **层次结构** 中，将 **Image** 对象拖动到 **Framed Photo | Image Object** 槽中。

我们的预制件现在几乎准备好使用了。当然，我们使用的图片并不是真的应该是正方形的，所以让我们将其缩放。

## 缩放图片的形状

我默认使用的照片是横向的，但我们的框架是正方形的，所以看起来被压扁了。为了修复它，我们需要获取图像的原始像素大小并计算其宽高比。例如，GitHub 上包含在此书中的`WinterBarn.jpg`图像是 4,032x3,024（宽度 x 高度），或 3:4（`高度:宽度`的横向比例）。现在让我们根据图像的宽高比（`0.75`）进行缩放。按照以下步骤操作：

1.  在**层次结构**窗口中，选择**缩放器**对象。

1.  设置其`0.75`（如果你的图像是竖直的，则缩放`1.0`）。

    正确缩放的预制件现在看起来如下：

    ![Figure 6.5 – FramedPhoto prefab with corrected 3:4 landscape aspect ratio]

    ![img/Figure_6.05-FramedPhoto-aspect.jpg]

    Figure 6.5 – FramedPhoto prefab with corrected 3:4 landscape aspect ratio

1.  通过点击**场景**窗口右上角的**保存**按钮保存预制件。

1.  使用**<**按钮返回场景编辑器，该按钮位于**层次结构**窗口的左上角。

1.  设置`1, 1, 1`)，无论其中照片的宽高比或框架的厚度如何。这将有助于放置和缩放场景中带框照片的用户界面。

1.  3:4 宽高比的高度为`0.75`。

1.  `0.05`大小的边框，所以`0.9`。

1.  图像的前后偏移量也将取决于框架的模型。在这种情况下，我将它移得更近，从`-0.06`到`-0.025`单位，因此它稍微位于框架表面之前。

当组装预制件时，思考它如何避免后续的麻烦。

在本节中，我们创建了一个可缩放的`Assets`文件夹，以便当用户在墙上放置图片时，可以在场景中实例化副本。预制件包括一个`FramedPhoto`脚本，该脚本管理预制件的一些行为方面，包括设置其图像纹理。这个脚本将在本章后面进行扩展。我们现在有一个带有框架的**FramedPhoto**预制件。我们准备好添加放置图片在墙上的用户交互。

# 在墙上挂虚拟照片

对于这个项目，应用会扫描环境中的垂直平面。当用户想要在墙上挂一幅画时，我们会显示一个 UI 面板，指导用户点击放置对象，并使用动画图形。一旦用户点击屏幕，**AddPicture**模式就会实例化一个**FramedPhoto**预制件，使其看起来挂在墙上，垂直且与墙面平面齐平。许多这些步骤与我们之前在*第五章**，使用 AR 用户框架*中做的类似，所以这里我会提供较少的解释。我们将从一个类似的脚本开始，然后进行增强。

## 检测垂直平面

由于 AR 会话原点已经有一个 AR Plane Manager 组件（在默认的`ARFramework`模板中提供），请按照以下步骤设置场景以扫描垂直平面（而不是水平平面）：

1.  在**层次结构**窗口中，选择**AR 会话原点**对象。

1.  在其**检查器**窗口中，首先选择**Nothing**（清除所有选择），然后选择**Vertical**，将**AR Plane Manager | Detection Mode**设置为**垂直**。

现在让我们创建一个**AddPicture** UI 面板，提示用户点击垂直平面以放置新图片。

## 创建 AddPicture UI 面板

**AddPicture UI**面板类似于场景模板中包含的**Scan UI**，因此我们可以复制并按以下方式修改它：

1.  在**层次结构**窗口中，展开**UI Canvas**。

1.  *右键单击* `AddPicture UI`。

1.  展开**AddPicture UI**并选择其子项，**动画提示**。

1.  在**检查器**中，将**动画提示 | 指示**设置为**点击放置**。

1.  要将面板添加到 UI 控制器中，在**层次结构**中，选择**UI Canvas**对象。

1.  在**检查器**中，在**UI Controller**组件的右下角点击**+**按钮，向 UI Panels 字典中添加一个项。

1.  在**Id**字段中输入`AddPicture`。

1.  从**层次结构**中将**AddPicture UI**游戏对象拖动到**值**槽位。

我们为**AddPicture** UI 添加了一个指导性用户提示。当用户选择将图片添加到场景中时，我们将进入**AddPicture**模式，并显示此面板。现在让我们创建**AddPicture**模式。

## 编写初始 AddPictureMode 脚本

要将模式添加到框架中，我们在**Interaction Controller**下创建一个子 GameObject 并编写一个模式脚本。模式脚本将显示模式的 UI，处理任何用户交互，并在完成后过渡到另一个模式。对于 AddPicture 模式，它将显示**AddPicture UI**面板，等待用户点击屏幕，实例化预制对象，然后返回主模式。

该脚本开始时类似于我们在*第五章**中编写的`PlaceObjectMode`脚本，使用*AR 用户框架*。然后我们将增强它以确保框架中的图片对象与墙面平面对齐，面向房间，并垂直悬挂。

让我们编写`AddPictureMode`脚本，如下所示：

1.  首先，通过右键单击并选择`AddPictureMode`在你的项目的`Scripts/`文件夹中创建一个新的脚本。

1.  *双击*文件以打开它进行编辑。粘贴以下代码，它与您可能已经拥有的**PlaceObjectMode**脚本相同，但有差异突出显示。脚本的前半部分如下：

    ```cs
    using System.Collections.Generic;
    using UnityEngine;
    using UnityEngine.InputSystem;
    using UnityEngine.XR.ARFoundation;
    using UnityEngine.XR.ARSubsystems;
    public class AddPictureMode : MonoBehaviour
    {
        [SerializeField] ARRaycastManager raycaster;
        [SerializeField] GameObject placedPrefab;
        List<ARRaycastHit> hits = new List<ARRaycastHit>();
        void OnEnable()
        {
            UIController.ShowUI("AddPicture");
        }
    ```

1.  脚本的后半部分实际上与`PlaceObjectMode`脚本没有变化：

    ```cs
        public void OnPlaceObject(InputValue value)
        {
            Vector2 touchPosition = value.Get<Vector2>();
            PlaceObject(touchPosition);
        }
        void PlaceObject(Vector2 touchPosition)
        {
            if (raycaster.Raycast(touchPosition, hits,            TrackableType.PlaneWithinPolygon))
            {
                Pose hitPose = hits[0].pose;
                Instantiate(placedPrefab, hitPose.position,                hitPose.rotation);
                InteractionController.EnableMode("Main");
            }
        }
    }
    ```

在 `AddPictureMode` 的顶部，我们声明了一个 `placedPrefab` 变量，它将引用 `ARRaycastManager` 和一个私有的 `ARRaycaseHit hits` 列表，我们将在 `PlaceObject` 函数中使用它。

当模式启用时，我们显示 `AddPicture` UI 面板。然后，当有 `OnPlaceObject` 用户输入动作事件时，`PlaceObject` 对可追踪平面进行 `Raycast`。如果有碰撞，它将在场景中实例化一个 **FramedPhoto** 的副本，然后返回到主模式。

让我们先使用这个初始脚本，稍后修复我们发现的任何问题。下一步是将 AddPicture 模式添加到应用中。

## 创建 AddPicture 模式对象

我们现在可以通过在 **Interaction Controller** 下创建一个 **AddPicture Mode** 对象来将 AddPicture 模式添加到场景中，如下所示：

1.  在 `AddPicture Mode`。

1.  将 `AddPictureMode` 脚本从 **Project** 窗口拖动到 **AddPicture Mode** 对象上，将其添加为组件。

1.  将 **AR Session Origin** 对象从 **Hierarchy** 拖动到 **Add Picture Mode | Raycaster** 插槽。

1.  在 **Project** 窗口中定位你的 **FramedPhoto Prefab** 资产，并将其拖动到 **Add Picture Mode | Placed Prefab** 插槽。现在 **AddPicture Mode** 组件看起来如下（注意，此截图还显示了我们在本章末尾添加到脚本中的两个更多参数，**Image Data** 和 **Default Scale**）：![图 6.6 – 场景中添加了 AddPicture 模式    ](img/Figure_6.06-addpicturemode-insp.jpg)

    图 6.6 – 场景中添加了 AddPicture 模式

1.  现在我们将模式添加到 **Interaction Controller**。在 **Hierarchy** 中选择 **Interaction Controller** 对象。

1.  在 **Inspector** 中，在 **Interaction Controller** 组件的右下角点击 **+** 按钮向 **Interaction Modes** 字典添加一个项。

1.  在 **Id** 字段中输入 `AddPicture`。

1.  将 **AddPicture Mode** 游戏对象从 **Hierarchy** 拖动到 **Value** 插槽。交互控制器组件现在看起来如下：

![图 6.7 – 在交互模式字典中添加了 AddPicture 模式的交互控制器](img/Figure_6.07-interactioncont-addpicture.jpg)

图 6.7 – 在交互模式字典中添加了 AddPicture 模式的交互控制器

现在我们有一个 **Addpicture** 模式，当用户点击 **Add** 按钮时将从 **Main** 模式启用。现在让我们创建这个按钮。

## 创建主菜单添加按钮

当应用处于主模式时，**Main UI** 面板显示。在这个面板上，我们将有一个 **Add** 按钮供用户在想要在场景中放置新图片时按下。我将使用一个大的加号作为其图标，以下步骤：

1.  在 **Hierarchy** 窗口中，展开 **UI Canvas** 对象，然后展开其子 **Main UI** 对象。

1.  面板中的默认子文本是一个临时占位符；我们可以将其删除。*右键点击* 子 **Text** 对象并选择 **Delete**。

1.  现在我们添加一个按钮。*右键单击*`Add Button`。

1.  选择**添加按钮**，在其**检查器**窗口中使用**锚点菜单**（左上角）选择**底右**锚点。然后按*Shift + Alt +*点击**底右**以在该角落设置其**枢轴**和**位置**。

1.  调整按钮大小和位置，可以使用以下截图中的**矩形工具**（`175, 175`），和`-30, 30`），如下所示：![图 6.8 – 添加按钮矩形变换设置    ](img/Figure_6.08-Addbutton-recttransform-insp.jpg)

    图 6.8 – 添加按钮矩形变换设置

1.  在**层次结构**窗口中，通过点击其**三角形图标**展开**添加按钮**，并选择其子对象，**文本（TMP）**。

1.  设置其`+`并设置其`192`。

1.  您可以添加另一个文本元素来标记按钮。*右键单击*`Add`，`24`，`55`。

    我们现在按钮看起来如下：

    ![图 6.9 – 添加按钮    ](img/Figure_6.09-AddButton-view.jpg)

    图 6.9 – 添加按钮

1.  要设置按钮以启用**放置图片模式**，在**层次结构**中选择**添加按钮**。在其**检查器**中，在**按钮组件**的**OnClick**部分，按下底部的**+**按钮以添加事件动作。

1.  从**层次结构**中拖动**交互控制器**并将其放置在**OnClick 动作**的**对象**槽中。

1.  在**函数选择列表**中，选择**交互控制器 | 启用模式**。

1.  在其字符串参数字段中，输入文本`AddPicture`。

现在的**OnClick**属性看起来如下：

![图 6.10 – 当添加按钮被点击时，它调用 EnableMode("AddPicture")](img/Figure_6.10-Addbutton-onclick.jpg)

图 6.10 – 当点击添加按钮时，它调用 EnableMode("AddPicture")

我们现在已将**AddPicture 模式**添加到我们的框架中。当点击**添加**按钮时，交互控制器将启用它。启用后，脚本显示**AddPicture**指导 UI，然后等待**放置对象**输入动作事件。然后它使用射线投射来确定用户想在 3D 空间中的哪个位置放置对象，实例化预制体，然后返回主模式。让我们试试。

## 构建和运行

保存场景。如果您想尝试并查看其外观，现在可以**构建和运行**，如下所示：

1.  选择**文件 | 构建设置**。

1.  点击`ARGallery`)是否已在**场景在构建**列表中。

1.  确保在**场景在构建**列表中只选中了`ARGallery`场景。

1.  点击**构建和运行**以构建项目。

应用程序将启动并提示您扫描房间。慢慢移动您的设备以扫描房间，集中注意您想放置照片的墙壁大致区域。

什么因素有助于良好的平面检测？

当 AR 平面检测使用设备内置的白色光摄像头扫描 3D 环境时，它依赖于摄像头图像的良好视觉保真度。房间应该光线充足。被扫描的表面应该具有独特且随机的纹理，以帮助检测软件。例如，如果您的墙壁太光滑，我们的 AR 画廊项目可能难以检测垂直平面。（较新的设备可能包括其他传感器，例如基于激光的**LIDAR**深度传感器，这些传感器不受这些限制）。如果您的设备在检测垂直墙面平面时遇到困难，请尝试在墙上战略性地添加一些贴纸或其他标记，使表面对软件更具辨识度。

当至少检测到一个垂直平面时，扫描提示将消失，你将看到主 UI 的**添加**按钮。点击**添加**按钮将启用**添加图片模式**，显示带有点击放置说明图形的**添加图片**UI 面板。当你点击一个跟踪到的平面时，**FramedPhoto**预制件将在场景中实例化。以下是我的样子，在左侧：

![图 6.11 – 在墙上放置一个物体（左）和校正表面法线和垂直（右）

![图 6.11 – 在墙上放置一个物体（左）和校正表面法线和垂直（右）

图 6.11 – 在墙上放置一个物体（左）和校正表面法线和垂直（右）

哎呀！图片垂直地从墙上突出出来，如前一张截图（左侧所示）。我们希望它像右侧图像中的墙上的画一样悬挂。让我们更新脚本以处理这个问题。

## 完成 AddPictureMode 脚本

我们需要实施一些改进来完成`AddPictureMode`脚本的编写，包括以下内容：

+   将图片旋转，使其垂直并平贴在墙面平面上。

+   告诉图片在其框架中显示哪张图片。

+   当图片首次放置在墙上时，包括一个默认的缩放比例。

`AddPictureMode`脚本中包含以下设置旋转为`hitPose.rotation`的代码行：

```cs
Instantiate(placedPrefab, hitPose.position, hitPose.rotation);
```

如您在上一张截图中所见，跟踪平面的“向上”方向垂直于平面表面，因此，使用此代码图片看起来像是从墙上突出出来。对于水平平面，您希望物体站立在地板或桌子上，因此使用此默认向上方向实例化放置的物体是有意义的。但在本项目，我们不想这样做。我们希望图片与墙面朝向相同。我们希望它垂直向上/向下悬挂。

我们不应该使用`hit.pose.rotation`，而应该使用平面的法线向量（`pose.up`）来计算旋转。然后我们调用`Quaternion.LookRotation`函数来创建具有指定前进和向上方向的旋转（见[`docs.unity3d.com/ScriptReference/Quaternion.LookRotation.html`](https://docs.unity3d.com/ScriptReference/Quaternion.LookRotation.html)）。

四元数

四元数是一种可以用于表示计算机图形中旋转的数学结构。作为 Unity 开发者，你只需要知道 Transform 中的旋转使用`Quaternion`类。参见[`docs.unity3d.com/ScriptReference/Quaternion.html`](https://docs.unity3d.com/ScriptReference/Quaternion.html)。然而，如果你想要了解底层数学的解释，可以查看*3Blue1Brown*的精彩视频，例如在[`www.youtube.com/watch?v=zjMuIxRvygQ`](https://www.youtube.com/watch?v=zjMuIxRvygQ)上互动解释的*四元数和 3D 旋转*。

我们还需要的是能够告诉 FramedPhoto 显示哪张图片的能力。我们将为`imageInfo`添加一个公共变量，它将由 Image Select 菜单（在本章下一节开发）设置。

此外，我们还将添加一个`defaultScale`属性，在实例化图片时对其进行缩放。如果你还记得，我们定义的预制件是按 1 单位最大尺寸归一化的，这会使它在墙上宽 1 米，除非我们对其进行缩放。我们只缩放 X 和 Y 轴，将 Z 轴保留为`1.0`，这样框架的深度就不会被缩放太多。我将默认缩放设置为`0.5`，但你可以稍后在检查器中更改它。

按照以下方式修改`AddPictureMode`脚本：

1.  在类的顶部添加以下声明：

    ```cs
        public ImageInfo imageInfo;
        [SerializeField] float defaultScale = 0.5f;
    ```

1.  将`PlaceObject`函数替换为以下内容：

    ```cs
        void PlaceObject(Vector2 touchPosition)
        {
            if (raycaster.Raycast(touchPosition, hits,            TrackableType.PlaneWithinPolygon))
            {
                ARRaycastHit hit = hits[0];
                Vector3 position = hit.pose.position;
                Vector3 normal = -hit.pose.up;
                Quaternion rotation = Quaternion.LookRotation                 (normal, Vector3.up);
                GameObject spawned = Instantiate(placedPrefab,                position, rotation);
                FramedPhoto picture =                 spawned.GetComponent<FramedPhoto>();
                picture.SetImage(imageInfo);
                spawned.transform.localScale = new                Vector3(defaultScale, defaultScale, 1.0f);
                InteractionController.EnableMode("Main");
            }
        }
    ```

1.  保存脚本并返回 Unity。

注意，我必须取反墙面的法向量（`-hit.pose.up`），因为在创建我们的预制件时，按照惯例，图片是朝向负 Z 方向的。

当你放置一张图片时，它现在应该垂直悬挂并紧贴墙面，如本节顶部截图的右侧面板所示。

## 在添加图片模式下显示追踪平面

另一个可能的增强是，在主模式下隐藏追踪的平面，在`添加图片`模式下显示它们。这将使用户能够在没有这种干扰的情况下享受他们的图片库。查看我们在*第五章*的*在不需要时隐藏追踪对象*主题中是如何做到的，使用*AR 用户框架*。当时，我们编写了一个脚本`ShowTrackablesOnEnable`，现在我们也可以使用它。按照以下步骤操作：

1.  在**层次结构**中（在**交互控制器**下）选择**添加图片模式**游戏对象。

1.  在`ShowTrackablesOnEnable`脚本中，并将其拖动到**添加图片模式**对象上。

1.  从**层次结构**中，将**AR 会话原点**对象拖动到**检查器**中，并将其放置在**启用时显示可追踪对象 | 会话原点**槽位上。

这就是我们实现这个功能所需的所有内容。

回顾一下，我们配置了场景以检测和跟踪垂直平面，用于你的房间墙壁。然后我们创建了一个 `AddPictureMode` 脚本。当用户在垂直平面上轻触时，该脚本实例化一个 **FramedPhoto** 预制件的副本。然后我们通过确保图片在墙上平放且直立来改进脚本。该脚本还允许我们更改框架中的图像及其缩放。最后，我们在 **AddPicture** 模式下显示可跟踪的平面，并在返回到主模式时隐藏它们。

下一步是为用户提供在墙上挂新画之前选择图像的选择。我们现在可以继续创建一个图像选择菜单，供用户选择使用。

# 选择要使用的图像

我们接下来要做的事情是创建一个包含图像按钮的图像选择菜单，用户可以在将其添加到场景之前选择一张照片。当使用你已添加到项目中的 **Images** 列表（**图像数据** 游戏对象）构建菜单的 `ImageButtons` 脚本时。然后我们将 **SelectImage** 模式插入到 **AddPicture** 模式之前，这样选中的图像就是放置在墙上的图像。让我们首先定义 **SelectImage** 模式。

## 创建 SelectImage 模式

当用户启用 `SelectImage` 模式时，我们只需要显示 SelectImage UI 菜单面板，其中包含供用户选择的按钮。点击按钮将通过调用公共函数 `SetSelectedImage` 通知模式脚本，该函数反过来告诉 `AddPictureMode` 使用哪张图像。

创建一个名为 `SelectImageMode` 的新 C# 脚本，并按照以下内容编写：

```cs
using UnityEngine;
public class SelectImageMode : MonoBehaviour
{
    void OnEnable()
    {
        UIController.ShowUI("SelectImage");
    }
}
```

简单。当 `SelectImageMode` 被启用时，我们显示 **SelectImage UI** 面板（包含按钮菜单）。

现在，我们可以按照以下方式将其添加到 **Interaction Controller**：

1.  在 `SelectImage 模式` 中。

1.  将 `SelectImageMode` 脚本从 **Project** 窗口拖到 **SelectImage Mode** 对象上，将其添加为组件。

1.  现在，我们将此模式添加到 **Interaction Controller**。在 **Hierarchy** 中选择 **Interaction Controller** 对象。

1.  在 **Inspector** 中，在 **Interaction Controller** 组件的右下角，点击 **+** 按钮向 **Interaction Modes** 字典添加一个项。

1.  在 **Id** 字段中输入 `SelectImage`。

1.  将 **SelectImage Mode** 游戏对象从 **Hierarchy** 拖到 **Value** 槽。现在 **Interaction Controller** 组件看起来如下所示：

![图 6.12 – 添加了 SelectImage 模式的交互控制器](img/Figure_6.12_(new).jpg)

图 6.12 – 添加了 SelectImage 模式的交互控制器

接下来，我们将添加此模式的 UI。

## 创建 Select Image UI 面板

要创建 **SelectImage UI** 面板，我们将复制现有的 **Main UI** 并进行适配。该面板将包括一个标题和 **取消** 按钮。按照以下步骤操作：

1.  在 `SelectImage UI` 中。使用 *右键点击* **删除** 删除任何子对象，包括 **添加按钮**。

1.  将面板大小设置为略小于全屏，使其看起来像模态弹出窗口。在`50`处设置，将`150`留出空间用于应用标题。

1.  我们希望这个面板有一个实心背景，因此在其**图像**组件中，从主菜单中选择**组件 | UI | 图像**。

1.  创建一个菜单标题子面板。在`Header`。

1.  定位并拉伸`100`。

1.  右键点击**标题文本**。

1.  在`Select Image`，`48`，**对齐**设置为**居中**和**中间**，**锚点预设**设置为**拉伸-拉伸**。同时，*Alt + Shift +* 点击**拉伸-拉伸**。

1.  我们还将添加一个**取消按钮**。

1.  设置取消按钮的`80`和`-20`。还将其**图像 | 颜色**设置为浅灰色。

1.  对于`X`和`48`。

1.  在**层次结构**中选择**取消按钮**，在其**检查器**中点击**按钮 | OnClick**动作右下角的**+**按钮。

1.  将**交互控制器**游戏对象从**层次结构**拖放到**OnClick** **对象**槽中。

1.  从文本参数字段中的`Main`。现在，X 取消按钮将带您返回主模式。

**SelectImage** UI 面板的标题如下截图所示：

![图 6.13 – SelectImage UI 的标题面板]

![img/Figure_6.13-SelectImageHeader.jpg]

图 6.13 – SelectImage UI 的标题面板

接下来，我们将添加一个面板来包含将显示供用户选择的图片的图像按钮。这些按钮将按网格排列。使用以下步骤：

1.  在`Image Buttons`。

1.  在**图像按钮**面板上，取消选中其**图像**组件或删除它。我们不需要单独的背景。

1.  其`100`。

1.  选择`layout`并添加一个**网格布局组**组件。

1.  在`20, 20, 20, 20`处设置`200, 200`，并将`20, 20`设置为`20, 20`。将**子对齐**设置为**上居中**。

现在我们有一个带有标题和图像按钮容器的**ImageSelect UI**面板。当前层次结构的一部分如下截图所示：

![图 6.14 – 带有 SelectImage UI 的 UI 画布和具有 SelectImage 模式的交互控制器]

![img/Figure_6.14-uicanvas-selectimage-hier.jpg]

图 6.14 – 带有 SelectImage UI 的 UI 画布和具有 SelectImage 模式的交互控制器

最后，我们需要按照以下步骤将面板添加到 UI 控制器中：

1.  要将面板添加到 UI 控制器中，在**层次结构**中选择**UI Canvas**对象。

1.  在**检查器**中，在**UI 控制器**组件的右下角，点击**+**按钮向**UI 面板**字典中添加一个项目。

1.  在**Id**字段中输入`SelectImage`。

1.  将**SelectImage UI**游戏对象从**层次结构**拖放到**值**槽中。

现在我们有一个包含图像按钮容器的 UI 面板。为了制作按钮，我们将创建一个预制件，然后编写一个脚本以填充**图像按钮**面板。

## 创建图像按钮预制件

我们将定义一个**图像按钮**为预制件，以便它可以复制，为用户在选择菜单中提供的每个图像。创建按钮如下：

1.  在`Image Button`中。

1.  在**图像按钮**下，删除其子**文本**元素。

1.  在**图像按钮**上，移除其**图像**组件（使用*右键单击*和**移除组件**），然后按**添加组件**。搜索并添加一个**原始图像**组件。

1.  其**按钮**组件需要对其刚刚替换的图形的引用。在**检查器**中，将**原始图像**组件拖到**按钮 | 目标图形**槽中。

1.  现在将默认图像纹理，例如`WinterBarn`资产，从`Photos/`文件夹拖到**检查器**，并将其放置在**原始图像 | 纹理**槽中。

    UI 图像与原始图像

    **图像**组件用于其图形的图像精灵。**原始图像**组件用于其图形的纹理。精灵是用于 UI 和 2D 应用的小型、高效、预处理的图像。纹理通常更大，具有更多的像素深度和保真度，用于 3D 渲染和摄影图像。您可以使用图像文件的检查器属性在导入的图像之间以及其他类型之间进行更改。为了同时使用相同的照片资产（PNG 文件）作为 FramedPhoto 预制件和按钮，我们在按钮上使用了一个原始图像组件。

1.  让我们保存`Prefabs/`文件夹。这会创建一个预制资产，并将其在**层次结构**中的颜色改为蓝色，表示它是一个预制实例。

1.  在**层次结构**窗口中，*右键单击***图像按钮**对象，选择**复制**（或按*Ctrl/Option + D*键），并制作几个副本。因为按钮由具有**网格布局组**的**Image Buttons**面板作为父级，所以它们以网格形式渲染，如下面的截图所示：

![](img/Figure_6.15-SelectImageGrid.jpg)

图 6.15 – 带有网格布局的图像按钮的图像选择面板

接下来，我们将编写一个脚本，用我们想要使用的实际图像填充按钮。

## 编写 ImageButtons 脚本

`ImageButtons`脚本将是`ImageButtons`上的一个组件，打开它进行编辑，并按照以下方式编写：

```cs
using UnityEngine;
using UnityEngine.UI;
public class ImageButtons : MonoBehaviour
{
    [SerializeField] GameObject imageButtonPrefab;
    [SerializeField] ImagesData imagesData;
    [SerializeField] AddPictureMode addPicture;
    void Start()
    {
        for (int i = transform.childCount - 1; i >= 0; i--)
        {
            GameObject.Destroy(                transform.GetChild(i).gameObject);
        }
        foreach (ImageInfo image in imagesData.images)
        {
            GameObject obj =                 Instantiate(imageButtonPrefab,transform);
            RawImage rawimage = obj.GetComponent<RawImage>();
            rawimage.texture = image.texture;
            Button button = obj.GetComponent<Button>();
            button.onClick.AddListener(() => OnClick(image));
        }
    }
    void OnClick(ImageInfo image)
    {
        addPicture.imageInfo = image;
        InteractionController.EnableMode("AddPicture");
    } 
}
```

让我们遍历这个脚本。在类的顶部，我们声明了三个变量。`imageButtonPrefab`将是一个对`imagesData`对象的引用，该对象包含我们的图像列表。`addPicture`是对每个按钮的`AddPictureMode`的引用，以告诉哪个图像已被选中。

`Start()`首先做的事情是清除按钮面板中的任何子对象。例如，我们创建了许多按钮的副本以帮助我们开发和可视化面板，当它运行时，除非我们首先删除它们，否则它们仍然会出现在场景中。

然后，`Start`遍历每个图像，并为每个图像创建一个`onClick`事件。

当其中一个按钮被点击时，我们的`OnClick`函数将被调用，并将该按钮的`image`作为参数传递。我们将此`image`数据传递给`AddPictureMode`，该模式将在**AddPictureMode**实例化新的**FramedPhoto**对象时使用。

按照以下方式将脚本添加到场景中：

1.  在**层次结构**中，选择**图像按钮**对象（位于**UI Canvas/选择图片面板**）。

1.  将**ImageButtons**脚本拖动到**Image Buttons**对象上，使其成为一个组件。

1.  从**项目**窗口中，将**Image Button**预制拖动到**检查器**中，并将其放置在**Image Buttons | Image Button Prefab**槽位上。

1.  从**层次结构**中，将**AddPicture Mode**对象拖动到**检查器**中，并将其放置在**Image Buttons | Add Picture**槽位上。

1.  同样从**层次结构**中，将**Images Data**对象拖动到**Image Buttons | Images Data**槽位上。

**Image Buttons**组件现在看起来如下截图所示：

![Figure 6.16 – Image buttons panel with the ImageButtons script that builds the menu at runtime]

![Figure 6.16-imagebuttons-insp.jpg]

图 6.16 – 带有在运行时构建菜单的 ImageButtons 脚本的图像按钮面板

好的。当应用启动时，`AddPictureMode`选择了哪张图片，然后启用了添加图片模式。

## 重定向添加按钮

在我们尝试之前，还有最后一步。目前，主菜单的添加按钮直接启用 AddPicture 模式。我们需要将其更改为调用**SelectImage**，如下所示：

1.  在**层次结构**中，选择**添加**按钮（位于**UI Canvas** / **主 UI**）。

1.  在**检查器**中，在**按钮 | 点击**动作列表中，将**EnableMode**参数从**AddPicture**更改为**SelectImage**。

1.  保存场景。

如果你把这些都做对了，你应该能够**构建并运行**场景，并运行完整的场景：按下**添加**按钮将显示一个**选择图片**菜单。点击一个图片，选择面板将被替换为提示点击以将图片及其框架放置在墙上的提示。以下是我手机上的截图显示了左边的**选择图片**菜单。选择图片并放置在墙上后，结果在右边显示。然后应用返回主菜单：

![Figure 6.17 – Pressing the Add button, I see an image menu (left), and the result after placing (right)]

![Figure 6.17-PlaceImageRun.jpg]

图 6.17 – 按下添加按钮，我看到一个图像菜单（左），放置后的结果（右）

总结一下，在本节中，我们添加了`ImageButtons`脚本，为应用中要包含的每个图片实例化按钮。点击其中一个按钮会将所选图片数据传递给**AddPicture**模式。当用户点击放置并实例化**FramedPhoto**时，我们将图片设置为用户所选的图片。我们还菜单中包含了一个**取消**按钮，以便用户可以取消添加操作。

目前看起来还不错。我们有一个问题是所有图片都在相同大小的横幅框架中渲染，因此可能看起来扭曲。让我们修复这个问题。

# 调整图像宽高比

目前，我们正在忽略图像的实际大小，并将所有图像都调整到 3:4 的宽高比，使其适应横幅方向。幸运的是，我们在`ImageInfo`中包含了图像的实际（原始）像素尺寸。我们现在可以使用它来相应地缩放图片。我们可以将这个更改应用到位于**FramedPhoto**预制件上的`FramedPhoto`脚本中。

计算宽高比的算法可以作为一个实用函数在`ImagesData`脚本中分离。打开`ImagesData`脚本并添加以下代码：

```cs
    public static Vector2 AspectRatio(float width, float         height)
    {
        Vector2 scale = Vector2.one;
        if (width == 0 || height == 0) 
            return scale;
        if (width > height)
        {
            scale.x = 1f;
            scale.y = height / width;
        }
        else
        {
            scale.x = width / height;
            scale.y = 1f;
        }
        return scale;
    }
```

当`width`大于`height`时，图片是横幅，我们将保持 X 缩放为`1.0`并缩小 Y。当`height`大于`width`时，它是肖像，我们将保持 Y 缩放为`1.0`并缩小 X。如果它们相同或为零，我们返回`(1,1)`。该函数被声明为`static`，因此可以使用`ImagesData`类名来调用它。

打开`FramedPhoto`脚本进行编辑，并对以下内容进行修改：

```cs
    public void SetImage(ImageData image)
    {
        imageData = image;
        Renderer renderer =             imageObject.GetComponent<Renderer>();
        Material material = renderer.material;
        material.SetTexture("_BaseMap", imageData.texture);
        AdjustScale();
    }
    public void AdjustScale()
    {
        Vector2 scale = ImagesData.AspectRatio(imageInfo.width,            imageInfo.height);
        scalerObject.localScale = new Vector3(scale.x, scale.y,            1f);
    }
```

如果你记得，`SetImage`函数在`ImageData.AspectRatio`之后立即由`AddPictureMode`调用，以获取新的局部缩放并更新`scalerObject`变换。

当图片不是正方形时，你可能注意到框架的宽度在水平方向和垂直方向上略有不同。解决这个问题需要额外调整`1.0 – 0.01/aspectratio`。我将把这个实现的细节留给你。

当你再次运行项目并在墙上放置图片时，它将根据你选择的图片具有正确的宽高比。你可以添加的一个改进是调整**选择图片面板**按钮上的图像大小，使它们也不会被压缩。我将这个练习留给你。

# 摘要

在本章的开头，我给你提供了这个增强现实画廊项目的需求和计划，包括项目目标、用例、用户体验设计和用户故事。你开始使用在*第四章**，创建 AR 用户框架*中创建的**ARFramework**模板进行实现，并在此基础上实现将带框照片放置在墙上等新功能。

为了实现这个功能，你创建了一个你创建的`ImageButton`预制件。点击图片后，你会被提示在 AR 追踪的墙上轻触，然后在该墙上放置一个新的带框照片，并正确缩放以适应图像的宽高比。

我们现在有一个有趣项目的良好开端。还有很多工作可以完成。例如，目前图片可以重叠放置，这将是错误的。此外，能够移动、调整大小和删除图片会更好。我们将在下一章中添加这个功能。
