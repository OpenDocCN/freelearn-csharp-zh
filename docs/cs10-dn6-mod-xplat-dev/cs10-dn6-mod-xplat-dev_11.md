# 第十一章：11

# 使用 LINQ 查询和操作数据

本章介绍了**语言集成查询**（**LINQ**）表达式。LINQ 是一组语言扩展，它增加了处理项目序列的能力，然后将它们过滤、排序和投影到不同的输出中。

本章将涵盖以下主题：

+   编写 LINQ 表达式

+   使用 LINQ 处理集合

+   使用 LINQ 与 EF Core

+   使用语法糖使 LINQ 语法更加美观

+   使用并行 LINQ 的多个线程

+   创建自己的 LINQ 扩展方法

+   使用 LINQ 处理 XML

# 编写 LINQ 表达式

尽管我们在*第十章*，*使用 Entity Framework Core 处理数据*中写了一些 LINQ 表达式，但它们不是重点，所以我没有正确解释 LINQ 的工作原理，所以现在让我们花时间来正确理解它们。

## 什么是 LINQ？

LINQ 有几个部分；一些是必需的，一些是可选的：

+   **扩展方法（必需）**：这些包括`Where`，`OrderBy`和`Select`等示例。这些是提供 LINQ 功能的内容。

+   **LINQ 提供程序（必需）**：这些包括用于处理内存中对象的 LINQ to Objects，用于处理存储在外部数据库中并使用 EF Core 建模的数据的 LINQ to Entities，以及用于处理存储为 XML 的数据的 LINQ to XML。这些提供程序以特定于不同类型数据的方式执行 LINQ 表达式。

+   **Lambda 表达式（可选）**：这可以用来简化 LINQ 查询，例如用于`Where`方法的条件逻辑的命名方法。

+   **LINQ 查询理解语法（可选）**：这些包括 C#关键字，如`from`，`in`，`where`，`orderby`，`descending`和`select`。这些是一些 LINQ 扩展方法的别名，它们的使用可以简化您编写的查询，特别是如果您已经有其他查询语言（如**结构化查询语言**（**SQL**））的经验。

当程序员首次接触 LINQ 时，他们经常认为 LINQ 查询理解语法就是 LINQ，但具有讽刺意味的是，那是 LINQ 的可选部分之一！

## 使用 Enumerable 类构建 LINQ 表达式

诸如`Where`和`Select`之类的 LINQ 扩展方法是由`Enumerable`静态类附加到任何实现`IEnumerable<T>`的类型（称为**序列**）上的。

例如，任何类型的数组都实现了`IEnumerable<T>`类，其中`T`是数组中的项目类型。这意味着所有数组都支持 LINQ 来查询和操作它们。

所有泛型集合，如`List<T>`，`Dictionary<TKey, TValue>`，`Stack<T>`和`Queue<T>`，都实现了`IEnumerable<T>`，因此它们也可以使用 LINQ 进行查询和操作。

`Enumerable`定义了 50 多个扩展方法，如下表所总结的：

| 方法 | 描述 |
| --- | --- |
| `First`，`FirstOrDefault`，`Last`，`LastOrDefault` | 获取序列中的第一个或最后一个项目，或抛出异常，或者如果没有第一个或最后一个项目，则返回类型的默认值，例如`int`的`0`和引用类型的`null`。 |
| `Where` | 返回与指定过滤器匹配的项目序列。 |
| `Single`，`SingleOrDefault` | 返回与特定过滤器匹配的项目，或抛出异常，或者如果没有完全匹配，则返回类型的默认值。 |
| `ElementAt`，`ElementAtOrDefault` | 返回指定索引位置的项目或抛出异常，或者如果该位置没有项目，则返回类型的默认值。在.NET 6 中，新增了可以传递`Index`而不是`int`的重载，这在使用`Span<T>`序列时更有效。 |
| `Select`，`SelectMany` | 将项目投影到不同的形状，即不同的类型，并展平嵌套的项目层次结构。 |
| `OrderBy`，`OrderByDescending`，`ThenBy`，`ThenByDescending` | 按指定字段或属性对项目进行排序。 |
| `Reverse` | 反转项目的顺序。 |
| `GroupBy` , `GroupJoin` , `Join` | 对两个序列进行分组和/或连接。 |
| `Skip` , `SkipWhile` | 跳过一定数量的项目；或者在表达式为`true`时跳过。 |
| `Take` , `TakeWhile` | 取一定数量的项目；或者在表达式为`true`时取。在.NET 6 中，新增了一个可以传递`Range`的`Take`重载，例如，`Take(range: 3..⁵)`表示从开头开始取 3 个项目，从末尾结束取 5 个项目，或者可以使用`Take(4..)`代替`Skip(4)`。 |
| `Aggregate` , `Average` , `Count` , `LongCount` , `Max` , `Min` , `Sum` | 计算聚合值。 |
| `TryGetNonEnumeratedCount` | `Count()`检查序列上是否实现了`Count`属性并返回其值，或者枚举整个序列以计算其项目数。在.NET 6 中，新增了这个方法，它只检查`Count`，如果缺少它，则返回`false`并将`out`参数设置为`0`，以避免潜在的性能低下的操作。 |
| `All` , `Any` , `Contains` | 如果所有项目或任何项目与过滤器匹配，或者序列包含指定项目，则返回`true`。 |
| `Cast` | 将项目转换为指定类型。在编译器会报错的情况下，将非泛型对象转换为泛型类型非常有用。 |
| `OfType` | 删除不匹配指定类型的项目。 |
| `Distinct` | 删除重复的项目。 |
| `Except` , `Intersect` , `Union` | 执行返回集合的操作。集合不能有重复的项目。虽然输入可以是任何序列，因此输入可以有重复项，但结果始终是一个集合。 |
| `Chunk` | 将序列分成指定大小的批次。 |
| `Append` , `Concat` , `Prepend` | 执行序列组合操作。 |
| `Zip` | 根据项目的位置执行两个序列的匹配操作，例如，第一个序列中位置 1 的项目与第二个序列中位置 1 的项目匹配。在.NET 6 中，新增了对三个序列进行匹配操作。以前，您需要两次运行两个序列的重载才能实现相同的目标。 |
| `ToArray` , `ToList` , `ToDictionary` , `ToHashSet` , `ToLookup` | 将序列转换为数组或集合。这些是唯一执行 LINQ 表达式的扩展方法。 |
| `DistinctBy` , `ExceptBy` , `IntersectBy` , `UnionBy` , `MinBy` , `MaxBy` | 在.NET 6 中新增了`By`扩展方法。它们允许在子集上执行比较，而不是在整个项目上执行比较。例如，不是通过比较整个`Person`对象来删除重复项，而是通过比较他们的`LastName`和`DateOfBirth`来删除重复项。 |

`Enumerable`类还有一些不是扩展方法的方法，如下表所示：

| 方法 | 描述 |
| --- | --- |
| `Empty<T>` | 返回指定类型`T`的空序列。将空序列传递给需要`IEnumerable<T>`的方法非常有用。 |
| `Range` | 从`start`值开始返回`count`个整数的序列。例如，`Enumerable.Range(start: 5, count: 3)`将包含整数 5、6 和 7。 |
| `Repeat` | 返回包含相同`element`重复`count`次的序列。例如，`Enumerable.Repeat(element: "5", count: 3)`将包含字符串值"5"，"5"和"5"。 |

### 理解延迟执行

LINQ 使用**延迟执行**。重要的是要理解，调用大多数这些扩展方法并不执行查询并获取结果。大多数这些扩展方法返回一个代表*问题*而不是*答案*的 LINQ 表达式。让我们来探索一下：

