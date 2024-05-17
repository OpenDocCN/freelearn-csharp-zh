# 第十二章：使用多任务处理提高性能和可扩展性

本章旨在通过允许多个操作同时发生，以提高您构建的应用程序的性能、可扩展性和用户生产力。

本章我们将涵盖以下主题：

+   理解进程、线程和任务

+   监控性能和资源使用情况

+   异步运行任务

+   同步访问共享资源

+   理解`async`和`await`

# 理解进程、线程和任务

**进程**，例如我们创建的每个控制台应用程序，都分配有内存和线程等资源。

**线程**执行您的代码，逐条语句执行。默认情况下，每个进程只有一个线程，当我们需要同时执行多个任务时，这可能会导致问题。线程还负责跟踪当前已验证的用户以及应遵循的当前语言和区域的任何国际化规则等事项。

Windows 和大多数其他现代操作系统使用**抢占式多任务处理**，它模拟任务的并行执行。它将处理器时间分配给各个线程，为每个线程分配一个**时间片**，一个接一个。当当前线程的时间片结束时，它会被挂起，处理器随后允许另一个线程运行一个时间片。

当 Windows 从一个线程切换到另一个线程时，它会保存当前线程的上下文，并重新加载线程队列中下一个线程之前保存的上下文。这个过程需要时间和资源来完成。

作为开发者，如果您有少量复杂的工作且希望完全控制它们，那么您可以创建和管理单独的`Thread`实例。如果您有一个主线程和多个可以在后台执行的小任务，那么您可以使用`ThreadPool`类将指向这些作为方法实现的任务的委托实例添加到队列中，它们将自动分配给线程池中的线程。

在本章中，我们将使用`Task`类型以更高的抽象级别管理线程。

线程可能需要竞争和等待访问共享资源，例如变量、文件和数据库对象。本章后面您将看到用于管理这些资源的各种类型。

根据任务的不同，将执行任务的线程（工作者）数量加倍并不一定会将完成任务所需的时间减半。事实上，它可能会增加任务的持续时间。

**最佳实践**：切勿假设增加线程数量会提高性能！在未使用多线程的基准代码实现上运行性能测试，然后在使用了多线程的代码实现上再次运行。您还应在尽可能接近生产环境的预生产环境中进行性能测试。

# 监控性能和资源使用

在我们能够改进任何代码的性能之前，我们需要能够监控其速度和效率，以记录一个基准，然后我们可以据此衡量改进。

## 评估类型的效率

对于某个场景，最佳类型是什么？要回答这个问题，我们需要仔细考虑我们所说的“最佳”是什么意思，并通过这一点，我们应该考虑以下因素：

+   **功能性**：这可以通过检查类型是否提供了你所需的功能来决定。

+   **内存大小**：这可以通过类型占用的内存字节数来决定。

+   **性能**：这可以通过类型的运行速度来决定。

+   **未来需求**：这取决于需求和可维护性的变化。

在存储数字等场景中，将会有多种类型具有相同的功能，因此我们需要考虑内存和性能来做出选择。

如果我们需要存储数百万个数字，那么最佳类型将是占用内存字节数最少的那个。但如果我们只需要存储几个数字，而我们又需要对它们进行大量计算，那么最佳类型将是在特定 CPU 上运行最快的那个。

你已经见过使用`sizeof()`函数的情况，它显示了内存中一个类型实例所占用的字节数。当我们存储大量值在更复杂的数据结构中，如数组和列表时，我们需要一种更好的方法来测量内存使用情况。

你可以在网上和书籍中阅读大量建议，但确定哪种类型最适合你的代码的唯一方法是自己比较这些类型。

在下一节中，你将学习如何编写代码来监控使用不同类型时的实际内存需求和性能。

今天，`short`变量可能是最佳选择，但使用`int`变量可能是更好的选择，尽管它在内存中占用两倍的空间。这是因为我们将来可能需要存储更广泛的值。

开发者经常忽视的一个重要指标是维护性。这是衡量另一个程序员为了理解和修改你的代码需要付出多少努力的指标。如果你做出一个不明显的类型选择，并且没有用有帮助的注释解释这个选择，那么可能会让后来需要修复错误或添加功能的程序员感到困惑。

## 使用诊断监控性能和内存

`System.Diagnostics`命名空间包含许多用于监控代码的有用类型。我们将首先查看的有用类型是`Stopwatch`类型：

1.  使用你偏好的编程工具创建一个名为`Chapter12`的新工作区/解决方案。

1.  添加一个类库项目，如以下列表所定义：

    1.  项目模板：**类库** / `classlib`

    1.  工作区/解决方案文件和文件夹：`Chapter12`

    1.  项目文件和文件夹：`MonitoringLib`

1.  添加一个控制台应用程序项目，如下所列：

    1.  项目模板：**控制台应用程序** / `console`

    1.  工作区/解决方案文件和文件夹：`Chapter12`

    1.  项目文件和文件夹：`MonitoringApp`

1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择的项目。

1.  在 Visual Studio Code 中，选择`MonitoringApp`作为活动的 OmniSharp 项目。

1.  在`MonitoringLib`项目中，将`Class1.cs`文件重命名为`Recorder.cs`。

1.  在`MonitoringApp`项目中，添加对`MonitoringLib`类库的项目引用，如下所示：

    ```cs
    <ItemGroup> 
      <ProjectReference
        Include="..\MonitoringLib\MonitoringLib.csproj" />
    </ItemGroup> 
    ```

1.  构建`MonitoringApp`项目。

### 有用的 Stopwatch 和 Process 类型成员

`Stopwatch`类型有一些有用的成员，如下表所示：

| 成员 | 描述 |
| --- | --- |
| `Restart` 方法 | 这会将经过时间重置为零，然后启动计时器。 |
| `Stop` 方法 | 这会停止计时器。 |
| `Elapsed` 属性 | 这是以`TimeSpan`格式存储的经过时间（例如，小时:分钟:秒） |
| `ElapsedMilliseconds` 属性 | 这是以毫秒为单位的经过时间，存储为`Int64`值。 |

`Process`类型有一些有用的成员，如下表所示：

| 成员 | 描述 |
| --- | --- |
| `VirtualMemorySize64` | 这显示了为进程分配的虚拟内存量，单位为字节。 |
| `WorkingSet64` | 这显示了为进程分配的物理内存量，单位为字节。 |

### 实现一个 Recorder 类

我们将创建一个`Recorder`类，使监控时间和内存资源使用变得简单。为了实现我们的`Recorder`类，我们将使用`Stopwatch`和`Process`类：

1.  在`Recorder.cs`中，修改其内容以使用`Stopwatch`实例记录时间，并使用当前`Process`实例记录内存使用情况，如下所示：

    ```cs
    using System.Diagnostics; // Stopwatch
    using static System.Console;
    using static System.Diagnostics.Process; // GetCurrentProcess()
    namespace Packt.Shared;
    public static class Recorder
    {
      private static Stopwatch timer = new();
      private static long bytesPhysicalBefore = 0;
      private static long bytesVirtualBefore = 0;
      public static void Start()
      {
        // force two garbage collections to release memory that is
        // no longer referenced but has not been released yet
        GC.Collect();
        GC.WaitForPendingFinalizers();
        GC.Collect();
        // store the current physical and virtual memory use 
        bytesPhysicalBefore = GetCurrentProcess().WorkingSet64; 
        bytesVirtualBefore = GetCurrentProcess().VirtualMemorySize64; 
        timer.Restart();
      }
      public static void Stop()
      {
        timer.Stop();
        long bytesPhysicalAfter =
          GetCurrentProcess().WorkingSet64;
        long bytesVirtualAfter =
          GetCurrentProcess().VirtualMemorySize64;
        WriteLine("{0:N0} physical bytes used.",
          bytesPhysicalAfter - bytesPhysicalBefore);
        WriteLine("{0:N0} virtual bytes used.",
          bytesVirtualAfter - bytesVirtualBefore);
        WriteLine("{0} time span ellapsed.", timer.Elapsed);
        WriteLine("{0:N0} total milliseconds ellapsed.",
          timer.ElapsedMilliseconds);
      }
    } 
    ```

    `Recorder`类的`Start`方法使用`GC`类型（垃圾收集器）确保在记录已用内存量之前，收集任何当前已分配但未引用的内存。这是一种高级技术，您几乎不应在应用程序代码中使用。

