# 第十二章：*第十二章*：多线程和异步编程

自第一台个人电脑以来，我们已经受益于 CPU 功率的持续增加-这一现象严重影响了开发人员对工具、语言和应用程序设计的选择，而在历史上并没有花费太多精力来编写利用多线程的程序。

在硬件方面，摩尔定律的预测是处理器中晶体管的密度应该每 2 年翻一番，从而提供更多的计算能力，这个预测在一些十年内有效，但我们已经可以观察到它放缓了。即使 CPU 制造商大约 20 年前开始生产多核 CPU，执行代码的能力主要由操作系统（OSes）用于使执行多个进程更加流畅。

这并不意味着代码无法利用并发的力量，而只是只有少量的应用程序完全拥抱了*多线程范式*。这主要是因为我们编写的所有代码都是从操作系统基础设施提供的单个线程顺序执行，除非我们明确请求创建其他线程并编排它们的执行。

这种趋势主要是因为许多编程语言没有提供构造来自动生成多线程代码。这是因为很难提供适合任何用例并有效利用现代 CPU 提供的并发处理能力的语义。

另一方面，有时我们并不真正需要并发执行应用程序代码，但我们无法继续执行，因为需要等待一些未完成的 I/O 操作。同时，阻塞代码执行也是不可接受的，因此需要采用不同的策略。这类问题领域被归类为*异步编程*，需要稍微不同的工具。

在本章中，我们将学习多线程和异步编程的基础知识，并具体了解以下内容：

+   什么是线程？

+   在.NET 中创建线程

+   理解同步原语

+   任务范式

在本章结束时，您将熟悉多线程技术，使用原语来同步代码执行、任务、继续和取消标记。您还将了解潜在的危险操作以及在多个线程之间共享资源时避免问题的基本模式。

我们现在将开始熟悉操作多线程和异步编程所需的基本概念。

# 什么是线程？

每个操作系统都提供抽象来允许多个程序共享相同的硬件资源，如 CPU、内存和输入输出设备。进程是这些抽象之一，提供了一个保留的虚拟地址空间，其运行代码无法逃离。这种基本的沙盒避免了进程代码干扰其他进程，为平衡生态系统奠定了基础。进程与代码执行无关，主要与内存有关。

负责代码执行的抽象是**线程**。每个进程至少有一个线程，但任何进程代码都可以请求创建更多的线程，它们都将共享相同的虚拟地址空间，由所属进程限定。在单个进程中运行多个线程大致相当于一组木工朋友共同完成同一个项目-他们需要协调，关注彼此的进展，并注意不要阻塞彼此的活动。

所有现代操作系统都提供抢占式多任务处理策略，而不是合作式多任务处理。这意味着操作系统的一个特殊组件安排每个线程可以运行的时间，而无需从正在运行的代码中获得任何合作。

提示

早期版本的 Windows，如 Windows 3.x 和 Windows 9x，使用协作式多任务处理，这意味着任何应用程序都可以通过简单的无限循环挂起整个操作系统。这主要是因为 CPU 功率和能力的限制。所有后来的操作系统，如从最初的**NT 3.1 高级服务器**开始的 Windows 版本和所有类 Unix 的操作系统，一直都使用抢占式多任务处理，使操作系统更加健壮，并提供更好的用户体验。

您可以使用任务管理器、Process Explorer 或 Process Hacker 工具查看每个运行进程中使用的线程数。您会立即注意到，许多应用程序，包括所有.NET 应用程序，都使用不止一个线程。这些信息并不能告诉我们太多关于应用程序的设计，因为现代运行时（如.NET CLR）使用后台线程进行内部处理，例如**垃圾回收器**、**终结队列**等。

提示

要查看运行进程使用的线程数，请打开**任务管理器**（*Ctrl* + *Shift* + *Esc*），单击**详细信息**选项卡，并添加**线程**列。可以通过右键单击其中一个网格标题，选择**选择列**菜单项，最后勾选**线程**选项来添加列。

以下屏幕截图显示了一个 C++控制台应用程序，用户的代码使用一个线程，而其他三个线程是由 C++运行时创建的：

![图 12.1 - 任务管理器显示具有四个线程的 NativeConsole.exe 进程](img/Figure_12.1_B12346.jpg)

图 12.1 - 任务管理器显示具有四个线程的 NativeConsole.exe 进程

包含处理线程的基元的命名空间是`System.Threading`，但在本章后面，我们还将介绍`System.Threading.Tasks`命名空间。

当.NET 应用程序启动时，.NET 运行时会准备我们的进程，分配内存并创建一些线程，包括将从`Main`入口点开始执行我们的代码的线程。

以下控制台应用程序访问当前线程并在屏幕上打印当前线程的`Id`：

```cs
static void Main(string[] args)
{
    Console.WriteLine($"Current Thread Id: {Thread.CurrentThread.ManagedThreadId}");
    Console.ReadKey();
}
```

`ManagedThreadId`属性在诊断多线程代码时很重要，因为它将某些代码的执行与特定线程相关联。

此`Id`只能在运行的进程中使用，并且与操作系统线程标识符不同。如果您需要访问本机标识符，您需要使用互操作性，如下面的仅限 Windows 的代码片段所示：

```cs
[DllImport("Kernel32.dll")]
private static extern int GetCurrentThreadId();
static void Main(string[] args)
{
    Console.WriteLine($"Current Thread Id: {Thread.CurrentThread.ManagedThreadId}");
    Console.WriteLine($"Current Native Thread Id: {GetCurrentThreadId()}");
    Console.ReadKey();
}
```

本机`Id`是您可以在**Process Explorer**和**Process Hacker**工具中看到的`Id`，这是与其他本机 API 进行交互所需的`Id`。在下面的屏幕截图中，您可以看到左侧控制台中打印的结果，右侧是 Process Hacker 线程窗口：

![图 12.2 - 控制台应用程序与 Process Hacker 并排显示相同的本机线程 Id](img/Figure_12.2_B12346.jpg)

图 12.2 - 控制台应用程序与 Process Hacker 并排显示相同的本机线程 Id

线程也可以由操作系统、.NET 运行时或某个库创建，而无需我们的代码明确请求。例如，以下类展示了`FileSystemWatcher`类的使用情况，并为每个文件系统操作打印了`ManagedThreadId`属性：`Run`方法打印与主线程关联的 ID，而`Wacher_Deleted`和`Watcher_Created`方法是由操作系统或基础架构创建的线程执行的：

```cs
public class FileWatcher
{
    private FileSystemWatcher _watcher;
    public void Run()
    {
        var path = Path.GetFullPath(".");
        Console.WriteLine($"Observing changes in path: {path}");
        _watcher = new FileSystemWatcher(path, "*.txt");
        _watcher.Created += Watcher_Created;
        _watcher.Deleted += Watcher_Deleted;
        Console.WriteLine($"TID: {Thread.CurrentThread.ManagedThreadId}");
        _watcher.EnableRaisingEvents = true;
    }
    private void Watcher_Deleted(object sender, FileSystemEventArgs e)
    {
        Console.WriteLine($"Deleted occurred in TID: {Thread.CurrentThread.ManagedThreadId}");
    }
    private void Watcher_Created(object sender, FileSystemEventArgs e)
    {
        Console.WriteLine($"Created occurred in TID: {Thread.CurrentThread.ManagedThreadId}");
    }
} 
```

您可以通过创建控制台应用程序并将以下代码添加到`Main`方法来尝试此代码：

```cs
var fw = new FileWatcher();
fw.Run();
Console.ReadKey();
```

现在，如果您开始在控制台文件夹中创建和删除一些`.txt`文件，您将看到类似于这样的东西：

```cs
Observing changes in path: C:\projects\Watch\bin\Debug\netcoreapp3.1
TID: 1
Created occurred in TID: 5
Created occurred in TID: 7
Deleted occurred in TID: 5
Deleted occurred in TID: 5
```

您看到的`TID`号码可能会在每次重新运行应用程序时发生变化：它们既不可预测，也不按相同顺序使用。

我们现在将看到如何创建一个新线程，同时执行一些代码，并检查线程的主要特征。

# 在.NET 中创建线程

创建原始线程在大多数情况下只有在有长时间运行的操作且仅依赖于 CPU 时才有意义。例如，假设我们想计算质数，而不真正关心可能的优化：

