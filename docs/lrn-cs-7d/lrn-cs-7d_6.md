# 第六章：第 06 天-深入探讨高级概念

今天是我们七天学习系列的第六天。在第五天，我们讨论了 C#语言的重要概念，并通过反射、集合、委托和事件进行了探讨。我们使用了代码片段来探讨非泛型集合。今天，我们将讨论使用泛型类型的集合的主要功能，然后我们将涵盖预处理指令和属性。

在本章中，我们将涵盖以下主题：

+   玩转集合和泛型

+   使用属性美化代码

+   利用预处理指令

+   开始使用 LINQ

+   编写不安全的代码

+   编写异步代码

+   重温第六天

+   实际练习

# 玩转集合和泛型

对于我们来说，集合并不新鲜，因为我们在第五天已经讨论了非泛型集合。因此，我们也有泛型集合。在本节中，我们将讨论使用代码示例的集合和泛型的所有内容。

# 理解集合类及其用法

如第五天讨论的那样，集合类是专门的类，用于数据交互（存储和检索）。我们已经讨论了各种集合类，包括

栈、队列、列表和哈希表，并且我们已经使用了`System.Collections.NonGeneric`命名空间编写了代码。以下表格为我们提供了非泛型集合类的用法和含义的概述：

| **属性** | **描述** | **用法** |
| --- | --- | --- |
| `ArrayList` | 名称本身描述了它包含一个可以使用索引访问的有序集合。我们可以这样声明`ArrayList`：`ArrayList arrayList = new ArrayList();` | 在第二天，我们讨论了数组，并学习了如何访问数组的各个元素。在`ArrayList`的情况下，我们可以获得各种方法来添加或移除集合元素的好处，就像在第五天讨论的那样。 |

| `HashTable` | `HashTable`只是键值对集合的表示，并且根据键进行组织，键实际上就是哈希码。当我们需要根据键访问数据时，建议使用`HashTable`。我们可以这样声明`HashTable`：

`Hashtable hashtable = new Hashtable();` | 当我们需要使用键访问元素时，`HashTable`非常有用。在这种情况下，我们有一个键，需要根据键在集合中找到值。 |

| `SortedList` | `SortedList`类只是键值对集合的表示，并且根据键进行组织并按键排序。`SortedList`类是`ArrayList`和`HashTable`的组合。因此，我们可以使用键或索引访问元素。我们可以这样声明`SortedList`：`SortedList sortedList = new SortedList();` | 如所述，排序列表是数组和哈希表的组合。可以使用键或索引访问项目。当使用索引访问项目时，它是`ArrayList`；另一方面，当使用哈希键访问项目时，它是`HashTable`。`SortedList`的主要特点是项目的集合始终按键值排序。 |
| --- | --- | --- |
| `Stack` | 栈表示对象的集合；对象按照**后进先出**（**LIFO**）的顺序可访问。它包含两个主要操作：push 和 pop。每当我们向列表中插入一个项目时，称为 push，当我们从列表中提取/移除一个项目时，称为 pop。当我们从列表中获取一个对象而不移除该项目时，称为 peeking。我们可以这样声明它：`Stack stackList = new Stack();` | 当需要首先检索插入的项目时，这是很重要的。 |
| `Queue` | 队列代表一个**先进先出**（FIFO）的对象集合。队列中有两个主要的操作--添加一个项目称为入队，移除一个项目称为出队。我们可以声明一个队列如下：`Queue queue = new Queue();` | 当需要首先检索插入的项目时，这一点很重要。 |
| `BitArray` | `BitArray`只是一个管理位值数组的数组。这些值被表示为布尔值。True 表示*ON*（1），False 表示*OFF*（0）。我们可以这样声明`BitArray`：`BitArray bitArray = new BitArray(8);` | 当我们需要存储位时，这个非泛型的集合类很重要。 |

前面的表只显示了非泛型的集合类。借助泛型，我们还可以通过使用`System.Collections`命名空间来实现泛型集合类。在接下来的部分，我们将讨论泛型集合类。

# 性能 - BitArray 与 boolArray

在前面的表中，我们讨论了`BitArray`只是一个管理 true 或 false 值（*0*或*1*）的数组。但在内部，`BitArray`对每个元素执行了大约 8 次的 Byte 操作，并进行了各种逻辑操作，需要更多的 CPU 周期。另一方面，`boolArray`（`bool[]`）将每个元素存储为 1 字节，因此它占用更多的内存，但需要更少的 CPU 周期。`BitArray`优于`bool[]`是内存优化器。

让我们考虑以下性能测试，并看看`BitArray`的表现如何：

```cs
private static long BitArrayTest(int max) 
{ 
    Stopwatch stopwatch = Stopwatch.StartNew(); 
    var bitarray = new BitArray(max); 
    for (int index = 0; index < bitarray.Length; index++) 
    { 
        bitarray[index] = !bitarray[index]; 
        WriteLine($"'bitarray[{index}]' = {bitarray[index]}"); 
    } 
    stopwatch.Stop(); 
    return stopwatch.ElapsedMilliseconds; 
} 
BitArray performance by applying a very simple test, where we run a for loop up to the maximum count of int MaxValue.
bool[] to make this test simpler; we just initiated a for loop up to the maximum value of int.MaxValue:
```

