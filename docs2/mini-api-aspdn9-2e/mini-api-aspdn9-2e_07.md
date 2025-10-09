

# 最小 API 中的依赖注入

在任何软件项目中，开发者很少从头开始构建应用程序。在某个层面上，通用库和工具集将被吸收到应用程序中，以加速和优化项目。**ASP.NET**作为一个框架也不例外。事实上，它要求开发者承担依赖项；第三方或独立创建的代码，这些代码在系统的平稳运行中起着关键作用。

结果是一个（希望是）精心调整和设计良好的架构，由模块和组件组成，其中一些运行由项目中的开发者编写的代码，其余的是更通用的、模板化的代码，这些代码在项目开始前编写并预编译。

跟踪依赖项是软件开发者面临的一个经典问题，由此产生的问题可能会发展到导致行业所说的**依赖地狱**——一个噩梦般的场景，其中开发者正在追溯他们的步骤，试图找出依赖项是如何被引入的，并找到克服跨可能庞大的代码库中冲突依赖项挑战的方法。

**依赖注入**（**DI**）是一种在软件项目中标准化和简化消费依赖项体验的方法。

在本章中，我们将涵盖以下主要主题：

+   理解 DI

+   在最小 API 中配置 DI

+   DI 最佳实践

到本章结束时，你将提高对 DI 原则的理解，以及它们可以为最小 API 和 ASP.NET 项目等一般项目带来的好处。

你还将获得配置 DI 容器和服务注册的实际经验。

让我们先从提高我们对 DI 的理解开始。

# 理解 DI

DI 最初是软件开发中的一个设计模式，旨在集中管理常用依赖项，并以一致的方式提供给消费者。

使用这种方法，常见的开发任务，如测试、替换依赖项、修改依赖项逻辑、集中依赖项等，都可以通过一个简单系统轻松实现。

随着时间的推移，.NET 将 DI 从仅仅是一个设计模式转变为一个功能。在 ASP.NET 中，有一个强大且易于使用和理解的 DI 工具集。

开发者可以在一个集中位置注册他们的依赖项，使它们在实例化时可以作为参数注入到类的构造函数中。

使用 DI，依赖项存在于一个*容器*中，使得它们对消费类中央可用。但什么是容器呢？

## DI 容器

在应用程序启动时，依赖项会在容器中注册。**容器**只是已经注册用于 DI 的一组依赖项。每个依赖项都有一个生命周期规范，它定义了它们在注入到消费类时是如何实例化的。我们将在本章后面更详细地探讨依赖项的生命周期。

当一个具有依赖项的类被实例化时，它会向容器发出请求，容器会负责解决依赖项并根据在依赖项注册时配置的生命周期设置来实例化它。

这可能听起来像是使用另一个类中的类这样简单的事情的额外复杂性层，但强制使用 DI 作为最佳实践有很好的理由。让我们更详细地探讨这一点。

## 依赖注入的案例

回想一下你作为一名软件工程师的职业生涯。无论你还在起步阶段，还是已经从事这一行业一段时间，你可能已经花费了大量时间以类的形式“new”依赖项。

假设你正在构建一个需要访问 SQL 数据库的 API 端点。（为了简化，我故意没有使用 Entity Framework。）你可能已经创建了一个抽象化具体细节的类（我们可以称之为 **SqlHelper**），例如创建一个 **SqlConnection** 实例、打开连接、构建 **SqlCommand** 等。当你意识到需要这个 **SqlHelper** 类时，你认为你需要做什么？

你首先会注意到，每当需要与你的 SQL Server 交互时，你必须创建一个新的 **SqlHelper** 实例。表面上，这似乎无害，但从设计角度来看，这存在一些问题。让我们更仔细地看看这种方法的潜在陷阱：

+   **紧密耦合**：没有依赖注入（DI），每次使用 **SqlHelper** 时都会创建其具体实现。每当有一个类的具体实现时，如果你需要显著更改 **SqlHelper**，你就面临着被迫更改每个使用它的类的风险。这意味着你的消费类会紧密耦合到 **SqlHelper**。

+   **测试困难**：能够模拟依赖项对于有效的测试至关重要。没有 DI，你将需要更手动地确保依赖项被正确实例化、模拟，并且对每个测试都是可访问的。手动实例化的额外需求增加了设置测试时出现错误的可能性。这是一个问题，因为它可能会使你的测试变得不可靠。

