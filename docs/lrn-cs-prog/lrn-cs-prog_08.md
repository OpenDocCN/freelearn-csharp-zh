# 第八章：高级主题

在前几章中，我们学习了语言语法、数据类型、类和结构的使用、泛型、集合等主题，这些知识使你能够编写至少简单的 C#程序。然而，语言还有更多内容，本章中我们将探讨更高级的概念。这将包括委托，它对于我们后面在本书中涵盖的函数式和异步编程至关重要，以及各种形式的模式匹配，包括用于文本的正则表达式。

我们将讨论的主题如下：

+   委托和事件

+   匿名类型

+   元组

+   模式匹配

+   正则表达式

+   扩展方法

完成本章后，你将了解如何使用委托来响应应用程序中发生的事件，如何使用元组处理多个值而不引入新类型，如何在代码中执行模式匹配，以及如何使用正则表达式搜索和替换文本。最后但同样重要的是，你将学会如何使用扩展方法在不修改其实际源代码的情况下扩展类型。

让我们通过学习委托和事件来开始本章。

# 委托和事件

**回调**是一个函数（或更一般地说，任何可执行代码），它作为参数传递给另一个函数，以便立即调用（**同步回调**）或在以后的某个时间调用（**异步回调**）。操作系统（如 Windows）广泛使用回调来允许应用程序响应鼠标事件或按键事件等事件。回调的另一个典型例子是通用算法，它使用回调来处理来自集合的元素，例如比较它们以对其进行排序或筛选。

在诸如 C 和 C++之类的语言中，回调只是一个*函数指针*（即函数的地址）。然而，在.NET 中，回调是*强类型对象*，它不仅保存了一个或多个方法的引用，还保存了关于它们的参数和返回类型的信息。在.NET 和 C#中，回调由委托表示。

## 委托

`delegate`关键字。声明看起来像一个函数签名，但编译器实际上引入了一个类，该类可以保存与委托签名匹配的方法的引用。委托可以保存对*静态*或*实例方法*的引用。

为了更好地理解委托的定义和使用方式，我们将考虑以下例子。

我们有一个表示引擎的类。引擎可以做不同的事情，但我们将专注于启动和停止这个引擎。当这些事件发生时，我们希望让使用引擎的客户端知道这一点，并给他们机会做一些事情。简单起见，客户端只会将事件记录到控制台。在这个简单的模型中，引擎可以处于这两种状态中的任何一种：`StatusChange`：

```cs
public enum Status { Started, Stopped }
public delegate void StatusChange(Status status);
```

`StatusChange`不是一个函数，而是一个*类型*。我们将用它来声明引擎中保存回调方法引用的变量。表示引擎的类如下：

```cs
public class Engine
{
    private StatusChange statusChangeHandler;
    public void RegisterStatusChangeHandler(StatusChange handler)
    {
        statusChangeHandler = handler;
    }
    public void Start()
    {
        // start the engine
        if (statusChangeHandler != null)
            statusChangeHandler(Status.Started);
    }
    public void Stop()
    {
        // stop the engine
        if (statusChangeHandler != null)
            statusChangeHandler(Status.Stopped);
    }
}
```

这里有几件事情需要注意：

+   首先，`RegisterStatusChangeHandler()` 方法接受委托类型（`StatusChange`）的参数，并将其分配给`statusChangeHandler`成员字段。

+   其次，`Start()`和`Stop()`方法实际上并没有做太多事情（仅为简单起见），但你可以想象它们正在启动和停止引擎。然而，在此之后，它们调用回调函数，就像普通函数一样，传递所有必要的参数。

+   在这个例子中，委托不返回任何值，但委托可以返回任何东西。然而，在调用回调方法之前，会执行*空引用检查*。如果委托没有被分配到一个方法的引用，调用委托会导致`NullReferenceException`。

客户端代码创建了`Engine`类的一个实例，注册了状态更改的处理程序，然后启动和停止它。代码如下：

```cs
class Program
{
    static void Main(string[] args)
    {
        Engine engine = new Engine();
        engine.RegisterStatusChangeHandler
          (OnEngineStatusChanged); 
        engine.Start();
        engine.Stop();
    }
    private static void OnEngineStatusChanged(Status status)
    {
        Console.WriteLine($"Engine is now {status}");
    }
}
```

静态方法`OnEngineStatusChanged()`用作引擎启动和停止事件的回调。其签名与委托的类型匹配。执行此程序将产生以下输出：

```cs
Engine is now Started
Engine is now Stopped
```

.NET 委托的一个重要方面是它们支持*多播*。这意味着您实际上可以设置对要调用的任意多个方法的引用；然后委托将按照它们被添加的顺序调用它们。多播委托由`System.MulticastDelegate`类表示。该类在内部具有称为*调用列表*的委托链表。此列表可以有任意数量的元素。当调用多播委托时，调用列表中的所有委托按照它们在列表中出现的顺序（即它们被添加的顺序）被调用。此操作是同步的，如果在调用列表的执行过程中出现任何错误，将抛出异常。

另一方面，当您不再希望调用某个方法时，可以从委托中移除对该方法的引用。这两个方面将在以下示例中得到说明，其中我们改变了`Engine`类以允许多个回调不仅被注册，而且还可以被注销：

```cs
public class Engine
{
    private StatusChange statusChangeHandler;
    public void RegisterStatusChangeHandler(StatusChange handler)
    {
        statusChangeHandler += handler;
    }
    public void UnregisterStatusChangeHandler(StatusChange handler)
    {
        statusChangeHandler -= handler;
    }
    public void Start()
    {
        statusChangeHandler?.Invoke(Status.Started);
    }
    public void Stop()
    {
        statusChangeHandler?.Invoke(Status.Stopped);
    }
}
```

再次，这里有两件事需要注意：

+   首先，`RegisterStatusChangeHandler()`方法不再简单地将其参数分配给`statusChangeHandler`字段，而是实际上使用`+=`运算符向委托内部持有的列表添加一个新引用。因此，`UnregisterStatusChangeHandler()`方法使用`-=`运算符从委托中移除一个引用。`+=`和`-=`运算符已被委托类型重载。

