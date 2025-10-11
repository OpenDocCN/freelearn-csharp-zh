# 第三章项目 B – 太空射击游戏

本章现在进入了一个新的领域，因为我们开始开发我们的第二个游戏，这是一个双摇杆太空射击游戏。双摇杆类型简单指的是任何玩家输入动作跨越两个维度或轴的游戏，通常一个轴用于移动，一个轴用于旋转。例如，双摇杆游戏包括*Zombies Ate My Neighbors*和*Geometry Wars*。我们的游戏将大量依赖 C#编程，正如我们将看到的。这样做的主要目的是通过示例展示，即使不使用编辑器和关卡构建工具，也可以通过 Unity 过程（即通过脚本）实现多少。我们仍然会在一定程度上使用这些工具，但在这里不会过多，这是一个故意的而不是偶然的选择。因此，本章假设您不仅完成了前两个章节中创建的游戏项目，而且对 C#脚本有良好的、基本的知识，尽管不一定是在 Unity 中。所以，让我们卷起袖子，如果有，开始制作双摇杆射击游戏。本章还涵盖了以下以及其他重要主题：

+   生成和预制体

+   双摇杆控制和轴向移动

+   玩家控制器和射击机制

+   基本敌人移动和 AI

### 注意

记住，以抽象的方式看待这里创建的游戏及其相关工作，即作为具有多种应用的通用工具和概念。对于您自己的项目，您可能不想制作双摇杆射击游戏，这完全可以。我无法知道您想要制作的所有类型的游戏。然而，将这里使用到的想法和工具视为可转移的，作为您可以创造性地用于您自己游戏的工具，这一点非常重要。当使用 Unity 或任何引擎工作时，能够看到这一点非常重要。

# 展望未来 – 完成的项目

在深入探讨双摇杆射击游戏之前，让我们先看看完成的项目的样子以及它是如何工作的。见*图 3.1*。即将创建的游戏将只包含一个场景。在这个场景中，玩家控制一艘可以射击迎面敌人的宇宙飞船。方向键和 WASD 键可以移动飞船在关卡中的位置，并且它总是会转向面对鼠标指针。点击左鼠标按钮将发射弹药。

![展望未来 – 完成的项目](img/B05118_03_01.png.jpg)

图 3.1：完成的双摇杆射击游戏

### 注意

如本章和下一章所讨论的，完成的`TwinStickShooter`项目可以在本书配套文件中的`Chapter03/TwinStickShooter`文件夹中找到。

