# 第十五章：*第十五章*：C# 8 的新功能

C#是一种成熟的编程语言，但它仍在不断发展，以满足新兴软件架构带来的新需求。C# 7 的四个语言版本的主要重点是提供在使用值类型时令人印象深刻的性能工具。

随着最新版本的推出，C# 8 引入了许多重要的新功能，重点放在四个主要领域：使代码更紧凑、更易阅读，以及性能、健壮性和表现力。C# 8 的根本变化在于它是该语言的第一个版本，没有在.NET Framework 中获得官方支持，因为其中一些功能需要.NET Core 运行时的增强。

在本章中，我们将介绍以下新的语言特性：

+   可空引用类型

+   接口成员的默认实现

+   范围和索引

+   模式匹配

+   使用声明

+   异步 Dispose

+   结构和 ref 结构中的可处置模式

+   异步流

+   只读结构成员

+   空合并赋值

+   静态局部函数

+   更好的插值原始字符串

+   在嵌套表达式中使用 stackalloc

+   未管理的构造类型

在本章结束时，您将了解使用每个功能的用例，并能够逐步在您的列表中采用它们。一如既往，您越多地将这些功能付诸实践，您就越快掌握它们。

我们现在将开始介绍一种语言特性，它有着减少.NET 应用程序中主要崩溃原因之一`NullReferenceException`的伟大抱负。

# 可空引用类型

在上一章中，我们了解到 C#中的类型系统分为**引用类型**和**值类型**。值类型分配在堆栈上，并且每次分配给新变量时都会进行内存复制。另一方面，引用类型分配在堆上，由垃圾收集器管理。每当我们分配一个新的引用类型时，我们都会收到一个引用，作为标识分配的内存的关键，从垃圾收集器那里。

引用本质上是一个指针，可以假定特殊的空值，这是指示值的缺失的最简单、因此最流行的方式。请记住，除了使用空值之外，另一个解决方案是采用特殊情况的架构模式，它的最简单形式是该对象的一个实例，其中包含一个布尔字段，指示对象是否有效，这就是`Nullable<T>`的工作原理。在许多其他情况下，开发人员实际上不需要使用空值，对其进行验证需要大量的代码，这将影响运行时性能。

空引用的问题在于编译器无法讨论潜在问题，因为它在语法上是正确的，但在运行时对其进行取消引用将导致`NullReferenceException`，这是.NET 世界中应用程序崩溃的首要原因。

让我们暂时考虑一个简单的类，它有两个构造函数，只有第二个初始化了`_name`字段：

```cs
public class SomeClass
{
    private string _name;
    public SomeClass() { }
    public SomeClass(string name) { _name = name; }
    public int NameLength
    {
        get { return _name.Length; }
    }
}
```

当使用第一个构造函数时，`NameLength`属性将导致`NullReferenceException`。

在测试方面，这是突出显示以下两种情况的代码：

```cs
Assert.ThrowsException<NullReferenceException>(() => new SomeClass().NameLength);
Assert.IsTrue(new SomeClass("Raf").NameLength >= 0);
```

根本问题在于我们的代码行为取决于运行时的值，显然，编译器无法知道我们是否会在调用默认构造函数后初始化`_name`字段。

信息框

空引用是由 Tony Hoare 爵士在 1965 年发明的概念。然而，2009 年，他对自己的发明感到遗憾，称其为*我的十亿美元的错误*（https://en.wikipedia.org/wiki/Tony_Hoare）。由于无法轻松地从框架中删除空值，可空引用类型旨在使用代码分析方法解决这个问题。

这个概念在大多数编程语言中都很普遍，包括.NET 生态系统中的所有语言。这意味着任何试图从框架中移除空概念的努力都将是一个巨大的破坏性变化，可能会破坏当前的应用程序。编译器可以做什么来解决这个问题？答案是进行*静态代码分析*，这是一种在不运行代码的情况下了解源代码运行时行为的技术。

信息框

2011 年，微软开始着手进行一项名为`Microsoft.CodeAnalysis`的革命性项目，即公开为编译器通常完成的所有处理提供 API。

传统上，编译器是黑匣子，但 Roslyn 使得可以以编程方式解析源代码，获取语法和语义树，使用访问者检索或重写特定节点，并分析源代码的语义。

当开发人员在编辑器中看到黄色灯泡或代码下方的波浪线建议进行一些重构或指出潜在问题时，他们可能已经看到了静态代码分析在 Visual Studio 中的工作。通过编写自定义分析器，这些能力可以进一步扩展，分发为 Visual Studio 扩展或 NuGet 包。

由于静态代码分析无法知道引用在运行时所假定的值，它只检查所有可能的使用路径，并尝试判断其中是否可能会取消引用（使用点或方括号）。但是分析可以提出两种不同的策略，取决于是否希望引用假定空值：

+   我们可能希望防止引用假定空值。在这种情况下，分析器将建议在声明或构造时进行初始化，并在任何后续赋值时进行初始化。

+   我们可能需要引用假定空值。在这种情况下，分析器将验证是否有足够的空检查代码（`if`语句或类似的）来避免任何可能取消引用空值的路径。

在这两种策略之间的选择是开发人员的选择，开发人员需要提供额外的信息，以便编译器知道应该提供哪种反馈。

C# 8 可空引用类型功能支持两种策略的高级静态代码分析功能，这得益于能够注释引用以告知编译器有关预期引用使用的能力。为此，C#语法已经扩展，提供了装饰引用类型为可空的能力。根据这个新规则，在前面示例类中声明的字符串字段假定引用不为空，并且必须在构造时初始化：

```cs
private string _name;	// must be initialized at construction time
```

当开发人员希望给编译器一个提示，使`_name`引用可能为空时，他们必须用问号装饰来声明它：

```cs
private string? _name;
```

使用问号字符作为装饰符并不是新的；它在 C# 2 中引入，将`Nullable<T>`声明缩短为`T?`，并且包括将*值类型*包装到一个结构中，使用布尔字段来知道值类型是否设置为空。

引用类型的问号装饰在 C# 8 中是新的，其含义类似，但不涉及包装。相反，这种装饰只是一种通知代码分析有关引用预期使用的方式。

默认情况下，代码分析是关闭的，因为现有的应用程序总是假定任何引用都可能为空，并且在现有代码上默认启用它将导致大量的波浪线和编译器消息遍布整个代码。

注意

当引用用问号装饰，并且可空引用类型功能尚未启用时，Visual Studio 会用绿色波浪线标记问号，建议问号功能尚未生效，因为该功能尚未激活。

除了问号外，C#还添加了**宽容运算符**，表示为*感叹号*，用于通知代码分析在特定情况下*宽容*一个语句。使用宽容运算符是很少见的，因为这意味着分析未能识别开发人员自己知道引用不可能为空的情况。其使用的一个现实例子是当一些不安全/本地代码改变了由引用指向的内存值，而托管代码中没有任何证据。在其他非常极端的情况下，纯托管代码可能非常复杂，编译器无法识别它。我个人会选择简化代码而不是使用宽容运算符。

请记住，在*声明*引用时使用*问号*，在*取消引用*时使用*感叹号*。以下示例显示了一个语句，该语句将不会从静态代码分析中进行分析，并且不会提供任何反馈，因为开发人员在强烈承诺引用永远不会为空：

```cs
var len = _name!.Length;
```

值得重申，它应该只在极其罕见的情况下使用。

## 启用可空引用类型功能

有多种选项可以启用此功能；这样做的原因是能够逐步调整现有代码上的功能，而不会被阻塞或收到大量消息。每次启动新项目时，您可能希望完全启用此功能，以避免通过打开 Visual Studio 解决方案资源管理器、双击项目节点并编辑`.csproj`文件而受到过多的干扰。或者，您可以右键单击项目节点，然后从上下文菜单中选择**编辑项目文件**。

通过添加可空的 XML 标记，该功能将在整个项目中启用，这是在启动新项目时的最佳选项：

```cs
<PropertyGroup>
  <TargetFramework>netcoreapp3.0</TargetFramework>
  <Nullable>enable</Nullable>
</PropertyGroup>
```

您可以在现有项目上执行相同的操作，但编译器提供的反馈可能过多，会分散开发人员的注意力。因此，C#编译器提供了四个新的编译指示，可以在选定的代码部分启用和禁用该功能。有趣的是，restore 编译指示可以恢复先前定义的设置，以允许编译指示的嵌套：

