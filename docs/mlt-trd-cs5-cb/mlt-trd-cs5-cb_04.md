# 第四章：使用任务并行库

在本章中，我们将深入研究一种新的异步编程范式，任务并行库。您将学习以下内容：

+   创建任务

+   执行任务的基本操作

+   将任务组合在一起

+   将 APM 模式转换为任务

+   将 EAP 模式转换为任务

+   实现取消选项

+   处理任务中的异常

+   并行运行任务

+   使用 TaskScheduler 调整任务执行

# 介绍

在之前的章节中，我们学习了什么是线程，如何使用线程，以及为什么我们需要线程池。使用线程池允许我们节省操作系统资源，但代价是降低了并行度。我们可以将线程池视为一个**抽象层**，它将线程使用的细节隐藏起来，使我们能够集中精力在程序逻辑上，而不是线程问题上。

然而，使用线程池也是复杂的。没有简单的方法从线程池工作线程中获取结果。我们需要实现自己的方法来获取结果，并且在发生异常时，我们必须正确地将其传播到原始线程。除此之外，没有简单的方法来创建一组依赖的异步操作，其中一个操作在另一个操作完成后运行。

有几次尝试解决这些问题，结果产生了异步编程模型和基于事件的异步模式，这在第三章*使用线程池*中提到。这些模式使得获取结果更容易，并且在传播异常方面做得很好，但是将异步操作组合在一起仍然需要大量的工作，并且导致了大量的代码。

为了解决所有这些问题，在.Net Framework 4.0 中引入了一种新的用于异步操作的 API。它被称为**任务并行库**（**TPL**）。它在.Net Framework 4.5 中略有改变，为了更清楚起见，我们将在我们的项目中使用.Net Framework 4.5 版本来使用最新版本的 TPL。TPL 可以被视为线程池上的另一种抽象层，它隐藏了与线程池一起工作的底层代码，使程序员无需关注，并提供了更方便和细粒度的 API。

TPL 的核心概念是任务。任务代表一个异步操作，可以以各种方式运行，使用单独的线程或不使用。我们将在本章中详细讨论所有可能性。

### 注意

默认情况下，程序员不知道任务的执行方式。TPL 通过隐藏任务的实现细节，提高了抽象级别。不幸的是，在某些情况下，这可能导致神秘的错误，比如在尝试从任务中获取结果时挂起应用程序。本章将帮助理解 TPL 底层的机制，以及如何避免以不当的方式使用它。

任务可以以不同的方式与其他任务组合。例如，我们可以同时启动几个任务，等待它们全部完成，然后运行一个任务，对所有先前任务的结果进行一些计算。与以前的模式相比，任务组合的便利 API 是 TPL 的关键优势之一。

还有几种处理任务异常的方法。由于一个任务可能由几个其他任务组成，它们又有自己的子任务，因此有一个`AggregateException`的概念。这种类型的异常包含了所有底层任务的异常，允许单独处理它们。

最后但并非最不重要的是，C# 5.0 内置了对 TPL 的支持，允许我们使用新的`await`和`async`关键字以非常流畅和舒适的方式处理任务。我们将在第五章*使用 C# 5.0*中讨论这个话题。

在本章中，我们将学习使用 TPL 执行异步操作。我们将学习任务是什么，覆盖创建任务的不同方式，以及如何将任务组合在一起。我们还将讨论如何将传统的 APM 和 EAP 模式转换为使用任务，如何正确处理异常，如何取消任务，以及如何同时处理多个任务。此外，我们将了解如何正确处理 Windows GUI 应用程序中的任务。

# 创建任务

这个配方展示了任务的基本概念。您将学习如何创建和执行任务。

## 准备工作

要按照这个配方进行，您将需要**Visual Studio 2012**。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter4\Recipe1`中找到。

## 如何操作...

要创建和执行任务，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

### 注意

这次，请确保您使用的是.Net Framework 4.5。从现在开始，我们将为每个项目使用这个版本。

![如何操作...](img/7644OT_04_01.jpg)

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void TaskMethod(string name){
  Console.WriteLine("Task {0} is running on a thread id{1}. Is thread pool thread: {2}", name,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
var t1 = new Task(() =>TaskMethod("Task 1"));
var t2 = new Task(() =>TaskMethod("Task 2"));
t2.Start();
t1.Start();
Task.Run(() =>TaskMethod("Task 3"));
Task.Factory.StartNew(() => TaskMethod("Task 4"));
Task.Factory.StartNew(() => TaskMethod("Task 5"),TaskCreationOptions.LongRunning);
Thread.Sleep(TimeSpan.FromSeconds(1));
```

