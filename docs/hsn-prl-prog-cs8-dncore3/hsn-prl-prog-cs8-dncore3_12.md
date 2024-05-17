# 第九章：异步、等待和基于任务的异步编程基础

在上一章中，我们介绍了 C#中可用的异步编程实践和解决方案，甚至在.NET Core 之前。我们还讨论了异步编程可以派上用场的场景，以及应该避免使用的场景。

在本章中，我们将更深入地探讨异步编程，并介绍两个使编写异步代码变得非常容易的关键字。本章将涵盖以下主题：

+   介绍`async`和`await`

+   异步委托和 lambda 表达式

+   **基于任务的异步模式**（**TAP**）

+   异步代码中的异常处理

+   使用 PLINQ 进行异步

+   测量异步代码性能

+   使用异步代码的指南

让我们从介绍`async`和`await`关键字开始，这两个关键字首次在 C# 5.0 中引入，并在.NET Core 中也被采用。

# 技术要求

读者应该对**任务并行库**（**TPL**）和 C#有很好的理解。本章的源代码可在 GitHub 上找到：[`github.com/PacktPublishing/-Hands-On-Parallel-Programming-with-C-8-and-.NET-Core-3/tree/master/Chapter09`](https://github.com/PacktPublishing/-Hands-On-Parallel-Programming-with-C-8-and-.NET-Core-3/tree/master/Chapter09)。

# 介绍异步和 await

`async`和`await`是.NET Core 开发人员中非常流行的两个关键字，用于在调用.NET Framework 提供的新异步 API 时标记代码。在上一章中，我们讨论了将同步方法转换为异步方法的挑战。以前，我们通过将方法分解为两个方法`BeginMethodName`和`EndMethodName`来实现异步调用。这种方法使代码变得笨拙，难以编写、调试和维护。然而，使用`async`和`await`关键字，代码可以保持与同步实现相同，只需要进行少量的更改。将方法分解、执行异步方法以及将响应返回给程序的所有困难工作都由编译器完成。

.NET Framework 提供的所有新 I/O API 都支持基于任务的异步性，我们在上一章中已经讨论过。现在让我们尝试理解一些涉及 I/O 操作的场景，我们可以利用`async`和`await`关键字。假设我们想从返回 JSON 格式数据的公共 API 中下载数据。在较旧版本的 C#中，我们可以使用`System.Net`命名空间中提供的`WebClient`类编写同步代码，如下所示。

首先，添加对`System.Net`程序集的引用：

```cs
WebClient client = new WebClient();
string reply = client.DownloadString("http://www.aspnet.com"); 
Console.WriteLine(reply);
```

接下来，创建一个`WebClient`类的对象，并通过传递要下载的页面的 URL 来调用`DownloadString`方法。该方法将同步运行，并且调用线程将被阻塞，直到下载操作完成。这可能会影响服务器的性能（如果在服务器端代码中使用）和应用程序的响应性（如果在 Windows 应用程序代码中使用）。

为了提高性能和响应性，我们可以使用稍后引入的`DownloadString`方法的异步版本。

以下是一个创建远程资源`http://www.aspnet.com`的下载请求并订阅`DownloadStringCompleted`事件的方法，而不是等待下载完成的方法：

```cs
private static void DownloadAsynchronously()
 {
     WebClient client = new WebClient(); 
     client.DownloadStringCompleted += new 
     DownloadStringCompletedEventHandler(DownloadComplete); 
     client.DownloadStringAsync(new Uri("http://www.aspnet.com"));
 }
```

以下是`DownloadComplete`事件处理程序，当下载完成时触发：

```cs
private static void DownloadComplete(object sender, DownloadStringCompletedEventArgs e)
{
     if (e.Error != null)
     {
         Console.WriteLine("Some error has occurred.");
         return;
     }
     Console.WriteLine(e.Result);
     Console.ReadLine();
 }
```

