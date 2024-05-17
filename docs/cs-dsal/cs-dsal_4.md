# 第四章：字典和集

本章将重点介绍与字典和集相关的数据结构。正确应用这些数据结构可以将键映射到值，并进行快速查找，以及对集合进行各种操作。为了简化对字典和集的理解，本章将包含插图和代码片段。

在本章的前几部分，您将学习字典的非泛型和泛型版本，即由键和值组成的一对集合。然后，还将介绍字典的排序变体。您还将看到字典和列表之间的一些相似之处。

本章的剩余部分将向您展示如何使用哈希集，以及名为“排序”集的变体。是否可能有一个“排序”集？在阅读最后一节时，您将了解如何理解这个主题。

本章将涵盖以下主题：

+   哈希表

+   字典

+   排序字典

+   哈希集

+   “排序”集

# 哈希表

让我们从第一个数据结构开始，即**哈希表**，也称为**哈希映射**。它允许将键**映射**到特定值，如下图所示：

![](img/e874bb03-d53d-432d-a1b6-cfb25bae7204.png)

哈希表最重要的假设之一是可以非常快速地查找基于**Key**的**Value**，这应该是*O(1)*操作。为了实现这一目标，使用了**哈希函数**。它将**Key**生成一个桶的索引，**Value**可以在其中找到。

因此，如果您需要查找键的值，您不需要遍历集合中的所有项，因为您可以使用哈希函数轻松定位适当的桶并获取值。由于哈希表的出色性能，在许多现实世界的应用程序中经常使用这样的数据结构，例如用于关联数组、数据库索引或缓存系统。

正如您所看到的，哈希函数的作用至关重要，理想情况下应该为所有键生成唯一的结果。然而，可能会为不同的键生成相同的结果。这种情况被称为**哈希冲突**，需要处理。

从头开始实现哈希表的实现似乎相当困难，特别是涉及使用哈希函数、处理哈希冲突以及将特定键分配给桶。幸运的是，在 C#语言中开发应用程序时可以使用合适的实现，而且使用起来非常简单。

哈希表相关类有两个变体，即非泛型（`Hashtable`）和泛型（`Dictionary`）。第一个在本节中描述，而另一个在下一节中描述。如果可以使用强类型的泛型版本，我强烈建议使用它。

让我们来看看`System.Collections`命名空间中的`Hashtable`类。如前所述，它存储了一组成对的集合，每个集合包含一个键和一个值。一对由`DictionaryEntry`实例表示。

您可以轻松地使用索引器访问特定元素。由于`Hashtable`类是与哈希表相关类的非泛型变体，您需要将返回的结果转换为适当的类型（例如`string`），如下所示：

```cs
string value = (string)hashtable["key"]; 
```

类似地，您可以设置值：

```cs
hashtable["key"] = "value"; 
```

值得一提的是，`null`值对于元素的`key`是不正确的，但对于元素的`value`是可以接受的。

除了索引器之外，该类还配备了一些属性，可以获取存储的元素数量（`Count`），以及返回键或值的集合（分别为`Keys`和`Values`）。此外，您可以使用一些可用的方法，例如添加新元素（`Add`），删除元素（`Remove`），删除所有元素（`Clear`），以及检查集合是否包含特定键（`Contains`和`ContainsKey`）或给定值（`ContainsValue`）。

如果要从哈希表中获取所有条目，可以使用`foreach`循环来迭代存储在集合中的所有对，如下所示：

```cs
foreach (DictionaryEntry entry in hashtable) 
{ 
    Console.WriteLine($"{entry.Key} - {entry.Value}"); 
} 
```

循环中使用的变量具有`DictionaryEntry`类型。因此，您需要使用其`Key`和`Value`属性分别访问键和值。

