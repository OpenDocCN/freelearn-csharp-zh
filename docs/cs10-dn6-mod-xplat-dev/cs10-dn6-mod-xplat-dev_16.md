# 第十六章：构建和消费 Web 服务

本章是关于学习如何使用 ASP.NET Core Web API 构建 Web 服务（即 HTTP 或 REST 服务）以及使用 HTTP 客户端消费 Web 服务，这些客户端可以是任何类型的.NET 应用，包括网站、移动或桌面应用。

本章要求您具备在*第十章*，*使用 Entity Framework Core 处理数据*，以及*第十三章*至*第十五章*中关于 C#和.NET 的实际应用以及使用 ASP.NET Core 构建网站的知识和技能。

本章我们将涵盖以下主题：

+   使用 ASP.NET Core Web API 构建 Web 服务

+   文档化和测试 Web 服务

+   使用 HTTP 客户端消费 Web 服务

+   为 Web 服务实现高级功能

+   使用最小 API 构建 Web 服务

# 使用 ASP.NET Core Web API 构建 Web 服务

在我们构建现代 Web 服务之前，需要先介绍一些背景知识，为本章设定上下文。

## 理解 Web 服务缩略语

尽管 HTTP 最初设计用于请求和响应 HTML 及其他供人类查看的资源，但它也非常适合构建服务。

罗伊·菲尔丁在其博士论文中描述**表述性状态转移**(**REST**)架构风格时指出，HTTP 标准适合构建服务，因为它定义了以下内容：

+   唯一标识资源的 URI，如`https://localhost:5001/api/products/23`。

+   对这些资源执行常见任务的方法，如`GET`、`POST`、`PUT`和`DELETE`。

+   请求和响应中交换的内容媒体类型协商能力，如 XML 和 JSON。内容协商发生在客户端指定类似`Accept: application/xml,*/*;q=0.8`的请求头时。ASP.NET Core Web API 默认的响应格式是 JSON，这意味着其中一个响应头会是`Content-Type: application/json; charset=utf-8`。

**Web 服务**采用 HTTP 通信标准，因此有时被称为 HTTP 或 RESTful 服务。本章讨论的就是 HTTP 或 RESTful 服务。

Web 服务也可指实现部分**WS-*标准**的**简单对象访问协议**(**SOAP**)服务。这些标准使不同系统上实现的客户端和服务能相互通信。WS-*标准最初由 IBM 定义，微软等其他公司也参与了制定。

### 理解 Windows Communication Foundation (WCF)

.NET Framework 3.0 及更高版本包含名为**Windows Communication Foundation**(**WCF**)的**远程过程调用**(**RPC**)技术。RPC 技术使一个系统上的代码能通过网络在另一系统上执行代码。

WCF 使开发者能轻松创建服务，包括实现 WS-*标准的 SOAP 服务。后来它也支持构建 Web/HTTP/REST 风格的服务，但如果仅需要这些，它显得过于复杂。

如果你有现有的 WCF 服务并希望将它们迁移到现代.NET，那么有一个开源项目在 2021 年 2 月发布了其首个**正式发布版**（**GA**）。你可以在以下链接中了解更多信息：

