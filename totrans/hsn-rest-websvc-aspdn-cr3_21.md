# 日志和健康检查

构建网络服务的一个重要方面是日志和健康检查。如今，人们更倾向于拥有范围较小的服务。这种方法提供了巨大的好处，但它使得我们难以验证服务是否以正确的方式运行。日志帮助我们跟踪网络服务另一侧的操作和处理的信息，而健康检查机制提供了一种验证服务是否健康以及所有必需的依赖是否满足的方法。本章将介绍 ASP.NET Core 的一些日志部分，并展示如何使用框架提供的工具实现健康检查。

在本章中，我们将涵盖以下主题：

+   在 ASP.NET Core 中登录

+   实现日志

+   实现日志提供者

+   在测试中实现自定义日志

+   Web 服务健康检查

# 在 ASP.NET Core 中登录

我们将从这个章节开始，提供 ASP.NET Core 不同日志组件的概述。该框架提供了不同的接口，支持日志记录：

+   `ILoggerProvider` 用于定义与输出通道绑定的一种特定类型的日志

+   `ILoggerFactory` 接收一个 `ILoggerProvider` 接口并对其进行初始化

+   `ILogger` 接口是日志组件的一个特定实例

ASP.NET Core 的日志接口结构可以用以下架构来描述：

![图片](img/86e0d6bc-3ac4-474f-bda7-eb95785ea19f.png)

简而言之，`ILoggerProvider` 接口代表日志的输出，`ILoggerFactory` 创建正确的实例类型，而 `ILogger` 是实际的日志实例。

这种方法保证了 `ILogger` 接口消费者和日志提供者部分之间的安全隔离。此外，我们可以在开发阶段选择添加对 `ILogger` 接口的调用；然后在发布阶段，我们可以决定使用哪个提供者来输出我们的日志数据。此外，我们的日志代码变得非常灵活，我们可以根据服务运行的环境类型来更改输出。让我们通过了解 ASP.NET Core 日志系统的不同特性来继续前进。

# 日志的关键特性

ASP.NET Core 日志系统以一些始终存在于每个日志记录中的关键属性为特征。此外，保持日志尽可能一致是非常重要的。ASP.NET Core 日志系统的关键组件是 *日志类别*、*日志级别*、*日志事件 ID* 和 *日志消息*。

当初始化 `ILogger<T>` 接口时，会指定 *日志类别*。它是日志过程的一个重要部分，因为它标识了发出日志记录的组件。*日志类别*通常对应于触发日志记录的类型或类。让我们以目录服务的 `ItemController` 类为例。*日志类别*是在 `ILogger` 接口注入过程中定义的：

[PRE0]

`ItemController` 类使用构造函数注入的广泛技术来初始化 `ILogger<ItemController>` 类型。因此，日志类别在 `_logger` 属性的签名中被隐式定义。尽管 *日志类别* 帮助我们识别哪个组件正在触发特定的日志消息，但我们还需要定义该消息的重要性。

*日志级别* 提供了指示日志记录严重性或重要性的信息。ASP.NET Core 提供了一个有用的扩展方法，它对日志级别提供了一些抽象：

[PRE1]

这些方法都是 `Microsoft.Extensions.Logging` 命名空间提供的抽象。例如，如果我们检查 `_logger.LogInformation` 方法的实现，底层实际上只是调用了通用的 `logger.Log` 方法：

[PRE2]

`LogInformation` 扩展方法通过隐式定义框架提供的信息级别来包装 `logger.Log` 方法调用。`LogLevel` 属性是一个由 `Microsoft.Extension.Logging` 命名空间暴露的 `enum` 结构，它提供了以下开箱即用的日志级别：

[PRE3]

上述代码描述了由 `LogLevel` 枚举提供的不同日志级别。日志级别从描述系统详细信息的 `Trace` 级别，到表示服务无法正确工作并正在关闭的 `Critical` 级别。`LogLevel` 属性是必不可少的，因为它通常用于过滤日志消息，告诉我们日志消息的优先级。

