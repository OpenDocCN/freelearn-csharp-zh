# 第十一章：*第十一章*：反射和动态编程

在上一章中，我们讨论了函数式编程、lambda 表达式以及它们所支持的功能，比如**语言集成查询（LINQ）**。本章侧重于反射服务和动态编程。您将学习什么是反射，以及如何在运行时获取有关类型的信息，以及代码和资源如何存储在程序集中，以及如何在运行时动态加载它们，无论是用于反射还是代码执行。

这对于构建支持插件或附加组件形式的扩展的应用程序至关重要。我们将看到属性是什么，以及它们在反射中扮演的角色。本章中我们将讨论的另一个重要主题是动态编程和**动态语言运行时**，它使动态语言能够在**公共语言运行时（CLR）**上运行，并为静态类型语言添加动态特性。

本章我们将讨论以下主题：

+   理解反射

+   动态加载程序集

+   理解后期绑定

+   使用`dynamic`类型

+   属性

在本章结束时，您将对反射、属性及其在反射中的使用，以及程序集加载和代码执行有很好的理解。另一方面，您还将学习关于`dynamic`类型，并能够与动态语言进行交互。

# 理解反射

.NET 中的部署单元是程序集。程序集是一个文件（可以是可执行文件或动态链接库），其中包含`ildasm.exe`（`ilspy.exe`（一个开源项目）；或其他允许您查看程序集内容的工具。以下是`ildasm.exe`的屏幕截图，显示了本书源代码中提供的`chapter_11_01.dll`程序集：

![图 11.1 - chapter 11 程序集的反汇编源代码。](img/Figure_11.1_B12346.jpg)

图 11.1 - chapter_11_01 程序集的反汇编源代码

**反射**是在运行时发现类型并对其进行更改的过程。这意味着我们可以在运行时检索有关类型、其成员和属性的信息。这带来了一些重要的好处：

+   在运行时动态加载程序集（后期绑定）、检查类型和执行代码的能力使得构建可扩展应用程序变得容易。应用程序可以通过接口和基类定义功能，然后在单独的模块（插件或附加组件）中实现或扩展这些功能，并根据各种条件在运行时加载和执行它们。

+   属性，我们稍后将在本章中看到，使得以声明方式提供有关类型、方法、属性和其他内容的元信息成为可能。通过能够在运行时读取这些属性，系统可以改变它们的行为。例如，工具可以警告某个方法的使用方式与预期不同（比如过时方法的情况），或以特定方式执行它们。测试框架（我们将在最后一章中看到一些）广泛使用了这种功能。

+   它提供了执行私有或其他访问级别的类型和成员的能力，否则这些类型和成员将无法访问。这对于测试框架来说非常方便。

+   它允许在运行时修改现有类型或创建全新类型，并使用它们执行代码。

反射也有一些缺点：

+   它会产生一个可能降低性能的开销。在运行时加载、发现和执行代码会更慢，可能会阻止优化。

+   它暴露了类型的内部，因为它允许对所有类型和成员进行内省，而不考虑它们的访问级别。

.NET 反射服务允许您使用`System.Reflection`命名空间中的 API 发现与前面提到的工具中看到的相同的信息。这个过程的关键是名为`System.Type`的类型，其中包含公开所有类型元数据的成员。这是通过`System.Reflection`命名空间中的其他类型的帮助完成的，其中一些列在以下表中：

![](img/Chapter_11_Table_1_01.jpg)

`System.Type`类的一些最重要的成员列在以下表中：

![](img/Chapter_11_Table_2_01.jpg)

有几种方法可以在运行时检索`System.Type`的实例以访问类型元数据；以下是其中的一些：

+   使用`System.Object`类型的`GetType()`方法。由于这是所有值类型和引用类型的基类，您可以使用任何类型的实例调用：

```cs
var engine = new Engine();
var type = engine.GetType();
```

+   使用`System.Type`的`GetType()`静态方法。有许多重载，允许您指定名称和各种参数：

```cs
var type = Type.GetType("Engine");
```

+   使用 C#的`typeof`运算符：

```cs
var type = typeof(Engine);
```

让我们看看如何通过查看一个实际的例子来使用反射。我们将考虑以下`Engine`类型，它具有几个属性、一个构造函数和一对改变引擎状态（启动或停止）的方法：

