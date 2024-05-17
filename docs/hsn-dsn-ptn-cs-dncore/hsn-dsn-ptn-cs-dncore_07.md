# 第五章：实现设计模式-.NET Core

上一章继续构建 FlixOne 库存管理应用程序，同时还包括其他模式。使用了更多的四人帮模式，包括 Singleton 和 Factory 模式。Singleton 模式用于说明用于维护 FlixOne 图书集合的 Repository 模式。Factory 模式用于进一步探索**依赖注入**（**DI**）。使用.NET Core 框架完成了初始库存管理控制台应用程序，以便实现**控制反转**（**IoC**）容器。

本章将继续构建库存管理控制台应用程序，同时还将探索.NET Core 的特性。将重新访问并创建上一章中介绍的 Singleton 模式，使用内置于.NET Core 框架中的 Singleton 服务生命周期。将展示使用框架的 DI 的配置模式，以及使用不同示例解释**构造函数注入（CI）**。

本章将涵盖以下主题：

+   .Net Core 服务生命周期

+   实现工厂

# 技术要求

本章包含用于解释概念的各种代码示例。代码保持简单，仅用于演示目的。大多数示例涉及使用 C#编写的.NET Core 控制台应用程序。

要运行和执行代码，您需要以下内容：

+   Visual Studio 2019（您也可以使用 Visual Studio 2017 版本 3 或更高版本运行应用程序）。

+   设置.NET Core。

+   SQL Server（本章中使用的是 Express 版本）。

# 安装 Visual Studio

要运行这些代码示例，您需要安装 Visual Studio 2010 或更高版本。您可以使用您喜欢的 IDE。要做到这一点，请按照以下说明进行操作：

