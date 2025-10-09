# 8

# 构建和执行表达式

C#中的表达式不仅非常适合用作捕获元数据和推理代码的手段；你还可以根据代码表示为 lambda 表达式生成表达式树，正如我们在*第七章*中看到的，*推理表达式*，或者通过程序化地添加不同的表达式节点类型自己生成。

你构建的表达式可以随后执行。由于表达式提供的功能范围很广，它们几乎和生成中间语言代码一样强大，正如我们在*第六章*中看到的，*动态代理生成*。因为每个表达式都是位于执行的具体表达式内部的代码，并执行表达式的任务，所以你不能期望其性能与在.NET 运行时本地运行的中间语言相同。因此，是否选择表达式生成而不是代理生成取决于你的用例。

在本章中，我们将探讨如何利用表达式来表达我们的意图并执行它们，并希望从中获得一些灵感，了解它们可能有什么用。

我们将涵盖以下主题：

+   创建你自己的表达式

+   将表达式作为委托创建并执行

+   创建查询引擎

通过本章，你应该理解表达式树是如何构建的，以及你如何可以利用它们生成可以随后执行的动态逻辑。

# 技术要求

本章的特定源代码可以在 GitHub 上找到([`github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Chapter8`](https://github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Chapter8))，它基于这里找到的**基础知识**代码：[`github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Fundamentals`](https://github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Fundamentals)。

# 创建你自己的表达式

表达式几乎和.NET 运行时一样强大和有能力。这意味着我们可以表达中间语言所包含的所有操作，最终由运行时执行。构造与中间语言非常不同，正如我们在*第七章*中看到的，*推理表达式*。它们都围绕一个树结构，左右表达式代表树中的节点。

使用表达式，我们不一定需要将它们限制为仅用作过滤数据的一种手段，实际上我们可以使用它们来持有执行操作的逻辑。由于它们的强大程度与中间语言相当，我们可以生成非常复杂的表达式树。

但权力越大，责任越大。尽管这是可能的，但这并不意味着这一定是个好主意。这些结构可能难以理解且难以调试，所以我建议不要全力以赴，而应将其视为一种新的编程语言。

假设我们有一个想要操作其属性的类型。为了简单起见，让我们将我们的类型设为一个具有字符串属性的类型，我们想要设置该属性：

```cs
public class MyType
{
    public string StringProperty { get; set; } =
      String.Empty;
}
```

对于这个示例，我们想要操作**StringProperty**属性，因此我们将其制作为一个**类**而不是**记录**，这样我们就可以操作其状态。

要创建一个表示我们需要构建的属性的表示式，首先从表示属性所属类型的某个东西开始。拥有类型也将由一个表示式表示。对于这个示例，我们不想让表达式创建**MyType**类型的实例，而是操作一个现有的实例。它将要操作的那个实例将由表示式的参数表示：

```cs
var parameter = Expression.Parameter(typeof(MyType));
```

参数接受参数的类型。但你也可以使用**.Parameter()**方法的某个重载给它一个名字，这在有多个参数时对于调试信息非常有用。

为了与属性一起工作，我们需要使用 C#反射来获取我们想要操作的属性的**PropertyInfo**：

```cs
var property = typeof(MyType).GetProperty
  (nameof(MyType.StringProperty))!;
```

使用表示实例的参数，我们将操作实际属性。现在我们可以创建一个在类型上表示属性的表达式：

```cs
var propertyExpression = Expression.Property
  (parameter,property);
```

我们最后想要做的是进行实际的赋值。为此，我们使用**Expression.Assign()**方法：

```cs
var assignExpression = Expression.Assign(
    propertyExpression,
    Expression.Constant("Hello world"));
```

**Expression.Assign()**方法接受表示目标的左侧表达式，而右侧表达式是源。在代码中，我们将其分配给一个**Hello world**的常量。

显然，这是一个非常简单的示例，实际上并没有做什么。你可以真正地利用表达式，利用诸如**Expression.IfThen()**和**Expression.IfElse()**之类的功能来处理**if**/**else**语句，你甚至可以执行**Expression.Call()**来调用方法，传递参数并处理结果。可以使用诸如**Expression.Add()**和**Expression.Subtract()**之类的功能来操作结果。你可以想象到的一切都可以做到。对于这本书，我们将在构建表示逻辑的表达式方面保持简单。

除非我们可以调用它们并获得结果，否则表达式实际上并没有什么用处。我们想要做的是构建我们想要的表达式树，然后能够非常容易地用标准的 C#代码调用它们。

# 创建表示式作为委托并执行它们

我们可以将表达式视为可以调用的方法或函数。它们可能包含参数或不包含参数，并且可能返回结果或不返回结果。我们可以将表达式表示为**Action**或**Func**。这两个都是**System**命名空间中找到的委托类型。**Action**表示一个无参数的操作，如果需要，可以返回结果。而**Func**表示一个具有一个或多个参数的函数，并且可以返回一个结果。这两种委托类型都可以接受泛型参数，代表输入参数类型和结果类型。

委托基本上只是方法的表示。它们定义了一个可调用签名。我们可以直接将委托作为方法调用。

使用表达式，我们可以将它们转换成可调用的表达式。这是通过**Expression.Lambda()**完成的。让我们基于我们之前拥有的属性赋值表达式来构建：

```cs
var parameter = Expression.Parameter(typeof(MyType));
var property = typeof(MyType).GetProperty
  (nameof(MyType.StringProperty))!;
var propertyExpression = Expression.Property
  (parameter,property);
var assignExpression = Expression.Assign(
    propertyExpression,
    Expression.Constant("Hello world"));
```

我们可以创建以下 lambda 表达式：

```cs
var lambdaExpression = Expression.Lambda<Action<MyType>>
  (assignExpression, parameter);
```

**Expression.Lambda()**方法接受一个泛型参数——委托的类型。这个委托可以是任何你想要的委托类型——你自己的自定义委托或者直接使用**Action**或**Func**。在这个例子中，我们将使用**Action<MyType>**，因为我们只是调用表达式，它将操作我们给出的实例。我们传递表示赋值表达式的表达式，然后是参数的定义。

在 lambda 表达式就位的情况下，我们可以将表达式编译成一个可调用的委托并调用它：

```cs
var expressionAction = lambdaExpression.Compile();
var instance = new MyType();
expressionAction(instance);
Console.WriteLine(instance.StringProperty);
```

代码调用**.Compile()**方法，然后会将表达式编译成一个可以直接调用的委托。它创建一个**MyType**类型的实例，并将其传递给委托。运行程序，你现在应该看到以下内容：

```cs
Hello world
```

这非常直接，这个例子可能没有展示出全部潜力。

# 创建查询引擎

让我们稍微改变一下方向，提高几个级别的复杂性，使其更加相关。我们可以使用表达式来做数据动态查询。一些应用程序甚至可能希望让最终用户能够为你的数据创建任意查询。我参与过提供这种类型能力给最终用户的项目，如果为最终用户提供一个灵活的查询系统而没有像表达式这样的工具，这可能会是一个难题。我见过一些解决方案，它们基本上只是直接将 SQL 暴露给最终用户，而不是尝试解决这个问题。给最终用户提供这种级别的权力可能会在未来造成问题。你最终会完全消除数据存储和最终用户之间的所有抽象。祝你好运，改变技术或支持多种数据存储机制。但幸运的是，我们手中拥有表达式的力量，这给了我们创建一个成功之坑的抽象的机会。

如在第*第七章*中讨论的，表达式与它们如何为目标数据源执行翻译是分离的。只要我们保持我们的表达式树足够简单，就应能够针对数据源进行优化执行。

构建一个交互式查询用户界面是复杂的，所以对于这本书，让我们让它更侧重于开发者，并使我们的查询界面成为一个 JSON 文档。我们希望创建一个能够使用定义的查询语言查询数据的数据库。

## 一个类似 MongoDB 的数据库

让我们构建一个类似于文档数据库的东西。MongoDB 可以是一个好的蓝图。显然，我们不会构建一个完全功能的文档数据库，但我们将使用其特性，并从中汲取灵感。

MongoDB 有一个非常类似于 JSON 的查询语言，这意味着你将构建不同的子句作为键/值表达式。假设我们有一个如下所示的 JSON 文档：

```cs
{ "FirstName": "Jane", "LastName": "Doe", "Age": 57 }
```

使用类似 MongoDB 的语法，我们可以执行一个对**LastName**属性的等于匹配查询，如下所示：

```cs
{
    "LastName": "Doe"
}
```

要添加更多需要匹配的属性——一个典型的**And**操作——你只需简单地添加另一个键/值属性，并设置你想要的值：

```cs
{
    "LastName": "Doe",
    "Age": 57
}
```

有时你会有**Or**语句，表示“这个”或“这个”，在 MongoDB 中，这将如下表示：

```cs
{
    "LastName": "Doe",
    "$or": [
        {
            "Age": 57
        }
    ]
}
```

如果你想要执行需要值大于、小于或类似的查询，你需要一个右侧的对象结构：

```cs
{
    "Age": {
        "$gt": 50
    }
}
```

如您所见，在 MongoDB 中，关键字前缀为**$**。我们不会实现所有这些，但只是几个——足以使其有趣。

## 构建一个简单的查询引擎

在满足要求后，我们现在可以开始构建能够解析查询并创建我们可以从代码中调用的表达式的引擎。

首先创建一个名为**Chapter8**的文件夹。在命令行界面中切换到这个文件夹，并创建一个新的控制台项目：

```cs
dotnet new console
```

添加一个名为**QueryParser.cs**的文件。向该文件添加以下内容：

```cs
using System.Linq.Expressions;
using System.Text.Json;
namespace Chapter8;
public static class QueryParser
{
    static readonly ParameterExpression
      _dictionaryParameter = Expression.Parameter(typeof
      (IDictionary<string, object>), "input");
}
```

代码添加了我们需要为此工作所需的命名空间，然后添加了一个类，该类目前持有我们将传递到查询评估表达式的参数的表示，该表达式将在最后拥有。

文档数据库的一个特点是，默认情况下，它们没有放入其中的对象的形状定义，与具有列的表定义的 SQL 数据库不同。为了模仿这种行为，你将使用表示为**IDictionary<string, object>**的键/值字典。这是表达式的参数。

查询表达式将具有以下结构：

```cs
left-hand (operand) right-hand
```

这里有一个例子：

```cs
LastName equals Doe
```

这意味着我们需要构建正确的左值表达式和右值表达式。向**QueryParser**类添加以下方法：

```cs
static Expression GetLeftValueExpression(JsonProperty
  parentProperty, JsonProperty property)
{
    var keyParam =
      Expression.Constant(parentProperty.Name);
    var indexer = typeof(IDictionary<string,
      object>).GetProperty("Item")!;
    var indexerExpr = Expression.Property(
      _dictionaryParameter, indexer, keyParam);
    return property.Value.ValueKind switch
    {
        JsonValueKind.Number =>
          Expression.Unbox(indexerExpr, typeof(int)),
        JsonValueKind.String =>
          Expression.TypeAs(indexerExpr, typeof(string)),
        JsonValueKind.True or JsonValueKind.False =>
          Expression.TypeAs(indexerExpr, typeof(bool)),
        _ => indexerExpr
    };
}
```

代码基于这样一个事实，即你的查询是以 JSON 构造的形式传入的。它是为递归而构建的，这就是为什么它需要一个 **parentProperty** 和一个 **property**。这是为了支持不仅仅是 **equals** 操作符，还支持嵌套的 **greater than** 类型操作符。在顶层，**parentProperty** 和 **property** 将是相同的，而当我们进行嵌套时，我们需要 **parentProperty** 而不是嵌套表达式中的 **property**，因为那代表操作符和值。

代码首先做的事情是创建一个访问字典的表达式。这是通过使用存在于 **IDictionary<string, object>** 上的 **Item** 属性来完成的。**Item** 属性不是你会在接口上看到的属性。它代表当你通常使用 **["SomeString"]** 进行索引时的索引器。索引器是接受参数的属性。因此，代码通过使用 **Expression.Property()** 方法的某个重载来设置一个 **IndexerExpression**。它传递索引器属性和一个表示文档上属性的常量。

代码需要做的最后一件事是确保返回的值是正确类型的。由于我们有 **IDictionary<string, object>**，值被表示为 **object**。你这样做是为了能够使用不同的操作符（**=**、**>**、**<**）。如果你不这样做，你会得到一个异常，因为它不知道如何处理将 **object** 与右侧表达式的实际类型进行比较。

数字是值类型，需要取消装箱，而字符串和布尔值需要转换为它们的实际类型。

重要提示

JSON 数字被硬编码为 **int**。这只是本示例的一个简化。数字可能不仅仅是那样，例如 **float**、**double**、**Int64** 以及更多。

现在你有了左侧表达式，你需要右侧表达式：

```cs
static Expression GetRightValueExpression(JsonProperty
  property)
{
    return property.Value.ValueKind switch
    {
        JsonValueKind.Number =>
          Expression.Constant(property.Value.GetInt32()),
        JsonValueKind.String => Expression.Constant(
          (object)property.Value.GetString()!),
        JsonValueKind.True or JsonValueKind.False =>
          Expression.Constant((object)property.Value
          .GetBoolean()),
        _ => Expression.Empty()
    };
}
```

代码创建常量表达式，以获取正确类型的值。在 **JsonProperty** 类型中，**Value** 属性是 **JsonElement** 类型。它具有获取你期望类型实际值的方法。由于查询引擎在类型方面有些限制，它仅支持 **int**、**string** 和 **bool**。

由于右侧也到位了，你需要某种东西来构建左右两侧的表达式，并将它们组合成一个可以使用的表达式。定义的查询功能包括能够执行 **greater than**、**less than** 以及更多，我们将其定义为查询 JSON 中键/值部分的复杂对象。有了这个，你需要一个构建正确表达式的函数。

将以下方法添加到 **QueryParser** 类中：

```cs
static Expression GetNestedFilterExpression(JsonProperty
  property)
{
    Expression? currentExpression = null;
    foreach (var expressionProperty in
      property.Value.EnumerateObject())
    {
        var getValueExpression = GetLeftValueExpression(
          property, expressionProperty);
        var valueConstantExpression =
          GetRightValueExpression(expressionProperty);
        Expression comparisonExpression =
          expressionProperty.Name switch
        {
            "$lt" => Expression.LessThan(
              getValueExpression, valueConstantExpression),
            "$lte" => Expression.LessThanOrEqual(
              getValueExpression, valueConstantExpression),
            "$gt" => Expression.GreaterThan(
              getValueExpression, valueConstantExpression),
            "$gte" => Expression.GreaterThanOrEqual(
              getValueExpression, valueConstantExpression),
            _ => Expression.Empty()
        };
        if (currentExpression is not null)
        {
            currentExpression = Expression.And(
              currentExpression, comparisonExpression);
        }
        else
        {
            currentExpression = comparisonExpression;
        }
    }
    return currentExpression ?? Expression.Empty();
}
```

代码枚举给定的对象，并使其能够有多个子句。它将这些子句作为 **And** 操作分组。它利用您之前创建的方法来获取左右表达式，然后对于每个支持的运算符，使用这些表达式来创建正确的运算符表达式。

由于 **GetNestedFilterExpression** 方法仅处理基于对象的嵌套过滤子句，您需要一个方法来处理顶层子句，并在它是嵌套对象时调用嵌套的子句：

```cs
static Expression GetFilterExpression(JsonProperty
  property)
{
    return property.Value.ValueKind switch
    {
        JsonValueKind.Object =>
          GetNestedFilterExpression(property),
        _ => Expression.Equal(GetLeftValueExpression(
          property, property), GetRightValueExpression(
          property))
    };
}
```

根据值的类型，代码选择仅当它是对象表示时才使用嵌套表达式。对于其他所有内容，它创建一个简单的 **equal** 表达式。

之前，我们讨论了能够执行 Or 操作而不仅仅是 And 操作的能力。将以下方法添加到 **QueryParser** 类中：

```cs
static Expression GetOrExpression(Expression expression,
  JsonProperty property)
{
   Foreach (var element in property.Value.EnumerateArray())
   {
       var elementExpression = GetQueryExpression(element);
       expression = Expression.OrElse(expression,
         elementExpression);
   }
   return expression;
}
```

我们将 Or 表达式定义为表达式数组。代码将值枚举为数组，并对每个元素调用 **GetQueryExpression** –这是我们接下来需要的方法。从结果中，它创建一个 **OrElse** 表达式。**OrElse** 表达式的原因是我们只想在先前的表达式评估为 **false** 时评估 Or。

继续添加以下方法到 **QueryParser** 类中：

```cs
static Expression GetQueryExpression(JsonElement element)
{
    Expression? currentExpression = null;
    foreach (var property in element.EnumerateObject())
    {
        Expression expression = property.Name switch
        {
            "$or" => GetOrExpression(currentExpression!,
              property),
            _ => GetFilterExpression(property)
        };
        if (currentExpression is not null && expression is
          not BinaryExpression)
        {
            currentExpression = Expression.And(
              currentExpression, expression);
        }
        else
        {
            currentExpression = expression;
        }
    }
    return currentExpression ?? Expression.Empty();
}
```

由于查询 JSON 可以包含多个语句，代码会遍历对象并评估每个属性。如果是 Or 表达式，则调用 **GetOrExpression** 方法。其他任何内容都转到 **GetFilterExpression**。这是您可以添加更多操作的地方。对于查询中的每个属性，它都会将它们组合在一起。

由于 **GetQueryExpression** 方法已经就位，我们已经拥有了查询引擎的所有逻辑。现在我们需要一个外部入口点。

将以下方法添加到 **QueryParser** 类中：

```cs
public static Expression<Func<IDictionary<string, object>,
  bool>> Parse(JsonDocument json)
{
    var element = json.RootElement;
    var query = GetQueryExpression(element);
    return Expression.Lambda<Func<IDictionary<string,
      object>, bool>>(query, _dictionaryParameter);
}
```

**Parse** 方法接受一个表示查询的 JSON 文档，并返回一个表达式，该表达式旨在与单个文档一起使用，其中每个文档是 **IDictionary<string, object>**。它调用 **GetQueryExpression** 来根据 JSON 的根元素创建实际的表达式，然后将其包装在一个可调用的 **lambda** 表达式中，该表达式接受一个字典作为参数。

现在您已经设置了查询引擎，是时候创建一些数据和查询，以及一些用于测试的代码了。

添加一个名为 **data.json** 的文件，并在其中添加以下内容或您自己的内容：

```cs
[
    { "FirstName": "Jane", "LastName": "Doe", "Age": 57 },
    { "FirstName": "John", "LastName": "Doe", "Age": 55 },
    { "FirstName": "Michael", "LastName": "Corleone",
      "Age": 47 },
    { "FirstName": "Anthony", "LastName": "Soprano",
      "Age": 51 },
    { "FirstName": "Paulie", "LastName": "Gualtieri",
      "Age": 58 }
]
```

对于查询本身，添加一个名为 **query.json** 的文件，并在其中添加以下内容或您自己的查询：

```cs
{
    "Age": {
        "$gte": 52
    },
    "$or": [
        {
            "LastName": "Doe"
        }
    ]
}
```

您接下来想要编写代码来解析这些文件，并从 **query.json** 创建一个表达式。

为了能够解析 **data.json** 文件并获取一个漂亮的 **Dictionary<string, object>** 类型集合，我们需要一个 JSON 转换器。默认情况下，如果你尝试将 JSON 反序列化到这个集合中，序列化器会给你一个 **JsonElement** 类型的值。我们想要实际的值。在这个上下文中，JSON 转换器的列表和说明没有价值。它们可以在章节开头引用的 GitHub 仓库中找到。

在 **Program.cs** 文件中，将所有内容替换为以下内容：

```cs
var query = File.ReadAllText("query.json");
var queryDocument = JsonDocument.Parse(query);
var expression = QueryParser.Parse(queryDocument);
var documentsRaw = File.ReadAllText("data.json");
var serializerOptions = new JsonSerializerOptions();
serializerOptions.Converters.Add(new Dictionary
  StringObjectJsonConverter());
var documents = JsonSerializer.Deserialize<Ienumerable
  <Dictionary<string, object>>>(documentsRaw,
    serializerOptions)!;
var filtered = documents.AsQueryable().Where(expression);
foreach (var document in filtered)
{
    Console.WriteLine(JsonSerializer.Serialize(document));
}
```

代码加载 **query.json** 和 **data.json** 文件并解析它们。对于查询，在将其传递给 **QueryParser.Parse()** 方法之前，你需要将其作为 **JsonDocument**。而数据则反序列化成一个 **Dictionary<string, object>** 类型的集合。

由于文档是 **IEnumerable**，你得不到你想要的 **.Where()** 扩展方法。因此，代码首先执行 **.AsQueryable()**，然后将解析的查询表达式传递给它。

经过过滤的对象可以被枚举。

运行代码应该给出以下结果：

```cs
{"FirstName":"Jane","LastName":"Doe","Age":57}
{"FirstName":"John","LastName":"Doe","Age":55}
{"FirstName":"Paulie","LastName":"Gualtieri","Age":58}
```

与向最终用户展示 SQL 等类似的方法相比，这种方法的优点是，你现在有一个可以与不同数据存储一起工作的抽象。你甚至可以将相同的表达式应用于内存中，就像应用于数据库一样。

# 摘要

在本章中，我们学习了构建表达式和表达式树的力量。它们不仅能够表示查询，实际上，它们可以与生成中间语言代码一样强大。

使用表达式，你得到了中间语言的替代品。它们在生成中间语言代码方面确实有一些限制，例如，你不能简单地创建类型和实现接口或覆盖继承的虚拟方法的行为。但作为表达简单动作的工具，它们真的非常出色。

在下一章中，我们将探讨另一种利用 **Dynamic** **语言运行时** 的能力动态创建类型和实现的方法。
