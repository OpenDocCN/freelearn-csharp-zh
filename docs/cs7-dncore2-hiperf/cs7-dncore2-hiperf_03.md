# 第三章：.NET Core 中的多线程和异步编程

多线程和异步编程是两种重要的技术，可以促进高度可扩展和高性能应用程序的开发。如果应用程序不响应，会影响用户体验并增加不满的程度。另一方面，它还会增加服务器端或应用程序运行位置的资源使用，并增加内存大小和/或 CPU 使用率。如今，硬件非常便宜，每台机器都配备了多个 CPU 核心。实现多线程和使用异步编程技术不仅可以提高应用程序的性能，还可以使应用程序具有更高的响应性。

本章将探讨多线程和异步编程模型的核心概念，以帮助您在项目中使用它们，并提高应用程序的整体性能。

以下是本章将学习的主题列表：

+   多线程与异步编程

+   .NET Core 中的多线程

+   .NET Core 中的线程

+   线程同步

+   任务并行库（TPL）

+   使用 TPL 创建任务

+   基于任务的异步模式

+   并行编程的设计模式

I/O 绑定操作是依赖于外部资源的代码。例如访问文件系统，访问网络等。

# 多线程与异步编程

如果正确实现，多线程和异步编程可以提高应用程序的性能。多线程是指同时执行多个线程以并行执行多个操作或任务的实践。通常有一个主线程和几个后台线程，通常称为工作线程，同时并行运行，同时执行多个任务，而同步和异步操作都可以在单线程或多线程环境中运行。

在单线程同步操作中，只有一个线程按照定义的顺序执行所有任务，并依次执行它们。在单线程异步操作中，只有一个线程执行任务，但它会分配一个时间片来运行每个任务。时间片结束后，它会保存该任务的状态并开始执行下一个任务。在内部，处理器在每个任务之间执行上下文切换，并分配一个时间片来运行它们。

在多线程同步操作中，有多个线程并行运行任务。与异步操作中的上下文切换不同，任务之间没有上下文切换。一个线程负责执行分配给它的任务，然后开始另一个任务，而在多线程异步操作中，多个线程运行多个任务，任务可以由单个或多个线程提供和执行。

以下图表描述了单线程和多线程同步和异步操作之间的区别：

![](img/00039.gif)

上图显示了四种操作类型。在单线程同步操作中，有一个线程按顺序运行五个任务。一旦**任务 1**完成，就执行**任务 2**，依此类推。在单线程异步操作中，有一个线程，但每个任务都会在执行下一个任务之前获得一个时间片来执行，依此类推。每个任务将被执行多次，并从暂停的地方恢复。在多线程同步操作中，有三个线程并行运行三个任务**任务 1**，**任务 2**和**任务 3**。最后，在多线程异步操作中，有三个任务**任务 1**，**任务 2**和**任务 3**由三个线程运行，但每个线程根据分配给每个任务的时间片进行一些上下文切换。

在异步编程中，并不总是每个异步操作都会在新线程上运行。`Async`/`Await`是一个没有创建额外线程的好例子。`*async*`操作在主线程的当前同步上下文中执行，并将异步操作排队在分配的时间片中执行。

# .NET Core 中的多线程

在 CPU 和/或 I/O 密集型应用程序中使用多线程有许多好处。它通常用于长时间运行的进程，这些进程具有更长或无限的生命周期，作为后台任务工作，保持主线程可用以管理或处理用户请求。然而，不必要的使用可能会完全降低应用程序的性能。有些情况下，创建太多线程并不是一个好的架构实践。

以下是一些多线程适用的示例：

+   I/O 操作

+   运行长时间的后台任务

+   数据库操作

+   通过网络进行通信

# 多线程注意事项

尽管多线程有许多好处，但在编写多线程应用程序时需要彻底解决一些注意事项。如果计算机是单核或双核计算机，并且应用程序创建了大量线程，则这些线程之间的上下文切换将减慢性能：

![](img/00040.jpeg)

上图描述了在单处理器机器上运行的程序。第一个任务是同步执行的，比在单处理器上运行的三个线程快得多。系统执行第一个线程，然后等待一段时间再执行第二个线程，依此类推。这增加了在线程之间切换的不必要开销，从而延迟了整体操作。在线程领域，这被称为上下文切换。每个线程之间的框表示在每个上下文切换之间发生的延迟。

就开发人员的经验而言，调试和测试是创建多线程应用程序时对开发人员具有挑战性的另外两个问题。

# .NET Core 中的线程

.NET 中的每个应用程序都从一个单线程开始，这是主线程。线程是操作系统用来分配处理器时间的基本单位。每个线程都有一个优先级、异常处理程序和保存在自己的线程上下文中的数据结构。如果发生异常，它是在线程的上下文中抛出的，其他线程不受其影响。线程上下文包含一些关于 CPU 寄存器、线程的主机进程的地址空间等低级信息。

如果应用程序在单处理器上运行多个线程，则每个线程将被分配一段处理器时间，并依次执行。时间片通常很小，这使得看起来好像线程在同时执行。一旦分配的时间结束，处理器就会移动到另一个线程，之前的线程等待处理器再次可用并根据分配的时间片执行。另一方面，如果线程在多个 CPU 上运行，则它们可能同时执行，但如果有其他进程和线程在运行，则时间片将被分配并相应地执行。