```cs
private static long BoolArrayTest(int max) 
{ 
    Stopwatch stopwatch = Stopwatch.StartNew(); 
    var boolArray = new bool[max]; 
    for (int index = 0; index < boolArray.Length; index++) 
    { 
        boolArray[index] = !boolArray[index]; 
        WriteLine($"'boolArray[{index}]' = {boolArray[index]}"); 
    } 
    stopwatch.Stop(); 
    return stopwatch.ElapsedMilliseconds; 
} 
BitArrayTest and BoolArrayTest methods:
```

```cs
private static void BitArrayBoolArrayPerformance() 
{ 
    //This is a simple test 
    //Not testing bitwiseshift  etc. 
    WriteLine("BitArray vs. Bool Array performance test.\n"); 
    WriteLine($"Total elements of bit array: {int.MaxValue}"); 
    PressAnyKey(); 
    WriteLine("Starting BitArray Test:"); 
    var bitArrayTestResult = BitArrayTest(int.MaxValue); 
    WriteLine("Ending BitArray Test:"); 
    WriteLine($"Total timeElapsed: {bitArrayTestResult}"); 

    WriteLine("\nStarting BoolArray Test:"); 
    WriteLine($"Total elements of bit array: {int.MaxValue}"); 
    PressAnyKey(); 
    var boolArrayTestResult = BoolArrayTest(int.MaxValue); 
    WriteLine("Ending BitArray Test:"); 
    WriteLine($"Total timeElapsed: {boolArrayTestResult}"); 
} 
```

在我的机器上，`BitArrayTest`花费了 6 秒，而`BoolArrayTest`花费了 15 秒。

从前面的测试中，我们可以得出结论，布尔数组占用了可以表示这些值的 8 倍大小/空间。简单来说，布尔数组需要每个元素 1 字节的空间。

# 理解泛型及其用法

用简单的话来说，借助泛型，我们可以创建或编写一个类的代码，该类旨在接受为其编写的不同数据类型。比如说，如果一个泛型类被编写成接受一个结构，那么它将接受 int、string 或自定义结构。这个类也被称为泛型类。当我们声明这个泛型类的实例时，它更加神奇地允许我们定义数据类型。让我们来学习下面的代码片段，我们在其中定义了一个泛型类，并在创建其实例时提供了数据类型：

```cs
    IList<Person> persons = new List<Person>()

persons variable of a generic type, List. Here, we have Person as a strong type. The following is the complete code snippet that populates this strongly typed list:
```

```cs
private static IEnumerable<Person> CreatePersonList() 
        { 
            IList<Person> persons = new List<Person> 
            { 
                new Person 
                { 
                    FirstName = "Denim", 
                    LastName = "Pinto", 
                    Age = 31 
                }, 
                new Person 
                { 
                    FirstName = "Vikas", 
                    LastName = "Tiwari", 
                    Age = 25 
                }, 
                new Person 
                { 
                    FirstName = "Shivprasad", 
                    LastName = "Koirala", 
                    Age = 40 
                }, 
                new Person 
                { 
                    FirstName = "Gaurav", 
                    LastName = "Aroraa", 
                    Age = 43 
                } 
            }; 

            return persons; 
        } 
Person type and its collection items. These items can be iterated as mentioned in the following code snippet:
```

```cs
private static void Main(string[] args) 
        { 
            WriteLine("Person list:"); 
            foreach (var person in Person.GetPersonList()) 
            { 
                WriteLine($"Name:{person.FirstName} {person.LastName}"); 
                WriteLine($"Age:{person.Age}"); 
            } 
            ReadLine(); 
        } 
```

在运行前面的代码片段后，我们将得到以下输出：

![](img/00085.jpeg)

我们可以创建一个泛型列表到一个强类型的列表，它可以接受`Person`以外的类型。为此，我们只需要创建一个这样的列表：

```cs
private IEnumerable<T> CreateGenericList<T>() 
{ 
    IList<T> persons = new List<T>(); 

    //other stuffs 

    return persons; 
} 
T could be Person or any related type.
```

# 集合和泛型

第二天，你学习了固定大小的数组。你可以使用固定大小的数组来创建强类型的列表对象。但是，如果我们想要将这些对象用或组织到其他数据结构中，比如队列、列表、栈等，该怎么办？我们可以通过使用集合（`System.Collections`）来实现所有这些。

