# 使用.NET Core 实现工作服务

.NET Core 的最新版本包括一种简单方便的方式来实现后台进程。此外，从版本 3.0 开始，可以使用工作服务内置模板创建新项目。.NET 工作服务适用于多种用例。此外，随着云计算技术和分布式系统的日益普及，也涉及到服务之间的事件驱动通信，这需要实现后台进程。本章将介绍 ASP.NET Core 提供的工作服务工具的一些概念和用例。我们还将探讨如何将 ASP.NET Core 的工作服务功能集成到消耗上一章中实现的`ItemSoldOut`事件队列中。

本章涵盖了以下主题：

+   工作服务简介

+   使用.NET Core 实现工作服务

+   在 Docker 上部署和运行工作服务

+   扩展背景服务类

到本章结束时，你将能够实现工作服务并使用 Docker 容器技术部署它。

# 工作服务介绍

.NET Core 工作服务在每次需要执行重复或后台运行的操作时都非常有用。更详细地说，它们可以在应用层中使用，以启用异步操作并处理基于事件的架构的事件。如果你每次需要发布或监听消息时都需要根据计划刷新数据，或者你的应用程序需要排队一个后台工作项，那么你可能需要使用工作服务。此外，使用工作服务，可以在同一服务器上运行多个后台任务，而不会消耗大量资源。

.NET Core 中工作服务的基础是`IHostedService`接口。内置的工作服务模板可以作为开始实现我们的工作服务项目的指南。更重要的是，`IHostedService`接口由一个`BackgroundService`类实现，这是我们实现工作服务时应扩展的基类。

# 理解工作服务生命周期

.NET Core 使用`BackgroundService`类的定义来识别工作服务。`BackgroundService`类公开了三个方法，这些方法代表了工作服务的生活周期阶段：

```cs
namespace Microsoft.Extensions.Hosting
{
    public abstract class BackgroundService : IHostedService, IDisposable
    {
        public virtual Task StartAsync(CancellationToken 
            cancellationToken);

        protected abstract Task ExecuteAsync(CancellationToken 
            stoppingToken);

        public virtual async Task StopAsync(CancellationToken 
            cancellationToken);
    }
}
```

以下代码是`BackgroundService`类的抽象实现。该类实现了`IHostedService`和`IDisposable`接口，并公开了以下方法：

+   `StartAsync`方法代表了工作生活周期的第一阶段。当宿主准备好运行工作服务时，会调用此方法。它接受一个`CancellationToken`类型参数，该参数可用于取消正在运行的任务。

+   `ExecuteAsync` 方法包含 `BackgroundService` 类的核心实现。该方法在 `IHostedService` 启动后调用，并返回一个表示 `Task` 状态的 `Task` 类型。

+   当托管应用程序优雅地停止时，会调用 `StopAsync` 方法。

本节提供了工作服务生命周期方法的概述。在下一节中，我们将看到 .NET Core 中可用于工作服务的托管模型。

# 托管模型

.NET Core 工作模板不过是一个普通的 .NET Core 应用程序。此外，我们还可以将工作模板作为普通的控制台应用程序运行。此外，工作模板还提供了托管 API，以便将工作作为始终运行的过程运行。在 Windows 系统中，可以使用 Windows 服务技术运行工作。在 Linux 系统中，工作通过 `systemd` 运行。

此外，.NET Core 提供了两个不同的 NuGet 包来指定工作的工作行为：`Microsoft.Extensions.Hosting.WindowsServices` 包和 `Microsoft.Extensions.Hosting.Systemd` 包，这两个包都在 NuGet 上可用。

`Microsoft.Extensions.Hosting.WindowsServices` 包提供了一个名为 `UseWindowsService()` 的扩展方法，该方法可以在 `Program` 类的 `Main` 方法中初始化托管之后应用。`UseWindowsService()` 方法设置 `WindowsServiceLifetime` 并使用 *Windows 事件日志* 作为默认的日志输出。