# 在.NET Core 中创建线程

在.NET Core 中，线程 API 与完整的.NET Framework 版本相同。可以通过创建`Thread`类对象并将`ThreadStart`或`ParameterizedThreadStart`委托作为参数来创建新线程。`ThreadStart`和`ParameterizedThreadStart`包装了在启动新线程时调用的方法。`ParameterizedThreadStart`用于包含参数的方法。

以下是一个基本示例，该示例在单独的线程上运行`ExecuteLongRunningOperation`方法：

```cs
static void Main(string[] args) 
{ 
  new Thread(new ThreadStart(ExecuteLongRunningOperation)).Start(); 
} 
static void ExecuteLongRunningOperation() 
{ 
  Thread.Sleep(100000); 
  Console.WriteLine("Operation completed successfully"); 
} 
```

在启动线程时，我们还可以传递参数并使用`ParameterizedThreadStart`委托：

```cs
static void Main(string[] args) 
{ 
  new Thread(new ParameterizedThreadStart
  (ExecuteLongRunningOperation)).Start(100000); 
} 

static void ExecuteLongRunningOperation(object milliseconds) 
{ 
  Thread.Sleep((int)milliseconds); 
  Console.WriteLine("Operation completed successfully"); 
} 
```

`ParameterizedThreadStart`委托接受一个对象作为参数。因此，如果要传递多个参数，可以通过创建自定义类并添加以下属性来实现：

```cs
public interface IService 
{ 
  string Name { get; set; } 
  void Execute(); 
} 

public class EmailService : IService 
{ 
  public string Name { get; set; } 
  public void Execute() => throw new NotImplementedException(); 

  public EmailService(string name) 
  { 
    this.Name = name; 
  } 
} 

static void Main(string[] args) 
{ 
  IService service = new EmailService("Email"); 
  new Thread(new ParameterizedThreadStart
  (RunBackgroundService)).Start(service); 
} 

static void RunBackgroundService(Object service) 
{ 
  ((IService)service).Execute(); //Long running task 
} 
```

每个线程都有一个线程优先级。当线程被创建时，其优先级被设置为正常。优先级影响线程的执行。优先级越高，线程将被赋予的优先级就越高。线程优先级可以在线程对象上定义，如下所示：

```cs
static void RunBackgroundService(Object service) 
{ 
  Thread.CurrentThread.Priority = ThreadPriority.Highest;      
  ((IService)service).Execute(); //Long running task
}
```

`RunBackgroundService`是在单独的线程中执行的方法，可以使用`ThreadPriority`枚举设置优先级，并通过调用`Thread.CurrentThread`引用当前线程对象，如上面的代码片段所示。

# 线程生命周期

线程的生命周期取决于在该线程中执行的方法。一旦方法执行完毕，CLR 将释放线程占用的内存并进行处理。另一方面，也可以通过调用`Interrupt`或`Abort`方法显式地处理线程。

另一个非常重要的因素是异常。如果异常在线程内部没有得到适当处理，它们将传播到`调用`方法，依此类推，直到它们到达调用堆栈中的`根`方法。当它达到这一点时，如果没有得到处理，CLR 将关闭线程。

对于持续或长时间运行的线程，关闭过程应该被正确定义。平滑关闭线程的最佳方法之一是使用`volatile bool`变量：

```cs
class Program 
{ 

  static volatile bool isActive = true;  
  static void Main(string[] args) 
  { 
    new Thread(new ParameterizedThreadStart
    (ExecuteLongRunningOperation)).Start(1000); 
  } 

  static void ExecuteLongRunningOperation(object milliseconds) 
  { 
    while (isActive) 
    { 
      //Do some other operation 
      Console.WriteLine("Operation completed successfully"); 
    } 
  } 
} 
```

在上面的代码中，我们使用了`volatile bool`变量`isActive`，它决定了`while`循环是否执行。

`volatile`关键字表示一个字段可能会被多个同时执行的线程修改。声明为 volatile 的字段不受编译器优化的影响，假设只有一个线程访问。这确保了字段中始终存在最新的值。要了解更多关于 volatile 的信息，请参考以下 URL：

[`docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/volatile`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/volatile)

# .NET 中的线程池

CLR 提供了一个单独的线程池，其中包含要用于异步执行任务的线程列表。每个进程都有自己特定的线程池。CLR 向线程池中添加和移除线程。

使用`ThreadPool`来运行线程，我们可以使用`ThreadPool.QueueUserWorkItem`，如下面的代码所示：

```cs
class Program 
{ 
  static void Main(string[] args) 
  { 
    ThreadPool.QueueUserWorkItem(ExecuteLongRunningOperation, 1000); 
    Console.Read(); 
  } 
  static void ExecuteLongRunningOperation(object milliseconds) 
  { 

    Thread.Sleep((int)milliseconds); 
    Console.WriteLine("Thread is executed"); 
  } 
} 
```

