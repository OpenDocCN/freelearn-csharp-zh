

# 使用异常处理捕获异常

在构建网络应用程序时，我们总是尽力使我们的代码尽可能稳定，但有时我们无法捕获所有内容。这就是为什么异常被认为是开发的基础部分。异常处理对于防止网络应用程序崩溃并在页面上显示丑陋的错误消息至关重要。用 `try`/`catch` 或 `try`/`finally` 语句包裹一切是很诱人的。但这应该避免。在应用程序中使用 `try`/`catch`/`finally` 语句应该是例外。

本章中的常见编码标准旨在消除这些类型的场景，并提供更好的开发者体验。

在本章中，我们将探讨异常处理对开发者的意义，何时使用它，以及在哪里处理全局异常，并检查性能考虑。一旦我们理解了异常处理的基础知识，最后一节将涵盖一些常见的异常处理实践，例如将“预防胜于异常”原则应用于代码，使用日志记录，了解异常处理与单元测试的相似之处，以及为什么应该避免空的 `catch` 块。

最后，我们将学习如何使用 .NET 的新异常过滤器，结合模式匹配，何时使用 `finally` 块，以及为什么重新抛出异常是个好主意。

在本章中，我们将涵盖以下主要主题：

+   使用异常处理

+   处理全局异常

+   性能考虑

+   常见的异常处理技术

完成本章后，您将更好地理解异常处理，何时以及如何使用它，如何实现全局异常处理，以及在使用异常处理时如何知道性能是否成为问题。

# 技术要求

我们建议使用您喜欢的编辑器，在本章中添加异常处理代码片段。我们的建议如下：

+   Visual Studio（最好是最新版本）

+   Visual Studio Code

+   JetBrains Rider

我们将要使用的编辑器是 Visual Studio 2022 Enterprise，但任何版本（社区版或专业版）都可以与代码一起使用。

# 使用异常处理

在本节中，我们将讨论什么是异常处理，两种错误处理类型，何时在应用程序中使用错误处理，以及异常如何影响性能。

## 什么是异常处理？

异常处理是在代码运行时从意外情况中优雅恢复的能力；我们如何处理在应用程序中遇到的问题或错误？它还涉及在出现问题时清理分配的资源，以避免内存泄漏。

有两种类型的错误：

+   **运行时错误**：这些是我们运行应用程序时遇到的意外错误。

+   在方法开头使用 `ArgumentNullException.ThrowIfNull()` 来确认参数是否为 null。

由于本书主要面向中级到高级的开发者，我们假设调试 ASP.NET 应用程序是一个常见的流程；我们都知道调试和异常处理是相辅相成的。当开发者用`try`/`catch`块包裹代码时，他们应该对这段代码的功能有一个大致的了解。创建有用的异常的能力非常重要。异常应该是清晰和简单的。

例如，在我职业生涯的早期，一位开发者遇到了一条错误信息，告诉他们他们的磁盘空间即将耗尽。其他开发者也在应用程序中遇到了相同的错误，并疯狂地试图找出问题。问题最终被证明是由一位开发者创建的一个糟糕的错误信息，以及一个磁盘空间耗尽的*服务器*，而不是个别开发者的机器。这可以通过创建更好的日志消息或检测是否有可用磁盘空间来避免。虽然我们可以在异常处理程序中编写更好的错误信息，但我们只能保护代码到一定程度。

说真的——在编码时创建简单明了的错误信息可能是一个挑战，但从长远来看，这确实会带来影响。我们将在下一节中介绍一些常见的异常处理技术。

## 何时使用异常处理

识别代码是否需要异常处理的能力可能很棘手。除了是否需要`try`/`catch`/`finally`块之外，是否涉及我们需要清理的资源？

在异常处理方面，上下文很重要。根据我的经验，在添加异常处理之前，我总是要问自己三个问题：

1.  使用`TryParse`而不是完整的`try`/`catch`/`finally`块，或者由于无效参数而手动抛出错误。

1.  **外部资源是否会抛出异常？** 例如，Web API、存储问题、文件丢失等。

