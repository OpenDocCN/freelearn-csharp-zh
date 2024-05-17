# 第十五章：15

# 介绍 ASP.NET Core MVC

在本章中，你将学习如何实现应用程序的表示层。具体来说，你将学习如何基于 ASP.NET Core MVC 实现 Web 应用程序。

ASP.NET Core 是一个实现 Web 应用程序的.NET 框架。ASP.NET Core 在之前的章节中已经部分描述过，因此本章主要将重点放在 ASP.NET Core MVC 上。具体来说，本章将涵盖以下主题：

+   理解 Web 应用程序的表示层

+   理解 ASP.NET Core MVC 结构

+   ASP.NET Core 的最新版本有哪些新特性？

+   理解 ASP.NET Core MVC 与设计原则之间的联系

+   用例 - 在 ASP.NET Core MVC 中实现 Web 应用程序

我们将回顾并详细介绍 ASP.NET Core 框架的结构，部分内容在*第十四章*，*使用.NET Core 应用服务导向架构*，和*第四章*，*决定最佳基于云的解决方案*中已经讨论过。这里的重点是如何基于所谓的**模型视图控制器**（**MVC**）架构模式实现基于 Web 的表示层。

我们还将分析 ASP.NET Core 5.0 版本中所有新功能，以及在 ASP.NET Core MVC 框架中包含的或在典型的 ASP.NET Core MVC 项目中使用的架构模式。其中一些模式在*第十一章*，*设计模式和.NET 5 实现*，和*第十二章*，*理解软件解决方案中的不同领域*中已经讨论过，而另一些，如 MVC 模式本身，是新的。

本章末尾的实际示例将教你如何实现一个 ASP.NET Core MVC 应用程序，以及如何组织整个 Visual Studio 解决方案。该示例描述了一个完整的 ASP.NET Core MVC 应用程序，用于编辑 WWTravelClub 书籍用例的包。

# 技术要求

本章需要免费的 Visual Studio 2019 社区版或更高版本，并安装了所有数据库工具。

