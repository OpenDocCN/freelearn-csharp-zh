# 实现领域逻辑

本章重点介绍目录 Web 服务的逻辑层。如前所述，逻辑将被封装在 `Catalog.Domain` 项目中。本章展示了如何使用服务类方法实现应用程序逻辑。这些类的目的是在请求和数据源层实际使用的实体之间执行映射逻辑，并提供我们应用程序所需的所有附加逻辑。此外，我们还将看到如何测试实现代码以验证行为。

本章将涵盖以下主题：

+   如何实现我们应用程序的服务类

+   如何实现请求 DTO 和相关的验证系统

+   如何应用测试来验证实现的逻辑

以下章节中的代码可在以下 GitHub 仓库中找到：[`github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3`](https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3)。

# 实现服务类

让我们通过实现服务类来继续本章的具体部分。这一层抽象将定义查询数据层的各种方法，包括 `IItemRepository` 接口，并将结果数据映射。

如前所述，我们的服务实现将使用 DTO 类来通过堆栈传递数据。首先，让我们在 `Catalog.Domain` 项目中创建一个新的 `Requests/Item` 文件夹结构，并在文件夹中添加一个新的 `AddItemRequest.cs` 文件：

```cs
using System;
using Catalog.Domain.Entities;

namespace Catalog.Domain.Requests.Item
{
    public class AddItemRequest 
    {
        public string Name { get; set; }
        public string Description { get; set; }
        public string LabelName { get; set; }
        public Price Price { get; set; }
        public string PictureUri { get; set; }
        public DateTimeOffset ReleaseDate { get; set; }
        public string Format { get; set; }
        public int AvailableStock { get; set; }
        public Guid GenreId { get; set; }
        public Guid ArtistId { get; set; }
    } 
}
```

前面的代码定义了添加项目请求。该类与 `Item` 实体类非常相似，除了 `Id` 字段、`Artist` 字段和 `Genre` 字段不存在。此外，`Id` 字段将由 EF Core 实现，`Artist` 和 `Genre` 字段由 ORM 处理，以表示实体之间的关系。

同样，我们可以在同一文件夹中定义 `EditItemRequest` 类：

```cs
using System;
using Catalog.Domain.Entities;

namespace Catalog.Domain.Requests.Item
{
    public class EditItemRequest
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
        public string Description { get; set; }
        public string LabelName { get; set; }
        public Price Price { get; set; }
        public string PictureUri { get; set; }
        public DateTimeOffset ReleaseDate { get; set; }
        public string Format { get; set; }
        public int AvailableStock { get; set; }
        public Guid GenreId { get; set; }
        public Guid ArtistId { get; set; }
    }
}
```

在前面的代码片段中，该类包含与 `Item` 实体相同的字段，除了 `Artist` 和 `Genre` 字段，原因与前面描述的相同。正如您可以从类名中理解的那样，它代表更新项目操作。同样的方法也可以用于以下所示获取项目操作：

```cs
using System;

namespace Catalog.Domain.Requests.Item
{
    public class GetItemRequest
    {
        public Guid Id { get; set; }
    }
}

```

可能看起来有点冗余，为单个字段定义一个请求类。尽管如此，我们应该考虑我们的服务收到的 HTTP 请求可能会随时间变化。因此，这种方法确保我们能够在不向服务类的各种方法添加大量参数的情况下，使我们的请求能够进化。此外，将我们的传入请求表示为类提供了一个轻松的方式来版本化随时间演变的不同的请求类型。

让我们继续定义我们的服务类所使用的响应类。此外，在响应类的案例中，理解这一点至关重要：这种方法确保我们有一种避免向我们的网络服务客户端暴露所有字段的方法。作为第一步，我们需要在`Catalog.Domain`项目中定义一个新的`Responses`文件夹，并创建以下类：

```cs
// /Responses/Item/PriceResponse.cs
namespace Catalog.Domain.Responses
{
    public class PriceResponse
    {
        public decimal Amount { get; set; }
        public string Currency { get; set; }
    }
}

// /Response/Item/ArtistResponse.cs
using System;

namespace Catalog.Domain.Responses
{
    public class ArtistResponse
    {
        public Guid ArtistId { get; set; }
        public string ArtistName { get; set; }
    }
}

// /Response/Item/GenreResponse.cs
using System;

namespace Catalog.Domain.Responses
{
    public class GenreResponse
    {
        public Guid GenreId { get; set; }
        public string GenreDescription { get; set; }
    }
}

```

为了简洁起见，我已将`PriceResponse`、`GenreResponse`和`ArtistResponse`类的实现定义在一个代码片段中。这些类定义了在数据库端使用的相同实体类所使用的字段。除此之外，我们还将定义一个`ItemReposonse`类，它代表我们的服务响应：

```cs
using System;

namespace Catalog.Domain.Responses
{
    public class ItemResponse
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
        public string Description { get; set; }
        public string LabelName { get; set; }
        public PriceResponse Price { get; set; }
        public string PictureUri { get; set; }
        public DateTimeOffset ReleaseDate { get; set; }
        public string Format { get; set; }
        public int AvailableStock { get; set; }
        public Guid GenreId { get; set; }
        public GenreResponse Genre { get; set; }
        public Guid ArtistId { get; set; }
        public ArtistResponse Artist { get; set; }
    }
}
```

