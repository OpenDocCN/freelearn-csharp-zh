# 第三章：使用线程池

在本章中，我们将描述用于从多个线程中处理共享资源的常用技术。你将学习以下配方：

+   在线程池上调用委托

+   在线程池上发布异步操作

+   线程池和并行度

+   实现取消选项

+   使用线程池的等待句柄和超时

+   使用计时器

+   使用`BackgroundWorker`组件

# 简介

在前面的章节中，我们讨论了几种创建线程和组织它们合作的方法。现在，让我们考虑另一种场景，我们将创建许多耗时很小的异步操作。正如我们在第一章的*简介*部分所讨论的，创建线程是一个昂贵的操作，因此对每个短暂、异步操作都这样做将包括显著的开销。

为了解决这个问题，有一个常见的称为**池化**的方法，可以成功应用于任何我们需要许多短暂、昂贵资源的情况。我们预先分配一定数量的这些资源，并将它们组织成一个资源池。每次我们需要新的资源时，我们只需从池中取出，而不是创建一个新的，在资源不再需要时将其返回到池中。

**.NET 线程池**是这个概念的实现。它可以通过`System.Threading.ThreadPool`类型访问。线程池由.NET **公共** **语言运行时**（**CLR**）管理，这意味着每个 CLR 有一个线程池实例。`ThreadPool`类型有一个`QueueUserWorkItem`静态方法，它接受一个**委托**，代表用户定义的异步操作。在调用此方法后，这个委托进入内部队列。然后，如果没有线程在池中，它将创建一个新的**工作线程**并将第一个委托放入队列中。

如果我们将新的操作放入线程池，在之前的操作完成后，可以重新使用这个线程来执行这些操作。然而，如果我们更快地放入新的操作，线程池将创建更多的线程来服务这些操作。存在一个限制以防止创建过多的线程，在这种情况下，新的操作将等待在队列中，直到池中的工作线程空闲出来服务它们。

### 注意

保持线程池上的操作短暂是非常重要的！不要将长时间运行的操作放入线程池或阻塞工作线程。这将导致所有工作线程变得忙碌，它们将无法再服务用户操作。这反过来会导致性能问题和难以调试的错误。

当我们停止向线程池添加新操作时，它最终会在一段时间空闲后移除不再需要的线程。这将释放不再需要的任何操作系统资源。

我再次强调，线程池旨在执行短运行时操作。使用线程池可以在减少并行度的代价下节省操作系统资源。我们使用更少的线程，但执行异步操作的速度比通常慢，通过可用的工作线程数量进行批处理。如果操作完成得很快，这样做是有意义的，但如果执行许多长时间运行的计算密集型操作，这将降低性能。

另一个非常重要的事情是要非常小心地在 ASP.NET 应用程序中使用线程池。ASP.NET 基础设施本身使用线程池，如果你浪费了线程池中的所有工作线程，那么 Web 服务器将无法再处理传入的请求。建议你在 ASP.NET 中仅使用输入/输出绑定的异步操作，因为它们使用不同的机制称为 **I/O 线程**。我们将在 第九章 中讨论 I/O 线程，*使用异步 I/O*。

### 注意

注意，线程池中的工作线程是后台线程。这意味着当所有前台线程（包括主应用程序线程）完成时，所有后台线程都将停止。

在本章中，你将学习如何使用线程池来执行异步操作。我们将介绍将操作放入线程池的不同方法，以及如何取消操作并防止其长时间运行。

# 在线程池上调用委托

本食谱将向你展示如何在线程池上异步执行委托。此外，我们还将讨论一种称为 **异步编程模型** （**APM**）的方法，这是历史上 .NET 中的第一个异步编程模式。

## 准备工作

要进入这个食谱，你需要 Visual Studio 2015。没有其他先决条件。本食谱的源代码可以在 `BookSamples\Chapter3\Recipe1` 中找到。

## 如何做...