```cs
#nullable enable
public class SomeClass
{
    private string? _name;
    public SomeClass() { }
    public SomeClass(string name) { _name = name; }
    public int NameLength
    {
        // you should see a green squiggle below _name
        get { return _name.Length; }
    }
}
#nullable restore
```

该功能的可能设置范围使其能够实现其他细微差别，具体取决于您是否希望能够使用修饰符（问号和感叹号），和/或在可能导致`NullReferenceException`的代码上获得警告：

+   **同时启用警告和注释**：这是通过仅启用该功能来完成的，正如我们之前提到的。根据此规则，可以使用问号对代码进行注释，以提示编译器有关引用的预期使用。代码编辑器将显示任何潜在问题，编译器将为这些问题生成警告：

```cs
Csproj: <Nullable>enable</Nullable>
Code: #nullable enable
```

+   `Nullable`功能可以在整个项目或选定的代码部分上使用：

```cs
Csproj: <Nullable>disable</Nullable>
Code: #nullable disable
```

+   **仅启用注释而不启用编译器警告**：在现有项目中采用此功能时，可以非常有用地开始注释代码而不在 IDE 或编译器输出中收到任何警告。值得记住的是，许多公司强制执行门控检入，拒绝产生警告的任何代码。在这种情况下，可以在整个项目中启用注释，并逐渐逐个文件启用警告，以逐步迁移代码：

```cs
Csproj: <Nullable>annotations</Nullable>
Code: #nullable enable annotations
```

+   `NullReferenceException`：

```cs
Csproj: <Nullable>warnings</Nullable>
Code: #nullable enable warnings
```

+   **在代码中恢复先前的设置（仅在代码文件中）**：在使用编译指示时，最好使用恢复编译指示标记给定区域的结束，而不是启用/禁用，以使嵌套区域正确运行：

```cs
#nullable restore annotations
#nullable restore warnings
```

+   **选择性地禁用设置（仅在代码文件中）**：最终设置是用于有选择地在代码的给定区域禁用注释或警告的设置。当您希望应用逆逻辑时，即为整个项目启用该功能并仅禁用代码的选定部分时，这将非常有用：

```cs
#nullable disable annotations
#nullable disable warnings
```

在现有项目中采用此功能时，这种细粒度控制可空引用类型功能的能力非常重要。除此之外，您可能会发现在整个项目范围内启用它更简单。

## 使用可空引用类型

一旦启用，代码分析将在代码编辑器中提供反馈，具体取决于引用是否已用问号装饰。开发人员可以选择不装饰变量，这意味着引用永远不应假定为空值。在这种情况下，声明看起来非常熟悉：

```cs
private string _name;
```

在这里，代码分析将标记负责不初始化字符串的构造函数代码，如果没有问号，则该字符串不能为 null。在这种情况下，补救措施很简单：您可以将`_name`变量初始化为一个空字符串，或者删除默认构造函数，强制所有调用者在创建对象时提供非空字符串。

另一种策略是将`_name`变量声明为可空的：

```cs
private string? _name;
```

当解除引用`Length`属性时，代码分析将显示绿色波浪线。在这种情况下，解决方案是显式检查`_name`是否为 null，并返回适当的值（或引发异常）。这是属性的可能实现：

```cs
public int NameLength2
{
    get
    {
        if (_name == null) return 0; else return _name.Length;
    }
}
```

以下是相同代码的另一种替代和更优雅的实现：

```cs
public int NameLength2 => _name?.Length ?? 0;
```

注释代码很简单，因为它类似于已经使用的可空类型的策略，但是对于数组，装饰略微复杂，因为游戏中存在两种可能的引用类型：数组本身和数组中保存的项目。

字符串数组可以声明如下：

```cs
private string[]?  _names; // array can be null
private string?[]  _names; // items in the array can be null
private string?[]? _names; // both the array and its items can
                           // be null
private string[]   _names; // neither of the two can be null
```

但请记住，我们使用的问号越多，我们需要做的空值检查就越多。让我们考虑这个简单的类：

```cs
public class OtherClass
{
    private string?[]? _names;
    public OtherClass() { }
    public int Count => _names?.Length ?? 0;
    public string GetItemLength(int index)
    {
        if (_names == null) return string.Empty;
        var name = _names[index];
        if (name == null) return string.Empty;
        return name;
    }
}
```

`Count`属性之所以短，仅因为我们使用了现代紧凑的语法，但它仍然包含了一个空值检查。`GetItemLength`返回数组中第 n 个项目的长度，由于数组和项目都可能为 null，因此需要两个不同的空值检查。

如果您只是考虑将`GetItemLength`方法的返回类型设置为`string?`，这种解决方案将使实现代码变得更短，但所有调用者都将被迫检查空值，需要进行更多的代码更改。

## 将现有代码迁移到可空引用类型

每个项目都有其自己的特点，但根据我的个人经验，我已经成功地确定了迁移现有项目到这一强大功能时的一些最佳实践。

第一个建议是从依赖树底部的项目开始启用此功能。在项目上下文中，您可能希望使用编译指示开始启用分析，从最常用的代码文件开始，例如助手、扩展方法等。

第二个建议是尝试避免使用问号：每次您用问号装饰引用时，代码分析都会要求您编写一些代码来证明空值解除引用不会发生，增加样板代码的数量，这可能会影响热路径的性能。

最后，当您使用这个功能编译一个库时，编译器会应用两个隐藏属性，以在元数据中留下关于代码中公开使用的引用的可空性的记录。每当编译引用您的库的一些代码时，编译器都会知道库方法是否接受可空引用，假设只有在属性明确宣传时才接受不可空引用参数。因此，在公共库中使用这个功能是最佳实践，这样其他人就可以从这些元数据中受益。

可空引用类型非常有用，可以减少运行时的`NullReferenceException`异常，这是应用程序崩溃的主要原因。

虽然这个功能是可选的，但使用编译指令逐渐应用所需的小改动以使代码具有空值保护是非常方便的。这是任何团队都应该将其技术债务添加到其中以提高代码质量的典型任务。除此之外，采用这一功能的库作者自动在其库中提供了可空性元数据，使整个引用链更加稳定。

# 接口成员的默认实现

我们已经学到，接口用于定义每个实现类型必须满足的合同。每个接口成员通过指定名称和其签名（输入和输出参数）来定义合同的一部分。然后，具体类型实现接口提供了定义成员的实现（或主体）。

通过*接口成员的默认实现*，C# 8 扩展了接口类型语法，包括以下功能：

+   接口现在可以为*方法*、*属性*、*索引器*和*事件*定义主体。

+   接口可以声明*静态成员*，包括*静态构造函数*和*嵌套类型*。

+   它们可以明确指定可见性修饰符，比如*private*、*protected*、*internal*和*public*（后者仍然是默认值）。

+   它们还可以指定其他修饰符，比如*virtual*、*abstract*、*sealed*、*extern*和*partial*。

这个新功能的语法很简单，就像给成员添加实现一样简单：

```cs
public interface ICalc
{
    int Add(int x, int y) => x + y;
    int Mul(int x, int y) => x * y;
}
```

乍一看，给接口成员添加实现似乎是矛盾的。事实上，前面的例子很好地演示了语法，但这绝对不是一个好的设计策略。您可能会想知道定义接口成员默认实现的一个好用例是什么。第一个原因是*接口版本控制*，传统上很难管理。

## 接口版本控制

例如，让我们从一个经典的接口`IWelcome`开始，声明两个简单的属性和一个`Person`类来实现它：

```cs
public interface IWelcome
{
    string FirstName { get; }
    string LastName { get; }
}
public class Person : IWelcome
{
    public Person(string firstName, string lastName)
    {
        this.FirstName = firstName;
        this.LastName = lastName;
    }
    public string FirstName { get; }
    public string LastName { get; }
}
```

现在可以添加一个具有默认实现的新方法：

```cs
public interface IWelcome
{
    string FirstName { get; }
    string LastName { get; }
    string Greet() => $"Welcome {FirstName}";
}
```

实现类不需要更新。它甚至可以驻留在不同的程序集中，而不会对接口的更改产生任何影响。

由于实现由接口提供，而类没有为`Greet`方法提供实现，因此仍然无法从`Person`引用中访问。换句话说，以下声明是不合法的：

```cs
var p = new Person("John", "Doe");
p.Greet(); // Wrong, Greet() is not available in Person
```

为了调用默认实现，我们需要一个`IWelcome`引用：

```cs
IWelcome p = new Person("John", "Doe");
Assert.AreEqual("Welcome John", p.Greet()); // valid code
```