```cs
public enum EngineStatus { Stopped, Started }
public class Engine
{
    public string Name { get; }
    public int Capacity { get; }
    public double Power { get; }
    public EngineStatus Status { get; private set; }
    public Engine(string name, int capacity, double power)
    {
        Name = name;
        Capacity = capacity;
        Power = power;
        Status = EngineStatus.Stopped;
    }
    public void Start()
    {
        Status = EngineStatus.Started;
    }
    public void Stop()
    {
        Status = EngineStatus.Stopped;
    }
}
```

我们将构建一个小程序，它将在运行时读取有关`Engine`类型的元数据，并将以下内容打印到控制台：

+   *类型*的名称

+   所有*属性*的名称以及它们的类型的名称

+   所有*声明的方法*的名称（不包括继承的方法）

+   它们的*返回类型*的名称

+   每个参数的名称和类型

以下是用于在运行时读取和打印有关`Engine`类型的元数据的程序：

```cs
static void Main(string[] args)
{
    var type = typeof(Engine);
    Console.WriteLine(type.Name);
    var properties = type.GetProperties();
    foreach(var p in properties)
    {
        Console.WriteLine($"{p.Name} ({p.PropertyType.Name})");
    }
    var methods = type.GetMethods(BindingFlags.Public |
                                  BindingFlags.Instance |
                                  BindingFlags.DeclaredOnly);
    foreach(var m in methods)
    {
        var parameters = string.Join(
            ',',
            m.GetParameters()
             .Select(p => $"{p.ParameterType.Name} {p.Name}"));
        Console.WriteLine(
          $"{m.ReturnType.Name} {m.Name} ({parameters})");
   }
}
```

在这个例子中，我们使用`typeof`运算符检索`System.Type`类型的实例，以发现`Engine`类型的元数据。为了检索属性，我们使用了没有参数的`GetProperties()`重载，它返回当前类型的所有公共属性。然而，对于方法，我们使用了`GetMethod()`方法的重载，它以一个由一个或多个`BindingFlags`值组成的位掩码作为参数。

`BindingFlags`类型是一个枚举，其中的标志控制绑定和在反射期间执行类型和方法搜索的方式。在我们的例子中，我们使用`Public`、`Instance`和`DeclareOnly`来指定仅在此类型中声明的公共非静态方法，并排除继承的方法。这个程序的输出如下：

```cs
Engine
Name (String)
Capacity (Int32)
Power (Double)
Status (EngineStatus)
String get_Name ()
Int32 get_Capacity ()
Double get_Power ()
EngineStatus get_Status ()
Void Start ()
Void Stop ()
```

`Engine`类型位于执行反射代码的程序集中。但是，您也可以反射来自其他程序集的类型，无论它们是从执行程序集引用还是在运行时加载的，这是我们将在下一节中看到的。

# 动态加载程序集

反射服务允许您在运行时加载程序集。这是使用`System.Reflection.Assembly`类型完成的，它提供了各种加载程序集的方法。

程序集可以是*公共*（也称为*共享*）或*私有*。共享程序集旨在供多个应用程序使用，并且通常位于**全局程序集缓存（GAC）**下，这是程序集的系统存储库。私有程序集旨在供单个应用程序使用，并存储在应用程序目录或其子目录中。共享程序集必须具有强名称并强制执行版本约束；对于私有程序集，这些要求是不必要的。

程序集可以在三个上下文中之一加载，也可以不加载：

+   *加载上下文*，其中包含从 GAC、应用程序目录（应用程序域的`ApplicationBase`）或其私有程序集的子目录（应用程序域的`PrivateBinPath`）加载的程序集

+   *加载上下文*，其中包含从除了程序集加载程序探测的路径加载的程序集

+   *仅反射上下文*，其中包含仅用于反射目的加载的程序集，不能用于执行代码

+   *无上下文*，在某些特殊情况下使用，例如从字节数组加载的程序集

用于加载程序集的最重要的方法列在下表中：

![](img/Chapter_11_Table_3_01.jpg)

我们将看几个动态加载程序集的例子。

在第一个例子中，我们使用`Assembly.Load()`从应用程序目录加载名为`EngineLib`的程序集：

```cs
var assembly = Assembly.Load("EngineLib");
```

在这里，我们只指定了程序集的名称，但我们也可以指定显示名称，该名称不仅由名称组成，还包括版本、文化和用于签名程序集的公钥标记。对于没有强名称的程序集，这是`null`。在下面的行中，我们使用显示名称，与先前使用的行等效：

