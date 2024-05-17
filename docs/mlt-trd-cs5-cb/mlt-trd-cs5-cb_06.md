# 第六章 使用并发集合

在本章中，我们将浏览包含在.NET Framework 基类库中的并发编程的不同数据结构。您将学习以下内容：

+   使用并发字典

+   使用并发队列实现异步处理

+   使用并发堆栈改变异步处理顺序

+   使用并发包创建可扩展的网络爬虫

+   使用阻塞集合泛化异步处理

# 介绍

编程需要理解和掌握基本的数据结构和算法。为了选择最适合并发情况的数据结构，程序员必须了解许多事情，比如算法时间、空间复杂度和大 O 符号。在不同的知名场景中，我们总是知道哪些数据结构更有效。

对于并发计算，我们需要适当的数据结构。这些数据结构必须是可扩展的，在可能的情况下避免锁，并且同时提供线程安全的访问。自.NET Framework 4 以来，具有几种数据结构的`System.Collections.Concurrent`命名空间。在本章中，我们将涵盖几种数据结构，并展示如何使用它们的非常简单的示例。

让我们从`ConcurrentQueue`开始。这个集合使用原子**比较和交换**（**CAS**）操作和`SpinWait`来确保线程安全。它实现了一个**先进先出**（**FIFO**）集合，这意味着项目以它们被添加到队列的顺序出队。要向队列添加项目，您调用`Enqueue`方法。`TryDequeue`方法尝试从队列中取出第一个项目，`TryPeek`方法尝试获取第一个项目而不从队列中移除它。

`ConcurrentStack`也是使用 CAS 操作而没有使用任何锁来实现的。它是一个**后进先出**（**LIFO**）集合，这意味着最近添加的项目将首先返回。要添加项目，您可以使用`Push`和`PushRange`方法，要检索，您可以使用`TryPop`和`TryPopRange`，要检查，您可以使用`TryPeek`方法。

`ConcurrentBag`是一个支持重复项目的无序集合。它针对多个线程以每个线程产生和消耗自己的任务的方式进行分区的场景进行了优化，很少处理其他线程的任务（在这种情况下，它使用锁）。您可以使用`Add`方法向包中添加项目，使用`TryPeek`进行检查，并使用`TryTake`方法进行获取。

### 注意

请避免在提到的集合上使用`Count`属性。它们使用链表实现，而`Count`是一个`O(N)`操作。如果您需要检查集合是否为空，请使用`IsEmpty`属性，这是一个`O(1)`操作。

`ConcurrentDictionary`是一个线程安全的字典集合实现。它对读操作是无锁的。但是，它对写操作需要锁定。并发字典使用多个锁，实现了对字典桶的细粒度锁定模型。锁的数量可以通过使用带有参数`concurrencyLevel`的构造函数来定义，这意味着估计数量的线程将同时更新字典。

### 注意

由于并发字典使用锁定，有许多操作需要在字典内部获取所有锁。请避免在不需要的情况下使用这些操作。它们是：`Count`，`IsEmpty`，`Keys`，`Values`，`CopyTo`和`ToArray`。

`BlockingCollection`是`IProducerConsumerCollection`泛型接口实现的高级包装器。它具有许多更先进的功能，并且在实现管道场景时非常有用，当您有一些步骤使用了处理前一步骤结果时。`BlockingCollection`类支持阻塞、限制内部集合容量、取消集合操作以及从多个阻塞集合中检索值等功能。

并发算法可能非常复杂，覆盖所有并发集合——无论是更先进还是更简单——都需要编写一本单独的书。在这里，我们只展示了使用并发集合的最简单的例子。

# 使用 ConcurrentDictionary

这个示例展示了一个非常简单的场景，在单线程环境中比较了普通字典集合与并发字典的性能。

## 准备工作

要按照这个示例，您需要 Visual Studio 2012。没有其他先决条件。这个示例的源代码可以在`BookSamples\Chapter6\Recipe1`中找到。

## 如何做...

