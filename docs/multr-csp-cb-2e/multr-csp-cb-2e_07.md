# 第七章：使用 PLINQ

在本章中，我们将回顾不同的并行编程范式，例如任务并行和数据并行，并介绍数据并行和并行 LINQ 查询的基础知识。你将学习以下技巧：

+   使用`Parallel`类

+   并行化 LINQ 查询

+   调整 PLINQ 查询的参数

+   在 PLINQ 查询中处理异常

+   管理 PLINQ 查询中的数据分区

+   为 PLINQ 查询创建自定义聚合器

# 简介

在.NET Framework 中，有一组被称为 Parallel Framework 的库子集，通常被称为**并行框架扩展**（**PFX**），这是这些库的第一个版本的名字。Parallel Framework 与.NET Framework 4.0 一起发布，包括三个主要部分：

+   任务并行库（TPL）

+   并发集合

+   并行 LINQ 或 PLINQ

到目前为止，你已经学习了如何并行运行多个任务，并使它们相互同步。实际上，我们将程序分割成一组任务，不同的线程运行不同的任务。这种方法被称为**任务并行**，而你到目前为止只学习了任务并行。

想象一下，我们有一个程序，它在一个大数据集上执行一些复杂的计算。将这个数据集分割成更小的块，并行地在这些数据块上运行所需的计算，然后汇总这些计算的结果，这是并行化这个程序最简单的方法。这种编程模型被称为**数据并行**。

任务并行具有最低的抽象级别。我们将程序定义为任务的组合，明确地定义它们是如何组合的。以这种方式组成的程序可能非常复杂和详细。在这个程序的不同地方定义了并行操作，随着程序的增长，程序变得越来越难以理解和维护。这种使程序并行化的方式被称为**无结构并行**。如果我们有复杂的并行化逻辑，这是我们必须付出的代价。

然而，当我们有更简单的程序逻辑时，我们可以尝试将更多的并行化细节卸载到 PFX 库和 C#编译器。例如，我们可以说：“我想并行运行这三个方法，我不关心具体如何并行化；让.NET 基础设施决定细节”。这样，抽象级别就提高了，因为我们不需要提供关于我们如何并行化的详细描述。这种做法被称为**结构化并行**，因为并行化通常是一种声明，并且每个并行化的案例都在程序中的确切位置定义。

### 注意

可能会有一种印象，认为无结构的并行化是坏习惯，而应该始终使用结构化并行化。我想强调，这并不正确。结构化并行化确实更易于维护，并且在可能的情况下是首选，但它是一种不太通用的方法。一般来说，有许多情况下我们根本无法使用它，以无结构的方式使用 TPL 任务并行化是完全 OK 的。

TPL 有一个`Parallel`类，它提供了结构化并行的 API。这仍然是 TPL 的一部分，但我们将在本章中回顾它，因为它是从较低抽象级别到较高抽象级别的完美示例。当我们使用`Parallel`类 API 时，我们不需要提供我们如何分区工作的细节。然而，我们仍然需要明确定义我们如何从分区结果中生成单个结果。

PLINQ 具有最高的抽象级别。它自动将数据分成块，并决定我们是否真的需要并行化查询，或者是否使用常规的顺序查询处理会更有效。然后，PLINQ 基础设施负责合并分区的结果。程序员可以调整许多选项以优化查询并实现最佳性能和结果。

在本章中，我们将介绍`Parallel`类 API 的使用以及许多不同的 PLINQ 选项，例如使 LINQ 查询并行化，设置执行模式，调整 PLINQ 查询的并行度，处理查询项顺序，以及处理 PLINQ 异常。你还将学习如何管理 PLINQ 查询的数据分区。

# 使用 Parallel 类

这个配方展示了如何使用`Parallel`类 API。你将学习如何并行调用方法，如何执行并行循环，以及调整并行化机制。

## 准备工作

要完成这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter7\Recipe1`中找到。

## 如何做到这一点...

要并行调用方法，执行并行循环，并使用`Parallel`类调整并行化机制，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static string EmulateProcessing(string taskName)
    {
      Sleep(TimeSpan.FromMilliseconds(
        new Random(DateTime.Now.Millisecond).Next(250, 350)));
      WriteLine($"{taskName} task was processed on a " +
                      $"thread id {CurrentThread.ManagedThreadId}");
      return taskName;
    }
    ```

