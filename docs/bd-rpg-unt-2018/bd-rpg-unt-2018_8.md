# 第八章：多人设置

每个独立游戏开发者的愿望都是制作一个多人游戏。现实是，创建多人游戏是困难的。作为游戏设计师/开发者，您需要考虑很多场景。除了创建在线多人游戏所固有的技术复杂性外，还有游戏玩法元素需要考虑。

本章的目的是为您提供一个使用新的 Unity 网络范式对开箱即用的网络功能的好概述。这是一个复杂的话题，因此我们无法在本章中涵盖所有内容。要真正深入了解细节，将需要一本全新的书籍。

话虽如此，我已经准备了这一章，包括一个简单的项目，该项目将用于说明网络的基础知识。然后我会向您展示如何使我们的游戏对象具备网络功能。

在本章中，我们将涵盖以下内容：

+   头戴式显示器

+   完成 HUD 设计

+   集成代码

这里我们开始！

# 多人游戏的挑战

一个普遍的规则是，如果您不需要使您的游戏成为多人游戏，那么就别这么做！这只会增加大量的复杂性和额外的需求和规范，您将需要开始担心。但如果是必须的，那么就必须这么做！

您现在可能已经知道，即使是创建最简单的多人游戏也有其挑战，作为游戏设计师，您需要解决这些问题。存在不同类型的在线多人游戏设计，如下所示：

+   实时多人游戏

+   轮流制多人游戏

+   异步多人游戏

+   本地多人游戏

在所有不同类型的多人游戏中，最具挑战性的是实时多人游戏。这是因为所有玩家必须在任何给定时间以适当和有效的方式与最新的游戏状态同步。

也就是说，如果我们让玩家 A 执行特定的动作，玩家 B 将同时在他的或她的屏幕上看到这个动作。现在，考虑我们还有另一个玩家加入，比如说玩家 C；玩家 A 和 B 需要与玩家 C 同步，而玩家 C 反过来需要将其自己的环境与玩家 A 和玩家 B 的状态同步。

不仅玩家的实际位置/旋转需要同步，所有玩家数据也需要同步。现在，想象一下当您将这个数字乘以 100、1,000 或 1,000,000 个连接的玩家时会发生什么。

对于现实世界的多人游戏，我们在这里涵盖的内容是不够的，Unity 提供的开箱即用的功能也不够。很可能您需要编写自己的服务器端代码来处理玩家数据。

现在，您可以看到设计和开发多人游戏所涉及的挑战，我们可以从构建我们的第一个多人游戏开始。

# 初始多人游戏

了解多人游戏的最佳方式是通过查看一个简单的示例。以下项目基于 Unity 网络教程，但已经扩展了一些其他功能，这些功能将有助于我们在 RPG 中实现网络功能。

# 基本网络组件

自从本书第一版以来，Unity 引擎内部发生了许多技术和架构上的变化。自第一版出版以来，网络领域有了显著改进。

让我们看看如何快速开始。首先你需要做的是从资源商店下载 *网络大厅* 资产，如下截图所示：

![](img/00169.jpeg)

这是一个很好的起点，因为我们有一个通用的网络大厅可以开始，所以我们不必浪费时间从头开始创建一切！

我们需要熟悉一些将用于创建我们的网络游戏的一些网络组件。这些组件如下：

+   网络管理器：`NetworkManager` 是一个高级类，允许你控制网络游戏的网络状态。它提供了一个编辑器界面来控制网络的配置、用于实例化的预制体以及用于不同网络游戏状态的场景。

+   网络大厅管理器：`NetworkLobbyManager` 是一种特殊的 `NetworkManager` 类型，在进入游戏的主玩场景之前提供多人游戏大厅。非常适合匹配。

+   网络管理器 HUD：这提供了一个默认的用户界面来控制游戏的网络状态。它还显示了编辑器中 `NetworkManager` 的当前状态信息。

+   网络身份：`NetworkIdentity` 组件是新的网络系统的核心。该组件控制对象的网络身份，并使网络系统了解它。

+   网络变换：`NetworkTransform` 组件同步网络中游戏对象的移动。该组件考虑了权限，因此 `LocalPlayer` 对象从客户端同步位置到服务器，然后同步到其他客户端。其他对象（具有服务器权限）从服务器同步位置到客户端。

# 我的坦克网络项目

假设你已经下载并导入了网络大厅资产。我们将使用两个场景来展示这个概念。第一个场景将是大厅场景，第二个场景将是游戏进行的场景。

继续创建一个场景，并将其命名为 `NetworkingGameLobby`。游戏将从这里开始。请看以下截图：

![](img/00170.jpeg)

我的坦克网络大厅

将`LobbyManager`预制体添加到场景中。在场景中选择`LobbyManager`游戏对象，并在检查器窗口中注意`LobbyManager`组件。这里有几个需要注意的地方。首先，你需要分配大厅场景和游戏场景属性。我们需要将我们的`NetworkingGameLobby`场景分配给大厅场景属性，将`NetworkingGamePlay`场景分配给游戏场景属性。接下来，你需要将`Game Player`预制体属性分配给`Player`预制体。别担心，我们将在下一部分创建`Player`预制体。最后，我们需要配置出生信息部分，并注册游戏过程中将生成的预制体（游戏对象）；在我们的例子中，这些将是子弹和敌方坦克。

# 添加玩家角色

我们现在将添加一个简单的玩家角色。你可以使用任何原始游戏对象来表示你的 PC；我将创建一个玩家，使其形状为一个简单的坦克。请看以下截图：

![图片](img/00171.jpeg)

下面的截图将展示`Tank`游戏对象的层次结构：

![图片](img/00172.jpeg)

我不会介绍如何创建游戏对象，因为你现在应该能够非常容易地做到这一点。我会介绍的是如何启用新的网络启用型坦克游戏对象。

我们将在`Tank`游戏对象上附加两个网络组件。第一个将是`NetworkIdentity`，可以通过选择`Tank`游戏对象，并在检查器窗口中选择添加组件 | 网络 | 网络身份来添加。

当你完成组件添加后，请确保检查本地玩家权限属性复选框，如图所示：

![图片](img/00173.jpeg)

本地玩家权限允许对象被拥有它的客户端控制。

接下来，我们需要将`NetworkTransform`组件添加到坦克游戏对象上。再次选择`Tank`游戏对象，在检查器窗口中，点击添加组件 | 网络 | 网络变换来添加组件：

