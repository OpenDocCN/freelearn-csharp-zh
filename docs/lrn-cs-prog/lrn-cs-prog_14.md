# 第十四章：错误处理

从历史上看，管理运行时错误一直是一个难题，因为它们的性质复杂而不同，涵盖了从硬件故障到业务逻辑错误的各种情况。

其中一些错误，如*除以零*和*空指针解引用*，是由 CPU 本身作为异常生成的，而其他一些是在软件级别生成的，并根据运行时和编程语言作为异常或错误代码传播。

.NET 平台已经设计了通过异常策略来管理错误条件，这具有极大的优势，可以大大简化处理代码。这意味着任何属性或方法都可能抛出异常，并通过异常对象传达错误条件。

抛出异常引发了一个重要问题——*异常是库实现者和其使用者之间的契约的一部分，还是实现细节？*

在本章中，我们将开始分析语言语法，以便从生产者或消费者的角度参与异常模型。然而，我们还需要超越语法，分析开发人员寻求调试原因和与错误抛出和错误处理相关的设计问题的影响。本章的以下三个部分将涵盖这些主题：

+   错误

+   异常

+   调试和监控异常

在本章结束时，您将能够捕获现有库中的异常，了解方法是否应返回失败代码或抛出异常，并在有意义时创建自定义异常类型。

# 错误

在软件开发中，用于管理错误的两种策略是`winerror.h`文件，即使它们都是 Windows 操作系统的一部分。换句话说，错误代码不是标准的一部分，当调用穿越边界时（如不同的操作系统或运行时环境），它们需要被转换。

错误代码的另一个重要方面是它们是方法声明的一部分。例如，定义除法方法如下会感觉非常自然：

```cs
double Div(double a, double b) { ... }
```

但如果分母是`0`，我们应该向调用者传达无效参数错误。采用错误代码对方法签名有直接影响，在这种情况下将是以下内容：

```cs
int Div(double a, double b, out double result) { ... }
```

这个最后的签名（返回整数类型的错误代码）并不像任何库用户所期望的那样整洁。此外，调用代码有责任确定操作是否成功，这会引发多个问题。

第一个问题是代码检查错误代码的复杂性，就像这个例子：

```cs
var api = new SomeApi();
if (api.Begin() == 0)
{
    if (api.DoWork() == 0)
    {
        api.End();
    }
}
```

假设`0`是成功代码，每个块内部的代码都必须缩进，创建一个烦人且令人困惑的三角形，其大小与调用方法的数量一样大。即使通过逆转逻辑并检查失败条件，情况也不会改善，因为必须放置大量的`if`语句以避免讨厌的错误。

前面的代码还显示了一个常见情况，即`api.End()`方法返回一个看似无用的错误代码，因为它*结束*了调用序列，而实际上可能需要处理它。这个问题的根源在于错误代码将决定错误严重性的责任留给了调用者。异常模型的一个优点是它将这种权力交给了被调用的方法，它可以*强制执行*错误的严重性。这绝对更有意义，因为严重性很可能是特定于实现的。

前面的代码也隐藏了一个潜在的性能问题，这是由于现代 CPU 的特性，提供了一种称为**分支预测**的功能，这是 CPU 在预加载跳转后的指令时所做的一种猜测。根据许多因素，CPU 可能预加载*一条*路径，使其他路径运行得更慢，因为它们的代码没有被预取。

最后，就所有现代语言中设计的类型成员属性而言，它们与错误代码不匹配，因为没有语法允许调用者了解错误，因此使用异常是沟通问题的唯一方式。

出于所有这些原因，当.NET 运行时最初设计时，团队决定采用异常范例，将任何错误条件视为*带外信息*，而不是方法签名的一部分。

# 异常

异常是运行时提供的一种机制，可以使执行突然中断并跳转到处理错误的代码。由于处理程序可能由调用路径中的任何调用者声明，运行时负责恢复堆栈和任何其他未完成的`finally`块，我们将在本章的*finally 块*部分进行讨论。

调用代码可能希望处理异常，如果是这样，它可以决定恢复正常执行，或者只是让异常继续传递给其他处理程序（如果有的话）。每当应用程序没有提供处理代码时，运行时都会捕获错误条件，并做唯一合理的事情——终止应用程序。

这让我们回到了我们在介绍中提出的最初问题——*异常是否是库实现者与其消费者之间的契约的一部分，还是一个实现细节？*

由于实现者通过异常向其调用者传达异常情况，看起来异常是契约的一部分。至少这是其他语言实现者的结论，包括 Java 和 C++，它们都具有指定方法生成的可能异常列表的能力。无论如何，最近的 C++标准已经废弃并后来删除了声明中的异常规范，只留下了指定方法是否可能抛出异常的能力。

.NET 平台决定不将异常与方法签名绑定在一起，因为它被认为是一个实现细节。事实上，同一个接口或基类的多个实现可能使用不同的技术抛出不同的异常。例如，当你创建一个对象的代理时，你可能需要抛出不同类型的异常，除了代理对象中声明的异常。

