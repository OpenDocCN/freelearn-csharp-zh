# 第十二章：12

# 使用多任务处理改进性能和可伸缩性

本章是关于允许多个操作同时发生，以改进您构建的应用程序的性能、可伸缩性和用户生产力。

在本章中，我们将涵盖以下主题：

+   了解进程、线程和任务

+   监视性能和资源使用

+   异步运行任务

+   同步访问共享资源

+   了解`async`和`await`

# 了解进程、线程和任务

一个进程，例如我们创建的每个控制台应用程序，都分配了内存和线程等资源。

线程逐条执行您的代码。默认情况下，每个进程只有一个线程，当我们需要同时执行多个任务时，这可能会导致问题。线程还负责跟踪当前经过身份验证的用户以及应遵循的当前语言和地区的任何国际化规则。

Windows 和大多数其他现代操作系统使用抢占式多任务处理，模拟任务的并行执行。它将处理器时间分配给线程，依次为每个线程分配一个时间片。当前线程的时间片结束时，该线程被挂起。然后处理器允许另一个线程运行一段时间。

当 Windows 从一个线程切换到另一个线程时，它会保存线程的上下文，并重新加载线程队列中下一个线程的先前保存的上下文。这需要时间和资源来完成。

作为开发人员，如果您有少量复杂的工作，并且希望完全控制它们，那么您可以创建和管理单独的`Thread`实例。如果您有一个主线程和多个可以在后台执行的小任务，那么您可以使用`ThreadPool`类将指向作为方法实现的这些小任务的委托实例添加到队列中，它们将自动分配给线程池中的线程。

在本章中，我们将使用`Task`类型以更高的抽象级别管理线程。

线程可能必须竞争并等待访问共享资源，例如变量、文件和数据库对象。有一些类型可以管理这一点，您将在本章后面看到它们的作用。

根据任务的不同，将线程（工作者）数量加倍以执行任务并不会减少完成该任务所需的秒数。事实上，这可能会增加任务的持续时间。

良好实践：永远不要假设更多的线程会提高性能！在没有多个线程的基线代码实现上运行性能测试，然后再在具有多个线程的代码实现上运行性能测试。您还应该在尽可能接近生产环境的暂存环境中进行性能测试。

# 监视性能和资源使用

在改进任何代码的性能之前，我们需要能够监视其速度和效率，以记录我们可以根据其进行改进的基线。

## 评估类型的效率

对于特定情景，使用哪种类型最好？要回答这个问题，我们需要仔细考虑“最好”的含义，并通过这一点，我们应该考虑以下因素：

+   功能：这可以通过检查类型是否提供所需的功能来决定。

+   内存大小：这可以通过类型占用的内存字节数来决定。

+   性能：这可以通过类型的速度来决定。

+   未来需求：这取决于需求变化和可维护性。

在存储数字等多种类型具有相同功能的情况下，我们需要考虑内存和性能来做出选择。

如果我们需要存储数百万个数字，那么最好使用需要最少字节内存的类型。但如果我们只需要存储少量数字，但需要对它们进行大量计算，那么最好使用在特定 CPU 上运行最快的类型。

您已经看到了 `sizeof()` 函数的使用，它显示了类型的单个实例在内存中使用的字节数。当我们在更复杂的数据结构中存储大量值时，例如数组和列表，那么我们需要更好的方法来测量内存使用情况。

您可以在网上和书籍中阅读很多建议，但要确定最适合您的代码的最佳类型的唯一方法是自己比较这些类型。

在下一节中，您将学习如何编写代码来监视使用不同类型时的实际内存需求和性能。

今天，`short` 变量可能是最佳选择，但使用 `int` 变量可能是更好的选择，即使它在内存中占用的空间是 `short` 的两倍。这是因为我们可能需要将来存储更广泛范围的值。

有一个重要的开发者经常忽视的指标：维护。这是另一个程序员需要付出多少努力来理解和修改您的代码的度量。如果您选择了一个不明显的类型而没有用有帮助的注释来解释这个选择，那么可能会让后来的程序员困惑，他们需要修复错误或添加功能。

## 使用诊断监视性能和内存

`System.Diagnostics` 命名空间有很多有用的类型来监视您的代码。我们将首先看一下有用的 `Stopwatch` 类型：

1.  使用您喜欢的编码工具创建一个名为 `Chapter12` 的新工作空间/解决方案。

1.  添加一个类库项目，如下列表所定义：

1.  项目模板：**类库** / `classlib`

1.  工作空间/解决方案文件和文件夹：`Chapter12`

1.  项目文件和文件夹：`MonitoringLib`

1.  添加一个控制台应用程序项目，如下列表所定义：

1.  项目模板：**控制台应用程序** / `console`

1.  工作空间/解决方案文件和文件夹：`Chapter12`

1.  项目文件和文件夹：`MonitoringApp`

1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择。

1.  在 Visual Studio Code 中，选择 `MonitoringApp` 作为活动的 OmniSharp 项目。

1.  在 `MonitoringLib` 项目中，将 `Class1.cs` 文件重命名为 `Recorder.cs` 。

1.  在 `MonitoringApp` 项目中，添加对 `MonitoringLib` 类库的项目引用，如下标记所示：

```cs

<ItemGroup>

<ProjectReference

Include="..\MonitoringLib\MonitoringLib.csproj"

/>

</ItemGroup>

```

1.  构建 `MonitoringApp` 项目。

### `Stopwatch` 和 `Process` 类型的有用成员

`Stopwatch` 类型有一些有用的成员，如下表所示：

| 成员 | 描述 |
| --- | --- |
| `Restart` 方法 | 这将重置经过的时间为零，然后启动计时器。 |
| `Stop` 方法 | 这将停止计时器。 |
| `Elapsed` 属性 | 这是以 `TimeSpan` 格式（例如，小时：分钟：秒）存储的经过时间。 |
| `ElapsedMilliseconds` 属性 | 这是以 `Int64` 值存储的毫秒为单位的经过时间。 |

`Process` 类型有一些有用的成员，如下表所示：

| 成员 | 描述 |
| --- | --- |
| `VirtualMemorySize64` | 这显示为进程分配的虚拟内存量（以字节为单位）。 |
| `WorkingSet64` | 这显示为进程分配的物理内存量（以字节为单位）。 |

### 实现一个 Recorder 类

我们将创建一个 `Recorder` 类，使得监视时间和内存资源使用变得容易。为了实现我们的 `Recorder` 类，我们将使用 `Stopwatch` 和 `Process` 类：

1.  在 `Recorder.cs` 中，更改其内容以使用 `Stopwatch` 实例记录时间和当前 `Process` 实例记录内存使用情况，如下面的代码所示：

```cs

使用

System.Diagnostics; // Stopwatch

使用

静态

System.Console;

使用

静态

System.Diagnostics.Process; // GetCurrentProcess()

命名空间

Packt.Shared

;

公共的

静态

类

Recorder

{

私人的

静态

Stopwatch timer =新的

();

私人的

静态

长

bytesPhysicalBefore = 0

;

私人的

静态

长

bytesVirtualBefore = 0

;

公共的

静态

void

开始

()

{

// 强制进行两次垃圾收集以释放内存

// 不再被引用，但尚未被释放

GC.Collect();

GC.WaitForPendingFinalizers();

GC.Collect();

// 存储当前的物理和虚拟内存使用情况

bytesPhysicalBefore = GetCurrentProcess().WorkingSet64;

bytesVirtualBefore = GetCurrentProcess().VirtualMemorySize64;

timer.Restart();

}

公共的

静态

void

停止

()

{

timer.Stop();

长

bytesPhysicalAfter =

GetCurrentProcess().WorkingSet64;

长

bytesVirtualAfter =

GetCurrentProcess().VirtualMemorySize64;

WriteLine("{0:N0}物理字节已使用。"

，

bytesPhysicalAfter - bytesPhysicalBefore);

WriteLine("{0:N0}虚拟字节已使用。"

,

bytesVirtualAfter - bytesVirtualBefore);

WriteLine("{0}时间跨度已过。"

, timer.Elapsed);

WriteLine("{0:N0}总毫秒已过。"

，

计时器.ElapsedMilliseconds);

}

}

```

