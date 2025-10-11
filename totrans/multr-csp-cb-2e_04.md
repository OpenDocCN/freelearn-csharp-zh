# 第四章 使用任务并行库

在本章中，我们将深入研究一种新的异步编程范式，即任务并行库。您将学习以下技巧：

+   创建任务

+   使用任务执行基本操作

+   组合任务

+   将 APM 模式转换为任务

+   将 EAP 模式转换为任务

+   实现取消选项

+   在任务中处理异常

+   并行运行任务

+   使用`TaskScheduler`调整任务的执行

# 简介

在前面的章节中，您学习了什么是线程，如何使用线程，以及为什么我们需要线程池。使用线程池可以在减少并行度度的代价下节省操作系统资源。我们可以将线程池视为一个**抽象层**，它隐藏了线程使用的细节，使我们能够专注于程序的逻辑，而不是线程问题。

然而，使用线程池也很复杂。没有简单的方法可以从线程池工作线程中获取结果。我们需要实现自己的方法来获取结果，并在发生异常的情况下，必须正确地将异常传播到原始线程。除此之外，没有简单的方法来创建一系列依赖的异步操作，其中一个操作在另一个操作完成其工作后运行。

为了解决这些问题，已经尝试了多种方法，这导致了异步编程模型和基于事件的异步模式（EAP）的创建，这些模式在第三章中提到，*使用线程池*。这些模式使得获取结果更容易，并且在传播异常方面做得很好，但组合异步操作仍然需要大量工作，并导致大量代码的产生。

为了解决所有这些问题，.Net Framework 4.0 中引入了一个新的异步操作 API。它被称为任务并行库（TPL）。在.Net Framework 4.5 中它有所改变，为了明确起见，我们将使用项目中.Net Framework 4.6 的最新版本来工作。TPL 可以被视为在线程池之上的另一个抽象层，它隐藏了与线程池工作的底层代码，并为程序员提供了一个更方便和更细粒度的 API。

TPL 的核心概念是任务。任务表示一种异步操作，它可以以多种方式运行，使用单独的线程或不使用。我们将在本章中详细探讨所有可能性。

### 注意

默认情况下，程序员并不了解任务是如何被具体执行的。TPL 通过隐藏任务实现的细节来提高抽象级别。不幸的是，在某些情况下，这可能导致神秘的错误，例如在尝试从任务获取结果时应用程序挂起。本章将帮助您理解 TPL 底层的机制以及如何避免以不恰当的方式使用它。

任务可以与其他任务以不同的变体组合。例如，我们能够同时启动几个任务，等待它们全部完成，然后运行一个任务，该任务将对所有先前任务的输出执行一些计算。方便的任务组合 API 是 TPL 相比于先前模式的关键优势之一。

还有几种方法可以处理任务产生的异常。由于一个任务可能由几个其他任务组成，而这些任务又可能有它们自己的子任务，因此存在 `AggregateException` 的概念。此类异常包含其内部所有底层任务的异常，允许我们分别处理它们。

最后但同样重要的是，从版本 5.0 开始，C# 内置了对 TPL 的支持，允许我们使用新的 `await` 和 `async` 关键字以非常顺畅和舒适的方式与任务一起工作。我们将在 第五章 中讨论此主题，*使用 C# 6.0*。

在本章中，您将学习如何使用 TPL 执行异步操作。我们将学习任务是什么，介绍创建任务的不同方法，并学习如何组合任务。我们还将讨论如何将遗留的 APM 和 EAP 模式转换为使用任务，如何正确处理异常，如何取消任务，以及如何处理同时执行的任务。此外，我们还将了解如何正确处理 Windows GUI 应用程序中的任务。

# 创建任务

此配方展示了任务的基本概念。您将学习如何创建和执行任务。

## 准备工作

要逐步执行此配方，您需要 Visual Studio 2015。没有其他先决条件。此配方的源代码可以在 `BookSamples\Chapter4\Recipe1` 中找到。

## 如何操作...

