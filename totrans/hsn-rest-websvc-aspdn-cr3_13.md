# 实现RESTful HTTP层

在上一章中，我们学习了如何在 `Catalog.Domain` 项目中处理我们网络服务的逻辑。本章将向您介绍网络服务的 HTTP 部分，以及 `Catalog.API` 项目中的所有组件。

我们还将演示如何实现和测试网络服务的控制器部分。到本章结束时，您将能够使用 ASP.NET Core 实现、测试和验证 HTTP 路由。我们将涵盖以下主题：

+   实现服务的 HTTP 层

+   使用 ASP.NET Core 提供的工具进行测试

+   提高HTTP层的弹性

本章中展示的代码可以从以下 GitHub 仓库获取：[https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3](https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3).

# 实现项目控制器

本节重点介绍构建读取、写入目录数据和使用 HTTP 协议公开我们在领域层中已构建的功能的路线。我们的控制器将包括以下路由表中列出的动词：

| **动词** | **路径** | **描述** |
| --- | --- | --- |
| `GET` | `/api/items` | 获取我们目录中存在的所有项目 |
| `GET` | `/api/items/{id}` | 获取具有相应 ID 的项目 |
| `POST` | `/api/items/` | 通过请求体有效载荷创建一个新的项目 |
| `PUT` | `/api/items/{id}` | 更新具有相应 ID 的项目 |

上述路由允许网络服务消费者获取、添加和更新 `Item` 实体。在开始实现之前，让我们看一下我们将要实现的解决方案架构概述：

![](img/d956dbc7-7b0d-4763-90fd-f7fa25b3b59f.png)

在 [第 8 章](84b281bd-11a2-4703-81ae-ca080e2a267a.xhtml)，*构建数据访问层* 和 [第 9 章](f8ac60e1-e948-4435-b804-e3e4ff305510.xhtml)，*实现领域逻辑* 中，我们分别实现了并测试了 `Catalog.Infrastructure` 和 `Catalog.Domain` 项目。本章重点介绍 `Catalog.API` 项目。我们将构建并测试将调用在 `Catalog.Domain` 项目中构建的服务层的动作方法。让我们首先在 `Catalog.API` 项目的 `Controllers` 文件夹中定义一个新的控制器，命名为 `ItemController`：

[PRE0]

`ItemController` 类将反映我们之前定义的路由。我们应该注意，我们使用 `Route` 和 `ApiController` 属性装饰了控制器类：第一个指定了控制器的基 URL，第二个为动作方法产生的响应类型提供了一些实用工具和约定。控制器还将使用 `IItemService` 接口来查询和写入我们的数据源。我们可以通过构造函数注入将 `IItemService` 接口用于 `ItemController` 类：

[PRE1]

之前的代码使用依赖注入将 `IItemService` 类添加为 `ItemController` 类的依赖项。一旦我们添加了 `IItemService` 接口，我们就可以通过实现控制器中的操作方法来继续进行。

# 实现操作方法

我们已经在 [第 5 章](deede298-fc20-4523-afa6-02ed2c0592fd.xhtml) 中处理了操作方法，*ASP.NET Core 中的网络服务堆栈*。在以下实现中，我们将在操作方法中使用 `IItemService` 接口，如下所示：*

[PRE2]

`Get` 和 `GetById` 操作方法通过引用 `IItemService` 接口并调用底层服务层（在这种情况下是 `IItemService` 接口）的 `GetItemsAsync` 和 `GetItemAsync` 方法来执行读取操作。让我们继续使用相同的方法来实现控制器的 `Post` 和 `Put` 操作方法：

[PRE3]

`Post` 和 `Put` 动作分别使用 `AddItemRequest` 和 `EditItemRequest` 来绑定来自 HTTP 请求的数据，并通过 `IItemService` 接口传递。在底层，`IItemService` 实现引用 `IItemMapper` 来从请求类型获取实体，并通过 `IItemRepository` 实现发送。借助依赖注入，我们可以轻松地将不同组件之间的依赖解耦。我们还应该注意，`Post` 操作方法使用 `ControllerBase` 提供的 `CreatedAtAction()` 方法来检索创建资源的位置作为响应的一部分。一旦我们将 `IItemService` API 绑定到 `ItemController` 操作方法中，我们就可以继续测试实现。

# 使用 WebApplicationFactory<T> 类测试控制器

