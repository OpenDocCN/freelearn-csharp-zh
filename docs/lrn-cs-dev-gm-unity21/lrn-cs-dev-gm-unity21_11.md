# 第十一章：11

# 介绍堆栈、队列和 HashSet

在上一章中，我们重新访问了变量、类型和类，看看它们在书的开头介绍的基本功能之外还提供了什么。在本章中，我们将更仔细地研究新的集合类型，并了解它们的中级能力。请记住，成为一个好的程序员并不是关于记忆代码，而是选择合适的工具来完成合适的工作。

本章中的每种新集合类型都有特定的目的。在大多数需要数据集合的情况下，列表或数组都可以很好地工作。然而，当您需要临时存储或控制集合元素的顺序，或更具体地说，它们被访问的顺序时，可以使用堆栈和队列。当您需要执行依赖于集合中每个元素都是唯一的操作时，可以使用 HashSet。

在您开始下一节中的代码之前，让我们列出您将要学习的主题：

+   介绍堆栈

+   查看和弹出元素

+   使用队列

+   添加、移除和查看元素

+   使用 HashSet

+   执行操作

# 介绍堆栈

在其最基本的层面上，堆栈是相同指定类型的元素集合。堆栈的长度是可变的，这意味着它可以根据它所持有的元素数量而改变。堆栈与列表或数组之间的重要区别在于元素的存储方式。而列表或数组按索引存储元素，堆栈遵循**后进先出**（**LIFO**）模型，这意味着堆栈中的最后一个元素是第一个可访问的元素。这在您想要以相反顺序访问元素时非常有用。您应该注意它们可以存储`null`和重复值。一个有用的类比是一叠盘子——您放在堆栈上的最后一个盘子是您可以轻松拿到的第一个盘子。一旦它被移除，您堆叠的倒数第二个盘子就可以访问，依此类推。

本章中的所有集合类型都是`System.Collections.Generic`命名空间的一部分，这意味着您需要在要在其中使用它们的任何文件的顶部添加以下代码：

```cs
using System.Collections.Generic; 
```

现在您知道您将要处理的内容，让我们来看一下声明堆栈的基本语法。

堆栈变量声明需要满足以下要求：

+   `Stack`关键字，其元素类型在左右箭头字符之间，以及一个唯一名称

+   `new`关键字用于在内存中初始化堆栈，后跟`Stack`关键字和箭头字符之间的元素类型

+   由分号结束的一对括号

在蓝图形式中，它看起来像这样：

```cs
Stack<elementType> name = new Stack<elementType>(); 
```

与您之前使用过的其他集合类型不同，堆栈在创建时不能用元素初始化。相反，所有元素都必须在创建堆栈后添加。

C#支持不需要定义堆栈中元素类型的非通用版本：

```cs
Stack myStack = new Stack(); 
```