`Recorder`类的`Start`方法使用`GC`类型（垃圾收集器）来确保在记录已使用的内存量之前收集当前分配但未引用的内存。这是一种高级技术，您几乎永远不应该在应用程序代码中使用。

1.  在`Program.cs`中，编写语句以启动和停止`Recorder`，同时生成一个包含 10,000 个整数的数组，如下面的代码所示：

```cs

使用

Packt.Shared; // Recorder

使用

静态

System.Console;

WriteLine("处理中。请稍候..."

);

Recorder.Start();

// 模拟需要一些内存资源的过程...

int

[] largeArrayOfInts = Enumerable.Range(

开始：1

，计数：10

_000).ToArray();

// ...并且需要一些时间来完成

Thread.Sleep(new

Random().Next(5

, 10

) * 1000

);

Recorder.Stop();

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

处理中。请稍候...

655,360 物理字节已使用。

使用了 536,576 虚拟字节。

00:00:09.0038702 时间跨度已过。

9,003 毫秒总时间已过。

```

请记住，经过的时间在 5 到 10 秒之间是随机的。您的结果会有所不同。例如，在我的 Mac mini M1 上运行时，使用的物理内存较少，但虚拟内存较多，如下面的输出所示：

```cs

处理中。请稍候...

294,912 物理字节已使用。

10,485,760 虚拟字节已使用。

00:00:06.0074221 时间跨度已过。

6,007 毫秒总时间已过。

```

## 衡量处理字符串的效率

现在您已经看到了`Stopwatch`和`Process`类型如何用于监视您的代码，我们将使用它们来评估处理`string`变量的最佳方法。

1.  在`Program.cs`中，通过使用多行注释字符`/* */`将先前的语句注释掉。

1.  编写语句以创建一个包含 50,000 个`int`变量的数组，然后使用`string`和`StringBuilder`类将它们连接起来，分隔符为逗号，如下面的代码所示：

```cs

int

[] numbers = Enumerable.Range(

开始：1

，计数：50

_000).ToArray();

WriteLine("使用带有+的字符串"

);

Recorder.Start();

字符串

s =字符串

.Empty; // 即""

为

(int

i = 0

; i < numbers.Length; i++)

{

s += numbers[i] + ", "

;

}

Recorder.Stop();

WriteLine("使用 StringBuilder"

);

Recorder.Start();

System.Text.StringBuilder builder =新的

();

for

(int

i = 0

; i < numbers.Length; i++)

{

builder.Append(numbers[i]);

builder.Append(", "

);

}

Recorder.Stop();

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

使用带有+

14,883,072 物理字节已使用。

3,609,728 虚拟字节已使用。

00:00:01.6220879 时间跨度已过。

1,622 毫秒总时间已过。

使用 StringBuilder

12,288 物理字节已使用。

0 虚拟字节已使用。

00:00:00.0006038 时间跨度已过。

0 毫秒总时间已过。

```

我们可以总结结果如下：

+   `string`类与`+`运算符使用了约 14 MB 的物理内存，1.5 MB 的虚拟内存，并花费了 1.5 秒。

+   `StringBuilder`类使用了 12 KB 的物理内存，零虚拟内存，并且花费不到 1 毫秒。

在这种情况下，`StringBuilder`连接文本时比`string`快 1000 多倍，内存效率约为 10000 倍！这是因为`string`连接每次使用时都会创建一个新的`string`，因为`string`值是不可变的，所以它们可以安全地用于重用。`StringBuilder`在追加更多字符时创建一个单一的缓冲区。

**良好实践**：避免在循环内使用`String.Concat`方法或`+`运算符。改用`StringBuilder`。

现在您已经学会了如何使用.NET 内置的类型来测量代码的性能和资源效率，让我们了解一个提供更复杂性能测量的 NuGet 包。

## 使用 Benchmark.NET 监视性能和内存

.NET 有一个流行的基准测试 NuGet 包，微软在其关于性能改进的博客文章中使用它，因此对于.NET 开发人员来说，了解它的工作原理并将其用于自己的性能测试是很好的。让我们看看如何使用它来比较`string`连接和`StringBuilder`之间的性能：

1.  使用您喜欢的代码编辑器将一个新的控制台应用程序添加到名为`Benchmarking`的`Chapter12`解决方案/工作区中。

1.  在 Visual Studio Code 中，选择`Benchmarking`作为活动的 OmniSharp 项目。

1.  添加对 Benchmark.NET 的包引用，记住您可以查找最新版本并使用该版本，而不是我使用的版本，如下面的标记所示：

```cs

<ItemGroup>

<PackageReference Include="BenchmarkDotNet"

Version="0.13.1"

/>

</ItemGroup>

```

1.  构建项目以恢复包。

1.  在`Program.cs`中，删除现有的语句，然后导入运行基准测试的命名空间，如下面的代码所示：

```cs

using

BenchmarkDotNet.Running;

```

1.  添加一个名为`StringBenchmarks.cs`的新类文件。

1.  在`StringBenchmarks.cs`中，添加语句来定义一个类，其中包含您想要运行的每个基准测试的方法，本例中，两种方法都使用`string`连接或`StringBuilder`连接二十个逗号分隔的数字，如下面的代码所示：

```cs

使用

BenchmarkDotNet.Attributes; // [Benchmark]

public

class

StringBenchmarks

{

int

[] numbers;

public

StringBenchmarks

()

{

numbers = Enumerable.Range(

start: 1

，计数：20

).ToArray();

}

[Benchmark(Baseline = true)

]

public

string

StringConcatenationTest

()

{

string

s = string

.Empty; // 例如""

for

(int

i = 0

; i < numbers.Length; i++)

{

s += numbers[i] + ", "

;

}

return

s;

}

[Benchmark

]

public

string

StringBuilderTest

()

{

System.Text.StringBuilder builder = new

();

for

(int

i = 0

; i < numbers.Length; i++)

{

builder.Append(numbers[i]);

builder.Append(", "

）;

}

return

builder.ToString();

}

}

```

1.  在`Program.cs`中，添加一个语句来运行基准测试，如下面的代码所示：

```cs

BenchmarkRunner.Run<StringBenchmarks>();

```

1.  在 Visual Studio 2022 中，在工具栏中，将**解决方案配置**设置为**Release**。

1.  在 Visual Studio Code 中，在终端中使用`dotnet run --configuration Release`命令。

1.  运行控制台应用程序并注意结果，包括一些报告文件等工件，最重要的是，一个总结表，显示`string`连接的平均时间为 412.990 ns，`StringBuilder`的平均时间为 275.082 ns，如下面的部分输出和*图 12.1*所示：

