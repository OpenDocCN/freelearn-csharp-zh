# 第十五章：保护您的服务

在前面的章节中，我们看到了如何使用事件总线和弹性客户端在多个服务之间交换数据，以及如何使用 ASP.NET Core 消费消息和运行后台服务。本章是关于保护服务数据。它涵盖了 SSL、**跨源资源共享**（**CORS**）和 HTTP/2 等概念，并介绍了基于令牌的认证的实现。

更详细地说，本章涵盖了以下主题：

+   SSL 的一般概述

+   如何在 ASP.NET Core 服务中启用 CORS

+   如何启用 HTTPS 和 HTTP/2

+   基于令牌的认证是如何工作的

+   如何在 ASP.NET Core 中构建基于令牌的认证

到本章结束时，你将广泛了解 ASP.NET Core 提供的安全功能，并且能够实现基于令牌的认证在 ASP.NET Core 中。

# 安全通信概述

安全性是构建应用程序的一个关键方面。Web 服务通常向第三方客户端和公司公开信息；因此，避免数据泄露至关重要。Web 服务的安全层通常是开发过程中的一个繁琐部分，因为它难以测试和验证。

即使是发布在公司内部网络中的 Web 服务，安全性也很重要，默认情况下，外部无法访问。作为软件工程师，我们应该尽可能保证在发布 Web 服务时具有强大的安全性。了解保护 Web 服务是识别数据消费者和防止过度使用 Web 服务的必要条件。下一节将首先描述 HTTPS 以及如何在 ASP.NET Core Web 服务中使用 HTTPS 保护数据。

# 使用 HTTPS 保护数据

攻击者常用的一个做法是拦截客户端和服务器之间交换的数据。因此，加密他们之间的通信以保持数据安全至关重要。SSL 使用 SSL 证书在服务器公司和客户端之间建立信任连接。SSL 使用 *对称* 和 *非对称加密* 加密在此通信过程中使用的密钥。让我们看看客户端和服务器之间典型 SSL 握手的方案：

![图片](img/38cf9627-a94f-4c11-a896-59138269b4f8.png)

此方案显示了 SSL 握手的传统步骤：

1.  连接从发起请求的客户端开始。在开始之前，服务器向客户端发送 SSL 证书，以确保证书有效且可信。

1.  客户端接着提取并加密 SSL 证书中包含的 *公钥*。

1.  客户端将加密的密钥（私钥）发送到服务器，服务器对数据进行编码并将数据传输回客户端。

1.  数据通信开始，加密密钥用于加密和解密客户端和服务器共享的数据。

SSL 是 HTTPS 协议的基础，是传输加密数据的标准方式。以下部分描述了如何在 ASP.NET Core 中设置和强制执行 HTTPS。

# 在 ASP.NET Core 中强制执行 HTTPS

ASP.NET Core 默认启用了 HTTPS。与 HTTPS 协议相关的中间件主要是`HttpsRedirection`中间件类，它强制从 HTTP 重定向到 HTTPS。因此，可以在`Startup`类的`Startup`类中调用`UseHttpsRedirection`扩展方法来启用中间件。

让我们看看如何在运行在 docker 容器中的 ASP.NET Core 应用程序中启用和强制执行 HTTPS。第一步是生成 ASP.NET Core 应用程序在容器中使用的自签名证书。.NET Core 提供了一个全局工具，称为`dotnet-dev-certs`，可以在本地环境中创建自签名证书。我们可以通过以下 CLI 命令在我们的本地环境中安装此工具：

```cs
dotnet tool install --global dotnet-dev-certs
```

之后，可以在以下命令中创建一个新的`.pfx`格式的证书：

```cs
dotnet dev-certs https -ep <path_to_certificate>/certificate.pfx -p <certificate_password>
```

上述指令使用`-ep`选项指定导出路径，并使用`-p`密码。此外，还可以使用`--trust`选项信任证书。