1.  在`Program.cs`中，编写语句以在生成 10,000 个整数的数组时启动和停止`Recorder`，如下所示：

    ```cs
    using Packt.Shared; // Recorder
    using static System.Console;
    WriteLine("Processing. Please wait...");
    Recorder.Start();
    // simulate a process that requires some memory resources...
    int[] largeArrayOfInts = Enumerable.Range(
      start: 1, count: 10_000).ToArray();
    // ...and takes some time to complete
    Thread.Sleep(new Random().Next(5, 10) * 1000);
    Recorder.Stop(); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Processing. Please wait...
    655,360 physical bytes used.
    536,576 virtual bytes used.
    00:00:09.0038702 time span ellapsed.
    9,003 total milliseconds ellapsed. 
    ```

请记住，时间间隔随机在 5 到 10 秒之间，您的结果可能会有所不同。例如，在我的 Mac mini M1 上运行时，虽然物理内存较少，但虚拟内存使用更多，如下所示：

```cs
Processing. Please wait...
294,912 physical bytes used.
10,485,760 virtual bytes used.
00:00:06.0074221 time span ellapsed.
6,007 total milliseconds ellapsed. 
```

## 测量字符串处理的效率

既然您已经了解了如何使用`Stopwatch`和`Process`类型来监控您的代码，我们将使用它们来评估处理`string`变量的最佳方式。

1.  在`Program.cs`中，通过使用多行注释字符`/* */`将之前的语句注释掉。

1.  编写语句以创建一个包含 50,000 个`int`变量的数组，然后使用`string`和`StringBuilder`类用逗号作为分隔符将它们连接起来，如下所示：

    ```cs
    int[] numbers = Enumerable.Range(
      start: 1, count: 50_000).ToArray();
    WriteLine("Using string with +");
    Recorder.Start();
    string s = string.Empty; // i.e. ""
    for (int i = 0; i < numbers.Length; i++)
    {
      s += numbers[i] + ", ";
    }
    Recorder.Stop();
    WriteLine("Using StringBuilder");
    Recorder.Start();
    System.Text.StringBuilder builder = new();
    for (int i = 0; i < numbers.Length; i++)
    {
      builder.Append(numbers[i]);
      builder.Append(", ");
    }
    Recorder.Stop(); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Using string with +
    14,883,072 physical bytes used.
    3,609,728 virtual bytes used.
    00:00:01.6220879 time span ellapsed.
    1,622 total milliseconds ellapsed.
    Using StringBuilder
    12,288 physical bytes used.
    0 virtual bytes used.
    00:00:00.0006038 time span ellapsed.
    0 total milliseconds ellapsed. 
    ```

我们可以总结结果如下：

+   `string`类使用`+`运算符大约使用了 14 MB 的物理内存，1.5 MB 的虚拟内存，耗时 1.5 秒。

+   `StringBuilder`类使用了 12 KB 的物理内存，零虚拟内存，耗时不到 1 毫秒。

在这种情况下，`StringBuilder`在连接文本时速度快了 1000 多倍，内存效率提高了约 10000 倍！这是因为`string`连接每次使用时都会创建一个新的`string`，因为`string`值是不可变的，所以它们可以安全地池化以供重用。`StringBuilder`在追加更多字符时创建一个单一缓冲区。

**最佳实践**：避免在循环内部使用`String.Concat`方法或`+`运算符。改用`StringBuilder`。

既然你已经学会了如何使用.NET 内置类型来衡量代码的性能和资源效率，接下来让我们了解一个提供更复杂性能测量的 NuGet 包。

## 使用 Benchmark.NET 监控性能和内存

有一个流行的.NET 基准测试 NuGet 包，微软在其关于性能改进的博客文章中使用，因此对于.NET 开发者来说，了解其工作原理并用于自己的性能测试是很有益的。让我们看看如何使用它来比较`string`连接和`StringBuilder`的性能：

1.  使用您喜欢的代码编辑器，向名为`Benchmarking`的`Chapter12`解决方案/工作区添加一个新的控制台应用程序。

1.  在 Visual Studio Code 中，选择`Benchmarking`作为活动 OmniSharp 项目。

1.  添加对 Benchmark.NET 的包引用，记住您可以查找最新版本并使用它，而不是我使用的版本，如下所示：

    ```cs
    <ItemGroup>
      <PackageReference Include="BenchmarkDotNet" Version="0.13.1" />
    </ItemGroup> 
    ```

1.  构建项目以恢复包。

1.  在`Program.cs`中，删除现有语句，然后导入运行基准测试的命名空间，如下所示：

    ```cs
    using BenchmarkDotNet.Running; 
    ```

1.  添加一个名为`StringBenchmarks.cs`的新类文件。

1.  在`StringBenchmarks.cs`中，添加语句来定义一个包含每个基准测试所需方法的类，在这种情况下，两个方法都使用`string`连接或`StringBuilder`将二十个数字以逗号分隔进行组合，如下所示：

    ```cs
    using BenchmarkDotNet.Attributes; // [Benchmark]
    public class StringBenchmarks
    {
      int[] numbers;
      public StringBenchmarks()
      {
        numbers = Enumerable.Range(
          start: 1, count: 20).ToArray();
      }
      [Benchmark(Baseline = true)]
      public string StringConcatenationTest()
      {
        string s = string.Empty; // e.g. ""
        for (int i = 0; i < numbers.Length; i++)
        {
          s += numbers[i] + ", ";
        }
        return s;
      }
      [Benchmark]
      public string StringBuilderTest()
      {
        System.Text.StringBuilder builder = new();
        for (int i = 0; i < numbers.Length; i++)
        {
          builder.Append(numbers[i]);
          builder.Append(", ");
        }
        return builder.ToString();
      }
    } 
    ```

1.  在`Program.cs`中，添加一个语句来运行基准测试，如下所示：

    ```cs
    BenchmarkRunner.Run<StringBenchmarks>(); 
    ```

1.  在 Visual Studio 2022 中，在工具栏上，将**解决方案配置**设置为**发布**。

1.  在 Visual Studio Code 中，在终端中使用`dotnet run --configuration Release`命令。