本章中的所有概念都将以基于 WWTravelClub 书籍用例的实际示例进行澄清。本章的代码可在[`github.com/PacktPublishing/Software-Architecture-with-C-9-and-.NET-5`](https://github.com/PacktPublishing/Software-Architecture-with-C-9-and-.NET-5)上找到。

# 理解 Web 应用程序的表示层

本章讨论了基于 ASP.NET Core 框架的 Web 应用程序的表示层的架构。Web 应用程序的表示层基于三种技术：

+   **通过 REST 或 SOAP 服务与服务器交换数据的移动或桌面本地应用程序**：我们没有讨论它们，因为它们严格绑定到客户端设备及其操作系统。因此，分析它们完全超出了本书的范围，这需要一本专门的书。

+   **单页应用程序**（**SPA**）：这些是基于 HTML 的应用程序，其动态 HTML 是在客户端使用 JavaScript 或 WebAssembly（一种可以作为 JavaScript 的高性能替代品的跨浏览器汇编）创建的。与本地应用程序一样，SPA 通过 REST 或 SOAP 服务与服务器交换数据，但它们的优势在于独立于设备及其操作系统，因为它们在浏览器中运行。*第十六章*，*Blazor WebAssembly*，描述了基于 WebAssembly 的 Blazor SPA 框架，因为它本身是基于 WebAssembly 编译的.NET 运行时。

+   **由服务器创建的 HTML 页面，其内容取决于要向用户显示的数据**：本章将讨论的 ASP.NET Core MVC 框架是用于创建这种动态 HTML 页面的框架。

本章的其余部分侧重于如何在服务器端创建 HTML 页面，更具体地说，侧重于 ASP.NET Core MVC，该内容将在下一节中介绍。

# 理解 ASP.NET Core MVC 结构

ASP.NET Core 基于通用主机的概念，如*第五章*的*将微服务架构应用于企业应用程序*的*使用通用主机*子部分中所解释的。ASP.NET Core 的基本架构在*第十四章*的*应用.NET Core 实现面向服务的架构*的*简短介绍 ASP.NET Core*子部分中概述了。

值得提醒您的是，主机配置是通过在`Startup.cs`文件中定义的`Startup`类来委托的，方法是调用`IWebHostBuilder`接口的`.UseStartup<Startup>()`方法。`Startup`类的`ConfigureServices(IServiceCollection services)`定义了可以通过**依赖注入**（**DI**）注入到对象构造函数中的所有服务。DI 在*第五章*的*将微服务架构应用于企业应用程序*的*使用通用主机*子部分中有详细描述。

另一方面，`Configure(IApplicationBuilder app, IWebHostEnvironment env)`启动方法定义了所谓的 ASP.NET Core 管道，它在*第十四章*的*应用.NET Core 实现面向服务的架构*的*简短介绍 ASP.NET Core*子部分中简要描述，并将在下一子部分中进行更详细的描述。

## ASP.NET Core 管道的工作原理

ASP.NET Core 提供了一组可配置的模块，您可以根据需要组装。每个模块负责您可能需要也可能不需要的功能。此类功能的示例包括授权、身份验证、静态文件处理、协议协商、CORS 处理等。由于大多数模块对传入请求和最终响应进行转换，因此通常将这些模块称为**中间件**。

您可以通过将它们插入到称为**ASP.NET Core 管道**的通用处理框架中，组合所有需要的模块。

更具体地说，ASP.NET Core 请求通过将上下文对象推送通过 ASP.NET Core 模块的管道来处理，如下图所示：

![](img/B16756_15_01.png)

图 15.1：ASP.NET Core 管道

插入到管道中的对象是包含传入请求数据的`HttpContext`实例。更具体地说，`HttpContext`的`Request`属性包含一个`HttpRequest`对象，其属性以结构化方式表示传入请求。有关标头、cookie、请求路径、参数、表单字段和请求正文的属性。

如果我们将它们写入`HttpContext`实例的`Response`属性中包含的`HttpResponse`对象中，各种模块可以为最终响应的构建做出贡献。`HttpResponse`类类似于`HttpRequest`类，但其属性指的是正在构建的响应。

一些模块可以构建一个中间数据结构，然后由管道中的其他模块使用。一般来说，这样的中间数据可以存储在`HttpContext`对象的`Items`属性中包含的`IDictionary<object, object>`的自定义条目中。但是，有一个预定义的属性`User`，其中包含有关当前登录用户的信息。已登录用户不会自动计算，因此它们必须由身份验证模块计算。*第十四章*的*应用.NET Core 实现面向服务的架构*的*ASP.NET Core 服务授权*子部分解释了如何向 ASP.NET Core 管道添加执行基于 JWT 令牌的身份验证的标准模块。

`HttpContext`还有一个`Connection`属性，其中包含与客户端建立的基础连接的信息，以及一个`WebSockets`属性，其中包含与客户端建立的可能的基于 WebSocket 的连接的信息。

`HttpContext`还有一个`Features`属性，其中包含`IDictionary<Type, object>`，指定了托管 Web 应用程序和管道模块的 Web 服务器支持的功能。功能可以使用`.Set<TFeature>(TFeature o)`方法进行设置，并可以使用`.Get<TFeature>()`方法进行检索。

Web 服务器功能是由框架自动添加的，而所有其他功能是在处理`HttpContext`时由管道模块添加的。

`HttpContext`还通过其`RequestServices`属性为我们提供了对依赖注入引擎的访问。您可以通过调用`.RequestService.GetService(Type t)`方法或者更好的是建立在其之上的`.GetRequiredService<TService>()`扩展方法来获取由依赖引擎管理的类型的实例。然而，正如我们将在本章的其余部分中看到的那样，所有由依赖注入引擎管理的类型通常都会自动注入到构造函数中，因此这些方法仅在构建自定义**中间件**或其他自定义 ASP.NET Core 引擎时使用。

为了访问`HttpContext`属性，不仅模块可以访问`HttpContext`实例，应用程序代码也可以通过 DI 访问。只需将`IHttpContextAccessor`参数插入到自动依赖注入的类的构造函数中，例如传递给控制器的服务（稍后在本节中），然后访问其`HttpContext`属性。

模块是具有以下结构的任何类：

```cs
public class CoreMiddleware
{
    private readonly RequestDelegate _next;
    public CoreMiddleware(RequestDelegate next, ILoggerFactory 
    loggerFactory)
    {
        ...
        _next = next;
        ...
    }
    public async Task Invoke(HttpContext context)
    {
        /*
            Insert here the module specific code that processes the 
            HttpContext instance before it is passed to the next 
            module.

        */

        await _next.Invoke(context);
        /*
            Insert here other module specific code that processes the 
            HttpContext instance, after all modules that follow this
            module finished their processing.
        */
    }
} 
```

通常，每个模块处理由管道中前一个模块传递的`HttpContext`实例，然后调用`await _next.Invoke(context)`来调用管道中其余模块。当其他模块完成其处理并为客户端准备好响应时，每个模块可以在`_next.Invoke(context)`调用后的代码中执行进一步的响应后处理。

通过在`Startup.cs`文件的`Configure`方法中调用`UseMiddleware<T>`方法，可以在 ASP.NET Core 管道中注册模块，如下所示：

```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env, 
IServiceProvider serviceProvider)
{
    ...
    app.UseMiddleware<MyCustomModule>
    ...
} 
```

当调用`UseMiddleware`时，模块以相同的顺序插入到管道中。由于添加到应用程序的每个功能可能需要多个模块，并且可能需要除添加模块之外的其他操作，通常会定义一个`IApplicationBuilder`扩展，例如`UseMyFunctionality`，如下面的代码所示：

```cs
public static class MyMiddlewareExtensions
{
    public static IApplicationBuilder UseMyFunctionality(this 
    IApplicationBuilder builder,...)
    {
        //other code
        ...
        builder.UseMiddleware<MyModule1>();
        builder.UseMiddleware<MyModule2>();
        ...
        //Other code
        ...
        return builder;
    }
} 
```

之后，可以通过调用`app.UseMyFunctionality(...)`将整个功能添加到应用程序中。例如，可以通过调用`app.UseEndpoints(....)`将 ASP.NET Core MVC 功能添加到 ASP.NET Core 管道中。

通常，使用每个`app.Use...`添加的功能需要将一些.NET 类型添加到应用程序 DI 引擎中。在这些情况下，我们还定义了一个名为`AddMyFunctionality`的`IServiceCollection`扩展，必须在`Startup.cs`文件的`ConfigureServices(IServiceCollection services)`方法中调用。例如，ASP.NET Core MVC 需要像下面这样的调用：

```cs
services.AddControllersWithViews(o =>
{
    //set here MVC options by modifying the o option parameter
} 
```

如果不需要更改默认的 MVC 选项，可以简单地调用`services.AddControllersWithViews()`。

下一小节描述了 ASP.NET Core 框架的另一个重要功能，即如何处理应用程序配置数据。

## 加载配置数据并使用选项框架

当 ASP.NET Core 应用程序启动时，它会从`appsettings.json`和`appsettings.[EnvironmentName].json`文件中读取配置信息（例如数据库连接字符串），其中`EnvironmentName`是取决于应用程序部署位置的字符串值。`EnvironmentName`的典型值如下：

+   `Production` 用于生产部署

+   `Development` 用于开发时

+   `Staging` 用于在测试阶段测试应用程序

从`appsettings.json`和`appsettings.[EnvironmentName].json`文件中提取的两个 JSON 树合并为一个唯一的树，其中`[EnvironmentName].json`中包含的值会覆盖`appsettings.json`相应路径中包含的值。这样，应用程序可以在不同的部署环境中以不同的配置运行。特别是，您可以在每个不同的环境中使用不同的数据库连接字符串，因此在每个不同的环境中使用不同的数据库实例。

`[EnvironmentName]`字符串取自`ASPNETCORE_ENVIRONMENT`操作系统环境变量。反过来，`ASPNETCORE_ENVIRONMENT`可以在应用程序部署期间通过 Visual Studio 的两种方式自动设置：

+   在 Visual Studio 部署期间，Visual Studio 的**发布**向导会创建一个 XML 发布配置文件。如果**发布**向导允许您从下拉列表中选择`ASPNETCORE_ENVIRONMENT`，那么您就完成了！[](img/B16756_15_02.png)

图 15.2：Visual Studio 部署设置

否则，您可以按照以下步骤进行：

1.  在向导中填写信息后，保存发布配置文件而不进行发布。

1.  然后，使用文本编辑器编辑配置文件并添加 XML 属性，例如`<EnvironmentName>Staging</EnvironmentName>`。由于应用程序发布期间可以选择所有已定义的发布配置文件，因此您可以为每个环境定义不同的发布配置文件，然后在每次发布时选择所需的配置文件。

+   在应用程序的 Visual Studio ASP.NET Core 项目文件（`.csproj`）中添加以下代码，可以指定部署期间必须将`ASPNETCORE_ENVIRONMENT`设置为的值：

```cs
<PropertyGroup> 
    <EnvironmentName>Staging</EnvironmentName>
</PropertyGroup> 
```

在 Visual Studio 中进行开发时，可以在 ASP.NET Core 项目的`Properties\launchSettings.json`文件中指定应用程序运行时要给`ASPNETCORE_ENVIRONMENT`的值。`launchSettings.json`文件包含几个命名的设置组。这些设置配置了在从 Visual Studio 运行 Web 应用程序时如何启动。您可以通过选择运行按钮旁边的下拉列表中的组名来应用组的所有设置：

![](img/B16756_15_03.png)

图 15.3：启动设置组的选择

您从下拉列表中的选择将显示在运行按钮上，默认选择为**IIS Express**。

以下代码显示了一个典型的`launchSettings.json`文件，您可以在其中添加新的设置组或更改现有默认组的设置：

```cs
{
  "iisSettings": {
    "windowsAuthentication": false, 
    "anonymousAuthentication": true, 
    "iisExpress": {
      "applicationUrl": "http://localhost:2575",
      "sslPort": 44393
    }
  },
  "profiles": {
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    ...
    ...
    }
  }
} 
```

命名的设置组位于`profiles`属性下。在那里，您可以选择在哪里托管应用程序（`IISExpress`），在哪里启动浏览器以及一些环境变量的值。

通过`IWebHostEnvironment`接口可以在 ASP.NET Core 管道定义期间测试从`ASPNETCORE_ENVIRONMENT`操作系统环境变量中加载的当前环境。这是因为`IWebHostEnvironment`实例作为参数传递给`Startup.cs`文件的`Configure`方法。`IWebHostEnvironment`也通过 DI 对用户代码的其余部分可用。

`IWebHostEnvironment.IsEnvironment(string environmentName)`检查`ASPNETCORE_ENVIRONMENT`的当前值是否为`environmentName`。还有特定的快捷方式用于测试开发（`.IsDevelopment()`）、生产（`.IsProduction()`）和暂存（`.IsStaging()`）。`IWebHostEnvironment`还包含 ASP.NET Core 应用程序的当前根目录（`.WebRootPath`）和用于静态文件的目录（`.ContentRootPath`），这些文件由 Web 服务器原样提供（CSS、JavaScript、图像等）。

在 Visual Studio 资源管理器中，`launchSettings.json`和所有发布配置文件都可以作为**Properties**节点的子节点访问，如下截图所示：

![](img/B16756_15_04.png)

图 15.4：启动设置文件

一旦`appsettings.json`和`appsettings.[EnvironmentName].json`被加载，它们合并后的配置树可以映射到.NET 对象的属性上。例如，假设我们在`appsettings`文件中有一个`Email`部分，其中包含连接到电子邮件服务器所需的所有信息，如下所示：

```cs
{
    "ConnectionStrings": {
        "DefaultConnection": "...."
    },
    "Logging": {
        "LogLevel": {
            "Default": "Warning"
        }
    },
    "Email": {
        "FromName": "MyName",
        "FromAddress": "info@MyDomain.com",
        "LocalDomain": "smtps.MyDomain.com",
        "MailServerAddress": "smtps.MyDomain.com",
        "MailServerPort": "465",
        "UserId": "info@MyDomain.com",
        "UserPassword": "mypassword" 
```

然后，整个`Email`部分可以映射到以下类的实例：

```cs
 public class EmailConfig
    {
        public String FromName { get; set; }
        public String FromAddress { get; set; }
        public String LocalDomain { get; set; }
        public String MailServerAddress { get; set; }
        public String MailServerPort { get; set; }
        public String UserId { get; set; }
        public String UserPassword { get; set; }
    } 
```

执行映射的代码必须插入到`Startup.cs`文件的`ConfigureServices`方法中，因为`EmailConfig`实例将通过 DI 可用。我们需要的代码如下所示：

```cs
public Startup(IConfiguration configuration)
{
    Configuration = configuration;
}
....
public void ConfigureServices(IServiceCollection services)
{
    ...
    services.Configure<EmailConfig>(Configuration.GetSection("Email"));
    .. 
```

一旦我们配置了上述设置，需要`EmailConfig`数据的类必须声明一个由 DI 引擎提供的`IOptions<EmailConfig> options`参数。`EmailConfig`实例包含在`options.Value`中。

值得一提的是，选项类的属性可以应用于我们将用于 ViewModels 的相同验证属性（参见*服务器端和客户端验证*小节）。

下一小节描述了 ASP.NET Core MVC 应用程序所需的基本 ASP.NET Core 管道模块。

## 定义 ASP.NET Core MVC 管道

如果在 Visual Studio 中创建一个新的 ASP.NET Core MVC 项目，在`Startup.cs`文件的`Configure`方法中会创建一个标准管道。在那里，如果需要，可以添加更多模块或更改现有模块的配置。

`Configure`方法的初始代码处理错误并执行基本的 HTTPS 配置：

```cs
if (env.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}
app.UseHttpsRedirection(); 
```

如果有错误，如果应用程序处于开发环境中，`UseDeveloperExceptionPage`安装的模块会向响应中添加详细的错误报告。这个模块是一个有价值的调试工具。

如果应用程序在非开发模式下发生错误，`UseExceptionHandler`将从接收到的路径恢复请求处理；也就是说，从`/Home/Error`开始。换句话说，它模拟了一个带有`/Home/Error`路径的新请求。此请求被推送到标准 MVC 处理，直到达到与`/Home/Error`路径相关联的端点，开发人员应该在那里放置处理错误的自定义代码。

当应用程序不处于开发模式时，`UseHsts`会向响应中添加`Strict-Transport-Security`头，通知浏览器只能使用 HTTPS 访问应用程序。声明后，符合规范的浏览器应自动将应用程序的任何 HTTP 请求转换为`Strict-Transport-Security`头中指定的时间内的 HTTPS 请求。默认情况下，`UseHsts`在头中指定 30 天的时间，但您可以通过向`Startup.cs`的`ConfigureServices`方法添加一个`options`对象来指定不同的时间和其他头参数。

```cs
services.AddHsts(options =>     {
    ...
    options.MaxAge = TimeSpan.FromDays(60); 
    ...
}); 
```

`UseHttpsRedirection`在接收到 HTTP URL 时会自动重定向到 HTTPS URL，以强制进行安全连接。一旦建立了第一个 HTTPS 安全连接，`Strict-Transport-Security`头将阻止未来可能用于执行中间人攻击的重定向。

以下代码显示了默认管道的其余部分：

```cs
app.UseStaticFiles();
app.UseCookiePolicy();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
... 
```

`UseStaticFiles` 使项目中 `wwwroot` 文件夹中包含的所有文件（通常是 CSS、JavaScript、图像和字体文件）通过其实际路径从 web 访问。

`UseCookiePolicy` 在 .NET 5 模板中已被移除，但您仍然可以手动添加。它确保 ASP.NET Core 管道处理 cookie，但仅在用户已经同意使用 cookie 的情况下。对 cookie 使用的同意是通过同意 cookie 给出的；也就是说，只有在请求 cookie 中找到此同意 cookie 时才启用 cookie 处理。当用户单击同意按钮时，此 cookie 必须由 JavaScript 创建。包含同意 cookie 名称及其内容的整个字符串可以从 `HttpContext.Features` 中检索，如下面的代码片段所示：

```cs
var consentFeature = context.Features.Get<ITrackingConsentFeature>();
var showBanner = !consentFeature?.CanTrack ?? false;
var cookieString = consentFeature?.CreateConsentCookie(); 
```

只有在需要同意且尚未给出同意时，`CanTrack` 才为 `true`。检测到同意 cookie 时，`CanTrack` 被设置为 `false`。这样，只有在需要同意且尚未给出同意时，`showBanner` 才为 `true`。因此，它告诉我们是否要向用户请求同意。

```cs
CookiePolicyOptions in the code instead of using the configuration file:
```

```cs
services.Configure<CookiePolicyOptions>(options =>
{
    options.CheckConsentNeeded = context => true;
}); 
```

`UseAuthentication` 启用身份验证方案，仅在创建项目时选择身份验证方案时才会出现。

可以通过在 `ConfigureServices` 方法中配置选项对象来启用特定的身份验证方案，如下所示：

```cs
services.AddAuthentication(o =>
{
    o.DefaultScheme = 
    CookieAuthenticationDefaults.AuthenticationScheme;
})
.AddCookie(o =>
{
    o.Cookie.Name = "my_cookie";
})
.AddJwtBearer(o =>
{
    ...
}); 
```

上述代码指定了自定义身份验证 cookie 名称，并为应用程序中包含的 REST 服务添加了基于 JWT 的身份验证。`AddCookie` 和 `AddJwtBearer` 都有重载，可以在操作之前接受身份验证方案的名称，这是您可以定义身份验证方案选项的地方。由于身份验证方案名称是指定特定身份验证方案的必要条件，因此在未指定时会使用默认名称：

+   `CookieAuthenticationDefaults.AuthenticationScheme` 中包含的 cookie 身份验证的标准名称。

+   `JwtBearerDefaults.AuthenticationScheme` 中包含的 JWT 身份验证的标准名称。

传递给 `o.DefaultScheme` 的名称选择用于填充 `HttpContext` 的 `User` 属性的身份验证方案。除了 `DefaultScheme`，还有其他属性允许进行更高级的自定义。

有关 JWT 身份验证的更多信息，请参阅 *第十四章* *使用 .NET Core 应用服务导向架构* 的 *ASP.NET Core 服务授权* 子章节。

如果只指定 `services.AddAuthentication()`，则假定使用具有默认参数的基于 cookie 的身份验证。

`UseAuthorization` 启用基于 `Authorize` 属性的授权。可以通过在 `ConfigureServices` 方法中放置 `AddAuthorization` 方法来配置选项。这些选项允许您定义基于声明的授权策略。

有关授权的更多信息，请参阅 *第十四章* *使用 .NET Core 应用服务导向架构* 的 *ASP.NET Core 服务授权* 子章节。

`UseRouting` 和 `UseEndpoints` 处理所谓的 ASP.NET Core 端点。端点是服务特定类的 URL 的处理程序的抽象。这些 URL 被转换为具有模式的 `Endpoint` 实例。当模式与 URL 匹配时，将创建 `Endpoint` 实例，并填充模式的名称和从 URL 提取的数据。这是将 URL 部分与模式的命名部分进行匹配的结果。这可以在以下代码片段中看到：

```cs
Request path: /UnitedStates/NewYork 
Pattern: Name="location", match="/{Country}/{Town}"
Endpoint: DisplayName="Location", Country="UnitedStates", Town="NewYork" 
```

`UseRouting` 添加一个模块，用于处理请求路径以获取请求的 `Endpoint` 实例，并将其添加到 `HttpContext.Features` 字典中的 `IEndpointFeature` 类型下。实际的 `Endpoint` 实例包含在 `IEndpointFeature` 的 `Endpoint` 属性中。

每个模式还包含应处理与模式匹配的所有请求的处理程序。创建`Endpoint`时，将此处理程序传递给`Endpoint`。

另一方面，`UseEndpoints`添加了执行由`UseRouting`逻辑确定的路由的中间件。它放置在管道的末尾，因为其中间件生成最终响应。将路由逻辑拆分为两个单独的中间件模块使得授权中间件可以坐在它们之间，并根据匹配的端点决定是否将请求传递给`UseEndpoints`中间件进行正常执行，还是立即返回 401（未经授权）/403（禁止）响应。

```cs
UseRouting middleware, but they are listed in the UseEndpoints method. While it might appear strange that URL patterns are not defined directly in the middleware that uses them, this was done mainly for coherence with the previous ASP.NET Core versions. In fact, previous versions contained no method analogous to UseRouting, but a unique middleware at the end of the pipeline. In the new version, patterns are still defined at the end of the pipeline for coherence with previous versions, but now, UseEndpoints just creates a data structure containing all patterns when the application starts. Then, this data structure is processed by the UseRouting middleware, as shown in the following code:
```

```cs
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");

}); 
```

`MapControllerRoute`定义了与 MVC 引擎相关的模式，这将在下一小节中描述。还有其他定义其他类型模式的方法。例如，像`.MapHub<MyHub>("/chat")`这样的调用将路径映射到处理**SignalR**的 hub，这是建立在`WebSocket`之上的抽象，而`.MapHealthChecks("/health")`将路径映射到返回应用程序健康数据的 ASP.NET Core 组件。您还可以直接将模式映射到自定义处理程序，例如`.MapGet`拦截 GET 请求，`.MapPost`拦截 POST 请求。这称为**路由到代码**。以下是`MapGet`的示例：

```cs
MapGet("hello/{country}", context => 
    context.Response.WriteAsync(
    $"Selected country is {context.GetRouteValue("country")}")); 
```

模式按照定义的顺序进行处理，直到找到匹配的模式为止。由于身份验证/授权中间件放置在路由中间件之后，它可以处理`Endpoint`请求，以验证当前用户是否具有执行`Endpoint`处理程序所需的授权。否则，将立即返回 401（未经授权）或 403（禁止）响应。只有通过身份验证和授权的请求才会由`UseEndpoints`中间件执行其处理程序。

在*第十四章*中描述的 ASP.NET Core RESTful API 中，ASP.NET Core MVC 还使用放置在控制器或控制器方法上的属性来指定授权规则。但是，也可以将`AuthorizeAttribute`的实例添加到模式中，以将其授权约束应用于匹配该模式的所有 URL，如下例所示：

```cs
endpoints
 .MapHealthChecks("/health")
 .RequireAuthorization(new AuthorizeAttribute(){ Roles = "admin", }); 
```

上述代码使健康检查路径仅对管理员用户可用。

在描述了 ASP.NET Core 框架的基本结构之后，我们现在可以转向更多 MVC 特定的功能。下一小节描述了控制器，并解释了它们如何通过 ViewModel 与称为 Views 的 UI 组件进行交互。

## 定义控制器和 ViewModels

`UseEndpoints`中的各种`.MapControllerRoute`调用将 URL 模式与控制器及这些控制器的方法关联起来，其中控制器是从`Microsoft.AspNetCore.Mvc.Controller`类继承的类。控制器是通过检查应用程序的所有`.dll`文件并将其添加到 DI 引擎来发现的。这项工作是由`startup.cs`文件的`ConfigureServices`方法中对`AddControllersWithViews`的调用执行的。

`UseEndpoints`添加的管道模块从`controller`模式变量中获取控制器名称，并从`action`模式变量中获取要调用的控制器方法的名称。由于按照惯例，所有控制器名称都应以`Controller`后缀结尾，因此实际的控制器类型名称是通过在`controller`变量中找到的名称后添加此后缀来获得的。因此，例如，如果在`controller`中找到的名称是`"Home"`，那么`UseEndpoints`模块会尝试从 DI 引擎中获取`HomeController`类型的实例。路由规则可以选择所有控制器的公共方法。通过使用`[NonAction]`属性装饰，可以防止使用控制器的公共方法。路由规则可用的所有控制器方法都称为操作方法。

MVC 控制器的工作方式类似于我们在*第十四章*的*使用.NET Core 实现 REST 服务*中描述的 API 控制器。唯一的区别是 API 控制器预期生成 JSON 或 XML，而 MVC 控制器预期生成 HTML。因此，虽然 API 控制器继承自`ControllerBase`类，但 MVC 控制器继承自`Controller`类，后者又继承自`ControllerBase`类，并添加了对 HTML 生成有用的方法，例如调用视图，在下一小节中描述，并创建重定向响应。

MVC 控制器也可以使用类似于 API 控制器之一的路由技术；即基于控制器和控制器方法属性的路由。通过在`UseEndpoints`中调用`MapControllerRoute()`方法来启用此行为。如果此调用放置在所有其他`MapControllerRoute`调用之前，则控制器路由优先于`MapControllerRoute`模式；否则，情况相反。

我们在 API 控制器中看到的所有属性也可以用于 MVC 控制器和动作方法（`HttpGet`，`HttpPost`，`...Authorize`等）。开发人员可以通过继承`ActionFilter`类或其他派生类来编写自定义属性。我现在不会详细介绍这些内容，但这些细节可以在官方文档中找到，官方文档在*进一步阅读*部分中有提到。

当`UseEndpoints`模块调用控制器时，由于控制器实例本身是由 DI 引擎返回的，因此所有构造函数参数都由 DI 引擎填充，并且由于 DI 会以递归方式自动填充构造函数参数。

另一方面，动作方法参数来自以下来源：

+   请求标头

+   当前请求匹配的模式中的变量

+   查询字符串参数

+   表单参数（在 POST 请求的情况下）

+   依赖注入（DI）

使用 DI 填充的参数按类型匹配，而所有其他参数按*名称*匹配，忽略字母大小写。也就是说，动作方法参数名称必须与标头、查询字符串、表单或模式变量匹配。当参数是复杂类型时，将在每个属性中搜索匹配，使用属性名称进行匹配。在嵌套复杂类型的情况下，将在每个嵌套属性的路径中搜索匹配，并且与路径相关联的名称是通过将路径中的所有属性名称链接在一起并用点分隔来获得的。例如，`Property1.Property2.Property3...Propertyn`是由嵌套属性`Property1`、`Property2`、....、`Propertyn`组成的路径相关联的名称。以这种方式获得的名称必须与标头名称、模式变量名称、查询字符串参数名称等匹配。例如，包含复杂`Address`对象的`OfficeAddress`属性将生成名称如`OfficeAddress.Country`、`OfficeAddress.Town`等。

默认情况下，简单类型参数与模式变量和查询字符串变量匹配，而复杂类型参数与表单参数匹配。然而，可以通过在参数前加上属性来更改前述默认值，如下所述：

+   `[FromForm]` 强制与表单参数匹配

+   `[FromHeader]` 强制与请求标头匹配

+   `[FromRoute]` 强制与模式变量匹配

+   `[FromQuery]` 强制与查询字符串变量匹配

+   `[FromServices]` 强制使用 DI

在匹配期间，从所选源中提取的字符串将使用当前线程文化转换为操作方法参数的类型。如果转换失败或者对于必需的操作方法参数找不到匹配项，则整个操作方法调用过程将失败，并且将自动返回 404 响应。例如，在以下示例中，`id`参数将与查询字符串参数或模式变量匹配，因为它是一个简单类型，而`myclass`属性和嵌套属性将与表单参数匹配，因为`MyClass`是一个复杂类型。最后，`myservice`将从 DI 中获取，因为它带有`[FromServices]`属性前缀：

```cs
 public class HomeController : Controller
    {
        public IActionResult MyMethod(
            int id, 
            MyClass myclass, 
            [FromServices] MyService myservice)
        {
            ... 
```

如果在`UseEndpoints`模式中找不到`id`参数的匹配项，并且`id`参数在`UseEndpoints`模式中被声明为必需的，由于模式匹配失败，将自动返回 404 响应。当参数必须匹配非空单一类型时，通常将参数声明为非可选。如果在 DI 容器中找不到`MyService`实例，则会抛出异常，因为在这种情况下，失败不是由于错误的请求，而是由于设计错误。

MVC 控制器返回`IActionResult`接口或`Task<IActionResult>`结果，如果它们被声明为`async`。`IActionResult`定义了具有`ExecuteResultAsync(ActionContext)`签名的唯一方法，当由框架调用时，会产生实际的响应。

对于每个不同的`IActionResult`，MVC 控制器都有返回它们的方法。最常用的`IActionResult`是`ViewResult`，它是由`View`方法返回的：

```cs
public IActionResult MyMethod(...)
{
   ...
   return View("myviewName", MyViewModel)
} 
```

`ViewResult`是控制器创建 HTML 响应的一种非常常见的方式。更具体地说，控制器与业务/数据层交互，以产生将显示在 HTML 页面中的数据的抽象。这种抽象是一个称为**ViewModel**的对象。ViewModel 作为第二个参数传递给`View`方法，而第一个参数是一个名为 View 的 HTML 模板的名称，该模板包含 ViewModel 中的数据。

总结一下，MVC 控制器的处理顺序如下：

1.  控制器执行一些处理以创建 ViewModel，这是要显示在 HTML 页面上的数据的抽象。

1.  然后，控制器通过将视图名称和 ViewModel 传递给`View`方法来创建`ViewResult`。

1.  MVC 框架调用`ViewResult`并导致包含在 View 中的模板与 ViewModel 中包含的数据实例化。

1.  模板实例化的结果将以适当的标头写入响应。

这样，控制器通过构建 ViewModel 执行 HTML 生成的概念性工作，而视图 - 也就是模板 - 则负责所有的图形细节。

视图将在下一小节中详细描述，而模型（ViewModel）视图控制器模式将在本章的*理解 ASP.NET Core MVC 和设计原则之间的联系*部分中更详细地讨论。最后，在本章的*用例 - 在 ASP.NET Core MVC 中实现 Web 应用程序*部分将提供一个实际的例子。

另一个常见的`IActionResult`是`RedirectResult`，它创建一个重定向响应，因此强制浏览器转到特定的 URL。一旦用户成功提交完成先前操作的表单，通常会使用重定向。在这种情况下，通常会将用户重定向到可以选择另一个操作的页面。

返回`RedirectResult`最简单的方法是通过将 URL 传递给`Redirect`方法。这是执行重定向到 Web 应用程序之外的 URL 的建议方式。另一方面，当 URL 在 Web 应用程序内部时，建议使用`RedirectToAction`方法，该方法接受控制器名称、操作方法名称和目标操作方法的所需参数。框架使用这些数据来计算一个 URL，以导致所需的操作方法被调用并提供参数。这样，如果在应用程序的开发或维护过程中更改了路由规则，框架会自动更新新的 URL，无需修改代码中旧 URL 的所有出现。

以下代码显示了如何调用`RedirectToAction`：

```cs
return RedirectToAction("MyActionName", "MyControllerName",
         new {par1Name=par1Value,..parNName=parNValue}); 
```

另一个有用的`IActionResult`是`ContentResult`，可以通过调用`Content`方法创建。`ContentResult`允许您将任何字符串写入响应并指定其 MIME 类型，如下例所示：

```cs
return Content("this is plain text", "text/plain"); 
```

最后，`File`方法返回`FileResult`，它在响应中写入二进制数据。该方法有几个重载，允许指定字节数组、流或文件的路径，以及二进制数据的 MIME 类型。

现在，让我们继续描述在 Views 中生成实际 HTML 的方法。

## 理解 Razor 视图

ASP.NET Core MVC 使用一种称为 Razor 的语言来定义 Views 中包含的 HTML 模板。Razor 视图是文件，当它们首次使用时，当应用程序构建时，或者当应用程序发布时，它们会被编译为.NET 类。默认情况下，每次构建和发布都启用了预编译，但您还可以启用运行时编译，以便在部署后可以修改 Views。在 Visual Studio 中创建项目时，可以通过选中**启用 Razor 运行时编译**复选框来启用此选项。您还可以通过向 Web 应用程序项目文件添加以下代码来禁用每次构建和发布的编译：

```cs
<PropertyGroup>
  <TargetFramework> net5.0 </TargetFramework>
  <!-- add code below -->
  <RazorCompileOnBuild>false</RazorCompileOnBuild>
  <RazorCompileOnPublish>false</RazorCompileOnPublish>
  <!-- end of code to add -->
    ...
</PropertyGroup> 
```

如果选择了 Razor 视图库项目，则视图也可以预编译为视图库。

此外，在编译后，视图仍然与其路径相关联，这些路径成为它们的完整名称。每个控制器在**Views**文件夹下有一个与控制器同名的关联文件夹，该文件夹预计包含该控制器使用的所有视图。

以下屏幕截图显示了与`HomeController`及其 Views 相关联的文件夹：

![](img/B16756_15_05.png)

图 15.5：与控制器相关联的视图文件夹和共享文件夹

前面的屏幕截图还显示了**Shared**文件夹，该文件夹预计包含多个控制器使用的所有视图或部分视图。控制器通过其路径引用`View`方法中的视图，而不包括`.cshtml`扩展名。如果路径以`/`开头，则将路径解释为相对于应用程序根的路径。否则，首先尝试将路径解释为相对于与控制器关联的文件夹。如果在那里找不到视图，则会在**Shared**文件夹中搜索视图。

因此，例如，在前面的屏幕截图中的`Privacy.cshtml`视图文件可以在`HomeController`中从`View("Privacy", MyViewModel)`中引用。如果 View 的名称与操作方法的名称相同，我们可以简单地写`View(MyViewModel)`。

Razor 视图是 HTML 代码与 C#代码的混合，加上一些特定于 Razor 的语句。它们都以包含 View 应该接收的 ViewModel 类型的标题开头：

```cs
@model MyViewModel 
```

每个视图还可以包含一些`using`语句，其效果与标准代码文件的`using`语句相同：

```cs
@model MyViewModel
@using MyApplication.Models 
```

在特殊的`_ViewImports.cshtml`文件中声明的`@using`语句（即在`Views`文件夹的根目录）会自动应用于所有视图。

每个视图还可以在其头部使用以下语法要求 DI 引擎中的类型的实例：

```cs
@model MyViewModel 
@using MyApplication.Models
@inject IViewLocalizer Localizer 
```

前面的代码需要`IViewLocalizer`接口的一个实例，并将其放在`Localizer`变量中。视图的其余部分是 C#代码、HTML 和 Razor 流程控制语句的混合。视图的每个区域可以是 HTML 模式或 C#模式。在 HTML 模式的视图区域中的代码被解释为 HTML，而在 C#模式的视图区域中的代码被解释为 C#。

接下来的主题将解释 Razor 流程控制语句。

### 学习 Razor 流程控制语句

如果要在 HTML 区域中编写一些 C#代码，可以使用`@{..}` Razor 流程控制语句创建一个 C#区域，如下所示：

```cs
@{
    //place C# code here
    var myVar = 5;
    ...
    <div>
        <!-- here you are in HTML mode again -->
        ...
    </div>
    //after the HTML block you are still in C# mode
    var x = "my string";
} 
```

前面的示例表明，只需编写 HTML 标签即可在 C#区域内创建 HTML 区域，依此类推。一旦 HTML 标签关闭，您又处于 C#模式。

C#代码不会产生 HTML，而 HTML 代码按照出现的顺序添加到响应中。您可以通过在 HTML 模式下使用`@`前缀来添加使用 C#代码计算的文本。如果表达式复杂，由一系列属性和方法调用组成，必须用括号括起来。以下代码显示了一些示例：

```cs
<span>Current date is: </span>
<span>@DateTime.Today.ToString("d")</span>
...
<p>
  User name is: @(myName+ " "+mySurname)
</p>
...
<input type="submit" value="@myUserMessage" /> 
```

类型使用当前的区域设置转换为字符串（有关如何设置每个请求的区域设置的详细信息，请参见*了解 ASP.NET Core MVC 和设计原则之间的关系*部分）。此外，字符串会自动进行 HTML 编码，以避免`<`和`>`符号干扰视图 HTML。可以使用`@HTML.Raw`函数来防止 HTML 编码，如下所示：

```cs
@HTML.Raw(myDynamicHtml) 
```

在 HTML 区域中，可以使用`@if` Razor 语句选择替代 HTML：

```cs
@if(myUser.IsRegistered)
{
    //this is a C# code area
    var x=5;
    ...
    <p>
     <!-- This is an HTML area -->
    </p>
    //this is a C# code area again
}
else if(callType == CallType.WebApi)
{
    ...
}
else
{
 ..
} 
```

如前面的代码所示，Razor 流程控制语句的每个块的开始都是在 C#模式下，并且直到遇到第一个 HTML 开放标签之前都保持在 C#模式下，然后开始 HTML 模式。在相应的 HTML 关闭标签之后，会恢复 C#模式。

可以使用`for`、`foreach`、`while`和`do` Razor 语句多次实例化 HTML 模板，如下例所示：

```cs
@for(int i=0; i< 10; i++)
{
}
@foreach(var x in myIEnumerable)
{
}
@while(true)
{

}
@do 
{

}
while(true) 
```

Razor 视图可以包含不生成任何代码的注释。在`@*...*@`中包含的任何文本都被视为注释，并在页面编译时被移除。下一个主题描述了所有视图中可用的属性。

### 了解 Razor 视图属性

每个视图中都预定义了一些标准变量。最重要的变量是`Model`，它包含传递给视图的 ViewModel。例如，如果我们将一个`Person`模型传递给一个视图，那么`<span>@Model.Name</span>`会显示传递给视图的`Person`模型的名称。

`ViewData`变量包含`IDictionary<string, object>`，与调用视图的控制器共享。也就是说，所有控制器都有一个包含`IDictionary<string, object>`的`ViewData`属性，并且在控制器中设置的每个条目也可以在调用视图的`ViewData`变量中使用。`ViewData`是控制器传递信息给调用视图的替代方法。值得一提的是，`ViewState`字典也可以通过`ViewBag`属性作为动态对象进行访问。这意味着动态的`ViewBag`属性被映射到`ViewData`字符串索引，它们的值被映射到与这些索引对应的`ViewState`条目。

`User`变量包含当前登录的用户；也就是说，包含在当前请求的`Http.Context.User`属性中的相同实例。`Url`变量包含一个`IUrlHelper`接口的实例，其方法是用于计算应用程序页面的 URL 的实用程序。例如，`Url.Action("action", "controller", new {par1=valueOfPar1,...})`计算出导致调用*controller*的*action*方法的 URL，并使用作为参数传递的匿名对象中指定的所有参数。

`Context`变量包含整个请求的`HttpContext`。`ViewContext`变量包含有关视图调用上下文的数据，包括有关调用视图的操作方法的元数据。

下一个主题将描述 Razor 如何增强 HTML 标记语法。

### 使用 Razor 标记助手

在 ASP.NET Core MVC 中，开发人员可以定义所谓的标记助手，这些标记助手可以增强现有的 HTML 标记，添加新的标记属性，或者定义新的标记。在 Razor 视图编译时，任何标记都会与现有的标记助手进行匹配。当找到匹配项时，源标记将被标记助手创建的 HTML 替换。可以为同一个标记定义多个标记助手。它们都按照可以通过与每个标记助手关联的优先级属性进行配置的顺序执行。

为同一个标记定义的所有标记助手在处理每个标记实例时可以进行合作。这是因为它们被传递了一个共享的数据结构，其中每个标记助手都可以应用一个贡献。通常，被调用的最终标记助手会处理这个共享的数据结构，以生成输出的 HTML。

标记助手是继承自`TagHelper`类的类。本主题不讨论如何创建新的标记助手，而是介绍了随 ASP.NET Core MVC 一起提供的主要预定义标记助手。如何定义标记助手的完整指南可在官方文档中找到，该文档在*进一步阅读*部分中有引用。

要使用标记助手，必须声明包含它的`.dll`文件，声明如下：

```cs
@addTagHelper *, Dll.Complete.Name 
```

如果您只想使用`.dll`文件中定义的标记助手中的一个，必须用标记名称替换`*`。

前面的声明可以放置在使用库中定义的标记助手的每个视图中，也可以一次性放置在`Views`文件夹的根目录中的`_ViewImports.cshtml`文件中。默认情况下，`_ViewImports.cshtml`添加了所有预定义的 ASP.NET Core MVC 标记助手，声明如下：

```cs
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers 
```

锚标记使用属性进行增强，这些属性可以自动计算 URL，并调用具有给定参数的特定操作方法，如下所示：

```cs
<a asp-controller="{controller name}"
asp-action="{action method name}" 
asp-route-{action method parameter1}="value1"
...
asp-route-{action method parametern}="valuen"> 
    put anchor text here
</a> 
```

类似的语法也适用于`form`标记：

```cs
<form asp-controller="{controller name}"
asp-action="{action method name}" 
asp-route-{action method parameter1}="value1"
...
asp-route-{action method parametern}="valuen"
...
> 
    ... 
```

`script`标记使用属性进行增强，允许我们在下载失败时回退到不同的源。典型用法是从某个云服务下载脚本以优化浏览器缓存，并在失败时回退到脚本的本地副本。以下代码使用回退技术下载`bootstrap` JavaScript 文件：

```cs
<script src="https://stackpath.bootstrapcdn.com/
bootstrap/4.3.1/js/bootstrap.bundle.min.js"
asp-fallback-src="~/lib/bootstrap/dist/js/
bootstrap.bundle.min.js"
asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal" crossorigin="anonymous"
integrity="sha384-xrRywqdh3PHs8keKZN+8zzc5TX0GRTLCcmivcbNJWm2rs5C8PRhcEn3czEjhAO9o">
</script> 
```

`asp-fallback-test`包含一个 JavaScript 测试，用于验证下载是否成功。在前面的示例中，测试验证是否已创建 JavaScript 对象。

`environment`标记可用于选择不同环境（开发、暂存和生产）的不同 HTML。其典型用法是在开发过程中选择 JavaScript 文件的调试版本，如下例所示：

```cs
<environment include="Development">
        @*development version of JavaScript files*@
</environment>
<environment exclude="Development">
        @*development version of JavaScript files *@
</environment> 
```

还有一个`cache`标记，它将其内容缓存在内存中以优化渲染速度：

```cs
<cache>
    @* heavy to compute content to cache *@
</cache> 
```

默认情况下，内容被缓存 20 分钟，但标记具有在缓存过期时必须定义的属性，例如`expires-on="{datetime}"`，`expires-after="{timespan}"`和`expires-sliding="{timespan}"`。在这里，`expires-sliding`和`expires-after`之间的区别在于，在第二个属性中，每次请求内容时，到期时间计数都会被重置。`vary-by`属性导致为传递给`vary-by`的每个不同值创建不同的缓存条目。还有一些属性，例如`vary-by-header`，它为指定属性中指定的请求标头的每个不同值创建一个不同的条目；`vary-by-cookie`；等等。

所有`input`标记 - 也就是`textarea`，`input`和`select` - 都有一个`asp-for`属性，其值接受以视图的 ViewModel 为根的属性路径。例如，如果视图有一个`Person` ViewModel，我们可能会有这样的东西：

```cs
<input type="text" asp-for"Address.Town"/> 
```

前面的代码的第一件事是将`Town`嵌套属性的值分配给`input`标记的`value`属性。通常情况下，如果值不是字符串，它会根据当前请求的区域设置转换为字符串。

但是，它还将输入字段的名称设置为`Address.Town`，将输入字段的 ID 设置为`Address_Town`。这是因为标记 ID 中不允许出现点。

可以通过在`ViewData.TemplateInfo.HtmlFieldPrefix`中指定前缀来添加到这些标准名称。例如，如果前一个属性设置为`MyPerson`，则名称变为`MyPerson.Address.Town`。

如果表单提交到一个具有与其参数之一相同的`Person`类的操作方法，则给`input`字段的`Address.Town`名称将导致该参数的`Town`属性被填充到`input`字段中。通常情况下，`input`字段中包含的字符串会根据当前请求的区域设置转换为其匹配的属性类型。总结一下，`input`字段的名称是以这样的方式创建的，以便在 HTML 页面被提交时，操作方法中可以恢复完整的`Person`模型。

相同的`asp-for`属性可以在`label`标记中使用，以使标签引用具有相同`asp-for`值的输入字段。

以下代码是`input`/`label`对的示例：

```cs
<label asp-for="Address.Town"></label
<input type="text" asp-for="Address.Town"/> 
```

当标签中没有插入文本时，标签中显示的文本将从装饰属性（在本例中为`Town`）中获取；否则，将使用属性的名称。

如果`span`或`div`包含`asp-validation-for ="Address.Town"`错误属性，则与`Address.Town`输入相关的验证消息将自动插入到该标记内。验证框架将在*了解 ASP.NET Core MVC 与设计原则之间的关系*部分中进行描述。

还可以通过在`div`或`span`后添加属性来自动创建验证错误摘要：

```cs
asp-validation-summary="ValidationSummary.{All, ModelOnly}" 
```

如果属性设置为`ValidationSummary.ModelOnly`，则仅会在摘要中显示与特定`input`字段不相关的消息，而如果值为`ValidationSummary.All`，则会显示所有错误消息。

`asp-items`属性可以应用于任何`select`标记，以便自动生成所有`select`选项。必须传递一个`IEnumerable<SelectListItem>`，其中每个`SelectListItem`包含选项的文本和值。`SelectListItem`还包含一个可选的`Group`属性，您可以使用它来将在`select`中显示的选项组织成组。

下一个主题将展示如何重用视图代码。

## 重用视图代码

ASP.NET Core MVC 包括几种重用视图代码的技术。最重要的是布局页面。

在每个 Web 应用程序中，几个页面共享相同的结构；例如，相同的主菜单或相同的左侧或右侧栏。在 ASP.NET Core 中，这种常见结构被分解为称为布局页面/视图的视图中。

每个视图都可以使用以下代码指定要用作其布局页面的视图：

```cs
@{
    Layout = "_MyLayout";
} 
```

如果未指定布局页面，则将使用位于`Views`文件夹中的`_ViewStart.cshtml`文件中定义的默认布局页面。_ViewStart.cshtml 的默认内容如下：

```cs
@{
    Layout = "_Layout";
} 
```

因此，Visual Studio 生成的文件中的默认布局页面是`_Layout.cshtml`，它包含在`Shared`文件夹中。

布局页面包含与其所有子页面共享的 HTML、HTML 页面头和对 CSS 和 JavaScript 文件的页面引用。每个视图生成的 HTML 都放在其布局位置中，布局页面调用`@RenderBody()`方法，如下例所示：

```cs
...
<main role="main" class="pb-3">
    ...
    @RenderBody()
    ...
</main>
... 
```

每个视图的`ViewState`都会被复制到其布局页面的`ViewState`中，因此`ViewState`可用于将信息传递给视图布局页面。通常用于将视图标题传递给布局页面，然后布局页面用它来组成页面的标题头，如下所示：

```cs
@*In the view *@
@{
    ViewData["Title"] = "Home Page";  
}
@*In the layout view*@
<head>
    <meta charset="utf-8" />
    ...
    <title>@ViewData["Title"] - My web application</title>
    ... 
```

虽然每个视图生成的主要内容都放在其布局页面的一个区域中，但每个布局页面还可以定义放置在不同区域的几个部分，每个视图可以在其中放置更多的次要内容。

例如，假设布局页面定义了一个`Scripts`部分，如下所示：

```cs
...
<script src="img/site.js" asp-append-version="true"></script>
@RenderSection("Scripts", required: false)
... 
```

然后，视图可以使用先前定义的部分来传递一些特定于视图的 JavaScript 引用，如下所示：

```cs
.....
@section scripts{
    <script src="img/pageSpecificJavaScript.min.js"></script>
}
..... 
```

如果操作方法预期向 AJAX 调用返回 HTML，则必须生成 HTML 片段而不是整个 HTML 页面。因此，在这种情况下，不必使用布局页面。这可以通过在控制器操作方法中调用`PartialView`方法而不是`View`方法来实现。`PartialView`和`View`具有完全相同的重载和参数。

重用视图代码的另一种方法是将几个视图共有的视图片段分解到另一个由所有先前视图调用的视图中。视图可以使用`partial`标签调用另一个视图，如下所示：

```cs
<partial name="_viewname" for="ModelProperty.NestedProperty"/> 
```

上述代码调用`_viewname`并将`Model.ModelProperty.NestedProperty`中包含的对象作为其`ViewModel`传递。当使用`partial`标签调用视图时，不使用布局页面，因为被调用视图预期返回 HTML 片段。

被调用视图的`ViewData.TemplateInfo.HtmlFieldPrefix`属性设置为`"ModelProperty.NestedProperty"`字符串。这样，在`_viewname.cshtml`中呈现的可能输入字段将具有与直接由调用视图呈现时相同的名称。

可以通过将`for`替换为`model`，直接传递包含在变量中或由 C#表达式返回的对象，而不是通过调用视图（ViewModel）的属性来指定`_viewname`的 ViewModel，如本例所示：

```cs
<partial name="_viewname" model="new MyModel{...})" /> 
```

在这种情况下，被调用视图的`ViewData.TemplateInfo.HtmlFieldPrefix`属性保持其默认值；即空字符串。

视图还可以调用比另一个视图更复杂的内容；也就是说，另一个控制器方法，反过来呈现一个视图。设计为由视图调用的控制器称为**视图组件**。以下代码是组件调用的示例：

```cs
<vc:[view-component-name] par1="par1 value" par2="parameter2 value"> </vc:[view-component-name]> 
```

参数名称必须与视图组件方法中使用的名称匹配。但是，组件的名称和参数名称都必须转换为 kebab case；也就是说，如果原始名称中的所有字符都是大写字母，那么所有字符都必须转换为小写字母，尽管第一个单词必须在前面加上`-`。例如，`MyParam`必须转换为`my-param`。

实际上，视图组件是从`ViewComponent`类派生的类。当调用组件时，框架会查找`Invoke`方法或`InvokeAsync`方法，并将组件调用中定义的所有参数传递给它。如果方法被定义为`async`，则必须使用`InvokeAsync`；否则，必须使用`Invoke`。

以下代码是视图组件定义的示例：

```cs
public class MyTestViewComponent : ViewComponent
    {

        public async Task<IViewComponentResult> InvokeAsync(
        int par1, bool par2)
        {
            var model= ....
            return View("ViewName", model);
        }

    } 
```

先前定义的组件必须通过以下方式调用：

```cs
<vc:my-test par1="10" par2="true"></my-test> 
```

如果组件由名为`MyController`的控制器的视图调用，则在以下路径中搜索`ViewName`：

+   `/Views/MyController/Components/MyTest/ViewName`

+   `/Views/Shared/Components/MyTest/ViewName`

现在，让我们看一下 ASP.NET Core 的更近期的相关功能。

# ASP.NET Core 的最新版本有什么新功能？

ASP.NET Core 的主要变化发生在 3.0 版本：路由引擎已从 MVC 引擎中分离出来，现在也可用于其他处理程序。在以前的版本中，路由和路由是 MVC 处理程序的一部分，并且通过`app.UseMvc(....)`添加；现在已经被`app.UseRouting()`和`UseEndpoints(...)`取代，它们不仅可以将请求路由到控制器，还可以将请求路由到其他处理程序。

现在，端点及其关联的处理程序是在`UseEndpoints`中定义的，如下所示：

```cs
app.UseEndpoints(endpoints =>
    {
        ...
        endpoints.MapControllerRoute("default", "
        {controller=Home}/{action=Index}/{id?}");
        ...
    }); 
```

`MapControllerRoute`将模式与控制器关联起来，但我们也可以使用诸如`endpoints.MapHub<ChatHub>("/chat")`之类的东西，将模式与处理 WebSocket 连接的 hub 关联起来。在前一节中，我们看到模式也可以使用`MapPost`和`MapGet`与自定义处理程序关联。

独立的路由器还允许我们不仅向控制器添加授权，还可以向任何处理程序添加授权，如下所示：

```cs
MapGet("hello/{country}", context => 
    context.Response.WriteAsync(
    $"Selected country is {context.GetRouteValue("country")}"))
    .RequireAuthorization(new AuthorizeAttribute(){ Roles = "admin" }); 
```

此外，ASP.NET Core 现在有一个独立的 JSON 格式化程序，不再依赖于第三方 Newtonsoft JSON 序列化器。但是，如果您有兼容性问题，仍然可以选择通过安装`Microsoft.AspNetCore.Mvc.NewtonsoftJson` NuGet 包并配置控制器来将最小的 ASP.NET Core JSON 格式化程序替换为 Newtonsoft JSON 序列化器，如下所示：

```cs
services.AddControllersWithViews()
    .AddNewtonsoftJson(); 
```

在这里，`AddNewtonsoftJson`还有一个重载，接受 Newtonsoft JSON 序列化器的配置选项：

```cs
.AddNewtonsoftJson(options =>
           options.SerializerSettings.ContractResolver =
              new CamelCasePropertyNamesContractResolver()); 
```

微软的 JSON 序列化器是在版本 3 中引入的，但一开始它的实现是最小的。现在，在.NET 5 中，它提供了与 Newtonsoft JSON 序列化器相当的选项。

在 3.0 之前的版本中，您被迫将控制器和视图都添加到 DI 引擎中。现在，您仍然可以使用`services.AddControllersWithViews`注入控制器和视图，但如果您只打算实现 REST 端点，也可以使用`AddControllers`添加控制器。

由于.NET 性能的改进、JIT 编译器的改进（现在生成更短更优化的代码）以及 HTTP/2 协议实现的改进，版本 5 带来了显著的性能改进。基本上，您可以依赖于加倍的计算速度，以及更高效的内存和垃圾收集处理。

# 理解 ASP.NET Core MVC 与设计原则之间的连接

整个 ASP.NET Core 框架是建立在我们在*第五章*、*将微服务架构应用于企业应用程序*、*第八章*、*在 C#中与数据交互-Entity Framework Core*、*第十一章*、*设计模式和.NET 5 实现*、*第十二章*、*理解软件解决方案中的不同领域*和*第十三章*、*在 C# 9 中实现代码重用性*中分析的设计原则和模式之上构建的。

此外，所有框架功能都通过 DI 提供，因此每个功能都可以被定制的替代品替换，而不会影响其余代码。然而，这些提供者不是单独添加到 DI 引擎中的；相反，它们被分组到选项对象中（参见*加载配置数据并与选项框架一起使用*小节），以符合 SOLID 单一职责原则。例如，所有模型绑定器、验证提供者和数据注释提供者都是如此。

此外，配置数据不再是从配置文件创建的唯一字典中获取，而是通过我们在本章第一节中描述的选项框架组织成选项对象。这也是对 SOLID 接口隔离原则的应用。

然而，ASP.NET Core 还应用了其他模式，这些模式是单一职责原则的特定实例，它是单一职责原则的泛化。它们如下：

+   中间件模块架构（ASP.NET Core 管道）

+   从应用程序代码中分解验证和全球化

+   MVC 模式本身

我们将在接下来的各个小节中分析每一个。

## ASP.NET Core 管道的优势

ASP.NET Core 管道架构具有两个重要的优势：

+   对初始请求执行的所有不同操作都根据单一职责原则分解到不同的模块中。

+   执行这些不同操作的模块不需要相互调用，因为每个模块都是由 ASP.NET Core 框架一次性调用的。这样，每个模块的代码都不需要执行与其他模块分配的责任相关的任何操作。

这确保了功能的最大独立性和更简单的代码。例如，一旦授权和认证模块启用，其他模块就不需要再担心授权问题。每个控制器代码可以专注于特定于应用程序的业务内容。

## 服务器端和客户端验证

验证逻辑已完全从应用程序代码中分解出来，并且已限制在验证属性的定义中。开发人员只需通过使用适当的验证属性装饰属性来指定要应用于每个模型属性的验证规则。

当实例化操作方法参数时，验证规则会自动进行检查。然后，错误和模型中的路径（发生错误的位置）将记录在包含在`ModelState`控制器属性中的字典中。开发人员有责任通过检查`ModelState.IsValid`来验证是否存在错误，如果存在错误，则开发人员必须将相同的 ViewModel 返回到相同的视图中，以便用户可以纠正所有错误。

错误消息会自动显示在视图中，开发人员无需采取任何操作。开发人员只需要做以下事情：

+   在每个输入字段旁边添加带有`asp-validation-for`属性的`span`或`div`，该属性将自动填充可能的错误。

+   添加带有`asp-validation-summary`属性的`div`，该属性将自动填充验证错误摘要。有关更多详细信息，请参阅*使用 Razor 标签助手*部分。

只需通过调用`_ValidationScriptsPartial.cshtml`视图，并使用`partial`标签添加一些 JavaScript 引用，即可在客户端启用相同的验证规则，以便在表单提交到服务器之前向用户显示错误。一些预定义的验证属性包含在`System.ComponentModel.DataAnnotations`和`Microsoft.AspNetCore.Mvc`命名空间中，包括以下属性：

+   `Required`属性要求用户为其装饰的属性指定一个值。隐式的`Required`属性会自动应用于所有非空属性，例如所有浮点数、整数和小数，因为它们不能有`null`值。

+   `Range`属性限制数字数量在一个范围内。

+   它们还包括限制字符串长度的属性。

自定义错误消息可以直接插入到属性中，或者属性可以引用包含它们的资源类型的属性。

开发人员可以通过在 C#和 JavaScript 中提供验证代码来定义其自定义属性，用于客户端验证。

基于属性的验证可以被其他验证提供程序替换，例如使用流畅接口为每种类型定义验证规则的流畅验证。只需在 MVC 选项对象中包含的集合中更改提供程序即可。这可以通过传递给`services.AddControllersWithViews`方法的操作来配置。MVC 选项可以配置如下：

```cs
services.AddControllersWithViews(o => {
    ...
    // code that modifies o properties
}); 
```

验证框架会自动检查数字和日期输入是否根据所选文化格式良好。

## ASP.NET Core 全球化

在多元文化应用程序中，页面必须根据每个用户的语言和文化偏好提供服务。通常，多元文化应用程序可以用几种语言提供其内容，并且可以处理更多语言的日期和数字格式。事实上，虽然所有支持的语言中的内容必须手动制作，但.NET Core 具有在所有文化中格式化和解析日期和数字的本机能力。

例如，Web 应用程序可能不支持所有基于英语的文化（en）的唯一内容，但可能支持所有已知的基于英语的文化的数字和日期格式（en-US、en-GB、en-CA 等）。

在.NET 线程中用于数字和日期的文化包含在`Thread.CurrentThread.CurrentCulture`属性中。因此，通过将此属性设置为`new CultureInfo("en-CA")`，数字和日期将根据加拿大文化进行格式化/解析。`Thread.CurrentThread.CurrentUICulture`则决定资源文件的文化；也就是说，它选择每个资源文件或视图的特定于文化的版本。因此，多元文化应用程序需要设置与请求线程关联的两种文化，并将多语言内容组织到依赖于语言的资源文件和/或视图中。

根据关注点分离原则，根据用户的偏好设置请求文化的整个逻辑被分解到 ASP.NET Core 管道的特定模块中。要配置此模块，首先我们设置支持的日期/数字文化，如下例所示：

```cs
var supportedCultures = new[]
{
   new CultureInfo("en-AU"),
   new CultureInfo("en-GB"),
   new CultureInfo("en"),
   new CultureInfo("es-MX"),
   new CultureInfo("es"),
   new CultureInfo("fr-CA"),
   new CultureInfo("fr"),
   new CultureInfo("it-CH"),
   new CultureInfo("it")
}; 
```

然后，我们设置了内容支持的语言。通常，选择一个不特定于任何国家的语言版本，以保持翻译数量足够小，如下所示：

```cs
var supportedUICultures = new[]
{
    new CultureInfo("en"),
    new CultureInfo("es"),
    new CultureInfo("fr"),
    new CultureInfo("it")
}; 
```

然后，我们将 culture 中间件添加到管道中，如下所示：

```cs
app.UseRequestLocalization(new RequestLocalizationOptions
{
     DefaultRequestCulture = new RequestCulture("en", "en"),
     // Formatting numbers, dates, etc.
     SupportedCultures = supportedCultures,
     // UI strings that we have localized.
     SupportedUICultures = supportedUICultures,
     FallBackToParentCultures = true,
     FallBackToParentUICultures = true
}); 
```

如果用户请求的文化在`supportedCultures`或`supportedUICultures`中列出的文化中明确找到，则使用它而不进行修改。否则，由于`FallBackToParentCultures`和`FallBackToParentUICultures`为`true`，将尝试父文化；例如，如果所需的`fr-FR`文化在列出的文化中找不到，那么框架将搜索其通用版本`fr`。如果此尝试也失败，则框架将使用`DefaultRequestCulture`中指定的文化。

默认情况下，`culture`中间件使用三个提供程序搜索当前用户选择的文化，按照以下顺序尝试：

1.  中间件查找`culture`和`ui-culture`查询字符串参数。

1.  如果前面的步骤失败，中间件将查找名为`.AspNetCore.Culture`的 cookie，其值预期如本例所示：`c=en-US|uic=en`。

1.  如果前两个步骤都失败，中间件将查找浏览器发送的`Accept-Language`请求头，该请求头可以在浏览器设置中更改，并且最初设置为操作系统的区域设置。

使用上述策略，用户第一次请求应用程序页面时，会采用浏览器的区域设置（*步骤 3*中列出的提供程序）。然后，如果用户点击带有正确查询字符串参数的语言更改链接，提供程序 1 会选择新的区域设置。通常，一旦点击了语言链接，服务器还会通过提供程序 2 生成一个语言 cookie 来记住用户的选择。

提供内容本地化的最简单方法是为每种语言提供不同的视图。因此，如果我们想要为不同的语言本地化`Home.cshtml`视图，我们必须提供名为`Home.en.cshtml`、`Home.es.cshtml`等的视图。如果没有找到特定于`ui-culture`线程的视图，则选择未本地化的`Home.cshtml`版本的视图。

视图本地化必须通过调用`AddViewLocalization`方法来启用，如下所示：

```cs
services.AddControllersWithViews()
    .AddViewLocalization(LanguageViewLocationExpanderFormat.Suffix) 
```

另一个选项是将简单的字符串或 HTML 片段存储在针对所有支持的语言的特定资源文件中。必须通过在配置服务部分调用`AddLocalization`方法来启用资源文件的使用，如下所示：

```cs
services.AddLocalization(options => 
    options.ResourcesPath = "Resources"); 
```

`ResourcesPath`是将放置所有资源文件的根文件夹。如果未指定，将假定为空字符串，并且资源文件将放置在 Web 应用程序根目录中。例如，特定视图（例如`/Views/Home/Index.cshtml`视图）的资源文件必须具有以下路径：

```cs
<ResourcesPath >/Views/Home/Index.<culture name>.resx 
```

因此，如果`ResourcesPath`为空，则资源必须具有`/Views/Home/Index.<culture name>.resx`路径；也就是说，它们必须放在与视图相同的文件夹中。

一旦为与视图关联的所有资源文件添加了键值对，就可以向视图添加本地化的 HTML 片段，如下所示：

+   使用`@inject IViewLocalizer Localizer`将`IViewLocalizer`注入视图。

+   在需要的地方，将视图中的文本替换为对`Localizer`字典的访问，例如`Localizer["myKey"]`，其中`"myKey"`是资源文件中使用的键。

以下代码显示了`IViewLocalizer`字典的示例：

```cs
@{
    ViewData["Title"] = Localizer["HomePageTitle"];
}
<h2>@ViewData["MyTitle"]</h2> 
```

如果本地化失败，因为在资源文件中找不到键，则返回键本身。如果启用了数据注释本地化，数据注释中使用的字符串（例如验证属性）将作为资源文件中的键使用，如下所示：

```cs
 services.AddControllersWithViews()
    .AddViewLocalization(LanguageViewLocationExpanderFormat.Suffix)
    .AddDataAnnotationsLocalization(); 
```

应用于名称为`MyWebApplication.ViewModels.Account.RegisterViewModel`的类的数据注释的资源文件必须具有以下路径：

```cs
<ResourcesPath >/ViewModels/Account/RegisterViewModel.<culture name>.resx 
```

值得指出的是，与`.dll`应用程序名称对应的命名空间的第一个段将替换为`ResourcesPath`。如果`ResourcesPath`为空，并且您使用 Visual Studio 创建的默认命名空间，则资源文件必须放在包含与其关联的类的相同文件夹中。

可以通过将每组资源文件与类型（例如`MyType`）关联，然后注入`IHtmlLocalizer<MyType>`用于 HTML 片段或`IStringLocalizer<MyType>`用于需要进行 HTML 编码的字符串，来在控制器或其他地方本地化字符串和 HTML 片段。

它们的使用与`IViewLocalizer`的使用相同。与数据注释的情况一样，与`MyType`相关的资源文件的路径是计算的。如果您想要为整个应用程序使用一组唯一的资源文件，一个常见选择是使用`Startup`类作为参考类型（`IStringLocalizer<Startup>`和`IHtmlLocalizer<Startup>`）。另一个常见选择是创建各种空类，以用作各种资源文件组的参考类型。

现在我们已经学会了如何在 ASP.NET Core 项目中管理全球化，在下一小节中，我们将描述 ASP.NET Core MVC 使用的更重要的模式，以强制*关注点分离*：MVC 模式本身。

## MVC 模式

MVC 是用于实现 Web 应用程序的演示层的模式。基本思想是在演示层的逻辑和图形之间应用*关注点分离*。逻辑由控制器处理，而图形被分解为视图。控制器和视图通过模型进行通信，通常称为 ViewModel，以区别于业务和数据层的模型。

然而，演示层的逻辑是什么？在*第一章*，*理解软件架构的重要性*中，我们看到软件需求可以用用例来记录，描述用户和系统之间的交互。

粗略地说，演示层的逻辑包括管理用例；因此，粗略地说，用例被映射到控制器，每个用例的单个操作被映射到这些控制器的操作方法。因此，控制器负责管理与用户的交互协议，并依赖业务层在每个操作期间涉及的任何业务处理。

每个操作方法从用户那里接收数据，执行一些业务处理，并根据此处理的结果决定向用户显示什么，并将其编码为 ViewModel。视图接收描述向用户显示什么以及决定要使用的图形的 ViewModel，并决定要使用的 HTML。

将逻辑和图形分离成两个不同的组件有什么优势？主要优势列在这里：

+   图形的更改不会影响其余的代码，因此您可以尝试各种图形选项，以优化与用户的交互，而不会危及其余代码的可靠性。

+   应用程序可以通过实例化控制器并传递参数来进行测试，而无需使用在浏览器页面上操作的测试工具。这样，测试更容易实现。此外，它们不依赖于图形的实现方式，因此不需要在图形更改时更新。

+   将工作分配给实现控制器的开发人员和实现视图的图形设计师更容易。通常，图形设计师在 Razor 方面有困难，因此他们可能只提供一个示例 HTML 页面，由开发人员将其转换为操作实际数据的 Razor 视图。

现在，让我们看看如何在 ASP.NET Core MVC 中创建 Web 应用程序。

# 用例 - 在 ASP.NET Core MVC 中实现 Web 应用程序

在本节中，作为 ASP.NET Core 应用程序的示例，我们将实现用于管理`WWTravelClub`书中目的地和套餐的管理面板。该应用程序将使用*第十二章*中描述的**领域驱动设计**（**DDD**）方法进行实现，因此对该章的充分理解是阅读本节的基本先决条件。接下来的小节描述了整体应用程序规范和组织，以及各种应用程序部分。

## 定义应用程序规范

目的地和套餐在*第八章*中描述，*在 C#中与数据交互 - Entity Framework Core*。在这里，我们将使用完全相同的数据模型，对其进行必要的修改以适应 DDD 方法。管理面板必须允许套餐、目的地列表，并对其进行 CRUD 操作。为了简化应用程序，这两个列表将非常简单：应用程序将显示所有按名称排序的目的地，而所有套餐将按照其有效日期开始排序。

此外，我们假设以下事项：

+   向编辑窗口添加：

+   价格修改和套餐删除立即用于更新用户的购物车。因此，管理应用程序必须发送关于价格变化和套餐删除的异步通信。我们不会在这里实现整个通信逻辑，但我们将所有这类事件添加到事件表中，该表应作为输入发送给负责将这些事件发送到所有相关微服务的并行线程。

在这里，我们将为套餐管理提供完整的代码；目的地管理的大部分代码留作练习。完整的代码可在与本书相关的 GitHub 存储库的`ch15`文件夹中找到。在本节的其余部分，我们将描述应用程序的整体组织并讨论一些相关的代码示例。

## 定义应用程序架构

应用程序根据*第十二章*中描述的指南进行组织，*理解软件解决方案中的不同领域*，同时考虑 DDD 方法并使用 SOLID 原则来映射您的领域部分。也就是说，应用程序分为三个层，每个层都作为不同的项目实现：

+   有一个数据层，其中包含存储库的实现和描述数据库实体的类。这是一个.NET Core 库项目。但是，由于它需要一些接口，如`IServiceCollection`，这些接口在`Microsoft.NET.Sdk.web`中定义，因此我们必须添加对.NET Core SDK 和 ASP.NET Core SDK 的引用。可以按照以下步骤完成：

1.  在解决方案资源管理器中右键单击项目图标，然后选择**编辑项目文件**。

1.  显示目的地和套餐给用户的应用程序与管理面板使用相同的数据库。由于只有管理面板应用程序需要修改数据，因此将只有一个写入数据库副本和多个只读副本。

```cs
 <ItemGroup>
          <FrameworkReference Include="Microsoft.AspNetCore.App" />
     </ItemGroup> 
```

+   还有一个域层，其中包含存储库规范；即描述存储库实现和 DDD 聚合的接口。在我们的实现中，我们决定通过在接口后隐藏根数据实体的禁止操作/属性来实现聚合。因此，例如，`Package`数据层类，它是一个聚合根，在域层中有一个相应的`IPackage`接口，它隐藏了`Package`实体的所有属性设置器。域层还包含所有领域事件的定义，而相应的事件处理程序在应用程序层中定义。

+   最后，还有应用程序层 - 即 ASP.NET Core MVC 应用程序 - 在这里我们定义 DDD 查询、命令、命令处理程序和事件处理程序。控制器填充查询对象并执行它们以获取它们可以传递给视图的 ViewModels。它们通过填充命令对象并执行其关联的命令处理程序来更新存储。反过来，命令处理程序使用来自域层的`IRepository`接口和`IUnitOfWork`来管理和协调事务。

应用程序使用查询命令分离模式；因此，它使用命令对象来修改存储和查询对象来查询它。

查询的使用和实现都很简单：控制器填充它们的参数，然后调用它们的执行方法。反过来，查询对象有直接的 LINQ 实现，直接将结果投影到控制器视图中使用的 ViewModels 上，使用`Select` LINQ 方法。您也可以决定将 LINQ 实现隐藏在用于存储更新操作的相同存储库类后面，但这将使得简单查询的定义和修改变得非常耗时。

无论如何，将查询对象隐藏在接口后面是一个很好的做法，这样当你测试控制器时，它们的实现可以被假实现所替换。

然而，在执行命令涉及的对象和调用链更加复杂。这是因为它需要构建和修改聚合，以及定义聚合之间以及聚合与其他应用程序之间的交互，通过域事件来提供。

以下图表是存储更新操作的执行方式的草图。圆圈是在各个层之间交换的数据，而矩形是处理它们的过程。此外，虚线箭头连接接口和实现它们的类型：

![](img/B16756_15_06.png)

图 15.6：命令执行的图表

以下是通过*图 15.6*的操作流程的步骤列表：

1.  控制器的操作方法接收一个或多个 ViewModel，并进行验证。

1.  一个或多个包含要应用的更改的 ViewModel 被隐藏在域层中定义的接口（`IMyUpdate`）后面。它们用于填充命令对象的属性。这些接口必须在域层中定义，因为它们将被用作在那里定义的存储库方法的参数。

1.  在控制器操作方法中通过 DI 检索与之前命令匹配的命令处理程序（通过我们在*定义控制器和 ViewModels*子部分中描述的`[FromServices]`参数属性）。然后执行处理程序。在执行过程中，处理程序与各种存储库接口方法以及它们返回的聚合进行交互。

1.  在创建*步骤 3*中讨论的命令处理程序时，ASP.NET Core DI 引擎会自动注入其构造函数中声明的所有参数。特别是，它会注入所有`IRepository`实现，以执行所有命令处理程序事务所需的操作。命令处理程序通过调用其构造函数中接收到的这些`IRepository`实现的方法来执行其工作，以构建聚合并修改构建的聚合。处理程序使用每个`IRepository`中包含的`IUnitOfWork`接口，以及数据层返回的并发异常，来组织它们的操作作为事务。值得指出的是，每个聚合都有自己的`IRepository`，更新每个聚合的整个逻辑都是在聚合本身中定义的，而不是在其关联的`IRepository`中，以保持代码更加模块化。

1.  在数据层的幕后，`IRepository`实现使用 Entity Framework 来执行它们的工作。聚合由在域层中定义的接口隐藏的根数据实体来实现，而处理事务并将更改传递给数据库的`IUnitOfWork`方法则使用`DbContext`方法来实现。换句话说，`IUnitOfWork`是用应用的`DbContext`来实现的。

1.  在每个聚合过程中生成域事件，并通过调用它们的`AddDomainEvent`方法将它们添加到聚合中。然而，它们不会立即触发。通常情况下，它们会在所有聚合处理结束之前触发，并在更改传递给数据库之前触发；然而，这并不是一个普遍的规则。

1.  应用程序通过抛出异常来处理错误。一个更有效的方法是在依赖引擎中定义一个请求范围的对象，每个应用程序子部分都可以将其错误作为领域事件添加。然而，虽然这种方法更有效，但它增加了代码和应用程序开发时间的复杂性。

Visual Studio 解决方案由三个项目组成：

+   有一个包含领域层的项目称为`PackagesManagementDomain`，这是一个.NET Standard 2.0 库。

+   有一个包含整个数据层的项目称为`PackagesManagementDB`，这是一个.NET 5.0 库。

+   最后，有一个名为`PackagesManagement`的 ASP.NET Core MVC 5.0 项目，其中包含应用程序和表示层。在定义此项目时，选择**无身份验证**；否则，用户数据库将直接添加到 ASP.NET Core MVC 项目而不是数据库层。我们将在数据层手动添加用户数据库。

让我们首先创建`PackagesManagement` ASP.NET Core MVC 项目，以便整个解决方案与 ASP.NET Core MVC 项目具有相同的名称。然后，我们将另外两个库项目添加到同一个解决方案中。

最后，让 ASP.NET Core MVC 项目引用这两个项目，而`PackagesManagementDB`引用`PackagesManagementDomain`。我们建议您定义自己的项目，然后在阅读本节时将本书的 GitHub 存储库中的代码复制到这些项目中。

下一小节描述了`PackagesManagementDomain`数据层项目的代码。

### 定义领域层

一旦`PackagesManagementDomain`标准 2.0 库项目已添加到解决方案中，我们将在项目根目录添加一个`Tools`文件夹。然后，我们将放置与第十二章相关的所有包含在代码中的`DomainLayer`工具。由于此文件夹中包含的代码使用数据注释并定义了 DI 扩展方法，因此我们还必须添加对`System.ComponentModel.Annotations`和`Microsoft.Extensions.DependencyInjection` NuGet 包的引用。

然后，我们需要一个包含所有聚合定义的`Aggregates`文件夹（请记住，我们将聚合实现为接口）；即`IDestination`，`IPackage`和`IPackageEvent`。在这里，`IPackageEvent`是与我们将事件放置到其他应用程序中传播的表相关联的聚合。

例如，让我们分析`IPackage`：

```cs
public interface IPackage : IEntity<int>
{
    void FullUpdate(IPackageFullEditDTO o);
    string Name { get; set; }
    string Description { get;}
    decimal Price { get; set; }
    int DurationInDays { get; }
    DateTime? StartValidityDate { get;}
    DateTime? EndValidityDate { get; }
    int DestinationId { get; }

} 
```

它包含了我们在*第八章*中看到的`Package`实体的相同属性，*在 C#中与数据交互 - Entity Framework Core*。唯一的区别是以下内容：

+   它继承自`IEntity<int>`，提供了所有聚合的基本功能。

+   它没有`Id`属性，因为它是从`IEntity<int>`继承的。

+   所有属性都是只读的，并且它具有`FullUpdate`方法，因为所有聚合只能通过用户域中定义的更新操作进行修改（在我们的情况下，`FullUpdate`方法）。

现在，让我们也添加一个`DTOs`文件夹。在这里，我们放置所有用于将更新传递给聚合的接口。这些接口由应用程序层的 ViewModels 实现，用于定义这些更新。在我们的情况下，它包含`IPackageFullEditDTO`，我们可以使用它来更新现有的包裹。如果您想要添加管理目的地的逻辑，您必须为`IDestination`聚合定义一个类似的接口。

一个`IRepository`文件夹包含所有存储库规范；即`IDestinationRepository`，`IPackageRepository`和`IPackageEventRepository`。在这里，`IPackageEventRepository`是与`IPackageEvent`聚合相关联的存储库。例如，让我们看一下`IPackageRepository`存储库：

```cs
public interface IPackageRepository: 
        IRepository<IPackage>
{
    Task<IPackage> Get(int id);
    IPackage New();
    Task<IPackage> Delete(int id);
} 
```

存储库始终只包含少量方法，因为所有业务逻辑应表示为聚合方法 - 在我们的例子中，只有创建新包、检索现有包和删除现有包的方法。修改现有包的逻辑包含在`IPackage`的`FullUpdate`方法中。

最后，与所有领域层项目一样，`PackagesManagementDomain`包含一个包含所有领域事件定义的事件文件夹。在我们的例子中，文件夹的名称为`Events`，包含了 package-deleted 事件和 price-changed 事件：

```cs
public class PackageDeleteEvent: IEventNotification
{
    public PackageDeleteEvent(int id, long oldVersion)
    {
        PackageId = id;
        OldVersion = oldVersion;
    }
    public int PackageId { get; }
    public long OldVersion { get; }

}
public class PackagePriceChangedEvent: IEventNotification
{
    public PackagePriceChangedEvent(int id, decimal price, 
        long oldVersion, long newVersion)
    {
            PackageId = id;
            NewPrice = price;
            OldVersion = oldVersion;
            NewVersion = newVersion;
     }
    public int PackageId { get; }
    public decimal NewPrice { get; }
    public long OldVersion { get; }
    public long NewVersion { get; }
} 
```

当一个聚合将所有更改发送到另一个应用程序时，它必须具有一个版本属性。接收更改的应用程序使用此版本属性以正确的顺序应用所有更改。显式版本号是必需的，因为更改是异步发送的，因此它们接收的顺序可能与它们发送的顺序不同。为此，用于在应用程序外部发布更改的事件具有`OldVersion`（更改之前的版本）和`NewVersion`（更改之后的版本）属性。与删除事件相关的事件没有`NewVersion`，因为在被删除后，实体无法存储任何版本。

下一小节解释了如何在数据层中实现领域层中定义的所有接口。

### 定义数据层

数据层项目包含对`Microsoft.AspNetCore.Identity.EntityFrameworkCore`和`Microsoft.EntityFrameworkCore.SqlServer` NuGet 包的引用，因为我们使用 Entity Framework Core 与 SQL Server。它引用了`Microsoft.EntityFrameworkCore.Tools`和`Microsoft.EntityFrameworkCore.Design`，这些是生成数据库迁移所需的，如*第八章*中*在 C#中与数据交互 - Entity Framework Core*的*Entity Framework Core 迁移*部分所述。

我们有一个包含所有数据库实体的`Models`文件夹。它们与*第八章*中*在 C#中与数据交互 - Entity Framework Core*中的实体类似。唯一的区别如下：

+   它们继承自`Entity<T>`，其中包含所有聚合的基本特性。请注意，只有聚合根需要继承自`Entity<T>`；所有其他实体必须按*第八章*中所述进行定义，*在 C#中与数据交互 - Entity Framework Core*。在我们的例子中，所有实体都是聚合根。

+   它们没有`Id`，因为它是从`Entity<T>`继承的。

+   其中一些具有使用`[ConcurrencyCheck]`属性修饰的`EntityVersion`属性。它包含发送所有实体更改到其他应用程序所需的实体版本。`ConcurrencyCheck`属性用于防止更新实体版本时的并发错误。这可以防止由事务所暗示的性能惩罚。

更具体地说，当保存实体更改时，如果带有`ConcurrencyCheck`属性的字段的值与在内存中加载实体时读取的值不同，则会抛出并发异常，以通知调用方法在我们尝试保存其更改之前，其他人修改了该值。这样，调用方法可以重复整个操作，希望这次在其执行期间没有人将相同的实体写入数据库。

值得分析`Package`实体：

```cs
public class Package: Entity<int>, IPackage
{
    public void FullUpdate(IPackageFullEditDTO o)
    {
        if (IsTransient())
        {
            Id = o.Id;
            DestinationId = o.DestinationId;
        }
        else
        {
            if (o.Price != this.Price)
                this.AddDomainEvent(new PackagePriceChangedEvent(
                        Id, o.Price, EntityVersion, EntityVersion+1));
        }
        Name = o.Name;
        Description = o.Description;
        Price = o.Price;
        DurationInDays = o.DurationInDays;
        StartValidityDate = o.StartValidityDate;
        EndValidityDate = o.EndValidityDate;
    }
    [MaxLength(128), Required]
    public string Name { get; set; }
    [MaxLength(128)]
    public string Description { get; set; }
    public decimal Price { get; set; }
    public int DurationInDays { get; set; }
    public DateTime? StartValidityDate { get; set; }
    public DateTime? EndValidityDate { get; set; }
    public Destination MyDestination { get; set; }
    [ConcurrencyCheck]
    public long EntityVersion{ get; set; }
    public int DestinationId { get; set; }
} 
```

`FullUpdate`方法是更新`IPackage`聚合的唯一方法，当价格更改时，将`PackagePriceChangedEvent`添加到实体事件列表中。

`MainDBContext.cs`文件包含数据层数据库上下文定义。它不是从`DBContext`继承，而是从以下预定义的上下文类继承：

```cs
IdentityDbContext<IdentityUser<int>, IdentityRole<int>, int> 
```

这个上下文定义了身份验证所需的用户表。在我们的情况下，我们选择了`IdentityUser<T>`标准和`IdentityRole<S>`用于用户和角色，并分别使用整数作为`T`和`S`实体键。然而，我们也可以使用从`IdentityUser`和`IdentityRole`继承的类，然后添加更多属性。

在`OnModelCreating`方法中，我们必须调用`base.OnModelCreating(builder)`以应用在`IdentityDbContext`中定义的配置。

`MainDBContext`实现了`IUnitOfWork`。以下代码显示了开始、回滚和提交事务的所有方法的实现：

```cs
public async Task StartAsync()
{
    await Database.BeginTransactionAsync();
}
public Task CommitAsync()
{
    Database.CommitTransaction();
    return Task.CompletedTask;
}
public Task RollbackAsync()
{
    Database.RollbackTransaction();
    return Task.CompletedTask;
} 
```

然而，在分布式环境中，它们很少被命令类使用。这是因为重试相同的操作直到不返回并发异常通常比事务保证更好的性能。

值得分析的是将所有应用于`DbContext`的更改传递到数据库的方法的实现：

```cs
public async Task<bool> SaveEntitiesAsync()
{ 
    try
    {
        return await SaveChangesAsync() > 0;
    }
    catch (DbUpdateConcurrencyException ex)
    {
        foreach (var entry in ex.Entries)
        {
            entry.State = EntityState.Detached; 

        }
        throw;
    }
} 
```

前面的实现只是调用`SaveChangesAsync DbContext`上下文方法，该方法将所有更改保存到数据库，然后拦截所有并发异常，并从上下文中分离涉及并发错误的所有实体。这样，下次命令重试整个失败的操作时，它们的更新版本将从数据库重新加载。

`Repositories`文件夹包含所有存储库实现。值得分析的是`IPackageRepository.Delete`方法的实现：

```cs
public async Task<IPackage> Delete(int id)
{
    var model = await Get(id);
    if (model is not Package package) return null;
    context.Packages.Remove(package);
    model.AddDomainEvent(
        new PackageDeleteEvent(
            model.Id, package.EntityVersion));
    return model;
} 
```

它从数据库中读取实体，并正式将其从`Packages`数据集中移除。这将在将更改保存到数据库时强制删除实体。此外，它将`PackageDeleteEvent`添加到事件的聚合列表中。

`Extensions`文件夹包含`DBExtensions`静态类，该类定义了两个扩展方法，分别添加到应用程序 DI 引擎和 ASP.NET Core 管道中。一旦添加到管道中，这两种方法将连接数据库层和应用程序层。

`AddDbLayer`的`IServiceCollection`扩展接受数据库连接字符串和包含所有迁移的`.dll`文件的名称作为其输入参数。然后，它执行以下操作：

```cs
services.AddDbContext<MainDbContext>(options =>
                options.UseSqlServer(connectionString, 
                b => b.MigrationsAssembly(migrationAssembly))); 
```

也就是说，它将数据库上下文添加到 DI 引擎并定义其选项；即使用 SQL Server、数据库连接字符串和包含所有迁移的`.dll`文件的名称。

然后，它执行以下操作：

```cs
services.AddIdentity<IdentityUser<int>, IdentityRole<int>>()
                .AddEntityFrameworkStores<MainDbContext>()
                .AddDefaultTokenProviders(); 
```

也就是说，它添加和配置了处理基于数据库的身份验证所需的所有类型。特别是，它添加了应用程序层可以使用的`UserManager`和`RoleManager`类型来管理用户和角色。`AddDefaultTokenProviders`添加了在用户登录时使用数据库中包含的数据创建身份验证令牌的提供程序。

最后，它通过调用在我们添加到域层项目中的 DDD 工具中定义的`AddAllRepositories`方法，发现并添加到 DI 引擎中所有存储库实现。

`UseDBLayer`扩展方法通过调用`context.Database.Migrate()`确保迁移应用到数据库，然后用一些初始对象填充数据库。在我们的情况下，它使用`RoleManager`和`UserManager`分别创建管理角色和初始管理员。然后，它创建一些示例目的地和包裹。

`context.Database.Migrate()`对于快速设置和更新暂存和测试环境非常有用。然而，在生产环境部署时，应该使用迁移工具从迁移中生成一个 SQL 脚本。然后，这个脚本应该在数据库维护人员应用之前进行检查。

创建迁移，我们必须将上述扩展方法添加到 ASP.NET Core MVC 的`Startup.cs`文件中，如下所示：

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...
    services.AddRazorPages();
    services.AddDbLayer(
        Configuration.GetConnectionString("DefaultConnection"),
        "PackagesManagementDB");
___________________________
public void Configure(IApplicationBuilder app, 
    IWebHostEnvironment env)
    ...
    app.UseAuthentication();
    app.UseAuthorization();
    ...
} 
```

请确保授权和认证模块都已添加到 ASP.NET Core 管道中；否则，认证/授权引擎将无法工作。

然后，我们必须像这样将连接字符串添加到`appsettings.json`文件中：

```cs
{
   "ConnectionStrings": {
        "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=package-management;Trusted_Connection=True;MultipleActiveResultSets=true"

    },
    ...
} 
```

最后，让我们将`Microsoft.EntityFrameworkCore.Design`添加到 ASP.NET Core 项目中。

此时，让我们打开 Visual Studio Package Manager 控制台，选择`PackageManagementDB`作为默认项目，然后执行以下命令：

```cs
Add-Migration Initial -Project PackageManagementDB 
```

前面的命令将生成第一个迁移。我们可以使用`Update-Database`命令将其应用到数据库。请注意，如果您从 GitHub 复制项目，您不需要生成迁移，因为它们已经被创建，但您仍然需要更新数据库。

下一小节描述了应用程序层。

### 定义应用程序层

作为第一步，为了简单起见，让我们通过将以下代码添加到 ASP.NET Core 管道中，将应用程序的文化设置为`en-US`：

```cs
app.UseAuthorization();
// Code to add: configure the Localization middleware
var ci = new CultureInfo("en-US"); 
app.UseRequestLocalization(new RequestLocalizationOptions
{
    DefaultRequestCulture = new RequestCulture(ci),
    SupportedCultures = new List<CultureInfo>
    {
        ci,
    },
     SupportedUICultures = new List<CultureInfo>
    {
        ci,
    }
}); 
```

然后，让我们创建一个`Tools`文件夹，并将`ApplicationLayer`代码放在那里，您可以在与本书相关的 GitHub 存储库的`ch12`代码中找到。有了这些工具，我们可以添加代码，自动发现并添加所有查询、命令处理程序和事件处理程序到 DI 引擎中，如下所示：

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...
    ...
    services.AddAllQueries(this.GetType().Assembly);
    services.AddAllCommandHandlers(this.GetType().Assembly);
    services.AddAllEventHandlers(this.GetType().Assembly);
} 
```

