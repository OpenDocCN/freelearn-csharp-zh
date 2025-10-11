# 第二章. 线程同步

在本章中，我们将描述一些从多个线程处理共享资源的常见技术。你将学习以下技巧：

+   执行基本的原子操作

+   使用`Mutex`结构

+   使用`SemaphoreSlim`结构

+   使用`AutoResetEvent`结构

+   使用`ManualResetEventSlim`结构

+   使用`CountDownEvent`结构

+   使用`Barrier`结构

+   使用`ReaderWriterLockSlim`结构

+   使用`SpinWait`结构

# 简介

如我们在第一章中看到的，*线程基础*，从几个线程同时使用共享对象是有问题的。然而，同步这些线程以使它们按正确顺序对共享对象执行操作非常重要。在*使用 C# lock 关键字进行锁定*技巧中，我们遇到了一个称为竞争条件的问题。问题发生是因为这些多个线程的执行没有正确同步。当一个线程执行增加和减少操作时，其他线程必须等待它们的轮次。以这种方式组织线程通常被称为**线程同步**。

实现线程同步有几种方法。首先，如果没有共享对象，就根本不需要同步。令人惊讶的是，我们经常可以通过重新设计我们的程序并删除共享状态来消除复杂的同步结构。如果可能的话，只需避免从多个线程使用单个对象。

如果我们必须有一个共享状态，第二种方法就是只使用**原子**操作。这意味着一个操作只占用一个时间量子并立即完成，因此其他线程在第一个操作完成之前不能执行另一个操作。因此，没有必要让其他线程等待这个操作完成，也不需要使用锁；这反过来，排除了死锁的情况。

如果这不可能，并且程序的逻辑更复杂，那么我们必须使用不同的结构来协调线程。这些结构中的一组将等待的线程置于`阻塞`状态。在`阻塞`状态下，线程尽可能少地使用 CPU 时间。然而，这意味着它至少会包含一个所谓的**上下文切换**——操作系统的线程调度器将保存等待线程的状态并切换到另一个线程，然后依次恢复其状态。这需要相当多的资源；然而，如果线程将要长时间挂起，这是好的。这类结构也被称为**内核模式**结构，因为只有操作系统的内核能够停止线程使用 CPU 时间。

如果我们必须等待一段时间，最好是简单地等待而不是将线程切换到`阻塞`状态。这将节省我们上下文切换的开销，同时线程等待时会浪费一些 CPU 时间。这样的构造被称为**用户模式**构造。它们非常轻量级且快速，但当一个线程需要长时间等待时，会浪费大量的 CPU 时间。

为了充分利用两者的优点，存在**混合**构造；这些尝试首先使用用户模式等待，然后，如果一个线程等待足够长的时间，它将切换到`阻塞`状态，从而节省 CPU 资源。

在本章中，我们将探讨线程同步的各个方面。我们将介绍如何执行原子操作，以及如何使用.NET Framework 中包含的现有同步构造。

# 执行基本原子操作

此配方将向您展示如何对一个对象执行基本原子操作，以防止竞态条件而不阻塞线程。

## 准备工作

要逐步执行此配方，您需要 Visual Studio 2015。没有其他先决条件。此配方的源代码可以在`BookSamples\Chapter2\Recipe1`中找到。

## 如何操作...

