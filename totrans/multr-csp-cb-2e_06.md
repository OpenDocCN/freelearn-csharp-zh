# 第六章。使用并发集合

在本章中，我们将查看 .NET Framework 基类库中包含的用于并发编程的不同数据结构。你将学习以下食谱：

+   使用 `ConcurrentDictionary`

+   使用 `ConcurrentQueue` 实现异步处理

+   使用 `ConcurrentStack` 改变异步处理顺序

+   使用 `ConcurrentBag` 创建可扩展的爬虫

+   使用 `BlockingCollection` 泛化异步处理

# 简介

编程需要理解和掌握基本数据结构和算法。为了选择最适合并发情况的数据结构，程序员必须了解许多事情，例如算法的时间复杂度、空间复杂度和大 O 符号。在不同的、众所周知的情况下，我们总是知道哪些数据结构更有效。

对于并发计算，我们需要有适当的数据结构。这些数据结构必须是可扩展的，尽可能避免使用锁，并且同时提供线程安全访问。自版本 4 以来，.NET Framework 中的 `System.Collections.Concurrent` 命名空间包含几个数据结构。在本章中，我们将介绍几个数据结构，并展示如何使用它们的非常简单的示例。

让我们从 `ConcurrentQueue` 开始。这个集合使用原子的 **比较和交换** (**CAS**) 操作，这允许我们安全地交换两个变量的值，并使用 `SpinWait` 来确保线程安全。它实现了一个 **先进先出** (**FIFO**) 集合，这意味着项目从队列中退出的顺序与它们被添加到队列中的顺序相同。要将项目添加到队列中，你调用 `Enqueue` 方法。`TryDequeue` 方法尝试从队列中取出第一个项目，而 `TryPeek` 方法尝试获取第一个项目而不从队列中移除它。

`ConcurrentStack` 集合也是在不使用任何锁的情况下，仅通过 CAS 操作实现的。这是一个 **后进先出** (**LIFO**) 集合，这意味着最近添加的项目将首先被返回。要添加项目，你可以使用 `Push` 和 `PushRange` 方法；要检索，你使用 `TryPop` 和 `TryPopRange`，要检查，你可以使用 `TryPeek` 方法。

`ConcurrentBag` 集合是一个无序集合，支持重复项。它针对的场景是多个线程以这种方式划分他们的工作，即每个线程产生并消费自己的任务，很少处理其他线程的任务（在这种情况下，它使用锁）。你使用 `Add` 方法向袋子中添加项目；使用 `TryPeek` 检查，使用 `TryTake` 方法从袋子中取出项目。

### 注意

避免在提到的集合上使用 `Count` 属性。它们使用链表实现，因此 `Count` 是一个 `O(N)` 操作。如果你需要检查集合是否为空，请使用 `IsEmpty` 属性，这是一个 `O(1)` 操作。

`ConcurrentDictionary` 是一个线程安全的字典集合实现。它在读取操作上是无锁的。然而，它需要锁定进行写入操作。并发字典使用多个锁，在字典桶上实现了一个细粒度锁定模型。锁的数量可以通过带有 `concurrencyLevel` 参数的构造函数来定义，这意味着预计将有多个线程并发更新字典。

### 注意

由于并发字典使用锁定，因此有许多操作需要获取字典内部的所有锁。这些操作包括：`Count`、`IsEmpty`、`Keys`、`Values`、`CopyTo` 和 `ToArray`。避免在没有需要的情况下使用这些操作。

`BlockingCollection` 是对 `IProducerConsumerCollection` 泛型接口实现的先进包装。它具有许多更高级的功能，并且在实现具有一些步骤使用前一步骤处理结果的管道场景时非常有用。`BlockingCollection` 类支持诸如阻塞、限制内部集合容量、取消集合操作和从多个阻塞集合中检索值等功能。

