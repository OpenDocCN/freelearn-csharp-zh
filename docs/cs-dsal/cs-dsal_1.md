# 第一章：入门

开发应用程序肯定是一件令人兴奋的工作，但也具有挑战性，特别是如果您需要解决涉及高级数据结构和算法的复杂问题。在这种情况下，您经常需要关注性能，以确保解决方案在资源有限的设备上能够平稳运行。这样的任务可能非常困难，可能需要对编程语言、数据结构和算法有相当的了解。

您知道吗，即使将一个数据结构替换为另一个，也可能导致性能结果增加数百倍？听起来不可能吗？也许，但这是真的！举个例子，我想告诉您一个我参与的项目的简短故事。其目标是优化在图形图表上查找块之间连接的算法。这样的连接应该在图表中的任何块移动时自动重新计算、刷新和重绘。当然，连接不能穿过块，也不能重叠其他线，并且交叉点和方向变化的数量应该是有限的。根据图表的大小和复杂性，性能结果会有所不同。然而，在进行测试时，我们得到了同一个测试用例的结果范围从 1 毫秒到近 800 毫秒。最令人惊讶的可能是，这样巨大的改进主要是通过...改变了两组数据结构来实现的。

现在，您可能会问自己一个显而易见的问题：在特定情况下应该使用哪些数据结构，以及可以用哪些算法来解决一些常见问题？不幸的是，答案并不简单。然而，在本书中，您将找到许多关于数据结构和算法的信息，以 C#编程语言的背景呈现，包括许多示例、代码片段和详细解释。这样的内容可以帮助您回答前面提到的问题，同时开发下一个伟大的解决方案，这些解决方案可以被世界各地的许多人使用！您准备好开始您的数据结构和算法之旅了吗？如果是的，让我们开始吧！

在本章中，您将涵盖以下主题：

+   编程语言

+   数据类型

+   IDE 的安装和配置

+   创建项目

+   输入和输出

+   启动和调试

# 编程语言

作为开发人员，您肯定听说过许多编程语言，如 C#、Java、C++、C、PHP 或 Ruby。在所有这些语言中，您可以使用各种数据结构，以及实现算法，来解决基本和复杂的问题。然而，每种语言都有其自身的特点，这在实现数据结构和相应的算法时可能是可见的。正如前面提到的，本书将专注于 C#编程语言，这也是本节的主要内容。

C#语言，发音为“C Sharp”，是一种现代的、通用的、强类型的、面向对象的编程语言，可用于开发各种应用程序，如 Web、移动、桌面、分布式和嵌入式解决方案，甚至游戏。它与各种其他技术和平台合作，包括 ASP.NET MVC、Windows Store、Xamarin、Windows Forms、XAML 和 Unity。因此，当您学习 C#语言，以及在这种编程语言的背景下更多地了解数据结构和算法时，您可以利用这些技能来创建多种特定类型的软件。

当前版本的语言是 C# 7.1。值得一提的是它与语言的以下版本（例如 2.0、3.0 和 5.0）的有趣历史，在这些版本中，已添加了新功能以增加语言的可能性并简化开发人员的工作。当您查看特定版本的发布说明时，您将看到语言如何随着时间的推移而得到改进和扩展。

C#编程语言的语法类似于其他语言，比如 Java 或 C++。因此，如果您了解这些语言，您应该很容易理解用 C#编写的代码。例如，与之前提到的语言类似，代码由以分号（`;`）结尾的语句组成，花括号（`{`和`}`）用于分组语句，比如在`foreach`循环中。您还可以找到类似的代码结构，比如`if`语句，或`while`和`for`循环。

在 C#语言中开发各种应用程序也因为许多额外的出色功能而变得简化，比如**语言集成查询**（**LINQ**），它允许开发人员以一致的方式从各种集合中获取数据，比如 SQL 数据库或 XML 文档。还有一些缩短所需代码的方法，比如使用 lambda 表达式、表达式主体成员、getter 和 setter，或者字符串插值。值得一提的是自动垃圾回收，它简化了释放内存的任务。当然，上述解决方案只是在 C#开发中可用功能的非常有限的子集。在本书的后续部分中，您将看到一些其他功能，以及示例和详细描述。

# 数据类型

在 C#语言中开发应用程序时，您可以使用各种数据类型，它们分为两组，即**值类型**和**引用类型**。它们之间的区别非常简单——值类型的变量直接包含数据，而引用类型的变量只是存储对数据的引用，如下所示：