`System.Collections` ([`www.nuget.org/packages/System.Collections/`](https://www.nuget.org/packages/System.Collections/) )是一个 NuGet 包，提供了所有泛型类型，以下是经常使用的类型：

| **泛型集合类型** | **描述** |
| --- | --- |
| `System.Collections.Generic.List<T>` | 一个强类型的泛型列表 |
| `System.Collections.Generic.Dictionary<TKey, TValue>` | 一个带有键值对的强类型泛型字典 |
| `System.Collections.Generic.Queue<T>` | 一个泛型`Queue` |
| `System.Collections.Generic.Stack<T>` | 一个泛型`Stack` |
| `System.Collections.Generic.HashSet<T>` | 一个泛型`HashSet` |
| `System.Collections.Generic.LinkedList<T>` | 一个泛型`LinkedList` |
| `System.Collections.Generic.SortedDictionary<TKey, TValue>` | 一个带有键值对集合并按键排序的泛型`SortedDictionary`。 |

上述表格只是`System.Collections.Generics`命名空间的泛型类的概述。在接下来的部分中，我们将通过代码示例详细讨论泛型集合。

有关`System.Collections.Generics`命名空间的完整类、结构和接口列表，请访问官方文档链接[`docs.microsoft.com/en-us/dotnet/api/system.collections.generic?view=netcore-2.0`](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic?view=netcore-2.0)。

# 我们为什么要使用泛型？

对于非泛型列表，我们使用来自对象类型的通用基类的集合[[`docs.microsoft.com/en-us/dotnet/api/system.object`](https://docs.microsoft.com/en-us/dotnet/api/system.object)]，这在编译时不是类型安全的。假设我们正在使用一个`ArrayList`的非泛型集合；请参考以下代码片段以了解更多详情：

```cs
ArrayList authorArrayList = new ArrayList {"Gaurav Aroraa", "43"}; 
foreach (string author in authorArrayList) 
{ 
    WriteLine($"Name:{author}"); 
} 
```

在这里，我们有一个包含字符串值的`ArrayList`。在这里，我们将年龄作为字符串，实际上应该是 int。让我们再拿一个 ArrayList，其中年龄是 int：

```cs
ArrayList editorArrayList = new ArrayList { "Vikas Tiwari", 25 }; 
foreach (int editor in editorArrayList) 
{ 
    WriteLine($"Name:{editor}"); 
} 
```

在这种情况下，我们的代码可以编译，但它会在运行时抛出类型转换异常。因此，我们的`ArrayList`没有编译时类型检查：

![](img/00086.jpeg)

通过查看上述代码，我们可以很容易地理解为什么在编译时没有错误；这是因为`ArrayList`接受任何类型（值和引用），然后将其转换为.NET 的通用基本类型，即对象。但是当我们运行代码时，它需要实际类型，例如，如果它被定义为字符串，那么在运行时它应该是字符串类型而不是对象类型。因此，我们会得到运行时异常。

在`ArrayList`中对象的转换、装箱和拆箱活动会影响性能，这取决于`ArrayList`的大小以及您正在迭代的数据有多大。

通过上述代码示例，我们知道了非泛型`ArrayList`的两个缺点：

1.  它不是编译时类型安全的。

1.  在处理大数据时会影响性能。

1.  `ArrayList`将所有内容转换为对象，因此无法在编译时阻止添加任何类型的项目。例如，在上述代码片段中，我们可以输入 int 和/或字符串类型的项目。

为了克服这些问题/缺点，我们有通用集合，它们阻止我们提供除了预期类型之外的任何内容。考虑以下代码片段：

```cs
List<string> authorName = new List<string> {"Gaurav Aroraa"}; 
```

我们有一个`List`，它被定义为只获取字符串类型的项目。因此，我们只能在这里添加字符串类型的值。现在考虑以下内容：

```cs
List<string> authorName = new List<string>(); 
authorName.Add("Gaurav Aroraa"); 
authorName.Add(43); 
```

在这里，我们试图提供一个 int 类型的项目（记住我们在`ArrayList`的情况下也做了同样的事情）。现在，我们得到了一个与转换相关的编译时错误，因此，一个定义为只接受字符串类型项目的泛型列表具有阻止客户端输入除字符串以外的任何类型项目的能力。如果我们将鼠标悬停在`43`上，它会显示完整的错误；请参考以下图片：

![](img/00087.jpeg)

在上述代码片段中，我们通过声明一个字符串列表解决了一个问题，它只允许我们输入字符串值，因此在作者的情况下，我们只能输入作者的姓名而不是作者的年龄。您可能会认为，如果我们需要另一个类型为 int 的列表，它可以让我们输入作者的年龄，那么为什么我们要使用泛型集合？目前，我们只需要两个项目--姓名和年龄--因此我们在此节点上创建了两个不同类型的列表，一个是字符串类型，一个是 int 类型。如果我们需要另一种类型的项目，那么我们会再创建一个新的列表。这是当我们有多种类型的事物时，例如字符串、int、decimal 等。我们可以创建我们自己的类型。考虑以下泛型列表的声明：

```cs
List<Person> persons = new List<Person>(); 
```

我们有一个`Person`类型的`List`。这个泛型列表将允许所有在这个类型中定义的项目。以下是我们的`Person`类：

```cs
internal class Person 
{ 
    public string FirstName { get; set; } 
    public string LastName { get; set; } 
    public int Age { get; set; } 
} 
```

我们的`Person`类包含三个属性，两个是字符串类型，一个是整数类型。在这里，我们有了解决前一节中讨论的问题的完整解决方案。借助于这个`Person`类型的`List`，我们可以输入字符串和/或整数类型的项目。以下代码片段展示了这一点：

```cs
private static void PersonList() 
{ 
    List<Person> persons = new List<Person> 
    { 
        new Person 
        { 
            FirstName = "Gaurav", 
            LastName = "Aroraa", 
            Age = 43 
        } 
    }; 
    WriteLine("Person list:"); 
    foreach (var person in persons) 
    { 
        WriteLine($"Name:{person.FirstName} {person.LastName}"); 
        WriteLine($"Age:{person.Age}"); 
    } 
} 
```

运行此代码后，我们的输出将如下所示：

![](img/00088.jpeg)

我们的`Person`类型的`List`将比`ArrayList`更高效，因为在我们的泛型类中，没有隐式类型转换为对象；项目实际上是它们期望的类型。

# 讨论约束

在前一节中，我们讨论了`Person`类型的`List`如何接受其定义类型的所有项目。在我们的示例代码中，我们只使用了字符串和整数数据类型，但在泛型中，您可以使用任何数据类型，包括整数、浮点数、双精度等。另一方面，可能存在一些情况，我们希望在泛型中将我们的使用限制在少数数据类型或特定数据类型。为了实现这一点，有泛型约束。考虑以下代码片段：

```cs
public class GenericConstraint<T> where T:class 
{ 
    public T ImplementIt(T value) 
    { 
        return value; 
    } 
} 
```

在这里，我们的类是一个泛型类。GenericConstraint，类型为`T`，实际上是一个引用类型；因此，我们创建了这个类来仅接受引用类型。这个类有一个`ImplementIt`方法，它接受一个`T`类型的参数，并返回一个`T`类型的值。

查看[`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-type-parameters`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-type-parameters)以了解有关泛型类型参数指南的更多信息。

以下声明是有效的，因为这些是引用类型：

```cs
GenericConstraint<string> genericConstraint = new GenericConstraint<string>(); 
Person person = genericPersonConstraint.ImplementIt(new Person()); 
```

以下是一个无效声明，因为这是值类型，不适用于当前的泛型类：

```cs
GenericConstraint<int> genericConstraint = new GenericConstraint<int>(); 
```

第二天，我们学到 int 是一个值类型，而不是引用类型。前面的声明会导致编译时错误。在 Visual Studio 中，您将看到以下错误：

![](img/00089.jpeg)

因此，借助泛型约束，我们限制了我们的类不接受除引用类型之外的任何类型。

**约束**基本上是一种行为，通过它您可以保护您的泛型类，防止客户端在实例化类时使用任何其他类型。如果客户端代码尝试提供不允许的类型，这将导致编译时错误。上下文关键字`where`帮助我们定义约束。

在现实世界中，您可以定义各种类型的约束，这些约束将限制客户端代码创建任何不需要的情况。让我们通过示例讨论这些类型：

# 值类型

此约束是使用上下文关键字`where T: struct`定义的。有了这个约束，客户端的代码应该包含一个值类型的参数；在这里，除了 Nullable 之外的任何值都可以指定。

**示例**

以下是声明带有值类型约束的泛型类的代码片段：

```cs
public class ValueTypeConstraint<T> where T : struct 
{ 
    public T ImplementIt(T value) 
    { 
        return value; 
    } 
} 
```

**用法**

以下是描述带有值类型约束的泛型类的客户端代码的代码片段：

```cs
private static void ImplementValueTypeGenericClass() 
{ 
    const int age = 43; 
    ValueTypeConstraint<int> valueTypeConstraint = new
    ValueTypeConstraint<int>(); 
    WriteLine($"Age:{valueTypeConstraint.ImplementIt(age)}"); 

} 
```

# 引用类型

此约束是使用上下文关键字`where T:class`定义的。使用这个约束，客户端代码被限制不能提供除引用类型之外的任何类型。有效类型包括类、接口、委托和数组。

**示例**

以下代码片段声明了一个带有引用类型约束的泛型类：

```cs
public class ReferenceTypeConstraint<T> where T:class 
{ 
    public T ImplementIt(T value) 
    { 
        return value; 
    } 
} 
```

**用法**

以下代码片段描述了带有引用类型约束的泛型类的客户端代码：

```cs
private static void ImplementReferenceTypeGenericClass() 
{ 
    const string thisIsAuthorName = "Gaurav Aroraa"; 
    ReferenceTypeConstraint<string> referenceTypeConstraint = new ReferenceTypeConstraint<string>(); 
    WriteLine($"Name:{referenceTypeConstraint.ImplementIt(thisIsAuthorName)}"); 

    ReferenceTypeConstraint<Person> referenceTypePersonConstraint = new ReferenceTypeConstraint<Person>(); 

    Person person = referenceTypePersonConstraint.ImplementIt(new Person 
    { 
        FirstName = "Gaurav", 
        LastName = "Aroraa", 
        Age = 43 
    }); 
    WriteLine($"Name:{person.FirstName}{person.LastName}"); 
    WriteLine($"Age:{person.Age}"); 
} 
```

# 默认构造函数

这个约束是用上下文关键字`where T: new()`定义的，它限制了泛型类型参数不能定义默认构造函数。还有一个必须的条件是类型`T`的参数必须有一个公共的无参数构造函数。当与其他约束一起使用时，`new()`约束必须在最后指定。

**示例**

以下代码片段声明了一个带有默认构造函数约束的通用类：

```cs
public class DefaultConstructorConstraint<T> where T : new() 
{ 
    public T ImplementIt(T value) 
    { 
        return value; 
    } 
} 
```

**用法**

以下代码片段描述了带有默认构造函数约束的通用类的客户端代码：

```cs
private static void ImplementDefaultConstructorGenericClass() 
{ 
    DefaultConstructorConstraint<ClassWithDefautConstructor>
    constructorConstraint = new
    DefaultConstructorConstraint<ClassWithDefautConstructor>(); 
    var result = constructorConstraint.ImplementIt(new
    ClassWithDefautConstructor { Name = "Gaurav Aroraa" }); 
    WriteLine($"Name:{result.Name}"); 
} 
```

# 基类约束

这个约束是用上下文关键字`where T: <BaseClass>`定义的。这个约束限制了所有客户端代码，其中提供的参数不是指定基类的或不是派生自指定基类的。

**示例**

以下代码片段声明了一个带有基类约束的通用类：

```cs
public class BaseClassConstraint<T> where T:Person 
{ 
    public T ImplementIt(T value) 
    { 
        return value; 
    } 
} 
```

**用法**

以下是一个代码片段，描述了带有基类约束的通用类的客户端代码：

```cs
private static void ImplementBaseClassConstraint() 
{ 
    BaseClassConstraint<Author>baseClassConstraint = new BaseClassConstraint<Author>(); 
    var result = baseClassConstraint.ImplementIt(new Author 
    { 
        FirstName = "Shivprasad", 
        LastName = "Koirala", 
         Age = 40 
    }); 

    WriteLine($"Name:{result.FirstName} {result.LastName}"); 
    WriteLine($"Age:{result.Age}"); 
} 
```

# 接口约束

这个约束是用上下文关键字`where T:<interface name>`定义的。客户端代码必须提供一个实现指定参数的类型的参数。在这个约束中可能定义多个接口。

**示例**

以下代码片段声明了一个带有接口约束的通用类：

```cs
public class InterfaceConstraint<T>:IDisposable where T : IDisposable 
{ 
    public T ImplementIt(T value) 
    { 
        return value; 
    } 

    public void Dispose() 
    { 
        //dispose stuff goes here 
    } 
} 
```

**用法**

以下代码片段描述了带有接口约束的通用类的客户端代码：

```cs
private static void ImplementInterfaceConstraint() 
{ 
    InterfaceConstraint<EntityClass> entityConstraint = new InterfaceConstraint<EntityClass>(); 
    var result=entityConstraint.ImplementIt(new EntityClass {Name = "Gaurav Aroraa"}); 
    WriteLine($"Name:{result.Name}"); 
} 
```

在本节中，我们讨论了泛型和集合，包括各种类型的泛型，我们还提到了为什么应该使用泛型。

有关泛型的更多详细信息，请访问官方文档[`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/)。

# 使用属性美化代码

属性提供了一种将信息与代码关联起来的方式。这些信息可以是简单的消息/警告，也可以包含复杂的操作或代码本身。这些只需用标签声明即可。这些还可以通过提供内置或自定义属性来美化我们的代码。考虑以下代码：

```cs
private void PeerOperation() 
{ 
    //other stuffs 
    WriteLine("Level1 is completed."); 
    //other stuffs 
} 
```

在这种方法中，我们显示一个信息消息来通知对等方。前面的方法将通过属性的帮助进行装饰。考虑以下代码：

```cs
[PeerInformation("Level1 is completed.")] 
private void PeerOperation() 
{ 
    //other stuffs 
} 
```

现在，我们可以看到我们只是用属性装饰了我们的方法。

根据官方文档[[`docs.microsoft.com/en-us/dotnet/csharp/tutorials/attributes`](https://docs.microsoft.com/en-us/dotnet/csharp/tutorials/attributes)]，属性提供了一种以声明方式将信息与代码关联起来的方式。它们还可以提供一个可重用的元素，可以应用于各种目标。

属性可以用于以下目的：

+   添加元数据信息

+   添加注释、描述、编译器指令等

在接下来的部分中，我们将详细讨论属性，包括代码示例。

# 属性的类型

在前面的部分中，我们讨论了属性，这些属性帮助我们装饰和美化我们的代码。在本节中，我们将详细讨论各种类型的属性。

# AttributeUsage

这是一个在框架中预定义的属性。它限制了属性的使用；换句话说，它告诉属性可以用于哪种类型的项目，也就是属性目标。这些可以是以下中的所有或一个：

+   程序集

+   类

+   构造函数

+   委托

+   枚举

+   事件

+   字段

+   GenericParameter

+   接口

+   方法

+   模块

+   参数

+   属性

+   返回值

+   结构

默认情况下，属性可以是任何类型的目标，除非你明确指定。

**示例**

以下属性被创建用于仅用于类：

```cs
[AttributeUsage(AttributeTargets.Class)] 
public class PeerInformationAttribute : Attribute 
{ 
    public PeerInformationAttribute(string information) 
    { 
        WriteLine(information); 
    } 
} 
```

在上述代码中，我们为类的唯一使用定义了属性。如果您尝试将此属性用于类以外的其他内容，则会收到编译时错误。请参阅以下图像，显示了一个为方法上的属性显示错误的图像，实际上该属性仅用于类：

![](img/00090.jpeg)

# 过时

在某些情况下，您可能希望为特定代码引发警告，以便在客户端传达。`Obsolete`属性是一个预定义属性，执行相同的操作并警告调用用户特定部分已经`过时`。

**示例**

```cs
Obsolete. You can compile and run the code even after a warning message because we have not asked this attribute to throw any error message on usage:
```

```cs
[Obsolete("Do not use this class use 'Person' instead.")] 
public class Author:Person 
{ 
    //other stuff goes here 
} 
```

以下图像显示了一个警告消息，表示不要使用`Author`类，因为它是`Obsolete`。但是客户端仍然可以编译和运行代码（我们没有要求此属性在使用时抛出错误）：

![](img/00091.jpeg)

以下将在使用时显示错误消息以及警告消息：

```cs
[Obsolete("Do not use this class use 'Person' instead.",true)] 
public class Author:Person 
{ 
    //other stuff goes here 
} 
```

考虑以下图像，用户在使用属性后出现异常，该属性被写入以在使用时抛出错误：

![](img/00092.jpeg)

# 条件

条件属性是一个预定义属性，根据应用于正在处理的代码的条件限制执行。

**示例**

考虑以下代码片段，它限制了在定义的调试预处理器下方法的条件执行（我们将在接下来的部分详细讨论预处理器）：

```cs
#define Debug 
using System.Diagnostics; 
using static System.Console; 

namespace Day06 
{ 
    internal class Program 
    { 
        private static void Main(string[] args) 
        { 
            PersonList(); 
            ReadLine(); 
        } 

        [Conditional("Debug")] 
        private static void PersonList() 
        { 
            WriteLine("Person list:"); 
            foreach (var person in Person.GetPersonList()) 
            { 
                WriteLine($"Name:{person.FirstName} {person.LastName}"); 
                WriteLine($"Age:{person.Age}"); 
            } 
        } 
    } 
} 
```

在定义预处理器符号时，请记住一件事；您要在文件的第一行上定义它。

# 创建和实现自定义属性

在上一节中，我们讨论了可用的或预定义的属性，并注意到这些属性非常有限，在实际应用中，我们的需求将需要更复杂的属性。在这种情况下，我们可以创建自己的自定义属性；这些属性类似于预定义属性，但具有我们自定义的操作代码和目标类型。所有自定义属性都应继承自`System.Attribute`类。

在本节中，我们将根据以下要求创建一个简单的自定义属性：

+   创建一个`ErrorLogger`属性

+   此属性将处理所有可用的环境，即调试、开发、生产等

+   此方法应仅限于方法

+   它应该显示自定义或提供的异常/异常消息

+   默认情况下，它应将环境视为`DEBUG`

+   如果为开发和`DEBUG`环境装饰，则应显示并抛出异常

# 先决条件

要创建和运行自定义属性，我们应该具备以下先决条件：

1.  Visual Studio 2017 或更高版本

1.  .NET Core 1.1 或更高版本

以下是创建我们期望的属性的代码片段：

```cs
public class ErrorLogger : Attribute 
{ 
    public ErrorLogger(string exception) 
    { 
        switch (Env) 
        { 
            case Env.Debug: 
            case Env.Dev: 
                WriteLine($"{exception}"); 
                throw new Exception(exception); 
            case Env.Prod: 
                WriteLine($"{exception}"); 
                break; 
            default: 
                WriteLine($"{exception}"); 
                throw new Exception(exception); 
        } 
    } 

    public Env Env { get; set; } 
} 
```

在上述代码中，我们只是向控制台写入客户端代码提供的任何异常。在`DEBUG`或`Dev`环境的情况下，进一步抛出异常。

以下代码片段显示了此属性的简单用法：

```cs
public class MathClass 
{ 
    [ErrorLogger("Add Math opetaion in development", Env =
    Env.Debug)] 
    public string Add(int num1, int num2) 
    { 
        return $"Sum of {num1} and {num2} = {num1 + num2}"; 
    } 

    [ErrorLogger("Substract Math opetaion in development", Env =
    Env.Dev)] 
    public string Substract(int num1, int num2) 
    { 
        return $"Substracrion of {num1} and {num2} = {num1 -
        num2}"; 
    } 

    [ErrorLogger("Multiply Math opetaion in development", Env =
    Env.Prod)] 
    public string Multiply(int num1, int num2) 
    { 
        return $"Multiplication of {num1} and {num2} = {num1 -
        num2}"; 
    } 
} 
```

在上述代码中，我们有不同的方法，标记为不同的环境。我们的属性将触发并编写为各个方法提供的异常。

# 利用预处理器指令

从名称上可以清楚地看出，预处理器指令是在实际编译开始之前进行的处理过程。换句话说，这些预处理器向编译器发出指令，对信息进行预处理，这是在编译器编译代码之前进行的。

# 重要点

在您使用预处理器时，请注意以下几点：

+   预处理器指令实际上是编译器的条件

+   预处理器指令必须以`#`符号开头

+   预处理器指令不应以分号（`;`）结尾，就像语句结束一样

+   预处理器不用于创建宏

+   预处理器应逐行声明

# 预处理器指令的作用

考虑以下预处理器指令：

```cs
#if ... #endif  
```

这个指令是一个条件指令，当这个指令应用到代码时，代码会执行，你也可以使用`#elseif`和/或`#else`指令。由于这是一个条件指令，C#中的`#if`条件是布尔值，这些运算符可以用来检查相等（`==`）和不相等（`!=`），以及多个符号之间的关系，以及（`&&`），或（`||`），和非（`!`）运算符也可以用来评估条件。

你应该在文件的第一行上定义一个符号，使用`#define`指令。

考虑以下代码片段，它让我们了解条件编译：

```cs
#define DEBUG 
#define DEV 
using static System.Console; 

namespace Day06 
{ 
    public class PreprocessorDirective 
    { 
        public void ConditionalProcessor() =>
        #if (DEBUG && !DEV) 
            WriteLine("Symbol is DEBUG."); 
            #elseif (!DEBUG && DEV) 
            WriteLine("Symbol is DEV"); 
            #else 
            WriteLine("Symbols are DEBUG & DEV"); 
            #endif 
    } 
} 
DEBUG and DEV, and now, on the basis of our condition the following will be the output of the preceding code.
```

![](img/00093.jpeg)

**#define 和#undef**

`#define`指令基本上为我们定义了一个将在条件预处理器指令中使用的符号。

`#define`不能用于声明常量值。

在使用`#define`声明符号时应该记住以下几点：

+   它不能用于声明常量

+   它可以定义一个符号，但不能为这些符号赋值

+   对符号的任何指令都应该在文件中定义符号之后，这意味着`#define`指令总是在使用之前出现

+   使用`#define`指令定义或创建的符号的作用域在它被声明/定义的文件中

回想一下我们在`#if`指令中讨论的代码示例，我们在那里定义了两个符号。所以，定义一个符号很容易，比如：`#define DEBUG`。

`#undef`指令让我们取消之前定义的符号。这个预处理器应该出现在任何非指令语句之前。考虑以下代码：

```cs
#define DEBUG 
#define DEV 
#undef DEBUG 
using static System.Console; 

namespace Day06 
{ 
    public class PreprocessorDirective 
    { 
        public void ConditionalProcessor() => 
#if (DEBUG && !DEV) 
            WriteLine("Symbol is DEBUG."); 
#elif (!DEBUG && DEV) 
            WriteLine("Symbol is DEV"); 
#else 
            WriteLine("Symbols are DEBUG & DEV"); 
#endif 
    } 
} 
```

在上面的代码中，我们取消了`DEBUG`符号，代码将产生以下输出：

![](img/00094.jpeg)

**#region 和#endregion 指令**

在处理长代码文件时，这些指令非常有用。有时候，当我们在处理一个长代码库时，比如一个企业应用，这种应用会有 1000 行代码，并且这些行会是不同函数/方法或业务逻辑的一部分。因此，为了更好地可读性，我们可以在区域内管理这些部分。在一个区域中，我们可以为区域包含的代码命名并给出简短的描述。让我们看一下以下图像：

![](img/00095.jpeg)

在上面的图像中，左侧部分显示了`#region`...`#endregion`指令的扩展视图，告诉我们如何将这些指令应用到我们的长代码文件中。图像的右侧显示了折叠视图，当你将鼠标悬停在折叠区域文本上时，你会看到在 Visual Studio 中出现了一个矩形块，它显示了这些区域包含的内容。因此，你无需展开区域来检查这个区域下写了什么代码。

**#line 指令**

`#line`指令提供了一种修改编译器实际行号的方式。你还可以为错误和警告提供输出`FileName`，这是可选的。这个指令在构建过程中的自动化中可能会有用。在原始源代码中删除了行号的情况下，你需要基于原始文件生成输出。

另外，`#line`默认指令将行号返回到默认值，并且它会计算之前重新编号的行。

`#line`隐藏指令不会影响错误报告中的文件名或行号。

`#line`文件名指令定义了一个在编译器输出中想要出现的文件名的方式。在这里，默认值是实际使用的文件名；你可以在双引号中提供一个新的名字，并且这个名字必须在行号之前。

考虑以下代码片段：

```cs
        public void LinePreprocessor() 
        { 
            #line 85 "LineprocessorIsTheFileName" 
            WriteLine("This statement is at line#85 and not at
            line# 25");
            #line default 
            WriteLine("This statement is at line#29 and not at
            line# 28");
            #line hidden 
            WriteLine("This statement is at line#30"); 
        } 
    } 
85 for the first statement, which was originally at line number 25.
```

**#warning 指令**

`#warning`指令提供了一种在代码的任何部分生成警告的方式，并通常在条件指令内工作。考虑以下代码片段：

```cs
        public void WarningPreProcessor() 
        { 
           #if DEBUG 
           #warning "This is a DEBUG compilation." 
           WriteLine("Environment is DEBUG."); 
           #endif 
        } 
    } 
```

上述代码将在编译时发出警告，并且警告消息将是您使用`#warning`指令提供的内容：

![](img/00096.jpeg)

**#error**

`#error`指令提供了一种在代码的任何部分生成错误的方式。考虑以下代码片段：

```cs
        public void ErrorPreProcessor() 
        { 
           #if DEV 
           #error "This is a DEV compilation." 
           WriteLine("Environment is DEV."); 
           #endif 
        } 
```

这将引发错误，由于这个错误，您的代码将无法正确构建；它将以您使用`#error`指令提供的错误消息失败构建。让我们看一下以下图片：

![](img/00097.jpeg)

在本节中，我们讨论了预处理指令及其在代码示例中的使用。

有关 C#预处理指令的完整参考，请参考官方文档：

[`docs.microsoft.com/en-us/dotnet/csharp/language-reference/preprocessor-directives/`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/preprocessor-directives/)

# 开始使用 LINQ

LINQ 只是语言集成查询的缩写，是编程语言的一部分。LINQ 提供了一种使用指定语法编写或查询数据的简单方法，就像我们在尝试为某些特定条件查询数据时使用 where 子句一样。因此，我们可以说 LINQ 是一种用于查询数据的语法。

在本节中，我们将看到一个简单的示例来查询数据。我们有`Person`列表，以下代码片段为我们提供了各种查询数据的方式：

```cs
private static void TestLINQ() 
{ 
    var person = from p in Person.GetPersonList() 
        where p.Id == 1 
        select p; 
    foreach (var per in person) 
    { 
        WriteLine($"Person Id:{per.Id}"); 
        WriteLine($"Name:{per.FirstName} {per.LastName}"); 
        WriteLine($"Age:{per.Age}"); 
    } 

} 
List of persons for *personId* =1\. The LINQ query returns a result of IEnumerable<Person> type which can be easily accessed using foreach. This code produces the following output:
```

![](img/00098.jpeg)

LINQ 的完整讨论超出了本书的范围。有关完整的 LINQ 功能，请参考：[`code.msdn.microsoft.com/101-LINQ-Samples-3fb9811b`](https://code.msdn.microsoft.com/101-LINQ-Samples-3fb9811b)

# 编写不安全代码

在本节中，我们将讨论如何使用 Visual Studio 编写不安全代码的介绍。语言 C#提供了一种编写代码的方式，该代码编译并创建对象，这些对象在根下由垃圾收集器管理有关垃圾收集器的更多详细信息，请参考[第 01 天]。简而言之，C#不像使用函数指针访问引用的 C、C++语言。但是在某些情况下，有必要在 C#语言中使用函数指针，类似于支持函数指针的语言如 C 或 C++，但 C#语言不支持它。为了克服这种情况，我们在 C#语言中有不安全代码。有一个修饰符不安全，告诉编译器这段代码不受垃圾收集器控制，在该块内我们可以使用函数指针和其他不安全的东西。要使用不安全代码，我们首先要求编译器从 Visual Studio 2017 或更高版本开始设置不安全编译，只需转到项目属性，在“生成”选项卡上，选择“允许不安全代码”选项，参考以下截图：

![](img/00099.jpeg)

如果未选择此选项，您将无法继续使用不安全代码，请参考以下截图：

![](img/00100.jpeg)

设置不安全编译后，让我们编写代码使用指针交换两个数字，考虑以下代码片段：

```cs
public unsafe void SwapNumbers(int*  num1, int* num2) 
{ 
    int tempNum = *num1; 
    *num1 = *num2; 
    *num2 = tempNum; 
} 
```

上面是一个非常简单的交换函数，它只是使用指针交换两个数字。让我们调用这个函数来看看实际结果：

```cs
private static unsafe void TestUnsafeSwap() 
{ 
    Write("Enter first number:"); 
    var num1 = Convert.ToInt32(ReadLine()); 
    Write("Enter second number:"); 
    var num2 = Convert.ToInt32(ReadLine()); 
    WriteLine("Before calling swap function:"); 
    WriteLine($"Number1:{num1}, Number2:{num2}"); 
    //call swap 
    new UnsafeSwap().SwapNumbers(&num1, &num2); 
    WriteLine("After calling swap function:"); 
    WriteLine($"Number1:{num1}, Number2:{num2}"); 
} 
```

在上面的代码片段中，我们输入了两个数字，然后显示交换前后的结果，这产生了以下输出：

![](img/00101.jpeg)

在本节中，我们讨论了如何处理不安全代码。

有关不安全代码的更多详细信息，请参考语言规范的官方文档：[`docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/unsafe-code`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/unsafe-code)

# 编写异步代码

在我们讨论异步方式的代码之前，让我们先讨论一下我们的普通代码，即同步代码，让我们考虑以下代码片段：

```cs
public class FilePolling 
{ 
    public void PoleAFile(string fileName) 
    { 
        Console.Write($"This is polling file:
        {fileName}"); 
        //file polling stuff goes here 
    } 
} 
```

前面的代码片段简短而简洁。它告诉我们它正在轮询一个特定的文件。在这里，系统必须等待完成轮询文件的操作，然后才能开始下一个操作。这就是同步代码。现在，考虑一种情况，我们不需要等待完成这个函数的操作就开始另一个操作或函数。为了满足这样的情况，我们有异步编码，这是可能的关键字是 async。

考虑以下代码：

```cs
public async void PoleAFileAsync(string fileName) 
{ 
    Console.Write($"This is polling file: {fileName}"); 
    //file polling async stuff goes here 
} 
```

仅仅通过`async`关键字，我们的代码就能够进行异步调用。

从先前的代码来看，我们可以说异步编程是一种不让客户端代码等待执行另一个函数或操作的任何异步操作的编程。简单地说，我们可以说异步代码不能阻止需要调用的另一个操作。

在本章中，我们讨论了异步编码。关于这个主题的完整讨论超出了我们书的范围。有关完整详情，请参阅官方文档：[`docs.microsoft.com/en-us/dotnet/csharp/async`](https://docs.microsoft.com/en-us/dotnet/csharp/async)

# 动手练习

1.  通过创建`StringCalculator`的泛型代码来定义泛型类：[`github.com/garora/TDD-Katas/tree/develop/Src/cs/StringCalculator`](https://github.com/garora/TDD-Katas/tree/develop/Src/cs/StringCalculator)

1.  创建一个泛型和非泛型集合，并测试哪一个在性能上更好。

1.  我们在“为什么应该使用泛型？”一节中讨论了代码片段，其中讲述了运行时编译异常。在这方面，为什么我们不应该以以下方式使用相同的代码？

```cs
internal class Program 
{ 
      private static void Main(string[] args) 
{ 
    //No exception at compile-time or run-time 
    ArrayList authorEditorArrayList = new ArrayList {
    "Gaurav Arora", 43, "Vikas Tiwari", 25 }; 
    foreach (var authorEditor in authorEditorArrayList) 
    { 
        WriteLine($"{authorEditor}"); 
    } 
}     
} 
```

1.  在泛型代码中，`default`关键字的用途是什么，通过一个现实世界的例子加以阐述。

1.  使用所有 3 种预定义属性编写简单代码。

1.  属性的默认限制类型是什么？编写一个程序来展示所有限制类型。

1.  创建一个名为*LogFailuresAttribute*的自定义属性，用于记录所有异常到文本文件中。

1.  为什么预处理器指令`#define`不能用于声明常量值？

1.  编写一个程序来创建一个`作者`的`列表`，并在其上应用 LINQ 功能。

1.  编写一个程序来对数组进行排序

1.  编写一个完整的程序来编写同步和异步方法来写一个文件。

# 重温第 6 天

今天，我们讨论了泛型、属性、预处理器、LINQ、不安全代码和异步编程等高级概念。

我们的一天从泛型开始，您通过代码片段了解了泛型类。然后，我们深入了解了属性，并学习了如何使用预定义属性装饰我们的 C#代码。我们创建了一个自定义属性，并在我们的代码示例中使用了它。我们讨论了预处理器指令，并学习了这些指令在我们编码中的用法。其他讨论的概念包括 LINQ、不安全代码和异步编程。

明天，也就是第七天将是我们七天学习系列的结束日。我们将介绍 OOP 概念及其在 C#语言中的实现。
