# 第八章。继续智能敌人

这最后一章继续上一章的内容，通过关注智能敌人的理论和相关编码来完善 AI 项目。敌人将展示三种主要行为：巡逻、追逐和攻击。在本章中，我们将深入以下主题：

+   如何规划和编码敌方角色的 AI 系统

+   如何编码有限状态机（FSM）

+   如何创建视线功能

    ### 注意

    起始项目和资源可以在本书配套文件中的 `Chapter08/Start` 文件夹中找到。如果您还没有自己的项目，可以从这里开始，并跟随本章内容进行学习。

# 敌方 AI – 视野范围

让我们现在通过思考我们的功能需求来开始开发敌方 AI。场景中的敌人将开始巡逻模式，在关卡中四处游荡，寻找玩家角色。如果发现玩家，敌人将改变巡逻状态，开始追逐玩家，试图靠近他们进行攻击。如果敌人到达玩家的攻击范围内，敌人将改变追逐状态，转为攻击状态。如果玩家跑得比敌人快并成功摆脱他们，敌人应该停止追逐并再次回到巡逻状态，像最初一样寻找玩家。总的来说，这描述了我们需要的敌方 AI 行为。

为了实现这种行为，我们需要为敌人编码视线功能。敌人依赖于能够看到玩家角色或确定玩家在任何时刻是否对敌人可见。这有助于敌人决定他们是否应该巡逻或追逐玩家角色。为了编码这个功能，请参考以下来自源文件 `LineSight.cs` 的代码。这个脚本文件应该附加到上一章创建的敌方角色上。

```cs
using UnityEngine;
using System.Collections;
//------------------------------------------
public class LineSight : MonoBehaviour
{
  //------------------------------------------
  //How sensitive should we be to sight
  public enum SightSensitivity {STRICT, LOOSE};

  //Sight sensitivity
  public SightSensitivity Sensitity = SightSensitivity.STRICT;

  //Can we see target
  public bool CanSeeTarget = false;

  //FOV
  public float FieldOfView = 45f;

  //Reference to target
  private Transform Target = null;

  //Reference to eyes
  public Transform EyePoint = null;

  //Reference to transform component
  private Transform ThisTransform = null;

  //Reference to sphere collider
  private SphereCollider ThisCollider = null;

  //Reference to last know object sighting, if any
  public Vector3 LastKnowSighting = Vector3.zero;
  //------------------------------------------
  void Awake()
  {
    ThisTransform = GetComponent<Transform>();
    ThisCollider = GetComponent<SphereCollider>();
    LastKnowSighting = ThisTransform.position;
    Target = GameObject.FindGameObjectWithTag("Player").GetComponent<Transform>();
  }
  //------------------------------------------
  bool InFOV()
  {
    //Get direction to target
    Vector3 DirToTarget = Target.position - EyePoint.position;

    //Get angle between forward and look direction
    float Angle = Vector3.Angle(EyePoint.forward, DirToTarget);

    //Are we within field of view?
    if(Angle <= FieldOfView)
      return true;

    //Not within view
    return false;
  }
  //------------------------------------------
  bool ClearLineofSight()
  {
    RaycastHit Info;

    if(Physics.Raycast(EyePoint.position, (Target.position - EyePoint.position).normalized, out Info, ThisCollider.radius))
    {
      //If player, then can see player
      if(Info.transform.CompareTag("Player"))
        return true;
    }

    return false;
  }
  //------------------------------------------
  void UpdateSight()
  {
    switch(Sensitity)
    {
      case SightSensitivity.STRICT:
        CanSeeTarget = InFOV() && ClearLineofSight();
      break;

      case SightSensitivity.LOOSE:
        CanSeeTarget = InFOV() || ClearLineofSight();
      break;
    }
  }
  //------------------------------------------
  void OnTriggerStay(Collider Other)
  {
    UpdateSight();

    //Update last known sighting
    if(CanSeeTarget)
      LastKnowSighting =  Target.position;
  }
  //------------------------------------------
  void OnTriggerExit(Collider Other)
  {
    if(!Other.CompareTag("Player"))return;

    CanSeeTarget = false;
  }
  //------------------------------------------
}
//------------------------------------------
```

## 代码示例 8.1

以下要点总结了代码示例：

+   `LineSight` 类应该附加到任何敌方角色对象上。它的目的是计算玩家和敌人之间是否存在直接的视线。