`ItemResponse`类引用其他响应类，以避免相关实体中包含的响应数据不匹配。此外，`IItemRepository`实现将使用我们在上一章中查看的`Include`扩展方法加载相关实体的所有数据，并且，正如我们稍后将看到的，数据将被映射到响应类型中。

# 服务类接口

由于我们已经定义了我们的服务所需的所有请求和响应类型，我们现在可以通过定义`IItemService`接口及其实现来继续前进。作为第一步，我们可以在`Catalog.Domain`项目中创建一个新的`Services`文件夹，并定义以下`IItemService`接口：

```cs
using System.Collections.Generic;
using System.Threading.Tasks;
using Catalog.Domain.Requests.Item;
using Catalog.Domain.Responses;

namespace Catalog.Domain.Services
{
    public interface IItemService
    {
        Task<IEnumerable<ItemResponse>> GetItemsAsync();
        Task<ItemResponse> GetItemAsync(GetItemRequest request);
        Task<ItemResponse> AddItemAsync(AddItemRequest request);
        Task<ItemResponse> EditItemAsync(EditItemRequest request);
        Task<ItemResponse> DeleteItemAsync(DeleteItemRequest request);
    }
}
```

上述定义暴露了我们的应用程序所需的方法。首先，我们应该注意到所有函数都返回一个`Task<T>`泛型类型。我们还可以看到所有方法都以`Async`前缀结尾，这表明实现将是异步的。

# 实现映射层

本小节和下一小节描述了我们可以在我们应用程序中使用的两种映射方法的实现：*手动方法*和*反射方法*。

*手动方法*涉及定义和实现我们自己的映射类：

```cs
using Catalog.Domain.Entities;
using Catalog.Domain.Requests.Item;
using Catalog.Domain.Responses;

namespace Catalog.Domain.Mappers
{
    public interface IItemMapper
    {
        Item Map(AddItemRequest request);
        Item Map(EditItemRequest request);
        ItemResponse Map(Item item);
    }
}
```

上述代码定义了一个`IItemMapper`接口，它提供了两个方法来映射`Item`类型中的`AddItemRequest`和`EditItemRequest`。此外，它还定义了将`Item`类型转换为`ItemResponse`实例的映射方法签名。这种策略可以通过以下`ItemMapper`类实现：

```cs
using Catalog.Domain.Entities;
using Catalog.Domain.Requests.Item;

namespace Catalog.Domain.Mappers
{
    public class ItemMapper : IItemMapper
    {
        public Item Map(AddItemRequest request)
        {
            if (request == null) return null;

            var item = new Item
            {
                Name = request.Name,
                Description = request.Description,
                LabelName = request.LabelName,
                PictureUri = request.PictureUri,
                ReleaseDate = request.ReleaseDate,
                Format = request.Format,
                AvailableStock = request.AvailableStock,
                GenreId = request.GenreId,
                ArtistId = request.ArtistId,
            };

            if (request.Price != null)
            {
                item.Price = new Price { Currency = request.Price.Currency, 
                 Amount = request.Price.Amount };
            }

            return item;
        }

        public Item Map(EditItemRequest request)
        {
            if (request == null) return null;

            var item = new Item
            {
                Id = request.Id,
                Name = request.Name,
                ...
                Format = request.Format,
                AvailableStock = request.AvailableStock,
                GenreId = request.GenreId,
                ArtistId = request.ArtistId,
            };

            if (request.Price != null)
            {
                item.Price = new Price { Currency = request.Price.Currency, 
                 Amount = request.Price.Amount };
            }

            return item;
        }
    }
}
```

请注意，为了简洁，我已经省略了映射中定义的实体所有字段，你可以在以下存储库中找到完整的映射类文件[`github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3/tree/master/Chapter09/Catalog`](https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3/tree/master/Chapter09/Catalog)。`IItemMapper`接口和`ItemMapper`类都位于`Catalog.Domain`项目的`Mappers`文件夹中。`ItemMapper`实现需要在开发方面投入一些开销，但它可以精确地完成你所需要的功能，而不产生任何运行时成本，例如反射。除此之外，逻辑被封装在单独的类中。同样的方法也可以应用于`ItemResponse`映射——在这种情况下，我们还需要为`Artist`和`Genre`实体创建一些隔离的映射器：

```cs
using Catalog.Domain.Entities;
using Catalog.Domain.Responses;

namespace Catalog.Domain.Mappers
{
    public interface IArtistMapper
    {
        ArtistResponse Map(Artist artist);
    }

    public class ArtistMapper : IArtistMapper
    {
        public ArtistResponse Map(Artist artist)
 {
           if (artist == null) return null; return new ArtistResponse
            {
 ArtistId = artist.ArtistId,
 ArtistName = artist.ArtistName
            };
 }
    }
}
```

为了简洁起见，我将接口和具体实现包含在一个独特的代码片段中。`IArtistMapper`公开了一个名为`Map`的方法，它根据`Artist`实体类初始化一个新的`ArtistResponse`。对于`Genre`实体，这种方法将是相同的：