1.  在`Main`方法内添加以下代码片段：

    ```cs
    Parallel.Invoke(
      () => EmulateProcessing("Task1"),
      () => EmulateProcessing("Task2"),
      () => EmulateProcessing("Task3")
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
        WriteLine(i);
        if (i == 20)
        {
          state.Break();
          WriteLine($"Loop is stopped: {state.IsStopped}");
        }
      });

    WriteLine("---");
    WriteLine($"IsCompleted: {result.IsCompleted}");
    WriteLine($"Lowest break iteration: {result.LowestBreakIteration}");
    ```

1.  运行程序。

## 它是如何工作的...

这个程序演示了`Parallel`类的不同功能。`Invoke`方法允许我们以比在 TPL 中定义任务更少的麻烦并行运行多个操作。与定义任务相比，`Invoke`方法会阻塞其他线程，直到所有操作都完成，这是一个非常常见且方便的场景。

下一个特性是并行循环，它通过`For`和`ForEach`方法定义。我们将仔细研究`ForEach`，因为它与`For`非常相似。使用`ForEach`并行循环，你可以通过将操作委托应用于每个集合项来并行处理任何`IEnumerable`集合。我们能够提供几个选项，自定义并行化行为，并获得一个显示循环是否成功完成的输出结果。

为了调整我们的并行循环，我们将`ParallelOptions`类的实例传递给`ForEach`方法。这允许我们使用`CancellationToken`取消循环，限制最大并行度（可以并行运行的最大操作数），并提供一个自定义的`TaskScheduler`类来安排操作任务。操作可以接受一个额外的`ParallelLoopState`参数，这对于中断循环或检查此时循环的状态非常有用。

有两种方法可以使用此状态停止并行循环。我们可以使用`Break`或`Stop`方法。`Stop`方法告诉循环停止处理任何更多的工作，并将并行循环状态的`IsStopped`属性设置为`true`。`Break`方法在迭代之后停止，但初始的迭代将继续工作。在这种情况下，循环结果的`LowestBreakIteration`属性将包含`Break`方法被调用的最低循环迭代数。

# 并行化 LINQ 查询

这个配方将描述如何使用 PLINQ 使查询并行化，以及如何从并行查询返回到顺序处理。

## 准备工作

为了完成这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter7\Recipe2`中找到。

## 如何操作...

为了使用 PLINQ 使查询并行化，并从并行查询返回到顺序处理，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Collections.Generic;
    using System.Diagnostics;
    using System.Linq;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static void PrintInfo(string typeName)
    {
      Sleep(TimeSpan.FromMilliseconds(150));
      WriteLine($"{typeName} type was printed on a thread " +
              $"id {CurrentThread.ManagedThreadId}");
    }

    static string EmulateProcessing(string typeName)
    {
      Sleep(TimeSpan.FromMilliseconds(150));
      WriteLine($"{typeName} type was processed on a thread " +
              $"id {CurrentThread.ManagedThreadId}");
      return typeName;
    }

    static IEnumerable<string> GetTypes()
    {
      return from assembly in AppDomain.CurrentDomain.GetAssemblies()
              from type in assembly.GetExportedTypes()
              where type.Name.StartsWith("Web")
              select type.Name;

    }
    ```

1.  在`Main`方法中添加以下代码片段：

    ```cs
    var sw = new Stopwatch();
    sw.Start();
    var query = from t in GetTypes()
      select EmulateProcessing(t);

    foreach (string typeName in query)
    {
      PrintInfo(typeName);
    }
    sw.Stop();
    WriteLine("---");
    WriteLine("Sequential LINQ query.");
    WriteLine($"Time elapsed: {sw.Elapsed}");
    WriteLine("Press ENTER to continue....");
    ReadLine();
    Clear();
    sw.Reset();

    sw.Start();
    var parallelQuery = from t in GetTypes().AsParallel()
            select EmulateProcessing(t);

    foreach (var typeName in parallelQuery)
    {
      PrintInfo(typeName);
    }
    sw.Stop();
    WriteLine("---");
    WriteLine("Parallel LINQ query. The results are being merged on a single thread");
    WriteLine($"Time elapsed: {sw.Elapsed}");
    WriteLine("Press ENTER to continue....");
    ReadLine();
    Clear();
    sw.Reset();

    sw.Start();
    parallelQuery = from t in GetTypes().AsParallel()
            select EmulateProcessing(t);

    parallelQuery.ForAll(PrintInfo);

    sw.Stop();
    WriteLine("---");
    WriteLine("Parallel LINQ query. The results are being processed in parallel");
    WriteLine($"Time elapsed: {sw.Elapsed}");
    WriteLine("Press ENTER to continue....");
    ReadLine();
    Clear();
    sw.Reset();

    sw.Start();
    query = from t in GetTypes().AsParallel().AsSequential()
        select EmulateProcessing(t);

    foreach (string typeName in query)
    {
      PrintInfo(typeName);
    }

    sw.Stop();
    WriteLine("---");
    WriteLine("Parallel LINQ query, transformed into sequential.");
    WriteLine($"Time elapsed: {sw.Elapsed}");
    WriteLine("Press ENTER to continue....");
    ReadLine();
    Clear();
    ```

