# 第十章 并行编程模式

在本章中，我们将回顾程序员在尝试实现并行工作流时经常面临的常见问题。您将学习以下内容：

+   实现延迟共享状态

+   使用 BlockingCollection 实现并行管道

+   使用 TPL DataFlow 实现并行管道

+   使用 PLINQ 实现 Map/Reduce

# 介绍

编程中的模式意味着针对特定问题的具体和标准解决方案。通常，编程模式是人们积累经验、分析常见问题并提供解决方案的结果。

由于并行编程已经存在了很长时间，因此有许多不同的模式用于编程并行应用程序。甚至有专门的编程语言来使特定并行算法的编程更容易。然而，事情开始变得越来越复杂。在本书中，我将提供一个起点，让您能够进一步学习并行编程。我们将回顾一些非常基本但非常有用的模式，这些模式对并行编程中的许多常见情况非常有帮助。

首先是关于从多个线程使用**共享状态对象**。我想强调的是，尽量避免这样做。正如我们在之前的章节中讨论过的，当您编写并行算法时，共享状态真的很糟糕，但在许多情况下是不可避免的。我们将找出如何延迟对象的实际计算直到需要它，并且如何实现不同的场景以实现线程安全。

接下来的两个示例将展示如何创建结构化的并行数据流。我们将回顾一个生产者/消费者模式的具体案例，称为**并行管道**。我们将首先通过阻塞集合来实现它，然后看看微软为并行编程提供的另一个库**TPL DataFlow**有多么有用。

我们将学习的最后一个模式是**Map/Reduce**模式。在现代世界中，这个名字可能意味着非常不同的东西。有些人认为 map/reduce 不是解决任何问题的常见方法，而是大型分布式集群计算的具体实现。我们将找出这个模式背后的含义，并回顾一些例子，说明它在小型并行应用程序的情况下如何工作。

# 实现延迟共享状态

这个示例展示了如何编写一个延迟共享状态对象。

## 准备工作

要开始使用这个示例，您需要运行 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`BookSamples`的`Chapter10\Recipe1`中找到。

## 如何做...

要实现延迟共享状态，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static async Task ProcessAsynchronously()
{
  var unsafeState = new UnsafeState();
  Task[] tasks = new Task[4];

  for (int i = 0; i < 4; i++)
  {
    tasks[i] = Task.Run(() => Worker(unsafeState));
  }
  await Task.WhenAll(tasks);
  Console.WriteLine(" --------------------------- ");

  var firstState = new DoubleCheckedLocking();
  for (int i = 0; i < 4; i++)
  {
    tasks[i] = Task.Run(() => Worker(firstState));
  }

  await Task.WhenAll(tasks);
  Console.WriteLine(" --------------------------- ");

  var secondState = new BCLDoubleChecked();
  for (int i = 0; i < 4; i++)
  {
    tasks[i] = Task.Run(() => Worker(secondState));
  }

  await Task.WhenAll(tasks);
  Console.WriteLine(" --------------------------- ");

  var thirdState = new Lazy<ValueToAccess>(Compute);
  for (int i = 0; i < 4; i++)
  {
    tasks[i] = Task.Run(() => Worker(thirdState));
  }

  await Task.WhenAll(tasks);
  Console.WriteLine(" --------------------------- ");

  var fourthState = new BCLThreadSafeFactory();
  for (int i = 0; i < 4; i++)
  {
    tasks[i] = Task.Run(() => Worker(fourthState));
  }

  await Task.WhenAll(tasks);
  Console.WriteLine(" --------------------------- ");

}

static void Worker(IHasValue state)
{
  Console.WriteLine("Worker runs on thread id {0}",Thread.CurrentThread.ManagedThreadId);
  Console.WriteLine("State value: {0}", state.Value.Text);
}

static void Worker(Lazy<ValueToAccess> state)
{
  Console.WriteLine("Worker runs on thread id {0}",Thread.CurrentThread.ManagedThreadId);
  Console.WriteLine("State value: {0}", state.Value.Text);
}

