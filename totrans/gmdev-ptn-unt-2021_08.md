使用事件总线管理游戏事件

事件总线充当一个中心枢纽，管理一组特定的全局事件，对象可以选择订阅或发布。这是我在工具箱中与事件管理相关最直接的模式。它将分配对象作为订阅者或发布者的过程简化为单行代码。正如你可以想象的那样，当你需要快速得到结果时，这可能会很有益。但像大多数简单解决方案一样，它也有一些缺点和限制，我们将在后面的内容中进一步探讨。

在本章提供的代码示例中，我们将使用事件总线向需要监听比赛整体状态变化的组件广播特定的比赛事件。但重要的是要记住；我提出使用事件总线作为管理全局比赛事件解决方案的原因是其简单性，而不是其可扩展性。因此，它可能不是所有情况下的最佳解决方案，但它是最直接的模式之一，我们将在接下来的章节中看到。

本章将涵盖以下主题：

+   事件总线模式的概述

+   Race Event Bus 的实现

为了简洁和清晰，本章包含简化的代码示例。如果你希望在一个实际的游戏项目中查看该模式的完整实现，请打开 GitHub 项目中的`FPP`文件夹。你可以在*技术要求*部分找到链接。

# 技术要求

我们还将使用以下特定的 Unity 引擎 API 功能：

+   `Static`

+   `UnityEvents`

+   `UnityActions`

如果你对这些概念不熟悉，请查阅第三章，《Unity 编程简明指南》。