然后，我们必须添加一个`Queries`文件夹来放置所有查询及其关联的接口。例如，让我们看一下列出所有包的查询：

```cs
public class PackagesListQuery:IPackagesListQuery
{
    private readonly MainDbContext ctx;
    public PackagesListQuery(MainDbContext ctx)
    {
        this.ctx = ctx;
    }
    public async Task<IEnumerable<PackageInfosViewModel>> GetAllPackages()
    {
        return await ctx.Packages.Select(m => new PackageInfosViewModel
        {
            StartValidityDate = m.StartValidityDate,
            EndValidityDate = m.EndValidityDate,
            Name = m.Name,
            DurationInDays = m.DurationInDays,
            Id = m.Id,
            Price = m.Price,
            DestinationName = m.MyDestination.Name,
            DestinationId = m.DestinationId
        })
            .OrderByDescending(m=> m.EndValidityDate)
            .ToListAsync();
    }
} 
```

查询对象会自动注入到应用程序 DB 上下文中。`GetAllPackages`方法使用 LINQ 将所有所需信息投影到`PackageInfosViewModel`中，并按`EndValidityDate`属性的降序对所有结果进行排序。

`PackageInfosViewModel`与所有其他 ViewModel 一起放在`Models`文件夹中。将 ViewModels 组织到文件夹中，为每个控制器定义一个不同的文件夹是一个很好的做法。值得分析的是用于编辑包的 ViewModel：

