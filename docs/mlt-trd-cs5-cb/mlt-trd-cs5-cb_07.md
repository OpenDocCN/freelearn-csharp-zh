# 第七章：使用 PLINQ

在本章中，我们将回顾不同的并行编程范式，如任务和数据并行性，并介绍数据并行性和并行 LINQ 查询的基础知识。您将学习：

+   使用 Parallel 类

+   并行化 LINQ 查询

+   调整 PLINQ 查询的参数

+   在 PLINQ 查询中处理异常

+   在 PLINQ 查询中管理数据分区

+   为 PLINQ 查询创建自定义聚合器

# 介绍

在.NET Framework 中，有一个称为并行框架的库子集，通常称为**并行框架扩展**（**PFX**），这是这些库的第一个版本的名称。并行框架是随.NET Framework 4.0 发布的，由三个主要部分组成：

+   **任务并行库**（**TPL**）

+   并发集合

+   并行 LINQ 或 PLINQ

通过本书，我们学习了如何并行运行多个任务并使它们相互同步。实际上，我们将程序分成一组任务，并且有不同的线程运行不同的任务。这种方法被称为**任务并行性**，到目前为止我们只学习了任务并行性。

想象一下，我们有一个程序，对一大批数据进行一些繁重的计算。并行化这个程序最简单的方法是将这批数据分成较小的块，对这些数据块进行并行计算，然后聚合这些计算的结果。这种编程模型称为**数据并行性**。

任务并行性具有最低的抽象级别。我们将程序定义为一组任务的组合，明确定义它们如何组合。以这种方式组成的程序可能非常复杂和详细。并行操作在程序的不同位置定义，随着程序的增长，程序变得更难理解和维护。这种使程序并行的方式被称为**非结构化并行性**。如果我们有复杂的并行逻辑，这就是要付出的代价。

然而，当我们有更简单的程序逻辑时，我们可以尝试将更多的并行化细节交给 PFX 库和 C#编译器。例如，我们可以说，“我想并行运行这三种方法，我不在乎这种并行化的具体细节; 让.NET 基础设施决定细节”。这提高了抽象级别，因为我们不必提供关于我们如何并行化的详细描述。这种方法被称为**结构化并行性**，因为并行化通常是一种声明，并且每种并行化情况在程序中的一个地方被定义。

### 注意

可能会有一种印象，即非结构化并行性是一种不好的实践，而应该始终使用结构化并行性。我想强调这是不正确的。结构化并行性确实更易于维护，并且在可能的情况下更受青睐，但它是一种不太通用的方法。一般来说，有许多情况下我们根本无法使用它，使用 TPL 任务并行性以非结构化方式是完全可以的。

任务并行库有一个`Parallel`类，提供了结构化并行性的 API。这仍然是 TPL 的一部分，但我们将在本章中进行审查，因为它是从较低抽象级别向较高抽象级别过渡的一个完美例子。当我们使用`Parallel`类的 API 时，我们不需要提供如何分区我们的工作的细节。但是，我们仍然需要明确定义如何从分区结果中得出一个单一的结果。

PLINQ 具有最高的抽象级别。它会自动将数据分成块，并决定我们是否真的需要并行化查询，或者使用通常的顺序查询处理效果更好。然后，PLINQ 基础设施会负责将分区结果合并在一起。程序员可以调整许多选项来优化查询，实现最佳性能和结果。

在本章中，我们将介绍`Parallel`类的 API 用法和许多不同的 PLINQ 选项，比如使 LINQ 查询并行化，设置执行模式和调整 PLINQ 查询的并行度，处理查询项顺序以及处理 PLINQ 异常。我们还将学习如何管理 PLINQ 查询的数据分区。

# 使用 Parallel 类

本示例展示了如何使用`Parallel`类的 API。我们将学习如何并行调用方法，如何执行并行循环，并调整并行化机制。

## 准备工作

要按照这个步骤，您需要 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`BookSamples\Chapter7\Recipe1`中找到。

## 如何做...

要并行调用方法，执行并行循环，并使用`Parallel`类调整并行化机制，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下方添加以下代码片段：