`QueueUserWorkItem`将任务排队，由 CLR 在线程池中可用的线程中执行。任务队列按照**先进先出**（**FIFO**）的顺序进行维护。但是，根据线程的可用性和任务本身，任务完成可能会延迟。

# 线程同步

在多线程应用程序中，我们有共享资源，可以被多个线程同时访问。资源在多个线程之间共享的区域称为临界区。为了保护这些资源并提供线程安全的访问，有一些技术将在本节中讨论。

让我们举一个例子，我们有一个用于将消息记录到文件系统的单例类。单例，根据定义，表示应该只有一个实例在多次调用之间共享。以下是一个基本的单例模式实现，它不是线程安全的：

```cs
public class Logger 
{ 
  static Logger _instance; 

  private Logger() { } 

  public Logger GetInstance() 
  { 
    _instance = (_instance == null ? new Logger() : _instance); 
    return _instance; 
  } 

  public void LogMessage(string str) 
  { 
    //Log message into file system 
  } 

} 
```

上面的代码是一个懒惰初始化的单例模式，它在第一次调用`GetInstance`方法时创建一个实例。`GetInstance`是临界区，不是线程安全的。如果多个线程进入临界区，将创建多个实例，并发条件将发生。

竞争条件是多线程编程中出现的问题，当结果取决于事件的时间时。当两个或多个并行任务访问共享对象时，就会出现竞争条件。

要实现线程安全的单例，我们可以使用锁定模式。锁定确保只有一个线程可以进入临界区，如果另一个线程尝试进入，它将等待直到线程被释放。以下是一个修改后的版本，使单例线程安全：

```cs
public class Logger 
{ 

  private static object syncRoot = new object(); 
  static Logger _instance; 

  private Logger() { } 

  public Logger GetInstance() 
  { 
    if (_instance == null) 
    { 
      lock (syncRoot) 
      { 
        if (_instance == null) 
        _instance = new Logger(); 
      } 
    } 
    return _instance; 
  } 

  public void LogMessage(string str) 
  { 
    //Log message into file system 
  } 
} 
```

# 监视器

监视器用于提供对资源的线程安全访问。它适用于多线程编程，在那里有多个线程需要同时访问资源。当多个线程尝试进入`monitor`以访问任何资源时，CLR 只允许一个线程一次进入，其他线程被阻塞。当线程退出监视器时，下一个等待的线程进入，依此类推。

如果我们查看`Monitor`类，所有方法如`Monitor.Enter`和`Monitor.Exit`都是在对象引用上操作的。与`lock`类似，`Monitor`也提供对资源的门控访问；但是，开发人员在 API 方面会有更大的控制。

以下是在.NET Core 中使用`Monitor`的基本示例：

```cs
public class Job 
{ 

  int _jobDone; 
  object _lock = new object(); 

  public void IncrementJobCounter(int number) 
  { 
    Monitor.Enter(_lock); 
    // access to this field is synchronous
    _jobDone += number; 
    Monitor.Exit(_lock); 
  } 

} 
IncrementJobCounter method to increment the _jobDone counter.
```

在某些情况下，关键部分必须等待资源可用。一旦它们可用，我们希望激活等待块以执行。

为了帮助我们理解，让我们举一个运行`Job`的例子，其任务是运行多个线程添加的作业。如果没有作业存在，它应该等待线程推送并立即开始执行它们。

```cs
JobExecutor: 
```

```cs
public class JobExecutor 
{ 
  const int _waitTimeInMillis = 10 * 60 * 1000; 
  private ArrayList _jobs = null; 
  private static JobExecutor _instance = null; 
  private static object _syncRoot = new object(); 

  //Singleton implementation of JobExecutor
  public static JobExecutor Instance 
  { 
    get{ 
    lock (_syncRoot) 
    { 
      if (_instance == null) 
      _instance = new JobExecutor(); 
    } 
    return _instance; 
  } 
} 

private JobExecutor() 
{ 
  IsIdle = true; 
  IsAlive = true; 
  _jobs = new ArrayList(); 
} 

private Boolean IsIdle { get; set; } 
public Boolean IsAlive { get; set; } 

//Callers can use this method to add list of jobs
public void AddJobItems(List<Job> jobList) 
{ 
  //Added lock to provide synchronous access. 
  //Alternatively we can also use Monitor.Enter and Monitor.Exit
  lock (_jobs) 
  { 
    foreach (Job job in jobList) 
    { 
      _jobs.Add(job); 
    } 
    //Release the waiting thread to start executing the //jobs
    Monitor.PulseAll(_jobs); 
  } 
} 

/*Check for jobs count and if the count is 0, then wait for 10 minutes by calling Monitor.Wait. Meanwhile, if new jobs are added to the list, Monitor.PulseAll will be called that releases the waiting thread. Once the waiting is over it checks the count of jobs and if the jobs are there in the list, start executing. Otherwise, wait for the new jobs */
public void CheckandExecuteJobBatch() 
{ 
  lock (_jobs) 
  { 
    while (IsAlive) 
    { 
      if (_jobs == null || _jobs.Count <= 0) 
      { 
        IsIdle = true; 
        Console.WriteLine("Now waiting for new jobs"); 
        //Waiting for 10 minutes 
        Monitor.Wait(_jobs, _waitTimeInMillis); 
      } 
      else 
      { 
        IsIdle = false; 
        ExecuteJob(); 
      } 
    } 
  } 
} 

//Execute the job
private void ExecuteJob() 
{ 
  for(int i=0;i< _jobs.Count;i++) 
  { 
    Job job = (Job)_jobs[i]; 
    //Execute the job; 
    job.DoSomething(); 
    //Remove the Job from the Jobs list 
    _jobs.Remove(job); 
    i--; 
  } 
} 
} 
```