+   `CanSeeTarget` 变量是一个布尔值（`True`/`False`），它每帧更新一次，以描述敌人是否现在（对于这一帧）可以看到玩家。`True` 表示玩家在敌人的视线中，而 `false` 表示玩家不可见。

+   `FieldOfView` 变量是一个浮点值，它决定了敌人眼睛点两侧的角边距，在这个范围内可以看见对象（如玩家）。这个值越高，敌人看到玩家的机会就越大。

+   `InFOV` 函数返回 `true` 或 `false`，以指示玩家是否在敌人的视野范围内。这忽略了玩家是否被墙壁或固体物体（如柱子）遮挡。它只是取敌人眼睛的位置，确定指向玩家的向量，并测量前向向量与玩家之间的角度。它将这个角度与视野进行比较，如果敌人与玩家之间的角度小于 `FieldOfView` 变量，则返回 `true`。简而言之，这个函数可以告诉你如果视线清晰，敌人是否能看到玩家。

+   `ClearLineOfSight` 函数返回 `true` 或 `false`，以指示敌人眼睛点和玩家之间是否存在任何物理障碍（碰撞器），如墙壁或道具。这不考虑玩家是否在敌人的视野范围内。这个函数与 `InFOV` 函数结合使用，可以确定敌人是否对玩家有清晰的视线并且在其视野范围内，从而确定玩家是否可见。

+   当玩家位于围绕敌人的触发体积内时，会调用 `OnTriggerStay` 和 `OnTriggerExit` 函数，而当玩家离开这个体积时也会调用。正如我们将看到的，可以将球体碰撞器附加到敌人角色对象上，以表示其视野范围。这意味着敌人可以看到玩家的总距离，或半径，前提是他们位于视野范围内并且存在清晰的视线。

现在，将 `LineSight.cs` 脚本文件附加到场景中的敌人角色上，以及一个球体碰撞器组件（标记为触发器），以近似敌人的观察视野。参见 *图 8.1*。将 **Field of View** 设置保留在 `45`，尽管如果需要，可以增加到大约 `90` 以调整敌人观察范围的有效性。

![代码示例 8.1](img/B05118_08_01.jpg)

图 8.1：为 NPC 添加视野

默认情况下，**Eye Point** 字段设置为 **None**，表示空值。这应该指的是敌人角色上的一个特定位置，该位置充当眼睛点——角色可以从中看到的地方。要创建这个点，使用应用程序菜单中的 **GameObject** | **Create Empty** 添加一个新的空游戏对象到场景中。将对象命名为 `Eye Point`，通过 Gizmo 图标从 **Inspector** 中激活其可见性（即使未选中也能可见），然后将它作为子对象添加到敌人上。之后，将对象定位到角色眼睛点，确保前向向量朝向相同的方向。参见 *图 8.2*：

![代码示例 8.1](img/B05118_08_02.jpg)

图 8.2：添加 EyePoint

现在，将**层次**面板中的 Eye Point 对象拖放到**检查器**中 LineSight 组件的**Eye Point**字段。这指定 Eye Point 对象为敌人角色的眼睛点。这将用于确定敌人是否能看见玩家。与使用通常位于脚部位置的字符位置相比，拥有这样一个独立的眼睛点对象是有用的。请参见*图 8.3*：

![代码示例 8.1](img/B05118_08_03.jpg)

图 8.3：定义 NPC 的眼睛点

最后，`LineSight`脚本通过首先在场景中使用**Player**标签找到玩家对象来确定玩家位置。因此，请确保**Player**使用**Player**标签进行了标记或标记。请参见*图 8.4*：

![代码示例 8.1](img/B05118_08_04.jpg)

图 8.4：标记玩家对象

现在运行一下你的游戏。当你接近 NPC 对象时，**Can See Target**字段将被启用。请参见*图 8.5*。做得好！视线功能现在已完成。让我们继续前进！

![代码示例 8.1](img/B05118_08_05.jpg)

图 8.5：测试视线功能

# 有限状态机的概述

要为 NPC 对象创建 AI，除了我们已有的视线代码外，我们还需要使用**有限状态机**（**FSM**）。FSM 不是 Unity 的*东西*或功能，也不是 C#语言的实体方面。相反，FSM 是一个概念、框架或想法，我们可以将其应用于代码以实现特定的 AI 行为。它源于对智能角色的特定思考方式。具体来说，我们可以总结我们的 NPC 在任意时刻存在于三种可能的状态之一。这些是巡逻（当敌人四处游荡时）、追逐（当敌人追击玩家时）和攻击（当敌人到达玩家并攻击时）。这些模式中的每一个都是一个状态，需要独特和特定的行为，敌人一次只能处于这三个状态中的一个。例如，敌人不能同时巡逻和追逐，或者巡逻和攻击，因为这不符合世界的逻辑和游戏。

