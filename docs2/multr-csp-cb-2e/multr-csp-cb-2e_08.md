# 第八章。Reactive Extensions

在本章中，我们将探讨另一个有趣的 .NET 库，它帮助我们创建异步程序，即 **Reactive Extensions**（**Rx**）。我们将涵盖以下食谱：

+   将集合转换为异步 `Observable`

+   编写自定义的 `Observable`

+   使用 `Subject` 类型

+   创建 `Observable` 对象

+   使用 LINQ 查询对 `Observable` 集合进行操作

+   使用 Rx 创建异步操作

# 简介

正如你已经学到的，在 .NET 和 C# 中创建异步程序有几种方法。其中之一是基于事件的异步模式，这在之前的章节中已经提到过。引入事件最初的目的是为了简化 `Observer` 设计模式的实现。这种模式在对象之间实现通知时很常见。

当我们讨论 Task Parallel Library 时，我们指出事件的主要缺点是它们无法有效地相互组合。另一个缺点是，基于事件的异步模式不应该用来处理通知的序列。想象一下，我们有一个 `IEnumerable<string>` 提供字符串值。然而，当我们迭代它时，我们不知道一次迭代将花费多少时间。它可能很慢，如果我们使用常规的 `foreach` 循环或其他同步迭代结构，我们将阻塞我们的线程，直到我们得到下一个值。这种情况被称为 **基于拉取** 的方法，当我们作为客户端从生产者拉取值时。

相反的方法是 **基于推送** 的方法，当生产者通知客户端有新值时。这允许将工作卸载到生产者，而客户端在等待下一个值时可以自由地做其他任何事情。因此，目标是得到类似异步版本的 `IEnumerable`，它产生一系列值，并在序列完成或抛出异常时通知消费者。

从 .NET Framework 4.0 版本开始，包含 `IObservable<out T>` 和 `IObserver<in T>` 接口的定义，它们一起代表异步推送式集合及其客户端。它们来自名为 Reactive Extensions（或简称 Rx）的库，该库是在微软内部创建的，以帮助我们有效地组合事件序列和所有其他类型的异步程序，使用可观察集合。这些接口已包含在 .NET Framework 中，但它们的实现和 Rx 库中的所有其他机制仍然分别分发。

### 注意

Rx 全局是一个跨平台库。它提供了 .NET 3.5、Silverlight 和 Windows Phone 的库。它还支持 JavaScript、Ruby 和 Python。它也是开源的；你可以在 CodePlex 网站上找到 .NET 的 Reactive Extensions 的源代码，以及其他实现可以在 GitHub 上找到。

最令人惊讶的是，可观察集合与 LINQ 兼容，因此我们可以使用声明性查询以异步方式转换和组合这些集合。这也使得我们可以使用扩展方法以与常规 LINQ 提供程序相同的方式向 Rx 程序添加功能。反应式扩展还支持从所有异步编程模式（包括异步编程模型、基于事件的异步模式和任务并行库）过渡到可观察集合，并支持其自己的异步操作运行方式，这与 TPL 仍然非常相似。

反应式扩展库是一个非常强大且复杂的工具，值得单独写一本书。在本章中，我想回顾最有用的场景，即如何有效地处理异步事件序列。我们将观察反应式扩展框架的关键类型，学习以不同的方式创建序列并操作它们，最后检查我们如何使用反应式扩展来运行异步操作并管理它们的选项。

# 将集合转换为异步可观察对象

本食谱将指导您如何从`Enumerable`类创建可观察集合以及如何异步处理它。

## 准备工作

为了完成这个食谱，您将需要 Visual Studio 2015。不需要其他先决条件。本食谱的源代码可以在`BookSamples\Chapter8\Recipe1`中找到。

## 如何做...

要了解如何从`Enumerable`类创建可观察集合并异步处理它，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C#控制台应用程序项目。

