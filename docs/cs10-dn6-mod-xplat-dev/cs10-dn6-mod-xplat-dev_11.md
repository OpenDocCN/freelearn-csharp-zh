# 第十一章：使用 LINQ 查询和操作数据

本章是关于**语言集成查询**（**LINQ**）表达式的。LINQ 是一系列语言扩展，它增加了处理项目序列的能力，然后对其进行过滤、排序，并将其投影到不同的输出中。

本章将涵盖以下主题：

+   编写 LINQ 表达式

+   使用 LINQ 处理集合

+   将 LINQ 与 EF Core 结合使用

+   用语法糖美化 LINQ 语法

+   使用并行 LINQ 进行多线程处理

+   创建自己的 LINQ 扩展方法

+   使用 LINQ to XML

# 编写 LINQ 表达式

尽管我们在*第十章*，*使用 Entity Framework Core 处理数据*中写了一些 LINQ 表达式，但它们并非重点，因此我没有适当地解释 LINQ 的工作原理，所以现在让我们花时间来正确理解它们。

## 何为 LINQ？

LINQ 包含多个部分；有些是必选的，有些是可选的：

+   **扩展方法（必选）**：这些包括`Where`、`OrderBy`和`Select`等示例。正是这些方法提供了 LINQ 的功能。

+   **LINQ 提供程序（必选）**：这些包括用于处理内存中对象的 LINQ to Objects、用于处理存储在外部数据库中并由 EF Core 建模的数据的 LINQ to Entities，以及用于处理存储为 XML 的数据的 LINQ to XML。这些提供程序是针对不同类型的数据执行 LINQ 表达式的方式。

+   **Lambda 表达式（可选）**：这些可以用来代替命名方法来简化 LINQ 查询，例如，用于`Where`方法的过滤条件逻辑。

+   **LINQ 查询理解语法（可选）**：这些包括`from`、`in`、`where`、`orderby`、`descending`和`select`等 C#关键字。它们是一些 LINQ 扩展方法的别名，使用它们可以简化你编写的查询，特别是如果你已经有其他查询语言（如**结构化查询语言**（**SQL**））的经验。

当程序员首次接触 LINQ 时，他们常常认为 LINQ 查询理解语法就是 LINQ，但讽刺的是，这是 LINQ 中可选的部分之一！

## 使用 Enumerable 类构建 LINQ 表达式

LINQ 扩展方法，如`Where`和`Select`，由`Enumerable`静态类附加到任何实现`IEnumerable<T>`的类型，这种类型被称为**序列**。

例如，任何类型的数组都实现了`IEnumerable<T>`类，其中`T`是数组中项目的类型。这意味着所有数组都支持 LINQ 来查询和操作它们。

所有泛型集合，如`List<T>`、`Dictionary<TKey, TValue>`、`Stack<T>`和`Queue<T>`，都实现了`IEnumerable<T>`，因此它们也可以用 LINQ 进行查询和操作。

`Enumerable`定义了超过 50 个扩展方法，如下表总结：

| 方法(s) | 描述 |
| --- | --- |
| `First`, `FirstOrDefault`, `Last`, `LastOrDefault` | 获取序列中的第一个或最后一个项，如果没有则抛出异常，或者返回类型的默认值，例如，`int`的`0`和引用类型的`null`。 |
| `Where` | 返回与指定筛选器匹配的项序列。 |
| `Single`, `SingleOrDefault` | 返回与特定筛选器匹配的项，如果没有恰好一个匹配项，则抛出异常，或者返回类型的默认值。 |
| `ElementAt`, `ElementAtOrDefault` | 返回指定索引位置的项，如果没有该位置的项，则抛出异常，或者返回类型的默认值。.NET 6 中新增了可以传入`Index`而不是`int`的重载，这在处理`Span<T>`序列时更高效。 |
| `Select`, `SelectMany` | 将项投影到不同形状，即不同类型，并展平嵌套的项层次结构。 |
| `OrderBy`, `OrderByDescending`, `ThenBy`, `ThenByDescending` | 按指定字段或属性排序项。 |
| `Reverse` | 反转项的顺序。 |
| `GroupBy`, `GroupJoin`, `Join` | 对两个序列进行分组和/或连接。 |
| `Skip`, `SkipWhile` | 跳过一定数量的项；或在表达式为`true`时跳过。 |
| `Take`, `TakeWhile` | 获取一定数量的项；或在表达式为`true`时获取。.NET 6 中新增了`Take`的重载，可以传入一个`Range`，例如，`Take(range: 3..⁵)`表示从开始处算起第 3 项到结束处算起第 5 项的子集，或者可以用`Take(4..)`代替`Skip(4)`。 |
| `Aggregate`, `Average`, `Count`, `LongCount`, `Max`, `Min`, `Sum` | 计算聚合值。 |
| `TryGetNonEnumeratedCount` | `Count()`检查序列上是否实现了`Count`属性并返回其值，或者枚举整个序列以计算其项数。.NET 6 中新增了这个方法，它仅检查`Count`，如果缺失则返回`false`并将`out`参数设置为`0`，以避免潜在的性能不佳的操作。 |
| `All`, `Any`, `Contains` | 如果所有或任何项匹配筛选器，或者序列包含指定项，则返回`true`。 |
| `Cast` | 将项转换为指定类型。在编译器可能抱怨的情况下，将非泛型对象转换为泛型类型时非常有用。 |
| `OfType` | 移除与指定类型不匹配的项。 |
| `Distinct` | 移除重复项。 |
| `Except`, `Intersect`, `Union` | 执行返回集合的操作。集合不能有重复项。尽管输入可以是任何序列，因此输入可以有重复项，但结果始终是一个集合。 |
| `Chunk` | 将序列分割成定长批次。 |
| `Append`, `Concat`, `Prepend` | 执行序列合并操作。 |
| `Zip` | 基于项的位置对两个序列执行匹配操作，例如，第一个序列中位置 1 的项与第二个序列中位置 1 的项匹配。.NET 6 中新增了对三个序列的匹配操作。以前，您需要运行两次两个序列的重载才能达到相同目的。 |
| `ToArray`, `ToList`, `ToDictionary`, `ToHashSet`, `ToLookup` | 将序列转换为数组或集合。这些是唯一执行 LINQ 表达式的扩展方法。 |
| `DistinctBy`, `ExceptBy`, `IntersectBy`, `UnionBy`, `MinBy`, `MaxBy` | .NET 6 中新增了`By`扩展方法。它们允许在项的子集上进行比较，而不是整个项。例如，您可以仅通过比较他们的`LastName`和`DateOfBirth`来移除重复项，而不是通过比较整个`Person`对象。 |

`Enumerable`类还包含一些非扩展方法，如下表所示：

| 方法 | 描述 |
| --- | --- |
| `Empty<T>` | 返回指定类型`T`的空序列。它对于向需要`IEnumerable<T>`的方法传递空序列非常有用。 |
| `Range` | 从`start`值开始返回包含`count`个整数的序列。例如，`Enumerable.Range(start: 5, count: 3)`将包含整数 5、6 和 7。 |
| `Repeat` | 返回一个包含相同`element`重复`count`次的序列。例如，`Enumerable.Repeat(element: "5", count: 3)`将包含字符串值“5”、“5”和“5”。 |

### 理解延迟执行

LINQ 使用**延迟执行**。重要的是要理解，调用这些扩展方法中的大多数并不会执行查询并获取结果。这些扩展方法中的大多数返回一个代表*问题*而非*答案*的 LINQ 表达式。让我们来探讨：

1.  使用您偏好的代码编辑器创建一个名为`Chapter11`的新解决方案/工作区。