除了状态本身之外，还有一组规则或状态之间的连接组，它决定了何时一个状态应该改变或移动到另一个状态。例如，NPC 只有在可以看到玩家且尚未攻击的情况下，才应该从巡逻状态移动到追逐状态。同样，NPC 只有在看不到玩家且尚未巡逻或追逐的情况下，才应该从攻击状态移动到巡逻状态。因此，状态及其连接规则的组合形成了一个有限状态机。因此，任何在功能上表示这种行为的代码实现都是 FSM。编码 FSM 本身没有对错之分。只是有不同方式，其中一些对于特定目的来说更好或更差。

在本节中，我们将使用协程编写 FSM。让我们首先创建主结构。参考文件 `AI_Enemy.cs` 中的以下代码：

```cs
using UnityEngine;
using System.Collections;
//------------------------------------------
public class AI_Enemy : MonoBehaviour
{
  //------------------------------------------
  public enum ENEMY_STATE {PATROL, CHASE, ATTACK};
  //------------------------------------------
  public ENEMY_STATE CurrentState
  {
    get{return currentstate;}

    set
    {
      //Update current state
      currentstate = value;

      //Stop all running coroutines
      StopAllCoroutines();

      switch(currentstate)
      {
        case ENEMY_STATE.PATROL:
          StartCoroutine(AIPatrol());
        break;

        case ENEMY_STATE.CHASE:
          StartCoroutine(AIChase());
        break;

        case ENEMY_STATE.ATTACK:
          StartCoroutine(AIAttack());
        break;
      }
    }
  }
  //------------------------------------------
  [SerializeField]
  private ENEMY_STATE currentstate = ENEMY_STATE.PATROL;

  //Reference to line of sight component
  private LineSight ThisLineSight = null;

  //Reference to nav mesh agent
  private NavMeshAgent ThisAgent = null;

  //Reference to player transform
  private Transform PlayerTransform = null;

  //------------------------------------------
  void Awake()
  {
    ThisLineSight = GetComponent<LineSight>();
    ThisAgent = GetComponent<NavMeshAgent>();
    PlayerTransform = GameObject.FindGameObjectWithTag("Player").GetComponent<Transform>();
  }
  //------------------------------------------
  void Start()
  {

    //Configure starting state
    CurrentState = ENEMY_STATE.PATROL;
  }
  //------------------------------------------
  public IEnumerator AIPatrol()
  {
      yield break;

  }
  //------------------------------------------
  public IEnumerator AIChase()
  {

      yield break;
  }
  //------------------------------------------
  public IEnumerator AIAttack()
  {
    yield break;
  }
  //------------------------------------------
}
//------------------------------------------
```

### 注意

