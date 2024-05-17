# 第四章：实施设计模式 - 基础知识第二部分

在上一章中，我们介绍了 FlixOne 以及新库存管理应用程序的初始开发。开发团队使用了多种模式，从旨在限制交付范围的模式（如**最小可行产品**（**MVP**））到辅助项目开发的模式（如**测试驱动开发**（**TDD**））。还应用了**四人帮**（**GoF**）的几种模式，作为解决方案，以利用他人过去解决类似问题的经验，以免重复常见错误。应用了单一责任原则、开闭原则、里氏替换原则、接口隔离原则和依赖反转原则（SOLID 原则），以确保我们正在创建一个稳定的代码库，将有助于管理和未来开发我们的应用程序。

本章将继续解释通过合并更多模式来构建 FlixOne 库存管理应用程序。将使用更多的 GoF 模式，包括单例模式和工厂模式。将使用单例模式来说明用于维护 FlixOne 图书收藏的存储库模式。工厂模式将进一步理解**依赖注入**（**DI**）。最后，我们将使用.NET Core 框架来促进**控制反转**（**IoC**）容器，该容器将用于完成初始库存管理控制台应用程序。

本章将涵盖以下主题：

+   单例模式

+   工厂模式

+   .NET Core 的特性

+   控制台应用程序

# 技术要求

本章包含各种代码示例，以解释这些概念。代码保持简单，仅用于演示目的。大多数示例涉及使用 C#编写的.NET Core 控制台应用程序。

要运行和执行代码，您需要以下内容：

+   Visual Studio 2019（您也可以使用 Visual Studio 2017 版本 3 或更高版本运行应用程序）

+   .NET Core

+   SQL Server（本章使用 Express Edition）

# 安装 Visual Studio

要运行这些代码示例，您需要安装 Visual Studio 或更高版本。您可以使用您喜欢的集成开发环境。要做到这一点，请按照以下说明进行操作：