这是一个单例类，其他线程可以使用静态的`Instance`属性访问`JobExecutor`实例，并调用`AddJobsItems`方法将要执行的作业列表添加到其中。`CheckandExecuteJobBatch`方法持续运行并每 10 分钟检查列表中的新作业。或者，如果通过调用`Monitor.PulseAll`方法中断了`AddJobsItems`方法，它将立即转移到`while`语句并检查项目计数。如果项目存在，`CheckandExecuteJobBatch`方法调用`ExecuteJob`方法来运行该作业。

```cs
Job class containing two properties, namely JobID and JobName, and the DoSomething method that will print the JobID on the console:
```

```cs
public class Job 
{ 
  // Properties to set and get Job ID and Name
  public int JobID { get; set; } 
  public string JobName { get; set; } 

  //Do some task based on Job ID as set through the JobID        
  //property
  public void DoSomething() 
  { 
    //Do some task based on Job ID  
    Console.WriteLine("Executed job " + JobID);  
  } 
} 
```

最后，在主`Program`类上，我们可以调用三个工作线程和一个`JobExecutor`线程，如下所示：

```cs
class Program 
{ 
  static void Main(string[] args) 
  { 
    Thread jobThread = new Thread(new ThreadStart(ExecuteJobExecutor)); 
    jobThread.Start(); 

    //Starting three Threads add jobs time to time; 
    Thread thread1 = new Thread(new ThreadStart(ExecuteThread1)); 
    Thread thread2 = new Thread(new ThreadStart(ExecuteThread2)); 
    Thread thread3 = new Thread(new ThreadStart(ExecuteThread3)); 
    Thread1.Start(); 
    Thread2.Start(); 
    thread3.Start(); 

    Console.Read(); 
  } 

  //Implementation of ExecuteThread 1 that is adding three 
  //jobs in the list and calling AddJobItems of a singleton 
  //JobExecutor instance
  private static void ExecuteThread1() 
  { 
    Thread.Sleep(5000); 
    List<Job> jobs = new List<Job>(); 
    jobs.Add(new Job() { JobID = 11, JobName = "Thread 1 Job 1" }); 
    jobs.Add(new Job() { JobID = 12, JobName = "Thread 1 Job 2" }); 
    jobs.Add(new Job() { JobID = 13, JobName = "Thread 1 Job 3" }); 
    JobExecutor.Instance.AddJobItems(jobs); 
  } 

  //Implementation of ExecuteThread2 method that is also adding 
  //three jobs and calling AddJobItems method of singleton 
  //JobExecutor instance 
  private static void ExecuteThread2() 
  { 
    Thread.Sleep(5000); 
    List<Job> jobs = new List<Job>(); 
    jobs.Add(new Job() { JobID = 21, JobName = "Thread 2 Job 1" }); 
    jobs.Add(new Job() { JobID = 22, JobName = "Thread 2 Job 2" }); 
    jobs.Add(new Job() { JobID = 23, JobName = "Thread 2 Job 3" }); 
    JobExecutor.Instance.AddJobItems(jobs); 
  } 

  //Implementation of ExecuteThread3 method that is again 
  // adding 3 jobs instances into the list and 
  //calling AddJobItems to add those items into the list to execute
  private static void ExecuteThread3() 
  { 
    Thread.Sleep(5000); 
    List<Job> jobs = new List<Job>(); 
    jobs.Add(new Job() { JobID = 31, JobName = "Thread 3 Job 1" }); 
    jobs.Add(new Job() { JobID = 32, JobName = "Thread 3 Job 2" }); 
    jobs.Add(new Job() { JobID = 33, JobName = "Thread 3 Job 3" }); 
    JobExecutor.Instance.AddJobItems(jobs); 
  } 

  //Implementation of ExecuteJobExecutor that calls the 
  //CheckAndExecuteJobBatch to run the jobs
  public static void ExecuteJobExecutor() 
  { 
    JobExecutor.Instance.IsAlive = true; 
    JobExecutor.Instance.CheckandExecuteJobBatch(); 
  } 
} 
```

以下是运行此代码的输出：

![](img/00041.gif)

# 任务并行库（TPL）

到目前为止，我们已经学习了一些关于多线程的核心概念，并使用线程执行多个任务。与.NET 中的经典线程模型相比，TPL 最小化了使用线程的复杂性，并通过一组 API 提供了抽象，帮助开发人员更多地专注于应用程序程序，而不是专注于如何提供线程以及其他事项。

使用 TPL 而不是线程有几个好处：

+   它将并发自动扩展到多核级别

+   它将 LINQ 查询自动扩展到多核级别