1.  运行程序。

## 它是如何工作的...

当程序运行时，我们创建一个 LINQ 查询，该查询使用反射 API 从当前应用程序域中加载的程序集获取所有以*Web*开头的类型。我们使用`EmulateProcessing`和`PrintInfo`方法模拟处理每个项和打印它的延迟。我们还使用`Stopwatch`类来测量每个查询的执行时间。

首先，我们运行一个普通的顺序 LINQ 查询。这里没有并行化，所以所有操作都在当前线程上运行。查询的第二版明确使用了`ParallelEnumerable`类。`ParallelEnumerable`包含 PLINQ 逻辑实现，并作为`IEnumerable`集合功能的扩展方法组织。通常，我们不会明确使用这个类；我们在这里使用它来展示 PLINQ 实际上是如何工作的。第二版并行运行`EmulateProcessing`；然而，默认情况下，结果会在单个线程上合并，所以查询执行时间应该比第一版少几秒钟。

第三版展示了如何使用`AsParallel`方法以声明性方式并行运行 LINQ 查询。在这里，我们并不关心实现细节，只是声明我们希望并行运行这个查询。然而，这个版本的关键区别在于我们使用了`ForAll`方法来输出查询结果。它将在处理每个查询项的同一线程上运行操作，跳过了结果合并步骤。这允许我们并行运行`PrintInfo`，并且这个版本比上一个版本运行得更快。

最后一个示例展示了如何使用`AsSequential`方法将 PLINQ 查询转换回顺序查询。我们可以看到这个查询的运行方式与第一个完全相同。

# 调整 PLINQ 查询的参数

这个配方展示了我们如何使用 PLINQ 查询来管理并行处理选项，以及这些选项在查询执行期间可能产生的影响。

## 准备工作

