# 第五章。使用 C# 5.0

在本章中，我们将研究 C# 5.0 编程语言中的本机异步编程支持。您将了解以下内容：

+   使用 await 运算符获取异步任务结果

+   在 lambda 表达式中使用 await 运算符

+   使用 await 运算符与随后的异步任务

+   使用 await 运算符执行并行异步任务

+   处理异步操作中的异常

+   避免使用捕获的同步上下文

+   解决异步 void 方法的问题

+   设计自定义可等待类型

+   使用动态类型与 await

# 介绍

到目前为止，我们了解了来自 Microsoft 的最新异步编程基础设施——任务并行库。它允许我们以模块化的方式设计程序，将不同的异步操作组合在一起。

不幸的是，当阅读这样的程序时，仍然很难理解实际的程序流程。在一个大型程序中，将会有许多任务和依赖于彼此的延续，运行其他延续的延续，用于异常处理的延续，它们都聚集在程序代码中的非常不同的地方。因此，理解哪个操作先进行，接下来发生什么的顺序成为一个非常具有挑战性的问题。

另一个需要注意的问题是要查看是否将适当的同步上下文传播到可能触及用户界面控件的每个异步任务。只有从 UI 线程才允许使用这些控件；否则，我们将得到一个多线程访问异常。

谈到异常，我们还必须使用单独的延续任务来处理发生在前置异步操作或操作中的错误。这反过来导致了复杂的错误处理代码，分散在代码的不同部分，彼此之间没有逻辑关联。

为了解决这些问题，C# 5.0 的作者引入了称为**异步函数**的新语言增强功能。它们确实使异步编程变得简单，但同时，它是 TPL 的高级抽象。正如我们在第四章中提到的，*使用任务并行库*，抽象隐藏了重要的实现细节，并使异步编程更加容易，但却剥夺了程序员的许多重要内容。了解异步函数背后的概念对于创建健壮和可扩展的应用程序非常重要。

要创建一个异步函数，首先要用`async`关键字标记一个方法。在没有这样做之前，不可能拥有带有 async 属性或事件访问器方法和构造函数。代码将如下所示：

```cs
async Task<string> GetStringAsync()
{
  await Task.Delay(TimeSpan.FromSeconds(2));
  return "Hello, World!";
}
```

另一个重要的事实是，异步函数必须返回`Task`或`Task<T>`类型。可以有`async void`方法，但最好使用`async Task`方法。只有在应用程序中使用顶层 UI 控件事件处理程序时，才能使用`async void`函数。

在标记有`async`关键字的方法内部，可以使用`await`运算符。该运算符与 TPL 中的任务一起工作，并获取任务内部异步操作的结果。详细内容将在本章后面介绍。您不能在`async`方法之外使用`await`运算符；这将导致编译错误。此外，异步函数应该至少在其代码中有一个`await`运算符。但这只会导致编译警告，而不是错误。

重要的是要注意，在`await`调用的行之后，此方法立即返回。在同步执行的情况下，执行线程将被阻塞 2 秒，然后返回结果。在这里，我们在返回一个工作线程到线程池的同时异步等待，立即在执行`await`操作符后返回一个工作线程。2 秒后，我们再次从线程池中获取工作线程，并在其上运行其余的异步方法。这使我们能够在这 2 秒内重复使用这个工作线程来做一些其他工作，这对应用程序的可伸缩性非常重要。通过异步函数的帮助，我们有一个线性的程序控制流，但它仍然是异步的。这既非常舒适又非常令人困惑。本章的食谱将帮助您学习异步函数的每个重要方面。

### 注意

根据我的经验，如果程序中有两个连续的`await`操作符，人们普遍存在一个误解。许多人认为，如果我们在一个异步操作之后使用`await`函数，它们会并行运行。然而，它们实际上是顺序运行的；第二个操作只有在第一个操作完成后才开始。记住这一点非常重要，在本章的后面，我们将详细讨论这个话题。

在 C# 5.0 中使用`async`和`await`存在一些限制。例如，不可能将控制台应用程序的`Main`方法标记为`async`；您不能在`catch`、`finally`、`lock`或`unsafe`块中使用`await`操作符。异步函数上不允许有`ref`和`out`参数。还有更多微妙之处，但这些是主要的要点。

异步函数在幕后由 C#编译器转换为复杂的程序构造。我故意不会详细描述这一点；生成的代码与另一个 C#构造，称为**迭代器**，非常相似，并且实现为一种状态机。由于许多开发人员几乎在每个方法上都开始使用`async`修饰符，我想强调的是，如果一个方法不打算以异步或并行方式使用，那么将方法标记为`async`是没有意义的。调用`async`方法会带来显著的性能损失，通常方法调用将比标记为`async`关键字的相同方法快 40 到 50 倍。请注意这一点。

