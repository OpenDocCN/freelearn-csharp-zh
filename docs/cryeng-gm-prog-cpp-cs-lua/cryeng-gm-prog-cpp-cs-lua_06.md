# 第六章。人工智能

CryENGINE AI 系统允许创建在游戏世界中漫游的非玩家控制角色。

在本章中我们将：

+   了解 AI 系统如何与 Lua 脚本集成

+   了解目标管道是什么，以及如何创建它们

+   使用 AI 信号

+   注册自定义 AI`Actor`类

+   学习如何使用行为选择树

+   创建我们自己的 AI 行为

# 人工智能（AI）系统

CryENGINE AI 系统的设计是为了方便创建灵活到足以处理更大量的复杂和不同世界的自定义 AI 角色。

在我们开始研究 AI 系统的本地实现之前，我们必须提到一个非常重要的事实：AI 不同于角色，不应该混淆。

在 CryENGINE 中，AI 仍然依赖于底层的角色实现，通常与玩家使用的完全相同。然而，AI 本身的实现是通过 AI 系统单独完成的，该系统将移动输入等发送给角色。

## 脚本

CryENGINE 的 AI 系统的主要思想是基于大量的脚本编写。可以使用`Scripts/AI`和`Scripts/Entities/AI`目录中包含的 Lua 脚本来创建新的 AI 行为，而不是强迫程序员修改复杂的 CryAISystem 模块。

### 注意

AI 系统目前主要是硬编码为使用`.lua`脚本，因此我们将无法在 AI 开发中更大程度地使用 C#和 C++。

## AI 角色

正如我们之前提到的，角色与 AI 本身是分开的。基本上这意味着我们需要创建一个`IActor`实现，然后指定角色应该使用哪种 AI 行为。

如果您的 AI 角色应该与您的玩家行为大致相同，您应该重用角色实现。

如前一章所述，注册一个角色可以通过`REGISTER_FACTORY`宏来完成。AI 角色的唯一区别是最后一个参数应该设置为 true 而不是 false：

```cs
  REGISTER_FACTORY(pFramework, "MyAIActor", CMyAIActor, true);
```

一旦注册，AI 系统将在`Scripts/Entities/AI`中搜索以您的实体命名的 Lua 脚本。在前面的片段中，系统将尝试加载`Scripts/Entities/AI/MyAIActor.lua`。

这个脚本应该包含一个同名的表，并且与其他 Lua 实体的功能相同。例如，要添加编辑器属性，只需在 Properties 子表中添加变量。

## 目标管道

目标管道定义了一组目标操作，允许在运行时触发一组目标。例如，一个目标管道可以包括 AI，增加其移动速度，同时开始搜索玩家控制的单位。

目标操作，如 LookAt，Locate 和 Hide 是在`CryAISystem.dll`中创建的，不能在没有访问其源代码的情况下进行修改。

### 创建自定义管道

管道最初是在`PipeManager:CreateGoalPipes`函数中在`Scripts/AI/GoalPipes/PipeManager.lua`中注册的，使用`AI.LoadGoalPipes`函数：

```cs
  AI.LoadGoalPipes("Scripts/AI/GoalPipes/MyGoalPipes.xml");
```

这段代码将加载`Scripts/AI/GoalPipes/MyGoalPipes.xml`，其中可能包含以下目标管道定义：

```cs
<GoalPipes>
  <GoalPipe name="myGoalPipes_findPlayer">
    <Locate name="player" />
    <Speed id="Run"/>
    <Script code="entity.Behavior:AnalyzeSituation(entity);"
  </GoalPipe>
</GoalPipes>
```

当选择了这个管道时，分配的 AI 将开始定位玩家，切换到`Run`移动速度状态，并调用当前选定的行为脚本中包含的`AnalyzeSituation`函数。

目标管道可以非常有效地推动一组目标，例如基于前面的脚本，我们可以简单地选择`myGoalPipes_findPlayer`管道，以便 AI 寻找玩家。

### 选择管道

目标管道通常使用 Lua 中的实体函数`SelectPipe`来触发：

```cs
  myEntity:SelectPipe(0, "myGoalPipe");
```

或者也可以通过 C++触发，使用`IPipeUser::SelectPipe`函数。

## 信号

为了为 AI 实体提供直观的相互通信方式，我们可以使用信号系统。信号是可以从另一个 AI 实体或从 C++或 Lua 代码的其他地方发送到特定 AI 单元的事件。

信号可以使用 Lua 中的`AI.Signal`函数或 C++中的`IAISystem::SendSignal`发送。

## AI 行为

每个角色都需要分配行为，并且它们定义了单位的决策能力。通过在运行时使用**行为选择树**选择行为，角色可以给人一种动态调整到周围环境的印象。

