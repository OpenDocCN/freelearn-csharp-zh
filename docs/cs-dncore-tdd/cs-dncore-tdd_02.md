# 第二章：开始使用.NET Core

当微软发布第一个版本的.NET Framework 时，这是一个创建、运行和部署服务和应用程序的平台，它改变了游戏规则，是微软开发社区的一场革命。使用初始版本的框架开发了几个尖端应用程序，然后发布了几个版本。

多年来，.NET Framework 得到了蓬勃发展和成熟，支持多种编程语言，并包含了多个功能，使得在该平台上编程变得简单而有价值。但是，尽管框架非常强大和吸引人，但限制了开发和部署应用程序只能在微软操作系统变体上进行。

为了为开发人员解决.NET Framework 的限制，创建一个面向云的、跨平台的.NET Framework 实现，微软开始使用.NET Framework 开发.NET Core 平台。随着 2016 年版本 1.0 的推出，.NET 平台的应用程序开发进入了一个新的维度，因为.NET 开发人员现在可以轻松地构建在 Windows、Linux、macOS 和云、嵌入式和物联网设备上运行的应用程序。.NET Core 与.NET Framework、Xamarin 和 Mono 兼容，通过.NET 标准。

本章将介绍.NET Core 和 C# 7 的超酷新跨平台功能。我们将在 Ubuntu Linux 上使用 TDD 创建一个 ASP.NET MVC 应用程序来学习。在本章中，我们将涵盖以下主题：

+   .NET Core 框架

+   .NET Core 应用程序的结构

+   微软的 Visual Studio Code 编辑器之旅

+   C# 7 的新功能一览

+   创建 ASP.NET MVC Core 应用程序

# .NET Core 框架

**.NET Core**是一个跨平台的开源开发框架，可以在 Windows、Linux 和 macOS 上运行，并支持 x86、x64 和 ARM 架构。.NET Core 是从.NET Framework 分叉出来的，从技术上讲，它是后者的一个子集，尽管是简化的、模块化的。.NET Core 是一个开发平台，可以让您在开发和部署应用程序时拥有很大的灵活性。新平台使您摆脱了通常在应用程序部署过程中遇到的麻烦。因此，您不必担心在部署服务器上管理应用程序运行时的版本。

目前，版本 2.0.7 中，.NET Core 包括具有出色性能和许多功能的.NET 运行时。微软声称这是最快的.NET 平台版本。它有更多的 API 和更多的项目模板，比如用于在.NET Core 上运行的 ReactJS 和 AngularJS 应用程序的模板。此外，版本 2.0.7 还有一组命令行工具，使您能够在不同平台上轻松构建和运行命令行应用程序，以及简化的打包和对 Macintosh 上的 Visual Studio 的支持。.NET Core 的一个重要副产品是跨平台模块化 Web 框架 ASP.NET Core，它是 ASP.NET 的全面重新设计，并在.NET Core 上运行。

.NET Framework 非常强大，并包含多个库用于应用程序开发。然而，一些框架的组件和库可能与 Windows 操作系统耦合。例如，`System.Drawing`库依赖于 Windows GDI，这就是为什么.NET Framework 不能被认为是跨平台的，尽管它有不同的实现。

为了使.NET Core 真正跨平台，像 Windows Forms 和**Windows Presentation Foundation**（**WPF**）这样对 Windows 操作系统有很强依赖的组件已经从平台中移除。ASP.NET Web Forms 和**Windows Communication Foundation**（**WCF**）也已被移除，并用 ASP.NET Core MVC 和 ASP.NET Core Web API 替代。此外，**Entity Framework**（**EF**）已经被简化，使其跨平台，并命名为 Entity Framework Core。

此外，由于.NET Framework 对 Windows 操作系统的依赖，微软无法开放源代码。然而，.NET Core 是完全开源的，托管在 GitHub 上，并拥有一个不断努力开发新功能和扩展平台范围的蓬勃发展的开发者社区。

# .NET 标准

