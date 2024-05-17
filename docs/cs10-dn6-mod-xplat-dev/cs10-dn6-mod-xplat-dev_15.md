# 第十五章：使用模型-视图-控制器模式构建网站

本章介绍了如何使用 Microsoft ASP.NET Core MVC 在服务器端使用现代 HTTP 架构构建网站，包括启动配置、身份验证、授权、路由、请求和响应管道、模型、视图和控制器，这些组成了 ASP.NET Core MVC 项目。

本章将涵盖以下主题：

+   设置 ASP.NET Core MVC 网站

+   探索 ASP.NET Core MVC 网站

+   自定义 ASP.NET Core MVC 网站

+   查询数据库并使用显示模板

+   使用异步任务来提高可伸缩性

# 设置 ASP.NET Core MVC 网站

ASP.NET Core Razor Pages 非常适合简单的网站。对于更复杂的网站，最好有一个更正式的结构来管理复杂性。

这就是 **模型-视图-控制器**（**MVC**）设计模式有用的地方。它使用像 Razor Pages 这样的技术，但允许在技术关注点之间进行更清晰的分离，如下列表所示：

+   **模型**：代表网站上使用的数据实体和视图模型的类。

+   **视图**：Razor 文件，即 `.cshtml` 文件，用于将视图模型中的数据呈现为 HTML 网页。Blazor 使用 `.razor` 文件扩展名，但不要将它们与 Razor 文件混淆！

+   **控制器**：当 HTTP 请求到达 Web 服务器时执行代码的类。控制器方法通常创建一个视图模型，其中可能包含实体模型，并将其传递给视图以生成要发送回 Web 浏览器或其他客户端的 HTTP 响应。

了解如何使用 MVC 设计模式进行 Web 开发的最佳方法是看一个实际的示例。

## 创建 ASP.NET Core MVC 网站

您将使用项目模板创建一个具有用于验证和授权用户的数据库的 ASP.NET Core MVC 网站项目。Visual Studio 2022 默认使用 SQL Server LocalDB 作为帐户数据库。Visual Studio Code（或更准确地说是 `dotnet` 工具）默认使用 SQLite，并且您可以指定一个开关来使用 SQL Server LocalDB。

让我们看看它的运行方式：

1.  使用您喜欢的代码编辑器添加一个具有存储在数据库中的身份验证帐户的 MVC 网站项目，如下列表所定义：

1.  项目模板：**ASP.NET Core Web App (Model-View-Controller)** / `mvc`

1.  语言：C#

1.  工作区/解决方案文件和文件夹：`PracticalApps`

1.  项目文件和文件夹：`Northwind.Mvc`

1.  选项：**身份验证类型：个人帐户** / `--auth Individual`

1.  对于 Visual Studio，将所有其他选项保留为默认值

1.  在 Visual Studio Code 中，将 `Northwind.Mvc` 选择为活动的 OmniSharp 项目。

1.  构建 `Northwind.Mvc` 项目。

1.  在命令行或终端中，使用 `help` 开关查看此项目模板的其他选项，如下命令所示：

```cs

dotnet new mvc --help

```

1.  注意结果，如下部分输出所示：

```cs

ASP.NET Core Web App (Model-View-Controller) (C#)

作者：Microsoft

描述：用于创建具有示例 ASP.NET Core MVC 视图和控制器的 ASP.NET Core 应用程序的项目模板。此模板也可用于 RESTful HTTP 服务。

此模板包含来自 Microsoft 以外的其他方的技术，请参阅 https://aka.ms/aspnetcore/6.0-third-party-notices 了解详情。

```

有许多选项，特别是与身份验证相关的选项，如下表所示：

| 开关 | 描述 |
| --- | --- |
| `-au&#124;--auth` | 要使用的身份验证类型：`None`（默认）：此选择还允许您禁用 HTTPS。`Individual`：将注册用户及其密码存储在数据库中（默认为 SQLite）的个人身份验证。我们将在本章为此项目使用此选项。`IndividualB2C`：带有 Azure AD B2C 的个人身份验证。`SingleOrg`：单租户的组织身份验证。`MultiOrg`：多租户的组织身份验证。`Windows`：Windows 身份验证。对于内部网络非常有用。 |
| `-uld&#124;--use-local-db` | 是否使用 SQL Server LocalDB 而不是 SQLite。此选项仅适用于指定了`--auth Individual`或`--auth IndividualB2C`的情况。该值是一个可选的`bool`，默认值为`false`。 |
| `-rrc&#124;--razor-runtime-compilation` | 确定项目是否在`Debug`构建中配置为使用 Razor 运行时编译。这可以提高调试期间启动的性能，因为它可以推迟 Razor 视图的编译。该值是一个可选的`bool`，默认值为`false`。 |
| `-f&#124;--framework` | 项目的目标框架。值可以是：`net6.0`（默认），`net5.0`或`netcoreapp3.1` |

## 为 SQL Server LocalDB 创建身份验证数据库

如果您使用 Visual Studio 2022 创建了 MVC 项目，或者使用了`dotnet new mvc`与`-uld`或`--use-local-db`开关，则用于身份验证和授权的数据库将存储在 SQL Server LocalDB 中。但是数据库尚不存在。现在让我们创建它。

在`Northwind.Mvc`文件夹中的命令提示符或终端中，输入命令以运行数据库迁移，以便创建用于存储身份验证凭据的数据库，如下命令所示：

```cs

dotnet ef database update

```

如果您使用`dotnet new`创建了 MVC 项目，则用于身份验证和授权的数据库将存储在 SQLite 中，并且文件已经创建，名称为`app.db`。

用于身份验证数据库的连接字符串名为`DefaultConnection`，存储在 MVC 网站项目的根文件夹中的`appsettings.json`文件中。

对于 SQL Server LocalDB（带有截断的连接字符串），请参阅以下标记：

```cs

{

"ConnectionStrings"

: {

"DefaultConnection"

: "Server=(localdb)\\mssqllocaldb;Database=aspnet-Northwind.Mvc-...;Trusted_Connection=True;MultipleActiveResultSets=true"

},

```

对于 SQLite，请参阅以下标记：

```cs

{

"ConnectionStrings"

: {

"DefaultConnection"

: "DataSource=app.db;Cache=Shared"

},

```

## 探索默认的 ASP.NET Core MVC 网站

让我们回顾一下默认的 ASP.NET Core MVC 网站项目模板的行为：

1.  在`Northwind.Mvc`项目中，展开`Properties`文件夹，打开`launchSettings.json`文件，并注意为项目配置的`HTTPS`和`HTTP`的随机端口号（您的端口号将不同），如下标记所示：

```cs

"profiles"

: {

"Northwind.Mvc"

: {

"commandName"

: "Project"

，

"dotnetRunMessages"

: true

，

"launchBrowser"

: true

，

"applicationUrl"

: "https://localhost:7274;http://localhost:5274"

，

"environmentVariables"

: {

"ASPNETCORE_ENVIRONMENT"

: "Development"

}

},

```

1.  将端口号更改为`5001`（`HTTPS`）和`5000`（`HTTP`），如下标记所示：

```cs

"applicationUrl"

: "https://localhost:5001;http://localhost:5000"

，

```

1.  保存对`launchSettings.json`文件的更改。

1.  启动网站。

1.  启动 Chrome 并打开**Developer Tools**。

1.  导航到`http://localhost:5000/`，并注意以下内容，如*图 15.1*所示：

+   HTTP 的请求会自动重定向到端口`5001`上的 HTTPS。

+   顶部导航菜单链接到**Home**，**Privacy**，**Register**和**Login**。如果视口宽度小于或等于 575 像素，则导航会折叠成汉堡菜单。

+   网站的标题**Northwind.Mvc**显示在页眉和页脚中。![](img/Image00120.jpg)

图 15.1：ASP.NET Core MVC 项目模板网站首页

### 理解访客注册

默认情况下，密码必须至少包含一个非字母数字字符，必须至少包含一个数字（0-9），并且必须至少包含一个大写字母（A-Z）。在这种情况下，我会使用`Pa$$w0rd`，因为我只是在探索。

MVC 项目模板遵循**双重确认**（**DOI**）的最佳实践，这意味着在填写电子邮件和密码进行注册后，会向该电子邮件地址发送一封电子邮件，访客必须点击该电子邮件中的链接以确认他们想要注册。

我们还没有配置电子邮件提供程序来发送该电子邮件，所以我们必须模拟该步骤：

1.  在顶部导航菜单中，点击**注册**。

1.  输入电子邮件和密码，然后点击**注册**按钮。（我使用了`test@example.com`和`Pa$$w0rd`。）

1.  点击带有文本**点击此处确认您的帐户**的链接，并注意您被重定向到一个**确认电子邮件**网页，您可以自定义。

1.  在顶部导航菜单中，点击**登录**，输入您的电子邮件和密码（请注意，有一个可选的复选框可以记住您，如果访客忘记了密码或者想要注册为新访客，还有链接），然后点击**登录**按钮。

1.  点击顶部导航菜单中的您的电子邮件地址。这将导航到一个帐户管理页面。请注意，您可以设置电话号码，更改电子邮件地址，更改密码，启用双因素身份验证（如果您添加了身份验证器应用程序），以及下载和删除您的个人数据。

1.  关闭 Chrome 并关闭 Web 服务器。

## 审查 MVC 网站项目结构

在您的代码编辑器中，在 Visual Studio **Solution Explorer**（切换到**显示所有文件**）或在 Visual Studio Code **EXPLORER**中，查看 MVC 网站项目的结构，如*图 15.2*所示：

![](img/Image00121.jpg)

图 15.2：ASP.NET Core MVC 项目的默认文件夹结构

我们稍后会更详细地查看其中的一些部分，但现在请注意以下内容：

+   `Areas`：这个文件夹包含了用于将您的网站项目与**ASP.NET Core Identity**集成的必需的嵌套文件夹和文件，用于身份验证。

+   `bin`，`obj`：这些文件夹包含了在构建过程中需要的临时文件和项目的编译程序集。

+   `Controllers`：这个文件夹包含了 C#类，其中有方法（称为操作），用于获取模型并将其传递给视图，例如`HomeController.cs`。

+   `Data`：这个文件夹包含了 Entity Framework Core 迁移类，用于 ASP.NET Core Identity 系统提供身份验证和授权的数据存储，例如`ApplicationDbContext.cs`。

+   `Models`：这个文件夹包含了代表控制器收集并传递给视图的所有数据的 C#类，例如`ErrorViewModel.cs`。

+   `Properties`：这个文件夹包含了一个用于在 Windows 上启动网站时配置 IIS 或 IIS Express 的配置文件，名为`launchSettings.json`。这个文件只在本地开发机器上使用，不会部署到生产网站上。