+   其次，`Start()`和`Stop()`中的代码略有改变。使用空值条件运算符（`?.`）仅在对象不为`null`时调用`Invoke()`方法。

另一方面，主程序中的更改如下：

```cs
class Program
{
    static void Main(string[] args)
    {
        Engine engine = new Engine();
        engine.RegisterStatusChangeHandler
          (OnEngineStatusChanged); 
        engine.RegisterStatusChangeHandler
          (OnEngineStatusChanged2); 
        engine.Start();
        engine.Stop();
        engine.UnregisterStatusChangeHandler
          (OnEngineStatusChanged2);
        engine.Start();
    }
    private static void OnEngineStatusChanged(Status status)
    {
        Console.WriteLine($"Engine is now {status}");
    }
    private static void OnEngineStatusChanged2(Status status)
    {
        File.AppendAllText(@"c:\temp\engine.log",
                           $"Engine is now {status}\n");
    }
}
```

这次，我们注册了两个回调：

+   一个在*控制台*上记录事件。

+   一个记录到*文件*的回调。

我们启动和停止引擎，然后注销记录到磁盘文件的回调函数。最后，我们再次启动引擎。因此，控制台上的输出将如下所示：

```cs
Engine is now Started
Engine is now Stopped
Engine is now Started
```

然而，只有前两行也出现在磁盘文件上，因为在重新启动引擎之前已经移除了第二个回调函数。

在这个第二个示例中，我们使用`Invoke()`方法调用委托引用的方法。`Invoke()`方法是从哪里来的呢？在幕后，当您声明委托类型时，编译器会生成一个从`System.MulticastDelegate`派生的密封类，该类又从`System.Delegate`派生。这些都是您不允许显式派生的系统类型。但是，它们提供了我们迄今为止看到的所有功能，例如能够向委托的调用列表中添加和移除方法的能力。

编译器创建的类包含三种方法——`Invoke()`（用于以*同步方式*调用回调函数）、`BeginInvoke()`和`EndInvoke()`（用于以*异步方式*调用回调函数）。有关异步委托的示例，请参考其他参考资料。您实际上可以通过在反汇编器（如**ildasm.exe**或**ILSpy**）中打开程序集来检查编译器生成的代码。

## 事件

到目前为止，我们编写的代码有点太*显式*了。我们不得不创建方法来注册和取消注册对回调方法的引用。这是因为在类中，持有这些引用的委托是私有的。我们可以将其设为公共的，但这样会破坏封装性，并有风险允许客户端错误地覆盖委托的调用列表。为了帮助处理这些方面，.NET 和 C#提供了*事件*，它们只是我们之前为注册和取消注册回调编写的显式代码的语法糖。事件是用`event`关键字引入的。

引擎的最后一个实现将更改为以下内容：

```cs
public class Engine
{
    public event StatusChange StatusChanged;
    public void Start()
    {
        StatusChanged?.Invoke(Status.Started);
    }
    public void Stop()
    {
        StatusChanged?.Invoke(Status.Stopped);
    }
}
```

请注意，我们不再有用于注册和取消注册回调的方法，只有一个名为`StatusChanged`的事件对象。这些是在客户端代码中在事件对象上完成的，使用`+=`（添加对方法的引用）和`-=`（删除对方法的引用）操作符。我们可以在以下代码中看到客户端代码。

在这个例子中，我们创建了一个`Engine`对象，并为`StatusChanged`事件注册了回调函数——一个是对`OnEngineStatusChanged()`方法的引用（将事件记录到文件中），另一个是一个 lambda 表达式（将事件记录到控制台）：

```cs
class Program
{
    static void Main(string[] args)
    {
        Engine engine = new Engine();
        engine.StatusChanged += OnEngineStatusChanged;
        engine.StatusChanged += 
            status => Console.WriteLine(
                        $"Engine is now {status}");
        engine.Start();
        engine.Stop();
        engine.StatusChanged -= OnEngineStatusChanged;
        engine.Start();
    }
    private static void OnEngineStatusChanged(Status status)
    {
        File.AppendAllText(@"c:\temp\engine.log",
                           $"Engine is now {status}\n");
    }
}
```

启动和停止引擎后，我们取消对`OnEngineStatusChanged()`的引用，然后重新启动引擎。执行此程序的结果与先前的程序相同。

到目前为止，所有的例子中，委托类型都有一个参数，即引擎的状态。然而，事件模式的正确实现（在整个.NET Framework 中都使用）是有两个参数：

+   第一个参数是`System.Object`，它保存了生成事件的对象的引用。由调用的客户端决定是否使用此引用。

+   第二个参数是从`System.EventArgs`派生的类型，其中包含与事件相关的所有信息。

为了符合这种模式，我们的`Engine`的实现将更改为以下内容：

```cs
public class EngineEventArgs : EventArgs
{
    public Status Status { get; private set; }
    public EngineEventArgs(Status s)
    {
        Status = s;
    }
}
public delegate void StatusChange(
         object sender, EngineEventArgs args);
public class Engine
{
    public event StatusChange StatusChanged;
    public void Start()
    {
        StatusChanged?.Invoke(this, 
           new EngineEventArgs(Status. Started));
    }
    public void Stop()
    {
        StatusChanged?.Invoke(this, 
           new EngineEventArgs(Status.Stopped));
    }
}
```

我们将留给读者练习对主程序进行必要的更改，以使用`Engine`类的新实现。

有关委托和事件的关键要点如下：

+   委托允许将方法作为参数传递，以便稍后调用，可以同步或异步调用。

+   委托支持多播，即调用多个回调方法。

+   静态方法、实例方法、匿名方法和 lambda 表达式都可以作为委托的回调使用。

+   委托可以是泛型的。

+   事件是一种语法糖，有助于注册和移除回调。

本章讨论的下一个主题是匿名类型。

# 匿名类型

有时需要构造临时对象来保存一些值，通常是某个较大对象的子集。为了避免仅为此目的创建特定类型，语言提供了所谓的*匿名类型*。这些是一种使用后即忘记的类型，通常与**语言集成查询**（**LINQ**）一起在查询表达式中使用。这个主题将在*第十章*中讨论，*Lambda、LINQ 和函数式编程*。