要理解基本原子操作，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Threading;
    using static System.Console;
    ```

1.  在`Main`方法下方，添加以下代码片段：

    ```cs
    static void TestCounter(CounterBase c)
    {
      for (int i = 0; i < 100000; i++)
      {
        c.Increment();
        c.Decrement();
      }
    }

    class Counter : CounterBase
    {
      private int _count;

      public int Count => _count;

      public override void Increment()
      {
        _count++;
      }

      public override void Decrement()
      {
        _count--;
      }
    }

    class CounterNoLock : CounterBase
    {
      private int _count;

      public int Count => _count;

      public override void Increment()
      {
        Interlocked.Increment(ref _count);
      }

      public override void Decrement()
      {
        Interlocked.Decrement(ref _count);
      }
    }

    abstract class CounterBase
    {
      public abstract void Increment();

      public abstract void Decrement();
    }
    ```

1.  在`Main`方法中，添加以下代码片段：

    ```cs
    WriteLine("Incorrect counter");

    var c = new Counter();

    var t1 = new Thread(() => TestCounter(c));
    var t2 = new Thread(() => TestCounter(c));
    var t3 = new Thread(() => TestCounter(c));
    t1.Start();
    t2.Start();
    t3.Start();
    t1.Join();
    t2.Join();
    t3.Join();

    WriteLine($"Total count: {c.Count}");
    WriteLine("--------------------------");

    WriteLine("Correct counter");

    var c1 = new CounterNoLock();

    t1 = new Thread(() => TestCounter(c1));
    t2 = new Thread(() => TestCounter(c1));
    t3 = new Thread(() => TestCounter(c1));
    t1.Start();
    t2.Start();
    t3.Start();
    t1.Join();
    t2.Join();
    t3.Join();

    WriteLine($"Total count: {c1.Count}");
    ```

1.  运行程序。

## 它是如何工作的...

当程序运行时，它创建三个线程，这些线程将在`TestCounter`方法中执行代码。此方法在对象上运行一系列增加/减少操作。最初，`Counter`对象不是线程安全的，我们在这里得到竞态条件。因此，在第一种情况下，计数器的值不是确定的。我们可能会得到零值；然而，如果您多次运行程序，最终会得到一些不正确的非零结果。

在第一章，*线程基础*中，我们通过锁定我们的对象来解决这个问题，导致其他线程被阻塞，而一个线程获取旧的计数器值，然后计算并分配新的计数器值。然而，如果我们以这种方式执行此操作，它不能中途停止，我们可以在没有任何锁定的情况下实现正确的结果，这可以通过`Interlocked`构造来实现。它提供了`Increment`、`Decrement`和`Add`原子方法用于基本数学运算，并帮助我们编写`Counter`类而不使用锁定。

# 使用 Mutex 构造

此配方将描述如何使用`Mutex`构造同步两个不同的程序。`Mutex`构造是一个同步原语，它只允许一个线程对共享资源进行独占访问。

## 准备工作

要逐步执行此配方，您需要 Visual Studio 2015。没有其他先决条件。此配方的源代码可以在`BookSamples\Chapter2\Recipe2`中找到。

## 如何操作...

要了解如何使用`Mutex`结构同步两个独立程序，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Threading;
    using static System.Console;
    ```

1.  在`Main`方法内部，添加以下代码片段：

    ```cs
    const string MutexName = "CSharpThreadingCookbook";

    using (var m = new Mutex(false, MutexName))
    {
      if (!m.WaitOne(TimeSpan.FromSeconds(5), false))
      {
        WriteLine("Second instance is running!");
      }
      else
      {
        WriteLine("Running!");
        ReadLine();
        m.ReleaseMutex();
      }
    }
    ```

1.  运行程序。

## 它是如何工作的...

当主程序启动时，它使用特定名称定义一个互斥锁，并将`initialOwner`标志设置为`false`。这允许程序在互斥锁已创建的情况下获取互斥锁。然后，如果没有获取到互斥锁，程序将简单地显示**运行中**并等待按下任意键以释放互斥锁并退出。

如果我们启动程序的第二个副本，它将等待 5 秒钟，试图获取互斥锁。如果我们按下第一个程序副本中的任意键，第二个程序将开始执行。然而，如果我们继续等待 5 秒钟，程序的第二副本将无法获取互斥锁。

### 小贴士

注意，互斥锁是一个全局操作系统对象！始终正确关闭互斥锁；最佳选择是将互斥锁对象包装在`using`块中。

这使得在不同程序中同步线程成为可能，这在许多场景中可能很有用。

# 使用`SemaphoreSlim`结构

此配方将向您展示如何使用`SemaphoreSlim`结构限制对某些资源的多线程访问。`SemaphoreSlim`是`Semaphore`的一个轻量级版本；它限制了可以并发访问资源的线程数。

## 准备工作

要逐步执行此配方，您需要 Visual Studio 2015。没有其他先决条件。此配方的源代码可以在`BookSamples\Chapter2\Recipe3`中找到。

## 如何操作...

要了解如何使用`SemaphoreSlim`结构限制对资源的多线程访问，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方，添加以下代码片段：

    ```cs
    static SemaphoreSlim _semaphore = new SemaphoreSlim(4);

    static void AccessDatabase(string name, int seconds)
    {
      WriteLine($"{name} waits to access a database");
      _semaphore.Wait();
      WriteLine($"{name} was granted an access to a database");
      Sleep(TimeSpan.FromSeconds(seconds));
      WriteLine($"{name} is completed");
      _semaphore.Release();
    }
    ```

