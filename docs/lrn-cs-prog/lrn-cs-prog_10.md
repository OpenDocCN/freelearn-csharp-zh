# 第十章：Lambda、LINQ 和函数式编程

尽管 C#在其核心是一种面向对象的编程语言，但它实际上是一种*多范式语言*。到目前为止，在本书中，我们已经讨论了命令式编程、面向对象编程和泛型编程。然而，C#也支持函数式编程特性。在*第七章*、*集合*和*第八章*、*高级主题*中，我们已经使用了其中一些，比如 lambda 和**语言集成查询（LINQ）**。

在本章中，我们将从功能编程的角度详细讨论这些内容。学习函数式编程技术将帮助您以声明性的方式编写代码，通常比等效的命令式代码更简单、更容易理解。

本章将涵盖以下主题：

+   函数式编程

+   函数作为一等公民

+   Lambda 表达式

+   LINQ

+   更多函数式编程概念

通过本章的学习，您将能够详细了解 lambda 表达式，并能够与 LINQ 一起查询各种来源的数据。此外，您将熟悉函数式编程的概念和技术，如高阶函数、闭包、单子和幺半群。

让我们从功能编程及其核心原则的概述开始这一章。

# 函数式编程

C#是一种通用的多范式编程语言。然而，到目前为止，在本书中，我们只涵盖了命令式编程范式，它使用语句来改变程序状态，并且专注于描述程序的操作方式。在命令式编程中，函数可能具有副作用，因此在执行时改变程序状态。或者，函数的执行可能取决于程序状态。

相反的范式是函数式编程，它关注描述程序做什么而不是如何做。函数式编程将计算视为函数的评估；它使用不可变数据并避免改变状态。函数式编程是一种声明性的编程范式，其中使用表达式而不是语句。函数不再具有副作用，而是幂等的。这意味着使用相同参数调用函数每次都会产生相同的结果。

函数式编程提供了几个优势，包括以下内容：

+   由于函数不改变状态，只依赖于它们接收的参数，代码更容易理解和维护。

+   由于数据是不可变的，函数没有副作用，因此更容易测试代码。

+   由于数据是不可变的，函数没有副作用，实现并发更简单高效，这避免了数据竞争。

`Rectangle`（这也可以是一个类）代表一个矩形：

```cs
struct Rectangle
{
    public int Left;
    public int Right;
    public int Top;
    public int Bottom;
    public int Width { get { return Right - Left; } }
    public int Height { get { return Bottom - Top; } }
    public Rectangle(int l, int t, int r, int b)
    {
        Left = l;
        Top = t;
        Right = r;
        Bottom = b;
    }
}
```

我们可以实例化这种类型并改变它的属性。例如，如果我们想要将矩形的宽度增加 10 个单位，每个方向都相等，我们可以这样做：

```cs
var r = new Rectangle(10, 10, 30, 20);
r.Left -= 5;
r.Right += 5;
r.Top -= 5;
r.Bottom += 5;
```

我们还可以编写一个我们可以调用的函数。这可以是一个*成员函数*，如下所示：

```cs
public void Inflate(int l, int t, int r, int b)
{
    Left -= l;
    Right += r;
    Top -= t;
    Bottom += b;
}
// invoked as
r.Inflate(5, 0, 5, 0);
```

这也可以是一个*非成员函数*，如下面的代码所示。两者之间的区别只是设计上的问题。如果我们无法修改源代码，将其编写为扩展方法是唯一的选择：

```cs
static void Inflate(ref Rectangle rect, 
                    int l, int t, int r, int b)
{
    rect.Left -= l;
    rect.Right += r;
    rect.Top -= t;
    rect.Bottom += b;
}
// invoked as
Inflate(ref r, 5, 0, 5, 0);
```

`Rectangle`数据类型是可变的，因为它的状态可以改变。`Inflate()`方法具有副作用，因为它改变了矩形的状态。在函数式编程中，`Rectangle`应该是不可变的。可能的实现如下所示：

```cs
struct Rectangle
{
    public readonly int Left;
    public readonly int Right;
    public readonly int Top;
    public readonly int Bottom;
    public int Width { get { return Right - Left; } }
    public int Height { get { return Bottom - Top; } }
    public Rectangle(int l, int t, int r, int b)
    {
        Left = l;
        Top = t;
        Right = r;
        Bottom = b;
    }
}
```

`Inflate()`方法的纯函数版本不会产生副作用。它的行为仅取决于参数，结果将是相同的，无论调用多少次具有相同参数。这样的实现示例如下：

```cs
static Rectangle Inflate(Rectangle rect, 
                         int l, int t, int r, int b)
{
    return new Rectangle(rect.Left - l, rect.Top - t,
                         rect.Right + r, rect.Bottom + b);
}
```

现在可以像下面的例子一样使用它们：

```cs
var r = new Rectangle(10, 10, 30, 20);
r = Inflate(r, 5, 0, 5, 0);
```

函数式编程源自λ演算（由阿隆佐·邱奇开发），它是一个基于函数抽象和应用的计算表达的框架或数学系统，使用变量绑定和替换。一些编程语言，比如 Haskell，是纯函数式的。其他的，比如 C#，支持多种范式，不是纯函数式的。

前面的例子展示了一个变量`r`，它被初始化为一个值，然后被改变。在纯函数式编程中，这是不可能的。一旦初始化，变量就不能改变值；而是必须分配一个新的变量。这使得表达式可以被它们的值替换，这是**引用透明性**的一个特性。

C#使我们能够使用函数式编程的概念和习语来编写代码。所有这些的核心都是 lambda 表达式，我们将很快深入研究。在那之前，我们需要探索另一个函数式编程的支柱，那就是将函数视为*一等公民*。

# 函数作为一等公民