1.  **如果发生错误，我必须自己清理吗？** 例如，丢失文件连接、加载位图或数据库连接。

开发者只有在遇到他们无法处理且被认为是超出他们控制范围的代码行时才应使用异常处理，类似于磁盘驱动器空间不足的可能性。

在本节中，我们回顾了异常处理是什么以及何时正确使用它。

# 处理全局异常

如本章前面所述，当我们谈到 Web 应用程序时，我们只能处理这么多错误。但如果我们想为所有未处理的异常提供一个通用的解决方案呢？

对于全局异常，我们需要重新审视中间件。在`Startup.cs`文件中有一个名为`UseExceptionHandler()`的方法，它指向一个`/Error`页面（无论是 Razor 还是 MVC），如下面的代码片段所示：

```cs
if (env.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}
```

特别注意 `env.IsDevelopment()` 条件。*错误页*仅用于非开发环境查看。正如我们在*第四章*中提到的关于安全性的内容，始终要小心在这个页面上显示什么。它可能会暴露系统数据，例如包含凭证或其他敏感数据的数据库连接字符串。

要通过错误页访问异常，我们需要通过 `HttpContext.Features` 获取 `IExceptionHandlerPathFeature` 实例。这可以在以下 `/Error` 页面的 `OnGet()` 方法中看到：

```cs
public void OnGet()
{
    RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier;
    var exceptionFeature =
        HttpContext.Features.Get<IExceptionHandlerPathFeature>();
    // Access the Exception through exceptionFeature?.Error
    // Access the Path through exceptionFeature?.Path
    if (exceptionFeature?.Path == "/")
    {
        ErrorMessage ??= string.Empty;
        ErrorMessage += " We have bigger problems if the main page is             bombing.";
        _logger.Log(LogLevel.Error, exceptionFeature?.Error,             ErrorMessage);
    }
}
```

`HttpContext.Features` 给我们提供了访问错误的权限。从那里，我们需要确定要在页面上显示什么。在这种情况下，我们可以看到主页面包含了错误。一旦我们确定了问题，我们可以创建一个公共消息，记录错误，并将其存储在 `ErrorMessage` 中，以便主页面可以显示它。

虽然这是一个简单的例子，但我们可以使用我们的中间件来捕获全局错误。我们不需要传递页面位置，可以在中间件中使用 Lambda 来处理异常，如下面的代码片段所示：

```cs
app.UseExceptionHandler(handler =>
{
    var logger = loggerFactory.CreateLogger("Middleware");
    handler.Run(async context =>
    {
        context.Response.StatusCode = StatusCodes.            Status501NotImplemented;
        context.Response.ContentType = MediaTypeNames.Text.Plain;
        await context.Response.WriteAsync("Uh-oh...an exception was             thrown.");
        var exceptionFeature =
            context.Features.Get<IExceptionHandlerPathFeature>();
        if (exceptionFeature?.Path == "/")
        {
            var message = " Yep, the home page isn't implemented                 yet.";
            await context.Response.WriteAsync(message);
            logger.Log(LogLevel.Error, exceptionFeature.Error,                 $@"Error:{message}");
        }
    });
});
```

我们以与在 `/Error` 页面上相同的方式检索 `ExceptionHandlerPathFeature` 实例。当然，我们总是想记录错误，以便知道要修复什么（“又是那个讨厌的首页”）。

在本节中，我们学习了如何使用中间件创建全局异常处理器。这使我们能够集中处理错误，并避免在应用程序中设置过多的异常处理器。接下来，我们将关注编写异常处理器时的性能考虑。

# 性能考虑

关于异常处理的一个常见误解是它不会影响性能。如果异常严重损害了应用程序的性能，那么这是一个迹象，表明异常被过度使用了。异常绝对不应该控制应用程序的流程。

理想情况下，代码应该流畅无中断。在小型具有少量用户的网络应用程序中，这种方法可能是足够的，但对于高性能、高流量的网站，在频繁调用的代码中放置 `try`/`catch` 块可能会使网站的性能受到影响。