1.  运行控制台应用并注意结果，包括一些报告文件等附属物，以及最重要的，一张总结表显示`string`拼接平均耗时 412.990 ns，而`StringBuilder`平均耗时 275.082 ns，如下部分输出及*图 12.1*所示：

    ```cs
    // ***** BenchmarkRunner: Finish  *****
    // * Export *
      BenchmarkDotNet.Artifacts\results\StringBenchmarks-report.csv
      BenchmarkDotNet.Artifacts\results\StringBenchmarks-report-github.md
      BenchmarkDotNet.Artifacts\results\StringBenchmarks-report.html
    // * Detailed results *
    StringBenchmarks.StringConcatenationTest: DefaultJob
    Runtime = .NET 6.0.0 (6.0.21.37719), X64 RyuJIT; GC = Concurrent Workstation
    Mean = 412.990 ns, StdErr = 2.353 ns (0.57%), N = 46, StdDev = 15.957 ns
    Min = 373.636 ns, Q1 = 413.341 ns, Median = 417.665 ns, Q3 = 420.775 ns, Max = 434.504 ns
    IQR = 7.433 ns, LowerFence = 402.191 ns, UpperFence = 431.925 ns
    ConfidenceInterval = [404.708 ns; 421.273 ns] (CI 99.9%), Margin = 8.282 ns (2.01% of Mean)
    Skewness = -1.51, Kurtosis = 4.09, MValue = 2
    -------------------- Histogram --------------------
    [370.520 ns ; 382.211 ns) | @@@@@@
    [382.211 ns ; 394.583 ns) | @
    [394.583 ns ; 411.300 ns) | @@
    [411.300 ns ; 422.990 ns) | @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    [422.990 ns ; 436.095 ns) | @@@@@
    ---------------------------------------------------
    StringBenchmarks.StringBuilderTest: DefaultJob
    Runtime = .NET 6.0.0 (6.0.21.37719), X64 RyuJIT; GC = Concurrent Workstation
    Mean = 275.082 ns, StdErr = 0.558 ns (0.20%), N = 15, StdDev = 2.163 ns
    Min = 271.059 ns, Q1 = 274.495 ns, Median = 275.403 ns, Q3 = 276.553 ns, Max = 278.030 ns
    IQR = 2.058 ns, LowerFence = 271.409 ns, UpperFence = 279.639 ns
    ConfidenceInterval = [272.770 ns; 277.394 ns] (CI 99.9%), Margin = 2.312 ns (0.84% of Mean)
    Skewness = -0.69, Kurtosis = 2.2, MValue = 2
    -------------------- Histogram --------------------
    [269.908 ns ; 278.682 ns) | @@@@@@@@@@@@@@@
    ---------------------------------------------------
    // * Summary *
    BenchmarkDotNet=v0.13.1, OS=Windows 10.0.19043.1165 (21H1/May2021Update)
    11th Gen Intel Core i7-1165G7 2.80GHz, 1 CPU, 8 logical and 4 physical cores
    .NET SDK=6.0.100
      [Host]     : .NET 6.0.0 (6.0.21.37719), X64 RyuJIT
      DefaultJob : .NET 6.0.0 (6.0.21.37719), X64 RyuJIT
    |                  Method |     Mean |   Error |   StdDev | Ratio | RatioSD |
    |------------------------ |---------:|--------:|---------:|------:|--------:|
    | StringConcatenationTest | 413.0 ns | 8.28 ns | 15.96 ns |  1.00 |    0.00 |
    |       StringBuilderTest | 275.1 ns | 2.31 ns |  2.16 ns |  0.69 |    0.04 |
    // * Hints *
    Outliers
      StringBenchmarks.StringConcatenationTest: Default -> 7 outliers were removed, 14 outliers were detected (376.78 ns..391.88 ns, 440.79 ns..506.41 ns)
      StringBenchmarks.StringBuilderTest: Default       -> 2 outliers were detected (274.68 ns, 274.69 ns)
    // * Legends *
      Mean    : Arithmetic mean of all measurements
      Error   : Half of 99.9% confidence interval
      StdDev  : Standard deviation of all measurements
      Ratio   : Mean of the ratio distribution ([Current]/[Baseline])
      RatioSD : Standard deviation of the ratio distribution ([Current]/[Baseline])
      1 ns    : 1 Nanosecond (0.000000001 sec)
    // ***** BenchmarkRunner: End *****
    // ** Remained 0 benchmark(s) to run **
    Run time: 00:01:13 (73.35 sec), executed benchmarks: 2
    Global total time: 00:01:29 (89.71 sec), executed benchmarks: 2
    // * Artifacts cleanup * 
    ```

    ![](img/B17442_13_02.png)

    图 12.1：总结表显示 StringBuilder 耗时为字符串拼接的 69%

`Outliers`部分尤为有趣，因为它表明不仅`string`拼接比`StringBuilder`慢，而且其耗时也更不稳定。当然，你的结果可能会有所不同。

你已经见识了两种性能测量方法。现在让我们看看如何异步运行任务以潜在提升性能。

# 异步执行任务

为了理解如何同时运行多个任务（同时进行），我们将创建一个需要执行三个方法的控制台应用程序。

需要执行三种方法：第一种耗时 3 秒，第二种耗时 2 秒，第三种耗时 1 秒。为了模拟这项工作，我们可以使用`Thread`类让当前线程休眠指定毫秒数。

## 同步执行多个操作

在我们让任务同时运行之前，我们将同步运行它们，即一个接一个地执行。

1.  使用你偏好的代码编辑器，在`Chapter12`解决方案/工作区中添加一个名为`WorkingWithTasks`的新控制台应用。

1.  在 Visual Studio Code 中，选择`WorkingWithTasks`作为活动 OmniSharp 项目。

1.  在`Program.cs`中，导入用于操作秒表的命名空间（与线程和任务相关的命名空间已隐式导入），并静态导入`Console`，如下代码所示：

    ```cs
    using System.Diagnostics; // Stopwatch
    using static System.Console; 
    ```

1.  在`Program.cs`底部，创建一个方法输出当前线程信息，如下代码所示：

    ```cs
    static void OutputThreadInfo()
    {
      Thread t = Thread.CurrentThread;
      WriteLine(
        "Thread Id: {0}, Priority: {1}, Background: {2}, Name: {3}",
        t.ManagedThreadId, t.Priority,
        t.IsBackground, t.Name ?? "null");
    } 
    ```

1.  在`Program.cs`底部，添加三个模拟工作的方法，如下代码所示：

    ```cs
    static void MethodA()
    {
      WriteLine("Starting Method A...");
      OutputThreadInfo();
      Thread.Sleep(3000); // simulate three seconds of work
      WriteLine("Finished Method A.");
    }
    static void MethodB()
    {
      WriteLine("Starting Method B...");
      OutputThreadInfo();
      Thread.Sleep(2000); // simulate two seconds of work
      WriteLine("Finished Method B.");
    }
    static void MethodC()
    {
      WriteLine("Starting Method C...");
      OutputThreadInfo();
      Thread.Sleep(1000); // simulate one second of work
      WriteLine("Finished Method C.");
    } 
    ```

1.  在`Program.cs`顶部，添加语句调用输出线程信息的方法，定义并启动秒表，调用三个模拟工作方法，然后输出经过的毫秒数，如下代码所示：

    ```cs
    OutputThreadInfo();
    Stopwatch timer = Stopwatch.StartNew();
    WriteLine("Running methods synchronously on one thread."); 
    MethodA();
    MethodB();
    MethodC();
    WriteLine($"{timer.ElapsedMilliseconds:#,##0}ms elapsed."); 
    ```

1.  运行代码，查看结果，并注意当仅有一个未命名前台线程执行任务时，所需总时间略超过 6 秒，如下输出所示：

    ```cs
    Thread Id: 1, Priority: Normal, Background: False, Name: null
    Running methods synchronously on one thread.
    Starting Method A...
    Thread Id: 1, Priority: Normal, Background: False, Name: null
    Finished Method A.
    Starting Method B...
    Thread Id: 1, Priority: Normal, Background: False, Name: null
    Finished Method B.
    Starting Method C...
    Thread Id: 1, Priority: Normal, Background: False, Name: null
    Finished Method C.
    6,017ms elapsed. 
    ```

## 使用任务异步执行多个操作

`Thread`类自.NET 的首个版本起就已存在，可用于创建新线程并管理它们，但直接使用可能较为棘手。

.NET Framework 4.0 于 2010 年引入了`Task`类，它是对线程的封装，使得创建和管理更为简便。通过管理多个封装在任务中的线程，我们的代码将能够同时执行，即异步执行。

每个`Task`都有一个`Status`属性和一个`CreationOptions`属性。`Task`有一个`ContinueWith`方法，可以通过`TaskContinuationOptions`枚举进行定制，并可以使用`TaskFactory`类进行管理。