要创建和执行任务，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

    ### 注意

    这次，请确保您在每个项目中使用 .Net Framework 4.5 或更高版本。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.Threading.Tasks;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    static void TaskMethod(string name)
    {
      WriteLine($"Task {name} is running on a thread id " +
          $"{CurrentThread.ManagedThreadId}. Is thread pool thread: " +
          $"{CurrentThread.IsThreadPoolThread}");
    }
    ```

1.  在 `Main` 方法内添加以下代码片段：

    ```cs
    var t1 = new Task(() => TaskMethod("Task 1"));
    var t2 = new Task(() => TaskMethod("Task 2"));
    t2.Start();
    t1.Start();
    Task.Run(() => TaskMethod("Task 3"));
    Task.Factory.StartNew(() => TaskMethod("Task 4"));
    Task.Factory.StartNew(() => TaskMethod("Task 5"), TaskCreationOptions.LongRunning);
    Sleep(TimeSpan.FromSeconds(1));
    ```

1.  运行程序。

## 工作原理...

当程序运行时，它使用其构造函数创建两个任务。我们将 lambda 表达式作为 `Action` 委托传递；这允许我们向 `TaskMethod` 提供一个字符串参数。然后，我们使用 `Start` 方法运行这些任务。

### 注意

注意，直到我们调用这些任务的 `Start` 方法，它们将不会开始执行。很容易忘记实际启动任务。

然后，我们使用`Task.Run`和`Task.Factory.StartNew`方法运行两个更多任务。区别在于创建的任务立即开始工作，因此我们不需要显式地调用任务的`Start`方法。所有任务，编号从`Task 1`到`Task 4`，都被放置在线程池工作线程上，并以不确定的顺序运行。如果你多次运行程序，你会发现任务执行顺序是没有定义的。

`Task.Run`方法只是`Task.Factory.StartNew`的快捷方式，但后者方法有额外的选项。通常，除非你需要做特别的事情，例如在`Task 5`的情况下，使用前者方法。我们将此任务标记为长时间运行，因此这个任务将在不使用线程池的单独线程上运行。然而，这种行为可能会根据运行任务的当前**任务调度器**而改变。你将在本章的最后一个食谱中学习什么是任务调度器。

# 使用任务执行基本操作

本食谱将描述如何从任务中获取结果值。我们将通过几个场景来了解在线程池或主线程上运行任务之间的区别。

## 准备工作

要开始这个食谱，你需要 Visual Studio 2015。没有其他先决条件。本食谱的源代码可以在`BookSamples\Chapter4\Recipe2`中找到。

## 如何操作...

要使用任务执行基本操作，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Threading.Tasks;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static Task<int> CreateTask(string name)
    {
      return new Task<int>(() => TaskMethod(name));
    }

    static int TaskMethod(string name)
    {
      WriteLine($"Task {name} is running on a thread id " +
         $"{CurrentThread.ManagedThreadId}. Is thread pool thread: " +
         $"{CurrentThread.IsThreadPoolThread}");
      Sleep(TimeSpan.FromSeconds(2));
      return 42;
    }
    ```

1.  在`Main`方法内部添加以下代码片段：

    ```cs
    TaskMethod("Main Thread Task");
    Task<int> task = CreateTask("Task 1");
    task.Start();
    int result = task.Result;
    WriteLine($"Result is: {result}");

    task = CreateTask("Task 2");
    task.RunSynchronously();
    result = task.Result;
    WriteLine($"Result is: {result}");

    task = CreateTask("Task 3");
    WriteLine(task.Status);
    task.Start();

    while (!task.IsCompleted)
    {
      WriteLine(task.Status);
      Sleep(TimeSpan.FromSeconds(0.5)); } 

    WriteLine(task.Status);
    result = task.Result;
    WriteLine($"Result is: {result}");
    ```

1.  运行程序。

## 它是如何工作的...

首先，我们不将`TaskMethod`包装成任务来运行它。结果，它是同步执行的，为我们提供了有关主线程的信息。显然，它不是线程池线程。

然后，我们使用`Start`方法启动`Task 1`，并等待结果。这个任务将被放置在线程池中，主线程将等待并阻塞，直到任务返回。

我们对`Task 2`也做同样的处理，只不过我们使用`RunSynchronously()`方法来运行它。这个任务将在主线程上运行，并且我们得到与第一次调用`TaskMethod`同步时完全相同的输出。这是一个非常有用的优化，它允许我们避免在非常短暂的操作中使用线程池。

我们以与`Task 1`相同的方式运行`Task 3`，但不是阻塞主线程，而是旋转，打印出任务状态，直到任务完成。这显示了几个任务状态，分别是`Created`、`Running`和`RanToCompletion`。

# 结合任务

此配方将向您展示如何设置相互依赖的任务。我们将学习如何创建在父任务完成后运行的任务。此外，我们还将发现一种为非常短暂的任务节省线程使用的方法。

## 准备工作

要逐步执行此配方，您需要 Visual Studio 2015。没有其他先决条件。此配方的源代码可以在 `BookSamples\Chapter4\Recipe3` 中找到。

## 如何操作...

