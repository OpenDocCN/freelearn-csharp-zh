# *第五章*: 配置 WebHostBuilder

当你在 *第四章*，*使用 Kestrel 配置和自定义 HTTPS* 中阅读时，你可能自己问过一个问题：

*我如何使用用户密钥来将密码传递给 HTTPS 配置？*

你甚至可能会想知道是否可以从 `Program.cs` 中获取配置。

在本章中，我们将通过以下主题来回答这些问题：

+   重新审视 `WebHostBuilderContext`

本章中的主题指的是 ASP.NET Core 架构的托管层：

![图 5.1 – ASP.NET Core 架构](img/Figure_5.1_B17996.jpg)

图 5.1 – ASP.NET Core 架构

# 技术要求

要遵循本章中的描述，你需要创建一个 **ASP.NET Core** 应用程序。为此，打开你的控制台、shell 或 Bash 终端，切换到你的工作目录。然后，使用以下命令创建一个新的 Web 应用程序：

```cs
dotnet new web -n HostBuilderConfig -o HostBuilderConfig
```

现在通过双击项目文件或在 Visual Studio Code 中在已打开的控制台中输入以下命令来在 Visual Studio 中打开项目：

```cs
cd HostBuilderConfig
code .
```

注意

在 .NET 6.0 中，简单的 `web` 项目模板发生了变化。在 6.0 版本中，Microsoft 引入了 **最小 API** 并将项目模板更改为使用最小 API 方法。我将在本章中向您展示这些模板之间的差异。

本章中的所有代码示例都可以在本书的 **GitHub** 仓库中找到，网址为 [`github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/tree/main/Chapter05`](https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/tree/main/Chapter05)。

# 重新审视 WebHostBuilderContext

记得我们在 *第四章*，*使用 Kestrel 配置和自定义 HTTPS* 中查看的 `WebHostBuilder` Kestrel 配置吗？在那个章节中，我们看到了你应该使用用户密钥来配置证书密码，如下面的代码片段所示：

```cs
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }
    public static IHostBuilder CreateHostBuilder(
        string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder
                    .UseKestrel(options =>
                    {
                        options.Listen(
                            IPAddress.Loopback, 
                            5000);
                        options.Listen(
                            IPAddress.Loopback, 
                            5001, 
                            listenOptions  =>
                            {
                                listenOptions.UseHttps(
                                    "certificate.pfx", 
                                    "topsecret");
                            });
                    })
                    .UseStartup<Startup>();
            });
    }
}
```

这个片段对于 .NET 5.0 及之前的版本仍然有效，对于几乎所有的 .NET 6.0 中的 Web 项目也仍然有效。但是，它不适用于我们在技术要求部分使用的 `web` 项目模板。最小 API 的 `Program.cs` 文件如下所示：

```cs
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();
app.MapGet("/", () => "Hello World!");
app.Run();
```

这意味着我们在上一章中实现的配置在最小 API 中看起来是这样的：

```cs
using System.Net;
var builder = WebApplication.CreateBuilder(args);
builder.WebHost.UseKestrel(options =>
{
    options.Listen(IPAddress.Loopback, 5000);
    options.Listen(IPAddress.Loopback, 5001,
        listenOptions =>
        {
            listenOptions.UseHttps(
                "certificate.pfx",
                "topsecret");
        });
});
```

以下适用于 .NET 6.0 及之前版本的所有项目模板。

通常，你无法在这个代码中获取配置。你需要知道 `UseKestrel()` 方法是重载的，就像你在这里看到的那样：

```cs
.UseKestrel((host, options) =>
{
    // ...
})
```

第一个参数是一个 `WebHostBuilderContext` 实例，你可以使用它来访问配置。所以，让我们稍微修改一下 lambda 表达式来使用这个上下文：

```cs
builder.WebHost.UseKestrel((host, options) =>
{
    var filename = host.Configuration.GetValue(
        "AppSettings:certfilename", "");
    var password = host.Configuration.GetValue(
        "AppSettings:certpassword", "");
    options.Listen(IPAddress.Loopback, 5000);
    options.Listen(IPAddress.Loopback, 5001, 
        listenOptions =>
        {
            listenOptions.UseHttps(filename, password);
        });
});
```

在此示例中，我们使用冒号分隔符写入密钥，因为这是从 `appsettings.json` 读取嵌套配置的方式：

```cs
{
    "AppSettings": {
        "certfilename": "certificate.pfx",
        "certpassword": "topsecret"
    },
    "Logging": {
        "LogLevel": {
            "Default": "Warning"
        }
    },
    "AllowedHosts": "*"
}
```

