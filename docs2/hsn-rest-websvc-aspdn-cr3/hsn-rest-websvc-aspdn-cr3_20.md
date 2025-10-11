# 缓存网络服务响应

在本章中，我们将探讨 ASP.NET Core 的缓存模式，以及框架提供的缓存策略和工具，以帮助开发者实现它们。缓存可能有助于避免在服务器上进行额外计算，并因此为网络服务的客户端检索最快的响应。此外，我们还将查看目录网络服务的具体缓存实现。

在本章中，我们将涵盖以下主题：

+   HTTP 缓存系统的介绍

+   在 ASP.NET Core 中实现响应缓存

+   实现分布式缓存

到本章结束时，你将了解缓存机制，以及使用`Redis`在 ASP.NET Core 中实现分布式缓存所需的知识。

# HTTP 缓存系统的介绍

缓存是网络服务开发的一个关键部分。网络服务缓存的主要目的是提高我们系统的性能并减少服务器的负载。此外，通过网络获取数据速度慢且成本高，因此有必要实现一个缓存系统来提高我们网络服务的响应速度，并避免不必要的额外计算。在本节中，我们将关注 HTTP 1.1 缓存规范中定义的一些特性。

这些缓存规范由网络服务器发送到客户端。任何拥有客户端的人都需要阅读缓存规范并做出相应的响应。通常，缓存响应有两个常见的用例：第一个是当网络服务公开非常*动态内容*时。在这种情况下，数据变化很大，*缓存时间*可以减少或完全避免。第二种情况是我们可能提供一些*静态内容*。在这种情况下，我们可以为不经常变化的内容设置较长的缓存时间。

# HTTP 缓存规范

