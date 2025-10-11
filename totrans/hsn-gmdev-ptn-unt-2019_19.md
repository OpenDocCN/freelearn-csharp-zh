# 装饰器

装饰器是那些名称完美代表其目的的罕见模式之一。正如其名所示，装饰器模式允许我们装饰一个对象；当然，这是一个非常模糊的解释。所以，一个更具体但简单的核心目的解释是，它为我们提供了一种用新代码装饰旧代码的方法，通过动态地向对象添加功能。

本章将涵盖以下主题：

+   我们将回顾装饰器模式的基本知识

+   我们将构建一个系统，以动态地向步枪添加附件

# 技术要求

本章是实践性的，因此您需要对 Unity 和 C#有一个基本的了解。

我们将使用以下特定的 Unity 引擎和 C#语言概念：

+   构造函数

如果你对这个概念不熟悉，请在开始这一章之前复习它。

本章的代码文件可以在 GitHub 上找到：

[`github.com/PacktPublishing/Hands-On-Game-Development-Patterns-with-Unity-2018`](https://github.com/PacktPublishing/Hands-On-Game-Development-Patterns-with-Unity-2018)

查看以下视频，以查看代码的实际效果：

[`bit.ly/2U0MT6x`](http://bit.ly/2U0MT6x)

# 装饰器模式的基本知识

装饰器模式是你需要通过代码实现以完全理解的模式类型，因此我们将保持理论部分简短。在其最基本的形式中，装饰器模式为我们提供了一种机制，允许我们在运行时向对象添加行为，而无需在过程中更改对象。

正如其名所示，它装饰对象；但它通过在基类的构造函数中链式引用装饰器对象来实现。这听起来可能很抽象，但在实践中它是有效的，因为对象在内存中相互引用的方式。

让我们看一下以下图表，以可视化装饰器模式中类之间的关系结构：

![图片](img/b3d39cef-7c29-4fb1-8c3b-c7fe649bc279.png)

如您所见，有一个接口`IRifle`，它为所有希望成为步枪的类提供了一个实现合同。但，图中最重要的类是`RifleDecorator`。它是允许我们将`WithScope`和`WithStabilizer`装饰器附加到实现`IRifle`的任何步枪对象的类。

但是，在编写代码之前，让我们回顾一下使用装饰器模式时的核心优点和可能的缺点。

# 优点和缺点

装饰器模式享有极高的声誉；它甚至是 Python 语言的一个重要部分。因此，其优点通常超过了其缺点，正如以下列表所示：

以下是一些优点：

+   **替代子类化**：装饰器模式侧重于向对象注入功能，而不是继承和扩展。

+   **可管理的排列组合**：在生产的整个过程中，功能和需求都在不断变化；装饰者模式提供了一种通过分布到自包含组件中来添加功能的方法，而不需要修改核心实现。

+   **运行时动态性**：装饰者模式允许我们在不直接修改对象的情况下在运行时向对象添加功能。

以下是一些缺点：

+   **代码复杂性**：像大多数高级模式一样，实现装饰者模式可能会导致代码库变得更加复杂。

+   **关系复杂性**：如果围绕一个对象有多个装饰者层，跟踪初始化链和装饰者之间的关系可能会变得非常复杂。

特定模式的缺点通常与它们给代码库带来的复杂性和冗余性有关。这主要是在一个由经验水平不同的成员组成的团队中工作时会遇到的问题，因为初级程序员可能没有通过阅读代码来识别特定模式的能力。

# 用例示例

武器是视频游戏的一个基本元素，尤其是在第一人称射击（FPS）类型中。在 FPS 游戏中拥有武器定制功能是一个非常酷且有利可图的特性。通过为新组件，如瞄准镜或消音器，附加到基本步枪上来升级它的能力，是非常吸引人的。但是，作为一个游戏程序员，必须以结构化和模块化的方式在代码中编写所有这些变体和配置，这可能会非常复杂。

但是，使用装饰者模式，我们可以在代码中重现将组件附加到可配置武器的现实生活概念。这正是我们将在下面的代码示例中做的。

适配器和装饰者模式相似，但适配器用于适配对象的接口，而装饰者增强了对象的责任。

# 代码示例

在下面的代码示例中，需要注意的一个特定事项是我们将使用构造函数。通常建议不要在 Unity 中使用它们，因为如果你正在使用`Monobehaviours`或其派生类的`ScritableObjects`，并将它们附加到场景中包含的`GameObjects`，引擎将自动初始化它们。但在这个例子中，我们将打破这个规则；主要是因为装饰者依赖于构造函数的内部机制，正如我们将在下面的代码片段中看到的：

1.  让我们通过编写一个接口来开始我们武器定制系统的实现，这个接口将作为所有派生步枪类型的*实现* *合约*：

```cs
public interface IRifle
{
    float GetAccuracy();
}
```

如我们所见，这是一个简单的接口，它有一个返回步枪精度值的浮点数的函数。

1.  现在我们已经为所有步枪对象定义了一个标准接口，让我们编写一个具体的步枪类，它将代表步枪的基本配置：

```cs
public class BasicRifle : IRifle
{
    private float m_BasicAccurancy = 5.0f;

    public float GetAccuracy()
    {
        return m_BasicAccurancy;
    }
}
```

`BasicRifle`类并没有做任何特别的事情；它只是实现了`IRifle`接口；但我们打算将其用作一个基础对象，我们将用附件来装饰它，从而提高其默认精度。

1.  我们现在需要一个类来负责将装饰器附加到我们的`BasicRifle`对象上：

```cs
abstract public class RifleDecorator : IRifle
{
    protected IRifle m_DecoaratedRifle;

    public RifleDecorator(IRifle rifle)
    {
        m_DecoaratedRifle = rifle;
    }

    public virtual float GetAccuracy()
    {
        return m_DecoaratedRifle.GetAccuracy();
    }
}
```

我们在`RifleDecorator`类中实现了装饰器模式的核心。我们可以看到`RifleDecorator`类实现了`IRifle`接口，但有一个非常重要的细节需要注意。`GetAccuracy()`函数是虚拟的，这意味着`RifleDecorator`的任何派生类都能够重写它。

1.  现在我们有了我们的装饰器类，让我们看看实际的装饰器对象如何在运行时将其自身附加到我们的`BasicRifle`对象上：

```cs
public class WithScope : RifleDecorator
{
    private float m_ScopeAccurancy = 20.0f;

    // Constructor
    public WithScope(IRifle rifle) : base(rifle) {}

    public override float GetAccuracy()
    {
        return base.GetAccuracy() + m_ScopeAccurancy;
    }
}
```

首先要注意的是构造函数；它接受一个`IRifle`类型的对象作为参数，然后调用其基类构造函数。这种做法乍一看可能非常混乱，但一旦我们实现了这个示例的客户端，它就会变得清晰。另一个需要注意的细节是，我们正在重写`GetAccuracy()`函数，但在返回路径中通过添加`m_ScopeAccurancy`到基值来改变整支枪的整体精度。

1.  为了展示装饰器模式的灵活性，让我们在我们的示例中添加另一个装饰器：

```cs
public class WithStabilizer : RifleDecorator
{
    private float m_StabilizerAccurancy = 10.0f;

    // Constructor
    public WithStabilizer(IRifle rifle) : base(rifle) {}

    public override float GetAccuracy()
    {
        return base.GetAccuracy() + m_StabilizerAccurancy;
    }
}
```

`WithStabilizer`的实现与`WithScope`相同，只是它返回的最终精度值不同。

1.  现在，是时候实现客户端了；这是我们触发装饰器模式装饰功能的地方：

```cs
using UnityEngine;

public class Client : MonoBehaviour
{
    void Update()
    {
        if (Input.GetKeyDown("b"))
        {
            IRifle rifle = new BasicRifle();
            Debug.Log("Basic accuracy: " + rifle.GetAccuracy()); 
        }

        if (Input.GetKeyDown("s"))
        {
            IRifle rifle = new BasicRifle();
            rifle = new WithScope(rifle);
            Debug.Log("WithScope accuracy: " + rifle.GetAccuracy()); 
        }

        if (Input.GetKeyDown("t"))
        {
            IRifle rifle = new BasicRifle();
            rifle = new WithScope(new WithStabilizer(rifle));
            Debug.Log("Stabilizer+Scope accuracy: " + 
            rifle.GetAccuracy()); 
        }
    }
}
```

在这个类中最重要的元素是以下行中的构造函数调用链：

```cs
rifle = new WithScope(new WithStabilizer(rifle));
```

通过这一行代码，我们基本上通过链式调用基类构造函数，将`WithScope`和`WithStabilizer`装饰器附加到`BasicRifle`实例上。

因此，我们现在可以根据附加到`BasicRifle`实例的附件数量来获取不同的精度输出，如下所示。

以下代码返回精度为`5.0f`：

```cs
IRifle rifle = new BasicRifle();
rifle.GetAccuracy();
```

以下代码返回精度为`25.0f`：

```cs
IRifle rifle = new WithScope(rifle);
rifle.GetAccuracy();
```

以下代码返回精度为`35.0f`：

```cs
IRifle rifle = new WithScope(new WithStabilizer(rifle));
rifle.GetAccuracy();
```

因此，通过使用装饰器模式，我们现在有一个动态武器定制系统的底层实现，我们可以扩展它来构建游戏武器运行时附件的集合。

# 摘要

在本章中，我们回顾了一个为游戏程序员提供灵活方式实现经常请求的功能——武器定制的模式。看起来装饰器模式非常适合完成这类任务。但正如你所想象的那样，装饰器可以用来实现所有类型的可定制系统和服务，例如以下内容：

+   车辆升级

+   装甲和服装

在下一章中，我们将从行为模式过渡出来，转而关注解耦器，从事件总线开始。

# 练习

在本章中，我们决定使用原生 C#构造函数，这是合适的，因为我们没有使用`MonoBehaviours`或`ScritableObjects`。但这种情况并不总是如此，因此，作为一个练习，你应该尝试重构我们刚刚完成的代码示例，但不使用任何构造函数，主要使用原生的 Unity `MonoBehaviours`和`ScriptableObjects`。

你可以在官方 Unity API 文档的`ScriptableObjects`部分找到如何实现这一点的提示；请查阅*进一步阅读*部分以获取更多信息。

# 进一步阅读

+   *Unity – 脚本 API: ScriptableObject* [`docs.unity3d.com/ScriptReference/ScriptableObject.html`](https://docs.unity3d.com/ScriptReference/ScriptableObject.html)

+   《设计模式：可复用面向对象软件元素》由 Erich Gamma, John Vlissides, Ralph Johnson 和 Richard Helm 所著

    [《设计模式：可复用面向对象软件元素》](http://www.informit.com/store/design-patterns-elements-of-reusable-object-oriented-9780201633610)
