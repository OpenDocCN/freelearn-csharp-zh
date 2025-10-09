# 12

# 提高性能的缓存策略

经常提到最小 API 应该是真正的最小化。在很大程度上，这种简约主义是基于最小化空间占用——尽量使我们的代码在页面上的可见足迹尽可能小。但 API 的简约主义也扩展到了资源占用，这意味着，在可能的情况下，我们应该最小化过度使用数据库/网络连接和 CPU 对系统造成的压力。

通过简约主义提高 API 的性能是我们的目标，这可以通过缓存部分实现。

当数据被缓存时，它将按照首次使用后的顺序存储，以便在未来的操作中重复使用。通过这样做，我们可以减少获取该数据时产生的延迟或开销。

在本章中，我们将涵盖以下主要主题：

+   最小 API 中的缓存介绍

+   内存缓存技术

+   分布式缓存策略

+   响应缓存

# 技术要求

要运行本章中的代码，需要 Visual Studio 2022 或 Visual Studio Code。您还需要在系统上安装 SQL Server 2022，并配置一个可以查询的工作数据库作为示例。建议您在阅读本章之前完成*第九章*，以便您可以使用配置好的示例员工数据库。

本章的代码可在 GitHub 仓库中找到：[`github.com/PacktPublishing/Minimal-APIs-in-ASP.NET-9`](https://github.com/PacktPublishing/Minimal-APIs-in-ASP.NET-9)。

本章将介绍需要内存缓存提供程序的分布式缓存策略——在本例中，是 Redis。安装 Redis 不在本书的范围内，但有关如何在 Azure 中安装 Redis 或托管 Redis 的文档可以在[`learn.microsoft.com/en-us/azure/azure-cache-for-redis/quickstart-create-redis`](https://learn.microsoft.com/en-us/azure/azure-cache-for-redis/quickstart-create-redis)和[`redis.io/docs/latest/operate/oss_and_stack/install/install-redis/`](https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/)找到。

在您的本地 Windows 机器上使用 Redis 的方法是通过**Windows Subsystem for Linux**（**WSL**）安装它，并在您的本地 WSL 实例上托管它。有关安装 WSL 的更多信息，请参阅此处：[`learn.microsoft.com/en-us/windows/wsl/install`](https://learn.microsoft.com/en-us/windows/wsl/install)。

# 最小 API 中的缓存介绍

API 执行操作，而操作（通常）依赖于数据或状态。数据需要被检索或计算，因为它要么存在于*静态*（即，在数据库中或在远程文件位置），要么存在于*使用中*（即，尚未计算以生成其他数据）的数据。

无论我们如何看待它，检索数据都会产生开销，无论是直接检索还是作为计算结果检索。缓存旨在通过利用已从原始来源产生的数据或状态来减少这种开销。

可以争论说，现在的计算速度如此之快，以至于开销应该微乎其微，以至于不再需要缓存。然而，这将是极其不准确的。单独查看一个操作，例如从 SQL 数据库中检索记录，可能看起来非常快，但在规模上，缓存的益处变得更加明显。

让我们用一个实际例子来看看缓存如何有益。一家初创公司构建了一个系统，可以用来通过最小 API 向移动设备发送警报。他们必须确保请求可以通过调用客户端处理，因此他们需要在每次请求的请求头中发送一个 API 密钥以进行验证。

为了验证密钥，初创公司的开发者决定将密钥验证外包给一家管理密钥和要使用的加密算法的云公司——为此自己托管一个 API。初创公司按请求验证密钥收费。

在早期，验证密钥的成本相对不明显，因为它们的请求数量很少且分散。然而，随着他们的业务开始增长，请求的数量也随之增加。很快，他们收到了来自云合作伙伴的令人恐惧的账单，账单上显示了大量按请求收费的 API 验证。

缓存本可以用来减轻验证 API 密钥的成本。可以首先发起一个请求来验证密钥，然后将其结果缓存起来。从那时起，当收到使用该密钥的请求时，首先会检查缓存。如果缓存中有验证密钥的记录，则无需调用付费 API 进行验证。每个缓存的记录都有一个过期日期，这意味着可以通过再次调用付费 API 来刷新它。这大大减少了验证 API 密钥的财务影响。

我们已经确定缓存对性能、减少延迟和整体应用可扩展性有好处，但我们应该使用哪种类型的缓存？为了回答这个问题，我们将探讨在最小 API 开发中可用的三种关键缓存方法：内存缓存、分布式缓存和响应缓存。

# 内存缓存技术

在 ASP.NET Core 支持的多种缓存技术中，**内存缓存**可能是最简单的。这种类型的缓存将其内容存储在托管最小 API 的机器的内存中。

缓存的实现基于**IMemoryCache**，它包含在**Microsoft.Extensions.Caching.Memory**包中，该包通常默认包含在 ASP.NET Core 项目中。

就像其他核心服务一样，**IMemoryCache** 可以通过依赖注入来使用，因此我们可以很容易地在最小 API 的各个区域中按需注入它。

使用这种缓存类型，我们可以存储一个对象，这是我们的最小要求，但我们也可以非常容易地指定一个过期时间，这是一个最佳实践，因为定期回收缓存可以使其运行顺畅。

让我们探索一个最小 API 中的简单示例。我将使用来自 *第九章*（可在 GitHub 上找到）的 API 项目作为这个示例项目的基础。我们的目标是减轻与数据库通信时产生的延迟和开销。

在这个 API 中，我们有一个允许客户端通过特定 ID 获取员工的端点。API 将使用 Entity Framework 对数据库执行 SQL 查询，并将结果返回在请求响应中。

使用内存缓存，我们可以为这个操作添加一些优化逻辑。以下是我们要执行的步骤：

1.  按照要求运行操作，从数据库中获取数据。

1.  检查内存缓存以查看是否已缓存具有此 ID 的员工。

1.  如果没有，将检索到的员工添加到缓存中。

1.  在请求响应中返回员工。

1.  为同一员工（相同的 ID）创建一个请求。

1.  从缓存而不是数据库中获取员工。

1.  将缓存的员工返回给客户端。

在我们实现这个目标之前，我们需要在项目中引用 **IMemoryCache**。

首先，在 **Program.cs** 中将 **IMemoryCache** 添加到依赖注入容器中：

```cs
builder.Services.AddMemoryCache();
```

然后，你可以创建 **GET** 端点，注入这个 **IMemoryCache** 对象以及 **DapperService**：

```cs
app.MapGet(
    "/employees/{id}",
    async (int id,
           [FromServices] DapperService dapperService,
           IMemoryCache memoryCache) =>
{
});
```

现在你有了缓存，你可以添加从其中检索值的代码：

```cs
if(memoryCache.TryGetValue(id, out var result))
{
    return result;
}
```

通过首先进行检查，我们可以避免不必要的代码执行，并更快地将所需对象传递给客户端，同时避免通过 Dapper 调用数据库。

假设该项目不存在，我们将使用我们的原始逻辑，通过 **DapperService** 从数据库中查找 **Employee** 记录。然而，我们不会立即返回项目，而是首先将其添加到缓存中：

```cs
  var employee = await dapperService.GetEmployeeById(id);
  memoryCache.Set<Employee>(employee.Id, employee);
```

这效果很好，但理想情况下，我们不想让它永远留在缓存中。定期刷新缓存是个好主意，因为数据可能会变化，我们希望确保尽可能多地获取最新数据，同时平衡减少数据库事务的延迟。

我们可以通过对缓存的对象设置过期时间来达到这个平衡。这需要在检索 **Employee** 对象之后但在将其添加到缓存之前完成。例如，我们可以将其设置为 **30** 秒的过期时间：

```cs
var cacheEntryOptions = new MemoryCacheEntryOptions()
    .SetSlidingExpiration(TimeSpan.FromSeconds(30));
```

通过创建一个 **MemoryCacheEntryOptions** 实例，我们定义了一些缓存配置参数，当我们将新对象添加到缓存中时可以传递给缓存。更新 **cache.Set()** 方法以包含此参数：

```cs
memoryCache.Set<Employee>(
    employee.Id,
    employee,
    cacheEntryOptions);
```

你的端点现在应该看起来像这样：

```cs
            app.MapGet("/employees/{id}",
            async (int id,
                   [FromServices] DapperService
                   dapperService, IMemoryCache memoryCache)
                   =>
            {
                if(memoryCache.TryGetValue(id,
                    out var result))
                {
                    return result;
                }
                var employee = await
                    dapperService.GetEmployeeById(id);
                var cacheEntryOptions = new
                    MemoryCacheEntryOptions()
                        .SetSlidingExpiration(
                            TimeSpan.FromSeconds(30));
                memoryCache.Set<Employee>(
                    employee.Id,
                    employee,
                    cacheEntryOptions);
                return Results.Ok(employee);
            });
```

就这样！你已经成功地将缓存引入到你的最小 API 端点，使用了 **IMemoryCache**。

在启动 API 项目时，内存缓存很可能是默认的缓存策略，但如果你的系统采用率有所增长，那么可扩展性和高可用性将变得越来越重要的成功衡量标准。当考虑扩展时，可以使用可靠的缓存框架采用分布式缓存策略。让我们看看最著名的缓存技术之一，Redis。

# 分布式缓存策略

**分布式缓存策略**使用诸如 **IMemoryCache** 之类的在支持可扩展性的架构中的方法。与 **IMemoryCache** 相比，分布式缓存涉及 ASP.NET 应用程序（托管你的最小 API）和缓存提供程序之间的连接。

在此示例中，我将使用的缓存提供程序是 **Redis**。

Redis 是一个内存缓存提供程序，也可以用作内存数据库。它作为一个开源产品提供，可以安装在本地或云端。

为了演示目的，我在 Ubuntu 机器上安装了 Redis 作为基本服务。然后我更新了 Redis 配置，使其绑定到 **0.0.0.0**，监听默认端口 **6379**。这仅在你像我一样，Redis 服务运行在单独的机器上时才相关。

现在有 Redis 实例可用，我可以向 API 项目添加所需的 NuGet 包，以便与 Redis 作为缓存进行交互。

将 **NRedisStack** 包添加到项目中：

```cs
dotnet add package NRedisStack
```

我们仍然会在 **Program.cs** 中与缓存进行交互，因此在这里我们需要从 **NRedisStack** 引用命名空间：

```cs
using StackExchange.Redis;
```

现在，我们可以更新通过 **Id** 获取员工的端点，使用新的缓存，用 Redis 替换 **IMemoryCache** 实现。

我们首先创建 **ConfigurationOptions**，这可以在连接到 Redis 实例时作为参数传递：

```cs
ConfigurationOptions options = new ConfigurationOptions
{
    EndPoints = { { "REDIS IP", 6379 } },
};
ConnectionMultiplexer redis =
    ConnectionMultiplexer.Connect(options);
IDatabase db = redis.GetDatabase();
```

接下来，我们现在应该有一个可以由 **db** 变量引用的 Redis 连接。接下来，我们将添加来自 **IMemoryCache** 示例的等效缓存逻辑，我们首先检查具有键（在这种情况下是 **Employee** ID 的字符串）的缓存条目，如果存在则返回它，如果不存在，则从缓存返回 **Employee** 实例：

```cs
var employeeIdKey = id.ToString();
var cachedEmployee = db.StringGet(employeeIdKey);
if (cachedEmployee.HasValue)
{
    return Results.Ok(
       JsonSerializer.Deserialize<Employee>(cachedEmployee)
    );
}
```

当调用 **StringGet()** 从 Redis 获取相关条目时，如果它不存在，将返回一个对象，其中 **HasValues** 设置为 **false**。假设 Redis 缓存不包含我们正在寻找的 **Employee** 记录，我们将从数据库中获取它并在返回给客户端之前将其缓存：

```cs
var employee = await dapperService.GetEmployeeById(id);
db.StringSet(
    employeeIdKey,
    JsonSerializer.Serialize(employee));
```

请注意，Redis 本身不支持直接插入强类型 .NET 对象，因此当保存时，我们需要通过序列化将 **Employee** 对象转换为 JSON 字符串，并在检索时从 JSON 字符串反序列化回 **Employee** 对象：

你更新的 Redis 连接端点现在应该看起来像这样：

```cs
app.MapGet(
    "/employees/{id}",
    async (int id,
           [FromServices] DapperService dapperService) =>
{
    ConfigurationOptions options = new ConfigurationOptions
        {
          EndPoints = { { "192.168.2.8", 6379 } },
        };
        ConnectionMultiplexer redis =
            ConnectionMultiplexer.Connect(options);
        IDatabase db = redis.GetDatabase();
        var employeeIdKey = id.ToString();
        var cachedEmployee = db.StringGet(employeeIdKey);
        if (cachedEmployee.HasValue)
        {
            return Results.Ok(
                JsonSerializer.Deserialize<Employee>(
                    cachedEmployee));
        }
        var employee = await
            dapperService.GetEmployeeById(id);
        return Results.Ok(employee);
});
```

使用 Redis 等工具在独立托管环境中实现缓存，为我们的最小 API 引入了更多的灵活性。我鼓励你通过创建一个通用的服务来进一步扩展这个简单的示例，该服务可以促进 ASP.NET 和 Redis 缓存之间的交互，从而最终将 API 与其缓存系统解耦。在未来，如果你希望从 Redis 迁移到不同的缓存技术，你需要能够做到这一点而不影响原始 API 代码。

我们已经介绍了两种缓存策略的示例。让我们以第三种技术结束，重点关注请求响应的缓存。

# 响应缓存

**响应缓存**与前面两种缓存策略的逻辑原理相同，但不是在内存中缓存数据库对象，而是在 HTTP 级别缓存响应。

与**IMemoryCache**一样，最小 API 可以通过在**Program.cs**中启用响应缓存作为功能来利用 ASP.NET 的本地中间件：

```cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddResponseCaching();
var app = builder.Build();
app.UseResponseCaching();
```

一旦启用，响应缓存添加到**GET**端点非常简单。我们可以将**HttpContext**添加到参数中，然后，每当我们有**Employee**对象并准备返回它时，我们可以设置响应以缓存一定的时间，这意味着在指定时间内请求相同的数据将直接返回缓存的 HTTP 响应，而不是接触数据库：

```cs
    app.MapGet(
        "/employees/{id}",
        async (int id,
               [FromServices] DapperService dapperService,
               HttpContext context) =>
    {
        var employee = await
            dapperService.GetEmployeeById(id);
        context.Response.GetTypedHeaders().CacheControl =
            new Microsoft.Net.Http.Headers
                .CacheControlHeaderValue()
    {
        Public = true,
        MaxAge = TimeSpan.FromSeconds(60)
    };
        context.Response.Headers[Microsoft.Net.Http.Headers
            .HeaderNames.Vary] =
                new string[] { "Accept-Encoding" };
        return Results.Ok(employee);
});
```

如你所见，这是一种非常直接的方式来缓存频繁的响应，并且过期时间当然可以根据需要进行调整。你甚至可以将缓存方法结合起来，有一个内存缓存来检索数据，然后缓存响应。

在手头有三个最小 API 中的缓存工作示例后，让我们回顾本章学到的内容。

# 摘要

在本章中，我们探讨了使用三种不同的策略进行缓存：ASP.NET 内存、分布式和响应缓存。

我们首先定义了缓存作为一个概念，将其与最小 API 的上下文相关联，然后探讨了公司希望通过缓存来节省通过 API 检索数据成本的假设场景。

在此之后，我们探讨了 ASP.NET 原生内存缓存方法，了解了**IMemoryCache**以及如何在端点中实现它以限制数据库事务产生的开销。我们还学习了如何使缓存数据过期。

然后，我们将这些知识扩展，遵循类似缓存原则，在分布式缓存中以 Redis 的形式进行。

最后，我们回顾了一个响应缓存的示例，使我们能够通过重新发送之前发送的 HTTP 请求来绕过数据库，处理频繁发送的请求。

在下一章中，我们将探讨你可以观察到的最佳实践，以增加你最小 API 的可读性、可扩展性和可维护性。

# 第四部分 - 最佳实践、设计和部署

在最后一部分，我们将关注点转移到稳健的 API 设计和部署原则。您将了解如何将生产就绪的最小 API 投入使用的最佳实践，以及在不同环境中进行测试和维护兼容性的策略。

本部分包含以下章节：

+   *第十三章* ，*最小 API 弹性的最佳实践*

+   *第十四章* ，*最小 API 的单元测试、兼容性和部署*