在上述代码中，我们使用了**基于事件的异步模式**（**EAP**）。正如您所看到的，我们已经订阅了`DownloadCompleted`事件，该事件将在`WebClient`类完成下载后被触发。然后，我们调用了`DownloadStringAsync`方法，该方法将异步调用代码并立即返回，避免了阻塞线程的需要。当后台下载完成时，将调用`DownloadComplete`方法，我们可以使用`DownloadStringCompletedEventArgs`的`e.Error`属性接收错误，或使用`e.Result`属性接收数据。

如果我们在 Windows 应用程序中运行上述代码，结果将如预期那样，但响应将始终由工作线程（在后台执行）接收，而不是由主线程接收。作为 Windows 应用程序开发人员，我们需要注意的是，我们不能从`DownloadComplete`方法更新 UI 控件，所有这样的调用都需要使用经典 Windows Forms 中的 Invoke 或 WPF 中的 Dispatcher 等技术委托回主 UI 线程。使用 Invoke/Dispatcher 方法的最大好处是主线程永远不会被阻塞，因此整个应用程序更加响应。

在本书附带的代码示例中，我们包括了 Windows Forms 和 WPF 的场景，尽管.NET Core 目前尚不支持 Windows 应用程序或 WPF。预计这种支持将在下一个版本的 Visual Studio，即 VS 2019 中引入。

让我们尝试在.NET Core 控制台应用程序的主线程中运行上述代码，如下所示：

```cs
 public static void Main()
        {
         DownloadAsynchronously();   
        }
```

我们可以通过在`DownloadComplete`方法中添加`Console.WriteLine`语句来修改它，如下所示：

```cs
private static void DownloadComplete(object sender, DownloadStringCompletedEventArgs e)
        {
            …
            …
            …
            Console.ReadLine() ;//Added this line
        }
```

根据逻辑，程序应该异步下载页面，打印输出，并在终止之前等待用户输入。当我们运行上述代码时，会发现程序在不打印任何内容且不等待用户输入的情况下终止了。为什么会发生这种情况呢？

正如前面所述，一旦主线程调用`DownloadStringAsync`方法，它就会被解除阻塞。主线程不会等待回调函数执行。这是设计上的考虑，异步方法预期以这种方式行为。然而，由于主线程没有其他事情可做，而且已经完成了它预期要做的事情，即调用方法，应用程序终止了。

作为 Web 应用程序开发人员，如果在使用 Web Forms 或 ASP.NET MVC 的服务器端应用程序中使用上述代码，可能会遇到类似的问题。如果您以异步方式调用了该方法，执行您的请求的 IIS 线程将立即返回，而不会等待下载完成。因此，结果将不如预期。我们不希望代码在 Web 应用程序中将输出打印到控制台，当在 Web 应用程序代码中运行时，`Console.WriteLine`语句会被简单地忽略。假设您的逻辑是将网页作为响应返回给客户端请求。我们可以使用 ASP.NET MVC 中的`WebClient`类同步实现这一点，如下例所示：

```cs
public IActionResult Index()
{
    WebClient client = new WebClient();
    string content = client.DownloadString(new 
     Uri("http://www.aspnet.com"));
    return Content(content,"text/html");
}
```

这里的问题是，上述代码将阻塞线程，这可能会影响服务器的性能，并导致自我发起的**拒绝服务**（**DoS**）攻击，当许多用户同时访问应用程序的某一部分时会发生。随着越来越多的线程被命中并被阻塞，将会有一个点，服务器将没有任何空闲线程来处理客户端请求，并开始排队请求。一旦达到队列限制，服务器将开始抛出 503 错误：服务不可用。

由于一旦调用`DownloadStringAsync`方法，线程将立即向客户端返回响应，而不等待`DownloadComplete`完成，因此我们无法使用该方法。我们需要一种方法使服务器线程等待而不阻塞它。在这种情况下，`async`和`await`来拯救我们。除了帮助我们实现我们的目标外，它们还帮助我们编写、调试和维护清晰的代码。

