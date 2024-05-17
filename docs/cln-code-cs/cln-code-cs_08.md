# 第八章：线程和并发

进程本质上是在操作系统上执行的程序。这个进程由多个执行线程组成。执行线程是由进程发出的一组命令。能够同时执行多个线程的能力称为**多线程**。在本章中，我们将研究多线程和并发。

多个线程被分配一定的执行时间，并且每个线程都由线程调度程序按轮换方式执行。线程调度程序使用一种称为**时间片分配**的技术来调度线程，然后在预定的时间将每个线程传递给 CPU 执行。

**并发**是能够同时运行多个线程的能力。这可以在具有多个处理器核心的计算机上实现。计算机的处理器核心越多，就可以同时执行更多的执行线程。

当我们在本章中研究并发和线程时，我们将遇到阻塞、死锁和竞争条件的问题。您将看到我们如何使用清晰的编码技术来克服这些问题。

在本章中，我们将涵盖以下每个主题：

+   理解线程生命周期

+   添加线程参数

+   使用线程池

+   使用互斥对象与同步线程

+   使用信号量处理并行线程

+   限制线程池中的处理器和线程数量

+   防止死锁

+   防止竞争条件

+   理解静态构造函数和方法

+   可变性、不可变性和线程安全

+   同步方法依赖

+   使用`Interlocked`类进行简单状态更改

+   一般建议

通过学习本章并发编程技能，您将获得以下技能：

+   理解和讨论线程生命周期的能力

+   理解和使用前台和后台线程的能力

+   通过线程池限制线程数量和设置并发使用的处理器数量的能力

+   理解静态构造函数和方法对多线程和并发的影响

+   考虑可变性和不可变性及其对线程安全的影响的能力

+   理解竞争条件的原因以及如何避免它们的能力

+   理解死锁的原因以及如何避免它们的能力

+   使用`Interlocked`类执行简单的状态更改的能力

要运行本章中的代码，您需要一个.NET Framework 控制台应用程序。除非另有说明，所有代码将放在`Program`类中。

# 理解线程生命周期

C#中的线程有一个相关的生命周期。线程的生命周期如下：

![](img/9f574fc2-1ade-4f26-9ecf-52f3cecde092.png)

当线程启动时，它进入**运行**状态。在运行时，线程可以进入**等待**、**睡眠**、**加入**、**停止**或**挂起**状态。线程也可以被中止。中止的线程进入停止状态。您可以通过分别调用`Suspend()`和`Resume()`方法来挂起和恢复线程。

当调用`Monitor.Wait(object obj)`方法时，线程将进入等待状态。然后当调用`Monitor.Pulse(object obj)`方法时，线程将继续。通过调用`Thread.Sleep(int millisecondsTimeout)`方法，线程进入睡眠模式。一旦经过了经过的时间，线程就会返回到运行状态。

`Thread.Join()`方法使线程进入等待状态。加入的线程将保持在等待状态，直到所有依赖线程都完成运行，然后它将进入运行状态。但是，如果任何依赖线程被中止，那么这个线程也会被中止并进入停止状态。

已完成或已中止的线程无法重新启动。

线程可以在前台或后台运行。让我们先看看前台和后台线程，从前台线程开始：

+   **前台线程**：默认情况下，线程在前台运行。只要至少有一个前台线程正在运行，进程就会继续运行。即使`Main()`完成了，但前台线程正在运行，应用程序进程仍将保持活动状态，直到前台线程终止。创建前台线程非常简单，如下面的代码所示：

```cs
var foregroundThread = new Thread(SomeMethodName);
foregroundThread.Start();
```

+   **后台线程**：创建后台线程的方式与创建前台线程的方式相同，只是您还必须显式地将线程设置为后台运行，如下所示：

```cs
var backgroundThread = new Thread(SomeMethodName);
backgroundThread.IsBackground = true;
backgroundThread.Start();
```

后台线程用于执行后台任务并保持用户界面对用户的响应。当主进程终止时，任何正在执行的后台线程也将被终止。但是，即使主进程终止，任何正在运行的前台线程也将运行到完成。

在下一节中，我们将看一下线程参数。

# 添加线程参数

在线程中运行的方法通常具有参数。因此，在线程中执行方法时，了解如何将方法参数传递到线程中是有用的。

假设我们有以下方法，它将两个整数相加并返回结果：

```cs
private static int Add(int a, int b)
{
 return a + b;
}
```

正如您所看到的，该方法很简单。有两个名为`a`和`b`的参数。这两个参数将需要传递到线程中，以便`Add()`方法能够正确运行。我们将添加一个示例方法来实现这一点：

```cs
private static void ThreadParametersExample()
{
    int result = 0;
    Thread thread = new Thread(() => { result = Add(1, 2); });
    thread.Start();
    thread.Join();
    Message($"The addition of 1 plus 2 is {result}.");
}
```

在这个方法中，我们声明一个初始值为`0`的整数。然后我们创建一个调用带有`1`和`2`参数值的`Add()`方法的新线程，然后将结果赋给整数变量。然后线程开始，我们等待它通过调用`Join()`方法执行完毕。最后，我们将结果打印到控制台窗口。

让我们添加我们的`Message()`方法：

```cs
internal static void Message(string message)
{
    Console.WriteLine(message);
}
```

`Message()`方法只是接受一个字符串并将其输出到控制台窗口。现在我们只需要更新`Main()`方法，如下所示：

```cs
static void Main(string[] args)
{
    ThreadParametersExample();
    Message("=== Press any Key to exit ===");
    Console.ReadKey();
}
```

在我们的`Main()`方法中，我们调用我们的示例方法，然后等待用户按任意键退出。您应该看到以下输出：

![](img/4d13b988-cbf1-4b6f-a5f1-8e5ecabdb34b.png)

正如你所看到的，`1`和`2`是传递给加法方法的方法参数，`3`是线程返回的值。我们将要看的下一个主题是使用线程池。

# 使用线程池

线程池通过在应用程序初始化期间创建一组线程来提高性能。当需要线程时，它被分配一个单独的任务。该任务将被执行。执行完毕后，线程将返回到线程池以便重用。

由于在.NET 中创建线程是昂贵的，我们可以通过使用线程池来提高性能。每个进程都有一定数量的线程，这取决于可用的系统资源，如内存和 CPU。但是，我们可以增加或减少线程池使用的线程数量。通常最好让线程池负责使用多少线程，而不是手动设置这些值。

创建线程池的不同方法如下：

+   使用**任务并行库**（**TPL**）（在.NET Framework 4.0 及更高版本）

+   使用`ThreadPool.QueueUserWorkItem()`

+   使用异步委托

+   使用`BackgroundWorker`

作为经验法则，您应该只在服务器端应用程序中使用线程池。对于客户端应用程序，根据需要使用前台和后台线程。