要组合任务，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.Threading.Tasks;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    static int TaskMethod(string name, int seconds)
    {
      WriteLine(
        $"Task {name} is running on a thread id " +
        $"{CurrentThread.ManagedThreadId}. Is thread pool thread: " +
        $"{CurrentThread.IsThreadPoolThread}");
      Sleep(TimeSpan.FromSeconds(seconds));
      return 42 * seconds;
    }
    ```

1.  在 `Main` 方法内添加以下代码片段：

    ```cs
    var firstTask = new Task<int>(() => TaskMethod("First Task", 3));
    var secondTask = new Task<int>(() => TaskMethod("Second Task", 2));

    firstTask.ContinueWith(
      t => WriteLine(
        $"The first answer is {t.Result}. Thread id " +
        $"{CurrentThread.ManagedThreadId}, is thread pool thread: " +
        $"{CurrentThread.IsThreadPoolThread}"),
      TaskContinuationOptions.OnlyOnRanToCompletion);

    firstTask.Start();
    secondTask.Start();

    Sleep(TimeSpan.FromSeconds(4));

    Task continuation = secondTask.ContinueWith(
      t => WriteLine(
        $"The second answer is {t.Result}. Thread id " +
        $"{CurrentThread.ManagedThreadId}, is thread pool thread: " +
        $"{CurrentThread.IsThreadPoolThread}"),
      TaskContinuationOptions.OnlyOnRanToCompletion 
        | TaskContinuationOptions.ExecuteSynchronously);

    continuation.GetAwaiter().OnCompleted(
      () => WriteLine(
        $"Continuation Task Completed! Thread id " +
        $"{CurrentThread.ManagedThreadId}, is thread pool thread: " +
        $"{CurrentThread.IsThreadPoolThread}"));

    Sleep(TimeSpan.FromSeconds(2));
    WriteLine();

    firstTask = new Task<int>(() =>
    {
      var innerTask = Task.Factory.StartNew(() => TaskMethod("Second Task", 5),TaskCreationOptions.AttachedToParent);

      innerTask.ContinueWith(t => TaskMethod("Third Task", 2),
            TaskContinuationOptions.AttachedToParent);

      return TaskMethod("First Task", 2);
    });

    firstTask.Start();

    while (!firstTask.IsCompleted)
    {
      WriteLine(firstTask.Status);
      Sleep(TimeSpan.FromSeconds(0.5));
    }
    WriteLine(firstTask.Status);

    Sleep(TimeSpan.FromSeconds(10));
    ```

1.  运行程序。

## 工作原理...

当主程序启动时，我们创建了两个任务，并为第一个任务设置了一个**延续**（在先决任务完成后运行的代码块）。然后，我们启动两个任务并等待 4 秒，这对于两个任务完成来说足够了。然后，我们对第二个任务运行另一个延续，并通过指定 `TaskContinuationOptions.ExecuteSynchronously` 选项来尝试同步执行它。当延续非常短暂时，这是一个有用的技术，在主线程上运行它比将其放在线程池中要快。我们能够实现这一点，因为此时第二个任务已经完成。如果我们注释掉 4 秒的 `Thread.Sleep` 方法，我们将看到此代码将被放在线程池中，因为我们还没有先决任务的结果。

最后，我们为之前的延续定义了一个延续，但方式略有不同，使用了新的 `GetAwaiter` 和 `OnCompleted` 方法。这些方法旨在与 C# 语言的异步机制一起使用。我们将在第五章使用 C# 6.0 中介绍这个主题。

演示的最后部分是关于父-子任务关系。我们创建一个新的任务，并在运行此任务的同时，通过提供 `TaskCreationOptions.AttachedToParent` 选项来运行所谓的子任务。

### 小贴士

子任务必须在运行父任务时创建，以确保它正确地附加到父任务上！

这意味着父任务将不会完成，直到所有子任务完成其工作。我们还可以运行提供 `TaskContinuationOptions.AttachedToParent` 选项的子任务的延续。这些延续任务也会影响父任务，并且它将不会完成，直到最后一个子任务结束。

# 将 APM 模式转换为任务

在此配方中，我们将看到如何将旧式的 APM API 转换为任务。在转换过程中可能发生不同情况都有示例。

## 准备工作

要开始此配方，您需要 Visual Studio 2015。没有其他先决条件。此配方的源代码可以在 `BookSamples\Chapter4\Recipe4` 中找到。

## 如何做...

要将 APM 模式转换为任务，执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.Threading.Tasks;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    delegate string AsynchronousTask(string threadName);
    delegate string IncompatibleAsynchronousTask(out int threadId);

    static void Callback(IAsyncResult ar)
    {
      WriteLine("Starting a callback...");
      WriteLine($"State passed to a callbak: {ar.AsyncState}");
      WriteLine($"Is thread pool thread: {CurrentThread.IsThreadPoolThread}");
      WriteLine($"Thread pool worker thread id: {CurrentThread.ManagedThreadId}");
    }

    static string Test(string threadName)
    {
      WriteLine("Starting...");
      WriteLine($"Is thread pool thread: {CurrentThread.IsThreadPoolThread}");
      Sleep(TimeSpan.FromSeconds(2));
      CurrentThread.Name = threadName;
      return $"Thread name: {CurrentThread.Name}";
    }

    static string Test(out int threadId)
    {
      WriteLine("Starting...");
      WriteLine($"Is thread pool thread: {CurrentThread.IsThreadPoolThread}");
      Sleep(TimeSpan.FromSeconds(2));
      threadId = CurrentThread.ManagedThreadId;
      return $"Thread pool worker thread id was: {threadId}";
    }
    ```

