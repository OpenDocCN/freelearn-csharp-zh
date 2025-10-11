# 使用 Swagger 记录您的 API

在本章中，我们将学习如何使用 OpenAPI 规范记录我们的 API，以及如何使用 Swagger 工具解析和生成文档。当我们的 Web 服务被外部公司或外国组织团队消费时，记录 API 特别重要。此外，一些服务可能相当复杂，并公开大量端点。因此，一些与 .NET 生态系统相关的工具保证了 API 文档的更新。在此过程中可以使用两个主要工具链：NSwag 和 Swashbuckle。在本书中，我们将介绍并使用 NSwag 来记录我们的 API。

在本章中，我们将涵盖以下主题：

+   理解 OpenAPI

+   在 ASP.NET Core 服务中实现 OpenAPI

到本章结束时，您将能够使用 Swagger 和 NSwag 包自动生成符合 OpenAPI 规范的最新文档。

# 理解 OpenAPI

OpenAPI 创新是 Linux 基金会的一部分，并定义了 **OpenAPI** **规范**（**OAS**）标准。OpenAPI 规范旨在为 REST API 提供一个语言无关的接口。这种方法保证了人类和客户端应用程序可以通过参考一个唯一的入口点来理解和发现 Web 服务的功能。此外，它还提供了一个高级抽象，也可以用于商业或设计目的。

此外，其查询服务的标准方式简化了各种自动化——从客户端的自动生成到文档的自动生成。

# 实现 Swagger 项目

就像 OpenAPI 规范一样，Swagger 作为一个描述 REST API 的语言无关规范而诞生。它最近已被 OpenAPI 项目采用，这意味着这两个项目之间没有差异。

Swagger 的主要目标是自动生成并公开一个名为 `swagger.json` 的文档，也称为 **Swagger 规范**。Swagger 规范是 API 的自动生成文档，提供了关于 Web 服务公开的每个单独路由的信息。以下代码展示了示例 `swagger.json` 文件的结构：

```cs
{
  "x-generator": "NSwag v12.0.12.0 (NJsonSchema v9.13.15.0 (Newtonsoft.Json v12.0.0.0))",
  "swagger": "2.0",
  "host": "localhost:5000",
  "schemes": [
    "http"
  ],
  "consumes": [
    "application/json"
  ],
  "paths": {
    "/api/artist": {
      "get": {
        "tags": [
          "Artist"
        ],
        "operationId": "Artist_Get",
        "parameters": [
          {
            "type": "string",
            "name": "artistId",
            "in": "query"
          }
        ],
        "responses": {
          "200": {
            "schema": {
              "type": "file"
            }
          }
        }
...
```

上述代码片段描述了在**目录服务**API 中定义的一些路由。正如你所见，在 JSON 的第一级中，有一些关于服务的一般信息，例如服务的`apiVersion`、`title`和`basePath`。此外，我们还可以看到一个名为`paths`的节点，它包含我们服务的所有路径。对于每个路由，它描述了不同的响应类型、不同的 HTTP 动词以及服务接受的全部有效负载信息。由于我们有一个独特的标准来描述我们的 API，因此也可以定义一个独特的用户界面，以便我们可以查询并向服务发送信息；这就是**Swagger UI**的作用。Swagger UI 是一个使用`swagger.json`文件提供用户友好 UI 的工具：

![图片](img/68000202-f3d8-455b-928f-d5a3f0a2952f.png)

上一张截图显示了我们可以用来浏览 API 公开的不同路由的有用 UI 示例。此外，它还允许消费者立即全面了解 API 提供的数据。现在，让我们学习如何在 ASP.NET Core 中实现 OpenAPI。

# 在 ASP.NET Core 服务中实现 OpenAPI

我们可以使用两个不同的包在 ASP.NET Core 中实现 OpenAPI：