要了解如何在线程池上调用委托，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    private delegate string RunOnThreadPool(out int threadId);

    private static void Callback(IAsyncResult ar)
    {
      WriteLine("Starting a callback...");
      WriteLine($"State passed to a callbak: {ar.AsyncState}");
      WriteLine($"Is thread pool thread: {CurrentThread.IsThreadPoolThread}");
      WriteLine($"Thread pool worker thread id: {CurrentThread.ManagedThreadId}");
    }

    private static string Test(out int threadId)
    {
      WriteLine("Starting...");
      WriteLine($"Is thread pool thread: {CurrentThread.IsThreadPoolThread}");
      Sleep(TimeSpan.FromSeconds(2));
      threadId = CurrentThread.ManagedThreadId;
      return $"Thread pool worker thread id was: {threadId}";
    }
    ```

1.  在 `Main` 方法内添加以下代码：

    ```cs
    int threadId = 0;

    RunOnThreadPool poolDelegate = Test;

    var t = new Thread(() => Test(out threadId));
    t.Start();
    t.Join();

    WriteLine($"Thread id: {threadId}");

    IAsyncResult r = poolDelegate.BeginInvoke(out threadId, Callback, "a delegate asynchronous call");
    r.AsyncWaitHandle.WaitOne();

    string result = poolDelegate.EndInvoke(out threadId, r);

    WriteLine($"Thread pool worker thread id: {threadId}");
    WriteLine(result);

    Sleep(TimeSpan.FromSeconds(2));
    ```

1.  运行程序。

## 工作原理...

当程序运行时，它以传统的方式创建一个线程，然后启动它并等待其完成。由于线程构造函数只接受不返回任何结果的函数，我们使用**lambda 表达式**来包装对`Test`方法的调用。我们确保这个线程不是来自线程池，通过打印出`Thread.CurrentThread.IsThreadPoolThread`属性值。我们还打印出一个托管线程 ID，以识别执行此代码的线程。

然后，我们定义一个委托并通过调用`BeginInvoke`方法来运行它。此方法接受一个回调，该回调将在异步操作完成后被调用，以及一个用户定义的状态，该状态将被传递到回调中。这个状态通常用于区分一个异步调用与另一个异步调用。因此，我们得到一个实现`IAsyncResult`接口的`result`对象。`BeginInvoke`方法立即返回结果，允许我们在异步操作在线程池的工作线程上执行时继续进行任何工作。当我们需要异步操作的结果时，我们使用从`BeginInvoke`方法调用返回的`result`对象。我们可以使用`IsCompleted`结果属性对其进行轮询，但在这个情况下，我们使用`AsyncWaitHandle`结果属性来等待它，直到操作完成。完成这些后，为了从它获取结果，我们在委托上调用`EndInvoke`方法，传递委托参数和我们的`IAsyncResult`对象。

### 注意

实际上，使用`AsyncWaitHandle`是不必要的。如果我们注释掉`r.AsyncWaitHandle.WaitOne`，代码仍然可以成功运行，因为`EndInvoke`方法实际上会等待异步操作完成。始终调用`EndInvoke`（或其他异步 API 的`EndOperationName`）总是很重要的，因为它会将任何未处理的异常抛回到调用线程。始终在使用此类异步 API 时调用`Begin`和`End`方法。

当操作完成时，传递给`BeginInvoke`方法的回调将被发布到线程池中，更具体地说，是一个工作线程。如果我们注释掉`Main`方法定义末尾的`Thread.Sleep`方法调用，回调将不会执行。这是因为当主线程完成时，所有后台线程都将停止，包括这个回调。可能异步调用委托和回调将由同一个工作线程服务，这可以通过工作线程 ID 很容易地看到。

在 .NET 中，使用`BeginOperationName`/`EndOperationName`方法和`IAsyncResult`对象的方法称为异步编程模型或 APM 模式，这样的方法对称为异步方法。这种模式仍在各种 .NET 类库 API 中使用，但在现代编程中，更倾向于使用**任务并行库**（**TPL**）来组织异步 API。我们将在第四章 *使用任务并行库* 中介绍这个主题。

# 在线程池上发布异步操作

本食谱将描述如何将异步操作放在线程池上。

## 准备工作

要进入这个食谱，你需要 Visual Studio 2015。没有其他先决条件。这个食谱的源代码可以在`BookSamples\Chapter3\Recipe2`中找到。

## 如何做...

要了解如何在线程池上发布异步操作，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    private static void AsyncOperation(object state)
    {
      WriteLine($"Operation state: {state ?? "(null)"}");
      WriteLine($"Worker thread id: {CurrentThread.ManagedThreadId}");
      Sleep(TimeSpan.FromSeconds(2));
    }
    ```