+   它处理工作的分区并在需要时使用`ThreadPool`

+   它易于使用，并减少了直接使用线程的复杂性

# 使用 TPL 创建任务

TPL API 可在`System.Threading`和`System.Threading.Tasks`命名空间中使用。它们围绕任务工作，任务是异步运行的程序或代码块。可以通过调用`Task.Run`或`TaskFactory.StartNew`方法来运行异步任务。当我们创建一个任务时，我们提供一个命名委托、匿名方法或 lambda 表达式，任务执行它。

```cs
ExecuteLongRunningTasksmethod using Task.Run:
```

```cs
class Program 
{ 
  static void Main(string[] args) 
  { 
    Task t = Task.Run(()=>ExecuteLongRunningTask(5000)); 
    t.Wait(); 
  } 

  public static void ExecuteLongRunningTask(int millis) 
  { 
    Thread.Sleep(millis); 
    Console.WriteLine("Hello World"); 

  } 
} 
ExecuteLongRunningTask method asynchronously using the Task.Run method. The Task.Run method returns the Task object that can be used to further wait for the asynchronous piece of code to be executed completely before the program ends. To wait for the task, we have used the Wait method.
```

或者，我们也可以使用`Task.Factory.StartNew`方法，这是更高级的并提供更多选项。在调用`Task.Factory.StartNew`方法时，我们可以指定`CancellationToken`、`TaskCreationOptions`和`TaskScheduler`来设置状态、指定其他选项和安排任务。

TPL 默认使用 CPU 的多个核心。当使用 TPL API 执行任务时，它会自动将任务分割成一个或多个线程，并利用多个处理器（如果可用）。创建多少个线程的决定是由 CLR 在运行时计算的。而线程只有一个处理器的亲和性，要在多个处理器上运行任何任务需要适当的手动实现。

# 基于任务的异步模式（TAP）

在开发任何软件时，总是要在设计其架构时实现最佳实践。基于任务的异步模式是在使用 TPL 时可以使用的推荐模式之一。然而，在实现 TAP 时有一些需要牢记的事情。

# 命名约定

异步执行的方法应该以`Async`作为命名后缀。例如，如果方法名以`ExecuteLongRunningOperation`开头，它应该有后缀`Async`，结果名称为`ExecuteLongRunningOperationAsync`。

# 返回类型

方法签名应该返回`System.Threading.Tasks.Task`或`System.Threading.Tasks.Task<TResult>`。任务的返回类型等同于返回`void`的方法，而`TResult`是数据类型。

# 参数

`out`和`ref`参数不允许作为方法签名中的参数。如果需要返回多个值，可以使用元组或自定义数据结构。方法应该始终返回`Task`或`Task<TResult>`，如前面所讨论的。

以下是同步和异步方法的一些签名：

| **同步方法** | **异步方法** |
| --- | --- |
| `Void Execute();` | `Task ExecuteAsync();` |
| `List<string> GetCountries();` | `Task<List<string>> GetCountriesAsync();` |
| `Tuple<int, string> GetState(int stateID);` | `Task<Tuple<int, string>> GetStateAsync(int stateID);` |
| `Person GetPerson(int personID);` | `Task<Person> GetPersonAsync(int personID);` |

# 异常

异步方法应该总是抛出分配给返回任务的异常。然而，使用错误，比如将空参数传递给异步方法，应该得到适当处理。

假设我们想根据预定义的模板列表动态生成多个文档，其中每个模板都使用动态值填充占位符并将其写入文件系统。我们假设这个操作将花费足够长的时间来为每个模板生成一个文档。下面是一个代码片段，显示了如何处理异常：

```cs
static void Main(string[] args) 
{ 
  List<Template> templates = GetTemplates(); 
  IEnumerable<Task> asyncDocs = from template in templates select 
  GenerateDocumentAsync(template); 
  try 
  { 
    Task.WaitAll(asyncDocs.ToArray()); 

  }catch(Exception ex) 
  { 
    Console.WriteLine(ex); 
  } 
  Console.Read(); 
} 

private static async Task<int> GenerateDocumentAsync(Template template) 
{ 
  //To automate long running operation 
  Thread.Sleep(3000); 
  //Throwing exception intentionally 
  throw new Exception(); 
}
```

在上面的代码中，我们有一个`GenerateDocumentAsync`方法，执行长时间运行的操作，比如从数据库中读取模板，填充占位符，并将文档写入文件系统。为了自动化这个过程，我们使用`Thread.Sleep`来让线程休眠三秒，然后抛出一个异常，这个异常将传播到调用方法。`Main`方法循环遍历模板列表，并为每个模板调用`GenerateDocumentAsync`方法。每个`GenerateDocumentAsync`方法都返回一个任务。在调用异步方法时，异常实际上是隐藏的，直到调用`Wait`、`WaitAll`、`WhenAll`和其他方法。在上面的例子中，一旦调用`Task.WaitAll`方法，异常将被抛出，并在控制台上记录异常。

# 任务状态