1.  使用您喜欢的代码编辑器创建一个名为`Chapter11`的新解决方案/工作空间。

1.  添加一个控制台应用程序项目，如下列表所示：

1.  项目模板：**控制台应用程序** / `console`

1.  工作空间/解决方案文件和文件夹：`Chapter11`

1.  项目文件和文件夹：`LinqWithObjects`

1.  在`Program.cs`中，删除现有代码并静态导入`Console`。

1.  添加语句以定义办公室工作人员的`string`值序列，如下面的代码所示：

```cs

//字符串数组是实现 IEnumerable<string>的序列

字符串

[] names = new

[] {“Michael”

，“Pam”

，“吉姆”

，“Dwight”

，

“Angela”

，“Kevin”

，“Toby”

，“Creed”

};

WriteLine（“延迟执行”

);

//问题：哪些名称以 M 结尾？

//（使用 LINQ 扩展方法编写）

变量

query1 = names.Where（name => name.EndsWith（“m”

）;

//问题：哪些名称以 M 结尾？

//（使用 LINQ 查询理解语法编写）

变量

query2 = from

名字在

名字在

name.EndsWith（“m”

）选择

名字;

```

1.  要提出问题并获得答案，即执行查询，必须通过调用`ToArray`或`ToLookup`之类的“To”方法之一来**实现**它，或者通过枚举查询，如下面的代码所示：

```cs

//答案作为包含 Pam 和 Jim 的字符串数组返回

字符串

[] result1 = query1.ToArray();

//答案作为包含 Pam 和 Jim 的字符串列表返回

List<string

> result2 = query2.ToList();

//枚举结果时返回的答案

对于

（字符串

名字在

query1）

{

WriteLine(name); //输出 Pam

names [2

] =“Jimmy”

; //将 Jim 更改为 Jimmy

//在第二次迭代时，吉米不以 M 结尾

}

```

1.  运行控制台应用程序并注意结果，如下面的输出所示：

```cs

延迟执行

Pam

```

由于延迟执行，在输出第一个结果`Pam`后，如果原始数组值发生更改，那么当我们再次循环时，就没有更多匹配项，因为“吉姆”已经变成了“吉米”，并且不以`M`结尾，因此只输出`Pam`。

在我们深入研究之前，让我们放慢脚步，逐一查看一些常见的 LINQ 扩展方法以及如何使用它们。

## 使用 Where 过滤实体

使用 LINQ 的最常见原因是使用`Where`扩展方法过滤序列中的项目。让我们通过定义一系列名称然后应用 LINQ 操作来探索过滤：

1.  在项目文件中，取消注释启用隐式使用的元素，如下标记中突出显示的那样：

```cs

<Project Sdk=“Microsoft.NET.Sdk”

<PropertyGroup>

<OutputType> Exe </ OutputType>

目标框架> net6.0

</ TargetFramework>

<Nullable> enable </ Nullable>

**<!--<ImplicitUsings> enable </ ImplicitUsings>-->**

</ PropertyGroup>

</Project>

```

1.  在`Program.cs`中，尝试调用名称数组上的`Where`扩展方法，如下面的代码所示：

```cs

WriteLine（“编写查询”

）;

变量

查询= names.W

```

1.  尝试键入`Where`方法时，请注意它在字符串数组的成员的 IntelliSense 列表中丢失，如*图 11.1*所示：！[](img/Image00095.jpg)

图 11.1：缺少 Where 扩展方法的智能感知

这是因为`Where`是一个扩展方法。它不存在于数组类型上。要使`Where`扩展方法可用，我们必须导入`System.Linq`命名空间。这在新的.NET 6 项目中默认情况下是隐式导入的，但我们已禁用它。

1.  在项目文件中，取消注释启用隐式使用的元素。

1.  重新输入`Where`方法，并注意 IntelliSense 列表现在包括`Enumerable`类添加的扩展方法，如*图 11.2*所示：！[](img/Image00096.jpg)

图 11.2：IntelliSense 显示 LINQ Enumerable 扩展方法现在

1.  当您为`Where`方法键入括号时，IntelliSense 告诉我们要调用`Where`，我们必须传递`Func<string，bool>`委托的实例。

1.  输入表达式以创建`Func<string，bool>`委托的新实例，并且现在请注意我们尚未提供方法名称，因为我们将在下一步中定义它，如下面的代码所示：

```cs

变量

查询= names.Where(new

Func<string

，布尔

>（））

```

`Func<string，bool>`委托告诉我们，对于传递给方法的每个`string`变量，方法必须返回一个`bool`值。如果方法返回`true`，则表示我们应该在结果中包含`string`，如果方法返回`false`，则表示我们应该将其排除。

## 定位命名方法

让我们定义一个只包含超过四个字符的名称的方法：

1.  在`Program.cs`底部，定义一个方法，该方法将仅包含超过四个字符的名称，如下面的代码所示：

```cs

静态

布尔

NameLongerThanFour

（

字符串

名称

）

{

返回

名称。长度> 4

；

}

```

1.  在`NameLongerThanFour`方法之上，将方法的名称传递给`Func<string，bool>`委托，然后循环遍历查询项，如下面的代码中所突出显示的那样：

```cs

变量

查询=名称。其中（

新的

Func<string

，布尔

>（

**NameLongerThanFour**

））;

**foreach**

**（**

**字符串**

**项目**

**在**

**查询）**

**{**

**WriteLine（item）;**

**}**

```

1.  运行代码并查看结果，注意只有超过四个字母的名称才会列出，如下面的输出所示：

```cs

编写查询

迈克尔

德怀特

安吉拉

凯文

克里德

```

## 通过删除显式委托实例化来简化代码

我们可以通过删除`Func<string，bool>`委托的显式实例化来简化代码，因为 C#编译器可以为我们实例化委托：

1.  为了帮助您通过逐步改进的代码来学习，复制并粘贴查询

1.  注释掉第一个示例，如下面的代码所示：

```cs

// var query = names.Where(

//   new Func<string，bool>（NameLongerThanFour））;

```

1.  修改副本以删除委托的显式实例化，如下面的代码所示：

```cs

变量

查询=名称。其中（NameLongerThanFour）;

```

1.  运行代码并注意它具有相同的行为。

## 定位 lambda 表达式

我们可以进一步简化我们的代码，使用**lambda 表达式**代替命名方法。

虽然一开始看起来可能很复杂，但 lambda 表达式只是一个*无名称的函数*。它使用`=>`（读作“转到”）符号来指示返回值：

1.  复制并粘贴查询，注释第二个示例，并修改查询，如下面的代码所示：

```cs

变量

查询=名称。其中（名称=>名称。长度> 4

）；

```

请注意，lambda 表达式的语法包括`NameLongerThanFour`方法的所有重要部分，但没有其他内容。 Lambda 表达式只需要定义以下内容：

+   输入参数的名称：`name`

+   返回值表达式：`name.Length> 4`

`name`输入参数的类型是从序列包含`string`值以及返回类型必须是`bool`值（由`Where`的委托定义）推断出的，因此`=>`符号后面的表达式必须返回`bool`值。

编译器为我们做了大部分工作，因此我们的代码可以尽可能简洁。

1.  运行代码并注意它具有相同的行为。

## 排序实体

其他常用的扩展方法是`OrderBy`和`ThenBy`，用于对序列进行排序。

如果前一个方法返回另一个序列，即实现`IEnumerable<T>`接口的类型，则可以链接扩展方法。

### 使用 OrderBy 按单个属性排序

让我们继续使用当前项目来探索排序：

1.  在现有查询的末尾附加一个对`OrderBy`的调用，如下面的代码所示：

```cs

变量

查询=名称

。其中（名称=>名称。长度> 4

）

.OrderBy（name=>name.Length）;

```

