# 第十六章：构建和消费 Web 服务

本章是关于学习如何使用 ASP.NET Core Web API 构建 Web 服务（也称为 HTTP 或 REST 服务），以及使用 HTTP 客户端消费 Web 服务的内容，这些客户端可以是任何其他类型的.NET 应用程序，包括网站、移动应用程序或桌面应用程序。

本章需要你在*第十章*、*使用 Entity Framework Core 处理数据*和*第 13 到 15 章*中学到的 C#和.NET 的实际应用知识和技能，以及使用 ASP.NET Core 构建网站的知识。

在本章中，我们将涵盖以下主题：

+   使用 ASP.NET Core Web API 构建 Web 服务

+   文档化和测试 Web 服务

+   使用 HTTP 客户端消费 Web 服务

+   实现 Web 服务的高级功能

+   使用最小 API 构建 Web 服务

# 使用 ASP.NET Core Web API 构建 Web 服务

在构建现代 Web 服务之前，我们需要了解一些背景知识，以便为本章设定背景。

## 理解 Web 服务的首字母缩写

尽管 HTTP 最初是设计用于请求和响应 HTML 和其他资源供人类查看，但它也非常适合构建服务。

Roy Fielding 在他的博士论文中描述了**表现状态转移**（**REST**）架构风格，指出 HTTP 标准非常适合构建服务，因为它定义了以下内容：

+   用 URI 唯一标识资源，比如`https://localhost:5001/api/products/23`。

+   执行这些资源上的常见任务的方法，比如`GET`、`POST`、`PUT`和`DELETE`。

+   能够协商请求和响应中交换的内容的媒体类型，比如 XML 和 JSON。内容协商发生在客户端指定请求头像`Accept: application/xml,*/*;q=0.8`。ASP.NET Core Web API 使用的默认响应格式是 JSON，这意味着响应头之一将是`Content-Type: application/json; charset=utf-8`。

**Web 服务**使用 HTTP 通信标准，因此有时被称为 HTTP 或 RESTful 服务。HTTP 或 RESTful 服务就是本章的内容。

Web 服务也可以指实现一些**WS-*标准**的**简单对象访问协议**（**SOAP**）服务。这些标准使得在不同系统上实现的客户端和服务能够相互通信。WS-*标准最初由 IBM 定义，并得到了微软等其他公司的支持。

### 理解 Windows Communication Foundation（WCF）

.NET Framework 3.0 及更高版本包括名为**Windows Communication Foundation**（**WCF**）的**远程过程调用**（**RPC**）技术。RPC 技术使一个系统上的代码能够在网络上执行另一个系统上的代码。

WCF 使开发人员能够轻松创建服务，包括实现 WS-*标准的 SOAP 服务。它后来也支持构建 Web/HTTP/REST 风格的服务，但如果这是你所需要的全部内容，那么它可能有点过度设计。

如果你有现有的 WCF 服务，并且想要将它们迁移到现代.NET，那么在 2021 年 2 月有一个开源项目发布了首个**正式版本**（**GA**）。你可以在以下链接阅读相关信息：