1.  在`Main`方法内部添加以下代码片段：

    ```cs
    const int x = 1;
    const int y = 2;
    const string lambdaState = "lambda state 2";

    ThreadPool.QueueUserWorkItem(AsyncOperation);
    Sleep(TimeSpan.FromSeconds(1));

    ThreadPool.QueueUserWorkItem(AsyncOperation, "async state");
    Sleep(TimeSpan.FromSeconds(1));

    ThreadPool.QueueUserWorkItem( state => {
      WriteLine($"Operation state: {state}");
      WriteLine($"Worker thread id: {CurrentThread.ManagedThreadId}");
      Sleep(TimeSpan.FromSeconds(2));
    }, "lambda state");

    ThreadPool.QueueUserWorkItem( _ =>
    {
      WriteLine($"Operation state: {x + y}, {lambdaState}");
      WriteLine($"Worker thread id: {CurrentThread.ManagedThreadId}");
      Sleep(TimeSpan.FromSeconds(2));
    }, "lambda state");

    Sleep(TimeSpan.FromSeconds(2));
    ```

1.  运行程序。

## 它是如何工作的...

首先，我们定义一个接受单个`object`类型参数的`AsyncOperation`方法。然后，我们使用`QueueUserWorkItem`方法将此方法发布到线程池。然后，我们再次发布此方法，但这次，我们向此方法调用传递一个`state`对象。此对象将被传递给`AsynchronousOperation`方法作为`state`参数。

在这些操作之后让线程休眠 1 秒，这允许线程池重用线程来执行新的操作。如果你注释掉这些`Thread.Sleep`调用，大多数情况下线程 ID 都会不同。如果不是，可能前两个线程将被重用来执行接下来的两个操作。

首先，我们将一个 lambda 表达式发布到线程池。这里没有特别之处；我们不是定义一个单独的方法，而是使用 lambda 表达式语法。

其次，我们不是传递 lambda 表达式的状态，而是使用**闭包**机制。这为我们提供了更多的灵活性，并允许我们向异步操作提供多个对象，并为这些对象提供静态类型。因此，将对象传递到方法回调的先前机制实际上是多余的，已经过时。现在我们有了 C# 中的闭包，就没有必要使用它了。

# 线程池和并行度

本食谱将展示线程池如何与许多异步操作一起工作，以及它与创建多个单独线程的不同之处。

## 准备工作

要进入这个食谱，你需要 Visual Studio 2015。没有其他先决条件。这个食谱的源代码可以在`BookSamples\Chapter3\Recipe3`中找到。

## 如何做...

要了解线程池如何与许多异步操作一起工作，以及它与创建许多单独线程的不同之处，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Diagnostics;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static void UseThreads(int numberOfOperations)
    {
      using (var countdown = new CountdownEvent(numberOfOperations))
      {
        WriteLine("Scheduling work by creating threads");
        for (int i = 0; i < numberOfOperations; i++)
        {
          var thread = new Thread(() =>
          {
            Write($"{CurrentThread.ManagedThreadId},");
            Sleep(TimeSpan.FromSeconds(0.1));
            countdown.Signal();
          });
          thread.Start();
        }
        countdown.Wait();
        WriteLine();
      }
    }

    static void UseThreadPool(int numberOfOperations)
    {
      using (var countdown = new CountdownEvent(numberOfOperations))
      {
        WriteLine("Starting work on a threadpool");
        for (int i = 0; i < numberOfOperations; i++)
        {
          ThreadPool.QueueUserWorkItem( _ => 
          {
            Write($"{CurrentThread.ManagedThreadId},");
            Sleep(TimeSpan.FromSeconds(0.1));
            countdown.Signal();
          });
        }
        countdown.Wait();
        WriteLine();
      }
    }
    ```

1.  在`Main`方法内部添加以下代码片段：

    ```cs
    const int numberOfOperations = 500;
    var sw = new Stopwatch();
    sw.Start();
    UseThreads(numberOfOperations);
    sw.Stop();
    WriteLine($"Execution time using threads: {sw.ElapsedMilliseconds}");

    sw.Reset();
    sw.Start();
    UseThreadPool(numberOfOperations);
    sw.Stop();
    WriteLine($"Execution time using the thread pool: {sw.ElapsedMilliseconds}");
    ```

1.  运行程序。

## 它是如何工作的...

当主程序启动时，我们创建了多个不同的线程，并在每个线程上运行一个操作。这个操作打印出线程 ID 并阻塞线程 100 毫秒。结果，我们创建了 500 个线程，并行运行所有这些操作。在我的机器上，总时间大约是 300 毫秒，但我们消耗了大量的操作系统资源。

然后，我们遵循相同的流程，但不是为每个操作创建一个线程，而是将它们发布到线程池。之后，线程池开始服务这些操作；它开始在接近结束时创建更多线程；然而，它仍然需要更多的时间，在我的机器上大约是 12 秒。我们为操作系统使用节省了内存和线程，但以应用程序性能为代价。

# 实现取消选项

这个配方展示了如何在一个线程池上取消异步操作的示例。

## 准备工作

要进入这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter3\Recipe4`中找到。