在*第八章*《高级主题》中，我们学习了关于委托和事件。委托看起来像一个函数，但它是一种保存与委托定义匹配的函数引用的类型。委托实例可以作为函数参数传递。让我们看一个例子，其中有一个委托接受两个`int`参数并返回一个`int`值：

```cs
public delegate int Combine(int a, int b);
```

然后我们有不同的函数，比如`Add()`，它可以将两个整数相加并返回和，`Sub()`，它可以将两个整数相减并返回差，或者`Mul()`，它可以将两个整数相乘并返回积。它们的签名与委托匹配，因此`Combine`委托的实例可以保存对所有这些函数的引用。这些函数如下所示：

```cs
class Math
{
    public static int Add(int a, int b) { return a + b; }
    public static int Sub(int a, int b) { return a - b; }
    public static int Mul(int a, int b) { return a * b; }
}
```

我们可以编写一个通用函数，可以将其中一个函数应用于两个参数。这样的函数可能如下所示：

```cs
int Apply(int a, int b, Combine f)
{
    return f(a, b);
}
```

调用它很简单——我们传递参数和我们想要调用的实际函数的引用：

```cs
var s = Apply(2, 3, Math.Add);
var d = Apply(2, 3, Math.Sub);
var p = Apply(2, 3, Math.Mul);
```

为了方便，.NET 定义了一组名为`Func`的通用委托，以避免一直定义自己的委托。这些定义在`System`命名空间中，如下所示：

```cs
public delegate TResult Func<out TResult>();
public delegate TResult Func<in T,out TResult>(T arg);
public delegate TResult Func<in T1,in T2,out TResult>(T1 arg1, T2 arg2);
...
public delegate TResult Func<in T1,in T2,in T3,in T4,in T5,in T6,in T7,in T8,in T9,in T10,in T11,in T12,in T13,in T14,in T15,in T16,out TResult>(T1 arg1, T2 arg2, T3 arg3, T4 arg4, T5 arg5, T6 arg6, T7 arg7, T8 arg8, T9 arg9, T10 arg10, T11 arg11, T12 arg12, T13 arg13, T14 arg14, T15 arg15, T16 arg16);
```

这是一组有 17 个重载的函数，可以接受 0、1 或多达 16 个参数（可能是不同类型的），并返回一个值。使用这些系统委托，我们可以将`Apply`函数重写如下：

```cs
T Apply<T>(T a, T b, Func<T, T, T> f)
{
    return f(a, b);
}
```

这个版本的函数是通用的，因此它可以用其他类型的参数来调用，而不仅仅是整数。在前面的例子中调用函数的方式并没有改变。

这些委托返回一个值，因此不能用于没有返回值的函数。在`System`命名空间中有一组类似的重载，称为`Action`，定义如下：

```cs
public delegate void Action();
public delegate void Action<in T>(T obj);
public delegate void Action<in T1,in T2>(T1 arg1, T2 arg2);
...
public delegate void Action<in T1,in T2,in T3,in T4,in T5,in T6,in T7,in T8,in T9,in T10,in T11,in T12,in T13,in T14,in T15,in T16>(T1 arg1, T2 arg2, T3 arg3, T4 arg4, T5 arg5, T6 arg6, T7 arg7, T8 arg8, T9 arg9, T10 arg10, T11 arg11, T12 arg12, T13 arg13, T14 arg14, T15 arg15, T16 arg16);
```

这些委托与我们之前看到的`Func`委托认非常相似。唯一的区别是它们不返回值。仍然有 17 个重载，可以接受 0、1 或多达 16 个输入参数。

在下面的例子中，`Apply`函数被重载，以便它还接受`Action<string>`类型的参数，这是一个具有`string`类型的单个参数并且不返回任何值的函数。在应用函数之后，但在返回结果之前，将调用此操作，并传递描述实际操作的字符串：

```cs
T Apply<T>(T a, T b, Func<T, T, T> f, Action<string> log)
{
    var r = f(a, b);
    log?.Invoke($"{f.Method.Name}({a},{b}) = {r}");
    return r;
}
```

我们可以通过将`Console.WriteLine`作为最后一个参数传递来调用这个新的重载，这样操作就会被记录到控制台上：

```cs
var s = Apply(2, 3, Math.Add, Console.WriteLine);
var p = Apply(2, 3, Math.Mul, Console.WriteLine);
```

`Apply`函数被称为*高阶函数*。高阶函数是一个接受一个或多个函数作为参数、返回一个函数或两者都有的函数。其他所有的函数都被称为*一阶函数*。

有许多高阶函数可能会在没有意识到的情况下使用。例如，`List<T>.Sort (Comparison<T> comparison)`就是这样一个函数。LINQ 中的大多数查询谓词（我们将在本章的*LINQ*部分中探讨）都是高阶函数。

高阶函数的一个例子是返回另一个函数的函数，如下面的代码片段所示。`ApplyReverse()`接受一个函数作为参数，并返回另一个函数，该函数以两个参数调用参数函数，但顺序相反：

```cs
Func<T, T, T> ApplyReverse<T>(Func<T, T, T> f)
{
    return delegate(T a, T b) { return f(b, a); };
}
```

这个函数被调用如下：

```cs
var s = ApplyReverse<int>(Math.Add)(2, 3);
var d = ApplyReverse<int>(Math.Sub)(2, 3);
```

到目前为止，我们所看到的是在 C#中将函数作为参数传递，从函数中返回函数，将函数分配给变量，将它们存储在数据结构中，或者定义匿名函数（即没有名称的函数）的可能性。还可以嵌套函数并测试函数的引用是否相等。一个能做到这些的编程语言被称为将函数视为一等公民，并且它的函数是一等公民。因此，C#就是这样一种语言。

回到之前的例子，调用`Apply()`方法的另一种更简单的方法如下：

```cs
var s = Apply(2, 3, (a, b) => a + b);
var d = Apply(2, 3, (a, b) => a - b);
var p = Apply(2, 3, (a, b) => a * b);
```