一旦我们确定了特定日志消息的严重性级别，我们需要提供 *日志事件 ID*，这有助于我们在日志输出中识别特定事件。虽然 *日志类别* 通常代表类的完整路径，但 *日志事件 ID* 在我们希望表达当前生成日志输出的方法时非常有用。让我们以 `ItemController` 类中包含的操作方法（`Get`、`GetById`、`Create`、`Update` 和 `Delete`）为例。我们可以创建一个将每个操作方法映射到特定事件 ID 的日志事件类：

[PRE4]

因此，当我们调用 `ItemController` 类的操作方法中的 `ILogger` 接口时，可以传递相应的日志事件 ID。通过这样做，我们可以识别和分组事件：

[PRE5]

最后，日志记录的另一个重要部分是与日志记录相关的消息。ASP.NET Core 的日志系统还提供了一个类似于 C# 字符串插值的模板系统。因此，可以使用以下方式使用模板系统：

[PRE6]

上述代码使用`LoggingEvents.GetById`事件ID记录了一条关于*信息*严重性的消息，并添加了消息`"GetById {id} "`。现在我们已经了解了ASP.NET Core提供的不同日志特性，我们将查看一个具体的应用实例，该实例已被应用于*目录服务*项目。

# 实现日志部分

在本节中，我们将学习如何在*目录网络服务*中执行日志记录。让我们首先选择一个我们将执行日志语句的层。由于逻辑封装在`Catalog.Domain`层项目中，我们将继续在项目中定义的服务类上实现日志部分。首先，让我们定义一个新的日志类，其中包含每个操作的相应*事件ID*：

[PRE7]

一旦为每个活动建立了相应的*日志事件ID*，我们需要定义`ILogger`接口将使用的日志消息。目前，我们可以确定以下消息：

[PRE8]

第一个指的是受变更影响记录的数量，而第二个是指被更改的实体。最后，第三个为处理器的目标实体提供一条消息。重要的是要注意，这两个常量遵循一个命名约定：它们名称的第一部分指的是消息的内容；在第一个下划线之后，我们有在日志模板系统中将被值替换的参数。

此外，我们可以通过更改`IItemService`方法并通过实现日志记录来继续进行。让我们从`AddItemAsync`方法开始：

[PRE9]

上述代码跟踪受影响记录的数量和添加记录的ID信息。我们也可以用`ItemService`类的其他方法做同样的事情。在只读操作的情况下，我们可以添加目标记录的ID；例如，在`GetItemAsync`方法的情况下：

[PRE10]

上述代码使用`Event.GetById`字段记录了服务检索到的ID相关的信息。它使用`Messages`类型来指定事件消息。在下一节中，我们将学习如何通过增强日志记录的异常处理实现来记录异常。

# 异常日志记录

如果服务的一部分抛出异常怎么办？处理异常是服务开发过程中的关键部分。正如我们在[第7章](13fd7d18-3ebe-4f60-89ff-4666d1c9671a.xhtml)“过滤器管道”中看到的，可以使用它们通过过滤器来捕获异常。过滤器是MVC堆栈的关键部分：它们在操作方法之前和之后执行，并且可以用于在单个实现中记录异常。让我们再次看看`Catalog.API`中的`JsonExceptionAttribute`：

[PRE11]

该类通过类似模式跟踪和返回异常，这可以在处理器的实现中看到：`ILogger<T>` 接口通过依赖注入注入到构造函数中，并使用 `_logger.LogError` 方法。

在下一节中，我们将学习如何使用 `Moq` 在我们的测试中验证日志记录。

# 使用 Moq 验证日志记录

让我们学习如何验证我们实现的日志记录。`ILogger` 接口的依赖注入系统帮助我们模拟日志机制并验证结果实现。重要的是要注意，我们的处理器正在使用 `ILogger` 接口的扩展方法。以下以 `Microsoft.Extensions.Logging` 命名空间中 `LogInformation` 扩展方法的 ASP.NET Core 实现为例：

[PRE12]

扩展方法**不是**面向模拟的结构。它们是静态方法，根据定义，在 C# 的运行时中不可能模拟静态结构。因此，我们需要提供一个对 `ILogger` 工厂进行抽象的机制，允许我们注入和模拟测试中使用的接口。

