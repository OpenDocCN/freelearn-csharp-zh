# *第四章*：使用 Kestrel 配置和自定义 HTTPS

在 **ASP.NET** **Core** 中，**HTTPS** 默认启用，并且是一个一等特性。在 Windows 上，启用 HTTPS 所需的证书是从 Windows 证书存储中加载的。如果你在 **Linux** 或 **Mac** 上创建项目，证书是从证书文件中加载的。

即使你想创建一个在 **IIS** 或 **NGINX** 网络服务器后面运行的项目，HTTPS 也会被启用。通常情况下，你会在 IIS 或 NGINX 网络服务器上管理证书。然而，在这里启用 HTTPS 不会成为问题，所以不要在 ASP.NET Core 设置中禁用它。

如果你在防火墙后面运行服务，例如不可从互联网访问的服务，如基于微服务的应用程序的后台服务，或者自托管的 ASP.NET Core 应用程序中的服务，直接在 ASP.NET Core 应用程序中管理证书是有意义的。

在 Windows 上，也有一些场景下从文件中加载证书是有意义的。这可能是你将在 **Docker** for Windows 或 Linux 上运行的应用程序中。我个人喜欢从文件中灵活加载证书的方式。

本简短章节将涵盖两个主题：

+   介绍 Kestrel

+   设置 Kestrel

本章中的主题涉及 ASP.NET Core 架构的托管层：

![图 4.1 – ASP.NET Core 架构](img/Figure_4.1_B17996.jpg)

图 4.1 – ASP.NET Core 架构

# 技术要求

要遵循本章的描述，你需要创建一个 ASP.NET Core MVC 应用程序。为此，打开你的控制台、shell 或 Bash 终端，切换到你的工作目录。然后，使用以下命令创建一个新的 MVC 应用程序：

```cs
dotnet new mvc -n HttpSample -o HttpSample
```

现在，通过双击项目文件在 Visual Studio 中打开项目，或在 Visual Studio Code 中通过在已打开的控制台中输入以下命令来打开项目：

```cs
cd HttpSample
code .
```

本章中的所有代码示例都可以在本书的 **GitHub** 仓库中找到，网址为 [`github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/tree/main/Chapter04`](https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/tree/main/Chapter04)。

# 介绍 Kestrel

**Kestrel** 是一个新实现的 HTTP 服务器，它是 ASP.NET Core 的托管引擎。每个 ASP.NET Core 应用程序都会在 Kestrel 服务器上运行。经典的 ASP.NET 应用程序（在 **.NET** **Framework** 上运行）通常直接在 IIS 上运行。在 ASP.NET Core 中，微软受到了 **Node.js** 的启发，它也提供了一个名为 **libuv** 的 HTTP 服务器。在 ASP.NET Core 的第一个版本中，微软也使用了 libuv，然后它添加了一个名为 Kestrel 的层。当时，Node.js 和 ASP.NET Core 共享同一个 HTTP 服务器。

由于.NET Core 框架已经发展，并且在其上实现了**.NET 套接字**，因此微软基于.NET 套接字构建了自己的 HTTP 服务器，并移除了 libuv，这是一个他们不拥有和控制依赖项。现在，Kestrel 是一个功能齐全的 HTTP 服务器，可以运行 ASP.NET Core 应用程序。

IIS 充当反向代理，将流量转发到 Kestrel 并管理 Kestrel 进程。在 Linux 上，通常使用 NGINX 作为 Kestrel 的反向代理。

# 设置 Kestrel

正如我们在本书的前两章中所做的那样，我们需要稍微覆盖默认的`WebHostBuilder`来设置 Kestrel。从 ASP.NET Core 3.0 开始，可以替换默认的 Kestrel 基础配置为自定义配置。这意味着 Kestrel 网络服务器被配置到主机构建器。让我们看看设置的步骤：

1.  您只需使用它，就可以手动添加和配置 Kestrel。以下代码展示了在`IwebHostBuilder`上调用`UseKestrel()`方法时会发生什么。现在让我们看看这是如何与`CreateWebHostBuilder`方法结合的：

    ```cs
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateWebHostBuilder(args).Build().Run();
        }
        public static IHostBuilder 
          CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder
                        .UseKestrel(options =>
                        {
                        })
                        .UseStartup<Startup>();
                }
    }
    ```

    上述代码显示了直到 ASP.NET Core 5.0 的`Program.cs`看起来如何。在 ASP.NET Core 6.0 中，使用新的最小 API 方法来配置您的应用程序：

    ```cs
    var builder = WebApplication.CreateBuilder(args);
    builder.WebHost.UseKestrel(options =>
    {
    });
    // Add services to the container.
    builder.Services.AddControllersWithViews();
    var app = builder.Build();
    // the rest of this file is not relevant
    ```

    我们将在本章的剩余部分专注于`UseKestrel()`方法。`UseKestrel()`方法接受一个操作来配置 Kestrel 网络服务器。

1.  我们实际上需要做的是配置网络服务器监听的地址和端口。对于 HTTPS 端口，我们还需要配置证书应该如何加载：

    ```cs
    builder.WebHost.UseKestrel(options =>
    {
        options.Listen(IPAddress.Loopback, 5000);
        options.Listen(IPAddress.Loopback,  5001, 
          listenOptions  =>
        {
            listenOptions.UseHttps("certificate.pfx", 
              "topsecret");
        });
    });
    ```

    不要忘记添加一个`using`语句到`System.Net`命名空间以解析`IPAddress`。

    在这个片段中，我们添加了要监听的地址和端口。配置被定义为使用 HTTPS 的安全端点。`UseHttps()`方法被多次重载，以便从 Windows 证书存储以及从文件中加载证书。在这种情况下，我们将使用位于项目文件夹中的名为`certificate.pfx`的文件。

1.  要创建一个证书文件来玩这个配置，请打开证书存储并导出 Visual Studio 创建的开发证书。它位于当前用户证书下的个人证书中：

![图片](img/Figure_4.2_B17996.jpg)

图 4.2 – 证书

右键单击此条目。在上下文菜单中，转到**所有任务**并单击**导出**。在**证书导出向导**中，单击**下一步**然后单击**是，导出私钥**，然后单击**下一步**。现在，在下一屏幕中选择**.PFX**格式并单击**下一步**。在这里，您需要设置一个密码。这就是您在代码中需要使用的密码，如以下代码示例所示。选择一个文件名和一个存储文件的位置，然后单击**下一步**。最后一个屏幕将显示摘要。单击**完成**将证书保存到文件。

## 为了您的安全

只使用以下行来尝试这个配置：

```cs
listenOptions.UseHttps("certificate.pfx", "topsecret");
```

为了阐明原因——问题是硬编码的密码。永远，**永远**不要将密码存储在会被推送到任何源代码仓库的代码文件中。请确保您从 ASP.NET Core 的配置 API 中加载密码。在您的本地开发机器上使用用户密钥，在服务器上使用环境变量。在 **Azure** 上，使用应用程序设置来存储密码。如果将它们标记为密码，密码将在 Azure 门户 UI 中被隐藏。

# 摘要

这只是一个小的定制，但如果您想在不同的平台之间共享代码，或者想在 Docker 上运行应用程序而不想担心证书存储等问题，它应该会有所帮助。

通常，如果您在 IIS 或 NGINX 等网络服务器后面运行应用程序，您不需要关心您的 ASP.NET Core 6.0 应用程序中的证书。然而，如果您在另一个应用程序内部、Docker 上或没有 IIS 或 NGINX 的情况下托管应用程序，您将需要这样做。

在下一章中，我们将讨论如何配置 ASP.NET Core 网络应用程序的托管。