### 启动任务

我们将探讨三种使用`Task`实例启动方法的方式。GitHub 仓库中的链接指向了讨论这些方法优缺点的文章。每种方法的语法略有不同，但它们都定义了一个`Task`并启动它：

1.  注释掉对三个方法及其相关控制台消息的调用，并添加语句以创建和启动三个任务，每个方法一个，如下所示：

    ```cs
    OutputThreadInfo();
    Stopwatch timer = Stopwatch.StartNew();
    **/***
    WriteLine("Running methods synchronously on one thread.");
    MethodA();
    MethodB();
    MethodC();
    ***/**
    **WriteLine(****"Running methods asynchronously on multiple threads."****);** 
    **Task taskA =** **new****(MethodA);**
    **taskA.Start();**
    **Task taskB = Task.Factory.StartNew(MethodB);** 
    **Task taskC = Task.Run(MethodC);**
    WriteLine($"{timer.ElapsedMilliseconds:#,##0}ms elapsed."); 
    ```

1.  运行代码，查看结果，并注意耗时毫秒数几乎立即出现。这是因为三个方法现在正由线程池分配的三个新后台工作线程执行，如下所示：

    ```cs
    Thread Id: 1, Priority: Normal, Background: False, Name: null
    Running methods asynchronously on multiple threads.
    Starting Method A...
    Thread Id: 4, Priority: Normal, Background: True, Name: .NET ThreadPool Worker
    Starting Method C...
    Thread Id: 7, Priority: Normal, Background: True, Name: .NET ThreadPool Worker
    Starting Method B...
    Thread Id: 6, Priority: Normal, Background: True, Name: .NET ThreadPool Worker
    6ms elapsed. 
    ```

甚至有可能控制台应用在任务有机会启动并写入控制台之前就结束了！

## 等待任务

有时，你需要等待一个任务完成后再继续。为此，你可以使用`Task`实例上的`Wait`方法，或者使用`Task`数组上的`WaitAll`或`WaitAny`静态方法，如下表所述：

| 方法 | 描述 |
| --- | --- |
| `t.Wait()` | 这会等待名为`t`的任务实例完成执行。 |
| `Task.WaitAny(Task[])` | 这会等待数组中的任意任务完成执行。 |
| `Task.WaitAll(Task[])` | 这会等待数组中的所有任务完成执行。 |

### 使用任务的等待方法

让我们看看如何使用这些等待方法来解决我们控制台应用的问题。

1.  在`Program.cs`中，在创建三个任务和输出耗时之间添加语句，将三个任务的引用合并到一个数组中，并将其传递给`WaitAll`方法，如下所示：

    ```cs
    Task[] tasks = { taskA, taskB, taskC };
    Task.WaitAll(tasks); 
    ```

1.  运行代码并查看结果，注意原始线程将在调用`WaitAll`时暂停，等待所有三个任务完成后再输出耗时，耗时略超过 3 秒，如下所示：

    ```cs
    Id: 1, Priority: Normal, Background: False, Name: null
    Running methods asynchronously on multiple threads.
    Starting Method A...
    Id: 6, Priority: Normal, Background: True, Name: .NET ThreadPool Worker
    Starting Method B...
    Id: 7, Priority: Normal, Background: True, Name: .NET ThreadPool Worker
    Starting Method C...
    Id: 4, Priority: Normal, Background: True, Name: .NET ThreadPool Worker
    Finished Method C.
    Finished Method B.
    Finished Method A.
    3,013ms elapsed. 
    ```

三个新线程同时执行其代码，并且它们可能以任意顺序启动。`MethodC`应该最先完成，因为它仅需 1 秒，接着是耗时 2 秒的`MethodB`，最后是耗时 3 秒的`MethodA`。

然而，实际的 CPU 使用对结果有很大影响。是 CPU 为每个进程分配时间片以允许它们执行其线程。你无法控制方法何时运行。

## 继续执行另一个任务

如果所有三个任务都能同时执行，那么等待所有任务完成就是我们所需做的全部。然而，通常一个任务依赖于另一个任务的输出。为了处理这种情况，我们需要定义**延续任务**。

我们将创建一些方法来模拟对返回货币金额的网络服务的调用，然后需要使用该金额来检索数据库中有多少产品成本超过该金额。从第一个方法返回的结果需要输入到第二个方法的输入中。这次，我们将使用`Random`类而不是等待固定时间，为每次方法调用等待 2 到 4 秒之间的随机间隔来模拟工作。

1.  在`Program.cs`底部，添加两个方法来模拟调用网络服务和数据库存储过程，如下面的代码所示：

    ```cs
    static decimal CallWebService()
    {
      WriteLine("Starting call to web service...");
      OutputThreadInfo();
      Thread.Sleep((new Random()).Next(2000, 4000));
      WriteLine("Finished call to web service.");
      return 89.99M;
    }
    static string CallStoredProcedure(decimal amount)
    {
      WriteLine("Starting call to stored procedure...");
      OutputThreadInfo();
      Thread.Sleep((new Random()).Next(2000, 4000));
      WriteLine("Finished call to stored procedure.");
      return $"12 products cost more than {amount:C}.";
    } 
    ```

1.  通过将它们包裹在多行注释字符`/* */`中来注释掉对前三个任务的调用。保留输出经过的毫秒数的语句。

1.  在现有语句之前添加语句以输出总时间，如下面的代码所示：

    ```cs
    WriteLine("Passing the result of one task as an input into another."); 
    Task<string> taskServiceThenSProc = Task.Factory
      .StartNew(CallWebService) // returns Task<decimal>
      .ContinueWith(previousTask => // returns Task<string>
        CallStoredProcedure(previousTask.Result));
    WriteLine($"Result: {taskServiceThenSProc.Result}"); 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Thread Id: 1, Priority: Normal, Background: False, Name: null
    Passing the result of one task as an input into another.
    Starting call to web service...
    Thread Id: 4, Priority: Normal, Background: True, Name: .NET ThreadPool Worker
    Finished call to web service.
    Starting call to stored procedure...
    Thread Id: 6, Priority: Normal, Background: True, Name: .NET ThreadPool Worker
    Finished call to stored procedure.
    Result: 12 products cost more than £89.99.
    5,463ms elapsed. 
    ```

您可能会看到不同的线程运行网络服务和存储过程调用，如上面的输出所示（线程 4 和 6），或者同一线程可能会被重用，因为它不再忙碌。

## 嵌套和子任务

除了定义任务之间的依赖关系外，您还可以定义嵌套和子任务。**嵌套任务**是在另一个任务内部创建的任务。**子任务**是必须在其父任务允许完成之前完成的嵌套任务。

让我们探索这些类型的任务是如何工作的：

1.  使用您喜欢的代码编辑器，在`Chapter12`解决方案/工作区中添加一个名为`NestedAndChildTasks`的新控制台应用程序。

1.  在 Visual Studio Code 中，选择`NestedAndChildTasks`作为活动 OmniSharp 项目。

1.  在`Program.cs`中，删除现有语句，静态导入`Console`，然后添加两个方法，其中一个方法启动一个任务来运行另一个方法，如下面的代码所示：

    ```cs
    static void OuterMethod()
    {
      WriteLine("Outer method starting...");
      Task innerTask = Task.Factory.StartNew(InnerMethod);
      WriteLine("Outer method finished.");
    }
    static void InnerMethod()
    {
      WriteLine("Inner method starting...");
      Thread.Sleep(2000);
      WriteLine("Inner method finished.");
    } 
    ```