这些类型被称为匿名，因为在源代码中没有指定名称。名称由编译器分配。它们只包含只读属性；不允许任何其他成员类型。只读属性的类型不能显式指定，而是由编译器推断。

使用`new`关键字引入匿名类型，后面跟着一系列属性（对象初始化器）的尖括号。以下代码片段显示了一个例子：

```cs
var o = new { Name = "M270 Turbo", Capacity = 1600, 
Power = 75.0 };
Console.WriteLine($"{o.Name} {o.Capacity / 1000.0}l 
{o.Power}kW");
```

在这里，我们定义了一个具有三个属性`Name`、`Capacity`和`Power`的匿名类型。这些属性的类型由编译器从它们的初始化值中推断出来。在这种情况下，它们分别是`Name`的`string`，`Capacity`的`int`和`Power`的`double`。

当从表达式初始化属性时，必须指定属性的名称。但是，如果它是从另一个对象的字段或属性初始化的，名称是可选的。在这种情况下，编译器使用与用于初始化它的成员相同的名称。举个例子，让我们考虑以下类型：

```cs
class Engine
{
    public string Name { get; }
    public int Capacity { get; }
    public double Power { get; }

    public Engine(string name, int capacity, double power)
    {
        Name = name;
        Capacity = capacity;
        Power = power;
    }
}
```

有了这个，我们可以写如下：

```cs
var e = new Engine("M270 Turbo", 1600, 75.0);
var o = new { e.Name, e.Power };
Console.WriteLine($"{o.Name} {o.Power}kW");
```

我们已经创建了`Engine`类的一个实例。从这个实例中，我们创建了另一个匿名类型的对象，它有两个属性，编译器称之为`Name`和`Power`，因为它们是从`Engine`类的`Name`和`Power`属性初始化的。

匿名类型具有以下属性：

+   它们被实现为密封类，因此是引用类型。CLI 不会区分匿名类型和其他引用类型。

+   它们直接派生自`System.Object`，只能转换为`System.Object`。

+   它们只能包含只读属性。不允许其他成员。

+   它们不能用作字段、属性、事件、方法的返回类型或方法、构造函数或索引器的参数类型。

+   您可以为匿名类型的只读属性指定名称。这在从表达式初始化时是强制性的，但在从字段或属性初始化时是可选的。在这种情况下，编译器使用成员的名称作为属性的名称。

+   用于初始化属性的表达式不能为 null、匿名函数或指针类型。

+   匿名类型的作用域是定义它的方法。

+   当声明匿名类型的变量时，必须使用`var`作为类型名称的占位符。

元组提供了一种类似的临时类型概念，但具有不同的语义，这是下一节的主题。

# 元组

`out`或`ref`参数，或者当您想要将多个值作为单个对象传递给方法时。

这个方面代表了匿名类型和元组之间的关键区别。前者用于在单个方法的范围内使用，不能作为参数传递或从方法返回。后者则是为了这个确切的目的而设计的。

在 C#中，有两种类型的元组：

+   `System.Tuple`类

+   `System.ValueTuple`结构

在下一小节中，我们将看看这两种类型。

## 元组类

引用元组是在.NET Framework 4.0 中引入的。泛型类`System.Tuple`可以容纳最多八个不同类型的值。如果需要超过八个值的元组，您将不得不创建嵌套元组。元组可以通过以下两种方式实例化：

+   通过使用`Tuple<T>`的*构造函数*

+   通过使用*辅助方法*，`Tuple.Create()`

以下两行是等价的：

```cs
var engine = new Tuple<string, int, double>("M270 Turbo", 1600, 75);
var engine = Tuple.Create("M270 Turbo", 1600, 75);
```

这里的第二行更好，因为它更简单，你不必指定每个值的类型。这是因为编译器从参数中推断出类型。

元组的元素可以通过名为`Item1`、`Item2`、`Item3`、`Item4`、`Item5`、`Item6`、`Item7`和`Rest`的属性访问。在下面的示例中，我们使用`Item1`、`Item2`和`Item3`属性将引擎名称、容量和功率打印到控制台上：

```cs
Console.WriteLine(
    $"{engine.Item1} {engine.Item2/1000.0}l {engine.Item3}kW");
```

当需要超过八个元素时，可以使用嵌套元组。在这种情况下，将嵌套元组放在最后一个元素是有意义的。以下示例创建了一个具有 10 个值的元组，其中最后三个值（表示不同功率的发动机功率，单位为千瓦）被分组在第二个嵌套元组中：

```cs
var engine = Tuple.Create(
    "M270 DE16 LA R", 1595, 83, 73.7, 180, "gasoline", 2015, 
    Tuple.Create(75, 90, 115));
Console.WriteLine($"{engine.Item1} powers: {engine.Rest.Item1}");
```

请注意这里我们使用的是`Rest.Item1`而不是简单的`Rest`。该程序的输出如下：

```cs
M270 DE16 LA R powers: (75, 90, 115)
```

这是因为变量 engine 的推断类型是 `Tuple<string, int, int, double, int, string, int, Tuple<Tuple<int, int, int>>>`。因此，`Rest` 表示一个包含单个值的元组，该值也是包含三个 `int` 值的元组。要访问嵌套元组的元素，您必须使用，对于这种情况，`Rest.Item1.Item1`、`Rest.Item1.Item2` 和 `Rest.Item1.Item3`。

要创建类型为 `Tuple<string, int, int, double, int, string, int, Tuple<int, int, int>>` 的元组，必须使用构造函数的显式语法：

```cs
var engine = new Tuple<string, int, int, double, int, string, int, Tuple<int, int, int>>
    ("M270 DE16 LA R", 1595, 83, 73.7, 180, "gasoline", 2015,
    new Tuple<int, int, int>(75, 90, 115));
Console.WriteLine($"{engine.Item1} powers: {engine.Rest}");
```

`System.Tuple` 是一个引用类型，因此此类型的对象分配在堆上。如果在程序执行过程中发生许多小对象的分配，可能会影响性能。

这增加了我们之前看到的限制——元素数量和未命名属性。为了克服这些问题，C# 7.0、.NET Framework 4.7 和 .NET Standard 2.0 引入了值类型元组，我们将在下一节中探讨。