并发算法可能非常复杂，要涵盖所有并发集合——无论是更高级还是不那么高级的——都需要编写一本单独的书。在这里，我们仅展示了使用并发集合的最简单示例。

# 使用 ConcurrentDictionary

这个配方展示了在一个单线程环境中，比较常规字典集合与并发字典性能的一个非常简单的场景。

## 准备工作

要完成这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在 `BookSamples\Chapter6\Recipe1` 中找到。

## 如何做到...

要了解常规字典集合与并发字典性能之间的差异，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System.Collections.Concurrent;
    using System.Collections.Generic;
    using System.Diagnostics;
    using static System.Console;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    const string Item = "Dictionary item";
    const int Iterations = 1000000;
    public static string CurrentItem;
    ```

1.  在 `Main` 方法内添加以下代码片段：

    ```cs
    var concurrentDictionary = new ConcurrentDictionary<int, string>();
    var dictionary = new Dictionary<int, string>();

    var sw = new Stopwatch();

    sw.Start();
    for (int i = 0; i < Iterations; i++)
    {
      lock (dictionary)
      {
        dictionary[i] = Item;
      }
    }
    sw.Stop();
    WriteLine($"Writing to dictionary with a lock: {sw.Elapsed}");

    sw.Restart();
    for (int i = 0; i < Iterations; i++)
    {
      concurrentDictionary[i] = Item;
    }
    sw.Stop();
    WriteLine($"Writing to a concurrent dictionary: {sw.Elapsed}");

    sw.Restart();
    for (int i = 0; i < Iterations; i++)
    {
      lock (dictionary)
      {
        CurrentItem = dictionary[i];
      }
    }
    sw.Stop();
    WriteLine($"Reading from dictionary with a lock: {sw.Elapsed}");

    sw.Restart();
    for (int i = 0; i < Iterations; i++)
    {
      CurrentItem = concurrentDictionary[i];
    }
    sw.Stop();
    WriteLine($"Reading from a concurrent dictionary: {sw.Elapsed}");
    ```

1.  运行程序。

## 它是如何工作的...

当程序启动时，我们创建了两个集合。其中一个是标准字典集合，另一个是新创建的并发字典。然后，我们开始向它们添加内容，使用带有锁的标准字典，并测量一百万次迭代完成所需的时间。然后，我们在相同场景下测量 `ConcurrentDictionary` 集合的性能，并最终比较从两个集合中检索值的性能。

在这个非常简单的场景中，我们发现`ConcurrentDictionary`在写入操作上比带有锁的普通字典要慢得多，但在检索操作上要快。因此，如果我们需要从字典中进行许多线程安全的读取，`ConcurrentDictionary`集合是最好的选择。

### 注意

如果你只需要对字典进行只读的多线程访问，可能没有必要执行线程安全的读取。在这种情况下，使用普通的字典或`ReadOnlyDictionary`集合会更好。

`ConcurrentDictionary`集合使用**细粒度锁定**技术实现，这允许它在多个写入操作上比使用带有锁的普通字典（称为**粗粒度锁定**）有更好的扩展性。正如我们在本例中所看到的，当我们只使用一个线程时，并发字典要慢得多，但当我们将其扩展到五到六个线程（如果我们有足够的 CPU 核心可以同时运行它们），并发字典实际上会表现得更好。

# 使用`ConcurrentQueue`实现异步处理

这个配方将向你展示创建一组由多个工作者异步处理的任务的示例。

## 准备工作

为了完成这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在`BookSamples\Chapter6\Recipe2`中找到。

## 如何实现...

要理解创建一组由多个工作者异步处理的任务的工作原理，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Collections.Concurrent;
    using System.Threading;
    using System.Threading.Tasks;
    using static System.Console;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static async Task RunProgram()
    {
      var taskQueue = new ConcurrentQueue<CustomTask>();
      var cts = new CancellationTokenSource();

      var taskSource = Task.Run(() => TaskProducer(taskQueue));

      Task[] processors = new Task[4];
      for (int i = 1; i <= 4; i++)
      {
        string processorId = i.ToString();
        processors[i-1] = Task.Run(
          () => TaskProcessor(taskQueue, $"Processor {processorId}", cts.Token));
      }

      await taskSource;
      cts.CancelAfter(TimeSpan.FromSeconds(2));

      await Task.WhenAll(processors);
    }

    static async Task TaskProducer(ConcurrentQueue<CustomTask> queue)
    {
      for (int i = 1; i <= 20; i++)
      {
        await Task.Delay(50);
        var workItem = new CustomTask {Id = i};
        queue.Enqueue(workItem);
        WriteLine($"Task {workItem.Id} has been posted");
      }
    }

    static async Task TaskProcessor(
      ConcurrentQueue<CustomTask> queue, string name, CancellationToken token)
    {
      CustomTask workItem;
      bool dequeueSuccesful = false;

      await GetRandomDelay();
      do
      {
        dequeueSuccesful = queue.TryDequeue(out workItem);
        if (dequeueSuccesful)
        {
          WriteLine($"Task {workItem.Id} has been processed by {name}");
        }

        await GetRandomDelay();
      }
      while (!token.IsCancellationRequested);
    }

    static Task GetRandomDelay()
    {
      int delay = new Random(DateTime.Now.Millisecond).Next(1, 500);
      return Task.Delay(delay);
    }

    class CustomTask
    {
      public int Id { get; set; }
    }
    ```