```cs
using Catalog.Domain.Entities;
using Catalog.Domain.Responses;

namespace Catalog.Domain.Mappers
{
    public interface IGenreMapper
    {
        GenreResponse Map(Genre genre);
    }

    public class GenreMapper : IGenreMapper
    {
        public GenreResponse Map(Genre genre)
 {
           if (genre == null) return null; return new GenreResponse
            {
 GenreId = genre.GenreId,
 GenreDescription = genre.GenreDescription
            };
 }
    }
}
```

此外，在这种情况下，我们将`Genre`定义为`GenreResponse`映射。这两个映射类都可以独立使用或被其他映射器引用。一旦我们实现了`Artist`和`Genre`的映射逻辑，我们就可以将它们引用到`IItemMapper`中，以便定义`ItemReponse Map(Item item)`映射方法的实现：

```cs
using Catalog.Domain.Entities;
using Catalog.Domain.Responses;

namespace Catalog.Domain.Mappers
{
    public class ItemMapper : IItemMapper
    {
        private readonly IArtistMapper _artistMapper;
 private readonly IGenreMapper _genreMapper; 
        public ItemMapper(IArtistMapper artistMapper, IGenreMapper 
            genreMapper)
        {
            _artistMapper = artistMapper;
 _genreMapper = genreMapper;
        }

        ...

      public ItemResponse Map(Item item)
 {
            if (request == null) return null;
            var response = new ItemResponse
            {
                Id = request.Id,
                Name = request.Name,
                ...
                GenreId = request.GenreId,
                Genre = _genreMapper.Map(request.Genre),
                ArtistId = request.ArtistId,
                Artist = _artistMapper.Map(request.Artist),
            };

            if (request.Price != null)
            {
                response.Price = new PriceResponse { Currency = 
                 request.Price.Currency, Amount = request.Price.Amount };
            }

            return response;
        }
   }
}
```

我们已经更改了`ItemMapper`实现类，并将依赖项与`IArtistMapper`和`IGenreMapper`接口相结合。此外，我们可以使用我们刚刚定义的`Map`方法来根据`Item`实体创建`ItemResponse`实例。你可能已经注意到，我没有为`PriceResponse`实现映射类。这是因为像`Price`这样的实体不太可能发生变化。另一个需要注意的关键部分是，我们在映射接口及其实现之间缺少依赖注入的初始化；这部分将在本章后面进行介绍。

最后，我想明确指出，这并不是实现映射层在我们应用程序中的唯一方法。实际上，还有其他模式，例如使用扩展方法。让我们以`Artist`到`ArtistResponse`的映射为例：

```cs
public static class MappingExtensions
{
    public static ArtistResponse MapToResponse(this Artist artist)
    {
        return new ArtistResponse
        {
            ArtistId = artist.ArtistId,
            ArtistName = artist.ArtistName
        };
    }
}

```

上述代码定义了一个新的`MappingExtensions`静态类，它可以作为所有我们需要用于映射逻辑的扩展方法的容器。此外，我们还可以定义一个`MapToResponse`扩展方法，可以应用于以下方式中的`Artist`实体：

```cs
ArtistResponse artistResponse = artistEntity.MapToResponse();
```

扩展方法方法可以应用于领域模型的所有实体。尽管这种方法看起来更加直接，但它没有突出显示服务类与映射逻辑之间的依赖关系。因此，我更喜欢通过使用单独的类来实现映射，因为它提供了更好地理解应用程序依赖流的方式。

# 使用 Automapper 的映射逻辑

另一种方法是使用 `Automapper` NuGet 包来实现映射。如第五章中所述，“ASP.NET Core 中的 Web 服务堆栈”，这种方法利用.NET 提供的反射系统来匹配和映射我们类中的字段。可以使用以下 CLI 指令将 `Automapper` 包添加到 `Catalog.Domain` 项目中：

```cs
dotnet add package Automapper
```

Automapper 使用基于配置的结构来定义我们类的映射行为。让我们继续在 `Mappers` 文件夹中定义一个新的 `CatalogProfile` 类：

```cs
using AutoMapper;
using Catalog.Domain.Entities;
using Catalog.Domain.Requests.Item;
using Catalog.Domain.Responses.Item;

namespace Catalog.Domain.Mapper
{
    public class CatalogProfile : Profile
    {
        public CatalogProfile()
        {
            CreateMap<ItemResponse, Item>().ReverseMap();
            CreateMap<GenreResponse, Genre>().ReverseMap();
            CreateMap<ArtistResponse, Artist>().ReverseMap();
            CreateMap<Price, PriceResponse>().ReverseMap();
            CreateMap<AddItemRequest, Item>().ReverseMap();
            CreateMap<EditItemRequest, Item>().ReverseMap();
        }
    }
}
```

上述配置将由依赖注入引擎用于定义映射行为的列表。`Profile` 基类提供的 `CreateMap` 方法匹配两个泛型类型：`TSource` 和 `TDestination`。通过链式调用 `ReverseMap()` 扩展方法，也可以执行反向过程。这可以应用于我们在应用程序中定义的每个请求和响应类型。为了在我们的方法中使用映射逻辑，必须将 `IMapper` 类型注入到目标类中，并按以下方式执行 `Map` 方法：