更多关于协程的信息可以在 Unity 在线文档中找到：[`docs.unity3d.com/Manual/Coroutines.html`](http://docs.unity3d.com/Manual/Coroutines.html)。

## 代码示例 8.2

以下要点总结了代码示例：

+   到目前为止创建的 `AI_Enemy` 类并不代表完整的完整状态机（FSM），而只是其开始阶段的框架。它展示了总体结构。它为每个状态提供了一个协程。

+   `CurrentState` 变量定义了一个属性，用于选择活动状态，终止所有现有协程并启动相关的协程。

+   每个状态协程将在状态激活期间在一个帧安全的无穷循环中运行，允许敌人对象更新其行为，正如我们很快将看到的。

在继续之前，请确保 `AI_Enemy` 脚本已附加到 NPC 对象上。参见 *图 8.6*：

![代码示例 8.2](img/B05118_08_06.jpg)

图 8.6：将 AI 脚本附加到 NPC 角色上

# 巡逻状态

对于 NPC AI 需要实现的三个状态中的第一个是巡逻状态。在上一章中，我们配置了一个动画巡逻对象，NPC 应在此状态下持续跟随。巡逻对象在关卡内从预定义的动画资源中移动，从一个位置移动到下一个位置。然而，之前 NPC 只是简单地跟随这个对象，而没有尽头，而巡逻状态要求 NPC 考虑玩家是否在其路线上可见。如果可见，状态应该改变。为了支持这个功能，已经编写了 `AI_Enemy` 类的巡逻状态和启动函数，如下面的代码所示：

```cs
  void Start()
  {
    //Get random destination
    GameObject[] Destinations = GameObject.FindGameObjectsWithTag("Dest");
    PatrolDestination = Destinations[Random.Range(0, Destinations.Length)].GetComponent<Transform>();

    //Configure starting state
    CurrentState = ENEMY_STATE.PATROL;
  }
  //------------------------------------------
  public IEnumerator AIPatrol()
  {
    //Loop while patrolling
    while(currentstate == ENEMY_STATE.PATROL)
    {
      //Set strict search
      ThisLineSight.Sensitity = LineSight.SightSensitivity.STRICT;

      //Chase to patrol position
      ThisAgent.Resume();
      ThisAgent.SetDestination(PatrolDestination.position);

      //Wait until path is computed
      while(ThisAgent.pathPending)
        yield return null;

      //If we can see the target then start chasing
      if(ThisLineSight.CanSeeTarget)
      {
        ThisAgent.Stop();
        CurrentState = ENEMY_STATE.CHASE;
        yield break;
      }

      //Wait until next frame
      yield return null;
    }
  }
```

## 代码示例 8.3

以下要点总结了代码示例：

+   `Start` 函数将敌人角色的初始状态设置为巡逻。协程 AIPatrol 处理此状态。

+   `AIPatrol`协程在`Patrol`状态活动期间无限循环。请记住，在协程中使用无限循环并不一定是坏事，尤其是在与`yield`语句结合使用时。这允许长时间的行为可以整洁且易于随时间编码。

+   调用`SetDestination`函数将`NavMeshAgent`发送到指定的目标。这之后是一个`pathPending`检查，这是`NavMeshAgent`的一个变量。这个检查等待`pathPending`变量变为 false，这表示从源到目标已经计算出一个完整的可穿越路径。对于短而简单的旅程，路径可能几乎立即计算出来，但对于更复杂的路径，这可能需要更长的时间。

+   在巡逻状态期间，我们不断检查`LineSight`组件以确定敌人是否有直接视线到玩家。如果有，敌人将从`Patrol`状态变为`Chase`状态。

+   请记住，`yield return null`语句将暂停协程直到下一帧。

现在，如果您还没有这样做，请将`AIEnemy`脚本拖放到场景中的 NPC 角色上。`Patrol`模式被配置为跟踪一个移动对象，也就是说，敌人将跟随一个移动的目标。在上一章中，使用**动画**窗口移动一个对象在场景中随时间移动，从一个地方跳到另一个地方创建了一个移动目标。见*图 8.7*：

![代码示例 8.3](img/B05118_08_07.jpg)

图 8.7：创建可移动对象

要实现可移动对象，在场景中创建一个或多个目标对象并分配它们一个**Dest**标签。请记住，`AIEnemy`的`start`函数在场景中搜索所有标记为**Dest**的对象，并使用这些作为目标点。见*图 8.8*：

![代码示例 8.3](img/B05118_08_08.jpg)

图 8.8：标记目标对象

# 追逐状态

Chase 状态是敌人有限状态机（FSM）中的第二个状态。此状态直接连接到巡逻和攻击状态。它可以通过两种方式之一到达。如果一个巡逻的 NPC 建立了与玩家的直接视线，那么 NPC 将从巡逻状态变为追逐状态。相反，如果一个攻击的 NPC 超出了玩家的攻击范围（可能是因为他在逃跑），NPC 将再次采取追逐状态。从追逐状态本身，可以移动到巡逻或攻击状态，条件与追逐相反。也就是说，如果 NPC 失去了对玩家的视线，它将返回巡逻状态，如果 NPC 到达玩家攻击范围内，它将切换到攻击状态。考虑以下代码示例，它修改了`AIEnemy`类以支持追逐行为：

```cs
  public IEnumerator AIChase()
  {
    //Loop while chasing
    while(currentstate == ENEMY_STATE.CHASE)
    {
      //Set loose search
      ThisLineSight.Sensitity = LineSight.SightSensitivity.LOOSE;

      //Chase to last known position
      ThisAgent.Resume();
      ThisAgent.SetDestination(ThisLineSight.LastKnowSighting);

      //Wait until path is computed
      while(ThisAgent.pathPending)
        yield return null;

      //Have we reached destination?
      if(ThisAgent.remainingDistance <= ThisAgent.stoppingDistance)
      {
        //Stop agent
        ThisAgent.Stop();

        //Reached destination but cannot see player
        if(!ThisLineSight.CanSeeTarget)
          CurrentState = ENEMY_STATE.PATROL;
        else //Reached destination and can see player. Reached attacking distance
          CurrentState = ENEMY_STATE.ATTACK;

        yield break;
      }

      //Wait until next frame
      yield return null;
    }
  }
```

## 代码示例 8.4

以下要点总结了代码示例：

+   当进入追逐状态时，将启动`AIChase`协程，并且像巡逻状态一样，只要状态处于活动状态，它就会在一个帧安全的无限循环中重复。

+   `NavMeshAgent`类的`remainingDistance`成员变量用于确定 NPC 是否已经到达攻击玩家的距离范围内。

+   `LineSight`类的`CanSeeTarget`布尔变量指示玩家是否可见，并在选择 NPC 是否应该返回巡逻状态时具有影响力。

优秀的工作！现在在编辑器中运行代码，你将拥有一个可以巡逻和追逐的敌人角色。太棒了！

# 攻击状态

NPC 的第三个也是最后一个状态是攻击状态，在这个状态下，NPC 会持续攻击玩家。这个状态只能从追逐状态到达。在追逐过程中，NPC 必须确定他们是否已经到达攻击距离。如果是这样，NPC 必须从追逐状态变为攻击状态。如果在攻击过程中，玩家离开了攻击距离，那么 NPC 必须从攻击状态变为追逐状态。考虑以下代码示例，它包括了完整的`EnemyAI`类，以及所有编码和完成的状态：

```cs
using UnityEngine;
using System.Collections;
//------------------------------------------
public class AI_Enemy : MonoBehaviour
{
  //------------------------------------------
  public enum ENEMY_STATE {PATROL, CHASE, ATTACK};
  //------------------------------------------
  public ENEMY_STATE CurrentState
  {
    get{return currentstate;}

    set
    {
      //Update current state
      currentstate = value;

      //Stop all running coroutines
      StopAllCoroutines();

      switch(currentstate)
      {
        case ENEMY_STATE.PATROL:
          StartCoroutine(AIPatrol());
        break;

        case ENEMY_STATE.CHASE:
          StartCoroutine(AIChase());
        break;

        case ENEMY_STATE.ATTACK:
          StartCoroutine(AIAttack());
        break;
      }
    }
  }
  //------------------------------------------
  [SerializeField]
  private ENEMY_STATE currentstate = ENEMY_STATE.PATROL;

  //Reference to line of sight component
  private LineSight ThisLineSight = null;

  //Reference to nav mesh agent
  private NavMeshAgent ThisAgent = null;

  //Reference to player health
  private Health PlayerHealth = null;

  //Reference to player transform
  private Transform PlayerTransform = null;

  //Reference to patrol destination
  private Transform PatrolDestination = null;

  //Damage amount per second
  public float MaxDamage = 10f;
  //------------------------------------------
  void Awake()
  {
    ThisLineSight = GetComponent<LineSight>();
    ThisAgent = GetComponent<NavMeshAgent>();
    PlayerHealth = GameObject.FindGameObjectWithTag("Player").GetComponent<Health>();
    PlayerTransform = PlayerHealth.GetComponent<Transform>();
  }
  //------------------------------------------
  void Start()
  {
    //Get random destination
    GameObject[] Destinations = GameObject.FindGameObjectsWithTag("Dest");
    PatrolDestination = Destinations[Random.Range(0, Destinations.Length)].GetComponent<Transform>();

    //Configure starting state
    CurrentState = ENEMY_STATE.PATROL;
  }
  //------------------------------------------
  public IEnumerator AIPatrol()
  {
    //Loop while patrolling
    while(currentstate == ENEMY_STATE.PATROL)
    {
      //Set strict search
      ThisLineSight.Sensitity = LineSight.SightSensitivity.STRICT;

      //Chase to patrol position
      ThisAgent.Resume();
      ThisAgent.SetDestination(PatrolDestination.position);

      //Wait until path is computed
      while(ThisAgent.pathPending)
        yield return null;

      //If we can see the target then start chasing
      if(ThisLineSight.CanSeeTarget)
      {
        ThisAgent.Stop();
        CurrentState = ENEMY_STATE.CHASE;
        yield break;
      }

      //Wait until next frame
      yield return null;
    }
  }
  //------------------------------------------
  public IEnumerator AIChase()
  {
    //Loop while chasing
    while(currentstate == ENEMY_STATE.CHASE)
    {
      //Set loose search
      ThisLineSight.Sensitity = LineSight.SightSensitivity.LOOSE;

      //Chase to last known position
      ThisAgent.Resume();
      ThisAgent.SetDestination(ThisLineSight.LastKnowSighting);

      //Wait until path is computed
      while(ThisAgent.pathPending)
        yield return null;

      //Have we reached destination?
      if(ThisAgent.remainingDistance <= ThisAgent.stoppingDistance)
      {
        //Stop agent
        ThisAgent.Stop();

        //Reached destination but cannot see player
        if(!ThisLineSight.CanSeeTarget)
          CurrentState = ENEMY_STATE.PATROL;
        else //Reached destination and can see player. Reached attacking distance
          CurrentState = ENEMY_STATE.ATTACK;

        yield break;
      }

      //Wait until next frame
      yield return null;
    }
  }
  //------------------------------------------
  public IEnumerator AIAttack()
  {
    //Loop while chasing and attacking
    while(currentstate == ENEMY_STATE.ATTACK)
    {
      //Chase to player position
      ThisAgent.Resume();
      ThisAgent.SetDestination(PlayerTransform.position);

      //Wait until path is computed
      while(ThisAgent.pathPending)
        yield return null;

      //Has player run away?
      if(ThisAgent.remainingDistance > ThisAgent.stoppingDistance)
      {
        //Change back to chase
        CurrentState = ENEMY_STATE.CHASE;
        yield break;
      }
      else
      {
        //Attack
        PlayerHealth.HealthPoints -= MaxDamage * Time.deltaTime;
      }

      //Wait until next frame
      yield return null;
    }

    yield break;
  }
  //------------------------------------------
}
//------------------------------------------
```

## 代码示例 8.5

以下要点总结了代码示例：

+   `AIAttack`协程在攻击状态激活期间（在这个状态下敌人将会攻击）运行在一个帧安全的无限循环中。

+   `MaxDamage`变量指定了敌人每秒对玩家造成的伤害量。

+   `AIAttack`协程依赖于`Health`组件来造成伤害。这是一个额外的自定义组件，用于编码健康值。玩家和敌人都应该有一个健康组件来表示他们的健康状态。

健康脚本（`Health.cs`）被`AIEnemy`类（攻击状态）引用，用于对玩家造成伤害。因此，玩家角色需要附加一个`Health`组件。该组件的代码包含在下面的代码示例中：

```cs
using UnityEngine;
using System.Collections;

public class Health : MonoBehaviour 
{
  public float HealthPoints
  {
    get{return healthPoints;}
    set
    {
      healthPoints = value;

      //If health is < 0 then die
      if(healthPoints <= 0)
        Destroy(gameObject);
    }
  }

  [SerializeField]
  private float healthPoints = 100f;
}
```

`Health`脚本相当简单。它维护一个数值健康值，当减少到`0`或以下时，将销毁宿主游戏对象。这至少应该附加到玩家角色上，允许 NPC 在接近时造成伤害。然而，它也可以附加到 NPC 对象上，允许玩家进行反击。参见*图 8.9*：

![代码示例 8.5](img/B05118_08_09.jpg)

图 8.9：配置玩家健康

太好了，我们几乎准备好测试这个项目了。首先，如果你还没有这样做，通过将 NPC 游戏对象从**场景**视图或**层次结构**面板拖放到**项目**面板来从敌人对象创建一个预制体。然后，将你想要的敌人数量添加到关卡中。参见*图 8.10*：

![代码示例 8.5](img/B05118_08_10.jpg)

图 8.10：创建 NPC 预制体

现在，通过在工具栏上按下播放图标来测试水平，你应该拥有一个完整的环境，其中智能敌人可以以相当逼真的程度寻找、追逐和攻击玩家。在某些情况下，你可能需要调整或细化敌人的视野范围（FOV），以更好地匹配你的环境和角色类型。干得好！参见*图 8.11*：

![代码示例 8.5](img/B05118_08_11.jpg)

图 8.11：完成的关卡

# 摘要

太棒了！你现在已经完成了 AI 项目，也完成了这本书。在完成这个项目的过程中，你组装了一个完整的地图、一个 NPC 预制件以及一系列协同工作的脚本，这些脚本策略性地创建出智能的外观，这对于 AI 来说已经足够好了。对于游戏来说，AI 不过是指那些*看起来智能*的对象以及创建它们的技巧。太棒了！此外，在完成这本书并完成所有四个项目后，你现在已经具备了一组多样且至关重要的技能，可以开发 2D 和 3D 的专业级游戏。