1.  在`Main`方法内添加以下代码片段：

    ```cs
    Task t = RunProgram();
    t.Wait();
    ```

1.  运行程序。

## 它是如何工作的...

当程序运行时，我们使用`ConcurrentQueue`集合的实例创建一个任务队列。然后，我们创建一个取消令牌，在将任务发布到队列后，我们将使用它来停止工作。接下来，我们启动一个单独的工作线程，该线程将任务发布到任务队列。这一部分为我们的异步处理产生工作负载。

现在，让我们定义程序的任务消费部分。我们创建四个工作者，它们将等待随机的时间，从任务队列中获取任务，处理它，然后重复整个过程，直到我们发出取消令牌的信号。最后，我们启动任务生成线程，等待其完成，然后使用取消令牌通知消费者我们已经完成工作。最后一步将是等待所有消费者完成，以完成所有任务的处理。

我们看到任务从开始到结束都在被处理，但后来任务可能会在早期任务之前被处理，因为我们有四个独立运行的工作者，并且任务处理时间不是恒定的。我们看到对队列的访问是线程安全的；没有工作项被取两次。

# 使用 ConcurrentStack 改变异步处理顺序

这个配方是对上一个配方的一点点修改。我们再次创建一组要由多个工作者异步处理的任务，但这次我们使用 `ConcurrentStack` 来实现，并观察其中的差异。

## 准备工作

为了完成这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在 `BookSamples\Chapter6\Recipe3` 中找到。

## 如何操作...

