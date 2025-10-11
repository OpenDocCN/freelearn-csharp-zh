# *第三章*：介绍 C# 10

C# 是一种优雅且类型安全的面向对象编程语言，它允许开发者构建各种安全且健壮的应用程序，这些应用程序在 .NET 生态系统中运行，并在 GitHub 发布的流行编程语言排行榜上名列前茅。

C# 最初由微软的 Anders Hejlsberg 开发，作为 .NET 创新计划的一部分。自 2002 年 1 月首次发布以来，该语言一直持续添加新功能，以提高性能和生产力。

C# 10 与 .NET 6 一起发布，带来了一些酷炫的新语言特性，以及对早期版本中发布特性的增强，从而提高了开发者的生产力。在本章中，我们将探讨一些新的 C# 语言特性：

+   使用指令的简化

+   记录结构体

+   Lambda 表达式的改进

+   插值字符串的增强

+   扩展属性模式

+   调用者参数属性的添加

到本章结束时，你将熟悉 C# 10 的主要新增功能。此外，本章将帮助我们提升技能，以便用 C# 构建我们的下一个企业级应用程序。

# 技术要求

为了理解本章的概念，你需要以下内容：

+   配备 .NET 6.0 运行的 Visual Studio 2022 版本 17.0 社区版

+   对 Microsoft .NET 的基本理解

