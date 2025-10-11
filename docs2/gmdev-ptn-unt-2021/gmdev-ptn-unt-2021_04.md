Unity 编程简明指南

本章是一个简短的入门指南，帮助你熟悉高级 C# 语言和 Unity 引擎特性。我不会尝试深入解释这里呈现的任何主题，因为这超出了本书的范围。尽管如此，我至少会介绍一些核心概念，以避免在即将到来的章节中引用它们时的混淆。我鼓励那些对 C# 和 Unity 编程有深入了解的人跳过这一章。但我建议初学者和中级开发者花时间回顾本章内容，以获得我们将用于实现和设计游戏系统的语言和引擎特性的总体概念。

在所有情况下，对 C# 和 Unity 的完全掌握不是理解本书的必要条件，只需对一些关键概念有一般意识和熟悉即可。

在本章中，我们将探讨以下主题：

+   你应该已经知道的内容

+   C# 语言特性

+   Unity 引擎特性

本章展示的代码仅用于教学目的。它未经优化，不应用于生产代码。

# 你应该已经知道的内容

在本节中，我列举了一些你应该在继续阅读本书更高级部分之前已经熟悉的 C# 语言和 Unity 引擎的核心特性。

**以下是一些 C# 的核心特性**：

+   熟悉类访问修饰符，如 public 和 private

+   基本原始数据类型（int、string、bool、float 和数组）的基本知识

+   对继承和基类与派生类之间关系的概念理解

**以下是一些 Unity 的核心特性**：

+   基本了解如何编写 `MonoBehaviour` 脚本并将其作为组件附加到 `GameObject` 上

+   能够从头创建新的 Unity 场景并在编辑器中操作 GameObject

+   熟悉 Unity 的基本事件函数（`Awake`、`Start`、`Update`）及其执行顺序

如果你不太熟悉前面列出的概念，我建议阅读本章 *进一步阅读* 部分中列出的书籍和文档。

# C# 语言特性

事件和委托等语言特性可能对初学者来说过于高级，所以如果你认为自己属于这一类别，请不要担心；你仍然可以享受这本书。只需阅读入门级章节，例如解释单例、状态、外观和适配器等模式的章节。

以下 C# 高级语言特性对于我们在即将到来的章节中实现的一些设计模式的最佳实现是基本的：

+   **静态**：带有`static`关键字的类的方法和成员可以直接通过其名称访问，而无需初始化一个实例。静态方法和成员非常有用，因为它们可以从代码的任何地方轻松访问。以下是一个使用该关键字建立全局可访问事件总线的类示例：

```cs
using UnityEngine.Events;
using System.Collections.Generic;

namespace Chapter.EventBus
{
    public class RaceEventBus
    {
        private static readonly 
            IDictionary<RaceEventType, UnityEvent> 
            Events = new Dictionary<RaceEventType, UnityEvent>();

        public static void Subscribe(
            RaceEventType eventType, UnityAction listener)
        {
            UnityEvent thisEvent;
            if (Events.TryGetValue(eventType, out thisEvent))
            {
                thisEvent.AddListener(listener);
            }
            else
            {
                thisEvent = new UnityEvent();
                thisEvent.AddListener(listener);
                Events.Add(eventType, thisEvent);
            }
        }

        public static void Unsubscribe(
            RaceEventType eventType, UnityAction listener)
        {
            UnityEvent thisEvent;
            if (Events.TryGetValue(eventType, out thisEvent))
            {
                thisEvent.RemoveListener(listener);
            }
        }

        public static void Publish(RaceEventType eventType)
        {
            UnityEvent thisEvent;
            if (Events.TryGetValue(eventType, out thisEvent))
            {
                thisEvent.Invoke();
            }
        }
    }
}
```

+   **事件**：事件允许一个充当发布者的对象发送其他对象可以接收的信号；这些监听特定事件的对象被称为订阅者。当你想要构建事件驱动架构时，事件非常有用。以下是一个发布事件的类示例：

```cs
using UnityEngine;
using System.Collections;

public class CountdownTimer : MonoBehaviour
{
    public delegate void TimerStarted(); 
    public static event TimerStarted OnTimerStarted;

    public delegate void TimerEnded(); 
    public static event TimerEnded OnTimerEnded;

    [SerializeField]
    private float duration = 5.0f;

    void Start()
    {
        StartCoroutine(StartCountdown());
    }

    private IEnumerator StartCountdown()
    {
        if (OnTimerStarted != null) 
            OnTimerStarted();

        while (duration > 0)
        {
            yield return new WaitForSeconds(1f);
            duration--;
        }

        if (OnTimerEnded != null) 
            OnTimerEnded();
    }

    void OnGUI()
    {
        GUI.color = Color.blue;
        GUI.Label(
            new Rect(125, 0, 100, 20), "COUNTDOWN: " + duration)
    }
}
```

+   **委托**：当你理解其底层低级机制时，委托的概念很简单。委托的高级定义是它们持有函数的引用。但这是对委托实际在幕后做什么的非常抽象的定义。它们是函数指针，这意味着它们持有其他函数的内存地址。因此，我们可以将它们想象成一个包含函数位置列表的地址簿。这就是为什么委托可以持有多个它们，并一次性调用它们。以下是一个通过将特定的本地函数分配给发布者的委托来订阅由发布者类触发的事件的类示例：