1.  通过以下步骤添加对**反应式扩展主库**NuGet 包的引用：

    1.  右键单击项目中的**引用**文件夹，并选择**管理 NuGet 包…**菜单选项。

    1.  现在，添加**反应式扩展 - 主库**NuGet 包。您可以在**管理 NuGet 包**对话框中搜索**rx-main**，如图所示：

    ![如何做...](img/B05292_08_01.jpg)

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Collections.Generic;
    using System.Reactive.Concurrency;
    using System.Reactive.Linq;
    using System.Threading;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static IEnumerable<int> EnumerableEventSequence()
    {
      for (int i = 0; i < 10; i++)
      {
        Sleep(TimeSpan.FromSeconds(0.5));
        yield return i;
      }
    }
    ```

1.  在`Main`方法内部添加以下代码片段：

    ```cs
    foreach (int i in EnumerableEventSequence())
    {
      Write(i);
    }

    WriteLine();
    WriteLine("IEnumerable");

    IObservable<int> o = EnumerableEventSequence().().ToObservable();
    using (IDisposable subscription = o.Subscribe(Write))
    {
      WriteLine();
      WriteLine("IObservable");
    }

    o = EnumerableEventSequence().ToObservable()
        .SubscribeOn(TaskPoolScheduler.Default);
    using (IDisposable subscription = o.Subscribe(Write))
    {
      WriteLine();
      WriteLine("IObservable async");
      ReadLine();
    }
    ```

1.  运行程序。

## 它是如何工作的...

在这里，我们使用`EnumerableEventSequence`方法模拟一个缓慢的可枚举集合。然后，我们使用常规的`foreach`循环迭代它，我们可以看到它实际上很慢；我们等待每个迭代完成。

我们随后使用 Reactive Extensions 库中的 `ToObservable` 扩展方法将这个可枚举集合转换为 Observable。接下来，我们订阅这个 Observable 集合的更新，提供 `Console.Write` 方法作为操作，该操作将在集合的每次更新时执行。因此，我们得到与之前完全相同的行为；我们等待每个迭代完成，因为我们使用主线程来订阅更新。

### 注意

我们将订阅对象包装在 using 语句中。尽管这不总是必要的，但销毁订阅是一个良好的实践，这有助于您避免与生命周期相关的错误。

要使程序异步，我们使用 `SubscribeOn` 方法，向其提供 TPL 任务池调度器。这个调度器会将订阅放置在 TPL 任务池中，从而将工作从主线程卸载。这允许我们保持 UI 响应，并在集合更新时做其他事情。要检查这种行为，您可以从代码中移除最后的 `Console.ReadLine` 调用。这样做时，我们将立即完成主线程，这迫使所有后台线程（包括 TPL 任务池工作线程）结束，并且我们将从异步集合中得不到任何输出。

如果我们使用 UI 框架，我们必须仅在 UI 线程内与 UI 控件交互。为了实现这一点，我们应该使用带有相应调度器的 `ObserveOn` 方法。对于 Windows Presentation Foundation，我们有 `DispatcherScheduler` 类和定义在名为 Rx-XAML 或 Reactive Extensions XAML 支持库的单独 NuGet 包中的 `ObserveOnDispatcher` 扩展方法。对于其他平台，也有相应的单独 NuGet 包。

# 编写自定义的 Observable

这个菜谱将描述如何实现 `IObservable<in T>` 和 `IObserver<out T>` 接口以获取自定义的 Observable 序列并正确消费它。

## 准备工作

要逐步实现这个菜谱，您需要 Visual Studio 2015。没有其他先决条件。这个菜谱的源代码可以在 `BookSamples\Chapter8\Recipe2` 中找到。

## 如何实现...

要了解如何实现 `IObservable<in T>` 和 `IObserver<out T>` 接口以获取自定义的 Observable 序列并消费它，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  添加对 **Reactive Extensions 主库** NuGet 包的引用。有关如何操作的更多详细信息，请参阅 *将集合转换为异步 Observable* 菜谱。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.Collections.Generic;
    using System.Reactive.Concurrency;
    using System.Reactive.Disposables;
    using System.Reactive.Linq;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    class CustomObserver : IObserver<int>
    {
      public void OnNext(int value)
      {
        WriteLine($"Next value: {value}; Thread Id: {CurrentThread.ManagedThreadId}");
      }

      public void OnError(Exception error)
      {
        WriteLine($"Error: {error.Message}");
      }

      public void OnCompleted()
      {
        WriteLine("Completed");
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

1.  在 `Main` 方法内添加以下代码片段：

    ```cs
    var observer = new CustomObserver();

    var goodObservable = new CustomSequence(new[] {1, 2, 3, 4, 5});
    var badObservable = new CustomSequence(null);

    using (IDisposable subscription = goodObservable.Subscribe(observer))
    {
    }

    using (IDisposable subscription = goodObservable
        .SubscribeOn(TaskPoolScheduler.Default).Subscribe(observer))
    {
      Sleep(TimeSpan.FromMilliseconds(100));
      WriteLine("Press ENTER to continue");
      ReadLine();
    }

    using (IDisposable subscription = badObservable
        .SubscribeOn(TaskPoolScheduler.Default).Subscribe(observer))
    {
      Sleep(TimeSpan.FromMilliseconds(100));
      WriteLine("Press ENTER to continue");
      ReadLine();
    }
    ```

1.  运行程序。

## 它是如何工作的...

在这里，我们首先通过简单地向控制台打印出可观察集合中下一个项目的信息、错误或序列完成信息来实现我们的观察者。这是一段非常简单的消费者代码，并没有什么特别之处。

有趣的部分是我们的可观察集合实现。我们故意在构造函数中接受一个数字枚举，并且不检查它是否为空。当我们有一个订阅观察者时，我们迭代这个集合，并通知观察者枚举中的每个项目。

然后，我们演示实际的订阅。正如我们所见，异步是通过调用 `SubscribeOn` 方法实现的，这是一个对 `IObservable` 的扩展方法，包含异步订阅逻辑。我们并不关心我们的可观察集合中的异步性；我们使用来自 Reactive Extensions 库的标准实现。

当我们订阅正常的可观察集合时，我们只是从中获取所有项目。现在它是异步的，因此我们需要等待一段时间，直到异步操作完成，然后打印消息并等待用户输入。

最后，我们尝试订阅下一个可观察集合，其中我们正在迭代一个空枚举，因此得到一个空引用异常。我们看到异常已经被正确处理，并且执行了 `OnError` 方法来打印错误详情。

# 使用 Subject 类型家族

这个菜谱展示了如何使用 Reactive Extensions 库中的 `Subject` 类型家族。

## 准备工作

为了完成这个菜谱，你需要 Visual Studio 2015。没有其他先决条件。这个菜谱的源代码可以在 `BookSamples\Chapter8\Recipe3` 中找到。

## 如何做到这一点...

要理解从 Reactive Extensions 库中 `Subject` 类型家族的使用，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  添加对 **Reactive Extensions 主库** NuGet 包的引用。有关如何操作的详细信息，请参阅 *将集合转换为异步可观察对象* 菜谱。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.Reactive.Subjects;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    static IDisposable OutputToConsole<T>(IObservable<T> sequence)
    {
      return sequence.Subscribe(
        obj => WriteLine($"{obj}")
        , ex => WriteLine($"Error: {ex.Message}")
        , () => WriteLine("Completed")
      );
    }
    ```