![](img/d38bd3e0-f4c1-4c4c-8b09-7125d7d0db69.png)

正如您所看到的，**值类型**直接将其实际**值**存储在**堆栈**内存中，而**引用类型**只在此处存储**引用**。实际值位于**堆**内存中。因此，也可能有两个或更多引用类型的变量引用完全相同的值。

当然，值类型和引用类型之间的区别在编程时非常重要，您应该知道哪些类型属于上述组。否则，您可能会在代码中犯错，这可能会很难找到。例如，您应该记住在更新引用类型的数据时要小心，因为更改也可能会反映在引用相同对象的其他变量中。此外，您在使用等号（`=`）运算符比较两个对象时也要小心，因为在比较两个引用类型的实例时，您可能会比较引用而不是数据本身。

C#语言还支持**指针类型**，可以声明为`type* identifier`或`void* identifier`。然而，这些类型超出了本书的范围。您可以在以下链接中了解更多信息：[`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/unsafe-code-pointers/pointer-types`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/unsafe-code-pointers/pointer-types)。

# 值类型

为了让您更好地理解数据类型，让我们从对第一组（即**值类型**）的分析开始，它可以进一步分为**结构**和**枚举**。

更多信息请访问：[`docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/value-types`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/value-types)。

# 结构

在结构体中，您可以访问许多内置类型，这些类型可以作为关键字或来自`System`命名空间的类型使用。

其中之一是`Boolean`类型（`bool`关键字），它可以存储**逻辑值**，也就是两个值中的一个，即`true`或`false`。

至于存储**整数值**，您可以使用以下类型之一：`Byte`（`byte`关键字）、`SByte`（`sbyte`）、`Int16`（`short`）、`UInt16`（`ushort`）、`Int32`（`int`）、`UInt32`（`uint`）、`Int64`（`long`）和`UInt64`（`ulong`）。它们通过存储值的字节数和可用值的范围而有所不同。例如，`short`数据类型支持范围从-32,768 到 32,767 的值，而`uint`支持范围从 0 到 4,294,967,295 的值。整数类型中的另一种类型是`Char`（`char`），它表示单个 Unicode 字符，例如`'a'`或`'M'`。

在**浮点值**的情况下，您可以使用两种类型，即`Single`（`float`）和`Double`（`double`）。第一种使用 32 位，而第二种使用 64 位。因此，它们的精度有很大的不同。

此外，`Decimal`类型（`decimal`关键字）也是可用的。它使用 128 位，是货币计算的一个很好的选择。

C#编程语言中变量的一个示例声明如下：

```cs
int number; 
```

您可以使用等号（`=`）将值赋给变量，如下所示：

```cs
number = 500; 
```

当然，声明和赋值可以在同一行中执行：

```cs
int number = 500; 
```

如果您想声明和初始化一个**不可变值**，也就是一个**常量**，您可以使用`const`关键字，如下面的代码行所示：

```cs
const int DAYS_IN_WEEK = 7; 
```

