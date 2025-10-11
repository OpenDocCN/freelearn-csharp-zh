# *第十章*：暂停游戏、调整声音和模拟测试

在本章中，我们将为我们的游戏添加背景音乐。然后，当关卡开始时，我们将使音乐淡入，当关卡完成时淡出，如果玩家死亡则停止。之后，我们将使用我们迄今为止学到的所有 UI 技能来创建一个暂停界面，并在其中添加一些滑动组件（这些组件将在下一章中用于音量控制）。在构建了暂停界面后，我们将通过冻结玩家、屏幕上的敌人、子弹和移动纹理来使游戏暂停。在暂停界面内，我们将给玩家提供继续游戏或退出的选项，以便游戏使用事件监听器回到标题屏幕，这些事件监听器我们在*第九章*中学习过，即*创建 2D 商店界面和游戏内 HUD*。最后，我们将提供一个包含 20 个问题的迷你模拟测试，涵盖我们从本章以及之前章节中学到的内容。

到本章结束时，我们将能够在脚本中直接修改`AudioSource`组件。我们将知道如何让每个 GameObject 在暂停界面中停止在屏幕上的移动。最后，我们将知道如何通过添加切换和滑动组件来创造更丰富的体验。

本章将涵盖以下主题：

+   应用和调整关卡音乐

+   创建暂停界面

+   添加游戏暂停按钮

+   模拟测试

在 Unity 程序员考试方面，下一节将列出本章将涵盖的核心目标。

# 本章涵盖的核心考试技能

以下是在本章中将涵盖的核心考试技能：

*编程核心交互：*

+   实现游戏对象和环境的行为和交互。

+   识别实现输入和控制的方 法。

*开发应用程序系统：*

+   应用程序界面流程，如菜单系统、UI 导航和应用设置。

*场景与环境设计编程：*

+   确定实现音频资源的脚本。

# 技术要求