```cs
public class Primes : IEnumerable<long>
{
	public Primes(long Max = long.MaxValue)
	{
		this.Max = Max;
	}
	public long Max { get; private set; }
	IEnumerator IEnumerable.GetEnumerator() => ((IEnumerable<long>)this).GetEnumerator();
	public IEnumerator<long> GetEnumerator()
	{
		yield return 1;
		bool bFlag;
		long start = 2;
		while (start < Max)
		{
			bFlag = false;
			var number = start;
			for (int i = 2; i < number; i++)
			{
				if (number % i == 0)
				{
					bFlag = true;
					break;
				}
			}
			if (!bFlag)
			{
				yield return number;
			}
			start++;
		}
	}
}
```

`Primes`类实现了`IEnumerable<long>`，这样我们可以轻松枚举质数，`Max`参数用于限制结果序列，否则将受`long.MaxValue`的限制。

调用上述代码非常容易，但是由于计算可能需要很长时间，它会完全阻塞执行线程：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading;
// namespace and class declaration omitted for clarity
Console.WriteLine("Start primes");
foreach (var n in new Primes(1000000))   {  /* ...  */ }
Console.WriteLine("End primes"); // the wait is too long!
```

这里发生的情况是主线程正在忙于计算质数。由于抢占式多任务处理，这个线程将被操作系统调度程序中断，以便让其他进程的线程有机会运行它们的代码。然而，由于我们的应用程序没有其他线程执行应用程序代码，我们只能等待。

在任何桌面应用程序中，无论是控制台还是 GUI，用户体验都会很糟糕，因为鼠标和键盘的任何交互都会*被阻塞*。更糟糕的是，GUI 甚至无法重新绘制屏幕内容，因为唯一的线程被质数计算占用了。

第一步是将阻塞代码移到一个单独的方法中，这样我们就可以在一个新的独立线程中执行它：

```cs
private void Worker(object param)
{
    PrintThreadInfo(Thread.CurrentThread);
    foreach (var n in new Primes(1000000))
    {
        Thread.Sleep(100);
    }
    Console.WriteLine("Computation ended!");
}
```

`Thread.Sleep`方法仅用于观察 CPU 使用情况。然后，`Sleep`告诉操作系统暂停当前线程的执行一段时间，以*毫秒*为单位。通常，不建议在生产代码中调用`Sleep`，因为它会阻止线程被重用。在本章后面，我们将发现更好的方法来在我们的代码中插入延迟。

`Worker`方法没有什么特别之处，它可能会选择性地获取一个对象参数，该参数可用于初始化局部变量。我们不直接调用它，而是要求基础设施在新线程的上下文中调用它：

```cs
Console.WriteLine("Start primes");
PrintThreadInfo(Thread.CurrentThread);
var t1 = new Thread(Worker);
//t1.IsBackground = true; // try with/without this line
t1.Start();
Console.WriteLine("Primes calculation is happening in background");
```

从上述代码中可以看出，创建了`Thread`对象，但线程尚未启动。我们必须显式调用`Start`方法才能启动它。这很重要，因为`Thread`类还有其他重要的属性，只能在线程启动之前设置。

最后，使用`PrintThreadInfo`方法打印主线程的详细信息。请注意，有些属性并不总是可用。因此，我们必须在打印`Priority`或`IsBackground`之前检查线程是否正在运行。由于`ThreadState`枚举具有`Flags`属性，而`Running`状态为零，官方文档（https://docs.microsoft.com/en-us/dotnet/api/system.threading.threadstate?view=netframework-4.8#remarks）提醒我们要检查`Stopped`和`Unstarted`位是否未设置：

```cs
private void PrintThreadInfo(Thread t)
{
    var sb = new StringBuilder();
    var state = t.ThreadState;
    sb.Append($"Id:{t.ManagedThreadId} Name:{t.Name} State:{state} ");
    if ((state & (ThreadState.Stopped | ThreadState.Unstarted)) == 0)
    {
        sb.Append($"Priority:{t.Priority} IsBackground:{t.IsBackground}");
    }
    Console.WriteLine(sb.ToString());
}
```

执行上述代码的结果如下：

```cs
Start primes
Id:1 Name: State:Running Priority:Normal IsBackground:False
Primes calculation is happening in background
Id:5 Name: State:Running Priority:Normal IsBackground:False
```

即使这是一个微不足道的例子，我们还是必须观察一些事情：

+   首先，我们无法保证关于`Primes calculation …`和`Id:5 …`行的*输出顺序*。它们可能以*相反的顺序*出现。为了获得*确定性行为*，您需要应用我们将在*理解同步原语*部分讨论的同步技术。

+   另一个重要的考虑是*CPU 使用率*。如果你打开**任务管理器**，在**性能**选项卡下，你可以设置查看每个逻辑 CPU 的单独图表。在下面的截图中，你可以看到一个四核 CPU，有八个逻辑核心（多亏了英特尔超线程技术！）。你可能还想显示内核时间（以较深的颜色显示），因为内核模式只执行操作系统和驱动程序的代码，而用户模式（以较浅的颜色显示）只执行我们编写的代码。这种区别将使你立即看到哪个应用程序代码正在执行：

![图 12.3 - 任务管理器显示所有逻辑处理器](img/Figure_12.3_B12346.jpg)

图 12.3 - 任务管理器显示所有逻辑处理器

如果我们现在执行我们的代码而没有`Sleep`调用，我们会发现其中一个 CPU 将显示更高的 CPU 使用率，因为一个线程一直在消耗操作系统分配的全部执行时间。这个单个线程会影响总共（100%）CPU 时间的*100% / 8 个 CPU = 12.5%*。事实上，在计算过程中，**任务管理器**的**详细信息**选项卡将显示你的进程大约消耗了 CPU 的 12%：

![图 12.4 - 任务管理器显示分布在所有可用逻辑 CPU 上的执行时间](img/Figure_12.4_B12346.jpg)

图 12.4 - 任务管理器显示分布在所有可用逻辑 CPU 上的执行时间

线程计算在多个逻辑 CPU 上*分布*。每当操作系统中断线程，安排另一个进程的其他工作，然后回到我们的线程时，线程可能在任何其他逻辑 CPU 上执行。

只是作为一个实验，你可以通过在`Worker`方法的开头添加以下代码来强制执行在特定的逻辑 CPU 上进行：

```cs
var threads = System.Diagnostics.Process.GetCurrentProcess().Threads;
var processThread = threads
    .OfType<System.Diagnostics.ProcessThread>()
    .Where(pt => pt.Id == GetCurrentThreadId())
    .Single();