要理解使用 `ConcurrentStack` 实现的一组任务的处理过程，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.Collections.Concurrent;
    using System.Threading;
    using System.Threading.Tasks;
    using static System.Console;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    static async Task RunProgram()
    {
      var taskStack = new ConcurrentStack<CustomTask>();
      var cts = new CancellationTokenSource();

      var taskSource = Task.Run(() => TaskProducer(taskStack));

      Task[] processors = new Task[4];
      for (int i = 1; i <= 4; i++)
      {
        string processorId = i.ToString();
        processors[i - 1] = Task.Run(
          () => TaskProcessor(taskStack, $"Processor {processorId}", cts.Token));
      }

      await taskSource;
      cts.CancelAfter(TimeSpan.FromSeconds(2));

      await Task.WhenAll(processors);
    }

    static async Task TaskProducer(ConcurrentStack<CustomTask> stack)
    {
      for (int i = 1; i <= 20; i++)
      {
        await Task.Delay(50);
        var workItem = new CustomTask { Id = i };
        stack.Push(workItem);
        WriteLine($"Task {workItem.Id} has been posted");
      }
    }

    static async Task TaskProcessor(
      ConcurrentStack<CustomTask> stack, string name, CancellationToken token)
    {
      await GetRandomDelay();
      do
      {
        CustomTask workItem;
        bool popSuccesful = stack.TryPop(out workItem);
        if (popSuccesful)
        {
          WriteLine($"Task {workItem.Id} has been processed by {name}");
        }

        await GetRandomDelay();
      }
      while (!token.IsCancellationRequested);
    }

    static Task GetRandomDelay()
    {
      int delay = new Random(DateTime.Now.Millisecond).Next(1, 500);
      return Task.Delay(delay);
    }

    class CustomTask
    {
      public int Id { get; set; }
    }
    ```

1.  在 `Main` 方法内部添加以下代码片段：

    ```cs
    Task t = RunProgram();
    t.Wait();
    ```

1.  运行程序。

## 工作原理...

当程序运行时，我们现在创建一个 `ConcurrentStack` 集合的实例。其余部分几乎与之前的配方相同，只是不再使用并发栈上的 `Push` 和 `TryPop` 方法，而是在并发队列上使用 `Enqueue` 和 `TryDequeue`。

我们现在可以看到任务处理顺序已经改变。栈是一个后进先出（LIFO）集合，工作者首先处理后面的任务。在并发队列的情况下，任务的处理顺序几乎与它们被添加的顺序相同。这意味着，根据工作者的数量，我们肯定会在给定的时间框架内处理最早创建的任务。在栈的情况下，较早创建的任务优先级较低，可能直到生产者停止向栈提供更多任务之前都不会被处理。这种行为非常特定，在这种情况下使用队列会更好。

# 使用 ConcurrentBag 创建可扩展的爬虫

这个配方展示了如何在不同数量的独立工作者之间扩展工作负载，这些工作者既生产工作又处理工作。

## 准备工作

为了完成这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在 `BookSamples\Chapter6\Recipe4` 中找到。

## 如何操作...

以下步骤演示了如何在多个既生产工作又处理工作的独立工作者之间扩展工作负载：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.Collections.Concurrent;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using static System.Console;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    static Dictionary<string, string[]> _contentEmulation = new Dictionary<string, string[]>();

    static async Task RunProgram()
    {
      var bag = new ConcurrentBag<CrawlingTask>();

      string[] urls = {"http://microsoft.com/", "http://google.com/", "http://facebook.com/", "http://twitter.com/"};

      var crawlers = new Task[4];
      for (int i = 1; i <= 4; i++)
      {
        string crawlerName = $"Crawler {i}";
        bag.Add(new CrawlingTask { UrlToCrawl = urls[i-1], ProducerName = "root"});
        crawlers[i - 1] = Task.Run(() => Crawl(bag, crawlerName));
      }

      await Task.WhenAll(crawlers);
    }

    static async Task Crawl(ConcurrentBag<CrawlingTask> bag, string crawlerName)
    {
      CrawlingTask task;
      while (bag.TryTake(out task))
      {
        IEnumerable<string> urls = await GetLinksFromContent(task);
        if (urls != null)
        {
          foreach (var url in urls)
          {
            var t = new CrawlingTask
            {
              UrlToCrawl = url,
              ProducerName = crawlerName
            };

            bag.Add(t);
          }
        }
        WriteLine($"Indexing url {task.UrlToCrawl} posted by " +
           $"{task.ProducerName} is completed by {crawlerName}!");
      }
    }

    static async Task<IEnumerable<string>> GetLinksFromContent(CrawlingTask task)
    {
      await GetRandomDelay();

      if (_contentEmulation.ContainsKey(task.UrlToCrawl)) return _contentEmulation[task.UrlToCrawl];

      return null;
    }

    static void CreateLinks()
    {
      _contentEmulation["http://microsoft.com/"] = new [] { "http://microsoft.com/a.html", "http://microsoft.com/b.html" };
      _contentEmulation["http://microsoft.com/a.html"] = new[] { "http://microsoft.com/c.html", "http://microsoft.com/d.html" };
      _contentEmulation["http://microsoft.com/b.html"] = new[] { "http://microsoft.com/e.html" };

      _contentEmulation["http://google.com/"] = new[] { "http://google.com/a.html", "http://google.com/b.html" };
      _contentEmulation["http://google.com/a.html"] = new[] { "http://google.com/c.html", "http://google.com/d.html" };
      _contentEmulation["http://google.com/b.html"] = new[] { "http://google.com/e.html", "http://google.com/f.html" };
      _contentEmulation["http://google.com/c.html"] = new[] { "http://google.com/h.html", "http://google.com/i.html" };

      _contentEmulation["http://facebook.com/"] = new [] { "http://facebook.com/a.html", "http://facebook.com/b.html" };
      _contentEmulation["http://facebook.com/a.html"] = new[] { "http://facebook.com/c.html", "http://facebook.com/d.html" };
      _contentEmulation["http://facebook.com/b.html"] = new[] { "http://facebook.com/e.html" };

      _contentEmulation["http://twitter.com/"] = new[] { "http://twitter.com/a.html", "http://twitter.com/b.html" };
      _contentEmulation["http://twitter.com/a.html"] = new[] { "http://twitter.com/c.html", "http://twitter.com/d.html" };
      _contentEmulation["http://twitter.com/b.html"] = new[] { "http://twitter.com/e.html" };
      _contentEmulation["http://twitter.com/c.html"] = new[] { "http://twitter.com/f.html", "http://twitter.com/g.html" };
      _contentEmulation["http://twitter.com/d.html"] = new[] { "http://twitter.com/h.html" };
      _contentEmulation["http://twitter.com/e.html"] = new[] { "http://twitter.com/i.html" };
    }

    static Task GetRandomDelay()
    {
      int delay = new Random(DateTime.Now.Millisecond).Next(150, 200);
      return Task.Delay(delay);
    }

    class CrawlingTask
    {
      public string UrlToCrawl { get; set; }

      public string ProducerName { get; set; }
    }
    ```