```cs
_mapper.Map<ItemResponse>(new Item());
```

重要的是要注意，在以下情况下，`Map` 方法将抛出运行时异常：

+   映射的源和目标类型不对应

+   对应的源和目标映射在配置中未显式定义

+   实体中存在一些未映射的成员（这防止了映射目标中的意外 `null` 字段）

最后，Automapper 还需要使用 .NET Core 的依赖注入进行初始化。我们将在本章后面看到如何将 `Automapper` 添加到 DI 引擎中。

# 服务类实现

一旦完成映射层，我们就可以继续实现服务层。让我们首先在 `Catalog.Domain` 项目的 `Services` 文件夹中定义 `ItemService.cs` 文件。以下代码描述了构造函数方法和读取操作：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Catalog.Domain.Mappers;
using Catalog.Domain.Repositories;
using Catalog.Domain.Requests.Item;
using Catalog.Domain.Responses;

namespace Catalog.Domain.Services
{
    public class ItemService : IItemService
    {
        private readonly IItemRepository _itemRepository;
        private readonly IItemMapper _itemMapper;

        public ItemService(IItemRepository itemRepository, 
 IItemMapper itemMapper)
        {
 _itemRepository = itemRepository;
 _itemMapper = itemMapper;
        }

        public async Task<IEnumerable<ItemResponse>> GetItemsAsync()
        {
            var result = await _itemRepository.GetAsync();

            return result.Select(x => _itemMapper.Map(x));
        }

        public async Task<ItemResponse> GetItemAsync(GetItemRequest 
         request)
        {
            if (request?.Id == null) throw new ArgumentNullException();

            var entity = await _itemRepository.GetAsync(request.Id);

            return _itemMapper.Map(entity);
        }
    }
}
```

首先，我们可以看到该类同时引用了 `IItemRepository` 和 `IItemMapper` 接口，这些接口是通过构造函数注入技术注入的。这个小节还描述了 `GetItemsAsync` 和 `GetItemAsync` 函数的实现。这两个方法都使用 `IItemRepository` 接口从数据源检索数据，并使用 `IItemMapper` 接口在 `Item` 实体和 `ItemResponse` 之间进行映射。编写操作也可以采用相同的方法，具体实现如下：

```cs

namespace Catalog.Domain.Services
{
    public class ItemService : IItemService
    {
        ...

        public async Task<ItemResponse> AddItemAsync(AddItemRequest 
         request)
        {
            var item = _itemMapper.Map(request);
            var result = _itemRepository.Add(item);

            await _itemRepository.UnitOfWork.SaveChangesAsync();

            return _itemMapper.Map(result);
        }

        public async Task<ItemResponse> EditItemAsync(EditItemRequest 
         request)
        {
            var existingRecord = await 
             _itemRepository.GetAsync(request.Id);

            if (existingRecord == null)
            {
                throw new ArgumentException($"Entity with {request.Id} 
                 is not present");
            }

            var entity = _itemMapper.Map(request);
            var result = _itemRepository.Update(entity);

            await _itemRepository.UnitOfWork.SaveChangesAsync();

            return _itemMapper.Map(result);
        }
    }
}
```

此外，在写入操作的情况下，它们使用 `IItemMapper` 实例将请求的类型与 `Item` 实体类型进行映射，并检索 `ItemResponse` 类型。此外，它们通过调用 `IItemRepository` 实例执行操作，随后调用 `SaveChangesAsync` 方法将那些更改应用到数据库中。一旦我们实现了服务层，我们就可以继续测试该类并验证实现。

# 测试服务层

本节涵盖了之前实现的服务层部分的测试。正如我们在第八章，“构建数据访问层”中所做的那样，我们需要设置一个模拟的目录上下文，为测试服务类提供必要的数据。`Catalog.Infrastructure` 项目已经实现了自己的 `TestCatalogContext` 类和 `ModelBuilderExtensions` 类。此外，我们可以使用这两个相同的类来实现服务层的测试。我们需要的只是对 `Catalog.Infrastructure` 项目进行一点重构和优化。

# 重构测试类

在实现了 `ItemsRepositoryTests` 类型后，在第八章，“构建数据访问层”中，你可能会注意到我们在 `ItemRepositoryTests` 类中使用了重复的模式：

```cs
... 
  var options = new DbContextOptionsBuilder<CatalogContext>()
                .UseInMemoryDatabase(databaseName: "should_get_data")
                .Options;

            await using var context = new TestCatalogContext(options);
            context.Database.EnsureCreated();