在本书中，我们只会看一下**TPL**和`QueueUserWorkItem()`方法。您可以在[`www.albahari.com/threading/`](http://www.albahari.com/threading/)上查看如何使用其他两种方法。我们接下来将看一下 TPL。

## 任务并行库

C#中的异步操作由任务表示。C#中的任务由 TPL 中的`Task`类表示。正如你从名称中可以得知的那样，任务并行使多个任务能够同时执行，我们将在接下来的小节中学习。我们将首先看一下`Parallel`类的`Invoke()`方法。

### Parallel.Invoke()

在我们的第一个示例中，我们将使用`Parallel.Invoke()`来调用三个单独的方法。添加以下三个方法：

```cs
private static void MethodOne()
{
    Message($"MethodOne Executed: Thread Id({Thread.CurrentThread.ManagedThreadId})");
}

private static void MethodTwo()
{
    Message($"MethodTwo Executed: Thread Id({Thread.CurrentThread.ManagedThreadId})");
}

private static void MethodThree()
{
    Message($"MethodThree Executed: Thread Id({Thread.CurrentThread.ManagedThreadId})");
}
```

如你所见，这三个方法几乎是相同的，除了它们的名称和通过我们之前编写的`Message()`方法打印到控制台窗口的消息。现在，我们将添加`UsingTaskParallelLibrary()`方法来并行执行这三个方法：

```cs
private static void UsingTaskParallelLibrary()
{
    Message($"UsingTaskParallelLibrary Started: Thread Id = ({Thread.CurrentThread.ManagedThreadId})");
    Parallel.Invoke(MethodOne, MethodTwo, MethodThree);
    Message("UsingTaskParallelLibrary Completed.");
}
```

在这个方法中，我们向控制台窗口写入一条消息，指示方法的开始。然后，我们并行调用`MethodOne`、`MethodTwo`和`MethodThree`方法。然后，我们向控制台窗口写入一条消息，指示方法已经到达结束，然后我们等待按键退出方法。运行代码，你应该看到以下输出：

![](img/320d0805-ddc4-4c24-a70d-3faea45cafd6.png)

在上面的截图中，你可以看到线程一被重用。现在让我们转到`Parallel.For()`循环。

### Parallel.For()

在我们下一个 TPL 示例中，我们将看一个简单的`Parallel.For()`循环。将以下方法添加到新的.NET Framework 控制台应用程序的`Program`类中：

```cs
private static void Method()
{
    Message($"Method Executed: Thread Id({Thread.CurrentThread.ManagedThreadId})");
}
```

这个方法只是在控制台窗口输出一个字符串。现在，我们将创建执行`Parallel.For()`循环的方法：

```cs
private static void UsingTaskParallelLibraryFor()
{
    Message($"UsingTaskParallelLibraryFor Started: Thread Id = ({Thread.CurrentThread.ManagedThreadId})");
    Parallel.For(0, 1000, X => Method());
    Message("UsingTaskParallelLibraryFor Completed.");
}
```

在这个方法中，我们循环从`0`到`1000`，调用`Method()`。你将看到线程如何在不同的方法调用中被重用，如下面的截图所示：

![](img/b4ae2005-16d0-422a-81aa-00d07a93a8db.png)

现在，我们将看看如何使用`ThreadPool.QueueUserWorkItem()`方法。

## ThreadPool.QueueUserWorkItem()

`ThreadPool.QueueUserWorkItem()`方法接受`WaitCallback`方法并将其排队准备执行。`WaitCallback`是一个代表回调方法的委托，将由线程池线程执行。当线程可用时，该方法将被执行。让我们添加一个简单的例子。我们将首先添加`WaitCallbackMethod`：

```cs
private static void WaitCallbackMethod(Object _)
{
    Message("Hello from WaitCallBackMethod!");
}
```

这个方法接受一个对象类型。然而，由于参数将不被使用，我们使用丢弃变量（`_`）。一条消息被打印到控制台窗口。现在，我们只需要调用这个方法的代码：

```cs
private static void ThreadPoolQueueUserWorkItem()
{
    ThreadPool.QueueUserWorkItem(WaitCallbackMethod);
    Message("Main thread does some work, then sleeps.");
    Thread.Sleep(1000);
    Message("Main thread exits.");
}
```

如你所见，我们使用`ThreadPool`类通过调用`QueueUserWorkItem()`方法将`WaitCallbackMethod()`排队到线程池中。然后我们在主线程上做一些工作。主线程然后进入睡眠状态。一个线程从线程池中可用，`WaitCallBackMethod()`被执行。然后线程被返回到线程池中以便重用。执行返回到主线程，然后完成并终止。

在下一节中，我们将讨论线程锁定对象，即**互斥对象**（**mutexes**）。

# 使用互斥体与同步线程

在 C#中，互斥体是一个跨多个进程工作的线程锁定对象。只有能够请求或释放资源的进程才能修改互斥体。当互斥体被锁定时，进程将不得不在队列中等待。当互斥体被解锁时，它就可以被访问。多个线程可以以同步的方式使用相同的互斥体。

使用互斥体的好处是，互斥体是在进入关键代码之前获取的简单锁。当关键代码退出时，该锁将被释放。因为在任何时候只有一个线程在关键代码中，数据将保持一致状态，因为不会出现竞争条件。

使用互斥体有一些缺点：

+   当现有线程获得锁并且要么进入睡眠状态要么被抢占（无法完成任务）时，线程饥饿就会发生。

+   当互斥体被锁定时，只有获得锁的线程才能解锁它。没有其他线程可以锁定或解锁它。

+   一次只允许一个线程进入关键代码段。CPU 时间可能会被浪费，因为互斥体的正常实现可能导致*忙等待*状态。

现在，我们将编写一个演示互斥体使用的程序。启动一个新的.NET Framework 控制台应用程序。将以下行添加到类的顶部：

```cs
private static readonly Mutex _mutex = new Mutex();
```

在这里，我们声明了一个名为`_mutex`的原始类型，我们将使用它进行进程间同步。现在，添加一个方法来演示使用互斥体进行线程同步：

```cs
private static void ThreadSynchronisationUsingMutex()
{
    try
    {
        _mutex.WaitOne();
        Message($"Domain Entered By: {Thread.CurrentThread.Name}");
        Thread.Sleep(500);
        Message($"Domain Left By: {Thread.CurrentThread.Name}");
    }
    finally
    {
        _mutex.ReleaseMutex();
    }
}
```

在这个方法中，当前线程被阻塞，直到当前等待句柄接收到信号。然后，当给出信号时，下一个线程可以安全地进入。完成后，其他线程将被解除阻塞，以尝试获得互斥体的所有权。接下来，添加`MutexExample()`方法：

```cs
private static void MutexExample()
{
    for (var i = 1; i <= 10; i++)
    {
        var thread = new Thread(ThreadSynchronisationUsingMutex)
        {
            Name = $"Mutex Example Thread: {i}"
        };
        thread.Start();
    }
}
```

在这个方法中，我们创建 10 个线程并启动它们。每个线程执行`ThreadSynchronisationUsingMutex()`方法。现在，最后更新`Main()`方法：

```cs
static void Main(string[] args)
{
    SemaphoreExample();
    Console.ReadKey();
}
```

`Main()`方法运行我们的互斥体示例。输出应该类似于以下截图中的输出：

![](img/cb8cc4bd-8486-4e2d-b085-c4d9f0915b5b.png)

再次运行示例，可能会得到不同的线程编号。如果它们是相同的数字，则它们可能以不同的顺序排列。

现在我们已经看过互斥体，让我们看看信号量。

# 使用信号量处理并行线程

在多线程应用程序中，一个非负数，称为**信号量**，在具有`1`或`2`的线程之间共享。在同步方面，`1`指定*等待*，`2`指定*信号*。我们可以将信号量与一些缓冲区相关联，每个缓冲区可以由不同的进程同时处理。

因此，本质上，信号量是可以通过等待和信号操作来修改的整数和二进制原始类型的信号机制。如果没有空闲资源，那么需要资源的进程应执行等待操作，直到信号量值大于 0。信号量可以有多个程序线程，并且可以被任何对象更改，获取资源或释放资源。

使用信号量的优势在于多个线程可以访问关键代码段。信号量在内核中执行，并且与机器无关。如果使用信号量，关键代码段将受到多个进程的保护。与互斥体不同，信号量永远不会浪费处理时间和资源。

与互斥体一样，信号量也有自己的一系列缺点。优先级反转是最大的缺点之一，当高优先级线程被迫等待低优先级拥有线程释放信号量时就会发生。

如果低优先级线程在释放之前被中优先级线程阻止完成，这种情况会更加复杂。这被称为**无界优先级反转**，因为我们无法预测高优先级线程的延迟。使用信号量时，操作系统必须跟踪所有等待和信号调用。

信号量是按照惯例使用的，但并非强制。您需要按正确的顺序执行等待和信号操作；否则，您的代码可能会出现死锁。由于使用信号量的复杂性，有时可能无法获得互斥体。在大型系统中失去模块化也是另一个缺点，信号量容易出现编程错误，导致死锁和互斥体违规。

现在我们将编写一个演示使用信号量的程序：

```cs
private static readonly Semaphore _semaphore = new Semaphore(2, 4); 
```

我们添加了一个新的信号量变量。第一个参数表示可以同时授予的信号量的初始请求数。第二个参数表示可以同时授予的信号量的最大请求数。添加`StartSemaphore()`方法：

```cs
private static void StartSemaphore(object id)
{
    Console.WriteLine($"Object {id} wants semaphore access.");
 try
 {
 _semaphore.WaitOne();
 Console.WriteLine($"Object {id} gained semaphore access.");
 Thread.Sleep(1000);
 Console.WriteLine($"Object {id} has exited semaphore.");
 }
 finally
 {
 _semaphore.Release();
 }
}
```

当前线程被阻塞，直到当前等待句柄接收到一个信号。然后线程可以执行它的工作。最后，信号量被释放，计数返回到之前的计数。现在，添加`SemaphoreExample()`方法：

```cs
private static void SemaphoreExample()
{
    for (int i = 1; i <= 10; i++)
    {
        Thread t = new Thread(StartSemaphore);
        t.Start(i);
    }
}
```

这个例子生成 10 个线程，这些线程执行`StartSemaphore()`方法。让我们更新`Main()`方法来运行这段代码：

```cs
static void Main(string[] args)
{
    SemaphoreExample();
    Console.ReadKey();
}
```

`Main()`方法调用`SemaphoreExample()`，然后等待用户按键退出。你应该看到以下输出：

![](img/33cf6f91-f372-464f-b382-81a7646148a4.png)

让我们继续看如何限制线程池中处理器和线程的数量。

# 限制线程池中处理器和线程的数量

有时候你可能需要限制计算机程序使用的处理器和线程的数量。

为了减少程序使用的处理器数量，你获取当前进程并设置它的处理器亲和性值。例如，假设我们有一台四核计算机，我们想将使用限制在前两个核心上。前两个核心的二进制值是`11`，在整数形式中是`3`。现在，让我们添加一个方法到一个新的.NET Framework 控制台应用程序，并称其为`AssignCores()`：

```cs
private static void AssignCores(int cores)
{
    Process.GetCurrentProcess().ProcessorAffinity = new IntPtr(cores);
}
```

我们向方法传递一个整数。这个整数值将被.NET Framework 转换为一个二进制值。这个二进制值将使用由`1`值确定的处理器。对于二进制值为`0`，处理器将不会被使用。因此，由二进制数表示的机器码，`0110`（`6`）将使用核心`2`和`3`，`1100`（`3`）将使用核心`1`和`2`，`0011`（`12`）将使用核心`3`和`4`。

如果你需要关于二进制的复习，请参考[`www.computerhope.com/jargon/b/binary.htm`](https://www.computerhope.com/jargon/b/binary.htm)。

现在，为了设置最大线程数，我们在`ThreadPool`类上调用`SetMaxThreads()`方法。这个方法接受两个参数，都是整数。第一个参数是线程池中工作线程的最大数量，第二个参数是线程池中异步 I/O 线程的最大数量。现在，我们将添加我们的方法来设置最大线程数：

```cs
private static void SetMaxThreads(int workerThreads, int asyncIoThreads)
{
    ThreadPool.SetMaxThreads(workerThreads, asyncIoThreads);
}
```

正如你所看到的，设置程序中线程的最大数量和处理器是非常简单的。大多数情况下，你不需要在程序中这样做。手动设置线程和/或处理器数量的主要原因是如果你的程序遇到性能问题。如果你的程序没有遇到性能问题，最好不要设置线程或处理器的数量。

下一个我们将要看的主题是死锁。

# 防止死锁

**死锁**发生在两个或更多线程执行并且互相等待对方完成时。当计算机程序出现这个问题时，会出现挂起的情况。对于最终用户来说，这可能非常糟糕，并且可能导致数据的丢失或损坏。一个例子是执行两批数据输入，其中一半事务崩溃并且无法回滚。这是不好的；让我用一个例子来解释为什么。

考虑一个重大的银行交易，将 100 万英镑从客户的企业银行账户中取出，用于支付**女王陛下的税务和海关**（HMRC）的税单。资金从企业账户中取出，但在将资金存入 HMRC 银行账户之前，发生了死锁。没有恢复选项，因此必须终止并重新启动应用程序。因此，企业银行账户减少了 100 万英镑，但 HMRC 税单尚未支付。客户仍然有责任支付税单。但已从账户中取出的资金会发生什么？因此，您可以看到由于可能引起的问题，消除死锁发生的可能性的重要性。

为了简化问题，我们将处理两个线程，如下图所示：

![](img/c6547929-7a72-40da-817a-87481d5face9.png)

我们将称我们的线程为 Thread 1 和 Thread 2，我们的资源为 Resource 1 和 Resource 2。Thread 1 在 Resource 1 上获取锁。Thread 2 在 Resource 2 上获取锁。Thread 1 需要访问 Resource 2，但必须等待，因为 Thread 2 已锁定 Resource 2。Thread 2 需要访问 Resource 1，但必须等待，因为 Thread 1 已锁定 Resource 1。这导致 Thread 1 和 Thread 2 都处于等待状态。由于没有线程可以继续，直到另一个线程释放其资源，因此两个线程都处于死锁状态。当计算机程序处于死锁状态时，它会*挂起*，强制您*终止*该程序。

死锁的代码示例将是说明这一点的好方法，因此在下一节中，我们将编写一个死锁示例。

## 编写死锁示例

理解这一点的最好方法是通过一个工作示例。我们将编写一些代码，其中包含两个具有两个不同锁的方法。它们将锁定彼此方法需要的对象。因为每个线程锁定了另一个线程需要的资源，所以它们都将进入死锁状态。一旦我们的示例工作正常，我们将修改代码，使我们的代码能够从死锁情况中恢复并继续。

创建一个新的.NET Framework 控制台应用程序，并将其命名为`CH08_Deadlocks`。我们将需要两个对象作为成员变量，因此让我们添加它们：

```cs
static object _object1 = new object();
 static object _object2 = new object();
```

这些对象将用作我们的锁对象。我们将有两个线程，每个线程将执行自己的方法。现在，在您的代码中添加`Thread1Method()`：

```cs
private static void Thread1Method()
 {
     Console.WriteLine("Thread1Method: Thread1Method Entered.");
     lock (_object1)
     {
         Console.WriteLine("Thread1Method: Entered _object1 lock. Sleeping...");
         Thread.Sleep(1000);
         Console.WriteLine("Thread1Method: Woke from sleep");
         lock (_object2)
         {
             Console.WriteLine("Thread1Method: Entered _object2 lock.");
         }
         Console.WriteLine("Thread1Method: Exited _object2 lock.");
     }
     Console.WriteLine("Thread1Method: Exited _object1 lock.");
 }
```

`Thread1Method()`在`_object1`上获取锁。然后休眠 1 秒。醒来时，在`_object2`上获得锁。然后该方法退出两个锁并终止。

`Thread2Method()`在`_object2`上获取锁。然后休眠 1 秒。醒来时，在`_object1`上获得锁。然后该方法退出两个锁并终止：

```cs
private static void Thread2Method()
 {
     Console.WriteLine("Thread2Method: Thread1Method Entered.");
     lock (_object2)
     {
         Console.WriteLine("Thread2Method: Entered _object2 lock. Sleeping...");
         Thread.Sleep(1000);
         Console.WriteLine("Thread2Method: Woke from sleep.");
         lock (_object1)
         {
             Console.WriteLine("Thread2Method: Entered _object1 lock.");
         }
         Console.WriteLine("Thread2Method: Exited _object1 lock.");
     }
     Console.WriteLine("Thread2Method: Exited _object2 lock.");
 }
```

好吧，我们现在已经有了两种方法来演示死锁。我们只需要编写调用它们的代码，以一种会导致死锁的方式。让我们添加`DeadlockNoRecovery()`方法：

```cs
private static void DeadlockNoRecovery()
 {
     Thread thread1 = new Thread((ThreadStart)Thread1Method);
     Thread thread2 = new Thread((ThreadStart)Thread2Method);

     thread1.Start();
     thread2.Start();

     Console.WriteLine("Press any key to exit.");
     Console.ReadKey();
 }
```

在`DeadlockNoRecovery()`方法中，我们创建两个线程。每个线程分配一个不同的方法。然后启动每个线程。然后，程序暂停，直到用户按下键。现在，更新`Main()`方法并运行您的代码：

```cs
static void Main()
 {
     DeadlockNoRecovery();
 }
```

运行程序时，您应该看到以下输出：

![](img/3cc47b74-1c00-4540-b392-39a9b241a776.png)

如您所见，因为`thread1`已锁定`_object1`，所以`thread2`无法获取`_object1`的锁。同样，因为`thread2`已锁定`_object2`，所以`thread1`无法获取`_object2`的锁。因此，两个线程都处于死锁状态，程序挂起。

我们现在将编写一些代码，演示如何避免发生这种死锁情况。我们将使用`Monitor.TryLock()`方法尝试在一定毫秒数内获得锁。然后我们将使用`Monitor.Exit()`退出成功的锁。

现在，添加`DeadlockWithRecovery()`方法：

```cs
private static void DeadlockWithRecovery()
 {
     Thread thread4 = new Thread((ThreadStart)Thread4Method);
     Thread thread5 = new Thread((ThreadStart)Thread5Method);

     thread4.Start();
     thread5.Start();

     Console.WriteLine("Press any key to exit.");
     Console.ReadKey();
 }
```

`DeadlockWithRecovery()`方法创建两个前台线程。然后启动线程，在控制台打印一条消息，并等待用户按键退出。现在我们将为`Thread4Method()`添加代码：

```cs
private static void Thread4Method()
 {
     Console.WriteLine("Thread4Method: Entered _object1 lock. Sleeping...");
     Thread.Sleep(1000);
     Console.WriteLine("Thread4Method: Woke from sleep");
     if (!Monitor.TryEnter(_object1))
     {
         Console.WriteLine("Thead4Method: Failed to lock _object1.");
         return;
     }
     try
     {
         if (!Monitor.TryEnter(_object2))
         {
             Console.WriteLine("Thread4Method: Failed to lock _object2.");
             return;
         }
         try
         {
             Console.WriteLine("Thread4Method: Doing work with _object2.");
         }
         finally
         {
             Monitor.Exit(_object2);
             Console.WriteLine("Thread4Method: Released _object2 lock.");
         }
     }
     finally
     {
         Monitor.Exit(_object1);
         Console.WriteLine("Thread4Method: Released _object2 lock.");
     }
 }
```

`Thread4Method()`休眠 1 秒。然后尝试锁定`_object1`。如果无法锁定`_object1`，则从方法返回。如果成功锁定`_object1`，则尝试锁定`_object2`。如果无法锁定`_object2`，则从方法返回。如果成功锁定`_object2`，则对`_object2`执行必要的工作。然后释放`_object2`的锁，然后释放`_object1`的锁。

我们的`Thread5Method()`方法做的事情完全相同，只是对象`_object1`和`_object2`以相反的顺序被锁定：

```cs
private static void Thread5Method()
 {
     Console.WriteLine("Thread5Method: Entered _object2 lock. Sleeping...");
     Thread.Sleep(1000);
     Console.WriteLine("Thread5Method: Woke from sleep");
     if (!Monitor.TryEnter(_object2))
     {
         Console.WriteLine("Thead5Method: Failed to lock _object2.");
         return;
     }
     try
     {
         if (!Monitor.TryEnter(_object1))
         {
             Console.WriteLine("Thread5Method: Failed to lock _object1.");
             return;
         }
         try
         {
             Console.WriteLine("Thread5Method: Doing work with _object1.");
         }
         finally
         {
             Monitor.Exit(_object1);
             Console.WriteLine("Thread5Method: Released _object1 lock.");
         }
     }
     finally
     {
         Monitor.Exit(_object2);
         Console.WriteLine("Thread5Method: Released _object2 lock.");
     }
 }
```

现在，将`DeadlockWithRecovery()`方法调用添加到你的`Main()`方法中：

```cs
static void Main()
 {
     DeadlockWithRecovery();
 }
```

然后运行你的代码几次。大多数情况下，你会看到以下截图中的情况，所有锁都已成功获取：

![](img/fa0d5689-7404-4341-9bb8-880b35507571.png)

然后按任意键，程序将退出。如果继续运行程序，最终会发现一个锁定失败。程序在`Thread5Method()`中无法获取`_object2`的锁。但是，如果按任意键，程序将退出。如你所见，通过使用`Monitor.TryEnter()`，你可以尝试锁定一个对象。但如果未获得锁定，则可以在程序挂起的情况下采取其他操作。

在下一节中，我们将看看如何防止竞争条件。

# 防止竞争条件

当多个线程使用相同的资源产生不同的结果，这是由于每个线程的时间安排不同，这被称为**竞争条件**。我们现在将演示这一点。

在我们的演示中，将有两个线程。每个线程将调用一个打印字母的方法。一个方法将使用*大写字母*打印字母表。第二个方法将使用*小写字母*打印字母表。从演示中，我们将看到输出是错误的，每次运行程序时，输出都将是错误的。

首先，添加`ThreadingRaceCondition()`方法：

```cs
static void ThreadingRaceCondition()
 {
     Thread T1 = new Thread(Method1);
     T1.Start();
     Thread T2 = new Thread(Method2);
     T2.Start();
 }
```

`ThreadingRaceCondition()`产生两个线程并启动它们。它还引用两个方法。`Method1()`以大写字母打印字母表，`Method2()`以小写字母打印字母表。让我们添加`Method1()`和`Method2()`：

```cs
static void Method1()
 {
     for (_alphabetCharacter = 'A'; _alphabetCharacter <= 'Z'; _alphabetCharacter ++)
     {
         Console.Write(_alphabetCharacter + " ");
     }
 }

private static void Method2()
 {
     for (_alphabetCharacter = 'a'; _alphabetCharacter <= 'z'; _alphabetCharacter++)
     {
         Console.Write(_alphabetCharacter + " ");
     }
 }
```

`Method1()`和`Method2()`都引用`_alphabetCharacter`变量。因此，在类的顶部添加该成员：

```cs
private static char _alphabetCharacter;
```

现在，更新`MainMethod()`：

```cs
static void Main(string[] args)
 {
     Console.WriteLine("\n\nRace Condition:");
     ThreadingRaceCondition();
     Console.WriteLine("\n\nPress any key to exit.");
     Console.ReadKey();
 }
```

我们现在已经准备好演示竞争条件的代码。如果多次运行程序，你会发现结果不是我们期望的。甚至会看到不属于字母表的字符：

![](img/0c017737-c3f3-4839-b064-e30731c5db83.png)

不完全是我们期望的，是吗？

我们将使用 TPL 来解决这个问题。TPL 的目标是简化**并行性**和**并发性**。由于今天大多数计算机都有两个或更多处理器，TPL 将动态地扩展并发度，以充分利用所有可用的处理器。

TPL 还执行工作分区、线程池中的线程调度、取消支持、状态管理等。本章的*进一步阅读*部分中可以找到官方 Microsoft TPL 文档的链接。

你将看到上述问题的解决方案是多么简单。我们有一个运行`Method1()`的任务。任务然后继续执行`Method2()`。然后我们调用`Wait()`等待任务完成执行。现在，在你的源代码中添加`ThreadingRaceConditionFixed()`方法：

```cs
static void ThreadingRaceConditionFixed()
 {
     Task
         .Run(() => Method1())
         .ContinueWith(task => Method2())
         .Wait();
 }
```

修改你的`Main()`方法如下：

```cs
static void Main(string[] args)
 {
     //Console.WriteLine("\n\nRace Condition:");
     //ThreadingRaceCondition();
     Console.WriteLine("\n\nRace Condition Fixed:");
     ThreadingRaceConditionFixed();
     Console.WriteLine("\n\nPress any key to exit.");
     Console.ReadKey();
 }
```

现在运行代码。如果多次运行，你会发现输出总是相同的，如下截图所示：

![](img/0cf0ae1c-9d51-4196-9316-36317f555b56.png)

到目前为止，我们已经了解了线程是什么以及如何在前台和后台使用它们。我们还看了死锁以及如何使用`Monitor.TryEnter()`解决它们。最后，我们看了什么是竞争条件以及如何使用 TPL 解决它们。

现在，我们将继续查看静态构造函数和方法。

# 理解静态构造函数和方法

如果多个类需要同时访问一个属性实例，则其中一个线程将被要求运行**静态构造函数**（也称为**类型初始化程序**）。在等待类型初始化程序运行时，所有其他线程将被锁定。一旦类型初始化程序运行，被锁定的线程将被解锁，并能够访问`Instance`属性。

静态构造函数是线程安全的，因为它们保证每个应用程序域只运行一次。它们在访问任何静态成员和执行任何类实例化之前执行。

如果静态构造函数中引发异常并且逃逸，那么将生成`TypeInitializationException`，这会导致 CLR 退出您的程序。

在任何线程可以访问一个类之前，静态初始化程序和静态构造函数必须执行完成。

**静态方法**只在类型级别保留方法及其数据的单个副本。这意味着相同的方法及其数据将在不同实例之间共享。应用程序中的每个线程都有自己的堆栈。传递给静态方法的值类型是在调用线程的堆栈上创建的，因此它们是线程安全的。这意味着如果两个线程调用相同的代码并传递相同的值，那么该值将有两个副本，分别在每个线程的堆栈上。因此，多个线程不会相互影响。

但是，如果您有一个访问成员变量的静态方法，那么它就不是线程安全的。两个不同的线程调用相同的方法，因此两者都将访问成员变量。在线程之间发生进程或上下文切换；每个线程将访问并修改成员变量。这会导致竞争条件，正如您在本章前面看到的那样。

如果将引用类型传递给静态方法，也会遇到问题，因为不同的线程将可以访问相同的引用类型。这也会导致竞争条件。

在使用将在多个线程之间使用的静态方法时，避免访问成员变量并且不要传递引用类型。只要传递原始类型并且不修改状态，静态方法就是线程安全的。

现在我们已经讨论了静态构造函数和方法，我们将运行一些示例代码。

## 向我们的示例代码添加静态构造函数

启动一个新的.NET Framework 控制台应用程序。向项目添加一个名为`StaticConstructorTestClass`的类。然后，添加一个名为`_message`的只读静态字符串变量：

```cs
public class StaticConstructorTestClass
{
    private readonly static string _message;
}
```

`Message()`方法通过`_message`变量将消息返回给调用者。现在让我们编写`Message()`方法：

```cs
public static string Message()
{
    return $"Message: {_message}";
}
```

该方法返回存储在`_message`变量中的消息。现在，我们需要编写我们的构造函数：

```cs
static StaticConstructorTestClass()
{
    Console.WriteLine("StaticConstructorTestClass static constructor started.");
    _message = "Hello, World!";
    Thread.Sleep(1000);
    _message = "Goodbye, World!";
    Console.WriteLine("StaticConstructorTestClass static constructor finished.");
}
```

在我们的构造函数中，我们向屏幕写入一条消息。然后，我们设置成员变量并让线程休眠一秒钟。然后，我们再次设置消息并向控制台写入另一条消息。现在，在`Program`类中，更新`Main()`方法如下：

```cs
static void Main(string[] args)
{
    var program = new Program();
    program.StaticConstructorExample();
    Thread.CurrentThread.Join();
}
```

我们的`Main()`方法实例化`Program`类。然后调用`StaticConstructorExample()`方法。当程序停止并且我们可以看到结果时，我们加入线程。您可以在以下截图中看到输出：

![](img/d79fb288-6c27-4ef6-8c6d-8cfa2aa74ac3.png)

现在我们将看一些静态方法的示例。

## 向我们的示例代码添加静态方法

我们现在将看看线程安全的静态方法和非线程安全的方法是如何运作的。向新的.NET Framework 控制台应用程序添加一个名为`StaticExampleClass`的新类。然后，添加以下代码：

```cs
public static class StaticExampleClass
{
    private static int _x = 1;
    private static int _y = 2;
    private static int _z = 3;
}
```

在我们的类顶部，我们添加了三个整数——`_x`、`_y`和`_z`——分别为`1`、`2`和`3`。这些变量可以在线程之间修改。现在，我们将添加一个静态构造函数来打印这些变量的值：

```cs
static StaticExampleClass()
{
    Console.WriteLine($"Constructor: _x={_x}, _y={_y}, _z={_z}");
}
```

如您所见，静态构造函数只是将变量的值打印到控制台窗口。我们的第一个方法将是一个名为`ThreadSafeMethod()`的线程安全方法：

```cs
internal static void ThreadSafeMethod(int x, int y, int z)
{
    Console.WriteLine($"ThreadSafeMethod: x={x}, y={y}, z={z}");
    Console.WriteLine($"ThreadSafeMethod: {x}+{y}+{z}={x+y+z}");
}
```

这个方法是线程安全的，因为它只对值参数进行操作。它不与成员变量交互，也不包括任何引用值。因此，无论传入什么值，您都将获得预期的结果。

这意味着无论是单个线程还是数百万个线程访问该方法，每个线程的输出都将是您期望的，即使发生上下文切换。以下截图显示了输出：

![](img/014d5718-ac04-4629-92ea-2e7e0405cc34.png)

既然我们已经看过了线程安全的方法，现在我们应该看看非线程安全的方法。到目前为止，您已经知道操作引用值或静态成员变量的静态方法是不线程安全的。

在我们的下一个示例中，我们将使用一个与`ThreadSafeMethod()`具有相同三个参数的方法，但这次我们将设置成员变量，输出一条消息，睡一会儿，然后醒来再次打印出值。将以下`NotThreadSafeMethod()`方法添加到`StaticExampleClass`中：

```cs
internal static void NotThreadSafeMethod(int x, int y, int z)
{
    _x = x;
    _y = y;
    _z = z;
    Console.WriteLine(
        $"{Thread.CurrentThread.ManagedThreadId}-NotThreadSafeMethod: _x={_x}, _y={_y}, _z={_z}"
    );
    Thread.Sleep(300);
    Console.WriteLine(
        $"{Thread.CurrentThread.ManagedThreadId}-ThreadSafeMethod: {_x}+{_y}+{_z}={_x + _y + _z}"
    );
}
```

在这个方法中，我们将成员变量设置为传入方法的值。然后，我们将这些值输出到控制台窗口，并睡眠 300 毫秒。然后，在醒来后，我们再次打印出这些值。在`Program`类中，更新`Main()`方法如下：

```cs
static void Main(string[] args)
{
    var program = new Program();
    program.ThreadUnsafeMethodCall();
    Console.ReadKey();
}
```

在`Main()`方法中，我们实例化程序类，调用`ThreadUnsafeMethodCall()`，然后等待用户按键退出。因此，让我们将`ThreadUnsafeMethodCall()`添加到`Program`类中：

```cs
private void ThreadUnsafeMethodCall()
{
    for (var index = 0; index < 10; index++)
    {
        var thread = new Thread(() =>
        {
            StaticExampleClass.NotThreadSafeMethod(index + 1, index + 2, index + 3);
        });
        thread.Start();
    }
}
```

该方法产生 10 个调用`StaticExampleClass`的`NotThreadSafeMethod()`的线程。如果运行代码，您将看到类似于以下截图的输出：

![](img/37710a9d-f8db-4e4d-9cb1-6bb2e7a4b3ec.png)

如您所见，输出不是我们所期望的。这是由于来自不同线程的污染。这很好地引出了下一节关于可变性、不可变性和线程安全性。

# 可变性、不可变性和线程安全

**可变性**是多线程应用程序中错误的来源。可变 bug 通常是由值被更新并在线程之间共享引起的数据 bug。为了消除可变性 bug 的风险，最好使用**不可变类型**。多个线程同时对一段代码的安全执行称为**线程安全**。在处理多线程程序时，重要的是您的代码是线程安全的。如果您的代码消除了竞态条件、死锁以及可变性引起的问题，那么您的代码就是线程安全的。

不可修改的对象在创建后无法修改。一旦创建，如果使用正确的线程同步在线程之间传递，所有线程将看到对象的相同有效状态。不可变对象允许您在线程之间安全地共享数据。

可修改的对象在创建后可以修改。可变对象的数据值可以在线程之间更改。这可能导致严重的数据损坏。因此，即使程序不崩溃，也可能使数据处于无效状态。因此，在处理多个执行线程时，重要的是您的对象是不可变的。在第三章中，*类、对象和数据结构*，我们介绍了为不可变对象创建和使用不可变数据结构。

为了确保线程安全，不要使用可变对象，通过引用传递参数，或修改成员变量——只通过值传递参数，只操作参数变量。不要访问成员变量。不可变结构是在对象之间传递数据的一种良好且线程安全的方式。

我们将简要介绍可变性、不可变性和线程安全，以下是一些示例。我们将从线程安全的可变性开始。

## 编写可变且非线程安全的代码

展示多线程应用程序中的可变性，我们将编写一个新的.NET Framework 控制台应用程序。在应用程序中添加一个名为`MutableClass`的新类，其中包含以下代码：

```cs
internal class MutableClass
{
    private readonly int[] _intArray;

    public MutableClass(int[] intArray)
    {
        _intArray = intArray;
    }

    public int[] GetIntArray()
    {
        return _intArray;
    }
}
```

在我们的`MutableClass`类中，我们有一个以整数数组作为参数的构造函数。然后，将成员整数数组分配给传递到构造函数的数组。`GetIntArray()`方法返回整数数组成员变量。如果您查看这个类，您可能不会认为它是可变的，因为一旦数组传递到构造函数中，类就没有提供修改它的方法。然而，传递到构造函数的整数数组是可变的。`GetIntArray()`方法返回对可变数组的引用。

在我们的`Program`类中，我们将添加`MutableExample()`方法来展示整数数组是可变的：

```cs
private static void MutableExample()
{
    int[] iar = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
    var mutableClass = new MutableClass(iar);

    Console.WriteLine($"Initial Array: {iar[0]}, {iar[1]}, {iar[2]}, {iar[3]}, {iar[4]}, {iar[5]}, {iar[6]}, {iar[7]}, {iar[8]}, {iar[9]}");

    for (var x = 0; x < 9; x++)
    {
        var thread = new Thread(() =>
            {
                iar[x] = x + 1;
                var ia = mutableClass.GetIntArray();
                Console.WriteLine($"Array [{x}]: {ia[0]}, {ia[1]}, {ia[2]}, {ia[3]}, {ia[4]}, {ia[5]}, {ia[6]}, {ia[7]}, {ia[8]}, {ia[9]}");
            });
            thread.Start();
    }
}
```

在我们的`MutableExample()`方法中，我们声明并初始化了一个从`0`到`9`的整数数组。然后，我们声明了`MutableClass`的一个新实例，并传入整数数组。接下来，我们打印出修改前的初始数组内容。然后，我们循环九次。对于每次迭代，我们增加由当前循环计数值`x`指定的数组的索引，使其等于`x + 1`。之后，我们启动线程。现在，更新`Main()`方法，如下所示：

```cs
static void Main(string[] args)
{
    MutableExample();
    Console.ReadKey();
}            
```

我们的`Main()`方法只是调用`MutableExample()`，然后等待按键。运行代码，您应该看到以下截图中的内容：

![](img/56fe7d8d-f687-47a3-86f6-909b4dfad3f1.png)

如您所见，即使在创建和运行线程之前我们只创建了一个`MutableClass`的实例，改变本地数组也会修改`MutableClass`实例中的数组。这证明了数组是可变的，因此它们不是线程安全的。

现在我们将从线程安全的不可变性开始。

## 编写不可变且线程安全的代码

在我们的不可变性示例中，我们将再次创建一个.NET Framework 控制台应用程序，并使用相同的数组。添加一个名为`ImmutableStruct`的类，并修改代码，如下所示：

```cs
internal struct ImmutableStruct
{ 
    private ImmutableArray<int> _immutableArray;

    public ImmutableStruct(ImmutableArray<int> immutableArray)
    {
        _immutableArray = immutableArray;
    }

    public int[] GetIntArray()
    {
        return _immutableArray.ToArray<int>();
    }
}
```

我们使用`ImmutableArray`而不是普通的整数数组。一个不可变数组被传递到构造函数，并赋值给`_immutableArray`成员变量。我们的`GetIntArray()`方法将不可变数组作为普通整数数组返回。

在`Program`类中添加`ImmutableExample()`数组：

```cs
private static void ImmutableExample()
{
    int[] iar = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
    var immutableStruct = new ImmutableStruct(iar.ToImmutableArray<int>());

    Console.WriteLine($"Initial Array: {iar[0]}, {iar[1]}, {iar[2]}, {iar[3]}, {iar[4]}, {iar[5]}, {iar[6]}, {iar[7]}, {iar[8]}, {iar[9]}");

    for (var x = 0; x < 9; x++)
    {
        var thread = new Thread(() =>
        {
            iar[x] = x + 1;
            var ia = immutableStruct.GetIntArray();
            Console.WriteLine($"Array [{x}]: {ia[0]}, {ia[1]}, {ia[2]}, {ia[3]}, {ia[4]}, {ia[5]}, {ia[6]}, {ia[7]}, {ia[8]}, {ia[9]}");
        });
        thread.Start();
    }
}
```

在我们的`ImmutableExample()`方法中，我们创建一个整数数组，将其作为不可变数组传递给`ImmutableStruct`的构造函数。然后，我们打印修改前的本地数组内容。然后，我们循环九次。在每次迭代中，我们访问数组中当前迭代计数的位置，并将当前迭代计数加一的值添加到数组中该位置的变量中。

然后，我们通过调用`GetIntArray()`将`immutableStruct`数组的副本赋给一个本地变量。然后，我们继续打印返回数组的值。最后，我们启动线程。从您的`Main()`方法中调用`ImmutableExample()`方法，然后运行代码。您应该看到以下输出：

![](img/4d54dd02-cdb2-4612-aba5-e67a652dbd9b.png)

如您所见，通过更新本地数组，数组的内容并未被修改。我们的程序的这个版本表明我们的程序是线程安全的。

让我们简要回顾一下我们在下一节中学到的关于线程安全的知识。

## 理解线程安全

正如您在前两节中看到的，编写多线程代码时非常重要要小心。编写线程安全的代码可能非常困难，特别是在较大的项目中。您必须特别小心处理集合、通过引用传递参数以及在静态类中访问成员变量。多线程应用程序的最佳实践是仅传递不可变类型，不要访问静态成员变量，如果必须执行任何不是线程安全的代码，则使用锁定、互斥体或信号量锁定代码。尽管您在本章中已经看到了这样的代码，我们将通过一些代码片段快速回顾一下。

以下代码片段显示了如何使用`readonly struct`编写不可变类型：

```cs
public readonly struct ImmutablePerson
{
    public ImmutablePerson(int id, string firstName, string lastName)
    {
        _id = id;
        _firstName = firstName;
        _lastName = lastName;
    }

    public int Id { get; }
    public string FirstName { get;
    public string LastName { get { return _lastName; } }
}
```

在我们的`ImmutablePerson`结构中，我们有一个公共构造函数，它接受一个整数作为 ID，以及名字和姓氏的字符串。我们将`id`、`firstName`和`lastName`参数分配给成员只读变量。对数据的唯一访问是通过只读属性。这意味着没有办法修改数据。由于数据一旦创建就无法修改，因此被归类为线程安全。因为它是线程安全的，所以不能被不同的线程修改。修改数据的唯一方法是创建一个包含新数据的新结构。

结构体可以是可变的，就像类一样。但是，为了传递不希望被修改的数据，只读结构体是一个很好的、轻量级的选择。它们比类更快地创建和销毁，因为它们被添加到堆栈中，除非它们是堆中的类的一部分。

在之前，我们看到了集合是可变的。但是，还有一个名为`System.Collections.Immutable`的不可变集合的命名空间。以下表列出了此命名空间中的各种项目：

![](img/ef71435c-f95c-44b7-b5ee-f046b7e1be1e.png)

`System.Collections.Immutable`命名空间包含许多不可变集合，您可以在线程之间安全使用。有关更多详细信息，请参阅[`docs.microsoft.com/en-us/dotnet/api/system.collections.immutable?view=netcore-3.1`](https://docs.microsoft.com/en-us/dotnet/api/system.collections.immutable?view=netcore-3.1)。

在 C#中使用锁对象非常简单，如下面的代码片段所示：

```cs
public class LockExample
{
    public object _lock = new object();

    public void UnsafeMethod() 
    {
        lock(_lock)
        {
            // Execute unsafe code.
        }
    }
}
```

我们创建并实例化了`_lock`成员变量。然后，在执行不是线程安全的代码时，我们将代码包装在锁中，并传入`_lock`变量作为锁对象。当线程进入锁时，所有其他线程都被禁止执行代码，直到线程离开锁。使用此代码的一个问题是线程可能进入死锁状态。解决这个问题的一种方法是使用互斥体。

您可以使用同步原语进行进程间同步。首先，在需要保护的代码所在的类顶部添加以下代码：

```cs
private static readonly Mutex _mutex = new Mutex();
```

然后，要使用互斥体，您需要使用以下`try/catch`块包装需要保护的代码：

```cs
try
{
    _mutex.WaitOne();
    // ... Do work here ...
}
finally
{
    _mutex.ReleaseMutex();
}
```

在上面的代码中，`WaitOne()`方法会阻塞当前线程，直到等待句柄接收到信号。一旦互斥体被发出信号，`WaitOne()`方法将返回`true`。调用线程然后拥有互斥体。然后，调用线程可以访问受保护的资源。在受保护资源上完成工作后，通过调用`ReleaseMutex()`释放互斥体。`ReleaseMutex()`在`finally`块中调用，因为您不希望线程因任何原因引发异常而保持资源被锁定。因此，始终在`finally`块中释放互斥体。

保护资源访问的另一种机制是使用信号量。信号量的编码方式与互斥体非常相似，它们执行保护资源的相同角色。信号量和互斥体之间的主要区别在于互斥体是一种锁定机制，而信号量是一种信号机制。要使用信号量而不是锁和互斥体，请在类的顶部添加以下行：

```cs
private static readonly Semaphore _semaphore = new Semaphore(2, 4); 
```

我们现在添加了一个新的信号量变量。第一个参数表示可以同时授予的信号量的初始请求数。第二个参数表示可以同时授予的信号量的最大请求数。然后您将保护对资源的访问，如下所示：

```cs
try
{
    _semaphore.WaitOne();
    // ... Do work here ...
}
finally
{
    _semaphore.Release();
}
```

当前线程被阻塞，直到当前等待句柄接收到信号。然后线程可以执行其工作。最后，信号量被释放。

在本章中，您已经看到如何使用锁定、互斥体和信号量来锁定不是线程安全的代码。还要记住，后台线程在进程完成和终止时终止，而前台线程将继续执行直到完成。如果您有任何必须在不被终止的线程中途运行完成的代码，那么最好使用前台线程而不是后台线程。

下一节涵盖了同步方法依赖关系。

# 同步方法依赖关系

要同步您的代码，使用锁定语句，就像我们之前所做的那样。您还可以在项目中引用`System.Runtime.CompilerServices`命名空间。然后，您可以在方法和属性中添加`[MethodImpl(MethodImplOptions.Synchronized)]`注释。

以下是应用于方法的`[MethodImpl(MethodImplOptions.Synchronized)]`注释的示例：

```cs
[MethodImpl(MethodImplOptions.Synchronized)]
 public static void ThisIsASynchronisedMethod()
 {
      Console.WriteLine("Synchronised method called.");
 }
```

以下是使用`[MethodImpl(MethodImplOptions.Synchronized)]`与属性的示例：

```cs
private int i;
 public int SomeProperty
 {
     [MethodImpl(MethodImplOptions.Synchronized)]
     get { return i; }
     [MethodImpl(MethodImplOptions.Synchronized)]
     set { i = value; }
 }
```

正如您所看到的，很容易遇到死锁或竞争条件，但使用`Monitor.TryEnter()`和`Task.ContinueWith()`同样容易克服死锁和竞争条件。

在下一节中，我们将看看 Interlocked 类。

# 使用 Interlocked 类

在多线程应用程序中，在线程调度程序上下文切换过程中可能会出现错误。一个主要的问题是不同线程更新相同变量。`mscorlib`程序集中`System.Threading.Interlocked`类的方法有助于防止这类错误。`Interlocked`类的方法不会抛出异常，因此在应用简单状态更改时比使用之前看到的`lock`语句更有效。

Interlocked 类中可用的方法如下：

+   `CompareExchange`：比较两个变量并将结果存储在不同的变量中

+   **`Add`**：将两个`Int32`或`Int64`整数变量相加，并将结果存储在第一个整数中

+   `Decrement`：减少`Int32`和`Int64`整数变量的值并存储它们的结果

+   `Increment`：增加`Int32`和`Int64`整数变量的值并存储它们的结果

+   `Read`：读取`Int64`类型的整数变量

+   `Exchange`：在变量之间交换值

现在我们将编写一个简单的控制台应用程序来演示这些方法。首先创建一个新的.NET Framework 控制台应用程序。将以下行添加到`Program`类的顶部：

```cs
private static long _value = long.MaxValue;
private static int _resourceInUse = 0;
```

`_value`变量将用于演示使用互锁方法更新变量。`_resourceInUse`变量用于指示资源是否正在使用。添加`CompareExchangeVariables()`方法：

```cs
private static void CompareExchangeVariables()
{
    Interlocked.CompareExchange(ref _value, 123, long.MaxValue);
}
```

在我们的`CompareExchangeVariables()`方法中，我们调用`CompareExchange()`方法来比较`_value`和`long.MaxValue`。如果两个值相等，则用`123`的值替换`_value`。现在我们将添加我们的`AddVariables()`方法：

```cs
private static void AddVariables()
{
    Interlocked.Add(ref _value, 321);
}
```

`AddVariables()`方法调用`Add()`方法来访问`_value`成员变量，并将其更新为`_value`加`321`的值。接下来，我们将添加`DecrementVariable()`方法：

```cs
private static void DecrementVariable()
{
    Interlocked.Decrement(ref _value);
}
```

这个方法调用`Decrement()`方法，该方法将`_value`成员变量减 1。我们的下一个方法是`IncrementValue()`：

```cs
private static void IncrementVariable()
{
    Interlocked.Increment(ref _value);
}
```

在我们的`IncrementVariable()`方法中，我们通过调用`Increment()`方法来增加`_value`成员变量。接下来我们将编写的方法是`ReadVariable()`方法：

```cs
private static long ReadVariable()
{
    // The Read method is unnecessary on 64-bit systems, because 64-bit 
    // read operations are already atomic. On 32-bit systems, 64-bit read 
    // operations are not atomic unless performed using Read.
    return Interlocked.Read(ref _value);
}
```

由于 64 位读操作是原子的，调用`Interlocked.Read()`方法是不必要的。但是，在 32 位系统上，为了使 64 位读操作是原子的，你需要调用`Interlocked.Read()`方法。添加`PerformUnsafeCodeSafely()`方法：

```cs
private static void PerformUnsafeCodeSafely()
{
    for (int i = 0; i < 5; i++)
    {
        UseResource();
        Thread.Sleep(1000);
    }
}
```

`PerformUnsafeCodeSafely()`方法循环五次。循环的每次迭代调用`UseResource()`方法，然后线程休眠 1 秒。现在，我们将添加`UseResource()`方法：

```cs
static bool UseResource()
{
    if (0 == Interlocked.Exchange(ref _resourceInUse, 1))
    {
        Console.WriteLine($"{Thread.CurrentThread.Name} acquired the lock");
        NonThreadSafeResourceAccess();
        Thread.Sleep(500);
        Console.WriteLine($"{Thread.CurrentThread.Name} exiting lock");
        Interlocked.Exchange(ref _resourceInUse, 0);
        return true;
    }
    else
    {
        Console.WriteLine($"{Thread.CurrentThread.Name} was denied the lock");
        return false;
    }
}
```

`UseResource()`方法防止在资源正在使用时获取锁，这由`_resourceInUse`变量标识。我们首先通过调用`Exchange()`方法将`_resourceInUse`成员变量的值设置为`1`。`Exchange()`方法返回一个整数，我们将其与`0`进行比较。如果`Exchange()`返回的值是`0`，那么该方法没有在使用中。

如果方法正在使用中，那么我们输出一条消息，通知用户当前线程被拒绝了锁。

如果方法没有在使用中，那么我们输出一条消息，通知用户当前线程已获得锁。然后我们调用`NonThreadSafeResourceAccess()`方法，然后让线程休眠半秒，模拟工作。

当线程唤醒时，我们输出一条消息，通知用户当前线程已退出锁。然后，我们通过调用`Exchange()`方法释放锁，并将`_resourceInUse`的值设置为`0`。添加`NonThreadSafeResourceAccess()`方法：

```cs
private static void NonThreadSafeResourceAccess()
{
    Console.WriteLine("Non-thread-safe code executed.");
}
```

`NonThreadSafeResourceAccess()`是在锁的安全性中执行非线程安全代码的地方。在我们的方法中，我们只是用一条消息通知用户。在运行代码之前，我们要做的最后一件事是更新我们的`Main()`方法，如下所示：

```cs
static void Main(string[] args)
{
    CompareExchangeVariables();
    AddVariables();
    DecrementVariable();
    IncrementVariable();
    ReadVariable();
    PerformUnsafeCodeSafely();
}
```

我们的`Main()`方法调用测试`Interlocked`方法的方法。运行代码，你应该看到类似以下的东西：

![](img/a355f2b5-6574-4bd0-8168-9f276553d81d.png)

我们现在将讨论一些一般性建议。

# 一般建议

在这最后一节中，我们将看一下微软针对多线程应用的一些一般性建议。它们包括以下内容：

+   避免使用`Thread.Abort`来终止其他线程。

+   使用互斥体、`ManualResetEvent`、`AutoResetEvent`和`Monitor`来同步多个线程之间的活动。

+   在可能的情况下，为你的工作线程使用线程池。

+   如果有任何工作线程被阻塞，那么使用`Monitor.PulseAll`来通知所有线程工作线程状态的改变。

+   避免使用这个，类型实例和字符串实例，包括字符串字面量作为`lock`对象。避免使用`lock`对象的类型。

+   实例锁可能导致死锁，因此在使用时要小心。

+   对于进入监视器的线程，使用`try/finally`块，以便在`finally`块中，通过调用`Monitor.Exit()`确保线程离开监视器。

+   为不同的资源使用不同的线程。

+   避免将多个线程分配给同一资源。

+   I/O 任务应该有自己的线程，因为它们在执行 I/O 操作时会阻塞。这样，你可以让其他线程运行。

+   用户输入应该有自己专用的线程。

+   通过使用`System.Threading.Interlocked`类的方法来改进简单状态改变的性能，而不是使用锁语句。

+   对于频繁使用的代码，避免同步，因为它可能导致死锁和竞争条件。

+   默认情况下，使静态数据线程安全。

+   实例数据默认情况下不是线程安全的；否则，会降低性能，增加锁竞争，并引入可能发生竞争条件和死锁的可能性。

+   避免使用会改变状态的静态方法，因为它们会导致线程错误。

这就结束了我们对线程和并发性的探讨。让我们总结一下我们学到的内容。

# 摘要

在本章中，我们介绍了什么是线程以及如何使用它。我们看到了死锁和竞争条件问题的实际情况，并了解了如何使用锁语句和 TPL 库来防止这些特殊情况。我们还讨论了静态构造函数、静态方法、不可变对象和可变对象的线程安全性。我们看到了为什么使用不可变对象是在线程之间传输数据的线程安全方式，并回顾了一些与线程工作相关的一般建议。

我们还看到了使代码线程安全可以带来很多好处。在下一章中，我们将看一下设计有效的 API。但现在，您可以通过回答以下问题来测试自己的知识，并通过参考提供的链接来进一步阅读。

# 问题

1.  什么是线程？

1.  单线程应用中有多少个线程？

1.  有哪些类型的线程？

1.  哪个线程会在程序退出时立即终止？

1.  哪个线程会一直运行直到完成，即使程序退出了？

1.  什么代码可以让线程休眠半毫秒？

1.  如何实例化一个调用名为`Method1`的方法的线程？

1.  如何将线程设置为后台线程？

1.  什么是死锁？

1.  如何退出使用`Monitor.TryEnter(objectName)`获取的锁？

1.  如何从死锁中恢复？

1.  什么是竞争条件？

1.  防止竞争条件的一种方法是什么？

1.  什么使得静态方法不安全？

1.  静态构造函数是否线程安全？

1.  谁负责管理一组线程？

1.  什么是不可变对象？

1.  为什么在线程应用中更喜欢不可变对象而不是可变对象？

# 进一步阅读

+   [`www.c-sharpcorner.com/blogs/mutex-and-semaphore-in-thread`](https://www.c-sharpcorner.com/blogs/mutex-and-semaphore-in-thread) 提供了使用互斥体和信号量的示例。

+   [`www.guru99.com/mutex-vs-semaphore.html`](https://www.guru99.com/mutex-vs-semaphore.html) 解释了互斥体和信号量之间的区别。

+   [`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-constructors`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-constructors) 是官方的微软静态构造函数文档。

+   [`docs.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices`](https://docs.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices) 是微软官方关于托管线程最佳实践的指南。

+   [`docs.microsoft.com/en-us/dotnet/standard/parallel-programming/task-parallel-library-tpl`](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/task-parallel-library-tpl) 是 TPL 的官方微软 API 文档。

+   [`www.c-sharpcorner.com/UploadFile/1d42da/interlocked-class-in-C-Sharp-threading/`](https://www.c-sharpcorner.com/UploadFile/1d42da/interlocked-class-in-C-Sharp-threading/) 讲解了 C#线程中的 Interlocked 类。

+   [`geekswithblogs.net/BlackRabbitCoder/archive/2012/08/23/c.net-little-wonders-interlocked-read-and-exchange.aspx`](http://geekswithblogs.net/BlackRabbitCoder/archive/2012/08/23/c.net-little-wonders-interlocked-read-and-exchange.aspx) 提供了关于`System.Threading.Interlocked`的讨论和示例。

+   [`www.albahari.com/threading/`](http://www.albahari.com/threading/) 是 Joseph Albahari 关于 C#中线程的免费电子书的链接。

+   [`docs.microsoft.com/en-us/dotnet/api/system.collections.immutable?view=netcore-3.1`](https://docs.microsoft.com/en-us/dotnet/api/system.collections.immutable?view=netcore-3.1) 是微软官方文档，介绍了`System.Collections.Immutable`命名空间中可用的不可变集合。