1.  在 `Main` 方法内部添加以下代码片段：

    ```cs
    WriteLine("Subject");
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

    WriteLine("ReplaySubject");
    var replaySubject = new ReplaySubject<string>();

    replaySubject.OnNext("A");
    using (var subscription = OutputToConsole(replaySubject))
    {
      replaySubject.OnNext("B");
      replaySubject.OnNext("C");
      replaySubject.OnNext("D");
      replaySubject.OnCompleted();
    }

    WriteLine("Buffered ReplaySubject");
    var bufferedSubject = new ReplaySubject<string>(2);

    bufferedSubject.OnNext("A");
    bufferedSubject.OnNext("B");
    bufferedSubject.OnNext("C");
    using (var subscription = OutputToConsole(bufferedSubject))
    {
      bufferedSubject.OnNext("D");
      bufferedSubject.OnCompleted();
    }

    WriteLine("Time window ReplaySubject");
    var timeSubject = new ReplaySubject<string>(TimeSpan.FromMilliseconds(200));

    timeSubject.OnNext("A");
    Sleep(TimeSpan.FromMilliseconds(100));
    timeSubject.OnNext("B");
    Sleep(TimeSpan.FromMilliseconds(100));
    timeSubject.OnNext("C");
    Sleep(TimeSpan.FromMilliseconds(100));
    using (var subscription = OutputToConsole(timeSubject))
    {
      Sleep(TimeSpan.FromMilliseconds(300));
      timeSubject.OnNext("D");
      timeSubject.OnCompleted();
    }

    WriteLine("AsyncSubject");
    var asyncSubject = new AsyncSubject<string>();

    asyncSubject.OnNext("A");
    using (var subscription = OutputToConsole(asyncSubject))
    {
      asyncSubject.OnNext("B");
      asyncSubject.OnNext("C");
      asyncSubject.OnNext("D");
      asyncSubject.OnCompleted();
    }

    WriteLine("BehaviorSubject");
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

## 它是如何工作的...

在这个程序中，我们查看 `Subject` 类型家族的不同变体。`Subject` 类型代表 `IObservable` 和 `IObserver` 的实现。这在不同的代理场景中非常有用，当我们想要将多个源的事件转换为单个流，或者相反，向多个订阅者广播事件序列时。主题对于在 Reactive Extensions 中进行实验也非常方便。

让我们从基本的 `Subject` 类型开始。它会在订阅者订阅后立即将事件序列重新翻译给订阅者。在我们的例子中，`A` 字符串将不会被打印出来，因为订阅发生在它被传输之后。除此之外，当我们对 `Observable` 调用 `OnCompleted` 或 `OnError` 方法时，它将停止进一步的事件序列翻译，所以最后一个字符串也不会被打印出来。

下一个类型，`ReplaySubject`，非常灵活，允许我们实现三个额外的场景。首先，它可以缓存从广播开始的所有事件，如果我们稍后订阅，我们将首先接收到所有先前的事件。这种行为在第二个例子中得到了说明。在这里，我们将在控制台上有所有四个字符串，因为第一个事件将被缓存并翻译给后来的订阅者。

然后，我们可以指定 `ReplaySubject` 的缓冲区大小和时间窗口大小。在下一个例子中，我们将主题设置为有两个事件缓冲。如果有更多事件被广播，只有最后两个将被重新翻译给订阅者。所以在这里，我们不会看到第一个字符串，因为我们订阅时主题缓冲区中有 `B` 和 `C`。情况与时间窗口相同。我们可以指定 `Subject` 类型只缓存发生时间在某个时间之前的所有事件，丢弃较旧的事件。因此，在第四个例子中，我们只会看到最后两个事件；较旧的事件不适合时间窗口。

`AsyncSubject` 类型类似于 TPL 中的 `Task` 类型。它代表一个单一的后台操作。如果有多个事件被发布，它将等待事件序列完成，并只将最后一个事件提供给订阅者。

`BehaviorSubject` 类型与 `ReplaySubject` 类型非常相似，但它只缓存一个值，并允许我们指定一个默认值，以防我们没有发送任何通知。在我们的最后一个例子中，我们会看到所有字符串都被打印出来，因为我们提供了一个默认值，并且所有其他事件都在订阅之后发生。如果我们把 `behaviorSubject.OnNext("B");` 这行代码向上移动到 `Default` 事件下面，它将替换输出中的默认值。

# 创建一个 `Observable` 对象

这个配方将描述创建 `Observable` 对象的不同方法。

## 准备工作

为了完成这个配方，你需要 Visual Studio 2015。不需要其他先决条件。这个配方的源代码可以在 `BookSamples\Chapter8\Recipe4` 中找到。

## 如何做到这一点...

要了解创建 `Observable` 对象的不同方法，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  添加对 **Reactive Extensions Main Library** NuGet 包的引用。有关如何操作的详细信息，请参阅 *将集合转换为异步 Observable* 配方。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.Reactive.Disposables;
    using System.Reactive.Linq;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    static IDisposable OutputToConsole<T>(IObservable<T> sequence)
    {
      return sequence.Subscribe(
        obj => WriteLine("{0}", obj)
        , ex => WriteLine("Error: {0}", ex.Message)
        , () => WriteLine("Completed")
      );
    }
    ```

