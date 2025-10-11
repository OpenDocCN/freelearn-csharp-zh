# *第十四章*：多线程编程

在本章中，你将学习关于 **多线程编程** 的内容。你将了解线程是什么，以及后台和前台线程。然后，你将学习在运行线程之前如何将数据传递给线程。你还将学习如何暂停、中断、销毁、调度和取消线程。

在本章中，我们将涵盖以下主题：

+   **理解线程和线程化**：本节涵盖了线程的生命周期。

+   **创建带参数和不带参数的线程**：本节提供了带参数和不带参数创建线程的示例。

+   **暂停和中断线程**：本节涵盖了如何暂停和中断线程。

+   **销毁和取消线程**：本节涵盖了销毁和取消线程。

+   **调度线程**：本节涵盖了如何调度线程。

+   **线程同步和锁**：本节涵盖了如何同步线程、保护资源以及防止死锁和竞态条件。

到本章结束时，你将掌握以下技能：

+   你将理解线程和线程化。

+   你将能够创建带参数和不带参数的线程。

+   你将能够暂停和中断线程。

+   你将能够销毁和取消线程。

+   你将能够调度线程。

# 技术要求

为了确保你从本章中受益，你应该满足以下要求：

+   Visual Studio 2022

+   从以下链接获取本书的源代码：[`github.com/PacktPublishing/High-Performance-Programming-in-CSharp-and-.NET/tree/master/CH14`](https://github.com/PacktPublishing/High-Performance-Programming-in-CSharp-and-.NET/tree/master/CH14)。

# 理解线程和线程化

在本节中，我们将了解线程的生命周期。C# 中的线程具有以下生命周期：

![图 14.1 – 线程生命周期](img/B16617_Figure_14.1.jpg)

图 14.1 – 线程生命周期

当线程启动时，它们进入 `Suspend` 方法，调用 `Resume` 方法可以恢复线程。

当调用 `Monitor.Wait(object obj)` 方法时，线程进入 `wait` 状态。当调用 `Monitor.Pulse(object obj)` 方法时，等待的线程将继续，你可以通过调用 `Thread.Sleep(int millisecondsTimeout)` 方法使线程休眠。

当你调用 `Thread.Join()` 方法时，它会导致线程进入 `wait` 状态。一旦依赖的线程完成运行，等待的线程将继续。如果任何依赖的线程被取消，线程将被中止并进入 `stop` 状态。一旦线程完成或取消，就无法重新启动它。

注意

如果项目针对 .NET 5 或更高版本，并且调用任何 `Thread.Abort` API，将引发 `SYSLIB0006` 编译时警告。Microsoft 建议您使用 `CancellationToken` 来中止 `running` 单元工作。`Thread.Abort` API 现已过时。

在下一节中，我们将探讨如何创建带参数和不带参数的背景和前台线程。

# 创建线程和使用参数

在本节中，我们探讨线程的创建。首先，我们将看到如何在前台和后台创建无参数线程。让我们如下定义前台和后台线程：

+   如果`Main`方法已经完成而前台线程仍在运行，进程将保持活动状态，直到前台线程终止。

+   **后台线程**：后台线程的创建方式与前台线程相同。主要区别在于你必须明确设置线程以在后台运行。

以下代码展示了如何创建和运行一个前台线程：

```cs
var foregroundThread = new Thread(methodName);
```

```cs
foregroundThread.Start();
```

要创建并运行一个后台线程，你可以运行以下代码：

```cs
var backgroundThread = new Thread(methodName);
```

```cs
backgroundThread.IsBackground = true;
```

```cs
backgroundThread.Start();
```

你刚才看到的生成前台和后台线程的两种代码版本都没有使用参数来创建线程。以下代码展示了如何使用参数创建线程：

```cs
static void ThreadCreationWithParameters()
```

```cs
{
```

```cs
    int result = 0;
```

```cs
    Thread thread = new Thread(() => { result = Add(1, 2); );
```

```cs
    thread.Start();
```

```cs
    thread.Join();
```

```cs
    Console.WriteLine($"The addition of 1 plus 2 is 
```

```cs
        {result}." + $"");
```

```cs
}
```

```cs
static int Add(int a, int b)
```

```cs
{
```

```cs
    return a + b;
```

```cs
}
```

如前述代码所示，线程用于计算两个数字的和并返回结果。线程调用`Add`方法并将要相加的两个整数传递给它。方法调用和结果都放置在传递给线程构造函数的匿名函数中。

创建多个线程可能会对性能造成影响。可以通过使用线程池来提高多线程创建的性能。线程池通过限制应该创建和管理的线程数量来提高多线程应用程序的性能。

当使用线程池创建新线程时，它会被保留在那里，直到需要它。当需要时，线程将运行并完成其任务。一旦任务完成，线程将返回到线程池以供以后重用。

你可以按照以下方式在线程池中创建线程：

```cs
ThreadPool
```

```cs
     .QueueUserWorkItem(
```

```cs
         new WaitCallback(ThreadPoolWorkerMethod)
```

```cs
     );
```

使用线程池时要注意的是，首次使用时，它们没有历史记录，但随时间推移，它们会调整自己以提高线程池性能。对于使用大量线程并对 CPU 造成重负载的应用程序，它们可能会遇到高昂的启动成本。线程必须被创建并可供线程池使用。这可能导致线程池必须等待直到那些线程可用。在启动时可以进行的一种性能调整是设置最小线程数。以下代码展示了如何设置最小线程数：

```cs
const int WorkerThreads = 12;
```

```cs
const int CompletionPortThreads = 12;
```

```cs
ThreadPool.SetMinThreads(WorkerThreads, 
```

```cs
    CompletionPortThreads);
```

`WorkerThreads`值是`ThreadPool`按需创建的最小工作线程数。`CompletionPortThreads`值是`ThreadPool`按需创建的异步 I/O 线程数。

除了设置最小线程数之外，你还可以设置最大线程数，如下所示：

```cs
const int WorkerThreads = 12;
```

```cs
const int CompletionPortThreads = 12;
```

```cs
ThreadPool.SetMaxThreads(WorkerThreads, CompletionPortThreads);
```

为了让这些设置有助于应用程序性能，您需要正确设置它们。否则，您可能会创建过多的线程并过度调度任务。这将通过增加上下文切换来降低性能，这将增加 CPU 的负载。`ThreadPool`足够智能，一旦收集到历史数据，就会切换到一种算法，以减少 CPU 需要完成的工作量。

在设置这些值之前，使用性能监控来监控应用程序的线程使用和上下文切换是一个好主意。您可以使用上下文可视化器进行性能计数器跟踪，这将在下一章中讨论。您还可以使用`ThreadPool.GetMaxThreads`和`ThreadPool.GetMinThreads`方法来帮助您分析设置最小和最大工作线程以及完成端口线程的最佳值。

您还可以设置线程的优先级。然而，您必须非常小心地设置线程优先级，因为它可能对其他线程和其他应用程序产生负面影响。将线程设置为更高的优先级可能会导致低优先级线程饥饿，从而使其很少运行。

只有在需要快速响应事件时，例如异常，您才应考虑将线程优先级更改为高值。当遇到竞态条件时，您可以合法地降低线程的优先级。由于优先级较低而一段时间没有运行的线程最终会运行。这是因为线程的动态优先级会随着 Windows 在没有运行的情况下时间的增加而提高。

如果您确实更改了线程的优先级，那么在返回到池中时，其优先级将被重置。然而，一个线程可能用于多个任务。在这种情况下，线程将不会返回到池中，直到这些任务完成。如果优先级设置不正确，这可能会降低应用程序性能和系统性能。

我们现在已经了解了如何创建和运行线程。让我们将注意力转向暂停和中断线程。

# 暂停和中断线程

在本节中，我们将探讨暂停和中断线程。您需要暂停或中断线程的一个例子是，如果正在运行的代码是调试器。如果一个线程正在执行并且遇到断点，它需要被暂停。

暂停/延迟线程最常见的方法是调用`Thread.Sleep(millisecondsDuration)`，但这可能会冻结主线程，您的用户可能会认为您的程序已停止工作，从而导致他们终止它。

延迟线程的更好方法是让`Task.Delay(TimeSpan)`在后台运行。这将允许线程在后台工作，并防止延迟的线程停止主线程执行其工作。

以下代码显示了如何延迟线程：

```cs
static void Main(string[] args)
```

```cs
{
```

```cs
     Console.WriteLine($"Current Time: {DateTime.Now}");
```

```cs
     var delay = Task.Delay(TimeSpan.FromSeconds(5));
```

```cs
     var duration = 0;
```

```cs
     while (!delay.IsCompleted)
```

```cs
     {
```

```cs
         duration++;
```

```cs
         Thread.Sleep(TimeSpan.FromSeconds(5));
```

```cs
         Console.WriteLine($"Slept for {seconds} seconds");
```

```cs
     }
```

```cs
     Console.WriteLine($"Delay End:{DateTime.Now} after 
```

```cs
         {duration} seconds");
```

```cs
  }
```

```cs
}
```

我们创建了一个具有五秒延迟的任务。循环会一直运行，直到延迟完成。

调用`Interrupt`方法来中断处于`wait`、`sleep`或`join`阻塞状态的线程。当方法被调用时，会引发`ThreadInterruptedException`异常。当在非阻塞状态的线程上调用`Interrupt`方法时，不会引发此异常。

# 销毁和取消线程

终止线程不是一个好主意，因为您并不总是知道线程的状态。如果线程是静态构造函数的一部分，情况可能会变得更糟。使用`Thread.Abort`来终止线程是导致应用程序崩溃的主要原因之一。`Thread.Abort` API 现在已过时。因此，建议您使用协作取消模式，定期使用`CancellationToken`检查取消操作。

在正常情况下，当一个线程被终止时，它将被销毁。线程的取消也会销毁线程。让我们编写一些示例代码，演示如何使用`CancellationToken`在超时时取消同步操作，如下所示：

1.  启动一个新的.NET 6 控制台应用程序，并将其命名为 CH14_Multithreading。

1.  在`CH14_Multithreading`项目的`Program.cs`文件中，添加以下方法：

    ```cs
    static bool TryCallWithTimeout<TResult>(
          Func<CancellationToken, TResult> function,
          TimeSpan timeout,
          out TResult result
    )
    {
         var cancellationTokentSource = 
             new CancellationTokenSource(timeout);
         try
         {
             result = 
             function(cancellationTokentSource.Token);
             return true;
         }
         catch (TaskCanceledException)
         {
         }
         finally
         {
             cancellationTokentSource.Dispose();
         }
         result = default;
         return false;
    }
    ```

此方法接收一个在指定超时期间执行的方法，并返回一个结果。`SleepyMethod`被执行，但如果它超过了超时值，则引发`TaskCanceledException`异常，然后`CancellationTokenSource`被释放。

1.  按如下方式添加`SleepyMethod`代码：

    ```cs
    static int SleepyMethod(CancellationToken ct)
    {
        for (var i = 0; i < 10; i++)
        {
            Thread.Sleep(TimeSpan.FromMilliseconds(500));
            if (ct.IsCancellationRequested) { throw new 
                TaskCanceledException(); }
        }
        return 1234567890;
    }
    ```

`SleepMethod`接受`CancellationToken`作为参数。然后它循环十次。在每次迭代中，它睡眠半秒钟。然后，它检查是否已请求取消。如果已请求取消，则引发`TaskCanceledException`异常。否则，返回方法的值。

1.  按如下方式添加`SynchronousThreadCancelation`方法：

    ```cs
    static void SyncrhonousThreadCancelation()
    {
         TimeSpan timeoutTimeSpan = TimeSpan
             .FromMilliseconds(750);
         bool callResult = TryCallWithTimeout(
             SleepyMethod,
             timeoutTimeSpan,
             out int result
         );
         Console.WriteLine($"SleepyMethod() {
             (callResult ? "Executed" : "Cancelled" )
         }");
    }
    ```

此方法创建了一个持续时间为三分之四秒的超时值。然后调用`TryCallWithTimeout`方法，该方法返回一个布尔值。传递给`TryCallWithTimeout`方法的参数如下：

+   `SleepyMethod`：要执行的方法的名称

+   `timoutTimeSpan`：方法运行前要运行的时间长度，直到超时

+   `result`：包含`CancellationToken`的结果

一旦调用完成，被调用方法的名称及其调用结果将被发送到控制台。在此代码中，我们没有将结果写入控制台窗口，但您可以修改代码来实现这一点。

1.  在班级顶部，更新代码如下：

    ```cs
    SyncrhonousThreadCancelation();
    ```

前面的代码调用我们的方法，是取消同步操作的一个示例。

1.  运行前面的代码，结果应该类似于以下内容：

![图 14.2 – 程序的输出控制台，显示线程被取消]

![图 14.2 – 显示线程被取消的程序的输出控制台](img/B16617_Figure_14.2.jpg)

图 14.2 – 程序的输出控制台，显示线程被取消

这就结束了取消和销毁线程的主题。现在让我们看看线程的调度。

# 线程调度

`Thread.Start`方法安排一个`Thread`开始执行。你可以通过不同的参数重载这个方法。在本节中，我们将查看两个示例。第一个示例将调用不带任何参数的`Thread.Start()`方法，第二个示例将调用`Thread.Start(object)`。

我们现在将按照以下方式编写代码：

1.  添加一个名为`Job`的类，如下所示：

    ```cs
    internal class Job
    {
         public void Execute()
         {
             Console.WriteLine(
                 "Execute() method execute.");
         }
         public void PrintMessage(object message)
         {
             Console.WriteLine($"Message: {message}");
         }
    }
    ```

这个类提供了两个方法，这些方法将在我们的`Thread`调度示例中使用。`Execute`方法与无参数的`Thread.Start`方法一起使用，而`PrintMessage`函数与接受参数的`Thread.Start`方法一起使用。

1.  在`Program.cs`类中，添加`SheduleThreadWithoutParameters`方法如下：

    ```cs
    static void ScheduleThreadWithoutParameters()
    {
         Job job = new();
         Thread thread = 
             new Thread(new ThreadStart(job.Execute));
         thread.Start();
    }
    ```

在前面的代码中，我们创建了一个新的`Job`类实例。然后，我们创建了一个新的`Thread`，通过将其构造函数中的`new ThreadStart`实例传递给构造函数。在`ThreadStart`构造函数中，我们传递`object.method`，这是我们希望执行的，然后我们启动线程。

1.  添加`ScheduleThreadWithParameters`方法如下：

    ```cs
    static void ScheduleThreadWithParameters()
    {
         Job job = new();
         var thread1 = new Thread(
             new ParameterizedThreadStart(
                 job.PrintMessage
             )
         );
         var thread2 = new Thread(
             new ParameterizedThreadStart(
                 job.PrintMessage
             )
         );
        thread1.Start("Hello, world!");
        thread2.Start("Goodbye, world!");
    }
    ```

在前面的代码中，我们通过调用每个线程的`ParameterizedThreadStart`类来创建一个新的`Job`实例和两个线程，以便在每个线程上执行一个参数化方法。然后我们启动每个线程。

1.  在类的顶部添加对每个方法的调用，然后运行前面的代码。你的控制台应该看起来像以下这样：

![14.3 – 我们的参数化线程输出![B16617_Figure_14.3.jpg](img/B16617_Figure_14.3.jpg)

14.3 – 我们的参数化线程输出

# 线程同步和锁定

在应用程序中使用多个线程时，你必须考虑线程同步和锁定。如果不这样做，你可能会遇到竞态条件和死锁。有几种同步线程的方法。你可以使用互锁方法和同步对象，如`Monitor`、`Semaphore`和`ManualResetEvent`。

注意

在《C#中的整洁代码》一书的*第八章*《线程和并发》中，我们详细讨论了线程，包括使用线程、线程安全、使用信号量进行并行线程、线程同步和防止死锁，以及竞态条件。

为了同步你的代码，你可以使用一个锁对象，如下所示：

```cs
internal class LockMutexExample
```

```cs
{
```

```cs
public object _lockObject = new();
```

```cs
public void UsingLockObject()
```

```cs
{
```

```cs
lock(_lockObject)
```

```cs
{
```

```cs
// Perform your unsafe code here.
```

```cs
}
```

```cs
}
```

```cs
}
```

当进入被锁定的代码时，其他所有线程都被禁止访问被锁定的代码。这种方法的唯一缺点是可能会导致死锁。可以通过使用互斥锁来克服这个问题，如下所示：

```cs
internal class LockMutextExample
```

```cs
{
```

```cs
    private static readonly Mutex _mutex = new();
```

```cs
    public void UsingMutext()
```

```cs
    {
```

```cs
      try
```

```cs
      {
```

```cs
          _mutex.WaitOne();
```

```cs
          // ... Do work here ...
```

```cs
       }
```

```cs
       finally
```

```cs
       {
```

```cs
           _mutex.ReleaseMutex();
```

```cs
       }
```

```cs
    }
```

```cs
}
```

上述代码声明了一个 `Mutex` 类级变量。需要保护的代码随后被包裹在 `try/catch` 块中。当前线程通过 `WaitOne()` 方法被阻塞，直到等待句柄收到信号。当 `Mutex` 被信号时，`WaitOne()` 方法返回 `True`。然后，`Mutex` 由调用线程拥有，可以访问受保护的资源。一旦受保护的资源完成，通过调用 `ReleaseMutex()` 释放 `Mutex`。始终在最终块中调用 `ReleaseMutex()` 方法，以防止在遇到异常时资源保持锁定。

当多个线程访问同一资源并基于它们的时序产生不同结果时，会发生竞态条件。可以通过使用如下代码来避免竞态条件：

```cs
Task
```

```cs
    .Run(() => Method1())
```

```cs
    .ContinueWith(task => Method2())
```

```cs
    .Wait();
```

`Task` 在运行 `Method1()` 后继续执行 `Method2()`。然后我们使用 `Wait()` 等待 `Task` 完成 `Method1()` 和 `Method2()` 的执行，再继续。

这就结束了我们对多线程编程的探讨。正如你所见，线程调度并没有太多内容。让我们总结一下本章我们学到了什么。

# 摘要

在本章中，我们了解了线程和线程生命周期。我们构建了一些示例代码，展示了如何带参数和不带参数创建线程。我们还探讨了在前台和后台运行线程。

接下来，我们探讨了暂停和中断线程。然后，我们转向了销毁和取消线程。你不再在代码中使用 `Thread.Abort`。`Thread.Abort` 导致应用程序在运行时崩溃。相反，你使用取消令牌。取消线程也会销毁它们。

我们探讨了带参数和不带参数的线程调度。在下一章中，我们将探讨并行编程。

最后，我们探讨了使用锁对象和互斥锁进行线程同步和锁定，并学习了如何避免死锁和竞态条件。

现在是时候回答一些问题，以检验你对本章知识的掌握程度。一旦你完成了这些问题，*进一步阅读*部分提供了一些外部资源，以进一步扩展你对线程和多线程编程的知识。

# 问题

1.  线程可以处于哪些状态？

1.  你在 `Thread.Abort` API 的哪个部分使用来终止线程？

1.  线程可以在哪两个位置执行？

1.  正确终止线程的方法是什么？

1.  使用哪种方法来调度线程？

# 进一步阅读

+   管理和实现多线程：[`subscription.packtpub.com/book/programming/9781789536577/6/ch06lvl1sec52/understanding-threads-and-the-threading-process`](https://subscription.packtpub.com/book/programming/9781789536577/6/ch06lvl1sec52/understanding-threads-and-the-threading-process)

)

+   暂停和中断线程：[`docs.microsoft.com/en-us/dotnet/standard/threading/pausing-and-resuming-threads`](https://docs.microsoft.com/en-us/dotnet/standard/threading/pausing-and-resuming-threads)

)

+   如何在 C#中终止线程：[`www.geeksforgeeks.org/how-to-terminate-a-thread-in-c-sharp/`](https://www.geeksforgeeks.org/how-to-terminate-a-thread-in-c-sharp/)

)

+   如何在 C#中销毁线程：[`www.tutorialspoint.com/How-to-destroy-threads-in-Chash`](https://www.tutorialspoint.com/How-to-destroy-threads-in-Chash)

+   如何在 C#中安排线程的执行：[`www.geeksforgeeks.org/how-to-schedule-a-thread-for-execution-in-c-sharp/#:~:text=%20How%20to%20schedule%20a%20thread%20for%20execution,1%20Start%20%28%29%202%20Start%20%28Object%29%20More%20`](https://www.geeksforgeeks.org/how-to-schedule-a-thread-for-execution-in-c-sharp/#:~:text=%20How%20to%20schedule%20a%20thread%20for%20execution,1%20Start%20%28%29%202%20Start%20%28Object%29%20More%20)

+   理解线程和线程过程：[`subscription.packtpub.com/book/programming/9781789536577/6/ch06lvl1sec52/understanding-threads-and-the-threading-process`](https://subscription.packtpub.com/book/programming/9781789536577/6/ch06lvl1sec52/understanding-threads-and-the-threading-process)

)

+   如何在 C#中暂停代码执行：[`csharpsage.com/c-delay/`](https://csharpsage.com/c-delay/)

+   暂停和中断线程：[`docs.microsoft.com/en-us/dotnet/standard/threading/pausing-and-resuming-threads`](https://docs.microsoft.com/en-us/dotnet/standard/threading/pausing-and-resuming-threads)