在这里，`Math`类的方法已被替换为诸如`(a, b) => a + b`这样的 lambda 表达式。我们甚至可以将`Apply()`函数定义为 lambda 表达式并相应地调用它：

```cs
Func<int, int, Func<int, int, int>, int> apply = 
   (a, b, f) => f(a, b);
var s = apply(2, 3, (a, b) => a + b);
var d = apply(2, 3, (a, b) => a - b);
var p = apply(2, 3, (a, b) => a * b);
```

我们将在下一节深入研究 lambda 表达式。

# Lambda 表达式

Lambda 表达式是一种方便的写匿名函数的方式。它们是一段代码，可以是一个表达式或一个或多个语句，表现得像一个函数，并且可以被分配给一个委托。因此，lambda 表达式可以作为参数传递给函数或从函数中返回。它们是编写 LINQ 查询、将函数传递给高阶函数（包括应该由`Task.Run()`异步执行的代码）以及创建表达式树的一种方便方式。

表达式树是一种以树状数据结构表示代码的方式，其中节点是表达式（如方法调用或二进制操作）。这些表达式树可以被编译和执行，从而使可执行代码能够进行动态更改。表达式树用于实现各种数据源的 LINQ 提供程序以及 DLR 中的.NET Framework 和动态语言之间的互操作性。

让我们从一个简单的例子开始，我们有一个整数列表，我们想要从中删除所有的奇数。可以写成如下形式（注意`IsOdd()`函数可以是类方法，也可以是本地函数）：

```cs
bool IsOdd(int n) { return n % 2 == 1; }
var list = new List<int>() { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
list.RemoveAll(IsOdd);
```

这段代码实际上可以用匿名方法来简化，允许我们将代码传递给委托，而无需定义单独的`IsOdd()`函数：

```cs
var list = new List<int>() { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
list.RemoveAll(delegate (int n) { return n % 2 == 1; });
```

Lambda 表达式允许我们使用更简单的语法进一步简化代码，编译器将其转换为类似于前面代码的内容：

```cs
var list = new List<int>() { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
list.RemoveAll(n => n % 2 == 1);
```

我们在这里看到的 lambda 表达式（`n => n % 2 == 1`）有两部分，由`=>`分隔，这是**lambda 声明运算符**：

+   表达式的左部是*参数列表*（如果有多个参数，则用逗号分隔并括在括号中）。

+   表达式的右部要么是*表达式，要么是语句*。如果右部是表达式（就像前面的例子中），lambda 被称为**表达式 lambda**。如果右部是一个语句，lambda 被称为**语句 lambda**。

语句总是用大括号`{}`括起来。任何表达式 lambda 实际上都可以写成一个语句 lambda。表达式 lambda 是语句 lambda 的简化版本。前面的例子使用表达式 lambda 可以写成以下形式的语句 lambda：

```cs
var list = new List<int>() { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
list.RemoveAll(n => { return n % 2 == 1; });
```

有几个 lambda 表达式的例子：

![](img/Chapter_10_Table_1_01.jpg)

lambda 没有自己的类型。相反，它的类型要么是分配给它的委托的类型，要么是当 lambda 用于构建表达式树时的`System.Expression`类型。不返回值的 lambda 对应于`System.Action`委托（并且可以分配给一个）。返回值的 lambda 对应于`System.Func`委托。

当你写一个 lambda 表达式时，你不需要写参数的类型，因为这些类型是由编译器推断的。类型推断的规则如下：

+   lambda 必须具有与其分配的委托相同数量的参数。

+   lambda 的每个参数必须隐式转换为它所分配的委托的对应参数。

+   如果 lambda 有返回值，它的类型必须隐式转换为它所分配的委托的返回类型。

Lambda 表达式可以是异步的。这样的 lambda 前面要加上`async`关键字，并且必须包含至少一个`await`表达式。下面的例子展示了一个 Windows Forms 表单上按钮的`Click`事件的异步处理程序：

```cs
public partial class MyForm : Form
{
    public MyForm()
    {
        InitializeComponent();

        myButton.Click += async (sender, e) =>
        {
            await ExampleMethodAsync();
        };
    }
    private async Task ExampleMethodAsync()
    {
        // a time-consuming action
        await Task.Delay(1000);
    }
}
```

在这个例子中，`MyForm`是一个表单类，在它的构造函数中，我们注册了一个`Click`事件的处理程序。这是使用 lambda 表达式完成的，但 lambda 是异步的（它调用一个异步函数），因此需要在前面加上`async`。

lambda 可以使用在方法或包含 lambda 表达式的类型范围内的变量。当变量在 lambda 中使用时，它被捕获，以便即使超出范围也可以使用。这些变量在 lambda 中使用之前必须被明确赋值。在下面的例子中，lambda 表达式捕获了两个变量——`value`函数参数和`Data`类成员：

```cs
class Foo
{
    public int Data { get; private set; }
    public Foo(int value)
    {
        Data = value;
    }
    public void Scramble(int value, int iterations)
    {
        Func<int, int> apply = (i) => Data ^ i + value;
        for(int i = 0; i < iterations; ++i)
           Data = apply(i);
    }
}
```

以下是 lambda 表达式中变量作用域的规则：

+   lambda 表达式中引入的变量在 lambda 之外是不可见的（例如，在封闭方法中）。

+   lambda 不能捕获封闭方法的`in`、`ref`或`out`参数。

+   被 lambda 表达式捕获的变量不会被垃圾回收，即使它们本来会超出范围，直到 lambda 分配的委托被垃圾回收。

+   lambda 表达式的返回语句仅指代 lambda 所代表的匿名方法，并不会导致封闭方法返回。

lambda 表达式最常见的用例是编写 LINQ 查询表达式。我们将在下一节中看到这一点。

# LINQ