有关内置数据类型的更多信息，以及完整的范围列表，请访问：[`msdn.microsoft.com/library/cs7y5x0x.aspx`](https://msdn.microsoft.com/library/cs7y5x0x.aspx)。

# 枚举

除了结构体，值类型还包括**枚举**。每个枚举都有一组命名的常量来指定可用的值集。例如，您可以创建可用语言或支持的货币的枚举。一个示例定义如下：

```cs
enum Language { PL, EN, DE }; 
```

然后，您可以将定义的枚举用作数据类型，如下所示：

```cs
Language language = Language.PL; 
switch (language) 
{ 
    case Language.PL: /* Polish version */ break; 
    case Language.DE: /* German version */ break; 
    default: /* English version */ break; 
} 
```

值得一提的是，枚举允许您用常量值替换一些*魔术字符串*（如`"PL"`或`"DE"`），这对代码质量有积极的影响。

您还可以从枚举的更高级特性中受益，例如更改基础类型或为特定常量指定值。您可以在此处找到更多信息：[`docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/enum`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/enum)。

# 引用类型

第二个主要类型组称为**引用类型**。作为一个快速提醒，引用类型的变量并不直接包含数据，因为它只是存储数据的引用。在这个组中，您可以找到三种内置类型，即`string`、`object`和`dynamic`。此外，您可以声明类、接口和委托。

有关引用类型的更多信息，请访问：[`docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/reference-types`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/reference-types)。

# 字符串

通常需要存储一些文本值。您可以使用`System`命名空间中的内置引用类型`String`来实现这一目标，也可以使用`string`关键字。`string`类型是 Unicode 字符的序列。它可以有零个字符、一个或多个字符，或者`string`变量可以设置为`null`。

您可以对`string`对象执行各种操作，例如连接或使用`[]`运算符访问特定字符，如下所示：

```cs
string firstName = "Marcin", lastName = "Jamro"; 
int year = 1988; 
string note = firstName + " " + lastName.ToUpper()  
   + " was born in " + year; 
string initials = firstName[0] + "." + lastName[0] + "."; 
```

一开始，声明了`firstName`变量，并将`"Marcin"`赋给它。同样，`"Jamro"`被设置为`lastName`变量的值。在第三行，您连接了五个字符串（使用`+`运算符），即`firstName`的当前值，空格，`lastName`的当前值转换为大写字符串（通过调用`ToUpper`方法），字符串`" was born in "`，以及`year`变量的当前值。在最后一行，使用`[]`运算符获取了`firstName`和`lastName`变量的第一个字符，并与两个点连接起来形成了缩写，即`M.J.`，这些缩写作为`initials`变量的值存储。

`Format`静态方法也可用于构造字符串，如下所示：

```cs
string note = string.Format("{0} {1} was born in {2}",  
   firstName, lastName.ToUpper(), year); 
```

在这个例子中，您指定了包含三个格式项的**复合格式字符串**，即`firstName`（由`{0}`表示），大写`lastName`（`{1}`），以及`year`（`{2}`）。要格式化的对象被指定为以下参数。

更多信息可在以下网址找到：[`docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/string`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/string)。

还值得一提的是**插值字符串**，它使用**插值表达式**来构造一个`string`。要使用这种方法创建一个`string`，需要在“”之前放置`$`字符，如下例所示：

```cs
string note = $"{firstName} {lastName.ToUpper()}  
   was born in {year}"; 
```

更多信息可在以下网址找到：[`docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interpolated-strings`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interpolated-strings)。

# 对象

`Object`类在`System`命名空间中声明，它在 C#语言中开发应用程序时扮演着非常重要的角色，因为它是所有类的基类。这意味着内置值类型和内置引用类型，以及用户定义的类型，都是从`Object`类派生出来的，也可以使用`object`别名来访问。

由于`object`类型是所有值类型的基本实体，这意味着可以将任何值类型的变量（例如`int`或`float`）转换为`object`类型，也可以将`object`类型的变量转换回特定的值类型。这些操作分别称为**装箱**（第一个）和**拆箱**（另一个）。它们如下所示：

```cs
int age = 28; 
object ageBoxing = age; 
int ageUnboxing = (int)ageBoxing; 
```

更多信息可在以下网址找到：[`docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/object`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/object)。

# 动态

除了已经描述的类型，还有`dynamic`类型可供开发人员使用。它允许在编译期间绕过类型检查，以便您可以在运行时执行它。这种机制在访问一些**应用程序编程接口**（**API**）时非常有用，但本书不会使用它。

更多信息可在以下网址找到：[`docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/dynamic`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/dynamic)。

# 类

如前所述，C#是一种面向对象的语言，支持声明类以及各种成员，包括构造函数、终结器、常量、字段、属性、索引器、事件、方法和运算符，以及委托。此外，类支持继承和实现接口。还有静态、抽象和虚拟成员可用。

以下是一个示例类：

```cs
public class Person 
{ 
    private string _location = string.Empty; 
    public string Name { get; set; } 
    public int Age { get; set; } 

    public Person() => Name = "---"; 

    public Person(string name, int age) 
    { 
        Name = name; 
        Age = age; 
    } 

    public void Relocate(string location) 
    { 
        if (!string.IsNullOrEmpty(location)) 
        { 
            _location = location; 
        } 
    } 

    public float GetDistance(string location) 
    { 
        return DistanceHelpers.GetDistance(_location, location); 
    } 
} 
```

`Person`类包含`_location`私有字段，默认值设置为空字符串（`string.Empty`），两个公共属性（`Name`和`Age`），一个默认构造函数，使用**表达式体定义**将`Name`属性的值设置为`---`，一个接受两个参数并设置属性值的额外构造函数，`Relocate`方法更新私有字段的值，以及`GetDistance`方法调用`DistanceHelpers`类的`GetDistance`静态方法，并返回两个城市之间的距离（以公里为单位）。

您可以使用`new`运算符创建类的实例。然后，您可以对创建的对象执行各种操作，比如调用方法，如下所示：

```cs
Person person = new Person("Mary", 20); 
person.Relocate("Rzeszow"); 
float distance = person.GetDistance("Warsaw");  
```

更多信息可在此处找到：[`docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/class`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/class)。

# 接口

在前面的部分中，提到了一个可以实现一个或多个**接口**的类。这意味着这样一个类必须实现所有在所有实现的接口中指定的方法、属性、事件和索引器。您可以使用`interface`关键字在 C#语言中轻松定义接口。

举个例子，让我们来看一下以下代码：

```cs
public interface IDevice 
{ 
    string Model { get; set; } 
    string Number { get; set; } 
    int Year { get; set; } 

    void Configure(DeviceConfiguration configuration); 
    bool Start(); 
    bool Stop(); 
} 
```

`IDevice`接口包含三个属性，分别表示设备型号（`Model`）、序列号（`Number`）和生产年份（`Year`）。此外，它还具有三个方法的签名，分别是`Configure`、`Start`和`Stop`。当一个类实现`IDevice`接口时，它应该包含上述属性和方法。

更多信息可在此处找到：[`docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface)。

# 委托

`delegate`引用类型允许指定方法的必需签名。然后可以实例化委托，并像下面的代码中所示那样调用它。

```cs
delegate double Mean(double a, double b, double c); 

static double Harmonic(double a, double b, double c) 
{ 
    return 3 / ((1 / a) + (1 / b) + (1 / c)); 
} 

static void Main(string[] args) 
{ 
    Mean arithmetic = (a, b, c) => (a + b + c) / 3; 
    Mean geometric = delegate (double a, double b, double c) 
    { 
        return Math.Pow(a * b * c, 1 / 3.0); 
    }; 
    Mean harmonic = Harmonic; 
    double arithmeticResult = arithmetic.Invoke(5, 6.5, 7); 
    double geometricResult = geometric.Invoke(5, 6.5, 7); 
    double harmonicResult = harmonic.Invoke(5, 6.5, 7); 
} 
```

在示例中，`Mean`委托指定了用于计算三个浮点数的平均值的方法的必需签名。它使用 lambda 表达式（`arithmetic`）、匿名方法（`geometric`）和命名方法（`harmonic`）进行实例化。通过调用`Invoke`方法来调用每个委托。

更多信息可在此处找到：[`docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/delegate`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/delegate)。

# IDE 的安装和配置

在阅读本书时，您将看到许多示例，展示了数据结构和算法，以及详细的描述。代码的最重要部分将直接显示在书中。此外，完整的源代码也可以下载。当然，您可以只从书中阅读代码，但强烈建议您自己编写这样的代码，然后启动和调试程序，以了解各种数据结构和算法的运行方式。

如前所述，本书中展示的示例将使用 C#语言准备。为了保持简单，将创建基于控制台的应用程序，但这样的数据结构也可以用在其他类型的解决方案中。

示例项目将在**Microsoft Visual Studio 2017 Community**中创建。这个**集成开发环境**（**IDE**）是开发各种项目的综合解决方案。要下载、安装和配置它，您应该：

1.  打开网站[`www.visualstudio.com/downloads/`](https://www.visualstudio.com/downloads/)，并在 Visual Studio Community 2017 部分的 Visual Studio Downloads 标题下选择免费下载选项。安装程序的下载过程应该会自动开始。

1.  运行下载的文件并按照说明开始安装。当显示可能选项的屏幕时，选择.NET 桌面开发选项，如下面的屏幕截图所示。然后，点击安装。安装可能需要一些时间，但可以使用获取和应用进度条来观察其进展。

![](img/271ec5a2-9e1a-4567-9fb2-98ae4aa24a33.png)

1.  当显示安装成功！的消息时，点击启动按钮启动 IDE。您将被要求使用 Microsoft 帐户登录。然后，您应该在“以熟悉的环境开始”部分选择适当的开发设置（如 Visual C#）。此外，您应该从蓝色、蓝色（额外对比）、深色和浅色中选择颜色主题。最后，点击“启动 Visual Studio”按钮。

# 创建项目

在启动 IDE 后，让我们继续创建一个新项目。在阅读本书时，根据特定章节提供的信息，将执行这样的过程多次，以创建示例应用程序。

要创建一个新项目：

1.  在主菜单中点击“文件 | 新建 | 项目”。

1.  在新项目窗口的左侧选择已安装 | Visual C# | Windows 经典桌面，如下面的屏幕截图所示。然后，在中间点击 Console App (.NET Framework)。您还应该输入项目的名称（名称）和解决方案的名称（解决方案名称），并通过按浏览按钮选择文件的位置（位置）。最后，点击确定以自动创建项目并生成必要的文件：

![](img/f27929bd-547a-45de-b61d-81356765196b.png)

恭喜，您刚刚创建了第一个项目！但里面有什么呢？

让我们看看“解决方案资源管理器”窗口，它显示了项目的结构。值得一提的是，该项目包含在同名的解决方案中。当然，一个解决方案可以包含多个项目，这在开发更复杂的应用程序时是常见的情况。

如果找不到“解决方案资源管理器”窗口，可以通过从主菜单中选择“查看 | 解决方案资源管理器”选项来打开它。类似地，您可以打开其他窗口，如输出或类视图。如果在“查看”选项中找不到合适的窗口（例如 C#交互），让我们尝试在“查看 | 其他窗口”节点中找到它。

自动生成的项目（名为`GettingStarted`）具有以下结构：

+   “属性”节点包含一个文件（`AssemblyInfo.cs`），其中包含有关应用程序的程序集的一般信息，例如标题、版权和版本。使用属性进行配置，例如`AssemblyTitleAttribute`和`AssemblyVersionAttribute`。

+   “引用”元素显示了项目使用的其他程序集或项目。值得注意的是，您可以通过从“引用”元素的上下文菜单中选择“添加引用”选项来轻松添加引用。此外，您可以使用 NuGet 软件包管理器安装其他软件包，该软件包可以通过从“引用”上下文菜单中选择“管理 NuGet 软件包”来启动。

在自己编写复杂模块之前，先看看已经可用的包是个好主意，因为适当的包可能已经为开发人员提供。在这种情况下，您不仅可以缩短开发时间，还可以减少引入错误的机会。

+   `App.config`文件包含应用程序的基于**可扩展标记语言**（**XML**）的配置，包括.NET Framework 平台的最低支持版本号。

+   `Program.cs`文件包含 C#语言中主类的代码。您可以通过更改以下默认实现来调整应用程序的行为：

```cs
using System; 
using System.Collections.Generic; 
using System.Linq; 
using System.Text; 
using System.Threading.Tasks; 

namespace GettingStarted 
{ 
    class Program 
    { 
        static void Main(string[] args) 
        { 
        } 
    } 
} 
```

`Program.cs`文件的初始内容包含了`GettingStarted`命名空间中`Program`类的定义。该类包含了`Main`静态方法，当应用程序启动时会自动调用。还包括了五个`using`语句，分别是`System`、`System.Collections.Generic`、`System.Linq`、`System.Text`和`System.Threading.Tasks`。

在继续之前，让我们在文件资源管理器中查看项目的结构，而不是在“解决方案资源管理器”窗口中。这些结构是否完全相同？

您可以通过在“解决方案资源管理器”窗口中的项目节点的上下文菜单中选择“在文件资源管理器中打开文件夹”选项来打开项目所在的目录。

首先，您可以看到自动生成的`bin`和`obj`目录。两者都包含与 IDE 中设置的配置相关的`Debug`和`Release`目录。构建项目后，`bin`目录的子目录（即`Debug`或`Release`）包含`.exe`、`.exe.config`和`.pdb`文件，而`obj`目录中的子目录，例如，包含`.cache`和一些临时`.cs`文件。此外，没有`References`目录，但是有项目的基于 XML 的`.csproj`和`.csproj.user`文件。类似地，基于解决方案的`.sln`配置文件位于解决方案的目录中。

如果您正在使用**版本控制系统**，比如**SVN**或**Git**，您可以忽略`bin`和`obj`目录，以及`.csproj.user`文件。所有这些都可以自动生成。

如果您想学习如何编写一些示例代码，以及启动和调试程序，让我们继续到下一节。

# 输入和输出

书的后面部分中展示的许多示例将需要与用户进行交互，特别是通过读取输入数据和显示输出。您可以按照本节中的说明轻松地向应用程序添加这些功能。

# 从输入中读取

应用程序可以使用`System`命名空间中`Console`静态类的几种方法从**标准输入流**中读取数据，例如`ReadLine`和`ReadKey`。这两者都在本节的示例中展示了。

让我们来看看下面的代码行：

```cs
string fullName = Console.ReadLine(); 
```

在这里，您使用`ReadLine`方法。它会等待用户按下*Enter*键。然后，输入的文本将作为`fullName`字符串变量的值存储。

以类似的方式，您可以读取其他类型的数据，例如`int`，如下所示：

```cs
string numberString = Console.ReadLine(); 
int.TryParse(numberString, out int number); 
```

在这种情况下，调用了相同的`ReadLine`方法，并将输入的文本存储为`numberString`变量的值。然后，您只需要将其解析为`int`并将其存储为`int`变量的值。您可以如何做到这一点？解决方案非常简单——使用`Int32`结构的`TryParse`静态方法。值得一提的是，这样的方法返回一个布尔值，指示解析过程是否成功完成。因此，当提供的`string`表示不正确时，您可以执行一些额外的操作。

在下面的示例中，展示了关于`DateTime`结构和`TryParseExact`静态方法的类似情况：

```cs
string dateTimeString = Console.ReadLine(); 
if (!DateTime.TryParseExact( 
    dateTimeString, 
    "M/d/yyyy HH:mm", 
    new CultureInfo("en-US"), 
    DateTimeStyles.None, 
    out DateTime dateTime)) 
{ 
    dateTime = DateTime.Now; 
} 
```

这个示例比之前的更复杂，所以让我们详细解释一下。首先，日期和时间的字符串表示被存储为`dateTimeString`变量的值。然后，调用了`DateTime`结构的`TryParseExact`静态方法，传递了五个参数，即日期和时间的字符串表示（`dateTimeString`）、日期和时间的预期格式（`M/d/yyyy HH:mm`）、支持的文化（`en-US`）、附加样式（`None`），以及通过`out`参数修饰符传递的输出变量（`dateTime`）。

如果解析未成功完成，则将当前日期和时间（`DateTime.Now`）分配给`dateTime`变量。否则，`dateTime`变量包含与用户提供的`string`表示一致的`DateTime`实例。

在涉及`CultureInfo`类名称的代码部分中，您可能会看到以下错误：`CS0246 The type or namespace name 'CultureInfo' could not be found (are you missing a using directive or an assembly reference?)`。这意味着您在文件顶部没有合适的`using`语句。您可以通过单击显示在错误行左侧边缘的灯泡图标并选择`using System.Globalization;`选项来轻松添加一个。IDE 将自动添加缺少的`using`语句，错误将消失。

除了读取整行外，您还可以了解用户按下了哪个字符或功能键。为此，您可以使用`ReadKey`方法，如下面的代码部分所示：

```cs
ConsoleKeyInfo key = Console.ReadKey(); 
switch (key.Key) 
{ 
    case ConsoleKey.S: /* Pressed S */ break; 
    case ConsoleKey.F1: /* Pressed F1 */ break; 
    case ConsoleKey.Escape: /* Pressed Escape */ break; 
} 
```

调用`ReadKey`静态方法后，一旦用户按下任意键，按下的键的信息就会被存储为`ConsoleKeyInfo`实例（在当前示例中为`key`）。然后，您可以使用`Key`属性获取表示特定键的枚举值（`ConsoleKey`）。最后，使用`switch`语句根据按下的键执行操作。在所示的示例中，支持三个键，即*S*，*F1*和*Esc*。

# 写入输出

现在，您知道如何读取输入数据，但如何向用户提问或在屏幕上显示结果呢？答案以及示例在本节中展示。

与读取数据一样，与**标准输出流**相关的操作使用`System`命名空间中`Console`静态类的方法执行，即`Write`和`WriteLine`。让我们看看它们的运作方式！

要写一些文本，您只需调用`Write`方法，将文本作为参数传递。代码示例如下：

```cs
Console.Write("Enter a name: "); 
```

前一行导致显示以下输出：

```cs
    Enter a name: 
```

这里重要的是，所写的文本后面没有跟随换行符。如果要写一些文本并移到下一行，可以使用`WriteLine`方法，如下面的代码片段所示：

```cs
Console.WriteLine("Hello!"); 
```

执行此行代码后，将呈现以下输出：

```cs
    Hello!
```

当然，您还可以在更复杂的情况下使用`Write`和`WriteLine`方法。例如，您可以向`WriteLine`方法传递许多参数，即格式和附加参数，如下面代码的部分所示：

```cs
string name = "Marcin"; 
Console.WriteLine("Hello, {0}!", name); 
```

在这种情况下，该行将包含`Hello`，逗号，空格，`name`变量的值（即`Marcin`），以及感叹号。输出如下所示：

```cs
    Hello, Marcin!
```

下一个示例呈现了一个更复杂的场景，涉及在餐厅预订桌子的确认。输出应具有格式`Table [number] has been booked for [count] people on [date] at [time]`。您可以通过使用`WriteLine`方法来实现这个目标，如下所示：

```cs
string tableNumber = "A100"; 
int peopleCount = 4; 
DateTime reservationDateTime = new DateTime( 
    2017, 10, 28, 11, 0, 0); 
CultureInfo cultureInfo = new CultureInfo("en-US"); 
Console.WriteLine( 
    "Table {0} has been booked for {1} people on {2} at {3}", 
    tableNumber, 
    peopleCount, 
    reservationDateTime.ToString("M/d/yyyy", cultureInfo), 
    reservationDateTime.ToString("HH:mm", cultureInfo)); 
```

该示例以声明四个变量开始，即`tableNumber`（`A100`），`peopleCount`（`4`），`reservationDateTime`（2017 年 10 月 28 日上午 11:00），以及`cultureInfo`（`en-US`）。然后，调用`WriteLine`方法，传递五个参数，即格式字符串，后跟应显示在标有`{0}`，`{1}`，`{2}`和`{3}`的位置的参数。值得一提的是最后两行，其中基于`reservationDateTime`变量的当前值创建了表示日期（或时间）的字符串。

执行此代码后，将在输出中显示以下行：

```cs
    Table A100 has been booked for 4 people on 10/28/2017 at 11:00 
```

当然，在现实场景中，您将在同一代码中使用读取和写入相关的方法。例如，您可以要求用户提供一个值（使用`Write`方法），然后读取输入的文本（使用`ReadLine`方法）。

这个简单的例子，在本章的下一节中也很有用，如下所示。它允许用户输入与表格预订相关的数据，即桌号和人数，以及预订日期。当所有数据都输入后，将呈现确认。当然，用户将看到应提供的数据的信息：

```cs
using System; 
using System.Globalization; 

namespace GettingStarted 
{ 
    class Program 
    { 
        static void Main(string[] args) 
        { 
            CultureInfo cultureInfo = new CultureInfo("en-US"); 

            Console.Write("The table number: "); 
            string table = Console.ReadLine(); 

            Console.Write("The number of people: "); 
            string countString = Console.ReadLine(); 
            int.TryParse(countString, out int count); 

            Console.Write("The reservation date (MM/dd/yyyy): "); 
            string dateTimeString = Console.ReadLine(); 
            if (!DateTime.TryParseExact( 
                dateTimeString, 
                "M/d/yyyy HH:mm", 
                cultureInfo, 
                DateTimeStyles.None, 
                out DateTime dateTime)) 
            { 
                dateTime = DateTime.Now; 
            } 

            Console.WriteLine( 
                "Table {0} has been booked for {1} people on {2}  
                 at {3}", 
                table, 
                count, 
                dateTime.ToString("M/d/yyyy", cultureInfo), 
                dateTime.ToString("HH:mm", cultureInfo)); 
        } 
    } 
} 
```

前面的代码片段是基于先前显示和描述的代码部分。启动程序并输入必要的数据后，输出可能如下所示：

```cs
    The table number: A100
    The number of people: 4
    The reservation date (MM/dd/yyyy): 10/28/2017 11:00
    Table A100 has been booked for 4 people on 10/28/2017 at 11:00
    Press any key to continue . . . 
```

编写代码时，改进其质量是个好主意。与 IDE 相关的有趣可能性之一是删除未使用的`using`语句，以及对剩余语句进行排序。您可以通过在文本编辑器中选择“删除并排序使用”选项来轻松执行此操作。

# 启动和调试

不幸的是，编写的代码并不总是按预期工作。在这种情况下，最好开始**调试**，看看程序的运行方式，找到问题的源头并进行更正。这项任务对于复杂的算法特别有用，其中流程可能很复杂，因此仅通过阅读代码就很难分析。幸运的是，IDE 配备了各种调试功能，将在本节中介绍。

首先，让我们启动应用程序，看看它的运行情况！要这样做，您只需从下拉列表中选择适当的配置（在本例中为调试），然后单击主工具栏中带有绿色三角形和“开始”标题的按钮，或按下*F5*。要停止调试，您可以选择调试 | 停止调试，或按下*Shift* + *F5*。

您还可以在不调试的情况下运行应用程序。要这样做，请从主菜单中选择调试 | 启动无调试，或按下*Ctrl* + *F5*。

如前所述，有各种调试技术，但让我们从基于断点的调试开始，因为这是提供巨大机会的最常见方法之一。您可以在代码的任何行中放置**断点**。程序将在达到该行之前停止执行。然后，您可以查看特定变量的值，以检查应用程序是否按预期工作。

要添加断点，您可以单击左边的边距（在应放置断点的行旁边）或将光标放在应添加断点的行上，并按下*F9*键。在这两种情况下，将显示红色圆圈，并且给定行的代码将标有红色背景，如下截图中的第 17 行所示：

![](img/eb41bd4a-e0b2-4398-a76c-c5790e103123.png)

当执行程序时到达带有断点的行时，程序将停止，并且该行将标有黄色背景，边距图标也会更改，如截图中的第 15 行所示。现在，您可以通过简单地将光标移动到其名称上来检查变量的值。当前值将显示在工具提示中。

您还可以单击位于工具提示右侧的图钉图标，将其固定在编辑器中。然后，该值将在不必移动光标到变量名称上的情况下可见。一旦值发生变化，该值将自动刷新。结果如下截图所示。

IDE 可以根据当前执行的操作调整其外观和功能。例如，在调试时，您可以访问一些特殊的窗口，例如 Locals、Call Stack 和 Diagnostic Tools。第一个显示可用的本地变量及其类型和值。Call Stack 窗口显示有关以下调用方法的信息。最后一个（即 Diagnostic Tools）显示有关内存和 CPU 使用情况以及事件的信息。

此外，IDE 支持条件断点，仅当关联的布尔表达式计算为`true`时才停止程序的执行。您可以通过选择上下文菜单中的 Conditions 选项来为给定的断点添加条件，该菜单在右键单击左侧边栏中的断点图标后显示。然后，断点设置窗口将出现，在那里您应该勾选条件复选框并指定条件表达式，例如在以下屏幕截图中显示的表达式。在示例中，只有当`count`变量的值大于`5`时，即`count > 5`时，执行才会停止：

![](img/b2228230-44fd-477c-bebc-3be5145fc5fd.png)

当执行停止时，您可以使用逐步调试技术。要将程序的执行移动到下一行（而不是加入另一个断点），您可以单击主工具栏中的 Step Over 图标，或按*F10*。如果要进入在执行停止的行中调用的方法，只需单击 Step Into 按钮或按*F11*。当然，您也可以通过单击 Continue 按钮或按*F5*来转到下一个断点。

IDE 中的下一个有趣功能称为 Immediate Window。它允许开发人员在程序执行停止时使用变量的当前值执行各种表达式。您只需在 Immediate Window 中输入表达式，然后按*Enter*键。示例如下屏幕截图所示：

![](img/a8826820-8d88-47c8-95ab-b75584792e00.png)

在这里，通过执行`table.ToLower()`返回表号的小写版本。然后，计算并显示当前日期和`dateTime`变量之间的总分钟数。

# 摘要

这只是本书的第一章，但它包含了很多信息，在阅读剩下的章节时将会很有用。一开始，您看到使用适当的数据结构和算法并不是一件容易的事，但可能会对开发解决方案的性能产生重大影响。然后，简要介绍了 C#编程语言，重点介绍了各种数据类型，包括值类型和引用类型。还描述了类、接口和委托。

在本章的后续部分，介绍了 IDE 的安装和配置过程。然后，您学习了如何创建一个新项目，并详细描述了其结构。接下来，您看到了如何从标准输入流中读取数据，以及如何将数据写入标准输出流。读取和写入相关的操作也混合在一个示例中。

在本章结束时，您学会了如何运行示例程序，以及如何使用断点和逐步调试来找到问题的根源。此外，您还了解了 Immediate Window 功能的可能性。

介绍完毕后，您应该准备继续下一章，了解如何使用数组和列表，以及相关的算法。让我们开始吧！