1.  从以下链接下载 Visual Studio：[`docs.microsoft.com/en-us/visualstudio/install/install-visual-studio`](https://docs.microsoft.com/en-us/visualstudio/install/install-visual-studio)。

1.  按照包含的安装说明进行操作。Visual Studio 有多个版本可供安装。在本章中，我们使用的是 Windows 版的 Visual Studio。

# 设置.NET Core

如果您没有安装.NET Core，则需要按照以下说明进行操作：

1.  从以下链接下载.NET Core：[`www.microsoft.com/net/download/windows`](https://www.microsoft.com/net/download/windows)。

1.  安装说明和相关库可以在以下链接找到：[`dotnet.microsoft.com/download/dotnet-core/2.2`](https://dotnet.microsoft.com/download/dotnet-core/2.2)。

完整的源代码可在 GitHub 存储库中找到。本章中显示的源代码可能不完整，因此建议检索源代码以运行示例。请参阅[`github.com/PacktPublishing/Hands-On-Design-Patterns-with-C-and-.NET-Core/tree/master/Chapter5.`](https://github.com/PacktPublishing/Hands-On-Design-Patterns-with-C-and-.NET-Core/tree/master/Chapter5)

# .Net Core 服务生命周期

在使用.NET Core 的 DI 时，理解服务生命周期是一个基本概念。服务生命周期定义了依赖项的管理方式，以及它被创建的频率。作为这一过程的说明，将 DI 视为管理依赖项的容器。依赖项只是 DI 知道的一个类，因为该类已经与它*注册*。对于.NET Core 的 DI，可以使用`IServiceCollection`的以下三种方法来完成这一过程：

+   `AddTransient<TService, TImplementation>()`

+   `AddScoped<TService, TImplementation>()`

+   `AddSingleton<TService, TImplementation>()`

`IServiceCollection`接口是已注册的服务描述的集合，基本上包含依赖项以及 DI 应该何时提供依赖项。例如，当请求`TService`时，会提供`TImplementation`（也就是注入）。

在本节中，我们将查看三种服务生命周期，并通过单元测试提供不同生命周期的示例。我们还将看看如何使用实现工厂来创建依赖项的实例。

# 瞬态

`瞬态`依赖项意味着每次 DI 接收到对依赖项的请求时，将创建依赖项的新实例。在大多数情况下，这是最合理使用的服务生命周期，因为大多数类应设计为轻量级、无状态的服务。在需要在引用之间保持状态和/或在实例化新实例方面需要大量工作的情况下，可能会更合理地使用另一种服务生命周期。

# 作用域

在.Net Core 中，有一个作用域的概念，可以将其视为执行过程的上下文或边界。在某些.Net Core 实现中，作用域是隐式定义的，因此您可能不知道它已经被放置。例如，在 ASP.Net Core 中，为接收到的每个 Web 请求创建一个作用域。这意味着，如果一个依赖项具有作用域生命周期，那么它将仅在每个 Web 请求中构造一次，因此，如果相同的依赖项在同一 Web 请求中多次使用，它将被共享。

在本章后面，我们将明确创建一个范围，以说明作用域生命周期，相同的概念也适用于单元测试，就像在 ASP.Net Core 应用程序中一样。

# 单例

在.Net Core 中，Singleton 模式的实现方式是依赖只被实例化一次，就像在上一章中实现的 Singleton 模式一样。与上一章中的 Singleton 模式类似，`singleton`类需要是线程安全的，只有用于创建单例类的工厂方法才能保证只被单个线程调用一次。

# 回到 FlixOne

为了说明.Net Core 的 DI，我们需要对 FlixOne 库存管理应用程序进行一些修改。首先要做的是更新之前定义的`InventoryContext`类，以便不再实现 Singleton 模式（因为我们将使用.Net Core 的 DI 来实现）：

```cs
public class InventoryContext : IInventoryContext
{
    public InventoryContext()
    {
       _books = new ConcurrentDictionary<string, Book>();
    }

    private readonly static object _lock = new object(); 

    private readonly IDictionary<string, Book> _books;

    public Book[] GetBooks()
    {
        return _books.Values.ToArray();
    }

    ...
}
```

`AddBook`和`UpdateQuantity`方法的详细信息如下所示：

```cs
public bool AddBook(string name)
{
    _books.Add(name, new Book {Name = name});
    return true;
}

public bool UpdateQuantity(string name, int quantity)
{
    lock (_lock)
    {
        _books[name].Quantity += quantity;
    }

    return true;
}
```

有几件事情需要注意。构造函数已从受保护更改为公共。这将允许类在类外部被实例化。还要注意，静态`Instance`属性和私有静态`_instance`字段已被删除，而私有`_lock`字段仍然存在。与上一章中定义的 Singleton 模式类似，这只保证了类的实例化方式；它并不阻止方法被并行访问。

`IInventoryContext`接口和`InventoryContext`和`Book`类都被设为公共，因为我们的 DI 是在外部项目中定义的。

随后，用于返回命令的`InventoryCommandFactory`类已更新，以便在其构造函数中注入`InventoryContext`的实例：

```cs
public class InventoryCommandFactory : IInventoryCommandFactory
{
    private readonly IUserInterface _userInterface;
    private readonly IInventoryContext _context;

    public InventoryCommandFactory(IUserInterface userInterface, IInventoryContext context)
    {
        _userInterface = userInterface;
        _context = context;
    }

    // GetCommand()
    ...
}
```

`GetCommand`方法使用提供的输入来确定特定的命令：

```cs
public InventoryCommand GetCommand(string input)
{
    switch (input.ToLower())
    {
        case "q":
        case "quit":
            return new QuitCommand(_userInterface);
        case "a":
        case "addinventory":
            return new AddInventoryCommand(_userInterface, _context);
        case "g":
        case "getinventory":
            return new GetInventoryCommand(_userInterface, _context);
        case "u":
        case "updatequantity":
            return new UpdateQuantityCommand(_userInterface, _context);
        case "?":
            return new HelpCommand(_userInterface);
        default:
            return new UnknownCommand(_userInterface);
    }
}
```

如前所述，`IInventoryContext`接口现在将由客户端项目中定义的 DI 容器提供。控制台应用程序现在有一个额外的行来使用`InventoryContext`类创建`IInventoryContext`接口的单例：

```cs
class Program
{
    private static void Main(string[] args)
    {
        IServiceCollection services = new ServiceCollection();
        ConfigureServices(services);
        IServiceProvider serviceProvider = services.BuildServiceProvider();

        var service = serviceProvider.GetService<ICatalogService>();
        service.Run();

        Console.WriteLine("CatalogService has completed.");
        Console.ReadLine();
    }

    private static void ConfigureServices(IServiceCollection services)
    {
        // Add application services.
        services.AddTransient<IUserInterface, ConsoleUserInterface>(); 
        services.AddTransient<ICatalogService, CatalogService>();
        services.AddTransient<IInventoryCommandFactory, InventoryCommandFactory>();

 services.AddSingleton<IInventoryContext, InventoryContext>();
    }
}
```

控制台应用程序现在可以使用与上一章中执行的手动测试相同的方式运行，但是单元测试是了解使用.Net Core 的 DI 实现的成果的好方法。

本章提供的示例代码显示了完成的项目。接下来的部分集中在`InventoryContext`测试上。`InventoryCommandFactory`测试也进行了修改，但由于更改是微不足道的，因此不会在此处进行介绍。

# 单元测试

随着对`InventoryContext`类的更改，我们不再有一个方便的属性来获取该类的唯一实例。这意味着`InventoryContext.Instance`需要被替换，首先，让我们创建一个方法来返回`InventoryContext`的新实例，并使用`GetInventoryContext()`代替`InventoryContext.Instance`：

```cs
private IInventoryContext GetInventoryContext()
{
    return new InventoryContext();
}
```

如预期的那样，单元测试失败，并显示错误消息：*给定的键在字典中不存在*：

![](img/270f856b-e1dc-475a-8a82-22e9288276cb.png)

正如我们在上一章中看到的，这是因为每次创建`InventoryContext`类时，书籍的列表都是空的。这就是为什么我们需要使用 Singleton 创建一个上下文的原因。

让我们更新`GetInventoryContext()`方法，现在使用.Net Core 的 DI 来提供`IInventoryContext`接口的实例：

```cs
private IInventoryContext GetInventoryContext()
{
    IServiceCollection services = new ServiceCollection();
    services.AddSingleton<IInventoryContext, InventoryContext>();
    var provider = services.BuildServiceProvider();

    return provider.GetService<IInventoryContext>();
}
```

在更新的方法中，创建了`ServiceCollection`类的一个实例，用于包含所有注册的依赖项。`InventoryContext`类被注册为 Singleton，以便在请求`IInventoryContext`依赖项时提供。然后生成了一个`ServiceProvider`实例，它将根据`IServiceCollection`接口中的注册执行 DI。最后一步是在请求`IInventoryContext`接口时提供`InventoryContext`类。

`Microsoft.Extensions.DependencyInjection`库需要添加到`InventoryManagementTests`项目中，以便能够引用.Net Core DI 组件。

很不幸，单元测试仍然无法通过，并且导致相同的错误：*给定的键在字典中不存在。*这是因为每次请求`IInventoryContext`时，我们都会创建一个新的 DI 框架实例。这意味着，即使我们的依赖是一个 Singleton，每个`ServiceProvider`实例都会提供一个新的`InventoryContext`类的实例。为了解决这个问题，我们将在测试启动时创建`IServiceCollection`，然后在测试期间使用相同的引用：

```cs
ServiceProvider Services { get; set; }

[TestInitialize]
public void Startup()
{
    IServiceCollection services = new ServiceCollection();
    services.AddSingleton<IInventoryContext, InventoryContext>();
    Services = services.BuildServiceProvider();
}
```

使用`TestInitialize`属性是在`TestClass`类中分离多个`TestMethod`测试所需的功能的好方法。该方法将在每次测试运行之前运行。

现在有了对同一个`ServiceProvider`实例的引用，我们可以更新以检索依赖项。以下说明了`AddBook()`方法的更新方式：

```cs
public Task AddBook(string book)
{
    return Task.Run(() =>
    {
        Assert.IsTrue(Services.GetService<IInventoryContext>().AddBook(book));
    });
}
```

我们的单元测试现在成功通过，因为在测试执行期间只创建了一个`InventoryContext`类的实例：

![](img/1c92dd81-8d8c-466a-982e-ab7862dcab15.png)

使用内置的 DI 相对容易实现 Singleton 模式，就像本节中所示。了解何时使用该模式是一个重要的概念。下一节将更详细地探讨作用域的概念，以便更深入地理解服务的生命周期。

# 作用域

在同时执行多个进程的应用程序中，了解服务生命周期对功能和非功能需求都非常重要。正如在上一个单元测试中所示，如果没有正确的服务生命周期，`InventoryContext`就无法按预期工作，并导致了一个无效的情况。同样，错误使用服务生命周期可能导致应用程序无法良好扩展。一般来说，在多进程解决方案中应避免使用锁和共享状态。 

为了说明这个概念，想象一下 FlixOne 库存管理应用程序被提供给多个员工。现在的挑战是如何在多个应用程序之间执行锁定，以及如何拥有一个单一的收集状态。在我们的术语中，这将是多个应用程序共享的单个`InventoryContext`类。当然，这就是我们改变解决方案以使用共享存储库（例如数据库）或改变解决方案以使用 Web 应用程序的地方。我们将在后面的章节中涵盖数据库和 Web 应用程序模式，但是，由于我们正在讨论服务生命周期，现在更详细地描述这些内容是有意义的。

以下图示了一个 Web 应用程序接收两个请求：

![](img/a28f0bf1-4c7f-43ab-9cc4-3de7694fdf4c.png)

在服务生命周期方面，单例服务生命周期将对两个请求都可用，而每个请求都会接收到自己的作用域生命周期。需要注意的重要事情是垃圾回收。使用瞬态服务生命周期创建的依赖项在对象不再被引用时标记为释放，而使用作用域服务生命周期创建的依赖项在 Web 请求完成之前不会被标记为释放。而使用单例服务生命周期创建的依赖项直到应用程序结束才会被标记为释放。

此外，如下图所示，重要的是要记住，在.Net Core 中，依赖项在 Web 园或 Web 农场中的服务器实例之间不共享：

![](img/13d3bc5b-0273-488b-9db0-f87daf198604.png)

在接下来的章节中，将展示共享状态的不同方法，包括使用共享缓存、数据库和其他形式的存储库。

# 实现工厂

.Net Core DI 支持在注册依赖项时指定*实现工厂*的能力。这允许对由提供的服务提供的依赖项的创建进行控制。在注册时使用`IServiceCollection`接口的以下扩展来完成：

```cs
public static IServiceCollection AddSingleton<TService, TImplementation>(this IServiceCollection services,     Func<IServiceProvider, TImplementation> implementationFactory)
                where TService : class
                where TImplementation : class, TService;
```

`AddSingleton`扩展接收要注册的类以及在需要依赖项时要提供的类。值得注意的是，.Net Core DI 框架将维护已注册的服务，并在请求时提供实现，或作为依赖项之一的实例化的一部分。这种自动实例化称为**构造函数注入**（**CI**）。我们将在以下章节中看到这两种的例子。

# IInventoryContext

举个例子，让我们重新审视一下用于管理书籍库存的`InventoryContext`类，通过将对书籍集合的读取和写入操作进行分离。`IInventoryContext`被分成了`IInventoryReadContext`和`IInventoryWriteContext`：

```cs
using FlixOne.InventoryManagement.Models;

namespace FlixOne.InventoryManagement.Repository
{
    public interface IInventoryContext : IInventoryReadContext, IInventoryWriteContext { }

    public interface IInventoryReadContext
    {
        Book[] GetBooks();
    }

    public interface IInventoryWriteContext
    {
        bool AddBook(string name);
        bool UpdateQuantity(string name, int quantity);
    }
}
```

# IInventoryReadContext

`IInventoryReadContext`接口包含读取书籍的操作，而`IInventoryWriteContext`包含修改书籍集合的操作。最初创建`IInventoryContext`接口是为了方便一个类需要两种依赖类型时。

在后面的章节中，我们将涵盖利用分割上下文的模式，包括**命令和** **查询责任分离**（**CQRS**）模式。

通过这种重构，需要进行一些更改。首先，只需要读取书籍集合的类将其构造函数更新为`IInventoryReadContext`接口，如`GetInventoryCommand`类所示：

```cs
internal class GetInventoryCommand : NonTerminatingCommand
{
    private readonly IInventoryReadContext _context;
    internal GetInventoryCommand(IUserInterface userInterface, IInventoryReadContext context) : base(userInterface)
    {
        _context = context;
    }

    protected override bool InternalCommand()
    {
        foreach (var book in _context.GetBooks())
        {
            Interface.WriteMessage($"{book.Name,-30}\tQuantity:{book.Quantity}"); 
        }

        return true;
    }
}
```

# IInventoryWriteContext

同样，需要修改书籍集合的类将其更新为`IInventoryWriteContext`接口，如`AddInventoryCommand`所示：

```cs
internal class AddInventoryCommand : NonTerminatingCommand, IParameterisedCommand
{
    private readonly IInventoryWriteContext _context;

    internal AddInventoryCommand(IUserInterface userInterface, IInventoryWriteContext context) : base(userInterface)
    {
        _context = context;
    }

    public string InventoryName { get; private set; }

    ...
}
```

以下显示了`GetParameters`和`InternalCommand`方法的详细信息：

```cs
/// <summary>
/// AddInventoryCommand requires name
/// </summary>
/// <returns></returns>
public bool GetParameters()
{
    if (string.IsNullOrWhiteSpace(InventoryName))
        InventoryName = GetParameter("name");
    return !string.IsNullOrWhiteSpace(InventoryName);
}

protected override bool InternalCommand()
{
    return _context.AddBook(InventoryName); 
}
```

请注意 `InternalCommand` 方法，其中将带有 `InventoryName` 参数中保存的书名添加到库存中。

接下来，我们将看看库存命令的工厂。

# InventoryCommandFactory

`InventoryCommandFactory` 类是使用 .Net 类实现工厂模式的一个实现，需要对书籍集合进行读取和写入：

```cs
public class InventoryCommandFactory : IInventoryCommandFactory
{
    private readonly IUserInterface _userInterface;
    private readonly IInventoryContext _context; 

    public InventoryCommandFactory(IUserInterface userInterface, IInventoryContext context)
    {
        _userInterface = userInterface;
        _context = context; 
    }

    public InventoryCommand GetCommand(string input)
    {
        switch (input.ToLower())
        {
            case "q":
            case "quit":
                return new QuitCommand(_userInterface);
            case "a":
            case "addinventory":
                return new AddInventoryCommand(_userInterface, _context);
            case "g":
            case "getinventory":
                return new GetInventoryCommand(_userInterface, _context);
            case "u":
            case "updatequantity":
                return new UpdateQuantityCommand(_userInterface, _context);
            case "?":
                return new HelpCommand(_userInterface);
            default:
                return new UnknownCommand(_userInterface);
        }
    }
}
```

值得注意的是，这个类实际上不需要修改前一章版本的内容，因为多态性处理了从 `IInventoryContext` 到 `IInventoryReadContext` 和 `IInventoryWriteContext` 接口的转换。

有了这些变化，我们需要改变与 `InventoryContext` 相关的依赖项的注册，以使用实现工厂：

```cs
private static void ConfigureServices(IServiceCollection services)
{
    // Add application services.
    ...            

    var context = new InventoryContext();
 services.AddSingleton<IInventoryReadContext, InventoryContext>(p => context);
 services.AddSingleton<IInventoryWriteContext, InventoryContext>(p => context);
 services.AddSingleton<IInventoryContext, InventoryContext>(p => context);
}
```

对于所有三个接口，将使用相同的 `InventoryContext` 实例，并且这是使用实现工厂扩展一次实例化的。当请求 `IInventoryReadContext`、`IInventoryWriteContext` 或 `IInventoryContext` 依赖项时提供。

# InventoryCommand

`InventoryCommandFactory` 在展示如何使用 .Net 实现工厂模式时非常有用，但现在让我们重新审视一下，因为我们现在正在使用 .Net Core 框架。我们的要求是给定一个字符串值；我们希望返回 `InventoryCommand` 的特定实现。这可以通过几种方式实现，在本节中将给出三个示例：

+   使用函数的实现工厂

+   使用服务

+   使用第三方容器

# 使用函数的实现工厂

`GetService()` 方法的实现工厂可以用于确定要返回的 `InventoryCommand` 类型。对于这个示例，在 `InventoryCommand` 类中创建了一个新的静态方法：

```cs
public static Func<IServiceProvider, Func<string, InventoryCommand>> GetInventoryCommand => 
                                                                            provider => input =>
{
    switch (input.ToLower())
    {
        case "q":
        case "quit":
            return new QuitCommand(provider.GetService<IUserInterface>());
        case "a":
        case "addinventory":
            return new AddInventoryCommand(provider.GetService<IUserInterface>(), provider.GetService<IInventoryWriteContext>());
        case "g":
        case "getinventory":
            return new GetInventoryCommand(provider.GetService<IUserInterface>(), provider.GetService<IInventoryReadContext>());
        case "u":
        case "updatequantity":
            return new UpdateQuantityCommand(provider.GetService<IUserInterface>(), provider.GetService<IInventoryWriteContext>());
        case "?":
            return new HelpCommand(provider.GetService<IUserInterface>());
        default:
            return new UnknownCommand(provider.GetService<IUserInterface>());
    }
};
```

如果您不熟悉 lambda 表达式体，这可能有点难以阅读，因此我们将详细解释一下代码。首先，让我们重新审视一下 `AddSingleton` 的语法：

```cs
public static IServiceCollection AddSingleton<TService, TImplementation>(this IServiceCollection services, Func<IServiceProvider, TImplementation> implementationFactory)
            where TService : class
            where TImplementation : class, TService;
```

这表明 `AddSingleton` 扩展的参数是一个函数：

```cs
Func<IServiceProvider, TImplementation> implementationFactory
```

这意味着以下代码是等价的：

```cs
services.AddSingleton<IInventoryContext, InventoryContext>(provider => new InventoryContext());

services.AddSingleton<IInventoryContext, InventoryContext>(GetInventoryContext);
```

`GetInventoryContext` 方法定义如下：

```cs
static Func<IServiceProvider, InventoryContext> GetInventoryContext => provider =>
{
    return new InventoryContext();
};
```

在我们的特定示例中，特定的 `InventoryCommand` 类型已被标记为 `FlixOne.InventoryManagement` 项目的内部，因此 `FlixOne.InventoryManagementClient` 项目无法直接访问它们。这就是为什么在 `FlixOne.InventoryManagement.InventoryCommand` 类中创建了一个新的静态方法，返回以下类型：

```cs
Func<IServiceProvider, Func<string, InventoryCommand>>
```

这意味着当请求服务时，将提供一个字符串来确定具体的类型。由于依赖项发生了变化，这意味着 `CatalogService` 构造函数需要更新：

```cs
public CatalogService(IUserInterface userInterface, Func<string, InventoryCommand> commandFactory)
{
    _userInterface = userInterface;
    _commandFactory = commandFactory;
}
```

当请求服务时，将提供一个字符串来确定具体的类型。由于依赖项发生了变化，`CatalogueService` 构造函数需要更新：

现在，当用户输入的字符串被提供给 `CommandFactory` 依赖项时，将提供正确的命令：

```cs
while (!response.shouldQuit)
{
    // look at this mistake with the ToLower()
    var input = _userInterface.ReadValue("> ").ToLower();
    var command = _commandFactory(input);

    response = command.RunCommand();

    if (!response.wasSuccessful)
    {
        _userInterface.WriteMessage("Enter ? to view options.");
    }
}
```

与命令工厂相关的单元测试也进行了更新。作为对比，从现有的 `InventoryCommandFactoryTests` 类创建了一个新的 `test` 类，并命名为 `InventoryCommandFunctionTests`。初始化步骤如下所示，其中突出显示了更改：

```cs
ServiceProvider Services { get; set; }

[TestInitialize]
public void Startup()
{
    var expectedInterface = new Helpers.TestUserInterface(
        new List<Tuple<string, string>>(),
        new List<string>(),
        new List<string>()
    );

    IServiceCollection services = new ServiceCollection();
    services.AddSingleton<IInventoryContext, InventoryContext>();
 services.AddTransient<Func<string, InventoryCommand>>(InventoryCommand.GetInventoryCommand);

    Services = services.BuildServiceProvider();
}
```

还更新了各个测试，以在 `QuitCommand` 中提供字符串作为获取服务调用的一部分，如下所示：

```cs
[TestMethod]
public void QuitCommand_Successful()
{
    Assert.IsInstanceOfType(Services.GetService<Func<string, InventoryCommand>>().Invoke("q"),             
                            typeof(QuitCommand), 
                            "q should be QuitCommand");

    Assert.IsInstanceOfType(Services.GetService<Func<string, InventoryCommand>>().Invoke("quit"),
                            typeof(QuitCommand), 
                            "quit should be QuitCommand");
}
```

这两个测试验证了当服务提供程序提供 `"q"` 或 `"quit"` 时，返回的服务类型是 `QuitCommand`。

# 使用服务

`ServiceProvider`类提供了一个`Services`方法，可以用来确定适当的服务，当同一类型有多个依赖项注册时。这个例子将采用不同的方法处理`InventoryCommands`，由于重构的范围，这将通过新创建的类来完成，以说明这种方法。

在单元测试项目中，创建了一个新的文件夹`ImplementationFactoryTests`，用于包含本节的类。在这个文件夹中，创建了一个新的`InventoryCommand`基类：

```cs
public abstract class InventoryCommand
{
    protected abstract string[] CommandStrings { get; }
    public virtual bool IsCommandFor(string input)
    {
        return CommandStrings.Contains(input.ToLower());
    } 
}
```

这个新类背后的概念是，子类将定义它们要响应的字符串。例如，`QuitCommand`将响应`"q"`和`"quit"`字符串：

```cs
public class QuitCommand : InventoryCommand
{
    protected override string[] CommandStrings => new[] { "q", "quit" };
}
```

以下显示了`GetInventoryCommand`、`AddInventoryCommand`、`UpdateQuantityCommand`和`HelpCommand`类，它们采用了类似的方法：

```cs
public class GetInventoryCommand : InventoryCommand
{
    protected override string[] CommandStrings => new[] { "g", "getinventory" };
}

public class AddInventoryCommand : InventoryCommand
{
    protected override string[] CommandStrings => new[] { "a", "addinventory" };
}

public class UpdateQuantityCommand : InventoryCommand
{
    protected override string[] CommandStrings => new[] { "u", "updatequantity" };
}

public class HelpCommand : InventoryCommand
{
    protected override string[] CommandStrings => new[] { "?" };
}
```

然而，`UnknownCommand`类将被用作默认值，因此它将始终通过重写`IsCommandFor`方法来评估为 true：

```cs
public class UnknownCommand : InventoryCommand
{
    protected override string[] CommandStrings => new string[0];

    public override bool IsCommandFor(string input)
    {
        return true;
    }
}
```

由于`UnknownCommand`类被视为默认值，注册的顺序很重要，在单元测试类的初始化中如下所示：

```cs
[TestInitialize]
public void Startup()
{
    var expectedInterface = new Helpers.TestUserInterface(
        new List<Tuple<string, string>>(),
        new List<string>(),
        new List<string>()
    );

    IServiceCollection services = new ServiceCollection(); 
    services.AddTransient<InventoryCommand, QuitCommand>();
    services.AddTransient<InventoryCommand, HelpCommand>(); 
    services.AddTransient<InventoryCommand, AddInventoryCommand>();
    services.AddTransient<InventoryCommand, GetInventoryCommand>();
    services.AddTransient<InventoryCommand, UpdateQuantityCommand>();
    // UnknownCommand should be the last registered
 services.AddTransient<InventoryCommand, UnknownCommand>();

    Services = services.BuildServiceProvider();
}
```

为了方便起见，创建了一个新的方法，以便在给定匹配输入字符串时返回`InventoryCommand`类的实例：

```cs
public InventoryCommand GetCommand(string input)
{
    return Services.GetServices<InventoryCommand>().First(svc => svc.IsCommandFor(input));
}
```

这个方法将遍历为`InventoryCommand`服务注册的依赖项集合，直到使用`IsCommandFor()`方法找到匹配项。

然后，单元测试使用`GetCommand()`方法来确定依赖项，如下所示，用于`UpdateQuantityCommand`：

```cs
[TestMethod]
public void UpdateQuantityCommand_Successful()
{
    Assert.IsInstanceOfType(GetCommand("u"), 
                            typeof(UpdateQuantityCommand), 
                            "u should be UpdateQuantityCommand");

    Assert.IsInstanceOfType(GetCommand("updatequantity"), 
                            typeof(UpdateQuantityCommand), 
                            "updatequantity should be UpdateQuantityCommand");

    Assert.IsInstanceOfType(GetCommand("UpdaTEQuantity"), 
                            typeof(UpdateQuantityCommand), 
                            "UpdaTEQuantity should be UpdateQuantityCommand");
}
```

# 使用第三方容器

.Net Core 框架提供了很大的灵活性和功能，但可能不支持一些功能，第三方容器可能是更合适的选择。幸运的是，.Net Core 是可扩展的，允许用第三方容器替换内置的服务容器。为了举例，我们将使用`Autofac`作为.Net Core DI 的 IoC 容器。

`Autofac`有很多很棒的功能，在这里作为一个例子展示出来；当然，还有其他 IoC 容器可以使用。例如，Castle Windsor 和 Unit 都是很好的替代方案，也应该考虑使用。

第一步是将所需的`Autofac`包添加到项目中。使用包管理器控制台，使用以下命令添加包（仅在测试项目中需要）：

```cs
install-package autofac
```

这个例子将再次通过使用`Autofac`的命名注册依赖项的功能来支持我们的`InventoryCommand`工厂。这些命名的依赖项将用于根据提供的输入来检索正确的`InventoryCommand`实例。

与之前的例子类似，依赖项的注册将在`TestInitialize`方法中完成。注册将根据将用于确定命令的命令命名。以下显示了创建`ContainerBuilder`对象的`Startup`方法结构，该对象将构建`Container`实例：

```cs
[TestInitialize]
public void Startup()
{
    IServiceCollection services = new ServiceCollection();

    var builder = new ContainerBuilder(); 

    // commands
    ...

    Container = builder.Build(); 
}
```

命令的注册如下：

```cs
// commands
builder.RegisterType<QuitCommand>().Named<InventoryCommand>("q");
builder.RegisterType<QuitCommand>().Named<InventoryCommand>("quit");
builder.RegisterType<UpdateQuantityCommand>().Named<InventoryCommand>("u");
builder.RegisterType<UpdateQuantityCommand>().Named<InventoryCommand>("updatequantity");
builder.RegisterType<HelpCommand>().Named<InventoryCommand>("?");
builder.RegisterType<AddInventoryCommand>().Named<InventoryCommand>("a");
builder.RegisterType<AddInventoryCommand>().Named<InventoryCommand>("addinventory");
builder.RegisterType<GetInventoryCommand>().Named<InventoryCommand>("g");
builder.RegisterType<GetInventoryCommand>().Named<InventoryCommand>("getinventory");
builder.RegisterType<UpdateQuantityCommand>().Named<InventoryCommand>("u");
builder.RegisterType<UpdateQuantityCommand>().Named<InventoryCommand>("u");
builder.RegisterType<UnknownCommand>().As<InventoryCommand>();
```

与之前的例子不同，生成的容器是`Autofac.IContainer`的实例。这将用于检索每个注册的依赖项。例如，`QuitCommand`将被命名为`"q"`和`"quit"`，这表示可以用于执行命令的两个命令。另外，注意最后注册的类型没有命名，并属于`UnknownCommand`。如果没有找到命令，则这将充当默认值。

为了确定一个依赖项，将使用一个新方法来按名称检索依赖项：

```cs
public InventoryCommand GetCommand(string input)
{
    return Container.ResolveOptionalNamed<InventoryCommand>(input.ToLower()) ?? 
           Container.Resolve<InventoryCommand>();
}
```

`Autofac.IContainer`接口具有`ResolveOptionalNamed<*T*>(*string*)`方法名称，该方法将返回具有给定名称的依赖项，如果找不到匹配的注册，则返回 null。如果未使用给定名称注册依赖项，则将返回`UnknownCommand`类的实例。这是通过使用空值合并操作`??`和`IContainer.Resolve<*T*>`方法来实现的。

如果依赖项解析失败，`Autofac.IContainer.ResolveNamed<*T*>(*string*)`将抛出`ComponentNotRegisteredException`异常。

为了确保正确解析命令，为每个命令编写了一个测试方法。再次以`QuitCommand`为例，我们可以看到以下内容：

```cs
[TestMethod]
public void QuitCommand_Successful()
{
    Assert.IsInstanceOfType(GetCommand("q"), typeof(QuitCommand), "q should be QuitCommand");
    Assert.IsInstanceOfType(GetCommand("quit"), typeof(QuitCommand), "quit should be QuitCommand");
}
```

请查看源代码中的`InventoryCommandAutofacTests`类，以获取其他`InventoryCommand`示例。

# 总结

本章的目标是更详细地探索.Net Core 框架，特别是.Net Core DI。支持三种类型的服务生命周期：瞬态（Transient）、作用域（Scoped）和单例（Singleton）。瞬态服务将为每个请求创建一个已注册依赖项的新实例。作用域服务将在定义的范围内生成一次，而单例服务将在 DI 服务集合的生命周期内执行一次。

由于.Net Core DI 对于自信地构建.Net Core 应用程序至关重要，因此了解其能力和局限性非常重要。重要的是要有效地使用 DI，同时避免重复使用已提供的功能。同样重要的是，了解.Net Core DI 框架的限制，以及其他 DI 框架的优势，以便在替换基本的.Net Core DI 框架为第三方 DI 框架可能对应用程序有益的情况下，能够明智地做出选择。

下一章将在前几章的基础上构建，并探索.Net Core ASP.Net Web 应用程序中的常见模式。

# 问题

以下问题将帮助您巩固本章中包含的信息：

1.  如果不确定要使用哪种类型的服务生命周期，最好将类注册为哪种类型？为什么？

1.  在.Net Core ASP.Net 解决方案中，作用域是按照每个 web 请求定义的，还是按照每个会话定义的？

1.  在.Net Core DI 框架中将类注册为单例是否会使其线程安全？

1.  .Net Core DI 框架只能被其他由微软提供的 DI 框架替换吗？
