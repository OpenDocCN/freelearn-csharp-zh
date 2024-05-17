# 第十四章：使用 ASP.NET Core Razor Pages 构建网站

本章介绍了如何使用 Microsoft ASP.NET Core 在服务器端构建具有现代 HTTP 架构的网站。您将了解如何使用 ASP.NET Core 2.0 引入的 Razor Pages 功能和 ASP.NET Core 2.1 引入的 Razor 类库功能构建简单的网站。

本章将涵盖以下主题：

+   了解 Web 开发

+   了解 ASP.NET Core

+   探索 ASP.NET Core Razor Pages

+   使用 Entity Framework Core 与 ASP.NET Core

+   使用 Razor 类库

+   配置服务和 HTTP 请求管道

# 了解 Web 开发

为 Web 开发意味着使用**超文本传输协议**（**HTTP**），因此我们将从回顾这一重要的基础技术开始。

## 了解 HTTP

为了与 Web 服务器通信，客户端，也称为**用户代理**，使用 HTTP 通过网络进行调用。因此，HTTP 是 Web 的技术基础。因此，当我们谈论网站和 Web 服务时，我们指的是它们使用 HTTP 在客户端（通常是 Web 浏览器）和服务器之间进行通信。

客户端发出 HTTP 请求以获取资源，例如页面，由**统一资源定位符**（**URL**）唯一标识，并且服务器发送 HTTP 响应，如*图 14.1*所示：

![图形用户界面，文本描述自动生成](img/Image00104.jpg)

图 14.1：HTTP 请求和响应

您可以使用 Google Chrome 和其他浏览器记录请求和响应。

**良好实践**：Google Chrome 可用于比其他任何浏览器更多的操作系统，并且具有强大的内置开发人员工具，因此它是测试网站的良好首选浏览器。始终使用 Chrome 和至少另外两个浏览器（例如，Firefox 和 macOS 和 iPhone 的 Safari）测试您的 Web 应用程序。微软 Edge 于 2019 年从使用微软自己的渲染引擎切换到使用 Chromium，因此测试它的重要性较小。如果微软的 Internet Explorer 被使用，它往往主要用于组织内部的内联网。

### 了解 URL 的组成部分

URL 由几个组件组成：

+   **方案**：`http`（明文）或`https`（加密）。

+   **域**：对于生产网站或服务，**顶级域**（**TLD**）可能是`example.com`。您可能有诸如`www`，`jobs`或`extranet`之类的子域。在开发过程中，您通常为所有网站和服务使用`localhost`。

+   **端口号**：对于生产网站或服务，`http`为`80`，`https`为`443`。这些端口号通常从方案中推断出来。在开发过程中，通常使用其他端口号，例如`5000`，`5001`等，以区分使用共享域`localhost`的所有网站和服务。

+   **路径**：到资源的相对路径，例如`/customers/germany`。

+   **查询字符串**：传递参数值的一种方法，例如`?country=Germany&searchtext=shoes`。

+   **片段**：使用其`id`引用网页上的元素，例如`#toc`。

### 为本书中的项目分配端口号

在本书中，我们将为所有网站和服务使用域`localhost`，因此我们将使用端口号来区分多个项目在同一时间需要执行的情况，如下表所示：

| 项目 | 描述 | 端口号 |
| --- | --- | --- |
| `Northwind.Web` | ASP.NET Core Razor Pages 网站 | `5000 HTTP`，`5001 HTTPS` |
| `Northwind.Mvc` | ASP.NET Core MVC 网站 | `5000 HTTP`，`5001 HTTPS` |
| `Northwind.WebApi` | ASP.NET Core Web API 服务 | `5002 HTTPS`，`5008 HTTP` |
| `Minimal.WebApi` | ASP.NET Core Web API（最小） | `5003 HTTPS` |
| `Northwind.OData` | ASP.NET Core OData 服务 | `5004 HTTPS` |
| `Northwind.GraphQL` | ASP.NET Core GraphQL 服务 | `5005 HTTPS` |
| `Northwind.gRPC` | ASP.NET Core gRPC 服务 | `5006 HTTPS` |
| `Northwind.AzureFuncs` | Azure Functions nanoservice | `7071 HTTP` |

## 使用谷歌浏览器进行 HTTP 请求

让我们探索如何使用谷歌浏览器进行 HTTP 请求：

1.  启动谷歌浏览器。

1.  导航到**更多工具** | **开发者工具**。

1.  点击**网络**选项卡，Chrome 应立即开始记录浏览器和任何 Web 服务器之间的网络流量（请注意红色圆圈），如*图 14.2*所示：![](img/Image00105.jpg)

图 14.2：Chrome 开发者工具记录网络流量

1.  在 Chrome 的地址栏中，输入微软学习 ASP.NET 的网站地址，如下 URL 所示：