LINQ 是一组技术，使开发人员能够以一致的方式查询多种数据源。通常，您会使用不同的语言和技术来查询不同类型的数据，比如关系数据库使用 SQL，XML 使用 XPath。SQL 查询是以字符串形式编写的，这使得它们无法在编译时进行验证，并增加了运行时错误的可能性。

LINQ 定义了一组操作符和用于查询数据的内置语言语法。LINQ 查询是强类型的，因此在编译时进行验证。LINQ 还提供了一个框架，用于构建自己的 LINQ 提供程序，这些提供程序是将查询转换为特定于特定数据源的 API 的组件。该框架提供了对查询对象（.NET 中的任何集合）、关系数据库和 XML 的内置支持。第三方已经为许多数据源编写了 LINQ 提供程序，比如 Web 服务。

LINQ 使开发人员能够专注于要做什么，而不太关心如何做。为了更好地理解这是如何工作的，让我们看一个例子，我们有一个整数数组，我们想找到所有奇数的和。通常，您会写类似以下的内容：

```cs
int[] arr = { 1, 1, 3, 5, 8, 13, 21, 34};
int sum = 0;
for(int i = 0; i < arr.Length; ++i)
{
    if (arr[i] % 2 == 1)
    sum += arr[i];
}
```

使用 LINQ，可以将所有这些冗长的代码简化为以下一行：

```cs
int sum = arr.Where(x => x % 2 == 1).Sum();
```

在这里，我们使用了 LINQ 标准查询操作符，它们是作用于序列的扩展方法，提供了包括过滤、投影、聚合、排序等在内的查询功能。然而，许多这些查询操作符在 LINQ 查询语法中都有直接的支持，这是一种非常类似于 SQL 的查询语言。使用查询语言，解决问题的方案可以写成如下形式：

```cs
int sum = (from x in arr
           where x % 2 == 1
           select x).Sum();
```

正如你在这个例子中所看到的，不是每个查询操作符都有查询语法中的等价物。`Sum()`和所有其他聚合操作符都没有等价物。在接下来的章节中，我们将更详细地研究这两种 LINQ 的用法。

## 标准查询操作符

LINQ 标准查询操作符是一组作用于实现`IEnumerable<T>`或`IQueryable<T>`的序列的扩展方法。前者导出一个允许对序列进行迭代的枚举器。后者是一个特定于 LINQ 的接口，它继承自`IEnumerable<T>`并为我们提供了对特定数据源进行查询的功能。标准查询操作符被定义为作用于`Enumerable`或`Queryable`类的扩展方法，具体取决于它们操作的序列的类型。作为扩展方法，它们可以使用静态方法语法或实例方法语法进行调用。

大多数查询操作符可能返回多个值。这些方法返回`IEnumerable<T>`或`IQueryable<T>`，这使得它们可以链接在一起。它们返回的可枚举对象上的实际查询在迭代时被推迟到数据源上。另一方面，返回单个值的标准查询操作符（如`Sum()`或`Count()`）不推迟执行并立即执行。

以下表格包含了所有 LINQ 标准查询操作符的名称：

![](img/Chapter_10_Table_2_01.jpg)

标准查询操作符的数量很大。讨论它们中的每一个超出了本书的范围。你应该阅读官方文档或其他资源，以熟悉它们所有。

为了更加熟悉 LINQ，我们将看几个例子。在第一个例子中，我们想要计算句子中的单词数量。我们将句子以句号（`.`）、逗号（`,`）和空格作为分隔符。我们将字符串分割成部分，然后过滤掉所有非空的部分并计数它们。使用 LINQ，这就像下面这样简单：

```cs
var text = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.";
var count = text.Split(new char[] { ' ', ',', '.' })
                .Where(w => !string.IsNullOrEmpty(w))
                .Count();
```

然而，如果我们想要根据它们的长度对所有单词进行分组并将它们打印到控制台上，问题就变得有点复杂了。我们需要以单词长度为键创建分组，以单词本身为元素，过滤掉长度为零的分组，并根据单词长度按升序排序剩下的部分：

```cs
var groups = text.Split(new char[] { ' ', ',', '.' })
                 .GroupBy(w => w.Length, w => w.ToLower())
                 .Select(g => new { Length =g.Key, Words = g })
                 .Where(g => g.Length > 0)
                 .OrderBy(g => g.Length);
foreach (var group in groups)
{
    Console.WriteLine($"Length={group.Length}");
    foreach (var word in group.Words)
    {
        Console.WriteLine($" {word}");
    }
}
```

前一个查询在调用`Count()`时执行，而这个查询的执行被推迟到我们实际迭代它时。

到目前为止，我们看到的例子并不是太复杂。然而，使用 LINQ，你可以构建更复杂的查询。为了说明这一点，让我们考虑一个处理客户订单的系统。该系统使用`Customer`、`Article`、`OrderLine`和`Order`等实体，这里以非常简化的形式显示：

```cs
class Customer
{
    public long Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
}
class Article
{
    public long Id { get; set; }
    public string EAN13 { get; set; }
    public string Name { get; set; }
    public double Price { get; set; }
}
class OrderLine
{
    public long Id { get; set; }
    public long OrderId { get; set; }
    public long ArticleId { get; set; }
    public double Quantity { get; set; }
    public double Discount { get; set; }
}
class Order
{
    public long Id { get; set; }
    public DateTime Date { get; set; }
    public long CustomerId { get; set; }
    public double Discount { get; set; }
}
```

让我们也考虑一下我们有这些类型的序列，如下所示（为简单起见，每种类型只显示了一些记录，但你可以在本书附带的源代码中找到完整的例子）：