[`corewcf.github.io/blog/2021/02/19/corewcf-ga-release`](https://corewcf.github.io/blog/2021/02/19/corewcf-ga-release)

### 替代 WCF 的方案

微软推荐的 WCF 替代方案是**gRPC**。gRPC 是一种现代的跨平台开源 RPC 框架，由谷歌创建（非官方地，“g”代表 gRPC）。你将在*第十八章*，*构建和消费专业化服务*中了解更多关于 gRPC 的信息。

## 理解 Web API 的 HTTP 请求和响应

HTTP 定义了标准的请求类型和标准代码来指示响应类型。大多数这些类型和代码可用于实现 Web API 服务。

最常见的请求类型是`GET`，用于检索由唯一路径标识的资源，并附带如可接受的媒体类型等额外选项，这些选项作为请求头设置，如下例所示：

```cs
GET /path/to/resource
Accept: application/json 
```

常见响应包括成功和多种失败类型，如下表所示：

| 状态码 | 描述 |
| --- | --- |
| `200 成功` | 路径正确形成，资源成功找到，序列化为可接受的媒体类型，然后返回在响应体中。响应头指定`Content-Type`、`Content-Length`和`Content-Encoding`，例如 GZIP。 |
| `301 永久移动` | 随着时间的推移，Web 服务可能会更改其资源模型，包括用于标识现有资源的路径。Web 服务可以通过返回此状态码和一个名为`Location`的响应头来指示新路径，该响应头包含新路径。 |
| `302 找到` | 类似于`301`。 |
| `304 未修改` | 如果请求包含`If-Modified-Since`头，则 Web 服务可以响应此状态码。响应体为空，因为客户端应使用其缓存的资源副本。 |
| `400 错误请求` | 请求无效，例如，它使用了一个整数 ID 的产品路径，但 ID 值缺失。 |
| `401 未授权` | 请求有效，资源已找到，但客户端未提供凭证或无权访问该资源。重新认证可能会启用访问，例如，通过添加或更改`Authorization`请求头。 |
| `403 禁止访问` | 请求有效，资源已找到，但客户端无权访问该资源。重新认证也无法解决问题。 |
| `404 未找到` | 请求有效，但资源未找到。如果稍后重复请求，资源可能会被找到。若要表明资源将永远无法找到，返回`410 已删除`。 |
| `406 不可接受` | 如果请求具有仅列出网络服务不支持的媒体类型的`Accept`头。例如，如果客户端请求 JSON 但网络服务只能返回 XML。 |
| `451 因法律原因不可用` | 在美国托管的网站可能会为来自欧洲的请求返回此状态，以避免不得不遵守《通用数据保护条例》（GDPR）。该数字的选择是对小说《华氏 451 度》的引用，其中书籍被禁止和焚烧。 |
| `500 服务器错误` | 请求有效，但在处理请求时服务器端出现问题。稍后再试可能有效。 |
| `503 服务不可用` | 网络服务正忙，无法处理请求。稍后再试可能有效。 |

其他常见的 HTTP 请求类型包括`POST`、`PUT`、`PATCH`或`DELETE`，用于创建、修改或删除资源。

要创建新资源，您可能会发出带有包含新资源的正文的`POST`请求，如下所示：

```cs
POST /path/to/resource
Content-Length: 123
Content-Type: application/json 
```

要创建新资源或更新现有资源，您可能会发出带有包含现有资源全新版本的正文的`PUT`请求，如果资源不存在，则创建它，如果存在，则替换它（有时称为**upsert**操作），如下所示：

```cs
PUT /path/to/resource
Content-Length: 123
Content-Type: application/json 
```

要更有效地更新现有资源，您可能会发出带有包含仅需要更改的属性的对象的正文的`PATCH`请求，如下所示：

```cs
PATCH /path/to/resource
Content-Length: 123
Content-Type: application/json 
```

要删除现有资源，您可能会发出`DELETE`请求，如下所示：

```cs
DELETE /path/to/resource 
```

除了上述表格中针对`GET`请求的响应外，所有创建、修改或删除资源的请求类型都有额外的可能的常见响应，如下表所示：

| 状态码 | 描述 |
| --- | --- |
| `201 已创建` | 新资源已成功创建，响应头名为`Location`包含其路径，响应正文包含新创建的资源。立即`GET`资源应返回`200`。 |
| `202 已接受` | 新资源无法立即创建，因此请求被排队等待稍后处理，立即`GET`资源可能会返回`404`。正文可以包含指向某种状态检查器或资源可用时间估计的资源。 |
| `204 无内容` | 通常用于响应`DELETE`请求，因为在删除后在正文中返回资源通常没有意义！有时用于响应`POST`、`PUT`或`PATCH`请求，如果客户端不需要确认请求是否正确处理。 |
| `405 方法不允许` | 当请求使用的方法不被支持时返回。例如，设计为只读的网络服务可能明确禁止`PUT`、`DELETE`等。 |
| `415 Unsupported Media Type` | 当请求体中的资源使用 Web 服务无法处理的媒体类型时返回。例如，如果主体包含 XML 格式的资源，但 Web 服务只能处理 JSON。 |

## 创建 ASP.NET Core Web API 项目

我们将构建一个 Web 服务，该服务提供了一种使用 ASP.NET Core 在 Northwind 数据库中处理数据的方法，以便数据可以被任何能够发出 HTTP 请求并在任何平台上接收 HTTP 响应的客户端应用程序使用：

1.  使用您喜欢的代码编辑器添加新项目，如以下列表所定义：

    1.  项目模板：**ASP.NET Core Web API** / `webapi`

    1.  工作区/解决方案文件和文件夹：`PracticalApps`

    1.  项目文件和文件夹：`Northwind.WebApi`

    1.  其他 Visual Studio 选项：**身份验证类型**：无，**为 HTTPS 配置**：已选中，**启用 Docker**：已清除，**启用 OpenAPI 支持**：已选中。

1.  在 Visual Studio Code 中，选择`Northwind.WebApi`作为活动的 OmniSharp 项目。

1.  构建`Northwind.WebApi`项目。

1.  在`Controllers`文件夹中，打开并审查`WeatherForecastController.cs`，如下所示：

    ```cs
    using Microsoft.AspNetCore.Mvc;
    namespace Northwind.WebApi.Controllers;
    [ApiController]
    [Route("[controller]")]
    public class WeatherForecastController : ControllerBase
    {
      private static readonly string[] Summaries = new[]
      {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild",
        "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
      };
      private readonly ILogger<WeatherForecastController> _logger;
      public WeatherForecastController(
        ILogger<WeatherForecastController> logger)
      {
        _logger = logger;
      }
      [HttpGet]
      public IEnumerable<WeatherForecast> Get()
      {
        return Enumerable.Range(1, 5).Select(index =>
          new WeatherForecast
          {
            Date = DateTime.Now.AddDays(index),
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = Summaries[Random.Shared.Next(Summaries.Length)]
          })
          .ToArray();
      }
    } 
    ```

    在审查前面的代码时，请注意以下几点：

    +   控制器类`Controller`继承自`ControllerBase`。这比 MVC 中使用的`Controller`类更简单，因为它没有像`View`这样的方法，通过将视图模型传递给 Razor 文件来生成 HTML 响应。

    +   `[Route]`属性为客户端注册了`/weatherforecast`相对 URL，用于发出将由该控制器处理的 HTTP 请求。例如，对`https://localhost:5001/weatherforecast/`的 HTTP 请求将由该控制器处理。一些开发人员喜欢在控制器名称前加上`api/`，这是一种区分混合项目中 MVC 和 Web API 的约定。如果使用`[controller]`，如所示，它使用类名中`Controller`之前的字符，在本例中为`WeatherForecast`，或者您可以简单地输入一个不同的名称，不带方括号，例如`[Route("api/forecast")]`。

    +   `[ApiController]`属性是在 ASP.NET Core 2.1 中引入的，它为控制器启用了 REST 特定的行为，例如对于无效模型的自动 HTTP `400`响应，如本章后面将看到的。

    +   `[HttpGet]`属性将`Controller`类中的`Get`方法注册为响应 HTTP `GET`请求，其实现使用共享的`Random`对象返回一个`WeatherForecast`对象数组，其中包含未来五天的随机温度和摘要，如`Bracing`或`Balmy`。

1.  添加第二个`Get`方法，该方法允许调用指定预测应提前多少天，通过实现以下内容：

    +   在原始方法上方添加注释，以显示其响应的操作方法和 URL 路径。

    +   添加一个带有整数参数`days`的新方法。

    +   将原始`Get`方法实现代码语句剪切并粘贴到新的`Get`方法中。

    +   修改新方法以创建一个整数`IEnumerable`，其上限为请求的天数，并修改原始`Get`方法以调用新`Get`方法并传递值`5`。

你的方法应如以下代码中突出显示的那样：

```cs
**// GET /weatherforecast**
[HttpGet]
public IEnumerable<WeatherForecast> Get() **// original method**
{
  **return** **Get(****5****);** **// five day forecast**
}
**// GET /weatherforecast/7**
**[****HttpGet(****"{days:int}"****)****]**
**public** **IEnumerable<WeatherForecast>** **Get****(****int** **days****)** **// new method**
{
**return** **Enumerable.Range(****1****, days).Select(index =>**
    new WeatherForecast
    {
      Date = DateTime.Now.AddDays(index),
      TemperatureC = Random.Shared.Next(-20, 55),
      Summary = Summaries[Random.Shared.Next(Summaries.Length)]
    })
    .ToArray();
} 
```

在`[HttpGet]`属性中，注意路由格式模式`{days:int}`，它将`days`参数约束为`int`值。

## 审查 Web 服务的功能

现在，我们将测试 Web 服务的功能：

1.  如果你使用的是 Visual Studio，在**属性**中，打开`launchSettings.json`文件，并注意默认情况下，它将启动浏览器并导航至`/swagger`相对 URL 路径，如下所示突出显示：

    ```cs
    "profiles": {
      "Northwind.WebApi": {
        "commandName": "Project",
        "dotnetRunMessages": "true",
    **"launchBrowser"****:** **true****,**
    **"launchUrl"****:** **"swagger"****,**
        "applicationUrl": "https://localhost:5001;http://localhost:5000",
        "environmentVariables": {
          "ASPNETCORE_ENVIRONMENT": "Development"
        }
      }, 
    ```

1.  修改名为`Northwind.WebApi`的配置文件，将`launchBrowser`设置为`false`。

1.  对于`applicationUrl`，将随机端口号更改为`HTTP`的`5000`和`HTTPS`的`5001`。

1.  启动 Web 服务项目。

1.  启动 Chrome。

1.  导航至`https://localhost:5001/`，注意你会收到一个`404`状态码响应，因为我们尚未启用静态文件，也没有`index.html`文件，或者配置了路由的 MVC 控制器。记住，此项目并非设计为人机交互界面，因此对于 Web 服务而言，这是预期行为。

    GitHub 上的解决方案配置为使用端口`5002`，因为在本书后面我们将更改其配置。

1.  在 Chrome 中，显示**开发者工具**。

1.  导航至`https://localhost:5001/weatherforecast`，注意 Web API 服务应返回一个包含五个随机天气预报对象的 JSON 文档数组，如图*16.1*所示：![](img/B17442_17_01.png)

    图 16.1：来自天气预报 Web 服务的请求与响应

1.  关闭**开发者工具**。

1.  导航至`https://localhost:5001/weatherforecast/14`，并注意请求两周天气预报时的响应，如图*16.2*所示：![](img/B17442_17_02.png)

    图 16.2：两周天气预报的 JSON 文档

1.  关闭 Chrome 并关闭 Web 服务器。

## 为 Northwind 数据库创建 Web 服务

与 MVC 控制器不同，Web API 控制器不会调用 Razor 视图以返回 HTML 响应供网站访问者在浏览器中查看。相反，它们使用与发起 HTTP 请求的客户端应用程序的内容协商，在 HTTP 响应中返回 XML、JSON 或 X-WWW-FORM-URLENCODED 等格式的数据。

客户端应用程序必须随后将数据从协商格式反序列化。现代 Web 服务最常用的格式是**JavaScript 对象表示法**（**JSON**），因为它紧凑且在构建使用 Angular、React 和 Vue 等客户端技术的**单页应用程序**（**SPAs**）时，能与浏览器中的 JavaScript 原生工作。

我们将引用你在*第十三章*，*C#与.NET 实用应用入门*中创建的 Northwind 数据库的 Entity Framework Core 实体数据模型：

1.  在`Northwind.WebApi`项目中，为 SQLite 或 SQL Server 添加对`Northwind.Common.DataContext`的项目引用，如下所示：

    ```cs
    <ItemGroup>
      <!-- change Sqlite to SqlServer if you prefer -->
      <ProjectReference Include=
    "..\Northwind.Common.DataContext.Sqlite\Northwind.Common.DataContext.Sqlite.csproj" />
    </ItemGroup> 
    ```

1.  构建项目并修复代码中的任何编译错误。

1.  打开`Program.cs`并导入用于处理 Web 媒体格式化程序和共享 Packt 类的命名空间，如下所示：

    ```cs
    using Microsoft.AspNetCore.Mvc.Formatters;
    using Packt.Shared; // AddNorthwindContext extension method
    using static System.Console; 
    ```

1.  在调用`AddControllers`之前，添加一条语句以注册`Northwind`数据库上下文类（它将根据您在项目文件中引用的数据库提供程序使用 SQLite 或 SQL Server），如下所示：

    ```cs
    // Add services to the container.
    builder.Services.AddNorthwindContext(); 
    ```

1.  在调用`AddControllers`时，添加一个 lambda 块，其中包含将默认输出格式化程序的名称和支持的媒体类型写入控制台的语句，然后添加 XML 序列化程序格式化程序，如下所示：

    ```cs
    builder.Services.AddControllers(options =>
    {
      WriteLine("Default output formatters:");
      foreach (IOutputFormatter formatter in options.OutputFormatters)
      {
        OutputFormatter? mediaFormatter = formatter as OutputFormatter;
        if (mediaFormatter == null)
        {
          WriteLine($"  {formatter.GetType().Name}");
        }
        else // OutputFormatter class has SupportedMediaTypes
        {
          WriteLine("  {0}, Media types: {1}",
            arg0: mediaFormatter.GetType().Name,
            arg1: string.Join(", ",
              mediaFormatter.SupportedMediaTypes));
        }
      }
    })
    .AddXmlDataContractSerializerFormatters()
    .AddXmlSerializerFormatters(); 
    ```

1.  启动 Web 服务。

1.  在命令提示符或终端中，请注意有四种默认的输出格式化程序，包括将`null`值转换为`204 No Content`的程序，以及支持纯文本、字节流和 JSON 响应的程序，如下所示：

    ```cs
    Default output formatters: 
      HttpNoContentOutputFormatter
      StringOutputFormatter, Media types: text/plain
      StreamOutputFormatter
      SystemTextJsonOutputFormatter, Media types: application/json, text/json, application/*+json 
    ```

1.  关闭 Web 服务器。

## 为实体创建数据仓库

定义和实现提供 CRUD 操作的数据仓库是良好的实践。CRUD 缩写包括以下操作：

+   C 代表创建

+   R 代表检索（或读取）

+   U 代表更新

+   D 代表删除

我们将为 Northwind 中的`Customers`表创建一个数据仓库。该表中只有 91 个客户，因此我们将整个表的副本存储在内存中，以提高读取客户记录时的可扩展性和性能。

**最佳实践**：在实际的 Web 服务中，应使用分布式缓存，如 Redis，这是一个开源的数据结构存储，可用作高性能、高可用性的数据库、缓存或消息代理。

我们将遵循现代最佳实践，使仓库 API 异步。它将通过构造函数参数注入由`Controller`类实例化，因此会为每个 HTTP 请求创建一个新实例：

1.  在`Northwind.WebApi`项目中，创建一个名为`Repositories`的文件夹。

1.  向`Repositories`文件夹添加两个类文件，名为`ICustomerRepository.cs`和`CustomerRepository.cs`。

1.  `ICustomerRepository`接口将定义五个方法，如下所示：

    ```cs
    using Packt.Shared; // Customer
    namespace Northwind.WebApi.Repositories;
    public interface ICustomerRepository
    {
      Task<Customer?> CreateAsync(Customer c);
      Task<IEnumerable<Customer>> RetrieveAllAsync();
      Task<Customer?> RetrieveAsync(string id);
      Task<Customer?> UpdateAsync(string id, Customer c);
      Task<bool?> DeleteAsync(string id);
    } 
    ```

1.  `CustomerRepository`类将实现这五个方法，记住，使用`await`的方法必须标记为`async`，如下所示：

    ```cs
    using Microsoft.EntityFrameworkCore.ChangeTracking; // EntityEntry<T>
    using Packt.Shared; // Customer
    using System.Collections.Concurrent; // ConcurrentDictionary
    namespace Northwind.WebApi.Repositories;
    public class CustomerRepository : ICustomerRepository
    {
      // use a static thread-safe dictionary field to cache the customers
      private static ConcurrentDictionary
        <string, Customer>? customersCache;
      // use an instance data context field because it should not be
      // cached due to their internal caching
      private NorthwindContext db;
      public CustomerRepository(NorthwindContext injectedContext)
      {
        db = injectedContext;
        // pre-load customers from database as a normal
        // Dictionary with CustomerId as the key,
        // then convert to a thread-safe ConcurrentDictionary
        if (customersCache is null)
        {
          customersCache = new ConcurrentDictionary<string, Customer>(
            db.Customers.ToDictionary(c => c.CustomerId));
        }
      }
      public async Task<Customer?> CreateAsync(Customer c)
      {
        // normalize CustomerId into uppercase
        c.CustomerId = c.CustomerId.ToUpper();
        // add to database using EF Core
        EntityEntry<Customer> added = await db.Customers.AddAsync(c);
        int affected = await db.SaveChangesAsync();
        if (affected == 1)
        {
          if (customersCache is null) return c;
          // if the customer is new, add it to cache, else
          // call UpdateCache method
          return customersCache.AddOrUpdate(c.CustomerId, c, UpdateCache);
        }
        else
        {
          return null;
        }
      }
      public Task<IEnumerable<Customer>> RetrieveAllAsync()
      {
        // for performance, get from cache
        return Task.FromResult(customersCache is null 
            ? Enumerable.Empty<Customer>() : customersCache.Values);
      }
      public Task<Customer?> RetrieveAsync(string id)
      {
        // for performance, get from cache
        id = id.ToUpper();
        if (customersCache is null) return null!;
        customersCache.TryGetValue(id, out Customer? c);
        return Task.FromResult(c);
      }
      private Customer UpdateCache(string id, Customer c)
      {
        Customer? old;
        if (customersCache is not null)
        {
          if (customersCache.TryGetValue(id, out old))
          {
            if (customersCache.TryUpdate(id, c, old))
            {
              return c;
            }
          }
        }
        return null!;
      }
      public async Task<Customer?> UpdateAsync(string id, Customer c)
      {
        // normalize customer Id
        id = id.ToUpper();
        c.CustomerId = c.CustomerId.ToUpper();
        // update in database
        db.Customers.Update(c);
        int affected = await db.SaveChangesAsync();
        if (affected == 1)
        {
          // update in cache
          return UpdateCache(id, c);
        }
        return null;
      }
      public async Task<bool?> DeleteAsync(string id)
      {
        id = id.ToUpper();
        // remove from database
        Customer? c = db.Customers.Find(id);
        if (c is null) return null;
        db.Customers.Remove(c);
        int affected = await db.SaveChangesAsync();
        if (affected == 1)
        {
          if (customersCache is null) return null;
          // remove from cache
          return customersCache.TryRemove(id, out c);
        }
        else
        {
          return null;
        }
      }
    } 
    ```

## 实现 Web API 控制器

对于返回数据而非 HTML 的控制器，有一些有用的属性和方法。

使用 MVC 控制器时，像`/home/index`这样的路由告诉我们控制器类名和操作方法名，例如`HomeController`类和`Index`操作方法。

使用 Web API 控制器，如`/weatherforecast`的路由仅告诉我们控制器类名，例如`WeatherForecastController`。为了确定要执行的操作方法名称，我们必须将 HTTP 方法（如`GET`和`POST`）映射到控制器类中的方法。

您应该使用以下属性装饰控制器方法，以指示它们将响应的 HTTP 方法：

+   `[HttpGet]`，`[HttpHead]`：这些操作方法响应`GET`或`HEAD`请求以检索资源，并返回资源及其响应头或仅返回响应头。

+   `[HttpPost]`：此操作方法响应`POST`请求以创建新资源或执行服务定义的其他操作。

+   `[HttpPut]`，`[HttpPatch]`：这些操作方法响应`PUT`或`PATCH`请求以更新现有资源，无论是替换还是更新其属性的子集。

+   `[HttpDelete]`：此操作方法响应`DELETE`请求以删除资源。

+   `[HttpOptions]`：此操作方法响应`OPTIONS`请求。

### 理解操作方法返回类型

操作方法可以返回.NET 类型，如单个`string`值、由`class`、`record`或`struct`定义的复杂对象，或复杂对象的集合。ASP.NET Core Web API 会将它们序列化为 HTTP 请求`Accept`头中设置的请求数据格式，例如，如果已注册合适的序列化器，则为 JSON。

为了更精细地控制响应，有一些辅助方法返回围绕.NET 类型的`ActionResult`包装器。

如果操作方法可能基于输入或其他变量返回不同的返回类型，则应声明其返回类型为`IActionResult`。如果操作方法将仅返回单个类型但具有不同的状态代码，则应声明其返回类型为`ActionResult<T>`。

**最佳实践**：使用`[ProducesResponseType]`属性装饰操作方法，以指示客户端应在响应中预期的所有已知类型和 HTTP 状态代码。此信息随后可以公开，以说明客户端应如何与您的 Web 服务交互。将其视为正式文档的一部分。本章后面，您将学习如何安装代码分析器，以便在您未按此方式装饰操作方法时给出警告。

例如，根据 id 参数获取产品的操作方法将装饰有三个属性——一个表示它响应`GET`请求并具有 id 参数，另外两个表示成功时和客户端提供无效产品 ID 时的处理方式，如下面的代码所示：

```cs
[HttpGet("{id}")]
[ProducesResponseType(200, Type = typeof(Product))] 
[ProducesResponseType(404)]
public IActionResult Get(string id) 
```

`ControllerBase`类具有方法，使其易于返回不同的响应，如下表所示：

| 方法 | 描述 |
| --- | --- |
| `Ok` | 返回`200`状态码和一个转换为客户端首选格式的资源，如 JSON 或 XML。常用于响应`GET`请求。 |
| `CreatedAtRoute` | 返回一个`201`状态码和到新资源的路径。通常用于响应`POST`请求以快速创建资源。 |
| `Accepted` | 返回一个`202`状态码以指示请求正在处理但尚未完成。通常用于响应`POST`、`PUT`、`PATCH`或`DELETE`请求，这些请求触发了一个需要很长时间才能完成的背景进程。 |
| `NoContentResult` | 返回一个`204`状态码和一个空的响应主体。通常用于响应`PUT`、`PATCH`或`DELETE`请求，当响应不需要包含受影响的资源时。 |
| `BadRequest` | 返回一个`400`状态码和一个可选的详细信息消息字符串。 |
| `NotFound` | 返回一个`404`状态码和一个自动填充的`ProblemDetails`主体（需要 2.2 或更高版本的兼容性版本）。 |

## 配置客户仓库和 Web API 控制器

现在您将配置仓库，以便它可以从 Web API 控制器内部调用。

当 Web 服务启动时，您将为仓库注册一个作用域依赖服务实现，然后使用构造函数参数注入在新 Web API 控制器中获取它，以便与客户工作。

为了展示使用路由区分 MVC 和 Web API 控制器的示例，我们将使用客户控制器的常见`/api`URL 前缀约定：

1.  打开`Program.cs`并导入`Northwind.WebApi.Repositories`命名空间。

1.  在调用`Build`方法之前添加一个语句，该语句将注册`CustomerRepository`以在运行时作为作用域依赖使用，如下所示高亮显示的代码：

    ```cs
    **builder.Services.AddScoped<ICustomerRepository, CustomerRepository>();**
    var app = builder.Build(); 
    ```

    **最佳实践**：我们的仓库使用一个注册为作用域依赖的数据库上下文。您只能在其他作用域依赖内部使用作用域依赖，因此我们不能将仓库注册为单例。您可以在以下链接了解更多信息：[`docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#scoped`](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#scoped)

1.  在`Controllers`文件夹中，添加一个名为`CustomersController.cs`的新类。

1.  在`CustomersController`类文件中，添加语句以定义一个 Web API 控制器类以与客户工作，如下所示的代码：

    ```cs
    using Microsoft.AspNetCore.Mvc; // [Route], [ApiController], ControllerBase
    using Packt.Shared; // Customer
    using Northwind.WebApi.Repositories; // ICustomerRepository
    namespace Northwind.WebApi.Controllers;
    // base address: api/customers
    [Route("api/[controller]")]
    [ApiController]
    public class CustomersController : ControllerBase
    {
      private readonly ICustomerRepository repo;
      // constructor injects repository registered in Startup
      public CustomersController(ICustomerRepository repo)
      {
        this.repo = repo;
      }
      // GET: api/customers
      // GET: api/customers/?country=[country]
      // this will always return a list of customers (but it might be empty)
      [HttpGet]
      [ProducesResponseType(200, Type = typeof(IEnumerable<Customer>))]
      public async Task<IEnumerable<Customer>> GetCustomers(string? country)
      {
        if (string.IsNullOrWhiteSpace(country))
        {
          return await repo.RetrieveAllAsync();
        }
        else
        {
          return (await repo.RetrieveAllAsync())
            .Where(customer => customer.Country == country);
        }
      }
      // GET: api/customers/[id]
      [HttpGet("{id}", Name = nameof(GetCustomer))] // named route
      [ProducesResponseType(200, Type = typeof(Customer))]
      [ProducesResponseType(404)]
      public async Task<IActionResult> GetCustomer(string id)
      {
        Customer? c = await repo.RetrieveAsync(id);
        if (c == null)
        {
          return NotFound(); // 404 Resource not found
        }
        return Ok(c); // 200 OK with customer in body
      }
      // POST: api/customers
      // BODY: Customer (JSON, XML)
      [HttpPost]
      [ProducesResponseType(201, Type = typeof(Customer))]
      [ProducesResponseType(400)]
      public async Task<IActionResult> Create([FromBody] Customer c)
      {
        if (c == null)
        {
          return BadRequest(); // 400 Bad request
        }
        Customer? addedCustomer = await repo.CreateAsync(c);
        if (addedCustomer == null)
        {
          return BadRequest("Repository failed to create customer.");
        }
        else
        {
          return CreatedAtRoute( // 201 Created
            routeName: nameof(GetCustomer),
            routeValues: new { id = addedCustomer.CustomerId.ToLower() },
            value: addedCustomer);
        }
      }
      // PUT: api/customers/[id]
      // BODY: Customer (JSON, XML)
      [HttpPut("{id}")]
      [ProducesResponseType(204)]
      [ProducesResponseType(400)]
      [ProducesResponseType(404)]
      public async Task<IActionResult> Update(
        string id, [FromBody] Customer c)
      {
        id = id.ToUpper();
        c.CustomerId = c.CustomerId.ToUpper();
        if (c == null || c.CustomerId != id)
        {
          return BadRequest(); // 400 Bad request
        }
        Customer? existing = await repo.RetrieveAsync(id);
        if (existing == null)
        {
          return NotFound(); // 404 Resource not found
        }
        await repo.UpdateAsync(id, c);
        return new NoContentResult(); // 204 No content
      }
      // DELETE: api/customers/[id]
      [HttpDelete("{id}")]
      [ProducesResponseType(204)]
      [ProducesResponseType(400)]
      [ProducesResponseType(404)]
      public async Task<IActionResult> Delete(string id)
      {
        Customer? existing = await repo.RetrieveAsync(id);
        if (existing == null)
        {
          return NotFound(); // 404 Resource not found
        }
        bool? deleted = await repo.DeleteAsync(id);
        if (deleted.HasValue && deleted.Value) // short circuit AND
        {
          return new NoContentResult(); // 204 No content
        }
        else
        {
          return BadRequest( // 400 Bad request
            $"Customer {id} was found but failed to delete.");
        }
      }
    } 
    ```

在审查此 Web API 控制器类时，请注意以下内容：

+   `Controller`类注册了一个以`api/`开头的路由，并包含控制器的名称，即`api/customers`。

+   构造函数使用依赖注入来获取注册的仓库以与客户工作。

+   有五个操作方法来执行对客户的 CRUD 操作——两个`GET`方法（获取所有客户或一个客户），`POST`（创建），`PUT`（更新）和`DELETE`。

+   方法`GetCustomers`可以接受一个`string`类型的参数，该参数为国名。若该参数缺失，则返回所有客户信息。若存在，则用于按国家筛选客户。

+   `GetCustomer`方法有一个显式命名的路由`GetCustomer`，以便在插入新客户后用于生成 URL。

+   `Create`和`Update`方法都使用`[FromBody]`装饰`customer`参数，以告知模型绑定器从`POST`请求体中填充其值。

+   `Create`方法返回的响应使用了`GetCustomer`路由，以便客户端知道将来如何获取新创建的资源。我们正在将两个方法匹配起来，以创建并获取客户。

+   `Create`和`Update`方法无需检查 HTTP 请求体中传递的客户模型状态，并在模型无效时返回包含模型验证错误详情的`400 Bad Request`，因为控制器装饰有`[ApiController]`，它会为你执行此操作。

当服务接收到 HTTP 请求时，它将创建一个`Controller`类实例，调用相应的动作方法，以客户端偏好的格式返回响应，并释放控制器使用的资源，包括仓库及其数据上下文。

## 指定问题详情

ASP.NET Core 2.1 及更高版本新增了一项特性，即实现了指定问题详情的 Web 标准。

在启用了 ASP.NET Core 2.2 或更高版本兼容性的项目中，使用`[ApiController]`装饰的 Web API 控制器中，返回`IActionResult`且返回客户端错误状态码（即`4xx`）的动作方法，将自动在响应体中包含`ProblemDetails`类的序列化实例。

如果你想自行控制，那么你可以创建一个`ProblemDetails`实例，并包含额外信息。

让我们模拟一个需要向客户端返回自定义数据的错误请求：

1.  在`Delete`方法的实现顶部，添加语句检查`id`是否匹配字符串值`"bad"`，如果是，则返回一个自定义的问题详情对象，如下所示：

    ```cs
    // take control of problem details
    if (id == "bad")
    {
      ProblemDetails problemDetails = new()
      {
        Status = StatusCodes.Status400BadRequest,
        Type = "https://localhost:5001/customers/failed-to-delete",
        Title = $"Customer ID {id} found but failed to delete.",
        Detail = "More details like Company Name, Country and so on.",
        Instance = HttpContext.Request.Path
      };
      return BadRequest(problemDetails); // 400 Bad Request
    } 
    ```

1.  你稍后将测试此功能。

## 控制 XML 序列化

在`Program.cs`文件中，我们添加了`XmlSerializer`，以便我们的 Web API 服务在客户端请求时，既能返回 JSON 也能返回 XML。

然而，`XmlSerializer`无法序列化接口，而我们的实体类使用`ICollection<T>`来定义相关子实体，这会在运行时导致警告，例如对于`Customer`类及其`Orders`属性，如下输出所示：

```cs
warn: Microsoft.AspNetCore.Mvc.Formatters.XmlSerializerOutputFormatter[1]
An error occurred while trying to create an XmlSerializer for the type 'Packt.Shared.Customer'.
System.InvalidOperationException: There was an error reflecting type 'Packt.Shared.Customer'.
---> System.InvalidOperationException: Cannot serialize member 'Packt.
Shared.Customer.Orders' of type 'System.Collections.Generic.ICollection`1[[Packt. Shared.Order, Northwind.Common.EntityModels, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null]]', see inner exception for more details. 
```

我们可以通过在将`Customer`序列化为 XML 时排除`Orders`属性来防止此警告：

1.  在`Northwind.Common.EntityModels.Sqlite`和`Northwind.Common.EntityModels.SqlServer`项目中，打开`Customers.cs`文件。

1.  导入`System.Xml.Serialization`命名空间，以便我们能使用`[XmlIgnore]`属性。

1.  为`Orders`属性添加一个属性，以便在序列化时忽略它，如下面的代码中突出显示的那样：

    ```cs
    [InverseProperty(nameof(Order.Customer))]
    **[****XmlIgnore****]**
    public virtual ICollection<Order> Orders { get; set; } 
    ```

1.  在`Northwind.Common.EntityModels.SqlServer`项目中，同样为`CustomerCustomerDemos`属性添加`[XmlIgnore]`装饰。

# 记录和测试网络服务

通过浏览器发起 HTTP `GET`请求，你可以轻松测试网络服务。要测试其他 HTTP 方法，我们需要更高级的工具。

## 使用浏览器测试 GET 请求

你将使用 Chrome 测试`GET`请求的三种实现——获取所有客户、获取指定国家的客户以及通过唯一客户 ID 获取单个客户：

1.  启动`Northwind.WebApi`网络服务。

1.  启动 Chrome。

1.  访问`https://localhost:5001/api/customers`并注意返回的 JSON 文档，其中包含 Northwind 数据库中的所有 91 位客户（未排序），如图*16.3*所示：![](img/B17442_17_03.png)

    图 16.3：Northwind 数据库中的客户作为 JSON 文档

1.  访问`https://localhost:5001/api/customers/?country=Germany`并注意返回的 JSON 文档，其中仅包含德国的客户，如图*16.4*所示：![](img/B17442_17_04.png)

    图 16.4：来自德国的客户列表作为 JSON 文档

    如果返回的是空数组，请确保你输入的国家名称使用了正确的字母大小写，因为数据库查询是区分大小写的。例如，比较`uk`和`UK`的结果。

1.  访问`https://localhost:5001/api/customers/alfki`并注意返回的 JSON 文档，其中仅包含名为**Alfreds Futterkiste**的客户，如图*16.5*所示：![](img/B17442_17_05.png)

    图 16.5：特定客户信息作为 JSON 文档

与国家名称不同，我们无需担心客户`id`值的大小写，因为在控制器类内部，我们已在代码中将`string`值规范化为大写。

但我们如何测试其他 HTTP 方法，如`POST`、`PUT`和`DELETE`？以及我们如何记录我们的网络服务，使其易于任何人理解如何与之交互？

为解决第一个问题，我们可以安装一个名为**REST Client**的 Visual Studio Code 扩展。为解决第二个问题，我们可以使用**Swagger**，这是全球最流行的 HTTP API 文档和测试技术。但首先，让我们看看 Visual Studio Code 扩展能做什么。

有许多工具可用于测试 Web API，例如**Postman**。尽管 Postman 很受欢迎，但我更喜欢**REST Client**，因为它不会隐藏实际发生的情况。我觉得 Postman 过于图形化。但我鼓励你探索不同的工具，找到适合你风格的工具。你可以在以下链接了解更多关于 Postman 的信息：[`www.postman.com/`](https://www.postman.com/)

## 使用 REST Client 扩展测试 HTTP 请求

REST Client 是一个扩展，允许你在 Visual Studio Code 中发送任何类型的 HTTP 请求并查看响应。即使你更喜欢使用 Visual Studio 作为代码编辑器，安装 Visual Studio Code 来使用像 REST Client 这样的扩展也是有用的。

### 使用 REST Client 进行 GET 请求

我们将首先创建一个文件来测试`GET`请求：

1.  如果你尚未安装由毛华超（`humao.rest-client`）开发的 REST Client，请立即在 Visual Studio Code 中安装它。

1.  在你偏好的代码编辑器中，启动`Northwind.WebApi`项目网络服务。

1.  在 Visual Studio Code 中，在`PracticalApps`文件夹中创建一个`RestClientTests`文件夹，然后打开该文件夹。

1.  在`RestClientTests`文件夹中，创建一个名为`get-customers.http`的文件，并修改其内容以包含一个 HTTP `GET`请求来检索所有客户，如下面的代码所示：

    ```cs
    GET https://localhost:5001/api/customers/ HTTP/1.1 
    ```

1.  在 Visual Studio Code 中，导航至**视图** | **命令面板**，输入`rest client`，选择命令**Rest Client: Send Request**，然后按 Enter，如图 16.6 所示：![](img/B17442_17_06.png)

    图 16.6：使用 REST Client 发送 HTTP GET 请求

1.  注意**响应**显示在一个新的选项卡窗口面板中，并且你可以通过拖放选项卡将打开的选项卡重新排列为水平布局。

1.  输入更多`GET`请求，每个请求之间用三个井号分隔，以测试获取不同国家的客户和使用其 ID 获取单个客户，如下面的代码所示：

    ```cs
    ###
    GET https://localhost:5001/api/customers/?country=Germany HTTP/1.1 
    ###
    GET https://localhost:5001/api/customers/?country=USA HTTP/1.1 
    Accept: application/xml
    ###
    GET https://localhost:5001/api/customers/ALFKI HTTP/1.1 
    ###
    GET https://localhost:5001/api/customers/abcxy HTTP/1.1 
    ```

1.  点击每个请求上方的**发送请求**链接来发送它；例如，具有请求头以 XML 而非 JSON 格式请求美国客户的`GET`请求，如图 16.7 所示：![](img/B17442_17_07.png)

图 16.7：使用 REST Client 发送 XML 请求并获取响应

### 使用 REST Client 进行其他请求

接下来，我们将创建一个文件来测试其他请求，如`POST`：

1.  在`RestClientTests`文件夹中，创建一个名为`create-customer.http`的文件，并修改其内容以定义一个`POST`请求来创建新客户，注意 REST Client 将在你输入常见 HTTP 请求时提供 IntelliSense，如下面的代码所示：

    ```cs
    POST https://localhost:5001/api/customers/ HTTP/1.1 
    Content-Type: application/json
    Content-Length: 301
    {
      "customerID": "ABCXY",
      "companyName": "ABC Corp",
      "contactName": "John Smith",
      "contactTitle": "Sir",
      "address": "Main Street",
      "city": "New York",
      "region": "NY",
      "postalCode": "90210",
      "country":  "USA",
      "phone": "(123) 555-1234",
      "fax": null,
      "orders": null
    } 
    ```

1.  由于不同操作系统中的行尾不同，`Content-Length`头的值在 Windows 和 macOS 或 Linux 上会有所不同。如果值错误，则请求将失败。要发现正确的内容长度，选择请求的主体，然后在状态栏中查看字符数，如图 16.8 所示：![](img/B17442_17_08.png)

    图 16.8：检查正确的内容长度

1.  发送请求并注意响应是`201 Created`。同时注意新创建客户的地址（即 URL）是`https://localhost:5001/api/Customers/abcxy`，并在响应体中包含新创建的客户，如图 16.9 所示：![](img/B17442_17_09.png)

图 16.9：添加新客户

我将留给您一个可选挑战，创建 REST 客户端文件以测试更新客户（使用`PUT`）和删除客户（使用`DELETE`）。尝试对存在和不存在的客户进行操作。解决方案位于本书的 GitHub 仓库中。

既然我们已经看到了一种快速简便的测试服务方法，这同时也是学习 HTTP 的好方法，那么外部开发者呢？我们希望他们学习和调用我们的服务尽可能简单。为此，我们将使用 Swagger。

## 理解 Swagger

Swagger 最重要的部分是**OpenAPI 规范**，它定义了您 API 的 REST 风格契约，详细说明了所有资源和操作，以易于开发、发现和集成的人机可读格式。

开发者可以使用 Web API 的 OpenAPI 规范自动生成其首选语言或库中的强类型客户端代码。

对我们来说，另一个有用的功能是**Swagger UI**，因为它自动为您的 API 生成文档，并内置了可视化测试功能。

让我们回顾一下如何使用`Swashbuckle`包为我们的 Web 服务启用 Swagger：

1.  如果 Web 服务正在运行，请关闭 Web 服务器。

1.  打开`Northwind.WebApi.csproj`并注意`Swashbuckle.AspNetCore`的包引用，如下所示：

    ```cs
    <ItemGroup>
      <PackageReference Include="Swashbuckle.AspNetCore" Version="6.1.5" />
    </ItemGroup> 
    ```

1.  将`Swashbuckle.AspNetCore`包的版本更新至最新，例如，截至 2021 年 9 月撰写时，版本为`6.2.1`。

1.  在`Program.cs`中，注意导入 Microsoft 的 OpenAPI 模型命名空间，如下所示：

    ```cs
    using Microsoft.OpenApi.Models; 
    ```

1.  导入 Swashbuckle 的 SwaggerUI 命名空间，如下所示：

    ```cs
    using Swashbuckle.AspNetCore.SwaggerUI; // SubmitMethod 
    ```

1.  在`Program.cs`大约中间位置，注意添加 Swagger 支持的语句，包括 Northwind 服务的文档，表明这是您服务的第一版，并更改标题，如下所示高亮显示：

    ```cs
    builder.Services.AddSwaggerGen(c =>
      {
        c.SwaggerDoc("v1", new()
          { Title = "**Northwind Service API**", Version = "v1" });
      }); 
    ```

1.  在配置 HTTP 请求管道的部分中，注意在开发模式下使用 Swagger 和 Swagger UI 的语句，并定义 OpenAPI 规范 JSON 文档的端点。

1.  添加代码以明确列出我们希望在 Web 服务中支持的 HTTP 方法，并更改端点名称，如下所示高亮显示：

    ```cs
    var app = builder.Build();
    // Configure the HTTP request pipeline.
    if (builder.Environment.IsDevelopment())
    {
      app.UseSwagger(); 
      app.UseSwaggerUI(c =>
     **{**
     **c.SwaggerEndpoint(****"/swagger/v1/swagger.json"****,**
    **"Northwind Service API Version 1"****);**
     **c.SupportedSubmitMethods(****new****[] {** 
     **SubmitMethod.Get, SubmitMethod.Post,**
     **SubmitMethod.Put, SubmitMethod.Delete });**
     **});**
    } 
    ```

## 使用 Swagger UI 测试请求

现在您已准备好使用 Swagger 测试 HTTP 请求：

1.  启动`Northwind.WebApi` Web 服务。

1.  在 Chrome 中导航至`https://localhost:5001/swagger/`，并注意**Customers**和**WeatherForecast** Web API 控制器已被发现并记录，以及 API 使用的**Schemas**。

1.  点击**GET /api/Customers/{id}**展开该端点，并注意客户**id**所需的参数，如*图 16.10*所示：![](img/B17442_17_10.png)

    图 16.10：在 Swagger 中检查 GET 请求的参数

1.  点击**试用**，输入`ALFKI`作为**ID**，然后点击宽大的蓝色**执行**按钮，如图*16.11*所示:![](img/B17442_17_11.png)

    图 16.11：点击执行按钮前输入客户 ID

1.  向下滚动并注意**请求 URL**、带有**代码**的**服务器响应**以及包含**响应体**和**响应头**的**详细信息**，如图*16.12*所示:![](img/B17442_17_12.png)

    图 16.12：成功 Swagger 请求中关于 ALFKI 的信息

1.  滚动回页面顶部，点击**POST /api/Customers**展开该部分，然后点击**试用**。

1.  点击**请求体**框内，修改 JSON 以定义新客户，如下所示：

    ```cs
    {
      "customerID": "SUPER",
      "companyName": "Super Company",
      "contactName": "Rasmus Ibensen",
      "contactTitle": "Sales Leader",
      "address": "Rotterslef 23",
      "city": "Billund",
      "region": null,
      "postalCode": "4371",
      "country": "Denmark",
      "phone": "31 21 43 21",
      "fax": "31 21 43 22"
    } 
    ```

1.  点击**执行**，并注意**请求 URL**、带有**代码**的**服务器响应**以及包含**响应体**和**响应头**的**详细信息**，注意响应代码为`201`表示客户已成功创建，如图*16.13*所示:![](img/B17442_17_13.png)

    图 16.13：成功添加新客户

1.  滚动回页面顶部，点击**GET /api/Customers**，点击**试用**，输入`Denmark`作为国家参数，点击**执行**，确认新客户已添加到数据库，如图*16.14*所示:![](img/B17442_17_14.png)

    图 16.14：成功获取包括新添加客户在内的丹麦客户

1.  点击**DELETE /api/Customers/{id}**，点击**试用**，输入`super`作为**ID**，点击**执行**，并注意**服务器响应代码**为`204`，表明成功删除，如图*16.15*所示:![](img/B17442_17_15.png)

    图 16.15：成功删除客户

1.  再次点击**执行**，并注意**服务器响应代码**为`404`，表明客户不再存在，**响应体**包含问题详情 JSON 文档，如图*16.16*所示:![](img/B17442_17_16.png)

    图 16.16：已删除的客户不再存在

1.  输入`bad`作为**ID**，再次点击**执行**，并注意**服务器响应代码**为`400`，表明客户确实存在但删除失败（此情况下，因为网络服务模拟此错误），**响应体**包含一个自定义问题详情 JSON 文档，如图*16.17*所示:![](img/B17442_17_17.png)

    图 16.17：客户确实存在但删除失败

1.  使用`GET`方法确认新客户已从数据库中删除（原丹麦仅有两个客户）。

    我将使用`PUT`方法更新现有客户的测试留给读者。

1.  关闭 Chrome 并关闭网络服务器。

## 启用 HTTP 日志记录

HTTP 日志记录是一个可选的中间件组件，它记录有关 HTTP 请求和 HTTP 响应的信息，包括以下内容：

+   HTTP 请求信息

+   头部

+   主体

+   HTTP 响应信息

这在网络服务中对于审计和调试场景非常有价值，但需注意，它可能对性能产生负面影响。你还可能记录**个人身份信息**（**PII**），这在某些司法管辖区可能导致合规问题。

让我们看看 HTTP 日志记录的实际效果：

1.  在`Program.cs`中，导入用于处理 HTTP 日志记录的命名空间，如下列代码所示：

    ```cs
    using Microsoft.AspNetCore.HttpLogging; // HttpLoggingFields 
    ```

1.  在服务配置部分，添加一条配置 HTTP 日志记录的语句，如下列代码所示：

    ```cs
    builder.Services.AddHttpLogging(options =>
    {
      options.LoggingFields = HttpLoggingFields.All;
      options.RequestBodyLogLimit = 4096; // default is 32k
      options.ResponseBodyLogLimit = 4096; // default is 32k
    }); 
    ```

1.  在 HTTP 管道配置部分，添加一条在路由调用前添加 HTTP 日志记录的语句，如下列代码所示：

    ```cs
    app.UseHttpLogging(); 
    ```

1.  启动`Northwind.WebApi`网络服务。

1.  启动 Chrome 浏览器。

1.  导航至`https://localhost:5001/api/customers`。

1.  在命令提示符或终端中，注意请求和响应已被记录，如下列输出所示：

    ```cs
    info: Microsoft.AspNetCore.HttpLogging.HttpLoggingMiddleware[1]
          Request:
          Protocol: HTTP/1.1
          Method: GET
          Scheme: https
          PathBase:
          Path: /api/customers
          QueryString:
          Connection: keep-alive
          Accept: */*
          Accept-Encoding: gzip, deflate, br
          Host: localhost:5001
    info: Microsoft.AspNetCore.HttpLogging.HttpLoggingMiddleware[2]
          Response:
          StatusCode: 200
          Content-Type: application/json; charset=utf-8
          ...
          Transfer-Encoding: chunked 
    ```

1.  关闭 Chrome 并关闭网络服务器。

你现在已准备好构建消费你的网络服务的应用程序。

# 使用 HTTP 客户端消费网络服务

既然我们已经构建并测试了 Northwind 服务，接下来我们将学习如何使用`HttpClient`类及其工厂从任何.NET 应用中调用该服务。

## 理解 HttpClient

最简便的网络服务消费方式是使用`HttpClient`类。然而，许多人错误地使用它，因为它实现了`IDisposable`，且微软的官方文档展示了其不当用法。请参阅 GitHub 仓库中的书籍链接，以获取更多关于此话题的讨论文章。

通常，当类型实现`IDisposable`时，你应该在`using`语句中创建它，以确保其尽快被释放。`HttpClient`则不同，因为它被共享、可重入且部分线程安全。

问题与底层网络套接字的管理方式有关。简而言之，你应该为应用程序生命周期内消费的每个 HTTP 端点使用单一的`HttpClient`实例。这将允许每个`HttpClient`实例设置适合其工作端点的默认值，同时高效管理底层网络套接字。

## 使用 HttpClientFactory 配置 HTTP 客户端

微软已意识到此问题，并在 ASP.NET Core 2.1 中引入了`HttpClientFactory`以鼓励最佳实践；这正是我们将采用的技术。

在下述示例中，我们将以 Northwind MVC 网站作为 Northwind Web API 服务的客户端。由于两者需同时托管于同一网络服务器上，我们首先需要配置它们使用不同的端口号，如下表所示：

+   Northwind Web API 服务将使用`HTTPS`监听端口`5002`。

+   Northwind MVC 网站将继续使用`HTTP`监听端口`5000`，使用`HTTPS`监听端口`5001`。

让我们来配置这些端口：

1.  在`Northwind.WebApi`项目的`Program.cs`中，添加一个对`UseUrls`的扩展方法调用，指定`HTTPS`端口为`5002`，如下列高亮代码所示：

    ```cs
    var builder = WebApplication.CreateBuilder(args);
    **builder.WebHost.UseUrls(****"https://localhost:5002/"****);** 
    ```

1.  在`Northwind.Mvc`项目中，打开`Program.cs`，并导入用于处理 HTTP 客户端工厂的命名空间，如下面的代码所示：

    ```cs
    using System.Net.Http.Headers; // MediaTypeWithQualityHeaderValue 
    ```

1.  添加一条语句以启用`HttpClientFactory`，并使用命名客户端通过 HTTPS 在端口`5002`上调用 Northwind Web API 服务，并请求 JSON 作为默认响应格式，如下面的代码所示：

    ```cs
    builder.Services.AddHttpClient(name: "Northwind.WebApi",
      configureClient: options =>
      {
        options.BaseAddress = new Uri("https://localhost:5002/");
        options.DefaultRequestHeaders.Accept.Add(
          new MediaTypeWithQualityHeaderValue(
          "application/json", 1.0));
      }); 
    ```

## 在控制器中以 JSON 形式获取客户

我们现在可以创建一个 MVC 控制器动作方法，该方法使用工厂创建 HTTP 客户端，发起一个针对客户的`GET`请求，并使用.NET 5 中引入的`System.Net.Http.Json`程序集和命名空间中的便捷扩展方法反序列化 JSON 响应：

1.  打开`Controllers/HomeController.cs`，并声明一个用于存储 HTTP 客户端工厂的字段，如下面的代码所示：

    ```cs
    private readonly IHttpClientFactory clientFactory; 
    ```

1.  在构造函数中设置字段，如下面的代码中突出显示的那样：

    ```cs
    public HomeController(
      ILogger<HomeController> logger,
      NorthwindContext injectedContext**,**
     **IHttpClientFactory httpClientFactory**)
    {
      _logger = logger;
      db = injectedContext;
     **clientFactory = httpClientFactory;**
    } 
    ```

1.  创建一个新的动作方法，用于调用 Northwind Web API 服务，获取所有客户，并将他们传递给一个视图，如下面的代码所示：

    ```cs
    public async Task<IActionResult> Customers(string country)
    {
      string uri;
      if (string.IsNullOrEmpty(country))
      {
        ViewData["Title"] = "All Customers Worldwide";
        uri = "api/customers/";
      }
      else
      {
        ViewData["Title"] = $"Customers in {country}";
        uri = $"api/customers/?country={country}";
      }
      HttpClient client = clientFactory.CreateClient(
        name: "Northwind.WebApi");
      HttpRequestMessage request = new(
        method: HttpMethod.Get, requestUri: uri);
      HttpResponseMessage response = await client.SendAsync(request);
      IEnumerable<Customer>? model = await response.Content
        .ReadFromJsonAsync<IEnumerable<Customer>>();
      return View(model);
    } 
    ```

1.  在`Views/Home`文件夹中，创建一个名为`Customers.cshtml`的 Razor 文件。

1.  修改 Razor 文件以渲染客户，如下面的标记所示：

    ```cs
    @using Packt.Shared
    @model IEnumerable<Customer>
    <h2>@ViewData["Title"]</h2>
    <table class="table">
      <thead>
        <tr>
          <th>Company Name</th>
          <th>Contact Name</th>
          <th>Address</th>
          <th>Phone</th>
        </tr>
      </thead>
      <tbody>
        @if (Model is not null)
        {
          @foreach (Customer c in Model)
          {
            <tr>
              <td>
                @Html.DisplayFor(modelItem => c.CompanyName)
              </td>
              <td>
                @Html.DisplayFor(modelItem => c.ContactName)
              </td>
              <td>
                @Html.DisplayFor(modelItem => c.Address) 
                @Html.DisplayFor(modelItem => c.City)
                @Html.DisplayFor(modelItem => c.Region)
                @Html.DisplayFor(modelItem => c.Country) 
                @Html.DisplayFor(modelItem => c.PostalCode)
              </td>
              <td>
                @Html.DisplayFor(modelItem => c.Phone)
              </td>
            </tr>
          }
        }
      </tbody>
    </table> 
    ```

1.  在`Views/Home/Index.cshtml`中，在渲染访客计数后添加一个表单，允许访客输入一个国家并查看客户，如下面的标记所示：

    ```cs
    <h3>Query customers from a service</h3>
    <form asp-action="Customers" method="get">
      <input name="country" placeholder="Enter a country" />
      <input type="submit" />
    </form> 
    ```

## 启用跨源资源共享

**跨源资源共享**（**CORS**）是一种基于 HTTP 头部的标准，用于保护当客户端和服务器位于不同域（源）时的 Web 资源。它允许服务器指示哪些源（由域、方案或端口的组合定义）除了它自己的源之外，它将允许从这些源加载资源。

由于我们的 Web 服务托管在端口`5002`上，而我们的 MVC 网站托管在端口`5000`和`5001`上，它们被视为不同的源，因此资源不能共享。

在服务器上启用 CORS，并配置我们的 Web 服务，使其仅允许来自 MVC 网站的请求，这将非常有用：

1.  在`Northwind.WebApi`项目中，打开`Program.cs`。

1.  在服务配置部分添加一条语句，以添加对 CORS 的支持，如下面的代码所示：

    ```cs
    builder.Services.AddCors(); 
    ```

1.  在 HTTP 管道配置部分添加一条语句，在调用`UseEndpoints`之前，使用 CORS 并允许来自具有`https://localhost:5001`源的 Northwind MVC 等任何网站的`GET`、`POST`、`PUT`和`DELETE`请求，如下面的代码所示：

    ```cs
    app.UseCors(configurePolicy: options =>
    {
      options.WithMethods("GET", "POST", "PUT", "DELETE");
      options.WithOrigins(
        "https://localhost:5001" // allow requests from the MVC client
      );
    }); 
    ```

1.  启动`Northwind.WebApi`项目，并确认 Web 服务仅在端口`5002`上监听，如下面的输出所示：

    ```cs
    info: Microsoft.Hosting.Lifetime[14]
      Now listening on: https://localhost:5002 
    ```

1.  启动`Northwind.Mvc`项目，并确认网站正在监听端口`5000`和`5002`，如下面的输出所示：

    ```cs
    info: Microsoft.Hosting.Lifetime[14]
      Now listening on: https://localhost:5001
    info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5000 
    ```

1.  启动 Chrome。

1.  在客户表单中，输入一个国家，如`Germany`、`UK`或`USA`，点击**提交**，并注意客户列表，如图*16.18*所示：![](img/B17442_17_18.png)

    图 16.18：英国的客户

1.  点击浏览器中的**返回**按钮，清除国家文本框，点击**提交**，并注意全球客户列表。

1.  在命令提示符或终端中，注意`HttpClient`会记录它发出的每个 HTTP 请求和接收的 HTTP 响应，如下面的输出所示：

    ```cs
    info: System.Net.Http.HttpClient.Northwind.WebApi.ClientHandler[100]
      Sending HTTP request GET https://localhost:5002/api/customers/?country=UK
    info: System.Net.Http.HttpClient.Northwind.WebApi.ClientHandler[101]
      Received HTTP response headers after 931.864ms - 200 
    ```

1.  关闭 Chrome 并关闭网络服务器。

你已成功构建了一个网络服务，并从 MVC 网站中调用了它。

# 为网络服务实现高级功能

既然你已经看到了构建网络服务及其从客户端调用的基础知识，让我们来看看一些更高级的功能。

## 实现健康检查 API

有许多付费服务执行基本的站点可用性测试，如基本 ping，有些则提供更高级的 HTTP 响应分析。

ASP.NET Core 2.2 及更高版本使得实现更详细的网站健康检查变得容易。例如，你的网站可能在线，但它准备好了吗？它能从数据库检索数据吗？

让我们为我们的网络服务添加基本的健康检查功能：

1.  在`Northwind.WebApi`项目中，添加一个项目引用以启用 Entity Framework Core 数据库健康检查，如下面的标记所示：

    ```cs
    <PackageReference Include=  
      "Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore"   
      Version="6.0.0" /> 
    ```

1.  构建项目。

1.  在`Program.cs`中，在服务配置部分的底部，添加一条语句以添加健康检查，包括到 Northwind 数据库上下文，如下面的代码所示：

    ```cs
    builder.Services.AddHealthChecks()
      .AddDbContextCheck<NorthwindContext>(); 
    ```

    默认情况下，数据库上下文检查调用 EF Core 的`CanConnectAsync`方法。你可以通过调用`AddDbContextCheck`方法来自定义运行的操作。

1.  在 HTTP 管道配置部分，在调用`MapControllers`之前，添加一条语句以使用基本健康检查，如下面的代码所示：

    ```cs
    app.UseHealthChecks(path: "/howdoyoufeel"); 
    ```

1.  启动网络服务。

1.  启动 Chrome。

1.  导航到`https://localhost:5002/howdoyoufeel`并注意网络服务以纯文本响应：`Healthy`。

1.  在命令提示符或终端中，注意用于测试数据库健康状况的 SQL 语句，如下面的输出所示：

    ```cs
    Level: Debug, Event Id: 20100, State: Executing DbCommand [Parameters=[], CommandType='Text', CommandTimeout='30']
    SELECT 1 
    ```

1.  关闭 Chrome 并关闭网络服务器。

## 实现 Open API 分析器和约定

在本章中，你学习了如何通过手动使用属性装饰控制器类来启用 Swagger 以记录网络服务。

在 ASP.NET Core 2.2 或更高版本中，有 API 分析器会反射带有`[ApiController]`属性的控制器类来自动记录它。分析器假设了一些 API 约定。

要使用它，你的项目必须启用 OpenAPI 分析器，如下面的标记中突出显示的那样：

```cs
<PropertyGroup>
  <TargetFramework>net6.0</TargetFramework>
  <Nullable>enable</Nullable>
  <ImplicitUsings>enable</ImplicitUsings>
 **<IncludeOpenAPIAnalyzers>****true****</IncludeOpenAPIAnalyzers>**
</PropertyGroup> 
```

安装后，未正确装饰的控制器应显示警告（绿色波浪线），并在编译源代码时发出警告。例如，`WeatherForecastController`类。

自动代码修复随后可以添加适当的`[Produces]`和`[ProducesResponseType]`属性，尽管这在当前仅适用于 Visual Studio。在 Visual Studio Code 中，您将看到分析器认为您应该添加属性的警告，但您必须手动添加它们。

## 实现瞬态故障处理

当客户端应用或网站调用 Web 服务时，可能来自世界的另一端。客户端与服务器之间的网络问题可能导致与您的实现代码无关的问题。如果客户端发起调用失败，应用不应就此放弃。如果它尝试再次调用，问题可能已经解决。我们需要一种方法来处理这些临时故障。

为了处理这些瞬态故障，微软建议您使用第三方库 Polly 来实现带有指数退避的自动重试。您定义一个策略，库将处理其余所有事务。

**最佳实践**：您可以在以下链接了解更多关于 Polly 如何使您的 Web 服务更可靠的信息：[`docs.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly`](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)

## 添加安全 HTTP 头部

ASP.NET Core 内置了对常见安全 HTTP 头部（如 HSTS）的支持。但还有许多其他 HTTP 头部您应考虑实现。

添加这些头部的最简单方法是使用中间件类：

1.  在`Northwind.WebApi`项目/文件夹中，创建一个名为`SecurityHeadersMiddleware.cs`的文件，并修改其语句，如下所示：

    ```cs
    using Microsoft.Extensions.Primitives; // StringValues
    public class SecurityHeaders
    {
      private readonly RequestDelegate next;
      public SecurityHeaders(RequestDelegate next)
      {
        this.next = next;
      }
      public Task Invoke(HttpContext context)
      {
        // add any HTTP response headers you want here
        context.Response.Headers.Add(
          "super-secure", new StringValues("enable"));
        return next(context);
      }
    } 
    ```

1.  在`Program.cs`中，在 HTTP 管道配置部分，添加一条语句，在调用`UseEndpoints`之前注册中间件，如下所示：

    ```cs
    app.UseMiddleware<SecurityHeaders>(); 
    ```

1.  启动 Web 服务。

1.  启动 Chrome。

1.  显示**开发者工具**及其**网络**标签以记录请求和响应。

1.  导航至`https://localhost:5002/weatherforecast`。

1.  注意我们添加的自定义 HTTP 响应头部，名为`super-secure`，如*图 16.19*所示：![](img/B17442_17_19.png)

    *图 16.19*：添加名为 super-secure 的自定义 HTTP 头部

# 使用最小 API 构建 Web 服务

对于.NET 6，微软投入了大量精力为 C# 10 语言添加新特性，并简化 ASP.NET Core 库，以实现使用最小 API 创建 Web 服务。

您可能还记得 Web API 项目模板中提供的天气预报服务。它展示了使用控制器类返回使用假数据的五天天气预报。我们现在将使用最小 API 重现该天气服务。

首先，天气服务有一个类来表示单个天气预报。我们将在多个项目中需要使用这个类，所以让我们为此创建一个类库：

1.  使用您喜欢的代码编辑器添加一个新项目，如下列清单所定义：

    1.  项目模板：**类库** / `classlib`

    1.  工作区/解决方案文件和文件夹：`PracticalApps`

    1.  项目文件和文件夹：`Northwind.Common`

1.  将`Class1.cs`重命名为`WeatherForecast.cs`。

1.  修改`WeatherForecast.cs`，如下面的代码所示：

    ```cs
    namespace Northwind.Common
    {
      public class WeatherForecast
      {
        public static readonly string[] Summaries = new[]
        {
          "Freezing", "Bracing", "Chilly", "Cool", "Mild",
          "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
        };
        public DateTime Date { get; set; }
        public int TemperatureC { get; set; }
        public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
        public string? Summary { get; set; }
      }
    } 
    ```

## 使用最小 API 构建天气服务

现在让我们使用最小 API 重新创建该天气服务。它将在端口`5003`上监听，并启用 CORS 支持，以便请求只能来自 MVC 网站，并且只允许`GET`请求：

1.  使用您喜欢的代码编辑器添加一个新项目，如下列清单所定义：

    1.  项目模板：**ASP.NET Core 空** / `web`

    1.  工作区/解决方案文件和文件夹：`PracticalApps`

    1.  项目文件和文件夹：`Minimal.WebApi`

    1.  其他 Visual Studio 选项：**身份验证类型**：无，**为 HTTPS 配置**：已选中，**启用 Docker**：已清除，**启用 OpenAPI 支持**：已选中。

1.  在 Visual Studio Code 中，选择`Minimal.WebApi`作为活动的 OmniSharp 项目。

1.  在`Minimal.WebApi`项目中，添加一个项目引用指向`Northwind.Common`项目，如下面的标记所示：

    ```cs
    <ItemGroup>
      <ProjectReference Include="..\Northwind.Common\Northwind.Common.csproj" />
    </ItemGroup> 
    ```

1.  构建`Minimal.WebApi`项目。

1.  修改`Program.cs`，如下面的代码中突出显示的那样：

    ```cs
    **using** **Northwind.Common;** **// WeatherForecast**
    var builder = WebApplication.CreateBuilder(args);
    **builder.WebHost.UseUrls(****"https://localhost:5003"****);**
    **builder.Services.AddCors();**
    var app = builder.Build();
    **// only allow the MVC client and only GET requests**
    **app.UseCors(configurePolicy: options =>**
    **{**
     **options.WithMethods(****"GET"****);**
     **options.WithOrigins(****"https://localhost:5001"****);**
    **});**
    **app.MapGet(****"/api/weather"****, () =>** 
    **{**
    **return** **Enumerable.Range(****1****,** **5****).Select(index =>**
    **new** **WeatherForecast**
     **{**
     **Date = DateTime.Now.AddDays(index),**
     **TemperatureC = Random.Shared.Next(****-20****,** **55****),**
     **Summary = WeatherForecast.Summaries[**
     **Random.Shared.Next(WeatherForecast.Summaries.Length)]**
     **})**
     **.ToArray();**
    **});**
    app.Run(); 
    ```

    **良好实践**：对于简单的 Web 服务，避免创建控制器类，而是使用最小 API 将所有配置和实现放在一个地方，即`Program.cs`。

1.  在**属性**中，修改`launchSettings.json`以配置`Minimal.WebApi`配置文件，使其通过 URL 中的端口`5003`启动浏览器，如下面的标记中突出显示的那样：

    ```cs
    "profiles": {
      "Minimal.WebApi": {
        "commandName": "Project",
        "dotnetRunMessages": "true",
        "launchBrowser": true,
    **"applicationUrl"****:** **"https://localhost:5003/api/weather"****,**
        "environmentVariables": {
          "ASPNETCORE_ENVIRONMENT": "Development"
        } 
    ```

## 测试最小天气服务

在创建服务客户端之前，让我们测试它是否返回 JSON 格式的预报：

1.  启动 Web 服务项目。

1.  如果你没有使用 Visual Studio 2022，请启动 Chrome 并导航至`https://localhost:5003/api/weather`。

1.  注意 Web API 服务应返回一个包含五个随机天气预报对象的 JSON 文档数组。

1.  关闭 Chrome 并关闭 Web 服务器。

## 向 Northwind 网站首页添加天气预报

最后，让我们向 Northwind 网站添加一个 HTTP 客户端，以便它可以调用天气服务并在首页显示预报：

1.  在`Northwind.Mvc`项目中，添加一个项目引用指向`Northwind.Common`，如下面的标记中突出显示的那样：

    ```cs
    <ItemGroup>
      <!-- change Sqlite to SqlServer if you prefer -->
      <ProjectReference Include="..\Northwind.Common.DataContext.Sqlite\Northwind.Common.DataContext.Sqlite.csproj" />
     **<ProjectReference Include=****"..\Northwind.Common\Northwind.Common.csproj"** **/>**
    </ItemGroup> 
    ```

1.  在`Program.cs`中，添加一条语句以配置 HTTP 客户端以调用端口`5003`上的最小服务，如下面的代码所示：

    ```cs
    builder.Services.AddHttpClient(name: "Minimal.WebApi",
      configureClient: options =>
      {
        options.BaseAddress = new Uri("https://localhost:5003/");
        options.DefaultRequestHeaders.Accept.Add(
          new MediaTypeWithQualityHeaderValue(
          "application/json", 1.0));
      }); 
    ```

1.  在`HomeController.cs`中，导入`Northwind.Common`命名空间，并在`Index`方法中，添加语句以获取并使用 HTTP 客户端调用天气服务以获取预报并将其存储在`ViewData`中，如下面的代码所示：

    ```cs
    try
    {
      HttpClient client = clientFactory.CreateClient(
        name: "Minimal.WebApi");
      HttpRequestMessage request = new(
        method: HttpMethod.Get, requestUri: "api/weather");
      HttpResponseMessage response = await client.SendAsync(request);
      ViewData["weather"] = await response.Content
        .ReadFromJsonAsync<WeatherForecast[]>();
    }
    catch (Exception ex)
    {
      _logger.LogWarning($"The Minimal.WebApi service is not responding. Exception: {ex.Message}");
      ViewData["weather"] = Enumerable.Empty<WeatherForecast>().ToArray();
    } 
    ```

1.  在`Views/Home`中，在`Index.cshtml`中，导入`Northwind.Common`命名空间，然后在顶部代码块中从`ViewData`字典获取天气预报，如下面的标记所示：

    ```cs
    @{
      ViewData["Title"] = "Home Page";
      string currentItem = "";
     **WeatherForecast[]? weather = ViewData[****"weather"****]** **as** **WeatherForecast[];**
    } 
    ```

1.  在第一个`<div>`中，在渲染当前时间后，除非没有天气预报，否则添加标记以枚举天气预报，并以表格形式呈现，如下所示：

    ```cs
    <p>
      <h4>Five-Day Weather Forecast</h4>
      @if ((weather is null) || (!weather.Any()))
      {
        <p>No weather forecasts found.</p>
      }
      else
      {
      <table class="table table-info">
        <tr>
          @foreach (WeatherForecast w in weather)
          {
            <td>@w.Date.ToString("ddd d MMM") will be @w.Summary</td>
          }
        </tr>
      </table>
      }
    </p> 
    ```

1.  启动`Minimal.WebApi`服务。

1.  启动`Northwind.Mvc`网站。

1.  导航至`https://localhost:5001/`，并注意天气预报，如图*16.20*所示：![](img/B17442_17_20.png)

    图 16.20：Northwind 网站主页上的五天天气预报

1.  查看 MVC 网站的命令提示符或终端，并注意指示请求已发送到最小 API Web 服务`api/weather`端点的信息消息，大约耗时 83ms，如下所示：

    ```cs
    info: System.Net.Http.HttpClient.Minimal.WebApi.LogicalHandler[100]
          Start processing HTTP request GET https://localhost:5003/api/weather
    info: System.Net.Http.HttpClient.Minimal.WebApi.ClientHandler[100]
          Sending HTTP request GET https://localhost:5003/api/weather
    info: System.Net.Http.HttpClient.Minimal.WebApi.ClientHandler[101]
          Received HTTP response headers after 76.8963ms - 200
    info: System.Net.Http.HttpClient.Minimal.WebApi.LogicalHandler[101]
          End processing HTTP request after 82.9515ms – 200 
    ```

1.  停止`Minimal.WebApi`服务，刷新浏览器，并注意几秒后 MVC 网站主页出现，但没有天气预报。

1.  关闭 Chrome 并关闭 Web 服务器。

# 实践与探索

通过回答一些问题测试您的知识和理解，进行一些实践练习，并深入研究本章的主题。

## 练习 16.1 – 测试您的知识

回答以下问题：

1.  为了创建 ASP.NET Core Web API 服务的控制器类，您应该继承自哪个类？

1.  如果您用`[ApiController]`属性装饰控制器类以获得默认行为，如对无效模型自动返回`400`响应，还需要做什么？

1.  指定哪个控制器操作方法将执行以响应 HTTP 请求，您必须做什么？

1.  指定调用操作方法时应预期哪些响应，您必须做什么？

1.  列出三种可以调用的方法，以返回具有不同状态码的响应。

1.  列出四种测试 Web 服务的方法。

1.  尽管`HttpClient`实现了`IDisposable`接口，为何不应在`using`语句中包裹其使用以在完成时释放它，以及应使用什么替代方案？

1.  CORS 缩写代表什么，为何在 Web 服务中启用它很重要？

1.  如何在 ASP.NET Core 2.2 及更高版本中使客户端能够检测您的 Web 服务是否健康？

1.  端点路由提供了哪些好处？

## 练习 16.2 – 使用 HttpClient 练习创建和删除客户

扩展`Northwind.Mvc`网站项目，使其拥有页面，访客可以在其中填写表单以创建新客户，或搜索客户并删除他们。MVC 控制器应调用 Northwind Web 服务来创建和删除客户。

## 练习 16.3 – 探索主题

使用以下页面上的链接，深入了解本章涵盖的主题：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-16---building-and-consuming-web-services`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-16---building-and-consuming-web-services)

# 总结

本章中，你学习了如何构建一个 ASP.NET Core Web API 服务，该服务可被任何能够发起 HTTP 请求并处理 HTTP 响应的平台上的应用调用。

你还学习了如何使用 Swagger 测试和文档化 Web 服务 API，以及如何高效地消费这些服务。

下一章，你将学习使用 Blazor 构建用户界面，这是微软推出的酷炫新技术，让开发者能用 C#而非 JavaScript 来构建网站的客户端单页应用（SPAs）、桌面混合应用，以及潜在的移动应用。