**良好的做法**：格式化 LINQ 语句，使每个扩展方法调用发生在自己的一行上，以便更容易阅读它们。

1.  运行代码并注意名称现在按最短的顺序排列，如下面的输出所示：

```cs

凯文

克里德

德怀特

安吉拉

迈克尔

```

要将最长的名称放在第一位，您将使用`OrderByDescending`。

### 使用 ThenBy 按后续属性排序

我们可能希望按多个属性进行排序，例如按字母顺序对相同长度的名称进行排序：

1.  在现有查询的末尾添加对`ThenBy`方法的调用，如下面的代码中所突出显示的那样：

```cs

var

查询=名称

.Where(name => name.Length > 4

)

.OrderBy(name => name.Length)

**.ThenBy(name => name);**

```

1.  运行代码并注意以下排序顺序的轻微差异。在相同长度的名称组中，名称按照`string`的完整值按字母顺序排序，因此`Creed`在`Kevin`之前，`Angela`在`Dwight`之前，如下面的输出所示：

```cs

Creed

凯文

安吉拉

Dwight

迈克尔

```

## 使用 var 或指定类型声明查询

在编写 LINQ 表达式时，使用`var`声明查询对象很方便。这是因为随着您在 LINQ 表达式上的工作，类型经常会发生变化。例如，我们的查询起初是`IEnumerable<string>`，目前是`IOrderedEnumerable<string>`：

1.  将鼠标悬停在`var`关键字上，并注意其类型为`IOrderedEnumerable<string>`

1.  用实际类型替换`var`，如下面的代码中所突出显示的那样：

```cs

**IOrderedEnumerable<**

**string**

**>**

查询=名称

.Where(name => name.Length > 4

)

.OrderBy(name => name.Length)

.ThenBy(name => name);

```

**良好的实践**：一旦完成对查询的工作，您可以将声明的类型从`var`更改为实际类型，以清楚地表明类型是什么。这很容易，因为您的代码编辑器可以告诉您它是什么。

## 按类型过滤

`Where`扩展方法非常适合按值过滤，例如文本和数字。但是，如果序列包含多种类型，并且您想按特定类型过滤并尊重任何继承层次结构，该怎么办？

想象一下，您有一系列异常。有数百种异常类型构成一个复杂的层次结构，部分显示在*图 11.3*中：

![自动生成的图表描述](img/Image00097.jpg)

图 11.3：部分异常继承层次结构

让我们探索按类型过滤：

1.  在`Program.cs`中，定义一个异常派生对象的列表，如下面的代码所示：

```cs

WriteLine("按类型过滤"

);

List<Exception> exceptions = new

()

{

新的

ArgumentException(),

新的

SystemException(),

新的

IndexOutOfRangeException(),

新的

InvalidOperationException(),

新的

NullReferenceException(),

新的

InvalidCastException(),

新的

OverflowException(),

新的

DivideByZeroException(),

新的

ApplicationException()

};

```

1.  使用`OfType<T>`扩展方法编写语句，以删除不是算术异常的异常，并将仅算术异常写入控制台，如下面的代码所示：

```cs

IEnumerable<ArithmeticException> arithmeticExceptionsQuery =

exceptions.OfType<ArithmeticException>();

foreach

(ArithmeticException exception in

arithmeticExceptionsQuery)

{

WriteLine(exception);

}

```

1.  运行代码并注意结果仅包括`ArithmeticException`类型的异常，或`ArithmeticException` -派生类型，如下面的输出所示：

```cs

System.OverflowException:算术运算导致溢出。

System.DivideByZeroException:尝试除以零。

```

## 使用 LINQ 处理集合和袋

集合是数学中最基本的概念之一。**集合**是一个或多个唯一对象的集合。**多重集**，又称**袋**，是一个或多个可以重复的对象的集合。

您可能还记得在学校学习文氏图。常见的集合操作包括集合之间的**交集**或**并集**。

让我们创建一个控制台应用程序，为学徒的三个`string`值数组定义一些常见的集合和多重集合操作：

1.  使用您喜欢的代码编辑器将一个名为`LinqWithSets`的新控制台应用程序添加到`Chapter11`解决方案/工作区中：

1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择。

1.  在 Visual Studio Code 中，将`LinqWithSets`选择为活动的 OmniSharp 项目。

1.  在`Program.cs`中，删除现有的代码，并静态导入`Console`类型，如下面的代码所示：

```cs

using

static

System.Console;

```

1.  在`Program.cs`的底部，添加以下方法，将任何`string`变量序列作为逗号分隔的单个`string`输出到控制台输出，以及一个可选的描述，如下面的代码所示：

```cs

static

void

输出

(

IEnumerable<

string

> cohort,

string

description =

""

)

{

if

(!string

.IsNullOrEmpty(description))

{

WriteLine(description);

}

Write(" "

);

WriteLine(string

.Join(", "

, cohort.ToArray()));

WriteLine();

}

```

1.  在`Output`方法上方，添加语句来定义三个名称数组，输出它们，然后对它们执行各种集合操作，如下面的代码所示：

```cs

string

[] cohort1 = new

[]

{ "Rachel"

, "Gareth"

, "Jonathan"

, "George"

};

string

[] cohort2 = new

[]

{ "Jack"

, "Stephen"

, "Daniel"

, "Jack"

, "Jared"

};

string

[] cohort3 = new

[]

{ "Declan"

, "Jack"

, "Jack"

, "Jasmine"

, "Conor"

};

Output(cohort1, "Cohort 1"

);

Output(cohort2, "Cohort 2"

);

Output(cohort3, "Cohort 3"

);

Output(cohort2.Distinct(), "cohort2.Distinct()"

);

Output(cohort2.DistinctBy(name => name.Substring(0

, 2

)),

"cohort2.DistinctBy(name => name.Substring(0, 2)):"

);

Output(cohort2.Union(cohort3), "cohort2.Union(cohort3)"

);

Output(cohort2.Concat(cohort3), "cohort2.Concat(cohort3)"

);

Output(cohort2.Intersect(cohort3), "cohort2.Intersect(cohort3)"

);

Output(cohort2.Except(cohort3), "cohort2.Except(cohort3)"

);

Output(cohort1.Zip(cohort2,(c1, c2) => $"

{c1}

matched with

{c2}

"

),

"cohort1.Zip(cohort2)"

);

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

Jack，Stephen，Daniel

cohort2.Union(cohort3)

Jack, Stephen, Daniel, Jared, Declan, Jasmine, Conor

cohort2.Concat(cohort3)

Jack, Stephen, Daniel, Jack, Jared, Declan, Jack, Jack, Jasmine, Conor

cohort2.Intersect(cohort3)

Jack

cohort2.Except(cohort3)

Stephen，Daniel，Jared

cohort1.Zip(cohort2)

Rachel 与 Jack 匹配，Gareth 与 Stephen 匹配，Jonathan 与 Daniel 匹配，George 与 Jack 匹配

```

使用`Zip`，如果两个序列中的项目数量不相等，则一些项目将没有匹配的伙伴。像`Jared`这样没有伙伴的人将不会包含在结果中。

对于`DistinctBy`的示例，我们不是通过比较整个名称来删除重复项，而是定义一个 lambda 键选择器来删除重复项，通过比较前两个字符，因此`Jared`被删除，因为`Jack`已经是以`Ja`开头的名称。

到目前为止，我们已经使用了 LINQ to Objects 提供程序来处理内存中的对象。接下来，我们将使用 LINQ to Entities 提供程序来处理存储在数据库中的实体。

# 使用 EF Core 的 LINQ