+   **资源管理问题**：当依赖使用资源时，例如**SqlHelper**（它将有一个与 SQL Server 的连接），总存在这些资源没有被有效管理的风险。在类似 SQL 连接的情况下，如果没有适当的释放，随着时间的推移创建大量这些连接可能会导致连接耗尽，从而引发性能问题。

+   **违反单一职责、开闭原则、里氏替换原则、接口隔离原则、依赖倒置原则（SOLID 原则）**：本书中我们尚未探讨 SOLID 原则，但它们是任何面向对象软件系统的重要组成部分。SOLID 的一个指导原则是**单一职责**，即我们期望确保类有一个主要职责。对于一个消费**SqlHelper**的类来说，它们的主要职责是对请求或操作数据的逻辑。迫使类实例化**SqlHelper**意味着你给它赋予了新的职责；管理其自身依赖的职责。依赖注入（DI）移除了这个附加的职责，简单地在类构造时将依赖传递给类。

希望这次分解已经描绘了不使用 DI 如何使你的代码库不一致和混乱的画面。现在，让我们探讨 ASP.NET 中如何实现 DI。

# 在最小 API 中配置 DI

作为标准，ASP.NET 提供了一种方法，允许我们声明我们创建的类可以被注册为服务。将类转换为服务意味着它可以通过 DI 进行重用。例如，假设你有一段逻辑，用于计算任何给定员工的加班费。这段逻辑是相同的，但你需要在代码库的许多其他区域使用它。为了避免再次编写相同的逻辑，显然你会调用相同的逻辑，但正如我们之前讨论的，每次需要时都创建类的新的实例以获取这段逻辑是混乱的；因此，通过将类注册为服务，我们可以干净地将其注入到任何需要它的其他类中。

此外，DI 允许我们控制注入时服务的生命周期。本质上，我们可以指定依赖在每个注入中是如何实例化的，以及它应该存在多久。

ASP.NET 中有三种内置的生命周期选项：

+   **单例**：服务被创建一次，作为一个单一实例。然后这个实例在代码库中被共享。当你需要在全球范围内维护状态时，这很有用。日志记录是一个很好的用例，因为所有日志条目都可以通过一个单一的服务进行通道，该服务可以访问相关的输出资源。例如，一个在文件中创建日志的日志服务。

+   **作用域**：为每个传入的请求创建一次服务。这意味着当客户端向 API 发出请求时，需要时创建服务，该实例在整个请求期间处于使用状态。这在需要管理请求内的状态时是理想的。如果您不希望在不同请求之间共享相同的服务，这也是有利的。

+   **瞬态**：每次注入服务时都会创建一个实例。这意味着无论对 API 发出的请求是什么，每次注入服务时，该服务都将是一个新实例。这在不需要维护状态的场景中是理想的。

让我们设置一个新的最小 API 项目，作为我们如何从依赖注入（DI）中受益的示例。

请注意，如果您还没有阅读它们，请参考前两章了解您如何创建一个新的最小 API 项目。这将使您能够跟随本章中的示例。

## 设置一个范围依赖注入（DI）项目

对于我们的新 API 项目，我们将以订单处理 API 为例。它将包含一系列可以组合成订单的产品或服务。

首先，我们需要模型来表示产品和订单。创建两个类，**Product**和**Order**。在第一段代码中，我们创建**Product**类：

```cs
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Description { get; set; }
        public float RRP { get; set; }
    }
```

在以下代码中，我们创建**Order**类：

```cs
    public class Order
    {
        public int Id { get; set; }
        public List<Product> Products { get; set; }
        public decimal DiscountAmount { get; set; }
        public DateTime DeliveryDate { get; set; }
    }
```

我们需要能够引用一组可用的产品。通常，我们会将此信息存储在数据库中，然后使用**SqlConnection**或**对象关系映射**（**ORM**）框架，例如微软的 Entity Framework，来访问数据库，将数据映射到我们创建的模型（**Product**和**Order**）。然而，数据库连接不在本章的范围内，将在本书的后续章节中介绍。

现在，为了简单起见，我们将简单地创建一个包含对象的 JSON 文件，这些对象可以作为文本读入项目，并反序列化为强类型对象**Product**。我已经创建了一个包含五个产品的示例，这些产品可以以 JSON 格式保存，以下代码。您可以随意复制我的示例或创建自己的示例。无论您做什么，请将产品保存在名为**Products.json**的文件中，并确保每个项目都是一个包含在单个 JSON 数组中的 JSON 对象，并且您使用的值与**Product**属性的数据类型相匹配；否则，将无法反序列化 JSON：