HTTP 1.1 缓存规范([`tools.ietf.org/html/rfc7234`](https://tools.ietf.org/html/rfc7234))描述了缓存如何在 HTTP 中表现。与 HTTP 缓存相关的主要头是`Cache-Control`头。此头用于指定请求和响应中的指令。还必须注意，请求和响应中定义的`Cache-Control`指令是独立的。以下图显示了典型的请求-响应工作流程：

![图片](img/b151bd77-6d75-42b3-b1da-f0ef2be74433.png)

该模式描述了通用客户端与它们之间带有缓存层的服务器之间的交互。

首先，客户端从服务器请求一个资源，*缓存层*将其转发到服务器。服务器为客户端生成数据并发送响应；在这个时候，缓存服务器在*缓存层*中缓存了响应。因此，如果客户端再次调用相同的内容，请求将不会击中服务器，但缓存系统将提供缓存的响应。

必须注意，*缓存层*会在响应中添加一些头：

```cs
Cache-Control: max-age=100
Age:0
```

这两个都是与缓存指令相关的 HTTP 头。`Cache-Control`添加了`max-age`指令来指示内容的 freshness lifetime 等于 100。`Age`头指定了缓存内容的年龄。当`Age`达到`Cache-Control: max-age`值时，*缓存层*将请求转发到服务器以提供新鲜数据。这两个头的值都是以*秒*为单位的。

`Cache-Control`头可以用来指定缓存机制。默认情况下，可以通过指定`no-cache`指令来禁用缓存，即`Cache-Control: no-cache`。`Cache-Control`头的另一个关键方面是公共和私有指令，例如`Cache-Control: public,max-age=100`。`public`指令意味着缓存的响应也可以存储在共享缓存中，并且任何其他客户端都可以访问该信息。另一方面，当一个响应是私有的，这意味着它可能包含敏感信息，并且不能与其他客户端共享。

缓存规范还定义了`Vary`头。这种头用于指示哪些字段影响缓存变化。更具体地说，它用于决定是否可以使用缓存的响应而不是从原始服务器请求新鲜的一个：

```cs
Vary: *
Vary: User-Agent
```

在前述代码的第一行中，每个请求的变化都被视为一个单独且不可缓存的请求。在第二行中，请求被处理为不可缓存，但添加了`User-Agent`头。

`Expires`头与`max-age`指令具有相同的目的：提供缓存过期时间。它们之间唯一的不同之处在于`max-age`指令关注的是固定的日期时间。例如，我们可以设置以下值：

```cs
Expires: Wed, 21 Oct 2015 07:28:00 GMT
```

必须注意，`max-age`指令覆盖了`Expires`头。因此，如果它们在同一个响应中都存在，则忽略`Expires`头。在下一节中，我们将学习如何使用 ASP.NET Core 工具实现响应缓存。

# 实现 ASP.NET Core 中的响应缓存

ASP.NET Core 通过声明性方式管理我们 Web 服务的响应中的缓存指令。此外，它提供了一个可用于缓存目的的属性。属性实现也与 HTTP 1.1 缓存规范兼容，因此，使用 ASP.NET Core 的即用型实现来实现这些缓存标准变得很容易，我们不必担心每个请求的细节。我们可以使用 ASP.NET Core 公开的`[ResponseCache]`属性来指定缓存行为：

```cs
namespace Catalog.API.Controllers
{
    [Route("api/items")]
    [ApiController]
    [Authorize]
    public class ItemController : ControllerBase
    {
        private readonly IItemService _itemService;
        private readonly IEndpointInstance _messageEndpoint;

        public ItemController(IItemService itemService, 
        IEndpointInstance messageEndpoint)
        {
            _itemService = itemService;
            _messageEndpoint = messageEndpoint;
        }

        ...

        [HttpGet("{id:guid}")]
        [ItemExists]
 [ResponseCache(Duration = 100, VaryByQueryKeys = new []{"*"})]
        public async Task<IActionResult> GetById(Guid id)
        {
            var result = await _itemService.GetItemAsync(new 
                GetItemRequest { Id = id });
            return Ok(result);
        }
        ...
    }
}
```

例如，在这种情况下，代码在`ItemController`类的`GetById`操作方法上定义了一个缓存指令。`ResponseCache`属性设置了一个`Duration`字段和一个`VaryByQueryKeys`字段：第一个对应于`max-age`指令，而第二个反映了`Vary` HTTP 头。

到目前为止，我们只向服务器响应中添加了`Cache-Control`指令。因此，我们并没有实现任何缓存。缓存指令可以被第三方系统或应用程序使用，例如我们服务的客户端，以缓存信息。

除了`ResponseCache`属性之外，还需要在 Web 服务前放置缓存中间件。`ResponseCachingMiddleware`是 ASP.NET Core 提供的默认中间件。它符合 HTTP 1.1 缓存规范([`tools.ietf.org/html/rfc7234`](https://tools.ietf.org/html/rfc7234))。因此，如果我们考虑`ResponseCachingMiddleware`类型，可以按以下方式更改之前的架构：

![](img/8213c3cc-1f17-4636-8e25-0774296650a9.png)

可以使用`Microsoft.Extensions.DependencyInjection`包提供的`AddResponseCaching`扩展方法初始化`ResponseCachingMiddleware`类。此外，我们可以在`Startup`类的`Configure`方法中执行`UseResponseCaching`扩展方法，将`ResponseCachingMiddleware`中间件添加到中间件管道中。`ResponseCachingMiddleware`类型检查响应是否可缓存，并存储响应并从缓存中提供答案。我们可以通过编辑`Catalog.API`项目中的`Startup`类将`ResponseCachingMiddleware`添加到服务管道中：

```cs
public class Startup
    {
         ...
        public void ConfigureServices(IServiceCollection services)
        {
            services
                  ...
               .AddResponseCaching();
        }

        public void Configure(IApplicationBuilder app, 
        IHostingEnvironment env)
        {     
             ...
             app.UseHsts();
             app.UseMiddleware<ResponseTimeMiddlewareAsync>();
             app.UseHttpsRedirection();
             app.UseAuthentication();
             app.UseResponseCaching();
             app.UseEndpoints(endpoints =>
                    {
                        endpoints.MapControllers();
                    });
        }
  }
```

上述代码添加了缓存中间件，但仅此不足以初始化缓存机制。因此，如果我们尝试使用`curl`命令调用带有`ResponseCache`属性的路线，我们将收到以下响应：

```cs
curl --verbose -X GET http://localhost:5000/api/items/08164f57-1e80-4d2a-739a-08d6731ac140
Note: Unnecessary use of -X or --request, GET is already inferred.
* Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 5000 (#0)
> GET /api/items/08164f57-1e80-4d2a-739a-08d6731ac140 HTTP/1.1
> Host: localhost:5000
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Sat, 05 Jan 2019 16:55:21 GMT
< Content-Type: application/json; charset=utf-8
< Server: Kestrel
< Cache-Control: public,max-age=100
< Transfer-Encoding: chunked
< X-Response-Time-ms: 21
< 
* Connection #0 to host localhost left intact
```

如您所见，`Cache-Control`告诉我们这些信息可以在缓存中共享，并且`max-age`为 100 秒。因此，如果我们调用相同的路线在*N（N<100）*秒后，我们也可以看到包含对象在缓存中存在时间的`Age`头。

此外，如果我们调用相同的路由后 *N 秒*（其中 *N >= 100*），我们将访问服务器并在内存中缓存一个新的响应。还必须注意的是，我们可以通过在调用 URL 中附加查询字符串参数来取消缓存中间件。这是因为我们指定了以下字段使用 `Vary` 头：

```cs
 VaryByQueryKeys = new []{"*"}
```

可能的情况是，定义 `ResponseCache` 属性时使用了 `VaryByQueryKeys` 字段。在这种情况下，该属性将无法检测查询字符串的变化。此外，受 `ResponseCache` 属性覆盖的路线将检索以下异常：

`"ClassName":"System.InvalidOperationException","Message":"'VaryByQueryKeys' requires the response cache middleware."`

关于 `ResponseCachingMiddleware` 类的一个重要注意事项是它使用 `IMemoryCache` 接口来存储响应内容。因此，如果您在 GitHub 上检查该类的定义（[`github.com/aspnet/AspNetCore/.../ResponseCachingMiddleware.cs`](https://github.com/aspnet/AspNetCore/blob/master/src/Middleware/ResponseCaching/src/ResponseCachingMiddleware.cs)），您将看到以下构造函数：

```cs
...
       public ResponseCachingMiddleware(
            RequestDelegate next,
            IOptions<ResponseCachingOptions> options,
            ILoggerFactory loggerFactory,
            ObjectPoolProvider poolProvider)
            : this(
                next,
                options,
                loggerFactory,
                new ResponseCachingPolicyProvider(),
                new MemoryResponseCache(new MemoryCache(new 
                    MemoryCacheOptions
                {
                    SizeLimit = options.Value.SizeLimit
                })),
                new ResponseCachingKeyProvider(poolProvider, options))
        { }

    internal ResponseCachingMiddleware(
            RequestDelegate next,
            IOptions<ResponseCachingOptions> options,
            ILoggerFactory loggerFactory,
            IResponseCachingPolicyProvider policyProvider,
            IResponseCache cache,
            IResponseCachingKeyProvider keyProvider)
        {
....
```

前面的方法签名定义了 `ResposeCachingMiddleware` 类的构造函数。构造函数使用 `this` 关键字引用另一个内部构造函数重载，该重载用于初始化类的属性。如您所见，`IResponseCache` 接口默认使用 `MemoryCache` 类型初始化，该类型扩展了 `IMemoryCache` 接口。

`IMemoryCache` 接口表示存储在 web 服务器中的缓存。此外，您还可以通过向 `Startup` 类中服务的初始化添加 `AddMemoryCache()` 扩展方法，将 `IMemoryCache` 接口用作独立组件：

```cs
        public void ConfigureServices(IServiceCollection services)
        {
            services
                  ...
               .AddMemoryCache();
        }
```

这种方法允许您通过依赖注入引擎使用 `IMemoryCache` 接口，这意味着您可以通过调用 `GetOrCreate` 和 `GetOrCreateAsync` 方法在 web 服务器的内存中设置缓存值。尽管内存缓存为我们提供了很好的抽象来缓存数据，但它不适用于分布式方法。因此，如果您想在不同的 web 服务器之间存储和共享缓存，ASP.NET Core 提供了您实现分布式缓存所需的工具。在下一节中，我们将了解如何在 ASP.NET Core 中实现分布式缓存。

# 实现分布式缓存

正如我们之前提到的，ASP.NET Core 允许我们实现分布式缓存。在本节中，我们将学习如何将 `Redis` 作为缓存存储的形式使用。您可以通过在 `Catalog.API` 项目的初始化中添加并执行以下 CLI 指令来扩展 ASP.NET Core 的缓存机制：

```cs
 dotnet add package Microsoft.Extensions.Caching.Redis

```

通过这样做，我们可以连接 `Redis` 服务器，并通过向 `Startup` 类添加以下扩展方法来使用它：

```cs
public class Startup
    {
        ...
        public void ConfigureServices(IServiceCollection services)
        {
            ...
            services
               .AddDistributedRedisCache(
                       options => { options.Configuration = 
                       "catalog_cache:6380";
                });
               ...
        }
    }
}
```

微软提供了`AddDistributedRedisCache`扩展方法，该方法接受用于定义`Redis`实例的选项，以便它可以作为缓存输入使用。`AddDistributedRedisCache`扩展方法连接到`catalog_cache`的`Redis`实例。此外，我们还需要在目录服务的`docker-compose.yml`文件中声明新的实例。

```cs
    ...
    catalog_cache:
        container_name: catalog_cache
 image: redis:alpine
 networks:
            - my_network

networks:
    my_network:
        driver: bridge
```

前面的代码定义了目录缓存的`Redis`实例。因此，应用程序将能够将此实例作为`my_network`的一部分使用。此外，我们可以通过查看`AddDistributedRedisCache`扩展方法的源代码来了解该包的主要用途：

```cs
public static class RedisCacheServiceCollectionExtensions
{
  public static IServiceCollection AddDistributedRedisCache(
    this IServiceCollection services,
    Action<RedisCacheOptions> setupAction)
  {
    if (services == null)
      throw new ArgumentNullException(nameof (services));
    if (setupAction == null)
      throw new ArgumentNullException(nameof (setupAction));
    services.AddOptions();
    services.Configure<RedisCacheOptions>(setupAction);
    services.Add(ServiceDescriptor.Singleton<IDistributedCache, 
        RedisCache>());
    return services;
  }
}
```

为了简洁起见，我省略了一些文档注释，但您可以在以下 GitHub 链接中找到开源代码：[`github.com/aspnet/Extensions/../StackExchangeRedisCacheServiceCollectionExtensions.cs`](https://github.com/aspnet/Extensions/blob/master/src/Caching/StackExchangeRedis/src/StackExchangeRedisCacheServiceCollectionExtensions.cs)。

从前面的代码中可以看出，扩展方法配置并绑定`RedisCacheOptions`类。其次，前面的代码片段向 ASP.NET Core 的内置依赖注入中添加了一个新的`IDistributedCache`接口。这个接口为我们提供了一些与`Redis`实例交互和存储缓存信息的有用方式。此外，实例被定义为单例，这意味着应用程序中的所有组件都将使用相同的实例。

让我们看看`IDistributedCache`接口：

```cs
  public interface IDistributedCache
  {

    byte[] Get(string key);

    Task<byte[]> GetAsync(string key, CancellationToken token = 
    default (CancellationToken));

    void Set(string key, byte[] value, DistributedCacheEntryOptions 
    options);

    Task SetAsync(
      string key,
      byte[] value,
      DistributedCacheEntryOptions options,
      CancellationToken token = default (CancellationToken));

    void Refresh(string key);
    Task RefreshAsync(string key, CancellationToken token = 
    default (CancellationToken));
    Task RemoveAsync(string key, CancellationToken token = 
    default (CancellationToken));
  }
```

在这种情况下，为了简洁起见，我省略了文档注释。您可以在[`github.com/aspnet/Extensions/../IDistributedCache.cs`](https://github.com/aspnet/Extensions/blob/master/src/Caching/Abstractions/src/IDistributedCache.cs)找到完整的版本。该接口提供了一个实用方法，我们可以使用它将信息读入和写入`Redis`实例。

此外，该接口公开了同步和异步方法来完成这项工作。由于该接口是依赖注入引擎的一部分，我们可以在任何组件中使用它，例如`ItemController`类：

```cs
namespace Catalog.API.Controllers
{
    [Route("api/items")]
    [ApiController]
    [JsonException]
    [Authorize]
    public class ItemController : ControllerBase
    {
        private readonly IItemService _itemService;
        private readonly IEndpointInstance _messageEndpoint;
        private readonly IDistributedCache _distributedCache;

        public ItemController(IItemService itemService, 
        IEndpointInstance messageEndpoint, 
 IDistributedCache distributedCache)
        {
            _itemService = itemService;
            _messageEndpoint = messageEndpoint;
            _distributedCache = distributedCache;
        }

        ...
...
```

现在，`ItemController`类和应用程序中的任何其他组件都可以通过将其包含在构造函数和操作注入中，通过包含`IDistributedCache`单例实例来解析和使用。

ASP.NET Core 还提供了`AddDistributedMemoryCache()`方法，它是`Microsoft.Extensions.DependencyInjection`命名空间的一部分。尽管其名称如此，但它并不会初始化任何分布式缓存。让我们深入了解一下它的实现：

```cs
public static IServiceCollection AddDistributedMemoryCache(
  this IServiceCollection services)
{
  if (services == null)
    throw new ArgumentNullException(nameof (services));
  services.AddOptions();
  services.TryAdd(ServiceDescriptor.Singleton<IDistributedCache, MemoryDistributedCache>());
  return services;
}
```

扩展方法使用网络服务器的内存来存储信息。更具体地说，`AddDistributedMemoryCache`扩展方法是为开发/测试环境设计的，它不是一个真正的分布式缓存。

最后，我们可以通过创建一个专门用于缓存的新的配置类来优化和重构 `Startup` 类。首先，让我们在目录的域项目的 `Configurations` 文件夹中创建以下类：

```cs
namespace Catalog.Domain.Configurations
{
    public class CacheSettings
    {
        public string ConnectionString { get; set; }
    }
}
```

类定义包含 `Redis` 实例的 `ConnectionString` 字段。我们可以通过将 `DistributedCacheExtensions` 类添加到 `Catalog.Infrastructure` 项目中继续操作，该项目位于 `Extensions` 文件夹内：

```cs
using Catalog.Domain.Configurations;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Newtonsoft.Json;
namespace Catalog.Infrastructure.Extensions
{
    public static class DistributedCacheExtensions
    {
        public static IServiceCollection AddDistributedCache(this 
            IServiceCollection services,
            IConfiguration configuration)
        {
            var settings = configuration.GetSection("CacheSettings");
            var settingsTyped = settings.Get<CacheSettings>();
            services.Configure<CacheSettings>(settings);
            services.AddDistributedRedisCache(options => { 
            options.Configuration = settingsTyped.ConnectionString; });
            return services;
        }
    }
}
```

上述代码声明了一个新的 `AddDistributedCache` 扩展方法。该方法通过读取和注册节点到依赖注入引擎来配置 `appsettings.json` 文件的 `CacheSettings` 节点。接下来，它调用 `AddDistributedCache` 方法来配置分布式缓存，使其可以使用提供的连接字符串。

让我们从向 `appsettings.json` 文件添加新的 `CacheSettings` 节点开始：

```cs
...
"CacheSettings": {
  "ConnectionString": "catalog_cache"
}
```

接下来，我们需要在 `Startup` 类中调用 `AddDistributedCache` 扩展方法：

```cs
public class Startup
{
    ...
    public void ConfigureServices(IServiceCollection services)
    {
        services
           ...
            .AddDistributedCache(Configuration)
    }
```

现在，我们的应用程序使用 `appsettings.json` 文件提供的连接字符串在依赖注入引擎中初始化一个新的 `IDistributedCache` 实例。在下一节中，我们将检查分布式缓存的额外定制级别。

# 实现 `IDistributedCache` 接口

可以通过重载扩展方法来扩展 `IDistributedCache` 接口的行为。此外，我们需要存储复杂信息而不是字节。例如，在 `ItemController` 类的情况下，我们希望将 `ItemResponse` 类型传递给 `IDistributedCache` 接口的 `Set` 方法。

让我们从修改 `Catalog.API` 项目中的 `DistributedCacheExtensions` 类开始：

```cs
using System.Threading.Tasks;
using Catalog.Domain.Configurations;
using Microsoft.Extensions.Caching.Distributed;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Newtonsoft.Json;

namespace Catalog.Infrastructure.Extensions
{
    public static class DistributedCacheExtensions
    {
        private static readonly JsonSerializerSettings 
            _serializerSettings = CreateSettings();

        ...

        public static async Task<T> GetObjectAsync<T>(this 
            IDistributedCache cache, string key)
        {
            var json = await cache.GetStringAsync(key);

            return json == null ? default(T) :

            JsonConvert.DeserializeObject<T>(json, 
                _serializerSettings);
        }
        public static async Task SetObjectAsync(this IDistributedCache 
            cache, string key, object value)
        {
            var json = JsonConvert.SerializeObject(value, 
                _serializerSettings);
            await cache.SetStringAsync(key, json);
        }

        public static async Task SetObjectAsync(this IDistributedCache 
            cache, string key,
        object value, DistributedCacheEntryOptions options)
        {
            var json = JsonConvert.SerializeObject(value, 
                _serializerSettings);
            await cache.SetStringAsync(key, json, options);
        }

        private static JsonSerializerSettings CreateSettings()
        {
            return new JsonSerializerSettings();
        }
    }
}
```

上述类定义了获取和设置 `Redis` 缓存中复杂对象的泛型扩展方法。它使用 `Newtonsoft.Json` 方法以 JSON 格式存储对象。该类还定义了两个具有不同签名的 `SetObject<T>` 方法。一个提供了 `DistributedCacheEntryOptions` 的默认配置，而另一个允许我们覆盖此配置中的选项。此外，我们可以在 `ItemController` 类中使用这些扩展方法，如下所示：

```cs
namespace Catalog.API.Controllers
{
    [Route("api/items")]
    [ApiController]
    [JsonException]
    [Authorize]
    public class ItemController : ControllerBase
    {
        private readonly IItemService _itemService;
        private readonly IEndpointInstance _messageEndpoint;
        private readonly IDistributedCache _distributedCache;

        public ItemController(IItemService itemService, 
        IEndpointInstance messageEndpoint,
            IDistributedCache distributedCache)
        {
            _itemService = itemService;
            _messageEndpoint = messageEndpoint;
            _distributedCache = distributedCache;
        }
        [HttpGet("{id:guid}")]
        [ItemExists]
        [ResponseCache(Duration = 100, VaryByQueryKeys = new[] { "*" })]
        public async Task<IActionResult> GetById(Guid id)
        {
            var key = $"{typeof(ItemController).FullName}.
            {nameof(GetById)}.{id}"; 
            var cachedResult = await _distributedCache. 
                GetObjectAsync<ItemResponse>(key);

 if (cachedResult != null)
 {
 return Ok(cachedResult);
 }

            var result = await _itemService.GetItemAsync(new 
                GetItemRequest{ Id = id });
            await _distributedCache.SetObjectAsync(key, result);

            return Ok(result);
        }
...
```

`GetById` 动作方法使用 `IDistributedCache` 接口来保存 `ItemResponse` 的信息：它定义了一个以下方式组合的键：

```cs
<controller_full_name>.<action_method_name>.<input_id>
```

动作方法尝试从缓存中检索此信息；如果此信息不存在，它将使用 `IItemService` 接口执行请求，并使用 `IDistributedCache` 接口的 `SetObjectAsync` 方法存储结果。

与 `ResponseCache` 属性相比，这种方法的实现开销更大，但 `IDistributedCache` 接口依赖于外部缓存系统。此外，可以使用过滤器管道堆栈实现这种类型的缓存逻辑。因此，可以将 `ItemController` 类中实现的缓存逻辑移动到自定义动作过滤器中：

```cs

namespace Catalog.API.Filters
{
    public class RedisCacheFilter : IAsyncActionFilter
    {
        private readonly IDistributedCache _distributedCache;
        private readonly DistributedCacheEntryOptions _options;

        public RedisCacheFilter(IDistributedCache distributedCache, 
        int cacheTimeSeconds)
        {
            _distributedCache = distributedCache;
            _options = new DistributedCacheEntryOptions
            {
                SlidingExpiration = 
                    TimeSpan.FromSeconds(cacheTimeSeconds)
            };
        }

        public async Task OnActionExecutionAsync(ActionExecutingContext 
        context, ActionExecutionDelegate next)
        {
            if (!context.ActionArguments.ContainsKey("id"))
            {
                await next();
            }

            var actionName = (string) 
                context.RouteData.Values["action"];
            var controllerName = (string) 
                context.RouteData.Values["controller"];

            var id = context.ActionArguments["id"];

            var key = $"{controllerName}.{actionName}.{id}";

            var result = await _distributedCache.
                GetObjectAsync<ItemResponse>(key);

            if (result != null)
            {
                context.Result = new OkObjectResult(result);
                return;
            }

            var resultContext = await next();

            if (resultContext.Result is OkObjectResult resultResponse 
            && resultResponse.StatusCode == 200)
 {
 await _distributedCache.SetObjectAsync(key, 
                    resultResponse.Value, _options);
 }
        }
    }
}
```

`RedisCacheFilter` 封装了缓存逻辑，以避免在每个动作方法中重复实现。它实现了以下逻辑：在动作执行之前，它尝试通过组合控制器名称、动作名称和请求的 `id` 值来从缓存中获取 `ItemResponse`。如果缓存键未填充，它将继续执行动作方法，如果结果 `StatusCode` 是 `200`，它将继续将结果存储在 `Redis` 缓存中。下一个具有相同 `id` 值的请求将填充缓存，动作过滤器将返回缓存的对象。

可以以下方式使用动作过滤器：

```cs

    [Route("api/items")]
    [ApiController]
    [JsonException]
    public class ItemController : ControllerBase
    {
        ...

        [HttpGet("{id:guid}")]
        [ItemExists]
        [TypeFilter(typeof(RedisCacheFilter), Arguments = new object[] 
            {20})]
        public async Task<IActionResult> GetById(Guid id)
        {
            ...
        }

       ...
    }
```

动作方法使用 `TypeFilter` 属性解析 `RedisCacheFilter`，并将表示缓存过期时间的秒数作为参数传递。这种实现策略更易于阅读，并允许我们避免在不同动作方法之间复制代码。

# 将内存缓存注入到测试中

如果我们尝试使用 `RedisCacheFilter` 执行单元测试，我们会注意到 `get_by_id_should_return_right_data` 将会失败。这是因为 Redis 实例在开发环境中不可用。此外，我们可以使用 `MemoryDistributedCache` 实现来执行我们的测试。`MemoryDistributedCache` 实现是在 `AddDistributedMemoryCache` 扩展方法所使用的同一类中完成的。正如我们在上一章中提到的，该类不提供真正的分布式缓存，它用于测试目的。

因此，我们可以向包含在 `Catalog.Fixtures` 项目中的 `InMemoryApplicationFactory<TStartup>` 类添加以下行：

```cs
namespace Catalog.Fixtures
{
    public class InMemoryApplicationFactory<TStartup>
        : WebApplicationFactory<TStartup> where TStartup : class
    {
        protected override void ConfigureWebHost(IWebHostBuilder 
            builder)
        {
            builder.UseEnvironment("Testing")
                .ConfigureTestServices(services =>
                {
                    ...
                   services.AddSingleton<IDistributedCache, 
                      MemoryDistributedCache>();              
                    ...
                });
        }
    }
}

```

之前的代码将 `IDistributedCache` 的实现替换为 `MemoryDistributedCache` 的一个实例。因此，每次我们的实现调用 `IDistributedCache` 接口时，数据都会保存在测试服务器的内存中。

# 摘要

在本章中，我们介绍了 ASP.NET Core 中的一些主要缓存场景，以便您了解 HTTP 缓存的工作原理以及如何实现。此外，我们学习了如何使用 `IDistributedCache` 接口实现分布式缓存以及如何将 Web 服务连接到 Redis 实例。在下一章中，我们将探讨如何处理日志管理以及如何对目录 Web 服务的依赖项进行健康检查。