本游戏的多数资源（包括声音和纹理）均来源于免费可访问的网站，[OpenGameArt.org](http://OpenGameArt.org)。在这里，您可以找到许多通过公共领域或创意共享许可或其他许可方式提供的游戏资源。

# 开始制作太空射击游戏

要开始，创建一个没有任何包或特定资源的空白 Unity 3D 项目。有关创建新项目的详细信息，请参阅第一章，*硬币收集游戏 – 第一部分*。这次我们将从头开始编写所有代码。一旦生成了一个项目，就创建一些基本的文件夹来从一开始就组织和结构化项目资源。这对于你在工作中跟踪文件非常重要。为`Textures`、`Scenes`、`Materials`、`Audio`、`Prefabs`和`Scripts`创建文件夹。参见*图 3.2*：

![开始制作太空射击游戏](img/B05118_03_02.png.jpg)

图 3.2：创建文件夹以进行结构和组织

接下来，我们的游戏将依赖于一些图形和音频资源。这些资源包含在本书的配套文件中，位于`Chapter03/Assets`文件夹，也可以从[OpenGameArt.org](http://OpenGameArt.org)在线下载。让我们从玩家飞船、敌对飞船和星场背景的纹理开始。将`Textures`从 Windows 资源管理器或 Finder 拖放到`Textures`文件夹中的 Unity **项目**面板。Unity 会自动导入和配置纹理。*见图 3.3*：

![开始制作太空射击游戏](img/B05118_03_03.png.jpg)

图 3.3：导入飞船、敌人、星背景和弹药纹理资源

### 注意

提供的资源的使用是可选的。如果你愿意，可以创建自己的。只需将你自己的纹理拖放到包含的资产的位置，你仍然可以很好地跟随教程。

默认情况下，Unity 将图像文件导入为常规纹理，用于 3D 对象，并假设它们的像素尺寸是 2 的幂次大小（4、8、16、32、64、128、256 等等）。如果大小实际上不是这些之一，那么 Unity 将放大或缩小纹理到最近的合法大小。然而，对于应该以原始（导入）大小显示，没有任何缩放或自动调整的 2D 俯视太空射击游戏来说，这不是适当的行为。为了解决这个问题，选择所有导入的纹理，从**对象检查器**中，将它们的**纹理类型**从**纹理**更改为**精灵（2D 和 UI）**。一旦更改，点击**应用**按钮更新设置，纹理将保留其导入的尺寸。参见*图 3.4*：

![开始制作太空射击游戏](img/B05118_03_04.png.jpg)

图 3.4：更改导入纹理的纹理类型

在将**纹理类型**设置更改为**精灵（2D 和 UI）**后，也请从**生成 Mip 映射**框中取消勾选，以防此框被启用。这将防止 Unity 根据场景中纹理与摄像机的距离自动降低纹理质量。这确保了您的纹理保持最高质量。有关 2D 纹理设置和 Mip 映射的更多信息，请参阅在线 Unity 文档[`docs.unity3d.com/Manual/class-TextureImporter.html`](http://docs.unity3d.com/Manual/class-TextureImporter.html)。*见图 3.5*：

![开始使用太空射击游戏](img/B05118_03_05.png.jpg)

图 3.5：从导入的纹理中移除 MipMapping

现在您可以将纹理轻松拖放到场景中，将它们作为精灵对象添加。您不能从**项目**面板拖放到视图中，但可以从**项目**面板拖放到**层次结构**面板。当您这样做时，纹理将自动作为精灵对象添加到**场景**中。在我们创建飞船对象的过程中，我们将频繁使用这个功能。*见图 3.6*：

![开始使用太空射击游戏](img/B05118_03_06.png.jpg)

图 3.6：将精灵添加到场景中

接下来，让我们导入音乐和音效，这些内容也包含在本书的配套文件中，位于`Chapter03/Assets/Audio`文件夹中。这些资产是从[OpenGameArt.org](http://OpenGameArt.org)下载的。要导入音频，只需将文件从 Windows 资源管理器或 Mac Finder 拖放到**项目**面板中。当您这样做时，Unity 会自动导入并配置这些资产。您可以通过在**对象检查器**的预览工具栏中按播放来在 Unity 编辑器内测试音频。*见图 3.7*：

![开始使用太空射击游戏](img/B05118_03_07.png.jpg)

图 3.7：在对象检查器中预览音频

与纹理文件一样，Unity 使用一组默认参数导入音频文件。这些参数通常适用于短音效，如脚步声、枪声和爆炸声，但对于较长的轨道，如音乐，它们可能存在问题，会导致长时间的水平加载时间。为了解决这个问题，在**项目**面板中选择音乐轨道，并在**对象检查器**中禁用**预加载音频数据**复选框。从**加载类型**下拉菜单中选择**流式传输**选项。这确保音乐轨道以流式传输而不是在关卡启动时完全加载到内存中。*见图 3.8*：

![开始使用太空射击游戏](img/B05118_03_08.png.jpg)

图 3.8：配置音乐轨道以进行流式传输

# 创建玩家对象

我们现在已导入大多数双摇杆射击游戏资产，并准备好创建玩家飞船对象，即玩家将控制并移动的对象。创建此对象可能看似只是简单地从**项目**面板将相关的玩家精灵拖放到场景中这么简单的事情，但实际上并非如此。玩家是一个具有许多不同行为的复杂对象，正如我们将看到的。因此，在创建玩家时需要更加小心。要开始，通过从应用程序菜单导航到**GameObject** | **Create Empty** 在场景中创建一个空游戏对象，并将对象命名为`Player`。参见*图 3.9*：

![创建玩家对象](img/B05118_03_09.png.jpg)

图 3.9：开始创建玩家

新创建的对象可能或可能不在世界原点 *(0, 0, 0)* 处居中，其旋转属性在 **X**、**Y** 和 **Z** 轴上可能不一致。为确保完全归零变换，您可以通过在**对象检查器**中直接输入值来手动将这些值设置为`0`。然而，您可以通过单击**变换**组件左上角的齿轮图标并从上下文菜单中选择**重置**来自动将它们全部设置为`0`。参见*图 3.10*：

![创建玩家对象](img/B05118_03_10.png.jpg)

图 3.10：重置变换组件

接下来，将`Player`飞船精灵（位于`Textures`文件夹中）从**项目**面板拖放到**层次**面板，使其成为空玩家对象的子对象。然后，将飞船精灵在**X**轴上旋转`90`度，在**Y**轴上旋转`-90`度。这使得精灵朝向其父对象的正向向量，并在地面平面上展开。游戏摄像机将采用俯视视角。参见*图 3.11*：

![创建玩家对象](img/B05118_03_11.png.jpg)

图 3.11：对齐玩家飞船

您可以通过选择`Player`对象并查看蓝色正向向量箭头来确认飞船精灵相对于其父对象的定位是否正确。飞船精灵的前部和蓝色正向向量应该指向同一方向。如果不是，则继续将精灵旋转 90 度，直到它们对齐。这在编写玩家移动代码，使飞船朝向其注视的方向移动时将非常重要。参见*图 3.12*：

![创建玩家对象](img/B05118_03_12.png.jpg)

图 3.12：蓝色箭头被称为正向向量

接下来，`Player` 对象应该对物理力做出反应，也就是说，`Player` 对象是实心的，会受到物理力的影响。它必须与其他固体碰撞，并在被击中时受到敌人弹药的伤害。为此，应向 `Player` 对象添加两个额外的组件，具体来说，是一个 **Rigidbody** 和 **Collider**。要做到这一点，选择 `Player` 对象（而不是 `Sprite` 对象），从应用程序菜单导航到 **Component** | **Physics** | **Rigidbody**。然后，从菜单中选择 **Component** | **Physics** | **Capsule Collider**。这会添加一个 **Rigidbody** 和 **Collider**。参见 *图 3.13*：

![创建玩家对象](img/B05118_03_13.png.jpg)

图 3.13：向玩家对象添加 Rigidbody 和 Capsule Collider

**Collider** 组件用于近似对象的体积，而 **Rigibody** 组件则使用 **Collider** 来确定物理力应该如何真实地应用。让我们稍微调整一下 **Capsule Collider**，因为默认设置通常与预期的 `Player` 精灵不匹配。具体来说，调整 **方向**、**半径** 和 **高度** 值，直到 **Capsule** 包围 `Player` 精灵并代表玩家的体积。参见 *图 3.14*：

![创建玩家对象](img/B05118_03_14.png.jpg)

图 3.14：调整飞船 Capsule Collider

默认情况下，`Rigidbody` 组件被配置为近似受重力影响的对象，这些对象会落到地面上，撞击并反应场景中的其他固体。这对于在空中飞行的飞船来说是不合适的。因此，应该调整 `Rigidbody`。具体来说，取消勾选 **Use Gravity** 复选框以防止对象落到地面上。此外，启用 **Freeze Position** **Y** 复选框和 **Freeze Rotation** **Z** 复选框以防止飞船在 2D 俯视游戏中沿不希望轴移动和旋转。参见 *图 3.15*：

![创建玩家对象](img/B05118_03_15.png.jpg)

图 3.15：配置玩家飞船的 Rigidbody 组件

优秀的工作！我们现在已经成功配置了玩家飞船对象。当然，它仍然不会移动或做任何特定的游戏动作。这仅仅是因为我们还没有添加任何代码。这正是我们接下来要做的——使玩家对象对用户输入做出反应。

# 玩家输入

`Player`对象现在已在场景中创建，配置了**Rigidbody**和**Collider**组件。然而，此对象不会对玩家控制做出反应。在双摇杆射击游戏中，玩家在两个轴向上提供输入，通常可以射击武器。这通常意味着 WASD 键盘按钮控制玩家向上、向下、向左和向右移动。此外，鼠标移动控制玩家查看和瞄准的方向，而左鼠标按钮通常用于射击武器。这是我们游戏所需的控制方案。为了实现这一点，我们需要创建一个`PlayerController`脚本文件。在**项目**面板的`Scripts`文件夹上右键单击，创建一个名为`PlayerController.cs`的新 C#脚本文件。参见*图 3.16*：

![玩家输入](img/B05118_03_16.png.jpg)

图 3.16：创建玩家控制器 C# 脚本文件

在`PlayerController.cs`脚本文件中，以下代码（如*代码示例 3.1*所示）应该被包含。注释跟在此示例之后：

```cs
//------------------------------
using UnityEngine;
using System.Collections;
//------------------------------
public class PlayerController : MonoBehaviour
{
  //------------------------------
  private Rigidbody ThisBody = null;
  private Transform ThisTransform = null;

  public bool MouseLook = true;
  public string HorzAxis = "Horizontal";
  public string VertAxis = "Vertical";
  public string FireAxis = "Fire1";
  public float MaxSpeed = 5f;

  //------------------------------
  // Use this for initialization
  void Awake ()
  {
    ThisBody = GetComponent<Rigidbody>();
    ThisTransform = GetComponent<Transform>();
  }
  //------------------------------
  // Update is called once per frame
  void FixedUpdate ()
  {
    //Update movement
    float Horz = Input.GetAxis(HorzAxis);
    float Vert = Input.GetAxis(VertAxis);
    Vector3 MoveDirection = new Vector3(Horz, 0.0f, Vert);
    ThisBody.AddForce(MoveDirection.normalized * MaxSpeed);

    //Clamp speed
    ThisBody.velocity = new Vector3(Mathf.Clamp(ThisBody.velocity.x, -MaxSpeed, MaxSpeed),
      Mathf.Clamp(ThisBody.velocity.y, -MaxSpeed, MaxSpeed),
      Mathf.Clamp(ThisBody.velocity.z, -MaxSpeed, MaxSpeed));

    //Should look with mouse?
    if(MouseLook)
    {
      //Update rotation - turn to face mouse pointer
      Vector3 MousePosWorld = Camera.main.ScreenToWorldPoint(new Vector3(Input.mousePosition.x,Input.mousePosition.y, 0.0f));
        MousePosWorld = new Vector3(MousePosWorld.x, 0.0f, MousePosWorld.z);
      //Get direction to cursor
      Vector3 LookDirection = MousePosWorld - ThisTransform.position;

      //FixedUpdate rotation
      ThisTransform.localRotation = Quaternion.LookRotation(LookDirection.normalized,Vector3.up);
    }

  }
}
//------------------------------
```

## 代码示例 3.1

以下要点总结了代码示例：

+   `PlayerController`类应该附加到场景中的`Player`对象上。总体而言，它接受玩家的输入并将控制太空船的移动。

+   当对象在关卡开始时创建时，会调用一次`Awake`函数。在此函数期间，检索了两个组件，即用于控制器玩家旋转的**Transform**组件和用于控制器玩家移动的**Rigidbody**组件。**Transform**组件可以通过**Position**属性来控制玩家移动，但这会忽略碰撞和固体物体。相比之下，**Rigidbody**组件可以防止玩家对象穿过其他固体物体。

+   `FixedUpdate`函数在物理系统每次更新时都会被调用一次，这是每秒固定次数的调用。`FixedUpdate`与`Update`不同，`Update`函数每次帧调用一次，并且会根据帧率的变化而变化。如果你需要通过物理系统控制对象，使用如**Rigidbody**之类的组件，那么你应该始终在`FixedUpdate`中这样做，而不是在`Update`中。这是 Unity 的一个约定，你应该记住以获得最佳效果。

+   `Input.GetAxis` 函数在每次 `FixedUpdate` 调用中都会被调用，用于从输入设备（如键盘或游戏手柄）读取轴向输入数据。此函数从两个命名轴读取数据，`Horizontal`（左右）和`Vertical`（上下）。这些轴在一个归一化空间中工作，范围从 *-1* 到 *1*。这意味着当按下并保持左键时，`Horizontal` 轴返回 *-1*，而当按下并保持右键时，`Horizontal` 轴返回 *1*。值为 *0* 表示没有按下相关键或同时按下左右键，相互抵消。对于 `Vertical` 轴，向上表示 *1*，向下表示 *-1*，没有按键按下对应于 *0*。有关 `GetAxis` 函数的更多信息，可以在 Unity 文档的在线文档中找到，网址为 [`docs.unity3d.com/ScriptReference/Input.GetAxis.html`](http://docs.unity3d.com/ScriptReference/Input.GetAxis.html)。

+   `Rigidbody.AddForce` 函数用于对 `Player` 对象施加物理力，使其沿特定方向移动。`AddForce` 编码一个速度，通过特定的强度将对象移动到特定方向。方向编码在 `MoveDirection` 向量中，该向量基于来自 `Horizontal` 和 `Vertical` 轴的玩家输入。这个方向乘以我们的最大速度，以确保对象以所需的速度移动。有关 `AddForce` 的更多信息，请参阅 Unity 在线文档，网址为 [`docs.unity3d.com/ScriptReference/Rigidbody.AddForce.html`](http://docs.unity3d.com/ScriptReference/Rigidbody.AddForce.html)。

+   `Camera.ScreenToWorldPoint` 函数用于将游戏窗口中鼠标光标的屏幕位置转换为游戏世界中的位置，为玩家提供一个目标目的地进行观察。此代码负责使玩家始终看向鼠标光标。然而，正如我们很快将看到的，需要对代码进行一些调整才能使其正常工作。有关 `ScreenToWorldPoint` 的更多信息，请参阅 Unity 在线文档，网址为 [`docs.unity3d.com/ScriptReference/Camera.ScreenToWorldPoint.html`](http://docs.unity3d.com/ScriptReference/Camera.ScreenToWorldPoint.html)。

# 配置游戏相机

上述代码允许您控制 `Player` 对象，但存在一些问题。其中之一是玩家似乎没有面向鼠标光标的位置，尽管我们的代码旨在实现这种行为。原因是默认情况下，相机没有配置为适用于俯视 2D 游戏所需的配置。我们将在本节中修复这个问题。要开始，场景相机应该以俯视视角查看场景。为此，通过单击 **ViewCube**（位于 **Scene** 视口右上角的向上箭头），将 **Scene** 视口切换到俯视 2D 视图。这将切换您的视口到俯视图。参见 *图 3.17*：

![配置游戏相机](img/B05118_03_17.png.jpg)

图 3.17：视图立方体可以改变视口视角

你可以看到视口处于俯视图，因为视图立方体将列出**Top**作为当前视图。见图 3.18：

![配置游戏相机](img/B05118_03_18.png.jpg)

图 3.18：场景视口中的俯视图

从这里，你可以使场景相机与视口相机完全一致，为你提供游戏中的即时俯视图。为此，在**场景**（或从**层次**面板）中选择**相机**，然后从应用程序菜单中选择**GameObject** | **Align With View**。见图 3.19：

![配置游戏相机](img/B05118_03_19.png.jpg)

图 3.19：将相机与场景视口对齐

这使你的游戏看起来比以前好得多，但仍然存在问题。当游戏运行时，飞船仍然没有按照预期看向鼠标光标。这是因为相机是**透视**相机，屏幕点与世界点之间的转换导致了意外的结果。我们可以通过将相机更改为**正交**相机来解决这个问题，这是一个真正的 2D 相机，它不允许透视失真。为此，在场景中选择**相机**，然后在**对象检查器**中，将**投影**设置从**透视**更改为**正交**：

![配置游戏相机](img/B05118_03_20.png.jpg)

图 3.20：将相机切换到正交模式

每个正交相机在**对象检查器**中都有一个**大小**字段，而透视相机则没有。该字段控制世界视图中多少单位对应屏幕上的像素。我们希望世界单位与像素之间有一个 1:1 的比例或关系，以确保我们的纹理以正确的尺寸显示，并且光标移动产生预期效果。我们游戏的目标分辨率将是全高清，即 1920 x 1080，其宽高比为 16:9。对于这个分辨率，正交的**大小**应该是`5.4`。这个值的原因超出了本书的范围，但得出这个值的公式是*屏幕高度（以像素为单位）/ 2 / 100*。因此，*1080 / 2 / 100 = 5.4*。见图 3.21：

![配置游戏相机](img/B05118_03_21.png.jpg)

图 3.21：为 1:1 像素到屏幕比例更改正交大小

最后，确保你的**游戏**标签视图配置为以**16:9**宽高比显示游戏。如果不是，点击**游戏**视图左上角的宽高比下拉列表，并选择**16:9**选项。见图 3.22：

![配置游戏相机](img/B05118_03_22.png.jpg)

图 3.22：以 16:9 宽高比显示游戏

现在尝试运行游戏，你将有一个基于 WASD 输入移动并转向面对鼠标光标的玩家飞船。干得好！见图 3.23。游戏真的开始成形了。然而，还有很多工作要做。

![配置游戏相机](img/B05118_03_23.png.jpg)

图 3.23：转向面对光标！

# 边界锁定

在预览到目前为止的游戏时，飞船可能看起来太大。我们可以通过更改`Player`对象的比例来轻松解决这个问题。我在**X**、**Y**和**Z**轴上使用了`0.5`的值。见*图 3.24*。然而，即使比例更合理，仍然存在一个问题。具体来说，玩家可以无限制地移动到屏幕边界之外。这意味着玩家可以飞到远处，消失在视野中，并且永远不再被看到。相反，相机应该保持静止，玩家的移动应该限制在相机视图或边界内，以确保它永远不会退出视图。

![边界锁定](img/B05118_03_24.png.jpg)

图 3.24：调整玩家大小

实现边界锁定有多种方法，其中大多数涉及脚本编写。一种方法是将`Player`对象的位置值简单地限制在指定的范围内，即最小值和最大值。考虑一下名为`BoundsLock`的新 C#类，如*代码示例 3.2*所示。此脚本文件应附加到玩家身上。

```cs
//------------------------------
using UnityEngine;
using System.Collections;
//------------------------------
public class BoundsLock : MonoBehaviour 
{
  //------------------------------
  private Transform ThisTransform = null;
  public Vector2 HorzRange = Vector2.zero;
  public Vector2 VertRange = Vector2.zero;
  //------------------------------
  // Use this for initialization
  void Awake ()
  {
    ThisTransform = GetComponent<Transform>();
  }
  //------------------------------
  // Update is called once per frame
  void LateUpdate () 
  {
    //Clamp position
    ThisTransform.position = new Vector3(Mathf.Clamp(ThisTransform.position.x, HorzRange.x, HorzRange.y),
    ThisTransform.position.y,
    Mathf.Clamp(ThisTransform.position.z, VertRange.x, VertRange.y));
  }
  //------------------------------
}
//------------------------------
```

## 代码示例 3.2

以下是对代码示例的总结：

+   `LateUpdate`函数总是在所有`FixedUpdate`和`Update`调用之后被调用，允许对象在渲染到屏幕之前修改其位置。有关`LateUpdate`的更多信息，请参阅[`docs.unity3d.com/ScriptReference/MonoBehaviour.LateUpdate.html`](http://docs.unity3d.com/ScriptReference/MonoBehaviour.LateUpdate.html)。

+   `Mathf.Clamp`函数确保指定的值在最小值和最大值范围内。

+   要使用`BoundsLock`脚本，只需将文件拖放到`Player`对象上，并指定其位置的最小值和最大值。这些值以世界坐标指定，可以通过临时移动`Player`对象到相机的极限位置并记录其从**变换**组件的位置来确定：

![代码示例 3.2](img/B05118_03_25.png.jpg)

图 3.25：设置边界锁定

现在通过工具栏上的播放按钮运行游戏测试。玩家飞船应该保持在视野中，并且无法移出屏幕。太棒了！

# 健康值

玩家飞船和敌人都需要健康值。健康值是衡量角色在场景中的存在感和合法性的指标，通常以 0-100 之间的值来衡量。0 表示死亡，100 表示满血。虽然健康值在许多方面都是针对每个实例特定的（玩家有一个独特的健康条，每个敌人也有自己的），但玩家和敌人健康值在行为上有很多共同之处，因此将健康值编码为一个单独的组件和类，可以附加到所有需要健康值的对象上是有意义的。考虑*代码示例 3.3*，它应附加到玩家和所有敌人或需要健康值的对象上。注释如下：

```cs
using UnityEngine;
using System.Collections;
//------------------------------
public class Health : MonoBehaviour
{
  public GameObject DeathParticlesPrefab = null;
  private Transform ThisTransform = null;
  public bool ShouldDestroyOnDeath = true;
  //------------------------------
  void Start()
  {
    ThisTransform = GetComponent<Transform>();
  }
  //------------------------------
  public float HealthPoints
  {
    get
    {
      return _HealthPoints;
    }

    set
    {
      _HealthPoints = value;

      if(_HealthPoints <= 0)
      {
        SendMessage("Die", SendMessageOptions.DontRequireReceiver);

        if(DeathParticlesPrefab != null)
          Instantiate(DeathParticlesPrefab, ThisTransform.position, ThisTransform.rotation);

        if(ShouldDestroyOnDeath)
          Destroy(gameObject);
      }
    }
  }
  //------------------------------
  [SerializeField]
  private float _HealthPoints = 100f;
}
//------------------------------
```

## 代码示例 3.3

以下是对代码示例的总结：

+   `Health` 类通过一个 `private` 变量 `_HealthPoints` 维护对象的生命值，该变量通过 C# 属性 `HealthPoints` 访问。这个属性具有 `get` 和 `set` 访问器，用于返回和设置 `Health` 变量。

+   `_HealthPoints` 变量被声明为 `SerializedField`，这使得其值在 **Inspector** 中可见。这有助于我们在运行时查看玩家的生命值，并调试和测试代码的效果。

+   `Health` 类是事件驱动编程的一个例子。这是因为该类本可以在 `Update` 函数中持续检查对象的生命状态；检查对象的生命值是否下降到 *0* 以判断其是否已死亡。相反，死亡检查是在 C# 属性 `set` 方法中进行的。这样做是有道理的，因为 `set` 是生命值唯一可能改变的地方。这意味着 Unity 在每一帧中节省了大量工作。这是一个很好的性能提升！

+   `Health` 类使用 `SendMessage` 函数。这个函数允许你通过指定函数名作为字符串来调用任何附加到对象上的组件的任何公共函数。在这种情况下，一个名为 `Die` 的函数将在每个附加到对象上的组件上执行（如果存在这样的函数）。如果没有匹配名称的函数，则该组件不会发生任何操作。这是一个快速简单的方法，可以在不使用任何多态的情况下以类型无关的方式在对象上运行自定义行为。缺点是 `SendMessage` 在内部使用一个称为 `Reflection` 的过程，这个过程很慢，并且会降低性能。因此，`SendMessage` 应该仅在不频繁的情况下使用，例如仅在死亡事件和类似事件中，而不是每帧都使用。有关 `SendMessage` 的更多信息，可以在 Unity 在线文档中找到，网址为 [`docs.unity3d.com/ScriptReference/GameObject.SendMessage.html`](http://docs.unity3d.com/ScriptReference/GameObject.SendMessage.html)。

+   当生命值下降到 *0* 以下，触发死亡条件时，代码将实例化一个死亡粒子系统以显示死亡效果（关于这一点稍后会有更多说明）。

当 `Health` 脚本附加到玩家飞船上时，它会在 **Inspector** 中作为一个组件出现。它包含一个用于 **Death Particles Prefab** 的字段。这是一个可选字段（它可以设置为 null），用于指定在对象死亡时生成粒子系统的粒子系统。这使得在对象死亡时轻松创建爆炸或血液飞溅效果。请参见 *图 3.26*：

![代码示例 3.3](img/B05118_03_26.png.jpg)

图 3.26：附加 Health 脚本

# 死亡与粒子

在这个双摇杆射击游戏中，玩家和敌人都是宇宙飞船。当它们被摧毁时，应该以火球的形式爆炸。这确实是唯一可能令人信服的效果。为了实现爆炸效果，我们可以使用粒子系统。这简单指的是一种具有两个主要部分的特殊对象，即**软管**（或**发射器**）和**粒子**。发射器指的是将新粒子生成到世界中的部分，而粒子是许多小对象或碎片，一旦生成，就会移动并沿着自己的轨迹移动。简而言之，粒子系统非常适合创建雨、雪、雾、闪光和爆炸。我们可以通过使用菜单选项**GameObject** | **粒子系统**从头开始创建自己的粒子系统，或者我们可以使用 Unity 附带的所有预制的粒子系统。让我们使用一些预制的粒子系统。为此，通过从应用程序菜单导航到**资产** | **导入包** | **粒子系统**将`ParticleSystems`包导入到项目中。见*图 3.27*：

![死亡与粒子](img/B05118_03_27.png.jpg)

图 3.27：将粒子系统导入到项目中

在**导入**对话框出现后，请保持所有设置为默认值，并简单地点击**导入**以导入完整包，包括所有粒子系统。`ParticleSystems`将被添加到**项目**面板中的`标准资产` | `粒子系统` | `预制体`文件夹中。见*图 3.28*。您可以通过将每个预制体简单地拖放到场景中来测试每个粒子系统。请注意，您只能在**场景**视图中选择粒子系统时预览它。

![死亡与粒子](img/B05118_03_28.png.jpg)

图 3.28：导入到项目面板中的粒子系统

从 *图 3.28* 可以看出，**爆炸** 系统包含在默认资源中，这是一个好消息！为了测试，我们只需将爆炸拖放到场景中，在工具栏上按下播放，就可以看到爆炸效果。很好！我们几乎完成了，但还有一些工作要做。我们已经看到，有一个合适的粒子系统可用，并且我们可以直接将这个系统拖放到 **Inspector** 中的 **Health** 组件的 **Death Particles Prefab** 槽中。这在技术上是可以工作的：当玩家或敌人死亡时，爆炸系统将被生成，产生爆炸效果。然而，粒子系统永远不会被销毁！这是问题所在，因为，在每次敌人死亡时，都会生成一个新的粒子系统。这可能导致，在经历了多次死亡后，场景中充满了未使用的粒子系统。我们不希望这样；场景中充满了闲置的对象，这对性能和内存使用都是不利的。为了解决这个问题，我们将稍微修改爆炸系统，创建一个新的、修改过的预制件，以满足我们的需求。为了创建这个预制件，将现有的爆炸系统拖放到场景中的任何位置，并将其放置在世界原点。参见 *图 3.29*：

![死亡和粒子](img/B05118_03_29.png.jpg)

图 3.29：将爆炸系统添加到场景中进行修改

接下来，我们必须细化粒子系统，使其在实例化后不久自行销毁。通过从这个配置创建一个预制件，每个生成的爆炸最终都会自行销毁。为了使对象在指定的时间间隔后自行销毁，我们将创建一个新的 C# 脚本。我将把这个脚本命名为 `TimeDestroy.cs`。参考 *代码示例 3.4* 中的以下代码：

```cs
//------------------------------
using UnityEngine;
using System.Collections;
//------------------------------
public class TimedDestroy : MonoBehaviour 
{
  public float DestroyTime = 2f;

  //------------------------------
  // Use this for initialization
  void Start ()
  {
    Invoke("Die", DestroyTime);
  }

  void Die () 
  {
    Destroy(gameObject);
  }
  //------------------------------
}
//------------------------------
```

## 代码示例 3.4

以下要点总结了代码示例：

+   `TimeDestroy` 类简单地在指定的时间间隔 (`DestroyTime`) 过去后销毁它附加的对象。

+   `Invoke` 函数在 `Start` 事件中被调用。`Invoke` 将在指定的时间间隔过去后执行指定名称的函数一次，并且仅执行一次。间隔以秒为单位。

+   与 `SendMessage` 类似，`Invoke` 函数依赖于 `Reflection`。因此，为了获得最佳性能，应谨慎使用。

+   `Die` 函数将在指定的时间间隔后由 `Invoke` 执行，以销毁 `gameobject`（如粒子系统）。

现在，将 `TimedDestroy` 脚本文件拖放到场景中的爆炸粒子系统中，然后在工具栏上按下播放以测试代码是否工作，并且对象在指定的时间间隔后销毁，这个时间间隔可以从 **Inspector** 中进行调整。参见 *图 3.30*：

![代码示例 3.4](img/B05118_03_30.png.jpg)

图 3.30：将 TimeDestroy 脚本添加到爆炸粒子系统中

`TimeDestroy` 脚本应在延迟过期后移除爆炸粒子系统。因此，让我们从这个修改版本中创建一个新的独立预制体。为此，在**层次结构**面板中将爆炸系统重命名为 `ExplosionDestroy`，然后从**层次结构**拖放到 `Prefabs` 文件夹中的**项目**面板。Unity 自动创建一个新的预制体，代表修改后的粒子系统。请参阅 *图 3.31*：

![代码示例 3.4](img/B05118_03_31.png.jpg)

图 3.31：创建定时爆炸预制体

现在，将新创建的预制体从**项目**面板拖放到**对象检查器**中**玩家**的**健康**组件的**死亡粒子系统**槽位。这确保了当玩家死亡时预制体会被实例化。请参阅 *图 3.32*：

![代码示例 3.4](img/B05118_03_32.png.jpg)

图 3.32：配置健康脚本

如果你现在运行游戏，你会看到你不能启动玩家死亡事件来测试粒子系统生成。场景中没有任何东西可以摧毁或伤害玩家，而且你不能从**检查器**中手动将**健康**点数设置为`0`，这种方式会被 C# 属性的 `set` 函数检测到。然而，目前我们可以将一些测试死亡功能插入到 `Health` 脚本中，当按下空格键时触发立即死亡。请参阅 *代码示例 3.5* 以获取修改后的 `Health` 脚本：

```cs
//------------------------------
using UnityEngine;
using System.Collections;
//------------------------------
public class Health : MonoBehaviour
{
  public GameObject DeathParticlesPrefab = null;
  private Transform ThisTransform = null;
  public bool ShouldDestroyOnDeath = true;
  //------------------------------
  void Start()
  {
    ThisTransform = GetComponent<Transform>();
  }
  //------------------------------
  public float HealthPoints
  {
    get
    {
      return _HealthPoints;
    }

    set
    {
      _HealthPoints = value;

      if(_HealthPoints <= 0)
      {
        SendMessage("Die", SendMessageOptions.DontRequireReceiver);

        if(DeathParticlesPrefab != null)
          Instantiate(DeathParticlesPrefab, ThisTransform.position, ThisTransform.rotation);

        if(ShouldDestroyOnDeath)Destroy(gameObject);
      }
    }
  }
  //------------------------------
  void Update()
  {
    if(Input.GetKeyDown(KeyCode.Space))
      HealthPoints = 0;
  }
  //------------------------------
  [SerializeField]
  private float _HealthPoints = 100f;
}
//------------------------------
```

现在运行游戏，使用修改后的 `Health` 脚本，你可以通过按下键盘上的空格键来触发玩家的立即死亡。当你这样做时，玩家对象将被销毁，粒子系统也会生成，直到计时器将其销毁。做得好！我们现在有一个可玩、可控制的玩家角色，它支持健康和死亡功能。一切看起来都很顺利。请参阅 *图 3.33*：

![代码示例 3.4](img/B05118_03_33.png.jpg)

图 3.33：触发爆炸粒子系统

# 敌人

下一步是为玩家创建可以射击和摧毁的东西，这也可以摧毁我们，即敌人角色。这些以游荡飞船的形式出现，将定期生成到场景中，并跟随玩家，越来越近。本质上，每个敌人代表一个由多个行为共同作用而成的复杂体，这些应该作为单独的脚本实现。让我们逐一考虑它们：

+   **健康**：每个敌人支持健康功能。它们在场景开始时具有指定数量的健康值，当健康值降至 *0* 以下时将被销毁。我们已经有了一个 `Health` 脚本来处理这种行为。

+   **移动**：每个敌人将始终处于运动状态，沿着前进轨迹直线行进。也就是说，每个敌人将不断朝向其注视的方向前进。

+   **转向**：即使玩家移动，每个敌人也会旋转并转向玩家。这确保了敌人始终面向玩家，并且与移动功能结合，将始终朝向玩家移动。

+   **得分**：每个敌人在被摧毁时都会给玩家奖励一个分数值。因此，敌人的死亡会增加玩家的分数。

+   **伤害**：每个敌人在碰撞时都会对玩家造成伤害。敌人不能射击，但会在接近时伤害玩家。

现在我们已经确定了适用于敌人的行为范围，让我们在场景中创建一个敌人。我们将创建一个特定的敌人，从该敌人创建一个预制件，并将其用作实例化许多敌人的基础。首先，在场景中选择玩家角色，并使用 *Ctrl* + **D** 或从应用程序菜单中选择 **编辑** | **复制** 来复制对象。这最初创建了一个第二个玩家。参见 *图 3.34*：

![敌人](img/B05118_03_34.png.jpg)

图 3.34：复制玩家对象

将对象重命名为 `Enemy`，并确保它没有被标记为 `Player`，因为场景中应该只有一个带有 `Player` 标签的对象，即真正的玩家。此外，暂时禁用 `Player` 游戏对象，以便我们可以在 **场景** 选项卡中更清晰地关注 `Enemy` 对象。参见 *图 3.35*：

![敌人](img/B05118_03_35.png.jpg)

图 3.35：如果适用，从敌人中移除玩家标签

选择复制的敌人精灵子对象，并在 **对象检查器** 中点击 **Sprite** 字段，选择 **Sprite Renderer** 组件的新精灵。为敌人角色选择一个较暗的帝国飞船精灵，精灵将在视图中更新。参见 *图 3.36*：

![敌人](img/B05118_03_36.png.jpg)

图 3.36：为精灵渲染器组件选择精灵

在将精灵更改为敌人角色后，您可能需要调整旋转值，以确保精灵与父对象的向前向量对齐，确保精灵朝向与向前向量相同的方向。参见 *图 3.37*：

![敌人](img/B05118_03_37.png.jpg)

图 3.37：调整敌人精灵的旋转

现在，选择敌人的父对象，并移除 **Rigidbody**、**PlayerController** 和 `BoundsLock` 组件，但保留 **Health** 组件，因为敌人应该支持生命值。参见 *图 3.38*。此外，您可以随意调整 **Capsule Collider** 组件的大小，以更好地逼近 `Enemy` 对象。

![敌人](img/B05118_03_38.png.jpg)

图 3.38：调整敌人精灵的旋转

让我们从编写敌人代码开始，重点关注移动。具体来说，敌人应该以指定的速度持续向前移动。为了实现这一点，创建一个名为 `Mover.cs` 的新脚本文件。这个文件应该附加到 `Enemy` 对象上。该类的代码包含在 *代码示例 3.6* 中。

```cs
//------------------------------
using UnityEngine;
using System.Collections;
//------------------------------
public class Mover : MonoBehaviour
{
  //------------------------------
  private Transform ThisTransform = null;
  public float MaxSpeed = 10f;
  //------------------------------
  // Use this for initialization
  void Awake () 
  {
    ThisTransform = GetComponent<Transform>();
  }
  //------------------------------
  // Update is called once per frame
  void Update () 
  {
    ThisTransform.position += ThisTransform.forward * MaxSpeed * Time.deltaTime;
  }
  //------------------------------
}
//------------------------------
```

## 代码示例 3.6

以下要点总结了代码示例：

+   `Mover` 脚本以指定的速度（每秒 `MaxSpeed`）沿着其前进向量移动一个对象。为此，它使用 **Transform** 组件。

+   `Update` 函数负责更新对象的位置。简而言之，它将前进向量乘以对象速度，并将其加到现有位置上，以使对象沿着其视线移动得更远。`Time.deltaTime` 值用于使运动帧率独立——每秒移动对象而不是每帧移动。有关 `deltaTime` 的更多信息，请参阅在线 Unity 文档，网址为 [`docs.unity3d.com/ScriptReference/Time-deltaTime.html`](http://docs.unity3d.com/ScriptReference/Time-deltaTime.html)。

在工具栏上按播放按钮以测试运行您的代码。频繁测试此类代码总是好的实践。您的敌人可能移动得太慢或太快。如果是这样，停止播放以退出游戏模式，并在场景中选择敌人。从 **对象检查器** 中调整 **Mover** 组件的 **最大速度** 值。参见 *图 3.39*：

![代码示例 3.6](img/B05118_03_39.png.jpg)

图 3.39：调整敌人速度

除了直线移动外，敌人还应不断转向以面对玩家，无论他们移动到哪里。为了实现这一点，我们需要另一个与玩家控制器脚本类似工作的脚本文件。当玩家转向面对光标时，敌人转向面对玩家。这种功能应编码在一个名为 `ObjFace.cs` 的新脚本文件中。此脚本应附加到敌人上。参见 *代码示例 3.7*：

```cs
//------------------------------
using UnityEngine;
using System.Collections;
//------------------------------
public class ObjFace : MonoBehaviour
{
  //------------------------------
  public Transform ObjToFollow = null;
  public bool FollowPlayer = false;
  private Transform ThisTransform = null;
  //------------------------------
  // Use this for initialization
  void Awake () 
  {
    //Get local transform
    ThisTransform = GetComponent<Transform>();

    //Should face player?
    if(!FollowPlayer)return;

    //Get player transform
    GameObject PlayerObj = GameObject.FindGameObjectWithTag("Player");
    if(PlayerObj != null)
      ObjToFollow = PlayerObj.GetComponent<Transform>();
  }
  //------------------------------
  // Update is called once per frame
  void Update ()
  {
    //Follow destination object
    if(ObjToFollow==null)return;

    //Get direction to follow object
    Vector3 DirToObject = ObjToFollow.position - ThisTransform.position;

    if(DirToObject != Vector3.zero)
      ThisTransform.localRotation = Quaternion.LookRotation(DirToObject.normalized,Vector3.up);
  }
  //------------------------------
}
//------------------------------
```

## 代码示例 3.7

以下要点总结了代码示例：

+   `ObjFace` 脚本将始终旋转对象，使其前进向量指向场景中的目标点。

+   在 `Awake` 事件中，调用 `FindGameObjectWithTag` 函数以获取场景中标记为玩家的唯一对象的引用，这应该是玩家飞船。玩家代表敌人对象的默认注视目标。

+   `Update` 函数每帧自动调用一次，并从对象位置到目标位置生成一个位移向量，这代表了对象应该朝向的方向。`Quaternion.LookRotation` 函数接受一个方向向量，并将对象旋转以使前进向量与提供的方向对齐。这使对象始终朝向目标。有关 `LookRotation` 的更多信息，请参阅在线 Unity 文档，网址为 [`docs.unity3d.com/ScriptReference/Quaternion.LookRotation.html`](http://docs.unity3d.com/ScriptReference/Quaternion.LookRotation.html)。

看起来非常好！然而，在测试此代码之前，请确保场景中的`Player`对象被标记为**Player**，处于启用状态，并且敌方角色与玩家偏移。务必从**对象检查器**中的**Obj Face**组件启用**跟随玩家**复选框。当你这样做时，敌方角色将始终转向面对玩家。参见*图 3.40*：

![代码示例 3.7](img/B05118_03_40.png.jpg)

图 3.40：敌方飞船向玩家移动

现在，如果敌方角色最终与玩家发生碰撞，它应该造成伤害并可能杀死玩家。为了实现这一点，必须检测敌方角色和玩家之间的碰撞。让我们先配置敌方角色。选择**敌方**对象，并在**对象检查器**中，在**胶囊碰撞器**组件中启用**是触发器**复选框。这会将**胶囊碰撞器**组件更改为允许玩家和敌方角色之间进行真实交叠，并防止 Unity 阻止碰撞。参见*图 3.41*：

![代码示例 3.7](img/B05118_03_41.png.jpg)

图 3.41：将敌方碰撞器更改为触发器

接下来，我们将创建一个脚本，用于检测碰撞，并在发生碰撞时以及碰撞状态持续期间不断对玩家造成伤害。请参考以下代码（`ProxyDamage.cs`），该代码应附加到敌方角色上：

```cs
//------------------------------
using UnityEngine;
using System.Collections;
//------------------------------
public class ProxyDamage : MonoBehaviour
{
  //------------------------------
  //Damage per second
  public float DamageRate = 10f;
  //------------------------------
  void OnTriggerStay(Collider Col)
  {
    Health H = Col.gameObject.GetComponent<Health>();

    if(H == null)return;

    H.HealthPoints -= DamageRate * Time.deltaTime;
  }
  //------------------------------
}
//------------------------------
```

## 代码示例 3.8

以下要点总结了代码示例：

+   `ProxyDamage`脚本应附加到敌方角色上，并将对任何具有`Health`组件的碰撞对象造成伤害。

+   `OnTriggerStay`事件在交叠状态持续期间每帧被调用一次。在此函数中，`Health`组件的`HealthPoints`值会减少`DamageRate`（以每秒伤害量衡量）。

在将`ProxyDamage`脚本附加到敌方角色后，使用**对象检查器**设置**代理伤害**组件的**损伤率**。这表示在碰撞期间，每秒应该减少玩家多少健康值。为了增加挑战性，我将值设置为`100`健康点。参见*图 3.42*：

![代码示例 3.8](img/B05118_03_42.png.jpg)

图 3.42：设置代理伤害组件的损伤率

现在我们来测试一下。在工具栏上按播放，尝试玩家和敌方角色的碰撞。一秒后，玩家应该被摧毁。一切进展顺利。然而，为了使游戏更具挑战性，我们需要不止一个敌方角色。

# 敌方生成

为了使游戏级别既有趣又具有挑战性，我们需要的不仅仅是单个敌人。实际上，对于一个本质上无限的游戏，我们需要不断地添加敌人。这些敌人应该随着时间的推移逐渐添加。本质上，我们需要敌人的定期或间歇性生成，本节将添加这一功能。然而，在我们能够做到这一点之前，我们需要从敌人对象制作一个预制件。这可以轻松实现。在**层次**面板中选择敌人，然后将其拖放到`Prefabs`文件夹中的**项目**面板。这样就创建了一个`Enemy`预制件。参见*图 3.43*：

![敌人生成](img/B05118_03_43.png.jpg)

*图 3.43*：创建敌人预制件

现在，我们将创建一个新的脚本（`Spawner.cs`），该脚本将在玩家飞船指定半径内随时间生成新的敌人。这个脚本应该附加到场景中的一个新空游戏对象上。参见*代码示例 3.9*：

```cs
//------------------------------
using UnityEngine;
using System.Collections;
//------------------------------
public class Spawner : MonoBehaviour
{
  public float MaxRadius = 1f;
  public float Interval = 5f;
  public GameObject ObjToSpawn = null;
  private Transform Origin = null;
  //------------------------------
  void Awake()
  {
  Origin = GameObject.FindGameObjectWithTag("Player").GetComponent<Transform>();
  }
  //------------------------------
  // Use this for initialization
  void Start () 
  {
    InvokeRepeating("Spawn", 0f, Interval);
  }
  //------------------------------
  void Spawn () 
  {
    if(Origin == null)return;

    Vector3 SpawnPos = Origin.position + Random.onUnitSphere * MaxRadius;
    SpawnPos = new Vector3(SpawnPos.x, 0f, SpawnPos.z);
    Instantiate(ObjToSpawn, SpawnPos, Quaternion.identity);
  }
  //------------------------------
}
//------------------------------
```

## 代码示例 3.9

以下要点总结了代码示例：

+   `Spawner`类将在每个间隔内生成`ObjToSpawn`（一个预制件）的实例。间隔以秒为单位测量。生成的对象将在以中心点`Origin`为起点的随机半径内创建。

+   在`Start`事件期间，`InvokeRepeating`函数被调用，以在每个间隔内持续执行`Spawn`函数来生成新的敌人。

+   `Spawn`函数将在场景中创建敌人的实例，这些实例从起点以随机半径生成。一旦生成，敌人将像正常一样行动，朝向玩家进行攻击。

`Spawner`类是一个全局行为，适用于整个场景。它不依赖于特定的玩家，也不依赖于任何特定的敌人。因此，它应该附加到一个空的游戏对象上。通过从应用程序菜单中选择**GameObject** | **Create Empty**来创建一个这样的对象。将其命名为`Spawner`，并将`Spawner`脚本附加到它上。参见*图 3.44*：

![代码示例 3.9](img/B05118_03_44.png.jpg)

*图 3.44*：创建空游戏对象

将其添加到场景后，从**对象检查器**中，将`Enemy`预制件拖放到**Spawner**组件中的**Obj To Spawn**字段。将**间隔**设置为`2`秒，并将**最大半径**增加到`5`。参见*图 3.45*：

![代码示例 3.9](img/B05118_03_45.png.jpg)

*图 3.45*：为敌人对象配置 Spawner

现在（掌声），让我们尝试这个级别。在工具栏上按播放，进行游戏测试运行。你现在应该有一个带有完全可控制玩家角色的级别，周围是不断增长的追踪敌人飞船的军队！干得好！参见*图 3.46*：

![代码示例 3.9](img/B05118_03_46.png.jpg)

*图 3.46*：生成的敌人对象向玩家移动

# 摘要

做得很好，已经走到这一步了！太空射击游戏现在真的有形了，它拥有一个可控的玩家角色，该角色依赖于原生物理，双摇杆机制，敌舰，以及用于生成敌人的场景级生成器。所有这些元素加在一起仍然不能构成一个游戏：我们无法射击，无法增加分数，也无法摧毁敌人。这些问题需要解决，还有我们肯定会遇到的其他技术问题。尽管如此，我们现在有一个坚实的基础来继续前进，在下一章中，我们将完成射击游戏。