ASP.NET Core 框架提供了一个使用 `WebApplicationFactory<T>` 类执行 *集成测试* 的方法。这个类允许我们创建一个新的 `TestServer`，它在一个单独的进程中模拟真实的 HTTP 服务器。因此，我们可以通过调用由工厂提供的 `HttpClient` 实例来测试我们的 `ItemController`。需要注意的是，`WebApplicationFactory` 是一个泛型类，它接受一个 `TEntryPoint` 类型，这由我们的网络服务的 `Startup` 类表示。在继续实现测试类之前，让我们在 `tests` 文件夹中创建一个新的项目，该项目将包含与 `Catalog.API` 项目相关的所有测试。因此，我们可以在 `tests` 文件夹中执行以下命令：

[PRE4]

之前的命令在解决方案的 `tests` 文件夹中添加了一个新的 `Catalog.API.Tests` 项目，它引用了 `Catalog.Fixtures` 和 `Catalog.API` 项目。该项目包含在项目的解决方案文件中。下一节将描述如何扩展 `WebApplicationFactory` 类以支持执行网络服务。

# 扩展 WebApplicationFactory

`WebApplicationFactory`类公开了用于配置`TestServer`实例和为我们的控制器创建适当测试固定值的属性和方法。此外，可以通过覆盖`ConfigureWebHost`方法并替换`Catalog.API`项目原始`Startup`类中声明的*依赖注入服务*的行为来扩展`WebApplicationFactory`。`WebApplicationFactory`类是`Microsoft.AspNetCore.Mvc.Testing`包的一部分；因此，有必要通过在项目的测试文件夹中运行以下命令将NuGet包添加到`Catalog.Fixture`项目和`Catalog.API.Tests`项目：

[PRE5]

让我们继续在`Catalog.Fixtures`项目中创建一个新的`InMemoryWebApplicationFactory`类。该类将由测试类用于实例化一个新的`TestServer`对象。因此，下一步是创建一个新的`InMemoryWebApplicationFactory`类，它扩展了`WebApplicationFactory`基类并覆盖了`ConfigureWebHost`方法以注入自定义的*内存*数据库提供程序：

[PRE6]

之前的`InMemoryApplicationFactory`类实现了`ConfigureWebHost`方法，并使用`UseInMemoryDatabase`扩展方法初始化内存数据库。它还在使用依赖注入注册的`CatalogContext`服务中插入`TestCatalogContext`类的新实例。因此，测试将使用我们在`Catalog.Infrastructure.Tests`和`Catalog.Domain.Tests`项目中实现的测试用例所使用的相同内存数据库基础设施。

此外，`InMemoryApplicationFactory`实现创建了一个新作用域，该作用域将用于执行`EnsureCreated`方法。因此，每个新的`InMemoryApplicationFactory`实例将生成来自相同数据快照的数据库。

最后，整个实现都在`ConfigureTestServices`方法中执行，该方法提供了一种覆盖`Catalog.API`项目`Startup`类中定义的依赖注入服务的方法。

如[第8章](84b281bd-11a2-4703-81ae-ca080e2a267a.xhtml)中所述，*构建数据访问层*，内存数据库并不总是首选的替代方案，有两个原因。首先，它不能反映具有真实数据约束的真实关系数据库。其次，当多个测试方法使用相同的实例时，处理内存数据库很棘手，因为它们可能会生成不一致的数据。因此，我们为每个测试类创建一个新的实例，使用`UseInMemoryDatabase(Guid.NewGuid().ToString());`语句。`Guid.NewGuid()`指令保证了实例之间的唯一性。在实际应用中，另一种常见的方法是创建一个临时数据源的新实例，并在每次测试后重新创建它。

# 测试控制器

一旦我们实现了 `InMemoryApplicationFactory` 类，就可以通过在测试类中实现 `IClassFixture` 接口来利用它。因此，让我们首先在 `Catalog.API.Tests` 项目中初始化一个新的 `ItemControllerTests` 类：

[PRE7]

`ItemControllerTests` 类为操作方法提供了显著的测试覆盖率。首先，测试类实现了由 `xUnit.Sdk` 包提供的通用 `IClassFixture` 接口。`IClassFixture` 接口引用了之前定义的 `InMemoryApplicationFactory<Startup>`，并将 `factory` 类的新实例注入到测试类的构造函数中。因此，对于每个执行的测试类，都将提供一个 `factory` 的新实例。

让我们看看覆盖 `ItemController` 获取操作的测试方法：

[PRE8]