1.  添加一个控制台应用项目，如下表所定义：

    1.  项目模板：**控制台应用程序** / `console`

    1.  工作区/解决方案文件和文件夹：`Chapter11`

    1.  项目文件和文件夹：`LinqWithObjects`

1.  在`Program.cs`中，删除现有代码并静态导入`Console`。

1.  添加语句以定义一个`string`值序列，表示在办公室工作的人员，如下列代码所示：

    ```cs
    // a string array is a sequence that implements IEnumerable<string>
    string[] names = new[] { "Michael", "Pam", "Jim", "Dwight", 
      "Angela", "Kevin", "Toby", "Creed" };
    WriteLine("Deferred execution");
    // Question: Which names end with an M?
    // (written using a LINQ extension method)
    var query1 = names.Where(name => name.EndsWith("m"));
    // Question: Which names end with an M?
    // (written using LINQ query comprehension syntax)
    var query2 = from name in names where name.EndsWith("m") select name; 
    ```

1.  要提出问题并获得答案，即执行查询，您必须**具体化**它，通过调用诸如`ToArray`或`ToLookup`之类的“To”方法之一，或者通过枚举查询，如下列代码所示：

    ```cs
    // Answer returned as an array of strings containing Pam and Jim
    string[] result1 = query1.ToArray();
    // Answer returned as a list of strings containing Pam and Jim
    List<string> result2 = query2.ToList();
    // Answer returned as we enumerate over the results
    foreach (string name in query1)
    {
      WriteLine(name); // outputs Pam
      names[2] = "Jimmy"; // change Jim to Jimmy
      // on the second iteration Jimmy does not end with an M
    } 
    ```

1.  运行控制台应用并注意结果，如下所示：

    ```cs
    Deferred execution
    Pam 
    ```

由于延迟执行，在输出第一个结果`Pam`后，如果原始数组值发生改变，那么当我们再次循环时，将不再有匹配项，因为`Jim`已变为`Jimmy`，且不再以`M`结尾，因此只输出`Pam`。

在我们深入细节之前，让我们放慢脚步，逐一查看一些常见的 LINQ 扩展方法及其使用方法。

## 使用 Where 过滤实体

LINQ 最常见的用途是使用`Where`扩展方法对序列中的项进行过滤。让我们通过定义一个名字序列，然后对其应用 LINQ 操作来探索过滤：

