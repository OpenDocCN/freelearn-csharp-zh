# *第六章*: 与数据访问基础设施协同工作

几乎所有业务应用程序都使用某种数据库系统。我们通常实现数据访问逻辑以从数据库中读取数据并向数据库写入数据。我们还需要处理数据库事务以确保数据源的一致性。

在本章中，我们将学习如何与 ABP 框架的数据访问基础设施协同工作，该基础设施通过实现**仓储**和**工作单元**（**UoW**）模式来提供数据访问的抽象。仓储提供了一种标准方式来执行针对你的**实体**的常见数据库操作。UoW 系统自动化数据库连接和事务管理，以确保用例（通常是**超文本传输协议**（**HTTP**）请求）是原子的；这意味着请求中执行的所有操作要么一起成功，要么在出现任何错误时一起回滚。

你将了解如何根据 ABP 框架预构建的基类定义你的实体。然后，你将学习如何使用仓储在数据库中插入、更新、删除和查询实体。你还将了解 UoW 系统，以控制应用程序中的事务作用域。

ABP 框架可以与任何数据库系统协同工作，同时它提供了内置的`DbContext`类集成包，将你的实体映射到数据库表，实现你的仓储，并在拥有实体时部署不同的加载相关实体的方式。你还将了解如何使用**MongoDB**作为第二个数据库提供者选项。

本章涵盖了 ABP 的基本数据访问基础设施，以下是一些主题：

+   定义实体

+   与仓储协同工作

+   EF Core 集成

+   MongoDB 集成

+   理解 UoW 系统

# 技术要求

如果你想要跟随并尝试示例，你需要安装一个**集成开发环境**（**IDE**）/编辑器（例如，Visual Studio）来构建 ASP.NET Core 项目。