如果我们选择将我们的工作作为 `systemd` 服务托管，我们需要使用 `Microsoft.Extensions.Hosting.Systemd` NuGet 包。此包提供了 `UseSystemd()` 方法，它可以像 `UseWindowsService()` 一样应用；在这种情况下，我们的工作服务将使用 `SystemdLifetime` 类，并配置日志以符合 `systemd` 格式。

重要的是要注意，`UseWindowsService()` 和 `UseSystemd()` 方法仅在作为 Windows 服务和 `systemd` 服务运行的工作服务上执行。

一旦我们发现了托管工作服务的方法，我们就可以通过将这些概念应用到具体的健康检查过程中来继续操作。此外，我们还将看到如何在 Docker 容器上运行工作。

# 实现健康检查工作

以下部分重点介绍工作服务的实现。我们将涵盖的用例是一个通用 Web 服务的健康检查工作。此外，假设我们想要定期检查我们 Web 服务的健康状态。这种检查通常在部署管道的末尾执行，或者一旦服务部署，以验证 Web 服务是否正在运行。

# 项目结构概述

本节为您概述了.NET Core 模板系统提供的开箱即用的工作模板项目。我们将使用此类项目来实现一个健康检查工作。首先，让我们使用以下 CLI 命令创建一个新的项目：

```cs
dotnet new worker -n HealthCheckWorker
```

此命令创建了一个名为`HealthCheckWorker`的新文件夹，并创建了基本工作服务项目所需的所有文件。让我们看看之前执行的`dotnet new worker`模板命令创建的文件。

其次，我们可以运行`tree` CLI 命令（在 macOS X 和 Windows 上均可用），该命令显示了之前创建的项目文件夹结构：

```cs
.
├── Program.cs
├── Properties
│   └── launchSettings.json
├── Worker.cs
├── HealthCheckWorker.csproj
├── appsettings.Development.json
├── appsettings.json
├── bin
│   └── Debug
│       └── netcoreapp3.0
└── obj
  ├── Debug
      └── netcoreapp3.0
```

`Program.cs`文件是我们工作服务的入口点。与`webapi`模板一样，工作模板使用`Program.cs`文件通过调用`Host.CreateDefaultBuilder`和`ConfigureServices`方法来初始化和检索一个新的`IHostBuilder`实例。`Program.cs`文件中的`Main`静态方法使用`AddHostedService`扩展方法初始化一系列工作：

```cs
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace HealthCheckWorker
{
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureServices((hostContext, services) =>
                {
                    services.AddHostedService<Worker>();
                });
    }
}
```

如前所述，前面的代码片段使用`AddHostedService`初始化作为默认工作模板的一部分创建的`Worker`类。需要注意的是，在底层，`AddHostedService`使用`Singleton`生命周期初始化类。因此，在整个工作服务执行期间，我们将有一个工作实例。在下一节中，我们将深入探讨工作生命周期的执行。

另一个将工作项目与其他任何.NET Core 项目区分开来的主要特征是使用了`Microsoft.NET.Sdk.Worker` SDK。此外，我们还应该注意到，`*.csproj`文件仅引用了一个额外的 NuGet 包，该包提供了`Program`类和`Main`方法使用的宿主扩展方法：`Microsoft.Extensions.Hosting`。

下一步是创建一个新的类，该类代表`HealthCheckWorker`项目的配置：

```cs
namespace HealthCheckWorker
{
    public class HealthCheckSettings
    {
        public string Url { get; set; }
        public int IntervalMs { get; set; }
    }
}
```

`HealthCheckSettings`将包含两个属性：`Url`和`IntervalMs`属性。第一个属性包含健康检查地址的 HTTP URL，该 URL 在`appsettings.json`中指定。`IntervalMs`属性表示工作生命周期的频率（以毫秒为单位）。

此外，还可以使用.NET Core 的配置系统在`Program.cs`文件的`Main`方法执行时绑定我们的配置对象，方法如下：