```cs
[
    {
        "Id": 1,
        "Name": "Laptop",
        "Description": "A high-performance laptop suitable
                       for all your computing needs.",
        "RRP": 999.99
    },
    {
        "Id": 2,
        "Name": "Smartphone",
        "Description": "A latest generation smartphone with
                       a stunning display and excellent
                       camera.",
        "RRP": 799.99
    },
    {
        "Id": 3,
        "Name": "Headphones",
        "Description": "Noise-cancelling headphones with
                       superior sound quality.",
        "RRP": 199.99
    },
    {
        "Id": 4,
        "Name": "Smartwatch",
        "Description": "A smartwatch with fitness tracking
                       and health monitoring features.",
        "RRP": 299.99
    },
    {
        "Id": 5,
        "Name": "Tablet",
        "Description": "A lightweight tablet with a vibrant
                       display, perfect for entertainment
                       on the go.",
        "RRP": 399.99
    }
]
```

现在，让我们创建一种方法，在需要时将这些对象带入内存。（再次强调，这不是最有效率的示例，因为我们没有使用数据库，但我们将在本书的后续章节中介绍数据库的使用。）

对于这个示例，我们将通过创建一个名为**ProductRepository**的类来实现。这个类可以用来访问类型为**Product**的对象列表。

按照以下示例添加**ProductRepository**类：

```cs
public class ProductRepository
{
    public List<Product> Products { get; private set; }
}
```

如您所见，这是一个非常简单的类，它只包含一个 **Product** 列表。我们需要以某种方式填充这个列表，以包含我们已保存为文本的 JSON 对象。我们可以在实例化类时轻松地获取这些项目，但我们要使用注入的服务来做这件事，所以我们会很快回到 **ProductRepository**。在那之前，让我们创建一个将负责从文本文件中检索产品的服务。我们将称之为 **ProductRetrievalService**：

```cs
    public class ProductRetrievalService
    {
        private const string _dataPath =
            @"C:/Products.json";
        public List<Product> LoadProducts()
        {
            var productJson = File.ReadAllText(_dataPath);
            return JsonSerializer
                .Deserialize<List<Product>>(
                    productJson
                );
        }
    }
```

这个简单的服务读取 JSON 文件的全部内容，并使用 **System.Text.Json** 中找到的 **JsonSerializer** 类将 JSON 内容转换或反序列化为强类型的 **Product** 类型，将每个 **Product** 放入列表中。

C:/ 的权限