```cs
var assembly = Assembly.Load(@"EngineLib, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null");
```

可以使用`AssemblyName`类以类型安全的方式创建显示名称。该类具有各种属性和方法，允许您构建显示名称。可以按照以下方式完成：

```cs
var assemblyName = new AssemblyName()
{
    Name = "EngineLib",
    Version = new Version(1,0,0,0),
    CultureInfo = null,
};
var assembly = Assembly.Load(assemblyName);
```

公共（或共享）程序集必须具有强名称。这有助于唯一标识程序集，从而避免可能的冲突。签名是使用公共-私钥完成的；私钥用于签名，公钥与程序集一起分发并用于验证签名。

可以使用与 Visual Studio 一起分发的`sn.exe`工具生成这样的加密对；此工具也可用于验证签名。对于强名称程序集，必须指定`PublicKeyToken`，否则加载将失败。以下示例显示了如何从 GAC 加载`WindowsBase.dll`：

```cs
var assembly = Assembly.Load(@"WindowsBase, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35");
```

使用程序集名称加载程序集的替代方法是使用其实际路径。但是，在这种情况下，您必须使用`LoadFrom()`的一个重载之一。这对于必须加载既不在 GAC 中也不在应用程序文件夹下的程序集的情况非常有用。一个例子可以是一个可扩展的系统，可以加载可能安装在某个自定义目录中的插件：

```cs
var assembly = Assembly.LoadFrom(@"c:\learningc#8\chapter_11_02\bin\Debug\netcoreapp2.1\EngineLib.dll");
```

`Assembly`类具有提供有关程序集本身信息的成员，以及提供有关其包含的类型信息的成员。以下是一些最重要的成员：

![](img/Chapter_11_Table_4_01.jpg)

在下面的例子中，使用先前显示的方法之一加载程序集后，我们列出程序集名称和程序集清单中的文件，以及引用程序集的名称。之后，我们搜索`EngineLib.Engine`类型并打印其所有属性的名称和类型：

```cs
if (assembly != null)
{
    Console.WriteLine(
$@"Name: {assembly.GetName().FullName}
Files: {string.Join(',', 
                    assembly.GetFiles().Select(
                        s=>Path.GetFileName(s.Name)))}
Refs:  {string.Join(',', 
                    assembly.GetReferencedAssemblies().Select(
                        n=>n.Name))}");
    var type = assembly.GetType("EngineLib.Engine");
    if (type != null)
    {
        var properties = type.GetProperties();
        foreach (var p in properties)
        {
            Console.WriteLine(
              $"{p.Name} ({p.PropertyType.Name})");
        }
    }
}
```

除了查询有关程序集及其内容的信息之外，还可以在运行时从中执行代码。这是我们将在下一节中讨论的内容。

# 理解后期绑定

在编译时引用程序集时，编译器可以完全访问该程序集中可用的类型。这称为**早期绑定**。但是，如果程序集仅在运行时加载，编译器将无法访问该程序集的内容。这称为**后期绑定**，是构建可扩展应用程序的关键。使用后期绑定，您不仅可以加载和查询程序集，还可以执行代码。我们将在下面的例子中看到。

假设先前显示的`Engine`类在名为`EngineLib`的程序集中可用。可以使用`Assembly.Load()`或`Assembly.LoadFrom()`加载该程序集。加载后，我们可以使用`Assembly.GetType()`和`Type`的类方法获取有关`Engine`类型的信息。但是，使用`Assembly.CreateInstance()`，我们可以实例化该类的对象：

```cs
var assembly = Assembly.LoadFrom("EngineLib.dll");
if (assembly != null)
{
    var type = assembly.GetType("EngineLib.Engine");
    object engine = assembly.CreateInstance(
        "EngineLib.Engine",
        true,
        BindingFlags.CreateInstance,
        null,
        new object[] { "M270 Turbo", 1600, 75.0 },
        null,
        null);
    var pi = type.GetProperty("Status");
    if (pi != null)
        Console.WriteLine(pi.GetValue(engine));
    var mi = type.GetMethod("Start");
    if (mi != null)
        mi.Invoke(engine, null);
    if (pi != null)
        Console.WriteLine(pi.GetValue(engine));
}
```

`Assembly.CreateInstance()`方法有许多参数，但其中三个最重要：

+   第一个参数`string typeName`，表示程序集的名称。

+   第三个参数，`BindingFlags bindingAttr`，表示绑定标志。