1.  在方法上方，添加语句以启动一个任务来运行外部方法并在停止前等待其完成，如下面的代码所示：

    ```cs
    Task outerTask = Task.Factory.StartNew(OuterMethod);
    outerTask.Wait();
    WriteLine("Console app is stopping."); 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Outer method starting...
    Inner method starting...
    Outer method finished.
    Console app is stopping. 
    ```

    请注意，尽管我们等待外部任务完成，但其内部任务不必同时完成。事实上，外部任务可能完成，控制台应用程序可能结束，甚至在内部任务开始之前！

    要将这些嵌套任务链接为父任务和子任务，我们必须使用一个特殊选项。

1.  修改定义内部任务的现有代码，添加一个`TaskCreationOption`值为`AttachedToParent`，如下面的代码中突出显示所示：

    ```cs
    Task innerTask = Task.Factory.StartNew(InnerMethod,
      **TaskCreationOptions.AttachedToParent**); 
    ```

1.  运行代码，查看结果，并注意内部任务必须在完成外部任务之前完成，如下面的输出所示：

    ```cs
    Outer method starting...
    Inner method starting...
    Outer method finished.
    Inner method finished.
    Console app is stopping. 
    ```

尽管`OuterMethod`可以在`InnerMethod`之前完成，如其在控制台上的输出所示，但其任务必须等待，如控制台在内外任务都完成之前不会停止所示。

## 围绕其他对象包装任务

有时你可能有一个想要异步的方法，但返回的结果本身不是一个任务。你可以将返回值包装在一个成功完成的任务中，返回一个异常，或者通过使用下表中所示的方法来表示任务已被取消：

| 方法 | 描述 |
| --- | --- |
| `FromResult<TResult>(TResult)` | 创建一个`Task<TResult>`对象，其`Result`属性是非任务结果，其`Status`属性是`RanToCompletion`。 |
| `FromException<TResult>(Exception)` | 创建一个因指定异常而完成的`Task<TResult>`。 |
| `FromCanceled<TResult>(CancellationToken)` | 创建一个因指定取消令牌而完成的`Task<TResult>`。 |

这些方法在你需要时很有用：

+   实现具有异步方法的接口，但你的实现是同步的。这在网站和服务中很常见。

+   在单元测试期间模拟异步实现。

在《第七章：打包和分发.NET 类型》中，我们创建了一个类库，用于检查有效的 XML、密码和十六进制代码。

如果我们想让那些方法符合要求返回`Task<T>`的接口，我们可以使用这些有用的方法，如下面的代码所示：

```cs
using System.Text.RegularExpressions;
namespace Packt.Shared;
public static class StringExtensions
{
  public static Task<bool> IsValidXmlTagAsync(this string input)
  {
    if (input == null)
    {
      return Task.FromException<bool>(
        new ArgumentNullException("Missing input parameter"));
    }
    if (input.Length == 0)
    {
      return Task.FromException<bool>(
        new ArgumentException("input parameter is empty."));
    }
    return Task.FromResult(Regex.IsMatch(input,
      @"^<([a-z]+)([^<]+)*(?:>(.*)<\/\1>|\s+\/>)$"));
  }
  // other methods
} 
```

如果你需要实现的方法返回一个`Task`（相当于同步方法中的`void`），那么你可以返回一个预定义的已完成`Task`对象，如下面的代码所示：

```cs
public Task DeleteCustomerAsync()
{
  // ...
  return Task.CompletedTask;
} 
```

# 同步访问共享资源

当多个线程同时执行时，有可能两个或更多线程会同时访问同一变量或其他资源，从而可能导致问题。因此，你应该仔细考虑如何使你的代码**线程安全**。

实现线程安全最简单的机制是使用对象变量作为标志或交通灯，以指示何时对共享资源应用了独占锁。

在威廉·戈尔丁的《*蝇王*》中，皮吉和拉尔夫发现了一个海螺壳，并用它召集会议。男孩们自行制定了“海螺规则”，决定只有持有海螺的人才能发言。

我喜欢将用于实现线程安全代码的对象变量命名为“海螺”。当一个线程持有海螺时，其他任何线程都不应访问由该海螺表示的共享资源。请注意，我说的是“不应”。只有尊重海螺的代码才能实现同步访问。海螺不是锁。

我们将探讨几种可用于同步访问共享资源的类型：

+   `Monitor`：一个可被多个线程用来检查是否应在同一进程内访问共享资源的对象。

+   `Interlocked`：一个用于在 CPU 级别操作简单数值类型的对象。

## 多线程访问资源

1.  使用你偏好的代码编辑器，在`Chapter12`解决方案/工作区中添加一个名为`SynchronizingResourceAccess`的新控制台应用。

1.  在 Visual Studio Code 中，选择`SynchronizingResourceAccess`作为活动 OmniSharp 项目。

1.  在`Program.cs`中，删除现有语句，然后添加执行以下操作的语句：

    +   导入诊断类型（如`Stopwatch`）的命名空间。

    +   静态导入`Console`类型。

    +   在`Program.cs`底部，创建一个具有两个字段的静态类：

        +   生成随机等待时间的字段。

        +   一个`string`字段用于存储消息（这是一个共享资源）。

    +   在类上方，创建两个静态方法，它们在循环中五次向共享`string`添加字母 A 或 B，并为每次迭代等待最多 2 秒的随机间隔：

    ```cs
    static void MethodA()
    {
      for (int i = 0; i < 5; i++)
      {
        Thread.Sleep(SharedObjects.Random.Next(2000));
        SharedObjects.Message += "A";
        Write(".");
      }
    }
    static void MethodB()
    {
      for (int i = 0; i < 5; i++)
      {
        Thread.Sleep(SharedObjects.Random.Next(2000));
        SharedObjects.Message += "B";
        Write(".");
      }
    }
    static class SharedObjects
    {
      public static Random Random = new();
      public static string? Message; // a shared resource
    } 
    ```

1.  在命名空间导入之后，编写语句以使用一对任务在单独的线程上执行两个方法，并在输出经过的毫秒数之前等待它们完成，如下面的代码所示：

    ```cs
    WriteLine("Please wait for the tasks to complete.");
    Stopwatch watch = Stopwatch.StartNew();
    Task a = Task.Factory.StartNew(MethodA);
    Task b = Task.Factory.StartNew(MethodB);

    Task.WaitAll(new Task[] { a, b });
    WriteLine();
    WriteLine($"Results: {SharedObjects.Message}.");
    WriteLine($"{watch.ElapsedMilliseconds:N0} elapsed milliseconds."); 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Please wait for the tasks to complete.
    ..........
    Results: BABABAABBA.
    5,753 elapsed milliseconds. 
    ```

这表明两个线程都在并发地修改消息。在实际应用中，这可能是个问题。但我们可以通过对海螺对象应用互斥锁，并让两个方法在修改共享资源前自愿检查海螺，来防止并发访问，我们将在下一节中这样做。

## 对海螺应用互斥锁

现在，让我们使用海螺确保一次只有一个线程访问共享资源。

1.  在`SharedObjects`中，声明并实例化一个`object`变量作为海螺，如下面的代码所示：

    ```cs
    public static object Conch = new(); 
    ```