由于异常不是签名的一部分，.NET 平台为所有可能的异常定义了一个名为`System.Exception`的基类。这种类型实际上是约束消费者（调用者）与生产者（被调用方法）之间的契约的一部分。

当然，.NET 运行时是捕获异常并负责执行匹配处理程序的主体。因此，异常只在.NET 上下文中有效，每次跨越边界时，都会有`Win32Exception`和`COMException`，它们都是从`Exception`派生而来。

显然，异常模型是管理错误的普遍良药，但仍有一个非常重要的方面需要考虑——*性能方面*。

捕获异常、展开堆栈、调用相关的`finally`块以及执行其他必要的基础设施代码的整个过程需要时间。从这个角度来看，毫无疑问，错误代码的性能要好得多，但这是我们已经提到的所有优势的回报。

当我们谈论性能时，必须进行测量，这又取决于影响性能的代码是否经常运行。换句话说，如果异常使用是*异常的*，它不会影响整体性能。例如，`System.IO.File.Exists`方法返回一个布尔值，告诉我们文件是否存在于文件系统中。但是，这不会抛出异常，因为找不到文件不是一个异常情况，在重复调用时抛出异常可能会严重影响性能。

现在让我们通过检查处理异常所需的语句来动手编写代码。当您阅读以下各节时，您会注意到我们在*第三章*中简要介绍了一些这些概念，*控制语句和异常*，当我们谈论异常处理时。在本章中，我们将更深入地涵盖这些主题。

## 捕获异常

一般来说，最好在异常被抛出之前避免错误。例如，验证来自表示层的输入参数是您最好的机会。

在尝试打开和读取文件之前，您可能希望检查其是否存在：

```cs
if (!File.Exists(filename)) return null;
var content = File.ReadAllText(filename);
```

但是，这个检查并不能保护代码免受其他可能的错误，因为文件名可能包含在 Windows 和 Linux 操作系统中都被禁止的斜杠(`/`)。尝试对文件名进行消毒是没有意义的，因为在访问文件系统时可能会发生其他错误，比如错误的路径或损坏的媒体。

每当错误发生且无法轻易预防时，代码必须受到 C#语言提供的建议的保护：`try`和`catch`块语句。

以下片段演示了如何保护`File.ReadAllText`免受任何可能的错误：

```cs
try
{
    var content = File.ReadAllText(filename);
    return content.Length;
}
catch (Exception) { /* ... */ }
return 0;
```

`try`块包围了我们想要保护的代码。因此，`File.ReadAllText`抛出的任何异常都会导致执行立即停止（`content.Length`不会被执行），并跳转到匹配的 catch 处理程序。

`catch`块必须紧随`try`块之后，并指定只有在抛出的异常与圆括号内指定的类型匹配时才必须执行的代码。

前面的示例能够在`catch`块中捕获任何错误，因为`Exception`是所有异常的*基类层次结构*。但这未必是一件好事，因为您可能希望从特定异常中恢复，同时将其他失败的责任留给调用者。

信息框

大多数与文件名相关的问题可以通过添加`File.Exists`的检查来避免，但我们故意省略了它，以便在我们的示例中有更多可能的异常选择。

前面的片段可能会因为为文件名提供不同的值而失败。例如，如果`filename`为 null，则从`File.ReadAllText`方法抛出`ArgumentNullException`。如果相反，`filename`为`/`，那么它会被解释为对根驱动器的访问，这需要管理员权限，因此异常将是`System.UnauthorizedAccessException`。当值为`//`时，会抛出`System.IO.IOException`，因为路径无效。

由于根据异常类型做出不同决策可能很有用，C#语法提供了指定多个`catch`块的能力，如下例所示：

```cs
try
{
    if (validateExistence && !File.Exists(filename)) return 0;
    var content = File.ReadAllText(filename);
    return content.Length;
}
catch (ArgumentNullException) { /* ... */ }
catch (IOException) { /* ... */ }
catch (UnauthorizedAccessException) { /* ... */ }
catch (Exception) { /* ... */ }
```

官方的.NET 类库文档包含了可以抛出异常的任何成员的*异常*部分。如果您使用 Visual Studio 并将鼠标悬停在 API 上，您将看到一个工具提示显示所有可能的异常的列表。以下截图显示了`File.ReadAllText`方法的工具提示：

![图 14.1 - 显示 File.ReadAllText 方法的异常的工具提示](img/Figure_14.1_B12346.jpg)

图 14.1 - 显示 File.ReadAllText 方法的异常的工具提示

现在让我们想象一下，`filename`指定了一个不存在的文件：在这段代码中会发生什么？根据工具提示异常列表，我们可以很容易地猜到会抛出`FileNotFoundException`异常。这个异常的类层次结构分别是`IOException`、`SystemException`，当然还有`Exception`。

有两个 catch 块满足匹配——`IOException`和`Exception`——但第一个获胜，因为`catch`块的顺序非常重要。如果尝试颠倒这些块的顺序，将会得到编译错误，并在编辑器中得到反馈，因为这将导致无法访问的`catch`块。下面的例子显示了当指定`catch(Exception)`作为第一个时，Visual Studio 编辑器生成的红色波浪线：

