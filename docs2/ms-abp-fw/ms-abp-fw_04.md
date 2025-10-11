# *第三章*：逐步应用程序开发

本章通过构建一个示例应用程序来介绍 ABP 框架的基本知识。示例应用程序用于在典型的 **CRUD** 页面上管理产品（请注意，CRUD 页面用于 **创建**、**读取**（查看）、**更新**和**删除**实体）。

本章中提供的示例比简单的 CRUD 页面更高级。它实现了应用开发中的许多方面，具有生产质量。到本章结束时，您将了解基础知识，并准备好使用 ABP 框架开始开发。

我将按照构建真实世界项目的顺序逐步进行。本章包括以下主题；每个主题代表此过程中的一个步骤：

+   创建解决方案

+   定义领域对象

+   **Entity Framework** (**EF**) Core 和数据库映射

+   列出产品数据

+   创建产品

+   编辑产品

+   删除产品

    用户界面（UI）和数据库偏好

    我更喜欢 **Razor Pages (MVC**) 作为 UI 框架和 **EF Core** 作为数据库提供者。我们将在单独的章节中介绍其他 UI 框架和数据库提供者。

# 技术要求

我们将构建一个应用程序，因此您需要安装 .NET 运行时、ABP CLI 以及一个 IDE/编辑器来构建 ASP.NET Core 项目。

请参阅 *第二章*，*使用 ABP 框架入门*，了解如何准备您的开发环境，以及创建和运行解决方案。