```cs
public class PackageFullEditViewModel: IPackageFullEditDTO
    {
        public PackageFullEditViewModel() { }
        public PackageFullEditViewModel(IPackage o)
        {
            Id = o.Id;
            DestinationId = o.DestinationId;
            Name = o.Name;
            Description = o.Description;
            Price = o.Price;
            DurationInDays = o.DurationInDays;
            StartValidityDate = o.StartValidityDate;
            EndValidityDate = o.EndValidityDate;
        }
        ...
        ... 
```

它有一个接受`IPackage`聚合的构造函数。这样，包数据就被复制到用于填充编辑视图的 ViewModel 中。它实现了在域层中定义的`IPackageFullEditDTO` DTO 接口。这样，它可以直接用于将`IPackage`更新发送到域层。

所有属性都包含验证属性，这些属性会被客户端和服务器端验证引擎自动使用。每个属性都包含一个`Display`属性，该属性定义了用于编辑属性的输入字段的标签。最好将字段标签放在 ViewModels 中，而不是直接放在视图中，因为这样，相同的名称会自动在使用相同 ViewModel 的所有视图中使用。以下代码块列出了所有属性：

```cs
public int Id { get; set; }
[StringLength(128, MinimumLength = 5), Required]
[Display(Name = "name")]
public string Name { get; set; }
[Display(Name = "package infos")]
[StringLength(128, MinimumLength = 10), Required]
public string Description { get; set; }
[Display(Name = "price")]
[Range(0, 100000)]
public decimal Price { get; set; }
[Display(Name = "duration in days")]
[Range(1, 90)]
public int DurationInDays { get; set; }
[Display(Name = "available from"), Required]
public DateTime? StartValidityDate { get; set; }
[Display(Name = "available to"), Required]
public DateTime? EndValidityDate { get; set; }
[Display(Name = "destination")]
public int DestinationId { get; set; } 
```