为了演示`async`和`await`，我们可以使用.NET Core 的另一个重要类`HttpClient`，它位于`System.Net.Http`命名空间中。应该使用`HttpClient`而不是`WebClient`，因为它完全支持基于任务的异步操作，具有大大改进的性能，并支持 GET、POST、PUT 和 DELETE 等 HTTP 方法。

以下是使用`HttpClient`类和引入`async`和`await`关键字的前面代码的异步版本：

```cs
public async Task<IActionResult> Index()
        {
            HttpClient client = new HttpClient();
            HttpResponseMessage response = await 
             client.GetAsync("http://www.aspnet.com");
            string content = await response.Content.ReadAsStringAsync();
            return Content(content,"text/html");
        }
```

首先，我们需要更改方法签名以包含`async`关键字。这是对编译器的指示，表明该方法将根据需要异步执行。然后，我们将方法的返回类型包装在`Task<T>`中。这很重要，因为.NET Framework 支持基于任务的异步操作，所有异步方法必须返回`Task`。

我们需要创建`HttpClient`类的一个实例，并调用`GetAsync()`方法，传递要下载的资源的 URL。与依赖于回调的 EAP 模式不同，我们只需在调用时写上`await`关键字。这确保了以下情况：

+   该方法异步执行。

+   调用线程被解除阻塞，以便它可以返回线程池并处理其他客户端请求，从而使服务器响应。

+   当下载完成时，`ThreadPool`从处理器接收到中断信号，并从`ThreadPool`中取出一个空闲线程，可以是正在处理请求的相同线程，也可以是不同的线程。

+   `ThreadPool`线程接收到响应并开始执行方法的其余部分。

当下载完成时，我们可以使用另一个异步操作`ReadAsStringAsync()`来读取下载的内容。本节已经表明，编写类似于同步方法的异步方法非常容易，使它们的逻辑也很直接。

# 异步方法的返回类型

在上面的示例中，我们将方法的返回类型从`IAsyncResult`更改为`Task<IAsyncResult>`。异步方法可以有三种返回类型：

+   `void`

+   `Task`

+   `Task<T>`

所有异步方法必须返回一个`Task`以便被等待（使用`await`关键字）。这是因为一旦调用它们，它们不会立即返回，而是异步执行一个长时间运行的任务。在这样做的过程中，调用线程也可能在上下文中切换。

`void`可以与调用线程不想等待的异步方法一起使用。这些方法可以是后台发生的任何操作，不是返回给用户的响应的一部分。例如，日志记录和审计可以是异步的。这意味着它们可以包装在异步的`void`方法中。调用操作时，调用线程将立即返回，日志记录和审计操作将稍后进行。因此，强烈建议从异步方法返回`Task`而不是`void`。

# 异步委托和 lambda 表达式

我们也可以使用`async`关键字创建异步委托和 lambda 表达式。

以下是返回数字的平方的同步委托：

```cs
Func<int, int> square = (x) => {return x * x;};
```

我们可以通过添加`async`关键字使前面的委托异步化，如下所示：

```cs
Func<int, Task<int>> square =async (x) => {return x * x;};
```

类似地，lambda 表达式可以转换如下：

```cs
Func<int, Task<int>> square =async (x) => x * x;
```

异步方法在一个链条中工作。一旦你将任何一个方法变成异步方法，那么调用该方法的所有方法也需要被转换为异步方法，从而创建一个长链的异步方法。

# 基于任务的异步模式

在第二章中，*任务并行性*，我们讨论了如何使用`Task`类实现 TAP。有两种实现这种模式的方法：

+   编译器方法，使用`async`关键字

+   手动方法

让我们在后续章节中看看这些方法是如何操作的。

# 编译器方法，使用 async 关键字

当我们使用`async`关键字使任何方法成为异步方法时，编译器会进行必要的优化，使用 TAP 在内部异步执行该方法。异步方法必须返回`System.Threading.Task`或`System.Threading.Task<T>`。编译器负责异步执行方法并将结果或异常返回给调用者。