为了完成这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter7\Recipe3`找到。

## 如何操作...

要了解如何使用 PLINQ 查询管理并行处理选项及其影响，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下面添加以下代码片段：

    ```cs
    static string EmulateProcessing(string typeName)
    {
      Sleep(TimeSpan.FromMilliseconds(
        new Random(DateTime.Now.Millisecond).Next(250,350)));
      WriteLine($"{typeName} type was processed on a thread " +
        $"id {CurrentThread.ManagedThreadId}");
      return typeName;
    }

    static IEnumerable<string> GetTypes()
    {
      return from assembly in AppDomain.CurrentDomain.GetAssemblies()
        from type in assembly.GetExportedTypes()
        where type.Name.StartsWith("Web")
        orderby type.Name.Length
        select type.Name;
    }
    ```

1.  在`Main`方法内部添加以下代码片段：

    ```cs
    var parallelQuery = from t in GetTypes().AsParallel()
            select EmulateProcessing(t);

    var cts = new CancellationTokenSource();
    cts.CancelAfter(TimeSpan.FromSeconds(3));

    try
    {
      parallelQuery
        .WithDegreeOfParallelism(Environment.ProcessorCount)
        .WithExecutionMode(ParallelExecutionMode.ForceParallelism)
        .WithMergeOptions(ParallelMergeOptions.Default)
        .WithCancellation(cts.Token)
        .ForAll(WriteLine);
    }
    catch (OperationCanceledException)
    {
      WriteLine("---");
      WriteLine("Operation has been canceled!");
    }

    WriteLine("---");
    WriteLine("Unordered PLINQ query execution");
    var unorderedQuery = from i in ParallelEnumerable.Range(1, 30)
           select i;

    foreach (var i in unorderedQuery)
    {
      WriteLine(i);
    }

    WriteLine("---");
    WriteLine("Ordered PLINQ query execution");
    var orderedQuery = from i in ParallelEnumerable.Range(1, 30).AsOrdered()
         select i;

    foreach (var i in orderedQuery)
    {
      WriteLine(i);
    }
    ```

1.  运行程序。

## 工作原理...

程序演示了程序员可以使用的不同有用的 PLINQ 选项。我们首先创建一个 PLINQ 查询，然后创建另一个提供 PLINQ 调整的查询。

让我们先从取消操作开始。为了能够取消 PLINQ 查询，有一个`WithCancellation`方法，它接受一个取消令牌对象。在这里，我们在 3 秒后发出取消令牌，这导致查询中的`OperationCanceledException`异常，并取消剩余的工作。

然后，我们可以为查询指定一个并行度。这是将要用于执行查询的确切并行分区数。在第一个食谱中，我们使用了 `Parallel.ForEach` 循环，它具有最大并行度选项。它不同之处在于它指定了一个最大分区值，但如果基础设施决定使用更少的并行度以节省资源并实现最佳性能，则可能会有更少的分区。

另一个有趣的选择是使用 `WithExecutionMode` 方法覆盖查询执行模式。如果 PLINQ 基础设施决定并行化查询只会增加开销并且实际上会运行得更慢，它可以将一些查询以顺序模式处理。使用 `WithExecutionMode`，我们可以强制查询以并行方式运行。

为了调整查询结果处理，我们有 `WithMergeOptions` 方法。默认模式是在将结果从查询返回之前，由 PLINQ 基础设施缓冲选择的一定数量的结果。如果查询需要花费大量时间，关闭结果缓冲以尽快获取结果更为合理。

最后一个选项是 `AsOrdered` 方法。当我们使用并行执行时，集合中的项目顺序可能不会保留。集合中的后续项目可能会在早期项目之前处理。为了防止这种情况，我们需要在并行查询上调用 `AsOrdered`，以明确告诉 PLINQ 基础设施我们打算保留项目顺序以进行处理。

# 在 PLINQ 查询中处理异常

本食谱将描述如何在 PLINQ 查询中处理异常。

## 准备工作

要完成这个食谱，你需要 Visual Studio 2015。没有其他先决条件。本食谱的源代码可以在 `BookSamples\Chapter7\Recipe4` 中找到。

## 如何做到这一点...

要了解如何在 PLINQ 查询中处理异常，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using static System.Console;
    ```

1.  在 `Main` 方法中添加以下代码片段：

    ```cs
    IEnumerable<int> numbers = Enumerable.Range(-5, 10);

    var query = from number in numbers
        select 100 / number;

    try
    {
      foreach(var n in query)
        WriteLine(n);
    }
    catch (DivideByZeroException)
    {
      WriteLine("Divided by zero!");
    }

    WriteLine("---");
    WriteLine("Sequential LINQ query processing");
    WriteLine();

    var parallelQuery = from number in numbers.AsParallel()
            select 100 / number;

    try
    {
      parallelQuery.ForAll(WriteLine);
    }
    catch (DivideByZeroException)
    {
      WriteLine("Divided by zero - usual exception handler!");
    }
    catch (AggregateException e)
    {
      e.Flatten().Handle(ex =>
      {
        if (ex is DivideByZeroException)
        {
          WriteLine("Divided by zero - aggregate exception handler!");
          return true;
        }

        return false;
      });
    }

    WriteLine("---");
    WriteLine("Parallel LINQ query processing and results merging");
    ```

1.  运行程序。

## 它是如何工作的...

首先，我们在从 -5 到 4 的数字范围内运行一个常规 LINQ 查询。当我们除以 0 时，我们得到 `DivideByZeroException`，我们像往常一样在 `try/catch` 块中处理它。

然而，当我们使用 `AsParallel` 时，我们得到 `AggregateException` 而不是其他，因为我们现在正在并行运行，利用幕后任务基础设施。`AggregateException` 将包含在运行 PLINQ 查询期间发生的所有异常。要处理内部的 `DivideByZeroException` 类，我们使用 `Flatten` 和 `Handle` 方法，这些方法在 第五章 的 *处理异步操作中的异常* 食谱中进行了说明，*使用 C# 6.0*。

### 注意

很容易忘记，当我们处理聚合异常时，内部有多个异常是一个非常常见的情况。如果你忘记处理所有这些异常，异常将会冒泡，应用程序将停止工作。