1.  在`MethodA`和`MethodB`中，在`for`循环周围添加一个`lock`语句，以锁定海螺，如下面的高亮代码所示：

    ```cs
    **lock** **(SharedObjects.Conch)**
    **{**
      for (int i = 0; i < 5; i++)
      {
        Thread.Sleep(SharedObjects.Random.Next(2000));
        SharedObjects.Message += "A";
        Write(".");
      }
    **}** 
    ```

    **最佳实践**：请注意，由于检查海螺是自愿的，如果你只在两个方法中的一个使用`lock`语句，共享资源将继续被两个方法访问。确保所有访问共享资源的方法都尊重海螺。

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Please wait for the tasks to complete.
    ..........
    Results: BBBBBAAAAA.
    10,345 elapsed milliseconds. 
    ```

尽管耗时更长，但一次只能有一个方法访问共享资源。`MethodA`或`MethodB`可以先开始。一旦某个方法完成了对共享资源的操作，海螺就会被释放，另一个方法就有机会执行其任务。

### 理解锁语句

你可能会好奇`lock`语句在“锁定”对象变量时做了什么（提示：它并没有锁定对象！），如下面的代码所示：

```cs
lock (SharedObjects.Conch)
{
  // work with shared resource
} 
```

C#编译器将`lock`语句转换为使用`Monitor`类*进入*和*退出*海螺对象的`try`-`finally`语句（我喜欢将其视为*获取*和*释放*海螺对象），如下面的代码所示：

```cs
try
{
  Monitor.Enter(SharedObjects.Conch);
  // work with shared resource
}
finally
{
  Monitor.Exit(SharedObjects.Conch);
} 
```

当线程对任何对象（即引用类型）调用`Monitor.Enter`时，它会检查是否有其他线程已经获取了海螺。如果已经获取，线程等待。如果没有，线程获取海螺并继续处理共享资源。一旦线程完成其工作，它调用`Monitor.Exit`，释放海螺。如果另一个线程正在等待，现在它可以获取海螺并执行其工作。这要求所有线程通过适当调用`Monitor.Enter`和`Monitor.Exit`来尊重海螺。

### 避免死锁

了解`lock`语句如何被编译器转换为`Monitor`类上的方法调用也很重要，因为使用`lock`语句可能导致死锁。

当存在两个或多个共享资源（每个资源都有一个海螺来监控当前哪个线程正在处理该共享资源）时，可能会发生死锁，如果以下事件序列发生：

+   线程 X“锁定”海螺 A 并开始处理共享资源 A。

+   线程 Y“锁定”海螺 B 并开始处理共享资源 B。

+   线程 X 在仍在处理资源 A 的同时，也需要与资源 B 合作，因此它试图“锁定”海螺 B，但由于线程 Y 已经拥有海螺 B 而被阻塞。

+   线程 Y 在仍在处理资源 B 的同时，也需要与资源 A 合作，因此它试图“锁定”海螺 A，但由于线程 X 已经拥有海螺 A 而被阻塞。

防止死锁的一种方法是在尝试获取锁时指定超时。为此，你必须手动使用`Monitor`类而不是使用`lock`语句。

1.  修改你的代码，将`lock`语句替换为尝试在超时后进入海螺的代码，并输出错误，然后退出监视器，允许其他线程进入监视器，如下所示高亮显示的代码：

    ```cs
    **try**
    **{**
    **if** **(Monitor.TryEnter(SharedObjects.Conch, TimeSpan.FromSeconds(****15****)))**
      {
        for (int i = 0; i < 5; i++)
        {
          Thread.Sleep(SharedObjects.Random.Next(2000));
          SharedObjects.Message += "A";
          Write(".");
        }
      }
    **else**
     **{**
     **WriteLine(****"Method A timed out when entering a monitor on conch."****);**
     **}**
    **}**
    **finally**
    **{**
     **Monitor.Exit(SharedObjects.Conch);**
    **}** 
    ```

1.  运行代码并查看结果，结果应与之前相同（尽管 A 或 B 可能首先抓住海螺），但这是更好的代码，因为它将防止潜在的死锁。

**最佳实践**：仅在你能编写避免潜在死锁的代码时使用`lock`关键字。如果你无法避免潜在死锁，则始终使用`Monitor.TryEnter`方法代替`lock`，并结合`try`-`finally`语句，以便你可以提供超时，如果发生死锁，其中一个线程将退出。你可以在以下链接阅读更多关于良好线程实践的内容：[`docs.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices`](https://docs.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices)

## 同步事件

在*第六章*，*实现接口和继承类*中，你学习了如何引发和处理事件。但.NET 事件不是线程安全的，因此你应该避免在多线程场景中使用它们，并遵循我之前展示的标准事件引发代码。

在了解到.NET 事件不是线程安全的之后，一些开发者尝试在添加和移除事件处理程序或触发事件时使用独占锁，如下面的代码所示：

```cs
// event delegate field
public event EventHandler Shout;
// conch
private object eventLock = new();
// method
public void Poke()
{
  lock (eventLock) // bad idea
  {
    // if something is listening...
    if (Shout != null)
    {
      // ...then call the delegate to raise the event
      Shout(this, EventArgs.Empty);
    }
  }
} 
```

**最佳实践**：您可以在以下链接中了解更多关于事件和线程安全的信息：[`docs.microsoft.com/en-us/archive/blogs/cburrows/field-like-events-considered-harmful`](https://docs.microsoft.com/en-us/archive/blogs/cburrows/field-like-events-considered-harmful)

但这很复杂，正如 Stephen Cleary 在以下博客文章中所解释的：[`blog.stephencleary.com/2009/06/threadsafe-events.html`](https://blog.stephencleary.com/2009/06/threadsafe-events.html)

## 使 CPU 操作原子化

原子一词来自希腊语**atomos**，意为*不可分割*。理解多线程中哪些操作是原子的很重要，因为如果它们不是原子的，那么它们可能会在操作中途被另一个线程中断。C#的增量运算符是原子的吗，如下面的代码所示？

```cs
int x = 3;
x++; // is this an atomic CPU operation? 
```

它不是原子的！递增一个整数需要以下三个 CPU 操作：

1.  从实例变量加载一个值到寄存器。

1.  递增该值。

1.  将值存储在实例变量中。

一个线程在执行前两步后可能会被中断。第二个线程随后可以执行所有三个步骤。当第一个线程恢复执行时，它将覆盖变量中的值，第二个线程执行的增减操作的效果将会丢失！

有一个名为`Interlocked`的类型，可以对值类型（如整数和浮点数）执行原子操作。让我们看看它的实际应用：

1.  在`SharedObjects`类中声明另一个字段，用于计数已发生的操作次数，如下面的代码所示：

    ```cs
    public static int Counter; // another shared resource 
    ```

1.  在方法 A 和 B 中，在`for`语句内并在修改`string`值后，添加一个语句以安全地递增计数器，如下面的代码所示：

    ```cs
    Interlocked.Increment(ref SharedObjects.Counter); 
    ```

1.  输出经过的时间后，将计数器的当前值写入控制台，如下面的代码所示：

    ```cs
    WriteLine($"{SharedObjects.Counter} string modifications."); 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Please wait for the tasks to complete.
    ..........
    Results: BBBBBAAAAA.
    13,531 elapsed milliseconds.
    **10 string modifications.** 
    ```

细心的读者会意识到，现有的海螺对象保护了锁定代码块内访问的所有共享资源，因此在这个特定的例子中实际上不需要使用`Interlocked`。但如果我们没有保护另一个像`Message`这样的共享资源，那么使用`Interlocked`将是必要的。

## 应用其他类型的同步

`Monitor`和`Interlocked`是互斥锁，它们简单有效，但有时，您需要更高级的选项来同步对共享资源的访问，如下表所示：

| 类型 | 描述 |
| --- | --- |
| `ReaderWriterLock`和`ReaderWriterLockSlim` | 这些允许多个线程处于**读模式**，一个线程处于**写模式**，拥有写锁的独占所有权，以及一个线程，该线程具有对资源的读访问权限，并处于**可升级读模式**，从中线程可以升级到写模式，而无需放弃其对资源的读访问权限。 |
| `Mutex` | 类似于`Monitor`，它为共享资源提供独占访问，但用于进程间同步。 |
| `Semaphore`和`SemaphoreSlim` | 这些通过定义槽限制可以同时访问资源或资源池的线程数量。这被称为资源节流，而不是资源锁定。 |
| `AutoResetEvent`和`ManualResetEvent` | 事件等待句柄允许线程通过相互发送信号和等待彼此的信号来同步活动。 |

# 理解异步和等待

C# 5 在处理`Task`类型时引入了两个 C#关键字。它们特别适用于以下情况：

+   为**图形用户界面**(**GUI**)实现多任务处理。

+   提升 Web 应用和 Web 服务的可扩展性。

在*第十五章*，*使用模型-视图-控制器模式构建网站*中，我们将看到`async`和`await`关键字如何提升网站的可扩展性。

在*第十九章*，*使用.NET MAUI 构建移动和桌面应用*中，我们将看到`async`和`await`关键字如何实现 GUI 的多任务处理。

但现在，让我们先学习这两个 C#关键字被引入的理论原因，之后您将看到它们在实践中的应用。

## 提高控制台应用的响应性

控制台应用程序的一个限制是，您只能在标记为`async`的方法中使用`await`关键字，但 C# 7 及更早版本不允许将`Main`方法标记为异步！幸运的是，C# 7.1 引入了一个新特性，即支持`Main`中的`async`：

1.  使用您偏好的代码编辑器，向`Chapter12`解决方案/工作区中添加一个名为`AsyncConsole`的新控制台应用。

1.  在 Visual Studio Code 中，选择`AsyncConsole`作为活动的 OmniSharp 项目。

1.  在`Program.cs`中，删除现有语句并静态导入`Console`，如下所示：

    ```cs
    using static System.Console; 
    ```

1.  添加语句以创建`HttpClient`实例，请求 Apple 主页，并输出其字节数，如下所示：

    ```cs
    HttpClient client = new();
    HttpResponseMessage response =
      await client.GetAsync("http://www.apple.com/");
    WriteLine("Apple's home page has {0:N0} bytes.",
      response.Content.Headers.ContentLength); 
    ```

1.  构建项目并注意它成功构建。在.NET 5 及更早版本中，您会看到一条错误消息，如下所示：

    ```cs
    Program.cs(14,9): error CS4033: The 'await' operator can only be used within an async method. Consider marking this method with the 'async' modifier and changing its return type to 'Task'. [/Users/markjprice/Code/ Chapter12/AsyncConsole/AsyncConsole.csproj] 
    ```

1.  您本需要向`Main`方法添加`async`关键字并将其返回类型更改为`Task`。使用.NET 6 及更高版本，控制台应用项目模板利用顶级程序功能自动为您定义具有异步`Main`方法的`Program`类。

1.  运行代码并查看结果，由于苹果经常更改其主页，因此结果可能会有不同的字节数，如下面的输出所示：

    ```cs
    Apple's home page has 40,252 bytes. 
    ```

## 提高 GUI 应用程序的响应性

到目前为止，本书中我们只构建了控制台应用程序。当构建 Web 应用程序、Web 服务以及带有 GUI 的应用程序（如 Windows 桌面和移动应用程序）时，程序员的生活会变得更加复杂。

原因之一是，对于图形用户界面（GUI）应用程序，存在一个特殊的线程：**用户界面**（**UI**）线程。

在 GUI 中工作的两条规则：

+   不要在 UI 线程上执行长时间运行的任务。

+   不要在除 UI 线程以外的任何线程上访问 UI 元素。

为了处理这些规则，程序员过去不得不编写复杂的代码来确保长时间运行的任务由非 UI 线程执行，但一旦完成，任务的结果会安全地传递给 UI 线程以呈现给用户。这很快就会变得混乱！

幸运的是，使用 C# 5 及更高版本，你可以使用`async`和`await`。它们允许你继续以同步方式编写代码，这使得代码保持清晰易懂，但在底层，C#编译器创建了一个复杂的**状态机**并跟踪运行线程。这有点神奇！

让我们看一个例子。我们将使用 WPF 构建一个 Windows 桌面应用程序，该应用程序从 SQL Server 数据库中的 Northwind 数据库获取员工信息，使用低级类型如`SqlConnection`、`SqlCommand`和`SqlDataReader`。只有当你拥有 Windows 和存储在 SQL Server 中的 Northwind 数据库时，你才能完成此任务。这是本书中唯一不跨平台且现代的部分（WPF 已有 16 年历史！）。

此时，我们专注于使 GUI 应用程序具有响应性。你将在*第十九章*，*使用.NET MAUI 构建移动和桌面应用程序*中学习 XAML 和构建跨平台 GUI 应用程序。由于本书其他部分不涉及 WPF，我认为这是一个很好的机会，至少可以看到一个使用 WPF 构建的示例应用程序，即使我们不详细讨论它。

我们开始吧！

1.  如果你使用的是 Windows 上的 Visual Studio 2022，请向`Chapter12`解决方案中添加一个名为`WpfResponsive`的**WPF 应用程序[C#]**项目。如果你使用的是 Visual Studio Code，请使用以下命令：`dotnet new wpf`。

1.  在项目文件中，注意输出类型是 Windows EXE，目标框架是面向 Windows 的.NET 6（它不会在其他平台如 macOS 和 Linux 上运行），并且项目使用了 WPF。

1.  向项目中添加对`Microsoft.Data.SqlClient`的包引用，如下面的标记中突出显示的那样：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>WinExe</OutputType>
        <TargetFramework>net6.0-windows</TargetFramework>
        <Nullable>enable</Nullable>
        <UseWPF>true</UseWPF>
      </PropertyGroup>
     **<ItemGroup>**
     **<PackageReference Include=****"Microsoft.Data.SqlClient"** **Version=****"3.0.0"** **/>**
     **</ItemGroup>**
    </Project> 
    ```