```cs
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace HealthCheckWorker
{
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureServices((hostContext, services) =>
 {
 var healthCheckSettings = hostContext
 .Configuration
                        .GetSection("HealthCheckSettings") services.Configure<HealthCheckSettings>
                        (healthCheckSettings);
 services.AddHostedService<Worker>();
 });
    }
}
```

前面的代码使用`hostContext`检索.NET Core 提供的`Configuration`实例。默认情况下，`hostContext`将填充`appsettings.json`文件中编写的设置结构。此外，可以使用`GetSection`方法从我们的`appsettings.json`文件中检索特定的配置部分，并将其绑定到`HealthCheckSettings`类型。

之后，我们可以继续进行`Worker`类的具体实现。此外，我们现在能够使用.NET Core 的内置依赖注入来注入`HealthCheckSettings`类型实例。设置是通过`IOption`接口注入的：

```cs
public class Worker : BackgroundService
    {
        private readonly ILogger<Worker> _logger;
        private readonly HealthCheckSettings _settings;
        private HttpClient _client;

        public Worker(ILogger<Worker> logger, 
 IOptions<HealthCheckSettings> options)
        {
            _logger = logger;
            _settings = options.Value;
        }
...
```

前面的代码定义了`Worker`类的属性。正如所述，该类实现了一个使用构造函数注入技术初始化的`_settings`属性，我们还可以看到一个`HttpClient`属性，它将由`BackgroundService`类公开的`StartAsync`方法初始化，并在`ExecuteAsync`方法中使用：

```cs
public class Worker : BackgroundService
{
    ...

 public override Task StartAsync(CancellationToken cancellationToken)
 {
 _client = new HttpClient();
 return base.StartAsync(cancellationToken);
 }

 protected override async Task ExecuteAsync(CancellationToken 
    stoppingToken)
 {
 while (!stoppingToken.IsCancellationRequested)
 {
 var result = await _client.GetAsync(_settings.Url);

 if (result.IsSuccessStatusCode)
 _logger.LogInformation($"The web service is up. 
                HTTP {result.StatusCode}");
 await Task.Delay(_settings.IntervalMs, stoppingToken);
 }
 }

    ...
}
```

在客户端初始化之后，`ExecuteAsync`方法实现了一个`while`循环，该循环将持续到`stoppingToken`请求取消进程。循环的核心部分使用`HttpClient`的`GetAsync`方法检查健康检查路由是否返回 HTTP 状态消息。最后，代码调用`Task.Delay`并使用填充了`_settings`实例的`IntervalMs`属性。

作为最后一步，`Worker`类覆盖了由`BackgroundService`类公开的`StopAsync`方法：

```cs
public class Worker : BackgroundService
{
    ...

 public override Task StopAsync(CancellationToken cancellationToken)
 {
 _client.Dispose();
 return base.StopAsync(cancellationToken);
 }

}
```

`StopAsync`方法通过调用`Dispose()`方法执行`HttpClient`实例的处置，并调用基类`BackgroundService`的`StopAsync(cancellationToken)`。

我们应该注意，`StartAsync`和`StopAsync`方法总是使用`base`关键字调用父方法。此外，在我们的情况下，我们需要保持基类`BackgroundService`类的行为。

本章的下一部分将专注于在 Docker 容器上执行工作。我们将看到如何配置`Dockerfile`，以及部署和运行应用程序。

# 在 Docker 上运行工作服务

本节专注于.NET 工作模板的部署步骤。我们将通过在 Docker Linux 镜像上运行我们的服务来继续操作。正如我们在第十二章，“服务的容器化”中已经看到的，我们将使用 Docker 在容器中运行应用程序。

让我们从配置项目文件夹中的`Dockerfile`开始：

```cs
FROM mcr.microsoft.com/dotnet/core/runtime:3.0 AS base
WORKDIR /app

# Step 1 - Building the project
FROM mcr.microsoft.com/dotnet/core/sdk:3.0 AS build
WORKDIR /src
COPY ["HealthCheckWorker.csproj", "./"]
RUN dotnet restore "./HealthCheckWorker.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "HealthCheckWorker.csproj" -c Release -o /app/build

# Step 2 - Publish the project
FROM build AS publish
RUN dotnet publish "HealthCheckWorker.csproj" -c Release -o /app/publish

# Step 3 - Run the project
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "HealthCheckWorker.dll"]
```

