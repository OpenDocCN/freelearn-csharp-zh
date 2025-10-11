# 构建数据访问层

从本章开始，我们将通过使用 .NET Core 来具体实现 Web 服务部分。我们将涵盖开发真实 Web 服务的一些关键方面——从数据访问层的设计到 HTTP 路由的实现。

在本章中，我们将首先定义数据访问部分。数据访问部分是必要的，用于在数据库或数据源中存储信息，并且通常是应用程序中最微妙的部分之一。我们将专注于实现一个目录 Web 服务。此外，我们还将探索不同的第三方工具来访问我们的数据，并解释如何设置项目和实现数据域。

本章将涵盖以下主题：

+   设计项目实体

+   选择合适的工具

+   使用 EF Core 实现数据访问层

+   使用 Dapper 实现数据访问层

+   使用内存数据库测试数据层

本章中的代码可以从以下 GitHub 仓库获取：[https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3](https://github.com/PacktPublishing/Hands-On-RESTful-Web-Services-with-ASP.NET-Core-3).

# 设置项目

就像之前的章节一样，我们可以通过使用 Web API 模板来创建一个新的项目。让我们打开终端并执行以下命令：

[PRE0]

第一个 `dotnet new` 命令创建了一个名为 `Catalog.API` 的新解决方案文件。第二个 `dotnet new` 指令在 `src` 文件夹中创建了一个新的 Web API 项目。最后，最后一个 `dotnet sln` 命令将项目添加到我们的解决方案中。

结果的文件系统结构如下所示：

[PRE1]

`src` 文件夹将包含我们所有的代码以及本书中我们将添加的附加项目。在本章的后面部分，我们还将添加一个 `tests` 文件夹，其中将包含我们项目的所有测试。

# 实现域模型

如第 5 章中在 *数据传输* 部分讨论的，[ASP.NET Core 中的 Web 服务堆栈](deede298-fc20-4523-afa6-02ed2c0592fd.xhtml)，*域模型* 是我们服务处理的数据的表示。考虑一个 *音乐商店的目录 Web 服务*，我们需要处理的主要数据包括 API 使用的 *实体*。

为了保证可重用性和松散耦合，我们将定义服务域模型为一个独立的项目。首先，让我们在 `src` 文件夹中通过执行以下命令创建一个新的 `Catalog.Domain` 项目：

[PRE2]

上述命令还指定了 `netstandard2.1` 版本为目标框架。此外，在创建 `Catalog.Domain` 项目后，我们需要将其添加到我们的解决方案中：

[PRE3]

上述指令将 `Catalog.Domain` 项目添加到 `Catalog.API.sln` 文件中。因此，我们现在已经准备好设计和实现我们 Web 服务的实体。

# 设计实体

现在我们可以继续进行我们需要实现实体的设计阶段。让我们从 `Item` 类开始，它将成为我们领域模型中的中心实体。它代表一张音乐专辑，包含与专辑相关的所有属性和特征，包括 *描述*、*名称*、*发行* *日期* 和 *格式*。实体还将提供一些通常出现在目录中的附加信息，例如可用库存、图片和价格。

让我们先设计一个图来描述我们的代码：

![图片](img/1390bf3f-dc67-4333-908f-e1df3a3fc65f.png)

此图定义了服务中涉及的实体：

+   `Item` 是我们声明中的主要实体。它包含关于专辑的所有信息以及指向 *艺术家* 和 *类型* 的引用。

+   `Artist` 实体表示与专辑相关的艺术家。

+   `Genre` 实体表示与专辑相关的音乐类型。

+   `Money` 实体表示专辑的价格。这是一个复杂类型，包含金额和货币单位。

# 实现实体

现在我们已经定义了目录服务中包含哪些属性和对象，让我们首先实现实体作为具体类型。所有领域模型都传统上存储在 `Catalog.Domain` 项目的 `Entities` 文件夹中。我们正在创建的第一个类型是 `Item` 类：

[PRE4]

首先，我们可以看到 `Item` 类的定义中包含了对其他类型的引用。需要注意的是，该实现使用 `Guid` 类型来指定 `Id`。这种方法的目的是在两个不同的数据源或目录合并时避免冲突。

让我们继续定义我们领域模型的相关实体：

[PRE5]

为了表示目的，以下类在相同的代码片段中实现。请注意，它们定义在不同的文件中：`Artist.cs`、`Genre.cs` 和 `Price.cs`。

上述代码定义了领域模型中包含的相关实体*.* `Artist` 类表示与专辑相关的艺术家。它包括 `Guid id` 和 `ArtistName`。同样，`Genre` 类是另一种类别类型，表示特定专辑的类型。最后，`Price` 类表示产品的价格（以及货币单位）。

# 使用 ORM 进行数据访问

数据访问是我们服务的一部分，帮助我们执行对数据源进行读取或写入操作。数据访问部分通常与 ORM 结合使用。一般来说，我们可以将 ORM 定义为一种面向对象编程方法，用于在互不兼容的类型系统之间转换关系数据。

ORM 工具或包是数据源和 Web 应用程序之间的桥梁。它们将关系表中的信息映射到类中，从而映射到对象中。

在 .NET 生态系统中，我们可以选择大量的不同 ORM。由微软官方维护的是 EF Core。

EF Core 是由微软和社区支持的开放源代码 ORM。它是 .NET Core 应用程序和 Web 服务中使用的默认 ORM。在本章中，我们还将概述由 Stack Exchange 和社区支持的开放源代码微 ORM **Dapper**。EF Core 和 Dapper 都作为 NuGet 包分发，并且通常与 .NET Core 框架非常良好地集成。

# 寻找适合工作的正确工具

EF Core 和 Dapper 都为我们提供了数据源的高级别抽象。尽管如此，它们都有一些优缺点。我们必须牢记，对于我们在做的每一个项目，我们都应该寻找适合这项工作的正确工具。

让我们分析一下这两个库的优缺点。在此之前，我们应该快速查看一些示例查询，以便了解两者之间的差异。以下代码片段描述了一个使用 EF Core 的示例查询：

[PRE6]

在前面的代码中，我们搜索每个具有相应描述的 `Item` 实体。让我们以一个 Dapper 查询为例继续进行：

[PRE7]

如您所见，EF Core 在我们的数据源上提供了高级别的抽象。

此外，EF Core 默认允许开发者使用集合（与 LINQ 集成）查询数据。这种方法快速且简单，但代价是：EF Core 将查询转换为 SQL 语言，有时生成的 SQL 查询没有优化。EF Core 还鼓励代码优先的方法，这意味着数据库侧的所有实体都使用 C# 代码生成。当你只有一个对象时，这可能看起来很简单，但当实体复杂时，可能会出现可维护性问题。

在复杂实体或使用代码优先方法的程序中，生成数据库实体的代码通常在单独的项目和存储库中实现，以避免数据库与整个解决方案之间的紧密耦合。

然而，EF Core 的一个常见问题是资源的早期获取。例如，考虑以下查询：

[PRE8]

它生成一个类似于以下查询的 SQL 查询：

[PRE9]

如您所见，尽管我们使用了 `Where` 子句，但之前 `ToList` 方法在评估查询时没有考虑 `Where` 子句。为了得到更好的结果，我们应该在评估之前执行 `Where` 语句：

[PRE10]

这种程序在性能和网络方面都更好。它可能看起来足够简单，但在分布式团队中，这些错误在代码库中很常见，而且在代码审查过程中很难发现。

在 Dapper 的层面上，这是一个微型 ORM，抽象级别发生了变化。Dapper 提供了对数据源更**透明**的访问。它默认保证通过使用纯 SQL 或存储过程以清晰的方式查询数据。因此，它也确保了更好的性能。另一方面，它与数据源紧密耦合，因为它使用数据源查询语言执行查询。

总结来说，本章将涵盖 EF Core 和 Dapper 库。在选择它们之间，你应该考虑你团队已有的技能以及如何优化服务的性能。如果你的团队对 SQL 有很强的了解，你可以通过实现存储过程而不是使用 EF Core 来进行。另一方面，如果你的团队还没有 SQL 技能，你应该考虑使用 EF Core。

# 使用 EF Core 实现数据访问层

在本节中，我们将探讨如何使用仓储模式和 EF Core 构建数据访问层。**仓储模式**是在我们的数据源之上提供的一个额外抽象。它提供了对数据的读取和写入操作。

# 定义仓储模式和单元工作

在我们开始之前，我们需要在 `Catalog.Domain` 项目中定义一些接口。让我们通过设置一个泛型接口来确定我们仓储的单元工作：

[PRE11]

`IUnitOfWork` 定义了两个方法：`SaveChangesAsync` 和 `SaveEntitiesAsync`。这两个方法用于有效地将我们的集合更改保存到数据库中。这两个方法都是异步的：它们返回 `Task` 类型，并接受一个 `CancellationToken` 类型的参数。`CancellationToken` 参数提供了一种停止挂起的异步操作的方法。

在某些情况下，仓储模式被实现为，当你更新或创建集合中的新元素时，这些更改会自动保存到数据库中。我更喜欢将有效的保存操作与读取和写入部分分开。这可以通过使用单元工作方法来实现。因此，仓储允许高层在内存集合上执行获取、创建和更新操作，而**单元工作**实现了一种将这些更改传输到数据库的方法。

让我们继续，通过在之前定义的 `IUnitOfWork` 接口相同的文件夹级别定义一个 `IRepository` 接口：

[PRE12]

如您所见，`IRepository` 并没有隐式使用 `IUnitOfWork` 接口。此外，它将 `UnitOfWork` 实例作为类的属性暴露出来。这种做法保证了所有 `IRepository` 接口的消费者都必须显式地通过调用 `SaveChangesAsync` 或 `SaveEntitiesAsync` 方法来更新数据库。

最后一步是定义 `IItemRepository` 接口，如下所示：

[PRE13]

该接口扩展了 `IRepository` 类，并引用了之前定义的 `Item` 实体。`IItemRepository` 定义了对数据源进行读取和写入操作。你可能注意到 `Add`、`Update` 等，这是因为它们只作用于应用程序内存中存储的集合，而有效的保存操作是由工作单元执行的。

# 将我们的存储库连接到数据库

一旦我们定义了 `IItemRepository` 接口和领域项目中所有的抽象，我们应该继续创建代表之前定义的抽象的具体实现的类。我们还将创建一个新的 `Catalog.Infrastructure` 项目，其中包含我们存储库的所有实现以及代表我们服务和数据库之间层的类。让我们通过在 `src` 文件夹中执行以下命令来创建 `Catalog.Infrastructure` 项目：

[PRE14]

完成后，我们的解决方案的文件夹结构如下所示：

[PRE15]

在开始实现 `IItemRepository` 之前，我们需要通过在项目文件夹内执行以下命令将 `Microsoft.EntityFrameworkCore` NuGet 包添加到 `Catalog.Infrastructure` 项目中：

[PRE16]

之后，我们可以通过使用以下命令将 `Catalog.Domain` 项目的引用添加到基础设施项目中继续操作：

[PRE17]

# DbContext 定义

`DbContext` 是我们应用程序和数据库之间的一种抽象。它使我们能够与数据交互，并对数据进行操作。`DbContext` 的实现也是我们应用程序和数据库之间会话的表示，我们可以用它来查询并将应用程序实体的实例保存到我们的数据源中。让我们快速看一下 `Catalog.Infrastructure` 项目中的 `DbContext` 实现：

[PRE18]

在快速查看代码后，我们应该提到以下几点：

+   `CatalogContext` 类代表我们的工作单元；因此，它实现了 `IUnitOfWork` 接口。

+   它使用 `DbSet<Item>` 类型来表示 `Item` 实例的集合。

+   `CatalogContext` 类的构造函数接受一个强制参数，它代表 `DbContextOptions`。这些选项用于指定有关数据库连接的一些关键信息。这包括要使用的数据库提供程序、数据库的连接字符串以及 ORM 所使用的所有跟踪策略。

+   `CatalogContext` 类还实现了 `SaveEntitiesAsync`，它调用由 `DbContext` 类派生的 `SaveChangesAsync` 方法。

一旦我们有了 `CatalogContext`，我们就可以继续实现 `IItemRepository` 接口。

# 实现存储库

下一个子节重点介绍 `IItemRepository` 接口的具体实现。重要的是要注意，`IItemRepository` 接口位于 `Catalog.Domain` 项目中，而 `ItemRepository` 的实现位于 `Catalog.Infrastructure` 项目中*.*

一旦我们构建了 `DbContext` 类，我们就可以通过实现具体的 `ItemRepository` 类来继续操作：

[PRE19]

上述代码实现了在 `IItemRepository` 接口中先前定义的 CRUD 操作。它还通过 `IUnitOfWork` 接口公开了 `CategoryContext`。这种方法的保证是，`IItemRepository` 的消费者可以修改和查询我们的集合，并使用相应的更改更新数据源。让我们回顾一下上述代码中实现的方法：

+   `GetAsync()` 方法使用上下文检索 `Items` 实体的集合。该方法显式使用 `AsNoTracking()` 方法以防止跟踪实体。此扩展方法可以在您不需要对实体执行写入操作时使用，并且它适用于只读数据。

+   `GetAsync(Guid id)` 重载了之前提到的方法，并使用 `AsNoTracking()` 实现了之前描述的相同目的。此方法还通过使用 EF Core 提供的 `Include()` 扩展方法获取相关实体的详细信息（`Genre` 和 `Artist`）。

+   `Add(Item entity)` 方法使用上下文添加作为参数传递的实体，并将添加的实体返回给调用者。

+   `Edit` 方法从上下文中更新目标实体，并将 `EntityState.Modified` 设置为实体状态。这种方法保证了，一旦实体处于修改状态，它将在保存步骤中被更新。

上述代码没有实现 `Delete` 方法。这是因为删除过程将在本书的后续部分实现。我们将执行软删除我们的数据。

# 将实体转换为 SQL 架构

EF Core 鼓励我们在服务中使用代码优先的方法。**代码优先**技术包括在 C# 中定义一些实体类，并使用它们在数据库端生成表。同样的方法应用于通常存在于 SQL 生态系统中的所有关系和约束，例如索引、主键和外键。本节演示了如何使用这种方法生成数据库架构。

首先，让我们从我们的 `Item.cs` 实体开始，并添加一个 ID 来表示与其他实体的关系：

[PRE20]

上述代码描述了 `Item` 类与 `Artist` 类以及 `Item` 类与 `Genre` 类之间的 **多对一** 关系。此外，`Artist` 和 `Genre` 实体都有一个集合，该集合引用了 `Item` 实体的集合。

让我们继续通过使用Fluent API方法实现我们的`Item`实体约束。一般来说，EF Core ORM实现了Fluent API技术，帮助我们处理约束定义。

通常，Fluent API，也称为**Fluent接口**，是一种组合面向对象API的方法，这些API本质上基于方法链。方法之间的链产生接近书面语法的源代码；例如，`myList.First().Items.Count().ShouldBe(2)`*.* 你可以看看这个例子有多易读；任何人都能理解。大多数EF Core约束通常都是使用这种方法构建的。

让我们在`Catalog.Infrastructure`项目中添加一个名为`SchemaDefinitions`的新文件夹。该文件夹将包含为应用程序实现的所有架构定义以及实体之间约束的所有定义。例如，在`Item`实体的情况下，我们需要创建一个新的`ItemEntitySchemaDefinition`类：

[PRE21]

这是`Item`实体约束的架构定义。该类实现了由`Microsoft.EntityFrameworkCore.SqlServer`包公开的`IEntityTypeConfiguration<T>`接口。必须通过以下命令向`Catalog.Infrastructure`项目添加一个新的引用：

[PRE22]

该包提供了一个扩展方法来与SQL服务器数据库交互：`Configure`方法实现定义了规则，这些规则将应用于`Item`实体。由于Fluent API方法，这些规则很容易理解：

+   `ToTable`方法用于显式定义SQL表名。

+   `HasKey`方法将属性设置为该实体类型的键。

+   `IsRequired`方法用于标记所有必需的功能。

EF Core为我们提供了不同的**开箱即用**配置选项；完整的列表可在[https://docs.microsoft.com/en-us/ef/core/modeling/](https://docs.microsoft.com/en-us/ef/core/modeling/)找到。这些属性可以组合起来，以获得关于正确表示我们的领域模型更好的结果。

`Configure`方法还向`Item`实体与`Artist`实体和`Genre`实体之间的**一对多**关系添加了一些额外的约束：

[PRE23]

此代码片段指定了`Item`实体的关系。请注意，我们遵循了Fluent方法。在这种情况下，我们通过指定`GenreId`和`ArtistId`作为外键，在`Item`类和`Artist`类以及`Genre`类之间定义了一个1-N关系。

# 使用Fluent API进行自定义转换。

EF Core还提供了一种添加自定义转换的方法。这种方法可能对于提供复杂实体的自定义表示很有用。例如，让我们看看在`ItemEntitySchemaDefinition`中声明的以下代码片段：

[PRE24]

`HasConversion`方法提供了一种自定义数据库中插入数据的方式。此方法通过以下格式将`Price`字段（`Price`类型）序列化为字符串：`34.05:EUR`。另一方面，当从数据库中读取`Price`数据时，字符串被反序列化为`Price`类型。

# 在当前数据上下文中应用架构定义

要利用`ItemEntitySchemaDefinition`类中实现的架构，我们应该将其应用于`CatalogContext`类中包含的`OnModelCreating`方法：

[PRE25]

上述代码使用`ApplyConfiguration`扩展方法在运行时执行期间将配置应用到SQL架构。需要注意的是，类中实现的`OnModelCreating`方法始终调用父类的`base.OnModelCreating`方法，以保留扩展类的行为。

# 为艺术家和流派实体生成架构

上述过程也可以应用于`Artist`和`Genre`实体。以下代码显示了`Catalog.Domain.Entities`命名空间中两个实体的定义：

[PRE26]

因此，我们可以将两个文件添加到`Catalog.Infrastructure`项目中，如下所示：

[PRE27]

`GenreEntitySchemaConfiguration.cs`文件如下所示：

[PRE28]

`GenreEntitySchemaConfiguration`和`ArtistEntitySchemaConfiguration`都使用`HasKey`方法定义了我们表的主键。正如我们之前讨论的，它们使用了应用于先前定义的`ItemEntitySchemaConfiguration`类的相同流畅方法。此外，我们还需要将`GenreEntitySchemaConfiguration`和`ArtistEntitySchemaConfiguration`包含在`CatalogContext`类的`OnModelCreating`方法中：

[PRE29]

为了简洁，我省略了`CatalogContext`类的完整定义。显著的变化是扩展了`OnModelCreating`方法，通过应用`GenreEntitySchemaConfiguration`和`ArtistEntitySchemaConfiguration`类的配置。

# 执行迁移

使用EF Core实现数据访问的最后一个步骤是将`DbContext`实例连接到数据库，并使用.NET CLI公开的命令运行迁移。在这样做之前，我们需要在我们的本地环境中有一个可工作的数据库。为了使我们的本地开发环境尽可能轻量，此示例将使用Linux上的Microsoft SQL Server Docker镜像。您可以从这里获取Docker镜像：[https://hub.docker.com/r/microsoft/mssql-server-linux/](https://hub.docker.com/r/microsoft/mssql-server-linux/)。如果您没有任何Docker的先前经验，可以遵循此指南在您的本地机器上安装和设置它：[https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-2017](https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-2017)。

容器是快速设置本地环境的一种极好方式，无需配置大量不同的工具和系统。如今，微软在简化他们的系统和流程方面投入了大量资金，无论是针对开发者还是云系统。

在运行我们的 SQL 实例后，让我们通过以下命令创建一个名为 `Store` 的新数据库：

[PRE30]

CLI 的一个有效替代方案是使用 SQL 编辑器。一个推荐的工具是 VS Code 的 `mssql` 扩展：[https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-develop-use-vscode?view=sql-server-2017](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-develop-use-vscode?view=sql-server-2017)。否则，您可以下载基于 VS Code 的跨平台 SQL 编辑器：[https://docs.microsoft.com/en-us/sql/azure-data-studio/download?view=sql-server-2017](https://docs.microsoft.com/en-us/sql/azure-data-studio/download?view=sql-server-2017)。

一旦我们在本地环境中使 Microsoft SQL Server 运行起来，我们就可以通过将我们的服务与数据库连接来继续操作。`Catalog.API` 项目中已经存在的 `Startup` 类将定义我们的服务使用的连接字符串和提供者。正如我们将看到的，所有迁移类也将存储在同一个项目中。这种方法保证了我们的 .NET CLI 指令有一个唯一的入口点，即 `Catalog.API`，而不与数据库逻辑（`Catalog.Infrastructure`）紧密耦合。

在继续之前，我们需要在 API 项目文件夹中使用以下命令将 `Catalog.Infrastructure` 项目添加为 API 项目的引用：

[PRE31]

API 项目还要求您引用 `Microsoft.EntityFrameworkCore.Design` NuGet 包，该包共享 EF Core 工具的设计时组件。我们可以通过在 `Catalog.API` 项目文件夹中执行以下 CLI 指令来添加包的最新版本：

[PRE32]

之后，我们可以在 `Startup` 类中添加数据库连接：

[PRE33]

`ConfigureServices` 方法包含了 SQL 连接的初始化。首先，它使用 `AddEntityFameworkSqlServer` 添加了 SQL 提供者所需的服务。随后，它添加了 `CatalogContext`，通过传递 `Action<DbContextOptionsBuilder>` 类型的动作方法来利用 `AddContext<T>` 泛型方法。

最后，动作方法通过使用 `UseSqlServer` 扩展方法和传递数据库的连接字符串来配置 SQL Server 提供者。`MigrationsAssembly` 方法定义了组件应该存储的位置。在这种情况下，它指定所有迁移都将存储在我们的 `Catalog.API` 项目中。

为了使我们的`Startup`类保持整洁和可读，我们可能需要创建一个自定义扩展方法来初始化对`Catalog`数据库的连接。让我们在`Catalog.API`项目中创建一个新的文件夹名为`Extensions`，添加一个新的`DatabaseExtension`类，并将我们的代码移动到一个新的`AddCatalogContext`方法中：

[PRE34]

我们可以简化`Startup`类如下：

[PRE35]

现在`Startup`类已经准备好了，请在`Catalog.API`项目文件夹中使用以下命令执行`migrations`：

[PRE36]

第一个命令生成了`Migration`文件夹以及其中的两个不同文件：

+   `{timestamp}_InitMigration.cs`：此类创建了数据库中存在的表、约束和索引。

+   `CatalogContextModelSnapshot.cs`：这个文件仅在第一次迁移命令中生成，并代表服务中实体的当前状态。

每个迁移类，包括我们刚刚生成的类，都具有以下结构：

[PRE37]

该类包含两个方法：`Up`和`Down`。`Up`方法在生成数据库模式时被调用。`Down`方法在删除模式时被调用。

生成的表和SQL实体位于`catalog`模式之下。每次我们执行以下命令时，`dotnet ef` CLI工具都会创建一个新的迁移类：

[PRE38]

每次我们在项目文件夹中运行EF Core更新过程时，数据库的模式都会被刷新。因此，我们可以在`Catalog.API`项目文件夹中执行以下CLI命令：

[PRE39]

前一个命令使用存储在项目`Migration`文件夹中的迁移创建了SQL模式：它将连接到`AddCatalogContext()`扩展方法中指定的数据库。在下一节中，我们将探讨如何将指定的连接字符串移动到`appsettings.json`文件中。

# 定义配置部分

如[第2章](6127f023-703b-42e3-a76e-b70a7b110b90.xhtml)“ASP.NET Core概述”中所述，`appsettings.json`文件通常包含应用程序设置。连接字符串通常存储在该文件中。因此，这种做法使我们的服务更具可重用性和可配置性，尤其是在它已经在预发布或生产环境中运行时。让我们以下述方式将连接字符串从`AddCatalogContext`方法移动到`appsettings.json`文件中：

[PRE40]

这样，我们可以使用以下语法读取连接字符串并将其作为参数传递给`AddCatalogContext`：

[PRE41]

因此，我们需要通过添加一个`connectionString`参数来更改`AddCatalogContext`扩展方法的签名，如下所示：

[PRE42]

我们可以将新定义的`connectionString`参数传递给`UseSqlServer`扩展方法。在下一节中，我们将继续测试本节中实现的仓库逻辑。

# 测试EF Core仓库

本节涵盖了用于测试.NET Core应用程序的一些常见测试实践。更具体地说，它侧重于测试应用程序的存储库部分。首先，让我们在项目根目录（与`.sln`文件相同的文件夹）中执行以下命令来创建一个新的测试项目：

[PRE43]

因此，我们创建了一个新的`tests`目录，它将包含服务中所有的测试项目。我们还使用`xunit`模板创建了一个新的`Catalog.Infrastructure.Tests`项目。

`xunit`是.NET生态系统中的一个非常流行的测试框架，并且是.NET Core框架模板中的默认选择。由于我们使用`xunit`模板创建了项目，因此`Catalog.Infrastructure.Tests.csproj`文件将包含对`xunit`包的引用：

[PRE44]

这些包允许我们通过在解决方案级别的测试项目文件夹中使用`dotnet test` CLI指令或在我们的首选IDE（如Visual Studio或Rider）中集成的测试运行器工具来运行单元测试。

# 使用DbContext进行数据种子

让我们继续探讨另一个EF Core功能，它允许我们进行数据种子。数据种子技术简化了测试环境，以便获取集成测试数据库的**默认**快照。

让我们通过一个.NET Core数据库种子示例来了解。首先，让我们创建一个新的`Data`文件夹，并添加包含测试记录的JSON文件。为了简洁，我在同一代码片段中包含了`artist.json`文件和`genre.json`文件。

[PRE45]

上述文件包含与`Genre`和`Artist`实体相关的数据。同样地，我们可以通过创建一个新的`item.json`文件来包含关于`Item`实体的信息：

[PRE46]

这些文件包含一些种子数据，需要在每次测试之前添加到我们的数据库中。为了读取它们，我们需要在`Catalog.Infrastructure.Tests`项目中包含`Newtonsoft.Json`包，使用以下命令在项目文件夹中执行：

[PRE47]

我们还应该确保在编译步骤中将JSON文件复制到`bin`文件夹中，通过在`Catalog.Infrastructure.Tests.csproj`中添加以下代码来实现：

[PRE48]

下一步是实现一个从JSON读取数据并将其序列化到我们的数据库上下文中的方法。我们还应该在测试项目中添加`Microsoft.EntityFrameworkCore` NuGet包，使用以下CLI命令：

[PRE49]

上述包将提供EF Core的`ModelBuilder`类型，该类型用于生成我们测试中使用的模拟数据。由于我们将使用`Catalog.Infrastructure`项目中实现的一些代码，我们应在解决方案根目录下使用以下命令将测试项目添加到引用中：

[PRE50]

之后，我们可以在`Catalog.Infrastructure.Tests`项目中的新`Extensions`文件夹内创建一个新的扩展方法，命名为`Seed<T>`。

[PRE51]

EF Core 2.1 通过公开 `HasData<T>` 方法引入了一种在数据库中执行数据播种的新方法。前面的代码允许我们读取 JSON 文件并将其序列化为由 `modelBuilder` 引用的实体。这种方法提供了一种使用 JSON 文件中写入的数据对模拟数据库进行播种的方法。

最后，我们可以在 `Catalog.Infrastructure.Tests` 项目中创建一个新的上下文，命名为 `TestCatalogContext`：

[PRE52]

在这里，`TestCatalogContext` 类扩展了位于 `Catalog.Infrastructure` 项目的 `CatalogContext` 类，并重写了 `OnModelCreating` 方法以在实体上调用 `Seed<T>` 扩展方法。因此，当消费者使用 `TestCatalogContext` 初始化数据库时，它将具有在 JSON 中编写的所有预填充数据。

注意这里，`TestCatalogContext` 在构造函数中扩展了 `DbContextOptions<CatalogContext>` 选项，以便初始化 `CatalogContext` 基类。

# 初始化测试类

让我们在 `Catalog.Infrastructure.Tests` 项目中创建一个新的测试类，命名为 `ItemRepositoryTests`：

[PRE53]

`Xunit` 框架使用 `Fact` 属性识别测试类。每个包含具有 `Fact` 属性的方法或，如本节稍后所示，具有 `Theory` 属性的类的类都将被视为测试，由单元测试运行器执行。

让我们继续添加我们的第一个测试方法。这个方法检查 `ItemRepository` 类的 `GetAsync` 方法：

[PRE54]

此代码使用 `DbContextOptionsBuilder<T>` 初始化一个新的 `Options` 对象，其类型为 `CatalogContext`。它还使用 `UseInMemoryDatabase` 扩展方法创建一个新的具有给定名称的内存数据库实例。由于 `DbContext` 由实现 `IAsyncDisposable` 类型的 `CatalogContext` 类扩展，因此可以使用 `await using var` 关键字。这种方法避免了任何类型的嵌套，并通过避免使用嵌套来提供更清晰的代码阅读方式：

[PRE55]

要构建代码，需要在 `Catalog.Infrastructure.Tests` 项目中添加以下包：

[PRE56]

`UseInMemoryDatabase` 扩展方法对于配置新的内存数据库实例很有用。需要注意的是，它不是设计为关系型数据库。此外，它不执行任何数据库完整性检查或约束检查。为了更合适的测试，我们应该使用 SQLite 的内存版本。您可以在以下文档中找到有关 SQLite 提供程序的更多信息：[https://docs.microsoft.com/en-us/ef/core/miscellaneous/testing/sqlite](https://docs.microsoft.com/en-us/ef/core/miscellaneous/testing/sqlite)。

在创建新的 `Options` 对象之后，`should_get_data` 方法创建了一个新的 `TestCatalogContext` 实例，并调用了 `EnsureCreated()` 方法，该方法确保上下文存在于内存数据库中。`EnsureCreate` 方法还隐式地调用了 `OnModelCreating` 方法。之后，测试通过上下文初始化了一个新的 `ItemRepository`，并执行了 `GetAsync` 方法。最后，它使用 `result.ShouldNotBeNull()` 检查结果。

注意，本书中的所有测试示例都使用了 `Shouldly` 作为断言框架。`Shouldly` 专注于在断言失败时提供简洁明了的错误信息。可以通过使用 .NET Core 内置的默认断言框架来避免使用 `Shouldly`。有关 `Shouldly` 的更多信息，请参阅以下链接：[https://github.com/shouldly/shouldly](https://github.com/shouldly/shouldly)。您可以在 `Catalog.Infrastructure.Tests` 项目中执行以下 CLI 指令来添加 `Shouldly` 包：`dotnet add package Shouldly`。

让我们继续实现 `ItemRepository` 类中所有方法的测试：

[PRE57]

前面的代码片段定义了覆盖 `GetAsync` 方法的测试。第一个方法 `should_get_data` 测试了没有参数的 `GetAsync()` 重载，而第二个方法测试了 `GetAsync(guid id)` 重载。在这两种情况下，我们使用 `InMemoryDatabase` 来模拟底层数据源。在同一个 `ItemRepositoryTests` 类中，也可以定义与创建/更新操作相关的测试用例：

[PRE58]

最后，`ItemRepositoryTests` 类为 `ItemRepository` 类实现的全部 CRUD 方法提供了测试覆盖率。`should_get_data`、`should_returns_null_with_id_not_present` 和 `should_return_record_by_id` 方法执行 `GetAsync` 方法并检查结果是否符合预期。`should_add_new_item` 和 `should_update_item` 测试用例为 `ItemRepository.Add` 和 `ItemRepository.Update` 方法提供了测试覆盖率。这两个测试都初始化了一个新的 `Item` 类型的记录，并通过 `ItemRepository` 类型公开的方法更新数据库。

因此，我们可以在 `Catalog.Infrastructure.Tests` 文件夹中执行以下命令来运行我们的测试：

[PRE59]

前面的命令执行了项目中实现的所有测试。因此，结果将是一个包含成功测试列表的报告。作为替代，我们也可以选择使用 IDE 提供的测试运行器来运行测试。现在我们已经使用 EF Core 和代码优先方法完成了数据访问部分的实现，我们也可以快速了解一下 Dapper，以及它如何通过提供一种更轻量级的数据访问方式来发挥作用。

# 使用 Dapper 实现数据访问层

另一个提供实现数据访问层方法的标准化工具是Dapper。我们已经对Dapper进行了概述，但本节将更详细地介绍如何处理这个包以及如何使用它来实现数据访问层。以下过程将更加侧重于SQL。我们还将演示如何处理一些存储CRUD过程。

注意，EF Core还提供了一种通过存储过程查询数据源的方法。此外，它公开了`DbSet<TEntity>.FromSql()`或`DbContext.Database.ExecuteSqlCommand()`等方法。那么，为什么使用Dapper呢？如前所述，Dapper是一个简单且比EF Core更快的微型ORM。EF更像是一个多用途ORM，它在每个操作上都会增加一些额外的开销。

在开始之前，让我们在`src`文件夹内创建另一个名为`Catalog.InfrastructureSP`的项目，通过在`src`文件夹内运行以下命令：

[PRE60]

在创建`Catalog.InfrastructureSP`项目之后，我们需要将其添加到我们的解决方案中：

[PRE61]

上述命令将`Catalog.InfrastructureSP`项目包含到解决方案中。一旦我们设置了包含所有数据访问层替代实现的新项目，我们就可以通过使用SQL-first方法来实现项目的核心部分。

# 创建存储CRUD过程

在当前示例中，我们使用了一些实现创建、读取和更新操作的存储过程。在这本书中，我们不会深入探讨SQL服务器编程模型，但理解代码-first方法不是唯一的方法是至关重要的。存储过程是实现服务与数据库之间交互的一种优秀方式。

存储过程是与数据库交互的最佳方式。开发者可以通过执行复杂查询并调用过程名称来继续操作。这种模块化方法在权限配置、更快的网络流量和更快的执行速度方面提供了一些好处。

首先，让我们创建用于读取数据的存储过程：

[PRE62]

代码的第一个片段定义了`GetAllItems`存储过程。它返回整个项目集合。出于演示目的，该过程不包括任何性能优化。当我们对一个包含大量记录的大表执行`select`查询时，至少需要插入一个top语句以避免长时间运行的查询和超时问题。此外，在现实世界的应用程序中，很少看到没有特定过滤器的查询。让我们继续创建`GetItemById`过程：

[PRE63]

这两个过程相当简单。第一个过程从`catalog.Item`表中选取所有记录。第二个过程接受一个`Id`作为参数，并允许我们检索相应的记录。

下一步是实现与创建和更新记录相关的操作。两种实现都非常简单——`InsertItem` 和 `UpdateItem` 存储过程封装了 `insert` 和 `update` SQL 语句：

[PRE64]

`InsertItem` 存储过程通过接受存储过程的参数来在数据库上执行简单的 `insert` 语句。让我们通过定义 `UpdateItem` 存储过程来继续：

[PRE65]

注意，这两个操作都使用 `output` 语句来检索执行结果中插入或更新的记录。这样，我们可以从我们的存储库模式中检索更新记录而无需额外努力。

Microsoft SQL Server 提供了一种使用 *output* 操作符返回插入或删除数据的方法。它返回由 `INSERT`、`UPDATE`、`DELETE` 或 `MERGE` 语句影响的每一行信息。

最后，为了使这些脚本正常工作，必须在我们的数据库中执行它们。我建议使用之前提到的 SQL Operations Studio 工具或另一个 SQL 客户端来在 `Catalog` 数据库中运行这些脚本。

# 实现 IItemRepository 接口

在 *使用 EF Core 实现数据访问层* 部分，我们使用了两个不同的接口来完成工作：`IItemRepository`，它包含所有 CRUD 操作，以及 `IUnitOfWork`，它涵盖了工作单元模式。对于每个 CRUD 操作，我们需要调用 `IUnitOfWork` 接口来将我们的更改保存到数据库中。另一方面，作为微 ORM 的 Dapper 应用不需要提供工作单元接口，因为 ORM 直接使用存储过程在数据库上执行查询。因此，我们不再需要实现 `IRepository` 接口，相应地，我们也不再实现 `IUnitOfWork` 接口。

因此，作为第一步，我们应该从我们的 `IItemRepository` 接口中移除 `IRepository` 接口的实现。此外，在这种情况下，我们可以看到依赖反转的真正力量：`Catalog.Domain` 不依赖于 `Catalog.Infrastructure`*.* 它也可以更改合同和需求，并迫使 `Catalog.Infrastructure` 改变其行为：

[PRE66]

下一步是将 Dapper 添加到我们的 `Catalog.InfrastructureSP` 项目中，通过执行以下命令：

[PRE67]

让我们通过在 `Catalog.InfrastructureSP` 项目中使用 `ItemRepository` 类来实现 `IItemRepository` 接口：

[PRE68]

为了初始化我们的具体类，必须在 `ItemRepository` 类的构造函数中将 `connectionString` 传递给 SQL 数据库。

如您所见，Dapper 方法与 EF Core 完全不同。它不会给我们的数据源添加任何特定的开销；它只是通过填充请求的参数来执行上述存储过程。

# 摘要

本章描述了如何使用EF Core和Dapper构建数据访问层。它还展示了如何使用内存数据库构建单元测试，以及如何使用EF Core执行迁移。我想重申，EF Core和Dapper之间的选择取决于不同的参数：我们正在构建的服务类型、团队成员的技能以及我们使用的类型基础设施。

本章涵盖的主题提供了使用代码优先和存储过程方法访问.NET Core数据源所需的知识。本章介绍了EF Core和Dapper等技术的使用。此外，它还展示了如何使用内存方法测试数据访问层。

在下一章中，我们将演示如何实现处理程序和我们的服务逻辑。