+   第五个参数，`object[]` `args`，表示用于调用构造函数的参数数组；对于默认构造函数，这个对象可以是`null`。

在创建类型的实例之后，我们可以使用`PropertyInfo`、`MethodInfo`等的实例来调用其成员。例如，在前面的示例中，我们首先检索了名为`Status`的属性的`PropertyInfo`实例，然后通过调用`GetValue()`并传递引擎对象来获取属性的值。

同样地，我们使用`GetMethod()`来检索一个名为`Start()`的方法的`MethodInfo`实例，然后通过调用`Invoke()`来调用它。这个方法接受一个对象的引用和一个表示参数的对象数组；由于`Start()`方法没有参数，在这里使用了`null`。

`Assembly.CreateInstance()`方法有很多参数，使用起来可能很麻烦。作为替代，`System.Activator`类提供了在运行时创建类型实例的更简单的方法。它有一个重载的`CreateInstance()`方法。实际上，`Assembly.CreateInstance()`在内部实际上就是使用了它。在最简单的形式中，它只需要`Type`和一个表示构造函数参数的对象数组，并实例化该类型的对象。示例如下：

```cs
object engine = Activator.CreateInstance(
    type,
    new object[] { "M270 Turbo", 1600, 75.0 });
```

`Activator.CreateInstance()`不仅更简单易用，而且在某些情况下可以提供一些好处。例如，它可以在其他应用程序域或另一台服务器上使用远程调用来创建对象。另一方面，`Assembly.CreateIntance()`如果尚未加载程序集，则不会尝试加载程序集，而`System.Activator`会将程序集加载到当前应用程序域中。

使用晚期绑定和以前展示的方式调用代码并不一定实用。在实践中，当构建一个可扩展的系统时，您可能会有一个或多个包含接口和公共类型的程序集，这些插件（或插件，取决于您希望如何称呼它们）依赖于这些基本程序集。您将对这些基本程序集进行早期绑定，然后使用插件进行晚期绑定。

为了更好地理解这一点，我们将通过以下示例进行演示。`EngineLibBase`是一个定义了名为`IEngine`和`EngineStatus`枚举的接口的程序集：

```cs
namespace EngineLibBase
{
    public enum EngineStatus { Stopped, Started }
    public interface IEngine
    {
        EngineStatus Status { get; }
        void Start();
        void Stop();
    }
}
```

这个程序集直接被`EngineLib`程序集引用，它提供了实现`IEngine`接口的`Engine`类。示例如下：

```cs
using EngineLibBase;
namespace EngineLib
{ 
    public class Engine : IEngine
    {
        public string Name { get; }
        public int Capacity { get; }
        public double Power { get; }
        public EngineStatus Status { get; private set; }
        public Engine(string name, int capacity, double power)
        {
            Name = name;
            Capacity = capacity;
            Power = power;
            Status = EngineStatus.Stopped;
        }
        public void Start()
        {
            Status = EngineStatus.Started;
        }
        public void Stop()
        {
            Status = EngineStatus.Stopped;
        }
    }
}
```

在我们的应用程序中，我们再次引用了`EngineLibBase`程序集，以便我们可以使用`IEngine`接口。在运行时加载`EngineLib`程序集后，我们实例化了`Engine`类的对象，并将其转换为`IEngine`接口，这样即使在编译时实际实例未知的情况下，也可以访问接口的成员。代码如下所示：

```cs
var assembly = Assembly.LoadFrom("EngineLib.dll");
if (assembly != null)
{
    var type = assembly.GetType("EngineLib.Engine");
    var engine = (IEngine)Activator.CreateInstance(
        type,
        new object[] { "M270 Turbo", 1600, 75.0 });
    Console.WriteLine(engine.Status);
    engine.Start();
    Console.WriteLine(engine.Status);
}
```

正如我们将在本章后面看到的那样，这并不是使用晚期绑定和在运行时动态执行代码的唯一方法。另一种可能性是使用 DLR 和`dynamic`类型。我们将在下一节中看到这一点。

# 使用动态类型

在本书中，我们已经谈到了**CLR**。.NET Framework，然而，还包含了另一个组件，称为**动态语言运行时（DLR）**。这是另一个运行时环境，它在 CLR 之上添加了一组服务，使动态语言能够在 CLR 上运行，并为静态类型语言添加动态特性。C#和 Visual Basic 是静态类型语言。相比之下，诸如 JavaScript、Python、Ruby、PHP、Smalltalk、Lua 等语言是动态语言。这些语言的关键特征是它们在运行时识别对象的类型，而不是在编译时像静态类型语言那样。

