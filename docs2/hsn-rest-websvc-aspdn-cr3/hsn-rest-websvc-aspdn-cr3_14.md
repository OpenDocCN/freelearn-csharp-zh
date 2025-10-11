# 第十一章：构建 API 的高级概念

上一章介绍了 Web 服务 HTTP 层的实现。尽管服务的核心功能已经到位，但仍有一些细节需要完善。本章将介绍一些额外的实现，这些实现将成为目录 Web 服务的一部分，例如软删除资源、HATEOAS 方法、添加响应时间中间件以及 ASP.NET Core 中异步代码的一些最佳实践。更具体地说，它将涵盖以下主题：

+   实现软删除技术

+   实现 HATEOAS

+   ASP.NET Core 中异步代码概述

+   使用中间件测量 API 的响应时间

本章中提供的代码可在以下 GitHub 仓库中找到：[`github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3`](https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3)。

# 实现软删除过程

如同在第五章中提到的*Web 服务堆栈在 ASP.NET Core 中*的*删除资源*部分，软删除技术在现实世界的应用中是一种常见的删除实践。此外，从我们的数据源物理删除实体相当不常见。

被删除的数据可能对历史目的以及执行分析和报告相关。软删除实现涉及我们在前三个章节中看到的所有项目。为了继续，让我们在`Catalog.Domain`项目中的`Item`实体中添加一个新的`IsInactive`字段：

```cs
namespace Catalog.Domain.Entities
{
    public class Item
    {
        ...
        public bool IsInactive { get; set; }
    }
}
```

由于我们更改了`Item`实体的模式，我们需要在`Catalog.API`项目中执行以下命令以执行另一个 EF Core 迁移：

```cs
dotnet ef migrations add Added_IsInactive_field
dotnet ef database update
```

执行上述命令的结果将在`Migrations`文件夹中生成一个新的迁移文件，并将新创建的迁移应用于`Startup`类中指定的数据库。

此外，正如我们在第八章中看到的，*构建数据访问层*，如果我们想连接到本地的 Docker SQL Server 实例，我们可以在`appsettings.json`文件中指定以下连接字符串：

```cs
{
    "Logging": {
        "LogLevel": {
            "Default": "Warning"
        }
    },
 "DataSource": {
 "ConnectionString": "Server=localhost,1433;Initial Catalog=Store;User Id=catalog_srv;Password=P@ssw0rd"
    }
}
```

之前提供的连接字符串提供了连接到本地 SQL Server 实例的连接。为了运行实例，必须遵循我们在第八章中看到的命令。以下使用名称 sql1 运行 docker 实例：

```cs
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=<YOUR_SA_PASSWORD>" -p 1433:1433 --name sql1 -d mcr.microsoft.com/mssql/server:2017-latest
```

此外，我们可以使用以下命令登录到容器：

```cs
docker exec -it sql1 "bash"
```

最后，我们可以执行`sqlcmd`以登录到 SQL 服务器实例：

```cs
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P '<YOUR_SA_PASSWORD>'
1> CREATE LOGIN catalog_srv WITH PASSWORD = 'P@ssw0rd';
2> CREATE DATABASE Store;
3> GO
1> USE Store;
2> CREATE USER catalog_srv;
3> GO
1> EXEC sp_addrolemember N'db_owner', N'catalog_srv';
2> GO
```

上面的代码创建了一个名为 `catalog_srv` 的登录用户的 `Store` 数据库。在本书的后面部分，我们将看到如何使用 Docker 提供的工具来自动化此过程。现在，数据库模式已更新，我们能够通过在应用程序的存储库和服务层中包含 `IsInactive` 字段来继续进行。

# 更新 IItemRepository 实现

自从我们在数据库中添加了新的 `IsInactive` 字段后，我们可以通过调整 `IItemRepository` 接口来根据 `IsInactive` 字段过滤我们的数据来继续进行。因此，我们将通过实施以下更改来保持数据源的一致性：

+   `IItemRepository.GetAsync()` 方法将通过 `IsInactive = false` 过滤所有字段。因此，生成的响应将仅包含活动实体。这种方法的优点是，当我们尝试从数据库中获取多个实体时，我们可以得到一个轻量级的响应。

+   同样，`GetItemByArtistIdAsync` 和 `GetItemByGenreIdAsync` 将使用 `IsInactive = false` 标志来过滤结果。此外，在这种情况下，我们希望尽可能保持响应尽可能轻量。

+   `IItemRepository.GetAsync(Guid id)` 方法将根据 `IsInactive` 标志检索所需实体的详细信息。此外，此方法被应用程序的验证检查所使用，因此，当我们插入新对象时，无论它们是活动状态还是非活动状态，我们都需要避免重复的 ID。

让我们通过实现 `IItemRepository` 接口中提到的这些规范来继续：

```cs
namespace Catalog.Infrastructure.Repositories
{
    public class ItemRepository
        : IItemRepository
    {
        ...

        public async Task<IEnumerable<Item>> GetAsync()
        {
            return await _context.Items
                .Where(x => !x.IsInactive)
                .AsNoTracking()
                .ToListAsync();
        }

        public async Task<IEnumerable<Item>> GetItemByArtistIdAsync(Guid id)
        {
            var items = await _context
                .Items
                .Where(x => !x.IsInactive)
                .Where(item => item.ArtistId == id)
                .Include(x => x.Genre)
                .Include(x => x.Artist)
                .ToListAsync();

            return items;
        }

        public async Task<IEnumerable<Item>> GetItemByGenreIdAsync(Guid id)
        {
            var items = await _context.Items
                .Where(x => !x.IsInactive)
                .Where(item => item.GenreId == id)
                .Include(x => x.Genre)
                .Include(x => x.Artist)
                .ToListAsync();

            return items;
        }
...
```