1.  在 `Main` 方法内添加以下代码片段：

    ```cs
    int threadId;
    AsynchronousTask d = Test;
    IncompatibleAsynchronousTask e = Test;

    WriteLine("Option 1");
    Task<string> task = Task<string>.Factory.FromAsync(
      d.BeginInvoke("AsyncTaskThread", Callback, 
        "a delegate asynchronous call"), d.EndInvoke);

    task.ContinueWith(t => WriteLine(
        $"Callback is finished, now running a continuation! Result: {t.Result}"));

    while (!task.IsCompleted)
    {
      WriteLine(task.Status);
      Sleep(TimeSpan.FromSeconds(0.5));
    }
    WriteLine(task.Status);
    Sleep(TimeSpan.FromSeconds(1));

    WriteLine("----------------------------------------------");
    WriteLine();
    WriteLine("Option 2");

    task = Task<string>.Factory.FromAsync(
      d.BeginInvoke, d.EndInvoke, "AsyncTaskThread", "a delegate asynchronous call");

    task.ContinueWith(t => WriteLine(
        $"Task is completed, now running a continuation! Result: {t.Result}"));
    while (!task.IsCompleted)
    {
      WriteLine(task.Status);
      Sleep(TimeSpan.FromSeconds(0.5));
    }
    WriteLine(task.Status);
    Sleep(TimeSpan.FromSeconds(1));

    WriteLine("----------------------------------------------");
    WriteLine();
    WriteLine("Option 3");

    IAsyncResult ar = e.BeginInvoke(out threadId, Callback, "a delegate asynchronous call");
    task = Task<string>.Factory.FromAsync(ar, _ => e.EndInvoke(out threadId, ar));

    task.ContinueWith(t => 
      WriteLine(
            $"Task is completed, now running a continuation! " +
            $"Result: {t.Result}, ThreadId: {threadId}"));

    while (!task.IsCompleted)
    {
      WriteLine(task.Status);
      Sleep(TimeSpan.FromSeconds(0.5));
    }
    WriteLine(task.Status);

    Sleep(TimeSpan.FromSeconds(1));
    ```

1.  运行程序。

## 它是如何工作的...

在这里，我们定义了两种类型的委托；其中一种使用 `out` 参数，因此与将 APM 模式转换为任务的 TPL 标准 API 不兼容。然后，我们有三个此类转换的示例。

将 APM 转换为 TPL 的关键点是 `Task<T>.Factory.FromAsync` 方法，其中 `T` 是异步操作的结果类型。此方法有几个重载；在第一种情况下，我们传递 `IAsyncResult` 和 `Func<IAsyncResult, string>`，这是一个接受 `IAsyncResult` 实现并返回字符串的方法。由于第一个委托类型提供了与该签名兼容的 `EndMethod`，因此我们可以将此委托异步调用转换为任务。

在第二个示例中，我们几乎做了同样的事情，但使用了不同的 `FromAsync` 方法重载，它不允许指定在异步委托调用完成后要执行的回调。我们能够用延续来替换它，但如果回调很重要，我们可以使用第一个示例。

最后一个示例展示了一个小技巧。这次，`IncompatibleAsynchronousTask` 委托的 `EndMethod` 使用 `out` 参数，并且与任何 `FromAsync` 方法重载都不兼容。然而，将 `EndMethod` 调用包装到一个适合任务工厂的 lambda 表达式中是非常容易的。

要查看底层任务的状态，我们在等待异步操作结果的同时打印其状态。我们看到第一个任务的状态是 `WaitingForActivation`，这意味着任务尚未由 TPL 基础设施实际启动。