# 手动实现 TAP

我们已经展示了如何在 EAP 和**异步编程模型**（**APM**）中手动实现 TAP。实现这种模式可以让我们更好地控制方法的整体实现。我们可以创建一个`TaskCompletionSource<TResult>`类，然后执行一个异步操作。当异步操作完成时，我们可以通过调用`TaskCompletionSource<TResult>`类的`SetResult`、`SetException`或`SetCanceled`方法将结果返回给调用者，如下面的代码所示：

```cs
public static Task<int> ReadFromFileTask(this FileStream stream, byte[] buffer, int offset, int count, object state)
{
    var taskCompletionSource = new TaskCompletionSource<int>();
    stream.BeginRead(buffer, offset, count, ar =>
    {
         try 
         { 
               taskCompletionSource.SetResult(stream.EndRead(ar));
         }
         catch (Exception exc) 
         { 
               taskCompletionSource.SetException(exc); 
         }
     }, state);
     return taskCompletionSource.Task;
}
```

在上面的代码中，我们创建了一个返回`Task<int>`的方法，可以作为扩展方法在任何`System.IO.FileStream`对象上工作。在方法内部，我们创建了一个`TaskCompletionSource<int>`对象，然后调用`FileStream`类提供的异步操作将文件读入字节数组。如果读取操作成功完成，我们使用`SetResult`方法将结果返回给调用者；否则，我们使用`SetException`方法返回异常。最后，该方法将从`TaskCompletionSource<int>`对象返回底层任务给调用者。

# 异步代码的异常处理

在同步代码的情况下，所有异常都会传播到堆栈的顶部，直到它们被 try-catch 块处理或作为未处理的异常抛出。当我们在任何异步方法上等待时，调用堆栈将不会相同，因为线程已经从方法转换到线程池，并且现在正在返回。然而，C#通过改变异步方法的异常行为，使我们更容易进行异常处理。所有异步方法都返回`Task`或`void`。让我们尝试用例子理解这两种情况，并看看程序的行为。

# 返回 Task 并抛出异常的方法

假设我们有以下方法，它是`void`。作为最佳实践，我们从中返回`Task`：

```cs
 private static Task DoSomethingFaulty()
 {
      Task.Delay(2000);
      throw new Exception("This is custom exception.");
 }
```

该方法在延迟两秒后抛出异常。

我们将尝试使用各种方法调用此方法，以尝试理解异步方法的异常处理行为。本节将讨论以下场景：

+   在 try-catch 块外部调用异步方法，没有使用`await`关键字

+   在 try-catch 块内部调用异步方法，没有使用`await`关键字

+   在 try-catch 块外部使用 await 关键字调用异步方法

+   返回`void`的方法

我们将在后续章节中详细介绍这些方法。

# 在 try-catch 块外部调用异步方法，没有使用 await 关键字

以下是一个返回`Task`的示例异步方法。该方法调用另一个方法`DoSomethingFaulty()`，该方法会抛出异常。

这是我们的`DoSomethingFaulty()`方法实现：

```cs
  private static Task DoSomethingFaulty()
  {
      Task.Delay(2000);
      throw new Exception("This is custom exception.");
  }
```

以下是`AsyncReturningTaskExample()`方法的代码：

```cs
private async static Task AsyncReturningTaskExample()
 {
      Task<string> task = DoSomethingFaulty();
      Console.WriteLine("This should not execute");
      try
      {
           task.ContinueWith((s) =>
           {
             Console.WriteLine(s);
           });
      }
      catch (Exception ex)
      {
       Console.WriteLine(ex.Message);
       Console.WriteLine(ex.StackTrace);
      }
  }
```

这是从`Main()`方法调用的：

```cs
 public static void Main()
 {
     Console.WriteLine("Main Method Starts");
     var task = AsyncReturningTaskExample();
     Console.WriteLine("In Main Method After calling method");
     Console.ReadLine();
 }
```

