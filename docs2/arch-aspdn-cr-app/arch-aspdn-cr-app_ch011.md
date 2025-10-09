# 10 日志模式

## 在开始之前：加入我们的 Discord 书籍社区

直接向作者本人提供反馈，并在我们的 Discord 服务器上与其他早期读者聊天（在“EARLY ACCESS SUBSCRIPTION”下找到“architecting-aspnet-core-apps-3e”频道）。

[`packt.link/EarlyAccess`](https://packt.link/EarlyAccess)

![二维码描述自动生成](img/file46.png)

本章涵盖了.NET 特定的功能，并结束了*为 ASP.NET Core 设计*部分。带有一些模式的日志功能是大多数应用程序需要的另一个构建块：内置的 ASP.NET Core。我们在不试图掌握每个方面的同时，亲手探索这个系统。日志是应用程序开发的一个关键方面，它服务于各种目的，如调试错误、跟踪操作、分析使用情况等。我们在这里探索的日志抽象是.NET Core 相对于.NET Framework 的另一个改进。我们不再依赖于第三方库，新的统一系统提供了一个清洁的接口，由一个灵活且健壮的机制支持，这有助于将日志集成到我们的应用程序中。在本章结束时，你将了解什么是日志以及如何编写应用程序日志。在本章中，我们将涵盖以下主题：

+   关于日志

+   编写日志

+   日志级别

+   日志提供者

+   配置日志

+   结构化日志

让我们先来探索一下什么是日志。

## 关于日志

记录日志是将消息写入日志并编制信息以备后用的实践。这些信息可以用于调试错误、追踪操作、分析使用情况或任何我们可以想到的其他原因。日志记录是一个横切关注点，这意味着它适用于你应用程序的每一部分。我们在*第十四章*，*分层与清洁架构*中讨论了层，但在此之前，我们只需说横切关注点影响所有层，不能仅集中在一个层；它影响了一切。日志由日志条目组成。我们可以将每个日志条目视为程序执行期间发生的事件。这些事件随后被写入日志。这个日志可以是文件、远程系统、`stdout`或多个目的地的组合。在创建日志条目时，我们还必须考虑该日志条目的严重性。从某种意义上说，这种严重性级别代表了我们要记录的消息类型或重要性的级别。我们还可以用它来过滤这些日志。"跟踪"、"错误"和"调试"是日志条目级别的例子。这些级别在`Microsoft.Extensions.Logging.LogLevel`枚举中定义。日志条目的另一个重要方面是其结构。你可以记录单个字符串。你团队中的每个人都可以以自己的方式记录单个字符串。但当你需要搜索信息时会发生什么？混乱随之而来！有找不到那个人正在寻找的东西的压力，以及同样的人体验到的日志结构的令人不快。解决这个问题的一种方法是通过使用结构化日志记录。它简单而又复杂；你必须为所有日志条目创建一个程序遵循的结构。这个结构可能更复杂或更简单，或者可以序列化为 JSON。重要的是日志条目是有结构的。我们不会在这里深入探讨这个主题，但如果你必须决定日志策略，我建议首先深入了解结构化日志记录。如果你是团队的一员，那么很可能会有人已经做了。如果不是这样，你总是可以提出这个话题。持续改进是生活的一个关键方面。我们本可以写一本关于日志、最佳日志实践、结构化日志记录和分布式跟踪的书，但本章旨在教你如何使用.NET 日志抽象。

## 编写日志

首先，日志系统是基于提供程序的，这意味着如果我们想让我们的日志条目有地方记录，我们必须注册一个或多个 `ILoggerProvider` 实例。默认情况下，当调用 `WebApplication.CreateBuilder(args)` 时，它会注册控制台、调试、事件源和事件日志（仅限 Windows）提供程序，但我们可以修改这个列表。如果需要，您可以添加和删除提供程序。使用日志系统的必需依赖项也作为此过程的一部分进行注册。在我们查看代码之前，让我们学习如何创建日志条目，这是日志记录的目标。要创建条目，我们可以使用以下接口之一：`ILogger`、`ILogger<T>` 或 `ILoggerFactory`。让我们更详细地看看它们：

| **接口** | **描述** |
| --- | --- |
| `ILogger` | 允许我们执行日志记录操作的基本类型。 |
| `ILogger<T>` | 允许我们执行日志记录操作的基本类型。从 `ILogger` 接口继承。系统使用泛型参数 `T` 作为日志条目的 *类别*。 |
| `ILoggerFactory` | 一个允许创建 `ILogger` 对象并手动指定类别名称（作为字符串）的工厂接口。 |

表 10.1：日志接口。

以下代码表示最常用的模式，该模式包括注入一个 `ILogger<T>` 接口并将其存储在一个 `ILogger` 字段中，然后再使用它，如下所示：

```cs
public class Service : IService
{
    private readonly ILogger _logger;
    public Service(ILogger<Service> logger)
    {
        _logger = logger;
    }
    public void Execute()
    {
        _logger.LogInformation("Service.Execute()");
    }
}
```

前面的 `Service` 类有一个私有 `_logger` 字段。它接受一个 `ILogger<Service>` 日志记录器作为参数并将其存储在该字段中。它使用该字段在 `Execute` 方法中向日志写入信息级别的消息。《IService》接口非常简单，仅公开一个 `Execute` 方法用于测试目的：

```cs
public interface IService
{
    void Execute();
}
```

我加载了一个我创建的小型库来测试这个功能，为测试目的提供了额外的日志提供程序。有了这个，我们创建了一个泛型宿主 (`IHost`)，因为我们不需要在测试中使用 `WebApplication`，然后我们进行配置：

```cs
namespace Logging;
public class BaseAbstractions
{
    [Fact]
    public void Should_log_the_Service_Execute_line()
    {
        // Arrange
        var lines = new List<string>();
        var host = Host.CreateDefaultBuilder()
            .ConfigureLogging(loggingBuilder =>
            {
                loggingBuilder.ClearProviders();
                loggingBuilder.AddAssertableLogger(lines);
            })
            .ConfigureServices(services =>
            {
                services.AddSingleton<Service>();
            })
            .Build();
        var service = host.Services.GetRequiredService<Service>(); 
        // Act
        service.Execute();
        // Assert
        Assert.Collection(lines,
            line => Assert.Equal("Service.Execute()", line)
        );
    }
    // Omitted other members
}
```

在测试的 `Arrange` 阶段，我们创建了一些变量，配置 `IHost`，并获取我们想要用于测试我们编写的日志功能的 `Service` 类的实例。突出显示的代码使用 `ClearProviders` 方法删除了所有提供程序。然后它使用 `AddAssertableLogger` 扩展方法添加了一个新的提供程序。扩展方法来自我们加载的库。如果我们想添加一个新的提供程序，我们也可以这样做，但我想要展示如何删除现有的提供程序，这样我们就可以从一张干净的纸开始。这可能是你将来需要的东西。

> 我加载的库可在 NuGet 上找到，名称为 `ForEvolve.Testing.Logging`，但您不需要理解任何这些内容就能理解日志抽象和示例。

在`Act`阶段，我们调用我们服务的`Execute`方法。该方法将一行记录到在实例化时注入的`ILogger`实现中。然后，我们断言该行被写入到`lines`列表中（这就是`AssertableLogger`的作用；它将内容写入到`List<string>`）。在 ASP.NET Core 应用程序中，所有这些日志默认都会输出到控制台。日志记录是了解应用程序运行时后台发生情况的好方法。《Service》类是一个简单的`ILogger<Service>`消费者。你可以为任何你想添加日志支持的类做同样的事情。通过将`Service`替换为那个类名来为你的类配置一个日志记录器。这个泛型参数成为写入日志条目时的日志记录器类别名称。由于 ASP.NET Core 使用`WebApplication`而不是通用`IHost`，以下是使用该结构的相同测试代码：

```cs
[Fact]
public void Should_log_the_Service_Execute_line_using_WebApplication()
{
    // Arrange
    var lines = new List<string>();
    var builder = WebApplication.CreateBuilder();
    builder.Logging.ClearProviders()
        .AddAssertableLogger(lines);
    builder.Services.AddSingleton<IService, Service>();
    var app = builder.Build();
    var service = app.Services.GetRequiredService<IService>();
    // Act
    service.Execute();
    // Assert
    Assert.Collection(lines,
        line => Assert.Equal("Service.Execute()", line)
    );
}
```

我在前面代码中高亮显示了变更。简而言之，在通用宿主上使用的扩展方法已经被`WebApplicationBuilder`的属性如`Logging`和`Services`所替代。最后，`Create`方法创建了一个`WebApplication`而不是`IHost`，这与`Program.cs`文件中的情况完全一样。总结来说，这些测试案例使我们能够实现 ASP.NET Core 中最常用的日志记录模式，并添加一个自定义提供程序以确保我们记录了正确的信息。日志记录是必不可少的，它为生产系统提供了可见性。除非你是唯一使用系统的人，否则没有日志，你不知道系统中发生了什么，这是非常不可能的。你还可以记录基础设施中发生的事情，并对这些日志流进行实时安全分析，以快速识别安全漏洞、入侵尝试或系统故障。这些主题超出了本书的范围，但拥有强大的应用程序级日志记录能力只能有助于你的整体日志记录策略。在继续下一个主题之前，让我们探索一个利用`ILoggerFactory`接口的示例。代码设置了一个自定义类别名称，并使用创建的`ILogger`实例记录一条消息。这与前面的例子非常相似。以下是整个代码：

```cs
namespace Logging;
public class LoggerFactoryExploration
{
    private readonly ITestOutputHelper _output;
    public LoggerFactoryExploration(ITestOutputHelper output)
    {
        _output = output ?? throw new ArgumentNullException(nameof(output));
    }
    [Fact]
    public void Create_a_ILoggerFactory()
    {
        // Arrange
        var lines = new List<string>();
        var host = Host.CreateDefaultBuilder()
            .ConfigureLogging(loggingBuilder => loggingBuilder
                .AddAssertableLogger(lines)
                .AddxUnitTestOutput(_output))
            .ConfigureServices(services => services.AddSingleton<Service>())
            .Build()
        ;
        var service = host.Services.GetRequiredService<Service>();
        // Act
        service.Execute();
        // Assert
        Assert.Collection(lines,
            line => Assert.Equal("LogInformation like any ILogger<T>.", line)
        );
    }
    public class Service
    {
        private readonly ILogger _logger;
        public Service(ILoggerFactory loggerFactory)
        {
            ArgumentNullException.ThrowIfNull(loggerFactory);
            _logger = loggerFactory.CreateLogger("My Service");
        }
        public void Execute()
        {
            _logger.LogInformation("LogInformation like any ILogger<T>.");
        }
    }
}
```

前面的代码应该看起来非常熟悉。让我们关注高亮显示的行，它们与当前模式相关：

1.  我们将`ILoggerFactory`接口注入到`Service`类的构造函数中（而不是`ILogger<Service>`）。

1.  我们使用`"` `My Service"`这个类别名称创建了一个`ILogger`实例。

1.  我们将日志记录器分配给`_logger`字段。

1.  然后，我们在`Execute`方法中使用那个`ILogger`。

作为一项经验法则，我建议默认使用 `ILogger<T>` 接口。如果不可能，或者如果您需要为日志条目设置更动态的类别名称，则利用 `ILoggerFactory`。默认情况下，使用 `ILogger<T>` 时，类别名称是 T 参数，应该是创建日志条目的类的名称。`ILoggerFactory` 接口更像是内部组件，而不是为我们消费的组件；尽管如此，它存在并满足某些用例。

> **注意**
> 
> > 在前面的示例中，`ITestOutputHelper` 接口是 `Xunit.Abstractions` 集合的一部分。它允许我们将行作为 *标准输出* 写入测试日志。该输出在 Visual Studio 测试资源管理器中可用。

现在我们已经介绍了如何编写日志条目，现在是时候学习如何管理它们的严重性了。

## 日志级别

在前面的示例中，我们使用了 `LogInformation` 方法来记录信息消息，但还有其他级别，如下表所示：

| **级别** | **方法** | **描述** | **生产** |
| --- | --- | --- | --- |
| 跟踪 | `LogTrace` | 这用于捕获有关程序、执行速度和调试的详细信息。您还可以在跟踪时记录敏感信息。 | 禁用。 |
| 调试 | `LogDebug` | 这用于记录调试和开发信息。 | 除非故障排除，否则禁用。 |
| 信息 | `LogInformation` | 这用于跟踪应用程序的流程。系统中发生的正常事件通常是信息级别的事件，例如系统启动、系统停止和用户登录。 | 启用。 |
| 警告 | `LogWarning` | 这用于记录应用程序流程中的异常行为，不会导致程序停止，但可能需要调查；例如，已处理的异常、失败的网络调用和访问不存在的资源。 | 启用。 |
| 错误 | `LogError` | 这用于记录应用程序流程中的错误，不会导致应用程序停止。错误通常必须进行调查。例如，当前操作失败和无法处理的异常。 | 启用。 |
| 严重 | `LogCritical` | 这用于记录需要立即注意的错误，表示灾难性状态。程序很可能会停止，应用程序的完整性可能受到损害；例如，硬盘已满，服务器内存不足，或数据库处于死锁状态。 | 启用，某些警报可以配置为自动触发。 |

表 10.2：日志条目级别

如前表所述，每个日志级别都服务于一个或多个目的。这些日志级别告诉记录器日志条目的严重性。然后，我们可以配置系统只记录至少达到一定级别的条目，例如，不要在生产日志中填充跟踪和调试条目。在我领导的项目中，我们使用 ASP.NET Core 对记录简单和复杂消息的多种方法进行了基准测试，以围绕这一点建立清晰和优化的指南。当消息被记录时，由于基准测试运行之间的大时间差异，我们无法得出公平的结论。然而，当消息没有被记录时（例如，配置了最小日志级别为*debug*的*trace*日志），我们观察到一种恒定的趋势。基于这个结论，我建议使用以下结构而不是插值、`string.Format`或其他方式来记录`Trace`和`Debug`消息。这可能听起来很奇怪，为了优化“不记录某些内容”，但如果你这么想，这些日志条目将在生产中被跳过，因此优化它们将节省你的生产应用在这里和那里的一些毫秒计算时间。此外，这并不更难或更耗时，所以这只是一个好习惯。让我们看看不写入日志条目的最快方式：

```cs
_logger.LogTrace("Some: {variable}", variable);
// Or
_logger.LogTrace("Some: {0}", variable);
```

当日志级别被禁用时，例如在生产环境中，你只需支付方法调用的代价，因为你的日志条目没有进行任何处理。另一方面，如果我们使用插值，则会进行处理，因此一个参数被传递给`Log[Level]`方法，导致每个日志条目的处理能力成本更高。以下是一个插值的例子（也就是不要这样做）：

```cs
_logger.LogTrace($"Some: {variable}");
```

对于警告及以上级别，你可以保持良好的习惯，使用相同的技巧或其他方法，因为我们知道这些行无论如何都会被记录。因此，在代码中使用插值或让记录器稍后执行应该会产生类似的结果。

> 最后一点。我建议在真正需要之前不要试图过度优化你的代码。在不需要优化的东西上投入大量精力进行优化的行为被称为**过早优化**。理念是提前优化到足够的程度，并在发现真正的问题时修复性能。

现在我们知道了.NET 为我们提供的日志级别，让我们来看看日志提供程序。

## 日志提供程序

为了让你了解可能的内置日志提供程序，以下是官方文档中的一个列表（请参阅本章末尾的*进一步阅读*部分）：

+   Console

+   Debug

+   EventSource

+   EventLog（仅限 Windows）

+   ApplicationInsights

以下是一个第三方日志提供程序的列表，也来自官方文档：

+   elmah.io

+   Gelf

+   JSNLog

+   KissLog.net

+   Log4Net

+   NLog

+   PLogger

+   Sentry

+   Serilog

+   Stackdriver

现在，如果你需要这些中的任何一个，或者你喜欢的日志库是前面列表的一部分，你知道你可以使用它。如果不是，也许它支持 ASP.NET Core，但在咨询时它并未包含在文档中。接下来，让我们学习如何配置日志系统。

## 配置日志记录

与 ASP.NET Core 的大多数功能一样，我们可以配置日志记录。默认的`WebApplicationBuilder`为我们做了很多工作，但如果我们想调整默认设置，我们也可以做到。除此之外，系统会加载配置中的`Logging`部分。该部分默认存在于`appsettings.json`文件中。像所有配置一样，它是累积的，因此我们可以在另一个文件或配置提供者中重新定义其部分。我们不会深入探讨自定义，但了解我们可以自定义我们记录的最小日志级别是很好的。我们还可以使用转换文件（如`appsettings.Development.json`）来根据环境自定义这些级别。例如，我们可以在`appsettings.json`中定义默认设置，然后在`appsettings.Development.json`中更新一些用于开发目的的设置，在`appsettings.Production.json`中更改生产设置，然后在`appsettings.Staging.json`中更改预发布设置，最后在`appsettings.Testing.json`中添加一些测试设置。在我们继续之前，让我们看一下默认设置：

```cs
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    }
  }
}
```

我们可以定义默认级别（使用`Logging:LogLevel:Default`）和每个类别（例如`Logging:LogLevel:Microsoft`）的自定义级别，代表基本命名空间。例如，从该配置文件中，最小级别是`Information`，而属于`Microsoft`或`Microsoft.*`命名空间的每个项目都有最小级别为`Warning`。这允许在运行应用程序时移除噪音。我们还可以利用这些配置通过降低日志级别到`Debug`或`Trace`来调试应用程序的某些部分，仅针对项目子集（例如来自一个或多个命名空间的项目）。我们还可以根据提供者基础使用配置或代码来过滤我们想要记录的内容。在配置文件中，我们可以将控制台提供者的默认级别更改为`Trace`，如下所示：

```cs
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    },
    "Console": {
      "LogLevel": {
        "Default": "Trace"
      }
    }
  }
}
```

我们保留了相同的默认值，但添加了`Logging:Console`部分（见高亮代码），并将默认`LogLevel`设置为`Trace`。我们可以定义我们需要的任何设置。而不是配置，我们可以使用`AddFilter`扩展方法，如下面的实验测试代码所示，或者与配置一起使用。以下是记录数据的消费者类：

```cs
public class Service
{
    private readonly ILogger _logger;
    public Service(ILogger<Service> logger)
    {
        _logger = logger;
    }
    public void Execute()
    {
        _logger.LogInformation("[info] Service.Execute()");
        _logger.LogWarning("[warning] Service.Execute()");
    }
}
```

前面的类与其他我们在本章中使用的类类似，但使用两种不同的级别记录消息：`Information`和`Warning`。以下是一个测试用例，其中我们利用了`AddFilter`方法：

```cs
[Fact]
public void Should_filter_logs_by_provider()
{
    // Arrange
    var lines = new List<string>();
    var host = Host.CreateDefaultBuilder()
        .ConfigureLogging(loggingBuilder =>
        {
            loggingBuilder.ClearProviders();
            loggingBuilder.AddConsole();
            loggingBuilder.AddAssertableLogger(lines);
            loggingBuilder.AddxUnitTestOutput(_output);
            loggingBuilder
                .AddFilter<XunitTestOutputLoggerProvider>(
                    level => level >= LogLevel.Warning
                );
        })
        .ConfigureServices(services =>
        {
            services.AddSingleton<Service>();
        })
        .Build();
    var service = host.Services.GetRequiredService<Service>();
    // Act
    service.Execute();
    // Assert
    Assert.Collection(lines,
        line => Assert.Equal("[info] Service.Execute()", line),
        line => Assert.Equal("[warning] Service.Execute()", line)
    );
}
```

在前面的测试代码中，我们创建了一个通用主机并添加了三个提供者：控制台和两个测试提供者——一个将日志记录到列表中，另一个记录到 xUnit 输出。然后，我们告诉系统从 `XunitTestOutputLoggerProvider` 中过滤掉所有至少不是 `Warning` 级别的日志（见高亮代码）；其他提供者不受该代码的影响。

> 在代码中，`_output` 成员是 `ITestOutputHelper` 类型的字段。

我们现在知道有两种设置最小日志级别的选项：

+   代码

+   配置

我们可以根据需要调整配置日志策略的方式。代码可以更容易地进行测试，而配置可以在运行时更新，无需重新部署。此外，使用级联模型，该模型允许我们覆盖配置，我们可以使用配置覆盖大多数用例。配置的最大缺点是，在 JSON 文件中编写字符串比编写代码更容易出错（假设你也没有退回到使用字符串）。我通常坚持使用配置来设置这些值，因为它们不经常改变。如果你更喜欢代码，我不知道有任何缺点，这只是个人偏好的问题；配置在某个时刻变成了代码。接下来，让我们看看结构化日志的一个简要示例。

## 结构化日志

如开头所述，结构化日志可以变得非常重要，并开辟了机会。查询数据结构始终比查询单行文本更灵活。如果没有关于日志的明确指南，无论是文本行还是 JSON 格式的数据结构，这一点尤其正确。为了保持简单，我们利用一个内置的格式化器（以下高亮行所示），将我们的日志条目序列化为 JSON。以下是 `Program.cs` 文件：

```cs
var builder = WebApplication.CreateBuilder(args);
builder.Logging.AddJsonConsole();
var app = builder.Build();
app.MapGet("/", (ILoggerFactory loggerFactory) =>
{
    const string category = "root";
    var logger = loggerFactory.CreateLogger(category);
    logger.LogInformation("You hit the {category} URL!", category);
    return "Hello World!";
});
app.Run();
```

这将控制台转换为记录 JSON。例如，每次我们点击 `/` 端点时，控制台会显示以下 JSON：

```cs
{
  "EventId": 0,
  "LogLevel": "Information",
  "Category": "root",
  "Message": "You hit the root URL!",
  "State": {
    "Message": "You hit the root URL!",
    "category": "root",
    "{OriginalFormat}": "You hit the {category} URL!"
  }
}
```

没有那个格式化器，通常的输出会是：

```cs
info: root[0]
      You hit the root URL!
```

基于这种比较，程序化查询 JSON 日志比查询 `stdout` 行更加灵活。结构化日志的最大好处是提高了可搜索性。您可以使用预定义的数据结构进行更精确的查询。当然，如果您正在设置生产系统，您可能希望将更多信息附加到这些日志项上，例如请求的相关 ID、可选的当前用户信息、运行代码的服务器名称，以及根据应用程序可能需要更多详细信息。您可能需要比开箱即用的功能更多，以充分利用结构化日志。一些第三方库，如 Serilog，提供了这些附加功能。然而，定义将纯文本发送到日志记录器的方式可能是一个起点。每个项目都应该规定每个功能的需求和深度，包括日志记录。此外，结构化日志是一个更广泛的主题，值得独立研究。尽管如此，我还是想简要地谈谈这个主题，并希望您已经学到了足够多的关于日志记录的知识，以便开始实践。

## 摘要

在本章中，我们深入探讨了日志记录的概念。我们了解到，日志记录是将消息记录到日志中以供以后使用的一种实践，例如调试错误、跟踪操作和分析使用情况。日志记录是必不可少的，ASP.NET Core 提供了各种方式来记录信息，而无需使用第三方库，同时允许我们使用我们喜欢的日志框架。我们可以自定义日志的编写和分类方式。我们可以使用零个或多个日志提供程序。我们还可以创建自定义日志提供程序。最后，我们可以使用配置或代码来过滤日志以及更多内容。以下是需要记住的默认日志模式：

1.  注入一个 `ILogger<T>`，其中 `T` 是注入日志记录器的类的类型。`T` 成为类别。

1.  将该日志记录器的引用保存到 `private readonly ILogger` 字段中。

1.  在您的函数中使用该日志记录器，使用适当的日志级别记录消息。

与 .NET Framework 相比，日志系统是 .NET Core 的一个很好的补充。它允许我们标准化日志机制，使我们的系统在长期内更容易维护。例如，如果您想使用一个新的第三方库，甚至是一个定制的库，您可以将提供程序加载到您的 `Program` 中，只要您只依赖于日志抽象，整个系统就会适应并开始使用它，而无需进行任何进一步的更改。这是一个很好的例子，说明了精心设计的抽象可以为系统带来什么。**以下是一些关键要点**：

+   日志记录是一个横切关注点，影响应用程序的所有层。

+   日志记录包含许多日志条目，这些条目代表程序执行期间运行时发生的事件。

+   日志条目的严重性对于过滤和优先级排序非常重要。

+   严重级别包括 Trace、Debug、Information、Warning、Error 和 Critical。

+   我们可以配置日志系统，使其仅根据每个条目的严重级别记录某些消息。

+   结构化日志可以帮助在日志中保持一致性，并简化搜索。

+   .NET 中的日志系统是基于提供者的，这允许我们自定义默认提供者。

+   我们可以使用`ILogger`、`ILogger<T>`或`ILoggerFactory`等接口来创建日志条目。

本章以 ASP.NET Core 为中心，结束了本书的第二部分。在接下来的几章中，我们将探讨设计模式，以创建灵活且健壮的组件。

## 问题

让我们看看几个练习题：

1.  我们能否同时将日志条目写入控制台和文件？

1.  在生产环境中，我们应该记录跟踪和调试级别的日志条目吗？

1.  结构化日志的目的是什么？

1.  我们如何在.NET 中创建日志条目？

## 进一步阅读

这里有一个链接，可以基于我们在本章中学到的内容进行构建：

+   [官方文档] *在.NET Core 和 ASP.NET Core 中的日志记录*: [`adpg.link/MUVG`](https://adpg.link/MUVG)

## 答案

1.  是的，你可以配置你想要的任何数量的提供者。一个可以用于控制台，另一个可以将条目追加到文件中。

1.  不，在生产环境中你不应该记录跟踪级别的条目。当你调试问题时，你应该只记录调试级别的条目。

1.  结构化日志在所有日志条目中保持一致的结构，这使得搜索和分析日志更加容易。

1.  我们可以使用`ILogger`、`ILogger<T>`和`ILoggerFactory`等接口来创建日志条目。