```cs
var articles = new List<Article>()
{
     new Article(){ Id = 1, EAN13 = "5901234123457", 
                    Name = "paper", Price = 100.0},
     new Article(){ Id = 2, EAN13 = "5901234123466", 
                    Name = "pen", Price = 200.0},
     /* more */
};
var customers = new List<Customer>()
{
     new Customer() { Id = 101, FirstName = "John", 
               LastName = "Doe", Email = "john.doe@email.com"},
     new Customer() { Id = 102, FirstName = "Jane", 
               LastName = "Doe", Email = "jane.doe@email.com"},
     /* more */
};
var orders = new List<Order>()
{
     new Order() { Id = 1001, Date = new DateTime(2020, 3, 12),
                   CustomerId = customers[0].Id },
     new Order() { Id = 1002, Date = new DateTime(2020, 4, 23),
                   CustomerId = customers[1].Id },
     /* more */
};
var orderlines = new List<OrderLine>()
{
    new OrderLine(){ Id = 1, OrderId=orders[0].Id, 
                     ArticleId = articles[0].Id, Quantity=2},
    new OrderLine(){ Id = 2, OrderId=orders[0].Id, 
                     ArticleId = articles[1].Id, Quantity=1},
    /* more */
};
```

我们想要找到答案的问题是，*一个特定客户自从某一天以来购买的所有文章的名称是什么？*使用命令式方法编写这个问题可能会很麻烦，但是使用 LINQ，可以表达如下：

```cs
var query = 
    orders.Join(orderlines,
                o => o.Id,
                ol => ol.OrderId,
                (o, ol) => new { Order = o, Line = ol })
          .Join(customers,
                o => o.Order.CustomerId,
                c => c.Id,
                (o, c) => new { o.Order, o.Line, Customer = c})
          .Join(articles,
                o => o.Line.ArticleId,
                a => a.Id,
                (o, a) => new { o.Order, o.Line, 
                               o.Customer, Article = a})
        .Where(o => o.Order.Date >= new DateTime(2020, 4, 1) &&
                    o.Customer.FirstName == "John")
          .OrderBy(o => o.Article.Name) 
          .Select(o => o.Article.Name);
```

在这个例子中，我们将订单与订单行和客户进行了连接，并将订单行与文章进行了连接，并且只保留了 2020 年 4 月 1 日后由名为 John 的客户下的订单。然后，我们按文章名称的字典顺序对它们进行了排序，并只选择了文章名称进行投影。

有几个`Join()`操作，语法可能看起来更难理解。让我们使用以下例子来解释一下：

```cs
orders.Join(orderlines,
            o => o.Id,
            ol => ol.OrderId,
            (o, ol) => new { Order = o, Line = ol })
```

在这里，`orders`被称为*外部序列*，`orderlines`被称为*内部序列*。`Join()`的第二个参数，即`o => o.Id`，被称为*外部序列的键选择器*。我们用它来选择订单。`Join()`的第三个参数，即`ol => ol.OrderId`，被称为*内部序列的键选择器*。我们用它来选择订单行。

基本上，这两个 lambda 表达式帮助匹配具有`OrderId`等于订单 ID 的订单行。最后一个参数`(o, ol) => new { Order = o, Line = ol }`是连接操作的投影。我们正在创建一个具有名为`Order`和`Line`的两个属性的新对象。

一些标准查询操作更容易使用，而其他一些可能更复杂，可能需要一些练习才能理解得很好。然而，对于其中许多操作，存在一个更简单的替代方案——LINQ 查询语法，我们将在下一节中探讨。

## 查询语法

LINQ 查询语法基本上是标准查询操作的语法糖（即，设计成更容易编写和理解的简化语法）。编译器将使用查询语法编写的查询转换为使用标准查询操作的查询。查询语法比标准查询操作更简单、更易读，但它们在语义上是等价的。然而，正如前面提到的，不是所有的标准查询操作在查询语法中都有等价物。

为了看到标准查询操作的方法语法和查询语法的比较，让我们使用查询语法重写上一节中的例子。

首先，让我们看一下在一段文本中计算单词数的问题。使用查询语法，查询变成了以下形式。请注意，`Count()`在查询语法中没有等价物：

```cs
var count = (from w in text.Split(new char[] { ' ', ',', '.' })
             where !string.IsNullOrEmpty(w)
             select w).Count();
```

另一方面，第二个问题可以完全使用查询语法来编写，如下所示：

```cs
var groups = from w in text.Split(new char[] { ' ', ',', '.' })
             group w.ToLower() by w.Length into g
             where g.Key > 0
             orderby g.Key
             select new { Length = g.Key, Words = g };
foreach (var group in groups)
{
    Console.Write($"Length={group.Length}: ");
    Console.WriteLine(string.Join(',', group.Words));
}
```

打印文本有点不同。单词以逗号分隔的形式显示在一行上。为了组成逗号分隔的单词文本，我们使用了`string.Join()`静态方法，它接受一个分隔符和一系列值，并将它们连接成一个字符串。这个程序的输出如下：

```cs
Length=2: do,ut,et
Length=3: sit,sed
Length=4: amet,elit
Length=5: lorem,ipsum,dolor,magna
Length=6: tempor,labore,dolore,aliqua
Length=7: eiusmod
Length=10: adipiscing,incididunt
Length=11: consectetur
```

我们将重写的最后一个问题是与客户订单相关的例子。这个查询可以非常简洁地表达，如下面的代码所示。这段代码类似于 SQL，`join`操作的写法确实更简单，更易读，更易理解：

```cs
var query = from o in orders
            join ol in orderlines on o.Id equals ol.OrderId
            join c in customers on o.CustomerId equals c.Id
            join a in articles on ol.ArticleId equals a.Id
            where o.Date >= new DateTime(2019, 4, 1) &&
                  c.FirstName == "John"
            orderby a.Name
            select a.Name;
```

从这些例子中可以看出，LINQ 帮助以比传统的命令式编程更简单的方式构建查询。不同性质的数据源可以以类似 SQL 的语言一致地进行查询。查询是强类型的，并且在编译时进行验证，这有助于解决许多潜在的错误。