# 将 EAP 模式转换为任务

此配方将描述如何将基于事件的异步操作转换为任务。在此配方中，您将找到一个适用于 .NET Framework 类库中每个基于事件的异步 API 的可靠模式。

## 准备工作

要开始此配方，您需要 Visual Studio 2015。没有其他先决条件。此配方的源代码可以在 `BookSamples\Chapter4\Recipe5` 中找到。

## 如何做...

要将 EAP 模式转换为任务，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.ComponentModel;
    using System.Threading.Tasks;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    static int TaskMethod(string name, int seconds)
    {
      WriteLine(
        $"Task {name} is running on a thread id " +
        $"{CurrentThread.ManagedThreadId}. Is thread pool thread: " +
        $"{CurrentThread.IsThreadPoolThread}");

      Sleep(TimeSpan.FromSeconds(seconds));
      return 42 * seconds;
    }
    ```

1.  在 `Main` 方法内添加以下代码片段：

    ```cs
    var tcs = new TaskCompletionSource<int>();

    var worker = new BackgroundWorker();
    worker.DoWork += (sender, eventArgs) =>
    {
      eventArgs.Result = TaskMethod("Background worker", 5);
    };

    worker.RunWorkerCompleted += (sender, eventArgs) =>
    {
      if (eventArgs.Error != null)
      {
        tcs.SetException(eventArgs.Error);
      }
      else if (eventArgs.Cancelled)
      {
        tcs.SetCanceled();
      }
      else
      {
        tcs.SetResult((int)eventArgs.Result);
      }
    };

    worker.RunWorkerAsync();

    int result = tcs.Task.Result;

    WriteLine($"Result is: {result}");
    ```

1.  运行程序。

## 它是如何工作的...

这是一个将 EAP 模式转换为任务的非常简单且优雅的例子。关键点是使用`TaskCompletionSource<T>`类型，其中`T`是异步操作的结果类型。

同样重要的是不要忘记将`tcs.SetResult`方法调用包装在`try/catch`块中，以确保错误信息始终被设置为任务完成源对象。也可以使用`TrySetResult`方法代替`SetResult`来确保结果已成功设置。

# 实现取消选项

这个配方是关于实现基于任务的异步操作的取消过程。您将学习如何正确使用取消令牌来处理任务，以及如何在任务实际运行之前找出任务是否被取消。

## 准备工作

要开始这个配方，您需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter4\Recipe6`中找到。

## 如何实现...

要为基于任务的异步操作实现取消选项，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Threading;
    using System.Threading.Tasks;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static int TaskMethod(string name, int seconds, CancellationToken token)
    {
      WriteLine(
        $"Task {name} is running on a thread id " +
        $"{CurrentThread.ManagedThreadId}. Is thread pool thread: " +
        $"{CurrentThread.IsThreadPoolThread}");

      for (int i = 0; i < seconds; i ++)
      {
        Sleep(TimeSpan.FromSeconds(1));
        if (token.IsCancellationRequested) return -1;
      }
      return 42*seconds;
    }
    ```

1.  在`Main`方法内部添加以下代码片段：

    ```cs
    var cts = new CancellationTokenSource();
    var longTask = new Task<int>(() => TaskMethod("Task 1", 10, cts.Token), cts.Token);
    WriteLine(longTask.Status);
    cts.Cancel();
    WriteLine(longTask.Status);
    WriteLine("First task has been cancelled before execution");

    cts = new CancellationTokenSource();
    longTask = new Task<int>(() => TaskMethod("Task 2", 10, cts.Token), cts.Token);
    longTask.Start();
    for (int i = 0; i < 5; i++ )
    {
      Sleep(TimeSpan.FromSeconds(0.5));
      WriteLine(longTask.Status);
    }
    cts.Cancel();
    for (int i = 0; i < 5; i++)
    {
      Sleep(TimeSpan.FromSeconds(0.5));
      WriteLine(longTask.Status);
    }

    WriteLine($"A task has been completed with result {longTask.Result}.");
    ```

1.  运行程序。

## 它是如何工作的...

这又是一个如何为 TPL 任务实现取消选项的简单示例。您已经熟悉我们在第三章中讨论的取消令牌概念，*使用线程池*。

首先，让我们仔细看看`longTask`的创建代码。我们一次性将取消令牌提供给底层任务，然后再次将其提供给任务构造函数。*为什么我们需要提供这个令牌两次？*

答案是，如果我们在一个任务实际开始之前取消它，那么它的 TPL 基础设施将负责处理取消操作，因为我们的代码根本不会执行。我们知道第一个任务是通过获取其状态被取消的。如果我们尝试在这个任务上调用`Start`方法，我们将得到`InvalidOperationException`。

然后，我们处理从我们自己的代码中取消操作的过程。这意味着我们现在完全负责取消过程，并且在取消任务后，其状态仍然是`RanToCompletion`，因为从 TPL 的角度来看，任务正常完成了其工作。区分这两种情况并理解每种情况下的责任差异非常重要。

# 处理任务中的异常

这个配方描述了处理异步任务中异常的非常重要的话题。我们将探讨任务抛出的异常的不同方面以及如何获取它们的信息。

## 准备工作

要逐步执行这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter4\Recipe7`找到。

