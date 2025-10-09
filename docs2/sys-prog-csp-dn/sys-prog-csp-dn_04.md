

# 线程缠结的问题

*并发* *和线程*

**线程**和**并发**是大多数开发者认为他们已经全部了解的东西。理论听起来很简单，但在实践中，线程是许多错误发生的地方，也是所有那些令人沮丧的 bug 的起源。线程可能相当复杂，但 BCL 和 CLR 团队的人们已经尽他们所能帮助我们，使事情尽可能简单。

一旦你掌握了它，线程就是你技能的一个很好的补充，并且可以在你的系统中产生重大影响。

在本章中，我们将探讨以下主题：

+   并发和线程是什么？

+   线程在.NET 和 Windows 内部是如何工作的？

+   CLR 是如何帮助我们的？

+   async/await 是什么？

+   我们如何同步线程并使它们协同工作？

+   我如何确保我的代码在处理线程时表现良好？

+   我如何使用线程上的集合？

让我们深入了解这个迷人的主题！

# 技术要求

本章中所有的源代码和示例都可以从本书的 GitHub 仓库下载，网址为[`github.com/PacktPublishing/Systems-Programming-with-C-Sharp-and-.NET/tree/main/SystemsProgrammingWithCSharpAndNet/Chapter04`](https://github.com/PacktPublishing/Systems-Programming-with-C-Sharp-and-.NET/tree/main/SystemsProgrammingWithCSharpAndNet/Chapter04)。

# 并发和线程——基础知识

今天早上，我像往常一样醒来。我起床，洗了个澡，然后穿好衣服。然后我遛狗 30 分钟（今天是星期天）。我回到家，泡了些咖啡，然后坐下来写这篇文档。

我敢肯定，你的日常活动在总体上看起来是一样的。你做一件事，然后做下一件事。事情按顺序完成。有时，当我遛狗时，我会给其他时区的人打电话，但大多数时候，我一次只做一件事。这样更有效率。如果我坐下来写这一章，但五分钟后停下来遛狗，然后让我跑回家写五分钟，再跑回去接狗再走 500 米，事情永远也完成不了。我会因为来回跑而得到锻炼，但这会很低效。

这是一种愚蠢的生活方式（没有评判；如果你这样做，我对此表示理解，但这对我来说不起作用）。

然而，在计算机的情况下，我们倾向于认为这种工作方式可以使工作更快地完成。我们为什么这么想？

计算机不能同时做两件事。不，等等。让我换个说法。CPU 核心不能同时做两件事。在 2005 年 AMD 发布了**Athlon 64 X2 处理器**之前，以及同一年 Intel 发布了 Pentium D 之前，普通的计算机都是单核的。这意味着在 2005 年之前，计算机通常一次只能做一件事。

现在，大多数设备都有多个核心。你的电脑、笔记本电脑和手机都有多核处理器。然而，作为系统程序员，你可能会遇到只有单个核心的设备。想想物联网设备：它们需要便宜且功耗非常低。这些系统通常只有一个核心。系统程序员遇到单核设备比编写其他软件的人更常见。

然而，最终，这并不重要。我的主要开发机器有 16 个核心。这听起来很多。然而，如果我看我的任务管理器，我可以看到许多事情同时运行，远超过这 16 个核心可以处理的。所以，即使在多核环境中，机器也必须做些事情来使所有这些任务得以运行。作为系统程序员，我们必须意识到如何编写我们的软件，以从这些核心中获得最大利益。

因此，我们在这里处理两个独立的话题。一个是并发；另一个是线程。

并发是一种概念，即系统在重叠的时段内执行多个操作序列。这并不是真正的同时执行；那被称为并行。它完全是关于任务在看似相同的时间运行，而不需要等待其他任务。这是一个概念，而不是一种编程技术。

另一方面，线程是程序员的结构。线程是实现并发的一种方式。

值得了解

线程可以是硬件线程或软件线程。CPU 处理第一种类型；第二种类型在我们的软件中处理。**操作系统**（**OS**）可以将线程分配给实际的硬件线程，但作为开发者，你几乎总是要处理软件线程。在这里，我主要会谈论软件线程，但当我指的是硬件线程时，我会指出这一点。

## 并发的起源——中断请求（IRQ）

目前，让我们忽略这样一个事实：除了将负载分散到 CPU 可能拥有的物理核心之外，计算机无法进行多任务处理。为了简化问题，我们将假设计算机可以同时做两件事。

这并不总是如此。在早期，计算机一次只做一件事。这意味着如果你为计算机编写了一些软件，你就完全控制了所有可用的硬件。一切都是你的，而且是你的独有。

嗯，当我说那是你的，我的意思是那主要是你的。有时，会发生一些需要 CPU 注意的事情。在那些日子里，我们有一种叫做**中断请求**（**IRQ**）的东西。中断请求是一种通常与硬件相关的硬件功能。一个外部设备，如软盘驱动器或调制解调器，可以通过在 CPU 的特定连接上施加电压来向 CPU 发出信号。当发生这种情况时，CPU 会完成它正在执行的指令，将所有状态存储在内存中，查找属于那个中断请求的地址（可能有多个），然后在该地址处启动代码。当该功能完成时，整个过程会逆转：CPU 将加载之前存储的状态，并继续执行原始代码，就像什么都没发生一样。

这种机制工作得相当不错，但存在许多潜在问题。例如，只有少数几个中断请求线路可用。如果你的代码覆盖了附加到某些硬件的另一段代码的注册，那么该硬件将无法正常工作。

更糟糕的是，如果你犯了一个愚蠢的错误，你的代码从未从中断请求中返回，你可能会使整个机器停止运行。它将简单地永远不会从你的代码中返回，并且正在运行的程序将无限期地挂起。因此，你必须非常小心，确保你的代码中没有这样的错误！

中断请求（IRQs）今天仍在使用，尤其是在像 Raspberry Pi 这样的低功耗设备中。我们将在本书的后面遇到它们。

## 协作式和抢占式多任务处理

中断请求（IRQs）工作得还可以，但它们应该由硬件设备使用。由于中断请求的数量并不多，并且它们有杀死正在运行进程的潜在能力，所以我们已经不再在常规软件中使用它们。

然而，拥有计算机却只能用它做一件事情，这似乎是一种资源的浪费。计算机变得越来越强大。它们很快就能做比我们要求它们做的事情更多的事情。那时，多任务操作系统就出现了。

例如，Windows 95 之前的 Windows 版本，如 Windows 3.1，使用了一种叫做**协作式多任务处理**的东西。原理相当简单。一段代码会做某事，当它认为可以休息一下时，它就会告诉操作系统：“嘿，我在休息；如果你需要我做些什么，请告诉我。”然后它会停止执行。这意味着操作系统可以将 CPU 时间分配给另一个进程。

我们称之为协作式多任务处理，因为我们期望软件能够合作并公平地共享资源。

当然，如果一个程序行为不当，它仍然可以声称所有的 CPU 时间，从而阻止其他软件按预期运行。

需要一种更好的方法。Windows NT 3.1 以及后来的 Windows 95 做得更好：它们引入了**抢占式多任务处理**。

这个想法很简单：为进程分配一些运行时间，当时间到了，就存储该进程的状态，将其停放某处，然后继续下一个进程。当原始进程再次需要做某事时，操作系统将程序重新加载到内存中，并恢复状态，然后进程可以继续。除非进程跟踪时钟，否则进程对它休眠的时间一无所知。

进程不能再声称所有可用的 CPU 时间。如果其时间已用完，操作系统将暂停该进程。

预先多任务处理仍然是现代操作系统今天的工作方式。

然而，所有这些都涉及在计算机上同时运行的多个进程。我们如何让一个进程同时做多件事情呢？好吧，一个解决方案就是使用线程。

# C#中的线程

线程是一个概念，它允许计算机在您的程序中同时做更多的事情。就像操作系统允许多个程序同时运行一样，线程允许您的程序在应用程序中并发运行多个流程。线程不过是您程序中的一个**执行流程**。您始终至少有一个线程：当程序开始执行时启动的那个线程。我们称之为主线程。运行时管理这个线程，您对其控制很少。然而，所有其他的线程都是您的，您可以随意对它们进行操作。

线程并不是什么神奇的东西。基本原理很简单：

1.  创建一个您想要运行的方法、函数或任何其他代码片段。

1.  创建一个线程，给它传递方法的地址。

1.  启动线程。

1.  操作系统或运行时在运行主线程的同时执行那个方法或函数。

1.  您可以监控那个线程的进度。您可以等待它结束，或者您可以使用一种“发射后不管”的策略，只需让它完成其工作即可。

您如何执行这些步骤取决于您想使用哪个版本。您是选择.NET 方式还是选择我们熟知的 Win32 API 的兔子洞？

在.NET 中，线程由一个实际的类（或者更准确地说，是一个类的实例）表示。在 Win32 中，它们只是由 Win32 API 创建的东西。

## Win32 线程

在 Win32 中，您使用`CreateThread` API 创建线程。我想向您展示它是如何工作的，但我要坦白：您可能永远不会在您的代码中这样做。使用 Win32 API 创建线程的方法有很多更好的选择。尽管如此，在某些情况下，完全控制 Win32 线程可能是必要的。

让我向您展示如何在 Win32 API 中完成这个操作。

我们将首先声明一个`delegate`。这个`delegate`是包含线程执行的工作的函数的形式：

```cs
public delegate uint ThreadProc(IntPtr lpParameter);
```

由于我们正在调用 Win32 API，我们需要导入它们：

```cs
[DllImport("kernel32.dll", SetLastError = true)]
public static extern IntPtr CreateThread(
    IntPtr lpThreadAttributes,
    uint dwStackSize,
    ThreadProc lpStartAddress,
    IntPtr lpParameter,
    uint dwCreationFlags,
    out uint lpThreadId
);
[DllImport("kernel32.dll", SetLastError = true)]
public static extern bool CloseHandle(IntPtr hObject);
[DllImport("kernel32.dll", SetLastError = true)]
public static extern uint WaitForSingleObject(IntPtr
hHandle, uint dwMilliseconds);
```

我们将导入三个 API：`CreateThread`、`CloseHandle`和`WaitForSingleObject`。

在我们可以使用这些 API 之前，我们必须编写执行有用操作的代码。在这种情况下，这并不是真正有用的代码，但这是将在线程中执行的代码：

```cs
public uint MyThreadFunction(IntPtr lpParameter)
{
    for (int i = 0; i < 1000; i++)
        Console.WriteLine("Unmanaged thread");
    return 0;
}
```

这个 `MyThreadFunction` 函数与之前定义的委托相匹配。

在清除所有这些之后，我们可以创建线程，并让我们的程序执行某些操作。或者更确切地说，它可以同时执行许多操作。下面是操作步骤：

```cs
public void DoWork()
{
    uint threadId;
    var threadHandle = CreateThread(
        IntPtr.Zero,
        0,
        MyThreadFunction,
        IntPtr.Zero,
        0,
        out threadId
    );
    // Wait for the thread to be finished
    WaitForSingleObject(threadHandle, 1000);
    // Clean up
    CloseHandle(threadHandle);
}
```

`DoWork()` 方法通过调用 `CreateThread` Win32 API 创建线程。此 API 有一些参数。让我借助 *表 4.1* 来解释它们的作用：

| **参数** | **描述** |
| --- | --- |
| IntPtr lpThreadAttributes | 安全属性结构的指针 |
| uint dwStackSize | 此线程所需的堆栈大小 |
| ThreadProc lpStartAddress | 线程运行的函数的指针 |
| IntPtr lpParameter | 指向传递给线程的变量的指针 |
| uint dwCreationFlags | 确定线程创建方式的附加标志 |
| out uint lpThreadId | 一个输出参数，包含线程的 ID |

表 4.1：CreateThread Win32 API 的参数

安全属性定义了谁或什么可以访问线程以及此线程可以使用什么。安全属性相当复杂。在这里，我们不会深入探讨它们，主要是因为在处理线程时它们并不常用。在这里，我们将安全属性设置为 `IntPtr.Zero`。

`dwStackSize` 参数定义了线程使用的堆栈大小。如前所述，每个线程都有自己的堆栈，可以在其中存储其值类型。当线程完成后，此堆栈将被回收。

然后，我们获取线程启动时将执行的函数指针。在 C#中，我们可以传递方法的名称，让编译器完成找出内存地址的繁琐工作。

在提供方法启动地址后，我们得到一些更有趣的东西：我们可以将数据传递到线程方法中。`lpParameter` 参数是指向数据所在内存的指针。除非你想使用简单的 `Int32`，否则将数据放入线程中相当繁琐。毕竟，`IntPtr` 是一个 32 位值，所以你可以将一个 int 转换回转换以获取线程函数中的数据。这里我没有传递任何东西，但稍后在本章中我会向你展示如何做到这一点。

接下来是定义系统如何创建线程的标志。除了默认的 `0`，表示“不执行特殊操作”之外，我们可以使用两个标志。这些标志在 *表 4.2* 中解释。

| **标志** | **值** | **含义** |
| --- | --- | --- |
| 0 | 0x00000000 | 不执行特殊操作 |
| `CREATE_SUSPENDED` | 0x00000004 | 创建线程，但立即挂起而不是启动。 |
| `STACK_SIZE_PARAM_IS_A_RESERVATION` | 0x00010000 | 如果设置此标志，则堆栈大小是预留的。如果没有设置，则堆栈大小是已提交的。 |

表 4.2：线程创建选项

`CREATE_SUSPENDED`创建线程，但在创建时将其置于挂起状态。默认行为是立即运行`lpStartAddress`指向的代码。

`STACK_SIZE_PARAM_IS_A_RESERVATION`是一个有趣的标志。这个标志是您可能想要使用 Win32 线程创建版本而不是.NET 版本的主要原因之一。每个线程都有自己的栈。您可以指定该栈应该有多大，但当你这样做时，发生的所有事情只是系统保留该内存。这种保留是一个快速操作。保留只是告诉系统您希望在某个时候使用这么多内存。如果系统没有足够的内存来满足您的请求，您将收到错误。

然而，内存尚未提交。提交意味着操作系统为您请求的内存保留，并将其标记为被进程使用。保留只是告诉它您希望在以后使用该内存。

页面错误

当您的应用程序请求内存或尝试从系统访问内存时，可能会发生某些事情。

第一种情况发生在内存已在您的栈或堆中可用时。您获得了该内存的指针；现在它完全属于您了。

如果内存尚未在您的栈或堆中，但系统上可用，接下来会发生什么。这会导致软页面错误。系统会将新内存添加到当前的栈或堆中。

接下来，可能您想要访问的内存不在您计算机的内存芯片中。在这种情况下，它可能已经被交换到磁盘上。这是一个硬页面错误。操作系统将从磁盘加载内存并将其添加到您的工作集。

页面错误对于增加系统的灵活性非常有用。然而，它们会带来很大的性能损失。

当您保留内存并想要访问它时，可能会发生页面错误。当这种情况发生时，您的应用程序性能将下降。

如果您提交内存，则它在需要时保证可用。这使得您的内存占用更大、速度更快，因为您不会遇到页面错误。

您必须在这里做出选择：您更喜欢两种场景中的哪一种？您可以通过`STACK_SIZE_PARAM_IS_A_RESERVATION`标志来控制栈。

代码示例以两个语句结束：`WaitForSingleObject()`和`CloseHandle()`。本章中的*同步线程*部分更详细地解释了`WaitForSingleObject()`。不过，简短的描述如下：在主线程继续之前等待线程完成。

`CloseHandle`清理所有已使用的资源。是的，这是一个未管理资源。这是一个使用`IDisposable`模式的好地方。

## .NET 线程

.NET BCL 中的线程使用起来要简单得多。当然，当某件事被简化时，您通常会因此牺牲灵活性。

以下示例显示了如何使用.NET 结构执行与 Win32 线程相同的工作。

我们将从线程函数开始，它在新的线程上运行。它几乎与 Win32 示例相同。以下代码片段显示了我们要在线程内运行的代码：

```cs
void MyThreadFunction()
{
    for (var i = 0; i < 1000; i++)
        Console.WriteLine("Managed thread");
}
```

在我们的代码的主体中，我们创建线程，给它运行的功能，然后启动它：

```cs
var myManagedThread = new Thread(MyThreadFunction);
myManagedThread.Start();
myManagedThread.Join();
```

我们创建`Thread`类的新实例，并在构造函数中传递我们想要使用的方法。然后我们启动它。然后，我们使用`Join()`等待它，实际上暂停了主线程，直到我们的新线程完成它正在做的事情。

就这样。如果你与 Win32 版本进行比较，我确信你会欣赏这种简单性。

然而，不要被这种简单性欺骗：这并不意味着你不能控制你的线程。你可以控制它们，并且你可以做比我所展示的更多的事情。例如，你也可以指定你想要为你的线程使用的堆栈大小：

```cs
var myHugeStackSize = 8 * 1024 * 1024; // 8 MB
var myManagedThread = new Thread(MyThreadFunction, myHugeStackSize);
```

在这里，我们为新线程分配了 8 MB 的堆栈。

很高兴了解

32 位应用程序的默认堆栈大小为 1 MB；对于 64 位应用程序，为 4 MB。你很少需要超过这个大小。只有在测试了你的应用程序并发现你确实需要它时，才应该请求大堆栈。

在 Win32 示例中，我们必须明确指出我们想要创建一个处于挂起状态的线程。如果我们没有这样做，它将立即启动。在.NET 中，情况不同。在.NET 中创建的新线程被认为是*未启动*。这意味着它不会立即启动。它也还没有挂起；在行为上存在相当大的差异。

一个挂起的线程已经完全形成，并被放置在操作系统的调度器列表中。它的堆栈已分配，所有资源都存在。

一个`Thread`类。堆栈尚未分配，它还没有被分配给操作系统，因此它还没有在调度器上，等等。

当我们在.NET 线程上调用`Start()`时，运行时会做所有这些工作。创建线程比 Win32 中的`CreateThread()`调用要快得多，但当你启动线程时，这种性能提升就丢失了。把它想象成懒加载初始化。

CLR 的设计者利用了这一点。如果创建线程相对便宜，而只有在使用它们时才变得昂贵，为什么不将创建的负担移到程序开始时呢？启动应用程序需要时间；如果我们稍微延长这一点，那就没关系了。然而，这意味着当它在使用时，我们有一个更快的系统。当我们需要一两个线程时，我们可以有一个线程池可用。这正是他们所做的事情。

一个例子可能会使这更清晰。然而，在我能向你展示那之前，我们必须做一些修改。我们想要创建许多同时运行的线程。为了区分每个线程的输出，我们需要向那个线程传递一些数据，以便它可以显示它。因此，我们需要有存储数据的地方。

我们需要不可变数据，原因将在本章后面讨论线程安全性时变得清晰。C# 9 中添加的`record`是一个实现这一点的绝佳方式：

```cs
internal record ThreadData(int LoopCounter);
```

我们现在可以开始编写在特定线程中执行的方法：

```cs
void MyThreadFunction(object? myObjectData)
{
    // Verify that we have a ThreadData object
    if (myObjectData is not ThreadData myData)
        throw new ArgumentException("Parameter is not a                     ThreadData object");
    // Get the thread ID
    var currentThreadId = Thread.CurrentThread.ManagedThreadId;
    // Write the data to the Console
    Console.WriteLine(
        $"Managed thread in Thread {currentThreadId} " +
        $"with loop counter {myData.LoopCounter}");
}
```

线程获取一个`Nullable<object>`类型的参数。我们不能将其声明为其他类型，因为这是运行时所期望的。

要使用这些数据，我们需要将其转换为正确的类型。

然后，我们将获取当前线程的 ID。每个线程都有一个唯一的 ID，因此我们可以与之交互，尽管我们在这里只会显示它。

让我们创建一些线程：

```cs
for (int i = 0; i < 100; i++)
{
    ThreadData threadData = new(i);
    var newThread = new Thread(MyThreadFunction);
    newThread.Start(threadData);
}
Console.ReadKey();
```

我们将创建一百个线程，并在创建后立即启动它们。我们将给它们一些数据，以查看我们在循环中的位置。

在循环之后，我添加了`Console.ReadKey()`，以确保在所有线程完成之前程序不会退出。当你运行程序时启动的主线程是特殊的：如果它结束，CLR 将结束整个程序并卸载所有内存。所以，保持你的主线程活跃直到你确信所有工作都完成是至关重要的。在实际场景中，你不会使用`Console.ReadLine()`来做这件事，但在这个演示中，它工作得很好。

如果你运行这个程序，你可能会看到线程 ID 随着循环计数器的增加而增加。它们并不相等。CLR 在你运行循环之前已经创建了一打或更多的线程。

如果你将循环增加到执行更多的迭代次数，你最终会看到线程 ID 偶尔相同。CLR 会重用线程以避免线程饥饿。

然而，我承诺要向你展示线程池。将代码中我们原本的 for 循环部分替换为以下代码：

```cs
for (int i = 0; i < 100; i++)
{
    ThreadData threadData = new(i);
    ThreadPool.QueueUserWorkItem(MyThreadFunction, threadData);
}
Console.ReadKey();
```

我们将在这里使用线程池，在需要时从池中取出线程。如果你运行这个程序，你会反复看到相同的线程 ID。线程从池中被取出并使用正确的数据启动。当线程完成时，它会关闭，其资源会被释放，然后被放回池中，以便在需要时再次使用。

负载很小，优势巨大。使用这种系统的系统效率更高。

`ThreadPool`隐藏了许多你可以使用的秘密和技巧，但它的使用在很大程度上已经被**任务并行库**（**TPL**）所取代，它为你处理了大部分工作。让我们看看。

# 任务和并行库 – TPL

TPL 已经存在了一段时间。它是在 2010 年随着.NET 4.0 的发布而引入的。

TPL 简化了我们过去用线程做的许多事情。线程仍然有其位置，尤其是在处理第三方库时。然而，在大多数情况下，我们可以让 TPL 来处理这些事情。

在 TPL 中，`Task`类是主要的操作类。`Task`是一个在需要时处理线程实例化的类。它做得多得多，但我们稍后再讨论。

我说“当需要时”，因为它是足够智能的，能够确定何时需要一个新的线程。

让我们从简单的例子开始，然后逐步深入：

```cs
Task myTask = Task.Run(() => { Console.WriteLine("Hello from the task."); });
Console.WriteLine("Main thread is done.");
Console.ReadKey();
```

`Task`只是另一个 C#类，它为我们处理了大部分并发。在这种情况下，我们调用`static method Run()`，它接受一个委托来执行。

我们可以将其重写如下：

```cs
Task myTask = Task.Run(DoWork);
Console.WriteLine("Main thread is done.");
Console.ReadKey();
return 0;
void DoWork()
{
    Console.WriteLine("Hello from the task.");
}
```

这个代码片段做的是同样的事情，但我们调用方法而不是使用 lambda 表达式。

我们可以用稍微不同的方式做到同样的事情：

```cs
Task myTask = new Task(DoWork);
myTask.Start();
```

我省略了`Console`相关的内容和实际的方法；它们将保持不变（直到我说我已经改变了它们）。

这段代码基本上与上一个示例做的是同样的事情。区别在于`Task`不会启动，除非我们明确调用`Start()`。

第二个示例为你提供了对`任务`更多的控制。你可以在启动任务之前设置属性和改变任务的行为。`Task.Run()`主要设计用于“发射后不管”的场景。`Start()`更加灵活；它允许我们改变调度，例如，指定它在一个特定的线程上运行。你也可以这样指定`Task`的优先级。

这个例子并不非常吸引人。让我们尝试让它变得更有趣。我们可以将我们的方法改为以下内容：

```cs
void DoWork(int id)
{
    Console.WriteLine($"call Id {id}.");
}
```

我们将在我们的方法中添加一个参数来识别调用者。由于我们现在有一个参数，我们必须也改变如何将这个参数传递给`Task`构造函数。让我们不要止步于此。想象一下，我们想要链式调用方法。在`Task`完成使用`Id 1`的`DoWork`之后，我们希望它再次调用那个方法，但这次使用`Id 2`。在现实生活中，你可能会链式调用两个完全不同的方法，但工作方式是相同的。

代码看起来是这样的：

```cs
Task myTask = new Task(() => DoWork(1));
myTask.ContinueWith((prevTask) => DoWork(2));
myTask.Start();
```

我们已经更改了构造函数中的参数，以便我们可以将那个`1`整数传递给方法。下一行更有趣。它说：“当你完成第一步后，再次调用`DoWork`，但这次使用`Id 2`。”`prevTask`参数是已经完成工作的前一个`Task`。这触发了第二个`Task`的开始。

如果你运行这个程序，你会看到控制台按正确顺序打印的行。

让我们重写一次被调用的方法：

```cs
void DoWork(int id)
{
    Console.WriteLine($"call Id {id}, " +
                      $"running on thread " +
                      $"{Thread.CurrentThread.ManagedThreadId}.");
}
```

我们将这个方法运行的线程的`id`添加到输出中。我还想在开始任务之前看到这个`id`线程。我们的调用代码现在看起来是这样的：

```cs
Console.WriteLine($"Our main thread id =
{Thread.CurrentThread.ManagedThreadId}.");
Task myTask = new Task(() => DoWork(1));
myTask.ContinueWith((prevTask) => DoWork(2));
myTask.Start();
```

如果你运行这个程序，你可能会看到任务在不同的线程上运行，而不是主线程。如果你重复几次，甚至可能发生第二个任务在第一个任务不同的线程上运行的情况。这种情况何时发生是不可预测的；调度器会根据当前条件选择最佳方案。我们不必担心这个问题。它只是正常工作。这不是很酷吗？

TPL 中另一个很棒的类是`Parallel`类。它允许我们并行执行操作。让我们看看这个：

```cs
Console.WriteLine($"Our main thread id =
{Thread.CurrentThread.ManagedThreadId}.");
int[] myIds = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
Parallel.ForEach(myIds, (i) => DoWork(i));
```

首先，我们将打印当前线程的 `id`。然后，我们将创建一个从 `1` 到 `10` 的整数数组；这里没有什么特别的。之后，我们将调用 `Parallel` 类的静态 `ForEach` 方法，并给它提供数组和要调用的 lambda 表达式。该方法并行地遍历数组，并使用正确的参数调用 lambda 表达式。它这样做的方式不是顺序的，而是与标准的 `ForEach` 循环不同。

当你运行这个程序时，你会看到一些令人兴奋的结果。程序打印 ID 的顺序完全是随机的。你会看到运行时使用了多个线程，但有时它会重用其中的一些线程。

再次强调，TPL 确定最佳做法，并为你处理所有线程的创建和调度。

TPL 非常强大。它也是 async/await 模式的支柱。这是一个简化并发工作的模式，以至于大多数用户都没有意识到幕后发生了什么。凭借你新获得的知识，你应该没有问题跟踪正在发生的事情。那么，让我们来看看 async/await。

# Async/await

软件几乎从不孤立运行。大多数软件需要在某个时刻超出其边界并访问代码块之外的东西。例如，读取和写入文件、从网络读取数据、向打印机发送数据等等。假设一台典型的机器可以在大约 10 纳秒内访问内存中的一个字节。从 SSD 读取相同的字节大约需要 1,000,000 纳秒或更长。从外部设备读取数据通常比从内存中读取本地数据慢 100,000 到 1,000,000 倍。当你尝试优化代码时，如果你知道你的软件在将数据传输到外部硬件和从外部硬件传输数据时，请考虑这一点。

让我们更进一步。让我们假设你有一台处理数据速度相当快的机器。你需要从外部网站读取数据。你的程序必须等待很长时间，数据才可用。数据到达我们这里可能需要毫秒。对我们这些普通人来说，这已经很快了，但计算机在这段时间内本可以做一百万件其他任务。这似乎是对我们昂贵的资源的巨大浪费，对吧？

线程当然可以帮助我们。你可以创建一个线程来调用外部网站，并等待它完成，同时做其他事情。然而，正如我们所看到的，线程可能会相当繁琐。TPL 有所帮助，但事情仍然可能变得复杂。从外部源读取数据或将数据写入外部目标如此常见，以至于 CLR 设计者决定通过引入 async/await 来帮助我们。

自顶向下的方法很简单：任何耗时超过简单操作的事情都应该异步执行。然而，我们不想直接处理线程本身。Async/await，它内部使用 TPL，是一种可以帮助我们的模式。

它所做的是这样：一旦你有需要异步运行的代码，编译器就会注入代码，将我们的代码包装到一个状态机中。这个状态机跟踪线程和我们的代码的进度，并在需要关注的代码块之间来回切换。

这听起来很复杂吗？嗯，确实是。然而，使用方法是直接的。然而，在我向你展示它之前，我想先介绍一个小小的辅助代码，我经常在讨论 async/await 时使用。这段代码只是对 `string` 类的一个扩展方法，并将一个 `string` 输出到控制台，并添加 `ManagedThreadId`。它甚至允许对输出进行着色，使得区分不同的线程更容易。如果你想使用这个，请随意。如果你宁愿在所有地方都使用 `Console.WriteLine()`，那也请随意。然而，使用这个可以使代码的关键部分更容易阅读。以下是我的扩展方法：

```cs
using static System.Threading.Thread;
namespace ExtensionLibrary;
public static class StringExtensions
{
    public static string Dump(this string message,         ConsoleColor printColor = ConsoleColor.Cyan)
    {
        var oldColor = Console.ForegroundColor;
        Console.ForegroundColor = printColor;
      Console.WriteLine($"({CurrentThread.ManagedThreadId})\t :         {message}");
        Console.ForegroundColor = oldColor;
        return message;
    }
}
```

你也可以在 GitHub 仓库中找到这段代码。

首先，我想给你展示一个最简单的例子：

```cs
using ExtensionLibrary;
DoWork();
// The program is paused until DoWork is finished.
// This is a waste of CPU!
"Just before calling the long-running DoWork()"
    .Dump(ConsoleColor.DarkBlue);
"Program has finished".Dump(ConsoleColor.DarkBlue);
Console.ReadKey();
void DoWork()
{
    "We are doing important stuff!".Dump(ConsoleColor.DarkYellow);
    // Do something useful, then wait a bit.
    Thread.Sleep(1000);
}
```

想象一下，我们想在 `DoWork()` 方法中做一件需要很长时间的事情，比如从存储中读取文件。我在这里通过暂停当前线程一秒钟来模拟这一点。当我们在这个主方法中调用它时，整个程序都会暂停。我们昂贵的强大 CPU 被闲置（至少不是为我们程序）。这似乎很浪费！我们已经看到我们可以使用线程或 TPL 来改进这一点。然而，这段代码也被包装在 async/await 模式之中，所以为什么不使用这个呢？

要做到这一点，我将 `Thread.Sleep()` 替换为对 `Task.Delay()` 的调用。这基本上做了同样的事情，但允许我们改进我们的代码。记住：这个 `Thread.Sleep()` 和新的 `Task.Delay()` 方法只是我们应用程序应该做的实际工作的替代品。在代码中有一个 `Sleep()` 或 `Delay()` 方法通常是一个坏主意。

如果你必须调用一个异步方法，你必须等待它。所以，我们在调用 `Task.Delay()` 之前添加了 `await` 关键字。

一旦我们完成了替换，我还会在方法前加上 `async` 关键字。这个关键字告诉编译器应该将这个方法包装在我之前提到的状态机中。然而，任何异步方法都不应该返回 void，原因将在稍后变得清楚。我们需要返回一个 `Task` 或 `Task<>`，如果你实际上返回了某些内容。所以，我们将我们的 void 改为 `Task`。同样，任何异步方法都需要用 `await` 关键字来调用。所以，结果看起来像这样：

```cs
using ExtensionLibrary;
"Just before calling the long-running DoWork()"
    .Dump(ConsoleColor.DarkBlue);
await DoWork();
// The program is no longer paused until DoWork is finished.
// This allows the CPU to keep working!
"Program has finished".Dump(ConsoleColor.DarkBlue);
Console.ReadKey();
async Task DoWork()
{
    "We are doing important stuff!".Dump(ConsoleColor.DarkYellow);
    // Do something useful, then wait a bit.
    await Task.Delay(1000);
}
```

运行这个看看会发生什么。

你可能会看到程序从一个线程开始，然后在同一个线程上执行 `DoWork()` 方法，但完成之后会切换到新的线程。这是因为编译器看到了我们的 `Task.Delay()` 等待操作，并决定释放 CPU 去做其他事情。运行时将我们的当前线程挂起，并将其状态存储在内存中，这样我们的主代码就可以自由地做其他事情。只有当 `Task.Delay()` 完成后，我们的主线程才会被恢复。然而，由于主线程不再与我们的代码相关联，我们需要一个新的线程。这个线程是从 `ThreadPool` 中拉取的（记住：那里很快，因为线程是在启动时创建的），并填充了我们之前的状态。然后系统可以继续在这个线程上运行。程序也是在那个新线程上结束的！

我提到所有异步方法都需要 `async` 修饰符，并应该返回一个 `Task` 而不是 `void`。这样做有一个简单的理由。如果你不这样做，你的代码会工作，但不会像预期的那样。**异步一直到底**规则很简单，但非常重要。

异步一直到底！

如果你有一个包含 `await` 关键字的方法，那么这个方法必须是异步的，并返回一个 `Task`。然而，由于你可能会在某个地方自己调用这个方法，所以调用代码也必须是异步的，并返回某种形式的 `Task` 或 `Task<>`。因为这个方法也会被调用……嗯，你明白这个意思。规则是：异步一直到底！链中的每个方法都需要有异步！

另一条规则，它不像“异步一直到底”规则那样严格，是所有异步方法都应该这样命名。我们的 `DoWork()` 方法应该重命名为 `DoWorkAsync()`。

然而，在我们这样做之前，让我们看看如果我们粗心大意，没有返回一个 `Task` 会发生什么。试试看：将 `Task` 返回类型替换为 `void`，并在 `DoWork()` 之前移除 `await`（你不能等待 `void`，所以如果你不移除它，你会得到一个错误）。

运行它。它运行得很好，对吧？好吧，没有创建新的线程，但谁在乎呢？软件做了它需要做的事情。

现在，让我们稍微修改一下我们的 `DoWork()` 方法：

```cs
using ExtensionLibrary;
"Just before calling the long-running DoWork()"
    .Dump(ConsoleColor.DarkBlue);
DoWork();
"Program has finished".Dump(ConsoleColor.DarkBlue);
//Console.ReadKey();
async void DoWork()
{
    "We are doing important stuff!"
        .Dump(ConsoleColor.DarkYellow);
    await Task.Delay(1000);
    throw new Exception(
        "Something went terribly wrong."
    );
    "We're done with the hard work."
        .Dump(ConsoleColor.DarkYellow);
}
```

我也暂时移除了 `ReadLine()`，使程序更贴近现实。主线程在一切完成后结束。

运行它。看到我们没有得到 “我们已经完成了艰苦的工作” 的消息。这是有道理的；它前面有一个异常。然而，请注意，我们也没有看到那个异常。

为什么会这样？这很复杂，但简化的解释是，由于`DoWork`仍然是一个异步方法，状态机仍然被创建。异常是在不同的线程上抛出的（在`Task.Delay()`等待之后）。然而，由于状态机没有配置为等待所有结果（因为我们省略了`await`关键字），它只是忽略了那个线程。如果你将那个“我们已经完成了艰苦的工作”的`Dump()`行移动到异常之前的行，你会看到它没有被调用。实际上，它*确实*被调用了；你只是没有看到它。这个线程已经变成了一个“发射并遗忘”的线程。你失去了对它的所有控制。

你能想象一个复杂的软件，其中在代码深处出了问题吗？你能想象没有获取到异常吗？你能想象调试那个的恐怖吗？

如果你一路使用 async/await，你会得到那个异常。

哦，在我忘记之前：我移除`Console.ReadKey()`行的原因是我这样做迫使主线程尽快退出，从而卸载应用程序从内存中。如果你恢复那行代码，你会看到异常，因为主线程在那里暂停。现在其他事情将被允许发生。

然而，这并不是我们问题的真正解决方案。你不想在获取异常之前等待主线程空闲。这可能需要很长时间才能发生。

请恢复 async/await 关键字，将`DoWork()`中的 void 替换为`Task`，然后运行它。异常正是在你预期的地方抛出的。

这真的很重要，所以我喜欢重复一遍：从上到下都是异步的！

## Task.Wait()和 Task.Result

关于为什么不应该使用`Task.Wait()`或`Task.Result`有很多博客文章和文章。这个原因很简单：这些调用会阻塞当前线程。使用它们会移除调度器在`Task`完成时恢复调用线程工作并返回执行流程的能力。如果你这样做，为什么还要使用 async/await 呢？Async/await 还允许线程同步，因此不需要使用`Wait()`和`Result`。

等一下。有些情况下，你可能会决定仍然使用它们：

+   如果你正在对正在现代化的遗留代码进行工作，你可能想使用它们。规则是“从上到下都是异步的”，这可能需要大量的代码重构。这并不总是可行的。在这些情况下，你可能使用`Wait()`和`Result`代替。

+   在单元测试中，你可以模拟或存根异步方法。然而，有时单元测试使用`Wait()`和`Result`可能更好。

+   在系统编程中，你可能不关心主线程保持响应。毕竟，没有用户界面。所以，阻塞主线程可能不是一个大问题。我仍然认为不使用 async/await 是不好的做法，但在这种情况下，你可以用`Wait()`和`Result`来解决问题。

就像软件开发中的所有规则一样，对这些规则保持警惕，尽可能多地应用它们，只有在你有充分的理由并且深思熟虑之后才打破这些规则。此外，请为你未来的自己行个方便，在源代码中记录你选择偏离常规工作方式的原因。

因此，现在你知道如何使用 async/await。尽管它们并不总是导致多线程，但它们是平衡应用程序负载的绝佳方式。它们在保持代码组织方面大有裨益。你无需再承担在线程之间进行所有同步的负担。然而，这并不意味着你永远不需要关心同步。这是不可避免的，所以我认为我们现在应该讨论一下。

# 线程同步

async/await 模式让我们的开发者生活变得更加容易。如果你有一个长时间运行的任务（记住：任何使用 CPU 之外的设备的事情都是长时间运行的），你可以异步调用该方法。然后你可以坐下来等待它完成，而不会阻塞应用程序其他地方的执行。TPL 负责线程管理。

然而，有时你可能想要有更多的控制权。你可能遇到这样的情况，你必须等待一个方法完成才能继续。想象一下，你有一个主线程并调用 `A()` 方法。该方法运行时间较长，因此你将其改为异步（将其重命名为以“async”结尾的名称）并更改返回类型为 `Task` 或 `Task<>`。现在你可以等待它。然而，另一个线程可能必须等待你的 `Aasync()` 方法完成。你如何做到这一点？

欢迎来到线程同步的奇妙世界。

## 同步——我们如何做到这一点？

在过去，当我们仍然使用线程和 `ThreadPool` 时，同步可能会很麻烦。然而，随着 `Task` 和 async/await 的出现，事情变得容易多了，而且没有真正的缺点。在我向你展示这一点之前，我想先展示如何同步线程而不是任务。

让我从基础程序开始：

```cs
using ExtensionLibrary;
"In the main part of the app.".Dump(ConsoleColor.White);
ThreadPool.QueueUserWorkItem(DoSomethingForTwoSeconds);
ThreadPool.QueueUserWorkItem(DoSomethingForOneSecond);
"Main app is done.\nPress any key to
stop.".Dump(ConsoleColor.White);
Console.ReadKey();
return 0;
void DoSomethingForOneSecond(object? notUsed)
{
    $"Doing something for one second.".Dump(ConsoleColor.Yellow);
    Thread.Sleep(1000);
    $"Finished something for one second".Dump(ConsoleColor.Yellow);
}
void DoSomethingForTwoSeconds(object? notUsed)
{
    "Doing something for two
        seconds.".Dump(ConsoleColor.DarkYellow);
    Thread.Sleep(2900);
    "Done doing something for two
        seconds.".Dump(ConsoleColor.DarkYellow)
}
```

这个示例现在应该很清楚。我有两个方法，它们执行一些需要很长时间才能完成的事情。我从 `ThreadPool` 中拉出一些线程，并使所有这些同时运行。

如果你运行这个，在 `Console.ReadKey()` 位置。如果我们想在继续之前等待两个方法完成，我们能做什么？

答案是使用同步机制。这意味着我们有一个对象，我们可以用它来标记某些状态。我们可以自己编写它，但我们必须注意很多同步和线程安全问题。幸运的是，我们不必这样做。Win32 API 提供了一些工具，它们被巧妙地封装在 BCL 类中。

其中之一是 `CountdownEvent` 类。正如其名所示，它允许我们进行事件倒计时。

将你的主方法修改如下：

```cs
"In the main part of the app.".Dump(ConsoleColor.White);
// Tell the system we want to wait for 2 threads to finish.
CountdownEvent countdown = new(2);
ThreadPool.QueueUserWorkItem(DoSomethingForOneSecond);
ThreadPool.QueueUserWorkItem(DoSomethingForTwoSeconds);
// Do the actual waiting.
countdown.Wait();
"Main app is done.\nPress any key to stop.".Dump(ConsoleColor.White);
Console.ReadKey();
return 0;
```

我们将创建一个 `CountdownEvent` 类的新实例并将其初始化为 `2`。

然后，我们将获取线程并允许它们完成工作。

在方法中的代码，我添加了一行：

```cs
void DoSomethingForOneSecond(object? notUsed)
{
    $"Doing something for one second.".Dump(ConsoleColor.Yellow);
    Thread.Sleep(1000);
    $"Finished something for one second".Dump(ConsoleColor.Yellow);
    countdown.Signal();
}
```

在方法底部，你会看到 `.Signal()` 倒计时。由于这个实例在这个方法中是可访问的，我可以使用它。`Signal()` 告诉倒计时减少等待事件的数量。

我对 `DoSomethingForTwoSeconds()` 方法也做了同样的处理。

这意味着当两个方法都完成后，它们会在倒计时上调用 `Signal()`。在主方法中，我在 `ThreadPool` 代码之后添加了 `countdown.Wait()`，告诉主线程暂停，直到倒计时达到零。

如果你运行这个程序，你会看到它运行得非常出色，并且主线程的其他部分与线程完美同步。

然而，如果我想在 `DoSomethingForOneSecond` 完成后启动 `DoSomethingForTwoSeconds` 方法呢？

这几乎同样简单。我们可以使用其他同步类之一来帮助我们。让我展示如何使用 `ManualResetEvent` 来完成这个操作。这个类或多或少与 `CountdownEvent` 类似。区别在于 `ManualResetEvent` 类不计数，它只是等待信号。

在主方法中，在调用 `ThreadPool` 之前，我添加了这一行：

```cs
ManualResetEvent mre = new(false);
```

我将其设置为初始的 `False` 状态。这样做会导致任何线程等待事件被设置。

在 `DoSomethingForOneSecond()` 中，我在最后添加了一行：

```cs
// Tell the second thread it can start
mre.Set();
```

对 `Set` 的调用告诉 `ManualResetEvent` 任何等待的线程都可以继续。

在 `DoSomethingForTwoSeconds()` 中，我在方法的开头添加了以下内容：

```cs
// Wait for the first thread to finish.
mre.WaitOne();
```

`WaitOne()` 告诉代码暂停线程，直到 `mre` 收到信号（这发生在 `DoSomethingForOneSecond()` 的末尾）。

如果你现在运行你的程序，你会注意到一切都很完美地同步，等待其他任务完成。

当然，你可能通过不使用线程就达到了完全相同的结果。我们基本上从我们的应用程序中移除了所有多任务。如果你需要同步线程，现在你知道如何做了。然而，要小心：如果你出错，可能会引入奇怪的错误。相信我：调试多线程应用程序可不是件轻松的事。

## 使用 async/await 进行同步

到现在为止，你可能已经猜到了，使用 async/await 可以显著降低处理线程和它们之间同步的复杂性。

让我们回到 `DoSomethingForOneSecond` 和 `DoSomethingForTwoSeconds` 方法的例子。这次，我们将重新编写它们以使用 async/await。

你的 `DoSomethingForOneSecond` 应该是这样的：

```cs
async Task DoSomethingForOneSecondAsync()
{
    $"Doing something for one second.".Dump(ConsoleColor.Yellow);
    await Task.Delay(1000);
    $"Finished something for one second".Dump(ConsoleColor.Yellow);
}
```

我将函数重命名，以异步结尾，因为我应该早就这样做。

`DoSomethingForTwoSecondsAsync()` 应该得到同样的处理。

在主方法中调用这些方法现在看起来是这样的：

```cs
"In the main part of the app.".Dump(ConsoleColor.White);
await DoSomethingForOneSecondAsync();
await DoSomethingForTwoSecondsAsync();
"Main app is done.\nPress any key to stop.".Dump(ConsoleColor.White);
Console.ReadKey();
return 0;
```

结果与我们在自己进行所有同步的示例中得到的相同，唯一的区别是我们不再有阻塞的线程。所以这不仅更容易做，而且也更好。

然而，如果我们不想按顺序执行这些方法，而是想让它们同时运行会怎样？如果我们想让它们同时运行会怎样？

好吧，这很简单。由于我们的方法返回一个 `Task`，我们可以处理它。我们不是逐个等待它们，而是可以同时等待它们。让我给你展示一下：

```cs
var task1 = DoSomethingForOneSecondAsync();
var task2 = DoSomethingForTwoSecondsAsync();
// Wait for all tasks to be finished
Task.WaitAll(task1, task2);
"Main app is done.\nPress any key to stop.".Dump(ConsoleColor.White);
Console.ReadKey();
return 0;
```

我们将利用我们获取任务的事实。`Task` 类有一个名为 `WaitAll()` 的静态方法，它只有在所有任务都完成时才返回。

还有其他方法，例如 `WaitAny`（只有当任何任务完成时才继续），`WhenAll`（当它们都完成时执行某些操作），以及 `WhenAny`（你可以自己弄清楚）。

`WaitAll` 或 `WaitAny` 与 `WhenAll` 或 `WhenAny` 之间的区别在于 `WaitXXX` 是一个阻塞调用。它会阻塞当前线程，直到条件满足。`WhenXXX` 返回一个 Task 本身，你可以 await 它，因此不会阻塞线程。

然而，这里有一个更大的区别：`WhenAll` 允许你捕获返回的结果。如果你想要等待完成的任何任务返回一个结果，你可以通过 `WhenAll` 获取它。`WhenAll` 将结果以数组的形式返回给你。你可以访问它们，这是 `WaitAll` 或 `WaitAny` 无法做到的。

如果你有所怀疑，`WhenAny` 返回一个 `Task<T>`。这个 `Task<T>` 有一个名为 `Result` 的属性；你可以读取这个属性来获取对那个 Task 结果的访问。这是一个使用 `Result` 实际上是一件好事的例子！

## 取消线程

有时候，你可能想要停止一个线程的运行。这样做可能有几个很好的理由，但无论你的理由是什么，一定要清理自己的“战场”。线程使用起来成本很高，将它们留在未知状态是一种糟糕的做法：你有一天会自食其果。

在 .NET Framework 的日子里，`Thread` 类有一个名为 `Abort()` 的方法。然而，结果证明这个方法弊大于利，因此 BCL 和 CLR 的人决定废除它。如果你尝试中止一个线程，你会得到一个 `PlatformNotSupportedException`。我想他们真的不希望我们再使用它了。

停止正在运行的线程的最佳方式与停止正在运行的 `Task` 的方式相同：使用我们称之为**协作取消**的东西。调用线程可以请求另一个线程停止。这取决于第二个线程是否遵守那个请求——或者不遵守。没有保证。

做这件事的标准方式是使用一个 `CancellationToken`。`CancellationToken` 是我们用来表示我们想要取消某事的信号的对象。

当然，你可以自己编写这个类。除了线程安全之外，没有太多的事情发生。然而，在你的线程或任务中包含一个 `CancellationToken` 可以清楚地让用户知道它可以被取消。

我将稍微重写我们的 `DoSomethingForOneSecondAsyncMethod()`：

```cs
async Task DoSomethingForOneSecondAsync()
{
    $"Doing something for one second.".Dump(ConsoleColor.Yellow);
    for(int i=0;i<1000;i++)
        await Task.Delay(1);
    $"Finished something for one second".Dump(ConsoleColor.Yellow);
}
```

代替使用`Task.Delay(1000)`调用，我使用`1000` await `Task.Delay1)`。从理论上讲，这将导致一秒的延迟。然而，当你运行这个时，它实际上会花费更长的时间。await 调用本身也会占用一些时间。

我可以测量它花费的时间然后重新计算迭代次数，或者我可以简单地将方法重命名为`DoSomethingForSomeUnderterminedAmountOfTimeAsync()`。我将把这个决定留给你。

假设我们在主方法中等待了 500 毫秒后感到无聊，并决定停止这个线程。我们该如何实现这一点？

这就是`CancellationToken`发挥作用的地方。再次强调，`CancellationToken`是一个简单的类。如果你想创建一个，当然可以，但最好使用一个专门的类。这个`CancellationTokenSource`类就是为此而创建的，并且能在各种奇怪的情况下工作。它是线程安全的。

让我们在主方法的开始处创建一个：

```cs
using ExtensionLibrary;
"In the main part of the app.".Dump(ConsoleColor.White);
using var cts = new CancellationTokenSource();
var task1 = DoSomethingForOneSecondAsync();
Task.WaitAny(task1);
"Main app is done.\nPress any key to stop.".Dump(ConsoleColor.White);
Console.ReadKey();
return 0;
```

我在这里使用`WaitAny`是因为我们想在创建任务后和它完成前取消该任务。同时，注意`using`语句。`CancellationTokenSource`实现了`IDisposable`接口，因此我们必须遵守这一点。

取消很简单。在`var task1`和`Task.WaitAny()`行之间，添加以下内容：

```cs
await Task.Delay(500);
"We got bored. Let's cancel.".Dump(ConsoleColor.White);
cts.Cancel();
```

我们会等待一会儿，然后感到无聊并调用`cts.Cancel()`。

然而，如果你运行这个，什么都不会发生。这并不完全正确；会有很多事情发生。更准确地说，`DoSomethingForOneSecondAsync`中的整个循环都会发生。

`CancellationToken`并不是取消正在运行的任务的魔法方法。你必须自己检查这个令牌。

我们必须给我们的方法添加一个`CancellationToken`类型的参数。现在方法签名将看起来像这样：

```cs
async Task DoSomethingForOneSecondAsync(CancellationToken cancellationToken)
```

我们在调用它时必须传入这个令牌。在我们的主方法中，将调用此方法的行更改为以下内容：

```cs
var task1 = DoSomethingForOneSecondAsync(cts.Token);
```

我们将获取我们的`CancellationTokenSource`并获取它的令牌。这就是我们将传递给我们的方法的。

在我们的方法内部，我们必须检查是否需要取消。是的，这就是为什么我在`Delay`周围添加了循环。现在完整的方法将看起来像这样：

```cs
async Task DoSomethingForOneSecondAsync(CancellationToken cancellationToken)
{
    $"Doing something for one second.".Dump(ConsoleColor.Yellow);
    bool hasBeenCancelled = false;
    int i = 0;
    for (i = 0; i < 1000; i++)
    {
        if (cancellationToken.IsCancellationRequested)
        {
            hasBeenCancelled = true;
            break;
        }
        await Task.Delay(1);
    }
    if(hasBeenCancelled)
    {
        $"We got interrupted after {i} iterations.".Dump(ConsoleColor.            Yellow);
    }
    else
    {
        $"Finished something for one second".Dump(ConsoleColor.            Yellow);
    }
}
```

如果有人调用`CancellationTokenSource`的`Cancel`，令牌上的`IsCancellationRequested`标志将被设置为`True`。我们必须尊重这一点。我通过跳出`for`循环来实现这一点。我还设置了一个`hasBeenCancelled`变量，这样我就可以通知我们的用户我们已经取消了循环，并告诉他们是在多少次迭代后取消的。

我们本可以跳过这个布尔值并再次使用`IsCancellationRequested`。然而，可能存在这样的风险：请求是在循环完成后、打印消息之前到达的。在这种情况下，循环没有被中断。但我们还是说它被中断了，这是不正确的。这样我们就可以避免打印错误的消息。

运行它并看看会发生什么。在我的机器上，它大约迭代了 40 次后就会取消。

这段代码中有一个错误。将`CancellationToken`传递给任何接受它的方法是一种良好的实践。在我们的例子中，那就是`Task.Delay()`。它有一个接受`CancellationToken`的重载。

我故意在这里省略了它。因为代码几乎 100%的时间都会在那个行上，等待 Delay，我们会取消它，永远不会看到任何打印的消息。然而，现在让我们添加它：

```cs
await Task.Delay(1, cancellationToken);
```

重新运行它并看看会发生什么。

你可能会注意到我们缺少了很多屏幕输出。原因很简单。`Task.Delay()`在取消时抛出`OperationCancelledException`。然而，我们在主方法中没有在`Task`上使用`await`，所以我们会错过这个异常。记得我之前说过，当一切没有做对的时候，错过异常是多么容易吗？

同步有助于防止错误发生。然而，有许多技术可以确保我们的代码是线程安全的。现在让我们深入探讨这些技术。

# 线程安全编程技术

看看这段代码。运行它并看看会发生什么：

```cs
using ExtensionLibrary;
int iterationCount = 100;
ThreadPool.QueueUserWorkItem(async (state) =>
{
    await Task.Delay(500);
    iterationCount = 0;
    $"We are stopping it...".Dump(ConsoleColor.Red);
});
await WaitAWhile();
$"In the main part of the app.".Dump(ConsoleColor.White);
"Main app is done.\nPress any key to stop.".Dump(ConsoleColor.White);
Console.ReadKey();
return 0;
async Task WaitAWhile()
{
    do
    {
        $"In the loop at iterations {iterationCount}".            Dump(ConsoleColor.Yellow);
        await Task.Delay(1);
    }while (--iterationCount > 0) ;
}
```

我们有一个从 100 倒数到 0 的`Task`。由于我们在这里`await`它，代码的主要部分会优雅地等待这个操作完成后再继续。然而，我们还有一个等待 500 毫秒后设置计数器为`0`的第二个线程。结果是循环提前结束。

我们看到的是容易调试的。每一行代码都在一个屏幕上，所以我猜想你将能够很容易地找到这个错误。

然而，如果这里使用的整数是类的一个成员呢？正如你所知，类的实例是引用类型。引用类型是通过引用传递的，而不是通过值。所以如果 Task 可以访问那个实例，它可以改变那个实例的成员。然而，其他任何任务、线程或代码都会看到这些变化的效果。

线程安全就是避免这些情况。

值类型本质上是安全的。如果你将一个整数值类型等值类型传递给`Task`，你将不会有任何问题。整数的值被复制，你并没有改变原始值。

然而，如果你需要访问更复杂的数据类型，你需要考虑这一点。好消息是运行时为我们提供了几个工具来减轻这个问题。

你得到的一个工具是`Lock()`关键字。

## Lock()

保护你的数据最简单的方法是在它周围使用锁。锁是一个对象，它在一定程度上就像是一段代码的护城河。锁确保只有一个线程可以同时进入那个代码块。语法很简单：

```cs
lock (new object())
{
    iterationCount--;
}
```

锁接受一个参数。它使用这个参数来识别要锁定的区域。它对这个对象不做任何事情；这只是为了将锁钩到某个地方。所以，有一个新的`object()`就足够了。

代码块中的任何代码都是安全的，这意味着只有一个线程可以同时递减`iterationCount`。当另一个线程尝试同时执行相同的操作时，它会在到达锁语句时立即阻塞。那个线程会一直阻塞，直到之前的线程退出代码块。

是的，这意味着如果其他线程在那个代码中崩溃（在这个例子中不太可能：在`--`运算符上崩溃非常罕见），整个系统永远无法进入那个代码块。

`Lock()`是围绕监视器对象的一种语法糖。编译器实际上使用`Monitor`s。所以，以下代码会产生相同的中间语言（IL）：

```cs
var lockObject = new object();
Monitor.Enter(lockObject);
try
{
    iterationCount--;
}
finally
{
    Monitor.Exit(lockObject);
    lockObject = null;
}
```

我不知道你，但对我来说，`lock()`语句看起来要轻松得多。

## 记录

确保数据不会意外覆盖的最佳方式是确保数据不能被更改。不可变类型正是为此而设计的。

让我先创建一个记录：

```cs
record Counter(int InitialValue);
```

记录是一种引用类型，所以它的内存是在堆上分配的。然而，记录旨在是不可变的。你可以创建不可变的记录，但这在这里没有帮助。

目前，我有一个只有一个成员`InitialValue`的记录。在构建`Counter`时，我必须设置该值，但之后我永远不能更改它。所以，没有线程可以再来修改这个值了。

然而，由于我无法在任何地方更改它，我也必须在`Task`中的代码进行更改。现在它看起来是这样的：

```cs
async Task WaitAWhile()
{
    var actualCounter = myCounter.InitialValue;
    do
    {
        $"In the loop at iterations {actualCounter}".            Dump(ConsoleColor.Yellow);
        await Task.Delay(1);
    } while (--actualCounter > 0);
}
```

我已经复制了值并在循环中递减它。如果你和我有点相似，你可能会说，“等等。为什么我没有只是将原始的`iterationCount`复制到一个局部变量中，并使用它而不是这个记录？”

我看到很多人这样做。然而，这并不保证能行得通。如果你在复制之前另一个线程改变了`iterationCount`的值怎么办？你将从一个错误的初始值开始。

不可变记录保证其内部值永远不会改变，永远如此。你很安全。

## 避免使用静态成员和类

我知道创建类的实例可能会很麻烦。有时，似乎更容易创建一个静态类，其中包含静态成员，并使用它们代替。它们确实有一些用例。然而，请记住这一点：静态类默认不是线程安全的。静态成员在多个线程之间共享，因此任何人都可以更改它们。

## 使用`volatile`关键字

有时，代码看起来很简单，但可能并非如此。看看这一行：

```cs
int a=42;
```

我们知道这是如何工作的。这个整数位于栈上。如果我们更改其值，该内存地址的值也会改变。简单，对吧？错了。编译器会使用各种技巧来优化我们的代码，尤其是在你以发布模式构建时。以发布模式构建意味着编译器可能会缓存简单整数的值以加快速度。它甚至可能决定将这一行移动到代码的另一个位置，如果它认为这不会影响执行。

这不是问题，直到多个线程或任务处理这段代码。编译器可能会出错。它无法确定哪些任务可以同时访问该变量。

是的，即使在多线程系统中简单地写入一个整数也可能出错。

如果我们使用`lock()`，我们可以保证只有一个线程可以访问那个代码块，但这仍然不能减轻编译器优化的影响。

为了解决这个问题，我们可以使用`volatile`关键字。它看起来是这样的：

```cs
private static volatile int _initialValue = 100;
```

与使用缓存值不同，编译器确保我们始终直接访问内存地址和存储的值。这意味着所有线程都将前往同一位置并使用相同的整数，从而消除了在旧、过时或缓存数据上工作的风险。

你可能会想将`volatile`关键字添加到每个地方，但我建议你避免这样做。它会干扰编译器的优化技术。你应该只在怀疑特定代码片段可能存在问题时使用它。

因此，现在你知道如何在与线程打交道时更加安全。这非常重要：如果你搞砸了，你可能会在代码中遇到可怕且难以调试的错误。这尤其适用于你在多个线程中处理集合的情况。你是如何保持它们同步的？幸运的是，BCL 已经为我们解决了这个问题。让我们来谈谈并发集合。

# .NET 中的并发集合

集合是许多程序的核心。数组、列表、字典——我们经常使用它们。然而，它们是线程安全的吗？让我们来了解一下：

```cs
using ExtensionLibrary;
var allLines = new List<string>();
for(int i = 0; i < 1000; i++)
{
    allLines.Add($"Line {i:000}");
}
ThreadPool.QueueUserWorkItem((_) =>
{
    Thread.Sleep(1000);
    allLines.Clear();
});
await DumpArray(allLines);
"Main app is done.\nPress any key to stop.".Dump(ConsoleColor.White);
Console.ReadKey();
return 0;
async Task DumpArray(List<string> someData)
{
    foreach(var data in someData)
    {
        data.Dump(ConsoleColor.Yellow);
        await Task.Delay(100);
    }
}
```

我们有一个`List<string>`。然后我们向该列表中添加了 1000 个字符串。我们有一个任务会遍历它们，将它们显示在屏幕上，并稍作等待。

我们还有一个单独的线程在等待一秒后清除列表。

如果你已经阅读了本章前面的部分，你可能会预期任务中的循环会提前终止。它不应该打印所有项目，因为列表突然为空，因此`ForEach()`停止。

然而，如果你运行它，你会看到不同的结果。你会得到一个友好的`InvalidOperationException`，告诉你集合已被修改，这破坏了`ForEach`代码。

BCL 中的集合不是线程安全的。如果一个线程正在处理它们，而另一个线程决定需要处理那个集合，事情就会出错。

以下集合不是线程安全的，在处理任务时应避免使用：

+   `List<T>`

+   `Dictionary<TKey` 和 `TValue>`

+   `Queue<T>`

+   `Stack<T>`

+   `HashSet<T>`

+   `ArrayList`

+   `HashTable`

+   `SortedList<TKey`, `TValue>`, 和 `TSortedList`

不要在多个线程或任务中同时使用这些。

一些集合是线程安全的。以下是它们的用途：

| **集合名称** | **描述** |
| --- | --- |
| `ConcurrentDictionary<TKey, TValue>` | 一个线程安全的键值对集合。它允许并发添加、更新和删除。 |
| `ConcurrentQueue<T>` | 一个线程安全的**先进先出**（FIFO）集合版本。 |
| `ConcurrentStack<T>` | 一个线程安全的**后进先出**（LIFO）集合版本。 |
| `ConcurrentBag<T>` | 一个线程安全、无序的对象集合。适用于顺序不重要的场景。 |
| `BlockingCollection<T>` | 表示一个可以限制大小的线程安全集合。它提供阻塞和非阻塞的`add`和`take`操作。 |

表 4.3：线程安全集合

前四个集合只是我们已知的集合的线程安全版本。然而，大多数人可能不会认出最后一个：`BlockingCollection<T>`集合。

这个集合首先是一个线程安全的集合。它还允许阻塞。让我给你举个例子：

```cs
using ExtensionLibrary;
using System.Collections.Concurrent;
// We have a collection that blocks as soon as
// 5 items have been added. Before this thread
// can continue, one has to be taken away first.
var allLines = new BlockingCollection<string>(boundedCapacity:5);
ThreadPool.QueueUserWorkItem((_) => {
    for (int i = 0; i < 10; i++)
    {
        allLines.Add($"Line {i:000}");
        Thread.Sleep(1000);
    }
    allLines.CompleteAdding();
});
// Give the first thread some time to add items before
// we take them away again.
Thread.Sleep(6000);
// Read all items by taking them away
ThreadPool.QueueUserWorkItem((_) => {
    while (!allLines.IsCompleted)
    {
        try
        {
            var item = allLines.Take();
            item.Dump(ConsoleColor.Yellow);
            Thread.Sleep(10);
        }
        catch (InvalidOperationException)
        {
            // This can happen if
            // CompleteAdding has been called
            // but the collection is already empty
            // in our case: this thread finished before the
            // first one
        }
    }
});
"Main app is done.\nPress any key to stop.".Dump(ConsoleColor.White);
Console.ReadKey();
return 0;
```

这里发生了很多事情，让我带你了解一下。

首先，我们创建了一个`BlockingCollection`的实例。这个类有一个很好的重载构造函数，它只允许添加这个数量的项目。如果有更多，则阻塞线程。我这里不需要这个功能，但我发现添加它很有趣。

然后我们启动了一个新线程，向这个集合添加项目。我们可以尝试添加 10 个，但同样，它只允许五个项目。所以，当第五个项目被添加时，这个线程会阻塞，直到我们移除其中一个项目。

在循环结束时，我们告诉集合我们没有剩下要添加的内容。我们通过调用`CompleteAdding()`来完成这个操作。

在我们读取第二个线程中的数据之前，我们等待了几秒钟，以便第一个线程有时间填充集合。

第二个线程（如果你也计算主线程的话是第三个）从这个集合中取出了一个项目。它是一个先进先出（FIFO）的集合，所以我们可以取出的第一个项目是列表中第一个添加的项目。我们展示了我们取出的内容，并等待了一段时间。我们需要捕获`InvalidOperationException`。如果在我们已经从集合中取出所有项目之后，由于时间原因调用了`CompleteAdding`，将会发生异常。我们需要捕获这个异常。

由于我们的时间安排和`Thread.Sleep()`调用，我们将看到一个迷人的效果。第一个线程用五个项目填充集合。然后它等待。这个操作总共需要五秒钟。程序开始后的六秒钟，我们将开始取项目。由于项目很多（确切地说有五个），程序将快速将这些项目打印到屏幕上。当我们取一个项目时，第一个线程获得添加新项目的权限。然而，由于添加一个项目需要一秒钟，第二个线程必须等待直到项目被添加。如果还没有可取的项目，`Take()`也会阻塞。

只有当第一个线程调用`CompleteAdding()`方法时，第二个线程才知道它已经完成（因为我们检查了`IsCompleted`属性）。然后，我们可以退出线程。

在幕后有许多同步操作，但它工作得非常出色。这无疑是您工具箱中的一个优秀补充！

# 下一步

这真是一次相当刺激的经历。线程编程可能很复杂，但我们成功地完成了所有这些。

在本章中，我们探讨了众多不同的事情。我们描述了多任务是什么，从老式的中断请求（IRQs）开始，经过协作多任务，最终到达现代的抢占式多任务。

然后，我们研究了 Win32 线程及其.NET 对应物。我们看到了如何创建线程，但很快发现`Threadpool`在大多数情况下提供了更好的方法。然而，我们了解到大多数这些都是多余的，因为 TPL 为我们处理了很多细节。

特别是，我们了解到 async/await 隐藏了很多复杂性，使得编写多线程代码变得轻而易举。就像所有工具一样，我们也了解到 async/await 伴随着风险。您必须知道会发生什么，以及坏事情可能发生在哪里。幸运的是，我们也涵盖了那些情况。

我们探讨了集合以及如何使您的代码线程安全。我们还学习了一些关于 async/await 的基本知识：从底层到顶层的 async！

异步编程在处理 CPU 外部的设备时是必不可少的。我们需要广泛使用这些技术的领域之一是文件系统。然而，文件系统还有很多其他您需要了解的事情。所以，下一章处理这个主题真是太好了！