1.  构建项目以恢复包。

1.  在`MainWindow.xaml`中，在`<Grid>`元素内，添加元素以定义两个按钮、一个文本框和一个列表框，它们在堆栈面板中垂直布局，如下面的标记中突出显示的那样：

    ```cs
    <Grid>
    **<****StackPanel****>**
    **<****Button****Name****=****"GetEmployeesSyncButton"**
    **Click****=****"GetEmployeesSyncButton_Click"****>**
     **Get Employees Synchronously****</****Button****>**
    **<****Button****Name****=****"GetEmployeesAsyncButton"**
    **Click****=****"GetEmployeesAsyncButton_Click"****>**
     **Get Employees Asynchronously****</****Button****>**
    **<****TextBox****HorizontalAlignment****=****"Stretch"****Text****=****"Type in here"** **/>**
    **<****ListBox****Name****=****"EmployeesListBox"****Height****=****"400"** **/>**
    **</****StackPanel****>**
    </Grid> 
    ```

    Windows 上的 Visual Studio 2022 对构建 WPF 应用提供了良好的支持，并在编辑代码和 XAML 标记时提供 IntelliSense。Visual Studio Code 则不支持。

1.  在`MainWindow.xaml.cs`中，在`MainWindow`类中，导入`System.Diagnostics`和`Microsoft.Data.SqlClient`命名空间，然后创建两个`string`常量用于数据库连接字符串和 SQL 语句，并为两个按钮的点击创建事件处理程序，使用这些`string`常量打开与 Northwind 数据库的连接，并在列表框中填充所有员工的 ID 和姓名，如下所示：

    ```cs
    private const string connectionString = 
      "Data Source=.;" +
      "Initial Catalog=Northwind;" +
      "Integrated Security=true;" +
      "MultipleActiveResultSets=true;";
    private const string sql =
      "WAITFOR DELAY '00:00:05';" +
      "SELECT EmployeeId, FirstName, LastName FROM Employees";
    private void GetEmployeesSyncButton_Click(object sender, RoutedEventArgs e)
    {
      Stopwatch timer = Stopwatch.StartNew();
      using (SqlConnection connection = new(connectionString))
      {
        connection.Open();
        SqlCommand command = new(sql, connection);
        SqlDataReader reader = command.ExecuteReader();
        while (reader.Read())
        {
          string employee = string.Format("{0}: {1} {2}",
            reader.GetInt32(0), reader.GetString(1), reader.GetString(2));
          EmployeesListBox.Items.Add(employee);
        }
        reader.Close();
        connection.Close();
      }
      EmployeesListBox.Items.Add($"Sync: {timer.ElapsedMilliseconds:N0}ms");
    }
    private async void GetEmployeesAsyncButton_Click(
      object sender, RoutedEventArgs e)
    {
      Stopwatch timer = Stopwatch.StartNew();
      using (SqlConnection connection = new(connectionString))
      {
        await connection.OpenAsync();
        SqlCommand command = new(sql, connection);
        SqlDataReader reader = await command.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
          string employee = string.Format("{0}: {1} {2}",
            await reader.GetFieldValueAsync<int>(0), 
            await reader.GetFieldValueAsync<string>(1), 
            await reader.GetFieldValueAsync<string>(2));
          EmployeesListBox.Items.Add(employee);
        }
        await reader.CloseAsync();
        await connection.CloseAsync();
      }
      EmployeesListBox.Items.Add($"Async: {timer.ElapsedMilliseconds:N0}ms");
    } 
    ```

    注意以下内容：

    +   SQL 语句使用 SQL Server 命令`WAITFOR DELAY`模拟耗时五秒的处理过程，然后从`Employees`表中选择三个列。

    +   `GetEmployeesSyncButton_Click`事件处理程序使用同步方法打开连接并获取员工行。

    +   `GetEmployeesAsyncButton_Click`事件处理程序标记为`async`，并使用带有`await`关键字的异步方法打开连接并获取员工行。

    +   两个事件处理程序均使用秒表记录操作耗费的毫秒数，并将其添加到列表框中。

