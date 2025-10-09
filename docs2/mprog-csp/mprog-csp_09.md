# 9

# 利用动态语言运行时的优势

C# 是一种静态类型语言，这意味着我们将代码以文本形式提交给编译器，然后它生成一个二进制文件，稍后执行。编译器完成后，代码不会改变。并非所有语言都像这样；像 Ruby、Python 和 JavaScript 这样的语言是动态语言，在执行之前不会编译成二进制。它们在运行时被解释，这意味着它们也可以在运行时逐渐改变。这是一个非常强大的特性。

在本章中，我们将探讨如何利用.NET 运行时的动态语言运行时部分，以与我们迄今为止所做的方式不同地动态创建代码。

我们将涵盖以下主题：

+   理解 DLR

+   对动态类型进行推理

+   创建 DynamicObject 并提供元数据

到本章结束时，你将了解动态语言运行时是什么，以及你如何利用它动态创建类型并对它们进行推理。

# 技术要求

本章的特定源代码可以在 GitHub 上找到 ([`github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Chapter9`](https://github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Chapter9))，并且它建立在以下位置找到的**基础知识**代码之上：[`github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Fundamentals`](https://github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Fundamentals)。

# 理解 DLR

**动态语言运行时**（**DLR**）于 2010 年随.NET 4 一起引入。它运行在.NET 运行时之上，即**公共语言运行时**（**CLR**），通过 IronPython 和 IronRuby 实现为 Python 和 Ruby 等动态语言提供语言服务。有了 DLR，还可以在不同语言之间进行互操作，从而有效地使 C#和 Python、Ruby 或任何其他受支持的动态语言能够在同一进程中并行运行。

## 一瞥 CLR

在深入研究 DLR 之前，了解它必须与之合作的约束条件是有帮助的。CLR 是.NET 的一切核心，这意味着所有运行的内容都必须遵守 CLR 的规则。当查看 CLR 的特性时，这一点变得非常有趣。

### 即时编译（JIT）

CLR 使用 JIT 编译器在运行时将**中间语言**（**IL**）代码编译为特定机器和操作系统的本地代码。此过程使优化更好、执行更快，并实现跨平台兼容性。

### 自动内存管理

CLR 使用垃圾回收来管理内存分配和释放，自动释放不再使用的对象占用的内存。这有助于防止内存泄漏并优化内存使用。

### 类型安全

CLR 通过在代码执行期间强制执行严格规则来确保类型安全。这有助于防止无效的内存操作，例如访问尚未初始化的内存位置或使用一种类型的对象作为另一种类型。

### 异常处理

CLR 为所有 .NET 语言提供了一致且健壮的异常处理机制，允许开发者优雅地处理运行时错误并保持应用程序的稳定性。

### 跨语言互操作性

CLR 支持不同 .NET 语言之间的互操作性，允许开发者在同一应用程序中使用用不同语言编写的组件。这促进了代码重用并简化了开发过程。

### 版本控制和程序集管理

CLR 管理程序集（编译代码单元）的版本控制，有助于防止诸如“DLL 地狱”等问题，即共享库的冲突版本导致应用程序不稳定。它还提供了诸如并行执行等特性，允许同一系统上共存多个版本的程序集。

### 反射

CLR 允许开发者在运行时检查和交互类型、对象和程序集的元数据。这允许动态类型创建、方法调用和其他高级编程技术。

### 调试和性能分析

CLR 为 .NET 应用程序提供了集成的调试和性能分析支持，使开发者能够有效地诊断和优化代码。

## DLR 的构建块

对于任何动态语言来说，DLR 需要支持一些与 CLR 更为静态的世界非常不同的特性。以下是需要的一些特性。

### 动态类型系统

一个动态运行时需要能够即时创建类型。类型永远不会是静态的，应该能够进行扩展。

### 动态方法分发

一个动态运行时应该允许调用事先未知的函数。

### 动态代码生成

一个动态运行时应该允许在运行时进行代码生成，允许你根据输入或其他代码的结果动态地添加代码。它还应该允许你修改代码。

从 CLR 的角度来看，这些类型的特性似乎非常不一致。微软的工程师扩展了核心功能，并实施了一个与 CLR 协作良好的动态运行时。

从元编程的角度来看，动态运行时的特性是完美的。作为 DLR 的一部分，有一系列构造和 API 可以让你动态创建类型，并对任何满足我们所需特性的动态类型进行推理。

由于 .NET 运行时本身是一个静态运行时，依赖于具有固定成员的已知类型，DLR 需要与此约束一起工作。因此，DLR 提供了一系列类和接口来表示动态对象、它们的属性和操作，例如 **IDynamicMetaObjectProvider**、**DynamicMetaObject**、**DynamicObject** 和 **ExpandoObject**。

为了让 DLR 能够与动态语言一起工作，它需要一种表示语言语义的方法。DLR 使用表达式树，我们在 *第七章*，*推理表达式* 和 *第八章*，*构建和执行表达式* 中更详细地讨论了这一点。随着 DLR 的到来，LINQ 表达式集增加了一系列扩展，为我们提供了控制流、赋值和其他语言建模节点。

从编译器的角度来看，C# 编译器有一个名为 **dynamic** 的关键字，允许你指定一个特定的变量或参数是动态类型。这告诉编译器在编译时不要评估实例的任何成员访问。相反，它将所有这些转换成将在执行时运行的适当 **Expression**。

要开始利用 DLR 及其动态功能，最简单的方法是使用 **ExpandoObject** 类型。**ExpandoObject** 提供了一种可以动态扩展的类型；它会保留你告诉它的任何内容，并且可以随着你的操作而扩展：

```cs
dynamic person = new ExpandoObject();
person.FirstName = "Jane";
person.LastName = "Doe";
```

代码创建了一个 **ExpandoObject** 实例，然后开始设置其上的值。**ExpandoObject** 类型本身不包含 **FirstName** 或 **LastName** 作为其类型的一部分，但通过设置它们，这些值就会存在。

访问这些成员的方式将相同；编译器理解 **ExpandoObject** 是 **dynamic** 的，并且将在运行时而不是在编译时尝试绑定到成员。以下代码将完全有效：

```cs
Console.WriteLine($"{person.FirstName} {person.LastName}");
```

结果将如预期：

```cs
Jane Doe
```

**ExpandoObject** 通过实现 **IDynamicMetaObjectProvider** 接口来完成这一点。通过 **IDynamicMetaObjectProvider**，它提供了自己的 **DynamicMetaObject** 实现，这实际上包含了执行我们告诉它执行的操作的所有魔法。

**ExpandoObject** 类型还实现了 **IDictionary<string, object?>** 接口，这是它表示所有赋予它的成员的方式，也是它能够动态增长的方式。通过其 **DynamicMetaObject** 的实现，它基本上只是在内部字典上工作。实际上，如果你想的话，完全可以把 **ExpandoObject** 当作字典来使用：

```cs
var person = new ExpandoObject() as IDictionary<string, object?>;
person["FirstName"] = "Jane";
person["LastName"] = "Doe";
```

如果你使用 **dynamic** 来处理数据对象，这会非常方便，因为它为你提供了一个简单的方式来推理对象及其内容，同时，通过将其视为具有属性的动态对象，提供了一个方便的编程体验。

## Call sites 和绑定器

每当你想要在动态对象上执行操作时——例如，获取或设置属性或调用方法——它将通过一个称为 **call site** 的东西。call site 实际上是将在运行时执行代码的位置。对于编译后的 C#，这将通常是 IL 代码驻留的内存位置。对于其他语言，如 Ruby 或 Python，这将是在文本代码中的位置。每种语言都有自己的 call site 表示。

静态语言和动态语言之间的主要区别在于，静态语言是编译的，在运行时，它们要么执行某种中间语言，要么执行实际的机器语言。相比之下，动态语言将在运行时解释代码并执行结果。动态语言的好处是它可以动态地改变自己的代码，显然，这是以性能损失为代价的。

一旦你有了位置，或者调用点，你就可以绑定到成员，然后执行你想要的操作。

存在着执行对象典型操作的绑定器 – 获取和设置属性、调用方法、索引等。在整个书中，我们探讨了元数据的力量以及我们如何对运行中的代码进行推理；DLR 在这方面提出了一些挑战。

# 推理动态类型

DLR 在推理动态类型方面非常有限。主要的是，DLR 被构建为能够托管动态语言。自然地，与动态语言一样，你实际上并不知道动态类型在发生什么，它可能会随时间改变，因此即使成员已经存在，DLR 也无法通过提供任何细节（如类型信息）来提供反射体验。

然而，我们可以请求它有哪些成员。每个动态对象都实现了一个名为**IDynamicMetaObjectProvider**的接口。在这个接口上，你可以调用**GetMetaObject()**方法来获取一个允许我们与动态对象交互的对象。

让我们看看具有以下属性的**ExpandoObject**：

```cs
dynamic person = new ExpandoObject();
person["FirstName"] = "Jane";
person["LastName"] = "Doe";
```

由于**ExpandoObject**实际上实现了**IDynamicMetaObjectProvider**接口，我们可以请求它的元对象，然后请求它的成员：

```cs
var provider = (person as IDynamicMetaObjectProvider)!;
var meta = provider.GetMetaObject(Expression.Constant(person));
var members = string.Join(',', meta.GetDynamicMemberNames());
Console.WriteLine(members);
```

运行此代码将给出以下结果：

```cs
FirstName,LastName
```

代码本身通过将其转换为类型来假设它是**IDynamicMetaObjectProvider**，以便我们调用**GetMetaObject()**。我们创建的**Expression**代表我们正在获取元对象的个人实例。然后，我们调用**GetDynamicMemberNames()**，它返回一个包含所有成员名称的字符串集合。

这就是可以推理的范围。

然而，我们可以动态调用成员，这在不知道对象形状时非常有用。当不知道对象包含什么时调用可能会有些挑战，如果你想要支持一切，你需要回退机制，因为如果运行时无法以你想要的方式绑定，它将抛出**RuntimeBinderException**。

**DynamicMetaObject**有一组方法，可以进行所有绑定操作。这些*绑定*方法需要一个实际的绑定器，你可以从调用点获取。绑定后，你将得到一个新的**DynamicMetaObject**。使用所有这些方法来获取实际值是可能的，但让我们直接跳到 C#绑定器，获取我们想要的绑定器，并使用它。

在 **Microsoft.CSharp.RuntimeBinder** 命名空间中，有一个名为 **Binder** 的类。这个类包含一组静态方法来获取特定的绑定器。你可以使用的绑定器类型如下：

| **二元运算** | 用于二元运算 |
| --- | --- |
| **转换** | 用于转换为特定类型 |
| **获取索引** | 用于获取索引 |
| **获取成员** | 用于获取成员的值 |
| **调用** | 用于调用方法 |
| **调用构造函数** | 用于调用/构造构造函数 |
| **设置索引** | 用于设置索引 |
| **设置成员** | 用于将成员设置为某个值 |
| **一元运算** | 用于一元运算 |

由于我们拥有的对象只有属性，我们将使用 **GetMember** 绑定器来获取属性值，并通过绑定点最终获取实际值：

```cs
foreach (var member in meta.GetDynamicMemberNames())
{
    var binder = Binder.GetMember(
        CSharpBinderFlags.None,
        member,
        person.GetType(),
        new[] {
          CSharpArgumentInfo.Create(CsharpArgumentInfoFlags
          .None, null) });
    var site = CallSite<Func<CallSite, object,
      object>>.Create(binder);
    var propertyValue = site.Target(site, person);
    Console.WriteLine($"{member} = {propertyValue}");
}
```

代码基于我们从 **GetMetaObject()** 获取的 **meta** 实例，并通过 **GetDynamicMemberNames()** 返回的成员名称进行迭代。对于每个成员，你可以获取它的绑定器——在我们的例子中，是 **GetMember** 绑定器——通过传递默认值 **member** 和 **person** 类型给它。

在 **System.Runtime.CompilerService** 命名空间中，你可以找到一个名为 **CallSite<>** 的类。这是一个动态站点类型，可以用来动态创建绑定点的站点。在我们的例子中，动态站点将是一个代表，**Func<>**，它允许我们用 **person** 来调用它并返回一个值。输入和输出的类型都是 **object**，因为实际类型是未知的。

运行此代码应该会给你以下结果：

```cs
FirstName = Jane
LastName = Doe
```

重要提示

代码假设所有成员都是属性，这可能会导致 **RuntimeBinder** **异常**。如果你需要支持更多成员类型，你需要处理这个异常并回退到你想要支持的类型。

另一种你可以使用的方法是使用表达式。**Expression** 类有一个 **Dynamic** 方法，它允许你传入绑定器并创建一个 **Lambda** 表达式，这给你一个类似的 **Func<>** 委托，可以调用。

以下提供了一个可以调用的方法来创建 **Func<>**，我们可以调用它来获取成员的值：

```cs
Func<object, object> BuildDynamicGetter(Type type, string propertyName)
{
    var binder = Binder.GetMember(
        CSharpBinderFlags.None,
        propertyName,
        type,
        new[] {
          CSharpArgumentInfo.Create(CsharpArgumentInfoFlags
          .None, null) });
    var rootParameter =
      Expression.Parameter(typeof(object));
    var binderExpression = Expression.Dynamic(binder,
      typeof(object), Expression.Convert(rootParameter,
      type));
    var getterExpression = Expression.Lambda<Func<object,
      object>>(binderExpression, rootParameter);
    return getterExpression.Compile();
}
```

代码获取 **GetMember** 绑定器，然后创建 **DynamicExpression**，这涉及到将我们将要传递的实例转换为 **object**。然后，代码创建一个 **Lambda** 表达式，我们随后对其进行编译以实现更高效的执行。

这个新方法可以按以下方式使用：

```cs
var firstNameExpression = BuildDynamicGetter(person.GetType(), "FirstName");
var lastNameExpression = BuildDynamicGetter(person.GetType(), "LastName");
Console.WriteLine($"{firstNameExpression(person)} {lastNameExpression(person)}");
```

此输出的结果如下：

```cs
Jane Doe
```

由于 DLR 本身在可发现性和推理方面有限，在某些用例中，自己提供元数据可能是一个好主意，使用其他格式和方法。

# 创建 DynamicObject 并提供元数据

有时候，你可能没有在代码中表示类型的奢侈。这可能是因为你正在调用某种外部 API，无论是 REST API、SOAP 服务还是类似的服务。然而，你调用的第三方可能有一个标准格式（如 WSDL 或 JSON 模式）中类型的表示。

尽管动态对象可以非常灵活，但在现实世界中，数据的形状往往更加严格。因此，你不必为所有内容都使用 **ExpandoObject**，你可以用自定义动态对象来表示这些类型，该对象从已知格式获取其元数据。今天，使用 JSON 作为数据载体非常普遍，利用 JSON 模式来表示数据的形状也很常见。让我们看看它如何成为元数据的提供者。

## 构建 JSON 模式类型

首先创建一个名为 **Chapter9** 的文件夹。在命令行界面中切换到这个文件夹，并创建一个新的控制台项目：

```cs
dotnet new console
```

我们将依赖第三方库，这为我们提供了一个轻松处理 JSON 模式的方法。通过运行以下命令将包添加到项目中：

```cs
dotnet add package NJsonSchema
```

JSON 模式是一个简单的结构，它描述了一个类型、其属性以及每个属性的类型。然而，它仅限于 JSON 可用的类型：

+   字符串

+   数字

+   布尔

+   数组

+   对象

然而，JSON 模式支持一个名为 *格式* 的概念，它可以作为类型的子类型使用。例如，日期不是 JSON 类型系统的一部分，但拥有一个类型为 **string** 且格式为 **date** 的属性将允许你支持任何你想要的类型，因为字符串可以包含任何内容。

JSON 模式还可以包含子模式，这使得在对象内拥有强类型对象定义成为可能。

对于这个示例，我们将保持非常简单，坚持使用简单类型。

将一个名为 **person.json** 的文件添加到你的项目中，并添加以下内容：

```cs
{
  „$schema": „http://json-schema.org/draft-04/schema#",
  "title": "Person",
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "FirstName": {
      "type": "string"
    },
    "LastName": {
      "type": "string"
    },
    "Birthdate": {
      "type": "string",
      "format": "date"
    }
  }
}
```

在 JSON 中，你可以找到 **title** 属性，它是类型的名称，而 **type** 属性设置为 **object**，因为它描述了一个对象。然后模式包含三个属性，其类型如下：

| **属性** | **类型** |
| --- | --- |
| 首字母 | 字符串 |
| 姓氏 | 字符串 |
| 出生日期 | 日期 |

表 9.1 – 人的模式

添加一个名为 **JsonSchemaType.cs** 的文件，并将以下代码添加到其中：

```cs
using System.Dynamic;
using NJsonSchema;
namespace Chapter9;
public class JsonSchemaType : DynamicObject
{
    readonly IDictionary<string, object?> _values = new
      Dictionary<string, object?>();
    readonly JsonSchema _schema;
    public JsonSchemaType(JsonSchema schema)
    {
        _schema = schema;
    }
```

代码创建了一个名为 **JsonSchemaType** 的新类型，它继承自 **System.Dynamic** 命名空间中找到的 **DynamicObject** 类型。这个特定的类型是一个辅助类型，创建它的目的是为了使实现动态对象更加容易。为了表示类型的实际内容，代码添加了 **Dictionary<string, object?>**。这使得你可以将其中的任何内容放入其中，就像 **ExpandoObject** 所做的那样。将 **object?** 作为值类型的原因是允许任何内容，并且明确表示我们允许其中包含空值。构造函数接受来自你之前添加的 **NJsonSchema** 依赖项的 **JsonSchema** 类型。

### 验证属性

由于 JSON 架构提供了类型的完整描述，包括其属性和属性的类型，它为你提供了验证对象上设置的值的机会。

让我们在对象中引入一些基本的验证。首先添加一个新文件，该文件将给出可能发生的具体问题的显式异常。添加一个名为**InvalidTypeForProperty.cs**的文件，并将以下代码添加到其中：

```cs
public class InvalidTypeForProperty : Exception
{
    public InvalidTypeForProperty(string type, string
      property) : base($"Property '{property}' on '{type}'
      is invalid.")
    {
    }
}
```

自定义异常有两个属性 - 错误属性所属类型的名称以及实际错误的属性。你也可以为了保险起见包括它试图设置的类型以及期望的类型，但为了使示例简单，让我们只使用这个。

要进行实际验证，你需要将.NET 类型转换为**JsonObjectType**的东西。回到**JsonSchemaType.cs**文件，添加一个将.NET 类型转换为**JsonObjectType**所需的方法。将以下方法添加到**JsonSchemaType**类中：

```cs
JsonObjectType GetSchemaTypeFrom(Type type)
{
    return type switch
    {
        Type _ when type == typeof(string) =>
          JsonObjectType.String,
        Type _ when type == typeof(DateOnly) =>
          JsonObjectType.String,
        Type _ when type == typeof(int) =>
          JsonObjectType.Integer,
        Type _ when type == typeof(float) =>
          JsonObjectType.Number,
        Type _ when type == typeof(double) =>
          JsonObjectType.Number,
        _ => JsonObjectType.Object
    };
}
```

代码使用简单的模式匹配并将.NET 类型转换为**JsonObjectType**。对于没有明确转换的情况，它默认为**JsonObjectType.Object**。

重要提示

此实现非常简单，并不涵盖所有.NET 类型，你可能需要为生产环境使其更复杂。不过，它应该给你一个一般性的概念。

现在你有了从 CLR 类型转换为**JsonObjectType**的方法，你需要一种实际进行验证的方法。将以下方法添加到**JsonSchemaType**类中：

```cs
void ValidateType(string property, object? value)
{
    if (value is not null)
    {
        var schemaType = GetSchemaTypeFrom(
          value.GetType());
        if (!_schema.ActualProperties[property]
          .Type.HasFlag(schemaType))
        {
            throw new InvalidTypeForProperty(_schema.Title,
              property);
        }
    }
}
```

代码只有在值不为 null 时才会验证该值，因为除非它有值，否则你无法知道它的类型。有了类型，它就获取实际的架构类型，然后检查属性是否具有此类型。**JsonObjectType**是一个标志枚举，允许组合，这就是为什么你必须使用**HasFlag**方法。如果类型不正确，代码会抛出**InvalidTypeForProperty**异常。

重要提示

在这里，一个提示是也要验证是否允许 null。此信息也由 JSON 架构支持。某些类型本身也不允许 null，例如整数或布尔值，你可能不应该允许这种情况。

### 实现获取和设置属性

在构造函数和验证就绪后，你现在可以覆盖**DynamicObject**类型的某些默认行为。**DynamicObject**类型提供了一组虚拟方法，可以覆盖以执行不同的操作，例如获取或设置属性和调用方法。

在这个示例中，我们将主要关注属性。将以下方法添加到**JsonSchemaType**类中：

```cs
public override bool TrySetMember(SetMemberBinder binder, object? value)
{
    if (!_schema.ActualProperties.ContainsKey(binder.Name))
    {
        return false;
    }
    ValidateType(binder.Name, value);
    _values[binder.Name] = value;
    return true;
}
```

**TrySetMember**签名接受**SetMemberBinder**，它包含有关正在设置的属性的详细信息。它还获取值，该值可能为 null。然后代码首先通过查看**ActualProperties**字典来验证该属性是否实际存在于模式中。如果不存在，它立即返回**false**，如果你尝试设置一个未知的属性，那么你会得到**RuntimeBinderException**。当属性存在时，代码验证值的类型。如果类型正确，代码然后将其值设置在其私有字典中。

一旦你设置了一个属性，你也想读取它。向**JsonSchemaType**类添加以下方法：

```cs
public override bool TryGetMember(GetMemberBinder binder, out object? result)
{
    if (!_schema.ActualProperties.ContainsKey(binder.Name))
    {
        result = null!;
        return false;
    }
    result = _values.ContainsKey(binder.Name)
        ? result = _values[binder.Name] : result = null!;
    return true;
}
```

与**TrySetMember**类似，代码会检查模式是否有该属性。然而，由于这是一个**get**操作，并且方法的签名规定结果应该作为**out**参数给出，我们需要显式地将它设置为**null!**。然后代码检查私有字典是否包含该属性的值，如果包含则返回它，如果不包含则返回一个**null!**值。

重要提示

在一个生产系统中，你通常不希望仅返回**null**，如果值没有被设置。它应该被设置为该类型的默认值，或者如果 JSON 模式包含具有默认值的附加元数据，你应该使用那个值。

作为一项**值得拥有的**功能，我们希望允许这种类型转换为另一种类型——在我们的例子中，是**Dictionary<string, object?>**。

**DynamicObject**提供的一个可以重写的方法是**TryConvert**方法。如果从类型显式转换为不同的目标类型，将调用此方法。向**JsonSchemaType**类添加以下方法：

```cs
public override bool TryConvert(ConvertBinder binder, out object? result)
{
    if (binder.Type.Equals(typeof(Dictionary<string,
      object?>)))
    {
        var returnValue = new Dictionary<string,
          object?>(_values);
        var missingProperties =
          _schema.ActualProperties.Where(_ =>
          !_values.Any(kvp => _.Key == kvp.Key));
        foreach (var property in missingProperties)
        {
            object defaultValue = property.Value.Type
              switch
            {
                JsonObjectType.Array =>
                  Enumerable.Empty<object>(),
                JsonObjectType.Boolean => false,
                JsonObjectType.Integer => 0,
                JsonObjectType.Number => 0,
                _ => null!
            };
            returnValue[property.Key] = defaultValue;
        }
        result = returnValue;
        return true;
    }
    return base.TryConvert(binder, out result);
}
```

代码只允许转换为**Dictionary<string, object?>**，因此它首先通过查看传入的**ConvertBinder**类型的**Type**属性来检查这一点。返回模式中所有属性的值是一种良好的实践，不要遗漏任何属性，并且由于所有属性可能都已设置，代码从现有的**_values**字典创建一个新的字典，然后找到任何缺失的属性。对于每个缺失的属性，它都会设置一个默认值。

重要提示

默认值已经简化。如前所述，对于生产，你应该考虑对默认值采用更复杂的方法，并查看 JSON 模式中可能存在的附加元数据。

我们想要添加的最后一块拼图是让外部人士能够推理出哪些成员可用。**DynamicObject**类型为我们提供了一个可以重写的方法。向**JsonSchemaType**类添加以下方法：

```cs
public override IEnumerable<string> GetDynamicMemberNames() => _schema.ActualProperties.Keys;
```

此方法将由**DynamicObject**产生的**DynamicMetaObject**调用。**DynamicObject**也是**IDynamicMetaObjectProvider**，并实现了**GetMetaObject()**方法。

**DynamicObject** 有更多可重写的方法来调用方法、执行二元运算等。然而，对于这个示例，我们将专注于使用属性的数据方面。

### 使用模式基础设施

您到目前为止构建的是一个用于处理 JSON 模式的基础设施，以及 DLR。让我们来试一试 **JsonSchemaType**。打开 **Program.cs** 文件并移除其所有内容。添加以下内容：

```cs
using NJsonSchema;
using Chapter9;
var schema = await JsonSchema.FromFileAsync("person.json");
dynamic personInstance = new JsonSchemaType(schema);
var personMetaObject = personInstance.GetMetaObject(Expression.Constant(personInstance));
var personProperties = personMetaObject.GetDynamicMemberNames();
Console.WriteLine(string.Join(',', personProperties));
```

代码从您之前创建的 **person.json** 文件中读取 JSON 模式。然后，它创建一个 **JsonSchemaType** 实例并将其模式传递给它。由于 **JsonSchemaType** 是一个动态对象，它实现了 **IDynamicMetaObjectProvider**，我们可以使用对象的实例调用 **GetMetaObject()** 方法，然后获取其成员。

运行时，代码应该产生以下结果：

```cs
FirstName,LastName
```

设置和获取属性应该按预期工作：

```cs
personInstance.FirstName = "Jane";
Console.WriteLine($"FirstName : '{personInstance.FirstName}'");
```

运行代码应该给出以下输出：

```cs
FirstName : 'Jane'
```

由于 **JsonSchemaType** 的实现支持未设置的属性的默认值，您可以无任何问题地获取一个有效的属性：

```cs
Console.WriteLine($"LastName : '{personInstance.LastName}'");
```

这应该产生以下结果：

```cs
LastName : ''
```

将此属性转换为字典也应该通过显式转换直接工作：

```cs
var dictionary = (Dictionary<string, object>)personInstance;
Console.WriteLine(JsonSerializer.Serialize(dictionary));
```

这应该产生以下结果：

```cs
{"FirstName":"Jane","LastName":null}
```

要测试验证，您可以尝试将 **LastName** 属性设置为不支持的数据类型：

```cs
personInstance.LastName = 42;
```

这应该产生以下结果：

```cs
Unhandled exception. Chapter9.InvalidTypeForProperty: Property 'LastName' on 'Person' is invalid.
```

为了验证一切按预期工作，我们可以设置一个在模式中不存在的属性：

```cs
personInstance.FullName = "Jane Doe";
```

这应该产生以下结果：

```cs
Unhandled exception. Microsoft.CSharp.RuntimeBinder.RuntimeBinderException: 'Chapter9.JsonSchemaType' does not contain a definition for 'FullName'
```

使用 **DynamicObject** 辅助类型简化了动态对象的开发，因为您不必担心移动部件的复杂性，可以专注于您想要提供的动态对象的实际形状和能力。

# 摘要

本章探讨了 .NET 运行时的动态领域以及您如何利用它来创建在 C# 代码之外定义类型的动态类型。当通过 API 与第三方合作时，这种方法可能非常有用。如果您查看 REST API 和 OpenAPI 标准，您会发现 JSON 模式的广泛使用，将本章中介绍的方法与这些标准相结合，可以为您提供一种强大的机制来动态集成第三方，同时您还可以对数据的形状保持严格。

DLR 可以成为您工具箱中的一个强大工具。它提供了另一种动态创建类型和代码的方法。与生成中间语言代码相比，它可能看起来更直观。

DLR 的一个缺点是生成的类型在现代 IDE 或代码编辑器中可能难以处理，因为它不了解这些类型，无法提供诸如 IntelliSense 成员等服务。

使用 DLR 和动态方法的一个方面可能是性能。它的性能可能不如生成中间语言代码。这是您必须做出的权衡之一，但在特定场景下，这可能根本不是问题。

在下一章中，我们将稍微改变一下方向，看看我们如何利用本书中讨论的一些技术，并开始从**约定** **而非配置**的角度思考。

# 第三部分：提高生产力、一致性和质量

在这部分，您将看到如何使用元编程来提高代码质量，并使您的代码库更加可维护和一致。同时，这部分还为您提供了关于如何通过技术提高您和您的开发者的生产力的想法。不同的章节涉及原则和软件设计模式，以及它们如何在现实生活中应用。

本部分包含以下章节：

+   *第十章*，*约定优于配置*

+   *第十一章*，*应用开闭原则*

+   *第十二章*，*超越继承*

+   *第十三章*，*应用横切关注点*

+   *第十四章*，*面向方面编程*