此实现通过使用 `Where` LINQ 子句过滤 `IsInactive == false` 来更改 `GetAsync`、`GetItemByArtistIdAsync` 和 `GetItemByGenreIdAsync` 方法的行为。另一方面，`GetAsync(Guid id)` 保持不变，因为，如规范中所述，我们希望获取详细信息操作始终检索信息，包括记录非活动的情况。因此，我们可以通过在项目根目录中执行以下命令来测试生成的实现：

```cs
dotnet test
```

所有测试都应该通过，因为默认情况下，`IsInactive` 布尔字段始终为 `false`。为了测试 `Where(x=>!x.IsInactive)` 的更改，我们可以在 `Catalog.API/tests/Catalog.Fixtures/Data/item.json` 文件中添加一条新记录，添加一个非活动项：

```cs
{
    "Id": "f5da5ce4-091e-492e-a70a-22b073d75a52",
    "Name": "Untitled",
    "Description": "Untitled by Kendrick Lamar",
    "PictureUri": "https://mycdn.com/pictures/32423423",
    "ReleaseDate": "2016-01-01T00:00:00+00:00",
    "Price": {
      "Amount": 23.5,
      "Currency": "EUR"
    },
    "Format": "Vinyl 33g",
    "AvailableStock": 6,
    "GenreId": "c04f05c0-f6ad-44d1-a400-3375bfb5dfd6",
    "Genre": null,
    "ArtistId": "3eb00b42-a9f0-4012-841d-70ebf3ab7474",
    "Artist": null,
    "IsInactive": true
  },
```

因此，我们可以通过创建一个新的测试并计算结果来验证通过获取操作检索的记录数是否发生变化。例如，如果 `item.json` 文件包含三个记录，其中一个是非活动的，则获取操作应检索三个记录。作为替代方案，我们可以通过验证结果中的 `Item.Id` 字段不包含非活动记录来双重检查：

```cs
namespace Catalog.Infrastructure.Tests
{
    ...
    public class ItemRepositoryTests : IClassFixture<CatalogContextFactory>
    {

        [Theory]
        [InlineData("f5da5ce4-091e-492e-a70a-22b073d75a52")]
        public async Task getitems_should_not_return_inactive_records(string id)
        {
            var result =
                await _sut.GetAsync();

            result.Any(x => x.Id == new Guid(id)).ShouldBeFalse();
        }

      ...
   }
}
```

我选择通过在`ItemRepositoryTests`类中添加一个新的`getitems_should_not_return_inactive_records`方法来测试删除行为。该测试验证当我调用仓库的`GetAsync`方法时，结果排除了作为测试参数指定的`Id`的`Item`实体。对于`GetItemByArtistIdAsync`和`GetItemByGenreIdAsync`方法，也可以采取类似的方法。

# 实现删除操作

现在我们已经在仓库端实现了正确的过滤逻辑，接下来让我们在`Catalog.Domain`项目中通过在`IItemService`接口声明中添加一个新的`DeleteItemAsync`方法来实施删除操作：

```cs
namespace Catalog.Domain.Services
{
    public interface IItemService
    {
        ...
        Task<ItemResponse> DeleteItemAsync(DeleteItemRequest request);
    }
}
```

`DeleteItemAsync`方法引用一个`DeleteItemRequest`类型，它可以声明如下：

```cs
using System;

namespace Catalog.Domain.Requests.Item
{
    public class DeleteItemRequest
    {
        public Guid Id { get; set; }
    }
}
```

一旦我们更新了`IItemService`接口声明，我们就可以继续向`ItemService`具体类添加实现：

```cs
public async Task<ItemResponse> DeleteItemAsync(DeleteItemRequest request)
{
    if (request?.Id == null) throw new ArgumentNullException();

    var result = await _itemRepository.GetAsync(request.Id);
    result.IsInactive = true;

    _itemRepository.Update(result);
    await _itemRepository.UnitOfWork.SaveChangesAsync();

    return _itemMapper.Map(result);
}
```

`DeleteItemAsync`方法接收一个`Id`字段，它是`DeleteItemRequest`类的一部分。此外，它使用`GetAsync`方法获取实体的详细信息，然后通过传递带有`IsInactive`标志设置为`true`的实体来调用`Update`方法。下一步是实现`ItemController`中的一个新操作方法，该方法涵盖了以下 HTTP 调用：

```cs
DELETE /items/{id}
```

此外，我们可以继续按照以下方式更改`ItemController`：

```cs
using System;
using System.Threading.Tasks;
using Catalog.API.Filters;
using Catalog.Domain.Requests.Item;
using Microsoft.AspNetCore.Mvc;

namespace Catalog.API.Controllers
{
    [Route("api/items")]
    [ApiController]
    public class ItemController : ControllerBase
    {
        ...

 [HttpDelete("{id:guid}")]
 [ItemExists]
 public async Task<IActionResult> Delete(Guid id)
 {
 var request = new DeleteItemRequest { Id = id }; 
 await _itemService.DeleteItemAsync(request); 
 return NoContent();
 }
    }
}
```