任务对象提供了`TaskStatus`，用于了解任务是否正在执行方法运行，已完成方法，遇到故障，或者是否发生了其他情况。使用`Task.Run`初始化的任务最初具有`Created`状态，但当调用`Start`方法时，其状态会更改为`Running`。在应用 TAP 模式时，所有方法都返回`Task`对象，无论它们是否在方法体内使用`Task.Run`，方法体都应该被激活。这意味着状态应该是除了`Created`之外的任何状态。TAP 模式确保消费者任务已激活，并且不需要启动任务。

# 任务取消

取消对于基于 TAP 的异步方法是可选的。如果方法接受`CancellationToken`作为参数，调用方可以使用它来取消任务。但是，对于 TAP，取消应该得到适当处理。这是一个基本示例，显示了如何实现取消：

```cs
static void Main(string[] args) 
{ 
  CancellationTokenSource tokenSource = new CancellationTokenSource(); 
  CancellationToken token = tokenSource.Token; 
  Task.Factory.StartNew(() => SaveFileAsync(path, bytes, token)); 
} 

static Task<int> SaveFileAsync(string path, byte[] fileBytes, CancellationToken cancellationToken) 
{ 
  if (cancellationToken.IsCancellationRequested) 
  { 
    Console.WriteLine("Cancellation is requested..."); 
    cancellationToken.ThrowIfCancellationRequested      
  } 
  //Do some file save operation 
  File.WriteAllBytes(path, fileBytes); 
  return Task.FromResult<int>(0); 
} 
```

在前面的代码中，我们有一个`SaveFileAsync`方法，它接受`byte`数组和`CancellationToken`作为参数。在`Main`方法中，我们初始化了`CancellationTokenSource`，可以在程序后面用于取消异步操作。为了测试取消场景，我们将在`Task.Factory.StartNew`方法之后调用`tokenSource`的`Cancel`方法，操作将被取消。此外，当任务被取消时，其状态设置为`Cancelled`，`IsCompleted`属性设置为`true`。

# 任务进度报告

使用 TPL，我们可以使用`IProgress<T>`接口从异步操作中获取实时进度通知。这可以用于需要更新用户界面或控制台应用程序的异步操作的场景。在定义基于 TAP 的异步方法时，在参数中定义`IProgress<T>`是可选的。我们可以有重载的方法，可以帮助消费者在特定需要的情况下使用。但是，它们只能在异步方法支持它们的情况下使用。这是修改后的`SaveFileAsync`，用于向用户更新实际进度：

```cs
static void Main(string[] args) 
{ 
  var progressHandler = new Progress<string>(value => 
  { 
    Console.WriteLine(value); 
  }); 

  var progress = progressHandler as IProgress<string>; 

  CancellationTokenSource tokenSource = new CancellationTokenSource(); 
  CancellationToken token = tokenSource.Token; 

  Task.Factory.StartNew(() => SaveFileAsync(path, bytes, 
  token, progress)); 
  Console.Read(); 

} 
static Task<int> SaveFileAsync(string path, byte[] fileBytes, CancellationToken cancellationToken, IProgress<string> progress) 
{ 
  if (cancellationToken.IsCancellationRequested) 
  { 
    progress.Report("Cancellation is called"); 
    Console.WriteLine("Cancellation is requested..."); 
  } 

  progress.Report("Saving File"); 
  File.WriteAllBytes(path, fileBytes);   
  progress.Report("File Saved"); 
  return Task.FromResult<int>(0); 

} 
```

# 使用编译器实现 TAP

任何使用`async`关键字（对于 C＃）或`Async`（对于 Visual Basic）标记的方法都称为异步方法。`async`关键字可以应用于方法、匿名方法或 Lambda 表达式，语言编译器可以异步执行该任务。

这是使用编译器方法的 TAP 方法的简单实现：

```cs
static void Main(string[] args) 
{ 
  var t = ExecuteLongRunningOperationAsync(100000); 
  Console.WriteLine("Called ExecuteLongRunningOperationAsync method, 
  now waiting for it to complete"); 
  t.Wait(); 
  Console.Read(); 
}   

public static async Task<int> ExecuteLongRunningOperationAsync(int millis) 
{ 
  Task t = Task.Factory.StartNew(() => RunLoopAsync(millis)); 
  await t; 
  Console.WriteLine("Executed RunLoopAsync method"); 
  return 0; 
} 

public static void RunLoopAsync(int millis) 
{ 
  Console.WriteLine("Inside RunLoopAsync method"); 
  for(int i=0;i< millis; i++) 
  { 
    Debug.WriteLine($"Counter = {i}"); 
  } 
  Console.WriteLine("Exiting RunLoopAsync method"); 
} 
```

在前面的代码中，我们有`ExecuteLongRunningOperationAsync`方法，它是根据编译器方法实现的。它调用`RunLoopAsync`，该方法执行一个传递的毫秒数的循环。`ExecuteLongRunningOperationAsync`方法上的`async`关键字实际上告诉编译器该方法必须异步执行，一旦达到`await`语句，该方法返回到`Main`方法，在控制台上写一行并等待任务完成。一旦`RunLoopAsync`执行，控制权回到`await`，并开始执行`ExecuteLongRunningOperationAsync`方法中的下一个语句。

# 实现对任务的更大控制的 TAP