**.NET 标准**是微软维护的一组规范和标准，所有.NET 平台都必须遵循和实现。它正式规定了所有.NET 平台变体都应该实现的 API。目前.NET 平台上有三个开发平台—.NET Core、.NET Framework 和 Xamarin。.NET 平台需要提供统一性和一致性，使得在这三个.NET 平台变体上更容易共享代码和重用库。

.NET 平台提供了一组统一的基类库 API 的定义，所有.NET 平台都必须实现，以便开发人员可以轻松地在.NET 平台上开发应用程序和可重用库。目前的版本是 2.0.7，.NET 标准提供了新的 API，这些 API 在.NET Core 1.0 中没有实现，但现在在 2.0 版本中已经实现。超过 20,000 个 API 已经添加到运行时组件中。

此外，.NET 标准是一个目标框架，这意味着你可以开发你的应用程序以针对特定版本的.NET 标准，使得应用程序可以在实现该标准的任何.NET 平台上运行，并且你可以轻松地在不同的.NET 平台之间共享代码、库和二进制文件。当构建应用程序以针对.NET 标准时，你应该知道较高版本的.NET 标准有更多可用的 API，但并不是许多平台都实现了。建议你始终针对较低版本的标准，这将保证它被许多平台实现：

![](img/79979ff9-3e9a-489c-ab1d-65ae71a535f1.png)

# .NET 核心组件

.NET Core 作为通用应用程序开发平台，由**CoreCLR**、**CoreFX**、**SDK 和 CLI 工具**、**应用程序主机**和**dotnet 应用程序启动器**组成：

![](img/fa1eaeb2-5f73-4cc8-8f01-bfb2c3b84a79.png)

CoreCLR，也称为.NET Core 运行时，是.NET Core 的核心，是 CLR 的跨平台实现；原始的.NET Framework CLR 已经重构为 CoreCLR。CoreCLR，即公共语言运行时，管理对象的使用和引用，不同编程语言中的对象的通信和交互，并通过在对象不再使用时释放内存来执行垃圾收集。CoreCLR 包括以下内容：

+   垃圾收集器

+   **即时**（**JIT**）编译器

+   本地互操作

+   .NET 基本类型

CoreFX 是.NET Core 的一组框架或基础库，它提供原始数据类型、文件系统、应用程序组合类型、控制台和基本实用工具。CoreFX 包含了一系列精简的类库。

.NET Core SDK 包含一组工具，包括**命令行界面**（**CLI**）工具和编译器，用于构建应用程序和库在.NET Core 上运行。SDK 工具和语言编译器提供功能，通过 CoreFX 库支持的语言组件，使编码更加简单和快速。

为了启动一个.NET Core 应用程序，dotnet 应用程序主机是负责选择和托管应用程序所需运行时的组件。.NET Core 有控制台应用程序作为主要应用程序模型，以及其他应用程序模型，如 ASP.NET Core、Windows 10 通用 Windows 平台和 Xamarin Forms。

# 支持的语言

.NET Core 1.0 仅支持**C#**和**F#**，但随着.NET Core 2.0 的发布，**VB.NET**现在也受到了平台的支持。支持的语言的编译器在.NET Core 上运行，并提供对平台基础功能的访问。这是可能的，因为.NET Core 实现了.NET 标准规范，并公开了.NET Framework 中可用的 API。支持的语言和.NET SDK 工具可以集成到不同的编辑器和 IDE 中，为您提供不同的编辑器选项，用于开发应用程序。

# 何时选择.NET Core 而不是.NET Framework

.NET Core 和.NET Framework 都非常适合用于*开发健壮和可扩展的企业应用程序*；这是因为这两个平台都建立在坚实的代码基础上，并提供了丰富的库和例程，简化了大多数开发任务。这两个平台共享许多相似的组件，因此可以在两个开发平台之间共享代码。然而，这两个平台是不同的，选择.NET Core 作为首选的开发平台应受开发方法以及部署需求和要求的影响。

# 跨平台要求

