# 第九章 使用异步 I/O

在本章中，我们将详细讨论异步输入/输出操作。您将学到以下内容：

+   异步处理文件

+   编写异步 HTTP 服务器和客户端

+   异步处理数据库

+   异步调用 WCF 服务

# 介绍

在之前的章节中，我们已经讨论了正确使用异步输入/输出操作的重要性。为什么这么重要呢？为了有一个坚实的理解，让我们考虑两种应用程序。

如果我们在客户端上运行应用程序，最重要的事情之一就是拥有一个响应迅速的用户界面。这意味着无论应用程序发生什么，所有用户界面元素，如按钮和进度条，都能快速运行，并且用户能够立即得到应用程序的反应。这并不容易实现！如果您尝试在 Windows 中打开记事本文本编辑器，并尝试加载一个几兆字节大小的文档，应用程序窗口将会在相当长的时间内冻结，因为整个文本首先要从磁盘加载，然后程序才开始处理用户输入。

这是一个非常重要的问题，在这种情况下，唯一的解决方案是尽一切可能避免阻塞 UI 线程。这反过来意味着为了防止阻塞 UI 线程，每个与 UI 相关的 API 都必须只允许异步调用。这是 Windows 8 操作系统重新设计 API 的关键原因，几乎用异步模拟替换了几乎每个方法。但是，如果我们的应用程序使用多个线程来实现这个目标，会影响性能吗？当然会！然而，考虑到我们只有一个用户，我们可以付出代价。如果应用程序能够利用计算机的所有能力，以更有效的方式为运行应用程序的单个用户提供服务，那就很好。

让我们再看看第二种情况。如果我们在服务器上运行应用程序，情况就完全不同了。我们把可扩展性作为首要任务，这意味着单个用户应尽可能少地消耗资源。如果我们为每个用户创建许多线程，那么我们就无法很好地扩展。在有效的方式中平衡应用程序资源消耗是一个非常复杂的问题。例如，在微软的 Web 应用程序平台 ASP.NET 中，我们使用一个工作线程池来为客户端请求提供服务。这个池有一定数量的工作线程，我们必须尽量减少每个工作线程的使用时间以实现可扩展性。这意味着我们必须尽快将其返回到池中，以便它可以为另一个请求提供服务。如果我们开始一个需要计算的异步操作，我们将会有一个非常低效的工作流程。首先我们从线程池中取出一个工作线程来为客户端请求提供服务。然后我们再取出另一个工作线程并在其上启动一个异步操作。现在我们有两个工作线程为我们的请求提供服务，如果第一个线程正在做一些有用的事情，那就很好了！不幸的是，通常情况是我们只是等待异步操作完成，我们消耗了两个工作线程而不是一个。在这种情况下，异步实际上比同步执行更糟糕！我们不需要加载所有的 CPU 核心，因为我们已经为许多客户端提供服务，因此正在使用所有的 CPU 计算能力。我们不需要保持第一个线程响应，因为我们没有用户界面。那么为什么我们要在服务器应用程序中使用异步呢？

答案是，当存在异步输入/输出操作时，我们应该使用异步处理。如今，现代计算机通常具有存储文件的硬盘驱动器和通过网络发送和接收数据的网络卡。这两个设备都有自己的微型计算机，用于在非常低的级别上管理输入/输出操作并向操作系统发出结果。这又是一个相当复杂的话题；但为了保持概念清晰，我们可以说程序员有一种方式来启动输入/输出操作，并在操作完成时向操作系统提供一个回调代码。在启动 I/O 任务和其完成之间，不涉及 CPU 工作；它是在相应的磁盘和网络控制器微型计算机中完成的。这种执行 I/O 任务的方式称为 I/O 线程；它们是使用.NET 线程池实现的，并且反过来使用操作系统的 I/O 完成端口基础设施。

在 ASP.NET 中，一旦从工作线程启动了异步 I/O 操作，它就可以立即返回到线程池！在操作进行时，此线程可以为其他客户端提供服务。最后，当操作完成时，ASP.NET 基础结构会从线程池中获取一个空闲的工作线程（可能与启动操作的线程不同），并完成操作。