我们知道，TPL 以`Task`和`Task<TResult>`对象为中心。我们可以通过调用`Task.Run`方法执行异步任务，并异步执行`delegate`方法或一段代码，并在该任务上使用`Wait`或其他方法。然而，这种方法并不总是适当，有些情况下我们可能有不同的方法来执行异步操作，我们可能会使用**基于事件的异步模式**（EAP）或**异步编程模型**（APM）。为了在这里实现 TAP 原则，并以不同的模型执行异步操作，我们可以使用`TaskCompletionSource<TResult>`对象。

`TaskCompletionSource<TResult>`对象用于创建执行异步操作的任务。异步操作完成后，我们可以使用`TaskCompletionSource<TResult>`对象设置任务的结果、异常或状态。

这是一个基本示例，执行`ExecuteTask`方法返回`Task`，其中`ExecuteTask`方法使用`TaskCompletionSource<TResult>`对象将响应包装为`Task`，并通过`Task.StartNew`方法执行`ExecuteLongRunningTask`：

```cs
static void Main(string[] args) 
{ 
  var t = ExecuteTask(); 
  t.Wait(); 
  Console.Read(); 
} 

public static Task<int> ExecuteTask() 
{ 
  var tcs = new TaskCompletionSource<int>(); 
  Task<int> t1 = tcs.Task; 
  Task.Factory.StartNew(() => 
  { 
    try 
    { 
      ExecuteLongRunningTask(10000); 
      tcs.SetResult(1); 
    }catch(Exception ex) 
    { 
      tcs.SetException(ex); 
    } 
  }); 
  return tcs.Task; 

} 

public static void ExecuteLongRunningTask(int millis) 
{ 
  Thread.Sleep(millis); 
  Console.WriteLine("Executed"); 
} 
```

# 并行编程的设计模式

任务可以以各种方式设计并行运行。在本节中，我们将学习 TPL 中使用的一些顶级设计模式：

+   管道模式

+   数据流模式

+   生产者-消费者模式

+   Parallel.ForEach

+   并行 LINQ（PLINQ）

# 管道模式

管道模式通常用于需要按顺序执行异步任务的场景：

![](img/00042.jpeg)

考虑一个任务，我们需要首先创建一个用户记录，然后启动工作流并发送电子邮件。要实现这种情况，我们可以使用 TPL 的`ContinueWith`方法。以下是一个完整的示例：

```cs
static void Main(string[] args) 
{ 

  Task<int> t1 = Task.Factory.StartNew(() =>  
  { return CreateUser(); }); 

  var t2=t1.ContinueWith((antecedent) => 
  { return InitiateWorkflow(antecedent.Result); }); 
  var t3 = t2.ContinueWith((antecedant) => 
  { return SendEmail(antecedant.Result); }); 

  Console.Read(); 

} 

public static int CreateUser() 
{ 
  //Create user, passing hardcoded user ID as 1 
  Thread.Sleep(1000); 
  Console.WriteLine("User created"); 
  return 1; 
} 

public static int InitiateWorkflow(int userId) 
{ 
  //Initiate Workflow 
  Thread.Sleep(1000); 
  Console.WriteLine("Workflow initiates"); 

  return userId; 
} 

public static int SendEmail(int userId) 
{ 
  //Send email 
  Thread.Sleep(1000); 
  Console.WriteLine("Email sent"); 

  return userId; 
}  
```

# 数据流模式

数据流模式是一种具有一对多和多对一关系的通用模式。例如，以下图表表示两个任务**任务 1**和**任务 2**并行执行，第三个任务**任务 3**只有在前两个任务都完成后才会开始。一旦**任务 3**完成，**任务 4**和**任务 5**将并行执行：

![](img/00043.jpeg)

我们可以使用以下代码实现上述示例：

```cs
static void Main(string[] args) 
{ 
  //Creating two tasks t1 and t2 and starting them at the same //time
  Task<int> t1 = Task.Factory.StartNew(() => { return Task1(); }); 
  Task<int> t2 = Task.Factory.StartNew(() => { return Task2(); }); 

  //Creating task 3 and used ContinueWhenAll that runs when both the 
  //tasks T1 and T2 will be completed
  Task<int> t3 = Task.Factory.ContinueWhenAll(
  new[] { t1, t2 }, (tasks) => { return Task3(); }); 

  //Task 4 and Task 5 will be started when Task 3 will be completed. 
  //ContinueWith actually creates a continuation of executing tasks 
  //T4 and T5 asynchronously when the task T3 is completed
  Task<int> t4 = t3.ContinueWith((antecendent) => { return Task4(); }); 
  Task<int> t5 = t3.ContinueWith((antecendent) => { return Task5(); }); 
  Console.Read(); 
} 
//Implementation of Task1
public static int Task1() 
{ 
  Thread.Sleep(1000); 
  Console.WriteLine("Task 1 is executed"); 
  return 1; 
} 

//Implementation of Task2 
public static int Task2() 
{ 
  Thread.Sleep(1000); 
  Console.WriteLine("Task 2 is executed"); 
  return 1; 
} 
//Implementation of Task3 
public static int Task3() 
{ 
  Thread.Sleep(1000); 
  Console.WriteLine("Task 3 is executed"); 
  return 1; 
} 
Implementation of Task4
public static int Task4() 
{ 
  Thread.Sleep(1000); 
  Console.WriteLine("Task 4 is executed"); 
  return 1; 
} 

//Implementation of Task5
public static int Task5() 
{ 
  Thread.Sleep(1000); 
  Console.WriteLine("Task 5 is executed"); 
  return 1; 
} 
```