这个功能对于一个历史悠久的接口的影响非常重要：例如，`List<T>`类公开了`AddRange`方法，而在`IList<T>`接口中不可用。在几乎 20 年的应用程序依赖于该接口之后，任何更改都将是一个巨大的破坏性变化。

接口上可能会发生什么变化？通过不鼓励使用`ObsoleteAttribute`来避免成员的使用，可以避免删除成员。也许几个版本之后，它将开始抛出`NotImplementedException`，而无需从接口中删除该成员。

改变成员总是一个不好的做法，因为接口是契约；通常，对于变更的需求可以通过使用不同名称和签名的新成员来建模。

因此，添加新成员是唯一的真正挑战，因为它会破坏二进制兼容性，并迫使每个接口实现者更改要求。例如，如果接口非常受欢迎，比如`IList<T>`，几乎不可能添加新成员，因为这将破坏所有人的代码。

传统上，接口版本问题是通过创建一个扩展先前接口的新接口来解决的，但这种解决方案并不实用，因为采用新接口需要实现者在对象继承声明中用新接口替换旧接口，并且当然要实现新成员。

C# 8 中的默认实现与普通类实现的行为不同，因为它为该层次结构定义了*基线*实现。假设您有一组接口层次结构和一个如下所示的类定义：

```cs
public interface IDog // defined in Assembly1
{
    string Name { get; }
    string Noise => "barks";
}
public interface ILabrador : IDog // defined in Assembly1
{
    int RetrieverAbility { get; }
}
public class Labrador : ILabrador // defined in Assembly2
{
    public Labrador(string name)
    {
        this.Name = name;
    }
    public string Name { get; }
    public int RetrieverAbility { get; set; }
}
```

在当前情况下，以下断言为真：

```cs
IDog archie = new Labrador("Archie");
Assert.AreEqual("barks", archie.Noise);
```

现在，修复`ILabrador`的默认实现并修改接口，如下所示：

```cs
public interface ILabrador : IDog
{
    int RetrieverAbility { get; }
    string IDog.Noise => "woofs"; // Version 2
}
```

值得注意的是，必须通过指定完整路径`IDog.Noise`来重新定义`Noise`方法。原因是因为.NET 允许接口进行多重继承；因此，在更复杂的继承结构中，可能会有多条路径导致`Noise`方法。

因此，语法要求指定完整路径以克服潜在的歧义。如果编译器发现无法通过指定完整路径解决的歧义，它将生成显式错误。

`ILabrador`的默认实现重新定义了`IDog`中`Noise`的基线实现。这意味着，即使我们使用的是`IDog`引用，`ILabrador`中的更改也会影响结果，如下所示：

```cs
IDog archie = new Labrador("Archie");
Assert.AreEqual("woofs", archie.Noise); 
```

此外，您可能已经注意到在前面示例的注释中，接口和类位于两个不同的程序集中。如果包含`ILabrador`的第一个程序集重新编译并添加了新成员，而第二个程序集保持不变，您仍将看到`Noise`被更新为`woofs`。这意味着修补第一个程序集将使所有应用程序受益于更新，即使不重新编译整个代码。

## 接口重新抽象

从派生接口重新定义默认实现的能力对于理解重新抽象是至关重要的。原则是相同的，但派生接口可以决定*擦除*默认接口实现，将成员标记为抽象。

继续上面的例子，我们可以定义以下接口：

```cs
public interface IYellowLabrador : ILabrador
{
    abstract string IDog.Noise { get; }
}
```

但是，这次，新接口的实现者需要实现`Noise`方法：

```cs
public class YellowLabrador : IYellowLabrador
{
    public YellowLabrador(string name)
    {
        this.Name = name;
    }
    public string Name { get; }
    public int RetrieverAbility { get; set; }
    public string Noise { get; set; }
}
```

这种能力很有用，因为默认实现是为了提供最佳实现，可以被层次结构中所有类型通用使用。但是，这些类型中的某个分支可能与该实现不太匹配，您希望在接口级别上擦除它以避免任何不当行为。

## 接口作为特质

详细讨论特质组合的概念需要一个完整的章节，但值得注意的是，C# 8 刚刚打开了特质的大门，让语言的未来版本有机会填补空白，您可以在 C#语言公共存储库的设计说明中阅读到相关内容。

**特质组合**是其他语言（如 C++）中众所周知的概念。它涉及定义一组成员以确定一个众所周知的行为。目标是定义不同类型（特质），以便任何类都能通过继承特质来组合自己的行为能力。

在这个语言的发布之前，我们通常会创建静态帮助类来定义一组可重用的行为。在 C# 8 中，我们可以将这些成员定义在接口内，这样它们可以通过继承接口来重用。接口的选择非常方便，因为.NET 只支持接口上的多重继承，允许多个特质被继承到一个新的类中。

如果你要尝试使用特质，试着在不考虑经典接口用法的情况下对其进行建模；相反，看看它们固有的能力，能够打开到多重继承，从而组合一组方法。

特质通常在你要定义的每个类的行为的可用性非常依赖于每个类时非常有用。在设计方面，这将转化为一个非常长的接口列表，每个接口定义一个单一的行为，或者一个单一的接口，许多对象通过抛出`NotImplementedException`来实现其部分方法。

让我们试着看一个非常简单的例子，你想要向你的应用程序公开一个字母转换服务。有多种实现方法：使用 Windows 本机 API、一个 NuGet 库或一个云服务。我们可能会尝试定义一个单一的接口，其中包含支持从一个字母表到另一个字母表的所有可能排列的长列表方法，但这并不是很实用，因为这些库或服务只支持所有可能的转换的一部分。这将导致许多实现抛出`NotImplementedException`。

另一种方法是为每种可能的转换定义一个接口，但实现这些接口的类需要将成员实现重定向到调用适当库的外部帮助类。

特质解决方案看起来更简单一些，因为它只是模拟我们可以做什么。例如，在这里，有两种可能的转换接口：

```cs
public interface ICyrillicToLatin
{
  public string Convert(string input)
  {
    return Transliteration.CyrillicToLatin(input, Language.Russian);
  }
}
public interface ILatinToCyrillic
{
  public string Convert(string input)
  {
    return Transliteration.LatinToCyrillic(input, Language.Russian);
  }
}
```

它们仍然是接口，但需要共同实现的类可以将接口添加到继承列表中，而无需其他任何操作：

```cs
class CompositeTransliterator : ICyrillicToLatin, ILatinToCyrillic
{
  // ...
}
```

最后，为了让消费者的生活更加便利，该类可以暴露一个使用模式匹配的开关表达式，以调用尝试转换成/从给定字母表的转换，并返回计算结果：

```cs
public string TransliterateCyrillic(string input)
{
    string result;
    return this switch
    {
        ICyrillicToLatin c when (result = c.Convert(input)) != input => result,
        ILatinToCyrillic l when (result = l.Convert(input)) != input => result,
        _ => throw new NotImplementedException("N/A"),
    };
}
```

这段代码尝试使用所有可用的服务对文本进行转换，如果其中一个服务由类实现，就尝试进行转换。一旦短语可以转换（即，转换结果与输入不同），就将其返回给调用者。

接口中的默认接口实现对所有实用主义者来说都是一个有价值的功能。Java 和 Swift 是已经支持这一功能的编程语言的例子。如果你是一个需要将你的代码移植到多种语言的库开发人员，它将使你的生活更加轻松，并避免重新设计代码的部分来克服它在语言先前版本中的缺失。

一如既往，建议明智地使用默认实现。如果用例已经很好地适应了以前的工具和模式，那么它就不会有用。

默认实现的一个有趣的边缘情况是，现在你可以用以下代码定义你的应用程序的入口点：

```cs
interface IProgram
{
    static void Main() => Console.WriteLine("Hello, world");
}
```

默认接口成员是一个具有争议性的功能，利用了.NET 接口支持多重继承的固有能力。实用主义者应该欣赏这个小革命所证明的实际用例，而其他人可以继续像以前一样使用接口。

现在我们可以继续下一个功能，这应该有助于避免在切片数组和列表时出现一些头痛和`IndexOutOfRangeException`异常。

# 范围和索引

C# 8 中引入的另一个方便的功能是用于标识序列中单个元素或范围的新语法。语言已经提供了使用方括号和数字索引在数组中获取或设置元素的能力，但通过添加两个运算符来标识从序列末尾获取项目和提取两个索引之间的范围，这个概念已经得到了扩展。

除了上述运算符之外，基类库现在还提供了两种新的系统类型，`System.Index`和`System.Range`，我们将立即看到它们的作用。让我们考虑一个包含六个国家名称的字符串数组：