1.  运行程序。

## 它是如何工作的...

当程序运行时，它使用构造函数创建两个任务。我们将 lambda 表达式作为`Action`委托传递；这允许我们向`TaskMethod`提供一个字符串参数。然后，我们使用`Start`方法运行这些任务。

### 注意

请注意，在调用这些任务的`Start`方法之前，它们不会开始执行。很容易忘记实际启动任务。

然后，我们使用`Task.Run`和`Task.Factory.StartNew`方法运行另外两个任务。不同之处在于，创建的任务立即开始工作，因此我们不需要在任务上显式调用`Start`方法。所有任务，从`Task 1`到`Task 4`，都放置在线程池工作线程上，并以未指定的顺序运行。如果多次运行程序，您会发现任务的执行顺序是不确定的。

`Task.Run`方法只是`Task.Factory.StartNew`的快捷方式，但后者有额外的选项。一般情况下，除非需要做一些特殊的事情，如`Task 5`的情况，否则使用前者方法。我们将这个任务标记为长时间运行，结果，这个任务将在一个单独的线程上运行，而不使用线程池。然而，这种行为可能会改变，取决于当前运行任务的**任务调度程序**。您将在本章的最后一个配方中了解什么是任务调度程序。

# 执行任务的基本操作

这个配方将描述如何从任务中获取结果值。我们将通过几种情景来理解在线程池或主线程上运行任务的区别。

## 准备工作

要开始这个配方，您将需要 Visual Studio 2012。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter4\Recipe2`中找到。

## 如何操作...

要执行任务的基本操作，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static Task<int>CreateTask(string name){
  return new Task<int>(() =>TaskMethod(name));
}

static int TaskMethod(string name){
  Console.WriteLine("Task {0} is running on a thread id{1}. Is thread pool thread: {2}",name,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
  Thread.Sleep(TimeSpan.FromSeconds(2));
  return 42;
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
TaskMethod("Main Thread Task");
Task<int> task = CreateTask("Task 1");
task.Start();
int result = task.Result;
Console.WriteLine("Result is: {0}", result);

task = CreateTask("Task 2");
task.RunSynchronously();
result = task.Result;
Console.WriteLine("Result is: {0}", result);

task = CreateTask("Task 3");
task.Start();

while (!task.IsCompleted){
  Console.WriteLine(task.Status);
  Thread.Sleep(TimeSpan.FromSeconds(0.5));
} 

Console.WriteLine(task.Status);
result = task.Result;
Console.WriteLine("Result is: {0}", result);
```

1.  运行程序。

## 它是如何工作的...

首先，我们运行`TaskMethod`，而不将其包装成任务。结果，它是同步执行的，为我们提供了关于主线程的信息。显然，这不是一个线程池线程。

然后我们运行`Task 1`，使用`Start`方法启动它并等待结果。这个任务将放在线程池上，主线程会等待并被阻塞，直到任务返回。

我们对`Task 2`做同样的操作，只是我们使用`RunSynchronously()`方法来运行它。这个任务将在主线程上运行，我们得到的输出与当我们只是同步调用`TaskMethod`时完全相同。这是一个非常有用的优化，允许我们避免对非常短暂的操作使用线程池。

我们以与`Task 1`相同的方式运行`Task 3`，但是不阻塞主线程，只是旋转，打印出任务状态，直到任务完成。这显示了几个任务状态，分别是`Created`，`Running`和`RanToCompletion`。

# 将任务组合在一起

这个示例将展示如何设置相互依赖的任务。我们将学习如何创建一个任务，在父任务完成后运行。此外，我们将发现一种节省线程使用的可能性，用于非常短暂的任务。

## 准备工作

要执行此示例，您需要 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`BookSamples\Chapter4\Recipe3`中找到。

## 如何做...