![图片](img/00174.jpeg)

我们将保留`NetworkTransform`组件的默认值。除了旋转属性外，因为我们只沿*Y*轴旋转，所以我们不需要发送所有 XYZ 信息，因此将其更改为 Y（俯视 2D）并增加插值旋转字段到 15，并将压缩比更改为高。你可以通过在线文档自行了解更多关于不同属性的信息。你可能想要调整的主要属性是网络发送速率。

接下来，我们想要创建一个脚本，使我们能够控制坦克的移动。现在就创建一个新的 C#脚本，并将其命名为`MyPlayerController.cs`。

这里是脚本列表：

```cs
using UnityEngine;
using UnityEngine.Networking;
using System.Collections;
public class MyPlayerController : NetworkBehaviour
{
public Transform mainCamera;
public float cameraDistance = 16f;
public float cameraHeight = 16f;
public Vector3 cameraOffset;
[SyncVar]
public Color myColor;
public GameObject bulletPrefab;
public Transform bulletSpawn;
private void Start()
{
GetComponent<MeshRenderer>().material.color = myColor;
cameraOffset = new Vector3(0f, cameraHeight, -cameraDistance);
mainCamera = Camera.main.transform;
MoveCamera();
}
void Update()
{
// only execute the following code if local player ...
if (!isLocalPlayer)
return;
#if UNITY_EDITOR
var x = Input.GetAxis("Horizontal") * Time.deltaTime * 150.0f;
var z = Input.GetAxis("Vertical") * Time.deltaTime * 3.0f;
#else
var x = ETCInput.GetAxis("Horizontal") * Time.deltaTime * 150.0f;
var z = ETCInput.GetAxis("Vertical") * Time.deltaTime * 3.0f;
#endif
transform.Rotate(0, x, 0);
transform.Translate(0, 0, z);
#if UNITY_EDITOR
if (Input.GetKeyDown(KeyCode.Space))
{
CmdFire();
}
#else
if (ETCInput.GetButtonDown("ButtonFire"))
{
CmdFire();
}
#endif
MoveCamera();
}
[Command]
void CmdFire()
{
// Create the Bullet from the Bullet Prefab
var bullet = Instantiate(
bulletPrefab,
bulletSpawn.position,
bulletSpawn.rotation) as GameObject;
// Add velocity to the bullet
bullet.GetComponent<Rigidbody>().velocity = bullet.transform.forward *6;
bullet.GetComponent<Bullet>().myColor = myColor;
// Spawn the bullet on the Clients
NetworkServer.Spawn(bullet);
// Destroy the bullet after 2 seconds
Destroy(bullet, 2.0f);
}
void MoveCamera()
{
mainCamera.position = transform.position;
mainCamera.rotation = transform.rotation;
mainCamera.Translate(cameraOffset);
mainCamera.LookAt(transform);
}
}
```

代码很简单，但有一些重要的概念我们需要讨论。首先，你会注意到我们是从`NetworkBehaviour`继承，而不是从`MonoBehaviour`继承。

`NetworkBehaviour`用于与具有`NetworkIdentity`组件的对象一起工作。这允许你执行与网络相关的函数，如`Commands`、`ClientRPCs`、`SyncEvents`和`SyncVars`。

# 变量同步

同步变量是多玩家游戏的重要方面之一。如果你还记得，多玩家游戏的一个挑战是确保游戏的所有关键数据在服务器和客户端之间同步。这是通过`SyncVar`属性实现的。你将在下一个脚本中看到这一点，我们将为 Unity 的健康状态创建这个脚本。

# 网络回调

网络回调是针对`NetworkBehaviour`脚本在多种网络事件上调用的函数。它们列示如下：

+   `OnStartServer()`在服务器上创建对象时或当服务器为场景中的对象启动时被调用

+   `OnStartClient()`在客户端上创建对象时或当客户端连接到服务器以处理场景中的对象时被调用

+   `OnSerialize()`被调用以收集从服务器发送到客户端的状态

+   `OnDeSerialize()`被调用以将状态应用于客户端上的对象

+   `OnNetworkDestroy()`在服务器告诉对象被销毁时在客户端上被调用

+   `OnStartLocalPlayer()`在本地客户端的玩家对象上被调用

+   `OnRebuildObservers()`在服务器上被调用，当对象的观察者集合被重建时

+   `OnSetLocalVisibility()`在主机上被调用，当对象的可见性对本地客户端发生变化时

+   `OnCheckObserver()`在服务器上被调用，用于检查新客户端的可见状态

在`PlayerController.cs`脚本中，你会注意到我们正在使用`OnStartClient()`通过将材质颜色更改为蓝色来突出显示本地玩家。

# 发送命令

命令是客户端请求在服务器上执行函数的方式。在服务器权威系统中，客户端只能通过命令做事。命令在服务器上对应发送命令的客户端的玩家对象上运行。这种路由是自动发生的，因此客户端不可能为不同的玩家发送命令。

命令必须以前缀“Cmd”开始，并在其上具有`[Command]`自定义属性。

在我们的`PlayerController.cs`脚本中，当玩家开火时，它使用`CmdFire()`函数向服务器发送命令。

# 客户端 RPC 调用

客户端 RPC 调用是服务器对象在客户端对象上引发事件的一种方式。这与命令发送消息的方向相反，但概念是相同的。然而，客户端 RPC 调用不仅可以在玩家对象上调用；它们也可以在任何`NetworkIdentity`对象上调用。它们必须以“Rpc”前缀开始，并具有`[ClientRPC]`自定义属性。

你将在我们接下来要创建的`Health.cs`脚本中看到这个示例。

我们还需要一种方法来跟踪玩家角色的健康状态。这将通过一个新的脚本`Health.cs`来实现。

脚本列表如下：

```cs
using UnityEngine;
using UnityEngine.Networking;
public class Health : NetworkBehaviour
{
public const int maxHealth = 100;
[SyncVar(hook = "OnChangeHealth")]
public int currentHealth = maxHealth;
public RectTransform healthBar;
public bool destroyOnDeath;
public GameObject[] listOfPlayers;
private void Start()
{
healthBar.sizeDelta = new Vector2(currentHealth, healthBar.sizeDelta.y);
}
public void TakeDamage(int amount)
{
currentHealth -= amount;
if (currentHealth <= 0)
{
if (destroyOnDeath)
{
RpcDied();
listOfPlayers = GameObject.FindGameObjectsWithTag("Player");
if (listOfPlayers.Length < 1)
{
Invoke("BackToLobby", 3.0f);
}
}
else
{
currentHealth = maxHealth;
```