```cs

// ***** BenchmarkRunner: Finish  *****

// *导出*

BenchmarkDotNet.Artifacts\results\StringBenchmarks-report.csv

BenchmarkDotNet.Artifacts\results\StringBenchmarks-report-github.md

BenchmarkDotNet.Artifacts\results\StringBenchmarks-report.html

// *详细结果*

StringBenchmarks.StringConcatenationTest: DefaultJob

Runtime = .NET 6.0.0 (6.0.21.37719), X64 RyuJIT; GC = Concurrent Workstation

平均值= 412.990 ns，StdErr= 2.353 ns (0.57%)，N= 46，StdDev= 15.957 ns

最小值= 373.636 ns，Q1= 413.341 ns，中位数= 417.665 ns，Q3= 420.775 ns，最大值= 434.504 ns

IQR= 7.433 ns，LowerFence= 402.191 ns，UpperFence= 431.925 ns

置信区间= [404.708 ns; 421.273 ns] (CI 99.9%)，边际= 8.282 ns (平均值的 2.01%)

偏度= -1.51，峰度= 4.09，M 值= 2

-------------------- 直方图 --------------------

[370.520 ns ; 382.211 ns) | @@@@@@

[382.211 ns ; 394.583 ns) | @

[394.583 ns ; 411.300 ns) | @@

[411.300 ns ; 422.990 ns) | @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

[422.990 ns ; 436.095 ns) | @@@@@

---------------------------------------------------

StringBenchmarks.StringBuilderTest：默认作业

Runtime = .NET 6.0.0 (6.0.21.37719)，X64 RyuJIT；GC = Concurrent Workstation

平均值= 275.082 ns，StdErr= 0.558 ns (0.20%)，N= 15，StdDev= 2.163 ns

最小值= 271.059 ns，Q1= 274.495 ns，中位数= 275.403 ns，Q3= 276.553 ns，最大值= 278.030 ns

IQR= 2.058 ns，LowerFence= 271.409 ns，UpperFence= 279.639 ns

置信区间= [272.770 ns; 277.394 ns] (CI 99.9%)，边际= 2.312 ns (平均值的 0.84%)

偏度= -0.69，峰度= 2.2，M 值= 2

-------------------- 直方图 --------------------

[269.908 ns ; 278.682 ns) | @@@@@@@@@@@@@@@

---------------------------------------------------

// * 摘要 *

BenchmarkDotNet=v0.13.1，OS=Windows 10.0.19043.1165 (21H1/May2021Update)

第 11 代英特尔酷睿 i7-1165G7 2.80GHz，1 个 CPU，8 个逻辑和 4 个物理内核

.NET SDK=6.0.100

[主机]：.NET 6.0.0 (6.0.21.37719)，X64 RyuJIT

DefaultJob：.NET 6.0.0 (6.0.21.37719)，X64 RyuJIT

| 方法 | 平均值 | 错误 | 标准偏差 | 比率 | RatioSD |
| --- | --- | --- | --- | --- | --- |

|------------------------ |---------:|--------:|---------:|------:|--------:|

| StringConcatenationTest | 413.0 ns | 8.28 ns | 15.96 ns |  1.00 |    0.00 |
| --- | --- | --- | --- | --- | --- |
|       StringBuilderTest | 275.1 ns | 2.31 ns |  2.16 ns |  0.69 |    0.04 |

// * 提示 *

异常值

StringBenchmarks.StringConcatenationTest：默认->删除了 7 个异常值，检测到 14 个异常值(376.78 ns..391.88 ns，440.79 ns..506.41 ns)

StringBenchmarks.StringBuilderTest：默认->检测到 2 个异常值(274.68 ns，274.69 ns)

// * 图例 *

平均值：所有测量的算术平均值

错误：99.9%置信区间的一半

StdDev：所有测量的标准偏差

比率：比率分布的平均值([当前]/[基线])

RatioSD：比率分布的标准偏差([当前]/[基线])

1 ns：1 纳秒(0.000000001 秒)

// ***** BenchmarkRunner: End *****

// ** 剩余 0 个基准要运行 **

运行时间：00:01:13 (73.35 秒)，执行基准测试：2

全局总时间：00:01:29 (89.71 秒)，执行基准测试：2

// * 清理工件 *

```

![](img/Image00099.jpg)

图 12.1：摘要表，显示 StringBuilder 所占时间为字符串连接的 69%

`异常值`部分特别有趣，因为它显示了`string`连接不仅比`StringBuilder`慢，而且在花费的时间上也更不一致。当然，您的结果会有所不同。

您现在已经看到了衡量性能的两种方法。现在让我们看看如何异步运行任务，以潜在地提高性能。

# 异步运行任务

为了理解如何同时运行多个任务(同时)，我们将创建一个需要执行三种方法的控制台应用程序。

需要执行三种方法：第一种需要 3 秒，第二种需要 2 秒，第三种需要 1 秒。为了模拟这项工作，我们可以使用`Thread`类告诉当前线程休眠指定的毫秒数。

## 同步运行多个操作

在我们使任务同时运行之前，我们将以同步方式运行它们，也就是一个接一个地运行。

1.  使用您喜欢的代码编辑器向`Chapter12`解决方案/工作区添加一个名为`WorkingWithTasks`的新控制台应用程序。

1.  在 Visual Studio Code 中，将`WorkingWithTasks`选择为活动的 OmniSharp 项目。

1.  在`Program.cs`中，导入用于使用秒表的命名空间（用于处理线程和任务的命名空间会被隐式导入），并静态导入`Console`，如下面的代码所示：

```cs

使用

System.Diagnostics; // Stopwatch

使用

静态

System.Console;

```

1.  在`Program.cs`的底部，创建一个方法来输出有关当前线程的信息，如下面的代码所示：

```cs

静态

void

OutputThreadInfo

()

{

Thread t = Thread.CurrentThread;

WriteLine(

"线程 Id：{0}，优先级：{1}，后台：{2}，名称：{3}"

，

t.ManagedThreadId, t.Priority,

t.IsBackground, t.Name ?? "null"

);

}

```

1.  在`Program.cs`的底部，添加三个模拟工作的方法，如下面的代码所示：

```cs

静态

void

MethodA

()

{

WriteLine("开始方法 A..."

);

OutputThreadInfo();

Thread.Sleep(3000

); // 模拟三秒的工作

WriteLine("完成方法 A。"

);

}

静态

void

MethodB

()

{

WriteLine("开始方法 B..."

);

OutputThreadInfo();

Thread.Sleep(2000

); // 模拟两秒的工作

WriteLine("完成方法 B。"

);

}

静态

void

MethodC

()

{

WriteLine("开始方法 C..."

);

OutputThreadInfo();

Thread.Sleep(1000

); // 模拟一秒的工作

WriteLine("完成方法 C。"

);

}

```

1.  在`Program.cs`的顶部，添加语句来调用输出有关线程的信息的方法，定义并启动秒表，调用三个模拟工作方法，然后输出经过的毫秒数，如下面的代码所示：

```cs

OutputThreadInfo();

Stopwatch timer = Stopwatch.StartNew();

WriteLine("在一个线程上同步运行方法。"

);

MethodA();

MethodB();

MethodC();

WriteLine($"

{timer.ElapsedMilliseconds:#,##

0

}

毫秒。"

);

```

1.  运行代码，查看结果，并注意当只有一个未命名的前台线程在工作时，所需的总时间略长于 6 秒，如下面的输出所示：

```cs

线程 Id：1，优先级：普通，后台：假，名称：null

在一个线程上同步运行方法。

开始方法 A...

线程 Id：1，优先级：普通，后台：假，名称：null

完成方法 A。

开始方法 B...

线程 Id：1，优先级：普通，后台：假，名称：null

完成方法 B。

开始方法 C...

线程 Id：1，优先级：普通，后台：假，名称：null

完成方法 C。

经过 6,017 毫秒。

```

## 使用任务异步运行多个操作

`Thread`类自.NET 的第一个版本就可用，可用于创建新线程并管理它们，但直接使用可能会有些棘手。

.NET Framework 4.0 在 2010 年引入了`Task`类，它是对线程的包装，使得创建和管理更加容易。管理在任务中包装的多个线程将允许我们的代码同时执行，也就是异步执行。

每个`Task`都有一个`Status`属性和一个`CreationOptions`属性。`Task`有一个`ContinueWith`方法，可以使用`TaskContinuationOptions`枚举进行自定义，并可以使用`TaskFactory`类进行管理。

### 开始任务

我们将看到三种使用`Task`实例启动方法的方式。GitHub 存储库中有链接到讨论优缺点的文章。每种语法略有不同，但它们都定义了一个`Task`并启动了它：