在本章中，我们将学习如何使用 C# 5.0 的`async`和`await`关键字来处理异步操作。我们将讨论如何顺序和并行等待异步操作。我们将讨论如何在 lambda 表达式中使用`await`，如何处理异常，以及在使用`async void`方法时如何避免陷阱。最后，我们将深入探讨同步上下文传播，并学习如何创建自己的可等待对象，而不是使用任务。

# 使用`await`操作符获取异步任务结果

这个食谱介绍了使用异步函数的基本场景。我们将比较如何使用 TPL 和`await`操作符获取异步操作结果。

## 准备就绪

要按照这个食谱，您需要 Visual Studio 2012。没有其他先决条件。此食谱的源代码可以在`BookSamples\Chapter5\Recipe1`中找到。

## 如何做...

使用`await`操作符获取异步任务结果的步骤如下：

1.  启动 Visual Studio 2012。创建一个新的 C# **控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static Task AsynchronyWithTPL()
{
  Task<string> t = GetInfoAsync("Task 1");
  Task t2 = t.ContinueWith(task => Console.WriteLine(t.Result), TaskContinuationOptions.NotOnFaulted);
  Task t3 = t.ContinueWith(task => Console.WriteLine(t.Exception.InnerException), TaskContinuationOptions.OnlyOnFaulted);

  return Task.WhenAny(t2, t3);
}

async static Task AsynchronyWithAwait()
{
  try
  {
    string result = await GetInfoAsync("Task 2");
    Console.WriteLine(result);
  }
  catch (Exception ex)
  {
    Console.WriteLine(ex);
  }
}