1.  从以下链接下载 Visual Studio：[`docs.microsoft.com/en-us/visualstudio/install/install-visual-studio`](https://docs.microsoft.com/en-us/visualstudio/install/install-visual-studio)。

1.  按照包含的安装说明进行操作。安装 Visual Studio 有多个版本可供选择；在本章中，我们使用的是 Windows 版的 Visual Studio。

# .NET Core 的设置

如果您尚未安装.NET Core，则需要按照以下说明进行操作：

1.  从以下链接下载.NET Core：[`www.microsoft.com/net/download/windows`](https://www.microsoft.com/net/download/windows)。

1.  按照相关库的安装说明进行操作：[`dotnet.microsoft.com/download/dotnet-core/2.2`](https://dotnet.microsoft.com/download/dotnet-core/2.2)。

完整的源代码可在 GitHub 上找到。本章中显示的源代码可能不完整，因此建议您检索源代码以运行示例（[`github.com/PacktPublishing/Hands-On-Design-Patterns-with-C-and-.NET-Core/tree/master/Chapter4`](https://github.com/PacktPublishing/Hands-On-Design-Patterns-with-C-and-.NET-Core/tree/master/Chapter4)）。

# 单例模式

单例模式是另一个 GoF 设计模式，用于限制类的实例化为一个对象。它用于需要协调系统内的操作或限制对数据的访问的情况。例如，如果需要在应用程序内将对文件的访问限制为单个写入者，则可以使用单例模式防止多个对象同时尝试向文件写入。在我们的场景中，我们将使用单例模式来维护书籍及其库存的集合。

单例模式的价值在使用示例时更加明显。本节将从一个基本类开始，然后继续识别单例模式所解决的不同问题。这些问题将被识别出来，然后通过单元测试进行更新和验证。

单例模式应仅在必要时使用，因为它可能为应用程序引入潜在的瓶颈。有时，该模式被视为反模式，因为它引入了全局状态。全局状态会引入应用程序中的未知依赖关系，因此不清楚有多少类型可能依赖于该信息。此外，许多框架和存储库已经在需要时限制了访问，因此引入额外的机制可能会不必要地限制性能。

.NET Core 支持许多讨论的模式。在下一章中，我们将利用`ServiceCollection`类对工厂方法和单例模式的支持。

在我们的场景中，单例模式将用于保存包含书籍集合的内存存储库。单例将防止多个线程同时更新书籍集合。这将要求我们*锁定*代码的一部分，以防止不可预测的更新。

将单例引入应用程序的复杂性可能是微妙的；因此，为了对该模式有一个坚实的理解，我们将涵盖以下主题：

+   .Net Framework 对进程和线程的处理

+   存储库模式

+   竞争条件

+   单元测试以识别竞争条件

# 进程和线程

要理解单例模式，我们需要提供一些背景。在.Net Framework 中，一个应用程序将由称为应用程序域的轻量级托管子进程组成，这些子进程可以包含一个或多个托管线程。为了理解单例模式，让我们将其定义为包含一个或多个同时运行的线程的多线程应用程序。从技术上讲，这些线程实际上并不是同时运行的，而是通过在线程之间分配可用处理器时间来实现的，因此每个线程将执行一小段时间，然后该线程将暂停活动，从而允许另一个线程执行。

回到单例模式，在多线程应用程序中，需要特别注意确保对单例的访问受限，以便只有一个线程同时进入特定逻辑区域。由于线程的同步，一个线程可以检索值并更新它，然后在存储之前，另一个线程也更新该值。

多个线程可能访问相同的共享数据并以不可预测的结果进行更新，这可能被称为**竞争条件**。

为了避免数据被错误更新，需要一些限制，以防止多个线程同时执行相同的逻辑块。在.Net Framework 中支持几种机制，在单例模式中，使用`lock`关键字。在下面的代码中，演示了`lock`关键字，以表明一次只有一个线程可以执行突出显示的代码，而所有其他线程将被阻塞：

```cs
public class Inventory
{
   int _quantity;
    private Object _lock = new Object();

    public void RemoveQuantity(int amount)
    {
        lock (_lock)
        {
            if (_quantity - amount < 0)
 {
 throw new Exception("Cannot remove more than we have!");
 }
 _quantity -= amount;
        }
    }
}
```

锁是限制代码段访问的简单方法，可以应用于对象实例，就像我们之前的例子所示的那样，也可以应用于标记为静态的代码段。

# 存储库模式

引入到项目中的单例模式应用于用于维护库存书籍集合的类。单例将防止多个线程访问被错误处理，另一个模式存储库模式将用于创建一个外观，用于管理的数据。

存储库模式提供了一个存储库的抽象，以在应用程序的业务逻辑和底层数据之间提供一层。这提供了几个优势。通过进行清晰的分离，我们的业务逻辑可以独立于底层数据进行维护和单元测试。通常，相同的存储库模式类可以被多个业务对象重用。一个例子是`GetInventoryCommand`、`AddInventoryCommand`和`UpdateInventoryCommand`对象；所有这些对象都使用相同的存储库类。这使我们能够在不受存储库影响的情况下测试这些命令中的逻辑。该模式的另一个优势是，它使得更容易实现集中的数据相关策略，比如缓存。

首先，让我们考虑以下描述存储库将实现的方法的接口；它包含了检索书籍、添加书籍和更新书籍数量的方法：

```cs
internal interface IInventoryContext
{
    Book[] GetBooks();
    bool AddBook(string name);
    bool UpdateQuantity(string name, int quantity);
}
```

存储库的初始版本如下：

```cs
internal class InventoryContext : IInventoryContext
{ 
    public InventoryContext()
    {
        _books = new Dictionary<string, Book>();
    }

    private readonly IDictionary<string, Book> _books; 

    public Book[] GetBooks()
    {
        return _books.Values.ToArray();
    }

    public bool AddBook(string name)
    {
        _books.Add(name, new Book { Name = name });
        return true;
    }

    public bool UpdateQuantity(string name, int quantity)
    {
        _books[name].Quantity += quantity;
        return true;
    }
}
```

在本章中，书籍集合以内存缓存的形式进行维护，而在后续章节中，这将被移动到提供持久数据的存储库中。当然，这种实现并不理想，因为一旦应用程序结束，所有数据都将丢失。但是，它用来说明单例模式。

# 单元测试

为了说明单例模式解决的问题，让我们从一个简单的单元测试开始，向存储库添加 30 本书，更新不同书籍的数量，然后验证结果。以下代码显示了整体单元测试，我们将逐个解释每个步骤：

```cs
 [TestClass]
public class InventoryContextTests
{ 
    [TestMethod]
    public void MaintainBooks_Successful()
    { 
        var context = new InventoryContext();

        // add thirty books
        ...

        // let's update the quantity of the books by adding 1, 2, 3, 4, 5 ...
        ...

        // let's update the quantity of the books by subtracting 1, 2, 3, 4, 5 ...
        ...

        // all quantities should be 0
        ...
    } 
}
```

为了添加 30 本书，使用`context`实例从`Book_1`到`Book_30`添加书籍：

```cs
        // add thirty books
        foreach(var id in Enumerable.Range(1, 30))
        {
            context.AddBook($"Book_{id}"); 
        }
```

接下来的部分通过将数字`1`到`10`添加到每本书的数量来更新书籍数量：

```cs
        // let's update the quantity of the books by adding 1, 2, 3, 4, 5 ...
        foreach (var quantity in Enumerable.Range(1, 10))
        {
            foreach (var id in Enumerable.Range(1, 30))
            {
                context.UpdateQuantity($"Book_{id}", quantity);
            }
        }
```

然后，在下一部分，我们将从每本书的数量中减去数字`1`到`10`：

```cs
        foreach (var quantity in Enumerable.Range(1, 10))
        {
            foreach (var id in Enumerable.Range(1, 30))
            {
                context.UpdateQuantity($"Book_{id}", -quantity);
            }
        }
```

由于我们为每本书添加和移除了相同的数量，所以我们测试的最后部分将验证最终数量是否为`0`：

```cs
        // all quantities should be 0
        foreach (var book in context.GetBooks())
        {
            Assert.AreEqual(0, book.Quantity);
        }
```

运行测试后，我们可以看到测试通过了：

![](img/9597d7f3-ef49-419f-b295-077228d9471e.png)

因此，当测试在单个进程中运行时，存储库将按预期工作。但是，如果更新请求在单独的线程中执行会怎样呢？为了测试这一点，单元测试将被重构为在单独的线程中对`InventoryContext`类进行调用。

书籍的添加被移动到一个执行添加书籍的方法中（即在自己的线程中）：

```cs
public Task AddBook(string book)
{
    return Task.Run(() =>
    {
        var context = new InventoryContext();
        Assert.IsTrue(context.AddBook(book));
    });
}
```

此外，更新数量步骤被移动到另一个具有类似方法的方法中：

```cs
public Task UpdateQuantity(string book, int quantity)
{
    return Task.Run(() =>
    {
        var context = new InventoryContext();
        Assert.IsTrue(context.UpdateQuantity(book, quantity));
    });
}
```

然后更新单元测试以调用新方法。值得注意的是，单元测试将等待所有书籍添加完成后再更新数量。

`添加三十本书`部分现在如下所示：

```cs
    // add thirty books
    foreach (var id in Enumerable.Range(1, 30))
    {
        tasks.Add(AddBook($"Book_{id}"));
    }

    Task.WaitAll(tasks.ToArray());
    tasks.Clear();
```

同样，更新数量被更改为在任务中调用`Add`和`subtract`方法：

```cs
    // let's update the quantity of the books by adding 1, 2, 3, 4, 5 ...
    foreach (var quantity in Enumerable.Range(1, 10))
    {
        foreach (var id in Enumerable.Range(1, 30))
        {
            tasks.Add(UpdateQuantity($"Book_{id}", quantity));
        }
    }

    // let's update the quantity of the books by subtractin 1, 2, 3, 4, 5 ...
    foreach (var quantity in Enumerable.Range(1, 10))
    {
        foreach (var id in Enumerable.Range(1, 30))
        {
            tasks.Add(UpdateQuantity($"Book_{id}", -quantity));
        }
    }

    // wait for all adds and subtracts to finish
    Task.WaitAll(tasks.ToArray());
```

重构后，单元测试不再成功完成，当单元测试现在运行时，会报告错误，指示在集合中找不到书籍。这将报告为“字典中未找到给定的键”。这是因为每次实例化上下文时，都会创建一个新的书籍集合。第一步是限制上下文的创建。这是通过更改构造函数的访问权限来完成的，以便该类不再可以直接实例化。相反，添加一个新的公共`static`属性，只支持`get`操作。该属性将返回`InventoryContext`类的底层`static`实例，并且如果实例丢失，将创建它：

```cs
internal class InventoryContext : IInventoryContext
{ 
    protected InventoryContext()
    {
        _books = new Dictionary<string, Book>();
    }

    private static InventoryContext _context;
    public static InventoryContext Singleton
    {
        get
        {
            if (_context == null)
            {
                _context = new InventoryContext();
            }

            return _context;
        }
    }
    ...
}    
```

这仍然不足以修复损坏的单元测试，但这是由于不同的原因。为了确定问题，单元测试在调试模式下运行，并在`UpdateQuantity`方法中设置断点。第一次运行时，我们可以看到已经创建了 28 本书并加载到书籍集合中，如下面的截图所示：

![](img/c1515640-71e6-43e4-a1f1-d0d8f21d063c.png)

在单元测试的这一点上，我们期望有 30 本书；然而，在我们开始调查之前，让我们再次运行单元测试。这一次，当我们尝试访问书籍集合以添加新书时，我们遇到了一个“对象引用未设置为对象的实例”错误，如下面的截图所示：

![](img/268b58d4-cbbe-4b12-9b77-df3b31df45db.png)

此外，当单元测试第三次运行时，不再遇到“对象引用未设置为对象的实例”错误，但我们的集合中只有 27 本书，如下面的截图所示：

![](img/46a4b8bc-48e9-4586-b1af-c64784920993.png)

这种不可预测的行为是竞争条件的典型特征，并且表明共享资源，即`InventoryContext`单例，正在被多个线程处理而没有同步访问。静态对象的构造仍然允许创建多个`InventoryContext`单例的实例：

```cs
public static InventoryContext Singleton
{
    get
    {
        if (_context == null)
        {
            _context = new InventoryContext();
        }

        return _context;
    }
}
```

竞争条件是多个线程评估`if`语句为真，并且它们都尝试构造`_context`对象。所有线程都会成功，但它们会通过这样做覆盖先前构造的值。当然，这是低效的，特别是当构造函数是昂贵的操作时，但单元测试发现的问题是`_context`对象实际上是由一个线程在另一个线程或多个线程更新书籍集合之后构造的。这就是为什么书籍集合`_books`在运行之间具有不同数量的元素。

为了防止这个问题，该模式在构造函数周围使用锁定，如下所示：

```cs
private static object _lock = new object();
public static InventoryContext Singleton
{
    get
    { 
        if (_context == null)
        {
 lock (_lock)
            {
                _context = new InventoryContext();
            }
        }

        return _context;
    }
}
```

不幸的是，单元测试仍然失败。这是因为虽然一次只有一个线程可以进入锁定，但所有被阻塞的实例仍然会在阻塞线程完成后进入锁定。该模式通过在锁定内部进行额外检查来处理这种情况，以防构造已经完成：

```cs
public static InventoryContext Singleton
{
    get
    { 
        if (_context == null)
        {
            lock (_lock)
            {
 if (_context == null)
                {
                    _context = new InventoryContext();
                }
            }
        }

        return _context;
    }
}
```

前面的锁定是必不可少的，因为它防止静态的`InventoryContext`对象被多次实例化。不幸的是，我们的测试仍然没有始终通过；随着每次更改，单元测试越来越接近通过。一些单元测试运行将在没有错误的情况下完成，但偶尔，测试将以失败的结果完成，如下面的截图所示：

![](img/8cf03f40-50e7-4459-8209-60388b3b9a71.png)

我们的静态存储库实例现在是线程安全的，但我们对书籍集合的访问不是。需要注意的一点是，所使用的`Dictionary`类不是线程安全的。幸运的是，.Net Framework 中有线程安全的集合可用。这些类确保了对集合的**添加和删除**是为多线程进程编写的。需要注意的是，只有添加和删除是线程安全的，因为这将在稍后变得重要。更新后的构造函数如下所示：

```cs
protected InventoryContext()
{
    _books = new ConcurrentDictionary<string, Book>();
}
```

微软建议在目标为.Net Framework 1.1 或更早版本的应用程序中，使用`System.Collections.Concurrent`中的线程安全集合，而不是`System.Collections`中对应的集合。

再次运行单元测试后，引入`ConcurrentDictionary`类仍然不足以防止书籍的错误维护。单元测试仍然失败。并发字典可以防止多个线程不可预测地添加和删除，但对集合中的项目本身没有任何保护。这意味着对集合中的对象的更新不是线程安全的。

让我们更仔细地看一下多线程环境中的竞争条件，以了解为什么会出现这种情况。

# 竞争条件示例

以下一系列图表描述了两个线程**ThreadA**和**ThreadB**之间概念上发生的情况。第一个图表显示了两个线程都没有从集合中获取任何值：

![](img/fe27049f-ade1-485f-a255-bcd56290bbc6.png)

下图显示了两个线程都从名称为`Chester`的书籍集合中读取：

![](img/b2b80704-df53-4c39-b3c5-e7176a6c2467.png)

下图显示了**ThreadA**通过增加`4`来更新书籍的数量，而**ThreadB**通过增加`3`来更新书籍的数量：

![](img/b5a8650f-1f3d-4414-9109-f15046958d42.png)

然后，当更新后的书籍被持久化回集合时，我们得到了一个未知数量的结果，如下图所示：

![](img/f4a99992-46d0-4eb7-a464-d82126476461.png)

为了避免这种竞争条件，我们需要在更新操作进行时阻止其他线程。在`InventoryContext`中，阻止其他线程的方法是在更新书籍数量时进行锁定：

```cs
public bool UpdateQuantity(string name, int quantity)
{
    lock (_lock)
    {
        _books[name].Quantity += quantity;
    }

    return true;
}
```

单元测试现在可以顺利完成，因为额外的锁定防止了不可预测的竞争条件。

`InventoryContext`类仍然不完整，因为它只是完成了足够的部分来说明单例和存储库模式。在后面的章节中，`InventoryContext`类将被改进以使用 Entity Framework，这是一个**对象关系映射**（**ORM**）框架。此时，`InventoryContext`类将被改进以支持额外的功能。

# AddInventoryCommand

有了我们的存储库后，三个`InventoryCommand`类可以完成。首先是`AddInventoryCommand`，如下所示：

```cs
internal class AddInventoryCommand : NonTerminatingCommand, IParameterisedCommand
{
    private readonly IInventoryContext _context;

    internal AddInventoryCommand(IUserInterface userInterface, IInventoryContext context) 
                                                            : base(userInterface)
    {
        _context = context;
    }

    public string InventoryName { get; private set; }

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
}
```

首先要注意的是，存储库`IInventoryContext`在构造函数中与前一章描述的`IUserInterface`接口一起被注入。命令还需要提供一个参数，即`name`*，*这在实现了前一章中也涵盖的`IParameterisedCommand`接口的`GetParameters`方法中被检索。然后在`InternalCommand`方法中运行命令，该方法简单地在存储库上执行`AddBook`方法，并返回一个指示命令是否成功执行的布尔值。

# TestInventoryContext

与上一章中使用的`TestUserInterface`类类似，`TestInventoryContext`类将用于模拟我们的存储库的行为，实现`IInventoryContext`接口。该类将支持接口的三种方法，以及支持在单元测试期间添加到集合中的两种附加方法和更新的书籍。

为了支持`TestInventoryContext`类，将使用两个集合：

```cs
private readonly IDictionary<string, Book> _seedDictionary;
private readonly IDictionary<string, Book> _books;
```

第一个用于存储书籍的起始集合，而第二个用于存储书籍的最终集合。构造函数如下所示；请注意字典是彼此的副本：

```cs
public TestInventoryContext(IDictionary<string, Book> books)
{
    _seedDictionary = books.ToDictionary(book => book.Key,
                                         book => new Book { Id = book.Value.Id, 
                                                            Name = book.Value.Name, 
                                                            Quantity = book.Value.Quantity });
    _books = books;
}
```

`IInventoryContext`方法被编写为更新和返回集合中的一本书，如下所示：

```cs
public Book[] GetBooks()
{
    return _books.Values.ToArray();
}

public bool AddBook(string name)
{
    _books.Add(name, new Book() { Name = name });

    return true;
}

public bool UpdateQuantity(string name, int quantity)
{
    _books[name].Quantity += quantity;

    return true;
}
```

在单元测试结束时，可以使用剩余的两种方法来确定起始和结束集合之间的差异：

```cs
public Book[] GetAddedBooks()
{
    return _books.Where(book => !_seedDictionary.ContainsKey(book.Key))
                                                    .Select(book => book.Value).ToArray();
}

public Book[] GetUpdatedBooks()
{ 
    return _books.Where(book => _seedDictionary[book.Key].Quantity != book.Value.Quantity)
                                                    .Select(book => book.Value).ToArray();
}
```

在软件行业中，关于模拟、存根、伪造和其他用于识别和/或分类测试中使用的类型或服务的差异存在一些混淆，这些类型或服务不适用于生产，但对于单元测试是必要的。这些依赖项可能具有与其*真实*对应项不同、缺失和/或相同的功能。

例如，`TestUserInterface`类可以被称为模拟，因为它提供了对单元测试的一些期望（例如，断言语句），而`TestInventoryContext`类将是伪造的，因为它提供了一个工作实现。在本书中，我们不会严格遵循这些分类。

# AddInventoryCommandTest

团队已经更新了`AddInventoryCommandTest`来验证`AddInventoryCommand`的功能。此测试将验证向现有库存中添加一本书。测试的第一部分是定义接口的预期，这只是一个单独的提示，用于接收新书名（请记住`TestUserInterface`类需要三个参数：预期输入、预期消息和预期警告）：

```cs
const string expectedBookName = "AddInventoryUnitTest";
var expectedInterface = new Helpers.TestUserInterface(
    new List<Tuple<string, string>>
    {
        new Tuple<string, string>("Enter name:", expectedBookName)
    },
    new List<string>(),
    new List<string>()
);
```

`TestInventoryContext`类将初始化为模拟现有书籍集合中的一本书：

```cs
var context = new TestInventoryContext(new Dictionary<string, Book>
{
    { "Gremlins", new Book { Id = 1, Name = "Gremlins", Quantity = 7 } }
});
```

以下代码片段显示了`AddInventoryCommand`的创建、命令的运行以及用于验证命令成功运行的断言语句：

```cs
// create an instance of the command
var command = new AddInventoryCommand(expectedInterface, context);

// add a new book with parameter "name"
var result = command.RunCommand();

Assert.IsFalse(result.shouldQuit, "AddInventory is not a terminating command.");
Assert.IsTrue(result.wasSuccessful, "AddInventory did not complete Successfully.");

// verify the book was added with the given name with 0 quantity
Assert.AreEqual(1, context.GetAddedBooks().Length, "AddInventory should have added one new book.");

var newBook = context.GetAddedBooks().First();
Assert.AreEqual(expectedBookName, newBook.Name, "AddInventory did not add book successfully."); 
```

命令运行后，将验证结果是否无错误运行，并且命令不是终止命令。`Assert`语句的其余部分验证了预期只添加了一本带有预期名称的书。

# UpdateQuantityCommand

`UpdateQuantityCommand`与`AddInventoryCommand`非常相似，其源代码如下：

```cs
internal class UpdateQuantityCommand : NonTerminatingCommand, IParameterisedCommand
{
    private readonly IInventoryContext _context; 

    internal UpdateQuantityCommand(IUserInterface userInterface, IInventoryContext context) 
                                                                            : base(userInterface)
    {
        _context = context;
    }

    internal string InventoryName { get; private set; }

    private int _quantity;
    internal int Quantity { get => _quantity; private set => _quantity = value; }

    ...
}
```

与`AddInventoryCommand`一样，`UpdateInventoryCommand`命令是一个带参数的非终止命令。因此，它扩展了`NonTerminatingCommand`基类，并实现了`IParameterisedCommand`接口。同样，`IUserInterface`和`IInventoryContext`的依赖项在构造函数中注入：

```cs
    /// <summary>
    /// UpdateQuantity requires name and an integer value
    /// </summary>
    /// <returns></returns>
    public bool GetParameters()
    {
        if (string.IsNullOrWhiteSpace(InventoryName))
            InventoryName = GetParameter("name");

        if (Quantity == 0)
            int.TryParse(GetParameter("quantity"), out _quantity);

        return !string.IsNullOrWhiteSpace(InventoryName) && Quantity != 0;
    }   
```

`UpdateQuantityCommand`类确实具有一个额外的参数*quantity*，该参数是作为`GetParameters`方法的一部分确定的。

最后，通过存储库的`InternalCommand`重写方法，更新书的数量：

```cs
    protected override bool InternalCommand()
    {
        return _context.UpdateQuantity(InventoryName, Quantity);
    }
```

现在`UpdateQuantityCommand`类已经定义，接下来的部分将添加一个单元测试来验证该命令。

# UpdateQuantityCommandTest

`UpdateQuantityCommandTest`包含一个测试，用于验证在现有集合中更新书籍的情景。预期接口和现有集合的创建如下代码所示（请注意，测试涉及将`6`添加到现有书的数量）：

```cs
const string expectedBookName = "UpdateQuantityUnitTest";
var expectedInterface = new Helpers.TestUserInterface(
    new List<Tuple<string, string>>
    {
        new Tuple<string, string>("Enter name:", expectedBookName),
        new Tuple<string, string>("Enter quantity:", "6")
    },
    new List<string>(),
    new List<string>()
);

var context = new TestInventoryContext(new Dictionary<string, Book>
{
    { "Beavers", new Book { Id = 1, Name = "Beavers", Quantity = 3 } },
    { expectedBookName, new Book { Id = 2, Name = expectedBookName, Quantity = 7 } },
    { "Ducks", new Book { Id = 3, Name = "Ducks", Quantity = 12 } }
});
```

下面的代码块显示了命令的运行以及非终止命令成功运行的初始验证：

```cs
// create an instance of the command
var command = new UpdateQuantityCommand(expectedInterface, context);

var result = command.RunCommand();

Assert.IsFalse(result.shouldQuit, "UpdateQuantity is not a terminating command.");
Assert.IsTrue(result.wasSuccessful, "UpdateQuantity did not complete Successfully.");
```

测试的期望是不会添加新书籍，并且现有书籍的数量为 7，将增加 6，结果为新数量为 13：

```cs
Assert.AreEqual(0, context.GetAddedBooks().Length, 
                    "UpdateQuantity should not have added one new book.");

var updatedBooks = context.GetUpdatedBooks();
Assert.AreEqual(1, updatedBooks.Length, 
                    "UpdateQuantity should have updated one new book.");
Assert.AreEqual(expectedBookName, updatedBooks.First().Name, 
                    "UpdateQuantity did not update the correct book.");
Assert.AreEqual(13, updatedBooks.First().Quantity, 
                    "UpdateQuantity did not update book quantity successfully.");
```

添加了 `UpdateQuantityCommand` 类后，将在下一节中添加检索库存的能力。

# GetInventoryCommand

`GetInventoryCommand` 命令与前两个命令不同，因为它不需要任何参数。它使用 `IUserInterface` 依赖项和 `IInventoryContext` 依赖项来写入集合的内容。如下所示：

```cs
internal class GetInventoryCommand : NonTerminatingCommand
{
    private readonly IInventoryContext _context;
    internal GetInventoryCommand(IUserInterface userInterface, IInventoryContext context) 
                                                           : base(userInterface)
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

实现了 `GetInventoryCommand` 命令后，下一步是添加一个新的测试。

# GetInventoryCommandTest

`GetInventoryCommandTest` 涵盖了当使用 `GetInventoryCommand` 命令检索书籍集合时的场景。测试将定义预期的消息（记住，第一个参数是用于参数，第二个参数是用于消息，第三个参数是用于警告），这些消息将在测试用户界面时发生：

```cs
var expectedInterface = new Helpers.TestUserInterface(
    new List<Tuple<string, string>>(),
    new List<string>
    {
        "Gremlins                      \tQuantity:7",
        "Willowsong                    \tQuantity:3",
    },
    new List<string>()
);
```

这些消息将对应于模拟存储库，如下所示：

```cs
var context = new TestInventoryContext(new Dictionary<string, Book>
{
    { "Gremlins", new Book { Id = 1, Name = "Gremlins", Quantity = 7 } },
    { "Willowsong", new Book { Id = 2, Name = "Willowsong", Quantity = 3 } },
});
```

单元测试使用模拟依赖项运行命令。它验证命令是否无错误执行，并且命令不是终止命令：

```cs
// create an instance of the command
var command = new GetInventoryCommand(expectedInterface, context); 
var result = command.RunCommand();

Assert.IsFalse(result.shouldQuit, "GetInventory is not a terminating command.");
```

预期的消息在 `TestUserInterface` 中进行验证，因此单元测试剩下的唯一任务就是确保命令没有神秘地添加或更新书籍：

```cs
Assert.AreEqual(0, context.GetAddedBooks().Length, "GetInventory should not have added any books.");
Assert.AreEqual(0, context.GetUpdatedBooks().Length, "GetInventory should not have updated any books.");
```

现在已经添加了适合 `GetInventoryCommand` 类的单元测试，我们将引入工厂模式来管理特定命令的创建。

# 工厂模式

团队应用的下一个模式是 GoF 工厂模式。该模式引入了一个**创建者**，其责任是实例化特定类型的实现。它的目的是封装围绕构造类型的复杂性。工厂模式允许更灵活地应对应用程序的变化，通过限制所需更改的数量，而不是在调用类中进行构造。这是因为构造的复杂性在一个位置，而不是分布在应用程序的多个位置。

在 FlixOne 示例中，`InventoryCommandFactory` 实现了该模式，并屏蔽了构造每个不同的 `InventoryCommand` 实例的细节。在这种情况下，从控制台应用程序接收到的输入将用于确定要返回的 `InventoryCommand` 的具体实现。重要的是要注意返回类型是 `InventoryCommand` 抽象类，因此屏蔽了调用类对具体类的细节。

`InventoryCommandFactory` 在下面的代码块中显示。但是，现在专注于 `GetCommand` 方法，因为它实现了工厂模式：

```cs
public class InventoryCommandFactory : IInventoryCommandFactory
{
    private readonly IUserInterface _userInterface;
    private readonly IInventoryContext _context = InventoryContext.Instance;

    public InventoryCommandFactory(IUserInterface userInterface)
    {
        _userInterface = userInterface;
    }

    ...
}
```

`GetCommand` 使用给定的字符串来确定要返回的 `InventoryCommand` 的特定实现：

```cs
public InventoryCommand GetCommand(string input)
{
    switch (input)
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

所有命令都需要提供 `IUserInterface`，但有些还需要访问存储库。这些将使用 `IInventoryContext` 的单例实例提供。

工厂模式通常与接口一起使用作为返回类型。在这里，它被说明为 `InventoryCommand` 基类。

# 单元测试

乍一看，为这样一个简单的类构建单元测试似乎是团队时间的浪费。通过构建单元测试，发现了两个重要问题，这些问题可能会被忽略。

# 问题一 - UnknownCommand

第一个问题是当接收到一个不匹配任何已定义的 `InventoryCommand` 输入的命令时该怎么办。在审查要求后，团队注意到他们错过了这个要求，如下面的截图所示：

![](img/2609a169-37d7-4a74-8aac-65d25b8c1a77.png)

团队决定引入一个新的`InventoryCommand`类，`UnknownCommand`，来处理这种情况。 `UnknownCommand`类应该通过`IUserInterface`的`WriteWarning`方法向控制台打印警告消息，不应导致应用程序结束，并且应返回 false 以指示命令未成功运行。 实现细节如下所示：

```cs
internal class UnknownCommand : NonTerminatingCommand
{ 
    internal UnknownCommand(IUserInterface userInterface) : base(userInterface)
    {
    }

    protected override bool InternalCommand()
    { 
        Interface.WriteWarning("Unable to determine the desired command."); 

        return false;
    }
}
```

为`UnknownCommand`创建的单元测试将测试警告消息以及`InternalCommand`方法返回的两个布尔值：

```cs
[TestClass]
public class UnknownCommandTests
{
    [TestMethod]
    public void UnknownCommand_Successful()
    {
        var expectedInterface = new Helpers.TestUserInterface(
            new List<Tuple<string, string>>(),
            new List<string>(),
            new List<string>
            {
                "Unable to determine the desired command."
            }
        ); 

        // create an instance of the command
        var command = new UnknownCommand(expectedInterface);

        var result = command.RunCommand();

        Assert.IsFalse(result.shouldQuit, "Unknown is not a terminating command.");
        Assert.IsFalse(result.wasSuccessful, "Unknown should not complete Successfully.");
    }
}
```

`UnknownCommandTests`覆盖了需要测试的命令。 接下来，将实现围绕`InventoryCommandFactory`的测试。

# InventoryCommandFactoryTests

`InventoryCommandFactoryTests`包含与`InventoryCommandFactory`相关的单元测试。 因为每个测试都将具有类似的模式，即构造`InventoryCommandFactory`及其`IUserInterface`依赖项，然后运行`GetCommand`方法，因此创建了一个共享方法，该方法将在测试初始化时运行：

```cs
[TestInitialize]
public void Initialize()
{
    var expectedInterface = new Helpers.TestUserInterface(
        new List<Tuple<string, string>>(),
        new List<string>(),
        new List<string>()
    ); 

    Factory = new InventoryCommandFactory(expectedInterface);
}
```

`Initialize`方法构造了一个存根`IUserInterface`并设置了`Factory`属性。 然后，各个单元测试采用简单的形式，验证返回的对象是否是正确的类型。 首先，当用户输入`"q"`或`"quit"`时，应返回`QuitCommand`类的实例，如下所示：

```cs
[TestMethod]
public void QuitCommand_Successful()
{ 
    Assert.IsInstanceOfType(Factory.GetCommand("q"), typeof(QuitCommand), 
                                                            "q should be QuitCommand");
    Assert.IsInstanceOfType(Factory.GetCommand("quit"), typeof(QuitCommand), 
                                                            "quit should be QuitCommand");
}
```

`QuitCommand_Successful`测试方法验证了当运行`InventoryCommandFactory`方法`GetCommand`时，返回的对象是`QuitCommand`类型的特定实例。 当提交`"?"`时，`HelpCommand`才可用：

```cs
[TestMethod]
public void HelpCommand_Successful()
{
    Assert.IsInstanceOfType(Factory.GetCommand("?"), typeof(HelpCommand), "h should be HelpCommand"); 
}
```

团队确实添加了一个针对`UnknownCommand`的测试，验证了当给出与现有命令不匹配的值时，`InventoryCommand`将如何响应：

```cs
[TestMethod]
public void UnknownCommand_Successful()
{
    Assert.IsInstanceOfType(Factory.GetCommand("add"), typeof(UnknownCommand), 
                                                        "unmatched command should be UnknownCommand");
    Assert.IsInstanceOfType(Factory.GetCommand("addinventry"), typeof(UnknownCommand), 
                                                        "unmatched command should be UnknownCommand");
    Assert.IsInstanceOfType(Factory.GetCommand("h"), typeof(UnknownCommand), 
                                                        "unmatched command should be UnknownCommand");
    Assert.IsInstanceOfType(Factory.GetCommand("help"), typeof(UnknownCommand), 
                                                        "unmatched command should be UnknownCommand");
}
```

有了测试方法，现在我们可以涵盖一种情况，即在应用程序中给出一个不匹配已知命令的命令。

# 问题二 - 不区分大小写的文本命令

第二个问题是在再次审查要求时发现的，即命令不应区分大小写：

![](img/10cf9ef0-f9f0-459d-969c-1ded9e9e093e.png)

通过对`UpdateInventoryCommand`的测试，发现`InventoryCommandFactory`在以下测试中是区分大小写的：

```cs
[TestMethod]
public void UpdateQuantityCommand_Successful()
{
    Assert.IsInstanceOfType(Factory.GetCommand("u"), 
                            typeof(UpdateQuantityCommand), 
                            "u should be UpdateQuantityCommand");
    Assert.IsInstanceOfType(Factory.GetCommand("updatequantity"), 
                            typeof(UpdateQuantityCommand), 
                            "updatequantity should be UpdateQuantityCommand");
    Assert.IsInstanceOfType(Factory.GetCommand("UpdaTEQuantity"), 
                            typeof(UpdateQuantityCommand), 
                            "UpdaTEQuantity should be UpdateQuantityCommand");
}
```

幸运的是，通过在确定命令之前对输入应用`ToLower()`方法，这个测试很容易解决，如下所示：

```cs
public InventoryCommand GetCommand(string input)
{
    switch (input.ToLower())
    {
        ...
    }
}
```

这种情况突出了`Factory`方法的价值以及利用单元测试来帮助验证开发过程中的需求的价值，而不是依赖用户测试。

# .NET Core 中的功能

第三章，*实现设计模式 - 基础部分 1*，以及本章的第一部分已经演示了 GoF 模式，而没有使用任何框架。 有必要覆盖这一点，因为有时针对特定模式可能没有可用的框架，或者在特定场景中不适用。 此外，了解框架提供的功能是很重要的，以便知道何时应该使用某种模式。 本章的其余部分将介绍.NET Core 提供的一些功能，支持我们迄今为止已经涵盖的一些模式。

# IServiceCollection

.NET Core 设计时内置了**依赖注入**（**DI**）。 通常，.NET Core 应用程序的启动包含为应用程序设置 DI 的过程，主要包括创建服务集合。 框架在应用程序需要时使用这些服务来提供依赖项。 这些服务为强大的**控制反转**（**IoC**）框架奠定了基础，并且可以说是.NET Core 最酷的功能之一。 本节将完成控制台应用程序，并演示.NET Core 如何基于`IServiceCollection`接口支持构建复杂的 IoC 框架。

`IServiceCollection`接口用于定义容器中可用的服务，该容器实现了`IServiceProvider`接口。这些服务本身是在应用程序需要时在运行时注入的类型。例如，之前定义的`ConsoleUserInterface`接口将在运行时作为服务注入。这在下面的代码中显示：

```cs
IServiceCollection services = new ServiceCollection();
services.AddTransient<IUserInterface, ConsoleUserInterface>();
```

在上述代码中，`ConsoleUserInterface`接口被添加为实现`IUserInterface`接口的服务。如果 DI 提供了另一种需要`IUserInterface`接口依赖的类型，那么将使用`ConsoleUserInterface`接口。例如，`InventoryCommandFactory`也被添加到服务中，如下面的代码所示：

```cs
services.AddTransient<IInventoryCommandFactory, InventoryCommandFactory>();
```

`InventoryCommandFactory`有一个需要`IUserInterface`接口实现的构造函数：

```cs
public class InventoryCommandFactory : IInventoryCommandFactory
{
    private readonly IUserInterface _userInterface;

    public InventoryCommandFactory(IUserInterface userInterface)
    {
        _userInterface = userInterface;
    }
    ...
}
```

稍后，请求一个`InventoryCommandFactory`的实例，如下所示：

```cs
IServiceProvider serviceProvider = services.BuildServiceProvider();
var service = serviceProvider.GetService<IInventoryCommandFactory>();
service.GetCommand("a");
```

然后，`IUserInterface`的一个实例（在这个应用程序中是注册的`ConsoleUserInterface`）被实例化并提供给`InventoryCommandFactory`的构造函数。

在注册服务时可以指定不同类型的服务*生命周期*。生命周期规定了类型将如何实例化，包括瞬态、作用域和单例。瞬态意味着每次请求时都会创建服务。作用域将在后面讨论，特别是在查看与网站相关的模式时，服务是按照网页请求创建的。单例的行为类似于我们之前讨论的单例模式，并且将在本章后面进行讨论。

# CatalogService

`CatalogService`接口代表团队正在构建的控制台应用程序，并被描述为具有一个`Run`方法，如`ICatalogService`接口中所示：

```cs
interface ICatalogService
{
    void Run();
}
```

该服务有两个依赖项，`IUserInterface`和`IInventoryCommandFactory`，它们将被注入到构造函数中并存储为局部变量：

```cs
public class CatalogService : ICatalogService
{
    private readonly IUserInterface _userInterface;
    private readonly IInventoryCommandFactory _commandFactory;

    public CatalogService(IUserInterface userInterface, IInventoryCommandFactory commandFactory)
    {
        _userInterface = userInterface;
        _commandFactory = commandFactory;
    }
    ...
}
```

`Run`方法基于团队在第三章中展示的早期设计。它打印一个问候语，然后循环，直到用户输入退出库存命令为止。每次循环都会执行命令，如果命令不成功，它将打印一个帮助消息：

```cs
public void Run()
{
    Greeting();

    var response = _commandFactory.GetCommand("?").RunCommand();

    while (!response.shouldQuit)
    {
        // look at this mistake with the ToLower()
        var input = _userInterface.ReadValue("> ").ToLower();
        var command = _commandFactory.GetCommand(input);

        response = command.RunCommand();

        if (!response.wasSuccessful)
        {
            _userInterface.WriteMessage("Enter ? to view options.");
        }
    }
}
```

现在我们已经准备好了`CatalogService`接口，下一步将是把所有东西放在一起。下一节将使用.NET Core 来完成这一点。

# IServiceProvider

有了`CatalogService`定义，团队最终能够在.NET Core 中将所有东西放在一起。所有应用程序的开始，即 EXE 程序，都是`Main`方法，.NET Core 也不例外。程序如下所示：

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
    }

    private static void ConfigureServices(IServiceCollection services)
    {
        // Add application services.
        services.AddTransient<IUserInterface, ConsoleUserInterface>(); 
        services.AddTransient<ICatalogService, CatalogService>();
        services.AddTransient<IInventoryCommandFactory, InventoryCommandFactory>(); 
    }
}
```

在`ConfigureServices`方法中，不同类型被添加到 IoC 容器中，包括`ConsoleUserInterface`、`CatalogService`和`InventoryCommandFactory`类。`ConsoleUserInterface`和`InventoryCommandFactory`类将根据需要注入，而`CatalogService`类将从`ServiceCollection`对象中包含的添加类型构建的`IServiceProvider`接口中显式检索出来。程序将一直运行，直到`CatalogService`的`Run`方法完成。

在第五章中，*实现设计模式-.NET Core*，将重新讨论单例模式，使用.NET Core 内置的能力，通过使用`IServiceCollection`的`AddSingleton`方法来控制`InventoryContext`实例。

# 控制台应用程序

控制台应用程序在命令行中运行时很简单，但它是一个遵循 SOLID 原则的良好设计代码的基础，这些原则在第三章中讨论过，*实现设计模式-基础部分 1*。运行时，应用程序提供一个简单的问候，并显示一个帮助消息，包括命令的支持和示例：

![](img/2b227f12-9b64-4501-9190-7385cf1f6d34.png)

然后应用程序循环执行命令，直到收到退出命令。以下屏幕截图说明了其功能：

![](img/23375dec-001c-4064-9278-19da5d745827.png)

这并不是最令人印象深刻的控制台应用程序，但它用来说明了许多原则和模式。

# 摘要

与第三章类似，*实现设计模式-基础部分 1*，本章继续描述了为 FlixOne 构建库存管理控制台应用程序，以展示使用面向对象编程（OOP）设计模式的实际示例。在本章中，GoF 的单例模式和工厂模式是重点。这两种模式在.NET Core 应用程序中起着特别重要的作用，并将在接下来的章节中经常使用。本章还介绍了使用内置框架提供 IoC 容器的方法。

本章以一个符合第三章 *实现设计模式-基础部分 1*中确定的要求的工作库存管理控制台应用程序结束。这些要求是两章中创建的单元测试的基础，并用于说明 TDD。通过拥有一套验证本开发阶段所需功能的测试，团队对应用程序能够通过用户验收测试（UAT）有更高的信心。

在下一章中，我们将继续描述构建库存管理应用程序。重点将从基本的面向对象编程模式转移到使用.NET Core 框架来实现不同的模式。例如，本章介绍的单例模式将被重构以利用`IServiceCollection`的能力来创建单例，我们还将更仔细地研究其依赖注入能力。此外，该应用程序将扩展以支持使用各种日志提供程序进行日志记录。

# 问题

以下问题将帮助您巩固本章中包含的信息：

1.  提供一个例子，说明为什么使用单例模式不是限制访问共享资源的好机制。

1.  以下陈述是否正确？为什么？`ConcurrentDictionary`可以防止集合中的项目被多个线程同时更新。

1.  什么是竞态条件，为什么应该避免？

1.  工厂模式如何帮助简化代码？

1.  .NET Core 应用程序需要第三方 IoC 容器吗？