## 如何做到这一点...

要了解如何在线程上实现取消选项，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static void AsyncOperation1(CancellationToken token)
    {
      WriteLine("Starting the first task");
      for (int i = 0; i < 5; i++)
      {
        if (token.IsCancellationRequested)
        {
          WriteLine("The first task has been canceled.");
          return;
        }
        Sleep(TimeSpan.FromSeconds(1));
      }
      WriteLine("The first task has completed succesfully");
    }

    static void AsyncOperation2(CancellationToken token)
    {
      try
      {
        WriteLine("Starting the second task");

        for (int i = 0; i < 5; i++)
        {
          token.ThrowIfCancellationRequested();
          Sleep(TimeSpan.FromSeconds(1));
        }
        WriteLine("The second task has completed succesfully");
      }
      catch (OperationCanceledException)
      {
        WriteLine("The second task has been canceled.");
      }
    }

    static void AsyncOperation3(CancellationToken token)
    {
      bool cancellationFlag = false;
      token.Register(() => cancellationFlag = true);
      WriteLine("Starting the third task");
      for (int i = 0; i < 5; i++)
      {
        if (cancellationFlag)
        {
          WriteLine("The third task has been canceled.");
          return;
        }
        Sleep(TimeSpan.FromSeconds(1));
      }
      WriteLine("The third task has completed succesfully");
    }
    ```

1.  在`Main`方法内部添加以下代码片段：

    ```cs
    using (var cts = new CancellationTokenSource())
    {
      CancellationToken token = cts.Token;
      ThreadPool.QueueUserWorkItem(_ => AsyncOperation1(token));
      Sleep(TimeSpan.FromSeconds(2));
      cts.Cancel();
    }

    using (var cts = new CancellationTokenSource())
    {
      CancellationToken token = cts.Token;
      ThreadPool.QueueUserWorkItem(_ => AsyncOperation2(token));
      Sleep(TimeSpan.FromSeconds(2));
      cts.Cancel();
    }

    using (var cts = new CancellationTokenSource())
    {
      CancellationToken token = cts.Token;
      ThreadPool.QueueUserWorkItem(_ => AsyncOperation3(token));
      Sleep(TimeSpan.FromSeconds(2));
      cts.Cancel();
    }

    Sleep(TimeSpan.FromSeconds(2));
    ```

1.  运行程序。

## 它是如何工作的...

在这里，我们介绍了`CancellationTokenSource`和`CancellationToken`构造。它们出现在.NET 4.0 中，现在已成为实现异步操作取消过程的实际标准。由于线程池已经存在很长时间，它没有为取消令牌提供特殊的 API；然而，它们仍然可以使用。

在这个程序中，我们看到有三种组织取消过程的方法。第一种只是轮询并检查`CancellationToken.IsCancellationRequested`属性。如果它被设置为`true`，这意味着我们的操作正在被取消，我们必须放弃操作。

第二种方式是抛出`OperationCancelledException`异常。这允许我们从被取消操作的外部代码中控制取消过程。

最后一种选项是在线程池中注册一个**回调**，当操作被取消时将被调用。这将允许我们将取消逻辑链入另一个异步操作。

# 使用线程池的等待句柄和超时

本食谱将描述如何实现线程池操作的超时以及如何在线程池上正确等待。

## 准备工作

要进入这个食谱，你需要 Visual Studio 2015。没有其他先决条件。本食谱的源代码可以在 `BookSamples\Chapter3\Recipe5` 中找到。

## 如何操作...

要了解如何实现超时以及如何在线程池上正确等待，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在 `Program.cs` 文件中添加以下 `using` 指令：

    ```cs
    using System;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    static void RunOperations(TimeSpan workerOperationTimeout)
    {
      using (var evt = new ManualResetEvent(false))
      using (var cts = new CancellationTokenSource())
      {
        WriteLine("Registering timeout operation...");
        var worker = ThreadPool.RegisterWaitForSingleObject(evt
                    , (state, isTimedOut) => WorkerOperationWait(cts, isTimedOut)
                    , null
                    , workerOperationTimeout
                    , true);

        WriteLine("Starting long running operation...");
        ThreadPool.QueueUserWorkItem(_ => WorkerOperation(cts.Token, evt));

        Sleep(workerOperationTimeout.Add(TimeSpan.FromSeconds(2)));
        worker.Unregister(evt);
      }
    }

    static void WorkerOperation(CancellationToken token, ManualResetEvent evt)
    {
      for(int i = 0; i < 6; i++)
      {
        if (token.IsCancellationRequested)
        {
          return;
        }
        Sleep(TimeSpan.FromSeconds(1));
      }
      evt.Set();
    }

    static void WorkerOperationWait(CancellationTokenSource cts, bool isTimedOut)
    {
      if (isTimedOut)
      {
        cts.Cancel();
        WriteLine("Worker operation timed out and was canceled.");
      }
      else
      {
        WriteLine("Worker operation succeded.");
      }
    }
    ```

1.  在 `Main` 方法内部添加以下代码片段：

    ```cs
    RunOperations(TimeSpan.FromSeconds(5));
    RunOperations(TimeSpan.FromSeconds(7));
    ```

1.  运行程序。

## 它是如何工作的...

线程池还有一个有用的方法：`ThreadPool.RegisterWaitForSingleObject`。此方法允许我们在线程池上排队一个回调，并且当提供的等待句柄被信号或发生超时时，此回调将被执行。这允许我们为线程池操作实现超时。

首先，我们注册超时处理异步操作。当以下事件之一发生时，它将被调用：在接收到 `ManualResetEvent` 对象的信号时，该对象由工作操作在成功完成时设置，或者当第一个操作完成之前发生超时时。如果发生这种情况，我们使用 `CancellationToken` 来取消第一个操作。

然后，我们在线程池上排队一个长时间运行的工作操作。它运行 6 秒，然后设置一个 `ManualResetEvent` 信号构造函数，以防它成功完成。在其他情况下，如果请求取消，操作将被放弃。

最后，如果我们为操作提供 5 秒的超时，这还不够。这是因为操作需要 6 秒才能完成，我们需要取消这个操作。所以，如果我们提供一个 7 秒的超时，这是可以接受的，操作将成功完成。

## 还有更多...

当你有大量线程必须等待在 `blocked` 状态，等待某些多线程事件构造函数发出信号时，这非常有用。我们不需要阻塞所有这些线程，我们可以使用线程池基础设施。这将允许我们在事件设置之前释放这些线程。这对于需要可扩展性和性能的服务器应用程序来说是一个非常重要的场景。

# 使用计时器

本食谱将描述如何使用 `System.Threading.Timer` 对象在线程池上创建周期性调用的异步操作。

## 准备工作

要进入这个食谱，你需要 Visual Studio 2015。没有其他先决条件。本食谱的源代码可以在 `BookSamples\Chapter3\Recipe6` 中找到。

## 如何操作...

要了解如何在线程池上创建周期性调用的异步操作，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    static Timer _timer;

    static void TimerOperation(DateTime start)
    {
      TimeSpan elapsed = DateTime.Now - start;
      WriteLine($"{elapsed.Seconds} seconds from {start}. " +
        $"Timer thread pool thread id: {CurrentThread.ManagedThreadId}");
    }
    ```