我们已经使用了 LINQ 查询来过滤和排序，但没有改变序列中项目的形状。这被称为**投影**，因为它是将一个形状的项目投影到另一个形状的项目。要了解投影，最好使用一些更复杂的类型来工作，因此在下一个项目中，我们将使用来自 Northwind 示例数据库的实体序列，而不是使用`string`序列。

我将提供使用 SQLite 的说明，因为它是跨平台的，但如果您更喜欢使用 SQL Server，那么请随意这样做。我已经包含了一些注释代码，以便在您选择时启用 SQL Server。

## 构建 EF Core 模型

我们必须定义一个 EF Core 模型来表示我们将使用的数据库和表。我们将手动定义模型，以完全控制并防止`Categories`和`Products`表之间自动定义关系。稍后，您将使用 LINQ 来连接这两个实体集：

1.  使用您喜欢的代码编辑器将新的控制台应用程序命名为`LinqWithEFCore`，并将其添加到`Chapter11`解决方案/工作区中。

1.  在 Visual Studio Code 中，将`LinqWithEFCore`选择为活动的 OmniSharp 项目。

1.  在`LinqWithEFCore`项目中，添加对 SQLite 和/或 SQL Server 的 EF Core 提供程序的包引用，如下标记所示：

```cs

<ItemGroup>

<PackageReference

Include="Microsoft.EntityFrameworkCore.Sqlite"

Version="6.0.0"

/>

<PackageReference

Include="Microsoft.EntityFrameworkCore.SqlServer"

Version="6.0.0"

/>

</ItemGroup>

```

1.  构建项目以恢复包。

1.  将`Northwind4Sqlite.sql`文件复制到`LinqWithEFCore`文件夹中。

1.  在命令提示符或终端上，通过执行以下命令创建 Northwind 数据库：

```cs

sqlite3 Northwind.db -init Northwind4Sqlite.sql

```

1.  请耐心等待，因为此命令可能需要一段时间来创建数据库结构。最终，您将看到 SQLite 命令提示符，如下输出所示：

```cs

--从 Northwind.sql 加载资源

SQLite 版本 3.36.0 2021-08-02 15:20:15

输入“.help”以获取使用提示。

sqlite>

```

1.  在 macOS 上按 cmd + D 或在 Windows 上按 Ctrl + C 退出 SQLite 命令模式。

1.  向项目添加三个名为`Northwind.cs`，`Category.cs`和`Product.cs`的类文件。

1.  修改名为`Northwind.cs`的类文件，如下所示：

```cs

使用

Microsoft.EntityFrameworkCore; // DbContext，DbSet<T>

命名空间

Packt.Shared

;

//这管理着与数据库的连接

公共

类

Northwind

: DbContext

{

//这些属性映射到数据库中的表

public

DbSet<Category>? 类别 { 获取

; 设置

; }

public

DbSet<Product>? 产品 { 获取

; 设置

; }

受保护的

覆盖

空

OnConfiguring

（

DbContextOptionsBuilder optionsBuilder

）

{

字符串

路径=Path.Combine（

Environment.CurrentDirectory，"Northwind.db"

）;

optionsBuilder.UseSqlite（$“文件名=

{路径}

"

）;

/*

字符串连接=“数据源=。;”+

“Initial Catalog=Northwind;”+

“集成安全性=true;”+

"MultipleActiveResultSets=true;";

optionsBuilder.UseSqlServer(connection);

*/

}

受保护的

覆盖

空

OnModelCreating

（

ModelBuilder modelBuilder

）

{

modelBuilder.Entity<Product>()

.Property（产品=>产品.UnitPrice）

.HasConversion<double

（）;

}

}

```

1.  修改名为`Category.cs`的类文件，如下所示：

```cs

使用

System.ComponentModel.DataAnnotations;

命名空间

Packt.Shared

;

公共

类

类别

{

公共

整数

CategoryId { 获取

; 设置

; }

[必需

]

[StringLength（15）

]

public

字符串

CategoryName { 获取

; 设置

; } = null

！;

公共

字符串

? 描述 { 获取

; 设置

; }

}

```

1.  修改名为`Product.cs`的类文件，如下所示：

```cs

使用

System.ComponentModel.DataAnnotations;

使用

System.ComponentModel.DataAnnotations.Schema;

命名空间

Packt.Shared

;

公共

类

Product

{

public

整数

ProductId { 获取

; 设置

; }

[必需

]

[StringLength（40）

]

公共

字符串

ProductName { 获取

; 设置

; } = null

！;

公共

整数

? 供应商 Id { 获取

; 设置

; }

公共

整数

? 类别 Id { 获取

; 设置

; }

[StringLength（20）

]

公共

字符串

? 每单位数量 { 获取

; 设置

; }

[Column（TypeName =

"money"

）

] // SQL Server 提供程序所需

公共

decimal

? 单价 { 获取

; 设置

; }

公共

短

? 库存单位 { 获取

; 设置

; }

public

短

? 订购单位 { 获取

; 设置

; }

公共

短

? 重新订购级别 { 获取

; 设置

; }

公共

布尔

已停产{获取

; 设置

; }

}

```

1.  构建项目并修复任何编译器错误。

如果你正在使用 Windows 的 Visual Studio 2022，那么编译后的应用程序将在`LinqWithEFCore\bin\Debug\net6.0`文件夹中执行，因此它不会找到数据库文件，除非我们指示它应该始终复制到输出目录。

1.  在**解决方案资源管理器**中，右键单击`Northwind.db`文件，然后选择**属性**。

1.  在**属性**中，将**复制到输出目录**设置为**始终复制**。

## 过滤和排序序列

现在让我们编写语句来过滤和排序表中的行序列：

1.  在`Program.cs`中，使用静态导入`Console`类型和用于使用 LINQ 处理 EF Core 和实体模型的命名空间，如下面的代码所示：

```cs

使用

Packt.Shared; // Northwind, Category, Product

using

Microsoft.EntityFrameworkCore; // DbSet<T>

使用

static

System.Console;

```

1.  在`Program.cs`的底部，编写一个方法来过滤和排序产品，如下面的代码所示：

```cs

static

void

FilterAndSort

()

{

using

（Northwind db = new

（）

{

DbSet<Product> allProducts = db.Products;

IQueryable<Product> filteredProducts =

allProducts.Where(product => product.UnitPrice < 10

M);

IOrderedQueryable<Product> sortedAndFilteredProducts =

filteredProducts.OrderByDescending(product => product.UnitPrice);

WriteLine("成本低于$10 的产品："

);

foreach

(Product p in

sortedAndFilteredProducts)

{

WriteLine("{0}: {1}的成本是{2:$#,##0.00}"

,

p.ProductId, p.ProductName, p.UnitPrice);

}

调用 WriteLine();

}

}

```

`DbSet<T>`实现了`IEnumerable<T>`，因此 LINQ 可以用于查询和操作为 EF Core 构建的实体模型中的实体集合。（实际上，我应该说`TEntity`而不是`T`，但是这个泛型类型的名称没有功能效果。唯一的要求是类型是一个`class`。名称只是指示这个类预期是一个实体模型。）

你可能也注意到序列实现了`IQueryable<T>`（或在调用排序 LINQ 方法后实现了`IOrderedQueryable<T>`），而不是`IEnumerable<T>`或`IOrderedEnumerable<T>`。

这表明我们正在使用一个在内存中使用表达式树构建查询的 LINQ 提供程序。它们代表树状数据结构中的代码，并且能够创建动态查询，这对于构建外部数据提供程序（如 SQLite）的 LINQ 查询非常有用。

LINQ 表达式将被转换为另一种查询语言，比如 SQL。使用`foreach`枚举查询或调用`ToArray`等方法将强制执行查询并实现结果。

