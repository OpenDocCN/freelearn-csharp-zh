# 第二章。线程同步

在本章中，我们将描述一些处理多个线程共享资源的常见技术。您将了解：

+   执行基本的原子操作

+   使用 Mutex 构造

+   使用 SemaphoreSlim 构造

+   使用 AutoResetEvent 构造

+   使用 ManualResetEventSlim 构造

+   使用 CountDownEvent 构造

+   使用 Barrier 构造

+   使用 ReaderWriterLockSlim 构造

+   使用 SpinWait 构造

# 介绍

正如我们在第一章 *线程基础*中看到的那样，同时从多个线程使用共享对象是有问题的。非常重要的是同步这些线程，以便它们按适当的顺序对共享对象执行操作。在多线程计数器示例中，我们遇到了一个称为竞争条件的问题。这是因为多个线程的执行没有得到适当的同步。当一个线程执行增量和减量操作时，其他线程必须等待它们的轮到。这个一般问题通常被称为**线程同步**。

有几种方法可以实现线程同步。首先，如果没有共享对象，就根本不需要同步。令人惊讶的是，我们经常可以通过重新设计程序并消除共享状态来摆脱复杂的同步构造。如果可能的话，尽量避免多个线程使用单个对象。

如果我们必须有共享状态，第二种方法是只使用**原子**操作。这意味着一个操作需要一个时间量并立即完成，因此在第一个操作完成之前，没有其他线程可以执行另一个操作。因此，没有必要让其他线程等待此操作完成，也没有必要使用锁；这反过来排除了死锁的情况。

如果这不可能，程序逻辑更复杂，那么我们必须使用不同的构造来协调线程。其中一组构造将等待线程置于**阻塞**状态。在阻塞状态下，线程使用尽可能少的 CPU 时间。然而，这意味着它将至少包括一个所谓的**上下文切换** - 操作系统的线程调度程序将保存等待线程的状态，并切换到另一个线程，轮流恢复其状态。这需要大量资源；但是，如果线程将被暂停很长时间，这是好的。这些构造也被称为**内核模式**构造，因为只有操作系统的内核能够阻止线程使用 CPU 时间。

如果我们必须等待很短的时间，最好是简单地等待而不是将线程切换到阻塞状态。这将节省我们上下文切换的开销，但会浪费一些 CPU 时间，因为线程在等待时会浪费一些 CPU 时间。这些构造被称为**用户模式**构造。它们非常轻量级和快速，但在线程必须长时间等待时会浪费大量 CPU 时间。

为了兼顾两者的优点，有**混合**构造；这些构造首先尝试使用用户模式等待，然后如果线程等待足够长的时间，它将切换到阻塞状态，节省 CPU 资源。

在本章中，我们将研究线程同步的各个方面。我们将介绍如何执行原子操作以及如何使用.NET 框架中包含的现有同步构造。

# 执行基本的原子操作

这个示例将向您展示如何对对象执行基本的原子操作，以防止竞争条件而不阻塞线程。

## 准备工作

要通过本示例，您需要 Visual Studio 2012\. 没有其他先决条件。此示例的源代码可以在`7644_Code\Chapter2\Recipe1`中找到。

## 如何做...

为了理解基本的原子操作，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
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
  public int Count { get { return _count; } }

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

  public int Count { get { return _count; } }

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
Console.WriteLine("Incorrect counter");

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

