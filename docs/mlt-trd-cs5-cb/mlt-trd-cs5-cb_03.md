# 第三章：使用线程池

在本章中，我们将描述使用多个线程从共享资源中工作的常见技术。您将了解：

+   在线程池上调用委托

+   在线程池上发布异步操作

+   线程池和并行度

+   实现取消选项

+   使用等待句柄和线程池的超时

+   使用定时器

+   使用 BackgroundWorker 组件

# 介绍

在前几章中，我们讨论了创建线程和组织它们的合作的几种方法。现在让我们考虑另一种情况，即创建许多需要很短时间完成的异步操作。正如我们在第一章的*介绍*部分中讨论的那样，创建线程是一项昂贵的操作，因此为每个短暂的异步操作进行这样的操作将包含显着的开销。

为了解决这个问题，有一种称为**池化**的常见方法，可以成功应用于任何需要许多短暂的昂贵资源的情况。我们预先分配一定数量的这些资源，并将它们组织成资源池。每当我们需要新资源时，我们只需从池中取出，而不是创建一个新的，并在资源不再需要时将其返回到池中。

**.NET 线程池**是这个概念的一种实现。它可以通过`System.Threading.ThreadPool`类型访问。线程池由.NET **公共语言运行时**（**CLR**）管理，这意味着每个 CLR 都有一个线程池实例。`ThreadPool`类型有一个`QueueUserWorkItem`静态方法，接受一个代表用户定义的异步操作的**委托**。调用此方法后，该委托进入内部队列。然后，如果线程池中没有线程，它会创建一个新的**工作线程**，并将第一个委托放入队列中。

如果我们在线程池中放置新操作，那么在前面的操作完成后，可以重用这个线程来执行这些操作。但是，如果我们更快地放置新操作，线程池将创建更多线程来为这些操作提供服务。有一个限制来防止创建太多的线程，在这种情况下，新操作将在队列中等待，直到线程池中的工作线程空闲为止。

### 注意

非常重要的是，要保持线程池中的操作生命周期短暂！不要将长时间运行的操作放在线程池中或阻塞工作线程。这将导致所有工作线程都变得忙碌，它们将无法再为用户操作提供服务。这反过来会导致性能问题和非常难以调试的错误。

当我们停止在线程池上放置新操作时，它最终会删除不再需要的线程，这些线程在一段时间后处于空闲状态。这将释放不再需要的任何操作系统资源。

我想再次强调，线程池旨在执行短期操作。使用线程池可以节省操作系统资源，但会降低并行度。我们使用较少的线程，但执行异步操作比通常慢，通过可用的工作线程数量进行批处理。如果操作完成速度很快，这是有意义的，但对于执行许多长时间运行的计算密集型操作，性能会下降。

另一个需要非常小心的重要事情是在 ASP.NET 应用程序中使用线程池。ASP.NET 基础设施本身使用线程池，如果浪费了所有线程池的工作线程，Web 服务器将无法再服务传入的请求。建议在 ASP.NET 中只使用输入/输出绑定的异步操作，因为它们使用一种称为 I/O 线程的不同机制。我们将在第九章中讨论 I/O 线程，*使用异步 I/O*。

### 注意

请注意，线程池的工作线程是后台线程。这意味着当前台中的所有线程（包括主应用程序线程）完成后，所有后台线程将停止。

在本章中，我们将学习如何使用线程池执行异步操作。我们将涵盖将操作放在线程池上的不同方法，以及如何取消操作并防止其长时间运行。

# 在线程池上调用委托

这个配方将向你展示如何在线程池上异步执行委托。此外，我们将讨论一种称为**异步编程模型**（**APM**）的方法，这是.NET 中历史上第一种异步编程模式。

## 准备工作

进入这个配方，你需要 Visual Studio 2012。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter3\Recipe1`中找到

## 如何做...

要了解如何在线程池上调用委托，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# **控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
private delegate string RunOnThreadPool(out int threadId);

private static void Callback(IAsyncResultar)
{
  Console.WriteLine("Starting a callback...");
  Console.WriteLine("State passed to a callback: {0}",ar.AsyncState);
  Console.WriteLine("Is thread pool thread: {0}",Thread.CurrentThread.IsThreadPoolThread);
  Console.WriteLine("Thread pool worker thread id: {0}",Thread.CurrentThread.ManagedThreadId);
}

private static string Test(out intthreadId)
{
  Console.WriteLine("Starting...");
  Console.WriteLine("Is thread pool thread: {0}",Thread.CurrentThread.IsThreadPoolThread);
  Thread.Sleep(TimeSpan.FromSeconds(2));
  threadId = Thread.CurrentThread.ManagedThreadId;
  return string.Format("Thread pool worker thread id was:{0}", threadId);
}
```