1.  在 `Main` 方法内添加以下代码片段：

    ```cs
    IObservable<int> o = Observable.Return(0);
    using (var sub = OutputToConsole(o));
    WriteLine(" ---------------- ");

    o = Observable.Empty<int>();
    using (var sub = OutputToConsole(o));
    WriteLine(" ---------------- ");

    o = Observable.Throw<int>(new Exception());
    using (var sub = OutputToConsole(o));
    WriteLine(" ---------------- ");

    o = Observable.Repeat(42);
    using (var sub = OutputToConsole(o.Take(5)));
    WriteLine(" ---------------- ");

    o = Observable.Range(0, 10);
    using (var sub = OutputToConsole(o));
    WriteLine(" ---------------- ");

    o = Observable.Create<int>(ob => {
      for (int i = 0; i < 10; i++)
      {
        ob.OnNext(i);
      }
      return Disposable.Empty;
    });
    using (var sub = OutputToConsole(o)) ;
    WriteLine(" ---------------- ");

    o = Observable.Generate(
      0 // initial state
      , i => i < 5 // while this is true we continue the sequence
      , i => ++i // iteration
      , i => i*2 // selecting result
    );
    using (var sub = OutputToConsole(o));
    WriteLine(" ---------------- ");

    IObservable<long> ol = Observable.Interval(TimeSpan.FromSeconds(1));
    using (var sub = OutputToConsole(ol))
    {
      Sleep(TimeSpan.FromSeconds(3));
    };
    WriteLine(" ---------------- ");

    ol = Observable.Timer(DateTimeOffset.Now.AddSeconds(2));
    using (var sub = OutputToConsole(ol))
    {
      Sleep(TimeSpan.FromSeconds(3));
    };
    WriteLine(" ---------------- ");
    ```