![图 14.2 - 当 catch(Exception)是第一个使用的异常时，编辑器会抱怨](img/Figure_14.2_B12346.jpg)

图 14.2 - 当 catch(Exception)是第一个使用的异常时，编辑器会抱怨

编译器发出的错误是`CS0160`：

```cs
error CS0160: A previous catch clause already catches all exceptions of this or of a super type ('Exception')
```

我们已经看到了如何在同一个方法中捕获异常。但异常模型的强大之处在于它能够沿着调用链向后查找最合适的处理程序。

在下面的例子中，我们有两个不同的方法，其中我们适当地处理了`ArgumentNullException`：

```cs
public string ReadTextFile(string filename)
{
    try
    {
        var content = File.ReadAllText(filename);
        return content;
    }
    catch (ArgumentNullException) { /* ... */ }
    return null;
}
public void WriteTextFile(string filename, string content)
{
    try
    {
        File.WriteAllText(filename, content);
    }
    catch (ArgumentNullException) { /* ... */ }
}
```

即使这两个方法中已经声明了`try..catch`块，但无论何时发生`IOException`，这些处理程序都不会被调用。相反，运行时会开始在调用者链中寻找兼容的处理程序。这个完全由.NET 运行时管理的过程称为**堆栈展开**，它包括从堆栈中检索的第一个兼容处理程序的调用处跳转。

在下面的例子中，`try..catch`块拦截了可能由`ReadAllText`或`WriteAllText`API 引发的`IOException`，这些 API 被`ReadTextFile`和`WriteTextFile`方法使用：

```cs
public void CopyReversedTextFile(string source, string target)
{
    try
    {
        var content = ReadTextFile(source);
        content = content.Replace("\r\n", "\r");
        WriteTextFile(target, content);
    }
    catch (IOException) { /*...*/ }
}
```

无论调用堆栈有多深，`try..catch`块都将保护这段代码免受任何`IOException`的影响。

通过前面的所有例子，我们已经学会了如何区分异常类型，但`catch`块接收到该类型的对象，提供了关于异常性质的上下文信息。现在让我们来看看异常对象。

## 异常对象

除了异常类型之外，`catch`块语法还可以指定变量的名称，引用被捕获的异常。下面的例子展示了一个计算所有指定文件的内容字符串长度的方法：

```cs
int[] GetFileLengths(params string[] filenames)
{
    try
    {
        var sizes = new int[filenames.Length];
        int i = 0;
        foreach(var filename in filenames)
        {
            var content = File.ReadAllText(filename);
            sizes[i++] = content.Length;  // may differ from file size
        }
        return sizes;
    }
    catch (FileNotFoundException err)
    {
        Debug.WriteLine($"Cannot find {err.FileName}");
        return null;
    }
}
```

每当我们打开一个文件而没有先使用`File.Exists`来避免异常时，我们可能会收到`FileNotFoundException`。这个对象是`IOException`的一个特例，并暴露了一个`Filename`属性，提供了找不到的文件名。我甚至记不清有多少次我希望从有故障的应用程序中获得这样的反馈！

信息框

我们将在*调试和监控*部分更详细地了解基本异常成员，但您现在可以开始调查基类库中抛出的众多异常所暴露的属性。

下面的代码展示了另一个有趣的例子，同时捕获`ArgumentException`——当参数未通过使用它的方法的验证时发生的异常：

```cs
private void CopyFrom(string source)
{
    try
    {
        var target = CreateFilename(source);
        File.Copy(source, target);
    }
    catch (ArgumentException err)
    {
        Debug.WriteLine($"The parameter {err.ParamName} is invalid");
        return;
    }
}
```

`catch`块拦截了`source`和`target`参数的故障。与`source`参数验证相关的任何错误都应该反弹到调用者，而`target`参数在本地计算。

我们如何只捕获我们感兴趣的异常？答案在于 C# 6 中引入的一种语言特性。

## 条件捕获

`catch`块可以选择性地指定一个`when`子句来限制处理程序的范围。

下面的示例与前一个示例非常相似，但将`catch`块限制为只钩住`ArgumentException`，其`ParamName`为`"destFileName"`，这是`File.Copy`方法的第二个参数的名称：

```cs
private void CopyFrom(string source)
{
    try
    {
        var target = CreateFilename(source);
        File.Copy(source, target);
    }
    catch (ArgumentException err) when (err.ParamName == "destFileName")
    {
        Debug.WriteLine($"The parameter {err.ParamName} is invalid");
        return;
    }
}
```

`when`子句接受任何有效的布尔表达式，不一定要使用`catch`块中指定的异常对象。

请注意，在此示例中，我们使用了`"destFileName"`字符串来指定`File.Copy`的第二个参数。如果您使用 Visual Studio，可以将光标放在所需的参数上，然后使用快捷键*Ctrl* + *Shift* + *空格键*来查看参数名称，会显示以下建议窗口：

![图 14.3 - 编辑器显示的建议窗口](img/Figure_14.3_B12346.jpg)