```cs

// called on the Server, will be invoked on the Clients
RpcRespawn();
}
}
}
void OnChangeHealth(int health)
{
healthBar.sizeDelta = new Vector2(health, healthBar.sizeDelta.y);
}
[ClientRpc]
void RpcRespawn()
{
if (isLocalPlayer)
{
// move back to zero location
transform.position = Vector3.zero;
}
}
[ClientRpc]
void RpcDied()
{
gameObject.tag = "Untagged";
GetComponent<Renderer>().material.color = Color.black;
if (GetComponent<MyPlayerController>() != null)
{
GetComponent<MyPlayerController>().enabled = false;
}
if (GetComponent<EnemyController>() != null)
{
GetComponent<EnemyController>().enabled = false;
}
}
void BackToLobby()
{
FindObjectOfType<NetworkLobbyManager>().ServerReturnToLobby();
}
}
```

注意，在这个脚本中，我们还在继承自`NetworkBehaviour`。我想重点介绍的两个主要项目是`SyncVar`、`ClientRpc`和`Start()`函数。

我们希望在网络中同步玩家的健康状态。为此，我们使用`SyncVarNetworkBehaviour`。`SyncVar`可以是任何基本类型，但不能是类、列表或其他集合。

当服务器上*SyncVar*的值发生变化时，它将被发送到游戏中所有准备好的客户端。当对象被实例化时，它们将在客户端上创建，并具有来自服务器的所有*SyncVars*的最新状态。

`OnStartClient()`函数确保所有附加了`Health.cs`脚本的物体都将具有最新的值，以便在健康条 UI 上显示。

我想在这里花点时间确保我给你一个关键提示。假设我们正在运行一个网络游戏会话，我们有`主机`、`玩家 A`和`玩家 B`连接并正在忙于自己的事情。在游戏过程中，`玩家 A`和`玩家 B`的健康值发生变化。现在，有第三个玩家连接到游戏，即`玩家 C`。如果没有实现`Start()`，`玩家 C`的客户端将具有所有带有`Health.cs`脚本的 GameObject 的正确数据同步；然而，数据将不会正确反映在 UI 上，因为我们需要一个触发器来实现这一点。这可以在`Start()`函数中处理，如代码所示。

下一个函数是`RpcRespawn()`函数。在`TakeDamage()`函数中，我们检查当前 GameObject 的健康值。如果健康值降至零以下，我们检查`destroyOnDeath`布尔变量是否设置。如果没有设置，我们将`currentHealth`值重置为`maxHealth`值，并使用`RpcRespawn()`方法在原点重新生成玩家。记住，这个函数在所有客户端上都会执行！

在函数内部，我们通过检查`isLocalPlayer`变量来查看调用者是否是本地玩家。是的，创建多人游戏确实会让人感到困惑！当你开始更多地进行实验时，这一点将变得更加明显。

# 创建坦克的炮弹

让我们创建一个预制体来代表我们的炮弹！非常简单，创建一个球体并使其与坦克炮的喷嘴大小相同。

我们需要将以下组件附加到炮弹游戏对象上：`NetworkIdentity`、`NetworkTransform`、`Rigidbody`和`Bullet.cs`脚本。

确保你在`Rigidbody`组件中将`Use Gravity`属性设置为`False`。同时，确保在`NetworkIdentity`组件中，`Server Only`和`Local Player Authority`属性都设置为`False`。在`NetworkTransform`组件中，将`Network Send Rate`改为`0`。一旦我们在服务器上生成了对象，物理引擎将负责每个客户端上的运动。

创建一个新的 C#脚本，命名为`Bullet.cs`。

脚本列表如下：

```cs
using UnityEngine;
using UnityEngine.Networking;
public class Bullet : NetworkBehaviour
{
[SyncVar]
public Color myColor;
private void Start()
{
GetComponent<MeshRenderer>().material.color = myColor;
}
void OnCollisionEnter(Collision collision)
{
var hit = collision.gameObject;
var health = hit.GetComponent<Health>();
if (health != null)
{
health.TakeDamage(10);
}
Destroy(gameObject);
}
}
```

我们在这里所做的只是检测碰撞。如果有碰撞，我们获取`Health`组件。如果`Health`组件不为空，我们调用`TakeDamage()`函数并传递一个值。

如果你还记得`Health.cs`脚本，`TakeDamage()`函数会减少玩家的`currentHealth`，这反过来是一个`SyncVar`，会在所有活动客户端上更新。

我们没有讨论的一个概念是`hook`。一个`SyncVar`可以有一个`hook`。将`hook`想象成一个事件处理器。`hook`属性可以用来指定当`SyncVar`在客户端上更改值时要调用的函数：

```cs
[SyncVar(hook = "OnChangeHealth")]
public int currentHealth = maxHealth;
```

`OnChangeHealth()`函数负责更新 UI 画布以显示我们的健康值：

```cs
void OnChangeHealth(int health)
{
healthBar.sizeDelta = new Vector2(health, healthBar.sizeDelta.y);
}
```

接下来，也创建炮弹的预制体，并将其实例从场景中删除。

确保你已经为每个脚本分配了所需的正确预制体关联。例如，`Tank`游戏对象的`PlayerController.cs`脚本需要一个对炮弹预制体的引用，以及大炮的生成位置。`Health.cs`脚本需要一个对健康条前景图像的引用等等。

# 创建坦克预制体并配置网络大堂管理员

现在我们已经创建了我们的`Tank`游戏对象，并将其所有必要的组件和脚本附加到它上面，我们需要为其创建一个预制体。这是因为我们将让`NetworkLobbyManger`生成我们的玩家角色，为了使其能够这样做，它需要引用一个代表你的玩家角色的预制体。

`NetworkLobbyManager`有一个`Spawn Info`部分，你可以在这里分配`Player`预制体，确定`NetworkLobbyManager`是否可以自动创建玩家以及`Player Spawn`方法。

也有一个`Registered Spawnable`预制体部分。我们需要注册所有将由`NetworkServer`生成的游戏对象。例如，炮弹预制体需要在这里注册，这样我们就可以在不同的客户端上通过网络生成它。