static ValueToAccess Compute()
{
  Console.WriteLine("The value is being constructed on athread id {0}", Thread.CurrentThread.ManagedThreadId);
  Thread.Sleep(TimeSpan.FromSeconds(1));
  return new ValueToAccess(string.Format("Constructed on thread id {0}",Thread.CurrentThread.ManagedThreadId));
}

class ValueToAccess
{
  private readonly string _text; 
  public ValueToAccess(string text)
  {
    _text = text;
  }

  public string Text
  {
    get { return _text; }
  }
}

class UnsafeState : IHasValue
{
  private ValueToAccess _value;

  public ValueToAccess Value
  {
    get
    {
      if (_value == null)
      {
        _value = Compute();
      }
      return _value;
    }
  }

}

class DoubleCheckedLocking : IHasValue
{
  private object _syncRoot = new object();
  private volatile ValueToAccess _value;

  public ValueToAccess Value
  {
    get
    {
      if (_value == null)
      {
        lock (_syncRoot)
        {
          if (_value == null) _value = Compute();
        }
      }
      return _value;
    }
  }
}

class BCLDoubleChecked : IHasValue
{
  private object _syncRoot = new object();
  private ValueToAccess _value;
  private bool _initialized = false;

  public ValueToAccess Value
  {
    get
    {
      return LazyInitializer.EnsureInitialized(
        ref _value, ref _initialized, ref _syncRoot,Compute);
    }
  }
}

class BCLThreadSafeFactory : IHasValue
{
  private ValueToAccess _value;

  public ValueToAccess Value
  {
    get
    {
      return LazyInitializer.EnsureInitialized(ref _value,Compute);
    }
  }
}