## 如何做...

要处理任务中的异常，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Threading.Tasks;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static int TaskMethod(string name, int seconds)
    {
      WriteLine(
        $"Task {name} is running on a thread id " +
        $"{CurrentThread.ManagedThreadId}. Is thread pool thread: " +
        $"{CurrentThread.IsThreadPoolThread}");

      Sleep(TimeSpan.FromSeconds(seconds));
      throw new Exception("Boom!");
      return 42 * seconds;
    }
    ```

1.  在`Main`方法内部添加以下代码片段：

    ```cs
    Task<int> task;
    try
    {
      task = Task.Run(() => TaskMethod("Task 1", 2));
      int result = task.Result;
      WriteLine($"Result: {result}");
    }
    catch (Exception ex)
    {
      WriteLine($"Exception caught: {ex}");
    }
    WriteLine("----------------------------------------------");
    WriteLine();

    try
    {
      task = Task.Run(() => TaskMethod("Task 2", 2));
      int result = task.GetAwaiter().GetResult();
          WriteLine($"Result: {result}");
    }
    catch (Exception ex)
    {
      WriteLine($"Exception caught: {ex}");
    }
    WriteLine("----------------------------------------------");
    WriteLine();

    var t1 = new Task<int>(() => TaskMethod("Task 3", 3));
    var t2 = new Task<int>(() => TaskMethod("Task 4", 2));
    var complexTask = Task.WhenAll(t1, t2);
    var exceptionHandler = complexTask.ContinueWith(t => 
        WriteLine($"Exception caught: {t.Exception}"), 
        TaskContinuationOptions.OnlyOnFaulted
      );
    t1.Start();
    t2.Start();

    Sleep(TimeSpan.FromSeconds(5));
    ```

1.  运行程序。

## 它是如何工作的...

当程序启动时，我们创建一个任务并尝试同步地获取任务结果。`Result`属性的`Get`部分使当前线程等待任务完成并将异常传播到当前线程。在这种情况下，我们很容易在捕获块中捕获异常，但这个异常是一个名为`AggregateException`的包装异常。在这种情况下，它只包含一个异常，因为只有一个任务抛出了这个异常，并且可以通过访问`InnerException`属性来获取底层异常。

第二个示例基本上相同，但为了访问任务结果，我们使用`GetAwaiter`和`GetResult`方法。在这种情况下，我们没有包装异常，因为它是通过 TPL 基础设施解包的。我们立即有一个原始异常，如果我们只有一个底层任务，这将非常方便。

最后一个示例显示了存在两个抛出异常的任务的情况。为了处理异常，我们现在使用一个延续，它仅在先决任务以异常结束的情况下执行。这种行为是通过向延续提供一个`TaskContinuationOptions.OnlyOnFaulted`选项来实现的。结果，我们打印出`AggregateException`，并且我们有两个来自它内部的内部异常。

## 还有更多...

由于任务可能以非常不同的方式连接，结果`AggregateException`异常可能包含其他聚合异常，以及通常的异常。这些内部聚合异常可能自身包含其他聚合异常。

要去除这些包装器，我们应该使用根聚合异常的`Flatten`方法。它将返回所有子聚合异常的内部异常的集合。

# 并行运行任务

这个配方展示了如何处理同时运行的大量异步任务。你将学习如何在所有任务完成或任何正在运行的任务需要完成其工作的情况下有效地得到通知。

## 准备工作

要开始这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter4\Recipe8`找到。

## 如何做...