1.  在`Main`方法中添加以下代码：

```cs
int threadId = 0;

RunOnThreadPool poolDelegate = Test;

var t = new Thread(() => Test(out threadId));
t.Start();
t.Join();

Console.WriteLine("Thread id: {0}", threadId);

IAsyncResult r = poolDelegate.BeginInvoke(out threadId,Callback, "a delegate asynchronous call");
r.AsyncWaitHandle.WaitOne();

string result = poolDelegate.EndInvoke(out threadId, r);

Console.WriteLine("Thread pool worker thread id: {0}",threadId);
Console.WriteLine(result);

Thread.Sleep(TimeSpan.FromSeconds(2));
```

1.  运行程序。

## 它是如何工作的...

程序运行时，以一种老式的方式创建一个线程，然后启动它并等待其完成。由于线程构造函数只接受一个不返回任何结果的方法，我们使用**lambda 表达式**来包装对`Test`方法的调用。我们确保这个线程不是来自线程池，通过打印`Thread.CurrentThread.IsThreadPoolThread`属性值。我们还打印一个托管线程 ID 来标识执行此代码的线程。

然后我们定义一个委托，并通过调用`BeginInvoke`方法来运行它。这个方法接受一个回调函数，在异步操作完成后将被调用，以及一个用户定义的状态传递到回调函数中。这个状态通常用于区分一个异步调用和另一个。结果，我们得到一个实现`IAsyncResult`接口的`result`对象。`BeginInvoke`立即返回结果，允许我们在异步操作在线程池的工作线程上执行时继续进行任何工作。当我们需要异步操作的结果时，我们使用从`BeginInvoke`方法调用返回的`result`对象。我们可以使用`result`属性`IsCompleted`进行轮询，但在这种情况下，我们使用`AsyncWaitHandle`结果属性来等待，直到操作完成。完成后，要从中获取结果，我们在委托上调用`EndInvoke`方法，传递委托参数和我们的`IAsyncResult`对象。

### 注意

实际上，使用`AsyncWaitHandle`是不必要的。如果我们注释掉`r.AsyncWaitHandle.WaitOne`，代码仍然会成功运行，因为`EndInvoke`方法实际上会等待异步操作完成。始终重要的是调用`EndInvoke`（或对于其他异步 API，调用`EndOperationName`），因为它会将任何未处理的异常抛回到调用线程。在使用这种类型的异步 API 时，始终调用`Begin`和`End`方法。

当操作完成时，传递给`BeginInvoke`方法的回调将被发布到线程池，更具体地说，是一个工作线程。如果我们在`Main`方法定义的末尾注释掉`Thread.Sleep`方法调用，回调将不会被执行。这是因为当主线程完成时，所有后台线程都将被停止，包括这个回调。可能会有两个异步调用委托和一个回调将由同一个工作线程提供服务，这很容易通过工作线程 ID 看到。

在.NET 中使用`BeginOperationName`/`EndOperationName`方法和`IAsyncResult`对象的方法被称为异步编程模型或 APM 模式，这样的方法对被称为异步方法。这种模式仍然被用于各种.NET 类库 API 中，但在现代编程中，最好使用**任务并行库**（**TPL**）来组织异步 API。我们将在第四章中涵盖这个主题，*使用任务并行库*。

# 在线程池上发布异步操作

这个配方将描述如何将异步操作放在线程池中。

## 准备就绪