Console.WriteLine("Total count: {0}", c.Count);
Console.WriteLine("--------------------------");
Console.WriteLine("Correct counter");

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
Console.WriteLine("Total count: {0}", c1.Count);
```

1.  运行程序。

## 工作原理...

当程序运行时，它会创建三个线程，这些线程将执行`TestCounter`方法中的代码。该方法在对象上运行一系列的增量/减量操作。最初，`Counter`对象是不安全的，我们在这里遇到了竞争条件。因此，在第一种情况下，计数器值是不确定的。我们可能会得到一个零值；然而，如果你多次运行程序，最终会得到一些不正确的非零结果。

在第一章中，*线程基础*，我们通过锁定对象来解决了这个问题，导致其他线程在一个线程获取旧计数器值时被阻塞，然后计算并将新值分配给计数器。然而，如果我们以这种方式执行这个操作，它是无法在中途停止的；我们可以在没有任何锁定的情况下使用`Interlocked`构造来实现正确的结果。它提供了原子方法`Increment`、`Decrement`和`Add`用于基本数学运算，并帮助我们编写`Counter`类而不使用锁定。

# 使用 Mutex 构造

这个示例将描述如何使用`Mutex`构造同步两个独立的程序。`Mutex`是一种原始同步，它只允许一个线程独占共享资源。

## 准备工作

要执行这个示例，你需要 Visual Studio 2012。没有其他先决条件。这个示例的源代码可以在`7644_Code\Chapter2\Recipe2`中找到。

## 如何操作...

要理解如何使用`Mutex`构造同步两个独立的程序，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法中，添加以下代码片段：

```cs
const string MutexName = "CSharpThreadingCookbook";

using (var m = new Mutex(false, MutexName))
{
  if (!m.WaitOne(TimeSpan.FromSeconds(5), false))
  {
    Console.WriteLine("Second instance is running!");
  }
  else
  {
    Console.WriteLine("Running!");
    Console.ReadLine();
    m.ReleaseMutex();
  }
}
```

1.  运行程序。

## 工作原理...

当主程序启动时，它使用特定名称定义了一个互斥体，并将`initialOwner`标志设置为`false`。这允许程序在互斥体已经创建时获取互斥体。然后，如果没有获取到互斥体，程序将简单地显示**Running**，并等待按下任意键来释放互斥体并退出。

如果我们启动程序的第二个副本，它将等待 5 秒，尝试获取互斥体。如果我们在程序的第一个副本中按下任意键，第二个副本将开始执行。然而，如果我们继续等待 5 秒，程序的第二个副本将无法获取互斥体。

### 提示

请注意，命名的互斥体是一个全局操作系统对象！始终正确关闭互斥体；最好的选择是使用块来包装互斥体对象。

这使得在不同程序中同步线程成为可能，这在许多场景下都是有用的。

# 使用 SemaphoreSlim 构造

这个示例将展示如何`SemaphoreSlim`是`Semaphore`的轻量级版本；它限制了可以同时访问资源的线程数量。

## 准备工作

要执行这个示例，你需要 Visual Studio 2012。没有其他先决条件。这个示例的源代码可以在`BookSamples\Chapter2\Recipe3`中找到。

## 如何操作...

要理解如何使用`SemaphoreSlim`构造限制对资源的多线程访问，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下方，添加以下代码片段：

```cs
static SemaphoreSlim _semaphore = new SemaphoreSlim(4);
```

```cs
static void AccessDatabase(string name, int seconds)
{
  Console.WriteLine("{0} waits to access a database", name);
  _semaphore.Wait();
  Console.WriteLine("{0} was granted an access to a database",name);
  Thread.Sleep(TimeSpan.FromSeconds(seconds));
  Console.WriteLine("{0} is completed", name);
  _semaphore.Release();

}
```

1.  在`Main`方法中，添加以下代码片段：

```cs
for (int i = 1; i <= 6; i++)
{
  string threadName = "Thread " + i;
  int secondsToWait = 2 + 2*i;
  var t = new Thread(() => AccessDatabase(threadName, secondsToWait));
  t.Start();
}
```

1.  运行程序。

## 工作原理...

当主程序启动时，它创建了一个`SemaphoreSlim`实例，在其构造函数中指定了允许的并发线程数。然后启动六个具有不同名称和启动时间的线程来运行。

每个线程都试图访问数据库，但我们通过信号量限制了对数据库的并发访问数量为四个线程。当四个线程访问数据库时，其他两个线程将等待，直到先前的一个线程完成其工作并通过调用`_semaphore.Release`方法发出信号。

## 还有更多...

在这里，我们使用了一个混合构造，它允许我们在等待时间较短的情况下节省上下文切换。然而，这个构造的旧版本称为`Semaphore`。这个版本是一个纯的内核时间构造。除了一个非常重要的场景之外，没有使用它的意义；我们可以创建一个命名信号量，就像创建一个命名互斥体一样，并且用它来同步不同程序中的线程。`SemaphoreSlim`不使用 Windows 内核信号量，也不支持进程间同步，因此在这种情况下使用`Semaphore`。

# 使用 AutoResetEvent 构造

在本教程中，有一个示例，说明如何使用`AutoResetEvent`构造从一个线程向另一个线程发送通知。`AutoResetEvent`通知等待的线程事件已发生。

## 准备工作

要按照本教程进行操作，您需要 Visual Studio 2012。没有其他先决条件。本教程的源代码可以在`7644_Code\Chapter2\Recipe4`中找到。

## 如何做...

要理解如何使用`AutoResetEvent`构造从一个线程向另一个线程发送通知，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下面，添加以下代码片段：

```cs
private static AutoResetEvent _workerEvent = newAutoResetEvent(false);
private static AutoResetEvent _mainEvent = newAutoResetEvent(false);