1.  在 `Main` 方法内部添加以下代码片段：

    ```cs
    CreateLinks();
    Task t = RunProgram();
    t.Wait();
    ```

1.  运行程序。

## 工作原理...

该程序通过多个网络爬虫模拟网页索引。网络爬虫是一个通过地址打开网页、索引内容、尝试访问该页面包含的所有链接，并将这些链接页面也进行索引的程序。一开始，我们定义一个包含不同网页 URL 的字典。这个字典模拟包含指向其他页面链接的网页。实现非常简单；它不考虑索引已访问的页面，但简单且使我们能够专注于并发工作负载。

然后，我们创建一个包含爬虫任务的并发袋。我们创建四个爬虫，并为每个爬虫提供一个不同的站点根 URL。然后，我们等待所有爬虫完成。现在，每个爬虫开始索引分配给它的站点 URL。我们通过等待一些随机的时间来模拟网络 I/O 过程；然后，如果页面包含更多 URL，爬虫会将更多爬虫任务发布到袋中。然后，它检查袋中是否还有待爬取的任务。如果没有，爬虫就完成了。

如果我们检查位于前四行（根 URL）下面的输出，通常我们会看到，由爬虫编号*N*发布的任务通常由同一个爬虫处理。然而，后面的行会有所不同。这是因为内部，`ConcurrentBag`针对的就是这种场景，即有多个线程既添加项目又移除项目。这是通过让每个线程使用自己的本地项目队列来实现的，因此，当这个队列被占用时，我们不需要任何锁。只有当我们本地队列中没有项目时，我们才会执行一些锁定操作，并尝试从另一个线程的本地队列中“窃取”工作。这种行为有助于在所有工作者之间分配工作，并避免锁定。