本章的代码文件可以在 GitHub 上找到，地址为[`github.com/PacktPublishing/Game-Development-Patterns-with-Unity-2021-Second-Edition/tree/main/Assets/Chapters/Chapter06`](https://github.com/PacktPublishing/Game-Development-Patterns-with-Unity-2021-Second-Edition/tree/main/Assets/Chapters/Chapter06)。

查看以下视频，以查看代码的实际应用：

[`bit.ly/2U7wrCM`](https://bit.ly/2U7wrCM)。

如果你发现自己难以理解委托背后的机制，请记住，它们在 C/C++中类似于函数指针。简单来说，委托指向一个方法。委托通常用于实现事件处理程序和回调。动作是一种可以与具有 void 返回类型的方法一起使用的委托类型。

# 理解事件总线模式

当一个对象（发布者）引发事件时，它会发送一个信号，其他对象（订阅者）可以接收。该信号以通知的形式出现，可以指示动作的发生。在事件系统中，一个对象广播事件。只有订阅了该事件的那些对象才会收到通知并选择如何处理它。因此，我们可以将其想象为一种突然爆发的无线电信号，只有那些将天线调谐到特定频率的人才能检测到。

事件总线模式是消息系统和发布-订阅模式的一个近亲，后者是事件总线所做事情的更准确名称。这个模式标题中的关键字是术语*总线*。在简单的计算机术语中，总线是组件之间的连接。在本章的上下文中，这些组件将是可以作为事件发布者或监听者的对象。

因此，事件总线是通过使用发布-订阅模型通过事件连接对象的一种方式。使用观察者模式和本地 C#事件等模式也可以实现类似的功能。然而，这些替代方案存在一些缺点。例如，在观察者模式的一个典型实现中，可能会发生紧密耦合，因为观察者（监听器）和主题（发布者）可能会相互依赖并意识到对方的存在。

但事件总线，至少在我们将在 Unity 中实现的方式中，抽象并简化了发布者和订阅者之间的关系，因此它们彼此完全不知情。另一个优点是它减少了将发布者或订阅者角色分配给单行代码的过程。因此，事件总线是一个值得学习和拥有的宝贵模式。您可以从以下图中看到，它确实在发布者和订阅者之间充当中间人：

![](img/272dfd72-4bd9-4a96-a736-c192239f3c9a.png)

图 6.1 – 事件总线模式的图示

如您从图中可以看出，有三个主要成分：

+   **发布者**：这些对象可以向订阅者发布事件总线声明的特定类型的事件。

+   **事件总线**：此对象负责协调发布者和订阅者之间的事件传输。

+   **订阅者**：这些对象通过事件总线将自己注册为特定事件的订阅者。

## 事件总线模式的优缺点

事件总线模式的**优点**如下：

+   **解耦**：使用事件系统的主要好处是它解耦了您的对象。对象可以通过事件进行通信，而不是直接引用彼此。

+   **简单性**：事件总线通过抽象发布或订阅事件机制，为客户端提供简单性。

事件总线模式的**缺点**如下：

+   **性能**：在任何一个事件系统的底层，都有一个管理对象间消息传递的低级机制。因此，使用事件系统可能会带来轻微的性能开销，但根据您的目标平台，这可能是微不足道的。

+   **全局**：在本章中，我们使用静态方法和属性实现事件总线，以便更容易从代码的任何地方访问它。使用全局可访问的变量和状态总会有风险，因为它们可能会使调试和单元测试变得更加困难。然而，这是一个非常具体的问题，而不是绝对的。

UnityEvents 实际上可以接受多达四个泛型类型参数。这使得将特定事件的数据传递给订阅者成为可能。请参阅以下 Unity API 文档部分以获取更多详细信息：[`docs.unity3d.com/2021.2/Documentation/ScriptReference/Events.UnityEvent.html`](https://docs.unity3d.com/2021.2/Documentation/ScriptReference/Events.UnityEvent.html)[.](https://docs.unity3d.com/ScriptReference/Events.UnityEvent.html)

## 何时使用事件总线

我过去曾使用事件总线来完成以下任务：

+   **快速原型设计**：我在快速原型设计新的游戏机制或功能时经常使用事件总线模式。使用这个模式，我可以轻松地让组件通过事件触发彼此的行为，同时保持它们解耦。这个模式允许我们通过一行代码添加和删除对象作为订阅者或发布者，这在你想快速轻松地原型设计某物时总是很有帮助。

+   **生产代码**：如果我没有找到合理的理由来实现更复杂的管理游戏事件的方法，我会在生产代码中使用事件总线。如果你不需要处理复杂的事件类型或结构，这个模式可以很好地完成任务。

我会避免使用本章中展示的类似的全局可访问事件总线来管理没有“全局范围”的事件。例如，如果我有一个在 HUD 中显示伤害警报的 UI 组件，当玩家与障碍物碰撞时，通过事件总线发布碰撞事件将是不高效的，因为这仅仅是自行车与赛道上的物体之间的局部交互。相反，我会使用类似观察者模式的东西，因为在那种情况下，我只需要一个 UI 组件来观察对象状态的一个特定变化，在这种情况下，是自行车。作为一个经验法则，每次你必须实现一个使用事件系统的系统时，都要通过已知模式的列表，选择一个适合你用例的模式，但不要总是退回到最简单的一个，在我看来，那就是事件总线。

## 管理全球赛车活动

我们正在开发的项目是一款赛车游戏，大多数比赛都分为几个阶段。以下是一些典型的赛车阶段简短列表：

+   **倒计时**：在这个阶段，自行车停在起跑线后，同时倒计时计时器正在运行。

+   **赛事开始**：一旦时钟达到零，绿灯信号被点亮，自行车开始在赛道上前进。

+   **赛事结束**：当玩家通过终点线的那一刻，赛事就结束了。

在赛事的开始和结束之间，可能会触发某些事件，这些事件可能会改变赛事的当前状态：

+   **赛事暂停**：玩家可以在比赛过程中暂停游戏。

+   **赛事退出**：玩家可以在任何时候退出赛事。

+   **赛事停止**：如果玩家参与致命碰撞，赛事可能会突然停止。

因此，我们希望广播一个通知，以表示赛事每个阶段的发生以及任何其他在赛事过程中发生的重要事件，这些事件将改变赛事的整体状态。这种方法将允许我们触发特定组件的特定行为，这些组件需要根据赛事的当前上下文以特定方式行为。

例如，以下是一些将根据赛事的特定状态进行更新的组件：

+   **HUD**：赛事状态指示器将根据赛事的上下文改变**HUD**（**抬头显示**）。

+   **赛事计时器**：赛事计时器仅在赛事开始时启动，并在玩家通过终点线时停止。

+   **自行车控制器**：一旦赛事开始，自行车控制器将释放自行车的刹车，这种机制防止玩家在绿灯亮起之前冲上赛道。

+   **输入记录器**：在赛事开始时，输入回放系统将开始记录玩家的输入，以便稍后回放。

所有这些组件在赛事的特定阶段都有特定的行为需要触发。因此，我们将使用事件总线来实现这些全局赛事事件。

# 实现赛事事件总线

我们将分两步实现赛事事件总线：

1.  首先，我们需要暴露我们支持的具体赛事事件类型，我们将使用以下枚举来完成：

```cs
namespace Chapter.EventBus
{
    public enum RaceEventType
    {
        COUNTDOWN, START, RESTART, PAUSE, STOP, FINISH, QUIT
    }
}
```

重要的是要注意，前面的枚举值代表从开始到结束的赛事各个阶段的具体事件。因此，我们限制自己只处理具有全局作用域的事件。

1.  下一个部分是模式的核心理念，实际的赛事事件总线类，我们将称之为`RaceEventBus`，以使我们的类命名约定更具领域特定性：

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

        public static void Subscribe
            (RaceEventType eventType, UnityAction listener) {

            UnityEvent thisEvent;

            if (Events.TryGetValue(eventType, out thisEvent)) {
                thisEvent.AddListener(listener);
            }
            else {
                thisEvent = new UnityEvent();
                thisEvent.AddListener(listener);
                Events.Add(eventType, thisEvent);
            }
        }

        public static void Unsubscribe
            (RaceEventType type, UnityAction listener) {

            UnityEvent thisEvent;

            if (Events.TryGetValue(type, out thisEvent)) {
                thisEvent.RemoveListener(listener);
            }
        }

        public static void Publish(RaceEventType type) {

            UnityEvent thisEvent;

            if (Events.TryGetValue(type, out thisEvent)) {
                thisEvent.Invoke();
            }
        }
    }
}
```

我们类中的关键成分是`Events`字典。它充当一个账本，在其中我们维护事件类型和订阅者之间关系列表。通过将其保持为`private`和`readonly`，我们确保它不能被另一个对象直接覆盖。

因此，客户端必须调用`Subscribe()`公共静态方法来将自己添加为特定事件类型的订阅者。`Subscribe()`方法接受两个参数；第一个是赛事事件类型，第二个是回调函数。因为`UnityAction`是一个委托类型，它为我们提供了一种将方法作为参数传递的方式。

因此，当客户端对象调用`Publish()`方法时，特定赛车事件类型的所有订阅者的注册回调方法将同时被调用。

`Unsubscribe()`方法很容易理解，因为它允许对象将自己从特定事件类型的订阅者中移除。因此，当对象发布事件时，事件总线不会调用它们的回调方法。

如果这仍然看起来很抽象，我们将在下一节实现客户端类，并看到我们如何使用事件总线以正确的顺序在特定时刻触发对象的行为。

## 测试赛车事件总线

现在我们已经设置了模式的核心元素，我们可以编写一些代码来测试我们的`RaceEventBus`类。为了简洁起见，我已经从每个客户端类中删除了所有行为代码，以专注于模式的使用：

1.  首先，我们将编写一个倒计时计时器，它订阅了`COUNTDOWN`赛车事件类型。一旦发布`COUNTDOWN`事件，它将触发一个 3 秒的倒计时，直到比赛开始。当计数达到结束时，它将发布`START`事件，以表示比赛的开始：

```cs
using UnityEngine;
using System.Collections;

namespace Chapter.EventBus
{
    public class CountdownTimer : MonoBehaviour
    {
        private float _currentTime;
        private float duration = 3.0f;

        void OnEnable() {
            RaceEventBus.Subscribe(
                RaceEventType.COUNTDOWN, StartTimer);
        }

        void OnDisable() {
            RaceEventBus.Unsubscribe(
                RaceEventType.COUNTDOWN, StartTimer);
        }

        private void StartTimer() {
            StartCoroutine(Countdown());
        }

        private IEnumerator Countdown() {
            _currentTime = duration;

            while (_currentTime > 0) {
                yield return new WaitForSeconds(1f);
                _currentTime--;
            }

            RaceEventBus.Publish(RaceEventType.START);
        }

        void OnGUI() {
            GUI.color = Color.blue;
            GUI.Label(
                new Rect(125, 0, 100, 20), 
                "COUNTDOWN: " + _currentTime);
        }
    }
}
```

上述类中最关键的代码行如下：

```cs
void OnEnable() {
    RaceEventBus.Subscribe(
        RaceEventType.COUNTDOWN, StartTimer);
}

void OnDisable() {
    RaceEventBus.Unsubscribe(
        RaceEventType.COUNTDOWN, StartTimer);
}
```

每当`CountdownTimer`对象被启用时，就会调用`Subscribe()`方法。而当它被禁用时，它会自己取消订阅。我们这样做是为了确保对象在活动时正在监听事件，或者在禁用或销毁时不会被`RaceEventBus`调用。

`Subscribe()`方法接受两个参数——事件类型和回调函数。这意味着每当发布`COUNTDOWN`事件时，`RaceEventBus`都会调用`CountdownTimer`的`StartTimer()`方法。

1.  接下来，我们将实现`BikeController`类的骨架来测试`START`和`STOP`事件：

```cs
using UnityEngine;

namespace Chapter.EventBus 
{
    public class BikeController : MonoBehaviour 
    {
        private string _status;

        void OnEnable() {
             RaceEventBus.Subscribe(
                 RaceEventType.START, StartBike);

             RaceEventBus.Subscribe(
                 RaceEventType.STOP, StopBike);
        }

        void OnDisable() {
             RaceEventBus.Unsubscribe(
                 RaceEventType.START, StartBike);

             RaceEventBus.Unsubscribe(
                 RaceEventType.STOP, StopBike);
        }

        private void StartBike() {
             _status = "Started";
        }

        private void StopBike() {
             _status = "Stopped";
        }

        void OnGUI() {
             GUI.color = Color.green;
             GUI.Label(
                 new Rect(10, 60, 200, 20), 
                 "BIKE STATUS: " + _status);
         }
     }
}
```

1.  最后，我们将编写一个`HUDController`类。这个类除了显示一个按钮来停止比赛之外，没有做太多的事情：

```cs
using UnityEngine;

namespace Chapter.EventBus
{
    public class HUDController : MonoBehaviour
    {
        private bool _isDisplayOn;

        void OnEnable() {
            RaceEventBus.Subscribe(
                RaceEventType.START, DisplayHUD);
        }

        void OnDisable() {
            RaceEventBus.Unsubscribe(
                RaceEventType.START, DisplayHUD);
        }

        private void DisplayHUD() {
            _isDisplayOn = true;
        }

        void OnGUI() {
            if (_isDisplayOn)
            {
                if (GUILayout.Button("Stop Race"))
                {
                    _isDisplayOn = false;
                    RaceEventBus.Publish(RaceEventType.STOP);
                }
            }
        }
    }
}
```

1.  为了测试事件序列，我们需要将以下客户端类附加到一个空的 Unity 场景中的 GameObject 上。有了这个，我们就可以触发倒计时计时器：

```cs
using UnityEngine;

namespace Chapter.EventBus
{
    public class ClientEventBus : MonoBehaviour
    {
        private bool _isButtonEnabled;

        void Start()
        {
            gameObject.AddComponent<HUDController>();
            gameObject.AddComponent<CountdownTimer>();
            gameObject.AddComponent<BikeController>();

            _isButtonEnabled = true;
        }

        void OnEnable()
        {
            RaceEventBus.Subscribe(
                RaceEventType.STOP, Restart);
        }

        void OnDisable()
        {
            RaceEventBus.Unsubscribe(
                RaceEventType.STOP, Restart);
        }

        private void Restart()
        {
            _isButtonEnabled = true;
        }

        void OnGUI()
        {
            if (_isButtonEnabled)
            {
                if (GUILayout.Button("Start Countdown"))
                {
                    _isButtonEnabled = false;
                    RaceEventBus.Publish(RaceEventType.COUNTDOWN);
                }
            }
        }
    }
}
```

前面的示例可能看起来过于简化，但其目的是展示我们如何通过事件触发特定顺序中单个对象的行为，同时保持它们解耦。没有任何对象之间直接通信。

它们唯一的共同参考点是`RaceEventBus`。因此，我们可以轻松地添加更多的订阅者和发布者；例如，我们可以让一个`TrackController`对象监听`RESTART`事件，以知道何时重置赛道。因此，我们现在可以使用事件来按顺序触发核心组件的行为，每个事件都代表比赛的特定阶段。

动作是一个委托，它指向一个接受一个或多个参数但不返回任何值的方法。例如，你应该在委托指向返回 void 的方法时使用 Action。UnityAction 的行为类似于原生 C#中的 Actions，但 UnityAction 是为了与 UnityEvents 一起使用而设计的。

## 回顾事件总线实现

通过使用事件总线，我们可以在保持核心组件解耦的同时触发行为。对我们来说，添加或删除作为订阅者或发布者的对象非常简单。我们还定义了一个特定的全局事件列表，代表比赛的每个阶段。因此，我们现在可以开始对核心组件的行为进行排序和触发，从比赛开始到结束，以及任何中间阶段。

在下一节中，我们将回顾一些替代事件总线的方法，每个解决方案都提供了一种可能根据上下文更好的不同方法。

# 回顾一些替代解决方案

事件系统和模式是一个广泛的话题，我们无法在这本书中深入探讨这个话题。因此，我们准备了一份简短的列表，供你在实现事件系统或机制时考虑，但请记住，还有更多内容，我们鼓励你作为读者在本书有限的范围内继续探索这个话题：

+   **观察者**：这是一个老式但实用的模式，其中一个对象（主题）维护一个对象（观察者）列表，并在内部状态发生变化时通知它们。当你需要在一组实体之间建立一对一的关系时，这是一个值得考虑的模式。

+   **事件队列**：这个模式允许我们将发布者生成的事件存储在队列中，并在方便的时候将它们转发给订阅者。这种方法解耦了发布者和订阅者之间的时间关系。

+   **ScriptableObjects**：在 Unity 中，你可以使用 ScriptableObjects 创建一个事件系统。这种方法的关键优势是它使得创建新的自定义游戏事件变得更加容易。如果你需要构建一个可扩展和可定制的的事件系统，这可能是一个不错的选择。

如果你自己在想为什么这本书没有展示更多高级的事件系统，包括使用 ScriptableObjects 实现的事件系统，答案是这本书的核心目的是向读者介绍设计模式，而不是用复杂性让他们感到不知所措。我们提供了一个核心概念的初步介绍，但鼓励读者在进步的过程中寻找更高级的材料。

# 概述

在本章中，我们回顾了事件总线，这是一种简化在 Unity 中发布和订阅事件过程的简单模式。这是一个在快速原型设计或当你有一个明确的全局事件列表需要管理时非常有用的工具。然而，它有其使用的局限性，因此在决定使用全局可访问的事件总线之前探索其他选项总是明智的。

在下一章中，我们将实现一个系统，使我们能够回放玩家输入。许多赛车游戏都有回放和倒退功能，而通过命令模式，我们将尝试从头开始构建一个这样的系统。