异步主方法是 C# 7.1 版本以后的一个方便的补充。它在 7.2 版本中出现了问题，但在.NET Core 3.0 中得到了修复。

如您所见，程序调用了异步方法——即`AsyncReturningTaskExample()`——而没有使用`await`关键字。`AsyncReturningTaskExample()`方法进一步调用了`DoSomethingFaulty()`方法，该方法抛出异常。当我们运行此代码时，将产生以下输出：

![](img/88073eb0-9010-45f1-838e-1239ced84266.png)

在同步编程的情况下，程序会导致未处理的异常，并且会崩溃。但在这里，程序会继续进行，就好像什么都没有发生一样。这是由于框架处理`Task`对象的方式。在这种情况下，任务将以故障状态返回给调用者，如下面的截图所示：

![](img/27e57487-af51-4fc0-94a8-7c5a17059fa6.png)

更好的代码应该是检查任务状态并在有异常时获取所有异常：

```cs
var task = AsyncReturningTaskExample();
if (task.IsFaulted)
    Console.WriteLine(task.Exception.Flatten().Message.ToString());
```

正如我们在第二章中看到的*任务并行性*，这个任务返回一个`AggregateExceptions`的实例。要获取所有抛出的内部异常，我们可以使用`Flatten()`方法，就像在前面的截图中演示的那样。

# 在 try-catch 块内部没有使用 await 关键字的异步方法

让我们将调用异步方法`GetSomethingFaulty()`的方法移动到 try-catch 块内，并从`Main()`方法调用。

这是`Main`方法：

```cs
public static void Main()
{
    Console.WriteLine("Main Method Started");
    var task = Scenario2CallAsyncWithoutAwaitFromInsideTryCatch();
    if (task.IsFaulted)
        Console.WriteLine(task.Exception.Flatten().Message.ToString());
    Console.WriteLine("In Main Method After calling method");
    Console.ReadLine();
}       
```

这里是`Scenario2CallAsyncWithoutAwaitFromInsideTryCatch()`方法：

```cs
private async static Task Scenario2CallAsyncWithoutAwaitFromInsideTryCatch()
{
     try
     {
         var task = DoSomethingFaulty();
         Console.WriteLine("This should not execute"); 
         task.ContinueWith((s) =>
         {
             Console.WriteLine(s);
         });
     }
     catch (Exception ex)
     {
         Console.WriteLine(ex.Message);
         Console.WriteLine(ex.StackTrace);
     }
}
```

这次，我们看到异常将被抛出并被 catch 块接收，之后程序将正常恢复。

值得一看的是`Main`方法中`Task`对象的值：

![](img/e97dd4ef-dfcd-4677-b88d-d4e855a6c81b.png)

如您所见，如果任务创建不在 try-catch 块内进行，异常将不会被观察到。这可能会导致问题，因为逻辑可能不会按预期工作。最佳实践是始终将任务创建包装在 try-catch 块内。

如您所见，由于异常已被处理，执行从异步方法正常返回。返回任务的状态变为`RanToCompletion`。

# 使用 await 关键字从 try-catch 块外部调用异步方法

以下代码块显示了调用有错误的方法`DoSomethingFaulty()`并等待方法完成的方法的代码，使用`await`关键字：

```cs
private async static Task Scenario3CallAsyncWithAwaitFromOutsideTryCatch()
{
    await DoSomethingFaulty();
    Console.WriteLine("This should not execute"); 
}
```

这是从`Main`方法调用的：

```cs
public static void Main()
{
      Console.WriteLine("Main Method Starts");
      var task = Scenario3CallAsyncWithAwaitFromOutsideTryCatch();
      if (task.IsFaulted)
          Console.WriteLine(task.Exception.Flatten().Message.ToString());
      Console.WriteLine("In Main Method After calling method");
      Console.ReadLine();
}
```

在这种情况下，程序的行为将与第一个场景相同。

# 返回 void 的方法