1.  启动 WPF 应用，无需调试。

1.  点击文本框，输入一些文本，注意 GUI 响应。

1.  点击**同步获取员工**按钮。

1.  尝试点击文本框，注意 GUI 无响应。

1.  等待至少五秒钟，直到列表框中填满员工信息。

1.  点击文本框，输入一些文本，注意 GUI 再次响应。

1.  点击**异步获取员工**按钮。

1.  点击文本框，输入一些文本，注意在执行操作时 GUI 仍然响应。继续输入，直到列表框中填满员工信息。

1.  注意两次操作的时间差异。同步获取数据时 UI 被阻塞，而异步获取数据时 UI 保持响应。

1.  关闭 WPF 应用。

## 提升 Web 应用和 Web 服务的可扩展性。

`async`和`await`关键字在构建网站、应用程序和服务时也可应用于服务器端。从客户端应用程序的角度来看，没有任何变化（或者他们甚至可能注意到请求返回所需时间略有增加）。因此，从单个客户端的角度来看，使用`async`和`await`在服务器端实现多任务处理会使他们的体验变差！

在服务器端，创建额外的、成本较低的工作线程来等待长时间运行的任务完成，以便昂贵的 I/O 线程可以处理其他客户端请求，而不是被阻塞。这提高了 Web 应用或服务的整体可扩展性。可以同时支持更多客户端。

## 支持多任务处理的常见类型

许多常见类型都具有异步方法，你可以等待这些方法，如下表所示：

| 类型 | 方法 |
| --- | --- |
| `DbContext<T>` | `AddAsync`, `AddRangeAsync`, `FindAsync`, 和 `SaveChangesAsync` |
| `DbSet<T>` | `AddAsync`, `AddRangeAsync`, `ForEachAsync`, `SumAsync`, `ToListAsync`, `ToDictionaryAsync`, `AverageAsync`, 和 `CountAsync` |
| `HttpClient` | `GetAsync`, `PostAsync`, `PutAsync`, `DeleteAsync`, 和 `SendAsync` |
| `StreamReader` | `ReadAsync`, `ReadLineAsync`, 和 `ReadToEndAsync` |
| `StreamWriter` | `WriteAsync`, `WriteLineAsync`, 和 `FlushAsync` |

**良好实践**：每当看到以`Async`为后缀的方法时，检查它是否返回`Task`或`Task<T>`。如果是，那么你可以使用它代替同步的非`Async`后缀方法。记得使用`await`调用它，并为你的方法添加`async`修饰符。

## 在 catch 块中使用 await

在 C# 5 中首次引入`async`和`await`时，只能在`try`块中使用`await`关键字，而不能在`catch`块中使用。在 C# 6 及更高版本中，现在可以在`try`和`catch`块中都使用`await`。

## 处理异步流

随着.NET Core 3.0 的推出，微软引入了流异步处理。

你可以在以下链接完成关于异步流的教程：[`docs.microsoft.com/en-us/dotnet/csharp/tutorials/generate-consume-asynchronous-stream`](https://docs.microsoft.com/en-us/dotnet/csharp/tutorials/generate-consume-asynchronous-stream)

在 C# 8.0 和.NET Core 3.0 之前，`await`关键字仅适用于返回标量值的任务。.NET Standard 2.1 中的异步流支持允许`async`方法返回一系列值。

让我们看一个模拟示例，该示例返回三个随机整数作为异步流。

1.  使用你偏好的代码编辑器，在`Chapter12`解决方案/工作区中添加一个名为`AsyncEnumerable`的新控制台应用。

1.  在 Visual Studio Code 中，选择`AsyncEnumerable`作为活动的 OmniSharp 项目。

1.  在`Program.cs`中，删除现有语句并静态导入`Console`，如下面的代码所示：

    ```cs
    using static System.Console; // WriteLine() 
    ```

1.  在`Program.cs`底部，创建一个使用`yield`关键字异步返回三个随机数字序列的方法，如下面的代码所示：

    ```cs
    async static IAsyncEnumerable<int> GetNumbersAsync()
    {
      Random r = new();
      // simulate work
      await Task.Delay(r.Next(1500, 3000));
      yield return r.Next(0, 1001);
      await Task.Delay(r.Next(1500, 3000));
      yield return r.Next(0, 1001);
      await Task.Delay(r.Next(1500, 3000));
      yield return r.Next(0, 1001);
    } 
    ```

1.  在`GetNumbersAsync`上方，添加语句以枚举数字序列，如下面的代码所示：

    ```cs
    await foreach (int number in GetNumbersAsync())
    {
      WriteLine($"Number: {number}");
    } 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Number: 509
    Number: 813
    Number: 307 
    ```

# 实践与探索

通过回答一些问题，进行实践操作，并深入研究本章主题，来测试你的知识和理解。

## 练习 12.1 – 测试你的知识

回答以下问题：

1.  关于进程，你能了解到哪些信息？

1.  `Stopwatch`类的精确度如何？

1.  按照惯例，返回`Task`或`Task<T>`的方法应附加什么后缀？

1.  要在方法内部使用`await`关键字，方法声明必须应用什么关键字？

1.  如何创建子任务？

1.  为什么要避免使用`lock`关键字？

1.  何时应使用`Interlocked`类？

1.  何时应使用`Mutex`类而不是`Monitor`类？

1.  在网站或网络服务中使用`async`和`await`有何好处？

1.  你能取消一个任务吗？如果可以，如何操作？

## 练习 12.2 – 探索主题

请使用以下网页上的链接，以了解更多关于本章所涵盖主题的详细信息：

[第十二章 - 使用多任务提高性能和可扩展性](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-12---improving-performance-and-scalability-using-multitasking)

# 总结

在本章中，你不仅学会了如何定义和启动任务，还学会了如何等待一个或多个任务完成，以及如何控制任务完成的顺序。你还学习了如何同步访问共享资源以及`async`和`await`背后的奥秘。

在接下来的七章中，你将学习如何为.NET 支持的**应用模型**，即**工作负载**，创建应用程序，例如网站和服务，以及跨平台的桌面和移动应用。