1.  在`Program.cs`中的命名空间导入之后，调用`FilterAndSort`方法。

1.  运行代码并查看结果，如下面的输出所示：

```cs

成本低于$10 的产品：

41: Jack's New England Clam Chowder 的成本是$9.65

45: Rogede sild 的成本是$9.50

47: Zaanse koeken 的成本是$9.50

19: Teatime Chocolate Biscuits 的成本是$9.20

23: Tunnbröd 的成本是$9.00

75: Rhönbräu Klosterbier 的成本是$7.75

54: Tourtière 的成本是$7.45

52: Filo Mix 的成本是$7.00

13: Konbu 的成本是$6.00

24: Guaraná Fantástica 的成本是$4.50

33: Geitost 的成本是$2.50

```

尽管这个查询输出了我们想要的信息，但它效率低下，因为它从`Products`表中获取了所有列，而不仅仅是我们需要的三列，这相当于以下 SQL 语句：

```cs

SELECT * FROM Products;

```

在*第十章*，*使用 Entity Framework Core 处理数据*中，您学会了如何记录针对 SQLite 执行的 SQL 命令，以便您自己查看。

## 将序列投影到新类型中

在我们查看投影之前，我们需要复习对象初始化语法。如果你有一个已定义的类，那么你可以使用类名、`new()`和大括号来实例化一个对象，为字段和属性设置初始值，如下面的代码所示：

```cs

public class

人

{

public

string

Name { get

; set

; }

public

DateTime DateOfBirth { get

; set

; }

}

已知类型对象 = new

()

{

Name = "Boris Johnson"

,

DateOfBirth = new

(year: 1964

, month: 6

, day: 19

)

};

```

C# 3.0 及更高版本允许使用`var`关键字实例化**匿名类型**，如下面的代码所示：

```cs

var

anonymouslyTypedObject = new

{

Name = "Boris Johnson"

,

DateOfBirth = new

DateTime(year: 1964

, month: 6

, day: 19

)

};

```

尽管我们没有指定类型，但编译器可以从两个名为`Name`和`DateOfBirth`的属性的设置中推断出匿名类型。编译器可以从分配的值推断出两个属性的类型：一个文字`string`和一个新的日期/时间值实例。

当编写 LINQ 查询以将现有类型投影到新类型时，此功能特别有用，而无需明确定义新类型。由于类型是匿名的，因此只能使用`var`声明的本地变量。

通过在`Select`方法中添加调用来使针对数据库表执行的 SQL 命令更有效，以将`Product`类的实例投影为仅具有三个属性的新匿名类型的实例：

1.  在`FilterAndSort`中，添加一个语句来扩展 LINQ 查询，使用`Select`方法仅返回我们需要的三个属性（即表列），并修改`foreach`语句以使用`var`关键字和投影 LINQ 表达式，如下面的代码中突出显示的那样：

```cs

IOrderedQueryable<Product> sortedAndFilteredProducts =

filteredProducts.OrderByDescending(product => product.UnitPrice);

**var**

**projectedProducts = sortedAndFilteredProducts**

**.Select(product =>**

**new**

**// 匿名类型**

**{**

**product.ProductId,**

**product.ProductName,**

**product.UnitPrice**

**});**

WriteLine("Products that cost less than $10:"

);

foreach

(

**var**

**p**

**in**

**projectedProducts**

)

{

```

1.  将鼠标悬停在`Select`方法调用中的`new`关键字和`foreach`语句中的`var`关键字上，并注意它是一个匿名类型，如*图 11.4*所示：![](img/Image00098.jpg)

图 11.4：在 LINQ 投影期间使用的匿名类型

1.  运行代码并确认输出与以前相同。

## 连接和分组序列

有两个用于连接和分组的扩展方法：

+   **Join**：此方法有四个参数：要与之连接的序列，左侧序列上要匹配的属性，右侧序列上要匹配的属性，以及一个投影。

+   **GroupJoin**：此方法具有相同的参数，但它将匹配项组合成一个具有匹配值的`Key`属性和一个`IEnumerable<T>`类型的多个匹配项的组对象。

### 连接序列

让我们在处理两个表`Categories`和`Products`时探索这些方法：

1.  在`Program.cs`的底部，创建一个方法来选择类别和产品，将它们连接起来，并输出它们，如下面的代码所示：

```cs

static

void

JoinCategoriesAndProducts

()

{

using

(Northwind db = new

())

{

// 将每个产品与其类别连接以返回 77 个匹配项

var

queryJoin = db.Categories.Join(

inner: db.Products,

outerKeySelector: category => category.CategoryId,

innerKeySelector: product => product.CategoryId,

resultSelector: (c, p) =>

new

{ c.CategoryName, p.ProductName, p.ProductId });

foreach

(var

item in

queryJoin)

{

WriteLine("{0}: {1} is in {2}."

,

arg0: item.ProductId,

arg1: item.ProductName,

arg2: item.CategoryName);

}

}

}

```

在连接中，有两个序列，*outer*和*inner*。在前面的示例中，`categories`是外部序列，`products`是内部序列。

1.  在`Program.cs`的顶部，注释掉对`FilterAndSort`的调用，并调用`JoinCategoriesAndProducts`。

1.  运行代码并查看结果。请注意，每个 77 个产品都有一行输出，如下面的输出所示（编辑为仅包括前 10 个项目）：

```cs

1: Chai is in Beverages.

2: Chang is in Beverages.

3: Aniseed Syrup is in Condiments.

4：厨师安东的卡真调味料在调味品中。

5：厨师安东的浓汤混合物在调味品中。

6：Grandma's Boysenberry Spread 在调味品中。

7：Uncle Bob's 有机干梨在生产中。

8：北森林蔓越莓酱在调味品中。

9：Mishi Kobe Niku 在肉类/家禽中。

10：Ikura 在海鲜中。

...

```

1.  在现有查询的末尾，调用`OrderBy`方法按`CategoryName`排序，如下面的代码所示：

```cs

.OrderBy(cp => cp.CategoryName);

```

1.  运行代码并查看结果。注意每个 77 个产品都有一行输出，并且结果首先显示所有`饮料`类别中的产品，然后是`调味品`类别，依此类推，如下面的部分输出所示：

```cs

1：柴在饮料中。

2：Chang 在饮料中。

24：Guaraná Fantástica 在饮料中。

34：Sasquatch Ale 在饮料中。

35：Steeleye Stout 在饮料中。

38：Côte de Blaye 在饮料中。

39：Chartreuse verte 在饮料中。

43：怡保咖啡在饮料中。

67：Laughing Lumberjack Lager 在饮料中。

70：澳洲啤酒在饮料中。

75：Rhönbräu Klosterbier 在饮料中。

76：Lakkalikööri 在饮料中。

3：茴香糖浆在调味品中。

4：厨师安东的卡真调味料在调味品中。

...

```

### 组合连接序列

1.  在`Program.cs`的底部，创建一个方法来分组和连接，显示组名，然后显示每个组内的所有项目，如下面的代码所示：

```cs

静态

空白

GroupJoinCategoriesAndProducts

（）

{

使用

（Northwind db = new

（）

{

//按类别分组所有产品以返回 8 个匹配项

var

queryGroup = db.Categories.AsEnumerable().GroupJoin（```

内部：db.Products，

outerKeySelector：category => category.CategoryId，

innerKeySelector：product => product.CategoryId，

resultSelector：（c，matchingProducts）=> new

{

c.CategoryName，

产品=matchingProducts.OrderBy(p => p.ProductName)

});

对于每一个

（var

类别在

queryGroup）

{

WriteLine("{0}有{1}个产品。"

，

arg0：category.CategoryName，

arg1：category.Products.Count（））;

对于每一个

（var

产品在

类别。产品）

{

WriteLine($"

{product.ProductName}

"

）;

}

}

}

}