如果方法返回`void`而不是`Task`，程序将崩溃。您可以尝试运行以下代码。

这是一个返回`void`而不是`Task`的方法：

```cs
private async static void Scenario4CallAsyncWithoutAwaitFromOutsideTryCatch()
{
    Task task = DoSomethingFaulty();
    Console.WriteLine("This should not execute");
}
```

这是从`Main`方法调用的：

```cs
public static void Main()
{
    Console.WriteLine("Main Method Started"); 
    Scenario4CallAsyncWithoutAwaitFromOutsideTryCatch();
    Console.WriteLine("In Main Method After calling method"); 
    Console.ReadLine();
}
```

不会有输出，因为程序会崩溃。

虽然从异步方法中返回`void`是没有意义的，但错误确实会发生。我们应该编写代码，使其永远不会崩溃，或者在记录异常后优雅地崩溃。

我们可以通过订阅两个全局事件处理程序来全局处理这个问题，如下所示：

```cs
AppDomain.CurrentDomain.UnhandledException += (s, e) => Console.WriteLine("Program Crashed", "Unhandled Exception Occurred");
TaskScheduler.UnobservedTaskException += (s, e) => Console.WriteLine("Program Crashed", "Unhandled Exception Occurred");
```

前面的代码将处理程序中的所有未处理异常，并考虑了异常管理中的良好实践。程序不应该随机崩溃，如果需要崩溃，那么应该记录信息并清理所有资源。

# 使用 PLINQ 进行异步

PLINQ 是开发人员非常方便的工具，可以通过并行执行一组任务来提高应用程序的性能。创建多个任务可以提高性能，但是，如果任务具有阻塞性质，那么应用程序最终将创建大量阻塞线程，并且在某些时候会变得无响应。特别是如果任务正在执行一些 I/O 操作。以下是一个需要尽快从网络下载 100 页的方法：

```cs
 public async static void Main()
        {
            var urls =  Enumerable.Repeat("http://www.dummyurl.com", 100);
            foreach (var url in urls)
            {
                HttpClient client = new HttpClient();
                HttpResponseMessage response = await 
                 client.GetAsync("http://www.aspnet.com");
                string content = await 
                  response.Content.ReadAsStringAsync();
                Console.WriteLine();
            }
```

如您所见，上述代码是同步的，具有*O(n)*的复杂度。如果一个请求需要一秒钟才能完成，那么该方法至少需要 100 秒（n = 100）。

为了加快下载速度（假设我们有一个能够处理此负载的良好服务器配置，乘以应用程序想要支持的用户数量），我们需要并行执行此方法。我们可以使用`Parallel.ForEach`来实现：

```cs
     Parallel.ForEach(urls, url =>
            {
                HttpClient client = new HttpClient();
                HttpResponseMessage response = await 
                 client.GetAsync("http://www.aspnet.com");
                string content = await 
                 response.Content.ReadAsStringAsync();
            });
```

突然，代码开始抱怨：

*'await'运算符只能在异步 lambda 表达式中使用。考虑使用'async'修饰符标记此 lambda 表达式。*

这是因为我们使用了 lambda 表达式，它也需要被标记为 async，如下面的代码所示：

```cs
Parallel.ForEach(urls,async url =>
            {
                HttpClient client = new HttpClient();
                HttpResponseMessage response = await 
                 client.GetAsync("http://www.aspnet.com");
                string content = await 
                 response.Content.ReadAsStringAsync();
            });
```

现在代码将会编译并按预期工作，性能得到了大幅提升。在下一节中，我们将更深入地讨论异步代码性能的测量方法。

# 测量异步代码的性能

异步代码可以提高应用程序的性能和响应性，但也存在一些权衡。在基于 GUI 的应用程序（如 Windows Forms 或 WPF）中，如果一个方法花费了很长时间，将其标记为异步是有意义的。然而，对于服务器应用程序，您需要权衡受阻线程所使用的额外内存和使方法异步所需的额外处理器开销之间的权衡。