`Commands`文件夹包含所有命令。例如，让我们看一下用于修改包的命令：

```cs
public class UpdatePackageCommand: ICommand
{
    public UpdatePackageCommand(IPackageFullEditDTO updates)
    {
        Updates = updates;
    }
    public IPackageFullEditDTO Updates { get; private set; }
} 
```

它的构造函数必须使用`IPackageFullEditDTO` DTO 接口的实现来调用，而在我们的情况下，这就是我们之前描述的编辑 ViewModel。命令处理程序放在`Handlers`文件夹中。值得分析的是更新包的命令：

```cs
IPackageRepository repo;
IEventMediator mediator;
public UpdatePackageCommandHandler(IPackageRepository repo, IEventMediator mediator)
{
    this.repo = repo;
    this.mediator = mediator;
} 
```

它的构造函数已自动注入了`IPackageRepository`存储库和一个触发事件处理程序所需的`IEventMediator`实例。以下代码还显示了标准的`HandleAsync`命令处理程序方法的实现：

```cs
public async Task HandleAsync(UpdatePackageCommand command)
{
    bool done = false;
    IPackage model;
    while (!done)
    {
        try
        {
            model = await repo.Get(command.Updates.Id);
            if (model == null) return;
            model.FullUpdate(command.Updates);
            await mediator.TriggerEvents(model.DomainEvents);
            await repo.UnitOfWork.SaveEntitiesAsync();
            done = true;
        }
        catch (DbUpdateConcurrencyException)
        {
          // add some logging here
        }
    }
} 
```