之前实现的代码使用了由 `InMemoryApplicationFactory<Startup>` 提供的 `CreateClient` 方法来初始化一个新的 `HttpClient` 实例。因此，如果我们以 `get_by_id_should_return_item_data` 方法为例，它使用客户端调用 `/api/items/{id}` 路由，并检查返回的信息不是 `null`。我们可以通过向 `ItemControllerTests` 类中添加以下测试方法来继续测试添加项目操作：

[PRE9]

因此，我们可以为控制器中实现的 `Put` 操作方法选择类似的方法：

[PRE10]

`add_should_create_new_record` 测试方法和 `update_should_modify_existing_item` 方法采用了相应的策略来测试 `Post` 和 `Put` 请求以及相应的操作方法。在这种情况下，我们使用了为 `ItemServiceTests` 和 `ItemRepositoryTests` 类定义的相同请求对象。

我们可以通过在解决方案文件夹中运行 `dotnet test` 命令或使用我们首选 IDE 的测试运行器来执行之前实现的测试。在接下来的下一小节中，我们将探讨如何优化请求的初始化并在一个独特的点保持测试数据。

使用 `IClassFixture` 意味着相同的 `InMemoryApplicationFactory` 实例将由所有测试方法共享。因此，每个测试方法将具有相同的基本数据。如果我们想完全隔离测试，我们可以避免使用类固定装置，并在测试类的构造函数中初始化一个新的 `InMemoryApplicationFactory` 实例：

[PRE11]

这种方法还保证了测试类中每个测试方法之间的隔离。此外，构造函数将在每次调用时提供一个新实例。

接下来，让我们看看如何使用 xUnit 数据属性来加载测试数据。

# 使用 xUnit 数据属性加载测试数据

*xUnit* 框架是测试 .NET 应用程序和服务的首选选择。该框架还提供了一些工具来扩展其功能并实现更易于维护的测试代码。可以扩展 `xUnit.Sdk` 命名空间公开的 `DataAttribute` 类以在属性内部执行自定义操作。例如，假设我们创建了一个新的自定义 `DataAttribute` 来从文件中加载测试数据，如下所示：

[PRE12]

在这种情况下，实现使用 `LoadData` 属性装饰测试方法，该属性从文件中读取一个 `item` 部分。因此，我们将有一个包含所有测试记录的 JSON 文件，我们将使用 `LoadData` 属性加载其中之一。为了为 `ItemControllerTests` 类自定义行为，我们应该创建一个新的类并扩展由 xUnit 提供的 `DataAttribute` 类：

[PRE13]

`LoadDataAttribute` 类重写了由 `DataAttribute` 类提供的 `GetData(MethodInfo testMethod);` 方法，并返回测试方法使用的数据。`GetData` 方法的实现读取由 `_filePath` 属性定义的文件内容；它尝试将文件指定的 `section` 的内容序列化为一个泛型 `object`。最后，实现调用 `ToObject` 方法将泛型 `JObject` 转换为与测试方法第一个参数关联的类型。该过程最后一步是在 `Catalog.API.Tests` 项目中创建一个新的名为 `record-data.json` 的 JSON 文件。该文件将包含我们测试使用的测试数据：

[PRE14]

JSON 片段包含以下字段：`item`、`artist` 和 `genre`。这些字段包含与测试实体相关的数据。因此，我们将使用它们将数据反序列化到请求模型和实体类型中。因此，我们可以将 `LoadData` 属性应用于以下方式的 `ItemControllerTests` 类：

[PRE15]

现在，测试方法接受一个 `Item`、`EditItemRequest` 或 `AddItemRequest` 类型的 `request` 参数，该参数将包含由 `record-data.json` 文件提供的数据。然后，该对象被序列化为 `request` 参数，并通过 `InMemoryApplicationFactory` 提供的 `HttpClient` 实例发送：

[PRE16]

`LoadData` 将 `record-data.json` 文件中定义的内容序列化为 `AddItemRequest` 类型。然后，请求被序列化为 `StringContent` 并通过工厂创建的 HTTP 客户端发送。最后，该方法断言结果代码是成功的，并且 `Location` 标头不是 `null`。

我们现在可以通过在解决方案根目录中执行 `dotnet test` 命令，或者通过运行我们首选 IDE 提供的测试运行器来验证 `ItemController` 类的行为。

总结来说，现在我们能够在一个独特的中央JSON文件中定义测试数据。除此之外，我们还可以通过向JSON文件添加新的部分来添加尽可能多的数据。本节接下来的部分将专注于通过添加一些存在性检查和处理异常使用过滤器来提高API的弹性。