前面的代码描述了我们的.NET Core 工作应用程序的容器构建和部署过程。`Dockerfile`指令可以分为五个步骤：

1.  第一步使用`dotnet/core/sdk` Docker 镜像执行项目的构建：

+   +   首先，它将工作目录设置为`/src`并将项目文件夹中的文件复制过来。

    +   其次，它执行了`dotnet restore`和`dotnet build`命令。

1.  第二步在`/app/publish`文件夹中使用`Release`配置运行`dotnet publish`命令。

1.  第三步使用`dotnet/core/runtime` Docker 镜像，通过`dotnet` CLI 命令运行之前执行过的`dotnet publish`的结果。

1.  可以使用以下命令构建 Docker 镜像：

```cs
docker build --rm -f "Dockerfile" -t healthcheckworker:latest 
```

1.  此外，我们可以使用以下命令运行之前构建的镜像：

```cs
docker run --rm -d healthcheckworker:latest
```

上述命令将运行 Docker 容器，从而运行工作服务进程。工作服务将通过应用节流（也在 `appsettings.json` 文件中指定），向项目配置的 URL 发送 HTTP `GET` 请求。

之前提到的 `Dockerfile` 使用了多阶段构建方法和其他一些技术来构建用于运行项目的 Docker 镜像。这些概念以及更多内容在 第十二章，*服务的容器化*中详细描述。

# 消费售罄事件

我们在 第十三章，*服务生态系统模式*中实现的售罄事件，提供了关于目录中不可用项目的信息。此外，我们可以通过使用本章中描述的 `BackgroundService` 类型功能来消费此事件。购物车服务将实现一个售罄处理程序，用于处理和从 Redis 实例中删除不可用的项目 ID。

# 创建售罄处理程序

首先，让我们在购物车服务中创建一个处理程序来管理产品的售罄状态。首先，我们应该通过执行以下命令将 `RabbitMQ.Client` 包添加到 `Cart.Domain` 项目中：

```cs
dotnet add ./src/Cart.Domain package RabbitMQ.Client
```

我们可以继续定义一个类来表示购物车服务在新的项目中使用的售罄事件。因此，我们将创建一个新的 `ItemSoldOutEvent` 类型，它代表一个售罄事件：

```cs
using MediatR;

namespace Cart.Domain.Events
{
    public class ItemSoldOutEvent : IRequest<Unit>
    {
        public string Id { get; set; }
    }
}
```

`ItemSoldOutEvent` 类型包含对售罄项目的 `Id` 的引用。该类型将用于将队列内容反序列化为强类型实例并通过 MediatR 实例发送消息。正如我们对目录服务的配置所做的那样，我们还需要在 `Cart.Infrastructure` 项目中有一个表示 RabbitMQ 配置的类似类：

```cs
namespace Cart.Infrastructure.Configurations
{
    public class EventBusSettings
    {
        public string HostName { get; set; }
        public string User { get; set; }
        public string Password { get; set; }
        public string EventQueue { get; set; }
    }
}
```

之前定义的 `EventBusSettings` 类型描述了 RabbitMQ 实例的 `HostName`、用户的 `User` 和 `Password` 以及用于推送消息的 `EventQueue` 名称。此外，我们可以通过在 `Cart.Domain` 项目中定义一个新的类来定义消费 `ItemSoldOutEvent` 事件的调解处理程序：

