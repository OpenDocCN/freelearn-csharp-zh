# 图片拼图 - AR 体验

在本章中，我们将创建另一个基于 AR 的应用程序。这次，重点将是一个可以用于教育的谜题，用于教授语言或词汇识别。这样做的原因是，基于 AR 的应用程序和游戏也是非常有前景的灵感来源和目标受众。

本章将向您介绍以下内容：

+   如何更新现有的 Unity 安装以添加 Vuforia 支持

+   Unity Hub

+   如何为 Windows、Android 和 iOS 构建基于教育的 AR 应用程序

让我们直接深入这个项目的背景，以及为什么它与 AR 相关。

# 项目背景

就像任何其他项目一样，最好从想法开始。当我第一次想到这个项目时，我想展示 AR 应用程序和游戏也能反映教育。我教育过孩子们英语，深知学习新语言的挫败感。

游戏和应用开发也应该教会用户一些东西；它不一定是历史、数学、语言、科学或地理；它也可以是某些无害的东西，比如反应训练或手眼协调。作为开发者，我们在世界上有一个独特的位置，能够以有趣的方式融入学习，而不会让用户觉得这是他们“必须”做的事情。

这并不意味着我们必须深思熟虑，试图将其纳入我们的应用程序和游戏中；它可能，并且通常就是这样，只是自然而然发生的事情。然而，在这个项目中，我特别针对学习方面，以展示它如何轻松地融入 AR 项目。

# 项目概述

本项目基于这样一个想法：通过为孩子们创建一个真正简单的现实世界谜题，让他们去解决，然后他们可以用这个应用程序来检查，看看他们是否解决了这个谜题，以此来教授孩子们词汇联想和拼写。

本项目的构建时间最多为 15 分钟。

# 开始学习

这里是 Unity 版本 2018.1.5 的系统要求：

+   **发布日期**：2018 年 6 月 15 日

+   **操作系统**：Windows 7 SP1+、8、10

+   **GPU**：具有 DX9（着色器模型 3.0）或具有 9.3 功能级别的 DX11 的显卡。

查看以下内容：