[`corewcf.github.io/blog/2021/02/19/corewcf-ga-release`](https://corewcf.github.io/blog/2021/02/19/corewcf-ga-release)

### WCF 的替代方案

微软推荐的 WCF 替代方案是**gRPC**。gRPC 是由 Google 创建的现代跨平台开源 RPC 框架（非官方的 gRPC 中的"g"）。你将在*第十八章*中了解更多关于 gRPC 的内容，*构建和使用专门的服务*（可在[`github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf`](https://github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf)找到）。

## 理解 Web API 的 HTTP 请求和响应

HTTP 定义了标准类型的请求和标准代码来指示响应的类型。其中大多数可以用于实现 Web API 服务。

最常见的请求类型是`GET`，用于检索由唯一路径标识的资源，还可以设置额外选项，例如可接受的媒体类型，设置为请求标头，如下例所示：

```cs

GET /path/to/resource

Accept: application/json

```

常见的响应包括成功和多种类型的失败，如下表所示：

| 状态码 | 描述 |
| --- | --- |
| `200 OK` | 路径正确形成，成功找到资源，序列化为可接受的媒体类型，然后在响应主体中返回。响应标头指定了`Content-Type`，`Content-Length`和`Content-Encoding`，例如 GZIP。 |
| `301 Moved Permanently` | 随着时间的推移，Web 服务可能会更改其资源模型，包括用于标识现有资源的路径。Web 服务可以通过返回此状态代码和名为`Location`的响应标头指示新路径。 |
| `302 Found` | 类似于`301`。 |
| `304 Not Modified` | 如果请求包括`If-Modified-Since`标头，则 Web 服务可以使用此状态代码响应。响应主体为空，因为客户端应使用其缓存的资源副本。 |
| `400 Bad Request` | 请求无效，例如，使用整数 ID 的产品路径，其中 ID 值丢失。 |
| `401 Unauthorized` | 请求有效，找到资源，但客户端未提供凭据或未被授权访问该资源。重新进行身份验证可能会启用访问，例如通过添加或更改`Authorization`请求标头。 |
| `403 Forbidden` | 请求有效，找到资源，但客户端未被授权访问该资源。重新进行身份验证将无法解决问题。 |
| `404 Not Found` | 请求有效，但未找到资源。如果稍后重复请求，可能会找到资源。要指示永远找不到资源，请返回`410 Gone`。 |
| `406 Not Acceptable` | 如果请求具有仅列出 Web 服务不支持的媒体类型的`Accept`标头。例如，如果客户端请求 JSON，但 Web 服务只能返回 XML。 |
| `451 Unavailable for Legal Reasons` | 美国托管的网站可能会针对来自欧洲的请求返回此代码，以避免不得不遵守《通用数据保护条例》（GDPR）。选择该数字是为了参考小说《华氏 451 度》，其中书籍被禁止和焚烧。 |
| `500 Server Error` | 请求有效，但在处理请求时服务器端出现了问题。稍后重试可能会起作用。 |
| `503 Service Unavailable` | Web 服务繁忙，无法处理请求。稍后再试可能会起作用。 |

其他常见类型的 HTTP 请求包括`POST`，`PUT`，`PATCH`或`DELETE`，用于创建，修改或删除资源。

要创建新资源，您可以使用包含新资源的主体进行`POST`请求，如下所示：

```cs

POST /path/to/resource

Content-Length: 123

Content-Type: application/json

```

创建新资源或更新现有资源，您可以使用包含现有资源的全新版本的主体进行`PUT`请求，如果资源不存在，则创建该资源，如果存在，则替换（有时称为**upsert**操作），如下所示：

```cs

PUT /path/to/resource

Content-Length: 123

Content-Type: application/json

```

要更有效地更新现有资源，您可以使用包含仅需要更改的属性的对象的主体进行`PATCH`请求，如下所示：

```cs

PATCH /path/to/resource

Content-Length: 123

Content-Type: application/json

```

要删除现有资源，您可以发出`DELETE`请求，如下面的代码所示：

```cs

DELETE /path/to/resource

```

以及上表中显示的`GET`请求的响应外，所有创建，修改或删除资源的请求类型都有额外可能的常见响应，如下表所示：

| 状态码 | 描述 |
| --- | --- |
| `201 Created` | 新资源已成功创建，响应头名为`Location`包含其路径，响应体包含新创建的资源。立即`GET`资源应返回`200`。 |
| `202 Accepted` | 新资源无法立即创建，因此请求被排队等待后续处理，并且立即`GET`资源可能会返回`404`。响应体可以包含指向某种状态检查器或资源何时可用的估计的资源。 |
| `204 No Content` | 通常用于对`DELETE`请求的响应，因为在删除资源后将其返回到响应体中通常没有意义！有时用于对`POST`，`PUT`或`PATCH`请求的响应，如果客户端不需要确认请求是否被正确处理。 |
| `405 Method Not Allowed` | 当请求使用不受支持的方法时返回。例如，设计为只读的 Web 服务可能明确禁止`PUT`，`DELETE`等。 |
| `415 Unsupported Media Type` | 当请求体中的资源使用 Web 服务无法处理的媒体类型时返回。例如，如果请求体包含 XML 格式的资源，但 Web 服务只能处理 JSON。 |

## 创建 ASP.NET Core Web API 项目

我们将构建一个 Web 服务，该服务提供了一种使用 ASP.NET Core 处理 Northwind 数据库中的数据的方式，以便数据可以被任何可以发出 HTTP 请求并接收 HTTP 响应的客户端应用程序在任何平台上使用：

1.  使用您喜欢的代码编辑器添加一个新项目，如下列表中定义的：

1.  项目模板：**ASP.NET Core Web API** / `webapi`

1.  工作空间/解决方案文件和文件夹：`PracticalApps`

1.  项目文件和文件夹：`Northwind.WebApi`

1.  其他 Visual Studio 选项：**身份验证类型**：无，**配置为 HTTPS**：已选择，**启用 Docker**：已清除，**启用 OpenAPI 支持**：已选择。

1.  在 Visual Studio Code 中，将`Northwind.WebApi`选择为活动的 OmniSharp 项目。

1.  构建`Northwind.WebApi`项目。

1.  在`Controllers`文件夹中，打开并查看`WeatherForecastController.cs`，如下面的代码所示：

```cs

使用

Microsoft.AspNetCore.Mvc;

命名空间

Northwind.WebApi.Controllers

;

[ApiController

]

[Route(

"[controller]"

)

]

公共的

类

WeatherForecastController

: ControllerBase

{

私人的

静态的

只读的

字符串

[] Summaries = new

[]

{

"冰冻"

，"令人振奋"

，"寒冷"

，"凉爽"

，"温和"

，

"温暖"

，"温暖"

，"热"

，"酷热"

，"灼热"

};

私人的

只读的

ILogger<WeatherForecastController> _logger;

公共的

WeatherForecastController

(

ILogger<WeatherForecastController> logger

)

{

_logger = logger;

}

[HttpGet

]

公共的

IEnumerable<WeatherForecast>

获取

()

{

返回

Enumerable.Range(1

，5

).Select(index =>

新的

WeatherForecast

{

日期= DateTime.Now.AddDays(index),

TemperatureC = Random.Shared.Next(-20

，55

),

摘要= Summaries[Random.Shared.Next(Summaries.Length)]

})

.ToArray();

}

}

```

在审查上述代码时，请注意以下内容：

+   `Controller`类继承自`ControllerBase`。这比 MVC 中使用的`Controller`类更简单，因为它没有像`View`这样的方法，通过将视图模型传递给 Razor 文件来生成 HTML 响应。

+   `[Route]`属性为客户端注册了相对 URL`/weatherforecast`，以便通过该控制器处理 HTTP 请求。例如，对`https://localhost:5001/weatherforecast/`的 HTTP 请求将由该控制器处理。一些开发人员喜欢在控制器名称前加上`api/`，这是一个在混合项目中区分 MVC 和 Web API 的约定。如果像所示使用`[controller]`，它将使用类名中`Controller`之前的字符，例如`WeatherForecast`，或者您可以简单地输入一个不带方括号的不同名称，例如`[Route("api/forecast")]`。

+   `[ApiController]`属性是在 ASP.NET Core 2.1 中引入的，它为控制器启用了特定于 REST 的行为，例如对无效模型的自动 HTTP`400`响应，您将在本章后面看到。

+   `[HttpGet]`属性将`Controller`类中的`Get`方法注册为响应 HTTP`GET`请求，并且其实现使用共享的`Random`对象返回具有随机温度和摘要的`WeatherForecast`对象数组，例如`Bracing`或`Balmy`，用于未来五天的天气。

1.  添加第二个`Get`方法，允许调用指定预测提前多少天，实现如下：

+   在原始方法上方添加注释，显示它响应的操作方法和 URL 路径。

+   添加一个名为`days`的整数参数的新方法。

+   剪切并粘贴原始的`Get`方法实现代码语句到新的`Get`方法中。

+   修改新方法以创建一个到请求的天数的整数的`IEnumerable`，并修改原始的`Get`方法调用新的`Get`方法并传递值`5`。

您的方法应如下代码中所示突出显示：

```cs

**// GET /weatherforecast**

[HttpGet

]

公共

IEnumerable<WeatherForecast>

获取

()

**// 原始方法**

{

**返回**

**Get(**

**5**

**);**

**// 五天预测**

}

**// GET /weatherforecast/7**

**[**

**HttpGet(**

**"{days:int}"**

**)**

**]**

**公共**

**IEnumerable<WeatherForecast>**

**获取**

**(**

**int**

**days**

**)**

**// 新方法**

{

**返回**

**Enumerable.Range(**

**1**

**, days).Select(index =>**

新

WeatherForecast

{

Date = DateTime.Now.AddDays(index),

TemperatureC = Random.Shared.Next(-20

, 55

),

摘要 = Summaries[Random.Shared.Next(Summaries.Length)]

})

.ToArray();

}

```

在`[HttpGet]`属性中，注意路由格式模式`{days:int}`，它将`days`参数限制为`int`值。

## 审查网络服务的功能

现在，我们将测试网络服务的功能：

1.  如果您使用 Visual Studio，在**属性**中，打开`launchSettings.json`文件，并注意默认情况下，它将启动浏览器并导航到`/swagger`相对 URL 路径，如下面的标记所示：

```cs

"profiles"

: {

"Northwind.WebApi"

: {

"commandName"

: "项目"

,

"dotnetRunMessages"

: "true"

,

**"launchBrowser"**

**:**

**true**

**,**

**"launchUrl"**

**:**

**"swagger"**

**,**

"applicationUrl"

: "https://localhost:5001;http://localhost:5000"

,

"environmentVariables"

: {

"ASPNETCORE_ENVIRONMENT"

: "开发"

}

},

```

1.  修改名为`Northwind.WebApi`的配置文件，将`launchBrowser`设置为`false`。

1.  对于`applicationUrl`，将`HTTP`的随机端口号更改为`5000`，将`HTTPS`的随机端口号更改为`5001`。

1.  启动网络服务项目。

1.  启动 Chrome。

1.  导航到`https://localhost:5001/`，注意您将收到`404`状态码响应，因为我们尚未启用静态文件，也没有`index.html`，也没有配置路由的 MVC 控制器。请记住，此项目不是为人类查看和交互而设计的，因此这是 Web 服务的预期行为。

GitHub 上的解决方案配置为使用端口`5002`，因为我们将在本书中稍后更改其配置。

1.  在 Chrome 中，显示**开发者工具**。

1.  转到`https://localhost:5001/weatherforecast`，并注意 Web API 服务应返回一个包含五个随机天气预报对象的 JSON 文档数组，如*图 16.1*所示：！[](img/Image00129.jpg)

图 16.1：天气预报 Web 服务的请求和响应

1.  关闭**开发人员工具**。

1.  转到`https://localhost:5001/weatherforecast/14`，并注意在请求两周天气预报时的响应，如*图 16.2*所示：！[](img/Image00130.jpg)

图 16.2：作为 JSON 文档的两周天气预报

1.  关闭 Chrome 并关闭 Web 服务器。

## 为 Northwind 数据库创建 Web 服务

与 MVC 控制器不同，Web API 控制器不会调用 Razor 视图以返回网站访问者在浏览器中看到的 HTML 响应。相反，它们使用内容协商与发出 HTTP 请求的客户端应用程序返回数据，格式如 XML，JSON 或 X-WWW-FORM-URLENCODED。

然后，客户端应用程序必须从协商的格式中反序列化数据。现代 Web 服务最常用的格式是**JavaScript 对象表示法**（**JSON**），因为它紧凑且在使用客户端端技术（如 Angular，React 和 Vue）构建**单页应用程序**（**SPA**）时与 JavaScript 原生工作。

我们将引用您在*第十三章*，*介绍 C＃和.NET 的实际应用*中创建的 Northwind 数据库的 Entity Framework Core 实体数据模型：

1.  在`Northwind.WebApi`项目中，根据以下标记向`Northwind.Common.DataContext`添加对 SQLite 或 SQL Server 的项目引用：

```cs

<ItemGroup>

<!--如果更改 Sqlite 为 SqlServer

您可以选择的-->

<ProjectReference Include =

“..\ Northwind.Common.DataContext.Sqlite \ Northwind.Common.DataContext.Sqlite.csproj”

/>

</ItemGroup>

```

1.  构建项目并修复代码中的任何编译错误。

1.  打开`Program.cs`并导入用于处理 Web 媒体格式化程序和共享 Packt 类的命名空间，如下所示：

```cs

使用

Microsoft.AspNetCore.Mvc.Formatters;

使用

Packt.Shared; // AddNorthwindContext 扩展方法

使用

静态

System.Console;

```

1.  在调用`AddControllers`之前添加一个语句，以注册`Northwind`数据库上下文类（它将使用 SQLite 或 SQL Server，具体取决于您在项目文件中引用的数据库提供程序），如下所示：”

```cs

//向容器添加服务。

builder.Services.AddNorthwindContext（）;

```

1.  在对`AddControllers`的调用中，添加一个 lambda 块，其中包含语句，以将默认输出格式化程序的名称和支持的媒体类型写入控制台，然后添加 XML 序列化程序格式化程序，如下所示：

```cs

builder.Services.AddControllers（选项=>

{

WriteLine（“默认输出格式化程序：”

）;

foreach

（IOutputFormatter formatter in

options.OutputFormatters）

{

OutputFormatter？mediaFormatter =格式化程序为

OutputFormatter;

如果

（mediaFormatter == null

）

{

WriteLine（$“

{formatter.GetType（）。Name}

“

）;

}

否则

// OutputFormatter 类有 SupportedMediaTypes

{

WriteLine（“ {0}，媒体类型：{1}”

，

arg0：mediaFormatter.GetType（）。Name，

参数 1：字符串

.Join（“，”

，

mediaFormatter.SupportedMediaTypes））;

}

}

}）

.AddXmlDataContractSerializerFormatters（）

.AddXmlSerializerFormatters（）;

```

1.  启动 Web 服务。

1.  在命令提示符或终端中，请注意有四个默认输出格式化程序，包括将`null`值转换为`204 No Content`的格式化程序，以及支持纯文本，字节流和 JSON 响应的格式化程序，如下所示：

```cs

默认输出格式化程序：

HttpNoContentOutputFormatter

StringOutputFormatter，媒体类型：text / plain

StreamOutputFormatter

SystemTextJsonOutputFormatter，媒体类型：application / json，text / json，application / * + json

```

1.  关闭 Web 服务器。

## 为实体创建数据存储库

定义和实现数据存储库以提供 CRUD 操作是一个良好的实践。CRUD 缩写包括以下操作：

+   C for Create

+   R for Retrieve (or Read)

+   U for Update

+   D for Delete

我们将为 Northwind 的`Customers`表创建一个数据存储库。这个表中只有 91 个客户，所以我们将在内存中存储整个表的副本，以提高读取客户记录的可伸缩性和性能。

**良好的实践**：在真实的 Web 服务中，您应该使用像 Redis 这样的分布式缓存，它是一个开源的数据结构存储，可以用作高性能、高可用性的数据库、缓存或消息代理。

我们将遵循现代良好的实践，并使存储库 API 异步化。它将由`Controller`类使用构造函数参数注入实例化，因此将创建一个新实例来处理每个 HTTP 请求：

1.  在`Northwind.WebApi`项目中，创建一个名为`Repositories`的文件夹。

1.  将两个类文件添加到`Repositories`文件夹中，命名为`ICustomerRepository.cs`和`CustomerRepository.cs`。

1.  `ICustomerRepository`接口将定义五种方法，如下面的代码所示：

```cs

using

Packt.Shared; // Customer

namespace

Northwind.WebApi.Repositories

;

public

interface

ICustomerRepository

{

Task<Customer?> CreateAsync(Customer c);

Task<IEnumerable<Customer>> RetrieveAllAsync();

Task<Customer?> RetrieveAsync(string

id);

Task<Customer?> UpdateAsync(string

id, Customer c);

Task<bool

?> DeleteAsync(string

id);

}

```

1.  `CustomerRepository`类将实现这五种方法，记住使用`await`的方法必须标记为`async`，如下面的代码所示：

```cs

using

Microsoft.EntityFrameworkCore.ChangeTracking; // EntityEntry<T>

using

Packt.Shared; // Customer

using

System.Collections.Concurrent; // ConcurrentDictionary

namespace

Northwind.WebApi.Repositories

;

public

class

CustomerRepository

: ICustomerRepository

{

// 使用静态线程安全的字典字段缓存客户

private

static

ConcurrentDictionary

<string

, Customer>? customersCache;

// 使用实例数据上下文字段，因为它不应该是

// 由于其内部缓存而被缓存

private

NorthwindContext db;

public

CustomerRepository

(

NorthwindContext injectedContext

)

{

db = injectedContext;

// 从数据库中预加载客户作为正常

// 以 CustomerId 为键的字典，

// 然后转换为线程安全的 ConcurrentDictionary

if

(customersCache is

null

)

{

customersCache = new

ConcurrentDictionary<string

, Customer>(

db.Customers.ToDictionary(c => c.CustomerId));

}

}

public

async

Task<Customer?>

CreateAsync

(

Customer c

)

{

// 规范化 CustomerId 为大写

c.CustomerId = c.CustomerId.ToUpper();

// 使用 EF Core 添加到数据库

EntityEntry<Customer> added = await

db.Customers.AddAsync(c);

int

affected = await

db.SaveChangesAsync();

if

(affected == 1

)

{

if

(customersCache is

null

) return

c;

// 如果客户是新的，则将其添加到缓存，否则

// 调用 UpdateCache 方法

return

customersCache.AddOrUpdate(c.CustomerId, c, UpdateCache);

}

else

{

return

null

;

}

}

public

Task<IEnumerable<Customer>> RetrieveAllAsync()

{

// 为了性能，从缓存中获取

return

Task.FromResult(customersCache is

null

? Enumerable.Empty<Customer>() : customersCache.Values);

}

public

Task<Customer?> RetrieveAsync(string

id)

{

// 为了性能，从缓存中获取

id = id.ToUpper();

if

(customersCache is

null

) return

null

!;

customersCache.TryGetValue(id, out

Customer? c);

return

Task.FromResult(c);

}

private

Customer

UpdateCache

(

string

id, Customer c

)

{

Customer? old;

if

(customersCache is

not

null

)

{

if

(customersCache.TryGetValue(id, out

old))

{

if

(customersCache.TryUpdate(id, c, old))

{

return

c;

}

}

}

return

null

!;

}

public

async

Task<Customer?> UpdateAsync(string

id, Customer c)

{

// 规范化客户 Id

id = id.ToUpper();

c.CustomerId = c.CustomerId.ToUpper();

// 在数据库中更新

db.Customers.Update(c);

int

affected = await

db.SaveChangesAsync();

如果

(affected == 1

)

{

// 在缓存中更新

返回

UpdateCache(id, c);

}

返回

空

;

}

public

async

Task<bool

?> DeleteAsync(string

id)

{

id = id.ToUpper();

// 从数据库中删除

Customer? c = db.Customers.Find(id);

如果

(c is

空

) 返回

空

;

db.Customers.Remove(c);

int

affected = await

db.SaveChangesAsync();

如果

(affected == 1

)

{

如果

(customersCache 是

空

) 返回

空

;

// 从缓存中删除

返回

customersCache.TryRemove(id, out

c);

}

否则

{

返回

空

;

}

}

}

```

## 实现 Web API 控制器

有一些有用的属性和方法，用于实现返回数据而不是 HTML 的控制器。

使用 MVC 控制器，像`/home/index`这样的路由告诉我们控制器类名和操作方法名，例如`HomeController`类和`Index`操作方法。

使用 Web API 控制器，像`/weatherforecast`这样的路由只告诉我们控制器类名，例如`WeatherForecastController`。要确定要执行的操作方法名称，我们必须将`GET`和`POST`等 HTTP 方法映射到控制器类中的方法。

应该使用以下属性装饰控制器方法，以指示它们将响应的 HTTP 方法：

+   `[HttpGet]`，`[HttpHead]`：这些操作方法响应`GET`或`HEAD`请求，以检索资源并返回资源及其响应标头，或仅返回响应标头。

+   `[HttpPost]`：此操作方法响应`POST`请求，以创建新资源或执行服务定义的其他操作。

+   `[HttpPut]`，`[HttpPatch]`：这些操作方法响应`PUT`或`PATCH`请求，以更新现有资源，可以是替换它或更新其属性的子集。

+   `[HttpDelete]`：此操作方法响应`DELETE`请求，以删除资源。

+   `[HttpOptions]`：此操作方法响应`OPTIONS`请求。

### 理解操作方法返回类型

操作方法可以返回.NET 类型，如单个`string`值，由`class`，`record`或`struct`定义的复杂对象，或复杂对象的集合。ASP.NET Core Web API 将根据 HTTP 请求的`Accept`标头中设置的请求数据格式（例如，如果已注册了合适的序列化程序，则为 JSON）对它们进行序列化。

为了更好地控制响应，有一些辅助方法返回一个围绕.NET 类型的`ActionResult`包装器。

如果操作方法根据输入或其他变量可能返回不同的返回类型，请将操作方法的返回类型声明为`IActionResult`。如果它只返回单个类型但具有不同的状态代码，请将操作方法的返回类型声明为`ActionResult<T>`。

**良好实践**：使用`[ProducesResponseType]`属性装饰操作方法，以指示客户端在响应中应该期望的所有已知类型和 HTTP 状态代码。然后可以公开此信息，以记录客户端应如何与您的网络服务进行交互。将其视为正式文档的一部分。在本章的后面，您将学习如何安装代码分析器，以在不像这样装饰操作方法时给出警告。

例如，根据 id 参数获取产品的操作方法将被装饰为三个属性 - 一个用于指示它响应`GET`请求并具有 id 参数，另外两个用于指示成功时发生什么以及客户端提供无效产品 ID 时发生什么，如下面的代码所示：

```cs

[HttpGet(

"{id}"

)

]

[ProducesResponseType(200, Type = typeof(Product))

]

[ProducesResponseType(404)

]

public

IActionResult

获取

(

字符串

id

)

```

`ControllerBase`类具有使返回不同响应变得容易的方法，如下表所示：

| 方法 | 描述 |
| --- | --- |
| `Ok` | 返回`200`状态代码和转换为客户端首选格式（如 JSON 或 XML）的资源。通常用于响应`GET`请求。 |
| `CreatedAtRoute` | 返回`201`状态代码和新资源的路径。通常用于响应`POST`请求以快速创建资源。 |
| `Accepted` | 返回`202`状态代码，表示请求正在处理但尚未完成。通常用于响应触发需要很长时间才能完成的后台进程的`POST`，`PUT`，`PATCH`或`DELETE`请求。 |
| `NoContentResult` | 返回`204`状态代码和空响应体。通常用于对`PUT`，`PATCH`或`DELETE`请求的响应，当响应不需要包含受影响的资源时。 |
| `BadRequest` | 返回`400`状态代码和可选的消息字符串以提供更多细节。 |
| `NotFound` | 返回`404`状态代码和自动填充的`ProblemDetails`体（需要兼容版本为 2.2 或更高）。 |

## 配置客户存储库和 Web API 控制器

现在，您将配置存储库，以便可以从 Web API 控制器中调用它。

当 Web 服务启动时，您将注册一个作用域依赖服务实现，然后使用构造函数参数注入在新的 Web API 控制器中获取它以处理客户。

为了展示使用路由区分 MVC 和 Web API 控制器的示例，我们将使用常见的`/api` URL 前缀约定来为客户控制器：

1.  打开`Program.cs`并导入`Northwind.WebApi.Repositories`命名空间。

1.  在调用`Build`方法之前添加一条语句，该语句将注册`CustomerRepository`以在运行时作为作用域依赖项使用，如下面代码中的突出显示所示：

```cs

**builder.Services.AddScoped<ICustomerRepository, CustomerRepository>();**

var

app = builder.Build();

```

**良好实践**：我们的存储库使用作为作用域依赖项注册的数据库上下文。您只能在其他作用域依赖项内使用作用域依赖项，因此我们不能将存储库注册为单例。您可以在以下链接中阅读更多信息：[`docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#scoped`](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#scoped)

1.  在`Controllers`文件夹中，添加一个名为`CustomersController.cs`的新类。

1.  在`CustomersController`类文件中，添加语句以定义一个与客户一起工作的 Web API 控制器类，如下面的代码所示：

```cs

使用

Microsoft.AspNetCore.Mvc; // [Route], [ApiController], ControllerBase

使用

Packt.Shared; // Customer

使用

Northwind.WebApi.Repositories; // ICustomerRepository

命名空间

Northwind.WebApi.Controllers

;

// 基本地址：api/customers

[Route(

“api/[controller]”

)

]

[ApiController

]

公开的

类

CustomersController

：ControllerBase

{

私人的

只读的

ICustomerRepository repo;

// 构造函数注入在 Startup 中注册的存储库

公开的

CustomersController

(

ICustomerRepository repo

)

{

这个

.repo = repo;

}

// GET: api/customers

// GET: api/customers/?country=[country]

// 这将始终返回一个客户列表（但可能为空）

[HttpGet

]

[ProducesResponseType(200, Type = typeof(IEnumerable<Customer>))

]

公开的

async

Task<IEnumerable<Customer>> GetCustomers(string

? country)

{

if

(string

.IsNullOrWhiteSpace(country))

{

返回

await

repo.RetrieveAllAsync();

}

else

{

返回

(await

repo.RetrieveAllAsync())

.Where(customer => customer.Country == country);

}

}

// GET: api/customers/[id]

[HttpGet(

"{id}"

，名称= nameof(GetCustomer))

] // 命名路由

[ProducesResponseType(200, Type = typeof(Customer))

]

[ProducesResponseType(404)

]

公共的

async

Task<IActionResult>

GetCustomer

(

字符串

id

)

{

Customer? c = await

repo.RetrieveAsync(id);

if

(c == null

)

{

返回

NotFound(); // 404 Resource not found

}

返回

Ok(c); // 200 OK with customer in body

}

// POST: api/customers

// BODY: Customer (JSON, XML)

[HttpPost

]

[ProducesResponseType(201, Type = typeof(Customer))

]

[ProducesResponseType(400)

]

public

async

Task<IActionResult>

Create

(

[FromBody] Customer c

)

{

if

(c == null

)

{

return

BadRequest(); // 400 错误的请求

}

Customer? addedCustomer = await

repo.CreateAsync(c);

if

(addedCustomer == null?

)

{

return

BadRequest("Repository failed to create customer."

);

}

else

{

return

CreatedAtRoute( // 201 Created

routeName: nameof

(GetCustomer),

routeValues: new

{ id = addedCustomer.CustomerId.ToLower() },

value

: addedCustomer);

}

}

// PUT: api/customers/[id]

// BODY: Customer (JSON, XML)

[HttpPut(

"{id}"

)

]

[ProducesResponseType(204)

]

[ProducesResponseType(400)

]

[ProducesResponseType(404)

]

public

async

Task<IActionResult>

Update

(

string

id, [FromBody] Customer c

)

{

id = id.ToUpper();

c.CustomerId = c.CustomerId.ToUpper();

if

(c == null

|| c.CustomerId != id)

{

return

BadRequest(); // 400 错误的请求

}

Customer? existing = await

repo.RetrieveAsync(id);

if

(existing == null

)

{

return

NotFound(); // 404 资源未找到

}

await

repo.UpdateAsync(id, c);

return

new

NoContentResult(); // 204 No content

}

// DELETE: api/customers/[id]

[HttpDelete(

"{id}"

)

]

[ProducesResponseType(204)

]

[ProducesResponseType(400)

]

[ProducesResponseType(404)

]

public

async

Task<IActionResult>

Delete

(

string

id

)

{

Customer? existing = await

repo.RetrieveAsync(id);

if

(existing == null

)

{

return

NotFound(); // 404 资源未找到

}

bool

? deleted = await

repo.DeleteAsync(id);

if

(deleted.HasValue && deleted.Value) // short circuit AND

{

return

new

NoContentResult(); // 204 No content

}

else

{

return

BadRequest( // 400 错误的请求

$"Customer

{id}

was found but failed to delete."

);

}

}

}

```

在审查此 Web API 控制器类时，请注意以下内容：

+   `Controller`类注册了一个以`api/`开头并包含控制器名称的路由，即`api/customers`。

+   构造函数使用依赖注入来获取用于处理客户端的注册存储库。

+   有五个操作方法来执行客户端的 CRUD 操作——两个`GET`方法（用于所有客户端或一个客户端），`POST`（创建），`PUT`（更新）和`DELETE`。

+   `GetCustomers`方法可以传递一个`string`参数，其中包含一个国家名称。如果缺少，将返回所有客户端。如果存在，则用于按国家筛选客户端。

+   `GetCustomer`方法具有显式命名为`GetCustomer`的路由，以便在插入新客户后生成 URL。

+   `Create`和`Update`方法都使用`[FromBody]`对`customer`参数进行装饰，以告知模型绑定器使用`POST`请求的主体中的值来填充它。

+   `Create`方法返回一个使用`GetCustomer`路由的响应，以便客户端知道如何在将来获取新创建的资源。我们正在匹配两种方法来创建然后获取客户端。

+   `Create`和`Update`方法不需要检查 HTTP 请求主体中传递的客户端模型状态，并且如果无效，则返回包含模型验证错误详细信息的`400 Bad Request`，因为控制器已经使用`[ApiController]`进行了装饰，这样就可以为您完成这些操作。

当服务接收到 HTTP 请求时，它将创建`Controller`类的一个实例，调用适当的操作方法，以客户端首选的格式返回响应，并释放控制器使用的资源，包括存储库和其数据上下文。

## 指定问题详细信息

ASP.NET Core 2.1 及更高版本中新增的一个功能是实现了一个用于指定问题详细信息的 Web 标准的实现。

在带有`[ApiController]`的 Web API 控制器中，如果启用了 ASP.NET Core 2.2 或更高版本的兼容性，返回客户端错误状态码（即`4xx`）的操作方法将自动在响应主体中包含`ProblemDetails`类的序列化实例。

如果您想要控制，那么可以自己创建`ProblemDetails`实例并包含其他信息。

让我们模拟一个需要返回自定义数据给客户端的错误请求：

1.  在`Delete`方法的实现顶部，添加语句来检查`id`是否匹配字面字符串值`"bad"`，如果是，则返回一个自定义问题详细对象，如下面的代码所示：

```cs

//控制问题详细信息

如果

（id == "bad"

）

{

ProblemDetails problemDetails = new

()

{

Status = StatusCodes.Status400BadRequest,

Type = "https://localhost:5001/customers/failed-to-delete"

，

Title = $"Customer ID

{id}

找到但无法删除。"

，

Detail = "More details like Company Name, Country and so on."

，

Instance = HttpContext.Request.Path

};

返回

BadRequest(problemDetails); // 400 Bad Request

}

```

1.  您将稍后测试此功能。

## 控制 XML 序列化

在`Program.cs`中，我们添加了`XmlSerializer`，以便我们的 Web API 服务可以在客户端请求时返回 XML 和 JSON。

但是，`XmlSerializer`无法序列化接口，我们的实体类使用`ICollection<T>`来定义相关的子实体。这会导致运行时出现警告，例如，对于`Customer`类及其`Orders`属性，如下面的输出所示：

```cs

warn: Microsoft.AspNetCore.Mvc.Formatters.XmlSerializerOutputFormatter[1]

在尝试为类型'Packt.Shared.Customer'创建 XmlSerializer 时发生错误。

System.InvalidOperationException: There was an error reflecting type 'Packt.Shared.Customer'.

--->

系统无效操作异常：无法序列化成员

'Packt.

Shared.Customer.Orders' of type 'System.Collections.Generic.ICollection`1[[Packt. Shared.Order, Northwind.Common.EntityModels, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null]]', see inner exception for more details.

```

我们可以通过在将`Customer`序列化为 XML 时排除`Orders`属性来防止此警告：

1.  在`Northwind.Common.EntityModels.Sqlite`和`Northwind.Common.EntityModels.SqlServer`项目中，打开`Customers.cs`。

1.  导入`System.Xml.Serialization`命名空间，以便我们可以使用`[XmlIgnore]`属性。

1.  在`Orders`属性上添加一个属性来忽略它在序列化时，如下面的代码中所突出显示的那样：

```cs

[InverseProperty(nameof(Order.Customer))

]

**[**

**XmlIgnore**

**]**

公共

虚拟

ICollection<Order> Orders { get

; set

; }

```

1.  在`Northwind.Common.EntityModels.SqlServer`项目中，也使用`[XmlIgnore]`装饰`CustomerCustomerDemos`属性。

# 文档化和测试 Web 服务

您可以通过使用浏览器进行 HTTP`GET`请求轻松测试 Web 服务。要测试其他 HTTP 方法，我们需要一个更高级的工具。

## 使用浏览器测试 GET 请求

您将使用 Chrome 测试`GET`请求的三种实现方式-所有客户，特定国家的客户以及使用其唯一客户 ID 的单个客户：

1.  启动`Northwind.WebApi`网络服务。

1.  启动 Chrome。

1.  导航到`https://localhost:5001/api/customers`，注意返回的 JSON 文档，其中包含 Northwind 数据库中的所有 91 个客户（未排序），如*图 16.3*所示：![](img/Image00131.jpg)

图 16.3：来自 Northwind 数据库的客户作为 JSON 文档

1.  导航到`https://localhost:5001/api/customers/?country=Germany`，注意返回的 JSON 文档，其中只包含德国的客户，如*图 16.4*所示：![](img/Image00132.jpg)

图 16.4：作为 JSON 文档的来自德国的客户列表

如果返回一个空数组，则确保输入国家名称时使用正确的大小写，因为数据库查询是区分大小写的。例如，比较`uk`和`UK`的结果。

1.  导航到`https://localhost:5001/api/customers/alfki`，并注意返回的 JSON 文档，其中只包含名为**Alfreds Futterkiste**的客户，如*图 16.5*所示：![](img/Image00133.jpg)

图 16.5：特定客户信息的 JSON 文档

与国家名称不同，对于客户`id`值，我们不需要担心大小写，因为在控制器类中，我们在代码中将`string`值标准化为大写。

但是我们如何测试其他 HTTP 方法，比如`POST`，`PUT`和`DELETE`？我们如何记录我们的 Web 服务，以便任何人都能轻松理解如何与它交互？

为了解决第一个问题，我们可以安装一个名为**REST Client**的 Visual Studio Code 扩展。为了解决第二个问题，我们可以使用**Swagger**，这是全球最流行的用于记录和测试 HTTP API 的技术。但首先，让我们看看 Visual Studio Code 扩展可以实现什么。

有许多用于测试 Web API 的工具，例如**Postman**。尽管 Postman 很受欢迎，但我更喜欢**REST Client**，因为它不会隐藏实际发生的事情。我觉得 Postman 太过于图形化。但我鼓励您探索不同的工具，并找到适合您风格的工具。您可以在以下链接了解更多关于 Postman 的信息：[`www.postman.com/`](https://www.postman.com/)

## 使用 REST Client 扩展测试 HTTP 请求

REST Client 是一个扩展，允许您发送任何类型的 HTTP 请求并在 Visual Studio Code 中查看响应。即使您更喜欢使用 Visual Studio 作为您的代码编辑器，安装 Visual Studio Code 并使用 REST Client 等扩展也是有用的。

### 使用 REST Client 进行 GET 请求

我们将首先创建一个用于测试`GET`请求的文件：

1.  如果您尚未安装 Huachao Mao 的 REST Client（`humao.rest-client`），请立即在 Visual Studio Code 中安装它。

1.  在您喜欢的代码编辑器中，启动`Northwind.WebApi`项目的 Web 服务。

1.  在 Visual Studio Code 中，在`PracticalApps`文件夹中，创建一个`RestClientTests`文件夹，然后打开该文件夹。

1.  在`RestClientTests`文件夹中，创建一个名为`get-customers.http`的文件，并修改其内容以包含一个 HTTP `GET`请求，以检索所有客户，如下面的代码所示：

```cs

GET https://localhost:5001/api/customers/ HTTP/1.1

```

1.  在 Visual Studio Code 中，导航到**视图** | **命令面板**，输入`rest client`，选择命令**Rest Client: Send Request**，然后按 Enter，如*图 16.6*所示：![](img/Image00134.jpg)

图 16.6：使用 REST Client 发送 HTTP GET 请求

1.  请注意**响应**显示在一个新的选项卡窗格中，并且您可以通过拖放选项卡来重新排列打开的选项卡以获得水平布局。

1.  输入更多的`GET`请求，每个请求之间用三个井号分隔，以测试获取不同国家的客户和使用其 ID 获取单个客户，如下面的代码所示：

```cs

###

GET https://localhost:5001/api/customers/?country=Germany HTTP/1.1

###

GET https://localhost:5001/api/customers/?country=USA HTTP/1.1

接受

: application/xml

###

GET https://localhost:5001/api/customers/ALFKI HTTP/1.1

###

GET https://localhost:5001/api/customers/abcxy HTTP/1.1

```

1.  单击每个请求上方的**发送请求**链接以发送请求；例如，具有请求头以 XML 而不是 JSON 请求美国客户的`GET`，如*图 16.7*所示：![](img/Image00135.jpg)

图 16.7：使用 REST Client 发送 XML 请求并获取响应

### 使用 REST Client 进行其他请求

接下来，我们将创建一个用于测试其他请求的文件，比如`POST`：

1.  在`RestClientTests`文件夹中，创建一个名为`create-customer.http`的文件，并修改其内容以定义一个`POST`请求来创建一个新的客户，注意 REST Client 在您输入常见 HTTP 请求时会提供智能感知，如下面的代码所示：

```cs

POST https://localhost:5001/api/customers/ HTTP/1.1

内容类型

: application/json

Content-Length

: 301

{

"customerID": "ABCXY",

"companyName": "ABC Corp",

"contactName": "John Smith",

"contactTitle": "先生",

"地址": "Main Street",

"city": "纽约",

"地区": "纽约",

"邮政编码": "90210",

"国家": "美国",

"phone": "(123) 555-1234",

"fax": null,

"orders": null

}

```

1.  由于不同操作系统中的换行符不同，因此 `Content-Length` 标头的值在 Windows 和 macOS 或 Linux 上会有所不同。如果值错误，则请求将失败。要发现正确的内容长度，请选择请求的正文，然后在状态栏中查看字符数，如 *图 16.8* 所示：![](img/Image00136.jpg)

图 16.8：检查正确的内容长度

1.  发送请求并注意响应是 `201 Created`。还要注意新创建的客户的位置（即 URL）是 `https://localhost:5001/api/Customers/abcxy`，并在响应正文中包含了新创建的客户，如 *图 16.9* 所示：![](img/Image00137.jpg)

图 16.9：添加新客户

我给你留下了一个可选的挑战，创建 REST 客户端文件来测试更新客户（使用 `PUT`）和删除客户（使用 `DELETE`）。尝试对现有客户和不存在的客户进行测试。本书的 GitHub 存储库中有解决方案。

现在我们已经看到了一种快速简单的测试服务的方法，这也是学习 HTTP 的好方法，那么外部开发人员呢？我们希望他们尽可能轻松地学习然后调用我们的服务。为此，我们将使用 Swagger。

## 理解 Swagger

Swagger 最重要的部分是 **OpenAPI 规范**，它为您的 API 定义了一个 REST 风格的契约，详细说明了所有资源和操作，以人类和机器可读的格式进行易于开发、发现和集成。

开发人员可以使用 OpenAPI 规范为 Web API 自动生成其首选语言或库中的强类型客户端代码。

对我们来说，另一个有用的功能是 **Swagger UI**，因为它会自动生成 API 的文档，并具有内置的可视化测试功能。

让我们回顾一下如何使用 `Swashbuckle` 包为我们的 Web 服务启用 Swagger：

1.  如果 Web 服务正在运行，请关闭 Web 服务器。

1.  打开 `Northwind.WebApi.csproj` 并注意 `Swashbuckle.AspNetCore` 的包引用，如下面的标记所示：

```cs

<ItemGroup>

<PackageReference Include="Swashbuckle.AspNetCore"

版本="6.1.5"

/>

</ItemGroup>

```

1.  将 `Swashbuckle.AspNetCore` 包的版本更新为最新版本，例如，截至 2021 年 9 月的版本为 `6.2.1`。

1.  在 `Program.cs` 中，请注意 Microsoft 的 OpenAPI 模型命名空间的导入，如下面的代码所示：

```cs

使用

Microsoft.OpenApi.Models;

```

1.  导入 Swashbuckle 的 SwaggerUI 命名空间，如下面的代码所示：

```cs

使用

Swashbuckle.AspNetCore.SwaggerUI; // SubmitMethod

```

1.  在 `Program.cs` 的中间位置，请注意添加 Swagger 支持的语句，包括 Northwind 服务的文档，指示这是您的服务的第一个版本，并更改标题，如下面的代码中所示：

```cs

builder.Services.AddSwaggerGen(c =>

{

c.SwaggerDoc("v1"

, 新

()

{ 标题 = "

**Northwind 服务 API**

"

, 版本 = "v1"

});

});

```

1.  在配置 HTTP 请求管道的部分，请注意在开发模式下使用 Swagger 和 Swagger UI 的语句，并为 OpenAPI 规范 JSON 文档定义一个端点。

1.  添加代码以明确列出我们希望在我们的 Web 服务中支持的 HTTP 方法，并更改端点名称，如下面的代码中所示：

```cs

var

app = builder.Build();

// 配置 HTTP 请求管道。

if

(builder.Environment.IsDevelopment())

{

app.UseSwagger();

app.UseSwaggerUI(c =>

**{**

**c.SwaggerEndpoint(**

**"/swagger/v1/swagger.json"**

**,**

**"Northwind Service API Version 1"**

**});**

**c.SupportedSubmitMethods(**

**新**

**[] {**

**SubmitMethod.Get, SubmitMethod.Post,**

**SubmitMethod.Put, SubmitMethod.Delete });**

**});**

}

```

## 使用 Swagger UI 测试请求

现在您已经准备好使用 Swagger 测试 HTTP 请求：

1.  启动`Northwind.WebApi` Web 服务。

1.  在 Chrome 中，导航至`https://localhost:5001/swagger/`，注意到**Customers**和**WeatherForecast** Web API 控制器都已被发现和记录，以及 API 使用的**模式**。

1.  点击**GET /api/Customers/{id}**以展开该端点，并注意客户**id**的必需参数，如*图 16.10*所示：![](img/Image00138.jpg)

图 16.10：在 Swagger 中检查 GET 请求的参数

1.  点击**试一下**，输入`ALFKI`的**id**，然后点击宽蓝色的**执行**按钮，如*图 16.11*所示：![](img/Image00139.jpg)

图 16.11：在点击执行按钮之前输入客户 id

1.  向下滚动并注意**请求 URL**，**服务器响应**与**代码**，以及包括**响应主体**和**响应头**在内的**详细信息**，如*图 16.12*所示：![](img/Image00140.jpg)

图 16.12：在成功的 Swagger 请求中关于 ALFKI 的信息

1.  滚动回到页面顶部，点击**POST /api/Customers**以展开该部分，然后点击**试一下**。

1.  点击**请求主体**框内，并修改 JSON 以定义新客户，如下所示：

```cs

{

"客户 ID"

: "超级"

，

"公司名称"

: "超级公司"

,

"联系人姓名"

: "拉斯姆斯·伊本森"

,

"联系人职称"

: "销售主管"

,

"地址"

: "罗特斯莱夫 23"

,

"城市"

: "比伦德"

,

"地区"

: null

,

"邮政编码"

: "4371"

,

"国家"

: "丹麦"

,

"电话"

: "31 21 43 21"

,

"传真"

: "31 21 43 22"

}

```

1.  点击**执行**，注意**请求 URL**，**服务器响应**与**代码**，以及包括**响应主体**和**响应头**在内的**详细信息**，注意到`201`的响应代码意味着客户已成功创建，如*图 16.13*所示：![](img/Image00141.jpg)

图 16.13：成功添加新客户

1.  滚动回到页面顶部，点击**GET /api/Customers**，点击**试一下**，为国家参数输入`丹麦`，然后点击**执行**，确认新客户已添加到数据库，如*图 16.14*所示：![](img/Image00142.jpg)

图 16.14：成功获取包括新添加客户在内的丹麦客户

1.  点击**DELETE /api/Customers/{id}**，点击**试一下**，为**id**输入`super`，点击**执行**，注意**服务器响应代码**为`204`，表示已成功删除，如*图 16.15*所示：![](img/Image00143.jpg)

图 16.15：成功删除客户

1.  再次点击**执行**，注意**服务器响应代码**为`404`，表示客户不再存在，而**响应主体**包含问题详细信息的 JSON 文档，如*图 16.16*所示：![](img/Image00144.jpg)

图 16.16：已删除的客户不再存在

1.  再次输入`bad`作为**id**，再次点击**执行**，注意**服务器响应代码**为`400`，表示客户确实存在但未能被删除（在这种情况下，是因为 Web 服务模拟了这个错误），而**响应主体**包含自定义问题详细信息的 JSON 文档，如*图 16.17*所示：![](img/Image00145.jpg)

图 16.17：客户确实存在但未能被删除

1.  使用`GET`方法确认新客户已从数据库中删除（丹麦原本只有两个客户）。

我将把更新现有客户的测试留给读者，使用`PUT`。

1.  关闭 Chrome 并关闭 Web 服务器。

## 启用 HTTP 日志记录

HTTP 日志记录是一个可选的中间件组件，用于记录有关 HTTP 请求和 HTTP 响应的信息，包括以下内容：

+   关于 HTTP 请求的信息

+   标题

+   正文

+   关于 HTTP 响应的信息

这在 Web 服务中对审计和调试场景非常有价值，但要注意，它可能会对性能产生负面影响。您可能还会记录**个人身份信息**（PII），这可能会在某些司法管辖区引起合规问题。

让我们看看 HTTP 日志记录的实际操作：

1.  在`Program.cs`中，导入用于处理 HTTP 日志的命名空间，如下所示：

```cs

使用

Microsoft.AspNetCore.HttpLogging；// HttpLoggingFields

```

1.  在服务配置部分，添加一个语句来配置 HTTP 日志记录，如下所示：

```cs

builder.Services.AddHttpLogging(options =>

{

options.LoggingFields = HttpLoggingFields.All;

options.RequestBodyLogLimit = 4096

; // 默认为 32k

options.ResponseBodyLogLimit = 4096

; // 默认为 32k

});

```

1.  在 HTTP 管道配置部分，添加一个语句以在使用路由之前添加 HTTP 日志记录，如下所示：

```cs

app.UseHttpLogging();

```

1.  启动`Northwind.WebApi` Web 服务。

1.  启动 Chrome。

1.  转到`https://localhost:5001/api/customers`。

1.  在命令提示符或终端中，注意请求和响应已被记录，如下所示：

```cs

信息：Microsoft.AspNetCore.HttpLogging.HttpLoggingMiddleware[1]

请求：

协议：HTTP/1.1

方法：GET

方案：https

路径基础：

路径：/api/customers

查询字符串：

连接：保持连接

接受：*/*

接受编码：gzip，deflate，br

主机：localhost:5001

信息：Microsoft.AspNetCore.HttpLogging.HttpLoggingMiddleware[2]

响应：

状态码：200

内容类型：application/json；charset=utf-8

...

传输编码：分块

```

1.  关闭 Chrome 并关闭 Web 服务器。

您现在已经准备好构建消费您的 Web 服务的应用程序了。

# 使用 HTTP 客户端消费 Web 服务

现在我们已经构建并测试了我们的 Northwind 服务，我们将学习如何使用`HttpClient`类及其工厂从任何.NET 应用程序中调用它。

## 了解 HttpClient

消费 Web 服务的最简单方法是使用`HttpClient`类。然而，许多人使用它的方式是错误的，因为它实现了`IDisposable`，而微软自己的文档显示了它的糟糕用法。请参阅 GitHub 存储库中的书籍链接，了解更多讨论。

通常，当一个类型实现`IDisposable`时，您应该在`using`语句内创建它，以确保尽快将其处理掉。`HttpClient`不同，因为它是共享的、可重入的，并且部分线程安全。

问题涉及到底层网络套接字的管理方式。关键是，您应该在应用程序的生命周期内为每个 HTTP 端点使用单个实例。这将允许每个`HttpClient`实例设置适用于其工作的端点的默认值，同时有效地管理底层网络套接字。

## 使用 HttpClientFactory 配置 HTTP 客户端

微软已经意识到了这个问题，在 ASP.NET Core 2.1 中引入了`HttpClientFactory`来鼓励最佳实践；这是我们将使用的技术。

在下面的示例中，我们将使用 Northwind MVC 网站作为 Northwind Web API 服务的客户端。由于两者需要同时托管在 Web 服务器上，我们首先需要配置它们使用不同的端口号，如下列表所示：

+   Northwind Web API 服务将使用`HTTPS`在端口`5002`上监听。

+   Northwind MVC 网站将继续使用`HTTP`在端口`5000`上监听，并使用`HTTPS`在端口`5001`上监听。

让我们配置这些端口：

1.  在`Northwind.WebApi`项目的`Program.cs`中，添加一个扩展方法调用`UseUrls`来指定`HTTPS`的端口`5002`，如下所示：

```cs

var

builder = WebApplication.CreateBuilder(args);

**builder.WebHost.UseUrls(**

**"https://localhost:5002/"**

**);**

```

1.  在`Northwind.Mvc`项目中，打开`Program.cs`并导入用于使用 HTTP 客户端工厂的命名空间，如下面的代码所示：

```cs

使用

System.Net.Http.Headers; // MediaTypeWithQualityHeaderValue

```

1.  添加一个语句以启用`HttpClientFactory`，使用命名客户端调用 Northwind Web API 服务，使用 HTTPS 在端口`5002`上请求 JSON 作为默认响应格式，如下面的代码所示：

```cs

builder.Services.AddHttpClient(name: "Northwind.WebApi"

,

configureClient: options =>

{

options.BaseAddress = new

Uri("https://localhost:5002/"

);

options.DefaultRequestHeaders.Accept.Add(

new

MediaTypeWithQualityHeaderValue(

"application/json"

, 1.0

));

});

```

## 在控制器中以 JSON 格式获取客户

现在我们可以创建一个 MVC 控制器动作方法，使用工厂创建一个 HTTP 客户端，发出一个`GET`请求以获取客户，并使用.NET 5 中引入的`System.Net.Http.Json`程序集和命名空间中的便利扩展方法对 JSON 响应进行反序列化：

1.  打开`Controllers/HomeController.cs`并声明一个字段来存储 HTTP 客户端工厂，如下面的代码所示：

```cs

私人的

readonly

IHttpClientFactory clientFactory;

```

1.  在构造函数中设置字段，如下面的代码中所示：

```cs

public

HomeController

(

ILogger<HomeController> logger,

NorthwindContext injectedContext

**,**

**IHttpClientFactory httpClientFactory**

)

{

_logger = logger;

db = injectedContext;

**clientFactory = httpClientFactory;**

}

```

1.  为调用 Northwind Web API 服务、获取所有客户并将它们传递给视图创建一个新的动作方法，如下面的代码所示：

```cs

公共的

async

Task<IActionResult>

顾客

(

string

国家

)

{

字符串

uri;

if

(string

.IsNullOrEmpty(country))

{

ViewData["Title"

] = "All Customers Worldwide"

;

uri = "api/customers/"

;

}

否则

{

ViewData["Title"

] = $"Customers in

{country}

"

;

uri = $"api/customers/?country=

{country}

"

;

}

HttpClient client = clientFactory.CreateClient(

name: "Northwind.WebApi"

);

HttpRequestMessage request = new

(

method: HttpMethod.Get, requestUri: uri);

HttpResponseMessage response = await

client.SendAsync(request);

IEnumerable<Customer>? model = await

response.Content

.ReadFromJsonAsync<IEnumerable<Customer>>();

return

View(model);

}

```

1.  在`Views/Home`文件夹中，创建一个名为`Customers.cshtml`的 Razor 文件。

1.  修改 Razor 文件以呈现客户，如下面的标记所示：

```cs

@using Packt.Shared

@model IEnumerable<Customer>

<

h2

@ViewData["Title"]</

h2

<

表

类

=

"table"

<

thead

<

tr

<

th

公司名称</

th

<

th

联系人姓名</

th

<

th

Address</

th

<

th

电话</

th

</

tr

</

thead

<

tbody

@if (Model is not null)

{

@foreach (Customer c in Model)

{

<

tr

<

td

@Html.DisplayFor(modelItem => c.CompanyName)

</

td

<

td

@Html.DisplayFor(modelItem => c.ContactName)

</

td

<

td

@Html.DisplayFor(modelItem => c.Address)

@Html.DisplayFor(modelItem => c.City)

@Html.DisplayFor(modelItem => c.Region)

@Html.DisplayFor(modelItem => c.Country)

@Html.DisplayFor(modelItem => c.PostalCode)

</

td

<

td

@Html.DisplayFor(modelItem => c.Phone)

</

td

</

tr

}

}

</

tbody

</

表

```

1.  在`Views/Home/Index.cshtml`中，在呈现访客计数后添加一个表单，允许访客输入一个国家并查看客户，如下面的标记所示：

```cs

<

h3

从服务中查询客户</

h3

<

form

asp-action

=

"Customers"

method

=

"get"

<

input

name

=

"country"

placeholder

=

"输入一个国家"

/>

<

input

类型

=

"提交"

/>

</

form

```

## 启用跨源资源共享

**跨源资源共享**（**CORS**）是基于 HTTP 头的标准，用于在客户端和服务器位于不同域（源）时保护 Web 资源。它允许服务器指示除自己之外的哪些来源（由域，方案或端口的组合定义）允许加载资源。

由于我们的 Web 服务托管在端口`5002`上，我们的 MVC 网站托管在端口`5000`和`5001`上，它们被视为不同的来源，因此资源不能共享。

在服务器上启用 CORS 并配置我们的 Web 服务，以仅允许来自 MVC 网站的请求将是有用的：

1.  在`Northwind.WebApi`项目中，打开`Program.cs`。

1.  在服务配置部分中添加一条语句以添加对 CORS 的支持，如下面的代码所示：

```cs

builder.Services.AddCors();

```

1.  在调用`UseEndpoints`之前，在 HTTP 管道配置部分中添加一条语句，以使用 CORS 并允许来自任何网站的`GET`，`POST`，`PUT`和`DELETE`请求，例如具有`https://localhost:5001`来源的 Northwind MVC，如下面的代码所示：

```cs

app.UseCors(configurePolicy: options =>

{

options.WithMethods("GET"

，"POST"

，"PUT"

，"DELETE"

);

options.WithOrigins(

"https://localhost:5001"

//允许来自 MVC 客户端的请求

);

});

```

1.  启动`Northwind.WebApi`项目，并确认 Web 服务仅在端口`5002`上侦听，如下面的输出所示：

```cs

信息：Microsoft.Hosting.Lifetime[14]

现在正在侦听：https://localhost:5002

```

1.  启动`Northwind.Mvc`项目，并确认网站正在端口`5000`和`5002`上侦听，如下面的输出所示：

```cs

信息：Microsoft.Hosting.Lifetime[14]

现在正在侦听：https://localhost:5001

信息：Microsoft.Hosting.Lifetime[14]

现在正在侦听：http://localhost:5000

```

1.  启动 Chrome。

1.  在客户表单中，输入国家，如`德国`，`英国`或`美国`，单击**提交**，并注意客户列表，如*图 16.18*所示：![](img/Image00146.jpg)

图 16.18：英国的客户

1.  单击浏览器中的**返回**按钮，清除国家文本框，单击**提交**，并注意全球客户列表。

1.  在命令提示符或终端中，注意`HttpClient`写入它发出的每个 HTTP 请求和它接收到的 HTTP 响应，如下面的输出所示：

```cs

信息：System.Net.Http.HttpClient.Northwind.WebApi.ClientHandler[100]

发送 HTTP 请求 GET https://localhost:5002/api/customers/?country=UK

信息：System.Net.Http.HttpClient.Northwind.WebApi.ClientHandler[101]

在 931.864ms 后收到 HTTP 响应头-200

```

1.  关闭 Chrome 并关闭 Web 服务器。

您已成功构建了一个 Web 服务并从 MVC 网站调用它。

# 实现 Web 服务的高级功能

现在您已经了解了构建 Web 服务然后从客户端调用它的基础知识，让我们看一些更高级的功能。

## 实现健康检查 API

有许多付费服务执行站点可用性测试，这些测试是基本的 ping，有些还具有对 HTTP 响应的更高级分析。

ASP.NET Core 2.2 及更高版本使实现更详细的网站健康检查变得容易。例如，您的网站可能是活动的，但它准备好了吗？它能从数据库中检索数据吗？

让我们为我们的 Web 服务添加基本的健康检查功能：

1.  在`Northwind.WebApi`项目中，添加项目引用以启用 Entity Framework Core 数据库健康检查，如下标记所示：

```cs

<PackageReference Include=

"Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore"

Version="6.0.0"

/>

```

1.  构建项目。

1.  在`Program.cs`中，在服务配置部分的底部，添加一条语句以添加健康检查，包括对 Northwind 数据库上下文的检查，如下面的代码所示：

```cs

builder.Services.AddHealthChecks()

.AddDbContextCheck<NorthwindContext>();

```

默认情况下，数据库上下文检查调用 EF Core 的 `CanConnectAsync` 方法。您可以通过调用 `AddDbContextCheck` 方法自定义要运行的操作。

1.  在 HTTP 管道配置部分，在调用 `MapControllers` 之前，添加一个语句以使用基本健康检查，如下面的代码所示：

```cs

app.UseHealthChecks(path: "/howdoyoufeel"

);

```

1.  启动 Web 服务。

1.  启动 Chrome。

1.  导航到 `https://localhost:5002/howdoyoufeel` 并注意到 Web 服务以纯文本响应`Healthy` 响应。

1.  在命令提示符或终端中，注意执行的 SQL 语句以测试数据库的健康状况，如下面的输出所示：

```cs

级别：调试，事件 ID：20100，状态：执行 DbCommand [Parameters=[]，CommandType='Text'，CommandTimeout='30']

SELECT 1

```

1.  关闭 Chrome 并关闭 Web 服务器。

## 实现 Open API 分析器和约定

在本章中，您学习了如何通过手动为控制器类添加属性来启用 Swagger 来记录 Web 服务。

在 ASP.NET Core 2.2 或更高版本中，有 API 分析器，可以自动记录已用 `[ApiController]` 属性注释的控制器类。分析器假定了一些 API 约定。

要使用它，您的项目必须启用 OpenAPI 分析器，如下面的标记所示：

```cs

<PropertyGroup>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<ImplicitUsings>enable</ImplicitUsings>

**<IncludeOpenAPIAnalyzers>**

**true**

**</IncludeOpenAPIAnalyzers>**

</PropertyGroup>

```

安装后，未经适当装饰的控制器应该在编译源代码时出现警告（绿色波浪线）。例如，`WeatherForecastController` 类。

然后，自动代码修复可以添加适当的 `[Produces]` 和 `[ProducesResponseType]` 属性，尽管这目前仅在 Visual Studio 中有效。在 Visual Studio Code 中，您将看到分析器认为您应该添加属性的位置的警告，但您必须自己添加它们。

## 实现瞬态故障处理

当客户端应用程序或网站调用 Web 服务时，可能来自世界的另一边。客户端和服务器之间的网络问题可能导致与您的实现代码无关的问题。如果客户端发起调用并失败，应用程序不应该放弃。如果再次尝试，问题可能已经解决。我们需要一种处理这些临时故障的方法。

为了处理这些瞬态故障，Microsoft 建议您使用第三方库 Polly 来实现指数退避的自动重试。您定义一个策略，库会处理其他一切。

**良好实践**：您可以在以下链接中了解有关 Polly 如何使您的 Web 服务更可靠的更多信息：[`docs.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly`](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)

## 添加安全 HTTP 标头

ASP.NET Core 具有内置支持常见安全 HTTP 标头的功能，如 HSTS。但是还有许多其他 HTTP 标头，您应该考虑实现。

使用中间件类是添加这些标头的最简单方法：

1.  在 `Northwind.WebApi` 项目/文件夹中，创建一个名为 `SecurityHeadersMiddleware.cs` 的文件，并修改其语句，如下面的代码所示：

```cs

using

Microsoft.Extensions.Primitives; // StringValues

public

class

SecurityHeaders

{

private

readonly

RequestDelegate next;

public

SecurityHeaders

(

RequestDelegate next

)

{

this

.next = next;

}

public

Task

Invoke

(

HttpContext context

)

{

// 在这里添加任何 HTTP 响应标头

context.Response.Headers.Add(

"super-secure"

, new

StringValues("enable"

));

return

next(context);

}

}

```

1.  在 `Program.cs` 中，在 HTTP 管道配置部分，添加一个语句在调用 `UseEndpoints` 之前注册中间件，如下面的代码所示：

```cs

app.UseMiddleware<SecurityHeaders>();

```

1.  启动网络服务。

1.  启动 Chrome。

1.  显示**开发者工具**及其**网络**选项卡以记录请求和响应。

1.  导航到 `https://localhost:5002/weatherforecast`。

1.  注意我们添加的名为 `super-secure` 的自定义 HTTP 响应标头，如 *图 16.19* 所示：![](img/Image00147.jpg)

图 16.19：添加名为 super-secure 的自定义 HTTP 标头

# 使用最小的 API 构建网络服务

对于 .NET 6，微软付出了很多努力，添加了新功能到 C# 10 语言，并简化了 ASP.NET Core 库，以便使用最小的 API 创建网络服务。

您可能还记得 Web API 项目模板中提供的天气预报服务。它展示了使用控制器类返回使用伪造数据的五天天气预报。我们现在将使用最小的 API 重新创建该天气服务。

首先，天气服务有一个类来表示单个天气预报。我们将需要在多个项目中使用这个类，所以让我们为此创建一个类库：

1.  使用您喜欢的代码编辑器添加一个新项目，如下列表所示：

1.  项目模板：**类库** / `classlib`

1.  工作区/解决方案文件和文件夹：`PracticalApps`

1.  项目文件和文件夹：`Northwind.Common`

1.  将 `Class1.cs` 重命名为 `WeatherForecast.cs`。

1.  修改 `WeatherForecast.cs`，如下面的代码所示：

```cs

namespace

Northwind.Common

{

public

class

WeatherForecast

{

public

静态

readonly

string

[] 摘要 = 新

[]

{

"寒冷"

, "清爽"

, "寒冷"

, "凉爽"

, "温和"

,

"温暖"

, "温暖"

, "炎热"

, "酷热"

, "灼热"

};

public

DateTime Date { get

; 设置

; }

public

int

TemperatureC { get

; 设置

; }

public

int

TemperatureF => 32

+ (int

)(TemperatureC / 0.5556

);

public

string

? Summary { get

; 设置

; }

}

}

```

## 使用最小的 API 构建天气服务

现在让我们使用最小的 API 重新创建该天气服务。它将在端口 `5003` 上监听，并启用 CORS 支持，以便请求只能来自 MVC 网站，并且只允许 `GET` 请求：

1.  使用您喜欢的代码编辑器添加一个新项目，如下列表所示：

1.  项目模板：**ASP.NET Core 空** / `web`

1.  工作区/解决方案文件和文件夹：`PracticalApps`

1.  项目文件和文件夹：`Minimal.WebApi`

1.  其他 Visual Studio 选项：**身份验证类型**：无，**配置为 HTTPS**：已选择，**启用 Docker**：已清除，**启用 OpenAPI 支持**：已选择。

1.  在 Visual Studio Code 中，选择 `Minimal.WebApi` 作为活动的 OmniSharp 项目。

1.  在 `Minimal.WebApi` 项目中，添加对 `Northwind.Common` 项目的项目引用，如下标记所示：

```cs

<ItemGroup>

<ProjectReference Include="..\Northwind.Common\Northwind.Common.csproj"

/>

</ItemGroup>

```

1.  构建 `Minimal.WebApi` 项目。

1.  修改 `Program.cs`，如下面的代码中所示：

```cs

**using**

**Northwind.Common;**

**// WeatherForecast**

var

builder = WebApplication.CreateBuilder(args);

**builder.WebHost.UseUrls(**

**"https://localhost:5003"**

**);**

**builder.Services.AddCors();**

var

app = builder.Build();

**// 仅允许 MVC 客户端，且仅允许 GET 请求**

**app.UseCors(configurePolicy: options =>**

**{**

**options.WithMethods(**

**"GET"**

**);**

**options.WithOrigins(**

**"https://localhost:5001"**

**);**

**});**

**app.MapGet(**

**"/api/weather"**

**, () =>**

**{**

**return**

**Enumerable.Range(**

**1**

**,**

**5**

**).Select(index =>**

**new**

**WeatherForecast**

**{**

**Date = DateTime.Now.AddDays(index),**

**TemperatureC = Random.Shared.Next(**

**-20**

**,**

**55**

**),**

**摘要 = WeatherForecast.Summaries[**

**Random.Shared.Next(WeatherForecast.Summaries.Length)]**

**})**

**.ToArray();**

**});**

app.Run();

```

**良好实践**：对于简单的 Web 服务，避免创建控制器类，而是使用最小的 API 将所有配置和实现放在一个地方，即`Program.cs`。

1.  在**属性**中，修改`launchSettings.json`以配置`Minimal.WebApi`配置文件，以使用 URL 中的端口`5003`启动浏览器，如下标记所示：

```cs

"profiles"

: {

"Minimal.WebApi"

: {

"commandName"

: "Project"

,

"dotnetRunMessages"

: "true"

,

"launchBrowser"

: true

,

**"applicationUrl"**

**:**

**"https://localhost:5003/api/weather"**

**，**

"environmentVariables"

: {

"ASPNETCORE_ENVIRONMENT"

: "Development"

}

```

## 测试最小天气服务

在创建服务的客户端之前，让我们测试它是否以 JSON 格式返回预测：

1.  启动 Web 服务项目。

1.  如果您没有使用 Visual Studio 2022，请启动 Chrome 并导航到`https://localhost:5003/api/weather`。

1.  注意 Web API 服务应返回一个包含五个随机天气预报对象的 JSON 文档的数组。

1.  关闭 Chrome 并关闭 Web 服务器。

## 向 Northwind 网站主页添加天气预报

最后，让我们向 Northwind 网站添加一个 HTTP 客户端，以便它可以调用天气服务并在主页上显示预测：

1.  在`Northwind.Mvc`项目中，添加对`Northwind.Common`的项目引用，如下标记所示：

```cs

<ItemGroup>

<!-- 如果要更改 Sqlite 为 SqlServer -->

you prefer -->

<ProjectReference Include="..\Northwind.Common.DataContext.Sqlite\Northwind.Common.DataContext.Sqlite.csproj"

/>

**<ProjectReference Include=**

**"..\Northwind.Common\Northwind.Common.csproj"**

**/>**

</ItemGroup>

```

1.  在`Program.cs`中，添加一个语句来配置一个 HTTP 客户端，以调用端口`5003`上的最小服务，如下所示的代码：

```cs

builder.Services.AddHttpClient(name: "Minimal.WebApi"

,

configureClient: options =>

{

options.BaseAddress = new

Uri("https://localhost:5003/"

);

options.DefaultRequestHeaders.Accept.Add(

new

MediaTypeWithQualityHeaderValue(

"application/json"

, 1.0

));

});

```

1.  在`HomeController.cs`中，导入`Northwind.Common`命名空间，并在`Index`方法中，添加语句以获取并使用 HTTP 客户端调用天气服务以获取预测并将它们存储在`ViewData`中，如下所示的代码：

```cs

try

{

HttpClient client = clientFactory.CreateClient(

名称："Minimal.WebApi"

);

HttpRequestMessage request = new

(

method: HttpMethod.Get, requestUri: "api/weather"

);

HttpResponseMessage response = await

client.SendAsync(request);

ViewData["weather"

] = await

response.Content

.ReadFromJsonAsync<WeatherForecast[]>();

}

catch (Exception ex)

{

_logger.LogWarning($"The Minimal.WebApi service is not responding. Exception:

{ex.Message}

"

);

ViewData["weather"

] = Enumerable.Empty<WeatherForecast>().ToArray();

}

```

1.  在`Views/Home`中的`Index.cshtml`中，导入`Northwind.Common`命名空间，然后在顶部代码块中从`ViewData`字典中获取天气预报，如下标记所示：

```cs

@{

ViewData["Title"

] = "主页"

;

string

currentItem = ""

;

**WeatherForecast[]? weather = ViewData[**

**"weather"**

**]**

**as**

**WeatherForecast[];**

}

```

1.  在第一个`<div>`中，在呈现当前时间后，添加标记以列举天气预报，除非没有任何天气预报，并在表中呈现它们，如下标记所示：

```cs

<

p

<

h4

五天天气预报</

h4

@if ((weather is null) || (!weather.Any()))

{

<

p

未找到天气预报。</

p

}

else

{

<

table

class

=

"table table-info"

<

tr

@foreach (WeatherForecast w in weather)

{

<

td

@w.Date.ToString("ddd d MMM") will be @w.Summary</

td

}

</

tr

</

table

}

</

p

```

1.  启动`Minimal.WebApi`服务。

1.  启动`Northwind.Mvc`网站。

1.  导航到`https://localhost:5001/`，并注意天气预报，如*图 16.20*所示：![](img/Image00148.jpg)

图 16.20：Northwind 网站首页上的五天天气预报

1.  查看 MVC 网站的命令提示符或终端，并注意信息消息，指示请求在约 83ms 内发送到 minimal API Web 服务的 `api/weather` 端点，如下所示：

```cs

信息：System.Net.Http.HttpClient.Minimal.WebApi.LogicalHandler[100]

开始处理 HTTP 请求 GET https://localhost:5003/api/weather

信息：System.Net.Http.HttpClient.Minimal.WebApi.ClientHandler[100]

发送 HTTP 请求 GET https://localhost:5003/api/weather

信息：System.Net.Http.HttpClient.Minimal.WebApi.ClientHandler[101]

在 76.8963ms 后收到 HTTP 响应头 - 200

信息：System.Net.Http.HttpClient.Minimal.WebApi.LogicalHandler[101]

在 82.9515ms 后结束处理 HTTP 请求 – 200

```

1.  停止 `Minimal.WebApi` 服务，刷新浏览器，并注意几秒钟后 MVC 网站首页出现而没有天气预报。

1.  关闭 Chrome 并关闭 Web 服务器。

# 练习和探索

通过回答一些问题来测试您的知识和理解，进行一些实践，并通过深入研究探索本章的主题。

## 练习 16.1 – 测试您的知识

回答以下问题：

1.  您应该从哪个类继承以创建 ASP.NET Core Web API 服务的控制器类？

1.  如果您使用 `[ApiController]` 属性装饰您的控制器类以获得默认行为，比如对无效模型自动返回 `400` 响应，您还必须做什么？

1.  您必须做什么来指定哪个控制器操作方法将在响应 HTTP 请求时执行？

1.  在调用操作方法时，您必须做什么来指定应该期望什么响应？

1.  列出三种可以调用的方法，以返回具有不同状态代码的响应。

1.  列出四种测试 Web 服务的方法。

1.  为什么不应该在使用 `HttpClient` 时将其包装在 `using` 语句中以在完成后处理它，即使它实现了 `IDisposable` 接口，而应该使用什么代替？

1.  CORS 是什么缩写，为什么在 Web 服务中启用它很重要？

1.  如何使客户端能够检测您的 ASP.NET Core 2.2 及更高版本的 Web 服务是否健康？

1.  端点路由提供了哪些好处？

## 练习 16.2 – 使用 HttpClient 练习创建和删除客户

扩展 `Northwind.Mvc` 网站项目，使访问者可以在页面上填写表单以创建新客户，或搜索客户然后删除他们。MVC 控制器应调用 Northwind Web 服务来创建和删除客户。

## 练习 16.3 – 探索主题

使用以下页面上的链接，了解本章涵盖的主题的更多细节：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-16---building-and-consuming-web-services`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-16---building-and-consuming-web-services)

# 摘要

在本章中，您学会了如何构建一个 ASP.NET Core Web API 服务，可以被任何可以发出 HTTP 请求并处理 HTTP 响应的平台上的任何应用程序调用。

您还学会了如何使用 Swagger 测试和记录 Web 服务 API，以及如何高效地使用服务。

在下一章中，您将学习如何使用 Blazor 构建用户界面，这是微软的新组件技术，使开发人员能够使用 C# 而不是 JavaScript 为网站构建客户端单页应用程序（SPA）、桌面混合应用程序和潜在的移动应用程序。