static void Process(int seconds)
{
  Console.WriteLine("Starting a long running work...");
  Thread.Sleep(TimeSpan.FromSeconds(seconds));
  Console.WriteLine("Work is done!");
  _workerEvent.Set();
  Console.WriteLine("Waiting for a main thread to completeits work");
  _mainEvent.WaitOne();
  Console.WriteLine("Starting second operation...");
  Thread.Sleep(TimeSpan.FromSeconds(seconds));
  Console.WriteLine("Work is done!");
  _workerEvent.Set();
}
```

1.  在`Main`方法内部，添加以下代码片段：

```cs
var t = new Thread(() => Process(10));
t.Start();

Console.WriteLine("Waiting for another thread to completework");
_workerEvent.WaitOne();
Console.WriteLine("First operation is completed!");
Console.WriteLine("Performing an operation on a mainthread");
Thread.Sleep(TimeSpan.FromSeconds(5));
_mainEvent.Set();
Console.WriteLine("Now running the second operation on asecond thread");
_workerEvent.WaitOne();
Console.WriteLine("Second operation is completed!");
```

1.  运行程序。

## 它是如何工作的...

当主程序启动时，它定义了两个`AutoResetEvent`实例。其中一个是从第二个线程向主线程发出信号的，另一个是从主线程向第二个线程发出信号的。我们在`AutoResetEvent`构造函数中提供`false`，指定了这两个实例的初始状态为`未发出信号`。这意味着调用这些对象中的一个的`WaitOne`方法的任何线程都将被阻塞，直到我们调用`Set`方法。如果我们将事件状态初始化为`true`，它将变为`发出信号`，然后第三个调用`WaitOne`的线程将立即继续。然后事件状态会自动变为`未发出信号`，因此我们需要再次调用`Set`方法，以便让其他线程调用这个实例上的`WaitOne`方法继续。

然后我们创建一个第二个线程，它将执行第一个操作 10 秒，并等待来自第二个线程的信号。信号意味着第一个操作已完成。现在第二个线程正在等待来自主线程的信号。我们在主线程上做一些额外的工作，并通过调用`_mainEvent.Set`方法发送一个信号。然后我们等待来自第二个线程的另一个信号。

`AutoResetEvent`是一个内核时间构造，因此如果等待时间不重要，最好使用下一个使用`ManualResetEventslim`的教程，这是一个混合构造。

# 使用 ManualResetEventSlim 构造

本教程将描述如何使用`ManualResetEventSlim`构造使线程之间的信号更加灵活。

## 准备工作

要按照本教程进行操作，您需要 Visual Studio 2012。没有其他先决条件。本教程的源代码可以在`BookSamples\Chapter2\Recipe5`中找到。

## 如何做...

要理解`ManualResetEventSlim`构造的使用，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下方，添加以下代码：

```cs
static ManualResetEventSlim _mainEvent = new ManualResetEventSlim(false);