现在，让我们来看一些更多的函数式编程概念：部分函数应用、柯里化、闭包、幺半群和单子。

# 更多的函数式编程概念

在本章的开头，我们看了一般的函数式编程概念，主要是高阶函数和不可变性。在本节中，我们将探讨几个更多的函数式编程概念和技术——部分函数应用、柯里化、闭包、幺半群和单子。

## 部分函数应用

部分函数应用是将具有*N 个参数*和*一个参数*的函数进行处理，并在将参数固定为函数的一个参数后返回具有*N-1 个参数*的另一个函数的过程。当然，也可能会使用多个参数进行调用，比如*M*，在这种情况下返回的函数将具有*N-M*个参数。

要理解这是如何工作的，让我们从一个具有多个参数并返回一个字符串（包含参数值）的函数开始：

```cs
string AsString(int a, double b, string c)
{
    return $"a={a}, b={b}, c={c}";
}
```

如果我们将这个函数作为`AsString(42, 43.5, "44")`调用，结果将是字符串`"a=42, b=43.5, c=44"`。然而，如果我们有一个函数（让我们称之为`Apply()`）可以将一个参数绑定到这个函数的第一个参数，那么我们可以用相同的结果来调用它：

```cs
var f1 = Apply<int, double, string, string>(AsString, 42);
var result = f1(43.5, "44");
```

实现这样一个`Apply()`函数的方法如下：

```cs
Func<T2, T3, TResult>
Apply<T1, T2, T3, TResult>(Func<T1, T2, T3, TResult> f, T1 arg)
{
    return (b, c) => f(arg, b, c);
}
```

这个高阶函数接受另一个函数和一个值作为参数，并返回另一个参数少一个的高阶函数。这个函数解析为使用`f`参数函数和`arg`参数值以及其他参数。

也可能继续将函数减少到另一个参数少一个的函数，直到我们有一个没有参数的函数，如下所示：

```cs
var f1 = Apply<int, double, string, string>(AsString, 42);
var f2 = Apply(f1, 43.5);
var f3 = Apply(f2, "44");
string result = f3();
```

然而，要实现这一点，我们需要`Apply()`函数的额外重载，以及相应数量的参数。对于这里显示的情况，我们需要以下内容（实际上，如果你有超过三个参数的函数，你需要更多的重载来考虑所有可能的参数数量）：

```cs
Func<T2, TResult> Apply<T1, T2, TResult>(Func<T1, T2, TResult> f, T1 arg)
{
    return b => f(arg, b);
}
Func<TResult> Apply<T1, TResult>(Func<T1, TResult> f, T1 arg)
{
    return () => f(arg);
}
```

在这个例子中，重要的是要注意，只有当所有参数都提供时，才会实际调用`AsString()`函数；也就是说，当我们调用`f3()`时。

你可能想知道部分函数应用何时有用。典型情况是当你多次（或多次）调用一个函数，而一些参数是相同的。在这种情况下，有几种替代方案，包括以下几种：

+   在定义函数时为函数参数提供默认值。然而，由于不同的原因，这可能是不可能的。也许默认值只在某些情况下有意义，或者你实际上并不拥有这段代码，所以无法提供默认值。

+   在多次调用函数的类中，可以编写一个带有较少参数的`helper`函数，以使用正确的默认值调用函数。

部分函数应用可能是（在许多情况下）更简单的解决方案。

## 柯里化

**柯里化**是将具有*N*个参数的函数分解为*接受一个*参数的*N*个函数的过程。这种技术得名于数学家和逻辑学家 Haskell Curry，函数式编程语言**Haskell**也是以他的名字命名的。

柯里化使得能够在只能使用一个参数的情况下使用具有多个参数的函数。数学中的分析技术就是一个例子，它只能应用于具有单个参数的函数。

考虑到上一节中的`AsString()`函数，对这个函数进行柯里化将会做如下操作：

+   返回一个函数`f1`。

+   当使用参数`a`调用时，它将返回一个函数`f2`。

+   当使用参数`b`调用时，它将返回一个函数`f3`。

+   当使用参数`c`调用时，它将调用`AsString(a, b, c)`。

将这些放入代码中，看起来如下：

```cs
var f1 = Curry<int, double, string, string>(AsString);
var f2 = f1(42);
var f3 = f2(43.5);
string result = f3("44");
```

在这里看到的通用`Curry()`函数类似于上一节中的`Apply()`函数。但是，它返回的不是具有*N-1*个参数的函数，而是具有一个参数的函数：

```cs
Func<T1, Func<T2, Func<T3, TResult>>> 
Curry<T1, T2, T3, TResult>(Func<T1, T2, T3, TResult> f)
{
    return a => b => c => f(a, b, c);
}
```

这个函数可以用于柯里化具有三个参数的函数。如果你需要对具有其他参数数量的函数进行柯里化，那么你需要适当的重载（就像在`Apply()`的情况下一样）。

您应该注意，您不一定需要将`AsString()`函数分解为三个不同的函数，就像之前的`f1`，`f2`和`f3`一样。您可以跳过中间函数，并通过适当调用函数来实现相同的结果，如下面的代码所示：

```cs
var f = Curry<int, double, string, string>(AsString);
string result = f(42)(43.5)("44");
```

函数编程中的另一个重要概念是闭包。我们将在下一节学习有关闭包的知识。

## 闭包

**闭包**被定义为在具有头等函数的语言中实现词法范围名称绑定的技术。词法或静态作用域是将变量的作用域设置为定义它的块，因此只能在该作用域内通过其名称引用它。

信息框

C#中的作用域称为**静态**或**词法**，可以在编译时查看。相反的是*动态作用域*，它只在运行时解析，但在 C#中不支持这种作用域。