```cs
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Cart.Domain.Entities;
using Cart.Domain.Events;
using Cart.Domain.Repositories;
using MediatR;

namespace Cart.Domain.Handlers.Cart
{
    public class ItemSoldOutEventHandler : 
        IRequestHandler<ItemSoldOutEvent>
    {
        private readonly ICartRepository _cartRepository;

        public ItemSoldOutEventHandler(ICartRepository cartRepository)
        {
            _cartRepository = cartRepository;
        }

        public async Task<Unit> Handle(ItemSoldOutEvent @event, 
            CancellationToken cancellationToken)
        {
            var cartIds = _cartRepository.GetCarts().ToList();

            var tasks = cartIds.Select(async x =>
            {
                var cart = await _cartRepository.GetAsync(new Guid(x));
                await RemoveItemsInCart(@event.Id, cart);
            });

            await Task.WhenAll(tasks);

            return Unit.Value;
        }

        private async Task RemoveItemsInCart(string itemToRemove, 
            CartSession cartSessionSession)
        {
            if (string.IsNullOrEmpty(itemToRemove)) return;

            var toDelete = cartSessionSession?.Items?.Where(x => 
                x.CartItemId.ToString() ==           
                     itemToRemove).ToList();

            if (toDelete == null || toDelete.Count == 0) return;

            foreach (var item in toDelete) 
                cartSessionSession.Items?.Remove(item);

            await _cartRepository.AddOrUpdateAsync(cartSessionSession);
        }
    }
}
```

之前的 `ItemSoldOutEventHandler` 类实现了由 MediatR 包提供的 `IRequestHandler<T>` 接口。该接口包含 `Handle` 方法，这是处理程序的主要入口点。当购物服务接收到 `ItemSoldOutEvent` 类型的事件时，将执行一个新的 `ItemSoldOutEventHandler` 实例。`ItemSoldOutEventHandler` 依赖于 `ICartRepository`。此外，`RemoveItemsInCart` 方法检索我们存储库中所有存储的购物车，并在每次接收到 `ItemSoldOutEvent` 类型的消息时移除售罄的商品。

# 测试售罄过程

可以通过测试 `ItemSoldOutHandler` 来检查售罄过程。处理程序将使用与其他处理程序相同的测试方法进行测试：

```cs
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Cart.Domain.Events;
using Cart.Domain.Handlers.Cart;
using Cart.Fixtures;
using Shouldly;
using Xunit;

namespace Cart.Domain.Tests.Handlers.Events
{
    public class ItemSoldOutEventHandlerTests : IClassFixture<CartContextFactory>
    {
        private readonly CartContextFactory _contextFactory;

        public ItemSoldOutEventHandlerTests(CartContextFactory 
            cartContextFactory)
        {
            _contextFactory = cartContextFactory;
        }

        [Fact]
        public async Task should_not_remove_records_when*soldout* message_contains_not_existing_id()
        {
            var repository = _contextFactory.GetCartRepository();
            var itemSoldOutEventHandler = new 
                ItemSoldOutEventHandler(repository);
            var found = false;

            await itemSoldOutEventHandler.Handle(new ItemSoldOutEvent { 
                Id = Guid.NewGuid().ToString() }, 
                   CancellationToken.None);

            var cartsIds = repository.GetCarts();

            foreach (var cartId in cartsIds)
            {
                var cart = await repository.GetAsync(new Guid(cartId));
                found = cart.Items.Any(i => i.CartItemId.ToString() == 
                    "be05537d-5e80-45c1-bd8c-
                     aa21c0f1251e");
            }

            found.ShouldBeTrue();
        }

        ...
    }
}
```

`ItemSoldOutEventHandlerTests` 类使用 `CartContextFactory` 类在测试方法的每个测试中初始化一个新的存储库，使用 `_contextFactory.GetCartRepository();` 方法。此外，`should_not_remove_records_when_soldout_message_contains_not_existing_id` 测试方法检查如果 `ItemSoldOutEvent` 实例具有不存在的 ID，则不会出现任何问题。另一方面，`should_remove_records_when_soldout_messages_contains_existing_ids` 测试方法检查当 `ItemSoldOutEvent` 实例包含现有 ID 时，`ItemSoldOutEventHandler` 是否会删除篮子中的商品：