图 14.3 - 编辑器显示的建议窗口

现在是时候转到生产者方面，看看我们如何抛出异常。

## 抛出异常

当我们使用已经提供所需参数验证的 API 时，您可以决定不验证参数，并最终抛出异常。在下面的示例中，我们打开一个日志文件，给定其名称由`logName`指定：

```cs
private string ReadLog(string logName)
{
    return File.ReadAllText(logName);
}
```

验证`logName`是否为 null 或空字符串的决定并没有提供任何价值，因为被调用的方法已经提供了考虑更多情况的验证，比如无效路径或不存在的文件。

但`logName`参数可能表达不同的语义，指定日志的名称而不是要写入磁盘的文件名（如果有的话）。协调两种可能含义的解决方案是，如果尚未存在，则添加`".log"`扩展名：

```cs
private string ReadLog(string logName)
{
    var filename = "App-" + (logName.EndsWith(".log") ? logName : logName + ".log");
    return File.ReadAllText(filename);
}
```

这更有意义，但`logName`可能为`null`，导致在突出显示的代码上引发`NullReferenceException`异常，这将使故障排除变得更加困难。

为了解决这个问题，我们可以添加`null`参数验证：

```cs
private string ReadLog(string logName)
{
    if(logName == null) throw new ArgumentNullException(nameof(logName));
    var filename = "App-" + (logName.EndsWith(".log") ? logName : logName + ".log");
    return File.ReadAllText(filename);
}
```

`throw`语句接受任何继承自异常的对象，并立即中断方法的执行。运行时会钩住异常并将其分派到适当的处理程序，正如我们在前面的章节中已经调查过的那样。

提示

请注意使用`nameof(logName)`来指定有问题的参数的名称。我们在前一节中使用了这个参数来捕获`File.Copy`方法中的异常。确保永远不要将参数名称指定为文字。使用`nameof()`可以保证名称始终有效，并避免重构时出现问题。

`throw`语句非常简单，但请记住只在异常情况下使用它；否则，您可能会遇到性能问题。在下面的示例中，我们使用流行的`Benchmark.NET`微基准库比较了两个循环。`LoopNop`方法中的一个执行永远不会抛出异常的代码，而`LoopEx`中的另一个在每次迭代时都会抛出异常：

```cs
public int Loop { get; } = 1000;
[Benchmark]
public void LoopNop()
{
    for (var i = 0; i < Loop; i++)
    {
        try { Nop(i); }
        catch (Exception) { }
    }
}
[MethodImpl(MethodImplOptions.NoOptimization | MethodImplOptions.NoInlining)]
private void Nop(int i) { }
```

`LoopNop`方法只是循环执行`Nop`空方法 1,000 次。`Nop`方法被标记为`NoInlining`，以避免编译器优化删除调用。

第二种方法执行相同的循环 1,000 次，但调用`Crash`方法，该方法在每次迭代时都会抛出异常：

```cs
[Benchmark]
public void LoopEx()
{
    for (var i = 0; i < Loop; i++)
    {
        try { Crash(i); }
        catch (Exception) { }
    }
}
[MethodImpl(MethodImplOptions.NoOptimization | MethodImplOptions.NoInlining)]
private void Crash(int i) => 
    throw new InvalidOperationException();
```

`Crash`方法每次都创建一个新的异常对象，这是异常对象的实际用法。但即使每次重复使用相同的对象，异常模型的性能损失也是巨大的。

基准测试的结果是了解影响异常使用的数量级的想法，我们的示例中是四个数量级。

以下输出显示了基准测试的结果：

```cs
|        Method |          Mean |       Error | Allocated |
|-------------- |--------------:|------------:|----------:|
|       LoopNop |      2.284 us |   0.0444 us |         - |
|        LoopEx | 25,365.467 us | 486.2660 us |  320000 B |
```

这个基准测试只是证明了抛出异常必须只用于异常情况，并不应该对异常模型的有效性产生任何疑问。

我们已经看到了基类库中提供的一些异常类型。现在，我们将看一下最常见的异常以及何时使用它们。

### 常见的异常类型

基类库中提供的异常表达了最流行的故障类别的语义。在基类库中提供的所有异常中，值得一提的是开发人员最常使用的异常。在本章中，我们已经看到其他流行的异常，比如`NullReferenceException`，但它们通常只会由运行时抛出：

+   `ArgumentNullException`：通常在方法开头验证方法参数时使用。由于引用类型可能假定空值，因此用于通知调用者空值不是方法的可接受值。

+   `ArgumentException`：这是另一个在方法开头使用的异常。它的含义更广泛，当参数值无效时抛出。

+   `InvalidOperationException`：通常用于拒绝方法调用，每当对象的状态对于所请求的操作无效时。

+   `FormatException`：类库使用它来表示格式错误的字符串。它也可以用于解析文本以进行任何其他目的的用户代码。

+   `IndexOutOfRangeException`：每当参数指向容器的预期范围之外时使用，比如数组或集合。