static void TravelThroughGates(string threadName,int seconds)
{
  Console.WriteLine("{0} falls to sleep", threadName);
  Thread.Sleep(TimeSpan.FromSeconds(seconds));
  Console.WriteLine("{0} waits for the gates to open!",threadName);
  _mainEvent.Wait();
  Console.WriteLine("{0} enters the gates!", threadName);
}
```

1.  在`Main`方法中，添加以下代码：

```cs
var t1 = new Thread(() => TravelThroughGates("Thread 1",5));
var t2 = new Thread(() => TravelThroughGates("Thread 2",6));
var t3 = new Thread(() => TravelThroughGates("Thread 3",12));
t1.Start();
t2.Start();
t3.Start();
Thread.Sleep(TimeSpan.FromSeconds(6));
Console.WriteLine("The gates are now open!");
_mainEvent.Set();
Thread.Sleep(TimeSpan.FromSeconds(2));
_mainEvent.Reset();
Console.WriteLine("The gates have been closed!");
Thread.Sleep(TimeSpan.FromSeconds(10));
Console.WriteLine("The gates are now open for the secondtime!");
_mainEvent.Set();
Thread.Sleep(TimeSpan.FromSeconds(2));
Console.WriteLine("The gates have been closed!");
_mainEvent.Reset();
```

1.  运行程序。

## 工作原理...

当主程序启动时，首先创建`ManualResetEventSlim`构造的实例。然后我们启动三个线程，它们将等待此事件发出信号以继续执行。

使用此构造的整个过程就像让人们通过一个大门。我们在上一个示例中看到的`AutoResetEvent`事件就像一个旋转门，一次只允许一个人通过。`ManualResetEventSlim`是`ManualResetEvent`的混合版本，直到我们手动调用`Reset`方法之前都保持打开。回到代码，当我们调用`_mainEvent.Set`时，我们打开它并允许准备接受此信号并继续工作的线程。然而，第三个线程仍在休眠，来不及。我们调用`_mainEvent.Reset`，因此关闭它。最后一个线程现在准备好继续，但必须等待下一个信号，这将在几秒钟后发生。

## 还有更多...

与之前的某个示例一样，我们使用了一个混合构造，它缺乏在操作系统级别工作的可能性。如果我们需要全局事件，我们应该使用`EventWaitHandle`构造，它是`AutoResetEvent`和`ManualResetEvent`的基类。

# 使用 CountDownEvent 构造

本示例将描述如何使用`CountdownEvent`信号构造等待直到某个操作完成。

## 准备工作

要执行此示例，您需要 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`BookSamples\Chapter2\Recipe6`中找到。

## 如何做...

要理解`CountDownEvent`构造的使用，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下方，添加以下代码：

```cs
static CountdownEvent _countdown = new CountdownEvent(2);

static void PerformOperation(string message, int seconds)
{
  Thread.Sleep(TimeSpan.FromSeconds(seconds));
  Console.WriteLine(message);
  _countdown.Signal();
}
```

1.  在`Main`方法中，添加以下代码：

```cs
Console.WriteLine("Starting two operations");
var t1 = new Thread(() => PerformOperation("Operation 1 iscompleted", 4));
var t2 = new Thread(() => PerformOperation("Operation 2 iscompleted", 8));
t1.Start();
t2.Start();
_countdown.Wait();
Console.WriteLine("Both operations have been completed.");
_countdown.Dispose();
```

1.  运行程序。

## 工作原理...

当主程序启动时，我们创建一个新的`CountdownEvent`实例，在其构造函数中指定我们希望在两个操作完成时发出信号。然后我们启动两个线程，在完成时向事件发出信号。一旦第二个线程完成，主线程就会从等待`CountdownEvent`中返回并继续进行。使用此构造，非常方便等待多个异步操作完成。

然而，有一个重大缺点；如果我们未能调用所需次数的`_countdown.Signal()`，`_countdown.Wait()`将永远等待。在使用`CountdownEvent`时，请确保所有线程都使用`Signal`方法调用完成。

# 使用屏障构造

这个示例说明了另一个有趣的同步构造，称为`Barrier`。`Barrier`构造有助于组织多个线程在某个时间点相遇，并提供一个回调，每当线程调用`SignalAndWait`方法时都会执行。

## 准备工作

要执行此示例，您需要 Visual Studio 2012。没有其他先决条件。此示例的源代码可以在`BookSamples\Chapter2\Recipe7`中找到。

## 如何做...

要理解`Barrier`构造的使用，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下方，添加以下代码：

```cs
static Barrier _barrier = new Barrier(2, 
  b => Console.WriteLine("End of phase {0}",b.CurrentPhaseNumber + 1));