interface IHasValue
{
  ValueToAccess Value { get; }
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
var t = ProcessAsynchronously();
t.GetAwaiter().GetResult();

Console.WriteLine("Press ENTER to exit");
Console.ReadLine();
```

1.  运行程序。

## 工作原理...

第一个示例展示了为什么不能在多个访问线程中使用`UnsafeState`对象是不安全的。我们看到`Construct`方法被多次调用，不同的线程使用不同的值，这显然是不正确的。为了解决这个问题，我们可以在读取值时使用锁定，如果它尚未初始化，则首先创建它。这样做是有效的，但是在每次读取操作时使用锁定并不高效。为了避免每次使用锁定，有一种传统的方法叫做**双重检查锁定**模式。我们首次检查值，如果不为空，我们避免不必要的锁定，直接使用共享对象。然而，如果尚未构造，我们使用锁定，然后第二次检查值，因为在我们的第一次检查和锁定操作之间它可能已经初始化。如果它仍未初始化，那么我们才计算值。我们可以清楚地看到这种方法适用于第二个示例——只有一次对`Construct`方法的调用，第一个调用的线程定义了共享对象的状态。

### 注意

请注意，如果延迟评估对象的实现是线程安全的，这并不意味着它的所有属性也是线程安全的。

例如，如果向`ValueToAccess`对象添加一个**int**公共属性，它将不是线程安全的；您仍然需要使用交错构造或锁定来确保线程安全。

这种模式非常常见，这就是为什么基类库中有几个类来帮助我们。首先，我们可以使用`LazyInitializer.EnsureInitialized`方法，它在内部实现了双重检查锁定模式。然而，最舒适的选项是使用`Lazy<T>`类，它允许我们拥有开箱即用的线程安全的延迟评估、共享状态。接下来的两个示例向我们展示它们等同于第二个示例，程序的行为也是相同的。唯一的区别是，由于`LazyInitializer`是一个静态类，我们不必像在`Lazy<T>`的情况下创建一个新的类的实例，因此在某些情况下第一种情况的性能会更好。

最后的选择是完全避免锁定，如果我们不关心`Construct`方法。如果它是线程安全的，没有副作用和/或严重的性能影响，我们可以多次运行它，但只使用第一次构造的值。最后一个示例展示了所描述的行为，我们可以通过使用另一个`LazyInitializer.EnsureInitialized`方法重载来实现这个结果。

# 使用 BlockingCollection 实现并行管道

本篇将描述如何使用标准的`BlockingCollection`数据结构实现生产者/消费者模式的特定场景，称为并行管道。

## 准备工作

要开始本篇，您需要运行 Visual Studio 2012。没有其他先决条件。本篇的源代码可以在`7644_Code\Chapter10\Recipe2`中找到。

## 操作步骤如下：

要了解如何使用`BlockingCollection`实现并行管道，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# **控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Collections.Concurrent;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
private const int CollectionsNumber = 4;
private const int Count = 10;

class PipelineWorker<TInput, TOutput>
{
  Func<TInput, TOutput> _processor = null;
  Action<TInput> _outputProcessor = null;
  BlockingCollection<TInput>[] _input;
  CancellationToken _token;

  public PipelineWorker(
      BlockingCollection<TInput>[] input,
      Func<TInput, TOutput> processor,
      CancellationToken token,
      string name)
  {
    _input = input;
    Output = new BlockingCollection<TOutput>[_input.Length];
    for (int i = 0; i < Output.Length; i++)
      Output[i] = null == input[i] ? null : new BlockingCollection<TOutput>(Count);

    _processor = processor;
    _token = token;
    Name = name;
  }

  public PipelineWorker(
      BlockingCollection<TInput>[] input,
      Action<TInput> renderer,
      CancellationToken token,
      string name)
  {
    _input = input;
    _outputProcessor = renderer;
    _token = token;
    Name = name;
    Output = null;
  }

  public BlockingCollection<TOutput>[] Output { get; private set; }

  public string Name { get; private set; }

  public void Run()
  {
    Console.WriteLine("{0} is running", this.Name);
    while (!_input.All(bc => bc.IsCompleted) && !_token.IsCancellationRequested)
    {
      TInput receivedItem;
      int i = BlockingCollection<TInput>.TryTakeFromAny(
          _input, out receivedItem, 50, _token);
      if (i >= 0)
      {
        if (Output != null)
        {
          TOutput outputItem = _processor(receivedItem);
          BlockingCollection<TOutput>.AddToAny(Output,outputItem);
          Console.WriteLine("{0} sent {1} to next,on thread id {2}", Name, outputItem,Thread.CurrentThread.ManagedThreadId);
          Thread.Sleep(TimeSpan.FromMilliseconds(100));
        }
        else
        {
          _outputProcessor(receivedItem);
        }
      }
      else
      {
        Thread.Sleep(TimeSpan.FromMilliseconds(50));
      }
    }
    if (Output != null)
    {
      foreach (var bc in Output) bc.CompleteAdding();
    }
  }
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
var cts = new CancellationTokenSource();

Task.Run(() =>
{
  if (Console.ReadKey().KeyChar == 'c')
    cts.Cancel();
});

var sourceArrays = new BlockingCollection<int>[CollectionsNumber];
for (int i = 0; i < sourceArrays.Length; i++)
{
  sourceArrays[i] = new BlockingCollection<int>(Count);
}

var filter1 = new PipelineWorker<int, decimal>
(sourceArrays,
  (n) => Convert.ToDecimal(n * 0.97),
  cts.Token,
  "filter1"
);

var filter2 = new PipelineWorker<decimal, string>
(filter1.Output,
  (s) => String.Format("--{0}--", s),
  cts.Token,
  "filter2"
  );

var filter3 = new PipelineWorker<string, string>
(filter2.Output,
  (s) => Console.WriteLine("The final result is {0} onthread id {1}", s,Thread.CurrentThread.ManagedThreadId), cts.Token,"filter3");

try
{
  Parallel.Invoke(
    () =>
    {
      Parallel.For(0, sourceArrays.Length * Count,(j, state) =>
      {
        if (cts.Token.IsCancellationRequested)
        {
          state.Stop();
        }
        int k = BlockingCollection<int>.TryAddToAny(sourceArrays, j);
        if (k >= 0)
        {
          Console.WriteLine("added {0} to source data onthread id {1}", j,Thread.CurrentThread.ManagedThreadId);
          Thread.Sleep(TimeSpan.FromMilliseconds(100));
        }
      });
      foreach (var arr in sourceArrays)
      {
        arr.CompleteAdding();
      }
    },
    () => filter1.Run(),
    () => filter2.Run(),
    () => filter3.Run()
  );
}
catch (AggregateException ae)
{
  foreach (var ex in ae.InnerExceptions)
    Console.WriteLine(ex.Message + ex.StackTrace);
}

if (cts.Token.IsCancellationRequested)
{
  Console.WriteLine("Operation has been canceled!Press ENTER to exit.");
}
else
{
  Console.WriteLine("Press ENTER to exit.");
}
Console.ReadLine();
```

1.  运行程序。

## 它是如何工作的...

在前面的示例中，我们实现了最常见的并行编程场景之一。想象一下，我们有一些数据必须通过几个计算阶段，这些阶段需要相当长的时间。后面的计算需要前面的结果，所以我们不能并行运行它们。

如果我们只有一个项目要处理，那么提高性能的可能性就不多。但是，如果我们通过相同的计算阶段运行许多项目，我们可以使用并行管道技术。这意味着我们不必等到所有项目通过第一个计算阶段才进入下一个阶段。只要有一个项目完成了阶段，我们就将其移动到下一个阶段，同时前一个阶段正在处理下一个项目，依此类推。结果几乎是并行处理，只是需要第一个项目通过第一个计算阶段所需的时间。

在这里，我们为每个处理阶段使用了四个集合，说明我们也可以并行处理每个阶段。我们做的第一步是通过按*C*键提供取消整个过程的可能性。我们创建一个取消令牌并运行一个单独的任务来监视*C*键。然后，我们定义我们的管道。它由三个主要阶段组成。第一个阶段是我们将初始数字放在作为后续管道的项目来源的前四个集合中。这段代码在`Parallel.For`循环内，而`Parallel.Invoke`语句内部，因为我们并行运行所有阶段；初始阶段也是并行运行的。

下一阶段是定义我们的管道元素。逻辑在`PipelineWorker`类中定义。我们使用输入集合初始化工作程序，提供转换函数，然后并行运行工作程序与其他工作程序。这样我们定义了两个工作程序，或者过滤器，因为它们过滤初始序列。其中一个将整数转换为十进制值，另一个将十进制转换为字符串。最后，最后一个工作程序只是将每个传入的字符串打印到控制台。我们在每个地方都提供了运行线程 ID 以查看一切是如何工作的。除此之外，我们添加了人为的延迟，以便项目处理更加自然，因为我们真的使用了繁重的计算。

结果，我们看到了确切的预期行为。首先，一些项目被创建在初始集合上。然后，我们看到第一个过滤器开始处理它们，随着它们被处理，第二个过滤器开始工作，最后项目进入最后一个工作程序，将其打印到控制台。

# 使用 TPL DataFlow 实现并行管道

这个教程展示了如何使用 TPL DataFlow 库实现并行管道模式。

## 准备工作

要开始这个教程，你需要一个运行的 Visual Studio 2012\. 没有其他先决条件。这个教程的源代码可以在`7644_Code\Chapter10\Recipe3`中找到。

## 如何做...

要了解如何使用 TPL DataFlow 实现并行管道，执行以下步骤：

1.  启动 Visual Studio 2012\. 创建一个新的 C# **控制台应用程序**项目。

1.  添加对**Microsoft TPL DataFlow** NuGet 包的引用。

1.  右键单击项目中的**References**文件夹，选择**管理 NuGet 包...**菜单选项。

1.  现在添加你喜欢的**Microsoft TPL DataFlow** NuGet 包的引用。你可以使用**管理 NuGet 包**对话框中的搜索选项，如下所示：

![如何做...](img/7644OT_10_01.jpg)

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Threading.Tasks.Dataflow;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
async static Task ProcessAsynchronously()
{
  var cts = new CancellationTokenSource();

  Task.Run(() =>
  {
    if (Console.ReadKey().KeyChar == 'c')
      cts.Cancel();
  });

  var inputBlock = new BufferBlock<int>(
    new DataflowBlockOptions { BoundedCapacity = 5,CancellationToken = cts.Token });

  var filter1Block = new TransformBlock<int, decimal>(
    n =>
    {
      decimal result = Convert.ToDecimal(n * 0.97);
      Console.WriteLine("Filter 1 sent {0} to the nextstage on thread id {1}", result,Thread.CurrentThread.ManagedThreadId);
      Thread.Sleep(TimeSpan.FromMilliseconds(100));
      return result;
    },
    new ExecutionDataflowBlockOptions {MaxDegreeOfParallelism = 4, CancellationToken =cts.Token });

  var filter2Block = new TransformBlock<decimal, string>(
    n =>
    {
      string result = string.Format("--{0}--", n);
      Console.WriteLine("Filter 2 sent {0} to the nextstage on thread id {1}", result,Thread.CurrentThread.ManagedThreadId);
      Thread.Sleep(TimeSpan.FromMilliseconds(100));
      return result;
    },
    new ExecutionDataflowBlockOptions {
     MaxDegreeOfParallelism = 4, CancellationToken =cts.Token });

  var outputBlock = new ActionBlock<string>(
    s =>
    {
      Console.WriteLine("The final result is {0} on threadid {1}", s, Thread.CurrentThread.ManagedThreadId);
    },
    new ExecutionDataflowBlockOptions {
      MaxDegreeOfParallelism = 4, CancellationToken =cts.Token });

  inputBlock.LinkTo(filter1Block, new DataflowLinkOptions {PropagateCompletion = true });
  filter1Block.LinkTo(filter2Block, new DataflowLinkOptions{ PropagateCompletion = true });
  filter2Block.LinkTo(outputBlock, new DataflowLinkOptions{ PropagateCompletion = true });

  try
  {
    Parallel.For(0, 20, new ParallelOptions {MaxDegreeOfParallelism = 4, CancellationToken =cts.Token }
    , i =>
    {
      Console.WriteLine("added {0} to source data on threadid {1}", i, Thread.CurrentThread.ManagedThreadId);
      inputBlock.SendAsync(i).GetAwaiter().GetResult();
    });
    inputBlock.Complete();
    await outputBlock.Completion;
    Console.WriteLine("Press ENTER to exit.");
  }
  catch (OperationCanceledException)
  {
    Console.WriteLine("Operation has been canceled!Press ENTER to exit.");
  }

  Console.ReadLine();
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
var t = ProcessAsynchronously();
t.GetAwaiter().GetResult();
```

1.  运行程序。

## 工作原理...

在上一个教程中，我们已经实现了一个并行管道模式，通过顺序阶段处理项目。这是一个很常见的问题，而且编写这样的算法的一种提议的方法是使用微软的 TPL DataFlow 库。它通过**NuGet**分发，很容易在你的应用程序中安装和使用。

TPL DataFlow 库包含不同类型的块，可以以不同的方式连接在一起，形成可以部分并行和顺序执行的复杂过程。要查看一些可用的基础设施，让我们使用 TPL DataFlow 库来实现前面的场景。

首先，我们定义将处理我们的数据的不同块。请注意，这些块在构建过程中可以指定不同的选项，这些选项可能非常重要。例如，我们将取消标记传递给我们定义的每个块，并且当我们发出取消信号时，所有这些块都将停止工作。

我们从`BufferBlock`开始我们的过程。这个块保存项目以便将其传递给流中的下一个块。我们将其限制为五个项目的容量，指定`BoundedCapacity`选项值。这意味着当这个块中有五个项目时，它将停止接受新项目，直到现有项目中的一个传递到下一个块。

下一个块类型是`TransformBlock`。这个块用于数据转换步骤。在这里，我们定义了两个转换块，其中一个从整数创建十进制数，另一个从十进制值创建一个字符串。对于这个块，有一个`MaxDegreeOfParallelism`选项，指定最大同时工作线程数。

最后一个块是`ActionBlock`类型。这个块将在每个传入的项目上运行指定的操作。我们使用这个块将我们的项目打印到控制台上。

现在，我们使用`LinkTo`方法将这些块连接在一起。在这里，我们有一个简单的顺序数据流，但也可以创建更复杂的方案。在这里，我们还提供了`DataflowLinkOptions`，其中`PropagateCompletion`属性设置为`true`。这意味着当步骤完成时，它将自动传播其结果和异常到下一个阶段。然后我们并行地开始向缓冲块添加项目，当完成添加新项目时，调用块的`Complete`方法。然后我们等待最后一个块完成。在取消的情况下，我们处理`OperationCancelledException`并取消整个过程。

# 使用 PLINQ 实现 Map/Reduce

这个示例将描述如何在使用 PLINQ 时实现**Map**/**Reduce**模式。

## 准备就绪

要开始这个示例，您需要运行 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`7644_Code\Chapter10\Recipe4`中找到。

## 如何做...

要了解如何使用 PLINQ 实现 Map/Reduce，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# **控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
private static readonly char[] delimiters =Enumerable.Range(0, 256).Select(i => (char)i).Where(c =>!char.IsLetterOrDigit(c)).ToArray();

private const string textToParse = @"
Call me Ishmael. Some years ago - never mind how long precisely - having little or no money in my purse, and nothing particular to interest me on shore, I thought I would sail about a little and see the watery part of the world. It is a way I have of driving off the spleen, and regulating the circulation. Whenever I find myself growing grim about the mouth; whenever it is a damp, drizzly November in my soul; whenever I find myself involuntarily pausing before coffin warehouses, and bringing up the rear of every funeral I meet; and especially whenever my hypos get such an upper hand of me, that it requires a strong moral principle to prevent me from deliberately stepping into the street, and methodically knocking people's hats off - then, I account it high time to get to sea as soon as I can.

― Herman Melville, Moby Dick.
";
```

1.  在`Main`方法中添加以下代码片段：

```cs
var q = textToParse.Split(delimiters)
  .AsParallel()
  .MapReduce(
    s => s.ToLower().ToCharArray()
  , c => c
  , g => new[] {new {Char = g.Key, Count = g.Count()}})
  .Where(c => char.IsLetterOrDigit(c.Char))
  .OrderByDescending( c => c.Count);

foreach (var info in q)
{
  Console.WriteLine("Character {0} occured in the text {1}{2}", info.Char, info.Count, info.Count == 1 ? "time" : "times");
}
Console.WriteLine(" -------------------------------------------");
const string searchPattern = "en";

var q2 = textToParse.Split(delimiters)
  .AsParallel()
  .Where(s => s.Contains(searchPattern))
  .MapReduce(
    s => new [] {s}
    , s => s
    , g => new[] {new {Word = g.Key, Count = g.Count()}})
  .OrderByDescending(s => s.Count);

Console.WriteLine("Words with search pattern '{0}':",searchPattern);
foreach (var info in q2)
{
  Console.WriteLine("{0} occured in the text {1} {2}",info.Word, info.Count,
    info.Count == 1 ? "time" : "times");
}

int halfLengthWordIndex = textToParse.IndexOf(' ',textToParse.Length/2);

using(var sw = File.CreateText("1.txt"))
{
  sw.Write(textToParse.Substring(0, halfLengthWordIndex));
}

using(var sw = File.CreateText("2.txt"))
{
  sw.Write(textToParse.Substring(halfLengthWordIndex));
}

string[] paths = new[] { ".\\" };

Console.WriteLine(" ------------------------------------------------");
var q3 = paths
  .SelectMany(p => Directory.EnumerateFiles(p, "*.txt"))
  .AsParallel()
  .MapReduce(
    path => File.ReadLines(path).SelectMany(line =>line.Trim(delimiters).Split(delimiters)),word => string.IsNullOrWhiteSpace(word) ? '\t' :word.ToLower()[0], g => new [] { new {FirstLetter = g.Key, Count = g.Count()}})
  .Where(s => char.IsLetterOrDigit(s.FirstLetter))
  .OrderByDescending(s => s.Count);

Console.WriteLine("Words from text files");

foreach (var info in q3)
{
  Console.WriteLine("Words starting with letter '{0}'occured in the text {1} {2}", info.FirstLetter,info.Count,
    info.Count == 1 ? "time" : "times");
}
```

1.  在`Program`类定义之后添加以下代码片段：

```cs
static class PLINQExtensions
{
  public static ParallelQuery<TResult> MapReduce<TSource,TMapped, TKey, TResult>(
    this ParallelQuery<TSource> source,
    Func<TSource, IEnumerable<TMapped>> map,
    Func<TMapped, TKey> keySelector,
    Func<IGrouping<TKey, TMapped>,
    IEnumerable<TResult>> reduce)
  {
    return source.SelectMany(map)
    .GroupBy(keySelector)
    .SelectMany(reduce);
  }
}
```

1.  运行程序。

## 它是如何工作的...

`Map`/`Reduce`函数是另一种重要的并行编程模式。它适用于小型程序和大型多服务器计算。这种模式的含义是你有两个特殊的函数来应用于你的数据。其中一个是`Map`函数。它以键/值列表形式的一组初始数据，并产生另一个键/值序列，将数据转换为进一步处理的舒适格式。然后我们使用另一个名为`Reduce`的函数。`Reduce`函数接受`Map`函数的结果，并将其转换为我们实际需要的最小可能数据集。要了解这个算法是如何工作的，让我们通过这个示例来看一下。

首先，我们在字符串变量`textToParse`中定义了一个相对较大的文本。我们需要这个文本来运行我们的查询。然后我们将我们的`Map`/`Reduce`实现定义为`PLINQExtensions`类中的 PLINQ 扩展方法。我们使用`SelectMany`将初始序列转换为我们需要的序列，通过应用`Map`函数。这个函数从一个序列元素中产生几个新元素。然后我们选择如何使用`keySelector`函数对新序列进行分组，并使用`GroupBy`与这个键来产生一个中间键/值序列。我们做的最后一件事就是对产生的分组序列应用`Reduce`来得到结果。

在我们的第一个例子中，我们将文本分割成单独的单词，然后我们使用`Map`函数将每个单词切割成字符序列，并按字符值对结果进行分组。`Reduce`函数最终将序列转换为键值对，其中我们有一个字符和一个数字，表示它在文本中被使用的次数，按使用次数排序。因此，我们能够并行计算文本中每个字符的出现次数（因为我们使用 PLINQ 来查询初始数据）。

下一个例子非常相似，但现在我们使用 PLINQ 来过滤序列，只留下包含我们搜索模式的单词，然后按它们在文本中的使用情况对所有这些单词进行排序。

最后一个例子使用文件 I/O。我们将示例文本保存在磁盘上，将其分成两个文件。然后我们将`Map`函数定义为从目录名称生成多个字符串，这些字符串都是初始目录中所有文本文件中所有行中的所有单词。然后我们通过第一个字母对这些单词进行分组（过滤掉空字符串），并使用 reduce 来查看哪个字母在文本中最常用作第一个单词的字母。好处在于我们可以很容易地通过使用其他 map 和 reduce 函数的实现来将此程序分布，并且我们仍然能够使用 PLINQ 来使我们的程序易于阅读和维护。