```cs
using UnityEngine;

public class Buzzer : MonoBehaviour
{
    void OnEnable()
    {
        // Assigning local functions to delegates defined in the 
        // Timer class
        CountdownTimer.OnTimerStarted += PlayStartBuzzer;
        CountdownTimer.OnTimerEnded += PlayEndBuzzer;
    }

    void OnDisable()
    {
        CountdownTimer.OnTimerStarted -= PlayStartBuzzer;
        CountdownTimer.OnTimerEnded -= PlayEndBuzzer;
    }

    void PlayStartBuzzer()
    {
        Debug.Log("[BUZZER] : Play start buzzer!");
    }

    void PlayEndBuzzer()
    {
        Debug.Log("[BUZZER] : Play end buzzer!");
    }
}
```

+   **泛型**：C# 的一个相关特性，允许在类被客户端声明和实例化之前延迟指定类型。当我们说一个类是泛型的时候，它没有定义的对象类型。以下是一个泛型类示例，它可以作为一个模板使用：

```cs
// <T> can be any type.
public class Singleton<T> : MonoBehaviour where T : Component
{
    // ...
}
```

+   **序列化**：序列化是将对象实例转换为二进制或文本形式的过程。这种机制允许我们将对象的状态保存在文件中。以下是一个将对象实例序列化并保存为文件的函数示例：

```cs
private void SerializePlayerData(PlayerData playerData)
{
    // Serializing the PlayerData instance
    BinaryFormatter bf = new BinaryFormatter();
    FileStream file = File.Create(Application.persistentDataPath + 
        "/playerData.dat");
    bf.Serialize(file, playerData);
    file.Close();
}
```

C# 是一门深度很大的编程语言，因此在此书的范围内不可能解释其所有核心特性。本书本节中介绍的都是非常有用的，并将帮助我们实现后续章节中描述的游戏系统和模式。

# Unity 引擎特性

Unity 是一个功能齐全的引擎，包括全面的脚本 API、动画系统以及许多用于游戏开发的附加功能。我们无法在本书中涵盖所有内容，因此我只会列出我们在即将到来的设计模式章节中将要使用的核心 Unity 组件：

+   **预制体**：预制体是由组装好的 GameObject 和组件组成的预制容器。例如，你可以在游戏中为每种类型的车辆创建单独的预制体，并在场景中动态加载它们。预制体允许你将可重用的游戏实体作为构建块进行构建和组织。

+   **Unity 事件和动作**：Unity 有一个本地的事件系统；它与 C#事件系统非常相似，但具有额外的引擎特定功能，例如在检查器中查看和配置它们的能力。

+   **ScriptableObjects**：从`ScriptableObject`基类派生的类可以作为数据容器使用。另一个名为 MonoBehaviour 的本地 Unity 基类用于实现行为。因此，建议使用 MonoBehaviours 来包含你的逻辑，使用 ScriptableObjects 来包含你的数据。`ScriptableObject`的实例可以保存为资产，通常用于创作工作流程。它允许非程序员无需一行代码即可创建特定类型实体的新变体。

以下是一个简单的`ScriptableObject`示例，允许创建新的可配置的`Sword`实例：

```cs
using UnityEngine;

[CreateAssetMenu(fileName = "NewSword", menuName = "Weaponary/Sword")]
public class Sword: ScriptableObject 
{
    public string swordName;
    public string swordPrefab;
}
```

+   **协程**：协程的概念不仅限于 Unity，而且是 Unity API 的一个基本工具。函数的典型行为是从开始到结束执行自己。但协程是一个具有额外等待、计时甚至暂停其执行过程能力的函数。这些附加功能使我们能够实现用传统函数难以实现的行为。协程类似于线程，但提供的是并发而不是并行。以下代码示例展示了使用协程实现倒计时计时器的实现：

```cs
using UnityEngine;
using System.Collections;

public class CountdownTimer : MonoBehaviour
{
    private float _duration = 10.0f;

    IEnumerator Start()
    {
        Debug.Log("Timer Started!");
        yield return StartCoroutine(WaitAndPrint(1.0F));
        Debug.Log("Timer Ended!");
    }

    IEnumerator WaitAndPrint(float waitTime)
    {
        while (Time.time < _duration)
        {
            yield return new WaitForSeconds(waitTime);
            Debug.Log("Seconds: " + Mathf.Round(Time.time));
        }
    }
}
```

如我们所见，Unity 有一些丰富且直观的功能，使我们能够实现系统和组织数据。由于这些核心功能超出了本书的预期范围，我们无法深入探讨每个功能。如果你在继续前进之前觉得需要更多关于引擎的信息，我建议你回顾*进一步阅读*部分下链接的材料。

# 摘要

本章旨在作为入门指南，在进入书籍的实践部分之前，建立一个共享知识库。但不需要掌握前几节中介绍的功能，就可以开始使用 Unity 中的设计模式。我们将在接下来的章节中更详细地回顾其中的一些，这次是在实际的应用场景中。

在下一章中，我们将回顾我们的第一个模式，臭名昭著的单例模式。我们将使用它来实现一个负责初始化游戏的`GameManager`类。

# 进一步阅读

如需更多信息，您可以参考以下材料：

+   *通过 Unity 2020 开发游戏学习 C#* 由 Harrison Ferrone 编写

+   *Unity 用户手册* 由 Unity Technologies 编写：[`docs.unity3d.com/Manual/index.html`](https://docs.unity3d.com/2021.1/Documentation/Manual/UnityManual.html)