DLR 为 C#（和 Visual Basic）提供了动态特性，使它们能够以简单的方式与动态语言进行互操作。如前所述，DLR 为 CLR 添加了一组服务。这些服务如下：

+   表达式树用于表示语言语义。这些是与 LINQ 一起使用的相同表达式树，但扩展到包括控制流、赋值和其他内容。

+   调用站点缓存是一个缓存有关操作和对象（如对象的类型）的信息的服务，这样当再次执行相同的操作时，它可以被快速分派。

+   `IDynamicMetaObjectProvider`、`DynamicMetaObject`、`DynamicObject`和`ExpandoObject`。

DLR 为 C# 4 引入的`dynamic`类型提供了基础设施。这是一个静态类型，这意味着在编译时为该类型的变量分配了`dynamic`类型。但是，它们绕过了静态类型检查。这意味着对象的实际类型只在运行时知道，编译器无法知道并且无法强制执行对该类型对象执行的任何检查。您实际上可以调用任何带有任何参数的方法，编译器不会检查和抱怨；但是，如果操作无效，运行时将抛出异常。

以下代码显示了`dynamic`类型的几个变量的示例。请注意，`s`是一个字符串，`l`是`List<int>`。调用`l.Add()`是有效的操作，因为`List<T>`包含这样的方法。但是，调用`s.Add()`是无效的，因为`string`类型没有这样的方法。因此，对于此调用，在运行时会抛出`RuntimeBinderException`类型的异常：

```cs
dynamic i = 42;
dynamic s = "42";
dynamic d = 42.0;
dynamic l = new List<int> { 42 };
l.Add(43); // OK
try
{
   s.Add(44); /* RuntimeBinderException:
            'string' does not contain a definition for 'Add' */
}
catch (Exception ex)
{
    Console.WriteLine(ex.Message);
}
```

`dynamic`类型使得在编译时不知道对象类型的情况下轻松消耗对象变得容易。考虑前一段中的第一个例子，在那里我们使用反射加载了一个程序集，实例化了一个`Engine`类型的对象并调用了它的方法和属性。可以用`dynamic`类型以更简单的方式重写该示例，如下所示：

```cs
var assembly = Assembly.LoadFrom("EngineLib.dll");
if (assembly != null)
{
    var type = assembly.GetType("EngineLib.Engine");
    dynamic engine = Activator.CreateInstance(
        type,
        new object[] { "M270 Turbo", 1600, 75.0 });
    Console.WriteLine(engine.Status);
    engine.Start();
    Console.WriteLine(engine.Status);
}
```

`dynamic`类型的对象在许多情况下的行为就像它具有`object`类型一样（除了没有编译时检查）。但是，对象值的实际来源是无关紧要的。它可以是.NET 对象、COM 对象、HTML DOM 对象、通过反射创建的对象，例如前面的示例等。

动态操作的结果类型也是`dynamic`，除了从`dynamic`到另一种类型的转换和包括`dynamic`类型参数的构造函数调用。从静态类型到`dynamic`的隐式转换以及相反的转换都会执行。代码块中显示了这一点：

```cs
dynamic d = "42";
string s = d;
```

对于静态类型，编译器执行重载解析以找出对函数调用的最佳匹配。因为在编译时没有关于`dynamic`类型的信息，所以对于至少有一个参数是`dynamic`类型的方法，同样的操作在运行时执行。

`dynamic`类型通常用于简化在互操作程序集不可用时消耗 COM 对象。以下是一个创建带有一些虚拟数据的 Excel 文档的示例：

```cs
dynamic excel = Activator.CreateInstance(
    Type.GetTypeFromProgID("Excel.Application.16")); 
if (excel != null)
{
    excel.Visible = true;

    dynamic workBook = excel.Workbooks.Add();
    dynamic workSheet = excel.ActiveWorkbook.ActiveSheet;
    workSheet.Cells[1, 1] = "ID";
    workSheet.Cells[1, 2] = "Name";
    workSheet.Cells[2, 1] = "1";
    workSheet.Cells[2, 2] = "One";
    workSheet.Cells[3, 1] = "2";
    workSheet.Cells[3, 2] = "Two";
    workBook.SaveAs("d:\\demo.xls", 
        Excel.XlFileFormat.xlWorkbookNormal, 
        AccessMode : Excel.XlSaveAsAccessMode.xlExclusive);
    workBook.Close(true);
    excel.Quit();
}
```