1.  在项目文件中，注释掉启用隐式引用的元素，如下列标记中高亮所示：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
        <Nullable>enable</Nullable>
     **<!--<ImplicitUsings>enable</ImplicitUsings>-->**
      </PropertyGroup>
    </Project> 
    ```

1.  在`Program.cs`中，尝试对名字数组调用`Where`扩展方法，如下列代码所示：

    ```cs
    WriteLine("Writing queries"); 
    var query = names.W 
    ```

1.  当你尝试输入`Where`方法时，注意它从字符串数组的 IntelliSense 成员列表中缺失，如*图 11.1*所示：![](img/B17442_12_01.png)

    图 11.1：缺少 Where 扩展方法的 IntelliSense

    这是因为`Where`是一个扩展方法。它并不存在于数组类型上。为了使`Where`扩展方法可用，我们必须导入`System.Linq`命名空间。这在新的.NET 6 项目中默认是隐式导入的，但我们禁用了它。

1.  在项目文件中，取消注释启用隐式引用的元素。

1.  重新输入`Where`方法，并注意 IntelliSense 列表现在包括了由`Enumerable`类添加的扩展方法，如*图 11.2*所示：![](img/B17442_12_02.png)

    图 11.2：IntelliSense 显示 LINQ Enumerable 扩展方法

1.  当你输入`Where`方法的括号时，IntelliSense 告诉我们，要调用`Where`，我们必须传入一个`Func<string, bool>`委托的实例。

1.  输入一个表达式以创建`Func<string, bool>`委托的新实例，目前请注意我们尚未提供方法名，因为我们将在下一步定义它，如下列代码所示：

    ```cs
    var query = names.Where(new Func<string, bool>( )) 
    ```

`Func<string, bool>`委托告诉我们，对于传递给该方法的每个`string`变量，该方法必须返回一个`bool`值。如果方法返回`true`，则表示我们应该在结果中包含该`string`，如果方法返回`false`，则表示我们应该排除它。

## 针对命名方法

让我们定义一个只包含长度超过四个字符的名字的方法：

1.  在`Program.cs`底部，定义一个方法，该方法将只包含长度超过四个字符的名字，如下列代码所示：

    ```cs
    static bool NameLongerThanFour(string name)
    {
      return name.Length > 4;
    } 
    ```

1.  在`NameLongerThanFour`方法上方，将方法名传递给`Func<string, bool>`委托，然后遍历查询项，如下列高亮代码所示：

    ```cs
    var query = names.Where(
      new Func<string, bool>(**NameLongerThanFour**));
    **foreach** **(****string** **item** **in** **query)**
    **{**
     **WriteLine(item);**
    **}** 
    ```

1.  运行代码并查看结果，注意只有长度超过四个字母的名字被列出，如下列输出所示：

    ```cs
    Writing queries
    Michael 
    Dwight 
    Angela 
    Kevin 
    Creed 
    ```

## 通过移除显式委托实例化来简化代码

我们可以通过删除`Func<string, bool>`委托的显式实例化来简化代码，因为 C#编译器可以为我们实例化委托：

1.  为了帮助你通过逐步改进的代码学习，复制并粘贴查询

1.  注释掉第一个示例，如下面的代码所示：

    ```cs
    // var query = names.Where(
    //   new Func<string, bool>(NameLongerThanFour)); 
    ```

1.  修改副本以删除委托的显式实例化，如下面的代码所示：

    ```cs
    var query = names.Where(NameLongerThanFour); 
    ```

1.  运行代码并注意它具有相同的行为。

## 针对 lambda 表达式

我们可以使用**lambda 表达式**代替命名方法，进一步简化代码。

虽然一开始看起来可能很复杂，但 lambda 表达式只是一个*无名函数*。它使用`=>`（读作“转到”）符号来指示返回值：

1.  复制并粘贴查询，注释第二个示例，并修改查询，如下面的代码所示：

    ```cs
    var query = names.Where(name => name.Length > 4); 
    ```

    请注意，lambda 表达式的语法包括了`NameLongerThanFour`方法的所有重要部分，但仅此而已。lambda 表达式只需要定义以下内容：

    +   输入参数的名称：`name`

    +   返回值表达式：`name.Length > 4`

    `name`输入参数的类型是从序列包含`string`值这一事实推断出来的，并且返回类型必须是一个`bool`值，这是由`Where`工作的委托定义的，因此`=>`符号后面的表达式必须返回一个`bool`值。

    编译器为我们完成了大部分工作，因此我们的代码可以尽可能简洁。

1.  运行代码并注意它具有相同的行为。

## 对实体进行排序

其他常用的扩展方法是`OrderBy`和`ThenBy`，用于对序列进行排序。

如果前一个方法返回另一个序列，即实现`IEnumerable<T>`接口的类型，则可以链接扩展方法。

### 使用 OrderBy 按单个属性排序

让我们继续使用当前项目来探索排序：

1.  在现有查询的末尾添加对`OrderBy`的调用，如下面的代码所示：

    ```cs
    var query = names
      .Where(name => name.Length > 4)
      .OrderBy(name => name.Length); 
    ```

    **最佳实践**：将 LINQ 语句格式化，使每个扩展方法调用都发生在一行上，以便更容易阅读。

1.  运行代码并注意，现在名字按最短的先排序，如下面的输出所示：

    ```cs
    Kevin 
    Creed 
    Dwight 
    Angela 
    Michael 
    ```

要将最长的名字放在前面，您将使用`OrderByDescending`。

### 使用 ThenBy 按后续属性排序

我们可能希望按多个属性排序，例如，对相同长度的名字按字母顺序排序：

1.  在现有查询的末尾添加对`ThenBy`方法的调用，如下面的代码中突出显示的那样：

    ```cs
    var query = names
      .Where(name => name.Length > 4)
      .OrderBy(name => name.Length)
     **.ThenBy(name => name);** 
    ```

1.  运行代码并注意以下排序顺序的微小差异。在长度相同的名字组中，名字按`string`的完整值进行字母排序，因此`Creed`出现在`Kevin`之前，`Angela`出现在`Dwight`之前，如下面的输出所示：

    ```cs
    Creed 
    Kevin 
    Angela 
    Dwight 
    Michael 
    ```

## 使用 var 或指定类型声明查询

在编写 LINQ 表达式时，使用`var`声明查询对象很方便。这是因为随着您在 LINQ 表达式上的工作，类型经常发生变化。例如，我们的查询最初是`IEnumerable<string>`，目前是`IOrderedEnumerable<string>`：

1.  将鼠标悬停在`var`关键字上，并注意其类型为`IOrderedEnumerable<string>`

1.  将`var`替换为实际类型，如下面的代码中突出显示的那样：

    ```cs
    **IOrderedEnumerable<****string****>** query = names
      .Where(name => name.Length > 4)
      .OrderBy(name => name.Length)
      .ThenBy(name => name); 
    ```

**最佳实践**：一旦完成查询工作，您可以将声明的类型从`var`更改为实际类型，以使其更清楚地了解类型是什么。这很容易，因为您的代码编辑器可以告诉您它是什么。

## 按类型筛选

`Where`扩展方法非常适合按值筛选，例如文本和数字。但如果序列包含多种类型，并且您想要按特定类型筛选并尊重任何继承层次结构，该怎么办？

想象一下，您有一个异常序列。有数百种异常类型形成了一个复杂的层次结构，部分显示在*图 11.3*中：

![图表描述自动生成](img/B17442_12_04.png)

图 11.3：部分异常继承层次结构

让我们探讨按类型筛选：

1.  在`Program.cs`中，定义一个异常派生对象列表，如下面的代码所示：

    ```cs
    WriteLine("Filtering by type");
    List<Exception> exceptions = new()
    {
      new ArgumentException(), 
      new SystemException(),
      new IndexOutOfRangeException(),
      new InvalidOperationException(),
      new NullReferenceException(),
      new InvalidCastException(),
      new OverflowException(),
      new DivideByZeroException(),
      new ApplicationException()
    }; 
    ```

1.  使用`OfType<T>`扩展方法编写语句，以删除不是算术异常的异常，并将仅算术异常写入控制台，如下面的代码所示：

    ```cs
    IEnumerable<ArithmeticException> arithmeticExceptionsQuery = 
      exceptions.OfType<ArithmeticException>();
    foreach (ArithmeticException exception in arithmeticExceptionsQuery)
    {
      WriteLine(exception);
    } 
    ```

1.  运行代码并注意结果仅包括`ArithmeticException`类型的异常，或`ArithmeticException`派生的类型，如下面的输出所示：

    ```cs
    System.OverflowException: Arithmetic operation resulted in an overflow.
    System.DivideByZeroException: Attempted to divide by zero. 
    ```

## 使用 LINQ 处理集合和包

集合是数学中最基本的概念之一。**集合**是一个或多个唯一对象的集合。**多重集合**，又称**包**，是一个或多个对象的集合，可以有重复项。

您可能还记得在学校学过的维恩图。常见的集合操作包括集合之间的**交集**或**并集**。

让我们创建一个控制台应用程序，该应用程序将定义三个`string`值数组，用于学徒队列，然后对它们执行一些常见的集合和多重集合操作：

1.  使用您喜欢的代码编辑器，在`Chapter11`解决方案/工作区中添加一个名为`LinqWithSets`的新控制台应用程序：

    1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择。

    1.  在 Visual Studio Code 中，选择`LinqWithSets`作为活动 OmniSharp 项目。

1.  在`Program.cs`中，删除现有代码并静态导入`Console`类型，如下面的代码所示：

    ```cs
    using static System.Console; 
    ```

1.  在`Program.cs`底部，添加以下方法，该方法将任何`string`变量序列输出为以逗号分隔的单个`string`到控制台输出，以及一个可选描述，如下面的代码所示：

    ```cs
    static void Output(IEnumerable<string> cohort, string description = "")
    {
      if (!string.IsNullOrEmpty(description))
      {
        WriteLine(description);
      }
      Write(" ");
      WriteLine(string.Join(", ", cohort.ToArray()));
      WriteLine();
    } 
    ```

1.  在`Output`方法上方，添加语句以定义三个名称数组，输出它们，然后对它们执行各种集合操作，如下面的代码所示：

    ```cs
    string[] cohort1 = new[]
      { "Rachel", "Gareth", "Jonathan", "George" }; 
    string[] cohort2 = new[]
      { "Jack", "Stephen", "Daniel", "Jack", "Jared" }; 
    string[] cohort3 = new[]
      { "Declan", "Jack", "Jack", "Jasmine", "Conor" }; 
    Output(cohort1, "Cohort 1");
    Output(cohort2, "Cohort 2");
    Output(cohort3, "Cohort 3"); 
    Output(cohort2.Distinct(), "cohort2.Distinct()"); 
    Output(cohort2.DistinctBy(name => name.Substring(0, 2)), 
      "cohort2.DistinctBy(name => name.Substring(0, 2)):");
    Output(cohort2.Union(cohort3), "cohort2.Union(cohort3)"); 
    Output(cohort2.Concat(cohort3), "cohort2.Concat(cohort3)"); 
    Output(cohort2.Intersect(cohort3), "cohort2.Intersect(cohort3)"); 
    Output(cohort2.Except(cohort3), "cohort2.Except(cohort3)"); 
    Output(cohort1.Zip(cohort2,(c1, c2) => $"{c1} matched with {c2}"), 
      "cohort1.Zip(cohort2)"); 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Cohort 1
      Rachel, Gareth, Jonathan, George 
    Cohort 2
      Jack, Stephen, Daniel, Jack, Jared 
    Cohort 3
      Declan, Jack, Jack, Jasmine, Conor 
    cohort2.Distinct()
      Jack, Stephen, Daniel, Jared 
    cohort2.DistinctBy(name => name.Substring(0, 2)):
      Jack, Stephen, Daniel 
    cohort2.Union(cohort3)
      Jack, Stephen, Daniel, Jared, Declan, Jasmine, Conor 
    cohort2.Concat(cohort3)
      Jack, Stephen, Daniel, Jack, Jared, Declan, Jack, Jack, Jasmine, Conor 
    cohort2.Intersect(cohort3)
      Jack 
    cohort2.Except(cohort3)
      Stephen, Daniel, Jared 
    cohort1.Zip(cohort2)
      Rachel matched with Jack, Gareth matched with Stephen, Jonathan matched with Daniel, George matched with Jack 
    ```

使用`Zip`时，如果两个序列中的项数不相等，那么有些项将没有匹配的伙伴。没有伙伴的项，如`Jared`，将不会包含在结果中。

对于`DistinctBy`示例，我们不是通过比较整个名称来移除重复项，而是定义了一个 lambda 键选择器，通过比较前两个字符来移除重复项，因此`Jared`被移除，因为`Jack`已经是一个以`Ja`开头的名称。

到目前为止，我们使用了 LINQ to Objects 提供程序来处理内存中的对象。接下来，我们将使用 LINQ to Entities 提供程序来处理存储在数据库中的实体。

# 使用 LINQ 与 EF Core

我们已经看过过滤和排序的 LINQ 查询，但没有改变序列中项的形状的查询。这称为**投影**，因为它涉及将一种形状的项投影到另一种形状。为了学习投影，最好有一些更复杂的类型来操作，所以在下一个项目中，我们将不再使用`string`序列，而是使用来自 Northwind 示例数据库的实体序列。

我将给出使用 SQLite 的指令，因为它跨平台，但如果你更喜欢使用 SQL Server，请随意。我已包含一些注释代码，以便在你选择时启用 SQL Server。

## 构建 EF Core 模型

我们必须定义一个 EF Core 模型来表示我们将要操作的数据库和表。我们将手动定义模型以完全控制并防止在`Categories`和`Products`表之间自动定义关系。稍后，您将使用 LINQ 来连接这两个实体集：

1.  使用您喜欢的代码编辑器向`Chapter11`解决方案/工作区中添加一个名为`LinqWithEFCore`的新控制台应用程序。

1.  在 Visual Studio Code 中，选择`LinqWithEFCore`作为活动 OmniSharp 项目。

1.  在`LinqWithEFCore`项目中，添加对 SQLite 和/或 SQL Server 的 EF Core 提供程序的包引用，如下所示：

    ```cs
    <ItemGroup>
      <PackageReference
        Include="Microsoft.EntityFrameworkCore.Sqlite"
        Version="6.0.0" />
      <PackageReference
        Include="Microsoft.EntityFrameworkCore.SqlServer"
        Version="6.0.0" />
    </ItemGroup> 
    ```

1.  构建项目以恢复包。

1.  将`Northwind4Sqlite.sql`文件复制到`LinqWithEFCore`文件夹中。

1.  在命令提示符或终端中，执行以下命令创建 Northwind 数据库：

    ```cs
    sqlite3 Northwind.db -init Northwind4Sqlite.sql 
    ```

1.  请耐心等待，因为这个命令可能需要一段时间来创建数据库结构。最终，您将看到 SQLite 命令提示符，如下所示：

    ```cs
     -- Loading resources from Northwind.sql 
    SQLite version 3.36.0 2021-08-02 15:20:15
    Enter ".help" for usage hints.
    sqlite> 
    ```

1.  在 macOS 上按 cmd + D 或在 Windows 上按 Ctrl + C 退出 SQLite 命令模式。

1.  向项目中添加三个类文件，分别命名为`Northwind.cs`、`Category.cs`和`Product.cs`。

1.  修改名为`Northwind.cs`的类文件，如下所示：

    ```cs
    using Microsoft.EntityFrameworkCore; // DbContext, DbSet<T>
    namespace Packt.Shared;
    // this manages the connection to the database
    public class Northwind : DbContext
    {
      // these properties map to tables in the database
      public DbSet<Category>? Categories { get; set; }
      public DbSet<Product>? Products { get; set; }
      protected override void OnConfiguring(
        DbContextOptionsBuilder optionsBuilder)
      {
        string path = Path.Combine(
          Environment.CurrentDirectory, "Northwind.db");
        optionsBuilder.UseSqlite($"Filename={path}");
        /*
        string connection = "Data Source=.;" +
            "Initial Catalog=Northwind;" +
            "Integrated Security=true;" +
            "MultipleActiveResultSets=true;";
        optionsBuilder.UseSqlServer(connection);
        */
      }
      protected override void OnModelCreating(
        ModelBuilder modelBuilder)
      {
        modelBuilder.Entity<Product>()
          .Property(product => product.UnitPrice)
          .HasConversion<double>();
      }
    } 
    ```

1.  修改名为`Category.cs`的类文件，如下所示：

    ```cs
    using System.ComponentModel.DataAnnotations;
    namespace Packt.Shared;
    public class Category
    {
      public int CategoryId { get; set; }
      [Required]
      [StringLength(15)]
      public string CategoryName { get; set; } = null!;
      public string? Description { get; set; }
    } 
    ```

1.  修改名为`Product.cs`的类文件，如下所示：

    ```cs
    using System.ComponentModel.DataAnnotations; 
    using System.ComponentModel.DataAnnotations.Schema;
    namespace Packt.Shared;
    public class Product
    {
      public int ProductId { get; set; }
      [Required]
      [StringLength(40)]
      public string ProductName { get; set; } = null!;
      public int? SupplierId { get; set; }
      public int? CategoryId { get; set; }
      [StringLength(20)]
      public string? QuantityPerUnit { get; set; }
      [Column(TypeName = "money")] // required for SQL Server provider
      public decimal? UnitPrice { get; set; }
      public short? UnitsInStock { get; set; }
      public short? UnitsOnOrder { get; set; }
      public short? ReorderLevel { get; set; }
      public bool Discontinued { get; set; }
    } 
    ```

1.  构建项目并修复任何编译器错误。

    如果您使用的是 Windows 上的 Visual Studio 2022，那么编译后的应用程序将在`LinqWithEFCore\bin\Debug\net6.0`文件夹中执行，因此除非我们指示应始终将其复制到输出目录，否则它将找不到数据库文件。

1.  在**解决方案资源管理器**中，右键单击`Northwind.db`文件并选择**属性**。

1.  在**属性**中，将**复制到输出目录**设置为**始终复制**。

## 过滤和排序序列

现在让我们编写语句来过滤和排序来自表的行序列：

1.  在`Program.cs`中，静态导入`Console`类型和用于使用 EF Core 和实体模型进行 LINQ 操作的命名空间，如下列代码所示：

    ```cs
    using Packt.Shared; // Northwind, Category, Product
    using Microsoft.EntityFrameworkCore; // DbSet<T>
    using static System.Console; 
    ```

1.  在`Program.cs`底部，编写一个方法来过滤和排序产品，如下列代码所示：

    ```cs
    static void FilterAndSort()
    {
      using (Northwind db = new())
      {
        DbSet<Product> allProducts = db.Products;
        IQueryable<Product> filteredProducts = 
          allProducts.Where(product => product.UnitPrice < 10M);
        IOrderedQueryable<Product> sortedAndFilteredProducts = 
          filteredProducts.OrderByDescending(product => product.UnitPrice);
        WriteLine("Products that cost less than $10:");
        foreach (Product p in sortedAndFilteredProducts)
        {
          WriteLine("{0}: {1} costs {2:$#,##0.00}",
            p.ProductId, p.ProductName, p.UnitPrice);
        }
        WriteLine();
      }
    } 
    ```

    `DbSet<T>`实现`IEnumerable<T>`，因此 LINQ 可用于查询和操作为 EF Core 构建的模型中的实体集合。（实际上，我应该说`TEntity`而不是`T`，但此泛型类型的名称没有功能性影响。唯一的要求是类型是一个`class`。名称仅表示预期该类是一个实体模型。）

    您可能还注意到，序列实现的是`IQueryable<T>`（或在调用排序 LINQ 方法后实现`IOrderedQueryable<T>`）而不是`IEnumerable<T>`或`IOrderedEnumerable<T>`。

    这表明我们正在使用一个 LINQ 提供程序，该提供程序使用表达式树在内存中构建查询。它们以树状数据结构表示代码，并支持创建动态查询，这对于构建针对 SQLite 等外部数据提供程序的 LINQ 查询非常有用。

    LINQ 表达式将被转换成另一种查询语言，如 SQL。使用`foreach`枚举查询或调用`ToArray`等方法将强制执行查询并具体化结果。

1.  在`Program.cs`中的命名空间导入之后，调用`FilterAndSort`方法。

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    Products that cost less than $10:
    41: Jack's New England Clam Chowder costs $9.65 
    45: Rogede sild costs $9.50
    47: Zaanse koeken costs $9.50
    19: Teatime Chocolate Biscuits costs $9.20 
    23: Tunnbröd costs $9.00
    75: Rhönbräu Klosterbier costs $7.75 
    54: Tourtière costs $7.45
    52: Filo Mix costs $7.00 
    13: Konbu costs $6.00
    24: Guaraná Fantástica costs $4.50 
    33: Geitost costs $2.50 
    ```