为了理解普通字典集合与并发字典集合性能差异，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Diagnostics;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
const string Item = "Dictionary item";
public static string CurrentItem;
```

1.  在`Main`方法中添加以下代码片段：

```cs
var concurrentDictionary = new ConcurrentDictionary<int, string>();
var dictionary = new Dictionary<int, string>();

var sw = new Stopwatch();

sw.Start();
for (int i = 0; i < 1000000; i++)
{
  lock (dictionary)
  {
    dictionary[i] = Item;
  }
}
sw.Stop();
Console.WriteLine("Writing to dictionary with a lock: {0}", sw.Elapsed);

sw.Restart();
for (int i = 0; i < 1000000; i++)
{
  concurrentDictionary[i] = Item;
}
sw.Stop();
Console.WriteLine("Writing to a concurrent dictionary: {0}", sw.Elapsed);

sw.Restart();
for (int i = 0; i < 1000000; i++)
{
  lock (dictionary)
  {
    CurrentItem = dictionary[i];
  }
}
sw.Stop();
Console.WriteLine("Reading from dictionary with a lock: {0}", sw.Elapsed);

sw.Restart();
for (int i = 0; i < 1000000; i++)
{
  CurrentItem = concurrentDictionary[i];
}
sw.Stop();
Console.WriteLine("Reading from a concurrent dictionary: {0}", sw.Elapsed);
```

1.  运行程序。

## 它是如何工作的...

当程序启动时，我们创建了两个集合。其中一个是标准字典集合，另一个是一个新的并发字典。然后我们开始添加到它，使用带锁的标准字典并测量一百万次迭代完成所需的时间。然后我们测量在相同情况下`ConcurrentDictionary`的性能，最后比较从两个集合中检索值的性能。

在这个非常简单的场景中，我们发现`ConcurrentDictionary`在写操作上比普通的带锁的字典慢得多，但在检索操作上更快。因此，如果我们需要从字典中进行许多线程安全的读取，`ConcurrendDictionary`集合是最佳选择。

### 注意

如果您只需要对字典进行只读、多线程访问，可能不需要执行线程安全读取。在这种情况下，最好只使用普通字典或`ReadOnlyDictionary`集合。

`ConcurrentDictionary`是使用**细粒度锁定**技术实现的，这使得它在多次写入时比使用带锁的常规字典更好地扩展（称为**粗粒度锁定**）。正如我们在这个例子中看到的，当我们只使用一个线程时，并发字典要慢得多，但当我们将其扩展到五六个线程时（如果我们有足够的 CPU 核心可以同时运行它们），并发字典实际上会表现得更好。

# 使用 ConcurrentQueue 实现异步处理

这个示例将展示一个创建一组任务，由多个工作线程异步处理的示例。

## 准备工作

要按照这个示例，您需要 Visual Studio 2012。没有其他先决条件。这个示例的源代码可以在`BookSamples\Chapter6\Recipe2`中找到。

## 如何做...

为了理解创建一组任务，由多个工作线程异步处理的工作原理，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Collections.Concurrent;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

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
    () => TaskProcessor(taskQueue, "Processor " + processorId, cts.Token));
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
    Console.WriteLine("Task {0} has been posted", workItem.Id);
  }
}

static async Task TaskProcessor(ConcurrentQueue<CustomTask> queue, string name, CancellationToken token){
  CustomTask workItem;
  bool dequeueSuccesful = false;

  await GetRandomDelay();
  do
  {
    dequeueSuccesful = queue.TryDequeue(out workItem);
    if (dequeueSuccesful)
    {
    Console.WriteLine("Task {0} has been processed by {1}", workItem.Id, name);
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

1.  在`Main`方法中添加以下代码片段：

```cs
Task t = RunProgram();
t.Wait();
```

1.  运行程序。

## 它是如何工作的...

程序运行时，我们使用`ConcurrentQueue`集合创建了一个任务队列。然后我们创建了一个取消标记，用于在我们将任务发布到队列后停止工作。接下来，我们启动一个单独的工作者线程，将任务发布到任务队列。这部分产生了我们异步处理的工作负载。

现在让我们定义程序的任务消耗部分。我们创建四个工作者，它们将等待一段随机时间，然后从任务队列获取一个任务，处理它，并重复整个过程，直到我们发出取消标记。最后，我们启动任务生成线程，等待其完成，然后使用取消标记向消费者发出我们完成工作的信号。最后一步是等待所有消费者完成。

我们看到我们有任务从头到尾处理，但可能会出现一个后续任务在较早的任务之前被处理，因为我们有四个独立运行的工作者，任务处理时间不是恒定的。我们看到队列的访问是线程安全的；没有工作项被重复获取。

# 更改异步处理顺序 ConcurrentStack

这个食谱是对上一个的轻微修改。我们将再次创建一组任务，由多个工作者异步处理，但这次我们使用`ConcurrentStack`来实现，并看到其中的区别。

## 准备工作

要按照这个食谱进行操作，你需要 Visual Studio 2012。没有其他先决条件。这个食谱的源代码可以在`BookSamples\Chapter6\Recipe3`中找到。

## 如何做...

为了理解使用`ConcurrentStack`实现的一组任务的处理，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Collections.Concurrent;
using System.Threading;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

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
    () => TaskProcessor(taskStack, "Processor " + processorId, cts.Token));
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
    Console.WriteLine("Task {0} has been posted", workItem.Id);
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
    Console.WriteLine("Task {0} has been processed by {1}", workItem.Id, name);
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

1.  在`Main`方法中添加以下代码片段：

```cs
Task t = RunProgram();
t.Wait();
```

1.  运行程序。

## 它是如何工作的...

当程序运行时，我们现在创建了`ConcurrentStack`集合的一个实例。其余部分几乎与上一个食谱相同，只是在并发栈上使用`Push`和`TryPop`方法的地方，我们在并发队列上使用`Enqueue`和`TryDequeue`。

现在我们看到任务处理顺序已经改变。栈是一个后进先出的集合，工作者首先处理后续任务。在并发队列的情况下，任务几乎按照它们被添加的顺序进行处理。这意味着根据工作者的数量，我们肯定会在给定的时间范围内处理首先创建的任务。在栈的情况下，较早创建的任务优先级较低，可能在生产者停止向栈中添加更多任务之前不会被处理。这种行为非常特殊，最好在这种情况下使用队列。

# 使用 ConcurrentBag 创建可扩展的爬虫

这个食谱展示了如何在多个独立的工作者之间分配工作负载，他们既生产工作，又处理工作。

## 准备工作

要按照这个食谱进行操作，你需要 Visual Studio 2012。没有其他先决条件。这个食谱的源代码可以在`BookSamples`的`\Chapter6\Recipe4`中找到。

## 如何做...

以下步骤演示了如何在多个独立的工作者之间分配工作负载，他们既生产工作，又处理工作：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static Dictionary<string, string[]> _contentEmulation = new Dictionary<string, string[]>();

static async Task RunProgram()
{
  var bag = new ConcurrentBag<CrawlingTask>();

  string[] urls = new[] {"http://microsoft.com/", "http://google.com/", "http://facebook.com/", "http://twitter.com/"};

  var crawlers = new Task[4];
  for (int i = 1; i <= 4; i++)
  {
    string crawlerName = "Crawler " + i.ToString();
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
  Console.WriteLine("Indexing url {0} posted by {1} is completed by {2}!",
      task.UrlToCrawl, task.ProducerName, crawlerName);
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

1.  在`Main`方法中添加以下代码片段：

```cs
CreateLinks();
Task t = RunProgram();
t.Wait();
```

1.  运行程序。

## 它是如何工作的...

该程序模拟了多个网络爬虫进行网页索引。网络爬虫是一个打开网页并索引内容的程序，并尝试访问该页面包含的所有链接，并索引这些链接页面。一开始，我们定义了一个包含不同网页 URL 的字典。这个字典模拟了包含指向其他页面链接的网页。实现非常天真；它不关心已经访问过的页面，但它很简单，可以让我们专注于并发工作负载。

然后我们创建一个包含爬行任务的并发包。我们创建四个爬虫，并为每个爬虫提供不同的站点根 URL。然后，我们等待所有爬虫竞争。现在，每个爬虫开始索引它所给定的站点 URL。我们通过等待一些随机时间来模拟网络 I/O 过程；然后，如果页面包含更多的 URL，爬虫会将更多的爬行任务发布到包中。然后，它检查包中是否还有任何任务需要爬行。如果没有，爬虫就完成了。

如果我们检查前四行下面的输出，这些行是根 URL，我们会发现通常由爬虫编号*N*发布的任务会被同一个爬虫处理。然而，后面的行会有所不同。这是因为内部`ConcurrentBag`针对有多个线程同时添加和删除项目的情况进行了优化。这是通过让每个线程使用自己的本地项目队列来实现的，因此，在这个队列被占用时，我们不需要任何锁。只有当本地队列中没有项目时，我们才会执行一些锁定，并尝试从另一个线程的本地队列中“窃取”工作。这种行为有助于在所有工作者之间分配工作并避免锁定。

# 使用 BlockingCollection 泛化异步处理

本示例将描述如何使用`BlockingCollection`来简化工作负载异步处理的实现。

## 准备工作

要执行此示例，您需要 Visual Studio 2012。不需要其他先决条件。此示例的源代码可以在`BookSamples\Chapter6\Recipe5`中找到。

## 如何做...

要理解`BlockingCollection`如何简化工作负载异步处理的实现，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Collections.Concurrent;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static async Task RunProgram(IProducerConsumerCollection<CustomTask> collection = null)
{
  var taskCollection = new BlockingCollection<CustomTask>();
  if (null != collection)
  taskCollection= new BlockingCollection<CustomTask>(collection);

  var taskSource = Task.Run(() => TaskProducer(taskCollection));

  Task[] processors = new Task[4];
  for (int i = 1; i <= 4; i++)
  {
    string processorId = "Processor " + i;
    processors[i - 1] = Task.Run(() => TaskProcessor(taskCollection, processorId));
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
    Console.WriteLine("Task {0} have been posted", workItem.Id);
  }
  collection.CompleteAdding();
}

static async Task TaskProcessor(BlockingCollection<CustomTask> collection, string name)
{
  await GetRandomDelay();
  foreach (CustomTask item in collection.GetConsumingEnumerable())
  {
    Console.WriteLine("Task {0} have been processed by {1}", item.Id, name);
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

1.  在`Main`方法中添加以下代码片段：

```cs
Console.WriteLine("Using a Queue inside of BlockingCollection");
Console.WriteLine();
Task t = RunProgram();
t.Wait();