在本节中，我们介绍了异常处理的概念，回顾了应用程序中的两种错误类型，确定了异常处理理想的位置，并讨论了有关异常的一些性能误解。接下来，我们将探讨一些常见的异常处理技术。

# 常见的异常处理技术

异常在 .NET 中代价高昂。当应用程序中发生异常时，会有一套资源来开始错误处理过程，例如堆栈跟踪过程。即使我们在捕获和处理错误时，ASP.NET 仍在创建 `Exception` 对象及其所有相关内容，并沿着调用堆栈向上查找处理器。

在本节中，我们将探讨行业中的常见方法，通过“预防异常”来最小化异常，为什么使用日志记录，为什么单元测试与异常处理相似，为什么应该避免空的`catch`块，如何使用异常过滤和模式匹配简化异常，为什么在释放资源时块很重要，以及如何正确地重新抛出异常。

## 在异常之前进行预防

正如我们在上一节中说的，当遇到异常时，异常会中断应用程序的流程，并可能造成比预期更多的问题，例如释放之前分配的资源，并在调用栈中触发多个异常。如果我们正在编写`try`/`catch`块来控制程序的流程，我们就是在做错事。最好在用`try`/`catch`块包装代码之前进行检查。

如果从这个章节中只能学到一点，那就是这个原则：预防异常。

预防异常的概念是指尝试使用更少破坏性的方法来防止错误发生，例如完全停止网站的执行。

例如，检查以下代码：

```cs
var number = "2";
int result;
try
{
    result = int.Parse(number);
}
catch
{
    result = 0;
}
// use result
```

在一个愉快的路径中，数字将被解析，结果将是`2`。然而，如果字符串包含值为“hi”，结果将变为零。

更好的方法是使用最新的`TryParse`与`var`结合，如下所示：

```cs
var number = "2";
if (!int.TryParse(number, out var result))
{
    result = 0;
}
// use result
```

采用这种方法，我们试图将数字转换并存储转换后的值到一个名为`result`的新变量中。`TryParse`在转换成功时返回`true`，不成功时返回`false`。如果转换失败，我们将`result`设置为`0`。

我们通过更简单的方式处理转换来预防异常。在.NET 中有大量的转换方法，我们可以不使用`try`/`catch`/`finally`块来完成相同的事情。

对于简单的异常，查看`try`/`catch`块，并询问我们是否可以在创建异常之前应用某种预防措施。

## 使用日志记录

在创建`try`/`catch`块时，最好展示我们想要从代码中获得的目的。错误是我们能够恢复的还是需要调查的？

如果是后者，在抛出错误之前最好向记录器发送一个构造良好的错误消息。正如在各个章节中提到的，最好使用日志策略来识别代码库中的问题。

如果我们使用之前的例子来确定导致错误的值，我们可以创建一个日志条目，如下所示：

```cs
var number = "hi";
int result;
try
{
    result = int.Parse(number);
}
catch
{
    // gives us "OnGetAsync:invalid number - hi"
    _logger.LogInformation($"{MethodBase.GetCurrentMethod()}:invalid number - {number}");
    result = 0;
}
// use result
```

虽然这为我们提供了清晰的日志条目，但对于我们的`try`/`catch`块，我们也可以将这一行复制到`TryParse`示例中。

理念是提供足够的信息给开发者，同时又不让错误打扰用户的体验。

## 应用单元测试方法

如果我们不得不使用 `try`/`catch` 块，就要把它看作是一个单元测试。我们提到单元测试有 AAA 方法（安排、行动和断言）。我们希望创建尽可能少的代码来完成工作并继续前进。

在开始时初始化对象（安排），并在导致错误的可疑行周围包裹一个 `try`/`catch` 块（行动）。尽量减少 `try`/`catch` 块内的代码量。

再次，我们可以将这种方法应用于前面修改过的代码示例，如下所示：

```cs
// Arrange
var number = "2";
int result;
try
{
    // Act
    result = int.Parse(number);
}
// Assert
catch
{
    // gives us "OnGetAsync:invalid number - hi"
    _logger.LogInformation($"{MethodBase.GetCurrentMethod()}:invalid number - {number}");
    result = 0;
}
// use result
```