您可以从 GitHub 仓库 [`github.com/PacktPublishing/Mastering-ABP-Framework`](https://github.com/PacktPublishing/Mastering-ABP-Framework) 下载最终应用程序的源代码。

# 创建解决方案

第一步是为产品管理应用程序创建一个解决方案。如果您在 *第二章* 中创建了 *ProductManagement* 解决方案，*使用 ABP 框架入门*，您可以使用它。否则，在您的计算机中创建一个空文件夹，在此文件夹中打开一个命令行终端，并运行以下 **ABP CLI** 命令以创建一个新的 Web 应用程序：

```cs
abp new ProductManagement -t app
```

在您最喜欢的 IDE 中打开解决方案，创建数据库，并运行 Web 项目。如果您在运行解决方案时遇到问题，请参阅上一章。

现在我们有一个正在运行的解决方案。我们可以通过定义解决方案的领域对象来开始开发。

# 定义领域对象

在本节中，您将学习如何使用 ABP 框架定义实体。对于此应用程序，领域很简单。我们有 **Product** 和 **Category** 实体以及一个 **ProductStockState** 枚举，如图 *3.1* 所示：

![图 3.1 – 一个示例产品管理领域](img/Figure_3.1_B17287.jpg)

图 3.1 – 一个示例产品管理领域

实体在解决方案的 *领域层* 中定义，领域层在解决方案中分为两个项目：

+   **ProductManagement.Domain**用于定义你的实体、值对象、领域服务、存储库接口以及其他核心领域相关的类。

+   **ProductManagement.Domain.Shared**用于定义一些原始的共享类型。在此项目中定义的类型对所有其他层都是可用的。通常，我们在这里定义枚举和一些常量。

因此，我们可以从创建`Category`和`Product`实体以及`ProductStockState`枚举开始。

## Category

`Category`实体用于对产品进行分类。在`ProductManagement.Domain`项目内创建一个*Categories*文件夹，并在其中创建一个`Category`类：

```cs
using System;
using Volo.Abp.Domain.Entities.Auditing;
namespace ProductManagement.Categories
{
    public class Category : AuditedAggregateRoot<Guid>
    {
        public string Name { get; set; }
    }
}
```

`Category`是一个从`AuditedAggregateRoot<Guid>`派生的类。在这里，`Guid`是实体的主键（`Id`）类型。只要你的数据库管理系统支持，你可以使用任何类型的键（如`int`、`long`或`string`）。

`AggregateRoot`是一种特殊类型的实体，用于创建聚合的根实体类型。聚合是**领域驱动设计**（**DDD**）的一个概念，我们将在接下来的章节中详细讨论。现在，请考虑我们从这个类继承主要实体。

`AuditedAggregateRoot`类向`AggregateRoot`类添加了一些额外的属性：`CreationTime`作为`DateTime`，`CreatorId`作为`Guid`，`LastModificationTime`作为`DateTime`，以及`LastModifierId`作为`Guid`。

ABP 会自动设置这些属性。例如，当你将实体插入数据库时，`CreationTime`会被设置为当前时间，而`CreatorId`会自动设置为当前用户的`Id`属性。

审计日志系统和基础`Audited`类将在*第八章*中介绍，*使用 ABP 的功能和服务*。

关于丰富的领域模型

在本章中，我保持了实体的简单性，具有公共的获取器和设置器。如果你想创建丰富的领域模型并应用 DDD 原则和其他最佳实践，我们将在接下来的章节中讨论它们。

## ProductStockState

`ProductStockState`是一个简单的枚举，用于设置和跟踪库存中产品的可用性。

在`ProductManagement.Domain.Shared`项目内创建一个*Products*文件夹，并在其中创建一个`ProductStockState`枚举：

```cs
namespace ProductManagement.Products
{
    public enum ProductStockState : byte
    {
        PreOrder,
        InStock,
        NotAvailable,
        Stopped
    }
}
```

我们在`ProductManagement.Domain.Shared`项目中定义这个`enum`，因为我们将在**数据传输对象**（**DTOs**）和 UI 层中重用它。

## Product

`Product`类代表一个真实的产品。我故意添加了不同类型的属性来展示它们的用法。在`ProductManagement.Domain`项目内创建一个*Products*文件夹，并在其中创建一个`Product`类：

```cs
using System;
using Volo.Abp.Domain.Entities.Auditing;
using ProductManagement.Categories;
namespace ProductManagement.Products
{
    public class Product : FullAuditedAggregateRoot<Guid>
    {
        public Category Category { get; set; }
        public Guid CategoryId { get; set; }
        public string Name { get; set; }
        public float Price { get; set; }
        public bool IsFreeCargo { get; set; }
        public DateTime ReleaseDate { get; set; }
        public ProductStockState StockState { get; set; }
    }
}
```

这次，我继承了`FullAuditedAggregateRoot`，它除了`AuditedAggregateRoot`类用于`Category`类外，还添加了`IsDeleted`作为`bool`，`DeletionTime`作为`DateTime`，以及`DeleterId`作为`Guid`属性。

`FullAuditedAggregateRoot` 实现了 `ISoftDelete` 接口，这使得实体可以进行 **软删除**。这意味着它不会被从数据库中删除，而是仅仅 *标记为已删除*。ABP 自动处理所有软删除逻辑。您像平常一样删除实体，但实际上它并没有被删除。下次查询时，已删除的实体将自动过滤，除非您故意请求它们，否则您不会在查询结果中看到它们。我们将在 *第八章*，*使用 ABP 的功能和服务* 的 *使用数据过滤系统* 部分中回到这个功能。

关于导航属性

在这个例子中，`Product.Category` 是 `Category` 实体的导航属性。如果您使用 MongoDB 或想真正实现 DDD，您不应向其他聚合添加导航属性。然而，对于关系数据库，它工作得非常好，并为我们的代码提供了灵活性。我们将在 *第十章*，*DDD – 领域层* 中讨论替代方法。

解决方案中的新文件应类似于 *图 3.2*：

![图 3.2 – 将领域对象添加到解决方案中](img/Figure_3.2_B17287.jpg)

图 3.2 – 将领域对象添加到解决方案中

我们已经创建了领域对象。此外，我们还将创建一些 `const` 值，以便在应用程序的后续部分使用。

## 常量

我们需要为实体的属性定义常量值。然后，我们将在输入验证和数据库映射阶段使用它们。

首先，在 *ProductManagement.Domain.Shared* 项目的内部创建一个 *Categories* 文件夹，并在其中添加一个 `CategoryConsts` 类：

```cs
namespace ProductManagement.Categories
{
    public static class CategoryConsts
    {
        public const int MaxNameLength = 128;
    }
}
```

在这里，`MaxNameLength` 值将被用于实现 `Category` 实例的 `Name` 属性的约束。

然后，在 *ProductManagement.Domain.Shared* 项目的 *Products* 文件夹内创建一个 `ProductConsts` 类：

```cs
namespace ProductManagement.Products
{
    public static class ProductConsts
    {
        public const int MaxNameLength = 128;
    }
}
```

`MaxNameLength` 值将被用于实现 `Product` 实例的 `Name` 属性的约束。

*ProductManagement.Domain.Shared* 项目应类似于 *图 3.3*：

![图 3.3 – 添加常量类](img/Figure_3.3_B17287.jpg)

图 3.3 – 添加常量类

现在领域层已经完成，我们现在可以配置 EF Core 的数据库映射。

# EF Core 和数据库映射

在这个应用程序中，我们使用 **EF Core**。EF Core 是由 Microsoft 提供的一个 **对象关系映射** (**ORM**) 提供商。ORM 提供了抽象，使您感觉像是在应用程序代码中操作对象，而不是数据库表。我们将在 *第六章*，*使用数据访问基础设施* 中介绍 ABP 的 EF Core 集成。然而，现在让我们专注于如何实际使用它。

首先，我们将向`DbContext`类添加实体并定义实体与数据库表之间的映射。然后，我们将使用 EF Core 的**代码优先迁移**方法来构建创建数据库表的必要代码。在此之后，我们将查看 ABP 的**数据初始化**系统，以将一些初始数据插入数据库。最后，我们将应用迁移和种子数据到数据库中，为应用程序做准备。

首先，让我们从定义实体的`DbSet`属性开始。

## 将实体添加到`DbContext`类中

EF 的`DbContext`类是用于定义实体与数据库表之间映射的主要类。此外，它还用于访问数据库并为相关实体执行数据库操作。

在*ProductManagement.EntityFrameworkCore*项目中打开`ProductManagementDbContext`类，并在其中添加以下`DbSet`属性（您需要导入`Product`和`Category`对象的命名空间）：

```cs
public DbSet<Product> Products { get; set; }
public DbSet<Category> Categories { get; set; }
```

为实体添加`DbSet`属性将实体与`DbContext`类相关联。然后，我们可以使用该`DbContext`类来对实体执行数据库操作。EF Core 可以通过基于属性名称和类型的约定来执行大部分映射。如果您想自定义默认映射配置或执行其他配置，您有两个选项：**数据注解**（属性）和**流畅式 API**。

在数据注解方法中，您将`[Required]`和`[StringLength]`等属性添加到实体属性中。它非常实用且易于使用。它还使阅读实体源代码时更容易理解。

数据注解属性的一个问题是它们有限制（与流畅式 API 相比），并且当您需要使用 EF Core 的自定义属性，如`[Index]`和`[Owned]`时，会使您的领域层依赖于 EF Core 的 NuGet 包。如果您不介意这个问题，您可以使用数据注解属性，并在它们不足够的地方结合使用流畅式 API。

在本章中，我将优先选择流畅式 API 方法，这种方法可以使实体更加简洁，并将所有 ORM 逻辑放置在基础设施层中。

## 将实体映射到数据库表

`ProductManagementDbContext`类（在*ProductManagement.EntityFrameworkCore*项目中）包含一个`OnModelCreating`方法来配置实体到数据库表的映射。当您首次创建解决方案时，此方法看起来如下：

```cs
protected override void OnModelCreating(ModelBuilder builder)
{
    base.OnModelCreating(builder);
    builder.ConfigurePermissionManagement();
    builder.ConfigureSettingManagement();
    builder.ConfigureIdentity();
    ...configuration of the other modules
    /* Configure your own tables/entities here */
}
```

在前面的注释之后添加以下代码：

```cs
builder.Entity<Category>(b =>
{
      b.ToTable("Categories");
      b.Property(x => x.Name)
            .HasMaxLength(CategoryConsts.MaxNameLength)
            .IsRequired();
      b.HasIndex(x => x.Name);
});
builder.Entity<Product>(b =>
{
      b.ToTable("Products");
      b.Property(x => x.Name)
            .HasMaxLength(ProductConsts.MaxNameLength)
            .IsRequired();
      b.HasOne(x => x.Category)
           .WithMany()
           .HasForeignKey(x => x.CategoryId)
           .OnDelete(DeleteBehavior.Restrict)
           .IsRequired();
b.HasIndex(x => x.Name).IsUnique();
});
```

这段代码部分定义了`Category`和`Product`的映射配置。

关于命名空间

您可能需要为`Product`类、`Category`类以及代码中使用的任何其他类的命名空间添加`using`语句。如果您遇到麻烦，您始终可以参考本章*技术要求*部分中我分享的 GitHub 仓库中的源代码。

`Category`实体映射到*Categories*数据库表。我们使用之前定义的`CategoryConsts.MaxNameLength`来设置数据库中`Name`字段的最大长度。`Name`字段也是一个*必需*属性。最后，我们为`Name`属性定义了一个*唯一*数据库索引，因为它有助于通过`Name`字段搜索类别。

`Product`映射类似于`Category`映射。此外，它定义了`Category`实体和`Product`实体之间的关系；一个`Product`实体属于一个`Category`实体，而一个`Category`实体可以拥有多个相关的`Product`实体。

EF Core 流畅映射

您可以参考 EF Core 文档来了解关于 Fluent Mapping API 的所有细节和其他选项。

映射配置已完成。现在是时候创建一个数据库迁移来更新新添加实体的数据库架构了。

## Add-Migration 命令

当您创建一个新的实体或修改现有的实体时，您还应该在数据库中创建或修改相关的表。EF Core 的**Code First Migration**系统是保持数据库架构与应用程序代码一致的理想方式。通常，您会生成迁移并将它们应用到数据库中。迁移是数据库的增量架构更改。当您更新数据库时，所有自上次更新以来的迁移都会被应用，数据库就会与应用程序代码保持一致。

有两种方法可以生成新的迁移。

### 使用 Visual Studio

如果您正在使用 Visual Studio，请从**视图** | **其他窗口** | **包管理控制台**菜单打开**包管理控制台（PMC）**：

![Figure 3.4 – 包管理控制台

![img/Figure_3.4_B17287.jpg]

图 3.4 – 包管理控制台

将*ProductManagement.EntityFrameworkCore*项目作为**默认项目**类型选择。确保*ProductManagement.Web*项目被选为启动项目。您可以在*ProductManagement.Web*项目上右键单击，然后单击**设置为启动项目**操作。

现在，您可以在 PMC 中输入以下命令以添加一个新的迁移类：

```cs
Add-Migration "Added_Categories_And_Products"
```

此命令的输出应类似于*图 3.5*：

![Figure 3.5 – Add-Migration 命令的输出

![img/Figure_3.5_B17287.jpg]

图 3.5 – Add-Migration 命令的输出

如果您遇到诸如*在程序集...中没有找到 DbContext*之类的错误，请确保您已将**默认项目**类型设置为*ProductManagement.EntityFrameworkCore*项目。

如果一切顺利，新的迁移类应该被添加到*ProductManagement.EntityFrameworkCore*项目的*Migrations*文件夹中。

### 在命令行中

如果你没有使用 Visual Studio，你可以使用 EF Core 命令行工具。如果你还没有安装它，请在命令行终端中执行以下命令：

```cs
dotnet tool install --global dotnet-ef
```

现在，在`ProductManagement.EntityFrameworkCore`项目的根目录下打开一个命令行终端，并输入以下命令：

```cs
dotnet ef migrations add "Added_Categories_And_Products"
```

应在`ProductManagement.EntityFrameworkCore`项目的`Migrations`文件夹内添加一个新的迁移类。

在应用新创建的迁移到数据库之前，我想提及 ABP 框架的数据初始化功能。

## 初始化数据

数据初始化系统用于在迁移数据库时添加一些初始数据。例如，身份模块在数据库中创建了一个拥有所有权限的 admin 用户，以便登录应用程序。

在我们的场景中，数据初始化不是必需的，但我想要在数据库中添加一些示例类别和产品，以便更容易地进行开发和测试。

关于 EF Core 数据初始化

本节使用 ABP 的数据初始化系统，而 EF Core 有自己的数据初始化功能。ABP 数据初始化系统允许你在数据初始化代码中注入运行时服务并实现高级逻辑，适用于开发、测试和生产环境。然而，对于更简单的开发测试场景，你可以使用 EF Core 的数据初始化系统。请查阅官方文档[`docs.microsoft.com/en-us/ef/core/modeling/data-seeding`](https://docs.microsoft.com/en-us/ef/core/modeling/data-seeding)。

在`ProductManagement.Domain`项目的`Data`文件夹中创建一个`ProductManagementDataSeedContributor`类：

```cs
using ProductManagement.Categories;
using ProductManagement.Products;
using System;
using System.Threading.Tasks;
using Volo.Abp.Data;
using Volo.Abp.DependencyInjection;
using Volo.Abp.Domain.Repositories;
namespace ProductManagement.Data
{
    public class ProductManagementDataSeedContributor :
           IDataSeedContributor, ITransientDependency
    {
        private readonly IRepository<Category,                           Guid>_categoryRepository;
        private readonly IRepository<Product,                           Guid>_productRepository;
        public ProductManagementDataSeedContributor(
            IRepository<Category, Guid> categoryRepository,
            IRepository<Product, Guid> productRepository)
        {
            _categoryRepository = categoryRepository;
            _productRepository = productRepository;
        }
        public async Task SeedAsync(DataSeedContext                     context)
        {
            /***** TODO: Seed initial data here *****/
        }
    }
}
```

这个类实现了`IDataSeedContributor`接口。当你想要初始化数据库时，ABP 会自动发现并调用它的`SeedAsync`方法。你可以在类中实现构造函数注入并使用任何服务（例如，本例中的存储库）。

然后，在`SeedAsync`方法内部编写以下代码：

```cs
if (await _categoryRepository.CountAsync() > 0)
{
    return;
}
var monitors = new Category { Name = "Monitors" };
var printers = new Category { Name = "Printers" };
await _categoryRepository
    .InsertManyAsync(new[] { monitors, printers });
var monitor1 = new Product
{
    Category = monitors,
    Name = "XP VH240a 23.8-Inch Full HD 1080p IPS LED               Monitor",
    Price = 163,
    ReleaseDate = new DateTime(2019, 05, 24),
    StockState = ProductStockState.InStock
};
var monitor2 = new Product
{
    Category = monitors,
    Name = "Clips 328E1CA 32-Inch Curved Monitor, 4K UHD",
    Price = 349,
    IsFreeCargo = true,
    ReleaseDate = new DateTime(2022, 02, 01),
    StockState = ProductStockState.PreOrder
};
var printer1 = new Product
{
    Category = monitors,
    Name = "Acme Monochrome Laser Printer, Compact All-In           One",
    Price = 199,
    ReleaseDate = new DateTime(2020, 11, 16),
    StockState = ProductStockState.NotAvailable
};
await _productRepository
    .InsertManyAsync(new[] { monitor1, monitor2, printer1 });
```

我们已经创建了两个类别，并添加了三个产品到数据库中。这个类在每次运行`DbMigrator`应用程序时都会执行（请参阅以下章节）。此外，我们检查了`if (await _categoryRepository.CountAsync() > 0)`以防止每次运行时插入相同的数据。

现在我们已经准备好迁移数据库，这将更新数据库架构并初始化初始数据。

## 迁移数据库

ABP 应用程序启动模板包括一个非常有用的 *DbMigrator* 控制台应用程序，它在开发和生产环境中都很有用。当您运行它时，所有挂起的迁移都会在数据库中应用，并且会执行数据生成器类。它支持多租户、多数据库场景，如果您使用标准的 `Update-Database` 命令，这是不可能的。此应用程序可以在生产环境中部署和执行，通常作为您 **持续部署** (**CD**) 管道的一部分。将迁移与主应用程序分离是一种很好的方法，因为在这种情况下主应用程序不需要更改数据库模式的权限。此外，如果您在主应用程序中应用迁移并运行多个应用程序实例，您还可以消除任何并发问题。

运行 *ProductManagement.DbMigrator* 应用程序以迁移数据库（即将其设置为启动项目，然后按 *Ctrl* + *F5*）。一旦应用程序退出，您就可以检查数据库以查看 *Categories* 和 *Products* 表已插入初始数据（如果您使用的是 Visual Studio，您可以使用 **SQL Server Object Explorer** 连接到 **LocalDB** 并探索数据库）。

EF Core 配置已完成，数据库已准备好开发。我们将继续在 UI 上显示产品数据。

# 列出产品数据

我更喜欢按功能特性开发应用程序功能。本节将解释如何在 UI 上以数据表的形式显示产品列表。

我们将首先定义一个 `ProductDto`，用于 `Product` 实体。然后，我们将创建一个应用程序服务方法，该方法将产品列表返回到表示层。此外，我们还将学习如何自动将 `Product` 实体映射到 `ProductDto`。

在创建 UI 之前，我将向您展示如何为应用程序服务编写 **自动化测试**。这样，我们就可以在开始 UI 开发之前确保应用程序服务正常工作。

在整个开发过程中，我们将探讨 ABP 框架的一些好处，例如自动 API 控制器和动态 JavaScript 代理系统。

最后，我们将创建一个新页面，在其中添加一个数据表，从服务器获取产品列表，并在 UI 上显示。

在下一节中，我们将首先创建一个 `ProductDto` 类。

## ProductDto 类

DTOs 用于在应用程序和表示层之间传输数据。最佳实践是将 DTOs 返回到表示层（UI），而不是实体。DTOs 允许您以受控的方式公开数据，并将实体从表示层抽象出来。直接将实体公开给表示层也可能导致序列化和安全问题。我们将在 *第十一章* 中讨论使用 DTOs 的好处，*领域驱动设计 – 应用层*。

DTOs 在*Application.Contracts*项目中定义，以便在 UI 层中使用。因此，我们首先在*ProductManagement.Application.Contracts*项目的*Products*文件夹中创建一个`ProductDto`类：

```cs
using System;
using Volo.Abp.Application.Dtos;
namespace ProductManagement.Products
{
    public class ProductDto : AuditedEntityDto<Guid>
    {
        public Guid CategoryId { get; set; }
        public string CategoryName { get; set; }
        public string Name { get; set; }
        public float Price { get; set; }
        public bool IsFreeCargo { get; set; }
        public DateTime ReleaseDate { get; set; }
        public ProductStockState StockState { get; set; }
    }
}
```

`ProductDto`类与`Product`实体类似，但有以下区别：

+   它从`AuditedEntityDto<Guid>`派生，该类定义了`Id`、`CreationTime`、`CreatorId`、`LastModificationTime`和`LastModifierId`属性（我们不需要删除审计属性，如`DeletionTime`，因为已删除的实体不会从数据库中读取）。

+   我们没有在`Category`实体中添加导航属性，而是使用了一个`string`类型的`CategoryName`属性，这对于在 UI 上显示是足够的。

我们将使用`ProductDto`类从`IProductAppService`接口返回产品列表。

## IProductAppService

**应用服务**实现了应用程序的使用案例。用户界面使用它们来执行用户交互的业务逻辑。通常，应用程序服务方法获取并返回 DTOs。

应用服务与 API 控制器比较

你可以在 ASP.NET Core MVC 应用程序中将应用服务与 API 控制器进行比较。虽然它们在某些用例中具有相似性，但应用服务是更适合 DDD 的纯类。它们不依赖于特定的 UI 技术。此外，ABP 可以自动将你的应用服务作为 HTTP API 公开，正如我们在本章的*自动 API 控制器和 Swagger UI*部分中所将发现的。

我们在解决方案的*Application.Contracts*项目中定义应用服务的接口。在*ProductManagement.Application.Contracts*项目的*Products*文件夹中创建一个`IProductAppService`接口：

```cs
using System.Threading.Tasks;
using Volo.Abp.Application.Dtos;
using Volo.Abp.Application.Services;
namespace ProductManagement.Products
{
    public interface IProductAppService :                           IApplicationService
    {
        Task<PagedResultDto<ProductDto>>
            GetListAsync(PagedAndSortedResultRequestDto                     input);
    }
}
```

你可以在前面的代码块中看到一些预定义的 ABP 类型：

+   `IProductAppService`是从`IApplicationService`接口派生的。通过这种方式，ABP 可以识别应用服务。

+   `GetListAsync`方法获取`PagedAndSortedResultRequestDto`，这是 ABP 框架的一个标准 DTO 类，它定义了`MaxResultCount`（整型）、`SkipCount`（整型）和`Sorting`（字符串）属性。

+   `GetListAsync`方法返回`PagedResultDto<ProductDto>`，它包含一个`TotalCount`（长整型）属性和一个`Items`集合，其中包含`ProductDto`对象。这是使用 ABP 框架返回分页结果的一种便捷方式。

你可以使用自己的 DTOs 而不是这些预定义的 DTO 类型。然而，当你在标准化某些常见模式并希望到处使用相同的命名时，它们非常有用。

异步方法

将所有应用服务方法定义为异步是一种最佳实践。如果你定义了同步的应用服务方法，在某些情况下，某些 ABP 功能（如工作单元）可能无法按预期工作。

现在，我们可以实现`IProductAppService`接口以执行用例。

## ProductAppService

在 *ProductManagement.Application* 项目的 *Products* 文件夹中创建一个 `ProductAppService` 类：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Linq.Dynamic.Core;
using System.Threading.Tasks;
using Volo.Abp.Application.Dtos;
using Volo.Abp.Domain.Repositories;
namespace ProductManagement.Products
{
    public class ProductAppService :
        ProductManagementAppService, IProductAppService
    {
        private readonly IRepository<Product, Guid>                     _productRepository;
        public ProductAppService(
            IRepository<Product, Guid> productRepository)
        {
            _productRepository = productRepository;
        }
        public async Task<PagedResultDto<ProductDto>>                   GetListAsync(
            PagedAndSortedResultRequestDto input)
        {
            /* TODO: Implementation */
        }
    }
}
```

`ProductAppService` 类继承自 `ProductManagementAppService`，该类在启动模板中定义，可以用作应用程序服务的基类。它实现了之前定义的 `IProductAppService` 接口。它注入了 `IRepository<Product, Guid>` 服务。这被称为 **默认仓储**。仓储是一个类似集合的接口，允许你在数据库上执行操作。ABP 自动为所有聚合根实体提供默认仓储实现。

我们可以实现 `GetListAsync` 方法，如下面的代码块所示：

```cs
public async Task<PagedResultDto<ProductDto>> GetListAsync(
    PagedAndSortedResultRequestDto input)
{
    var queryable = await _productRepository
        .WithDetailsAsync(x => x.Category);
    queryable = queryable
        .Skip(input.SkipCount)
        .Take(input.MaxResultCount)
        .OrderBy(input.Sorting ?? nameof(Product.Name));
    var products = await                                             AsyncExecuter.ToListAsync(queryable);
    var count = await _productRepository.GetCountAsync();
    return new PagedResultDto<ProductDto>(
        count,
        ObjectMapper.Map<List<Product>, List<ProductDto>>                (products)
    );
}
```

在这里，`_productRepository.WithDetailsAsync` 通过包含类别（`WithDetailsAsync` 方法类似于 EF Core 的 `Include` 扩展方法，它将相关数据加载到查询中）返回一个 `IQueryable<Product>` 对象。我们可以在查询对象上使用标准的 `Skip`、`Take` 和 `OrderBy`。

`AsyncExecuter` 服务（在基类中预先注入）用于执行 `IQueryable` 对象以异步执行数据库查询。这使得可以在应用层不依赖于 EF Core 包的情况下使用异步 LINQ 扩展方法。

最后，我们使用 `ObjectMapper` 服务（在基类中预先注入）将 `Product`（实体）对象列表映射到 `ProductDto`（DTO）对象列表。在下一节中，我们将解释对象映射是如何配置的。

仓储和异步查询执行

我们将在 *第六章*，*与数据访问基础设施一起工作* 中更详细地探讨 `IRepository` 和 `AsyncExecuter`。

## 对象到对象映射

`ObjectMapper`（`IObjectMapper` 服务）自动化类型转换，并默认使用 **AutoMapper** 库。它要求你在使用之前定义映射。启动模板包含一个配置类，你可以在其中创建映射。

在 *ProductManagement.Application* 项目的 `ProductManagementApplicationAutoMapperProfile` 类中打开，并将其更改为以下内容：

```cs
using AutoMapper;
using ProductManagement.Products;
namespace ProductManagement
{
    public class ProductManagementApplicationAutoMapperProfile
        : Profile
    {
        public ProductManagementApplicationAutoMapperProfile()
        {
            CreateMap<Product, ProductDto>();
        }
    }
}
```

在这里，`CreateMap` 定义了映射。然后，你可以在需要的地方自动将 `Product` 对象转换为 `ProductDto` 对象。

AutoMapper 的一个有趣特性是 `Product` 类有一个 `Category` 属性，而 `Category` 类有一个 `Name` 属性。所以，如果你想访问产品的类别名称，你应该使用 `Product.Category.Name` 表达式。然而，`ProductDto` 有一个直接的 `CategoryName` 属性，可以使用 `ProductDto.CategoryName` 表达式访问。AutoMapper 自动通过将 `Category.Name` 展平到 `CategoryName` 来映射这些表达式。

应用层已完成。在开始 UI 之前，我想向你展示如何为应用层编写自动化测试。

## 测试 ProductAppService 类

启动模板附带使用 **xUnit**、**Shouldly** 和 **NSubstitute** 库正确配置的测试基础设施。它使用 *SQLite 内存数据库* 来模拟数据库。为每个测试创建一个单独的数据库。测试结束时，数据库将被初始化和销毁。这样，测试不会相互影响，并且你的真实数据库保持不变。

*第十七章*，*构建自动化测试* 将探讨测试的所有细节。然而，在这里，我想向你展示如何轻松编写 `ProductAppService` 类的 `GetListAsync` 方法的自动化测试代码。在 UI 上使用之前编写应用程序服务的测试代码是很好的。

在 *ProductManagement.Application.Tests* 项目中创建一个 *Products* 文件夹，并在其中创建一个 `ProductAppService_Tests` 类：

```cs
using Shouldly;
using System.Threading.Tasks;
using Volo.Abp.Application.Dtos;
using Xunit;
namespace ProductManagement.Products
{
    public class ProductAppService_Tests
        : ProductManagementApplicationTestBase
    {
        private readonly IProductAppService                             _productAppService;
        public ProductAppService_Tests()
        {
            _productAppService =
                GetRequiredService<IProductAppService>();
        }
        /* TODO: Test methods */
    }
}
```

此类继承自 `ProductManagementApplicationTestBase` 类（包含在你的解决方案中），该类集成了 ABP 框架和其他基础设施库，使我们能够编写测试。由于测试中不可能使用构造函数注入，我们使用 `GetRequiredService` 方法在测试代码中解析依赖项。

现在，我们可以编写第一个测试方法。在 `ProductAppService_Tests` 类中添加以下方法：

```cs
[Fact]
public async Task Should_Get_Product_List()
{
    //Act
    var output = await _productAppService.GetListAsync(
        new PagedAndSortedResultRequestDto()
    );
    //Assert
    output.TotalCount.ShouldBe(3);
    output.Items.ShouldContain(
        x => x.Name.Contains("Acme Monochrome Laser                     Printer")
    );
}
```

此方法调用 `GetListAsync` 方法并检查结果是否正确。如果你打开 **测试资源管理器** 窗口（在 Visual Studio 的 **视图** | **测试资源管理器** 菜单下），你可以看到我们添加的测试方法。**测试资源管理器** 用于显示和运行解决方案中的测试：

![图 3.6 – 测试资源管理器窗口](img/Figure_3.6_B17287.jpg)

图 3.6 – 测试资源管理器窗口

运行测试以检查它是否按预期工作。如果 `GetListAsync` 方法工作正常，你将在测试方法名称的左侧看到绿色图标，如图 *图 3.6* 所示。单元测试和集成测试将在 *第十七章*，*构建自动化测试* 中介绍。

## 自动 API 控制器和 Swagger UI

**Swagger** 是一个流行的工具，用于探索和测试 HTTP API。它随启动模板预安装。

运行 *ProductManagement.Web* 项目以启动 Web 应用程序（如果尚未设置，请将其设置为启动项目，然后按 *Ctrl* + *F5*）。一旦应用程序启动，输入如图 *图 3.7* 所示的 `/swagger` URL：

![图 3.7 – Swagger UI](img/Figure_3.7_B17287.jpg)

图 3.7 – Swagger UI

你将看到来自应用程序中安装的模块的大量 API 端点。如果你向下滚动，你将看到一个 **产品** 端点。你可以测试它以获取产品列表：

![图 3.8 – 产品端点](img/Figure_3.8_B17287.jpg)

图 3.8 – 产品端点

我们还没有创建 *ProductController* 端点。那么这个端点是如何在这里可用的呢？这被称为 ABP 框架的 **Auto API 控制器** 功能。它根据命名约定和配置自动将您的应用程序服务公开为 HTTP API。通常，我们不手动编写控制器。

Auto API 控制器功能将在 *第十四章*，*构建 HTTP API 和实时服务*中详细介绍。

因此，我们有 HTTP API 来获取产品列表。下一步是从客户端代码中消费这个 API。

## 动态 JavaScript 代理

通常，您从 JavaScript 代码中调用 HTTP API 端点。ABP 动态为所有 HTTP API 创建客户端代理。然后，您可以使用这些动态 JavaScript 函数从客户端应用程序中消费您的 API。

再次运行 *ProductManagement.Web* 项目，并在您位于应用程序的着陆页时打开浏览器的 **开发者控制台**。开发者控制台在任何现代浏览器中都是可用的，通常使用 *F12* 快捷键（在 Windows 上）打开。它用于通过开发者探索、跟踪和调试应用程序。

打开 **控制台** 选项卡，并输入以下 JavaScript 代码：

```cs
productManagement.products.product.getList({}).then(function(result) {
    console.log(result);
});
```

一旦您执行此代码，就会向服务器发出请求，并将返回的结果记录在 **控制台** 选项卡中，如图 *3.9* 所示：

![图 3.9 – 使用动态 JavaScript 代理](img/Figure_3.9_B17287.jpg)

图 3.9 – 使用动态 JavaScript 代理

我们可以看到产品列表已记录在 **控制台** 选项卡中。这意味着我们可以轻松地从 JavaScript 代码中消费服务器端 API，而无需处理底层细节。

如果您想知道那个 `getList` JavaScript 定义在哪里，您可以在应用程序中的 `/Abp/ServiceProxyScript` 端点查看 ABP 框架动态创建的 JavaScript 代理函数。

在下一节中，我们将创建一个 **Razor 页面**来在 UI 上显示产品表。

## 创建产品页面

Razor Pages 是在 ASP.NET Core MVC 框架中创建 UI 的推荐方式。

首先，在 *ProductManagement.Web* 项目的 *Pages* 文件夹下创建一个 *Products* 文件夹。然后，通过右键单击 *Products* 文件夹并选择 `Index.cshtml` 来添加一个新的空 razor 页面。*图 3.10* 显示了我们添加的页面位置：

![图 3.10 – 创建 Razor 页面](img/Figure_3.10_B17287.jpg)

图 3.10 – 创建 Razor 页面

编辑 `Index.cshtml` 内容，如下面的代码块所示：

```cs
@page
@using ProductManagement.Web.Pages.Products
@model IndexModel
<h1>Products Page</h1>
```

在这里，我刚刚放置了一个`h1`元素作为页面标题。当我们创建一个页面时，通常，我们希望向主菜单添加一个项目以打开此页面。

## 添加新的菜单项

ABP 提供了一个动态和模块化的菜单系统。每个模块都可以向主菜单添加项目。

打开 *ProductManagement.Web* 项目的 *Menus* 文件夹中的 `ProductManagementMenuContributor` 类，并在 `ConfigureMainMenuAsync` 方法的末尾添加以下代码：

```cs
context.Menu.AddItem(
    new ApplicationMenuItem(
        "ProductManagement",
        l["Menu:ProductManagement"],
        icon: "fas fa-shopping-cart"
            ).AddItem(
        new ApplicationMenuItem(
            "ProductManagement.Products",
            l["Menu:Products"],
            url: "/Products"
        )
    )
);
```

此代码添加了一个包含 *产品* 菜单项的 *产品管理* 主菜单项。它使用我们应定义的本地化键（使用 `l["…"]` 语法）。打开 *ProductManagement.Domain.Shared* 项目的 *Localization/ProductManagement* 文件夹中的 `en.json` 文件，并在 `texts` 部分的末尾添加以下条目：

```cs
"Menu:ProductManagement": "Product Management",
"Menu:Products": "Products"
```

本地化键是任意的，这意味着你可以使用任何字符串值作为本地化键。我更喜欢使用 `Menu:` 前缀作为菜单项的本地化键，例如本例中的 `Menu:Products`。我们将在 *第八章*，*使用 ABP 的功能和服务* 中回到本地化的话题。

现在，你可以重新运行应用程序，并使用新的 *产品管理* 菜单项打开 *产品* 页面，如图 3.11 所示：

![图 3.11 – 产品页面](img/Figure_3.11_B17287.jpg)

图 3.11 – 产品页面

因此，我们已经创建了一个页面，并且可以使用菜单元素打开该页面。我们现在准备好创建一个数据表来显示产品列表。

## 创建产品数据表

我们将创建一个数据表来显示带有分页和排序的产品列表。ABP 启动模板预安装并配置了 **Datatables.net** JavaScript 库。这是一个灵活且功能丰富的库，用于显示表格数据。

打开 *Pages/Products* 文件夹中的 `Index.cshtml` 页面，并将其内容更改为以下内容：

```cs
@page
@using ProductManagement.Web.Pages.Products
@using Microsoft.Extensions.Localization
@using ProductManagement.Localization
@model IndexModel
@inject IStringLocalizer<ProductManagementResource> L
@section scripts
{
    <abp-script src="img/Index.cshtml.js" />
}
<abp-card>
    <abp-card-header>
        <h2>@L["Menu:Products"]</h2>
    </abp-card-header>
    <abp-card-body>
        <abp-table id="ProductsTable" striped-rows="true" />
    </abp-card-body>
</abp-card>
```

在这里，`abp-script` 是一个 ABP 标签助手，用于向页面添加带有自动捆绑、压缩和版本支持功能的脚本文件。`abp-card` 是另一个标签助手，以类型安全和简单的方式渲染卡片组件（它渲染一个 Bootstrap 卡片）。

我们可以使用标准的 HTML 标签。然而，ABP 标签助手极大地简化了 MVC/Razor Page 应用程序中的 UI 创建。此外，它们通过 IntelliSense 和编译时类型检查来防止错误。我们将在 *第十二章*，*使用 MVC/Razor Pages* 中研究标签助手。

在 *Pages/Products* 文件夹下创建一个新的 JavaScript 文件，命名为 `Index.cshtml.js`（你可能更喜欢不同的命名风格，例如 `index.js`；这没关系，只要你在 `abp-script` 标签中使用相同的文件名即可），内容如下：

```cs
$(function () {
    var l = abp.localization.getResource('ProductManagement');
    var dataTable = $('#ProductsTable').DataTable(
        abp.libs.datatables.normalizeConfiguration({
            serverSide: true,
            paging: true,
            order: [[0, "asc"]],
            searching: false,
            scrollX: true,
            ajax: abp.libs.datatables.createAjax(
                productManagement.products.product.getList),
            columnDefs: [
                /* TODO: Column definitions */
            ]
        })
    );
});
```

ABP 允许你在 JavaScript 代码中重用本地化文本。这样，你可以在服务器端定义它们，并在两边使用。`abp.localization.getResource` 返回一个用于本地化值的函数。

ABP 简化了数据表库的配置，并提供了内置集成：

+   `abp.libs.datatables.normalizeConfiguration` 是 ABP 框架定义的辅助函数。它通过为缺失的选项提供常规默认值来简化数据表的配置。

+   `abp.libs.datatables.createAjax` 是另一个辅助函数，它将 ABP 的动态 JavaScript 客户端代理适配到数据表的参数格式。

+   `productManagement.products.product.getList` 是之前介绍过的动态 JavaScript 代理函数。

在 `columnDefs` 数组内部定义数据表的列：

```cs
{
    title: l('Name'),
    data: "name"
},
{
    title: l('CategoryName'),
    data: "categoryName",
    orderable: false
},
{
    title: l('Price'),
    data: "price"
},
{
    title: l('StockState'),
    data: "stockState",
    render: function (data) {
        return l('Enum:StockState:' + data);
    }
},
{
    title: l('CreationTime'),
    data: "creationTime",
    dataFormat: 'date'
}
```

通常，列定义有一个 `title` 字段（显示名称）和一个 `data` 字段。数据字段与 `ProductDto` 类中的属性名称匹配，格式为 **camelCase**（一种命名风格，其中每个单词的首字母大写，除了第一个单词；它通常用于 JavaScript 语言中）。

可以使用 `render` 选项来精细控制如何显示列数据。我们提供了一个函数来自定义库存状态列的渲染。

在这个页面上，我们使用了一些本地化键。我们应该在本地化资源中定义它们。打开 *ProductManagement.Domain.Shared* 项目的 *Localization/ProductManagement* 文件夹中的 `en.json` 文件，并在 `texts` 部分的末尾添加以下条目：

```cs
"Name": "Name",
"CategoryName": "Category name",
"Price": "Price",
"StockState": "Stock state",
"Enum:StockState:0": "Pre-order",
"Enum:StockState:1": "In stock",
"Enum:StockState:2": "Not available",
"Enum:StockState:3": "Stopped",
"CreationTime": "Creation time"
```

您可以再次运行 Web 应用程序以查看产品数据表的实际效果：

![Figure 3.12 – 产品数据表](img/Figure_3.12_B17287.jpg)

![Figure 3.12 – The Products data table](img/Figure_3.12_B17287.jpg)

图 3.12 – 产品数据表

我们已经创建了一个完全工作的页面，该页面列出了具有分页和排序支持的产品。在接下来的章节中，我们将添加创建、编辑和删除产品的功能。

# 创建产品

在本节中，我们将创建添加新产品的必要功能。一个产品应该有一个类别。因此，在添加新产品时，我们应该选择一个类别。我们将定义新的应用程序服务方法来获取类别和创建产品。在 UI 部分，我们将使用 ABP 的动态表单功能自动生成基于 C# 类的产品创建表单。

## 应用程序服务合约

让我们从向 `IProductAppService` 接口添加两个新方法开始：

```cs
Task CreateAsync(CreateUpdateProductDto input);
Task<ListResultDto<CategoryLookupDto>> GetCategoriesAsync();
```

我们将使用 `GetCategoriesAsync` 方法在创建产品时显示类别下拉列表。我们已经介绍了两个新的 DTO，我们应该定义它们。

`CreateUpdateProductDto` 用于创建和更新产品（我们将在 *编辑产品* 部分重用它）。在 *ProductManagement.Application.Contracts* 项目的 *Products* 文件夹中定义它：

```cs
using System;
using System.ComponentModel.DataAnnotations;
namespace ProductManagement.Products
{
    public class CreateUpdateProductDto
    {
        public Guid CategoryId { get; set; }
        [Required]
        [StringLength(ProductConsts.MaxNameLength)]
        public string Name { get; set; }
        public float Price { get; set; }
        public bool IsFreeCargo { get; set; }
        public DateTime ReleaseDate { get; set; }
        public ProductStockState StockState { get; set; }
    }
}
```

接下来，在 *ProductManagement.Application.Contracts* 项目的 *Categories* 文件夹中定义一个 `CategoryLookupDto` 类：

```cs
using System;
namespace ProductManagement.Categories
{
    public class CategoryLookupDto
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
    }
}
```

我们已经创建了合约，现在我们可以在应用程序层实现这些合约。

## 应用程序服务实现

在 `ProductAppService`（在 *ProductManagement.Application* 项目中）中实现 `CreateAsync` 和 `GetCategoriesAsync` 方法，如下面的代码块所示：

```cs
public async Task CreateAsync(CreateUpdateProductDto input)
{
    await _productRepository.InsertAsync(
        ObjectMapper.Map<CreateUpdateProductDto, Product>                (input)
    );
}
public async Task<ListResultDto<CategoryLookupDto>>
       GetCategoriesAsync()
{
    var categories = await _categoryRepository.GetListAsync();
    return new ListResultDto<CategoryLookupDto>(
        ObjectMapper
        .Map<List<Category>, List<CategoryLookupDto>>                    (categories)
    );
}
```

在这里，`_categoryRepository` 是一种 `IRepository<Category, Guid>` 服务类型。你就像之前对 `_productRepository` 所做的那样注入它。我认为方法实现相当简单，不需要额外的解释。

我们在两个地方使用了对象映射，现在我们必须定义映射配置。打开 `ProductManagement.Application` 项目中的 `ProductManagementApplicationAutoMapperProfile.cs` 文件，并添加以下代码：

```cs
CreateMap<CreateUpdateProductDto, Product>();
CreateMap<Category, CategoryLookupDto>();
```

此代码设置了对象映射的 AutoMapper 配置。

自动化测试

我不会在本章中展示更多的自动化测试；然而，我已经将它们添加到解决方案中。你可以检查 GitHub 仓库中的源代码。

现在，我们可以去 UI 层调用这些方法。

## UI

在 `ProductManagement.Web` 项目的 `Pages/Products` 文件夹下创建一个新的 `CreateProductModal.cshtml` Razor 页面。打开 `CreateProductModal.cshtml.cs` 文件，并使用以下代码更改 `CreateProductModalModel` 类：

```cs
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Rendering;
using ProductManagement.Products;
namespace ProductManagement.Web.Pages.Products
{
    Public class CreateProductModalModel:
        ProductManagementPageModel
    {
        [BindProperty]
        public CreateEditProductViewModel Product { get;                 set; }
        public SelectListItem[] Categories { get; set; }
        private readonly IProductAppService                             _productAppService;
        public CreateProductModalModel(
            IProductAppService productAppService)
        {
            _productAppService = productAppService;
        }
        public async Task OnGetAsync()
        {
            // TODO
        }
        public async Task<IActionResult> OnPostAsync()
        {
            // TODO
        }
    }
}
```

在这里，`ProductManagementPageModel` 是在启动模板中定义的一个基类。你可以继承它来创建 `PageModel` 类。`Categories` 将用于在下拉列表中显示类别列表。`[BindProperty]` 是一个标准的 ASP.NET Core 属性，用于将请求数据绑定到 HTTP Post 请求上的 `Product` 属性。我们注入了 `IProductAppService` 接口来使用之前定义的方法。

我们已经使用了 `CreateEditProductViewModel`，因此需要定义它。在 `CreateProductModal.cshtml` 文件所在的同一文件夹中定义它：

```cs
using ProductManagement.Products;
using System;
using System.ComponentModel;
using System.ComponentModel.DataAnnotations;
using Volo.Abp.AspNetCore.Mvc.UI.Bootstrap.TagHelpers.Form;
namespace ProductManagement.Web.Pages.Products
{
    public class CreateEditProductViewModel
    {
        [SelectItems("Categories")]
        [DisplayName("Category")]
        public Guid CategoryId { get; set; }
        [Required]
        [StringLength(ProductConsts.MaxNameLength)]
        public string Name { get; set; }
        public float Price { get; set; }
        public bool IsFreeCargo { get; set; }
        [DataType(DataType.Date)]
        public DateTime ReleaseDate { get; set; }
        public ProductStockState StockState { get; set; }
    }
}
```

`SelectItems` 告诉我们 `CategoryId` 属性将从 `Categories` 列表中选择。我们将在这个编辑模态对话框中重用这个类。这就是为什么我将其命名为 `CreateEditProductViewModel`。

DTO 与 ViewModel 的比较

由于它与 DTO (`CreateUpdateProductDto`) 非常相似，因此定义视图模型 (`CreateEditProductViewModel`) 可能看起来是不必要的。然而，它只是多了几个属性。这些属性可以轻松地添加到 DTO 中，并且我们可以在视图端重用 DTO。这取决于你的设计决策，你可以这样做。然而，考虑到这些类具有不同的目的，并且随着时间的推移会向不同的方向发展，我认为将每个关注点分开是更好的实践。例如，`[SelectItems("Categories")]` 属性指的是 Razor 页面模型，在应用层没有意义。

现在，我们可以在 `CreateProductModalModel` 类中实现 `OnGetAsync` 方法：

```cs
public async Task OnGetAsync()
{
    Product = new CreateEditProductViewModel
    {
        ReleaseDate = Clock.Now,
        StockState = ProductStockState.PreOrder
    };

    var categoryLookup =
        await _productAppService.GetCategoriesAsync();
    Categories = categoryLookup.Items
        .Select(x => new SelectListItem(x.Name,                         x.Id.ToString()))
                .ToArray();
}
```

我们使用默认值创建 `Product` 类，然后使用产品应用服务填充 `Categories` 列表。`Clock` 是 ABP 框架提供的一个服务，用于获取当前时间，无需处理时区以及本地/UTC 时间。我们用它代替 `DateTime.Now`。这将在 *第八章*，*使用 ABP 的功能和服务* 中解释。

我们可以像下面这样实现 `OnPostAsync`：

```cs
public async Task<IActionResult> OnPostAsync()
{
    await _productAppService.CreateAsync(
        ObjectMapper
            .Map<CreateEditProductViewModel,CreateUpdateProductDto>                   (Product)
    );
    return NoContent();
}
```

由于我们将 `CreateEditProductViewModel` 映射到 `CreateProductDto`，我们需要定义映射配置。在 *ProductManagement.Web* 项目中打开 `ProductManagementWebAutoMapperProfile` 类，并使用以下代码块更改内容：

```cs
public class ProductManagementWebAutoMapperProfile : Profile
{
    public ProductManagementWebAutoMapperProfile()
    {
        CreateMap<CreateEditProductViewModel,                                   CreateUpdateProductDto>();
    }
}
```

此类定义了 AutoMapper 库的对象映射。

我们已经完成了产品创建 UI 的 C# 代码部分。

现在，我们可以开始构建 UI 标记和 JavaScript 代码。为此，打开 `CreateProductModal.cshtml` 文件，并按如下方式更改内容：

```cs
@page
@using Microsoft.AspNetCore.Mvc.Localization
@using ProductManagement.Localization
@using Volo.Abp.AspNetCore.Mvc.UI.Bootstrap.TagHelpers.Modal
@model ProductManagement.Web.Pages.Products.CreateProductModalModel
@inject IHtmlLocalizer<ProductManagementResource> L
@{
    Layout = null;
}
<abp-dynamic-form abp-model="Product"
                  asp-page="/Products/CreateProductModal">
    <abp-modal>
        <abp-modal-header title="@L["NewProduct"].Value"></abp-                  modal-header>
        <abp-modal-body>
            <abp-form-content />
        </abp-modal-body>
        <abp-modal-footer buttons="@(AbpModalButtons.Cancel|AbpModalButtons.Save)"></abp-modal-footer>
    </abp-modal>
</abp-dynamic-form>
```

在这里，`abp-dynamic-form` 会根据 C# 模型类自动创建表单元素。`abp-form-content` 是表单元素渲染的地方。`abp-modal` 用于创建模态对话框。

您可以使用标准的 Bootstrap HTML 元素和 ASP.NET Core 的绑定来创建表单元素。然而，ABP 的 Bootstrap 和动态表单标签助手大大简化了 UI 代码。我们将在 *第十二章*，*使用 MVC/Razor Pages* 中介绍 ABP 标签助手。

我们已经完成了产品创建模态代码。现在，我们将在 *Pages/Products* 文件夹中添加一个 `Index.cshtml` 文件，并按如下方式更改 `abp-card-header` 部分：

```cs
<abp-card-header>
    <abp-row>
        <abp-column size-md="_6">
            <abp-card-title>@L["Menu:Products"]</abp-card-                  title>
        </abp-column>
        <abp-column size-md="_6" class="text-end">
            <abp-button id="NewProductButton"
                        text="@L["NewProduct"].Value"
                        icon="plus"
                        button-type="Primary"/>
        </abp-column>
    </abp-row>
</abp-card-header>
```

我添加了 2 列，每列都有一个 `size-md="_6"` 属性（即 12 列 Bootstrap 网格的一半）。然后，我在左侧保留卡片标题的同时，在右侧放置了一个按钮。

在此之后，我在 `Index.cshtml.js` 文件的末尾添加了以下代码（在最后的 `});` 代码部分之前）：

```cs
var createModal = new abp.ModalManager(abp.appPath +                     'Products/CreateProductModal');
createModal.onResult(function () {
    dataTable.ajax.reload();
});
$('#NewProductButton').click(function (e) {
    e.preventDefault();
    createModal.open();
});
```

`abp.ModalManager` 用于在客户端管理模态对话框。内部，它使用 Twitter Bootstrap 的标准模态组件，但通过提供简单的 API 抽象了许多细节。`createModal.onResult()` 是当模态框保存时被调用的回调。`createModal.open();` 用于打开模态对话框。

最后，我们需要在 *ProductManagement.Domain.Shared* 项目的 *Localization/ProductManagement* 文件夹中的 `en.json` 文件中定义一些本地化文本：

```cs
"NewProduct": "New Product",
"Category": "Category",
"IsFreeCargo": "Free Cargo",
"ReleaseDate": "Release Date"
```

您可以再次运行 Web 应用程序并尝试创建一个新的产品：

![图 3.13 – 新产品模态框](img/Figure_3.13_B17287.jpg)

图 3.13 – 新产品模态框

ABP 已经根据 C# 类模型自动创建了表单字段。本地化和验证也会通过读取属性和使用约定自动完成。尝试留空名称字段并保存模态框，以查看验证错误消息的示例。我们将在 *第十二章*，*使用 MVC/Razor Pages* 中更详细地介绍验证和本地化。

现在，我们可以在 UI 上创建产品。现在，让我们看看如何编辑产品。

# 编辑产品

编辑产品与添加新产品类似。这次，我们需要获取要编辑的产品并准备编辑表单。

## 应用程序服务合约

让我们从为 `IProductAppService` 接口定义两个新方法开始：

```cs
Task<ProductDto> GetAsync(Guid id);
Task UpdateAsync(Guid id, CreateUpdateProductDto input);
```

第一个方法将用于通过 ID 获取产品数据。我们在 `UpdateAsync` 方法中重用了之前定义的 `CreateUpdateProductDto`。

我们还没有引入新的 DTO，因此我们可以直接进入实现。

## 应用服务实现

实现这些新方法相当简单。将以下方法添加到 `ProductAppService` 类中：

```cs
public async Task<ProductDto> GetAsync(Guid id)
{
    return ObjectMapper.Map<Product, ProductDto>(
        await _productRepository.GetAsync(id)
    );
}
public async Task UpdateAsync(Guid id, CreateUpdateProductDto input)
{
    var product = await _productRepository.GetAsync(id);
    ObjectMapper.Map(input, product);
}
```

`GetAsync` 方法使用 `productRepository.GetAsync` 从数据库中获取产品，并将其映射到 `ProductDto` 对象后返回。`UpdateAsync` 方法获取产品并将给定的输入属性映射到产品的属性。这样，我们就用新值覆盖了产品属性。

对于这个示例，我们不需要调用 `_productRepository.UpdateAsync`，因为 EF Core 有一个更改跟踪系统。ABP 的 **工作单元** 系统在请求结束时自动保存更改（如果没有抛出异常）。我们将在 *第六章*，*与数据访问基础设施一起工作* 中介绍工作单元系统。

应用层现在已完成。在下一节中，我们将创建产品编辑用户界面。

## 用户界面

在 *ProductManagement.Web* 项目的 *Pages/Products* 文件夹下创建一个新的 `EditProductModal.cshtml` Razor 页面。打开 `EditProductModal.cshtml.cs`，并使用以下代码更改内容：

```cs
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Rendering;
using ProductManagement.Products;
namespace ProductManagement.Web.Pages.Products
{
    public class EditProductModalModel :                             ProductManagementPageModel
    {
        [HiddenInput]
        [BindProperty(SupportsGet = true)]
        public Guid Id { get; set; }
        [BindProperty]
        public CreateEditProductViewModel Product { get; set; }
        public SelectListItem[] Categories { get; set; }
        private readonly IProductAppService _productAppService;
        public EditProductModalModel(IProductAppService                         productAppService)
        {
            _productAppService = productAppService;
        }
        public async Task OnGetAsync()
        {
            // TODO
        }
        public async Task<IActionResult> OnPostAsync()
        {
            // TODO
        }
    }
}
```

`Id` 属性将在表单中作为一个隐藏字段。它也应该支持 HTTP GET 请求，因为 GET 请求打开这个模态，我们需要产品的 ID 来准备编辑表单。`Product` 和 `Categories` 属性与创建模态类似。我们还在构造函数中注入了 `IProductAppService` 接口。

我们可以像以下代码块所示实现 `OnGetAsync` 方法：

```cs
public async Task OnGetAsync()
{
    var productDto = await _productAppService.GetAsync(Id);
    Product = ObjectMapper.Map<ProductDto,                                   CreateEditProductViewModel>(productDto);

    var categoryLookup = await                                               _productAppService.GetCategoriesAsync();
    Categories = categoryLookup.Items
        .Select(x => new SelectListItem(x.Name, x.Id.ToString()))
        .ToArray();
}
```

首先，我们正在获取要编辑的产品（`ProductDto`）。我们将其转换为 `CreateEditProductViewModel`，然后在 UI 中用于创建编辑表单。然后，我们获取表单上要选择的类别，就像我们之前为创建表单所做的那样。

我们已经将 `ProductDto` 映射到 `CreateEditProductViewModel`，因此现在我们需要在 `ProductManagementWebAutoMapperProfile` 类（在 *ProductManagement.Web* 项目中）中定义映射配置，就像我们之前做的那样：

```cs
CreateMap<ProductDto, CreateEditProductViewModel>();
```

`OnPostAsync` 方法很简单；我们通过将 `CreateEditProductViewModel` 转换为 `CreateUpdateProductDto` 来调用 `UpdateAsync` 方法：

```cs
public async Task<IActionResult> OnPostAsync()
{
    await _productAppService.UpdateAsync(Id,
        ObjectMapper.Map<CreateEditProductViewModel,                             CreateUpdateProductDto>(Product)
    );
    return NoContent();
}
```

现在，我们可以切换到 `EditProductModal.cshtml`，并按照以下方式更改其内容：

```cs
@page
@using Microsoft.AspNetCore.Mvc.Localization
@using ProductManagement.Localization
@using Volo.Abp.AspNetCore.Mvc.UI.Bootstrap.TagHelpers.Modal
@model ProductManagement.Web.Pages.Products.EditProductModalModel
@inject IHtmlLocalizer<ProductManagementResource> L
@{
    Layout = null;
}
<abp-dynamic-form abp-model="Product"
                  asp-page="/Products/EditProductModal">
    <abp-modal>
        <abp-modal-header title="@Model.Product.Name"></abp-modal-              header>
        <abp-modal-body>
            <abp-input asp-for="Id" />
            <abp-form-content/>
        </abp-modal-body>
        <abp-modal-footer buttons="@(AbpModalButtons.Cancel|AbpModalButtons.Save)"></abp-modal-footer>
    </abp-modal>
</abp-dynamic-form>
```

这个页面与 `CreateProductModal.cshtml` 非常相似。我只是向表单中添加了一个 `Id` 字段（作为一个隐藏输入），用于存储正在编辑的产品 `Id` 属性。

最后，我们可以添加一个 `Index.cshtml.js` 文件，并在 `dataTable` 初始化代码上方添加一个新的 `ModalManager` 对象：

```cs
var editModal = new abp.ModalManager(abp.appPath + 'Products/EditProductModal');
```

然后，在 `dataTable` 初始化代码中的 `columnDefs` 数组的第一项添加一个新的列定义：

```cs
{
    title: l('Actions'),
    rowAction: {
        items:
            [
                {
                    text: l('Edit'),
                    action: function (data) {
                        editModal.open({ id: data.record.id });
                    }
                }
            ]
    }
},
```

这段代码添加了一个新的 `rowAction`，这是 ABP 框架提供的一个特殊选项。它用于在表格的某一行添加一个或多个操作。

最后，在 `dataTable` 初始化代码之后添加以下代码：

```cs
editModal.onResult(function () {
    dataTable.ajax.reload();
});
```

此代码在保存产品编辑对话框后刷新数据表，这样我们可以在表格上看到最新的数据。最终的 UI 看起来类似于 *图 3.14*：

![图 3.14 – 编辑现有产品](img/Figure_3.14_B17287.jpg)

图 3.14 – 编辑现有产品

我们现在可以看到产品，创建新产品，以及编辑现有产品。最后一节将添加一个新操作来删除现有产品。

# 删除产品

与创建或编辑操作相比，删除产品操作相当简单，因为在这种情况下，我们不需要构建表单。首先，向 `IProductAppService` 接口添加一个新方法：

```cs
Task DeleteAsync(Guid id);
```

然后，在 `ProductAppService` 类中实现它：

```cs
public async Task DeleteAsync(Guid id)
{
    await _productRepository.DeleteAsync(id);
}
```

我们现在可以向产品数据表添加一个新操作。打开 `Index.cshtml.js`，并在 `rowAction.items` 数组之后添加以下定义）：

```cs
{
    text: l('Delete'),
    confirmMessage: function (data) {
        return l('ProductDeletionConfirmationMessage',                           data.record.name);
    },
    action: function (data) {
        productManagement.products.product
            .delete(data.record.id)
            .then(function() {
                abp.notify.info(l('SuccessfullyDeleted'));
                dataTable.ajax.reload();
            });
    }
}
```

在这里，`confirmMessage` 是一个在执行操作之前从用户那里获取确认的函数。`productManagement.products.product.delete` 函数是由 ABP 框架动态创建的，如前所述。这样，你可以在 JavaScript 代码中直接调用服务器端方法。我们只是传递当前记录的 ID。它返回一个承诺，这样我们就可以将回调注册到 `then` 函数。最后，我们使用 `abp.notify.info` 向用户发送通知，然后刷新数据表。

我们使用了一些本地化文本，因此需要将以下行添加到本地化文件中（位于 *ProductManagement.Domain.Shared* 项目的 *Localization/ProductManagement* 文件夹中的 `en.json` 文件）：

```cs
"ProductDeletionConfirmationMessage": "Are you sure to delete this book: {0}",
"SuccessfullyDeleted": "Successfully deleted!"
```

你可以再次运行 Web 项目以查看结果：

![图 3.15 – 删除操作](img/Figure_3.15_B17287.jpg)

图 3.15 – 删除操作

由于我们现在有两个操作，**编辑**按钮自动变成了**操作**下拉按钮。当你点击**删除**操作时，你会看到一个确认消息来删除产品：

![图 3.16 – 删除确认消息](img/Figure_3.16_B17287.jpg)

图 3.16 – 删除确认消息

如果你点击**是**按钮，你将在页面上看到一个通知，数据表将被刷新。

实现产品删除功能相当简单。ABP 内置的功能通过实现常见的模式，如客户端到服务器的通信、确认对话框和 UI 通知，帮助我们完成了这一功能。

注意，`Product`实体是从`FullAuditedAggregateRoot`类继承的，这使得它支持软删除。在删除一个产品后检查数据库。你会发现它并没有真正被删除，而是`IsDeleted`字段被设置为`true`。将`IsDeleted`设置为`true`使得产品实体软删除（即逻辑上删除但物理上未删除）。下次你查询产品时，已删除的产品将自动过滤，不包括在查询结果中。这是通过 ABP 框架的数据过滤系统完成的，将在*第六章*中介绍，*与数据访问基础设施一起工作*。

# 摘要

在本章中，我们创建了一个完全工作的 CRUD 页面。我们走过了应用程序的所有层，并看到了基于 ABP 的应用程序开发的根本方法。

你已经接触到了许多不同的概念，例如实体、存储库、数据库映射和迁移、自动化测试、API 控制器、动态 JavaScript 代理、对象到对象映射、软删除等。如果你正在构建一个严肃的软件解决方案，你将使用所有这些，无论是否使用 ABP。ABP 是一个全栈应用程序框架，它帮助你以最佳实践实现这些概念。它提供了必要的基础设施，使你的日常开发更加容易。

你可能现在无法理解所有细节。这不是问题，因为剩余章节的目的是深入探讨这些概念，并展示它们的细节和不同的用例。

这个示例应用程序相对简单。它不包含任何重要的业务逻辑，因为我在其中引入了许多概念，并试图保持应用程序简单，以便专注于这些概念而不是业务复杂性。在这个示例中，我忽略了授权。授权将在*第七章*中解释，*探索横切关注点*。

在书中展示具有真实世界复杂性的示例应用程序并不容易。然而，我为本书的读者准备了一个具有真实世界质量和复杂性的完整参考应用程序。这个参考应用程序是开源的，可在 GitHub 上找到。此外，它是一个实时应用程序，所以你可以直接尝试它。

下一章将介绍这个参考应用程序，并展示参考解决方案的功能、层和代码结构。剩余章节经常引用该应用程序的源代码。