# 使用 BlockingCollection 泛化异步处理

本食谱将描述如何使用`BlockingCollection`简化工作负载异步处理的实现。

## 准备工作

要完成这个食谱，你需要 Visual Studio 2015。不需要其他先决条件。本食谱的源代码可以在`BookSamples\Chapter6\Recipe5`中找到。

## 如何实现...

要了解`BlockingCollection`如何简化工作负载异步处理的实现，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Collections.Concurrent;
    using System.Threading.Tasks;
    using static System.Console;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static async Task RunProgram(IProducerConsumerCollection<CustomTask> collection = null)
    {
      var taskCollection = new BlockingCollection<CustomTask>();
      if(collection != null)
        taskCollection= new BlockingCollection<CustomTask>(collection);

      var taskSource = Task.Run(() => TaskProducer(taskCollection));

      Task[] processors = new Task[4];
      for (int i = 1; i <= 4; i++)
      {
        string processorId = $"Processor {i}";
        processors[i - 1] = Task.Run(
          () => TaskProcessor(taskCollection, processorId));
      }

      await taskSource;

      await Task.WhenAll(processors);
    }

    static async Task TaskProducer(BlockingCollection<CustomTask> collection)
    {
      for (int i = 1; i <= 20; i++)
      {
        await Task.Delay(20);
        var workItem = new CustomTask { Id = i };
        collection.Add(workItem);
        WriteLine($"Task {workItem.Id} has been posted");
      }
      collection.CompleteAdding();
    }

    static async Task TaskProcessor(
      BlockingCollection<CustomTask> collection, string name)
    {
      await GetRandomDelay();
      foreach (CustomTask item in collection.GetConsumingEnumerable())
      {
        WriteLine($"Task {item.Id} has been processed by {name}");
        await GetRandomDelay();
      }
    }

    static Task GetRandomDelay()
    {
      int delay = new Random(DateTime.Now.Millisecond).Next(1, 500);
      return Task.Delay(delay);
    }

    class CustomTask
    {
      public int Id { get; set; }
    }
    ```

1.  在`Main`方法内部添加以下代码片段：

    ```cs
    WriteLine("Using a Queue inside of BlockingCollection");
    WriteLine();
    Task t = RunProgram();
    t.Wait();

    WriteLine();
    WriteLine("Using a Stack inside of BlockingCollection");
    WriteLine();
    t = RunProgram(new ConcurrentStack<CustomTask>());
    t.Wait();
    ```

1.  运行程序。

## 工作原理...

在这里，我们正好采用第一种场景，但现在，我们使用一个提供许多有用功能的`BlockingCollection`类。首先，我们能够改变任务在阻塞集合中存储的方式。默认情况下，它使用`ConcurrentQueue`容器，但我们能够使用任何实现了`IProducerConsumerCollection`泛型接口的集合。为了说明这一点，我们运行程序两次，第二次使用`ConcurrentStack`作为底层集合。

工作者通过迭代阻塞集合上`GetConsumingEnumerable`方法调用的结果来获取工作项。如果集合中没有项，迭代器将仅阻塞工作线程，直到有项被发布到集合中。当生产者调用集合上的`CompleteAdding`方法时，循环结束。这表示工作已完成。

### 注意

很容易犯错，直接迭代`BlockingCollection`，因为它本身实现了`IEnumerable`接口。不要忘记使用`GetConsumingEnumerable`，否则，你将仅迭代集合的“快照”，并得到完全意外的程序行为。

工作负载生产者将任务插入`BlockingCollection`，然后调用`CompleteAdding`方法，这将导致所有工作线程完成。现在，在程序输出中，我们看到两个结果序列，说明了并发队列和堆栈集合之间的差异。