1.  在 `Main` 方法内部添加以下代码片段：

    ```cs
    WriteLine("Press 'Enter' to stop the timer...");
    DateTime start = DateTime.Now;
    _timer = new Timer(_ => TimerOperation(start), null
        , TimeSpan.FromSeconds(1)
        , TimeSpan.FromSeconds(2));
    try
    {
      Sleep(TimeSpan.FromSeconds(6));

      _timer.Change(TimeSpan.FromSeconds(1), TimeSpan.FromSeconds(4));

      ReadLine();
    }
    finally
    {
      _timer.Dispose();
    }
    ```

1.  运行程序。

## 它是如何工作的...

首先，我们创建一个新的 `Timer` 实例。第一个参数是一个将在线程池上执行的表达式。我们调用 `TimerOperation` 方法，并给它提供一个开始日期。我们不使用用户 `state` 对象，因此第二个参数为 null；然后，我们指定何时第一次运行 `TimerOperation` 以及调用之间的周期。所以，第一个值实际上意味着我们将在 1 秒后开始第一次操作，然后，我们每 2 秒运行一次。

之后，我们等待 6 秒并更改我们的计时器。在调用 `_timer.Change` 方法后 1 秒启动 `TimerOperation`，然后每个操作运行 4 秒。

### 提示

**计时器可能比这更复杂！**

可以以更复杂的方式使用计时器。例如，我们可以通过提供一个具有 `Timeout.Infinite` 值的计时器周期参数来仅运行计时器操作一次。然后，在计时器异步操作内部，我们可以根据某些自定义逻辑设置计时器操作下一次执行的时间。