            var sut = new ItemRepository(context);

...
```

之前的小节内容已经复制到迄今为止编写的每一个测试方法中。我们可以通过将实现提取到不同的类型来改进我们的测试代码。`xunit` 框架通过提供一个名为 `IClassFixture` 的接口，提供了一种在同一个测试类的方法之间共享测试上下文的方法。

`IClassFixture` 是一个泛型类型，它构成一个单独的测试上下文，并在类中的所有测试方法之间共享。因此，`xunit` 框架在类中的所有测试完成后清理固定装置。我们将要实现的 `IClassFixture` 接口将被 `Catalog.Infrastructure.Tests` 项目和 `Catalog.Domain.Tests` 项目使用。因此，我们可以在一个独特的 `Catalog.Fixtures` 项目中通用化实现。

让我们从在 `tests` 文件夹中创建新的项目开始：

```cs
dotnet new xunit -n Catalog.Fixtures -f netcoreapp3.1

dotnet sln ../Catalog.API.sln add  ./Catalog.Fixtures/Catalog.Fixtures.csproj
```

上述指令创建了一个新的 `Catalog.Fixtures` 项目并将其添加到解决方案中。之后，我们可以继续添加依赖项：

```cs
dotnet add ./Catalog.Fixtures reference ../src/Catalog.Domain/ dotnet add ./Catalog.Fixtures reference ../src/Catalog.Infrastructure/

```

最后，我们可以将之前在 `Catalog.Infrastructure.Tests` 项目中实现的全部类移动到刚刚创建的 `Catalog.Fixtures` 项目中：`TestCatalogContext.cs`、`Extensions/ModelBuilderExtensions.cs` 以及所有 `.json` 文件。

让我们继续创建一个新的 `CatalogContextFactory` 类，该类将被 `IClassFixture` 接口引用：

```cs
using System;
using Catalog.Domain.Mappers;
using Catalog.Infrastructure;
using Microsoft.EntityFrameworkCore;

namespace Catalog.Fixtures
{
    public class CatalogContextFactory
    {
        public readonly TestCatalogContext ContextInstance;
        public readonly IGenreMapper GenreMapper;
        public readonly IArtistMapper ArtistMapper;
        public readonly IItemMapper ItemMapper;

        public CatalogContextFactory()
        {
            var contextOptions = new 
                DbContextOptionsBuilder<CatalogContext>()
                .UseInMemoryDatabase(Guid.NewGuid().ToString())
                .EnableSensitiveDataLogging()
                .Options;

            EnsureCreation(contextOptions);
            ContextInstance = new TestCatalogContext(contextOptions);

            GenreMapper = new GenreMapper();
            ArtistMapper = new ArtistMapper();
            ItemMapper = new ItemMapper(ArtistMapper, GenreMapper);
        }