# 在 PLINQ 查询中管理数据分区

这个配方展示了如何创建一个非常基本的自定义分区策略以特定方式并行化 LINQ 查询。

## 准备工作

为了完成这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter7\Recipe5`找到。

## 如何做到这一点...

要学习如何创建一个非常基本的自定义分区策略以并行化 LINQ 查询，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Collections.Concurrent;
    using System.Collections.Generic;
    using System.Diagnostics;
    using System.Linq;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static void PrintInfo(string typeName)
    {
      Sleep(TimeSpan.FromMilliseconds(150));
      WriteLine($"{typeName} type was printed on a thread " +
    $"id {CurrentThread.ManagedThreadId}");
    }

    static string EmulateProcessing(string typeName)
    {
      Sleep(TimeSpan.FromMilliseconds(150));
      WriteLine($"{typeName} type was processed on a thread " +
      $"id { CurrentThread.ManagedThreadId}. Has " +
      $"{(typeName.Length % 2 == 0 ? "even" : "odd")} length.");

      return typeName;
    }

    static IEnumerable<string> GetTypes()
    {
      var types = AppDomain.CurrentDomain
        .GetAssemblies()
        .SelectMany(a => a.GetExportedTypes());

      return from type in types
        where type.Name.StartsWith("Web")
        select type.Name;
    }

    public class StringPartitioner : Partitioner<string>
    {
      private readonly IEnumerable<string> _data;

      public StringPartitioner(IEnumerable<string> data)
      {
        _data = data;
      }

      public override bool SupportsDynamicPartitions => false;

      public override IList<IEnumerator<string>>GetPartitions(
    int partitionCount)
      {
        var result = new List<IEnumerator<string>>(
    partitionCount);

        for (int i = 1; i <= partitionCount; i++)
        {
          result.Add(CreateEnumerator(i, partitionCount));
        }

        return result;
      }

      IEnumerator<string> CreateEnumerator(int partitionNumber, int partitionCount)
      {
        int evenPartitions = partitionCount / 2;
        bool isEven = partitionNumber % 2 == 0;
        int step = isEven ? evenPartitions : 
    partitionCount - evenPartitions;

        int startIndex = partitionNumber / 2 +
        partitionNumber % 2;

        var q = _data
          .Where(v => !(v.Length % 2 == 0 ^ isEven)
    || partitionCount == 1)
          .Skip(startIndex - 1);

        return q
          .Where((x, i) => i % step == 0)
          .GetEnumerator();

      }
    }
    ```

1.  在`Main`方法内部添加以下代码片段：

    ```cs
    var timer = Stopwatch.StartNew();
    var partitioner = new StringPartitioner(GetTypes());
    var parallelQuery = from t in partitioner.AsParallel()
    //      .WithDegreeOfParallelism(1)
          select EmulateProcessing(t);

    parallelQuery.ForAll(PrintInfo);
    int count = parallelQuery.Count();
    timer.Stop();
    WriteLine(" ----------------------- ");
    WriteLine($"Total items processed: {count}");
    WriteLine($"Time elapsesd: {timer.Elapsed}");
    ```

1.  运行程序。

## 它是如何工作的...

为了说明我们能够为 PLINQ 查询选择自定义的分区策略，我们创建了一个非常简单的分区器，该分区器并行处理奇数长度和偶数长度的字符串。为了实现这一点，我们使用`string`作为类型参数，从标准基类`Partitioner<T>`派生我们的自定义`StringPartitioner`类。

我们通过重写`SupportsDynamicPartitions`属性并将其设置为`false`来声明我们只支持静态分区。这意味着我们预定义了我们的分区策略。这是一种简单的分区初始集合的方法，但可能会根据集合内部的数据而效率低下。例如，在我们的情况下，如果我们有很多奇数长度的字符串和只有一个偶数长度的字符串，那么其中一个线程会提前完成，而不会帮助处理奇数长度的字符串。另一方面，动态分区意味着我们在运行时对初始集合进行分区，平衡工作负载在工作线程之间。

然后，我们实现`GetPartitions`方法，其中我们定义以下逻辑：如果只有一个分区，我们简单地处理它上面的所有内容。然而，如果我们有多个分区，那么我们在奇数分区上处理奇数长度的字符串，在偶数分区上处理偶数长度的字符串。

### 注意