# 生产者/消费者模式

执行长时间运行操作的最佳模式之一是生产者/消费者模式。在这种模式中，有生产者和消费者，一个或多个生产者通过共享的数据结构`BlockingCollection`连接到一个或多个消费者。`BlockingCollection`是并行编程中使用的固定大小的集合。如果集合已满，生产者将被阻塞，如果集合为空，则不应再添加更多的消费者：

![](img/00044.jpeg)

在现实世界的例子中，生产者可以是从数据库中读取图像的组件，消费者可以是处理该图像并将其保存到文件系统的组件：

```cs
static void Main(string[] args) 
{ 
  int maxColl = 10; 
  var blockingCollection = new BlockingCollection<int>(maxColl); 
  var taskFactory = new TaskFactory(TaskCreationOptions.LongRunning, 
  TaskContinuationOptions.None); 

  Task producer = taskFactory.StartNew(() => 
  { 
    if (blockingCollection.Count <= maxColl) 
    { 
      int imageID = ReadImageFromDB(); 
      blockingCollection.Add(imageID); 
      blockingCollection.CompleteAdding(); 
    } 
  }); 

  Task consumer = taskFactory.StartNew(() => 
  { 
    while (!blockingCollection.IsCompleted) 
    { 
      try 
      { 
        int imageID = blockingCollection.Take(); 
        ProcessImage(imageID); 
      } 
      catch (Exception ex) 
      { 
        //Log exception 
      } 
    } 
  }); 

  Console.Read(); 

} 

public static int ReadImageFromDB() 
{ 
  Thread.Sleep(1000); 
  Console.WriteLine("Image is read"); 
  return 1; 
} 

public static void ProcessImage(int imageID) 
{ 
  Thread.Sleep(1000); 
  Console.WriteLine("Image is processed"); 

} 
```

在上面的示例中，我们初始化了通用的`BlockingCollection<int>`来存储由生产者添加并通过消费者处理的`imageID`。我们将集合的最大大小设置为 10。然后，我们添加了一个`Producer`项，它从数据库中读取图像并调用`Add`方法将`imageID`添加到阻塞集合中，消费者可以进一步提取并处理。消费者任务只需检查集合中是否有可用项目并对其进行处理。

要了解有关并行编程可用的数据结构，请参阅[`docs.microsoft.com/en-us/dotnet/standard/parallel-programming/data-structures-for-parallel-programming`](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/data-structures-for-parallel-programming)。

# Parallel.ForEach

`Parallel.ForEach`是经典`foreach`循环的多线程版本。`foreach`循环在单个线程上运行，而`Parallel.ForEach`在多个线程上运行，并利用 CPU 的多个核心（如果可用）。

以下是一个基本示例，使用`Parallel.ForEach`处理需要处理的文档列表，并包含 I/O 绑定操作：

```cs
static void Main(string[] args) 
{ 
  List<Document> docs = GetUserDocuments(); 
  Parallel.ForEach(docs, (doc) => 
  { 
    ManageDocument(doc); 
  }); 
} 
private static void ManageDocument(Document doc) => Thread.Sleep(1000); 
```

为了复制 I/O 绑定的操作，我们只是在`ManageDocument`方法中添加了 1 秒的延迟。如果您使用`foreach`循环执行相同的方法，差异将是明显的。

# 并行 LINQ（PLINQ）

并行 LINQ 是 LINQ 的一个版本，它在多核 CPU 上并行执行查询。它包含完整的标准 LINQ 查询操作符以及一些用于并行操作的附加操作符。强烈建议您在长时间运行的任务中使用此功能，尽管不正确的使用可能会降低应用程序的性能。并行 LINQ 操作集合，如`List`，`List<T>`，`IEnumerable`，`IEnumerable<T>`等。在底层，它将列表分割成段，并在 CPU 的不同处理器上运行每个段。

以下是上一个示例的修改版本，使用`Parallel.ForEach`而不是 PLINQ 操作：

```cs
static void Main(string[] args) 
{ 
  List<Document> docs = GetUserDocuments(); 

  var query = from doc in docs.AsParallel() 
  select ManageDocument(doc); 
} 

private static Document ManageDocument(Document doc) 
{ 
  Thread.Sleep(1000); 
  return doc; 
} 
```

# 摘要

在本章中，我们学习了多线程和异步编程的核心基础知识。本章从两者之间的基本区别开始，并介绍了一些关于多线程的核心概念，可用的 API 以及如何编写多线程应用程序。我们还看了任务编程库如何用于提供异步操作以及如何实现任务异步模式。最后，我们探讨了并行编程技术以及用于这些技术的一些最佳设计模式。

在下一章中，我们将探讨数据结构的类型及其对性能的影响，如何编写优化的代码以及一些最佳实践。