直到不返回并发异常为止，重复命令操作。`HandleAsync`使用存储库获取要修改的实体的实例。如果未找到实体（已删除），则命令将停止执行。否则，所有更改都将传递给检索到的聚合。更新后，立即触发聚合中包含的所有事件。特别是，如果价格已更改，则执行与价格更改相关的事件处理程序。在`Package`实体的`EntityVersion`属性上声明的`[ConcurrencyCheck]`属性确保包版本正确更新（通过将其先前版本号增加 1），以及价格更改事件传递正确的版本号。

此外，事件处理程序放置在`Handlers`文件夹中。例如，让我们看一下价格更改事件处理程序：

```cs
public class PackagePriceChangedEventHandler :
    IEventHandler<PackagePriceChangedEvent>
{
    private readonly IPackageEventRepository repo;
    public PackagePriceChangedEventHandler(IPackageEventRepository repo)
    {
        this.repo = repo;
    }
    public Task HandleAsync(PackagePriceChangedEvent ev)
    {
        repo.New(PackageEventType.CostChanged, ev.PackageId, 
            ev.OldVersion, ev.NewVersion, ev.NewPrice);
      return Task.CompletedTask;
    }
} 
```

构造函数已自动注入了处理数据库表和发送到其他应用程序的所有事件的`IPackageEventRepository`存储库。`HandleAsync`实现只是调用存储库方法，向该表添加新记录。