```cs
var countries = new[] { "Italy", "Romania", "Switzerland", "Germany", "France", "England" };
var length = countries.Length;
```

我们已经知道如何使用数字索引器来获取对第一个项目的引用：

```cs
Assert.IsTrue(countries[0] == "Italy");
```

新的`System.Index`类型只是一个方便的包装器，可以直接用于数组上：

```cs
var italyIndex = new Index(0);
Assert.IsTrue(countries[0] == countries[italyIndex]);
```

有趣的部分是当我们需要从序列末尾开始处理项目时：

```cs
// first item from the end is length - 1
Assert.IsTrue(countries[length - 1] == "England");
var englandIndex = new Index(1, true);
Assert.IsTrue(countries[length - 1] == countries[englandIndex]);
```

新的`^`运算符为我们提供了一种简洁而有效的方法来获取最后一个项目：

```cs
Assert.IsTrue(countries[¹] == countries[englandIndex]);
```

重要的是要注意，从开始计数时零是第一个索引，但从末尾计数时，它指向总长度之外的一个项目。这意味着`[⁰]`表达式将始终抛出`IndexOutOfRangeException`：

```cs
Assert.ThrowsException<IndexOutOfRangeException>(() => countries[⁰]);
```

在涉及范围时，新语法的价值更加明显，因为它是一种全新的概念，在语言或基类库中以前从未存在过。新的`..`运算符界定了两个用于标识范围的索引。运算符左侧和右侧的界定符也可以省略，无论何时都应该跳过边界处的项目。

以下示例展示了指定数组中所有项目的三种方式：

```cs
var countries = new[] { "Italy", "Romania", "Switzerland", "Germany", "France", "England" };
var expected = countries.ToArray();
var all1 = countries[..];
var all2 = countries[0..⁰];
var allRange = new Range(0, new Index(0, true));
var all3 = countries[allRange];
Assert.IsTrue(expected.SequenceEqual(all1));
Assert.IsTrue(expected.SequenceEqual(all2));
Assert.IsTrue(expected.SequenceEqual(all3));
```

`expected`变量只是获得了国家数组的克隆，方便的`SequenceEqual` Linq 扩展方法在两个序列中的项目相同且排序相同时返回 true。前面的示例并不是很有用，但突出了边界的语义：*左*边界始终是*包含*的，而*右*边界始终是*排除*的。

以下示例更加现实，并展示了指定范围的三种不同方式，只跳过序列的第一个项目：

```cs
var countries = new[] { "Italy", "Romania", "Switzerland", "Germany", "France", "England" };
var expected = new[] { "Romania", "Switzerland", "Germany", "France", "England" };
var skipFirst1 = countries[1..];
var skipFirst2 = countries[1..⁰];
var skipFirstRange = new Range(1, new Index(0, true));
var skipFirst3 = countries[skipFirstRange];
Assert.IsTrue(expected.SequenceEqual(skipFirst1));
Assert.IsTrue(expected.SequenceEqual(skipFirst2));
Assert.IsTrue(expected.SequenceEqual(skipFirst3));
```

同样，以下示例展示了如何跳过序列中的最后一个项目：

```cs
var countries = new[] { "Italy", "Romania", "Switzerland", "Germany", "France", "England" };
var expected = new[] { "Italy", "Romania", "Switzerland", "Germany", "France" };
var skipLast1 = countries[..¹];
var skipLast2 = countries[0..¹];
var skipLastRange = new Range(0, new Index(1, true));
var skipLast3 = countries[skipLastRange];
Assert.IsTrue(expected.SequenceEqual(skipLast1));
Assert.IsTrue(expected.SequenceEqual(skipLast2));
Assert.IsTrue(expected.SequenceEqual(skipLast3));
```

将所有内容放在一起很简单，以下示例展示了如何跳过序列的第一个和最后一个元素：

```cs
var countries = new[] { "Italy", "Romania", "Switzerland", "Germany", "France", "England" };
var expected = new[] { "Romania", "Switzerland", "Germany", "France" };
var skipFirstAndLast1 = countries[1..¹];
var skipFirstAndLastRange = new Range(1, new Index(1, true));
var skipFirstAndLast2 = countries[skipFirstAndLastRange];
Assert.IsTrue(expected.SequenceEqual(skipFirstAndLast1));
Assert.IsTrue(expected.SequenceEqual(skipFirstAndLast2));
```

指定起始和结束索引的范围语法可以从开始或末尾开始计数。在以下示例中，切片的数组将只返回第二个和第三个元素，都是从开始计数的：

```cs
var countries = new[] { "Italy", "Romania", "Switzerland", "Germany", "France", "England" };
var expected = new[] { "Romania", "Switzerland" };
var skipSecondAndThird1 = countries[1..3];
var skipSecondAndThirdRange = new Range(1, 3);
var skipSecondAndThird2 = countries[skipSecondAndThirdRange];
Assert.IsTrue(expected.SequenceEqual(skipSecondAndThird1));
Assert.IsTrue(expected.SequenceEqual(skipSecondAndThird2));
```

当从末尾计数时，同样有效，这是以下示例的目标：

```cs
var countries = new[] { "Italy", "Romania", "Switzerland", "Germany", "France", "England" };
var expected = new[] { "Germany", "France" };
var fromEnd1 = countries[³..¹];
var fromEndRange = new Range(new Index(3, true), new Index(1, true));
var fromEnd2 = countries[fromEndRange];
Assert.IsTrue(expected.SequenceEqual(fromEnd1));
Assert.IsTrue(expected.SequenceEqual(fromEnd2));
```

这种语法非常简单，但您可能已经注意到，我们只使用了数组，以及字符串，它们在 C#中被视为特殊。事实上，如果我们尝试在`List<T>`中使用相同的语法，它将无法工作，因为没有成员知道`Index`和`Range`是什么：

```cs
var countries = new MyList<string>(new[] { "Italy", "Romania", "Switzerland", "Germany", "France", "England" });
var expected = new[] { "Romania", "Switzerland", "Germany", "France" };
MyList<string> sliced = countries[1..¹];
Assert.IsTrue(expected.SequenceEqual(sliced));
```

现在的问题是，我们如何使以下测试通过？有三种不同的方法可以使其编译并工作。第一种方法很直接，就是提供一个接受`System.Range`作为参数的索引器：

```cs
public class MyList<T> : List<T>
{
  public MyList() { }
  public MyList(IEnumerable<T> items) : base(items) { }
  public MyList<T> this[Range range]
  {
    get
    {
      (var from, var count) = range.GetOffsetAndLength(this.Count);
      return new MyList<T>(this.GetRange(from, count));
    }
  }
}
```

`List<T>`基类提供了一个接受整数的索引器，而`MyList<T>`添加了一个接受`Range`类型的重载，它在 C# 8 中作为`..`语法的别名使用。在新的索引器中，我们使用`Range.GetOffsetAndLength`，这是一个非常方便的方法，它返回一个元组，其中包含切片的初始索引和长度。最后，`List<T>.GetRange`基本方法提供了用于创建新的`MyList<T>`集合的切片序列。

另一个使先前的测试通过的可能解决方案是利用特殊的`Slice`方法，C# 8 编译器会通过模式搜索。在我们之前编写的索引器不存在的情况下，如果编译器找到一个名为`Slice`的方法，该方法接受两个整数，它会将范围语法重新映射为对`Slice`方法的调用。因此，以下代码更整洁，更易于阅读：

```cs
public class MyList<T> : List<T>
{
    public MyList() { }
    public MyList(IEnumerable<T> items) : base(items) { }
    public MyList<T> Slice(int offset, int count)
    {
        return new MyList<T>(this.GetRange(offset, count));
    }
}
```

请注意，任何使用范围语法的调用，如`countries[1..¹]`，都将调用`Slice`方法。

这个解决方案很好，但无法解决流行的`List<T>`类的问题，这个类几乎可以在代码的任何地方找到，特别是因为 Linq 扩展方法`ToList()`返回一个`IList<T>`。编写一个`Slice`扩展方法是行不通的，因为编译器会在实例方法中寻找`Slice`，而扩展方法是静态的。

解决方案是编写一个接受`Range`的扩展方法，如下例所示。这次，`countries`引用是任何继承`ICollection<T>`并支持使用`countries.Slice(1..¹)`的漂亮语法进行切片的集合：

```cs
public static class CollectionExtensions
{
    public static IEnumerable<T> Slice<T>(this ICollection<T> items, Range range)
    {
        (var offset, var count) = range.GetOffsetAndLength(items.Count);
        return items.Skip(offset).Take(count);
    }
}
```