+   **Swashbuckle**: [`docs.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle?view=aspnetcore-2.2`](https://docs.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle?view=aspnetcore-2.2)

+   **NSwag**: [`docs.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-nswag?view=aspnetcore-2.2`](https://docs.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-nswag?view=aspnetcore-2.2)

这两个都使用中间件来生成和提供`swagger.json`文件，并允许用户界面浏览服务定义。在本节中，我们将讨论如何将 NSwag 集成到我们的 vinyl 目录服务中。以下架构显示了 NSwag 是如何集成到我们的 ASP.NET Core 服务中的：

![图片](img/9316aeb8-6ddc-4e23-b63f-140dc116b563.png)

让我们从通过以下命令将`NSwag.AspNetCore`添加到`Catalog*.*API`项目开始：

```cs
dotnet add package NSwag.AspNetCore
```

之后，我们可以通过结合生成和提供 OpenAPI 规范的中介件以及初始化 UI 的中介件来继续操作。正如我们在第三章“与中介件管道一起工作”中看到的，我们需要使用在`Startup`类中实现的`Configure`和`ConfigureServices`方法：

```cs
...
    public class Startup
    {

        public void ConfigureServices(IServiceCollection services)
        {
            services
                .AddCatalogContext(Configuration.
                GetSection("DataSource:ConnectionString").Value);

            services
               ..
                .AddOpenApiDocument(settings =>{
                                       settings.Title = "Catalog API";
                                       settings.DocumentName = "v3";
                                       settings.Version = "v3";                                   
                                    });
        }

        public void Configure(IApplicationBuilder app, 
        IHostingEnvironment env)
        {
            ...
             app
                .UseOpenApi()
 .UseSwaggerUi3();
        }
    }
}
```

`AddOpenApiDocument` 添加了生成 OpenAPI 3.0 所需的服务。`UseOpenApi` 添加了 OpenAPI/Swagger 生成器，它使用 API 描述来执行 Swagger 生成，而 `UseSwaggerUi3` 创建并实例化提供 Swagger UI 的中间件。由于我们已经将 OpenAPI 中间件集成到我们的服务中，我们可以通过运行我们的服务并使用我们首选的浏览器浏览 `https://localhost/swagger` URL 来继续操作。

NSwag 和 Swashbuckle 使用反射来浏览我们控制器内的操作方法。幸运的是，这个过程只在服务第一次运行时执行。有时，复杂的响应类型可能会阻止生成 `swagger.json` 文件。因此，强烈建议您检查我们控制器操作方法提供的所有响应类型。

NSwag 还提供了一些有用的工具，以便我们可以在我们的 Web 服务上执行代码生成，例如以下这些：

+   `NSwag.CodeGeneration.CSharp` ([`www.nuget.org/packages/NSwag.CodeGeneration.CSharp/`](https://www.nuget.org/packages/NSwag.CodeGeneration.CSharp/))

+   `NSwag.CodeGeneration.TypeScript` ([`www.nuget.org/packages/NSwag.CodeGeneration.TypeScript/`](https://www.nuget.org/packages/NSwag.CodeGeneration.TypeScript/))

这些允许我们分别为 C# 和 Typescript 自动生成客户端类。

在本节中，我们学习了如何安装和配置 NSwag，以便我们可以公开与 OpenAPI 规范兼容的 Swagger 文档。在下一节中，我们将学习如何显式定义我们 API 的约定，以及如何在 `swagger.json` 合同中包含额外的信息。

# 理解 ASP.NET Core 的约定

Swagger UI 的默认响应类型会产生一些错误信息。如果我们查看响应部分，我们会看到响应代码是不正确的，并且它与由 Web 服务返回的实际 HTTP 代码不对应。当使用 ASP.NET Core 2.2 或更高版本时，可以使用约定来指定响应类型：

```cs
..
    [ApiController]
    public class ItemController : ControllerBase
    {
        [HttpGet]
        [ApiConventionMethod(typeof(DefaultApiConventions), 
        nameof(DefaultApiConventions.Get))]
        public async Task<IActionResult> Get([FromQuery] int pageSize = 
        10, [FromQuery] int pageIndex = 0)

        [HttpGet("{id:guid}")]
        [ApiConventionMethod(typeof(DefaultApiConventions), 
        nameof(DefaultApiConventions.Get))]
        public async Task<IActionResult> GetById(Guid id)
      ...
```

例如，前面的代码使用 `ApiConventionMethod` 属性传递一个自定义类型和方法名称。`ApiConventionMethod` 属性是 `Microsoft.AspNetCore.Mvc` 命名空间的一部分，并使用 `DefaultApiConventions` 静态类，它为通用 API 中的每个操作提供一组默认约定。同样，我们还可以将此属性添加到 `ItemController` 的写入方法中，例如 `Create`、`Update` 和 `Delete` 方法：

```cs
        ...

        [HttpPost]
        [ApiConventionMethod(typeof(DefaultApiConventions), 
        nameof(DefaultApiConventions.Create))]
        public async Task<IActionResult> Create(AddItemRequest request)

        [HttpPut("{id:guid}")]
        [ApiConventionMethod(typeof(DefaultApiConventions), 
        nameof(DefaultApiConventions.Update))]
        public async Task<IActionResult> Update(Guid id, 
        EditItemRequest request)

        [HttpDelete("{id:guid}")]
        [ApiConventionMethod(typeof(DefaultApiConventions), 
        nameof(DefaultApiConventions.Delete))]
        public async Task<IActionResult> Delete(Guid id)
    }
}
```

这种方法是一种快捷方式，我们可以用它来声明操作方法响应，而无需显式使用 `ProducesResponseType` 属性。让我们看看 `DefaultApiConventions` 静态类，如果我们声明一些静态 void 方法，它将提供一组默认响应类型：

```cs
using Microsoft.AspNetCore.Mvc.ApiExplorer;

namespace Microsoft.AspNetCore.Mvc
{
  public static class DefaultApiConventions
  {
    [ProducesResponseType(200)]
 [ProducesResponseType(404)]
 [ProducesDefaultResponseType]
 [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Prefix)]
 public static void Get([ApiConventionNameMatch
    (ApiConventionNameMatchBehavior.Suffix), ApiConventionTypeMatch(
    ApiConventionTypeMatchBehavior.Any)] object id)
 {
 }

    ...
  }
}
```

例如，对于 `Get` 方法，它声明了 `HTTP 200 OK` 响应和 `HTTP 404 Not found`。通过这样做，我们可以轻松地为每个操作声明适当的响应类型。`DefaultApiConventions` 类是 `Microsoft.AspNetCore.Mvc` 命名空间的一部分。

# 自定义约定

`DefaultApiConvention` 类并不总是适合我们的控制器。此外，它过于通用，操作方法通常过于具体，不适合 `DefaultApiConvention` 类。因此，ASP.NET Core 允许我们根据我们的需求创建自定义的 API 约定。要声明一个新的约定，我们需要创建一个新的静态类，其中包含相应的静态方法，如下所示：

```cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.ApiExplorer;

namespace Catalog.API.Conventions
{
    public static class ItemApiConvention
    {

        [ProducesResponseType(200)]
        [ProducesResponseType(404)]
        [ProducesResponseType(400)]
        [ProducesDefaultResponseType]
        [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Prefix)]
        public static void Get([ApiConventionNameMatch
        (ApiConventionNameMatchBehavior.Suffix),
                                ApiConventionTypeMatch
                                (ApiConventionTypeMatchBehavior.Any)]
                                 object id)
        {
        }

        ...
    }
}
```

我们在这里实现的约定描述了 `ItemController` 的 `Get` 操作方法。如您所见，此方法产生以下 HTTP 响应：`200`、`404` 和 `400`。这种方法还允许我们生成和扩展由路由返回的响应类型。此外，我们可以通过以下方式应用属性来分配和使用这些约定：

```cs
[HttpGet]
[ApiConventionMethod(typeof(ItemApiConvention), nameof(ItemApiConvention.Get))]
public async Task<IActionResult> Get([FromQuery] int pageSize = 10, [FromQuery] int pageIndex = 0)
{

    ...
}
```

这种方法使我们能够将 API 约定自定义并分组到一个独特的类中，并完全自定义 API 的合同。同样的方法也可以用于您服务中控制器类中存在的其他操作方法。

# 摘要

在本章中，我们学习了如何通过使用 OpenAPI 规范来记录 Web 服务，从而提高其可发现性。OpenAPI 技术还为我们提供了一种标准方式，以生成每种语言的客户端并生成自动维护的文档。当服务被第三方团队和消费者使用时，记录 API 是有用的，并且它还为我们提供了服务公开的信息和动作的高级概述。

在下一章中，我们将学习关于 Postman 以及如何使用它来查询、测试和检查 Web 服务的响应。