        private void EnsureCreation(DbContextOptions<CatalogContext> 
         contextOptions)
        {
            using var context = new TestCatalogContext(contextOptions);
            context.Database.EnsureCreated();
        }
    }
}
```

`CatalogContextFactory` 类使用先前分配的 `contextOptions` 对象定义了一个新的 `TestCatalogContext` 实例。需要注意的是，我们正在使用 `Guid.NewGuid().ToString()` 属性作为数据库名来构建 `ContextOptions`，以便为每个测试类提供一个新的、干净的内存实例。此外，该类还初始化了三个类型为 `IGenreMapper`、`IArtistMapper` 和 `IItemMapper` 的属性，这些属性将被服务层测试用于执行字段的映射。

因此，我们可以在测试中使用以下构造函数注入方法来访问工厂类的实例：

```cs
using System;
using System.Linq;
using System.Threading.Tasks;
using Catalog.Domain.Entities;
using Catalog.Fixtures;
using Catalog.Infrastructure.Repositories;
using Newtonsoft.Json;
using Shouldly;
using Xunit;

namespace Catalog.Infrastructure.Tests
{
    public class ItemRepositoryTests :
        IClassFixture<CatalogContextFactory>
    {
        private readonly ItemRepository _sut;
 private readonly TestCatalogContext _context;

        public ItemRepositoryTests(CatalogContextFactory catalogContextFactory)
        {
            _context = catalogContextFactory.ContextInstance;
 _sut = new ItemRepository(_context);
        }
...
```

`IClassFixture` 接口包含对刚刚创建的工厂类的引用。依赖关系将在测试类的构造函数中运行时解析。请注意，整个实例在各个唯一的测试类之间是共享的。因此，测试类中的每个测试方法都将与其他方法共享相同的数据库快照。

最后，我们可以重构 `ItemRepositoryTests` 类以使用新的 `CatalogContextFactory` 实现。例如，如果我们以 `should_add_new_item` 测试方法为参考，我们可以按以下方式进行：

```cs

    public class ItemRepositoryTests : IClassFixture<CatalogContextFactory>
    {
        private readonly ItemRepository _sut;
 private readonly TestCatalogContext _context;

 public ItemRepositoryTests(CatalogContextFactory 
            catalogContextFactory)
 {
 _context = catalogContextFactory.ContextInstance;
 _sut = new ItemRepository(_context);
 }

        [Fact]
        public async Task should_add_new_item()
        {
            var testItem = new Item
            {
                Name = "Test album",
                Price = new Price { Amount = 13, Currency = "EUR" },
                GenreId = new Guid("c04f05c0-f6ad-44d1-a400-
                    3375bfb5dfd6"),
                ArtistId = new Guid("f08a333d-30db-4dd1-b8ba-
                    3b0473c7cdab"),
                ...
            };

            _sut.Add(testItem);
            await _sut.UnitOfWork.SaveEntitiesAsync();

            _context.Items
                .FirstOrDefault(item => item.Id == testItem.Id)
                .ShouldNotBeNull();
        }
    }
```

`_sut` 类属性用于执行我们想要测试的实际操作。例如，在上面的测试用例中，我们正在验证 `ItemRepository` 类公开的 `Add` 方法。`_context` 属性用于验证结果。这种方法通过为测试提供更好的可维护性，确保了我们的测试代码在不同测试类之间的可重用性。所有数据都由 `CatalogContextFactory` 类型提供，该类型使用 ASP.NET Core 提供的内存数据库技术将数据存储在内存中，并模拟对真实数据库的数据操作。

正如我们在 `ItemRepositoryTests` 类中所做的那样，我们还将看到如何在服务层测试中使用 `CatalogContextFactory` 类。

本节中的代码可在以下 GitHub 仓库中找到：[`github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3`](https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3)。

# 实现 ItemService 测试

让我们继续通过实现 `ItemService` 测试部分来继续。首先，我们应该通过在 `tests` 文件夹中使用以下 CLI 指令创建一个新的 `Catalog.Domain.Tests` 项目来开始：

```cs
dotnet new xunit -n Catalog.Domain.Tests -f netcoreapp3.1
```

上述命令在`tests`文件夹中创建了一个新的`Catalog.Domain.Tests`项目。因此，我们可以通过以下指令将新项目添加到解决方案中：

```cs
dotnet sln ../Catalog.API.sln add Catalog.Domain.Tests 
```

此外，测试项目还有一些依赖项。此外，我们可以通过以下命令将引用添加到`Catalog.Domain.Tests`文件夹中：

```cs
dotnet add reference ../Catalog.Fixtures
dotnet add reference ../../src/Catalog.Domain
dotnet add package Shouldly
```

之后，我们创建一个新的`ItemServiceTests.cs`文件，其实现如下：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;
using Catalog.Domain.Entities;
using Catalog.Domain.Mappers;
using Catalog.Domain.Requests.Item;
using Catalog.Domain.Services;
using Catalog.Fixtures;
using Catalog.Infrastructure.Repositories;
using Shouldly;
using Xunit;

namespace Catalog.Domain.Tests.Services
{
    public class ItemServiceTests : 
        IClassFixture<CatalogContextFactory>
    {
 private readonly ItemRepository _itemRepository;
 private readonly IItemMapper _mapper;

 public ItemServiceTests(CatalogContextFactory catalogContextFactory)
 {
 _itemRepository = new ItemRepository(catalogContextFactory.ContextInstance);
 _mapper = catalogContextFactory.ItemMapper;
 }

        [Fact]
        public async Task getitems_should_return_right_data()
        {
            ItemService sut = new ItemService(_itemRepository, _mapper);

            var result = await sut.GetItemsAsync();
            result.ShouldNotBeNull();
        }

        [Theory]
        [InlineData("b5b05534-9263-448c-a69e-0bbd8b3eb90e")]
        public async Task getitem_should_return_right_data(string guid)
        {
            ItemService sut = new ItemService(_itemRepository, _mapper);

            var result = await sut.GetItemAsync(new GetItemRequest { Id = new Guid(guid) });

            result.Id.ShouldBe(new Guid(guid));
        }

        [Fact]
        public void getitem_should_thrown_exception_with_null_id()
        {
            ItemService sut = new ItemService(_itemRepository, _mapper);

            sut.GetItemAsync(null).ShouldThrow<ArgumentNullException>();
        }
    }
}
```

上述代码测试了`ItemService`类读取操作的实现。`ItemServiceTests`类使用`CatalogContextFactory`类型来初始化并获取服务所使用的基本数据。每个测试方法都使用`_itemRepository`类属性和`_mapper`实例来初始化一个新的`ItemService`并验证由服务类提供的`GetItemAsync`和`GetItemsAsync`方法。

同样，我们可以使用相同的技巧来实现写入操作的测试：

```cs

    public class ItemServiceTests : 
        IClassFixture<CatalogContextFactory>
    {
        private readonly ItemRepository _itemRepository;
        private readonly IItemMapper _mapper;

        public ItemServiceTests(CatalogContextFactory catalogContextFactory)
        {
            _itemRepository = new ItemRepository(catalogContextFactory.ContextInstance);
            _mapper = catalogContextFactory.ItemMapper;
        }

        ...

        [Fact]
 public async Task additem_should_add_right_entity()
        {
            var testItem = new AddItemRequest
            {
                Name = "Test album",
                GenreId = new Guid("c04f05c0-f6ad-44d1-a400-3375bfb5dfd6"),
                ArtistId = new Guid("f08a333d-30db-4dd1-b8ba-3b0473c7cdab"),
                Price = new Price { Amount = 13, Currency = "EUR" }
                ...
            };

            IItemService sut = new ItemService(_itemRepository, _mapper);

            var result = await sut.AddItemAsync(testItem);

            result.Name.ShouldBe(testItem.Name);
            result.Description.ShouldBe(testItem.Description);
            result.GenreId.ShouldBe(testItem.GenreId);
            result.ArtistId.ShouldBe(testItem.ArtistId);
            result.Price.Amount.ShouldBe(testItem.Price.Amount);
            result.Price.Currency.ShouldBe(testItem.Price.Currency);
        }

        [Fact]
 public async Task edititem_should_add_right_entity()
        {
            var testItem = new EditItemRequest
            {
                Id = new Guid("b5b05534-9263-448c-a69e-0bbd8b3eb90e"),
                Name = "Test album",
                GenreId = new Guid("c04f05c0-f6ad-44d1-a400-3375bfb5dfd6"),
                ArtistId = new Guid("f08a333d-30db-4dd1-b8ba-3b0473c7cdab"),
                Price = new Price { Amount = 13, Currency = "EUR" }
                ...
            };

            ItemService sut = new ItemService(_itemRepository, _mapper);

            var result = await sut.EditItemAsync(testItem);

            result.Name.ShouldBe(testItem.Name);
            result.Description.ShouldBe(testItem.Description);
            result.GenreId.ShouldBe(testItem.GenreId);
            result.ArtistId.ShouldBe(testItem.ArtistId);
            result.Price.Amount.ShouldBe(testItem.Price.Amount);
            result.Price.Currency.ShouldBe(testItem.Price.Currency);
        }
    }
}
```

`additem_should_add_the_entity`和`edititem_should_edit_the_entity`方法正在验证`IItemMapper`逻辑实现以及`IItemService`实现。这种方法可以用来测试服务类层的逻辑。在这种情况下，映射逻辑并不复杂。此外，我还建议在映射逻辑更复杂的情况下实现单独的测试。

最后，我们可以在解决方案文件夹中通过执行`dotnet test` CLI 指令或在首选的 IDE 测试运行器中运行我们刚刚实现的测试用例。CLI 结果应该类似于以下内容：

![图片](img/43a9d778-e29d-47e7-92c4-d90d8836bdde.png)

上述报告列出了由`dotnet test`命令执行的测试，并提供了成功和失败测试的概述。此外，还可以通过在`dotnet test -v <q[uiet], m[inimal], n[ormal], d[etailed], and diag[nostic]>`命令旁边附加`-v`选项来指定测试的详细程度。

# 实现请求模型验证

`Catalog.Domain`项目也拥有我们请求模型的验证逻辑。在本节中，我们将看到如何实现请求验证逻辑部分，这部分也将被我们的控制器用来验证传入的数据。在这里，我通常依赖于`FluentValidation`包，它提供了一种非常易于阅读的方式来对每种类型的对象和数据结构执行验证检查。

为了将`FluentValidation`包添加到我们的`Catalog.Domain`项目中，我们可以在项目文件夹中执行以下命令：

```cs
dotnet add package FluentValidation
dotnet add package FluentValidation.AspNetCore

```

`FluentValidation`包公开了`AbstractValidation`类，它可以被扩展以实现针对请求模型类的自定义验证标准：

```cs
//Requests/Item/Validators/AddItemRequestValidator.cs using FluentValidation;

namespace Catalog.Domain.Requests.Item.Validators
{
    public class AddItemRequestValidator : AbstractValidator<AddItemRequest>
    {
        public AddItemRequestValidator()
        {
            RuleFor(x => x.GenreId).NotEmpty();
            RuleFor(x => x.ArtistId).NotEmpty();
            RuleFor(x => x.Price).NotEmpty();
            RuleFor(x => x.ReleaseDate).NotEmpty();
            RuleFor(x => x.Name).NotEmpty();
            RuleFor(x => x.Price).Must(x => x?.Amount > 0);
            RuleFor(x => x.AvailableStock).Must(x => x > 0);
        }
    }
}
//Requests/Item/Validators/EditItemRequestValidator.cs

using FluentValidation;

namespace Catalog.Domain.Requests.Item.Validators
{
    public class EditItemRequestValidator : AbstractValidator<EditItemRequest>
    {
        public EditItemRequestValidator()
        {
            RuleFor(x => x.Id).NotEmpty();
            RuleFor(x => x.GenreId).NotEmpty();
            RuleFor(x => x.ArtistId).NotEmpty();
            RuleFor(x => x.Price).NotEmpty();
            RuleFor(x => x.Price).Must(x => x?.Amount > 0);
            RuleFor(x => x.ReleaseDate).NotEmpty();
            RuleFor(x => x.Name).NotEmpty();
        }
    }
}
```

这些验证类位于`Requests/Item/Validators`路径。让我们通过分析`AddItemRequestValidator`类中实现的一些验证标准来继续：

+   `GenreId`和`ArtistId`字段是必需的，因为黑胶唱片总是有这类信息。

+   该类在`Name`、`ReleaseDate`和`Price`字段上使用相同的`NotEmpty`方法。

+   `Price`类的`Amount`字段始终应该大于 0。验证器类使用`Must`方法应用此规则。

对于`EditItemRequestValidator`类，采用相同的方法，除了`Id`字段，它在实体的更新过程中被定义为必需的。流畅的工作方式确实非常有用，原因有很多：它易于阅读，易于维护，并且在我们想要测试逻辑时非常有帮助。

# 测试请求模型验证

如前所述，`FluentValidation`包提供了一种出色的方式来测试我们的验证标准。

`AddItemRequestValidator`和`EditItemRequestValidator`类实现了基本检查。此外，可能有用的是用一些测试来覆盖它们，以记录这些类中实现的逻辑。`FluentValidation`提供了一个`TestHelper`类，它提供了验证我们验证逻辑行为的必要断言条件。

让我们看看如何为`AddItemRequestValidator`类进行一些单元测试：

```cs
using Catalog.Domain.Entities;
using Catalog.Domain.Requests.Item;
using Catalog.Domain.Requests.Item.Validators;
using FluentValidation.TestHelper;
using Xunit;

namespace Catalog.Domain.Tests.Requests.Item.Validators
{
    public class AddItemRequestValidatorTests
    {
        private readonly AddItemRequestValidator _validator;

        public AddItemRequestValidatorTests()
        {
            _validator = new AddItemRequestValidator();
        }

        [Fact]
        public void should_have_error_when_ArtistId_is_null()
        {
            var addItemRequest = new AddItemRequest { Price = new Price() };
            _validator.ShouldHaveValidationErrorFor(x => x.ArtistId, addItemRequest);
        }

        [Fact]
        public void should_have_error_when_GenreId_is_null()
        {
            var addItemRequest = new AddItemRequest { Price = new Price() };
            _validator.ShouldHaveValidationErrorFor(x => x.GenreId, addItemRequest);
        }
    }
}
```

前述代码中定义的测试类验证了如果`GenreId`或`ArtistId`字段为空，则`AddItemRequestValidator`会触发验证错误。它使用`TestHelper`类公开的`ShouldHaveValidationErrorFor`扩展方法来验证行为。`ShouldHaveValidationErrorFor`方法还公开了一个`IEnumerable`的`ValidationError`，可以用来检查类型为`ValidationError`的每个消息的详细信息。

# 依赖项注册

在本章中，我们看到了如何实现映射类、验证器和服务类。所有这些类型都通过.NET Core 的依赖注入一起工作。依赖项注册通常通过使用按某些标准分组注册类的扩展方法来完成。在这种情况下，我将按以下方式分组注册的类：

+   服务指的是在`Catalog.Domain`项目中定义的所有服务接口和类。

+   Mappers 指的是在`Catalog.Domain`项目中定义的所有映射类。

+   验证指的是应用程序使用的所有流畅验证要求和依赖项。

现在我们已经定义了依赖项分离背后的逻辑，我们可以在`Catalog.Domain`项目的`Extensions`文件夹中定义一个新的`DependencyRegistration`静态类来继续：

```cs
using System.Reflection;
using Catalog.Domain.Mappers;
using Catalog.Domain.Services;
using FluentValidation.AspNetCore;
using Microsoft.Extensions.DependencyInjection;

namespace Catalog.Domain.Extensions
{
    public static class DependenciesRegistration
    {
        public static IServiceCollection AddMappers(this IServiceCollection services)
        {
            services
                .AddSingleton<IArtistMapper, ArtistMapper>()
                .AddSingleton<IGenreMapper, GenreMapper>()
                .AddSingleton<IItemMapper, ItemMapper>();

            return services;
        }

        public static IServiceCollection AddServices(this IServiceCollection services)
        {
            services
                .AddScoped<IItemService, ItemService>();

            return services;
        }

        public static IMvcBuilder AddValidation(this IMvcBuilder builder)
        {
            builder
                .AddFluentValidation(configuration =>
                    configuration.RegisterValidatorsFromAssembly
                        (Assembly.GetExecutingAssembly()));

            return builder;
        }
    }
}
```

上述代码定义了三个扩展方法，每个组一个：`AddMappers`、`AddServices`和`AddValidation`。

`AddMappers` 扩展方法使用 *单例* 生命周期来注册映射实例，因此，映射器没有任何依赖，也不执行任何与请求相关的操作。另一方面，`AddServices` 扩展方法使用作用域生命周期，因为服务类依赖于在数据库上执行 I/O 操作的仓储。最后，`AddValidation` 扩展方法与 `IMvcBuilder` 链接，并且严格依赖于 MVC 堆栈。

此外，它使用 `FluentValidation` 包提供的 `AddFluentValidation` 方法来注册所有验证类。

总之，我们可以以下方式注册应用程序的依赖项：

```cs
using Catalog.API.Extensions;
...

namespace Catalog.API
{
    public class Startup
    {
        ...
        public void ConfigureServices(IServiceCollection services)
        {
            services
                .AddCatalogContext(Configuration.GetSection("DataSource:ConnectionString").Value)
                .AddScoped<IItemRepository, ItemRepository>()
                .AddMappers()
 .AddServices()
 .AddControllers()
 .AddValidation();
        }
...
```

最后，我们可以通过在解决方案文件夹中再次运行 `dotnet test` 命令，或者执行我们首选 IDE 的测试运行器来验证本章编写的实现。

# 摘要

`Catalog.Domain` 项目现在包含整个应用程序的核心逻辑。尽管在 `Domain` 项目中实现的逻辑仍然相当简单，但在这本书的后续部分，它将变得更加复杂。

本章涵盖的主题包括一些与 Web 服务领域逻辑实现相关的概念：如何实现服务和映射类，如何使用流畅的方法实现请求验证过程，以及最后如何使用一些单元测试技术来测试我们的代码。

下一章将探讨应用程序的所有 HTTP 部分。此外，我们将关注 `Catalog.API` 项目，以及如何组合数据访问、服务和 API 层。
