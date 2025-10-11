# 8

# 使用最小API构建和保障Web服务

本章介绍使用ASP.NET Core最小API构建和保障Web服务。这包括实施保护Web服务免受攻击的技术以及身份验证和授权。

本章将涵盖以下主题：

+   使用ASP.NET Core最小API构建Web服务

+   使用CORS放宽同源安全策略

+   使用速率限制预防拒绝服务攻击

+   使用原生AOT改进启动时间和资源

+   理解身份服务

# 使用ASP.NET Core最小API构建Web服务

在ASP.NET Core的旧版本中，您会使用控制器构建Web服务，每个端点都有一个操作方法，有点像使用ASP.NET Core MVC和控制器以及模型构建网站，但没有视图。从.NET 6开始，您有另一个通常更好的选择：**ASP.NET Core** **最小API**。

## 最小API基于Web服务的优势

在ASP.NET Core的早期版本中，与替代Web开发平台相比，实现一个简单的Web服务需要大量的样板代码。例如，一个最小的`Hello World` Web服务实现，它有一个返回纯文本的单个端点，可以使用**Express.js**仅用九行代码实现，如下所示：

[PRE0]

在ASP.NET Core 5或更早版本中，这需要超过五十行代码！

使用ASP.NET Core 6或更高版本，现在只需五行代码和六行配置即可，如下所示的两个代码块：

[PRE1]

平台在项目文件中指定，隐式`using`语句SDK功能执行一些繁重的工作。默认情况下启用，如下所示，在以下标记中突出显示：

[PRE2]

**良好实践**：最小API的另一个好处是它们不使用动态生成的代码，与基于控制器的Web API不同。这使得它们可以使用原生AOT生成更小、更快的服务，这对于在容器中实现和托管微服务更有利。我们将在本章后面介绍原生AOT与最小API。尽可能使用最小API而不是控制器来实现您的Web服务。

## 理解最小API路由映射

`WebApplication`实例具有您可以调用的方法，用于将路由映射到lambda表达式或语句：

+   `MapGet`: 将路由映射到`GET`请求以检索实体。

+   `MapPost`: 将路由映射到`POST`请求以插入实体。

+   `MapPut`: 将路由映射到`PUT`请求以更新实体。

+   `MapPatch`: 将路由映射到`PATCH`请求以更新实体。

+   `MapDelete`: 将路由映射到`DELETE`请求以删除实体。

+   `MapMethods`: 将路由映射到任何其他HTTP方法或方法，例如`CONNECT`或`HEAD`。

例如，您可能希望将相对路径 `api/customers` 的 HTTP `GET` 请求映射到由 lambda 表达式或返回包含客户列表的 JSON 文档的函数定义的委托，以及插入和删除的等效映射，如下面的代码所示：

[PRE3]

您可能希望将相对路径 `api/customers` 的 HTTP `CONNECT` 请求映射到 lambda 语句块，如下面的代码所示：

[PRE4]

如果您有多个共享相同相对路径的端点，则可以定义一个 **路由组**。`MapGroup` 方法在 .NET 7 中引入：

[PRE5]

**更多信息**：您可以在以下链接中了解更多关于路由映射的信息：[https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/route-handlers](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/route-handlers).

## 理解参数映射

委托可以定义参数，这些参数可以自动设置。尽管大多数映射可以在不显式指定的情况下进行配置，但您可以选择使用属性来定义 ASP.NET Core 最小 API 应从何处设置参数值：

+   `[FromServices]`：参数将从已注册的依赖服务设置。

+   `[FromRoute]`：参数将从匹配的命名路由段设置。

+   `[FromQuery]`：参数将从匹配的命名查询字符串参数设置。

+   `[FromBody]`：参数将从 HTTP 请求体设置。

例如，要更新数据库中的实体，您需要从已注册的依赖服务中检索数据库上下文，将标识符作为查询字符串或路由段传递，以及请求体中的新实体，如下面的代码所示：

[PRE6]

## 理解返回值

最小 API 服务可以以一些常见的格式返回数据，如下表 8.1 所示：

| **类型** | **Lambda** |
| --- | --- |
| 纯文本 | `() => "Hello World!"``() => Results.Text("Hello World!")` |
| JSON 文档 | `() => new { FirstName = "Bob", LastName = "Jones" }``() => Results.Json(new { FirstName = "Bob", LastName = "Jones" })` |
| 带状态码的 `IResult` | `() => Results.Ok(new { FirstName = "Bob", LastName = "Jones" })``() => Results.NoContent()``() => Results.Redirect("new/path")``() => Results.NotFound()``() => Results.BadRequest()``() => Results.Problem()``() => Results.StatusCode(405)` |
| 文件 | `() => Results.File("/path/filename.ext")` |

表 8.1：最小 API 返回值的示例

## 记录 Minimal APIs 服务

您可以根据需要调用额外的方法多次，以指定可以从端点期望的返回类型和状态码，例如：

+   `Produces<T>(StatusCodes.Status200OK)`：当成功时，此路由返回包含类型 `T` 和状态码 `200` 的响应。

+   `Produces(StatusCodes.Status404NotFound)`：当找不到路由匹配项时，此路由返回空响应和状态码 404。

## 设置 ASP.NET Core Web API 项目

首先，我们将创建一个简单的 ASP.NET Core Web API 项目，稍后我们将使用各种技术来保护它，例如速率限制、CORS、身份验证和授权。

该 web 服务的 API 定义如下所示 *表 8.2*：

| **方法** | **路径** | **请求体** | **响应体** | **成功代码** |
| --- | --- | --- | --- | --- |
| `GET` | `/` | None | Hello World! | `200` |
| `GET` | `/api/products` | None | 库存 `Product` 对象数组 | `200` |
| `GET` | `/api/products/outofstock` | None | 已售罄 `Product` 对象数组 | `200` |
| `GET` | `/api/products/discontinued` | None | 已停产的 `Product` 对象数组 | `200` |
| `GET` | `/api/products/{id}` | None | `Product` 对象 | `200` |
| `GET` | `/api/products/{name}` | None | 包含名称的 `Product` 对象数组 | `200` |
| `POST` | `/api/products` | `Product` 对象（无 `Id` 值） | `Product` 对象 | `201` |
| `PUT` | `/api/products/{id}` | `Product` 对象 | None | `204` |
| `DELETE` | `/api/products/{id}` | None | None | `204` |

表 8.2：示例项目实现的 API 方法

让我们开始：

1.  使用您首选的代码编辑器创建一个名为 `Chapter08` 的新解决方案。

1.  添加一个如以下列表中定义的 Web API 项目：

    +   项目模板：**ASP.NET Core Web API** / `webapi`

    +   解决方案文件和文件夹：`Chapter08`

    +   项目文件和文件夹： `Northwind.WebApi.Service`

    +   **身份验证类型**：**无**。

    +   **配置 HTTPS**：已选择。

    +   **启用 Docker**：已清除。

    +   **启用 OpenAPI 支持**：已选择。

    +   **不使用顶级语句**：已清除。

    +   **使用控制器**：已清除。

    要使用 `dotnet new` 为预-.NET 8 SDK 创建使用最小 API 的 Web API 项目，您必须使用 `-minimal` 开关或 `--use-minimal-apis` 开关。对于 .NET 8 SDK，最小 API 是默认的，要使用控制器，您必须指定 `--use-controllers` 或 `-controllers` 开关。

    **警告！**如果您正在使用 JetBrains Rider，其用户界面可能还没有创建使用最小 API 的 Web API 项目的选项。我建议使用 `dotnet new` 创建项目，然后将项目添加到您的解决方案中。

1.  将您在 *第 3 章* 中创建的 Northwind 数据库上下文项目（针对 SQL Server）添加为项目引用，如下所示：

    [PRE7]

    路径不能有换行符。如果您没有完成在 *第 3 章* 中创建类库的任务，那么请从 GitHub 仓库下载解决方案项目。