正如我们在本章前面看到的，C#是一种具有头等函数的语言，因为您可以将函数分配给变量，传递它们并调用它们。然而，这种对闭包的定义可能更难理解，因此我们将使用一个示例逐步解释它。

让我们考虑以下示例：

```cs
class Program
{
    static Func<int, int> Increment()
    {
        int step = 1;
        return x => x + step;
    }
    static void Main(string[] args)
    {
        var inc = Increment();
        Console.WriteLine(inc(42));
    }
}
```

在这里，我们有一个名为`Increment()`的函数，它返回另一个函数，该函数使用一个值递增其参数。然而，该值既不作为参数传递给 lambda，也不在 lambda 中定义为局部变量。相反，它是从外部范围捕获的。因此，在 lambda 的范围内，step 变量被称为`step`变量；如果在那里找不到它，它会查找封闭范围，这种情况下是`Increment()`函数。如果那里也找不到它，它将进一步查找类范围，依此类推。

接下来发生的是，我们将从`Increment()`函数返回的值（另一个函数）分配给`inc`变量，然后使用值`42`调用它。结果是将值`43`打印到控制台。

问题是，*这是如何工作的？* `step`变量实际上是一个局部函数变量，应该在调用`Increment()`后立即超出范围。然而，在调用从`Increment()`返回的函数时，它的值是已知的。这是因为 lambda 表达式`x => x + step`被认为是*闭合*在自由变量`step`上，从而定义了一个闭包。lambda 表达式和`step`一起传递（作为闭包的一部分），以便变量通常会超出范围，但在调用闭包时仍然存在。

闭包经常被使用，而我们甚至没有意识到。考虑以下示例，我们有一个引擎列表，我们想要搜索具有最小功率和容量的引擎。您通常会使用 lambda 表达式编写如下内容：

```cs
var list = new List<Engine>();
var minp = 75.0;
var minc = 1600;
var engine = list.Find(e => e.Power >= minp && 
                       e.Capacity >= minc);
```

但这实际上创建了一个闭包，因为 lambda 闭合了`minp`和`minc`自由变量。如果语言不支持闭包，编写相同功能的代码将会很麻烦。你基本上需要编写一个捕获这些变量值的类，并且有一个方法，该方法接受一个`Engine`对象并将其属性与这些值进行比较。在这种情况下，代码可能如下所示：

```cs
sealed class EngineFinder
{
    public EngineFinder(double minPower, int minCapacity)
    {
        this.minPower = minPower;
        this.minCapacity = minCapacity;
    }
    public double minPower;
    public int minCapacity;
    public bool IsMatch(Engine engine)
    {
        return engine.Power >= minPower && 
            engine.Capacity >= minCapacity;
    }
}
var engine = list.Find(new EngineFinder(minp, minc).IsMatch);
```

这与编译器在遇到闭包时所做的事情非常相似，但这是你不必担心的细节。

您还应该注意，lambda 中捕获的自由变量的值可以改变。我们通过以下示例来说明这一点，其中`GetNextId()`函数定义了一个闭包，该闭包在每次调用时递增捕获的自由变量`id`的值：

```cs
Func<int> GetNextId()
{
    int id = 1;
    return () => id++;
}
var nextId = GetNextId();
Console.WriteLine(nextId()); // prints 1
Console.WriteLine(nextId()); // prints 2
Console.WriteLine(nextId()); // prints 3
```

我们将在下一节学习有关单子的知识。

## 单子

**单子**是一种具有单一可结合二元操作和单位元的代数结构。任何具有这两个元素的 C#类型都是单子。单子对于定义概念和重用代码非常有用。它们帮助我们从简单的组件构建复杂的行为，而无需在我们的代码中引入新的概念。让我们看看如何在 C#中创建和使用单子。

我们可以在 C#中定义一个通用接口来表示单子，如下所示：

```cs
interface IMonoid<T>
{
    T Combine(T a, T b);
    T Identity { get; }
}
```

单子确保结合性和左右单位性，以便对于任何值`a`、`b`和`c`，我们有以下内容：

+   `Combine((Combine(a, b), c) == Combine(a, Combine(b, c))`

+   `Combine(Identify, a) == a`

+   `Combine(a, Identity) == a`

连接字符串或列表是一个可结合的二元操作的例子。提供该函数的类型，以及一个单位元（在这些情况下是一个空字符串或一个空列表），就是一个单子。因此，我们实际上可以在 C#中实现这些功能，如下所示：

```cs
struct ConcatList<T> : IMonoid<List<T>>
{
    public List<T> Identity => new List<T> { };
    public List<T> Combine(List<T> a, List<T> b)
    {
        var l = new List<T>(a);
        l.AddRange(b);
        return l;
    }
}
struct ConcatString : IMonoid<string>
{
    public string Identity => string.Empty;
    public string Combine(string a, string b)
    {
        return a + b;
    }
}
```

`ConcatList`和`ConcatString`都是单子的例子。后者可以如下使用：

```cs
var m = new ConcatString();
var text = m.Combine("Learning", m.Combine(" ", "C# 8"));
Console.WriteLine(text);
```

这将在控制台上打印`Learning C# 8`。然而，这段代码有点繁琐。我们可以通过创建一个带有静态方法`Concat()`的辅助类来简化它，该方法接受一个单子和一系列元素，并使用单子的二元操作和其初始值的单位元将它们组合在一起：

```cs
static class Monoid
{
    public static T Concat<MT, T>(IEnumerable<T> seq)
        where MT : struct, IMonoid<T>
    {
       var result = default(MT).Identity;
       foreach (var e in seq)
           result = default(MT).Combine(result, e);
       return result;
    }
}
```

有了这个辅助类，我们可以编写以下简化的代码：

