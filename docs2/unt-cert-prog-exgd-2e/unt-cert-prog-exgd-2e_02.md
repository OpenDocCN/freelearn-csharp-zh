# *第二章*：添加和操作对象

在上一章中，我们讨论了官方 Unity 程序员考试的重要性以及它可以为任何寻求确保自己或他人理解 Unity 编程的开发者带来的好处。我们还讨论了成为程序员的一般构建块以及我们游戏的设计概要。

作为在游戏引擎上工作的程序员，你可能会为多个行业工作。在这些行业中，你可能会收到一份技术概要/文档（嗯，你应该会！）用于构建应用程序。在这个项目中，我们正在制作一个游戏，游戏设计概要实际上是制作这个游戏的蓝图。在本章中，我们将根据概要和游戏框架的指导，应用我们的大部分代码、游戏对象、预制体等。我们将在本章中提醒自己概要和游戏框架，并将具体信息转移到我们的代码中。

关于我们的代码，我们将讨论接口和可脚本化对象的重要性，以帮助结构化和统一我们的代码，防止其无必要地膨胀，这一点我们在*第一章*，“设置和结构化我们的项目”，基于 SOLID 原则中已经讨论过。我们还将熟悉 Unity 编辑器，并了解游戏对象、预制体以及导入三维模型进行动画制作。

在本章中，我们将讨论以下主题：

+   设置我们的 Unity 项目

+   介绍我们的接口（`IActorTemplate`）

+   介绍我们的`ScriptableObject`（`SOActorModel`）

+   设置我们的`Player`、`PlayerSpawner`和`PlayerBullet`脚本

+   规划和创建我们的敌人

+   设置我们的`EnemySpawner`和敌人脚本

下一个部分将概述本章涵盖的考试目标。

# 本章涵盖的核心考试技能

*编程核心交互*：

+   实现和配置游戏对象行为和物理。

+   实现和配置输入和控制。

+   实现和配置摄像机视图和移动。

*在艺术管道中工作*：

+   理解光照并编写与 Unity 光照 API 交互的脚本。

+   理解二维和三维动画，并编写与 Unity 动画 API 交互的脚本。

*场景和环境设计编程*：

+   确定实现游戏对象实例化、销毁和管理的方法。

+   展示对开发者测试及其对软件开发过程影响的了解，包括 Unity Profiler 和传统的调试和测试技术。

+   认识到结构化脚本的模块化、可读性和可重用性技术。

# 技术要求