static void PlayMusic(string name, string message,int seconds)
{
  for (int i = 1; i < 3; i++)
  {
    Console.WriteLine("----------------------------------------------");
    Thread.Sleep(TimeSpan.FromSeconds(seconds));
    Console.WriteLine("{0} starts to {1}", name, message);
    Thread.Sleep(TimeSpan.FromSeconds(seconds));
    Console.WriteLine("{0} finishes to {1}", name,message);
    _barrier.SignalAndWait();
  }
}
```

1.  在`Main`方法中，添加以下代码：

```cs
var t1 = new Thread(() => PlayMusic("the guitarist","play an amazing solo", 5));
var t2 = new Thread(() => PlayMusic("the singer","sing his song", 2));

t1.Start();
t2.Start();
```

1.  运行程序。

## 工作原理...

我们创建了一个`Barrier`构造，指定我们要同步两个线程，并且在这两个线程中的每个调用了`_barrier.SignalAndWait`方法后，我们需要执行一个回调，打印出完成的阶段数。

每个线程将向`Barrier`发送信号两次，因此我们将有两个阶段。每当两个线程都调用`SignalAndWait`方法时，`Barrier`将执行回调。这对于使用多线程迭代算法进行工作很有用，以在每次迭代结束时执行一些计算。当最后一个线程调用`SignalAndWait`方法时，迭代结束。

# 使用“ReaderWriterLockSlim”构造

本教程将描述如何使用`ReaderWriterLockSlim`构造创建一个线程安全的机制，以从多个线程读取和写入集合。`ReaderWriterLockSlim`表示用于管理对资源的访问的锁，允许多个线程进行读取或独占访问进行写入。

## 准备工作

要执行本教程，您将需要 Visual Studio 2012。没有其他先决条件。本教程的源代码可以在`BookSamples\Chapter2\Recipe8`中找到。

## 如何做...

了解如何创建一个线程安全的机制，以使用`ReaderWriterLockSlim`构造从多个线程读取和写入集合。

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Collections.Generic;
using System.Threading;
```

1.  在`Main`方法下面，添加以下代码：

```cs
static ReaderWriterLockSlim _rw = newReaderWriterLockSlim();
static Dictionary<int, int> _items =new Dictionary<int, int>();

static void Read()
{
  Console.WriteLine("Reading contents of a dictionary");
  while (true)
  {
    try
    {
      _rw.EnterReadLock();
      foreach (var key in _items.Keys)
      {
        Thread.Sleep(TimeSpan.FromSeconds(0.1));
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
          Console.WriteLine("New key {0} is added to adictionary by a {1}", newKey, threadName);
        }
        finally
        {
          _rw.ExitWriteLock();
        }
      }
      Thread.Sleep(TimeSpan.FromSeconds(0.1));
    }
    finally
    {
      _rw.ExitUpgradeableReadLock();
    }
  }
}
```

1.  在`Main`方法中，添加以下代码：

```cs
new Thread(Read){ IsBackground = true }.Start();
new Thread(Read){ IsBackground = true }.Start();
new Thread(Read){ IsBackground = true }.Start();

new Thread(() => Write("Thread 1")){ IsBackground =true }.Start();
new Thread(() => Write("Thread 2")){ IsBackground =true }.Start();

Thread.Sleep(TimeSpan.FromSeconds(30)); 
```

1.  运行程序。

## 工作原理...