[`dotnet.microsoft.com/learn/aspnet`](https://dotnet.microsoft.com/learn/aspnet)

1.  在开发者工具中，在记录的请求列表中，滚动到顶部，点击第一个条目，**类型**为**文档**的行，如*图 14.3*所示：![](img/Image00106.jpg)

图 14.3：开发者工具中记录的请求

1.  在右侧，点击**头部**选项卡，您将看到有关**请求头**和**响应头**的详细信息，如*图 14.4*所示：![](img/Image00107.jpg)

图 14.4：请求和响应头

注意以下方面：

+   请求方法是`GET`。您在这里可能看到的其他 HTTP 方法包括`POST`，`PUT`，`DELETE`，`HEAD`和`PATCH`。

+   **状态码**是`200 OK`。这意味着服务器找到了浏览器请求的资源，并将其在响应的正文中返回。对于`GET`请求的响应中可能看到的其他状态码包括`301 Moved Permanently`，`400 Bad Request`，`401 Unauthorized`和`404 Not Found`。

+   浏览器发送到 Web 服务器的**请求头**包括：

+   **accept**，列出浏览器接受的格式。在这种情况下，浏览器表示它理解 HTML、XHTML、XML 和一些图像格式，但它将接受所有其他文件（`*/*`）。默认权重，也称为质量值，为`1.0`。XML 指定了质量值为`0.9`，因此它比 HTML 或 XHTML 更不受欢迎。所有其他文件类型的质量值为`0.8`，因此最不受欢迎。

+   **accept-encoding**，列出浏览器理解的压缩算法，例如 GZIP、DEFLATE 和 Brotli。

+   **accept-language**，列出它希望内容使用的人类语言。在这种情况下，美国英语具有默认质量值为`1.0`，然后是任何具有显式指定质量值为`0.9`的英语方言，然后是任何具有显式指定质量值为`0.8`的瑞典方言。

+   **响应头**，`content-encoding`告诉我服务器已经使用 GZIP 算法压缩了 HTML 网页响应，因为它知道客户端可以解压缩该格式。（这在*图 14.4*中不可见，因为没有足够的空间来展开**响应头**部分。）

1.  关闭 Chrome。

## 了解客户端 Web 开发技术

在构建网站时，开发人员需要了解的不仅仅是 C#和.NET。在客户端（即在 Web 浏览器中），您将使用以下技术的组合：

+   **HTML5**：用于网页的内容和结构。

+   **CSS3**：用于网页上元素的样式。

+   **JavaScript**：用于在网页上编写所需的任何业务逻辑，例如验证表单输入或调用 Web 服务以获取网页所需的更多数据。

尽管 HTML5、CSS3 和 JavaScript 是前端网页开发的基本组件，但还有许多其他技术可以使前端网页开发更加高效，包括 Bootstrap、全球最流行的前端开源工具包，以及用于样式的 CSS 预处理器，如 SASS 和 LESS，用于编写更健壮代码的 Microsoft 的 TypeScript 语言，以及诸如 jQuery、Angular、React 和 Vue 等 JavaScript 库。所有这些高级技术最终都会转换或编译成基础的三种核心技术，因此它们可以在所有现代浏览器上运行。

在构建和部署过程中，您可能会使用诸如 Node.js、Node Package Manager（npm）和 Yarn 等技术，它们都是客户端包管理器；以及 webpack，它是一个流行的模块打包工具，用于编译、转换和打包网站源文件。

# 理解 ASP.NET Core

Microsoft ASP.NET Core 是 Microsoft 技术发展历史的一部分，用于构建多年来不断发展的网站和服务：

+   **Active Server Pages**（**ASP**）于 1996 年发布，是微软首次尝试构建动态服务器端执行网站代码的平台。ASP 文件包含了在服务器上用 VBScript 语言编写的 HTML 和代码的混合。

+   **ASP.NET Web Forms**于 2002 年与.NET Framework 一起发布，旨在使非 Web 开发人员（如熟悉 Visual Basic 的人）通过拖放可视化组件并在 Visual Basic 或 C#中编写事件驱动代码来快速创建网站。对于新的.NET Framework Web 项目，应避免使用 Web Forms，而应使用 ASP.NET MVC。

+   **Windows Communication Foundation**（**WCF**）于 2006 年发布，使开发人员能够构建 SOAP 和 REST 服务。SOAP 功能强大但复杂，因此除非需要分布式事务和复杂的消息拓扑等高级功能，否则应避免使用。

+   **ASP.NET MVC**于 2009 年发布，以清晰地将 Web 开发人员的关注点分离开来，包括**模型**（临时存储数据）、**视图**（以各种格式在 UI 中呈现数据）和**控制器**（获取模型并将其传递给视图）。这种分离能够提高重用性和单元测试。

+   **ASP.NET Web API**于 2012 年发布，使开发人员能够创建比 SOAP 服务更简单和更可扩展的 HTTP 服务（也称为 REST 服务）。

+   **ASP.NET SignalR**于 2013 年发布，通过抽象底层技术和技巧（如 WebSockets 和长轮询）实现网站的实时通信。这使得网站可以实现实时聊天或对时间敏感数据（如股票价格）的更新，即使它们不支持底层技术（如 WebSockets）。

+   **ASP.NET Core**于 2016 年发布，结合了现代的.NET Framework 技术实现，如 MVC、Web API 和 SignalR，以及新技术，如 Razor Pages、gRPC 和 Blazor，都在现代.NET 上运行。因此，它可以跨平台执行。ASP.NET Core 有许多项目模板，可帮助您开始使用其支持的技术。

**良好实践**：选择 ASP.NET Core 来开发网站和服务，因为它包括现代且跨平台的 Web 相关技术。

ASP.NET Core 2.0 到 2.2 可以在.NET Framework 4.6.1 或更高版本（仅限 Windows）以及.NET Core 2.0 或更高版本（跨平台）上运行。ASP.NET Core 3.0 仅支持.NET Core 3.0。ASP.NET Core 6.0 仅支持.NET 6.0。

## 经典 ASP.NET 与现代 ASP.NET Core

到目前为止，ASP.NET 一直是建立在.NET Framework 中一个名为`System.Web.dll`的大型程序集之上，并且与微软的仅限于 Windows 的 Web 服务器**Internet Information Services**（**IIS**）紧密耦合。多年来，这个程序集积累了许多功能，其中许多不适合现代跨平台开发。

ASP.NET Core 是 ASP.NET 的一次重大重新设计。它消除了对`System.Web.dll`程序集和 IIS 的依赖，并由模块化的轻量级包组成，就像现代.NET 的其余部分一样。ASP.NET Core 仍然支持将 IIS 作为 Web 服务器，但有一个更好的选择。

您可以在 Windows、macOS 和 Linux 上跨平台开发和运行 ASP.NET Core 应用程序。微软甚至创建了一个跨平台、超高性能的 Web 服务器，名为**Kestrel**，整个堆栈都是开源的。

ASP.NET Core 2.2 或更高版本项目默认使用新的进程内托管模型。这在使用 Microsoft IIS 进行托管时可以提供 400%的性能提升，但微软仍建议使用 Kestrel 以获得更好的性能。

## 创建一个空的 ASP.NET Core 项目

我们将创建一个 ASP.NET Core 项目，该项目将显示来自 Northwind 数据库的供应商列表。

`dotnet`工具有许多项目模板可以为您做很多工作，但很难知道哪个最适合特定情况，因此我们将从空网站项目模板开始，然后逐步添加功能，以便您可以理解所有部分：

1.  使用您喜欢的代码编辑器添加一个新项目，如下列表所定义：

1.  项目模板：**ASP.NET Core Empty** / `web`

1.  语言：C#

1.  工作区/解决方案文件和文件夹：`PracticalApps`

1.  项目文件和文件夹：`Northwind.Web`

1.  对于 Visual Studio 2022，将所有其他选项保持为默认值，例如，选择**为 HTTPS 配置**，并清除**启用 Docker**。

1.  在 Visual Studio Code 中，选择`Northwind.Web`作为活动的 OmniSharp 项目。

1.  构建`Northwind.Web`项目。

1.  打开`Northwind.Web.csproj`文件，并注意项目类似于类库，只是 SDK 是`Microsoft.NET.Sdk.Web`，如下标记所示：

```cs

<Project Sdk="

**Microsoft.NET.Sdk.Web**

"

<PropertyGroup>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<ImplicitUsings>enable</ImplicitUsings>

</PropertyGroup>

</Project>

```

1.  如果您正在使用 Visual Studio 2022，在**Solution Explorer**中，切换**显示所有文件**。

1.  展开`obj`文件夹，展开`Debug`文件夹，展开`net6.0`文件夹，选择`Northwind.Web.GlobalUsings.g.cs`文件，并注意隐式导入的命名空间包括所有控制台应用程序或类库的命名空间，以及一些 ASP.NET Core 的命名空间，例如`Microsoft.AspNetCore.Builder`，如下面的代码所示：

```cs

// <autogenerated />

全局

使用

全局

::Microsoft.AspNetCore.Builder;

全局

使用

全局

::Microsoft.AspNetCore.Hosting;

全局

使用

全局

::Microsoft.AspNetCore.Http;

全局

使用

全局

::Microsoft.AspNetCore.Routing;

全局

使用

全局

::Microsoft.Extensions.Configuration;

全局

使用

全局

::Microsoft.Extensions.DependencyInjection;

全局

使用

全局

::Microsoft.Extensions.Hosting;

全局

使用

全局

::Microsoft.Extensions.Logging;

全局

使用

全局

::System;

全局

使用

全局

::System.Collections.Generic;

全局

使用

全局

::System.IO;

全局

使用

全局

::System.Linq;

全局

使用

全局

::System.Net.Http;

全局

使用

全局

::System.Net.Http.Json;

全局

使用

全局

::System.Threading;

全局

使用

全局

::System.Threading.Tasks;

```

1.  折叠`obj`文件夹。

1.  打开`Program.cs`，并注意以下内容：

+   ASP.NET Core 项目类似于顶级控制台应用程序，其隐藏的`Main`方法作为其入口点，其参数使用名称`args`传递。

+   它调用`WebApplication.CreateBuilder`，该方法使用默认值创建一个用于构建网站的主机。

+   网站将对所有 HTTP`GET`请求以纯文本`Hello World!`进行响应。

+   对`Run`方法的调用是一个阻塞调用，因此隐藏的`Main`方法直到 Web 服务器停止运行后才会返回，如下面的代码所示：

```cs

var

builder = WebApplication.CreateBuilder(args);

var

app = builder.Build();

app.MapGet("/"

, () => "Hello World!"

);

app.Run();

```

1.  在文件底部，添加一个语句，在调用`Run`方法之后以及 Web 服务器停止后向控制台写入消息，如下面代码中的突出显示部分所示：

```cs

app.Run();

**Console.WriteLine(**

**"这在 Web 服务器停止后执行！"**

**);**

```

## 测试和保护网站

我们现在将测试 ASP.NET Core 空网站项目的功能。我们还将通过从 HTTP 切换到 HTTPS 来启用浏览器和 Web 服务器之间所有流量的加密，以保护隐私。HTTPS 是 HTTP 的安全加密版本。

1.  对于 Visual Studio：

1.  在工具栏中，确保选择**Northwind.Web**而不是**IIS Express**或**WSL**，并将**Web 浏览器（Microsoft Edge）**切换到**Google Chrome**，如*图 14.5*所示：![](img/Image00108.jpg)

图 14.5：在 Visual Studio 中选择带有 Kestrel Web 服务器的 Northwind.Web 配置文件

1.  转到**调试**|**无调试启动…**。

1.  第一次启动安全网站时，您将收到提示，您的项目已配置为使用 SSL，并且为了避免浏览器中的警告，您可以选择信任 ASP.NET Core 生成的自签名证书。单击**是**。

1.  当您看到**安全警告**对话框时，再次单击**是**。

1.  对于 Visual Studio Code，在**终端**中，输入`dotnet run`命令。

1.  在 Visual Studio 的命令提示窗口或 Visual Studio Code 的终端中，注意 Kestrel Web 服务器已开始监听 HTTP 和 HTTPS 的随机端口，您可以按 Ctrl+C 关闭 Kestrel Web 服务器，并且托管环境是`开发`，如下面的输出所示：

```cs

信息：Microsoft.Hosting.Lifetime[14]

现在正在侦听：https://localhost:5001

信息：Microsoft.Hosting.Lifetime[14]

现在正在侦听：http://localhost:5000

信息：Microsoft.Hosting.Lifetime[0]

应用程序已启动。按 Ctrl+C 关闭。

信息：Microsoft.Hosting.Lifetime[0]

托管环境：开发

信息：Microsoft.Hosting.Lifetime[0]

内容根路径：C:\Code\PracticalApps\Northwind.Web

```

Visual Studio 还将自动启动您选择的浏览器。如果您使用的是 Visual Studio Code，则必须手动启动 Chrome。

1.  让 Web 服务器保持运行。

1.  在 Chrome 中，显示**开发人员工具**，并单击**网络**选项卡。

1.  输入地址`http://localhost:5000/`，或者分配给 HTTP 的任何端口号，并注意响应是纯文本`Hello World!`，来自跨平台 Kestrel Web 服务器，如*图 14.6*所示：![](img/Image00109.jpg)

图 14.6：从 http://localhost:5000/得到的纯文本响应

Chrome 还会自动请求`favicon.ico`文件以在浏览器标签中显示，但这个文件丢失了，所以会显示`404 Not Found`错误。

1.  输入地址`https://localhost:5001/`，或者分配给 HTTPS 的任何端口号，并注意如果您没有使用 Visual Studio，或者在提示是否信任 SSL 证书时点击了**否**，那么响应将是隐私错误，如*图 14.7*所示：![图形用户界面，应用程序描述自动生成](img/Image00110.jpg)

图 14.7：显示 SSL 加密未启用证书的隐私错误

当您尚未配置浏览器可以信任以加密和解密 HTTPS 流量的证书时，您将看到此错误（如果您没有看到此错误，那是因为您已经配置了证书）。

在生产环境中，您可能希望向 Verisign 等公司支付 SSL 证书，因为它们提供责任保护和技术支持。

**对于 Linux 开发人员**：如果您使用的 Linux 变体无法创建自签名证书，或者您不介意每 90 天重新申请新证书，那么您可以从以下链接获取免费证书：[`letsencrypt.org`](https://letsencrypt.org)

在开发过程中，您可以告诉操作系统信任由 ASP.NET Core 提供的临时开发证书。

1.  在命令行或**TERMINAL**中，按 Ctrl + C 关闭 Web 服务器，并注意所写的消息，如下面的输出中所示：

```cs

info: Microsoft.Hosting.Lifetime[0]

应用正在关闭...

**这在 Web 服务器停止后执行！**

C:\Code\PracticalApps\Northwind.Web\bin\Debug\net6.0\Northwind.Web.exe（进程 19888）退出，代码为 0。

```

1.  如果您需要信任本地自签名 SSL 证书，那么在命令行或**TERMINAL**中，输入`dotnet dev-certs https --trust`命令，并注意消息，**请求信任 HTTPS 开发证书**。您可能需要输入密码，有效的 HTTPS 证书可能已经存在。

### 启用更强大的安全性并重定向到安全连接

启用更严格的安全性并自动将 HTTP 请求重定向到 HTTPS 是一个好习惯。

**最佳实践**：**HTTP 严格传输安全性**（**HSTS**）是一种选择性的安全增强功能，您应该始终启用。如果网站指定了它并且浏览器支持它，那么它将强制所有通信通过 HTTPS，并防止访问者使用不受信任或无效的证书。

现在让我们这样做：

1.  在`Program.cs`中，添加一个`if`语句以在非开发环境中启用 HSTS，如下面的代码所示：

```cs

如果

（！app.Environment.IsDevelopment（））

{

app.UseHsts（）;

}

```

1.  在调用`app.MapGet`之前添加一个语句，将 HTTP 请求重定向到 HTTPS，如下面的代码所示：

```cs

app.UseHttpsRedirection();

```

1.  启动**Northwind.Web**网站项目。

1.  如果 Chrome 仍在运行，请关闭并重新启动它。

1.  在 Chrome 中，显示**开发者工具**，并单击**网络**选项卡。

1.  输入地址`http://localhost:5000/`，或者分配给 HTTP 的任何端口号，并注意服务器如何响应`307 临时重定向`到端口`5001`，并且证书现在有效和受信任，如*图 14.8*所示：！[](img/Image00111.jpg)

图 14.8：现在使用有效证书和 307 重定向进行了安全连接

1.  关闭 Chrome。

1.  关闭 Web 服务器。

**最佳实践**：记得在测试网站完成后关闭 Kestrel Web 服务器。

## 控制托管环境

在较早版本的 ASP.NET Core 中，项目模板设置了一条规则，即在开发模式下，任何未处理的异常都将显示在浏览器窗口中，供开发人员查看异常的详细信息，如下面的代码所示：

```cs

如果

（app.Environment.IsDevelopment（））

{

app.UseDeveloperExceptionPage（）;

}

```

使用 ASP.NET Core 6 及更高版本，默认情况下会自动执行此代码，因此不包括在项目模板中。

ASP.NET Core 如何知道我们何时处于开发模式，以便`IsDevelopment`方法返回`true`？让我们找出来。

ASP.NET Core 可以从环境变量中读取以确定要使用的托管环境，例如`DOTNET_ENVIRONMENT`或`ASPNETCORE_ENVIRONMENT`。

您可以在本地开发期间覆盖这些设置：

1.  在`Northwind.Web`文件夹中，展开名为`Properties`的文件夹，打开名为`launchSettings.json`的文件，并注意将托管环境设置为`Development`的配置文件名为`Northwind.Web`，如下面的配置所示：

```cs

{

"iisSettings"

：{

"windowsAuthentication"

：假

，

"anonymousAuthentication"

: true

,

"iisExpress"

: {

"applicationUrl"

: "http://localhost:56111"

,

"sslPort"

: 44329

}

},

"profiles"

: {

**"Northwind.Web"**

**: {**

**"commandName"**

**:**

**"Project"**

**,**

**"dotnetRunMessages"**

**:**

**"true"**

**,**

**"launchBrowser"**

**:**

**true**

**,**

**"applicationUrl"**

**:**

**"https://localhost:5001;http://localhost:5000"**

**,**

**"environmentVariables"**

**: {**

**"ASPNETCORE_ENVIRONMENT"**

**:**

**"Development"**

**}**

**},**

"IIS Express"

: {

"commandName"

: "IISExpress"

,

"launchBrowser"

: true

,

"environmentVariables"

: {

"ASPNETCORE_ENVIRONMENT"

: "Development"

}

}

}

}

```

1.  将 HTTP 的随机分配的端口号更改为`5000`，HTTPS 更改为`5001`。

1.  将环境更改为`Production`。可选地，将`launchBrowser`更改为`false`，以防止 Visual Studio 自动启动浏览器。

1.  启动网站，并注意托管环境为`Production`，如下所示：

```cs

info: Microsoft.Hosting.Lifetime[0]

托管环境：生产

```

1.  关闭 Web 服务器。

1.  在`launchSettings.json`中，将环境更改回`Development`。

`launchSettings.json` 文件还有一个配置，可以使用随机端口号将 IIS 作为 Web 服务器。在本书中，我们将只使用 Kestrel 作为 Web 服务器，因为它是跨平台的。

## 为服务和管道分离配置

将所有初始化简单 Web 项目的代码放在`Program.cs`中可能是一个好主意，特别是对于 Web 服务，所以我们将在*第十六章*中再次看到这种风格，*构建和使用 Web 服务*。

然而，对于任何不止是最基本的 Web 项目，您可能更喜欢将配置分离到一个名为`Startup`的单独类中，其中包含两种方法：

+   `ConfigureServices(IServiceCollection services)`: 向依赖注入容器中添加依赖服务，例如 Razor 页面支持，**跨源资源共享**（**CORS**）支持，或用于与 Northwind 数据库一起工作的数据库上下文。

+   `Configure(IApplicationBuilder app, IWebHostEnvironment env)`：设置 HTTP 管道，通过该管道请求和响应流动。在`app`参数上调用各种`Use`方法，以按照应该处理特性的顺序构建管道。![](img/Image00112.jpg)

图 14.9：Startup 类 ConfigureServices 和 Configure 方法图示

两种方法都将由运行时自动调用。

现在让我们创建一个`Startup`类：

1.  向`Northwind.Web`项目添加一个名为`Startup.cs`的新类文件。

1.  修改`Startup.cs`，如下所示：

```cs

namespace

Northwind.Web

;

public

类

Startup

{

public

void

ConfigureServices

(

IServiceCollection services

)

{

}

public

void

Configure

(

IApplicationBuilder app, IWebHostEnvironment env

)

{

if

(!env.IsDevelopment())

{

app.UseHsts();

}

app.UseRouting(); // 启动端点路由

app.UseHttpsRedirection();

app.UseEndpoints(endpoints =>

{

endpoints.MapGet("/"

, () => "Hello World!"

);

});

}

}

```

注意代码的以下内容：

+   `ConfigureServices`方法目前为空，因为我们还没有需要添加的任何依赖服务。

+   `Configure`方法设置了 HTTP 请求管道，并启用了端点路由的使用。它配置了一个路由端点，以等待使用相同的映射对每个 HTTP `GET`请求的根路径`/`进行响应，通过返回纯文本`"Hello World!"`来响应这些请求。我们将在本章末学习有关路由端点及其好处的知识。

现在，我们必须指定我们要在应用程序入口点中使用`Startup`类。

1.  修改`Program.cs`，如下所示：

```cs

使用

Northwind.Web; // Startup

Host.CreateDefaultBuilder(args)

.ConfigureWebHostDefaults(webBuilder =>

{

webBuilder.UseStartup<Startup>();

}).Build().Run();

Console.WriteLine("This executes after the web server has stopped!"

);

```

1.  启动网站，并注意它具有与以前相同的行为。

1.  关闭 Web 服务器。

在本书中创建的所有其他网站和服务项目中，我们将使用由.NET 6 项目模板创建的单个`Program.cs`文件。如果您喜欢`Startup.cs`的做事方式，那么您将在本章中看到如何使用它。

## 使网站能够提供静态内容

一个只返回单个纯文本消息的网站并不是很有用！

至少应该返回静态 HTML 页面、网页用于样式设计的 CSS 以及其他静态资源，如图像和视频。

按照惯例，这些文件应存储在名为`wwwroot`的目录中，以使它们与网站项目的动态执行部分分开。

### 创建静态文件和网页的文件夹

现在，您将为静态网站资源创建一个文件夹和一个使用 Bootstrap 进行样式设计的基本主页：

1.  在`Northwind.Web`项目/文件夹中，创建一个名为`wwwroot`的文件夹。

1.  在`wwwroot`文件夹中添加一个新的 HTML 页面文件，命名为`index.html`。

1.  修改其内容以链接到 CDN 托管的 Bootstrap 进行样式设计，并使用现代的良好实践，例如设置视口，如下面的标记所示：

```cs

<!doctype

html

<

html

lang

=

"en"

<

头

<!-- 必需的元标记 -->

<

meta

charset

=

"utf-8"

/>

<

meta

名称

=

"视口"

内容

=

"width=device-width, initial-scale=1 "

/>

<!-- Bootstrap CSS -->

<

链接

href

=

"https://cdn.jsdelivr.net/npm/bootstrap@5.1.0/dist/css/bootstrap.min.css"

rel

=

"stylesheet"

完整性

=

"sha384-KyZXEAg3QhqLMpG8r+8fhAXLRk2vvoC2f3B09zVXn8CA5QIVfZOJ3BCsw2P0p/We"

crossorigin

=

"anonymous"

<

标题

欢迎 ASP.NET Core！</

标题

</

头

<

身体

<

div

类

=

"container"

<

div

类

=

"jumbotron"

<

h1

类

=

"display-3"

欢迎来到 Northwind B2B</

h1

<

p

类

=

"引导"

我们向客户提供产品。</

p

<

hr

/>

<

h2

这是一个静态的 HTML 页面。</

h2

<

p

我们的客户包括餐馆、酒店和游轮公司。</

p

<

p

<

a

类

=

"btn btn-primary"

href

=

"https://www.asp.net/"

了解更多</

a

</

p

</

div

</

div

</

身体

</

html

```

**最佳实践**：要获取最新的 Bootstrap`<link>`元素，请从以下链接的文档中复制并粘贴：[`getbootstrap.com/docs/5.0/getting-started/introduction/#starter-template`](https://getbootstrap.com/docs/5.0/getting-started/introduction/#starter-template)。

### 启用静态和默认文件

如果您现在启动网站并在地址栏中输入`http://localhost:5000/index.html`，网站将返回一个`404 Not Found`错误，表示找不到网页。要使网站能够返回静态文件，如`index.html`，我们必须显式配置该功能。

即使我们启用了静态文件，如果您启动网站并在地址栏中输入`http://localhost:5000/`，网站将返回一个`404 Not Found`错误，因为 Web 服务器不知道在没有请求命名文件时默认返回什么。

现在，您将启用静态文件，显式配置默认文件，并更改注册的 URL 路径，以返回纯文本`Hello World!`响应：

1.  在`Startup.cs`中的`Configure`方法中，添加语句以启用 HTTPS 重定向后启用静态文件和默认文件，并修改将`GET`请求映射到返回`Hello World!`纯文本响应的语句，只响应 URL 路径`/hello`，如下面代码中突出显示的那样：

```cs

app.UseHttpsRedirection();

**app.UseDefaultFiles();**

**// index.html, default.html, and so on**

**app.UseStaticFiles();**

app.UseEndpoints(endpoints =>

{

endpoints.MapGet(

**"/hello"**

, () => "Hello World!"

);

});

```

对`UseDefaultFiles`的调用必须在对`UseStaticFiles`的调用之前，否则它将无法工作！您将在本章末尾了解有关中间件和端点路由顺序的更多信息。

1.  启动网站。

1.  启动**Chrome**并显示**开发者工具**。

1.  在 Chrome 中，输入`http://localhost:5000/`，注意到您被重定向到端口`5001`上的 HTTPS 地址，并且`index.html`文件现在通过安全连接返回，因为它是此网站的可能默认文件之一。

1.  在**开发人员工具**中，注意到对 Bootstrap 样式表的请求。

1.  在 Chrome 中，输入`http://localhost:5000/hello`，注意到它返回与以前相同的纯文本`Hello World!`。

1.  关闭 Chrome 并关闭 Web 服务器。

如果所有网页都是静态的，也就是说，它们只能通过网页编辑器手动更改，那么我们的网站编程工作就完成了。但几乎所有的网站都需要动态内容，这意味着通过执行代码在运行时生成的网页。

最简单的方法是使用 ASP.NET Core 的一个特性，名为**Razor Pages**。

# 探索 ASP.NET Core Razor 页面

ASP.NET Core Razor 页面允许开发人员轻松地将 C#代码语句与 HTML 标记混合在一起，使生成的网页动态化。这就是为什么它们使用`.cshtml`文件扩展名。

按照惯例，ASP.NET Core 在名为`Pages`的文件夹中查找 Razor 页面。

## 启用 Razor 页面

现在您将复制并将静态 HTML 页面更改为动态 Razor 页面，然后添加并启用 Razor 页面服务：

1.  在`Northwind.Web`项目文件夹中，创建一个名为`Pages`的文件夹。

1.  将`index.html`文件复制到`Pages`文件夹中。

1.  对于`Pages`文件夹中的文件，将文件扩展名从`.html`更改为`.cshtml`。

1.  删除说这是一个静态 HTML 页面的`<h2>`元素。

1.  在`Startup.cs`中，在`ConfigureServices`方法中，添加一个语句来添加 ASP.NET Core Razor 页面及其相关服务，如模型绑定、授权、防伪造、视图和标签助手，到构建器中，如下面的代码所示：

```cs

services.AddRazorPages();

```

1.  在`Startup.cs`中，在`Configure`方法中，在使用端点的配置中，添加一个语句来调用`MapRazorPages`方法，如下面的代码所示：

```cs

app.UseEndpoints(endpoints =>

{

**endpoints.MapRazorPages();**

endpoints.MapGet("/hello"

,  () => "Hello World!"

);

});

```

## 向 Razor 页面添加代码

在网页的 HTML 标记中，Razor 语法由`@`符号表示。Razor 页面可以描述如下：

+   它们需要在文件顶部添加`@page`指令。

+   它们可以选择性地具有定义以下任何内容的`@functions`部分：

+   用于存储数据值的属性，就像在类定义中一样。自动实例化该类的一个实例，命名为`Model`，可以在特殊方法中设置其属性，并且可以在 HTML 中获取属性值。

+   命名为`OnGet`、`OnPost`、`OnDelete`等的方法，当进行 HTTP 请求时执行，例如`GET`、`POST`和`DELETE`。

现在让我们将静态 HTML 页面转换为 Razor 页面：

1.  在`Pages`文件夹中，打开`index.cshtml`。

1.  在文件顶部添加`@page`语句。

1.  在`@page`语句之后，添加一个`@functions`语句块。

1.  定义一个属性来存储当前日期的名称作为`string`值。

1.  定义一个方法来设置`DayName`，当页面发出 HTTP `GET`请求时执行，如下面的代码所示：

```cs

@page

@functions

{

public

string

? DayName { get

; set

; }

public

void

OnGet

()

{

Model.DayName = DateTime.Now.ToString("dddd"

);

}

}

```

1.  在第二个 HTML 段落中输出日期名称，如下标记所示：

```cs

<

p

今天是@Model.DayName！

我们的客户包括餐馆、酒店和游轮公司。</

p

```

1.  启动网站。

1.  在 Chrome 中，输入`https://localhost:5001/`，注意到当前日期名称输出在页面上，如*图 14.10*所示：![](img/Image00113.jpg)

图 14.10：欢迎来到 Northwind 页面，显示当前日期

1.  在 Chrome 中，输入`https://localhost:5001/index.html`，它与静态文件名完全匹配，并且注意到它返回与以前相同的静态 HTML 页面。

1.  在 Chrome 中，输入`https://localhost:5001/hello`，这与返回纯文本的端点路由完全匹配，并注意它像以前一样返回`Hello World!`。

1.  关闭 Chrome 并关闭 Web 服务器。

## 使用 Razor 页面的共享布局

大多数网站都有多个页面。如果每个页面都必须包含当前在`index.cshtml`中的所有样板标记，那将变得难以管理。因此，ASP.NET Core 有一个名为**layouts**的功能。

要使用布局，我们必须创建一个 Razor 文件来定义所有 Razor 页面（和所有 MVC 视图）的默认布局，并将其存储在`Shared`文件夹中，以便可以轻松地按照约定找到它。此文件的名称可以是任何名称，因为我们将指定它，但`_Layout.cshtml`是一个很好的做法。

我们还必须创建一个特别命名的文件来设置所有 Razor 页面（和所有 MVC 视图）的默认布局文件。此文件必须命名为`_ViewStart.cshtml`。

让我们看看布局的实际效果：

1.  在`Pages`文件夹中，添加一个名为`_ViewStart.cshtml`的文件。（Visual Studio 项目模板名为**Razor View Start**。）

1.  修改其内容，如下面的标记所示：

```cs

@{

Layout = "_Layout"

;

}

```

1.  在`Pages`文件夹中，创建一个名为`Shared`的文件夹。

1.  在`Shared`文件夹中，创建一个名为`_Layout.cshtml`的文件。（Visual Studio 项目模板名为**Razor Layout**。）

1.  修改`_Layout.cshtml`的内容（它类似于`index.cshtml`，因此您可以从那里复制和粘贴 HTML 标记），如下面的标记所示：

```cs

<!doctype

html

<

html

lang

=

"en"

<

head

<!-- Required meta tags -->

<

meta

charset

=

"utf-8"

/>

<

meta

name

=

"viewport"

content

=

"width=device-width, initial-scale=1, shrink-to-fit=no"

/>

<!-- Bootstrap CSS -->

<

link

href

=

"https://cdn.jsdelivr.net/npm/bootstrap@5.1.0/dist/css/bootstrap.min.css"

rel

=

"stylesheet"

integrity

=

"sha384-KyZXEAg3QhqLMpG8r+8fhAXLRk2vvoC2f3B09zVXn8CA5QIVfZOJ3BCsw2P0p/We"

crossorigin

=

"anonymous"

<

title

@ViewData["Title"

]</

title

</

head

<

body

<

div

class

=

"container"

@RenderBody()

<

hr

/>

<

footer

<

p

Copyright &copy;

2021 - @ViewData["Title"

]</

p

</

footer

</

div

<!-- JavaScript to enable features like carousel -->

<

script

src

=

"https://cdn.jsdelivr.net/npm/bootstrap@5.1.0/dist/js/bootstrap.bundle.min.js"

integrity

=

"sha384-U1DAWAznBHeqEIlVSCgzq+c9gqGAJn5c/t99JyeKa9xxaYpSvHU5awsuZVVFIhvj"

crossorigin

=

"anonymous"

></

script

@RenderSection("Scripts"

, required: false

)

</

body

</

html

```

在审查上述标记时，请注意以下内容：

+   `<title>`是使用来自名为`ViewData`的字典的服务器端代码动态设置的。这是在 ASP.NET Core 网站的不同部分之间传递数据的简单方法。在这种情况下，数据将在 Razor 页面类文件中设置，然后在共享布局中输出。

+   `@RenderBody()`标记了视图请求的插入点。

+   每个页面底部都会出现一个水平线和页脚。

+   在布局底部有一个脚本，用于实现 Bootstrap 的一些很酷的功能，我们以后可以使用，比如图像的轮播。

+   在 Bootstrap 的`<script>`元素之后，我们定义了一个名为`Scripts`的部分，以便 Razor Page 可以选择性地注入它需要的其他脚本。

1.  修改`index.cshtml`，除了`<div class="jumbotron">`及其内容之外，删除所有 HTML 标记，并保留您之前添加的`@functions`块中的 C#代码。

1.  在`OnGet`方法中添加一个语句，将页面标题存储在`ViewData`字典中，并修改按钮以导航到供应商页面（我们将在下一节中创建），如下面标记中所示：

```cs

@page

@functions

{

public

string

? DayName { get

; set

; }

public

void

OnGet()

{

**ViewData[**

**"Title"**

**] =**

**"Northwind B2B"**

**;**

Model.DayName = DateTime.Now.ToString("dddd"

);

}

}

<

div

class

=

"jumbotron"

<

h1

class

=

"display-3"

欢迎来到 Northwind B2B</

h1

<

p

class

=

"lead"

我们向客户提供产品。</

p

<

hr

/>

<

p

它是@Model.DayName！我们的客户包括餐馆、酒店和游轮公司。</

p

<

p

**<**

**a**

**类**

**=**

**“btn btn-primary”**

**href**

**=**

**“供应商”**

**>**

**了解更多关于我们的供应商**

**</**

**a**

**>**

</

p

</

div

```

1.  启动网站，使用 Chrome 访问，并注意它与以前的行为类似，尽管单击供应商按钮会给出`404 Not Found`错误，因为我们还没有创建该页面。

## 使用 Razor Pages 的 code-behind 文件

有时，将 HTML 标记与数据和可执行代码分开是更好的，因此 Razor Pages 允许您通过将 C#代码放在**code-behind**类文件中来实现这一点。它们与`.cshtml`文件同名，但以`.cshtml.cs`结尾。

现在，您将创建一个显示供应商列表的页面。在这个例子中，我们专注于学习代码后台文件。在下一个主题中，我们将从数据库加载供应商列表，但现在，我们将用硬编码的`string`值数组来模拟：

1.  在`Pages`文件夹中，添加两个名为`Suppliers.cshtml`和`Suppliers.cshtml.cs`的新文件。（Visual Studio 项目模板命名为**Razor Page - Empty**，它会创建这两个文件。）

1.  在名为`Suppliers.cshtml.cs`的 code-behind 文件中添加语句，如下所示：

```cs

使用

Microsoft.AspNetCore.Mvc.RazorPages; // PageModel

namespace

Northwind.Web.Pages

;

public

类

SuppliersModel

: PageModel

{

public

IEnumerable<string

>? Suppliers { get

; set

; }

public

void

OnGet

()

{

ViewData["Title"

] = “Northwind B2B - 供应商”

;

Suppliers = new

[]

{

“Alpha Co”

，“Beta 有限公司”

，“Gamma Corp”

};

}

}

```

在审查前面的标记时，请注意以下内容：

+   `SuppliersModel`继承自`PageModel`，因此它具有诸如用于共享数据的`ViewData`字典之类的成员。您可以右键单击`PageModel`，然后选择**转到定义**，以查看它还有更多有用的功能，比如当前请求的整个`HttpContext`。

+   `SuppliersModel`定义了一个用于存储名为`Suppliers`的`string`值集合的属性。

+   当为此 Razor 页面发出 HTTP`GET`请求时，`Suppliers`属性将从`string`值数组中的一些示例供应商名称中填充。稍后，我们将从 Northwind 数据库填充这个属性。

1.  修改`Suppliers.cshtml`的内容，如下标记所示：

```cs

@page

@model Northwind.Web.Pages.SuppliersModel

<

div

类

=

“row”

<

h1

类

=

“display-2”

供应商</

h1

<

table

类

=

“表”

<

thead

类

=

“thead-inverse”

<

tr

><

th

公司名称</

th

></

tr

</

thead

<

tbody

@if (Model.Suppliers is not null)

{

@foreach(string name in Model.Suppliers)

{

<

tr

><

td

@name</

td

></

tr

}

}

</

tbody

</

table

</

div

```

在审查前面的标记时，请注意以下内容：

+   此 Razor 页面的模型类型设置为`SuppliersModel`。

+   该页面输出一个带有 Bootstrap 样式的 HTML 表格。

+   表中的数据行是通过循环访问`Model`的`Suppliers`属性来生成的，如果它不是`null`。

1.  启动网站并使用 Chrome 访问。

1.  单击按钮了解更多有关供应商的信息，并注意供应商表，如*图 14.11*所示：![](img/Image00114.jpg)

图 14.11：从字符串数组加载的供应商表

# 使用 Entity Framework Core 与 ASP.NET Core

Entity Framework Core 是将真实数据导入网站的一种自然方式。在*第十三章*，*介绍 C#和.NET 的实际应用*中，您创建了两对类库：一个用于实体模型，一个用于 Northwind 数据库上下文，用于 SQL Server 或 SQLite 或两者。现在，您将在您的网站项目中使用它们。

## 配置 Entity Framework Core 作为服务

像 ASP.NET Core 所需的 Entity Framework Core 数据库上下文这样的功能必须在网站启动期间注册为服务。GitHub 存储库解决方案中的代码以及下面的代码使用了 SQLite，但如果您愿意，您也可以轻松地使用 SQL Server。

让我们看看：

1.  在`Northwind.Web`项目中，为 SQLite 或 SQL Server 添加对`Northwind.Common.DataContext`项目的项目引用，如下面的标记所示：

```cs

<!-- 如果要更改为 SQL Server，请将 Sqlite 更改为 SqlServer

您更喜欢的话 -->

<ItemGroup>

<ProjectReference Include="..\Northwind.Common.DataContext.Sqlite\

Northwind.Common.DataContext.Sqlite.csproj"

/>

</ItemGroup>

```

项目引用必须在一行上，不能换行。

1.  构建`Northwind.Web`项目。

1.  在`Startup.cs`中，导入命名空间以使用您的实体模型类型，如下面的代码所示：

```cs

使用

Packt.Shared; // AddNorthwindContext 扩展方法

```

1.  在`ConfigureServices`方法中添加一个语句来注册`Northwind`数据库上下文类，如下面的代码所示：

```cs

services.AddNorthwindContext();

```

1.  在`Northwind.Web`项目中，在`Pages`文件夹中，打开`Suppliers.cshtml.cs`，并导入我们数据库上下文的命名空间，如下面的代码所示：

```cs

使用

Packt.Shared; // NorthwindContext

```

1.  在`SuppliersModel`类中，添加一个私有字段来存储`Northwind`数据库上下文，并添加一个构造函数来设置它，如下面的代码所示：

```cs

私人

NorthwindContext db;

公共

SuppliersModel

(

注入的 NorthwindContext

)

{

db = injectedContext;

}

```

1.  将`Suppliers`属性更改为包含`Supplier`对象而不是`string`值。

1.  在`OnGet`方法中，修改语句以从数据库上下文的`Suppliers`属性中设置`Suppliers`属性，按国家和公司名称排序，如下面代码中所示：

```cs

公共

void

OnGet

()

{

ViewData["Title"

] = "Northwind B2B - 供应商"

;

Suppliers =

**db.Suppliers**

**.OrderBy(c => c.Country).ThenBy(c => c.CompanyName)**

;

}

```

1.  修改`Suppliers.cshtml`的内容，以导入`Packt.Shared`命名空间，并为每个供应商呈现多个列，如下面的标记所示：

```cs

@page

**@using Packt.Shared**

@model Northwind.Web.Pages.SuppliersModel

<

div

类

=

"row"

<

h1

类

=

"display-2"

供应商</

h1

<

表

类

=

"table"

<

thead

类

=

"thead-inverse"

<

tr

<

th

公司名称</

th

**<**

**th**

**>**

**国家**

**</**

**th**

**>**

**<**

**th**

**>**

**Phone**

**</**

**th**

**>**

</

tr

</

thead

<

tbody

@if（Model.Suppliers 不为空）

{

**@foreach(Supplier s in Model.Suppliers)**

{

<

tr

**<**

**td**

**>**

**@s.CompanyName**

**</**

**td**

**>**

**<**

**td**

**>**

**@s.Country**

**</**

**td**

**>**

**<**

**td**

**>**

**@s.Phone**

**</**

**td**

**>**

</

tr

}

}

</

tbody

</

table

</

div

```

1.  启动网站。

1.  在 Chrome 中，输入`https://localhost:5001/`。

1.  点击**了解有关我们供应商的更多信息**，并注意供应商表现在从数据库加载，如*图 14.12*所示：![](img/Image00115.jpg)

图 14.12：从 Northwind 数据库加载的供应商表

## 使用 Razor Pages 操纵数据

现在，您将添加插入新供应商的功能。

### 启用模型以插入实体

首先，您将修改供应商模型，以便在访问者提交表单以插入新供应商时，它会响应 HTTP `POST`请求：

1.  在`Northwind.Web`项目中，在`Pages`文件夹中，打开`Suppliers.cshtml.cs`，并导入以下命名空间：

```cs

使用

Microsoft.AspNetCore.Mvc; // [BindProperty]，IActionResult

```

1.  在`SuppliersModel`类中，添加一个属性来存储单个供应商，并添加一个名为`OnPost`的方法，如果其模型有效，则将供应商添加到 Northwind 数据库中的`Suppliers`表中，如下面的代码所示：

```cs

[BindProperty

]

公共

供应商？ 供应商 { 获取

; set

; }

公共

IActionResult

OnPost

()

{

if

((供应商是

不

null

）&& ModelState.IsValid)

{

db.Suppliers.Add(Supplier);

db.SaveChanges();

返回

RedirectToPage("/suppliers"

);

}

否则

{

返回

Page(); // 返回原始页面

}

}

```

在审查上述代码时，请注意以下内容：

+   我们添加了一个名为`Supplier`的属性，该属性带有`[BindProperty]`属性修饰，以便我们可以轻松地将 Web 页面上的 HTML 元素连接到`Supplier`类中的属性。

+   我们添加了一个响应 HTTP`POST`请求的方法。它检查所有属性值是否符合`Supplier`类实体模型上的验证规则（如`[Required]`和`[StringLength]`），然后将供应商添加到现有表中，并保存对数据库上下文的更改。这将生成一个 SQL 语句来执行插入到数据库中。然后它重定向到`Suppliers`页面，以便访问者看到新添加的供应商。

### 定义一个插入新供应商的表单

接下来，您将修改 Razor 页面以定义一个访问者可以填写并提交以插入新供应商的表单：

1.  在`Suppliers.cshtml`中，在`@model`声明之后添加标签助手，以便我们可以在 Razor 页面上使用`asp-for`等标签助手，如下面的标记所示：

```cs

@addTagHelper *，Microsoft.AspNetCore.Mvc.TagHelpers

```

1.  在文件底部，添加一个表单来插入一个新的供应商，并使用`asp-for`标签助手将`Supplier`类的`CompanyName`，`Country`和`Phone`属性绑定到输入框，如下面的标记所示：

```cs

<

div

类

=

"行"

<

p

输入新供应商的详细信息：</

p

<

表单

方法

=

"POST"

<

div

><

输入

asp-for

=

"Supplier.CompanyName"

占位符

=

"公司名称"

/></

div

<

div

><

输入

asp-for

=

"Supplier.Country"

占位符

=

"国家"

/></

div

<

div

><

输入

asp-for

=

"Supplier.Phone"

占位符

=

"电话"

/></

div

<

输入

类型

=

"submit"

/>

</

形式

</

div

```

在审查上述标记时，请注意以下内容：

+   `<form>`元素使用`POST`方法是普通的 HTML，因此在其中的`<input type="submit" />`元素将使 HTTP`POST`请求返回到当前页面，并携带该表单中任何其他元素的值。

+   带有名为`asp-for`的标签助手的`<input>`元素使数据绑定到 Razor 页面背后的模型。

1.  启动网站。

1.  点击**了解更多关于我们的供应商**，滚动到页面底部，输入`Bob's Burgers`，`USA`和`(603) 555-4567`，然后点击**提交**。

1.  请注意，您会看到一个刷新后的供应商表，其中包含新添加的供应商。

1.  关闭 Chrome 并关闭 Web 服务器。

## 将依赖服务注入到 Razor 页面中

如果您有一个没有代码后台文件的`.cshtml` Razor 页面，那么您可以使用`@inject`指令注入一个依赖服务，而不是构造函数参数注入，然后直接在标记的中间使用 Razor 语法引用注入的数据库上下文。

让我们创建一个简单的例子：

1.  在`Pages`文件夹中，添加一个名为`Orders.cshtml`的新文件。（Visual Studio 项目模板名为**Razor Page - Empty**，它会创建两个文件。删除`.cs`文件。）

1.  在`Orders.cshtml`中，编写代码以输出 Northwind 数据库中订单的数量，如下面的标记所示：

```cs

@page

@using Packt.Shared

@inject NorthwindContext db

@{

string title = "Orders"

;

ViewData["Title"

] = $"Northwind B2B -

{title}

"

;

}

<

div

类

=

"行"

<

h1

类

=

"display-2"

@title</

h1

<

p

Northwind 数据库中有@db.Orders.Count()个订单。

</

p

</

div

```

1.  启动网站。

1.  导航到`/orders`，注意您会看到 Northwind 数据库中有 830 个订单。

1.  关闭 Chrome 并关闭 Web 服务器。

# 使用 Razor 类库

与 Razor 页面相关的所有内容都可以编译到一个类库中，以便在多个项目中更轻松地重用。在 ASP.NET Core 3.0 及更高版本中，这可能包括静态文件，如 HTML、CSS、JavaScript 库和媒体资产，如图像文件。网站可以使用类库中定义的 Razor 页面视图，也可以覆盖它。

## 创建 Razor 类库

让我们创建一个新的 Razor 类库：

使用您喜欢的代码编辑器添加一个新项目，如下面的列表所定义：

1.  项目模板：**Razor 类库** / `razorclasslib`

1.  复选框/开关：**支持页面和视图** / `-s`

1.  工作区/解决方案文件和文件夹：`PracticalApps`

1.  项目文件和文件夹：`Northwind.Razor.Employees`

`-s`是`--support-pages-and-views`开关的缩写，该开关使类库能够使用 Razor 页面和`.cshtml`文件视图。

## 禁用 Visual Studio Code 的紧凑文件夹

在我们实现 Razor 类库之前，我想解释一下 Visual Studio Code 的一个功能，这个功能让上一版的一些读者感到困惑，因为这个功能是在发布后添加的。

紧凑文件夹功能意味着如果层次结构中的中间文件夹不包含文件，则嵌套文件夹（例如`/Areas/MyFeature/Pages/`）以紧凑形式显示，如*图 14.13*所示：

![](img/Image00116.jpg)

图 14.13：启用或禁用紧凑文件夹

如果您想要禁用 Visual Studio Code 的紧凑文件夹功能，请完成以下步骤：

1.  在 Windows 上，转到**文件** | **首选项** | **设置**。在 macOS 上，转到**Code** | **首选项** | **设置**。

1.  在**搜索**设置框中，输入`compact`。

1.  清除**资源管理器：紧凑文件夹**复选框，如*图 14.14*所示：![图形用户界面，文本，应用程序，电子邮件描述自动生成](img/Image00117.jpg)

图 14.14：禁用 Visual Studio Code 的紧凑文件夹

1.  关闭**设置**选项卡。

## 实现员工功能使用 EF Core

现在我们可以添加对实体模型的引用，以在 Razor 类库中显示员工：

1.  在`Northwind.Razor.Employees`项目中，为 SQLite 或 SQL Server 添加对`Northwind.Common.DataContext`项目的项目引用，并注意 SDK 是`Microsoft.NET.Sdk.Razor`，如下面的标记所示：

```cs

<Project Sdk="

**Microsoft.NET.Sdk.Razor**

"

<PropertyGroup>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

隐式使用>启用</ImplicitUsings>

<AddRazorSupportForMvc>true

</AddRazorSupportForMvc>

</PropertyGroup>

<ItemGroup>

<FrameworkReference Include="Microsoft.AspNetCore.App"

/>

</ItemGroup>

**<!-- 将 Sqlite 更改为 SqlServer**

**如果**

**你喜欢-->**

**<ItemGroup>**

**<ProjectReference Include=**

**"..\Northwind.Common.DataContext.Sqlite**

**\Northwind.Common.DataContext.Sqlite.csproj"**

**/>**

**</ItemGroup>**

</Project>

```

项目引用必须在一行上，不能换行。同时不要混合我们的 SQLite 和 SQL Server 项目，否则会看到编译器错误。如果在`Northwind.Web`项目中使用了 SQL Server，则在`Northwind.Razor.Employees`项目中也必须使用 SQL Server。

1.  构建`Northwind.Razor.Employees`项目。

1.  在`Areas`文件夹中，右键单击`MyFeature`文件夹，选择**重命名**，输入新名称`PacktFeatures`，然后按 Enter。

1.  在`PacktFeatures`文件夹中，在`Pages`子文件夹中，添加一个名为`_ViewStart.cshtml`的新文件。（Visual Studio 项目模板命名为**Razor View Start**。或者从`Northwind.Web`项目中复制。）

1.  修改其内容以通知该类库，任何 Razor 页面都应该寻找与`Northwind.Web`项目中使用的相同名称的布局，如下面的标记所示：

```cs

@{

Layout = "_Layout"

;

}

```

我们不需要在这个项目中创建`_Layout.cshtml`文件。它将使用其宿主项目中的文件，例如`Northwind.Web`项目中的文件。

1.  在`Pages`子文件夹中，将`Page1.cshtml`重命名为`Employees.cshtml`，并将`Page1.cshtml.cs`重命名为`Employees.cshtml.cs`。

1.  修改`Employees.cshtml.cs`以定义一个页面模型，其中包含从 Northwind 数据库加载的`Employee`实体实例数组，如下面的代码所示：

```cs

使用

Microsoft.AspNetCore.Mvc.RazorPages; // PageModel

使用

Packt.Shared; // Employee, NorthwindContext

命名空间

PacktFeatures.Pages

;

公共

类

EmployeesPageModel

: PageModel

{

私人的

NorthwindContext db;

公共

EmployeesPageModel

(

NorthwindContext injectedContext

)

{

db = injectedContext;

}

公共

Employee[] Employees { get

; set

; } = null

!;

公共

void

OnGet

()

{

ViewData["Title"

] = "Northwind B2B - Employees"

;

Employees = db.Employees.OrderBy(e => e.LastName)

.ThenBy(e => e.FirstName).ToArray();

}

}

```

1.  修改`Employees.cshtml`，如下面的标记所示：

```cs

@page

@using Packt.Shared

@addTagHelper *，Microsoft.AspNetCore.Mvc.TagHelpers

@model PacktFeatures.Pages.EmployeesPageModel

<

div

类

=

"row"

<

h1

类

=

"display-2"

Employees</

h1

</

div

<

div

类

=

"row"

@foreach(Employee employee in Model.Employees)

{

<

div

类

=

"col-sm-3"

<

partial

名称

=

"_Employee"

模型

=

"employee"

/>

</

div

}

</

div

```

在审查前面的标记时，请注意以下内容：

+   我们导入`Packt.Shared`命名空间，以便我们可以使用其中的类，如`Employee`。

+   我们添加了对标签助手的支持，以便我们可以使用`<partial>`元素。

+   我们声明了这个 Razor Page 的`@model`类型，以使用您刚刚定义的页面模型类。

+   我们枚举模型中的`Employees`，使用局部视图输出每个员工。

## 实现一个局部视图来显示单个员工

`<partial>`标签助手是在 ASP.NET Core 2.1 中引入的。局部视图就像 Razor Page 的一部分。您将在接下来的几个步骤中创建一个，以渲染一个单独的员工：

1.  在`Northwind.Razor.Employees`项目的`Pages`文件夹中，创建一个`Shared`文件夹。

1.  在`Shared`文件夹中，创建一个名为`_Employee.cshtml`的文件。（Visual Studio 项目模板的名称为**Razor View - Empty**。）

1.  修改`_Employee.cshtml`，如下面的标记所示：

```cs

@model Packt.Shared.Employee

<

div

类

=

"card border-dark mb-3"

样式

=

"max-width: 18rem;"

<

div

类

=

"card-header"

@Model?.LastName, @Model?.FirstName</

div

<

div

类

=

"card-body text-dark"

<

h5

类

=

"card-title"

@Model?.Country</

h5

<

p

类

=

"card-text"

@Model?.Notes</

p

</

div

</

div

```

在审查前面的标记时，请注意以下内容：

+   按照惯例，局部视图的名称以下划线开头。

+   如果将局部视图放在`Shared`文件夹中，它将自动找到。

+   这个局部视图的模型类型是单个`Employee`实体。

+   我们使用 Bootstrap 卡片样式来输出有关每个员工的信息。

## 使用和测试 Razor 类库

现在您将在网站项目中引用并使用 Razor 类库：

1.  在`Northwind.Web`项目中，添加对`Northwind.Razor.Employees`项目的项目引用，如下面的标记所示：

```cs

<ProjectReference Include=

"..\Northwind.Razor.Employees\Northwind.Razor.Employees.csproj"

/>

```

1.  修改`Pages\index.cshtml`，在供应商页面的链接之后添加一个链接到 Packt 功能员工页面的段落，如下面的标记所示：

```cs

<

p

<

a

类

=

"btn btn-primary"

href

=

"packtfeatures/employees"

联系我们的员工

</

a

</

p

```

1.  启动网站，使用 Chrome 访问网站，并单击**联系我们的员工**按钮，以查看员工的卡片，如*图 14.15*所示：![](img/Image00118.jpg)

图 14.15：Razor 类库功能的员工列表

# 配置服务和 HTTP 请求管道

现在我们已经建立了一个网站，我们可以返回到`Startup`配置，并更详细地审查服务和 HTTP 请求管道的工作方式。

## 理解端点路由

在早期版本的 ASP.NET Core 中，路由系统和可扩展的中间件系统并不总是很容易地协同工作；例如，如果您想要在中间件和 MVC 中都实现 CORS 等策略。微软已经投资于通过 ASP.NET Core 2.2 引入的名为**端点路由**的系统来改进路由。

**良好实践**：如果可能的话，端点路由取代了在 ASP.NET Core 2.1 及更早版本中使用的基于`IRouter`的路由。微软建议每个较旧的 ASP.NET Core 项目尽可能迁移到端点路由。

端点路由旨在实现需要路由的框架（如 Razor Pages、MVC 或 Web API）和需要了解路由如何影响它们的中间件（如本地化、授权、CORS 等）之间更好的互操作性。

端点路由得名是因为它将路由表表示为端点的编译树，可以被路由系统高效地遍历。最大的改进之一是路由和动作方法选择的性能。

如果兼容性设置为 2.2 或更高版本，则默认情况下在 ASP.NET Core 2.2 或更高版本中启用。使用`MapRoute`方法或属性注册的传统路由将映射到新系统。

新的路由系统包括一个链接生成服务，注册为不需要`HttpContext`的依赖服务。

### 配置端点路由

端点路由需要对`UseRouting`和`UseEndpoints`方法进行一对调用：

+   `UseRouting`标记了管道位置，其中进行路由决策。

+   `UseEndpoints`标记了管道位置，其中执行选定的端点。

中间件，如本地化，运行在这些方法之间，可以看到选定的端点，并在必要时切换到不同的端点。

端点路由使用自 2010 年以来在 ASP.NET MVC 中使用的相同路由模板语法，以及 2013 年引入的`[Route]`属性。迁移通常只需要对`Startup`配置进行更改。

MVC 控制器、Razor Pages 和诸如 SignalR 之类的框架以前是通过调用`UseMvc`或类似的方法启用的，但现在它们都被添加到`UseEndpoints`方法调用中，因为它们都集成到同一个路由系统中，以及中间件。

## 审查项目中的端点路由配置

请查看以下代码中显示的`Startup.cs`类文件：

```cs

using

Packt.Shared; // AddNorthwindContext 扩展方法

namespace

Northwind.Web

;

public

class

Startup

{

public

void

ConfigureServices

(

IServiceCollection services

)

{

services.AddRazorPages();

services.AddNorthwindContext();

}

public

void

Configure

(

IApplicationBuilder app, IWebHostEnvironment env

)

{

if

(!env.IsDevelopment())

{

app.UseHsts();

}

app.UseRouting();

app.UseHttpsRedirection();

app.UseDefaultFiles(); // index.html, default.html 等

app.UseStaticFiles();

app.UseEndpoints(endpoints =>

{

endpoints.MapRazorPages();

endpoints.MapGet("/hello"

, () => "Hello World!"

);

});

}

}

```

`Startup`类有两个方法，由主机自动调用来配置网站。

`ConfigureServices`方法注册服务，然后可以使用依赖注入在需要它们提供功能时检索它们。我们的代码注册了两个服务：Razor Pages 和一个 EF Core 数据库上下文。

### 在 ConfigureServices 方法中注册服务

在下表中显示了注册依赖服务的常见方法，包括结合其他注册服务的方法调用的服务。

| 方法 | 注册的服务 |
| --- | --- |
| `AddMvcCore` | 路由请求和调用控制器所需的最小服务集。大多数网站将需要比这更多的配置。 |
| `AddAuthorization` | 身份验证和授权服务。 |
| `AddDataAnnotations` | MVC 数据注释服务。 |
| `AddCacheTagHelper` | MVC 缓存标签助手服务。 |
| `AddRazorPages` | Razor Pages 服务，包括 Razor 视图引擎。通常用于简单的网站项目。它调用以下附加方法：`AddMvcCore``AddAuthorization``AddDataAnnotations``AddCacheTagHelper` |
| `AddApiExplorer` | Web API 资源管理器服务。 |
| `AddCors` | 用于增强安全性的 CORS 支持。 |
| `AddFormatterMappings` | URL 格式与其对应的媒体类型之间的映射。 |
| `AddControllers` | 控制器服务，但不包括视图或页面服务。通常用于 ASP.NET Core Web API 项目。它调用以下附加方法：`AddMvcCore``AddAuthorization``AddDataAnnotations``AddCacheTagHelper``AddApiExplorer``AddCors``AddFormatterMappings` |
| `AddViews` | 支持`.cshtml`视图，包括默认约定。 |
| `AddRazorViewEngine` | 支持 Razor 视图引擎，包括处理`@`符号。 |
| `AddControllersWithViews` | 控制器、视图和页面服务。通常用于 ASP.NET Core MVC 网站项目。它调用以下附加方法：`AddMvcCore``AddAuthorization``AddDataAnnotations``AddCacheTagHelper``AddApiExplorer``AddCors``AddFormatterMappings``AddViews``AddRazorViewEngine` |
| `AddMvc` | 类似于`AddControllersWithViews`，但您应该仅在向后兼容性时使用它。 |
| `AddDbContext<T>` | 您的`DbContext`类型及其可选的`DbContextOptions<TContext>`。 |
| `AddNorthwindContext` | 我们创建的一个自定义扩展方法，使得更容易为项目引用的 SQLite 或 SQL Server 注册`NorthwindContext`类。 |

在接下来的几章中，您将看到更多使用这些扩展方法来注册服务的示例，这些服务是与 MVC 和 Web API 服务一起使用的。

### 在 Configure 方法中设置 HTTP 请求管道

`Configure`方法配置了 HTTP 请求管道，它由一系列连接的委托组成，可以执行处理，然后决定是自己返回响应还是将处理传递给管道中的下一个委托。返回的响应也可以被操纵。

请记住，委托定义了委托实现可以插入的方法签名。HTTP 请求管道的委托很简单，如下面的代码所示：

```cs

public

委托

Task

RequestDelegate

(

HttpContext 上下文

)

;

```

您可以看到输入参数是`HttpContext`。这提供了访问处理传入的 HTTP 请求所需的一切，包括 URL 路径、查询字符串参数、cookie 和用户代理。

这些委托通常被称为中间件，因为它们位于浏览器客户端和网站或服务之间。

中间件委托是使用以下方法之一或调用它们本身的自定义方法进行配置的：

+   `Run`：添加一个中间件委托，立即返回响应而不调用下一个中间件委托，从而终止管道。

+   `Map`：添加一个中间件委托，当有匹配的请求时（通常基于 URL 路径如`/hello`），它会在管道中创建一个分支。

+   `Use`：添加一个中间件委托，作为管道的一部分，因此它可以决定是否将请求传递给管道中的下一个委托，并且可以在下一个委托之前和之后修改请求和响应。

为了方便起见，有许多扩展方法可以更轻松地构建管道，例如`UseMiddleware<T>`，其中`T`是一个具有以下内容的类：

1.  带有`RequestDelegate`参数的构造函数，该参数将传递给下一个管道组件

1.  带有`HttpContext`参数并返回`Task`的`Invoke`方法

## 总结关键中间件扩展方法

我们代码中使用的关键中间件扩展方法包括以下内容：

+   `UseDeveloperExceptionPage`：从管道中捕获同步和异步的`System.Exception`实例，并生成 HTML 错误响应。

+   `UseHsts`：添加使用 HSTS 的中间件，添加`Strict-Transport-Security`头。

+   `UseRouting`：添加中间件，定义了管道中的一个点，路由决策在此处做出，并且必须与调用`UseEndpoints`结合使用，然后在此处执行处理。这意味着对于我们的代码，任何匹配`/`或`/index`或`/suppliers`的 URL 路径都将映射到 Razor 页面，而匹配`/hello`的 URL 路径将映射到匿名委托。任何其他 URL 路径将被传递给下一个委托进行匹配，例如静态文件。这就是为什么，尽管在管道中看起来 Razor 页面和`/hello`的映射发生在静态文件之后，但它们实际上具有优先级，因为在`UseStaticFiles`之前调用`UseRouting`。

+   `UseHttpsRedirection`：添加中间件以将 HTTP 请求重定向到 HTTPS，因此在我们的代码中，对`http://localhost:5000`的请求将被修改为`https://localhost:5001`。

+   `UseDefaultFiles`：添加中间件，启用当前路径上的默认文件映射，因此在我们的代码中，它将识别诸如`index.html`之类的文件。

+   `UseStaticFiles`：添加中间件，查找`wwwroot`中的静态文件以在 HTTP 响应中返回。

+   `UseEndpoints`：添加中间件以执行从管道中较早做出的决策生成响应。如下所示，添加了两个端点，如下面的子列表所示：

+   `MapRazorPages`：添加中间件，将诸如`/suppliers`之类的 URL 路径映射到`/Pages`文件夹中名为`suppliers.cshtml`的 Razor 页面文件，并将结果作为 HTTP 响应返回。

+   `MapGet`：添加中间件，将诸如`/hello`之类的 URL 路径映射到直接向 HTTP 响应写入纯文本的内联委托。

## 可视化 HTTP 管道

HTTP 请求和响应管道可以被可视化为一系列请求委托，依次调用，如下面的简化图表所示，其中排除了一些中间件委托，例如`UseHsts`：

![自动生成的图表描述](img/Image00119.jpg)

图 14.16：HTTP 请求和响应管道

如前所述，必须一起使用`UseRouting`和`UseEndpoints`方法。虽然定义映射路由的代码（如`/hello`）写在`UseEndpoints`中，但关于传入的 HTTP 请求 URL 路径是否匹配以及因此要执行哪个端点的决定是在管道中的`UseRouting`点上做出的。

## 实现匿名内联委托作为中间件

可以指定内联匿名方法作为委托。我们将注册一个插入到管道中的委托，用于在端点的路由决策之后。

它将输出选择的端点，以及处理一个特定路由：`/bonjour`。如果匹配该路由，它将以纯文本形式响应，而不会进一步调用管道：

1.  在`Startup.cs`中，静态导入`Console`，如下面的代码所示：

```cs

使用

静态

System.Console;

```

1.  在调用`UseRouting`之后和调用`UseHttpsRedirection`之前添加语句以使用匿名方法作为中间件委托，如下面的代码所示：

```cs

app.Use(async

(HttpContext context, Func<Task> next) =>

{

RouteEndpoint? rep = context.GetEndpoint() as

RouteEndpoint;

如果

(rep is

不是

null

)

{

WriteLine($"端点名称：

{rep.DisplayName}

"

);

WriteLine($"端点路由模式：

{rep.RoutePattern.RawText}

"

);

}

如果

(context.Request.Path == "/bonjour"

)

{

// 在 URL 路径匹配的情况下，这将成为终止`

// 返回的委托，因此不调用下一个委托

等待

context.Response.WriteAsync("Bonjour Monde!"

);

返回

;

}

// 在调用下一个委托之前，我们可以修改请求

等待

next();

// 在调用下一个委托之后，我们可以修改响应

});

