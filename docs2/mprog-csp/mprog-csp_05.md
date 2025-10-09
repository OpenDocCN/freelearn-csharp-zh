# 5

# 利用属性

我们简要介绍了 C# 属性的概念，见*第二章*，*元编程概念*。它们是向源代码添加显式元数据的明显选择。这正是它们的目的。属性不应携带复杂的逻辑，而应被视为仅是元数据。

在本章中，我们将探讨如何在代码库中利用它们，提供为类型和成员添加有价值、丰富信息的机制，这些信息可用于不同的场景。

我们将涵盖以下主题：

+   属性是什么，以及如何应用它？

+   查找具有特定属性的类型

+   泛型属性

从本章中，您应该了解属性作为元编程构建块的力量，如何创建自己的自定义属性，以及您如何发现它们的使用。

# 技术要求

该章节的特定源代码可以在 GitHub 上找到（[`github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Chapter5`](https://github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Chapter5)），并且它建立在**基础**代码之上，该代码位于[`github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Fundamentals`](https://github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Fundamentals)。

# 属性是什么，以及如何应用它？

**属性**是 C# 编译器理解的特殊类型。它可以用来将元数据关联到程序集、类型以及类型的任何成员。在编译期间，编译器将拾取属性并将它们作为元数据添加到编译的程序集中。您可以在每个项目上放置多个属性。

创建自己的自定义属性就像这样：

```cs
public class CustomAttribute : Attribute
{
}
```

然后使用它的方法如下：

```cs
[Custom]
public class MyClass
{
}
```

注意，您在属性名称中使用**Attribute**后缀。在使用它时，您不需要它，您只需要**[Custom]**。C# 编译器内置了一个约定，即您必须使用后缀，但在使用时它将忽略它。这有点奇怪，并且肯定违反了最小惊讶原则。

属性的优点在于它们存在于元素本身的范围之外，这意味着您不需要创建具有元数据的类型的实例来访问元数据。

属性可以接受参数以提供您想要捕获的特定信息。然而，所有参数必须在编译时可用。这意味着您不能为任何参数动态创建新对象。

例如，我们可以通过获取另一个类型的实例来向属性添加参数：

```cs
public class CustomAttribute : Attribute
{
    public CustomAttribute(SomeType instance)
    {
    }
}
```

编译器将允许属性接受类型——毕竟，它是一个有效的 C# 类型。然而，当您尝试使用它时，不允许您创建新实例：

```cs
[Custom(new SomeType())] // Will give you a compiler error
public class MyClass
{
}
```

这是因为编译器需要在编译时获取这些值。尽管属性最终是在运行时实例化的，但捕获并添加到编译后的汇编中的信息永远不会被执行。这意味着你只能限于编译器可以解析的事物，例如原始类型（例如，**int**、**float** 和 **double**）以及诸如字符串之类的对象——任何可以表示为常量且不需要由运行时创建以工作的事物。

一个有效的参数可以是字符串：

```cs
public class CustomAttribute : Attribute
{
    public CustomAttribute(string information)
    {
    }
}
```

现在构造函数接受一个字符串，它不仅会在编译时工作，也会在运行时工作。

由于字符串可以是字面常量，你可以使用它们：

```cs
[Custom("I'm a valid parameter")]
public class MyClass
{
}
```

已经，你可以看到属性的力量——能够拥有附加信息，你的代码可以据此进行推理，你可以用它来做决策，甚至用于报告。

## 限制属性使用

对于属性，你还可以向它们添加元数据，这有点像“自我包含”；元数据的元数据。你添加的元数据是为了限制属性的使用范围。

你可以对代码中的哪些元素使用属性（类、属性、字段等）非常具体。

**[AttributeUsage]** 属性允许你具体指定属性。假设你只想将使用限制为类——你可以这样做：

```cs
[AttributeUsage(AttributeTargets.Class)]
public class CustomAttribute : Attribute
{
}
```

如果你尝试将属性添加到除类之外的其他事物上，你将得到一个编译器错误：

```cs
public class MyClass
{
    [Custom] // Compiler error
    public void SomeMethod()
    {
    }
}
```

**[AttributeUsage]** 类型是一个枚举，包含支持不同代码元素属性的不同值。枚举中的每个值都代表一个标志，这使得它们可以组合起来并针对多个代码元素类型。

让我们通过应用具有这些指定的自定义属性 **[AttributeUsage]** 属性来限制代码元素为 **Class**、**Method** 和 **Property**：

```cs
[AttributeUsage(
    AttributeTargets.Class |
    AttributeTargets.Method |
    AttributeTargets.Property)]
public class CustomAttribute : Attribute
{
}
```

正如你所见，使用位运算符 **OR** 构造（**|**）你可以添加所有你想要支持的元素。

关于 **[AttributeUsage]** 的一个有趣的事实是，它使用自身来告诉编译器它只能用于类。再次，那里有点“自我包含”；**[AttributeUsage]** 属性正在使用 **[AttributeUsage]** 来提供关于自身的元数据。

除了限制属性可以关联的代码元素之外，你还可以指定是否允许应用属性的多个实例。你还可以指定是否允许属性作为具有该属性的应用类型的元数据。

**[AttributeUsage]** 属性在其构造函数中实际上只接受一个参数。这意味着我们必须显式地使用其属性。

默认情况下，属性仅限于每个代码元素类型仅关联一次。尝试多次关联属性将会导致编译器错误：

```cs
[Custom]
[Custom] // Compiler error
public class MyClass
{
}
```

通过简单地使用**AllowMultiple**属性，您可以改变这种行为：

```cs
[AttributeUsage(AttributeTargets.Class, AllowMultiple =
  true)]
public class CustomAttribute : Attribute
{
}
```

现在将允许编译相同的代码。

您还可以使用**Inherited**属性来限制属性的用法。将此属性设置为**false**将告诉编译器，相关的属性仅与显式使用的特定类型相关联，而不是与派生类型相关联：

```cs
[AttributeUsage(AttributeTargets.Class, Inherited = false)]
public class CustomAttribute : Attribute
{
}
```

如您之前所见，您可以通过常规方式将属性添加到类中：

```cs
[Custom]
public class MyClass
{
}
```

您可以添加一个继承自具有**[Custom]**属性的类型的类：

```cs
public class MyOtherClass : MyClass
{
}
```

当**Inherited**属性设置为**false**时，与**MyClass**基类型关联的元数据将不会与**MyOtherClass**关联。默认情况下，这是开启的，这意味着派生类型将具有与之关联的相同元数据。

要创建一个属性，您只需要从**Attribute**继承并应用**[AttributeUsage]**。然而，您可能希望通过不允许从您的属性继承来为您的元数据带来更多的清晰性和明确性。密封您的类将阻止任何人从您的自定义属性继承。

## 封闭您的属性类

由于属性代表特定的元数据，它们不是用于存储逻辑的常规代码结构。因此，您会发现您不需要创建其他更具体属性继承的基属性。实际上，如果您使用继承作为元数据，可能会使元数据变得不清晰，并且您会引入隐含性。

由于这个原因，被认为是一个好的实践，不允许属性的继承，并在编译器级别通过使属性类**密封**来停止它：

```cs
public sealed class CustomAttribute : Attribute
{
}
```

如果您尝试创建一个继承自它的更具体属性，您将得到编译器错误。

现在我们已经涵盖了创建自定义属性及其如何针对您的特定用例进行定制所涉及的所有机制，您可能已经迫不及待地想要开始实际发现它们并将它们用于良好的用途。

# 查找具有特定属性的类型

由于属性是在编译时创建的，并且不需要您有一个与属性关联的类型实例，您可以使用类型系统来发现属性。

如果您查看**System.Type**类型，您会发现它实现了一个名为**MemberInfo**的类型，该类型位于**System.Reflection**命名空间中。这个基类作为**PropertyInfo**、**MethodInfo**、**FieldInfo**以及大多数代表我们可以通过类型系统发现的代码元素的特定信息类型的基类。

在**MemberInfo**类型上，您会发现一个名为**GetCustomAttributes()**的方法。这允许您获取与特定代码元素关联的属性集合。

考虑我们之前拥有的类：

```cs
[Custom]
public class MyClass
{
}
```

然后，您可以非常容易地获取类型上的自定义属性，遍历它们并执行您想要的操作：

```cs
foreach( var attr in typeof(MyClass).GetCustomAttributes() )
{
    // Do something based on the attribute
}
```

使用 **typeof()** 非常明确，并且可以仅用于此类型。对于更动态的解决方案，你可以发现具有特定属性的类型，你可以利用这些属性来完成我们在 *第四章*，[*“使用反射进行类型推理”*] 中所做的 **“类型推理”** 工作。

## 个人可识别信息 (PII)

让我们回到之前章节中提到的 GDPR 主题。在 *第四章*，[*“使用反射进行类型推理”*] 中，我们使用类型来发现哪些是个人可识别信息。另一种方法可能是使用自定义属性作为显式的元数据方法。通过属性，我们可以关联比我们在 *第四章*，[*“使用反射进行类型推理”*] 中使用的基本类型更多的信息。你可以添加关于收集数据原因的元数据。

你可以使用以下属性来捕获它：

```cs
[AttributeUsage(AttributeTargets.Class |
  AttributeTargets.Property | AttributeTargets.Parameter,
    AllowMultiple = false, Inherited = true)]
public class PersonalIdentifiableInformationAttribute :
  Attribute
{
    public PersonalIdentifiableInformationAttribute(string
      reasonForCollecting = "")
    {
        ReasonForCollecting = reasonForCollecting;
    }
    public string ReasonForCollecting { get; }
}
```

代码创建了一个可以应用于类、属性和参数的属性。它不允许将多个实例应用于将要应用到的代码元素。它允许元数据对任何继承自应用了此元数据的类型或其成员的类型都可用。关于 GDPR，有趣的是了解系统收集特定数据的原因——因此，属性作为可选元数据具有这个功能。

重要提示

你可以在 GitHub 仓库中的 **Fundamentals** 项目中找到这个实现。

首先创建一个名为 *第五章* 的文件夹。在命令行界面中切换到该文件夹并创建一个新的控制台项目：

```cs
dotnet new console
```

接下来你需要做的是引用 Fundamentals 项目。如果你将项目放在 **Chapter5** 文件夹旁边，请执行以下操作：

```cs
dotnet add reference ../Fundamentals/Fundamentals.csproj
```

在此基础上，假设你想要创建一个封装员工的对象：

```cs
public class Employee
{
    public string FirstName { get; set; } = string.Empty;
    public string LastName { get; set; } = string.Empty;
    public string SocialSecurityNumber { get; set; } =
      string.Empty;
}
```

此类型显然持有对个人可识别的属性；让我们为其成员添加适当的元数据：

```cs
using Fundamentals.Compliance.GDPR;
public class Employee
{
    [PersonalIdentifiableInformation("Employment records")]
    public string FirstName { get; set; } = string.Empty;
    [PersonalIdentifiableInformation("Employment records")]
    public string LastName { get; set; } = string.Empty;
    [PersonalIdentifiableInformation("Uniquely identifies
      the employee")]
    public string SocialSecurityNumber { get; set; } =
      string.Empty;
}
```

现在，我们有了足够的信息来发现系统中任何持有此类信息的类型。

基于 **第四章** 中引入的汇编和类型发现系统，*“使用反射进行类型推理”*，我们可以专门查询这个。

由于类型上的每个成员都继承自 **System.Reflection** 命名空间中找到的 **MemberInfo** 类型，我们可以轻松创建一个便利的扩展方法，允许我们检查成员是否具有与其关联的特定属性。

然后，你可以创建一个简单的扩展方法，允许你检查属性是否与成员关联：

```cs
public static class MemberInfoExtensions
{
    public static bool HasAttribute<TAttribute>(this
      MemberInfo memberInfo) where TAttribute : Attribute
        => memberInfo.GetCustomAttributes<TAttribute>()
          .Any();
}
```

重要提示

你可以在 GitHub 仓库中的 **Fundamentals** 项目中找到这个实现。

在此基础上，你可以发现所有包含此类信息的类型：

```cs
using Fundamentals;
var types = new Types();
var piiTypes = types.All.Where(_ => _
                    .GetMembers()
                    .Any(m => m.HasAttribute<Personal
                      IdentifiableInformation
                        Attribute>()));
var typeNames = string.Join("\n", piiTypes.Select(_ =>
  _.FullName));
Console.WriteLine(typeNames);
```

**HasAttribute<>** 扩展方法是一个强大的小助手，你会在所有想要根据属性进行简单类型元数据查询的场景中找到它。

要创建包含收集信息原因的 GDPR 报告，将 **Program.cs** 文件修改如下：

```cs
using System.Reflection;
using Fundamentals;
var types = new Types();
Console.WriteLine("\n\nGDPR Report");
var typesWithPII = types.All
                        .SelectMany(_ =>
                            _.GetProperties()
                                .Where(p => p.HasAttribute
                                 <PersonalIdentifiable
                                  InformationAttribute>()))
                        .GroupBy(_ => _.DeclaringType);
foreach (var typeWithPII in typesWithPII)
{
    Console.WriteLine($"Type: {typeWithPII.Key!
      .FullName}");
    foreach (var property in typeWithPII)
    {
        var pii = property.GetCustomAttribute<
          PersonalIdentifiableInformationAttribute>();
        Console.WriteLine($"  Property : {property.Name}");
        Console.WriteLine($"    Reason :
          {pii.ReasonForCollecting}");
    }
}
```

代码利用了在 *第四章* 中引入的类型发现，*使用反射进行类型推理*，并使用 LINQ 扩展方法选择所有应用了 **[PersonalIdentifiableInformationAttribute]** 属性的类型。然后按类型分组，这样你可以轻松地遍历并按类型展示具有该属性的成员。

运行此代码将产生以下结果：

```cs
GDPR Report
Type: Main.Employee
  Property : FirstName
    Reason : Employment records
  Property : LastName
    Reason : Employment records
  Property : SocialSecurityNumber
    Reason : Uniquely identifies the employee
```

这种类型的元数据对商业来说非常有价值。如果你的业务收到政府关于 GDPR 审计的查询，并且你的代码完全加载了元数据，你可以轻松地创建一份关于你收集的数据类型及其收集原因的报告。

你也可以将此类信息展示给系统最终用户。了解系统收集关于他们的信息对用户来说非常有价值。这建立了系统与用户之间的信任关系。

GDPR 是将非常有用的元数据引入代码库的一个非常好的用例，但它只是众多用例之一。当然，你可以以更可操作的方式利用元数据，而不仅仅是用于报告。

# 泛型属性

我们之前使用的 C# 属性的一个限制是属性不能是接受泛型参数的泛型类型。在 C# 11 之前，如果你在你的属性类中添加了泛型参数，你会得到编译器错误。这种限制随着 C# 11 的发布而被解除。

一直到 C# 11，你收集类型信息的方式只有当属性有参数或属性类型为 **System.Type** 时。这变得非常冗长：

```cs
public class CustomAttribute : Attribute
{
    public CustomAttribute(Type theType)
}
```

然后用属性装饰类型的方式如下：

```cs
[Custom(typeof(string))]
public class MyClass
{
}
```

使用 C# 11，现在你可以改进获取类型信息的方式：

```cs
public class CustomAttribute<T> : Attribute
{
}
```

当你用属性装饰类型时，你使用泛型参数：

```cs
[Custom<string>]
public class MyClass
{
}
```

如果你需要一个动态类型的参数，你可以这样做：

```cs
[AttributeUsage(AttributeTargets.Class, AllowMultiple =
  true)]
public class CustomAttribute<T> : Attribute
{
    public CustomAttribute(T someParameter)
    {
        SomeParameter = someParameter;
    }
    public T SomeParameter { get; }
}
```

代码定义了一个接受泛型参数的属性，然后要求属性有一个参数，该参数将是泛型类型。然后它使用相同的泛型类型在属性上公开元数据作为属性。

当用属性装饰类型时，你指定类型和参数，因为属性必须是指定类型：

```cs
[Custom<int>(42)]
[Custom<string>("Forty two)]
public class MyClass
{
}
```

重要提示

通常，C# 编译器非常擅长根据传入的类型推断泛型参数的类型。但使用泛型属性时，你必须每次都显式地给出泛型类型。

泛型属性可以是一种强大的元数据收集方法。它增加了你构建元数据的方式的灵活性。

# 摘要

在本章中，我们探讨了 C#属性是什么以及它们在描述代码中的显式元数据方面的强大功能。我们研究了如何创建自己的自定义属性并将它们应用到不同的代码元素上的一切机制。从这种类型的元数据中，你现在可以丰富你的代码。

通过本章探讨的丰富内容，你看到了如何非常容易地发现这种元数据并将其用于你的业务。

在下一章中，我们将进一步深入探讨.NET 运行时的功能，并查看你如何根据元数据动态生成代码，这样可以使你在作为开发者进行这项工作时更加高效。