尽管此查询输出了我们所需的信息，但这样做效率低下，因为它从`Products`表中获取了所有列，而不是我们需要的三个列，这相当于以下 SQL 语句：

```cs
SELECT * FROM Products; 
```

在*第十章*，*使用 Entity Framework Core 处理数据*中，您学习了如何记录针对 SQLite 执行的 SQL 命令，以便您可以亲自查看。

## 将序列投影到新类型

在查看投影之前，我们需要回顾对象初始化语法。如果您定义了一个类，那么您可以使用类名、`new()`和花括号来设置字段和属性的初始值，如下列代码所示：

```cs
public class Person
{
  public string Name { get; set; }
  public DateTime DateOfBirth { get; set; }
}
Person knownTypeObject = new()
{
  Name = "Boris Johnson",
  DateOfBirth = new(year: 1964, month: 6, day: 19)
}; 
```

C# 3.0 及更高版本允许使用`var`关键字实例化**匿名类型**，如下列代码所示：

```cs
var anonymouslyTypedObject = new
{
  Name = "Boris Johnson",
  DateOfBirth = new DateTime(year: 1964, month: 6, day: 19)
}; 
```

尽管我们没有指定类型，但编译器可以从设置的两个属性`Name`和`DateOfBirth`推断出匿名类型。编译器可以从分配的值推断出这两个属性的类型：一个字符串字面量和一个新的日期/时间值实例。