```

1.  启动网站。

1.  在 Chrome 中，导航到`https://localhost:5001/`，查看控制台输出，并注意在端点路由`/`上有一个匹配，它被处理为`/index`，并且执行了`Index.cshtml` Razor Page 以返回响应，如下面的输出所示：

```cs

端点名称：/index

端点路由模式：

```

1.  导航到`https://localhost:5001/suppliers`，注意到在端点路由`/Suppliers`上有一个匹配，并且执行了`Suppliers.cshtml` Razor Page 以返回响应，如下面的输出所示：

```cs

端点名称：/Suppliers

端点路由模式：供应商

```

1.  导航到`https://localhost:5001/index`，注意到在端点路由`/index`上有一个匹配，并且执行了`Index.cshtml` Razor Page 以返回响应，如下面的输出所示：

```cs

端点名称：/index

端点路由模式：索引

```

1.  导航到`https://localhost:5001/index.html`，注意到控制台没有输出，因为在端点路由上没有匹配，但是有一个静态文件的匹配，因此它作为响应返回。

1.  导航到`https://localhost:5001/bonjour`，注意到控制台没有输出，因为在端点路由上没有匹配。相反，我们的委托匹配了`/bonjour`，直接写入响应流，并且没有进一步处理就返回了。

1.  关闭 Chrome 并关闭 Web 服务器。