+   `Views`：这个文件夹包含了`.cshtml` Razor 文件，用于动态生成 HTML 响应的 HTML 和 C#代码。`_ViewStart`文件设置了默认布局，`_ViewImports`导入了所有视图中使用的常见命名空间，如标签助手：

+   `Home`：这个子文件夹包含了主页和隐私页面的 Razor 文件。

+   `Shared`：这个子文件夹包含了共享布局、错误页面以及用于登录和验证脚本的两个部分视图的 Razor 文件。

+   `wwwroot`：此文件夹包含网站使用的静态内容，例如用于样式的 CSS，JavaScript 库，用于此网站项目的 JavaScript 和`favicon.ico`文件。您还可以在此处放置图像和其他静态文件资源，如 PDF 文档。项目模板包括 Bootstrap 和 jQuery 库。

+   `app.db`：这是存储注册访客的 SQLite 数据库。（如果您使用了 SQL Server LocalDB，则将不需要。）

+   `appsettings.json`和`appsettings.Development.json`：这些文件包含您的网站可以在运行时加载的设置，例如 ASP.NET Core Identity 系统的数据库连接字符串和日志级别。

+   `Northwind.Mvc.csproj`：此文件包含项目设置，例如使用 Web .NET SDK，SQLite 的条目以确保`app.db`文件被复制到网站的输出文件夹，并且项目所需的 NuGet 包列表，包括：

+   `Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore`

+   `Microsoft.AspNetCore.Identity.EntityFrameworkCore`

+   `Microsoft.AspNetCore.Identity.UI`

+   `Microsoft.EntityFrameworkCore.Sqlite`或`Microsoft.EntityFrameworkCore.SqlServer`

+   `Microsoft.EntityFrameworkCore.Tools`

+   `Program.cs`：此文件定义了一个隐藏的`Program`类，其中包含`Main`入口点。它构建了一个用于处理传入 HTTP 请求的管道，并使用默认选项托管网站，例如配置 Kestrel Web 服务器和加载`appsettings`。它添加和配置了网站所需的服务，例如用于身份验证的 ASP.NET Core Identity，用于身份数据存储的 SQLite 或 SQL Server 等，并为应用程序添加了路由。

## 审查 ASP.NET Core Identity 数据库

打开`appsettings.json`以查找用于 ASP.NET Core Identity 数据库的连接字符串，如以下标记中对 SQL Server LocalDB 进行了突出显示的那样：

```cs

{

"ConnectionStrings"

: {

"DefaultConnection"

: "

**Server=(localdb)\\mssqllocaldb;Database=aspnet-Northwind.Mvc-2F6A1E12-F9CF-480C-987D-FEFB4827DE22;Trusted_Connection=True;MultipleActiveResultSets=true**

"

},

"Logging"

: {

"LogLevel"

: {

"默认"

: "信息"

，

"Microsoft"

: "警告"

，

"Microsoft.Hosting.Lifetime"

: "信息"

}

},

"AllowedHosts"

: "*"

}

```

如果您使用 SQL Server LocalDB 作为身份数据存储，则可以使用**服务器资源管理器**连接到数据库。您可以从`appsettings.json`文件中复制并粘贴连接字符串（但删除`(localdb)`和`mssqllocaldb`之间的第二个反斜杠）。

如果您安装了 SQLiteStudio 等 SQLite 工具，则可以打开 SQLite `app.db`数据库文件。

然后，您可以看到 ASP.NET Core Identity 系统用于注册用户和角色的表，包括用于存储注册访客的`AspNetUsers`表。