好了，现在我们明白了 I/O 线程对服务器应用程序有多么重要。不幸的是，很难看出任何给定的 API 是否在后台使用 I/O 线程。除了研究源代码之外，唯一的方法就是知道.NET Framework 类库利用了 I/O 线程。在本章中，我们将看到如何使用其中一些 API。我们将学习如何异步处理文件，如何使用网络 I/O 创建 HTTP 服务器并调用 Windows Communication Foundation 服务，以及如何使用异步 API 查询数据库。

### 注意

另一个需要考虑的重要问题是并行性。由于许多原因，密集的并行磁盘操作可能会导致性能非常差。请注意，并行 I/O 操作通常非常低效，可能会合理地按顺序但是以异步方式处理 I/O。

# 异步处理文件

本教程将指导我们如何创建文件，以及如何异步读取和写入数据。

## 准备工作

要按照本教程，您需要 Visual Studio 2012。没有其他先决条件。本教程的源代码可以在`BookSamples\Chapter9\Recipe1`中找到。

## 如何做...

要了解如何异步处理文件，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# **控制台应用程序**项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
const int BUFFER_SIZE = 4096;

async static Task ProcessAsynchronousIO()
{
  using (var stream = new FileStream("test1.txt", FileMode.Create, FileAccess.ReadWrite, FileShare.None, BUFFER_SIZE))
  {
    Console.WriteLine("1\. Uses I/O Threads: {0}", stream.IsAsync);

    byte[] buffer = Encoding.UTF8.GetBytes(CreateFileContent());
    var writeTask = Task.Factory.FromAsync(stream.BeginWrite, stream.EndWrite, buffer, 0, buffer.Length, null);

    await writeTask;
  }

  using (var stream = new FileStream("test2.txt", FileMode.Create, FileAccess.ReadWrite,FileShare.None, BUFFER_SIZE, FileOptions.Asynchronous))
  {
    Console.WriteLine("2\. Uses I/O Threads: {0}", stream.IsAsync);

    byte[] buffer = Encoding.UTF8.GetBytes(CreateFileContent());
    var writeTask = Task.Factory.FromAsync(stream.BeginWrite, stream.EndWrite, buffer, 0, buffer.Length, null);

    await writeTask;
  }

  using (var stream = File.Create("test3.txt", BUFFER_SIZE, FileOptions.Asynchronous))
  using (var sw = new StreamWriter(stream))
  {
    Console.WriteLine("3\. Uses I/O Threads: {0}", stream.IsAsync);
    await sw.WriteAsync(CreateFileContent());
  }

  using (var sw = new StreamWriter("test4.txt", true))
  {
    Console.WriteLine("4\. Uses I/O Threads: {0}", ((FileStream)sw.BaseStream).IsAsync);
    await sw.WriteAsync(CreateFileContent());
  }

  Console.WriteLine("Starting parsing files in parallel");

  Task<long>[] readTasks = new Task<long>[4];
  for (int i = 0; i < 4; i++)
  {
    readTasks[i] = SumFileContent(string.Format("test{0}.txt", i + 1));
  }

  long[] sums = await Task.WhenAll(readTasks);

  Console.WriteLine("Sum in all files: {0}", sums.Sum());

  Console.WriteLine("Deleting files...");

  Task[] deleteTasks = new Task[4];
  for (int i = 0; i < 4; i++)
  {
    string fileName = string.Format("test{0}.txt", i + 1);
    deleteTasks[i] = SimulateAsynchronousDelete(fileName);
  }

  await Task.WhenAll(deleteTasks);

  Console.WriteLine("Deleting complete.");
}

static string CreateFileContent()
{
  var sb = new StringBuilder();
  for (int i = 0; i < 100000; i++)
  {
    sb.AppendFormat("{0}", new Random(i).Next(0, 99999));
    sb.AppendLine();
  }
  return sb.ToString();
}

async static Task<long> SumFileContent(string fileName)
{
  using (var stream = new FileStream(fileName, FileMode.Open, FileAccess.Read,FileShare.None, BUFFER_SIZE, FileOptions.Asynchronous))
  using (var sr = new StreamReader(stream))
  {
    long sum = 0;
    while (sr.Peek() > -1)
    {
      string line = await sr.ReadLineAsync();
      sum += long.Parse(line);
    }

    return sum;
  }
}

