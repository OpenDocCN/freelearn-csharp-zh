# 4. 数据结构和 LINQ

概述

在本章中，你将学习 C#中主要集合及其主要用法。然后，你将了解如何使用语言集成查询（LINQ）通过高效且简洁的代码查询内存中的集合。到本章结束时，你将熟练掌握使用 LINQ 进行排序、过滤和聚合数据等操作。

# 简介

在前面的章节中，你已经使用了引用单个值的变量，例如`string`和`double`系统类型、系统`class`实例以及你自己的类实例。.NET 有多种数据结构可以用来存储多个值。这些结构通常被称为集合。本章通过介绍来自`System.Collections.Generic`命名空间中的集合类型来扩展这一概念。

你可以使用集合类型创建可以存储多个对象引用的变量。这些集合包括可以调整大小以容纳元素数量的列表和提供使用唯一键作为标识符访问元素的字典。例如，你可能需要使用代码作为唯一标识符来存储国际电话区号列表。在这种情况下，你需要确保相同的电话区号不会被添加到集合中两次。

这些集合的实例化方式与其他类类似，并且在大多数应用程序中被广泛使用。选择正确的集合类型主要取决于你打算如何添加项目以及你希望如何访问这些项目。常用的集合类型包括`List`、`Set`和`HashSet`，你将在稍后详细学习。

LINQ 是一种提供表达性和简洁语法的查询对象技术。使用类似 SQL 的语言或如果你更喜欢，一组可以串联在一起的扩展方法，可以消除围绕过滤、排序和分组对象的许多复杂性，从而可以轻松枚举生成的集合。

# 数据结构

.NET 提供了各种内置数据结构类型，如`Array`、`List`和`Dictionary`类型。所有数据结构的核心是`IEnumerable`和`ICollection`接口。实现这些接口的类提供了一种遍历单个元素和操作其项的方法。很少需要创建直接从这些接口派生的自己的类，因为所有必需的功能都由内置的集合类型提供，但了解关键属性是值得的，因为它们在.NET 中被广泛使用。

每种集合类型的泛型版本需要一个单一的类型参数，该参数定义了可以添加到集合中的元素类型，使用泛型类型的标准`<T>`语法。

`IEnumerable` 接口有一个属性，即 `GetEnumerator<T>()`。此属性返回一个类型，它提供了允许调用者遍历集合中元素的方法。您无需直接调用 `GetEnumerator()` 方法，因为编译器会在您使用 `foreach` 语句（例如 `foreach(var book in books)`）时自动调用它。您将在接下来的章节中了解更多关于如何使用它的信息。

`ICollection` 接口具有以下属性：

+   `int Count { get; }`: 返回集合中的项目数量。

+   `bool IsReadOnly { get; }`: 表示集合是否为只读。某些集合可以被标记为只读，以防止调用者向集合中添加、删除或移动元素。C# 不会阻止您修改只读集合中单个项目的属性。

+   `void Add(T item)`: 将类型为 `<T>` 的项目添加到集合中。

+   `void Clear()`: 从集合中删除所有项目。

+   `bool Contains(T item)`: 如果集合包含特定值，则返回 `true`。根据集合中项目类型的不同，这可以是值相等，其中对象基于其成员相似，或者引用相等，其中对象指向相同的内存位置。

+   `void CopyTo(T[] array, int arrayIndex)`: 将集合中的每个元素复制到目标数组中，从指定索引位置的第一个元素开始。如果您需要从集合的开始跳过特定数量的元素，这可能很有用。

+   `bool Remove(T item)`: 从集合中删除指定的项目。如果有多个实例，则只删除第一个实例。如果成功删除项目，则返回 `true`。

`IEnumerable` 和 `ICollection` 是所有集合都实现的接口：

![图 4.1: ICollection 和 IEnumerable 类图](img/B16835_04_01.jpg)

图 4.1: ICollection 和 IEnumerable 类图

根据在集合内如何访问元素，某些集合实现了进一步的接口。

`IList` 接口用于可以按索引位置访问的集合，从零开始。因此，对于包含两个项目“红色”和“蓝色”的列表，索引零的元素是“红色”，索引一的元素是“蓝色”。

![图 4.2: IList 类图](img/B16835_04_02.jpg)

图 4.2: IList 类图

`IList` 接口具有以下属性：

+   `T this[int index] { get; set; }`: 获取或设置指定索引位置上的元素。

+   `int Add(T item)`: 添加指定的项目并返回该项目在列表中的索引位置。

+   `void Clear()`: 从列表中删除所有项目。

+   `bool Contains(T item)`: 如果列表包含指定的项目，则返回 `true`。

+   `int IndexOf(T item)`: 返回项目的索引位置，如果未找到则返回 `-1`。

+   `void Insert(int index, T item)`: 在指定的索引位置插入项目。

+   `void Remove(T item)`: 如果列表中存在该项目，则将其删除。

+   `void RemoveAt(int index)`: 删除指定索引位置的项目。

您现在已经看到了集合的常见主要接口。因此，现在您将了解可用的主要集合类型以及它们的使用方法。

## 列表

`List<T>` 类型是 C# 中最广泛使用的集合之一。当您有一组项目并希望使用它们的索引位置来控制项目顺序时，它会使用。它实现了 `IList` 接口，允许使用索引位置插入、访问或删除项目：

![图 4.3：List 类图](img/B16835_04_03.jpg)

图 4.3：List 类图

列表具有以下行为：

+   可以在集合的任何位置插入项目。任何尾随项目都将增加它们的索引位置。

+   可以通过索引或值移除项目。这将更新尾随项目的索引位置。

+   可以使用它们的索引值设置项目。

+   可以将项目添加到集合的末尾。

+   可以在集合内重复项目。

+   可以使用各种 `Sort` 方法对项目的位置进行排序。

列表的一个例子可能是网页浏览器应用程序中的标签页。通常，用户可能希望在其他标签页之间拖动浏览器标签页，在末尾打开新的标签页，或在标签页列表的任何位置关闭标签页。可以使用 `List` 实现控制这些操作的代码。

内部，`List` 维护一个数组来存储其对象。当在末尾添加项目时，这可能很高效，但在插入项目时可能效率不高，尤其是在列表的起始位置附近，因为需要重新计算项目的索引位置。

以下示例展示了如何使用泛型 `List` 类。代码使用了 `List<string>` 类型参数，这允许将 `string` 类型添加到列表中。尝试添加任何其他类型将导致编译器错误。这将展示 `List` 类的多种常用方法。

1.  在您的源代码文件夹中创建一个名为 `Chapter04` 的新文件夹。

1.  切换到 `Chapter04` 文件夹并创建一个新的控制台应用程序，名为 `Chapter04`，使用以下 .NET 命令：

    ```cs
    source\Chapter04>dotnet new console -o Chapter04
    The template "Console Application" was created successfully.
    ```

1.  删除 `Class1.cs` 文件。

1.  添加一个名为 `Examples` 的新文件夹。

1.  添加一个名为 `ListExamples.cs` 的新类文件。

1.  将 `System.Collections.Generic` 命名空间添加到访问 `List<T>` 类并声明一个名为 `colors` 的新变量：

    ```cs
    using System;
    using System.Collections.Generic;
    namespace Chapter04.Examples
    {
        class ListExamples
        {     
            public static void Main()
            {
                var colors = new List<string> {"red", "green"};
                colors.Add("orange");
    ```

代码声明了新的 `colors` 变量，该变量可以存储多个颜色名称作为 `strings`。在这里，使用了集合初始化语法，以便将 `red` 和 `green` 添加为变量初始化的一部分。调用 `Add` 方法，将 `orange` 添加到列表中。

1.  类似地，`AddRange` 将 `yellow` 和 `pink` 添加到列表的末尾：

    ```cs
                colors.AddRange(new [] {"yellow", "pink"});
    ```

1.  到目前为止，列表中有五个颜色，`red` 在索引位置 `0`，`green` 在位置 `1`。您可以使用以下代码进行验证：

    ```cs
                Console.WriteLine($"Colors has {colors.Count} items");
                Console.WriteLine($"Item at index 1 is {colors[1]}");
    ```

运行代码会产生以下输出：

```cs
Colors has 5 items
Item at index 1 is green
```

1.  使用`Insert`，可以将`blue`插入列表的开头，即索引`0`，如下面的代码所示。请注意，这将`red`从索引`0`移动到`1`，其他所有颜色的索引都将增加一个：

    ```cs
                Console.WriteLine("Inserting blue at 0");
                colors.Insert(0, "blue");
                Console.WriteLine($"Item at index 1 is now {colors[1]}");
    ```

运行此代码后，你应该看到以下输出：

```cs
Inserting blue at 0
Item at index 1 is now red
```

1.  使用`foreach`可以遍历列表中的字符串，如下将每个字符串写入控制台：

    ```cs
                Console.WriteLine("foreach");
                foreach (var color in colors)
                    Console.Write($"{color}|");
                Console.WriteLine();
    ```

你应该得到以下输出：

```cs
foreach
blue|red|green|orange|yellow|pink|
```

1.  现在，添加以下代码以反转数组。在这里，每个`color`字符串都使用`ToCharArray`转换为`char`类型的数组：

    ```cs
                Console.WriteLine("ForEach Action:");
                colors.ForEach(color =>
                {
                    var characters = color.ToCharArray();
                    Array.Reverse(characters);
                    var reversed = new string(characters);
                    Console.Write($"{reversed}|");
                });
                Console.WriteLine();
    ```

这不会影响`colors`列表中的任何值，因为`characters`引用的是不同的对象。请注意，`foreach`遍历每个字符串，而`ForEach`定义一个 Action 委托，用于调用每个字符串（回想一下，在*第三章*，*委托、事件和 Lambda*中，你看到了如何使用 Lambda 语句创建`Action`委托）。

1.  运行代码会导致以下输出：

    ```cs
    ForEach Action:
    eulb|der|neerg|egnaro|wolley|knip|
    ```

1.  在下一个代码片段中，`List`构造函数接受一个源集合。这创建了一个包含`colors`字符串副本的新列表，在这种情况下，它使用默认的`Sort`实现进行排序：

    ```cs
                var backupColors = new List<string>(colors);
                backupColors.Sort();
    ```

字符串类型使用值类型语义，这意味着`backupColors`列表填充了每个源字符串值的**副本**。更新一个列表中的字符串将**不会**影响另一个列表。相反，类被定义为引用类型，因此将类实例的列表传递给构造函数仍将创建一个新的列表，具有独立的元素索引，但每个元素将指向内存中相同的共享引用而不是独立的副本。

1.  在以下代码片段中，在移除所有颜色（使用`colors.Clear`）之前，每个值都会写入控制台（列表将很快被重新填充）：

    ```cs
                Console.WriteLine("Foreach before clearing:");
                foreach (var color in colors)
                    Console.Write($"{color}|");
                Console.WriteLine();
                colors.Clear();
                Console.WriteLine($"Colors has {colors.Count} items");
    ```

运行代码会产生以下输出：

```cs
Foreach before clearing:
blue|red|green|orange|yellow|pink|
Colors has 0 items
```

1.  然后，再次使用`AddRange`，将完整的颜色列表添加回`colors`列表，使用排序后的`backupColors`项作为源：

    ```cs
                colors.AddRange(backupColors);
                Console.WriteLine("foreach after addrange (sorted items):");
                foreach (var color in colors)
                    Console.Write($"{color}|");
                Console.WriteLine();
    ```

你应该看到以下输出：

```cs
foreach after addrange (sorted items):
blue|green|orange|pink|red|yellow|
```

1.  `ConvertAll`方法传递一个委托，可以用来返回任何类型的新列表：

    ```cs
                var indexes = colors.ConvertAll(color =>                      $"{color} is at index {colors.IndexOf(color)}");
                Console.WriteLine("ConvertAll:");
                Console.WriteLine(string.Join(Environment.NewLine, indexes));
    ```

在这里，返回一个新的`List<string>`，其中每个项目都使用其值和列表中的索引进行格式化。正如预期的那样，运行代码会产生以下输出：

```cs
ConvertAll:
blue is at index 0
green is at index 1
orange is at index 2
pink is at index 3
red is at index 4
yellow is at index 5
```

1.  在下一个代码片段中，使用了两个`Contains()`方法来展示字符串值相等性的实际应用：

    ```cs
                Console.WriteLine($"Contains RED: {colors.Contains("RED")}");
                Console.WriteLine($"Contains red: {colors.Contains("red")}");
    ```

注意，大写的`RED`是`red`。运行代码会产生以下输出：

```cs
Contains RED: False
Contains red: True
```

1.  现在，添加以下代码片段：

    ```cs
                var existsInk = colors.Exists(color => color.EndsWith("ink"));
                Console.WriteLine($"Exists *ink: {existsInk}");
    ```

在这里，`Exists`方法传递一个 Predicate 委托，如果满足测试条件，则返回`True`或`False`。Predicate 是一个内置委托，它返回一个布尔值。在这种情况下，如果任何项的字符串值以字母`ink`结尾（例如`pink`），则返回`True`。

你应该看到以下输出：

```cs
Exists *ink: True
```

1.  你知道已经有 `red` 颜色了，但如果你在列表的非常开始处插入 `red`，两次，将会很有趣：

    ```cs
                Console.WriteLine("Inserting reds");
                colors.InsertRange(0, new [] {"red", "red"});
                foreach (var color in colors)
                    Console.Write($"{color}|");
                Console.WriteLine();
    ```

你将得到以下输出：

```cs
Inserting reds
red|red|blue|green|orange|pink|red|yellow|
```

这表明可以在列表中插入相同的项多次。

1.  下一个片段显示了如何使用 `FindAll` 方法。`FindAll` 方法与 `Exists` 方法类似，因为它传递了一个 `Predicate` 条件。所有符合该规则的项目都将被返回。添加以下代码：

    ```cs
                var allReds = colors.FindAll(color => color == "red");
                Console.WriteLine($"Found {allReds.Count} red");
    ```

你应该得到以下输出。正如预期的那样，返回了三个 `red` 项：

```cs
Found 3 red
```

1.  完成示例后，使用 `Remove` 方法从列表中删除第一个 `red`。还有两个 `FindLastIndex` 来获取最后一个 `red` 项的索引：

    ```cs
                colors.Remove("red");
                var lastRedIndex = colors.FindLastIndex(color => color == "red");
                Console.WriteLine($"Last red found at index {lastRedIndex}");
                Console.ReadLine();
            }
        }
    }
    ```

运行代码产生以下输出：

```cs
Last red found at index 5
```

注意