在所有先前的例子中，我们都是使用它们的构造函数显式创建了`Index`和`Range`，但我建议花一些时间探索`Index`和`Range`类提供的便捷静态工厂，比如`Range.All()`或`Index.FromEnd()`。

范围和索引提供了强大且表达力强的运算符和类型，以简化序列中单个或多个项目的选择。其主要目的是使代码更易读，减少错误，而不影响性能。

关于范围的最重要建议是，范围的边界只在范围的左侧是包含的。

# 模式匹配

模式匹配是在 C# 7 中引入的，但语言规范的第 8 版通过平滑语法和更紧凑可读的方式扩大了其使用范围。本章将避免重复之前版本中已经看到的功能，只专注于新概念。

流行的`switch`语句在 C#中已经发展成为一个具有非常流畅语法的*表达式*。例如，假设您正在使用`Console.ReadKey`方法读取应用程序中的控制台键，以获取与`R`、`G`和`B`字符匹配的颜色：

```cs
public Color ToColor(ConsoleKey key) 
{
    return key switch
    {
        ConsoleKey.R => Color.Red,
        ConsoleKey.G => Color.Green,
        ConsoleKey.B => Color.Blue,
        _ => throw new ArgumentException($"Invalid {nameof(key)}"),
    };
}
```

或者，如果您更喜欢更紧凑的版本，我们可以这样写：

```cs
public Color ToColor(ConsoleKey key) => key switch
    {
        ConsoleKey.R => Color.Red,
        ConsoleKey.G => Color.Green,
        ConsoleKey.B => Color.Blue,
        _ => throw new ArgumentException($"Invalid {nameof(key)}"),
    };
```

`switch`表达式在语义上与 C# 7 模式匹配的先前创新没有改变；相反，它变得更简单，更紧凑，有一些重要的事情需要强调：

+   作为表达式，`switch`语句必须返回一个值（在我们的示例中是`Color`枚举）。

+   弃用字符`(_)`取代了经典`switch`语句中的`default`关键字。

+   将键映射到颜色的子表达式按顺序进行评估，第一个匹配就会退出。

当使用`switch`表达式匹配类型时，事情可能变得更加有趣，如下例所示：

```cs
string GetString(object o) => o switch
   {
     string s   => $"string '{s}'",
     int i      => $"integer {i:d4}",
     double d   => $"double {d:n}",
     Derived d  => $"Derived: {d.Other}",
     Base b     => $"Base: {b.Name}",
     null       =>  "null",
     _          => $"Fallback: {o}",
   };
```

这个方法接受一个未知对象作为输入，并返回一个根据其运行时类型格式化不同的字符串，该类型必须与确切的类型匹配。例如，`GetString((Int16)1)`将不匹配，也不会返回字符串`Fallback: 1`。另一个失败的匹配是`GetString(10.6m)`，因为字面量是十进制，返回的字符串将是`Fallback: 10.6`。

在 C# 7 之前，对值类型或引用类型进行类型识别测试非常麻烦，因为它需要第二步，要么将值类型转换为所需类型，要么对引用类型进行空检查条件操作。多亏了 C# 7，我们学会了使用`is`模式匹配，当检查单个类型时非常完美。

使用新的 C# 8 语法，生成的代码更加简洁，更不容易出错，具有许多优点：

+   在每种情况下都不必担心空引用，这对于方法成为**即时编译器**（**JIT**）内联的更好候选者有积极的影响，从而提高性能。

+   评估遵循顺序，这在测试类型层次结构时非常有用。在我们的例子中，评估`Derived`类在`Base`之前是至关重要的，否则`switch`表达式将始终匹配`Base`。

+   显式捕获*null case*中的空值避免了任何条件表达式。

`switch`表达式非常强大，但模式匹配的改进还没有结束。

## 递归模式匹配

模式匹配已经扩展，允许深入到对象属性和元组中。这一改进的基础语法包括在模式后的花括号中指定表达式的能力：

```cs
var weekDays = Enum.GetNames(typeof(DayOfWeek));
var expected = new[] { "Sunday", "Monday", "Friday", };
var six = weekDays
    .Where(w => w is string { Length: 6 })
    .ToArray();
Assert.IsTrue(six.SequenceEqual(expected));
```

花括号内的表达式只能指定属性，并且必须使用常量文字。这使我们能够匹配类型，并同时评估其属性，可能会在子表达式中重复。

当我们需要评估以图形结构化的对象时，真正的力量就会发挥作用，就像在`Order`类的两个`Customer`属性中一样。

```cs
public class Order
{
    public Guid Id { get; set; }
    public bool IsMadeOnWeb { get; set; }
    public Customer Customer { get; set; }
    public decimal Quantity { get; set; }
}
public class Customer
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Country { get; set; }
}
```

现在，假设我们正在开发一个电子商务应用程序，其中折扣取决于订单属性：

```cs
public decimal GetDiscount(Order order) => order switch
{
    var o when o.Quantity > 100 => 7.5m,
    { IsMadeOnWeb: true } => 5.0m,
    { Customer: { Country: "Italy" } } => 2.0m,
    _ => 0,
};
```

在这里，第一个子表达式重新分配了对`o`变量的引用，然后通过`when`子句评估了`Quantity`属性。如果满足`o.Quantity > 100`，则返回 7.5%的折扣。

在第二种情况下，当`Order.IsMadeOnWeb`为真时，返回 5%的折扣。第三种情况评估了通过导航`Order.Customer.Country`获得的属性，仅因为订单来自意大利，返回 2%的折扣。最后，丢弃字符表示回退到零折扣。

属性的语法很棒，但是当涉及元组时，情况会变得更加复杂，因为您可能希望匹配单个元组项，以及多个元组项，它们的位置也是至关重要的。

例如，考虑一个简单的`Point`结构，其中有两个整数属性`X`和`Y`：

```cs
struct Point
{
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
    public int X { get; set; }
    public int Y { get; set; }
}
```

我们如何编写一个方法，返回点是否位于水平轴或垂直轴上？如果`X`或`Y`为零，则满足条件；因此，一个可能的方法是这样做：

```cs
bool IsOnAxis(Point p) => (p.X, p.Y) switch
{
    (0, _) => true,
    (_, 0) => true,
    (_, _) => false,
};
```

传统上，我们会使用一个`if`和一个`or`运算符来编写这个方法，但是参数越多，代码就变得越难读。前面例子的一个有趣的地方是，我们动态构建了一个元组，并在`switch`表达式中对其进行了评估，通过它们的位置匹配参数，并丢弃（使用`_`字符）与评估无关的参数。

当编写`Point`结构中的特殊`Deconstruct`方法时，情况变得更加有趣，因为它简化了元组的创建：

```cs
public struct Point
{
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
    public int X { get; set; }
    public int Y { get; set; }
    public void Deconstruct(out int x, out int y)
    {
        x = X;
        y = Y;
    }
}
public bool IsOnAnyAxis(Point p) => p switch
{
    (0, _) => true,
    (_, 0) => true,
    _ => false,
};
```

在`switch`表达式中使用元组时，通过使用`when`子句评估其值，可以获得更多的功能。

在下面的例子中，我们使用`when`子句来识别对角线位置，除了轴。为此，我们定义了`SpecialPosition`枚举器，并使用`switch`表达式以及`when`子句来匹配对角线：

```cs
enum SpecialPosition
{
    None,
    Origin,
    XAxis,
    YAxis,
    MainDiagonal,
    AntiDiagonal,
}
SpecialPosition GetSpecialPosition(Point p) => p switch
{
    (0, 0) => SpecialPosition.Origin,
    (0, _) => SpecialPosition.YAxis,
    (_, 0) => SpecialPosition.XAxis,
    var (x, y) when x ==  y => SpecialPosition.MainDiagonal,
    var (x, y) when x == -y => SpecialPosition.AntiDiagonal,
    _ => SpecialPosition.None,
};
```

模式匹配在过去两个语言版本中获得了很大的力量，现在允许开发人员专注于代码的重要部分，而不会被以前语言规则所需的样板代码分散注意力。

`switch`表达式特别适用于那些结果可以从多个选择中得出的表达式，如果评估需要深入到对象图或评估元组。强大的丢弃字符允许部分评估，避免了通常复杂且容易出错的代码。

# using 声明

`using`声明是一个非常方便的语法，相当于`try/finally`块，并确定性地调用`Dispose`方法。这个声明可以用于所有实现`IDisposable`接口的对象：