## 值元组

这些由 `System.ValueTuple` 结构表示。如果您的项目不针对 .NET Framework 4.7 或更高版本，或 .NET Standard 2.0 或更高版本，您仍然可以通过将其安装为 NuGet 包来使用 `ValueTuple`。

在几个 7.x 版本的语言中添加了各种值元组功能。这里描述的功能与 C# 8 对齐。

除了值语义之外，值元组在几个重要方面与引用元组不同：

+   它们可以容纳任意数量的元素序列，但至少需要两个。

+   它们可能具有编译时命名字段。

+   它们具有更简单但更丰富的语法，用于创建、赋值、解构和比较值。

使用*括号语法*和指定的值来创建值元组。以下三个声明是等价的：

```cs
ValueTuple<string, int, double> engine = ("M270 Turbo", 1600, 75.0);
(string, int, double) engine = ("M270 Turbo", 1600, 75.0);
var engine = ("M270 Turbo", 1600, 75.0);
```

在所有这些情况下，变量 engine 的类型是 `ValueTuple<string, int, double>`，元组被称为*未命名*。在这种情况下，它的值可以在公共字段中访问——`Item1`、`Item2` 和 `Item3`，这些是编译器隐式分配的名称：

```cs
Console.WriteLine(
    $"{engine.Item1} {engine.Item2/1000.0}l {engine.Item3}kW");
```

但是，在创建值元组时，您可以选择为值指定名称，从而为字段创建同义词，如 `Item1`、`Item2` 等。这种值元组称为**命名元组**。您可以在以下代码片段中看到一个命名元组的示例：

```cs
var engine = (Name: "M270 Turbo", Capacity: 1600, Power: 75.0);
Console.WriteLine(
    $"{engine.name} {engine.capacity / 1000.0}l {engine.power}kW");
```

这些同义词仅在编译时可用，因为 IDE 利用 Roslyn API 从源代码中为您提供它们，但在编译器中间语言代码中，它们不可用，只有未命名字段——`Item1`、`Item2` 等。

字段的名称可以出现在赋值的任一侧；此外，它们可以同时出现在两侧，在这种情况下，*左侧名称* 将*优先*，*右侧名称* 将*被忽略*。以下两个声明将产生一个与前面代码中看到的命名值元组相同的命名值元组：

```cs
(string Name, int Capacity, double Power) engine = 
    ("M270 Turbo", 1600, 75.0);
(string Name, int Capacity, double Power) engine = 
    (name: "M270 Turbo", cap: 1600, pow: 75.0);
```

字段的名称也可以从用于初始化值元组的变量中推断出（如 C# 7.1）。在以下示例中，值元组将具有名为 `name`、`capacity`（小写）和 `Item3` 的字段，因为最后一个值是一个没有明确指定名称的文字：

```cs
var name = "M270 Turbo";
var capacity = 1600;
var engine = (name, capacity, 75);
Console.WriteLine(
    $"{engine.name} {engine.capacity / 1000.0}l {engine.Item3}kW");
```

从方法返回值元组非常简单。在以下示例中，`GetEngine()` 函数返回一个未命名的值类型：

```cs
(string, int, double) GetEngine()
{
    return ("M270 Turbo", 1600, 75.0);
}
```

但是，您可以选择返回一个命名值类型，在这种情况下，您需要指定字段的名称，如下所示：

```cs
(string Name, int Capacity, double Power) GetEngine2()
{
    return ("M270 Turbo", 1600, 75.0);
}
```

从 C# 7.3 开始，可以使用`==`和`!=`运算符测试值元组的*相等性*和*不相等性*。这些运算符通过按顺序比较左侧的每个元素与右侧的每个元素来工作。当第一对不相等时，比较停止。但是，这仅在元组的形状相同时发生，即字段的数量和它们的类型。名称不参与相等性或不相等性的测试。下一个示例比较了两个值元组：

```cs
var e1 = ("M270 Turbo", 1600, 75.0);
var e2 = (Name: "M270 Turbo", Capacity: 1600, Power: 75.0);
Console.WriteLine(e1 == e2);
```

**元组相等**如果一个元组是可空元组，则执行*提升转换*，以及对两个元组的每个成员进行*隐式转换*。后者包括提升转换、扩展转换或其他隐式转换。例如，以下元组是相等的：

```cs
(int, long) t1 = (1, 2);
(long, int) t2 = (1, 2);
Console.WriteLine(t1 == t2);
```

可以解构元组的值。可以通过显式指定变量的类型或使用`var`来实现。以下声明都是等效的。在以下和最后一个示例中，`var`的使用与显式类型名称相结合：

```cs
(string name, int capacity, double power) = GetEngine();
(var name, var capacity, var power) = GetEngine();
var (name, capacity, power) = GetEngine();
(var name, var capacity, double power) = GetEngine();
```

如果有您不感兴趣的值，可以使用`_`占位符来忽略它们，如下所示：

```cs
(var name, _, _) = GetEngine();
```

可以对任何.NET 类型进行解构，只要提供了一个名为`Deconstruct`的方法，该方法具有您想要检索的每个值的`out`参数。

在下面的示例中，`Engine`类有三个属性：`Name`，`Capacity`和`Power`。`Deconstruct()`公共方法使用三个输出参数匹配这些属性。这使得可以使用元组语法对此类型的对象进行解构。以下清单显示了提供元组解构的`Engine`类的实现：

```cs
class Engine
{
    public string Name { get; }
    public int Capacity { get; }
    public double Power { get; }
    public Engine(string name, int capacity, double power)
    {
        Name = name;
        Capacity = capacity;
        Power = power;
    }
    public void Deconstruct(out string name, out int capacity, 
                            out double power)
    {
        name = Name;
        capacity = Capacity;
        power = Power;
    }
}
var engine = new Engine("M270 Turbo", 1600, 75.0);
var (Name, Capacity, Power) = engine;
```

`Deconstruct`方法可以作为扩展方法提供，使您能够为您没有编写的类型提供解构语义，前提是您只需要解构通过类型的公共接口可访问的值。这里展示了一个示例：