1.  注释掉对三个方法及其相关控制台消息的调用，并添加语句来创建和启动三个任务，每个方法一个，如下面代码中所示：

```cs

OutputThreadInfo();

Stopwatch timer = Stopwatch.StartNew();

**/***

WriteLine("在一个线程上同步运行方法。");

MethodA();

MethodB();

MethodC();

***/**

**WriteLine(**

**"在多个线程上异步运行方法。"**

**);**

**Task taskA =**

新

**(MethodA);**

**taskA.Start();**

**Task taskB = Task.Factory.StartNew(MethodB);**

**Task taskC = Task.Run(MethodC);**

WriteLine($"

{timer.ElapsedMilliseconds:#,##

0

}

毫秒已过。"

);

```

1.  运行代码，查看结果，并注意经过的毫秒几乎立即出现。这是因为现在每个方法都是由从线程池分配的三个新后台工作线程执行的，如下面的输出所示：

```cs

线程 Id: 1，优先级: 正常，后台: False，名称: null

在多个线程上异步运行方法。

开始方法 A...

线程 Id: 4，优先级: 正常，后台: True，名称: .NET ThreadPool Worker

开始方法 C...

线程 Id: 7，优先级: 正常，后台: True，名称: .NET ThreadPool Worker

开始方法 B...

线程 Id: 6，优先级: 正常，后台: True，名称: .NET ThreadPool Worker

6ms 经过。

```

甚至可能控制台应用程序在一个或多个任务有机会开始并写入控制台之前结束！

## 等待任务

有时，您需要等待任务完成后才能继续。为此，您可以在`Task`实例上使用`Wait`方法，或者在任务数组上使用`WaitAll`或`WaitAny`静态方法，如下表所述：

| 方法 | 描述 |
| --- | --- |
| `t.Wait()` | 这等待任务实例` t`完成执行。 |
| `Task.WaitAny(Task[])` | 这等待数组中的任何任务完成执行。 |
| `Task.WaitAll(Task[])` | 这等待数组中的所有任务完成执行。 |

### 使用任务的等待方法

让我们看看如何使用这些等待方法来解决控制台应用程序的问题。

1.  在`Program.cs`中，在创建三个任务之后并在输出经过的时间之前，添加语句将对三个任务的引用组合成数组，并将它们传递给`WaitAll`方法，如下面的代码所示：

```cs

任务[]任务= {任务 A，任务 B，任务 C};

Task.WaitAll(tasks);

```

1.  运行代码并查看结果，并注意原始线程将在调用`WaitAll`时暂停，等待所有三个任务完成后再输出经过的时间，如下面的输出所示：超过 3 秒。

```cs

Id: 1, Priority: Normal, Background: False, Name: null

在多个线程上异步运行方法。

开始方法 A...

Id: 6，优先级: 正常，后台: True，名称: .NET ThreadPool Worker

开始方法 B...

Id: 7, Priority: Normal, Background: True, Name: .NET ThreadPool Worker

开始方法 C...

Id: 4，优先级: 正常，后台: True，名称: .NET ThreadPool Worker

完成方法 C。

完成方法 B。

完成方法 A。

3,013ms 经过。

```

三个新线程同时执行它们的代码，并且它们可以以任何顺序开始。`MethodC`应该首先完成，因为它只需要 1 秒，然后是需要 2 秒的`MethodB`，最后是需要 3 秒的`MethodA`。

然而，实际使用的 CPU 对结果有很大影响。CPU 分配时间片给每个进程，以允许它们执行它们的线程。您无法控制方法何时运行。

## 继续进行另一个任务

如果三个任务都可以同时执行，那么等待所有任务完成将是我们需要做的。然而，通常一个任务依赖于另一个任务的输出。为了处理这种情况，我们需要定义**继续任务**。

我们将创建一些方法来模拟调用返回货币金额的网络服务，然后需要使用该金额来检索数据库中超过该金额的产品数量。第一个方法返回的结果需要输入到第二个方法中。这一次，我们将使用`Random`类来等待随机的时间间隔，每个方法调用的时间间隔在 2 到 4 秒之间，以模拟工作。

1.  在`Program.cs`的底部，添加两个模拟调用网络服务和数据库存储过程的方法，如下面的代码所示：

```cs

静态的

小数

CallWebService

()

{

WriteLine("开始调用 web 服务..."

);

OutputThreadInfo();

Thread.Sleep((new

随机()).下一个(2000

，4000

));

WriteLine("完成调用 web 服务。"

);

返回

89.99

M;

}

静态

字符串

CallStoredProcedure

(

十进制

金额

)

{

WriteLine("开始调用存储过程..."

);

OutputThreadInfo();

Thread.Sleep((new

Random()).Next(2000

，4000

));

WriteLine("完成对存储过程的调用。"

);

返回

$"12 个产品的成本超过

{amount:C}

。"

;

}

```

1.  通过将它们包装在多行注释字符`/* */`中注释掉对前三个任务的调用。保留输出经过的毫秒数的语句。

1.  在现有语句之前添加语句以输出总时间，如下所示：

```cs

WriteLine("将一个任务的结果作为另一个任务的输入传递。"

);

任务<string

> taskServiceThenSProc = Task.Factory

.StartNew(CallWebService) // 返回任务<十进制>

.ContinueWith(previousTask => // 返回任务<string>

CallStoredProcedure(previousTask.Result));

WriteLine($"结果：

{taskServiceThenSProc.Result}

"

);

```

1.  运行代码并查看结果，如下所示：

```cs

线程 ID：1，优先级：正常，后台：假，名称：null

将一个任务的结果作为另一个任务的输入传递。

开始调用 web 服务...

线程 ID：4，优先级：正常，后台：真，名称：.NET 线程池工作程序

完成调用 web 服务。

开始调用存储过程...

线程 ID：6，优先级：正常，后台：真，名称：.NET 线程池工作程序

完成调用存储过程。

结果：12 个产品的成本超过 89.99 英镑。

5,463 毫秒已过。

```

您可能会看到不同的线程运行 web 服务和存储过程调用，如上面的输出中所示（线程 4 和 6），或者同一个线程可能会被重用，因为它不再忙碌。

## 嵌套和子任务

除了定义任务之间的依赖关系，您还可以定义嵌套和子任务。**嵌套任务**是在另一个任务内创建的任务。**子任务**是必须在其父任务允许完成之前完成的嵌套任务。

让我们探讨这些类型的任务如何工作：

1.  使用您喜欢的代码编辑器向`Chapter12`解决方案/工作区添加一个名为`NestedAndChildTasks`的新控制台应用程序。

1.  在 Visual Studio Code 中，选择`NestedAndChildTasks`作为活动的 OmniSharp 项目。

1.  在`Program.cs`中，删除现有语句，静态导入`Console`，然后添加两个方法，其中一个启动一个任务来运行另一个，如下所示：

```cs

静态

空

OuterMethod

()

{

WriteLine("外部方法开始..."

);

任务内部任务=任务工厂.StartNew(InnerMethod);

WriteLine("外部方法完成。"

);

}

静态

空

InnerMethod

()

{

WriteLine("内部方法开始..."

);

线程休眠(2000

);

WriteLine("内部方法完成。"

);

}

```

1.  在这些方法之上，添加语句以启动一个任务来运行外部方法，并在停止之前等待它完成，如下所示：

```cs

任务外部任务=任务工厂.StartNew(OuterMethod);

outerTask.Wait();

WriteLine("控制台应用程序正在停止。"

);

```

1.  运行代码并查看结果，如下所示：

```cs

外部方法开始...

内部方法开始...```

外部方法完成。

控制台应用程序正在停止。

```

请注意，尽管我们等待外部任务完成，但它的内部任务不必也完成。实际上，外部任务可能会完成，控制台应用程序甚至在内部任务开始之前就结束了！

要将这些嵌套任务链接为父任务和子任务，我们必须使用特殊选项。

1.  修改定义内部任务的现有代码，以添加`TaskCreationOption`值`AttachedToParent`，如下所示：

```cs

