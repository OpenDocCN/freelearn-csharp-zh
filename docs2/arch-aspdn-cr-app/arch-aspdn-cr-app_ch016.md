# 第十五章：15 个对象映射器、聚合服务和外观

## 在开始之前：加入我们的 Discord 书籍社区

直接向作者本人提供反馈，并在我们的 Discord 服务器上与其他早期读者聊天（在“architecting-aspnet-core-apps-3e”频道下找到“EARLY ACCESS SUBSCRIPTION”）。

[`packt.link/EarlyAccess`](https://packt.link/EarlyAccess)

![二维码描述自动生成](img/file93.png)

在本章中，我们探讨对象映射。正如我们在上一章中看到的，与层一起工作通常会导致从一层复制模型到另一层。对象映射器解决了这个问题。我们首先手动实现一个对象映射器。然后，通过重新组合映射器到映射服务中，探索聚合服务模式和映射外观模式，我们改进了我们的设计。最后，我们用两个开源工具替换了这项手动工作，这些工具帮助我们生成业务价值而不是编写映射代码。在本章中，我们涵盖了以下主题：

+   对象映射和对象映射器的概述

+   实现一个简单的对象映射器

+   探索过多的依赖代码异味

+   探索聚合服务模式

+   通过利用外观模式实现映射外观

+   使用服务定位器模式在映射器前面创建一个灵活的映射服务

+   使用 AutoMapper 将对象映射到另一个对象，替换我们自制的代码

+   使用 Mapperly 而不是 AutoMapper

## 对象映射器

什么是对象映射？简单来说，它是将一个对象属性的值复制到另一个对象的属性中的动作。但有时，属性名称不匹配；可能需要将对象层次结构扁平化并转换。正如我们在上一章中看到的，每一层都可以拥有自己的模型，这可能是好事，但这也需要以从一层到另一层复制对象为代价。我们也可以在层之间共享模型，但即使这样，我们通常也需要将一个对象映射到另一个对象。即使只是将你的模型映射到**数据传输对象**（**DTOs**），这也是不可避免的。

> 记住 DTOs 定义了我们的 API 合同。独立的合同类有助于维护系统，让我们选择何时修改它们。如果你跳过这部分，每次你更改模型时，它都会自动更新你的端点合同，可能会破坏一些客户端。此外，如果你直接输入模型，恶意用户可能会尝试绑定他们不应该绑定的属性值，导致潜在的安全问题（称为**过度提交**或**过度提交攻击**）。拥有良好的数据交换合同是设计健壮系统的一个关键。

在以前的项目中，映射逻辑是硬编码的，有时是重复的，并为执行映射的类添加了额外的职责。在本章中，我们将映射逻辑提取到对象映射器中，以解决这个问题。

### 目标

对象映射器的目标是复制一个对象属性值到另一个对象的属性中。它将映射逻辑封装在映射发生的地方之外。映射器还负责在两个对象不遵循相同结构时，将值从原始格式转换为目标格式。

### 设计

我们可以以多种方式设计对象映射器。以下是最基本的对象映射器设计：

![图 15.1：对象映射器的基本设计](img/file94.png)

图 15.1：对象映射器的基本设计

在图中，**消费者**使用`IMapper`接口将`Type1`类型的对象映射到`Type2`类型的对象。这并不是非常可重用，但它说明了概念。通过使用**泛型**的力量，我们可以将这个简单的设计升级到更可重用的版本：

![图 15.2：泛型对象映射器设计](img/file95.png)

图 15.2：泛型对象映射器设计

此设计允许我们通过为每个映射规则实现一次`IMapper<TSource, TDestination>`接口，将任何`TSource`映射到任何`TDestination`。一个类也可以实现多个映射规则。例如，我们可以在同一个类中实现`Type1`到`Type2`和`Type2`到`Type1`的映射（双向映射器）。另一种方式是使用以下设计，创建一个具有单个方法处理应用程序所有映射的`IMapper`接口：

![图 15.3：使用单个 IMapper 作为入口点的对象映射](img/file96.png)

图 15.3：使用单个 IMapper 作为入口点的对象映射

此最后设计的最大优点是易于使用。我们总是注入一个单一的`IMapper`，而不是为每种映射类型注入一个`IMapper<TSource, TDestination>`，这减少了依赖项的数量和消费此类映射器的复杂性。你可以以任何你想象的方式实现对象映射，但关键部分是映射器负责将一个对象映射到另一个对象。映射器应避免复杂的过程，例如从数据库加载数据等。它应该将一个对象的值复制到另一个对象中：仅此而已。考虑一下**单一职责原则**（**SRP**）：类必须有单一的理由进行更改，并且由于它是一个对象映射器，这个理由应该是对象映射。让我们通过一些代码深入了解每个项目的设计。

### 项目 – 映射器

此项目是前一章中 Clean Architecture 代码的更新版本。该项目旨在展示将实体映射逻辑封装到映射器类中的设计灵活性，将此逻辑从消费者那里移开。当然，项目再次专注于当前用例，使学习这些主题变得更容易。首先，我们需要一个接口，它位于`Core`项目中，以便其他项目可以实施它们所需的映射。让我们采用我们看到的第二个设计：

```cs
namespace Core.Mappers;
public interface IMapper<TSource, TDestination>
{
    TDestination Map(TSource entity);
}
```

使用这个接口，我们可以先创建数据映射器。但首先，让我们先创建记录类而不是匿名类型来命名端点返回的 DTO。以下是所有 DTO（来自 `Program.cs` 文件）：

```cs
// Input stock DTOs
public record class AddStocksCommand(int Amount);
public record class RemoveStocksCommand(int Amount);
// Output stock DTO
public record class StockLevel(int QuantityInStock);
// Output "read all products" DTO
public record class ProductDetails(int Id, string Name, int QuantityInStock);
// Output Exceptions DTO
public record class ProductNotFound(int ProductId, string Message);
public record class NotEnoughStock(int AmountToRemove, int QuantityInStock, string Message);
```

四个输出 DTO 中的三个需要映射：

+   `Product` 到 `ProductDetails`

+   `ProductNotFoundException` 到 `ProductNotFound`

+   `NotEnoughStockException` 到 `NotEnoughStock`

> 为什么不映射 `StockLevel` DTO？在我们的案例中，当我们添加或移除库存时，`StockService` 返回一个 `int`，因此将原始值如 `int` 转换为 `StockLevel` 对象不需要对象映射器。此外，创建这样的对象映射器没有任何价值，并且会使代码更复杂。如果服务返回了一个对象，创建一个将对象映射到 `StockLevel` 的映射器将更有意义。

让我们从产品映射器开始（来自 `Program.cs` 文件）：

```cs
public class ProductMapper : IMapper<Product, ProductDetails>
{
    public ProductDetails Map(Product entity)
        => new(entity.Id ?? default, entity.Name, entity.QuantityInStock);
}
```

上述代码很简单；`ProductMapper` 类实现了 `IMapper<Product, ProductDetails>` 接口。`Map` 方法返回一个 `ProductDetails` 实例。高亮显示的代码确保 `Id` 属性不是 `null`，这不应该发生。我们也可以添加一个保护子句来确保 `Id` 属性不是 `null`。总的来说，`Map` 方法接收一个 `Product` 作为输入，并输出一个包含相同值的 `ProductDetails` 实例。然后让我们继续处理异常映射器（来自 `Program.cs` 文件）：

```cs
public class ExceptionsMapper : IMapper<ProductNotFoundException, ProductNotFound>, IMapper<NotEnoughStockException, NotEnoughStock>
{
    public ProductNotFound Map(ProductNotFoundException exception)
        => new(exception.ProductId, exception.Message);
    public NotEnoughStock Map(NotEnoughStockException exception)
        => new(exception.AmountToRemove, exception.QuantityInStock, exception.Message);
}
```

与 `ProductMapper` 类相比，`ExceptionsMapper` 类通过两次实现 `IMapper` 接口来处理剩余的两个用例。两个 `Map` 方法处理将异常映射到其 DTO，导致一个类负责将异常映射到 DTO。让我们看看 `products` 端点（来自 *第十四章*，*分层和整洁架构*的 `clean-architecture` 项目的原始值）：

```cs
app.MapGet("/products", async (
    IProductRepository productRepository, 
    CancellationToken cancellationToken) =>
{
    var products = await productRepository.AllAsync(cancellationToken);
    return products.Select(p => new
    {
        p.Id,
        p.Name,
        p.QuantityInStock
    });
});
```

在分析代码之前，让我们看看更新后的版本（来自 `Program.cs` 文件）：

```cs
app.MapGet("/products", async (
    IProductRepository productRepository, 
    IMapper<Product, ProductDetails> mapper, 
    CancellationToken cancellationToken) =>
{
    var products = await productRepository.AllAsync(cancellationToken);
    return products.Select(p => mapper.Map(p));
});
```

在前面的代码中，请求委托使用映射器替换了复制逻辑（原始代码中的高亮行）。这简化了处理器，将映射责任移动到映射对象中（前面代码中的高亮显示）——这是向 SRP（单一职责原则）迈出的又一步。让我们跳过添加库存端点，因为它与移除库存端点非常相似，但更简单，让我们关注后面的（来自 *第十四章*，*分层和整洁架构*的 `clean-architecture` 项目的原始值）：

```cs
app.MapPost("/products/{productId:int}/remove-stocks", async (
    int productId, 
    RemoveStocksCommand command, 
    StockService stockService, 
    CancellationToken cancellationToken) =>
{
    try
    {
        var quantityInStock = await stockService.RemoveStockAsync(productId, command.Amount, cancellationToken);
        var stockLevel = new StockLevel(quantityInStock);
        return Results.Ok(stockLevel);
    }
    catch (NotEnoughStockException ex)
    {
        return Results.Conflict(new
        {
            ex.Message,
            ex.AmountToRemove,
            ex.QuantityInStock
        });
    }
    catch (ProductNotFoundException ex)
    {
        return Results.NotFound(new
        {
            ex.Message,
            productId,
        });
    }
});
```

再次，在分析代码之前，让我们看看更新后的版本（来自 `Program.cs` 文件）：

```cs
app.MapPost("/products/{productId:int}/remove-stocks", async (
    int productId, 
    RemoveStocksCommand command, 
    StockService stockService, 
    IMapper<ProductNotFoundException, ProductNotFound> notFoundMapper, 
    IMapper<NotEnoughStockException, NotEnoughStock> notEnoughStockMapper, 
    CancellationToken cancellationToken) =>
{
    try
    {
        var quantityInStock = await stockService.RemoveStockAsync(productId, command.Amount, cancellationToken);
        var stockLevel = new StockLevel(quantityInStock);
        return Results.Ok(stockLevel);
    }
    catch (NotEnoughStockException ex)
    {
        return Results.Conflict(notEnoughStockMapper.Map(ex));
    }
    catch (ProductNotFoundException ex)
    {
        return Results.NotFound(notFoundMapper.Map(ex));
    }
});
```

对于这个请求代理也发生了同样的事情，但我们注入了两个映射器而不是一个。我们将映射逻辑从使用匿名类型内联移动到映射器对象。尽管如此，这里出现了一个代码异味；你能闻到吗？我们将在完成这个项目后进行调查；同时，继续思考注入依赖项的数量。现在，由于代理依赖于封装映射责任的映射器接口，我们必须配置组合根并将映射器实现绑定到`IMapper<TSource, TDestination>`接口。服务绑定看起来像这样：

```cs
.AddSingleton<IMapper<Product, ProductDetails>, ProductMapper>()
.AddSingleton<IMapper<ProductNotFoundException, ProductNotFound>, ExceptionsMapper>()
.AddSingleton<IMapper<NotEnoughStockException, NotEnoughStock>, ExceptionsMapper>()
```

由于`ExceptionsMapper`实现了两个接口，我们将它们都绑定到该类上。这是抽象之美之一；移除股票的代理请求需要两个映射器，但即使不知道，它也两次接收了`ExceptionsMapper`的实例。我们也可以注册这些类，以便相同的实例被注入两次，如下所示：

```cs
.AddSingleton<ExceptionsMapper>()
.AddSingleton<IMapper<ProductNotFoundException, ProductNotFound>, ExceptionsMapper>(sp => sp.GetRequiredService<ExceptionsMapper>())
.AddSingleton<IMapper<NotEnoughStockException, NotEnoughStock>, ExceptionsMapper>(sp => sp.GetRequiredService<ExceptionsMapper>())
```

> 是的，我故意进行了相同类的双重注册。这证明了我们可以按照我们的意愿组合应用程序，而不会影响**消费者**。这是通过依赖于抽象而不是实现来实现的，正如**依赖倒置原则**（**DIP**——SOLID 中的“D”）所要求的。此外，根据**接口隔离原则**（**ISP**——SOLID 中的“I”），将接口划分为小块，使得这种场景成为可能。最后，我们可以利用**依赖注入**（**DI**）的力量将这些碎片粘合在一起。

### 结论

在探索更多替代方案之前，让我们看看对象映射如何帮助我们遵循**SOLID**原则：

+   **S**：使用映射器对象有助于将映射类型的责任从消费者中分离出来，使得维护和重构映射逻辑更容易。

+   **O**：通过注入映射器，我们可以更改映射逻辑，而无需更改消费者代码。

+   **L**：N/A

+   **我**：我们探索了不同的设计，这些设计提供了一个小的映射器接口，减少了组件之间的依赖性。

+   **D**：消费者只依赖于抽象，将实现的绑定移动到组合根，并反转了依赖流。

现在我们已经探讨了如何提取和使用映射器，让我们看看出现的代码异味。

## 代码异味——过多的依赖

长期来看，使用这种映射可能会变得繁琐，我们很快就会看到将三个或更多映射器注入单个请求代理或控制器的情况。消费者可能已经拥有其他依赖项，导致依赖项增加到四个或更多。这应该会引发以下警示：

+   这个类是否做得太多，承担了太多的责任？

在这种情况下，细粒度的`IMapper`接口使我们的请求代理充满了对映射器的依赖，这并不理想，也使得我们的代码更难阅读。

> 最佳解决方案是将异常处理责任从代理或控制器中移除，利用中间件或异常过滤器等。这种策略会将样板代码从端点移除。
> 
> > 由于我们正在讨论映射器，我们将探索更多对象映射概念，以帮助我们解决这个问题，这适用于许多用例。

作为一项经验法则，你希望**将依赖项的数量限制在三个或更少**。超过这个数量，问问自己这个类是否有问题；它是否承担了太多的责任？拥有超过三个依赖项本身并不是错误的；它只是表明你应该调查设计。如果没有问题，保持 4 个或 5 个，或者 10 个；这并不重要。如果你找到了减少依赖数量的方法，或者发现该类实际上做了太多事情，那么重构代码。如果你不喜欢有那么多依赖项，你可以提取封装两个或更多依赖项的服务聚合，并注入该聚合。请注意，移动你的依赖项并不能解决问题；如果最初存在问题，它只是将问题转移到别处。使用聚合可能会提高代码的可读性。

> 而不是盲目地移动依赖项，分析问题以查看是否可以创建具有实际逻辑的类，从而对减少依赖数量有所贡献。

接下来，让我们快速看一下聚合服务。

## 模式 - 聚合服务

即使聚合服务模式不是一个神奇的解决问题的模式，它也是向另一个类注入过多依赖项的一个可行的替代方案。其目标是聚合一个类中的许多依赖项，以减少其他类中注入的服务数量，将依赖项分组在一起。管理聚合的方式是将它们按关注点或责任分组。仅仅为了将一堆服务放入另一个服务中通常不是正确的做法；目标是内聚性。

> 创建一个或多个暴露其他服务的聚合服务可以是实现项目中的服务发现的一种方式。像往常一样，首先分析问题是否不在于其他地方。加载暴露其他服务的服务可能很有用。然而，这可能会产生问题，所以一开始也不要把所有东西都放入一个聚合中。

这里是一个将映射聚合用于减少**创建-读取-更新-删除**（**CRUD**）控制器依赖数量的示例，该控制器允许创建、更新、删除和读取一个、多个或所有产品。以下是聚合服务代码和使用示例：

```cs
public interface IProductMappers
{
    IMapper<Product, ProductDetails> EntityToDto { get; }
    IMapper<InsertProduct, Product> InsertDtoToEntity { get; }
    IMapper<UpdateProduct, Product> UpdateDtoToEntity { get; }
}
public class ProductMappers : IProductMappers
{
    public ProductMappers(IMapper<Product, ProductDetails> entityToDto, IMapper<InsertProduct, Product> insertDtoToEntity, IMapper<UpdateProduct, Product> updateDtoToEntity)
    {
        EntityToDto = entityToDto ?? throw new ArgumentNullException(nameof(entityToDto));
        InsertDtoToEntity = insertDtoToEntity ?? throw new ArgumentNullException(nameof(insertDtoToEntity));
        UpdateDtoToEntity = updateDtoToEntity ?? throw new ArgumentNullException(nameof(updateDtoToEntity));
    }
    public IMapper<Product, ProductDetails> EntityToDto { get; }
    public IMapper<InsertProduct, Product> InsertDtoToEntity { get; }
    public IMapper<UpdateProduct, Product> UpdateDtoToEntity { get; }
}
public class ProductsController : ControllerBase
{
    private readonly IProductMappers _mapper;
    // Constructor injection, other methods, routing attributes, ...
    public ProductDetails GetProductById(int id)
    {
        Product product = ...; // Fetch a product by id
        ProductDetails dto = _mapper.EntityToDto.Map(product);
        return dto;
    }
}
```

在这个例子中，`IProductMappers` 聚合可能是有逻辑性的，因为它将 `ProductsController` 类使用的映射器归入其旗下。它负责将 `ProductsController` 相关的领域对象映射到 DTO 并反之，而控制器放弃了这一责任。你可以用任何东西创建聚合服务，而不仅仅是映射器。这在依赖注入（DI）密集型应用中是一个相当常见的模式（这也可以指向一些设计缺陷）。现在我们已经探讨了聚合服务模式，让我们来看看如何制作一个映射外观。

## 模式 – 映射外观

我们已经研究了外观；在这里，我们通过利用该设计模式探索另一种组织我们众多映射器的方法。我们不会像刚才那样做，而是创建一个映射外观来替换我们的聚合服务。使用外观的代码将更加优雅，因为它直接使用 `Map` 方法而不是属性。外观的责任与聚合相同，但它实现接口而不是将其作为属性公开。让我们看看代码：

```cs
public interface IProductMapperService : 
    IMapper<Product, ProductDetails>, 
    IMapper<InsertProduct, Product>, 
    IMapper<UpdateProduct, Product>
{
}
public class ProductMapperService : IProductMapperService
{
    private readonly IMapper<Product, ProductDetails> _entityToDto;
    private readonly IMapper<InsertProduct, Product> _insertDtoToEntity;
    private readonly IMapper<UpdateProduct, Product> _updateDtoToEntity;
    // Omitted constructor injection code
    public ProductDetails Map(Product entity)
    {
        return _entityToDto.Map(entity);
    }
    public Product Map(InsertProduct dto)
    {
        return _insertDtoToEntity.Map(dto);
    }
    public Product Map(UpdateProduct dto)
    {
        return _updateDtoToEntity.Map(dto);
    }
}
```

在前面的代码中，`ProductMapperService` 类通过 `IProductMapperService` 接口实现 `IMapper` 接口，并将映射逻辑委托给每个注入的映射器：一个封装多个单个映射器的外观。接下来，我们看看消费外观的 `ProductsController`：

```cs
public class ProductsController : ControllerBase
{
    private readonly IProductMapperService _mapper;
    // Omitted constructor injection, other methods, routing attributes, ...
    public ProductDetails GetProductById(int id)
    {
        Product product = ...; // Fetch a product by id
        ProductDetails dto = _mapper.Map(product);
        return dto;
    }
}
```

从消费者角度来看（`ProductsController` 类），我发现写 `_mapper.Map(...)` 而不是 `_mapper.SomeMapper.Map(...)` 更干净。消费者不关心映射器在做什么映射；它只想映射需要映射的内容。如果我们将映射外观与前面例子中的聚合服务进行比较，外观承担了选择映射器的责任，并将其从消费者那里移开。这种设计更好地在类之间分配了责任。这是一个审查外观设计模式的绝佳机会。尽管如此，现在我们已经探讨了多种映射选项并检查了过多依赖的问题，是时候带着我们映射外观的增强版本继续我们的对象映射冒险了。

## 项目 – 映射服务

目标是简化具有通用接口的映射外观实现。为了实现这一点，我们正在实现如图 *图 13.3* 所示的图。这里有一个提醒：

![图 15.4：使用单个 IMapper 接口进行对象映射](img/file97.png)

图 15.4：使用单个 IMapper 接口进行对象映射

我们不会将接口命名为 `IMapper`，而是使用 `IMappingService` 这个名字。这个名字更合适，因为它不是映射任何东西；它是一个服务映射请求到正确映射器的调度器。让我们看看：

```cs
namespace Core.Mappers;
public interface IMappingService
{
    TDestination Map<TSource, TDestination>(TSource entity);
}
```

该界面是自我解释的；它将任何 `TSource` 映射到任何 `TDestination`。在实现方面，我们正在利用 **服务定位器** 模式，所以我将类命名为 `ServiceLocatorMappingService`：

```cs
namespace Core.Mappers;
public class ServiceLocatorMappingService : IMappingService
{
    private readonly IServiceProvider _serviceProvider;
    public ServiceLocatorMappingService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider ?? throw new ArgumentNullException(nameof(serviceProvider));
    }
    public TDestination Map<TSource, TDestination>(TSource entity)
    {
        var mapper = _serviceProvider.GetService<IMapper<TSource, TDestination>>();
        if (mapper == null)
        {
            throw new MapperNotFoundException(typeof(TSource), typeof(TDestination));
        }
        return mapper.Map(entity);
    }
}
```

逻辑很简单：

+   找到适当的 `IMapper<TSource, TDestination>` 服务，然后使用它来映射实体

+   如果你找不到任何，则抛出 `MapperNotFoundException`

该设计的关键是将映射器注册到 IoC 容器中，而不是服务本身。然后我们使用映射器，而不需要知道它们中的每一个，就像上一个例子中那样。`ServiceLocatorMappingService` 类不知道任何映射器；它只是在需要时动态地请求一个。

> 服务定位器模式不应该成为应用程序代码的一部分。然而，有时它可能很有帮助。例如，我们并不是试图在这个案例中欺骗依赖注入（DI）。相反，我们正在利用它的力量。
> 
> > 当以去除从组合根控制程序组合的可能性的方式来获取依赖项时，使用服务定位器是错误的，这违反了 IoC 原则。
> > 
> > 在这种情况下，我们从 IoC 容器中动态加载映射器，限制了容器控制要注入的内容的能力，这是可以接受的，因为它对程序的维护性、灵活性和可靠性几乎没有负面影响。例如，我们可以替换 `ServiceLocatorMappingService` 实现为另一个类，而不会影响 `IMappingService` 接口消费者。

现在，我们可以在需要映射的任何地方注入该服务并直接使用它。因为我们已经注册了映射器，所以我们只需要将 `IMappingService` 绑定到其 `ServiceLocatorMappingService` 实现并更新消费者。以下是 DI 绑定：

```cs
.AddSingleton<IMappingService, ServiceLocatorMappingService>();
```

如果我们查看移除股票端点的新实现，我们可以看到我们减少了映射器依赖项的数量到一个：

```cs
app.MapPost("/products/{productId:int}/remove-stocks", async (
    int productId, 
    RemoveStocksCommand command, 
    StockService stockService, 
    IMappingService mapper, 
    CancellationToken cancellationToken) => {
    try
    {
        var quantityInStock = await stockService.RemoveStockAsync(productId, command.Amount, cancellationToken);
        var stockLevel = new StockLevel(quantityInStock);
        return Results.Ok(stockLevel);
    }
    catch (NotEnoughStockException ex)
    {
        return Results.Conflict(mapper.Map<NotEnoughStockException, NotEnoughStock>(ex));
    }
    catch (ProductNotFoundException ex)
    {
        return Results.NotFound(mapper.Map<ProductNotFoundException, ProductNotFound>(ex));
    }
});
```

之前的代码与上一个示例类似，但我们用新的服务（突出显示的行）替换了映射器。就是这样；我们现在有一个通用的映射服务，它将映射委托给我们在 IoC 容器中注册的任何映射器。

> 即使你不太可能经常手动实现对象映射器，探索和重新审视这些模式和代码异味是非常好的，并将帮助你编写更好的软件。

这并不是我们对象映射探索的终点。我们有两个工具要探索，首先是 AutoMapper，它为我们完成所有的对象映射工作。

## 项目 – AutoMapper

我们刚刚介绍了实现对象映射的不同方法，但在这里我们利用一个名为 AutoMapper 的开源工具来为我们完成这项工作，而不是自己实现。如果已经有工具做了这件事，为什么还要费心学习所有这些？这样做有几个原因：

+   理解这些概念很重要；你并不总是需要一个完整的工具，比如 AutoMapper。

+   这使我们能够涵盖多个在其他上下文中可以使用并应用于具有不同职责的组件的模式。所以，总的来说，你应该在这次对象映射过程中学习了多种新技术。

+   最后，我们深入研究了将 SOLID 原则应用于编写更好的程序。

AutoMapper 项目也是 Clean Architecture 示例的副本。这个项目和其它项目最大的不同之处在于我们不需要定义任何接口，因为 AutoMapper 提供了一个包含所有所需方法的`IMapper`接口。要安装 AutoMapper，你可以使用 CLI（`dotnet add package AutoMapper`）、Visual Studio 的 NuGet 包管理器或手动更新`.csproj`文件来安装`AutoMapper` NuGet 包。定义我们的映射器最好的方式是使用 AutoMapper 的配置文件机制。配置文件是一个简单的类，它继承自`AutoMapper.Profile`并包含一个对象到另一个对象的映射。我们使用配置文件来分组映射器，但在我们的情况下，只有三个映射，我决定创建一个单独的`WebProfile`类。最后，我们不再手动注册配置文件，而是可以使用`AutoMapper.Extensions.Microsoft.DependencyInjection`包扫描一个或多个程序集，将所有配置文件加载到 AutoMapper 中。

> 当安装`AutoMapper.Extensions.Microsoft.DependencyInjection`包时，你不需要加载`AutoMapper`包。

AutoMapper 的内容远不止我们在这里所涵盖的，但网上有足够的资源，包括官方文档，可以帮助你深入了解这个工具。这个项目的目标是进行基本的对象映射。在*Web*项目中，我们必须创建以下映射：

+   将`Product`映射到`ProductDetails`

+   将`NotEnoughStockException`映射到`NotEnoughStock`

+   将`ProductNotFoundException`映射到`ProductNotFound`

要做到这一点，我们创建以下`WebProfile`类（位于`Program.cs`文件中，但可以存在于任何位置）：

```cs
using AutoMapper;
public class WebProfile : Profile
{
    public WebProfile()
    {
        CreateMap<Product, ProductDetails>();
        CreateMap<NotEnoughStockException, NotEnoughStock>();
        CreateMap<ProductNotFoundException, ProductNotFound>();
    }
}
```

AutoMapper 中的配置文件不过是一个在构造函数中创建映射的类。`Profile`类为你添加了执行此操作所需的方法，例如`CreateMap`方法。这会做什么？调用`CreateMap<Product, ProductDetails>()`方法告诉 AutoMapper 注册一个将`Product`映射到`ProductDetails`的映射器。其他两个`CreateMap`调用为其他两个映射执行相同的操作。目前我们只需要这些，因为 AutoMapper 使用约定来映射属性，并且我们的模型和 DTO 类具有相同名称的属性集合。

> 在前面的例子中，我们在 `Core` 层定义了一些映射器。在这个例子中，我们依赖于一个库，因此考虑依赖流就更加重要。我们仅在 `Web` 层映射对象，因此没有必要在 `Core` 层放置对 AutoMapper 的依赖。记住，所有层都直接或间接依赖于 `Core`，所以在那一层有对 AutoMapper 的依赖意味着所有层也会依赖它。
> 
> > 因此，在这个例子中，我们在 `Web` 层创建了 `WebProfile` 类，将对 AutoMapper 的依赖限制在该层。仅让 `Web` 层依赖 AutoMapper 允许所有外部层（如果我们添加更多的话）控制它们如何进行对象映射，从而为每一层提供更多的独立性。尽可能限制对象映射也是一个最佳实践。
> > 
> > 我在章节末尾的“进一步阅读”部分添加了一个链接到 *AutoMapper 使用指南*。

既然我们已经有一个配置文件，我们需要将其注册到 IoC 容器中，但不必手动操作；我们可以通过使用 `AddAutoMapper` 扩展方法之一从组合根扫描配置文件，以扫描一个或多个程序集：

```cs
builder.Services.AddAutoMapper(typeof(WebProfile).Assembly);
```

前面的方法接受一个 `params Assembly[] assemblies` 参数，这意味着我们可以向它传递多个 `Assembly` 实例。

> `AddAutoMapper` 扩展方法来自 `AutoMapper.Extensions.Microsoft.DependencyInjection` 包。

由于我们只有一个配置文件在一个程序集中，我们利用这个类通过传递 `typeof(WebProfile).Assembly` 参数到 `AddAutoMapper` 方法来访问该程序集。从那里，AutoMapper 在该程序集中扫描配置文件并找到 `WebProfile` 类。如果有多个，它会注册所有找到的。扫描这种类型的好处是，一旦将 AutoMapper 注册到 IoC 容器中，你可以在任何已注册的程序集中添加配置文件，它们会自动加载；之后无需做任何事情，只需编写有用的代码。扫描程序集也鼓励约定式组合，从长远来看更容易维护。程序集扫描的缺点是，当某些内容未注册时，调试可能很困难，因为注册过程不够明确。现在我们已经创建并注册了配置文件到 IoC 容器中，是时候使用 AutoMapper 了。让我们看看最初创建的三个端点：

```cs
app.MapGet("/products", async (
    IProductRepository productRepository, 
    IMapper mapper, 
    CancellationToken cancellationToken) =>
{
    var products = await productRepository.AllAsync(cancellationToken);
    return products.Select(p => mapper.Map<Product, ProductDetails>(p));
});
app.MapPost("/products/{productId:int}/add-stocks", async (
    int productId, 
    AddStocksCommand command, 
    StockService stockService, 
    IMapper mapper, 
    CancellationToken cancellationToken) =>
{
    try
    {
        var quantityInStock = await stockService.AddStockAsync(productId, command.Amount, cancellationToken);
        var stockLevel = new StockLevel(quantityInStock);
        return Results.Ok(stockLevel);
    }
    catch (ProductNotFoundException ex)
    {
        return Results.NotFound(mapper.Map<ProductNotFound>(ex));
    }
});
app.MapPost("/products/{productId:int}/remove-stocks", async (
    int productId, 
    RemoveStocksCommand command, 
    StockService stockService, 
    IMapper mapper, 
    CancellationToken cancellationToken) =>
{
    try
    {
        var quantityInStock = await stockService.RemoveStockAsync(productId, command.Amount, cancellationToken);
        var stockLevel = new StockLevel(quantityInStock);
        return Results.Ok(stockLevel);
    }
    catch (NotEnoughStockException ex)
    {
        return Results.Conflict(mapper.Map<NotEnoughStock>(ex));
    }
    catch (ProductNotFoundException ex)
    {
        return Results.NotFound(mapper.Map<ProductNotFound>(ex));
    }
});
```

前面的代码展示了使用 AutoMapper 与其他选项的相似性。我们注入一个 `IMapper` 接口，然后使用它来映射实体。与上一个例子中显式指定 `TSource` 和 `TDestination` 不同，当使用 AutoMapper 时，我们只需指定 `TDestination` 泛型参数，从而简化代码的复杂性。

> 假设您正在使用 EF Core 返回的`IQueryable`集合上的 AutoMapper。在这种情况下，您应该使用`ProjectTo`方法，该方法将 EF 查询的字段数量限制为您需要的那些。在我们的情况下，这不会改变什么，因为我们需要整个实体。
> 
> > 这里有一个示例，它从 EF Core 获取所有产品并将它们投影到`ProductDto`实例：

```cs
public IEnumerable<ProductDto> GetAllProducts()
{
    return _mapper.ProjectTo<ProductDto>(_db.Products);
}
```

> 在性能方面，这是推荐的方式使用 AutoMapper 与 EF Core。

最后但同样重要的是，我们可以在应用程序启动时断言我们的映射器配置是否有效。这并不识别缺失的映射器，但验证已注册的映射器配置是否正确。推荐的做法是在单元测试中这样做。为了实现这一点，我在末尾添加了以下行，使自动生成的`Program`类公开：

```cs
public partial class Program { }
```

然后，我创建了一个名为`Web.Tests`的测试项目，其中包含以下代码：

```cs
namespace Web;
public class StartupTest
{
    [Fact]
    public async Task AutoMapper_configuration_is_valid()
    {
        // Arrange
        await using var application = new AutoMapperAppWebApplication();
        var mapper = application.Services.GetRequiredService<IMapper>();
        mapper.ConfigurationProvider.AssertConfigurationIsValid();
    }
}
internal class AutoMapperAppWebApplication : WebApplicationFactory<Program>{}
```

在前面的代码中，我们验证了所有的 AutoMapper 映射是否有效。要使测试失败，您可以在`WebProfile`类的以下行取消注释：

```cs
CreateMap<NotEnoughStockException, Product>();
```

`AutoMapperAppWebApplication`类存在是为了在存在多个测试用例时集中初始化测试。在测试项目中，我创建了一个第二个测试用例，确保`products`端点是可到达的。为了使这两个测试一起工作，我们必须更改数据库名称以避免播种冲突，这样每个测试都在自己的数据库上运行。这与我们在`Program.cs`文件中播种数据库的方式有关，这通常不是我们通常做的事情，除非是开发或概念验证。尽管如此，针对多个数据库的测试可以很有用，以便隔离测试。以下是第二个测试用例和更新的`AutoMapperAppWebApplication`类，以供您参考：

```cs
public class StartupTest
{
    [Fact]
    public async Task The_products_endpoint_should_be_reachable()
    {
        await using var application = new AutoMapperAppWebApplication();
        using var client = application.CreateClient();
        using var response = await client.GetAsync("/products");
        response.EnsureSuccessStatusCode();
    }
    // Omitted AutoMapper_configuration_is_valid method
}
internal class AutoMapperAppWebApplication : WebApplicationFactory<Program>
{
    private readonly string _databaseName;
    public AutoMapperAppWebApplication([CallerMemberName]string? databaseName = default)
    {
        _databaseName = databaseName ?? nameof(AutoMapperAppWebApplication);
    }
    protected override IHost CreateHost(IHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            services.AddScoped(sp =>
            {
                return new DbContextOptionsBuilder<ProductContext>()
                    .UseInMemoryDatabase(_databaseName)
                    .UseApplicationServiceProvider(sp)
                    .Options;
            });
        });
        return base.CreateHost(builder);
    }
}
```

运行测试确保我们的应用程序中的映射正常工作，并且有一个端点是可到达的。我们可以添加更多测试，但这两个测试覆盖了我们代码的大约 50%。

> 在前面的代码中使用的`CallerMemberNameAttribute`是`System.Runtime.CompilerServices`命名空间的一部分，允许其装饰的成员访问调用它的方法的名称。在这种情况下，`databaseName`参数接收测试方法名称。

这样就完成了 AutoMapper 项目。此时，你应该开始熟悉对象映射。我建议每次项目需要对象映射时，都评估一下 AutoMapper 是否是合适的工具。如果 AutoMapper 不符合你的需求，你总是可以加载另一个工具或实现自己的映射逻辑。如果映射在太多层级上完成，可能另一种应用程序架构模式会更好，或者需要重新思考。AutoMapper 是基于约定的，并且在我们没有进行任何配置的情况下做了很多工作。它也是基于配置的，通过缓存转换来提高性能。我们还可以创建 **类型转换器**、**值解析器**、**值转换器**等等。AutoMapper 让我们远离编写无聊的映射代码。然而，AutoMapper 已经很老了，功能完善，由于许多项目都在使用它，几乎不可避免。但是，它并不是最快的，这就是为什么我们接下来要探索 Mapperly。

## 项目 – Mapperly

Mapperly 是一个较新的对象映射库，它利用源生成技术使其运行速度极快。要开始使用，我们必须添加对 `Riok.Mapperly` NuGet 包的依赖。

> 源生成器是在 .NET 5 中引入的，允许开发者在编译时生成 C# 代码。

使用 Mapperly 创建对象映射有许多方法，调整映射过程也有许多选项。以下代码示例与其他示例类似，但使用了 Mapperly。我们介绍了以下使用 Mapperly 的方法：

+   注入映射器类。

+   使用静态方法。

+   使用扩展方法。

让我们从注入的映射器开始。首先，类必须是 `partial` 的，这样源生成器才能扩展它（这就是源生成器的工作方式）。用 `[Mapper]` 属性（突出显示）装饰这个类。然后，在那个 `partial` 类中，我们必须创建一个或多个具有我们想要创建的映射器签名的 `partial` 方法（如 `MapToProductDetails` 方法），如下所示：

```cs
[Mapper]
public partial class ProductMapper
{
    public partial ProductDetails MapToProductDetails(Product product);
}
```

在编译时，代码生成器会创建以下类（我已格式化代码以便于阅读）：

```cs
public partial class ProductMapper
{
    public partial ProductDetails MapToProductDetails(Product product)
    {
        var target = new ProductDetails(
            product.Id ?? throw new ArgumentNullException(nameof(product.Id)),
            product.Name, 
            product.QuantityInStock
        );
        return target;
    }
}
```

Mapperly 在生成的 `partial` 类中为我们编写了样板代码，这就是它为什么运行得如此之快。要使用映射器，我们必须将其注册到 IoC 容器中，并将其注入到我们的端点。让我们再次将其设置为单例：

```cs
builder.Services.AddSingleton<ProductMapper>();
```

然后，我们可以这样注入和使用它：

```cs
app.MapGet("/products", async (
    IProductRepository productRepository, 
    ProductMapper mapper, 
    CancellationToken cancellationToken) =>
{
    var products = await productRepository.AllAsync(cancellationToken);
    return products.Select(p => mapper.MapToProductDetails(p));
});
```

前一个代码块中突出显示的代码表明我们可以像使用任何其他类一样使用我们的映射器。最大的缺点是，如果我们没有明智地创建它们，我们可能会将许多映射器注入到单个类或端点中。此外，我们必须将所有映射器注册到 IoC 容器中，这会创建大量的样板代码，但使过程更加明确。另一方面，我们可以扫描程序集以查找所有带有`[Mapper]`属性的类。如果你想为你的映射器提供一个类似于接口的抽象层，你必须自己设计它，因为 Mapperly 只生成映射器。以下是一个示例：

```cs
public interface IMapper
{
    NotEnoughStock MapToDto(NotEnoughStockException source);
    ProductNotFound MapToDto(ProductNotFoundException source);
    ProductDetails MapToProductDetails(Product product);
}
[Mapper]
public partial class Mapper : IMapper
{
    public partial NotEnoughStock MapToDto(NotEnoughStockException source);
    public partial ProductNotFound MapToDto(ProductNotFoundException source);
    public partial ProductDetails MapToProductDetails(Product product);
}
```

前面的代码将所有映射方法集中在一个类和接口下，允许你注入一个类似于 AutoMapper 的接口。在随后的章节中，我们将探讨组织映射器和应用程序代码的方法，这些方法不涉及创建中央映射器类。

> 要检查生成的代码，你可以在项目文件中的`PropertyGroup`标签内添加`EmitCompilerGeneratedFiles`属性，并将其值设置为`true`，如下所示：

```cs
<PropertyGroup>
    <EmitCompilerGeneratedFiles>true</EmitCompilerGeneratedFiles>
</PropertyGroup>
```

> 然后，生成的 C#文件将在`obj\Debug\net8.0\generated`目录下可用。根据你的配置将`net8.0`子目录更改为 SDK 版本和`Debug`。

接下来，我们将探讨如何创建一个静态映射器，其过程非常相似，但我们必须将类和方法都设置为`static`，如下所示：

```cs
[Mapper]
public static partial class ExceptionMapper
{
    public static partial ProductNotFound Map(ProductNotFoundException exception);
}
```

Mapperly 将前面的代码生成以下内容（格式化以提高可读性）：

```cs
public static partial class ExceptionMapper
{
    public static partial ProductNotFound Map(ProductNotFoundException exception)
    {
        var target = new ProductNotFound(
            exception.ProductId, 
            exception.Message
        );
        return target;
    }
}
```

一次又一次，代码生成器编写了样板代码。区别在于我们不需要注入任何依赖，因为这是一个静态方法。我们可以这样使用它（我只包括了捕获块，其余代码保持不变）：

```cs
catch (ProductNotFoundException ex)
{
    return Results.NotFound(ExceptionMapper.Map(ex));
}
```

这非常直接，但会在生成的类及其消费者之间建立强大的联系。如果你的项目可以接受对静态类的硬依赖，你可以使用这些静态方法。我们正在探索的将对象映射的最后一種方法非常相似，但我们是在同一个类中创建一个扩展方法，而不是仅仅一个静态方法。下面是新的方法：

```cs
public static partial NotEnoughStock ToDto(this NotEnoughStockException exception);
```

该方法的生成代码如下（格式化）：

```cs
public static partial NotEnoughStock ToDto(this NotEnoughStockException exception)
{
    var target = new NotEnoughStock(
        exception.AmountToRemove, 
        exception.QuantityInStock, 
        exception.Message
    );
    return target;
}
```

唯一的区别是添加了`this`关键字，将常规静态方法转换为扩展方法，我们可以这样使用它：

```cs
catch (NotEnoughStockException ex)
{
    return Results.Conflict(ex.ToDto());
}
```

扩展方法比静态方法更优雅，但仍然创建了一个类似于静态方法的联系。再次强调，选择如何进行映射完全取决于你。关于 Mapperly 的一个值得注意的事情是，当映射代码不正确或可能不正确时，其分析器会提供信息、警告或错误。消息的严重性是可以配置的。例如，如果我们向`ExceptionMapper`类添加以下方法，Mapperly 会生成`RMG013`错误：

```cs
public static partial Product NotEnoughStockExceptionToProduct(
    NotEnoughStockException exception
);
```

错误信息：

```cs
RMG013 Core.Models.Product has no accessible constructor with mappable arguments
```

此外，两个异常映射方法会提供关于目标类上不存在属性的信息。以下是一个此类消息的示例：

```cs
RMG020 The member TargetSite on the mapping source type Core.ProductNotFoundException is not mapped to any member on the mapping target type ProductNotFound
```

在这些基础上，我们知道何时某些事情是错误的或可能出错，这保护我们免受配置错误的影响。让我们结束这一章。

## 摘要

在许多情况下，对象映射是一个不可避免的现实。然而，正如我们在本章中看到的，有几种实现对象映射的方法，这可以从我们应用程序的其他组件中移除这一责任，或者简单地手动内联编码。同时，我们有机会探索聚合服务模式，它为我们提供了一种将多个依赖项集中到一个地方的方法，从而降低了其他类中所需的依赖项数量。这种模式可以帮助解决“过多的依赖项”代码异味，这是一种经验法则，指出我们应该调查具有超过三个依赖项的对象以查找设计缺陷。当将依赖项移动到聚合中时，请确保聚合内部具有内聚性，以避免向程序添加不必要的复杂性，并仅仅移动依赖项。我们还探讨了利用外观模式来实现映射外观，这导致了一个更易于阅读和优雅的映射器。之后，我们实现了一个模仿外观的映射器服务。尽管在用法上不够优雅，但它更加灵活。我们最终探索了 AutoMapper 和 Mapperly，这两个开源工具为我们执行对象映射，为我们提供了许多配置对象映射的选项。在我们探索的过程中，仅使用 AutoMapper 的默认约定就允许我们消除所有的映射代码。在 Mapperly 方面，我们必须使用部分类和方法定义映射器合约，以便其代码生成器为我们实现映射代码。你可以从许多现有的对象映射库中选择，AutoMapper 是其中最古老、最著名同时也是最被憎恨的一个，而 Mapperly 是最新且最快的，但仍然处于起步阶段。希望随着我们越来越多地将各个部分组合在一起，你现在开始看到我在本书开头提到这是架构之旅时的心思。现在我们已经完成了对象映射，下一章我们将探讨中介者和 CQRS 模式。

## 问题

让我们看看几个练习问题：

1.  注入聚合服务而不是多个服务是否真的能提高我们的系统？

1.  使用映射器是否真的有助于我们从消费者中提取责任到映射器类中？

1.  真的是应该始终使用 AutoMapper 吗？

1.  当使用 AutoMapper 时，你应该将你的映射代码封装到配置文件中吗？

1.  应该有多少依赖项开始引起你的注意，告诉你你正在向单个类注入过多的依赖项？

## 进一步阅读

这里有一些链接，可以帮助我们巩固本章所学的内容：

+   如果你想了解更多关于对象映射的信息，我在 2017 年写过一篇关于这个主题的文章，标题为 *设计模式：ASP.NET Core Web API、服务和仓储 | 第九部分：NinjaMappingService 和外观模式*：[`adpg.link/hxYf`](https://adpg.link/hxYf)

+   AutoMapper 官方网站：[`adpg.link/5AUZ`](https://adpg.link/5AUZ)

+   *AutoMapper 使用指南* 是一份优秀的“做/不做”清单，可以帮助你正确使用 AutoMapper，由库的作者编写：[`adpg.link/tTKg`](https://adpg.link/tTKg)

+   Mapperly（GitHub）：[`adpg.link/Dwcj`](https://adpg.link/Dwcj)

## 答案

1.  是的，聚合服务可以改进系统，但并不一定总是如此。移动依赖关系并不能修复设计缺陷；它只是将这些缺陷移动到其他地方。

1.  是的，映射器帮助我们遵循单一职责原则（SRP）。然而，它们并不总是必需的。

1.  不，它并不适合每个场景。例如，当映射逻辑变得复杂时，考虑不使用 AutoMapper。过多的映射器也可能意味着应用程序设计本身存在缺陷。

1.  是的，使用配置文件来组织你的映射规则是有益的。

1.  四个或更多的依赖项应该开始引起注意。再次强调，这只是一个指导原则；将四个或更多的服务注入到类中可能是可接受的。