```cs
...
        [Fact]
        public async Task should_remove_records_when*soldout* messages_contains_existing_ids()
        {
            var itemSoldOutId = "be05537d-5e80-45c1-bd8c-aa21c0f1251e";
            var repository = _contextFactory.GetCartRepository();
            var itemSoldOutEventHandler = new 
                ItemSoldOutEventHandler(repository);
            var found = false;

            await itemSoldOutEventHandler.Handle(new ItemSoldOutEvent { 
              Id = itemSoldOutId }, 
                CancellationToken.None);

            foreach (var cartId in repository.GetCarts())
            {
                var cart = await repository.GetAsync(new Guid(cartId));
                found = cart.Items.Any(i => i.CartItemId.ToString() == 
                    itemSoldOutId);
            }

            found.ShouldBeFalse();
        }
    }
}
```

第二个测试方法提供了一个现有的商品 ID，并通过检查过程是否有效地移除了商品来验证处理方法。现在我们已经验证了 `ItemSoldOutHandler` 的行为，我们可以继续配置和注册事件总线实例。

# 配置后台服务

现在我们已经定义了事件总线抽象，我们可以创建一个新的 `BackgroundService` 类型，该类型将使用 `IMediator` 接口来分发售罄消息。本书使用 RabbitMQ，因为它开源且易于配置，但请记住，与此主题相关的产品和技术有成千上万种。在继续实现后台服务之前，有必要向 `Cart.Infrastructure` 项目添加一些包：

```cs
dotnet add package RabbitMQ.Client
```

此外，我们可以通过创建一个新的类来扩展 `BackgroundService` 类型：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
using Cart.Domain.Events;
using Cart.Infrastructure.Configurations;
using MediatR;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;
using Newtonsoft.Json;
using RabbitMQ.Client;
using RabbitMQ.Client.Events;

namespace Cart.Infrastructure.BackgroundServices
{
    public class ItemSoldOutBackgroundService : BackgroundService
    {
        private readonly IMediator _mediator;
        private readonly ILogger<ItemSoldOutBackgroundService> _logger;
        private readonly EventBusSettings _options;
        private readonly IModel _channel;

        public ItemSoldOutBackgroundService( IMediator mediator,
            EventBusSettings options, ConnectionFactory factory, 
                ILogger<ItemSoldOutBackgroundService> logger)
        {
            _options = options;
            _logger = logger;
            _mediator = mediator;

            try
            {
                var connection = factory.CreateConnection();
 _channel = connection.CreateModel();
            }
            catch (Exception e)
            {
                _logger.LogWarning("Unable to initialize the event bus: 
                    {message}", e.Message);
            }
        }

        ...
    }
}
```

前面的代码片段定义了 `ItemSoldOutBackgroundService` 类型。该类扩展了由 `Microsoft.Extensions.Hosting` 命名空间公开的 `BackgroundService` 基类。构造函数注入 `IMediator` 接口以将收集的事件分发给 `ItemSoldOutEventHandler` 类型。此外，它还定义了将由构造函数中注入的 `ConnectionFactory` 类型填充的 `IModel` 类型的属性。`_channel` 属性将由 `BackgroundService` 类提供的 `ExecuteAsync` 方法使用，以分发事件。让我们通过重写 `ExecuteAsync` 方法来继续：

```cs
using MediatR;
using Microsoft.Extensions.Hosting;
using Newtonsoft.Json;
using RabbitMQ.Client;
using RabbitMQ.Client.Events;

namespace Cart.Infrastructure.BackgroundServices
{
    public class ItemSoldOutBackgroundService : BackgroundService
    {
         ...