当编写 LINQ 查询以将现有类型投影到新类型而不必显式定义新类型时，此功能特别有用。由于类型是匿名的，因此这只能与`var`声明的局部变量一起工作。

让我们通过添加对`Select`方法的调用，将`Product`类的实例投影到仅具有三个属性的新匿名类型的实例，从而使针对数据库表执行的 SQL 命令更高效：

1.  在`FilterAndSort`中，添加一条语句以扩展 LINQ 查询，使用`Select`方法仅返回我们需要的三个属性（即表列），并修改`foreach`语句以使用`var`关键字和投影 LINQ 表达式，如下所示高亮显示：

    ```cs
    IOrderedQueryable<Product> sortedAndFilteredProducts = 
      filteredProducts.OrderByDescending(product => product.UnitPrice);
    **var** **projectedProducts = sortedAndFilteredProducts**
     **.Select(product =>** **new****// anonymous type**
     **{**
     **product.ProductId,**
     **product.ProductName,** 
     **product.UnitPrice**
     **});**
    WriteLine("Products that cost less than $10:");
    foreach (**var** **p** **in** **projectedProducts**)
    { 
    ```

1.  将鼠标悬停在`Select`方法调用中的`new`关键字和`foreach`语句中的`var`关键字上，并注意它是一个匿名类型，如*图 11.4*所示：![](img/B17442_12_05.png)

    *图 11.4*：LINQ 投影期间使用的匿名类型

1.  运行代码并确认输出与之前相同。

## 连接和分组序列

连接和分组有两种扩展方法：

+   **Join**：此方法有四个参数：您想要连接的序列，要匹配的*左侧*序列上的属性或属性，要匹配的*右侧*序列上的属性或属性，以及一个投影。

+   **GroupJoin**：此方法具有相同的参数，但它将匹配项合并到一个组对象中，该对象具有用于匹配值的`Key`属性和用于多个匹配项的`IEnumerable<T>`类型。

### 连接序列

让我们在处理两个表：`Categories`和`Products`时探索这些方法：

1.  在`Program.cs`底部，创建一个方法来选择类别和产品，将它们连接起来并输出，如下所示：

    ```cs
    static void JoinCategoriesAndProducts()
    {
      using (Northwind db = new())
      {
        // join every product to its category to return 77 matches
        var queryJoin = db.Categories.Join(
          inner: db.Products,
          outerKeySelector: category => category.CategoryId,
          innerKeySelector: product => product.CategoryId,
          resultSelector: (c, p) =>
            new { c.CategoryName, p.ProductName, p.ProductId });
        foreach (var item in queryJoin)
        {
          WriteLine("{0}: {1} is in {2}.",
            arg0: item.ProductId,
            arg1: item.ProductName,
            arg2: item.CategoryName);
        }
      }
    } 
    ```

    在连接中，有两个序列，*外部*和*内部*。在前面的示例中，`categories`是外部序列，`products`是内部序列。

1.  在`Program.cs`顶部，注释掉对`FilterAndSort`的调用，改为调用`JoinCategoriesAndProducts`。

1.  运行代码并查看结果。请注意，对于 77 种产品中的每一种，都有一行输出，如下所示的输出（编辑后仅包括前 10 项）：

    ```cs
    1: Chai is in Beverages. 
    2: Chang is in Beverages.
    3: Aniseed Syrup is in Condiments.
    4: Chef Anton's Cajun Seasoning is in Condiments. 
    5: Chef Anton's Gumbo Mix is in Condiments.
    6: Grandma's Boysenberry Spread is in Condiments. 
    7: Uncle Bob's Organic Dried Pears is in Produce. 
    8: Northwoods Cranberry Sauce is in Condiments.
    9: Mishi Kobe Niku is in Meat/Poultry. 
    10: Ikura is in Seafood.
    ... 
    ```

1.  在现有查询的末尾，调用`OrderBy`方法按`CategoryName`排序，如下所示：

    ```cs
    .OrderBy(cp => cp.CategoryName); 
    ```

1.  运行代码并查看结果。请注意，对于 77 种产品中的每一种，都有一行输出，结果首先显示`Beverages`类别中的所有产品，然后是`Condiments`类别，依此类推，如下所示的部分输出：

    ```cs
    1: Chai is in Beverages. 
    2: Chang is in Beverages.
    24: Guaraná Fantástica is in Beverages. 
    34: Sasquatch Ale is in Beverages.
    35: Steeleye Stout is in Beverages. 
    38: Côte de Blaye is in Beverages. 
    39: Chartreuse verte is in Beverages. 
    43: Ipoh Coffee is in Beverages.
    67: Laughing Lumberjack Lager is in Beverages. 
    70: Outback Lager is in Beverages.
    75: Rhönbräu Klosterbier is in Beverages. 
    76: Lakkalikööri is in Beverages.
    3: Aniseed Syrup is in Condiments.
    4: Chef Anton's Cajun Seasoning is in Condiments.
    ... 
    ```

### 分组连接序列

1.  在`Program.cs`底部，创建一个方法来分组和连接，显示组名，然后显示每个组内的所有项，如下列代码所示：

    ```cs
    static void GroupJoinCategoriesAndProducts()
    {
      using (Northwind db = new())
      {
        // group all products by their category to return 8 matches
        var queryGroup = db.Categories.AsEnumerable().GroupJoin(
          inner: db.Products,
          outerKeySelector: category => category.CategoryId,
          innerKeySelector: product => product.CategoryId,
          resultSelector: (c, matchingProducts) => new
          {
            c.CategoryName,
            Products = matchingProducts.OrderBy(p => p.ProductName)
          });
        foreach (var category in queryGroup)
        {
          WriteLine("{0} has {1} products.",
            arg0: category.CategoryName,
            arg1: category.Products.Count());
          foreach (var product in category.Products)
          {
            WriteLine($" {product.ProductName}");
          }
        }
      }
    } 
    ```

    如果我们没有调用`AsEnumerable`方法，那么将会抛出一个运行时异常，如下列输出所示：

    ```cs
    Unhandled exception. System.ArgumentException:  Argument type 'System.Linq.IOrderedQueryable`1[Packt.Shared.Product]' does not match the corresponding member type 'System.Linq.IOrderedEnumerable`1[Packt.Shared.Product]' (Parameter 'arguments[1]') 
    ```

    这是因为并非所有 LINQ 扩展方法都能从表达式树转换为其他查询语法，如 SQL。在这些情况下，我们可以通过调用`AsEnumerable`方法将`IQueryable<T>`转换为`IEnumerable<T>`，这迫使查询处理仅使用 LINQ to EF Core 将数据带入应用程序，然后使用 LINQ to Objects 在内存中执行更复杂的处理。但通常，这效率较低。