1.  运行程序。

## 它是如何工作的...

在这里，我们探讨了创建 `可观察` 对象的不同场景。大部分这个功能都是作为 `Observable` 类型的静态工厂方法提供的。前两个示例展示了我们可以创建一个产生单个值的 `Observable` 方法和一个不产生任何值的 `Observable` 方法。在下一个示例中，我们使用 `Observable.Throw` 来构建一个触发其观察者 `OnError` 处理器的 `Observable` 类。

`Observable.Repeat` 方法表示一个无限序列。这个方法有不同的重载形式；在这里，我们通过重复 42 个值来构建一个无限序列。然后，我们使用 LINQ 的 `Take` 方法从这个序列中取出五个元素。`Observable.Range` 表示一系列值，与 `Enumerable.Range` 类似。

`Observable.Create` 方法支持更多自定义场景。有许多重载允许我们使用取消令牌和任务，但让我们看看最简单的一个。它接受一个函数，该函数接受观察者实例并返回一个表示订阅的 `IDisposable` 对象。如果我们有任何资源需要清理，我们就可以在这里提供清理逻辑，但因为我们实际上不需要它，所以我们只返回一个空的可丢弃对象。

`Observable.Generate` 方法是创建自定义序列的另一种方式。我们必须提供一个序列的初始值，然后提供一个确定是否生成更多项或完成序列的谓词。然后，我们提供一个迭代逻辑，在我们的例子中是增加计数器。最后一个参数是一个选择函数，它允许我们自定义结果。

最后两个方法处理计时器。`Observable.Interval` 以 `TimeSpan` 期间开始产生计时器滴答事件，而 `Observable.Timer` 指定启动时间。

# 使用 LINQ 查询对可观察集合进行查询

这个菜谱展示了如何使用 LINQ 查询异步事件序列。

## 准备工作

要完成这个菜谱，你需要 Visual Studio 2015。没有其他先决条件。这个菜谱的源代码可以在 `BookSamples\Chapter8\Recipe5` 找到。

## 如何操作...

要理解对可观察集合使用 LINQ 查询的使用，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  添加对 **Reactive Extensions Main Library** NuGet 包的引用。有关如何操作的详细信息，请参阅 *将集合转换为异步可观察序列* 菜谱。

1.  在 `Program.cs` 文件中，添加以下 `using` 指令：

    ```cs
    using System;
    using System.Reactive.Linq;
    using static System.Console;
    ```