通常情况下，无法测试静态方法。此外，模拟库通常通过在运行时动态创建类来创建模拟。通常，它们通过扩展类型来覆盖类型的行为。由于静态方法不能被覆盖，因此无法模拟它们。因此，当需要模拟一个静态元素时，建议我们抽象它们并将它们封装到类中。

让我们看看 `Catalog.Fixture` 项目中 `LoggerAbstraction` 类的声明：

[PRE13]

`LoggerAbstraction` 是一个实现 `ILogger<T>` 接口的泛型类。更具体地说，抽象类通过执行 `void Log` 方法的重载版本来定义 `Log<TState>` 方法。`Log` 方法和 `LoggerAbstraction` 类都是抽象的，这意味着我们可以模拟它们的行为。因此，可以模拟日志记录类的行为，如下面修改后的 `ItemServiceTests` 类所示：

[PRE14]

`ItemServiceTests` 类将 `LoggerAbstraction<IItemService>` 类型初始化为一个类属性。类的构造函数使用 `ITestOutputHelper` 模拟服务层使用的日志系统：

[PRE15]

`ITestOutputHelper` 是由 `Xunit.Abstractions` 命名空间公开并由 `Xunit` 运行时解析的一个接口。它允许我们在测试运行器的测试控制台中编写代码。最后，测试类实现了 `additem_should_log_information` 测试方法。测试方法调用我们在 `ItemService` 类中实现的 `AddItemAsync` 方法。最后，可以使用以下代码片段验证 `ILogger` 接口：

[PRE16]

前面的代码片段验证了 `Log` 方法被调用了两次。它还将结果日志作为由 `ITestOutputHelper` 接口定义的日志系统的一部分输出。现在我们已经实现了 `Catalog.Domain` 项目服务层的日志机制，我们将检查和实现必要的日志提供者。

# 实现日志提供者

到目前为止，我们已经定义了在应用程序中要记录的内容；在本节中，我们将说明如何实现这一点。在 ASP.NET Core 中，日志提供者是通过依赖注入初始化的。此外，根据环境或其他启动选项，我们可以有条件地初始化它。

ASP.NET Core 提供了一些内置的日志提供者，如下表所示：

| **提供者** | **命名空间** | **描述** |
| --- | --- | --- |
| `Console` | `Microsoft.Extensions.Logging.Console` | 将运行应用程序的控制台设置为日志的输出。 |
| `Debug` | `Microsoft.Extensions.Logging.Debug` | 当附加调试器时，在调试输出窗口中写入消息。 |
| `EventSource` | `Microsoft.Extensions.Logging.EventSource` | 将 Windows ETW 设置为主要日志的输出。 |
| `EventLog` | `Microsoft.Extensions.Logging.EventLog` | 将 Windows 事件日志设置为主要日志的输出。 |
| `TraceSource` | `Microsoft.Extensions.Logging.TraceSource` | 要使用此提供者，ASP.NET Core 需要在 .NET Framework 上运行。应用程序将使用由跟踪源提供的监听器。 |

重要的是要注意，所有日志提供者都是互补的。此外，我们可以添加许多提供者，以便在更多源中进行跟踪日志记录。我们可以在 `Startup` 类和 `Program` 类中配置提供者。让我们看看 `Catalog.API` 的 `Program` 类：

[PRE17]

在这里，`Program` 类使用 `WebHost.CreateDefaultBuilder` 来创建服务的 `WebHostBuilder` 实例。如果我们进一步查看该方法，我们会看到它使用以下语法来定义日志的提供者：

[PRE18]

正如您所看到的，`Program` 类默认定义了三个内置提供者。此外，它使用 `Configuration` 实例来传递在 `appsettings.json` 文件中描述的配置。

此外，我们还可以通过在 `Program` 类中显式调用 `ConfigureLogging` 扩展方法来覆盖默认提供者：

[PRE19]

如果我们想在应用程序中添加自定义日志提供者，前面的代码片段非常有用。ASP.NET Core 还为我们提供了一个方便的方法来在 `Startup` 类中配置日志提供者：在 `ConfigureServices` 中可以使用 `AddLogging` 扩展方法：