processThread.ProcessorAffinity = (IntPtr)2; // CPU 2
```

这段代码需要在类内部进行以下声明：

```cs
[DllImport("Kernel32.dll")]
private static extern int GetCurrentThreadId();
```

这些新的代码行检索了我们进程的所有`ProcessThread`对象的列表，然后过滤出与正在执行的本机 ID 匹配的`ProcessThread`对象。

设置`ProcessorAffinity`后，新的执行将完全加载逻辑 CPU `2`，如下面的截图所示（CPU `2`的浅蓝色部分完全填满了矩形）：

![图 12.5 - 任务管理器显示 CPU 2 完全加载了示例代码的执行](img/Figure_12.5_B12346.jpg)

图 12.5 - 任务管理器显示 CPU 2 完全加载了示例代码的执行

在启动线程之前，我们有可能通过设置一个或多个这些属性来塑造线程的特性：

+   `Priority`属性是由操作系统调度程序使用的，用于决定线程可以运行的时间段。给予它高优先级将减少线程挂起的时间。

+   `Name`属性在调试时很有用，因为你可以在 Visual Studio 线程窗口中看到它。

+   我们简要讨论了`ThreadState`属性，它可以有许多不同的值。其中之一——`WaitSleepJoin`——代表一个正在`Wait`方法中或正在睡眠的线程。

+   `CurrentCulture`和`CurrentUICulture`属性由某些依赖于*区域*的 API 读取。例如，当你将数字或日期转换为字符串（使用`ToString`方法）或使用相反的转换的`Parse`静态方法时，当前的区域设置将被使用。

+   `IsBackground`属性指定线程是否应该在仍然活动时阻止进程终止。当为 true 时，进程将不会等待线程完成工作。在我们的示例中，如果你将其设置为 true，那么你可以通过按任意键来结束进程。

你可能已经注意到`Thread`类有`Abort`方法。它不应该被使用，因为它可能会破坏内存状态或阻止托管资源的正确处理。

终止线程的正确方法是从最初启动的方法中正常退出。在我们的情况下，这是`Worker`方法。你只需要一个简单的`return`语句。

我们已经看到了如何手动创建线程，但还有一种更方便的方法可以在单独的线程中运行一些代码——`ThreadPool`类。

## 使用 ThreadPool 类

我们花了一些时间研究线程的特性，这确实非常有用，因为线程是基本的代码执行构建块。手动创建线程是正确的，只要它执行与 CPU 相关且运行时间长的代码。无论如何，由于线程的成本取决于操作系统，因此最好创建适量的线程并重用它们。它们的数量非常依赖于可用的逻辑 CPU 和其他因素，这就是为什么最好使用`ThreadPool`抽象的原因。

静态的`ThreadPool`类提供了一个线程池，可以用来运行一些并发计算。一旦代码终止，线程就会回到池中，可以在不需要销毁和重新创建的情况下，为将来的操作提供可用性。

提示

请注意不要修改从`ThreadPool`中选择的线程的任何属性。例如，如果修改了`ProcessorAffinity`，即使线程被重用于不同的目的，此设置仍将有效。如果需要修改线程的属性，手动创建仍然是最佳选择。

使用`ThreadPool`类运行我们的`Worker`非常简单：

```cs
Console.WriteLine("Start primes");
PrintThreadInfo(Thread.CurrentThread);
ThreadPool.QueueUserWorkItem(Worker);
Console.WriteLine("Primes calculation is happening in background");
```

请注意，`Thread`类构造函数和`QueueUserWorkItem`接受的委托参数是不同的，但接受对象参数的委托对两者都兼容。

我们已经看到了如何启动并行计算，但我们仍然无法编排它们的执行。如果算法应在不同的线程上运行，我们需要知道它的终止以及如何访问结果。

提示

`ThreadPool`被许多流行的库使用，包括随.NET 运行时一起提供的基类库。每当需要访问需要 I/O 操作的资源，而这些操作可能需要一段时间才能成功或失败时，大多数情况下会使用`ThreadPool`。这些资源包括数据库、文件系统对象或可以通过网络访问的任何资源。

每当需要并发访问资源时，无论是通过 I/O 操作检索的资源还是内存中的对象实例，都可能需要同步其访问。在下一节中，我们将看到如何同步线程执行。

# 理解同步原语

每当编写单线程代码时，任何方法执行都是顺序进行的，开发人员无需采取特殊操作。另一方面，当一些代码在单独的线程上执行时，需要同步以确保避免两种危险的并发条件——**竞争**和**死锁**。这些问题的类别在设计时必须小心避免，因为它们的检测很困难，而且可能偶尔发生。

**竞争条件**是指两个或多个线程访问未受保护的共享资源，或者线程的执行根据时间和底层进程架构的不同而表现不同的情况。

*死锁条件*发生在两个或多个线程之间存在循环依赖以访问资源的情况。

编写可能从多个线程执行的代码时，一般建议如下：

+   尽量避免共享资源。它们的访问必须通过锁进行同步，这会影响执行性能。

+   栈是你的朋友。每当调用一个方法时，局部栈是私有的，确保局部变量不会与其他调用者和线程共享。

+   每当您需要在多个线程之间共享资源时，请使用文档验证它是否是线程安全的。每当它不是线程安全的时候，锁必须保护资源或代码序列。

+   即使共享资源是线程安全的，您也必须考虑是否需要原子地执行一些语句，以保证它们的可靠性。

线程库有许多可用于保护资源的原语，但我们将更多地关注那些更有可能在异步上下文中使用的原语，这是本章将涵盖的最重要的主题。

有两组同步原语：

+   由操作系统在*内核模式*中实现的原语

+   由.NET 类库提供的*用户模式*中的同步原语

这种区别非常重要，因为每当您通过系统调用转换到内核模式时，操作系统都必须保存本地调用和堆栈，这将在操作的性能上产生影响。内核模式原语的优势在于能够为它们命名并使它们跨进程共享，提供强大的机器级同步机制。

以下示例显示了来自`ThreadPool`的两个线程打印`Ping`和`Pong`。每个线程通过等待匹配的`ManualResetEventSlim`来与另一个线程同步：

```cs
public void PingPong()
{
    bool quit = false;
    var ping = new ManualResetEventSlim(false);
    var pong = new ManualResetEventSlim(false);
    ThreadPool.QueueUserWorkItem(_ =>
    {
        Console.WriteLine($"Ping thread: {Thread.CurrentThread.ManagedThreadId}");
        while (!quit)
        {
            pong.Wait();
            pong.Reset();
            Console.WriteLine("Ping");
            Thread.Sleep(1000);
            ping.Set();
        }
    });
    ThreadPool.QueueUserWorkItem(_ =>
    {
        Console.WriteLine($"Pong thread: {Thread.CurrentThread.ManagedThreadId}");
        while (!quit)
        {
            ping.Wait();
            ping.Reset();
            Console.WriteLine("Pong");
            Thread.Sleep(1000);
            pong.Set();
        }
    });
    pong.Set();
    Console.ReadKey();
    quit = true;
}
```

创建了两个事件之后，两个线程被运行并打印它们正在运行的线程的 ID。在这些线程内部，每次执行都会在`Wait`方法中暂停，这样可以避免线程消耗任何 CPU 资源。在清单的末尾，`pong.Set`方法启动游戏并解除第一个线程的阻塞。由于事件是*手动*的，它们必须被重置为未发信号状态以供下一次使用。此时，会打印一条消息，延迟模拟一些艰苦的工作，最后，另一个事件被发信号，这将导致第二个线程解除阻塞。

或者，我们可以使用`ManualResetEvent`内核事件，其使用方法非常相似。例如，它具有`WaitOne`方法，而不是`Wait`。但是，如果我们在高性能同步算法中使用这些事件，将会有很大的差异。以下表格显示了使用流行的 Benchmark.NET 微基准库测量的两种同步原语的比较。这两个测试只是调用`Set()`，然后调用`Reset()`方法：

```cs
|          Method |        Mean |     Error |    StdDev |
|---------------- |------------:|----------:|----------:|
| KernelModeEvent | 1,892.11 ns | 24.463 ns | 22.883 ns |
|   UserModeEvent |    25.67 ns |  0.320 ns |  0.283 ns |
```

这两者之间存在大约两个数量级的差异，这绝对不可忽视。

除了能够使用内核事件来同步在不同进程中运行的代码之外，它们还可以与强大的`WaitHandle.WaitAny`和`WaitAll`方法结合使用，如下例所示：

```cs
public void WaitMultiple()
{
    var one = new ManualResetEvent(false);
    var two = new ManualResetEvent(false);
    ThreadPool.QueueUserWorkItem(_ =>
    {
        Thread.Sleep(3000);
        one.Set();
    });
    ThreadPool.QueueUserWorkItem(_ =>
    {
        Thread.Sleep(2000);
        two.Set();
    });
    int signaled = WaitHandle.WaitAny(
        new WaitHandle[] { one, two }, 500);
    switch(signaled)
    {
        case 0:
            Console.WriteLine("One was set");
            break;
        case 1:
            Console.WriteLine("Two was set");
            break;
        case WaitHandle.WaitTimeout:
            Console.WriteLine("Time expired");
            break;
    }
}
```

您可以通过以毫秒为单位表示的三个超时时间来查看不同的结果。主要思想是尽快退出等待，只要任何事件或超时到期，以先到者为准。

提示

Windows 操作系统的内核对象可以在等待原语中全部使用。例如，如果您想等待多个进程退出，您可以使用前面代码块中显示的`WaitHandle`原语与进程句柄一起使用。

我们只是刚刚触及了表面，但官方文档中有许多示例展示了各种同步对象的使用。相反，我们将继续专注于对本书更为相关的内容，例如从多个线程访问共享资源。

在以下示例中，我们有一个名为`_shared`的共享变量，一个用于同时启动所有线程的`ManualResetEvent`对象，以及一个简单的对象。`Shared`属性利用`Thread.Sleep`，在 setter 上引起了显式的线程上下文切换。当操作系统调度程序在系统中将控制权预先交给另一个线程时，这种切换通常会发生。这不是一个技巧；它只是增加了 getter 和 setter 不会被每个线程连续执行的概率：

```cs
int _shared;
int Shared
{
    get => _shared;
    set { Thread.Sleep(1); _shared = value; }
}
ManualResetEvent evt = new ManualResetEvent(false);
object sync = new object();
```

以下方法将共享变量初始化为`0`并创建 10 个线程，所有线程都执行相同的 lambda 中的代码：

```cs
public void SharedResource()
{
    Shared = 0;
    var loop = 100;
    var threads = new List<Thread>();
    for (int i = 0; i < loop; i++)
    {
        var t = new Thread(() =>
        {
            evt.WaitOne();
            //lock (sync)
            {
                Shared++;
            }
        });
        t.Start();
        threads.Add(t);
    }
    evt.Set(); // make all threads start together
    foreach (var t in threads)
        t.Join();   // wait for the thread to finish
    Console.WriteLine($"actual:{Shared}, expected:{loop}");
}
```

所有线程立即启动并阻塞在`WaitOne`事件中，该事件由`Set`方法解除阻塞。这为许多线程以相同的时间执行 lambda 中的代码提供了更多机会。最后，我们调用`Join`等待每个线程的执行结束并打印结果。

这段代码的同步问题存在于线程将读取一个值，将数字增加到 CPU 寄存器中，并将结果写回变量。由于许多线程将读取相同的值，写回变量的值是旧的，其真实的*当前*值丢失了。

通过取消注释锁定语句，我们指示编译器用**关键部分**包围大括号中的语句，这是最快的用户模式同步对象。这将导致对该代码的访问进行序列化，对性能产生非常显著的影响，这是必要且不可避免的。

我们在开始时创建的空对象实例不应更改；否则，不同的线程将等待不同的临界区。请注意，`lock`参数可以是任何引用类型。例如，如果您需要保护一个集合，可以直接锁定它，而无需外部对象的帮助。无论如何，在我们的示例中，`Shared`是一个值类型，必须借助一个单独的引用类型来保护它。

如果您用一个简单的字段替换`Shared`属性，问题发生的可能性将会降低。此外，编译器配置（调试与发布）将产生很大的差异，因为*内联*和其他优化使得在访问字段或简单属性时更有可能发生线程上下文切换。物理硬件配置和 CPU 架构是可能会极大影响这些测试结果的其他变量。

提示

单元测试*不适合*确保不存在竞争条件或死锁等问题。此外，请注意，虚拟机是最不适合测试并发代码的环境，因为调度程序比在物理硬件上运行的操作系统更可预测。

我们已经看到了如何确保一系列语句被原子地执行，没有干扰。但如果只是为了确保底层`_shared`字段的原子增量，有一个更方便的工具——`Interlocked`类。

`Interlocked`是一个静态类，公开了一些有用的方法来确保某些操作的原子性。例如，我们可以使用以下代码而不是`lock`语句，这样做会更快，即使只限于`Interlocked`公开的操作。以下代码显示了如何原子地增加`_shared`变量：

```cs
Interlocked.Increment(ref _shared);
```

除其他事项外，我们可以用它来原子地写入变量并获取旧值（`Exchange`方法），或者读取大小大于可用本机寄存器的变量（`Read`方法）。

我们已经看到了为什么需要同步以及我们可以用来防止这些并发访问问题的主要工具。但现在，是时候引入一个抽象，这将使每个开发人员的生活更轻松——任务范式。

# 任务范式

并发主要是关于设计具有非常松散耦合的工作单元的算法，这通常是不可能的，或者会使复杂性超出任何可能的好处。

异步编程与操作系统和设备的异步性相关，无论是因为它们触发事件还是因为完成所请求的操作需要时间。每当用户移动鼠标、在键盘上输入键或从互联网检索一些数据时，操作系统都会在一个单独的线程中向我们的进程呈现数据，我们的代码必须准备好消费它。

最简单的例子之一是从磁盘加载文本文件并计算字符串长度，这可能与文件长度不同，这取决于编码：

```cs
public int ReadLength(string filename)
{
    string content = File.ReadAllText(filename);
    return content.Length;
}
```

一旦调用此方法，调用线程将被阻塞，直到操作系统和库完成读取。该操作可能非常快速，也可能非常缓慢，这取决于其大小和技术。文本文件可能位于网络附加存储（NAS）、本地磁盘、损坏的 USB 键或通过虚拟专用网络（VPN）访问的远程服务器上。

在桌面应用程序的上下文中，任何阻塞线程都会导致不愉快的用户体验，因为主线程已经负责重绘用户界面并响应来自输入设备的事件。

服务器应用程序也不例外，因为任何阻塞线程都是一种资源，无法有效地与其他请求一起使用，从而阻止应用程序扩展并为其他用户提供服务。

几十年来，解决这个问题的方法是通过手动创建一个单独的线程来执行长时间运行的代码，但是最近，.NET 运行时引入了任务范式，C#语言引入了`async`和`await`关键字。从那时起，整个.NET 库已经进行了修订，以拥抱这种范式，提供返回基于任务的操作的方法。

任务库，位于`System.Threading.Tasks`命名空间中，以及语言集成提供了一个抽象，大大简化了异步操作的管理。任务代表了执行明确定义的工作单元。无论您处理并发性还是异步事件，任务都定义了给定的工作及其生命周期，从创建到完成，其选项包括成功、失败或取消。

通过定义其他任务应该在给定操作之后立即执行来组合任务。这个链接的任务称为**延续**，并且通过**任务调度程序**从库中自动安排。

默认情况下，任务库提供了一个默认实现（`TaskScheduler.Default`静态属性），大多数开发人员永远不需要深入研究。默认实现使用`ThreadPool`来编排任务的执行，并使用*工作窃取*技术将任务队列重新分配到多个线程上，以提供负载平衡，并防止任务被阻塞太长时间。请注意，这个默认实现足够聪明，最终会决定直接在主线程上安排任务的执行，而不是从池中选择一个。勇敢的人可以尝试创建自定义调度程序来改变调度策略，但这并不是很多开发人员真正需要做的事情。

稍后，在*同步上下文*部分，我们将讨论**同步上下文**，它允许延续在调用线程中执行，并避免使用前一节中描述的同步原语的需要。

让我们从读取文本文件的异步版本开始研究任务：

```cs
Task<string> content = File.ReadAllTextAsync(filename);
```

这个方法的新版本*立即完成*，而不是返回文件的内容，而是返回表示*正在进行*操作的对象。

由于我们刚刚启动了尚未完成的操作，管理完成所需的步骤如下：

1.  将异步操作后面的代码（获取字符串长度）重构为一个单独的方法。这个方法相当于旧式的回调，不能在异步操作完成之前调用。

1.  监视正在进行的任务，并在完成或失败时提供通知。

1.  完成后，检索结果并在主线程上同步执行（通过**同步上下文**），或者如果出现问题则抛出异常。如果我们不想搞乱潜在的竞争条件，这一步是至关重要的。

1.  调用我们在第一个点重构出来的回调。

当然，我们不必手动管理所有这些机制。任务库的第一个有趣的优势是它支持继续，这允许开发人员指定任务成功完成后要执行的代码：

```cs
public Task<int> ReadLengthAsync(string filename)
{
    Task<int> lengthTask = File.ReadAllTextAsync(filename)
        .ContinueWith(t => t.Result.Length);
    return lengthTask;
}
```

这个新版本比创建线程和手动编写同步代码要好，即使它还可以进一步改进。`ContinueWith`方法包含了确定其他代码在文件成功读取后立即执行的代码。

`t`变量包含任务，该任务要么失败，要么成功完成。如果成功，`t.Result`包含从`ReadAllTextAsync`方法获取的字符串内容。

无论如何，我们仍然没有长度；我们只是表达了如何在*将来*检索`ReadAllTextAsync`的结果后检索长度。这就是为什么`lengthTask`变量是`Task<int>`，即整数的承诺。

我强烈建议尝试使用任务和继续，因为有时它们需要直接管理。

但 C#语言还引入了两个宝贵的关键字，进一步简化了我们需要编写的代码。`await`关键字用于指示操作的结果以及其后的所有内容都是一个继续的一部分。

由于`await`关键字，编译器重构并生成新的**中间语言**（**IL**）代码，以提供适当的异步操作和继续的管理。最终的代码以异步方式加载文件内容并返回字符串长度如下：

```cs
public async Task<int> ReadLengthAsync(string filename)
{
    string content = await File.ReadAllTextAsync(filename);
    return content.Length;
}
```

编译器重构的代码部分不仅仅是一个继续。编译器生成一个*类*来负责监视任务进度的状态机，并生成一个调用适当代码或抛出异常的方法，一旦任务状态发生变化。

提示

如果您想深入了解生成的代码的更多细节，可以使用**ILSpy**工具（https://github.com/icsharpcode/ILSpy/releases）并查看生成的 IL 代码。

显然，编译器可以摆脱承诺，让我们处理返回的内容，对吗？实际上不是 - 这段代码被重构了，我们编写的代码是表达我们的期望，而不是方法中通常和顺序发生的事情。

事实上，前面的代码看起来矛盾，因为`content.Length`整数只会在将来可用，但我们直接从返回类型为`Task<int>`的方法中返回它。

这就是`async`关键字发挥作用的地方：

+   `async`关键字是一个修饰符，每次我们想在方法内部使用`await`时都必须指定。

+   `async`关键字告诉我们，`return`语句指定了一个未来的对象或值。在我们的情况下，我们返回`int`，但`async`告诉我们它实际上是一个`Task<int>`。

+   如果一个`async`方法返回`void`，返回类型变成了非泛型的`Task`。

我们现在有一个异步处理文件的方法，但我们不得不将签名从`int`改为`Task<int>`。

当您在 lambda 中使用`await`关键字时，也需要使用`async`关键字。例如，让我们看一下以下代码：

```cs
Func<int, int, Task<int>> adder = 
    async (a, b) => await AddAsync(a, b);