```cs
class DisposableClass : IDisposable
{
    public void Dispose() => Console.WriteLine("Dispose!");
}
```

我们已经知道，`using`声明在遇到其闭合大括号时会确定性地调用`Dispose`方法：

```cs
void SomeMethod()
{
    using (var x = new DisposableClass())
    {
        //...
    }	// Dispose is called
}
```

每当需要在同一作用域中使用多个可释放对象时，嵌套的`using`声明会导致令人讨厌的三角形代码对齐：

```cs
using (var x = new Disposable1())
{
    using (var y = new Disposable2())
    {
        using (var z = new Disposable3())
        {
            //...
        }
    }
}
```

如果`Dispose`方法在当前块（闭合大括号）的末尾被调用，无论该块是一个语句（如`for`/`if`/…）还是当前方法，这种烦恼最终可以被消除。

C# 8 中的新语法允许我们完全删除`using`声明中的大括号，将前面的示例转换为以下形式：

```cs
void SomeMethod()
{
    using (var x = new Disposable1());
    using (var y = new Disposable2());
    using (var z = new Disposable3());
    //...
} // Dispose methods are called
```

当前块的第一个闭合大括号将自动触发三个`Dispose`方法，按照声明的相反顺序。但关于`Dispose`还有更多内容要讨论；事实上，这种紧凑的语法也适用于`async using`声明，这将在下一节中介绍。

# 异步 Dispose

在.NET 中引入 Tasks 之后，大多数管理 I/O 操作的库逐渐转向异步行为。例如，`System.Net.Websocket`类成员采用基于任务的编程策略，提供更好的开发人员体验和更高效的行为。

每当开发人员需要编写一个 C#客户端来访问基于 WebSocket 协议的某些服务时，他们通常会编写一个包装类，公开专门的*send*方法，并实现释放模式以调用`Websocket.CloseAsync`方法。我们也知道任何异步方法都应该返回一个`Task`，但是 Dispose 方法在`Task`时代之前就已经定义为 void，因此不太适合`Task`链中。

Websocket 示例非常现实，因为我曾经遇到过这个确切的问题，即在 Dispose 内部阻塞当前线程等待 CloseAsync 完成会导致死锁。

从 C# 8 和.NET Core 3.0 开始，我们现在有两个重要的工具：

+   在.NET Core 3 中定义的`IAsyncDisposable`接口，返回轻量级的`ValueTask`类型

+   利用新的`AsyncDisposable`接口的`await using`构造

让我们看看如何在代码中使用它们：

```cs
public class AsyncDisposableClass : IAsyncDisposable
{
    public ValueTask DisposeAsync()
    {
        Console.WriteLine("Dispose called");
        return new ValueTask();
    }
}
private async Task SomeMethodAsync()
{
    await using (var x = new AsyncDisposableClass())
    {
        // ...
    }
}
```

值得记住的是，`await using`声明受益于简洁的单行语法，正如我们之前讨论的那样：

```cs
private async Task SomeMethodAsync()
{
    await using (var x = new AsyncDisposableClass());
}
```

如果您是一个公开可释放类型的库作者，您可以实现这两种接口中的任何一种，甚至同时实现`IDisposable`和`IAsyncDisposable`接口。

# 结构体和 ref 结构中的可释放模式

随着时间的推移，C#引入了一些*基于模式*的构造来解决由于规则无法在每种情况下应用而导致的问题。例如，`foreach`语句不需要对象实现`IEnumerable<>`接口，而只需依赖于`GetEnumerator`方法的存在，同样，`GetEnumerator`返回的对象不需要实现`IEnumerator`，而只需公开所需的成员即可。

这一变化是由最近引入的`ref 结构`驱动的，它对减少垃圾收集器的压力很重要，因为它们保证只在堆栈上存在，但不允许实现接口。

基于模式的方法现在已经在特定条件下扩展到了`Dispose`和`DisposeAsync`方法，我们现在将讨论这些条件。

从 C# 8 开始，开发人员可以定义`Dispose`或`DisposeAsync`而无需实现`IDisposable`或`IAsyncDisposable`。通过模式实现`Dispose`方法已经被*限制*为`ref struct`类型，因为将其扩展到任何其他类型最终可能会导致已经定义了`Dispose`方法但未在继承列表中声明`IDisposable`的类型发生破坏性变化。

以下定义是`Dispose`和`DisposeAsync`方法的有效实现：

```cs
ref struct MyRefStruct
{
    public void Dispose() => Debug.WriteLine("Dispose");
    public ValueTask DisposeAsync()
    {
        Debug.WriteLine("DisposeAsync");
        return default(ValueTask);
    }
}
```

`Dispose`方法可以像往常一样使用：

```cs
public void TestMethod1()
{
    using var s1 = new MyRefStruct();
}
```

但这种声明是不允许的，因为我们不能在异步方法中使用`ref`：

```cs
public async Task TestMethod2()
{
    //await using var s2 = new MyRefStruct(); // Error!
}
```

解决方法是扩展`await using`声明，使用完整的`try`/`finally`：

```cs
public Task TestMethod3()
{
    var s2 = new MyRefStruct();
    Task result;
    try { /*...*/ }
    finally
    {
        result = s2.DisposeAsync().AsTask();
    }
    return result;
}
```

这段代码肯定不好阅读，但我们应该考虑，在一个生命周期仅限于堆栈的类型中声明`Dispose`的异步版本可能不是一个好主意。

虽然通过模式实现的`Dispose`已经被预防性地限制为`ref structs`，但通过模式实现的`DisposeAsync`没有限制，因此在老式类中声明`DisposeAsync`并在`await using`语句中使用它是完全合法的。

# 异步流

异步流是任务故事的最后一块缺失的部分，这个故事始于几年前`Task`类、`async`和`await`首次引入。一个未解决的用例示例是在从互联网下载数据时处理数据块。基本点在于我们不想等待整个数据流，而是一次获取一个数据块，处理它，然后等待下一个。这样处理可以在其他数据仍在下载时进行，未使用的线程时间也可以用来为其他用户提供服务，增加应用程序的总可伸缩性。

在深入研究新的 C#特性之前，让我们快速回顾一下在同步世界中如何创建可枚举对象。以下示例展示了一个可在`foreach`语句中使用的可枚举序列；您可能会注意到，枚举类型是整数，而不是假设的从互联网下载的数据块组成的字节数组，但这并不重要。

最简单的实现利用了 C#迭代器，通过`yield`关键字实现：

```cs
static IEnumerable<int> SyncIterator()
{
    foreach (var item in Enumerable.Range(0, 10))
    {
        Thread.Sleep(500);
        yield return item;
    }
}
```

它的主要使用者当然是`foreach`语句：

```cs
foreach (var item in SyncIterator())
{
    // ...
}
```

在底层，编译器生成代码，暴露一个`IEnumerable<T>`，其职责是提供枚举器，一个由`Current`、`Reset`和`MoveNext`成员组成的类来展开序列。这段代码的相关部分是`MoveNext`方法中的`Thread.Sleep`，模拟了一个缓慢的迭代。

以下代码是等效的，但手动实现了`IEnumerable`和`IEnumerator`接口：

```cs
public class SyncSequence : IEnumerable<int>
{
    private int[] _data = Enumerable.Range(0, 10).ToArray();
    public IEnumerator<int> GetEnumerator() => new SyncSequenceEnumerator<int>(_data);
    IEnumerator IEnumerable.GetEnumerator() => new SyncSequenceEnumerator<int>(_data);
    private class SyncSequenceEnumerator<T> : IEnumerator<T>, IEnumerator, IDisposable
    {
        private T[] _sequence;
        private int _index;
        public SyncSequenceEnumerator(T[] sequence)
        {
            _sequence = sequence;
            _index = -1;
        }
        object IEnumerator.Current => _sequence[_index];
        public T Current => _sequence[_index];
        public void Dispose() { }
        public void Reset() => _index = -1;
        public bool MoveNext()
        {
            Thread.Sleep(500);
            _index++;
            if (_sequence.Length <= _index) return false;
            return true;
        }
    }
}
```

再一次，`foreach`语句可以轻松地消耗序列，共享由`Thread.Sleep`引起的阻塞线程的问题，而在现实生活中，这将是操作系统网络堆栈中正在进行的 I/O 操作：

```cs
foreach (var item in new SyncSequence())
{
    // ...
}
```

为了解决这个问题，C# 8 引入了非常方便的`await foreach`，用于迭代异步枚举，这又需要两个新接口：`IAsyncEnumerable<T>`和`IAsyncEnumerator<T>`。

