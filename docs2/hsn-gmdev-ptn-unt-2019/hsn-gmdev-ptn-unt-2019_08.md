# 抽象工厂

在上一章中，我们探讨了工厂方法（Factory Method），这是工厂模式的一种直接、简单直接的变体。现在，我们将实现工厂模式的一个更高级版本：被良好命名的抽象工厂（Abstract Factory）。这两种工厂模式的主要目标都是封装对象的创建过程。在本章中，我们将专注于区分工厂方法和抽象工厂之间的主要区别，以便我们更好地理解在什么情况下我们可能会选择其中一个而不是另一个。

本章将涵盖以下主题：

+   抽象工厂的基本原理

+   使用抽象工厂模式设计 NPC 生成器

解释工厂方法（Factory Method）和抽象工厂（Abstract Factory）之间的核心区别有时会被用作技术面试中的陷阱问题。因此，对这类问题有一个清晰的答案可以给面试官留下深刻印象。

# 技术要求

本章非常注重实践，因此你需要对 Unity 和 C#有基本的了解。

我们将使用以下 Unity 引擎和 C#语言概念：

+   枚举

如果你对这些概念不熟悉，请在开始本章之前复习它们。

本章的代码文件可以在 GitHub 上找到：

[`github.com/PacktPublishing/Hands-On-Game-Development-Patterns-with-Unity-2018`](https://github.com/PacktPublishing/Hands-On-Game-Development-Patterns-with-Unity-2018)

查看以下视频，以查看代码的实际应用：

[`bit.ly/2HKybdy`](http://bit.ly/2HKybdy)

# 抽象工厂概述

抽象工厂通常在学术文档中被过度复杂化地解释，但如果你将其简化到基本形式，其设计和意图相当简单。抽象工厂的主要目的是将产品的制造过程（对象）组织成相关的组。这种方法使我们能够管理生产特定家族产品的工厂（对象）。换句话说，我们能够为特定类别的产品（对象）的创建过程添加抽象层，以及特定的单个类型。

在以下图中，我们可以看到抽象工厂的基本结构以视觉方式描述：

![](img/880e98a3-46b7-4544-a961-ac8bb087e9b7.png)

注意以下该模式的成员：

+   `FactoryProducer` 负责返回特定类别（家族）的产品（人类或动物）的单独工厂。

+   `HumanFactory` 和 `AnimalFactory` 负责创建人类或动物产品

当你处理创建产品目录时，每个组都有其独特的制造规范，使用抽象工厂的好处是显而易见的。

当描述工厂模式时，我们经常使用现实世界的术语，如*产品*、*制造*和*生产者*。找到现实生活概念和计算机科学术语之间的相关性总是一个明智的方法，因为它有助于识别和记住它们的核心目的。

# 好处和缺点

抽象工厂的好处和缺点与工厂方法非常相似，所以在这个章节中不需要重复。但有一个显著的好处，这是两种模式的核心区别，我们需要解决。虽然工厂方法侧重于提供一个方法，允许我们请求创建特定类型的对象，但抽象工厂则更进一步，它为我们提供了一种管理特定组对象创建的方式。

这种区别一开始可能看起来并不重要，但如果考虑现实世界的类比，比如制造汽车的过程，我们可以看到为什么抽象工厂是有优势的。典型的汽车是由单独制造的组件组装而成，但发动机和轮胎的制造过程完全不同，因此需要单独的工厂来创建这些最终产品的关键部件。这就是为什么抽象工厂在软件开发中是有益的，因为它为我们提供了一种类似的方法来构建和组织我们生产最终复杂产品的方式，该产品具有多个组件和不同的对象创建过程。

# 用例示例

我们将扩展第四章中的用例“*工厂方法*”，通过添加一个新的可生成 NPC 的类型，称为“动物”。因此，在我们的例子中，人类和动物被认为是不可玩的角色，但它们有独立的制造过程，所以它们将需要单独的工厂。这种需求可以通过抽象工厂轻松实现。

# 代码示例

我们的代码示例几乎与我们在第四章中完成的那个相同，*工厂方法*。但我们将通过包括特定的 NPC 系列来增加系统的深度；在我们的情况下，人类和动物。

1.  抽象工厂的一个关键元素是每个产品系列都有一个相关的工厂：

```cs
using UnityEngine;

public class FactoryProducer : MonoBehaviour
{
    public static AbstractFactory GetFactory(FactoryType factoryType)
    {
        switch (factoryType)
        {
            case FactoryType.Human:
                AbstractFactory humanFactory = new HumanFactory();
                return humanFactory;
            case FactoryType.Animal:
                AbstractFactory animalFactory = new AnimalFactory();
                return animalFactory;
        }
            return null;
    }
}
```

注意到这个类是按照工厂方法模式实现的，因为我们使用简单的`switch` case 来根据请求的类型返回正确的`Factory`给客户端。

1.  现在，我们需要一个`抽象`类来保持每个特定产品`Factory`实现的连贯性：

```cs
public abstract class AbstractFactory
{
    public abstract IHuman GetHuman(HumanType humanType);
    public abstract IAnimal GetAnimal(AnimalType animalType);
}
```

1.  接下来是我们的第一个具体产品工厂，`HumanFactory`：

```cs
public class HumanFactory : AbstractFactory
{
    public override IHuman GetHuman(HumanType humanType)
    {
        switch (humanType)
        {
            case HumanType.Beggar:
                IHuman beggar = new Beggar();
                return beggar;
            case HumanType.Farmer:
                IHuman farmer = new Farmer();
                return farmer;
            case HumanType.Shopowner:
                IHuman shopowner = new Shopowner();
                return shopowner;
        }
        return null;
    }

    public override IAnimal GetAnimal(AnimalType animalType)
    {
        return null;
    }
}
```

1.  现在，`AnimalFactory`，它将生产猫和狗：

```cs
public class AnimalFactory : AbstractFactory
{
    public override IAnimal GetAnimal(AnimalType animalType)
    {
        switch (animalType)
        {
            case AnimalType.Cat:
                IAnimal cat = new Cat();
                return cat;
            case AnimalType.Dog:
                IAnimal dog = new Dog();
                return dog;
        }
        return null;
    }

    public override IHuman GetHuman(HumanType humanType)
    {
        return null;
    }
}
```

注意到这两个类都实现了对方的`GetAnimal()`或`GetHuman()`函数，但根据上下文返回`null`。这种方法是在客户端在请求特定类型的 NPC 时引用了错误的工厂时使用的；而不是抛出异常，它将收到一个`null`。

1.  我们将不再在`switch`类型的条件块中使用字符串，我们将为每个支持的产品类型实现枚举，包括相关的工厂，如下所示。这种方法将避免错误并保持一致性：

+   `FactoryType`:

```cs
public enum FactoryType
{
    Human,
    Animal
}
```

+   `HumanType`:

```cs
public enum HumanType
{
    Farmer,
    Beggar,
    Shopowner
}
```

+   `AnimalType`:

```cs
public enum AnimalType
{
    Dog,
    Cat
}
```

1.  我们的食物不会说话，但人类会说话，所以它们不能共享一个标准接口。在这种情况下，我们将为每种类型实现一个，如下所示：

+   `IHuman`:

```cs
public interface IHuman
{
    void Speak();
}
```

+   `IAnimal`:

```cs
public interface IAnimal
{
    void Voice();
}
```

1.  现在，我们需要编写所有人类和动物 NPC 的每个具体类，如下所示：

+   `Beggar`:

```cs
using UnityEngine;

public class Beggar : IHuman
{
    public void Speak()
    {
        Debug.Log("Beggar: Do you have some change to spare?");
    }
}
```

+   `Farmer`:

```cs
using UnityEngine;

public class Farmer : IHuman
{
    public void Speak()
    {
            Debug.Log("Farmer: You reap what you sow!");
    }
}
```

+   `Shopowner`:

```cs
using UnityEngine;

public class Shopowner : IHuman
{
    public void Speak()
    {
        Debug.Log("Shopowner: Do you wish to purchase something?");
    }
}
```

+   `Dog`:

```cs
using UnityEngine;

public class Dog : IAnimal
{
    public void Voice()
    {
        Debug.Log("Dog: Woof!");
    }
}
```

+   `Cat`:

```cs
public class Cat : IAnimal
{
    public void Voice()
    {
        Debug.Log("Cat: Meow!");
    }
}
```

1.  最后，我们可以扩展我们的`NPCSpawner`类以支持生成动物和人类 NPC：

```cs
public class NPCSpawner : MonoBehaviour
{
    private IAnimal m_Cat;
    private IAnimal m_Dog;

    private IHuman m_Farmer;
    private IHuman m_Beggar;
    private IHuman m_Shopowner;

    private AbstractFactory factory;

    public void SpawnAnimals()
    {
        factory = FactoryProducer.GetFactory(FactoryType.Animal);

        m_Cat = factory.GetAnimal(AnimalType.Cat);
        m_Dog = factory.GetAnimal(AnimalType.Dog);

        m_Cat.Voice();
        m_Dog.Voice();
    }

    public void SpawnHumans()
    {
        factory = FactoryProducer.GetFactory(FactoryType.Human);

        m_Beggar = factory.GetHuman(HumanType.Beggar);
        m_Farmer = factory.GetHuman(HumanType.Farmer);
        m_Shopowner = factory.GetHuman(HumanType.Shopowner);

        m_Beggar.Speak();
        m_Farmer.Speak();
        m_Shopowner.Speak();
    }
}
```

1.  作为我们概念的证明，我们的`Client`类可以从我们的`Spawner`请求`Animal`和`Human` NPC，而无需了解最终产品的创建过程：

```cs
public class Client : MonoBehaviour
{
    public NPCSpawner m_SpawnerNPC;

    public void Update()
    {
        if (Input.GetKeyDown(KeyCode.U))
        {
            m_SpawnerNPC.SpawnHumans();
        }

        if (Input.GetKeyDown(KeyCode.A))
        {
            m_SpawnerNPC.SpawnAnimals();
        }
    }
}
```

如你所见，抽象工厂比其表亲工厂方法提供了更多的灵活性。我们现在可以管理产品系列，并为制造过程添加更多抽象层。

# 摘要

在本章中，我们学习了抽象工厂，它是工厂方法的近亲。正如其名称所暗示的，抽象工厂允许我们在特定类型产品的制造过程中添加抽象层。当处理多个产品系列时，这种模式非常有用。与工厂方法相比，抽象工厂的主要缺点是它更冗长且复杂。

在下一章中，我们将探讨可能是所有设计模式中最著名的一个：单例。

在学习工厂模式的过程中，你可能会注意到我们使用了与传统制造过程相关的几个术语。制造业和软件产业密切相关。与工厂管理相关的最佳实践启发了 DevOps 和看板背后的几个核心思想，这些思想现在是稳健软件开发流程的基石。

# 实践练习

在本书的这一部分，我们已经回顾了工厂模式中最流行的两种形式：抽象工厂和工厂方法。然而，作为一个练习，我建议通过实现第三种形式来扩展你对工厂的了解，例如，静态工厂方法。

你可以在 Joshua Bloch 的经典著作《Effective Java》中了解静态工厂方法模式。更多信息可以在“进一步阅读”部分找到。

# 进一步阅读

+   Kevin Behr、George Spafford 和 Gene Kim 合著的《凤凰项目》[*The Phoenix Project*](https://itrevolution.com/book/the-phoenix-project)

+   *《有效 Java》* by Joshua Bloch[`www.pearson.com/us/higher-education/program/Bloch-Effective-Java-3rd-Edition/PGM1763855.html`](https://www.pearson.com/us/higher-education/program/Bloch-Effective-Java-3rd-Edition/PGM1763855.html)