# 提高API的弹性

前面的章节展示了`ItemController`类的可能实现以及如何使用ASP.NET Core提供的工具对其进行测试。在本节中，我们将学习如何通过在`ItemController`公开的信息上执行一些*限制检查*来提高我们服务的弹性。此外，我们还将探讨如何展示验证错误以及如何分页返回的数据。本节将应用前几章中解释的概念到Web服务项目中。

# 存在性检查

让我们先实现一个执行请求数据存在性检查的动作过滤器。该过滤器将被用于获取或编辑单个项的动作方法。如[第7章](13fd7d18-3ebe-4f60-89ff-4666d1c9671a.xhtml)中所述的*过滤器管道*，我们将实现以下过滤器：

[PRE17]

动作过滤器在构造函数中解析`IItemService`接口，并使用注入的实例通过请求中存在的`id`验证实体的存在。如果请求包含有效的`Guid id`，并且`id`存在于我们的数据源中，`OnActionExecutionAsync`方法将通过调用`await next()`方法继续管道。否则，它将停止管道并返回一个`NotFoundObjectResult`实例。我们可以通过添加`[ItemExists]`属性来将过滤器应用于`ItemController`的动作方法：

[PRE18]

在应用了`ItemExists`属性后，如果请求发送的ID不存在，API将返回404。我们还可以通过注入`IItemService`接口的模拟实例并对结果响应进行一些断言来验证在动作过滤器中实现的逻辑。在下面的测试类中，我们将使用`Moq`来验证对`next()`方法的调用。作为第一步，我们需要在项目文件夹内使用以下命令将`Moq`添加到`Catalog.API.Tests`项目中：

[PRE19]

此外，我们可以继续定义`ItemExistsAttributeTests`类：

[PRE20]

在前面的`ItemExistsAttributeTests`类中，我们模拟了整个`IItemService`接口以模拟`GetItemAsync`方法的响应。然后，它通过注入模拟的`IItemService`接口初始化`ItemExistsAttribute`。最后，它调用由`filter`类公开的`OnActionExecutionAsync`方法，并将其与`Moq`框架提供的`Verify`方法结合使用，以检查`ItemExistsFilter`类是否正确调用了`next()`回调方法。

# JSON自定义错误

异常的自定义和序列化是简化*错误处理*和改进Web服务监控的有用方法*.* 这些技术有时对于将异常传达给客户端以处理和管理错误是必要的。一般来说，虽然*HTTP状态码*提供了关于请求状态的摘要信息，但响应内容提供了关于错误的更详细信息。

使用过滤器扩展错误处理行为是可能的。首先，让我们创建一个新的标准模型来表示错误结果：

[PRE21]

上述类位于`Filters`文件夹结构下。它包含一个`EventId`属性和一个`DetailedMessage`，类型为`object`。其次，我们应该继续实现一个新的过滤器，该过滤器扩展了`IExceptionFilter`接口。当引发异常时，该过滤器将被触发，并将修改返回给客户端的响应内容：

[PRE22]

上述代码实现了`IExceptionFilter`接口。该类包含用于注入过滤器依赖项的一些构造函数的定义。它还包含`OnException`方法，该方法初始化一个新的`JsonErrorPayload`对象，该对象包含`eventId`字段和异常中包含的消息内容。根据环境，查看`IsDevelopment()`检查；它还向结果异常对象填充详细的错误消息。最后，`OnException`方法使用定义为参数的`HttpContext`来设置`HttpStatusCode.InternalServerError`，并添加之前创建的`exceptionObject`作为执行结果。这种方法保证了以独特的方式处理异常，通过集中序列化和所有由Web服务返回的错误的结果消息格式。

# 实现分页

分页是API的另一个基本功能。`Get`操作通常返回大量记录和信息。有时，实现分页以避免大响应大小是必要的。

如果您的API暴露给外部客户端，在可能的情况下减少响应大小是至关重要的。此外，客户端可能将信息存储在具有有限内存的设备内存中，例如智能手机或物联网设备。

让我们看看如何在`ItemController`类中实现可维护和可重用的分页。首先，我们需要创建一个新的分页响应模型来表示请求的页面。我们可以在`Catalog.API`项目中的`ResponseModels`文件夹内创建一个新的`PaginatedItemResponseModel.cs`文件，内容如下：

[PRE23]