新的异步流的最简单的生产者和消费者与以前的非常相似：

```cs
async IAsyncEnumerable<int> AsyncIterator()
{
    foreach (var item in Enumerable.Range(0, 10))
    {
        await Task.Delay(500);
        yield return item;
    }
}
await foreach (var item in AsyncIterator())
{
    // ...
}
```

如果我们需要手动实现这两个接口，那么与同步实现并没有太大不同，不出所料，我们需要实现`MoveNext`的异步版本`MoveNextAsync`：

```cs
public class AsyncSequence : IAsyncEnumerable<int>
{
    private int[] _data = Enumerable.Range(0, 10).ToArray();
    public IAsyncEnumerator<int> GetAsyncEnumerator(CancellationToken cancellationToken = default)
    {
        return new MyAsyncEnumerator<int>(_data);
    }
    private class MyAsyncEnumerator<T> : IAsyncEnumerator<T>
    {
        private T[] _sequence;
        private int _index;
        public MyAsyncEnumerator(T[] sequence)
        {
            _sequence = sequence;
            _index = -1;
        }
        public T Current => _sequence[_index];
        public ValueTask DisposeAsync() => default(ValueTask);
        public async ValueTask<bool> MoveNextAsync()
        {
            await Task.Delay(500);
            _index++;
            if (_sequence.Length <= _index) return false;
            return true;
        }
    }
}
```

与`IEnumerator<T>`从`IDisposable<T>`派生一样，`IAsyncEnumerator<T>`接口从我们已经讨论过的`IAsyncDisposable<T>`派生。

`MoveNextAsync`和`Current`是`IAsyncEnumerator<T>`接口需要的唯一其他成员，其方法返回*轻量级*的`ValueTask`类型，这在`DisposeAsync`中已经见过。

注意

在撰写本文时，基类库中实现`IAsyncEnumerable<T>`的唯一类是`System.Threading.Channel`，因此为了充分利用异步流的功能，您应该采用外部库或自己实现这两个接口，这非常简单。

消费新的异步序列的代码在结构上是相同的：

```cs
await foreach (var item in new AsyncSequence())
{
    // ...
} 
```

为了完整起见，消费代码等同于以下内容：

```cs
var sequence = new AsyncSequence();
IAsyncEnumerator<int> enumerator = sequence.GetAsyncEnumerator();
try
{
    while (await enumerator.MoveNextAsync())
    {
        // some code using enumerator.Current
    }
}
finally { await enumerator.DisposeAsync(); }
```

静态的`TaskAsyncEnumerableExtensions`类包含一些扩展方法，允许配置`IAsyncEnumerable`对象，就像您从任何其他`Task`对象中期望的那样。

第一个扩展方法是`ConfigureAwait`，我们已经在*第十二章*中进行了讨论，*多线程和异步编程*。另一个是`WithCancellation`，它接受一个`CancellationToken`值，可以用于取消正在进行的任务。

异步流非常强大，因为它简化了开发人员的代码，同时使其更加强大。在生产者方面，实现所需的接口（`IAsyncEnumerable`和`IAsyncEnumerator`）非常简单，而在消费者方面，由于新的`async foreach`，可以轻松地异步枚举序列。

一个缺点是当前的库生态系统与新接口不兼容。因此，社区已经编写了一组新的 Linq 风格的扩展方法，提供了与基类库中内置方法相同的*外观和感觉*。

对于每种用例使用合适的工具也很重要。换句话说，并不是因为语言已经扩展了就需要将一切都转换成异步的。这只是每个开发人员在有意义时可以使用的重要工具。

# 只读结构成员

在 C# 7 引入`readonly`结构后，现在可以单独在其成员上指定`readonly`修饰符。这个特性是为了所有那些结构类型不能完全标记为只读的情况而添加的，但是当一个或多个成员可以保证不修改实例状态时。

我喜欢这个功能的主要原因是因为明确表达意图在维护和可用性方面是最佳实践。

从性能的角度来看也很重要，因为`readonly`结构为编译器提供了一种*提示*，可以应用更好的优化。修饰符可以应用于字段、属性和方法，以确保它不会改变结构实例，但不会对引用的对象提供任何保证。

处理属性时，修饰符可以应用于属性或者仅应用于其中一个访问器：

```cs
public readonly int Num0
{
    get => _i;
    set { } // not useful but valid
}
public readonly int Num1
{
    get => _i;
    //set => _i = value; // not valid
}
public int Num2
{
    readonly get => _i;
    set => _i = value; // ok
}
public int Num3
{
    get => ++_i;     // strongly discouraged but it works
    readonly set { } // does not make sense but it works
}
```

例如，让我们定义一个`Vector 结构`，公开两个返回向量长度的方法，其中只有一个标记为`readonly`：

```cs
public struct Vector
{
    public float x;
    public float y;
    private readonly float SquaredRo => (x * x) + (y * y);
    public readonly float GetLengthRo() => MathF.Sqrt(SquaredRo);
    public float GetLength() => MathF.Sqrt(SquaredRo);
}
```

由于值类型（如`Vector`）在作为参数传递时会被复制，一个常见的解决方案是应用`in`修饰符（意味着`readonly` `ref`），就像以下示例中一样：

```cs
public static float SomeMethod(in Vector vector)
{
    // a local copy is done because GetLength is not readonly
    return vector.GetLength();
}
```

不幸的是，`in`修饰符不能保证引用地址的其他数据的不可变性。因此，一旦编译器看到调用`GetLength`方法，它就必须假设可能会对向量实例进行潜在更改，导致`Vector`的防御性隐藏本地副本，而不管它是通过引用传递的。

如果我们用只读的`GetLengthRo`方法替换对`GetLength`的调用，编译器会理解在修改`Vector`内容时没有风险，并且可以避免生成本地副本，从而提供更好的应用性能：

```cs
public static float ReadonlyBehavior(in Vector vector)
{
    // no local copy is done because GetLengthRo is readonly
    return vector.GetLengthRo();
}
```

值得一提的是，编译器足够聪明，可以提供一些自动优化。例如，自动生成的属性 getter 已经标记为只读，但请记住对所有其他不改变实例状态的成员应用`readonly`修饰符，为编译器提供重要提示，并获得尽可能好的优化。

注意

版本之后，编译器改进了其检测潜在副作用的能力，例如局部副本。您可以使用`ildasm`或`ILSpy`等反编译器自行验证生成的 IL 代码，但请注意，这些优化可能随时间而变化。

如果将方法标记为只读，即使它修改了实例的状态，编译器也会生成错误或警告，具体取决于情况：

+   如果`readonly`方法尝试修改结构的任何字段，编译器将报告`CS1604`错误。

+   每当代码访问不是只读属性 getter 时，编译器都会生成一个`CS8656`警告，以提醒生成所需的代码来创建结构的防御性隐藏本地副本，如消息描述中所述。

在 CS8656 警告消息中，编译器建议生成`'this'`的副本以避免改变当前实例：

```cs
"Call to a non readonly member '...' from a 'readonly' member results in an implicit copy of 'this'".
```

关于编译器识别不良情况的能力有一个重要的副作用。它无法检测任何试图修改对引用对象的更改，如下面的代码所示：

```cs
struct Undetected
{
    private IDictionary<string, object> _bag;
    public Undetected(IDictionary<string, object> bag)
    {
        _bag = bag;
    }
    public readonly string Description
    {
        get => (string)_bag["Description"];
        set => _bag["Description"] = value;
    }
}
```

虽然我们似乎在不对不修改值类型状态的结构成员应用`readonly`修饰符中看不到任何缺点，但一定要小心，因为这可能对热点路径的性能产生很大影响。

# 空值合并赋值

在 C# 8 中，空值合并运算符`??`已扩展以支持赋值。空值合并运算符的一个常见用法涉及方法开头的参数检查，就像以下示例中所示：

```cs
class Person
{
    public Person(string firstName, string lastName, int age)
    {
        this.FirstName = firstName ?? throw new ArgumentNullException(nameof(firstName));
        this.LastName = lastName ?? throw new ArgumentNullException(nameof(lastName));
        this.Age = age;
    }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; }
}
```

新的赋值允许我们在引用为 null 时重新分配引用，就像以下示例所示：

```cs
void Accumulate(ref List<string> list, params string[] words)
{
    list ??= new List<string>();
    list.AddRange(words);
}
```

参数列表最初可以为 null，在这种情况下，它将被重新分配给一个新实例，但在接下来的时间里，赋值将不再发生：