在场景中选择`Network Lobby Manager`游戏对象，在检查器窗口中根据需要分配适当的预制体。

下面是此时`NetworkLobbyManager`的截图：

![图片](img/00175.jpeg)

大堂管理员

到目前为止，你已经准备好测试我们所构建的内容。使用构建设置窗口创建游戏的独立版本。一旦你的构建就绪，启动两个应用程序实例。我们将使用一个实例来托管游戏，另一个作为客户端连接，如图所示：

![截图](img/00176.jpeg)

以下截图展示了运行游戏实例时的外观：

![截图](img/00177.jpeg)

以下截图展示了运行时屏幕的外观。游戏是在 Android 平板电脑上创建的：

![截图](img/00178.jpeg)

第二个玩家选择“列出服务器”并获取可用匹配，然后点击“加入”进入房间，如图所示：

![截图](img/00179.jpeg)

一旦所有人都准备好了，游戏就开始。注意，每个玩家都有一个独特的坦克颜色，这是基于他们在大厅中的颜色选择分配的。

在前面的截图中，请注意每个客户端都突出显示了它控制的玩家角色，即它控制的坦克。捕捉火命令将很困难，但你可以使用空格键发射加农炮，它将在所有活动客户端上相应触发。

你还会注意到，如果坦克受到攻击，它们的健康值将准确反映。现在我们准备创建一个敌方角色来展示游戏中的非玩家角色。

# 添加敌方坦克

现在是时候在我们的多人演示中添加一些非玩家角色了。添加敌方坦克将很简单，因为我们将以坦克预制体为基础。将坦克预制体拖放到场景中，并将其名称更改为`TankEnemy`。

从 GameObject 中移除`MyPlayerCharacter.cs`脚本。我们将创建一个独立的脚本作为敌方坦克的控制脚本。我还在敌方坦克上应用了不同的材质，以便我们可以从视觉上区分哪些坦克将由玩家控制，哪些不是。请看以下截图：

![截图](img/00180.jpeg)

敌方坦克设置

前面的截图展示了你的`Tank`和`TankEnemy`预制体应该看起来是什么样子。两者之间的主要区别是控制脚本。坦克有`MylayerController.cs`脚本，而`TankEnemy`有`EnemyController.cs`脚本。

`EnemyController.cs`脚本的列表如下：

```cs
using UnityEngine;
using UnityEngine.Networking;
public class EnemyController : NetworkBehaviour
{
public GameObject bulletPrefab;
public Transform bulletSpawn;
public float distance = 1000;
public GameObject[] listOfPlayers;
[SyncVar(hook = "OnChangePlayerToAttack")
public GameObject playerToAttack;
float coolOffTime = 0.0f;
void Update()
{
// only execute the following code if local player ...
if (!isServer)
return;
listOfPlayers = GameObject.FindGameObjectsWithTag("Player");
if (listOfPlayers.Length > 0)
{
float distance = 100f;
foreach (var player in listOfPlayers)
{
float d = Vector3.Distance(transform.position, player.transform.position);
if (d < distance)
{
distance = d;
playerToAttack = player;
}
}
if (playerToAttack != null)
{
Vector3 direction = playerToAttack.transform.position - transform.position;
transform.rotation =
Quaternion.Slerp(transform.rotation,
Quaternion.LookRotation(direction), 0.1f);
float d = Vector3.Distance(transform.position, playerToAttack.transform.position);
if (d < 15.0f)
{
if(coolOffTime<Time.time)
{
CmdFire();
coolOffTime = Time.time + 1.0f;
}
}
}
}
}
void OnChangePlayerToAttack(GameObject player)
{
playerToAttack = player;
}
[Command]
void CmdFire()
{
// Create the Bullet from the Bullet Prefab
var bullet = Instantiate(
bulletPrefab,
bulletSpawn.position,
bulletSpawn.rotation) as GameObject;
// Add velocity to the bullet
bullet.GetComponent<Rigidbody>().velocity = bullet.transform.forward * 6;
// Spawn the bullet on the Clients
NetworkServer.Spawn(bullet);
// Destroy the bullet after 2 seconds
Destroy(bullet, 2.0f);
}
}
```

脚本持续搜索场景中所有活跃的玩家，并为他们创建一个列表。然后它找到离自己最近的一个。一旦确定哪个玩家最近，它就会旋转以面对玩家。

之后，它计算自己与所选玩家的距离；如果距离小于可接受的阈值，它就开始向玩家开火。每次敌方坦克开火时，它实际上会调用一个名为`[Command]`的`CmdFire()`函数。

此函数在服务器上运行；它实例化一个炮弹预制体，并在网络上生成它。

`EnemyController.cs` 脚本也有一个 `SyncVar` 用于 `playertoAttack` 变量，并附加了一个作为 `OnChangePlayerToAttack()` 函数的 `hook`。这反过来确保所有客户端都更新了每个敌方 `Tank` GameObject 的最新数据。

`Health.cs` 脚本在 `Tank` GameObject 上的工作方式相同。

我们还需要讨论另一个项目：服务器通过生成 `Enemy Tanks`。我们可以通过创建另一个空的 GameObject 并将其命名为 `Enemy Spawner` 来轻松完成此操作。我们需要附加一个 `NetworkIdentity` 组件，并确保将 `Server Only` 属性设置为 `True`。这将确保只有服务器可以实例化敌方对象。

下一步是创建 `EnemySpawner.cs` 脚本。如下所示：

```cs
using UnityEngine;
using UnityEngine.Networking;
public class EnemySpawner : NetworkBehaviour {
public GameObject enemyPrefab;
public int numberOfEnemies;
public override void OnStartServer()
{
for (int i = 0; i < numberOfEnemies; i++)
{
var spawnPosition = new Vector3(
Random.Range(-20.0f, 20.0f),
0.0f,
Random.Range(-20.0f, 20.0f));
var spawnRotation = Quaternion.Euler(
0.0f,
Random.Range(0, 180),
0.0f);
var enemy = (GameObject)Instantiate(enemyPrefab, spawnPosition, spawnRotation);
NetworkServer.Spawn(enemy);
}
}
}
```

这段代码在技术上以提供的预制件作为敌方坦克，并在网络上随机在范围内生成每个敌方坦克。