`Delete`操作方法被`HttpDelete`属性动词装饰，这意味着它与 HTTP `Delete`请求映射。实现通过`DeleteItemAsync`方法设置`IsInactive`标志，并返回一个`NoContent`结果，这代表 HTTP `204 No Content`状态码。

我们将继续通过实现以下测试来测试对`ItemController`类所做的更改：

```cs
namespace Catalog.API.Tests.Controllers
{
    public class ItemsController : IClassFixture<InMemoryApplicationFactory<Startup>>
    {
        private readonly InMemoryApplicationFactory<Startup> _factory;

        public ItemsController(InMemoryApplicationFactory<Startup> factory)
        {
            _factory = factory;
        }

        ...

        [Theory]
        [LoadData("item")]
        public async Task delete_should_returns_no_content_when_called_with_right_id(DeleteItemRequest 
        request)
        {
            var client = _factory.CreateClient();

            var response = await client.DeleteAsync($"/api/items/{request.Id}");

            response.StatusCode.ShouldBe(HttpStatusCode.NoContent);
        }

        [Fact]
        public async Task delete_should_returns_not_found_when_called_with_not_existing_id()
        {
            var client = _factory.CreateClient();

            var response = await client.DeleteAsync($"/api/items/{Guid.NewGuid()}");

            response.StatusCode.ShouldBe(HttpStatusCode.NotFound);
        }
    }
}

```

此测试代码涵盖了以下两个行为：

+   `delete_should_returns_no_content_when_called_with_right_id`测试方法检查操作方法是否正确返回 HTTP `NoContent`响应。

+   `delete_should_returns_not_found_when_called_with_not_existing_id`方法检查当传递一个不存在的 GUID（`Guid.NewGuid`）时，操作方法是否返回`HttpStatusCode.NotFound`。

在这两个测试用例中，我们使用已经实现的测试基础设施来验证 Web 服务的新行为。在本节中，我们看到了如何扩展我们的 Web 服务以支持软删除操作。

下一个部分将专注于实现 HATEOAS 响应方法。

# 实现 HATEOAS

我们已经在第一章中讨论了 HATEOAS 原则的理论，*REST 101 和 ASP.NET Core 入门指南*。本节解释了如何为`Catalog.API`项目中现有的`ItemController`实现 HATEOAS 方法。以下代码片段展示了通用 HATEOAS 响应的示例：

```cs
{
    "_links": {
        "get": {
            "rel": "ItemsHateoas/Get",
            "href": "https://localhost:5001/api/hateoas/items",
            "method": "GET"
        },
        "get_by_id": {
            "rel": "ItemsHateoas/GetById",
            "href": "https://localhost:5001/api/hateoas/items/8ff0fe8f-9dbc-451f-7a57-08d652340f56",
            "method": "GET"
        },
        "create": {
            "rel": "ItemsHateoas/Post",
            "href": "https://localhost:5001/api/hateoas/items",
            "method": "POST"
        },
        "update": {
            "rel": "ItemsHateoas/Put",
            "href": "https://localhost:5001/api/hateoas/items/8ff0fe8f-9dbc-451f-7a57-08d652340f56",
            "method": "PUT"
        },
        "delete": {
            "rel": "ItemsHateoas/Delete",
            "href": "https://localhost:5001/api/hateoas/items/8ff0fe8f-9dbc-451f-7a57-08d652340f56",
            "method": "DELETE"
        }
    },
    "id": "8ff0fe8f-9dbc-451f-7a57-08d652340f56",
    "name": "Malibu",
    "description": "Malibu. by Anderson Paak",
    "labelName": "Steel Wool/OBE/Art Club",
    "price": {
        "amount": 23.5,
        "currency": "EUR"
    },
    "pictureUri": "https://mycdn.com/pictures/32423423",
    "releaseDate": "2016-01-01T00:00:00+00:00",
    "format": "Vinyl 43",
    "availableStock": 3,
    "genreId": "7fcdde39-342b-4f80-0db1-08d65233f5a6",
    "genre": null,
    "artistId": "ff1921a8-f49a-4db2-0c2e-08d65233875e",
    "artist": null
}
```

尽管这种响应给有效的 JSON 响应增加了大量负载，但它通过提供我们数据中每个操作和资源的 URL 来帮助客户端。HATEOAS 原则将与 `ItemController` 并行实施，并且它将向客户端提供所有必要的信息以与目录服务拥有的数据进行交互*.*

# 用 HATEOAS 数据丰富我们的模型

要使 HATEOAS 正常工作，我们将依赖一个名为 `RiskFirst.Hateoas` 的第三方 NuGet 包*.* 此包允许我们有效地在我们的服务中集成 HATEOAS 原则。首先，让我们通过在两个项目文件夹中执行以下命令将包添加到 `Catalog.API`：

```cs
 dotnet add package RiskFirst.Hateoas
```

接下来，创建一个新的实体，称为 `ItemHateoasResponse` 类，它表示 HATEOAS 响应。此类引用已实现的 `ItemResponse` 类，并实现了 `RiskFirst.Hateoas` 包公开的 `ILinkContainer` 接口：