[PRE20]

前面的代码片段在`Startup`类的`ConfigureServices`执行时初始化日志服务。在`Startup`级别初始化日志提供程序对于我们需要根据环境或特定的配置标志初始化日志提供程序时非常有用。让我们通过学习如何实现自定义日志提供程序来继续前进。

# 在测试中实现自定义日志提供程序

如我们所见，ASP.NET Core的日志系统旨在提供最大的可扩展性。在本节中，我们将学习如何实现一个自定义日志提供程序，我们可以在测试中使用它。`Catalog.API.Tests`项目中的所有测试类都使用`InMemoryApplicationFactory<T>`来运行Web服务器并提供`HttpClient`来调用API。如您所注意到的，当其中一个测试失败时，测试不会返回显式的错误。例如，让我们检查`ItemControllerTests`类中的以下测试方法：

[PRE21]

如果由于任何原因调用返回错误，测试端将收到以下消息：

[PRE22]

在这里，我们不知道*为什么*API返回了`InternalServerError`。在这里，我们可以使用`Xunit`提供的`ITestOutputHelper`接口来创建一个新的日志提供程序，并在我们的测试中使用它。要在ASP.NET Core中声明一个日志记录器，我们需要以下结构和组件：

![图片](img/5dfecc86-0882-4390-82df-2e36edc66afb.png)

前面的架构描述了两个主要组件：`TestOutputLoggerProvider`类型和`TestOutputLogger`类型。`TestOutputLoggerProvider`类型的目的是管理日志实例的列表。`TestOutputLogger`类描述了实际的日志机制实现。让我们首先定义自定义的`ILogger`组件：

[PRE23]

`ITestOutputClass`实现了`ILogger`接口提供的方法。首先，它在构造函数中声明了一个`ITestOutputHelper`字段。然后，它通过调用`_output.WriteLine`方法在`Log`方法的具体实现中使用`_output`属性。该类还实现了`IsEnabled`方法来检查日志级别是否对应于`LogLevel.Error`字段。如果日志记录不这样做，`LogLevel.Error`将被写入控制台输出。为了完成此实现，我们需要一个日志提供程序来初始化自定义日志记录器。让我们继续创建另一个名为`TestOutputLoggerProvider`的类，该类扩展了`ILoggerProvider`接口：

[PRE24]

`TestOutputLoggerProvider`定义了一个`ConcurrentDictionary`，其中包含一个`string`和`TestOutputLogger`的键值对；它还接受`ITestOutputHelper`接口作为接口，该接口在`CreateLogger`方法中使用，以将日志记录器添加到日志管道中。通过这样做，我们可以将自定义日志记录器集成到我们的测试中。我们将使用在`Catalog.Fixtures`项目中实现的`InMemoryApplicationFactory<T>`类来添加自定义日志记录器，如下所示：

[PRE25]

前面的类声明了一个新的 `ITestOutputHelper` 属性类型，并定义了一个设置器。我们可以在 `ConfigureTestService` 类内部通过调用 `AddProvider` 扩展方法来添加我们的自定义日志记录器，创建 `TestOutputLoggerProvider` 的新实例。在做出这些更改后，我们可以通过将自定义日志记录器集成到 `ItemControllerTests` 类中继续操作：

[PRE26]

正如你所见，`ItemControllerTests` 在构造函数中通过设置注入的 `ITestOutputHelper` 接口来调用 `_factory.SetTestOutputHelper`。现在，每次测试抛出错误时，我们都会得到详细的错误信息。重要的是要注意，`ITestOutputHelper` 接口是在测试类中分配的，这意味着这是唯一可能通过依赖注入获取该接口的点。在下一节中，我们将学习如何实现与网络服务依赖相关的健康检查。

# 网络服务健康检查

另一个始终存在于网络服务中的关键特性是对依赖项的 *健康检查过程*。通常，健康检查过程由 CI/CD 管道用于检查服务在部署后是否健康，或者在监控仪表板中执行功能检查。健康检查通常通过调用 HTTP 路由来检测网络服务中是否存在任何正在进行的问题。