```

在方法上使用`async`意味着所有调用者也必须采用任务范式，否则他们可能无法知道操作何时完成。

## 异步方法的同步实现

我们已经看到了任务范例如何影响方法签名，我们知道方法签名有多重要。当它出现在公共 API 或接口中时，它是一个合同，大多数情况下我们不能更改。从设计的角度来看，对于预期可能使用任务实现的方法的可能性，这可能非常有价值，但也有一些不需要异步性的情况。

对于这些情况，`Task`类公开了一个静态方法，允许我们直接构建一个带有或不带结果的已完成任务。在下面的示例中，异步方法同步返回一个已完成的任务：

```cs
public Task WriteEmptyJsonObjectAsync(string filename)
{
    File.WriteAllText(filename, "{}");
    return Task.CompletedTask;
}
```

`CompletedTask`属性仅为整个应用程序域创建一次；因此，它非常轻量级，不应引起性能方面的担忧。

如果需要返回一个值，我们可以使用静态的`FromResult`方法，它在每次调用时内部创建一个新的已完成`Task`：

```cs
public Task<int> AddAsync(int a, int b)
{
    return Task.FromResult(a + b);
}
```

每次我们添加两个数字时创建一个对象绝对是性能问题，因为它直接影响垃圾收集器需要做的工作量。因此，最近，微软引入了`ValueTask`类。

## 偶尔的异步方法

`ValueTask`不可变结构是对同步结果或`Task`的便捷包装。这种进一步的抽象旨在简化那些需要方法具有异步签名，但其实现只是偶尔异步的情况。

我们在上一节中使用任务定义的`AddAsync`方法可以很容易地转换为使用`ValueTask`结构：

```cs
public ValueTask<int> AddAsync(int a, int b)
{
    return new ValueTask<int>(a + b);
}
```

对于微不足道的总和使用`Task`的开销是明显的；因此，每当在热路径（一些性能关键代码）中应该调用这样的方法时，肯定会引起性能问题。

无论如何，有些情况下，您可能需要将`ValueTask`转换为`Task`，以便从本章剩余部分讨论的所有实用工具中受益。转换可通过`AsTask`方法实现，该方法返回包装的任务（如果有），或者如果没有，则创建一个全新的`Task`。

## 中断任务链 - 阻塞线程

给定一个任务，如果调用`Wait`方法或访问`Result`获取器属性，它们将阻塞线程执行，直到任务完成或取消。任务范例背后的理念是避免阻塞线程，以便它们可以被重用于其他目的。但是阻塞也可能引发非常严重的副作用。

由于异步编程的默认线程来源是`ThreadPool`（如果耗尽其线程），任何进一步的请求都将自动阻塞。这种现象被称为**线程饥饿**。

一般建议是避免等待，而是使用`await`关键字或延续来完成一些工作。

## 手动创建任务

有时库不提供异步行为，但您不希望保持当前线程忙碌太长时间。在这种情况下，您可以使用`Task.Run`方法，该方法安排执行 lambda，这很可能会发生在一个单独的线程中。下面的示例展示了如何读取文件的长度，如果我们之前使用的异步`ReadAllTextAsync`方法不可用：

```cs
public Task<int> ReadLengthAsync(string filename)
{
    return Task.Run<int>(() =>
    {
        var content = File.ReadAllText(filename);
        return content.Length;
    });
}
```

您应该始终优先使用提供的异步版本，而不是使用`Run`方法，因为安排此任务的线程将一直阻塞，直到同步执行结束。

现在，我们将看看在任务内部有大量工作要做时，采取的最佳行动方案是什么。

## 长时间运行的任务

即使您不阻塞线程，当异步堆栈从不等待并成为长时间运行的作业时，仍然存在饥饿的风险，使线程保持忙碌。

这些情况可以用两种不同的策略来处理：

+   第一种是手动“创建线程”，这是我们在本章开头已经讨论过的。当你需要更多控制或需要修改线程属性时，这是最好的策略。

+   第二种可能性是*通知任务调度程序*任务将要运行很长时间。这样，调度程序将采取不同的策略，完全避免`ThreadPool`。以下代码显示了如何运行一个长时间运行的任务：

```cs
var t = new Task(() => Thread.Sleep(30000),
    TaskCreationOptions.LongRunning);