最后，我们等待按下 *Enter* 键并结束应用程序。在它运行时，我们可以看到自程序开始以来经过的时间。

# 使用 BackgroundWorker 组件

本食谱通过一个 `BackgroundWorker` 组件的示例描述了异步编程的另一种方法。借助此对象，我们可以将异步代码组织成一系列事件和事件处理器。您将学习如何使用此组件进行异步编程。

## 准备工作

要进入此食谱，您需要 Visual Studio 2015。没有其他先决条件。此食谱的源代码可在 `BookSamples\Chapter3\Recipe7` 中找到。

## 如何做...

要学习如何使用 `BackgroundWorker` 组件，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.ComponentModel;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    static void Worker_DoWork(object sender, DoWorkEventArgs e)
    {
      WriteLine($"DoWork thread pool thread id: {CurrentThread.ManagedThreadId}");
      var bw = (BackgroundWorker) sender;
      for (int i = 1; i <= 100; i++)
      {
        if (bw.CancellationPending)
        {
          e.Cancel = true;
          return;
        }
        if (i%10 == 0)
        {
          bw.ReportProgress(i);
        }

        Sleep(TimeSpan.FromSeconds(0.1));
      }

      e.Result = 42;
    }

    static void Worker_ProgressChanged(object sender, ProgressChangedEventArgs e)
    {
      WriteLine($"{e.ProgressPercentage}% completed. " +
        $"Progress thread pool thread id: {CurrentThread.ManagedThreadId}");
    }

    static void Worker_Completed(object sender, RunWorkerCompletedEventArgs e)
    {
      WriteLine($"Completed thread pool thread id: {CurrentThread.ManagedThreadId}");
      if (e.Error != null)
      {
        WriteLine($"Exception {e.Error.Message} has occured.");
      }
      else if (e.Cancelled)
      {
        WriteLine($"Operation has been canceled.");
      }
      else
      {
        WriteLine($"The answer is: {e.Result}");
      }
    }
    ```

1.  在 `Main` 方法内部添加以下代码片段：

    ```cs
    var bw = new BackgroundWorker();
    bw.WorkerReportsProgress = true;
    bw.WorkerSupportsCancellation = true;

    bw.DoWork += Worker_DoWork;
    bw.ProgressChanged += Worker_ProgressChanged;
    bw.RunWorkerCompleted += Worker_Completed;

    bw.RunWorkerAsync();

    WriteLine("Press C to cancel work");
    do
    {
      if (ReadKey(true).KeyChar == 'C')
      {
        bw.CancelAsync();
      }
    }
    while(bw.IsBusy);
    ```

1.  运行程序。

## 它是如何工作的...

当程序启动时，我们创建一个 `BackgroundWorker` 组件的实例。我们明确表示我们希望我们的后台工作线程支持取消操作和操作进度的通知。

现在，最有趣的部分来了。我们不是使用线程池和委托来操作，而是使用另一种 C#习语，称为**事件**。事件代表一个**通知源**和多个**订阅者**，当通知到达时，它们准备做出反应。在我们的情况下，我们声明我们将订阅三个事件，当它们发生时，我们调用相应的**事件处理器**。这些是具有特别定义签名的函数，当事件通知其订阅者时将被调用。

因此，我们不是在`Begin`/`End`方法对中组织异步 API，而是可以直接启动异步操作，然后订阅在操作执行期间可能发生的事件。这种方法被称为**基于事件的异步模式**（**EAP**）。这在历史上是第二次尝试结构化异步程序，而现在，推荐使用 TPL（Task Parallel Library），这将在第四章中描述，*使用任务并行库*。

因此，我们订阅了三个事件。其中第一个是`DoWork`事件。当后台工作对象使用`RunWorkerAsync`方法启动异步操作时，将调用此事件的处理器。事件处理器将在线程池上执行，这是工作取消请求时取消工作以及我们提供操作进度信息的主要操作点。最后，当我们得到结果时，我们将它设置到事件参数中，然后调用`RunWorkerCompleted`事件处理器。在这个方法内部，我们找出我们的操作是否成功，是否有错误发生，或者是否被取消。

此外，`BackgroundWorker`组件实际上旨在用于**Windows Forms** **应用程序**（**WPF**）。其实现使得从后台工作的事件处理器代码中直接操作 UI 控件成为可能，与线程池中的工作线程与 UI 控件交互相比，这非常方便。