1.  在`Main`方法内部，添加以下代码片段：

    ```cs
    for (int i = 1; i <= 6; i++)
    {
      string threadName = "Thread " + i;
      int secondsToWait = 2 + 2 * i;
      var t = new Thread(() => AccessDatabase(threadName, secondsToWait));
      t.Start();
    }
    ```

1.  运行程序。

## 它是如何工作的...

当主程序启动时，它创建一个`SemaphoreSlim`实例，并在其构造函数中指定允许的并发线程数。然后，它启动六个具有不同名称和启动时间的线程来运行。

每个线程都试图获取访问数据库的权限，但我们通过使用信号量将并发访问数据库的线程数限制为四个。当四个线程获取到数据库的访问权限时，其他两个线程将等待，直到之前的某个线程完成其工作并通过调用`_semaphore.Release`方法向其他线程发出信号。

## 更多内容...

在这里，我们使用一个混合构造函数，它允许我们在等待时间非常短的情况下节省上下文切换。然而，这个构造函数有一个较老的版本，称为 `Semaphore`。这个版本是一个纯内核时间构造函数。除了在一个非常重要的场景之外，使用它没有意义；我们可以创建一个命名信号量，就像命名互斥锁一样，并使用它来在不同程序中同步线程。`SemaphoreSlim` 不使用 Windows 内核信号量，也不支持进程间同步，所以在这种情况下使用 `Semaphore`。

# 使用 `AutoResetEvent` 构造函数

在此配方中，有一个示例，说明如何使用 `AutoResetEvent` 构造函数从线程发送通知。`AutoResetEvent` 通知等待的线程已发生事件。

## 准备工作

要逐步执行此配方，您需要 Visual Studio 2015。没有其他先决条件。此配方的源代码可在 `BookSamples\Chapter2\Recipe4` 中找到。

## 如何操作...

要了解如何使用 `AutoResetEvent` 构造函数在两个线程之间发送通知，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在 `Main` 方法下方，添加以下代码片段：

    ```cs
    private static AutoResetEvent _workerEvent = new AutoResetEvent(false);
    private static AutoResetEvent _mainEvent = new AutoResetEvent(false);

    static void Process(int seconds)
    {
      WriteLine("Starting a long running work...");
      Sleep(TimeSpan.FromSeconds(seconds));
      WriteLine("Work is done!");
      _workerEvent.Set();
      WriteLine("Waiting for a main thread to complete its work");
      _mainEvent.WaitOne();
      WriteLine("Starting second operation...");
      Sleep(TimeSpan.FromSeconds(seconds));
      WriteLine("Work is done!");
      _workerEvent.Set();
    }
    ```

1.  在 `Main` 方法内部，添加以下代码片段：

    ```cs
    var t = new Thread(() => Process(10));
    t.Start();

    WriteLine("Waiting for another thread to complete work");
    _workerEvent.WaitOne();
    WriteLine("First operation is completed!");
    WriteLine("Performing an operation on a main thread");
    Sleep(TimeSpan.FromSeconds(5));
    _mainEvent.Set();
    WriteLine("Now running the second operation on a second thread");
    _workerEvent.WaitOne();
    WriteLine("Second operation is completed!");
    ```

1.  运行程序。

## 它是如何工作的...

当主程序启动时，它定义了两个 `AutoResetEvent` 实例。其中一个是用于从第二个线程向主线程发送信号，另一个是用于从主线程向第二个线程发送信号。我们将 `false` 传递给 `AutoResetEvent` 构造函数，指定这两个实例的初始状态为 `未发送信号`。这意味着任何调用这些对象之一的 `WaitOne` 方法的线程都将被阻塞，直到我们调用 `Set` 方法。如果我们初始化事件状态为 `true`，它将变为 `已发送信号`，第一个调用 `WaitOne` 的线程将立即继续。然后事件状态将自动变为 `未发送信号`，因此我们需要再次调用 `Set` 方法，以便其他调用此实例上 `WaitOne` 方法的线程继续。

然后，我们创建一个第二个线程，该线程执行第一个操作 10 秒并等待第二个线程的信号。信号通知第一个操作已完成。现在，第二个线程等待来自主线程的信号。我们在主线程上执行一些额外的工作，并通过调用 `_mainEvent.Set` 方法发送信号。然后，我们等待来自第二个线程的另一个信号。