本章的项目内容可以在[`github.com/PacktPublishing/Unity-Certified-Programmer-Exam-Guide-Second-Edition/tree/main/Chapter_10`](https://github.com/PacktPublishing/Unity-Certified-Programmer-Exam-Guide-Second-Edition/tree/main/Chapter_10)找到。

您可以在[`github.com/PacktPublishing/Unity-Certified-Programmer-Exam-Guide-Second-Edition`](https://github.com/PacktPublishing/Unity-Certified-Programmer-Exam-Guide-Second-Edition)下载每个章节的项目文件的全部内容。

本章的所有内容都包含在本章的`unitypackage`文件中，包括一个`Complete`文件夹，其中包含本章我们将执行的所有工作。

观看以下视频，了解*代码的实际应用*：[`bit.ly/3kjkSBW`](https://bit.ly/3kjkSBW)。

# 应用和调整关卡音乐

在本节中，我们将探讨如何将背景音乐添加到我们的游戏关卡中。我们还将更新我们的脚本，以便在游戏的不同阶段改变音乐音量。

在接下来的章节中，我们将进行以下操作：

+   将音乐添加到每个关卡。

+   当玩家完成关卡时，让音乐淡出。

+   如果玩家死亡，让音乐立即停止。

+   确保音乐只在关卡场景中播放，不要在其他场景中播放。

因此，让我们开始，将游戏音乐添加到`level1`场景中。

## 更新我们的 GameManager 预制件

在本节中，我们将更新`GameManager`游戏对象，使其在`AudioSource`组件中包含一个新的游戏对象（称为`LevelMusic`）作为子对象，并播放 MP3 文件。这种设置对于简单游戏来说很理想；否则，我们可能会增加另一个管理器，这对于更大、更复杂的游戏来说才合适。

要创建一个游戏对象并将音乐文件添加到其中，我们需要做以下操作：

1.  在 Unity 编辑器中，打开`bootUp`场景（`Assets /Scene`）。

1.  在**层次窗口**中右键点击**GameManager**，从下拉菜单中选择**音频 | 音频源**。

1.  将新游戏对象重命名为`LevelMusic`。以下截图显示了创建组件的游戏对象：

![图 10.1 – 创建音频源组件](img/Figure_10.01_B18381.jpg)

![图 10.1 – 创建音频源组件使用`LevelMusic`游戏对象仍然被选中，我们现在可以将我们的`lvlMusic` MP3 文件从**项目**窗口中的`Assets/Resources/Sound`拖动到**AudioClip**参数，如图下截图所示：![图 10.2 – 在音频源组件中将 lvlMusic MP3 添加到 AudioClip 字段](img/Figure_10.02_B18381.jpg)

图 10.2 – 在音频源组件中将 lvlMusic MP3 添加到 AudioClip 字段

现在是保存我们的`GameManager`预制件的好时机，在**层次窗口**中选择它，然后在**检查器**窗口的右上角点击**覆盖 | 应用所有**。

如果我们现在点击`level1`场景，游戏将开始播放音乐。这是因为，默认情况下，**音频源**组件设置为**唤醒时播放**。这是好的，但它不会停止播放，直到场景改变，这对于大多数游戏来说已经足够了。然而，我们希望通过脚本添加对音乐音量的控制。

在下一节中，我们将更新`ScenesManager`脚本，并控制音乐何时以及如何播放。

## 为我们的游戏音乐准备状态

在本节中，我们将确保我们的游戏音乐不再设置为默认的`ScenesManager`脚本，因为它相对连接。

要添加我们的三个音乐状态（播放、停止和淡出），我们需要做以下操作：

1.  在`ScenesManager`脚本（`Assets/Script`）。

1.  在`ScenesManager`脚本顶部，我们定义变量的地方，就在我们的`public enum Scenes`属性的作用域下方，输入以下`enum`及其三个状态：

    ```cs
      public MusicMode musicMode;
      public enum MusicMode
      {
        noSound, fadeDown, musicOn
      }
    ```

我们在*设置场景管理器脚本*部分介绍了枚举，见*第三章*，*管理脚本和进行模拟测试*；其原理与标记状态相同。对于我们的`enum`，我们给它分配了一个数据类型名为`MusicMode`。

现在我们已经为三种状态贴上了标签，我们需要将这些状态付诸实践。我们需要让我们的三种状态执行它们预期的动作：

+   `noSound`: 没有音乐播放。

+   `fadeDown`: 音乐的音量将淡至零。

+   `musicOn`: 音乐将播放，并将音量设置为最大。

在游戏的各个阶段，我们希望触发这些状态，而访问这些短状态集的最佳方式是使用 switch case 来过滤出每个结果。

现在，我们需要为我们的三种音乐状态添加一个`switch`语句。

仍然在`ScenesManager`脚本中，我们将添加一个`IEnumerator`，它将对任一状态进行操作。我们已经在*第二章*的*设置我们的敌人生成器脚本*部分介绍了`StartCoroutine`/`IEnumerator`，*添加和操作对象*。

因此，因为我们添加了`IEnumerator`，所以我们也需要添加一个额外的库来支持此功能：

1.  在`ScenesManager`脚本的最顶部，添加以下库：

    ```cs
    using System.Collections;
    ```

1.  我们的脚本现在支持协程和 IEnumerators。

我将把我的`IEnumerator`放在`Update`函数的作用域之外，命名为`MusicVolume`，它接受`MusicMode`数据类型，我们将称之为`musicMode`：

```cs
 IEnumerator MusicVolume(MusicMode musicMode)
  {
```

1.  在`MusicVolume` `IEnumerator`的作用域内，我们将从`switch`语句开始，并接收从`musicMode`引用发送过来的三个状态之一：

    ```cs
        switch (musicMode)
        {
    ```

1.  如果`musicMode`包含`noSound`状态，那么我们使用`GetComponentInChildren<AudioSource>()`来获取唯一的包含`AudioSource`的子游戏对象，即新创建的`LevelMusic`游戏对象。

1.  然后我们使用`Stop`函数停止音乐，然后跳出 case：

    ```cs
    case MusicMode.noSound :
          {
            GetComponentInChildren<AudioSource>().Stop();
            break;
          }
    ```

1.  下一个 case 是如果`musicMode`包含`fadeDown`状态。在这里，我们获取`LevelMusic`游戏对象的引用，并随时间减少其`volume`值：

    ```cs
     case MusicMode.fadeDown :
          {
            GetComponentInChildren<AudioSource>().volume -=
               Time.deltaTime/3;
            break;
          }
    ```

1.  第三个也是最后一个情况是`musicOn`；在 case 内部，我们首先检查是否已经将音频剪辑加载到了`AudioSource`中。如果没有音频剪辑，我们丢弃 case 的其余部分；否则，我们播放加载的音乐并将其设置为全音量（`1`为最高）：

    ```cs
    case MusicMode.musicOn :
          {
            if (GetComponentInChildren<AudioSource>().clip != null)
            {
              GetComponentInChildren<AudioSource>().Play();
              GetComponentInChildren<AudioSource>().volume = 1;
            }
            break;
    ```

为了关闭`switch`语句，我们添加一个带有几秒延迟的`yield return`，以便游戏有时间从`switch`语句中更改设置：

```cs
      }
    }
      yield return new WaitForSeconds(0.1f);
  }
```

现在我们已经创建了我们的 `enum` `musicMode` 状态，并在 `IEnumerator` 中设置了触发时每个状态将执行的操作，我们可以继续实现协程以更改音乐。

## 实现我们游戏的音状态

在本节中，我们将继续修改我们的 `ScenesManager` 脚本，并在代码的特定部分添加 `StartCoroutines`，使用 `musicMode` 状态，这是我们音乐音量将要改变的地方。例如，如果玩家在游戏中死亡，我们希望使用 `noSound` 状态立即停止音乐。

让我们从将音乐加载到游戏关卡开始，按照以下步骤操作：

1.  在 `ScenesManager` 脚本中，向下滚动到 `GameTimer` 方法。对于第一个情况，检查玩家是否在关卡 1、2 或 3 上，添加以下 `if` 语句：

    ```cs
            if (GetComponentInChildren<AudioSource>().clip ==   null)
            {
              AudioClip lvlMusic = Resources.Load<AudioClip>
                 ("Sound/lvlMusic") as AudioClip;
              GetComponentInChildren<AudioSource>().clip = lvlMusic;
              GetComponentInChildren<AudioSource>().Play();
            }
    ```

我们的 `if` 语句检查 LevelMusic 的 `AudioSource` 的音频剪辑是否为空（`null`）。如果没有音频剪辑，`if` 语句将执行以下操作：

+   从其文件夹中抓取我们的音频文件 (`lvlMusic.mp3`) 并将其存储为 `AudioClip` 数据类型。

+   将音频剪辑应用于 `AudioSource` 组件。

+   从 `AudioSource` 运行 `Play` 函数。

现在我们启动关卡时音乐就会播放，我们需要确保在关卡完成时音乐能够淡出。这部分相当简单，因为我们已经找到了在关卡完成后淡出游戏音乐的正确方法。

向下滚动到 `//if level is completed` 注释并添加以下代码行，以便在关卡完成后淡出游戏音乐：

```cs
StartCoroutine(MusicVolume(MusicMode.fadeDown));
```

在 `switch` 语句中的最后一件事是添加一行代码，将音频剪辑重置为 `null` 作为安全措施：

```cs
      default :
      {
        GetComponentInChildren<AudioSource>().clip = null;
        break;
      }
```

现在，如果调用 `GamerTimer` 方法，并且没有匹配的情况（我们的玩家不在关卡 1、2 或 3 上），那么玩家可能处于标题、游戏结束或启动场景，这意味着我们不会播放任何关卡音乐。

现在，我们将看看如何使用 `StartCoroutines`。

## 使用 `StartCoroutine` 与我们的音乐状态

现在，我们需要学习如何停止和开始音乐，通常是在关卡即将开始或突然结束（通常是在玩家死亡时）。仍然在 `ScenesManager` 中，回到需要更新的方法，以便它们可以支持音乐设置。按照以下步骤操作：

我们将要更新的第一个方法是 `ResetScene`。在方法的作用域内，输入以下代码：

```cs
 StartCoroutine(MusicVolume(MusicMode.noSound));
```

这将调用 `MusicVolume` `IEnumrator` 来关闭音乐。以下代码块显示了更新后的 `ResetScene` 方法的外观：

```cs
  public void ResetScene()
  {
    StartCoroutine(MusicVolume(MusicMode.noSound));
    gameTimer = 0;
    SceneManager.LoadScene(GameManager.currentScene);
  }
```

我们将要更新的下一个方法是 `NextLevel` 方法。我们可以随时开始音乐，无论玩家在何处。我们可以使用以下代码随时播放：

```cs
StartCoroutine(MusicVolume(MusicMode.musicOn));
```

以下代码块显示了更新后的 `NextLevel` 方法的外观：

```cs
  void NextLevel()
  {
    gameEnding = false;
    gameTimer = 0;
    SceneManager.LoadScene(GameManager.currentScene+1);
      StartCoroutine(MusicVolume(MusicMode.musicOn));
  }
```

现在，我们将继续到 `Start` 函数，它作为启动场景的安全措施，并查看它是否应该播放音乐。

当 `ScenesManager` 脚本处于活动状态时，它将自动尝试从我们的 `LevelMusic` 游戏对象的 `AudioSource` 组件播放音乐。

如果 `AudioSource` 不包含有效的 `AudioClip`（未找到 MP3），则我们的代码将假设玩家所在的关卡不需要音乐。

以下代码块展示了添加了 `StartCoroutine` 的 `Start` 函数的完整内容：

```cs
    void Start()
   {
      StartCoroutine(MusicVolume(MusicMode.musicOn));
      SceneManager.sceneLoaded += OnSceneLoaded;
   }
```

最后一个需要更新的方法是 `OnSceneLoaded`。当一个关卡被加载时，我们将尝试打开音乐。下面的代码块展示了添加了 `StartCoroutine` 的 `OnSceneLoaded` 方法：

```cs
     private void OnSceneLoaded(Scene aScene, LoadSceneMode aMode)
   {
    StartCoroutine(MusicVolume(MusicMode.musicOn));
    GetComponent<GameManager> ().SetLivesDisplay(GameManager.
        playerLives);
    if (GameObject.Find("score"))
    {
      GameObject.Find("score").GetComponent<Text>().text =
         GetComponent<ScoreManager>().PlayersScore.ToString();
    } 
   }
```

保存脚本和 `bootUp` 场景。

我们为关卡场景编写的音乐操作代码已经完成。

在本节中，我们更新了 `GameManager`，使其包含一个名为 `LevelMusic` 的第二个游戏对象。这个 `LevelMusic` 游戏对象将包含一个 `AudioSource` 组件，当玩家开始关卡、完成关卡或通过 `ScenesManager` 脚本死亡时，可以对其进行操作。

在下一节中，我们将向游戏中添加一个暂停界面，并学习如何调整音乐和音效的音量，以及更多内容。

# 创建暂停界面

目前，我们无法暂停游戏，也没有一个允许我们操作游戏设置的选项屏幕。在本节中，我们将结合这些想法，使我们的游戏能够暂停，我们还将能够更改音乐和音效的音量。

在本节中，我们将执行以下操作：

+   在屏幕的右上角添加一个暂停按钮。

+   创建一个暂停界面。

+   添加继续游戏的功能。

+   添加退出游戏的功能。

+   添加音乐和音效的滑动条。

+   创建并连接 **Audio Mixer** 到两个滑动条。

暂停界面的最终效果可以在以下屏幕截图中看到：

![图 10.3 – 暂停界面的最终视图](img/Figure_10.03_B18381.jpg)

![图 10.3 – 暂停界面的最终视图](img/Figure_10.03_B18381.jpg)

图 10.3 – 暂停界面的最终视图

让我们从关注暂停界面的视觉效果开始。然后，我们将连接滑动条和按钮。

要开始处理暂停界面的视觉效果，我们需要做以下几步：

1.  从 `Assets/Scene` 加载 `level1` 场景。

1.  当 `level1` 场景加载后，我们现在可以专注于在 **Hierarchy** 窗口中创建一些用于暂停界面的游戏对象。

1.  在 **Hierarchy** 窗口中右键单击 `Canvas` 游戏对象，从下拉列表中选择 **Create Empty**。

1.  选择新创建的游戏对象，右键单击它，选择 `PauseContainer`。

`PauseContainer` 现在需要调整到游戏屏幕的大小，以便这个游戏对象的任何子对象都可以调整到正确的缩放和位置。

1.  要使 `PauseContainer` 完全缩放到游戏屏幕的比例，确保在 **Hierarchy** 窗口中 `PauseContainer` 仍然被选中，并在 **Inspector** 窗口中将其 **Rect Transform** 属性设置为以下截图所示的属性：

![图 10.4 – PauseContainer 矩形变换属性值](img/Figure_10.04_B18381.jpg)

图 10.4 – PauseContainer 矩形变换属性值

这样我们的 `PauseContainer` 就创建好了，并设置为包含两个主要游戏对象。第一个游戏对象将包含暂停屏幕的所有单个按钮和滑块。第二个游戏对象用于屏幕左上角的暂停按钮，它将使游戏暂停并弹出暂停控制。

以下截图显示了我们的游戏，其中暂停按钮位于屏幕左上角：

![图 10.5 – 向我们的游戏添加暂停按钮](img/Figure_10.05_B18381.jpg)

图 10.5 – 向我们的游戏添加暂停按钮

但在我们开始处理游戏中的暂停按钮之前，让我们先专注于暂停屏幕及其内容。为了创建一个包含游戏对象的 `PauseScreen` 游戏对象，我们需要对 `PauseContainer` 的 **Rect Transform** 属性重复类似的步骤。

要在 `PauseContainer` 中创建和包含一个 `PauseScreen` 游戏对象，请按照以下步骤操作：

1.  在 **Hierarchy** 窗口中右键点击 `PauseContainer` 游戏对象。

1.  从下拉菜单中选择 **Create Empty**。

1.  新创建的游戏对象将是 `PauseContainer` 游戏对象的子对象。现在，让我们将新创建的游戏对象重命名为 `PauseScreen`。

1.  右键点击 `PauseScreen`。

1.  在 `PauseContainer` 中仍然选中 `PauseScreen`。使用之前的 **Rect Transform** 图像作为参考。

现在，我们可以开始填充我们的 `PauseScreen` 游戏对象，添加其自身的游戏对象。

让我们开始降低屏幕亮度，这样当游戏暂停时玩家就不会分心。

要创建一个暗淡效果，请按照以下步骤操作：

1.  在 `PauseScreen` 游戏对象中选中 `blackOutScreen`。

1.  应用与最后两个游戏对象相同的 **Rect Transform** 属性。

1.  现在，我们需要添加 **Image** 组件，以便我们可以用半透明的黑色覆盖屏幕。

1.  在 `blackOutScreen` 仍然被选中时，点击 `Image`。一旦你看到 `blackOutScreen`。

1.  对于 `blackOutScreen` 组件的图像属性，最后一步是将其 **Color** 设置为以下截图所示的值：

![图 10.6 – 将 blackOutScreen 图像组件颜色值（RGBA）设置为截图中的值](img/Figure_10.06_B18381.jpg)

图 10.6 – 将 blackOutScreen 图像组件颜色值（RGBA）设置为截图中的值

现在，屏幕上将会出现一片半暗的屏幕。

现在，让我们添加 **Pause** 文本。为此，请按照以下步骤操作：

1.  在 `PauseScreen` 游戏对象中选中 `PauseText`。

1.  这次，给 `PauseText` 的 **Rect Transform** 属性设置以下值：

![图 10.7 – 将 PauseText Rect Transform 属性值设置为截图中所显示的值](img/Figure_10.07_B18381.jpg)

图 10.7 – 将 PauseText Rect Transform 属性值设置为截图中所显示的值

接下来，我们需要添加`PauseText`游戏对象。

1.  仍然选择`PauseText`，点击`Text`直到你能在下拉列表中看到它。一旦做到这一点，就选择它。

1.  将**文本组件**的设置更改为以下截图中所显示的设置：

![图 10.8 – 将所有 PauseText 文本组件属性值更新为截图中所显示的值](img/Figure_10.08_B18381.jpg)

图 10.8 – 将所有 PauseText 文本组件属性值更新为截图中所显示的值

如果你需要有关**文本组件**的更多信息，请查看*第八章*中的*将文本和图像应用于你的场景*部分，*添加自定义字体和 UI*。

以下截图显示了当前**层次结构**和**场景**视图的样式：

![图 10.9 – 暂停文本应看起来像这样](img/Figure_10.09_B18381.jpg)

图 10.9 – 暂停文本应看起来像这样

我们已经自定义并居中了暂停标题。现在，让我们继续为**音乐**和**效果**音量设置添加一些滑块。我们将从**音乐**滑块开始，然后将其复制到屏幕的另一侧用于**效果**滑块。

## 将音量 UI 滑块添加到暂停屏幕

在本节中，我们将为暂停屏幕提供标题名称，并为游戏的音乐及其音效创建和自定义暂停屏幕音量滑块。

要创建、自定义和定位**音乐**滑块，请按照以下步骤操作：

1.  在**层次结构**窗口中右键单击`PauseScreen`游戏对象。然后，从下拉菜单中选择**UI**，接着选择**滑块**，如图所示：

![图 10.10 – 从 UI 下拉菜单添加一个滑块](img/Figure_10.10_B18381.jpg)

图 10.10 – 从 UI 下拉菜单添加一个滑块

1.  选择新创建的`Slider`游戏对象，右键单击它，并将其重命名为`Music`。

1.  接下来，通过更改其**Rect Transform**属性到以下截图中所显示的值来定位“音乐”滑块：

![图 10.11 – 将音乐 Rect Transform 属性值设置为这里所示](img/Figure_10.11_B18381.jpg)

图 10.11 – 将音乐 Rect Transform 属性值设置为这里所示

我们现在将改变“音乐”滑块的条形颜色，使其更适合暂停屏幕。我们将将其从浅灰色改为红色。

要更改滑块的颜色，请执行以下操作：

1.  点击`Fill Area`游戏对象中`Music`游戏对象左侧的箭头。

1.  从“音乐”游戏对象的下拉菜单中选择“填充”游戏对象，如图所示，就像它在**层次结构**窗口中看起来一样：

![图 10.12 – 在层次结构窗口的填充区域下拉菜单中选择填充

![Figure_10.12_B18381.jpg]

Figure 10.12 – 在层次结构窗口的填充区域下拉列表中选择填充

1.  在仍然选择`Fill`的情况下，在其**检查器**窗口中，将**图像**组件的**颜色**值更改为红色，如图所示以下截图：

![Figure 10.13 – 将填充图像组件的颜色值更改为本截图中的值

![Figure 10.13_B18381.jpg]

Figure 10.13 – 将填充图像组件的颜色值更改为本截图中的值

如果你仍然选择了`Fill`游戏对象，你可以通过调整**检查器**窗口中**滑块**组件底部的**值**滑块来查看滑块的红色背景，如图所示以下截图：

![Figure 10.14 – 将填充滑块组件的值更新到本截图中的值

![Figure 10.14_B18381.jpg]

Figure 10.14 – 将填充滑块组件的值更新到本截图中的值

此外，如前一个截图所示，我们需要设置滑块的`-80`和它的`0`。这样做的原因是，在下一章中，这些值将与音频混音器的相同值匹配。

音乐滑块的大小已经设置好了；我们只需要调整把手，使其不那么拉伸，更容易用手指点击或拖动。按照以下步骤进行操作：

1.  在`Handle`游戏对象中。然后，选择它。

1.  在`Image`组件中停止`Handle`游戏对象看起来那么拉伸。

1.  在仍然选择`Handle`的情况下，将所有轴上的`3`进行更改。

以下截图显示了我们的把手现在看起来是什么样子：

![Figure 10.15 – 滑块的把手现在已视觉更新

![Figure 10.15_B18381.jpg]

Figure 10.15 – 滑块的把手现在已视觉更新

音乐滑块现在已设置。这意味着我们可以继续到文本部分，以便我们可以为玩家标注滑块。为了给滑块添加自己的 UI 文本，我们需要执行以下操作：

1.  在`PauseScreen`游戏对象和下拉列表中，选择**UI**，然后选择**文本**。

1.  右键单击我们新创建的`Text`游戏对象并选择`MusicText`。

1.  在仍然选择`MusicText`游戏对象的情况下，将其**矩形变换**更改为以下值以定位和缩放文本到正确的位置：

![Figure 10.16 – 更改 MusicText 矩形变换的值

![Figure 10.16_B18381.jpg]

Figure 10.16 – 更改 MusicText 矩形变换的值

1.  在**检查器**窗口中仍然选择`MusicText`游戏对象，更新**文本**组件的值到以下属性值：

![Figure 10.17 – 更新 MusicText 文本组件属性值到本截图中的值

![Figure 10.17_B18381.jpg]

Figure 10.17 – 更新 MusicText 文本组件属性值到本截图中的值

我们的暂停屏幕开始成形。以下截图显示了我们目前拥有的内容：

![Figure 10.18 – 我们现在有了音乐音量滑块

![Figure 10.18_B18381.jpg]

图 10.18 – 现在我们有一个音乐音量滑块

现在，我们可以将我们的音乐文本和滑块复制并粘贴到屏幕的另一侧，并调整一些属性值，以便它被识别为音效音量条。

要复制并调整音乐文本和滑块，请按照以下步骤操作：

1.  按 *Ctrl* (*Command* 在 Mac 上) 并从 **Hierarchy** 窗口中选择 `MusicText` 和 `Music` 以便它们被突出显示。然后，按键盘上的 *D* 键来复制这两个游戏对象。

1.  选择 `Music (1)` 游戏对象，右键点击它，选择 `Effects`。

1.  选择 `MusicText (1)` 游戏对象，右键点击它，选择 `EffectsText`。

1.  在 `EffectsText` 仍然被选中时，在 **Inspector** 窗口中更新其 **Rect Transform** 属性，使用以下属性值：

![Figure 10.19 – 更新 EffectsText Rect Transform 属性值到本截图所示]

![img/Figure_10.19_B18381.jpg]

图 10.19 – 更新 EffectsText Rect Transform 属性值到本截图所示

1.  在 `EffectsText` 仍然被选中时，我们现在可以注意重命名文本。其余的 EffectsText 的 `MUSIC` 更改为 `EFFECTS`，如以下截图所示：

![Figure 10.20 – 将 EffectsText Text Component 文本从 MUSIC 更改为 EFFECTS]

![img/Figure_10.20_B18381.jpg]

图 10.20 – 将 EffectsText Text Component 文本从 MUSIC 更改为 EFFECTS

接下来，我们可以将我们的 **Effects** 滑块移动到场景视图中 **EFFECTS** 文本下方。为此，请按照以下步骤操作：

1.  在 **Hierarchy** 窗口中选择 `Effects` 游戏对象。在 **Inspector** 窗口中，将它的 **Rect Transform** 属性更改为以下截图所示：

![Figure 10.21 – 更新 Effects Rect Transform 组件属性值]

![img/Figure_10.21_B18381.jpg]

图 10.21 – 更新 Effects Rect Transform 组件属性值

在视觉元素方面，我们的暂停屏幕几乎完成了。最后两件事是我们需要配置的 **Quit** 和 **Resume** 按钮。与滑块游戏对象一样，我们可以创建一个，复制并粘贴以创建第二个，然后编辑它们。

要创建和自定义一个 **Quit** 按钮，请按照以下步骤操作：

1.  在 **Hierarchy** 窗口中右键点击 `PauseScreen` 游戏对象，然后从下拉列表中选择 **UI**，然后选择 **Button**。

1.  使用新创建的 `Button` 游戏对象，我们可以将其重命名为 `Quit`；在 `Button` 中右键点击 `Button` 游戏对象并将其重命名为 `Quit`。

1.  现在，我们可以将 `Quit` 游戏对象放置到正确的位置，并在 `PauseScreen` 游戏对象内调整其大小。

1.  在 `Quit` 游戏对象仍然被选中时，在 **Inspector** 窗口中将其 **Rect Transform** 属性更改为以下截图所示：

![Figure 10.22 – 更新 Quit Rect Transform 属性值]

![img/Figure_10.22_B18381.jpg]

图 10.22 – 更新 Quit Rect Transform 属性值

1.  现在，`Quit` 游戏对象将位于暂停屏幕的右下角：

![图 10.23 – 我们的游戏退出按钮已添加到暂停屏幕](img/Figure_10.23_B18381.jpg)

图 10.23 – 我们的游戏退出按钮已添加到暂停屏幕

接下来，我们可以通过更改按钮的精灵、颜色和文本来自定义它。我们将从移除之前截图中所见的圆角按钮精灵开始。

在仍然选择我们的 `Quit` 按钮的情况下，我们可以通过以下步骤移除单个精灵：

1.  在 **检查器** 窗口中，点击右上角的远程按钮（以下截图中的 **1** 所示）。

1.  将出现一个新窗口。从下拉菜单中选择 **无**（以下截图中的 **2** 所示）：

![图 10.24 – 移除退出按钮的源图像精灵](img/Figure_10.24_B18381.jpg)

图 10.24 – 移除退出按钮的源图像精灵

接下来，我们将更改按钮的颜色，如下所示：

1.  在 **层次结构** 窗口中仍然选择 `Quit` 游戏对象，我们可以在 **检查器** 窗口中更改 **按钮组件** 的 **正常颜色** 属性。

1.  选择标题为 `255`、`0`、`0`、`150` 的颜色字段。

接下来我们需要对按钮做的下一件事是更改其文本。

1.  在 **层次结构** 窗口中仍然选择我们的 `Quit` 游戏对象，选择其名称左侧的向下箭头。

1.  从 `Quit` 游戏对象中选择 `Text` 子游戏对象，并在 **检查器** 窗口中的 **文本** 组件中设置以下属性设置：

![图 10.25 – 将 Text 游戏对象的 Text 组件更新为截图所示的值](img/Figure_10.25_B18381.jpg)

图 10.25 – 将 Text 游戏对象的 Text 组件更新为截图所示的值

我们将得到一个看起来更适合我们游戏的按钮：

![图 10.26 – 我们的游戏退出按钮现在看起来更适合我们的游戏](img/Figure_10.26_B18381.jpg)

图 10.26 – 我们的游戏退出按钮现在看起来更适合我们的游戏

在本节中最后要做的就是复制我们刚刚创建的 `Quit` 游戏对象，并将文本重命名为 `RESUME`。**Resume** 按钮将用于取消暂停屏幕，让玩家继续玩游戏。

要创建 `Resume` 游戏对象，我们需要执行以下操作：

1.  在 **层次结构** 窗口中选择 `Quit` 游戏对象。

1.  按下键盘上的 *Ctrl* 键（在 Mac 上为 *command* 键）和 *D* 键来复制游戏对象。

1.  将复制的游戏对象从 `Quit (1)` 重命名为 `Resume`。

1.  在仍然选择 `Resume` 的情况下，在 **检查器** 窗口中更改其 **矩形变换** 属性值，如以下截图所示：

![图 10.27 – 给 Resume 矩形变换赋予与截图相同的属性值](img/Figure_10.27_B18381.jpg)

图 10.27 – 给 Resume 矩形变换赋予与截图相同的属性值

对于 `Resume` 游戏对象，剩下的只是将其文本从 `QUIT` 更改为 `RESUME`。通过在 **Hierarchy** 窗口中点击左侧的箭头来扩展 **Resume** 选择，遵循以下步骤：

1.  在 **Hierarchy** 窗口中选择 `Text` 游戏对象。

1.  在 `QUIT` 到 `RESUME`，如图所示：

![图 10.28 – 暂停按钮![图 10.28 – 图 10.28_B18381.jpg](img/Figure_10.28_B18381.jpg)

图 10.28 – 暂停按钮

暂停屏幕现在在视觉上已经完成，并且由于使用了我们的 **Anchors**（来自 **Rect Transform** 属性），可以支持各种屏幕比例。之前我们提到，我们将在游戏屏幕的左上角有一个暂停按钮，这样我们就可以暂停游戏并加载我们刚刚制作的暂停屏幕。

本节中我们所做的一切都是在 Unity 编辑器中完成的，没有使用任何代码。在本节中，我们涵盖了以下主题：

+   如何访问我们的暂停屏幕

+   暂停屏幕将如何覆盖我们游戏中的关卡

+   在暂停屏幕的背景上应用半透明的黑屏以降低游戏亮度

+   为我们的音乐和效果创建滑块

+   将自定义文本应用到各个点

+   使用 Unity 的 **Button** 组件为玩家提供退出或继续游戏的选择

现在，让我们制作暂停按钮。之后，我们可以开始查看如何将这些滑块和按钮与我们的代码连接起来。

# 添加游戏暂停按钮

在上一节的开头，我们简要地提到了 **游戏内暂停按钮**。这个按钮将在关卡开始时出现，一旦按下，玩家、敌人和已经发射的子弹将暂停时间。在本节中，我们只关注视觉效果，就像我们在上一节中处理暂停屏幕一样。

暂停按钮的行为将略不同于我们之前制作的按钮。这次，按钮将是一个 `toggle` 游戏对象，执行以下操作：

1.  在 **Hierarchy** 窗口中选择 `PauseContainer` 游戏对象，右键单击它，从下拉列表中选择 **UI**，然后选择 **Toggle**，如图所示：

![图 10.29 – 添加一个 Toggle（开关）![图 10.29_B18381.jpg](img/Figure_10.29_B18381.jpg)

图 10.29 – 添加一个 Toggle（开关）

1.  在 `PauseButton` 中仍然选择 `Toggle` 游戏对象。

1.  目前，我们的 `PauseButton` 看起来与我们想要的样子完全不同，就像下面的截图所示，它更像是一个复选框。然而，我们可以修复这个问题，让它看起来像一个正常的**暂停**按钮，但具有切换（开启或关闭）的功能：

![图 10.30 – 我们将把复选框改为暂停按钮![图 10.30_B18381.jpg](img/Figure_10.30_B18381.jpg)

图 10.30 – 我们将把复选框改为暂停按钮

要改变 `PauseButton` 游戏对象当前的外观，使其看起来像前面截图中的预期样子，我们需要执行以下操作：

1.  在**暂停按钮**游戏对象中展开其内容，如下截图所示：

![图 10.31 – 在层次结构窗口中展开所有暂停按钮子对象](img/Figure_10.31_B18381.jpg)

图 10.31 – 在层次结构窗口中展开所有暂停按钮子对象

1.  在**层次结构**窗口中选择`Label`游戏对象，并在键盘上按*Delete*键。

将**切换**标签删除。

1.  接下来，我们将设置我们的游戏对象到正确的位置和缩放。在**层次结构**窗口中选择`PauseButton`，并在**检查器**窗口中为其**矩形变换**设置以下属性：

![图 10.32 – 更新暂停按钮矩形变换属性值](img/Figure_10.32_B18381.jpg)

图 10.32 – 更新暂停按钮矩形变换属性值

现在将切换放置并缩放到游戏画布的左上角，如下截图所示：

![图 10.33 – 暂停按钮的锚点设置为屏幕的左上角](img/Figure_10.33_B18381.jpg)

图 10.33 – 暂停按钮的锚点设置为屏幕的左上角

1.  注意我们的**暂停按钮**游戏对象包含另一个名为`Background`的游戏对象，但在**层次结构**窗口中没有选择其`Background`游戏对象：

![图 10.34 – 从层次结构窗口中选择背景游戏对象](img/Figure_10.34_B18381.jpg)

图 10.34 – 从层次结构窗口中选择背景游戏对象

1.  要纠正**背景**游戏对象的**背景**游戏对象，并在**检查器**窗口中设置以下值：

![图 10.35 – 将背景矩形变换属性值设置为截图中的值](img/Figure_10.35_B18381.jpg)

图 10.35 – 将背景矩形变换属性值设置为截图中的值

在**锚点**大小方面，**背景**游戏对象现在与**暂停按钮**游戏对象的大小相同。

我们现在可以开始调整大小并使用合适的图像填充**背景**。我们将用深色圆圈替换带有勾选标记的白色正方形图标。按照以下步骤进行操作：

1.  如果尚未这样做，在**背景**游戏对象中展开`PauseButton`。

1.  在**检查器**窗口中的**图像**组件的右侧选择**源图像**的小遥控按钮。

1.  从出现的下拉菜单中，用`UISprite`替换其当前选择，并将其更改为**旋钮**。其选择在以下截图中显示。

现在正方形已经变成了圆形。现在，我们可以改变它的颜色，使其与游戏 UI 的其余部分相匹配。

1.  在选择**背景**游戏对象的情况下，选择其`92`、`92`、`92`和`123`，如下截图所示：

![图 10.36 – 我们的游戏背景图像组件应具有与截图中的相同值](img/Figure_10.36_B18381.jpg)

图 10.36 – 我们背景图像组件应具有与本截图所示相同的值

接下来，我们可以将灰色椭圆形形状变成圆形。

仍然在**图像**组件中，将**图像类型**设置为**简单**，并勾选**保持纵横比**框，如图中所示的前一个屏幕截图。

**图像类型**为图像提供不同的行为；例如，**分割**在进度条/计时器中表现良好，可以随时间增加可见图像的部分。

**保持纵横比**意味着无论图像如何缩放，它都将保持其原始形式 – 将不会有挤压或拉伸的外观。

这是**场景**视图中`PauseButton`的特写视图：

![图 10.37 – 我们暂停按钮当前的外观](img/Figure_10.37_B18381.jpg)

图 10.37 – 我们暂停按钮当前的外观

现在，我们需要将勾选图像替换为一个大型的暂停符号。按照以下步骤进行操作：

1.  从`Background`游戏对象中选择`Checkmark`游戏对象，并在**检查器**窗口中，为其**矩形变换**设置以下设置：

![图 10.38 – 设置勾选矩形变换属性值到本截图所示](img/Figure_10.38_B18381.jpg)

图 10.38 – 设置勾选矩形变换属性值到本截图所示

1.  在`Checkmark`中仍然选择`Checkmark`游戏对象，通过点击**远程**按钮并从下拉列表中选择**暂停**精灵来将其设置为`pause`。

1.  选择`152`、`177`、`178`、`125`。

1.  如果尚未设置，将**图像类型**更改为**简单**。

1.  勾选**保持纵横比**框，如图中所示的下个屏幕截图：

![图 10.39 – 更新勾选图像组件属性值到本截图所示](img/Figure_10.39_B18381.jpg)

图 10.39 – 更新勾选图像组件属性值到本截图所示

**场景**窗口应该看起来像这样，我们的暂停按钮位于左上角：

![图 10.40 – 我们暂停按钮现在视觉上完整](img/Figure_10.40_B18381.jpg)

图 10.40 – 我们暂停按钮现在视觉上完整

最后，为了确保当我们在它上点击时，`toggle`按钮实际上会执行某些操作，我们需要确保在我们的**层级**窗口中有一个**EventSystem**。这样做非常简单；按照以下步骤进行：

1.  在**层级**窗口中，右键单击空白区域。

1.  如果**层级**中没有**EventSystem**，请选择**UI**，然后选择**EventSystem**。

1.  保存场景。

在本节中，我们在一个支持各种横版变体的屏幕上混合了我们的 UI 图像、按钮、文本和滑块。

在下一节中，我们将继续讨论当玩家按下按钮或移动滑块时，我们在暂停屏幕中制作的每个 UI 组件将执行的操作。

## 创建我们的 PauseComponent 脚本

`PauseComponent`脚本将负责管理与访问和修改暂停屏幕提供给玩家的条件相关的所有事情。在这里，我们将遵循一系列小节，这些小节将带我们设置`PauseComponent`脚本的各个部分。在我们这样做之前，我们需要创建我们的脚本。如果你不知道如何创建脚本，那么请回顾*第二章*中的*设置我们的相机*部分，*添加和操作对象*。一旦你完成了这个，将脚本重命名为`PauseComponent`。出于维护目的，在**项目**窗口中将你的脚本存储在`Assets/Script`文件夹中。

现在，让我们继续到`PauseComponent`脚本的第一个小节，通过应用游戏中的暂停按钮的逻辑。

## 暂停屏幕基本设置和暂停按钮功能

在本节中，我们将使暂停按钮在玩家在关卡中控制游戏时出现。当玩家按下暂停按钮时，我们需要确保所有移动组件和滚动纹理都冻结。最后，我们需要引入暂停屏幕本身。

如果我们以当前状态开始关卡，我们会看到`PauseScreen`游戏对象覆盖了屏幕。这看起来很棒，但我们需要暂时关闭它。要关闭`PauseScreen`游戏对象，请执行以下操作：

1.  在 Unity 编辑器中，通过双击`Assets/Script`中持有的文件来打开新创建的`PauseComponent`脚本。

1.  在脚本打开后，在顶部添加`UnityEngine` UI 库，以便为我们的代码提供额外的功能（操作文本和图像组件），包括常用的`UnityEngine`库和类的名称，以及其继承自`MonoBehaviour`：

    ```cs
    using UnityEngine.UI;
    using UnityEngine;
    public class PauseComponent : MonoBehaviour 
    {
    ```

1.  将以下变量添加到`PauseComponent`类中：

    ```cs
     [SerializeField]
              GameObject pauseScreen;
    }
    ```

`[SerializeField]`将使`pauseScreen`变量在`public`中可见。第二行是一个`GameObject`类型，它将存储对整个`PauseScreen`游戏对象的引用。

1.  保存脚本。

1.  回到 Unity 编辑器，从`PauseComponent`中选择`PauseContainer`游戏对象，直到你在下拉列表中看到它。

1.  现在，将`PauseScreen`游戏对象从`PauseScreen`拖放到相应的位置，如下面的截图所示：

![Figure 10.41 – Drag and drop the PauseScreen game object into the Pause Component | Pause Screen field](img/Figure_10.41_B18381.jpg)

![Figure 10.41 – Drag and drop the PauseScreen game object into the Pause Component | Pause Screen field](img/Figure_10.41_B18381.jpg)

图 10.41 – 将 PauseScreen 游戏对象拖放到暂停组件 | 暂停屏幕字段

回到`PauseComponent`脚本，我们现在可以在关卡开始时关闭`PauseScreen`游戏对象，并在玩家按下暂停按钮时重新打开它。要关闭`PauseScreen`，我们可以执行以下操作：

1.  在`PauseComponent`脚本中，创建一个`Awake`函数，并在其中关闭`pauseScreen`游戏对象，如下面的代码所示：

    ```cs
      void Awake()
      {
        pauseScreen.SetActive(false);
      }
    ```

1.  保存脚本。

现在，当我们按下屏幕顶部的 **播放** 按钮时，我们可以在编辑器中测试它。游戏将运行，而不会显示暂停屏幕。现在，我们可以在关卡开始后的几秒钟内向玩家介绍暂停按钮。

让我们先创建一个方法，该方法将关闭/打开玩家暂停按钮的视觉效果和交互性：

1.  返回到 `PauseComponent` 脚本，并创建一个接受一个 `bool` 参数的方法，如下面的代码所示：

    ```cs
      void SetPauseButtonActive(bool switchButton)
      {
    ```

由于我们的 `PauseComponent` 脚本附加到 `PauseContainer` 游戏对象上，我们可以轻松访问任何游戏对象及其组件。附加的其他两个主要游戏对象是 `PauseScreen` 和 `PauseButton`。接下来我们将添加到 `SetPauseButtonActive` 的几行代码将与 `PauseButton` 游戏对象的视觉效果和交互性相关。

1.  要更改我们 `PauseButton` 的可见性，我们需要访问其 `Toggle` 组件的 `colors` 值并将其存储在一个临时的 `ColorBlock` 类型中。在 `SetPauseButtonActive` 方法中输入以下代码行：

    ```cs
    ColorBlock col = GetComponentInChildren<Toggle>().colors;
    ```

1.  接下来，我们需要通过查看方法接收到的 `bool` 参数来检查值的条件。如果 `switchButton` 的 `bool` 设置为关闭，那么我们将所有与切换相关的颜色设置为全零，即黑色和零 alpha（完全透明）。

在我们之前输入的代码行之后输入以下代码：

```cs
    if (switchButton == false)
    {
      col.normalColor = new Color32(0,0,0,0);
      col.highlightedColor = new Color32(0,0,0,0);
      col.pressedColor = new Color32(0,0,0,0);
      col.disabledColor = new Color32(0,0,0,0); 
      GetComponentInChildren<Toggle>().interactable =     false;
    }
```

前面的代码显示我们运行了一个检查，以查看 `bool` 参数是否为 `false`。

1.  如果 `switchButton` 包含一个 `false` 值，那么我们将进入 `if` 语句并将 `col`（暂停按钮的颜色）的 `normalColor` 属性设置为全零。这意味着它根本不会显示这个按钮。然后，我们将相同的值应用到暂停按钮的所有其他可能的颜色状态上。我们还需要将 `Toggle` 的 `interactable` 值设置为 `false`，这样玩家就不会意外地按下暂停按钮。

下一个图中的左侧截图显示了我们刚刚输入的代码。右侧的截图是经过我们 `if` 语句更改属性的 `Toggle` 组件：

![图 10.42 – 左侧的代码将操作右侧的 Toggle 属性值](img/Figure_10.42_B18381.jpg)

图 10.42 – 左侧的代码将操作右侧的 Toggle 属性值

如果 `switchButton` 设置为 `true`，我们将所有零的值设置为所选的颜色值，并使 `PauseButton` 不可交互。

1.  在我们刚刚写的代码之后输入以下代码：

    ```cs
        else
        {
          col.normalColor = new Color32(245,245,245,255);
          col.highlightedColor = new Color32(245,245,245,255);
          col.pressedColor = new Color32(200,200,200,255);
          col.disabledColor = new Color32(200,200,200,128); 
          GetComponentInChildren<Toggle>().interactable = true;
        }
    ```

在此代码段之后的最后两行将 `col` 值重新应用到 `Toggle` 组件上。

第二行代码打开或关闭暂停符号。如果没有设置，那么暂停按钮将出现/消失，而不会影响两个白色的暂停条纹。

1.  在前面的代码之后，添加了最后两个 `GetComponentInChildren` 行，这会将颜色重新应用到 `Toggle` 组件，并使用 `switchButton` 变量将暂停符号设置为开或关：

    ```cs
    GetComponentInChildren<Toggle>().colors = col;
    GetComponentInChildren<Toggle>()
      .transform.GetChild(0).GetChild(0).gameObject.SetActive
         (switchButton);
    }
    ```

1.  现在，我们只需要使用我们刚刚编写的函数。最初，我们希望暂停按钮在关卡开始时不可见，直到玩家控制他们的飞船。要关闭暂停按钮，我们需要重新访问 `Awake` 函数并执行以下操作：

    ```cs
      void Awake()
      {
        pauseScreen.SetActive(false);
        SetPauseButtonActive(false);
        Invoke("DelayPauseAppear",5);
      }
    ```

在这里，我在 `Awake` 函数中添加了两行额外的代码。`SetPauseButtonActive(false)` 使用我们刚刚创建的方法关闭暂停按钮，而 `Invoke` 函数将延迟 5 秒，直到我们运行 `DelayPauseAppear` 方法。在 `DelayPauseAppear` 中是 `SetPauseButtonActive(true)`，这是我们玩家获得对飞船控制的时间。

1.  在 `Invoke` 函数中添加我们提到的额外方法来打开暂停按钮，如下所示：

    ```cs
      void DelayPauseAppear()
      {
        SetPauseButtonActive(true);
      }
    ```

1.  保存脚本。

在 Unity 编辑器中，按**播放**；我们的游戏将正常开始，5 秒后，暂停按钮将出现在左上角。如果我们按下暂停按钮，它将中断，不会发生任何额外的事情。这是因为我们没有在按下暂停按钮时让它执行任何操作。

让我们回到 `PauseComponent` 脚本，并添加一个可以在按下暂停按钮时运行的小方法。要添加一个暂停方法，该方法冻结游戏并显示我们之前构建的暂停屏幕，请按照以下步骤操作：

1.  重新打开 `PauseComponent` 脚本并输入以下方法：

    ```cs
      public void PauseGame()
      {
        pauseScreen.SetActive(true);
        SetPauseButtonActive(false);
        Time.timeScale = 0;
      }
    ```

1.  在 `PauseGame` 方法中，我们设置以下内容：

    +   我们将暂停屏幕游戏对象的活跃度设置为 `true`。

    +   关闭暂停按钮（因为我们有 `QUIT` 按钮可以使用）。

1.  将游戏的 `timeScale` 设置为 0，这将停止场景中所有移动和动画对象。有关 `timeScale` 的更多信息，请查看官方 Unity 文档，[`docs.unity3d.com/ScriptReference/Time-timeScale.html`](https://docs.unity3d.com/ScriptReference/Time-timeScale.html)。

**timeScale** 也可以在 Unity 编辑器的**时间管理器**中找到。它位于**编辑器**窗口的顶部，在**编辑 | 项目设置 | 时间**下。

你还有其他有用的属性，例如**固定时间步长**，你可以更改其值以使你的物理模拟更精确。有关**时间管理器**及其属性的更多信息，请查看以下链接：[`docs.unity3d.com/Manual/class-TimeManager.html`](https://docs.unity3d.com/Manual/class-TimeManager.html)。

1.  保存脚本并返回到编辑器。

现在，我们需要将新的 `PauseGame` 方法附加到 `PauseButton` 事件系统，如下所示：

1.  从**层次结构**窗口中选择 `PauseButton` 游戏对象。

1.  在**检查器**窗口的底部，点击加号（**+**）来添加一个事件：

![图 10.43 – 向切换组件添加事件](img/Figure_10.43_B18381.jpg)

图 10.43 – 向切换组件添加事件

1.  接下来，将包含我们的`PauseComponent`脚本的`PauseContainer`拖到空字段（从下拉菜单中标记为`PauseComponent`）。

1.  最后，选择`PauseGame ()`公共方法（在以下屏幕截图中标记为**3**）。

以下屏幕截图显示了我们在选择`PauseGame ()`方法中经历的标记步骤：

![图 10.44 – 从层次结构窗口拖动 PauseContainer 到事件槽；最后将其功能设置为 PauseGame ()](img/Figure_10.44_B18381.jpg)

![图 10.44 – B18381.jpg](img/Figure_10.44_B18381.jpg)

![图 10.44 – 从层次结构窗口拖动 PauseContainer 到事件槽；最后将其功能设置为 PauseGame ()](img/Figure_10.44_B18381.jpg)

现在是尝试查看当我们按下暂停按钮时是否会出现暂停屏幕的好时机。在 Unity 编辑器中按下**播放**，在**游戏**窗口中，当它出现时，在左上角按下暂停按钮。暂停屏幕将出现；除非我们为我们的**继续**和**退出**按钮编写逻辑，否则我们将无法从中逃脱。

到目前为止，在本节中，我们已经赋予了玩家暂停游戏的能力。在下一节中，我们将使玩家能够在暂停屏幕中继续或退出游戏。

## 从暂停屏幕恢复或退出游戏

在本小节中，我们将通过添加两个方法继续扩展`PauseComponent`脚本：

+   `继续`

+   `退出`

让我们从添加**继续**按钮的逻辑开始；按照以下说明操作：

1.  如果`PauseComponent`脚本尚未打开，请转到`Assets/Script`。双击文件以打开它。

1.  在`PauseComponent`脚本内部，滚动到可以添加新方法的位置 – 不重要，只要它在`PauseComponent`类内部，并且不干扰其他方法即可。

1.  现在，我们将添加一个`Resume`方法，如果玩家希望关闭暂停屏幕，游戏动画将继续，左上角的暂停按钮将重新出现。要使所有这些发生，请添加以下代码：

    ```cs
      public void Resume()
      {
        pauseScreen.SetActive(false);
        SetPauseButtonActive(true);
        Time.timeScale = 1;
      }
    ```

此代码与上一节中显示的代码类似；只是顺序相反（值设置为 true，现在是 false，反之亦然，以恢复原始设置）。

1.  保存脚本并返回到 Unity 编辑器。

1.  在 Unity 编辑器中，选择`PauseScreen`游戏对象。确保**层次结构**窗口也显示已选择**继续**。

1.  在**层次结构**窗口中，将`PauseContainer`游戏对象拖到**无（对象）**的**点击()**事件系统中。

1.  选择`PauseComponent`，然后为`Resume`游戏对象按钮设置正确的事件系统`On Click ()`：

![图 10.45 – 继续按钮的 On Click () 事件连接到 Resume 函数](img/Figure_10.45_B18381.jpg)

![图 10.45 – B18381.jpg](img/Figure_10.45_B18381.jpg)

![图 10.45 – 继续按钮的 On Click () 事件连接到 Resume 函数](img/Figure_10.45_B18381.jpg)

1.  在继续到**退出**按钮之前，让我们先测试**继续**按钮。在编辑器中按**播放**。一旦暂停按钮出现在**游戏**窗口的左上角，点击它。

1.  最后，点击大号**继续**按钮。我们将返回到正在进行的游戏。

1.  我们暂停屏幕中要连接的最后一个按钮是`PauseComponent`脚本，并将以下方法添加到脚本中：

    ```cs
      public void Quit()
      {
        Time.timeScale = 1;
        GameManager.Instance.GetComponent<ScoreManager>().    ResetScore();
        GameManager.Instance.GetComponent<ScenesManager>().    BeginGame(0);
      }
    ```

我们刚刚输入的代码重置了游戏；`timescale`值回到`1`。我们直接从`ScoreManager`重置玩家的分数，并直接告诉`ScenesManager`带我们回到场景零，即我们的`bootUp`场景。

1.  在结束本节之前保存脚本。

这与**继续**按钮在设置事件到我们的脚本方面相似。

1.  从暂停屏幕中选择**退出**按钮，并确保在**检查器**窗口的底部，你遵循与**继续**按钮相同的步骤。

1.  当我们开始应用**退出**方法时。

以下截图显示了`Quit`游戏对象的按钮设置：

![图 10.46 – 当按下退出按钮时，它将运行退出函数![图片](img/Figure_10.46_B18381.jpg)

图 10.46 – 当按下退出按钮时，它将运行退出函数

在我们结束本章之前，我们需要确保当游戏暂停时，玩家和敌人的行为符合我们的预期。

## 暂停玩家和敌人

因此，我们已经到达了可以按下游戏中的暂停按钮并观察游戏暂停的时刻的点。为了确保场景已保存，包括新的和编辑的脚本，让我们测试暂停屏幕：

1.  在 Unity 编辑器中按**播放**。

1.  当暂停按钮出现时，按下它。

1.  游戏暂停了，但我们的敌人看起来像是漂浮离开了。此外，当我们按下玩家的射击按钮时，其玩家子弹光在船上闪烁。

让我们先修复敌人的漂浮问题。

1.  这是一个简单的修复——我们需要更改`EnemyWave`脚本的更新时间。

1.  停止播放。然后，在`Assets/Script`中双击`EnemyWave`脚本。

1.  找到以下内容的行：

    ```cs
    void Update()
    ```

更改为以下内容：

```cs
void FixedUpdate()
```

1.  保存`EnemyWave`脚本。

    更多信息

    更多关于**FixedUpdate**的信息可以在这里找到：[`docs.unity3d.com/ScriptReference/MonoBehaviour.FixedUpdate.html`](https://docs.unity3d.com/ScriptReference/MonoBehaviour.FixedUpdate.html)。

现在，让我们加强玩家的行为，以便在游戏暂停时冻结其所有功能。为了冻结我们的玩家，我们需要重新打开其脚本：

1.  在`Assets/Script`文件夹中。

1.  双击`Player`脚本。

1.  滚动到`Update`函数，并用以下`if`语句包裹玩家的`Movement`和`Attack`方法：

    ```cs
       void Update ()
      {
        if(Time.timeScale == 1)
        {
          Movement();
          Attack();
        }
      }
    ```

上述代码会检查游戏的时间缩放是否以全速（`1`）运行，然后继续执行`Movement`和`Attack`方法。

1.  保存`Player`脚本。

太好了！我们现在有了暂停游戏、继续游戏或退出游戏的能力。不用担心将这个暂停屏幕添加到其他关卡中，因为我们在下一章中会这样做。说到下一章，在那里，我们将探讨如何更改 **音乐** 和 **效果** 滑块。现在，让我们回顾一下本章中我们所学到的内容。

# 摘要

通过完成本章，我们的游戏得到了进一步的改进，现在有了暂停屏幕，就像你从任何游戏中期望的那样。我们还学习了如何使用 **timeScale** 值在游戏中冻结时间。我们回顾了之前章节中的一些内容，例如事件监听器和 UI 定位和缩放，但我们还使用了其他 UI 组件，如切换和滑块，并将它们修改以适应我们的暂停屏幕。我们涵盖的其他内容还包括引入一些 MP3 音乐，并确保脚本知道何时淡入淡出和停止声音。

在你在这本书之外创建的下一个游戏中，你将不仅知道何时以及如何添加背景音乐以便在播放时播放，而且还将了解如何将你的音频附加到状态机。有了状态机，你可以使音乐在特定时刻播放、停止和淡出，例如当游戏屏幕暂停时。现在，你将能够使用本章中你学到的 UI 组件来创建你自己的菜单/暂停屏幕。通过这样做，你可以运行事件来关闭或恢复你的游戏。你还知道如何使用 **timeScale** 函数完全暂停你的游戏或减慢时间。

在下一章中，我们将探讨 Unity 的音频混音器，以控制玩家子弹和音乐的音量，并将其连接到我们的暂停屏幕音量滑块。我们还将探讨需要存储的不同类型的数据，例如游戏记住音量设置，这样我们就不必每次启动游戏时都调整滑块。

目前，我祝愿你在你的迷你模拟测试中一切顺利！

# 模拟测试

1.  如果你想在 `[Header]` 中保持一个私有变量可见，

1.  `[SerializeField]`

1.  `[AddComponentMenu]`

1.  `[Tooltip]`

+   你为移动设备创建了一个弹球游戏；游戏机制都运作良好，但你还需要应用一个暂停屏幕。显然，当玩家按下暂停时，整个游戏应该冻结。你将通过将 Unity 的 `timeScale` 设置为零来实现这一点。

当我们将 `Time.timeScale` 设置为 `0` 时，哪个时间属性不受影响？

1.  `captureFramerate`

1.  `frameCount`

1.  `realtimeSinceStartup`

1.  `timeSinceLevelLoad`

1.  在你的 `BuildSettings` 窗口中有一个场景列表。你知道第一个场景是你的标题场景，其余的则是你的游戏关卡场景。你的游戏设计师还没有确定场景的名称，并且一直在更改它们。作为一个程序员，你可以通过使用 `SceneManager` 的哪个方法来选择要加载的场景？

    1.  `GetSceneByBuildIndex()`

    1.  `GetActiveScene()`

    1.  `SceneManager.GetSceneByName()`

    1.  `SceneManager.GetSceneByPath()`

1.  如果你有一个可以启用或禁用的暂停屏幕，哪个是切换两个的最佳 UI 组件？

    1.  切换

    1.  `Button`

    1.  `Slider`

    1.  `Scroll Rect`

1.  你收到了一个移动设备上的 Killer Wave 新原型进行测试。你注意到当你进入屏幕中间有 UI 文本的水平时，你可以移动船只或射击。是什么导致了移动的限制？

    1.  屏幕比例的比率未校准。

    1.  UI 文本已勾选 Raycast Target。

    1.  移动设备需要充电。

    1.  屏幕上同时有太多手指会混淆输入系统。

1.  你创建了一个 UI 按钮，当你的账户中有钱时，它会显示硬币的图片，而当你的账户为空时，它会显示一个空空的棕色袋子。

在 Unity 检查器中，按钮的过渡字段应设置为哪个值以支持这些图像更改？

1.  颜色色调

1.  无

1.  动画

1.  精灵交换

1.  当在屏幕底部输入一些 UI 详细信息以显示玩家的生命值和所在关卡时，你注意到需要文本具有特定的大小。你可以将文本更改为任何大小，但还需要适应屏幕的比率。

最好的方法来修改字体以确保它不会显得挤压？

1.  减小字体大小。

1.  打开最佳适配。

1.  将垂直溢出设置为截断。

1.  将水平溢出设置为溢出。

1.  你开始制作一个游戏，该游戏依赖于时间暂停、倒退和快进，但仅限于你的敌人，利用`Time.timeScale`功能。一些敌人没有受到时间变化的影响。

哪个属性值可能会在敌人的**Animator**组件中引起这种情况？

1.  将`Update Mode`设置为动画物理。

1.  将`Culling Mode`设置为完全剔除。

1.  将`Culling Mode`设置为始终动画。

1.  将`Update Mode`设置为未缩放时间。

1.  你有一组番茄植物的游戏对象。番茄植物上的每个番茄都有一个名为`Tomato`的脚本。

为了避免番茄植物重复出现，一些艺术家已经关闭了`tomato`游戏对象，这样它们就看不见了。

在场景开始时，我们需要计算场景中包括隐藏的番茄总数。

哪个命令可以获取所有`Tomato`脚本的引用？

1.  `GetComponentsInChildren(typeof(Tomato), true)`

1.  `GetComponentInChildren(typeof(Tomato), true)`

1.  `GetComponentsInChildren(typeof(Tomato))`

1.  `GetComponenstInParent(typeof(Tomato), true)`

1.  哪个静态`Time`类属性会用来冻结时间？

    1.  `timeScale`

    1.  `maximumDeltaTime`

    1.  `captureFramerate`

    1.  `time`

1.  在状态机中，以下哪个选项对标签最有用？

    1.  枚举

    1.  字符串

    1.  浮点数

    1.  整数

1.  以下哪个与游戏中触发事件相关？

    1.  正在运行粒子效果。

    1.  玩家在菜单屏幕上闲置了 20 分钟。

    1.  玩家按下 UI 按钮。

    1.  玩家移动鼠标光标。

1.  你创建了一个游戏，你的玩家必须潜行并避开敌人。在其中一个任务中，你的玩家必须注意仓库中的敌人（听脚步声、谈话声等）。

你会为这个游戏添加哪个音频属性？

1.  为每个敌人添加音频源组件，将其空间混合设置为 3D，并播放声音。

1.  当敌人靠近时，使用音频混频器快照添加低通滤波器。

1.  测量每个敌人与玩家之间的距离，如果距离低于某个阈值，则播放声音。

1.  添加一个背景音乐播放的音频源，并根据最近敌人的距离增加或减少音量。

1.  在你的`CustomRolloff`

1.  `SpatialBlend`

1.  `ReverbZoneMix`

1.  `Spread`

+   你已经开始为你的游戏添加音乐和音效。在测试时，你注意到当播放某些音效时，背景音乐会中断。

在**音频源**组件中哪个属性可以解决这个问题，使得你的音乐不会中断？

1.  提高优先级。

1.  增加音量。

1.  增加 MinDistance。

1.  降低 SpatialBlend。

1.  你被要求制作一个 UI 菜单屏幕。你已经创建了一个**画布**，并将其**渲染模式**设置为**屏幕空间 - 叠加**。

在**画布缩放器**组件中，UI 缩放模式中的哪个属性可以使 UI 元素在屏幕大小变化时保持相同的像素大小？

1.  **恒定像素大小**

1.  **与屏幕大小缩放**

1.  **恒定物理大小**

1.  **禁用画布缩放器**

1.  当在**图像**组件中勾选“保留纵横比”复选框时，这会做什么？

    1.  将相机的纵横比设置为与图像的透视相匹配。

    1.  使图像与相机的相同纵横比匹配。

    1.  图片保留了其原始尺寸。

    1.  对**图像**组件没有影响，仅对精灵渲染器有影响。

1.  是否可以使用精灵渲染器代替**画布**中的**图像**组件？

    1.  不。尽管精灵渲染器可以在 2D/3D 空间中工作，但它不是用于**画布**的，因此不会工作。

    1.  是的，但精灵渲染器功能较少，是**图像**组件的较旧版本。

    1.  根据 Unity 项目，如果你的场景处于 2D 模式，是的。

    1.  是的，当用于动画精灵图集时。

1.  `Toggle`按钮是开启还是关闭

1.  当玩家在运行时按下它时，使图形激活或非激活。

1.  持有**勾选标记**图像

+   `parent`游戏对象。
