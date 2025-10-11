# 5. 并发：多线程、并行和异步代码

概述

C# 和 .NET 提供了一种高效的方式来运行并发代码，使得执行复杂且通常耗时的操作变得容易。在本章中，你将探索可用的各种模式，从使用 `Task` 工厂方法创建任务到使用延续将任务链接起来，然后转向 `async`/`await` 关键字，这些关键字极大地简化了此类代码。在本章结束时，你将看到如何使用 C# 执行并发运行的代码，并且通常比单线程应用程序产生结果更快。

# 简介

并发是一个通用术语，描述了软件同时执行多项任务的能力。通过利用并发的力量，你可以通过将 CPU 密集型活动从主 UI 线程卸载来提供更响应的用户界面。在服务器端，通过利用多处理器和多核架构的现代处理能力，可以通过并行处理操作来实现可伸缩性。

多线程是一种并发形式，其中使用多个线程来执行操作。这通常是通过创建许多 `Thread` 实例并在它们之间协调操作来实现的。它被认为是一种遗留实现，已被并行和异步编程所取代；你可能会在旧项目中找到它的使用。

并行编程是一种多线程类别，其中相似的操作彼此独立运行。通常，相同的操作会通过多个循环重复执行，其中操作的参数或目标本身会随着迭代而变化。.NET 提供了库，可以保护开发者免受线程创建的低级复杂性的影响。短语**令人尴尬的并行**常用来描述一种只需要额外努力就能分解成可以并行运行的任务集的活动，通常在这些子任务之间几乎没有交互。并行编程的一个例子可能是计算文件夹中每个文本文件中找到的单词数量。打开文件和扫描单词的工作可以分解成并行任务。每个任务执行相同的代码行，但被分配处理不同的文本文件。

异步编程是一种更近期的并发形式，其中一旦启动操作，将在未来的某个时刻完成，并且调用代码能够继续执行其他操作。这种完成通常被称为 `Task<>` 等价物。在 C# 和 .NET 中，异步编程已成为实现并发操作的首选方法。

异步编程的一个常见应用场景是，在调用最终步骤之前，需要初始化和整理多个慢速运行或昂贵的依赖项。例如，一个移动徒步旅行应用程序可能需要在用户安全开始徒步之前等待可靠的 GPS 卫星信号、计划好的导航路线和心率监测服务准备就绪。每个这些独立的步骤都会使用一个专用的任务来初始化。

异步编程的另一个非常常见的用例出现在 UI 应用程序中，例如，将客户的订单保存到数据库可能需要 5-10 秒才能完成。这可能涉及验证订单、打开与远程服务器或数据库的连接、打包并发送订单以供通过网络传输的格式，然后最终等待确认客户的订单已成功存储在数据库中。在一个单线程应用程序中，这会花费更长的时间，这种延迟很快就会被用户注意到。应用程序会在操作完成之前无响应。在这种情况下，用户可能会错误地认为应用程序已崩溃，并尝试关闭它。这不是一个理想的用户体验。

通过使用执行任何慢速操作的专用任务的异步代码，可以减轻此类问题。这些任务可以选择在进度中提供反馈，UI 的主线程可以使用这些反馈来通知用户。总体而言，操作应该会更快完成，从而使用户能够继续与应用程序交互。在现代应用程序中，用户已经习惯了这种操作方式。事实上，许多 UI 指南建议，如果某个操作可能需要几秒钟才能完成，那么它应该使用异步代码来执行。

注意，当代码正在执行时，无论是同步代码还是异步代码，它都是在 `Thread` 实例的上下文中运行的。在异步代码的情况下，这个 `Thread` 实例是由 .NET 调度程序从可用的线程池中选择。

`Thread` 类具有多种属性，但其中最有用的一项是 `ManagedThreadId`，它将在本章中广泛使用。这个整数值用于在您的进程中唯一标识一个线程。通过检查 `Thread.ManagedThreadId`，您可以确定正在使用多个线程实例。这可以通过在代码中使用静态的 `Thread.CurrentThread` 方法来访问 `Thread` 实例来完成。

例如，如果您启动了五个长时间运行的任务，并检查每个任务的 `Thread.ManagedThreadId`，您会观察到五个唯一的 ID，可能编号为二、三、四、五和六。在大多数情况下，ID 为一的线程是进程的主线程，它在进程首次启动时创建。

跟踪线程 ID 可以非常有用，尤其是在你需要执行耗时操作时。正如你所见，使用并发编程，可以同时执行多个操作，而不是使用传统的单线程方法，在后续操作开始之前，必须完成一个操作。

在现实世界中，考虑在山脉中建造火车隧道的案例。如果两个团队从山脉的两侧开始，同时向对方挖掘隧道，那么这个过程可以大大加快。两个团队可以独立工作；一个团队在一边遇到的问题不应对另一边的团队产生不利影响。一旦两边都完成了隧道挖掘，就应有一个单一的隧道，然后可以继续进行下一个任务，例如铺设火车线路。

下一节将探讨使用 C# 的 `Task` 类，它允许你同时独立地执行代码块。再次考虑 UI 应用程序的例子，其中客户的订单需要保存到数据库。为此，你有两种选择：

第一种选择是创建一个 C# `Task`，依次执行每个步骤：

+   验证订单。

+   连接到服务器。

+   发送请求。

+   等待响应。

第二种选择是为每个步骤创建一个 C# `Task`，尽可能并行执行每个步骤。

这两种选择都达到了相同的结果，即释放 UI 的主线程以响应用户交互。第一种选择可能完成得较慢，但优点是代码会更简单。然而，第二种选择将是首选，因为你正在卸载多个步骤，因此它应该完成得更快。尽管如此，这可能会涉及额外的复杂性，因为你可能需要协调每个单独的任务，一旦它们完成。

在接下来的章节中，你将首先了解如何采用第一种选择，即使用单个 `Task` 来运行代码块，然后再继续探讨第二种选择，即使用多个任务并协调它们的复杂性。

# 使用任务运行异步代码

`Task` 类用于异步执行代码块。它的使用已被较新的 `async` 和 `await` 关键字所取代，但本节将介绍创建任务的基础知识，因为它们在较大的或成熟的 C# 应用程序中普遍存在，并构成了 `async`/`await` 关键字的基石。

在 C# 中，有三种方法可以使用 `Task` 类及其泛型等效 `Task<T>` 来安排异步代码的运行。

## 创建新任务

你将从最简单的形式开始，即执行操作但不向调用者返回结果的形式。你可以通过调用任何`Task`构造函数并传递基于`Action`的委托来声明一个`Task`实例。此委托包含在未来的某个时刻执行的实际代码。许多构造函数重载允许使用取消令牌和`Task`运行。

一些常用的构造函数如下：

+   `public Task(Action action)`: `Action`委托代表要运行的代码主体。

+   `public Task(Action action, CancellationToken cancellationToken)`: 可以使用`CancellationToken`参数作为中断正在运行代码的方式。通常，这用于调用者已提供一种请求停止操作的手段，例如添加一个用户可以按下的`Cancel`按钮。

+   `public Task(Action action, TaskCreationOptions creationOptions)`: `TaskCreationOptions`提供了一种控制`Task`如何运行的方式，允许你向调度器提供提示，表明某个`Task`可能需要额外的时间来完成。这有助于在运行相关任务时。

以下是最常用的`Task`属性：

+   `public bool IsCompleted { get; }`: 如果`Task`已完成（完成并不表示成功），则返回`true`。

+   `public bool IsCompletedSuccessfully { get; }`: 如果`Task`成功完成，则返回`true`。

+   `public bool IsCanceled { get; }`: 如果在完成之前`Task`被取消，则返回`true`。

+   `public bool IsFaulted { get; }`: 如果在完成之前`Task`抛出了未处理的异常，则返回`true`。

+   `public TaskStatus Status { get; }`: 返回任务当前状态的指示器，例如`Canceled`、`Running`或`WaitingToRun`。

+   `public AggregateException Exception { get; }`: 如果有，返回导致`Task`提前结束的异常。

注意，`Action`委托中的代码只有在调用`Start()`方法之后才会执行。这可能是几毫秒之后，由.NET调度器决定。

从创建一个新的VS Code控制台应用程序开始，添加一个名为`Logger`的实用工具类，你将在接下来的练习和示例中使用它。它将用于将消息记录到控制台，并附带当前时间和当前线程的`ManagedThreadId`。

这些步骤如下：

1.  切换到你的源文件夹。

1.  通过运行以下命令创建一个名为`Chapter05`的新控制台应用程序项目：

    [PRE0]

1.  将`Class1.cs`文件重命名为`Logger.cs`并移除所有模板代码。

1.  一定要包含`System`和`System.Threading`命名空间。`System.Threading`包含基于`Threading`的类：

    [PRE1]

1.  将`Logger`类标记为静态，这样就可以在不创建实例的情况下使用它：

    [PRE2]

    注意

    如果你使用 `Chapter05` 命名空间，那么 `Logger` 类将可供示例和活动中的代码访问，前提是它们也使用 `Chapter05` 命名空间。如果你更喜欢为每个示例和练习创建一个文件夹，那么你应该将文件 `Logger.cs` 复制到你创建的每个文件夹中。

1.  现在声明一个名为 `Log` 的 `static` 方法，它接受一个 `string message` 参数：

    [PRE3]

当被调用时，它将使用 `WriteLine` 方法将消息记录到控制台窗口。在上面的代码片段中，C# 中的字符串插值功能使用 `$` 符号定义一个字符串；这里，`:T` 将当前时间 (`DateTime.Now`) 格式化为一个时间格式的字符串，`:00` 用于包含带前导 0 的 `Thread.ManagedThreadId`。

因此，你已经创建了一个静态的 `Logger` 类，它将在本章的其余部分中使用。

注意