`AutoResetEvent` 是一个内核时间构造函数，所以如果等待时间不显著，最好使用带有 `ManualResetEventslim` 的下一个配方，它是一个混合构造函数。

# 使用 `ManualResetEventSlim` 构造函数

此配方将描述如何使用 `ManualResetEventSlim` 构造函数使线程之间的信号更加灵活。

## 准备工作

要逐步完成这个菜谱，你需要 Visual Studio 2015。没有其他先决条件。这个菜谱的源代码可以在`BookSamples\Chapter2\Recipe5`找到。

## 如何操作...

要理解`ManualResetEventSlim`构造的使用，执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中添加以下`using`指令：

    ```cs
    using System;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码：

    ```cs
    static void TravelThroughGates(string threadName, int seconds)
    {
      WriteLine($"{threadName} falls to sleep");
      Sleep(TimeSpan.FromSeconds(seconds));
      WriteLine($"{threadName} waits for the gates to open!");
      _mainEvent.Wait();
      WriteLine($"{threadName} enters the gates!");
    }

    static ManualResetEventSlim _mainEvent = new ManualResetEventSlim(false);
    ```

1.  在`Main`方法中添加以下代码：

    ```cs
    var t1 = new Thread(() => TravelThroughGates("Thread 1", 5));
    var t2 = new Thread(() => TravelThroughGates("Thread 2", 6));
    var t3 = new Thread(() => TravelThroughGates("Thread 3", 12));
    t1.Start();
    t2.Start();
    t3.Start();
    Sleep(TimeSpan.FromSeconds(6));
    WriteLine("The gates are now open!");
    _mainEvent.Set();
    Sleep(TimeSpan.FromSeconds(2));
    _mainEvent.Reset();
    WriteLine("The gates have been closed!");
    Sleep(TimeSpan.FromSeconds(10));
    WriteLine("The gates are now open for the second time!");
    _mainEvent.Set();
    Sleep(TimeSpan.FromSeconds(2));
    WriteLine("The gates have been closed!");
    _mainEvent.Reset();
    ```

1.  运行程序。

## 工作原理...

当主程序启动时，它首先创建一个`ManualResetEventSlim`构造的实例。然后，我们启动三个线程，等待这个事件信号它们继续执行。

使用这个构造的过程就像让人们通过一个门。我们在之前的菜谱中看到的`AutoResetEvent`事件就像一个旋转门，一次只允许一个人通过。`ManualResetEventSlim`是`ManualResetEvent`的混合版本，它保持开启状态，直到我们手动调用`Reset`方法。回到代码中，当我们调用`_mainEvent.Set`时，我们打开它，允许准备好接受这个信号的线程继续工作。然而，线程三还在睡眠中，没有及时到达。我们调用`_mainEvent.Reset`，因此关闭了它。最后一个线程现在可以继续前进，但它必须等待下一个信号，这个信号将在几秒钟后发生。

## 更多内容...

就像在之前的菜谱中一样，我们使用了一个混合构造，它不具备在操作系统级别工作的可能性。如果我们需要一个全局事件，我们应该使用`EventWaitHandle`构造，它是`AutoResetEvent`和`ManualResetEvent`的基类。

# 使用`CountDownEvent`构造

这个菜谱将描述如何使用`CountdownEvent`信号构造来等待直到一定数量的操作完成。

## 准备工作

要逐步完成这个菜谱，你需要 Visual Studio 2015。没有其他先决条件。这个菜谱的源代码可以在`BookSamples\Chapter2\Recipe6`找到。

## 如何操作...

要理解`CountDownEvent`构造的使用，执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中添加以下`using`指令：

    ```cs
    using System;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码：

    ```cs
    static CountdownEvent _countdown = new CountdownEvent(2);

    static void PerformOperation(string message, int seconds)
    {
      Sleep(TimeSpan.FromSeconds(seconds));
      WriteLine(message);
      _countdown.Signal();
    }
    ```

1.  在`Main`方法中，添加以下代码：

    ```cs
    WriteLine("Starting two operations");
    var t1 = new Thread(() => PerformOperation("Operation 1 is completed", 4));
    var t2 = new Thread(() => PerformOperation("Operation 2 is completed", 8));
    t1.Start();
    t2.Start();
    _countdown.Wait();
    WriteLine("Both operations have been completed.");
    _countdown.Dispose();
    ```

1.  运行程序。

## 工作原理...

