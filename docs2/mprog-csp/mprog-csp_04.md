

# 使用反射进行类型推理

现在我们已经介绍了一些元编程如何为你带来好处的基础知识，其核心概念以及一个真实世界的例子，现在是时候深入.NET 运行时，看看我们如何利用其力量。

在本章中，我们将探讨编译器和运行时提供的隐式元数据。我们将了解如何收集运行系统中的所有类型，并用于发现。

我们将涵盖以下主题：

+   运行过程中的程序集发现

+   利用库元数据获取项目引用的程序集

+   发现类型

+   应用开放/封闭原则

到本章结束时，你将了解如何使用.NET 运行时的强大 API 来推理系统中已存在的类型。你将学习如何将其应用于创建一个更具弹性和动态的代码库，它可以增长并确保其可维护性得到保留。

# 技术要求

本章的特定源代码可以在 GitHub 上找到（[`github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Chapter4`](https://github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Chapter4)），并且基于[`github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Fundamentals`](https://github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Fundamentals)中找到的**基础知识**代码。

# 运行过程中的程序集发现

在.NET 中，我们编译的所有内容最终都会放入一个称为**程序集**的容器中。这是以**中间语言**（**IL**）的形式存储编译代码的二进制文件。与此并行，还有一些元数据与之相关，用于标识类型、方法、属性、字段以及任何其他符号和元数据。

所有这些工件都可以通过 API 发现，一切始于**System.Reflection**命名空间。在这个命名空间中，有一些 API 允许你反射正在运行的内容。在其他语言中，这通常被称为**内省**。

观察任何类型的任何实例，你会发现总有一个可以调用的**GetType()**方法。这是反射能力的一部分，并在**Object**的基类型中实现，所有类型都隐式继承自该类型。

**GetType()**方法返回一个**Type**类型，它描述了特定类型的特性——其字段、属性、方法等。它还包含有关它可能实现或继承自的基类型的信息。这些是非常强大的结构，我们将在本章后面利用它们。

我们不妨从单个类型开始，退一步看看，从没有任何类型开始，我们如何发现我们在运行过程中的内容。

## 程序集

**System.Reflection**命名空间中的一个类型是**Assembly**。这是表示程序集的类型，通常是包含 IL 代码的**.dll**文件。在**Assembly**类型上，有几个静态方法。我们想要关注一个特别的方法：**.GetEntryAssembly()**。此方法允许我们在任何时间获取作为起始点的程序集，即.NET 运行时调用以启动我们的应用程序的入口点：

1.  让我们创建一个名为**Chapter4**的文件夹。对于这一章，我们将创建几个项目，所以让我们先创建一个名为**AssemblyDiscovery**的另一个文件夹。

1.  在您的命令行界面中切换到这个文件夹，并创建一个新的控制台项目：

    ```cs
    dotnet new console
    ```

1.  这将设置项目所需的必要工件。打开**Program.cs**文件，并用以下内容替换其内容：

    ```cs
    using System.Reflection;
    var assembly = Assembly.GetEntryAssembly();
      Console.WriteLine(assembly.FullName)
    ```

代码正在访问**Assembly**上的**GetEntryAssembly()**静态方法以获取作为应用程序入口点的程序集。

现在运行此程序，你应该得到一个类似于以下输出的结果：

```cs
AssemblyDiscovery, Version=1.0.0.0, Culture=neutral,
  PublicKeyToken=null
```

这只是打印出程序集的名称、版本、程序集针对的文化信息，以及如果程序集已签名，则其公钥。

从入口程序集，我们现在可以开始查看哪些程序集已被引用。将**Program.cs**中的代码替换为以下内容：

```cs
using System.Reflection;
var assembly = Assembly.GetEntryAssembly();
Console.WriteLine(assembly!.FullName);
var assemblies = assembly!.GetReferencedAssemblies();
var assemblyNames = string.Join(", ", assemblies.Select(_
  => _.Name));
Console.WriteLine(assemblyNames);
```

代码从入口程序集获取引用的程序集并打印出来。您应该看到以下类似的内容：

```cs
AssemblyDiscovery, Version=1.0.0.0, Culture=neutral,
  PublicKeyToken=null
System.Runtime, System.Console, System.Linq
```

由于我们没有显式引用任何程序集，所以我们只看到**System.Runtime**、**System.Consol**和**System.Linq**作为可用的程序集。

这一切都是非常基础的，但展示了关于您的应用程序及其构建方式的推理起点。但我们可以做更多。

# 利用库元数据获取项目引用的程序集

如果您打算跨正在运行的过程收集元数据，那么您可能只对解决方案中的程序集感兴趣，而不是所有的.NET 框架库或第三方库。查找所有程序集以获取元数据会有性能影响，因此过滤下来可能是个好主意。

在.NET 项目中，我们可以添加包引用，通常来自**NuGet**或您自己的包源，但我们也可以添加本地项目引用。这些引用指向其他**.csproj**文件，代表我们希望将其打包到自己的程序集中的内容。在**.csproj**文件中，您可以通过它们的 XML 标签来识别不同的引用 – **<PackageReference/>**或**<ProjectReference/>**。在 Visual Studio 或 Rider 中，您通常会在资源管理器视图中看到这些标签。

**C#**编译器为所有项目产生的不同类型的引用生成额外的元数据。这可以在代码中利用。

## 可重用的基础知识

让我们构建一个可重用的库，我们可以在接下来的章节中利用和扩展它。让我们把这个项目命名为 **Fundamentals**：

1.  在你开始构建章节的根目录下创建一个新的文件夹，命名为 **Fundamentals**。切换到这个文件夹并创建新的项目：

    ```cs
    dotnet new classlib
    ```

这将创建一个新的项目并添加一个名为 **Class1.cs** 的文件。由于我们根本不需要它，所以请将其删除。

1.  为了让我们能够获取到适当的元数据，使我们能够区分包或项目引用，我们需要添加一个 NuGet 包引用。

我们正在寻找的包是 **Microsoft.Extensions.DependencyModel**。在终端中，你可以通过以下步骤添加引用：

```cs
dotnet add package
  Microsoft.Extensions.DependencyModel
```

1.  现在我们有了这个包，我们想要创建一个能够发现项目引用和任何我们明确告诉它加载的额外包引用的系统。从这些中，我们想要获取所有类型，这样我们就可以开始对我们系统进行一些严肃的探索。

首先，创建一个名为 **Types.cs** 的文件。然后向其中添加以下代码：

```cs
namespace Fundamentals;
public class Types
{
}
```

在这个新类中，我们希望能够公开所有发现的类型。在我们能够做到这一点之前，我们需要发现它们。

1.  在 **Types.cs** 文件的命名空间声明之前，在顶部添加以下 **using** 语句：

    ```cs
    using System.Reflection;
    using Microsoft.Extensions.DependencyModel;
    ```

1.  添加一个名为 **DiscoverAllTypes()** 的私有方法，并首先让它看起来如下：

    ```cs
    IEnumerable<Type> DiscoverAllTypes()
    {
        var entryAssembly = Assembly.GetEntryAssembly();
        var dependencyModel =
          DependencyContext.Load(entryAssembly);
        var projectReferencedAssemblies =
          dependencyModel.RuntimeLibraries
                            .Where(_ => _.Type.Equals
                              ("project"))
                            .Select(_ => Assembly.Load
                              (_.Name))
                            .ToArray();
    }
    ```

这段代码将获取入口程序集，然后利用你引用的包中的 **DependencyContext**。这个模型包含有关程序集（或称为在 **DependencyModel** 扩展中称为库）的更多元数据。我们最后要做的就是加载我们找到的程序集。大多数情况下，程序集已经加载。但这将保证它们被加载，并返回一个程序集的实例。如果它已经加载，则不会再次加载它。

这样做结果是我们在一个数组中拥有所有项目引用的程序集。但这可能不是你想要的全部。例如，如果你在维护一套在组织内部共享的通用包，并且想从这些包中查找，一个常见的模式是将组织名称作为程序集名称的一部分，通常在开头。这意味着很容易添加包含以特定字符串为前缀的所有程序集的内容。

1.  将 **DiscoverAllTypes()** 方法的签名更改为能够包含一个用于收集用于发现的程序集前缀的集合：

    ```cs
    IEnumerable<Type> DiscoverAllTypes(IEnumerable<string>
      assemblyPrefixesToInclude)
    {
        // ... leave the content of this method as before
    }
    ```

1.  在 **DiscoverAllTypes()** 方法中，在末尾添加以下内容：

    ```cs
    var assemblies = dependencyModel.RuntimeLibraries
                        .Where(_ =>
                      _.RuntimeAssemblyGroups.Count > 0 &&
                        assemblyPrefixesToInclude.Any(asm
                        => _.Name.StartsWith(asm)))
                        .Select(_ =>
                        {
                            try
                            {
                                return Assembly.Load
                                  (_.Name);
                            }
                            catch
                            {
                                return null!;
                            }
                        })
                        .Where(_ => _ is not null)
                        .Distinct()
                        .ToList();
    ```

代码检查相同的 **RuntimeLibraries**，但这次是针对以特定前缀开始的程序集。这里也有错误处理功能：一些程序集无法使用常规的 **Assembly.Load()** 方法加载，所以我们只是忽略它们，因为它们对我们没有兴趣。

由于我们在 LINQ 查询的末尾调用了**.ToList()**，我们可以轻松地将两个程序集集合和所有加载的程序集中的所有类型组合起来，并从**DiscoverAllTypes()**方法返回它们。在**DiscoverAllTypes()**方法的末尾添加以下内容：

```cs
assemblies.AddRange(projectReferencedAssemblies);
return assemblies.SelectMany(_ => _.GetTypes())
  .ToArray();
```

1.  现在我们已经收集了所有程序集及其类型并返回了它们，现在是时候将它们暴露给外部世界了。我们通过添加一个返回**IEnumerable**类型**Type**的公共属性来实现这一点。我们不希望它从外部被潜在地修改：

    ```cs
    public IEnumerable<Type> All { get; }
    ```

1.  **Types**类的最后一部分是添加一个构造函数。我们希望构造函数能够接受**DiscoverAllTypes()**方法的任何程序集前缀：

    ```cs
    public Types(params string[]
        assemblyPrefixesToInclude)
    {
        All = DiscoverAllTypes(assemblyPrefixesToInclude);
    }
    ```

我们现在封装了一个类型注册表，我们可以利用它来处理其他用例。现在我们有一些基本的构建块。让我们继续创建一个多项目应用程序。

## 商业应用程序

现在我们已经有一些基本的构建块。让我们继续创建一个多项目应用程序。

在**Chapter4**文件夹中，创建一个名为**BusinessApp**的文件夹。在这个文件夹中，我们将创建多个项目，这些项目将构成应用程序。在**BusinessApp**文件夹中，创建一个名为**Domain**的文件夹和一个名为**Main**的文件夹。**Domain**文件夹将代表我们应用程序的领域逻辑，应该是一个类库项目。在**Domain**文件夹中创建一个新的.NET 项目：

```cs
dotnet new classlib
```

删除生成的**Class1.cs**文件。然后，在**Main**文件夹中，我们想要创建一个控制台应用程序：

```cs
dotnet new console
```

在**Main**项目中，你现在应该添加对**Fundamentals**项目和**Domain**项目的引用：

```cs
dotnet add reference ../../../Fundamentals/Fundamentals.csproj
dotnet add reference ../Domain/Domain.csproj
```

在**Program.cs**文件中，在**Main**项目中，将内容替换为以下内容：

```cs
using Fundamentals;
var types = new Types();
var typeNames = string.Join("\n", types.All.Select(_ =>
  _.Name));
Console.WriteLine(typeNames);
```

这段代码现在利用了我们放入**Fundamentals**中的**Types**类，该类将查看所有项目引用程序集，并给我们所有类型。运行此代码应该会得到类似以下输出：

```cs
Program
<>c
EmbeddedAttribute
NullableAttribute
NullableContextAttribute
EmbeddedAttribute
NullableAttribute
NullableContextAttribute
Types
```

你会注意到其中一些类型根本不是你创建的。这些是 C#编译器放入项目的类型。

前往**Domain**项目，在其中添加一个名为**Employees**的文件夹，并在其中创建一个名为**RegisterEmployee.cs**的文件，并添加以下内容：

```cs
namespace Domain.Employees;
public class RegisterEmployee
{
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string SocialSecurityNumber { get; set; } =
      string.Empty;
}
```

再次运行**Main**项目后，你现在应该看到添加的类型：

```cs
Program
<>c
EmbeddedAttribute
NullableAttribute
NullableContextAttribute
RegisterEmployee   <-- New
EmbeddedAttribute
NullableAttribute
NullableContextAttribute
Types
```

这样，我们就为使用类型打下了坚实的基础。我们只需要提高其功能，使其更加有用。

# 类型发现

通过**Types**类，我们现在有一种原始的方法来获取运行系统中的所有类型。这本身就可以非常有用，因为你现在可以使用它来执行**语言集成查询**（**LINQ**）查询，以找到符合你感兴趣的具体标准的类型。

在代码中，一个非常常见的场景是我们有基类或接口，它们代表了我们系统中已知工件类型的特征，并且我们创建了专门的版本来覆盖虚拟或抽象方法，或者只是实现一个特定的接口来表示它是什么。这是面向对象编程语言的力量。

通过这种方式，我们向我们的系统中添加了额外的隐式元数据，我们可以利用这些数据。例如，在 *第三章*，*通过现有真实世界示例去神秘化* 中，我们探讨了 ASP.NET 是如何通过发现所有以 **Controller** 为基类的类来实现这一点的。

根据我的经验，这是我在进行这些项目时使用最多的典型模式。事实上，它如此普遍，以至于优化类型查找以避免基于继承查找时的复杂性是一个好主意。优化的查找将显著提高性能。

完整地解释完整的查找缓存机制可能会有些冗长，因为其中涉及许多动态部分。你可以在以下链接中找到完整的代码：[`github.com/PacktPublishing/Metaprogramming-in-C-Sharp/blob/main/Fundamentals/ContractToImplementorsMap.cs`](https://github.com/PacktPublishing/Metaprogramming-in-C-Sharp/blob/main/Fundamentals/ContractToImplementorsMap.cs).

**ContractToImplementorsMap** 所表示的接口如下：

```cs
public interface IContractToImplementorsMap
{
    IDictionary<Type, IEnumerable<Type>>
      ContractsAndImplementors { get; }
    IEnumerable<Type> All { get; }
    void Feed(IEnumerable<Type> types);
    IEnumerable<Type> GetImplementorsFor<T>();
    IEnumerable<Type> GetImplementorsFor(Type contract);
}
```

**IContractToImplementorsMap** API 的目的是提供一个快速获取特定类型实现的方法，无论是基类还是接口。**IContractToImplementorsMap** 的实现会接收所有传入的类型，并将这些类型正确映射到这个缓存中。

你会注意到有一个名为 **Feed()** 的方法。我们将在我们的 **Types** 类中调用这个方法。此外，我们还想有一些方法，使发现不同类型变得更加有用；例如，以下方法将很有帮助：

```cs
public Type FindSingle<T>();
public IEnumerable<Type> FindMultiple<T>();
public Type FindSingle(Type type);
public IEnumerable<Type> FindMultiple(Type type);
public Type FindTypeByFullName(string fullName);
```

这些方法允许我们找到实现特定接口或继承自基类型的方法。它们允许我们根据类型（通过泛型参数或通过传递 **Type** 对象）找到单个实例或多个实现者。为了方便起见，还有一个通过名称查找类型的方法。

你可以在本章开头指定的链接中找到 **Fundamentals** 文件夹中的完整实现。

## 回到正题

回到我们的业务示例，让我们在 **Domain** 项目中的 **Employee** 文件夹内添加一个名为 **SetSalaryLevelForEmployee.cs** 的第二个类。让它看起来如下：

```cs
namespace Domain.Employees;
public class SetSalaryLevelForEmployee
{
    public string SocialSecurityNumber { get; set; } =
      string.Empty;
    public decimal SalaryLevel { get; set; }
}
```

**注册员工**和**为员工设置薪资级别**类都代表了我们在领域业务逻辑中需要执行特定操作的数据。这类操作通常被称为命令。如果我们想发现所有的命令，我们可以创建一个空接口，我们可以使用这个接口让所有命令实现，这样我们就可以轻松地发现它们。这种以这种方式使用的空接口通常被称为标记接口。

在**基础**项目中，添加一个名为**ICommand.cs**的文件，并使其看起来如下：

```cs
namespace Fundamentals;
public interface ICommand
{
}
```

现在，我们需要从**域**项目到**基础**项目的项目引用。从**域**项目运行以下命令：

```cs
dotnet add reference ../../../Fundamentals/
  Fundamentals.csproj
```

使用新的**ICommand**接口，我们可以用它标记**注册员工**和**为员工设置薪资级别**命令。打开**注册员工**和**为员工设置薪资级别**的文件，并在每个文件的顶部添加以下**using**语句：

```cs
using Fundamentals;
```

对于这两个类定义，在末尾添加**: ICommand**，使它们实现**ICommand**接口。

打开**Main**项目中的**Program.cs**文件，并使用以下内容进行更改：

```cs
using Fundamentals;
var types = new Types();
var commands = types.FindMultiple<ICommand>();
var typeNames = string.Join("\n", commands.Select(_ => _.Name));
Console.WriteLine(typeNames);
```

运行**Main**项目现在应该得到以下结果：

```cs
SetSalaryLevelForEmployee
RegisterEmployee
```

有效地，我们现在已经复制了必要的基础设施来模仿 ASP.NET 对项目发现的操作。有了这个，我们使我们的软件更容易扩展。

## 领域概念

在大多数编程语言中最无聊的类型是语言提供的原始数据；通常，你的整数、布尔值、字符串等。它们提供绝对没有有趣或有意义的元数据。它们只是非常原始的。

很容易陷入使用这些数据的陷阱，最终导致对原始数据的执着。这不仅会使你失去宝贵的元数据，而且有时还会创建出不清楚的代码，甚至可能因为两个相同原始类型的属性可以互换而容易出错。

对于应用程序，有一个很好的机会通过将原始数据封装到有意义的数据类型中，从而从原始数据中脱离出来，给你的领域带来意义。

这种美在于，你不仅会使你的代码更易于阅读，而且更易于理解，并且减少歧义。这样做可以使你从对原始数据的执着转变为一个地方，在那里如果你做错了什么，你会得到编译器错误，从而让开发者从一开始就站在正确的位置。除此之外，你还会带来大量的元数据，这些数据可以被利用。

原始类型的关键特征是它们是值类型。这意味着你可以有两个相同的值实例，并且相等性检查将表明它们是相同的。在**C# 9.0**中，我们得到了一个新的结构体**record**。这使我们能够创建具有复杂类型但具有与值类型相同特性的类型。比较具有相同值的两个复杂**record**类型将被视为相等。

让我们介绍一种可用于概念的基本类型。在**基础**项目中，添加一个名为**ConceptAs.cs**的文件，并使其看起来如下所示：

```cs
namespace Fundamentals;
public record ConceptAs<T>
{
    public ConceptAs(T value)
    {
        ArgumentNullException.ThrowIfNull(value,
          nameof(value));
        Value = value;
    }
    public T Value { get; init; }
    public static implicit operator T(ConceptAs<T> value)
      => value.Value;
}
```

此实现为你提供了一种封装领域概念的方法。它是使用泛型构建的，允许你指定概念所表示的值的内部类型。它对封装内不允许 null 值执行严格的检查。如果你想在代码中允许 null 值，它不应在概念内，而应在概念的实例上。此外，实现提供了一个便利操作符，用于自动将概念转换为原始类型。

通常，当你与数据库一起工作或在不同格式之间传输数据时，你希望去除概念包装，只获取原始类型。然后，具有公共基类型的信息对于这些序列化器可以与之一起工作的类型信息来说是一份极好的信息。

在本章开头提到的**基础**链接中，你可以找到一个如何为**System.Text.Json**创建**JsonConverter**的示例，该转换器将在序列化期间自动将概念实现的任何实例转换为底层值，并在反序列化时将它们转换回概念类型。在**基础**链接的同一位置，你可以看到**ConceptAsJsonConverterFactory**实现，它根据**ConceptAs**基类型识别类型是否可以转换，并创建正确的转换器来序列化和反序列化概念。

使用**ConceptAs**结构体，我们现在可以创建特定领域的实现。正如你将在本章代码（[`github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Chapter4/BusinessApp`](https://github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Chapter4/BusinessApp)）中找到的那样，你可以创建特定的概念。例如，对于之前创建的**RegisterEmployee**命令，你不仅可以使用原始类型，还可以创建以下特定类型：

```cs
public record FirstName(string Value) :
  ConceptAs<string>(Value);
public record LastName(string Value) :
  ConceptAs<string>(Value);
public record SocialSecurityName(string Value) :
  ConceptAs<string>(Value);
```

然后，你可以将之前的**RegisterEmployee**修改为以下内容：

```cs
namespace Domain.Employees;
public class RegisterEmployee
{
    public FirstName FirstName { get; set; } =
      new(string.Empty)
    public LastName LastName { get; set; } = new(string.Empty);
    public SocialSecurityNumber SocialSecurityNumber { get;
      set; } = new(string.Empty);
}
```

这将消除代码中可能出现的任何潜在错误。你不可能意外地将**FirstName**放入**LastName**或反之亦然，因为编译器会告诉你它们不是同一类型。

## 横切关注点

在有了类似概念的基础上，我们真正开始丰富我们的应用程序代码，使其包含有意义的元数据。这创造了新的机会。如前所述，对于序列化，由于我们有了所有事物都使用的基类型，我们可以轻松地在单一位置处理概念类型的序列化。这就是我们谈论横切关注点时所指的：创建一次即可应用于多个事物的结构或行为，自动完成。

这些事情的可能性是无限的。我们可以自动创建在类型被使用时应用的验证规则。或者，我们可以在使用时根据类型应用授权策略。对于命令模式和强大的基于管道的框架如 ASP.NET，这意味着我们可以继续创建可以注入处理这些事物的管道中的操作过滤器或中间件——仅仅因为我们现在有了所需的元数据。

谈到安全，这是你真正想要做对的一件事。不要误解我们，我们旨在把我们所做的一切都做好——但安全是你绝对不希望马虎对待的事情。类型系统的丰富性和与之相关的所有元数据，让你有机会让开发者更容易做正确的事情。

例如，可以做的事情之一是创建基于命名空间的授权策略。如果你有一个进入的命令属于需要用户特定角色或声明的命名空间，你可以在一个地方进行这个检查，并且按照惯例，让开发者将命令或其他工件放入正确的位置，它们就会是安全的。

合规性是另一个你真正想要做对的地方。如果你的软件不合规，可能会非常昂贵。在过去的几年中，最被谈论的合规法律可能是欧盟的法规**GDPR**。如果你不遵守这项规定，你可能会被罚款。

GDPR 的整个想法是为了保护计算机系统最终用户的隐私。许多系统收集被称为**个人身份信息**，简称**PII**的数据。诸如你的姓名、地址、出生日期、社会保障号码等许多信息都被归类为 PII。还有一项要求是透明度，让最终用户知道你拥有他们的哪些数据，以及收集数据的原因。如果你的公司接受审计，你必须也在报告中展示你正在收集的数据类型。

在你刚刚学到的概念的基础上，我们可以更进一步，创建基础 **ConceptAs<>** 类型的专用版本。让我们称它为 **PIIConceptAs<>**：

```cs
namespace Fundamentals;
public record PIIConceptAs<T>(T Value) : ConceptAs<T>
  (Value)
{
}
```

正如你所见，它继承自 **ConceptAs<>**，这给了我们一次机会，可以围绕这个基础类型一次性创建序列化器和其他工具。但它添加了元数据，说明这是一个专门用于 PII 的概念。

在应用程序的运行时，你可以相当容易地向用户展示所有这些信息，以及任何审计机构或执法机构。

以我们之前创建的**SocialSecurityNumber**类型为例。将其更改为**PIIConceptAs<>**：

```cs
public record SocialSecurityName(string Value) :
PIIConceptAs<string>(Value);
```

如您从代码中看到的，要丰富它以利用元数据并不需要太多。而且，在章节开头构建的强大的类型发现功能，你现在可以轻松地创建一个简单的控制台报告，包含所有这些信息：

```cs
Console.WriteLine("GDPR Report");
var typesWithConcepts = types.All
                            .SelectMany(_ =>
                                _.GetProperties()
                                  .Where(p =>
                                    p.PropertyType
                                       .IsPIIConcept()))
                            .GroupBy(_ => _.DeclaringType);
foreach (var typeWithConcepts in typesWithConcepts)
{
    Console.WriteLine($"Type: {typeWithConcepts
      .Key!.FullName}");
    foreach (var property in typeWithConcepts)
    {
        Console.WriteLine($"  Property : {property.Name}");
    }
}
```

代码首先执行的操作是使用 LINQ 查询收集系统中所有**PIIConceptAs<>**类型的所有属性。如您所见，它使用了一个扩展方法，这是基础部分的一部分([`github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Fundamentals`](https://github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Fundamentals))。由于它选择了所有具有**.SelectMany()**的属性，所以我们根据声明类型将其分组。然后它只是遍历所有类型和所有属性，并打印出信息。

它应该产生以下结果：

```cs
GDPR Report
Type: Domain.Employees.RegisterEmployee
  Property : FirstName
  Property : LastName
  Property : SocialSecurityNumber
```

仅用少量代码，我们就从没有额外的元数据转变为一个更丰富的模型，这为我们提供了在代码中应用逻辑的机会。实际上，有了它，我们在一定程度上为代码提供了未来保障。我们可以在以后回到它，并决定我们想要更改安全性。实际上，我们不需要更改任何实际代码，只需根据我们已有的元数据应用一条新规则。这很强大，并使系统变得灵活、可扩展和高度可维护。

# 应用开放/封闭原则

**开放/封闭原则**是伯特兰·梅耶在其 1988 年出版的《面向对象软件构造》一书中提出的原则。这是一个关于我们可以应用于我们软件中的类型的原理：

+   如果一个类型可以被扩展，则称其为开放

+   当一个类型对其他类型可用时，它被称为封闭

C#中的类默认是开放的，可以扩展。我们可以从它们继承并给它们添加新的含义。但是，我们正在继承的基类应该是封闭的，这意味着对于新类型来说，不应该需要更改基类型。

这有助于我们为代码的可扩展性设计，并保持责任在正确的位置。

退一步来说，我们可以在系统级别应用一些相同的思考。如果我们的系统可以仅通过扩展新功能来扩展，而无需在类型的中心添加配置以了解这些添加，那会怎样？

这种思维方式就是我们之前在**ICommand**和实现中做的事情。我们添加的两个命令没有被系统的任何部分所知，但通过实现**ICommand**接口，我们可以通过系统的内省看到这两种类型。

# 摘要

在本章中，我们学习了如何通过查看我们正在运行的过程并收集所有引用的程序集及其类型来发挥其力量。我们探讨了如何利用更多的元数据来获取程序集引用的类型，无论是包引用还是项目引用。

通过这一点，我们得以以更有意义的方式对类型进行推理，并真正利用类型系统。接口可以作为标记类型的非常强大的方法。当然，接口可以强制实现需要存在的成员，但它们也可以仅仅作为空标记接口，作为将明确元数据引入程序集的方式。

在下一章中，我们将深入探讨如何充分利用自定义属性为您的应用程序提供明确的元数据。