```cs
class Engine
{
    public string Name { get; }
    public int Capacity { get; }
    public double Power { get; }
    public Engine(string name, int capacity, double power)
    {
        Name = name;
        Capacity = capacity;
        Power = power;
    } 
}
static class EngineExtension
{
    public static void Deconstruct(this Engine engine, 
                                   out string name, 
                                   out int capacity, 
                                   out double power)
    {
        name = engine.Name;
        capacity = engine.Capacity;
        power = engine.Power;
    }
}
```

如果您有一个类的层次结构，并且提供了`Deconstruct()`方法，则必须确保不会引入歧义，例如在不同重载具有相同数量的参数的情况下。应该注意，解构运算符不参与测试相等性。因此，以下示例将生成编译器错误：

```cs
var engine = new Engine("M270 Turbo", 1600, 75.0);
Console.WriteLine(engine == ("M270 Turbo", 1600, 75.0));
```

总结一下，C# 7 中对值元组的支持使得在关键场景中更容易使用元组，比如保存临时值或来自数据库的记录。这可以在不引入新类型或返回多个值的情况下完成，而不使用`out`或`ref`参数。通过值语义的性能优势以及基于名称的元素访问的改进，以及其他关键特性，命名值是本节开始时看到的引用类型元组的重要改进。

# 模式匹配

在`if`和`switch`语句中，我们检查对象是否具有某个值，然后继续从中提取信息。然而，这是一种基本形式的模式匹配。

在 C# 7 中，对`is`和`switch`语句添加了新的功能，以实现模式匹配功能，从而更好地分离数据和代码，并导致更简洁和可读的代码。C# 8 中的新功能扩展了模式匹配功能。您将在*第十五章*中了解这些内容，*C# 8 的新功能*。

## is 表达式

在运行时，`is`运算符检查对象是否与给定类型兼容（一般形式为`expr is type`）。然而，在 C# 7 中，这被扩展为包括几种形式的模式匹配：

+   `expr is type varname`形式，检查表达式是否可以转换为指定类型，如果可以，则将其转换为指定类型的变量。

+   `expr is constant`形式，检查表达式是否评估为指定的常量。特定常量是`null`，其模式为`expr is null`。

+   `expr is var varname`形式，总是成功并将值绑定到一个新的局部变量。与类型模式的一个关键区别是`null`总是匹配，并且新变量被赋值为`null`。

为了理解这些工作原理，我们将使用几个代表车辆的类：

```cs
class Airplane
{
    public void Fly() { }
}
class Bike
{
    public void Ride() { }
}
class Car
{
    public bool HasAutoDrive { get; }
    public void Drive() { }
    public void AutoDrive() { }
}
```

这些车辆类不是类层次结构的一部分，但它们有设置车辆运动的公共方法，根据其类型。例如，飞机飞行，自行车骑行，汽车驾驶。下一个代码清单显示了使用几种形式的模式匹配的函数：

```cs
void SetInMotion(object vehicle)
{
    if (vehicle is null)
        throw new ArgumentNullException(
            message: "Vehicle must not be null",
            paramName: nameof(vehicle));
    else if (vehicle is Airplane a)
        a.Fly();
    else if (vehicle is Bike b)
        b.Ride();
    else if (vehicle is Car c)
    {
        if (c.HasAutoDrive) c.AutoDrive();
        else c.Drive();
    }
    else
        throw new ArgumentException(
           message: "Unexpected vehicle type", 
           paramName: nameof(vehicle)); 
}
```

该函数根据其特定的方式使车辆运动起来。像`if(vehicle is Airplane a)`这样的语句测试变量 vehicle 是否可以转换为`Airplane`类型，如果是，则将其分配给`Airplane`类型的新变量（在本例中为`a`）。这适用于值类型和引用类型。

这里看到的变量`a`、`b`和`c`只在`if`或`else`语句的局部范围内。然而，只有在匹配成功时，这些变量才在范围内并被赋值。这可以防止您在模式匹配表达式未匹配时访问结果。

除了类型模式，这里还使用了常量模式。`if (vehicle is null)`语句是一个测试，用于查看引用是否实际设置为对象的实例；如果没有，就会抛出异常。然而，如前所述，常量模式匹配可以与任何常量一起使用——文字值、用 const 修饰符声明的变量，或者枚举值。常量表达式的评估方式如下：

+   如果`expr`和常量都是整数类型，它基本上评估`expr == constant`表达式。

+   否则，它调用静态方法`Object.Equals(expr, constant)`。

以下函数显示了更多的常量模式匹配示例。`IsTrue()`函数将提供的参数转换为布尔值。布尔值（`true`），整数值（`1`），字符串（`"1"`）和字符串（`"true"`）都转换为`true`；包括`null`在内的其他所有内容都转换为`false`：

```cs
bool IsTrue(object value)
{
    if (value is null) return false;
    else if (value is 1) return true;
    else if (value is true) return true;
    else if (value is "true") return true;
    else if (value is "1") return true;
    return false;
}
Console.WriteLine(IsTrue(null));   // False
Console.WriteLine(IsTrue(0));      // False
Console.WriteLine(IsTrue(1));      // True
Console.WriteLine(IsTrue(true));   // True
Console.WriteLine(IsTrue("true")); // True
Console.WriteLine(IsTrue("1"));    // True
Console.WriteLine(IsTrue("demo")); // False
```

## switch 表达式

您需要检查的模式越多，编写这些`if-else`语句就越繁琐。自然地，您会想用`switch`替换它们。相同类型的模式匹配也支持`switch`语句，具有类似的语法。

直到 C# 7.0，`switch`语句支持整数类型和字符串的常量模式匹配。自 C# 7.0 以来，前面看到的类型模式也支持在`switch`语句中。

在前一节中显示的`SetInMotion()`函数可以修改为使用`switch`语句：

```cs
void SetInMotion(object vehicle)
{
    switch (vehicle)
    {
        case Airplane a:
            a.Fly();
            break;
        case Bike b:
            b.Ride();
            break;
        case Car c:
            if (c.HasAutoDrive) c.AutoDrive();
            else c.Drive();
            break;
        case null:
            throw new ArgumentNullException(
                message: "Vehicle must not be null",
                paramName: nameof(vehicle));
        default:
            throw new ArgumentException(
               message: "Unexpected vehicle type", 
               paramName: nameof(vehicle));
    }
}
```