当主程序启动时，我们创建一个新的`CountdownEvent`实例，在构造函数中指定我们希望在两个操作完成时发出信号。然后，我们启动两个线程，当它们完成时向事件发出信号。一旦第二个线程完成，主线程从等待`CountdownEvent`中返回，并继续执行。使用这个构造，等待多个异步操作完成非常方便。

然而，存在一个显著的缺点；如果我们未能按照要求次数调用`_countdown.Signal()`，则`_countdown.Wait()`将永远等待。确保在使用`CountdownEvent`时，所有线程都通过`Signal`方法调用完成。

# 使用`Barrier`构造

此配方演示了另一个有趣的同步构造，称为`Barrier`。`Barrier`构造有助于组织多个线程，以便它们在某个时间点相遇，提供一个在每次线程调用`SignalAndWait`方法时执行的回调。

## 准备工作

要逐步执行此配方，您需要 Visual Studio 2015。没有其他先决条件。此配方的源代码可以在`BookSamples\Chapter2\Recipe7`中找到。

## 如何操作...

要理解`Barrier`构造的使用，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方，添加以下代码：

    ```cs
    static Barrier _barrier = new Barrier(2,
      b => WriteLine($"End of phase {b.CurrentPhaseNumber + 1}"));

    static void PlayMusic(string name, string message, int seconds)
    {
      for (int i = 1; i < 3; i++)
      {
        WriteLine("----------------------------------------------");
        Sleep(TimeSpan.FromSeconds(seconds));
        WriteLine($"{name} starts to {message}");
        Sleep(TimeSpan.FromSeconds(seconds));
        WriteLine($"{name} finishes to {message}");
        _barrier.SignalAndWait();
      }
    }
    ```

1.  在`Main`方法内部，添加以下代码：

    ```cs
    var t1 = new Thread(() => PlayMusic("the guitarist", "play an amazing solo", 5));
    var t2 = new Thread(() => PlayMusic("the singer", "sing his song", 2));

    t1.Start();
    t2.Start();
    ```

1.  运行程序。

## 工作原理...

我们创建一个`Barrier`构造，指定我们想要同步两个线程，并且在这两个线程中的每一个调用`_barrier.SignalAndWait`方法之后，我们需要执行一个回调，该回调将打印出已完成的阶段数。

每个线程将向`Barrier`发送两次信号，因此我们将有两个阶段。每次两个线程都调用`SignalAndWait`方法时，`Barrier`将执行回调。这对于与多线程迭代算法一起使用，在每个迭代结束时执行一些计算非常有用。迭代结束时，是最后一个线程调用`SignalAndWait`方法。

# 使用`ReaderWriterLockSlim`构造

此配方将描述如何使用`ReaderWriterLockSlim`构造创建一个线程安全的机制，以便从多个线程中读取和写入集合。`ReaderWriterLockSlim`代表一个锁，用于管理对资源的访问，允许多个线程进行读取或为写入提供独占访问。

## 准备工作

要逐步执行此配方，您需要 Visual Studio 2015。没有其他先决条件。此配方的源代码可以在`BookSamples\Chapter2\Recipe8`中找到。

## 如何操作...

要理解如何使用`ReaderWriterLockSlim`构造来创建一个线程安全的机制，以便从多个线程中读取和写入集合，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Collections.Generic;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方，添加以下代码：

    ```cs
    static ReaderWriterLockSlim _rw = new ReaderWriterLockSlim();
    static Dictionary<int, int> _items = new Dictionary<int, int>();

    static void Read()
    {
      WriteLine("Reading contents of a dictionary");
      while (true)
      {
        try
        {
          _rw.EnterReadLock();
          foreach (var key in _items.Keys)
          {
            Sleep(TimeSpan.FromSeconds(0.1));
          }
        }
        finally
        {
          _rw.ExitReadLock();
        }
      }
    }

    static void Write(string threadName)
    {
      while (true)
      {
        try
        {
          int newKey = new Random().Next(250);
          _rw.EnterUpgradeableReadLock();
          if (!_items.ContainsKey(newKey))
          {
            try
            {
              _rw.EnterWriteLock();
              _items[newKey] = 1;
              WriteLine($"New key {newKey} is added to a dictionary by a {threadName}");
            }
            finally
            {
              _rw.ExitWriteLock();
            }
          }
          Sleep(TimeSpan.FromSeconds(0.1));
        }
        finally
        {
          _rw.ExitUpgradeableReadLock();
        }
      }
    }
    ```

