# 第八章. 反应扩展

在本章中，我们将看看另一个有趣的.NET 库，它可以帮助我们创建异步程序，即反应扩展（或 Rx）。您将学习以下配方：

+   将集合转换为异步`Observable`

+   编写自定义`Observable`

+   使用`Subjects`

+   创建一个`Observables`对象

+   使用 LINQ 查询对`Observable`集合

+   使用 Rx 创建异步操作

# 介绍

正如我们已经了解的，有几种方法可以在.NET 和 C#中创建异步程序。其中之一是基于事件的异步模式，这在前几章中已经提到过。引入事件的最初目标是简化实现`Observer`设计模式。这种模式通常用于在对象之间实现通知。

当我们讨论任务并行库时，我们注意到事件的主要缺点是它们无法有效地相互组合。另一个缺点是基于事件的异步模式不应该用来处理通知的顺序。想象一下，我们有一个`IEnumerable<string>`给我们字符串值。然而，当我们遍历它时，我们不知道一个迭代会花费多少时间。它可能很慢，如果我们使用常规的`foreach`或其他同步迭代结构，我们将阻塞我们的线程，直到我们有下一个值。这种情况被称为**拉取式**方法，当我们作为客户端从生产者那里拉取值时。

另一种方法是**推送式**方法，当生产者通知客户端有新值时。这允许将工作卸载给生产者，而客户端在等待另一个值时可以自由做任何其他事情。因此，目标是获得类似于异步版本的`IEnumerable`，它产生一系列值，并在序列中的每个项目完成时通知消费者，或者在抛出异常时通知消费者。

.NET Framework 从 4.0 版本开始包含了接口`IObservable<out T>`和`IObserver<in T>`的定义，它们一起代表了异步推送式集合及其客户端。它们来自一个名为 Reactive Extensions（简称 Rx）的库，该库是在微软内部创建的，旨在帮助有效地组合事件序列以及实际上所有其他类型的使用可观察集合的异步程序。这些接口被包含在.NET Framework 中，但它们的实现和所有其他机制仍然分别在 Rx 库中分发。

### 注意

反应扩展首先是一个跨平台库。有.NET 3.5、Silverlight 和 Windows Phone 的库。它也可用于 JavaScript、Ruby 和 Python。它也是开源的；你可以在 CodePlex 网站上找到.NET 的反应扩展源代码，也可以在 GitHub 上找到其他实现。

最令人惊讶的是，可观察集合与 LINQ 兼容，因此我们能够使用声明性查询以异步方式转换和组合这些集合。这也使得可以使用扩展方法来为 Rx 程序添加功能，就像在通常的 LINQ 提供程序中使用的方式一样。反应扩展还支持从所有异步编程模式（包括异步编程模型、基于事件的异步模式和任务并行库）过渡到可观察集合，并支持其自己的运行异步操作的方式，这仍然与 TPL 非常相似。

Reactive Extensions 库是一个非常强大和复杂的工具，值得写一本单独的书。在本章中，我想回顾最有用的场景，即如何有效地处理异步事件序列。我们将观察 Reactive Extensions 框架的关键类型，学习如何创建序列并以不同的方式操纵它们，最后，检查我们如何使用 Reactive Extensions 来运行异步操作并管理它们的选项。

# 将集合转换为异步 Observable

本篇将介绍如何从`Enumerable`类创建一个可观察集合，并异步处理它。

## 准备工作

要完成本篇，您需要 Visual Studio 2012。不需要其他先决条件。本篇的源代码可以在`BookSamples\Chapter8\Recipe1`中找到。

## 如何做...

要理解如何从`Enumerable`类创建一个可观察集合并异步处理它，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  将对**Reactive Extensions Main Library** NuGet 包添加引用。

1.  在项目中右键单击**引用**文件夹，然后选择**管理 NuGet 包...**菜单选项。

1.  现在添加您首选的**Reactive Extensions - Main Library** NuGet 包引用。您可以在**管理 NuGet 包**对话框中使用搜索，如下面的屏幕截图所示：