# 练习和探索

通过回答一些问题来测试您的知识和理解，进行一些实践，并深入研究本章的主题。

## 练习 14.1–测试您的知识

回答以下问题：

1.  列出六个可以在 HTTP 请求中指定的方法名。

1.  列出六个状态代码及其在 HTTP 响应中的描述。

1.  在 ASP.NET Core 中，`Startup`类用于什么？

1.  HSTS 是什么意思，它是做什么的？

1.  如何为网站启用静态 HTML 页面？

1.  如何将 C#代码混合到 HTML 中间以创建动态页面？

1.  如何为 Razor Pages 定义共享布局？

1.  你如何在 Razor Page 中将标记与代码分离？

1.  如何配置用于 ASP.NET Core 网站的 Entity Framework Core 数据上下文？

1.  如何在 ASP.NET Core 2.2 或更高版本中重用 Razor Pages？

## 练习 14.2–练习构建数据驱动的网页

向`Northwind.Web`网站添加一个 Razor Page，使用户可以查看按国家分组的客户列表。当用户点击客户记录时，他们将看到显示该客户的完整联系信息以及其订单列表的页面。

## 练习 14.3–练习为控制台应用程序构建网页

重新实现一些早期章节的控制台应用程序，例如*第四章*，*编写、调试和测试函数*，为其提供一个 Web 用户界面来输出乘法表、计算税收，并生成阶乘和斐波那契数列。

## 练习 14.4–探索主题

使用以下页面上的链接了解本章涵盖的主题：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-14---building-websites-using-aspnet-core-razor-pages`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-14---building-websites-using-aspnet-core-razor-pages)

# 摘要

在本章中，您学习了使用 HTTP 进行 Web 开发的基础知识，如何构建返回静态文件的简单网站，并且使用 ASP.NET Core Razor Pages 与 Entity Framework Core 创建了从数据库中动态生成的网页。

我们回顾了 HTTP 请求和响应管道，辅助扩展方法的作用，以及如何添加自己的中间件来影响处理过程。

在下一章中，您将学习如何使用 ASP.NET Core MVC 构建更复杂的网站，该框架将构建网站的技术问题分离为模型、视图和控制器，使其更易于管理。
