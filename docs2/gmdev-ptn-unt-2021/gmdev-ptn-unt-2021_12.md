使用访问者模式实现升级

在本章中，我们将为我们的游戏实现一个升级机制。自游戏早期开始，升级一直是视频游戏的核心成分之一。第一个实现升级的游戏之一是 1980 年的*吃豆人*。在游戏中，你可以吃掉能量豆，使 Pac-Man 获得暂时的无敌状态。另一个经典例子是*马里奥兄弟*中的蘑菇，它使马里奥变得更高大更强壮。

我们将要构建的升级成分将与经典类似，但具有更多的粒度。我们不仅可以使一个实体的单一能力得到升级，还可以创建组合，一次提供多个好处。例如，我们可以有一个名为“**保护者**”的升级，它增加了面向前方的护盾的耐久性，并增加了主要武器的强度。

因此，在本章中，我们将实现一个可扩展和可配置的升级机制，不仅对我们程序员来说如此，而且对于可能被分配创建和调整独特升级成分的设计师来说也是如此。我们将通过结合访问者模式和独特的 Unity API 功能 ScriptableObjects 来实现这一点。

本章将涵盖以下主题：

+   访问者模式背后的基本原理

+   为赛车游戏实现一个升级机制

为了简化代码示例并提高可读性，本节包含了一些简化的代码示例。如果你希望在真实游戏项目的上下文中查看完整的实现，请打开 GitHub 项目中的`FPP`文件夹，该链接可以在*技术要求*部分找到。

# 技术要求

本章是实践性的。你需要对 Unity 和 C#有一个基本的了解。我们将使用以下 Unity 引擎和 C#语言概念：

+   接口

+   ScriptableObjects