1.  在`Program.cs`顶部，注释掉之前的方法调用，并调用`GroupJoinCategoriesAndProducts`。

1.  运行代码，查看结果，并注意每个类别内的产品已按其名称排序，正如查询中所定义，并在以下部分输出中所示：

    ```cs
    Beverages has 12 products.
      Chai
      Chang
      Chartreuse verte
      Côte de Blaye
      Guaraná Fantástica
      Ipoh Coffee
      Lakkalikööri
      Laughing Lumberjack Lager
      Outback Lager
      Rhönbräu Klosterbier
      Sasquatch Ale
      Steeleye Stout
    Condiments has 12 products.
      Aniseed Syrup
      Chef Anton's Cajun Seasoning
      Chef Anton's Gumbo Mix
    ... 
    ```

## 聚合序列

有 LINQ 扩展方法可执行聚合函数，如`Average`和`Sum`。让我们编写一些代码，看看这些方法如何从`Products`表中聚合信息：

1.  在`Program.cs`底部，创建一个方法来展示聚合扩展方法的使用，如下列代码所示：

    ```cs
    static void AggregateProducts()
    {
      using (Northwind db = new())
      {
        WriteLine("{0,-25} {1,10}",
          arg0: "Product count:",
          arg1: db.Products.Count());
        WriteLine("{0,-25} {1,10:$#,##0.00}",
          arg0: "Highest product price:",
          arg1: db.Products.Max(p => p.UnitPrice));
        WriteLine("{0,-25} {1,10:N0}",
          arg0: "Sum of units in stock:",
          arg1: db.Products.Sum(p => p.UnitsInStock));
        WriteLine("{0,-25} {1,10:N0}",
          arg0: "Sum of units on order:",
          arg1: db.Products.Sum(p => p.UnitsOnOrder));
        WriteLine("{0,-25} {1,10:$#,##0.00}",
          arg0: "Average unit price:",
          arg1: db.Products.Average(p => p.UnitPrice));
        WriteLine("{0,-25} {1,10:$#,##0.00}",
          arg0: "Value of units in stock:",
          arg1: db.Products
            .Sum(p => p.UnitPrice * p.UnitsInStock));
      }
    } 
    ```

1.  在`Program.cs`顶部，注释掉之前的方法调用，并调用`AggregateProducts`

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    Product count:                    77
    Highest product price:       $263.50
    Sum of units in stock:         3,119
    Sum of units on order:           780
    Average unit price:           $28.87
    Value of units in stock:  $74,050.85 
    ```

# 用语法糖美化 LINQ 语法

C# 3.0 在 2008 年引入了一些新的语言关键字，以便有 SQL 经验的程序员更容易编写 LINQ 查询。这种语法糖有时被称为**LINQ 查询理解语法**。

考虑以下`string`值数组：

```cs
string[] names = new[] { "Michael", "Pam", "Jim", "Dwight", 
  "Angela", "Kevin", "Toby", "Creed" }; 
```

要筛选和排序名称，可以使用扩展方法和 lambda 表达式，如下列代码所示：

```cs
var query = names
  .Where(name => name.Length > 4)
  .OrderBy(name => name.Length)
  .ThenBy(name => name); 
```

或者，你可以使用查询理解语法实现相同的结果，如下列代码所示：

```cs
var query = from name in names
  where name.Length > 4
  orderby name.Length, name 
  select name; 
```

编译器会将查询理解语法转换为等效的扩展方法和 lambda 表达式。

`select`关键字在 LINQ 查询理解语法中始终是必需的。当使用扩展方法和 lambda 表达式时，`Select`扩展方法是可选的，因为如果你没有调用`Select`，那么整个项会被隐式选中。

并非所有扩展方法都有 C#关键字等效项，例如，常用的`Skip`和`Take`扩展方法，用于为大量数据实现分页。

使用查询理解语法无法编写跳过和获取的查询，因此我们可以使用所有扩展方法编写查询，如下列代码所示：

```cs
var query = names
  .Where(name => name.Length > 4)
  .Skip(80)
  .Take(10); 
```

或者，你可以将查询理解语法括在括号内，然后切换到使用扩展方法，如下列代码所示：

```cs
var query = (from name in names
  where name.Length > 4
  select name)
  .Skip(80)
  .Take(10); 