使用放置在`Scripts/AI/SelectionTrees`中的 XML 文件创建行为选择树。每个树管理一组**行为叶子**，每个叶子代表一种可以根据条件启用的 AI 行为类型。

![AI 行为](img/5909_06_01.jpg)

### 样本

例如，如下所示，查看选择树 XML 定义的非常基本形式：

```cs
<SelectionTrees>
  <SelectionTree name="SelectionTreeSample" type="BehaviorSelectionTree">
    <Variables>
      <Variable name="IsEnemyClose"/>
    </Variables>
    <SignalVariables>
      <Signal name="OnEnemySeen" variable="IsEnemyClose" value="true"/>
      <Signal name="OnNoTarget" variable="IsEnemyClose" value="false"/>
      <Signal name="OnLostSightOfTarget" variable="IsEnemyClose" value="false"/>
    </SignalVariables>
    <LeafTranslations />
    <Priority name="Root">
      <Leaf name="BehaviorSampleCombat" condition="IsEnemyClose"/>
      <Leaf name="BehaviorSampleIdle"/>
    </Priority>
  </SelectionTree>
</SelectionTrees>
```

为了更好地理解示例，我们将对其进行一些分解：

```cs
  <SelectionTree name="SelectionTreeSample" type="BehaviorSelectionTree">
```

这个第一个片段只是定义了选择树的名称，并且在 AI 初始化期间将被 AI 系统解析。如果要重命名树，只需更改`name`属性：

```cs
<Variables>
  <Variable name="IsEnemyClose"/>
</Variables>
```

每个选择树可以定义一组变量，这些变量可以根据信号（请参见下一个片段）或在每个行为脚本内部进行设置。

变量只是可以查询的布尔条件，以确定下一个叶子或行为选择：

```cs
<SignalVariables>
  <Signal name="OnEnemySeen" variable="IsEnemyClose" value="true"/>
  <Signal name="OnNoTarget" variable="IsEnemyClose" value="false"/>
  <Signal name="OnLostSightOfTarget" variable="IsEnemyClose" value="false"/>
</SignalVariables>
```

每个行为树还可以监听诸如`OnEnemySeen`之类的信号，以便轻松设置变量的值。例如，在我们刚刚看到的片段中，当发现敌人时，`IsEnemyClose`变量将始终设置为 true，然后在目标丢失时设置为 false。

然后我们可以在查询新叶子时使用变量（请参见下面的代码片段），允许 AI 根据简单的信号事件切换到不同的行为脚本：

```cs
<Priority name="Root">
  <Leaf name="BehaviorSampleCombat" condition="IsEnemyClose"/>
  <Leaf name="BehaviorSampleIdle"/>
</Priority>
```

通过在`Priority`元素内指定叶子，我们可以根据简单的条件在运行时启用行为（叶子）。

例如，前面的片段将在敌人接近时启用`BehaviorSampleCombat`行为脚本，否则将退回到`BehaviorSampleIdle`行为。

### 注意

行为选择树系统将按顺序查询叶子，并退回到最后剩下的叶子。在这种情况下，它将首先查询`BehaviorSampleCombat`，然后在`IsEnemyClose`变量设置为 false 时退回到`BehaviorSampleIdle`。

## IAIObject

已向 AI 系统注册的实体可以调用`IEntity::GetAI`来获取它们的`IAIObject`指针。

通过访问实体的 AI 对象指针，我们可以在运行时操纵 AI，例如设置自定义信号，然后在我们的 AI 行为脚本中拦截：

```cs
if(IAIObject *pAI = pEntity->GetAI())
{
  gEnv->pAISystem->SendSignal(SIGNALFILTER_SENDER, 0, "OnMySignal", pAI);
}
```

# 创建自定义 AI

创建自定义 AI 的过程相对简单，特别是如果您对上一章介绍的角色系统感到满意。

每个角色都有两个部分；它的`IActor`实现和 AI 实体定义。

## 注册 AI 角色实现

AI 角色通常使用与玩家相同的`IActor`实现，或者至少是共享的派生。

### 在 C#中

在 C#中注册 AI 角色与我们在第五章中所做的非常相似，*创建自定义角色*。基本上，我们只需要从`CryEngine.AIActor`派生，而不是`CryEngine.Actor`。

`AIActor`类直接从`Actor`派生，因此不会牺牲任何回调和成员。但是，必须明确实现它，以使 CryENGINE 将此角色视为由 AI 控制。

```cs
public class MyCSharpAIActor
: CryEngine.AIActor
{
}
```

现在，您应该能够在 Sandbox 中的**Entity**浏览器中的**AI**类别中放置您的实体：