如果你对这些概念不熟悉，请在开始本章之前复习它们。本章的代码文件可以在[`github.com/PacktPublishing/Game-Development-Patterns-with-Unity-2021-Second-Edition/tree/main/Assets/Chapters/Chapter10`](https://github.com/PacktPublishing/Game-Development-Patterns-with-Unity-2021-Second-Edition/tree/main/Assets/Chapters/Chapter10)找到[.](https://github.com/PacktPublishing/Game-Development-Patterns-with-Unity-2021-Second-Edition/tree/main/Assets/Chapters/Chapter10)

查看以下视频以查看代码的实际效果：[`bit.ly/3eeknGC`](https://bit.ly/3eeknGC)[.](https://bit.ly/3eeknGC)

我们在这本书的代码示例中经常使用 ScriptableObjects，因为在构建游戏系统和机制时，使非程序员能够轻松配置它们是至关重要的。平衡系统和编写新成分的过程通常属于游戏和关卡设计师的责任。因此，我们使用 ScriptableObjects，因为它提供了一种一致的方式来建立创作管道，以创建和配置游戏中的资产。

# 理解访问者模式

一旦你掌握了它，访问者模式的初衷就很简单；一个*可访问*对象允许一个*访问者*对其结构中的特定元素进行操作。这个过程允许被访问的对象从访问者那里获得新的功能，而无需直接修改。这种描述一开始可能看起来非常抽象，但如果我们将对象想象成一个结构而不是一个封闭的数据和逻辑容器，它就更容易可视化。因此，使用访问者模式，我们可以遍历对象的结构，对其元素进行操作，并扩展其功能，而无需修改它。

另一种想象访问者模式在游戏中发挥作用的方法是想象我们的游戏中的自行车与一个能量提升器相撞。就像电子电流一样，能量提升器流经车辆的内部结构。标记为可访问的组件会被能量提升器访问，并添加新的功能，但没有任何修改。

在以下图中，我们可以可视化这些原则：

![图片](img/84326d4d-0653-464b-9d6c-eb5f5d361c0f.png)

图 10.1 - 访问者模式的 UML 图

在这个模式中，我们需要熟悉两个关键参与者：

+   **IVisitor**是希望成为访问者的类必须实现的接口。访问者类将必须为每个可访问元素实现一个访问者方法。

+   **IVisitable**是希望成为可访问的类必须实现的接口。它包括一个`accept()`方法，为访问者对象提供一个入口点来访问。

在继续之前，我们必须声明，我们将在接下来的部分中审查的代码示例违反了访问者模式的一个潜在规则。在这个例子中，访问者对象更改了一些被访问对象的属性。然而，这种违规是否破坏了模式的完整性并使其原始设计意图无效，是一个值得讨论的问题。

尽管如此，在本章中，我们更关注访问者模式如何使我们能够遍历组成可访问对象结构的元素。在我们的用例中，这个结构代表了我们自行车的核心元素，包括引擎、护盾和主要武器。

访问者被认为是最难理解的模式之一。所以如果你一开始没有掌握其核心概念，请不要感到难过。我相信它之所以难以学习，是因为它是最难解释的模式之一。

## 访问者模式的优缺点

我已经列出了一份使用此模式的好处和缺点的简短列表。

以下是一些益处：

+   **开放/封闭**：你可以添加新的行为，这些行为可以与不同类的对象一起工作，而无需直接修改它们。这种方法遵循面向对象编程原则中的开放/封闭原则，即实体应该对扩展开放，但对修改封闭。

+   **单一职责**：访问者模式可以在某种意义上遵守单一职责原则，即你可以有一个对象（可访问者）来存储数据，另一个对象（访问者）负责引入特定的行为。

以下是一些潜在的缺点：

+   **可访问性**：访问者可能缺乏访问它们访问的特定元素的私有字段和方法所需的必要权限。因此，如果我们不使用该模式，我们可能需要在我们的类中暴露比通常更多的公共属性。

+   **复杂性**：我们可以争论，访问者模式在结构上比像单例、状态和对象池这样的直接模式更复杂。因此，它可能会给代码库带来其他程序员可能觉得令人困惑的复杂性，如果他们不熟悉该模式的结构和复杂性。

访问者使用了一种名为**双重分派**的软件工程概念来设计其基本架构。这个概念最简单的定义是，它是一种机制，根据运行时调用中涉及的两个对象类型将方法调用委托给不同的具体方法。*完全理解这个概念并不是理解本章中展示的模式的示例所必需的。*

# 设计增强效果机制

如本章开头所述，增强效果是视频游戏的一个基本元素。它是我们游戏的核心成分。但首先，我们将回顾我们机制的一些关键规范：

+   **粒度**：我们的增强实体将能够同时增强多个属性。例如，我们可能有一个增强效果，它增加攻击能力，如主要武器的射程，同时修复前向护盾。

+   **时间**：增强效果不是时间相关的，因此它们不会在一段时间后过期。下一个增强效果的益处会叠加到前一个效果之上，直到达到增强属性的极限设置。

注意，这些规范仅限于以下代码示例，但并非最终版本。我们可以轻松地将增强效果的益处设置为临时性的，或者通过修改下一节中展示的代码示例的微小变化来改变整个机制的整体设计。

我们的水平设计师将在赛道上的战略点上定位增强效果；它们将具有在高速下易于识别的 3D 形状。玩家必须与增强效果发生碰撞以激活其包含的能力和益处。

我们将结合使用访问者模式和 ScriptableObjects 来实现这个游戏机制，以便设计师可以编写新的提升变体，而无需编写任何代码。

在视频游戏中，物品和提升之间的核心区别是玩家可以收集物品、存储它们并在需要时使用它们的益处。但相比之下，提升在玩家接触后立即生效。

# 实现提升机制

在本节中，我们将编写必要的骨架代码来实现具有访问者模式的提升系统。我们的目标是到本节结束时有一个有效的概念证明。

## 实现提升系统

让我们看看实现步骤：

1.  我们将首先编写模式的核心元素，即 `Visitor` 接口：

```cs
namespace Pattern.Visitor
{
    public interface IVisitor
    { 
        void Visit(BikeShield bikeShield);
        void Visit(BikeEngine bikeEngine);
        void Visit(BikeWeapon bikeWeapon);
    }
}
```

1.  接下来，我们将编写一个接口，每个可访问元素都必须实现：

```cs
namespace Pattern.Visitor
{
    public interface IBikeElement
    { 
        void Accept(IVisitor visitor);
    }
}
```

1.  现在我们已经有了主要接口，让我们实现使提升机制工作的主要类；由于其长度，我们将分两部分进行回顾：

```cs
using UnityEngine;

namespace Pattern.Visitor
{
    [CreateAssetMenu(fileName = "PowerUp", menuName = "PowerUp")]
    public class PowerUp : ScriptableObject, IVisitor
    {
        public string powerupName;
        public GameObject powerupPrefab;
        public string powerupDescription;

        [Tooltip("Fully heal shield")]
        public bool healShield;

        [Range(0.0f, 50f)]
        [Tooltip(
            "Boost turbo settings up to increments of 50/mph")]
        public float turboBoost;

        [Range(0.0f, 25)]
        [Tooltip(
            "Boost weapon range in increments of up to 25 units")]
        public int weaponRange;

        [Range(0.0f, 50f)]
        [Tooltip(
            "Boost weapon strength in increments of up to 50%")]
        public float weaponStrength;

```

首先要注意的是，这个类是一个带有 `CreateAssetMenu` 属性的 `ScriptableObject` 类。因此，我们将能够从资产菜单中创建新的提升资产。然后，我们可以在引擎的检查器中配置每个新提升的参数。但另一个重要的细节是，这个类实现了 `IVisitor` 接口，我们将在下一部分进行回顾：

```cs
        public void Visit(BikeShield bikeShield) 
        {
            if (healShield) 
                bikeShield.health = 100.0f;
        } 

        public void Visit(BikeWeapon bikeWeapon) 
        {
            int range = bikeWeapon.range += weaponRange;

            if (range >= bikeWeapon.maxRange)
                bikeWeapon.range = bikeWeapon.maxRange;
            else
                bikeWeapon.range = range;

            float strength = 
                bikeWeapon.strength += 
                    Mathf.Round(
                        bikeWeapon.strength 
                        * weaponStrength / 100);

            if (strength >= bikeWeapon.maxStrength)
                bikeWeapon.strength = bikeWeapon.maxStrength;
            else
                bikeWeapon.strength = strength;
        }

        public void Visit(BikeEngine bikeEngine)
        {
            float boost = bikeEngine.turboBoost += turboBoost;

            if (boost < 0.0f)
                bikeEngine.turboBoost = 0.0f;

            if (boost >= bikeEngine.maxTurboBoost)
                bikeEngine.turboBoost = bikeEngine.maxTurboBoost;
        }
    }
}
```

如我们所见，对于每个可访问元素，我们都有一个与之关联的独特方法；在它们内部，我们实现了在访问特定元素时要执行的操作。

在我们的情况下，我们正在更改访问对象的具体属性，同时考虑到定义的最大值。因此，我们将提升在访问自行车结构特定可访问元素时预期的行为封装在单个 `Visit()` 方法中。

我们需要更改特定值、修改操作和为特定可访问元素添加新行为；我们可以在单个类中完成这些事情。

1.  接下来是 `BikeController` 类，它负责控制构成自行车结构的自行车关键组件：

```cs
using UnityEngine;
using System.Collections.Generic;

namespace Pattern.Visitor
{
    public class BikeController : MonoBehaviour, IBikeElement
    {
        private List<IBikeElement> _bikeElements = 
            new List<IBikeElement>();

        void Start()
        {
            _bikeElements.Add(
                gameObject.AddComponent<BikeShield>());

            _bikeElements.Add(
                gameObject.AddComponent<BikeWeapon>());

            _bikeElements.Add(
                gameObject.AddComponent<BikeEngine>());
        }

        public void Accept(IVisitor visitor)
        {
            foreach (IBikeElement element in _bikeElements)
            {
                element.Accept(visitor);
            }
        }
    }
}
```

注意，该类正在实现 `IBikeElement` 接口的 `Accept()` 方法。当自行车与位于赛道上的提升物品相撞时，将自动调用此方法。通过此方法，提升实体将能够将访客对象传递给 `BikeController`。

控制器将依次将接收到的访客对象转发给其每个可访问元素。可访问元素将根据访客对象实例中的配置更新其属性。因此，这就是提升机制被触发和运行的方式。

1.  现在是时候实现我们各自的访问元素了，从我们的骨骼 `BikeWeapon` 类开始：

```cs
using UnityEngine;

namespace Pattern.Visitor
{
    public class BikeWeapon : MonoBehaviour, IBikeElement
    {
        [Header("Range")]
        public int range = 5; 
        public int maxRange = 25;

        [Header("Strength")]
        public float strength = 25.0f;
        public float maxStrength = 50.0f;

        public void Fire()
        {
            Debug.Log("Weapon fired!");
        }

        public void Accept(IVisitor visitor)
        {
            visitor.Visit(this);
        }

        void OnGUI() 
        {
            GUI.color = Color.green;

            GUI.Label(
                new Rect(125, 40, 200, 20), 
                "Weapon Range: " + range);

            GUI.Label(
                new Rect(125, 60, 200, 20), 
                "Weapon Strength: " + strength);
        }
    }
}
```

1.  以下是`BikeEngine`类；请注意，在完整的实现中，这个类将负责模拟一些引擎的行为，包括激活涡轮增压，管理冷却系统，以及控制速度：

```cs
using UnityEngine;

namespace Pattern.Visitor
{
    public class BikeEngine : MonoBehaviour, IBikeElement
    {
        public float turboBoost = 25.0f; // mph
        public float maxTurboBoost = 200.0f;

        private bool _isTurboOn;
        private float _defaultSpeed = 300.0f; // mph

        public float CurrentSpeed
        {
            get
            {
                if (_isTurboOn) 
                    return _defaultSpeed + turboBoost;
                return _defaultSpeed;
            }
        }

        public void ToggleTurbo()
        {
            _isTurboOn = !_isTurboOn;
        }

        public void Accept(IVisitor visitor)
        {
            visitor.Visit(this);
        }

        void OnGUI() 
        {
            GUI.color = Color.green;

            GUI.Label(
                new Rect(125, 20, 200, 20), 
                "Turbo Boost: " + turboBoost);
        }
    }
}
```

1.  最后，`BikeShield`的名称暗示了其主要功能，其实现如下：

```cs
using UnityEngine;

namespace Pattern.Visitor
{
    public class BikeShield : MonoBehaviour, IBikeElement
    { 
        public float health = 50.0f; // Percentage

        public float Damage(float damage)
        {
            health -= damage;
            return health;
        }

        public void Accept(IVisitor visitor)
        {
            visitor.Visit(this);
        }

        void OnGUI() 
        {
            GUI.color = Color.green;

            GUI.Label(
                new Rect(125, 0, 200, 20), 
                "Shield Health: " + health);
        }
    }
}
```

如我们所见，每个单独的可访问自行车元素类都实现了`Accept()`方法，从而使自己变得可访问。

## 测试`PowerUp`系统实现

要快速测试您自己的 Unity 实例中的实现，您需要遵循以下步骤：

1.  将我们刚刚审查的所有脚本复制到您的 Unity 项目中。

1.  创建一个新的场景。

1.  将一个 GameObject 添加到场景中。

1.  将以下`ClientVisitor`脚本附加到新的 GameObject 上：

```cs
using UnityEngine;

namespace Pattern.Visitor
{
    public class ClientVisitor : MonoBehaviour
    {
        public PowerUp enginePowerUp;
        public PowerUp shieldPowerUp;
        public PowerUp weaponPowerUp;

        private BikeController _bikeController;

        void Start()
        {
            _bikeController = 
                gameObject.
                    AddComponent<BikeController>();
        }

        void OnGUI()
        {
            if (GUILayout.Button("PowerUp Shield"))
                _bikeController.Accept(shieldPowerUp);

            if (GUILayout.Button("PowerUp Engine"))
                _bikeController.Accept(enginePowerUp);

            if (GUILayout.Button("PowerUp Weapon"))
                _bikeController.Accept(weaponPowerUp);
        }
    }
}
```

1.  转到**资产/创建**菜单选项并创建三个`PowerUp`资产。

1.  使用您期望的参数配置并命名新的`PowerUp`资产。

1.  在检查器中为新`PowerUp`资产命名并调整其参数。

1.  将新的`PowerUp`资产添加到`ClientVisitor`组件的公共属性中。

一旦开始场景，您应该在屏幕上看到以下 GUI 按钮和调试输出：

![图片](img/c84cf0c5-925b-4d5f-af9f-4cd806b5e0ee.png)

图 10.2 - 代码示例运行时的截图

但你可能想知道，我该如何创建一个实际的拾取实体，我可以在赛道上生成它？

做这件事的一个快速方法是简单地创建一个类似于以下的`Pickup`类：

```cs
using System;
using UnityEngine;

public class Pickup : MonoBehaviour
{
    public PowerUp powerup;

    private void OnTriggerEnter(Collider other)
    {
        if (other.GetComponent<BikeController>())
        {
            other.GetComponent<BikeController>().Accept(powerup);
            Destroy(gameObject);
        }
    }
}
```

通过将此脚本附加到一个配置为触发器的碰撞器组件的 GameObject 上，我们可以检测具有`BikeController`组件的实体何时进入触发器。然后我们只需调用它的`Accept()`方法并传递`PowerUp`实例。设置触发器超出了本章的范围，但我建议查看 Git 仓库中的 FPP 项目，以了解我们如何在游戏的可玩原型中设置它。

## 检查`PowerUp`系统实现

我们能够结合访问者模式的结构和 ScriptableObjects 的 API 功能，创建一个允许我们项目中的任何人为其编写和配置新的`PowerUp`而无需编写任何代码的机制。

如果我们需要调整`PowerUp`对我们车辆各个组件的影响，我们可以通过修改单个类来实现。因此，总的来说，我们在保持代码易于维护的同时，实现了一定程度的可扩展性。

本书中对标准软件设计模式的实现是实验性的，并且以创新的方式进行了调整。我们将它们调整为利用 Unity API 功能，并调整以适应游戏开发用例。因此，我们不应将示例视为学术或标准化的参考，而应将其视为解释。

# 摘要

在本书的这一章节中，我们使用访问者模式作为基础，为我们的游戏构建了一个增强机制。我们还建立了一个工作流程来创建和配置增强功能。因此，我们结合了技术和创造性思维，这是游戏开发的核心。

在下一章中，我们将设计和实现敌方无人机的攻击机动。我们将使用策略模式作为我们系统的基石。