显然，当您开发的应用程序要在多个平台上运行时，应该使用.NET Core。由于.NET Core 是跨平台的，因此适用于开发可以在**Windows**、**Linux**和**macOS**上运行的服务和 Web 应用程序。此外，微软推出了**Visual Studio Code**，这是一个具有对.NET Core 的全面支持的编辑器，提供智能感知和调试功能，以及传统上仅在**Visual Studio IDE**中可用的其他 IDE 功能。

# 部署的便利性

使用.NET Core，您可以并排安装不同的版本，这是在使用.NET Framework 时不可用的功能。通过.NET Core 的并排安装，可以在单个服务器上安装多个应用程序，使每个应用程序都可以在其自己的.NET Core 版本上运行。最近，人们对容器和应用程序容器化引起了很多关注。容器用于创建软件应用程序的独立包，包括使应用程序在共享操作系统上与其他应用程序隔离运行所需的运行时。当使用.NET Core 作为开发平台时，将.NET 应用程序容器化要好得多。这是因为它具有跨平台支持，从而允许将应用程序部署到不同操作系统的容器中。此外，使用.NET Core 创建的容器映像更小、更轻量。

# 可扩展性和性能

使用.NET Core，开发使用微服务架构的应用程序相对较容易。使用微服务架构，您可以开发使用不同技术混合的应用程序，例如使用 PHP、Java 或 Rails 开发的服务。您可以使用.NET Core 开发微服务，以部署到云平台或容器中。使用.NET Core，您可以开发可扩展的应用程序，可以在高性能计算机或高端服务器上运行，从而使您的应用程序可以轻松为数十万用户提供服务。

# .NET Core 的限制

虽然.NET Core 是强大的、易于使用的，并在应用程序开发中提供了几个好处，但它目前并不适用于所有的开发问题和场景。微软从.NET Framework 中删除了几项技术，以使.NET Core 变得简化和跨平台。因此，这些技术在.NET Core 中不可用。

当您的应用程序将使用.NET Core 中不可用的技术时，例如在表示层使用 WPF 或 Windows Forms，WCF 服务器实现，甚至目前没有.NET Core 版本的第三方库，建议您使用.NET Framework 开发应用程序。

# .NET Core 应用程序的结构

随着.NET Core 2.0 的发布，添加了新的模板，为可以在平台上运行的不同应用程序类型提供了更多选项。除了现有的项目模板之外，还添加了以下**单页应用程序**（**SPA**）模板：

+   角度

+   ReactJS

+   ReactJS 和 Redux

.NET Core 中的控制台应用程序与.NET Framework 具有类似的结构，而 ASP.NET Core 具有一些新组件，包括以前版本的 ASP.NET 中没有的文件夹和文件。

# ASP.NET Core MVC 项目结构

多年来，ASP.NET Web 框架已经完全成熟，从 Web 表单过渡到 MVC 和 Web API。ASP.NET Core 是一个新的 Web 框架，用于开发可以在.NET Core 上运行的 Web 应用程序和 Web API。它是 ASP.NET 的精简和更简化版本，易于部署，并具有内置的依赖注入。ASP.NET Core 可以与 AngularJS、Bootstrap 和 ReactJS 等框架集成。

ASP.NET Core MVC，类似于 ASP.NET MVC，是构建 Web 应用程序和 API 的框架，使用*模型视图控制器模式*。与 ASP.NET MVC 一样，它支持模型绑定和验证，标签助手，并使用*Razor 语法*用于 Razor 页面和 MVC 视图。

ASP.NET Core MVC 应用程序的结构与 ASP.NET MVC 不同，添加了新的文件夹和文件。当您从 Visual Studio 2017，Visual Studio for Mac 或通过解决方案资源管理器中的 CLI 工具创建新的 ASP.NET Core 项目时，您可以看到添加到项目结构的新组件。

# wwwroot 文件夹

在 ASP.NET Core 中，新添加的`wwwroot`文件夹用于保存库和静态内容，例如图像，JavaScript 文件和库，以及 CSS 和 HTML，以便轻松访问并直接提供给 Web 客户端。`wwwroot`文件夹包含`.css`，图像，`.js`和`.lib`文件夹，用于组织站点的静态内容。