1.  在项目文件中，将 `invariantGlobalization` 更改为 `false`，并将警告视为错误，如下所示：

    [PRE8]

    在 .NET 8 的 ASP.NET Core Web API 项目模板中，显式设置不变量全球化为 `true` 是新的。它旨在使网络服务不受文化限制，因此可以在世界任何地方部署并具有相同的行为。通过将此属性设置为 `false`，网络服务将默认为当前托管计算机的文化。你可以在以下链接中了解更多关于不变量全球化模式的信息：[https://github.com/dotnet/runtime/blob/main/docs/design/features/globalization-invariant-mode.md](https://github.com/dotnet/runtime/blob/main/docs/design/features/globalization-invariant-mode.md)

1.  在命令提示符或终端中，构建 `Northwind.WebApi.Service` 项目，以确保当前解决方案外部的实体模型类库项目被正确编译，如下所示（命令）：

    [PRE9]

1.  在 `Properties` 文件夹中，在 `launchSettings.json` 文件中，将名为 `https` 的配置文件的 `applicationUrl` 修改为使用端口 `5081`，如下所示（高亮显示）的配置：

    [PRE10]

    Visual Studio 2022 将读取此设置文件，如果 `launchBrowser` 设置为 `true`，则自动运行一个网络浏览器，然后导航到 `applicationUrl` 和 `launchUrl`。Visual Studio Code 和 `dotnet run` 不会这样做，因此你需要手动运行一个网络浏览器并手动导航到 [https://localhost:5081/swagger](https://localhost:5081/swagger)。

1.  在 `Program.cs` 文件中，删除关于天气服务的语句，并用导入命名空间以将 `NorthwindContext` 添加到配置服务的语句替换，如下所示（高亮显示）：

    [PRE11]

1.  添加一个名为 `WebApplication.Extensions.cs` 的新类文件。

    **良好实践**：不要在 `Program.cs` 文件中添加数百行代码，而是为在最小 API 中配置的常见类型定义扩展方法，如 `WebApplication` 和 `IServiceCollection`。

1.  在 `WebApplication.Extensions.cs` 文件中，导入用于控制 HTTP 结果、将参数绑定到依赖服务以及与 `Northwind` 实体模型一起工作的命名空间，然后定义一个扩展方法为 `WebApplication` 类配置对所有记录在我们 API 表中的 HTTP `GET` 请求的响应，如下所示：

    [PRE12]

1.  在 `WebApplication.Extensions.cs` 文件中，为 `WebApplication` 类定义一个扩展方法，以配置对 API 表中记录的 HTTP `POST` 请求的响应，如下所示：

    [PRE13]

1.  在 `WebApplication.Extensions.cs` 文件中，为 `WebApplication` 类定义一个扩展方法，以配置对 API 表中记录的 HTTP `PUT` 请求的响应，如下所示：

    [PRE14]

1.  在 `WebApplication.Extensions.cs` 文件中，为 `WebApplication` 类定义一个扩展方法，以配置对 API 表中记录的 HTTP `DELETE` 请求的响应，如下所示：

    [PRE15]

1.  在 `Program.cs` 文件中，导入你刚刚定义的扩展方法所使用的命名空间，如下所示：

    [PRE16]

1.  在 `Program.cs` 中，在调用 `app.Run()` 之前，调用你的自定义扩展方法来映射 `GET`、`POST`、`PUT` 和 `DELETE` 请求，注意当你请求所有产品时，你可以覆盖默认的 10 个实体页面大小，如下面的代码所示：

    [PRE17]

1.  在 `Program.cs` 中，确保文件中的最后一个语句运行了网络应用，如下面的代码所示：

    [PRE18]

## 使用 Swagger 测试网络服务

现在我们可以启动网络服务，使用 Swagger 查看其文档，并进行基本的手动测试：

1.  如果你的数据库服务器没有运行（例如，因为你正在 Docker、虚拟机或云中托管它），那么请确保启动它。

1.  使用 `https` 配置文件启动网络服务项目：

    +   如果你正在使用 Visual Studio 2022，那么在下拉列表中选择 **https** 配置文件，然后导航到 **Debug** | **Start Without Debugging** 或按 *Ctrl* + *F5*。网页浏览器应该会自动导航到 Swagger 文档网页。

    +   如果你正在使用 Visual Studio Code，那么输入命令 `dotnet run --launch-profile https`，手动启动一个网页浏览器，并导航到 Swagger 文档网页：[https://localhost:5081/swagger](https://localhost:5081/swagger)。

    在 Windows 上，如果需要，你必须将 Windows Defender 防火墙设置为允许访问你的本地网络服务。

1.  在控制台或终端中，注意有关你的网络服务的信息，如下面的输出所示：

    [PRE19]

1.  在你的网页浏览器中，注意 Swagger 文档，如图 *8.1* 所示：

![](img/B19587_08_01.png)

图 8.1：Northwind Web API 服务的 Swagger 文档

1.  点击 **GET /api/products** 以展开该部分。

1.  点击 **Try it out** 按钮，注意可选的查询字符串参数名为 **page**，然后点击 **Execute** 按钮。

1.  注意响应包括库存中且未停售的前十种产品：`1`、`2`、`3`、`4`、`6`、`7`、`8`、`10`、`11` 和 `12`。

1.  对于 **page** 参数，输入 `3`，然后点击 **Execute** 按钮。

1.  注意响应包括库存中且未停售的第三页的十种产品：`25`、`26`、`27`、`30`、`32`、`33`、`34`、`35`、`36` 和 `37`。

1.  点击 **GET /api/products** 以折叠该部分。

1.  尝试执行 **GET /api/products/outofstock** 路径，并注意它返回了一种产品，**31 Gorgonzola Telino**，库存为零且未停售。

1.  尝试执行 **GET /api/products/discontinued** 路径，并注意它返回了八种产品，`5`、`9`、`17`、`24`、`28`、`29`、`42` 和 `53`，它们的 `Discontinued` 属性都设置为 `true`。

1.  点击 **GET /api/products/{id}** 以展开该部分。

1.  点击 **Try it out**，输入所需的 **id** 参数为 `77`，点击 **Execute**，并注意响应包含名为 **Original Frankfurter grüne Soße** 的产品，如下面的 JSON 文档所示：

    [PRE20]

1.  点击 **GET /api/products/{id}** 以折叠该部分。

1.  点击 **GET /api/products/{name}** 来展开该部分。

1.  点击 **尝试一下**，输入所需的 **name** 参数为 `man`，点击 **执行**，并注意响应包含名为 **Queso Manchego La Pastora** 和 **Manjimup Dried Apples** 的产品。

1.  保持 Web 服务运行。

## 使用代码编辑器工具测试 Web 服务

使用 Swagger 用户界面测试 Web 服务可能会变得繁琐。更好的工具是名为 **REST Client** 的 Visual Studio Code 扩展或 Visual Studio 2022 版本 17.6 或更高版本提供的 **Endpoints Explorer** 和 `.http` 文件支持。

**更多信息**：你可以在以下链接了解 Visual Studio 2022 及其 HTTP 编辑器：[https://learn.microsoft.com/en-us/aspnet/core/test/http-files](https://learn.microsoft.com/en-us/aspnet/core/test/http-files)。

JetBrains Rider 有一个类似的工具窗口名为 **Endpoints**。

如果你正在使用 JetBrains Rider，你可以在以下链接了解其 HTTP 文件工具：[https://www.jetbrains.com/help/rider/Http_client_in__product__code_editor.html](https://www.jetbrains.com/help/rider/Http_client_in__product__code_editor.html)。它与另外两个代码编辑器略有不同。特别是，Rider 处理设置变量的方式更为繁琐，如图所示：[https://www.jetbrains.com/help/rider/Exploring_HTTP_Syntax.html#example-working-with-environment-files](https://www.jetbrains.com/help/rider/Exploring_HTTP_Syntax.html#example-working-with-environment-files)。你可能更喜欢使用带有 REST Client 扩展的 Visual Studio Code 来处理这一部分。

让我们看看这些工具如何帮助我们测试 Web 服务：

1.  确保你已经安装了 Web 服务测试工具：

    +   如果你正在使用 Visual Studio 2022，请确保你安装的版本为 17.6 或更高版本（2023 年 5 月发布）。

    +   如果你正在使用 Visual Studio Code，请确保你已经安装了由 Huachao Mao 开发的 REST Client 扩展（`humao.rest-client`）。

    +   如果你正在使用 Visual Studio 2022，导航到 **视图** | **其他窗口** | **Endpoints Explorer**，并注意当前项目正在扫描潜在的 Web API 端点，如图 8.2 所示。

1.  在你偏好的代码编辑器中，使用 `https` 配置启动 `Northwind.WebApi.Service` 项目（如果尚未运行），并保持其运行。

1.  在 `apps-services-net8` 文件夹中，如果尚不存在，创建一个 `HttpRequests` 文件夹。

1.  在 `HttpRequests` 文件夹中，创建一个名为 `webapi-get-products.http` 的文件，并修改其内容以声明一个变量来保存 Web API 服务产品端点的基址，以及一个获取前 10 页产品的请求，如下面的代码所示：

    [PRE21]

    **良好实践**：REST 客户端在请求开始时不要求使用 `GET`，因为它会默认假设为 `GET`。但截至写作时，Visual Studio 的 HTTP 编辑器需要显式指定 `GET`。因此，我建议您为所有工具指定 HTTP 方法，并且我将为我的所有 `.http` 文件这样做。

1.  点击 **发送请求**，并注意响应与 Swagger 返回的相同，它是一个包含前十个库存且未停售产品的 JSON 文档响应，如 Visual Studio 2022 中的 *图 8.2* 和 Visual Studio Code 中的 *图 8.3* 所示：

![](img/B19587_08_02.png)

图 8.2：Visual Studio 2022 从 Web API 服务获取产品

![](img/B19587_08_03.png)

图 8.3：Visual Studio Code REST 客户端从 Web API 服务获取产品

1.  在 `webapi-get-products.http` 文件中，通过 `###` 分隔添加更多请求，如下所示：

    [PRE22]

    您可以通过点击绿色三角形“播放”按钮、右键单击并选择 **发送请求** 或按 *Ctrl* + *Alt* + *S* 在 Visual Studio 2022 中执行 HTTP 请求。在 Visual Studio Code 中，点击每个查询上方的 **发送请求**，或导航到 **视图** | **命令面板** 并选择 **Rest Client: 发送请求**，或使用其快捷键（在 Windows 上为 *Ctrl* + *Alt* + *R*）。

1.  在 `HttpRequests` 文件夹中，创建一个名为 `webapi-insert-product.http` 的文件，并修改其内容以包含一个用于插入新产品的 **POST** 请求，如下所示：

    [PRE23]

1.  点击 **发送请求**，并注意响应表明新产品已成功添加，因为状态码为 `201`，其位置包括其产品 ID，如 *图 8.4* 所示：![](img/B19587_08_04.png)

    图 8.4：REST 客户端通过调用 Web API 服务插入新产品

    原本，Northwind 数据库中有 77 个产品。下一个产品 ID 将是 78。实际分配的自动产品 ID 将取决于您是否之前添加了其他产品，因此您分配的数字可能更高。

1.  在 `HttpRequests` 文件夹中，创建一个名为 `webapi-update-product.http` 的文件，并修改其内容以包含一个用于更新产品 ID 为 `78`（或分配给您的 `Harry's Hamburgers` 的任何数字）的 `PUT` 请求，其中包含不同的每单位数量、单价和库存单位，如下所示：

    [PRE24]

1.  发送请求并注意您应该收到一个 204 状态码的响应，表示更新成功。

1.  通过执行针对产品 ID 的 `GET` 请求来确认产品是否已更新。

1.  在 `HttpRequests` 文件夹中，创建一个名为 `webapi-delete-product.http` 的文件，并修改其内容以包含针对新产品的 `DELETE` 请求，如下所示：

    [PRE25]

1.  注意成功响应，如 *图 8.5* 所示：

![](img/B19587_08_05.png)

图 8.5：使用 Web API 服务删除产品

1.  再次发送请求，并注意响应包含 404 状态码，因为产品已经被删除。

1.  关闭网络服务器。

## 排除来自 OpenAPI 文档的路径

有时您可能想要一个可以工作但不显示在 Swagger 文档中的路径。让我们看看如何从 Swagger 文档网页中移除返回纯文本 `Hello World!` 响应的服务基本地址：

1.  在 `WebApplication.Extensions.cs` 中，对于返回 `Hello World` 的根路径，将其从 OpenAPI 文档中排除，如下面的代码所示，高亮显示：

    [PRE26]

1.  使用 `https` 配置启动 `Northwind.WebApi.Service` 项目，不进行调试，并注意路径现在没有文档记录。

我们使用 ASP.NET Core Minimal APIs 实现了一个工作的网络服务。现在让我们攻击它！（这样我们可以学习如何防止这些攻击。）

## Visual Studio 2022 为 Minimal APIs 提供的脚手架

学习如何从头开始使用最小 API 实现服务非常重要，这样您可以正确理解它。但是一旦您知道如何手动操作，这个过程可以自动化，并且可以为您编写样板代码，尤其是如果您正在构建一个包装 EF Core 实体模型的网络 API。

例如，Visual Studio 2022 有一个名为 **使用 Entity Framework 的 API 读写端点**的项目项模板，允许您选择：

+   一个实体模型类，例如 `Customer`。

+   一个端点类，它将包含所有映射方法。

+   一个由 `DbContext` 派生的类，例如 `NorthwindContext`。

+   一个数据库提供程序，例如 SQLite 或 SQL Server。

**更多信息**：您可以在以下链接中了解更多关于此项目项模板的信息：[https://devblogs.microsoft.com/visualstudio/web-api-development-in-visual-studio-2022/#scaffolding-in-visual-studio/](https://devblogs.microsoft.com/visualstudio/web-api-development-in-visual-studio-2022/#scaffolding-in-visual-studio/)。

# 使用 CORS 放宽同源安全策略

现代网络浏览器支持多个标签页，用户可以高效地同时访问多个网站。如果在一个标签页中执行的代码可以访问另一个标签页中的资源，那么这可能是一个攻击向量。

所有网络浏览器都实现了名为 **同源策略** 的安全功能。这意味着只有来自同一源头的请求被允许。例如，如果一段 JavaScript 代码是从托管网络服务或 `<iframe>` 的同一源头提供的，那么该 JavaScript 可以调用该服务并访问 `<iframe>` 中的数据。如果请求来自不同的源头，则请求失败。但“同源”是什么意思呢？

原因定义为：

+   **方案**也称为协议，例如，`http` 或 `https`。

+   **端口**，例如，`801` 或 `5081`。`http` 的默认端口是 `80`，而 `https` 的默认端口是 `443`。

+   **主机/域名/子域名**，例如，`www.example.com`，`www.example.net`，`example.com`。

如果源是 `https://www.example.com/about-us/`，则以下不是同一个源：

+   不同的方案：`http://www.example.com/about-us/`

+   不同的主机/域名：`https://www.example.co.uk/about-us/`

+   不同的子域名：`https://careers.example.com/about-us/`

+   不同的端口：`https://www.example.com:444/about-us/`

是网页浏览器在发起请求时自动设置 `Origin` 标头的。这不能被覆盖。

**警告**！同源策略不适用于来自非网页浏览器的任何请求，因为在那些情况下，程序员仍然可以更改 `Origin` 标头。如果您创建了一个控制台应用程序或甚至是一个使用 .NET 类如 `HttpClient` 来发起请求的 ASP.NET Core 项目，则同源策略不适用，除非您明确设置 `Origin` 标头。

让我们看看从具有不同源的网页和 .NET 应用程序调用 Web 服务的示例。

## 配置 Web 服务的 HTTP 日志记录

首先，让我们为 Web 服务启用 HTTP 日志记录并配置它以显示请求的源：

1.  在 `Northwind.WebApi.Service` 项目中，添加一个名为 `IServiceCollection.Extensions.cs` 的新类文件。

1.  在 `IServiceCollection.Extensions.cs` 文件中，导入用于控制哪些 HTTP 字段被记录的命名空间，然后为 `IServiceCollection` 接口定义一个扩展方法以添加 HTTP 日志记录，包括 `Origin` 标头以及包括响应体在内的所有字段，如下所示：

    [PRE27]

1.  在 `Program.cs` 文件中，在调用 `builder.Build()` 之前，添加一条语句以添加自定义 HTTP 日志记录，如下所示：

    [PRE28]

1.  在 `Program.cs` 文件中，在调用 `UseHttpsRedirection()` 之后，添加一条语句以使用 HTTP 日志记录，如下所示：

    [PRE29]

1.  在 `appsettings.Development.json` 文件中，添加一个条目以设置 HTTP 日志记录级别为 `Information`，如下所示（高亮显示）：

    [PRE30]

**良好实践**：JSON 规范不允许注释，但带有注释的 JSON 格式允许。您可以使用 JavaScript 风格的注释 `//` 或 `/* */`。您可以在以下链接中了解更多信息：[https://code.visualstudio.com/docs/languages/json#_json-with-comments](https://code.visualstudio.com/docs/languages/json#_json-with-comments)。如果您使用的是挑剔的代码编辑器，只需删除我添加的注释即可。

## 创建网页 JavaScript 客户端

接下来，让我们创建一个网页客户端，它将尝试在不同的端口上使用 JavaScript 调用 Web 服务：

1.  使用您首选的代码编辑器添加一个新项目，如下列表所示：

    +   项目模板：**ASP.NET Core Web 应用程序（模型-视图-控制器）** / `mvc`

    +   解决方案文件和文件夹：`Chapter08`

    +   项目文件和文件夹：`Northwind.WebApi.Client.Mvc`

    +   其他 Visual Studio 2022 选项：

        +   **认证类型**：**无**。

        +   **配置为 HTTPS**：已选择。

        +   **启用 Docker**：已清除。

        +   **不要使用顶级语句**：已清除。

        +   在Visual Studio 2022中，配置启动项目为当前选择。

1.  在`Northwind.WebApi.Client.Mvc`项目中，在`Properties`文件夹中，在`launchSettings.json`文件中，将`https`配置文件的`applicationUrl`更改为使用端口`5082`，如下所示：

    [PRE31]

1.  在`Northwind.WebApi.Client.Mvc`项目文件中，将警告视为错误。

1.  在`Views/Home`文件夹中，在`Index.cshtml`中，将现有的标记替换为下面的标记，该标记包含一个指向尚未定义的路由的链接，用于定义一个文本框和按钮，以及一个JavaScript块，该块调用Web服务以获取包含部分名称的产品，如下所示代码：

    [PRE32]

1.  使用`https`配置文件启动`Northwind.WebApi.Service`项目，不进行调试。

1.  使用`https`配置文件启动`Northwind.WebApi.Client.Mvc`项目，不进行调试。

    如果你正在使用Visual Studio Code，那么浏览器将不会自动启动。启动Chrome，然后导航到[https://localhost:5082](https://localhost:5082)。

1.  在Chrome浏览器中，显示**开发者工具**和**控制台**。

1.  在**使用JavaScript的**产品网页中，在文本框中输入`man`，点击**获取产品**按钮，并注意错误，如下所示输出和*图8.6*：

    [PRE33]

![](img/B19587_08_06.png)

图8.6：Chrome开发者工具控制台中的CORS错误

1.  在`Northwind.WebApi.Service`项目的命令提示符或终端中，注意请求的HTTP日志，以及`Host`在不同的端口号上，与`Origin`不同，因此它们不是同源，如下所示突出显示的输出：

    [PRE34]

1.  还请注意输出显示Web服务确实执行了数据库查询，并将产品以JSON文档响应返回给浏览器，如下所示输出：

    [PRE35]

    虽然浏览器收到了包含请求数据的响应，但是浏览器通过拒绝向JavaScript显示HTTP响应来强制执行同源策略。Web服务不是通过CORS“安全”的。

1.  关闭浏览器和关闭Web服务器。

## 创建.NET客户端

接下来，让我们创建一个.NET客户端来访问Web服务，以查看同源策略不适用于非Web浏览器：

1.  在`Northwind.WebApi.Client.Mvc`项目中，添加对实体模型项目的引用，以便我们可以使用`Product`类，如下所示标记：

    [PRE36]

1.  在命令提示符或终端中通过输入以下命令构建`Northwind.WebApi.Client.Mvc`项目：`dotnet build`。

1.  在`Northwind.WebApi.Client.Mvc`项目中，在`Program.cs`中，导入用于处理HTTP头部的命名空间，如下所示代码：

    [PRE37]

1.  在`Program.cs`中，在调用`builder.Build()`之前，添加语句来配置HTTP客户端工厂以调用Web服务，如下所示代码：

    [PRE38]

1.  在`Controllers`文件夹中，在`HomeController.cs`中，导入实体模型的命名空间，如下所示代码：

    [PRE39]

1.  在`HomeController.cs`中，添加语句将注册的HTTP客户端工厂存储在私有的`readonly`字段中，如下代码所示高亮显示：

    [PRE40]

1.  在`HomeController.cs`中，添加一个名为`Products`的异步操作方法，该方法将使用HTTP工厂请求包含在自定义MVC路由中作为可选`name`参数输入的值的产品的名称，如下代码所示：

    [PRE41]

1.  在`Views/Home`文件夹中，添加一个名为`Products.cshtml`的新文件。（Visual Studio 2022项目项模板命名为**Razor View - Empty**。JetBrains Rider项目项模板命名为**Razor MVC View**。）

1.  在`Products.cshtml`中，修改其内容以输出一个表格，显示与在文本框中输入的产品名称部分匹配的产品，如下标记所示：

    [PRE42]

1.  使用无调试的`https`配置启动`Northwind.WebApi.Service`项目。

1.  使用无调试的`https`配置启动`Northwind.WebApi.Client.Mvc`项目。

1.  在主页上，点击链接跳转到**使用.NET的产品**，并注意表中显示了前十个库存且未停售的产品，从**Chai**到**Queso Manchego La Pastora**。

1.  在文本框中输入`man`，点击**Get Products**，注意表中显示了两个产品，如图8.7所示：![](img/B19587_08_07.png)

    图8.7：使用.NET从Web服务获取两个产品

    是.NET HTTP客户端在调用Web服务，因此不适用同源策略。如果你像之前一样在命令行或终端中检查日志，你会看到端口不同，但这并不重要。

1.  点击产品名称之一，直接向该Web服务请求单个产品并注意响应，如下文档所示：

    [PRE43]

1.  关闭浏览器并关闭Web服务器。

## 理解CORS

**跨源资源共享**（**CORS**）是一种基于HTTP头的功能，它要求浏览器在特定场景下禁用其同源安全策略。HTTP头指示哪些源应该被允许，除了同源之外。

让我们在Web服务中启用CORS，以便它可以发送额外的头信息，告知浏览器它允许从不同的源访问资源：

1.  在`Northwind.WebApi.Service`项目中，在`WebApplication.Extensions.cs`中，添加一个扩展方法以向Web服务添加CORS支持，如下代码所示：

    [PRE44]

1.  在`Program.cs`中，在创建`builder`之后，调用自定义扩展方法以添加CORS支持，如下代码所示：

    [PRE45]

1.  在`Program.cs`中，在调用`UseHttpLogging`之后，在映射`GET`请求之前，添加一个语句来使用CORS策略，如下代码所示：

    [PRE46]

1.  使用无调试的`https`配置启动`Northwind.WebApi.Service`项目。

1.  使用无调试的`https`配置启动`Northwind.WebApi.Client.Mvc`项目。

1.  显示**开发者工具**和**控制台**。

1.  在主页上的文本框中输入 `man`，点击 **Get Products**，注意控制台显示了来自 web 服务的 JSON 文档，并且表格中填充了两个产品，如图 8.8 所示：

![图片](img/B19587_08_08.png)

图 8.8：使用 JavaScript 向 web 服务发起成功的跨源请求

1.  关闭浏览器并关闭 web 服务器。

## 为特定端点启用 CORS

在上一个示例中，我们为整个 web 服务启用了相同的 CORS 策略。你可能需要在端点级别进行更精细的控制：

1.  在 `Northwind.WebApi.Service` 项目中，在 `Program.cs` 文件中，将 `UseCors` 调用的策略名称指定改为不指定，如下所示，高亮显示的代码：

    [PRE47]

1.  在 `WebApplication.Extensions.cs` 文件中，在获取包含部分产品名称的产品时 `MapGet` 调用的末尾，添加一个 `RequiresCors` 调用，如下所示，高亮显示的代码：

    [PRE48]

1.  使用 `https` 配置文件启动 `Northwind.WebApi.Service` 项目，且不进行调试。

1.  使用 `https` 配置文件启动 `Northwind.WebApi.Client.Mvc` 项目，且不进行调试。

1.  显示 **开发者工具** 和 **控制台**。

1.  在主页上的文本框中输入 `cha`，点击 **Get Products**，注意控制台显示了来自 web 服务的 JSON 文档，并且表格中填充了三个产品。

1.  关闭浏览器并关闭 web 服务器。

## 理解其他 CORS 策略选项

你可以控制以下内容：

+   允许的来源，例如，`https://*.example.com/`。

+   允许的 HTTP 方法，例如，`GET`、`POST`、`DELETE` 等等。

+   允许的 HTTP 请求头，例如，`Content-Type`、`Content-Language`、`x-custom-header` 等等。

+   暴露 HTTP 响应头，意味着在响应中包含哪些未加密的头信息（因为默认情况下，响应头会被加密），例如，`x-custom-header`。

你可以在以下链接中了解更多关于 CORS 策略的选项：[https://learn.microsoft.com/en-us/aspnet/core/security/cors#cors-policy-options](https://learn.microsoft.com/en-us/aspnet/core/security/cors#cors-policy-options)

既然你知道 CORS 并不能保护 web 服务，那么让我们看看一种有用的技术，它可以防止一种常见的攻击形式。

# 使用速率限制来防止拒绝服务攻击

**拒绝服务**（**DoS**）攻击是一种恶意尝试通过大量请求来中断 web 服务的攻击。如果请求都来自同一个地方，例如，同一个 IP 地址，那么一旦检测到攻击，就相对容易切断它们。但这些攻击通常作为来自许多位置的 **分布式拒绝服务**（**DDoS**）攻击来实施，因此你无法将攻击者与真正的客户端区分开来。

另一种方法是针对所有人应用速率限制，但允许真正的已识别客户端有更多的请求通过。

真实客户端应仅发出它们所需的最低请求。合理的数量将取决于您的服务。防止 DDoS 攻击的一种方法是对任何客户端每分钟允许的请求数量进行限制。

这种技术不仅有助于防止攻击。即使是真正的客户端也可能意外地发出过多的请求，或者对于商业 Web 服务，您可能希望根据不同的速率收取不同的费用，例如在控制订阅时。现在，从 Twitter/X 到 Reddit 的商业 Web 服务现在对访问其 Web API 收取了大量的费用。

当客户端超过设定的请求速率限制时，客户端应收到 `429 Too Many Requests` 或 `503 Service Unavailable` 的 HTTP 响应。

**良好实践**：如果您需要构建一个大规模可扩展的 Web 服务并保护其 API，您应该使用像 Azure API Management 这样的云服务，而不是尝试实现自己的速率限制。您可以在以下链接中了解更多信息：[https://learn.microsoft.com/en-us/azure/api-management/](https://learn.microsoft.com/en-us/azure/api-management/)。

## 使用 AspNetCoreRateLimit 包进行速率限制

`AspNetCoreRateLimit` 是一个针对 .NET 6 或更高版本的第三方包，它提供基于 IP 地址或客户端 ID 的灵活速率限制中间件：

1.  在 `Northwind.WebApi.Service` 项目中，添加对 `AspNetCoreRateLimit` 包的引用，如下所示：

    [PRE49]

1.  构建包含 `Northwind.WebApi.Service` 项目的解决方案以恢复包。

1.  在 `appsettings.Development.json` 中，添加默认速率限制选项和客户端特定策略的配置，如下所示的高亮配置：

    [PRE50]

    注意：

    +   `EnableEndpointRateLimiting` 设置为 `false`，意味着所有端点将共享相同的规则。

    +   如果客户端需要标识自己，它可以设置一个名为 `X-Client-Id` 的头，并将其设置为唯一的 `string` 值。

    +   如果客户端达到速率限制，服务将开始向该客户端返回 `429` 状态码响应。

    +   两个端点将不会被全局速率限制，因为它们位于端点白名单中。一个端点是获取许可证，另一个端点是检查服务状态。我们实际上不会实现这些功能，并且您可能希望对这些端点应用不同的速率限制；否则，有人可能会调用它们来使您的服务器崩溃。

    +   两个客户端 ID，名为 `dev-id-1` 和 `dev-id-2`，将不会被速率限制，因为它们位于客户端白名单中。这些可能是仅供内部开发者使用的特殊客户端账户，这些账户不会在组织外部共享。

    +   配置了两个通用（默认）规则：第一个规则每10秒设置2个请求的速率限制，第二个规则每12小时设置100个请求的速率限制。

    +   配置了两个比默认规则更宽松的客户端特定规则：对于名为 `console-client-abc123` 的客户端 ID，允许每 10 秒内最多发送 5 个请求，每 12 小时最多发送 250 个请求。

1.  在 `IServiceCollection.Extensions.cs` 中，导入用于处理速率限制选项的命名空间，如下所示：

    [PRE51]

1.  在 `IServiceCollection.Extensions.cs` 中，定义一个扩展方法，从应用程序设置中加载速率限制配置并设置速率限制选项，如下所示：

    [PRE52]

1.  在 `Program.cs` 中，在创建 `builder` 之后，添加语句从应用程序设置中加载速率限制配置并设置速率限制选项，如下所示：

    [PRE53]

1.  在 `IServiceCollection.Extensions.cs` 中，在配置 HTTP 记录的调用中，添加一条语句以允许两个速率限制头不被截断，如下所示（高亮显示）：

    [PRE54]

1.  在 `WebApplication.Extensions.cs` 中，导入用于处理速率限制策略存储的命名空间，如下所示：

    [PRE55]

1.  在 `WebApplication.Extensions.cs` 中，添加语句以定义一个扩展方法来初始化客户端策略存储，这意味着从配置中加载策略，然后使用客户端速率限制，如下所示：

    [PRE56]

1.  在 `Program.cs` 中，在调用 `UseHttpLogging` 之后，添加一个调用以使用客户端速率限制，如下所示：

    [PRE57]

## 创建速率限制控制台客户端

现在，我们可以创建一个控制台应用程序，它将成为 Web 服务的客户端：

1.  使用您首选的代码编辑器将一个新的控制台应用程序添加到 `Chapter08` 解决方案中，命名为 `Northwind.WebApi.Client.Console`。

1.  在 `Northwind.WebApi.Client.Console` 项目中，将警告视为错误，全局和静态导入 `System.Console` 类，并添加对实体模型项目的引用，如下所示：

    [PRE58]

1.  在命令提示符或终端中构建 `Northwind.WebApi.Client.Console` 项目，以编译引用的项目并将其实例复制到适当的 `bin` 文件夹。

1.  在 `Northwind.WebApi.Client.Console` 项目中，添加一个名为 `Program.Helpers.cs` 的新类文件。

1.  在 `Program.Helpers.cs` 中，添加语句以定义一个方法，用于 `partial Program` 类，以便以前景色写入一些文本，如下所示：

    [PRE59]

1.  在 `Program.cs` 中，删除现有的语句。添加语句提示用户输入客户端名称以识别它，然后创建一个 HTTP 客户端，每秒向 Web 服务发送一次请求以获取产品第一页，直到用户按下 *Ctrl* + *C* 停止控制台应用程序，如下所示：

    [PRE60]

1.  如果您的数据库服务器没有运行（例如，因为您正在 Docker、虚拟机或云中托管它），请确保启动它。

1.  使用 `https` 配置不带调试启动 `Northwind.WebApi.Service` 项目。

1.  不带调试启动 `Northwind.WebApi.Client.Console` 项目。

1.  在控制台应用程序中，按 *Enter* 键生成基于 GUID 的客户端 ID。

1.  再次不进行调试地使用 `https` 配置启动 `Northwind.WebApi.Client.Console` 项目，以便我们有两个客户端。

1.  在控制台应用程序中，按 *Enter* 键生成基于 GUID 的客户端 ID。

1.  注意，每个客户端在开始接收 `429` 状态代码之前可以发出两个请求，如下所示，并在 *图 8.9* 中显示：

    [PRE61]

![](img/B19587_08_09.png)

图 8.9：超出其 Web 服务速率限制的控制台应用程序

1.  停止两个控制台应用程序。保持 Web 服务运行。

1.  在 Web 服务命令行中，注意显示每个来自控制台客户端的请求的 HTTP 日志，该请求以名为 `X-Client-Id` 的标头发送客户端 ID，请求被阻止，因为该客户端已超出配额，以及包含客户端应在重试前等待的秒数的名为 `Retry-After` 的标头的响应，如下所示，代码中已突出显示：

    [PRE62]

1.  在 `Northwind.WebApi.Client.Console` 项目中，在 `Program.cs` 文件中，在将错误信息以深红色写入控制台之前，添加语句读取 `Retry-After` 标头以获取等待的秒数，如下所示，代码中已突出显示：

    [PRE63]

    注意 `waitFor` 变量是从 `Retry-After` 标头值设置的。这随后用于使用异步延迟暂停控制台应用程序，如下所示，代码中已突出显示：

    `await Task.Delay(TimeSpan.FromSeconds(waitFor));`

1.  不进行调试地启动 `Northwind.WebApi.Client.Console` 项目。

1.  在控制台应用程序中，按 *Enter* 键生成基于 GUID 的客户端 ID。

1.  注意控制台应用程序现在将合理地等待建议的秒数，然后再进行对服务的下一次调用，如下所示，代码中已突出显示：

    [PRE64]

1.  停止并重新启动 `Northwind.WebApi.Client.Console` 项目，不进行调试。

1.  在控制台应用程序中，输入名称 `dev-id-1`，并注意速率限制不适用于此控制台应用程序客户端。这可能是一个内部开发人员的特殊账户。

1.  停止并重新启动 `Northwind.WebApi.Client.Console` 项目，不进行调试。

1.  在控制台应用程序中，输入名称 `console-client-abc123`，并注意此控制台应用程序客户端 ID 的速率限制不同，如下所示，代码中已突出显示：

    [PRE65]

## 使用 ASP.NET Core 中间件进行速率限制

ASP.NET Core 7 引入了它自己的基本速率限制中间件，最初作为单独的 NuGet 包分发，但现在包含在 ASP.NET Core 中。它依赖于另一个 Microsoft 包，`System.Threading.RateLimiting`。它不如第三方包功能丰富，本书中不会介绍它，尽管我已在以下链接处编写了一个仅在线的章节：

[https://github.com/markjprice/apps-services-net8/blob/main/docs/ch08-rate-limiting.md](https://github.com/markjprice/apps-services-net8/blob/main/docs/ch08-rate-limiting.md)

你可以在以下链接中了解 ASP.NET Core 速率限制器：[https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit](https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit)。

保护你的网络服务免受攻击很重要。那么，提高你的网络服务性能呢？我们能做些什么？

# 使用原生 AOT 提高启动时间和资源

原生 AOT 生成的应用和服务具有：

+   **自包含**，意味着它们可以在未安装 .NET 运行时系统的计算机上运行。

+   **提前编译 (AOT) 为原生代码**，意味着更快的启动时间和可能更小的内存占用。当你有很多实例（例如，在部署大规模可扩展的微服务时）频繁停止和启动时，这可以产生积极的影响。

原生 AOT 在发布时将**中间代码**（**IL**）编译为原生代码，而不是在运行时使用**即时编译器**（**JIT**）进行编译。但原生 AOT 应用和服务必须针对特定的运行时环境，如 Windows x64 或 Linux ARM。

由于原生 AOT 在发布时发生，因此在代码编辑器中调试和实时工作在项目上时，它使用运行时 JIT 编译器，而不是原生 AOT，即使你在项目中启用了 AOT！但与原生 AOT 不兼容的一些功能将被禁用或抛出异常，并启用源分析器以显示有关潜在代码不兼容性的警告。

## 原生 AOT 的限制

原生 AOT 存在限制，以下列出了一些：

+   不允许动态加载程序集。

+   不允许运行时代码生成，例如使用 `System.Reflection.Emit`。

+   它需要修剪，这有其自身的限制。

+   程序集必须是自包含的，因此它们必须嵌入它们调用的任何库，这增加了它们的大小。

虽然你的应用和服务可能没有使用上述功能，但 .NET 本身的许多主要部分都使用了。例如，ASP.NET Core MVC（包括使用控制器的 Web API 服务）和 EF Core 通过运行时代码生成来实现其功能。

.NET 团队正在努力工作，尽可能快地将尽可能多的 .NET 与原生 AOT 兼容，.NET 8 仅在您使用最小 API 时包括对 ASP.NET Core 的基本支持，并且对 EF Core 没有支持。

我的猜测是 .NET 9 将包括对 ASP.NET Core MVC 和 EF Core 部分的支持，但可能需要到 .NET 10 我们才能自信地使用大多数 .NET，并知道我们可以使用原生 AOT 构建我们的应用和服务以获得这些好处。

原生 AOT 发布过程包括代码分析器，以警告你如果使用了任何不受支持的特性，但并非所有包都已被注释以与该特性良好协作。

最常用的注释来指示类型或成员不支持 AOT 是 `[RequiresDynamicCode]` 属性。

**更多信息**：您可以在以下链接中了解更多关于 AOT 警告的信息：[https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/fixing-warnings](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/fixing-warnings).

## 反射和本地 AOT

反射经常用于运行时检查类型元数据、成员的动态调用以及代码生成。

本地 AOT 允许一些反射功能，但在本地 AOT 编译过程中进行的剪裁无法静态确定类型具有可能仅通过反射访问的成员。这些成员将被 AOT 移除，这会导致运行时异常。

**良好实践**：开发者必须使用 `[DynamicallyAccessedMembers]` 注解他们的类型，以指示仅通过反射动态访问的成员，因此应保留未剪裁。

## 本地 AOT for ASP.NET Core

.NET 7 仅支持 Windows 或 Linux 上的控制台应用程序和类库的本地 AOT。它不支持 macOS 或 ASP.NET Core。.NET 8 是第一个支持 macOS 和 ASP.NET Core 部分功能版本。

以下 ASP.NET Core 功能完全受支持：`gRPC`、`CORS`、`HealthChecks`、`HttpLogging`、`Localization`、`OutputCaching`、`RateLimiting`、`RequestDecompression`、`ResponseCaching`、`ResponseCompression`、`Rewrite`、`StaticFiles` 和 `WebSockets`。

以下 ASP.NET Core 功能部分受支持：最小 API。

以下 ASP.NET Core 功能目前不受支持：MVC、Blazor Server、SignalR、身份验证（除 JWT 外）、会话和 SPA。

正如您之前所看到的，您通过将 HTTP 请求映射到 lambda 表达式来实现 ASP.NET Core 最小 API 网络服务，例如以下代码所示：

[PRE66]

在运行时，ASP.NET Core 使用 `RequestDelegateFactory` （**RDF**）类将您的 `MapX` 调用转换为 `RequestDelegate` 实例。但这是动态代码，因此与本地 AOT 不兼容。

在 ASP.NET Core 8 中，当启用本地 AOT 时，运行时使用 RDF 被一个名为 **请求委托生成器** （**RDG**）的源生成器所取代，该生成器执行类似的工作，但发生在编译时。这确保生成的代码可以被本地 AOT 发布过程静态分析。

**更多信息**：您可以在以下链接中学习如何创建自己的源生成器：[https://github.com/markjprice/apps-services-net8/blob/main/docs/ch01-dynamic-code.md#creating-source-generators](https://github.com/markjprice/apps-services-net8/blob/main/docs/ch01-dynamic-code.md#creating-source-generators).

## 本地 AOT 的要求

对于不同的操作系统，还有额外的要求：

+   在 Windows 上，您必须安装包含所有默认组件的 Visual Studio 2022 **桌面开发与 C++** 工作负载。

+   在Linux上，你必须安装.NET运行时所依赖的库的编译器工具链和开发包。例如，对于Ubuntu 18.04或更高版本：`sudo apt-get install clang zlib1g-dev`。

**警告！**跨平台原生AOT发布不受支持。这意味着你必须在你将部署的操作系统上运行发布。例如，你不能在Linux上发布原生AOT项目，然后将其在Windows上运行。

## 为项目启用原生AOT

要在项目中启用原生AOT发布，请将`<PublishAot>`元素添加到项目文件中，如下所示，高亮显示的标记：

[PRE67]

## 启用与原生AOT的JSON序列化

使用原生AOT进行JSON序列化需要使用`System.Text.Json`源生成器。所有作为参数或返回值传递的模型类型都必须在`JsonSerializerContext`中注册，如下所示：

[PRE68]

您必须将自定义JSON序列化器上下文添加到服务依赖项中，如下所示：

[PRE69]

## 构建原生AOT项目

现在让我们看看使用新项目模板的一个实际例子：

1.  在名为`Chapter08`的解决方案中，添加一个与原生AOT兼容的Web服务项目，如下所示：

    +   项目模板：**ASP.NET Core Web API (native AOT)** / `webapiaot`

        这是.NET 8引入的新项目模板。它与**Web API** / `webapi`项目模板不同。由于原生AOT支持目前仅限于最小API，因此它没有使用控制器选项。它也没有HTTPS选项，因为在云原生部署中HTTPS通常由反向代理处理。在JetBrains Rider中，选择**ASP.NET Core Web应用程序**，然后选择**类型**为**Web API (native AOT)**。

    +   解决方案文件和文件夹：`Chapter08`

    +   项目文件和文件夹：`Northwind.MinimalAot.Service`

    +   **启用Docker**：已清除。

    +   **不要使用顶层语句**：已清除。

    +   在`Properties`文件夹中，在`launchSettings.json`中，注意只配置了`http`；删除`launchUrl`并修改端口号为`5083`，如下所示，高亮显示的配置：

        [PRE70]

1.  在项目文件中，将不变全球化设置为`false`，将警告视为错误，注意原生AOT发布已启用，并为ADO.NET的SQL Server添加一个包引用，如下所示，高亮显示的标记：

    [PRE71]

1.  添加一个名为`Product.cs`的新类文件，并修改其内容以定义一个类，该类仅代表从`Products`表中的每一行中我们想要的三列，如下所示：

    [PRE72]

1.  添加一个名为`NorthwindJsonSerializerContext.cs`的新类文件。

1.  在`NorthwindJsonSerializerContext.cs`中，定义一个类，该类允许将`Product`和`Product`对象列表序列化为JSON，如下所示：

    [PRE73]

1.  删除`Northwind.MinimalAot.Service.http`文件。

1.  添加一个名为`WebApplication.Extensions.cs`的新类文件。

1.  在`WebApplication.Extensions.cs`中，为`WebApplication`类定义一个扩展方法，将一些HTTP `GET`请求映射为返回纯文本响应，以及使用ADO.NET从Northwind数据库获取所有产品或最低价格产品的列表，如下所示代码：

    [PRE74]

    使用原生AOT时，我们无法使用EF Core，因此我们正在使用较低级别的ADO.NET SqlClient API。无论如何，这更快、更高效。在未来，也许在.NET 9或.NET 10中，我们将能够使用我们的EF Core模型来处理Northwind。

1.  在`Program.cs`中，注意对`CreateSlimBuilder`方法的调用，这确保默认情况下只启用ASP.NET Core的基本功能，因此它最小化了部署的web服务大小，如下所示代码：

    [PRE75]

    `CreateSlimBuilder`方法不包括对HTTPS或HTTP/3的支持，尽管如果你需要，你可以自己添加这些。它支持`appsettings.json`的JSON文件配置和日志记录。

1.  在`Program.cs`中，在调用`builder.Build()`之后，删除生成一些示例待办事项和映射一些端点的语句，如下所示代码：

    [PRE76]

1.  在`Program.cs`文件底部，删除定义`Todo`记录和`AppJsonSerializerContext`类的语句，如下所示代码：

    [PRE77]

1.  在`Program.cs`中，删除命名空间导入，导入我们的JSON序列化上下文和扩展方法的命名空间，修改插入JSON序列化上下文的语句以使用Northwind的，然后在运行web `app`之前调用`MapGets`方法，如下所示高亮显示的代码：

    [PRE78]

1.  如果你的数据库服务器没有运行（例如，因为你正在Docker、虚拟机或云中托管它），那么请确保启动它。

1.  使用`http`配置文件启动web服务项目：

    +   如果你正在使用Visual Studio 2022，则在下拉列表中选择**http**配置文件，然后导航到**调试** | **不调试启动**或按*Ctrl* + *F5*。

    +   如果你正在使用Visual Studio Code，则输入命令`dotnet run --launch-profile http`，手动启动一个web浏览器，并导航到web服务：`https://localhost:5083/`。

1.  在web浏览器中，注意纯文本响应：`Hello from a native AOT minimal API web service.`。

1.  在地址栏中添加 `/products`，并注意响应中显示的产品数组，如下所示的部分输出：

    [PRE79]

1.  在地址栏中添加 `/products/100`，并注意响应中的两个产品数组，如下所示的部分输出：

    [PRE80]

1.  关闭浏览器并关闭web服务器。

## 发布原生AOT项目

在开发期间，当服务未经修剪且经过即时编译（JIT）时，功能正常的服务在发布时使用原生AOT可能仍然会失败。因此，在假设项目可以工作之前，你应该先进行发布。

如果您的项目在发布时没有产生任何AOT警告，那么您可以有信心，您的服务在发布后将会正常工作。

让我们回顾一下源生成的代码并发布我们的网络服务：

1.  在`Northwind.MinimalAot.Service`项目文件中，添加一个元素以输出编译器生成的文件，如下所示突出显示的标记：

    [PRE81]

1.  构建项目`Northwind.MinimalAot.Service`。

1.  如果您正在使用Visual Studio 2022，请在**解决方案资源管理器**中切换**显示所有文件**。

1.  展开文件夹`obj\Debug\net8.0\generated`，并注意源生成器为AOT和JSON序列化创建的文件夹和文件，并注意您将在接下来的几个步骤中打开其中一些文件，如图8.10所示：

![](img/B19587_08_10.png)

图8.10：AOT网络服务项目中源生成器创建的文件夹和文件

1.  打开`GeneratedRouteBuilderExtensions.g.cs`文件，并注意它包含用于定义最小API网络服务的映射路由的代码。

1.  打开`NorthwindJsonSerializerContext.Decimal.g.cs`文件，并注意它包含用于将`decimal`值序列化为作为最小单位价格参数传递给路由之一的代码。

1.  打开`NorthwindJsonSerializerContext.ListProduct.g.cs`文件，并注意它包含用于将`Product`对象列表序列化为响应的代码，这些对象是两条路由之一返回的。

1.  在`Northwind.MinimalAot.Service`文件夹中，在命令提示符或终端中，使用本地AOT发布网络服务，如下所示命令：

    [PRE82]

1.  注意有关生成本地代码以及针对`Microsoft.Data.SqlClient`等包的裁剪警告的消息，如下部分输出所示：

    [PRE83]

    我们没有使用`DataTable.ReadXml`方法，该方法调用一个可能被裁剪的成员，因此我们可以忽略前面的警告。

1.  启动**文件资源管理器**并打开`bin\Release\net8.0\win-x64\publish`文件夹，并注意EXE文件大约有30MB。这些以及`Microsoft.Data.SqlClient.SNI.dll`文件是需要在另一台Windows计算机上部署以使网络服务正常工作的唯一文件。`appsettings.json`文件仅在需要覆盖配置时才需要。PDB文件仅在调试时需要。

1.  运行`Northwind.MinimalAot.Service.exe`，并注意网络服务启动非常快，它默认将使用端口`5000`，如图8.11所示：

![](img/B19587_08_11.png)图8.11：文件资源管理器显示已发布的可执行文件和正在运行AOT网络服务

1.  启动一个网页浏览器，导航到`http://localhost:5000/`，并注意网络服务通过返回纯文本响应来正确工作。

1.  导航到`http://localhost:5000/products/100`，并注意网络服务响应包含最小单位价格为100的两个产品。

1.  关闭网页浏览器并关闭网络服务。

1.  在`Northwind.WebApi.Service`项目文件中，在命令提示符或终端中发布网络服务，如下所示命令：

    [PRE84]

1.  启动 **文件资源管理器**，打开 `bin\Release\net8.0\win-x64\publish` 文件夹，并注意 `Northwind.WebApi.Service.exe` 文件小于 154 KB。这是因为它是框架依赖的，意味着它需要在计算机上安装 .NET 才能工作。此外，还有许多必须与 EXE 文件一起部署的文件，总大小约为 14 MB。

1.  运行 `Northwind.MinimalAot.Service.exe` 并注意网络服务启动速度比 AOT 版本慢，并且它将默认使用端口 `5000`。

1.  启动网页浏览器，导航到 `http://localhost:5000/`，并注意通过返回纯文本响应，该网络服务运行正常。

1.  导航到 `http://localhost:5000/products/100`，并注意网络服务响应了两个最小单位价格为 100 的产品，但响应速度比 AOT 版本慢。

1.  关闭网页浏览器并关闭网络服务。

许多 .NET 开发者已经期待 AOT 编译很长时间了。微软终于兑现了承诺，并且它将在接下来的几个主要版本中扩展以覆盖更多项目类型，因此这是一个需要关注的科技。

# 理解身份服务

身份服务用于验证和授权用户。对于这些服务实现开放标准非常重要，这样您就可以集成不同的系统。常见的标准包括 **OpenID Connect** 和 **OAuth 2.0**。

微软没有计划正式支持像 **IdentityServer4** 这样的第三方身份验证服务器，因为“创建和维护一个身份验证服务器是一项全职工作，微软已经在该领域有一个团队和一个产品，即 Azure Active Directory，它允许免费使用 500,000 个对象。”

## JWT 承载授权

**JSON Web Token** (**JWT**) 是一个标准，它定义了一种紧凑且安全的方法来以 JSON 对象的形式传输信息。该 JSON 对象经过数字签名，因此可以信任。使用 JWT 最常见的场景是授权。

用户使用用户名和密码或生物识别扫描或双因素认证等凭证登录到受信任方，受信任方颁发 JWT。然后，它将与每个请求一起发送到安全的网络服务。

在其紧凑形式中，JWT 由三个部分组成，由点分隔。这些部分是 *头部*、*负载* 和 *签名*，如下所示：`aaa.bbb.ccc`。头部和负载是 Base64 编码的。

## 使用 JWT 承载身份验证验证服务客户端

在本地开发过程中，使用 `dotnet user-jwts` 命令行工具来创建和管理本地 JWT。这些值存储在本地机器用户配置文件文件夹中的 JSON 文件中。

让我们使用 JWT 承载身份验证来保护网络服务，并使用本地令牌进行测试：

1.  在 `Northwind.WebApi.Service` 项目中，添加对 JWT 承载身份验证包的引用，如下所示：

    [PRE85]

1.  构建 `Northwind.WebApi.Service` 项目以恢复包。

1.  在 `Program.cs` 中，在创建 `builder` 之后，添加语句以使用 JWT 添加授权和身份验证，如下所示（高亮显示）：

    [PRE86]

1.  在 `Program.cs` 中，在构建应用程序之后，添加一个语句来使用授权，如下所示（高亮显示）：

    [PRE87]

1.  在 `WebApplication.Extensions.cs` 中，导入安全声明的命名空间，如下所示：

    [PRE88]

1.  在 `WebApplication.Extensions.cs` 中，在将根路径的 HTTP `GET` 请求映射到返回纯文本 `Hello World` 响应之后，添加一个语句将秘密路径的 HTTP `GET` 请求映射到如果授权则返回认证用户的名称，如下所示：

    [PRE89]

1.  在 `Northwind.WebApi.Service` 项目文件夹中，在命令提示符或终端中，创建一个本地 JWT，如下所示：

    [PRE90]

1.  注意自动分配的 `ID`、`Name` 和 `Token`，如下所示的部分输出：

    [PRE91]

1.  在命令提示符或终端中，打印分配的 ID 的所有信息，如下所示：

    [PRE92]

1.  注意方案是 `Bearer`，因此必须在每次请求中发送令牌，受众列表列出了授权客户端域名和端口号，令牌在三个月后过期，JSON 对象表示头和有效负载，最后是紧凑型令牌，其 Base64 编码的三部分由点分隔，如下所示的部分输出：

    [PRE93]

1.  在 `Northwind.WebApi.Service` 项目中，在 `appsettings.Development.json` 中，注意名为 `Authentication` 的新部分，如下所示（高亮显示）：

    [PRE94]

1.  使用无调试的 `https` 配置启动 `Northwind.WebApi.Service` 项目。

1.  在浏览器中，将相对路径更改为 `/secret` 并注意响应被拒绝，状态码为 401。

1.  启动 Visual Studio Code 并打开 `HttpRequests` 文件夹。

1.  在 `HttpRequests` 文件夹中，创建一个名为 `webapi-secure-request.http` 的文件，并修改其内容以包含获取秘密成分的请求，如下所示（但当然使用您的 `Bearer` 令牌）：

    [PRE95]

1.  点击 **发送请求**，并注意响应，如下所示：

    [PRE96]

1.  关闭浏览器并关闭网络服务。

# 练习和探索

通过回答一些问题、进行一些实际操作练习，以及更深入地研究本章主题来测试您的知识和理解。

## 练习 8.1 – 测试您的知识

回答以下问题：

1.  列出六个可以在 HTTP 请求中指定的方法名。

1.  列出六个可以返回的 HTTP 响应状态码及其描述。

1.  ASP.NET Core Minimal APIs 服务技术与 ASP.NET Core Web APIs 服务技术有何不同？

1.  使用 ASP.NET Core Minimal APIs 服务技术，您如何将 HTTP `PUT` 请求映射到 `api/customers` 的 lambda 语句块？

1.  使用 ASP.NET Core 最小 API 服务技术，您如何将方法或 lambda 参数映射到路由、查询字符串或请求体中的值？

1.  启用 CORS 是否会增加 Web 服务的安全性？

1.  您已在 `Program.cs` 中添加了语句以启用 HTTP 日志记录，但 HTTP 请求和响应并未被记录。最可能的原因是什么，以及如何修复它？

1.  您如何使用 `AspNetCoreRateLimit` 包限制特定客户端的请求数量？

1.  您如何使用 `Microsoft.AspNetCore.RateLimiting` 包限制特定端点的请求数量？

1.  JWT 代表什么？

## 练习 8.2 – 审查 Microsoft HTTP API 设计策略

微软有内部 HTTP/REST API 设计指南。微软团队在设计他们的 HTTP API 时会参考此文档。它们是您自己 HTTP API 标准的绝佳起点。您可以在以下链接中查看它们：

[https://github.com/microsoft/api-guidelines](https://github.com/microsoft/api-guidelines)

指南中有一个专门针对 CORS 的部分，您可以通过以下链接查看它们：

[https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#8-cors](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#8-cors)

## 练习 8.3 – 探索主题

使用以下页面上的链接了解本章涵盖主题的更多详细信息：

[https://github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-8---building-and-securing-web-services-using-minimal-apis](https://github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-8---building-and-securing-web-services-using-minimal-apis)

## 练习 8.4 – 通过 OData 服务公开数据

在本在线章节中学习如何快速实现一个可以使用 OData 包装 EF Core 实体模型的 Web API 服务：

[https://github.com/markjprice/apps-services-net8/blob/main/docs/ch08-odata.md](https://github.com/markjprice/apps-services-net8/blob/main/docs/ch08-odata.md)

## 练习 8.5 – Auth0 项目模板

如果您需要实现 Auth0 进行身份验证和授权，则可以使用项目模板来生成代码。描述这些项目模板及其使用方法的文章可在以下链接中找到：

[https://auth0.com/blog/introducing-auth0-templates-for-dotnet/](https://auth0.com/blog/introducing-auth0-templates-for-dotnet/)

# 摘要

在本章中，你学习了如何：

+   构建一个实现 REST 架构风格的 Web 服务，使用最小 API。

+   使用 CORS 释放特定域名和端口的同源安全策略。

+   实现两个不同的速率限制包以防止拒绝服务攻击。

+   使用 JWT 持有者授权来保护服务。

在下一章中，您将学习如何通过添加缓存、队列和自动处理短暂故障等特性，使用 Polly 等库构建可靠和可扩展的服务。