```cs
static string EmulateProcessing(string taskName)
{
  Thread.Sleep(TimeSpan.FromMilliseconds(new Random(DateTime.Now.Millisecond).Next(250, 350)));
  Console.WriteLine("{0} task was processed on a thread id {1}",taskName, Thread.CurrentThread.ManagedThreadId);
  return taskName;
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
Parallel.Invoke(() => EmulateProcessing("Task1"),() => EmulateProcessing("Task2"),() => EmulateProcessing("Task3")
);

var cts = new CancellationTokenSource();

var result = Parallel.ForEach(
  Enumerable.Range(1, 30),
  new ParallelOptions
  {
    CancellationToken = cts.Token,
    MaxDegreeOfParallelism = Environment.ProcessorCount,
    TaskScheduler = TaskScheduler.Default
  },
  (i, state) =>
  {
    Console.WriteLine(i);
    if (i == 20)
    {
      state.Break();
      Console.WriteLine("Loop is stopped: {0}", state.IsStopped);
    }
  });

Console.WriteLine("---");
Console.WriteLine("IsCompleted: {0}", result.IsCompleted);
Console.WriteLine("Lowest break iteration: {0}", result.LowestBreakIteration);
```

1.  运行程序。

## 它是如何工作的...

该程序演示了`Parallel`类的不同特性。`Invoke`方法允许我们在并行运行多个操作，而不像在任务并行库中定义任务那样麻烦。`Invoke`方法会阻塞其他线程，直到所有操作完成，这是一个常见且方便的场景。

下一个功能是并行循环，使用`For`和`ForEach`方法定义。我们将仔细研究`ForEach`，因为它与`For`非常相似。关于并行`ForEach`循环，您可以处理任何`IEnumerable`集合，通过将动作委托应用于每个集合项来并行处理。我们能够提供几个选项，自定义并行化行为，并获得一个显示循环是否成功完成的结果。

为了调整我们的并行循环，我们向`ForEach`方法提供了`ParallelOptions`类的实例。这允许我们使用`CancellationToken`取消循环，限制最大并行度（可以并行运行的最大操作数），并提供自定义的`TaskScheduler`类来调度动作任务。动作可以接受额外的`ParallelLoopState`参数，这对于中断循环或检查循环当前发生了什么非常有用。

有两种方法可以使用此状态停止并行循环。我们可以使用`Break`或`Stop`方法。`Stop`方法告诉循环停止处理更多的工作，并将并行循环状态的`IsStopped`属性设置为`true`。`Break`方法在此后停止迭代，但最初的迭代将继续工作。在这种情况下，循环结果的`LowestBreakIteration`属性将包含调用`Break`方法的最低循环迭代的编号。

# 使 LINQ 查询并行化

本示例将描述如何使用 PLINQ 使查询并行化，以及如何从并行查询返回到顺序处理。

## 准备工作

要按照这个步骤，您需要 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`BookSamples\Chapter7\Recipe2`中找到。

## 如何做...

要使用 PLINQ 使查询并行化，并从并行查询返回到顺序处理，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#`控制台应用程序`项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void PrintInfo(string typeName)
{
  Thread.Sleep(TimeSpan.FromMilliseconds(150));
  Console.WriteLine("{0} type was printed on a thread id {1}", typeName, Thread.CurrentThread.ManagedThreadId);
}

static string EmulateProcessing(string typeName)
{
  Thread.Sleep(TimeSpan.FromMilliseconds(150));
  Console.WriteLine("{0} type was processed on a thread id {1}",typeName, Thread.CurrentThread.ManagedThreadId);
  return typeName;
}