`PaginatedItemsResponseModel`函数接受一个泛型模型，它表示分页响应类型。它还实现了与页面相关的某些属性，例如`PageIndex`、`PageSize`和`Total`。此外，它包括一个`IEnumerable`接口，表示响应返回的记录。下一步是更改`ItemController`类中已经存在的`Get`操作方法，如下所示：

[PRE24]

我们将`Get`操作方法更改为实现分页。首先，请注意，该方法接收一些与分页相关的参数：`pageSize`和`pageIndex`参数。其次，它执行`IItemService`以获取相关记录并执行LINQ查询以仅获取所选页面的元素。最后，它使用与页面和数据相关的元数据实例化一个新的`PaginatedItemsResponseModel<ItemResponse>`，并返回该实例。

我们可以通过更改现有的`ItemsControllerTests`文件来使用单元测试来覆盖实现：

[PRE25]

`should_get_item_using_pagination`测试用例使用`InlineData`属性来测试一些分页路由。它调用`Get`操作方法，将结果序列化为`PaginatedItemsResponseModel<ItemResponse>`，最后检查结果。

尽管本章中检查的分页实现技术提供了一些性能优势，但它并没有限制我们的服务与数据库之间的交互。为了将性能优势扩展到服务的数据源，我们应该考虑实现一个专门查询，直接从我们的数据源分页数据。

在下一节中，我们将继续我们的旅程，通过扩展API来处理相关实体。目前，请注意，我们正在暴露`Item`、`Artist`和`Genre`实体的信息，而不管理相关实体，并且我们没有暴露任何编辑`Artist`和`Genre`实体的路由。

# 暴露相关实体

目前，`Catalog.API`项目中的当前实现允许我们读取和修改`Item`实体及其与`Genre`和`Artist`实体的关系。在本节中，我们将使客户端能够列出和添加`Genre`和`Artist`实体。因此，我们将扩展允许客户端与这些实体交互的API。此实现要求我们对整个网络服务栈采取行动；此外，它还涉及`Catalog.Infrastructure`、`Catalog.Domain`和`Catalog.API`项目。在我们开始之前，让我们看看我们将要实现的路由：

| **动词** | **路径** | **描述** |
| --- | --- | --- |
| `GET` | `/api/artists` | 获取数据库中所有存在的艺术家 |
| `GET` | `/api/artist/{id}` | 获取具有相应ID的艺术家 |
| `GET` | `/api/artist/{id}/items/` | 获取与相应艺术家ID对应的物品 |
| `POST` | `/api/artist/` | 创建一个新的艺术家并检索它 |

同样，我们将为`Genre`实体获取相应的路由：

| **动词** | **路径** | **描述** |
| --- | --- | --- |
| `GET` | `/api/genre` | 获取数据库中存在的所有流派 |
| `GET` | `/api/genre/{id}` | 获取具有相应ID的流派 |
| `GET` | `/api/genre/{id}/items/` | 获取具有相应流派ID的项目 |
| `POST` | `/api/genre/` | 创建一个新的流派并获取它 |

在前表中提到的路由提供了一种与`Genre`和`Artist`实体交互的方式。这些功能的实现将遵循我们在前几节中看到的项实体所使用的相同方法。在继续之前，我们需要通过添加代表`Artist`和`Genre`实体的属性来扩展`CatalogContext`类：

[PRE26]

现在`CatalogContext`也通过使用`modelBuilder.ApplyConfiguration`方法处理`Artists`和`Genres`实体。在下一个小节中，我们将通过使用`Repositories`类扩展数据访问层的实现。

# 扩展数据访问层

为了扩展我们的API以包含相关实体，我们应该从我们的堆栈底部开始。首先，让我们通过在`Catalog.Domain`项目的`Repositories`文件夹中添加以下接口来扩展数据访问层的功能：

[PRE27]

`IArtistRepository`和`IGenreRepository`接口反映了本节最初定义的路由：`GetAsync`方法返回次要实体的列表，`GetAsync(Guid id)`返回单个对象，而`Add`方法允许我们创建新实体。我们现在可以定义指定接口的实际实现。同样，`ItemRepository`类的实现将存储在`Catalog.Infrastructure`项目中：

[PRE28]

上述代码定义了`ArtistRepository`类型，并为`IArtistRepository`接口提供了实现。该类使用`CatalogContext`作为我们的应用程序和SQL数据库之间的通信中心。`GetAsync`和`GetAsync(Guid id)`方法使用已在`ItemRepository`类中实现的相同模式来检索与所需实体相关的信息。此外，`Add`方法引用了`CatalogContext`公开的`Artists`字段以添加新艺术家。需要注意的是，在这种情况下，`Add`操作并不直接更新数据源。