        protected override Task ExecuteAsync(CancellationToken 
            stoppingToken)
        {
            stoppingToken.ThrowIfCancellationRequested();

            var consumer = new EventingBasicConsumer(_channel);

            consumer.Received += async (ch, ea) =>
            {
                var content = System.Text.Encoding.UTF8.
                    GetString(ea.Body);
                var @event = JsonConvert.DeserializeObject
                    <ItemSoldOutEvent>(content);

                await _mediator.Send(@event, stoppingToken);
                _channel.BasicAck(ea.DeliveryTag, false);
            };

            try
            {
                consumer.Model.QueueDeclare(_settings.EventQueue, true, 
                    false);                                         
                _channel.BasicConsume(_options.EventQueue, false, 
                    consumer);
            }
            catch (Exception e)
            {
                _logger.LogWarning("Unable to consume the event bus: 
                    {message}", e.Message);
            }

            return Task.CompletedTask;
        }
    }
}
```

前面的代码片段使用 `_channel` 属性初始化一个新的 `EventingBasicConsumer` 实例。对于每个接收到的消息，它将 `Body` 属性反序列化为 `ItemSoldOutEvent` 类型，并使用 `Send` 方法将事件发送到 `IMediator` 实例。最后，它通过使用 `EventBusSettings` 类型提供的 `EventQueue` 名称激活消费过程。此外，在这种情况下，消费过程被 try-catch 包装，以便在发生失败时隔离过程。

在我们能够使用 RabbitMQ 之前，有必要配置客户端以便我们连接到正确的消息总线实例。让我们首先在 `Cart.Infrastructure` 项目中创建一个新的扩展方法，这个方法可以在 `Extensions` 文件夹下找到：

```cs
using Cart.Infrastructure.Configurations;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using RabbitMQ.Client;

namespace Cart.Infrastructure.Extensions
{
    public static class EventsExtensions
    {
        public static IServiceCollection AddEventBus(this 
            IServiceCollection services, IConfiguration    
         configuration)
        {
            var config = new EventBusSettings();
            configuration.Bind("EventBus", config);
            services.AddSingleton(config);

            ConnectionFactory factory = new ConnectionFactory
            {
                HostName = config.HostName,
                UserName = config.User,
                Password = config.Password
            };

            services.AddSingleton(factory);

            return services;
        }
    }
}
```

之前的定义实现了一个与 `IServiceCollection` 接口绑定的扩展方法，该接口由 ASP.NET Core 的依赖注入系统提供。它在 `Startup` 类中使用，以连接到 RabbitMQ。`AddEventBus` 方法通过传递 RabbitMQ 参数初始化 `ConnectionFactory` 类。

最后，我们可以通过向 `Startup` 类的 `ConfigureServices` 方法中添加以下扩展方法来激活后台服务：

```cs
using System;
...

namespace Cart.API
{
    public class Startup
    {

        public void ConfigureServices(IServiceCollection services)
        {
 services
                   ...
                .AddEventBus(Configuration)
                .AddHostedService<ItemSoldOutBackgroundService>();
        }

        ...
    }
}
```

`AddEventBus` 方法将销售一空事件消费过程所需的全部依赖项添加到依赖注入中。此外，`AddHostedService` 方法将 `ItemSoldOutBackgroundService` 注册为 `IHostedService` 类型。最后，为了使用消息总线，我们可以在 `Cart.API` 项目的 `appsettings.json` 文件中定义连接信息，如下所示：

```cs
{
...
  "EventBus": {
    "HostName": "catalog_esb",
    "User": "guest",
    "Password": "guest",
    "EventQueue": "ItemSoldOut"
  }
}

```

连接参数指向在目录服务中定义的 `docker-compose.yml` 文件中定义的 `catalog_esb` 实例。此外，`ItemSoldOutBackgroundService` 类将处理 `ItemSoldOut` 队列中的消息并触发移除商品。

# 摘要

本章描述了如何使用 .NET Core 工作模板实现持续运行的任务。.NET Core 工作者在分布式系统中非常有用，可以在不增加服务器负载的情况下执行所有异步和基于事件的计算。

与 Windows 服务和 `systemd` 的集成也提供了一种高效的方式来部署和运行工作。本章为您概述了 .NET Core 工作服务托管模型，并展示了如何使用 .NET Core 设置和实现工作服务以及如何在 Docker 上运行它。最后，我们看到了如何将 `BackgroundService` 类的功能与之前实现的销售一空消息集成相结合。

下一章将介绍 .NET Core 提供并实现的常见安全实践。