你可以在 [https://packt.link/cg6c5](https://packt.link/cg6c5) 找到用于此示例的代码。

在下一个示例中，你将使用 `Logger` 类来记录线程即将开始和结束时的一些细节。

1.  首先添加一个名为 `TaskExamples.cs` 的新类文件：

    [PRE4]

1.  `Main` 入口点将记录 `taskA` 正在创建：

    [PRE5]

1.  接下来，添加以下代码：

    [PRE6]

在这里，最简单的 `Task` 构造函数传递了一个 `Action` lambda 表达式，这是你想要实际执行的代码的目标。目标代码将消息 `Inside taskA` 写入控制台。它使用 `Thread.Sleep` 暂停五秒钟以阻塞当前线程，从而模拟一个长时间运行的活动，最后将 `Leaving taskA` 写入控制台。

1.  现在你已经创建了 `taskA`，确认它只有在调用 `Start()` 方法时才会调用其目标代码。你将通过在方法调用前后立即记录一条消息来完成此操作：

    [PRE7]

1.  将 `Logger.cs` 文件的全部内容复制到与 `TaskExamples.cs` 示例相同的文件夹中。

1.  接下来运行控制台应用程序，产生以下输出：

    [PRE8]

注意，即使你调用了 `Start`，任务的状态仍然是 `WaitingToRun`。这是因为你要求 .NET 调度器安排代码运行——也就是说，将其添加到其挂起操作队列中。根据你的应用程序中其他任务的繁忙程度，它可能在你调用 `Start` 后不会立即运行。

注意

你可以在 [https://packt.link/DHxt3](https://packt.link/DHxt3) 找到用于此示例的代码。

在 C# 的早期版本中，这是直接创建和启动 `Task` 对象的主要方式。现在不再推荐使用，这里仅包括它，因为你在旧代码中可能会遇到它的使用。它的使用已被 `Task.Run` 或 `Task.Factory.StartNew` 静态工厂方法所取代，这些方法为最常见的使用场景提供了一个更简单的接口。

### 使用 Task.Factory.StartNew

静态方法 `Task.Factory.StartNew` 包含各种重载，使得创建和配置 `Task` 更加容易。注意方法被命名为 `StartNew`。它创建一个 `Task` 并自动为您启动方法。.NET 团队认识到，创建一个在首次创建后不会立即启动的 `Task` 几乎没有价值。通常，您希望 `Task` 立即开始执行其操作。

第一个参数是要执行的熟悉 `Action` 委托，后面跟着可选的取消令牌、创建选项和一个 `TaskScheduler` 实例。

以下是一些常见的重载：

+   `Task.Factory.StartNew(Action action)`: `Action` 委托包含要执行的代码，正如您之前所看到的。

+   `Task.Factory.StartNew(Action action, CancellationToken cancellationToken)`: 在这里，`CancellationToken` 协调任务的取消。

+   `Task.Factory.StartNew(Action<object> action, object state, CancellationToken cancellationToken, TaskCreationOptions creationOptions, TaskScheduler scheduler)`: `TaskScheduler` 参数允许您指定一种低级调度程序类型，该调度程序负责排队任务。此选项很少使用。

考虑以下代码，它使用了第一个也是最简单的重载：

[PRE9]

运行此代码将产生以下输出：

[PRE10]

从输出中，您可以看到此代码实现了与创建 `Task` 相同的结果，但更加简洁。要考虑的主要点是，`Task.Factory.StartNew` 被添加到 C# 中是为了使创建由您启动的任务更加容易。与直接创建任务相比，使用 `StartNew` 更为可取。

注意

在软件开发中，术语 **Factory** 通常用于表示帮助创建对象的函数。

`Task.Factory.StartNew` 提供了一种高度可配置的方式来启动任务，但实际上，许多重载很少使用，并且需要传递很多额外的参数。因此，`Task.Factory.StartNew` 本身也变得有些过时，转而使用更新的 `Task.Run` 静态方法。尽管如此，`Task.Factory.StartNew` 仍将简要介绍，因为您可能会在遗留的 C# 应用程序中看到它的使用。

### 使用 Task.Run

作为替代和首选的 `static` 工厂方法，`Task.Run` 有各种重载，后来被添加到 .NET 中以简化并缩短最常见的任务场景。对于新代码来说，使用 `Task.Run` 创建已启动的任务更为可取，因为需要的参数更少，可以完成常见的线程操作。

一些常见的重载如下：

+   `public static Task Run(Action action)`: 包含要执行的 `Action` 委托代码。

+   `public static Task Run(Action action, CancellationToken cancellationToken)`: 此外还包含一个用于协调任务取消的取消令牌。

例如，考虑以下代码：

[PRE11]

运行此代码将产生以下输出：

[PRE12]

如你所见，输出与前面两个代码片段的输出非常相似。在相关的 `Action` 委托完成之前，每个等待的时间都比前一个短。

主要的区别是直接创建 `Task` 实例是一种过时的做法，但它将允许你在显式调用 `Start` 方法之前添加一个额外的日志调用。这是直接创建 `Task` 的唯一好处，但这并不是一个特别有说服力的理由去做这件事。

同时运行这三个示例会产生以下结果：

[PRE13]

你可以看到各种 `ManagedThreadIds` 被记录，并且由于在每个情况下 `Thread.Sleep` 调用中指定的秒数逐渐减少，`taskC` 在 `taskB` 之前完成，而 `taskB` 在 `taskA` 之前完成。

更倾向于使用两种静态方法中的任何一种，但当你安排一个新的任务时应该使用哪一个？应该使用 `Task.Run`，因为 `Task.Run` 将会递归到 `Task.Factory.StartNew`。

当你有更高级的要求时，例如使用接受 `TaskScheduler` 实例的重载之一来定义任务队列的位置时，应该使用 `Task.Factory.StartNew`，但在实践中，这很少是必需的。

注意

你可以在 [https://devblogs.microsoft.com/pfxteam/task-run-vs-task-factory-startnew/](https://devblogs.microsoft.com/pfxteam/task-run-vs-task-factory-startnew/) 和 [https://blog.stephencleary.com/2013/08/startnew-is-dangerous.xhtml](https://blog.stephencleary.com/2013/08/startnew-is-dangerous.xhtml) 找到关于 `Task.Run` 和 `Task.Factory.StartNew` 的更多信息。

到目前为止，你已经看到了如何启动小任务，每个任务在完成前都有一个小延迟。这样的延迟可以模拟代码访问慢速网络连接或运行复杂计算时产生的效果。在接下来的练习中，你将通过启动运行时间越来越长的数值计算来扩展你的 `Task.Run` 知识。

这作为一个例子，展示了如何启动可能复杂的任务，并允许它们相互独立地运行到完成。请注意，在传统的同步实现中，这种计算的吞吐量会受到严重限制，因为需要在下一个操作开始之前等待一个操作完成。现在是时候通过练习来实践你所学的知识了。

## 练习 5.01：使用任务执行多个慢速运行的计算

在这个练习中，你将创建一个递归函数 Fibonacci，该函数调用自身两次来计算累积值。这是一个使用 `Thread.Sleep` 来模拟慢速调用的潜在慢速运行代码的例子。你将创建一个控制台应用程序，该程序会反复提示输入一个数字。这个数字越大，每个任务计算并输出其结果所需的时间就越长。以下步骤将帮助你完成这个练习：

1.  在 `Chapter05` 文件夹中，添加一个名为 `Exercises` 的新文件夹。在该文件夹内，添加一个名为 `Exercise01` 的新文件夹。你应该有如下文件夹结构：`Chapter05\Exercises\Exercise01`。

1.  创建一个名为 `Program.cs` 的新文件。

1.  按如下方式添加递归的 `Fibonacci` 函数。如果你请求的迭代小于或等于 `2`，可以返回 `1` 以节省一些处理时间：

    [PRE14]

1.  将 `static Main` 入口点添加到控制台应用程序中，并使用 `do`-循环提示输入一个数字。

1.  如果用户输入一个字符串，使用 `int.TryParse` 将其转换为整数：

    [PRE15]

1.  定义一个 lambda 表达式，使用 `DateTime.Now` 获取当前时间，调用慢速运行的 `Fibonacci` 函数，并记录运行时间：

    [PRE16]

lambda 表达式被传递给 `Task.Run`，并将很快由 `Task.Run` 启动，从而释放 `do-while` 循环以提示输入另一个数字。

1.  当输入空值时，程序应退出循环：

    [PRE17]

1.  对于运行控制台应用程序，首先输入数字 `1` 和 `2`。由于这些计算非常快，它们都在一秒内返回。

    [PRE18]

注意，对于 `1` 和 `2`，`ThreadId` 都是 `[04]`。这表明 `Task.Run` 在这两个迭代中都使用了相同的线程。当输入 `2` 时，之前的计算已经完成。所以 .NET 决定再次重用线程 `04`。对于值 `45`，尽管它是第三次请求，但它仍然花费了 `27` 秒来完成。

你可以看到，输入超过 `40` 的值会导致经过的时间显著增加（每次增加一个，所需时间几乎翻倍）。从更高的数字开始，向下递减，你可以看到 `41`、`40` 和 `42` 的计算都在 `44` 和 `43` 之前完成，尽管它们是在类似的时间启动的。在少数情况下，同一个线程出现了两次。同样，这是 .NET 重新使用空闲线程来运行任务的操作。

注意

你可以在 [https://packt.link/YLYd4](https://packt.link/YLYd4) 找到用于此练习的代码。

## 协调任务

在之前的 *练习 5.01* 中，你看到了如何启动多个任务并在它们完成时无需任何交互地运行。一个这样的场景是需要一个过程来搜索文件夹以查找图像文件，并为每个找到的图像文件添加版权水印。这个过程可以使用多个任务，每个任务处理一个不同的文件。不需要协调每个任务及其生成的图像。

相反，启动各种长时间运行的任务并在某些或所有任务完成后再继续的情况相当常见；也许你有一系列复杂的计算需要启动，并且只能在其他任务完成后执行最终计算。

在 *简介* 部分中，提到一个徒步旅行应用程序在使用前需要 GPS 卫星信号、导航路线和心率监测器。这些依赖项都可以使用 `Task` 创建，并且只有当所有这些都表示它们已准备好使用时，应用程序才应允许用户开始他们的路线。

在接下来的几节中，你将了解 C# 提供的各种任务协调方式。例如，你可能需要启动许多独立任务运行，每个任务运行一个复杂的计算，并在所有先前任务完成后计算一个最终值。你可能喜欢从多个网站下载数据，但希望取消耗时过长的下载。下一节将介绍这种情况。

### 等待任务完成

可以使用 `Task.Wait` 来等待单个任务完成。如果你正在处理多个任务，那么静态的 `Task.WaitAll` 方法将等待所有任务完成。`WaitAll` 重载允许传递取消和超时选项，其中大多数返回一个布尔值以指示成功或失败，如以下列表所示：

+   `public static bool WaitAll(Task[] tasks, TimeSpan timeout)`: 这接受一个等待的 `Task` 项目数组。如果 `TimeSpan` 允许表达特定单位，如小时、分钟和秒，则返回 `true`。

+   `public static void WaitAll(Task[] tasks, CancellationToken cancellationToken)`: 这接受一个等待的 `Task` 项目数组和一个可以用来协调任务取消的取消令牌。

+   `public static bool WaitAll(Task[] tasks, int millisecondsTimeout, CancellationToken cancellationToken)`: 这接受一个等待的 `Task` 项目数组和一个可以用来协调任务取消的取消令牌。`millisecondsTimeout` 指定了等待所有任务完成所需的毫秒数。

+   `public static void WaitAll(params Task[] tasks)`: 这允许等待一个 `Task` 项数组。

如果你需要等待任务列表中的任何任务完成，则可以使用 `Task.WaitAny`。所有的 `WaitAny` 重载都返回第一个完成的任务的索引号，或者在超时发生时返回 `-1`（等待的最大时间）。

例如，如果你传递一个包含五个 `Task` 项的数组，并且该数组中的最后一个 `Task` 完成，那么你将返回值四（数组索引始终从零开始计数）。

+   `public static int WaitAny(Task[] tasks, int millisecondsTimeout, CancellationToken cancellationToken)`: 这接受一个等待的 `Task` 项目数组，等待任何 `Task` 完成的毫秒数，以及一个可以用来协调任务取消的取消令牌。

+   `public static int WaitAny(params Task[] tasks)`: 这个方法接收一个 `Task` 项的数组，等待任何 `Task` 完成。

+   `public static int WaitAny(Task[] tasks, int millisecondsTimeout)`: 在这里，您传递等待任何任务完成的毫秒数。

+   `public static int WaitAny(Task[] tasks, CancellationToken cancellationToken) CancellationToken`: 这个方法接收一个可以用来协调任务取消的取消令牌。

+   `public static int WaitAny(Task[] tasks, TimeSpan timeout)`: 这个方法接收等待的最大时间周期。

调用 `Wait`、`WaitAll` 或 `WaitAny` 将阻塞当前线程，这可能会抵消最初使用任务的好处。因此，最好在可等待的任务（如 `Task.Run`）内部调用这些方法，如下例所示。

代码使用 lambda 表达式创建 `outerTask`，然后它本身创建了两个内部任务 `inner1` 和 `inner2`。使用 `WaitAny` 获取 `inner2` 将首先完成的索引，因为它暂停的时间较短，所以结果索引值将是 `1`：

[PRE19]

当代码运行时，它会产生以下输出：

[PRE20]

应用程序保持响应，因为您在 `Task` 内部调用了 `WaitAny`。您没有阻塞应用程序的主线程。如您所见，线程 ID `01` 记录了以下消息：“15:47:43 [01] 按下 ENTER”。

这种模式可以在需要一次性忘记任务的情况下使用。例如，您可能希望将信息性消息记录到数据库或日志文件中，但如果任一任务未能完成，程序流程不必要改变。

从“一次性”任务到常见进阶的例子是那些需要在一定时间限制内等待多个任务完成的场景。下一个练习将涵盖这个场景。

## 练习 5.02：在时间周期内等待多个任务完成

在这个练习中，您将启动三个长时间运行的任务，并在它们在随机选择的时间跨度内全部完成时决定您的下一步行动。

在这里，您将看到通用 `Task<T>` 类的使用。`Task<T>` 类包含一个 `Value` 属性，可以用来访问 `Task` 的结果（在这个练习中，它是一个基于字符串的泛型，因此 `Value` 将是字符串类型）。在这里您不会使用 `Value` 属性，因为这个练习的目的是展示 void 和泛型任务可以一起等待。完成以下步骤以完成此练习：

1.  将主入口点添加到控制台应用程序中：

    [PRE21]

1.  声明一个名为 `taskA` 的变量，将 `Task.Run` 传递一个 lambda 表达式，该表达式暂停当前线程 `5` 秒：

    [PRE22]

1.  使用方法组语法创建两个更多任务：

    [PRE23]

如您所知，如果编译器可以确定零参数或单参数方法所需的参数类型，则可以使用这种简短的语法。

1.  现在随机选择一个以秒为单位的最大超时时间。这意味着两个任务中的任何一个在超时期间都可能**不会**完成：

    [PRE24]

注意，每个任务仍然会运行到完成，因为您没有在`Task.Run` `Action` lambda的主体中添加停止执行代码的机制。

1.  调用`WaitAll`，传入三个任务和`timeout`超时时间：

    [PRE25]

如果所有任务都能及时完成，这将返回`true`。然后您将记录所有任务的状态，并等待按下`Enter`键以退出应用程序。

1.  最后添加两个运行缓慢的`Action`方法：

    [PRE26]

每个任务在开始和离开任务时都会记录一条消息，几秒钟后。有用的`nameof`语句用于包含方法的名称以提供额外的日志信息。通常，检查日志文件以查看已访问的方法的名称比将名称硬编码为字面字符串更有用。

1.  运行代码后，您将看到以下输出：

    [PRE27]

在运行代码时，运行时随机选择了七秒的超时时间。这使得所有任务都能及时完成，因此`WaitAll`返回`true`，此时所有任务的状态都是`RanToCompletion`。请注意，三个任务中的线程ID，用方括号括起来，是不同的。

1.  再次运行代码：

    [PRE28]

这次运行时选择了两秒的最大等待时间，因此`WaitAll`调用超时，返回`false`。

您可能已经注意到输出中`Inside TaskBActivity`有时会出现在`Inside TaskCActivity`之前。这展示了.NET调度器的排队机制。当您调用`Task.Run`时，您是在要求调度器将其添加到队列中。您调用`Task.Run`和它调用您的lambda之间可能只有几毫秒的差距，但这可能取决于您最近添加到队列中的其他任务数量；待处理任务的数量越多，这个时间间隔可能会更长。

有趣的是，输出显示了`Leaving TaskBActivity`，但`taskB`的状态在`WaitAll`等待结束后仍然是`Running`。这表明有时在超时任务的状态改变时可能会有一个非常小的延迟。

在按下`Enter`键大约三秒后，记录了`Leaving TaskA`。这表明任何超时任务中的`Action`将继续运行，.NET不会为您停止它。

注意

您可以在[https://packt.link/5lH0o](https://packt.link/5lH0o)找到用于此练习的代码。

## 延续任务

到目前为止，您已经创建了相互独立的任务，但假设您需要使用前一个任务的结果来继续一个任务怎么办？而不是通过调用`Wait`或访问`Result`属性来阻塞当前线程，您可以使用`Task`的`ContinueWith`方法来实现这一点。

这些方法返回一个新的任务，称为**延续**任务，或者更简单地说，延续，它可以消费前一个任务或前驱的结果。

与标准任务一样，它们不会阻塞调用线程。有几个`ContinueWith`重载可用，许多允许广泛的定制。以下是一些更常用的重载：

+   `public Task ContinueWith(Action<Task<TResult>> continuationAction)`: 这定义了一个基于泛型`Action<T>`的`Task`，在先前的任务完成时运行。

+   `public Task ContinueWith(Action<Task<TResult>> continuationAction, CancellationToken cancellationToken)`: 这有一个要运行的任务和一个可以用来协调任务取消的取消令牌。

+   `public Task ContinueWith(Action<Task<TResult>> continuationAction, TaskScheduler scheduler)`: 这也有一个要运行的任务和一个低级别的`TaskScheduler`，可以用来排队任务。

+   `public Task ContinueWith(Action<Task<TResult>> continuationAction, TaskContinuationOptions continuationOptions)`: 一个要运行的任务，其行为由`TaskContinuationOptions`指定。例如，指定`NotOnCanceled`表示如果先前的任务被取消，则不调用后续操作。

后续操作有一个初始的`WaitingForActivation`状态。.NET Framework将在先前的任务或任务完成时执行此任务。需要注意的是，您不需要启动后续操作，尝试这样做将导致异常。

以下示例模拟调用一个长时间运行的功能，`GetStockPrice`（这可能是一种需要几秒钟才能返回的网页服务或数据库调用）：

[PRE29]

调用`GetStockPrice`返回一个`double`，这导致将泛型`Task<double>`作为后续操作传递（请参阅突出显示的部分）。`prev`参数是一个类型为`Task<double>`的泛型`Action`，允许您访问先前的任务及其`Result`以检索从`GetStockPrice`返回的值。

如果您将鼠标悬停在`ContinueWith`方法上，您将看到以下IntelliSense描述：

![图5.1：ContinueWith方法签名](img/B16835_05_01.jpg)

图5.1：ContinueWith方法签名

注意

`ContinueWith`方法有多种选项可以用来微调行为，您可以从[https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.taskcontinuationoptions](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.taskcontinuationoptions)获取更多详细信息。

运行示例会产生类似于以下输出的结果：

[PRE30]

在输出中，线程 `[01]` 代表控制台的主线程。调用`GetStockPrice`的任务是由线程ID `[03]` 执行的，但后续操作是使用不同的线程（线程 `[04]`）执行的。

注意

您可以在[https://packt.link/rpNcx](https://packt.link/rpNcx)找到此示例使用的代码。

在不同线程上运行的延续操作可能不会成问题，但如果你正在开发 UWP、WPF 或 WinForms UI 应用程序，并且必须使用主 UI 线程来更新 UI 元素（除非你使用绑定语义），那么这肯定是一个问题。

值得注意的是，可以使用 `TaskContinuationOptions.OnlyOnRanToCompletion` 选项来确保延续操作仅在先决任务首先完成时运行。例如，你可能创建一个 `Task` 来从数据库中检索客户的订单，然后使用延续任务来计算平均订单价值。如果先前的任务失败或被用户取消，那么如果用户不再关心结果，就没有必要浪费处理能力来计算平均值。

注意

`ContinueWith` 方法有多种选项可以用来微调行为，你可以查看 [https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.taskcontinuationoptions](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.taskcontinuationoptions) 获取更多详细信息。

如果你访问正在抛出的 `AggregateException` 上的 `Task<T> Result` 属性。这将在稍后详细说明。

### 使用 Task.WhenAll 和 Task.WhenAny 与多个任务

你已经看到了如何使用单个任务来创建延续任务，但如果你有多个任务，需要在之前的任务完成时执行最终操作，该怎么办呢？

之前，使用 `Task.WaitAny` 和 `Task.WaitAll` 方法来等待任务完成，但这些会阻塞当前线程。这就是 `Task.WhenAny` 和 `Task.WhenAll` 可以使用的地方。它们返回一个新的 `Task`，其 `Action` 委托在 **任何** 或 **所有** 先前的任务完成时被调用。

有四个 `WhenAll` 重载，两个返回 `Task`，两个返回泛型 `Task<T>`，允许访问任务的结果：

1.  `public static Task WhenAll(IEnumerable<Task> tasks)`: 当任务集合完成时，这个操作会继续执行。

1.  `public static Task WhenAll(params Task[] tasks)`: 当任务数组完成时，这个操作会继续执行。

1.  `public static Task<TResult[]> WhenAll<TResult>(params Task<TResult>[] tasks)`: 当泛型 `Task<T>` 项的数组完成时，这个操作会继续执行。

1.  `public static Task<TResult[]> WhenAll<TResult>(IEnumerable<Task<TResult>> tasks)`: 当泛型 `Task<T>` 项的集合完成时，这个操作会继续执行。

`WhenAny` 具有一组类似的重载，但返回的是 `Task` 或 `Task<T>`，这在实践中相当于 `WhenAll` 和 `WhenAny`。

## 练习 5.03：等待所有任务完成

假设你被一位汽车经销商要求创建一个控制台应用程序，用于计算不同地区销售的汽车的平均销售价值。经销商是一个繁忙的地方，但他们知道获取和计算平均值可能需要一段时间。因此，他们希望输入他们准备等待平均计算的最大秒数。如果时间更长，他们将离开应用程序并忽略结果。

经销商有10个区域销售中心。为了计算平均值，首先需要调用一个名为`FetchSales`的方法，该方法为这些区域中的每一个返回一个`CarSale`项目的列表。

每次调用`FetchSales`可能是一个可能运行缓慢的服务（你将实现随机暂停来模拟这种延迟），因此你需要为每个调用使用一个`Task`，因为你不能确定每个调用需要多长时间才能完成。你也不希望运行缓慢的任务影响其他任务，但为了计算有效的平均值，在继续之前，重要的是要**所有**结果都返回。

创建一个`SalesLoader`类，实现`IEnumerable<CarSale> FetchSales()`以返回汽车销售详情。然后，一个`SalesAggregator`类应该传递一个`SalesLoader`列表（在这个练习中，将有10个加载器实例，每个地区一个）。聚合器将等待所有加载器完成使用`Task.WhenAll`，然后再继续执行一个计算所有地区平均值的任务。

执行以下步骤：

1.  首先，创建一个`CarSale`记录。构造函数接受两个值，汽车的名字和其销售价格（`name`和`salePrice`）：

    [PRE31]

1.  现在创建一个接口，`ISalesLoader`，它表示销售数据加载服务：

    [PRE32]

它只有一个调用，`FetchSales`，返回一个类型为`CarSale`的可枚举。现在，了解加载器的工作原理并不重要；只需知道调用时会返回汽车销售列表。在这里使用接口允许根据需要使用各种类型的加载器。

1.  使用聚合类调用`ISalesLoader`实现：

    [PRE33]

它被声明为`static`，因为在调用之间没有状态。定义一个`Average`函数，该函数接受一个`ISalesLoader`项目的可枚举，并返回一个通用的`Task<Double>`用于最终的平均值计算。

1.  对于每个加载器参数，使用LINQ投影将`loader.FetchSales`方法传递给`Task.Run`：

    [PRE34]

这些都会返回一个`Task<IEnumerable<CarSale>>`实例。`WhenAll`用于创建一个在`ContinueWith`调用时继续的单个任务。

1.  使用LINQ的`SelectMany`从每个加载器调用结果中获取所有的`CarSale`项目，在调用Linq的`Average`之前对每个`CarSale`项目的`SalePrice`字段进行操作：

    [PRE35]

1.  从一个名为`SalesLoader`的类中实现`ISalesLoader`接口：

    [PRE36]

构造函数将传递一个用于日志记录的`int`变量和一个`Random`实例，以帮助创建随机数量的`CarSale`项目。

1.  你的`ISalesLoader`实现需要一个`FetchSales`函数。包括`1`到`3`秒之间的随机延迟来模拟不太可靠的服务：

    [PRE37]

你正在尝试测试你的应用程序在各种时间延迟下的行为。因此，使用了随机类。

1.  使用`Enumerable.Range`和`random.Next`来选择一个介于一和五之间的随机数：

    [PRE38]

这是使用你的`GetRandomCar`函数返回的`CarSale`项目总数。

1.  使用`GetRandomCar`生成一个具有随机制造商名称的`CarSale`项目，该名称来自硬编码的列表。

1.  使用`carNames.length`属性来选择一个介于零和四之间的随机索引号用于汽车名称：

    [PRE39]

1.  现在，创建你的控制台应用程序来测试这个功能：

    [PRE40]

你的应用程序将反复询问用户愿意等待的最大时间，以便在下载数据时使用。一旦所有数据都下载完毕，应用程序将使用此信息来计算平均价格。单独按下`Enter`键将导致程序循环结束。`MaxSalesHubs`是请求数据的最大销售中心数。

1.  将输入的值转换为`int`类型，然后再次使用`Enumerable.Range`来创建一个随机数量的新`SalesLoader`实例（你有最多10个不同的销售中心）：

    [PRE41]

1.  将加载器传递给静态`SalesAggregator.Average`方法以接收一个`Task<Double>`。

1.  调用`Wait`，传入最大等待时间：

    [PRE42]

如果`Wait`调用在规定时间内返回，那么你将看到`has completed`的值为`true`。

1.  最后，检查`hasCompleted`并相应地记录消息：

    [PRE43]

1.  当运行控制台应用程序并输入短的最大等待时间`1`秒时，你会看到随机创建的三个加载器实例：

    [PRE44]

每个加载器在返回一个随机的`CarSale`记录列表之前会睡眠`1`秒（你可以看到各种线程ID被记录），然后达到最大超时值，因此显示没有平均值的`Timeout!`消息。

1.  输入一个更大的超时时间`10`秒：

    [PRE45]

输入`10`秒的值允许`7`个随机加载器及时完成并最终创建平均值为`639`的值。

注意

你可以在[https://packt.link/kbToQ](https://packt.link/kbToQ)找到用于此练习的代码。

到目前为止，本章已经考虑了创建单个任务的各种方法以及如何使用静态`Task`方法来创建为我们启动的任务。你看到了如何使用`Task.Factory.StartNew`来创建配置任务，尽管它有一组更长的配置参数。最近添加到C#中的`Task.Run`方法，在大多数常规场景下，由于其更简洁的签名而更受欢迎。

使用延续，单个和多个任务可以独立运行，只有当所有或任何前面的任务都完成时，才会继续执行最终任务。

现在是时候看看`async`和`await`关键字来运行异步代码了。这些关键字是C#语言中相对较新的添加。`Task.Factory.StartNew`和`Task.Run`方法可以在旧的C#应用程序中找到，但希望你会看到`async`/`await`提供了更清晰的语法。

# 异步编程

到目前为止，你已经创建了任务，并使用静态`Task`工厂方法来运行和协调这些任务。在C#的早期版本中，这些是创建任务的唯一方式。

C#语言现在提供了`async`和`await`关键字，这使得`async`/`await`风格的代码更简洁，所创建的代码通常更容易理解，因此也更容易维护。

注意

你可能会经常发现，遗留的并发启用应用程序最初是使用`Task.Factory.StartNew`方法创建的，随后更新为使用等效的`Task.Run`方法，或者直接更新为`async`/`await`风格。

`async`关键字表示方法在完成其操作之前将返回给调用者，因此调用者应该在某个时间点等待其完成。

将`async`关键字添加到方法中指示编译器它可能需要生成额外的代码来创建状态机。本质上，状态机将你的原始方法中的逻辑提取到一系列委托和局部变量中，允许代码在`await`表达式之后继续执行到下一个语句。编译器生成可以跳回方法中相同位置的委托。

注意

你通常不会看到额外的编译代码，但如果你对C#中的状态机感兴趣，可以访问[https://devblogs.microsoft.com/premier-developer/dissecting-the-async-methods-in-c](https://devblogs.microsoft.com/premier-developer/dissecting-the-async-methods-in-c)了解更多信息。

添加`async`关键字并不意味着`async`方法立即执行，它一开始是同步运行的，直到遇到带有`await`关键字的代码段。在此点，会检查可等待的代码块（在以下示例中，由于前面的`async`关键字，`BuildGreetings`调用是可等待的）是否已经完成。如果是，它将继续同步执行。如果不是，异步方法将被暂停，并返回一个不完整的`Task`给调用者。这将在`async`代码完成时完成。

在以下控制台应用程序中，入口点`static Main`已被标记为`async`，并添加了`Task`返回类型。你不能将返回`int`或`void`的`Main`入口点标记为`async`，因为当控制台应用程序关闭时，运行时必须能够返回`Task`结果给调用环境：

[PRE46]

运行示例会产生如下输出：

[PRE47]

当 `Main` 运行时，它记录 `Starting`。注意 `ThreadId` 是 `[01]`。正如你之前看到的，控制台应用程序的主线程编号为 `1`（因为 `Logger.Log` 方法使用 `00` 格式字符串，它为范围零到九的数字添加前导 `0`）。

然后调用异步方法 `BuildGreetings`。它将字符串变量 `message` 设置为 `"Morning"` 并记录消息。`ThreadId` 仍然是 `[01]`；这目前是同步运行的。

到目前为止，你一直使用 `Thread.Sleep` 来阻塞调用线程以模拟或模拟长时间运行的操作，但 `async`/`await` 使得使用静态 `Task.Delay` 方法模拟慢动作并等待该调用变得更容易。`Task.Delay` 返回一个任务，因此它也可以用于后续任务。

使用 `Task.Delay`，你将进行两个不同的可等待调用（一个等待 10 秒，另一个等待两秒），然后继续并将它们附加到你的本地 `message` 字符串。这两个 `Task.Delay` 调用可以是你的代码中返回 `Task` 的任何方法。

这里很棒的一点是，每个等待的部分都会按照它在代码中声明的顺序获得其正确的状态，无论之前等待了 10 秒（或两秒）。线程 ID 都已从 `[01]` 变为 `[04]`。这告诉你不同的线程正在运行这些语句。甚至最后的 `Press Enter` 消息也有一个与原始线程不同的线程。

`Async/await` 使得使用熟悉的 `WhenAll`、`WhenAny` 和 `ContinueWith` 方法交替运行一系列基于任务的操作变得更容易。

以下示例展示了如何在程序的不同阶段使用各种可等待调用混合应用多个 `async`/`await` 调用。这模拟了一个调用数据库（`FetchPendingAccounts`）以获取用户账户列表的应用程序。待处理账户列表中的每个用户都分配了一个唯一的 ID（每个用户使用一个任务）。

根据用户的区域，`Task.WhenAll` 调用中的创建账户操作表示一切已完成。

[PRE48]

使用 `enum` 定义一个 `RegionName`：

[PRE49]

`User` 记录构造函数接收一个 `userName` 和用户的 `region`：

[PRE50]

`AccountGenerator` 是主要的控制类。它包含一个可以被控制台应用程序等待的 `async` `CreateAccounts` 方法（这在示例的末尾实现）：

[PRE51]

使用 `await` 关键字，你定义了对 `FetchPendingAccounts` 的可等待调用：

[PRE52]

对于 `FetchPendingAccounts` 返回的每个用户，你都会对 `GenerateId` 进行一个可等待调用。这表明循环可以包含多个可等待调用。运行时将为正确的用户实例设置用户 ID：

[PRE53]

使用 Linq 的 `Select` 函数，你创建了一个任务列表。根据用户的区域，为每个用户创建一个北方或其他账户（每个调用都是一个针对用户的 `Task`）：

[PRE54]

使用`static`的`WhenAll`调用等待账户创建任务列表。一旦完成，`UpdatePendindAccounts`将被调用，并传递更新的用户列表。这表明你可以在`async`语句之间传递任务列表：

[PRE55]

`FetchPendingAccounts`方法返回一个包含用户列表的`Task`（在这里，你使用`Task.Delay`模拟了`3`秒的延迟）：

[PRE56]

`GenerateId`使用`Task.FromResult`通过`Guid`类生成一个全局唯一ID。`Task.FromResult`用于当你想要返回一个结果但不需要创建一个运行的任务时，就像使用`Task.Run`一样：

[PRE57]

两个`bool`任务方法创建一个北方账户或其他账户。在这里，你返回`true`以指示每个账户创建调用都成功，无论：

[PRE58]

接下来，`UpdatePendingAccounts`传递一个用户列表。对于每个用户，你创建一个任务来模拟一个慢速运行的调用以更新每个用户，并返回随后更新的用户数量：

[PRE59]

最后，控制台应用程序创建一个`AccountGenerator`实例，在写入`All done`消息之前等待`CreateAccounts`完成：

[PRE60]

运行控制台应用程序会产生以下输出：

[PRE61]

在这里，你可以看到线程`[01]`写入`Starting`消息。这是应用程序的主线程。注意，主线程还从`FetchPendingAccounts`方法中写入`Fetching pending accounts...`。这仍然是以同步方式运行的，因为可等待的块（`Task.Delay`）尚未到达。

线程`[4]`、`[5]`和`[7]`创建四个用户账户中的每一个。你使用`Task.Run`调用`CreateNorthernAccount`或`CreateOtherAccount`方法。线程`[5]`运行`CreateAccounts: Updated 4 pending accounts`中的最后一个语句。线程号可能因系统而异，因为.NET使用一个基于每个线程繁忙程度的内部线程池。

注意

你可以在[https://packt.link/ZIK8k](https://packt.link/ZIK8k)找到此示例使用的代码。

## 异步Lambda表达式

第3章*委托、事件和Lambda表达式*探讨了Lambda表达式及其如何用于创建简洁的代码。你还可以在Lambda表达式中使用`async`关键字来创建包含各种`async`代码的事件处理程序代码。

以下示例使用`WebClient`类展示从网站下载数据的两种不同方式（这将在第8章*创建和使用Web API客户端*和第9章*创建API服务*中详细讨论）。

[PRE62]

在这里，你使用带有`async`关键字的Lambda语句将你自己的事件处理程序添加到`WebClient`类的`DownloadDataCompleted`事件中。编译器将允许你在Lambda表达式的主体内部添加可等待的调用。

在调用`DownloadData`并为我们下载所需数据之后，此事件将被触发。代码使用可等待的块`Task.Delay`在另一个线程上模拟一些额外的处理：

[PRE63]

你调用`DownloadData`方法，传入你的URL，然后记录接收到的网络数据的长度。这个特定的调用本身会阻塞主线程，直到数据下载完成。`WebClient`提供了`DownloadData`方法的基于任务的异步版本，称为`DownloadDataTaskAsync`。因此，建议使用更现代的`DownloadDataTaskAsync`方法，如下所示：

[PRE64]

再次强调，你请求相同的URL，但可以简单地使用`await`语句，该语句将在数据下载完成后执行。正如你所见，这需要更少的代码，并且语法更简洁：

[PRE65]

运行代码会产生以下输出：

[PRE66]

注意

在运行程序时，你可能会看到以下警告：“`Warning SYSLIB0014: 'WebClient.WebClient()' is obsolete: 'WebRequest, HttpWebRequest, ServicePoint, and WebClient are obsolete. Use HttpClient instead.'"`。在这里，Visual Studio建议使用`HttpClient`类，因为`WebClient`已被标记为过时。

`DownloadData`由线程`[01]`（主线程）记录，该线程在下载完成前大约被阻塞一秒钟。然后使用`downloadBytes.Length`属性记录下载文件的长度。

`DownloadDataTaskAsync`请求由线程`06`处理。最后，`DownloadDataCompleted`事件处理器内部的延迟代码通过线程`04`完成。

注意

你可以在[https://packt.link/IJEaU](https://packt.link/IJEaU)找到这个示例使用的代码。

## 取消任务

任务取消是一个两步过程：

+   你需要添加一种请求取消的方式。

+   任何可取消的代码都需要支持这一点。

如果没有这两种机制，你无法提供取消功能。

通常，你会启动一个支持取消的长运行任务，并允许用户通过在UI上按按钮来取消操作。在许多实际场景中需要这样的取消，例如图像处理，需要修改多个图像，如果用户时间不够，允许他们取消剩余的任务。另一个常见场景是向不同的Web服务器发送多个数据请求，并在收到第一个响应后立即取消慢速运行或挂起的请求。

在C#中，`CancellationTokenSource`作为一个顶级对象，通过其`Token`属性`CancellationToken`来发起取消请求，并将其传递给可以定期检查并对此取消状态做出响应的并发/慢速运行代码。理想情况下，你不会希望低级方法随意取消高级操作，因此源和令牌之间有明确的分离。

`CancellationTokenSource`有各种构造函数，包括一个在指定时间过后会发起取消请求的构造函数。以下是`CancellationTokenSource`的一些方法，提供了多种发起取消请求的方式：

+   `public bool IsCancellationRequested { get; }`: 如果已请求取消此令牌源（调用者已调用`Cancel`方法），则该属性返回`true`。这可以在目标代码的间隔中检查。

+   `public CancellationToken Token { get; }`: 与此源对象关联的`CancellationToken`通常传递给`Task.Run`的重载版本，允许.NET检查挂起任务的状态，或者允许您的代码在运行时进行检查。

+   `public void Cancel()`: 启动取消请求。

+   `public void Cancel(bool throwOnFirstException)`: 启动取消请求并确定是否在发生异常时进一步处理操作。

+   `public void CancelAfter(int millisecondsDelay)`: 在指定数量的毫秒后安排取消请求。

`CancellationTokenSource`具有`Token`属性。`CancellationToken`包含各种方法和属性，可用于代码检测取消请求：

+   `public bool IsCancellationRequested { get; }`: 该属性返回`true`，如果已请求取消此令牌。

+   `public CancellationTokenRegistration Register(Action callback)`: 允许代码注册一个在令牌被取消时由系统执行的委托。

+   `public void ThrowIfCancellationRequested()`: 调用此方法将在请求取消时抛出`OperationCanceledException`。这通常用于跳出循环。

在前面的示例中，您可能已经注意到`CancellationToken`可以传递给许多静态`Task`方法。例如，`Task.Run`、`Task.Factory.StartNew`和`Task.ContinueWith`都包含接受`CancellationToken`的重载版本。

.NET不会尝试中断或停止任何正在运行的代码，无论您在`CancellationToken`上调用`Cancel`多少次。本质上，您将这些令牌传递到目标代码中，但该代码必须在其能够时定期检查取消状态，例如在循环中，然后决定如何响应。这在逻辑上是合理的；.NET如何知道何时可以安全地中断一个方法，比如一个可能有数百行代码的方法？

将`CancellationToken`传递给`Task.Run`仅向队列调度器提供提示，表明可能不需要启动任务的操作，但一旦启动，.NET不会为您停止正在运行的代码。运行中的代码本身必须随后观察取消状态。

这与一个行人等待在交通灯处过马路的情况类似。机动车辆可以被视为在其他地方已启动的任务。当行人到达交叉路口并按下按钮（在`CancellationTokenSource`上调用`Cancel`）时，交通灯最终应该变为红色，以便请求移动的车辆停止。是否停止车辆取决于每个驾驶员是否注意到红灯已变亮（检查`IsCancellationRequested`），然后决定停止他们的车辆。交通灯不会强制停止每辆车（.NET 运行时）。如果驾驶员注意到后面的车辆太近，并且很快停止可能会导致碰撞，他们可能会决定不立即停车。一个完全不观察交通灯状态的驾驶员可能会未能停车。

下面的章节将继续通过练习展示`async`/`await`的实际应用，一些常用的取消任务选项，在这些选项中，你需要控制是否允许挂起的任务完成或中断，以及何时应该尝试捕获异常。

## 练习5.04：取消长时间运行的任务

你将分两部分创建这个练习：

+   使用返回基于双精度浮点数结果的`Task`。

+   第二个提供了通过检查`Token.IsCancellationRequested`属性提供精细级别控制的选项。

执行以下步骤来完成这个练习：

1.  创建一个名为`SlowRunningService`的类。正如其名所示，服务内部的方法已被设计为执行缓慢：

    [PRE67]

1.  添加第一个慢速运行的操作`Fetch`，它接受一个延迟时间（通过简单的`Thread.Sleep`调用实现），以及取消令牌，你将其传递给`Task.Run`：

    [PRE68]

当调用`Fetch`时，在休眠线程醒来之前，令牌可能会被取消。

1.  为了测试`Fetch`是否会停止运行或返回一个数字，添加一个控制台应用程序来测试这个。在这里，使用默认延迟（`DelayTime`）为`3`秒：

    [PRE69]

1.  添加一个辅助函数来提示你准备等待的最大秒数。如果输入了有效的数字，将输入的值转换为`TimeSpan`：

    [PRE70]

1.  为控制台应用程序添加一个标准的`Main`入口点。这个入口点标记为异步并返回一个`Task`：

    [PRE71]

1.  创建服务的一个实例。你将在循环中使用相同的实例，很快：

    [PRE72]

1.  现在添加一个`do`循环，反复请求最大延迟时间：

    [PRE73]

这允许你尝试不同的值，以查看它如何影响取消令牌和返回的结果。在`null`值的情况下，你将退出`do`循环。

1.  创建`CancellationTokenSource`，传入最大等待时间：

    [PRE74]

这将触发取消，而无需你自己调用`Cancel`方法。

1.  使用`CancellationToken.Register`方法，传递一个在令牌被信号取消时要调用的`Action`委托。在这里，简单地在发生这种情况时记录一条消息：

    [PRE75]

1.  现在对于主要活动，调用服务的`Fetch`方法，传入默认的`DelayTime`和令牌：

    [PRE76]

1.  在你等待`resultTask`之前，添加一个`try-catch`块来捕获任何`TaskCanceledException`：

    [PRE77]

当使用可取消的任务时，它们可能会抛出`TaskCanceledException`。在这种情况下，这是可以接受的，因为你确实期望这种情况发生。请注意，你只有在任务被标记为`IsCompletedSuccessfully`时才访问`resultTask.Result`。如果你尝试访问一个已故障任务的`Result`属性，则会抛出`AggregateException`实例。在一些较旧的项目中，你可能看到捕获`AggregateException`的非异步/await代码。

1.  运行应用并输入一个大于三秒预计到达时间的等待时间，在这个例子中是`5`：

    [PRE78]

如预期的那样，在完成之前令牌没有被取消，所以你看到`Result=3`（已过时间，单位为秒）。

1.  再试一次。为了触发并检测取消，将秒数输入为`2`：

    [PRE79]

注意，当`Fetch`唤醒时，记录了`Cancelled token`消息，但你最终仍然收到了`3`秒的结果，没有`TaskCanceledException`消息。这强调了将取消令牌传递给`Start.Run`不会停止任务动作的启动，更重要的是，它也没有中断它。

1.  最后，使用`0`作为最大等待时间，这将有效地立即触发取消：

    [PRE80]

你将看到取消令牌消息和捕获到的`TaskCanceledException`，但没有任何`Sleeping`或`Awake`消息被记录。这表明传递给`Task.Run`的`Action`实际上并没有被运行时启动。当你将`CancelationToken`传递给`Start.Run`时，任务的动作会被排队，但如果`TaskScheduler`在启动之前注意到令牌已被取消，它将不会运行该动作；它只会抛出`TaskCanceledException`。

现在对于一种替代的慢速运行方法，它允许你通过轮询取消状态的变化来支持可取消的动作。

1.  在`SlowRunningService`类中，添加一个`FetchLoop`函数：

    [PRE81]

这会产生与之前`Fetch`函数类似的结果，但其目的是展示一个函数如何被分解成一个重复的循环，该循环在每次循环迭代运行时能够检查`CancellationToken`。

1.  定义`for...next`循环的主体，该循环在每次迭代中检查`IsCancellationRequested`属性是否为`true`，如果检测到请求取消，则简单地返回一个可空的`double`值：

    [PRE82]

这是一种相当坚决的退出循环的方式，但就这段代码而言，不需要做其他任何事情。

注意

你也可以使用`continue`语句并在返回之前进行清理。另一种选择是调用`token.ThrowIfCancellationRequested()`而不是检查`token.IsCancellationRequested`，这将迫使你退出`for`循环。

1.  在 `Main` 控制台应用程序中，添加一个类似的 `while` 循环，这次调用 `FetchLoop` 方法。代码与之前的循环代码类似：

    [PRE83]

1.  现在调用 `FetchLoop` 并等待结果：

    [PRE84]

1.  运行控制台应用程序并使用 `5` 秒最大值允许所有迭代运行，没有任何检测到取消请求。结果是预期的 `3`：

    [PRE85]

1.  使用 `2` 作为最大值。这次在迭代 `4` 时自动触发令牌，并在迭代 `5` 时被发现，因此返回了一个空结果：

    [PRE86]

1.  使用 `0`，您将看到与之前的 `Fetch` 示例相同的输出：

    [PRE87]

动作没有机会运行。您可以看到一个 `Cancelled token` 消息和 `TaskCanceledException` 被记录。

通过运行此练习，您已经看到长时间运行的任务如果未在指定时间内完成，.NET 运行时将自动将其标记为取消。通过使用 `for` 循环，任务被分解成小的迭代步骤，这提供了频繁检测是否请求取消的机会。

注意

您可以在 [https://packt.link/xa1Yf](https://packt.link/xa1Yf) 找到用于此练习的代码。

## 异步/等待代码中的异常处理

您已经看到取消任务可能导致抛出 `TaskCanceledException`。异步代码的异常处理可以像标准同步代码一样实现，但您需要注意一些事项。

当 `async` 方法中的代码抛出异常时，任务的状态被设置为 **Faulted**。然而，异常不会重新抛出，直到等待的表达式被重新安排。这意味着如果您不等待调用，则可能抛出异常，并且这些异常在代码中可能完全未被观察。

除非您绝对无法避免，否则不应创建 `async void` 方法。这样做会使调用者难以等待您的代码。这意味着他们无法捕获抛出的任何异常，默认情况下，这会导致程序终止。如果调用者没有 `Task` 引用以等待，那么他们就没有办法知道被调用的方法是否运行完成。

此指南的一般例外是在本章开头提到的“一次性”方法。异步记录应用程序使用情况的方法可能不是那么关键，因此您可能不关心这些调用是否成功。

可以检测和处理未观察到的任务异常。如果您将事件委托附加到静态 `TaskScheduler.UnobservedTaskException` 事件，您将收到通知，表示任务异常未被观察。您可以通过以下方式将委托附加到该事件：

[PRE88]

当任务对象被最终化后，运行时会将任务异常视为 **未观察到的**。

注意

您可以在 [https://packt.link/OkH7r](https://packt.link/OkH7r) 找到用于此示例的代码。

继续使用一些异常处理示例，看看你如何可以像同步代码一样捕获特定类型的异常。

在以下示例中，`CustomerOperations`类提供了`AverageDiscount`函数，它返回`Task<int>`。然而，它可能会抛出`DivideByZeroException`，所以你需要捕获它；否则，程序将崩溃。

[PRE89]

创建一个`CustomerOperations`实例并等待`AverageDiscount`方法返回一个值：

[PRE90]

在`0`和`2`之间选择一个随机的`ordercount`值。尝试除以零将导致.NET运行时抛出异常：

[PRE91]

结果显示，当`orderCount`为零时，你确实如预期那样捕获了`DivideByZeroException`：

[PRE92]

第二次运行时，没有捕获到错误：

[PRE93]

在你的系统上，你可能需要多次运行程序，`DivideByZeroException`才会被引发。这是由于使用了随机实例来分配`orderCount`的值。

注意

你可以在[https://packt.link/18kOK](https://packt.link/18kOK)找到这个示例使用的代码。

所以到目前为止，你已经创建了可能会抛出异常的单个任务。接下来的练习将查看一个更复杂的变体。

## 练习5.05：处理异步异常

假设你有一个`CustomerOperations`类，它可以用来通过`Task`获取客户列表。对于每个客户，你需要运行一个额外的`async`任务，该任务将去到一个服务中计算该客户订单的总价值。

一旦你有了客户列表，需要按销售额降序对客户进行排序，但由于一些安全限制，如果你读取的客户区域名称是`West`，则不允许读取客户的`TotalOrders`属性。在这个练习中，你将创建一个在早期示例中使用的`RegionName`枚举的副本。

执行以下步骤来完成这个练习：

1.  首先添加`Customer`类：

    [PRE94]

构造函数传递客户的`name`和他们的`region`，以及一个标识`protectedRegion`名称的第二个区域。如果客户的`region`与这个`protectedRegion`相同，则在尝试读取`TotalOrders`属性时抛出访问违规异常。

1.  然后添加一个`CustomerOperations`类：

    [PRE95]

这个类知道如何加载一个客户的名字并填充他们的总订单价值。这里的要求是来自`West`区域的客户需要有一个硬编码的限制，所以添加一个名为`ProtectedRegion`的常量，其值为`RegionName.West`。

1.  添加一个`FetchTopCustomers`函数：

    [PRE96]

这将返回一个`Customer`的`Task`枚举，并且被标记为`async`，因为你在函数内部将进行进一步的`async`调用以填充每个客户的订单详情。使用`Task.Delay`来模拟一个慢速运行的操作。在这里，有一个硬编码的客户样本列表。创建每个`Customer`实例，传递他们的名字、实际区域和受保护的区域常量`ProtectedRegion`。

1.  在 `FetchOrders` 中添加一个 `await` 调用（稍后将声明）：

    [PRE97]

1.  现在，遍历客户列表，但请确保将每个对 `TotalOrders` 的调用用 `try-catch` 块包装起来，该块明确检查如果尝试查看受保护的客户将抛出的访问违规异常：

    [PRE98]

1.  现在，`filteredCustomers` 列表已经填充了一个过滤后的客户列表，使用 Linq 的 `OrderByDescending` 扩展方法按每个客户的 `TotalOrders` 值返回项目：

    [PRE99]

1.  完成带有 `FetchOrders` 实现的 `CustomerOperations`。

1.  对于列表中的每个客户，使用一个暂停 `500` 毫秒的 `async` lambda，然后为 `TotalOrders` 分配一个随机值：

    [PRE100]

延迟可能代表另一个运行缓慢的服务。

1.  使用 `Task.WhenAll` 等待 `orderUpdateTasks` 完成：

    [PRE101]

1.  现在创建一个控制台应用程序来运行操作：

    [PRE102]

1.  在运行控制台时，没有错误，因为来自 `West` 区域的 `Roy Batty` 被安全地跳过了：

    [PRE103]

在这个练习中，您看到了如何使用异步代码优雅地处理异常。您在所需位置放置了一个 `try-catch` 块，而不是过度复杂化并添加过多的不必要的嵌套 `try-catch` 块。当代码运行时，捕获了一个异常而没有使应用程序崩溃。

注意

您可以在 [https://packt.link/4ozac](https://packt.link/4ozac) 找到用于此练习的代码。

## AggregateException 类

在本章开头，您看到 `Task` 类有一个 `Exception` 属性，其类型为 `AggregateException`。此类包含有关在异步调用期间发生的一个或多个错误的详细信息。

`AggregateException` 有各种属性，但主要如下：

+   `public ReadOnlyCollection<Exception> InnerExceptions { get; }`: 由当前异常引起的异常集合。单个异步调用可能导致多个异常被引发并收集在此处。

+   `public AggregateException Flatten()`: 将 `InnerExeceptions` 属性中的所有 `AggregateException` 实例合并为一个单一的新实例。这可以避免您需要遍历嵌套在异常列表中的 `AggregateException`。

+   `public void Handle(Func<Exception, bool> predicate)`: 对此聚合异常中的每个异常调用指定的 Func 处理程序。这允许处理程序返回 `true` 或 `false` 以指示每个异常是否被处理。任何剩余未处理的异常将抛出，由调用者按需捕获。

当出现问题时并且此异常被调用者捕获时，`InnerExceptions` 包含导致当前异常的异常列表。这些可能来自多个任务，因此每个单独的异常都被添加到结果任务的 `InnerExceptions` 集合中。

你可能会经常遇到带有`try-catch`块来捕获`AggregateException`并记录每个`InnerExceptions`详细信息的`async`代码。在这个例子中，`BadTask`返回一个基于`int`的任务，但它运行时可能会引发异常。执行以下步骤来完成此示例：

[PRE104]

如果传入的数字是偶数（使用`%`运算符来查看数字是否可以被`2`整除且没有余数），则它会在抛出`InvalidOperationException`之前睡眠`1,000`毫秒：

[PRE105]

添加一个辅助函数`CreateBadTasks`，该函数创建一个包含五个坏任务的集合。当启动时，每个任务最终都会抛出`InvalidOperationException`类型的异常：

[PRE106]

现在，创建控制台应用程序的`Main`入口点。你将`CreateBadTasks`的结果传递给`WhenAll`，传递字符串`[WhenAll]`以便更容易地在输出中看到正在发生的事情：

[PRE107]

在尝试等待`whenAllCompletedTask`任务之前，你需要将其包裹在`try-catch`中，以捕获基础`Exception`类型（如果你期望更具体的一个，则可以捕获更具体的一个）。

你不能在这里捕获`AggregateException`，因为它是你接收到的`Task`中的第一个异常，但你仍然可以使用`whenAllCompletedTask`的`Exception`属性来获取`AggregateException`本身：

[PRE108]

你已经捕获了一个异常，因此记录其类型（这将是你抛出的`InvalidOperationException`实例）和消息：

[PRE109]

现在，你可以检查`whenAllCompletedTask`，通过迭代此任务的`AggregateException`来查看其`InnerExceptions`列表：

[PRE110]

运行代码，你会看到五个任务在睡眠，最终数字`0`、`2`和`4`各自抛出`InvalidOperationException`，这将由你捕获：

[PRE111]

注意`数字 0`似乎是被捕获的唯一错误（`(Message=Oh dear from `[WhenAll] number 0)`）。然而，通过记录`InnerExceptions`列表中的每个条目，你会看到`数字 0`再次出现。

你可以尝试相同的代码，但这次使用`WhenAny`。记住，`WhenAny`将在列表中的第一个任务完成时完成，所以请注意在这种情况下**错误处理**的完全缺失：

[PRE112]

除非你等待所有任务完成，否则在使用`WhenAny`时可能会错过任务引发的异常。运行此代码会导致没有捕获到任何错误，并且应用执行`3`，因为这是第一个完成的：

[PRE113]

你将通过查看C#中处理`async`结果流的一些较新选项来完成对`async`/`await`代码的审视。这提供了一种方法，可以在调用代码等待整个集合被填充并返回之前，高效地遍历集合中的项目。

注意

你可以在[https://packt.link/SuCXK](https://packt.link/SuCXK)找到此示例使用的代码。

## IAsyncEnumerable 流

如果你的应用程序针对.NET 5、.NET6、.NET Core 3.0、.NET Standard 2.1或任何后续版本，那么你可以使用`IAsyncEnumerable`流来创建可等待的代码，将`yield`关键字结合到枚举器中，以异步方式遍历对象集合。

注意

微软的文档提供了`yield`关键字的以下定义：当在迭代方法中达到`yield`返回语句时，表达式返回，并保留代码中的当前位置。下一次迭代函数被调用时，从该位置重新启动执行。

使用`yield`语句，你可以创建返回项目枚举的方法。此外，调用者不需要等待返回**整个列表**的所有项目，就可以开始遍历列表中的每个项目。相反，调用者可以在项目可用时立即访问每个项目。

在此示例中，你将创建一个控制台应用程序，该程序复制了一个保险报价系统。你将发出五个保险报价请求，再次使用`Task.Delay`来模拟接收每个报价的1秒延迟。

对于基于列表的方法，你只能在所有五个结果都返回到`Main`方法后才能记录每个引用一次。使用`IAsyncEnumerable`和`yield`关键字，引用接收之间存在相同的一秒间隔，但一旦接收到每个引用，`yield`语句就允许调用`Main`方法接收并处理引用的值。如果你希望立即开始处理项目或者可能不想在处理单个项目所需的时间之外在列表中保留数千个项目，这是理想的：

[PRE114]

首先，通过`GetInsuranceQuotesAsTask`返回一个字符串列表并遍历每个，记录每个报价的详细信息。此代码将在接收到所有报价之前等待所有报价：

[PRE115]

现在是`async`流版本。如果你将以下代码与前面的代码块进行比较，你会看到需要迭代的代码行更少。此代码不会等待接收到所有引用项，而是从`GetInsuranceQuotesAsync`接收到每个引用后立即将其写出：

[PRE116]

`GetInsuranceQuotesAsTask`方法返回一个字符串的`Task`。在五个引用之间的每个引用之间，你等待一秒钟来模拟延迟，然后将结果添加到列表中，并最终将整个列表返回给调用者：

[PRE117]

`GetInsuranceQuotesAsync`方法在每个引用之间有相同的延迟，但不是填充列表以返回给调用者，而是使用`yield`语句允许`Main`方法立即处理每个引用项：

[PRE118]

运行控制台应用程序会产生以下输出：

[PRE119]

线程 `[04]` 在应用程序启动后5秒内记录了所有五个基于任务的引用详情。在这里，它等待所有引用返回后才记录每个引用。然而，请注意，基于流的每个引用都在线程`4`和`5`之间产生了1秒的间隔后立即被记录。

两次调用所需的总时间相同（总共5秒），但当你想要一有结果就立即开始处理时，`yield`更可取。这在UI应用程序中非常有用，你可以向用户提供早期结果。

注意

你可以在[https://packt.link/KarKW](https://packt.link/KarKW)找到此示例使用的代码。

## 并行编程

到目前为止，本章已经介绍了使用`Task`类和`async`/`await`关键字进行异步编程。你已经看到了如何定义任务和`async`代码块，以及随着这些结构的完成，程序流程可以精细控制。

并行框架（PFX）提供了进一步利用多核处理器以高效运行并发操作的方法。术语TPL（任务并行库）通常用来指代C#中的`Parallel`类。

使用并行框架，你不需要担心创建和重用线程或协调多个任务的复杂性。框架为你管理这些，甚至调整使用的线程数量，以最大化吞吐量。

为了使并行编程有效，每个任务执行的顺序必须是无关紧要的，并且所有任务都应该相互独立，因为你不能确定何时一个任务完成，下一个任务开始。协调会抵消任何好处。并行编程可以分解为两个不同的概念：

+   数据并行

+   任务并行

### 数据并行

当你有多个数据值，并且需要将这些值中的每个值都并发应用相同的操作时，就会使用数据并行。在这种情况下，对每个值的处理被分配到不同的线程中。

一个典型的例子可能是计算从1到1,000,000之间的所有质数。对于范围内的每个数字，都需要应用相同的函数来确定该值是否为质数。与其逐个迭代每个数字，不如采用异步方法，将数字分配到多个线程中。

### 任务并行

相反，当一组线程同时执行不同的操作时，例如调用不同的函数或代码段，就会使用任务并行。一个这样的例子是分析一本书中找到的单词的程序，通过下载书的文本并定义以下单独的任务：

+   计算单词数量。

+   找到最长的单词。

+   计算平均单词长度。

+   计算噪声词（例如，the、and、of）的数量。

每个这些任务都可以并发运行，并且它们之间互不依赖。

对于`Parallel`类，Parallel Framework提供了各种层，提供了并行性，包括Parallel Language Integrated Query (PLINQ)。PLINQ是一组扩展方法，它将并行编程的力量添加到LINQ语法中。这里不会详细介绍PLINQ，但会对`Parallel`类进行更详细的介绍。

Note

如果你想了解更多关于PLINQ的信息，可以参考在线文档[https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/introduction-to-plinq](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/introduction-to-plinq)。

### The Parallel Class

`Parallel`类仅包含三个`static`方法，但提供了许多重载，以提供控制并影响操作执行方式的选择。`Parallel`类中的每个方法通常在`awaitable`块（如`Task.Run`）内部调用。

值得记住的是，运行时只有在认为有理由的情况下才会并行运行所需的操作。在单个步骤比其他步骤更快完成的情况下，运行时可能会决定并行运行剩余操作的开销是不合理的。

一些常用的`Parallel`方法重载如下：

+   `public static ParallelLoopResult For(int from, int to, Action<int> body)`: 这是一个数据并行调用，通过调用`Action`委托体来执行循环，将一个`int`值传递给从到到数字范围的每个值。它返回`ParallelLoopResult`，其中包含循环完成后的详细信息。

+   `public static ParallelLoopResult For(int from, int to, ParallelOptions options, Action<int, ParallelLoopState> body)`: 这是一个数据并行调用，在数字范围内执行循环。`ParallelOptions`允许配置循环选项，`ParallelLoopState`用于在运行时监控或操作循环的状态。它返回`ParallelLoopResult`。

+   `public static ParallelLoopResult ForEach<TSource>(IEnumerable<TSource> source, Action<TSource, ParallelLoopState> body)`: 一个数据并行调用，在`IEnumerable`源中的每个项目上调用`Action`委托体。它返回`ParallelLoopResult`。

+   `public static ParallelLoopResult ForEach<TSource>(Partitioner<TSource> source, Action<TSource> body)`: 一个高级数据并行调用，调用`Action`委托体，并允许你指定`Partitioner`来提供针对特定数据结构优化的分区策略，以提高性能。它返回`ParallelLoopResult`。

+   `public static void Invoke(params Action[] actions)`: 一个任务并行调用，执行传递的每个操作。

+   `public static void Invoke(ParallelOptions parallelOptions, params Action[] actions)`: 一个任务并行调用，执行每个操作，并允许指定`ParallelOptions`来配置方法调用。

The `ParallelOptions` class can be used to configure how the `Parallel` methods operate:

+   `public CancellationToken CancellationToken { get; set; }`：熟悉的取消令牌，可用于在循环中检测调用者是否请求取消。

+   `public int MaxDegreeOfParallelism { get; set; }`：一个高级设置，确定一次可以启用多少个并发任务的最大数量。

+   `public TaskScheduler? TaskScheduler { get; set; }`：一个高级设置，允许设置某种类型的任务队列调度程序。

`ParallelLoopState`可以被传递到`Action`的主体中，以便该操作可以确定或监控通过循环的流程。最常用的属性如下：

+   `public bool IsExceptional { get; }`: 如果迭代抛出了未处理的异常，则返回`true`。

+   `public bool IsStopped { get; }`: 如果迭代通过调用`Stop`方法停止了循环，则返回`true`。

+   `public void Break()`: `Action`循环可以调用此方法来指示执行应在当前迭代之后停止。

+   `public void Stop()`: 请求循环应在当前迭代处停止执行。

+   `ParallelLoopResult`，由`For`和`ForEach`方法返回，包含`Parallel`循环的完成状态。

+   `public bool IsCompleted { get; }`: 表示循环运行完成，并且在完成之前没有收到结束请求。

+   `public long? LowestBreakIteration { get; }`：如果`Break`在循环运行时被调用。这返回循环到达的最低迭代索引。

使用`Parallel`类并不意味着特定的批量操作会自动完成得更快。在任务调度中存在开销，因此在运行过短或过长的任务时应谨慎。遗憾的是，没有简单的指标可以确定这里的最佳数值。通常情况下，需要通过分析来查看是否使用`Parallel`类确实可以更快地完成操作。

注意

你可以在[https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/potential-pitfalls-in-data-and-task-parallelism](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/potential-pitfalls-in-data-and-task-parallelism)找到有关数据和任务并行性的更多信息。

### Parallel.For和Parallel.ForEach

这两个方法提供数据并行性。相同的操作应用于数据对象或数字的集合。为了从中受益，每个操作应该是CPU密集型的，也就是说，它应该需要CPU周期来执行，而不是I/O密集型（例如访问文件）。

使用这两种方法，你定义一个要应用的操作，该操作传递一个对象实例或数字来处理。在`Parallel.ForEach`的情况下，`Action`传递一个对象引用参数。`Parallel.For`传递一个数字参数。

如你在*第3章*，*委托、事件和Lambda表达式*中看到的，`Action`委托代码可以像你需要的那样简单或复杂：

[PRE120]

在这个示例中，调用 `Parallel.For` 时，你传递一个包含的 `int` 值作为起始点（`99`）和一个排他的结束值（`105`）。第三个参数是一个 lambda 表达式，`Action`，你希望对每个迭代进行调用。这个重载使用 `Action<int>`，通过 `i` 参数传递一个整数：

[PRE121]

检查 `ParallelLoopResult` 的 `IsCompleted` 属性：

[PRE122]

运行代码，你会看到它在 `104` 处停止。每个迭代由一组不同的线程执行，顺序似乎有些随机，某些迭代在另一些迭代之前唤醒。你使用了相对较短的时间延迟（使用 `Thread.Sleep`），因此并行任务调度器可能需要额外几毫秒来激活每个迭代。这就是为什么迭代执行的顺序应该相互独立：

[PRE123]

使用 `ParallelLoopState` 重载，你可以通过 `Action` 代码控制迭代。在下面的示例中，代码检查它是否在迭代编号 `15`：

[PRE124]

在 `loopState` 上调用 `Break` 传达了 `Parallel` 循环应尽快停止进一步迭代的意图：

[PRE125]

从结果中，你可以看到在实际上停止之前，你到达了项目 `17`，尽管在迭代 `15` 时请求中断，如下面的片段所示：

[PRE126]

代码使用了 `ParallelLoopState.Break`；这表明循环 `17` 尽管在迭代 `15` 时请求停止。这通常发生在运行时已经开始后续迭代，然后刚刚检测到一个 `Break` 请求。这些是停止请求；运行时可能在停止之前运行额外的迭代。

或者，可以使用 `ParallelLoopState.Stop` 方法实现更突然的停止。一个替代的 `Parallel.For` 重载允许将状态传递到每个循环，并返回一个单个聚合值。

为了更好地了解这些重载，你将在下一个示例中计算 `pi` 的值。这是一个非常适合 `Parallel.For` 的任务，因为它意味着重复计算一个值，该值在传递给下一个迭代之前被聚合。迭代次数越高，最终数值越精确。

注意

你可以在 [https://www.mathscareers.org.uk/article/calculating-pi/](https://www.mathscareers.org.uk/article/calculating-pi/) 上找到有关公式的更多信息。

你使用循环提示用户输入系列数（要显示的小数位数）作为百万的倍数（以节省输入许多零）：

[PRE127]

尝试解析输入：

[PRE128]

将输入的值乘以一百万，并将其传递给可等待的 `CalcPi` 函数（稍后将定义）：

[PRE129]

你最终会收到 `pi` 的值，因此使用字符串插值功能将 `pi` 写入 `18` 位小数，使用 `:N18` 数值格式样式：

[PRE130]

重复循环，直到输入 `0`：

[PRE131]

现在是 `CalcPi` 函数。你知道 `Parallel` 方法都会阻塞调用线程，所以你需要使用 `Task.Run`，它最终将返回最终计算出的值。

将简要介绍线程同步的概念。当使用多个线程和共享变量时，存在一种危险，即一个线程可能从内存中读取一个值，并在另一个线程尝试执行相同操作的同时尝试写入新值，该线程使用自己的值和它认为的正确当前值，而此时它可能已经读取了一个已经过时的共享值。

为了防止此类问题，可以使用互斥锁，以便在给定线程持有锁时执行其语句，并在完成后释放该锁。所有其他线程都被阻止获取锁，并被迫等待直到锁被释放。

这可以通过使用 `lock` 语句来实现。当使用 `lock` 语句实现线程同步时，所有复杂性都由运行时处理。`lock` 语句具有以下形式：

[PRE132]

概念上，你可以将 `lock` 语句想象成一个狭窄的通道，足够容纳一个人一次通过。无论一个人通过通道需要多长时间以及他们在那里做什么，其他人必须等待，直到持有钥匙的人离开（释放锁）才能通过通道。

返回到 `CalcPi` 函数：

[PRE133]

`gate` 变量是 `object` 类型，并在 lambda 表达式中与 `lock` 语句一起使用，以保护 `sum` 变量免受不安全访问：

[PRE134]

这里事情变得稍微复杂一些，因为你使用了 `Parallel.For` 重载，它还允许你传递额外的参数和委托：

+   `fromInclusive`：起始索引（在本例中为 `0`）。

+   `toExclusive`：结束索引（步数）。

+   `localInit`：一个 `Func` 委托，返回每个迭代的局部数据的 **初始状态**。

+   `body`：实际计算 Pi 值的 `Func` 委托。

+   `localFinal`：一个 `Func` 委托，用于对每个迭代的局部状态执行最终操作。

[PRE135]

在这里，你现在使用 `lock` 语句来确保一次只有一个线程可以递增 `sum` 的值，并使用其正确的值：

[PRE136]

通过使用 `lock(obj)` 语句，你已经提供了一种最低级别的线程安全性，运行程序会产生以下输出：

[PRE137]

`Parallel.ForEach` 遵循类似的语义；而不是将数字范围传递给 `Action` 委托，你传递一个要处理的对象集合。

注意

你可以在 [https://packt.link/1yZu2](https://packt.link/1yZu2) 找到用于此示例的代码。

以下示例显示了使用 `ParallelOptions` 和取消令牌的 `Parallel.ForEach`。在这个例子中，你有一个控制台应用程序，它创建了 10 个客户。每个客户都有一个包含所有已下订单值的列表。你想要模拟一个按需获取客户订单的慢速运行服务。每当任何代码访问 `Customer.Orders` 属性时，列表只填充一次。在这里，你将为每个客户实例使用另一个 `lock` 语句来确保列表安全地填充。

`Aggregator` 类将遍历客户列表，并使用 `Parallel.ForEach` 调用来计算总订单成本和平均订单成本。允许用户输入他们愿意等待所有聚合操作完成的最大时间周期，然后显示前五名客户。

首先，创建一个 `Customer` 类，其构造函数接收一个 `name` 参数：

[PRE138]

你希望按需填充 `Orders` 列表，并且每个客户只填充一次，因此使用另一个 `lock` 示例来确保订单列表安全地只填充一次。你只需使用 `Orders` 的 `get` 访问器来检查 `_orders` 变量的空引用，然后使用 `Enumerable.Range` LINQ 方法创建一个随机数量的订单值来生成一个数字范围。

注意，你还可以通过添加 `Thread.Sleep` 来模拟慢速请求，这将阻塞第一次访问此客户订单的线程（由于你使用了 `Parallel` 类，这将是一个后台线程而不是主线程）：

[PRE139]

你的 `Aggregator` 类将计算以下 `Total` 和 `Average` 属性：

[PRE140]

观察一下 `Aggregator` 类，注意它的 `Aggregate` 方法接收一个要处理的客户列表和 `CancellationToken`，该令牌将根据控制台用户的偏好时间周期自动发出取消请求。该方法返回一个基于 `bool` 的 `Task`。结果将指示操作是否在处理客户过程中被取消：

[PRE141]

主 `Parallel.ForEach` 方法通过创建一个 `ParallelOptions` 类并传入取消令牌来配置。当由 `Parallel` 类调用时，`Action` 委托传递一个 `Customer` 实例（`customer =>`），该实例仅简单地对订单值求和并计算平均值，然后将平均值分配给客户的属性。

注意 `Parallel.ForEach` 调用被包裹在一个 `try-catch` 块中，该块捕获任何类型的 `OperationCanceledException` 异常。如果超过最大时间周期，则运行时会抛出异常以停止处理。你必须捕获此异常；否则，应用程序将因未处理的异常错误而崩溃：

[PRE142]

主控制台应用程序提示输入最大等待时间，`maxWait`：

[PRE143]

创建 `100` 个客户，这些客户可以传递给聚合器：

[PRE144]

创建`CancellationTokenSource`实例，传入最大等待时间。如您之前所见，如果超过时间限制，使用此令牌的任何代码都将被取消异常中断：

[PRE145]

任务完成后，您只需取出按总订单排序的前五个客户。使用`PadRight`方法对输出中的客户姓名进行对齐：

[PRE146]

使用`1`秒的短时间运行控制台应用程序会产生以下输出：

[PRE147]

使用线程`01`创建`10`个客户的操作是同步进行的。

注意

Visual Studio在您第一次运行程序时可能会显示以下警告：“非空字段`_orders`在退出构造函数时必须包含一个非空值。考虑将字段声明为可空。”这是检查代码以检查空引用可能性的建议。

`Aggregator`随后开始处理每个客户。注意如何使用不同的线程，并且处理并不从第一个客户开始。这是任务调度器决定队列中下一个任务的过程。在令牌引发取消异常之前，您只成功处理了八个客户。

注意

您可以在[https://packt.link/1LDxI](https://packt.link/1LDxI)找到此示例使用的代码。

您已经查看了一些`Parallel`类中可用的功能。您可以看到它提供了一个简单而有效的方法来跨多个任务或数据片段运行代码。

术语`Parallel`类是此类的一个例子，并且可以是一个非常有用的工具。

下一节将把这些并发概念引入到一个使用多个任务生成一系列图像的活动。由于每个图像的创建可能需要几秒钟，因此您需要提供一个让用户选择取消任何剩余任务的方法。

## 活动5.01：从斐波那契数列创建图像

在*练习5.01*中，您查看了一个创建称为斐波那契数的值的递归函数。这些数字可以组合成所谓的斐波那契数列，并用于创建有趣的螺旋形状的图像。

对于这个活动，您需要创建一个控制台应用程序，允许将各种输入传递给序列计算器。一旦用户输入了他们的参数，应用程序将开始创建1,000个图像的耗时任务。

序列中的每个图像可能需要几秒钟来计算和创建，因此您需要提供一个方法，使用`TaskCancellationSource`在操作中途取消操作。如果用户取消任务，他们仍然可以访问取消请求之前的图像。本质上，您允许用户尝试不同的参数，看看这对输出图像有什么影响。

![图5.2：斐波那契数列图像文件](img/B16835_05_02.jpg)

图5.2：斐波那契数列图像文件

如果您更喜欢`async`/`await`任务，这是一个理想的`Parallel`类示例。以下是需要从用户那里获取的输入：

+   输入`phi`的值（`1.0`到`6.0`之间的值提供理想的图像）。

+   输入要创建的图像数量（建议每个周期`1,000`个）。

+   输入可选的每图像点数（建议默认为`3,000`）。

+   输入可选的图像大小（默认为`800`像素）。

+   输入可选的点大小（默认为`5`）。

+   接下来，输入可选的文件格式（默认为`.png`格式）。

+   控制台应用程序应使用循环来提示用户输入前一个参数，并在为前一个参数创建图像的同时允许用户输入新的标准。

+   如果用户在创建前一组图像时按下`Enter`键，则应取消该任务。

+   按下`x`键应关闭应用程序。

由于此活动旨在测试您的异步技能，而不是数学或图像处理，您有以下类来帮助进行计算和图像创建：

+   这里定义的`Fibonacci`类计算连续序列项的`X`和`Y`坐标。对于每个图像循环，返回一个`Fibonacci`类的列表。

+   通过调用`CreateSeed`创建第一个元素。列表的其余部分应使用`CreateNext`，传入前一个项：

    [PRE148]

[PRE149]

+   使用以下`FibonacciSequence`.`Calculate`方法创建一个Fibonacci项的列表。这将传递要绘制的点的数量和`phi`的值（两者均由用户指定）：

    [PRE150]

[PRE151]

+   使用`dotnet add package`命令导出生成的数据到`.png`格式图像文件，以添加对`System.Drawing.Common`命名空间的引用。在您的项目源文件夹中运行以下命令：

    [PRE152]

+   此图像创建类`ImageGenerator`可用于创建每个最终图像文件：

    [PRE153]

[PRE154]

要完成此活动，请执行以下步骤：

1.  创建一个新的控制台应用程序项目。

1.  生成的图像应保存在系统`Temp`文件夹内的一个文件夹中，因此请使用`Path.GetTempPath()`获取`Temp`路径，并使用`Directory.CreateDirectory`创建一个名为`Fibonacci`的子文件夹。

1.  声明一个`do`循环，重复以下*步骤4*到*步骤7*。

1.  提示用户输入`phi`的值（这通常在`1.0`到`6.00`之间）。您需要将用户输入读取为字符串，并使用`double.TryParse`尝试将输入转换为有效的双精度浮点变量。

1.  接下来，提示用户输入要创建的图像文件数量（`1,000`是一个可接受的示例值）。将解析后的输入存储在名为`imageCount`的`int`变量中。

1.  如果输入的任一值是空的，这表明用户仅按下了`Enter`键，因此退出`do`循环。理想情况下，也可以定义并使用`CancellationTokenSource`来取消任何挂起的计算。

1.  `phi`和`imageCount`的值应传递给名为`GenerateImageSequences`的新方法，该方法返回一个`Task`。

1.  `GenerateImageSequences`方法需要使用一个循环，该循环为请求的每个图像计数迭代。每次迭代应增加`phi`，然后是一个常数值（建议为`0.015`），在等待调用`Task.Run`方法之前，该方法调用`FibonacciSequence.Calculate`，传入`phi`和用于点的常数值（例如`3,000`是一个可接受的示例值）。这将返回一个斐波那契项列表。

1.  然后`GenerateImageSequences`应将生成的斐波那契列表传递给图像创建者`ImageGenerator.ExportSequence`，使用`Task.Run`调用等待。对于`ExportSequence`的调用，建议的常数值为图像大小`800`和点大小`5`。

1.  运行控制台应用程序应该产生以下控制台输出：

    [PRE155]

你会发现系统`Temp`文件夹中的斐波那契文件夹中生成了各种图像文件：

![图5.3：Windows 10资源管理器图像文件夹内容（生成的图像子集）](img/B16835_05_03.jpg)

图5.3：Windows 10资源管理器图像文件夹内容（生成的图像子集）

通过完成这个活动，你看到了如何启动多个长时间运行的操作，然后协调它们以产生单个结果，每个步骤都在隔离中运行，允许其他操作按需继续。

注意

该活动的解决方案可以在[https://packt.link/qclbF](https://packt.link/qclbF)找到。

# 摘要

在本章中，你考虑了并发提供的部分强大和灵活的功能。你首先通过将目标操作传递到你创建的任务来开始，然后查看静态的`Task`工厂辅助方法。通过使用延续任务，你看到单个任务和任务集合可以被协调以执行聚合操作。

接下来，你研究了`async`/`await`关键字，这些关键字可以帮助你编写更简单、更简洁的代码，希望这样更容易维护。

本章探讨了C#如何以相对简单的方式提供并发模式，这使得可以利用多核处理器的强大功能。这对于卸载耗时计算非常有用，但这也带来了一定的代价。你看到了如何使用`lock`语句来安全地防止多个线程同时读取或写入一个值。

在下一章中，你将了解如何使用Entity Framework和SQL Server在C#应用程序中与关系型数据交互。本章是关于数据库操作的内容。如果你对数据库结构不熟悉或想复习PostgreSQL的基本知识，请参阅本书GitHub仓库中提供的附加章节。
