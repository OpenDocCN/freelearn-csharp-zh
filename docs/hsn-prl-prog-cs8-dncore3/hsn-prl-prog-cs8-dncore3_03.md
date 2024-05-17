# 第二章：任务并行性

在上一章中，我们介绍了并行编程的概念。在本章中，我们将继续讨论 TPL 和任务并行性。

.NET 作为一个编程框架的主要目标之一是通过将所有常见的任务封装为 API 来使开发人员的生活更轻松。正如我们已经看到的，线程自.NET 的早期版本以来就存在，但最初它们非常复杂，并且伴随着很多开销。微软引入了许多新的并行原语，使得从头开始编写、调试和维护并行程序变得更加容易，而无需处理与传统线程相关的复杂性。

本章将涵盖以下主题：

+   创建和启动任务

+   从已完成的任务获取结果

+   如何取消任务

+   如何等待运行任务

+   处理任务异常

+   将**异步编程模型**（**APM**）模式转换为任务

+   将**基于事件的异步模式**（**EAPs**）转换为任务

+   更多关于任务的内容：

+   继续任务

+   父任务和子任务

+   本地和全局队列和存储

+   工作窃取队列

# 技术要求

要完成本章，您应该对 C#和一些高级概念（如委托）有很好的理解。

本章的源代码可在 GitHub 上找到：[`github.com/PacktPublishing/Hands-On-Parallel-Programming-with-C-8-and-.NET-Core-3/tree/master/Chapter02`](https://github.com/PacktPublishing/Hands-On-Parallel-Programming-with-C-8-and-.NET-Core-3/tree/master/Chapter02)。

# 任务

**任务**是.NET 中提供异步单元的抽象，就像 JavaScript 中的 promise 一样。在.NET 的初始版本中，我们只能依赖于线程，这些线程是直接创建或使用`ThreadPool`类创建的。`ThreadPool`类提供了对线程的托管抽象层，但开发人员仍然依赖于`Thread`类来获得更好的控制。通过使用`Thread`类创建线程，我们可以获得底层对象，可以等待、取消或移动到前台或后台。然而，在实时中，我们需要线程持续执行工作。这要求我们编写大量难以维护的代码。`Thread`类也是不受管理的，这对内存和 CPU 都造成了很大的负担。我们需要两全其美，这就是任务的用武之地。任务只是通过`ThreadPool`创建的线程的包装器。任务提供了等待、取消和继续等功能，这些功能在任务完成后运行。

任务具有以下重要特点：

+   任务由`TaskScheduler`执行，默认调度程序简单地在`ThreadPool`上运行。

+   我们可以从任务中返回值。

+   任务让您知道它们何时完成，不像`ThreadPool`或线程。

+   可以使用`ContinueWith()`构造来运行任务的后续任务。

+   我们可以通过调用`Task.Wait()`来等待任务。这会阻塞调用线程，直到任务完成为止。

+   与传统线程或`ThreadPool`相比，任务使代码更易读。它们还为引入 C# 5.0 中的异步编程构造铺平了道路。

+   当一个任务从另一个任务启动时，我们可以建立父/子关系。

+   我们可以将子任务的异常传播到父任务。

+   可以使用`CancellationToken`类取消任务。

# 创建和启动任务

我们可以使用 TPL 的许多方法来创建和运行任务。在本节中，我们将尝试理解所有这些方法，并在可能的情况下进行比较分析。首先，您需要向`System.Threading.Tasks`命名空间添加引用：

```cs
using System.Threading.Tasks;
```

我们将尝试使用以下方法创建任务：

+   `System.Threading.Tasks.Task`类

+   `System.Threading.Tasks.Task.Factory.StartNew` 方法

+   `System.Threading.Tasks.Task.Run` 方法

+   `System.Threading.Tasks.Task.Delay`

+   `System.Threading.Tasks.Task.Yield`

+   `System.Threading.Tasks.Task.FromResult<T>方法`

+   `System.Threading.Tasks.Task.FromException`和`Task.FromException<T>`

+   `System.Threading.Tasks.Task.FromCancelled`和`Task.FromCancelled<T>`

# System.Threading.Tasks.Task 类

任务类是一种以`ThreadPool`线程异步执行工作的方式，它基于**基于任务的异步模式**（**TAP**）。非泛型的`Task`类不返回结果，所以每当我们需要从任务中返回值时，我们需要使用泛型版本`Task<T>`。通过`Task`类创建的任务直到我们调用`Start`方法才被安排运行。

我们可以通过`Task`类的各种方式创建一个任务，所有这些方式我们将在以下小节中讨论。

# 使用 lambda 表达式语法

在以下代码中，我们通过调用`Task`构造函数并传递包含我们要执行的方法的 lambda 表达式来创建一个任务：

```cs
Task task = new Task (() => PrintNumber10Times ());
task.Start();
```

# 使用 Action delegate

在以下代码中，我们通过调用`Task`构造函数并传递包含我们要执行的方法的 delegate 来创建一个任务：

```cs
Task task = new Task (new Action (PrintNumber10Times));
task.Start();
```

# 使用 delegate

在以下代码中，我们通过调用`Task`构造函数并传递包含我们要执行的方法的匿名`delegate`来创建一个`task`对象：

```cs
Task task = new Task (delegate {PrintNumber10Times ();});
task.Start();
```

在所有这些情况下，输出将如下所示：

![](img/40ef715e-dff5-40e9-a0a0-acafb3317b7c.png)

所有前面的方法都是做同样的事情 - 它们只是有不同的语法。

我们只能对以前未运行过的任务调用`Start`方法。如果您需要重新运行已经完成的任务，您需要创建一个新的任务并在其上调用`Start`方法。

# System.Threading.Tasks.Task.Factory.StartNew 方法

我们也可以使用`TaskFactory`类的`StartNew`方法创建一个任务，如下所示。在这种方法中，任务被创建并安排在`ThreadPool`内执行，并将该任务的引用返回给调用者。

我们可以使用`Task.Factory.StartNew`方法创建一个任务。我们将在以下小节中讨论这个问题。

# 使用 lambda 表达式语法

在以下代码中，我们通过在`TaskFactory`上调用`StartNew()`方法并传递包含我们要执行的方法的 lambda 表达式来创建一个`Task`：

```cs
Task.Factory.StartNew(() => PrintNumber10Times());          
```

# 使用 Action delegate

在以下代码中，我们通过在`TaskFactory`上调用`StartNew()`方法并传递包装我们要执行的方法的 delegate 来创建一个`Task`：

```cs
Task.Factory.StartNew(new Action( PrintNumber10Times));
```

# 使用 delegate

在以下代码中，我们通过在`TaskFactory`上调用`StartNew()`方法并传递我们要执行的`delegate`包装方法来创建一个`Task`：

```cs
 Task.Factory.StartNew(delegate { PrintNumber10Times(); });
```

所有前面的方法都是做同样的事情 - 它们只是有不同的语法。

# System.Threading.Tasks.Task.Run 方法

我们也可以使用`Task.Run`方法创建一个任务。这与`StartNew`方法的工作方式相同，并返回一个`ThreadPool`线程。

我们可以通过以下方式使用`Task.Run`方法创建一个`Task`，所有这些方式将在以下小节中讨论。

# 使用 lambda 表达式语法

在以下代码中，我们通过在`Task`上调用静态的`Run()`方法并传递包含我们要执行的方法的 lambda 表达式来创建一个`Task`：

```cs
Task.Run(() => PrintNumber10Times ());
```

# 使用 Action delegate

在以下代码中，我们通过在`Task`上调用静态的`Run()`方法并传递包含我们要执行的方法的 delegate 来创建一个`Task`：

```cs
Task.Run(new Action (PrintNumber10Times));
```

# 使用 delegate

在以下代码中，我们通过在`Task`上调用静态的`Run()`方法并传递包含我们要执行的方法的 delegate 来创建一个`Task`：

```cs
Task.Run(delegate {PrintNumber10Times ();});
```

# System.Threading.Tasks.Task.Delay 方法

我们可以创建一个在指定时间间隔后完成或可以随时被用户取消的任务，使用`CancellationToken`类。过去，我们使用`Thread`类的`Thread.Sleep()`方法创建阻塞构造以等待其他任务。然而，这种方法的问题是它仍然使用 CPU 资源并且同步运行。`Task.Delay`提供了一个更好的等待任务的替代方法，而不利用 CPU 周期。它也是异步运行的：

```cs
Console.WriteLine("What is the output of 20/2\. We will show result in 2 seconds.");
Task.Delay(2000);
Console.WriteLine("After 2 seconds delay");
Console.WriteLine("The output is 10");
```

前面的代码询问用户一个问题，然后等待两秒钟才呈现答案。在这两秒钟内，主线程不必等待，但必须执行其他任务以改善用户体验。代码在系统时钟上异步运行，一旦时间到期，其余代码就会被执行。

前面代码的输出如下：

![](img/e92b0ddd-683c-4c1f-92e9-6d9092ff30b0.png)

在查看我们可以用来创建任务的其他方法之前，我们将看一下在 C# 5.0 中引入的两个异步编程构造：`async`和`await`关键字。

`async`和`await`是代码标记，使我们更容易编写异步程序。我们将在第九章中深入学习这些关键字，*异步、等待和基于任务的异步编程基础*。顾名思义，我们可以使用`await`关键字等待任何异步调用。一旦执行线程在方法内遇到`await`关键字，它就返回到`ThreadPool`，将方法的其余部分标记为继续委托，并开始执行其他排队的任务。一旦异步任务完成，`ThreadPool`中的任何可用线程都会完成方法的其余部分。

# System.Threading.Tasks.Task.Yield 方法

这是创建`await`任务的另一种方式。底层任务对调用者不直接可访问，但在涉及与程序执行相关的异步编程的某些场景中使用。它更像是一个承诺而不是一个任务。使用`Task.Yield`，我们可以强制我们的方法是异步的，并将控制返回给操作系统。当方法的其余部分在以后的时间点执行时，它可能仍然作为异步代码运行。我们可以使用以下代码实现相同的效果：

```cs
await Task.Factory.StartNew(() => {},
    CancellationToken.None,
    TaskCreationOptions.None,
    SynchronizationContext.Current != null?
    TaskScheduler.FromCurrentSynchronizationContext():
    TaskScheduler.Current);
```

这种方法可以通过在长时间运行的任务中不时地将控制权交给 UI 线程来使 UI 应用程序响应。然而，这不是 UI 应用程序的首选方法。有更好的替代方法，例如 WinForms 中的`Application.DoEvents()`和 WPF 中的`Dispatcher.Yield(DispatcherPriority.ApplicationIdle)`：

```cs
private async static void TaskYield()
{
     for (int i = 0; i < 100000; i++)
     {
        Console.WriteLine(i);
        if (i % 1000 == 0)
        await Task.Yield();
     }
}
```

在控制台或 Web 应用程序的情况下，当我们运行代码并在任务的 yield 上应用断点时，我们会看到随机线程池线程切换上下文来运行代码。以下截图描述了各个阶段控制执行的各个线程。

以下截图显示了程序流中所有线程同时执行。我们可以看到当前线程 ID 为 1664：

![](img/5a633f20-226e-49d4-aa64-3d7f3ca6092d.png)

如果我们按下*F5*并允许断点命中`i`的另一个值，我们会看到代码现在由 ID 为 10244 的另一个线程执行：

![](img/e7dd5bc6-0a11-4692-8ade-63a6fef19e9d.png)

我们将在第十一章中学习更多关于线程窗口和调试技术，*为并行和异步代码编写单元测试用例*。

# System.Threading.Tasks.Task.FromResult<T>方法

这种方法是最近在.NET 框架 4.5 中引入的，它非常被低估。我们可以通过这种方法返回带有结果的完成任务，如下所示：

```cs
static void Main(string[] args)
{
    StaticTaskFromResultUsingLambda();
}
private static void StaticTaskFromResultUsingLambda()
{
    Task<int> resultTask = Task.FromResult<int>( Sum(10));
    Console.WriteLine(resultTask.Result);
}
private static int Sum (int n)
{
    int sum=0;
    for (int i = 0; i < 10; i++)
    {
        sum += i;
    }
    return sum;
}
```

如前面的代码所示，我们实际上将同步的`Sum`方法转换为使用`Task.FromResult<int>`类以异步方式返回结果。这种方法经常用于 TDD 中模拟异步方法，以及在异步方法内根据条件返回默认值。我们将在第十一章中进一步解释这些方法，*编写并行和异步代码的单元测试用例**.*

# System.Threading.Tasks.Task.FromException 和 System.Threading.Tasks.Task.FromException<T>方法

这些方法创建了由预定义异常完成的任务，并用于从异步任务中抛出异常，以及在 TDD 中。我们将在第十一章中进一步解释这种方法，*编写并行和异步代码的单元测试用例**.*

```cs
return Task.FromException<long>(
new FileNotFoundException("Invalid File name."));
```

正如你在前面的代码中看到的，我们将`FileNotFoundException`包装为一个任务并将其返回给调用者。

# System.Threading.Tasks.Task.FromCanceled 和 System.Threading.Tasks.Task.FromCanceled<T>方法

这些方法用于创建由取消令牌导致完成的任务：

```cs
CancellationTokenSource source = new CancellationTokenSource();
var token = source.Token;
source.Cancel();
Task task = Task.FromCanceled(token);
Task<int> canceledTask = Task.FromCanceled<int>(token);
```

如前面的代码所示，我们使用`CancellationTokenSource`类创建了一个取消令牌。然后，我们从该令牌创建了一个任务。这里需要考虑的重要事情是，在我们可以使用`Task.FromCanceled`方法之前，令牌需要被取消。

如果我们想要从异步方法中返回值，以及在 TDD 中，这种方法是有用的。

# 从已完成的任务中获取结果

为了从任务中返回值，TPL 提供了我们之前定义的所有类的泛型变体：

+   `Task<T>`

+   `Task.Factory.StartNew<T>`

+   `Task.Run<T>`

任务完成后，我们应该能够通过访问`Task.Result`属性来获取结果。让我们尝试使用一些代码示例来理解这一点。我们将创建各种任务，并在完成后尝试返回值：

```cs
using System;
using System.Threading.Tasks;
namespace Ch02
{
    class _2GettingResultFromTasks
    {
        static void Main(string[] args)
        {
            GetResultsFromTasks();
            Console.ReadLine();
        }
        private static void GetResultsFromTasks()
        {
            var sumTaskViaTaskOfInt = new Task<int>(() => Sum(5));
            sumTaskViaTaskOfInt.Start();
            Console.WriteLine($"Result from sumTask is
             {sumTaskViaTaskOfInt.Result}" );
            var sumTaskViaFactory = Task.Factory.StartNew<int>(() => 
             Sum(5));
            Console.WriteLine($"Result from sumTask is 
             {sumTaskViaFactory.Result}");
            var sumTaskViaTaskRun = Task.Run<int>(() => Sum(5));
            Console.WriteLine($"Result from sumTask is 
             {sumTaskViaTaskRun.Result}");
            var sumTaskViaTaskResult = Task.FromResult<int>(Sum(5));
            Console.WriteLine($"Result from sumTask is 
             {sumTaskViaTaskResult.Result}");
        }
        private static int Sum(int n)
        {
            int sum = 0;
            for (int i = 0; i < n; i++)
            {
                sum += i;
            }
            return sum;
        }
    }
}
```

如前面的代码所示，我们使用了泛型变体创建了任务。一旦它们完成，我们就能够使用结果属性获取结果：

![](img/bc4ef585-380d-4bed-82c2-49d3ea97a72a.png)

在下一节中，我们将学习如何取消任务。

# 如何取消任务

TPL 的另一个重要功能是为开发人员提供现成的数据结构来取消运行中的任务。那些有经典线程背景的人会意识到，以前要使线程支持取消是多么困难，需要使用自定义的逻辑，但现在不再是这样。.NET Framework 提供了两个类来支持任务取消：

+   `CancellationTokenSource`**: **这个类负责创建取消令牌，并将取消请求传递给通过该源创建的所有令牌

+   `CancellationToken`**: **这个类被监听器用来监视请求的当前状态

要创建可以取消的任务，我们需要执行以下步骤：

1.  创建`System.Threading.CancellationTokenSource`类的实例，该类通过`Token Property`进一步提供`System.Threading.CancellationToken`。

1.  在创建任务时传递令牌。

1.  在需要时，调用`Cancel()`方法取消`CancellationTokenSource`上的任务。

让我们试着理解如何创建一个令牌并将其传递给任务。

# 创建令牌

可以使用以下代码创建令牌：

```cs
CancellationTokenSource tokenSource = new CancellationTokenSource();
CancellationToken token = tokenSource.Token;
```

首先，我们使用`CancellationTokenSource`构造函数创建了一个`tokenSource`。然后，我们使用**`tokenSource`**的 token 属性获取了我们的令牌。

# 使用令牌创建任务

我们可以通过将`CancellationToken`作为任务构造函数的第二个参数来创建任务，如下所示：

```cs
var sumTaskViaTaskOfInt = new Task<int>(() => Sum(5), token);
var sumTaskViaFactory = Task.Factory.StartNew<int>(() => Sum(5), token);
var sumTaskViaTaskRun = Task.Run<int>(() => Sum(5), token);
```

在经典的线程模型中，我们曾经在非确定性的线程上调用`Abort()`方法。这会突然停止线程，从而导致资源未受管理时内存泄漏。使用 TPL，我们可以调用`Cancel`方法，这是一个取消令牌源，将进而在令牌上设置`IsCancellationRequested`属性。任务执行的底层方法应该监视此属性，并且如果设置了，应该优雅地退出。

有各种方法可以监视令牌源是否请求了取消：

+   通过轮询令牌的`IsCancellationRequested`属性的状态

+   注册请求取消回调

# 通过轮询令牌的状态来检查`IsCancellationRequested`属性

这种方法在涉及递归方法或包含通过循环进行长时间计算逻辑的方法的场景中非常有用。在我们的方法或循环中，我们编写代码以在某些最佳间隔时轮询`IsCancellationRequested`。如果设置了，它通过调用`token`类的`ThrowIfCancellationRequested`方法来中断循环。

以下代码是通过轮询令牌来取消任务的示例：

```cs
        private static void CancelTaskViaPoll()
        {
            CancellationTokenSource cancellationTokenSource = 
             new CancellationTokenSource();
            CancellationToken token = cancellationTokenSource.Token;
            var sumTaskViaTaskOfInt = new Task(() => 
             LongRunningSum(token), token);
            sumTaskViaTaskOfInt.Start();
            //Wait for user to press key to cancel task
            Console.ReadLine();
            cancellationTokenSource.Cancel();
        }
        private static void LongRunningSum(CancellationToken token)
        {
            for (int i = 0; i < 1000; i++)
            {
                //Simulate long running operation
                Task.Delay(100);
                if (token.IsCancellationRequested)
                    token.ThrowIfCancellationRequested();
            }
        }
```

在前面的代码中，我们通过`CancellationTokenSource`类创建了一个取消令牌。然后，我们通过传递令牌创建了一个任务。该任务执行一个长时间运行的方法`LongRunningSum`（模拟），该方法不断轮询令牌的`IsCancellationRequested`属性。如果用户在方法完成之前调用了`cancellationTokenSource.Cancel()`，它会抛出异常。

轮询不会带来任何显著的性能开销，并且可以根据您的需求使用。当您对任务执行的工作有完全控制时使用它，例如如果它是您自己编写的核心逻辑。

# 使用回调委托注册请求取消

这种方法利用了一个`Callback`委托，当底层令牌请求取消时会被调用。我们应该将其与那些以一种使得无法以常规方式检查`CancellationToken`值的方式阻塞的操作一起使用。

让我们看一下以下代码，它从远程 URL 下载文件：

```cs
private static void DownloadFileWithoutToken()
{
    WebClient webClient = new WebClient();
    webClient.DownloadStringAsync(new 
     Uri("http://www.google.com"));
    webClient.DownloadStringCompleted += (sender, e) => 
     {
        if (!e.Cancelled)
          Console.WriteLine("Download Complete.");
        else
          Console.WriteLine("Download Cancelled.");
     };
}
```

从前面的方法中可以看到，一旦我们调用`WebClient`的`DownloadStringAsync`方法，控制权就离开了用户。虽然`WebClient`类允许我们通过`webClient.CancelAsync()`方法取消任务，但我们无法控制何时调用它。

前面的代码可以修改为使用`Callback`委托，以便更好地控制任务取消，如下所示：

```cs
static void Main(string[] args)
{
    CancellationTokenSource cancellationTokenSource = new 
     CancellationTokenSource();
    CancellationToken token = cancellationTokenSource.Token;
    DownloadFileWithToken(token);
    //Random delay before we cancel token
    Task.Delay(2000);
    cancellationTokenSource.Cancel();
    Console.ReadLine();
 }
private static void DownloadFileWithToken(CancellationToken token)
{    
    WebClient webClient = new WebClient();
    //Here we are registering callback delegate that will get called 
    //as soon as user cancels token
    token.Register(() => webClient.CancelAsync());
    webClient.DownloadStringAsync(new 
     Uri("http://www.google.com"));
    webClient.DownloadStringCompleted += (sender, e) => {
    //Wait for 3 seconds so we have enough time to cancel task
    Task.Delay(3000);
    if (!e.Cancelled)
        Console.WriteLine("Download Complete.");
    else
    Console.WriteLine("Download Cancelled.");};
}
```

如您所见，在这个修改后的版本中，我们传递了一个取消令牌，并通过`Register`方法订阅了取消回调。

一旦用户调用`cancellationTokenSource.Cancel()`方法，它将通过调用`webClient.CancelAsync()`取消下载操作。

`CancellationTokenSource`也可以与传统的`ThreadPool.QueueUserWorkItem`很好地配合使用。

以下是创建`CancellationTokenSource`的代码，可以传递给`ThreadPool`以支持取消：

```cs
// Create the token source.
CancellationTokenSource cts = new CancellationTokenSource();
// Pass the token to the cancellable operation.
ThreadPool.QueueUserWorkItem(new WaitCallback(DoSomething), cts.Token);
```

在本节中，我们讨论了取消任务的各种方法。取消任务可以在任务可能变得多余的情况下节省大量 CPU 时间。例如，假设我们创建了多个任务，使用不同的算法对一组数字进行排序。虽然所有算法都会返回相同的结果（一组排序好的数字），但我们希望尽快获得结果。我们将接受第一个（最快的）算法的结果，并取消其余的任务以提高系统性能。在下一节中，我们将讨论如何等待运行中的任务。

# 如何等待运行中的任务

在之前的示例中，我们调用了`Task.Result`属性来从已完成的任务中获取结果。这会阻塞调用线程，直到结果可用。TPL 为我们提供了另一种等待一个或多个任务的方法。

TPL 中有各种 API 可供我们等待一个或多个任务。这些包括：

+   `Task.Wait`

+   `Task.WaitAll`

+   `Task.WaitAny`

+   `Task.WhenAll`

+   `Task.WhenAny`

这些 API 将在以下子节中定义。

# Task.Wait

这是一个实例方法，用于等待单个任务。我们可以指定调用者等待任务完成的最长时间，然后在超时异常中解除阻塞。我们还可以通过向方法传递取消令牌来完全控制已取消的监视事件。调用方法将被阻塞，直到线程完成、取消或抛出异常：

```cs
var task = Task.Factory.StartNew(() => Console.WriteLine("Inside Thread"));
//Blocks the current thread until task finishes.
task.Wait();
```

`Wait`方法有五个重载版本：

+   `Wait()`:无限期地等待任务完成。调用线程将被阻塞，直到子线程完成。

+   `Wait(CancellationToken)`:等待任务无限期地执行或取消令牌被取消时。

+   `Wait(int)`:在指定的时间段内等待任务完成执行，以毫秒为单位。

+   `Wait(TimeSpan)`:在指定的时间间隔内等待任务完成执行。

+   `Wait(int, CancellationToken)`:在指定的时间段内等待任务完成执行，以毫秒为单位，或者取消令牌被取消时。

# Task.WaitAll

这是`Task`类中定义的静态方法，用于等待多个任务。任务作为数组传递给方法，调用者将被阻塞，直到所有任务完成。该方法还支持超时和取消令牌。使用此方法的一些示例代码如下：

```cs
    Task taskA = Task.Factory.StartNew(() => 
     Console.WriteLine("TaskA finished"));
    Task taskB = Task.Factory.StartNew(() => 
     Console.WriteLine("TaskB finished"));
    Task.WaitAll(taskA, taskB);
    Console.WriteLine("Calling method finishes");
```

上述代码的输出如下：

![](img/d8f72c1c-e31b-45b7-a3f7-083ea597cfa8.png)

正如您所看到的，当两个任务都完成执行时，调用方法完成语句被执行。

该方法的一个示例用例可能是当我们需要来自多个来源的数据（我们为每个来源都有一个任务），并且我们希望将所有任务的数据组合起来，以便在 UI 上显示。

# Task.WaitAny

这是`Task`类中定义的另一个静态方法。就像`WaitAll`一样，`WaitAny`用于等待多个任务，但只要传递给方法的任何任务完成执行，调用者就会解除阻塞。与其他方法一样，`WaitAny`支持超时和取消令牌。使用此方法的一些示例代码如下：

```cs
Task taskA = Task.Factory.StartNew(() => 
 Console.WriteLine("TaskA finished"));
Task taskB = Task.Factory.StartNew(() => 
 Console.WriteLine("TaskB finished"));
Task.WaitAny(taskA, taskB);
Console.WriteLine("Calling method finishes");
```

在上面的代码中，我们启动了两个任务，并使用`WaitAny`等待它们。这个方法会阻塞当前线程。一旦任何一个任务完成，调用线程就会解除阻塞。

该方法的一个示例用例可能是当我们需要的数据来自不同的来源并且我们需要尽快获取它时。在这里，我们创建了请求不同来源的任务。一旦任何一个任务完成，我们将解除调用线程的阻塞并从完成的任务中获取结果。

# Task.WhenAll

这是`WaitAll`方法的非阻塞变体。它返回一个代表所有指定任务的等待操作的任务。与阻塞调用线程的`WaitAll`不同，`WhenAll`可以在异步方法中等待，从而释放调用线程以执行其他操作。使用此方法的一些示例代码如下：

```cs
Task taskA = Task.Factory.StartNew(() => 
 Console.WriteLine("TaskA finished"));
Task taskB = Task.Factory.StartNew(() => 
 Console.WriteLine("TaskB finished"));
Task.WhenAll(taskA, taskB);
Console.WriteLine("Calling method finishes");
```

这段代码的工作方式与`Task.WaitAll`相同，除了调用线程返回到`ThreadPool`而不是被阻塞。

# Task.WhenAny

这是`WaitAny`的非阻塞变体。它返回一个封装了对单个基础任务的等待操作的任务。与`WaitAny`不同，它不会阻塞调用线程。调用线程可以在异步方法内调用 await。使用此方法的一些示例代码如下：

```cs
Task taskA = Task.Factory.StartNew(() => 
 Console.WriteLine("TaskA finished"));
Task taskB = Task.Factory.StartNew(() => 
 Console.WriteLine("TaskB finished"));
Task.WhenAny(taskA, taskB);
Console.WriteLine("Calling method finishes");
```

这段代码的工作方式与`Task.WaitAny`相同，除了调用线程返回到`ThreadPool`而不是被阻塞。

在本节中，我们讨论了如何在处理多个线程时编写高效的代码，而不需要代码分支。代码流看起来是同步的，尽管在需要的地方是并行的。在下一节中，我们将学习任务如何处理异常。

# 处理任务异常

异常处理是并行编程中最重要的方面之一。所有良好的干净代码从业者都专注于高效处理异常。这在并行编程中变得更加重要，因为线程或任务中的任何未处理异常都可能导致应用程序突然崩溃。幸运的是，TPL 提供了一个很好的、高效的设计来处理和管理异常。在任务中发生的任何未处理异常都会被延迟，然后传播到一个观察任务异常的加入线程。

任何在任务内部发生的异常都会被包装在`AggregateException`类下，并返回给观察异常的调用者。如果调用者正在等待单个任务，`AggregateException`类的内部异常属性将返回原始异常。然而，如果调用者正在等待多个任务，比如`Task.WaitAll`、`Task.WhenAll`、`Task.WaitAny`或`Task.WhenAny`，所有来自任务的异常都将作为集合返回给调用者。它们可以通过`InnerExceptions`属性访问。

现在，让我们看看在任务内部处理异常的各种方法。

# 从单个任务处理异常

在下面的代码中，我们创建了一个简单的任务，试图将一个数字除以 0，从而引发`DivideByZeroException`。异常被返回给调用者，并在 catch 块内处理。由于它是一个单一任务，异常对象被包装在`AggregateException`对象的`InnerException`属性下：

```cs
class _4HandlingExceptions
{
    static void Main(string[] args)
    {
        Task task = null;
         try
           {
                task = Task.Factory.StartNew(() =>
                {
                    int num = 0, num2 = 25;
                    var result = num2 / num;
                });
            task.Wait();
        }
        catch (AggregateException ex)
        {
            Console.WriteLine($"Task has finished with 
             exception {ex.InnerException.Message}");
        }
        Console.ReadLine();
    }
}
```

当我们运行上述代码时，输出如下：

![](img/f806dff4-80eb-4d19-88da-254dbe97be32.png)

# 从多个任务处理异常

现在，我们将创建多个任务，然后尝试从中抛出异常。然后，我们将学习如何从调用者列出来自不同任务的不同异常：

```cs
static void Main(string[] args)
{
    Task taskA = Task.Factory.StartNew(()=> throw 
     new DivideByZeroException());
    Task taskB = Task.Factory.StartNew(()=> throw 
     new ArithmeticException());
    Task taskC = Task.Factory.StartNew(()=> throw 
     new NullReferenceException());
    try
    {
        Task.WaitAll(taskA, taskB, taskC);
    }
    catch (AggregateException ex)
    {
        foreach (Exception innerException in ex.InnerExceptions)
        {
            Console.WriteLine(innerException.Message);
        }
    }
    Console.ReadLine();
}
```

当我们运行上述代码时，输出如下：

![](img/e1465881-0a19-42c5-979e-243cdf7845c6.png)

在上述代码中，我们创建了三个抛出不同异常的任务，并使用`Task.WaitAll`等待所有线程。正如你所看到的，通过调用`WaitAll`观察异常，而不仅仅是启动任务，这就是为什么我们将`WaitAll`包装在`try`块中。`WaitAll`方法将在所有传递给它的任务都通过抛出异常而故障，并执行相应的`catch`块时返回。我们可以通过迭代`AggregateException`类的`InnerExceptions`属性找到所有任务产生的异常。

# 使用回调函数处理任务异常

找出这些异常的另一个选项是使用回调函数来访问和处理来自任务的异常：

```cs
static void Main(string[] args)
      {
         Task taskA = Task.Factory.StartNew(() => throw 
          new DivideByZeroException());    
         Task taskB = Task.Factory.StartNew(() => throw 
          new ArithmeticException());                       
         Task taskC = Task.Factory.StartNew(() => throw 
          new NullReferenceException()); 
         try
         {
             Task.WaitAll(taskA, taskB, taskC);
         }
         catch (AggregateException ex)
         {
              ex.Handle(innerException =>
              {
                 Console.WriteLine(innerException.Message);
                 return true; 
              });
          }
          Console.ReadLine();
        }
```

在 Visual Studio 中运行上述代码时，输出如下：

![](img/68deb400-78ae-4c72-9467-8a6008210a12.png)

如前面的代码所示，我们订阅了`AggregateException`上的处理回调函数，而不是整合`InnerExceptions`。这对所有抛出异常的任务都会触发，我们可以返回`true`，表示异常已经得到了优雅处理。

# 将 APM 模式转换为任务

传统的 APM 方法使用`IAsyncResult`接口来创建使用两种方法设计模式的异步方法：`BeginMethodName`和`EndMethodName`。让我们试着理解程序从同步到 APM 再到任务的过程。

以下是一个从文本文件中读取数据的同步方法：

```cs
private static void ReadFileSynchronously()        
{            
    string path = @"Test.txt";
    //Open the stream and read content.
    using (FileStream fs = File.OpenRead(path))
    {
         byte[] b = new byte[1024];
         UTF8Encoding encoder = new UTF8Encoding(true);
         fs.Read(b, 0, b.Length);
         Console.WriteLine(encoder.GetString(b));
     }
 }
```

在前面的代码中没有什么花哨的。首先，我们创建了一个`FileStream`对象并调用了`Read`方法，该方法将文件从磁盘同步读入缓冲区，然后将缓冲区写入控制台。我们使用`UTF8Encoding`类将缓冲区转换为字符串。然而，这种方法的问题在于一旦调用`Read`，线程就会被阻塞，直到读取操作完成。I/O 操作由 CPU 使用 CPU 周期来管理，因此没有必要让线程等待 I/O 操作完成。让我们试着理解 APM 的做法：

```cs
private static void ReadFileUsingAPMAsyncWithoutCallback()
        {
            string filePath = @"Test.txt";
            //Open the stream and read content.
            using (FileStream fs = new FileStream(filePath, 
             FileMode.Open, FileAccess.Read, FileShare.Read, 
             1024, FileOptions.Asynchronous))
            {
                byte[] buffer = new byte[1024];
                UTF8Encoding encoder = new UTF8Encoding(true);
                IAsyncResult result = fs.BeginRead(buffer, 0, 
                 buffer.Length, null, null);
                Console.WriteLine("Do Something here");
                int numBytes = fs.EndRead(result);
                fs.Close();
                Console.WriteLine(encoder.GetString(buffer));
            }
        }
```

如前面的代码所示，我们用异步版本替换了同步的`Read`方法，即`BeginRead`。一旦编译器遇到`BeginRead`，就会向 CPU 发送指令开始读取文件，并解除线程阻塞。我们可以在同一方法中执行其他任务，然后通过调用`EndRead`再次阻塞线程，等待`Read`操作完成并收集结果。这是一个简单而有效的方法，以便制作响应式应用程序，尽管我们也在阻塞线程以获取结果。我们可以使用`Overload`而不是在同一方法中调用`EndRead`，它接受一个回调方法，当读取操作完成时会自动调用，以避免阻塞线程。该方法的签名如下：

```cs
public override IAsyncResult BeginRead(
        byte[] array,
        int offset,
        int numBytes,
        AsyncCallback userCallback,
        object stateObject)
```

在这里，我们已经看到了我们是如何从同步方法转换为 APM 的。现在，我们将把 APM 实现转换为一个任务。这在以下代码中进行了演示：

```cs
private static void ReadFileUsingTask()
        {
            string filePath = @"Test.txt";
            //Open the stream and read content.
            using (FileStream fs = new FileStream(filePath, FileMode.Open, 
             FileAccess.Read, FileShare.Read, 1024, 
             FileOptions.Asynchronous))
            {
                byte[] buffer = new byte[1024];
                UTF8Encoding encoder = new UTF8Encoding(true);
                //Start task that will read file asynchronously
                var task = Task<int>.Factory.FromAsync(fs.BeginRead, 
                 fs.EndRead, buffer, 0, buffer.Length,null);
                Console.WriteLine("Do Something while file is read 
                  asynchronously");
                //Wait for task to finish
                task.Wait();
                Console.WriteLine(encoder.GetString(buffer));
            }
        }
```

如前面的代码所示，我们用`Task<int>.Factory.FromAsync`替换了`BeginRead`方法。这是一种实现 TAP 的方法。该方法返回一个任务，在我们在同一方法中继续做其他工作的同时在后台运行，然后通过`task.Wait()`再次阻塞线程以获取结果。这就是你可以轻松地将任何 APM 代码转换为 TAP 的方法。

# 将 EAP 转换为任务

EAP 用于创建包装昂贵和耗时操作的组件。因此，它们需要被异步化。这种模式已经被用于.NET Framework 中创建诸如`BackgroundWorker`和`WebClient`等组件。

实现这种模式的方法在后台异步执行长时间运行的任务，但通过事件不断通知用户它们的进度和状态，这就是为什么它们被称为基于事件的。

以下代码显示了一个使用 EAP 的组件的实现：

```cs
  private static void EAPImplementation()
        {
            var webClient = new WebClient();
            webClient.DownloadStringCompleted += (s, e) =>
            {
                if (e.Error != null)
                    Console.WriteLine(e.Error.Message);
                else if (e.Cancelled)
                    Console.WriteLine("Download Cancel");
                else
                    Console.WriteLine(e.Result);
            };
            webClient.DownloadStringAsync(new 
             Uri("http://www.someurl.com"));
        }
```

在前面的代码中，我们订阅了`DownloadStringCompleted`事件，一旦`webClient`从 URL 下载文件，该事件就会触发。正如你所看到的，我们尝试使用 if-else 结构来读取各种结果选项，如异常、取消和结果。与 APM 相比，将 EAP 转换为 TAP 更加棘手，因为它需要对 EAP 组件的内部性质有很好的理解，因为我们需要将新代码插入到正确的事件中使其工作。让我们来看一下转换后的实现：

```cs
private static Task<string> EAPToTask()
        {
            var taskCompletionSource = new TaskCompletionSource<string>();
            var webClient = new WebClient();
            webClient.DownloadStringCompleted += (s, e) =>
            {
                if (e.Error != null)
                    taskCompletionSource.TrySetException(e.Error);
                else if (e.Cancelled)
                    taskCompletionSource.TrySetCanceled();
                else
                    taskCompletionSource.TrySetResult(e.Result);
            };
            webClient.DownloadStringAsync(new 
             Uri("http://www.someurl.com"));
            return taskCompletionSource.Task;
        }
```

将 EAP 转换为 TAP 的最简单方法是使用`TaskCompletionSource`类。我们已经插入了所有的情景，并将结果、异常或取消结果设置为`TaskCompletionSource`类的实例。然后，我们将包装的实现作为任务返回给用户。

# 更多关于任务

现在，让我们学习一些关于任务的更重要的概念，这可能会派上用场。到目前为止，我们创建的任务是独立的。然而，为了创建更复杂的解决方案，有时我们需要在任务之间定义关系。我们可以创建子任务、子任务以及继续任务来做到这一点。让我们通过例子来理解每一个。在本节的后面，我们将学习有关线程存储和队列的知识。

# 继续任务

继续任务更像是承诺。当我们需要链接多个任务时，我们可以利用它们。第二个任务在第一个任务完成时开始，并且第一个任务的结果或异常被传递给子任务。我们可以链式地创建多个任务，也可以使用 TPL 提供的方法创建选择性的继续链。TPL 提供了以下任务继续构造：

+   `Task.ContinueWith`

+   `Task.Factory.ContinueWhenAll`

+   `Task.Factory.ContinueWhenAll<T>`

+   `Task.Factory.ContinueWhenAny`

+   `Task.Factory.ContinueWhenAny<T>`

# 使用 Task.ContinueWith 方法继续任务

通过 TPL 提供的`ContinueWith`方法可以轻松实现任务的继续。

让我们通过一个例子来理解简单的链接：

```cs
var task = Task.Factory.StartNew<DataTable>(() =>
       {
           Console.WriteLine("Fetching Data");
           return FetchData();
       }).ContinueWith(
           (e) => {
               var firstRow = e.Result.Rows[0];
               Console.WriteLine("Id is {0} and Name is {0}", 
                firstRow["Id"], firstRow["Name"]);
       });
```

在上面的例子中，我们需要获取并显示数据。**主任务**调用`FetchData`方法。当它完成时，结果作为输入传递给**继续任务**，负责打印数据。输出如下：

![](img/7f9e4067-d9cf-4c5d-8f73-8e08028d1cf7.png)

我们也可以链式地创建多个任务，从而创建一系列任务，如下所示：

```cs
 var task = Task.Factory.StartNew<int>(() => GetData()).
             .ContinueWith((i) => GetMoreData(i.Result)).
             .ContinueWith((j) => DisplayData(j.Result)));
```

我们可以通过将`System.Threading.Tasks.TaskContinuationOptions`枚举作为参数传递来控制继续任务何时运行，该枚举具有以下选项：

+   `None`: 这是默认选项。当主任务完成时，继续任务将运行。

+   `OnlyOnRanToCompletion`: 当主任务成功完成时，继续任务将运行，这意味着它未被取消或出现故障。

+   `NotOnRanToCompletion`: 当主任务已被取消或出现故障时，继续任务将运行。

+   `OnlyOnFaulted`: 当主任务出现故障时，继续任务将运行。

+   `NotOnFaulted`: 当主任务未出现故障时，继续任务将运行。

+   `OnlyOnCancelled`: 当主任务已被取消时，继续任务将运行。

+   `NotOnCancelled`: 当主任务未被取消时，继续任务将运行。

# 使用 Task.Factory.ContinueWhenAll 和 Task.Factory.ContinueWhenAll<T>继续任务

我们可以等待多个任务，并链式地继续代码，只有当所有任务都成功完成时才会运行。让我们看一个例子：

```cs
 private async static void ContinueWhenAll()
        {
            int a = 2, b = 3;
            Task<int> taskA = Task.Factory.StartNew<int>(() => a * a);
            Task<int> taskB = Task.Factory.StartNew<int>(() => b * b);
            Task<int> taskC = Task.Factory.StartNew<int>(() => 2 * a * b);
            var sum = await Task.Factory.ContinueWhenAll<int>(new Task[] 
              { taskA, taskB, taskC }, (tasks)     
              =>tasks.Sum(t => (t as Task<int>).Result));
            Console.WriteLine(sum);
        }
```

在上面的代码中，我们想要计算`a*a + b*b +2 *a *b`。我们将任务分解为三个单元：`a*a`、`b*b`和`2*a*b`。每个单元由三个不同的线程执行：`taskA`、`taskB`和`taskC`。然后，我们等待所有任务完成，并将它们作为第一个参数传递给`ContinueWhenAll`方法。当所有线程完成执行时，由`ContinueWhenAll`方法的第二个参数指定的继续委托执行。继续委托对所有线程执行的结果进行求和，并将其返回给调用者，然后在下一行打印出来。

# 使用 Task.Factory.ContinueWhenAny 和 Task.Factory.ContinueWhenAny<T>继续任务

我们可以等待多个任务，并链式地继续代码，只有当任何一个任务成功完成时才会运行：

```cs
private static void ContinueWhenAny()
      {
          int number = 13;
          Task<bool> taskA = Task.Factory.StartNew<bool>(() => 
           number / 2 != 0);
          Task<bool> taskB = Task.Factory.StartNew<bool>(() => 
           (number / 2) * 2 != number);
          Task<bool> taskC = Task.Factory.StartNew<bool>(() => 
           (number & 1) != 0);
          Task.Factory.ContinueWhenAny<bool>(new Task<bool>[] 
           { taskA, taskB, taskC }, (task) =>
          {
              Console.WriteLine((task as Task<bool>).Result);
          }
        ); 
      }
```

如前面的代码所示，我们有三种不同的逻辑来判断一个数字是否为奇数。假设我们不知道哪种逻辑会最快。为了计算结果，我们创建了三个任务，每个任务封装了不同的奇数查找逻辑，并并发运行它们。由于一个数字同时可以是奇数或偶数，所有线程的结果将是相同的，但在执行速度上会有所不同。因此，只需获取第一个结果并丢弃其余结果是有意义的。这就是我们使用`ContinueWhenAny`方法实现的。

# 父任务和子任务

线程之间可能发生的另一种关系是父子关系。子任务作为父任务主体内的嵌套任务创建。子任务可以作为附加或分离创建。默认情况下，创建的任务是分离的。我们可以通过将任务的`AttachedToParent`属性设置为`true`来创建附加任务。您可能希望在以下情况之一中考虑创建附加任务：

+   所有在子任务中抛出的异常都需要传播到父任务

+   父任务的状态取决于子任务

+   父任务需要等待子任务完成

# 创建一个分离的任务

创建分离类的代码如下：

```cs
Task parentTask = Task.Factory.StartNew(() =>
 {
           Console.WriteLine(" Parent task started");
           Task childTask = Task.Factory.StartNew(() => {
               Console.WriteLine(" Child task started");
           });
           Console.WriteLine(" Parent task Finish");
       });
       //Wait for parent to finish
       parentTask.Wait();
       Console.WriteLine("Work Finished");
```

如您所见，我们在一个任务的主体内创建了另一个任务。默认情况下，子任务或嵌套任务是作为分离的创建的。我们通过调用`parentTask.Wait()`等待父任务完成。在以下输出中，您可以看到父任务不等待子任务完成，先完成，然后是子任务的开始：

![](img/3b1d53c7-a4a2-40a7-8dcd-ef1160ffd33d.png)

# 创建一个附加任务

附加任务的创建方式与分离任务类似。唯一的区别是我们将任务的`AttachedParent`属性设置为`true`。这在以下代码片段中得到了演示：

```cs
     Task parentTask = Task.Factory.StartNew(() =>
            {
                Console.WriteLine("Parent task started");
                Task childTask = Task.Factory.StartNew(() => {
                    Console.WriteLine("Child task started");
                },TaskCreationOptions.AttachedToParent);
                Console.WriteLine("Parent task Finish");
            });
            //Wait for parent to finish
            parentTask.Wait();
            Console.WriteLine("Work Finished");
```

输出如下：

![](img/3d5d2a16-4cbe-48c5-8ecd-66c57980b423.png)

在这里，您可以看到父任务直到子任务执行完成才结束。

在本节中，我们讨论了任务的高级方面，包括创建任务之间的关系。在下一节中，我们将更深入地了解任务内部的工作，理解工作队列的概念以及任务如何处理它们。

# 工作窃取队列

工作窃取是线程池的性能优化技术。每个线程池维护一个任务的全局队列，这些任务是在进程内创建的。在第一章中，*并行编程简介*，我们了解到线程池维护了一定数量的工作线程来处理任务。`ThreadPool`还维护一个线程全局队列，在这里它将所有工作项排队，然后才能分配给可用线程。由于这是一个单一队列，并且我们在多线程场景中工作，我们需要使用同步原语来实现线程安全。由于存在单一全局队列，同步会导致性能损失。

.NET Framework 通过引入本地队列的概念来解决这种性能损失，本地队列由线程管理。每个线程都可以访问全局队列，并且还维护自己的线程本地队列来存储工作项。父任务可以在全局队列中调度。当任务执行并且需要创建子任务时，它们可以堆叠在本地队列上，并且在线程执行完成后立即使用 FIFO 算法进行处理。

下图描述了全局队列、本地队列、线程和`Threadpool`之间的关系：

![](img/f8426a23-055d-4869-ba04-97bc06d88270.png)

假设主线程创建了一组任务。所有这些任务都排队到全局队列中，以便根据线程池中线程的可用性稍后执行。以下图表描述了带有所有排队任务的全局队列：

![](img/df6416be-11a8-4b46-9021-b2016fb2b9e9.png)

假设**任务 1**被安排在**线程 1**上，**任务 2**被安排在**线程 2**上，依此类推，如下图所示：

![](img/6dd8f4d1-e506-4240-9917-bf715c603c82.png)

如果**任务 1**和**任务 2**生成更多的任务，新任务将被存储在线程本地队列中，如下图所示：

![](img/d08e9c26-4b93-413a-8a96-3f403a3877a3.png)

同样，如果这些子任务创建了更多的任务，它们将进入本地队列而不是全局队列。一旦**线程 1**完成了**任务 1**，它将查看其本地队列并选择最后一个任务（LIFO）。最后一个任务可能仍然在缓存中，因此不需要重新加载。这再次提高了性能。

一旦线程（T1）耗尽了其本地队列，它将在全局队列中搜索。如果全局队列中没有项目，它将在其他线程（比如 T2）的本地队列中搜索。这种技术称为工作窃取，是一种优化技术。这次，它不会从 T2 中选择最后一个任务（LIFO），因为最后一个项目可能仍然在 T2 线程的缓存中。相反，它选择第一个任务（FIFO），因为线程已经移出了 T2 的缓存，这样可以提高性能。这种技术通过使缓存任务可用于本地线程和使缓存之外的任务可用于其他线程来提高性能。

# 总结

在本章中，我们讨论了如何将任务分解为更小的单元，以便每个单元可以由一个线程独立处理。我们还学习了利用`ThreadPool`创建任务的各种方法。我们介绍了与任务的内部工作相关的各种技术，包括工作窃取和任务创建或取消的概念。我们将在本书的其余部分利用本章中获得的知识。

在下一章中，我们将介绍数据并行性的概念。这将包括使用并行循环和处理其中的异常。