让我们通过定义`GenreRepository`类来继续：

[PRE29]

与 `ArtistRepository` 类似，我们正在实现查询和操作 `Genre` 实体的操作。尽管方法和实现方式非常相似，但我选择保持存储库接口分离，并分别重新定义每个实现。一个更快的方法是创建一个代表 `ItemRepository`、`ArtistRepository` 和 `GenreRepository` 典型行为的通用类，但通用存储库并不总是最佳选择。此外，构建错误抽象的成本远高于代码重复，为所有内容构建唯一的通用存储库意味着实体之间紧密耦合。

# 扩展测试覆盖率

正如我们在 `ItemRepositoryTests` 类中所做的那样，我们可以通过使用相同的方法来测试 `ArtistRepository` 和 `GenreRepository`。在 [第 8 章](84b281bd-11a2-4703-81ae-ca080e2a267a.xhtml) “构建数据访问层” 中，我们定义了 `TestDataContextFactory`，它是 `Catalog.Fixtures` 项目的一部分。我们可以使用它来为测试目的实例化一个内存数据库：

[PRE30]

之前的代码探索了实现 `ArtistRepository` 类测试的方法。`ArtistRepositoryTests` 类扩展了 `IClassFixture<CatalogContextFactory>` 以检索 `CatalogContextFactory` 类型的实例。测试方法使用 `ContextInstance` 属性来检索一个新的目录上下文并初始化一个新的存储库。

他们通过执行方法作为测试并检查结果来继续。重要的是要注意，每个测试方法都使用 `LoadData` 属性来加载 `record-data.json` 文件的 `artist` 部分。为了简洁，我省略了一些测试用例；然而，它们背后的概念与我们之前看到的相同，并且可以扩展到 `GenreRepository` 测试。

# 扩展 IItemRepository 接口

我们可以采取的另一步是扩展我们的 Web 服务项目，以包含相关的实体，即在 `IItemRepository` 接口中实现两个新的方法来检索与艺术家或流派相关的项目：`GetItemsByArtistIdAsync` 和 `GetItemsByGenreIdAsync` 方法。这两个方法都可以由 `GET /api/artists/{id}/items` 和 `GET /api/genre/{id}/items` 路由使用来检索项目。

让我们通过向 `IItemsRepository` 接口添加以下方法并在相应的实现中实现它们来继续：

[PRE31]

此代码扩展了 `IItemRepository` 接口及其实现，以便包括使用 `ArtistId` 和 `GenreId` 查询项目的功能。这两个方法都使用 `Where` 子句和调用 `Include` 语句来包含查询结果中的相关实体。一旦我们完全扩展了存储库层，我们还可以继续扩展服务类。

# 扩展服务层的功能

新的`Catalog.Infrastructure`功能扩展了`Catalog.Domain`项目，并将`IArtistRepository`和`IGenreRepository`暴露给API项目的控制器。首先，我们应该在`Catalog.Domain`项目中创建几个新的服务类，以便查询底层的`Catalog.Infrastructure`层。让我们从在`Catalog.Domain`项目的`Services`文件夹中定义`IArtistService`和`IGenreService`接口开始：

[PRE32]

上述代码片段包含了`IArtistService`和`IGenreService`接口的声明。为了简洁，我将它们保留在同一代码片段中。两个接口都定义了列表中的方法，获取详细信息，然后添加相关实体。`GetArtistsAsync()`和`GetGenreAsync()`方法可以根据是否指定`request`参数返回完整的实体列表或单个实体。此外，还可以使用`GetItemByArtistIdAsync`和`GetItemByGenreIdAsync`方法通过艺术家ID或流派ID检索`ItemResponse`列表。最后，我们可以使用`AddArtistAsync`和`AddGenreAsync`方法添加新的艺术家和流派。

上述实现还依赖于以下请求模型的定义：

[PRE33]

在这里，请求类定义了`Artist`和`Genre`实体的允许操作。它们分别存储在`Requests/Artist`和`Requests/Genre`文件夹中。我们可以继续实现`ArtistService`类的具体部分，如下所示：

[PRE34]

上述代码定义了`ArtistService`类的属性和构造函数。实现注入了`IArtistRepository`、`IItemRepository`、`IArtistMapper`和`IItemMapper`依赖项。`Repositories`类将被用于与应用程序底层数据源通信。另一方面，调用映射器来初始化和映射作为响应发送的值。

