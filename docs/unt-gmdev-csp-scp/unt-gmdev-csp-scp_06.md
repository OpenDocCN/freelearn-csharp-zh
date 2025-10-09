

# 第六章：Unity 中的数据结构 – 数组、列表、字典、HashSet 和游戏逻辑

在*第五章*中，我们探讨了 Unity API 的广泛功能，通过物理模拟、场景管理和环境交互增强游戏功能，第六章深入探讨了游戏开发的核心——**数据管理**。本章从动态的游戏物理和 API 交互的世界过渡到结构化的数据处理领域，介绍了如数组、列表、字典和 HashSet 等基本数据结构。在这里，我们将揭示如何有效地组织和操作游戏数据，从管理游戏对象集合到实现复杂的库存系统。通过将这些数据结构集成到您的游戏逻辑中，您将为复杂和高效的游戏设计奠定基础，将您的开发技能提升到新的高度。

本章将涵盖以下主题：

+   实现和操作数组和列表

+   探索字典和 HashSet 以处理复杂数据

+   创建和使用自定义数据结构

+   将数据结构应用于开发游戏机制

# 技术要求

在开始之前，请确保您的开发环境已按照*第一章*中描述的设置。这包括安装最新推荐的 Unity 版本和适合您系统的代码编辑器。

## 硬件要求

确保您的计算机满足 Unity 的最低硬件规格，特别是至少支持 DX10（着色器模型 4.0）的显卡和至少 8 GB RAM 以实现最佳性能。

## 软件要求

本章的软件要求如下：