# 模型，视图和控制器文件夹

与 ASP.NET MVC 项目类似，ASP.NET MVC 核心应用程序的根文件夹也包含**模型**，**视图**和**控制器**，遵循 MVC 模式的约定，以正确分离 Web 应用程序文件，代码和表示逻辑。

# JSON 文件 - bower.json，appsettings.json，bundleconfig.json

引入的一些其他文件包括`appsettings.json`，其中包含所有应用程序设置，`bower.json`，其中包含用于管理项目中使用的客户端包括 CSS 和 JavaScript 框架的条目，以及`bundleconfig.json`，其中包含用于配置项目的捆绑和最小化的条目。

# Program.cs

与 C#控制台应用程序类似，ASP.NET Core 具有`Program`类，这是一个重要的类，包含应用程序的入口点。该文件具有用于运行应用程序的`Main()`方法，并用于创建`WebHostBuilder`的实例，用于创建应用程序的主机。在`Main`方法中指定要由应用程序使用的`Startup`类：

```cs
 public class Program
 {
        public static void Main(string[] args)
        {
            BuildWebHost(args).Run();
        }

        public static IWebHost BuildWebHost(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>()
                .Build();
    }
```

# Startup.cs

ASP.NET Core 应用程序需要`Startup`类来管理应用程序的请求管道，配置服务和进行依赖注入。

不同的`Startup`类可以为不同的环境创建；例如，您可以在应用程序中创建两个`Startup`类，一个用于开发环境，另一个用于生产环境。您还可以指定一个`Startup`类用于所有环境。

`Startup`类有两个方法——`Configure()`，这是必须的，用于确定应用程序如何响应 HTTP 请求，以及`ConfigureServices()`，这是可选的，用于在调用`Configure`方法之前配置服务。这两种方法在应用程序启动时都会被调用：

```cs
 public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc();
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
            }

            app.UseStaticFiles();

            app.UseMvc(routes =>
            {
                routes.MapRoute(
                    name: "default",
                    template: "{controller=Home}/{action=Index}/{id?}");
            });
        }
    }

```

# 微软的 Visual Studio Code 编辑器之旅

开发.NET Core 应用程序变得更加容易，不仅因为平台的流畅性和健壮性，还因为引入了**Visual Studio Code**，这是一个跨平台编辑器，可以在 Windows、Linux 和 macOS 上运行。在创建.NET Core 应用程序之前，您不需要在系统上安装 Visual Studio IDE。

Visual Studio Code 虽然没有 Visual Studio IDE 那么强大和功能丰富，但确实具有内置的生产力工具和功能，使得使用它轻松创建.NET Core 应用程序。您还可以在 Visual Studio Code 中安装用于多种编程语言的扩展，从 Visual Studio Marketplace 中获取，从而可以灵活地编辑其他编程语言编写的代码。

# 在 Linux 上安装.NET Core

为了展示.NET Core 的跨平台功能，让我们在 Ubuntu 17.04 桌面版上设置.NET Core 开发环境。在安装 Visual Studio Code 之前，让我们在**Ubuntu OS**上安装.NET Core。首先，您需要通过在添加 Microsoft 产品 feed 之前注册 Microsoft 签名密钥来进行一次性注册：

1.  启动系统终端并运行以下命令注册微软签名密钥：

```cs
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
```

1.  使用此命令注册 Microsoft 产品 feed：

```cs
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-zesty-prod zesty main" > /etc/apt/sources.list.d/dotnetdev.list
```

1.  要在 Linux 操作系统上安装.NET Core SDK 和其他开发.NET Core 应用程序所需的组件，请运行以下命令：

```cs
sudo apt-get update
sudo apt-get install dotnet-sdk-2.0.0
```

1.  这些命令将更新系统，您应该会看到之前添加的 Microsoft 存储库在 Ubuntu 尝试从中获取更新的存储库列表中。更新后，.NET Core 工具将被下载并安装到系统上。您终端屏幕上显示的信息应该与以下截图中的信息类似：