确保所有预制件都已分配到检查器窗口中的 `Enemy Spawner` GameObject 和 `TankEnemy` GameObject。如果你还没有这样做，创建一个 `TankEnemy` 的预制件，并将其从场景中删除。不要删除 `Enemy Spawner`。

我们需要将 `TankEnemy` 预制件注册到 `NetworkLobbyManager`。请继续选择 `Network Manager` GameObject，并在检查器窗口中，将一个新的预制件添加到 `Registered Spawnable` 预制件选项中。

你的 `Network Manager` 应该如下所示：

![](img/00181.jpeg)

大厅网络管理器设置

# 构建和测试

我们已经准备好进行最终测试。请继续构建项目的独立版本，并启动一个新的游戏实例。玩家 1 将创建房间，其余玩家将加入游戏，如下面的截图所示：

![](img/00182.jpeg)

加入游戏演示

你也会注意到，初始化后，所有敌方坦克都会转向玩家角色坦克，如果在其射程内，它们将开始对其开火。

以下截图说明了游戏的运行时状态：

![](img/00183.jpeg)

注意，当我试图捕获屏幕时，敌方坦克无情地连续向玩家 4 开火。你可以看到我的生命条大幅减少。你也可以看到敌方坦克之一也受到了一些伤害。

我向你保证，我与敌方坦克受到的伤害无关；实际上，这是由友军火力造成的。是的，目前，敌方坦克还不够聪明，无法在另一名团队成员处于火力线时停止开火！

我将让你自己处理这个实现的细节。不应该太复杂。

使用射线投射来确保在开火之前敌方坦克和玩家之间没有物体。

恭喜！你刚刚创建了你第一个多人游戏！如前所述，创建、维护和托管多人游戏是一项不小的任务，在几页纸上涵盖每个方面是如何做的，简直是不可行的。

这里的想法是给你提供一个基础和基础，你可以在此基础上扩展。我鼓励你花些时间研究我们刚刚覆盖的内容，并对材料进行更多阅读，即使资料不多。事实是，你将需要自己进行大量的实验和尝试。

现在我们已经了解了基础知识，让我们将所学应用到我们的 RPG 资产上。

# 网络化 RPG 角色

为了让生活更简单，我决定创建一个新的场景，该场景将用于测试和实现我们的网络启用角色。这个例子将向你展示如何使玩家角色网络化，以及如何同步玩家角色数据，如库存物品，通过网络，以及提供你使非玩家角色网络化并使其数据在客户端之间同步的能力。

# 为我们的 RPG 创建一个场景

就像上一节中的坦克演示一样，我们将使用两个新的场景来展示 RPG 网络概念。我们将创建两个场景，一个叫做 `NetworkingMenu`，另一个叫做 `DeathMatchMultiplayer`。

`NetworkingMenu` 场景是我们的大厅场景，所以它将与坦克多人游戏大厅场景完全相同，除了玩家角色将被野蛮人预制件替换，`Registered Spawnable` 预制件将不同。我们稍后会看到这一点。

在 `DeathMatchMultiplayer` 场景中，我的以下级别设计如下：

![图片](img/00184.jpeg)

级别设计生成敌人

你的大厅场景应该看起来像以下这样：

![图片](img/00185.jpeg)

下一步是确保我们的 GameObject 启用网络功能。

# 网络化玩家角色

请将你创建的玩家预制件拖入场景。我们将将其用作创建新预制件的基础，该预制件将用于游戏的网络版本。

请从实例中移除现有的 `BarbarianCharacterController.cs` 和 `BarbarianCharacterCustomization.cs` 组件。我们将创建新的脚本，这些脚本将启用网络功能并使用它们。将 `PC` GameObject 实例重命名为 `PC-C6-Network`。现在创建实例的预制件。你现在应该有一个名为 `PC-C6-Network` 的新预制件。

请使用“检查器”窗口，选择添加组件 *|* 网络 *|* <组件名称>，将以下组件附加到预制件：`NetworkIdentiy`、`NetworkTransform` 和 `NetworkAnimator`。

在 `NetworkIdentity` 组件上，将 `Local Player Authority` 设置为 `True`。在 `NetworkTransform` 组件中，将变换同步模式更改为同步 Rigidbody 3D。在 `NetworkAnimator` 组件中，您需要将附加到 GameObject 上的 `Animator` 组件拖放到 `Animator` 槽中。

您需要选择 `Animator` 组件，并将其拖放到 `NetworkAnimator` 组件上的 `Animator` 槽中。

接下来，我们需要创建一个新的角色控制器，使其具有网络兼容性。

创建一个新的 C# 脚本，命名为 `BarbarianCharacterNetworkController.cs`。将脚本附加到 `PC-C6-Network` 预制件上。新的角色控制器是原始角色控制器的一个简化版本。

它的列表如下：

```cs
using UnityEngine;
using UnityEngine.Networking;
namespace com.noorcon.rpg2e
{
public class BarbarianCharacterNetworkController : NetworkBehaviour
{
public Transform mainCamera;
public float cameraDistance = 16f;
public float cameraHeight = 16f;
public Vector3 cameraOffset;
public Animator animator;
public float directionDampTime;
public float speed = 6.0f;
public float h = 0.0f;
public float v = 0.0f;
bool attack = false;
bool punch = false;
bool run = false;
bool jump = false;
[HideInInspector]
public bool die = false;
bool dead = false;
[SyncVar]
public bool EnemyInSight;
public GameObject EnemyToAttack;
Quaternion StartingAttackAngle = Quaternion.AngleAxis(-25, Vector3.up);
Quaternion StepAttackAngle = Quaternion.AngleAxis(5, Vector3.up);
Vector3 AttackDistance = new Vector3(0, 0, 2);
// Use this for initialization
void Start()
{
cameraOffset = new Vector3(0f, cameraHeight, -cameraDistance);
mainCamera = Camera.main.transform;
MoveCamera();
animator = GetComponent<Animator>() as Animator;
EnemyInSight = false;
}
// Update is called once per frame
private Vector3 moveDirection = Vector3.zero;
void Update()
{
if (!isLocalPlayer)
return;
if (dead)
{
animator.SetBool("Die", false);
return;
}
if (Input.GetKeyDown(KeyCode.C))
{
attack = true;
this.GetComponent<IKHandle>().enabled = false;
}
if (Input.GetKeyUp(KeyCode.C))
{
attack = false;
this.GetComponent<IKHandle>().enabled = true;
}
animator.SetBool("Attack", attack);
if (Input.GetKeyDown(KeyCode.P))
{
punch = true;
this.GetComponent<IKHandle>().enabled = false;
}
if (Input.GetKeyUp(KeyCode.P))
{
punch = false;
this.GetComponent<IKHandle>().enabled = true;
}
animator.SetBool("Punch", punch);
if (Input.GetKeyDown(KeyCode.LeftShift))
{
this.run = true;
this.GetComponent<IKHandle>().enabled = false;
}
if (Input.GetKeyUp(KeyCode.LeftShift))
{
this.run = false;
this.GetComponent<IKHandle>().enabled = true;
}
animator.SetBool("Run", run);
if (Input.GetKeyDown(KeyCode.Space))
{
jump = true;
this.GetComponent<IKHandle>().enabled = false;
}
if (Input.GetKeyUp(KeyCode.Space))
{
...
```