```cs
using System.Collections.Generic;
using Newtonsoft.Json;
using RiskFirst.Hateoas.Models;
using Catalog.Domain.Responses.Item;

namespace Catalog.API.ResponseModels
{
        public class ItemHateoasResponse :  ILinkContainer
        {
            public ItemResponse Data;          
            private Dictionary<string, Link> _links;

            [JsonProperty(PropertyName = "_links")]
            public Dictionary<string, Link> Links
            {
                get => _links ?? (_links = new Dictionary<string, Link>());
                set => _links = value;
            }

            public void AddLink(string id, Link link)
            {
                Links.Add(id, link);
            }
        }
}
```

`Links` 字段是 `Dictionary<string, Link>` 类型，它包含与响应相关的资源的 URL。例如，如果我们获取特定项目的响应，`Link` 属性将提供添加、更新和删除项目的 URL。`AddLink` 方法用于向 `Links` 字典中添加新字段。因此，我们将继续在控制器中使用这种类型的响应，以便向客户端提供符合 HATEOAS 的反应。

# 在控制器中实现 HATEOAS

最后一步是实现一个可以处理之前实现的 `ItemHateoasResponse` 响应模型的控制器。更具体地说，我们可以在我们的 `Catalog.API` 项目中创建一个新的 `ItemHateoasController` 类。请注意，我们正在为了演示目的构建一个新的控制器。另一种选择是编辑已定义的 `ItemController` 以返回符合 HATEOAS 的响应。

`ItemHateoasController` 类将使用 `RiskFirst.Hateoas` 命名空间提供的 `ILinksService` 接口来丰富 `ItemHateoasResponse` 模型的 `Links` 属性并将其返回给我们的客户端。让我们通过实现控制器来继续：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Catalog.API.Filters;
using Catalog.API.ResponseModels;
using Catalog.Domain.Requests.Item;
using Catalog.Domain.Services;
using Microsoft.AspNetCore.Mvc;
using RiskFirst.Hateoas;

namespace Catalog.API.Controllers
{
    [Route("api/hateoas/items")]
    [ApiController]
    [JsonException]
    public class ItemsHateoasController : ControllerBase
    {
        private readonly IItemService _itemService;
        private readonly ILinksService _linksService;

        public ItemsHateoasController(ILinksService linkService, IItemService itemService)
        {
            _linksService = linkService;
            _itemService = itemService;
        }

        [HttpGet(Name = nameof(Get))]
        public async Task<IActionResult> Get([FromQuery] int pageSize = 10, [FromQuery] int pageIndex = 0)
        {
            var result = await _itemService.GetItemsAsync();

            var totalItems = result.Count();

            var itemsOnPage = result.OrderBy(c => c.Name)
                .Skip(pageSize * pageIndex)
                .Take(pageSize);

            var hateoasResults = new List<ItemHateoasResponse>();

            foreach (var itemResponse in itemsOnPage)
            {
                var hateoasResult = new ItemHateoasResponse { Data = itemResponse };
                await _linksService.AddLinksAsync(hateoasResult);

                hateoasResults.Add(hateoasResult);
            }

            var model = new PaginatedItemResponseModel<ItemHateoasResponse>(
                pageIndex, pageSize, totalItems, hateoasResults);

            return Ok(model);
        }

        [HttpGet("{id:guid}", Name = nameof(GetById))]
        [ItemExists]
        public async Task<IActionResult> GetById(Guid id)
        {
            var result = await _itemService.GetItemAsync(new GetItemRequest { Id = id });
            var hateoasResult = new ItemHateoasResponse { Data = result };
            await _linksService.AddLinksAsync(hateoasResult);

            return Ok(hateoasResult);
        }
```

`Get` 和 `GetById` 动作方法与 `ItemController` 中现有的方法非常相似。唯一的区别是它们返回不同的响应类型，该类型由 `ItemHateoasResponse` 类表示。此外，动作方法将 `response` 对象分配给 `Data` 字段。每个动作方法还调用由 `ILinksService` 接口提供的 `AddLinksAsync` 方法来填充链接属性。同样，我们还可以扩展控制器类中其他动作方法的行为：

```cs
...

        [HttpPost(Name = nameof(Post))]
        public async Task<IActionResult> Post(AddItemRequest request)
        {
            var result = await _itemService.AddItemAsync(request);
            return CreatedAtAction(nameof(GetById), new { id = result.Id }, null);
        }

        [HttpPut("{id:guid}", Name = nameof(Put))]
        [ItemExists]
        public async Task<IActionResult> Put(Guid id, EditItemRequest request)
        {
            request.Id = id;
            var result = await _itemService.EditItemAsync(request);

            var hateoasResult = new ItemHateoasResponse { Data = result };
            await _linksService.AddLinksAsync(hateoasResult);

            return Ok(hateoasResult);
        }

        [HttpDelete("{id:guid}", Name = nameof(Delete))]
        [ItemExists]
        public async Task<IActionResult> Delete(Guid id)
        {
            var request = new DeleteItemRequest { Id = id };
            await _itemService.DeleteItemAsync(request);
            return NoContent();
        }
..
```

这是创建、更新和删除操作方法的实现声明。在本例中，我们使用 `ItemHateoasResponse` 模型类来检索操作方法的响应。我们应该注意，操作方法在 `[HttpVerb]` 属性装饰器中声明了 `Name`，例如 `[HttpDelete("{id:guid}", Name = nameof(Delete))]`。实际上，我们将使用在 `Startup` 类中声明的 `Name` 来引用每个路由并将其包含在 `Link` 属性中：

```cs
namespace Catalog.API
{
    public class Startup
    {
        ...