static IEnumerable<string> GetTypes()
{
  return from assembly in AppDomain.CurrentDomain.GetAssemblies()from type in assembly.GetExportedTypes()where type.Name.StartsWith("Web")select type.Name;
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
var sw = new Stopwatch();
sw.Start();
var query = from t in GetTypes()select EmulateProcessing(t);

foreach (string typeName in query)
{
  PrintInfo(typeName);
}
sw.Stop();
Console.WriteLine("---");
Console.WriteLine("Sequential LINQ query.");
Console.WriteLine("Time elapsed: {0}", sw.Elapsed);
Console.WriteLine("Press ENTER to continue....");
Console.ReadLine();
Console.Clear();
sw.Reset();

sw.Start();
var parallelQuery = from t in ParallelEnumerable.AsParallel(GetTypes())select EmulateProcessing(t);

foreach (string typeName in parallelQuery)
{
  PrintInfo(typeName);
}
sw.Stop();
Console.WriteLine("---");
Console.WriteLine("Parallel LINQ query. The results are being merged on a single thread");
Console.WriteLine("Time elapsed: {0}", sw.Elapsed);
Console.WriteLine("Press ENTER to continue....");
Console.ReadLine();
Console.Clear();
sw.Reset();

sw.Start();
parallelQuery = from t in GetTypes().AsParallel()select EmulateProcessing(t);

parallelQuery.ForAll(PrintInfo);

sw.Stop();
Console.WriteLine("---");
Console.WriteLine("Parallel LINQ query. The results are being processed in parallel");
Console.WriteLine("Time elapsed: {0}", sw.Elapsed);
Console.WriteLine("Press ENTER to continue....");
Console.ReadLine();
Console.Clear();
sw.Reset();

sw.Start();
query = from t in GetTypes().AsParallel().AsSequential()select EmulateProcessing(t);

foreach (var typeName in query)
{
  PrintInfo(typeName);
}

sw.Stop();
Console.WriteLine("---");
Console.WriteLine("Parallel LINQ query, transformed into sequential.");
Console.WriteLine("Time elapsed: {0}", sw.Elapsed);
Console.WriteLine("Press ENTER to continue....");
Console.ReadLine();
Console.Clear();
```

1.  运行程序。

## 它是如何工作的...

当程序运行时，我们创建一个 LINQ 查询，该查询使用反射 API 从当前应用程序域中加载的程序集中获取所有以“Web”开头的类型。我们使用`EmulateProcessing`和`PrintInfo`方法模拟处理每个项目和打印项目的延迟。我们还使用`Stopwatch`类来测量每个查询的执行时间。

首先运行一个通常的顺序 LINQ 查询。这里没有并行化，所以一切都在当前线程上运行。查询的第二个版本明确使用了`ParallelEnumerable`类。`ParallelEnumerable`包含了 PLINQ 逻辑实现，并且组织为`IEnumerable`集合功能的一些扩展方法。通常我们不会显式使用这个类；这里是为了说明 PLINQ 实际上是如何工作的。第二个版本并行运行`EmulateProcessing`；然而，默认情况下结果会在单个线程上合并，因此查询执行时间应该比第一个版本少几秒。

第三个版本展示了如何使用`AsParallel`方法以声明方式并行运行 LINQ 查询。我们不关心这里的实现细节，只是说明我们想要并行运行。然而，这个版本的关键区别在于我们使用`ForAll`方法来打印查询结果。它在相同的线程上运行操作以处理查询中的所有项目，跳过结果合并步骤。这使我们也可以并行运行`PrintInfo`，这个版本甚至比上一个版本运行得更快。

最后一个示例展示了如何使用`AsSequential`方法将 PLINQ 查询转换回顺序。我们可以看到这个查询的运行方式与第一个查询完全相同。

# 调整 PLINQ 查询的参数

该示例展示了如何使用 PLINQ 查询来管理并行处理选项，以及这些选项在查询执行期间可能会产生的影响。

## 准备工作

要执行此示例，您需要 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`BookSamples\Chapter7\Recipe3`中找到。

## 如何做...

要了解如何使用 PLINQ 查询来管理并行处理选项及其影响，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static string EmulateProcessing(string typeName)
{
  Thread.Sleep(TimeSpan.FromMilliseconds(new Random(DateTime.Now.Millisecond).Next(250,350)));
  Console.WriteLine("{0} type was processed on a thread id {1}",typeName, Thread.CurrentThread.ManagedThreadId);
  return typeName;
}

static IEnumerable<string> GetTypes()
{
  return from assembly in AppDomain.CurrentDomain.GetAssemblies()from type in assembly.GetExportedTypes()where type.Name.StartsWith("Web")orderby type.Name.Lengthselect type.Name;
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
var parallelQuery = from t in GetTypes().AsParallel()select EmulateProcessing(t);

var cts = new CancellationTokenSource();
cts.CancelAfter(TimeSpan.FromSeconds(3));

try
{
  parallelQuery.WithDegreeOfParallelism(Environment.ProcessorCount).WithExecutionMode(ParallelExecutionMode.ForceParallelism).WithMergeOptions (ParallelMergeOptions.Default).WithCancellation(cts.Token).ForAll(Console.WriteLine);
}
catch (OperationCanceledException)
{
  Console.WriteLine("---");
  Console.WriteLine("Operation has been canceled!");
}

Console.WriteLine("---");
Console.WriteLine("Unordered PLINQ query execution");
var unorderedQuery = from i in ParallelEnumerable.Range(1, 30) select i;

foreach (var i in unorderedQuery)
{
  Console.WriteLine(i);
}

Console.WriteLine("---");
Console.WriteLine("Ordered PLINQ query execution");
var orderedQuery = from i in ParallelEnumerable.Range(1, 30).AsOrdered() select i;

foreach (var i in orderedQuery)
{
  Console.WriteLine(i);
}
```

1.  运行程序。

## 它是如何工作的...

该程序演示了程序员可以使用的不同有用的 PLINQ 选项。我们首先创建一个 PLINQ 查询，然后创建另一个提供 PLINQ 调整的查询。

让我们先从取消开始。为了能够取消 PLINQ 查询，有一个接受取消令牌对象的`WithCancellation`方法。在这里，我们在三秒后发出取消令牌信号，这导致查询中的`OperationCanceledException`和其余工作的取消。

然后，我们可以为查询指定并行度。这是执行查询时将使用的精确并行分区的数量。在第一个示例中，我们使用了`Parallel.ForEach`循环，它具有最大并行度选项。这是不同的，因为它指定了最大分区值，但如果基础设施决定最好使用较少的并行性来节省资源并实现最佳性能，可能会有更少的分区。

另一个有趣的选项是使用`WithExecutionMode`方法覆盖查询执行模式。如果 PLINQ 基础设施决定并行化查询只会增加更多的开销，并且实际上运行得更慢，它可以以顺序模式处理一些查询。我们可以强制查询并行运行。

为了调整查询结果处理，我们有`WithMergeOptions`方法。默认模式是在从查询中返回结果之前，由 PLINQ 基础设施选择的一定数量的结果进行缓冲。如果查询需要大量时间，关闭结果缓冲以尽快获得结果更为合理。

最后一个选项是`AsOrdered`方法。当我们使用并行执行时，集合中的项目顺序可能不会被保留。在处理更早的项目之前，集合中的后续项目可能会被处理。为了防止这种情况，我们需要在并行查询上调用`AsOrdered`，明确告诉 PLINQ 基础设施我们打算保留项目顺序进行处理。

# 在 PLINQ 查询中处理异常

这个教程将描述如何处理 PLINQ 查询中的异常。

## 准备工作

要按照这个教程进行，你需要 Visual Studio 2012。没有其他先决条件。这个教程的源代码可以在`BookSamples\Chapter7\Recipe4`中找到。

## 如何做...

要了解如何处理 PLINQ 查询中的异常，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
```

1.  在`Main`方法中添加以下代码片段：

```cs
IEnumerable<int> numbers = Enumerable.Range(-5, 10);

var query = from number in numbersselect 100 / number;

try
{
  foreach(var n in query)
    Console.WriteLine(n);
}
catch (DivideByZeroException)
{
  Console.WriteLine("Divided by zero!");
}

Console.WriteLine("---");
Console.WriteLine("Sequential LINQ query processing");
Console.WriteLine();

var parallelQuery = from number in numbers.AsParallel()select 100 / number;

try
{
  parallelQuery.ForAll(Console.WriteLine);
}
catch (DivideByZeroException)
{
  Console.WriteLine("Divided by zero - usual exception handler!");
}
catch (AggregateException e)
{
  e.Flatten().Handle(ex =>
  {
    if (ex is DivideByZeroException)
      {
      Console.WriteLine("Divided by zero - aggregate exception handler!");
      return true;
      }

    return false;
  });
}

Console.WriteLine("---");
Console.WriteLine("Parallel LINQ query processing and results merging");
```

1.  运行程序。

## 它是如何工作的...

首先，我们对从-5 到 4 的数字范围进行了一个通常的 LINQ 查询。当我们除以零时，我们得到`DivideByZeroException`，并像通常一样在 try/catch 块中处理它。

然而，当我们使用`AsParallel`时，我们将得到`AggregateException`，因为现在我们是在并行运行，利用了后台的任务基础设施。`AggregateException`将包含在运行 PLINQ 查询时发生的所有异常。为了处理内部的`DivideByZeroException`类，我们使用了在第五章的*处理异步操作中的异常*教程中解释过的`Flatten`和`Handle`方法，*使用 C# 5.0*。

### 注意

很容易忘记当我们处理聚合异常时，内部有多个异常是非常常见的情况。如果你忘记处理所有这些异常，异常将冒泡并且应用程序将停止工作。

# 在 PLINQ 查询中管理数据分区

这个教程展示了如何创建一个非常基本的自定义分区策略，以特定方式并行化 LINQ 查询。

## 准备工作

要按照这个教程进行，你需要 Visual Studio 2012。没有其他先决条件。这个教程的源代码可以在`BookSamples\Chapter7\Recipe5`中找到。

## 如何做...

要了解如何创建一个非常基本的自定义分区策略来并行化 LINQ 查询，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static void PrintInfo(string typeName)
{
  Thread.Sleep(TimeSpan.FromMilliseconds(150));
  Console.WriteLine("{0} type was printed on a thread id {1}",typeName, Thread.CurrentThread.ManagedThreadId);
}

static string EmulateProcessing(string typeName)
{
  Thread.Sleep(TimeSpan.FromMilliseconds(150));
  Console.WriteLine("{0} type was processed on a thread id {1}. Has {2} length.",typeName, Thread.CurrentThread.ManagedThreadId, typeName.Length % 2 == 0 ? "even" : "odd");
  return typeName;
}

static IEnumerable<string> GetTypes()
{
  var types = AppDomain.CurrentDomain.GetAssemblies().SelectMany(a => a.GetExportedTypes());

  return from type in types where type.Name.StartsWith("Web")select type.Name;
}

public class StringPartitioner : Partitioner<string>
{
  private readonly IEnumerable<string> _data;

  public StringPartitioner(IEnumerable<string> data)
  {
    _data = data;
  }

  public override bool SupportsDynamicPartitions
  {
    get
    {
     return false;
    }
  }

  public override IList<IEnumerator<string>> GetPartitions(int partitionCount)
  {
    var result = new List<IEnumerator<string>>(2);
    result.Add(CreateEnumerator(true));
    result.Add(CreateEnumerator(false));

    return result;
  }

  IEnumerator<string> CreateEnumerator(bool isEven)
  {
    foreach (var d in _data)
    {
      if (!(d.Length % 2 == 0 ^ isEven))
      yield return d;
    }
  }
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
var partitioner = new StringPartitioner(GetTypes());
var parallelQuery = from t in partitioner.AsParallel()select EmulateProcessing(t);

parallelQuery.ForAll(PrintInfo);
```

1.  运行程序。

## 它是如何工作的...

为了说明我们能够为 PLINQ 查询选择自定义分区策略，我们创建了一个非常简单的分区器，以并行方式处理奇数和偶数长度的字符串。为了实现这一点，我们从标准基类`Partitioner<T>`中使用`string`作为类型参数派生出我们自定义的`StringPartitioner`类。

我们声明只支持静态分区，通过覆盖`SupportsDynamicPartitions`属性并将其设置为`false`。这意味着我们预定义了我们的分区策略。这是对初始集合进行分区的一种简单方法，但根据集合中的数据内容可能效率低下。例如，在我们的情况下，如果我们有许多奇数长度的字符串和只有一个偶数长度的字符串，其中一个线程将提前完成并且不会帮助处理奇数长度的字符串。另一方面，动态分区意味着我们在飞行中对初始集合进行分区，平衡工作负载在工作线程之间。

然后我们实现了`GetPartitions`方法，在其中定义了两个迭代器。第一个从源集合返回奇数长度的字符串，第二个返回偶数长度的字符串。最后，我们创建了我们的分区器的实例，并对其执行了 PLINQ 查询。我们可以看到不同的线程处理奇数长度和偶数长度的字符串。

# 为 PLINQ 查询创建自定义聚合器

本示例演示了如何为 PLINQ 查询创建自定义聚合函数。

## 准备就绪

要按照本示例进行操作，您需要 Visual Studio 2012。没有其他先决条件。本示例的源代码可以在`BookSamples\Chapter7\Recipe6`中找到。

## 如何做...

了解 PLINQ 查询的自定义聚合函数的工作原理，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static ConcurrentDictionary<char, int> AccumulateLettersInformation(ConcurrentDictionary<char, int> taskTotal , string item)
{
  foreach (var c in item)
  {
    if (taskTotal.ContainsKey(c))
    {
      taskTotal[c] = taskTotal[c] + 1;
    }
    else
    {
      taskTotal[c] = 1;
    }
  }
  Console.WriteLine("{0} type was aggregated on a thread id {1}",item, Thread.CurrentThread.ManagedThreadId);
  return taskTotal;
}

static ConcurrentDictionary<char, int> MergeAccumulators(ConcurrentDictionary<char, int> total, ConcurrentDictionary<char, int> taskTotal)
{
  foreach (var key in taskTotal.Keys)
  {
    if (total.ContainsKey(key))
    {
      total[key] = total[key] + taskTotal[key];
    }
    else
    {
      total[key] = taskTotal[key];
    }
  }
  Console.WriteLine("---");
  Console.WriteLine("Total aggregate value was calculated on a thread id {0}",Thread.CurrentThread.ManagedThreadId);
  return total;
}

static IEnumerable<string> GetTypes()
{
  var types = AppDomain.CurrentDomain.GetAssemblies().SelectMany(a => a.GetExportedTypes());

  return from type in typeswhere type.Name.StartsWith("Web")select type.Name;
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
var parallelQuery = from t in GetTypes().AsParallel() select t;

var parallelAggregator = parallelQuery.Aggregate(() => new ConcurrentDictionary<char, int>(),(taskTotal, item) => AccumulateLettersInformation(taskTotal, item), (total, taskTotal) => MergeAccumulators(total, taskTotal), total => total);

Console.WriteLine();
Console.WriteLine("There were the following letters in type names:");
var orderedKeys = from k in parallelAggregator.Keysorderby parallelAggregator[k] descending select k;

foreach (var c in orderedKeys)
{
  Console.WriteLine("Letter '{0}' ---- {1} times", c, parallelAggregator[c]);
}
```

1.  运行程序。

## 它是如何工作的...

在这里，我们实现了能够处理 PLINQ 查询的自定义聚合机制。为了实现这一点，我们必须了解，由于查询正在由多个任务同时并行处理，我们需要提供机制来并行聚合每个任务的结果，然后将这些聚合值合并为一个单一的结果值。

在本示例中，我们编写了一个聚合函数，用于计算 PLINQ 查询中字母的数量，该查询返回`IEnumerable<string>`集合。它计算每个集合项中的所有字母。为了说明并行聚合过程，我们打印出了关于哪个线程处理聚合的每个部分的信息。

我们使用`ParallelEnumerable`类中定义的`Aggregate`扩展方法对 PLINQ 查询结果进行聚合。它接受四个参数，每个参数都是执行聚合过程不同部分的函数。第一个是构造聚合器的空初始值的工厂。它也被称为种子值。

### 注意

请注意，提供给`Aggregate`方法的第一个值实际上不是聚合器函数的初始种子值，而是一个构造此初始种子值的工厂方法。如果您只提供一个实例，它将在所有并行运行的分区中使用，这将导致不正确的结果。

第二个函数将每个集合项聚合到分区聚合对象中。我们使用`AccumulateLettersInformation`方法实现此函数。它遍历字符串并计算其中的字母。这里聚合对象对于并行运行的每个查询分区都是不同的，这就是为什么我们称它们为`taskTotal`。

第三个函数是一个高级别的聚合函数，它从分区中获取聚合器对象并将其合并到全局聚合器对象中。我们使用`MergeAccumulators`方法实现它。最后一个函数是一个选择器函数，指定我们需要从全局聚合器对象中获取的确切数据。

最后，我们打印出聚合结果，并按集合项中最常用的字母对其进行排序。