![在 C#中](img/5909_06_02.jpg)

### 在 C++中

与我们刚刚看到的 C#角色一样，注册一个角色到 AI 系统并不需要太多工作。只需从我们在上一章创建的角色实现派生即可：

```cs
class CMyAIActor
  : public CMyCppActor
{
};
```

然后打开你的 GameDLL 的`GameFactory.cpp`文件，并使用相同的设置来注册角色，只是最后一个参数应该是 true，告诉 CryENGINE 这种角色类型将由 AI 控制：

```cs
  REGISTER_FACTORY(pFramework, "MyAIActor", CMyAIActor, true);
```

在重新编译后，你的角色现在应该出现在**实体**浏览器中的**AI**实体类别中。

## 创建 AI 实体定义

当我们的 AI 角色生成时，AI 系统将搜索 AI 实体定义。这些定义用于设置角色的默认属性，例如其编辑器属性。

我们需要做的第一件事是打开`Scripts/Entities/AI`并创建一个与我们的`Actor`类同名的新的`.lua`文件。在我们的情况下，这将是为了刚刚创建的 C++实现的`MyAIActor.lua`，以及为了 C#角色的`MyCSharpAIActor.lua`。

脚本保持了最少量的代码，因为我们只需要加载基本 AI。基本 AI 是使用`Script.ReloadScript`函数加载的。

默认情况下，CryENGINE 使用`Scripts/Entities/AI/Shared/BasicAI.lua`作为基本 AI 定义。我们将使用自定义实现，`Scripts/Entities/AI/AISample_x.lua`，以减少与本章节无关的不必要代码：

```cs
  Script.ReloadScript( "SCRIPTS/Entities/AI/AISample_x.lua");
--------------------------------------------------------------

  MyCSharpAIActor = CreateAI(AISample_x);
```

就是这样！你的 AI 现在已经正确注册，现在应该可以通过编辑器放置。

### 注意

有关基本 AI 定义的更多信息，请参见本章后面的*AI 基本定义分解*部分。

## AI 行为和角色

当我们生成自定义 AI 角色时，默认情况下应该出现四个实体属性。这些属性确定 AI 应该使用哪些系统进行决策：

![AI 行为和角色](img/5909_06_03.jpg)

### 理解和使用行为选择树

行为选择树是我们的 AI 角色最重要的实体属性，因为它确定了角色使用哪个行为选择树。如果我们的项目包含多个行为选择树，我们可以轻松生成行为非常不同的多个 AI 角色，因为它们使用了不同的选择树。选择树系统存在是为了提供一种在运行时查询和选择行为脚本的方法。

要查看当前可用的树，或创建自己的树，请导航至`Scripts/AI/SelectionTrees`。对于我们的示例，我们将使用`Scripts/AI/SelectionTrees/FogOfWar.xml`中的`FogOfWar`选择树：

```cs
<SelectionTree name="FogOfWar" type="BehaviorSelectionTree">
  <Variables>
    <Variable name="IsFar"/>
    <Variable name="IsClose"/>
    <Variable name="AwareOfPlayer"/>
  </Variables>
  <SignalVariables>
    <Signal name="OnEnemySeen" variable="AwareOfPlayer" value="true"/>
    <Signal name="OnNoTarget" variable="AwareOfPlayer" value="false"/>
    <Signal name="OnLostSightOfTarget" variable="AwareOfPlayer" value="false"/>
  </SignalVariables>
  <LeafTranslations />
  <Priority name="Root">
    <Leaf name="FogOfWarSeekST" condition="IsFar"/>
    <Leaf name="FogOfWarEscapeST" condition="IsClose"/>
    <Leaf name="FogOfWarAttackST" condition="AwareOfPlayer"/>
    <Leaf name="FogOfWarIdleST"/>
  </Priority>
</SelectionTree>
```

#### 变量

每个选择树都公开一组可以在运行时设置的变量。叶子将查询这些变量，以确定激活哪种行为。

#### 信号变量

信号变量提供了一种在接收到信号时设置变量的简单方法。

例如，在前面的树中，我们可以看到当接收到`OnEnemySeen`信号时，`AwareOfPlayer`会动态设置。然后当 AI 失去对玩家的追踪时，这些变量将被设置为 false。

#### 叶子/行为查询

叶子确定根据变量条件播放哪种行为。

在前面的树中，我们可以看到当所有其他条件都设置为 false 时，默认情况下会激活`FogOfWarIdleST`行为。但是，假设`IsFar`变量设置为 true，系统将自动切换到`FogOfWarSeekST`行为。

### 注意

行为从`Scripts/AI/Behaviors/Personalities/`目录中加载，在我们的情况下，它将在`Scripts/AI/Behaviors/Personalities/FogOfWarST/`中找到参考行为。

### 角色

`Character`属性用于设置角色的 AI 角色。

### 注意

在我们的示例中，`Character`属性将默认为空字符串，因为自从引入行为选择树以来，该系统被视为已弃用（请查看*理解和使用行为选择树*部分）。

AI 角色包含在`Scripts/AI/Characters/Personalities`中，以`.lua`脚本的形式。例如，我们可以打开并修改`Scripts/AI/Characters/Personalities/FogOfWar.lua`以修改我们的默认个性。

您还可以通过在`Personalities`目录中添加新文件，以`FogOfWar`作为基线，来创建新的个性。

`Character`属性定义了所有适用的行为，在我们的例子中是`FogOfWarAttack`、`FogOfWarSeek`、`FogOfWarEscape`和`FogOfWarIdle`。角色将能够在运行时根据内部和外部条件在这些行为之间切换。

### 导航类型

`NavigationType`属性确定要使用哪种类型的 AI 导航。这允许系统动态确定哪些路径适用于该类型的 AI。

在我们的示例中，默认为 MediumSizedCharacter，并且可以设置为包含在`Scripts/AI/Navigation.xml`中的任何导航定义。

## 创建自定义行为

我们快要完成了！唯一剩下的步骤是理解如何创建和修改 AI 行为，使用我们之前描述的行为选择树来激活。

首先，使用您选择的文本编辑器打开`Scripts/AI/Behaviors/Personalities/FogOfWarST/FogOfWarIdleST.lua`。由于之前描述的行为树设置，这是在所有其他变量都设置为 false 时将被激活的行为。

通过调用`CreateAIBehavior`函数来创建行为，第一个参数设置为新行为的名称，第二个包含行为本身的表。

因此，行为的最低要求是：

```cs
local Behavior = CreateAIBehavior("MyBehavior",
{
  Alertness = 0,

  Constructor = function (self, entity)
  end,

  Destructor = function(self, entity)
  end,
})
```

这段代码片段会始终将 AI 的`Alertness`设置为 0，并且在行为开始（`Constructor`）和结束（`Destructor`）时什么也不做。

通过查看`FogOfWarIdleST`行为定义，我们可以看到它的作用：

```cs
  Constructor = function (self, entity)
    Log("Idling...");
    AI.SetBehaviorVariable(entity.id, "AwareOfPlayer", false);
    entity:SelectPipe(0,"fow_idle_st");
  end,
```

当激活行为时，我们应该在控制台中看到“Idling…”，假设日志详细程度足够高（使用`log_verbosity CVar`设置）。

在记录之后，该行为将通过`AI.SetBehaviorVariable`函数将`AwareOfPlayer`变量重置为 false。我们可以随时使用该函数来改变变量的值，有效地告诉行为选择树应该查询另一个行为。

将变量设置为 false 后，构造函数会选择`fow_idle_st`目标管道。

### 监听信号

要在行为中监听信号，只需创建一个新函数：

```cs
OnMySignal = function(self, entity, sender)
{
}
```

当发送`OnMySignal`信号时，将调用此函数，并附带相关的实体和行为表。

# AI 基本定义分解

在本章中，我们之前创建了依赖于`Scripts/Entities/AI/AISample_x.lua`基本定义的自定义 AI 定义。本节将描述基本定义的作用，以便更好地理解定义设置。

首先，使用您选择的文本编辑器（例如 Notepad++）打开定义。

## AISample_x 表

当打开`AISample_x.lua`时，我们将看到的第一行代码是其表定义，它定义了每个角色的默认属性。

### 注意

每个 AI 定义都可以覆盖基本定义中设置的属性。

### 属性表

属性表的工作方式与标准 Lua 实体相同，用于定义在编辑器中选择实体时出现的属性。

### 注意

我们基本 AI 定义中的默认属性是从`CryAISystem.dll`中读取的。不支持删除这些属性，否则会导致 AI 初始化失败。

### AIMovementAbility 表

`AIMovementAbility`子表定义了我们角色的移动能力，例如行走和奔跑速度。

## CreateAI 函数

`CreateAI`函数将基本 AI 表与指定子表合并。这意味着在 AI 基本定义中存在的任何表都将存在于从中派生的任何 AI 定义中。

`CreateAI`函数还使实体可生成，并通过调用 AI 的`Expose()`函数将其暴露给网络。

## RegisterAI 函数

`RegisterAI`函数在应该将角色注册到 AI 系统时调用。这在实体生成时和编辑器属性更改时会自动调用。

# 总结

在本章中，我们已经了解了 AI 系统的核心思想和实现，并创建了自定义的 AI 角色实现。

通过创建我们自己的 AI 实体定义和行为选择树，您应该了解到在 CryENGINE 中如何创建 AI 角色。

现在，您应该对如何利用 AI 系统有了很好的理解，从而可以创建巡逻游戏世界的 AI 控制单位。

如果您对 AI 还没有完全掌握，为什么不尝试利用您新获得的知识来创建自己选择的更复杂的东西呢？

在下一章中，我们将介绍创建自定义用户界面的过程，允许创建主菜单和**抬头显示**（**HUD**）。