t.Start();
```

基本建议是尝试将长时间的工作拆分成可以轻松转换为任务的较小工作单元。

## 打破任务链 - 火而忘

我们已经看到，拥抱任务范式需要修改整个调用链。但有时这是不可能的，也不可取。例如，在桌面 WPF 应用程序的上下文中，您可能需要在按钮点击事件处理程序中写入文件：

```cs
void Button_Click(object sender, RoutedEventArgs e) { ... }
```

我们不能改变它的签名来返回一个`Task`；而且，出于两个原因，这也没有意义：

+   调用库在任务之前设计过，它将无法管理任务的进度。

+   这是设计为**火而忘**操作之一，意味着你并不真的在乎它们会花多长时间或者它们将计算出什么结果。

对于这些情况，你可以拥抱`async`/`await`关键字，同时根本不使用返回的`Task`：

```cs
async void Button_Click(object sender, RoutedEventArgs e)
{
    await File.WriteAllTextAsync("log.txt", "something");
    // ... other code
}
```

但请记住，当你打破任务链时，你失去了知道操作是否会完成或失败的可能性。

信息框

每当你在你的代码中看到`async void`时，你应该想知道它是否可能是一个潜在的错误，或者只是你真的不想知道最终会发生什么。多年来，使用`async void`而不是`async Task`的习惯一直是异步代码中错误的主要来源。

同样，如果你只是调用一个异步方法而不等待它（或使用`ContinueWith`方法之一），你将失去对调用的控制，获得相同的*火而忘*行为，因为异步方法在启动异步操作后立即返回。此外，不等待异步操作之后的所有代码将同时执行，存在竞争条件或访问尚不可用的数据的风险：

```cs
void Button_Click(object sender, RoutedEventArgs e)
{
    File.WriteAllTextAsync("log.txt", "something");
}
```

我们已经看到了当一切顺利完成时管理异步操作是多么简单，但代码可能会抛出异常，我们需要适当地捕获它们。

## 任务和异常

当出现问题时，有两种异常可能发生。第一种是在调用任何异步方法之前发生的，而第二种与异步代码中发生的异常有关。

以下示例展示了这两种情况：

```cs
public Task<int> CrashBeforeAsync()
{
    throw new Exception("Boom");
}
public Task<int> CrashAfterAsync()
{
    return Task.FromResult(0)
        .ContinueWith<int>(t => throw new Exception("Boom"));
}
```

在第一种情况下，我们告诉调用者我们将返回一个`Task<int>`，但还没有开始任何异步操作。这种情况与同步方法中发生的情况完全相同，可以相应地捕获：

```cs
public Task<int> HandleCrashBeforeAsync()
{
    Task<int> resultTask;
    try
    {
        resultTask = CrashBeforeAsync();
    }
    catch (Exception) { throw; }
    return resultTask;
}
```

另一方面，如果异常发生在继续执行中，异常不会立即发生；它只会在任务被“消耗”时发生：

```cs
public async Task<int> HandleCrashAfterAsync()
{
    Task<int> resultTask = CrashAfterAsync();
    int result;
    try
    {
        result = await resultTask;
    }
    catch (Exception) { throw; }
    return result;
}
```

一旦`resultTask`完成为*故障*，异常已经发生，但是编译器生成的代码捕获了它并将其分配给`Task.Exception`属性。由于在`Task`内可能同时发生多个异常，生成的代码将所有捕获的异常封装在单个`AggregateException`中。`AggregateException`中的`InnerException`和`InnerExceptions`属性包含原始异常。

每当你想要处理异常并立即解决它们时，你可能希望使用继续而不是`await`关键字：

```cs
public Task<int> HandleCrashAfter2Async()
{
    Task<int> resultTask = CrashAfterAsync();
    try
    {
        return resultTask.ContinueWith<int>(t =>
        {
           if (t.IsCompletedSuccessfully) return t.Result;
           if(t.Exception.InnerException is OverflowException)
               return -1;
           throw t.Exception.InnerException;
        });                
    }
    catch (Exception) { throw; }
}
```

正如我们之前提到的，在*faulted*任务中的异常会在结果被*消耗*时立即抛出，我们之前在使用`await`的情况下提到过。然而，当访问`t.Result`属性时，这也可能发生。

提示

`Task`类公开了`GetAwaiter`方法，该方法返回表示异步操作的内部结构。你可以使用`task.GetAwaiter().GetResult()`来获取异步操作的结果，以及`task.Result`，但两者有一点不同。实际上，在发生异常时，前者返回原始异常，而后者返回包含原始异常的`AggregateException`。

最后，值得一提的是，我们可以使用静态的`Task.FromException<T>`方法来重写`CrashAfterAsync`方法：

```cs
public Task<int> CrashAfterAsync() =>
    Task.FromException<int>(new Exception("Boom"));