要进入这个配方，你需要 Visual Studio 2012。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter3\Recipe2`中找到。

## 如何做...

要理解如何将异步操作发布到线程池，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
usingSystem.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
private static void AsyncOperation(object state)
{
  Console.WriteLine("Operation state: {0}",state ?? "(null)");
  Console.WriteLine("Worker thread id: {0}",Thread.CurrentThread.ManagedThreadId);
  Thread.Sleep(TimeSpan.FromSeconds(2));
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
const int x = 1;
const int y = 2;
const string lambdaState = "lambda state 2";

ThreadPool.QueueUserWorkItem(AsyncOperation);
Thread.Sleep(TimeSpan.FromSeconds(1));

ThreadPool.QueueUserWorkItem(AsyncOperation,"async state");
Thread.Sleep(TimeSpan.FromSeconds(1));

ThreadPool.QueueUserWorkItem( state => {
  Console.WriteLine("Operation state: {0}", state);
  Console.WriteLine("Worker thread id: {0}",Thread.CurrentThread.ManagedThreadId);
  Thread.Sleep(TimeSpan.FromSeconds(2));
}, "lambda state");

ThreadPool.QueueUserWorkItem( _ => {
  Console.WriteLine("Operation state: {0}, {1}", x+y,lambdaState);
  Console.WriteLine("Worker thread id: {0}",Thread.CurrentThread.ManagedThreadId);
  Thread.Sleep(TimeSpan.FromSeconds(2));
}, "lambda state");

Thread.Sleep(TimeSpan.FromSeconds(2));
```

1.  运行程序。

## 它是如何工作的...

首先，我们定义了接受一个对象类型参数的`AsyncOperation`方法。然后，我们使用`QueueUserWorkItem`方法将这个方法发布到线程池。然后我们再次发布这个方法，但这次我们将一个`state`对象传递给这个方法调用。这个对象将作为`state`参数传递给`AsynchronousOperation`方法。

在这些操作之后让一个线程休眠 1 秒，为线程池提供重用线程的可能性进行新的操作。如果你注释掉这些`Thread.Sleep`调用，几乎可以肯定，所有情况下线程 ID 都会不同。如果不是，可能前两个线程将被重用来运行接下来的两个操作。

首先，我们将一个 lambda 表达式发布到线程池。这里没有什么特别的；我们使用 lambda 表达式语法来定义一个单独的方法。

其次，我们不是传递 lambda 表达式的状态，而是使用**闭包**机制。这给了我们更多的灵活性，并允许我们为异步操作提供多个对象和这些对象的静态类型。因此，以前将对象传递给方法回调的机制实际上是多余和过时的。现在当我们在 C#中有闭包时，就没有必要使用它了。

# 线程池和并行度

这个配方将展示线程池如何处理许多异步操作，以及它与创建许多单独线程的不同之处。

## 准备就绪

要进入这个配方，你需要 Visual Studio 2012。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter3\Recipe3`中找到。

## 如何做...

要了解线程池如何处理许多异步操作以及它与创建许多单独线程的不同之处，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Diagnostics;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void UseThreads(int numberOfOperations)
{
  using (var countdown = new CountdownEvent(numberOfOperations)) {

    Console.WriteLine("Scheduling work by creatingthreads");
    for (int i=0; i<numberOfOperations; i++) {
      var thread = new Thread(() => {
        Console.Write("{0},", Thread.CurrentThread.ManagedThreadId);
        Thread.Sleep(TimeSpan.FromSeconds(0.1));
        countdown.Signal();
      });
      thread.Start();
    }
    countdown.Wait();
    Console.WriteLine();
  }
}

static void UseThreadPool(int numberOfOperations)
{
  using (var countdown = new CountdownEvent(numberOfOperations)) {

    Console.WriteLine("Starting work on a threadpool");
    for (int i=0; i<numberOfOperations; i++) {
      ThreadPool.QueueUserWorkItem( _ => {
        Console.Write("{0},", Thread.CurrentThread.ManagedThreadId);
        Thread.Sleep(TimeSpan.FromSeconds(0.1));
        countdown.Signal();
      });
    }
    countdown.Wait();
    Console.WriteLine();
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
Console.WriteLine("Execution time using threads: {0}",sw.ElapsedMilliseconds);

sw.Reset();
sw.Start();
UseThreadPool(numberOfOperations);
sw.Stop();
Console.WriteLine("Execution time using threads: {0}",sw.ElapsedMilliseconds);
```

1.  运行程序。

## 它是如何工作的...

当主程序启动时，我们创建许多不同的线程，并在每个线程上运行一个操作。这个操作打印出一个线程 ID，并阻塞一个线程 100 毫秒。结果，我们创建了 500 个线程，它们都并行运行这些操作。在我的机器上，总时间约为 300 毫秒，但我们消耗了许多操作系统资源来运行所有这些线程。

然后，我们按照相同的程序进行，但是不是为每个操作创建一个线程，而是将它们发布到线程池上。之后，线程池开始为这些操作提供服务；它在最后开始创建更多的线程，但仍然需要更多的时间，大约在我的机器上需要 12 秒。我们节省了内存和线程供操作系统使用，但为此付出了执行时间。