使用常量模式匹配的`switch`语句只能有一个与`switch`表达式的值匹配的情况标签。此外，`switch`部分不能穿过下一个部分，而必须以`break`、`return`或`goto`结束。然而，它们可以以任何顺序排列，而不会影响程序语义和执行的行为。

使用类型模式匹配，规则会发生变化。`switch`部分可以穿过下一个，`goto`不再支持作为跳转机制。情况标签表达式按照它们在文本中出现的顺序进行评估，只有在没有任何情况标签与模式匹配时才执行默认情况。默认情况可以出现在`switch`的任何位置，但始终在最后执行。

如果默认情况缺失，并且没有任何现有的情况标签与模式匹配，执行将在`switch`语句之后继续，而不会执行任何情况标签中的代码。

`switch`表达式的类型模式匹配还支持`when`子句。以下示例展示了`SetInMotion()`方法的另一个版本，它使用了两个 case 标签来匹配`Car`类型，但其中一个带有条件——即`Car`对象的`HasAutoDrive`属性设置为`true`：

```cs
void SetInMotion(object vehicle)
{
    switch (vehicle)
    {
        case Airplane a:
            a.Fly();
            break;
        case Bike b:
            b.Ride();
            break;
        case Car c when c.HasAutoDrive:
            c.AutoDrive();
            break;
        case Car c:
            c.Drive();
            break;
        case null:
            throw new ArgumentNullException(
                message: "Vehicle must not be null",
                paramName: nameof(vehicle));
        default:
            throw new ArgumentException(
              message: "Unexpected vehicle type", 
              paramName: nameof(vehicle)); 
    }
}
```

需要注意的是，匹配类型模式保证了*非空值*，因此不需要进一步测试`null`。对于在语言中匹配`null`有特殊规则。`null`值不匹配类型模式，无论变量的类型如何。可以在具有类型模式匹配的 switch 表达式中添加一个用于特别处理`null`值的模式匹配的 case 标签。在前面的实现中就有这样的例子。

一种特殊的类型模式匹配形式是使用`var`。规则与`is`表达式相似——类型是从 switch 表达式的静态类型中推断出来的，而`null`值总是匹配的。因此，在使用`var`模式时，您必须添加显式的`null`检查，因为值实际上可能是`null`。`var`声明可能与默认情况匹配相同的条件；在这种情况下，即使存在默认情况，它也永远不会执行。

让我们看一下以下函数，它执行作为字符串参数接收的命令：

```cs
void ExecuteCommand(string command)
{
    switch(command)
    {
        case "add":  /* add */    break;
        case "del":  /* delete */ break;
        case "exit": /* exit */   break;
        case var o when (o?.Trim().Length ?? 0) == 0:
            /* do nothing */
            break;
        default:
            /* invalid command */
            break;
    }
}
```

这个函数尝试匹配`add`、`del`和`exit`命令，并适当地执行它们。但是，如果参数是`null`、空或只包含空格，它将不执行任何操作。但这与不支持或无法识别的实际命令是不同的情况。`var`模式匹配有助于以简单而优雅的方式区分这两种情况。

以下是本主题的关键要点：

+   C# 7.0 中添加的模式匹配功能是对已有简单模式匹配能力的增量更新。

+   新支持的模式包括常量模式、类型模式和`var`模式。

+   模式匹配与`is`表达式和`switch`语句中的 case 块一起工作。

+   `switch`表达式模式匹配支持`where`子句。

+   `var`模式始终匹配任何值，包括`null`，因此需要进行`null`测试。

C# 8.0 还为 switch 表达式模式匹配引入了更多功能：属性模式、元组模式和位置模式。您可以在*第十五章*中了解这些内容，*C# 8 的新功能*。

# 正则表达式

另一种模式匹配形式是正则表达式。`System.Text.RegularExpressions`命名空间。在接下来的页面中，我们将看看如何使用这个类来匹配输入文本，找到其中的部分，或替换文本的部分。

正则表达式由常量（代表字符串集合）和操作符号（代表对这些集合进行操作的操作符）组成。构建正则表达式的实际语言比本章节的范围所能描述的更加复杂。如果您对正则表达式不熟悉，我们建议您使用其他资源来学习。您也可以使用在线工具（例如 https://regex101.com/或 https://regexr.com/）构建和测试您的正则表达式。

## 概述

.NET 中的正则表达式是基于 Perl 5 正则表达式构建的。因此，大多数 Perl 5 正则表达式与.NET 正则表达式兼容。另一方面，该框架支持另一种表达式风格，称为**ECMAScript**，这基本上是 JavaScript 的另一个名称（**ECMAScript**实际上是脚本语言的 ECMA 标准，JavaScript 是其最著名的实现）。但是，在使用正则表达式时，您必须明确指定此风格。自.NET 2.0 以来，.NET 正则表达式的实现保持不变，在.NET Core 中也是如此。

以下是此实现支持的一些功能：

+   不区分大小写匹配

+   从右到左搜索（用于具有从右到左书写系统的语言，如阿拉伯语、希伯来语或波斯语）

+   多行或单行搜索模式，改变一些符号的含义，如`ˆ`、`$`或`.`（点）

+   将正则表达式编译为程序集，并在使用模式搜索大量字符串时提高性能的可能性

+   无限宽度的后行断言使我们能够向后移动到任意长度，并在字符串中检查后行断言内的文本是否可以在那里匹配

+   字符类减法允许您从另一个字符类中指定一个字符类来减去

+   平衡组允许您确保子表达式与另一个子表达式匹配的类型数量相等

其中一些功能是通过作为`Regex`类构造函数参数提供的标志来启用的。`RegexOptions`枚举提供以下标志，可以组合使用：

![](img/Chapter_8_Table_1_01.jpg)

在我们转到下一节来看如何在 C#中实际使用正则表达式之前，还有两件重要的事情要提到：