`IPackageEventRepository`处理表中的所有记录，可以通过在 DI 引擎中定义并行任务的方式检索并发送到所有感兴趣的微服务，例如`services.AddHostedService<MyHostedService>();`，详细信息请参见*第五章*的*将微服务架构应用于企业应用程序*的*使用通用主机*小节。但是，本章关联的 GitHub 代码中未实现此并行任务。

下一小节描述了控制器和视图的设计方式。

## 控制器和视图

我们需要向 Visual Studio 自动脚手架生成的一个控制器添加另外两个控制器；即`AccountController`，负责用户登录/注销和注册，以及`ManagePackageController`，处理所有与包相关的操作。只需右键单击`Controllers`文件夹，然后选择**添加** | **控制器**。然后，选择控制器名称并选择空的 MVC 控制器，以避免 Visual Studio 生成不需要的代码。

为简单起见，`AccountController`只有登录和注销方法，因此您只能使用初始管理员用户登录。但是，您可以添加更多使用`UserManager`类定义、更新和删除用户的动作方法。`UserManager`类可以通过 DI 提供，如下所示：

```cs
private readonly UserManager<IdentityUser<int>> _userManager;
private readonly SignInManager<IdentityUser<int>> _signInManager;
public AccountController(
    UserManager<IdentityUser<int>> userManager,
    SignInManager<IdentityUser<int>> signInManager)
{
    _userManager = userManager;
    _signInManager = signInManager;
} 
```