你首先应该注意到的是，我们正在从 `NetworkBehaviour` 继承，而不是 `MonoBehaviour`。如果我们想在 GameObject 上启用某些网络行为，这是必要的。看看下面的截图：

![img/00186.jpeg]

RPG 网络级别设置

接下来，让我们看看需要同步到网络上的某些变量，这些变量适用于每个连接的玩家角色。这些变量是 `enemyToAttack` 和 `Health`。还有两个其他变量，`Shield` 和 `Helmet`，我们将在稍后讨论。

在 `Update()` 函数中，我们需要一种方法来检查并确保在给控制器机会执行玩家之前，它是本地玩家。这是通过以下代码检查当前客户端是否是本地玩家来完成的：

```cs
if (!isLocalPlayer)
return;
```

这将确保代码只为当前客户端（玩家）运行。`Update()` 函数中的其余代码检查敌人是否在视野中，并确保玩家角色面向敌人进行攻击。

如果玩家处于攻击模式，并且敌人在我们的视野中，我们将 `enemyInSight` 设置为 `True`，并将 `enemyToAttack` 设置为存储在 `hitAttack` 变量中的敌人 GameObject，该变量类型为 `RacastHit`。这里的重要元素是 `CmdEnemyToAttack()` 函数。客户端需要向服务器发送一个命令，告诉服务器攻击的目标是谁：

```cs
[Command]
void CmdEnemyToAttack(GameObject go)
{
this.enemyInSight = true;
this.enemyToAttack = go;
}
```

这将确保数据在服务器上正确注册，并且与其他客户端同步。我们还有一个名为 `CmdEnemyTakeDamage()` 的函数，用于在服务器上减少敌人角色的健康值。然后服务器调用 `RpcEnemyTakeDamage()` 函数来同步所有客户端上的敌人健康值：

```cs
[Command]
void CmdEnemyTakeDamage(float value)
{
RpcEnemyTakeDamage(value);
}
[ClientRpc]
void RpcEnemyTakeDamage(float value)
{
if(this.enemyToAttack != null)
this.enemyToAttack.GetComponent<NPC_Movement_Network>().Damage(value);
}
```

这个想法一开始有点令人困惑，但随着你开始更仔细地研究它，它将变得更加清晰。

我们还有一个函数用于在玩家死亡时向服务器发送命令：

```cs
[Command]
void CmdPlayerCharacterIsDead()
{
RpcPlayerCharacterIsDead();
}
[ClientRpc]
void RpcPlayerCharacterIsDead()
{
this.die = true;
Destroy(this.gameObject, 2.0f);
}
```

前面的函数确保在游戏时刻，玩家角色在所有连接的客户端上死亡并被销毁。

最后，以下钩子函数被用于健康和 `enemyToAttack` 变量的 `SyncVar`：

```cs
// Var Sync hook function ...
void OnChangePlayerHealth(float health)
{
this.Health = health;
}
// Var Sync hook function
void OnChangeEnemyToAttack(GameObject enemy)
{
this.enemyToAttack = enemy;
}
```

这个想法一开始可能有点令人困惑，但随着你开始更仔细地研究它，它将会变得清晰起来。

如果你还没有这样做，请应用并保存你所有的更改到你的`PC-CC-Network`预制件。

在这个阶段，你的角色已经准备好与`NetworkManager`*；*集成；你可以将预制件拖放到`Player`预制件槽中，并构建一个独立版本来测试你的角色移动和同步。

# 网络化非玩家角色

就像玩家角色网络启用预制件一样，我们将使用非玩家角色预制件作为我们的基础来开始。请创建场景中 NPC 的一个实例。

现在请从预制件中移除现有的`NPC_Movement.cs`组件。将预制件重命名为`B1-Network`，并通过从检查器窗口中选择`Add Component` *|* Network *|* <component name>来附加以下组件：`NetworkIdentity`、`NetworkTransform`和`NetworkAnimator`。

在`NetworkIdentity`组件中，将`Local Player Authority`设置为`True`*；*在`NetworkTransform`组件中，将`Transform Sync Mode`设置为`Sync Transform`；对于`NetworkAnimator`组件，将`Animator`槽设置为附加到预制件的`Animator`控制器，通过将其拖放到槽中，如图所示：

![图片](img/00187.jpeg)

我们现在需要为我们的 NPC 移动创建一个新的网络启用脚本。请创建一个新的 C#脚本，并将其命名为`NPC_Movement_Network.cs`。脚本列表如下：