+   首先，正则表达式具有一组特殊字符。其中之一是`\`（反斜杠）。与另一个文字字符结合使用时，这将创建一个具有特殊含义的新标记。例如，`\d`匹配 0 到 9 之间的任何单个数字。由于反斜杠在 C#中也是一个特殊字符，用于引入字符转义序列，因此在字符串中编写正则表达式时，您需要使用双反斜杠，例如`"(\\d+)"`。但是，您可以使用逐字字符串来避免这种情况，并保持正则表达式的自然形式。前面的示例可以写成`@"(\d+)"`。

+   另一个重要的事情是`Regex`类隐式假定要匹配的字符串采用 UTF-8 编码。这意味着`\w`、`\d`和`\s`标记匹配任何 UTF-8 代码点，该代码点是任何语言中的有效字符、数字或空白字符。例如，如果您使用`\d+`来匹配任意数量的数字，您可能会惊讶地发现它不仅匹配 0-9，还匹配以下字符：![](img/Figure_8.1_B12346.jpg)

如果要将匹配限制为`\d`的英文数字，`\w`的英文数字和字母以及下划线，以及`\s`的标准空白字符，则需要使用`RegexOptions.ECMAScript`选项。

现在让我们看看如何定义正则表达式并使用它们来确定某些文本是否与表达式匹配。

## 匹配输入文本

正则表达式提供的最简单功能是检查输入字符串是否具有所需的格式。这对于执行验证非常有用，例如检查字符串是否是有效的电子邮件地址、IP 地址、日期等。

为了理解这是如何工作的，我们将验证输入文本是否是有效的 ISO 8061 日期。为简单起见，我们只考虑*YYYY-MM-DD*的形式，但是作为练习，您可以扩展此以支持其他格式。我们将用于此的正则表达式是`(\d{4})-(1[0-2]|0[1-9]|[0-9]{1})-(3[01]|[12][0-9]|0[1-9]|[1-9]{1})`。

分解成部分，子表达式如下：

![](img/Chapter_8_Table_2_01.jpg)

以下两个例子是等价的。`Regex`类对于`IsMatch()`有静态和非静态的重载，你可以使用任何一个得到相同的结果。其他方法也是如此，我们将在接下来的章节中看到，比如`Match()`、`Matches()`、`Replace()`和`Split()`：

```cs
var pattern = @"(\d{4})-(1[0-2]|0[1-9]|[1-9]{1})-(3[01]|[12][0-9]|0[1-9]|[1-9]{1})";
var success = Regex.IsMatch("2019-12-25", pattern);
// or
var regex = new Regex(pattern);
var success = regex.IsMatch("2019-12-25");
```

如果你只需要匹配一个模式一次或几次，那么你可以使用静态方法，因为它们更简单。然而，如果你需要匹配数万次或更多次相同的模式，使用类的实例并调用非静态成员可能更快。对于大多数常见的用法，情况并非如此。在下面的例子中，我们将只使用静态方法。

`IsMatch()`方法有一些重载，使我们能够为正则表达式指定选项和超时时间间隔。当正则表达式过于复杂，或者输入文本过长，解析所需的时间超过了期望的时间时，这是很有用的。看下面的例子：

```cs
var success = Regex.IsMatch("2019-12-25",
                            pattern,
                            RegexOptions.ECMAScript,
                            TimeSpan.FromMilliseconds(1));
```

在这里，我们启用了正则表达式的 ECMAScript 兼容行为，并设置了一毫秒的超时值。

现在我们已经看到了如何匹配文本，让我们学习如何搜索子字符串和模式的多次出现。

## 查找子字符串

到目前为止的例子中，我们只检查了输入文本是否符合特定的模式。但也可以获取有关结果的信息。例如，每个标题组中匹配的文本、整个匹配值、输入文本中的位置等。为了做到这一点，必须使用另一组重载。

`Match()`方法检查输入字符串中与正则表达式匹配的子字符串，并返回第一个匹配项。`Matches()`方法也进行相同的搜索，但返回所有匹配项。前者的返回类型是`System.Text.RegularExpressions.Match`（表示单个匹配项），后者的返回类型是`System.Text.RegularExpressions.MatchCollection`（表示匹配项的集合）。考虑下面的例子：

```cs
var pattern =
    @"(\d{4})-(1[0-2]|0[1-9]|[1-9]{1})-(3[01]|[12][0-9]|0[1-9]|[1-9]{1})";
var match = Regex.Match("2019-12-25", pattern);
Console.WriteLine(match.Value);
Console.WriteLine(
    $"{match.Groups[1]}.{match.Groups[2]}.{match.Groups[3]}");
```

控制台打印的第一个值是`2019-12-25`，因为这是整个匹配的值。第二个值是由每个捕获组的单独值组成的，但是用点(`.`)作为分隔符。因此，输出文本是`2019.12.25`。

捕获组可能有名称；形式为`(?<name>...)`。在下面的例子中，我们称正则表达式的三个捕获组为`year`、`month`和`day`：

```cs
var pattern =
    @"(?<year>\d{4})-(?<month>1[0-2]|0[1-9]|[1-9]{1})-(?<day>3[01]|[12][0-9]|0[1-9]|[1-9]{1})";
var match = Regex.Match("2019-12-25", pattern);
Console.WriteLine(
    $"{match.Groups["year"]}-{match.Groups["month"]}-{match.Groups["day"]}");
```

如果输入文本有多个与模式匹配的子字符串，我们可以使用`Matches()`函数获取所有这些子字符串。在下面的例子中，日期每行提供一个，但最后两个日期不合法（`2019-13-21`和`2019-1-32`）；因此，这些在结果中找不到。为了解析字符串，我们使用了多行选项，这样`^`和`$`就分别指向每行的开头和结尾，而不是整个字符串，如下面的例子所示：

```cs
var text = "2019-05-01\n2019-5-9\n2019-12-25\n2019-13-21\n2019-1-32";
var pattern =
    @"^(\d{4})-(1[0-2]|0[1-9]|[1-9]{1})-(3[01]|[12][0-9]|0[1-9]|[1-9]{1})$";
var matches = Regex.Matches(
  text, pattern, RegexOptions. Multiline); 
