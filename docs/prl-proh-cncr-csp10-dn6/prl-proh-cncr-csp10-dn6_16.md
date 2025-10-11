# 第十三章：评估

本节包含所有章节的问题答案。

# *第一章*，托管线程概念

1.  托管线程是在 .NET 托管代码中使用 `System.Threading.Thread` 对象创建的线程。

1.  在调用 `Thread.Start()` 之前将 `Thread.IsBackground` 属性设置为 `true`。

1.  .NET 将抛出 `ThreadStateException` 异常。

1.  .NET 主要根据线程的 `Thread.Priority` 值来优先处理托管线程。

1.  `ThreadPriority.Highest`。

1.  `.NET 6` 不支持 `Thread.Abort()`。代码将无法编译。

1.  将对象参数添加到新线程启动的方法中，并在调用 `Thread.Start(data)` 时传递数据。

1.  将委托传递给取消令牌的 `Register` 方法。

# *第二章*，.NET 中多线程编程的演变

1.  `ThreadPool `

1.  C# 5.0

1.  .NET Framework 4.5

1.  .NET Core 3.0

1.  `Task`、`Task<T>`、`ValueTask` 或 `ValueTask<T>`

1.  `ConcurrentDictionary<TKey, TValue>`

1.  `BlockingCollection<T>`

1.  **并行 LINQ** (**PLINQ**)

# *第三章*，托管线程的最佳实践

1.  单例。

1.  `ThreadStatic`。

1.  当多个线程都在等待访问一个被锁定的资源且无法继续时，会发生死锁。

1.  `Monitor.TryEnter`。

1.  `Interlocked`。

1.  `Interlocked.Add`。

1.  `MaxDegreeOfParallelism`。

1.  使用 `WithDegreeOfParallelism` 扩展方法。

1.  `ThreadPool.GetMinThreads()`。

# *第四章*，用户界面响应性和线程

1.  `Task` 或 `Task<T>`。

1.  `Task.WhenAll`。

1.  `Task.Factory.StartNew`。

1.  `ThreadPool` 的后台线程。

1.  `Application.Current.Dispatcher.Invoke`。

1.  `this.BeginInvoke`。

1.  检查 `this.InvokeRequired` 属性。

# *第五章*，使用 C# 进行异步编程

1.  `Task.Result`。

1.  `Task.WhenAll()`。

1.  `Task.WaitAll()`。

1.  `Task`、`Task<TResult>`、`ValueTask` 或 `ValueTask<TResult>`。

1.  I/O 密集型操作，如文件或网络访问，最适合异步方法。

1.  错误。始终以 `Async` 后缀异步方法是一种最佳实践。

1.  `Task.Run`。

# *第六章*，并行编程概念

1.  `Parallel.For`。

1.  `Parallel.ForEachAsync`。

1.  `Parallel.Invoke`。

1.  `TaskCreationOptions.AttachToParent`。

1.  `TaskCreationOptions.DenyAttach`。

1.  `Task.Run` 将始终拒绝子任务附加。此外，`Task.Run` 没有重载方法来提供 `TaskCreationOptions`。

1.  不，如果每个循环迭代运行得快，并且/或者循环迭代次数很少，则常规 `for` 和 `foreach` 循环可能会更快。

# *第七章*，任务并行库 (TPL) 和数据流

以下是该章节问题的答案：

1.  `JoinBlock`。

1.  `BufferBlock` 是一个传播块。

1.  `BufferBlock`。

1.  `JoinTo()`。

1.  `Complete()`。

1.  `SendAsync()`。

1.  `ReceiveAsync()`。

# *第八章*，并行数据结构和并行 LINQ

1.  `AsParallel()`.

1.  `AsSequential()`.

1.  `AsOrdered()`.

1.  `ForAll()`.

1.  `AsOrdered()` 可以显著降低查询的性能。

1.  `OrderBy` 和 `OrderByDescending`。它们将默认为 `ParallelMergeOptions.FullyBuffered`。

1.  No. PLINQ 有额外的开销，这可能导致对较小数据集或简单查询的查询速度变慢。

1.  `ParallelMergeOptions.NotBuffered`.

# *第九章*，在 .NET 中使用并发集合

1.  `BlockingCollection<T>`.

1.  `ConcurrentQueue<T>`.

1.  `BlockingCollection<T>`.

1.  `ConcurrentDictionary<TKey, TValue>`.

1.  `Enqueue()`.

1.  `TryAdd()` 和 `TryGetValue()`。

1.  No. 在使用并发集合的扩展方法时，始终添加自己的同步机制，包括标准 LINQ 操作符。

# *第十章*，使用 Visual Studio 调试多线程应用程序

1.  使用**附加到进程**窗口或在解决方案文件中设置多个启动项目。

1.  它们按进程分组。

1.  在窗口中右键单击并选择**列**。

1.  **并行堆栈**窗口。

1.  `.PNG` 文件。

1.  四.

1.  **调试位置**工具栏。

1.  点击**仅我的代码**按钮。

# *第十一章*，取消异步工作

1.  `CancellationToken.IsCancellationRequested`

1.  `CancellationTokenSource`

1.  `OperationCanceledException`

1.  注册回调

1.  `ManualResetEventSlim`

1.  `ManualResetEventSlim.Reset`

1.  `CancellationTokenSource.CreateLinkedTokenSource`

# *第十二章*，单元测试异步、并发和并行代码

1.  `Fact`

1.  `SpinLock.WaitUntil`

1.  `AggregateException`

1.  `Exception`

1.  `Assert.NotNull`

1.  **测试资源管理器 **

1.  MSTest、NUnit 和 xUnit .NET

1.  ReSharper、Rider 和 dotMemory 单独控制台运行器