1.  在 `Main` 方法下方添加以下代码片段：

    ```cs
    static IDisposable OutputToConsole<T>(IObservable<T> sequence, int innerLevel)
    {
      string delimiter = innerLevel == 0 
            ? string.Empty 
            : new string('-', innerLevel*3);

      return sequence.Subscribe(
        obj => WriteLine($"{delimiter}{obj}")
        , ex => WriteLine($"Error: {ex.Message}")
        , () => WriteLine($"{delimiter}Completed")
      );
    }
    ```

1.  在 `Main` 方法内添加以下代码片段：

    ```cs
    IObservable<long> sequence = Observable.Interval(
        TimeSpan.FromMilliseconds(50)).Take(21);

    var evenNumbers = from n in sequence
            where n % 2 == 0
            select n;

    var oddNumbers = from n in sequence
            where n % 2 != 0
            select n;

    var combine = from n in evenNumbers.Concat(oddNumbers)
          select n;

    var nums = (from n in combine
          where n % 5 == 0
          select n)
        .Do(n => WriteLine($"------Number {n} is processed in Do method"));

    using (var sub = OutputToConsole(sequence, 0))
    using (var sub2 = OutputToConsole(combine, 1))
    using (var sub3 = OutputToConsole(nums, 2))
    {
      WriteLine("Press enter to finish the demo");
      ReadLine();
    }
    ```

1.  运行程序。

## 它是如何工作的...

能够使用 LINQ 对 `Observable` 事件序列进行操作是 Reactive Extensions 框架的主要优势。还有很多不同的有用场景；不幸的是，在这里展示所有这些场景是不可能的。我试图提供一个简单但非常具有说明性的例子，它没有太多复杂细节，展示了 LINQ 查询在应用于异步可观察集合时的工作原理。

首先，我们创建一个 `Observable` 事件，它生成一个数字序列，每 50 毫秒一个数字，我们从初始值零开始，取 21 个这样的事件。然后，我们将 LINQ 查询组合到这个序列中。首先，我们只选择序列中的偶数，然后只选择奇数。然后，我们将这两个序列连接起来。

最后的查询展示了如何使用一个非常有用的方法 `Do`，它允许我们引入副作用，例如，记录结果序列中的每个值。要运行所有查询，我们创建嵌套的订阅，由于序列最初是异步的，我们必须非常小心订阅的生存期。外部作用域代表对计时器的订阅，内部订阅分别处理组合序列查询和副作用查询。如果我们太早按下 *Enter*，我们只是取消对计时器的订阅，从而停止演示。

当我们运行演示时，我们可以看到不同查询如何实时交互的实际过程。我们可以看到我们的查询是懒加载的，并且只有在订阅其结果时才开始运行。计时器事件的序列打印在第一列。当偶数查询得到一个偶数时，它也会打印出来，并使用 `---` 前缀来区分这个序列结果和第一个。最终的查询结果打印在右侧列。

当程序运行时，我们可以看到计时器序列、偶数序列和副作用序列是并行运行的。只有连接操作等待偶数序列完成。如果我们不连接这些序列，我们将有四个并行的事件序列以最有效的方式相互交互！这展示了 Reactive Extensions 的真正力量，并且可以作为一个深入学习这个库的好起点。

# 使用 Rx 创建异步操作

这个配方展示了如何从其他编程模式定义的异步操作中创建 `Observable`。

## 准备工作

为了完成这个配方，你需要 Visual Studio 2015。没有其他先决条件。这个配方的源代码可以在 `BookSamples\Chapter8\Recipe6` 找到。

## 如何操作...

要了解如何使用 Rx 创建异步操作，请执行以下步骤：

1.  启动 Visual Studio 2015。创建一个新的 C# 控制台应用程序项目。

1.  添加对**响应式扩展主库**NuGet 包的引用。有关如何操作的详细信息，请参阅*将集合转换为异步可观察对象*配方。

1.  在`Program.cs`文件中，添加以下`using`指令：

    ```cs
    using System;
    using System.Reactive;
    using System.Reactive.Linq;
    using System.Reactive.Threading.Tasks;
    using System.Threading.Tasks;
    using System.Timers;
    using static System.Console;
    using static System.Threading.Thread;
    ```