1.  在`Main`方法内部，添加以下代码：

    ```cs
    new Thread(Read){ IsBackground = true }.Start();
    new Thread(Read){ IsBackground = true }.Start();
    new Thread(Read){ IsBackground = true }.Start();

    new Thread(() => Write("Thread 1")){ IsBackground = true }.Start();
    new Thread(() => Write("Thread 2")){ IsBackground = true }.Start();

    Sleep(TimeSpan.FromSeconds(30)); 
    ```

1.  运行程序。

## 工作原理...

当主程序启动时，它同时运行三个线程，从字典中读取数据，以及两个线程将一些数据写入这个字典。为了实现线程安全，我们使用`ReaderWriterLockSlim`构造，它专门为这种场景设计。

它有两种类型的锁：一种读锁允许多个线程读取，一种写锁会阻塞其他线程的所有操作，直到这个写锁被释放。还有一个有趣的场景，当我们获得读锁，从集合中读取一些数据，并根据这些数据决定获得写锁并更改集合。如果我们一次性获得写锁，会花费太多时间，不允许我们的读者读取数据，因为当我们获得写锁时，集合被阻塞。为了最小化这种时间，有`EnterUpgradeableReadLock`/`ExitUpgradeableReadLock`方法。我们获得读锁并读取数据；如果我们发现我们需要更改底层集合，我们只需使用`EnterWriteLock`方法升级我们的锁，然后快速执行写操作，并使用`ExitWriteLock`释放写锁。

在我们的情况下，我们获取一个随机数；然后我们获得一个读锁并检查这个数字是否存在于字典键集合中。如果没有，我们将锁升级为写锁，然后将这个新键添加到字典中。使用`try`/`finally`块确保我们总是在获取锁后释放它们是一个好习惯。

我们创建的所有线程都是作为后台线程创建的，在等待 30 秒后，主线程以及所有后台线程都将完成。

# 使用`SpinWait`构造

这个配方将描述如何在不涉及内核模式构造的情况下等待一个线程。此外，我们引入了`SpinWait`，这是一种混合同步构造，设计用于在用户模式下等待一段时间，然后切换到内核模式以节省 CPU 时间。

## 准备工作

要逐步执行这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter2\Recipe9`中找到。

## 如何做...

要了解如何在不涉及内核模式构造的情况下等待一个线程，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方，添加以下代码：

    ```cs
    static volatile bool _isCompleted = false;

    static void UserModeWait()
    {
      while (!_isCompleted)
      {
        Write(".");
      }
      WriteLine();
      WriteLine("Waiting is complete");
    }

    static void HybridSpinWait()
    {
      var w = new SpinWait();
      while (!_isCompleted)
      {
        w.SpinOnce();
        WriteLine(w.NextSpinWillYield);
      }
      WriteLine("Waiting is complete");
    }
    ```

1.  在`Main`方法内部，添加以下代码：

    ```cs
    var t1 = new Thread(UserModeWait);
    var t2 = new Thread(HybridSpinWait);

    WriteLine("Running user mode waiting");
    t1.Start();
    Sleep(20);
    _isCompleted = true;
    Sleep(TimeSpan.FromSeconds(1));
    _isCompleted = false;
    WriteLine("Running hybrid SpinWait construct waiting");
    t2.Start();
    Sleep(5);
    _isCompleted = true;
    ```

1.  运行程序。

## 它是如何工作的...

当主程序启动时，它会定义一个线程，该线程将执行一个持续 20 毫秒的无穷循环，直到主线程将`_isCompleted`变量设置为`true`。我们可以尝试将这个循环运行 20-30 秒，并用 Windows 任务管理器测量 CPU 负载。它将显示大量的处理器时间，这取决于 CPU 有多少核心。

我们使用`volatile`关键字来声明`_isCompleted`静态字段。`volatile`关键字表示该字段可能被同时执行的多线程修改。被声明为`volatile`的字段不受编译器和处理器优化影响，这些优化假设只有一个线程访问。这确保了字段中始终存在最新的值。

然后，我们使用一个`SpinWait`版本，它在每次迭代中打印一个特殊标志，显示一个线程是否将要切换到`blocked`状态。我们运行这个线程 5 毫秒来观察这一点。一开始，`SpinWait`试图保持在用户模式，大约经过九次迭代后，它开始将线程切换到阻塞状态。如果我们尝试使用这个版本来测量 CPU 负载，我们在 Windows 任务管理器中将看不到任何 CPU 使用情况。