你可以在 [`packt.link/dLbK6`](https://packt.link/dLbK6) 找到用于此示例的代码。

在了解了如何使用泛型 `List` 类之后，现在是时候做练习了。

## 练习 4.01：在列表中维护秩序

在本章开头，网络浏览器标签被描述为一个理想的列表示例。在这个练习中，你将把这个想法付诸实践，并创建一个控制模拟网络浏览器的应用程序中标签导航的类。

为了做到这一点，你将创建一个 `Tab` 类和一个 `TabController` 应用程序，允许打开新的标签和关闭或移动现有的标签。以下步骤将帮助你完成这个练习：

1.  在 VSCode 中，选择你的 `Chapter04` 项目。

1.  添加一个名为 `Exercises` 的新文件夹。

1.  在 `Exercises` 文件夹内，添加一个名为 `Exercise01` 的文件夹，并添加一个名为 `Exercise01.cs` 的文件。

1.  打开 `Exercise01.cs` 文件并定义一个带有字符串 URL 构造参数的 `Tab` 类，如下所示：

    ```cs
    using System;
    using System.Collections;
    using System.Collections.Generic;
    namespace Chapter04.Exercises.Exercise01
    {
        public class Tab 
        {
            public Tab()
            {}
            public Tab(string url) => (Url) = (url);
            public string Url { get; set; }
            public override string ToString() => Url;
        }   
    ```

在这里，`ToString` 方法已被重写以返回当前 URL，以帮助在控制台记录详细信息。

1.  按照以下方式创建 `TabController` 类：

    ```cs
        public class TabController : IEnumerable<Tab>
        {
            private readonly List<Tab> _tabs = new();
    ```

`TabController` 类包含一个标签列表。注意这个类是如何从 `IEnumerable` 接口继承的。这个接口被用来使类提供一种遍历其项的方法，使用 `foreach` 语句。你将在下一步提供打开、移动和关闭标签的方法，这些方法将直接控制 `_tabs` 列表中项的顺序。请注意，你可以直接将 `_tabs` 列表暴露给调用者，但最好是限制通过你自己的方法访问标签。因此，它被定义为 `readonly` 列表。

1.  接下来，定义 `OpenNew` 方法，它将新标签添加到列表的末尾：

    ```cs
            public Tab OpenNew(string url)
            {
                var tab = new Tab(url);
                _tabs.Add(tab);
                Console.WriteLine($"OpenNew {tab}");
                return tab;
            }
    ```

1.  定义另一个方法 `Close`，如果标签存在，则从列表中删除它。为此添加以下代码：

    ```cs
            public void Close(Tab tab)
            {
                if (_tabs.Remove(tab))
                {
                    Console.WriteLine($"Removed {tab}");
                }
            }
    ```

1.  要将标签移动到列表的起始位置，添加以下代码：

    ```cs
            public void MoveToStart(Tab tab)
            {
                if (_tabs.Remove(tab))
                {
                    _tabs.Insert(0, tab);
                    Console.WriteLine($"Moved {tab} to start");
                }
    ```

在这里，`MoveToStart` 将尝试删除标签并将其插入索引 `0`。

1.  类似地，添加以下代码以将标签移动到末尾：

    ```cs
            public void MoveToEnd(Tab tab)
            {
                if (_tabs.Remove(tab))
                {
                    _tabs.Add(tab);
                    Console.WriteLine($"Moved {tab} to end. Index={_tabs.IndexOf(tab)}");
                }
            }
    ```

在这里，调用`MoveToEnd`首先删除标签页，然后将其添加到末尾，并将新的索引位置记录到控制台。

最后，`IEnumerable`接口要求您实现两个方法，`IEnumerator<Tab> GetEnumerator()`和`IEnumerable.GetEnumerator()`。这些允许调用者使用类型为`Tab`的泛型或使用第二个方法通过基于对象的类型迭代集合。第二个方法是对 C#早期版本的回顾，但需要为了兼容性。

1.  对于两种方法的实际结果，您可以使用`_tab`列表的`GetEnumerator`方法，因为该列表包含标签页的列表形式。添加以下代码以实现此目的：

    ```cs
            public IEnumerator<Tab> GetEnumerator() => _tabs.GetEnumerator();
            IEnumerator IEnumerable.GetEnumerator() => _tabs.GetEnumerator();
        }
    ```

1.  您现在可以创建一个控制台应用程序来测试控制器的行为。首先打开三个新的标签页，并通过`LogTabs`记录标签页详情（这将在稍后定义）：

    ```cs
        static class Program
        {
            public static void Main()
            {
                var controller = new TabController();
                Console.WriteLine("Opening tabs...");
                var packt = controller.OpenNew("packtpub.com");
                var msoft = controller.OpenNew("microsoft.com");
                var amazon = controller.OpenNew("amazon.com");
                controller.LogTabs();
    ```

1.  现在，将`amazon`移动到开头，将`packt`移动到末尾，并记录标签页详情：

    ```cs
                Console.WriteLine("Moving...");
                controller.MoveToStart(amazon);
                controller.MoveToEnd(packt);
                controller.LogTabs();
    ```

1.  关闭`msoft`标签页并再次记录详情：

    ```cs
                Console.WriteLine("Closing tab...");
                controller.Close(msoft);
                controller.LogTabs();
                Console.ReadLine();
            }
    ```

1.  最后，添加一个扩展方法，帮助在`TabController`中记录每个标签页的 URL。将其定义为`IEnumerable<Tab>`的扩展方法，而不是`TabController`，因为您只需要一个迭代器来使用`foreach`循环遍历标签页。

1.  使用`PadRight`将每个 URL 左对齐，如下所示：

    ```cs
            private static void LogTabs(this IEnumerable<Tab> tabs)
            {
                Console.Write("TABS: |");
                foreach(var tab in tabs)
                    Console.Write($"{tab.Url.PadRight(15)}|");
                Console.WriteLine();
            }    
       } 
    }
    ```

1.  运行代码将产生以下输出：

    ```cs
    Opening tabs...
    OpenNew packtpub.com
    OpenNew microsoft.com
    OpenNew amazon.com
    TABS: |packtpub.com   |microsoft.com  |amazon.com     |
    Moving...
    Moved amazon.com to start
    Moved packtpub.com to end. Index=2
    TABS: |amazon.com     |microsoft.com  |packtpub.com   |
    Closing tab...
    Removed microsoft.com
    TABS: |amazon.com     |packtpub.com   |
    ```

    注意

    有时，Visual Studio 可能会在您第一次执行程序时报告不可为空的属性错误。这是一个有用的提醒，表明您正在尝试使用可能在运行时具有 null 值的字符串值。

打开了三个标签页。然后`amazon.com`和`packtpub.com`被移动到`microsoft.com`之前，最后关闭并从标签页列表中删除。

注意

您可以在[`packt.link/iUcIs`](https://packt.link/iUcIs)找到用于此练习的代码。

在这个练习中，您已经看到了如何使用列表来存储相同类型的多个项目，同时保持项目的顺序。下一节将介绍`Queue`和`Stack`类，这些类允许以预定义的顺序添加和删除项目。

## 队列

`Queue`类提供了一种先进先出机制。项目使用`Enqueue`方法添加到队列的末尾，并使用`Dequeue`方法从队列的前端移除。队列中的项目不能通过索引元素访问。

队列通常用于需要确保项目按它们添加到队列的顺序处理的流程。一个典型的例子可能是一个繁忙的在线票务系统，向客户销售有限数量的音乐会门票。为了确保公平性，客户在登录时立即被添加到排队系统中。然后系统将逐个出队每个客户并处理每个订单，直到所有门票售罄或客户队列变空。

以下示例创建了一个包含五个 `CustomerOrder` 记录的队列。当是时候处理订单时，每个订单都会使用 `TryDequeue` 方法出队，该方法将返回 `true`，直到所有订单都已被处理。客户订单的处理顺序与它们被添加的顺序相同。如果请求的票数多于或等于剩余的票数，则显示成功消息。如果剩余的票数少于请求的数量，则显示道歉信息。

![图 4.4：队列的 Enqueue() 和 Dequeue() 工作流程](img/B16835_04_04.jpg)

图 4.4：队列的 Enqueue() 和 Dequeue() 工作流程

完成此示例的步骤如下：

1.  在您的 `Chapter04` 源文件夹的 `Examples` 文件夹中，添加一个名为 `QueueExamples.cs` 的新类，并按以下方式编辑它：

    ```cs
    using System;
    using System.Collections.Generic;
    namespace Chapter04.Examples
    {
        class QueueExamples
        {      
            record CustomerOrder (string Name, int TicketsRequested)
            {}
            public static void Main()
            {
                var ticketsAvailable = 10;
                var customers = new Queue<CustomerOrder>();
    ```

1.  使用 `Enqueue` 方法添加五个订单，如下所示：

    ```cs
                customers.Enqueue(new CustomerOrder("Dave", 2));
                customers.Enqueue(new CustomerOrder("Siva", 4));
                customers.Enqueue(new CustomerOrder("Julien", 3));
                customers.Enqueue(new CustomerOrder("Kane", 2));
                customers.Enqueue(new CustomerOrder("Ann", 1));
    ```

1.  现在，使用一个 `while` 循环，直到 `TryDequeue` 返回 `false`，这意味着所有当前订单都已处理：

    ```cs
                // Start processing orders...
                while(customers.TryDequeue(out CustomerOrder nextOrder))
                {
                    if (nextOrder.TicketsRequested <= ticketsAvailable)
                    {
                        ticketsAvailable -= nextOrder.TicketsRequested;   
                        Console.WriteLine($"Congratulations {nextOrder.Name}, you've purchased {nextOrder.TicketsRequested} ticket(s)");
                    }
                    else
                    {
                        Console.WriteLine($"Sorry {nextOrder.Name}, cannot fulfil {nextOrder.TicketsRequested} ticket(s)");
                    }
                }
                Console.WriteLine($"Finished. Available={ticketsAvailable}");
                Console.ReadLine();
            }
        }
    }
    ```

1.  运行示例代码会产生以下输出：

    ```cs
    Congratulations Dave, you've purchased 2 ticket(s)
    Congratulations Siva, you've purchased 4 ticket(s)
    Congratulations Julien, you've purchased 3 ticket(s)
    Sorry Kane, cannot fulfil 2 ticket(s)
    Congratulations Ann, you've purchased 1 ticket(s)
    Finished. Available=0
    ```

    注意

    第一次运行此程序时，Visual Studio 可能会显示一个不可为 null 的类型错误。这个错误是一个提醒，说明您正在使用可能为 null 值的变量。

输出显示`Dave`请求了两张票。由于有两张或更多票可用，他成功了。`Siva` 和 `Julien` 也成功了，但到 `Kane` 下单两张票时，只剩下一张票了，所以他看到了道歉信息。最后，`Ann`请求了一张票，并成功下单。

注意

您可以在 [`packt.link/Zb524`](https://packt.link/Zb524) 找到用于此示例的代码。

## 栈

`Stack` 类提供了与 `Queue` 类相反的机制；项目以后进先出的顺序处理。与 `Queue` 类一样，您不能通过索引位置访问元素。项目通过 `Push` 方法添加到栈中，通过 `Pop` 方法移除。

一个应用程序的“撤销”菜单可以使用栈来实现。例如，在一个文字处理程序中，当用户编辑文档时，会创建一个“操作”代理，用户每次按下 `Ctrl` + `Z` 时，都可以撤销最近的更改。最近的操作会被从栈中弹出，并撤销更改。这允许撤销多个步骤。

![图 4.5：栈的 Push() 和 Pop() 工作流程](img/B16835_04_05.jpg)

图 4.5：栈的 Push() 和 Pop() 工作流程

以下示例展示了这一点。

您将首先创建一个支持多个撤销操作的 `UndoStack` 类。调用者决定每次调用 `Undo` 请求时应运行什么操作。

一个典型的可撤销操作是在用户添加单词之前存储文本的副本。另一个可撤销操作是在应用新字体之前存储当前字体的副本。您可以开始添加以下代码，其中您创建`UndoStack`类并定义一个名为`_undoStack`的只读`Action`委托`Stack`：

1.  在您的`Chapter04\Examples`文件夹中，添加一个名为`StackExamples.cs`的新类，并按以下方式编辑它：

    ```cs
    using System;
    using System.Collections.Generic;
    namespace Chapter04.Examples
    {
        class UndoStack
        {
            private readonly Stack<Action> _undoStack = new Stack<Action>();
    ```

1.  当用户完成某些操作后，可以撤销相同的操作。因此，将一个`Action`推送到`_undoStack`的前面：

    ```cs
            public void Do(Action action)
            {
                _undoStack.Push(action);
            }
    ```

1.  `Undo`方法检查是否有任何要撤销的项目，然后调用`Pop`来移除最近的`Action`并调用它，从而撤销刚刚应用的变化。以下是如何添加此代码的示例：

    ```cs
            public void Undo()
            {
                if (_undoStack.Count > 0)
                {
                    var undo = _undoStack.Pop();
                    undo?.Invoke();
                }
            }
        }
    ```

1.  现在，您可以创建一个`TextEditor`类，允许将编辑添加到`UndoStack`。此构造函数传递`UndoStack`，因为可能有多个编辑器需要将各种`Action`委托添加到堆栈中：

    ```cs
        class TextEditor
        {
            private readonly UndoStack _undoStack;
            public TextEditor(UndoStack undoStack)
            {
                _undoStack = undoStack;
            }
            public string Text {get; private set; }
    ```

1.  接下来，添加`EditText`命令，它复制`previousText`的值并创建一个`Action`委托，如果被调用，可以将文本恢复到其先前值：

    ```cs
            public void EditText(string newText)
            {
                var previousText = Text;
                _undoStack.Do( () =>
                {
                    Text = previousText;
                    Console.Write($"Undo:'{newText}'".PadRight(40));
                    Console.WriteLine($"Text='{Text}'");
                });
    ```

1.  现在，应使用`+=`运算符将`newText`值附加到`Text`属性上。有关此操作的详细信息记录到控制台，使用`PadRight`来改进格式：

    ```cs
                Text += newText;
                Console.Write($"Edit:'{newText}'".PadRight(40));
                Console.WriteLine($"Text='{Text}'");
            }
        }
    ```

1.  最后，是时候创建一个测试`TextEditor`和`UndoStack`的控制台应用程序了。最初进行四次编辑，然后进行两次**撤销操作**，最后再进行两次文本编辑：

    ```cs
        class StackExamples
        {

            public static void Main()
            {
                var undoStack = new UndoStack();
                var editor = new TextEditor(undoStack);
                editor.EditText("One day, ");
                editor.EditText("in a ");
                editor.EditText("city ");
                editor.EditText("near by ");
                undoStack.Undo(); // remove 'near by'
                undoStack.Undo(); // remove 'city'
                editor.EditText("land ");
                editor.EditText("far far away ");
                Console.ReadLine();
            }
        }    
    }
    ```

1.  运行控制台应用程序会产生以下输出：

    ```cs
    Edit:'One day, '                        Text='One day, '
    Edit:'in a '                            Text='One day, in a '
    Edit:'city '                            Text='One day, in a city '
    Edit:'near by '                         Text='One day, in a city near by '
    Undo:'near by '                         Text='One day, in a city '
    Undo:'city '                            Text='One day, in a '
    Edit:'land '                            Text='One day, in a land '
    Edit:'far far away '                    Text='One day, in a land far far away '
    ```

    注意

    Visual Studio 在代码首次执行时可能会显示不可为空的属性错误。这是因为 Visual Studio 注意到`Text`属性在运行时可能为 null 值，因此提供了改进代码的建议。

左侧输出显示了应用时的文本编辑和撤销操作，以及右侧的最终`Text`值。两次`Undo`调用导致`near by`和`city`从`Text`值中移除，然后`land`和`far far away`最终添加到`Text`值中。

注意

您可以在[`packt.link/tLVyf`](https://packt.link/tLVyf)找到用于此示例的代码。

## HashSets

`HashSet`类以高效且高性能的方式提供了数学集合操作，用于对象集合。`HashSet`不允许重复元素，并且项目不按任何特定顺序存储。使用`HashSet`类非常适合高性能操作，例如需要快速找到两个对象集合重叠的位置。

通常，`HashSet`用于以下操作：

+   `public void UnionWith(IEnumerable<T> other)`: 生成集合并集。这修改`HashSet`以包括当前`HashSet`实例中存在的项目、其他集合或两者。

+   `public void IntersectWith(IEnumerable<T> other)`: 产生一个集合交集。这修改`HashSet`以包含当前`HashSet`实例和其他集合中存在的项目。

+   `public void ExceptWith(IEnumerable<T> other)`: 产生一个集合减法。这从当前`HashSet`实例和其他集合中删除存在的项目。

`HashSet`在需要从**集合**中包含或排除某些元素时很有用。例如，考虑一个代理人管理各种名人，并被要求找到三组明星：

+   那些既能**行动**又能**唱歌**的人。

+   那些既能**行动**又能**唱歌**的人。

+   那些只能**行动**（不允许唱歌）的人。

在以下代码片段中，创建了一个演员和歌手名字的列表：

1.  在你的`Chapter04\Examples`文件夹中，添加一个名为`HashSetExamples.cs`的新类，并按照以下方式编辑它：

    ```cs
    using System;
    using System.Collections.Generic;
    namespace Chapter04.Examples
    {
        class HashSetExamples
        {
            public static void Main()
            {
                var actors = new List<string> {"Harrison Ford", "Will Smith", 
                                               "Sigourney Weaver"};
                var singers = new List<string> {"Will Smith", "Adele"};
    ```

1.  现在，创建一个新的只包含歌手的`HashSet`实例，然后使用`UnionWith`修改集合以包含既能**行动**又能**唱歌**的人的独特集合：

    ```cs
                var actingOrSinging = new HashSet<string>(singers);
                actingOrSinging.UnionWith(actors);
                Console.WriteLine($"Acting or Singing: {string.Join(", ", 
                                  actingOrSinging)}");
    ```

1.  对于那些可以**行动**的歌手`HashSet`实例，并使用`IntersectWith`修改`HashSet`实例以包含两个集合中都有的人的独特列表：

    ```cs
                var actingAndSinging = new HashSet<string>(singers);
                actingAndSinging.IntersectWith(actors);
                Console.WriteLine($"Acting and Singing: {string.Join(", ", 
                                  actingAndSinging)}");
    ```

1.  最后，对于那些可以`ExceptWith`从`HashSet`实例中删除也能唱歌的人：

    ```cs
                var actingOnly = new HashSet<string>(actors);
                actingOnly.ExceptWith(singers);
                Console.WriteLine($"Acting Only: {string.Join(", ", actingOnly)}");
                Console.ReadLine();
            }
        }
    }
    ```

1.  运行控制台应用程序会产生以下输出：

    ```cs
    Acting or Singing: Will Smith, Adele, Harrison Ford, Sigourney Weaver
    Acting and Singing: Will Smith
    Acting Only: Harrison Ford, Sigourney Weaver
    ```

从输出中，你可以看到在给定的演员和歌手列表中，只有`Will Smith`既能**行动**又能**唱歌**。

注意

你可以在[`packt.link/ZdNbS`](https://packt.link/ZdNbS)找到这个示例使用的代码。

## 字典

另一个常用的集合类型是泛型`Dictionary<TK, TV>`。这允许添加多个项目，但需要一个独特的**键**来识别项目实例。

字典通常用于使用已知键查找值。键和值类型参数可以是任何类型。一个值可以在`Dictionary`中存在多次，前提是其键是**唯一的**。尝试添加已存在的键将导致抛出运行时异常。

`Dictionary`的一个常见例子可能是按 ISO 国家代码键控的已知国家注册表。客户服务应用程序可以从数据库中加载客户详细信息，然后使用 ISO 代码从国家列表中查找客户的国籍，而不是为每个客户创建一个新的国家实例。

注意

你可以在[`www.iso.org/iso-3166-country-codes.xhtml`](https://www.iso.org/iso-3166-country-codes.xhtml)找到有关标准 ISO 国家代码的更多信息。

`Dictionary`类中使用的**主要方法**如下：

+   `public TValue this[TKey key] {get; set;}`：获取或设置与键关联的值。如果键不存在，则抛出异常。

+   `Dictionary<TKey, TValue>.KeyCollection Keys { get; }`：返回一个包含所有键的`KeyCollection`字典实例。

+   `Dictionary<TKey, TValue>.ValueCollection Values { get; }`: 返回一个包含所有值的 `ValueCollection` 字典实例。

+   `public int Count { get; }`: 返回 `Dictionary` 中的元素数量。

+   `void Add(TKey key, TValue value)`: 添加键和关联的值。如果键已存在，则抛出异常。

+   `void Clear()`: 从 `Dictionary` 中清除所有键和值。

+   `bool ContainsKey(TKey key)`: 如果指定的键存在，则返回 `true`。

+   `bool ContainsValue(TValue value)`: 如果指定的值存在，则返回 `true`。

+   `bool Remove(TKey key)`: 删除与相关键关联的值。

+   `bool TryAdd(TKey key, TValue value)`: 尝试添加键和值。如果键已存在，不会抛出异常。如果值已添加，则返回 `true`。

+   `bool TryGetValue(TKey key, out TValue value)`: 如果存在，获取与键关联的值。如果找到，则返回 `true`。

以下代码显示了如何使用 `Dictionary` 添加和导航 `Country` 记录：

1.  在你的 `Chapter04\Examples` 文件夹中，添加一个名为 `DictionaryExamples.cs` 的新类。

1.  首先定义一个 `Country` 记录，它传递一个 `Name` 参数：

    ```cs
    using System;
    using System.Collections.Generic;
    namespace Chapter04.Examples
    {
        public record Country(string Name)
        {}
        class DictionaryExamples
        {
            public static void Main()
            {
    ```

1.  使用 `Dictionary` 初始化语法创建一个包含五个国家的 `Dictionary`，如下所示：

    ```cs
                var countries = new Dictionary<string, Country>
                {
                    {"AFG", new Country("Afghanistan")},
                    {"ALB", new Country("Albania")},
                    {"DZA", new Country("Algeria")},
                    {"ASM", new Country("American Samoa")},
                    {"AND", new Country("Andorra")}
                };
    ```

1.  在下一个代码片段中，`Dictionary` 实现了 `IEnumerable` 接口，这允许你检索表示 `Dictionary` 中键和值项的键值对：

    ```cs
                Console.WriteLine("Enumerate foreach KeyValuePair");
                foreach (var kvp in countries)
                {
                    Console.WriteLine($"\t{kvp.Key} = {kvp.Value.Name}");
                }
    ```

1.  运行示例代码会产生以下输出。通过遍历 `countries` 中的每个项目，你可以看到五个国家代码及其名称：

    ```cs
    Enumerate foreach KeyValuePair
            AFG = Afghanistan
            ALB = Albania
            DZA = Algeria
            ASM = American Samoa
            AND = Andorra
    ```

1.  存在一个具有 `AFG` 键的条目，因此使用传递 `AFG` 作为键的 `set indexer` 允许设置一个新的 `Country` 记录，该记录替换了具有 `AGF` 键的先前项。你可以添加以下代码来完成此操作：

    ```cs
                Console.WriteLine("set indexor AFG to new value");
                countries["AFG"] = new Country("AFGHANISTAN");
                Console.WriteLine($"get indexor AFG: {countries["AFG"].Name}");
    ```

1.  当你运行代码时，添加 `AFG` 的键允许你使用该键获取值：

    ```cs
    set indexor AFG to new value
    get indexor AFG: AFGHANISTAN
    ContainsKey AGO: False
    ContainsKey and: False
    ```

1.  对于字符串键，键比较是区分大小写的，因此 `AGO` 存在，但 `and` 不存在，因为相应的国家（安道尔）使用大写 `AND` 键定义。你可以添加以下代码来检查这一点：

    ```cs
                Console.WriteLine($"ContainsKey {"AGO"}:                          {countries.ContainsKey("AGO")}");
                Console.WriteLine($"ContainsKey {"and"}:                          {countries.ContainsKey("and")}"); // Case sensitive
    ```

1.  使用 `Add` 添加新条目时，如果键已存在，将抛出异常。这可以通过添加以下代码来查看：

    ```cs
                var anguilla = new Country("Anguilla");
                Console.WriteLine($"Add {anguilla}...");
                countries.Add("AIA", anguilla);
                try
                {
                    var anguillaCopy = new Country("Anguilla");
                    Console.WriteLine($"Adding {anguillaCopy}...");
                    countries.Add("AIA", anguillaCopy);
                }
                catch (Exception e)
                {
                    Console.WriteLine($"Caught {e.Message}");
                }
    ```

1.  相反，`TryAdd` 不接受 `AIA` 键，因此使用 `TryAdd` 仅返回一个 `false` 值而不是抛出异常：

    ```cs
                var addedAIA = countries.TryAdd("AIA", new Country("Anguilla"));
                Console.WriteLine($"TryAdd AIA: {addedAIA}");
    ```

1.  如以下输出所示，使用 `AIA` 键添加 `Anguilla` 一次是有效的，但再次使用 `AIA` 键添加它会导致捕获到异常：

    ```cs
    Add Country { Name = Anguilla }...
    Adding Country { Name = Anguilla }...
    Caught An item with the same key has already been added. Key: AIA
    TryAdd AIA: False
    ```

1.  `TryGetValue`，正如其名所示，允许你通过键尝试获取值。你传入一个可能不存在于 `Dictionary` 中的键。请求一个键缺失的对象将确保不会抛出异常。如果你不确定是否已为指定的键添加了值，这很有用：

    ```cs
                var tryGet = countries.TryGetValue("ALB", out Country albania1);
                Console.WriteLine($"TryGetValue for ALB: {albania1}                              Result={tryGet}");
                countries.TryGetValue("alb", out Country albania2);
                Console.WriteLine($"TryGetValue for ALB: {albania2}");
            }
        }
    }
    ```

1.  运行此代码后，你应该看到以下输出：

    ```cs
    TryGetValue for ALB: Country { Name = Albania } Result=True
    TryGetValue for ALB:
    ```

    注意

    Visual Studio 可能会报告以下警告：`Warning CS8600: 将空字面量或可能的空值转换为不可为 null 的类型`。这是 Visual Studio 提醒您，变量可能在运行时具有空值。

您已经看到如何使用 `Dictionary` 类来确保仅将唯一标识符与值相关联。即使您不知道 `Dictionary` 中有哪些键直到运行时，您也可以使用 `TryGetValue` 和 `TryAdd` 方法来防止运行时异常。

注意

您可以在[`packt.link/vzHUb`](https://packt.link/vzHUb)找到用于此示例的代码。

在此示例中，使用了字符串键作为 `Dictionary`。然而，任何类型都可以用作键。您会发现，当从关系型数据库检索源数据时，通常使用整数值作为键，因为整数在内存中通常比字符串更高效。现在，您将通过练习使用此功能。

## 练习 4.02：使用 Dictionary 计算句子中的单词数量

您被要求创建一个控制台应用程序，要求用户输入一个句子。然后控制台应将输入分割成单个单词（使用空格字符作为单词分隔符）并计算每个单词出现的次数。如果可能，应从输出中移除简单的标点符号，并且您需要忽略大写字母的单词，例如，`Apple` 和 `apple` 都应视为单个单词。

这是对 `Dictionary` 的理想使用。`Dictionary` 将使用字符串作为键（每个单词的唯一条目）以及一个 `int` 值来计数单词。您将使用 `string.Split()` 将句子分割成单词，并使用 `char.IsPunctuation` 移除任何尾随的标点符号。

执行以下步骤：

1.  在您的 `Chapter04\Exercises` 文件夹中，创建一个新的文件夹，称为 `Exercise02`。

1.  在 `Exercise02` 文件夹中，添加一个名为 `Program.cs` 的新类。

1.  首先定义一个新的类，称为 `WordCounter`。这个类可以被标记为 `static`，这样就可以在不创建实例的情况下使用：

    ```cs
    using System;
    using System.Collections.Generic;
    namespace Chapter04.Exercises.Exercise02
    {
        static class WordCounter 
        {
    ```

1.  定义一个名为 `Process` 的 `static` 方法：

    ```cs
            public static IEnumerable<KeyValuePair<string, int>> Process(            string phrase)
            {
                var wordCounts = new Dictionary<string, int>();
    ```

这将传递一个短语并返回 `IEnumerable<KeyValuePair>`，允许调用者遍历结果的 `Dictionary`。在此定义之后，`wordCounts` 的 `Dictionary` 使用 `string`（每个找到的单词）和一个 `int`（单词出现的次数）作为键。

1.  您需要忽略首字母大写的单词的案例，因此在使用 `string.Split` 方法分割短语之前，将字符串转换为小写等效形式。

1.  然后，您可以使用 `RemoveEmptyEntries` 选项移除任何空字符串值。为此，添加以下代码：

    ```cs
                 var words = phrase.ToLower().Split(' ',                        StringSplitOptions.RemoveEmptyEntries);
    ```

1.  使用简单的 `foreach` 循环遍历短语中找到的各个单词：

    ```cs
                foreach(var word in words)
                {
                    var key = word;
                    if (char.IsPunctuation(key[key.Length-1]))
                    {
                        key = key.Remove(key.Length-1);
                    }
    ```

使用 `char.IsPunctuation` 方法从单词末尾移除标点符号。

1.  使用 `TryGetValue` 方法检查是否存在具有当前单词的 `Dictionary` 条目。如果有，则通过一个更新 `count` 为一：

    ```cs
                    if (wordCounts.TryGetValue(key, out var count))
                    {
                        wordCounts[key] = count + 1;
                    }
                    else
                    {
                        wordCounts.Add(key, 1);
                    }
                }
    ```

如果单词不存在，则添加一个新的单词键，其起始值为`1`。

1.  一旦处理完短语中的所有单词，返回`wordCounts Dictionary`：

    ```cs
                return wordCounts;
            }
        }
    ```

1.  现在，编写一个控制台应用程序，允许用户输入一个短语：

    ```cs
        class Program
        {
            public static void Main()
            {
                string input;
                do
                {
                    Console.Write("Enter a phrase:");
                    input = Console.ReadLine();
    ```

当用户输入一个空字符串时，`do`循环将结束；你将在接下来的步骤中添加这段代码。

1.  调用`WordCounter.Process`方法以返回一个可以枚举的键值对。

1.  对于每个`key`和`value`，写出单词及其计数，并将每个单词右对齐：

    ```cs
                    if (!string.IsNullOrEmpty(input))
                    {
                        var countsByWord = WordCounter.Process(input);
                        var i = 0;
                        foreach (var (key, value) in countsByWord)
                        {
                            Console.Write($"{key.PadLeft(20)}={value}\t");
                            i++;
                            if (i % 3 == 0)
                            {
                                Console.WriteLine();
                            }
                        }
                        Console.WriteLine();
    ```

每隔三个单词后开始新的一行（使用`i % 3 = 0`），以改善输出格式。

1.  完成循环：

    ```cs
                        }
                } while (input != string.Empty);
            }
        }
    }
    ```

1.  使用 1863 年《葛底斯堡演说》的开头文本运行控制台产生以下输出：

    ```cs
    Enter a phrase: Four score and seven years ago our fathers brought forth, upon this continent, a new nation, conceived in liberty, and dedicated to the proposition that all men are created equal. Now we are engaged in a great civil war, testing whether that nation, or any nation so conceived, and so dedicated, can long endure.
                    four=1                 score=1                 and=3
                   seven=1                 years=1                 ago=1
                     our=1               fathers=1             brought=1
                   forth=1                  upon=1                this=1
               continent=1                     a=2                 new=1
                  nation=3             conceived=2                  in=2
                 liberty=1             dedicated=2                  to=1
                     the=1           proposition=1                that=2
                     all=1                   men=1                 are=2
                 created=1                 equal=1                 now=1
                      we=1               engaged=1               great=1
                   civil=1                   war=1             testing=1
                 whether=1                    or=1                 any=1
                      so=2                   can=1                 long=1
                  endure=1
    ```

    注意

    您可以在网上搜索《葛底斯堡演说》或访问[`rmc.library.cornell.edu/gettysburg/good_cause/transcript.htm`](https://rmc.library.cornell.edu/gettysburg/good_cause/transcript.htm)。

从结果中，您可以看到每个单词只显示一次，并且某些单词，如`and`和`that`，在演讲中出现了多次。单词按其在文本中出现的顺序列出，但这种情况并不总是适用于`Dictionary`类。应假定顺序不会以这种方式保持固定；应使用键访问字典的值。

注意

您可以在[`packt.link/Dnw4a`](https://packt.link/Dnw4a)找到此练习使用的代码。

到目前为止，您已经了解了.NET 中常用的主要集合。现在是时候看看 LINQ 了，它广泛使用了基于`IEnumerable`接口的集合。

# LINQ

LINQ（发音为**link**）代表语言集成查询。LINQ 是一种通用语言，可以使用类似于结构化查询语言（SQL）的语法来查询内存中的对象，即它用于查询数据库。它是 C#语言的一个增强，使得使用类似 SQL 的查询表达式或查询运算符（通过一系列扩展方法实现）与内存中的对象交互变得更加容易。

微软最初为 LINQ 的想法是使用 LINQ 提供者来弥合.NET 代码和数据源（如关系数据库和 XML）之间的差距。LINQ 提供者形成了一套构建块，可以用来查询各种数据源，使用类似的一组查询运算符，而不需要调用者了解每个数据源如何工作的复杂性。以下是一个提供者列表及其使用方法：

+   LINQ to Objects：应用于内存中对象（如列表中定义的对象）的查询。

+   LINQ to SQL：应用于 SQL Server、Sybase 或 Oracle 等关系数据库的查询。

+   LINQ to XML：应用于 XML 文档的查询。

本章将介绍 LINQ to Objects。这无疑是 LINQ 提供程序最常见的使用方式，它提供了一种灵活的方式来查询内存中的集合。实际上，当谈论 LINQ 时，大多数人指的是 LINQ to Objects，主要归因于它在 C# 应用程序中的普遍使用。

LINQ 的核心在于，可以使用简洁且易于使用的语法将集合转换为、过滤和聚合成新的形式。LINQ 可以使用两种可互换的风格：

+   查询运算符

+   查询表达式

每种风格都提供不同的语法来实现相同的结果，你选择使用哪一种通常取决于个人喜好。每种风格都可以在代码中轻松交织。

## 查询运算符

这些基于一系列核心扩展方法。一个方法的结果可以被链式组合成编程风格，这通常比基于表达式的对应风格更容易理解。

扩展方法通常接受一个 `IEnumerable<T>` 或 `IQueryable<T>` 输入源，例如列表，并允许应用 `Func<T>` 断言到该源。源基于泛型，因此查询运算符可以与所有类型一起工作。例如，处理 `List<string>` 与处理 `List<Customer>` 一样简单。

在以下代码片段中，`.Where`、`.OrderBy` 和 `.Select` 是被调用的扩展方法：

```cs
books.Where(book => book.Price > 10)
     .OrderBy(book => book.Price)
     .Select(book => book.Name)
```

在这里，你正在使用 `.Where` 扩展方法从结果中找到所有单价大于 `10` 的书籍，然后使用 `.OrderBy` 扩展方法进行排序。最后，使用 `.Select` 方法提取每本书的名称。这些方法本来可以声明为单行代码，但以这种方式链式调用提供了更直观的语法。这将在接下来的章节中详细介绍。

## 查询表达式

查询表达式是 C# 语言的增强功能，类似于 SQL 语法。C# 编译器将查询表达式编译成一系列查询运算符扩展方法调用。请注意，并非所有查询运算符都有等效的查询表达式实现。

查询表达式有以下规则：

+   它们以 `from` 子句开始。

+   它们可以包含至少一个或多个可选的 `where`、`orderby`、`join`、`let` 以及额外的 `from` 子句。

+   它们以 `select` 或 `group` 子句结束。

以下代码片段在功能上与上一节中定义的查询运算符风格等效：

```cs
from book in books where book.Price > 10 orderby book.Price select book.Name
```

当你学习标准查询运算符时，将更深入地了解这两种风格。

## 延迟执行

无论你选择使用查询运算符、查询表达式，还是两者的混合，重要的是要记住，对于许多运算符，你定义的查询在定义时并不会执行，只有在枚举时才会执行。这意味着只有在调用 `foreach` 语句或 `ToList`、`ToArray`、`ToDictionary`、`ToLookup` 或 `ToHashSet` 方法时，实际的查询才会执行。

这允许在代码的其他地方构建查询，包括额外的标准，然后使用或甚至重新使用不同的数据集。回想一下在 *第三章*，*委托、Lambda 表达式和事件* 中，您看到了类似的行为。委托不是在定义的地方执行，而是在被调用时执行。

在以下简短的查询操作符示例中，输出将是 `abz`，即使 `z` 是在查询定义 **之后** 但在 **枚举之前** 添加的。这表明 LINQ 查询是在需要时评估的，而不是在声明的地方。

```cs
var letters = new List<string> { "a", "b"}
var query = letters.Select(w => w.ToUpper());
letters.Add("z");
foreach(var l in query) 
  Console.Write(l);
```

## 标准查询操作符

LINQ 由一组核心扩展方法驱动，称为标准查询操作符。这些方法根据其功能分组。有许多标准查询操作符可用，因此在本介绍中，您将探索您可能经常使用的所有主要操作符。

### 投影操作

投影操作允许您仅使用所需的属性将对象转换为新的结构。您可以创建一个新类型，应用数学运算，或返回原始对象：

+   `Select`：将源中的每个项目投影到新的形式。

+   `SelectMany`：将源中的所有项目投影，将结果扁平化，并可选择将它们投影到新的形式。没有 `SelectMany` 的查询表达式等效项。

### Select

考虑以下代码片段，它遍历一个包含值 `Mon`，`Tues` 和 `Wednes` 的 `List<string>`，并将每个值后面附加单词 `day` 输出。

在您的 `Chapter04\Examples` 文件夹中，添加一个名为 `LinqSelectExamples.cs` 的新文件，并按照以下方式编辑它：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
namespace Chapter04.Examples
{
    class LinqSelectExamples
    {
        public static void Main()
        {
            var days = new List<string> { "Mon", "Tues", "Wednes" };
            var query1 = days.Select(d => d + "day");
            foreach(var day in query1)
                Console.WriteLine($"Query1: {day}");         
```

首先查看查询操作符语法，您可以看到 `query1` 使用了 `Select` 扩展方法，并定义了一个 `Func<T>`，如下所示：

```cs
d => d + "day"
```

当执行时，变量 `d` 被传递到 lambda 表达式，该表达式将单词 `day` 添加到 `days` 列表中的每个字符串：`"Mon"`，`"Tues"`，`"Wednes"`。这返回一个新的 `IEnumerable<string>` 实例，其中源变量 `days` 中的原始值保持不变。

您现在可以使用 `foreach` 遍历新的 `IEnumerable` 实例，如下所示：

```cs
            var query2 = days.Select((d, i) => $"{i} : {d}day");
            foreach (var day in query2)
                Console.WriteLine($"Query2: {day}");
```

注意，`Select` 方法还有一个重载，允许访问源中的索引位置和值，而不仅仅是值本身。在这里，使用 `( d , i ) =>` 语法传递 `d`（字符串值）和 `i`（其索引），并将它们连接成一个新的字符串。输出将显示为 `0 : Monday`，`1 : Tuesday`，依此类推。

### 匿名类型

在您继续查看 `Select` 投影之前，值得注意的是，C# 并不限制您仅从现有字符串创建新字符串。您可以投影到任何类型。

你还可以创建匿名类型，这些类型是由编译器根据你命名的和指定的属性创建的。例如，考虑以下示例，它将创建一个新类型，该类型表示`Select`方法的结果：

```cs
            var query3 = days.Select((d, i) => new
            {
                Index = i, 
                UpperCaseName = $"{d.ToUpper()}DAY"
            });
            foreach (var day in query3)
                Console.WriteLine($"Query3: Index={day.Index},                                             UpperCaseDay={day.UpperCaseName}");
```

在这里，`query3`产生了一个具有索引和`UpperCaseName`属性的新类型；值使用`Index = i`和`UpperCaseName = $"{d.ToUpper()}DAY"`分配。

这些类型的作用域是可在你的局部方法中使用，然后可以在任何局部语句中使用，例如在之前的`foreach`块中。这可以节省你创建类以临时存储`Select`方法中值的需要。

运行代码会产生以下格式的输出：

```cs
Index=0, UpperCaseDay=MONDAY
```

作为替代，考虑等效的查询表达式看起来如何。在以下示例中，你从`from day in days`表达式开始。这将为`days`列表中的字符串值分配名称`day`。然后使用`select`将其投影到一个新的字符串，并为每个字符串添加`"day"`。

这在功能上与`query1`中的示例等效。唯一的区别是代码的可读性：

```cs
            var query4 = from day in days
                         select day + "day";
            foreach (var day in query4)
                Console.WriteLine($"Query4: {day}");
```

以下示例片段混合了查询操作符和查询表达式。`select`查询表达式不能用于选择值和索引，因此使用`Select`扩展方法来创建一个具有`Name`和`Index`属性的匿名类型：

```cs
                       var query5 = from dayIndex in 
                         days.Select( (d, i) => new {Name = d, Index = i})
                         select dayIndex;
            foreach (var day in query5)
                Console.WriteLine($"Query5: Index={day.Index} : {day.Name}");
            Console.ReadLine();
        }
    }
}
```

运行完整示例会产生以下输出：

```cs
Query1: Monday
Query1: Tuesday
Query1: Wednesday
Query2: 0 : Monday
Query2: 1 : Tuesday
Query2: 2 : Wednesday
Query3: Index=0, UpperCaseDay=MONDAY
Query3: Index=1, UpperCaseDay=TUESDAY
Query3: Index=2, UpperCaseDay=WEDNESDAY
Query4: Monday
Query4: Tuesday
Query4: Wednesday
Query5: Index=0 : Mon
Query5: Index=1 : Tues
Query5: Index=2 : Wednes
```

再次强调，这主要取决于个人喜好，选择使用哪种形式。随着查询变长，一种形式可能比另一种形式需要更少的代码。

注意

你可以在[`packt.link/wKye0`](https://packt.link/wKye0)找到此示例使用的代码。

### SelectMany

你已经看到`Select`如何用于从源集合中的每个项目投影值。在具有可枚举属性的源的情况下，`SelectMany`扩展方法可以将多个项目提取到单个列表中，然后可以可选地将其投影到新形式。

以下示例创建了两个`City`记录，每个记录包含多个`Station`名称，并使用`SelectMany`从两个城市中提取所有车站：

1.  在你的`Chapter04\Examples`文件夹中，添加一个名为`LinqSelectManyExamples.cs`的新文件，并按照以下方式编辑它：

    ```cs
    using System;
    using System.Collections.Generic;
    using System.Linq;
    namespace Chapter04.Examples
    {
        record City (string Name, IEnumerable<string> Stations);
        class LinqSelectManyExamples
        {
            public static void Main()
            {
                var cities = new List<City>
                {
                    new City("London", new[] {"Kings Cross KGX",                                           "Liverpool Street LVS",                                           "Euston EUS"}),
                    new City("Birmingham", new[] {"New Street NST"})
                };
                Console.WriteLine("All Stations: ");
                foreach (var station in cities.SelectMany(city => city.Stations))
                {
                    Console.WriteLine(station);
                }
    ```

传递给`SelectMany`的`Func`参数要求你指定一个可枚举属性，在这种情况下，是`City`类的`Stations`属性，它包含一个字符串名称列表（参见突出显示的代码）。

注意这里是如何使用快捷方式，直接将查询集成到`foreach`语句中。你没有修改或重用查询变量，所以单独定义它（如之前所做）没有好处。

`SelectMany` 从 `List<City>` 变量中的所有项中提取所有车站名称。从元素 `0` 处的 `City` 类开始，其名称为 `London`，它将提取三个车站名称 `("Kings Cross KGX"`, `"Liverpool Street LVS"`, 和 `"Euston EUS"`）。然后它将移动到第二个 `City` 元素，名为 `Birmingham`，并提取单个车站，名为 `"New Street NST"`。

1.  运行示例会产生以下输出：

    ```cs
    All Stations:
    Kings Cross KGX
    Liverpool Street LVS
    Euston EUS
    New Street NST
    ```

1.  作为替代，考虑以下代码片段。在这里，您恢复使用查询变量 `stations`，以使代码更容易理解：

    ```cs
                Console.Write("All Station Codes: ");
                var stations = cities
                    .SelectMany(city => city.Stations.Select(s => s[³..]));
                foreach (var station in stations)
                {
                    Console.Write($"{station} ");
                }
                Console.WriteLine();
                Console.ReadLine();
            }
        }
    }
    ```

而不是仅仅返回每个 `Station` 字符串，此示例使用嵌套的 `Select` 方法和 `Range` 操作符，通过 `s[³..]` 从车站名称中提取最后三个字符，其中 `s` 是每个车站名称的字符串，`³` 表示 `Range` 操作符应该提取从字符串最后三个字符开始的字符串。

1.  运行示例会产生以下输出：

    ```cs
    All Station Codes: KGX LVS EUS NST
    ```

您可以在输出中看到每个车站名称的最后三个字符。

注意

您可以在 [`packt.link/g8dXZ`](https://packt.link/g8dXZ) 找到用于此示例的代码。

在下一节中，您将了解根据条件过滤结果的过滤操作。

## 过滤操作

过滤操作允许您过滤结果，只返回符合特定条件的那些项。例如，考虑以下代码片段，其中包含一个订单列表：

1.  在您的 `Chapter04\Examples` 文件夹中，添加一个名为 `LinqWhereExample.cs` 的新文件，并按以下方式编辑它：

    ```cs
    LinqWhereExamples.cs
    using System;
    using System.Collections.Generic;
    using System.Linq;
    namespace Chapter04.Examples
    {
        record Order (string Product, int Quantity, double Price);
        class LinqWhereExamples
        {
            public static void Main()
            {
                var orders = new List<Order>
                {
                    new Order("Pen", 2, 1.99),
                    new Order("Pencil", 5, 1.50),
                    new Order("Note Pad", 1, 2.99),
    ```

```cs
You can find the complete code here: https://packt.link/ZJpb5.
```

在这里，定义了一些用于各种文具产品的订单项。假设您想输出所有数量大于五的订单（这应该输出源中的 `Ruler` 和 `USB Memory Stick` 订单）。

1.  为了做到这一点，您可以添加以下代码：

    ```cs
                Console.WriteLine("Orders with quantity over 5:");
                foreach (var order in orders.Where(o => o.Quantity > 5))
                {
                    Console.WriteLine(order);
                }
    ```

1.  现在，假设您将标准扩展到查找所有产品名称为 `Pen` 或 `Pencil` 的产品。您可以将该结果链入 `Select` 方法，该方法将返回每个订单的总价值；请记住，`Select` 可以从源返回任何内容，甚至是一个简单的额外计算，如下所示：

    ```cs
                Console.WriteLine("Pens or Pencils:");
                foreach (var orderValue in orders
                    .Where(o => o.Product == "Pen"  || o.Product == "Pencil")
                    .Select( o => o.Quantity * o.Price))
                {
                    Console.WriteLine(orderValue);
                }
    ```

1.  接下来，以下代码片段中的查询表达式使用 `where` 子句查找价格小于或等于 `3.99` 的订单。它将这些订单投影到一个具有 `Name` 和 `Value` 属性的匿名类型中，您可以使用 `foreach` 语句遍历这些属性：

    ```cs
                var query = from order in orders
                   where order.Price <= 3.99
                   select new {Name=order.Product, Value=order.Quantity*order.Price};
                Console.WriteLine("Cheapest Orders:");
                foreach(var order in query)
                {
                    Console.WriteLine($"{order.Name}: {order.Value}");
                }
            }
        }
    }
    ```

1.  运行完整示例会产生以下结果：

    ```cs
    Orders with quantity over 5:
    Order { Product = Ruler, Quantity = 10, Price = 0.5 }
    Order { Product = USB Memory Stick, Quantity = 6, Price = 20 }
    Pens or Pencils:
    3.98
    7.5
    Cheapest Orders:
    Pen: 3.98
    Pencil: 7.5
    Note Pad: 2.99
    Stapler: 3.99
    Ruler: 5
    ```

现在您已经看到了查询操作的实际应用，值得回顾延迟执行，看看它如何影响多次枚举的查询。

在下一个示例中，您有一个由车辆完成的旅程集合，这些旅程通过 `TravelLog` 记录填充。`TravelLog` 类包含一个 `AverageSpeed` 方法，该方法在每次执行时记录一个控制台消息，并且，正如其名称所暗示的，返回该旅程期间车辆的平均速度：

1.  在您的 Chapter04\Examples 文件夹中，添加一个名为`LinqMultipleEnumerationExample.cs`的新文件，并按以下方式编辑它：

    ```cs
    using System;
    using System.Collections.Generic;
    using System.Linq;
    namespace Chapter04.Examples
    {
        record TravelLog (string Name, int Distance, int Duration)
        {
            public double AverageSpeed()
            {
                Console.WriteLine($"AverageSpeed() called for '{Name}'");
                return Distance / Duration;
            }
        }
        class LinqMultipleEnumerationExample
        {
    ```

1.  接下来，定义控制台应用程序的`Main`方法，该方法使用四个`TravelLog`记录填充`travelLogs`列表。您将为这个添加以下代码：

    ```cs
            public static void Main()
            {
                var travelLogs = new List<TravelLog>
                {
                    new TravelLog("London to Brighton", 50, 4),
                    new TravelLog("Newcastle to London", 300, 24),
                    new TravelLog("New York to Florida", 1146, 19),
                    new TravelLog("Paris to Berlin", 546, 10)
                };
    ```

1.  现在，您将创建一个包含`Where`子句的`fastestJourneys`查询变量。当枚举时，此`Where`子句将调用每个行程的`AverageSpeed`方法。

1.  然后，使用`foreach`循环遍历`fastestJourneys`中的项目，并将名称和距离写入控制台（注意在`foreach`循环中调用`AverageSpeed`方法）：

    ```cs
                var fastestJourneys = travelLogs.Where(tl => tl.AverageSpeed() > 50);
                Console.WriteLine("Fastest Distances:");
                foreach (var item in fastestJourneys)
                {
                    Console.WriteLine($"{item.Name}: {item.Distance} miles");
                }
                Console.WriteLine();
    ```

1.  运行代码块将产生以下输出，每个行程的`Name`和`Distance`：

    ```cs
    Fastest Distances:
    AverageSpeed() called for 'London to Brighton'
    AverageSpeed() called for 'Newcastle to London'
    AverageSpeed() called for 'New York to Florida'
    New York to Florida: 1146 miles
    AverageSpeed() called for 'Paris to Berlin'
    Paris to Berlin: 546 miles
    ```

1.  您可以看到`AverageSpeed`被调用为`Where`条件。到目前为止，这是预期的，但现在，您可以使用相同的查询输出`Name`，或者，作为替代，输出`Duration`：

    ```cs
                Console.WriteLine("Fastest Duration:");
                foreach (var item in fastestJourneys)
                {
                    Console.WriteLine($"{item.Name}: {item.Duration} hours");
                }
                Console.WriteLine();
    ```

1.  运行此代码块将产生相同的`AverageSpeed`方法：

    ```cs
    Fastest Duration:
    AverageSpeed() called for 'London to Brighton'
    AverageSpeed() called for 'Newcastle to London'
    AverageSpeed() called for 'New York to Florida'
    New York to Florida: 19 hours
    AverageSpeed() called for 'Paris to Berlin'
    Paris to Berlin: 10 hours
    ```

这表明每当查询被枚举时，完整的查询是`AverageSpeed`，但如果一个方法需要访问数据库以提取一些数据怎么办？这将导致多次数据库调用，并且可能是一个非常慢的应用程序。

1.  您可以使用`ToList`、`ToArray`、`ToDictionary`、`ToLookup`或`ToHashSet`等方法确保查询可以多次枚举，但`Where`子句中包含一个额外的`ToList`调用以立即执行查询并确保它不会被重新评估：

    ```cs
                Console.WriteLine("Fastest Duration Multiple loops:");
                var fastestJourneysList = travelLogs
                      .Where(tl => tl.AverageSpeed() > 50)
                      .ToList();
                for (var i = 0; i < 2; i++)
                {
                    Console.WriteLine($"Fastest Duration Multiple loop iteration {i+1}:");
                    foreach (var item in fastestJourneysList)
                    {
                        Console.WriteLine($"{item.Name}: {item.Distance} in {item.Duration} hours");
                    }
                }
            }
        }
    }
    ```

1.  运行代码块将产生以下输出。注意`AverageSpeed`是如何在`Fastest Duration Multiple loop iteration`消息中调用的：

    ```cs
    Fastest Duration Multiple loops:
    AverageSpeed() called for 'London to Brighton'
    AverageSpeed() called for 'Newcastle to London'
    AverageSpeed() called for 'New York to Florida'
    AverageSpeed() called for 'Paris to Berlin'
    Fastest Duration Multiple loop iteration 1:
    New York to Florida: 1146 in 19 hours
    Paris to Berlin: 546 in 10 hours
    Fastest Duration Multiple loop iteration 2:
    New York to Florida: 1146 in 19 hours
    Paris to Berlin: 546 in 10 hours
    ```

注意，从车辆行程集合中，代码返回了车辆在行程中的平均速度。

注意

您可以在[`packt.link/CIZJE`](https://packt.link/CIZJE)找到此示例使用的代码。

## 排序操作

在源中对项目进行排序有五种操作。项目主要按顺序排序，之后可以跟一个可选的次要排序，该排序对主要组内的项目进行排序。例如，您可以使用主要排序按`City`属性对人员列表进行排序，然后使用次要排序进一步按`Surname`属性对它们进行排序：

+   `OrderBy`: 按升序排列值。

+   `OrderByDescending`: 按降序排列值。

+   `ThenBy`: 将主要排序的值按次要升序排序。

+   `ThenByDescending`: 将主要排序的值按次要降序排序。

+   `Reverse`: 简单地返回一个集合，其中源中元素的顺序被反转。没有等效的表达式。

### OrderBy 和 OrderByDescending

在此示例中，您将使用`System.IO`命名空间查询宿主机的`temp`文件夹中的文件，而不是从列表中创建小对象。

静态`Directory`类提供了可以查询文件系统的功能。`FileInfo`检索有关特定文件的信息，例如其大小或创建日期。`Path.GetTempPath`方法返回系统的`temp`文件夹。为了说明这一点，在 Windows 操作系统中，这通常可以在`C:\Users\username\AppData\Local\Temp`找到，其中`username`是特定的 Windows 登录名。对于其他用户和其他系统，这将是不同的：

1.  在您的`Chapter04\Examples`文件夹中，添加一个名为`LinqOrderByExamples.cs`的新文件，并按照以下方式编辑它：

    ```cs
    using System;
    using System.IO;
    using System.Linq;
    namespace Chapter04.Examples
    {
        class LinqOrderByExamples
        {
            public static void Main()
            {
    ```

1.  使用`Directory.EnumerateFiles`方法在`temp`文件夹中查找所有具有`.tmp`扩展名的文件名：

    ```cs
                var fileInfos = Directory.EnumerateFiles(Path.GetTempPath(), "*.tmp")
                    .Select(filename => new FileInfo(filename))
                    .ToList();
    ```

在这里，每个文件名被投影到一个`FileInfo`实例中，并通过`ToList`链接到一个已填充的集合中，这允许您进一步查询结果`fileInfos`的详细信息。

1.  接下来，使用`OrderBy`方法通过比较文件的`CreationTime`属性来按最早的时间排序文件：

    ```cs
                Console.WriteLine("Earliest Files");
                foreach (var fileInfo in fileInfos.OrderBy(fi => fi.CreationTime))
                {
                    Console.WriteLine($"{fileInfo.CreationTime:dd MMM yy}: {fileInfo.Name}");
                }
    ```

1.  要找到最大的文件，重新查询`fileInfos`，并使用`OrderByDescending`按每个文件的`Length`属性排序：

    ```cs
                Console.WriteLine("Largest Files");
                foreach (var fileInfo in fileInfos                                        .OrderByDescending(fi => fi.Length))
                {
                    Console.WriteLine($"{fileInfo.Length:N0} bytes: \t{fileInfo.Name}");
                }
    ```

1.  最后，使用`where`和`orderby`降序表达式找到小于`1,000`字节的长度的大文件：

    ```cs
                Console.WriteLine("Largest smaller files");
                foreach (var fileInfo in
                    from fi in fileInfos
                    where fi.Length < 1000
                    orderby fi.Length descending
                    select fi)
                {
                    Console.WriteLine($"{fileInfo.Length:N0} bytes: \t{fileInfo.Name}");
                }
                Console.ReadLine();
            }
        }
    }
    ```

1.  根据您的`temp`文件夹中的文件，您应该看到如下输出：

    ```cs
    Earliest Files
    05 Jan 21: wct63C3.tmp
    05 Jan 21: wctD308.tmp
    05 Jan 21: wctFE7.tmp
    04 Feb 21: wctE092.tmp
    Largest Files
    38,997,896 bytes:       wctE092.tmp
    4,824,572 bytes:        cb6dfb76-4dc9-494d-9683-ce31eab43612.tmp
    4,014,036 bytes:        492f224c-c811-41d6-8c5d-371359d520db.tmp
    Largest smaller files
    726 bytes:      wct38BC.tmp
    726 bytes:      wctE239.tmp
    512 bytes:      ~DF8CE3ED20D298A9EC.TMP
    416 bytes:      TFR14D8.tmp
    ```

在这个示例中，您查询了宿主机的`temp`文件夹中的文件，而不是从列表中创建小对象。

注意

您可以在[`packt.link/mWeVC`](https://packt.link/mWeVC)找到用于此示例的代码。

### 然后是 ThenBy 和 ThenByDescending

以下示例根据每个引用中找到的单词数量对流行引用进行排序。

在您的`Chapter04\Examples`文件夹中，添加一个名为`LinqThenByExamples.cs`的新文件，并按照以下方式编辑它：

```cs
using System;
using System.IO;
using System.Linq;
namespace Chapter04.Examples
{
    class LinqThenByExamples
    {
        public static void Main()
        {
```

您首先声明一个引用字符串数组，如下所示：

```cs
            var quotes = new[]
            {
                "Love for all hatred for none",
                "Change the world by being yourself",
                "Every moment is a fresh beginning",
                "Never regret anything that made you smile",
                "Die with memories not dreams",
                "Aspire to inspire before we expire"
            };
```

在下一个片段中，每个字符串引用都被投影到一个基于引用中单词数量（使用`String.Split()`找到）的新匿名类型中。项目首先按降序排序以显示单词最多的项目，然后按字母顺序排序：

```cs
            foreach (var item in quotes
                .Select(q => new {Quote = q, Words = q.Split(" ").Length})
                .OrderByDescending(q => q.Words)
                .ThenBy(q => q.Quote))
            {
                Console.WriteLine($"{item.Words}: {item.Quote}");
            }
            Console.ReadLine();
        }
    }
}
```

运行代码按单词计数顺序列出引用，如下所示：

```cs
7: Never regret anything that made you smile
6: Aspire to inspire before we expire
6: Change the world by being yourself
6: Every moment is a fresh beginning
6: Love for all hatred for none
5: Die with memories not dreams
```

注意具有六个单词的引用是如何按字母顺序显示的。

以下（突出显示的代码）是带有`orderby quote.Words descending`的等效查询表达式，后面跟着`quote.Words`升序子句：

```cs
var query = from quote in 
            (quotes.Select(q => new {Quote = q, Words = q.Split(" ").Length}))
orderby quote.Words descending, quote.Words ascending 
            select quote;
foreach(var item in query)        
            {
                Console.WriteLine($"{item.Words}: {item.Quote}");
            }
            Console.ReadLine();
        }
    }
}
```

注意

您可以在[`packt.link/YWJRz`](https://packt.link/YWJRz)找到用于此示例的代码。

现在您已经根据每个引用中找到的单词数量对流行引用进行了排序。现在是时候应用在下一个练习中学到的技能了。

## 练习 4.03：按大陆和面积过滤国家列表

在前面的示例中，您已经看到了可以选择、过滤和排序集合源的代码。现在，您将把这些结合到一个练习中，该练习通过两个大陆（南美洲和非洲）过滤一个小国家列表，并按地理面积排序结果。

执行以下步骤来完成此操作：

1.  在您的`Chapter04\Exercises`文件夹中，创建一个新的`Exercise03`文件夹。

1.  在`Exercise03`文件夹中添加一个名为`Program.cs`的新类。

1.  首先，添加一个`Country`记录，它将传递国家的`Name`、它所属的`Continent`以及其平方英里的`Area`：

    ```cs
    using System;
    using System.Linq;
    namespace Chapter04.Exercises.Exercise03
    {
        class Program
        {
            record Country (string Name, string Continent, int Area);
            public static void Main()
            {
    ```

1.  现在创建一个小的国家数据子集，定义在数组中，如下所示：

    ```cs
                var countries = new[]
                {
                    new Country("Seychelles", "Africa", 176),
                    new Country("India", "Asia", 1_269_219),
                    new Country("Brazil", "South America",3_287_956),
                    new Country("Argentina", "South America", 1_073_500),
                    new Country("Mexico", "South America",750_561),
                    new Country("Peru", "South America",494_209),
                    new Country("Algeria", "Africa", 919_595),
                    new Country("Sudan", "Africa", 668_602)
                };
    ```

该数组包含一个国家的名称、它所属的大陆以及其平方英里的地理大小。

1.  您的搜索条件必须包括`南美洲`或`非洲`。因此，请将它们定义在数组中，而不是使用两个特定的字符串硬编码`where`子句：

    ```cs
                var requiredContinents = new[] {"South America", "Africa"};
    ```

如果您需要修改它，这提供了额外的代码灵活性。

1.  通过按大陆过滤和排序、按面积排序以及使用`.Select`扩展方法（该方法返回`Index`和`item`值）来构建查询：

    ```cs
                var filteredCountries = countries
                    .Where(c => requiredContinents.Contains(c.Continent))
                    .OrderBy(c => c.Continent)
                    .ThenByDescending(c => c.Area)
                    .Select( (cty, i) => new {Index = i, Country = cty});

                foreach(var item in filteredCountries)
                    Console.WriteLine($"{item.Index+1}: {item.Country.Continent}, {item.Country.Name} = {item.Country.Area:N0} sq mi");
            }
        }
    }
    ```

最后，将每个项目投影到一个新的匿名类型中，以便写入控制台。

1.  运行代码块产生以下结果：

    ```cs
    1: Africa, Algeria = 919,595 sq mi
    2: Africa, Sudan = 668,602 sq mi
    3: Africa, Seychelles = 176 sq mi
    4: South America, Brazil = 3,287,956 sq mi
    5: South America, Argentina = 1,073,500 sq mi
    6: South America, Mexico = 750,561 sq mi
    7: South America, Peru = 494,209 sq mi
    ```

注意，`阿尔及利亚`在`非洲`拥有最大的面积，而`巴西`在`南美洲`拥有最大的面积（基于这个小的数据子集）。注意您是如何为每个`Index`添加`1`以提高可读性的（因为从零开始对用户来说不太友好）。

注意

您可以在[`packt.link/Djddw`](https://packt.link/Djddw)找到用于此练习的代码。

您已经看到了如何使用 LINQ 扩展方法访问数据源中的项。现在，您将学习关于分区数据的内容，这可以用于提取项目子集。

## 分区操作

到目前为止，您已经看到了如何过滤数据源中匹配定义条件的项。分区用于您需要将数据源分成两个不同的部分并返回这两个部分中的任意一个以进行后续处理。

例如，假设您有一个按价值排序的车辆列表，并想使用某种方法处理五个最便宜的车辆。如果列表是升序排序的，那么您可以使用`Take(5)`方法（如下文定义）来分区数据，这将提取前五个项并丢弃剩余的项。

有六个分区操作用于分割源数据，返回两个部分中的任意一个。没有分区查询表达式：

+   `Skip`: 返回一个跳过源序列中指定数字位置的项的集合。当您需要跳过源集合中的前 N 个项时使用。

+   `SkipLast`: 返回一个跳过源序列中最后 N 个项的集合。

+   `SkipWhile`: 返回一个跳过源序列中满足指定条件的项的集合。

+   `Take`: 返回一个包含序列中前 N 个项的集合。

+   `TakeLast`: 返回一个包含序列中最后 N 个项的集合。

+   `TakeWhile`: 返回一个只包含满足指定条件的项的集合。

以下示例演示了在未排序的考试等级列表上进行的各种 `Skip` 和 `Take` 操作。在这里，你使用 `Skip(1)` 来忽略排序列表中的最高等级。

1.  在你的 `Chapter04\Examples` 文件夹中，添加一个名为 `LinqSkipTakeExamples.cs` 的新文件，并按照以下方式编辑它：

    ```cs
    using System;
    using System.Linq;
    namespace Chapter04.Examples
    {
        class LinqSkipTakeExamples
        {
            public static void Main()
            {
                var grades = new[] {25, 95, 75, 40, 54, 9, 99};
                Console.Write("Skip: Highest Grades (skipping first):");
                foreach (var grade in grades
                    .OrderByDescending(g => g)
                    .Skip(1))
                {
                    Console.Write($"{grade} ");
                }
                Console.WriteLine();
    ```

1.  接下来，使用关系运算符 `is` 来排除小于 `25` 或大于 `75` 的那些：

    ```cs
                Console.Write("SkipWhile@ Middle Grades (excluding 25 or 75):");
                foreach (var grade in grades
                    .OrderByDescending(g => g)
                    .SkipWhile(g => g is <= 25 or >=75))
                {
                    Console.Write($"{grade} ");
                }
                Console.WriteLine();
    ```

1.  通过使用 `SkipLast`，你可以显示结果的后半部分。添加以下代码：

    ```cs
                Console.Write("SkipLast: Bottom Half Grades:");
                foreach (var grade in grades
                    .OrderBy(g => g)
                    .SkipLast(grades.Length / 2))
                {
                    Console.Write($"{grade} ");
                }
                Console.WriteLine();
    ```

1.  最后，这里使用 `Take(2)` 来显示两个最高等级：

    ```cs
                Console.Write("Take: Two Highest Grades:");
                foreach (var grade in grades
                    .OrderByDescending(g => g)
                    .Take(2))
                {
                    Console.Write($"{grade} ");
                }
            }
        }
    }
    ```

1.  运行示例产生以下输出，这是预期的：

    ```cs
    Skip: Highest Grades (skipping first):95 75 54 40 25 9
    SkipWhile Middle Grades (excluding 25 or 75):54 40 25 9
    SkipLast: Bottom Half Grades:9 25 40 54
    Take: Two Highest Grades:99 95
    ```

此示例演示了在未排序的考试等级列表上进行的各种 `Skip` 和 `Take` 操作。

注意

你可以在 [`packt.link/TsDFk`](https://packt.link/TsDFk) 找到此示例使用的代码。

## 分组操作

`GroupBy` 对具有相同属性的元素进行分组。它通常用于对数据进行分组或按公共属性对项目计数。结果是类型为 `IGrouping<K, V>` 的可枚举集合，其中 `K` 是键类型，`V` 是指定的值类型。`IGrouping` 本身也是可枚举的，因为它包含所有匹配指定键的项。

例如，考虑以下代码片段，它按名称对客户订单的 `List` 进行分组。在你的 `Chapter04\Examples` 文件夹中，添加一个名为 `LinqGroupByExamples.cs` 的新文件，并按照以下方式编辑它：

```cs
LinqGroupByExamples.cs
using System;
using System.Collections.Generic;
using System.Linq;
namespace Chapter04.Examples
{
    record CustomerOrder(string Name, string Product, int Quantity);
    class LinqGroupByExamples
    {
        public static void Main()
        {
            var orders = new List<CustomerOrder>
            {
                new CustomerOrder("Mr Green", "LED TV", 4),
                new CustomerOrder("Mr Smith", "iPhone", 2),
                new CustomerOrder("Mrs Jones", "Printer", 1),
You can find the complete code here: https://packt.link/GbwF2.
```

在此示例中，你有一个 `CustomerOrder` 对象的列表，并希望按 `Name` 属性对其进行分组。为此，`GroupBy` 方法传递一个 `Func` 委托，该委托从每个 `CustomerOrder` 实例中选择 `Name` 属性。

`GroupBy` 结果中的每一项都包含一个 `Key`（在这种情况下，是客户的 `Name`）。然后你可以对分组项进行排序，以显示按 `Quantity` 排序的 `CustomerOrders` 项，如下所示：

```cs
                foreach (var item in grouping.OrderByDescending(i => i.Quantity))
                {
                    Console.WriteLine($"\t{item.Product} * {item.Quantity}");
                }
            }
            Console.ReadLine();
        }
    }
}
```

运行代码产生以下输出：

```cs
Customer Mr Green:
        LED TV * 4
        MP3 Player * 1
        Microwave Oven * 1
Customer Mr Smith:
        PC * 5
        iPhone * 2
        Printer * 2
Customer Mrs Jones:
        Printer * 1
```

你可以看到数据首先按客户 `Name` 进行分组，然后在每个客户分组内按订单 `Quantity` 排序。等效的查询表达式如下所示：

```cs
            var query = from order in orders
                        group order by order.Name;
            foreach (var grouping in query)
            {
                Console.WriteLine($"Customer {grouping.Key}:");
                foreach (var item in from item in grouping 
                                     orderby item.Quantity descending 
                                     select item)
                {
                    Console.WriteLine($"\t{item.Product} * {item.Quantity}");
                }
            }
```

你现在已经看到了一些常用的 LINQ 操作符。现在你将在练习中将它们组合起来。

## 练习 4.04：在书中查找最常用的单词

在 *第三章*，*委托、事件和 Lambda 表达式* 中，你使用了 `WebClient` 类从网站下载数据。在这个练习中，你将使用从 *Project Gutenberg* 下载的数据。

注意

Project Gutenberg 是一个包含 60,000 本免费电子书的图书馆。你可以在网上搜索 Project Gutenberg 或访问 [`www.gutenberg.org/`](https://www.gutenberg.org/)。

你将创建一个控制台应用程序，允许用户输入一个 URL。然后，你将从 Project Gutenberg URL 下载书籍文本，并使用各种 LINQ 语句来查找书籍文本中最频繁出现的单词。

此外，你还想排除一些常见的停用词；这些词如`and`、`or`和`the`在英语中经常出现，但对句子的意义贡献很小。你将使用`Regex.Split`方法来帮助比简单的空格分隔符更准确地分割单词。执行以下步骤来完成此操作：

注意

你可以在[`packt.link/v4hGN`](https://packt.link/v4hGN)上找到有关正则表达式的更多信息。

1.  在你的`Chapter04\Exercises`文件夹中，创建一个新的`Exercise04`文件夹。

1.  在`Exercise04`文件夹中添加一个名为`Program.cs`的新类。

1.  首先，定义`TextCounter`类。这个类将传入文件的路径，你很快就会添加。这个类应该包含常见的英语停用词：

    ```cs
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Linq;
    using System.Net;
    using System.Text;
    using System.Text.RegularExpressions;
    namespace Chapter04.Exercises.Exercise04
    {
        class TextCounter
        {
            private readonly HashSet<string> _stopWords;
            public TextCounter(string stopWordPath)
            {
                Console.WriteLine($"Reading stop word file: {stopWordPath}");
    ```

1.  使用`File.ReadAllLines`将每个单词添加到`_stopWords` `HashSet`中。

    ```cs
              _stopWords = new HashSet<string>(File.ReadAllLines(stopWordPath));
            }
    ```

你已经使用了`HashSet`，因为每个停用词都是唯一的。

1.  接下来，将包含书籍文本的字符串和要显示的最大单词数传递给`Process`方法。

1.  将结果作为`Tuple<string, int>`集合返回，这样你就不需要创建一个类或记录来保存结果：

    ```cs
            public IEnumerable<Tuple<string, int>> Process(string text,                                                        int maximumWords)
            {
    ```

1.  现在执行查询部分。使用`Regex.Split`和模式`@"\s+"`来分割所有单词。

在最简单的形式中，这个模式将字符串分割成单词列表，通常使用空格或标点符号来识别单词边界。例如，字符串`Hello Goodbye`将被分割成一个包含两个元素的数组，`Hello`和`Goodbye`。返回的字符串项通过`where`进行过滤，以确保使用`Contains`方法忽略所有停用词。然后，单词按值分组，`GroupBy(t=>t)`，使用单词作为`Key`，并使用`grp.Count`来获取出现的次数：

1.  最后，按`Item2`排序，对于这个`Tuple`来说，它是单词计数，然后只取所需数量的单词：

    ```cs
                var words = Regex.Split(text.ToLower(), @"\s+")
                    .Where(t => !_stopWords.Contains(t))
                    .GroupBy(t => t)
                    .Select(grp => Tuple.Create(grp.Key, grp.Count()))
                    .OrderByDescending(tup => tup.Item2) //int
                    .Take(maximumWords);
                return words;
            }
        }
    ```

1.  现在开始创建主控制台应用程序：

    ```cs
        class Program
        {
            public static void Main()
            {
    ```

1.  在`Chapter04`源文件夹中包含一个名为`StopWords.txt`的文本文件：

    ```cs
                const string StopWordFile = "StopWords.txt";
                var counter = new TextCounter(StopWordFile);
    ```

    注意

    你可以在 GitHub 上找到`StopWords.txt`，网址为[`packt.link/Vi8JH`](https://packt.link/Vi8JH)，或者你可以下载任何标准的停用词文件，例如 NLTK 的[`packt.link/ZF1Tf`](https://packt.link/ZF1Tf)。这个文件应该保存在`Chapter04\Exercises`文件夹中。

1.  一旦创建了`TextCounter`，提示用户输入一个 URL：

    ```cs
                string address;
                do
                {
                    //https://www.gutenberg.org/files/64333/64333-0.txt
                    Console.Write("Enter a Gutenberg book URL: ");
                    address = Console.ReadLine();
                    if (string.IsNullOrEmpty(address)) 
                        continue;
    ```

1.  输入一个有效的地址并创建一个新的`WebClient`实例，将数据文件下载到临时文件中。

1.  在将文本文件的内容传递给`TextCounter`之前，对文本文件进行额外处理：

    ```cs
                    using var client = new WebClient();
                    var tempFile = Path.GetTempFileName();
                    Console.WriteLine("Downloading...");
                    client.DownloadFile(address, tempFile);
    ```

古腾堡文本文件包含额外的细节，如作者和标题。这些可以通过读取文件中的每一行来读取。实际的书本文本直到找到以`*** START OF THE PROJECT GUTENBERG EBOOK`开头的行才开始，因此你需要读取每一行以寻找这个起始信息：

```cs
                Console.WriteLine($"Processing file {tempFile}");
                const string StartIndicator = "*** START OF THE PROJECT GUTENBERG EBOOK";
                //Title: The Little Review, October 1914(Vol. 1, No. 7)
                //Author: Various
                var title = string.Empty;
                var author = string.Empty;
```

1.  接下来，将读取到的每一行追加到`StringBuilder`实例中，这对于此类字符串操作来说效率很高：

    ```cs
                    var bookText = new StringBuilder();
                    var isReadingBookText = false;
                    var bookTextLineCount = 0;
    ```

1.  现在解析`tempFile`中的每一行，寻找`Author`、`Title`或`StartIndicator`：

    ```cs
                    foreach (var line in File.ReadAllLines(tempFile))
                    {
                        if (line.StartsWith("Title"))
                        {
                            title = line;
                        }
                        else if (line.StartsWith("Author"))
                        {
                            author = line;
                        }
                        else if (line.StartsWith(StartIndicator))
                        {
                            isReadingBookText = true;
                        }
                        else if (isReadingBookText)
                        {
                            bookText.Append(line);
                            bookTextLineCount++;
                        }
                    }
    ```

1.  如果找到书籍文本，在调用`counter.Process`方法之前提供读取的行和字符的摘要。这里，你想要前`50`个单词：

    ```cs
                    if (bookTextLineCount > 0)
                    {
                        Console.WriteLine($"Processing {bookTextLineCount:N0} lines ({bookText.Length:N0} characters)..");
                      var wordCounts = counter.Process(bookText.ToString(), 50);
                      Console.WriteLine(title);
                      Console.WriteLine(author);
    ```

1.  一旦你得到结果，使用一个`foreach`循环来输出单词计数详情，在每三个单词后添加一个空白行：

    ```cs
                        var i = 0;
                        //deconstruction
                        foreach (var (word, count) in wordCounts)
                        {
                            Console.Write($"'{word}'={count}\t\t");
                            i++;
                            if (i % 3 == 0)
                            {
                                Console.WriteLine();
                            }
                        }
                        Console.WriteLine();
                    }
                    else
                    {
    ```

1.  使用`https://www.gutenberg.org/files/64333/64333-0.txt`作为示例 URL 运行控制台应用程序，会产生以下输出：

    ```cs
    Reading stop word file: StopWords.txt
    Enter a Gutenberg book URL: https://www.gutenberg.org/files/64333/64333-0.txt
    Downloading...
    Processing file C:\Temp\tmpB0A3.tmp
    Processing 4,063 lines (201,216 characters)..
    Title: The Little Review, October 1914 (Vol. 1, No. 7)
    Author: Various
    'one'=108               'new'=95                'project'=62
    'man'=56                'little'=54             'life'=52
    'would'=51              'work'=50               'book'=42
    'must'=42               'people'=39             'great'=37
    'love'=37               'like'=36               'gutenberg-tm'=36
    'may'=35                'men'=35                'us'=32
    'could'=30              'every'=30              'first'=29
    'full'=29               'world'=28              'mr.'=28
    'old'=27                'never'=26              'without'=26
    'make'=26               'young'=24              'among'=24
    'modern'=23             'good'=23               'it.'=23
    'even'=22               'war'=22                'might'=22
    'long'=22               'cannot'=22             '_the'=22
    'many'=21               'works'=21              'electronic'=21
    'always'=20             'way'=20                'thing'=20
    'day'=20                'upon'=20               'art'=20
    'terms'=20              'made'=19
    ```

    注意

    当代码第一次运行时，Visual Studio 可能会显示以下警告：`warning SYSLIB0014: 'WebClient.WebClient()' is obsolete: 'WebRequest, HttpWebRequest, ServicePoint, and WebClient are obsolete. Use HttpClient instead.'`

    这是一个建议使用较新的`HttpClient`类而不是`WebClient`类的推荐。然而，两者在功能上是等效的。

输出显示在下载的`4,063`行文本中找到的单词列表。计数器显示`one`、`new`和`project`是最受欢迎的单词。注意`mr.`、`gutenberg-tm`、`it.`和`_the`如何作为单词出现。这表明在分割单词时使用的正则表达式并不完全准确。

注意

你可以在[`packt.link/Q7Pf8`](https://packt.link/Q7Pf8)找到用于此练习的代码。

对这个练习的一个有趣的增强是按计数对单词进行排序，包括找到的停用词的数量，或者找到平均单词长度。

## 聚合操作

聚合操作用于从数据源中的值集合计算单个值。一个例子可以是收集一个月的降雨量中的最大值、最小值和平均值。

+   `Average`：计算集合中的平均值。

+   `Count`：计算匹配谓词的项的数量。

+   `Max`：计算最大值。

+   `Min`：计算最小值。

+   `Sum`：计算值的总和。

以下示例使用`System.Diagnostics`命名空间中的`Process.GetProcess`方法来检索系统上当前正在运行的进程列表：

在你的`Chapter04\Examples`文件夹中，添加一个名为`LinqAggregationExamples.cs`的新文件，并按以下方式编辑它：

```cs
using System;
using System.Diagnostics;
using System.Linq;
namespace Chapter04.Examples
{
    class LinqAggregationExamples
    {
        public static void Main()
        {
```

首先，调用`Process.GetProcesses().ToList()`来检索系统上正在运行的活动的进程列表：

```cs
            var processes = Process.GetProcesses().ToList();
```

然后，`Count`扩展方法获取返回项的数量。`Count`有一个额外的重载，它接受一个`Func`委托，用于过滤要计数的每个项。`Process`类有一个`PrivateMemorySize64`属性，它返回进程当前消耗的字节数，因此你可以用它来计数`1,000,000`字节的内存：

```cs
            var allProcesses = processes.Count;
            var smallProcesses = processes.Count(proc =>                                        proc.PrivateMemorySize64 < 1_000_000);
```

接着，`Average`扩展方法返回`processes`列表中所有项的特定值的总体平均值。在这种情况下，你使用它来计算平均内存消耗，再次使用`PrivateMemorySize64`属性：

```cs
            var average = processes.Average(p => p.PrivateMemorySize64);
```

`PrivateMemorySize64`属性还用于计算所有进程的最大和最小内存使用量，以及总内存，如下所示：

```cs
            var max = processes.Max(p => p.PrivateMemorySize64);
            var min = processes.Min(p => p.PrivateMemorySize64);
            var sum = processes.Sum(p => p.PrivateMemorySize64);
```

计算完统计数据后，每个值都会写入控制台：

```cs
            Console.WriteLine("Process Memory Details");
            Console.WriteLine($"  All Count: {allProcesses}");
            Console.WriteLine($"Small Count: {smallProcesses}");
            Console.WriteLine($"    Average: {FormatBytes(average)}");
            Console.WriteLine($"    Maximum: {FormatBytes(max)}");
            Console.WriteLine($"    Minimum: {FormatBytes(min)}");
            Console.WriteLine($"      Total: {FormatBytes(sum)}");
        }
```

在前面的代码片段中，`Count`方法返回所有进程的数量，并使用`Predicate`重载，计算内存小于 1,000,000 字节的进程数量（通过检查`process.PrivateMemorySize64`属性）。您还可以看到，`Average`、`Max`、`Min`和`Sum`用于计算系统上进程内存使用的统计数据。

注意

如果尝试使用不包含任何元素的源集合进行计算，聚合运算符将抛出`InvalidOperationException`错误，错误信息为`Sequence contains no elements`。在调用任何聚合运算符之前，您应该检查`Count`或`Any`方法。

最后，`FormatBytes`将内存量格式化为它们的兆字节等效值：

```cs
        private static string FormatBytes(double bytes)
        {
            return $"{bytes / Math.Pow(1024, 2):N2} MB";
        }
    }
}
```

运行示例会产生类似以下的结果：

```cs
Process Memory Details
  All Count: 305
Small Count: 5
    Average: 38.10 MB
    Maximum: 1,320.16 MB
    Minimum: 0.06 MB
      Total: 11,620.03 MB
```

从输出中，您将观察到程序如何检索系统上当前正在运行的进程列表。

注意

您可以在`Chapter04\Examples`文件夹中找到此示例使用的代码，链接为[`packt.link/HI2eV`](https://packt.link/HI2eV)。

## 量词操作

量词操作返回一个`bool`值，指示是否满足`Predicate`条件。这通常用于验证集合中的任何元素是否满足某些标准，而不是依赖于`Count`，后者会枚举集合中的**所有**项，即使您只需要一个结果。

使用以下扩展方法访问量词操作：

+   `All`：如果源序列中的**所有**元素都匹配条件，则返回`true`。

+   `Any`：如果源序列中的**任何**元素匹配条件，则返回`true`。

+   `Contains`：如果源序列包含指定的项，则返回`true`。

以下发牌示例随机选择三张牌，并返回所选牌的摘要。摘要使用`All`和`Any`扩展方法来确定是否有任何牌是梅花或红色，以及所有牌是否是钻石或偶数：

1.  在您的`Chapter04\Examples`文件夹中，添加一个名为`LinqAllAnyExamples.cs`的新文件。

1.  首先，声明一个`enum`来表示一副扑克牌中的四种花色，以及一个`record`类来定义一张扑克牌：

    ```cs
    using System;
    using System.Collections.Generic;
    using System.Linq;
    namespace Chapter04.Examples
    {
        enum PlayingCardSuit
        {
            Hearts,
            Clubs,
            Spades,
            Diamonds
        }
        record PlayingCard (int Number, PlayingCardSuit Suit)
        {
    ```

1.  在运行时描述对象状态的常用做法是重写`ToString`方法，以提供一种用户友好的方式。在这里，牌的数字和花色以字符串的形式返回：

    ```cs
            public override string ToString()
            {
                return $"{Number} of {Suit}";
            }
        }
    ```

1.  现在创建一个类来表示一副牌（为了方便，只创建编号为 1 到 10 的牌）。牌组的构造函数将使用每种花色的 10 张牌填充`_cards`集合：

    ```cs
        class Deck
        {
            private readonly List<PlayingCard> _cards = new();
            private readonly Random _random = new();
            public Deck()
            {
                for (var i = 1; i <= 10; i++)
                {
                    _cards.Add(new PlayingCard(i, PlayingCardSuit.Hearts));
                    _cards.Add(new PlayingCard(i, PlayingCardSuit.Clubs));
                    _cards.Add(new PlayingCard(i, PlayingCardSuit.Spades));
                    _cards.Add(new PlayingCard(i, PlayingCardSuit.Diamonds));
                }
            }
    ```

1.  接下来，`Draw`方法从`_cards`列表中随机选择一张牌，并在返回给调用者之前将其移除：

    ```cs
            public PlayingCard Draw()
            {
                var index = _random.Next(_cards.Count);
                var drawnCard = _cards.ElementAt(index);
                _cards.Remove(drawnCard);
                return drawnCard;
            }
        }
    ```

1.  控制台应用程序使用牌组的`Draw`方法选择三张牌。按照以下方式添加此代码：

    ```cs
        class LinqAllAnyExamples
        {
            public static void Main()
            {
                var deck = new Deck();
                var hand = new List<PlayingCard>();

                for (var i = 0; i < 3; i++)
                {
                    hand.Add(deck.Draw());
                }
    ```

1.  要显示摘要，使用`OrderByDescending`和`Select`操作提取每个`PlayingCard`的用户友好`ToString`描述。然后将其连接成一个分隔的字符串，如下所示：

    ```cs
                var summary = string.Join(" | ", 
                    hand.OrderByDescending(c => c.Number)
                        .Select(c => c.ToString()));
                Console.WriteLine($"Hand: {summary}");
    ```

1.  使用`All`或`Any`，您可以通过卡片的数字总和来概述卡片及其分数。通过使用`Any`，您确定是否`PlayingCardSuit.Clubs`：

    ```cs
                Console.WriteLine($"Any Clubs: {hand.Any(card => card.Suit == PlayingCardSuit.Clubs)}");
    ```

1.  类似地，`Any`用于查看是否是`Hearts`或`Diamonds`花色，因此是`Red`：

    ```cs
                Console.WriteLine($"Any Red: {hand.Any(card => card.Suit == PlayingCardSuit.Hearts || card.Suit == PlayingCardSuit.Diamonds)}");
    ```

1.  在下一个代码片段中，`All`扩展方法检查集合中的每个项目，并返回`true`，在这种情况下，如果`Diamonds`：

    ```cs
                Console.WriteLine($"All Diamonds: {hand.All(card => card.Suit == PlayingCardSuit.Diamonds)}");
    ```

1.  再次使用`All`来查看是否所有卡片号码都能被 2 整除，即它们是偶数：

    ```cs
                Console.WriteLine($"All Even: {hand.All(card => card.Number % 2 == 0)}");
    ```

1.  最后，使用`Sum`聚合方法计算手中的卡片价值：

    ```cs
                Console.WriteLine($"Score :{hand.Sum(card => card.Number)}");
            }
        }
    }
    ```

1.  运行控制台应用程序会产生如下输出：

    ```cs
    Hand: 8 of Spades | 7 of Diamonds | 6 of Diamonds
    Any Clubs: False
    Any Red: True
    All Diamonds: False
    All Even: False
    Score :21
    ```

卡片是随机选择的，所以每次运行程序时您都会有不同的手牌。在这个例子中，得分是`21`，这在纸牌游戏中通常是一个获胜的手牌。

注意

您可以在[`packt.link/xPuTc`](https://packt.link/xPuTc)找到此示例使用的代码。

## 连接操作

连接操作用于根据一个数据源中对象与第二个数据源中具有共同属性的关联来连接两个源。如果您熟悉数据库设计，这可以被视为表之间的主键和外键关系。

一个常见的连接示例是单向关系，例如`Orders`，它有一个类型为`Products`的属性，但`Products`类没有表示到`Orders`集合的逆向关系的集合属性。通过使用`Join`运算符，您可以创建逆向关系以显示`Products`的`Orders`。

以下两个连接扩展方法如下：

+   `Join`: 使用键选择器将两个序列连接起来，以提取值对。

+   `GroupJoin`: 使用键选择器将两个序列连接起来，并将结果项分组。

以下示例包含三个`Manufacturer`记录，每个记录都有一个唯一的`ManufacturerId`。这些数字 ID 用于定义各种`Car`记录，但为了节省内存，您将不会从`Manufacturer`直接到`Car`的内存引用。您将使用`Join`方法在`Manufacturer`和`Car`实例之间创建关联：

1.  在您的`Chapter04\Examples`文件夹中，添加一个名为`LinqJoinExamples.cs`的新文件。

1.  首先，声明`Manufacturer`和`Car`记录如下：

    ```cs
    using System;
    using System.Collections.Generic;
    using System.Linq;
    namespace Chapter04.Examples
    {
        record Manufacturer(int ManufacturerId, string Name);
        record Car (string Name, int ManufacturerId);
    ```

1.  在`Main`入口点内部，创建两个列表，一个用于制造商，另一个用于表示`cars`：

    ```cs
    LinqJoinExamples.cs
        class LinqJoinExamples
        {
            public static void Main()
            {
                var manufacturers = new List<Manufacturer>
                {
                    new(1, "Ford"),
                    new(2, "BMW"),
                    new(3, "VW")
                };
                var cars = new List<Car>
                {
                    new("Focus", 1),
                    new("Galaxy", 1),
                    new("GT40", 1),
    ```

```cs
You can find the complete code here: https://packt.link/Ue7Fj.
```

1.  到目前为止，没有直接的引用，但如您所知，您可以使用`ManufacturerId`通过`int` ID 将它们链接起来。您可以为此添加以下代码：

    ```cs
                var joinedQuery = manufacturers.Join(
                    cars,
                    manufacturer => manufacturer.ManufacturerId,
                    car => car.ManufacturerId,
                    (manufacturer, car) => new                        {ManufacturerName = manufacturer.Name,                         CarName = car.Name});
                foreach (var item in joinedQuery)
                {
                    Console.WriteLine($"{item}");
                }
            }
        }
    }
    ```

在前面的代码片段中，`Join`操作有多种参数。你传入`cars`列表并定义在`manufacturer`和`car`类中应该使用哪些属性来创建连接。在这种情况下，`manufacturer.ManufacturerId = car.ManufacturerId`确定了正确的连接。

最后，`manufacturer`和`car`参数返回一个包含`manufacturer.Name`和`car.Name`属性的新匿名类型。

1.  运行控制台应用程序会产生以下输出：

    ```cs
    { ManufacturerName = Ford, CarName = Focus }
    { ManufacturerName = Ford, CarName = Galaxy }
    { ManufacturerName = Ford, CarName = GT40 }
    { ManufacturerName = BMW, CarName = 1 Series }
    { ManufacturerName = BMW, CarName = 2 Series }
    { ManufacturerName = VW, CarName = Golf }
    { ManufacturerName = VW, CarName = Polo }
    ```

如你所见，每个`Car`和`Manufacturer`实例都使用`ManufacturerId`正确连接。

1.  相等的查询表达式如下（注意，在这种情况下，它比查询运算符语法更简洁）：

    ```cs
    var query = from manufacturer in manufacturers
                join car in cars
                  on manufacturer.ManufacturerId equals car.ManufacturerId
                  select new
                  {
                    ManufacturerName = manufacturer.Name, CarName = car.Name
                  };
    foreach (var item in query)
    {
      Console.WriteLine($"{item}");
    }
    ```

    注意

    你可以在[`packt.link/Wh8jK`](http://packt.link/Wh8jK)找到用于此示例的代码。

在你完成对 LINQ 的探索之前，还有一个与 LINQ 查询表达式相关的区域——`let`子句。

## 在查询表达式中使用 let 子句

在早期的查询表达式中，你通常需要在各种子句中重复类似的外观代码。使用`let`子句，你可以在表达式查询内部引入新变量，并在查询的其余部分重用变量的值。例如，考虑以下查询：

```cs
var stations = new List<string>
{
    "Kings Cross KGX", 
    "Liverpool Street LVS", 
    "Euston EUS", 
    "New Street NST"
};
var query1 = from station in stations
             where station[³..] == "LVS" || station[³..] == "EUS" || 
                   station[0..³].Trim().ToUpper().EndsWith("CROSS")
             select new { code= station[³..],                           name= station[0..³].Trim().ToUpper()};
```

这里，你正在搜索具有`LVS`或`EUS`代码或以`CROSS`结尾的名称的站点。为此，你必须使用范围提取最后三个字符，`station[³..]`，但你已经在两个`where`子句和最终的投影中重复了它。

可以使用`let`子句将站点代码和站点名称都转换为局部变量：

```cs
var query2 = from station in stations
             let code = station[³..]
             let name = station[0..³].Trim().ToUpper()
             where code == "LVS" || code == "EUS" || 
                   name.EndsWith("CROSS") 
             select new {code, name};
```

这里，你使用`let`子句定义了`code`和`name`，并在整个查询中重用了它们。这段代码看起来更整洁，也更易于理解和维护。

运行代码会产生以下输出：

```cs
Station Codes: 
KGX : KINGS CROSS
LVS : LIVERPOOL STREET
EUS : EUSTON
Station Codes (2):
KGX : KINGS CROSS
LVS : LIVERPOOL STREET
EUS : EUSTON
```

注意

你可以在[`packt.link/b2KiG`](https://packt.link/b2KiG)找到用于此示例的代码。

到现在为止，你已经看到了 LINQ 的主要部分。现在你将把这些部分组合到一个活动中，该活动根据用户的准则过滤一组飞行记录，并对找到的飞行子集提供各种统计数据。

## 活动 4.01：国库飞行数据分析

你被要求创建一个控制台应用程序，允许用户下载公开可用的飞行数据文件，并对这些文件进行统计分析。这种分析应用于计算找到的总记录数，以及在该子集中支付的平均、最小和最大票价。

用户应该能够输入多个命令，并且每个命令都应该基于飞行类别、起点或目的地属性添加特定的过滤器。一旦用户输入了所需的准则，就必须输入`go`命令，控制台应该运行查询并输出结果。

你将用于此活动的数据文件包含英国财政部（HM Treasury）在 2011 年 1 月 1 日至 12 月 31 日期间执行的航班详细信息（共有 714 条记录）。你需要使用 `WebClient.DownloadFile` 从以下网址下载数据：[`www.gov.uk/government/uploads/system/uploads/attachment_data/file/245855/HMT_-_2011_Air_Data.csv`](https://www.gov.uk/government/uploads/system/uploads/attachment_data/file/245855/HMT_-_2011_Air_Data.csv)

注意

网站可能在 Internet Explorer 或 Google Chrome 中打开方式不同。这取决于 IE 或 Chrome 在您的机器上的配置。使用 `WebClient.DownloadFile`，您可以按建议下载数据。

理想情况下，程序应一次性下载数据，然后在每次启动时从本地文件系统中重新读取。

![图 4.6：Excel 中 HM 财政部交通数据的预览](img/B16835_04_06.jpg)

图 4.6：Excel 中 HM 财政部交通数据的预览

下载后，数据应被读取到合适的记录结构中，然后添加到集合中，以便应用各种查询。输出应显示所有符合用户标准的行的以下汇总值：

+   记录计数

+   平均票价

+   最低票价

+   最高票价

用户应能够输入以下控制台命令：

+   `Class c`: 添加一个类别过滤器，其中 `c` 是要搜索的航班类别，例如 `经济舱` 或 `商务舱`。

+   `Origin o`: 添加一个 `origin` 过滤器，其中 `o` 是航班起点，例如 `都柏林`、`伦敦` 或 `巴塞尔`。

+   `Destination d`: 添加一个目的地过滤器，其中 `d` 是航班目的地，例如 `德里`。

+   `Clear`: 清除所有过滤器。

+   `go`: 应用当前过滤器。

如果用户输入了多个同类型的过滤器，则应将这些过滤器视为一个 `OR` 过滤器。

可以使用 `enum` 来标识输入的过滤标准类型，如下面的代码行所示：

```cs
enum FilterCriteriaType {Class, Origin, Destination}
```

类似地，可以使用记录来存储每个过滤类型和比较运算符，如下所示：

```cs
record FilterCriteria(FilterCriteriaType Filter, string Operand)
```

每个指定的过滤器都应添加到 `List<FilterCriteria>` 实例中。例如，如果用户输入了两个起点过滤器，一个用于 `dublin`，另一个用于 `london`，那么列表应包含两个对象，每个对象代表一个起点类型过滤器。

当用户输入 `go` 命令时，应构建一个查询，执行以下步骤：

+   将所有 `class` 过滤器的值提取到一个字符串列表（`List<string>`）中。

+   将所有 `origin` 过滤器的值提取到 `List<string>` 中。

+   将所有 `destination` 过滤器的值提取到 `List<string>` 中。

+   使用 `where` 扩展方法根据指定的 `List<string>` 过滤每个标准类型的航班记录。它包含一个执行不区分大小写搜索的方法。

以下步骤将帮助您完成此活动：

1.  在 `Chapter04` 文件夹中创建一个名为 `Activities` 的新文件夹。

1.  在新文件夹中添加一个名为 `Activity01` 的新文件夹。

1.  添加一个名为 `Flight.cs` 的新类文件。这将是一个 `Record` 类，其字段与航班数据中的字段匹配。应该使用 `Record` 类，因为它仅提供一种简单的类型来存储数据，而不是任何形式的行为。

1.  添加一个名为 `FlightLoader.cs` 的新类文件。这个类将用于下载或导入数据。`FlightLoader` 应该包含数据文件中字段索引位置列表，用于读取每一行数据并将内容分割成字符串数组，例如：

    ```cs
    public const int Agency = 0;
    public const int PaidFare = 1; 
    ```

1.  现在来实现 `FlightLoader`，使用一个 `static` 类来定义数据文件中已知字段位置索引。这将使得处理数据布局的任何未来变化更加容易。

1.  接下来，应该给 `Download` 方法传递一个 URL 和目标文件。使用 `WebClient.DownloadFile` 下载数据文件，然后委托给 `Import` 来处理下载的文件。

1.  需要添加一个 `Import` 方法。这个方法接收要导入的本地文件名（使用 `Import` 方法下载）并返回一个 `Flight` 记录列表。

1.  添加一个名为 `FilterCriteria.cs` 的类文件。这个文件应该包含一个 `FilterCriteriaType` `enum` 定义。您将提供基于航班类、出发地和目的地属性的过滤器，因此 `FilterCriteriaType` 应该代表这些属性中的每一个。

1.  现在，对于主要的过滤类，添加一个名为 `FlightQuery.cs` 的新类文件。构造函数将传递一个 `FlightLoader` 实例。在其内部，创建一个名为 `_flights` 的列表来包含通过 `FlightLoader` 导入的数据。创建一个名为 `_filters` 的 `List<FilterCriteria>` 实例，代表每次用户指定新的过滤条件时添加的每个标准项。

1.  `FlightLoader` 的 `Import` 和 `Download` 方法应该在启动时由控制台调用，以便通过 `_loader` 实例处理之前下载的数据。

1.  创建一个 `Count` 变量，它返回已导入的航班记录数。

1.  当用户指定要添加的过滤器时，控制台将调用 `AddFilter`，传递一个 `enum` 来定义标准类型和要过滤的字符串值。

1.  `RunQuery` 是主要方法，它返回符合用户标准的航班。您需要使用内置的 `StringComparer.InvariantCultureIgnoreCase` 比较器来确保字符串比较忽略任何大小写差异。您定义一个查询变量，它对航班调用 `Select`；目前，这将产生一个过滤后的结果集。

1.  可用的每种过滤类型都是基于字符串的，因此需要提取所有字符串项。如果有任何要过滤的项，为每种类型（`Class`、`Destination` 或 `Origin`）在查询中添加一个额外的 `Where` 调用。每个 `Where` 子句使用一个 `Contains` 断言，它检查相关的属性。

1.  接下来，添加`RunQuery`使用的两个辅助方法。`GetFiltersByType`传递代表已知类型标准类型的每个`FilterCriteriaType`枚举，并使用`.Where`方法在过滤器列表中查找这些类型。例如，如果用户添加了两个`目的地`标准，如印度和德国，这将导致返回两个字符串`India`和`Germany`。

1.  `FormatFilters`简单地将`filterValues`字符串列表连接成一个用户友好的字符串，每个项目之间用单词`OR`分隔，例如`伦敦 OR 都柏林`。

1.  现在创建主控制台应用程序。添加一个名为`Program.cs`的新类，它将允许用户输入请求并处理他们的命令。

1.  将下载的 URL 和目标文件名硬编码。

1.  创建主`FlightQuery`类，传入一个`FlightLoader`实例。如果应用程序之前已经运行过，你可以`导入`本地航班数据，如果没有，则使用`下载`。

1.  显示导入的记录摘要和可用的命令。

1.  当用户输入命令时，也可能有一个参数，例如`destination united kingdom`，其中`destination`是命令，`united kingdom`是参数。为了确定这一点，使用`IndexOf`方法查找输入中第一个空格字符的位置（如果有的话）。

1.  对于`go`命令，调用`RunQuery`并在返回的结果上使用各种聚合运算符。

1.  对于剩余的命令，根据要求清除或添加过滤器。如果指定了`清除`命令，调用查询的`ClearFilters`方法，这将清除标准项列表。

1.  如果指定了`class`过滤器命令，调用`AddFilter`指定`FilterCriteriaType.Class`枚举和字符串`Argument`。

1.  应该使用相同的模式为`起始地`和`目的地`命令。调用`AddFilter`，传入所需的`枚举`值和参数。

控制台输出应类似于以下内容，这里列出用户可用的命令：

```cs
Commands: go | clear | class value | origin value | destination value
```

1.  用户应该能够添加两个类过滤器，对于`经济舱`或`商务舱`（所有字符串比较应不区分大小写），如下面的代码片段所示：

    ```cs
    Enter a command:class economy
    Added filter: Class=economy
    Enter a command:class Business Class
    Added filter: Class=business class
    ```

1.  同样，用户应该能够添加一个起始地过滤器，如下所示（此示例为`伦敦`）：

    ```cs
    Enter a command:origin london
    Added filter: Origin=london
    ```

1.  添加目的地过滤器应如下所示（此示例为`苏黎世`）：

    ```cs
    Enter a command:destination zurich
    Added filter: Destination=zurich
    ```

1.  输入`go`应显示所有指定的过滤器摘要，然后是匹配过滤器的航班结果：

    ```cs
    Enter a command:go
    Classes: economy OR business class
    Destinations: zurich
    Origins: london
    Results: Count=16, Avg=266.92, Min=-74.71, Max=443.49
    ```

    注意

    本活动的解决方案可在[`packt.link/qclbF`](https://packt.link/qclbF)找到。

# 摘要

在本章中，您了解了`IEnumerable`和`ICollection`接口构成了.NET 数据结构的基础，以及它们如何用于存储多个项目。您根据每个集合的用途创建了不同类型的集合。您了解到`List`集合最广泛地用于存储项目集合，尤其是当元素数量在编译时未知时。您还看到`Stack`和`Queue`类型允许以受控的方式处理项目顺序，而`HashSet`提供基于集合的处理，而`Dictionary`则使用键标识符存储唯一值。

您随后通过使用 LINQ 查询表达式和查询运算符进一步探索了数据结构，展示了如何将查询应用于数据，并说明查询可以根据过滤需求在运行时进行更改。您对数据进行排序和分区，并看到如何使用查询运算符和查询表达式实现类似操作，每个都根据上下文提供偏好和灵活性。

在下一章中，您将看到如何使用并行和异步代码一起运行复杂或长时间运行的操作。