```

与我们在`FromResult<T>`中看到的类似，创建了一个新的`Task`，但这次，它的状态被初始化为*faulted*，并包含所需的异常。

前面的例子相当抽象，但足够简洁，让你了解如何根据抛出异常的时间来正确处理异常。有许多常见的情况会发生这种情况。这种二元性的一个真实例子是，在准备 JSON 参数时发生序列化异常，或者在 HTTP rest 调用期间由于网络故障而发生异常。

除了转换为故障状态，任务也可以被取消，这要归功于任务范例提供的内置标准机制。

## 取消任务

与故障不同，取消是由调用者请求来中断一个或多个任务的执行。取消可以是强制性的，也可以是超时，当给定任务不应该花费超过一定时间时，这是非常有用的。

从调用者的角度来看，取消模式源自`CancellationTokenSource`类，它提供了三种不同的构造函数：

+   当你愿意通过强制调用`Cancel`方法来取消任务时，使用默认构造函数。

+   其他构造函数接受`int`或`TimeSpan`，它们确定在触发取消之前的最长时间，除非任务在此之前完成。

在下面的例子中，我们将使用从定时`CancellationTokenSource`获得的`CancellationToken`来取消三个工作方法中的一个：

```cs
public async Task CancellingTask()
{
    CancellationTokenSource cts2 = new
        CancellationTokenSource(TimeSpan.FromSeconds(2));
    var tok2 = cts2.Token;
    try
    {
        await WorkForever1Async(tok2);
        //await WorkForever2Async(tok2);
        //await WorkForever3Async(tok2);
        Console.WriteLine("let's continue");
    }
    catch (TaskCanceledException err)
    {
        Console.WriteLine(err.Message);
    }
}
```

`Token`属性返回一个只读结构，可以被多个消费者使用，而不会影响垃圾收集器，甚至不会被复制，因为它是不可变的。

这里正在检查的第一个消费者接受`CancellationToken`，并将其正确传播给任何其他接受取消的方法。在我们的例子中，只有`Task.Delay`，这是一个非常方便的方法，用于指示基础设施在 5 秒后触发继续执行：

```cs
public async Task WorkForever1Async(
    CancellationToken ct = default(CancellationToken))
{
    while (true)
    {
        await Task.Delay(5000, ct);
    }
}
```

前面代码的执行结果是任务被取消，这通过从`await`关键字生成的代码转换为`TaskCanceledException`：

```cs
A task was canceled.
```

另一种可能性是，当工作程序只执行*同步*代码并且仍然需要被取消时：

```cs
public Task WorkForever2Async(
    CancellationToken ct = default(CancellationToken))
{
    while (true)
    {
        Thread.Sleep(5000);
        if (ct.IsCancellationRequested)
            return Task.FromCanceled(ct);
    }
}
```

请注意使用`Thread.Sleep`而不是`Delay`方法，这是因为我们需要同步实现。

`Thread.Sleep`方法非常不同，因为它完全阻塞线程，并防止线程在其他任何地方被重用，而`Task.Delay`会生成一个请求，在指定的时间过去后立即调用以下代码作为继续执行。

更有趣的部分是测试`IsCancellationRequested`布尔属性，以允许协作取消任务。通过显式检查该属性来进行协作是必要的，因为在释放某些资源之前，你可能不需要中断执行，无论是在数据库上还是其他地方。

再次执行前面的方法的结果将如下：

```cs
A task was canceled.
```

第三种情况是当你不想抛出任何异常，而只是从执行中返回：

```cs
public async Task WorkForever3Async(
    CancellationToken ct = default(CancellationToken))
{
    while (true)
    {
        await Task.Delay(5000);
        if (ct.IsCancellationRequested) return;
    }
}
```

在这种情况下，我们小心地避免将`CancellationToken`传播到底层调用，因为使用`await`会触发异常。

这个最终的`WorkForever3Async`方法的执行不会引发任何异常，并让执行继续正常进行：

```cs
let's continue
```

这种实现的缺点是取消可能不会立即发生。`Task.Delay`将需要完成，而不管取消，这在最坏的情况下可能在 5 秒之前无法发生。

我们已经看到任务范式如何使运行异步操作变得极其容易，但我们如何同时运行多个异步请求呢？它们可能会并行运行，以避免无用的等待。

## 监视任务的进度

用户开始长时间运行操作后，提供反馈非常重要，以避免用户变得沮丧。当你控制正在发生的事情时，比如一些耗时的算法，这是可能的。然而，当长时间运行的操作依赖于对外部库的调用时，监视进度是不可能的。

任务库没有专门支持监视进度，但.NET 库提供了`IProgress<T>`，可以轻松实现这一目标。这个接口只提供一个成员——`void Report(T value)`——这给了实现细节完全的自由。在最简单的情况下，`T`将是一个表示进度的整数值，表示为百分比。

例如，加载操作可以实现如下：

```cs
public async Task Load(IProgress<int> progress = null)
{
    var steps = 30;
    for (int i = 0; i < steps; i++)
    {
        await Task.Delay(300);
        progress?.Report((i + 1) * 100 / steps);
    }
}
```

在我们的情况下，这个方法通过调用`Task.Delay`来模拟异步操作，必须预测与进度的 100%相关的总步数。在每一步之后，调用`Report`方法来通知我们当前的百分比，但要确保代码受到保护，以防进度为空，因为消费者可能对接收这样的反馈不感兴趣。

在消费者端，首先要做的是创建进度提供程序，这只是一个实现`IProgress<int>`的类：

```cs
public class ConsoleProgress : IProgress<int>
{
    void IProgress<int>.Report(int value) =>
        Console.Write($"{value}%  ");
}
```

最后，调用者只需将提供程序实例传递给`Load`方法：

```cs
await test.Load(new ConsoleProgress());
```

正如你所期望的那样，输出如下：

```cs
3%  6%  10%  13%  16%  20%  23%  26%  30%  33%  36%  40%  43%  46%  50%  53%  56%  60%  63%  66%  70%  73%  76%  80%  83%  86%  90%  93%  96%  100%
```

`IProgress<T>`的通用参数可能被用来暂停执行或触发更复杂的逻辑，比如暂停/恢复行为。

## 并行化任务

一个常见的编程任务是从互联网上检索一些资源。例如，通过 HTTP 下载资源的基本代码如下：

```cs
public async Task<byte[]> GetResourceAsync(string uri)
{
    using var client = new HttpClient();
    using var response = await client.GetAsync(uri);
    response.EnsureSuccessStatusCode();
    return await response.Content.ReadAsByteArrayAsync();
}
```

由于`EnsureSuccessStatusCode`，任何失败都会触发异常，将捕获的责任留给调用者。此外，我们甚至没有设置任何标头，但对我们的目的来说已经足够了。

我们已经知道如何调用这个异步方法来下载图像，但现在的挑战是选择正确的策略来下载许多图像：

+   第一个问题是：*我们如何并行下载多个图像？* 如果我们需要下载 10 张图像，我们不想将下载每张图像所需的时间相加。无论如何，我们不会讨论如果我们需要下载数百万张图像时可以扩展多少。这超出了关于异步机制的讨论范围。

+   第二个问题是：*我们需要同时使用它们吗？* 在这种情况下，我们可以使用`Task.WhenAll`辅助方法，它接受一个任务数组，并返回一个表示整体操作的单个任务。

对于这些示例，我们将使用名为*Lorem PicSum*（[`picsum.photos/`](https://picsum.photos/)）的在线免费服务。每次你向代码中看到的 URI 发出请求时，都会检索到一个新的不同大小为 200 x 200 的图像。当然，你可以使用你选择的任何 URI：

```cs
public async Task NeedAll()
{
    var uri = "https://picsum.photos/200";
    Task<byte[]>[] tasks = Enumerable.Range(0, 10)
        .Select(_ => GetResourceAsync(uri))
        .ToArray();
    Task allTask = Task.WhenAll(tasks);
    try
    {
        await allTask;
    }
    catch (Exception)
    {
        Console.WriteLine("One or more downloads failed");
    }
    foreach (var completedTask in tasks)
        Console.WriteLine(
            $"New image: {completedTask.Result.Length}");
}
```

使用`Enumerable.Range`是一种很好的方式，可以重复执行给定次数的操作。实际上，我们并不关心生成的数字；事实上，我们在`Select`方法中使用了`discard (_)`标记而不是变量。

`Select` lambda 只是启动下载操作，返回相应的任务，我们还没有等待。相反，我们要求`WhenAll`方法创建一个新的`Task`，一旦所有任务都成功完成，就会发出信号。如果任何任务失败，从`await`关键字生成的代码将导致抛出异常。

从`WhenAll`方法获得的任务不能用于检索结果，但它保证我们可以访问所有任务的`Result`属性。因此，在等待`allTask`之后，我们迭代`tasks`数组，检索所有已下载图像的`byte[]`数组。以下是同时等待所有下载的输出：

```cs
New image: 6909
New image: 3846
New image: 8413
New image: 9000
New image: 7057
New image: 8565
New image: 6617
New image: 8720
New image: 4107
New image: 6763
```

在许多情况下，这是一个很好的策略，因为我们可能需要在继续之前获取所有资源。另一种选择是等待第一次下载，这样我们就可以开始处理它，但我们仍然希望同时下载它们以节省时间。

这种替代策略可以借助`WaitAny`方法来实现。在下面的示例中，开始下载没有什么不同。我们只是添加了一个`Stopwatch`类，以显示下载结束时花费的毫秒数：

```cs
public async Task NeedAny()
{
    var sw = new Stopwatch();
    sw.Start();
    var uri = "https://picsum.photos/200";
    Task<byte[]>[] tasks = Enumerable.Range(0, 10)
        .Select(_ => GetResourceAsync(uri))
        .ToArray();
    while (tasks.Length > 0)
    {
        await Task.WhenAny(tasks);
        var elapsed = sw.ElapsedMilliseconds;
        var completed = tasks.Where(t => t.IsCompleted).ToArray();
        foreach (var completedTask in completed)
            Console.WriteLine($"{elapsed} New image: {completedTask.Result.Length}");
        tasks = tasks.Where(t => !t.IsCompletedSuccessfully).ToArray();
    }
}
```

`while`循环用于处理所有未完成的任务。最初，`tasks`数组包含所有任务，但每次`WhenAny`完成时，至少一个任务已完成。已完成的任务立即在屏幕上打印出来，并显示自操作开始以来经过的毫秒数。其他任务被重新分配给`tasks`变量，这样我们就可以循环回去处理已完成的任务，直到最后一个任务。这种新方法的输出如下：

```cs
368 New image: 9915
368 New image: 6032
419 New image: 6486
452 New image: 9810
471 New image: 7030
514 New image: 10009
514 New image: 10660
593 New image: 6871
658 New image: 2738
12850 New image: 6072
The last image took a lot of time to download, probably because the online service throttles the requests. Using WhenAll, we would have to wait about 13 seconds before getting them all. Instead, we could start processing as soon as each image was available.
```

当然，你可以将这两种方法结合起来。例如，如果你想在不超过 100 毫秒的时间内尽可能多地获取已下载的图像，只需用以下一行替换`WhenAny`行：

```cs
await Task.WhenAll(Task.Delay(100), Task.WhenAny(tasks));
```

换句话说，我们要求等待任何任务（至少一个），但不超过 100 毫秒。`while`循环将重复操作，就像我们之前所做的那样，消耗所有剩余的任务：

```cs
345 New image: 8416
345 New image: 7315
345 New image: 8237
345 New image: 6391
345 New image: 5477
457 New image: 9592
457 New image: 3922
457 New image: 8870
563 New image: 3695
```

在测试这些代码片段时，一定要在循环中运行它们，因为第一次运行可能会受到**即时**编译器的严重影响。

我们已经看到`Task`类提供了一个非常强大的构建块来消耗异步操作，但这需要提供异步行为的库。在下一节中，我们将看到如何暴露手动任务并触发其完成。

## 使用`TaskCompletionSource`对象发出任务信号

回到本章开头*什么是线程？*部分的文件监视器示例中，你可能还记得`FileSystemWatcher`暴露了事件，而没有采用任务范例。你可能会想知道我们是否编写了某种适配器来利用任务库提供的所有好工具的能力，答案是*是*。

`TaskCompletionSource`对象提供了一个重要的构建块，我们可以用它来暴露异步行为。它在生产者端创建和使用，以信号操作的完成，无论是成功还是失败。它通过`Task`属性提供了任务对象，客户端必须使用它来等待通知。

以下类使用`FileSystemWatcher`来监视当前文件夹中的文件系统。`Deleted`事件停止通知并通知完成源文件成功删除。类似地，`Error`事件设置了最终将在`await`语句的消费方触发的异常：

```cs
public class DeletionNotifier : IDisposable
{
   private TaskCompletionSource<FileSystemEventArgs> _tcs;
   private FileSystemWatcher _watcher;
   public DeletionNotifier()
   {
      var path = Path.GetFullPath(".");
      Console.WriteLine($"Observing changes in path: {path}");
      _watcher = new FileSystemWatcher(path, "*.txt");
      _watcher.Deleted += (s, e) =>
      {
         _watcher.EnableRaisingEvents = false;
         _tcs.SetResult(e);
      };
      _watcher.Error += (s, e) =>
      {
         _watcher.EnableRaisingEvents = false;
         _tcs.SetException(e.GetException());
      };
  }
  public Task<FileSystemEventArgs> WhenDeleted()
  {
    _tcs = new TaskCompletionSource<FileSystemEventArgs>();
    _watcher.EnableRaisingEvents = true;
    return _tcs.Task;
  }
  public void Dispose() => _watcher.Dispose();
}
```

每当调用`WhenDeleted`方法时，都会创建一个新的完成源，启动文件监视器，并将负责通知的`Task`返回给客户端。

从消费者的角度来看，这个解决方案很棒，因为它消除了任何复杂性：

```cs
var dn = new DeletionNotifier();
var deleted = await dn.WhenDeleted();
Console.WriteLine($"Deleted: {deleted.Name}");
```

这种解决方案的缺点是一次只能检测到一个删除。

此外，由于`Deleted`事件中的代码关闭了通知，循环内调用`WhenDeleted`方法可能会导致删除事件丢失。

但我们可以解决这个问题！稍微复杂一点的解决方案是将事件缓冲在一个线程安全的队列中，并通过出队可用事件的方式改变`WhenDeleted`方法的策略，如果有的话。

以下是修改后的代码：

```cs
public class DeletionNotifier : IDisposable
{
  private TaskCompletionSource<FileSystemEventArgs> _tcs;
  private FileSystemWatcher _watcher;
  private ConcurrentQueue<FileSystemEventArgs> _queue;
  private Exception _error;
  public DeletionNotifier()
  {
    var path = Path.GetFullPath(".");
    Console.WriteLine($"Observing changes in path: {path}");
    _queue = new ConcurrentQueue<FileSystemEventArgs>();
    _watcher = new FileSystemWatcher(path, "*.txt");
    _watcher.Deleted += (s, e) =>
    {
      _queue.Enqueue(e);
      _tcs.TrySetResult(e);
    };
    _watcher.Error += (s, e) =>
    {
      _watcher.EnableRaisingEvents = false;
      _error = e.GetException();
      _tcs.TrySetException(_error);
    };
    _watcher.EnableRaisingEvents = true;
  }
  public Task<FileSystemEventArgs> WhenDeleted()
  {
    if (_queue.TryDequeue(out FileSystemEventArgs fsea))
      return Task.FromResult(fsea);
    if (_error != null)
      return Task.FromException<FileSystemEventArgs>(_error);
    _tcs = new TaskCompletionSource<FileSystemEventArgs>();
    return _tcs.Task;
  }
  public void Dispose() => _watcher.Dispose();
}
```

再一次，我们可以仅使用任务库工具来解决问题。根据用例，这种策略需要每次重新创建一个新的`TaskCompletionSource<T>`，并且由于它是一个引用类型，可能会影响性能，受到垃圾回收的影响。如果我们需要重用相同的通知对象，我们可以通过创建一个自定义通知对象来实现。

实际上，`await`关键字只需要一个实现了名为`GetAwaiter`的方法的对象，返回一个实现了`INotifyCompletion`接口的对象。这个对象又必须实现一个`IsCompleted`属性和模拟`TaskCompletionSource`行为的所有必需机制。

在*进一步阅读*部分，你会发现一篇有趣的文章，名为*await anything*，来自微软官方博客，深入探讨了这个主题。

## 同步上下文

根据我们正在编写的应用程序，不是所有的线程都是平等的。桌面应用程序有一个主线程，只允许在屏幕上绘制和处理图形控件。GUI 库围绕着消息队列的概念工作，每个请求都被发布。主线程负责出队这些消息，并将它们分派到实现所需行为的用户定义处理程序中。

每当在 UI 线程之外的线程上发生某些事件时，必须进行编组操作，这将导致消息被发布到主线程管理的队列中。在 UI 线程中编组消息的两个常见示例是 Windows Forms 应用程序中的`Control.Invoke`和 Windows Presentation Foundation 中的`Dispatcher.Invoke`。

信息框

WPF 的第一个预发布版本是多线程的。但是，代码复杂性要求用户处理多线程，并且用户代码中可能出现的 bug 提高了门槛。甚至许多 C++库，如 DirectX 和 OpenGL，大多数都是单线程的，以减少复杂性。

在服务器端，ASP.NET 应用程序也有主线程的上下文，但实际上不只有一个——事实上，每个用户的请求都有自己的主线程。

`SynchronizationContext`是一个抽象的基类，定义了一种在*特殊*线程上执行一些代码的标准方式。这并不是魔术；事实上，正在执行的代码是在一个 lambda 中定义的，并且被发布到一个队列中。在主线程上，基础设施提供的一些代码会出队 lambda，并在其上下文中执行它。

这种自动编组是基本的，因为在执行任何异步方法之后，比如从互联网下载图像，你希望避免调用`Invoke`方法，该方法需要将结果编组回主线程，这是为了使用返回的数据更新用户界面所必需的。

每当你等待某个异步操作时，生成的代码会负责*捕获*当前的`SynchronizationContext`，并确保继续在特定线程上执行。基本上，你不需要做任何事情，因为基础设施已经为你做了。

我们完成了吗？实际上并没有，因为有时情况并非如此。根据我们所说，以下示例中的三个 ID 应该都是相同的：

```cs
public async Task AsyncTest1()
{
    Console.WriteLine($"Id: {Thread.CurrentThread.ManagedThreadId}");
    await Task.Delay(100);
    Console.WriteLine($"Id: {Thread.CurrentThread.ManagedThreadId}");
    await Task.Delay(100);
    Console.WriteLine($"Id: {Thread.CurrentThread.ManagedThreadId}");
}
```

这不是因为它是一个默认情况下不设置任何同步上下文的控制台应用程序。这是因为在 Microsoft 的`Console`类的文档中有原因。您会在文档页面的末尾看到*线程安全*部分，其中指出*此类型是线程安全的*。换句话说，没有理由返回到原始线程。

如果您创建一个新的 Windows Forms 应用程序，并在按钮单击处理程序中调用该代码，您会发现 ID 始终相同，这要归功于`SynchronizationContext`。

始终重要的是要了解异步代码在线程方面发生了什么，因为有时将结果返回到主线程不是理想的，因为返回有性能影响。例如，库开发人员在编写异步代码时必须非常小心，因为他们无法知道他们的代码是否会在有同步上下文或没有同步上下文的情况下执行。

一个明显的例子是库开发人员正在处理来自网络的数据块。每个块都是通过异步 HTTP 请求检索的，块的数量可能非常多，就像以下示例中一样：

```cs
public async Task AsyncLoop()
{
    Console.WriteLine($"Id: {Thread.CurrentThread.ManagedThreadId}");
    byte[] data;
    while((data = await GetNextAsync()).Length > 0)
    {
        Console.WriteLine($"Id: {Thread.CurrentThread.ManagedThreadId}");
        // process data
    }
}
```

除非处理代码将与 UI（或与主线程相关的任何内容）交互，禁用同步上下文绝对是性能的提升，并且非常容易实现：

```cs
public async Task AsyncLoop()
{
    Console.WriteLine($"Id: {Thread.CurrentThread.ManagedThreadId}");
    byte[] data;
    while((data = await GetNextAsync().ConfigureAwait(false)).Length > 0)
    {
        Console.WriteLine($"Id: {Thread.CurrentThread.ManagedThreadId}");
        // process data
    }
} 
```

通过将`ConfigureAwait`方法应用于异步方法，操作的结果将不会发布回主线程，并且生成的继续将在辅助线程上执行（无论异步操作是否计划在不同的线程上）。

这种修改后的行为有两个后果：

+   将消息发布到主线程队列会产生*性能影响*。例如，库开发人员可能希望在进行一些内部工作时将`ConfigureAwait`设置为`false`以提高性能。

+   每当您决定使用`Wait`方法或`Result`属性同步执行异步方法时，您可能会遇到*死锁*。这可能是因为同步上下文将执行返回到繁忙的主线程。虽然应该通过永远不使用`Wait`和`Result`来避免这种情况，但另一种方法是通过将`ConfigureAwait`设置为`false`，使调用在辅助线程上完成执行。

请注意，如果您真的希望在辅助线程上继续执行，确保对所有后续调用应用`ConfigureAwait`。事实上，第一个异步调用在没有使用`ConfigureAwait`的情况下执行将导致执行返回到主线程。

由于`ConfigureAwait`后面的代码在辅助线程上执行，记得手动返回到主线程，以避免竞争条件。例如，要更新 UI，您必须调用相关的*Windows Forms*或*WPF* `Invoke`方法。

任务范式是编程语言中的一场革命，如果没有新语言关键字和编译器生成的魔法，它是无法存在的。这一新特性在其他语言中也引起了很大的共鸣。例如，ECMAScript 2017 通过提供承诺和 async/await 关键字支持来采纳了这些概念。

在这一漫长的章节中，我们学到了异步编程的重要性，以及任务库如何使异步代码直观且易于编写，同时又不会让我们过多地去关注隐含的复杂性。除了获得对这些工具的一般理解之外，现在重要的是要进行实验并深入研究每个方面，以掌握这些技术。

# 总结

在本章中，我们讨论了任何开发人员都可以利用的最重要的工具，以利用多线程和异步编程技术。

构建块是基本的抽象，允许代码在不同的执行上下文中运行，而不管它们当前运行在哪个操作系统上。这些原语必须以智慧使用，但与本地语言和库相比，这并不以任何方式限制开发人员的可能性。

除此之外，当涉及到与那些本质是异步的事件交互时，任务范式提供了一种自然的方法。`System.Threading.Tasks`命名空间提供了与异步现象交互所需的所有抽象。

该库已经被广泛重组和扩展以支持任务范式。最重要的是，该语言提供了`async`和`await`关键字来分解复杂性，并使异步世界流畅地进行，就像是过程性代码一样。

在下一章中，我们将学习文件、文件流和序列化的概念。

# 测试你所学到的东西

1.  如果你有一个非常消耗 CPU、持续时间很长的算法要运行，你会采用手动创建线程、使用任务库还是使用线程池中的哪种策略？

1.  命名一个可以用来写文件并增加内存中整数值的高效同步技术。

1.  你应该使用什么方法来暂停执行 100 毫秒，为什么？

1.  你应该怎么做来等待多个异步操作产生的结果？

1.  你如何创建一个等待 CLR 事件的任务？

1.  当一个方法的签名中有`Task`但没有使用任何异步方法时，你应该返回什么？

1.  你如何创建一个长时间运行的任务？

1.  一个按钮点击处理程序正在异步访问互联网以加载一些数据。你应该使用`Control.Invoke`来更新屏幕上的结果吗？为什么？

1.  在一个`Task`上评估使用`ConfigureAwait`方法的原因是什么？

1.  在使用了`ConfigureAwait(false)`之后，你能直接更新 UI 吗？

# 进一步阅读

+   一个非常强大的库，可以用来测量一些代码的性能是 Benchmark.NET ([`benchmarkdotnet.org/articles/overview.html`](https://benchmarkdotnet.org/articles/overview.html))，这也被微软内部用来对运行时和核心库进行优化。

+   如果你想构建自己的*awaitable*对象，你不能错过微软团队的这篇文章，描述了基础架构的工作原理：[`devblogs.microsoft.com/pfxteam/await-anything/`](https://devblogs.microsoft.com/pfxteam/await-anything/)。

+   要深入了解同步上下文和`ConfigureAwait`，你可以阅读以下文章：[`devblogs.microsoft.com/dotnet/configureawait-faq/`](https://devblogs.microsoft.com/dotnet/configureawait-faq/)。