考虑以下代码，它创建了三个任务。每个任务都是异步运行的，一个接一个地执行。当一个方法完成时，它会继续异步执行另一个任务。使用`Stopwatch`可以计算完成方法所需的总时间：

```cs
public static void Main(string[] args)
{
    MainAsync(args).GetAwaiter().GetResult();
    Console.ReadLine();
}
public static async Task MainAsync(string[] args)
{
    Stopwatch stopwatch = Stopwatch.StartNew();
    var value1 = await Task1();
    var value2 = await Task2();
    var value3 = await Task3();
    stopwatch.Stop();
    Console.WriteLine($"Total time taken is 
     {stopwatch.ElapsedMilliseconds}");
}
public static async Task<int> Task1()
{
    await Task.Delay(2000);
    return 100;
}
public static async Task<int> Task2()
{
    await Task.Delay(2000);
    return 200;
}
public static async Task<int> Task3()
{
    await Task.Delay(2000);
    return 300;
}
```

上述代码的输出如下：

![](img/c0d8cd2f-b74e-4b8c-ba02-0640678401d6.png)

这与编写同步代码一样好。好处是线程不会被阻塞，但应用程序的整体性能较差，因为所有代码现在都是同步运行的。我们可以改变上述代码以提高性能，如下所示：

```cs
Stopwatch stopwatch = Stopwatch.StartNew();
       await Task.WhenAll(Task1(), Task2(), Task3());
       stopwatch.Stop();
       Console.WriteLine($"Total time taken is {stopwatch.ElapsedMilliseconds}");
```

如您所见，这是更好地使用并行和异步以获得更好的性能：

![](img/f7c04eec-fc9f-43ca-8fe3-a329e7f7046b.png)

为了更好地理解异步，我们还需要了解哪个线程运行我们的代码。由于新的异步 API 与`Task`类一起工作，所有调用都由`ThreadPool`线程执行。当我们进行异步调用时，比如从网络获取数据，控制权会转移到由操作系统管理的 I/O 完成端口线程。通常，这只是一个线程，跨所有网络请求共享。当 I/O 请求完成时，操作系统会触发中断信号，将作业添加到 I/O 完成端口的队列中。在通常以**多线程公寓**（**MTA**）模式工作的服务器端应用程序中，任何线程都可以启动异步请求，任何其他线程都可以接收它。

在 Windows 应用程序的情况下（包括 WinForms 和 WPF），它们以**单线程公寓**（STA）模式工作，因此异步调用返回到启动它的同一线程（通常是 UI 线程）变得很重要。Windows 应用程序中的每个 UI 线程都有一个`SynchronizationContext`，它确保代码始终由正确的线程执行。这对于控件所有权很重要。为了避免跨线程问题，只有所有者线程才能更改控件的值。`SynchronizationContext`类的最重要方法是`Post`，它可以使委托在正确的上下文中运行，从而避免跨线程问题。

每当我们等待一个任务时，当前的`SynchronizationContext`都会被捕获。然后，当方法需要恢复时，`await`关键字在内部使用`Post`方法在捕获的`SynchronizationContext`中恢复方法。然而，调用`Post`方法非常昂贵，但框架提供了内置的性能优化。如果捕获的`SynchronizationContext`与返回线程的当前`SynchronizationContext`相同，则不会调用`Post`方法。

如果我们正在编写一个类库，并且我们并不真的关心调用将返回到哪个`SynchronizationContext`，我们可以完全关闭`Post`方法。我们可以通过在返回的任务上调用`ConfigureAwait()`方法来实现这一点，如下所示：

```cs
HttpClient client = new HttpClient();
HttpResponseMessage response = await client.GetAsync(url).ConfigureAwait(false);
```

到目前为止，我们已经学习了异步编程的重要方面。现在我们需要了解在编程时使用异步代码的指南！

# 使用异步代码的指南

在编写异步代码时的一些建议/最佳实践如下：

+   避免使用异步 void。

+   异步链一直延续。