foreach(Match match in matches)
{
    Console.WriteLine(
      $"[{match.Index}..{match.Length}]={match. Value}");
}
```

程序的输出如下：

```cs
[0..10]=2019-05-01
[11..8]=2019-5-9
[20..10]=2019-12-25
```

有时，我们不仅想要找到输入文本的子字符串；我们还想用其他东西替换它们。这个主题在下一节中讨论。

## 替换文本的部分

正则表达式也可以用来用另一个字符串替换匹配正则表达式的字符串的部分。`Replace()`方法有一组重载，你可以指定一个字符串或一个所谓的`Match`参数，并返回一个字符串。在下面的例子中，我们将使用这个方法将日期的格式从*YYYY-MM-DD*改为*MM/DD/YYYY*：

```cs
var text = "2019-12-25";
var pattern = @"(\d{4})-(1[0-2]|0[1-9]|[1-9]{1})-(3[01]|[12]
    [0-9]|0[1-9]|[1-9]{1})";
var result = Regex.Replace(
    text, pattern,
    m => $"{m.Groups[2]}/{m.Groups[3]}/{m.Groups[1]}");
```

作为进一步的练习，你可以编写一个程序，将形式为 2019-12-25 的输入日期转换为 Dec 25, 2019 的形式。

作为本节的总结，正则表达式提供了丰富的模式匹配功能。.NET 提供了代表具有丰富功能的正则表达式引擎的 `Regex` 类。在本节中，我们已经看到了如何基于模式匹配、搜索和替换文本。这些是您将在各种应用程序中遇到的常见操作。您可以选择这些方法的静态和实例重载，并使用各种选项自定义它们的工作方式。

# 扩展方法

有时候，向类型添加功能而不改变实现、创建派生类型或重新编译代码是很有用的。我们可以通过在辅助类中创建方法来实现这一点。假设我们想要一个函数来颠倒字符串的内容，因为 `System.String` 没有这样的函数。这样的函数可以实现如下：

```cs
static class StringExtensions
{
    public static string Reverse(string s)
    {
        var charArray = s.ToCharArray();
        Array.Reverse(charArray);
        return new string(charArray);
    }
}
```

可以按以下方式调用：

```cs
var text = "demo";
var rev = StringExtensions.Reverse(text);
```

C#语言允许我们以一种使我们能够调用它就像它是 `System.String` 的实际成员的方式来定义这个函数。这样的函数被称为 `Reverse()` 方法，使其成为扩展方法。新的实现如下所示：

```cs
static class StringExtensions
{
    public static string Reverse(this string s)
    {
        var charArray = s.ToCharArray();
        Array.Reverse(charArray);
        return new string(charArray);
    }
}
```

请注意，实现的唯一变化是在函数参数前面加上了 `this` 关键字。通过这些变化，函数可以被调用，就好像它是字符串类的一部分：

```cs
var text = "demo";
var rev = text.Reverse();
```

扩展方法的定义和行为适用以下规则：

+   它们可以扩展类、结构和枚举。

+   它们必须声明为静态、非嵌套、非泛型类的静态方法。

+   它们的第一个参数是它们要添加功能的类型。该参数前面带有 `this` 关键字。

+   它们只能调用它们扩展的类型的公共成员。

+   只有当它们声明的命名空间通过 `using` 指令引入到当前范围时，扩展方法才可用。

+   如果一个扩展方法（在当前范围内可用）与类的实例方法具有相同的签名，编译器将始终优先选择实例成员，扩展方法将永远不会被调用。

以下示例显示了一个名为 `AllMessages()` 的扩展方法，它扩展了 `System.Exception` 类型的功能。这代表了一个异常，有一个消息，但也可能包含内部异常。这个扩展方法返回一个由所有嵌套异常的所有消息连接而成的字符串。布尔参数指示是否应该从主异常到最内部异常连接消息，还是以相反的顺序：

```cs
static class ExceptionExtensions
{
    public static string AllMessages(this Exception exception, 
                                     bool reverse = false)
    {
        var messages = new List<string>();
        var ex = exception;
        while(ex != null)
        {
            messages.Add(ex.Message);
            ex = ex.InnerException;
        }
        if (reverse) messages.Reverse();
        return string.Join(Environment.NewLine, messages);
    }
}
```

然后可以按以下方式调用扩展方法：

```cs
var exception = 
    new InvalidOperationException(
        "An invalid operation occurred",
        new NotSupportedException(
            "The operation is not supported",
            new InvalidCastException(
                "Cannot apply cast!")));
Console.WriteLine(exception.AllMessages());
Console.WriteLine(exception.AllMessages(true));
```

来自.NET 的最常见的扩展方法是扩展 `IEnumerable` 和 `IEnumerable<T>` 类型的 LINQ 标准运算符。我们将在*第十章* *Lambdas, LINQ, and Functional Programming*中探讨 LINQ。如果您实现扩展方法来扩展无法更改的类型，您必须牢记将来对类型的更改可能会破坏扩展方法。

# 总结

在本章中，我们讨论了一系列高级语言特性。我们从实现强类型回调的委托和事件开始。我们继续讨论了匿名类型和元组，这些是轻量级类型，可以保存任何值，并帮助我们避免定义新的显式类型。然后我们看了模式匹配，这是检查值是否具有特定形状以及提取有关它的信息的过程。我们继续讨论了正则表达式，这是具有明确定义的语法的模式，可以与文本匹配。最后，我们学习了扩展方法，它使我们能够向类型添加功能，而不改变它们的实现，比如当我们不拥有源代码时。

在下一章中，我们将讨论垃圾回收和资源管理。

# 测试你学到的知识

1.  什么是回调函数，它们与委托有什么关系？

1.  你如何定义委托？事件又是什么？

1.  有多少种类型的元组？它们之间的主要区别是什么？

1.  什么是命名元组，如何创建它们？

1.  什么是模式匹配，它可以与哪些语句一起使用？

1.  模式匹配空值的规则是什么？

1.  哪个类实现了正则表达式，它默认使用什么编码？

1.  这个类的`Match()`和`Matches()`方法有什么区别？

1.  什么是扩展方法，它们为什么有用？

1.  你如何定义一个扩展方法？