请注意，我们需要创建与`partitionCount`参数中所述一样多的分区，否则我们将得到`Partitioner returned a wrong number of partitions`错误。

最后，我们创建我们分区器的实例，并使用它执行一个 PLINQ 查询。我们可以看到不同的线程处理奇数长度和偶数长度的字符串。此外，我们可以通过取消注释`WithDegreeOfParallelism`方法并更改其参数值来实验。当参数值为`1`时，将会有顺序的工作项处理，而当增加值时，我们可以看到更多的工作可以并行完成。

# 为 PLINQ 查询创建自定义聚合器

这个配方展示了如何为 PLINQ 查询创建一个自定义聚合函数。

## 准备工作

为了完成这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter7\Recipe6`中找到。

## 如何操作...

要理解 PLINQ 查询的自定义聚合函数的工作原理，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Collections.Concurrent;
    using System.Collections.Generic;
    using System.Linq;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static ConcurrentDictionary<char, int> AccumulateLettersInformation(
        ConcurrentDictionary<char, int> taskTotal , string item)
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
      WriteLine($"{item} type was aggregated on a thread " +
              $"id {CurrentThread.ManagedThreadId}");
      return taskTotal;
    }

    static ConcurrentDictionary<char, int> MergeAccumulators(
        ConcurrentDictionary<char, int> total, ConcurrentDictionary<char, int> taskTotal)
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
      WriteLine("---");
      WriteLine($"Total aggregate value was calculated on a thread " +
              $"id {CurrentThread.ManagedThreadId}");
      return total;
    }

    static IEnumerable<string> GetTypes()
    {
      var types = AppDomain.CurrentDomain
        .GetAssemblies()
        .SelectMany(a => a.GetExportedTypes());

      return from type in types
             where type.Name.StartsWith("Web")
             select type.Name;
    }
    ```

1.  在`Main`方法内部添加以下代码片段：

    ```cs
    var parallelQuery = from t in GetTypes().AsParallel()
                        select t;

    var parallelAggregator = parallelQuery.Aggregate(
      () => new ConcurrentDictionary<char, int>(),
      (taskTotal, item) => AccumulateLettersInformation(taskTotal, item), 
      (total, taskTotal) => MergeAccumulators(total, taskTotal),
      total => total);

    WriteLine();
    WriteLine("There were the following letters in type names:");
    var orderedKeys = from k in parallelAggregator.Keys
              orderby parallelAggregator[k] descending
              select k;

    foreach (var c in orderedKeys)
    {
      WriteLine($"Letter '{c}' ---- {parallelAggregator[c]} times");
    }
    ```

1.  运行程序。

## 工作原理...

在这里，我们实现了能够与 PLINQ 查询一起工作的自定义聚合机制。为了实现这一点，我们必须理解，由于查询正在由多个任务同时并行处理，我们需要提供机制来并行聚合每个任务的结果，然后将这些聚合值组合成一个单一的结果值。

在这个配方中，我们编写了一个聚合函数，用于在 PLINQ 查询中计算字母，它返回`IEnumerable<string>`集合。它计算每个集合项中的所有字母。为了说明并行聚合过程，我们打印出关于哪个线程处理聚合的每个部分的信息。

我们使用在`ParallelEnumerable`类中定义的`Aggregate`扩展方法来聚合 PLINQ 查询结果。它接受四个参数，每个参数都是一个执行聚合过程不同部分的功能。第一个是一个工厂，用于构建聚合器的空初始值。它也被称为种子值。

### 注意

注意，提供给`Aggregate`方法的第一个值实际上不是聚合函数的初始种子值，而是一个构造这个初始种子值的工厂方法。如果你只提供一个实例，它将在所有并行运行的分区中使用，这会导致结果不正确。

第二个函数将每个集合项聚合到分区聚合对象中。我们使用`AccumulateLettersInformation`方法来实现这个函数。它遍历字符串并计算其中的字母。在这里，聚合对象对于每个并行运行的查询分区都是不同的，这就是为什么我们称之为`taskTotal`。

第三个功能是一个高级聚合函数，它从一个分区中获取一个聚合对象并将其合并到一个全局聚合对象中。我们使用`MergeAccumulators`方法来实现它。最后一个函数是一个选择函数，它指定了我们从全局聚合对象中需要的确切数据。

最后，我们打印出聚合结果，按集合项中最常用的字母进行排序。