static Task SimulateAsynchronousDelete(string fileName)
{
  return Task.Run(() => File.Delete(fileName));
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
var t = ProcessAsynchronousIO();
t.GetAwaiter().GetResult();
```

1.  运行程序。

## 工作原理...

程序运行时，我们以不同的方式创建四个文件，并用随机数据填充它们。在第一种情况下，我们使用`FileStream`类及其方法，将异步编程模型 API 转换为任务；在第二种情况下，我们做同样的事情，但是我们为`FileStream`构造函数提供了`FileOptions.Asynchronous`。

### 注意

非常重要的是使用`FileOptions.Asynchronous`选项。如果我们省略此选项，仍然可以以异步方式处理文件，但这只是线程池上的异步委托调用！只有在提供此选项（或另一个构造函数重载中的`bool useAsync`）时，我们才能使用`FileStream`类进行 I/O 异步处理。

第三种情况使用了一些简化的 API，比如`File.Create`方法和`StreamWriter`类。它仍然使用 I/O 线程，我们可以通过使用`stream.IsAsync`属性来检查。最后一种情况说明了过度简化也是不好的。在这里，我们没有利用 I/O 的异步性，而是通过异步委托调用来模拟它。

现在我们从文件中进行并行异步读取，对它们的内容进行求和，然后再将它们相加。最后，我们删除所有的文件。由于在任何非 Windows 存储应用程序中都没有异步删除文件的方法，我们使用`Task.Run`工厂方法来模拟异步。

# 编写一个异步 HTTP 服务器和客户端

这个步骤展示了如何创建一个简单的异步 HTTP 服务器。

## 准备工作

要按照这个步骤，你需要 Visual Studio 2012。不需要其他先决条件。这个步骤的源代码可以在`BookSamples\Chapter9\Recipe2`中找到。

## 如何做...

以下步骤演示了如何创建一个简单的异步 HTTP 服务器：

1.  启动 Visual Studio 2012。创建一个新的 C#控制台应用程序项目。

1.  添加对`System.Net.Http`框架库的引用。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.IO;
using System.Net;
using System.Net.Http;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
static async Task GetResponseAsync(string url)
{
  using (var client = new HttpClient())
  {
    HttpResponseMessage responseMessage = await client.GetAsync(url);
    string responseHeaders = responseMessage.Headers.ToString();
    string response = await responseMessage.Content.ReadAsStringAsync();

    Console.WriteLine("Response headers:");
    Console.WriteLine(responseHeaders);
    Console.WriteLine("Response body:");
    Console.WriteLine(response);
  }
}

class AsyncHttpServer
{
  readonly HttpListener _listener;
  const string RESPONSE_TEMPLATE = "<html><head><title>Test</title></head><body><h2>Test page</h2><h4>Today is: {0}</h4></body></html>";

  public AsyncHttpServer(int portNumber)
  {
    _listener = new HttpListener();
    _listener.Prefixes.Add(string.Format("http://+:{0}/", portNumber));
  }

  public async Task Start()
  {
    _listener.Start();

    while (true)
    {
      var ctx = await _listener.GetContextAsync();
      Console.WriteLine("Client connected...");
      string response = string.Format(RESPONSE_TEMPLATE, DateTime.Now);

      using (var sw = new StreamWriter(ctx.Response.OutputStream))
      {
        await sw.WriteAsync(response);
        await sw.FlushAsync();
      }
    }
  }

  public async Task Stop()
  {
    _listener.Abort();
  }
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
var server = new AsyncHttpServer(portNumber: 1234);
var t = Task.Run(() => server.Start());
Console.WriteLine("Listening on port 1234\. Open http://localhost:1234 in your browser.");
Console.WriteLine("Trying to connect:");
Console.WriteLine();

GetResponseAsync("http://localhost:1234").GetAwaiter().GetResult();

Console.WriteLine();
Console.WriteLine("Press Enter to stop the server.");
Console.ReadLine();

server.Stop().GetAwaiter().GetResult();
```

1.  运行程序。

## 工作原理...

在这里，我们使用`HttpListener`类实现了一个非常简单的 Web 服务器。还有一个`TcpListener`类用于 TCP 套接字 I/O 操作。我们配置我们的监听器以接受来自本地机器上任何主机的连接，端口为`1234`。然后我们在一个单独的工作线程中启动监听器，这样我们就可以从主线程中控制它。

当我们使用`GetContextAsync`方法时，异步 I/O 操作发生。不幸的是，它不接受`CancellationToken`用于取消场景；所以当我们想要停止服务器时，我们只需调用`_listener.Abort`方法，它会放弃所有连接并停止服务器。

要对这个服务器执行异步请求，我们使用`System.Net.Http`程序集中的`HttpClient`类和相同的命名空间。我们使用`GetAsync`方法来发出异步的 HTTP `GET`请求。还有其他 HTTP 请求的方法，比如`POST`、`DELETE`和`PUT`。`HttpClient`还有许多其他选项，比如使用不同格式（如 XML 和 JSON）对对象进行序列化和反序列化，指定代理服务器地址、凭据等。

当你运行程序时，你会看到服务器已经启动。在服务器代码中，我们使用`GetContextAsync`方法来接受新的客户端连接。当一个新的客户端连接时，这个方法会返回，我们只是简单地输出一个包含当前日期和时间的基本 HTML 到响应中。然后我们请求服务器并打印响应头和内容。你也可以打开浏览器并浏览到`http://localhost:1234/`的 URL。你会在浏览器窗口中看到相同的响应。

# 异步处理数据库

这个步骤将指导我们创建一个数据库，用数据填充它，并异步读取数据的过程。

## 准备工作

要按照这个步骤，你需要运行 Visual Studio 2012。不需要其他先决条件。这个步骤的源代码可以在`BookSamples\Chapter9\Recipe3`中找到。

## 如何做...

为了理解创建数据库、填充数据和异步读取数据的过程，执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C#控制台应用程序项目。

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.Data;
using System.Data.SqlClient;
using System.IO;
using System.Reflection;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
async static Task ProcessAsynchronousIO(string dbName)
{
  try
  {
    const string connectionString = @"Data Source=(LocalDB)\v11.0;Initial Catalog=master;Integrated Security=True";
    string outputFolder = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
    string dbFileName = Path.Combine(outputFolder, string.Format(@".\{0}.mdf", dbName));
    string dbLogFileName = Path.Combine(outputFolder, string.Format(@".\{0}_log.ldf", dbName));
    string dbConnectionString = string.Format(@"Data Source=(LocalDB)\v11.0;AttachDBFileName={1};Initial Catalog={0};Integrated Security=True;", dbName, dbFileName);

    using (var connection = new SqlConnection(connectionString))
    {
      await connection.OpenAsync();

      if (File.Exists(dbFileName))
      {
        Console.WriteLine("Detaching the database...");

        var detachCommand = new SqlCommand("sp_detach_db", connection);
        detachCommand.CommandType = CommandType.StoredProcedure;
        detachCommand.Parameters.AddWithValue("@dbname", dbName);

        await detachCommand.ExecuteNonQueryAsync();

        Console.WriteLine("The database was detached successfully.");
        Console.WriteLine("Deleting the database...");

        if(File.Exists(dbLogFileName)) File.Delete(dbLogFileName);
        File.Delete(dbFileName);

        Console.WriteLine("The database was deleted successfully.");
      }

      Console.WriteLine("Creating the database...");
      string createCommand = String.Format("CREATE DATABASE {0} ON (NAME = N'{0}', FILENAME = '{1}')", dbName, dbFileName);
      var cmd = new SqlCommand(createCommand, connection);

      await cmd.ExecuteNonQueryAsync();
      Console.WriteLine("The database was created successfully");
    }

    using (var connection = new SqlConnection(dbConnectionString))
    {
      await connection.OpenAsync();

      var cmd = new SqlCommand("SELECT newid()", connection);
      var result = await cmd.ExecuteScalarAsync();

      Console.WriteLine("New GUID from DataBase: {0}", result);

      cmd = new SqlCommand(@"CREATE TABLE [dbo].CustomTable NOT NULL, [Name] nvarchar NOT NULL,CONSTRAINT [PK_ID] PRIMARY KEY CLUSTERED ([ID] ASC) ON [PRIMARY]) ON [PRIMARY]", connection);
      await cmd.ExecuteNonQueryAsync();

      Console.WriteLine("Table was created successfully.");

      cmd = new SqlCommand(@"INSERT INTO [dbo].[CustomTable] (Name) VALUES ('John');
      INSERT INTO [dbo].[CustomTable] (Name) VALUES ('Peter');
      INSERT INTO [dbo].[CustomTable] (Name) VALUES ('James');
      INSERT INTO [dbo].[CustomTable] (Name) VALUES ('Eugene');", connection);
      await cmd.ExecuteNonQueryAsync();

      Console.WriteLine("Inserted data successfully	");
      Console.WriteLine("Reading data from table...");

      cmd = new SqlCommand(@"SELECT * FROM [dbo].[CustomTable]", connection);
      using (SqlDataReader reader = await cmd.ExecuteReaderAsync())
      {
        while (await reader.ReadAsync())
        {
          var id = reader.GetFieldValue<int>(0);
          var name = reader.GetFieldValue<string>(1);

          Console.WriteLine("Table row: Id {0}, Name {1}", id, name);
        }
      }
    }
  }
  catch(Exception ex)
  {
    Console.WriteLine("Error: {0}", ex.Message);
  }
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
const string dataBaseName = "CustomDatabase";
var t = ProcessAsynchronousIO(dataBaseName);
t.GetAwaiter().GetResult();
Console.WriteLine("Press Enter to exit");
Console.ReadLine();
```

1.  运行程序。

## 工作原理...

这个程序使用一个名为 SQL Server 2012 LocalDb 的软件。它与 Visual Studio 2012 一起安装，应该可以正常工作。但是，如果出现错误，您可能需要从安装向导中修复此组件。

我们首先配置到我们的数据库文件的路径。我们将数据库文件放在程序执行文件夹中。将有两个文件：一个用于数据库本身，另一个用于事务日志文件。我们还配置了两个连接字符串，定义了我们如何连接到我们的数据库。第一个是连接到 LocalDb 引擎以分离我们的数据库；如果它已经存在，则删除然后重新创建它。我们利用 I/O 异步性来打开连接，并使用`OpenAsync`和`ExecuteNonQueryAsync`方法分别执行 SQL 命令。

完成此任务后，我们将附加一个新创建的数据库。在这里，我们创建一个新表并向其中插入一些数据。除了前面提到的方法之外，我们使用`ExecuteScalarAsync`来异步从数据库引擎获取标量值，并使用`SqlDataReader.ReadAsync`方法来异步从数据库表中读取数据行。

如果我们的数据库中有一个包含大型二进制值的大表，那么我们将使用`CommandBehavior.SequentialAcess`枚举来创建数据读取器，并使用`GetFieldValueAsync`方法来异步从读取器中获取大字段值。

# 异步调用 WCF 服务

本教程将描述如何创建一个 WCF 服务，在控制台应用程序中托管它，使服务元数据可用于客户端，并以异步方式消费它。

## 准备工作

要按照本教程进行步骤，您需要运行 Visual Studio 2012。没有其他先决条件。本教程的源代码可以在`BookSamples\Chapter9\Recipe4`中找到。

## 如何做...

要了解如何使用 WCF 服务，请执行以下步骤：

1.  启动 Visual Studio 2012。创建一个新的 C# **控制台应用程序**项目。

1.  添加对`System.ServiceModel`库的引用。在项目中右键单击`引用`文件夹，然后选择**添加引用...**菜单选项。添加对`System.ServiceModel`库的引用。您可以使用引用管理器对话框中的搜索功能，如下截图所示：![如何做...](img/7644OT_09_01.jpg)

1.  在`Program.cs`文件中添加以下`using`指令：

```cs
using System;
using System.ServiceModel;
using System.ServiceModel.Description;
using System.Threading.Tasks;
```

1.  在`Main`方法下面添加以下代码片段：

```cs
const string SERVICE_URL = "http://localhost:1234/HelloWorld";

static async Task RunServiceClient()
{
  var endpoint = new EndpointAddress(SERVICE_URL);
  var channel = ChannelFactory<IHelloWorldServiceClient>.CreateChannel(new BasicHttpBinding(), endpoint);

  var greeting = await channel.GreetAsync("Eugene");
  Console.WriteLine(greeting);
}

  [ServiceContract(Namespace = "Packt", Name = "HelloWorldServiceContract")]
public interface IHelloWorldService
{
  [OperationContract]
  string Greet(string name);
}

[ServiceContract(Namespace = "Packt", Name = "HelloWorldServiceContract")]
public interface IHelloWorldServiceClient
{
  [OperationContract]string Greet(string name);

  [OperationContract]Task<string> GreetAsync(string name);
}

public class HelloWorldService : IHelloWorldService
{
  public string Greet(string name)
  {
    return string.Format("Greetings, {0}!", name);
  }
}
```

1.  在`Main`方法中添加以下代码片段：

```cs
ServiceHost host = null;

try
{
  host = new ServiceHost(typeof (HelloWorldService), new Uri(SERVICE_URL));
  var metadata = host.Description.Behaviors.Find<ServiceMetadataBehavior>();
  if (null == metadata)
  {
    metadata = new ServiceMetadataBehavior();
  }

  metadata.HttpGetEnabled = true;
  metadata.MetadataExporter.PolicyVersion = PolicyVersion.Policy15;
  host.Description.Behaviors.Add(metadata);

  host.AddServiceEndpoint(ServiceMetadataBehavior.MexContractName, MetadataExchangeBindings.CreateMexHttpBinding(),"mex");
  var endpoint = host.AddServiceEndpoint(typeof (IHelloWorldService), new BasicHttpBinding(), SERVICE_URL);

  host.Faulted += (sender, e) => Console.WriteLine("Error!");

  host.Open();

  Console.WriteLine("Greeting service is running and listening on:");
  Console.WriteLine("{0} ({1})", endpoint.Address, endpoint.Binding.Name);

  var client = RunServiceClient();
  client.GetAwaiter().GetResult();

  Console.WriteLine("Press Enter to exit");
  Console.ReadLine();
}
catch (Exception ex)
{
  Console.WriteLine("Error in catch block: {0}", ex);
}
finally
{
  if (null != host)
  {
    if (host.State == CommunicationState.Faulted)
    {
      host.Abort();
    }
    else
    {
      host.Close();
    }
  }
}
```

1.  运行程序。

## 工作原理...

Windows Communication Foundation 或 WCF 是一个框架，允许我们以不同的方式调用远程服务。其中一种曾经非常流行的方式是使用基于 XML 的协议**简单对象访问协议**（**SOAP**）通过 HTTP 调用远程服务。当服务器应用程序调用另一个远程服务时，这是相当常见的，也可以使用 I/O 线程来完成。

Visual Studio 2012 对 WCF 服务有很好的支持；例如，您可以使用**添加服务引用**菜单选项添加对这些服务的引用。您也可以对我们的服务进行此操作，因为我们提供了服务元数据。

创建这样一个服务，我们需要使用`ServiceHost`类来托管我们的服务。我们通过提供服务实现类型和服务的基本 URI 来描述我们将托管的服务。然后我们配置元数据端点和服务端点。最后，我们处理`Faulted`事件以处理错误，并运行主机服务。

为了消费这个服务，我们创建一个客户端，这就是主要的技巧所在。在服务器端，我们有一个带有通常同步方法的服务，称为`Greet`。这个方法在服务契约`IHelloWorldService`中定义。然而，如果我们想利用异步网络 I/O，我们必须异步调用这个方法。我们可以通过创建一个新的服务契约来做到这一点，其中包含匹配的命名空间和服务名称，在这里我们定义同步和基于任务的异步方法。尽管我们在服务器端没有异步方法的定义，但我们遵循命名约定，WCF 基础设施会理解我们想要创建一个异步代理方法。

因此，当我们创建一个`IHelloWorldServiceClient`代理通道时，WCF 会正确地将异步调用路由到服务器端的同步方法。如果您让应用程序保持运行状态，您可以打开浏览器并使用其 URL 访问服务，即`http://localhost:1234/HelloWorld`。将打开一个服务描述，您可以浏览到允许我们从 Visual Studio 2012 添加服务引用的 XML 元数据。如果您尝试生成引用，您将看到稍微复杂一些的代码，但它是自动生成的并且易于使用。