[`unity3d.com/`](https://unity3d.com/) [`www.turbosquid.com/FullPreview/Index.cfm/ID/967597`](https://www.turbosquid.com/FullPreview/Index.cfm/ID/967597)

# 安装 Vuforia

我知道我们在第一章，“什么是 AR 以及如何设置”中讨论过这个问题，但为了以防那一章被快速浏览或者 Unity 已经更新，这里需要简要回顾一下。

要在 macOS 和 Windows 上安装 Vuforia，步骤相当简单；然而，我想向您展示获取 Vuforia 软件和 Unity 的不同方法。

Unity 还有一种可以安装的类型，称为 Unity Hub，您可以从 Unity 网站获取，而不是 2018 安装程序文件。Unity Hub 的作用是允许您在单个位置拥有多个 Unity 安装，这是一种设置您首选 Unity 编辑器的方法，将您的项目合并到单个启动器中，轻松更新您想要安装的组件，并且它还提供了对项目预设类型模板的访问。按照以下步骤操作：

1.  导航到 Unity 网站并点击获取 Unity：

![图片](img/d56bdd76-c0b5-43d2-a604-4dd1fa3189c6.png)

1.  您将看到一个选项，可以选择个人、高级或专业。如果您适合并需要个人版本，请点击尝试个人：

![图片](img/5f358e12-4595-4217-a8ba-ff96bb7dc5fc.png)

1.  这将带您到下载页面，在那里您需要勾选复选框以接受条款，并给您提供下载 Unity 应用程序本身或预览中的 Unity Hub 的选项。我们想要 Unity Hub：

![图片](img/fd05337d-356d-44a2-919f-fd8dfc81f0be.png)

1.  一旦您下载并安装了 Unity Hub，您就可以打开它，它将为您提供项目、学习或安装的选项。点击安装以查看可用的版本：

![图片](img/77f3f0f2-2179-443d-97c9-8f4e3fdf60c2.png)

1.  我们想安装 Unity 的最新版本，即 2018.1.5f1：

![图片](img/e60be635-e06b-48ee-adf6-9baef4f71c5a.png)

1.  一旦您点击安装您想要的版本，点击您想要的组件，然后按完成安装 Unity 编辑器：

![图片](img/ae3ef697-0ba8-4958-9367-8a640d25e1d9.png)

现在，假设您在安装 Unity 时忘记选择在此步骤中安装 Vuforia——没问题；您可以跳过前面的点，从这里继续操作。

1.  在 Unity 中打开一个项目；这可能是一个您不想要的虚拟项目，或者它可能是本章项目的起点。

1.  点击文件菜单中打开的构建菜单：

![图片](img/84168b0c-dad3-4241-924a-31906b5831e1.png)

1.  我们想从窗口中选择玩家设置：

![图片](img/de9fed39-b21a-49bc-807a-cfd8f0b3b6c3.png)

1.  应该会打开一个窗口，其中包含通常的检查器面板。找到 XR 支持安装程序并点击 Vuforia 增强现实：

![图片](img/b01ee587-9251-46b8-bad8-b35e4165391e.png)

1.  这将打开您的浏览器并要求您下载一个文件。点击保存以下载文件：

![图片](img/6ca7cee8-b9a4-4b33-be7c-8be1cf7e1b4e.png)

1.  关闭 Unity 编辑器并安装此文件。这将添加 Vuforia 支持到您的 Unity 安装中，而无需重新安装整个编辑器。

现在完成这些后，我们可以在 Windows 中创建此项目。

# macOS 和 Windows 设置之间的差异

在为各自平台构建之前，macOS 和 Windows 的基本设置之间几乎没有区别。我已经以完全相同的方式设置了项目，以便更容易地遵循流程。如果您同时拥有 macOS 和 Windows 计算机，那么当您进入 macOS 部分时，可以跳到构建部分。如果您只有 Windows，那么您只需遵循那里的说明。相反，如果您只有 macOS 设备，那么您将拥有完整的说明，并且可以跳过*Windows 项目设置*部分。

# Windows 项目设置

如果你还记得，在第一章，*什么是增强现实以及如何设置*，我们创建了我们的 Vuforia 开发者账户。我们将需要它，因为我们将会使用 Vuforia 来创建这个项目。导航到 Vuforia 开发者门户并登录您的账户。现在按照以下步骤操作：

1.  在 Vuforia 开发者门户中，点击开发，并确保子菜单已选择许可证管理器。我们需要创建一个新的开发许可证密钥，应用名称为`Chapter5`或`Picture Puzzle`：

![图片](img/c74c4481-28bc-4bf1-9793-d65ec230d1e5.png)

1.  在创建新密钥后，您应该看到许可证管理器显示了我们所创建的`VuforiaIntro`和`Chapter5`密钥：

![图片](img/8b1f8190-22f0-48b1-bd80-b3ef1c6e354c.png)

1.  点击`Chapter5`以访问您的许可证密钥。您应该将其复制并粘贴到记事本或 Notepad++中以备后用：

![图片](img/815ebbdd-4097-422b-a52a-669c1713c5bb.png)

1.  点击目标管理器，因为我们将要为这个项目创建自己的图像目标：

![图片](img/c4fc0ae4-05fe-4450-b5da-ec8802cc5444.png)

1.  点击添加数据库以创建我们将要在项目中使用的全新数据库：

![图片](img/67ba6f93-6a6e-4c0b-9c8d-7ab0a7dedb77.png)

1.  您可以给数据库起任何名字；在我的情况下，我将称之为`Words_Pictures`，类型为设备，然后点击创建：

![图片](img/61463265-b0f0-44b1-8ba7-21d42cf2e96f.png)

1.  应该带我们回到目标管理器页面，并展示我们的新`Words_Pictures`数据库：

![图片](img/d0b7c3d4-7264-40c8-a9d9-07495a8ad41d.png)

1.  点击`Words_Pictures`以访问数据库，然后当您看到它时点击添加目标：

![图片](img/8e04376b-c1e3-44ea-80b6-ae913ea325eb.png)

1.  现在，我们将能够向数据库添加一个新的目标。在这种情况下，我们想要一个单独的图像。

我强烈建议您创建并使用 JPG 格式，因为 PNG 格式需要一个 8 位灰度图像或 24 位 RGB。

1.  宽度应设置为与您的图像相同的宽度。名称应反映图像的内容：

![图片](img/b843fb05-8cff-4db9-baf9-fdcd4df203a2.png)

1.  打开 Microsoft Paint，我们可以开始创建我们将用于目标的文件：

![图片](img/fce1968c-2568-47b0-83c9-f46558772c75.png)

1.  下一步是找到我们想要用于这张图片的确切大小，我认为最好的方法是知道我们将会使用什么图片。在我们的例子中，它将是 72 磅的字体大小，字体名为 Bastion，并显示单词`TREE`：

![图片](img/6c5cb071-b0e4-4139-a1c6-5823d400becb.png)

1.  调整比例，使其接近单词的边缘：

![图片](img/689332e2-d1d4-4acc-a20c-9f02a82fa676.png)

1.  将文件保存为 JPG 格式，命名为`Tree`：

![图片](img/08ecc3ad-3071-42c0-a36d-c1adbdc2f596.png)

1.  如果你查看画图工具菜单的底部，它会告诉你我们刚刚创建的文件的尺寸。在这种情况下是`253`x`106`，这是我们想要的大小。

1.  导航回添加目标网页，并选择我们刚刚创建的`Tree`文件：

![图片](img/1d5e99ec-d1f0-4935-9eb0-90ef583eb2fd.png)

1.  将宽度设置为`253`，名称设置为`Tree`，然后点击添加：

![图片](img/1d5e99ec-d1f0-4935-9eb0-90ef583eb2fd.png)

1.  这将带你回到数据库，你应该看到名为`TreeWord`的条目，名称为`Tree`。类型应该是单张图片，并且应该有一个三星评级：

![图片](img/0490fd03-a50f-4075-98c3-8d4285073169.png)

1.  评级系统旨在告诉你它是否是正确的尺寸，以便你的 AR 设备能够正确读取。我们目前有一个三星评级，这意味着它应该很好，但可以更好。我们能做什么来修复这个问题？我们可以放大图片。

1.  让我们删除数据库中的图片。为此，点击`Tree`旁边的复选框，上面将出现一个非常小的删除按钮：

![图片](img/2c562614-37d6-49e9-9634-af6ed636c38f.png)

1.  让我们回到画图工具并调整图片大小。`680`x`480`应该很完美，尽管`500`x`300`也可以：

![图片](img/78cc4d08-2bf3-4af9-ac62-dd0af6d2a1d6.png)

1.  上传新的目标，结果应该有五星评级：

![图片](img/55d0bb90-b1fa-4efd-aac4-4fb471cf715a.png)

1.  点击下载数据库（全部）：

![图片](img/c993ddd6-88f4-4a39-b363-f78d8a391795.png)

1.  这将打开一个新窗口，询问我们想要使用哪个开发平台。我们想要 Unity 编辑器。点击下载：

![图片](img/71373071-bc66-4a4f-a61b-c3eff0e6ebf1.png)

它将下载一个 Unity 文件，我们需要将其导入到我们的 Unity 项目中——现在，我们可以在 Unity 中开始工作，而无需离开编辑器去做其他工作。打开 Unity，让我们开始构建我们的项目。

# 构建 Windows 项目

创建一个新的 Unity 项目，如果你还没有的话，一开始就命名为`Chapter5`。然后加载项目。现在按照以下步骤操作：

1.  我们下载的`Words_Pictures`文件现在需要被定位并导入到项目中：

![图片](img/5a77e1d9-5dbd-4cc1-8d4b-b0592aa0c3b4.png)

1.  在我们深入创建项目之前，让我们看看导入时创建的文件夹。

1.  我们的主要`Assets`文件夹现在将有一个`Editor`文件夹、一个`StreamingAssets`文件夹和一个`Scenes`文件夹：

![图片](img/3b031415-43c9-49c7-a931-47284def4280.png)

1.  在`Editor`文件夹内，将有一个名为`Vuforia`的文件夹：

![图片](img/f4820476-6775-4b9a-af72-21385e2130db.png)

1.  在`Vuforia`文件夹内将有一个名为`ImageTargetTextures`的文件夹：

![图片](img/dbfa803e-f638-480f-955b-5dbc2f523e02.png)

1.  在`ImageTargetTextures`文件夹内，将有一个名为`Word_Pictures`的文件夹：

![图片](img/c8e9aa66-20fc-47a2-905a-9fd873e19dbb.png)

1.  `Word_Pictures`文件夹将包含我们的树图像精灵：

![图片](img/948eae16-3df0-4409-9e08-cf9362087adb.png)

1.  返回主`Assets`文件夹，让我们看一下，从`StreamingAssets`开始：

![图片](img/cdbdc6d4-7a32-4a79-a8dd-547e6d502f6b.png)

1.  在`StreamingAssets`文件夹内将有一个`Vuforia`文件夹：

![图片](img/7b3d7488-0b1d-42dc-a315-9f8bfc3b2305.png)

1.  在`Vuforia`文件夹内将有两个文件：`Words_Pictures.xml`和`Words_Pictures.dat`文件：

![图片](img/6be7b771-8680-4aa4-90d0-da609216b419.png)

1.  让我们深入查看 XML 文件，看看里面具体有什么。看看这段代码：

```cs
<?xml version="1.0" encoding="UTF-8"?>
<QCARConfig  xsi:noNamespaceSchemaLocation="qcar_config.xsd">
  <Tracking>
    <ImageTarget name="Tree" size="680.000000 480\. 000000" />
  </Tracking>
</QCARConfig>
```

XML 文件已经设置了默认的架构，主标签是`QCarconfig`。

下一个标签，包含我们的图像，是`ImageTarget`。它有名称，我们设置为`Tree`，以及用浮点值表示的大小。

XML 文件非常简短且直接。这个文件专门用于存放 Vuforia 需要知道的数据，我们使用图像的大小，以及如果有多张图像时能够引用正确的文件。继续按照步骤操作：

1.  前往 TurboSquid 网站下载我们将使用的免费`Tree`模型：

![图片](img/983301e8-c4eb-4cf7-b74c-aaee822b30c1.png)

1.  你需要`Tree_FBX`和`Tree_textures`文件来完成下一部分：

![图片](img/bfc2aa59-317d-4179-8d26-2c32a5561d90.png)

1.  返回主`Assets`文件夹，创建一个名为`Models`的新文件夹：

![图片](img/33deaca5-b967-482a-9a55-9ba7a07ba239.png)

1.  提取树模型和纹理。将模型和纹理复制粘贴到 Unity 内部的`Models`文件夹中：

![图片](img/9efec29f-32b6-4a7d-8da1-93ecfde74565.png)

1.  从层次面板中删除标准相机。

1.  右键点击层次面板，导航到 Vuforia；点击添加 AR 相机：

![图片](img/60a9508d-bd98-47c7-a8f4-a29e2cc8db5b.png)

1.  在层次面板中点击`AR Camera`。转到检查器面板，点击打开 Vuforia 配置：

![图片](img/5cc8ab46-04fc-40a8-8e62-77c8258c6e19.png)

1.  Unity 会要求你导入和下载更多 Vuforia 项目，并接受 Vuforia 许可：

![图片](img/a45a9911-35aa-4042-875e-a07615db248c.png)

1.  将你的应用许可密钥复制粘贴到应用许可密钥部分：

![图片](img/bbf29b64-9b9e-49fe-b0d2-2f742ddd3176.png)

1.  右键点击层次面板，创建一个名为`ImageTarget`的空游戏对象。

1.  右键点击`ImageTarget`对象，高亮显示 Vuforia 并点击图像：

![图片](img/83bf9bb7-de32-46e2-9030-28c8e303a750.png)

1.  点击图像对象，查看检查器面板。图像目标行为应设置为预定义类型；数据库应为 Words_Pictures，图像目标应为 Tree：

![图片](img/520ad372-4451-465d-a950-26fcb33ab741.png)

现在，我们需要添加我们的模型。我假设您知道如何为模型设置材质，所以这里不会详细说明。

1.  将模型拖放到场景中。将*x*、*y*、*z*位置设置为`0`,`0`,`0`，比例设置为*x*、*y*、*z*坐标的`0.09`。最后一步是将它设置为 ImageTarget 对象内图像的子对象：

![图片](img/778f75ac-f3ae-4d6b-9ca0-a72c87096bb8.png)

1.  打印出`Tree`文本，并将纸张剪成四条条带。

1.  通过点击“文件”|“构建设置”，为 PC、Mac 和 Linux Standalone 或 Android 构建项目，然后点击“构建”：

![图片](img/e2bdeef2-3c1e-465c-81ce-4920996b3223.png)

现在，使用您的 Android 设备或 PC 摄像头，将条带按正确顺序拼接在一起，Tree 模型将出现在纸张上方。

# macOS 项目设置

在 Mac 上设置项目的步骤几乎相同，主要区别在于我们将使用什么软件来创建文本文件，但无论如何，这里应该简要说明，因为预计 Mac 用户不会想阅读本章的 Windows 部分。我们将使用 Vuforia 来创建此项目。导航到 Vuforia 开发者门户并登录您的账户。现在按照以下步骤操作：

1.  在 Vuforia 开发者门户中，点击“开发”，并确保子菜单已选择许可证管理器。我们需要为名为`Chapter5`的应用程序创建一个新的开发许可证密钥：

![图片](img/235bf106-3383-45c9-a33e-f539e7f32fd1.png)

1.  在创建新密钥后，您应该会看到许可证管理器显示了创建的`VuforiaIntro`和`Chapter5`密钥：

![图片](img/f87c2016-4e45-4131-a412-df718aba3174.png)

1.  点击`Chapter5`以获取您的许可证密钥。您应该将其复制并粘贴到记事本或 Notepad++中以备后用：

![图片](img/002e56b9-d1d0-4e51-9f47-960b5d36248d.png)

1.  点击目标管理器，因为我们将为这个项目创建自己的图像目标：

![图片](img/edf79e22-64df-486d-a61a-77dbf637921f.png)

1.  点击“添加数据库”以创建我们将要在项目中使用的全新数据库：

![图片](img/9f97ea57-5eb6-4e6a-8b10-460168c5acd3.png)

1.  您可以命名数据库为任何您想要的名称；在我的情况下，我将称之为`Words_Pictures`，类型为设备，然后点击创建：

![图片](img/37d1beff-c9a0-49ba-8cb5-e42186a4c60f.png)

1.  这应该会带我们回到目标管理器页面，并展示我们的新 Words_Pictures 数据库：

![图片](img/06f98cd7-9b23-420e-ac19-0dcf95aceac6.png)

1.  点击`Words_Pictures`以访问数据库，然后当您看到它时点击“添加目标”：

![图片](img/982b606f-cf1f-45c9-bce9-0902a82c78b0.png)

1.  现在，我们能够向数据库添加一个新的目标。在这种情况下，我们想要一个单独的图像。

我强烈建议您创建并使用 JPG 格式，因为 PNG 格式需要一个 8 位灰度图像或 24 位 RGB 图像。

1.  宽度应设置为与您的图片相同的宽度。名称应反映图片的内容：

![图片](img/5078ac27-1d7f-4036-b052-cd867fc4c16f.png)

1.  打开 Microsoft Paint，我们可以开始创建用于目标的文件：

![图片](img/17417755-b3cc-4573-98a6-a8a72a36b625.png)

1.  下一步是找到我们想要用于此图片的确切尺寸，我认为最好的方法是知道我们将要使用的图片。在我们的案例中，它将是 72 磅的字体大小，字体名为 Bastion，并显示单词`Tree`：

![图片](img/2f2e4012-f3dd-447b-aaff-11cd45f6d3ef.png)

1.  将比例调整得接近单词边缘：

![图片](img/2a8ac2df-d3a7-4e80-9e86-b93d29cae620.png)

1.  将文件保存为 JPG 格式，并命名为`Tree`：

![图片](img/88f031d3-ca21-4352-b838-00481bbeb993.png)

1.  如果您查看 Paint 菜单的底部，它会告诉您我们刚刚创建的文件尺寸。在这种情况下是`253`x`106`。

1.  返回到添加目标网页，并选择我们刚刚创建的`Tree`文件：

![图片](img/863d7564-14f1-4ca9-bc5a-a48b09af9fc3.png)

1.  设置宽度为`253`，名称为 Tree，然后点击添加：

![图片](img/863d7564-14f1-4ca9-bc5a-a48b09af9fc3.png)

1.  这将带您回到数据库，您应该看到单词`Tree`，名称为`Tree`，类型为单张图片，并且应该有一个三星级评分：

![图片](img/afa42e6b-e4f5-4a7a-9684-d7baf3f07f83.png)

评分系统旨在告诉您它是否具有正确的尺寸，以便您的 AR 设备正确读取。我们目前有一个三星级评分，这意味着它应该很好，但可以更好。我们能做什么来修复它？我们可以放大图片。让我们继续按照步骤操作：

1.  让我们从数据库中删除这张图片。为此，点击`Tree`旁边的复选框，然后在其上方会出现一个非常小的删除按钮：

![图片](img/c3c66c4e-b994-4987-ad15-324e9c30589e.png)

1.  让我们回到 Paint 并调整图片大小。`680`x`480`应该很完美：

![图片](img/0f5ee378-2d1c-447c-85a6-86a680a5ad37.png)

1.  上传新的目标，结果应该有一个五星级评分：

![图片](img/86f71c1c-077e-4ec0-828e-0dbaff79ff37.png)

1.  点击下载数据库（全部）：

![图片](img/44b7ce8c-aeb0-4627-af34-2f2f5df66648.png)

1.  这将打开一个新窗口，询问我们想要使用哪个开发平台。我们想要 Unity 编辑器。点击下载：

![图片](img/7b77e799-7098-4cfd-aa05-54d3bb315f51.png)

它将下载一个 Unity 文件，我们需要将其导入到我们的 Unity 项目中——现在，我们可以在 Unity 中开始工作，而无需离开编辑器去做其他工作。打开 Unity，让我们开始构建我们的项目。

# 构建 macOS 项目

如果您还没有创建，请创建一个新的 Unity 项目，并首先将其命名为`Chapter5`。然后，加载项目。现在按照以下步骤操作：

1.  我们下载的 `Words_Pictures` 文件现在需要定位并导入到项目中。

1.  在我们深入创建项目之前，让我们看看导入时创建的文件夹。我们的主要 `Assets` 文件夹现在将有一个 `Editor` 文件夹，一个 `StreamingAssets` 文件夹，和一个 `Scenes` 文件夹：

![图片](img/14a9fad5-ec6e-439b-b67e-03ff8b6b7254.png)

1.  在 `Editor` 文件夹内，将有一个名为 `Vuforia` 的文件夹：

![图片](img/98f6ad30-657b-4e4d-ac04-a514ac8c2ed5.png)

1.  在 `Vuforia` 文件夹内将有一个名为 `ImageTargetTextures` 的文件夹：

![图片](img/36c9a025-5d0b-4e88-834d-dc5f4d261845.png)

1.  在 `ImageTargetTextures` 文件夹内，将有一个名为 `Word_Pictures` 的文件夹：

![图片](img/01663648-a53c-4aef-8b4d-91359cc5fc1d.png)

1.  在 `Word_Pictures` 文件夹内，我们将有我们的树图像精灵：

![图片](img/799822db-4d21-40c8-a127-0f53831fd17c.png)

1.  返回主 `Assets` 文件夹，让我们看看，从 `StreamingAssets` 开始：

![图片](img/7ae43cc6-2af5-47e3-a66a-f554864e99e9.png)

1.  在 `StreamingAssets` 文件夹内将有一个 `Vuforia` 文件夹：

![图片](img/3aa4f151-ae0d-40a4-a62f-32af2f44be7a.png)

1.  在 `Vuforia` 文件夹内将有两个文件：`Words_Pictures.xml` 和 `Words_Pictures.dat`：

![图片](img/5cc6ca96-1007-4946-8ee5-fdf21759a592.png)

1.  让我们深入查看 XML 文件，看看里面具体有什么内容：

```cs
<?xml version="1.0" encoding="UTF-8"?>
<QCARConfig  xsi:noNamespaceSchemaLocation="qcar_config.xsd">
  <Tracking>
    <ImageTarget name="Tree" size="680.000000 480\. 000000" />
  </Tracking>
</QCARConfig>
```

XML 文件已经设置了默认的架构，主标签是 `QCarconfig`。

下一个标签，包含我们的图像，是 `ImageTarget`。它有名称，我们设置为 `Tree`，以及用浮点值表示的大小。

XML 文件非常简短且直接。此文件专门用于存放 Vuforia 需要知道的数据，我们使用的图像大小，以及如果我们要使用多个图像时能够引用正确的文件。让我们继续下一步：

1.  前往 TurboSquid 网站，下载我们将使用的免费 `Tree` 模型：

![图片](img/6166745b-3ba2-4286-97a6-c5ba5ee73eae.png)

1.  你将需要 `Tree_FBX` 和 `Tree_textures` 文件来完成下一部分：

![图片](img/263c0be5-77cd-4259-ab41-048f095fba1d.png)

1.  返回主 `Assets` 文件夹，创建一个名为 `Models` 的新文件夹：

![图片](img/344d75c6-c0a6-4ac5-85f7-97d525d2c460.png)

1.  提取树模型和纹理。将模型和纹理复制粘贴到 Unity 中的 `Models` 文件夹内：

![图片](img/c4bc6e6d-a84c-498f-b5cf-90fb71efe1a4.png)

1.  从层次结构面板中删除标准相机。

1.  在层次结构面板中右键点击，导航到 Vuforia；点击添加 AR Camera：

![图片](img/580e8155-af48-492b-9978-0aa6796d7a26.png)

1.  在层次结构面板中点击 `AR Camera`。切换到检查器面板，并点击打开 Vuforia 配置：

![图片](img/66428e17-697c-4658-bb79-d54d35dd4206.png)

1.  Unity 应该会要求你导入和下载更多 Vuforia 项目，并接受 Vuforia 许可协议：

![图片](img/369a9404-00ce-4102-b063-8c1cf65b5a04.png)

1.  将你的应用程序许可证密钥复制并粘贴到应用程序许可证密钥部分：

![](img/7be65a34-3925-4c80-a000-74f20083da5f.png)

1.  在层次结构面板上右键单击，创建一个名为 `ImageTarget` 的空游戏对象。

1.  在 `ImageTarget` 对象上右键单击，高亮显示 Vuforia，然后点击 Image：

![](img/e6e24f45-47fd-4d1e-85a2-29c4c96a10fa.png)

1.  点击 `Image` 对象，查看检查器面板。Image Target Behavior 的类型应该是 Predefined。数据库应该是 Words_Pictures，Image Target 应该是 Tree：

![](img/1f2b6110-134b-459a-8820-a2336c9652ea.png)

现在我们需要添加我们的模型。我假设你知道如何为模型设置材质，所以这里不会详细说明。让我们继续下一步：

1.  将模型拖放到场景中。将 *x*、*y* 和 *z* 位置设置为 `0`,`0`,`0`，并将比例设置为 *x*、*y* 和 *z* 坐标的 `0.09`。最后一步是将它设置为 ImageTarget 对象内图像的子对象：

![](img/18621823-5654-43e7-b537-addba37110f8.png)

1.  打印出 `Tree` 文本，并将纸张剪成四条带。

1.  通过点击 File | Build Settings 来为 iOS 构建项目。确保选中 Development Build 复选框：

![](img/61770937-9ead-471a-8750-72fb5a35a030.png)

# 使用 Xcode

我们可以导航到应用程序的 `Build` 文件夹，并点击它以打开我们的 XCode 项目。按照以下步骤操作：

1.  在屏幕的左侧，你应该看到 `Unity-iPhone` 是你可以选择的项目之一。点击它，你应该在中心看到 Unity-iPhone，在右侧看到 Identity 和 Type：

![](img/7d8a3149-50b2-4a36-9ee8-5e4b72e994b3.png)

1.  确认 Identity 是否正确。我的显示名称是 `Chapter5`，包标识符为 `com.rpstudios.Packtbook`：

![](img/13f0a40d-107f-4a06-aded-a26fc8dbbc81.png)

1.  现在，在签名设置中，你需要确保自动管理签名的复选框被勾选，并且 Team 中附有你的电子邮件地址：

![](img/fa853d05-8fe4-4767-8676-b0a3134e9cf6.png)

1.  滚动并查找 Linked Frameworks and Libraries。AVFoundation 应该从 Optional 设置为 Required。我注意到，当它设置为 Optional 时，链接器无法正常工作：

![](img/fe4e8e82-cc4f-4581-8fda-5cf039c3c78a.png)

1.  定位到 Architectures，因为我们需要从默认的设置更改为 Standard。这是由于存在不同的架构，iOS 不再使用 ARM：

![](img/c735919c-3270-4d42-a88b-a4643fdb174c.png)

1.  现在你可以点击 Build，并将你的 iPhone 6 或更高版本连接到你的 macOS 计算机。构建并在设备上运行它。它将要求你在手机上信任该应用程序，所以按照指示进行信任设置。

1.  点击手机上的应用程序，——哇！——它将加载，你可以玩这个应用程序。

现在利用你的 iOS 设备，将条带按正确的顺序放在一起，Tree 模型将出现在纸张的上方。

# 摘要

在本章中，我们学习了如何为儿童创建教育游戏，使他们能够了解与他们相关的单词所指的对象。我们学习了如何使用 Vuforia 进行开发，适用于 macOS 和 Windows，以及 Android 和 iOS 设备。我们还了解到，在 Windows 和 macOS 设备上构建的基本构建块在代码上相当相似，唯一的重大区别是编译到 iOS 或 macOS 所需的额外步骤，使用 XCode。

在下一章中，我们将构建一个健身应用的原型，该应用允许用户随机选择他们想要散步的位置，以增添乐趣。

# 问题

1.  要为 Unity 安装 Vuforia，您必须访问 Vuforia 网站下载 SDK：

A.) 正确

B.) 错误

1.  您需要为 Unity 2017 版本安装旧版插件：

A.) 正确

B.) 错误

1.  Unity Hub 使得安装多个 Unity 版本变得容易：

A.) 正确

B.) 错误

1.  Microsoft Paint 可以创建 Vuforia 图像目标所需的 PNG 和 JPG 文件：

A.) 正确

B.) 错误

1.  Vuforia 图像目标可以接受 TIFF 文件格式：

A.) 正确

B.) 错误

1.  Vuforia 对 PNG 和 JPG 文件的文件大小限制为 5MB：

A.) 正确

B.) 错误

1.  Vuforia 在 macOS 上不可用：

A.) 正确

B.) 错误

1.  Unity Hub 在 macOS 和 Windows 上可用：

A.) 正确

B.) 错误

1.  Vuforia 数据库中的星级评分系统是用来衡量图像质量的：

A.) 正确

B.) 错误

1.  使用 Unity 时，您不需要 Vuforia 许可证密钥：

A.) 正确

B.) 错误