要并行运行任务，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static int TaskMethod(string name, int seconds)
    {
      WriteLine(
        $"Task {name} is running on a thread id " +
        $"{CurrentThread.ManagedThreadId}. Is thread pool thread: " +
        $"{CurrentThread.IsThreadPoolThread}");

      Sleep(TimeSpan.FromSeconds(seconds));
      return 42 * seconds;
    }
    ```

1.  在`Main`方法内添加以下代码片段：

    ```cs
    var firstTask = new Task<int>(() => TaskMethod("First Task", 3));
    var secondTask = new Task<int>(() => TaskMethod("Second Task", 2));
    var whenAllTask = Task.WhenAll(firstTask, secondTask);

    whenAllTask.ContinueWith(t =>
      WriteLine($"The first answer is {t.Result[0]}, the second is {t.Result[1]}"),
      TaskContinuationOptions.OnlyOnRanToCompletion);

    firstTask.Start();
    secondTask.Start();

    Sleep(TimeSpan.FromSeconds(4));

    var tasks = new List<Task<int>>();
    for (int i = 1; i < 4; i++)
    {
      int counter = i;
      var task = new Task<int>(() => TaskMethod($"Task {counter}", counter));
      tasks.Add(task);
      task.Start();
    }

    while (tasks.Count > 0)
    {
      var completedTask = Task.WhenAny(tasks).Result;
      tasks.Remove(completedTask);
      WriteLine($"A task has been completed with result {completedTask.Result}.");
    }

    Sleep(TimeSpan.FromSeconds(1));
    ```

1.  运行程序。

## 它是如何工作的...

当程序启动时，我们创建两个任务，然后，借助`Task.WhenAll`方法，我们创建第三个任务，该任务将在所有初始任务完成后完成。结果任务为我们提供了一个答案数组，其中第一个元素包含第一个任务的结果，第二个元素包含第二个结果，依此类推。

然后，我们创建另一个任务列表，并使用`Task.WhenAny`方法等待这些任务中的任何一个完成。在我们有一个完成的任务后，我们将其从列表中删除，并继续等待其他任务完成，直到列表为空。此方法用于获取任务完成进度或在运行任务时使用超时。例如，我们等待一定数量的任务，其中有一个任务是计数超时。如果这个任务首先完成，我们就取消所有尚未完成的任务。

# 使用`TaskScheduler`调整任务的执行

这个配方描述了处理任务的一个重要方面，即从异步代码中正确处理 UI 的方式。你将学习什么是任务调度器，为什么它如此重要，它如何损害我们的应用程序，以及如何使用它来避免错误。

## 准备工作

要逐步执行此配方，你需要 Visual Studio 2015。没有其他先决条件。此配方的源代码可以在`BookSamples\Chapter4\Recipe9`中找到。

## 如何做到这一点...

要使用`TaskScheduler`调整任务执行，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# **WPF 应用程序**项目。这次，我们需要一个具有消息循环的用户界面线程，这在控制台应用程序中是不可用的。

1.  在`MainWindow.xaml`文件中，在网格元素内（即在`<Grid>`和`</Grid>`标签之间）添加以下标记：

    ```cs
    <TextBlock Name="ContentTextBlock"
    HorizontalAlignment="Left"
    Margin="44,134,0,0"
    VerticalAlignment="Top"
    Width="425"
    Height="40"/>
    <Button Content="Sync"
    HorizontalAlignment="Left"
    Margin="45,190,0,0"
    VerticalAlignment="Top"
    Width="75"
    Click="ButtonSync_Click"/>
    <Button Content="Async"
    HorizontalAlignment="Left"
    Margin="165,190,0,0"
    VerticalAlignment="Top"
    Width="75"
    Click="ButtonAsync_Click"/>
    <Button Content="Async OK"
    HorizontalAlignment="Left"
    Margin="285,190,0,0"
    VerticalAlignment="Top"
    Width="75"
    Click="ButtonAsyncOK_Click"/>
    ```

1.  在`MainWindow.xaml.cs`文件中，使用以下`using`指令：

    ```cs
    using System;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Windows;
    using System.Windows.Input;
    ```

1.  在`MainWindow`构造函数下方添加以下代码片段：

    ```cs
    void ButtonSync_Click(object sender, RoutedEventArgs e)
    {
      ContentTextBlock.Text = string.Empty;
      try
      {
        //string result = TaskMethod(
        //  TaskScheduler.FromCurrentSynchronizationContext()).Result;
        string result = TaskMethod().Result;
        ContentTextBlock.Text = result;
      }
      catch (Exception ex)
      {
        ContentTextBlock.Text = ex.InnerException.Message;
      }
    }

    void ButtonAsync_Click(object sender, RoutedEventArgs e)
    {
      ContentTextBlock.Text = string.Empty;
      Mouse.OverrideCursor = Cursors.Wait;
      Task<string> task = TaskMethod();
      task.ContinueWith(t => 
        {
          ContentTextBlock.Text = t.Exception.InnerException.Message;
          Mouse.OverrideCursor = null;
        }, 
        CancellationToken.None,
        TaskContinuationOptions.OnlyOnFaulted,
        TaskScheduler.FromCurrentSynchronizationContext());
    }

    void ButtonAsyncOK_Click(object sender, RoutedEventArgs e)
    {
      ContentTextBlock.Text = string.Empty;
      Mouse.OverrideCursor = Cursors.Wait;
      Task<string> task = TaskMethod(
        TaskScheduler.FromCurrentSynchronizationContext());

      task.ContinueWith(t => Mouse.OverrideCursor = null,
        CancellationToken.None,
        TaskContinuationOptions.None,
        TaskScheduler.FromCurrentSynchronizationContext());
    }

    Task<string> TaskMethod()
    {
      return TaskMethod(TaskScheduler.Default);
    }

    Task<string> TaskMethod(TaskScheduler scheduler)
    {
      Task delay = Task.Delay(TimeSpan.FromSeconds(5));

      return delay.ContinueWith(t =>
      {
        string str =
          "Task is running on a thread id " +
          $"{CurrentThread.ManagedThreadId}. Is thread pool thread: " +
          $"{CurrentThread.IsThreadPoolThread}";

        ContentTextBlock.Text = str;
        return str;
      }, scheduler);
    }
    ```

1.  运行程序。

## 它是如何工作的...

这里，我们遇到了许多新事物。首先，我们创建了一个 WPF 应用程序而不是控制台应用程序。这是必要的，因为我们需要一个具有消息循环的用户界面线程来演示异步运行任务的不同选项。

有一个非常重要的抽象叫做`TaskScheduler`。这个组件实际上负责任务将如何执行。默认任务调度器将任务放在线程池工作线程上。这是最常见的情况；不出所料，它是 TPL 中的默认选项。我们也知道如何同步运行任务，以及如何将它们附加到父任务以一起运行这些任务。现在，让我们看看我们还可以用任务做什么。

当程序启动时，我们创建了一个包含三个按钮的窗口。第一个按钮调用同步任务执行。代码放置在 `ButtonSync_Click` 方法内部。在任务运行期间，我们甚至无法移动应用程序窗口。当用户界面线程忙于运行任务且无法响应任何消息循环直到任务完成时，用户界面会完全冻结。这在 GUI 窗口应用程序中是一种相当常见的坏习惯，我们需要找到一种方法来解决这个问题。

第二个问题是，我们尝试从另一个线程访问 UI 控件。图形用户界面控件从未被设计为可以从多个线程使用，为了避免可能出现的错误，不允许从创建它的线程以外的线程访问这些组件。当我们尝试这样做时，我们会得到一个异常，并且异常消息将在 5 秒内在主窗口上打印出来。

为了解决第一个问题，我们尝试异步运行任务。这就是第二个按钮所做的事情；这个按钮的代码放置在 `ButtonAsync_Click` 方法内部。如果你在调试器中运行任务，你会看到它被放置在线程池中，最终我们将会得到相同的异常。然而，在整个任务运行期间，用户界面始终保持响应。这是一件好事，但我们需要消除异常。

我们已经做到了！为了输出错误消息，我们使用 `TaskScheduler.FromCurrentSynchronizationContext` 选项提供了一个延续。如果没有这样做，我们就不会看到错误消息，因为我们可能会得到任务内部发生的相同异常。此选项指示 TPL 基础设施将代码放在延续中，并在 UI 线程上运行，同时利用 UI 线程的消息循环异步执行。这解决了从另一个线程访问 UI 控件的问题，但仍然保持了我们的 UI 响应。

为了验证这是否正确，我们按下运行 `ButtonAsyncOK_Click` 方法内部代码的最后一个按钮。唯一不同的是，我们为任务提供了 UI 线程任务调度器。任务完成后，你会看到它是以异步方式在 UI 线程上运行的。UI 保持响应，即使在等待光标激活的情况下，也可以按下另一个按钮。

然而，有一些技巧可以用来在 UI 线程上运行任务。如果我们回到同步任务代码，并取消注释使用 UI 线程任务调度器获取结果的行，我们将永远不会得到任何结果。这是一个经典的死锁情况：我们在 UI 线程的队列中调度了一个操作，UI 线程等待这个操作完成，但是当它等待时，它无法运行这个操作，它将永远不会结束（甚至开始）。如果我们对一个任务调用 `Wait` 方法，也会发生这种情况。为了避免死锁，永远不要在调度到 UI 线程的任务上使用同步操作；只需使用 C# 中的 `ContinueWith` 或 `async`/`await`。