```cs
List<string> x = null;
Accumulate(ref x, "one", "two");
Accumulate(ref x, "three");
Assert.IsTrue(x.Count == 3);
```

空值合并赋值看起来并不是很重要，但它执行最右边表达式的能力是一个你不应该低估的重要价值。

# 静态本地函数

引入了本地函数，通过将某个代码片段的可见性限制为单个方法，使代码更易读：

```cs
void PrintName(Person person)
{
    var p = person ?? throw new ArgumentNullException(nameof(person));
    Console.WriteLine(Obfuscated());
    string Obfuscated()
    {
        if (p.Age < 18) return $"{p.FirstName[0]}. {p.LastName[0]}.";
        return $"{p.FirstName} {p.LastName}"; 
    }
}
```

在这个例子中，`Obfuscated`方法只能被`PrintName`使用，并且具有忽略任何参数检查的优势，因为`p`捕获的参数在使用它的上下文中不允许其值为 null。这可以在复杂的场景中提供性能优势，但它捕获局部变量（包括`this`）的能力可能会令人困惑。

在 C# 8 中，现在可以通过将本地函数标记为静态来避免任何捕获：

```cs
private void PrintName(Person person)
{
    var p = person ?? throw new ArgumentNullException(nameof(person));
    Console.WriteLine(Obfuscated(p));
    static string Obfuscated(Person p)
    {
        if (p.Age < 18) return $"{p.FirstName[0]}. {p.LastName[0]}.";
        return $"{p.FirstName} {p.LastName}";
    }
}
```

这个方法的新版本强化了它自我描述的能力，同时仍然具有忽略由于已知上下文而导致的任何参数检查的优势。值得注意的是，捕获通常在性能方面不是问题，但可能严重影响可读性，因为 C#默认允许自动捕获，与 C++ lambda 等其他语言形成对比。

# 更好的插值原始字符串

我们已经学会了字符串文字支持一些*变体*以避免转义字符：

```cs
string s1 = "c:\\temp";
string s2 = @"c:\temp";
Assert.AreEqual(s1, s2);
```

它们也可以用于改进格式，感谢插值：

```cs
var s3 = $"The path for {folder} is c:\\{folder}";
```

自从插值字符串被引入以来，我们一直能够混合两种格式化样式：

```cs
var s4 = $@"The path for {folder} is c:\{folder}";
Assert.AreEqual(s3, s4);
```

但在 C# 8 之前，不可能颠倒`$`和`@`字符：

```cs
var s5 = @$"The path for {folder} is c:\{folder}";
Assert.AreEqual(s3, s5);
```

有了这个小改进，你就不必再担心前缀的顺序了。

# 在嵌套表达式中使用 stackalloc

在 C# 7 中，我们开始使用`Span<T>`、`ReadOnlySpan<T>`和`Memory<T>`，因为它们是保证在堆栈上分配的`ref struct`实例，因此不会影响垃圾收集器。由于`Span`，也可以避免声明直接分配给`Span`或`ReadOnlySpan`的`stackalloc`语句作为不安全的：

```cs
Span<int> nums = stackalloc int[10];
```

从 C# 8 开始，编译器将`stackalloc`的使用扩展到任何期望`Span`或`ReadOnlySpan`的表达式。在下面的例子中，测试从`input`字符串中修剪了三个特殊字符，得到了`expected`变量中指定的字符串：

```cs
string input = " this string can be trimmed \r\n";
var expected = "this string can be trimmed";
ReadOnlySpan<char> trimmedSpan = input.AsSpan()
    .Trim(stackalloc[] { ' ', '\r', '\n' });
string result = trimmedSpan.ToString();
Assert.AreEqual(expected, result);
```

前面例子中执行的操作如下：

+   `AsSpan`扩展方法将字符串转换为`ReadOnlySpan<char>`。

+   `Trim`扩展方法将`ReadOnlySpan<char>`的边界缩小到`stackalloc`数组指定的字符。这个`Trim`方法不需要任何分配。

+   最后，调用`ToString`方法从`ReadOnlySpan<char>`创建一个新的字符串。

这段代码的优势在于，除了用于验证测试的新的`int[]`表达式和用于创建结果的`ToString`方法之外，不会执行其他堆分配。

# 未管理构造类型

在深入研究这个新的 C#特性之前，有必要通过分析语言规范中引用的“未管理”和“构造类型”的定义来理解这个主题：

+   如果类型是泛型的，并且类型参数已经定义，则称为“构造”类型。例如，`List<string>`是一个构造类型，而`List<T>`则不是。

+   当它可以在不安全的上下文中使用时，类型被称为“未管理”。这对许多内置的基本类型都是正确的。官方文档包括这些类型的列表：`sbyte`、`byte`、`short`、`ushort`、`int`、`uint`、`long`、`ulong`、`char`、`float`、`double`、`decimal`、`bool`、`enums`、`pointers`和`struct`。

在 C# 8 之前无法声明的未管理构造类型的一个例子如下：

```cs
struct Header<T>
{
    T Word1;
    T Word2;
    T Word3;
}
```

允许泛型结构体为未管理的两个主要优势如下：

+   它们可以使用`stackalloc`在堆栈上分配。

+   我们可以使用这些类型与指针和不安全的代码一起与本机代码进行互操作。当处理本机块时，这是有用的，其字段可以是 32 位或 64 位：

```cs
Span<Header<int>> records1 = stackalloc Header<int>[10];
Span<Header<long>> records2 = stackalloc Header<long>[10];
```

有了这个特性，语言规范朝着简化本机互操作性的方向发展，而不会导致以往需要使用 C 或 C++语言的性能损失。

# 总结

毫无疑问，新的 C# 8 功能标志着代码健壮性和清晰度方面的重要里程碑。语言变得越来越复杂和难以阅读并不罕见，但 C#引入了诸如模式匹配和范围等功能，使任何开发人员都能用更简洁和明确的代码表达其意图。

尽管有争议，但默认接口成员将 Traits 范式引入了.NET 世界，并解决了接口版本化等多年来困扰开发人员的问题。

我们了解了一个关键特性，即内置可空引用静态代码分析，它允许我们逐步审查代码并大大减少由于取消引用空引用而导致的错误数量。

这并不是为了提高生产力而调整语言的终点，因为我们继续通过 C#7 性能之旅，引入了异步流、只读结构成员以及对`stackalloc`和未管理构造类型的更新，所有这些都使 C#成为本机语言中的一个引人注目的竞争者，同时仍然强制执行代码安全性。

其他较小的特性，如简洁的`using`声明、异步`Dispose`、可处置模式、静态局部函数、插值字符串的修复和空值合并赋值，都非常容易记住并提供实际优势。

新的语言特性不仅仅是开发人员瑞士军刀中的额外工具，而是改进代码库的重大机会。如果我们回到过去，想想 C# 2.0 引入的泛型类型，它们大大提高了生产力和性能。后来，语言引入了 LINQ 查询、lambda 表达式和扩展方法，从而带来了更多的表现力，并开启了之前更加困难的新设计策略。整个编程语言的历史，不仅仅是 C#，都以改进满足现代开发需求为特点。如今，应用程序开发明显倾向于通过采用**持续集成/持续交付**（**CI/CD**）流水线来缩短开发周期，这带来了对代码质量和生产力的强烈要求。考虑到这个更广泛的视角，毫无疑问，跟上最新语言特性的步伐对于任何开发人员来说都是必须的。

在下一章中，我们将学习.NET Core 3 如何将语言形式转化为运行代码，无论是在 Windows 还是 Linux 上。我们将学习创建一个可以从任何.NET 运行时环境中使用的库；使用包，这是这个生态系统的真正丰富之处；最后，发布应用程序，将我们所有的工作转化为最终用户的巨大价值。

# 测试你所学到的知识

1.  你如何最小化代码中的`NullReferenceException`异常数量？

1.  使用什么语法来读取数组中的最后一个项目？

1.  在使用`switch`表达式时，什么关键字等同于使用弃置字符(_)？

1.  你如何等待一个异步调用来关闭`Dispose`方法中的文件？

1.  在下面的语句中，当分配`orders`变量时，方法调用是否在每次执行时都被调用？

```cs
var orders ??= GetOrders();
```

1.  定义一个序列为`IAsyncEnumerable`是必须的，才能用新的`async foreach`语句进行迭代吗？

# 进一步阅读

如果你想要跟踪 C#的发展，你可以在 GitHub 上查看关于语言下一个版本的提案和讨论：https://github.com/dotnet/csharplang。