这段代码的作用如下：

+   它检索由程序标识符`Excel.Application.16`标识的 COM 对象的`System.Type`，并创建其实例。

+   它将 Excel 应用程序的`Visible`属性设置为`true`，这样您就可以看到窗口。

+   它创建一个工作簿并向其活动工作表添加一些数据。

+   它将文档保存在名为`demo.xls`的文件中。

+   它关闭工作簿并退出 Excel 应用程序。

在本章的最后一节中，我们将看看如何在反射服务中使用属性。

# 属性

属性提供有关程序集、类型和成员的元信息。编译器、CLR 或使用反射服务读取它们的工具会消耗这些元信息。属性实际上是从`System.Attribute`抽象类派生的类型。.NET 框架提供了大量的属性，但用户也可以定义自己的属性。

属性在方括号中指定，例如`[SerializableAttribute]`。属性的命名约定是类型名称总是以`Attribute`一词结尾。C#语言提供了一种语法快捷方式，允许在不带后缀`Attribute`的情况下指定属性的名称，例如`[Serializable]`。但是，只有在类型名称根据此约定正确后缀时才可能。

我们将在下一节首先介绍一些广泛使用的系统属性。

## 系统属性

.NET Framework 在不同的程序集和命名空间中提供了数百个属性。枚举它们不仅几乎不可能，而且也没有多大意义。然而，以下表格列出了一些经常使用的属性；其中一些我们在本书中已经见过：

![](img/Chapter_11_Table_5_01.jpg)

另一方面，通常需要或有用的是创建自己的属性类。在下一节中，我们将看看用户定义的属性。

## 用户定义的属性

您可以创建自己的属性来标记程序元素。您需要从`System.Attribute`派生，并遵循将类型后缀命名为`Attribute`的命名约定。以下是一个名为`Description`的属性，其中包含一个名为`Text`的属性：

```cs
class DescriptionAttribute : Attribute
{
    public string Text { get; private set; }
    public DescriptionAttribute(string description)
    {
        Text = description;
    }
}
```

此属性可用于装饰任何程序元素。在下面的示例中，我们可以看到这个属性用在了一个类、属性和方法参数上：

```cs
[Description("Main component of the car")]
class Engine
{
    public string Name { get; }
    [Description("cm³")]
    public int Capacity { get; }
    [Description("kW")]
    public double Power { get; }
    public Engine([Description("The name")] string name, 
                  [Description("The capacity")] int capacity, 
                  [Description("The power")] double power)
    {
        Name = name;
        Capacity = capacity;
        Power = power;
    }
}
```

属性可以有*位置*和*命名*参数：

+   位置参数由公共实例构造函数的参数定义。每个这样的构造函数的参数定义了一组命名参数。

+   另一方面，每个非静态公共字段和可读写属性定义了一个命名参数。

以下示例显示了早期介绍的`Description`属性，修改后可以使用一个名为`Required`的公共属性：

```cs
class DescriptionAttribute : Attribute
{
    public string Text { get; private set; }
    public bool Required { get; set; }
    public DescriptionAttribute(string description)
    {
        Text = description;
    }
}
```

此属性可以在程序元素上的属性声明中用作命名参数。如下例所示：

```cs
[Description("Main component of the car", Required = true)]
class Engine
{
}
```

让我们在下一节中学习如何使用属性。

## 如何使用属性？

程序元素可以标记多个属性。有两种等效的方法可以实现这一点：

+   第一种方法（因为它最具描述性和清晰，所以被广泛使用）是在一对方括号内分别声明每个属性。以下示例显示了如何完成此操作：

```cs
[Serializable]
[Description("Main component of the car")]
[ComVisible(false)]
class Engine
{
}
```

+   另一种方法是在同一对方括号内声明多个属性，用逗号分隔。以下代码等同于之前的代码：

```cs
[Serializable, 
 Description("Main component of the car"), 
 ComVisible(false)]
class Engine
{
}
```

让我们在下一节中看看如何指定属性的目标。

## 属性目标

默认情况下，属性应用于它前面的任何程序元素。但是，可以指定目标，比如类型、方法等。这是通过使用另一个名为`AttributeUsage`的属性标记属性类型来完成的。除了指定目标外，此属性还允许指定新定义的属性是否可以多次应用以及是否可以继承。