在 `try` 语句之上的任何内容都会被认为是 `Arrange` 部分；try 括号内的单行语句被认为是 `Act`。`Assert` 部分将是 `catch` 语句的一部分。这个语句将相当于一个失败的 `Assert`。

## 避免空的 `catch` 语句

虽然防止错误对我们应用程序至关重要，但请考虑以下示例中的空 `catch` 语句：

```cs
private void Deposit(Account myAccount, decimal amount)
{
    try
    {
        myAccount.Deposit(amount);
    }
    catch { }
}
```

当 `Deposit()` 方法不起作用时会发生什么？如果我们代码中确实有错误，`catch` 语句应该包含一些内容来让开发者知道发生了错误。至少，应该有一个 `Log` 语句来通知团队有关问题。

虽然空的 `try`/`catch` 块可以防止程序崩溃，但它会引发更大的问题。一旦有人发现存款有问题，由于没有记录问题，可能很难找到问题所在。

## 使用异常过滤和模式匹配

如果我们正在处理代码段中的多个异常，.NET 中的一个新特性，称为异常过滤，可以使异常处理更加简洁。如果我们有紧凑的代码，异常过滤可以提供更高效和现代的代码库。

例如，文件处理常常会导致各种异常。考虑以下代码片段：

```cs
FileStream fileStream = null;
try
{
    fileStream = new FileStream(@"C:\temp\myfile.txt", FileMode.Append);
}
catch (DirectoryNotFoundException e)
{
    _logger.Log(LogLevel.Information, MethodBase.GetCurrentMethod()+":Directory not found - " + e.Message);
}
catch (FileNotFoundException e)
{
    _logger.Log(LogLevel.Information, MethodBase.GetCurrentMethod()+":File Not Found - " + e.Message);
}
catch (IOException e)
{
    _logger.Log(LogLevel.Information, MethodBase.GetCurrentMethod()+":I/O Error - " + e.Message);
}
catch (NotSupportedException e)
{
    _logger.Log(LogLevel.Information, MethodBase.GetCurrentMethod()+":Not Supported - " + e.Message);
}
catch (Exception e)
{
    _logger.Log(LogLevel.Information, MethodBase.GetCurrentMethod()+":General Exception - " + e.Message);
}
// Use filestream
```

在前面的代码中，我们记录每个异常，并根据特定的异常提供消息。使用异常过滤，我们可以缩短行数，使其更容易阅读：

```cs
FileStream fileStream = null;
try
{
    fileStream = new FileStream(@"C:\temp\myfile.txt", FileMode.Append);
}
catch (Exception e) when
    (  e is DirectoryNotFoundException
    || e is FileNotFoundException)
{
    _logger.Log(LogLevel.Warning, $"{MethodBase.GetCurrentMethod()}:{nameof(e)} - {e.Message}");
}
catch (Exception e) when
    (e is NotSupportedException
    || e is IOException)
{
    _logger.Log(LogLevel.Error, $"{MethodBase.GetCurrentMethod()}:{nameof(e)} - {e.Message}");
}
catch (Exception e)
{
    _logger.Log(LogLevel.Error, $"{MethodBase.GetCurrentMethod()}:{nameof(e)} - {e.Message}");
}
// Use filestream
```

在前面的代码中，我们将 `DirectoryNotFoundException` 和 `FileNotFoundException` 进行分组，并将它们记录为警告。当我们遇到 `NotSupportedException` 或 `IOException` 错误时，我们认为这是一个更大的问题，并将这些错误记录为错误。任何其他通过的内容都将被捕获为一般异常，并以消息的形式记录为错误。

除了异常过滤之外，.NET 还引入了另一个新特性，称为模式匹配。使用模式匹配，我们可以进一步缩短代码：