要将任务组合在一起，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# **Console Application**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static int TaskMethod(string name, int seconds){
  Console.WriteLine("Task {0} is running on a thread id
    {1}. Is thread pool thread: {2}", name,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
  Thread.Sleep(TimeSpan.FromSeconds(seconds));
  return 42 * seconds;
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
var firstTask = new Task<int>(() =>TaskMethod("First Task",3));
var secondTask = new Task<int>(() =>TaskMethod("SecondTask", 2));

firstTask.ContinueWith(
  t =>Console.WriteLine("The first answer is {0}. Thread id{1}, is thread pool thread: {2}", t.Result,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread),TaskContinuationOptions.OnlyOnRanToCompletion);

firstTask.Start();
secondTask.Start();

Thread.Sleep(TimeSpan.FromSeconds(4));

Task continuation = secondTask.ContinueWith(
  t =>Console.WriteLine("The second answer is {0}. Threadid {1}, is thread pool thread: {2}", t.Result,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread),TaskContinuationOptions.OnlyOnRanToCompletion |TaskContinuationOptions.ExecuteSynchronously);

continuation.GetAwaiter().OnCompleted(
  () =>Console.WriteLine("Continuation Task Completed!Thread id {0}, is thread pool thread: {1}",Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread));

Thread.Sleep(TimeSpan.FromSeconds(2));
Console.WriteLine();

firstTask = new Task<int>(() => {varinnerTask = Task.Factory.StartNew(() =>TaskMethod("Second Task", 5), TaskCreationOptions.AttachedToParent);
  innerTask.ContinueWith(t =>TaskMethod("Third Task", 2),TaskContinuationOptions.AttachedToParent);
  return TaskMethod("First Task", 2);
});

firstTask.Start();

while (!firstTask.IsCompleted){
  Console.WriteLine(firstTask.Status);
  Thread.Sleep(TimeSpan.FromSeconds(0.5));
}
Console.WriteLine(firstTask.Status);

Thread.Sleep(TimeSpan.FromSeconds(10));
```

1.  运行程序。

## 它是如何工作的...

当主程序启动时，我们创建两个任务，对于第一个任务，我们设置了一个**continuation**（在前一个任务完成后运行的代码块）。然后我们启动这两个任务并等待 4 秒，这足够让两个任务都完成。然后我们对第二个任务运行另一个 continuation，并尝试通过指定`TaskContinuationOptions.ExecuteSynchronously`选项同步执行它。当 continuation 非常短暂时，这是一种有用的技术，它将更快地在主线程上运行而不是放在线程池中。我们能够做到这一点是因为第二个任务在那时已经完成。如果我们注释掉 4 秒的`Thread.Sleep`方法，我们将看到这段代码将被放在线程池中，因为我们还没有从前一个任务得到结果。

最后，我们以稍微不同的方式为前一个 continuation 定义一个 continuation，使用新的`GetAwaiter`和`OnCompleted`方法。这些方法旨在与 C# 5.0 语言的异步机制一起使用。我们将在第五章中详细介绍这个主题，*使用 C# 5.0*。

演示的最后部分是关于父子任务关系。我们创建一个新任务，同时运行这个任务，通过提供`TaskCreationOptions.AttachedToParent`选项来运行所谓的子任务。

### 提示

在运行父任务时必须创建子任务以正确附加到父任务！

这意味着父任务*不会完成*直到所有子任务完成其工作。我们还能够在子任务上运行 continuations，提供`TaskContinuationOptions.AttachedToParent`选项。这个 continuation 也会影响父任务，并且直到最后一个子任务结束才会完成。

# 将 APM 模式转换为任务

在这个示例中，我们将看到如何将老式的 APM API 转换为任务。有不同情况的示例可能发生在转换过程中。

## 准备工作

要开始这个示例，您需要 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`BookSamples\Chapter4\Recipe4`中找到。

## 如何做...

要将 APM 模式转换为任务，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# **Console Application**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
private delegate string AsynchronousTask(stringthreadName);
private delegate string IncompatibleAsynchronousTask(outint threadId);

private static void Callback(IAsyncResultar){
  Console.WriteLine("Starting a callback...");
  Console.WriteLine("State passed to a callback: {0}",ar.AsyncState);
  Console.WriteLine("Is thread pool thread: {0}",Thread.CurrentThread.IsThreadPoolThread);
  Console.WriteLine("Thread pool worker thread id: {0}",Thread.CurrentThread.ManagedThreadId);
}

private static string Test(string threadName){
  Console.WriteLine("Starting...");
  Console.WriteLine("Is thread pool thread: {0}",Thread.CurrentThread.IsThreadPoolThread);
  Thread.Sleep(TimeSpan.FromSeconds(2));
  Thread.CurrentThread.Name = threadName;
  return string.Format("Thread name: {0}",Thread.CurrentThread.Name);
}

private static string Test(out int threadId){
  Console.WriteLine("Starting...");
  Console.WriteLine("Is thread pool thread: {0}",Thread.CurrentThread.IsThreadPoolThread);
  Thread.Sleep(TimeSpan.FromSeconds(2));
  threadId = Thread.CurrentThread.ManagedThreadId;
  return string.Format("Thread pool worker thread id was:{0}", threadId);
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
int threadId;
AsynchronousTask d = Test;
IncompatibleAsynchronousTask e = Test;

Console.WriteLine("Option 1");
Task<string> task = Task<string>.Factory.FromAsync(
  d.BeginInvoke("AsyncTaskThread", Callback, "a delegateasynchronous call"), d.EndInvoke);

task.ContinueWith(t =>Console.WriteLine("Callback isfinished, now running a continuation! Result: {0}",t.Result));

while (!task.IsCompleted){
  Console.WriteLine(task.Status);
  Thread.Sleep(TimeSpan.FromSeconds(0.5));
}
Console.WriteLine(task.Status);
Thread.Sleep(TimeSpan.FromSeconds(1));

Console.WriteLine("----------------------------------------");
Console.WriteLine();
Console.WriteLine("Option 2");

task = Task<string>.Factory.FromAsync(
  d.BeginInvoke, d.EndInvoke, "AsyncTaskThread", "adelegate asynchronous call");
task.ContinueWith(t =>Console.WriteLine("Task is completed,now running a continuation! Result: {0}",t.Result));
while (!task.IsCompleted){
  Console.WriteLine(task.Status);
  Thread.Sleep(TimeSpan.FromSeconds(0.5));
}
Console.WriteLine(task.Status);
Thread.Sleep(TimeSpan.FromSeconds(1));

Console.WriteLine("------------------------------------------");
Console.WriteLine();
Console.WriteLine("Option 3");

IAsyncResult ar = e.BeginInvoke(out threadId, Callback, "adelegate asynchronous call");
ar = e.BeginInvoke(out threadId, Callback, "a delegateasynchronous call");
task = Task<string>.Factory.FromAsync(ar, _ =>e.EndInvoke(out threadId, ar));
task.ContinueWith(t =>
  Console.WriteLine("Task is completed, now running acontinuation! Result: {0}, ThreadId: {1}",t.Result, threadId));

while (!task.IsCompleted){
  Console.WriteLine(task.Status);
  Thread.Sleep(TimeSpan.FromSeconds(0.5));
}
Console.WriteLine(task.Status);

Thread.Sleep(TimeSpan.FromSeconds(1));
```

1.  运行程序。

## 工作原理...

在这里，我们定义了两种类型的委托；其中一种使用了`out`参数，因此与将 APM 模式转换为任务的标准 TPL API 不兼容。然后我们有三个这样转换的示例。

将 APM 转换为 TPL 的关键点是`Task<T>.Factory.FromAsync`方法，其中`T`是异步操作的结果类型。该方法有几种重载；在第一种情况下，我们传递`IAsyncResult`和`Func<IAsyncResult, string>`，这是一个接受`IAsyncResult`实现并返回一个字符串的方法。由于第一个委托类型提供了与此签名兼容的`EndMethod`，因此我们可以毫无问题地将这个委托异步调用转换为任务。

在第二个示例中，我们做了几乎相同的事情，但使用了不同的`FromAsync`方法重载，它不允许指定在异步委托调用完成后将执行的回调。我们可以用延续来替换这个，但如果回调很重要，我们可以使用第一个示例。

最后一个示例展示了一个小技巧。这次，`IncompatibleAsynchronousTask`委托的`EndMethod`使用了`out`参数，并且与任何`FromAsync`方法重载都不兼容。然而，很容易将`EndMethod`调用包装成适用于任务工厂的 lambda 表达式。

为了查看底层任务的情况，我们在等待异步操作结果时打印其状态。我们看到第一个任务的状态是`WaitingForActivation`，这意味着任务实际上还没有被 TPL 基础架构启动。

# 将 EAP 模式转换为任务

本教程将描述如何将基于事件的异步操作转换为任务。在本教程中，您将找到一个适用于.NET Framework 类库中的每个基于事件的异步 API 的可靠模式。

## 准备工作

要开始本教程，您需要 Visual Studio 2012。没有其他先决条件。本教程的源代码可以在`BookSamples\Chapter4\Recipe5`中找到。

## 如何做...

要将 EAP 模式转换为任务，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.ComponentModel;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static int TaskMethod(string name, int seconds){
  Console.WriteLine("Task {0} is running on a thread id{1}. Is thread pool thread: {2}", name,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
  Thread.Sleep(TimeSpan.FromSeconds(seconds));
  return 42 * seconds;
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
var tcs = new TaskCompletionSource<int>();

var worker = new BackgroundWorker();
worker.DoWork += (sender, eventArgs) =>
{
  eventArgs.Result = TaskMethod("Background worker", 5);
};

worker.RunWorkerCompleted += (sender, eventArgs) =>{
  if (eventArgs.Error != null) {
    tcs.SetException(eventArgs.Error);
  }
  else if (eventArgs.Cancelled) {
    tcs.SetCanceled();
  }
    else {
      tcs.SetResult((int)eventArgs.Result);
    }
};

worker.RunWorkerAsync();

int result = tcs.Task.Result;

Console.WriteLine("Result is: {0}", result);
```

1.  运行程序。

## 工作原理...

这是一个非常简单而优雅的将 EAP 模式转换为任务的例子。关键点是使用`TaskCompletionSource<T>`类型，其中`T`是异步操作的结果类型。

同样重要的是不要忘记将`tcs.SetResult`方法调用包装在`try`-`catch`块中，以确保错误信息始终设置到任务完成源对象中。也可以使用`TrySetResult`方法代替`SetResult`，以确保结果已成功设置。

# 实现取消选项

本教程是关于为基于任务的异步操作实现取消过程。我们将学习如何正确使用取消令牌来处理任务，以及如何在任务实际运行之前找出任务是否已取消。

## 准备工作

要开始本教程，您需要 Visual Studio 2012。没有其他先决条件。本教程的源代码可以在`BookSamples\Chapter4\Recipe6`中找到。

## 如何做...

要为基于任务的异步操作实现取消选项，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
private static int TaskMethod(string name, int seconds,CancellationToken token){

  Console.WriteLine("Task {0} is running on a thread id{1}. Is thread pool thread: {2}", name,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
  for (int i = 0; i< seconds; i ++) {
    Thread.Sleep(TimeSpan.FromSeconds(1));
    if (token.IsCancellationRequested)
      return -1;
  }
  return 42*seconds;
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
var cts = new CancellationTokenSource();
var longTask = new Task<int>(() =>TaskMethod("Task 1", 10,cts.Token), cts.Token);
Console.WriteLine(longTask.Status);
cts.Cancel();
Console.WriteLine(longTask.Status);
Console.WriteLine("First task has been cancelled beforeexecution");
cts = new CancellationTokenSource();
longTask = new Task<int>(() =>TaskMethod("Task 2", 10,cts.Token), cts.Token);
longTask.Start();
for (int i = 0; i< 5; i++ ){
  Thread.Sleep(TimeSpan.FromSeconds(0.5));
  Console.WriteLine(longTask.Status);
}
cts.Cancel();
for (int i = 0; i< 5; i++){
  Thread.Sleep(TimeSpan.FromSeconds(0.5));
  Console.WriteLine(longTask.Status);
}

Console.WriteLine("A task has been completed with result{0}.", longTask.Result);
```

1.  运行程序。

## 工作原理...

这是另一个非常简单的例子，说明如何为 TPL 任务实现取消选项，因为你已经熟悉我们在第三章中讨论的取消标记概念，*使用线程池*。

首先，让我们仔细看看 `longTask` 创建代码。我们将一次性传递一个取消标记给底层任务，然后第二次传递给任务构造函数。*为什么我们需要两次提供这个标记？*

答案是，如果我们在任务实际开始之前取消了任务，它的 TPL 基础结构负责处理取消，因为我们的代码根本不会执行。我们知道第一个任务被取消了，通过获取它的状态。如果我们尝试在这个任务上调用 `Start` 方法，我们将得到 `InvalidOperationException`。

然后，我们从我们自己的代码中处理取消过程。这意味着我们现在完全负责取消过程，而在我们取消任务后，它的状态仍然是 `RanToCompletion`，因为从 TPL 的角度来看，任务正常完成了它的工作。在每种情况下理解责任差异非常重要。

# 处理任务中的异常

这个步骤描述了在异步任务中处理异常的非常重要的主题。我们将讨论从任务中抛出的异常发生的不同方面以及如何获取它们的信息。

## 准备就绪

要按照这个步骤，你需要 Visual Studio 2012。没有其他先决条件。这个步骤的源代码可以在 `BookSamples\Chapter4\Recipe7` 中找到。

## 如何做...

要处理任务中的异常，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# **控制台应用程序** 项目。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
```

1.  在 `Main` 方法下面添加以下代码片段：

```cs
static int TaskMethod(string name, int seconds){
  Console.WriteLine("Task {0} is running on a thread id{1}. Is thread pool thread: {2}", name,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
  Thread.Sleep(TimeSpan.FromSeconds(seconds));
  throw new Exception("Boom!");
  return 42 * seconds;
}
```

1.  在 `Main` 方法中添加以下代码片段：

```cs
Task<int> task;
try{
  task = Task.Run(() =>TaskMethod("Task 1", 2));
  int result = task.Result;
  Console.WriteLine("Result: {0}", result);
}
catch (Exception ex){
  Console.WriteLine("Exception caught: {0}", ex);
}
Console.WriteLine("----------------------------------------------");
Console.WriteLine();

try{
  task = Task.Run(() =>TaskMethod("Task 2", 2));
  int result = task.GetAwaiter().GetResult();
  Console.WriteLine("Result: {0}", result);
}
catch (Exception ex){
  Console.WriteLine("Exception caught: {0}", ex);
}
Console.WriteLine("----------------------------------------------");
Console.WriteLine();

var t1 = new Task<int>(() =>TaskMethod("Task 3", 3));
var t2 = new Task<int>(() =>TaskMethod("Task 4", 2));
var complexTask = Task.WhenAll(t1, t2);
var exceptionHandler = complexTask.ContinueWith(t =>Console.WriteLine("Exception caught: {0}", t.Exception),TaskContinuationOptions.OnlyOnFaulted);
t1.Start();
t2.Start();

Thread.Sleep(TimeSpan.FromSeconds(5));
```

1.  运行程序。

## 它是如何工作的...

程序启动时，我们创建一个任务，并尝试同步获取任务结果。`Result` 属性的 `Get` 部分使当前线程等待任务完成，并将异常传播到当前线程。在这种情况下，我们很容易在 catch 块中捕获异常，但这个异常是一个名为 `AggregateException` 的包装异常。在这种情况下，它只包含一个异常，因为只有一个任务抛出了这个异常，可以通过访问 `InnerException` 属性来获取底层异常。

第二个例子大部分相同，但是为了访问任务结果，我们使用 `GetAwaiter` 和 `GetResult` 方法。在这种情况下，我们没有包装异常，因为它被 TPL 基础结构解包了。我们一次性获得原始异常，如果只有一个底层任务，这是非常舒适的。

最后一个例子展示了我们有两个任务抛出异常的情况。为了处理异常，我们现在使用一个继续，只有在前置任务以异常结束时才执行。通过为继续提供 `TaskContinuationOptions.OnlyOnFaulted` 选项来实现这种行为。结果，我们打印出 `AggregateException`，并且其中包含来自两个任务的两个内部异常。

## 还有更多...

由于任务可能以非常不同的方式连接，因此生成的 `AggregateException` 异常可能包含其他聚合异常以及通常的异常。这些内部聚合异常本身可能包含其中的其他聚合异常。

为了摆脱这些包装器，我们应该使用根聚合异常的 `Flatten` 方法。它将返回层次结构中每个子聚合异常的所有内部异常的集合。

# 并行运行任务

这个示例展示了如何处理同时运行的许多异步任务。我们将学习如何在所有任务完成或任何正在运行的任务必须完成它们的工作时有效地得到通知。

## 准备工作

要开始这个示例，你需要 Visual Studio 2012。没有其他先决条件。这个示例的源代码可以在`BookSamples\Chapter4\Recipe8`中找到。

## 如何做...

要并行运行任务，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static int TaskMethod(string name, int seconds){
  Console.WriteLine("Task {0} is running on a thread id{1}. Is thread pool thread: {2}", name,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
  Thread.Sleep(TimeSpan.FromSeconds(seconds));
  return 42 * seconds;
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
var firstTask = new Task<int>(() =>TaskMethod("First Task",3));
var secondTask = new Task<int>(() =>TaskMethod("SecondTask", 2));
var whenAllTask = Task.WhenAll(firstTask, secondTask);

whenAllTask.ContinueWith(t =>
  Console.WriteLine("The first answer is {0}, the second is{1}", t.Result[0], t.Result[1]),TaskContinuationOptions.OnlyOnRanToCompletion);

firstTask.Start();
secondTask.Start();

Thread.Sleep(TimeSpan.FromSeconds(4));

var tasks = new List<Task<int>>();
for (int i = 1; i< 4; i++)
{
  int counter = i;
  var task = new Task<int>(() =>TaskMethod(string.Format("Task {0}", counter), counter));
  tasks.Add(task);
  task.Start();
}

while (tasks.Count> 0){
  var completedTask = Task.WhenAny(tasks).Result;
  tasks.Remove(completedTask);
  Console.WriteLine("A task has been completed with result{0}.", completedTask.Result);
}

Thread.Sleep(TimeSpan.FromSeconds(1));
```

1.  运行程序。

## 工作原理...

程序启动时，我们创建两个任务，然后借助`Task.WhenAll`方法创建一个第三个任务，该任务将在所有任务完成后完成。结果任务为我们提供了一个答案数组，其中第一个元素保存第一个任务的结果，第二个元素保存第二个结果，依此类推。

然后，我们创建另一个任务列表，并使用`Task.WhenAny`方法等待其中任何一个任务完成。在我们有一个完成的任务后，我们将其从列表中移除，并继续等待其他任务完成，直到列表为空。这种方法对于获取任务的完成进度或在运行任务时使用超时非常有用。例如，我们等待一些任务，其中一个任务正在计算超时。如果这个任务首先完成，我们就取消那些尚未完成的任务。

# 使用 TaskScheduler 调整任务执行

这个示例描述了处理任务的另一个非常重要的方面，即从异步代码中正确处理 UI 的方法。我们将学习任务调度程序是什么，为什么它如此重要，它如何损害我们的应用程序，以及如何使用它来避免错误。

## 准备工作

要完成这个示例，你需要 Visual Studio 2012。没有其他先决条件。这个示例的源代码可以在`BookSamples\Chapter4\Recipe9`中找到。

## 如何做...

通过使用`TaskScheduler`调整任务执行，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# WPF 应用程序项目。这一次，我们将需要一个带有消息循环的 UI 线程，这在控制台应用程序中是不可用的。

1.  在`MainWindow.xaml`文件中，在一个网格元素内添加以下标记（即在`<Grid>`和`</Grid>`标记之间）：

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

1.  在`MainWindow`构造函数下面添加以下代码片段：

```cs
void ButtonSync_Click(object sender, RoutedEventArgs e){
  ContentTextBlock.Text = string.Empty;
  try {
    //string result = TaskMethod(TaskScheduler.//FromCurrentSynchronizationContext()).Result;
    string result = TaskMethod().Result;
    ContentTextBlock.Text = result;
  }
  catch (Exception ex) {
    ContentTextBlock.Text = ex.InnerException.Message;
  }
}

void ButtonAsync_Click(object sender, RoutedEventArgs e) {
  ContentTextBlock.Text = string.Empty;
  Mouse.OverrideCursor = Cursors.Wait;
  Task<string> task = TaskMethod();
  task.ContinueWith(t => {
    ContentTextBlock.Text = t.Exception.InnerException.Message;
    Mouse.OverrideCursor = null;
  }, 
  CancellationToken.None, TaskContinuationOptions.OnlyOnFaulted,
  TaskScheduler.FromCurrentSynchronizationContext());
}

void ButtonAsyncOK_Click(object sender, RoutedEventArgs e){
  ContentTextBlock.Text = string.Empty;
  Mouse.OverrideCursor = Cursors.Wait;
  Task<string> task = TaskMethod(TaskScheduler.FromCurrentSynchronizationContext());
  task.ContinueWith(t =>Mouse.OverrideCursor = null,
    CancellationToken.None,
    TaskContinuationOptions.None,
    TaskScheduler.FromCurrentSynchronizationContext());
}

Task<string> TaskMethod() {
  return TaskMethod(TaskScheduler.Default);
}

Task<string> TaskMethod(TaskScheduler scheduler) {
  Task delay = Task.Delay(TimeSpan.FromSeconds(5));

  return delay.ContinueWith(t => {
    string str = string.Format("Task is running on a threadid {0}. Is thread pool thread: {1}",Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
    ContentTextBlock.Text = str;
    return str;
  }, scheduler);
}
```

1.  运行程序。

## 工作原理...

在这里，我们遇到了许多新的东西。首先，我们创建了一个 WPF 应用程序，而不是控制台应用程序。这是必要的，因为我们需要一个用户界面线程和消息循环来演示异步运行任务的不同选项。

有一个非常重要的抽象叫做`TaskScheduler`。这个组件实际上负责任务的执行方式。默认的任务调度程序将任务放在线程池工作线程上。这是最常见的情况，也不奇怪它是 TPL 中的默认选项。我们还知道如何同步运行任务，以及如何将它们附加到父任务以一起运行。现在让我们看看我们可以用任务做什么。

程序启动时，我们创建一个带有三个按钮的窗口。第一个按钮调用同步任务执行。代码放在`ButtonSync_Click`方法中。当任务运行时，即使我们无法移动应用程序窗口。用户界面在任务运行时完全冻结，直到任务完成之前，用户界面线程无法响应任何消息循环。这是 GUI Windows 应用程序的一个常见的不良实践，我们需要找到一种解决这个问题的方法。

第二个问题是，我们试图从另一个线程访问 UI 控件。图形用户界面控件从未设计为从多个线程中使用，并且为了避免可能的错误，不允许您从创建它的线程之外的线程访问这些组件。当我们尝试这样做时，我们会收到异常，并且异常消息将在 5 秒钟后打印在主窗口中。

为了解决第一个问题，我们尝试异步运行任务。这就是第二个按钮的作用；其中的代码放在`ButtonAsync_Click`方法中。如果在调试器下运行任务，您将看到它被放置在线程池中，最后，我们将得到相同的异常。然而，用户界面在任务运行时始终保持响应。这是一件好事，但我们需要摆脱异常。

我们已经做到了！为了输出错误消息，使用了`TaskScheduler.FromCurrentSynchronizationContext`选项提供了一个继续。如果不这样做，我们将看不到错误消息，因为我们会得到与任务内部发生的相同异常。此选项指示 TPL 基础结构将代码放在 UI 线程的继续中，并借助 UI 线程消息循环异步运行它。这解决了从另一个线程访问 UI 控件的问题，但仍然保持了我们的 UI 响应性。

要检查这是否属实，我们按下最后一个按钮，运行`ButtonAsyncOK_Click`方法中的代码。唯一不同的是，我们为我们的任务提供了 UI 线程任务调度程序。任务完成后，您将看到它以异步方式在 UI 线程上运行。UI 保持响应，并且即使等待光标处于活动状态，也可以按下另一个按钮。

然而，对于在 UI 线程上运行任务有一些技巧。如果我们回到同步任务代码，并取消注释使用 UI 线程任务调度程序获取结果的行，我们将永远得不到任何结果。这是一个经典的死锁情况：我们正在将操作调度到 UI 线程的队列中，而 UI 线程等待此操作完成，但当它等待时，它无法运行操作，这将永远不会结束（甚至不会开始）。如果在任务上调用`Wait`方法，也会发生这种情况。为了避免死锁，永远不要在计划为 UI 线程的任务上使用同步操作；只使用`ContinueWith`，或者来自 C# 5.0 的`async`/`await`。