以下修改后的`DescriptionAttribute`版本指示它只能用于类、结构、方法、属性和字段。此外，它指定了该属性被派生类继承，并且可以在同一元素上多次使用：

```cs
[AttributeUsage(AttributeTargets.Class|
                AttributeTargets.Struct|
                AttributeTargets.Method|
                AttributeTargets.Property|
                AttributeTargets.Field,
                AllowMultiple = true,
                Inherited = true)]
class DescriptionAttribute : Attribute
{
    public string Text { get; private set; }
    public bool Required { get; set; }
    public DescriptionAttribute(string description)
    {
        Text = description;
    }
}
```

由于这些变化，这个属性不能再用于方法参数，就像之前的例子中所示的那样。那将导致编译器错误。

到目前为止，我们使用的属性针对程序元素，如类型和方法。但是也可以使用程序集级属性。我们将在下一节中看到这些。

## 程序集属性

有一些属性可以针对程序集并指定有关程序集的信息。这些信息可以是程序集的标识（即名称、版本和文化）、清单信息、强名称或其他信息。这些属性使用语法`[assembly: attribute]`指定。这些属性通常可以在为每个.NET Framework 项目生成的`AssemblyInfo.cs`文件中找到。以下是这些属性的一个示例：

```cs
[assembly: AssemblyTitle("project_name")]
[assembly: AssemblyDescription("")]
[assembly: AssemblyConfiguration("")]
[assembly: AssemblyCompany("")]
[assembly: AssemblyProduct("project_name")]
[assembly: AssemblyCopyright("Copyright © 2019")]
[assembly: AssemblyTrademark("")]
[assembly: AssemblyCulture("")]
[assembly: AssemblyVersion("1.0.0.0")]
[assembly: AssemblyFileVersion("1.0.0.0")]
```

属性用于反射服务。既然我们已经看到了如何创建和使用属性，让我们看看如何在反射中使用它们。

## 反射中的属性

属性本身没有太大的价值，直到有人反映它们并根据属性的含义和值执行特定的操作。`System.Type`类型以及`System.Reflection`命名空间中的其他类型都有一个名为`GetCustomAttributes()`的重载方法，用于检索特定程序元素标记的属性。其中一个重载采用属性的类型，因此它只返回该类型的实例；另一个不是，返回所有属性。

以下示例从`Engine`类型中首先检索所有`Description`属性的实例，然后从类型的所有属性中检索并在控制台中显示描述文本：

```cs
var e = new Engine("M270 Turbo", 1600, 75.0);
var type = e.GetType();
var attributes = type.GetCustomAttributes(typeof(DescriptionAttribute), 
                                          true);
if (attributes != null)
{
    foreach (DescriptionAttribute attr in attributes)
    {
        Console.WriteLine(attr.Text);
    }
}
var properties = type.GetProperties();
foreach (var property in properties)
{
    var pattributes = 
      property.GetCustomAttributes(
         typeof(DescriptionAttribute), false);
    if (attributes != null)
    {
        foreach (DescriptionAttribute attr in pattributes)
        {
            Console.WriteLine(
              $"{property.Name} [{attr.Text}]");
        }
    }
}
```

该程序的输出如下：

```cs
Main component of the car
Capacity [cm3]
Power [kW]
```

# 摘要

在本章中，我们看了反射服务，如何在运行时加载程序集，并查询关于类型的元信息。我们还学习了如何使用系统反射和 DLR 以及动态类型来动态执行代码。DLR 为 C#提供了动态特性，并以简单的方式实现了与动态语言的互操作性。本章最后涵盖的主题是属性。我们学习了常见的系统属性是什么，以及如何创建自己的类型以及如何在反射中使用它们。

在下一章中，我们将专注于并发和并行性。

# 测试你学到的东西

1.  .NET 中的部署单位是什么，它包含什么？

1.  什么是反射？它提供了什么好处？

1.  .NET 类型暴露了关于类型的元数据？你如何创建这种类型的实例？

1.  公共程序集和私有程序集之间有什么区别？

1.  在.NET Framework 中，程序集可以在什么上下文中被加载？

1.  什么是早期绑定？晚期绑定呢？后者提供了什么好处？

1.  什么是动态语言运行时？

1.  动态类型是什么，它通常在哪些场景中使用？

1.  属性是什么，你如何在代码中指定它们？

1.  你如何创建用户定义的属性？