```cs
var text = Monoid.Concat<ConcatString, string>(
              new[] { "Learning", " ", "C# 8"});
Console.WriteLine(text);
var list = Monoid.Concat<ConcatList<int>, List<int>>(
    new[] { new List<int>{ 1,2,3},
    new List<int> { 4, 5 },
    new List<int> { } });
Console.WriteLine(string.Join(",", list));
```

在这个例子的第一部分中，我们将一系列字符串连接成一个单一的字符串并打印到控制台。在第二部分中，我们将一系列整数的列表连接成一个单一的整数列表，然后也打印到控制台。

在接下来的部分，我们将看看单子。

## 单子

这通常是一个更难解释，也许更难理解的概念，尽管已经有很多文献写过它。在这本书中，我们将尝试用简单的术语来解释它，但我们建议您阅读其他资源。

简而言之，单子是一个封装了一些功能的容器，它包裹在它的值之上。我们经常在 C#中使用单子而没有意识到。`Nullable<T>`是一个定义了特殊功能的单子，即*可空性*，这意味着一个值可能存在，也可能不存在。带有`await`的`Task<T>`是一个定义了特殊功能的单子，即*异步性*，这意味着一个值可以在实际计算之前被使用。带有 LINQ 查询`SelectMany()`操作符的`IEnumerable<T>`也是一个单子。

单子有两个操作：

+   一个将值`v`转换为包装它的容器（`v -> C(v)`）的函数。在函数式编程中，这个函数被称为**return**。

+   一个将两个容器扁平化为一个单一容器的函数（`C(C(v)) -> C(v)`）。在函数式编程中，这被称为**bind**。

让我们看下面的例子：

```cs
var numbers = new int[][]{ new[]{ 1, 2, 3},
                           new[]{ 4, 5 },
                           new[]{ 6, 7} };
IEnumerable<int> odds = numbers.SelectMany(
                           n => n.Where(x => x % 2 == 1));
```

在这里，`numbers`是一个整数数组的数组。`SelectMany()`用于选择奇数的子序列。然而，这将结果扁平化为`IEnumerable<int>`而不是`IEnumerable<IEnumerable<int>>`。正如我们之前提到的，带有`SelectMany()`的`IEnumerable<T>`是一个单子。

但是你如何在 C#中实现一个单子呢？最简单的形式如下：

```cs
class Monad<T>
{
    public Monad(T value) => Value = value;
    public T Value { get; }
    public Monad<U> Bind<U>(Func<T, Monad<U>> f) => f(Value);
}
```

实际上被称为`x => x`，你将得到初始单子：

```cs
var m = new Monad<int>(42);
var mm = new Monad<Monad<int>>(m);
var r = mm.Bind(x => x); // r equals m
```

这个单子如何使用的另一个例子在下面的代码中展示：

```cs
var m = new Monad<int>(21);
var r = m.Bind(x => new Monad<int>(x * 2))
         .Bind(x => new Monad<string>($"x={x}"));
Console.WriteLine(r.Value); // prints x=42
```

在这个例子中，`m`是一个包装整数值`21`的单子。我们使用一个返回新单子的函数进行绑定，该单子的值是初始值的两倍。我们可以再次使用一个将整数转换为字符串的函数对这个单子进行绑定。

从这个例子中，你可以看到这些绑定操作可以链接在一起。这就是流畅接口提供的功能——通过链接方法来编写类似书面散文的代码。这可以通过以下示例进一步说明——假设一个企业有客户，客户下订单，订单可以包含一个或多个商品，你需要找出一个特定企业所有客户购买的所有不同商品。

为简单起见，让我们考虑以下类：

```cs
class Business
{
    public IEnumerable<Customer> GetCustomers() { 
      return /* … */; }
}
class Customer
{
    public IEnumerable<Order> GetOrders() { return /* … */; }
}
class Order
{
    public IEnumerable<Article> GetArticles() { return /* … */; }
}
class Article { }
```

在典型的命令式风格中，你可以按照以下方式实现解决方案：

```cs
IEnumerable<Article> GetArticlesSoldBy(Business business)
{
    var articles = new HashSet<Article>();
    foreach (var customer in business.GetCustomers())
    {
        foreach (var order in customer.GetOrders())
        {
            foreach (var article in order.GetArticles())
            {
                articles.Add(article);
            }
        }
    }
    return articles;
}
```

然而，通过使用 LINQ 和`IEnumerable<T>`和“SelectMany（）”单子，这可以更简化。函数式编程风格的实现可能如下所示：

```cs
IEnumerable<Article> GetArticlesSoldBy(Business business)
{
    return business.GetCustomers()
                   .SelectMany(c => c.GetOrders())
                   .SelectMany(o => o.GetArticles())
                   .Distinct()
                   .ToList();
}
```

这使用了流畅接口模式，结果是更简洁的代码，也更容易理解。

# 总结

这一章是对 C#命令式编程特性的一次离开，因为我们探讨了内置到语言中的函数式编程概念和技术。我们研究了高阶函数、lambda 表达式、部分函数应用、柯里化、闭包、幺半群和单子。我们还介绍了 LINQ 及其两种风格：方法语法和查询语法。这些大多数主题都比本书的建议范围复杂和更高级。因此，我们建议您使用其他资源来掌握它们。

在下一章中，我们将研究.NET 中可用的反射服务以及 C#的动态编程能力。

# 测试你学到了什么

1.  函数式编程的主要特征是什么？它提供了什么优势？

1.  什么是高阶函数？

1.  是什么让函数在 C#语言中成为一等公民？

1.  什么是 lambda 表达式？写 lambda 表达式的语法是什么？

1.  lambda 表达式中变量作用域适用的规则是什么？

1.  什么是 LINQ？标准查询操作符是什么？查询语法是什么？

1.  “Select（）”和“SelectMany（）”之间有什么区别？

1.  什么是部分函数应用，它与柯里化有什么不同？

1.  什么是幺半群？

1.  什么是单子？