如果你写或读 **C:/** 时遇到麻烦，你可能没有权限这样做。你可以通过在具有读写权限的位置创建一个文件夹来解决这个问题，然后更改代码中的路径以匹配新的路径。

到目前为止，产品已被检索。这意味着我们可以简单地调用 **LoadProducts()**，我们总是会得到最新的数据。然而，我们如何访问 **ProductRetrievalService** 来做到这一点？我们的 **ProductRepository** 类需要这个逻辑来填充其 **Product** 列表。

在这里，依赖注入变得有用。我们可以在使用 **ProductRepository** 的任何时候注入一个 **ProductRetrievalService** 实例。为了使这成为可能，我们首先需要将 **ProductRetrievalService** 注册为服务。

以下代码演示了在 **Program.cs** 中注册此服务以进行依赖注入的示例：

```cs
public static void Main(string[] args)
{
    var builder = WebApplication.CreateBuilder(args);
    builder.Services.AddScoped<ProductRetrievalService>();
    var app = builder.Build();
    app.Run();
}
```

通过将 **ProductRetrievalService** 作为作用域服务添加，将为每个传入请求创建一个实例。现在它已注册，我们可以在实例化时通过其构造函数将 **ProductRetrievalService** 注入到 **ProductRepository** 中。让我们通过一个 API 端点示例来看看这个例子。

创建一个新的 HTTP **GET** 方法，映射到 **getProductById** 路由，如下面的代码所示：

```cs
    app.MapGet("/getProductById/{id}", (int id) =>
    {
    });
```

端点接受一个整数参数，即产品 ID。我们现在可以使用这个参数来获取具有匹配 ID 的产品。首先，让我们向端点添加一个新的 **ProductRepository** 实例：

```cs
  app.MapGet("/getProductById{id}", (int id) =>
  {
      var productRepository = new ProductRepository();
  });
```

现在我们有一个 **ProductRepository** 实例，它有一个 **Product** 列表，但这个列表是空的。我们需要修改 **ProductRepository** 以注入 **ProductRetreivalService** ，以填充这个列表。下面的代码展示了如何通过构造函数将服务注入到 **ProductRepository** 中，以便在使用之前填充 **List<Product>** 中持有的产品：

```cs
public class ProductRepository
{
    public List<Product> Products { get; private set; }
    public ProductRepository(
        ProductRetrievalService productRetrievalService
    )
    {
        Products = productRetrievalService.LoadProducts();
    }
}
```

现在，我们应该能够在端点中使用一些逻辑来从 **ProductRepository** 获取相关产品。然而，我们有一个问题。如果我们尝试在端点中实例化一个新的 **ProductRepository** 实例，我们会看到一个错误。

我们看到错误的原因是我们改变了 **ProductRepository** 的实例化方式。现在它需要一个 **ProductRetrievalService** 作为构造函数的参数，但我们应该如何获取这个服务呢？

正是这里，最小化 API 允许我们在端点内部利用 DI 容器中注册的服务。

**ProductRetreivalService** 可以作为参数传递给端点体内部 lambda 表达式中的参数。这使得它和客户端传入的 ID 参数相同，只不过它不是来自客户端，而是来自 DI 容器。

要实现这一点，您需要将 **ProductRetrievalService** 参数前缀为一个表示其被注入的属性。这个属性是 **[FromServices]** 。

使用此属性注入 **ProductRetrievalService** 现在将允许我们将所需的 **ProductRetrievalService** 传递给 **ProductRepository** 的构造函数，如下所示：

```cs
app.MapGet("/getProductById/{id}",
    (
        int id,
        [FromServices] ProductRetrievalService
            productRetrievalService
    ) =>
    {
    var productRepository = new ProductRepository(
        productRetrievalService
    );
    return Results.Ok(
        productRepository.Products
            .FirstOrDefault(x => x.Id == id)
    );
});
```

值得注意的是，我们在本例中创建的 **ProductRepository** 实例本身也可以通过 DI 注入到类中。

现在让我们继续下一个例子。

## 创建一个单例 DI 项目

让我们来看另一个例子，但这次我们将使用不同的依赖生命周期。在这个用例中，我们将创建一个用于创建订单的端点。传入的 **Order** 对象将包含一个 **Product** 列表，可以用来将新的客户订单提交到系统中。然而，我们还需要确定一个交货日期。

注意

为了避免重复，在这个例子中，我们可以简单地使用一个集合，例如 **List<DateTime>**，而不是从 JSON 文件中读取它们。

让我们假设有一个即将可用的日期的流，这些日期是集中管理的。我们可以创建一个服务，它具有选择下一个可用日期的上下文。这将从端点逻辑中解耦，并可以在其他端点中重用。

代码展示了这种类型服务的示例：

```cs
public class DeliveryDateBookingService
{
    private ConcurrentQueue<DateTime>
        _availableDates = new ConcurrentQueue<DateTime>();
    public DeliveryDateBookingService()
    {
        _availableDates.Enqueue(DateTime.Now.AddDays(1));
        _availableDates.Enqueue(DateTime.Now.AddDays(2));
        _availableDates.Enqueue(DateTime.Now.AddDays(3));
        _availableDates.Enqueue(DateTime.Now.AddDays(4));
        _availableDates.Enqueue(DateTime.Now.AddDays(5));
    }
    public DateTime GetNextAvailableDate()
    {
        if(_availableDates.Count == 0)
        {
            throw new Exception("No Dates Available");
        }
        var dequeuedDate = _availableDates
            .TryDequeue(out var result);
        if (dequeuedDate == false)
        {
            throw new Exception("An error occured");
        }
        return result;
    }
}
```

此服务允许请求获取下一个可用日期，如果没有可用日期则抛出异常。我们有一个队列来保存可用日期，以确保在检索时不会重复提供相同的日期。

我们还必须考虑线程安全性。可能会有多个请求都在尝试获取一个可用的日期，这很可能导致竞态条件，其中两个请求最终会出队相同的可用日期。为了避免这种情况，我们正在使用 **ConcurrentQueue**，它将处理确保请求之间线程安全的事务。

现在，我们需要将其注册为一个可以被注入到发布订单端点的服务。考虑到可能会有多个请求，我们想要确保所有请求都是从同一个列表中检索日期。因此，我们将使用**AddSingleton()**来注册此服务，这将确保在注入期间线程和请求之间只使用一个服务实例。

一旦以这种方式注册了服务，**Program.cs**应该看起来像下面的代码所示：

```cs
public class Program
    {
        public static void Main(string[] args)
        {
            var builder = WebApplication
                .CreateBuilder(args);
            builder.Services
                .AddScoped<ProductRetrievalService>();
            builder.Services
                .AddSingleton<DeliveryDateBookingService>()
                ;
            var app = builder.Build();
            app.MapGet("/getProductById/{id}",
                (int id,
                [FromServices] ProductRetrievalService
                productRetreivalService) =>
            {
                var productRepository = new
                    ProductRepository(
                        productRetreivalService
                );
                return Results.Ok(
                    productRepository.Products
                        .FirstOrDefault(
                            x => x.Id == id)
                );
            });
            app.Run();
        }
    }
```

现在 API 已经注册了第二个服务，是时候创建创建订单的端点了。

由于我们正在创建一个新的记录，我们应该使用**POST**方法来实现我们的目标。**POST**方法将接受一个 JSON 对象，该对象在端点参数内隐式解析为**Order**对象。

随后，我们指出我们正在将**DeliveryDateBookingService**注入到请求中。

完成此操作后，我们可以通过向 lambda 表达式的主体中添加相关逻辑来完成端点。

获取下一个交货日期的逻辑端点如下所示：

```cs
  app.MapPost(
      "/order",
      (Order order,
       [FromServices] DeliveryDateBookingService
       deliveryDateBookingService) =>
  {
      order.DeliveryDate =
          deliveryDateBookingService.GetNextAvailableDate()
          ;
      // save order to repository
  });
```

虽然我们为**Product**创建了一个仓库，但我们还没有为**Order**创建一个。此外，我们还没有演示将实体保存到各自的仓库中。

在我们探讨设计模式（如仓库模式）和数据源时，这种逻辑将在本书的后续部分进行介绍，但就目前而言，这里有一个例子说明你如何在先前的端点中保存新订单：

1.  创建一个**OrderRetreivalService**类，以便我们保持使用服务检索实体的一致性（就像我们为产品所做的那样）：

    ```cs
    public class OrderRetrievalService
    {
        private const string _dataPath =
            @"C:/Orders.json";
        public List<Order> LoadOrders()
        {
            var ordersJson = File.ReadAllText(_dataPath);
            return JsonSerializer
                .Deserialize<List<Order>>(ordersJson);
        }
    }
    ```

1.  在**Program.cs**中注册**OrderRetrievalService**为一个作用域服务：

    ```cs
                builder.Services
                    .AddScoped<OrderRetrievalService>();
    ```

1.  创建一个遵循与**ProductRespository**相同风格的**OrderRespository**类。添加的不同之处在于增加了一个**SaveOrder()**方法，允许从**POST**端点保存**Order**。此外，用于存储订单的集合是**ConcurrentQueue<Order>**而不是列表。这是因为我们预计订单将从多个并发请求中保存，并且我们需要允许线程安全：

    ```cs
    public class OrderRepository
    {
        public ConcurrentQueue<Order>
            Orders { get; private set; }
        public OrderRepository(
            OrderRetrievalService orderRetrievalService)
        {
            var retrievedOrders =
                orderRetrievalService.LoadOrders();
            foreach (var order in retrievedOrders)
            {
                Orders.Enqueue(order);
            }
        }
        public void SaveOrder(Order order)
        {
            Orders.Enqueue(order);
        }
    }
    ```

1.  在**Program.cs**中将**OrderRepository**注册为单例服务，这样无论保存订单的请求数量有多少，我们都可以始终添加到单个实例中：

    ```cs
                builder.Services
                    .AddSingleton<OrderRepository>();
    ```

1.  现在可以将**POST**端点更新为注入**OrderRepository**并使用它来保存传入的订单：

    ```cs
      app.MapPost(
          "/order",
          (Order order,
           [FromServices] DeliveryDateBookingService
           deliveryDateBookingService,
           [FromServices] OrderRepository
           orderRepository) =>
          {
              order.DeliveryDate =
                  deliveryDateBookingService
                      .GetNextAvailableDate();
              orderRepository.SaveOrder(order);
      });
    ```

现在你已经有一些创建依赖项作为服务和注册它们以供注入的经验，让我们回顾一下在最小 API 中使用 DI 的一些基本最佳实践。

# DI 最佳实践

DI 对于大多数 ASP.NET 项目至关重要，而最小 API 通常特别依赖于它们。因此，确保我们在依赖关系及其访问方法方面遵循最佳实践非常重要。

在最小化 API 中实现依赖注入（DI）时，有一些简单的经验法则。我们将在接下来的几节中探讨这些规则。

## 避免使用服务定位器模式

在最小化 API 中存在一种称为**服务定位器模式**的反模式。在这种模式中，你不是显式注入依赖项，而是注入包含依赖项的**IServiceProvider**，然后在你的方法或函数体中从其中提取服务。

下面的代码示例展示了服务定位器模式的一个例子，其中我们为创建订单而创建的**POST**方法被修改为使用**IServiceProvider**：

```cs
  app.MapPost(
      "/order",
      (Order order,
       IServiceProvider provider) =>
      {
          var deliveryDateBookingService =
              provider.GetService
                  <DeliveryDateBookingService>();
          order.DeliveryDate =
              deliveryDateBookingService
                  .GetNextAvailableDate();
      // save order to repository in same way we did for
      // Product using ProductRepository
  });
```

这种做法的一个显著缺点是它使得代码更难阅读。从参数中不明显看出你正在注入特定的服务，你必须编写额外的代码来获取**IServiceProvider**的服务。

它还使得为你的端点编写单元测试更加困难，因为不清楚你需要实例化哪些对象来进行模拟。

这种反模式最具破坏性的方面可能是运行时故障难以诊断。当注入**IServiceProvider**时，编译器不知道你实际需要的服务是否已注册，而如果你尝试显式注入你的服务且它未注册，问题将更快地显现出来，从而便于调试。

## 使用扩展方法注册服务

你可以通过在**IServiceCollection**上创建扩展方法来使你的代码更具可读性。这意味着在**Program.cs**中，你可以用一行代码注册所有服务，或者以适当的方式将服务分组在一起并分别注册每个组。

下面是一个如何编写此类扩展方法的示例：

```cs
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddMyServices(
        this IServiceCollection services)
    {
        services.AddScoped<IMyService, MyService>();
        services.AddSingleton<IOtherService,
            OtherService>();
        return services;
    }
}
```

实现扩展方法后，你可以在**Program.cs**中简单地写下以下内容来注册所有服务：

```cs
builder.Services.AddMyServices();
```

## 使用合理的服务生命周期

在注册服务时，考虑分配给它们的生命周期很重要。以下是一些你可能会使用每种服务生命周期的例子：

+   **Transient**：如果你的服务轻量级、无状态，并且只用于短时间内，请使用此生命周期。

+   **Scoped**：当你的服务必须在单个请求中维护状态，并且状态需要对于当前请求是唯一的时候，请使用此生命周期。

+   **Singleton**：当你的服务必须在整个应用程序中维护状态时，请使用此生命周期。在需要创建成本高昂的重型服务的情况下，这也很有用。一旦创建，就可以减少开销。

在创建和管理依赖项时努力遵循最佳实践是一种长期投资，这是一种无私的行为，确保代码库在未来对其他开发者来说易于维护。

让我们总结一下本章所涵盖的内容。

# 摘要

我们从高层次开始本章，通过了解它如何通过鼓励良好的设计、松散耦合和代码库中的可重用性，为最小 API 带来好处。

然后，我们通过一个订单处理 API 的例子，探讨了如何以服务的形式创建依赖项，并在注册后进行注入。

已经证明，可以在最小的 API 端点中使用参数属性将服务注入到端点执行的范围中，我们还介绍了服务在注册时可以使用的各种生命周期。

最后，我们概述了一些最佳实践，帮助您确保您的依赖注入（DI）使用是高效、可持续、可测试的，同时对于可能不太熟悉项目的其他开发者来说也易于阅读。

依赖注入（DI）不仅是最小 API 的基本方面，也是软件工程的一般基础。对它的良好理解对于您作为开发者的成功至关重要。

在本章中，我们还使用了相当非正统的方法来存储和读取数据，以便在示例 API 端点中使用。这样做有很好的理由。通常，我们会使用更标准化的数据源来托管和检索实体，这是我们将在下一章中详细探讨的内容。