async static Task<string> GetInfoAsync(string name)
{
  await Task.Delay(TimeSpan.FromSeconds(2));
  //throw new Exception("Boom!");

  return string.Format("Task {0} is running on a thread id{1}. Is thread pool thread: {2}", name,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
Task t = AsynchronyWithTPL();
t.Wait();

t = AsynchronyWithAwait();
t.Wait();
```

1.  运行程序。

## 工作原理...

当程序运行时，我们运行两个异步操作。其中一个是标准的 TPL 驱动代码，另一个使用新的`async`和`await` C#特性。`AsynchronyWithTPL`方法启动一个运行 2 秒钟的任务，然后返回一个包含有关工作线程信息的字符串。然后，我们定义一个继续打印异步操作结果的操作，另一个用于在发生错误时打印异常详细信息。最后，我们返回表示一个继续任务的任务，并在`Main`方法中等待其完成。

在`AsynchronyWithAwait`方法中，我们通过使用`await`与任务实现了相同的结果。就好像我们只是编写了普通的同步代码-我们从任务中获取结果，打印结果，并在任务完成时捕获异常。关键区别在于我们实际上有一个异步程序。在使用`await`后立即，C#创建了一个任务，该任务具有在`await`运算符之后的所有剩余代码的继续任务，并处理异常传播。然后，我们将此任务返回给`Main`方法，并等待其完成。

### 注意

请注意，根据底层异步操作的性质和当前的同步上下文，执行异步代码的确切方式可能有所不同。我们将在本章后面解释这一点。

因此，我们可以看到程序的第一部分和第二部分在概念上是等价的，但在第二部分中，C#编译器隐式地处理异步代码。实际上，它甚至比第一部分更复杂，我们将在本章的接下来的几个食谱中详细介绍。

请记住，在诸如 Windows GUI 或 ASP.NET 之类的环境中，不建议使用`Task.Wait`和`Task.Result`方法。如果程序员对代码的实际情况不是 100%了解，这可能会导致死锁。这在第四章的*使用任务并行库*中的*使用 TaskScheduler 调整任务执行*食谱中有所说明，当我们在 WPF 应用程序中使用`Task.Result`时。

要测试异常处理的工作原理，只需取消注释`GetInfoAsync`方法中的`throw new Exception`行。

# 在 lambda 表达式中使用 await 运算符

这个食谱将展示如何在 lambda 表达式中使用`await`。我们将编写一个使用`await`的匿名方法，并异步地获得方法执行的结果。

## 准备工作

要按照这个食谱进行操作，您需要 Visual Studio 2012。没有其他先决条件。此食谱的源代码可以在`BookSamples\Chapter5\Recipe2`中找到。

## 如何做...

要编写一个使用`await`的匿名方法，并通过在 lambda 表达式中使用`await`运算符异步地获得方法执行的结果，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
async static Task AsynchronousProcessing()
{
  Func<string, Task<string>> asyncLambda = async name => {
    await Task.Delay(TimeSpan.FromSeconds(2));
    return string.Format("Task {0} is running on a threadid {1}. Is thread pool thread: {2}", name,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
  };

  string result = await asyncLambda("async lambda");

  Console.WriteLine(result);
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
Task t = AsynchronousProcessing();
t.Wait();
```

1.  运行程序。

## 工作原理...

首先，我们将异步函数移到`AsynchronousProcessing`方法中，因为我们不能在`Main`方法中使用`async`。然后，我们使用`async`关键字描述一个 lambda 表达式。由于任何 lambda 表达式的类型不能从 lambda 本身推断出来，我们必须明确地向 C#编译器指定其类型。在我们的情况下，类型意味着我们的 lambda 接受一个字符串参数，并返回一个`Task<string>`对象。

然后，我们定义 lambda 表达式的主体。一个异常是，该方法被定义为返回一个`Task<string>`对象，但实际上我们返回一个字符串，并且没有编译错误！C#编译器会自动生成一个任务并为我们返回它。

最后一步是等待异步 lambda 表达式的执行并打印出结果。

# 使用 await 操作符进行连续异步任务的执行

这个步骤将展示当代码中有几个连续的`await`方法时，程序流程是如何的。我们将学习如何阅读带有`await`方法的代码，并理解为什么`await`调用是一个异步操作。

## 准备工作

要按照这个步骤，你需要 Visual Studio 2012。没有其他先决条件。这个步骤的源代码可以在`BookSamples\Chapter5\Recipe3`找到。

## 如何做...

理解在连续的`await`方法存在的情况下程序流程，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static Task AsynchronyWithTPL()
{
  var containerTask = new Task(() => { 
    Task<string> t = GetInfoAsync("TPL 1");
    t.ContinueWith(task => {
      Console.WriteLine(t.Result);
      Task<string> t2 = GetInfoAsync("TPL 2");
      t2.ContinueWith(innerTask =>Console.WriteLine(innerTask.Result),TaskContinuationOptions.NotOnFaulted |TaskContinuationOptions.AttachedToParent);
      t2.ContinueWith(innerTask =>Console.WriteLine(innerTask.Exception.InnerException),TaskContinuationOptions.OnlyOnFaulted |TaskContinuationOptions.AttachedToParent);
      },
      TaskContinuationOptions.NotOnFaulted |TaskContinuationOptions.AttachedToParent);

    t.ContinueWith(task =>Console.WriteLine(t.Exception.InnerException),TaskContinuationOptions.OnlyOnFaulted |TaskContinuationOptions.AttachedToParent);
  });

  containerTask.Start();
  return containerTask;
}

async static Task AsynchronyWithAwait()
{
  try
  {
    string result = await GetInfoAsync("Async 1");
    Console.WriteLine(result);
    result = await GetInfoAsync("Async 2");
    Console.WriteLine(result);
  }
  catch (Exception ex)
  {
    Console.WriteLine(ex);
  }
}

async static Task<string> GetInfoAsync(string name)
{
  Console.WriteLine("Task {0} started!", name);
  await Task.Delay(TimeSpan.FromSeconds(2));
  if(name == "TPL 2")
    throw new Exception("Boom!");
  return string.Format("Task {0} is running on a thread id{1}. Is thread pool thread: {2}",name, Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
Task t = AsynchronyWithTPL();
t.Wait();

t = AsynchronyWithAwait();
t.Wait();
```

1.  运行程序。

## 工作原理...

当程序运行时，我们运行两个异步操作，就像在第一个步骤中一样。然而，这一次我们将从`AsynchronyWithAwait`方法开始。它看起来仍然像通常的同步代码；唯一的区别是两个`await`语句。最重要的一点是，代码仍然是顺序的，`Async 2`任务只有在前一个任务完成后才会开始。当我们阅读代码时，程序流程非常清晰：我们看到什么先运行，然后是什么之后。那么，这个程序是如何异步的呢？嗯，首先，它并不总是异步的。如果一个任务在我们使用`await`时已经完成，我们将同步地得到它的结果。否则，当我们在代码中看到`await`语句时，通常的做法是注意到此时方法将立即返回，剩下的代码将在一个继续任务中运行。由于我们不阻塞等待操作的结果，这是一个异步调用。我们可以在`Main`方法中调用`t.Wait`之外的任何其他任务，而`AsynchronyWithAwait`方法中的代码正在执行。然而，主线程必须等待直到所有异步操作完成，否则它们将在后台线程上运行时被停止。

`AsynchronyWithTPL`方法模拟了与`AsynchronyWithAwait`方法相同的程序流程。我们需要一个容器任务来一起处理所有依赖任务。然后，我们启动主任务，并为其添加一组继续任务。当任务完成时，我们打印出结果；然后，我们启动另一个任务，该任务在第二个任务完成后继续工作。为了测试异常处理，我们故意在运行第二个任务时抛出异常，并打印出其信息。这一系列的继续任务创建了与第一种方法相同的程序流程，当我们将其与带有`await`方法的代码进行比较时，我们可以看到它更容易阅读和理解。唯一的诀窍是要记住，异步并不总是意味着并行执行。

# 使用 await 操作符执行并行异步任务执行

在这个步骤中，我们将学习如何使用`await`来并行运行异步操作，而不是通常的顺序执行。

## 准备工作

要按照这个步骤，你需要 Visual Studio 2012。没有其他先决条件。这个步骤的源代码可以在`BookSamples\Chapter5\Recipe4`找到。

## 如何做...

要理解使用`await`操作符进行并行异步任务执行，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码：

```cs
async static Task AsynchronousProcessing()
{
  Task<string> t1 = GetInfoAsync("Task 1", 3);
  Task<string> t2 = GetInfoAsync("Task 2", 5);

  string[] results = await Task.WhenAll(t1, t2);
  foreach (string result in results)
  {
    Console.WriteLine(result);
  }
}

async static Task<string> GetInfoAsync(string name, int seconds)
{
  await Task.Delay(TimeSpan.FromSeconds(seconds));
  /*await Task.Run(() =>Thread.Sleep(TimeSpan.FromSeconds(seconds)));*/
  return string.Format("Task {0} is running on a thread id{1}. Is thread pool thread: {2}", name,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
Task t = AsynchronousProcessing();
t.Wait();
```

1.  运行程序。

## 工作原理...

在这里，我们定义了两个分别运行 3 秒和 5 秒的异步任务。然后，我们使用`Task.WhenAll`辅助方法创建另一个任务，只有当所有底层任务完成时才会完成。然后，我们等待此组合任务的结果。5 秒后，我们得到了所有结果，这意味着任务是同时运行的。

然而，有一个有趣的观察。当您运行程序时，您可能会注意到两个任务很可能由线程池中的同一个工作线程提供服务。当我们并行运行任务时，这是如何可能的？为了使事情更有趣，让我们注释掉`GetIntroAsync`方法中的`await Task.Delay`行，并取消注释`await Task.Run`行，然后运行程序。

我们将看到在这种情况下，两个任务将由不同的工作线程提供服务。不同之处在于`Task.Delay`在内部使用了一个计时器，处理过程如下：我们从线程池中获取工作线程，它等待`Task.Delay`方法返回结果。然后，`Task.Delay`方法启动计时器，并指定在计时器计算`Task.Delay`方法指定的秒数时将调用的代码。然后我们立即将工作线程返回到线程池。当计时器事件运行时，我们再次从线程池中获取任何可用的工作线程（可能是我们首先使用的相同线程），并在其上运行计时器提供的代码。

当我们使用`Task.Run`方法时，我们从线程池中获取一个工作线程，并使其阻塞一段时间，提供给`Thread.Sleep`方法。然后，我们获取第二个工作线程并阻塞它。在这种情况下，我们消耗了两个工作线程，它们完全没有做任何事情，无法执行任何其他任务。

我们将在第九章中详细讨论第一种情况，*使用异步 I/O*，在那里我们将讨论一大堆与数据输入和输出一起工作的异步操作。在可能的情况下始终使用第一种方法是创建可扩展服务器应用程序的关键。

# 在异步操作中处理异常

本示例将描述如何在 C#中使用异步函数处理异常。我们将学习如何处理使用`await`进行多个并行异步操作时的聚合异常。

## 准备工作

要执行此示例，您需要 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`BookSamples\Chapter5\Recipe5`中找到。

## 如何做到这一点...

要了解异步操作中的异常处理，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
async static Task AsynchronousProcessing()
{
  Console.WriteLine("1\. Single exception");

  try
  {
    string result = await GetInfoAsync("Task 1", 2);
    Console.WriteLine(result);
  }
  catch (Exception ex)
  {
    Console.WriteLine("Exception details: {0}", ex);
  }

  Console.WriteLine();
  Console.WriteLine("2\. Multiple exceptions");

  Task<string> t1 = GetInfoAsync("Task 1", 3);
  Task<string> t2 = GetInfoAsync("Task 2", 2);
  try
  {
    string[] results = await Task.WhenAll(t1, t2);
    Console.WriteLine(results.Length);
  }
  catch (Exception ex)
  {
    Console.WriteLine("Exception details: {0}", ex);
  }

  Console.WriteLine();
  Console.WriteLine("2\. Multiple exceptions with AggregateException");

  t1 = GetInfoAsync("Task 1", 3);
  t2 = GetInfoAsync("Task 2", 2);
  Task<string[]> t3 = Task.WhenAll(t1, t2);
  try
  {
    string[] results = await t3;
    Console.WriteLine(results.Length);
  }
  catch
  {
    var ae = t3.Exception.Flatten();
    var exceptions = ae.InnerExceptions;
    Console.WriteLine("Exceptions caught: {0}", exceptions.Count);
    foreach (var e in exceptions)
    {
      Console.WriteLine("Exception details: {0}", e);
      Console.WriteLine();
    }
  }
}

async static Task<string> GetInfoAsync(string name, int seconds)
{
  await Task.Delay(TimeSpan.FromSeconds(seconds));
  throw new Exception(string.Format("Boom from {0}!", name));
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
Task t = AsynchronousProcessing();
t.Wait();
```

1.  运行程序。

## 它是如何工作的...

我们运行三个场景来说明在 C#中使用`async`和`await`处理错误的最常见情况。第一种情况非常简单，几乎与通常的同步代码相同。我们只是使用`try`/`catch`语句并获取异常的详细信息。

常见的错误是在等待多个异步操作时使用相同的方法。如果我们像以前一样使用`catch`块，我们将只从底层的`AggregateException`对象中得到第一个异常。

为了收集所有信息，我们必须使用等待任务的`Exception`属性。在第三种情况下，我们展平`AggregateException`层次结构，然后使用`AggregateException`的`Flatten`方法解开其中的所有异常。

# 避免使用捕获的同步上下文

本教程讨论了使用`await`获取异步操作结果时同步上下文行为的细节。我们将学习何时以及如何关闭同步上下文流。

## 准备就绪

要按照本教程进行，您需要 Visual Studio 2012。没有其他先决条件。本教程的源代码可以在`BookSamples\Chapter5\Recipe6`中找到。

## 如何做...

要了解使用`await`时同步上下文行为的细节，并学习何时以及如何关闭同步上下文流，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# **Console Application**项目。

1.  添加对 Windows Presentation Foundation Library 的引用。

1.  在项目中右键单击**References**文件夹，然后选择**Add reference…**菜单选项。

1.  添加对以下库的引用：**PresentationCore**，**PresentationFramework**，**System.Xaml**和**Windows.Base**。您可以使用引用管理器对话框中的搜索功能，如下所示：

![如何做...](img/7644OT_05_01.jpg)

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Diagnostics;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
private static Label _label;

async static void Click(object sender, EventArgs e)
{
  _label.Content = new TextBlock {Text = "Calculating..."};
  TimeSpan resultWithContext = await Test();
  TimeSpan resultNoContext = await TestNoContext();
  /*TimeSpan resultNoContext = awaitTestNoContext().ConfigureAwait(false);*/
  var sb = new StringBuilder();
  sb.AppendLine(string.Format("With the context: {0}",resultWithContext));
  sb.AppendLine(string.Format("Without the context: {0}",resultNoContext));
  sb.AppendLine(string.Format("Ratio: {0:0.00}",resultWithContext.TotalMilliseconds/resultNoContext.TotalMilliseconds));
  _label.Content = new TextBlock {Text = sb.ToString()};
}

async static Task<TimeSpan> Test()
{
  const int iterationsNumber = 100000;
  var sw = new Stopwatch();
  sw.Start();
  for (int i = 0; i < iterationsNumber; i++)
  {
    var t = Task.Run(() => { });
    await t;
  }
  sw.Stop();
  return sw.Elapsed;
}

async static Task<TimeSpan> TestNoContext()
{
  const int iterationsNumber = 100000;
  var sw = new Stopwatch();
  sw.Start();
  for (int i = 0; i < iterationsNumber; i++)
  {
    var t = Task.Run(() => { });
    await t.ConfigureAwait(continueOnCapturedContext: false);
  }
  sw.Stop();
  return sw.Elapsed;
}
```

1.  用以下代码片段替换`Main`方法：

```cs
[STAThread]
static void Main(string[] args)
{
  var app = new Application();
  var win = new Window();
  var panel = new StackPanel();
  var button = new Button();
  _label = new Label();
  _label.FontSize = 32;
  _label.Height = 200;
  button.Height = 100;
  button.FontSize = 32;
  button.Content = new TextBlock {Text = "Start asynchronous operations"};
  button.Click += Click;
  panel.Children.Add(_label);
  panel.Children.Add(button);
  win.Content = panel;
  app.Run(win);

  Console.ReadLine();
}
```

1.  运行程序。

## 工作原理...

在这个例子中，我们将研究异步函数默认行为的最重要方面之一。我们已经从第四章*使用任务并行库*中了解了任务调度程序和同步上下文。默认情况下，`await`操作符会尝试捕获同步上下文，并在其上执行后续代码。正如我们已经知道的那样，这有助于我们通过使用用户界面控件编写异步代码。此外，使用`await`时不会发生死锁情况，因为我们在等待结果时不会阻塞 UI 线程，就像在上一章中描述的那样。

这是合理的，但让我们看看可能发生的情况。在这个例子中，我们通过编程方式创建了一个 Windows Presentation Foundation 应用程序，并订阅了它的按钮点击事件。单击按钮时，我们运行两个异步操作。其中一个使用常规的`await`操作符，而另一个使用`ConfigureAwait`方法，并将`false`作为参数值。它明确指示我们不应该使用捕获的同步上下文来在其上运行继续代码。在每个操作中，我们测量它们完成所需的时间，然后在主屏幕上显示相应的时间和比率。

结果是，我们看到常规的`await`操作符需要更长的时间才能完成。这是因为我们在 UI 线程上发布了十万个继续任务，它使用其消息循环来异步处理这些任务。在这种情况下，我们不需要此代码在 UI 线程上运行，因为我们不从异步操作中访问 UI 组件；使用`ConfigureAwait`和`false`将是一个更有效的解决方案。

还有一件值得注意的事情。尝试只点击按钮运行程序并等待结果。现在再做同样的事情，但这次在点击按钮的同时尝试随机拖动应用程序窗口的一侧。您会注意到捕获的同步上下文中的代码变得更慢！这个有趣的副作用完美地说明了异步编程是多么危险。很容易遇到这样的情况，如果您以前从未经历过这样的行为，几乎不可能进行调试。

公平起见，让我们看看相反的情况。在前面的代码片段中，在`Click`方法内部取消注释已注释的行，并注释其前面的行。运行应用程序时，我们将收到一个多线程控制访问异常，因为设置`Label`控件文本的代码不会发布在捕获的上下文上，而是在一个线程池工作线程上执行。

# 解决`async void`方法的问题

本示例描述了为什么使用`async void`方法非常危险。我们将学习在什么情况下可以使用此方法，以及在可能的情况下应该使用什么。

## 准备工作

要执行此示例，您需要 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`BookSamples\Chapter5\Recipe7`中找到。

## 如何做...

要学习如何使用`async void`方法，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
async static Task AsyncTaskWithErrors()
{
  string result = await GetInfoAsync("AsyncTaskException",2);
  Console.WriteLine(result);
}

async static void AsyncVoidWithErrors()
{
  string result = await GetInfoAsync("AsyncVoidException",2);
  Console.WriteLine(result);
}

async static Task AsyncTask()
{
  string result = await GetInfoAsync("AsyncTask", 2);
  Console.WriteLine(result);
}

private static async void AsyncVoid()
{
  string result = await GetInfoAsync("AsyncVoid", 2);
  Console.WriteLine(result);
}

async static Task<string> GetInfoAsync(string name,int seconds)
{
  await Task.Delay(TimeSpan.FromSeconds(seconds));
  if(name.Contains("Exception"))
    throw new Exception(string.Format("Boom from {0}!",name));
  return string.Format("Task {0} is running on a thread id{1}. Is thread pool thread: {2}", name,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
Task t = AsyncTask();
t.Wait();

AsyncVoid();
Thread.Sleep(TimeSpan.FromSeconds(3));

t = AsyncTaskWithErrors();
while(!t.IsFaulted)
{
  Thread.Sleep(TimeSpan.FromSeconds(1));
}
Console.WriteLine(t.Exception);

//try
//{
//  AsyncVoidWithErrors();
//  Thread.Sleep(TimeSpan.FromSeconds(3));
//}
//catch (Exception ex)
//{
//  Console.WriteLine(ex);
//}

//int[] numbers = new[] {1, 2, 3, 4, 5};
//Array.ForEach(numbers, async number => {
//  await Task.Delay(TimeSpan.FromSeconds(1));
//  if (number == 3) throw new Exception("Boom!");
//  Console.WriteLine(number);
//});

Console.ReadLine();
```

1.  运行程序。

## 它是如何工作的...

程序启动时，我们通过调用两个方法`AsyncTask`和`AsyncVoid`启动了两个异步操作。第一个方法返回一个`Task`对象，而另一个返回的是`async void`，因为它没有返回值。它们都立即返回，因为它们是异步的，但是第一个可以通过返回的任务状态轻松监视，或者只需调用其上的`Wait`方法。等待第二个方法完成的唯一方法是真正等待一段时间，因为我们没有声明任何可以用来监视异步操作状态的对象。当然，可以使用某种共享状态变量，并从`async void`方法中设置它，同时从`调用`方法中检查它，但最好还是返回一个`Task`对象。

最危险的部分是异常处理。在`async void`方法的情况下，异常处理方法将被发布到当前同步上下文；在我们的情况下，是线程池。线程池上的未处理异常将终止整个进程。可以使用`AppDomain.UnhandledException`事件拦截未处理的异常，但没有办法从那里恢复进程。要体验这一点，我们应该取消注释`Main`方法内部的`try`/`catch`块，然后运行程序。

关于使用`async void` lambda 表达式的另一个事实：它们与广泛使用的标准.NET Framework 类库中的`Action`类型兼容。很容易忘记在此 lambda 内部进行异常处理，这将再次使程序崩溃。要查看此示例，请取消注释`Main`方法内部的第二个已注释的块。

我强烈建议只在 UI 事件处理程序中使用`async void`。在所有其他情况下，请使用返回`Task`的方法。

# 设计自定义可等待类型

本示例展示了如何设计一个非常基本的可等待类型，与`await`运算符兼容。

## 准备工作

要执行此示例，您需要 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`BookSamples\Chapter5\Recipe8`中找到。

## 如何做...

要设计自定义可等待类型，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Runtime.CompilerServices;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
async static Task AsynchronousProcessing()
{
  var sync = new CustomAwaitable(true);
  string result = await sync;
  Console.WriteLine(result);

  var async = new CustomAwaitable(false);
  result = await async;

  Console.WriteLine(result);
}

class CustomAwaitable
{
  public CustomAwaitable(bool completeSynchronously)
  {
    _completeSynchronously = completeSynchronously;
  }

  public CustomAwaiter GetAwaiter()
  {
    return new CustomAwaiter(_completeSynchronously);
  }

  private readonly bool _completeSynchronously;
}

class CustomAwaiter : INotifyCompletion
{
  private string _result = "Completed synchronously";
  private readonly bool _completeSynchronously;

  public bool IsCompleted { get {return _completeSynchronously; } }

  public CustomAwaiter(bool completeSynchronously)
  {
    _completeSynchronously = completeSynchronously;
  }

  public string GetResult()
  {
    return _result;
  }

  public void OnCompleted(Action continuation)
  {
    ThreadPool.QueueUserWorkItem( state => {
      Thread.Sleep(TimeSpan.FromSeconds(1));
      _result = GetInfo();
      if(continuation != null) continuation();
    });
  }

  private string GetInfo()
  {
    return string.Format("Task is running on a thread id{0}. Is thread pool thread: {1}", name,Thread.CurrentThread.ManagedThreadId,Thread.CurrentThread.IsThreadPoolThread);
  }
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
Task t = AsynchronousProcessing();
t.Wait();
```

1.  运行程序。

## 它是如何工作的...

为了与`await`运算符兼容，类型应符合 C# 5.0 规范中规定的一些要求。如果您已安装 Visual Studio 2012，则可以在`C:\Program Files\Microsoft Visual Studio 11.0\VC#\Specifications\1033`文件夹中找到规范文档（假设您已使用默认安装路径）。

在第 7.7.7.1 段中，我们找到了可等待表达式的定义：

*等待表达式的任务需要是可等待的。如果表达式 t 是可等待的，则满足以下条件之一：*

+   *t 是动态的编译时类型*

+   *t 具有一个名为 GetAwaiter 的可访问的实例或扩展方法，没有参数和类型参数，并且返回类型 A，对于该类型，满足以下所有条件：*

+   *A 实现了接口 System.Runtime.CompilerServices.INotifyCompletion（以下简称 INotifyCompletion）*

+   *A 具有可访问的、可读的 bool 类型的 IsCompleted 实例属性*

+   *A 具有一个名为 GetResult 的可访问的实例方法，没有参数和类型参数*

这些信息足以让我们开始。首先，我们定义一个可等待类型`CustomAwaitable`并实现`GetAwaiter`方法，该方法反过来返回`CustomAwaiter`类型的实例。`CustomAwaiter`实现了`INotifyCompletion`接口；具有`bool`类型的`IsCompleted`属性，并且具有`GetResult`方法，该方法返回`string`类型。最后，我们编写了一段代码，创建了两个`CustomAwaitable`对象，并等待它们两个。

现在我们应该了解`await`表达式的评估方式。这次，为了避免不必要的细节，规范没有被引用。基本上，如果`IsCompleted`属性返回`true`，我们只需同步调用`GetResult`方法。这样，如果操作已经完成，我们就不需要为异步任务执行分配资源。我们通过向`CustomAwaitable`对象的构造方法提供`completeSynchronously`参数来覆盖这种情况。

否则，我们将一个回调操作注册到`CustomAwaiter`的`OnCompleted`方法，并启动异步操作。当它完成时，它将调用提供的回调，该回调将通过在`CustomAwaiter`对象上调用`GetResult`方法来获取结果。

### 注意

此实现仅用于教育目的。每当编写异步函数时，最自然的方法是使用标准的`Task`类型。只有在您无法使用`Task`并且确切知道自己在做什么的情况下，才应该定义自己的可等待类型。

还有许多其他与设计自定义可等待类型相关的主题，例如`ICriticalNotifyCompletion`接口实现和同步上下文传播。在了解了可等待类型的基本设计原理之后，您将能够使用 C#语言规范和其他信息源轻松找到所需的详细信息。但我想强调的是，除非您有非常充分的理由，否则请使用`Task`类型。

# 使用动态类型与等待

这个示例展示了如何设计一个与`await`运算符和动态 C#类型兼容的非常基本的类型。

## 准备工作

要按照这个示例进行操作，您需要 Visual Studio 2012。您需要互联网访问以下载 NuGet 包。没有其他先决条件。此示例的源代码可以在`BookSamples\Chapter5\Recipe9`中找到。

## 如何做...

要了解如何使用`dynamic`类型与`await`，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# **控制台应用程序**项目。

1.  通过以下步骤添加对**ImpromptuInterface** NuGet 包的引用：

1.  在项目中右键单击**引用**文件夹，然后选择**管理 NuGet 包...**菜单选项。

1.  现在将您喜欢的引用添加到**ImpromptuInterface NuGet**包中。您可以使用**管理 NuGet 包**对话框中的搜索功能，如下所示：

![如何操作...](img/7644OT_05_02.jpg)

1.  在`Program.cs`文件中，使用以下`using`指令：

```cs
using System;
using System.Dynamic;
using System.Runtime.CompilerServices;
using System.Threading;
using System.Threading.Tasks;
using ImpromptuInterface;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
async static Task AsynchronousProcessing()
{
  string result = await GetDynamicAwaitableObject(true);
  Console.WriteLine(result);

  result = await GetDynamicAwaitableObject(false);
  Console.WriteLine(result);
}

static dynamic GetDynamicAwaitableObject(bool completeSynchronously)
{
  dynamic result = new ExpandoObject();
  dynamic awaiter = new ExpandoObject();

  awaiter.Message = "Completed synchronously";
  awaiter.IsCompleted = completeSynchronously;
  awaiter.GetResult = (Func<string>)(() => awaiter.Message);

  awaiter.OnCompleted = (Action<Action>) ( callback => 
    ThreadPool.QueueUserWorkItem(state => {
      Thread.Sleep(TimeSpan.FromSeconds(1));
      awaiter.Message = GetInfo();
      if (callback != null) callback();
    })
  );

  IAwaiter<string> proxy = Impromptu.ActLike(awaiter);

  result.GetAwaiter = (Func<dynamic>) ( () => proxy );

  return result;
}

static string GetInfo()
{
  return string.Format("Task is running on a thread id {0}. Is thread pool thread: {1}",
      Thread.CurrentThread.ManagedThreadId, Thread.CurrentThread.IsThreadPoolThread);
}

public interface IAwaiter<T> : INotifyCompletion
{
bool IsCompleted { get; }

T GetResult();
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
Task t = AsynchronousProcessing();
t.Wait();
```

1.  运行程序。

## 工作原理...

在这里，我们重复了上一个示例中的技巧，但这次是借助动态表达式的帮助。我们可以通过 NuGet 来实现这个目标——一个包管理器，其中包含许多有用的库。这次我们将使用一个动态创建包装器并实现我们需要的接口的库。

首先，我们创建两个`ExpandoObject`类型的实例，并将它们分配给动态局部变量。这些变量将是我们的 awaitable 和 awaiter 对象。由于 awaitable 对象只需要具有`GetAwaiter`方法，因此提供它没有问题。`ExpandoObject`与`dynamic`关键字结合使用，允许我们自定义它，并通过分配相应的值添加属性和方法。实际上，它是一种具有`string`类型键和`object`类型值的字典类型集合。如果您熟悉 JavaScript 编程语言，您可能会注意到这与 JavaScript 对象非常相似。

由于`dynamic`允许我们在 C#中跳过编译时检查，`ExpandoObject`是这样编写的，如果您将某些内容分配给属性，它会创建一个字典条目，其中键是属性名称，值是提供的任何值。当您尝试获取属性值时，它会进入字典并提供存储在相应字典条目中的值。如果值是`Action`或`Func`类型，我们实际上存储了一个委托，该委托反过来可以像方法一样使用。因此，`dynamic`类型与`ExpandoObject`的组合允许我们创建一个对象，并动态为其提供属性和方法。

现在，我们需要构建我们的 awaiter 和 awaitable 对象。让我们从 awaiter 开始。首先，我们提供一个名为`Message`的属性，并为该属性提供一个初始值。然后，我们定义`GetResult`方法，使用`Func<string>`类型，我们分配一个 lambda 表达式，该表达式返回`Message`属性的值。接下来，我们实现`IsCompleted`属性。如果设置为`true`，我们可以跳过其余的工作并继续进行我们的 awaitable 对象，存储在`result`局部变量中。我们只需要添加一个返回`dynamic`对象的方法，并从中返回我们的 awaiter。然后，我们可以使用`result`作为 await 表达式；但是，它将以同步方式运行。

主要挑战是在我们的动态对象上实现异步处理。C#语言规范规定 awaiter 必须实现`INotifyCompletion`或`ICriticalNotifyCompletion`接口，而`ExpandoObject`并没有这样做。即使我们动态实现`OnCompleted`方法，并将其添加到 awaiter 对象中，我们也不会成功，因为我们的对象没有实现上述任何接口。

为了解决这个问题，我们使用了从 NuGet 获取的`ImpromptuInterface`库。它允许我们使用`Impromptu.ActLike`方法动态创建代理对象，这些对象将实现所需的接口。如果我们尝试创建一个实现`INotifyCompletion`接口的代理，我们仍然会失败，因为代理对象不再是动态的，而这个接口只有`OnCompleted`方法，但没有`IsCompleted`属性或`GetResult`方法。作为最后的解决方法，我们定义了一个通用接口`IAwaiter<T>`，它实现了`INotifyCompletion`并添加了所有必需的属性和方法。现在，我们将其用于代理生成，并将`result`对象更改为从`GetAwaiter`方法返回代理而不是 awaiter。程序现在可以工作了；我们刚刚构建了一个在运行时完全动态的可等待对象。