![](img/3a3748ac-cf5e-4ae0-8041-eadcea29414c.png)

1.  安装完成后，在`Documents`文件夹内创建一个新文件夹，并将其命名为`testapp`。将目录更改为新创建的文件夹，并创建一个新的控制台应用程序来测试安装。请参阅以下命令和命令的结果截图：

```cs
cd /home/user/Documents/testapp
dotnet new console
```

这将产生以下输出：

![](img/b8149bd8-8130-4331-910b-e87bb25ef0c0.png)

1.  您会在终端上看到.NET Core 正在创建项目和所需的文件。项目成功创建后，终端上将显示`Restore succeeded`。在`testapp`文件夹中，框架将添加一个`obj`文件夹，`Program.cs`和`testapp.csproj`文件。

1.  您可以继续使用`dotnet run`命令运行控制台应用程序。该命令将在终端上显示`Hello World!`之前编译和运行项目。

# 在 Linux 上安装和设置 Visual Studio Code

由于 Visual Studio Code 是一个跨平台编辑器，可以安装在许多 Linux OS 的变体上，逐渐添加其他 Linux 发行版的软件包。要在**Ubuntu**上安装 Visual Studio Code，请执行以下步骤：

1.  从[`code.visualstudio.com/download`](https://code.visualstudio.com/)下载适用于 Ubuntu 和 Debian Linux 变体的`.deb`软件包。

1.  从终端安装下载的文件，这将安装编辑器、`apt`存储库和签名密钥，以确保在运行系统更新命令时可以自动更新编辑器：

```cs
sudo dpkg -i <package_name>.deb
sudo apt-get install -f
```

1.  安装成功后，您应该能够启动新安装的 Visual Studio Code 编辑器。该编辑器的外观和感觉与 Visual Studio IDE 略有相似。

# 探索 Visual Studio Code

成功安装 Visual Studio Code 在您的 Ubuntu 实例上后，您需要在开始使用编辑器编写代码之前进行初始环境设置：

1.  从“开始”菜单启动 Visual Studio Code，并从 Visual Studio Marketplace 安装 C#扩展到编辑器。您可以通过按下*Ctrl* + *Shift* + *X*来启动扩展，通过“查看”菜单并单击“扩展”，或直接单击“扩展”选项卡；这将加载一个可用扩展的列表，因此单击并安装 C#扩展。

1.  安装扩展后，单击“重新加载”按钮以在编辑器中激活 C#扩展：

![](img/4756b0c5-1806-4efb-a83a-79512ae250c5.png)

1.  打开您之前创建的控制台应用程序的文件夹；要做到这一点，单击“文件”菜单并选择“打开文件夹”，或按下*Ctrl* + *K*，*Ctrl* + *O.* 这将打开文件管理器；浏览到文件夹的路径并单击打开。这将在 Visual Studio Code 中加载项目的内容。在后台，Visual Studio Code 将尝试下载 Linux 平台所需的依赖项，包括 Linux 的 Omnisharp 和.NET Core 调试器：

![](img/fc691b0d-0075-472a-aa73-f1d3c45d5634.png)

1.  要创建一个新项目，您可以使用编辑器的集成终端，而无需通过系统终端。单击“查看”菜单，然后选择“集成终端”。这将在编辑器中打开终端选项卡，您可以在其中输入命令来创建新项目：

![](img/fe857388-8cc6-41cf-ba2b-a016c8f76981.png)

1.  在打开的项目中，您将看到一个通知，需要构建和调试应用程序所需的资源缺失。如果单击“是”，在资源管理器选项卡中，您可以看到一个`.vscode`树，其中添加了`launch.json`和`tasks.json`文件。单击`Program.cs`文件以将文件加载到编辑器中。从“调试”菜单中选择“开始调试”，或按下*F5*运行应用程序；您应该在编辑器的调试控制台上看到`Hello World!`的显示：

![](img/0e38a634-11cd-4c49-b6e4-d3241bbb1b12.png)

当您启动 Visual Studio Code 时，它会加载上次关闭时的状态，打开您上次访问的文件和文件夹。编辑器的布局易于导航和使用，并带有诸如：

+   状态栏显示您当前打开文件的信息。

+   活动栏提供了访问资源管理器视图以查看项目文件夹和文件，以及源代码控制视图以管理项目的源代码版本控制。调试视图用于查看变量、断点和与调试相关的活动，搜索视图允许您搜索文件夹和文件。扩展视图允许您查看可以安装到编辑器中的可用扩展。

+   编辑区用于编辑项目文件，允许您同时打开最多三个文件进行编辑。

+   面板区域显示不同的面板，用于输出、调试控制台、终端和问题：

![](img/2d24345d-99b6-4fbd-9ae5-864b36bc6da1.png)

# 查看 C# 7 的新功能

多年来，C#编程语言已经成熟；随着每个版本的发布，越来越多的语言特性和构造被添加进来。这门语言最初只是由微软内部开发，并且只能在 Windows 操作系统上运行，现在已经成为开源和跨平台。这是通过.NET Core 和语言的 7 版（7.0 和 7.1）实现的，它增加了语言的特色并改进了可用的功能。特别是语言的 7.2 版和 8.0 版的路线图承诺为语言增加更多功能。

# 元组增强

**元组**在 C#语言中的第 4 版中引入，并以简化形式使用，以提供具有两个或更多数据元素的结构，允许您创建可以返回两个或更多数据元素的方法。在 C# 7 之前，引用元组的元素是通过使用*Item1，Item2，...ItemN*来完成的，其中*N*是元组结构中元素的数量。从 C# 7 开始，元组现在支持包含字段的语义命名，引入了更清晰和更有效的创建和使用元组的方法。

您现在可以通过直接为每个成员分配一个值来创建元组。此赋值将创建一个包含元素*Item1*，*Item2*的元组：

```cs
var names = ("John", "Doe");
```

您还可以创建具有元组中包含的元素的语义名称的元组：

```cs
(string firstName, string lastName) names = ("John", "Doe");
```

元组的名称，而不是具有*Item1*，*Item2*等字段，将在编译时具有可以作为`firstName`和`lastName`引用的字段。

当使用 POCO 可能过于繁琐时，您可以创建自己的方法来返回具有两个或更多数据元素的元组：

```cs
private (string, string) GetNames()
{
    (string firstName, string lastName) names = ("John", "Doe");
    return names;
}
```

# Out 关键字

在 C#中，参数可以按引用或值传递。当您通过引用将参数传递给方法、属性或构造函数时，参数的值将被更改，并且在方法或构造函数超出范围时所做的更改将被保留。使用`out`关键字，您可以在 C#中将方法的参数作为引用传递。在 C# 7 之前，要使用`out`关键字，您必须在将其作为`out`参数传递给方法之前声明一个变量：

```cs
class Program
{
    static void Main(string[] args)
    {
        string firstName, lastName;
        GetNames(out firstName, out lastName);
    }
    private static void GetNames(out string firstName, out string lastName)
    {
        firstName="John";
        lastName="Doe";
    }
}
```

在 C# 7 中，您现在可以将 out 变量传递给方法，而无需先声明变量，前面的代码片段现在看起来像以下内容，这样可以防止您在分配或初始化变量之前错误地使用变量，并使代码更加清晰：

```cs
class Program
{
    static void Main(string[] args)
    {
        GetNames(out string firstName, out string lastName);
    }
    private static void GetNames(out string firstName, out string lastName)
    {
        firstName="John";
        lastName="Doe";
    }
}
```

语言中已添加了对隐式类型输出变量的支持，允许编译器推断变量的类型：

```cs
class Program
{
    static void Main(string[] args)
    {
        GetNames(out var firstName, out var lastName);
    }
    private static void GetNames(out string firstName, out string lastName)
    {
        firstName="John";
        lastName="Doe";
    }
}
```

# Ref 局部变量和返回

C#语言一直有`ref`关键字，允许您使用并返回对其他地方定义的变量的引用。C# 7 添加了另一个功能，`ref`局部变量和`returns`，它提高了性能，并允许您声明在较早版本的语言中不可能的辅助方法。`ref`局部变量和`returns`关键字有一些限制——您不能在`async`方法中使用它们，也不能返回具有相同执行范围的变量的引用。

# Ref 局部变量

`ref`局部关键字允许您通过使用`ref`关键字声明局部变量来存储引用，并在方法调用或赋值之前添加`ref`关键字。例如，在以下代码中，`day`字符串变量引用`dayOfWeek`；更改`day`的值也会更改`dayOfWeek`的值，反之亦然：

```cs
string dayOfWeek = "Sunday";
ref string day = ref dayOfWeek;
Console.WriteLine($"day-{day}, dayOfWeek-{dayOfWeek}");
day = "Monday";
Console.WriteLine($"day-{day}, dayOfWeek-{dayOfWeek}");
dayOfWeek = "Tuesday";
Console.WriteLine($"day-{day}, dayOfWeek-{dayOfWeek}");

-----------------
Output:

day: Sunday
dayOfWeek:  Sunday

day: Monday
dayOfWeek:  Monday

day: Tuesday
dayOfWeek:  Tuesday
```

# Ref 返回

您还可以将`ref`关键字用作方法的返回类型。要实现这一点，将`ref`关键字添加到方法签名中，并在方法体内，在`return`关键字之后添加`ref`。在以下代码片段中，声明并初始化了一个字符串数组。然后，该方法将字符串数组的第五个元素作为引用返回：

```cs
public ref string GetFifthDayOfWeek()
{
    string [] daysOfWeek= new string [7] {"Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"};
    return ref daysOfWeek[4];
}
```

# 局部函数

**局部**或**嵌套函数**允许您在另一个函数内定义一个函数。这个特性在一些编程语言中已经有很多年了，但是在 C# 7 中才刚刚引入。当您需要一个小型且在`container`方法的上下文之外不可重用的函数时，这是一个理想的选择：

```cs
class Program
{
    static void Main(string[] args)
    {
        GetNames(out var firstName, out var lastName); 

        void GetNames(out string firstName, out string lastName)
        {
            firstName="John";
            lastName="Doe";
        }
    }
}
```

# 模式匹配

C# 7 包括模式，这是一种语言元素特性，允许您在除了对象类型之外的属性上执行方法分派。它扩展了已经在覆盖和虚拟方法中实现的语言构造，用于实现类型和数据元素的分派。在语言的 7.0 版本中，`is`和`switch`表达式已经更新以支持**模式匹配**，因此您现在可以使用这些表达式来确定感兴趣的对象是否具有特定模式。

使用`is`模式表达式，您现在可以编写包含处理不相关类型元素的算法例程的代码。`is`表达式现在可以与模式一起使用，除了能够测试类型之外。

引入的模式匹配可以采用三种形式：

+   **类型模式**：这涉及检查对象是否是某种类型，然后将对象的值提取到表达式中定义的新变量中：

```cs
public void ProcessLoan(Loan loan)
{
    if(loan is CarLoan carLoan)
    {
        // do something
    }
}
```

+   **Var 模式**：创建一个与对象相同类型的新变量并赋值：

```cs
public void ProcessLoan(Loan loan)
{
    if(loan is var carLoan)
    {
        // do something
    }
}
```

+   常量模式：检查提供的对象是否等同于一个常量表达式：

```cs
public void ProcessLoan(Loan loan)
{
    if(loan is null)
    {
        // do something
    }
}
```

通过更新的 switch 表达式，您现在可以在 case 语句中使用模式和条件，并且可以在除了基本或原始类型之外的任何类型上进行 switch，同时允许您使用 when 关键字来额外指定模式的规则：

```cs
public void ProcessLoan(Loan loan)
{
    switch(loan)
    {
        case CarLoan carLoan:
            // do something
            break;
        case HouseLoan houseLoan when (houseLoan.IsElligible==true):
            //do something
            break;
        case null:
            //throw some custom exception
            break;
        default:
            // do something       
    }
}
```

# 数字分隔符和二进制字面量

在 C# 7 中添加了一种新的语法糖，即**数字分隔符**。这种构造极大地提高了代码的可读性，特别是在处理 C#支持的不同数值类型的大量数字时。在 C# 7 之前，操作大数值以添加分隔符有点混乱和难以阅读。引入数字分隔符后，您现在可以使用下划线（`_`）作为数字的分隔符：

```cs
var longDigit = 2_300_400_500_78;
```

在这个版本中还新增了**二进制字面量**。现在可以通过简单地在二进制值前加上`0b`来创建二进制字面量：

```cs
var binaryValue = 0b11101011;
```

# 创建一个 ASP.NET MVC Core 应用程序

ASP.NET Core 提供了一种优雅的方式来构建在 Windows、Linux 和 macOS 上运行的 Web 应用程序和 API，这要归功于.NET Core 平台的工具和 SDK，这些工具和 SDK 简化了开发尖端应用程序并支持应用程序版本的并行。使用 ASP.NET Core，您的应用程序的表面积更小，这可以提高性能，因为您只需要包含运行应用程序所需的 NuGet 包。ASP.NET Core 还可以与客户端库和框架集成，允许您使用您已经熟悉的 CSS 和 JS 库来开发 Web 应用程序。

ASP.NET Core 使用 Kestrel 运行，Kestrel 是包含在 ASP.NET Core 项目模板中的 Web 服务器。Kestrel 是一个基于**libuv**的进程内跨平台 HTTP 服务器实现，libuv 是一个跨平台的异步 I/O 库，使构建和调试 ASP.NET Core 应用程序变得更加容易。它监听 HTTP 请求，然后将请求的详细信息和特性打包到一个`HttpContext`对象中。Kestrel 可以作为独立的 Web 服务器使用，也可以与 IIS 或 Apache Web 服务器一起使用，其他 Web 服务器接收到的请求将被转发到 Kestrel，这个概念被称为反向代理。

**ASP.NET MVC Core**为您提供了一个可测试的框架，用于使用*Model View Controller*模式进行现代 Web 应用程序开发，这使您可以充分实践测试驱动开发。在 ASP.NET 2.0 中新增的是对 Razor 页面的支持，这现在是开发 ASP.NET Core Web 应用程序用户界面的推荐方法。

要创建一个新的 ASP.NET MVC Core 项目：

1.  打开 Visual Studio Code，并通过选择“视图”菜单中的“集成终端”来访问集成终端面板。在终端上，运行以下命令：

```cs
cd /home/<user>/Documents/
mkdir LoanApp
cd LoanApp
dotnet new mvc
```

1.  创建应用程序后，使用 Visual Studio Code 打开项目文件夹，并选择`Startup.cs`文件。您应该注意到屏幕顶部的通知，提示“从'LoanApp'缺少构建和调试所需的资产。是否添加？”，选择是：

![](img/b461a4e5-e5e9-4e40-8d1f-91b0d0b99697.png)

1.  按下*F5*键来构建和运行 MVC 应用程序。这告诉 Kestrel web 服务器运行该应用程序，并在计算机上启动默认浏览器，地址为`http://localhost:5000`。

![](img/5859d47a-a4f2-4833-96ea-d321ad99d212.png)

# 摘要

.NET Core 平台虽然新，但正在迅速成熟，2.0.7 版本引入了许多功能和增强功能，简化了构建不同类型的跨平台应用程序。在本章中，我们已经对平台进行了介绍，介绍了 C# 7 的新功能，并在 Ubuntu Linux 上设置了开发环境，同时创建了我们的第一个 ASP.NET MVC Core 应用程序。

在下一章中，我们将解释要注意避免编写不可测试代码，并且我们将带领您了解可以帮助您编写可测试和高质量代码的 SOLID 原则。
