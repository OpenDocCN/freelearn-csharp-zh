# 命令

我必须承认，命令模式可能一开始很难理解。我知道我花了很长时间才掌握它。即使它的名字表明了其核心目的，其实际应用一开始并不明显。但一旦你开始尝试并理解其复杂性，它就可以成为设计特定类型系统（如用户界面）时应用的一个坚固的模式。它的主要目的是提供一个封装所需数据的方法，以便在特定时刻执行操作或触发事件。

本章将涵盖以下主题：

+   命令模式背后的基本原理

+   实现一个我们可以用其控制多个设备的通用控制器

# 技术要求

下一章是实践性的，因此你需要对 Unity 和 C#有基本的了解。

我们将使用以下特定的 Unity 引擎和 C#语言概念：

+   构造函数

如果你对这个概念不熟悉，请在继续之前先复习一下。

本章的代码文件可以在 GitHub 上找到：

[`github.com/PacktPublishing/Hands-On-Game-Development-Patterns-with-Unity-2018`](https://github.com/PacktPublishing/Hands-On-Game-Development-Patterns-with-Unity-2018)

查看以下视频，以查看代码的实际运行情况：

[`bit.ly/2Our6OF`](http://bit.ly/2Our6OF)

# 命令模式的概述

命令模式是一种解决方案，它使得在对象上集中调用特定命令的过程成为可能。在思考命令模式时，我常常会联想到一个*通用控制器*。在互联网和智能手机出现之前，大多数客厅里都有多个设备，每个设备都有其特定的功能。你有一个音响来播放音乐，一个电视来观看节目，一个 VHS 来播放磁带，等等，但每个系统都有一个特定的遥控器与之相关联，因此由于需要管理多种控制器，这常常会导致混乱。

由于这个原因，发明了可编程的*通用控制器*，它解决了这个问题，并允许你从单个遥控器控制多个设备。这种方法之所以有效，是因为*通用控制器*有一组标准的按钮，你可以将其与命令和设备相关联。

从某种意义上说，命令模式与*通用控制器*的概念非常相似，因为它允许你将特定的命令与可以处理请求的对象链接和调用。

让我们回顾以下图示，这是命令模式典型实现的示例：

![](img/8cad29e3-cc4d-44ca-a948-c53825d38454.png)

通过查看图表来学习命令模式并不是正确的方法，但它确实帮助我们隔离了参与此模式的根本类：

+   一个`Invoker`是一个知道如何执行命令并且可以记录已执行命令的对象。

+   `Receiver`是一种可以接收命令并执行它们的对象类型。

+   `CommandBase`通常是具体命令类的接口或抽象类。它是这种模式的主要抽象层。

这些核心类封装了在特定时刻执行命令所需的所有信息。这将在我们实现用例时变得更加明显。

# 优点和缺点

与策略和状态等模式类似，程序员似乎避免使用命令模式的主要原因是其*冗长性*。

这些是命令模式的主要优点：

+   **顺序和时间**: 命令模式为你提供了在特定顺序或时间范围内执行命令的灵活性

+   **撤销/重做**: 命令模式通常用于实现簿记功能，允许以特定顺序回滚命令

+   **可扩展性**: 行为模式的一个典型优点是它们给你提供了添加行为的能力，而无需对主要类进行大量更改

这些是命令模式的缺点：

+   **冗长性**: 这种模式的一个常见缺点是它会使你的代码非常冗长，并且向你的代码库中添加更多类

# 用例示例

作为用例，我们将实际实现一个通用控制器，它将允许我们控制多个设备，包括电视和收音机。我们将使用通用控制器概念作为示例的主要原因是因为这将更容易让我们学习命令模式的复杂性，我们将通过实现一个直接与在特定对象上管理命令调用相关的系统来完成这一点。

# 代码示例

通过特定的用例实现命令模式是掌握它的最佳方式。我们将编写的示例非常适合命令模式，因为它全部关于向特定接收器发送命令：

1.  对于我们的第一个类，我们需要以抽象类的形式声明`RemoteControlDevice`类型：

```cs
abstract class RemoteControlDevice
{
    public abstract void TurnOn();
    public abstract void TurnOff();
}
```

1.  接下来是`Command`类，这是我们这种模式的核心类型：

```cs
abstract class Command
{
    protected RemoteControlDevice m_Receiver;

    public Command(RemoteControlDevice receiver)
    {
        m_Receiver = receiver;
    }

    public abstract void Execute();
}
```

正如我们所见，它的主要职责是分配一个`Receiver`对象并执行`Execute()`命令。

1.  现在，让我们实现具体的命令类，每个类都有其独特的职责。首先，让我们实现`TurnOnCommand`，它用于打开我们的设备（接收器）：

```cs
class TurnOnCommand : Command
{
    public TurnOnCommand(RemoteControlDevice receiver) : base(receiver)
    {
    }

    public override void Execute()
    {
        m_Receiver.TurnOn();
    }
}
```

1.  接下来是`TurnOffCommand`，它当然会关闭我们的设备（接收器）：

```cs
class TurnOffCommand : Command
{
    public TurnOffCommand(RemoteControlDevice receiver) : base(receiver)
    {
    }

    public override void Execute()
    {
        m_Receiver.TurnOff();
    }
}
```

1.  还有我们的`KillSwitchCommand`，它是独特的：

```cs
class KillSwitchCommand : Command
{
    private RemoteControlDevice[] m_Devices;
    private static RemoteControlDevice receiver;

    public KillSwitchCommand(RemoteControlDevice[] devices) : base(receiver)
    {
        m_Devices = devices;
    }

    public override void Execute()
    {
        foreach (RemoteControlDevice device in m_Devices)
        {
            device.TurnOff();
        }
    }
}
```

正如我们所见，`KillSwitchCommand`不仅仅是在`Receiver`对象上调用`Execute()`函数，而是遍历一系列设备并在每个设备上调用`TurnOff()`函数。这意味着我们正在批量执行特定命令。

1.  现在，我们需要实现接收特定命令以执行特定命令的设备。我们的第一个接收器是`Television`：

```cs
using UnityEngine;

class TelevisionReceiver : RemoteControlDevice
{
    public override void TurnOn()
    {
        Debug.Log("TV turned on.");
    }

    public override void TurnOff()
    {
        Debug.Log("TV turned off.");
    }
}
```

1.  我们下一个接收器是`Radio`：

```cs
using UnityEngine;

class RadioReceiver : RemoteControlDevice
{
    public override void TurnOn()
    {
        Debug.Log("Radio is turned on.");
    }

    public override void TurnOff()
    {
        Debug.Log("Radio is turned off.");
    }
}
```

如我们所见，两个接收者都实现了`TurnOn()`和`TurnOff()`函数。因此，它们封装了各自独特行为的细节。

1.  此外，让我们实现命令模式（`Command pattern`）的一个关键角色——`Invoker`：

```cs
class Invoker
{
    private Command m_Command;

    public void SetCommand(Command command)
    {
        m_Command = command;
    }

    public void ExecuteCommand()
    {
        m_Command.Execute();
    }
}
```

这个`Invoker`的例子很简单，但可以很容易地扩展以记录通过它执行的命令。

1.  最后，我们有我们的`Client`类：

```cs
using UnityEngine;

public class Client : MonoBehaviour
{
    private RemoteControlDevice m_RadioReceiver;
    private RemoteControlDevice m_TelevisionReceiver;
    private RemoteControlDevice[] m_Devices = new RemoteControlDevice[2];

    void Start()
    {
        m_RadioReceiver = new RadioReceiver();
        m_TelevisionReceiver = new TelevisionReceiver();

        m_Devices[0] = m_RadioReceiver;
        m_Devices[1] = m_TelevisionReceiver;
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.O))
        {
            Command commandTV = new TurnOnCommand(m_Devices[0]);
            Command commandRadio = new TurnOnCommand(m_Devices[1]);

            Invoker invoker = new Invoker();

            invoker.SetCommand(commandTV);
            invoker.ExecuteCommand();

            invoker.SetCommand(commandRadio);
            invoker.ExecuteCommand();
        }

        if (Input.GetKeyDown(KeyCode.K))
        {
            Command commandKill = new KillSwitchCommand(m_Devices);
            Invoker invoker = new Invoker();
            invoker.SetCommand(commandKill);
            invoker.ExecuteCommand();
        }
    }
}
```

你会注意到在调用命令时有一个特定的调用顺序：

1.  初始化一个新的命令

1.  将其传递给`Invoker`

1.  `Invoker`执行指定的命令

采用这种方法，我们维护了调用命令和接收命令之间的一个一致通信渠道。

# 摘要

在本章中，我们回顾了命令模式（Command pattern），这是一个独特的模式，一开始往往会让很多程序员感到困惑，因为它的核心用途并不总是显而易见。但一旦正确应用，它确实在实现依赖于在多个组件上按特定顺序执行命令的系统时提供了很多可扩展性。

接下来是观察者模式（Observer pattern），这个模式比命令模式（Command）更容易理解，并且是 C#事件系统的核心。

# 练习题

命令模式通常用于实现大多数文本编辑器中常见的经典撤销/重做功能。在我们的代码示例中，我们实现了支持此功能的基础。因此，作为一个练习题，我建议你集成自己的撤销/重做功能。你可以在`Invoker`和`KillSwitchCommand`类中找到关于实现此功能的最佳方法的线索。

# 进一步阅读

+   *《应用 UML 和模式》，作者：Craig Larman*：[`www.craiglarman.com`](http://www.craiglarman.com)