需要注意的是，`dotnet-dev-certs`工具仅在 Windows 和 macOS 上工作。在 Linux 的情况下，我们应该通过使用 OpenSSL 生成证书来继续操作。以下教程（[`www.humankode.com/asp-net-core/develop-locally-with-https-self-signed-certificates-and-asp-net-core`](https://www.humankode.com/asp-net-core/develop-locally-with-https-self-signed-certificates-and-asp-net-core)）提供了有关使用 OpenSSL 创建 HTTPS 证书的更多信息。

一旦我们创建了新的证书文件，我们就可以通过调整`Catalog.API`和`Cart.API`解决方案的`docker-compose.yml`文件来继续操作：

```cs
version: "3.7"
services:
  catalog_api:
    container_name: catalog_api
    build:
      context: .
      dockerfile: containers/api/Dockerfile
 volumes:
 - ./<path_to_certificate>/:/root/.dotnet/https
    env_file:
      - containers/api/api.env
    networks:
      - my_network
    ports:
      - 5000:5000
 - 5001:5001
    depends_on:
      - catalog_db
      - catalog_esb

 ...
```

上述`docker-compose.yml`定义声明了一个`volumes`节点，用于在容器实例中将本地`./certificate/`文件夹与`/root/.dotnet/https`文件夹绑定。此外，我们还可以通过在`containers/api.env`文件中添加以下变量来继续操作：

```cs
ASPNETCORE_ENVIRONMENT=Integration
ASPNETCORE_URLS=https://*:5001
ASPNETCORE_Kestrel__Certificates__Default__Password=<certificate_password>
ASPNETCORE_Kestrel__Certificates__Default__Path=/root/.dotnet/https/certificate.pfx
```

该文件添加了两个与证书相关的环境变量：`ASPNETCORE_Kestrel__Certificates__Default__Password`提供证书密码，而`ASPNETCORE_Kestrel__Certificates__Default__Path`定义其路径。新的`docker-compose.yml`文件定义还公开了`5001`端口，并且它还向 Kestrel 运行的 URL 池中添加了`https://*:5001 URL` URL。此外，现在我们可以在`Startup`类的`Configure`方法中添加以下行来强制执行 HTTPS：

```cs
app.UseHttpsRedirection();
```

在应用 HTTPS 限制后，客户端在每次请求时都会被重定向到 Web 服务的 HTTPS 端点。

# Kestrel 上的 HTTP/2

ASP.NET Core 自 2.2.0 版本起支持 Kestrel 上的 HTTP/2，如果你使用 HTTPS，则默认启用。此外，另一个 HTTP/2 的要求是支持**应用层协议协商**（**ALPN**）协议。ALPN 协议增强了客户端和服务器之间的握手过程：客户端列出所有支持的协议，服务器将确认用于 HTTP 传输的协议。此外，这种方法还允许在客户端或服务器不支持 HTTP/2 的情况下回退到 HTTP 1.1。

作为默认配置，HTTP 1.1 和 HTTP/2 在相同的绑定上运行，但可以通过在`static void Main`方法中扩展 Kestrel 配置来自定义并创建一个专门的 HTTP/2 绑定：

```cs
using System.Net;
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Server.Kestrel.Core;

namespace Catalog.API
{
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateWebHostBuilder(args).Build().Run();
        }

        public static IWebHostBuilder CreateWebHostBuilder(string[] args)
        {
            return WebHost.CreateDefaultBuilder(args)
                .ConfigureKestrel(options =>
 {
 options.Listen(IPAddress.Any, 5002,listenOptions =>
 {
 listenOptions.Protocols = HttpProtocols.Http2;
 });
 })
                .UseStartup<Startup>();
        }
    }
}
```

这个片段展示了如何在端口`5002`上设置 HTTP/2 绑定。这种方法强制使用 HTTP/2 绑定，而不提供任何回退到 HTTP 1.1 的选项。

# 在 ASP.NET Core 中启用 CORS

安全性的另一个关键方面是保护我们的 API 免受 CORS 调用的攻击。默认情况下，无法使用客户端代码调用托管在其他域上的服务，因为诈骗网站可能使用跨源调用获取有关用户的敏感信息。这种安全限制被称为**同源策略**。

对**同源策略**的限制作用于使用以下标准的 HTTP 调用：

+   请求来自不同的域名（例如，位于`example.com`的网站调用`api.com`）。

+   请求来自不同的子域名（例如，位于`example.com`的网站调用`api.example.com`）。

+   请求来自不同的端口（例如，位于`example.com`的网站调用`example.com:3001`）。

+   请求来自不同的协议（例如，`https://example.com`网站调用`http://example.com`）。

CORS 提供了一种允许特定域名向不同域名托管的服务进行客户端调用的方法。当我们想要允许客户或第三方客户端无限制地调用我们的服务时，这种方法非常有用。还必须注意，CORS 可以启用以允许所有域名。这种方法必须避免，因为它将使攻击者能够不当使用我们的 API。

ASP.NET Core 提供了一个开箱即用的方式来启用 CORS。该框架允许使用两种方法创建 CORS 策略：*中间件方法*和*属性方法*。正如我们在第三章中看到的，*与中间件管道一起工作*，中间件通常被实现来开发覆盖整个 Web 服务的跨切面逻辑。另一方面，*属性*用于对单个操作应用限制。同样，这种方法也用于 CORS 策略。

# 使用中间件方法实现 CORS

可以使用 *CORS 中间件* 方法来启用特定的 HTTP 域、方法或端口调用我们的服务。与任何中间件一样，它可以在服务的 `Startup` 类的 `Configure` 方法中定义：

```cs
namespace Catalog.API
{
    public class Startup
    {
       ...

        public void Configure(IApplicationBuilder app, 
        IWebHostingEnvironment env)
        {
            ...
            app.UseCors(cfg =>
 {
 cfg.AllowAnyOrigin();
 });

            ..
        }
    }
}
```

`UseCors` 中间件扩展方法接受一个操作方法来配置不同的规则。例如，之前的代码执行了 `AllowAnyOrigin` 方法以允许来自任何网站的调用。同样，可以在特定域上定义更限制性的规则，如下所示：

```cs
namespace Catalog.API
{
    public class Startup
    {
       ...
        public void Configure(IApplicationBuilder app, 
        IWebHostingEnvironment env)
        {
            ...
            app.UseCors(cfg =>
 {
 cfg.AllowAnyOrigin("https://samuele.dev");
 });
            ...
        }
    }
}
```

在此情况下，我们阻止了所有来自[`samuele.dev/`](https://samuele.dev/)网站之外的所有跨源请求。定义 CORS 规则的更高级和简洁的方法是使用命名策略进行分组。也可以使用以下方法：

```cs
namespace Catalog.API
{
    public class Startup
    {

        public void ConfigureServices(IServiceCollection services)
        {
            ...
            services.AddCors(opt =>
            {
                opt.AddPolicy("BlogDomainPolicy", cfg => 
                { cfg.WithOrigins("https://samuele.dev"); });
            });
            ..
        }

        public void Configure(IApplicationBuilder app, 
        IWebHostingEnvironment env)
        {
           ...
            app.UseCors("BlogDomainPolicy");
           ...
        }
    }
}
```

之前的代码使用 `ConfigureServices` 方法中的 `app.AddCors` 构造定义了一个名为 `BlogDomainPolicy` 的 CORS 策略。一旦我们描述了策略的规则，我们就可以在 `Startup` 类的 `Configure` 方法中使用 `app.UseCors` 方法来使用定义的策略。这将使我们能够在 `Startup` 类中建立不同的策略，并条件性地应用它们。

# 使用属性方法实现 CORS

在某些情况下，可能需要为某些路由或操作定义特定的策略。因此，可以使用属性应用 CORS 策略，如下所示：

```cs
namespace Catalog.API.Controllers
{
    [Route("api/items")]
    [ApiController]
    [JsonException]
    [EnableCors("BlogDomainPolicy")]
    public class ItemController : ControllerBase
    {

        ...
    }
}
```

在此情况下，我们仅将 `BlogDomainPolicy` 的使用限制在 `ItemController` 上。因此，控制器下定义的所有路由都将使用相同的策略。同样，我们可以在控制器中的特定操作方法上添加策略：

```cs
namespace Catalog.API.Controllers
{
    [Route("api/items")]
    [ApiController]
    [JsonException]
    [EnableCors("BlogDomainPolicy")]
    public class ItemController : ControllerBase
    {
        ...

        [HttpGet("{id:guid}")]
        [EnableCors("GetByIdActionPolicy")]
        public async Task<IActionResult> GetById(string id)
        {
            ...
        }

        ...
    }
}
```

在那种情况下，`GetByIdActionPolicy` 将仅对 `GetById` 操作方法起作用，而 `BlogDomainPolicy` 将作用于整个控制器。这种方法提供了很好的粒度；此外，它提供了一种为服务中的单个路由指定策略的方法。下一节将描述基于令牌的认证方法的特点。

# 使用基于令牌的认证保护 API

应用程序传统上通过会话 cookie 持久化身份，依赖于存储在服务器端的会话 ID。这种方法带来了一些显著的问题和陷阱：它 *不是* *可扩展的*，因为您需要一个共同点来存储会话，并且每次用户进行认证时，服务器都需要在数据源中创建一个新的记录。因此，这种方法可能会成为您的网络服务的瓶颈。

现在，基于令牌的认证对于验证和授权用户非常有帮助，尤其是在分布式系统环境中。基于令牌的认证的主要优势在于消费者向身份服务请求令牌。接下来，客户端可以本地存储令牌，并用于认证和授权目的。

因此，令牌认证是无状态的，并且**设计用于可扩展性**。让我们看看令牌认证过程以及它是如何工作的，以便更好地理解这种方法的优点：

![图片](img/1ee505ee-009d-4b3e-a04b-4499295d996b.png)

此架构描述了在实现基于令牌的认证时的典型工作流程。该架构描述了三个实体：

+   **客户端**是尝试访问我们资源的应用程序。

+   **身份提供者**是提供服务的服务，给定用户名和密码，提供加密的认证令牌。

+   **资源提供者**是客户端调用的另一个服务。此外，**资源提供者**将接受加密的认证令牌，并在授权的情况下提供客户端请求的信息。

由于基于令牌的认证采用无状态的方法，应用程序不会存储认证令牌。因此，必须注意客户端必须在每次请求中传递认证令牌。

基于令牌的认证可以以不同的方式实现。**JSON Web Token**（**JWT**）是一个标准，定义在 RFC 7519（[`tools.ietf.org/html/rfc7519`](https://tools.ietf.org/html/rfc7519)）开放指令中，它描述了在双方之间表示声明的途径。JWT 被定义为包含有效载荷和签名的 JSON 对象，其中签名加密了令牌中的数据。换句话说，它通过使用密钥提供了一种加密以 JSON 格式安全数据的方法。近年来，JWT 令牌标准变得非常流行，因为网络服务可以用它实现两个目的：

+   **授权**：网络服务返回 JWT 令牌以传输有关声明和个人细节的信息给已登录的用户。此外，单点登录功能和令牌认证功能使用这种技术将数据传输到客户端。

+   **信息交换**：您可以使用 JWT 令牌标准通过使用提供的密钥对数据进行签名来防止数据被利用，并验证您收到的数据的真实性。

JWT 令牌的解剖结构非常类似于 Web 请求的结构。它由三部分组成：*头部*、*载荷*和*签名*。头部部分包含有关令牌类型和令牌使用的签名算法的信息：

```cs
{   "alg": "HS256",   "typ": "JWT" }
```

在那种情况下，我们可以推断出该令牌使用的是*HMAC* *SHA256*算法，并且它是一个 JWT 令牌类型。*载荷*部分是我们令牌的核心部分，它包含要发送给用户的信息。默认情况下，有一组预定义的信息用于填充，例如，`exp`（**过期时间**）字段。以下 JSON 是一个载荷的示例：

```cs
{
  "email": "example@handsonaspnetcore.com",
  "nbf": 1546196276,
  "exp": 1546801076,
  "iat": 1546196276
}
```

`邮箱`字段是对令牌的声明。`nbf`代表**在...之前无效**，而`iat`代表**签发于**。这三个字段代表自 UNIX 纪元以来计算的时间。

最后，令牌的*签名*部分使用在头部指定的密钥和算法对编码的头部和编码的有效负载进行签名。

生成的编码令牌类似于以下内容：

```cs
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InNhbXVlbGUucmVzY2FAZ21haWwuY29tIiwibmJmIjoxNTQ2MTk2Mjc2LCJleHAiOjE1NDY4MDEwNzYsImlhdCI6MTU0NjE5NjI3Nn0.yQGT1TJYL4U_IqBpoQ6MjUchET06BRE-YJ0sf-MRA
```

需要强调的是，编码令牌中的每个点分隔符（`.`）代表一个加密令牌，如前所述。

# 实现基于令牌的认证

在本节中，我们将了解如何使用 ASP.NET Core 执行基于令牌的认证。更详细地说，我们将深入了解基于令牌的认证的开发和测试，并学习如何使用 ASP.NET Core 的默认身份提供者将用户数据存储在数据库中。

此外，我们将实现身份验证作为*目录服务*解决方案的一部分。需要注意的是，在实际应用中，身份验证和整个身份过程都有一个专门的服务和独立的数据存储。

为了演示目的，我们将在*目录服务*中实现认证部分。请考虑将您应用程序的身份验证部分保留在单独的服务中。

让我们先添加我们需要的包来开发基于令牌的认证，这些包在`Catalog.Domain`和`Catalog.Infrastructure`项目中。请注意，这些包仅与`netcoreapp3.1`框架兼容，因此您需要将 csproj 中的`TargetFramework`变量从`netstandard2.1`更改为`netcoreapp3.1`：

```cs
dotnet add Catalog.Domain package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add Catalog.Infrastructure package Microsoft.AspNetCore.Identity.EntityFrameworkCore
```

下一步是在`Catalog.Domain`项目中定义`User`实体：

```cs
using Microsoft.AspNetCore.Identity;

namespace Catalog.Domain.Entities
{
    public class User : IdentityUser
    {
        public string Name { get; set; }
    }
}
```

`User`实体代表服务*领域模型*中的通用用户。需要注意的是，它扩展了`IdentityUser`类，为实体提供了额外的字段。`IdentityUser`类标识一个可存储的用户实体。此外，该实体可以通过`Microsoft.AspNetCore.Identity`包存储数据。

让我们继续在`Catalog.Domain`项目中声明`IUserRepository`接口：

```cs
using System.Threading;
using System.Threading.Tasks;
using Catalog.Domain.Entities;

namespace Catalog.Domain.Repositories
{
    public interface IUserRepository
    {
        Task<bool> AuthenticateAsync(string email, string password, 
            CancellationToken cancellationToken = default);
        Task<bool> SignUpAsync(User user, string password, 
            CancellationToken cancellationToken = default);
        Task<User> GetByEmailAsync(string requestEmail, 
            CancellationToken cancellationToken = default);
    }
}
```

此接口代表了*目录服务*和数据层之间的中介。此外，它可以用于验证、注册和检索`User`实体。`IUserRepository`充当用户数据的数据存储，并执行与用户相关的操作，例如注册过程和身份验证。需要注意的是，`AuthenticateAsync`和`SignUpAsync`方法返回一个布尔值，表示相应的操作是否成功。

# 定义服务层

在定义了`User`实体和`IUserRepository`接口之后，我们可以在`Catalog.Domain`项目中继续定义服务层。让我们先描述`IUserService`接口：

```cs
using System.Threading;
using System.Threading.Tasks;
using Catalog.Domain.Repositories;
using Catalog.Domain.Requests.User;
using Catalog.Domain.Responses;

namespace Catalog.Domain.Services
{
    public interface IUserService
    {
        Task<UserResponse> GetUserAsync(GetUserRequest request, 
            CancellationToken cancellationToken = default);
        Task<UserResponse> SignUpAsync(SignUpRequest request, 
            CancellationToken cancellationToken = default);
        Task<TokenResponse> SignInAsync(SignInRequest request, 
            CancellationToken cancellationToken = default);
    }
}
```

接口定义了认证阶段所需的必要方法。`GetUserAsync` 方法使用 `GetUserRequest` 类型检索与特定用户相关的信息。`SignUpAsync` 和 `SignInAsync` 方法定义了注册和登录过程：注册操作返回一个新的 `UserResponse` 实例，它确定了注册用户的有关信息，而登录操作返回包含结果的 `TokenResponse` 实例，该实例将包含客户端将存储的令牌。因此，让我们通过定义服务接口使用的请求 DTO 来继续：

```cs
namespace Catalog.Domain.Requests.User
{
    public class GetUserRequest
    {
        public string Email { get; set; }
    }

    public class SignInRequest
    {
        public string Email { get; set; }
        public string Password { get; set; }
    }

    public class SignUpRequest
    {
        public string Email { get; set; }
        public string Password { get; set; }
        public string Name { get; set; }
    }
}
```

为了简洁起见，请求类以唯一的代码片段表示。`GetUserRequest` 类型包含一个 `Email` 字段，指定要检索的电子邮件地址。正如我们将在本章后面看到的那样，控制器上的 `GetUser` 动作方法需要认证才能检索用户数据。

`SignInRequest` 类型定义了用于认证用户的 `Email` 和 `Password` 字段。最后，`SignUpRequest` 类型也包含用字符串表示的用户 `Name`。请注意，出于演示目的，代码仅存储用户的 `Name`。在实际应用中，`SignUpRequest` 类型的复杂性可能会随着更多个人信息的增加而增加。

让我们继续定义 `IUserService` 接口使用的响应类型：

```cs
namespace Catalog.Domain.Responses
{
    public class TokenResponse
    {
        public string Token { get; set; }
    }

    public class UserResponse
    {
        public string Name { get; set; }
        public string Email { get; set; }
    }
}
```

`UserResponse` 类型旨在检索用户的全部个人信息。需要注意的是，出于安全原因，它显然省略了实体的 `Password` 字段。另一方面，`TokenResponse` 类型检索包含认证过程产生的 JWT 令牌的 `Token` 字段。

因此，我们可以继续描述 `IUserService` 接口的实现：它将包含与 ASP.NET Core 用于认证的令牌生成以及用户实体上的获取和注册操作相关的逻辑。以下代码展示了实现中的依赖关系：

```cs
using System;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using Catalog.Domain.Repositories;
using Catalog.Domain.Requests.User;
using Catalog.Domain.Responses;
using Microsoft.Extensions.Options;
using Microsoft.IdentityModel.Tokens;

namespace Catalog.Domain.Services
{
    public class UserService : IUserService
    {
 private readonly AuthenticationSettings 
            _authenticationSettings;
 private readonly IUserRepository _userRepository;

        public UserService(IUserRepository userRepository, 
 IOptions<AuthenticationSettings> authenticationSettings)
        {
            _userRepository = userRepository;
            _authenticationSettings = authenticationSettings.Value;
        }
```

如前所述，`IUserRepository` 接口用作查询和操作我们数据源的主要入口点。`IOption<AuthenticationSettings>` 类型定义了认证过程所需的设置：

```cs
 namespace Catalog.Domain.Configurations
{
    public class AuthenticationSettings
    {
        public string Secret { get; set; }
        public int ExpirationDays { get; set; }
    }
}
```

该类存储在 `Catalog.Domain` 项目的 `Configurations` 文件夹中。`AuthenticationSettings` 类包含 `Secret` 字段，它描述了一个用于加密令牌信息的短语，以及 `ExpirationDays` 字段，它提供了省略的令牌在多少天后过期。此外，我们可以继续定义 `GetUserAsync` 和 `SignUpAsync` 方法：

```cs
...
public async Task<UserResponse> GetUserAsync(GetUserRequest request, 
    CancellationToken cancellationToken)
{
    var response = await _userRepository.GetByEmailAsync(request.Email, 
        cancellationToken);

    return new UserResponse { Name = response.Name, Email = 
        response.Email };
}

public async Task<UserResponse> SignUpAsync(SignUpRequest request, 
    CancellationToken cancellationToken)
{
    var user = new Entities.User { Email = request.Email, UserName = 
        request.Email, Name = request.Name };

    bool result = await _userRepository.SignUpAsync(user, 
        request.Password, cancellationToken);

    return !result ? null : new UserResponse { Name = request.Name, 
        Email = request.Email };
}
...
```

`GetUserAsync` 方法使用高级层提供的请求来执行 `IUserRepository` 接口的 `GetByEmailAsync` 方法。它还映射响应并检索一个新的 `UserReponse` 类型实例。

另一方面，`SignUpAsync` 方法使用相应的值初始化一个新的 `User` 实例，并执行 `IUserRepository` 接口提供的 `SignUpAsync` 方法。最后，如果用户已创建，`SignUpAsync` 方法检索一个新的 `UserResponse` 实例。让我们继续通过定义 `SignInAsync` 方法来完成 `IUserService` 的实现：

```cs
...
public async Task<TokenResponse> SignInAsync(SignInRequest request,     CancellationToken cancellationToken)
{
    bool response = await _userRepository.
       AuthenticateAsync(request.Email, request.Password,
       cancellationToken);

    return response == false ? null : new TokenResponse { Token = 
        GenerateSecurityToken(request)          k . };

}

private string GenerateSecurityToken(SignInRequest request)
{
    var tokenHandler = new JwtSecurityTokenHandler();
    var key = Encoding.ASCII.GetBytes(_authenticationSettings.Secret);

    var tokenDescriptor = new SecurityTokenDescriptor
    {
        Subject = new ClaimsIdentity(new[]
        {
            new Claim(ClaimTypes.Email, request.Email)
        }),
        Expires = 
        DateTime.UtcNow.AddDays
        (_authenticationSettings.ExpirationDays),
        SigningCredentials = new SigningCredentials(new 
        SymmetricSecurityKey(key), 
            SecurityAlgorithms.HmacSha256Signature)
    };

    var token = tokenHandler.CreateToken(tokenDescriptor);
    return tokenHandler.WriteToken(token);
}
...
```

作为第一步，`SignInAsync` 方法通过提供客户端发送的 `Email` 和 `Password`，调用由 `IUserRepository` 提供的底层 `AuthenticateAsync` 方法。该语句返回一个布尔变量，指示用户是否已认证。如果用户已认证，该方法通过调用 `GenerateSecurityToken` 方法检索 `TokenResponse` 类的新实例。

`GenerateSecurityToken` 方法定义了一个新的 `JwtSecurityTokenHandler` 类型的实例，该实例通过使用 `CreateToken` 和 `WriteToken` 方法提供了一些生成和创建令牌的实用工具。

此外，它定义了一个新的 `SecurityTokenDescriptor` 类型的实例，通过在 `AuthorizationSettings` 实例的 `Secret` 字段上签名，声明了 `Expire` 时间和 `SigningCredentials` 字段。

# 在控制器上应用认证

接下来的步骤包括在依赖注入引擎中注册依赖项，并在控制器层中使用生成的依赖项，例如 `IUserService` 实例。因此，本节重点介绍了 `Catalog.API` 和 `Catalog.Infrastructure` 项目。

让我们从在 `Catalog.Infrastructure` 项目中定义一个新的扩展方法开始，该方法添加了认证部分：

```cs
using System.Text;
using Catalog.Domain.Configurations;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.IdentityModel.Tokens;
namespace Catalog.Infrastructure.Extensions
{
    public static class AuthenticationExtensions
    {
        public static IServiceCollection AddTokenAuthentication(this 
            IServiceCollection services, IConfiguration configuration)
        {
            var settings = configuration.GetSection
                ("AuthenticationSettings");
            var settingsTyped = settings.Get<AuthenticationSettings>();

            services.Configure<AuthenticationSettings>(settings);
            var key = Encoding.ASCII.GetBytes(settingsTyped.Secret);
            services.AddAuthentication(x =>
                {
                    x.DefaultAuthenticateScheme = 
                        JwtBearerDefaults.AuthenticationScheme;
                    x.DefaultChallengeScheme = 
                        JwtBearerDefaults.AuthenticationScheme;
                })
                .AddJwtBearer(x =>
                {
                    x.TokenValidationParameters = new 
                        TokenValidationParameters
                        {
                            IssuerSigningKey = new 
                                SymmetricSecurityKey(key),
                            ValidateIssuer = false,
                            ValidateAudience = false
                        };
                });
            return services;
        }
    }
}
```

上述代码的核心部分是执行两个方法：`AddAuthentication` 和 `AddJwtBearer`。这两个扩展方法都添加了认证过程中使用的中间件和服务。更详细地说，`AddAuthentication` 通过应用 JWT 携带者认证方案来指定 `DefaultAuthenticationScheme` 和 `DefaultChallengeScheme`。

同时，`AddJwtBearer` 方法定义了与令牌认证相关的选项，例如 `TokenValidationParameters` 字段，它包括用于验证令牌参数的 `SigningKey`。

此外，`IssuerSigningKey` 必须与生成令牌时使用的密钥相同。否则，验证将失败。重要的是要注意，`ValidateIssuer` 和 `ValidateAudience` 字段设置为 `false`。因此，ASP.NET Core 不会验证发行者或受众 URL。尽管这种方法在测试环境中运行良好，但我强烈建议在生产环境中使用以下设置：

```cs
.AddJwtBearer(x =>
{
    x.TokenValidationParameters = new TokenValidationParameters
    {
        IssuerSigningKey = new SymmetricSecurityKey(key),
        ValidateIssuer = true,
 ValidateAudience = true,
 ValidIssuer = "yourhostname",
 ValidAudience = "yourhostname"
    };
});
```

在这种情况下，将验证发行者和受众；因此，它将检查令牌的发行者和受众是否与配置中指定的相匹配。`AddTokenAuthentication` 扩展方法还拥有 `UserService` 类使用的 `AuthenticationSettings` 的注册。因此，让我们看看在 `appsettings.json` 文件中定义的 `AuthenticationSettings` 值：

```cs
...
"AuthenticationSettings": {
    "Secret": "My Super long secret",
    "ExpirationDays": "7"
}
...
```

然后，我们可以通过向 `Catalog.API` 项目的 `Startup` 类中添加身份验证实现来继续操作：

```cs
namespace Catalog.API
{
    public class Startup
    {
        public Startup(IConfiguration configuration, 
        IWebHostingEnvironment environment)
        {
            Configuration = configuration;
            CurrentEnvironment = environment;
        }

        ...
        public void ConfigureServices(IServiceCollection services)
        {
            ...

            services
                 .AddTokenAuthentication(Configuration)

            ...
        }

        public void Configure(IApplicationBuilder app, 
        IHostingEnvironment env)
        {
            ...
          app.UseAuthentication();
            app.UseAuthorization(); app.UseEndpoints(endpoints =>     {     endpoints.MapControllers();});
        }
    }
}
```

`Startup` 类是初始化身份验证过程的核心理念。在 `ConfigureServices` 方法中，它通过从 `appsettings.json` 文件中读取来配置和初始化 `AuthorizationSettings` 类。接下来，它通过传递 `AuthorizationSettings` 类型实例调用 `AddAuthentication` 扩展方法。还必须注意的是，`Configure` 方法通过调用 `UseAuthentication` 方法添加身份验证中间件。

最后，我们可以通过添加 `UserController` 并公开身份验证路由来继续操作：

```cs
using System.Linq;
using System.Security.Claims;
using System.Threading.Tasks;
using Catalog.API.Filters;
using Catalog.Domain.Requests.User;
using Catalog.Domain.Services;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace Catalog.API.Controllers
{
    [Authorize]
    [ApiController]
    [Route("api/user")]
    [JsonException]
    public class UserController : ControllerBase
    {
        private readonly IUserService _userService;

        public UserController(IUserService userService)
        {
            _userService = userService;
        }

        [HttpGet]
        public async Task<IActionResult> Get()
        {
            var claim = HttpContext.User.Claims.FirstOrDefault(x => 
                x.Type == ClaimTypes.Email);

            if (claim == null) return Unauthorized();

            var token = await _userService.GetUserAsync(new 
                GetUserRequest { Email = claim.Value });
            return Ok(token);
        }

        [AllowAnonymous]
        [HttpPost("auth")]
        public async Task<IActionResult> SignIn(SignInRequest request)
        {
            var token = await _userService.SignInAsync(request);

            if (token == null) return BadRequest();

            return Ok(token);
        }

        [AllowAnonymous]
        [HttpPost]
        public async Task<IActionResult> SignUp(SignUpRequest request)
        {
            var user = await _userService.SignUpAsync(request);
            if (user == null) return BadRequest();
            return CreatedAtAction(nameof(Get), new { }, null);
        }
    }
}
```

上述代码定义了 `UserController` 类，该类公开身份验证路由。重要的是要注意，整个控制器都使用 `[Authorize]` 属性进行装饰，这意味着每个路由都受到身份验证的保护。因此，要访问控制器中声明的路由，必须在请求中使用有效的令牌。该类为服务层中定义的每个操作定义了一个动作方法：

+   `Get` 动作方法暴露了有关当前用户的一些细节，例如 `Email` 字段和 `Name` 字段。动作方法从传入的令牌中获取用户详情。令牌信息通过访问 `HttpContext.User` 属性并获取 `ClaimType.Email` 的值来表示。

+   `SignIn` 动作方法也使用 `[AllowAnonymous]` 属性进行装饰。此外，还可以在不进行身份验证的情况下调用动作方法。动作方法将 `request.Email` 和 `request.Password` 字段绑定，并使用 `IUserService` 接口发送请求对象。动作方法返回带有生成令牌的 `TokenResponse`。

+   `SignUp` 动作方法也使用 `[AllowAnonymous]` 属性进行装饰。在这种情况下，动作方法注册一个新用户，如果操作成功，则返回 `201 Created` HTTP 状态码。

我们现在的设置几乎完成了。我们需要做的是定义 `IUserRepository` 接口和底层数据存储之间的最后一个共同点。为此，我们将再次使用 EF Core 框架和由 Microsoft 维护的 `Microsoft.AspNetCore.Identity.EntityFrameworkCore` 包。

# 使用 EF Core 存储数据

让我们继续实现数据访问层，并创建 `IUserRepository` 接口的具体实现。`UserRepository` 类将有两个主要依赖项，即 `SignInManager` 和 `UserManager` 类，这两个类都由 `Microsoft.AspNetCore.Identity` 包提供：

```cs
using System.Threading;
using System.Threading.Tasks;
using Catalog.Domain.Entities;
using Catalog.Domain.Repositories;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;

namespace Catalog.Infrastructure.Repositories
{
    public class UserRepository : IUserRepository
    {
        private readonly SignInManager<User> _signInManager;
        private readonly UserManager<User> _userManager;

        public UserRepository(UserManager<User> userManager, 
            SignInManager<User> signInManager)
        {
            _userManager = userManager;
            _signInManager = signInManager;
        }

        public async Task<bool> AuthenticateAsync(string email, 
            string password, CancellationToken cancellationToken)
        {
            var result = await _signInManager.PasswordSignInAsync(
                email, password, false, false);
            return result.Succeeded;
        }

        public async Task<bool> SignUpAsync(User user, string password, 
            CancellationToken cancellationToken)
        {
            var result = await _userManager.CreateAsync(user, 
                password);
            return result.Succeeded;
        }

        public async Task<User> GetByEmailAsync(string requestEmail, 
            CancellationToken cancellationToken)
        {
            return await _userManager
                .Users
                .FirstOrDefaultAsync(u => u.Email == requestEmail, 
                    cancellationToken);
        }
    }
}
```

如您所见，生成的代码实现了 `IUserRepository` 接口。该类依赖于 `SignInManager<User>` 和 `UserManager<User>` 类型。这些类型接受一个泛型实体类，该类是认证对象的表示。`SignInManager<T>` 泛型类提供了与用户登录过程交互的功能。

它公开了 `PasswordSignInAsync` 方法，该方法由 `UserRepository.Authenticate` 方法使用。另一方面，`UserManager<T>` 类提供了与持久存储中用户交互的方法。此外，`UserRepository` 使用 `SignUp` 和 `UserRepository.GetByEmail` 方法与数据库交互。

# 声明身份数据库上下文

一旦我们声明了 `IUserRepository` 的实现，我们就可以继续声明 *身份数据上下文*。*身份数据上下文* 通过扩展 `IdentityDbContext` 类来识别。这种类型的 `DbContext` 由 EF Core 用于定位和访问用作持久用户存储的数据源。为了声明 *身份数据上下文*，有必要以下列方式扩展 `CatalogContext`：

```cs
using System.Threading;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;
using Catalog.Domain.Entities;
using Catalog.Domain.Repositories;
using Catalog.SchemaDefinitions;

namespace Catalog.Infrastructure
{
    public class CatalogContext : IdentityDbContext<User>, IUnitOfWork
    {
          ...

        protected override void OnModelCreating(ModelBuilder 
            modelBuilder)
        {
            modelBuilder.ApplyConfiguration(new 
                ItemEntitySchemaDefinition());
            modelBuilder.ApplyConfiguration(new 
                GenreEntitySchemaConfiguration());
            modelBuilder.ApplyConfiguration(new 
                ArtistEntitySchemaConfiguration());

            base.OnModelCreating(modelBuilder);
        }
    }
}
```

重要的是要注意，`IdentityDbContext` 类扩展了 `DbContext` 类。此外，`DbContext` 类中存在的每个属性和行为也被 `IdentityDbContext` 类继承。因此，必须注意，重写方法 `OnModelCreating` 也必须调用基方法。

为了提供使用 EF Core 存储用户信息的方法，还必须通过调用 `AddIdentity` 扩展方法添加和配置指定 `User` 类型的身份系统。此外，还必须调用 `AddEntityFrameworkStores` 并引用 `CatalogContext` 类以添加实体框架实现。以下代码是之前创建的 `AddAuthentication` 扩展方法：

```cs
using System.Text;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Identity;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.IdentityModel.Tokens;
using Catalog.Domain;
using Catalog.Domain.Entities;

namespace Catalog.Infrastructure.Extensions
{
    public static class AuthenticationExtensions
    {
        public static IServiceCollection AddTokenAuthentication(this 
        IServiceCollection services, AuthenticationSettings settings)
        {
            var key = Encoding.ASCII.GetBytes(settings.Secret);

            services.AddIdentity<User, IdentityRole>()
 .AddEntityFrameworkStores<CatalogContext>();

            ...
            return services;
        }
    }
}
```

最后，我们可以通过初始化 `IUserRepository` 的具体实现来继续操作。在这种情况下，我们将在 `Startup` 类中声明依赖注入解析：

```cs
        public void ConfigureServices(IServiceCollection services)
        {
            ...
            services.AddScoped<IUserRepository, UserRepository>();            
```

同样，我们可以通过向 `AddServices` 扩展方法添加以下行来注册 `IUserService`：

```cs
public static IServiceCollection AddServices(this IServiceCollection services)
{
    services
          ...
        .AddScoped<IUserService, UserService>();

    return services;
}
```

总结来说，现在我们已经部署了整个认证栈。`Catalog.API` 项目通过 `UserController` 类公开了 HTTP 路由。该控制器依赖于 `IUserService` 接口，该接口公开了认证过程所需的操作。

因此，`UserService` 类依赖于 `IUserRepository` 接口，这是调用 EF Core 框架公开的 API 的主要入口点。因此，我们现在可以通过一些测试来验证认证逻辑。

# 测试认证

在这个阶段测试我们的代码是至关重要的：我们应该在将应用程序运行在服务器实例之前检查、记录并验证我们系统的行为。此外，测试认证行为也非常关键，因为它是我们服务的一个敏感部分。

由于 `UserRepository` 实现是认证堆栈中最底层的部分，并且是第一个依赖于 EF Core 来检索、更新和验证用户的组件，我们可以将其保持隔离，并通过模拟 `IUserRepository` 接口来排除它从测试过程中。

`SignInManager<T>` 和 `UserManager<T>` 类代表了我们认证过程的核心部分，它们是微软维护的第三方包的一部分。此外，没有必要用测试来覆盖它们的实现。

让我们从在 `tests` 文件夹中的 `Catalog.Fixture` 项目中定义一个新的 `UserContextFactory` 开始：

```cs
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using Catalog.Domain.Entities;
using Catalog.Domain.Repositories;
using Microsoft.AspNetCore.Identity;
using Moq;

namespace Catalog.Fixtures
{
    public class UsersContextFactory
    {
        private readonly PasswordHasher<User> _passwordHasher;
        private readonly IList<User> _users;

        public UsersContextFactory()
        {
            _passwordHasher = new PasswordHasher<User>();

            _users = new List<User>();

            var user = new User
            {
                Id = "test_id",
                Email = "samuele.resca@example.com",
                Name = "Samuele Resca"
            };
            user.PasswordHash = _passwordHasher.HashPassword(user, 
            "P@$$w0rd");

            _users.Add(user);
        }

        public IUserRepository InMemoryUserManager => 
        GetInMemoryUserManager();

        private IUserRepository GetInMemoryUserManager()
 {
 ...
 }
    }
}
```

工厂类公开了一个带有一些预填充数据的 `IUserRepository` 实例，并且它依赖于 `PasswordHasher<T>` 泛型类型，该类型在 `SignUp` 和 `Authenticate` 模拟方法声明中用于通过 `HashPassword` 和 `VerifyHashedPassword` 方法来 *编码-解码* 密码。

重要的是要注意，`IUserRepository` 接口是通过使用 `GetInMemoryUserManager` 方法模拟的。此外，它使用 `List<User>` 来模拟数据源，并使用 `Moq` 库实现了 `IUserRepository` 接口暴露的 `AuthenticateAsync`、`GetByEmailAsync` 和 `SignUpAsync` 方法：

```cs
namespace Catalog.Fixtures
{
    public class UsersContextFactory
    {
    ...
        private IUserRepository GetInMemoryUserManager()
        {
            var fakeUserService = new Mock<IUserRepository>();

            fakeUserService.Setup(x => 
            x.AuthenticateAsync(It.IsAny<string>(), 
            It.IsAny<string>(), CancellationToken.None))
                .ReturnsAsync((string email, string password, 
                CancellationToken token) =>
                {
                    var user = _users.FirstOrDefault(x => 
                    x.Email == email);

                    if (user == null) return false;

                    var result = _passwordHasher.
                        VerifyHashedPassword(user,  
                    user.PasswordHash, password);
                    return result == PasswordVerificationResult.
                        Success;
                });
            fakeUserService.Setup(x => 
            x.GetByEmailAsync(It.IsAny<string>(), 
                CancellationToken.None))
                .ReturnsAsync((string email, CancellationToken token) 
            => 
                 _users.First(x => x.Email == email));
            fakeUserService.Setup(x => x.SignUpAsync(It.IsAny<User>(), 
            It.IsAny<string>(), CancellationToken.None))
                .ReturnsAsync((User user, string password,  
                CancellationToken token) =>
                {
                    user.PasswordHash = 
                        _passwordHasher.HashPassword(user, 
                    password);
                    _users.Add(user);
                    return true;
                });
            return fakeUserService.Object;
        }
    }
}
```

之前的代码通过为接口的方法提供模拟行为来返回一个模拟的 `IUserRepository` 实例。因此，我们可以通过实现以下测试类来验证 `IUserService` 类：

```cs
using System.Threading.Tasks;
...
namespace Catalog.Domain.Tests.Services
{
    public class UserServiceTests : IClassFixture<UsersContextFactory>
    {
        private readonly IUserService _userService;

        public UserServiceTests(UsersContextFactory 
            usersContextFactory)
        {
            _userService = new UserService(usersContextFactory.
             InMemoryUserManager, Options.Create( 
             new AuthenticationSettings { Secret = 
             "Very Secret key-word to match", ExpirationDays = 7 }));
        }
        [Fact]
        public async Task 
        signin_with_invalid_user_should_return_a_valid_token_response()
        {
            var result =
                await _userService.SignInAsync(new SignInRequest { 
                Email = "invalid.user", Password = "invalid_password" });
            result.ShouldBeNull();
        }
        [Fact]
        public async Task 
        signin_with_valid_user_should_return_a_valid_token_response()
        {
            var result =
                await _userService.SignInAsync(new SignInRequest { 
                Email = "samuele.resca@example.com",
                Password = "P@$$w0rd" });
            result.Token.ShouldNotBeEmpty();
        }
        ...
    }
}
```

测试类实现了两个不同的测试：`signin_with_invalid_user_should_return_a_valid_token_response` 和 `signin_with_valid_user_should_return_a_valid_token_response`。在两种情况下，测试都将使用 `UserContextFactory` 来解决类的依赖。我们还将使用 ASP.NET Core 提供的 `Option.Create` 方法来生成 `AuthenticationSettings` 选项。在这种情况下，我们正在测试在处理层中实现的整体堆栈。

重要的是要注意，我们正在排除与用户信息管理和存储相关的整个底层部分。我们可以通过包括控制器部分来扩展测试范围。更详细地说，我们可以实现测试来检查`UserController`类中实现的功能。为此，我们将在`TStartup`时间注入一个假的`IUserRepository`实现，使用`services.Replace`指令：

```cs
using System;
...

namespace Catalog.Fixtures
{
    public class InMemoryApplicationFactory<TStartup>
        : WebApplicationFactory<TStartup> where TStartup : class
    {
        protected override void ConfigureWebHost(IWebHostBuilder 
            builder)
        {
            builder
                .UseEnvironment("Testing")
                .ConfigureTestServices(services =>
                {
                    ...
                    services.Replace(ServiceDescriptor.Scoped(_ => new       
                    UsersContextFactory().InMemoryUserManager));

                    var sp = services.BuildServiceProvider();

                    using var scope = sp.CreateScope();
                    var scopedServices = scope.ServiceProvider;
                    var db = scopedServices.GetRequiredService
                    <CatalogContext>();
                    db.Database.EnsureCreated();
                });
        }
    }
}
```

我们可以通过初始化`UsersContextFactory`类来对`InMemoryApplicationFactory<TStartup>`类进行操作，用模拟类的新的实例替换`IUserService`。之后，将能够通过解析`InMemoryApplicationFactory<TStartup>`工厂类来测试`UserController`类的操作：

```cs
using System.Net;
...
namespace Catalog.API.Tests.Controllers
{
    public class UserControllerTests : 
 IClassFixture<InMemoryApplicationFactory<Startup>>
    {
        private readonly InMemoryApplicationFactory<Startup> _factory;

        public UserControllerTests(InMemoryApplicationFactory<Startup> 
        factory)

        {
            _factory = factory;
        }

        [Theory]
        [InlineData("/api/user/auth")]
        public async Task sign_in_should_retrieve_a_token(string url)
        {
            var client = _factory.CreateClient();
            var request = new SignInRequest { Email = 
            "samuele.resca@example.com", Password = "P@$$w0rd" };
            var httpContent =
                new StringContent(JsonConvert.SerializeObject(request), 
                Encoding.UTF8, "application/json");

            var response = await client.PostAsync(url, httpContent);
            string responseContent = await 
            response.Content.ReadAsStringAsync();

            response.EnsureSuccessStatusCode();
            response.StatusCode.ShouldBe(HttpStatusCode.OK);
            responseContent.ShouldNotBeEmpty();
        }

        [Theory]
        [InlineData("/api/user/auth")]
        public async 
        Task sign_in_should_retrieve_bad_request_with_invalid_password
        (string url)
        {
            var client = _factory.CreateClient();
            var request = new SignInRequest { Email = 
            "samuele.resca@example.com", Password = "NotValidPWD" };
            var httpContent =
                new StringContent(JsonConvert.SerializeObject(request), 
                Encoding.UTF8, "application/json");

            var response = await client.PostAsync(url, httpContent);
            string responseContent = await 
            response.Content.ReadAsStringAsync();
            response.StatusCode.ShouldBe(HttpStatusCode.BadRequest);
            responseContent.ShouldNotBeEmpty();
        }
            ...
        }
    }
}
```

之前的代码验证了`UserController`类中定义的路由，并且还通过检查认证过程执行了集成测试。`sign_in_should_retrieve_a_token`测试方法使用 HTTP `POST`动词调用`/api/user/auth`地址来验证登录过程的实现。

此外，它还验证了在用户密码错误时的操作。此外，我们还可以提供更多测试来验证从登录阶段到检索已认证用户数据的整个认证过程：

```cs
...

[Theory]
[InlineData("/api/user")]
public async Task get_with_authorized_user_should_retrieve_the_right_user(string url)
{
    var client = _factory.CreateClient();

    var signInRequest = new SignInRequest { Email = 
    "samuele.resca@example.com", Password = "P@$$w0rd" };
    var httpContent = new StringContent(JsonConvert.SerializeObject
    (signInRequest), Encoding.UTF8, "application/json");

    var response = await client.PostAsync(url + "/auth", httpContent);
    string responseContent = await 
        response.Content.ReadAsStringAsync();

    response.EnsureSuccessStatusCode();

    var tokenResponse = JsonConvert.DeserializeObject<TokenResponse>
    (responseContent);

    client.DefaultRequestHeaders.Authorization =
        new AuthenticationHeaderValue("Bearer", tokenResponse.Token);

    var restrictedResponse = await client.GetAsync(url);

    restrictedResponse.EnsureSuccessStatusCode();
    restrictedResponse.StatusCode.ShouldBe(HttpStatusCode.OK);
}
...
```

更详细地说，`get_with_authorized_user_should_retrieve_the_right_user`测试执行以下操作：

1.  它向以下路由/`auth`执行`POST`请求以认证用户。

1.  它反序列化`POST`请求的结果，并获取令牌字段。

1.  它通过传递令牌添加一个认证头，并对`/api/user`路由执行请求。

1.  它检查结果状态码是否为 HTTP `200 OK`。

以这种方式，我们正在测试`UsersController`类以及每个操作方法中使用的底层处理程序。通过运行`docker-compose up --build`命令来运行目录网络服务也可以测试认证过程。首先，我们需要通过添加一些必要的信息，如用户的电子邮件和姓名来创建一个新用户：

```cs
curl -X POST \
  https://localhost:5001/api/users \
  -H 'Content-Type: application/json' \
  -d '{
    "email": "newuser@example.com",
    "password": "<my_secret_password>",
    "name": "Your name"
}'
```

之前的 HTTP 调用（以`curl`调用形式编写）使用指定的凭据创建了一个新用户。我们可以通过使用我们的凭据生成令牌来继续操作：

```cs
curl -X POST \
  https://localhost:5001/api/users/auth \
  -H 'Content-Type: application/json' \
  -d '{
    "email": "newuser@example.com",
    "password": "<my_secret_password>"
}'
```

最后，我们可以通过以下调用继续调用秘密端点：

```cs
curl -X GET https://localhost:5001/api/users -H 'Authorization: Bearer <my_token>'
```

上述`curl`命令通过在请求的`Authorization`头中传递令牌来调用`https://localhost:5001/api/users/`地址。

# 摘要

在本章中，我们学习了如何使用一些标准实践来保护一个网络服务。HTTPS 现在是一个标准和必须具备的功能，如果你想要保护数据。此外，我们还看到了基于令牌的身份验证如何提供一种保护暴露的数据和信息的有用方式。

本章涵盖的主题提供了一种确保由网络服务暴露的信息的安全的方法，并探讨了在分布式系统中基于令牌的认证实现。

在下一章中，我们将看到如何在 ASP.NET Core 中缓存响应，并了解缓存机制的一般工作原理。