任务内部任务=任务工厂.StartNew(InnerMethod，

**TaskCreationOptions.AttachedToParent**

);

```

1.  运行代码，查看结果，并注意内部任务必须在外部任务之前完成，如下所示：

```cs

外部方法开始...

内部方法开始...

外部方法完成。

内部方法完成。

控制台应用程序正在停止。

```

`OuterMethod` 可以在 `InnerMethod` 之前完成，如其写入控制台所示，但它的任务必须等待，如控制台不会停止直到外部和内部任务都完成所示。

## 将任务包装在其他对象周围

有时您可能有一个希望是异步的方法，但要返回的结果本身不是一个任务。您可以使用下表中所示的方法之一将返回值包装在成功完成的任务中，返回一个异常，或者指示任务被取消。

| 方法 | 描述 |
| --- | --- |
| `FromResult<TResult>(TResult)` | 创建一个 `Task<TResult>` 对象，其 `Result` 属性是非任务结果，`Status` 属性是 `RanToCompletion` 。 |
| `FromException<TResult>(Exception)` | 使用指定的异常创建一个已完成的 `Task<TResult>` 。 |
| `FromCanceled<TResult>(CancellationToken)` | 使用指定的取消令牌创建一个由于取消而完成的 `Task<TResult>` 。 |

当您需要时，这些方法非常有用：

+   实现一个具有异步方法的接口，但您的实现是同步的。这在网站和服务中很常见。

+   在单元测试期间模拟异步实现。

在 *第七章*，*打包和分发.NET 类型* 中，我们创建了一个包含用于检查有效 XML、密码和十六进制代码的函数的类库。

如果我们想要使这些方法符合要求返回 `Task<T>` 的接口，我们可以使用下面代码中所示的这些有用的方法：

```cs

using

System.Text.RegularExpressions;

namespace

Packt.Shared

;

public

static

class

StringExtensions

{

public

static

Task<

bool

IsValidXmlTagAsync

(

this

string

input

)

{

if

(input == null

)

{

return

Task.FromException<bool

new

ArgumentNullException("缺少输入参数"

));

}

if

(input.Length == 0

)

{

return

Task.FromException<bool

new

ArgumentException("输入参数为空。"

));

}

return

Task.FromResult(Regex.IsMatch(input,

@"^<([a-z]+)([^<]+)*(?:>(.*)<\/\1>|\s+\/>)$"

));

}

// 其他方法

}

```

如果您需要实现的方法返回一个 `Task`（相当于同步方法中的 `void`），那么您可以返回一个预定义的已完成的 `Task` 对象，如下面的代码所示：

```cs

public

Task

DeleteCustomerAsync

()

{

// ...

return

Task.CompletedTask;

}

```

# 同步访问共享资源

当多个线程同时执行时，有可能两个或更多的线程同时访问同一个变量或其他资源，从而可能导致问题。因此，您应该仔细考虑如何使您的代码**线程安全**。

实现线程安全的最简单机制是使用一个对象变量作为标志或交通灯，指示共享资源何时应用了独占锁。

在威廉·戈尔丁的《蝇王》中，皮基和拉尔夫发现了一只海螺壳，并用它召开会议。男孩们对自己实施了“海螺规则”，决定除非他们拿着海螺，否则谁也不能说话。

我喜欢给用于实现线程安全代码的对象变量命名为“conch”。当一个线程拥有了 conch，其他线程就不应该访问由该 conch 表示的共享资源。请注意，我说的是*应该*。只有尊重 conch 的代码才能实现同步访问。conch 不是锁。

我们将探讨一些可以用来同步访问共享资源的类型：

+   `Monitor`：一个可以被多个线程使用的对象，用于检查它们是否应该在同一个进程中访问共享资源。

+   `Interlocked`：一个用于在 CPU 级别操作简单数值类型的对象。

## 从多个线程访问资源

1.  使用您喜欢的代码编辑器向 `Chapter12` 解决方案/工作区中添加一个名为 `SynchronizingResourceAccess` 的新控制台应用程序。

1.  在 Visual Studio Code 中，将`SynchronizingResourceAccess`选择为活动的 OmniSharp 项目。

1.  在`Program.cs`中，删除现有的语句，然后添加语句以执行以下操作：

+   导入诊断类型的命名空间，如`Stopwatch`。

+   静态地导入`Console`类型。

+   在`Program.cs`的底部，创建一个带有两个字段的静态类：

+   一个字段来生成随机等待时间。

+   一个`string`字段来存储消息（这是一个共享资源）。

+   在类的上方，创建两个静态方法，在循环中向共享的`string`添加五次字母 A 或 B，并等待每次迭代的随机间隔最多 2 秒：

```cs

静态

void

MethodA

()

{

为

(int

i = 0

; i < 5

; i ++)

{

线程休眠(SharedObjects.Random.Next(2000

));

SharedObjects.Message += "A"

;

写("."

);

}

}

静态

void

MethodB

()

{

为

(int

i = 0

; i < 5

; i ++)

{

线程休眠(SharedObjects.Random.Next(2000

));

SharedObjects.Message += "B"

;

Write("."

);

}

}

静态

类

SharedObjects

{

public

静态

Random Random = new

();

public

静态

字符串

？消息； // 一个共享资源

}

```

1.  在命名空间导入之后，编写语句以使用一对任务在单独的线程上执行两种方法，并在输出经过的毫秒数之前等待它们完成，如下面的代码所示：

```cs

WriteLine("请等待任务完成。"

);

Stopwatch watch = Stopwatch.StartNew();

任务 a = Task.Factory.StartNew(MethodA);

任务 b = Task.Factory.StartNew(MethodB);

Task.WaitAll(new

任务[] { a, b });

WriteLine();

WriteLine($"结果：

{SharedObjects.Message}

."

);

WriteLine($"

{watch.ElapsedMilliseconds:N0}

经过的毫秒数。"

);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

请等待任务完成。

..........

结果：BABABAABBA。

经过了 5,753 毫秒。

```

这表明两个线程同时修改了消息。在实际应用中，这可能是一个问题。但是我们可以通过对海螺对象应用互斥锁并在两种方法中自愿检查海螺来防止并发访问，我们将在下一节中进行。

## 对海螺应用互斥锁

现在，让我们使用海螺确保只有一个线程一次访问共享资源。

1.  在`SharedObjects`中，声明并实例化一个`object`变量，作为一个海螺，如下面的代码所示：

```cs

public

静态

object

Conch = new

();

```

1.  在`MethodA`和`MethodB`中，为海螺周围的`for`语句添加一个`lock`语句，如下面的代码中所突出显示的那样：

```cs

**锁**

**(SharedObjects.Conch)**

**{**

为

(int

i = 0

; i < 5

; i ++)

{

线程休眠(SharedObjects.Random.Next(2000

));

SharedObjects.Message += "A"

;

Write("."

);

}

**}**

```

**良好实践**：请注意，由于检查海螺是自愿的，如果您只在两种方法中的一个中使用`lock`语句，则共享资源将继续被两种方法访问。确保所有访问共享资源的方法都尊重海螺。

1.  运行代码并查看结果，如下面的输出所示：

```cs

请等待任务完成。

..........

结果：BBBBBAAAAA。

经过了 10,345 毫秒。

```

尽管经过的时间更长，但一次只能有一个方法访问共享资源。`MethodA`或`MethodB`可以先开始。一旦一个方法完成了对共享资源的工作，那么海螺就被释放，另一个方法就有机会做它的工作。

### 理解锁定语句

当“锁”语句“锁定”对象变量时，您可能会想知道它的作用（提示：它不会锁定对象！），如下面的代码所示：

```cs

锁