```cs
using UnityEngine; 
using UnityEngine.Networking; 

using System.Collections; 

public class NPC_Movement_Network : NetworkBehaviour { 
  // reference to the animator 
  public Animator animator; 

  public bool jump = false;     // used for jumping 

  [SyncVar(hook ="OnNPCIsDead")] 
  public bool die = false;      // are we alive? 

...

  // what is the field of view for our NPC? 
  // currently set to 110 degrees 
  [SyncVar] 
  public float fieldOfViewAngle = 110.0f; 

  // calculate the angle between PC and NPC 
  [SyncVar] 
  public float calculatedAngle; 

  [SyncVar(hook = "OnChangePlayerToAttackInNPC")] 
  public GameObject playerToAttack; 

  [SyncVar(hook = "OnChangeNPCHealth")] 
  public float Health = 100.0f; 

  void Awake() 
  { 
    // get reference to the animator component 
    this.animator = GetComponent<Animator>() as Animator; 

    // get reference to nav mesh agent  
    this.nav = GetComponent<NavMeshAgent>() as NavMeshAgent; 

    // get reference to the sphere collider 
    this.col = GetComponent<SphereCollider>() as SphereCollider; 

    // we don't see the player by default 
    this.playerInSight = false; 
  } 

  void Update() 
  { 
    // only execute the following code if local player ... 
    if (!isServer) 
      return; 

    this.CmdUpdateNetwork(); 
  } 

  [Command] 
  void CmdUpdateNetwork() 
  { 
    this.RpcUpdateNetwork(); 
  } 

  [ClientRpc] 
  void RpcUpdateNetwork() 
  { 
    // if player is in sight let's slerp towards the player 
    if(this.playerToAttack!=null) 
    { 
      if (playerInSight) 
      { 
        this.transform.rotation = 
            Quaternion.Slerp(this.transform.rotation, 
            Quaternion.LookRotation(direction), 0.1f); 

        if (this.playerToAttack.transform.GetComponent<CharacterController_Network>().die) 
        { 
          animator.SetBool("Attack", false); 
          animator.SetFloat("Speed", 0.0f); 
          animator.SetFloat("AngularSpeed", 0.0f); 
          this.playerInSight = false; 
          this.playerToAttack = null; 
        } 
      } 
    } 

    if(this.Health<=0.0f) 
    { 
      this.die = true; 
      this.Health = 0.0f; 

      animator.SetBool("Attack", false); 
      animator.SetFloat("Speed", 0.0f); 
      animator.SetFloat("AngularSpeed", 0.0f); 
      this.playerInSight = false; 
      this.playerToAttack = null; 
    }
```

```cs
    animator.SetBool("Die", die); 
  } 
...
```

有几个变量已被标记为`SyncVars`*；*这些是：`die`、`distance`、`direction`、`angle`、`playerInSight`、`fieldOfViewAngle`、`calculatedAngle`、`playerToAttack`和`Health`。

看看下面的代码：

```cs
  void OnTriggerExit(Collider other) 
  { 
    if (other.transform.tag.Equals("Player")) 
    { 
      distance = 0.0f; 
      angle = 0.0f; 
      this.attack1 = false; 
      this.playerInSight = false; 

      this.playerToAttack = null; 
    } 
  } 

  // this is a helper function at this point 
  // in the future we will use it to calculate distance around the corners 
  // it currently is also used to draw the path of the nav mesh agent in the  
  // editor 
  float CalculatePathLength(Vector3 targetPosition) 
  { 
    // Create a path and set it based on a target position. 
    NavMeshPath path = new NavMeshPath(); 
    if (nav.enabled) 
      nav.CalculatePath(targetPosition, path); 

    // Create an array of points which is the length of the number of corners in the path + 2\. 
    Vector3[] allWayPoints = new Vector3[path.corners.Length + 2]; 

    // The first point is the enemy's position. 
    allWayPoints[0] = transform.position; 

    // The last point is the target position. 
    allWayPoints[allWayPoints.Length - 1] = targetPosition; 

    // The points inbetween are the corners of the path. 
    for (int i = 0; i < path.corners.Length; i++) 
    { 
      allWayPoints[i + 1] = path.corners[i]; 
    } 

    // Create a float to store the path length that is by default 0\. 
    float pathLength = 0; 

    // Increment the path length by an amount equal to the distance between each waypoint and the next. 
    for (int i = 0; i < allWayPoints.Length - 1; i++) 
    { 
      pathLength += Vector3.Distance(allWayPoints[i], allWayPoints[i + 1]); 

      if (DEBUG_DRAW) 
        Debug.DrawLine(allWayPoints[i], allWayPoints[i + 1], Color.red); 
    } 

    return pathLength; 
  } 
} 
```

一些`SyncVars`有`hook`*；*这些是`Health`、`playerToAttack`、`playerInSight`和`die`。

在`Update()`函数中，我们通过以下行来确保我们是服务器：

```cs
// only execute the following code if server ...
if (!isServer)
return;
```

如果我们是服务器，我们使用`CmdUpdateNetwork()`和`RpcUpdateNetwork()`函数来执行我们的职责。这些只是用于 NPC 的移动和动作。关键在于用于将 NPC 数据同步到所有客户端的`SyncVars`和`hook`函数：

```cs
public void OnChangePlayerPlayerInSight(bool value)
{
this.playerInSight = value;
}
// Var Sync hook function ...
void OnChangeNPCHealth(float health)
{
this.Health = health;
}
void OnNPCIsDead(bool value)
{
die = true;
}
void OnChangePlayerToAttackInNPC(GameObject player)
{
this.playerToAttack = player;
}
```

对于 NPC 来说，这就足够了。请将脚本添加到预制件中并应用更改。保存它。

你的新 NPC 预制件应该附加以下组件：

# 同步玩家自定义和物品

为了使这起作用，我们需要配置并创建几个更多的新的库存物品预制件。我将使用两个库存物品来演示这个特定的点。

我将使用我库存物品中的一个`Helmet`预制件。复制它，并移除`InventoryItemAgent.cs`组件。我们将创建一个新的网络启用脚本，就像我们为我们的 PC 和 NPC 所做的那样。

将以下组件附加到实例上：`NetworkIdentity`和`NetworkTransform`，从检查器窗口使用添加组件 | 网络 | `<组件名称>`：*

![](img/00188.jpeg)

创建一个名为`InventoryItemAgent_Network.cs`的新脚本。如下所示：

```cs
using UnityEngine; 
using UnityEngine.Networking; 
using System.Collections; 

public class InventoryItemAgent_Network : NetworkBehaviour { 

  public InventoryItem ItemDescription; 

  public void OnTriggerEnter(Collider c) 
  { 
    // make sure we are colliding with the player 
    if (c.gameObject.tag.Equals("Player")) 
    { 
      // Make a copy of the Inventory Item Object 
      InventoryItem myItem = new InventoryItem(); 
      myItem.CopyInventoryItem(this.ItemDescription); 

      c.gameObject.GetComponent<CharacterController_Network>().PlayerArmourChanged(myItem); 
    } 
  } 
} 
```

所有这些脚本所做的只是使用`CharacterController_Network.cs`脚本中的`PlayerArmourChanged()`函数将库存物品分配给玩家角色。

`PlayerArmourChanged()`函数使用我们需要创建的另一个脚本，这是一个网络启用脚本，即`CharacterCustomization_Network.cs`脚本。我不会在这里列出脚本，因为它非常长。您可以在书中提供的代码中查看脚本。