```cs
FileStream fileStream = null;
try
{
    fileStream = new FileStream(@"C:\temp\myfile.txt", FileMode.Append);
}
catch (Exception e)
{
    var logLevel = e switch
    {
        DirectoryNotFoundException => LogLevel.Warning,
        FileNotFoundException => LogLevel.Warning,
        _ => LogLevel.Error
    };
    _logger.Log(logLevel, $"{MethodBase.GetCurrentMethod()}:{nameof(e)} - {e.Message}");
}
// Use filestream
```

前面的代码在 `catch` 括号内使用 `switch` 语句来识别我们正在经历的异常类型。将 switch 模式匹配视为内联的 `if` 语句。前面的行根据异常类型返回 `logLevel` 的值。

下划线（`_`）被称为丢弃变量（它类似于 switch 语句中的默认值）。如果其他所有情况都通过 switch，那么我们将默认将日志级别设置为`LogLevel.Error`，并使用当前方法、异常类型名称和异常消息记录消息。

异常处理可能会很冗长，例如，基于 I/O 的方法和连接。异常过滤和模式匹配可以帮助减轻捕获各种异常的冗长性。

## 使用`finally`块进行清理

当处理数据库连接、基于文件的操作或资源时，最好使用`finally`来进行清理。

例如，如果我们连接到数据库并在之后想要关闭连接，我们就必须创建一段类似于以下代码的代码：

```cs
using System.Data.SqlClient;
var connectionString = @"Data Source=localhost;Initial Catalog=myDatabase;Integrated Security=true;";
using SqlConnection connection = new SqlConnection(connectionString);
var command = new SqlCommand("UPDATE Users SET Name='Jonathan' WHERE ID=1 ", connection);
try
{
    command.Connection.Open();
    command.ExecuteNonQuery();
}
catch (SqlException ex)
{
    Console.WriteLine(ex.ToString());
    throw;
}
finally
{
    connection.Close();
}
```

在前面的代码中，连接对象被传递到`SqlCommand`构造函数中。当我们执行 SQL 查询时，命令被传递到连接并执行。一旦我们的代码执行完毕，我们在`finally`语句中关闭连接。由于我们有`using`语句，并且`SqlConnection`类实现了`IDisposable`接口，它将自动被销毁。

有时我们需要`finally`语句来进行清理，但有时它们是不必要的。

## 知道何时抛出

在之前的代码示例中，我们使用了`throw;`而不是`throw ex;`。

如果我们运行上一节中的代码示例并使用`throw;`，我们会在 Visual Studio 的**调用栈**窗格中看到栈跟踪：

![图 8.1 – 使用简单`throw;`的调用栈快照](img/B19493_08_1.jpg)

图 8.1 – 使用简单`throw;`的调用栈快照

如果我们将该行更改为`throw ex;`会发生什么？

![图 8.2 – 使用`throw ex;`的调用栈快照](img/B19493_08_2.jpg)

图 8.2 – 使用`throw ex;`的调用栈快照

栈跟踪完全消失。如果没有栈跟踪，错误将更难追踪。有时我们只想简单地抛出异常。总是重新抛出异常更好，这样栈跟踪就可以保持完整，以便定位错误。

在本节中，我们讨论了许多在代码中应用异常处理时被认为是实用的标准。我们学习了如何通过“预防胜于异常”来最小化异常，为什么记录日志很重要，以及为什么异常处理就像单元测试一样。

我们还学习了如何避免空的捕获块，使用异常过滤和模式匹配简化异常，何时使用`finally`块，以及如何正确地重新抛出异常。

# 摘要

异常处理很重要，但在编写真正健壮的应用程序时需要一定的经验。应用程序需要恢复，以便用户不会有一个令人不快的体验。

在本章中，我们学习了异常处理、何时使用它以及它在哪些情况下是有意义的，以及性能考虑。

我们通过理解“预防胜于异常”原则、了解日志记录的重要性以及异常处理为何像单元测试一样重要，来结束本章的学习。

我们还了解到空捕获块是浪费的，如何通过异常过滤和模式匹配简化异常，何时使用 finally 块，以及如何正确地重新抛出异常。

在下一章中，我们将探讨 Web API 标准以及它们对 ASP.NET Core 生态系统极端重要性的原因。