(SharedObjects.Conch)

{

// 使用共享资源

}

```

C#编译器将`lock`语句更改为使用`Monitor`类*进入*和*退出*海螺对象（我喜欢把它看作*拿*和*释放*海螺对象）的`try`-`finally`语句，如下面的代码所示：

```cs

try

{

Monitor.Enter(SharedObjects.Conch);

// 使用共享资源

}

finally

{

Monitor.Exit(SharedObjects.Conch);

}

```

当线程在任何对象上调用`Monitor.Enter`时，也就是引用类型时，它会检查是否有其他线程已经拿到了海螺。如果有，线程会等待。如果没有，线程会拿到海螺并开始在共享资源上进行工作。一旦线程完成了它的工作，它会调用`Monitor.Exit`，释放海螺。如果另一个线程在等待，它现在可以拿到海螺并进行工作。这要求所有线程通过适当调用`Monitor.Enter`和`Monitor.Exit`来尊重海螺。

### 避免死锁

了解`lock`语句如何被编译器转换为`Monitor`类上的方法调用也很重要，因为使用`lock`语句可能会导致死锁。

当有两个或更多个共享资源（每个资源都有一个监视哪个线程当前正在对每个共享资源进行工作的海螺）时，死锁可能会发生，并且会发生以下事件序列：

+   Thread X "锁定"海螺 A 并开始在共享资源 A 上工作。

+   线程 Y“锁定”海螺 B 并开始在共享资源 B 上工作。

+   在仍在处理资源 A 时，线程 X 还需要处理资源 B，因此它尝试“锁定”海螺 B，但由于线程 Y 已经拿到了海螺 B 而被阻塞。

+   在仍在处理资源 B 时，线程 Y 还需要处理资源 A，因此它尝试“锁定”海螺 A，但由于线程 X 已经拿到了海螺 A 而被阻塞。

防止死锁的一种方法是在尝试获取锁时指定超时。为此，您必须手动使用`Monitor`类，而不是使用`lock`语句。

1.  修改您的代码，将`lock`语句替换为尝试使用超时进入海螺的代码，并输出错误，然后退出监视器，允许其他线程进入监视器，如下面代码中所示：

```cs

**try**

**{**

**if**

**(Monitor.TryEnter(SharedObjects.Conch, TimeSpan.FromSeconds(**

**15**

**)))**

{

for

(int

i = 0

; i < 5

; i++)

{

Thread.Sleep(SharedObjects.Random.Next(2000

));

SharedObjects.Message += "A"

;

Write("."

);

}

}

**else**

**{**

**WriteLine(**

**"在海螺上进入监视器时，方法 A 超时。"**

**);**

**}**

**}**

**finally**

**{**

**Monitor.Exit(SharedObjects.Conch);**

**}**

```

1.  运行代码并查看结果，应该返回与之前相同的结果（尽管 A 或 B 可能首先抓住海螺），但这是更好的代码，因为它将防止潜在的死锁。

良好的实践：只有在您可以编写代码以避免潜在死锁的情况下才使用`lock`关键字。如果无法避免潜在死锁，那么始终使用`Monitor.TryEnter`方法而不是`lock`，结合`try`-`finally`语句，以便您可以提供超时，并且如果发生死锁，则其中一个线程将退出。您可以在以下链接中阅读有关良好线程实践的更多信息：[`docs.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices`](https://docs.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices)

## 同步事件

在*第六章*，*实现接口和继承类*中，您学习了如何引发和处理事件。但是.NET 事件不是线程安全的，因此您应该避免在多线程场景中使用它们，并遵循我之前向您展示的标准事件引发代码。

在了解.NET 事件不是线程安全之后，一些开发人员尝试在添加和移除事件处理程序或引发事件时使用独占锁，如下面的代码所示：

```cs

// 事件委托字段

public

事件

EventHandler Shout;

// 海螺

private

object

eventLock = new

();

// 方法

public

void

Poke

()

{

lock

(eventLock) // 不好的主意

{

// 如果有东西在监听...

if

(Shout != null

)

{

// ...然后调用委托来触发事件

Shout(this

, EventArgs.Empty);

}

}

}

```

**良好的实践**：您可以在以下链接中了解有关事件和线程安全性的更多信息：[`docs.microsoft.com/en-us/archive/blogs/cburrows/field-like-events-considered-harmful`](https://docs.microsoft.com/en-us/archive/blogs/cburrows/field-like-events-considered-harmful)

但是，正如 Stephen Cleary 在以下博客文章中所解释的那样，情况很复杂：[`blog.stephencleary.com/2009/06/threadsafe-events.html`](https://blog.stephencleary.com/2009/06/threadsafe-events.html)

## 使 CPU 操作变为原子

原子来自希腊词 **atomos** ，意思是 *不可分割的*。了解多线程中哪些操作是原子的很重要，因为如果它们不是原子的，那么它们可能会在操作过程中被另一个线程中断。C#的增量运算符是原子的吗，如下面的代码所示？

```cs

int

x = 3

;

x++; // 这是一个原子 CPU 操作吗？

```

它不是原子的！增加整数需要以下三个 CPU 操作：

1.  将实例变量的值加载到寄存器中。

1.  增加值。

1.  将值存储在实例变量中。

一个线程在执行前两个步骤后可能会被中断。然后第二个线程可能执行所有三个步骤。当第一个线程恢复执行时，它将覆盖变量中的值，并且第二个线程执行的增量或减量的效果将丢失！

有一个名为`Interlocked`的类型，可以对值类型（如整数和浮点数）执行原子操作。让我们看看它的运行情况：

1.  在`SharedObjects`类中声明另一个字段，用于计算已发生的操作次数，如下面的代码所示：

```cs

public

static

int

Counter; // 另一个共享资源

```

1.  在方法 A 和 B 中，在`for`语句内和修改`string`值后，添加一个语句来安全地增加计数器，如下面的代码所示：

```cs

Interlocked.Increment(ref

SharedObjects.Counter);

```

1.  在输出经过的时间后，将计数器的当前值写入控制台，如下面的代码所示：

```cs

WriteLine($"

{SharedObjects.Counter}

字符串修改。"

);

```

1.  运行代码并查看结果，如下面的输出所示：```

```cs

请等待任务完成。

..........

结果：BBBBBAAAAA。

13,531 毫秒经过。

**10 个字符串修改。**

```

细心的读者会意识到现有的 conch 对象保护了由 conch 锁定的代码块内访问的所有共享资源，因此在这个特定的例子中实际上不需要在这里使用`Interlocked`。但是，如果我们没有已经保护像`Message`这样的另一个共享资源，那么使用`Interlocked`将是必要的。

## 应用其他类型的同步

`Monitor` 和 `Interlocked` 是相互排斥的锁，简单而有效，但有时，您需要更高级的选项来同步对共享资源的访问，如下表所示：

| 类型 | 描述 |
| --- | --- |
| `ReaderWriterLock` 和 `ReaderWriterLockSlim` | 这允许多个线程处于**读模式**，一个线程处于具有写锁的**写模式**，并且一个具有读访问权限的线程处于**可升级读模式**，从中线程可以升级到写模式，而无需放弃对资源的读访问权限。 |
| `Mutex` | 像 `Monitor` 一样，它提供对共享资源的独占访问，但它用于进程间同步。 |
| `Semaphore` 和 `SemaphoreSlim` | 这些通过定义插槽来限制可以同时访问资源或资源池的线程数量。这被称为资源限流，而不是资源锁定。 |
| `AutoResetEvent`和`ManualResetEvent` | 事件等待句柄允许线程通过向彼此发出信号并等待彼此的信号来同步活动。 |

# 理解异步和等待

C# 5 引入了两个 C#关键字，用于处理`Task`类型。它们在以下情况下特别有用：

+   为**图形用户界面**（**GUI**）实现多任务处理。