# 生成 NPC 和其他物品

我们需要一种方法来生成我们的 NPC 以及我们将用于下一个演示的库存物品。

在层次结构窗口中，右键单击并选择创建空对象*；*这将创建一个`Empty`GameObject。将其重命名为`SpawnEnemy`，并通过从检查器窗口选择添加组件 | 网络 | 网络身份将其添加一个`NetworkIdentity`组件。

我们将创建一个名为`EnemySpawn_Network.cs`的新脚本。如下所示：

```cs
using UnityEngine; 
using UnityEngine.Networking; // used for chapter 8 

using System.Collections; 

public class EnemySpawn_Network : NetworkBehaviour 
{ 
  public GameObject enemyPrefab; 
  public Transform spawnLocation; 

  public GameObject inventoryItemPrefab; 
  public GameObject inventoryItemShield; 

  public override void OnStartServer() 
  { 
    GameObject go = GameObject.Instantiate(enemyPrefab, spawnLocation.position, Quaternion.identity) as GameObject; 
    NetworkServer.Spawn(go); 

    GameObject goInventoryItem1 = GameObject.Instantiate(inventoryItemPrefab, new Vector3(2, 1, 2), Quaternion.identity) as GameObject; 
    NetworkServer.Spawn(goInventoryItem1); 

    GameObject goInventoryItem2 = GameObject.Instantiate(inventoryItemShield, new Vector3(3, 1, 2), Quaternion.identity) as GameObject; 
    NetworkServer.Spawn(goInventoryItem2); 

  } 
} 
```

如您所见，脚本非常简单。我们只是引用代表 NPC 和库存物品预制件的 GameObject。

在层次结构窗口中，将新脚本附加到`SpawnEnemy`预制件上。

# 测试我们的网络启用 PC 和 NPC

到目前为止，我们已经拥有了测试我们的网络启用 RPG 角色所需的全部资产。如果您还没有这样做，我们需要执行一个最后的步骤。

在层次结构窗口中选择`NetworkManager`GameObject，并在检查器窗口中，您需要确保在*生成*信息部分已经分配了某些内容。

*玩家预制件*应该分配给您的玩家角色预制件。我的名字叫`PC-CC-Network-1`。确保自动创建玩家设置为`True`。

你还需要在“已注册可生成预制件”中注册你的 NPC 预制件和其他网络启用非角色预制件。我已经将名为`B1-Network-1`的野蛮人预制件分配，`barbarian_helmet_01_LOD0_Network`，以及`shield_01_LOD0_Networ`*k*。

看看下面的屏幕截图：

![](img/00189.jpeg)

好吧，最后，我们可以进行构建。让我们继续制作我们游戏的独立构建。确保当前场景在构建配置中：

![](img/00190.jpeg)

好吧，继续启动两个构建实例。其中一个作为主机，另一个作为客户端。

看看下面的屏幕截图：

![](img/00191.jpeg)

在前面的屏幕截图中，我们已经启动了一个作为服务器的客户端，玩家角色已经拿起了一个库存物品，一个盾牌。当我们连接第二个客户端时，它应该正确考虑游戏中所有活跃的 PC 和 NPC 的当前状态：

![](img/00192.jpeg)

保持两个实例运行，使用 Unity IDE 连接第三个客户端。你可以使用客户端在客户端端进行调试，并查看正在发生的事情。

看看下面的截图：

![图片](img/00193.jpeg)

在前面的截图中，你可以看到所有玩家角色以及它们是如何被准确同步的。从层次窗口中选择 B1-Network-1 GameObject，并使用一个客户端实例来控制玩家角色攻击 NPC。

我们将暂停编辑器，检查变量以及它们是如何被正确同步的，如下面的截图所示：

![图片](img/00194.jpeg)

# 接下来是什么？

正如你所见证的，网络编程很简单，但同时也可能很困难。困难在于以高效和有意义的方式管理和理解所有玩家之间的数据同步。

如果你真正考虑创建一个拥有大量客户端的游戏，实际上可能会更加复杂。Unity 网络将无法处理这种情况；你需要创建自己的后端服务器管理器和消息系统。

本章所涵盖的内容将为你提供一个很好的理解，以便将其提升到下一个层次。继续编码，直到我们再次见面！

# 概述

在本章中，我们使用了 Unity 网络组件来研究网络编程。本章的主要目标是通过对两个示例的实现，向您介绍 Unity 网络的基础。

我们本章开始时讨论了作为多人游戏的游戏设计师和开发者可能会面临的挑战。提出的主要问题之一是，你是否真的需要投入时间和精力为你的游戏创建多人模式。假设你真的想要或需要创建多人游戏，我们首先研究了目前流行的不同类型的多人游戏。

然后，我们继续我们的第一个简化版多人游戏的示例。我们开发的多人游戏是实时的，也就是说，所有客户端都是基于每个活跃玩家状态的活动同步的；也就是说，位置、旋转、移动和其他重要数据需要与所有连接到游戏会话的客户端同步。

我们研究了 Unity 网络组件的基础，例如网络管理器、网络管理器 HUD、网络身份和网络变换。这些是用于说明 Unity 中多人编程的基础组件。一旦我们了解了这些组件的用途，我们就从一个简单的示例开始。

我们创建了一个简单的坦克游戏，展示了如何将多人游戏所需的所有基本组件组合在一起。我们创建了必要的玩家角色预制体，并添加了适当的网络启用脚本和组件。我们还创建了非玩家角色预制体，并为其添加了自身的网络启用脚本。游戏演示了如何生成玩家角色和非玩家角色，以及如何在它们之间进行同步。

在构建坦克游戏的过程中，我们介绍了如何同步变量和使用对多人游戏开发至关重要的网络回调。我们还解释了什么是*命令*和*ClientRPC*调用，以及如何正确使用它们。

接下来，我们将所学知识应用到之前章节中开发的 RPG 角色上。我们以现有的预制体为基础，扩展了它们以包含网络组件，并创建了新的网络启用脚本以处理角色移动、角色定制和非玩家角色移动脚本。

我们讨论的一个关键元素是，如何将每个玩家角色的库存物品在视觉上与其他玩家同步。我们在章节的最后通过测试和讨论如何在开发多人游戏时调试客户端和服务器上的代码来结束本章。