+   在可能的情况下使用`ConfigureAwait`。

我们将在接下来的部分中了解更多。

# 避免使用异步 void

我们已经看到从异步方法返回`void`实际上会影响异常处理。异步方法应该返回`Task`或`Task<T>`，以便可以观察异常并且不会变成未处理的异常。

# 异步链一直延续

混合异步和阻塞方法会影响性能。一旦决定将方法设置为异步，从该方法调用的整个方法链也应该设置为异步。不这样做有时会导致死锁，如下面的代码示例所示：

```cs
private async Task DelayAsync()
{
    await Task.Delay(2000);
}
public void Deadlock()
{
    var task = DelayAsync();
    task.Wait();
}
```

如果我们从任何 ASP.NET 或基于 GUI 的应用程序中调用`Deadlock()`方法，它将创建死锁，尽管相同的代码在控制台应用程序中可以正常运行。当我们调用`DelayAsync()`方法时，它会捕获当前的`SynchronizationContext`，或者如果`SynchronizationContext`为 null，则捕获当前的`TaskScheduler`。当等待的任务完成时，它会尝试使用捕获的上下文执行方法的其余部分。问题在于已经有一个线程在同步等待异步方法完成。在这种情况下，两个线程都将等待另一个线程完成，从而导致死锁。这个问题只会在基于 GUI 或 ASP.NET 的应用程序中出现，因为它们依赖于只能一次执行一块代码的`SynchronizationContext`。另一方面，控制台应用程序使用`ThreadPool`而不是`SynchronizationContext`。当等待完成时，挂起的异步方法部分被安排在`ThreadPool`线程上。该方法在单独的线程上完成并将任务返回给调用者，因此不会发生死锁。

永远不要在控制台应用程序中尝试创建示例`async`/`await`代码，然后将其复制粘贴到 GUI 或 ASP.NET 应用程序中，因为它们有不同的执行异步代码的模型。

# 在可能的情况下使用 ConfigureAwait

我们可以通过完全跳过使用`SynchronizationContext`来避免前面代码示例中的死锁：

```cs
private async Task DelayAsync()
{
await Task.Delay(2000);
}
public void Deadlock()
{
var task = DelayAsync().ConfigureAwait(false);
task.Wait();
}
```

当我们使用`ConfigureAwait(false)`时，该方法会被等待。当等待完成时，处理器会尝试在线程池上下文中执行剩余的异步方法。由于没有阻塞上下文，该方法能够顺利完成。该方法完成了其返回的任务，没有死锁。

我们已经到达了本章的结尾。现在让我们看看我们学到了什么！

# 摘要

在本章中，我们讨论了两个非常重要的构造，使得编写异步代码变得非常容易。当我们使用这些关键字时，所有繁重的工作都是由编译器完成的，代码看起来与其同步对应物非常相似。我们还讨论了当我们使方法异步化时，代码运行在哪个线程上，以及利用`SynchronizationContext`会带来的性能损失。最后，我们看了如何完全关闭`SynchronizationContext`以提高性能。

在下一章中，我们将介绍使用 Visual Studio 进行并行调试技术。我们还将学习 Visual Studio 中可用的工具，以帮助并行代码调试。

# 问题

1.  在异步方法中，用什么关键字来解除线程阻塞？

1.  `异步`

1.  `await`

1.  `Thread.Sleep`

1.  `Task`

1.  以下哪些是异步方法的有效返回类型？

1.  `无`

1.  `Task`

1.  `Task<T>`

1.  `IAsyncResult`

1.  `TaskCompletionSource<T>`可以用来手动实现基于任务的异步模式。

1.  真

1.  假

1.  我们可以将`Main`方法写成异步的吗？

1.  是

1.  不

1.  `Task`类的哪个属性可以用来检查异步方法是否抛出了异常？

1.  `IsException`

1.  `IsFaulted`

1.  我们应该总是将`void`作为异步方法的返回类型使用。

1.  真

1.  假