Console.WriteLine();
Console.WriteLine("Using a Stack inside of BlockingCollection");
Console.WriteLine();
t = RunProgram(new ConcurrentStack<CustomTask>());
t.Wait();
```

1.  运行程序。

## 它是如何工作的...

这里我们正好采用了第一种情况，但现在我们使用了一个提供许多有用好处的`BlockingCollection`类。首先，我们能够改变任务在阻塞集合中存储的方式。默认情况下，它使用`ConcurrentQueue`容器，但我们可以使用任何实现`IProducerConsumerCollection`泛型接口的集合。为了说明这一点，我们运行程序两次，第二次使用`ConcurrentStack`作为底层集合。

工作者通过迭代阻塞集合上的`GetConsumingEnumerable`方法调用结果来获取工作项。如果集合中没有项目，迭代器将阻塞工作者线程，直到有项目发布到集合中。当生产者在集合上调用`CompleteAdding`方法时，循环结束。这表示工作已完成。

### 注意

很容易犯一个错误，只是迭代`BlockingCollection`，因为它本身实现了`IEnumerable`。不要忘记使用`GetConsumingEnumerable`，否则你将只是迭代集合的“快照”，并得到完全意想不到的程序行为。

工作负载生产者将任务插入`BlockingCollection`，然后调用`CompleteAdding`方法，导致所有工作人员完成。现在在程序输出中，我们看到两个结果序列，说明并发队列和堆栈集合之间的区别。