您可以在[`msdn.microsoft.com/library/system.collections.hashtable.aspx`](https://msdn.microsoft.com/library/system.collections.hashtable.aspx)找到有关`Hashtable`类的更多信息。

在这个简短的介绍之后，现在是时候看一个例子了。

# 示例-电话簿

例如，您将创建一个电话簿应用程序。`Hashtable`类将用于存储条目，其中人名是键，电话号码是值，如下图所示：

![](img/b596dbfa-5ace-48a2-be5f-68657c98bb64.png)

该程序将演示如何向集合中添加元素，检查存储的项目数量，遍历所有项目，检查是否存在具有给定键的元素，以及如何基于键获取值。

此处呈现的整个代码应放在`Program`类的`Main`方法的主体中。首先，让我们创建`Hashtable`类的新实例，并使用一些条目对其进行初始化，如下面的代码所示：

```cs
Hashtable phoneBook = new Hashtable() 
{ 
    { "Marcin Jamro", "000-000-000" }, 
    { "John Smith", "111-111-111" } 
}; 
phoneBook["Lily Smith"] = "333-333-333"; 
```

您可以以各种方式向集合中添加元素，例如在创建类的新实例时（在前面的示例中为`Marcin Jamro`和`John Smith`的电话号码），通过使用索引器（`Lily Smith`），以及使用`Add`方法（`Mary Fox`），如下面的代码部分所示：

```cs
try 
{ 
    phoneBook.Add("Mary Fox", "222-222-222"); 
} 
catch (ArgumentException) 
{ 
    Console.WriteLine("The entry already exists."); 
} 
```

如您所见，`Add`方法的调用位于`try-catch`语句中。为什么？答案很简单——您不能添加具有相同键的多个元素，在这种情况下会抛出`ArgumentException`。为了防止应用程序崩溃，使用`try-catch`语句，并在控制台中显示适当的消息，通知用户情况。

当您使用索引器为特定键设置值时，如果已经存在具有给定键的项目，它不会抛出任何异常。在这种情况下，将更新此元素的值。

在代码的下一部分中，您将遍历集合中的所有对，并在控制台中呈现结果。当没有项目时，将向用户呈现附加信息，如下面的代码片段所示：

```cs
Console.WriteLine("Phone numbers:"); 
if (phoneBook.Count == 0) 
{ 
    Console.WriteLine("Empty"); 
} 
else 
{ 
    foreach (DictionaryEntry entry in phoneBook) 
    { 
        Console.WriteLine($" - {entry.Key}: {entry.Value}"); 
    } 
} 
```

您可以使用`Count`属性检查集合中是否没有元素，并将其值与`0`进行比较。通过`foreach`循环的可用性，遍历所有对的方式变得更加简单。但是，您需要记住，`Hashtable`类中的单个对由`DictionaryEntry`实例表示，您可以使用`Key`和`Value`属性访问其键和值。

最后，让我们看看如何检查特定键是否存在于集合中，以及如何获取其值。第一个任务可以通过调用`Contains`方法来完成，该方法返回一个值，指示是否存在合适的元素（`true`）或不存在（`false`）。另一个任务（获取值）使用索引器，并且需要将返回的值转换为适当的类型（在本例中为`string`）。这个要求是由哈希表相关类的非泛型版本引起的。代码如下：

```cs
Console.WriteLine(); 
Console.Write("Search by name: "); 
string name = Console.ReadLine(); 
if (phoneBook.Contains(name)) 
{ 
    string number = (string)phoneBook[name]; 
    Console.WriteLine($"Found phone number: {number}"); 
} 
else 
{ 
    Console.WriteLine("The entry does not exist."); 
} 
```

您的第一个使用哈希表的程序已经准备好了！启动后，您将收到类似以下的结果：

```cs
    Phone numbers:
     - John Smith: 111-111-111
     - Mary Fox: 222-222-222
     - Lily Smith: 333-333-333
     - Marcin Jamro: 000-000-000

    Search by name: Mary Fox
    Found phone number: 222-222-222

```

值得注意的是，使用`Hashtable`类存储的键值对的顺序与它们添加或键的顺序不一致。因此，如果需要呈现排序后的结果，您需要自行对元素进行排序，或者使用另一个数据结构，即稍后在本书中描述的`SortedDictionary`。

然而，现在让我们来看一下在 C#中开发时最常用的类之一，即`Dictionary`，它是哈希表相关类的泛型版本。

# 字典

在上一节中，您了解了`Hashtable`类作为哈希表相关类的非泛型变体。但是，它有一个重要的限制，因为它不允许您指定键和值的类型。`DictionaryEntry`类的`Key`和`Value`属性都是`object`类型。因此，即使所有键和值都具有相同的类型，您仍需要执行装箱和拆箱操作。

如果要使用强类型变体，可以使用`Dictionary`泛型类，这是本章节的主要内容。

首先，在创建`Dictionary`类的实例时，您应该指定两种类型，即键的类型和值的类型。此外，可以使用以下代码定义字典的初始内容：

```cs
Dictionary<string, string> dictionary = 
    new Dictionary<string, string> 
{ 
    { "Key 1", "Value 1" }, 
    { "Key 2", "Value 2" } 
}; 
```

在上面的代码中，创建了`Dictionary`类的一个新实例。它存储基于`string`的键和值。默认情况下，字典中存在两个条目，即键`Key 1`和`Key 2`。它们的值分别是`Value 1`和`Value 2`。

与`Hashtable`类类似，您也可以使用索引器来访问集合中的特定元素，如下面的代码行所示：

```cs
string value = dictionary["key"]; 
```

值得注意的是，不需要将类型转换为`string`类型，因为`Dictionary`是哈希表相关类的强类型版本。因此，返回的值已经具有正确的类型。

如果集合中不存在具有给定键的元素，则会抛出`KeyNotFoundException`。为了避免问题，您可以选择以下之一：

+   将代码行放在`try-catch`块中

+   检查元素是否存在（通过调用`ContainsKey`）

+   使用`TryGetValue`方法

您可以使用索引器添加新元素或更新现有元素的值，如下面的代码行所示：

```cs
dictionary["key"] = "value"; 
```

与非泛型变体类似，`key`不能等于`null`，但`value`可以，当然，如果允许存储在集合中的值的类型。此外，获取元素的值、添加新元素或更新现有元素的性能接近*O(1)*操作。

`Dictionary`类配备了一些属性，可以获取存储元素的数量（`Count`），以及返回键或值的集合（分别是`Keys`和`Values`）。此外，您可以使用可用的方法，例如添加新元素（`Add`），删除项目（`Remove`），删除所有元素（`Clear`），以及检查集合是否包含特定键（`ContainsKey`）或给定值（`ContainsValue`）。您还可以使用`TryGetValue`方法尝试获取给定键的值并返回它（如果元素存在），否则返回`null`。

虽然通过给定键返回值（使用索引器或`TryGetValue`）和检查给定键是否存在（`ContainsKey`）的场景接近*O(1)*操作，但检查集合是否包含给定值（`ContainsValue`）的过程是*O(n)*操作，并且需要您搜索整个集合以查找特定值。

如果要遍历集合中存储的所有对，可以使用`foreach`循环。但是，循环中使用的变量是`KeyValuePair`泛型类的实例，具有`Key`和`Value`属性，允许您访问键和值。`foreach`循环显示在以下代码片段中：

```cs
foreach (KeyValuePair<string, string> pair in dictionary) 
{ 
    Console.WriteLine($"{pair.Key} - {pair.Value}"); 
} 
```

您还记得上一章中一些类的线程安全版本吗？如果记得，那么在`Dictionary`类的情况下，情况看起来与`ConcurrentDictionary`类相当相似，因为`System.Collections.Concurrent`命名空间中提供了`ConcurrentDictionary`类。它配备了一组方法，例如`TryAdd`、`TryUpdate`、`AddOrUpdate`和`GetOrAdd`。

您可以在[`msdn.microsoft.com/library/xfhwa508.aspx`](https://msdn.microsoft.com/library/xfhwa508.aspx)找到有关`Dictionary`泛型类的更多信息，而有关线程安全替代方案`ConcurrentDictionary`的详细信息则显示在[`msdn.microsoft.com/library/dd287191.aspx`](https://msdn.microsoft.com/library/dd287191.aspx)。

让我们开始编码！在接下来的部分，您将找到两个展示字典的示例。

# 示例-产品位置

第一个示例是帮助商店员工找到产品应放置的位置的应用程序。假设每个员工都有一部手机，上面安装了您的应用程序，用于扫描产品的代码，应用程序会告诉他们产品应放置在**A1**或**C9**区域。听起来很有趣，不是吗？

由于商店中的产品数量通常非常庞大，因此有必要快速找到结果。因此，产品的数据以及其位置将存储在哈希表中，使用泛型`Dictionary`类。键将是条形码，而值将是区域代码，如下图所示：

![](img/47f0b01a-098f-4b52-b227-d3d1daf4b0ce.png)

让我们看一下应该添加到`Program`类的`Main`方法中的代码。首先，您需要创建一个新的集合，并添加一些数据：

```cs
Dictionary<string, string> products = 
    new Dictionary<string, string> 
{ 
    { "5900000000000", "A1" }, 
    { "5901111111111", "B5" }, 
    { "5902222222222", "C9" } 
}; 
products["5903333333333"] = "D7"; 
```

代码显示了向集合中添加元素的两种方法，即在创建类的新实例时传递它们的数据和使用索引器。还存在第三种解决方案，使用`Add`方法，如代码的以下部分所示：

```cs
try 
{ 
    products.Add("5904444444444", "A3"); 
} 
catch (ArgumentException) 
{ 
    Console.WriteLine("The entry already exists."); 
} 
```

在`Hashtable`类的情况下提到，如果您想要添加与集合中已存在的元素具有相同键的元素，则会抛出`ArgumentException`。您可以通过使用`try-catch`块来防止应用程序崩溃。

在代码的下一部分中，您会展示系统中所有可用产品的数据。为此，您使用`foreach`循环，但在此之前，您要检查字典中是否有任何元素。如果没有，则向用户呈现适当的消息。否则，控制台中显示所有对的键和值。值得一提的是，在`foreach`循环中的变量类型是`KeyValuePair<string, string>`，因此其`Key`和`Value`属性是`string`类型，而不是`object`类型，与非泛型变体的情况相同。代码如下所示：

```cs
Console.WriteLine("All products:"); 
if (products.Count == 0) 
{ 
    Console.WriteLine("Empty"); 
} 
else 
{ 
    foreach (KeyValuePair<string, string> product in products) 
    { 
        Console.WriteLine($" - {product.Key}: {product.Value}"); 
    } 
}
```

最后，让我们看一下代码的一部分，该代码使得可以通过其条形码找到产品的位置。为此，您使用`TryGetValue`来检查元素是否存在。如果是，控制台中会显示带有目标位置的消息。否则，会显示其他信息。重要的是，`TryGetValue`方法使用`out`参数来返回找到的元素的值。代码如下：

```cs
Console.WriteLine(); 
Console.Write("Search by barcode: "); 
string barcode = Console.ReadLine(); 
if (products.TryGetValue(barcode, out string location)) 
{ 
    Console.WriteLine($"The product is in the area {location}."); 
} 
else 
{ 
    Console.WriteLine("The product does not exist."); 
} 
```

运行程序时，您将看到商店中所有产品的列表，并且程序会要求您输入条形码。输入后，您将收到带有区域代码的消息。控制台中显示的结果将类似于以下内容：

```cs
    All products:
     - 5900000000000: A1
     - 5901111111111: B5
     - 5902222222222: C9
     - 5903333333333: D7
     - 5904444444444: A3

    Search by barcode: 5902222222222
    The product is in the area C9.
```

您刚刚完成了第一个示例！让我们继续到下一个。

# 示例-用户详细信息

第二个示例将向您展示如何在字典中存储更复杂的数据。在这种情况下，您将创建一个应用程序，根据用户的标识符显示用户的详细信息，如下图所示：

![](img/12009a61-2e0f-4ab3-9a8a-de5495759aff.png)

程序应该以三个用户的数据开始。您应该能够输入标识符并查看找到的用户的详细信息。当然，应该通过在控制台中呈现适当的信息来处理给定用户不存在的情况。

首先，让我们添加`Employee`类，它只存储员工的数据，即名字、姓氏和电话号码。代码如下：

```cs
public class Employee 
{ 
    public string FirstName { get; set; } 
    public string LastName { get; set; } 
    public string PhoneNumber { get; set; } 
} 
```

下面的修改将在`Program`类的`Main`方法中执行。在这里，您创建了`Dictionary`类的一个新实例，并使用`Add`方法添加了三个员工的数据，如下面的代码片段所示：

```cs
Dictionary<int, Employee> employees =  
    new Dictionary<int, Employee>(); 
employees.Add(100, new Employee() { FirstName = "Marcin",  
    LastName = "Jamro", PhoneNumber = "000-000-000" }); 
employees.Add(210, new Employee() { FirstName = "Mary",  
    LastName = "Fox", PhoneNumber = "111-111-111" }); 
employees.Add(303, new Employee() { FirstName = "John",  
    LastName = "Smith", PhoneNumber = "222-222-222" }); 
```

最有趣的操作是在以下`do-while`循环中执行的：

```cs
bool isCorrect = true; 
do 
{ 
    Console.Write("Enter the employee identifier: "); 
    string idString = Console.ReadLine(); 
    isCorrect = int.TryParse(idString, out int id); 
    if (isCorrect) 
    { 
        Console.ForegroundColor = ConsoleColor.White; 
        if (employees.TryGetValue(id, out Employee employee)) 
        { 
            Console.WriteLine("First name: {1}{0}Last name:  
                {2}{0}Phone number: {3}", 
                Environment.NewLine, 
                employee.FirstName, 
                employee.LastName, 
                employee.PhoneNumber); 
        } 
        else 
        { 
            Console.WriteLine("The employee with the given  
                identifier does not exist."); 
        } 
        Console.ForegroundColor = ConsoleColor.Gray; 
    } 
} 
while (isCorrect); 
```

首先，用户被要求输入员工的标识符，然后将其解析为整数值。如果此操作成功完成，则使用`TryGetValue`方法尝试获取用户的详细信息。如果找到用户，即`TryGetValue`返回`true`，则在控制台中呈现详细信息。否则，显示`“给定标识符的员工不存在。”`消息。循环执行，直到提供的标识符无法解析为整数值为止。

当您运行应用程序并输入一些数据时，您将收到以下结果：

```cs
    Enter the employee identifier: 100
    First name: Marcin
    Last name: Jamro
    Phone number: 000-000-000
    Enter the employee identifier: 500
    The employee with the given identifier does not exist.
```

就是这样！您刚刚完成了两个示例，展示了如何在 C#语言中开发应用程序时使用字典。

然而，在关于`Hashtable`类的部分提到了另一种字典，即有序字典。您是否有兴趣了解它的作用以及如何在程序中使用它？如果是的话，让我们继续到下一节。

# 有序字典

与哈希表相关的类的非泛型和泛型变体都不保留元素的顺序。因此，如果您需要按键排序的方式呈现来自集合的数据，您需要在呈现之前对它们进行排序。但是，您可以使用另一种数据结构，**有序字典**，来解决这个问题，并始终保持键的排序。因此，您可以在必要时轻松获取排序后的集合。

有序字典实现为`SortedDictionary`泛型类，位于`System.Collections.Generic`命名空间中。您可以在创建`SortedDictionary`类的新实例时指定键和值的类型。此外，该类包含与`Dictionary`类类似的属性和方法。

首先，您可以使用索引器访问集合中的特定元素，如下面的代码行所示：

```cs
string value = dictionary["key"]; 
```

您应该确保元素存在于集合中。否则，将抛出`KeyNotFoundException`。

您可以添加新元素或更新现有元素的值，如下所示的代码：

```cs
dictionary["key"] = "value"; 
```

与`Dictionary`类类似，键不能等于`null`，但值可以，当然，如果允许存储在集合中的值的类型允许的话。

该类配备了一些属性，可以获取存储元素的数量（`Count`），以及返回键和值的集合（`Keys`和`Values`）。此外，您可以使用可用的方法，例如添加新元素（`Add`），删除项目（`Remove`），删除所有元素（`Clear`），以及检查集合是否包含特定键（`ContainsKey`）或给定值（`ContainsValue`）。您可以使用`TryGetValue`方法尝试获取给定键的值并返回它（如果元素存在），否则返回`null`。

如果您想要遍历集合中存储的所有键值对，可以使用`foreach`循环。循环中使用的变量是`KeyValuePair`泛型类的实例，具有`Key`和`Value`属性，允许您访问键和值。

尽管自动排序有优势，但与`Dictionary`相比，`SortedDictionary`类在性能上有一些缺点，因为检索、插入和删除都是*O(log n)*操作，其中*n*是集合中的元素数量，而不是*O(1)*。此外，`SortedDictionary`与第二章中描述的`SortedList`非常相似，*数组和列表*。然而，它们在与内存相关和性能相关的结果上有所不同。这两个类的检索都是*O(log n)*操作，但对于未排序的数据，`SortedDictionary`的插入和删除是*O(log n)*，而`SortedList`是*O(n)*。当然，`SortedDictionary`需要比`SortedList`更多的内存。正如您所看到的，选择合适的数据结构并不是一件容易的事，您应该仔细考虑特定数据结构将被使用的场景，并考虑其优缺点。

您可以在[`msdn.microsoft.com/library/f7fta44c.aspx`](https://msdn.microsoft.com/library/f7fta44c.aspx)找到关于`SortedDictionary`泛型类的更多信息。

让我们通过创建一个示例来看看排序字典的实际操作。

# 示例-定义

例如，您可以创建一个简单的百科全书，可以添加条目，并显示其完整内容。百科全书可以包含数百万条目，因此至关重要的是为其用户提供按正确顺序浏览条目的可能性，按键的字母顺序排列，以及快速找到条目。因此，在这个例子中，排序字典是一个很好的选择。

百科全书的概念如下图所示：

![](img/9915bf6c-68cd-49a8-8a8b-98078a4c5462.png)

当程序启动时，它会显示一个简单的菜单，包括两个选项，即`[a] - add`和`[l] - list`。按下*A*键后，应用程序会要求您输入条目的名称和解释。如果提供的数据是正确的，新条目将被添加到百科全书中。如果用户按下*L*键，则按键排序的所有条目数据将显示在控制台中。当按下其他键时，会显示额外的确认信息，如果确认，则程序退出。

让我们来看看代码，它应该放在`Program`类的`Main`方法的主体中：

```cs
SortedDictionary<string, string> definitions =  
    new SortedDictionary<string, string>(); 
do 
{ 
    Console.Write("Choose an option ([a] - add, [l] - list): "); 
    ConsoleKeyInfo keyInfo = Console.ReadKey(); 
    Console.WriteLine(); 
    if (keyInfo.Key == ConsoleKey.A) 
    { 
        Console.ForegroundColor = ConsoleColor.White; 
        Console.Write("Enter the name: "); 
        string name = Console.ReadLine(); 
        Console.Write("Enter the explanation: "); 
        string explanation = Console.ReadLine(); 
        definitions[name] = explanation; 
        Console.ForegroundColor = ConsoleColor.Gray; 
    } 
    else if (keyInfo.Key == ConsoleKey.L) 
    { 
        Console.ForegroundColor = ConsoleColor.White; 
        foreach (KeyValuePair<string, string> definition  
            in definitions) 
        { 
            Console.WriteLine($"{definition.Key}:  
                {definition.Value}"); 
        } 
        Console.ForegroundColor = ConsoleColor.Gray; 
    } 
    else 
    { 
        Console.ForegroundColor = ConsoleColor.White; 
        Console.WriteLine("Do you want to exit the program?  
            Press [y] (yes) or [n] (no)."); 
        Console.ForegroundColor = ConsoleColor.Gray; 
        if (Console.ReadKey().Key == ConsoleKey.Y) 
        { 
            break; 
        } 
    } 
} 
while (true); 
```

首先，创建了`SortedDictionary`类的新实例，它表示具有基于`string`的键和基于`string`的值的一组对。然后，使用无限的`do-while`循环。在其中，程序会等待用户按下任意键。如果是*A*键，程序将从用户输入的值中获取条目的名称和解释。然后，使用索引器将新条目添加到字典中。因此，如果具有相同键的条目已经存在，它将被更新。如果按下*L*键，则使用`foreach`循环显示所有输入的条目。当按下其他键时，会向用户显示另一个问题，并等待确认。如果用户按下*Y*，则跳出循环。

当运行程序时，您可以输入一些条目，并将它们显示出来。控制台的结果如下所示：

```cs
    Choose an option ([a] - add, [l] - list): a
    Enter the name: Zakopane
    Enter the explanation: a city located in Tatra mountains in Poland
    Choose an option ([a] - add, [l] - list): a
    Enter the name: Rzeszow
    Enter the explanation: a capital of the Subcarpathian voivodeship 
    in Poland
    Choose an option ([a] - add, [l] - list): a
    Enter the name: Warszawa
    Enter the explanation: a capital city of Poland
    Choose an option ([a] - add, [l] - list): a
    Enter the name: Lancut
    Enter the explanation: a city located near Rzeszow with 
    a beautiful castle
    Choose an option ([a] - add, [l] - list): l
    Lancut: a city located near Rzeszow with a beautiful castle
    Rzeszow: a capital of the Subcarpathian voivodeship in Poland
    Warszawa: a capital city of Poland
    Zakopane: a city located in Tatra mountains in Poland
    Choose an option ([a] - add, [l] - list): q
    Do you want to exit the program? Press [y] (yes) or [n] (no).
    yPress any key to continue . . .

```

到目前为止，您已经学习了三个与字典相关的类，分别是`Hashtable`、`Dictionary`和`SortedDictionary`。它们都有一些特定的优势，并且可以在各种场景中使用。为了更容易理解它们，我们提供了一些示例，并附有详细的解释。

然而，你知道还有一些其他只存储键而没有值的数据结构吗？想要了解更多吗？如果是的话，让我们继续到下一节。

# 哈希集

在一些算法中，有必要对具有不同数据的集合执行操作。但是，什么是**集合**？集合是一组不重复元素的集合，没有重复的元素，也没有特定的顺序。因此，你只能知道给定的元素是否在集合中。集合与数学模型和操作紧密相关，如并集、交集、差集和对称差。

集合可以存储各种数据，如整数或字符串值，如下图所示。当然，你也可以创建一个包含用户定义类实例的集合，并随时向集合中添加和删除元素。

![](img/711ea288-1b69-41e1-bc10-ce9441ca8978.png)

在看到集合的实际操作之前，值得提醒一下可以对两个集合**A**和**B**执行的一些基本操作。让我们从并集和交集开始，如下图所示。如你所见，**并集**（左侧显示为**A∪B**）是一个包含属于**A**或**B**的所有元素的集合。**交集**（右侧显示为**A∩B**）仅包含属于**A**和**B**的元素：

![](img/d5053aab-bdec-4534-a3a4-65677880f858.png)

另一个常见的操作是**集合减法**。**A \ B**的结果集包含属于**A**而不属于**B**的元素。在下面的示例中，分别呈现了**A \ B**和**B \ A**：

![](img/7c8d365f-f84c-4d1c-abaf-abe4805fad0e.png)

在对集合执行操作时，还值得提到**对称差**，如下图左侧所示的**A ∆ B**。最终集合可以解释为两个集合的并集，即（**A \ B**）和（**B \ A**）。因此，它包含属于只属于一个集合的元素，要么是**A**，要么是**B**。属于两个集合的元素被排除在结果之外：

![](img/ba83077a-cb31-42c0-bc5b-2df29ff3445e.png)

另一个重要的主题是集合之间的**关系**。如果**B**的每个元素也属于**A**，那么**B**是**A**的**子集**，如前图中右侧所示。同时，**A**是**B**的**超集**。此外，如果**B**是**A**的子集，但**B**不等于**A**，那么**B**是**A**的**真子集**，而**A**是**B**的**真超集**。

在 C#语言中开发应用程序时，你可以从`System.Collections.Generic`命名空间中的`HashSet`类提供的高性能操作中受益。该类包含一些属性，包括返回集合中元素数量的`Count`。此外，你可以使用许多方法来执行集合操作，如下面所述。

第一组方法使得可以修改当前集合（调用方法的集合）以创建以下集合，其中传递的集合作为参数：

+   并集（`UnionWith`）

+   交集（`IntersectWith`）

+   差集（`ExceptWith`）

+   对称差（`SymmetricExceptWith`）

你还可以检查两个集合之间的关系，例如检查调用方法的当前集合是否是：

+   传递的集合的子集（`IsSubsetOf`）

+   传递的集合的超集（`IsSupersetOf`）

+   传递的集合的真子集（`IsProperSubsetOf`）

+   传递的集合的真超集（`IsProperSupersetOf`）

此外，你可以验证两个集合是否包含相同的元素（`SetEquals`），或者两个集合是否至少有一个公共元素（`Overlaps`）。

除了上述操作，您还可以向集合中添加新元素（`Add`），删除特定元素（`Remove`）或删除所有元素（`Clear`），以及检查给定元素是否存在于集合中（`Contains`）。

您可以在[`msdn.microsoft.com/library/bb359438.aspx`](https://msdn.microsoft.com/library/bb359438.aspx)找到有关`HashSet`泛型类的更多信息。

在这个介绍之后，尝试将学到的信息付诸实践是一个好主意。因此，让我们继续进行两个示例，它们将向您展示如何在应用程序中应用哈希集。

# 示例 - 优惠券

第一个示例代表了一个系统，用于检查一次性优惠券是否已经被使用。如果是，应向用户呈现适当的消息。否则，系统应通知用户优惠券有效，并且应标记为已使用，不能再次使用。由于优惠券数量众多，有必要选择一种数据结构，可以快速检查某个集合中是否存在元素。因此，哈希集被选择为存储已使用优惠券的标识符的数据结构。因此，您只需要检查输入的标识符是否存在于集合中。

让我们来看看应该添加到`Program`类的`Main`方法的代码。第一部分如下所示：

```cs
HashSet<int> usedCoupons = new HashSet<int>(); 
do 
{ 
    Console.Write("Enter the coupon number: "); 
    string couponString = Console.ReadLine(); 
    if (int.TryParse(couponString, out int coupon)) 
    { 
        if (usedCoupons.Contains(coupon)) 
        { 
            Console.ForegroundColor = ConsoleColor.Red; 
            Console.WriteLine("It has been already used :-("); 
            Console.ForegroundColor = ConsoleColor.Gray; 
        } 
        else 
        { 
            usedCoupons.Add(coupon); 
            Console.ForegroundColor = ConsoleColor.Green; 
            Console.WriteLine("Thank you! :-)"); 
            Console.ForegroundColor = ConsoleColor.Gray; 
        } 
    } 
    else 
    { 
        break; 
    } 
} 
while (true); 
```

首先，创建存储整数值的`HashSet`泛型类的新实例。然后，大多数操作都在`do-while`循环内执行。在这里，程序会等待用户输入优惠券标识符。如果无法解析为整数值，则跳出循环。否则，将检查集合是否已包含等于优惠券标识符的元素（使用`Contains`方法）。如果是，将呈现适当的警告信息。但是，如果不存在，则将其添加到已使用优惠券的集合中（使用`Add`方法）并通知用户。

当您跳出循环时，您只需要显示已使用优惠券的标识符的完整列表。您可以使用`foreach`循环实现此目标，遍历集合，并在控制台中写入其元素，如下面的代码所示：

```cs
Console.WriteLine(); 
Console.WriteLine("A list of used coupons:"); 
foreach (int coupon in usedCoupons) 
{ 
    Console.WriteLine(coupon); 
} 
```

现在您可以启动应用程序，输入一些数据，然后查看它的运行情况。控制台中的结果如下所示：

```cs
    Enter the coupon number: 100
    Thank you! :-)
    Enter the coupon number: 101
    Thank you! :-)
    Enter the coupon number: 500
    Thank you! :-)
    Enter the coupon number: 345
    Thank you! :-)
    Enter the coupon number: 101
    It has been already used :-(
    Enter the coupon number: l

    A list of used coupons:
    100
    101
    500
    345

```

这是第一个示例的结束。让我们继续进行下一个示例，在这个示例中，您将看到一个使用哈希集的更复杂的解决方案。

# 示例 - 游泳池

这个例子展示了一个 SPA 中心的系统，有四个游泳池，分别是休闲、比赛、温泉和儿童。每位访客都会收到一个特殊的手腕带，可以进入所有游泳池。但是，必须在进入任何游泳池时扫描手腕带，您的程序可以使用这些数据来创建各种统计数据。

在这个例子中，哈希集被选择为存储已经在每个游泳池入口扫描的手腕带的唯一编号的数据结构。将使用四个集合，每个游泳池一个，如下图所示。此外，它们将被分组在字典中，以简化和缩短代码，以及使未来的修改更容易：

![](img/5e73a997-35ca-427f-8258-207a9986b8a1.png)

为了简化测试应用程序，初始数据将被随机设置。因此，您只需要创建统计数据，即按游泳池类型统计的访客人数，最受欢迎的游泳池，至少访问过一个游泳池的人数，以及访问过所有游泳池的人数。所有统计数据将使用集合。

让我们从`PoolTypeEnum`枚举开始（在`PoolTypeEnum.cs`文件中声明），它表示可能的游泳池类型，如下面的代码所示：

```cs
public enum PoolTypeEnum 
{ 
    RECREATION, 
    COMPETITION, 
    THERMAL, 
    KIDS 
}; 
```

接下来，向`Program`类添加`random`私有静态字段。它将用于使用一些随机值填充哈希集。代码如下：

```cs
private static Random random = new Random(); 
```

然后，在`Program`类中声明`GetRandomBoolean`静态方法，返回`true`或`false`值，根据随机值。代码如下所示：

```cs
private static bool GetRandomBoolean() 
{ 
    return random.Next(2) == 1; 
} 
```

接下来的更改只需要在`Main`方法中进行。第一部分如下：

```cs
Dictionary<PoolTypeEnum, HashSet<int>> tickets =  
    new Dictionary<PoolTypeEnum, HashSet<int>>() 
{ 
    { PoolTypeEnum.RECREATION, new HashSet<int>() }, 
    { PoolTypeEnum.COMPETITION, new HashSet<int>() }, 
    { PoolTypeEnum.THERMAL, new HashSet<int>() }, 
    { PoolTypeEnum.KIDS, new HashSet<int>() } 
}; 
```

在这里，你创建了一个`Dictionary`的新实例。它包含四个条目。每个键都是`PoolTypeEnum`类型，每个值都是`HashSet<int>`类型，也就是一个包含整数值的集合。

在接下来的部分，你会用随机值填充集合，如下所示：

```cs
for (int i = 1; i < 100; i++) 
{ 
    foreach (KeyValuePair<PoolTypeEnum, HashSet<int>> type  
        in tickets) 
    { 
        if (GetRandomBoolean()) 
        { 
            type.Value.Add(i); 
        } 
    } 
}
```

为此，你使用两个循环，即`for`和`foreach`。第一个循环 100 次，模拟 100 个手环。其中有一个`foreach`循环，遍历所有可用的游泳池类型。对于每一个，你随机检查访客是否进入了特定的游泳池。通过获取一个随机的布尔值来检查。如果收到`true`，则将标识符添加到适当的集合中。`false`值表示具有给定手环号（`i`）的用户没有进入当前游泳池。

剩下的代码与生成各种统计数据有关。首先，让我们按游泳池类型呈现访客人数。这样的任务非常简单，因为你只需要遍历字典，以及写入游泳池类型和集合中的元素数量（使用`Count`属性），如下面的代码部分所示：

```cs
Console.WriteLine("Number of visitors by a pool type:"); 
foreach (KeyValuePair<PoolTypeEnum, HashSet<int>> type in tickets) 
{ 
    Console.WriteLine($" - {type.Key.ToString().ToLower()}:  
        {type.Value.Count}"); 
} 
```

接下来的部分找到了访客人数最多的游泳池。这是使用 LINQ 及其方法执行的，即：

+   `OrderByDescending`按集合中元素的数量降序排序元素

+   `Select`来选择游泳池类型

+   `FirstOrDefault`来获取第一个结果

然后，你只需呈现结果。做这件事的代码如下所示：

```cs
PoolTypeEnum maxVisitors = tickets 
    .OrderByDescending(t => t.Value.Count) 
    .Select(t => t.Key) 
    .FirstOrDefault(); 
Console.WriteLine($"Pool '{maxVisitors.ToString().ToLower()}'  
    was the most popular.");
```

然后，你需要获取至少访问了一个游泳池的人数。你可以通过创建所有集合的并集并获取最终集合的计数来执行此任务。首先，创建一个新的集合，并用有关休闲游泳池的标识符填充它。在代码的下面几行中，你调用`UnionWith`方法创建与以下三个集合的并集。代码的这部分如下所示：

```cs
HashSet<int> any =  
    new HashSet<int>(tickets[PoolTypeEnum.RECREATION]); 
any.UnionWith(tickets[PoolTypeEnum.COMPETITION]); 
any.UnionWith(tickets[PoolTypeEnum.THERMAL]); 
any.UnionWith(tickets[PoolTypeEnum.KIDS]); 
Console.WriteLine($"{any.Count} people visited at least  
    one pool."); 
```

最后的统计数据是在 SPA 中心一次访问中访问了所有游泳池的人数。要执行这样的计算，你只需要创建所有集合的交集，并获取最终集合的计数。为此，让我们创建一个新的集合，并用有关休闲游泳池的标识符填充它。然后，调用`IntersectWith`方法创建与以下三个集合的交集。最后，使用`Count`属性获取集合中的元素数量，并呈现结果，如下所示：

```cs
HashSet<int> all =  
    new HashSet<int>(tickets[PoolTypeEnum.RECREATION]); 
all.IntersectWith(tickets[PoolTypeEnum.COMPETITION]); 
all.IntersectWith(tickets[PoolTypeEnum.THERMAL]); 
all.IntersectWith(tickets[PoolTypeEnum.KIDS]); 
Console.WriteLine($"{all.Count} people visited all pools."); 
```

就是这样！当你运行应用程序时，你可能会收到类似以下的结果：

```cs
 Number of visitors by a pool type:
     - recreation: 54
     - competition: 44
     - thermal: 48
     - kids: 51

 Pool 'recreation' was the most popular.
 93 people visited at least one pool.
 5 people visited all pools.
```

你刚刚完成了两个关于哈希集的例子。尝试修改代码并添加新功能是了解这种数据结构的更好方法。当你准备好学习下一个数据结构时，让我们继续阅读。

# “排序”集合

前面描述的`HashSet`类可以被理解为一个只存储键而没有值的字典。所以，如果有`SortedDictionary`类，也许还有`SortedSet`类？确实有！但是，一个集合可以被“排序”吗？为什么“排序”一词用引号括起来？答案很简单——根据定义，一个集合存储一组不重复的对象，没有重复的元素，也没有特定的顺序。如果一个集合不支持顺序，它怎么能被“排序”呢？因此，“排序”集合可以被理解为`HashSet`和`SortedList`的组合，而不是一个集合本身。

如果您想要一个排序的不重复元素集合，可以使用“sorted”集合。适当的类名为`SortedSet`，并且位于`System.Collections.Generic`命名空间中。它具有一组方法，类似于已经描述的`HashSet`类的方法，例如`UnionWith`，`IntersectWith`，`ExceptWith`，`SymmetricExceptWith`，`Overlaps`，`IsSubsetOf`，`IsSupersetOf`，`IsProperSubsetOf`和`IsProperSupersetOf`。但是，它还包含用于返回最小值和最大值（分别为`Min`和`Max`）的附加属性。还值得一提的是`GetViewBetween`方法，它返回一个具有给定范围内的值的`SortedSet`实例。

您可以在[`msdn.microsoft.com/library/dd412070.aspx`](https://msdn.microsoft.com/library/dd412070.aspx)找到有关`SortedSet`泛型类的更多信息。

让我们继续进行一个简单的示例，看看如何在代码中使用“sorted”集合。

# 示例 - 删除重复项

例如，您将创建一个简单的应用程序，从名称列表中删除重复项。当然，名称的比较应该是不区分大小写的，因此不允许在同一集合中同时拥有`"Marcin"`和`"marcin"`。

要查看如何实现此目标，让我们将以下代码添加为`Program`类中`Main`方法的主体：

```cs
List<string> names = new List<string>() 
{ 
    "Marcin", 
    "Mary", 
    "James", 
    "Albert", 
    "Lily", 
    "Emily", 
    "marcin", 
    "James", 
    "Jane" 
}; 
SortedSet<string> sorted = new SortedSet<string>( 
    names, 
    Comparer<string>.Create((a, b) =>  
        a.ToLower().CompareTo(b.ToLower()))); 
foreach (string name in sorted) 
{ 
    Console.WriteLine(name); 
} 
```

首先，创建一个包含九个元素的名称列表，并初始化，包括`"Marcin"`和`"marcin"`。然后，创建`SortedSet`类的新实例，传递两个参数，即名称列表和不区分大小写的比较器。最后，只需遍历集合以在控制台中写入名称。

运行应用程序后，您将看到以下结果：

```cs
    Albert
    Emily
    James
    Jane
    Lily
    Marcin
    Mary

```

这是本章中展示的最后一个例子。因此，让我们继续进行总结。

# 总结

本书的第四章着重介绍了哈希表、字典和集合。所有这些集合都是有趣的数据结构，可以在各种场景中使用。通过详细描述和示例介绍这些集合，您已经看到选择适当的数据结构并不是一项微不足道的任务，需要分析与性能相关的主题，因为其中一些在检索值方面运行更好，而另一些则促进数据的添加和删除。

首先，您学习了如何使用哈希表的两个变体，即非泛型（`Hashtable`类）和泛型（`Dictionary`）。这些的巨大优势是基于键进行值查找的非常快速，接近*O(1)*的操作。为了实现这个目标，使用了哈希函数。此外，已经介绍了排序字典作为解决集合中无序项目问题并始终保持键排序的有趣解决方案。

随后，介绍了高性能解决方案的集合操作。它使用`HashSet`类，表示一个没有重复元素和特定顺序的对象集合。该类使得可以对集合执行各种操作，如并集、交集、差集和对称差。然后，介绍了“sorted”集合（`SortedSet`类）的概念，作为一个排序的不重复元素集合。

您是否想深入了解数据结构和算法，同时在 C#语言中开发应用程序？如果是这样，让我们继续进行下一章，介绍树。