```

**良好实践**：学习使用 Lambda 表达式的扩展方法和查询理解语法两种编写 LINQ 查询的方式，因为你可能需要维护使用这两种方式的代码。

# 使用并行 LINQ 的多线程

默认情况下，LINQ 查询仅使用一个线程执行。**并行 LINQ**（**PLINQ**）是一种启用多个线程执行 LINQ 查询的简便方法。

**良好实践**：不要假设使用并行线程会提高应用程序的性能。始终测量实际的计时和资源使用情况。

## 创建一个从多线程中受益的应用

为了实际演示，我们将从一段仅使用单个线程计算 45 个整数的斐波那契数的代码开始。我们将使用`StopWatch`类型来测量性能变化。

我们将使用操作系统工具来监控 CPU 和 CPU 核心的使用情况。如果你没有多个 CPU 或至少多个核心，那么这个练习就不会显示太多信息！

1.  使用你偏好的代码编辑器，在`Chapter11`解决方案/工作区中添加一个名为`LinqInParallel`的新控制台应用。

1.  在 Visual Studio Code 中，选择`LinqInParallel`作为活动的 OmniSharp 项目。

1.  在`Program.cs`中，删除现有语句，然后导入`System.Diagnostics`命名空间，以便我们可以使用`StopWatch`类型，并静态导入`System.Console`类型。

1.  添加语句以创建一个秒表来记录时间，等待按键开始计时，创建 45 个整数，计算每个整数的最后一个斐波那契数，停止计时器，并显示经过的毫秒数，如下面的代码所示：

    ```cs
    Stopwatch watch = new(); 
    Write("Press ENTER to start. "); 
    ReadLine();
    watch.Start();
    int max = 45;
    IEnumerable<int> numbers = Enumerable.Range(start: 1, count: max);
    WriteLine($"Calculating Fibonacci sequence up to {max}. Please wait...");
    int[] fibonacciNumbers = numbers
      .Select(number => Fibonacci(number)).ToArray(); 
    watch.Stop();
    WriteLine("{0:#,##0} elapsed milliseconds.",
      arg0: watch.ElapsedMilliseconds);
    Write("Results:");
    foreach (int number in fibonacciNumbers)
    {
      Write($" {number}");
    }
    static int Fibonacci(int term) =>
      term switch
      {
        1 => 0,
        2 => 1,
        _ => Fibonacci(term - 1) + Fibonacci(term - 2)
      }; 
    ```

1.  运行代码，但不要按 Enter 键启动秒表，因为我们首先需要确保监控工具显示处理器活动。

### 使用 Windows

1.  如果你使用的是 Windows，那么右键点击 Windows **开始**按钮或按 Ctrl + Alt + Delete，然后点击**任务管理器**。

1.  在**任务管理器**窗口底部，点击**更多详细信息**。

1.  在**任务管理器**窗口顶部，点击**性能**选项卡。

1.  右键点击**CPU 利用率**图表，选择**更改图表为**，然后选择**逻辑处理器**。

### 使用 macOS

1.  如果你使用的是 macOS，那么启动**活动监视器**。

1.  导航至**视图** | **更新频率非常频繁（1 秒）**。

1.  要查看 CPU 图表，请导航至**窗口** | **CPU 历史**。

### 对于所有操作系统

1.  调整你的监控工具和代码编辑器，使它们并排显示。

1.  等待 CPU 稳定后，按 Enter 键启动秒表并运行查询。结果应显示为经过的毫秒数，如下面的输出所示：

    ```cs
    Press ENTER to start. 
    Calculating Fibonacci sequence up to 45\. Please wait...
    17,624 elapsed milliseconds.
    Results: 0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584 4181 6765 10946 17711 28657 46368 75025 121393 196418 317811 514229 832040 1346269 2178309 3524578 5702887 9227465 14930352 24157817 39088169 63245986 102334155 165580141 267914296 433494437 701408733 
    ```

    监控工具可能会显示，有一两个 CPU 使用率最高，随着时间交替变化，其他 CPU 可能同时执行后台任务，如垃圾收集器，因此其他 CPU 或核心不会完全空闲，但工作显然没有均匀分布在所有可能的 CPU 或核心上。还要注意，一些逻辑处理器达到了 100%的峰值。

1.  在`Program.cs`中，修改查询以调用`AsParallel`扩展方法并对结果序列进行排序，因为在并行处理时结果可能会变得无序，如下面的代码所示：

    ```cs
    int[] fibonacciNumbers = numbers.**AsParallel()**
      .Select(number => Fibonacci(number))
     **.OrderBy(number => number)**
      .ToArray(); 
    ```

    **最佳实践**：切勿在查询的末尾调用`AsParallel`。这没有任何作用。你必须在调用`AsParallel`之后至少执行一个操作，以便该操作可以并行化。.NET 6 引入了一个代码分析器，它会警告这种误用。

1.  运行代码，等待监控工具中的 CPU 图表稳定，然后按 Enter 键启动秒表并运行查询。这次，应用程序应该在更短的时间内完成（尽管可能不会像你希望的那样短——管理那些多线程需要额外的努力！）：

    ```cs
    Press ENTER to start. 
    Calculating Fibonacci sequence up to 45\. Please wait...
    9,028 elapsed milliseconds.
    Results: 0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584 4181 6765 10946 17711 28657 46368 75025 121393 196418 317811 514229 832040 1346269 2178309 3524578 5702887 9227465 14930352 24157817 39088169 63245986 102334155 165580141 267914296 433494437 701408733 
    ```

1.  监控工具应该显示所有 CPU 都平均用于执行 LINQ 查询，并注意没有逻辑处理器达到 100%的峰值，因为工作分布更为均匀。

你将在*第十二章*，*使用多任务提高性能和可扩展性*中了解更多关于管理多线程的知识。

# 创建自己的 LINQ 扩展方法

在*第六章*，*实现接口和继承类*中，你学习了如何创建自己的扩展方法。要创建 LINQ 扩展方法，你所需要做的就是扩展`IEnumerable<T>`类型。

**最佳实践**：将你自己的扩展方法放在一个单独的类库中，以便它们可以轻松地作为自己的程序集或 NuGet 包部署。

我们将以改进`Average`扩展方法为例。一个受过良好教育的学童会告诉你，*平均*可以指三种情况之一：

+   **均值**：将数字求和并除以计数。

+   **众数**：最常见的数字。

+   **中位数**：当数字排序时位于中间的数字。

微软实现的`Average`扩展方法计算的是*均值*。我们可能希望为`Mode`和`Median`定义自己的扩展方法：

1.  在`LinqWithEFCore`项目中，添加一个名为`MyLinqExtensions.cs`的新类文件。

1.  按照以下代码所示修改类：

    ```cs
    namespace System.Linq; // extend Microsoft's namespace
    public static class MyLinqExtensions
    {
      // this is a chainable LINQ extension method
      public static IEnumerable<T> ProcessSequence<T>(
        this IEnumerable<T> sequence)
      {
        // you could do some processing here
        return sequence;
      }
      public static IQueryable<T> ProcessSequence<T>(
        this IQueryable<T> sequence)
      {
        // you could do some processing here
        return sequence;
      }
      // these are scalar LINQ extension methods
      public static int? Median(
        this IEnumerable<int?> sequence)
      {
        var ordered = sequence.OrderBy(item => item);
        int middlePosition = ordered.Count() / 2;
        return ordered.ElementAt(middlePosition);
      }
      public static int? Median<T>(
        this IEnumerable<T> sequence, Func<T, int?> selector)
      {
        return sequence.Select(selector).Median();
      }
      public static decimal? Median(
        this IEnumerable<decimal?> sequence)
      {
        var ordered = sequence.OrderBy(item => item);
        int middlePosition = ordered.Count() / 2;
        return ordered.ElementAt(middlePosition);
      }
      public static decimal? Median<T>(
        this IEnumerable<T> sequence, Func<T, decimal?> selector)
      {
        return sequence.Select(selector).Median();
      }
      public static int? Mode(
        this IEnumerable<int?> sequence)
      {
        var grouped = sequence.GroupBy(item => item);
        var orderedGroups = grouped.OrderByDescending(
          group => group.Count());
        return orderedGroups.FirstOrDefault()?.Key;
      }
      public static int? Mode<T>(
        this IEnumerable<T> sequence, Func<T, int?> selector)
      {
        return sequence.Select(selector)?.Mode();
      }
      public static decimal? Mode(
        this IEnumerable<decimal?> sequence)
      {
        var grouped = sequence.GroupBy(item => item);
        var orderedGroups = grouped.OrderByDescending(
          group => group.Count());
        return orderedGroups.FirstOrDefault()?.Key;
      }
      public static decimal? Mode<T>(
        this IEnumerable<T> sequence, Func<T, decimal?> selector)
      {
        return sequence.Select(selector).Mode();
      }
    } 
    ```

如果这个类位于一个单独的类库中，要使用你的 LINQ 扩展方法，你只需引用类库程序集，因为`System.Linq`命名空间已经隐式导入。

**警告！**上述扩展方法中除一个外，都不能与`IQueryable`序列（如 LINQ to SQLite 或 LINQ to SQL Server 使用的序列）一起使用，因为我们没有实现将我们的代码翻译成底层查询语言（如 SQL）的方法。

### 尝试使用链式扩展方法

首先，我们将尝试将`ProcessSequence`方法与其他扩展方法链接起来：

1.  在`Program.cs`中，在`FilterAndSort`方法中，修改`Products`的 LINQ 查询以调用您的自定义链式扩展方法，如下面的代码中突出显示的那样：

    ```cs
    DbSet<Product>? allProducts = db.Products;
    if (allProducts is null)
    {
      WriteLine("No products found.");
      return;
    }
    **IQueryable<Product> processedProducts = allProducts.ProcessSequence();**
    IQueryable<Product> filteredProducts = **processedProducts**
      .Where(product => product.UnitPrice < 10M); 
    ```

1.  在`Program.cs`中，取消注释`FilterAndSort`方法，并注释掉对其他方法的任何调用。

1.  运行代码并注意您看到与之前相同的输出，因为您的方法没有修改序列。但现在您知道如何通过自己的功能扩展 LINQ 表达式。

### 尝试使用众数和中位数方法

其次，我们将尝试使用`Mode`和`Median`方法来计算其他类型的平均值：

1.  在`Program.cs`底部，创建一个方法来输出产品的`UnitsInStock`和`UnitPrice`的平均值、中位数和众数，使用您的自定义扩展方法和内置的`Average`扩展方法，如下面的代码所示：

    ```cs
    static void CustomExtensionMethods()
    {
      using (Northwind db = new())
      {
        WriteLine("Mean units in stock: {0:N0}",
          db.Products.Average(p => p.UnitsInStock));
        WriteLine("Mean unit price: {0:$#,##0.00}",
          db.Products.Average(p => p.UnitPrice));
        WriteLine("Median units in stock: {0:N0}",
          db.Products.Median(p => p.UnitsInStock));
        WriteLine("Median unit price: {0:$#,##0.00}",
          db.Products.Median(p => p.UnitPrice));
        WriteLine("Mode units in stock: {0:N0}",
          db.Products.Mode(p => p.UnitsInStock));
        WriteLine("Mode unit price: {0:$#,##0.00}",
          db.Products.Mode(p => p.UnitPrice));
      }
    } 
    ```

1.  在`Program.cs`中，注释掉任何之前的方法调用，并调用`CustomExtensionMethods`。

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Mean units in stock: 41 
    Mean unit price: $28.87 
    Median units in stock: 26 
    Median unit price: $19.50 
    Mode units in stock: 0 
    Mode unit price: $18.00 
    ```