```

如果我们没有调用`AsEnumerable`方法，那么会抛出运行时异常，如下面的输出所示：

```cs

未处理的异常。System.ArgumentException：参数类型'System.Linq.IOrderedQueryable`1[Packt.Shared.Product]'与相应的成员类型'System.Linq.IOrderedEnumerable`1[Packt.Shared.Product]'不匹配（参数'arguments[1]'）

```

这是因为并非所有的 LINQ 扩展方法都可以从表达式树转换为 SQL 等其他查询语法。在这些情况下，我们可以通过调用`AsEnumerable`方法从`IQueryable<T>`转换为`IEnumerable<T>`，这将强制查询处理仅使用 LINQ 到 EF Core 将数据带入应用程序，然后使用 LINQ 到对象在内存中执行更复杂的处理。但通常这样做效率较低。

1.  在`Program.cs`的顶部，注释掉先前的方法调用，并调用`GroupJoinCategoriesAndProducts`。

1.  运行代码，查看结果，并注意每个类别内的产品已按其名称排序，如查询中定义的那样，并如下部分输出所示：

```cs

饮料有 12 种产品。

柴

Chang

Chartreuse verte

Côte de Blaye

Guaraná Fantástica

怡保咖啡

Lakkalikööri

Laughing Lumberjack Lager

Outback Lager

Rhönbräu Klosterbier

Sasquatch Ale

Steeleye Stout

调味品有 12 种产品。

茴香糖浆

厨师安东的卡真调味料

厨师安东的浓汤混合物

...

```

## 聚合序列

有 LINQ 扩展方法来执行聚合函数，比如`Average`和`Sum`。让我们编写一些代码来看到这些方法中的一些在`Products`表中聚合信息的操作：

1.  在`Program.cs`的底部，创建一个方法来展示聚合扩展方法的使用，如下面的代码所示：

```cs

静态

空白

AggregateProducts

（）

{

使用

（Northwind db = new

（）

{

WriteLine("{0,-25} {1,10}"

，

arg0: "产品数量："

，

arg1: db.Products.Count());

WriteLine("{0,-25} {1,10:$#,##0.00}"

，

arg0: "最高产品价格："

，

arg1: db.Products.Max(p => p.UnitPrice));

WriteLine("{0,-25} {1,10:N0}"

，

arg0: "库存单位总和："

，

arg1: db.Products.Sum(p => p.UnitsInStock));

WriteLine("{0,-25} {1,10:N0}"

，

arg0: "库存订单总和："

，

arg1: db.Products.Sum(p => p.UnitsOnOrder));

WriteLine("{0,-25} {1,10:$#,##0.00}"

，

arg0: "平均单价："

，

arg1：db.Products.Average(p => p.UnitPrice));

WriteLine("{0,-25} {1,10:$#,##0.00}"

，

arg0: "库存单位的价值："

，

arg1: db.Products

.Sum(p => p.UnitPrice * p.UnitsInStock));

}

}

```

1.  在`Program.cs`的顶部，注释掉以前的方法并调用`AggregateProducts`

1.  运行代码并查看结果，如下面的输出所示：

```cs

产品数量：77

最高产品价格：$263.50

库存单位总和：3,119

库存订单总和：780

平均单价：$28.87

库存单位的价值：$74,050.85

```

# 用语法糖使 LINQ 语法更加美观

C# 3.0 在 2008 年引入了一些新的语言关键字，以使具有 SQL 经验的程序员更容易编写 LINQ 查询。这种语法糖有时被称为**LINQ 查询理解语法**。

考虑以下的`string`值数组：

```cs

字符串

[] names = new

[] { "迈克尔"

，“帕姆”

，“吉姆”

，“德怀特”

，

“安吉拉”

，“凯文”

，“托比”

，“克里德”

};

```

要过滤和排序名称，您可以使用扩展方法和 lambda 表达式，如下面的代码所示：

```cs

var

查询=名称