+   **Unity 编辑器**：使用从*第一章*安装的 Unity 编辑器版本，理想情况下是最新**长期支持**（**LTS**）版本（[`unity.com/download`](https://unity.com/download)）。

+   **代码编辑器**：Visual Studio 或 Visual Studio Code，应已根据初始设置集成 Unity 开发工具（[`visualstudio.microsoft.com/`](https://visualstudio.microsoft.com/))）。

您可以在此处找到与本章相关的示例/文件：[`github.com/PacktPublishing/Unity-6-Game-Development-with-C-Scripting/tree/main/Chapter06`](https://github.com/PacktPublishing/Unity-6-Game-Development-with-C-Scripting/tree/main/Chapter06)

# 理解数组和列表

在本节中，我们将深入探讨 Unity 中数据结构的基础知识，重点关注对游戏开发至关重要的基础数据类型。**数组**是一系列相同类型的元素集合，具有固定的大小和直接访问每个元素的能力。另一方面，**列表**是一种可以动态调整大小的集合，提供了更简单的操作。数组提供了直接的数据存储方法，而列表则适用于更复杂和不断变化的场景。我们将引导您了解每个数据结构的基础知识，从语法和初始化到实际用例，如库存系统和游戏对象管理。当我们深入研究使用这些结构（访问元素、迭代和修改内容）时，我们还会权衡它们的性能影响和最佳实践。这确保您可以在何时以及如何使用数组和列表来优化您的 Unity 项目做出明智的决定。

## 介绍数组

数组在编程中至关重要，提供了一种组织和管理工作数据的方法，例如 Unity 和 C#游戏中的角色统计数据或库存项目。它们是存储在一起、通过索引访问的相似元素集合，非常适合高效处理多个游戏实体。在 C#中声明数组，例如`int[] scores;`，并初始化它，无论是使用特定元素，如`int[] scores = {90, 85, 100};`，还是通过大小，如`int[] scores = new int[3];`，都非常简单，支持各种游戏开发需求。

在游戏开发中，数组被广泛用于存储敌人位置、库存项目 ID 或玩家得分，便于快速访问和更新，这对于游戏机制至关重要。例如，遍历数组以更新游戏对象位置是一个典型的用例。本质上，数组简化了游戏开发中的数据处理，其简单的语法和快速的元素访问提高了游戏系统复杂性的管理。学会有效地使用数组对于在 Unity 和 C#游戏编程中取得进步至关重要。

### 使用数组

在 Unity 和 C#中，数组在组织游戏元素（如对象、得分和库存）方面发挥着关键作用，提供了索引访问，以便高效地管理和更新数据。这使得开发者可以轻松访问特定元素以执行操作，如修改玩家的得分或敌人的健康值，增强游戏动态。

然而，数组的固定大小限制了它们的适应性，尤其是在添加或删除元素时，这需要创建新的数组以适应变化。这与更动态的数据结构（如列表）形成对比，后者在执行此类操作时提供了更大的灵活性，尽管在性能方面有所妥协。

考虑以下 C#脚本，它演示了遍历游戏对象位置数组并更新它们：

```cs
using UnityEngine;
public class ArrayExample : MonoBehaviour
{
    // Array of positions for game objects
    public Vector3[] positions = new Vector3[5];
    void Start()
    {
        // Initialize positions
        for (int i = 0; i < positions.Length; i++)
        {
            positions[i] = new Vector3(i * 2.0f, 0, 0);
        }
    }
    void Update()
    {
        // Iterate over positions and update each
        for (int i = 0; i < positions.Length; i++)
        {
            // Example update: move each position upwards every frame
            positions[i] += Vector3.up * Time.deltaTime;
        }
    }
}
```

此代码初始化了一个具有特定起始值的`Vector3`位置数组，并在每一帧更新每个位置向上移动。

因此，虽然 C#中的数组提供了可靠地存储和访问数据集合的方法，但它们的固定大小为动态操作（如添加或删除元素）带来了挑战。理解如何应对这些限制，以及熟练地访问和迭代数组元素，对于在 Unity 游戏开发中有效地利用数组至关重要。这种结构和灵活性的平衡使得数组成为游戏开发者工具箱中既强大又细腻的工具。

## 引入列表

从数组过渡到列表，C#中的列表提供了动态调整大小，非常适合游戏开发场景中元素计数变化的场景，如库存物品或屏幕上的敌人。作为 Unity 的`System.Collections.Generic`命名空间的一部分，列表提供了对数组的灵活替代方案，允许通过`Add()`和`Remove()`等方法轻松添加和删除元素。

尽管列表带来了适应性，但在高频操作中，它们与数组相比可能性能略低。然而，它们的灵活性和易用性通常超过了这些限制，尤其是在管理动态游戏元素时。

这里是一个简单的 C#脚本，演示了在 Unity 中使用列表管理敌人：

```cs
using System.Collections.Generic;
using UnityEngine;
public class ListExample : MonoBehaviour
{
    public List<GameObject> activeEnemies;
    void Start()
    {
        activeEnemies = new List<GameObject>();
        // Populate the list with Active Enemy
        for (int i = 0; i < 5; i++)
        {
            GameObject obj = new GameObject("Enemy" + i);
            activeEnemies.Add(obj);
        }
    }
    void Update()
    {
        // Example: Remove a game object from the list
        // when the 'R' key is pressed
        if (Input.GetKeyDown(KeyCode.R) &&
              activeEnemies.Count > 0)
        {
            GameObject objToRemove = activeEnemies[0];
            activeEnemies.Remove(objToRemove);
            Destroy(objToRemove);
        }
    }
}
```

此脚本在启动时填充一个包含活动敌人的列表，并在按下*R*键时从列表中删除一个对象。

理解何时以及如何有效地使用列表，与数组结合使用，可以显著增强游戏逻辑和数据管理的结构和动态性。

### 使用列表

在 C#中引入列表后，我们探讨了它们在 Unity 中的应用，其中它们的动态特性在管理游戏元素（如角色状态）方面表现出色。本节涵盖了基本操作：添加、访问、删除和迭代列表元素，这些对于游戏数据操作至关重要。

使用`Add()`方法，开发者可以轻松扩展列表，而通过索引和如`Remove()`等方法访问和删除元素变得简单。迭代通常通过循环完成，允许对元素进行批量操作。尽管列表具有多功能性，开发者应留意列表的性能影响，尤其是在频繁修改时，以防止其影响游戏性能。

这里是一个示例 C#脚本，展示了 Unity 中基本列表操作的示例：

```cs
using System.Collections.Generic;
using UnityEngine;
public class ListExample : MonoBehaviour
{
    List<string> collectedItems = new List<string>();
    void Start()
    {
        // Adding elements
        collectedItems.Add("Health Potion");
        collectedItems.Add("Mana Potion");
        // Accessing elements
        Debug.Log("First collected item: " +
                   collectedItems[0]);
        // Iterating over the list
        foreach (string item in collectedItems)
        {
            Debug.Log("Collected item: " + item);
        }
        // Removing elements
        collectedItems.Remove("Health Potion");
    }
}
```

此脚本演示了在游戏中添加、访问、迭代和从收集物品列表中删除项目。

虽然列表为游戏开发带来了无与伦比的灵活性，但谨慎使用对于保持最佳性能至关重要。掌握列表的这些方面将显著提高开发者有效管理游戏数据的能力，从而为玩家提供更丰富和更具动态性的游戏体验。

## Unity 中列表和数组的实际应用

在探索了数组和列表之后，我们转向它们在 Unity 中的实际应用，例如管理库存或敌人追踪。数组对于稳定元素来说非常理想，提供快速访问，而列表则很好地适应了动态场景，如扩展库存和根据游戏需求在结构性和灵活性之间进行平衡。

在数组与列表之间进行选择，需要权衡它们的速度和适应性，以适应游戏的设计和性能需求，确保高效的游戏对象管理。

为了说明，考虑一个简单的 Unity 中的 C#脚本，该脚本使用列表来管理角色的库存：

```cs
using System.Collections.Generic;
using UnityEngine;
public class ConversationManager : MonoBehaviour
{
    List<string> conversationHistory = new List<string>();
    public void AddItem(string item)
    {
        conversationHistory.Add(item);
    }
    public void RemoveItem(string item)
    {
        conversationHistory.Remove(item);
    }
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.I))
        {
            foreach (var item in conversationHistory)
            {
                Debug.Log("Conversation Item: " + item);
            }
        }
    }
}
```

此脚本演示了列表如何动态管理对话系统，其中可以通过`AddItem`和`RemoveItem`方法由其他游戏组件添加或删除项目。按下*I*键记录当前库存中的所有项目，展示了遍历列表的简便性。

因此，数组对于在 Unity 中稳定环境中组织游戏元素至关重要，而列表则更适合动态环境，其中元素可以频繁更改。它们的战略使用增强了游戏结构和玩家参与度。接下来，我们将探索字典和哈希集，它们为更复杂场景提供了高级数据管理选项，进一步扩展了我们的游戏开发工具集。

# 探索字典和哈希集

深入了解字典和哈希集，我们将分别探索 C#中的键值对结构和集合。**字典**是一个键值对集合，允许基于唯一键高效地访问数据，使其非常适合库存系统和游戏状态管理。**哈希集**是一个确保唯一项目并提供快速查找的集合。我们将它们与数组列表进行对比，以在 Unity 中进行复杂的数据管理。我们将探讨它们的语法、用法和性能在游戏开发中的应用，了解何时以及如何利用这些结构来实现优化和吸引人的游戏机制。本节还涵盖了管理字典中复杂键类型的高级技术和最佳实践，以及利用哈希集进行有效数据处理。首先，让我们深入了解字典。

## 介绍字典

C#中的字典是灵活的数据结构，旨在使用键值对进行高效的数据存储和检索，与数组列表中看到的顺序元素访问不同。这种独特的结构允许快速查找、更新和管理相关数据，使字典非常适合数据点之间关系重要的场景，例如音频设置或游戏难度设置值。

要声明和初始化一个 Dictionary，需要指定键和值的类型（例如，int、string、float），使用如 `Dictionary<KeyType, ValueType> dictionaryName = new Dictionary<KeyType, ValueType>();` 这样的语法。这种结构的初始化可以是直接的，也可以是预先填充元素。与数组相比，Dictionary 的突出特点是它通过键而不是索引直接访问元素，当元素顺序不是重点但快速高效地访问特定元素是必不可少的时，这提供了一种更直观的数据处理方式。

### 在 Unity 中使用 Dictionary

在 Unity 游戏开发中，Dictionary 对于结构化复杂数据，如 **非玩家角色**（NPC）对话树或玩家进度跟踪，非常有价值，它能够实现高效直观的数据操作。通过将键与值关联，开发者可以快速添加、访问和修改游戏数据，增强游戏机制和用户体验。

例如，使用 Dictionary 管理游戏成就变得简单高效，允许通过成就名称或 ID 作为键快速查找和更新成就。同样，游戏状态变量可以有效地组织，使得保存和加载游戏状态变得更容易。添加（`dictionary.Add(key, value)`）、访问（`var value = dictionary[key]`）和删除（`dictionary.Remove(key)`）键值对的操作都很简单。通过键、值或两者遍历 Dictionary，便于执行批量操作，如显示任务状态或更新角色属性。

考虑这个简单的 C# 示例，演示了用于库存系统的 Dictionary：

```cs
using System.Collections.Generic;
using UnityEngine;
public class InventoryManager : MonoBehaviour
{
    Dictionary<string, int> inventory = new
        Dictionary<string, int>();
    void Start()
    {
        // Adding items to the inventory
        inventory.Add("Potion", 5);
        inventory.Add("Sword", 1);
    }
    void Update()
    {
        // Accessing and updating an item's quantity
        if (Input.GetKeyDown(KeyCode.U))
        {
            inventory["Potion"] += 1;
            Debug.Log("Potions: " + inventory["Potion"]);
        }
    }
}
```

此脚本初始化一个 Dictionary 来管理库存，在 `Start` 方法中添加两个项目，并监听 *U* 键以增加 `Potion` 的数量 `1`，在控制台显示更新后的值。

Unity 中的 Dictionary 促进了强大灵活的数据管理，这对于玩家统计或弹药库等特性至关重要。它们处理键值对的能力使得添加、访问和遍历游戏数据变得高效，显著提高了游戏逻辑和设计工作流程。

### 在 Unity 中使用 Dictionary 的性能考虑

当在 Unity 中集成 Dictionary 时，考虑其对游戏性能的影响是至关重要的。虽然 Dictionary 通过键值对提供快速数据访问，但使用不当可能导致效率低下，尤其是在资源密集型游戏中，最佳性能至关重要。

Dictionary 通常效率很高，但使用不当会导致性能下降，例如过度添加、删除或大数据集。最佳实践包括最小化性能关键循环中的操作频率，并考虑初始容量设置以减少重新散列。此外，使用适当的键类型并确保良好的哈希值分布可以防止瓶颈。

## 介绍 HashSet

在我们探索了字典之后，我们介绍 C#中的 HashSet，这是一种功能强大的集合类型，它提供了与列表和字典不同的独特功能。HashSet 存储唯一元素，使其成为需要无重复值操作的理想选择。与负责管理键值对的字典不同，HashSet 专注于单个元素，提供高效的插入、删除和查找。

HashSet 使用类型说明符声明，类似于列表，使用`HashSet<T> myHashSet = new HashSet<T>();`语法，其中`T`表示存储的元素类型。初始化 HashSet 可以涉及逐个添加元素或传递整个集合。HashSet 中元素的唯一性固有地防止了重复，简化了某些操作，例如检查现有值或从集合中消除重复条目。

它们的简单语法和初始化以及唯一元素的保证使 HashSet 成为 Unity 开发中特定场景的有价值工具，增强了数据处理和性能。

### 在 Unity 中使用 HashSet

在 Unity 中，HashSet 在管理唯一元素和简化添加、检查和删除项目等操作方面表现出色，这对于游戏开发任务，如跟踪收集品或管理敌人至关重要。它们在防止重复和促进快速查找方面的效率使它们成为维护游戏数据完整性和动态环境性能的有价值工具。

例如，管理玩家收集的独特道具集合可以使用 HashSet 高效处理：

```cs
using System.Collections.Generic;
using UnityEngine;
public class PowerUpManager : MonoBehaviour
{
    HashSet<string> collectedPowerUps = new
       HashSet<string>();
    void CollectPowerUp(string powerUpName)
    {
        // Adds the power-up to the collection if it's
        // not already present
        bool added = collectedPowerUps.Add(powerUpName);
        if (added)
        {
            Debug.Log(powerUpName + " collected!");
        }
    }
}
```

此脚本使用 HashSet 来管理收集的道具，确保每个道具只添加一次，并在收集新道具时记录一条消息，通过其他脚本调用`CollectPowerUp`方法。

现在我们已经定义了 HashSet 并探讨了它们的独特优势，让我们将它们与字典进行比较，以了解每个如何在不同的游戏开发场景中有效利用。

## 比较字典和哈希集合

在 Unity 游戏开发中，在字典和 HashSet 之间选择合适的数据结构对于高效性能和代码清晰度至关重要。具有键值对的字典在需要关联数据管理的情况下表现卓越，而 HashSet 在维护无重复项的唯一集合方面是最优的。

当需要根据特定键检索或修改数据时，例如根据 ID 跟踪玩家得分或管理游戏状态设置，字典是理想的。它们的主要优势在于快速的查找和数据组织，但如果没有键值关联，仅需要唯一性时，它们可能会变得繁琐。另一方面，哈希集合在强调项目唯一性和快速成员检查的情况下表现出色，例如确保不会生成重复的敌人或管理独特的可收集物品。虽然哈希集合在添加、删除和搜索操作中提供了高性能，但它们缺乏字典中键和值之间的直接关联。

比较了字典和哈希集合的功能和应用后，让我们深入了解一些高级技巧和技术，以进一步优化它们在 Unity 中的使用，从而提高游戏项目中的性能和可扩展性。

## 高级技巧和技术

探索字典和哈希集合揭示了 Unity 中的高级数据管理技术。在字典中使用复杂键时，需要注意相等性和哈希码方法，而哈希集合在快速存在检查方面表现出色，这对于独特的游戏元素非常有用。这些高级用法要求对 C#的数据结构有扎实的掌握，从而提高游戏性能和可扩展性。

从本质上讲，字典非常适合复杂的键值关联，而哈希集合在处理唯一项集合时表现出色，侧重于性能。选择正确的结构取决于游戏的数据需求，影响整体效率和架构。

在掌握了使用字典和哈希集合的高级技巧之后，我们现在可以探索在 Unity 中创建自定义数据结构，以进一步定制数据管理以满足您游戏的具体需求。

# Unity 中的自定义数据结构

从字典和哈希集合转移到，我们现在探索 Unity 中的自定义数据结构，这对于寻求高级游戏设计解决方案的开发者来说是必不可少的。这些结构为特定的游戏需求提供了定制的方法，提高了性能和可用性。本节涵盖了为复杂场景设计独特结构，例如关卡布局或 AI 逻辑，以及使用 C#、ScriptableObject 等在 Unity 中的实现和集成。我们将讨论序列化、内存管理以及泛型和设计模式等高级概念，以实现高效的定制结构。通过实际示例和强调最佳实践，我们的目标是指导您制作和优化定制数据结构，最终以创建自定义库存系统的教程作为总结。

自定义数据结构在游戏开发中至关重要，它能够提供标准类型无法提供的定制解决方案。当出现独特的游戏特性或数据管理需求时，这些结构尤其有价值，它们能够提供比数组或列表等预定义结构更多的功能。面对诸如优化性能、增强数据组织或实现复杂游戏机制等特定挑战时，考虑自定义结构至关重要。需要采用新颖的数据处理方法的情况，例如复杂的库存系统或角色属性，表明需要定制解决方案。这些结构将数据管理紧密地与游戏的概念设计相结合，提高了代码的可维护性和游戏功能。

### 设计自定义数据结构

在游戏开发中，设计自定义数据结构需要平衡性能、内存效率和可用性，以有效地管理游戏数据。最佳性能确保这些结构不会减慢游戏速度，尤其是在资源密集型场景中，而谨慎的内存使用有助于保持轻量级的足迹，这对于复杂环境至关重要。此外，易用性是核心，它允许开发者将这些结构顺利地集成到他们的工作流程中，从而提高开发效率。

在创建复杂的库存系统、通过空间分区优化碰撞检测或管理程序生成世界的数据等场景中，自定义数据结构变得必要。这些独特的挑战超出了标准结构的范围，使得自定义解决方案对于复杂的游戏机制和性能优化至关重要。

### 在 C#中实现自定义数据结构

在 C#中实现自定义数据结构需要深入理解类和结构体的基础知识，从而能够创建定制且高效的容器。类提供了适用于需要继承和广泛功能的复杂数据场景的引用类型结构，而结构体提供了适用于轻量级、不可变数据存储的值类型语义。

自定义结构是通过构造函数进行初始化、通过属性进行数据封装以及通过方法定义行为来构建的。构造函数设置初始状态，属性管理对结构体数据的访问，而方法实现了结构体的功能。这种方法允许创建与游戏独特需求紧密对齐的高度专业化的数据结构。

考虑以下简单的 C#示例，用于表示二维点的自定义数据结构：

```cs
public struct Point2D
{
    public int X { get; set; }
    public int Y { get; set; }
    public Point2D(int x, int y)
    {
        X = x;
        Y = y;
    }
    public void Translate(int dx, int dy)
    {
        X += dx;
        Y += dy;
    }
}
```

这个`Point2D`结构体演示了一个基本的自定义数据结构，具有`X`和`Y`坐标属性以及一个用于平移点的`translate`方法：

```cs
using UnityEngine;
public class Point2DExample : MonoBehaviour
{
    void Start()
    {
        // Create an instance of Point2D
        Point2D point = new Point2D(3, 4);
        // Log the initial coordinates
        Debug.Log("Initial Point: (" + point.X + ", " +
           point.Y + ")");
        // Translate the point
        point.Translate(2, 3);
        // Log the new coordinates after translation
        Debug.Log("Translated Point: (" + point.X + ", " +
           point.Y + ")");
    }
}
```

`Point2DExample`脚本演示了创建`Point2D`实例，记录其初始坐标，通过向其 X 和 Y 坐标添加值来平移点，然后记录更新后的坐标。这展示了如何使用`Point2D`结构在 Unity 中管理和操作 2D 点数据。这种简单、自定义的结构增强了可读性和可维护性，使开发者能够更直观地建模游戏数据。

### 在 Unity 中集成自定义结构

将自定义数据结构集成到 Unity 中涉及利用 Unity 特定的功能，如`ScriptableObject`进行复杂的数据存储，以及理解序列化细节以实现游戏数据持久化。这种方法通过允许更复杂的数据管理和针对 Unity 环境的定制状态保存机制来增强游戏架构。

`ScriptableObject`允许创建不绑定到场景对象的数据容器，非常适合存储游戏设置、角色统计数据或库存物品，并且可以在 Unity 编辑器中轻松编辑。在设计自定义结构时，确保它们可序列化以保持游戏状态跨会话，需要关注 Unity 的序列化规则和属性。

例如，在 C#中创建一个用于库存系统的`ScriptableObject`可能看起来像这样：

```cs
using UnityEngine;
using System.Collections.Generic;
[CreateAssetMenu(fileName = "New Inventory", menuName = "Inventory System/Inventory")]
public class Quests : ScriptableObject
{
    public List<Quest> completedQuests = new List<Quest>();
    public void AddItem(Item itemToAdd)
    {
        // Add quests to the list
    }
}
```

本示例定义了一个基本的任务追踪系统，其中可以通过利用`ScriptableObject`来创建一个灵活且可重用的任务资产，从而动态地添加项目。将此类自定义结构有效地集成到 Unity 中不仅拓宽了游戏的设计可能性，而且简化了开发工作流程。

### 游戏开发中的常见自定义结构

在游戏开发中，常见的自定义数据结构，如网格、树和图，在创建沉浸式世界和智能行为中发挥着关键作用。这些结构支撑着游戏设计的各个方面，从关卡布局到 AI 决策和导航，为复杂问题提供定制解决方案。

网格和矩阵是设计游戏关卡的基础，为地图创建和空间组织提供了一种结构化的方法。树，尤其是行为树，对于结构化 AI 决策过程至关重要，它使得 AI 行为脚本的编写具有清晰、模块化的方法。图有助于表示相互连接的空间，这对于路径查找算法和地图导航至关重要。实现这些自定义结构增强了游戏功能，有助于创造更动态的环境和更复杂的游戏机制。

### 游戏开发中数据结构的高级技术

探索游戏开发中的高级技术，如泛型和设计模式的使用，可以显著提高数据结构的灵活性和效率。这些方法允许编写更适应性和可维护的代码，这对于复杂游戏系统至关重要。

C# 中的泛型允许开发者创建可通用的数据结构，这些数据结构可以与各种数据类型一起操作，从而生成可重用且类型安全的代码，这确保了与数据类型错误相关的错误在编译时而不是在运行时被捕获。设计模式，作为解决常见软件设计问题的可重用解决方案，例如工厂和构建者模式，通过提供对象创建和配置的清晰范例进一步精炼数据结构。工厂模式简化了对象创建，而不需要指定确切的类，而构建者模式允许逐步构建复杂对象。这些模式简化了复杂游戏组件和系统的开发过程。采用这些高级技术促进了健壮和可扩展的游戏架构，适应游戏开发项目的不断变化需求。

### 实际示例 - 构建自定义库存系统

创建一个自定义库存系统展示了在游戏开发中自定义数据结构的实际应用，展示了如何动态和高效地管理游戏中的物品。这种方法允许进行定制的库存管理，适应游戏的具体需求和机制。

要构建一个库存系统，首先定义一个可以存储物品的数据结构，以及添加、删除和查询这些物品的方法。这个系统应该足够灵活，以适应各种物品类型和数量。处理物品添加涉及向库存结构中添加，而删除则需要相应地检查和更新库存。物品查询可能涉及检查物品的存在或根据某些标准检索物品列表。

这里是一个基本的 C# 库存系统示例：

```cs
using System.Collections.Generic;
using UnityEngine;
public class Recipes : MonoBehaviour
{
    private List<string> availableRecipes = new
      List<string>();
    public void AddItem(string recipe)
    {
        availableRecipes.Add(recipe);
    }
    public bool RemoveItem(string recipe)
    {
        return availableRecipes.Remove(recipe);
    }
    public bool CheckItem(string recipe)
    {
        return availableRecipes.Contains(item);
    }
}
```

此示例概述了一个使用 `List` 管理物品的简单库存系统。`AddItem` 方法允许向食谱列表中添加新物品。`RemoveItem` 方法从食谱列表中删除食谱，并返回一个布尔值，指示删除是否成功。`CheckItem` 方法检查特定食谱是否存在于食谱列表中，并返回一个布尔结果。这为游戏开发中更复杂的食谱跟踪需求提供了一个基础结构，允许在游戏中实现基本的食谱管理功能。

在 Unity 中，自定义数据结构对于满足特定游戏开发需求至关重要，从基础概念和设计考虑，如性能和可用性，到其实际应用和通过 C#和`ScriptableObject`在 Unity 中的集成。这次探索涵盖了各种结构，如用于关卡设计的网格和用于 AI 的树，以及涉及泛型和设计模式的先进技术。我们专注于优化和最佳实践，如高效的内存管理和避免常见陷阱，为创建健壮且高效的自定义结构奠定了基础。构建自定义库存系统的实际演练展示了这些概念的实际应用。当我们过渡到讨论游戏逻辑时，从自定义数据结构中获得的知识将在构建复杂游戏机制时变得极为宝贵。

# 游戏逻辑的数据结构

让我们深入探讨 Unity 游戏开发中游戏逻辑与数据结构的关键交汇点。从基础开始，我们将探讨数组和列表等基本数据结构如何构成游戏逻辑的骨架，促进核心功能，如库存系统和角色管理。接着，我们将探讨复杂的数据管理实践，例如在协调游戏状态、资产管理以及独特物品跟踪中使用字典和哈希集。本节还将阐明针对特定游戏开发需求定制数据结构的方法，例如技能树或网络交互。这一旅程的最终目标是全面讨论这些数据结构与 Unity 组件的无缝集成，强调性能优化和实际应用。通过这种结构化的方法，本节旨在为游戏开发者提供有效利用数据结构的知识，从而增强游戏逻辑和整体项目性能。

## 游戏逻辑和数据结构的基本原理

游戏逻辑构成了游戏开发中交互体验的核心，从角色移动到复杂的决策过程，无所不包。实现这些游戏逻辑元素的核心是基础数据结构，如数组和列表，它们为高效组织和操作游戏数据提供了必要的框架。这些结构促进了各种游戏逻辑的实现，使开发者能够轻松精确地创建动态库存系统、管理角色属性以及跟踪游戏中的实体。

尤其是数组和列表，作为构建游戏元素结构化且易于访问的基础。无论是处理玩家可以携带的物品集合，还是在游戏世界中维护非玩家角色列表，这些数据结构提供了实施游戏逻辑所需的灵活性和性能。通过理解和利用数组和列表，开发者可以确保他们游戏的核心组件——如库存管理和角色状态跟踪——既强大又能够适应游戏开发的复杂性。

## 游戏开发中的高级数据管理

随着游戏开发项目复杂性的增加，对更高级数据管理策略的需求变得至关重要。字典和哈希集作为高级数据结构，在管理游戏状态和资产以及确保集合的唯一性方面表现出色，例如库存物品或敌人实体。字典通过其键值对提供了一种有效的方法，将特定的游戏状态或资产与唯一的标识符关联起来，从而促进快速访问和修改。哈希集在管理唯一性至关重要的集合时非常有价值，消除了检查重复项的开销，并提高了性能。

除了这些，数据结构的定制设计针对复杂游戏系统独特的挑战提供解决方案，例如技能树，它需要层次化组织，或者需要高效、低延迟数据交换的网络系统。构建这些定制结构需要深入理解游戏的需求和底层算法，确保数据管理骨干既强大又能够随着游戏不断发展的需求进行扩展。这些高级数据管理工具共同构成了开发者可用的多功能工具包，使他们能够构建更丰富、更动态的游戏世界。

## Unity 中数据结构的优化和集成

在 Unity 中集成和优化数据结构涉及确保它们与 Unity 的组件无缝工作以实现最佳性能。利用 Unity 的本地类型，如 GameObject 和 ScriptableObjects，确保与内置功能的兼容性，从而实现更流畅的开发和更简单的维护。遵循 Unity 的约定简化了序列化、资产管理以及场景组织等流程。创建自定义系统可能导致兼容性问题并增加复杂性。适当的集成和性能优化可以防止在资源密集型场景或复杂游戏机制中出现瓶颈，从而提高游戏代码的可维护性。

在现实世界的游戏开发场景中应用这些概念需要精心设计、实施和改进游戏功能的方法。例如，在开发角色库存系统时，开发者必须考虑数据结构如何与 Unity 的 UI 组件交互，处理游戏保存的序列化，并确保库存更新不会妨碍游戏性能。通过使用本章讨论的概念并定期评估性能，开发者可以有效地集成和优化数据结构。这涉及到使用各种数据容器来增强游戏和用户体验。确保不同数据结构的平衡使用可以有效地进行资源管理，并使游戏机制流畅，突出了在 Unity 游戏项目中熟练的数据结构优化和集成的重要性。

# 摘要

在这次对 Unity 中数据结构的探索中，我们深入研究了数组与列表的基础元素，它们在游戏逻辑中的关键作用，以及它们为管理游戏对象和系统（如库存和角色属性）提供的动态能力。我们考察了这些结构如何支撑核心游戏机制和逻辑，为游戏功能提供基本框架。此外，我们还扩展到了字典和哈希集的领域，强调了它们在状态管理、资产跟踪和确保唯一集合方面的专用用途，这对于高级游戏开发场景至关重要。这次旅程还包括了定制数据结构，这些结构旨在适应复杂系统并增强游戏功能，强调了在 Unity 生态系统中的集成和优化，以确保最佳性能和无缝的游戏体验。

当我们从游戏开发的结构基础过渡到 Unity 的用户界面（UI）元素领域时，从数据管理中获得的原则和见解将证明非常有价值。理解如何有效地组织和操作数据不仅支撑了游戏逻辑和系统设计，还支撑了直观和响应式的 UI 创建，弥合了后端机制和前端用户交互之间的差距。接下来的章节将在此基础上构建，探讨数据结构如何影响 UI 设计和功能，增强玩家参与度和整体游戏质量。

# 加入我们的 Discord 社区

加入我们的社区 Discord 空间，与作者和其他读者进行讨论：[`packt.link/gamedevelopment`](https://discord.com/invite/NnJesrUJbu?link_from_packtlink=yes)

![免责声明二维码](img/Disclaimer_QR2.jpg)