**良好实践**：ASP.NET Core MVC 项目模板遵循良好实践，存储密码的哈希而不是密码本身，您将在*第二十章* *保护您的数据和应用程序*中了解更多（可在[`github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf`](https://github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf)找到）。

# 探索一个 ASP.NET Core MVC 网站

让我们逐步了解构成现代 ASP.NET Core MVC 网站的部分。

## 理解 ASP.NET Core MVC 初始化

恰如其名，我们将从探索 MVC 网站的默认初始化和配置开始：

1.  打开`Program.cs`文件，并注意它使用了顶级程序功能（因此有一个隐藏的`Program`类和一个`Main`方法）。从上到下，可以将此文件分为四个重要部分。

.NET 5 及更早的 ASP.NET Core 项目模板使用`Startup`类将这些部分分开为单独的方法，但是在.NET 6 中，Microsoft 鼓励将所有内容放在单个`Program.cs`文件中。

1.  第一部分导入了一些命名空间，如下所示：

```cs

使用

Microsoft.AspNetCore.Identity; // IdentityUser

使用

Microsoft.EntityFrameworkCore; // UseSqlServer, UseSqlite

使用

Northwind.Mvc.Data; // ApplicationDbContext

```

请记住，默认情况下，许多其他命名空间是使用.NET 6 及更高版本的隐式使用功能导入的。构建项目，然后可以在以下路径找到全局导入的命名空间：`obj\Debug\net6.0\Northwind.Mvc.GlobalUsings.g.cs`。

1.  第二部分创建和配置 Web 主机生成器。它注册一个使用 SQL Server 或 SQLite 的应用程序数据库上下文，其数据库连接字符串从`appsettings.json`文件中加载用于数据存储，添加了 ASP.NET Core Identity 以进行身份验证并配置其使用应用程序数据库，并添加了对 MVC 控制器的视图支持，如下所示：

```cs

var

builder = WebApplication.CreateBuilder(args);

// 向容器添加服务。

var

connectionString = builder.Configuration

调用`GetConnectionString("DefaultConnection"`方法

);

builder.Services.AddDbContext<ApplicationDbContext>(options =>

options.UseSqlServer(connectionString)); // 或 UseSqlite

builder.Services.AddDatabaseDeveloperPageExceptionFilter();

builder.Services.AddDefaultIdentity<IdentityUser>(options =>

options.SignIn.RequireConfirmedAccount = true

）

.AddEntityFrameworkStores<ApplicationDbContext>();

builder.Services.AddControllersWithViews();

```

`builder`对象有两个常用对象：`Configuration`和`Services`：

+   `Configuration`包含您可以设置配置的所有位置的合并值：`appsettings.json`，环境变量，命令行参数等等

+   `Services`是已注册的依赖服务的集合

对`AddDbContext`的调用是注册依赖服务的示例。ASP.NET Core 实现了**依赖注入**（**DI**）设计模式，以便其他组件（如控制器）可以通过它们的构造函数请求所需的服务。开发人员在`Program.cs`的这一部分注册这些服务（或者如果使用`Startup`类，则在其`ConfigureServices`方法中）。

1.  第三部分配置 HTTP 请求管道。如果网站在开发中运行，则配置相对 URL 路径以运行数据库迁移，或者为生产环境配置更友好的错误页面和 HSTS。启用了 HTTPS 重定向、静态文件、路由和 ASP.NET Identity，并配置了 MVC 默认路由和 Razor Pages，如下所示：

```cs

// 配置 HTTP 请求管道。

如果

（app.Environment.IsDevelopment()）

{

app.UseMigrationsEndPoint();

}

否则

{

app.UseExceptionHandler("/Home/Error"

);

// 默认的 HSTS 值为 30 天。您可能希望为生产场景更改此值，请参阅 https://aka.ms/aspnetcore-hsts。

app.UseHsts();

}

app.UseHttpsRedirection();

app.UseStaticFiles();

app.UseRouting();

app.UseAuthentication();

app.UseAuthorization();

app.MapControllerRoute(

name: "default"

，

模式："{controller=Home}/{action=Index}/{id?}"

);

app.MapRazorPages();

```

我们在*第十四章*中学习了大多数这些方法和特性，*使用 ASP.NET Core Razor Pages 构建网站*。

**良好的实践**：扩展方法`UseMigrationsEndPoint`是做什么的？你可以阅读官方文档，但它并没有太大帮助。例如，它没有告诉我们默认定义的相对 URL 路径是什么：[`docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.migrationsendpointextensions.usemigrationsendpoint`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.migrationsendpointextensions.usemigrationsendpoint)。幸运的是，ASP.NET Core 是开源的，所以我们可以阅读源代码并发现它的作用，链接如下：[`github.com/dotnet/aspnetcore/blob/main/src/Middleware/Diagnostics.EntityFrameworkCore/src/MigrationsEndPointOptions.cs#L18`](https://github.com/dotnet/aspnetcore/blob/main/src/Middleware/Diagnostics.EntityFrameworkCore/src/MigrationsEndPointOptions.cs#L18)。养成探索 ASP.NET Core 源代码的习惯，以了解它的工作原理。

除了`UseAuthentication`和`UseAuthorization`方法之外，在`Program.cs`的这一部分中最重要的新方法是`MapControllerRoute`，它映射了 MVC 可以使用的默认路由。这个路由非常灵活，因为它将映射到几乎任何传入的 URL，你将在下一个主题中看到。

虽然我们在本章中不会创建任何 Razor 页面，但我们需要保留映射 Razor 页面支持的方法调用，因为我们的 MVC 网站使用 ASP.NET Core Identity 进行身份验证和授权，并且它使用 Razor 类库作为其用户界面组件，比如访客注册和登录。

1.  第四部分有一个阻塞线程的方法调用，运行网站并等待传入的 HTTP 请求进行响应，如下面的代码所示：

```cs

app.Run(); // 阻塞调用

```

## 理解默认的 MVC 路由

路由的责任是发现要实例化的控制器类的名称，并执行一个操作方法，可选地传递一个`id`参数到将生成 HTTP 响应的方法中。

MVC 配置了默认路由，如下面的代码所示：

```cs

endpoints.MapControllerRoute(

name: "default"

,

模式: "{controller=Home}/{action=Index}/{id?}"

);

```

路由模式中有花括号`{}`中的部分称为**segments**，它们就像方法的命名参数。这些 segments 的值可以是任何`string`。URL 中的 segments 不区分大小写。

路由模式查看浏览器请求的任何 URL 路径，并匹配以提取`controller`的名称，`action`的名称和可选的`id`值（`?`符号表示可选）。

如果用户没有输入这些名称，则使用`Home`作为控制器的默认值，使用`Index`作为操作的默认值（`=`赋值为命名 segment 设置默认值）。

以下表格包含示例 URL 以及默认路由如何确定控制器和操作的名称：

| URL | 控制器 | 操作 | ID |
| --- | --- | --- | --- |
| `/` | 主页 | 索引 |  |
| `/Muppet` | Muppet | Index |  |
| `/Muppet/Kermit` | Muppet | Kermit |  |
| `/Muppet/Kermit/Green` | Muppet | Kermit | Green |
| `/Products` | Products | Index |  |
| `/Products/Detail` | Products | Detail |  |
| `/Products/Detail/3` | Products | Detail | 3 |

## 理解控制器和操作

在 MVC 中，C 代表*controller*。根据路由和传入的 URL，ASP.NET Core 知道控制器的名称，因此它将寻找一个带有`[Controller]`属性或从带有该属性的类派生的类，例如，Microsoft 提供的名为`ControllerBase`的类，如下面的代码所示：

```cs

namespace

Microsoft.AspNetCore.Mvc

{

//

// 摘要:

// 一个没有视图支持的 MVC 控制器的基类。

[Controller

]

公共

抽象

class

ControllerBase

{

...

```

### 理解 ControllerBase 类

正如您在 XML 注释中所看到的，`ControllerBase`不支持视图。它用于创建 Web 服务，正如您将在*第十六章*，*构建和使用 Web 服务*中看到的。

`ControllerBase`有许多有用的属性用于处理当前 HTTP 上下文，如下表所示：

| 属性 | 描述 |
| --- | --- |
| `Request` | 仅 HTTP 请求。例如，标头，查询字符串参数，请求正文作为可以读取的流，内容类型和长度，以及 cookie。 |
| `Response` | 仅 HTTP 响应。例如，标头，响应正文作为可以写入的流，内容类型和长度，状态代码和 cookie。还有像`OnStarting`和`OnCompleted`这样的委托，您可以将方法连接到它们。 |
| `HttpContext` | 当前 HTTP 上下文的所有内容，包括请求和响应，有关连接的信息，已在服务器上启用中间件的功能集合，以及用于身份验证和授权的`User`对象。 |

### 了解 Controller 类

微软提供了另一个名为`Controller`的类，如果您的类确实需要视图支持，可以从这个类继承，如下面的代码所示：

```cs

namespace

Microsoft.AspNetCore.Mvc

{

//

// 摘要：

// 一个带有视图支持的 MVC 控制器的基类。

public

抽象

类

Controller

: ControllerBase

，

IActionFilter

, IFilterMetadata

, IAsyncActionFilter

, IDisposable

{

...

```

`Controller`有许多有用的属性用于处理视图，如下表所示：

| 属性 | 描述 |
| --- | --- |
| `ViewData` | 控制器可以在其中存储键/值对的字典，在视图中可以访问。该字典的生命周期仅限于当前请求/响应。 |
| `ViewBag` | 包装`ViewData`的动态对象，以提供更友好的语法来设置和获取字典值。 |
| `TempData` | 控制器可以在其中存储键/值对的字典，在视图中可以访问。该字典的生命周期限于当前请求/响应和同一访客会话的下一个请求/响应。这对于在初始请求期间存储值，响应重定向，然后在随后的请求中读取存储的值非常有用。 |

`Controller`有许多有用的方法用于处理视图，如下表所示：

| 属性 | 描述 |
| --- | --- |
| `View` | 在执行呈现完整响应的视图后返回`ViewResult`，例如，动态生成的网页。可以使用约定选择视图，也可以使用字符串名称指定。可以向视图传递模型。 |
| `PartialView` | 在执行作为完整响应的一部分的视图后返回`PartialViewResult`，例如，动态生成的 HTML 块。可以使用约定选择视图，也可以使用字符串名称指定。可以向视图传递模型。 |
| `ViewComponent` | 在执行动态生成 HTML 的组件后返回`ViewComponentResult`。必须通过指定其类型或名称来选择组件。可以将对象作为参数传递。 |
| `Json` | 返回包含 JSON 序列化对象的`JsonResult`。这对于在 MVC 控制器的一部分实现简单的 Web API 并主要返回供人查看的 HTML 非常有用。 |

### 了解控制器的职责

控制器的职责如下：

+   确定控制器在其类构造函数中需要的服务，以使其处于有效状态并能够正常运行。

+   使用操作名称来标识要执行的方法。

+   从 HTTP 请求中提取参数。

+   使用参数获取构造视图模型所需的任何其他数据，并将其传递给客户端的适当视图。例如，如果客户端是 Web 浏览器，则呈现 HTML 的视图最合适。其他客户端可能更喜欢替代呈现，比如文档格式，如 PDF 文件或 Excel 文件，或数据格式，如 JSON 或 XML。

+   将视图的结果作为适当状态码的 HTTP 响应返回给客户端。

让我们回顾一下用于生成主页、隐私和错误页面的控制器：

1.  展开`Controllers`文件夹

1.  打开名为`HomeController.cs`的文件

1.  请注意，如下面的代码所示：

+   导入了额外的命名空间，我已经添加了注释以显示它们所需的类型。

+   声明一个私有的只读字段来存储对`HomeController`的记录器的引用，该引用在构造函数中设置。

+   所有三个操作方法都调用一个名为`View`的方法，并将结果作为`IActionResult`接口返回给客户端。

+   `Error`操作方法将视图模型传递到其视图中，并使用用于跟踪的请求 ID。错误响应不会被缓存：

```cs

using

Microsoft.AspNetCore.Mvc; // Controller, IActionResult

using

Northwind.Mvc.Models; // ErrorViewModel

using

System.Diagnostics; // Activity

namespace

Northwind.Mvc.Controllers

;

public

class

HomeController

：控制器

{

private

readonly

ILogger<HomeController> _logger;

public

HomeController

(

ILogger<HomeController> logger

)

{

_logger = logger;

}

public

IActionResult

Index

()

{

return

View();

}

public

IActionResult

Privacy

()

{

return

View();

}

[ResponseCache(Duration = 0,

Location = ResponseCacheLocation.None, NoStore = true)

]

public

IActionResult

Error

()

{

return

View(new

ErrorViewModel { RequestId =

Activity.Current?.Id ?? HttpContext.TraceIdentifier });

}

}

```

如果访问者导航到路径`/`或`/Home`，那么它相当于`/Home/Index`，因为这些是默认路由中控制器和操作的默认名称。

## 理解视图搜索路径约定

`Index`和`Privacy`方法在实现上是相同的，但它们返回不同的网页。这是因为**约定**。对`View`方法的调用在不同的路径中查找 Razor 文件以生成网页。

让我们故意破坏一个页面名称，以便我们可以看到默认搜索的路径：

1.  在`Northwind.Mvc`项目中，展开`Views`文件夹，然后展开`Home`文件夹。

1.  将`Privacy.cshtml`文件重命名为`Privacy2.cshtml`。

1.  启动网站。

1.  启动 Chrome，导航到`https://localhost:5001/`，点击**隐私**，并注意搜索用于呈现网页的视图的路径（包括 MVC 视图和 Razor 页面的`Shared`文件夹），如*图 15.3*所示：![](img/Image00122.jpg)

图 15.3：显示默认视图搜索路径的异常

1.  关闭 Chrome 并关闭 Web 服务器。

1.  将`Privacy2.cshtml`文件重命名为`Privacy.cshtml`。

您现在已经看到了视图搜索路径约定，如下面的列表所示：

+   特定的 Razor 视图：`/Views/{controller}/{action}.cshtml`

+   共享的 Razor 视图：`/Views/Shared/{action}.cshtml`

+   共享的 Razor 页面：`/Pages/Shared/{action}.cshtml`

## 理解日志记录

您刚刚看到有些错误被捕获并写入控制台。您可以使用记录器以相同的方式向控制台写入消息。

1.  在`Controllers`文件夹中，在`HomeController.cs`文件中，在`Index`方法中，添加语句使用记录器将各种级别的消息写入控制台，如下面的代码所示：

```cs

_logger.LogError("这是一个严重错误（并不是真的！）"

);

_logger.LogWarning("这是你的第一个警告！"

);

_logger.LogWarning("第二个警告！"

);

_logger.LogInformation("我在 HomeController 的 Index 方法中。"

);

```

1.  启动`Northwind.Mvc`网站项目。

1.  启动 Web 浏览器，导航到网站的主页。

1.  在命令提示符或终端中，注意消息，如下面的输出所示：

```cs

失败：Northwind.Mvc.Controllers.HomeController[0]

这是一个严重的错误（实际上并不是！）

警告：Northwind.Mvc.Controllers.HomeController[0]

这是你的第一个警告！

警告：Northwind.Mvc.Controllers.HomeController[0]

第二个警告！

信息：Northwind.Mvc.Controllers.HomeController[0]

我在 HomeController 的 Index 方法中。

```

1.  关闭 Chrome 并关闭 Web 服务器。

## 理解过滤器

当您需要向多个控制器和操作添加一些功能时，您可以使用或定义自己的过滤器，这些过滤器被实现为属性类。

过滤器可以应用在以下级别：

+   通过使用属性装饰一个操作方法来设置操作级别。这只会影响一个操作方法。

+   通过使用属性装饰控制器类来设置控制器级别。这将影响控制器的所有方法。

+   通过将属性类型添加到`MvcOptions`实例的`Filters`集合中，在调用`AddControllersWithViews`方法时可以用于配置 MVC，如下面的代码所示：

```cs

builder.Services.AddControllersWithViews(options =>

{

options.Filters.Add(typeof

（MyCustomFilter）;

});

```

### 使用过滤器保护操作方法

您可能希望确保控制器类的一个特定操作方法只能被某些安全角色的成员调用。您可以通过使用`[授权]`属性装饰该方法来实现这一点，如下列表所述：

+   `[授权]`：只允许经过身份验证（非匿名，已登录）的访问者访问此操作方法。

+   `[授权（角色 = "销售，市场营销")]`：只允许属于指定角色的访问者访问此操作方法。

让我们看一个例子：

1.  在`HomeController.cs`中，导入`Microsoft.AspNetCore.Authorization`命名空间。

1.  向`Privacy`方法添加一个属性，只允许属于名为`Administrators`的组/角色的已登录用户访问，如下面的代码所示：

```cs

**[**

**授权（角色 =**

**"Administrators"**

**）**

**]**

公共

IActionResult

隐私

（）

```

1.  启动网站。

1.  点击**隐私**，注意您被重定向到登录页面。

1.  输入您的电子邮件和密码。

1.  点击**登录**，注意您被拒绝访问。

1.  关闭 Chrome 并关闭 Web 服务器。

### 启用角色管理并以编程方式创建角色

在 ASP.NET Core MVC 项目中，默认情况下未启用角色管理，因此我们必须在创建角色之前先启用它，然后我们将创建一个控制器，该控制器将以编程方式创建一个`Administrators`角色（如果尚不存在），并将一个测试用户分配给该角色：

1.  在`Program.cs`中，在设置 ASP.NET Core Identity 及其数据库时，添加一个调用`AddRoles`以启用角色管理，如下面的代码所示：

```cs

services.AddDefaultIdentity<IdentityUser>(

options => options.SignIn.RequireConfirmedAccount = true

）

**.AddRoles<IdentityRole>()**

**// 启用角色管理**

.AddEntityFrameworkStores<ApplicationDbContext>();

```

1.  在`Controllers`中，添加一个名为`RolesController.cs`的空控制器类，并修改其内容，如下面的代码所示：

```cs

使用

Microsoft.AspNetCore.Identity; // RoleManager, UserManager

使用

Microsoft.AspNetCore.Mvc; // Controller, IActionResult

使用

静态的

System.Console;

命名空间

Northwind.Mvc.Controllers

;

公共

类

RolesController

控制器

{

私人的

字符串

AdminRole = "Administrators"

;

私人的

字符串

UserEmail = "test@example.com"

;

私人的

只读

RoleManager<IdentityRole> roleManager;

私人的

只读

UserManager<IdentityUser> userManager;

公共

RolesController

（

RoleManager<IdentityRole> roleManager，

UserManager<IdentityUser> userManager

）

{

这个

.roleManager = roleManager;

这个

.userManager = userManager;

}

公共

async

Task<IActionResult>

索引

（）

{

如果

（！（等待

roleManager.RoleExistsAsync(AdminRole)))

{

await

roleManager.CreateAsync(new

IdentityRole(AdminRole));

}

IdentityUser user = await

userManager.FindByEmailAsync(UserEmail);

如果

（user == null

）

{

user = new

();

user.UserName = UserEmail;

user.Email = UserEmail;

IdentityResult result = await

userManager.CreateAsync(

user, "Pa$$w0rd"

);

如果

（result.Succeeded）

{

WriteLine($"用户

{user.UserName}

成功创建。"

);

}

否则

{

foreach

（IdentityError error in

result.Errors)

{

WriteLine(error.Description);

}

}

}

如果

（！user.EmailConfirmed）

{

字符串

token = await

userManager

.GenerateEmailConfirmationTokenAsync(user);

IdentityResult result = await

userManager

.ConfirmEmailAsync(user, token);

如果

（result.Succeeded）

{

WriteLine($"用户

{user.UserName}

电子邮件成功确认。"

);

}

否则

{

foreach

（IdentityError error in

result.Errors)

{

WriteLine(error.Description);

}

}

}

如果

（！（await

userManager.IsInRoleAsync(user, AdminRole)))

{

IdentityResult result = await

userManager

.AddToRoleAsync(user, AdminRole);

如果

（result.Succeeded）

{

WriteLine($"用户

{user.UserName}

添加到

{AdminRole}

成功。"

);

}

否则

{

foreach

（IdentityError error in

result.Errors)

{

WriteLine(error.Description);

}

}

}

返回

重定向“/”

）;

}

}

```

注意以下内容：

+   角色名称和用户电子邮件的两个字段。

+   构造函数获取并存储注册用户和角色管理器依赖服务。

+   如果`Administrators`角色不存在，我们使用角色管理器来创建它。

+   我们尝试通过其电子邮件找到测试用户，如果不存在则创建用户，然后将用户分配给`Administrators`角色。

+   由于网站使用 DOI，我们必须生成电子邮件确认令牌，并使用它来确认新用户的电子邮件地址。

+   成功消息和任何错误都会写入控制台。

+   您将被自动重定向到主页。

1.  启动网站。

1.  点击**隐私**，注意您被重定向到登录页面。

1.  输入您的电子邮件和密码。（我使用了`mark@example.com`。）

1.  点击**登录**，注意您仍然被拒绝访问。

1.  点击**主页**。

1.  在地址栏中，手动输入`roles`作为相对 URL 路径，如下面的链接所示：`https://localhost:5001/roles`。

1.  查看写入控制台的成功消息，如下面的输出所示：

```cs

用户 test@example.com 成功创建。

用户 test@example.com 的电子邮件成功确认。

用户 test@example.com 成功添加到管理员。

```

1.  点击**注销**，因为您必须注销并重新登录以在已经登录后创建角色成员资格时加载它们。

1.  尝试再次访问**隐私**页面，输入以编程方式创建的新用户的电子邮件，例如`test@example.com`，以及他们的密码，然后单击**登录**，现在您应该可以访问。

1.  关闭 Chrome 并关闭 Web 服务器。

### 使用过滤器缓存响应

为了提高响应时间和可伸缩性，您可能希望通过使用`[ResponseCache]`属性装饰方法来缓存由操作方法生成的 HTTP 响应。

您可以通过设置参数来控制响应的缓存位置和持续时间，如下面的列表所示：

+   `Duration`：以秒为单位。这将设置以秒为单位的`max-age` HTTP 响应标头。常见选择是一小时（3600 秒）和一天（86400 秒）。

+   `位置`：`ResponseCacheLocation`值之一，`Any`，`Client`或`None`。这将设置`cache-control` HTTP 响应标头。

+   `NoStore`：如果为`true`，则会忽略`Duration`和`Location`，并将`cache-control` HTTP 响应标头设置为`no-store`。

让我们看一个例子：

1.  在`HomeController.cs`中，为`Index`方法添加一个属性，以在浏览器或服务器和浏览器之间的任何代理上缓存响应 10 秒，如下面的代码中所示：

```cs

**[**

**ResponseCache(Duration = 10, Location = ResponseCacheLocation.Any)**

**]**

公共

IActionResult

索引

()

```

1.  在`Views`中，在`Home`中，打开`Index.cshtml`，并添加一个段落，以长格式输出当前时间，包括秒，如下面的标记所示：

```cs

<

p

类

=

"警报警报-主要"

@DateTime.Now.ToLongTimeString()</

p

```

1.  启动网站。

1.  注意主页上的时间。

1.  单击**注册**。

1.  单击**主页**，注意主页上的时间相同，因为使用了页面的缓存版本。

1.  单击**注册**。等待至少十秒。

1.  单击**主页**，注意时间现在已更新。

1.  单击**登录**，输入您的电子邮件和密码，然后单击**登录**。

1.  注意主页上的时间。

1.  单击**隐私**。

1.  单击**主页**，注意页面未被缓存。

1.  查看控制台，并注意警告消息，解释您的缓存已被覆盖，因为访问者已登录，在这种情况下，ASP.NET Core 使用防伪令牌，它们不应该被缓存，如下面的输出所示：

```cs

警告：Microsoft.AspNetCore.Antiforgery.DefaultAntiforgery[8]

已覆盖'Cache-Control'和'Pragma'标头，并分别设置为'no-cache, no-store'和'no-cache'，以防止缓存此响应。任何使用防伪的响应都不应该被缓存。

```

1.  关闭 Chrome 并关闭 Web 服务器。

### 使用过滤器定义自定义路由

您可能希望为操作方法定义一个简化的路由，而不是使用默认路由。

例如，要显示隐私页面当前需要以下 URL 路径，其中指定了控制器和操作：

```cs

https://localhost:5001/home/privacy

```

我们可以简化路由，如下面的链接所示：

```cs

https://localhost:5001/private

```

让我们看看如何做到这一点：

1.  在`HomeController.cs`中，向`Privacy`方法添加属性以定义简化路由，如下面的代码中突出显示的那样：

```cs

**[**

**Route（**

**"私人"**

**)**

**]**

[授权（角色=

"管理员"

）

]

公共

IActionResult

隐私

（）

```

1.  启动网站。

1.  在地址栏中，输入以下 URL 路径：

```cs

https://localhost:5001/private

```

1.  输入您的电子邮件和密码，单击**登录**，注意简化路径显示**隐私**页面。

1.  关闭 Chrome 并关闭 Web 服务器。

## 理解实体和视图模型

在 MVC 中，M 代表*模型*。模型表示响应请求所需的数据。通常使用两种类型的模型：实体模型和视图模型。

**实体模型**表示数据库中的实体，如 SQL Server 或 SQLite。根据请求，可能需要从数据存储中检索一个或多个实体。实体模型是使用类定义的，因为它们可能需要更改，然后用于更新底层数据存储。

我们希望在响应请求时显示的所有数据都是**MVC 模型**，有时称为**视图模型**，因为它是传递到视图以呈现为 HTML 或 JSON 等响应格式的模型。视图模型应该是不可变的，因此通常使用记录来定义。

例如，以下 HTTP `GET`请求可能意味着浏览器正在请求产品编号为 3 的产品详情页面：

[`www.example.com/products/details/3`](http://www.example.com/products/details/3)

控制器需要使用 ID 路由值 3 来检索该产品的实体，并将其传递给一个视图，然后可以将模型转换为 HTML 以在浏览器中显示。

想象一下，当用户来到我们的网站时，我们想向他们展示一个类别的轮播图，产品列表以及本月我们有多少访问者的计数。

我们将引用您在*第十三章*中创建的 Northwind 数据库的 Entity Framework Core 实体数据模型，*介绍 C#和.NET 的实际应用*：

1.  在`Northwind.Mvc`项目中，向`Northwind.Common.DataContext`添加对 SQLite 或 SQL Server 的项目引用，如下面的标记所示：

```cs

<ItemGroup>

<!--如果要切换到 SqlServer，请将 Sqlite 更改为 SqlServer

您喜欢的-->

<ProjectReference Include=

"..\Northwind.Common.DataContext.Sqlite\Northwind.Common.DataContext.Sqlite.csproj"

/>

</ItemGroup>

```

1.  构建`Northwind.Mvc`项目以编译其依赖项。

1.  如果您正在使用 SQL Server，或者可能想要在 SQL Server 和 SQLite 之间切换，那么在`appsettings.json`中，添加一个连接字符串，用 SQL Server 连接到 Northwind 数据库，如下面标记的部分所示：

```cs

{

"ConnectionStrings"

{

"DefaultConnection"

: "Server=(localdb)\\mssqllocaldb;Database=aspnet-Northwind.Mvc-DC9C4FAF-DD84-4FC9-B925-69A61240EDA7;Trusted_Connection=True;MultipleActiveResultSets=true"

,

**"NorthwindConnection"**

**:**

**"Server=.;Database=Northwind;Trusted_Connection=True;MultipleActiveResultSets=true"**

},

```

1.  在`Program.cs`中，导入命名空间以使用您的实体模型类型，如下面的代码所示：

```cs

using

Packt.Shared; // AddNorthwindContext 扩展方法

```

1.  在`builder.Build`方法调用之前，添加语句来加载适当的连接字符串，然后注册`Northwind`数据库上下文，如下面的代码所示：

```cs

//如果您正在使用 SQL Server

string

sqlServerConnection = builder.Configuration

.GetConnectionString("NorthwindConnection"

);

builder.Services.AddNorthwindContext(sqlServerConnection);

//如果您使用的是 SQLite，默认是..\Northwind.db

builder.Services.AddNorthwindContext();

```

1.  将一个类文件添加到`Models`文件夹，并将其命名为`HomeIndexViewModel.cs`。

**良好的实践**：虽然 MVC 项目模板创建的`ErrorViewModel`类不遵循此约定，但我建议您为视图模型类使用命名约定`{Controller}{Action}ViewModel`。

1.  修改语句以定义一个记录，该记录具有三个属性，用于统计访问者数量，以及类别和产品的列表，如下面的代码所示：

```cs

using

Packt.Shared; // Category, Product

namespace

Northwind.Mvc.Models

;

public

记录

HomeIndexViewModel

(

int

VisitorCount,

IList<Category> Categories,

IList<Product> Products

)

;

```

1.  在`HomeController.cs`中，导入`Packt.Shared`命名空间，如下面的代码所示：

```cs

using

Packt.Shared; // NorthwindContext

```

1.  添加一个字段来存储对`Northwind`实例的引用，并在构造函数中进行初始化，如下面标记的部分所示：

```cs

public

类

HomeController

: 控制器

{

private

readonly

ILogger<HomeController> _logger;

**private**

**readonly**

**NorthwindContext db;**

public

HomeController

(

ILogger<HomeController> logger,

**NorthwindContext injectedContext**

**)**

{

_logger = logger;

**db = injectedContext;**

}

...

```

ASP.NET Core 将使用构造函数参数注入来传递`NorthwindContext`数据库上下文的实例，使用您在`Program.cs`中指定的连接字符串。

1.  修改`Index`操作方法中的语句，以创建此方法的视图模型的实例，使用`Random`类模拟访问者计数生成 1 到 1000 之间的数字，并使用`Northwind`数据库获取类别和产品的列表，然后将模型传递给视图，如下面的代码所示：

```cs

[ResponseCache(Duration = 10, Location = ResponseCacheLocation.Any)

]

public

IActionResult

Index

()

{

_logger.LogError("这是一个严重的错误（实际上并不是！）"

);

_logger.LogWarning("这是您的第一个警告！"

);

_logger.LogWarning("第二个警告！"

);

_logger.LogInformation("我在 HomeController 的 Index 方法中。"

);

**HomeIndexViewModel model =**

**new**

**(**

**VisitorCount: (**

**new**

**Random()).Next(**

**1**

**,**

**1001**

**),**

**Categories: db.Categories.ToList(),**

**Products: db.Products.ToList()**

**);**

**return**

**View(model);**

**//将模型传递给视图**

}

```

请记住视图搜索约定：当在控制器的操作方法中调用`View`方法时，ASP.NET Core MVC 会在`Views`文件夹中查找与当前控制器相同名称的子文件夹，即`Home`。然后，它会查找与当前操作相同名称的文件，即`Index.cshtml`。它还将在`Shared`文件夹中查找与操作方法名称匹配的视图，并在`Pages`文件夹中查找 Razor Pages。

## 理解视图

在 MVC 中，V 代表*视图*。视图的责任是将模型转换为 HTML 或其他格式。

有多个**视图引擎**可用于执行此操作。默认视图引擎称为**Razor**，它使用`@`符号来指示服务器端代码执行。与 ASP.NET Core 2.0 一起引入的 Razor Pages 功能使用相同的视图引擎，因此可以使用相同的 Razor 语法。

让我们修改主页视图以呈现类别和产品的列表：

1.  展开`Views`文件夹，然后展开`Home`文件夹。

1.  打开`Index.cshtml`文件，注意用`@{ }`包装的 C#代码块。这将首先执行，并且可以用于存储需要传递到共享布局文件中的数据，例如网页的标题，如下面的代码所示：

```cs

@{

ViewData["Title"

] = "主页"

;

}

```

1.  请注意在使用 Bootstrap 进行样式设置的`<div>`元素中的静态 HTML 内容。

**良好实践**：除了定义自己的样式外，还应基于实现响应式设计的常见库（例如 Bootstrap）来定义样式。

就像 Razor Pages 一样，有一个名为`_ViewStart.cshtml`的文件，它由`View`方法执行。它用于设置适用于所有视图的默认值。

例如，它将所有视图的`Layout`属性设置为共享布局文件，如下面的标记所示：

```cs

@{

Layout = "_Layout"

;

}

```

1.  在`Views`文件夹中，打开`_ViewImports.cshtml`文件，并注意它导入了一些命名空间，然后添加了 ASP.NET Core 标记助手，如下面的代码所示：

```cs

@using Northwind.Mvc

@using Northwind.Mvc.Models

@addTagHelper *，Microsoft.AspNetCore.Mvc.TagHelpers

```

1.  在`Shared`文件夹中，打开`_Layout.cshtml`文件。

1.  请注意标题是从`ViewData`字典中读取的，该字典是在`Index.cshtml`视图中设置的，如下面的标记所示：

```cs

<

title

@ViewData["Title"] – Northwind.Mvc</

title

```

1.  请注意链接的呈现以支持 Bootstrap 和站点样式表，其中`〜`表示`wwwroot`文件夹，如下面的标记所示：

```cs

<

link

rel

=

"stylesheet"

href

=

"〜/lib/bootstrap/dist/css/bootstrap.css"

/>

<

link

rel

=

"stylesheet"

href

=

"〜/css/site.css"

/>

```

1.  请注意标题中导航栏的呈现，如下面的标记所示：

```cs

<

body

<

header

<

nav

class

=

"navbar ..."

```

1.  请注意呈现可折叠的`<div>`，其中包含用于登录的部分视图和使用 ASP.NET Core 标记助手的超链接，以允许用户在页面之间导航，属性如`asp-controller`和`asp-action`，如下面的标记所示：

```cs

<

div

class

=

"navbar-collapse collapse d-sm-inline-flex justify-content-between"

<

ul

class

=

"navbar-nav flex-grow-1"

<

li

class

=

"nav-item"

<

a

class

=

"nav-link text-dark"

asp-area

=

""

asp-controller

=

"Home"

asp-action

=

"Index"

Home</

a

</

li

<

li

class

=

"nav-item"

<

a

class

=

"nav-link text-dark"

asp-area

=

""

asp-controller

=

"Home"

asp-action

=

"Privacy"

Privacy</

a

</

li

</

ul

<

partial

name

=

"_LoginPartial"

/>

</

div

```

`<a>`元素使用名为`asp-controller`和`asp-action`的标记助手属性来指定单击链接时将执行的控制器名称和操作名称。如果要导航到 Razor 类库中的功能，例如在上一章中创建的`employees`组件，则使用`asp-area`来指定功能名称。

1.  注意在`<main>`元素内部呈现主体，如下面的标记所示：

```cs

<

div

类

=

"容器"

<

主要

角色

=

"主要"

类

=

"pb-3"

@RenderBody()

</

主要

</

div

```

`RenderBody`方法在共享布局的特定 Razor 视图中注入内容，例如`Index.cshtml`文件。

1.  注意在页面底部呈现`<script>`元素，以便不会减慢页面的显示，并且您可以将自己的脚本块添加到一个名为`scripts`的可选定义部分中，如下面的标记所示：

```cs

<

脚本

src

=

"〜/lib/jquery/dist/jquery.min.js"

></

脚本

<

脚本

src

=

"〜/lib/bootstrap/dist/js/bootstrap.bundle.min.js"

</

脚本

<

脚本

src

=

"〜/js/site.js"

asp-append-version

=

"真"

></

脚本

@await RenderSectionAsync("scripts", required: false)

```

当`asp-append-version`与`true`值一起在任何元素中指定，如`<img>`或`<script>`以及`src`属性时，将调用 Image Tag Helper（此助手命名不当，因为它不仅影响图像！）。

它通过自动附加一个名为`v`的查询字符串值来工作，该值是从引用源文件的哈希生成的，如下面的示例生成的输出所示：

```cs

<script src="img/site.js? v=Kl_dqr9NVtnMdsM2MUg4qthUnWZm5T1fCEimBPWDNgM"></script>

```

如果`site.js`文件中的任何一个字节发生变化，那么它的哈希值将不同，因此如果浏览器或 CDN 缓存了脚本文件，它将破坏缓存副本并用新版本替换它。

# 自定义 ASP.NET Core MVC 网站

现在您已经审查了基本 MVC 网站的结构，您将对其进行自定义和扩展。您已经为`Northwind`数据库注册了一个 EF Core 模型，因此下一个任务是在主页上输出一些数据。

## 定义自定义样式

主页将显示北风数据库中的 77 个产品列表。为了有效利用空间，我们希望以三列显示列表。为此，我们需要自定义网站的样式表：

1.  在`wwwroot\css`文件夹中，打开`site.css`文件。

1.  在文件底部，添加一个新的样式，将应用于具有`product-columns` ID 的元素，如下面的代码所示：

```cs

#product-columns

{

列数

：3

;

}

```

## 设置类别图片

北风数据库包括一个包含八个类别的表，但它们没有图像，网站看起来更好看一些有些丰富多彩的图片：

1.  在`wwwroot`文件夹中，创建一个名为`images`的文件夹。

1.  在`images`文件夹中，添加八个图像文件，命名为`category1.jpeg`，`category2.jpeg`等，直到`category8.jpeg`。

您可以从本书的 GitHub 存储库中下载图像，链接如下：[`github.com/markjprice/cs10dotnet6/tree/master/Assets/Categories`](https://github.com/markjprice/cs10dotnet6/tree/master/Assets/Categories)

## 理解 Razor 语法

在我们自定义主页视图之前，让我们回顾一个初始的 Razor 文件示例，该文件具有一个初始的 Razor 代码块，用于实例化具有价格和数量的订单，然后在网页上输出有关订单的信息，如下面的标记所示：

```cs

@{

订单订单=新的

（）

{

订单 ID = 123

，

产品=“寿司”

，

价格= 8.49

M，

数量= 3

};

}

<div

您的@order.Quantity @order.Product 的订单总成本为$@ order.Price * @order.Quantity</div

```

上述的 Razor 文件将导致以下不正确的输出：

```cs

您的寿司 3 份订单总成本为$8.49 * 3

```

尽管 Razor 标记可以使用`@object.property`语法包括任何单个属性的值，但你应该用括号括起表达式，如下面的标记所示：

```cs

<

div

您的@order.Quantity @order.Product 的订单总成本为$@（order.Price * order.Quantity）</

div

```

上述的 Razor 表达式将导致以下正确的输出：

```cs

您 3 份寿司的订单总成本为$25.47

```

## 定义一个类型化视图

为了在编写视图时改进智能感知，您可以在顶部使用`@model`指令定义视图可以期望的类型：

1.  在`Views\Home`文件夹中，打开`Index.cshtml`。

1.  在文件顶部，添加一个语句来设置模型类型，以使用`HomeIndexViewModel`，如下面的代码所示：

```cs

@model HomeIndexViewModel

```

现在，每当我们在这个视图中输入`Model`时，您的代码编辑器将知道模型的正确类型，并为其提供智能感知。

在视图中输入代码时，请记住以下内容：

+   声明模型的类型，请使用`@model`（小写 m）。

+   与模型的实例交互，使用`@Model`（大写 M）。

让我们继续定制主页的视图。

1.  在初始的 Razor 代码块中，添加一个语句来声明当前项目的`string`变量，并在现有的`<div>`元素下添加新的标记，以输出轮播中的类别和产品作为无序列表，如下面的标记所示：

```cs

@using Packt.Shared

@model HomeIndexViewModel

@{

ViewData["Title"

] = "主页"

;

string

currentItem = ""

;

}

<

div

class

=

"text-center"

<

h1

class

=

"display-4"

欢迎</

h1

<

p

了解 <

a

href

=

"https://docs.microsoft.com/aspnet/core"

使用 ASP.NET Core 构建 Web 应用程序</

a

。</

p

<

p

class

=

"alert alert-primary"

@DateTime.Now.ToLongTimeString()</

p

</

div

@if (Model is not null)

{

<

div

id

=

"categories"

class

=

"carousel slide"

data-ride

=

"carousel"

data-interval

=

"3000"

data-keyboard

=

"true"

<

ol

class

=

"carousel-indicators"

@for (int

c = 0

; c < Model.Categories.Count; c++)

{

if

(c == 0

)

{

currentItem = "active";

}

else

{

currentItem = "";

}

<

li

data-target

=

"#categories"

data-slide-to

=

"@c"

class

=

"@currentItem"

></

li

}

</

ol

<

div

class

=

"carousel-inner"

@for (int c = 0

; c < Model.Categories.Count; c++)

{

if

(c == 0

)

{

currentItem = "active";

}

else

{

currentItem = "";

}

<

div

class

=

"carousel-item @currentItem"

<

img

class

=

"d-block w-100"

src

=

"~/images/category@(Model.Categories[c].CategoryId).jpeg"

alt

=

"@Model.Categories[c].CategoryName"

/>

<

div

class

=

"carousel-caption d-none d-md-block"

<

h2

@Model.Categories[c].CategoryName</

h2

<

h3

@Model.Categories[c].Description</

h3

<

p

<

a

class

=

"btn btn-primary"

href

=

"/category/@Model.Categories[c].CategoryId"

View</

a

</

p

</

div

</

div

}

</

div

<

a

class

=

"carousel-control-prev"

href

=

"#categories"

role

=

"button"

data-slide

=

"prev"

<

span

class

=

"carousel-control-prev-icon"

aria-hidden

=

"true"

></

span

<

span

class

=

"sr-only"

Previous</

span

</

a

<

a

class

=

"carousel-control-next"

href

=

"#categories"

role

=

"button"

data-slide

=

"next"

<

span

class

=

"carousel-control-next-icon"

aria-hidden

=

"true"

></

span

<

span

class

=

"sr-only"

下一个</

span

</

a

</

div

}

<

div

class

=

"row"

<

div

class

=

"col-md-12"

<

h1

Northwind</

h1

<

p

class

=

"lead"

本月我们有@Model?.VisitorCount 位访客。

</

p

@if (Model is not null)

{

<

h2

Products</

h2

<

div

id

=

"product-columns"

<

ul

@foreach (Product p in @Model.Products)

{

<

li

<

a

asp-controller

=

"Home"

asp-action

=

"ProductDetail"

asp-route-id

=

"@p.ProductId"

@p.ProductName costs

@(p.UnitPrice is null ? "zero" : p.UnitPrice.Value.ToString("C"))

</

a

</

li

}

</

ul

</

div

}

</

div

</

div

```

在审查上述的 Razor 标记时，请注意以下内容：

+   很容易将静态 HTML 元素（如`<ul>`和`<li>`）与 C#代码混合，以输出类别的轮播和产品名称的列表。

+   具有`id`属性为`product-columns`的`<div>`元素将使用我们之前定义的自定义样式，因此该元素中的所有内容将以三列显示。

+   每个类别的`<img>`元素使用 Razor 表达式周围的括号，以确保编译器不将`.jpeg`包含在表达式中，如下面的标记所示：`"~/images/category@(Model.Categories[c].CategoryID).jpeg"`

+   产品链接的`<a>`元素使用标签助手生成 URL 路径。点击这些超链接将由`HomeController`及其`ProductDetail`操作方法处理。这个操作方法目前还不存在，但你将在本章后面添加它。产品的 ID 作为名为`id`的路由段传递，如 Ipoh Coffee 的以下 URL 路径所示：`https://localhost:5001/Home/ProductDetail/43`。

## 审查定制的主页

让我们看看我们定制的主页的结果：

1.  启动`Northwind.Mvc`网站项目。

1.  请注意主页上有一个旋转的轮播图显示类别，随机数量的访客，以及以三列显示的产品列表，如*图 15.4*所示：![](img/Image00123.jpg)

图 15.4：更新后的 Northwind MVC 网站主页

目前，点击任何类别或产品链接都会出现**404 Not Found**错误，所以让我们看看如何实现使用传递的参数来查看产品或类别的详细信息的页面。

1.  关闭 Chrome 并关闭 Web 服务器。

## 使用路由值传递参数

传递简单参数的一种方法是使用默认路由中定义的`id`段：

1.  在`HomeController`类中，添加一个名为`ProductDetail`的操作方法，如下所示：

```cs

公共的

IActionResult

ProductDetail

(

int

? id

)

{

如果

(!id.HasValue)

{

返回

BadRequest("您必须在路由中传递产品 ID，例如，/Home/ProductDetail/21"

);

}

Product? model = db.Products

.SingleOrDefault(p => p.ProductId == id);

如果

(model == null

)

{

返回

NotFound($"ProductId

{id}

未找到。"

);

}

返回

View(model); //将模型传递给视图，然后返回结果

}

```

请注意以下内容：

+   这种方法使用了 ASP.NET Core 的一个特性，叫做**模型绑定**，自动将路由中传递的`id`与方法中命名为`id`的参数匹配。

+   在方法内部，我们检查`id`是否没有值，如果是，我们调用`BadRequest`方法返回一个带有自定义消息的`400`状态码，解释正确的 URL 路径格式。

+   否则，我们可以连接到数据库并尝试使用`id`值检索产品。

+   如果我们找到了产品，我们将其传递给视图；否则，我们调用`NotFound`方法返回`404`状态码和一个自定义消息，解释数据库中没有找到该 ID 的产品。

1.  在`Views/Home`文件夹中，添加一个名为`ProductDetail.cshtml`的新文件。

1.  修改内容，如下面的标记所示：

```cs

@model Packt.Shared.Product

@{

ViewData["Title"

] = "产品详情 - "

+ Model.ProductName;

}

<

h2

产品详情</

h2

<

hr

/>

<

div

<

dl

类

=

"dl-horizontal"

<

dt

产品 ID</

dt

<

dd

@Model.ProductId</

dd

<

dt

产品名称</

dt

<

dd

@Model.ProductName</

dd

<

dt

类别 ID</

dt

<

dd

@Model.CategoryId</

dd

<

dt

单价</

dt

<

dd

@Model.UnitPrice.Value.ToString("C")</

dd

<

dt

库存单位</

dt

<

dd

@Model.UnitsInStock</

dd

</

dl

</

div

```

1.  启动`Northwind.Mvc`项目。

1.  当主页显示产品列表时，点击其中一个，例如第二个产品，**Chang**。

1.  注意浏览器地址栏中的 URL 路径，浏览器标签中显示的页面标题，以及产品详细信息页面，如*图 15.5*所示：![](img/Image00124.jpg)

图 15.5：Chang 的产品详细页面

1.  查看**开发者工具**。

1.  编辑 Chrome 地址栏中的 URL，请求一个不存在的产品 ID，比如 99，注意 404 Not Found 状态码和自定义错误响应。

## 更详细地了解模型绑定器

模型绑定器功能强大，默认绑定器为您做了很多事情。在默认路由确定要实例化的控制器类和要调用的操作方法之后，如果该方法有参数，则这些参数需要设置值。

模型绑定器通过查找 HTTP 请求中传递的参数值来设置参数值，这些参数值可以是以下类型之一：

+   **路由参数**，如我们在上一节中所做的`id`，如下面的 URL 路径所示：`/Home/ProductDetail/2`

+   **查询字符串参数**，如下面的 URL 路径所示：`/Home/ProductDetail?id=2`

+   **表单参数**，如下面的标记所示：

```cs

<

form

操作

=

"post"

操作

=

"/Home/ProductDetail"

<

输入

类型

=

"text"

名称

=

"id"

值

=

"2"

/>

<

输入

类型

=

"submit"

/>

</

form

```

模型绑定器可以填充几乎任何类型：

+   简单类型，如`int`，`string`，`DateTime`和`bool`。

+   由`class`，`record`或`struct`定义的复杂类型。

+   集合类型，如数组和列表。

让我们创建一个有点人为的例子来说明默认模型绑定器可以实现的功能：

1.  在`Models`文件夹中，添加一个名为`Thing.cs`的新文件。

1.  修改内容以定义一个具有两个属性的类，用于可空整数`Id`和名为`Color`的`string`，如下面的代码所示：

```cs

命名空间

Northwind.Mvc.Models

;

公共

类

事物

{

public

int

? Id { get

; 设置

; }

公共

字符串

? Color { get

; 设置

; }

}

```

1.  在`HomeController`中，添加两个新的操作方法，一个用于显示带有表单的页面，另一个用于使用新模型类型显示带有参数的事物，如下面的代码所示：

```cs

公共

IActionResult

ModelBinding

()

{

返回

View(); // 带有表单的页面

}

公共

IActionResult

ModelBinding

(

事物

)

{

返回

View(thing); // 显示模型绑定的事物

}

```

1.  在`Views\Home`文件夹中，添加一个名为`ModelBinding.cshtml`的新文件。

1.  修改其内容，如下面的标记所示：

```cs

@model Thing

@{

ViewData["Title"

] = "模型绑定演示"

;

}

<

h1

@ViewData["Title"

]</

h1

<

div

在以下表单中输入您的事物的值：

</

div

<

form

方法

=

"POST"

操作

=

"/home/modelbinding?id=3"

<

输入

名称

=

"color"

值

=

"Red"

/>

<

输入

类型

=

"submit"

/>

</

form

@if (Model != null)

{

<

h2

提交的事物</

h2

<

hr

/>

<

div

<

dl

类

=

"dl-horizontal"

<

dt

Model.Id</

dt

<

dd

@Model.Id</

dd

<

dt

Model.Color</

dt

<

dd

@Model.Color</

dd

</

dl

</

div

}

```

1.  在`Views/Home`中，打开`Index.cshtml`，并在第一个`<div>`中添加一个指向模型绑定页面的链接的新段落，如下面的标记所示：

```cs

<

p

><

a

asp-action

=

"ModelBinding"

asp-controller

=

"Home"

绑定</

a

></

p

```

1.  启动网站。

1.  在主页上，单击**绑定**。

1.  注意未处理的关于模糊匹配的异常，如*图 15.6*所示：![](img/Image00125.jpg)

图 15.6：未处理的模糊操作方法不匹配异常

1.  关闭 Chrome 并关闭 Web 服务器。

### 消除歧义的操作方法

尽管 C#编译器可以通过注意到签名不同来区分两种方法，但从 HTTP 请求路由的角度来看，这两种方法都是潜在的匹配项。我们需要一种 HTTP 特定的方法来消除操作方法的歧义。

我们可以通过为操作创建不同的名称或指定一个方法应用于特定的 HTTP 动词（如`GET`，`POST`，`DELETE`）来解决这个问题。这就是我们将解决问题的方式：

1.  在`HomeController`中，装饰第二个`ModelBinding`操作方法，指示它应用于处理 HTTP `POST`请求，也就是说，当提交表单时，如下面的代码中所突出显示的那样：

```cs

**[**

**HttpPost**

**]**

公共

IActionResult

ModelBinding

(

事物

)

```

另一个`ModelBinding`操作方法将隐式用于所有其他类型的 HTTP 请求，如`GET`，`PUT`，`DELETE`等。

1.  启动网站。

1.  在主页上，单击**Binding**。

1.  单击**提交**按钮，注意`Id`属性的值是从查询字符串参数设置的，`color`属性的值是从表单参数设置的，如*图 15.7*所示：![](img/Image00126.jpg)

图 15.7：模型绑定演示页面

1.  关闭 Chrome 并关闭 Web 服务器。

### 传递路由参数

现在我们将使用路由参数设置属性：

1.  修改表单的动作，将值`2`作为路由参数传递，如下面的标记所示：

```cs

<

form

method

=

"POST"

动作

=

"/home/modelbinding

**/2**

?id=3"

```

1.  启动网站。

1.  在主页上，单击**Binding**。

1.  单击**提交**按钮，注意`Id`属性的值是从路由参数设置的，`Color`属性的值是从表单参数设置的。

1.  关闭 Chrome 并关闭 Web 服务器。

### 传递表单参数

现在我们将使用表单参数设置属性：

1.  修改表单的动作，将值 1 作为表单参数传递，如下面的标记所示：

```cs

<form method="POST"

action="/home/modelbinding/2?id=3"

**<input name=**

**"id"**

**value**

**=**

**"1"**

**/>**

<input name="color"

value

="Red"

/>

<input type="submit"

/>

</form>

```

1.  启动网站。

1.  在主页上，单击**Binding**。

1.  单击**提交**按钮，注意`Id`和`Color`属性的值都是从表单参数设置的。

**良好实践**：如果您有多个同名参数，请记住，表单参数具有最高优先级，查询字符串参数具有自动模型绑定的最低优先级。

## 验证模型

模型绑定过程可能会导致错误，例如，如果模型已经使用验证规则进行了装饰，则可能会出现数据类型转换或验证错误。已绑定的数据以及任何绑定或验证错误都存储在`ControllerBase.ModelState`中。

让我们通过对绑定模型应用一些验证规则，然后在视图中显示无效数据消息来探索我们可以使用模型状态做什么：

1.  在`Models`文件夹中，打开`Thing.cs`。

1.  导入`System.ComponentModel.DataAnnotations`命名空间。

1.  使用验证属性装饰`Id`属性，限制允许的数字范围为 1 到 10，并确保访问者提供颜色，并添加一个新的`Email`属性，其中包含用于验证的正则表达式，如下面的代码中所示：

```cs

public

class

Thing

{

**[**

**Range(1, 10)**

**]**

public

int

? Id { get

; set

; }

**[**

**Required**

**]**

public

string

? Color { get

; set

; }

**[**

**EmailAddress**

**]**

**public**

**string**

**? Email {**

**get**

**;**

**set**

**; }**

}

```

1.  在`Models`文件夹中，添加一个名为`HomeModelBindingViewModel.cs`的新文件。

1.  修改其内容以定义一个记录，其中包含存储绑定模型的属性，指示是否存在错误的标志，以及一系列错误消息，如下面的代码所示：

```cs

namespace

Northwind.Mvc.Models

;

public

记录

HomeModelBindingViewModel

(

Thing Thing,

bool

HasErrors,

IEnumerable<

string

> ValidationErrors

)

;

```

1.  在`HomeController`中，处理 HTTP `POST`的`ModelBinding`方法中，注释掉之前传递 thing 到视图的语句，而是添加语句来创建视图模型的实例。验证模型并存储错误消息的数组，然后将视图模型传递给视图，如下面的代码中所示：

```cs

[HttpPost

]

public

IActionResult

ModelBinding

(

Thing thing

)

{

**HomeModelBindingViewModel model =**

**new**

**(**

**thing,**

**!ModelState.IsValid,**

**ModelState.Values**

**.SelectMany(state => state.Errors)**

**.Select(error => error.ErrorMessage)**

**);**

**return**

**View(model);**

}

```

1.  在`Views\Home`中，打开`ModelBinding.cshtml`。

1.  修改模型类型声明以使用视图模型类，如下面的标记所示：

```cs

@model Northwind.Mvc.Models.HomeModelBindingViewModel

```

1.  添加一个`<div>`以显示任何模型验证错误，并更改事物属性的输出，因为视图模型已更改，如下面的标记中所示：

```cs

<

形式

方法

=

"POST"

行动

=

"/home/modelbinding/2?id=3"

<

输入

名称

=

"id"

价值

=

"1"

/>

<

输入

名称

=

"颜色"

价值

=

"红色"

/>

<

输入

名称

=

"电子邮件"

价值

=

"test@example.com"

/>

<

输入

类型

=

"提交"

/>

</

形式

@if (Model != null)

{

<

h2

提交的事物</

h2

<

hr

/>

<

div

<

dl

类

=

"dl-horizontal"

<

dt

模型

**.Thing**

.Id</

dt

<

dd

@Model

**.Thing**

.Id</

dd

<

dt

模型

**.Thing**

.Color</

dt

<

dd

@Model

**.Thing**

.Color</

dd

**<**

**dt**

**>**

**Model.Thing.Email**

**</**

**dt**

**>**

**<**

**dd**

**>**

**@Model.Thing.Email**

**</**

**dd**

**>**

</

dl

</

div

@if (Model.HasErrors)

{

<

div

@foreach(string errorMessage in Model.ValidationErrors)

{

<

div

类

=

"alert alert-danger"

角色

=

"警报"

@errorMessage</

div

}

</

div

}

}

```

1.  启动网站。

1.  在主页上，单击**绑定**。

1.  单击**提交**按钮，注意`1`，`红色`和`test@example.com`是有效值。

1.  输入`Id`为`13`，清除颜色文本框，从电子邮件地址中删除`@`，单击**提交**按钮，并注意错误消息，如*图 15.8*所示：![](img/Image00127.jpg)

图 15.8：具有字段验证的模型绑定演示页面

1.  关闭 Chrome 并关闭 Web 服务器。

**良好的做法**：Microsoft 在`EmailAddress`验证属性的实现中使用了什么正则表达式？请在以下链接找到：[`github.com/microsoft/referencesource/blob/5697c29004a34d80acdaf5742d7e699022c64ecd/System.ComponentModel.DataAnnotations/DataAnnotations/EmailAddressAttribute.cs#L54`](https://github.com/microsoft/referencesource/blob/5697c29004a34d80acdaf5742d7e699022c64ecd/System.ComponentModel.DataAnnotations/DataAnnotations/EmailAddressAttribute.cs#L54)

## 理解视图助手方法

在为 ASP.NET Core MVC 创建视图时，您可以使用`Html`对象及其方法生成标记。

一些有用的方法包括以下内容：

+   `ActionLink`：使用此功能生成包含指定控制器和操作的 URL 路径的锚`<a>`元素。例如，`Html.ActionLink(linkText: "Binding", actionName: "ModelBinding", controllerName: "Home")`将生成`<a href="/home/modelbinding">Binding</a>`。您可以使用锚标签助手实现相同的结果：`<a asp-action="ModelBinding" asp-controller="Home">Binding</a>`。

+   `AntiForgeryToken`：在`<form>`内使用此功能插入包含防伪标记的`<hidden>`元素，当提交表单时将对其进行验证。

+   `Display`和`DisplayFor`：使用此功能为相对于当前模型的表达式生成 HTML 标记，使用显示模板。 .NET 类型和自定义模板有内置的显示模板可以在`DisplayTemplates`文件夹中创建。文件夹名称在区分大小写的文件系统上区分大小写。

+   `DisplayForModel`：使用此功能为整个模型生成 HTML 标记，而不是单个表达式。

+   `Editor`和`EditorFor`：使用此功能为相对于当前模型的表达式生成 HTML 标记，使用编辑器模板。 .NET 类型有内置的编辑器模板，使用`<label>`和`<input>`元素，自定义模板可以在`EditorTemplates`文件夹中创建。文件夹名称在区分大小写的文件系统上区分大小写。

+   `EditorForModel`：使用此功能为整个模型生成 HTML 标记，而不是单个表达式。

+   `Encode`：使用此功能将对象或字符串安全地编码为 HTML。例如，字符串值`"<script>"`将被编码为`"&lt;script&gt;"`。这通常是不必要的，因为 Razor 的`@`符号默认情况下会对字符串值进行编码。

+   `Raw`：使用此功能呈现字符串值*不*编码为 HTML。

+   `PartialAsync`和`RenderPartialAsync`：使用这些方法为部分视图生成 HTML 标记。您可以选择传递模型和视图数据。

让我们看一个例子：

1.  在`Views/Home`中，打开`ModelBinding.cshtml`。

1.  修改`Email`属性的渲染以使用`DisplayFor`，如下所示：

```cs

<

dd

@Html.DisplayFor（model => model.Thing.Email）</

dd

```

1.  启动网站。

1.  点击**绑定**。

1.  点击**提交**。

1.  请注意电子邮件地址是可点击的超链接，而不仅仅是文本。

1.  关闭 Chrome 并关闭 Web 服务器。

1.  在`Models/Thing.cs`中，注释掉`Email`属性上方的`[EmailAddress]`属性。

1.  启动网站。

1.  点击**绑定**。

1.  点击**提交**。

1.  请注意电子邮件地址只是文本。

1.  关闭 Chrome 并关闭 Web 服务器。

1.  在`Models/Thing.cs`中取消注释`[EmailAddress]`属性。

这是将`Email`属性与`[EmailAddress]`验证属性装饰并使用`DisplayFor`渲染它的组合，通知 ASP.NET Core 将该值视为电子邮件地址，因此将其呈现为可点击链接。

# 查询数据库并使用显示模板

让我们创建一个新的操作方法，可以将查询字符串参数传递给它，并使用它来查询 Northwind 数据库中价格高于指定价格的产品。

在以前的示例中，我们定义了一个视图模型，其中包含了需要在视图中呈现的每个值的属性。在本例中，将有两个值：产品列表和访客输入的价格。为了避免必须为视图模型定义一个类或记录，我们将产品列表作为模型传递，并将最高价格存储在`ViewData`集合中。

让我们实现这个功能：

1.  在`HomeController`中，导入`Microsoft.EntityFrameworkCore`命名空间。我们需要这样做以添加`Include`扩展方法，以便我们可以包含相关实体，正如您在*第十章*，*使用 Entity Framework Core 处理数据*中所学到的那样。

1.  根据以下代码添加新的操作方法：

```cs

公共

IActionResult

ProductsThatCostMoreThan

（

decimal

？价格

）

{

如果

（！price.HasValue）

{

返回

BadRequest（“您必须在查询字符串中传递产品价格，例如，/Home/ProductsThatCostMoreThan？price=50”

）；

}

IEnumerable<Product> model = db.Products

.Include（p => p.Category）

.Include（p => p.Supplier）

.Where(p => p.UnitPrice > price);

如果

（！model.Any（））

{

返回

未找到（

“没有产品的成本超过

{price:C}

。”

）；

}

ViewData[“MaxPrice”

] = price.Value.ToString（“C”

）；

返回

视图（模型）；//将模型传递给视图

}

```

1.  在`Views/Home`文件夹中，添加名为`ProductsThatCostMoreThan.cshtml`的新文件。

1.  根据以下代码修改内容：

```cs

@using Packt.Shared

@model IEnumerable<

产品

@{

字符串

标题=

“成本超过的产品”

+ ViewData[“MaxPrice”

];

ViewData[“Title”

] =标题；

}

<

h2

@title</

h2

@if（模型为空）

{

<

div

未找到产品。</

div

}

否则

{

<

表

类

=

“表”

<

标题

<

tr

<

th

类别名称</

th

<

th

供应商的公司名称</

th

<

th

产品名称</

th

<

th

单价</

th

<

th

库存单位</

th

</

tr

</

标题

<

tbody

@foreach（Product p in Model）

{

<

tr

<

td

@Html.DisplayFor（modelItem => p.Category.CategoryName）

</

td

<

td

@Html.DisplayFor（modelItem => p.Supplier.CompanyName）

</

td

<

td

@Html.DisplayFor（modelItem => p.ProductName）

</

td

<

td

@Html.DisplayFor（modelItem => p.UnitPrice）

</

td

<

td

@Html.DisplayFor（modelItem => p.UnitsInStock）

</

td

</

tr

}

<

tbody

</

表

}

```

1.  在`Views/Home`文件夹中，打开`Index.cshtml`。

1.  在访客计数下方和**产品**标题上方添加以下表单元素。这将为用户提供一个输入价格的表单。然后用户可以单击**提交**来调用仅显示高于输入价格的产品的操作方法：

```cs

<

h3

按价格查询产品</

h3

<

形式

asp-action

=

“成本超过的产品”

方法

=

“GET”

<

输入

名称

=

“价格”

占位符

=

“输入产品价格”

/>

<

输入

类型

=

“提交”

/>

</

形式

```

1.  启动网站。

1.  在主页上，在表单中输入一个价格，例如`50`，然后单击**提交**。

1.  注意输入的价格高于您输入的价格的产品的表格，如*图 15.9*所示：！[](img/Image00128.jpg)

图 15.9：成本超过 50 英镑的产品的过滤列表

1.  关闭 Chrome 并关闭 Web 服务器。

# 使用异步任务提高可伸缩性

构建桌面或移动应用程序时，可以使用多个任务（及其基础线程）来提高响应性，因为当一个线程忙于任务时，另一个可以处理与用户的交互。

任务及其线程在服务器端也可以很有用，特别是对于需要处理文件的网站，或者从商店或 Web 服务请求数据可能需要一段时间来响应。但它们对于 CPU 密集型的复杂计算是有害的，因此将这些留给正常的同步处理。

当 HTTP 请求到达 Web 服务器时，将从其池中分配一个线程来处理该请求。但是，如果该线程必须等待资源，那么它将被阻止处理更多的传入请求。如果网站接收的同时请求多于其池中的线程，则其中一些请求将响应服务器超时错误，**503 服务不可用**。

被锁定的线程没有做有用的工作。 *可以*处理其他请求中的一个，但前提是我们在网站中实现了异步代码。

每当一个线程在等待它需要的资源时，它可以返回到线程池并处理不同的传入请求，从而提高网站的可伸缩性，即增加它可以处理的同时请求的数量。

为什么不只是有一个更大的线程池？在现代操作系统中，池中的每个线程都有 1 MB 的堆栈。异步方法使用较小的内存量。它还消除了在池中创建新线程的需要，这需要时间。向池中添加新线程的速率通常是每两秒一个，与切换异步线程相比，这是一个很长的时间。

**良好实践**：使您的控制器操作方法异步化。

## 使控制器操作方法异步化

现有操作方法异步化很容易：

1.  修改`Index`操作方法为异步，返回一个任务，并等待异步方法调用以获取类别和产品，如下面的代码中所示：

```cs

公共

**异步**

**任务<IActionResult>**

索引

（）

{

HomeIndexViewModel model = new

（

访客计数=（新的

Random（）。下一个（1

，1001

），

类别=

**等待**

db.Categories.ToList

**异步**

（），

产品=

**等待**

db.Products.ToList

**异步**

（）

）;

返回

View（model）; //将模型传递给视图

}

```

1.  以类似的方式修改`ProductDetail`操作方法，如下面代码中所示：

```cs

公共

**异步**

**任务<IActionResult>**

产品详情

（

int

？id

）

{

如果

（！id.HasValue）

{

返回

BadRequest（“您必须在路由中传递产品 ID，例如，

/Home/ProductDetail/21"

）;

}

产品？模型=

**等待**

db.Products

.SingleOrDefault

**异步**

（p => p.ProductId == id);

如果

（模型== null

）

{

返回

未找到（“ProductId

{id}

未找到。”

）;

}

返回

View（model）; //将模型传递给视图，然后返回结果

}

```

1.  启动网站并注意网站的功能相同，但相信现在它将更好地扩展。

1.  关闭 Chrome 并关闭 Web 服务器。

# 练习和探索

通过回答一些问题来测试您的知识和理解，进行一些实践，并通过更深入的研究来探索本章的主题。

## 练习 15.1-测试您的知识

回答以下问题：

1.  在`Views`文件夹中创建具有特殊名称`_ViewStart`和`_ViewImports`的文件时，它们会做什么？

1.  默认的 ASP.NET Core MVC 路由中定义了三个段的名称是什么，它们代表什么，哪些是可选的？

1.  默认的模型绑定器是做什么的，它可以处理哪些数据类型？

1.  在像`_Layout.cshtml` 这样的共享布局文件中，如何输出当前视图的内容？

1.  在像`_Layout.cshtml` 这样的共享布局文件中，如何输出当前视图可以提供内容的部分，视图如何提供该部分的内容？

1.  在控制器的操作方法中调用`View`方法时，按照惯例会搜索哪些路径来查找视图？

1.  如何指示访问者的浏览器将响应缓存 24 小时？

1.  即使您不创建 Razor 页面，为什么可能要启用 Razor 页面？

1.  ASP.NET Core MVC 如何识别可以充当控制器的类？

1.  ASP.NET Core MVC 有哪些方面使得网站测试更容易？

## 练习 15.2 – 通过实现类别详细页面来实现 MVC 的练习

`Northwind.Mvc` 项目有一个显示类别的主页，但当单击`View`按钮时，网站返回`404 Not Found`错误，例如以下 URL：

`https://localhost:5001/category/1`

通过添加显示类别详细信息页面的功能来扩展`Northwind.Mvc`项目。

## 练习 15.3 – 通过理解和实现异步操作方法来练习提高可伸缩性

几年前，Stephen Cleary 在 MSDN 杂志上撰写了一篇关于实现 ASP.NET 异步操作方法的可伸缩性好处的优秀文章。相同的原则适用于 ASP.NET Core，但更甚，因为与文章中描述的旧 ASP.NET 不同，ASP.NET Core 支持异步过滤器和其他组件。

阅读以下链接中的文章：

[`docs.microsoft.com/en-us/archive/msdn-magazine/2014/october/async-programming-introduction-to-async-await-on-asp-net`](https://docs.microsoft.com/en-us/archive/msdn-magazine/2014/october/async-programming-introduction-to-async-await-on-asp-net)

## 练习 15.4 – 练习单元测试 MVC 控制器

控制器是网站业务逻辑运行的地方，因此通过单元测试测试该逻辑的正确性非常重要，就像您在*第四章*，*编写、调试和测试函数*中学到的那样。

为`HomeController`编写一些单元测试。

**良好实践**：您可以在以下链接中阅读有关如何对控制器进行单元测试的更多信息：[`docs.microsoft.com/en-us/aspnet/core/mvc/controllers/testing`](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/testing)

## 练习 15.5 – 探索主题

使用以下页面上的链接了解本章涵盖的主题：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-15---building-websites-using-the-model-view-controller-pattern`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-15---building-websites-using-the-model-view-controller-pattern)

# 摘要

在本章中，您学习了如何以易于进行单元测试的方式构建大型、复杂的网站，方法是通过注册和注入依赖服务，如数据库上下文和记录器，并且使用 ASP.NET Core MVC 更容易管理程序员团队。您学习了有关配置、身份验证、路由、模型、视图和控制器的知识。

在下一章中，您将学习如何构建和使用使用 HTTP 作为通信层的服务，也就是 Web 服务。