一旦我们定义了`ArtistService`类的依赖关系，我们就可以继续实现核心方法：

[PRE35]

实现表示在`IArtistService`接口中已定义的方法。通过查看

它们的签名。

`GetAsync`方法调用`IArtistRepository`依赖项来映射和检索结果，在`ArtistResponse`对象中。`GetItemByArtistIdAsync`执行在`IItemRepository`接口中定义的同名方法。最后，`AddArtistAsync`执行`IArtistRepository`接口中定义的`Add`方法，并执行`UnitOfWork.SaveChangesAsync`方法将更改应用到数据源，然后映射结果数据。

`GenreService`实现类也可以采用相同的方法，它将依赖于`IGenreRepository`和`IGenreMapper`接口。

你可以在本书的官方GitHub仓库中找到`GenreService`类的实现：[https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3](https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3)。

最后，我们还应该记得将之前章节中定义的`AddServices`扩展方法中的这些接口定义包括在内，通过以下更改：

[PRE36]

这些更改注册了`IArtistService`和`IGenreService`接口，以便它们可以被控制器和其他应用程序依赖项使用。在下一节中，我们将继续实现，通过向请求模型添加一些验证逻辑来完成。

# 改进验证机制

正如我们在上一章中解释的，我们正在使用`FluentValidation`包来实现Web服务的验证机制。由于我们已经构建了服务接口来处理`Artist`和`Genre`实体，现在我们可以改进`AddItemRequestValidator`和`EditItemRequestValidator`类中现有的验证检查。现在，我们将实现`Artist`和`Genre`相关实体的存在检查。

让我们从扩展`AddItemRequestValidator`类的实现开始：

[PRE37]

现在，`AddItemRequestValidator`类使用构造函数注入模式注入了`IArtistService`和`IGenreService`接口。除此之外，验证类还定义了两个方法，`ArtistExists`和`GenreExists`，这些方法将通过调用`IArtistService`和`IGenreService`接口方法来验证`ArtistId`和`GenreId`字段是否存在于数据库中。此外，我们可以通过以下方式改进验证规则：

[PRE38]

新的规则将`ArtistId`和`GenreId`字段分别绑定到`ArtistExists`和`GenreExists`方法。同样的方法也可以用于`EditItemRequestValidator`的实现，它将使用相同的模式来验证相关实体。因此，我们现在需要扩展测试类以验证新的验证规则：

[PRE39]

上述代码将`Mock<IArtistService>`和`Mock<IGenreService>`类型注入到验证类构造函数中，以模拟服务层的操作并验证验证类的逻辑。它还使用`ShouldHaveValidationErrorFor`方法来检查预期的响应。如果相关实体的ID缺失于数据源，`ArtistExists`和`GenreExists`方法应该抛出验证错误。

# 更新`Startup`类中的依赖项

在前面的几个部分中，我们创建了 `IArtistService` 和 `IGenreService` 接口及其相应的实现。因此，我们想要更新应用程序的依赖图。尽管我们已经在 `ConfigureService` 方法中调用了 `DependenciesRegistration` 静态类的 `AddMappers` 和 `AddServices` 扩展方法，但我们需要以下方式更新依赖项：

[PRE40]

我们还应该在 `ConfigureService` 方法中添加 `IArtistRepository` 和 `IGenreRepository`：

[PRE41]

现在，`Startup` 类的 `ConfigureServices` 方法定义了我们堆栈中所需的所有依赖项。我们目前能够解析与 `ArtistController` 和 `GenreController` 类相关的依赖链，这些类我们将在下一节中定义。

# 添加相关控制器

现在，`Catalog.Domain` 项目能够通过我们在 `IArtistService` 和 `IGenreService` 类中实现的逻辑来处理与 `Artist` 和 `Genre` 实体相关的请求。因此，我们可以通过创建控制器层来处理传入的 HTTP 请求。由于我们有不同的独立实体，我们将实现 `ArtistController` 和 `GenreController` 控制器类。让我们首先关注 `ArtistController`：

[PRE42]

上述代码定义了 `ArtistController` 类的初始签名：实现注入了来自服务层的 `IArtistService` 接口，以便与存储在数据库中的信息进行交互。我们可以进一步定义 `Get` 和 `GetById` 动作方法的实现：

[PRE43]

前面的代码展示了 `ArtistController` 的实现，它对应于 `/api/artist` 路由。`ArtistController` 使用 `PaginatedItemsResponseModel` 来检索所有艺术家的相关信息。同样地，我们可以使用 `IAritstService` 接口来执行其他操作，并将它们映射到动作方法：