1.  在`Main`方法下方添加以下代码片段：

    ```cs
    static async Task<T> AwaitOnObservable<T>(IObservable<T> observable)
    {
      T obj = await observable;
      WriteLine($"{obj}" );
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
      Sleep(TimeSpan.FromSeconds(1));
      return $"Task {name} is completed. Thread Id {CurrentThread.ManagedThreadId}";
    }

    static IDisposable OutputToConsole(IObservable<EventPattern<ElapsedEventArgs>> sequence)
    {
      return sequence.Subscribe(
        obj => WriteLine($"{obj.EventArgs.SignalTime}")
        , ex => WriteLine($"Error: {ex.Message}")
        , () => WriteLine("Completed")
      );
    }

    static IDisposable OutputToConsole<T>(IObservable<T> sequence)
    {
      return sequence.Subscribe(
        obj => WriteLine("{0}", obj)
        , ex => WriteLine("Error: {0}", ex.Message)
        , () => WriteLine("Completed")
      );
    }
    ```

1.  将`Main`方法替换为以下代码片段：

    ```cs
    delegate string AsyncDelegate(string name);

    static void Main(string[] args)
    {
    IObservable<string> o = LongRunningOperationAsync("Task1");
    using (var sub = OutputToConsole(o))
    {
      Sleep(TimeSpan.FromSeconds(2));
    };
    WriteLine(" ---------------- ");

    Task<string> t = LongRunningOperationTaskAsync("Task2");
    using (var sub = OutputToConsole(t.ToObservable()))
    {
      Sleep(TimeSpan.FromSeconds(2));
    };
    WriteLine(" ---------------- ");

    AsyncDelegate asyncMethod = LongRunningOperation;

    // marked as obsolete, use tasks instead
    Func<string, IObservable<string>> observableFactory = 
      Observable.FromAsyncPattern<string, string>(
            asyncMethod.BeginInvoke, asyncMethod.EndInvoke);

    o = observableFactory("Task3");
    using (var sub = OutputToConsole(o))
    {
      Sleep(TimeSpan.FromSeconds(2));
    };
    WriteLine(" ---------------- ");

    o = observableFactory("Task4");
    AwaitOnObservable(o).Wait();
    WriteLine(" ---------------- ");

    using (var timer = new Timer(1000))
    {
      var ot = Observable.
                   FromEventPattern<ElapsedEventHandler, ElapsedEventArgs>(
        h => timer.Elapsed += h,
                h => timer.Elapsed -= h);
      timer.Start();

      using (var sub = OutputToConsole(ot))
      {
        Sleep(TimeSpan.FromSeconds(5));
      }
      WriteLine(" ---------------- ");
      timer.Stop();
    }
    ```

1.  运行程序。

## 它是如何工作的...

这个配方展示了如何将不同类型的异步操作转换为`Observable`类。第一个代码片段使用了`Observable.Start`方法，这与 TPL 中的`Task.Run`非常相似。它启动了一个异步操作，该操作输出一个字符串结果，然后完成。

### 注意

我强烈建议您使用任务并行库（Task Parallel Library，简称 TPL）进行异步操作。响应式扩展（Reactive Extensions）也支持这种场景，但为了避免歧义，在讨论单独的异步操作时坚持使用任务会更好，只有在我们需要处理事件序列时才使用 Rx。另一个建议是将所有类型的单独异步操作转换为任务，然后如果需要，再将任务转换为可观察类（`Observable` class）。

然后，我们用同样的方法处理任务，通过简单地调用`ToObservable`扩展方法将任务转换为`Observable`方法。下一个代码片段是关于将异步编程模型（Asynchronous Programming Model，简称 APM）转换为`Observable`。通常，你会将 APM 转换为任务，然后将任务转换为`Observable`。然而，存在一种直接转换，这个例子说明了如何运行异步委托并将其包装为`Observable`操作。

代码片段的下一部分展示了我们能够在`Observable`操作中使用`await`运算符。由于我们无法在如`Main`这样的入口方法上使用`async`修饰符，我们引入了一个单独的方法，该方法返回一个任务，并在`Main`方法内部等待这个结果任务的完成。

代码片段的最后部分与将 APM 模式转换为`Observable`的代码相同，但现在，我们将基于事件的异步模式（Event-based Asynchronous Pattern，简称 EAP）直接转换为`Observable`类。我们创建了一个计时器，并消耗其 5 秒的事件。然后，我们销毁计时器以清理资源。