+   `NotImplementedException`：用于通知调用者所调用的方法没有可用的实现。例如，当您要求 Visual Studio 在类主体内实现一个接口时，代码生成器会生成抛出此异常的属性和方法。

+   `TypeLoadException`：您可能很少需要抛出此异常。它通常发生在无法将类型加载到内存中时。每当在静态构造函数中发生异常时，通常会发生，并且除非您记得这个说明，否则可能很难诊断。

基类库的所有异常的详尽列表可以在`Exception`类文档中找到（[`docs.microsoft.com/en-us/dotnet/api/system.exception?view=netcore-3.1`](https://docs.microsoft.com/en-us/dotnet/api/system.exception?view=netcore-3.1)）。

在决定抛出异常时，非常重要的是使用完全表达错误语义的异常。每当在.NET 中找不到合适的类时，最好定义一个自定义异常类型。

## 创建自定义异常类型

定义异常类型就像编写一个简单的类一样简单；唯一的要求是继承自`Exception`等异常类型。

以下代码声明了一个用于表示应用程序数据层中的失败的自定义异常：

```cs
public class DataLayerException : Exception
{
    public DataLayerException(string queryKeyword = null)
        : base()
    {
        this.QueryKeyword = queryKeyword;
    }
    public DataLayerException(string message, string queryKeyword = null)
        : base(message)
    {
        this.QueryKeyword = queryKeyword;
    }
    public DataLayerException(string message, Exception innerException, string queryKeyword = null)
        : base(message, innerException)
    {
        this.QueryKeyword = queryKeyword;
    }
    public string QueryKeyword { get; private set; }
}
```

前面的自定义异常类定义了三个构造函数，因为它们旨在在开发人员构造它们时提供一致的体验：

+   默认构造函数可能存在，每当您不需要使用额外参数构建异常时。在我们的情况下，默认情况下允许使用空的`QueryKeyword`构建异常对象。

+   接受`message`参数的构造函数在表达可能简化诊断的任何人类信息时非常重要。消息应该只提供诊断信息，永远不应该显示给最终用户。

+   接受内部异常的构造函数在提供有关导致当前错误情况的任何底层异常的额外信息方面非常有价值。

一旦定义了新的自定义异常，它就可以与`throw`语句一起使用。在下面的示例中，我们看到一些假设的代码向存储库发出查询并将底层错误条件转换为我们的自定义异常：

```cs
public IList<string> GetCustomerNames(string queryKeyword)
{
    var repository = new Repository();
    try
    {
        return repository.GetCustomerNames(queryKeyword);
    }
    catch (Exception err)
    {
        throw new DataLayerException($"Error on repository {repository.RepositoryName}", err, queryKeyword);
    }
}
```

被捕获的异常作为参数传递给构造函数，以保留错误的原始原因，同时抛出更好地表示错误性质的自定义异常。

在`catch`块内部抛出异常揭示了关于错误语义的架构问题。在前面的例子中，我们无法恢复错误，但仍然希望捕获它，因为我们的应用程序的安装可能会导致被查询的存储库非常不同。例如，如果存储库是数据库，内部异常将与*SQL Server*相关，而如果是文件系统，它将是`IOException`。

如果我们希望应用程序的更高级别能够适当地处理错误并有机会恢复错误，我们需要抽象底层错误并提供业务逻辑异常，例如我们定义的`DataLayerException`。

信息框

.NET Framework 最初将`ApplicationException`定义为所有自定义异常的基类。由于没有强制执行，基类库本身从未广泛采用这种最佳实践。因此，当前的最佳实践是从`Exception`派生所有自定义异常，正如您可以在官方文档中阅读的那样：

[`docs.microsoft.com/en-us/dotnet/api/system.applicationexception?view=netcore-3.1`](https://docs.microsoft.com/en-us/dotnet/api/system.applicationexception?view=netcore-3.1)

在`catch`块内部抛出异常的能力并不局限于自定义异常。

## 重新抛出异常

我们刚刚看到了如何在`catch`块内部抛出一个新的异常，但是有一个重要的快捷方式可以重新抛出相同的异常。

`catch`块通常用于尝试恢复错误或仅仅记录它。在这两种情况下，我们可能希望让异常继续，就好像根本没有被捕获一样。C#语言为这种情况提供了`throw`语句的简单用法，如下例所示：

```cs
public string ReadAllText(string filename)
{
    try
    {
        return File.ReadAllText(filename);
    }
    catch (Exception err)
    {
        Log(err.ToString());
        throw;
    }
}
```

`throw`语句后面没有任何参数，但它相当于在`catch`块中指定相同的异常：

```cs
    catch (Exception err)
    {
        Log(err.ToString());
        throw err;
    }
}
```

除非`err`引用被更改以指向不同的对象，否则这两个语句是等价的，并且具有保留导致错误的原始堆栈的重大优势。无论如何，我们仍然能够向异常对象添加更多信息（`HelpLink`属性是一个典型的例子）。

如果我们抛出一个不同的异常对象，原始堆栈不是被抛出的异常的一部分，这就是为什么`innerException`存在的原因。

在某些情况下，您可能希望保存`catch`块捕获的异常，并稍后重新抛出它。通过简单地抛出捕获的异常，捕获的堆栈将会不同且不太有用。如果您需要保留最初捕获异常的堆栈，可以使用`ExceptionDispatchInfo`类，该类提供了两个简单的方法。`Capture`静态方法接受一个异常并返回一个包含`Capture`调用时刻的所有堆栈信息的`ExceptionDispatchInfo`实例。您可以保存这个对象，然后使用它的`Throw`方法抛出异常以及原始堆栈信息。这种模式在以下示例中显示：

```cs
public void Foo()
{
    ExceptionDispatchInfo exceptionDispatchInfo = null;
    try
    {
        ExecuteFunctionThatThrows();
    }
    catch(Exception ex)
    {                 
        exceptionDispatchInfo = ExceptionDispatchInfo.Capture(ex);
    }

    // do something you cannot do in the catch block

    // rethrow
    if (exceptionDispatchInfo != null)
        exceptionDispatchInfo.Throw();
}
```

在这里，我们调用一个抛出异常的方法，然后在`catch`子句中捕获它。我们使用静态的`ExceptionDispatchInfo.Capture`方法存储对这个异常的引用，这有助于保留调用堆栈。在方法的最后，我们使用`ExceptionDispatchInfo`的`Throw`方法重新抛出异常。

## 最后的块

`finally`块是与*异常管理*相关的最后一个 C#语句。它非常重要，因为它允许表达在`try`块之后必须被调用的代码部分，无论是否发生异常。

在前面的章节中，我们已经看到了代码执行的行为，具体取决于是否发生异常。`try`块中的代码执行可能会被未决的异常中断，跳过该代码的部分。一旦发生错误，我们保证将执行匹配的`catch`块，使其有机会将问题写入日志，可能执行一些恢复逻辑。

`finally`块甚至可以在没有任何`catch`块的情况下指定，这意味着任何异常都将反弹到调用链，但是`finally`块中指定的代码将在`try`块之后的任何情况下执行。

以下示例显示了三个方法，它们的调用是*嵌套*的。第一个方法`M1`调用`M2`，`M2`调用`M3`，`M3`调用`Crash`，最终抛出异常，如下面的代码所示：

```cs
private void M1()
{
    try { M2(); }
    catch (Exception) { Debug.WriteLine("catch in M1"); }
    finally { Debug.WriteLine("finally in M1"); }
}
private void M2()
{
    try { M3(); }
    catch (Exception) { Debug.WriteLine("catch in M2"); }
    finally { Debug.WriteLine("finally in M2"); }
}
private void M3()
{
    try { Crash(); }
    finally { Debug.WriteLine("finally in M3"); }
}
private void Crash() => throw new Exception("Boom");
```

当我们调用`M1`并且调用链到达`Crash`时，在`M3`中没有`catch`块来处理异常，但是在离开方法之前它的`finally`块被调用。此时，运行时会反弹到`M2`的调用者，它捕获了异常，但也调用了它的`finally`代码。最后，由于异常已经被处理，`M2`自然地将控制返回给`M1`，并且它的`finally`代码也被执行，如下面的输出所示：

```cs
finally in M3
catch in M2
finally in M2
finally in M1
```

如果愿意，您可以通过向`try`块添加额外详细的日志记录来重复此实验，但这里的重点是`finally`块总是在离开方法之前*始终执行*。

`try..finally`组合的另一个常见用途是确保资源已被正确释放，C#已经将这种模式作为关键字，即`using`语句。以下示例显示了两个等效的代码片段。C#编译器生成的 IL 代码基本相同，您可以使用`ILSpy`工具自行测试 IL 语言的反编译结果：

```cs
void FinallyBlock()
{
    Resource res = new Resource();
    try
    {
        Console.WriteLine();
    }
    finally
    {
        res?.Dispose();
    }
}
void UsingStatement()
{
    using(var res = new Resource())
    {
        Console.WriteLine();
    }
}
```

当然，`using`语句将其使用限制在实现`IDisposable`接口的对象上，但它生成相同的模式。这是我们在*第九章*中深入研究的一个主题，*资源管理*。

现在我们已经从消费者和生产者的角度看到了异常的所有方面，我们将讨论与异常相关的问题的诊断调查。

# 调试和监视异常

调试异常与调试普通代码有些不同，因为自然流程被运行时中断和处理。除非在处理异常的代码上设置断点，否则有可能不理解问题从何处开始。当捕获异常并且没有重新抛出或者方法在`catch`块内没有重新抛出时，就会发生这种情况。

这可能看起来是异常模型的一个重要缺点，但.NET 运行时提供了克服这个问题的所有必要支持。事实上，运行时内置了对调试器的支持，为愿意拦截异常的调试器提供了有价值的钩子。

从调试器的角度来看，您有两种可能性，或者*机会*，来拦截任何被抛出的异常：

+   **First-chance** **exceptions**代表异常在非常早期阶段的状态，即它们被抛出并在跳转到其处理程序之前。拦截异常（在第一次出现时）的优势在于我们可以精确地确定是哪段代码导致了异常。相反，被拦截的异常可能是合法的并且被正确处理。换句话说，调试器将停止任何发生的异常，即使那些没有引起麻烦的异常。默认情况下，调试器在第一次出现异常时不会停止，但会在调试器输出窗口中打印跟踪。

+   **第二次机会**或**未处理**的**异常**是致命的。这意味着.NET 运行时没有找到任何合适的处理程序来管理它们，并在强制关闭崩溃的应用程序之前调用调试器。当发生第二次机会异常时，调试器总是会停止，这总是代表一个错误条件。第二次机会异常会在输出窗口中打印，并在异常对话框中呈现为未处理的异常。

使用默认设置，Visual Studio 调试器将中断，显示在崩溃应用程序之前可能运行的最后一行代码。这段代码不一定是导致应用程序崩溃的原因；因此，您可能需要修改这些设置以更好地理解原因。

## 调试第二次机会异常

当抛出的异常在我们的源代码中可用时，调试器的默认设置足以理解问题的原因，如下例所示：

```cs
public void TestMethod1() => Crash1();
private void Crash1() => throw new Exception("This will make the app crash");
```

Visual Studio 调试器将在突出显示的代码处停止，显示臭名昭著的异常对话框：

![图 14.4 - 显示异常类型、消息和获取更多信息的链接的对话框](img/Figure_14.4_B12346.jpg)

图 14.4 - 显示异常类型、消息和获取更多信息的链接的对话框

输出窗口还提供了其他信息：

```cs
Exception thrown: 'System.Exception' in ExceptionDiagnostics.dll
An unhandled exception of type 'System.Exception' occurred in ExceptionDiagnostics.dll
This will make the app crash
```

Visual Studio 调试器不断改进诊断输出版本。在许多情况下，它能够打印出完全代表问题起源的消息。在以下示例代码中，异常是由`null`引用引起的：

```cs
public void TestMethod2() => Crash2(null);
private void Crash2(string str) => Console.WriteLine(str.Length);
```

对话框显示了一个**str was null**消息，告诉我们发生了什么：

![图 14.5 - 在变量为 null 之前看到的异常对话框显示细节](img/Figure_14.5_B12346.jpg)

图 14.5 - 在变量为 null 之前看到的异常对话框显示细节

同样，输出窗口显示了类似的消息：

```cs
**str** was null.
```

现在我们已经看到了调试器的默认行为，让我们考虑一个稍微复杂一点的场景。

## 调试首次机会异常

在本章中，我们强调了尝试从异常中恢复或重新抛出不同异常的价值，以便为调用代码提供更好的语义。在以下代码中，这增加了一些调试的困难：

```cs
public void TestMethod3()
{
    try
    {
        Crash1();
    }
    catch (Exception) { }
}
```

由于`catch`块没有重新抛出，异常被简单地吞噬，因此调试器根本不会中断。但这种情况可能会揭示问题的真正原因。*我们如何要求调试器在这个异常处停止？*

答案在 Visual Studio（或其他暴露相同功能的调试器）的异常窗口中。从**调试** | **窗口** | **异常设置**菜单，Visual Studio 将显示以下窗口：

![图 14.6 - 异常设置窗口](img/Figure_14.6_B12346.jpg)

图 14.6 - 异常设置窗口

.NET 运行时的相关异常是**公共语言运行时异常**项下的异常：

![图 14.7 - 显示可选择异常的异常设置窗口的一部分](img/Figure_14.7_B12346.jpg)

图 14.7 - 显示可选择异常的异常设置窗口的一部分

其中大多数异常都未选中，这意味着，正如我们已经说过的，调试器*不会*在首次机会异常处中断，除非选中该复选框。

例如，如果我们想在上一个示例的`throw`语句处中断，我们只需从列表中选择`System.Exception`。

提示

请注意，此列表中的每个异常只包括确切的类型，而不包括派生类型的层次结构。换句话说，`System.Exception`不会挂钩整个层次结构。

通过浏览列表，您可能会注意到`System.NullReferenceException`和其他异常默认已被选中，因为这些异常通常被认为是应该通过在代码中验证参数来避免的错误。

由于异常列表非常长，**Common Language Runtime Exceptions**根项目是一个三状态切换器，可以选择所有项目、无项目或重置为默认设置。

## AppDomain 异常事件

第一次和第二次机会异常也可以通过`AppDomain`对象提供的两个事件进行监视，但不能拦截。您可以通过在应用程序中使用以下代码订阅这些事件：

```cs
AppDomain.CurrentDomain.FirstChanceException += CurrentDomain_FirstChanceException;
AppDomain.CurrentDomain.UnhandledException += CurrentDomain_UnhandledException;
// ...
private static void CurrentDomain_FirstChanceException(object sender,     System.Runtime.ExceptionServices.FirstChanceExceptionEventArgs e)
{
    Console.WriteLine($"First-Chance. {e.Exception.Message}");
}
private static void CurrentDomain_UnhandledException(object sender, UnhandledExceptionEventArgs e)
{
    var ex = (Exception)e.ExceptionObject;
    Console.WriteLine($"Unhandled Exception. IsTerminating: {e.IsTerminating} - {ex.Message}");
}
```

大多数情况下，您不会希望监视第一次机会异常，因为它们可能不会对应用程序造成任何麻烦。无论如何，当您认为它们可能由于合理处理的异常而导致性能问题时，摆脱它们可能是有用的。

第二次机会（未处理）异常对于提供任何无法捕获或意外的异常的日志非常有用。此外，在桌面应用程序环境中，典型的用例是显示自定义崩溃对话框。

提示

请注意，.NET Core 始终只有一个应用程序域，而.NET Framework 可能有多个，这在**Internet Information** **Services**（**IIS**）在重新启动主机进程时通常是真实的，特别是在 ASP.NET 应用程序中。

我们已经看到了在调试会话期间如何获取关于异常的详细信息以及记录它们的最佳选项。现在我们将看到异常对象中提供的调试信息的类型，这些信息可以在应用程序崩溃后使用。

## 记录异常

创建异常对象后，运行时会丰富其状态，以提供最详细的诊断信息，以便识别故障。无论您如何访问异常对象，无论是从`catch`块还是`AppDomain`事件，都有额外的信息可以访问。

我们已经讨论了`InnerException`属性，它递归地提供了对链中所有内部异常的访问。以下示例显示了如何迭代整个链：

```cs
private static void Dump(Exception err)
{
    var current = err;
    while (current != null)
    {
        Console.WriteLine(current.InnerException?.Message);
        current = current.InnerException;
    }
}
```

创建转储时访问内部异常并不是真正需要的，因为异常对象的`ToString`方法即使非常冗长也提供了整个链的转储。

`ToString`方法打印了运行时提供的`StackTrace`字符串属性，以捕获异常发生的整个方法链。

由于`StackTrace`是从运行时组装的字符串，异常对象还提供了`TargetSite`属性，它是`MethodBase`类型的反射对象，表示出错的方法。该对象公开了`Name`属性和方法名。

最后，`GetBaseException`方法返回最初生成故障的第一个异常，前提是任何重新抛出语句都保留了内部异常或未指定参数，正如我们已经在*重新抛出异常*部分讨论过的那样。如果您需要知道是否有异常被某个处理程序吞没，您将需要挂钩第一次机会异常事件。

还有更高级的调试技术，您可能希望使用“进一步阅读”部分提供的链接进行调查。它们包括创建转储文件，这是一个包含应用程序进程在崩溃时刻的内存的二进制文件。转储文件可以在以后使用调试工具进行调查。另一个强大且非常高级的工具是`dotnet-dump analyze` .NET Core 工具。

这些都是低级工具，通常用于所谓的`dotnet-dump`，它提供了.NET 特定的信息，除了本机调试器提供的标准元素。

例如，使用这些工具，您可以获取有关每个线程的当前堆栈状态、最近的异常数据、内存中每个对象是如何引用或被其他对象引用的、每个对象使用的内存以及与应用程序元数据和 .NET 运行时相关的其他信息。

# 摘要

在本章中，我们首先了解了为什么 .NET 采用了异常模型，而不是许多其他技术所使用的错误代码。

异常模型已经证明它非常强大，提供了一种高效而干净的方式来向调用链报告错误。它避免了用额外的参数和错误检查条件来污染代码，这可能在某些情况下导致效率损失。我们还通过基准测试验证了异常模型只能用于异常情况，否则可能严重影响应用程序的性能。

我们还详细看了 `try`、`catch` 和 `finally` 语句的语法，这些语句允许我们拦截和处理异常，并对任何未决资源进行确定性处理。

最后，我们还研究了诊断和日志记录选项，这些选项在提供所有必要信息以修复错误方面非常有用。

在下一章中，我们将学习 C# 8 的新功能，这些功能通过在性能和健壮性方面提供更多的表达能力和功能来增强语言。

# 测试你所学到的知识

1.  哪个 `block` 语句可以用来包围可能引发异常的一些代码？

1.  在任何 `catch` 块内的典型任务是什么？

1.  在指定多个 `catch` 块时，应该遵守什么顺序，为什么？

1.  在 `catch` 语句中应该指定异常变量名吗？为什么？

1.  你在 `catch` 块中捕获了一个异常。为什么你要重新抛出它？

1.  `finally` 块的作用是什么？

1.  你可以指定一个没有 `catch` 块的 `finally` 块吗？

1.  什么是第一次机会异常？

1.  如何将 Visual Studio 调试器中的第一次机会异常中断？

1.  何时需要挂钩 AppDomain 的 `UnhandledException` 事件？

# 进一步阅读

+   **dotnet-dump** **工具**（**仅适用于** **.NET** **Core**）：[`docs.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-dump`](https://docs.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-dump)

+   **WinDbg** **调试器**：[`docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools)

+   **使用** **WinDbg** **中的** **SOS** **调试** **扩展**（**仅适用于** **.NET** **Framework**）：[`docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugging-managed-code`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugging-managed-code)