[PRE44]

前面的 `ArtistController` 类定义了以下路由：

| **动词** | **路径** | **描述** |
| --- | --- | --- |
| `GET` | `/api/artist` | 检索数据库中存在的所有艺术家 |
| `GET` | `/api/artist/{id}` | 获取具有相应 *ID* 的艺术家 |
| `GET` | `/api/artist/{id}/items/` | 获取具有相应艺术家 *ID* 的项目 |
| `POST` | `/api/artist/` | 创建一个新的艺术家并检索它 |

对于每个路由，*目录服务* 将通过执行 `IArtistService` 接口方法来检索相应的数据。`ArtistService` 的实现将请求调度到包含在 `Catalog.Infrastructure` 项目中的存储库实现对应的相应方法。类似于 `ItemController`，我们可以通过使用 `FluentValidation` 来实现任何额外的验证行为。例如，在 `AddArtistRequest` 的情况下，我们可以进行以下实现：

[PRE45]

在`AddArtistRequest`的情况下，我们希望防止用户添加空的`ArtistName`字段。我们可以在`Requests/Artist/Validator`路径下创建一个额外的`AddArtistRequestValidator`类。验证类只包含一个与非空`ArtistName`字段相关的规则。

同样的实现模式也可以应用于`GenreController`类。以下代码定义了类实现的签名：

[PRE46]

上述代码使用*构造函数注入*实践注入了`IGenreService`接口。此外，我们可以继续创建为`ArtistController`类定义的相应路由：

[PRE47]

上述代码在`GenreController`类中定义了`Get`、`GetById`、`GetItemById`和`Post`操作方法。每个操作都调用底层服务层以与数据库交互。尽管操作方法的实现与`ArtistController`类相似，但我仍然保持类分开；因此，`Genre`和`Artist`实体可以独立演变，而不相互依赖。

# 扩展`ArtistController`类的测试

一旦我们通过`ArtistController`和`GenreController`类公开了API路由，我们就可以使用与`ItemController`相同的方法来测试这些公开的路径。我们应该继续在`Catalog.API.Tests`项目中创建一个新的`ArtistControllerTests`类：

[PRE48]

上述代码描述了获取分页艺术家操作方法和按ID获取操作方法的测试用例。两者都使用相同的`InMemoryApplicationFactory`工厂来设置模拟的内存数据源。也可以结合额外的测试来验证添加艺术家过程：*

[PRE49]

固定类为每个测试方法初始化一个新的`TestServer`实例，并通过`CreateClient`方法提供一个新的`HttpClient`实例来测试`ArtistController`类公开的路由。测试还使用`LoadData`属性从`record-data.json`文件中获取有关测试记录的匹配信息。可以采用类似的实现方法来测试`GenreController`类的操作方法。

# 最终概述

最后，让我们快速概述一下我们在[第8章](84b281bd-11a2-4703-81ae-ca080e2a267a.xhtml)，*构建数据访问层*，[第9章](f8ac60e1-e948-4435-b804-e3e4ff305510.xhtml)，*实现领域逻辑*和本章中实现的代码结构。以下架构图展示了`Catalog.API`解决方案的实际构建：

![图片](img/ef6629e2-50a2-4aa8-8e6a-a66028bcd14a.png)

一旦 Web 服务实例接收到客户端请求，它将请求调度到控制器的相应操作方法。随后，操作方法执行由 `Catalog.Domain` 项目中定义的服务类公开的相应功能，将请求发送到 `Catalog.Infrastructure` 项目中定义的相关存储库，然后映射实体与请求和响应模型。

# 摘要

本章涵盖了关于 `Catalog.API` 项目设计和开发的各个方面。此外，它还展示了如何构建我们的 API 路由。本章涵盖的主题是我们 Web 服务核心实现的一部分；因此，它们提供了在 .NET Core 中公开 HTTP 路由和处理请求和响应所需的所有知识。除此之外，本章还涵盖了使用框架提供的 Web 工厂工具进行的集成测试和实现。

在下一章中，我们将介绍关于 Web 服务的一些其他附加主题，例如如何实现数据软删除方法以及如何使用 **Hypermedia As The Engine Of Application State** （**HATEOAS**） 方法。本章还将涵盖一些与 .NET Core 相关的主题，例如对异步编程的简要介绍以及在 .NET Core 中的最佳实践。