本章使用的代码可以在 [`github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition/tree/main/Chapter03`](https://github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition/tree/main/Chapter03) 找到。

# 使用指令的简化

顶级语句是 C# 9.0 中引入的新特性，它使得开发者能够轻松移除仪式性代码。Visual Studio 2022 中的项目模板采用了 C# 中引入的语言变化，如顶级语句。例如，如果你创建一个 `Console` 应用程序，你将看到 `Program.cs` 文件包含以下代码片段：

```cs
// See https://aka.ms/new-console-template for more //information
```

```cs
Console.WriteLine("Hello, World!");
```

前面的代码包含了一个没有仪式性代码（如类定义和 `main` 方法）的 `console` 应用程序模板。通过删除冗余代码，这简化了我们能够编写的行数。

C# 10 中引入的 `implicit` 使用指令和 `global` 使用指令的概念减少了每个 CS 文件中 `using` 语句的重复。

## 全局使用指令

使用全局 `using` 指令，我们不需要在所有 `.cs` 文件中重复 `namespace using` 语句。`global` 关键字用于标记全局 `using` 指令，如下面的代码片段所示：

```cs
global using System.Threading;
```

在前面的代码中，我们将 `System.Threading` 标记为 `global`。现在，我们可以在 `.cs` 文件的开头使用 `using` 指令来引用 `System.Threading` 下的类型。

我们还可以创建对命名空间的 `global` 别名，以解决命名空间冲突，如下面的代码片段所示：

```cs
global using SRS = System.Runtime.Serialization;
```

通过定义这一点，我们可以使用别名 `SRS` 来引用 `System.Runtime.Serialization` 下定义的所有类。我们还可以定义一个全局 `using static` 指令，如下所示：

```cs
global using static System.Console;
```

因此，我们可以直接使用 `System.Console` 类中定义的所有 `static` 函数，而无需引用类名。例如，要向控制台写入一行，我们只需调用 `WriteLine` 方法，而无需引用 `Console` 类名，如下所示：

```cs
WriteLine("Hello C# 10");
```

我们可以在项目的任何 `.cs` 文件中指定全局 `using` 指令。唯一的约束是它们应该出现在任何常规文件作用域的 `using` 指令之前。开发者中常见的做法是创建一个名为 `GlobalUsings.cs` 的 `.cs` 文件，并在该文件中添加全局 `using` 指令。这样，当我们需要添加或删除全局 `using` 指令时，可以限制更改仅限于单个文件。全局 `using` 指令的作用域是当前项目的工作单元。

## 隐式 `using` 指令

在 C# 中，我们发现一些框架命名空间，如 `System` 和 `System.Linq`，几乎存在于所有类中。在 C# 10 中，这些常用命名空间被隐式添加为全局 `using` 指令。隐式添加的命名空间基于项目目标 SDK，并在以下位置进行文档说明：https://docs.microsoft.com/en-us/dotnet/core/project-sdk/overview#implicit-using-directives。

此外，如果我们希望将任何其他命名空间包含在这些隐式指令中或从预定义的命名空间中删除任何命名空间，我们可以通过向 `.csproj` 文件中添加 `ItemGroup` 来实现，如下所示：

```cs
<ItemGroup>
```

```cs
  <Using Include="System.Threading" />
```

```cs
  <Using Remove="System.IO" />
```

```cs
</ItemGroup>
```

在前面的代码片段中，我们包括了 `System.Threading` 并从隐式 `using` 指令中移除了 `System.IO`。

要完全移除隐式 `using` 指令，我们可以在 `.csproj` 文件中取消选中 `ImplicitUsings` 标志，如下所示：

```cs
<ImplicitUsings>disable</ImplicitUsings>
```

`using` 指令的简化是朝着移除冗余的仪式代码并使 `.cs` 文件中的内容简洁的又一步。

在下一节中，我们将探讨 C# 10 中引入的 `record` 结构体。

# 记录结构体

C# 9 中引入的 *record types* 提供了类型声明，用于创建具有合成方法（用于检查相等性和 `ToString`）的不可变引用类型。C# 10 带来了 *record structs*。在本节中，我们将了解 `record struct` 是什么以及它与 `record class` 的区别。

我们使用 `record` 关键字来声明 `record class`，同样使用相同的 `record` 关键字来声明 `record struct`，如下面的代码所示：

```cs
public record struct Employee(string Name);
```

注意

在 C# 9 中，要声明一个 `record` 类，我们不需要显式使用 `class` 关键字。我们只需指定 `record` 来声明它，如下所示：`public` record `Shape(string Name);`

仅使用 `record` 关键字将继续在 C# 10 中工作以声明 `record` 类，但为了更好的可读性，建议明确指定 `class` 或 `struct`。

`record struct` 提供了与 `record class` 类似的好处，例如以下内容：

+   简化的声明语法

+   值相等性

+   引用语义

+   解构

+   有意义的 `ToString` 输出

让我们通过创建一个示例 `Console` 应用程序并定义一个 `EmployeeRecord` 记录结构体来理解这些内容。将以下代码添加到 `Program.cs` 文件中，该文件使用前面代码片段中定义的 `EmployeeRecord` 记录结构体：

```cs
using static System.Console;
```

```cs
 public record struct EmployeeRecord(string Name);
```

```cs
Employee employee1 = new EmployeeRecord("Suneel", "Kunani");
```

```cs
Employee employee2 = new EmployeeRecord("Suneel", "Kunani");
```

```cs
WriteLine(employee1.ToString());
```

```cs
WriteLine($"HashCode of s1 is :{ employee1.GetHashCode()}");
```

```cs
WriteLine($"HashCode of s2 is :{ employee2.GetHashCode()}");
```

```cs
WriteLine($"Is s1 equals s2 : { employee1 == employee2}");
```

```cs
//deconstruct the fields from the employee object
```

```cs
string firstName;
```

```cs
 (firstname, var lastname) = employee1;
```

```cs
Console.WriteLine($"firstname: {firstname}, lastname:{lastname}");
```

在前面的代码中，我们使用简化的声明语法创建了两个 `EmployeeRecord` 记录结构体实例，并使用名称字段值，然后打印实例对象的哈希码，并检查相等性。在这里，我们还在从 `employee` 对象中解构字段。

当我们运行代码时，我们看到如下截图所示的输出：

![Figure 3.1 – 记录结构体示例的输出![Figure 3.1 – 图 3.1](img/Figure_3.1_B18507.jpg)

图 3.1 – 记录结构体示例的输出

当我们查看输出时，我们观察到 `ToString` 被重写以包含实例的内容。正如预期的那样，两个实例的哈希码相同，因为字段值相同。在常规结构体中，对象的哈希码是基于第一个非空字段生成的。在 `record` 结构体中，`GetHashCode` 方法也被重写以包含所有字段以生成哈希码。

`record` 结构体综合了 `IEquatable<T>` 接口的实现。它还实现了 `==` 和 `!=` 操作符。常规结构体默认没有实现这些操作符。常规结构体从 `ValueType` 继承了 `Equals` 方法，该方法使用反射来进行相等性检查。因此，`record` 结构体中的综合相等性检查性能更优。`record` 结构体还综合了 `Deconstruct` 方法来填充对象外的字段。如果你仔细查看解构代码，你会注意到变量的混合声明。我们在解构过程中声明了 `lastName`，而在前面的语句中声明了 `firstName`。这种变量的混合声明仅在 C#10 及以上版本中是可能的。

当我们在反汇编工具（如 ILSpy 或 Reflector）中反汇编代码时，我们看到生成的代码如下所示：

![Figure 3.2 – 员工类生成的代码![Figure 3.2 – 图 3.2](img/Figure_3.2_B18507.jpg)

图 3.2 – 员工类生成的代码

注意

你可以从 [`marketplace.visualstudio.com/items?itemName=SharpDevelopTeam.ILSpy`](https://marketplace.visualstudio.com/items?itemName=SharpDevelopTeam.ILSpy) 安装 ILSpy。

如果我们仔细查看 `Employee` 类型的定义，我们可以看到 C# 编译器为 `record struct` 类型合成的所有管道。从这一点，我们可以理解 `record` 结构体基本上是一个实现了 `IEquatable` 接口并重写了 `GetHashCode` 和 `ToString` 方法的结构体。您可以重写 `ToString` 方法以创建记录类型的自定义字符串表示形式。从 C# 10 开始，您还可以将 `ToString` 重写标记为 sealed，这可以防止编译器合成 `ToString` 方法或派生类型重写它。在基记录类型中密封 `ToString` 方法确保了所有派生类型的字符串表示形式的一致性。编译器还提供了 `Deconstruct` 方法，它用于将 `record` 结构体分解为其组件属性。与 `record` 类不同，`record` 结构体是可变的。要使 `record` 结构体不可变，我们可以在声明中添加 `readonly` 修饰符：

```cs
public readonly record struct Employee(string Name);
```

要更改 `readonly record` 结构体的字段，我们可以使用如下所示的运算符，就像使用 `record` 类一样：

```cs
Employee employee2 = employee1 with { LastName = string.Empty };
```

在本节中，我们学习了 C# 10 中引入的 `record` 结构体以及它与 `record` 类和常规结构体的比较。在下一节中，我们将学习 Lambda 表达式的改进。

# Lambda 表达式的改进

Lambda 表达式是一种表示匿名方法的方式。它允许我们内联定义方法实现。

可以从任何 Lambda 表达式创建委托类型。Lambda 表达式的参数类型和返回值类型决定了它可以转换成的委托类型。如果 Lambda 表达式不返回值，则可以将其更改为 `Action` 委托类型；否则，它可以转换为 `Func` 委托类型之一。在本节中，我们将学习 C# 10 带给 Lambda 表达式的改进。

## 推断表达式类型

如果参数的类型是显式的并且可以推断出返回类型，则 C# 语言编译器现在将推断表达式类型。例如，考虑以下代码片段，其中我们定义了一个 Lambda 表达式来查找给定整数的平方：

```cs
Var Square = (int x) => x * x;
```

在前面的代码中，参数 `x` 的类型指定为 `int`，返回类型从表达式中推断为 `int`。如果我们将鼠标悬停在 Visual Studio 中的 `var` 上，我们可以看到 `Square` Lambda 表达式的推断类型，如下一张截图所示，它使用 `Func` 委托：

![图 3.3 – Square 表达式的推断类型]

![图片](img/Figure_3.3_B18507.jpg)

图 3.3 – Square 表达式的推断类型

对于这里显示的代码，编译器将使用 `Action` 委托，因为表达式的返回类型是 `void`：

```cs
var SayHello = (string name) => Console.WriteLine($"Hello {name}");
```

如果合适，推断的类型将使用 `Func` 或 `Action` 委托。否则，编译器将合成一个委托类型，例如，如果 Lambda 表达式接受 `ref` 类型，如下面的代码片段所示：

```cs
Var SayWelcome = (ref string name) => Console.WriteLine($"Welcome {name}");
```

之前表达式的合成类型将是一个匿名委托类型。

编译器将尝试根据表达式推断返回类型。有时，可能无法推断类型。如果无法推断类型信息，我们将得到编译错误。

## Lambda 表达式的返回类型

在编译器无法推断返回类型的情况下，我们可以在 C# 10 中显式指定。考虑以下代码片段，其中我们定义了从 `Person` 记录类继承的 `Employee` 和 `Manager` 记录类：

```cs
public record class Person();
```

```cs
public record class Employee() : Person();
```

```cs
public record class Manager() : Person();
```

```cs
var createExpression = (bool condition) => condition ? new Employee() : new Manager();
```

上一段代码片段中的 `createExpression` 术语根据传入的条件创建 `Employee` 或 `Manager` 类型的实例。在这种情况下，编译器无法推断返回类型，这将导致编译错误。从 C#10 开始，我们可以显式指定 Lambda 表达式的返回类型，如下面的代码所示：

```cs
var createEmployee = Person (bool hasReportees) => condition ? new Manager() : new Employee();
```

```cs
// Create the Person object based on condition
```

```cs
var manager = createEmployee(true);
```

上一段代码推断的表达式类型是 `Func<bool, Person>`。

## 向 Lambda 表达式添加属性

从 C# 10 开始，我们可以向 Lambda 表达式及其参数和返回类型添加属性。以下代码片段定义了一个 Lambda 表达式，用于检索给定 ID 的员工：

```cs
var GetEmployeeById =  [Authorize] Employee ([FromRoute]int id) => { return new Employee { }; };
```

`GetEmployeeeById` 表达式具有 `[Authorize]` 属性，而 `id` 参数带有 `[FromRoute]` 属性。

Lambda 表达式上的属性在调用时没有任何效果，因为调用是通过底层委托类型进行的。Lambda 表达式上定义的属性可以通过反射发现。

ASP.NET 6.0 中引入的最小 API 是这些改进背后的驱动力之一。我们将在 *第十章* 中看到它的用法，即 *创建一个 ASP.NET Core 6 Web API*。

在本节中，我们学习了 Lambda 表达式的改进；在下一节中，我们将看到对插值字符串的改进。

# 对插值字符串的改进

几乎每个应用程序都会有一些文本处理的需求。在 .NET 中，有许多字符串操作的方法可用，例如 `string` 原始类型、`StringBuilder`、类型上的 `ToString` 覆盖、字符串连接以及 `string.Format`，后者提供了从复合格式字符串构建字符串的功能。`String.Format` 接收一个格式字符串和格式项作为输入，并生成如以下代码所示的格式化字符串：

```cs
string message = string.Format("{0}, {1}!", Greeting, Message);
```

在之前的代码中，格式字符串中的 `{0}` 和 `{1}` 位置将被分别传递的 `Greeting` 和 `Message` 格式项填充。为了使其更友好和易于阅读，C# 6 添加了一种新的语言语法，称为 **插值字符串**，如下面的代码片段所示：

```cs
string Greeting = "Hello";
```

```cs
string Language = "C#";
```

```cs
int version = 10;
```

```cs
string message = $"{Greeting}, {Language}!";
```

```cs
string messageWithVersion = $"{Greeting}, {Language} {version}!";
```

当使用插值字符串语法时，.NET 编译器生成最适合插值字符串以产生相同结果的代码。

使用反汇编器，如 ILSpy 或 SharpLab，查看之前代码片段生成的代码；它将类似于以下代码片段：

```cs
String text = "Hello";
```

```cs
string text2 = "C#";
```

```cs
int num = 10;
```

```cs
string text3 = text + ", " + text2 + "!";
```

```cs
string text4 = string.Format("{0}, {1} {2}!", text, text2, num);
```

注意

[`sharplab.io/`](https://sharplab.io/) 是一个 .NET 代码沙盒，它显示了代码编译的中间结果。

对于 `message` 插值字符串，代码是通过连接生成的。对于第二个字符串 `messageWithVersion`，其中涉及非字符串字面量，生成的代码使用 `string.Format`。

编译器做了它打算做的事情，但它有几个问题，其中代码是通过使用 `string.Format` 生成的：

+   编译器解析了插值字符串以生成使用 `string.Format` 的代码。相同的字符串也必须由 .NET 运行时解析以找到字面量位置。

+   `string.Format` 方法中字面量的参数类型是 `Object`。因此，在 `string.Format` 中使用的任何值类型都会涉及装箱。

+   `string.Format` 的重载最多接受三个参数。超过三个参数由接受 `params object[]` 的重载提供支持。因此，超过三个参数需要实例化一个数组。

+   由于 `string.Format` 只接受 `Object` 类型，因此我们不能使用 `ref struct` 类型，如 `Span<T>` 和 `ReadOnlySpan<char>`。

+   由于 `ToString` 是在捕获的表达式上被调用的，因此会创建多个临时字符串。

这里提到的所有缺点都将通过 C# 10 生成一系列追加到字符串构建器的代码来解决。对于之前讨论的相同代码，如果你查看 C# 10 中生成的代码，它将使用 `DefaultInterpolatedStringHandler`，如下面的代码片段所示：

```cs
string Greeting = "Hello";
```

```cs
string Language = "C#";
```

```cs
int version = 10;
```

```cs
string message = Greeting + ", " + Language + "!";
```

```cs
DefaultInterpolatedStringHandler defaultInterpolatedStringHandler = new DefaultInterpolatedStringHandler(4, 3);
```

```cs
defaultInterpolatedStringHandler.AppendFormatted(Greeting);
```

```cs
defaultInterpolatedStringHandler.AppendLiteral(", ");
```

```cs
defaultInterpolatedStringHandler.AppendFormatted(Language);
```

```cs
defaultInterpolatedStringHandler.AppendLiteral(" ");
```

```cs
defaultInterpolatedStringHandler.AppendFormatted(version);
```

```cs
defaultInterpolatedStringHandler.AppendLiteral("!");
```

```cs
string messageWithVersion = defaultInterpolatedStringHandler.ToStringAndClear();
```

对于插值字符串，C# 10 编译器现在使用 `DefaultInterpolatedStringHandler` 而不是 `string.Format`。在之前生成的代码中，`DefaultInterpolatedStringHandler` 是通过传递两个参数构建的，即插值字符串字面部分中的字符数和要填充的字符串中的位置数。通过调用 `AppendLiteral` 或 `AppendFormatted` 来分别追加字面量或格式化字符串。随着插值字符串处理器的引入，之前讨论的问题得到了解决。

对于用 C# 早期版本编写的相同插值字符串代码，在 C# 10 中将会有性能提升。我们还可以构建我们自己的自定义插值字符串处理器，这在数据不打算用作字符串或条件执行对目标方法来说是逻辑上的合适选择的情况下可能很有用。

在本节中，我们学习了插值字符串的改进，它使我们比早期版本有更好的性能。在下一节中，让我们学习扩展属性模式。

# 扩展属性模式

模式匹配是一种检查对象值或具有完整或部分匹配到序列的属性值的方法。这在 C# 中以 `if…else` 和 `switch…case` 语句的形式得到支持。在现代语言中，尤其是在像 F# 这样的函数式编程语言中，有对模式匹配的高级支持。从 C# 7.0 开始，引入了新的模式匹配概念。模式匹配提供了一种不同的方式来表达条件，以使代码更具可读性。自 C# 7 引入以来，模式匹配在每次主要版本发布时都得到了扩展。

在本节中，让我们学习 C# 10 中引入的扩展属性模式。

考虑以下代码片段：

```cs
Product product = new Product
```

```cs
{
```

```cs
    Name ="Men's Shirt",
```

```cs
    Price =10.0m,
```

```cs
    Location = new Address
```

```cs
    {
```

```cs
        Country ="USA",
```

```cs
        State ="NY"
```

```cs
    }
```

```cs
};
```

在这个代码片段中，我们有一个 `Product` 类型的对象，它包含产品的产地位置。在 C# 10 之前，如果我们想检查这个产品的原产国是否为美国，我们会做类似于以下代码片段的事情：

```cs
if (product is Product { Location: { Country: "USA" } })
```

```cs
    Console.WriteLine("USA"); 
```

在 C# 10 中，我们可以访问扩展属性以使其更具可读性，如下面的代码片段所示：

```cs
if (product is Product { Location.Country : "USA"  })
```

```cs
    Console.WriteLine("USA");
```

在前面的代码中，我们正在使用扩展属性模式验证 `Location` 的 `Country` 属性。

在本节中，我们学习了关于扩展属性模式的内容。让我们在下一节中学习 `caller` 参数属性的新增内容。

# 添加到调用者参数属性

C# 5 首次引入了 `caller` 参数属性。它们是 `CallerMemberName`、`CallerFilePath` 和 `CallerLineNumber`。这些属性使得编译器在生成的代码中填充方法参数。它们在各种场景中使用，例如在 MVVM 模式下触发 `OnNotifyPropertyChanged` 事件时填充更多的调试跟踪数据。例如，考虑以下代码片段，它定义了一个 `Gift` 模型：

```cs
public class Gift : INotifyPropertyChanged
```

```cs
{
```

```cs
    private string _description;
```

```cs
    public string Description
```

```cs
    {
```

```cs
        get
```

```cs
        {
```

```cs
            return _description;
```

```cs
        }
```

```cs
        set
```

```cs
        {
```

```cs
            _description = value;
```

```cs
            OnPropertyRaised();
```

```cs
        }
```

```cs
    }
```

```cs
    public event PropertyChangedEventHandler 
```

```cs
      PropertyChanged;
```

```cs
    private void OnPropertyRaised([CallerMemberName] string 
```

```cs
      propertyname="")
```

```cs
    {
```

```cs
        if (PropertyChanged != null)
```

```cs
        {
```

```cs
            PropertyChanged(this, new 
```

```cs
              PropertyChangedEventArgs(propertyname));
```

```cs
        }
```

```cs
    }
```

```cs
} 
```

在前面的 `Gift` 类定义中，每当 `Description` 属性的设置器被调用时，都会调用 `OnPropertyChanged` 方法。在 `OnPropertyChanged` 方法实现中，我们有一个带有 `CallerMemberName` 属性的 `propertyName` 参数。这将使编译器生成如下所示的设置器：

```cs
public string Description
```

```cs
{
```

```cs
    get { return _description;     }
```

```cs
    set
```

```cs
    {
```

```cs
        _description = value;
```

```cs
        OnPropertyRaised("Description");
```

```cs
    }
```

```cs
}
```

在这段生成的代码中，编译器自动填充了 `OnProperyChanged` 的参数，即属性名 `Description`。这对于开发者来说是一个方便的功能，有助于编写无错误的代码。其他两个 `caller` 参数属性 `CallerFilePath` 和 `CallerLineNumber` 分别填充调用方法的文件路径和行号。

`CallerArgumentExpression` 是 C# 10 中新增的功能之一。正如其名所示，该属性使编译器自动填充参数表达式。让我们构建一个简单的参数验证辅助类，该类对传递的参数执行 `null` 检查。考虑以下 `ArgumentValidation` 类的实现，它实现了一个辅助方法，如果参数值为 `null`，则抛出 `ArgumentException`：

```cs
public static class ArgumentValidation
```

```cs
{
```

```cs
    public static void ThrowIfNull<T>(T value,
```

```cs
    [CallerArgumentExpression("value")] string expression = 
```

```cs
      null) where T : class
```

```cs
    {
```

```cs
        if (value == null)
```

```cs
            Throw(expression);
```

```cs
    }
```

```cs
    private static void Throw(string expression)
```

```cs
        => throw new ArgumentException($"Argument 
```

```cs
           {expression} must not be null");
```

```cs
} 
```

在 `ThrowIfNull` 方法中，我们执行 `null` 检查，并使用从 `CallerArgumentExpression` 中选择的参数名抛出 `ArgumentException`。我们可以使用前面的辅助类对传递给方法的参数执行 `null` 检查。例如，考虑以下方法，该方法将传入的产品添加到购物车中：

```cs
public async Task<ProductDetailsViewModel> AddProductAsync (ProductDetailsViewModel product)
```

```cs
{
```

```cs
    ArgumentValidation.ThrowIfNull(product);
```

```cs
    // Implementation to add the product to cart
```

```cs
}
```

在此方法中，我们使用 `ArgumentValidation` 辅助类来检查 `product` 参数的 `null` 条件。调用 `ThrowIfNull` 辅助方法的生成代码将是 `ArgumentValidation.ThrowIfNull(product, "product");`。

编译器自动填充了字符串参数中的参数名。调用参数将在我们想要向跟踪添加更多详细信息的地方很有用，这将有助于解决问题。

# 摘要

在本章中，我们学习了 C# 10 版本中语言特性的主要新增功能。我们看到了 C# 10 如何简化使用 `implicit` 和 `global` 使用指令编写的代码。我们了解了 `record` 结构体以及它们与 C# 9 中引入的 `record` 类的比较。我们还学习了 Lambda 表达式、表达式类型推断以及显式指定表达式返回类型的改进。我们还看到了字符串插值的性能提升。我们还学习了如何使用 `CallerArgumentExpression` 属性构建抛出辅助器。

通过本章，我们获得了利用 C# 10 新特性的技能，这些特性将在接下来的章节中构建的企业电子商务应用中使用。除此之外，还有一些其他的小增强。您可以参考 C# 语言文档以了解更多信息：[`docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10`](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-10)。在实现我们的电子商务应用的不同功能的同时，我们将在这本书中突出显示 C# 10 和 .NET 6 的新特性。

在接下来的部分，我们将学习构成我们电子商务应用程序构建块的一些横切关注点。

# 问题

阅读本章后，你应该能够回答以下问题：

1.  正确或错误？记录结构体是可变的：

a. 正确

b. 错误

**答案：a**

1.  你应该使用哪个关键字来使记录结构体不可变？

a. 记录结构体默认是不可变的。

b. `readonly`.

c. `finally`.

d. `sealed`.

**答案：b**

1.  正确或错误？编译器将在所有场景中推断类型表达式：

a. 正确

b. 错误

**答案：b**

1.  在哪个版本的 C# 中首次引入了调用者参数属性？

a. C# 9

b. C# 8

c. C# 5

d. C# 7

**答案：c**