注意，这些服务暴露了健康检查路由。这些健康检查的监控是在一个独立且分开的应用中实现的。此外，这种做法使我们能够保持服务与监控逻辑的独立性。

ASP.NET Core 提供了一些开箱即用的实现，以帮助开发者将健康检查过程添加到他们的服务中。这些功能是通过一种 *面向中间件的方法* 实现的。此外，健康检查作为 HTTP 端点暴露，可以用来检查第三方服务、数据库和数据存储系统的状态。因此，我们可以测试服务的依赖项，以确认它们的可用性和功能。

健康检查功能是在 2018 年 8 月的 *ASP.NET Core 2.2.0-preview1* 版本中引入的。这个功能引入了 `IHealthCheck` 接口*.* 在未来，它将与 `Polly` 库集成，以提供更好的弹性，当用户执行日志检查时。

# 在数据库上实现健康检查

数据库通常是网络服务的主要依赖之一。因此，始终需要检查服务与数据库之间的连接。让我们先学习如何使用`AspNetCore.HealthChecks.SqlServer` ([https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/)) 和 `Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore` ([https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/)) 包来实现健康检查。我们将将这些更改应用到 *目录服务* 项目中。让我们首先将此包添加到 `Catalog.API` 项目中：

[PRE27]

`AspNetCore.HealthChecks.SqlServer` 包允许我们对 SQL Server 实例执行健康检查。让我们继续在 `Startup` 类中注册以下服务：

[PRE28]

如您所见，代码使用 `AddHealthChecks` 扩展方法配置了 *健康检查中间件*，该方法返回一个 `IHealthChecksBuilder` 接口；它调用由构建器提供的 `AddSqlServer` 扩展方法来绑定健康检查与 SQL Server 数据库。最后，通过调用 `UseHealthChecks` 方法并传入健康检查路由来添加中间件。如果我们可以在 `https://<hostname>:<port>/health` 路由上调用我们的服务，我们将根据与数据源的连接收到 `Healthy`/`Unhealthy` 响应。

# 实现自定义健康检查

ASP.NET Core 的健康检查功能不仅适用于我们服务的数据源；它还可以用于执行复杂和自定义的健康检查。框架提供了 `IHealthCheck` 接口，以便我们可以实现我们的健康检查。让我们在 `Catalog.API` 项目的 `HealthCheck` 文件夹中创建一个名为 `RedisCacheHealthCheck` 的新类：

[PRE29]

`RedisCacheHealthCheck` 类使用 `StackExchange.Redis` 包通过设置连接字符串中指定的Redis实例创建连接。这个类的核心部分是 `CheckHealthAsync` 方法；它根据Redis实例的响应时间返回 `HealthCheckResult.Healthy` 或 `HealthCheckResult.Unhealthy`。如果ping响应时间少于五秒，则表示实例是健康的；否则，它不是。该类是 ASP.NET Core 堆栈的一部分，并且可以使用框架的依赖注入引擎来解决依赖关系。

因此，可以通过向`Startup`类的`ConfigureServices`方法中添加以下代码片段来将类添加到健康检查堆栈中：

[PRE30]

这种方法将 SQL Server 连接的检查以及我们在 `RedisCacheHealthCheck` 类中实现的自定义检查添加到中间件管道中。如果两者都成功，则该服务将被归类为健康状态。此外，还可以使用 `docker-compose up --build` 命令运行目录容器，并通过调用 `http://<hostname:port>/health` 路由来验证目录网络服务的依赖项状态。

# 摘要

在本章中，我们学习了如何使用日志来跟踪我们服务的状态。我们还学习了如何自定义日志提供程序以及如何将其与 ASP.NET Core 应用程序集成。此外，我们还处理了 ASP.NET Core 的新健康检查功能，并学习了如何构建自定义健康检查。

现在，你已经知道如何在 ASP.NET Core 应用程序中实现日志记录并创建自定义日志提供程序。你将能够使用这些技能来跟踪你的服务暴露的数据并监控你的网络服务实例的状态。

在下一章中，我们将学习如何使用 Azure 将我们的网络服务迁移到云端。