当主程序启动时，它同时运行三个从字典中读取数据的线程和两个向该字典中写入一些数据的线程。为了实现线程安全，我们使用了专为这种情况设计的`ReaderWriterLockSlim`构造。

它有两种锁：读取锁允许多个线程读取，写入锁阻止其他线程的每个操作，直到释放此写入锁。还有一个有趣的场景，当我们获取读取锁，从集合中读取一些数据，并根据该数据决定获取写入锁并更改集合。如果我们立即获得写锁，会花费太多时间，不允许我们的读取器读取数据，因为当我们获取写锁时，集合被阻塞。为了最小化这段时间，有`EnterUpgradeableReadLock`/`ExitUpgradeableReadLock`方法。我们获取读取锁并读取数据；如果我们发现我们必须更改底层集合，我们只需使用`EnterWriteLock`方法升级我们的锁，然后快速执行写操作，并使用`ExitWriteLock`释放写锁。

在我们的情况下，我们得到一个随机数；然后我们获取一个读取锁，并检查该数字是否存在于字典键集合中。如果不存在，我们将升级我们的锁为写锁，然后将这个新键添加到字典中。最好使用`try`/`finally`块来确保我们在获取锁后始终释放锁。

我们的所有线程都已创建为后台线程，并在等待 30 秒后，主线程以及所有后台线程都完成。

# 使用 SpinWait 构造

本教程将描述如何在不涉及内核模式构造的情况下等待线程。此外，我们介绍了`SpinWait`，这是一种混合同步构造，旨在在用户模式下等待一段时间，然后切换到内核模式以节省 CPU 时间。

## 准备工作

要执行本教程，您将需要 Visual Studio 2012。没有其他先决条件。本教程的源代码可以在`BookSamples\Chapter2\Recipe9`中找到。

## 如何做...

了解在不涉及内核模式构造的情况下等待线程，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Threading;
```

1.  在`Main`方法下面，添加以下代码：

```cs
static volatile bool _isCompleted = false;

static void UserModeWait()
{
  while (!_isCompleted)
  {
    Console.Write(".");
  }
  Console.WriteLine();
  Console.WriteLine("Waiting is complete");
}

static void HybridSpinWait()
{
  var w = new SpinWait();
  while (!_isCompleted)
  {
    w.SpinOnce();
    Console.WriteLine(w.NextSpinWillYield);
  }
  Console.WriteLine("Waiting is complete");
}
```

1.  在`Main`方法中，添加以下代码：

```cs
var t1 = new Thread(UserModeWait);
var t2 = new Thread(HybridSpinWait);

Console.WriteLine("Running user mode waiting");
t1.Start();
Thread.Sleep(20);
_isCompleted = true;
Thread.Sleep(TimeSpan.FromSeconds(1));
_isCompleted = false;
Console.WriteLine("Running hybrid SpinWait constructwaiting");
t2.Start();
Thread.Sleep(5);
_isCompleted = true;
```

1.  运行程序。

## 工作原理...

当主程序启动时，它定义了一个线程，该线程将执行一个无限循环，每 20 毫秒一次，直到主线程将`_isCompleted`变量设置为`true`。我们可以尝试将这个循环运行 20-30 秒，使用 Windows 任务管理器来测量 CPU 负载。这将显示出相当大量的处理器时间，取决于 CPU 有多少个核心。

我们使用`volatile`关键字来声明`_isCompleted`静态字段。`volatile`关键字表示一个字段可能会被多个线程同时修改。声明为`volatile`的字段不受编译器和处理器优化的影响，这些优化假定只有一个线程访问。这确保了字段中始终存在最新的值。

然后我们使用`SpinWait`版本，每次迭代都打印一个特殊的标志，显示线程是否将切换到阻塞状态。我们运行这个线程 5 毫秒来观察。在开始时，`SpinWait`试图保持在用户模式下，大约经过九次迭代后，它开始将线程切换到阻塞状态。如果我们尝试使用这个版本来测量 CPU 负载，我们将在 Windows 任务管理器中看不到任何 CPU 使用率。