你可以从以下 GitHub 仓库下载代码示例：[`github.com/PacktPublishing/Mastering-ABP-Framework`](https://github.com/PacktPublishing/Mastering-ABP-Framework)。

# 定义实体

实体是定义你的领域模型的主要类。如果你使用的是关系型数据库，实体通常映射到数据库表。**对象关系映射器**（**ORM**），如 EF Core，提供抽象，让你感觉在应用程序代码中与对象协同工作，而不是与数据库表协同工作。

ABP 框架通过提供一些接口和基类来标准化实体的定义。在接下来的章节中，你将了解 ABP 框架的`AggregateRoot`和`Entity`基类（及其变体），使用这些类使用单个**主键**（**PK**）和**复合主键**（**CPK**），以及与**全局唯一标识符**（**GUID**）PK 协同工作。

## AggregateRoot 类

**聚合**是一组由聚合根对象绑定在一起的对象（实体和值对象）的集合。

关系型数据库没有物理的聚合概念。每个实体都与一个单独的数据库表相关联，而聚合则分散在多个表中。你通过 **外键**（**FKs**）定义关系。然而，在文档/对象数据库（如 MongoDB）中，聚合作为一个单独的文档（类似于 **JavaScript 对象表示法**（**JSON**）的对象）进行序列化并保存到单个集合中。聚合根映射到集合，子实体在聚合根对象中进行序列化。这意味着子实体没有自己的集合，并且总是通过聚合根进行访问。

聚合概念

我们将在 *第十章*，*DDD – 领域层*中涵盖聚合概念，并详细介绍这一点。现在，你可以将聚合根视为你领域中的主要（根）实体。

在 ABP 框架中，你可以通过从 `AggregateRoot` 类之一派生来定义主要实体和聚合根。`BasicAggregateRoot` 是定义聚合根的最简单类。

以下示例实体类是从 `BasicAggregateRoot` 类派生出来的：

```cs
using System;
using System.Collections.Generic;
using Volo.Abp.Domain.Entities;
namespace FormsApp
{
    public class Form : BasicAggregateRoot<Guid>
    {
        public string Name { get; set; }
        public string Description { get; set; }
        public bool IsDraft { get; set; }
        public ICollection<Question> Questions { get; set; }
    }
}
```

`BasicAggregateRoot` 仅定义了一个 `Id` 属性作为主键，并将主键类型作为泛型参数。在这个例子中，`Form` 的主键类型是 `Guid`。你可以使用任何类型作为主键（例如，`int`、`string` 等），只要底层数据库提供程序支持即可。

这里有一些其他的基础类，你可以从中派生出你的聚合根，具体如下：

+   `AggregateRoot` 类具有额外的属性以支持乐观并发和对象扩展功能。

+   `CreationAuditedAggregateRoot` 类继承自 `AggregateRoot` 类，并添加了 `CreationTime` (`DateTime`) 和 `CreatorId` (`Guid`) 属性以存储创建审计信息。

+   `AuditedAggregateRoot` 类继承自 `CreationAuditedAggregateRoot` 类，并添加了 `LastModificationTime` (`DateTime`) 和 `LastModifierId` (`Guid`) 属性以存储修改审计信息。

+   `FullAuditedAggregateRoot` 类继承自 `AuditedAggregateRoot` 类，并添加了 `DeletionTime` (`DateTime`) 和 `DeleterId` (`Guid`) 属性以存储删除审计信息。它还通过实现 `ISoftDelete` 接口添加了 `IsDeleted` (`bool`)，这使得实体可以进行软删除。

    乐观并发和对象扩展功能

    这些主题在本书中没有涉及。如需使用，请查阅 ABP 框架文档。

ABP 自动设置审计属性。我们将在 *第八章*，*使用 ABP 的功能和服务*中返回审计日志和软删除主题。

## 实体类

`Entity`基类类似于`AggregateRoot`类，但它们用于子集合实体而不是主（根）实体。例如，上一节中的`Form`聚合根示例有一个问题集合。`Question`类是从`Entity`类派生的，并在以下代码片段中显示：

```cs
public class Question : Entity<Guid>
{
    public Guid FormId { get; set; }
    public string Title { get; set; }
    public bool AllowMultiSelect { get; set; }
    public ICollection<Option> Options { get; set; }
}
```

与`AggregateRoot`类一样，`Entity`类也定义了一个给定类型的`Id`属性。在这个例子中，`Question`实体还有一个选项集合，其中`Option`是另一种实体类型。

还有一些其他预定义的基本实体类，例如`CreationAuditedEntity`、`AuditedEntity`和`FullAuditedEntity`。它们与上一节中解释的已审计聚合根类类似。

## 具有 CPK 的实体

关系型数据库支持 CPKs，其中您的 PK 由多个值的组合组成。复合键对于具有**多对多**关系的关联表特别有用。

假设您想为表单对象设置多个管理器，并将集合属性添加到`Form`类中，如下所示：

```cs
public class Form : BasicAggregateRoot<Guid>
{
    ...
    public ICollection<FormManager> Managers { get; set; }
}
```

然后，您可以定义一个从非泛型的`Entity`类派生的`FormManager`类，如下所示：

```cs
public class FormManager : Entity
{
    public Guid FormId { get; set; }
    public Guid UserId { get; set; }
    public Guid IsOwner { get; set; }
    public override object[] GetKeys()
    {
        return new object[] {FormId, UserId};
    }
}
```

当您从非泛型的`Entity`类继承时，您必须实现`GetKeys`方法以返回键的数组。这样，ABP 可以在需要的地方使用 CPK 的值。例如，在这个例子中，`FormId`和`UserId`是其他表的 FK，它们构成了`FormManager`实体的 CPK。

聚合根的 CPK

`AggregateRoot`类也有非泛型的 CPK 版本，而对于聚合根实体设置 CPK 并不常见。

## GUID PK

ABP 主要使用 GUIDs 作为预建实体的 PK 类型。GUIDs 通常与自增 ID（如`int`或`long`，由关系型数据库支持）进行比较。以下是使用 GUIDs 作为 PK 与自增键相比的一些常见好处：

+   GUIDs 自然是唯一的。如果您正在构建分布式系统，使用非关系型数据库，并且需要拆分或合并表或集成外部系统，这效果很好。

+   GUIDs 可以在客户端生成，无需数据库往返。这样，客户端代码可以在保存实体之前知道 PK 值。

+   GUIDs 无法猜测，因此在某些情况下可以更安全（例如，如果最终用户看到实体的 ID，他们无法找到另一个实体的 ID）。

与自增整数值相比，GUIDs 也有一些缺点，如下所示：

+   GUID 在存储中占用 16 字节，高于`int`（4 字节）和`long`（8 字节）。

+   GUIDs 在本质上不是顺序的，这会导致在聚类索引上出现性能问题。然而，ABP 提供了解决这个问题的方案。

ABP 提供了`IGuidGenerator`服务，该服务默认生成顺序的`Guid`值。虽然它生成顺序值，但算法生成的值仍然是通用和随机的。生成顺序值解决了聚集索引性能问题。

如果你手动设置实体的`Id`值，始终使用`IGuidGenerator`服务；永远不要使用`Guid.NewGuid()`。如果你没有为新的实体设置`Id`值并使用仓库将其插入数据库，仓库将自动使用`IGuidGenerator`服务设置它。

GUID 与自增的比较

在软件开发中，GUID 与自增 PK 的讨论很热烈，但没有明确的赢家。ABP 与任何 PK 类型都兼容，因此你可以根据自己的需求做出选择。

我们现在已经学习了实体定义的基础知识，并将探讨在*第十章*，*领域驱动设计 - 领域层*中的实体最佳实践。但现在，让我们继续探讨仓库，了解如何与数据库一起工作以持久化我们的实体。

# 与仓库一起工作

**仓库模式**是一种常见的抽象数据访问代码的方法，从应用程序的其他服务中分离出来。在接下来的章节中，你将学习如何使用 ABP 框架的通用仓库为你的实体查询或操作数据库中的数据，使用预定义的仓库方法。你还将看到如何在需要扩展通用仓库并添加自己的仓库方法以封装你的数据访问逻辑时创建自定义仓库。

集成数据库提供者

为了使用仓库，需要完成数据库提供者的集成。我们将在本章的*EF Core 集成*和*MongoDB 集成*部分完成此操作。

## 通用仓库

一旦你有一个实体，你可以直接注入并使用该实体的通用仓库。以下是一个使用仓库的示例类：

```cs
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Volo.Abp.DependencyInjection;
using Volo.Abp.Domain.Repositories;
namespace FormsApp
{
    public class FormService : ITransientDependency
    {
        private readonly IRepository<Form, Guid>                         _formRepository;
        public FormService(IRepository<Form, Guid>                       formRepository)
        {
            _formRepository = formRepository;
        }
        public async Task<List<Form>> GetDraftForms()
        {
            return await _formRepository
                .GetListAsync(f => f.IsDraft);
        }
    }
}
```

在本例中，我们注入了`IRepository<Form, Guid>`，这是`Form`实体的默认通用仓库。然后，我们使用了`GetListAsync`方法从数据库中获取表单的筛选列表。通用的`IRepository`接口有两个泛型参数：实体类型（在本例中为`Form`）和 PK 类型（在本例中为`Guid`）。

非聚合根实体的仓库

默认情况下，通用仓库仅适用于*聚合根*实体，因为这是一种最佳实践，通过聚合根对象访问聚合。然而，如果你使用的是关系数据库，你可以为其他实体类型启用通用仓库。我们将在*EF Core 集成*部分看到配置点。

通用仓库提供了许多内置方法来查询、插入、更新和删除实体。

### 插入、更新和删除实体

以下方法可用于在数据库中操作数据：

+   `InsertAsync`用于插入新实体。

+   `InsertManyAsync`用于在一次调用中插入多个实体。

+   `UpdateAsync`用于更新现有实体。

+   `UpdateManyAsync`用于在一次调用中更新多个实体。

+   `DeleteAsync`用于删除现有实体。

+   `DeleteManyAsync`用于在一次调用中插入多个实体。

    关于异步编程

    所有仓库方法都是异步的。在.NET 中，强烈建议尽可能使用`async`/`await`模式编写应用程序代码，因为在.NET 中，将异步代码与同步代码混合会导致潜在的死锁、超时和可伸缩性问题，这些问题难以检测和解决。

如果您使用 EF Core，这些方法可能不会立即执行实际的数据库操作，因为 EF Core 使用更改跟踪系统。它仅在您调用`DbContext.SaveChanges`方法时保存更改。ABP 框架的 UoW 系统会在当前 HTTP 请求成功完成后自动调用`SaveChanges`方法。如果您想立即将更改保存到数据库中，可以将`autoSave`参数传递给仓库方法并设置为`true`。

以下示例在`InsertAsync`方法中创建一个新的`Form`实体，并将其立即保存到数据库中：

```cs
var form = new Form(); // TODO: set the form properties
await _formRepository.InsertAsync(form, autoSave: true);
```

即使您将更改保存到数据库中，这些更改可能也不会立即可见，这取决于事务隔离级别，如果当前事务失败，则更改将被回滚。我们将在本章的*理解 UoW 系统*部分中介绍 UoW 系统。

`DeleteAsync`方法有一个额外的重载版本，用于删除满足给定条件的所有实体。以下示例展示了如何删除数据库中所有草稿表单：

```cs
await _formRepository.DeleteAsync(form => form.IsDraft);
```

您也可以使用逻辑运算符，如`&&`和`||`，来设置复杂条件。

关于取消令牌

所有仓库方法都接受一个可选的`CancellationToken`参数。取消令牌用于在需要时取消数据库操作。例如，如果用户关闭浏览器窗口，就没有必要继续长时间运行的数据库查询操作。大多数情况下，您不需要手动传递取消令牌，因为 ABP 框架在您没有明确传递取消令牌时，会自动捕获并使用 HTTP 请求中的取消令牌。

### 查询单个实体

以下方法可以用来获取单个实体：

+   `GetAsync`：通过其`Id`值或谓词表达式返回单个实体。如果请求的实体未找到，则抛出`EntityNotFoundException`。

+   `FindAsync`：通过其`Id`值或谓词表达式返回单个实体。如果请求的实体未找到，则返回`null`。

如果您有自定义逻辑或回退代码，并且给定的实体在数据库中不存在，则应仅使用`FindAsync`方法。否则，请使用`GetAsync`，它会在 HTTP 请求中抛出一个已知的异常，导致返回`404`状态码给客户端。

以下示例使用 `GetAsync` 方法查询具有其 `Id` 值的 `Form` 实体：

```cs
public async Task<Form> GetFormAsync(Guid formId)
{
    return await _formRepository.GetAsync(formId);
}
```

两种方法都有重载，可以传递一个谓词表达式来查询具有给定条件的实体。以下示例使用 `GetAsync` 方法获取具有其唯一名称的 `Form` 实体：

```cs
public async Task<Form> GetFormAsync(string name)
{
    return await _formRepository
        .GetAsync(form => form.Name == name);
}
```

仅在你期望单个实体时使用这些重载。如果你的查询返回多个实体，则它们会抛出 `InvalidOperationException`。例如，如果你的系统中表单名称总是 *唯一的*，你可以按名称查找表单，如下例所示。然而，如果你的查询可能返回多个实体，请使用返回实体列表的查询方法。

### 查询实体列表

通用存储库提供了许多从数据库查询实体的选项。以下方法可以直接用于获取实体列表：

+   `GetListAsync`：返回满足给定条件的所有实体或实体列表

+   `GetPagedListAsync`：用于分页查询实体

以下代码块显示了如何通过给定的名称获取过滤后的表单列表：

```cs
public async Task<List<Form>> GetFormsAsync(string name)
{
    return await _formRepository
        .GetListAsync(form => form.Name.Contains(name));
}
```

我已将一个 lambda 表达式传递给 `GetListAsync` 方法，以获取所有具有给定 `name` 参数值包含在其名称中的 `Form` 实体。

这些方法简单但有限。如果你想编写高级查询，你可以在存储库上使用 **语言集成查询**（**LINQ**）。

### 在存储库上使用 LINQ

存储库提供了 `GetQueryableAsync()` 方法，它返回一个 `IQueryable<TEntity>` 对象。然后你可以使用此对象在数据库中的实体上执行 LINQ。

以下示例使用 LINQ 操作在 `Form` 实体上获取按名称过滤和排序的表单列表：

```cs
public class FormService2 : ITransientDependency
{
    private readonly IRepository<Form, Guid>                         _formRepository;
    private readonly IAsyncQueryableExecuter                         _asyncExecuter;
    public FormService2(
        IRepository<Form, Guid> formRepository,
        IAsyncQueryableExecuter asyncExecuter)
    {
        _formRepository = formRepository;
        _asyncExecuter = asyncExecuter;
    }

    public async Task<List<Form>>                                   GetOrderedFormsAsync(string name)
    {
        var queryable = await                                           _formRepository.GetQueryableAsync();
        var query = from form in queryable
            where form.Name.Contains(name)
            orderby form.Name
            select form;
        return await _asyncExecuter.ToListAsync(query);
    }
}
```

我们首先获取了一个 `IQueryable<Form>` 对象，然后编写了一个 LINQ 查询，最后使用 `IAsyncQueryableExecuter` 服务执行了查询。

另一种编写先前查询的替代方法可以是使用 LINQ 扩展方法，如下所示：

```cs
var query = queryable
    .Where(form => form.Name.Contains(name))
    .OrderBy(form => form.Name);
```

拥有一个 `IQueryable` 对象为你提供了 LINQ 的所有功能。你甚至可以在来自不同存储库的多个 `IQueryable` 对象之间进行连接。

使用 `IAsyncQueryableExecuter` 服务可能对你来说有些奇怪。你可能期望直接在查询对象上调用 `ToListAsync` 方法，如下所示：

```cs
return await query.ToListAsync();
```

不幸的是，`ToListAsync` 是由 EF Core（或如果你使用 MongoDB，则位于）定义的扩展方法，位于 `Microsoft.EntityFrameworkCore` NuGet 包内。如果你从应用程序层引用该包没有问题，那么你可以在代码中直接使用这些异步扩展方法。然而，如果你想保持应用程序层 ORM 依赖性，ABP 的 `IAsyncQueryableExecuter` 服务提供了必要的抽象。

### `IRepository` 异步扩展方法

ABP 框架为 `IRepository` 接口提供了所有标准的异步 LINQ 扩展方法：`AllAsync`, `AnyAsync`, `AverageAsync`, `ContainsAsync`, `CountAsync`, `FirstAsync`, `FirstOrDefaultAsync`, `LastAsync`, `LastOrDefaultAsync`, `LongCountAsync`, `MaxAsync`, `MinAsync`, `SingleAsync`, `SingleOrDefaultAsync`, `SumAsync`, `ToArrayAsync`, 和 `ToListAsync`。您可以直接在存储库对象上使用这些方法中的任何一个。

以下示例使用 `CountAsync` 方法获取以 `"A"` 开头的表单的数量：

```cs
public async Task<int> GetCountAsync()
{
    return await _formRepository
        .CountAsync(x => x.Name.StartsWith("A"));
}
```

注意，这些扩展方法仅在 `IRepository` 接口上可用。如果您想使用可查询的扩展，您仍然应该遵循上一节中解释的方法。

### 具有复合主键（CPK）的实体的泛型存储库

如果您的实体有一个复合主键（CPK），您不能使用 `IRepository<TEntity, TKey>` 接口，因为它获取一个单一的主键（`Id`）类型。在这种情况下，您可以使用 `IRepository<TEntity>` 接口。

例如，您可以使用 `IRepository<FormManager>` 获取给定表单的管理员，如下所示：

```cs
public class FormManagementService : ITransientDependency
{
    private readonly IRepository<FormManager>                       _formManagerRepository;
    public FormManagementService(
        IRepository<FormManager> formManagerRepository)
    {
        _formManagerRepository = formManagerRepository;
    }
    public async Task<List<FormManager>>                             GetManagersAsync(Guid formId)
    {
        return await _formManagerRepository
            .GetListAsync(fm => fm.FormId == formId);
    }
}
```

在此示例中，我使用了 `IRepository<FormManager>` 接口对 `FormManager` 实体执行查询。

非聚合根实体的存储库

如本章中“泛型存储库”部分所述，默认情况下不能使用 `IRepository<FormManager>`，因为 `FormManager` 不是一个聚合根实体。通常，您想要获取 `Form` 聚合根并访问其 `Managers` 集合以获取表单管理器。但是，如果您使用 EF Core，可以为不是聚合根的实体创建默认的泛型存储库。请参阅“EF Core 集成”部分了解如何这样做。

没有提供 `TKey` 泛型参数的泛型存储库有一个限制，就是它们没有获取 `Id` 参数的方法，因为它们不知道 `Id` 的类型。然而，您仍然可以使用 LINQ 编写任何需要的查询。

### 其他泛型存储库类型

您通常想要使用上一节中解释的存储库接口，因为它们是最功能丰富的存储库类型。然而，还有一些更有限的存储库类型在某些场景中可能很有用，例如以下内容：

+   `IBasicRepository<TEntity, TPrimaryKey>` 和 `IBasicRepository<TEntity>` 提供了基本的存储库方法，但它们不支持 LINQ 和 `IQueryable` 功能。如果您的底层数据库提供程序不支持 LINQ 或您不想将 LINQ 查询泄露到应用程序层，则可以使用这些存储库。在这种情况下，您可能需要通过继承这些接口并使用自定义方法实现查询来编写自定义存储库。

+   `IReadOnlyRepository<TEntity, TKey>`, `IReadOnlyRepository<TEntity>`, `IReadOnlyBasicRepository<Tentity, TKey>`, 和 `IReadOnlyBasicRepository<TEntity, TKey>` 提供了获取数据的方法，但不包括任何操作数据库的方法。

通用仓库方法对于大多数情况已经足够。然而，您可能仍然需要向您的仓库中添加自定义方法。

## 自定义仓库

您可以创建自定义仓库接口和类来访问底层数据库提供者的**应用程序编程接口**（**API**），封装您的 LINQ 表达式，调用存储过程，等等。

要创建一个自定义仓库，首先，定义一个新的仓库接口。仓库接口在启动模板提供的`Domain`项目中定义。您可以从一个通用仓库接口继承以将标准方法包含在您的仓库接口中。以下代码片段展示了如何实现：

```cs
public interface IFormRepository : IRepository<Form, Guid>
{
    Task<List<Form>> GetListAsync(
        string name,
        bool includeDrafts = false
    );
}
```

`IFormRepository`继承自`IRepository<Form, Guid>`并添加了一个新方法来获取具有某些筛选条件的表单列表。然后，您可以将`IFormRepository`注入到您的服务中，而不是使用通用仓库，并使用您自定义的方法。如果您不想包含标准仓库方法，只需从`IRepository`（不带任何泛型参数）接口派生您的接口。这是一个空接口，用于标识您的接口作为仓库。

当然，我们必须在我们的应用程序的某个地方实现`IFormRepository`接口。ABP 启动模板为底层数据库提供者提供了集成项目，因此我们可以在数据库集成项目中实现自定义仓库接口。在下一节中，我们将为 EF Core 和 MongoDB 实现该接口。

# EF Core 集成

微软的 EF Core 是.NET 的事实上的 ORM，您可以使用它与主要的数据库提供者一起工作，例如 SQL Server、Oracle、MySQL、PostgreSQL 和 Cosmos DB。当您使用 ABP **命令行界面**（**CLI**）创建新的 ABP 解决方案时，它是默认的数据库提供者。

启动模板默认使用*SQL Server*。如果您在创建新解决方案时更喜欢另一个`-dbms`参数，如下所示：

```cs
abp new DemoApp -dbms PostgreSQL
```

`SqlServer`、`MySQL`、`SQLite`、`Oracle`和`PostgreSQL`直接支持。

其他数据库

您可以参考 ABP 的文档来了解最新的支持数据库选项以及如何切换到 ABP CLI 默认不支持的另一个数据库提供者：[`docs.abp.io/en/abp/latest/Entity-Framework-Core-Other-DBMS`](https://docs.abp.io/en/abp/latest/Entity-Framework-Core-Other-DBMS)。

在下一节中，您将学习如何配置 DBMS（尽管它已经在启动模板中完成），定义一个`DbContext`类，并将其注册到**依赖注入**（**DI**）系统中。然后，您将看到如何使用 Code First Migrations 将您的实体映射到数据库表中，为您的实体创建自定义仓库。最后，我们将探讨为实体加载相关数据的不同方法。

## 配置 DBMS

我们使用 `AbpDbContextOptions` 在模块的 `ConfigureServices` 方法中配置 DBMS。以下示例配置使用 SQL Server 作为 DBMS：

```cs
Configure<AbpDbContextOptions>(options =>
{
    options.UseSqlServer();
});
```

当然，如果您选择了不同的 DBMS，`UseSqlServer()` 方法调用将会有所不同。我们不需要设置连接字符串，因为它会自动从 `ConnectionStrings:Default` 配置中获取。您可以在项目的 `appsettings.json` 文件中查看并更改连接字符串。

我们已经配置了 DBMS，但尚未定义 `DbContext` 对象，这是在 EF Core 中与数据库工作所必需的。

## 定义 DbContext

`DbContext` 是 EF Core 中您与之交互数据库的主要对象。您通常创建一个继承自 `DbContext` 的类来创建自己的 `DbContext`。在 ABP 框架中，我们继承自 `AbpDbContext` 而不是 `DbContext`。

这里是一个使用 ABP 框架的 `DbContext` 类定义示例：

```cs
using Microsoft.EntityFrameworkCore;
using Volo.Abp.EntityFrameworkCore;
namespace FormsApp
{
    public class FormsAppDbContext :                                 AbpDbContext<FormsAppDbContext>
    {
        public DbSet<Form> Forms { get; set; }
        public FormsAppDbContext(
            DbContextOptions<FormsAppDbContext> options)
            : base(options)
        {
        }
    }
}
```

`FormsAppDbContext` 继承自 `AbpDbContext<FormsAppDbContext>`。`AbpDbContext` 是一个泛型类，它将 `DbContext` 类型作为泛型参数。它还强制我们创建一个构造函数，如下所示。然后我们可以为我们的实体添加 `DbSet` 属性。添加 `DbSet` 属性是必要的，因为 ABP 只能为具有定义了 `DbSet` 属性的实体创建默认的泛型仓储。

一旦我们定义了 `DbContext`，我们就应该将其注册到 DI 系统中，以便在应用程序中使用它。

## 将 DbContext 注册到 DI

使用 `AddAbpDbContext` 扩展方法将 `DbContext` 类注册到 DI 系统中。您可以在模块的 `ConfigureServices` 方法中使用此方法（它位于启动解决方案中的 `EntityFrameworkCore` 项目内），如下面的代码块所示：

```cs
public override void ConfigureServices(
    ServiceConfigurationContext context)
{
    context.Services.AddAbpDbContext<FormsAppDbContext>              (options =>
    {
        options.AddDefaultRepositories();
    });
}
```

使用 `AddDefaultRepositories()` 启用与该 `DbContext` 相关的实体的默认泛型仓储。默认情况下，它只为聚合根实体启用泛型仓储，因为如果您想为其他实体类型也使用仓储，可以将 `includeAllEntities` 参数设置为 `true`，如下所示：

```cs
options.AddDefaultRepositories(includeAllEntities: true);
```

使用此选项，您可以在应用程序代码中注入任何实体的 `IRepository` 服务。

启动模板中的 includeAllEntities 选项

ABP 启动模板将 `includeAllEntities` 选项设置为 `true`，因为从事关系数据库开发的开发者习惯于查询所有数据库表。如果您想严格应用 DDD 原则，您应该始终使用聚合根来访问子实体。在这种情况下，您可以从 `AddDefaultRepositories` 方法调用中删除此选项。

我们已经看到了如何注册 `DbContext` 类。我们可以在 `DbContext` 类中注入并使用所有实体的 `IRepository` 接口。然而，我们首先应该配置实体的 EF Core 映射。

## 配置实体映射

EF Core 是一个对象关系映射器，它将你的实体映射到数据库表。我们可以通过以下两种方式配置这些映射的细节，如下所述：

+   在你的实体类上使用数据注释属性

+   通过重写 `OnModelCreating` 方法使用 Fluent API

使用数据注释属性会使你的领域层依赖于 EF Core。如果你不介意这个问题，你可以简单地遵循 EF Core 的文档来使用这些属性。在这本书中，我将使用 Fluent API 方法。

要使用 Fluent API 方法，你可以在你的 `DbContext` 类中重写 `OnModelCreating` 方法，如下面的代码块所示：

```cs
public class FormsAppDbContext : AbpDbContext<FormsAppDbContext>
{
    ...
    protected override void OnModelCreating(ModelBuilder             builder)
    {
        base.OnModelCreating(builder);
        // TODO: configure entities...
    }
}
```

当你重写 `OnModelCreating` 方法时，始终要调用 `base.OnModelCreating()`，因为 ABP 也会在该方法内部执行默认配置，这对于正确使用 ABP 功能（如审计日志和数据过滤器）是必要的。然后，你可以使用 `builder` 对象来执行你的配置。

例如，我们可以配置本章中定义的 `Form` 类的映射，如下所示：

```cs
builder.Entity<Form>(b =>
{
    b.ToTable("Forms");
    b.ConfigureByConvention();
    b.Property(x => x.Name)
        .HasMaxLength(100)
        .IsRequired();
    b.HasIndex(x => x.Name);
});
```

在重写 `OnModelCreating` 方法时调用 `b.ConfigureByConvention()` 方法很重要。如果实体是从 ABP 预定义的 `Entity` 或 `AggregateRoot` 类派生的，它将配置实体的基本属性。剩余的配置代码相当干净和标准，你可以从 EF Core 的文档中学习所有详细信息。

这里还有一个配置实体之间关系的示例：

```cs
builder.Entity<Question>(b =>
{
    b.ToTable("FormQuestions");
    b.ConfigureByConvention();
    b.Property(x => x.Title)
        .HasMaxLength(200)
        .IsRequired();
    b.HasOne<Form>()
        .WithMany(x => x.Questions)
        .HasForeignKey(x => x.FormId)
        .IsRequired();
});
```

在这个示例中，我们正在定义 `Form` 和 `Question` 实体之间的关系：一个表可以有多个问题，而一个问题始终属于单个表。

我们所做的配置确保了 EF Core 知道如何读取和写入数据库表中的实体。然而，数据库中的相关表也应该可用。你当然可以手动创建数据库及其内部的表。然后，在实体发生每次变化时，你都需要手动在数据库架构中反映相关的更改。然而，以这种方式保持实体和数据库表的一致性是困难的。如果拥有多个环境（如开发和生产），手动进行所有这些操作既繁琐又容易出错。

幸运的是，有一种更好的方法：Code First Migrations。EF 的 Code First Migrations 系统提供了一种有效的方法，可以增量更新数据库架构，使其与你的实体模型保持同步。我们已经在 *第三章*，*逐步应用开发* 中使用了 Code First Migration 系统。你可以参考该章节来学习如何添加新的数据库迁移并在数据库中应用它。

## 实现自定义仓储

我们在本章 *与仓储一起工作* 部分的 *自定义仓储* 部分创建了一个 `IFormRepository` 接口。现在，是时候使用 EF Core 来实现这个仓储接口了。

你可以在你的解决方案的 EF Core 集成项目中实现仓库，如下所示：

```cs
public class FormRepository :
    EfCoreRepository<FormsAppDbContext, Form, Guid>,
    IFormRepository
{
    public FormRepository(
        IDbContextProvider<FormsAppDbContext>                           dbContextProvider)
        : base(dbContextProvider)
    { }
    public async Task<List<Form>> GetListAsync(
        string name, bool includeDrafts = false)
    {
        var dbContext = await GetDbContextAsync();
        var query = dbContext.Forms
            .Where(f => f.Name.Contains(name));
        if (!includeDrafts)
        {
            query = query.Where(f => !f.IsDraft);
        }
        return await query.ToListAsync();
    }
}
```

这个类是从 ABP 的 `EfCoreRepository` 类派生出来的。这样，我们就继承了所有标准仓库方法。`EfCoreRepository` 类获取三个泛型参数：`DbContext` 类型、实体类型和实体类的 PK 类型。

`FormRepository` 还实现了 `IFormRepository`，它定义了一个自定义的 `GetListAsync` 方法。我们在该方法中使用 `DbContext` 实例来利用 EF Core API 的所有功能。

关于 WhereIf 的提示

条件过滤是一个广泛使用的模式，ABP 提供了一个很好的 `WhereIf` 扩展方法，可以简化我们的代码。

我们可以重写 `GetListAsync` 方法，如下面的代码块所示：

```cs
var dbContext = await GetDbContextAsync();
return await dbContext.Forms
    .Where(f => f.Name.Contains(name))
    .WhereIf(!includeDrafts, f => !f.IsDraft)
    .ToListAsync();
```

由于我们有 `DbContext` 实例，我们可以使用它来执行 **结构化查询语言**（**SQL**）命令或存储过程。以下方法执行一个原始 SQL 命令来删除所有草稿表单：

```cs
public async Task DeleteAllDraftsAsync()
{
    var dbContext = await GetDbContextAsync();
    await dbContext.Database
        .ExecuteSqlRawAsync("DELETE FROM Forms WHERE                     IsDraft = 1");
}
```

执行存储过程和函数

您可以参考 EF Core 的文档（[`docs.microsoft.com/en-us/ef/core`](https://docs.microsoft.com/en-us/ef/core)）来了解如何执行存储过程和函数。

一旦实现了 `IFormRepository`，你就可以注入并使用它，而不是使用 `IRepository<Form, Guid>`，如下所示：

```cs
public class FormService : ITransientDependency
{
    private readonly IFormRepository _formRepository;
    public FormService(IFormRepository formRepository)
    {
        _formRepository = formRepository;
    }

    public async Task<List<Form>> GetFormsAsync(string               name)
    {
        return await _formRepository
            .GetListAsync(name, includeDrafts: true);
    }
}
```

这个类使用了 `IFormRepository` 的自定义 `GetListAsync` 方法。

即使你为 `Form` 实体实现了自定义仓库类，仍然可以注入并使用该实体的默认泛型仓库（例如，`IRepository<Form, Guid>`）。这是一个很好的特性，特别是如果你从泛型仓库开始，然后决定稍后创建自定义仓库。你不需要更改使用泛型仓库的现有代码。

如果你在你的仓库中覆盖了 `EfCoreRepository` 类的基方法并进行了自定义，可能会出现一个潜在问题。在这种情况下，使用泛型仓库引用的服务将继续使用未覆盖的方法。为了防止这种碎片化，在将 `DbContext` 注册到 DI 时使用 `AddRepository` 方法，如下所示：

```cs
context.Services.AddAbpDbContext<FormsAppDbContext>(options =>
{
    options.AddDefaultRepositories();
    options.AddRepository<Form, FormRepository>();
});
```

使用此配置，`AddRepository` 方法将泛型仓库重定向到你的自定义仓库类。

## 加载相关数据

如果你的实体有指向其他实体的导航属性或具有其他实体的集合，那么在处理主实体时，你将经常需要访问这些相关实体。例如，之前引入的 `Form` 实体有一个 `Question` 实体的集合，你可能需要在处理 `Form` 对象时访问这些问题。

有多种方式访问相关实体：**显式加载**、**延迟加载**和**贪婪加载**。

### 显式加载

仓库提供了 `EnsurePropertyLoadedAsync` 和 `EnsureCollectionLoadedAsync` 扩展方法来显式加载导航属性或子集合。

例如，我们可以显式地加载表单的问题，如下面的代码块所示：

```cs
public async Task<IEnumerable<Question>> GetQuestionsAsync(Form form)
{
    await _formRepository
        .EnsureCollectionLoadedAsync(form, f =>                         f.Questions);
    return form.Questions;
}
```

如果我们不在这里使用`EnsureCollectionLoadedAsync`，那么`form.Questions`集合可能会为空。如果我们不确定它是否已填充，我们可以使用`EnsureCollectionLoadedAsync`来确保它被加载。如果相关的属性或集合已经加载，`EnsurePropertyLoadedAsync`和`EnsureCollectionLoadedAsync`方法不会做任何事情，因此多次调用它们对性能没有问题。

### 懒加载

懒加载是 EF Core 的一个功能，当你第一次访问它们时，它会加载相关的属性和集合。懒加载默认是禁用的。如果你想为你的`DbContext`启用它，请按照以下步骤操作：

1.  在你的 EF Core 层中安装`Microsoft.EntityFrameworkCore.Proxies` NuGet 包。

1.  在配置`AbpDbContextOptions`时使用`UseLazyLoadingProxies`方法，如下所示：

    ```cs
    Configure<AbpDbContextOptions>(options =>
    {
        options.PreConfigure<FormsAppDbContext>(opts =>
        {
            opts.DbContextOptions.UseLazyLoadingProxies();
        });
        options.UseSqlServer();
    });
    ```

1.  确保你的实体中的导航属性和集合属性是虚拟的，如下所示：

    ```cs
    public class Form : BasicAggregateRoot<Guid>
    {
        ...
        public virtual ICollection<Question> Questions {           get; set; }
        public virtual ICollection<FormManager> Owners {            get; set; }
    }
    ```

当你启用懒加载时，你不再需要使用显式加载。

懒加载是 ORMs 中讨论的一个概念。一些开发者认为它有用且实用，而另一些开发者建议根本不要使用它。我倾向于不使用它，因为它有一些潜在的问题，例如这些：

+   懒加载不能使用异步编程，因为没有方法可以通过`async`/`await`模式访问属性。因此，它会阻塞调用线程，这对吞吐量和可扩展性来说是不良的做法。

+   如果你忘记在使用`foreach`循环之前预先加载相关数据，你可能会遇到`1+N`加载问题。`1+N`加载意味着你通过单个数据库操作（`1`）从数据库中查询一系列实体，然后执行一个循环，访问这些实体的导航属性（或集合）。在这种情况下，它会为每个循环（`N` = 第一次数据库操作中查询到的实体的数量）懒加载相关的属性。因此，你进行了`1+N`次数据库调用，这会极大地降低你的应用程序性能。在这种情况下，你应该预先加载相关的实体，以便总共进行一次数据库调用。

+   它使得对你的代码进行预测和优化变得困难，因为你可能不容易看到相关数据何时从数据库中加载。

我建议采取更受控的方法，并在可能的情况下使用预加载。

### 预加载

预加载是在首先查询主实体时加载相关数据的一种方式。

假设你已经创建了一个自定义仓库方法，在从数据库获取`Form`对象的同时加载相关的问题，如下所示：

```cs
public async Task<Form> GetWithQuestions(Guid formId)
{
    var dbContext = await GetDbContextAsync();
    return await dbContext.Forms
        .Include(f => f.Questions)
        .SingleAsync(f => f.Id == formId);
}
```

如果你创建了这样的自定义存储库方法，你可以使用完整的 EF Core API。然而，如果你正在使用 ABP 的存储库，并且不希望在应用程序层依赖于 EF Core，你不能使用 EF Core 的 `Include` 扩展方法（用于预加载相关数据）。在这种情况下，你有两种选择，下一节将讨论。

#### `IRepository.WithDetailsAsync`

`IRepository` 的 `WithDetailsAsync` 方法通过包含给定的属性或集合，返回一个 `IQueryable` 实例，如下所示：

```cs
public async Task EagerLoadDemoAsync(Guid formId)
{
    var queryable = await _formRepository
        .WithDetailsAsync(f => f.Questions);
    var query = queryable.Where(f => f.Id == formId);
    var form = await                                                 _asyncExecuter.FirstOrDefaultAsync(query);
    foreach (var question in form.Questions)
    {
        //...
    }
}
```

`WithDetailsAsync(f => f.Questions)` 返回包含问题的 `IQueryable<Form>`，因此我们可以安全地遍历 `form.Questions` 集合。`IAsyncQueryableExecuter` 在本章的 *通用存储库* 部分中已经解释过。如果需要包含多个属性，`WithDetailsAsync` 方法可以获取多个表达式。如果需要嵌套包含（EF Core 中的 `ThenInclude` 扩展方法），则不能使用 `WithDetailsAsync`。在这种情况下，创建一个自定义存储库方法。

#### 聚合模式

聚合模式将在*第十章*，*领域层*中进行深入探讨。然而，为了提供一些简要信息，聚合被认为是一个单一单元；它作为一个单元读取和保存，包括所有子集合。这意味着你始终在加载表单时加载相关的问题。

ABP 很好地支持聚合模式，并允许你在全局点为实体配置预加载。我们可以在我们的模块类（在解决方案中的 `EntityFrameworkCore` 项目）的 `ConfigureServices` 方法中编写以下配置：

```cs
Configure<AbpEntityOptions>(options =>
{
    options.Entity<Form>(orderOptions =>
    {
        orderOptions.DefaultWithDetailsFunc = query =>                   query
            .Include(f => f.Questions)
            .Include(f => f.Owners);
    });
});
```

建议包含所有子集合。一旦你配置了 `DefaultWithDetailsFunc` 方法，如下所示，那么以下情况将会发生：

+   返回单个实体（例如 `GetAsync`）的存储库方法默认会预加载相关实体，除非你通过在方法调用中指定 `includeDetails` 参数为 `false` 来显式禁用该行为。

+   返回多个实体（例如 `GetListAsync`）的存储库方法将允许预加载相关实体，但默认情况下不会进行预加载。

这里有一些示例。

获取包含子集合的单个表单如下：

```cs
var form = await _formRepository.GetAsync(formId);
```

获取不带子集合的单个表单如下：

```cs
var form = await _formRepository.GetAsync(formId, includeDetails: false);
```

获取不带子集合的表单列表如下：

```cs
var forms = await _formRepository.GetListAsync(f => f.Name.StartsWith("A"));
```

获取包含子集合的表单列表如下：

```cs
var forms = await _formRepository.GetListAsync(f => f.Name.StartsWith("A"), includeDetails: true);
```

聚合模式在大多数情况下简化了你的应用程序代码，同时你仍然可以针对需要性能优化的情况进行微调。请注意，如果你真正实现了聚合模式，则不会使用导航属性（到其他聚合）。我们将在*第十章*，*领域层*中再次讨论这个话题。

我们已经涵盖了使用 ABP 框架的 EF Core 的基本知识。下一节将解释 MongoDB 集成，这是 ABP 框架的另一个内置数据库提供程序。

# MongoDB 集成

MongoDB 是一种流行的非关系型 **文档数据库**，它以类似于 JSON 的文档形式存储数据，而不是传统的基于行/列的表。

ABP CLI 提供了一个选项，可以创建使用 MongoDB 的新应用程序，如下所示：

```cs
abp new FormsApp -d mongodb
```

如果你想要检查和更改数据库连接字符串，你可以查看应用程序的 `appsettings.json` 文件。

MongoDB 客户端包

ABP 使用官方的 `MongoDB.Driver` NuGet 包来实现 MongoDB 集成。

在接下来的章节中，你将学习如何使用 ABP 的 `AbpMongoDbContext` 类来定义 `DbContext` 对象，执行对象映射配置，将 `DbContext` 对象注册到 DI 系统中，并在需要扩展实体通用存储库时实现自定义存储库。

我们通过定义一个 `DbContext` 类来开始 MongoDB 集成。

## 定义 DbContexts

MongoDB 驱动程序包没有像 EF Core 那样的 `DbContext` 概念。然而，ABP 引入了 `AbpMongoDbContext` 类，以提供一个定义和配置 MongoDB 集成的标准方式。我们需要定义一个从 `AbpMongoDbContext` 基类派生的类，如下所示：

```cs
public class FormsAppDbContext : AbpMongoDbContext
{
    [MongoCollection("Forms")]
    public IMongoCollection<Form> Forms =>                           Collection<Form();
}
```

`MongoCollection` 属性在数据库侧设置集合名称。它是可选的，如果你没有指定它，则使用驱动程序的默认值。在 `FormsAppDbContext` 类上定义一个集合属性是使用默认泛型存储库所必需的。

## 配置对象映射

虽然 MongoDB C# 驱动程序不是一个 ORM，但它仍然将你的实体映射到数据库中的集合，你可能想要自定义映射配置。在这种情况下，像这样覆盖你的 `DbContext` 类中的 `CreateModel` 方法：

```cs
protected override void CreateModel(IMongoModelBuilder builder)
{
    builder.Entity<Form>(b =>
    {
        b.BsonMap.UnmapProperty(f => f.Description);
    });
}
```

在本例中，我已经配置了 MongoDB，使其在保存和检索数据时忽略 `Form` 实体的 `Description` 属性。请参阅 `MongoDB.Driver` NuGet 包的文档，了解所有配置选项。

## 将 DbContext 注册到 DI

一旦创建并配置了你的 `DbContext` 类，它就会在模块类的 `ConfigureServices` 方法中注册到 DI 系统中（通常在解决方案的 MongoDB 集成项目中）。以下代码片段说明了这一点：

```cs
public override void ConfigureServices(
    ServiceConfigurationContext context)
{
    context.Services.AddMongoDbContext<FormsAppDbContext>(
        options =>
            {
                options.AddDefaultRepositories();
            });
}
```

`AddDefaultRepositories()` 用于为与该 `DbContext` 相关的实体启用默认泛型存储库。然后，你可以将 `IRepository<Form>` 注入到你的类中，并开始使用你的 MongoDB 数据库。

`AddDefaultRepositories`方法仅对聚合根实体（从`AggregateRoot`类派生的实体类）启用默认仓储。将`includeAllEntities`设置为`true`以启用所有实体类型的默认仓储。然而，在处理 MongoDB 时，强烈建议应用聚合模式。聚合模式将在*第十章*中深入探讨，*领域层 – DDD*。

在大多数情况下，默认的泛型仓储就足够了，但你可能需要访问 MongoDB API 或将查询抽象到自定义仓储方法中。

## 实现自定义仓储

我们在本章“与仓储一起工作”部分的“自定义仓储”部分创建了一个`IFormRepository`接口。我们可以使用 MongoDB 实现这个仓储接口。

你可以在你的解决方案的 MongoDB 集成项目中实现仓储，如下所示：

```cs
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using MongoDB.Driver;
using MongoDB.Driver.Linq;
using Volo.Abp.Domain.Repositories.MongoDB;
using Volo.Abp.MongoDB;
namespace FormsApp
{
    public class FormRepository : 
        MongoDbRepository<FormsAppDbContext, Form, Guid>, 
        IFormRepository
    {
        public FormRepository(
            IMongoDbContextProvider<FormsAppDbContext>                       dbContextProvider)
            : base(dbContextProvider)
        { }
        // TODO: implement the GetListAsync method
    }
}
```

`FormRepository`类是从 ABP 的`MongoDbRepository`类派生的。这样，我们就继承了所有标准仓储方法。`MongoDbRepository`类获取三个泛型参数：`DbContext`类型、实体类型和实体类的 PK 类型。

`FormRepository`类应实现由`IFormRepository`接口定义的`GetListAsync`方法，如下所示：

```cs
public async Task<List<Form>> GetListAsync(
    string name, bool includeDrafts = false)
{
    var queryable = await GetMongoQueryableAsync();
    var query = queryable.Where(f =>                                 f.Name.Contains(name));
    if (!includeDrafts)
    {
        query = queryable.Where(f => !f.IsDraft);
    }
    return await query.ToListAsync();
}
```

在这个例子中，我使用了 MongoDB 驱动程序的 LINQ API，但你可以通过获取`IMongoCollection`对象来使用其他 API，如下面的代码片段所示：

```cs
IMongoCollection<Form> formsCollection = await GetCollectionAsync();
```

现在，你可以将`IFormRepository`注入到你的服务中，而不是泛型的`IRepository<Form, Guid>`仓储，并使用所有标准和自定义仓储方法。

即使为`Form`实体实现了自定义仓储类，仍然可以为此实体注入和使用默认的泛型仓储（例如`IRepository<Form, Guid>`）。如果你实现了自定义仓储，建议在`DbContext`注册代码中使用`AddRepository`方法，如下面的代码片段所示：

```cs
context.Services.AddMongoDbContext<FormsAppDbContext>(options =>
{
    options.AddDefaultRepositories();
    options.AddRepository<Form, FormRepository>();
});
```

这样，泛型默认仓储将重定向到你的自定义仓储类。如果你在自定义仓储中重写了基类方法，它们也将使用你的重载而不是基类方法。

我们已经学习了如何使用 EF Core 和 MongoDB 作为数据库提供者。在下一节中，我们将理解 UoW 系统，使其能够连接这些数据库并应用事务。

# 理解 UoW 系统

UoW 是 ABP 用来启动、管理和释放数据库连接和事务的主要系统。UoW 系统是按照**环境上下文模式**设计的。这意味着当我们创建一个新的 UoW 时，它会创建一个范围上下文，该上下文由当前范围内执行的、共享相同上下文的全部数据库操作参与，并被视为一个单独的事务边界。在 UoW 中执行的所有操作要么（在成功时）提交，要么（在异常时）回滚。

虽然你可以手动创建 UoW 范围并控制事务属性，但大多数时候，它将无缝地按照你的期望工作。然而，如果你更改默认行为，它提供了一些选项。

UoW 和数据库操作

所有数据库操作必须在 UoW 范围内执行，因为 UoW 是管理 ABP 框架中数据库连接和事务的方式。否则，你会得到一个异常指示。

在接下来的章节中，你将了解 UoW 系统的工作原理以及如何通过配置选项来自定义它。我还会解释如何在传统系统不适合你的用例时手动控制 UoW 系统。

## 配置 UoW 选项

在默认设置下，在 ASP.NET Core 应用程序中，一个 HTTP 请求被视为 UoW 范围。ABP 在请求开始时启动 UoW，如果请求成功完成，则将更改保存到数据库。如果请求因异常失败，则回滚 UoW。

ABP 根据 HTTP 请求类型确定数据库事务的使用。HTTP `GET`请求不创建数据库事务。UoW 仍然工作，但在这种情况下不使用数据库事务。如果你没有其他配置，所有其他 HTTP 请求类型（`POST`、`PUT`、`DELETE`和其他）都将使用数据库事务。

HTTP GET 请求和事务

不在`GET`请求中更改数据库是一个最佳实践。如果你在`GET`请求中执行多个写操作，并且请求以某种方式失败，你的数据库状态可能会处于不一致的状态，因为 ABP 不会为`GET`请求创建数据库事务。在这种情况下，你可以使用`AbpUnitOfWorkDefaultOptions`为`GET`请求启用事务，或者像下一节中描述的那样手动控制 UoW。

如果你想更改 UoW 选项，请在你的模块（数据库集成项目中）的`ConfigureServices`方法中使用`AbpUnitOfWorkDefaultOptions`，如下所示：

```cs
public override void ConfigureServices(
    ServiceConfigurationContext context)
{
    Configure<AbpUnitOfWorkDefaultOptions>(options =>
    {
        options.TransactionBehavior =                                   UnitOfWorkTransactionBehavior.Enabled;
        options.Timeout = 300000; // 5 minutes
        options.IsolationLevel =                                      IsolationLevel.Serializable;
    });
}
```

`TransactionBehavior`可以取以下三个值：

+   `Auto` (默认): 自动确定是否使用数据库事务（非`GET` `HTTP`请求启用事务）

+   `Enabled`: 总是使用数据库事务，即使是`HTTP` `GET`请求

+   `Disabled`: 从不使用数据库事务

`Auto`行为是默认值，适用于大多数应用。`IsolationLevel`仅对关系型数据库有效。如果你没有指定，ABP 将使用底层提供者的默认值。最后，`Timeout`选项允许你将事务的默认超时值设置为毫秒。如果 UoW 操作在给定超时值内未完成，将抛出超时异常。

在本节中，我们学习了如何配置所有 UoW 的默认选项。如果你手动控制，也可以为单个 UoW 配置这些值。

## 手动控制 UoW

对于 Web 应用，你很少需要手动控制 UoW 系统。然而，对于后台工作者或非 Web 应用，你可能需要自己创建 UoW 作用域。你可能还需要控制 UoW 系统以创建内部事务作用域。

创建 UoW 作用域的一种方式是在你的方法上使用`[UnitOfWork]`属性，如下所示：

```cs
[UnitOfWork(isTransactional: true)]
public async Task DoItAsync()
{
    await _formRepository.InsertAsync(new Form() { ... });
    await _formRepository.InsertAsync(new Form() { ... });
}
```

UoW 系统使用环境上下文模式。如果已经存在一个周围的 UoW，你的`UnitOfWork`属性将被忽略，你的方法将参与周围的 UoW。否则，ABP 在进入`DoItAsync`方法之前启动一个新的事务性 UoW，如果没有抛出异常，则提交事务。如果该方法抛出异常，则回滚事务。

如果你想要精细控制 UoW 系统，你可以注入并使用`IUnitOfWorkManager`服务，如下面的代码块所示：

```cs
public async Task DoItAsync()
{
    using (var uow = _unitOfWorkManager.Begin(
        requiresNew: true,
        isTransactional: true,
        timeout: 15000))
    {
        await _formRepository.InsertAsync(new Form() { });
        await _formRepository.InsertAsync(new Form() { });
        await uow.CompleteAsync();
    }
}
```

在此示例中，我们以 15 秒作为`timeout`参数值的值启动一个新的事务性 UoW 作用域。使用此用法（`requiresNew: true`），即使存在周围的 UoW，ABP 也会始终启动一个新的 UoW。如果一切顺利，始终调用`uow.CompleteAsync()`方法。如果你想回滚当前事务，可以使用`uow.RollbackAsync()`方法。

如前所述，UoW 使用环境作用域。你可以使用`IUnitOfWorkManager.Current`属性在任何此作用域中访问当前 UoW。如果没有正在进行的 UoW，它可以是`null`。

以下代码片段使用`SaveChangesAsync`方法与`IUnitOfWorkManager.Current`属性：

```cs
await _unitOfWorkManager.Current.SaveChangesAsync();
```

我们已将所有挂起的更改保存到数据库中。然而，如果这是一个事务性 UoW，如果你回滚 UoW 或在该作用域中抛出任何异常，这些更改也将被回滚。

# 摘要

在本章中，我们学习了如何使用 ABP 框架与数据库交互。ABP 通过提供基类来标准化定义实体。它还帮助自动跟踪更改时间和更改实体的用户，当你从审计实体类派生时。

仓储系统提供了读取和写入实体的基本功能。您可以使用 LINQ 在仓储上进行高级查询。此外，您还可以创建自定义仓储类，直接与底层数据提供程序一起工作，将复杂查询隐藏在简单的仓储接口后面，调用存储过程等。

ABP 是数据库无关的，但它提供了与 EF Core 和 MongoDB 的集成包，开箱即用。ABP 应用程序启动模板包含这些提供程序之一，您可以选择您喜欢的。

EF Core 是.NET 平台上的事实上的 ORM，ABP 将其作为一等公民支持。应用程序启动模板经过精心调整，以配置您的映射并管理您的数据库模式迁移，同时支持模块化应用程序结构。

最后，UoW 系统为我们提供了一个无缝的方式来管理数据库连接和事务。它通过自动化这些重复性任务，使应用程序代码保持整洁。

数据访问是任何业务应用的核心需求，理解其细节至关重要。下一章将继续介绍每个应用所需的横切关注点，例如授权、验证和异常处理。