.Where(name => name.Length > 4

）

.OrderBy(name => name.Length)

.ThenBy(name => name);

```

或者您可以通过使用查询理解语法来实现相同的结果，如下面的代码所示：

```cs

var

查询=从

名称在

names

在

name.Length > 4

orderby

name.Length, name

选择

名称；

```

编译器会为您更改查询理解语法，以便使用等效的扩展方法和 lambda 表达式。

`select`关键字在 LINQ 查询理解语法中总是必需的。如果使用扩展方法和 lambda 表达式，则`Select`扩展方法是可选的，因为如果您不调用`Select`，则整个项目将被隐式选择。

并非所有扩展方法都有 C#关键字等效，例如`Skip`和`Take`扩展方法，它们通常用于为大量数据实现分页。

跳过和获取的查询不能仅使用查询理解语法编写，因此我们可以使用所有扩展方法编写查询，如下面的代码所示：

```cs

var

查询=名称

.Where(name => name.Length > 4

）

.Skip(80

）

.Take(10

）；

```

或者您可以将查询理解语法括在括号中，然后切换到使用扩展方法，如下面的代码所示：

```cs

var

查询=（从

名称在

names

在

name.Length > 4

选择

name)

.Skip(80

）

.Take(10

）；

```

**良好的实践**：学习使用扩展方法和 lambda 表达式以及查询理解语法编写 LINQ 查询的两种方式，因为您可能需要维护使用这两种方式的代码。

# 使用并行 LINQ 使用多个线程

默认情况下，只使用一个线程来执行 LINQ 查询。**并行 LINQ**（**PLINQ**）是启用多个线程执行 LINQ 查询的简单方法。

**良好的实践**：不要假设使用并行线程会提高应用程序的性能。始终测量实际的时间和资源使用情况。

## 创建一个从多个线程中受益的应用程序

为了看到它的作用，我们将从只使用单个线程来计算 45 个整数的斐波那契数的代码开始。我们将使用`StopWatch`类型来测量性能的变化。

我们将使用操作系统工具来监视 CPU 和 CPU 核使用情况。如果您没有多个 CPU 或至少多个核心，那么这个练习就不会显示出太多！

1.  使用您喜欢的代码编辑器向 `Chapter11` 解决方案/工作区添加一个名为 `LinqInParallel` 的新控制台应用。

1.  在 Visual Studio Code 中，选择 `LinqInParallel` 作为活动的 OmniSharp 项目。

1.  在 `Program.cs` 中，删除现有语句，然后导入 `System.Diagnostics` 命名空间，以便我们可以使用 `StopWatch` 类型，并静态导入 `System.Console` 类型。

1.  添加语句创建一个秒表来记录时间，等待按键后开始计时器，创建 45 个整数，计算每个整数的最后一个斐波那契数，停止计时器，并显示经过的毫秒数，如下面的代码所示：

```cs

秒表监视 = 新的

();

写("按 ENTER 开始。"

);

ReadLine();

watch.Start();

int

最大 = 45

;

IEnumerable<int

> 数字 = Enumerable.Range(start: 1

，计数：最大);

写行($"计算斐波那契序列直到

{最大}

。 请稍候..."

);

int

[] fibonacciNumbers = 数字

.Select(number => Fibonacci(number)).ToArray();

watch.Stop();

写行("{0:#,##0} 经过的毫秒。"

，

arg0: watch.ElapsedMilliseconds);

写("结果："

);

foreach

（int

数字在

fibonacciNumbers)

{

写($"

{数字}

"

);

}

静态

int

Fibonacci

(

int

术语

)

=>

术语开关

{

1

=> 0

，

2

=> 1

，

_ => 斐波那契（项 - 1

) + Fibonacci(term - 2

）

};

```

1.  运行代码，但不要按 Enter 开始秒表，因为我们需要确保监视工具显示处理器活动。

### 使用 Windows

1.  如果您使用的是 Windows，则右键单击 Windows **开始** 按钮或按 Ctrl + Alt + Delete，然后单击 **任务管理器**。

1.  在 **任务管理器** 窗口底部，单击 **详细信息**。

1.  在 **任务管理器** 窗口顶部，单击 **性能** 选项卡。

1.  右键单击 **CPU 利用率** 图表，选择 **更改图表为**，然后选择 **逻辑处理器**。

### 使用 macOS

1.  如果您使用的是 macOS，则启动 **活动监视器**。

1.  导航到 **查看** | **更新频率非常频繁（1 秒）**。

1.  要查看 CPU 图表，请导航到 **窗口** | **CPU 历史记录**。

### 对于所有操作系统

1.  重新排列您的监视工具和代码编辑器，使它们并排。

1.  等待 CPU 稳定下来，然后按 Enter 开始秒表并运行查询。结果应该是经过的毫秒数，如下面的输出所示：

```cs

按 ENTER 开始。

计算斐波那契序列直到 45\. 请稍候...

17,624 经过的毫秒。

结果：0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584 4181 6765 10946 17711 28657 46368 75025 121393 196418 317811 514229 832040 1346269 2178309 3524578 5702887 9227465 14930352 24157817 39088169 63245986 102334155 165580141 267914296 433494437 701408733

```

监视工具可能会显示一个或两个 CPU 最多被使用，随着时间的推移交替。其他可能同时执行后台任务，例如垃圾回收器，因此其他 CPU 或核心不会完全平坦，但工作肯定不会均匀分布在所有可能的 CPU 或核心之间。还要注意，一些逻辑处理器的使用率达到了 100%。

1.  在 `Program.cs` 中，修改查询以调用 `AsParallel` 扩展方法，并对生成的序列进行排序，因为在并行处理时，结果可能会变得无序，如下面的代码中所突出显示的那样：

```cs

int

[] fibonacciNumbers = 数字。

**AsParallel()**

.Select(number => Fibonacci(number))

**.OrderBy(number => number)**

.ToArray();

```

**良好的实践**：永远不要在查询的末尾调用 `AsParallel`。这样做没有任何作用。您必须在调用 `AsParallel` 后执行至少一个操作，以便对该操作进行并行化。.NET 6 引入了一个代码分析器，将警告此类误用。

1.  运行代码，等待 CPU 图表在您的监控工具中稳定，然后按 Enter 键启动秒表并运行查询。这次，应用程序应该在更短的时间内完成（尽管可能不像您希望的那样少-管理这些多个线程需要额外的努力！）：

```cs

按 Enter 键开始。

计算 Fibonacci 序列直到 45\. 请稍等...

9,028 毫秒已过去。

结果：0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584 4181 6765 10946 17711 28657 46368 75025 121393 196418 317811 514229 832040 1346269 2178309 3524578 5702887 9227465 14930352 24157817 39088169 63245986 102334155 165580141 267914296 433494437 701408733

```

1.  监控工具应显示所有 CPU 均被用于执行 LINQ 查询，并注意到没有一个逻辑处理器达到 100%的最大值，因为工作更加均匀。

您将在*第十二章*中了解更多关于管理多个线程的内容，*通过多任务改进性能和可伸缩性*。

# 创建您自己的 LINQ 扩展方法

在*第六章*，*实现接口和继承类*中，您学会了如何创建自己的扩展方法。要创建 LINQ 扩展方法，您所需做的就是扩展`IEnumerable<T>`类型。

**良好实践**：将您自己的扩展方法放在一个单独的类库中，以便它们可以轻松部署为它们自己的程序集或 NuGet 包。

我们将改进`Average`扩展方法作为一个例子。 一个受过良好教育的学童会告诉你*average*可以表示三种意思：

+   **Mean**：求和并除以计数。

+   **Mode**：最常见的数字。

+   **Median**：当排序时，数字中间的数字。

微软的`Average`扩展方法的实现计算*mean*。我们可能想要为`Mode`和`Median`定义我们自己的扩展方法：

1.  在`LinqWithEFCore`项目中，添加一个名为`MyLinqExtensions.cs`的新类文件。

1.  修改类，如下所示的代码：

```cs

命名空间

System.Linq

; //扩展微软的命名空间

public

静态

类

MyLinqExtensions

{

// 这是一个可链接的 LINQ 扩展方法

public

static

IEnumerable

<

T

ProcessSequence

<

T

>(

这

IEnumerable<T>序列

)

{

//你可以在这里做一些处理

返回

sequence;

}

public

静态

IQueryable

<

T

ProcessSequence

<

T

>(

这

IQueryable<T>序列

)

{

//你可以在这里做一些处理

返回

sequence;

}

//这些是标量 LINQ 扩展方法

public

静态

int

? Median(

这

IEnumerable<int

?>序列)

{

var

ordered = sequence.OrderBy(item => item);

int

middlePosition = ordered.Count() / 2

;

return

ordered.ElementAt(middlePosition);

}

public

静态

int

? Median<T>(

这

IEnumerable<T>序列，Func<T，int

?>选择器)

{

返回

sequence.Select(selector).Median();

}

public

静态

decimal

? Median(

这

IEnumerable<decimal

?>序列)

{

var

ordered = sequence.OrderBy(item => item);

int

middlePosition = ordered.Count() / 2

;

返回

ordered.ElementAt(middlePosition);

}

public

静态

decimal

? Median<T>(

这

IEnumerable<T>序列，Func<T，decimal

?>选择器)

{

返回

sequence.Select(selector).Median();

}

public

static

int

? Mode(

这

IEnumerable<int

?>序列)

{

var

grouped = sequence.GroupBy(item => item);

var

orderedGroups = grouped.OrderByDescending(

group

=> group

.Count());

返回

orderedGroups.FirstOrDefault()?.Key;

}

public

静态

int

? Mode<T>(

this

IEnumerable<T>序列，Func<T，int

?>选择器)

{

return

sequence.Select(selector)?.Mode();

}

public

静态

decimal

? Mode(

这

IEnumerable<decimal

?>序列)

{

var

grouped = sequence.GroupBy(item => item);

var

orderedGroups = grouped.OrderByDescending(

group

=> group

.Count());

返回

orderedGroups.FirstOrDefault()?.Key;

}

public

静态

decimal

? Mode<T>(

this

IEnumerable<T>序列，Func<T，decimal

?>选择器)

{

返回

sequence.Select(selector).Mode();

}

}

```

如果这个类在一个单独的类库中，要使用你的 LINQ 扩展方法，你只需要引用类库程序集，因为`System.Linq`命名空间已经隐式导入。

**警告！**除了一个以上的扩展方法不能与`IQueryable`序列一起使用，就像 LINQ 到 SQLite 或 LINQ 到 SQL Server 所使用的那样，因为我们还没有实现将我们的代码转换成底层查询语言如 SQL 的方法。

### 尝试可链式扩展方法

首先，我们将尝试将`ProcessSequence`方法与其他扩展方法链接起来：

1.  在`Program.cs`中，在`FilterAndSort`方法中，修改`Products`的 LINQ 查询，调用你的自定义可链式扩展方法，如下面代码中的高亮部分所示：

```cs

DbSet<Product>? allProducts = db.Products;

如果

（allProducts is

空

）

{

WriteLine("未找到产品。"

）;

返回

；

}

IQueryable<Product> processedProducts = allProducts.ProcessSequence();

IQueryable<Product> filteredProducts =

已处理的产品

.Where(product => product.UnitPrice < 10

M);

```

1.  在`Program.cs`中，取消注释`FilterAndSort`方法，并注释掉对其他方法的调用。

1.  运行代码并注意到你看到与之前相同的输出，因为你的方法不修改序列。但是现在你知道如何用自己的功能扩展 LINQ 表达式了。

### 尝试使用众数和中位数方法

其次，我们将尝试使用`Mode`和`Median`方法来计算其他类型的平均值：

1.  在`Program.cs`的底部，创建一个方法来输出`UnitsInStock`和`UnitPrice`的平均值、中位数和众数，使用你自定义的扩展方法和内置的`Average`扩展方法，如下面的代码所示：

```cs

静态的

void

CustomExtensionMethods

()

{

使用

（Northwind db = new

（）

{

WriteLine("平均库存量：{0:N0}"

，

db.Products.Average(p => p.UnitsInStock));

WriteLine("平均单价：{0:$#,##0.00}"

，

db.Products.Average(p => p.UnitPrice));

WriteLine("中位数库存量：{0:N0}"

，

db.Products.Median(p => p.UnitsInStock));

WriteLine("中位数单价：{0:$#,##0.00}"

，

db.Products.Median(p => p.UnitPrice));

WriteLine("众数库存量：{0:N0}"

，

db.Products.Mode(p => p.UnitsInStock));

WriteLine("众数单价：{0:$#,##0.00}"

，

db.Products.Mode(p => p.UnitPrice));

}

}

```

1.  在`Program.cs`中，注释掉之前的方法调用，并调用`CustomExtensionMethods`。

1.  运行代码并查看结果，如下面的输出所示：

```cs

平均库存量：41

平均单价：$28.87

中位数库存量：26

中位数单价：$19.50

众数库存量：0

众数单价：$18.00

```

有四种单价为$18.00 的产品。有五种产品库存为 0。

# 使用 LINQ to XML

**LINQ to XML**是一个 LINQ 提供程序，允许你查询和操作 XML。

## 使用 LINQ to XML 生成 XML

让我们创建一个方法将`Products`表转换成 XML：

1.  在`LinqWithEFCore`项目中，在`Program.cs`的顶部，导入`System.Xml.Linq`命名空间。

1.  在`Program.cs`的底部，创建一个方法以 XML 格式输出产品，如下面的代码所示：

```cs

静态的

void

OutputProductsAsXml

()

{

使用

（Northwind db = new

（）

{

Product[] productsArray = db.Products.ToArray();

XElement xml = new

("products"

，

从

p

在

productsArray

select

新的

XElement

(

"产品"

，

new

XAttribute(

"id"

，  p.ProductId

），

new

XAttribute

(

"price"

， p.UnitPrice

），

新的

XElement

(

"name"

， p.ProductName

)))

;

WriteLine(xml.ToString());

}

}

```

1.  在`Program.cs`中，注释掉之前的方法调用，并调用`OutputProductsAsXml`。

1.  运行代码，查看结果，并注意生成的 XML 结构与 LINQ to XML 语句中声明的元素和属性匹配，如下面部分输出所示：

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

1.  修改其内容，如下标记所示：

```cs

<?xml version="1.0" encoding="utf-8"?>

<

应用程序设置

<

添加

键

=

"颜色"

值

=

“红色”

/>

<

添加

键

=

"大小"

值

=

"大"

/>

<

添加

键

=

"价格"

值

=

"23.99"

/>

</

应用程序设置

```

如果您正在使用 Windows 的 Visual Studio 2022，则编译的应用程序将在`LinqWithEFCore\bin\Debug\net6.0`文件夹中执行，因此它将无法找到`settings.xml`文件，除非我们指示它应始终复制到输出目录。

1.  在**解决方案资源管理器**中，右键单击`settings.xml`文件，然后选择**属性**。

1.  在**属性**中，将**复制到输出目录**设置为**始终复制**。

1.  在`Program.cs`的底部，创建一个方法来完成这些任务，如下面的代码所示：

+   加载 XML 文件。

+   使用 LINQ to XML 搜索名为`appSettings`及其后代名为`add`的元素。

+   将 XML 投影到具有`Key`和`Value`属性的匿名类型数组中。

+   枚举数组以显示结果：

```cs

静态

空

ProcessSettings

()

{

XDocument doc = XDocument.Load("settings.xml"

);

变量

appSettings = doc.Descendants("appSettings"

)

.Descendants("add"

)

.Select(node => new

{

键 = 节点.属性("键"

)?.Value,

值 = 节点.属性("值"

)?.Value

}).ToArray();

foreach

（var

项目中

appSettings)

{

WriteLine($"

{item.Key}

：

{item.Value}

"

);

}

}

```

1.  在`Program.cs`中，注释掉先前的方法调用，然后调用`ProcessSettings`。

1.  运行代码并查看结果，如下面的输出所示：

```cs

颜色：红色

大小：大

价格：23.99

```

# 练习和探索

通过回答一些问题来测试您的知识和理解，进行一些动手实践，并深入研究本章涵盖的主题。

## 练习 11.1-测试您的知识

回答以下问题：

1.  LINQ 的两个必需部分是什么？

1.  您将使用哪个 LINQ 扩展方法来返回类型的属性子集？

1.  您将使用哪个 LINQ 扩展方法来过滤序列？

1.  列出执行聚合的五个 LINQ 扩展方法。

1.  `Select`和`SelectMany`扩展方法之间有什么区别？

1.  `IEnumerable<T>`和`IQueryable<T>`之间有什么区别？如何在它们之间切换？

1.  泛型`Func`委托中的最后一个类型参数`T`代表什么？

1.  以`OrDefault`结尾的 LINQ 扩展方法有什么好处？

1.  为什么查询理解语法是可选的？

1.  如何创建自己的 LINQ 扩展方法？

## 练习 11.2-使用 LINQ 进行查询

在`Chapter11`解决方案/工作区中，创建一个控制台应用程序，名为`Exercise02`，提示用户输入一个城市，然后列出该城市的 Northwind 客户的公司名称，如下面的输出所示：

```cs

输入一个城市的名称：伦敦

伦敦有 6 位客户：

环绕角

B 的饮料

合并控股

东方连接

北/南

七海进口

```

然后，增强应用程序，通过显示客户已经居住的所有唯一城市的列表，作为用户输入其首选城市之前的提示，如下面的输出所示：

```cs

Aachen，Albuquerque，Anchorage，Århus，Barcelona，Barquisimeto，Bergamo，Berlin，Bern，Boise，Bräcke，Brandenburg，Bruxelles，Buenos Aires，Butte，Campinas，Caracas，Charleroi，Cork，Cowes，Cunewalde，Elgin，Eugene，Frankfurt a.M.，Genève，Graz，Helsinki，I. de Margarita，Kirkland，Kobenhavn，Köln，Lander，Leipzig，Lille，Lisboa，London，Luleå，Lyon，Madrid，Mannheim，Marseille，México D.F.，Montréal，München，Münster，Nantes，Oulu，Paris，Portland，Reggio Emilia，Reims，Resende，Rio de Janeiro，Salzburg，San Cristóbal，San Francisco，Sao Paulo，Seattle，Sevilla，Stavern，Strasbourg，Stuttgart，Torino，Toulouse，Tsawassen，Vancouver，Versailles，Walla Walla，Warszawa

```

## 练习 11.3 - 探索主题

使用以下页面上的链接，了解本章涵盖的主题的更多细节：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-11---querying-and-manipulating-data-using-linq`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-11---querying-and-manipulating-data-using-linq)

# 摘要

在本章中，您学习了如何编写 LINQ 查询来选择、投影、过滤、排序、连接和分组数据，这些是您每天都会执行的任务。

在下一章中，您将使用`Task`类型来提高应用程序的性能。