        public void ConfigureServices(IServiceCollection services)
        {
          ...

            services.AddLinks(config => 
            {
                config.AddPolicy<ItemHateoasResponse>(policy =>
                {
                    policy
                        .RequireRoutedLink(nameof(ItemsHateoasController.Get), 
                         nameof(ItemsHateoasController.Get))
                        .RequireRoutedLink(nameof(ItemsHateoasController.GetById), 
                         nameof(ItemsHateoasController.GetById), _ => new {id = _.Data.Id})
                        .RequireRoutedLink(nameof(ItemsHateoasController.Post), 
                         nameof(ItemsHateoasController.Post))
                        .RequireRoutedLink(nameof(ItemsHateoasController.Put), 
                         nameof(ItemsHateoasController.Put), x => new {id = x.Data.Id})
                        .RequireRoutedLink(nameof(ItemsHateoasController.Delete), 
                         nameof(ItemsHateoasController.Delete), x => new {id = x.Data.Id});
                });
            });
        }
        ...
    }
}
```

由 `RiskFirst.Hateoas` 包提供的 `AddLinks` 扩展方法允许我们定义与响应模型相关的策略。这是 `config.AddPolicy<ItemHateoasResponse>` 方法的例子，它为 `ItemsHateoasController` 中声明的每个操作方法名称调用 `RequireRoutedLink`。请注意，与前面的情况类似，我们可以将此代码片段提取到外部扩展方法中，以使 `Startup` 类尽可能干净。最后，这种方法允许我们为不同的响应模型定义不同的链接组。此外，可以针对特定的响应模型建立策略。

因此，我们现在可以通过在 `Catalog.API` 文件夹中执行 `dotnet run` 命令来运行并验证结果。请注意，要运行 Docker SQL Server，请在 `appsetting.json` 文件中指定连接字符串。之后，我们可以运行以下 `curl` 请求来验证 `ItemsHateoasController`。生成的 JSON 响应将如下所示：

```cs
{
    "_links": {
        "get": {
            "rel": "ItemsHateoas/Get",
            "href": "https://localhost:5001/api/hateoas/items",
            "method": "GET"
        },
        "get_by_id": {
            "rel": "ItemsHateoas/GetById",
            "href": "https://localhost:5001/api/hateoas/items/8ff0fe8f-9dbc-451f-7a57-08d652340f56",
            "method": "GET"
        },
        "create": {
            "rel": "ItemsHateoas/Post",
            "href": "https://localhost:5001/api/hateoas/items",
            "method": "POST"
        },
        "update": {
            "rel": "ItemsHateoas/Put",
            "href": "https://localhost:5001/api/hateoas/items/8ff0fe8f-9dbc-451f-7a57-08d652340f56",
            "method": "PUT"
        },
        "delete": {
            "rel": "ItemsHateoas/Delete",
            "href": "https://localhost:5001/api/hateoas/items/8ff0fe8f-9dbc-451f-7a57-08d652340f56",
            "method": "DELETE"
        }
    },
    "id": "8ff0fe8f-9dbc-451f-7a57-08d652340f56",
    "name": "Malibu",

...
```

如您所见，除了项目列表之外，`ItemHateoasController` 还检索了一个提供额外信息的信封，例如客户端需要的其他用于获取、添加、更新和删除操作的路由。此外，这种方法提供了客户端需要的所有 URI，以便通过 Web 服务公开的信息进行导航。

# ASP.NET Core 中的异步代码

在本节中，我们将讨论 ASP.NET Core 的异步代码堆栈。我们已经处理了异步模式，并且已经看到前几章中的一些实现广泛使用了异步代码。

注意，以下部分主要关注基于 .NET Core 的 ASP.NET Core 中的异步代码。请记住，.NET Core 和 .NET Framework 中的异步编程之间存在一些差异。我们习惯于在 .NET Framework 中看到的某些死锁问题在 .NET Core 中不存在。

在我们深入探讨异步代码之前，让我们看看 ASP.NET Core 中同步和异步系统之间的区别。

首先，当一个同步 API 收到请求时，线程池中的一个线程将处理该请求。分配的线程将一直忙碌到请求的生命周期结束。在大多数情况下，API 对数据或第三方 API 执行 I/O 操作，这意味着线程将一直阻塞，直到这些操作结束。更具体地说，一个阻塞的线程不能被其他操作和请求使用。然而，对于异步代码，行为完全不同。

请求将被分配到一个线程，但该线程不会被锁定，它可以被其他操作分配和使用。如果任务被等待且不涉及 CPU 密集型工作，则可以将线程释放回池中以执行其他工作。

# 任务定义

`Task` 类型代表异步任务。`Task` 类代表一个工作单元。我们可以在其他语言中找到类似的概念。例如，在 JavaScript 中，`Task` 的概念由 `Promise` 表示。

将 `Task` 类型与 CPU 中的线程相关联是常见的，但这并不正确：将方法执行包装在 `Task` 中并不能保证该操作将在另一个线程上执行。

`async` 和 `await` 关键字是处理 C# 中异步操作的最简单方法：`async` 关键字将方法转换为状态机，并在方法实现中启用 `await` 关键字。`await` 关键字向编译器指示需要等待的操作，以便继续执行异步方法。因此，在 C# 代码库中经常可以找到类似的内容：

```cs
public async Task<String> GetStringAsync(String url)
{
    var request = await _httpClient.GetAsync(url);
    var responseContent = await request.Content.ReadAsStringAsync();
    return responseContent;
}
```

前面的代码片段非常直观：它通过 `_httpClient` 实例的 `GetAsync` 方法调用一个 `url`，并使用 `ReadAsStringAsync` 获取结果字符串并将其存储在 `responseContent` 对象中。重要的是要理解 `async` 和 `await` 是语法糖关键字，同样可以通过使用 `ContinueWith` 方法以不太易读的方式达到相同的结果：

```cs
public Task<String> GetStringAsync(String url) 
{ 
     var request =_httpClient.GetAsync(url); 
     var responseContentTask = request.ContinueWith(http => 
                         http.Result.Content.ReadAsStringAsync()); 
     return responseContentTask.Unwrap(); 
}
```

这个代码片段与前面的代码具有相同的效果。主要区别在于这个代码不太直观。此外，在执行大量嵌套事务的复杂操作中，我们会创建很多嵌套层级。

接受 `async`/`await` 语法作为处理异步代码的主要方式是至关重要的。其他语言也采取了类似的方法来使代码更加整洁和易于阅读。ECMAScript 就是这种情况，它在 ES 2016 中引入了 `async`/`await` 语法：[`github.com/tc39/ecma262/blob/master/README.md`](https://github.com/tc39/ecma262/blob/master/README.md).

# 在 ASP.NET Core 中异步代码的需求

首先需要强调的是，异步代码并非关乎速度。正如之前提到的，异步编程仅仅是关于不阻塞传入的请求。因此，真正的益处在于更好的垂直可伸缩性*，而不是提高我们代码的速度。例如，假设我们的网络服务执行一些 I/O 操作，比如对数据库的查询：如果我们以同步方式运行我们的代码堆栈，那么用于传入请求的线程将被阻塞，直到读取或写入（I/O 操作）过程完成，期间不会被其他请求使用。通过采用异步方法，我们能够在读取/写入操作执行后立即释放线程。因此，一旦 I/O 操作完成，应用程序将选择另一个或相同的线程以继续执行堆栈。

异步代码也为我们系统增加了开销，实际上，有必要添加额外的逻辑来协调异步操作。例如，让我们看一下以下代码：

```cs
[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
    // GET api/values
    [HttpGet]
    public async Task<ActionResult<string>> Get()
    {
        return await OperationAsync();
    }

    public async Task<string> OperationAsync()
    {
        await Task.Delay(100);
        return "Response";
    }
}
```

上述示例定义了一个调用异步方法的操作方法。借助一些外部工具，例如 ILSpy，可以反编译 C#代码并分析 IL 代码。**IL 代码**也被称为**中间代码**，这是由 C#编译器生成并在运行时执行的代码。

之前定义的`Get`方法的 IL 代码的结果转换如下：

```cs
[CompilerGenerated]
private sealed class <Get>d__0 : IAsyncStateMachine
{
    // Fields
    public int <>1__state;
    public AsyncTaskMethodBuilder<ActionResult<string>> <>t__builder;
    public ValuesController <>4__this;
    private string <>s__1;
    private TaskAwaiter<string> <>u__1;

    // Methods
    public <Get>d__0();
    private void MoveNext();
    [DebuggerHidden]
    private void SetStateMachine(IAsyncStateMachine stateMachine);
}
```

如您所见，该方法已被转换为一个实现了`IAsyncStateMachine`接口的密封类。`MoveNext`方法持续检查状态机的`__state`是否发生变化，并更新操作的结果。所有这些操作都是针对我们代码中包含的每个异步操作执行的。

# ASP.NET Core 的新特性是什么？

自从在旧 ASP.NET 框架中引入以来，`async`/`await`代码之前受到了一些*死锁问题*的影响。旧版本的 ASP.NET 使用一个名为`SynchronizationContext`的类，它本质上提供了一种将`Task`排队到上下文中的方式，当方法调用其他嵌套的异步方法时，并强制它们使用`.Result`或`.Wait()`关键字以同步方式执行，因此，这会导致请求上下文的死锁。

例如，假设以下代码在旧版本的 ASP.NET 上执行：

```cs
   public class ValuesController : ApiController
    {
        public string Get()
        {
            return Operation1Async().Result;
        }

        public async Task<string> Operation1Async()
        {
            await Task.Delay(1000);
            return "Test";
        }
    }
```

这导致应用程序出现死锁，因为`Get()`和`Operation1Async()`方法使用了相同的上下文。当`Operation1Async()`方法捕获上下文，该上下文将被用于继续运行方法时，`Get()`操作方法会阻塞上下文，因为它正在等待结果。

因此，开发者开始在他们异步代码中填充 `.ConfigureAwait(false)` 指令，这主要提供了一种通过避免之前看到的死锁问题来在不同的上下文中执行 `Task` 的方法：

```cs
   public class ValuesController : ApiController
    {
        public string Get()
        {
            return Operation1Async().Result;
        }

        public async Task<string> Operation1Async()
        {
            await Task.Delay(1000)
                      .ConfigureAwait(continueOnCapturedContext:false);
            return "Test";
        }
    }
```

这种方法可行，但我们应该考虑，通过调用 `Result` 或 `.Wait()`，我们正在失去异步编程模式提供的所有好处。

关于 ASP.NET Core 的好消息是它不使用 `SynchronizationContext`。*无上下文*的概念提供了异步代码的轻量级管理：当 ASP.NET 在将异步单元分配给线程之前，在请求上下文中排队每个异步单元，而 ASP.NET Core 则从分配的线程池中选取一个线程并将其附加到异步任务。

此外，ASP.NET 团队在 ASP.NET Core 方面做得非常出色，正如我们在前面的章节中看到的，ASP.NET Core 管道中的几乎每个组件都有同步和异步版本。例如，我们看到的动作过滤器就有同步和异步版本。第六章*，*过滤器管道*。

虽然在新的 .NET Core 版本中不再需要 `.ConfigureAwait(false)` 方法，但我们应记住，在某些代码库中它仍然很有用。如果您正在构建一个在 .NET Core 中编译但在旧版 .NET Framework 中也编译的 .NET 库，您仍然应该使用 `.ConfigureAwait(false)` 方法，以避免死锁。

# 异步编程的最佳实践

让我们来看看在 ASP.NET Core 中关于异步编程模型的良好和不良实践。首先，需要牢记的核心概念是避免在同步和异步代码之间混合。如前所述，ASP.NET Core 提供了大多数类和接口的同步和 `async` 版本。因此，在用同步代码阻塞异步堆栈之前，你应该探索并检查框架提供的所有替代方案。

以下建议来自 .NET 异步编程的高级视角。本书专注于框架的其他方面。如果您想了解更多关于如何使用异步编程的细节，我建议您访问以下链接：[`github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md`](https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md)。此外，如果您想了解 .NET 中并发的更一般概述，我建议您阅读以下书籍：*Concurrency in C# Cookbook* by Stephen Cleary ([`stephencleary.com/book/`](https://stephencleary.com/book/))。

# 不要使用 async void 方法

关于.NET Core 和更广泛的.NET 生态系统中异步编程的一个常见错误是声明一个方法为`async void`。实际上，当我们声明一个方法为`async void`时，我们将无法捕获该方法抛出的异常。此外，`Task`类型用于捕获方法的异常并将它们传播给调用者。总之，我们应该始终通过在方法返回`void`时返回`Task`类型来实现我们的`async`方法，否则，当方法返回一个类型时，我们应该返回`Task<T>`泛型类型。此外，`Task`类型在我们要通知调用者操作状态时也非常有用：`Task`类型通过以下属性公开操作状态：`Status`、`IsCanceled`、`IsCompleted`和`IsFaulted`。

# 使用 Task.FromResult 代替 Task.Run

如果你之前使用过.NET Core 或.NET Framework，你可能已经处理过`Task.FromResult`和`Task.Run`。两者都可以用来返回`Task<T>`。它们之间的主要区别在于它们的输入参数。看看下面的`Task.Run`代码片段：

```cs
 public Task<int> AddAsync(int a, int b)
   {
       return Task.Run(() => a + b);
   }
```

`Task.Run`方法会将执行作为线程池中的工作项排队。工作项将立即完成，并带有预计算的值。因此，我们浪费了线程池。此外，我们还应该注意到，`Task.Run`方法的初始目的是针对客户端.NET 应用程序的：ASP.NET Core 没有针对`Task.Run`操作进行优化，它不应该用来卸载代码的一部分执行。相反，让我们看看另一个案例：

```cs
 public Task<int> AddAsync(int a, int b)
   {
       return Task.FromResult(a + b);
   }
```

在这种情况下，`Task.FromResult`方法将预计算的值包装起来，而不会浪费线程池，这意味着我们不会承受`Task.Run`操作执行带来的开销。

# 启用取消

另一个重要的话题是启用异步操作的取消。如果我们查看我们在第八章和第九章中实现的服务层，我们可以看到在某些情况下，我们有传递`CancellationToken`作为参数的可能性。`CancellationToken`提供了一种轻量级的方式通知所有异步操作，消费者想要取消当前事务。此外，我们的代码可以检查`CancellationToken.IsCancellationRequested`属性以检测消费者是否请求取消任务。这种方法特别适合长时间运行的操作，因为我们的异步代码的消费者可以在过程中的任何时刻请求取消当前执行的任务。

# I/O 绑定操作中的异步代码

当我们以异步方式实现代码时，我们应该问自己底层过程是否涉及任何类型的 I/O 操作。如果是这种情况，我们应该使用该堆栈的异步方法进行操作。例如，`ItemRepository`类的`Get`操作涉及到对数据库的查询：

```cs
namespace Catalog.Infrastructure.Repositories
{
    public class ItemRepository : IItemRepository
    {
        ..
        public async Task<Item> GetAsync(Guid id)
        {
            var item = await _context.Items
                .AsNoTracking()
                .Where(x => x.Id == id)
                .Include(x => x.Genre)
                .Include(x => x.Artist).FirstOrDefaultAsync();

            return item;
        }
        ...
    }
}
```

在这种情况下，我们使用`FirstOrDefaultAsync`方法以异步方式执行操作。相反，如果我们以另一个操作为例，例如`ItemRepository`的`Add`方法，我们可以看到即使 EF Core 框架公开了`AddAsync`方法，该操作也不是异步的：

```cs
public Item Add(Item order)
{
    return _context.Items
        .Add(order).Entity;
}
```

这是因为在添加操作的情况下，我们并没有执行任何类型的 I/O 操作：实体被添加到`context`属性中，并标记为已添加状态。当调用`SaveChangesAsync`方法时，与数据库的正确同步发生，这个方法是异步的，因为它涉及到与数据库的 I/O 操作。总之，我们应该始终牢记理解我们想要以异步方式执行的操作的完整上下文和堆栈。一般来说，每次我们必须处理文件系统、数据库和任何其他网络调用时，我们应该使用异步堆栈来实现我们的代码，否则我们可以保持代码的同步，以避免额外的线程开销。

# 使用中间件测量响应时间

测量 ASP.NET Core 操作响应时间的一种常见方法是使用*中间件组件*。如第三章中所述，*与中间件管道一起工作*，这些组件在 ASP.NET Core 请求生命周期的边缘操作，并且对于执行跨切面实现非常有用。测量动作方法的响应时间属于这种实现情况。此外，中间件是第一个被请求击中的组件，也是最后一个可以处理输出响应的组件。这意味着我们可以分析和包含请求的几乎整个生命周期。

让我们看看`ResponseTimeMiddlewareAsync`的一个实现示例：

```cs
using System.Diagnostics;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Http;

namespace Catalog.API.Infrastructure.Middleware
{
    public class ResponseTimeMiddlewareAsync {  

        private const string X_RESPONSE_TIME_MS = "X-Response-Time-ms";  

        private readonly RequestDelegate _next;  

        public ResponseTimeMiddlewareAsync(RequestDelegate next) {  
            _next = next;  
        }  

        public Task InvokeAsync(HttpContext context) {  

            var watch = new Stopwatch(); 

            watch.Start();  

            context.Response.OnStarting(() => {  

                watch.Stop();  

                var responseTimeForCompleteRequest = watch.ElapsedMilliseconds;  
                context.Response.Headers[X_RESPONSE_TIME_MS] = responseTimeForCompleteRequest.ToString(); 

                return Task.CompletedTask;  
            });  

            return _next(context);  
        }  
    }  
}
```

之前的类定义了一个`async`中间件来检测请求的响应时间。它遵循以下步骤：

1.  它在`InvokeAsync`方法中声明了一个`Stopwatch`实例。

1.  它执行`Stopwatch`实例的`Start()`方法。

1.  它使用`OnStarting`方法定义了一个新的响应委托。`OnStarting`方法允许我们在响应头之前调用一个委托动作，这个动作将被发送到客户端。

1.  它调用`Stop()`方法，并使用`X-Response-Time-ms`自定义头设置`ElapsedMilliseconds`属性。

可以通过在`Startup`类中添加以下行将`ResponseTimeMiddlewareAsync`类包含在中间件管道中：

```cs
...
public void Configure(IApplicationBuilder app, IWebHostingEnvironment env)
{
    ...
    app.UseMiddleware<ResponseTimeMiddlewareAsync>();
    ...
}
...

```

此外，还可以通过实现以下方法，按照我们测试控制器所采取的相同方法来测试中间件：

```cs
using System.Threading.Tasks;
using Shouldly;
using Catalog.Fixtures;
using Xunit;

namespace Catalog.API.Tests.Middleware
{
    public class ResponseTimeMiddlewareTests : IClassFixture<InMemoryApplicationFactory<Startup>>
    {
        public ResponseTimeMiddlewareTests(InMemoryApplicationFactory<Startup> factory)
        {
            _factory = factory;
        }

        private readonly InMemoryApplicationFactory<Startup> _factory;

        [Theory]
        [InlineData("/api/items/?pageSize=1&pageIndex=0")]
        [InlineData("/api/artist/?pageSize=1&pageIndex=0")]
        [InlineData("/api/genre/?pageSize=1&pageIndex=0")]
        public async Task middleware_should_set_the_correct_response_time_custom_header(string url)
        {
            var client = _factory.CreateClient();
            var response = await client.GetAsync(url);

            response.EnsureSuccessStatusCode();
 response.Headers.GetValues("X-Response-Time-ms").ShouldNotBeEmpty();
        }
    }
}
```

`ResponseTimeMiddlewareTests` 类通过扩展 `InMemoryApplicationFactory` 类，使我们能够通过 HTTP 调用。我们可以检查响应对象中是否存在 `X-Response-Time-ms` 头部。需要注意的是，其他测量不包括 Web 服务器或应用程序池的启动时间。此外，当 Web 服务器未初始化时，它还会花费一些额外的时间。

# 摘要

在本章中，我们介绍了构建 API 的不同主题：从我们资源的软删除到如何测量响应的性能。我们还概述了 ASP.NET Core 的异步编程堆栈以及如何使用它的建议。本章涵盖的主题将在高级开发阶段对您有所帮助，以便提高返回数据的可读性和 Web 服务的性能。在下一章中，我们将看到如何使用容器化技术运行我们的目录解决方案。本章提供了 Docker 的概述以及相关的 Docker Compose 工具。