+   提高 Web 应用程序和 Web 服务的可扩展性。

在*第十五章*，*使用模型-视图-控制器模式构建网站*中，我们将看到`async`和`await`关键字如何提高网站的可扩展性。

在*第十九章*，*使用.NET MAUI 构建移动和桌面应用程序*中，我们将看到`async`和`await`关键字如何为 GUI 实现多任务处理。您可以在[`github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf`](https://github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf)找到它。

但现在，让我们学习为什么引入了这两个 C#关键字的理论，然后稍后您将看到它们在实践中的使用。

## 提高控制台应用程序的响应性

控制台应用程序的局限之一是您只能在标记为`async`的方法中使用`await`关键字，但是 C# 7 及更早版本不允许`Main`方法标记为 async！幸运的是，C# 7.1 中引入的一个新功能是支持`Main`中的`async`：

1.  使用您喜欢的代码编辑器将新的控制台应用程序添加到`Chapter12`解决方案/工作区中，命名为`AsyncConsole`。

1.  在 Visual Studio Code 中，选择`AsyncConsole`作为活动的 OmniSharp 项目。

1.  在`Program.cs`中，删除现有的语句并静态导入`Console`，如下面的代码所示：

```cs

使用

static

System.Console;

```

1.  添加语句以创建`HttpClient`实例，请求苹果的主页，并输出它有多少字节，如下面的代码所示：

```cs

HttpClient client = new

（）;

HttpResponseMessage response =

等待

client.GetAsync("http://www.apple.com/"

）;

WriteLine("苹果的主页有{0:N0}字节。"

，

response.Content.Headers.ContentLength);

```

1.  构建项目并注意它是否成功构建。在.NET 5 及更早版本中，您会看到错误消息，如下面的输出所示：

```cs

Program.cs(14,9): error CS4033: The 'await' operator can only be used within an async method. Consider marking this method with the 'async' modifier and changing its return type to 'Task'. [/Users/markjprice/Code/ Chapter12/AsyncConsole/AsyncConsole.csproj]

```

1.  您必须在`Main`方法中添加`async`关键字，并将其返回类型更改为`Task`。从.NET 6 开始，控制台应用程序项目模板使用顶级程序功能为您自动定义具有异步`Main`方法的`Program`类。

1.  运行代码并查看结果，由于苹果经常更改其主页，所以可能会有不同数量的字节，如下面的输出所示：

```cs

苹果的主页有 40,252 字节。

```

## 提高 GUI 应用程序的响应性

到目前为止，在本书中，我们只构建了控制台应用程序。当构建 Web 应用程序、Web 服务和 Windows 桌面和移动应用程序等具有 GUI 的应用程序时，程序员的生活变得更加复杂。

其中一个原因是对于 GUI 应用程序，有一个特殊的线程：**用户界面**（**UI**）线程。

在 GUI 中有两条规则：

+   不要在 UI 线程上执行长时间运行的任务。

+   不要在 UI 线程以外的任何线程上访问 UI 元素。

为了处理这些规则，程序员过去必须编写复杂的代码，以确保长时间运行的任务由非 UI 线程执行，但一旦完成，任务的结果就会安全地传递给 UI 线程呈现给用户。这可能会很快变得混乱！

幸运的是，使用 C# 5 及更高版本，您可以使用`async`和`await`。它们允许您继续编写代码，就好像它是同步的，这使得您的代码清晰易懂，但在底层，C#编译器创建了一个复杂的状态机，并跟踪运行的线程。这有点神奇！

让我们看一个例子。我们将构建一个使用 WPF 的 Windows 桌面应用程序，使用低级类型如`SqlConnection`、`SqlCommand`和`SqlDataReader`从 SQL Server 数据库中的 Northwind 数据库获取员工。只有在您拥有 Windows 和 Northwind 数据库存储在 SQL Server 中时，您才能完成此任务。这是本书中唯一一个不跨平台和现代的部分（WPF 已有 16 年历史！）。

此时，我们专注于使 GUI 应用程序响应。您将在*第十九章*中了解有关 XAML 和构建跨平台 GUI 应用程序的知识，*使用.NET MAUI 构建移动和桌面应用程序*（可在[`github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf`](https://github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf)找到）。由于本书其他地方没有涉及 WPF，我认为这个任务是一个很好的机会，至少可以看到一个使用 WPF 构建的示例应用程序，即使我们不详细讨论它。

让我们走吧！

1.  如果您正在使用 Windows 的 Visual Studio 2022，请向`Chapter12`解决方案添加一个名为`WpfResponsive`的新**WPF Application [C#]**项目。如果您正在使用 Visual Studio Code，请使用以下命令：`dotnet new wpf`。

1.  在项目文件中，请注意输出类型是 Windows EXE，目标框架是.NET 6 for Windows（它不会在其他平台上运行，如 macOS 和 Linux），并且项目使用 WPF。

1.  为项目添加`Microsoft.Data.SqlClient`的包引用，如下面的标记所示：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

<OutputType>WinExe</OutputType>

<TargetFramework>net6.0

-windows</TargetFramework>

<Nullable>enable</Nullable>

<UseWPF>true

</UseWPF>

</PropertyGroup>

**<ItemGroup>**

**<PackageReference Include=**

**"Microsoft.Data.SqlClient"**

**Version=**

**"3.0.0"**

**/>**

**</ItemGroup>**

</Project>

```

1.  构建项目以恢复包。

1.  在`MainWindow.xaml`中，在`<Grid>`元素中，添加元素来定义两个按钮、一个文本框和一个列表框，以垂直方式布置在一个堆栈面板中，如下面的标记所示：

```cs

<

Grid

**<**

**StackPanel**

**>**

**<**

**Button**

**Name**

**=**

**"GetEmployeesSyncButton"**

**Click**

**=**

**"GetEmployeesSyncButton_Click"**

**>**

**同步获取员工**

**</**

**Button**

**>**

**<**

**Button**

**Name**

**=**

**"GetEmployeesAsyncButton"**

**Click**

**=**

**"GetEmployeesAsyncButton_Click"**

**>**

**异步获取员工**

**</**

**Button**

**>**

**<**

**TextBox**

**HorizontalAlignment**

**=**

**"Stretch"**

**Text**

**=**

**"Type in here"**

**/>**

**<**

**ListBox**

**Name**

**=**

**"EmployeesListBox"**

**Height**

**=**

**"400"**

**/>**

**</**

**StackPanel**

**>**

</

网格

```

Visual Studio 2022 for Windows 对构建 WPF 应用程序有很好的支持，并且在编辑代码和 XAML 标记时会提供智能感知。Visual Studio Code 则没有。

1.  在`MainWindow.xaml.cs`中，在`MainWindow`类中，导入`System.Diagnostics`和`Microsoft.Data.SqlClient`命名空间，然后创建两个`string`常量，用于数据库连接字符串和 SQL 语句，并创建点击这两个按钮的事件处理程序，这些事件处理程序使用这些`string`常量来打开到 Northwind 数据库的连接，并将列表框填充为所有员工的 ID 和姓名，如下面的代码所示：

```cs

private

const

string

connectionString =

"Data Source=.;"

+

"Initial Catalog=Northwind;"

+

"Integrated Security=true;"

+

"MultipleActiveResultSets=true;"

;

private

const

string

sql =

"WAITFOR DELAY '00:00:05';"

+

"SELECT EmployeeId, FirstName, LastName FROM Employees"

;

private

void

GetEmployeesSyncButton_Click

(

object

sender, RoutedEventArgs e

)

{

Stopwatch timer = Stopwatch.StartNew();

using

(SqlConnection connection = new

(connectionString))

{

connection.Open();

SqlCommand command = new

(sql, connection);

SqlDataReader reader = command.ExecuteReader();

while

(reader.Read())

{

string

employee = string

.Format("{0}: {1} {2}"

,

reader.GetInt32(0

), reader.GetString(1

), reader.GetString(2

));

EmployeesListBox.Items.Add(employee);

}

reader.Close();

connection.Close();

}

EmployeesListBox.Items.Add($"同步：

{timer.ElapsedMilliseconds:N0}

ms"

);

}

private

async

void

GetEmployeesAsyncButton_Click

(

object

sender, RoutedEventArgs e

)

{

Stopwatch timer = Stopwatch.StartNew();

using

(SqlConnection connection = new

(connectionString))

{

等待

connection.OpenAsync();

SqlCommand command = new

(sql, connection);

SqlDataReader reader = await

command.ExecuteReaderAsync();

while

(await

reader.ReadAsync())

{

string

employee = string

.Format("{0}: {1} {2}"

,

等待

reader.GetFieldValueAsync<int

>(0

),

等待

reader.GetFieldValueAsync<string

>(1

),

等待

reader.GetFieldValueAsync<string

>(2

));

EmployeesListBox.Items.Add(employee);

}

等待

reader.CloseAsync();

等待

connection.CloseAsync();

}

EmployeesListBox.Items.Add($"异步：

{timer.ElapsedMilliseconds:N0}

ms"

);

}

```

注意以下内容：

+   SQL 语句使用 SQL Server 命令`WAITFOR DELAY`来模拟需要五秒钟的处理。然后从`Employees`表中选择三列。

+   `GetEmployeesSyncButton_Click`事件处理程序使用同步方法打开连接并获取员工行。

+   `GetEmployeesAsyncButton_Click`事件处理程序标记为`async`，并使用`await`关键字使用异步方法打开连接并获取员工行。

+   两个事件处理程序都使用秒表记录操作所花费的毫秒数，并将其添加到列表框中。

1.  不带调试启动 WPF 应用程序。

1.  点击文本框，输入一些文本，注意 GUI 是响应的。

1.  点击“同步获取员工”按钮。

1.  尝试点击文本框，注意 GUI 没有响应。

1.  等待至少五秒，直到列表框填满员工。

1.  点击文本框，输入一些文本，注意 GUI 再次响应。

1.  点击“异步获取员工”按钮。

1.  点击文本框，输入一些文本，注意 GUI 在执行操作时仍然响应。继续输入，直到列表框填满员工。

1.  注意两个操作的时间差异。当同步获取数据时，UI 被阻塞，而异步获取数据时，UI 保持响应。

1.  关闭 WPF 应用程序。

## 改善 Web 应用程序和 Web 服务的可伸缩性

`async`和`await`关键字也可以应用在服务器端构建网站、应用程序和服务时。从客户端应用程序的角度来看，没有任何变化（或者他们甚至可能注意到请求返回所花费的时间略有增加）。因此，从单个客户端的角度来看，使用`async`和`await`在服务器端实现多任务处理会使他们的体验变差！

在服务器端，会创建额外的廉价工作线程来等待长时间运行的任务完成，以便昂贵的 I/O 线程可以处理其他客户端请求而不被阻塞。这提高了 Web 应用程序或服务的整体可伸缩性。可以同时支持更多客户端。

## 支持多任务处理的常见类型

有许多常见类型具有您可以等待的异步方法，如下表所示：

| 类型 | 方法 |
| --- | --- |
| `DbContext<T>` | `AddAsync` , `AddRangeAsync` , `FindAsync` , 和 `SaveChangesAsync` |
| `DbSet<T>` | `AddAsync` , `AddRangeAsync` , `ForEachAsync` , `SumAsync` , `ToListAsync` , `ToDictionaryAsync` , `AverageAsync` , 和 `CountAsync` |
| `HttpClient` | `GetAsync` , `PostAsync` , `PutAsync` , `DeleteAsync` , 和 `SendAsync` |
| `StreamReader` | `ReadAsync`，`ReadLineAsync`和`ReadToEndAsync` |
| `StreamWriter` | `WriteAsync`，`WriteLineAsync`和`FlushAsync` |

**良好实践**：每当您看到以后缀`Async`结尾的方法时，请检查它是否返回`Task`或`Task<T>`。如果是，则可以使用它而不是同步的非`Async`后缀方法。记得使用`await`调用它，并在方法上加上`async`修饰符。

## 在 catch 块中使用 await

当`async`和`await`首次在 C# 5 中引入时，只能在`try`块中使用`await`关键字，而不能在`catch`块中使用。在 C# 6 及更高版本中，现在可以在`try`和`catch`块中都使用`await`。

## 使用异步流

.NET Core 3.0 中，微软引入了流的异步处理。

您可以在以下链接完成有关异步流的教程：[`docs.microsoft.com/en-us/dotnet/csharp/tutorials/generate-consume-asynchronous-stream`](https://docs.microsoft.com/en-us/dotnet/csharp/tutorials/generate-consume-asynchronous-stream)

在 C# 8.0 和.NET Core 3.0 之前，`await`关键字只能与返回标量值的任务一起使用。 .NET Standard 2.1 中的异步流支持允许`async`方法返回一系列值。

让我们看一个模拟的例子，返回三个随机整数作为异步流。

1.  使用您喜欢的代码编辑器将新的控制台应用添加到名为`AsyncEnumerable`的`Chapter12`解决方案/工作区中。

1.  在 Visual Studio Code 中，将`AsyncEnumerable`选择为活动的 OmniSharp 项目。

1.  在`Program.cs`中，删除现有的语句并静态导入`Console`，如下面的代码所示：

```cs

使用

静态

System.Console; // WriteLine()

```

1.  在`Program.cs`底部，创建一个使用`yield`关键字异步返回三个随机数字序列的方法，如下面的代码所示：

```cs

async

静态

IAsyncEnumerable<

int

GetNumbersAsync

()

{

Random r = new

（）；

//模拟工作

await

Task.Delay(r.Next(1500

, 3000

）；

产量

返回

r.Next(0

, 1001

）；

await

Task.Delay(r.Next(1500

, 3000

）；

产量

返回

r.Next(0

, 1001

）；

await

Task.Delay(r.Next(1500

, 3000

）；

产量

返回

r.Next(0

, 1001

）；

}

```

1.  在`GetNumbersAsync`上方，添加语句以枚举数字序列，如下面的代码所示：

```cs

await

foreach

（int

数字

在

GetNumbersAsync

（）

{

WriteLine($"数字：

{number}

"

）；

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

数字：509

数字：813

数字：307

```

# 练习和探索

通过回答一些问题来测试您的知识和理解，进行一些动手实践，并深入研究本章的主题。

## 练习 12.1-测试您的知识

回答以下问题：

1.  您可以找到有关进程的哪些信息？

1.  `Stopwatch`类有多准确？

1.  按照惯例，返回`Task`或`Task<T>`的方法应该应用什么后缀？

1.  在方法内部使用`await`关键字，方法声明必须应用哪个关键字？

1.  如何创建子任务？

1.  为什么应该避免使用`lock`关键字？

1.  何时应该使用`Interlocked`类？

1.  何时应该使用`Mutex`类而不是`Monitor`类？

1.  在网站或网络服务中使用`async`和`await`的好处是什么？

1.  您可以取消任务吗？如果可以，怎么做？

## 练习 12.2-探索主题

使用以下网页上的链接，了解本章涵盖的主题的更多详细信息：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-12---improving-performance-and-scalability-using-multitasking`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-12---improving-performance-and-scalability-using-multitasking)

# 总结

在本章中，您不仅学会了如何定义和启动任务，还学会了如何等待一个或多个任务完成，以及如何控制任务完成顺序。您还学会了如何同步访问共享资源以及`async`和`await`背后的魔法。

在接下来的七章中，您将学习如何为.NET 支持的**应用程序模型**，也就是**工作负载**，如网站和服务，以及跨平台桌面和移动应用程序创建应用程序。