重要提示

这只是如何读取配置以配置 **Kestrel** 的一个示例。请永远不要，*永远不要* 在代码中存储任何凭据。相反，使用以下概念。

您还可以从用户密钥存储中读取，其中密钥是通过以下 **.NET** **CLI** 命令设置的，您需要在项目文件夹中执行这些命令：

```cs
dotnet user-secrets init
dotnet user-secrets set "AppSettings:certfilename" 
  "certificate.pfx"
dotnet user-secrets set "AppSettings:certpassword" 
  "topsecret"
```

这也适用于环境变量：

```cs
SET APPSETTINGS_CERTFILENAME=certificate.pfx
SET APPSETTINGS_CERTPASSWORD=topsecret
```

重要提示

由于用户密钥存储仅用于本地开发，因此您应在生产环境中通过环境变量将凭据传递给应用程序，或者类似生产的应用程序。此外，您在 Azure 中创建的应用程序设置配置将作为环境变量传递给您的应用程序。

那么，这一切是如何工作的呢？

## 它是如何工作的？

您还记得那些需要设置 ASP.NET Core 1.0 中 `Startup.cs` 文件中的应用程序配置的日子吗？它们是在 `StartUp` 类的构造函数中配置的，如果您添加了用户密钥，它们看起来类似于此：

```cs
var builder = new ConfigurationBuilder()
    .SetBasePath(env.ContentRootPath)
    .AddJsonFile("appsettings.json")
    .AddJsonFile($"appsettings.{env.EnvironmentName}.json",
      optional:  true);
if (env.IsDevelopment())
{
    builder.AddUserSecrets();
}
builder.AddEnvironmentVariables();
Configuration = builder.Build();
```

此代码现在被封装在 `CreateDefaultBuilder` 方法中（如您在 GitHub 上所见 – 详细信息请参阅 *进一步阅读* 部分），其外观如下：

```cs
builder.ConfigureAppConfiguration((hostingContext, config) 
  =>
{
    var env = hostingContext.HostingEnvironment;
    config
        .AddJsonFile(
            "appsettings.json", 
            optional: true, 
            reloadOnChange: true)
        .AddJsonFile(
            $"appsettings.{env.EnvironmentName}.json", 
            optional:  true,
            reloadOnChange: true);
    if (env.IsDevelopment())
    {
        var appAssembly = Assembly.Load(
            new AssemblyName(env.ApplicationName));
        if (appAssembly != null)
        {
            config.AddUserSecrets(appAssembly, 
                optional: true);
        }
    }
    config.AddEnvironmentVariables();
    if (args != null)
    {
        config.AddCommandLine(args);
    }
});
```

这几乎与之前的代码相同，并且是构建 `WebHost` 时最早执行的事情之一。

这需要是我们设置的第一件事之一，因为 Kestrel 可以通过应用程序配置进行配置。您可以通过使用环境变量或 `appsettings.json` 来指定端口、URL 等。

您可以在 GitHub 上的 `WebHost.cs` 文件中找到这些行：

```cs
builder.UseKestrel((builderContext, options) =>
  {
    options.Configure(
        builderContext.Configuration.GetSection("Kestrel"));
})
```

这意味着您可以将这些行添加到 `appsettings.json` 中来配置 Kestrel 端点：

```cs
"Kestrel": {
    "EndPoints": {
        "Http": {
            "Url": "http://localhost:5555"
        }
    }
}
```

或者，可以使用以下环境变量来配置端点：

```cs
SET KESTREL_ENDPOINTS_HTTP_URL=http://localhost:5555
```

让我们现在回顾一下本章中我们涵盖的所有内容。

# 概述

在 `Program.cs` 中，如果您有权访问 `WebHostBuilderContext`，则可以在配置方法的 lambda 表达式中进行应用程序配置。这样，您可以使用所有喜欢的配置来配置 `WebHostBuilder`。

在下一章中，我们将查看托管细节。您将了解不同的托管模型以及如何以不同的方式托管 ASP.NET Core 应用程序。

# 进一步阅读

ASP.NET GitHub 仓库中的 `WebHost.cs` 文件：[`github.com/aspnet/MetaPackages/blob/d417aacd7c0eff202f7860fe1e686aa5beeedad7/src/Microsoft.AspNetCore/WebHost.cs`](https://github.com/aspnet/MetaPackages/blob/d417aacd7c0eff202f7860fe1e686aa5beeedad7/src/Microsoft.AspNetCore/WebHost.cs).