![如何做...](img/7644OT_08_01.jpg)

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Collections.Generic;
using System.Reactive.Concurrency;
using System.Reactive.Linq;
using System.Threading;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static IEnumerable<int> EnumerableEventSequence()
{
  for (int i = 0; i < 10; i++)
  {
    Thread.Sleep(TimeSpan.FromSeconds(0.5));
    yield return i;
  }
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
foreach (int i in EnumerableEventSequence())
{
  Console.Write(i);
}
Console.WriteLine();
Console.WriteLine("IEnumerable");

IObservable<int> o = EnumerableEventSequence().ToObservable();
using (IDisposable subscription = o.Subscribe(Console.Write))
{
  Console.WriteLine();
  Console.WriteLine("IObservable");
}

o = EnumerableEventSequence().ToObservable().SubscribeOn(TaskPoolScheduler.Default);
using (IDisposable subscription = o.Subscribe(Console.Write))
{
  Console.WriteLine();
  Console.WriteLine("IObservable async");
  Console.ReadLine();
}
```

1.  运行程序。

## 它是如何工作的...

我们使用`EnumerableEventSequence`方法模拟一个慢的可枚举集合。然后我们在通常的`foreach`循环中对其进行迭代，我们可以看到它实际上是慢的；我们等待每次迭代完成。

然后，我们使用 Reactive Extensions 库中的`ToObservable`扩展方法将这个可枚举集合转换为`Observable`。接下来，我们订阅这个可观察集合的更新，提供`Console.Write`方法作为操作，这将在每次集合更新时执行。结果我们得到了与之前完全相同的行为；我们等待每次迭代完成，因为我们使用主线程订阅更新。

### 注意

我们将订阅对象包装到使用语句中。虽然这并不总是必要的，但处理订阅是一个良好的实践，可以避免生命周期相关的错误。

为了使程序异步，我们使用`SubscribeOn`方法，并提供 TPL 任务池调度程序。这个调度程序将订阅到 TPL 任务池，从主线程卸载工作。这使我们能够保持 UI 的响应性，并在集合更新时做其他事情。要检查这种行为，您可以从代码中删除最后一个`Console.ReadLine`调用。这样做会立即结束我们的主线程，这将迫使所有后台线程（包括 TPL 任务池工作线程）也结束，并且我们将得不到异步集合的输出。

如果我们使用任何 UI 框架，我们必须只在 UI 线程内与 UI 控件交互。为了实现这一点，我们应该使用相应的调度程序的`ObserveOn`方法。对于 Windows Presentation Foundation，我们有`DispatcherScheduler`类和在名为 Rx-XAML 的单独 NuGet 包中定义的`ObserveOnDispatcher`扩展方法，或者 Reactive Extensions XAML 支持库。对于其他平台，也有相应的单独 NuGet 包。

# 编写自定义 Observable

本篇将描述如何实现`IObservable<in T>`和`IObserver<out T>`接口以获取自定义的 Observable 序列并正确消耗它。

## 准备工作

要执行此配方，您需要 Visual Studio 2012。不需要其他先决条件。此配方的源代码可以在`BookSamples\Chapter8\Recipe2`中找到。

## 操作步骤...

要理解如何实现`IObservable<in T>`和`IObserver<out T>`接口以获取自定义的 Observable 序列并消费它，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  添加对**Reactive Extensions Main Library** NuGet 包的引用。有关如何执行此操作的详细信息，请参阅*将集合转换为异步可观察对象*配方。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Collections.Generic;
using System.Reactive.Concurrency;
using System.Reactive.Disposables;
using System.Reactive.Linq;
using System.Threading;
```

1.  在`Main`方法下方添加以下代码片段：

```cs
class CustomObserver : IObserver<int>
{
  public void OnNext(int value)
  {
    Console.WriteLine("Next value: {0}; Thread Id: {1}", value, Thread.CurrentThread.ManagedThreadId);
  }

  public void OnError(Exception error)
  {
    Console.WriteLine("Error: {0}", error.Message);
  }

  public void OnCompleted()
  {
    Console.WriteLine("Completed");
  }
}

class CustomSequence : IObservable<int>
{
  private readonly IEnumerable<int> _numbers;

  public CustomSequence(IEnumerable<int> numbers)
  {
    _numbers = numbers;
  }
  public IDisposable Subscribe(IObserver<int> observer)
  {
    foreach (var number in _numbers)
    {
      observer.OnNext(number);
    }
    observer.OnCompleted();
    return Disposable.Empty;
  }
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
var observer = new CustomObserver();

var goodObservable = new CustomSequence(new[] {1, 2, 3, 4, 5});
var badObservable = new CustomSequence(null);

using (IDisposable subscription = goodObservable.Subscribe(observer))
{
}

using (IDisposable subscription = goodObservable.SubscribeOn(TaskPoolScheduler.Default).Subscribe(observer))
{
  Thread.Sleep(100);
}

using (IDisposable subscription = badObservable.SubscribeOn(TaskPoolScheduler.Default).Subscribe(observer))
{
  Console.ReadLine();
}
```

1.  运行程序。

## 工作原理...

在这里，我们首先实现了我们的观察者，简单地将来自可观察集合的下一个项目的信息打印到控制台上，错误，或者序列完成。这是一个非常简单的消费者代码，没有什么特别之处。

有趣的部分是我们的可观察集合实现。我们在构造函数中接受一个数字的枚举，并且故意不检查它是否为空。当我们有一个订阅的观察者时，我们遍历这个集合，并通知观察者枚举中的每个项目。

然后我们演示了实际的订阅。正如我们所看到的，通过调用`SubscribeOn`方法实现了异步，这是一个扩展方法，包含了异步订阅逻辑。我们不关心可观察集合中的异步性；我们使用了 Reactive Extensions 库中的标准实现。

当我们订阅普通的可观察集合时，我们只会得到其中的所有项目。现在它是异步的，所以我们需要等待一段时间，等待异步操作完成，然后才打印消息并等待用户输入。

最后，我们尝试订阅下一个可观察集合，我们正在遍历一个空枚举，因此会得到一个空引用异常。我们看到异常已经被正确处理，并且执行了`OnError`方法来打印出错误详情。

# 使用 Subjects

这个配方展示了如何使用 Reactive Extensions 库中的 Subject 类型家族。

## 准备工作

要执行此配方，您需要 Visual Studio 2012。不需要其他先决条件。此配方的源代码可以在`BookSamples\Chapter8\Recipe3`中找到。

## 操作步骤...

要理解如何使用 Reactive Extensions 库中的 Subject 类型家族，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  添加对**Reactive Extensions Main Library** NuGet 包的引用。有关如何执行此操作的详细信息，请参阅*将集合转换为异步可观察对象*配方。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Reactive.Subjects;
using System.Threading;
```

1.  在`Main`方法下方添加以下代码片段：

```cs
static IDisposable OutputToConsole<T>(IObservable<T> sequence)
{
  return sequence.Subscribe(obj => Console.WriteLine("{0}", obj), ex => Console.WriteLine("Error: {0}", ex.Message), () => Console.WriteLine("Completed"));
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
Console.WriteLine("Subject");
var subject = new Subject<string>();

subject.OnNext("A");
using (var subscription = OutputToConsole(subject))
{
  subject.OnNext("B");
  subject.OnNext("C");
  subject.OnNext("D");
  subject.OnCompleted();
  subject.OnNext("Will not be printed out");
}

Console.WriteLine("ReplaySubject");
var replaySubject = new ReplaySubject<string>();

replaySubject.OnNext("A");
using (var subscription = OutputToConsole(replaySubject))
{
  replaySubject.OnNext("B");
  replaySubject.OnNext("C");
  replaySubject.OnNext("D");
  replaySubject.OnCompleted();
}

Console.WriteLine("Buffered ReplaySubject");
var bufferedSubject = new ReplaySubject<string>(2);

bufferedSubject.OnNext("A");
bufferedSubject.OnNext("B");
bufferedSubject.OnNext("C");
using (var subscription = OutputToConsole(bufferedSubject))
{
  bufferedSubject.OnNext("D");
  bufferedSubject.OnCompleted();
}

Console.WriteLine("Time window ReplaySubject");
var timeSubject = new ReplaySubject<string>(TimeSpan.FromMilliseconds(200));

timeSubject.OnNext("A");
Thread.Sleep(TimeSpan.FromMilliseconds(100));
timeSubject.OnNext("B");
Thread.Sleep(TimeSpan.FromMilliseconds(100));
timeSubject.OnNext("C");
Thread.Sleep(TimeSpan.FromMilliseconds(100));
using (var subscription = OutputToConsole(timeSubject))
{
  Thread.Sleep(TimeSpan.FromMilliseconds(300));
  timeSubject.OnNext("D");
  timeSubject.OnCompleted();
}

Console.WriteLine("AsyncSubject");
var asyncSubject = new AsyncSubject<string>();

asyncSubject.OnNext("A");
using (var subscription = OutputToConsole(asyncSubject))
{
  asyncSubject.OnNext("B");
  asyncSubject.OnNext("C");
  asyncSubject.OnNext("D");
  asyncSubject.OnCompleted();
}

Console.WriteLine("BehaviorSubject");
var behaviorSubject = new BehaviorSubject<string>("Default");
using (var subscription = OutputToConsole(behaviorSubject))
{
  behaviorSubject.OnNext("B");
  behaviorSubject.OnNext("C");
  behaviorSubject.OnNext("D");
  behaviorSubject.OnCompleted();
}
```

1.  运行程序。

## 工作原理...

在这个程序中，我们查看了 Subject 类型家族的不同变体。Subject 代表了`IObservable`和`IObserver`的实现。在不同的代理场景中，当我们想要将来自多个来源的事件转换为一个流，或者反之亦然，将事件序列广播给多个订阅者时，这是非常有用的。Subject 也非常方便用于对反应扩展进行实验。

让我们从基本的 Subject 类型开始。它会在订阅者订阅后立即将事件序列重新传递给订阅者。在我们的情况下，`A`字符串不会被打印出来，因为订阅发生在它被传输之后。此外，当我们在`Observable`上调用`OnCompleted`或`OnError`方法时，它会停止进一步传递事件序列，因此最后一个字符串也不会被打印出来。

下一个类型`ReplaySubject`非常灵活，允许我们实现三种额外的场景。首先，它可以缓存从它们广播开始的所有事件，如果我们稍后订阅，我们将首先得到所有先前的事件。这种行为在第二个例子中有所体现。在这里，我们将在控制台上看到所有四个字符串，因为第一个事件将被缓存并转换给后来的订阅者。

然后我们可以为`ReplaySubject`指定缓冲区大小和时间窗口大小。在下一个例子中，我们将主题设置为具有两个事件的缓冲区。如果广播了更多的事件，只有最后两个事件将被重新传递给订阅者。因此，在这里我们将看不到第一个字符串，因为当订阅它时，我们的主题缓冲区中有`B`和`C`。时间窗口也是一样的。我们可以指定主题只缓存在某个时间之前发生的事件，丢弃较旧的事件。因此，在第四个例子中，我们将只看到最后两个事件；较旧的事件不符合时间窗口限制。

`AsyncSubject`类似于任务并行库中的`Task`类型。它表示单个异步操作。如果有多个事件被发布，它会等待事件序列完成，并且只向订阅者提供最后一个事件。

`BehaviorSubject`与`ReplaySubject`类型非常相似，但它只缓存一个值，并允许在我们尚未发送任何通知的情况下指定默认值。在我们的最后一个例子中，我们将看到所有的字符串被打印出来，因为我们提供了一个默认值，并且所有其他事件都发生在订阅之后。如果我们将`behaviorSubject.OnNext("B");`行向上移动到`Default`事件下面，它将替换输出中的默认值。

# 创建一个 Observable 对象

这个步骤将描述创建`Observable`对象的不同方法。

## 准备工作

要按照这个步骤，你需要一个运行中的 Visual Studio 2012。不需要其他先决条件。这个步骤的源代码可以在`BookSamples\Chapter8\Recipe4`中找到。

## 如何做到...

要了解创建`Observable`对象的不同方法，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# **控制台应用程序**项目。

1.  添加对**Reactive Extensions Main Library** NuGet 包的引用。有关如何执行此操作的详细信息，请参考*将集合转换为异步 Observable*步骤。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Reactive.Disposables;
using System.Reactive.Linq;
using System.Threading;
```

1.  在`Main`方法下方添加以下代码片段：

```cs
static IDisposable OutputToConsole<T>(IObservable<T> sequence)
{
  return sequence.Subscribe(obj => Console.WriteLine("{0}", obj), ex => Console.WriteLine("Error: {0}", ex.Message), () => Console.WriteLine("Completed"));
}
```

1.  在`Main`方法内添加以下代码片段：

```cs
IObservable<int> o = Observable.Return(0);
using (var sub = OutputToConsole(o));
Console.WriteLine(" ---------------- ");

o = Observable.Empty<int>();
using (var sub = OutputToConsole(o));
Console.WriteLine(" ---------------- ");

o = Observable.Throw<int>(new Exception());
using (var sub = OutputToConsole(o));
Console.WriteLine(" ---------------- ");

o = Observable.Repeat(42);
using (var sub = OutputToConsole(o.Take(5)));
Console.WriteLine(" ---------------- ");

o = Observable.Range(0, 10);
using (var sub = OutputToConsole(o));
Console.WriteLine(" ---------------- ");

o = Observable.Create<int>(ob => {
  for (int i = 0; i < 10; i++)
  {
    ob.OnNext(i);
  }
  return Disposable.Empty;
});
using (var sub = OutputToConsole(o)) ;
Console.WriteLine(" ---------------- ");

o = Observable.Generate(0 // initial state, i => i < 5 // while this is true we continue the sequence, i => ++i // iteration, i => i*2 // selecting result);
using (var sub = OutputToConsole(o));
Console.WriteLine(" ---------------- ");

IObservable<long> ol = Observable.Interval(TimeSpan.FromSeconds(1));
using (var sub = OutputToConsole(ol))
{
  Thread.Sleep(TimeSpan.FromSeconds(3));
};
Console.WriteLine(" ---------------- ");

ol = Observable.Timer(DateTimeOffset.Now.AddSeconds(2));
using (var sub = OutputToConsole(ol))
{
  Thread.Sleep(TimeSpan.FromSeconds(3));
};
Console.WriteLine(" ---------------- ");
```

1.  运行程序。

## 工作原理...

在这里，我们将介绍创建`observables`的不同场景。大部分这些功能都是`Observable`类型的静态工厂方法提供的。前两个示例展示了如何创建一个产生单个值的`Observable`方法和一个不产生值的方法。在下一个示例中，我们使用`Observable.Throw`来构造一个触发其观察者的`OnError`处理程序的`Observable`类。

`Observable.Repeat`方法表示一个无限序列。这个方法有不同的重载；在这里，我们通过重复 42 个值来构造一个无限序列。然后我们使用 LINQ 的`Take`方法从这个序列中取出五个元素。`Observable.Range`表示一个值的范围，就像`Enumerable.Range`一样。

`Observable.Create`方法支持更多的自定义场景。有很多重载允许我们使用取消标记和任务，但让我们看看最简单的一个。它接受一个函数，该函数接受一个观察者实例，并返回一个表示订阅的`IDisposable`对象。如果我们有任何需要清理的资源，我们可以在这里提供清理逻辑，但我们只返回一个空的可处置对象，因为实际上我们并不需要它。

`Observable.Generate`是创建自定义序列的另一种方法。我们必须为序列提供一个初始值，然后提供一个确定是否应生成更多项或完成序列的谓词。然后我们提供一个迭代逻辑，在我们的情况下是递增计数器。最后一个参数是一个选择器函数，允许我们自定义结果。

最后两种方法处理定时器。`Observable.Interval`开始生成具有`TimeSpan`周期的定时器滴答事件，而`Observable.Timer`也指定了启动时间。

# 针对可观察集合使用 LINQ 查询

这个配方展示了如何使用 LINQ 来查询异步事件序列。

## 准备就绪

要按照这个示例，您需要 Visual Studio 2012。不需要其他先决条件。此示例的源代码可以在`BookSamples\Chapter8\Recipe5`中找到。

## 如何做...

要理解针对可观察集合使用 LINQ 查询，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#**控制台应用程序**项目。

1.  添加对**Reactive Extensions Main Library** NuGet 包的引用。有关如何执行此操作的详细信息，请参阅*将集合转换为异步可观察*配方。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Reactive.Linq;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static IDisposable OutputToConsole<T>(IObservable<T> sequence, int innerLevel)
{
  string delimiter = innerLevel == 0 ? string.Empty : new string('-', innerLevel*3);
  return sequence.Subscribe(obj => Console.WriteLine("{0}{1}", delimiter, obj), ex => Console.WriteLine("Error: {0}", ex.Message), () => Console.WriteLine("{0}Completed", delimiter));
}
```

1.  在`Main`方法内部添加以下代码片段：

```cs
IObservable<long> sequence = Observable.Interval(TimeSpan.FromMilliseconds(50)).Take(21);

var evenNumbers = from n in sequencewhere n % 2 == 0select n;

var oddNumbers = from n in sequencewhere n % 2 != 0select n;

var combine = from n in evenNumbers.Concat(oddNumbers)select n;

var nums = (from n in combinewhere n % 5 == 0select n).Do(n => Console.WriteLine("------Number {0} is processed in Do method", n));

using (var sub = OutputToConsole(sequence, 0))
using (var sub2 = OutputToConsole(combine, 1))
using (var sub3 = OutputToConsole(nums, 2))
{
  Console.WriteLine("Press enter to finish the demo");
  Console.ReadLine();
}
```

1.  运行程序。

## 它是如何工作的...

针对`Observable`事件序列使用 LINQ 的能力是 Reactive Extensions 框架的主要优势。有许多不同的有用场景；不幸的是，这里不可能展示所有这些场景。我尝试提供一个简单但非常有说明性的示例，它没有太多复杂的细节，展示了当应用于异步可观察集合时，LINQ 查询的工作原理的本质。

首先，我们创建一个`Observable`事件，每 50 毫秒生成一个数字序列，从零开始，取其中的 21 个事件。然后，我们对这个序列进行 LINQ 查询。首先，我们只选择序列中的偶数，然后只选择奇数，然后我们连接这两个序列。

最终的查询显示了如何使用非常有用的方法`Do`，它允许引入副作用，例如记录结果序列中的每个值。为了运行所有查询，我们创建了嵌套的订阅，因为序列最初是异步的，所以我们必须非常小心地处理订阅的生命周期。外部范围表示对定时器的订阅，内部订阅处理组合序列查询和副作用查询。如果我们过早按*Enter*键，我们只需取消订阅定时器，从而停止演示。

当我们运行演示时，我们可以看到不同查询如何实时交互的实际过程。我们可以看到我们的查询是惰性的，它们只有在我们订阅它们的结果时才开始运行。定时器事件序列打印在第一列中。当偶数查询得到偶数时，它也打印出来，使用`---`前缀来区分这个序列结果和第一个序列结果。最终的查询结果打印到右列中。

当程序运行时，我们可以看到定时器序列、偶数序列和副作用序列并行运行。只有连接等待偶数序列完成。如果我们不连接这些序列，我们将有四个并行事件序列相互交互的最有效方式！这显示了 Reactive Extensions 的真正力量，并且可能是深入学习这个库的良好起点。

# 创建具有 Rx 的异步操作

这个配方展示了如何从其他编程模式中定义的异步操作中创建`Observable`。

## 准备就绪

要按照此示例操作，您需要 Visual Studio 2012。不需要其他先决条件。此示例的源代码可以在`BookSamples\Chapter8\Recipe6`中找到。

## 如何操作...

要了解如何使用 Rx 创建异步操作，请执行以下步骤：

1.  开始 Visual Studio 2012。创建一个新的 C# **控制台应用程序**项目。

1.  将对**Reactive Extensions Main Library** NuGet 包添加引用。有关如何执行此操作的详细信息，请参阅*将集合转换为异步可观察对象*的示例。

1.  在`Program.cs`文件中，添加以下`using`指令：

```cs
using System;
using System.Reactive;
using System.Reactive.Linq;
using System.Reactive.Threading.Tasks;
using System.Threading;
using System.Threading.Tasks;
using System.Timers;
using Timer = System.Timers.Timer;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static async Task<T> AwaitOnObservable<T>(IObservable<T> observable)
{
  T obj = await observable;
  Console.WriteLine("{0}", obj );
  return obj;
}

static Task<string> LongRunningOperationTaskAsync(string name)
{
  return Task.Run(() => LongRunningOperation(name));
}

static IObservable<string> LongRunningOperationAsync(string name)
{
  return Observable.Start(() => LongRunningOperation(name));
}

static string LongRunningOperation(string name)
{
  Thread.Sleep(TimeSpan.FromSeconds(1));
  return string.Format("Task {0} is completed. Thread Id {1}", name, Thread.CurrentThread.ManagedThreadId);
}

static IDisposable OutputToConsole(IObservable<EventPattern<ElapsedEventArgs>> sequence)
{
  return sequence.Subscribe(obj => Console.WriteLine("{0}", obj.EventArgs.SignalTime), ex => Console.WriteLine("Error: {0}", ex.Message), () => Console.WriteLine("Completed"));
}

static IDisposable OutputToConsole<T>(IObservable<T> sequence)
{
  return sequence.Subscribe(
    obj => Console.WriteLine("{0}", obj), ex => Console.WriteLine("Error: {0}", ex.Message), () => Console.WriteLine("Completed"));
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
IObservable<string> o = LongRunningOperationAsync("Task1");
using (var sub = OutputToConsole(o))
{
  Thread.Sleep(TimeSpan.FromSeconds(2));
};
Console.WriteLine(" ---------------- ");

Task<string> t = LongRunningOperationTaskAsync("Task2");
using (var sub = OutputToConsole(t.ToObservable()))
{
  Thread.Sleep(TimeSpan.FromSeconds(2));
};
Console.WriteLine(" ---------------- ");

AsyncDelegate asyncMethod = LongRunningOperation;

// marked as obsolete, use tasks instead
Func<string, IObservable<string>> observableFactory = Observable.FromAsyncPattern<string, string>(asyncMethod.BeginInvoke, asyncMethod.EndInvoke);
o = observableFactory("Task3");
using (var sub = OutputToConsole(o))
{
  Thread.Sleep(TimeSpan.FromSeconds(2));
};
Console.WriteLine(" ---------------- ");

o = observableFactory("Task4");
AwaitOnObservable(o).Wait();
Console.WriteLine(" ---------------- ");

using (var timer = new Timer(1000))
{
  var ot = Observable.FromEventPattern<ElapsedEventHandler, ElapsedEventArgs>(h => timer.Elapsed += h,h => timer.Elapsed -= h);
  timer.Start();

  using (var sub = OutputToConsole(ot))
  {
    Thread.Sleep(TimeSpan.FromSeconds(5));
  }
  Console.WriteLine(" ---------------- ");
  timer.Stop();
}
```

1.  运行程序。

## 它是如何工作的...

此示例说明了如何将不同类型的异步操作转换为`Observable`类。步骤 5 中的第一个代码片段使用了`Observable.Start`方法，这与 TPL 中的`Task.Run`非常相似。它启动一个给出字符串结果然后完成的异步操作。

### 注意

我强烈建议使用任务并行库进行异步操作。Reactive Extensions 也支持这种情况，但为了避免歧义，最好在单独的异步操作时坚持使用任务，并且只有在需要处理事件序列时才使用 Rx。另一个建议是将每种类型的单独异步操作转换为任务，然后只有在需要时将任务转换为`observable`类。

然后，我们使用任务做同样的事情，并通过简单调用`ToObservable`扩展方法将任务转换为`Observable`方法。步骤 5 中显示的下一个代码片段是关于将异步编程模型模式转换为`Observable`。通常，您会将 APM 转换为任务，然后将任务转换为`Observable`。但是，这里有一个直接的转换，这个示例说明了如何运行一个异步委托并将其包装成`Observable`操作。

步骤 5 中代码片段的下一部分显示我们能够`await`一个`Observable`操作。由于我们无法在`Main`等入口方法上使用`async`修饰符，因此我们引入一个返回任务并等待结果任务完成的单独方法到`Main`方法中。

步骤 5 中代码片段的最后部分是相同的，但现在我们直接将基于事件的异步模式转换为`Observable`类。我们创建一个计时器，并在 5 秒内使用其事件。然后我们释放计时器以清理资源。