然而，这比使用前面的通用版本更不安全且更昂贵，因此建议使用上面的通用版本。您可以在[`github.com/dotnet/platform-compat/blob/master/docs/DE0006.md`](https://github.com/dotnet/platform-compat/blob/master/docs/DE0006.md)上阅读有关 Microsoft 的建议的更多信息。

您的下一个任务是创建自己的堆栈，并亲自体验使用其类方法。

为了测试这一点，您将使用堆栈修改*英雄诞生*中的现有物品收集逻辑，以存储可以收集的可能战利品。堆栈在这里很有效，因为我们不必担心提供索引来获取战利品，我们可以每次都获取最后添加的战利品：

1.  打开`GameBehavior.cs`并添加一个名为`LootStack`的新堆栈变量：

```cs
**// 1**
public Stack<string> LootStack = new Stack<string>(); 
```

1.  使用以下代码更新`Initialize`方法以向堆栈添加新项：

```cs
public void Initialize() 
{
    _state = "Game Manager initialized..";
    _state.FancyDebug();
    Debug.Log(_state);
    **// 2**
    **LootStack.Push(****"Sword of Doom"****);**
    **LootStack.Push(****"HP Boost"****);**
    **LootStack.Push(****"Golden Key"****);**
    **LootStack.Push(****"Pair of Winged Boots"****);**
    **LootStack.Push(****"Mythril Bracer"****);**
} 
```

1.  在脚本底部添加一个新方法来打印堆栈信息：

```cs
**// 3**
public void PrintLootReport()
{
    Debug.LogFormat("There are {0} random loot items waiting 
       for you!", LootStack.Count);
} 
```

1.  打开`ItemBehavior.cs`，并从`GameManager`实例中调用`PrintLootReport`：

```cs
void OnCollisionEnter(Collision collision)
{
    if(collision.gameObject.name == "Player")
    {
        Destroy(this.transform.parent.gameObject);
        Debug.Log("Item collected!");
        GameManager.Items += 1;

        **// 4**
        **GameManager.PrintLootReport();**
    }
} 
```

将其分解，它执行以下操作：

1.  创建一个空堆栈，其中包含字符串类型的元素，用于保存我们接下来要添加的战利品

1.  使用`Push`方法向堆栈中添加字符串元素（即战利品名称），每次增加其大小

1.  每当调用`PrintLootReport`方法时，都会打印出堆栈计数

1.  在`OnCollisionEnter`中调用`PrintLootReport`，每当玩家收集一个物品时都会调用，我们在之前的章节中使用 Collider 组件进行了设置。

在 Unity 中点击播放，收集一个物品预制件，并查看打印出来的新战利品报告。

![](img/B17573_11_01.png)

图 11.1：使用堆栈的输出

现在您已经有一个可以保存所有游戏战利品的工作堆栈，您可以开始尝试使用堆栈类的`Pop`和`Peek`方法访问物品。

## 弹出和窥视

我们已经讨论过堆栈如何使用 LIFO 方法存储元素。现在，我们需要看一下如何访问熟悉但不同的集合类型中的元素——通过窥视和弹出：

+   `Peek`方法返回堆栈中的下一个物品，而不移除它，让您可以在不改变任何内容的情况下“窥视”它

+   `Pop`方法返回并移除堆栈中的下一个物品，实质上是“弹出”它并交给您

这两种方法可以根据您的需要单独或一起使用。在接下来的部分中，您将亲身体验这两种方法。

您的下一个任务是抓取添加到`LootStack`中的最后一个物品。在我们的示例中，最后一个元素是在`Initialize`方法中以编程方式确定的，但您也可以在`Initialize`中以编程方式随机排列添加到堆栈中的战利品的顺序。无论哪种方式，都要在`GameBehavior`中更新`PrintLootReport()`，使用以下代码：

```cs
public void PrintLootReport()
{
    **// 1**
    **var** **currentItem = LootStack.Pop();**
    **// 2**
    **var** **nextItem = LootStack.Peek();**
    **// 3**
    **Debug.LogFormat(****"You got a {0}! You've got a good chance of finding a {1} next!"****, currentItem, nextItem);**
    Debug.LogFormat("There are {0} random loot items waiting for you!", LootStack.Count);
} 
```

以下是正在发生的事情：

1.  在`LootStack`上调用`Pop`，移除堆栈中的下一个物品，并存储它。请记住，堆栈元素是按照 LIFO 模型排序的。

1.  在`LootStack`上调用`Peek`，并存储堆栈中的下一个物品，而不移除它。

1.  添加一个新的调试日志，打印出弹出的物品和堆栈中的下一个物品。

您可以从控制台看到，**秘银护腕**是最后添加到堆栈中的物品，被最先弹出，接着是**一双翅膀靴**，它被窥视但没有被移除。您还可以看到`LootStack`还有四个剩余的可以访问的元素：

![](img/B17573_11_02.png)

图 11.2：从堆栈中弹出和窥视的输出

我们的玩家现在可以按照堆栈中添加的相反顺序拾取战利品。例如，首先拾取的物品将始终是**秘银护腕**，然后是**一双翅膀靴**，然后是**金色钥匙**，依此类推。

现在您知道如何创建、添加和查询堆栈中的元素，我们可以继续学习通过堆栈类可以访问的一些常见方法。

## 常见方法

本节中的每个方法仅用于示例目的，它们不包括在我们的游戏中，因为我们不需要这些功能。

首先，您可以使用`Clear`方法清空或删除堆栈的全部内容：

```cs
// Empty the stack and reverting the count to 0
LootStack**.Clear();** 
```

如果您想知道您的堆栈中是否存在某个元素，请使用`Contains`方法并指定您要查找的元素：

```cs
// Returns true for "Golden Key" item
var itemFound = LootStack**.Contains(****"Golden Key"****);** 
```

如果您需要将堆栈的元素复制到数组中，`CopyTo`方法将允许您指定目标和复制操作的起始索引。当您需要在数组的特定位置插入堆栈元素时，这个功能非常有用。请注意，您要将堆栈元素复制到的数组必须已经存在：

```cs
// Creates a new array of the same length as LootStack
string[] CopiedLoot = new string[5]; 
/* 
Copies the LootStack elements into the new CopiedLoot array at index 0\. The index parameter can be set to any index where you want the copied elements to be stored
*/
LootStack**.CopyTo(copiedLoot,** **0****);** 
```

如果您需要将堆栈转换为数组，只需使用`ToArray()`方法。这种转换会从您的堆栈中创建一个新数组，这与`CopyTo()`方法不同，后者将堆栈元素复制到现有数组中：

```cs
// Copies an existing stack to a new array
LootStack.ToArray(); 
```

您可以在 C#文档中找到完整的堆栈方法列表[`docs.microsoft.com/dotnet/api/system.collections.generic.stack-1?view=netcore-3.1`](https://docs.microsoft.com/dotnet/api/system.collections.generic.stack-1?view=netcore-3.1)。

这就结束了我们对堆栈的介绍，但是我们将在下一节中讨论它的堂兄，队列。

# 使用队列

与堆栈一样，队列是相同类型的元素或对象的集合。任何队列的长度都是可变的，就像堆栈一样，这意味着随着元素的添加或移除，其大小会发生变化。但是，队列遵循**先进先出**（**FIFO**）模型，这意味着队列中的第一个元素是第一个可访问的元素。您应该注意，队列可以存储`null`和重复的值，但在创建时不能用元素初始化。本节中的代码仅用于示例目的，不包括在我们的游戏中。

队列变量声明需要具备以下内容：

+   `Queue`关键字，其元素类型在左右箭头字符之间，以及一个唯一名称

+   使用`new`关键字在内存中初始化队列，然后是`Queue`关键字和箭头字符之间的元素类型

+   一对括号，以分号结束

以蓝图形式，队列如下所示：

```cs
Queue<elementType> name = new Queue<elementType>(); 
```

C#支持队列类型的非泛型版本，无需定义存储的元素类型：

```cs
Queue myQueue = new Queue(); 
```

但是，这比使用前面的泛型版本更不安全且更昂贵。您可以在[`github.com/dotnet/platform-compat/blob/master/docs/DE0006.md`](https://github.com/dotnet/platform-compat/blob/master/docs/DE0006.md)上阅读有关 Microsoft 建议的更多信息。

一个空的队列本身并不那么有用；您希望能够在需要时添加、移除和查看其元素，这是下一节的主题。

## 添加、移除和查看

由于前几节中的`LootStack`变量很容易成为队列，我们将保持以下代码不包含在游戏脚本中以提高效率。但是，您可以自由地探索这些类在您自己的代码中的差异或相似之处。

要创建一个字符串元素的队列，请使用以下方法：

```cs
// Creates a new Queue of string values.
Queue<string> activePlayers = new Queue<string>(); 
```

要向队列添加元素，请使用`Enqueue`方法并提供要添加的元素：

```cs
// Adds string values to the end of the Queue.
activePlayers**.Enqueue(****"Harrison"****);**
activePlayers**.Enqueue(****"Alex"****);**
activePlayers**.Enqueue(****"Haley"****);** 
```

要查看队列中的第一个元素而不移除它，请使用`Peek`方法：

```cs
// Returns the first element in the Queue without removing it.
var firstPlayer = activePlayers**.Peek();** 
```

要返回并移除队列中的第一个元素，请使用`Dequeue`方法：

```cs
// Returns and removes the first element in the Queue.
var firstPlayer = activePlayers**.Dequeue();** 
```

现在您已经了解了如何使用队列的基本特性，请随意探索队列类提供的更中级和高级方法。

## 常见方法

队列和堆栈几乎具有完全相同的特性，因此我们不会再次介绍它们。您可以在 C#文档中找到完整的方法和属性列表[`docs.microsoft.com/dotnet/api/system.collections.generic.queue-1?view=netcore-3.1`](https://docs.microsoft.com/dotnet/api/system.collections.generic.queue-1?view=netcore-3.1)。

在结束本章之前，让我们来看看 HashSet 集合类型及其独特适用的数学运算。

# 使用 HashSets

本章中我们将接触的最后一个集合类型是 HashSet。这个集合与我们遇到的任何其他集合类型都非常不同：它不能存储重复的值，也不是排序的，这意味着它的元素没有以任何方式排序。将 HashSets 视为只有键而不是键值对的字典。

它们可以执行集合操作和元素查找非常快，我们将在本节末尾进行探讨，并且最适合元素顺序和唯一性是首要考虑的情况。

HashSet 变量声明需要满足以下要求：

+   `HashSet`关键字，其元素类型在左右箭头字符之间，以及一个唯一名称

+   使用`new`关键字在内存中初始化 HashSet，然后是`HashSet`关键字和箭头字符之间的元素类型

+   由分号结束的一对括号

在蓝图形式中，它看起来如下：

```cs
HashSet<elementType> name = new HashSet<elementType>(); 
```

与栈和队列不同，你可以在声明变量时使用默认值初始化 HashSet：

```cs
HashSet<string> people = new HashSet<string>();
// OR
HashSet<string> people = new HashSet<string>() { "Joe", "Joan", "Hank"}; 
```

添加元素时，使用`Add`方法并指定新元素：

```cs
people**.Add(****"Walter"****);**
people**.Add(****"Evelyn"****);** 
```

要删除一个元素，调用`Remove`并指定你想要从 HashSet 中删除的元素：

```cs
people**.Remove(****"Joe"****);** 
```

这就是简单的内容了，在你的编程之旅中，这一点应该开始感觉相当熟悉了。集合操作是 HashSet 集合真正发光的地方，这是接下来章节的主题。

## 执行操作

集合操作需要两样东西：一个调用集合对象和一个传入的集合对象。

调用集合对象是你想要根据使用的操作修改的 HashSet，而传入的集合对象是由集合操作进行比较使用的。我们将在接下来的代码中详细介绍这一点，但首先，让我们先了解一下在编程场景中最常见的三种主要集合操作。

在以下定义中，`currentSet`指的是调用操作方法的 HashSet，而`specifiedSet`指的是传入的 HashSet 方法参数。修改后的 HashSet 始终是当前集合：

```cs
currentSet.Operation(specifiedSet); 
```

在接下来的这一部分，我们将使用三种主要操作：

+   `UnionWith`将当前集合和指定集合的元素添加在一起。

+   `IntersectWith`仅存储当前集合和指定集合中都存在的元素

+   `ExceptWith`从当前集合中减去指定集合的元素

还有两组处理子集和超集计算的集合操作，但这些针对特定用例，超出了本章的范围。你可以在[`docs.microsoft.com/dotnet/api/system.collections.generic.hashset-1?view=netcore-3.1`](https://docs.microsoft.com/dotnet/api/system.collections.generic.hashset-1?view=netcore-3.1)找到所有这些方法的相关信息。

假设我们有两组玩家名称的集合——一个是活跃玩家的集合，另一个是非活跃玩家的集合：

```cs
HashSet<string> activePlayers = new HashSet<string>() { "Harrison", "Alex", "Haley"};
HashSet<string> inactivePlayers = new HashSet<string>() { "Kelsey", "Basel"}; 
```

我们将使用`UnionWith()`操作来修改一个集合，以包括两个集合中的所有元素：

```cs
activePlayers.UnionWith(inactivePlayers);
/* activePlayers now stores "Harrison", "Alex", "Haley", "Kelsey", "Basel"*/ 
```

现在，假设我们有两个不同的集合——一个是活跃玩家的集合，另一个是高级玩家的集合：

```cs
HashSet<string> activePlayers = new HashSet<string>() { "Harrison", "Alex", "Haley"};
HashSet<string> premiumPlayers = new HashSet<string>() { "Haley", "Basel"}; 
```

我们将使用`IntersectWith()`操作来查找任何既是活跃玩家又是高级会员的玩家：

```cs
activePlayers.IntersectWith(premiumPlayers);
// activePlayers now stores only "Haley" 
```

如果我们想找到所有活跃玩家中不是高级会员的玩家怎么办？我们将通过调用`ExceptWith`来执行与`IntersectWith()`操作相反的操作：

```cs
HashSet<string> activePlayers = new HashSet<string>() { "Harrison", "Alex", "Haley"};
HashSet<string> premiumPlayers = new HashSet<string>() { "Haley",
  "Basel"};
activePlayers.ExceptWith(premiumPlayers);
// activePlayers now stores "Harrison" and "Alex" but removed "Haley" 
```

请注意，我在每个操作中使用了两个示例集合的全新实例，因为当前集合在执行每个操作后都会被修改。如果你一直使用相同的集合，你会得到不同的结果。

现在你已经学会了如何使用 HashSets 执行快速数学运算，是时候结束我们的章节，总结我们所学到的知识了。

# 中间集合总结

在你继续阅读总结和下一章之前，让我们再次强调一些我们刚刚学到的关键点。有时，与我们正在构建的实际游戏原型不总是一对一关系的主题需要额外的关注。

在这一点上，我确定你会问自己一个问题：为什么在任何情况下都要使用这些其他集合类型，而不是只使用列表呢？这是一个完全合理的问题。简单的答案是，当在正确的情况下应用时，栈、队列和 HashSets 比列表提供更好的性能。例如，当你需要按特定顺序存储项目并按特定顺序访问它们时，栈比列表更有效。

更复杂的答案是，使用不同的集合类型会强制规定您的代码如何与它们及其元素进行交互。这是良好代码设计的标志，因为它消除了您计划如何使用集合的任何歧义。到处都是列表，当您不记得要求它们执行什么功能时，事情就会变得混乱。

与本书中学到的一切一样，最好始终使用合适的工具来完成手头的工作。更重要的是，您需要有不同的工具可供选择。

# 摘要

恭喜，您几乎到达终点了！在本章中，您了解了三种新的集合类型，以及它们在不同情况下的用法。

如果您想以添加顺序的相反顺序访问集合元素，则堆栈非常适合，如果您想以顺序顺序访问元素，则队列是您的选择，两者都非常适合临时存储。这些集合类型与列表或数组之间的重要区别在于它们如何通过弹出和查看操作进行访问。最后，您了解了强大的 HashSet 及其基于性能的数学集合操作。在需要处理唯一值并对大型集合执行添加、比较或减法操作的情况下，这些是关键。

在下一章中，您将深入了解 C#的中级世界，包括委托、泛型等，因为您接近本书的结尾。即使您已经学到了所有知识，最后一页仍然只是另一段旅程的开始。

# 中级集合小测验

1.  哪种集合类型使用 LIFO 模型存储其元素？

1.  哪种方法让您查询堆栈中的下一个元素而不移除它？

1.  堆栈和队列能存储`null`值吗？

1.  如何从一个 HashSet 中减去另一个 HashSet？

# 加入我们的 Discord！

与其他用户、Unity/C#专家和 Harrison Ferrone 一起阅读本书。提出问题，为其他读者提供解决方案，通过*问我任何事*会话与作者交流等等。

立即加入！

[`packt.link/csharpunity2021`](https://packt.link/csharpunity2021)

![](img/QR_Code_9781801813945.png)