# 实现取消选项

在这个示例中，有一个关于如何在线程池上取消异步操作的示例。

## 准备就绪

要进入这个示例，您需要 Visual Studio 2012。没有其他先决条件。这个示例的源代码可以在`BookSamples\Chapter3\Recipe4`中找到。

## 如何做...

要了解如何在线程上实现取消选项，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void AsyncOperation1(CancellationToken token) {
  Console.WriteLine("Starting the first task");
  for (int i=0; i<5; i++) {
    if (token.IsCancellationRequested) {
      Console.WriteLine("The first task has beencanceled.");
      return;
    }
    Thread.Sleep(TimeSpan.FromSeconds(1));
  }
  Console.WriteLine("The first task has completedsuccesfully");
}

static void AsyncOperation2(CancellationToken token) {
  try {
    Console.WriteLine("Starting the second task");

    for (int i=0; i<5; i++) {
      token.ThrowIfCancellationRequested();
      Thread.Sleep(TimeSpan.FromSeconds(1));
    }
    Console.WriteLine("The second task has completedsuccessfully");
  }
  catch (OperationCanceledException) {
    Console.WriteLine("The second task has beencanceled.");
  }
}

private static void AsyncOperation3(CancellationToken token) {

  boolcancellationFlag = false;
  token.Register(()=>cancellationFlag=true);
  Console.WriteLine("Starting the third task");
  for (int i=0; i<5; i++) {
    if (cancellationFlag) {
      Console.WriteLine("The third task has beencanceled.");
      return;
    }
    Thread.Sleep(TimeSpan.FromSeconds(1));
  }
  Console.WriteLine("The third task has completedsuccesfully");
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
using (var cts = new CancellationTokenSource()) {
  CancellationToken token = cts.Token;
  ThreadPool.QueueUserWorkItem(_ => AsyncOperation1(token));
  Thread.Sleep(TimeSpan.FromSeconds(2));
  cts.Cancel();
}

using (var cts = new CancellationTokenSource()) {
  CancellationToken token = cts.Token;
  ThreadPool.QueueUserWorkItem(_ => AsyncOperation2(token));
  Thread.Sleep(TimeSpan.FromSeconds(2));
  cts.Cancel();
}

using (var cts = new CancellationTokenSource()) {
  CancellationToken token = cts.Token;
  ThreadPool.QueueUserWorkItem(_ => AsyncOperation3(token));
  Thread.Sleep(TimeSpan.FromSeconds(2));
  cts.Cancel();
}

Thread.Sleep(TimeSpan.FromSeconds(2));
```

1.  运行程序。

## 它是如何工作的...

在这里，我们引入了新的`CancellationTokenSource`和`CancellationToken`构造。它们出现在.NET 4.0 中，现在已经成为实现异步操作取消过程的事实标准。由于线程池已经存在很长时间，它没有专门的 API 用于取消标记；然而，它们仍然可以使用。

在这个程序中，我们看到了三种组织取消过程的方法。第一种方法是轮询和检查`CancellationToken.IsCancellationRequested`属性。如果它被设置为`true`，这意味着我们的操作正在被取消，我们必须放弃这个操作。

第二种方法是抛出`OperationCancelledException`异常。这允许从被取消的操作内部控制取消过程，而不是从外部代码控制。

最后的选择是在线程池上注册一个**回调**，当操作被取消时将在线程池上调用。这将允许将取消逻辑链接到另一个异步操作中。

# 使用等待句柄和线程池的超时

本示例将描述如何为线程池操作实现超时，以及如何在线程池上正确等待。

## 准备就绪

要进入这个示例，您需要 Visual Studio 2012。没有其他先决条件。这个示例的源代码可以在`BookSamples\Chapter3\Recipe5`中找到。

## 如何做...

要学习如何实现超时以及如何在线程池上正确等待，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void RunOperations(TimeSpanworkerOperationTimeout) {
  using (var evt = new ManualResetEvent(false))
  using (var cts = new CancellationTokenSource()) {
    Console.WriteLine("Registering timeout operations...");
    var worker = ThreadPool.RegisterWaitForSingleObject(evt, (state, isTimedOut) => WorkerOperationWait(cts,isTimedOut), null, workerOperationTimeout, true);

    Console.WriteLine("Starting long runningoperation...");

    ThreadPool.QueueUserWorkItem(_ => WorkerOperation(cts.Token, evt));

    Thread.Sleep(workerOperationTimeout.Add(TimeSpan.FromSeconds(2)));
    worker.Unregister(evt);
  }
}

static void WorkerOperation(CancellationToken token,ManualResetEventevt) {

  for(int i=0; i<6; i++) {
    if (token.IsCancellationRequested) {
      return;
    }
    Thread.Sleep(TimeSpan.FromSeconds(1));
  }
  evt.Set();
}

static void WorkerOperationWait(CancellationTokenSource ctsbool isTimedOut) {

  if (isTimedOut) {
    cts.Cancel();
    Console.WriteLine("Worker operation timed out and wascanceled.");
  }
  else {
    Console.WriteLine("Worker operation succeded.");
  }
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
RunOperations(TimeSpan.FromSeconds(5));
RunOperations(TimeSpan.FromSeconds(7));
```

1.  运行程序。

## 它是如何工作的...

线程池还有另一个有用的方法：`ThreadPool.RegisterWaitForSingleObject`。这个方法允许我们在线程池上排队一个回调，当提供的等待句柄被信号或超时发生时，这个回调将被执行。这允许我们为线程池操作实现超时。

首先，我们在线程池上排队一个长时间运行的操作。它运行了 6 秒，然后设置了一个`ManualResetEvent`信号构造，以防它成功完成。在其他情况下，如果请求取消，操作就会被放弃。

然后，我们注册第二个异步操作，当它从`ManualResetEvent`对象接收到信号时将被调用，该对象由第一个操作设置，如果第一个操作成功完成。另一个选项是在第一个操作完成之前发生超时。如果发生这种情况，我们使用`CancellationToken`来取消第一个操作。

最后，如果我们为操作提供了 5 秒的超时，这是不够的。这是因为操作需要 6 秒才能完成，我们需要取消这个操作。所以如果我们提供了一个 7 秒的超时，这是可以接受的，操作将成功完成。

## 还有更多...

当您有大量的线程必须在阻塞状态下等待某个多线程事件构造发出信号时，这是非常有用的。我们可以使用线程池基础设施，而不是阻塞所有这些线程。它将允许释放这些线程，直到事件被设置。这对于需要可伸缩性和性能的服务器应用程序来说是一个非常重要的场景。

# 使用定时器

这个食谱将描述如何使用`System.Threading.Timer`对象在线程池上创建周期调用的异步操作。

## 准备工作

要进入这个食谱，您将需要 Visual Studio 2012。没有其他先决条件。这个食谱的源代码可以在`BookSamples\Chapter3\Recipe6`中找到。

## 如何做...

要学习如何在线程池上创建周期调用的异步操作，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下方添加以下代码片段：

```cs
static Timer _timer;

static void TimerOperation(DateTime start) {
  TimeSpan elapsed = DateTime.Now - start;
  Console.WriteLine("{0} seconds from {1}. Timer threadpool thread id: {2}", elapsed.Seconds, start,
    Thread.CurrentThread.ManagedThreadId);
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
Console.WriteLine("Press 'Enter' to stop the timer...");
DateTime start = DateTime.Now;
_timer = new Timer(_ => TimerOperation(start), null,TimeSpan.FromSeconds(1), TimeSpan.FromSeconds(2));

Thread.Sleep(TimeSpan.FromSeconds(6));

_timer.Change(TimeSpan.FromSeconds(1),TimeSpan.FromSeconds(4));

Console.ReadLine();

_timer.Dispose();
```

1.  运行程序。

## 它是如何工作的...

首先，我们创建一个新的`Timer`实例。第一个参数是一个将在线程池上执行的 lambda 表达式。我们调用`TimerOperation`方法，提供一个开始日期。我们不使用用户`state`对象，所以第二个参数是 null；然后，我们指定何时第一次运行`TimerOperation`，以及调用之间的时间间隔。因此，第一个值实际上意味着我们在一秒钟内开始第一个操作，然后每个操作运行 2 秒。

之后，我们等待 6 秒并改变我们的定时器。我们在调用`_timer.Change`方法后的一秒钟开始`TimerOperation`，然后每个运行 4 秒。

### 提示

**定时器可能比这更复杂！**

可以以更复杂的方式使用定时器。例如，我们可以仅运行定时器操作一次，提供一个`Timeout.Infinte`值的定时器周期参数。然后，在定时器异步操作中，我们能够根据一些自定义逻辑设置下一次定时器操作将被执行的时间。

最后，我们等待*Enter*键被按下并完成应用程序。当它运行时，我们可以看到自程序启动以来经过的时间。

# 使用 BackgroundWorker 组件

这个食谱描述了另一种异步编程的方法，以`BackgroundWorker`组件为例。借助这个对象，我们能够将异步代码组织成一组事件和事件处理程序。您将学习如何使用这个组件进行异步编程。

## 准备工作

要进入这个食谱，您将需要 Visual Studio 2012。没有其他先决条件。这个食谱的源代码可以在`BookSamples\Chapter3\Recipe7`中找到。

## 如何做...

要学习如何使用`BackgroundWorker`组件，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.ComponentModel;
using System.Threading;
```

1.  在`Main`方法下方添加以下代码片段：

```cs
static void Worker_DoWork(object sender, DoWorkEventArgs e) 
{
  Console.WriteLine("DoWork thread pool thread id: {0}",Thread.CurrentThread.ManagedThreadId);
  var bw = (BackgroundWorker) sender;
  for (int i=1; i<=100; i++) {

    if (bw.CancellationPending) {
      e.Cancel = true;
      return;
    }

    if (i%10 == 0) {
      bw.ReportProgress(i);
    }

    Thread.Sleep(TimeSpan.FromSeconds(0.1));
  }
  e.Result = 42;
}

static void Worker_ProgressChanged(object sender,ProgressChangedEventArgs e){

  Console.WriteLine("{0}% completed. Progress thread poolthread id: {1}", e.ProgressPercentage, Thread.CurrentThread.ManagedThreadId);
}

static void Worker_Completed(object sender,RunWorkerCompletedEventArgs e) {

  Console.WriteLine("Completed thread pool thread id: {0}",Thread.CurrentThread.ManagedThreadId);
  if (e.Error != null) {
    Console.WriteLine("Exception {0} has occured.",e.Error.Message);
  }
  else if (e.Cancelled) {
    Console.WriteLine("Operation has been canceled.");
  }
  else {
    Console.WriteLine("The answer is: {0}", e.Result);
  }
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
var bw = new BackgroundWorker();
bw.WorkerReportsProgress = true;
bw.WorkerSupportsCancellation = true;

bw.DoWork += Worker_DoWork;
bw.ProgressChanged += Worker_ProgressChanged;
bw.RunWorkerCompleted += Worker_Completed;

bw.RunWorkerAsync();

Console.WriteLine("Press C to cancel work");
do {
  if (Console.ReadKey(true).KeyChar == 'C') {
    bw.CancelAsync();
  }

}
while(bw.IsBusy);
```

1.  运行程序。

## 它是如何工作的...

程序启动时，我们创建了一个`BackgroundWorker`组件的实例。我们明确表示我们希望我们支持后台工作的操作取消和操作进度的通知。

现在，最有趣的部分出现了。与其使用线程池和委托进行操作，我们使用另一个 C#习语，称为**事件**。事件代表某种通知的一个*源*，以及一些准备好在通知到达时做出反应的*订阅者*。在我们的情况下，我们声明我们将订阅三个事件，当它们发生时，我们将调用相应的**事件处理程序**。这些是具有特别定义签名的方法，当事件通知其订阅者时将被调用。

因此，与其在一对`Begin`/`End`方法中组织异步 API，不如只是启动异步操作，然后订阅在执行此操作时可能发生的不同事件。这种方法被称为**基于事件的异步模式**（**EAP**）。这在历史上是对异步程序进行结构化的第二次尝试，现在建议使用 TPL，这将在第四章中描述，*使用任务并行库*。

因此，我们已经订阅了三个事件。其中之一是`DoWork`事件。当后台工作程序对象使用`RunWorkerAsync`方法开始异步操作时，将调用此事件的处理程序。事件处理程序将在线程池上执行，这是主要的操作点，在这里，如果请求取消，则工作将被取消，并且我们提供操作的进度信息。最后，当我们获得结果时，我们将其设置为事件参数，然后调用`RunWorkerCompleted`事件处理程序。在此方法内部，我们会找出我们的操作是否成功，或者可能出现了一些错误，或者它被取消了。

此外，`BackgroundWorker`组件实际上是用于**Windows 窗体应用程序**（**WPF**）。它的实现使得可以直接从后台工作程序事件处理程序的代码中使用 UI 控件，这与线程池的工作线程与 UI 控件的交互相比非常方便。