有四种产品的单价为$18.00。有五种产品的库存量为 0。

# 使用 LINQ to XML 进行工作

**LINQ to XML**是一种 LINQ 提供程序，允许您查询和操作 XML。

## 使用 LINQ to XML 生成 XML

让我们创建一个方法将`Products`表转换为 XML：

1.  在`LinqWithEFCore`项目中，在`Program.cs`顶部导入`System.Xml.Linq`命名空间。

1.  在`Program.cs`底部，创建一个方法以 XML 格式输出产品，如下面的代码所示：

    ```cs
    static void OutputProductsAsXml()
    {
      using (Northwind db = new())
      {
        Product[] productsArray = db.Products.ToArray();
        XElement xml = new("products",
          from p in productsArray
          select new XElement("product",
            new XAttribute("id",  p.ProductId),
            new XAttribute("price", p.UnitPrice),
           new XElement("name", p.ProductName)));
        WriteLine(xml.ToString());
      }
    } 
    ```

1.  在`Program.cs`中，注释掉之前的方法调用，并调用`OutputProductsAsXml`。

1.  运行代码，查看结果，并注意生成的 XML 结构与 LINQ to XML 语句在前述代码中声明性地描述的元素和属性相匹配，如下面的部分输出所示：

    ```cs
    <products>
      <product id="1" price="18">
        <name>Chai</name>
      </product>
      <product id="2" price="19">
        <name>Chang</name>
      </product>
    ... 
    ```

## 使用 LINQ to XML 读取 XML

您可能希望使用 LINQ to XML 轻松查询或处理 XML 文件：

1.  在`LinqWithEFCore`项目中，添加一个名为`settings.xml`的文件。

1.  修改其内容，如下面的标记所示：

    ```cs
    <?xml version="1.0" encoding="utf-8" ?>
    <appSettings>
      <add key="color" value="red" />
      <add key="size" value="large" />
      <add key="price" value="23.99" />
    </appSettings> 
    ```

    如果您使用的是 Windows 上的 Visual Studio 2022，那么编译后的应用程序将在`LinqWithEFCore\bin\Debug\net6.0`文件夹中执行，因此除非我们指示它始终复制到输出目录，否则它将找不到`settings.xml`文件。

1.  在**解决方案资源管理器**中，右键单击`settings.xml`文件并选择**属性**。

1.  在**属性**中，将**复制到输出目录**设置为**始终复制**。

1.  在`Program.cs`底部，创建一个方法来完成这些任务，如下面的代码所示：

    +   加载 XML 文件。

    +   使用 LINQ to XML 搜索名为`appSettings`的元素及其名为`add`的后代。

    +   将 XML 投影成具有`Key`和`Value`属性的匿名类型数组。

    +   遍历数组以显示结果：

    ```cs
    static void ProcessSettings()
    {
      XDocument doc = XDocument.Load("settings.xml");
      var appSettings = doc.Descendants("appSettings")
        .Descendants("add")
        .Select(node => new
        {
          Key = node.Attribute("key")?.Value,
          Value = node.Attribute("value")?.Value
        }).ToArray();
      foreach (var item in appSettings)
      {
        WriteLine($"{item.Key}: {item.Value}");
      }
    } 
    ```

1.  在`Program.cs`中，注释掉之前的方法调用，并调用`ProcessSettings`。

1.  运行代码并查看结果，如下所示：

    ```cs
    color: red 
    size: large 
    price: 23.99 
    ```

# 实践与探索

通过回答一些问题，进行一些实践练习，并深入研究本章涵盖的主题，来测试你的知识和理解。

## 练习 11.1 – 测试你的知识

回答以下问题：

1.  LINQ 的两个必要组成部分是什么？

1.  要返回一个类型的部分属性子集，你会使用哪个 LINQ 扩展方法？

1.  要过滤序列，你会使用哪个 LINQ 扩展方法？

1.  列出五个执行聚合操作的 LINQ 扩展方法。

1.  扩展方法`Select`和`SelectMany`之间有何区别？

1.  `IEnumerable<T>`与`IQueryable<T>`的区别是什么？以及如何在这两者之间切换？

1.  泛型`Func`委托（如`Func<T1, T2, T>`）中最后一个类型参数`T`代表什么？

1.  以`OrDefault`结尾的 LINQ 扩展方法有何好处？

1.  为什么查询理解语法是可选的？

1.  如何创建自己的 LINQ 扩展方法？

## 练习 11.2 – 实践 LINQ 查询

在`Chapter11`解决方案/工作区中，创建一个名为`Exercise02`的控制台应用程序，提示用户输入城市，然后列出该城市中 Northwind 客户的公司名称，如下所示：

```cs
Enter the name of a city: London 
There are 6 customers in London: 
Around the Horn
B's Beverages 
Consolidated Holdings 
Eastern Connection 
North/South
Seven Seas Imports 
```

然后，通过显示所有客户已居住的独特城市列表作为用户输入首选城市前的提示，来增强应用程序，如下所示：

```cs
Aachen, Albuquerque, Anchorage, Århus, Barcelona, Barquisimeto, Bergamo, Berlin, Bern, Boise, Bräcke, Brandenburg, Bruxelles, Buenos Aires, Butte, Campinas, Caracas, Charleroi, Cork, Cowes, Cunewalde, Elgin, Eugene, Frankfurt a.M., Genève, Graz, Helsinki, I. de Margarita, Kirkland, Kobenhavn, Köln, Lander, Leipzig, Lille, Lisboa, London, Luleå, Lyon, Madrid, Mannheim, Marseille, México D.F., Montréal, München, Münster, Nantes, Oulu, Paris, Portland, Reggio Emilia, Reims, Resende, Rio de Janeiro, Salzburg, San Cristóbal, San Francisco, Sao Paulo, Seattle, Sevilla, Stavern, Strasbourg, Stuttgart, Torino, Toulouse, Tsawassen, Vancouver, Versailles, Walla Walla, Warszawa 
```

## 练习 11.3 – 探索主题

使用以下页面上的链接，深入了解本章涉及的主题：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-11---querying-and-manipulating-data-using-linq`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-11---querying-and-manipulating-data-using-linq)

# 总结

本章中，你学习了如何编写 LINQ 查询来选择、投影、过滤、排序、连接和分组多种不同格式的数据，包括 XML，这些都是你每天要执行的任务。

下一章中，你将使用`Task`类型来提升应用程序的性能。