本章的项目内容可以在[`github.com/PacktPublishing/Unity-Certified-Programmer-Exam-Guide-Second-Edition/tree/main/Chapter_02`](https://github.com/PacktPublishing/Unity-Certified-Programmer-Exam-Guide-Second-Edition/tree/main/Chapter_02)找到。

您可以下载整个章节的项目文件[`github.com/PacktPublishing/-Unity-Certified-Programmer-Exam-Guide-Second-Edition`](https://github.com/PacktPublishing/-Unity-Certified-Programmer-Exam-Guide-Second-Edition)。

本章的所有内容都存储在相关的 `unitypackage` 文件中，包括包含我们在本章中将要执行的所有工作的 `Complete` 文件夹，所以如果您在任何时候需要一些参考资料或额外指导，请务必查看。

查看以下视频，了解 *Code in Action*：[`bit.ly/3yfWyt5`](https://bit.ly/3yfWyt5)。

# 设置我们的 Unity 项目

如果我们不正确地管理文件，将它们放置在分配的文件夹中，项目可能会很快变得混乱。如果您想以自己的方式组织文件夹，或者在本书中，您决定偏离我的做法，那也是可以的。只是当涉及到查找和组织文件时，尽量意识到您未来的自己或其他在这个项目上工作的人。

如果您还没有打开项目，请创建以下文件夹：

+   `Model` 包含 3D 模型（玩家飞船、敌人、子弹等）。

+   `Prefab` 存储游戏对象的实例（这些是在 Unity 中创建的）。

+   `Scene` 存储我们的第一级场景以及其他级别。

+   `Script` 包含我们所有的代码。

+   `Material` 存储我们的游戏对象材质。

+   `Resources` 存储要加载到游戏中的资源和对象。

+   `ScriptableObject` 是能够存储大量数据的数据容器。

    小贴士

    您应该了解预制件是什么，因为它是使 Unity 迅速易用的主要部分之一。然而，如果您不知道：它通常是存储在实例中的具有其设置和组件的游戏对象。您可以通过从 **Hierarchy** 窗口中拖动游戏对象到 **Project** 窗口中，将游戏对象作为预制件存储在您的 **Project** 窗口中。游戏对象名称之后将生成一个蓝色框图标，如果您在 **Project** 窗口中选择预制件，其 **Inspector** 窗口将显示所有存储的值。如果您想了解更多关于预制件的信息，可以查看[`docs.unity3d.com/Manual/Prefabs.html`](https://docs.unity3d.com/Manual/Prefabs.html)上的文档。

以下截图显示了如何创建这些文件夹：

![Figure 2.1 – 在 Unity 编辑器中创建文件夹

![img/Figure_2.01_B18381.jpg]

图 2.1 – 在 Unity 编辑器中创建文件夹

接下来，我们将创建子文件夹；我们需要执行以下操作：

1.  在我们的 `Prefab` 文件夹中，创建另外两个文件夹，`Enemies` 和 `Player`：

![Figure 2.2 – 在 Unity 编辑器中创建的文件夹

![img/Figure_2.02_B18381.jpg]

图 2.2 – Unity 编辑器中创建的文件夹

`Resources`是一个 Unity 识别的特殊文件夹。它将允许我们在游戏运行时加载资源。有关`Resources`文件夹的更多信息，请查看[`docs.unity3d.com/Manual/BestPracticeUnderstandingPerformanceInUnity6.html`](https://docs.unity3d.com/Manual/BestPracticeUnderstandingPerformanceInUnity6.html)上的文档。

更多信息

在这里提一下`StreamingAssets`文件夹。尽管我们在这个项目中没有使用它，但它很好地说明了它与`Resources`文件夹的相似之处（以及不同之处）。

`Resources`文件夹导入资源并将它们转换为与目标平台兼容的内部格式。例如，当项目构建时，PNG 文件将被转换和压缩。

`StreamingAssets`文件夹将保存 PNG 文件，不会对其进行转换或压缩。有关 Streaming Assets 的更多信息，请参阅以下链接：[`docs.unity3d.com/Manual/StreamingAssets.html`](https://docs.unity3d.com/Manual/StreamingAssets.html)。

在**技术要求**部分提供了本章 GitHub 仓库的下载链接。下载后，双击`Chapter2.unitypackage`文件，我们将得到一个资产列表，这些资产将被导入到我们的 Unity 项目中：

+   `Player_ship.fbx`

+   `enemy_wave.fbx`

以下截图显示了我们将要导入到项目中的资产的**导入**窗口：

![图 2.3 – 将资源导入到你的项目中](img/Figure_2.03_B18381.jpg)

图 2.3 – 将资源导入到你的项目中

确保所有资源都被勾选，然后点击窗口右下角的**导入**按钮。我们现在可以继续到下一节，在**项目**窗口中组织我们的文件和文件夹。

## 创建预制件

在本节中，我们将创建三个预制件：玩家、玩家的子弹和敌人。这些预制件将包含我们游戏中的组件、设置和其他属性值，我们可以在整个游戏中实例化它们。

让我们从将我们的`player_ship.fbx`文件制作成预制件实例开始，操作如下。

有时，在导入任何三维文件时，它可能包含我们可能不需要的额外数据。例如，我们的`player_ship`模型附带其自身的材质和动画属性。我们不需要这些，所以让我们在将模型完全导入 Unity 项目之前移除这些属性。

要移除`player_ship`模型，我们需要执行以下操作：

1.  在`Assets/Model`中，选择`player_ship`文件。

1.  在**检查器**窗口中，选择**材质**按钮。

1.  确保从下拉列表中将**材质创建模式**设置为**无**，然后点击**应用**按钮。

1.  现在，点击**材质**按钮旁边的**动画**按钮。

1.  取消勾选**导入动画**复选框，然后点击**应用**按钮。

1.  在**动画**按钮旁边选择**绑定**按钮。

1.  在**动画类型**下拉菜单中选择当前值，然后选择**无**，接着点击**应用**按钮。

1.  这就是`player_ship`模型的全部内容。

    重要信息

    在整本书中，每次我们选择一个三维模型时，确保运行相同的流程，因为我们不会需要导入额外的元素，就像我们刚才移除的那些。这意味着我现在希望您重复我们刚刚对`enemy_wave.fbx`模型所进行的流程。

让我们继续为我们的游戏准备`player_ship`模型：

1.  从`Assets/Model`中点击并拖动`player_ship`到**层次结构**窗口。

1.  在“玩家飞船”中选中`player_ship`

1.  在所有轴向上与`1`不同`0`

下面的截图显示了**检查器**窗口中的`player_ship`值：

![图 2.4 – 玩家飞船值在检查器窗口中![图 2.4 – 创建预制体对话框](img/Figure_2.04_B18381.jpg)

图 2.4 – 检查器窗口中的玩家飞船值

1.  从`Assets/Prefab/Player`文件夹中点击并拖动`player_ship`。

在创建预制体时，有时您可能会被问及这是**原始**还是**变体**：

![图 2.5 – 创建预制体对话框](img/Figure_2.05_B18381.jpg)

图 2.5 – 创建预制体对话框

变体预制体将是原始预制体的副本，但也将携带从原始预制体中做出的任何更改。例如，如果原始预制体是一辆有 4 个轮子的车，那么变体也将有相同的。如果原始预制体将其数量从 4 改为 3，变体将复制原始预制体。

注意，**层次结构**窗口中的`player_ship`已经变成蓝色，这意味着它已经变成了一个预制体。

1.  从**层次结构**窗口中删除`player_ship`。

我们将使用类似的过程来创建我们的`enemy_wave`预制体，但我们也需要创建它自己的名称标签，因为目前还没有**敌人**标签...

### 敌人预制体和自定义标签

在本节中，我们将创建一个`enemy_wave`预制体以及一个自定义标签。该标签将用于识别和分类所有相关敌人。

要创建一个`enemy_wave`预制体和自定义名称标签，请按照以下说明操作：

1.  将`Assets/Model`中的`enemy_wave.fbx`文件拖入**层次结构**窗口。

1.  在`enemy_wave`中选中`enemy_wave`文件。

1.  在所有轴向上与`1`不同`0`：

![图 2.6 – 玩家飞船值在检查器窗口中](img/Figure_2.06_B18381.jpg)

图 2.6 – 检查器窗口中的敌人波值

现在，让我们通过以下步骤为`enemy_wave`游戏对象创建一个新的标签：

1.  在**检查器**窗口中选择**未标记**参数。

1.  从**标签**下拉菜单中选择**添加标签...**。

1.  **检查器**窗口现在将显示**标签和层**窗口。

1.  点击**+**以添加一个新的标签，如图下截图所示。

1.  在弹出窗口中输入`Enemy`，如图下截图所示，然后点击**保存**按钮：

![图 2.7 – 玩家飞船值在检查器窗口中](img/Figure_2.07_B18381.jpg)

图 2.7 – 向标签和层列表添加标签

1.  返回到`enemy_wave`游戏对象以恢复我们的**检查器**窗口详细信息。

1.  再次单击**未标记**参数。

1.  我们现在可以在下拉列表中看到**敌人**，因此选择它。

1.  将`enemy_wave`游戏对象从`Assets/Prefab/Enemies`拖动。

1.  从**层次结构**窗口中删除`enemy_wave`

我们现在继续进行第三个预制体创建 – 玩家的子弹。但这次，我们不会导入三维模型 – 我们将在 Unity 编辑器中创建一个，然后在下一节中将其创建为预制体。

### 创建玩家的子弹预制体

接下来，我们将在 Unity 编辑器中创建玩家子弹的视觉效果。我们将制作一个蓝色球体并为其添加一个环绕光源。让我们先创建一个三维球体游戏对象。

在**层次结构**窗口中右键单击，从下拉列表中选择**3D 对象** | **球体**。

在**层次结构**窗口中仍然选中新创建的`Sphere`，在**检查器**窗口中进行以下更改：

1.  将游戏对象名称从`Sphere`更改为`player_bullet`。

1.  将**标签**从**未标记**更改为**玩家**。标签名称使得在章节的后续部分更容易识别。

1.  除了所有轴上的`2`之外，`0`。

以下截图显示了所有三个更改：

![图 2.8 – 检查器窗口中的 player_bullet 值](img/Figure_2.08_B18381.jpg)

图 2.8 – 检查器窗口中的 player_bullet 值

接下来，我们将给`player_bullet`游戏对象添加一个新的蓝色材质。

#### 创建并应用玩家的子弹材质

在本节中，我们将创建一个简单的无光照材质，由于材质的简单性，它不会占用设备太多性能。要创建基本材质并将其应用到`player_bullet`对象上，请执行以下操作：

1.  在`Assets/Material`文件夹中。

1.  在`Material`文件夹内，创建一个新的文件夹（与我们在*设置我们的 Unity 项目*部分中所做的方式相同）并命名为`Player`。这样，任何与玩家相关的材质都可以存储在其中。

1.  双击新创建的`Player`文件夹，然后在**项目**窗口中（在窗口右侧的空白区域）再次右键单击，从下拉列表中选择**创建** | **材质**。

1.  将创建一个新的材质文件。将其重命名为`player_bullet`。

1.  选择`player_bullet`材质，并在**检查器**窗口中，将材质从**标准**着色器更改为**不发光** | **颜色**，按照以下截图中的三个步骤操作：

![图 2.9 – 创建不发光颜色材质](img/Figure_2.09_B18381.jpg)

图 2.9 – 创建不发光颜色材质

**检查器**窗口将移除大部分属性并将材质简化，以便在任何设备上更容易执行。

1.  仍然在`0`，`190`，`255`和`255`。

我们已经创建并校准了玩家的子弹，现在我们可以通过以下步骤将材质应用到`player_bullet`预制件上：

1.  在以下位置选择`player_bullet`预制件：`Assets/Prefab/Player`。

1.  在下拉列表中选择`player_bullet`，直到你看到材质，然后选择它。

以下截图显示了`player_bullet`预制件的**网格渲染器**组件更新到我们新的无光照材质：

![Figure 2.10 – player_bullet 现在具有 player_bulletMat 材质![img/Figure_2.10_B18381.jpg](img/Figure_2.10_B18381.jpg)

图 2.10 – `player_bullet`现在具有`player_bulletMat`材质

在*第四章**，应用艺术、动画和粒子中，我们将回到材料和艺术的一般讨论，如果你对此感兴趣，这将值得关注。我们还将玩转粒子系统，创建一队星星飞掠过玩家的飞船。

我们将添加到玩家子弹的最后一个组件是一个环绕灯光，给子弹一个能量发光的效果。

#### 向玩家的子弹添加灯光

在本节中，我们将向玩家的子弹添加一个灯光组件，以隐藏我们只是在发射球体的印象。它还将介绍 Unity 的点光源，它充当发光球体。

要添加和自定义一个球状灯光到玩家的子弹，我们需要做以下操作：

1.  在`Assets/Prefab/Player`文件夹中，选择`player_bullet`预制件，并将其拖入**层次结构**窗口（如果它还没有在层次结构窗口中）。

1.  在组件列表底部的**检查器**中，点击**添加组件**按钮，从下拉列表中选择**灯光**。

`player_bullet`预制件现在将附加一个**灯光**组件。我们只需更改三个属性值，使灯光更适合游戏对象。

1.  在`player_bullet`文件的`50`中更改以下属性值

1.  `0`, `190`, `255`, 和 `255`

1.  `20`

以下截图显示了更新值后的**灯光**组件：

![Figure 2.11 – 检查器窗口中的灯光组件值![img/Figure_2.11_B18381.jpg](img/Figure_2.11_B18381.jpg)

图 2.11 – 检查器窗口中的灯光组件值

在进入下一节之前，因为我们已经对一个现有的预制件添加了材质和灯光组件，我们需要点击**覆盖**按钮以确认新的更改。

以下截图显示了`player_bullet`预制件：

![Figure 2.12 – 更新 player_bullet 预制件![img/Figure_2.12_B18381.jpg](img/Figure_2.12_B18381.jpg)

图 2.12 – 更新 player_bullet 预制件

1.  最后，点击**层次结构**中的`player_bullet`。

在下一节中，我们将继续通过应用 Unity 自己的物理系统，即**刚体**组件，来更新我们的三个预制件，以帮助检测碰撞。

## 添加刚体组件和修复游戏对象

因为这个游戏涉及到与游戏对象的碰撞，我们需要对玩家、玩家的子弹和敌人应用碰撞检测。Unity 提供了一系列不同的形状来围绕游戏对象，使其作为一个不可见的盾牌；我们可以设置我们的代码以响应与盾牌接触。

在我们将碰撞器添加到玩家和敌人游戏对象之前（**Sphere**游戏对象自动带有碰撞器），我们需要添加一个名为**Rigidbody**的 Unity 组件。如果一个游戏对象将要与至少一个其他游戏对象发生碰撞，它需要一个**Rigidbody**组件，该组件可以影响游戏对象的质量、重力、阻力、约束等。如果您想了解更多关于**Rigidbody**组件的信息，请查看[`docs.unity3d.com/Manual/class-Rigidbody.html`](https://docs.unity3d.com/Manual/class-Rigidbody.html)上的文档。

Rigidbody 联结

Unity 除了碰撞器之外还有其他物理类型。**关节**也需要**Rigidbody**系统，并且它们有不同的形式，如**铰链**、**弹簧**等。

这些**关节**将在一个固定点进行模拟；例如，**铰链****关节**非常适合使门在门铰链的旋转点来回摆动。

如果您想了解更多关于**关节**的信息，请查看[`docs.unity3d.com/Manual/Joints.html`](https://docs.unity3d.com/Manual/Joints.html)上的文档。

让我们一次性添加`player_ship`和`player_bullet`预制件：

1.  在**项目**窗口中，导航到**Prefab | Player**。

1.  按*Ctrl*（在 Mac 上为*command*）并点击`player_ship`和`player_bullet`文件。

1.  在**检查器**窗口中，点击**添加组件**按钮。

1.  在下拉菜单中，输入`Rigidbody`。

1.  选择**Rigidbody**（不是**Rigidbody 2D**）。

1.  **Rigidbody**组件现在已经被分配给了我们的两个游戏对象。

1.  在**检查器**窗口中仍然选择两个游戏对象，在**Rigidbody**下，确保**重力**复选框没有被勾选。如果勾选了，我们的游戏对象在游戏进行时会开始沉入场景。

现在，我们可以将碰撞器添加到我们的`player_ship`和`enemy_wave`游戏对象中（我们的`player_bullet`已经有一个**SphereCollider**）。我们将为我们的游戏对象添加一个**SphereCollider**，因为它相对于性能成本来说是最便宜的碰撞器：

1.  将`player_ship`预制件从`Assets/Prefab/Player`拖动到**层次结构**窗口中。

1.  在下拉菜单中的`Sphere Collider`下仍然选择`player_ship`。

1.  一旦你看到`player_ship`游戏对象。

你会注意到在`player_ship`的碰撞器中有一个绿色的线框围绕在`player_ship`周围，这将用来检测击中。它可能对于击中框的用途来说太大，所以让我们减小其大小。

1.  在**检查器**窗口中仍然选择`player_ship`预制件，在`0.3`处，如图所示：

![图 2.13 – 添加到`player_ship`预制体的触发球体碰撞器](img/Figure_2.13_B18381.jpg)

图 2.13 – 添加到`player_ship`预制体的触发球体碰撞器

1.  此外，当我们仍然选择`player_ship`预制体时，检查`player_ship`预制体中是否有另一个碰撞器，而不会引起任何形式的潜在物理碰撞。

1.  在**Inspector**窗口的右上角点击**Override**，然后点击**Apply All**以更新我们对预制体的**Rigidbody**和**SphereCollider**组件所做的修改。

1.  我们现在可以在**Hierarchy**窗口中选择`player_ship`预制体，然后在键盘上按*Delete*键，因为我们不再需要在**Scene**中使用它。

我们现在需要将同样的方法应用到`player_bullet`上：

1.  将`player_bullet`预制体从`Assets /Prefab/Player`拖放到**Hierarchy**窗口中。

1.  在**Inspector**窗口的**SphereCollider**组件中勾选**Is Trigger**框，并调整**Radius**。

1.  点击`player_bullet`更改，并从**Hierarchy**窗口中删除`player_bullet`预制体。

我们需要更新的最后一个游戏对象是`enemy_wave`预制体。我们已经用`player_ship`和`player_bullet`预制体覆盖了步骤，所以完全重复指令并不理想；然而，我们需要做以下事情：

1.  简而言之，我想让你将`enemy_wave`预制体从**Project**窗口中`Assets/Prefab/Enemies`的位置拖放到**Hierarchy**窗口中..

1.  在**Inspector**窗口中添加一个`enemy_wave`预制体。

1.  调整`enemy_wave`预制体的正确比例，就像我们调整`player_ship`一样。

1.  `enemy_wave`预制体不需要**Rigidbody**组件，因为它将与自身包含一个组件的相关游戏对象发生碰撞。

1.  最后，从**Hierarchy**窗口中删除`enemy_wave`预制体。

使用以下截图作为前面简短说明的参考，如果你卡住了，请使用本节中讨论的先前步骤：

![图 2.14 – 添加并缩放到`enemy_wave`预制体的触发碰撞器](img/Figure_2.14_B18381.jpg)

图 2.14 – 添加并缩放到`enemy_wave`预制体的触发碰撞器

希望这对你来说进展顺利。如果你在任何地方卡住了，请参考包含所有完成文件的[`github.com/PacktPublishing/Unity-Certified-Programmer-Exam-Guide-Second-Edition/blob/main/Chapter_02/ProjectFIles/Chapter-02-Complete.unitypackage`](https://github.com/PacktPublishing/Unity-Certified-Programmer-Exam-Guide-Second-Edition/blob/main/Chapter_02/ProjectFIles/Chapter-02-Complete.unitypackage)文件夹，检查它们并进行比较。

在继续之前，请注意，如果一个游戏对象是粉红色的，例如我们之前截图中的`enemy_wave`对象，这仅仅意味着它没有附加材质。在其他情况下，这也可能意味着附加到材质上的着色器有问题。

我们可以通过以下方式修复这个粉红色问题：

1.  在 `Assets/Prefab/Enemies`。

1.  将 enemy_wave 拖放到层次窗口中。展开层次窗口中 `enemy_wave` 旁边的下拉菜单。

1.  选择第一个游戏对象，标题为 `enemy_wave_core`。

1.  在 **检查器** 窗口中，选择 **Mesh Renderer** 组件中 **元素 0** 参数旁边的小 **远程** 圆圈（在下述截图中用 **1** 表示），然后从下拉列表中选择 **默认材质**（在下述截图中用 **2** 表示），如图所示：

![图 2.15 – 将默认材质添加到 enemy_wave_core 游戏对象](img/Figure_2.15_B18381.jpg)

图 2.15 – 将默认材质添加到 enemy_wave_core 游戏对象

1.  对于其兄弟游戏对象 `enemy_wave_ring`，遵循相同的步骤。

`enemy_wave` 对象现在将应用默认材质。如果对预制体进行了任何更改，请确保点击 **覆盖，应用全部**。

属性

如果一个游戏对象需要像 `Rigidbody` 这样的组件，我们可以在类名上方放置一个实际上是对脚本的一个提醒，表明游戏对象需要它：

`[需要组件(typeof(Rigidbody))]`

如果游戏对象没有组件，脚本将创建一个，如果我们尝试移除 `Rigidbody` 组件，我们将在 Unity 编辑器中收到一条消息，表明这是一个必需的组件。

这段代码并不是必需的，更多的是一种在组件中的一般良好实践。

如果你想要了解更多关于 `RequireComponent` 属性的信息，请查看 [`docs.unity3d.com/ScriptReference/RequireComponent.html`](https://docs.unity3d.com/ScriptReference/RequireComponent.html) 的文档。

因此，现在我们已经将碰撞体和 **Rigidbody** 组件应用到我们的游戏对象上。这使我们能够在碰撞体相互接触时创建反应。

由于我们开始构建我们的项目，让我们快速讨论保存我们的场景、项目等。

## 保存和发布我们的工作

容易陷入我们的项目，但作为简短的提醒，尽可能频繁地保存你的工作。这样，如果发生任何不好的事情，你总是可以回滚。

由于我们已经从前一章创建了并保存了 `testLevel` 场景，我们还可以将此场景添加到 **构建设置** 窗口中。这样做的原因是让 Unity 知道我们想在项目中包含哪些场景。在将游戏打包为部署的构建时，这也是一个要求。

要将我们的场景添加到 **构建设置**，请执行以下操作：

1.  在 Unity 编辑器顶部，点击 **文件 | 构建设置**。将出现 **构建设置** 窗口。

1.  点击 `testLevel` 场景。

1.  以下截图显示了 `testLevel` 场景。当我们稍后添加更多场景时，每个场景都将被编号：

![图 2.16 – 将 testLevel 场景添加到构建场景列表中](img/Figure_2.16_B18381.jpg)

图 2.16 – 将测试级别场景添加到构建场景列表中

1.  关闭 **Build Settings** 窗口。我们将在下一章添加更多场景时回到这里。

1.  通过点击 **文件 | 保存项目** 来保存项目是一个好习惯。

现在我们继续在 Unity 编辑器中设置场景相机。

## Unity 编辑器布局

对于我们的横向卷轴射击游戏 *Killer Wave*，我们需要控制相机来显示场景的宽高比和可见深度，并确保我们展示了正确数量的游戏环境。

让我们开始并决定我们游戏的屏幕比例。我们将创建自己的分辨率，这在大多数平台上将是相当常见的。

要将 **Game** 窗口的屏幕比例更改为自定义比例，请执行以下操作：

1.  在 **Game** 窗口选项卡下点击当前宽高比并选择 **+** 符号。

1.  输入以下截图所示的自定义宽高比值。

1.  点击我们刚刚创建的 `1080` 分辨率：

![图 2.18 – 设置自定义游戏窗口分辨率](img/Figure_2.18_B18381.jpg)

图 2.17 – 设置自定义游戏窗口分辨率

了解我们需要使我们的游戏艺术作品支持尽可能多的屏幕比例（或者给它扩展到）的需求是很好的，尤其是如果我们想为平板电脑或手机等便携式设备制作游戏的话。这是因为几乎每个主要品牌的手机和平板电脑都有不同的比例尺寸，我们不希望开始挤压和压缩我们的内容，因为它看起来不会正确。也有可能我们的小型移动游戏会取得成功，并且以后可以被移植到游戏机或 PC 上。如果是这样的话，我们需要让游戏屏幕也支持这些比例。从所有这些中我们可以得出的主要观点是我们正在针对我们的游戏覆盖所有可能的常见屏幕比例。我们可以覆盖的平台（游戏机、便携式设备等）越多，灵活的屏幕比例，就越容易将我们的游戏扩展到那些设备上，而无需额外的工作。我们在*第八章*，“添加自定义字体和 UI”，和*第九章*，“创建 2D 商店界面和游戏内 HUD”，中解释了更多关于屏幕尺寸比例的内容，其中我们讨论了 UI 显示设置。此外，在*第十三章*，“效果、测试、性能和替代控制”，中我们将解释如何在一个原始图像组件上显示我们的游戏屏幕。

在我们继续我们的项目之前，确认我们对 Unity 自身 UI 布局的了解可能是一个好时机。以下截图显示了 Unity 编辑器，我在其中勾勒并标注了相关的窗口：

![图 2.19 – Unity 编辑器窗口布局](img/Figure_2.19_B18381.jpg)

图 2.18 – Unity 编辑器窗口布局

通常，Unity 编辑器窗口由五个主要窗口组成：

+   **场景**：这是我们二维/三维的工作空间。

+   **游戏**：这是最终用户将看到的窗口。默认情况下，**游戏**选项卡与**场景**窗口共享相同的空间。

+   **层次结构**：我们场景中的所有游戏对象都将列在这里。

+   **检查器**：当选择一个对象时，其信息将在这里显示。

+   **项目**：这是我们 Unity 项目文件夹。将其视为我们可以用于游戏的文件和文件夹的结构。

    小贴士

    要单独拖动每个窗口，左键点击并拖动标签名称，然后它会自动定位到不同的位置。

我的**游戏**窗口设置为**1080**，因为我没有第二个屏幕的奢侈，所以我点击了它的名称标签（**游戏**）并将其拉到右下角。窗口很小，但正如您在**游戏**窗口顶部所看到的，缩放设置为 1x，这意味着我有一个完整的画面；没有任何东西被隐藏或从视野中裁剪掉。

要检查我们是否具有主摄像机的`0`。我们还可以按照以下方式重置**变换**选项：

1.  在**层次结构**窗口中选择主摄像机后，在**检查器**窗口中，点击**变换**面板右上角的三点，如图下截图所示：

![图 2.20 – 变换设置齿轮位置![图片](img/Figure_2.20_B18381.jpg)

图 2.19 – 变换设置齿轮位置

1.  当下拉菜单出现时，点击**重置**。

继续设置我们的主摄像机，让我们通过更改其**背景**设置来从**场景**/**游戏**窗口中移除景观背景：

1.  在**层次结构**窗口中点击**主摄像机**。

1.  在**检查器**窗口中，我们有**摄像机**组件，其属性名为**清除标志**。点击**天空盒**值。

1.  将出现一个下拉菜单。点击**纯色**，如图下截图所示：

![图 2.21 – 将背景更改为纯色![图片](img/Figure_2.21_B18381.jpg)

图 2.20 – 将背景更改为纯色

1.  我们现在将看到一个蓝色背景，这会减少干扰。

1.  如果您不喜欢蓝色，您可以将它更改为`0`，`0`，`0`和`255`中的任何颜色，如图下截图所示：

![图 2.22 – 设置背景颜色值![图片](img/Figure_2.22_B18381.jpg)

图 2.21 – 设置背景颜色值

太好了，现在让我们继续为主摄像机编写这些属性。

### 通过脚本更新摄像机属性

我们现在已经在我们的**场景**中设置了主摄像机的行为。接下来，我们需要将这段代码编写到一个脚本中，这样每当场景被加载时，Unity 都会读取脚本并理解主摄像机应该如何设置。

再次观察我们的框架，让我们看看摄像机脚本应该放置在哪里：

![图 2.23 – 杀手波 UML![图片](img/Figure_2.23_B18381.jpg)

图 2.22 – 杀手波 UML

如您在图中所见，图中没有提及相机，那么我们应该编写一个脚本来支持这一点吗？可以说，基于相机编写脚本的唯一原因可能是如果相机具有复杂的目的，并且包含多个属性和功能。然而，在我们的游戏中，相机是在游戏开始时放置的。稍后，在第三级，相机将使用简单的组件脚本从左向右移动，但它不包含任何其他复杂性。因此，使用`GameManager`会更理想，因为它只扮演着一个小角色。如果游戏变得更大，相机承担了更多角色，那么这可能就为相机拥有自己的类提供了理由。其他人可能会根据个人喜好提出不同意见，但我们将采取这种方法。

让我们创建`GameManager`脚本，如下所示：

1.  以创建文件夹的方式创建脚本。在**项目**窗口的空白区域右键单击，将出现一个下拉菜单。点击**创建 | C# 脚本**，如下所示：

![图 2.24 – 在 Unity 编辑器中创建 C#脚本](img/Figure_2.24_B18381.jpg)

图 2.23– 在 Unity 编辑器中创建 C#脚本

1.  脚本以标题`NewBehaviourScript`出现。我们不想这样称呼它，所以（以驼峰命名法）输入`GameManager`。

    什么是驼峰命名法？

    驼峰命名法是一种避免单词之间空格的方法。这在编程中相当常见，因为出于各种原因，通常不欢迎空格。每个新单词都以大写字母开头，所以在这种情况下，`GameManager`中的 M 就是驼峰的顶部。然而，变量通常以小写字母开头，您很快就会看到。

我们现在有了我们的`GameManager`脚本。注意 Unity 如何试图提供帮助，将图标更改为银色齿轮，因为我们正在执行的是 Unity 中认可的方法：

![图 2.25 – GameManager 图标](img/Figure_2.25_B18381.jpg)

图 2.24 – GameManager 图标

就像我们将三维模型放入`GameManager`的`Script`文件夹中时做的那样。

很好。现在，在我们打开脚本进行编码之前，我们需要将其附加到场景中的游戏对象上，这样当场景运行时，附加到游戏对象上的脚本也会运行。

要创建我们的`GameManager`游戏对象，我们需要执行以下操作：

1.  在**层次结构**窗口的空白区域右键单击。

1.  从下拉菜单中选择**创建空对象**。

1.  右键单击新创建的游戏对象，从下拉菜单中选择**重命名**。

1.  将此游戏对象重命名为`GameManager`。

1.  最后，仍然选择`GameManager`游戏对象，点击最右侧的**检查器**窗口中的**添加组件**按钮。

1.  从下拉菜单中输入`GameManager`，直到您看到`GameManager`脚本并选择它。

    小贴士

    每次我们创建一个空的游戏对象时，我们必须确保将其所有**变换**属性值重置为默认值，除非我们明确更改它们。

    要重置游戏对象的**Transform**值，请确保我们正在重置的游戏对象已被选中。点击**Inspector**窗口右上角的金属齿轮，然后选择**Reset**。

双击`GameManager`脚本以在您的 IDE（Visual Studio 或您使用的任何 IDE）中打开它，然后按照以下步骤操作：

1.  在`GameManager`脚本内部，我们将面临将`UnityEngine`库导入我们的脚本以向 Unity 自身组件添加额外功能的情况：

    ```cs
    using UnityEngine;
    public class GameManager : MonoBehaviour 
          {
    ```

在前面的代码中，我们还有我们脚本的名称以及`MonoBehaviour`被继承，以向我们的脚本添加更多功能。`MonoBehaviour`也是必需的，如果附加到该脚本的游戏对象需要在 Unity 编辑器中使用。

让我们开始在我们的`GameManager`脚本中添加一些自己的代码。

1.  创建一个空的方法，`CameraSetup`，然后在`Start`函数中运行此方法：

    ```cs
        void Start()
                  {
                 CameraSetup();  
             }
             void CameraSetup()
             {

             }
    ```

1.  在`CameraSetup`方法内部，添加对相机的引用，并将相机的位置和角度设置为除其*z*轴外的零。我们将`Z`设置为`-300`，这将使相机后退，并确保所有游戏对象都在镜头中：

    ```cs
    GameObject gameCamera =
             GameObject.FindGameObjectWithTag("MainCamera");

         //Camera Transform
         gameCamera.transform.position = new Vector3(0,0,-300);
         gameCamera.transform.eulerAngles = new Vector3(0,0,0);
    ```

1.  接下来，我们将在`CameraSetup`方法中更改相机属性：

    ```cs
     //Camera Properties
          gameCamera.GetComponent<Camera>().clearFlags =
             CameraClearFlags.SolidColor;
          gameCamera.GetComponent<Camera>().backgroundColor = 
             new Color32(0,0,0,255);
          }
    ```

这将执行以下操作：

+   移除天空背景，并用实色替换

+   将实色从默认的蓝色更改为黑色

1.  最后，保存脚本。

现在，您应该有类似以下的内容：

![Figure 2.26 – The current code layout for the GameManager script](img/Figure_2.26_B18381.jpg)

图 2.25– GameManager 脚本的当前代码布局

提示

如果您想更改与相机相关的其他设置，您可以在[`docs.unity3d.com/ScriptReference/Camera.html`](https://docs.unity3d.com/ScriptReference/Camera.html)了解更多信息。

在编辑器窗口的右上中部按下**Play**按钮，或使用快捷键*Ctrl* + *P*（在 Mac 上为*Command* + *P*）。以下截图显示了**Play**按钮的位置：

![](img/Figure_2.27_B18381.jpg)

图 2.26 – 播放、暂停和步骤按钮的位置

在场景处于播放模式时，我们可以通过以下方式检查**Main Camera**游戏对象的属性：

1.  在**Hierarchy**窗口中，选择**Main Camera**。

观察下一张截图中的**Inspector**窗口，以查看我们的脚本所做的以下更改。

1.  在**Inspector**窗口的**Transform**组件中，我们可以看到**Position**和**Rotation**属性被设置为与我们在脚本中设置相同的值（以下截图中的**1**所示）。

1.  在**Inspector**窗口的**Camera**组件中，我们可以看到**Clear Flags**和**Background**值也被设置为与我们在脚本中设置的相同值（以下截图中的**2i**和**2ii**所示）。

以下截图显示了在播放模式下**Main Camera**组件属性更新的情况：

![Figure 2.28 – Main Camera values changing with our script![图片](img/Figure_2.28_B18381.jpg)

图 2.27 – 主相机值随着我们的脚本变化

现在，希望我们的属性应该与我们所编写的脚本相同（没有错误）。如果不是，你可能会在**控制台**窗口中看到一个错误消息。如果有错误，它可能会告诉你错误所在的行。你还可以双击错误，它会带你到错误所在的行。

为了确保一切正常，请在编辑器中更改相机的**位置**和**旋转**，然后按**播放**按钮。相机的属性现在应该设置为脚本中的**位置**和**旋转**属性。

在这一点上，当编辑器仍在播放时，我们还可以创建一个相机的预制体：

1.  点击并拖动**相机**从**层次结构**窗口到**项目**窗口，我们将生成一个带有相机名称或空图标蓝色的立方体。根据我们图标的尺寸，可以通过以下截图所示的滑块来调整图标的大小：

![图片](img/Figure_2.29_B18381.jpg)

![图片](img/Figure_2.29_B18381.jpg)

图 2.28 – 项目窗口右下角的滑块可以放大和缩小缩略图

1.  将这个相机预制体移动到`Prefab`文件夹。

你可能会想，*为什么我们一开始不直接创建一个相机的预制体，而不是在代码中调整其属性设置呢？*然而，这里有两个关键点很重要：首先，我们正在为可能涵盖此类属性的考试做准备；其次，你现在知道如何通过代码动态更改这些设置。

小贴士

使用 Unity 组件进行脚本编写的好处是，我们有时可以获得比编辑器中显示的更多功能。例如，`Camera`组件有一个`layerCullDistances`属性，只能通过脚本访问。这可以提供诸如跳过渲染远距离较小游戏对象以增加游戏性能等功能。

要了解更多关于`layerCullDistances`的信息，请查看[`docs.unity3d.com/ScriptReference/Camera-layerCullDistances.html`](https://docs.unity3d.com/ScriptReference/Camera-layerCullDistances.html)的文档。

这部分内容到此结束。到目前为止，我们已经涵盖了以下内容：

+   设置游戏相机的比例

+   设置我们的 Unity 编辑器中的单独窗口

+   在 Unity 编辑器中更改我们的**相机**组件属性

+   重复我们在`GameManager`脚本中对相机所做的更改

+   将我们的`GameManager`脚本作为游戏对象添加到场景中

作为程序员，能够理解和更改 Unity 编辑器中的设置（同时也能在代码中做到这一点）的重要性可以扩展到编辑器中的其他组件。这就是我们接下来要做的，重点关注方向光。

## 设置我们的灯光

作为默认设置，每个场景都附带一个方向光，目前这是我们开始所需的所有；理想情况下，我们希望场景得到良好的照明。

在场景中已经存在作为默认灯光的方向光，在 `50`，`-30` 和 `0` 中选择它。

当我们将玩家飞船放入场景中时，这将很好地照亮它，如下面的截图所示：

![Figure 2.30 – 照亮的玩家飞船

![img/Figure_2.30_B18381.jpg]

图 2.29 – 照亮的玩家飞船

不同的灯光

Unity 提供了三种不同类型的实时灯光。除了我们提到的 **方向** 光之外，它还提供了一个 **点** 光，这是一种 360° 的发光，我们将在 *第四章* *应用艺术、动画和粒子* 中介绍。第三种灯光类型是聚光灯，或如 Unity 所称的 **spot**。**spot** 也可以应用遮罩，因此它可以投射称为 cookies 的图像。

想了解更多关于三种灯光类型的信息，请查看 [`docs.unity3d.com/Manual/Lighting.html`](https://docs.unity3d.com/Manual/Lighting.html)。

我们现在可以通过将它们添加到 `GameManager` 脚本来确保这些设置保持不变。我们还可以更改灯光的颜色。

### 通过脚本更新我们的灯光属性

在 `GameManager` 中，我们将设置 **Transform** **Rotation** 值，并将颜色色调从浅黄色更改为冷蓝色：

1.  打开 `GameManager` 脚本并输入以下方法：

    ```cs
    void LightSetup()
           {
              GameObject dirLight = GameObject.Find("Directional Light");
              dirLight.transform.eulerAngles = new Vector3(50,-30,0);
              dirLight.GetComponent<Light>().color = 
                 new Color32(152,204,255,255);
           }
    ```

1.  在 `Start` 函数的作用域内添加 `LightSetup();`。

1.  保存脚本。

`LightSetup` 方法做了三件事：

1.  它从场景中获取灯光并将其存储为引用。

1.  它使用 `EulerAngles` 设置灯光的旋转。

1.  最后，它改变了灯光的颜色。

    EulerAngles

    `eulerAngles` 允许我们给出 `Vector3` 坐标而不是 `Quaternion` 值。`eulerAngles` 使得旋转操作更加简单。有关 `eulerAngles` 的更多信息，请参阅 [`docs.unity3d.com/ScriptReference/Transform-eulerAngles.html`](https://docs.unity3d.com/ScriptReference/Transform-eulerAngles.html)。

这就是我们需要的关于灯光的所有操作。与相机一样，我们可以通过脚本访问灯光并更改其属性。

我们通过在 Unity 编辑器和 `GameManager` 脚本中更改其设置已经熟悉了我们的灯光。接下来，我们将为大多数游戏对象设置我们的接口。

# 介绍我们的界面 – IActorTemplate

`IActorTemplate` 接口是我们用来提示伤害控制、死亡和可脚本化对象资产的。使用此类接口的原因是它将继承它的类之间的通用用途联系起来。

总共有六个类将使用 `IActorTemplate` 接口，如下所示：

+   `Player`

+   `PlayerBullet`

+   `PlayerSpawner`

+   `Enemy`

+   `EnemyBullet`

+   `EnemySpawner`

下图显示了 `IActorTemplate` 接口以及我们游戏框架的部分概述：

![图 2.31 – IActorTemplate UML](img/Figure_2.31_B18381.jpg)

图 2.30 – IActorTemplate UML

让我们创建我们的接口并在过程中解释其内容：

1.  在`Assets/Scripts`文件夹中创建一个名为`IActorTemplate`的脚本。

1.  打开脚本并输入以下代码：

    ```cs
    public interface IActorTemplate
          {
          int SendDamage();
          void TakeDamage(int incomingDamage);
          void Die();
          void ActorStats(SOActorModel actorModel);
          }
    ```

1.  确保保存脚本。

我们刚刚输入的代码看起来像我们声明了一个类，但它的行为本质上不同。我们不是使用`class`关键字，而是输入`interface`后跟接口名称`IActorTemplate`。虽然不是必须以`I`开头命名任何接口，但这使得脚本易于识别。

在`interface`中，我们创建了一个方法列表，这些方法就像是对实现它们的任何类的合同。例如，我们将在本章后面创建的`Player`脚本继承自`IActorTemplate`接口。`Player`脚本必须声明来自`IActorTemplate`的函数名，否则`Player`脚本将抛出错误。

在`interface`的作用域内，我们声明方法而不需要访问器（这意味着每个方法的开头不需要`private`或`public`）。方法也不需要任何内容（也就是说，它们是空的）。

更多关于接口的信息，请参阅[`learn.unity.com/tutorial/interfaces`](https://learn.unity.com/tutorial/interfaces)。

我们`interface`中的最后一个方法是`ActorStats`，它接受一个`SOActorModel`类型。`SOActorModel`是一个可脚本化的对象，我们将在下一节中解释和创建。

# 介绍我们的可脚本化对象 – SOActorModel

在本节中，我们将介绍可脚本化对象及其优点。与我们的`interface`一样，可脚本化对象覆盖了相同的六个类。这样做的原因是，我们的`interface`使用了`SOActorModel`，因此与其他变量建立了关联。

也要提醒自己**游戏设计文档**（**GDD**）以及它是如何融入我们游戏创建概述中的。

我们的游戏有三个系列的游戏对象，它们将具有相似属性：`EnemyWave`、`EnemyFlee`和`Player`。这些属性将包括健康、速度、得分值等。这些对象之间的区别，如游戏设计简报所述，在于它们的行为以及它们在我们游戏中的实例化方式。

`Player`将在每个级别实例化，`EnemyWave`将从`EnemySpawner`生成，而`EnemyFlee`将被放置在第三级的特定区域。

所述的所有游戏对象都将与`SOActorModel`对象相关联。

以下图表也是我们游戏框架的部分视图，显示了可脚本化对象及其继承的六个类：

![图 2.32 – SOActorModel UML](img/Figure_2.32_B18381.jpg)

图 2.31 – SOActorModel UML

与前面提到的 `interface` 脚本类似，脚本化对象的名字以 `SO` 开头，这并不是命名脚本的常规方式，但它更容易被识别为 `ScriptableObject`。

此脚本化对象的目的在于为每个分配给它的游戏对象保存通用值。例如，所有游戏对象都有一个名称，因此在我们的 `SOActorModel` 中有一个名为 `actorName` 的 `string`。这个 `actorName` 将被用来命名敌人、生成器或子弹的类型。

让我们创建一个脚本化对象，如下所示：

1.  在 `Assets/Scripts` 文件夹中，文件名为 `SOActorModel`。

1.  打开脚本并输入以下代码：

    ```cs
    using UnityEngine; 
         [CreateAssetMenu(fileName = "Create Actor", menuName = 
             "Create  Actor")]
         public class SOActorModel : ScriptableObject 

         { 
              public string actorName;
              public AttackType attackType;

              public enum AttackType
               {
                  wave, player, flee, bullet
               }
              public string description;
              public int health;
              public int speed;
              public int hitPower;
              public GameObject actor;
              public GameObject actorsBullets;
         }
    ```

1.  保存脚本。

在 `SOActorModel` 中，我们将命名 `Player` 脚本中的大多数，如果不是所有这些变量。类似于 `interface` 与类签订合同的方式，`SOActorModel` 也做了同样的事情，因为它正在被继承，但它不像 `interface` 那样严格，如果脚本化对象的内容没有被应用，它会抛出一个错误。

以下是我们刚刚输入的 `SOActorModel` 代码的概述。

我们将可脚本化对象命名为 `SOActorModel`，作为一个通用术语，试图涵盖尽可能多的可能使用脚本化对象的游戏对象。这种工作方式也支持我们在第一章中提到的 SOLID 原则，它鼓励我们尝试保持代码简洁高效。

我们将为本脚本涵盖的主要类别如下：

+   `SOActorModel` 脚本使用 `using UnityEngine`；不需要其他库。

+   `CreateAssetMenu` 属性在 Unity 编辑器中右键单击并选择 **创建** 时，在 **项目** 窗口的下拉列表中创建一个额外的选择，如下面的截图所示：

![Figure 2.33 – 在 Unity 编辑器中创建 Actor]

![img/Figure_2.33_B18381.jpg]

图 2.32 – 在 Unity 编辑器中创建 Actor

+   使用 `MonoBehaviour` 而不是 `ScriptableObject`，因为这是创建资产时的一个要求。

+   **变量**：最后，这些是将被发送到我们选定类中的变量。

在接下来的几节中，我们将从脚本化对象脚本创建资产，以赋予我们的脚本不同的值。

## 创建 PlayerSpawner 脚本化对象资产

在我们创建的 `SOActorModel` `ScriptableObject` 之后，我们现在可以创建一个资产，它将作为一个模板，不仅可供程序员使用，还可以供想要调整游戏属性/设置但不需要了解如何编码的设计师使用。

要创建 `Actor Model` 资产，请按照以下步骤操作：

1.  在 Unity 编辑器中，回到 **项目** 窗口，右键单击并选择 **创建 | 创建 Actor**。

1.  将新创建的资产文件重命名为 `Player_Default` 并将其存储在 `Assets/Resources` 文件夹中。

1.  点击新资产，在 **检查器** 窗口中，您将看到资产的内容。

以下截图显示了 `Actor Model` 资产的字段，其中我已输入自己的值：

![图 2.34 – 玩家值

![图 2.34 – 图 2.34_B18381.jpg]

图 2.33 – 玩家值

让我们逐一分析添加到我们新创建的资产中的每个值：

+   `Player`）。

+   **船型**：选择这个游戏对象属于哪个类别。

+   **描述**：设计师/内部备注，不影响游戏但可能有所帮助。

+   **健康值**：玩家在死亡之前可以承受多少次打击。

+   **速度**：玩家的移动速度。

+   **打击力**：确定玩家与敌人相撞时会造成多少伤害。

+   `player_ship`预制体在这里（`Assets/Prefab/Player`）

+   `player_bullet`预制体在这里（`Assets/Prefab /Player/`）。

我们将在本章后面构建此资产后将其添加到`PlayerSpawner`脚本中。让我们继续下一个可脚本对象资产。

## 创建一个`EnemySpawner` ScriptableObject 资产

在本节中，我们将使我们的敌人资产附加到`EnemySpawner`，以便在章节的后面部分使用。为了保持我们的工作新鲜完整，让我们继续进行，然后再转到`EnemySpawner`脚本。

要创建一个敌人资产，请按照以下说明操作：

1.  在编辑器中，在**项目**窗口中，右键单击并选择**创建 | 创建演员**。

1.  将新文件重命名为它所附加的内容（`BasicWave Enemy`）并将文件存储在`Assets/ScriptableObject`位置。

1.  点击新脚本，我们的**检查器**窗口将显示脚本的内容。

以下截图显示了完成后的`BasicWave Enemy`资产将看起来像什么：

![图 2.35 – 基本波敌人值

![图 2.35 – 图 2.35_B18381.jpg]

图 2.34 – 基本波敌人值

让我们简要地过一下我们敌人的每个值：

+   `enemy_wave`。

+   `Wave`。这解释了敌人的类型以及它是如何攻击玩家的。

+   `通常在组中`。如前所述，这更多的是一个指南而不是规则，用于注释任何内容。

+   `1`，这意味着它需要 1 次打击才能死亡。

+   `-50`，因为我们的敌人是从右向左移动的，所以我们给它一个负数。

+   `1`，这意味着如果这个敌人与玩家相撞，它将造成 1 点伤害。

+   `enemy_wave`预制体在这里（`Assets/Prefab/Enemies`）。

+   **演员子弹**：这个敌人不发射子弹。

希望你能看到可脚本对象有多有用。想象一下，如果我们继续开发这个游戏，有`50`个敌人，我们只需要创建一个资产并对其进行自定义。

我们将在下一节继续本章的最后一个可脚本对象资产。

## 创建 PlayerBullet ScriptableObject 资产

在本节中，我们将为玩家开火时创建一个玩家子弹的资产。与最后两个部分一样，创建一个资产，命名为`PlayerBullet`，并将其存储在其他资产相同的文件夹中。

以下截图显示了`PlayerBullet`资产的最后结果：

![图 2.36 – 玩家子弹值

![图 2.36 – 图 2.36_B18381.jpg]

图 2.35 – 玩家子弹的值

让我们简要地过一下每个变量的值：

+   `player_bullet`。

+   **船型**：子弹。

+   **描述**：在此处输入有关资产的任何详细信息是可选的。

+   `1`。

+   `700`。

+   `1`发送 1 点伤害值。

+   `player_bullet`预制体在这里（`Assets/Prefab/Player`）。

+   **演员子弹**：**无（游戏对象）**。

在后续章节中，当我们为游戏构建商店时，我们将能够为玩家的飞船购买增强功能。其中一个增强功能将与我们刚刚制作的类似，但**演员名称**将不同，**伤害力**将更高。

现在，我们可以继续到下一部分，创建玩家的脚本并将这些资产附加到它们上。

# 设置我们的 Player、PlayerSpawner 和 PlayerBullet 脚本

在接下来的几个部分中，我们将创建三个脚本，这些脚本将涵盖以下内容：创建玩家、玩家的控制以及玩家的子弹。

我们将创建并包含的脚本如下：

+   `PlayerSpawner`：创建和校准玩家

+   `Player`：玩家控制和一般功能

+   `PlayerBullet`：子弹移动和一般功能

+   `IActorTemplate`：分配给特定对象的预期规则模板（已创建）

+   `SOActorModel`：一组可以被非程序员修改的值（已创建）

我们将对所有这些脚本进行彻底的讲解，并分解它们各自的目的，以及它们如何相互依赖和通信。我们将从`PlayerSpawner`开始，它将创建玩家的飞船并分配其值。

## 设置我们的 PlayerSpawner 脚本

`PlayerSpawner`脚本的目的是将它附加到一个游戏对象上，从而使玩家在游戏中的该位置出现。当`PlayerSpawner`脚本被创建时，它还将设置玩家的值。例如，如果我们的玩家有一个特定的速度值，或者如果他们在商店中获得了升级，`PlayerSpawner`脚本将获取这些值并将它们应用到`Player`脚本上。

以下图表显示了游戏框架中`PlayerSpawner`类的部分视图及其与其他类的关联：

![图 2.37 – PlayerSpawner UML](img/Figure_2.37_B18381.jpg)

图 2.36 – PlayerSpawner UML

如我们所见，`PlayerSpawner`脚本与四个其他脚本相连：

+   `Player`：`PlayerSpawner`与`Player`相连，因为它创建了玩家。

+   `SOActorModel`：这是一个`ScriptableObject`，它为`PlayerSpawner`提供其值，这些值随后传递给`Player`。

+   `IActorTemplate`：这是通用于其他常见功能的`interface`。

+   `GameManager`：这将从`PlayerSpawner`脚本发送和接收一般游戏信息。

在我们创建`PlayerSpawner`脚本之前，创建一个空的游戏对象来存储与我们的玩家、他们的子弹以及玩家可能在`testLevel`场景中创建的其他任何东西有关的内容是很好的管理。

按照以下步骤创建并命名游戏对象：

1.  右键点击打开空间中的**层次结构**窗口。

1.  将出现一个下拉列表。从列表中选择**创建空对象**。

1.  将游戏对象命名为`_Player`。

这就是我们需要做的全部。现在，让我们从`PlayerSpawner`脚本开始：

1.  在`Assets/Scripts`文件夹中，文件名为`PlayerSpawner`。

1.  打开脚本，确保我们在脚本顶部输入以下库：

    ```cs
    using UnityEngine;
    ```

我们只需要`using UnityEngine`，因为它涵盖了脚本中需要的所有对象。

1.  继续确保我们的类如下标记：

    ```cs
    public class PlayerSpawner : MonoBehaviour 
         {
    ```

在 Unity 中，继承`MonoBehaviour`是常见的，以便在 Unity 中为脚本提供更多功能。它的常见目的是让脚本可以附加到游戏对象上。

1.  继续输入脚本的变量：

    ```cs
       SOActorModel actorModel;
            GameObject playerShip;
    ```

在`PlayerSpawner`类内部，我们添加两个全局变量：第一个变量是`actorModel`，它包含一个可脚本化对象资产，该资产将包含玩家飞船的值，第二个变量将持有从我们的`CreatePlayer`方法创建的玩家飞船。

1.  继续通过输入脚本的`Start`函数：

    ```cs
    void Start()
          {
            CreatePlayer();
          }
    ```

在全局变量之后，我们添加一个`Start`函数，该函数将在运行时`PlayerSpawner`脚本所持有的游戏对象激活时自动运行。

在`Start`函数的作用域内，有一个我们将要创建的方法，称为`CreatePlayer`。

1.  继续通过输入`CreatePlayer`方法：

    ```cs
    void CreatePlayer()
           {
             //CREATE PLAYER
             actorModel = Object.Instantiate(Resources.Load
                ("Player_Default")) 
                    as SOActorModel;
             playerShip = GameObject.Instantiate(actorModel.actor) 
                as GameObject;
             playerShip.GetComponent<Player>().ActorStats(actorModel);

         //SET PLAYER UP

          }
         }
    ```

我将`CreatePlayer`方法分为两个注释部分（`//CREATE PLAYER`和`//SET PLAYER UP`），因为其大小。

`CreatePlayer`方法的第一部分将`实例化`玩家飞船的`ScriptableObject`资产，并将其存储在`actorModel`变量中。然后，我们`实例化`一个游戏对象，该对象引用我们的`ScriptableObject`，其中包含名为`actor`的游戏对象，位于名为`playerShip`的游戏对象变量中。最后，我们将我们的`ScriptableObject`资产应用到`Player`组件脚本中的`ActorStats`方法（我们将在本章后面创建）。

1.  继续在`CreatePlayer`方法中添加第二部分：

    ```cs
    //SET PLAYER UP
         playerShip.transform.rotation = Quaternion.Euler(0,180,0);
         playerShip.transform.localScale = new Vector3(60,60,60);
         playerShip.name = "Player";
         playerShip.transform.SetParent(this.transform);
         playerShip.transform.position = Vector3.zero;
    ```

在`CreatePlayer`方法的第二部分，我们在注释`//SET PLAYER UP`的位置添加了更多代码。

从`//SET PLAYER UP`开始的代码专门用于在关卡开始时将玩家的飞船设置在正确的位置。

代码执行以下操作：

+   设置玩家飞船的旋转方向。

+   将玩家飞船的缩放设置为所有轴上的`60`。

+   当我们`实例化`任何游戏对象时，Unity 会在游戏对象名称的末尾添加`(Clone)`。我们可以将其重命名为`Player`。

+   我们在**层次结构**窗口中，将`playerShip`游戏对象设置为`_Player`游戏对象的孩子，这样我们就可以轻松找到它。

+   最后，我们重置玩家飞船的位置。

这就是我们的 `PlayerSpawner` 脚本编写完成。现在，在下一节中，我们需要创建并将此脚本附加到游戏对象上并为其命名。确保在继续之前保存脚本。

### 创建 PlayerSpawner 游戏对象

在本节中，我们将创建一个游戏对象，它将包含我们刚刚创建的 `PlayerSpawner` 脚本，然后我们将 `PlayerSpawner` 游戏对象放置在 `testLevel` 场景中。

要创建和设置我们的 `PlayerSpawner` 游戏对象，我们需要执行以下操作：

1.  在 `PlayerSpawner` 中。

1.  将 `PlayerSpawner` 游戏对象拖放到 `_Player`（记住 `_Player` 是我们场景中的空游戏对象）游戏对象上，使其成为 `PlayerSpawner` 的子对象。

由于我们的 `PlayerSpawner` 游戏对象没有应用任何视觉元素，我们可以给它一个图标。

1.  在 **检查器** 窗口中仍然选中 `PlayerSpawner` 游戏对象，点击其名称左侧的多彩方块。将提供一系列颜色选项，如以下截图所示：

![Figure 2.38 – 为 PlayerSpawner 选择图标![img/Figure_2.38_B18381.jpg](img/Figure_2.38_B18381.jpg)

图 2.37 – 为 PlayerSpawner 选择图标

1.  选择一个颜色。现在，`PlayerSpawner` 游戏对象将被赋予一个标签，以显示它在场景中的位置。这将现在出现在 **场景** 窗口中。

    小贴士

    如果你仍然在 **场景** 窗口中看不到图标，请确保 **3D 图标** 已关闭。你可以通过点击 **场景** 窗口右上角的 ** Gizmos** 按钮并取消选中 **3D 图标** 复选框来检查。

在 `_Player` 游戏对象内部的 `PlayerSpawner` 游戏对象中，以下值：

1.  在 `PlayerSpawner` 游戏对象仍然被选中时，在 **检查器** 窗口中，给它以下 **变换** 值：

![Figure 2.39 – 检查器窗口中的 PlayerSpawner 变量![img/Figure_2.39_B18381.jpg](img/Figure_2.39_B18381.jpg)

图 2.38 – 检查器窗口中的 PlayerSpawner 变量

1.  在 `PlayerSpawner` 中继续操作，直到你在下拉列表中看到脚本出现。

1.  点击 `PlayerSpawner` 脚本来将其添加到 `PlayerSpawner` 游戏对象中。

我们还不能移动飞船，也不能开火，因为我们还没有编写这部分代码。在下一节中，我们将介绍玩家的控制方式，然后我们将继续编写玩家的代码以及子弹在屏幕上移动的代码。

## 设置我们的输入管理器

记住这是一个横向卷轴射击游戏，所以控制将是二维的，尽管我们的视觉效果是三维的。我们现在的重点是设置 `Players` 的控制。为此，我们需要访问 **输入管理器**：

1.  选择 **编辑**，然后选择 **项目设置**，然后从列表中选择 **输入管理器**：

![Figure 2.40 – 在 Unity 编辑器中选择输入管理器![img/Figure_2.40_B18381.jpg](img/Figure_2.40_B18381.jpg)

图 2.39 – 在 Unity 编辑器中选择输入管理器

**输入管理器**将提供我们游戏所有可用控制器的列表。我们首先检查默认设置的控制。这里有很多选项，但如前所述，我们只需要浏览对我们有意义的属性，即以下属性：

+   **水平**：沿着玩家的 x 轴移动玩家的飞船

+   **垂直**：沿着玩家的 y 轴移动玩家的飞船

+   **Fire1**：使玩家开火

要检查这三个属性，我们需要做以下几步：

+   通过点击其旁边的箭头展开 **轴** 下拉菜单。

+   如下截图所示，展开 **水平**：

![图 2.41 – 输入管理器](img/Figure_2.41_B18381.jpg)

图 2.40 – 输入管理器

+   `-1`），右键配置为正数（`+1`）。其他具有相同效果的按键是左边的 *A* 和右边的 *D*。

如果我们有类似摇杆或方向盘的模拟控制，那么当玩家释放控制并返回中心时，我们可能需要关注重力的影响。死点指的是模拟控制器的中心。有时，控制器可能不平衡，并且自然倾向于一侧，因此通过增加死区，我们可以消除玩家可能检测到的虚假反馈。

+   `-1`），而正数按钮向上（`+1`）。其他按钮是向下的 *S* 和向上的 *W*。

+   `mouse 0`（即左鼠标按钮）。目前，从替代按钮中移除 `mouse 0`。

要了解更多关于 **输入管理器** 窗口的信息，请点击 **输入管理器** 面板右上角的蓝色小书。

我们现在在 `Player` 脚本中设置了控制，以利用这些控制。

## 设置我们的玩家脚本

`Player` 脚本将被附加到玩家飞船游戏对象上，玩家将能够移动和开火，以及造成和承受伤害。我们还将确保玩家飞船不会超出剧本区域。在我们继续之前，让我们提醒自己 `Player` 脚本在我们游戏框架中的位置：

![图 2.42 – 玩家 UML](img/Figure_2.42_B18381.jpg)

图 2.41 – 玩家 UML

`Player` 脚本将与以下脚本进行交互：

+   `PlayerBullet`：`Player` 脚本将创建子弹进行射击。

+   `PlayerSpawner`：`Player` 脚本由 `PlayerSpawner` 创建。

+   `IActorTemplate`：包含伤害控制和 `Player` 的属性。

+   `GameManager`：存储额外的信息，如生命值、分数、等级以及玩家飞船积累的任何升级。

+   `SOActorModel`：存储 `ScriptableObject` 的 `Player` 属性。

现在我们熟悉了 `Player` 脚本与其他脚本的关系，我们可以开始编写它：

1.  在 `Assets/Scripts` 文件夹中，文件名为 `Player`。

1.  打开脚本，并将 `IActorTemplate` 接口添加到现有的默认代码中：

    ```cs
    using UnityEngine;

          public class Player : MonoBehaviour, IActorTemplate
          {
    ```

默认情况下，脚本将导入一个`UnityEngine`库（包括其他一些库），类的名称，以及`MonoBehaviour`。所有这些对于使脚本在 Unity 编辑器中工作都是必需的。

1.  继续在`Player`脚本中输入以下全局变量：

    ```cs
             int travelSpeed;
             int health;
             int hitPower;
             GameObject actor;
             GameObject fire;    

             public int Health
             {
                 get {return health;}
                 set {health = value;}
             }

             public GameObject Fire
             {
                 get {return fire;}
                 set {fire = value;}
             }

             GameObject _Player;

             float width;
             float height;

    ```

我们在我们的全局变量中输入了整数、浮点数和游戏对象混合体；从顶部开始，前六个变量将从玩家的`SOActorModel`脚本中更新。`travelSpeed`是玩家飞船的速度，`health`是玩家在死亡之前可以承受的打击次数，`hitPower`是飞船在撞击可以承受伤害的物体（敌人）时造成的伤害，`actor`是用于表示玩家的三维模型，最后，`fire`变量是玩家发射的三维模型。如果这看起来有点仓促，请回到*介绍我们的 ScriptableObject – SOActorModel*部分，在那里我们更详细地介绍了这些变量。

`Health`和`Fire`这两个公共属性存在是为了让其他需要访问的类能够访问我们的两个`private health`和`fire`变量。

`_Player`变量将用于引用场景中的`_Player`游戏对象。

最后两个变量`width`和`height`将用于存储游戏在游戏中使用的屏幕世界空间尺寸的测量结果。我们将在下一块代码中进一步讨论这两个变量。

在我们开始以下`Start`函数代码块之前，你可能想知道为什么在运行函数代码内容时我们会选择`Start`而不是`Awake`。这两个函数在运行时都只运行一次；唯一明显的区别是`Awake`在对象创建时运行。`Start`在对象启用时执行，如[`docs.unity3d.com/Manual/ExecutionOrder.html`](https://docs.unity3d.com/Manual/ExecutionOrder.html)上的文档所示。

为了简化我们的 Unity 项目，我们将在这两个函数之间进行选择。这是为了避免多个`Awake`函数同时运行时的冲突。例如，一个脚本可能试图更新其 Text UI，但更新文本的变量在运行时可能仍然是 null，因为包含该变量的脚本仍在等待其内容更新。

通过转到 Unity 的**脚本执行顺序**在**编辑** | **项目设置** | **脚本执行顺序**，可以避免在运行时由多个脚本调用多个`Awake`函数之间的冲突。

如果你想了解更多关于**脚本执行顺序**的信息，请查看[`docs.unity3d.com/Manual/class-MonoManager.html`](https://docs.unity3d.com/Manual/class-MonoManager.html)上的文档。

1.  继续在`Player`脚本中输入代码，接下来，我们将输入`Start`函数及其内容：

    ```cs
     void Start()
          {
            height = 1/(Camera.main.WorldToViewportPoint (new
               Vector3(1,1,0)).y - .5f);
            width = 1/(Camera.main.WorldToViewportPoint(new Vector3(1,1,0))
               .x - .5f);

            _Player = GameObject.Find("_Player");
          }
    ```

如前所述，`height`和`width`变量将存储我们的世界空间测量值。这些是必需的，以便我们可以将玩家的飞船限制在屏幕内。高度和宽度代码行使用类似的方法；唯一的区别是我们读取的轴。

`Camera.main`组件指的是场景中的摄像机，它使用的函数`WorldToViewportPoint`是将游戏的三维世界空间的结果转换为视口空间。如果你不确定视口空间是什么，它类似于我们知道的屏幕分辨率，只是其测量单位是点而不是像素，这些点是从`0`到`1`进行测量的。以下图表显示了屏幕与视口测量的比较：

![图 2.43 – 屏幕与视口测量](img/Figure_2.43_B18381.jpg)

图 2.42 – 屏幕与视口测量

因此，使用视口，无论屏幕的分辨率如何，完整的高度和宽度都是`1`，而所有介于其间的都是分数。所以，对于高度，我们向`WorldToViewportPoint`提供`Vector3`，其中`Vector3`表示世界空间值，后面跟着`-0.5f`，将其偏移量设置回`0`。然后，我们将`1`（这是我们全屏大小）除以我们公式的结果。这将给我们当前屏幕的世界空间高度。然后，我们应用相同的原则来处理宽度，使用`x`而不是`y`，并存储结果。

最后，代码的最后一行将场景中`_Player`游戏对象的引用存储到我们的变量中。

1.  继续使用`Player`脚本，我们有一个在每一帧被调用的`Update`函数。输入函数以及以下两个方法：

    ```cs
     void Update ()
          {
              //Movement();
              //Attack();
          }
    ```

`Update`函数在每一帧运行`Movement`方法和`Attack`方法。我们将在本章后面深入探讨这两个方法，现在我们将这两个方法注释掉（"//"），以避免脚本无法运行。

我们接下来要放入`Player`脚本中的下一个方法是`ActorStats`方法。这个方法是必需的，因为我们声明了我们在继承的接口中。

1.  在我们的`Update`函数作用域之后，输入以下代码段：

    ```cs
    public void ActorStats(SOActorModel actorModel)
          {
              health = actorModel.health;
              travelSpeed = actorModel.speed;
              hitPower = actorModel.hitPower;
              fire = actorModel.actorsBullets;
          }
    ```

我们刚刚输入的代码将分配来自玩家`SOActorModel` `ScriptableObject`资产中的值，这是我们本章早期制作的。

这种方法不会在我们的脚本中运行，而是由其他类访问，原因在于这些变量持有关于我们的玩家的值，并且不需要在任何其他地方。

1.  保存`Player`脚本。

在我们测试到目前为止的内容之前，我们需要在**项目**窗口中将我们的`Player`脚本附加到`player_ship`上。

1.  在`Assets/Prefab`中，选择`player_ship`预制体。

1.  选择`Player`，直到脚本出现，然后选择它。

使用我们的`_Player`、`PlayerSpawner`和`GameManager`游戏对象，现在是时候测试游戏了。我们可以在编辑器中按**播放**键看到玩家飞船在我们的**游戏**窗口中创建。

以下截图显示了我们的游戏中的`PlayerSpawner`游戏对象作为`Player`游戏对象的父亲；同时注意`PlayerSpawner`图标：

![图 2.44 – 我们游戏中当前的玩家设置](img/Figure_2.44_B18381.jpg)

图 2.43 – 我们游戏中当前的玩家设置

小贴士

在进入下一节之前，通过将`PlayerSpawner`游戏对象拖放到`Assets/Player`中创建一个预制体。这样，如果你因为任何原因丢失了场景及其**层次结构**内容，你可以将你的预制体拖回。这应该适用于任何常见的活动游戏对象。

让我们进入下一节，我们将继续在`Player`脚本上工作，但这次，我们将查看当玩家的游戏对象与敌人接触时会发生什么。

### 与敌人碰撞 – OnTriggerEnter

在本节中，我们将在`Player`脚本中添加一个函数，用于在运行时检查我们的玩家游戏对象发生了什么碰撞。目前，唯一可以与我们的玩家发生碰撞的是敌人，但我们仍然可以展示 Unity 自带的`OnTriggerEnter`函数的使用，该函数为我们处理了大部分工作：

1.  在`Player`脚本中我们的上一个方法（`ActorStats`）的作用域之后继续，我们将添加以下代码，用于检测我们的敌人是否与玩家的飞船发生碰撞：

    ```cs
      void OnTriggerEnter(Collider other)
           {
             if (other.tag == "Enemy")
             {
               if (health >= 1)
               {
                if (transform.Find("energy +1(Clone)"))
                 {
                  Destroy(transform.Find("energy +1(Clone)").              gameObject);
                  health -= other.GetComponent<IActorTemplate>
                    ().SendDamage();
                 }
                 else
                 {
                     health -= 1;
                 }
               }

               if (health <= 0)
               {
                 Die();
               }
             }
           }
    ```

让我们解释一下我们刚刚输入到`Player`脚本中的部分代码：

+   `OnTriggerEnter(Collider other)`是一个 Unity 识别的函数，用于检查什么进入了玩家的触发碰撞器。

+   我们使用一个`if`语句来检查碰撞器的`tag`是否被命名为`Enemy`。注意当我们创建我们的敌人时，我们将给他们一个`Enemy` `tag`，这样它们就容易被识别。如果`tag`等于`Enemy`，我们就将其放入那个`if`语句中。

+   下一个`if`语句检查玩家的`health`是否等于或大于`1`。如果是，这意味着玩家可以承受打击并继续游戏而不会死亡，同时也意味着我们可以进入其`if`语句。

+   我们接近第三个`if`语句，该语句检查碰撞器是否有一个名为`energy +1(Clone)`的游戏对象。这个对象的名称是玩家可以在游戏商店购买的护盾的名称，我们将在*第六章*，*购买游戏内物品和广告*中添加。如果玩家有这个`energy +1(Clone)`对象，我们可以使用 Unity 的预制函数`Destroy`它。我们还从敌人的`SendDamage`函数中扣除玩家的额外生命值。我们将在本章后面讨论`SendDamage`。

+   在第三个 `if` 语句之后是一个 `else` 条件，其中，如果玩家没有 `energy +1(Clone)` 游戏对象，他们的健康值会被扣除。

+   最后，如果玩家的 `health` 值为零或以下，我们将运行 `Die` 方法，我们将在本章后面介绍。

    小贴士

    在我们继续向项目中添加更多代码的同时，不要忘记保存你的工作。

让我们继续我们的 `Player` 脚本并添加功能，以便玩家可以从敌人那里接收和发送伤害。

1.  在下一个方法中，我们将添加两个方法。第一个方法 (`TakeDamage`) 将接受一个名为 `incomingDamage` 的整数，并使用该值从我们的玩家 `health` 值中扣除。

第二个方法 (`SendDamage`) 将返回我们的 `hitPower` 值的整数。

1.  在我们的 `ActorStats` 方法下方和范围之外，现在添加以下代码：

    ```cs
    public void TakeDamage(int incomingDamage)
          {
            health -= incomingDamage;
          }

          public int SendDamage()
          {
            return hitPower;
          }
    ```

让我们继续另一个 `Player` 脚本的方法，并使其能够控制玩家飞船在 **游戏** 窗口周围移动。

### 移动方法

在本节中，我们将编写 `Movement` 方法，它将从玩家的摇杆/键盘接收输入，并利用 `height` 和 `width` 浮点数来确保玩家的飞船保持在屏幕内：。

1.  仍然在 `Player` 脚本中，使用以下内容开始以下方法以检查玩家的输入：

    ```cs
    void Movement()
         {
           if (Input.GetAxisRaw("Horizontal") > 0)
           {
              if (transform.localPosition.x < width +              width/0.9f)
              {
                transform.localPosition += new Vector3
                   (Input.GetAxisRaw("Horizontal")
                     *Time.deltaTime*travelSpeed,0,0);                                                                                                                            
              }
           }
    ```

    +   `Movement` 方法将包括检测玩家从四个方向进行的移动；我们将从玩家按下控制器/键盘上的右键开始。我们运行一个 `if` 语句来检查输入管理器是否检测到来自 `Horizontal` 属性的任何移动。如果 `GetAxisRaw` 检测到一个大于零的值，我们就会进入 `if` 语句的条件。请注意，`GetAxisRaw` 没有平滑处理，所以除非添加额外的代码，否则玩家的飞船会立即移动。

    +   接下来，我们还有一个 `if` 语句；这个语句检查玩家是否超过了 `width`（即我们之前在章节中计算的屏幕世界空间）。我们还添加了一个额外的部分 `width` 以避免玩家的飞船几何形状离开屏幕。如果玩家的位置仍然低于 `width`（及其缓冲区）值，我们将在 `if` 语句内运行内容。

    +   玩家的位置通过一个 `Vector3` 结构更新，它包含 `Horizontal` 方向的值，乘以每帧的时间以及我们从 `ScriptableObject` 设置的 `travelSpeed`。

1.  让我们在 `Movement` 方法中继续，并添加一个类似的 `if` 语句以将玩家飞船向左移动：

    ```cs
    if (Input.GetAxisRaw("Horizontal") < 0)
            {
              if (transform.localPosition.x > width +               width/6)
              {
               transform.localPosition += new Vector3
                 (Input.GetAxisRaw("Horizontal")
                   *Time.deltaTime*travelSpeed,0,0);                                                                       
              }
            } 
    ```

如我们所见，代码接近之前的块；唯一的区别是，我们的第一个 `if` 语句检查我们是否在向左移动；第二个 `if` 语句检查玩家的位置是否大于宽度以及一个略微不同的缓冲区。

除了这些，`if`语句及其内容在相反的方向上服务于相同的位置。

1.  让我们继续使用我们的`Movement`方法，并添加移动玩家飞船下方的`if`语句代码：

    ```cs
    if (Input.GetAxisRaw("Vertical") < 0)
         {
             if (transform.localPosition.y > -height/3f)
             {
              transform.localPosition += new Vector3
               (0,Input.GetAxisRaw("Vertical")*Time.deltaTime*travelSpeed,0);
             }
         }
    ```

再次强调，我们遵循与前面两个`if`语句相同的规则，但这次，我们添加了`Vertical` `string`属性，而不是`Horizontal`。在第二个`if`语句中，我们检查玩家的 y 轴是否高于负的`height/3`。我们之所以除以这个值，是因为在本书的后面部分（*第九章**，创建 2D 商店界面和游戏内 HUD*），我们将在屏幕底部添加图形，这将限制玩家的视野。

1.  让我们继续到`Movement`方法中的最后一个`if`语句，即向上移动：

    ```cs
    if (Input.GetAxisRaw("Vertical") > 0)
            {
             if (transform.localPosition.y < height/2.5f)
            {
             transform.localPosition += new Vector3
               (0,Input.GetAxisRaw("Vertical")*Time.deltaTime*travelSpeed,0);
            }
           }
         }
    ```

与之前一样，这个`if`语句承担着类似的角色，但这次，它检查的是玩家的位置是否低于`height/2.5f`值。应用了一个缓冲区以防止三维几何体离开屏幕顶部。

小贴士

在制作游戏时，有时会遇到玩家斜向移动时速度增加的情况。这是因为玩家实际上同时按下了两个方向，而不是一个方向。

为了确保一个方向只有`1`的量级，我们可以使用 Unity 预制的`Normalize`函数。

要了解更多关于这个函数的信息，请查看[`docs.unity3d.com/ScriptReference/Vector3.Normalize.html`](https://docs.unity3d.com/ScriptReference/Vector3.Normalize.html)上的文档。

1.  不要忘记保存脚本。

我们将继续通过添加`Die`方法来完善`Player`脚本。

### `Die`方法

将`Die`方法添加到`Player`脚本中，将使我们的玩家可以被摧毁。目前，在`Die`方法中有一个 Unity 函数叫做`Destroy`；这个函数将删除其参数内的任何游戏对象。

将以下方法输入到`Player`脚本中，以摧毁玩家：

```cs
public void Die()
      {
          Destroy(this.gameObject);
      }
```

让我们继续到`Player`脚本中的最后一个方法，即攻击方法。

### `Attack`方法

在本节中，我们将向`Player`脚本中的`Attack`方法添加内容。

这个`Attack`方法的目的是从玩家那里接收输入，创建子弹，将子弹指向正确的方向，并将子弹设置为`Player`游戏对象的子对象，以保持我们的**层次结构**窗口整洁。

将以下`Attack`方法输入到`Player`脚本中，以允许玩家开火：

```cs
public void Attack()
      {
       if (Input.GetButtonDown("Fire1"))
         {
             GameObject bullet = GameObject.Instantiate
                (fire,transform.position,Quaternion.Euler
                  (new Vector3(0, 0, 0))) as GameObject;
             bullet.transform.SetParent(_Player.transform);
             bullet.transform.localScale = new Vector3(7,7,7);
         }
       }

```

在`Attack`方法内部，我们调用一个`if`语句来检查玩家是否按下了`Fire1`按钮（Windows 上的*左 Ctrl*；如果你使用的是 Mac，则是*command*）。如果玩家按下了`Fire1`按钮，我们将进入`if`语句的作用域。

注意

当开发者提到函数的作用域、`if` 语句、类等时，他们指的是大括号开闭之间的内容。例如，如果以下代码中的 `money` 变量值更高，下面的 `if` 语句将会执行：

`if (money > costOfPizza)`

`{`

`//两个大括号顶部和底部之间发生的事情都在 if 语句的作用域内。`

`}`

在 `if` 语句内部，我们再添加一个 `if` 语句以确保在点击鼠标时，我们点击的是屏幕而不是任何 UI 相关的内容。当我们在 *第十章* 中查看添加暂停按钮、更改音效和模拟测试时，这会变得更加相关。如果我们点击了任何 UI 相关的内容，我们调用 `return`，这意味着我们退出 `if` 语句，这样就不会发射子弹。

由于我们已经输入了移动和攻击函数的内容，我们可以滚动回 `Update` 函数并移除我们添加的注释。

我们的 `Update` 函数现在看起来如下所示：

```cs
void Update()
{
	Movement();
	Attack();
}
```

接下来，我们从其实例名称 `fire` 中 `Instantiate` 我们的 `PlayerBullet` 游戏对象。我们还使 `fire` 游戏对象相对于屏幕向右，并移动它以朝向迎面而来的敌人。我们将创建和定位我们的游戏对象的结果存储在一个名为 `bullet` 的变量中。

然后，我们将子弹的大小设置为原始大小的七倍，使其看起来更大。

最后，在 `if` 语句内部，我们让我们的 `bullet` 游戏对象位于一个名为 `_Player` 的单个游戏对象内。

那就是为 `Player` 脚本所需的全部代码！在继续之前，请确保保存脚本。

在下一节中，我们将继续到另一个玩家脚本，该脚本控制玩家发射子弹时发生的情况。

## 设置我们的 `PlayerBullet` 脚本

在本节中，我们将创建一个子弹，它将从玩家的飞船穿越屏幕。

你会注意到 `PlayerBullet` 脚本与 `Player` 脚本非常相似，因为它携带了 `IActorTemplate` 和 `SOActorModel` 脚本，这些脚本已经编码在 `Player` 脚本中。

让我们创建我们的 `PlayerBullet` 脚本：

1.  在 `Assets/Scripts` 文件夹中，文件名为 `PlayerBullet`。

1.  打开脚本，并在脚本顶部检查/输入以下代码：

    ```cs
    using UnityEngine;
    ```

如前所述，默认情况下，我们需要 `UnityEngine` 库。

1.  让我们继续检查正确的类名并输入以下继承：

    ```cs
    public class PlayerBullet : MonoBehaviour, IActorTemplate {
    ```

我们声明 `public` 类并默认继承 `MonoBehaviour`。我们还继承 `IActorTemplate` 接口，以便从其他游戏对象脚本（如 `SendDamage` 和 `TakeDamage`）中获取与游戏对象相关的函数。

1.  将以下全局变量输入到 `PlayerBullet` 脚本中：

    ```cs
    GameObject actor;
         int hitPower;
         int health;
         int travelSpeed;

         [SerializeField]
         SOActorModel bulletModel;
    ```

我们添加的所有变量都是 `private`。最后一个变量添加了 `SerializeField` 属性。`SerializeField` 使得这个变量在 `private` 中可见，我们仍然可以将资产拖放到其字段中（我们将在稍后这样做）。有关 `SerializeField` 属性的更多信息，请参阅 [`docs.unity3d.com/ScriptReference/SerializeField.html`](https://docs.unity3d.com/ScriptReference/SerializeField.html)。

1.  接下来，我们将进入 `Awake` 函数及其内容：

    ```cs
    void Awake()
          {
              ActorStats(bulletModel);
          }
    ```

在我们的 `Awake` 函数中是 `ActorStats` 方法，这是必需的，因为我们正在继承一个声明它的 `interface`。

1.  继续输入 `SendDamage` 和 `TakeDamage` 方法：

    ```cs
     public int SendDamage()
          {
            return hitPower;
          }

          public void TakeDamage(int incomingDamage)
          {
            health -= incomingDamage;
          }
    ```

如本章已提到的，我们需要这些方法来发送和接收伤害。

1.  继续前进，我们进入 `Die` 方法及其内容：

    ```cs
     public void Die()
          {
              Destroy(this.gameObject);
          }
    ```

从我们的 `interface` 中包含的另一个方法是 `Die` 方法。

1.  接下来，进入 `ActorStats` 方法：

    ```cs
     public void ActorStats(SOActorModel actorModel)
          {
            hitPower = actorModel.hitPower;
            health = actorModel.health;
            travelSpeed = actorModel.speed;
            actor = actorModel.actor;
          }
    ```

我们从 `interface` 继承的最后一个方法是 `ActorStats` 方法，它将包含我们的 `ScriptableObject` 资产。然后，这个资产将被分配给 `PlayerBullet` 脚本的全局变量。

1.  下一个函数是 `OnTriggerEnter`，以及其 `if` 语句条件检查，如下所示：

    ```cs
    void OnTriggerEnter(Collider other)
          {
          if (other.tag == "Enemy")
          {
              if(other.GetComponent<IActorTemplate>() != null)
              {
                  if (health >= 1)
                  {
                      health -= other.GetComponent<IActorTemplate>
                        ().SendDamage();
                  }
                  if (health <= 0)
                  {
                      Die();
                  }
              }
           }
          }
    ```

在前面的代码块中，我们进行了一个检查，看看我们的子弹是否与被标记为 `"Enemy"` 的碰撞器发生了碰撞。如果碰撞器被标记为 `"Enemy"` 并指向玩家，我们接着检查该碰撞器是否包含 `IActorTemplate` 接口。如果没有，那么 `"Enemy"` 碰撞器可能是一个障碍物。否则，我们从 `"Enemy"` 游戏对象中扣除 `health` 并检查它是否已死亡。

1.  现在，让我们进入子弹移动的 Unity `Update` 函数：

    ```cs
    void Update ()
          {
            transform.position += new
               Vector3(travelSpeed,0,0)*Time.deltaTime;
          }
    ```

`Update` 函数根据其 `travelSpeed` 值乘以 `Time.deltaTime`（`Time.deltaTime` 是从上一帧开始的秒数）在每一帧向其 x 轴添加值。

重要提示

如果你想了解更多关于 `Time.deltaTime` 的信息，请查看文档 [`docs.unity3d.com/ScriptReference/Time-deltaTime.html`](https://docs.unity3d.com/ScriptReference/Time-deltaTime.html)。

1.  接下来，进入 Unity 的 `OnBecameInvisible` 函数：

    ```cs
        void OnBecameInvisible() 
             {
               Destroy(gameObject);
             }
         }
    ```

最后一个函数将移除任何已离开屏幕的不必要子弹。这将有助于提高我们游戏的表现并保持其整洁。在继续之前，请确保已保存脚本。

接下来，我们需要将 `PlayerBullet` 脚本应用到我们的 `player_bullet` 预制件上：

1.  导航到 `Assets/Prefab/Player` 并选择 `player_bullet`。

1.  在选择 `Player_Bullet` 后，点击 `PlayerBullet` 直到看到 `PlayerBullet` 脚本。

1.  选择脚本，并将 `PlayerBullet` 资产从 **Bullet Model** 字段添加到其中（将资产拖放到字段中或点击其字段右侧的远程按钮）。

以下截图显示了带有脚本和资产的 `player_bullet`：

![图 2.45 – 检查器窗口中的玩家 _ 子弹组件](img/Figure_2.45_B18381.jpg)

图 2.44 – 检查器窗口中的 player_bullet 组件

我们现在可以继续下一节，关于为玩家攻击而创建敌人的内容！

# 规划和创建我们的敌人

我们有一个可以移动、射击和承受伤害的玩家；我们现在可以开始考虑创建一个具有这些属性的敌人。

为了提醒自己我们正在制作的类型，我们的游戏具有与经典街机射击游戏（如 Konami 的*Gradius*、Capcom 的*UN Squadron*和 Irem 的*R-Type*）相同的特征([`github.com/retrophil/Unity-Certified-Programmer-Exam-Guide-2nd-Edition/blob/main/Reference/shootEmUps.png`](https://github.com/retrophil/Unity-Certified-Programmer-Exam-Guide-2nd-Edition/blob/main/Reference/shootEmUps.png)）。通常，在这类游戏中，玩家会被从屏幕右侧涌来的敌人包围，并从左侧退出。

在本节中，我们将重复`PlayerSpawner`和`Player`脚本的一些类似方面。`EnemySpawner`脚本需要调整，以便以一定速率实例化一定数量的敌人飞船。

`Enemy`游戏对象将自行移动，因此需要对其行为应用一些额外的代码。在我们开始创建第一个敌人脚本之前，让我们看看我们的游戏框架的一部分，并注意布局基本上与游戏框架的玩家端相同：

![图 2.46 – EnemySpawner 和 Enemy UML](img/Figure_2.46_B18381.jpg)

图 2.45 – EnemySpawner 和 Enemy UML

在我们跳入`EnemySpawner`脚本之前，让我们做与我们的玩家游戏对象相同的清理工作，即创建一个空的游戏对象并将所有相关的游戏对象存储在该游戏对象中。我们这样做是为了在**层次结构**窗口中去除杂乱，所以让我们为我们的敌人也这样做：

1.  在**层次结构**窗口的空白区域右键点击。

1.  将出现一个下拉列表；选择**创建空对象**。

1.  将游戏对象命名为`_Enemies`。

让我们继续编写敌人脚本。

# 设置我们的 EnemySpawner 和 Enemy 脚本

在本节中，我们将开始编写我们的`EnemySpawner`脚本和游戏对象。`EnemySpawner`脚本的目的是在设定速率下多次生成一个敌人游戏对象。一旦我们的`testLevel`场景开始，我们的敌人生成器将开始释放敌人。然后敌人将移动到屏幕的左侧。这相当简单，正如前一小节简要提到的，`EnemySpawner`使用与`PlayerSpawner`相同的`interface`和可脚本化的对象来`实例化`敌人。让我们首先创建我们的`EnemySpawner`脚本：

1.  在`Assets/Scripts`文件夹中，文件名为`EnemySpawner`。

1.  打开脚本并输入以下代码：

    ```cs
    using System.Collections;
         using UnityEngine;
    ```

如往常一样，我们使用默认的`UnityEngine`库。

我们还将使用另一个名为 `System.Collections` 的库。当我们使用协程时，需要这个库，这将在本节后面解释。

1.  接下来，我们将检查/输入类名及其继承：

    ```cs
    public class EnemySpawner : MonoBehaviour
         {
    ```

确保类名为 `EnemySpawner`，并且它默认继承自 `MonoBehaviour`。

1.  接下来，向 `EnemySpawner` 脚本添加四个全局变量：

    ```cs
     [SerializeField]
          SOActorModel actorModel;
          [SerializeField]
          float spawnRate;
          [SerializeField]
          [Range(0,10)]
     int quantity;
          GameObject enemies;
    ```

在前面的代码中输入的所有变量都具有 `private` 的可访问级别，除了 `enemies` 变量之外的所有变量都应用了 `SerializeField` 和 `Range` 属性，范围在 `0` 到 `10` 之间。这样做的原因是，我们可以或其他设计师可以轻松地从 **Inspector** 窗口中更改 `EnemySpawner` 的生成速率和敌人数量，如下面的截图所示：

![Figure 2.47 – Enemy spawn rate slider](img/Figure_2.47_B18381.jpg)

![Figure 2.47_B18381.jpg](img/Figure_2.47_B18381.jpg)

![Figure 2.46 – Enemy spawn rate slider](img/Figure_2.46_B18381.jpg)

1.  现在，让我们输入 Unity 的 `Awake` 函数和一些内容：

    ```cs
    void Awake()
          {
              enemies = GameObject.Find("_Enemies");
              StartCoroutine(FireEnemy(quantity, spawnRate));
          }
    ```

在 `Awake` 函数中，我们从一个空的 `_Enemies` 游戏对象分隔符创建一个实例，并将其存储在 `enemies` 变量中。

在我们的 `Awake` 函数中的第二行代码是一个 `StartCoroutine`。

重要信息

`StartCoroutine()` 和 `IEnumerator` 两者相辅相成。它们的作用类似于一个方法，接受参数并在其中运行代码。与协程的主要区别在于，它们可以被帧更新或时间延迟。你可以把它们看作是 Unity 自身 `Invoke` 函数的更高级版本。

要了解更多关于协程以及如何在 `IEnumerator` 实例中实现它们的信息，请查看 Unity 的文档：[`docs.unity3d.com/ScriptReference/MonoBehaviour.StartCoroutine.html`](https://docs.unity3d.com/ScriptReference/MonoBehaviour.StartCoroutine.html)。

这将用于运行我们创建敌人的方法，但正如你可能也注意到的，它接受两个参数。第一个是它所包含的敌人数量，第二个是 `spawnRate`，它延迟每个生成的敌人。

1.  接下来，在我们的 `EnemySpawner` 脚本中，我们有 `FireEnemy`，它将被用来运行创建和定位每个敌人的循环，然后等待重复该过程。

1.  接下来，在 `Awake` 函数下方和外部，我们可以添加我们的 `IEnumerator`：

    ```cs
    IEnumerator FireEnemy(int qty, float spwnRte)
          {
           for (int i = 0; i < qty; i++)
           {
            GameObject enemyUnit = CreateEnemy();
            enemyUnit.gameObject.transform.SetParent(this.transform);
            enemyUnit.transform.position = transform.position;
            yield return new WaitForSeconds(spwnRte); 
           }
            yield return null;
           }
    ```

在 `FireEnemy` `IEnumerator` 中，我们启动一个 `for` 循环，该循环将遍历其 `qty` 值。

在 `for` 循环内，添加以下内容：

+   我们还没有介绍的一个方法，称为 `CreateEnemy`。`CreateEnemy` 的结果将被转换成一个名为 `enemyUnit` 的游戏对象实例。

+   `enemyUnit` 是从 `EnemySpawner` 游戏对象中飞出的敌人。

+   我们将 `EnemySpawner` 位置分配给我们的 `enemyUnit`。

+   我们等待 `spwnRte` 值所设置的秒数。

+   最后，这个过程会一直重复，直到 `for` 循环达到其总数。

1.  最后，在 `FireEnemy` `IEnumerator` 下方和外部添加以下方法：

    ```cs
    GameObject CreateEnemy()
         {
           GameObject enemy = GameObject.Instantiate(actorModel.actor) 
              as GameObject;
           enemy.GetComponent<IActorTemplate>().ActorStats(actorModel);
           enemy.name = actorModel.actorName.ToString();
           return enemy;
         }
         }
    ```

正如我们提到的，有一个名为 `CreateEnemy` 的方法。除了显而易见的功能外，此方法还将执行以下操作：

1.  从其 `ScriptableObject` 资产中 `Instantiate` `enemy` 游戏对象。

1.  从其 `ScriptableObject` 资产中为我们的敌人应用值。

1.  从其 `ScriptableObject` 资产中为敌人游戏对象命名。

不要忘记保存脚本。

我们现在可以继续到下一节，我们将创建并准备带有其游戏对象的 `EnemySpawner`。

## 将我们的脚本添加到敌人生成器游戏对象

最后，我们需要将我们的 `EnemySpawner` 脚本附加到一个空的游戏对象上，这样我们就可以在 `testLevel` 场景中使用它。要设置 `EnemySpawner` 游戏对象，请执行以下操作：

1.  创建一个空的游戏对象并将其命名为 `EnemySpawner`。

1.  正如我们处理 `_Player` 和 `PlayerSpawner` 一样，我们需要在 **层次结构** 窗口中将 `EnemySpawner` 游戏对象移动到 `_Enemies` 游戏对象内部。

1.  将 `EnemySpawner` 游戏对象移动到 `_Enemies` 游戏对象后，我们现在需要在 **检查器** 窗口中更新 `EnemySpawner` 游戏对象的 **变换** 属性值：

![Figure 2.48 – 在检查器窗口中敌人生成器的变换值![img/Figure_2.48_B18381.jpg](img/Figure_2.48_B18381.jpg)

图 2.47 – 在检查器窗口中敌人生成器的变换值

1.  仍然在 `EnemySpawner` 中，直到你在列表中看到它，然后点击它。

此外，为了在 `EnemySpawner` 游戏对象中提供视觉辅助，就像我们在 *创建玩家生成器游戏对象* 部分中所做的那样。

以下截图显示了给我的 `EnemySpawner` 分配的图标：

![Figure 2.49 – 敌人生成器图标![img/Figure_2.49_B18381.jpg](img/Figure_2.49_B18381.jpg)

图 2.48 – 敌人生成器图标

我们现在可以在 **检查器** 窗口中将一个敌人添加到我们的 `EnemySpawner` 游戏对象中，并使用其脚本：

![Figure 2.50 – 持有基本波敌人演员的敌人生成器组件![img/Figure_2.50_B18381.jpg](img/Figure_2.50_B18381.jpg)

图 2.49 – 持有基本波敌人演员的敌人生成器组件

我们现在可以继续到下一节，创建我们的敌人脚本。

## 设置我们的敌人脚本

就像我们的玩家飞船是从 `PlayerSpawner` 创建出来的一样，我们的第一个敌人将是从其 `EnemySpawner` 创建出来的。敌人脚本将包含类似的变量和函数，但它还将有自己的移动，类似于 `PlayerBullet` 沿其 *x* 轴移动。

让我们开始创建我们的敌人脚本：

1.  在 `Assets/Scripts` 文件夹中，文件名为 `EnemyWave`。

1.  打开脚本，并在脚本顶部检查/输入以下所需的库代码：

    ```cs
    using UnityEngine;
    ```

就像我们的大多数类一样，我们需要 `UnityEngine` 库。

1.  检查并输入类名及其继承：

    ```cs
    public class EnemyWave : MonoBehaviour, IActorTemplate
         {
    ```

我们有一个名为 `EnemyWave` 的 `public class`，默认继承自 `MonoBehaviour`，但还添加了我们的 `IActorTemplate` 接口。

1.  在 `EnemyWave` 类中，输入以下全局变量：

    ```cs
     int health;
          int travelSpeed;
          int fireSpeed;
          int hitPower;

          //wave enemy
          [SerializeField]
          float verticalSpeed = 2;
          [SerializeField]
          float verticalAmplitude = 1;
          Vector3 sineVer;
          float time;
    ```

`EnemyWave`类的全局变量是前四个变量，这些变量使用其`ScriptableObject`资产中的值进行更新。其他变量是针对特定敌人的，我们为其中两个变量赋予了`SerializeField`属性，以便在**检查器**窗口中进行调试。

1.  添加 Unity 的`Update`函数及其内容：

    ```cs
    void Update ()
          {
              Attack();
          }
    ```

在全局变量之后，我们添加一个包含`Attack`方法的`Update`函数。

1.  添加我们的`ScriptableObject`方法`ActorStats`及其内容：

    ```cs
    public void ActorStats(SOActorModel actorModel)
          {
              health = actorModel.health;
              travelSpeed = actorModel.speed;
              hitPower = actorModel.hitPower;
          }
    ```

我们有一个`ActorStats`方法，它接受一个`ScriptableObject` `SOActorModel`。然后，这个`ScriptableObject`应用它持有的变量值，并将它们应用到`EnemyWave`脚本的变量中。

1.  仍然在`EnemyWave`脚本中，添加`Die`方法及其内容：

    ```cs
    public void Die()
          {
              Destroy(this.gameObject);
          }
    ```

如果您一直跟随着，另一个熟悉的方法是`Die`方法，该方法在玩家摧毁敌人时被调用。

1.  将 Unity 的`OnTriggerEnter`函数添加到`EnemyWave`脚本中：

    ```cs
    void OnTriggerEnter(Collider other)
          {
            // if the player or their bullet hits you.
            if (other.tag == "Player")
            {
               if (health >= 1)
               {
                  health -= other.GetComponent<IActorTemplate>
                    ().SendDamage();
               }
               if (health <= 0)
               {
                  Die();
               }
             }
           }

    ```

Unity 自带的`OnTriggerEnter`函数将检查它们是否与玩家发生碰撞，如果是，将发送伤害，并且敌人将使用`Die`方法摧毁自己。

1.  继续输入`TakeDamage`和`SendDamage`方法：

    ```cs
    public void TakeDamage(int incomingDamage)
          {
            health -= incomingDamage;
          }
          public int SendDamage()
          {
            return hitPower;
          }
    ```

来自`IActorTemplate`接口的另一组常见方法是，从`EnemyWave`脚本发送和接收伤害。

接下来是`Attack`方法，它控制敌人的移动/攻击。该方法在每一帧的`Update`函数中被调用。

使用这个攻击，我们将使敌人以波浪动画（类似于蛇）的形式从右向左移动，而不是直接从右向左移动。以下图像显示了我们的敌人以波浪线从右向左移动：

![Figure 2.51 – 敌人的波浪攻击模式![img/Figure_2.51_B18381.jpg](img/Figure_2.51_B18381.jpg)

Figure 2.50 – 敌人的波浪攻击模式

1.  将以下`Attack`方法代码输入到`EnemyWave`脚本中：

    ```cs
    public void Attack()
          {
            time += Time.deltaTime;
            sineVer.y = Mathf.Sin(time * verticalSpeed) *     verticalAmplitude;
            transform.position = new Vector3(transform.            position.x + travelSpeed * Time.deltaTime,
            transform.position.y + sineVer.y,
            transform.position.z);
          }}
    ```

`Attack`方法从收集在标记为`time`的`float`变量中的`Time.deltaTime`开始。

然后，我们使用 Unity 的一个预制函数，该函数返回一个正弦值（[`docs.unity3d.com/ScriptReference/Mathf.Sin.html`](https://docs.unity3d.com/ScriptReference/Mathf.Sin.html)），使用我们的`time`变量，乘以`verticalSpeed`变量中的设置速度，然后乘以结果乘以`verticalAmplitude`。

最终结果存储在`Vector3` *y*轴上。这基本上会使我们的敌人飞船上下移动。`verticalSpeed`参数设置其速度，而`verticalAmplitude`则改变其上下移动的距离。

然后，我们执行与`PlayerBullet`类似的任务，使敌人飞船沿着*x*轴移动，并且我们还添加了正弦计算来改变其`Y`位置，使其上下移动。

在我们结束这一章之前，请确保保存脚本。

在我们总结之前，请在编辑器中点击**播放**，希望一切顺利的话，你将能够驾驶一艘飞船在**游戏**窗口的边界内飞行；敌人将会从屏幕右侧漂浮进入并向左移动；你将能够用你的子弹摧毁这些敌人。如果敌人与你接触，它们也会摧毁你。最后，我们的**层次结构**窗口在游戏前后都整洁且结构良好。以下截图展示了我所解释的内容：

![图 2.52 – 包含当前游戏玩法和层次结构游戏对象结构的游戏窗口]

![图 2.52 – Figure_2.52_B18381.jpg]

图 2.51 – 包含当前游戏玩法和层次结构游戏对象结构的游戏窗口

你已经做了很多！好消息是，你刚刚征服了本书中最大的章节之一——我知道这很巧妙。但我们已经有了我们游戏的核心，最重要的是，我们已经覆盖了 Unity 程序员考试的大部分内容。

可以理解的是，你可能已经在路上遇到了一些可能的问题，你可能感到卡住了。如果这种情况发生，请不要担心——检查本章的`Complete`文件夹以加载 Unity 项目，并将该文件夹中的代码与你的代码进行比较以进行双重检查。确保你的场景中有正确的游戏对象，检查正确的游戏对象是否被标记，检查你的**球体**碰撞器的半径大小，如果你在**控制台**窗口中看到任何错误或警告，双击它们，它们将带你到导致问题的代码。

让我们总结本章，并谈谈到目前为止我们的游戏。

# 摘要

我们已经完成了本章的内容，我们已经征服了我们游戏框架的大部分内容，正如我们可以在以下图表中看到的那样：

![图 2.53 – Killer Wave UML]

![图 2.53 – Figure_2.53_B18381.jpg]

图 2.52 – Killer Wave UML

我们创建了一个游戏框架，无论我们为游戏添加 1 个还是 1000 个更多的敌人，都需要进行很少的更改。这种使用可重用代码和`ScriptableObject`的一些好处是，它将使非程序员受益，节省时间，并防止合作者陷入代码的困境。

我们还使得，如果我们想要添加更多的`EnemySpawner`点，我们可以将更多的预制体拖放到我们的场景中，并更新其`ScriptableObject`来更改敌人，而无需在精确的`Vector3`位置编码。

我们已经涵盖了其他常见的 Unity 功能，包括实例化游戏对象，如敌人和玩家子弹。

在下一章中，我们将介绍以下脚本：

+   `ScoreManager`：当一个敌人被摧毁时，玩家将获得分数。

+   `ScenesManager`：如果玩家死亡，将扣除一条生命；如果玩家失去所有生命，则关卡将重置。

+   `Sounds`：我们的飞船和子弹也将添加声音。

最后，我们将更新我们代码的整体结构。