`SignInManager`负责登录/注销操作。`Logout`动作方法非常简单，如下所示：

```cs
[HttpPost]
public async Task<IActionResult> Logout()
{
    await _signInManager.SignOutAsync();
    return RedirectToAction(nameof(HomeController.Index), "Home");
} 
```

它只调用`signInManager.SignOutAsync`方法，然后将浏览器重定向到主页。为了避免通过单击链接调用它，它使用`HttpPost`进行修饰，因此只能通过表单提交调用。

另一方面，登录需要两个动作方法。第一个是通过`Get`调用的，显示登录表单，用户必须在其中放置用户名和密码。如下所示：

```cs
[HttpGet]
public async Task<IActionResult> Login(string returnUrl = null)
{
    // Clear the existing external cookie 
    //to ensure a clean login process
    await HttpContext
         .SignOutAsync(IdentityConstants.ExternalScheme);
    ViewData["ReturnUrl"] = returnUrl;
    return View();
} 
```

当浏览器被授权模块自动重定向到登录页面时，它将`returnUrl`作为参数接收。这发生在未登录用户尝试访问受保护页面时。`returnUrl`存储在传递给登录视图的`ViewState`字典中。登录视图中的表单在提交时将其与用户名和密码一起传递回控制器，如下所示：

```cs
<form asp-route-returnurl="@ViewData["ReturnUrl"]" method="post">
...
</form> 
```

表单提交由具有相同`Login`名称的动作方法拦截，但使用`[HttpPost]`属性进行修饰，如下所示：

```cs
[ValidateAntiForgeryToken]
public async Task<IActionResult> Login(
    LoginViewModel model,
    string returnUrl = null)
        {
            ... 
```

前面的方法接收登录视图使用的`Login`模型，以及`returnUrl`查询字符串参数。`ValidateAntiForgeryToken`属性验证 MVC 表单自动添加的令牌（称为防伪令牌）。然后将其添加到隐藏字段中，以防止跨站点攻击。

作为第一步，如果用户已经登录，操作方法会注销用户：

```cs
if (User.Identity.IsAuthenticated)
{
      await _signInManager.SignOutAsync();

} 
```

否则，它会验证是否存在验证错误，如果有，则显示填充有 ViewModel 数据的相同视图，以便用户纠正其错误：

```cs
if (ModelState.IsValid)
{
     ...
}
else
 // If we got this far, something failed, redisplay form
 return View(model); 
```

如果模型有效，将使用`_signInManager`来登录用户：

```cs
var result = await _signInManager.PasswordSignInAsync(
    model.UserName, 
    model.Password, model.RememberMe, 
    lockoutOnFailure: false); 
```

如果操作返回的结果成功，操作方法将浏览器重定向到`returnUrl`（如果不为空）；否则，将浏览器重定向到主页：

```cs
if (result.Succeeded)
{
    if (!string.IsNullOrEmpty(returnUrl))
        return LocalRedirect(returnUrl);
    else
        return RedirectToAction(nameof(HomeController.Index), "Home");
}
else
{
    ModelState.AddModelError(string.Empty, 
        "wrong username or password");
    return View(model);
} 
```

如果登录失败，它会向`ModelState`添加一个错误，并显示相同的表单，让用户再次尝试。

`ManagePackagesController`包含一个`Index`方法，以表格格式显示所有包：

```cs
[HttpGet]
public async Task<IActionResult> Index(
    [FromServices]IPackagesListQuery query)
{
    var results = await query.GetAllPackages();
    var vm = new PackagesListViewModel { Items = results };
    return View(vm);
} 
```

查询对象通过 DI 注入到操作方法中。然后，操作方法调用它，并将结果的`IEnumerable`插入到`PackagesListViewModel`实例的`Items`属性中。将`IEnumerables`包含在 ViewModels 中是一个很好的做法，而不是直接将它们传递给视图，这样如果需要，可以添加其他属性，而无需修改现有的视图代码。结果显示在 Bootstrap 4 表中，因为 Visual Studio 自动创建了 Bootstrap 4 CSS。

结果如下所示：

![](img/B16756_15_07.png)

图 15.7：应用程序包处理页面

**新包**链接（它的形状类似于**Bootstrap 4**按钮，但它是一个链接）调用`Create`操作方法的控制器，而每行中的**删除**和**编辑**链接分别调用`Delete`和`Edit`操作方法，并将它们传递给行中显示的包的 ID。以下是两行链接的实现：

```cs
@foreach(var package in Model.Items)
{
<tr>
    <td>
        <a asp-controller="ManagePackages"
            asp-action="@nameof(ManagePackagesController.Delete)"
            asp-route-id="@package.Id">
            delete
        </a>
    </td>
    <td>
        <a asp-controller="ManagePackages"
            asp-action="@nameof(ManagePackagesController.Edit)"
            asp-route-id="@package.Id">
            edit
        </a>
    </td>
    ...
    ... 
```

值得描述的是`HttpGet`和`HttpPost`的`Edit`操作方法的代码：

```cs
[HttpGet]
public async Task<IActionResult> Edit(
    int id,
    [FromServices] IPackageRepository repo)
{
    if (id == 0) return RedirectToAction(
        nameof(ManagePackagesController.Index));
    var aggregate = await repo.Get(id);
    if (aggregate == null) return RedirectToAction(
        nameof(ManagePackagesController.Index));
    var vm = new PackageFullEditViewModel(aggregate);
    return View(vm);
} 
```

`HttpGet`的`Edit`方法使用`IPackageRepository`来检索现有的包。如果找不到包，这意味着它已被其他用户删除，并且浏览器会再次重定向到列表页面，以显示更新后的包列表。否则，聚合将传递给`PackageFullEditViewModel` ViewModel，该 ViewModel 由`Edit`视图呈现。

用于呈现包的视图必须呈现带有所有可能的包目的地的`select`，因此它需要`IDestinationListQuery`查询的一个实例，该查询已实现以辅助目的地选择 HTML 逻辑。由于视图有责任决定如何使用户能够选择目的地，因此该查询直接注入到视图中。注入查询并使用它的代码如下所示：

```cs
@inject PackagesManagement.Queries.IDestinationListQuery destinationsQuery
@{
    ViewData["Title"] = "Edit/Create package";
    var allDestinations = 
        await destinationsQuery.AllDestinations();
} 
```

处理视图表单的帖子的操作方法如下所示：

```cs
[HttpPost]
public async Task<IActionResult> Edit(
    PackageFullEditViewModel vm,
    [FromServices] ICommandHandler<UpdatePackageCommand> command)
{
    if (ModelState.IsValid)
    {
        await command.HandleAsync(new UpdatePackageCommand(vm));
        return RedirectToAction(
            nameof(ManagePackagesController.Index));
    }
    else
        return View(vm);
} 
```

如果`ModelState`有效，则会创建`UpdatePackageCommand`并调用其关联的处理程序；否则，再次向用户显示视图，以便他们纠正所有错误。

必须将指向包列表页面和登录页面的新链接添加到主菜单中，该菜单位于`_Layout`视图中，如下所示：

```cs
<li class="nav-item">
    <a class="nav-link text-dark" 
        asp-controller="ManagePackages" 
            asp-action="Index">Manage packages</a>
</li>
@if (User.Identity.IsAuthenticated)
{
    <li class="nav-item">
        <a class="nav-link text-dark"
            href="javascript:document.getElementById('logoutForm').submit()">
            Logout
        </a>
    </li>
}
else
{
    <li class="nav-item">
        <a class="nav-link text-dark" 
            asp-controller="Account" asp-action="Login">Login</a>
    </li>
} 
```

`logoutForm`是一个空表单，其唯一目的是向`Logout`操作方法发送一个帖子。它已添加到正文的末尾，如下所示：

```cs
@if (User.Identity.IsAuthenticated)
{
    <form asp-area="" asp-controller="Account" 
            asp-action="Logout" method="post" 
            id="logoutForm" ></form>
} 
```

现在，应用程序已准备就绪！您可以运行它，登录并开始管理包。

# 总结

在本章中，我们详细分析了 ASP.NET Core 管道和组成 ASP.NET Core MVC 应用程序的各种模块，如身份验证/授权、选项框架和路由。然后，我们描述了控制器和视图如何将请求映射到响应 HTML。我们还分析了最新版本中引入的所有改进。

最后，我们分析了 ASP.NET Core MVC 框架中实现的所有设计模式，特别是关注了关注点分离原则的重要性以及 ASP.NET Core MVC 如何在 ASP.NET Core 管道中以及其验证和全球化模块中实现它。我们更详细地关注了演示层逻辑和图形之间关注点分离的重要性，以及 MVC 模式如何确保它。

下一章将解释如何使用新的 Blazor WebAssembly 框架将演示层实现为**单页应用程序**（**SPA**）。

# 问题

1.  您能列出 Visual Studio 在 ASP.NET Core 项目中脚手架生成的所有中间件模块吗？

1.  ASP.NET Core 管道模块默认需要继承自基类或实现某个接口吗？

1.  一个标签必须只有一个为其定义的标签助手，否则会抛出异常，这是真的吗？

1.  您还记得如何在控制器中测试是否发生了验证错误吗？

1.  在布局视图中包含被调用的主视图的指令是什么？

1.  主视图的次要部分如何在布局视图中调用？

1.  控制器如何调用视图？

1.  全球化模块默认安装了多少提供程序？

1.  ViewModels 是控制器与调用的视图通信的唯一方式吗？

# 进一步阅读

+   有关 ASP.NET MVC 框架的更多详细信息，请参阅其官方文档[`docs.microsoft.com/en-US/aspnet/core/`](https://docs.microsoft.com/en-US/aspnet/core/)

+   有关 Razor 语法的更多细节，请访问[`docs.microsoft.com/en-us/aspnet/core/razor-pages/?tabs=visual-studio`](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/?tabs=visual-studio)。

+   在本章中未讨论的创建自定义标签助手的文档可以在[`docs.microsoft.com/en-US/aspnet/core/mvc/views/tag-helpers/authoring`](https://docs.microsoft.com/en-US/aspnet/core/mvc/views/tag-helpers/authoring)找到。

+   有关创建自定义控制器属性的文档可以在[`docs.microsoft.com/en-US/aspnet/core/mvc/controllers/filters`](https://docs.microsoft.com/en-US/aspnet/core/mvc/controllers/filters)找到。

+   有关自定义验证属性的定义，请参阅本文：[`blogs.msdn.microsoft.com/mvpawardprogram/2017/01/03/asp-net-core-mvc/`](https://blogs.msdn.microsoft.com/mvpawardprogram/2017/01/03/asp-net-core-mvc/)。
