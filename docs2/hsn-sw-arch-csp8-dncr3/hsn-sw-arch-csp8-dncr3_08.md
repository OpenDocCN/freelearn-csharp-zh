# 在 C# 中与数据交互 - Entity Framework Core

正如我们在第五章中提到的，*将微服务架构应用于企业应用程序*，软件系统被组织成层，并且每一层通过不依赖于层实现方式的接口与前一层和后一层进行通信。当软件是业务/企业系统时，它通常至少包含三个层：数据层、业务层和表示层。一般来说，每一层提供的接口以及层的实现方式取决于应用程序。

然而，结果证明数据层提供的功能相当标准化，因为它们只是将数据从数据存储子系统映射到对象，反之亦然。这导致了在相当声明式的方式下实现数据层的通用框架的概念。这些工具被称为 **对象关系映射**（**ORM**）工具，因为它们是基于关系数据库的数据存储子系统。然而，它们与现代非关系存储（如 MongoDB 和 Azure Cosmos DB）也很好地协同工作，因为它们的数据模型比纯关系模型更接近目标对象模型。

本章将涵盖以下主题：

+   理解 ORM 基础知识

+   配置 Entity Framework Core

+   Entity Framework Core 迁移

+   使用 Entity Framework Core 查询和更新数据

+   部署您的数据层

+   理解 Entity Framework Core 高级功能 - 全局过滤器

本章描述了 ORM 以及如何对其进行配置，然后重点介绍了 Entity Framework Core，它是 .NET Core 中包含的 ORM。

# 技术要求

本章需要 Visual Studio 2017 或 2019 免费社区版或更高版本，并安装所有数据库工具。

本章中的所有概念都将通过基于 WWTravelClub 书籍用例的实用示例进行说明。您可以在[`github.com/PacktPublishing/Hands-On-Software-Architecture-with-CSharp-8`](https://github.com/PacktPublishing/Hands-On-Software-Architecture-with-CSharp-8)找到本章的代码。

# 理解 ORM 基础知识

ORM 将关系数据库表映射到内存中的对象集合，其中对象属性对应于数据库表字段。来自 C# 的类型，如布尔值、数值类型和字符串，有对应的数据库类型。如果映射的数据库中没有 GUID，而单个字符映射到数据库的单字符字符串，那么像 GUID 这样的类型则映射到它们的等效字符串表示。所有日期和时间类型要么映射到 C# 的 `DateTime`（当日期/时间不包含时区信息时），要么映射到 `DateTimeOffset`（当日期/时间也包含显式时区信息时）。任何数据库时间长度都映射到 `TimeSpan`。

由于大多数面向对象语言的字符串属性没有与之关联的长度限制（而数据库字符串字段通常有长度限制），因此在数据库映射配置中考虑了数据库限制。一般来说，当需要指定数据库类型和面向对象语言类型之间的映射选项时，这些选项在映射配置中声明。

整个配置的定义方式取决于具体的 ORM。Entity Framework Core 提供了三种选项：

+   数据注释（属性属性）

+   命名约定

+   基于配置对象和方法的流畅配置接口

虽然流畅式接口可以用来指定任何配置选项，但数据注释和命名约定可以用于其中较小的一部分。

每个 ORM 都适应特定的数据库类型（Oracle、MySQL、SQL Server 等），并使用称为**提供程序**或**连接器**的数据库特定适配器。Entity Framework Core 为大多数可用的数据库引擎提供了提供程序。

可以在[`docs.microsoft.com/en-US/ef/core/providers/`](https://docs.microsoft.com/en-US/ef/core/providers/)找到提供程序的完整列表。

适配器对于数据库类型之间的差异、事务处理方式以及 SQL 语言未标准化的所有其他功能都是必要的。

表之间的关系用对象指针表示。例如，在一对多关系中，映射到关系一方的类包含一个集合，该集合填充了关系多方相关的对象。另一方面，映射到关系多方的类有一个简单的属性，该属性填充了关系一方唯一相关的对象。

整个数据库（或其一部分）由一个内存缓存类表示，该类包含映射到数据库表的每个集合的属性。首先，在内存缓存类的实例上执行查询和更新操作，然后与此实例同步数据库。Entity Framework Core 使用的内存缓存类称为`DBContext`，它还包含映射配置。更具体地说，通过继承`DBContext`并向所有映射集合和所有必要的配置信息添加，可以获得特定于应用程序的内存缓存类。

总结来说，`DBContext`子类实例包含与数据库同步的数据库的部分快照，以获取/更新实际数据。

数据库查询是通过在内存缓存类集合上调用方法来执行的查询语言。实际的 SQL 是在同步阶段创建和执行的。例如，Entity Framework Core 在映射到数据库表的集合上执行**语言集成查询**（**LINQ**）。

通常，LINQ 查询生成`IEnumerable`实例，即元素在查询结束时创建`IEnumerable`时并未计算，而是在你实际尝试从`IEnumerable`检索集合元素时计算。其工作原理如下：

+   从`DBContext`的映射集合开始的 LINQ 查询创建一个名为`IQueryable`的特定`IEnumerable`子类。

+   `IQueryable` 包含发出数据库查询所需的所有信息，但实际的 SQL 是在检索`IQueryable`的第一个元素时生成和执行的。

+   因此，在 Entity Framework Core 的情况下，数据库同步是在从最终的`IQueryable`检索元素时执行的。

+   通常，每个 Entity Framework 查询都以`ToList`或`ToArray`操作结束，该操作将`IQueryable`转换为列表或数组，从而在数据库上实际执行查询。

+   如果预期查询将仅返回单个元素或根本不返回任何元素，我们通常执行一个`FirstOrDefault`操作，该操作返回一个元素（如果有的话），或者返回`null`。

此外，对 DB 表的新实体进行更新、删除和添加操作是通过模拟在表示数据库表的`DBContext`集合属性上执行这些操作来完成的。然而，实体只能通过查询将其加载到内存集合中后以这种方式更新或删除。更新查询需要根据需要修改实体的内存表示，而删除查询则需要从其内存映射集合中删除实体的内存表示。在 Entity Framework Core 中，通过调用集合的`Remove(entity)`方法执行删除操作。

添加新实体没有其他要求。只需将新实体添加到内存集合中即可。在内存集合上执行的各种更新、删除和添加操作实际上是通过显式调用 DB 同步方法传递给数据库的。例如，当你调用`DBContext.SaveChanges()`方法时，Entity Framework Core 会将`DBContext`实例上执行的所有更改传递到数据库。

在同步操作期间传递给数据库的更改将在单个事务中执行。此外，对于具有显式事务表示的 ORM，例如 Entity Framework Core，同步操作是在事务的作用域内执行的，因为它使用该事务而不是创建一个新的。

本章的其余部分解释了如何使用 Entity Framework Core，以及一些基于本书 WWTravelClub 用例的示例代码。

# 配置 Entity Framework Core

由于数据库处理被限制在专用应用程序层内，将 Entity Framework Core (`DBContext`)定义在单独的库中是一种良好的做法。相应地，我们需要定义一个.NET Core 类库项目。正如我们在第二章，*功能和非功能需求*部分的*书籍用例 - .NET Core 的实际应用，.NET Core 项目的类型*中讨论的，我们有两种不同类型的库项目：**.NET** **Standard** 和 **.NET Core**。

虽然.NET Core 库与特定的.NET Core 版本相关联，但.NET Standard 2.0 库具有广泛的应用范围，因为它们可以与任何大于 2.0 的.NET 版本以及经典的.NET Framework 一起工作。

然而，`Microsoft.EntityFrameworkCore`包（我们在数据库层需要它）仅依赖于.NET Standard 2.0。它被设计为与特定的.NET Core 版本一起工作（其版本号与.NET Core 版本相同）。因此，如果我们定义我们的数据库层为.NET Standard 2.0，我们添加作为依赖项的特定`Microsoft.EntityFrameworkCore`包可能与另一个系统组件中包含的同一库的另一个版本发生冲突，该组件与特定的.NET Core 版本相关联。

由于我们的库不是通用库（它只是特定应用程序的一个组件），我们更倾向于将其绑定到特定的.NET Core 版本，而不是在整个应用程序设计中跟踪其版本依赖关系。因此，让我们选择一个.NET Core 库项目，用于安装在我们机器上的最新.NET Core 版本。我们的.NET Core 库项目可以创建和准备如下：

1.  打开 Visual Studio，定义一个名为`WWTravelClubDB`的新解决方案，然后选择适用于最新.NET Core 版本的类库 (.NET Core)。

1.  我们必须安装所有与 Entity Framework Core 相关的依赖项。安装所有必需依赖项的最简单方法是将我们打算使用的数据库引擎提供者的 NuGet 包添加到项目中——在我们的例子中，是 SQL Server——正如我们在第四章，*选择最佳云解决方案*中提到的。实际上，任何提供者都会安装所有所需的包，因为它将它们作为依赖项。因此，让我们添加`Microsoft.EntityFrameworkCore.SqlServer`的最新稳定版本。如果你打算使用多个数据库引擎，你也可以添加其他提供者，因为它们可以并行工作。在本章的后面部分，我们将安装其他包含我们需要的工具的 NuGet 包，然后我们将解释如何安装进一步处理 Entity Framework Core 配置所需的工具。

1.  让我们将默认的`Class1`类重命名为`MainDBContext`。这被自动添加到类库中。

1.  现在，让我们用以下代码替换其内容：

```cs
using System;
using Microsoft.EntityFrameworkCore;

namespace WWTravelClubDB
{
    public class MainDBContext: DbContext
    {
        public MainDBContext(DbContextOptions options)
            : base(options)
        {
        }
        protected override void OnModelCreating(ModelBuilder 
        builder)
        {
        } 
    }
}
```

1.  我们从`DbContext`继承，并且必须将`DbContextOptions`传递给`DBContext`构造函数。`DbContextOptions`包含创建选项，如数据库连接字符串，这些选项取决于目标数据库引擎。

1.  所有已映射到数据库表中的集合都将作为`MainDBContext`的属性添加。映射配置将在重写的`OnModelCreating`方法中定义，该方法通过参数传递的`ModelBuilder`对象来实现。

下一步是创建所有代表数据库表行的类。这些被称为**实体**。我们需要为每个想要映射的数据库表创建一个实体类。让我们在项目根目录下创建一个`Models`文件夹来存放所有这些类。下一个小节将解释如何定义所有必需的实体。

# 定义数据库实体

数据库设计，就像整个应用程序设计一样，是有组织的迭代过程。让我们假设在第一次迭代中，我们需要一个包含两个数据库表的原型：一个用于所有旅行套餐，另一个用于所有由套餐引用的位置。每个套餐只覆盖一个位置，而单个位置可能被多个套餐覆盖，因此两个表通过一对一的关系连接。

因此，让我们从位置数据库表开始。正如我们在上一节末尾提到的，我们需要一个实体类来表示这个表的行。让我们将实体类命名为`Destination`：

```cs
namespace WWTravelClubDB.Models
{
    public class Destination
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Country { get; set; }
        public string Description { get; set; }
    }
}
```

所有数据库字段都必须由可读/写的 C#属性表示。假设每个目的地就像一个城镇或地区，可以通过其名称和所在国家来定义，并且所有相关信息都包含在其`Description`中。在未来的迭代中，我们可能会添加更多字段。`Id`是一个自动生成的键。

然而，现在，我们需要添加有关所有字段如何映射到数据库字段的信息。在 Entity Framework Core 中，所有原始类型都由特定于数据库引擎的提供程序自动映射到数据库类型（在我们的情况下是 SQL Server 提供程序）。我们唯一关心的是以下内容：

+   **字符串长度限制**：可以通过为每个字符串属性应用适当的`MaxLength`和`MinLength`属性来考虑。所有对实体配置有用的属性都包含在`System.ComponentModel.DataAnnotation`和`System.ComponentModel.DataAnnotations.Schema`命名空间中。因此，将它们都添加到所有实体定义中是一个好的实践。

+   **指定哪些字段是必需的，哪些是可选的**：默认情况下，所有引用类型（如所有字符串）都被假定为可选的，而所有值类型（如数字和 GUID）都被假定为必需的。如果我们想使一个引用类型成为必需的，那么我们必须用`Required`属性来装饰它。然而，如果我们想使`T`值类型属性成为可选的，那么我们必须用`T?`来替换它。

+   **指定哪个属性代表主键**：键可以通过使用 `Key` 属性来指定。然而，如果没有找到 `Key` 属性，则将名为 `Id` 的属性（如果有的话）视为主键。在我们的案例中，不需要 `Key` 属性。如果主键由多个属性组成，只需将 `Key` 属性添加到所有这些属性上即可。

由于每个目的地在一对多关系的一侧，它必须包含一个相关包实体的集合；否则，我们无法在我们的 LINQ 查询子句中引用相关实体。

将一切整合，`Destination` 类的最终版本如下：

```cs
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace WWTravelClubDB.Models
{
    public class Destination
    {
        public int Id { get; set; }
        [MaxLength(128), Required]
        public string Name { get; set; }
        [MaxLength(128), Required]
        public string Country { get; set; }
        public string Description { get; set; }
        public ICollection<Package> Packages { get; set; }
    }
}
```

由于 `Description` 属性没有长度限制，它将使用不定长度的 SQL Server `ntext` 字段实现。我们可以用类似的方式编写 `Package` 类的代码：

```cs
using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
namespace WWTravelClubDB.Models
{
    public class Package
    {
        public int Id { get; set; }
        [MaxLength(128), Required]
        public string Name { get; set; }
        [MaxLength(128)]
        public string Description { get; set; }
        public decimal Price { get; set; }
        public int DuratioInDays { get; set; }
        public DateTime? StartValidityDate { get; set; }
        public DateTime? EndValidityDate { get; set; }
        public Destination MyDestination { get; set; }
        public int DestinationId { get; set; }
    }
}
```

每个包都有持续的天数，以及可选的开始和结束日期，这些日期是包优惠有效的日期。`MyDestination` 通过与 `Destination` 实体之间的一对多关系将包与其目的地连接起来，而 `DestinationId` 是同一关系的外部键。

虽然指定外部键不是强制性的，但这是一个好的实践，因为这是指定关系某些属性的唯一方式。例如，在我们的案例中，由于 `DestinationId` 是一个 `int`（值类型），这是强制性的。因此，这里的关系是一对多，而不是（0，1）到多。将 `DestinationId` 定义为 `int?`，而不是 `int`，将使一对多关系变为（0，1）到多关系。

在下一节中，我们将解释如何定义表示数据库表的内存集合。

# 定义映射的集合

一旦我们定义了所有表示数据库行面向对象的实体，我们需要定义表示数据库表的内存集合。正如我们在 *ORM 基础* 小节中提到的，所有数据库操作都映射到这些集合的操作上（本章的 *使用 Entity Framework Core 查询和更新数据* 小节解释了 *如何*）。对于每个实体 `T`，我们只需要在我们的 `DBContext` 中为每个实体添加一个 `DbSet<T>` 集合属性。通常，这些属性的名称是通过将实体名称复数化得到的。因此，我们需要向我们的 `MainDBContext` 添加以下两个属性：

```cs
public DbSet<Package> Packages { get; set; }
public DbSet<Destination> Destinations { get; set; }
```

到目前为止，我们已经将数据库内容转换为属性、类和数据注释。然而，Entity Framework 需要更多信息才能与数据库交互。下一小节将解释如何提供这些信息。

# 完成映射配置

在实体定义中无法指定的映射配置信息必须在 `OnModelCreating DBContext` 方法中添加。每个与实体 `T` 相关的配置信息从 `builder.Entity<T>()` 开始，然后通过调用指定该类型约束的方法继续。进一步嵌套的调用指定约束的进一步属性。例如，我们的多对一关系可以配置如下：

```cs
builder.Entity<Destination>()
    .HasMany(m => m.Packages)
    .WithOne(m => m.MyDestination)
    .HasForeignKey(m => m.DestinationId)
    .OnDelete(DeleteBehavior.Cascade);
```

关系的两侧通过我们添加到实体的导航属性来指定。`HasForeignKey` 指定外部键。最后，`OnDelete` 指定在删除目标时对包的处理。在我们的例子中，它执行与该目标相关的所有包的级联删除。

同样的配置可以从关系的另一端开始定义，即从 `builder.Entity<Package>()` 开始：

```cs
builder.Entity<Package>()
    .HasOne(m => m.MyDestination)
    .WithMany(m => m.Packages)
    .HasForeignKey(m => m.DestinationId)
    .OnDelete(DeleteBehavior.Cascade);
```

唯一的区别是，由于我们从关系的另一端开始，所以之前的 `HasMany`-`WithOne` 方法被 `HasOne`-`WithMany` 方法所取代。

`ModelBuilder builder` 对象允许我们使用如下方式指定数据库索引：

```cs
builder.Entity<T>()
   .HasIndex(m => m.PropertyName);
```

多属性索引的定义如下：

```cs
builder.Entity<T>()
    .HasIndex("propertyName1", "propertyName2", ...);
```

如果我们添加所有必要的配置信息，那么我们的 `OnModelCreating` 方法将如下所示：

```cs
protected override void OnModelCreating(ModelBuilder builder)
{
    builder.Entity<Destination>()
        .HasMany(m => m.Packages)
        .WithOne(m => m.MyDestination)
        .HasForeignKey(m => m.DestinationId)
        .OnDelete(DeleteBehavior.Cascade);

    builder.Entity<Destination>()
        .HasIndex(m => m.Country);

    builder.Entity<Destination>()
        .HasIndex(m => m.Name);

    builder.Entity<Package>()
 .HasIndex(m => m.Name);

    builder.Entity<Package>()
        .HasIndex("StartValidityDate", "EndValidityDate");
} 
```

一旦配置了 Entity Framework Core，我们就可以使用所有配置信息来创建实际的数据库，并将所有需要的工具放置到位，以便随着应用程序的发展更新数据库的结构。下一节将解释如何操作。

# Entity Framework Core 迁移

现在我们已经配置了 Entity Framework 并定义了我们的应用程序特定的 `DBContext` 子类，我们可以使用 Entity Framework Core 设计工具生成物理数据库并创建 Entity Framework Core 与数据库交互所需的数据库结构快照。

Entity Framework Core 设计工具必须作为 NuGet 包安装在每个需要它们的项目中。有两个等效选项：

+   **在任何 Windows 控制台中工作的工具**：这些通过 `Microsoft.EntityFrameworkCore.Design` NuGet 包提供。所有 Entity Framework Core 命令都在 `dotnet ef ......` 格式中，因为它们包含在 .NET Core 命令行应用程序的 `ef` 命令中。

+   **Visual Studio 特定的工具**：这些包含在 `Microsoft.EntityFrameworkCore.Tools` NuGet 包中。由于它们只能从 Visual Studio 内部的包管理器控制台启动，因此不需要 `dotnet ef` 前缀。

Entity Framework Core 的设计工具在设计/更新过程中使用。该过程如下：

1.  我们根据需要修改 `DBContext` 和实体的定义。

1.  我们启动设计工具，要求 Entity Framework Core 检测和处理我们所做的所有更改。

1.  一旦启动，设计工具将更新数据库结构快照并生成一个新的*迁移*，即包含所有我们需要修改物理数据库以反映所有更改的指令的文件。

1.  我们推出另一个工具来更新数据库，以包含新创建的迁移。

1.  我们测试新配置的 DB 层，如果需要新的更改，我们将返回到*步骤 1*。

1.  当数据层准备就绪时，它将在预发布或生产环境中部署，此时所有迁移将再次应用到实际的预发布/生产数据库中。

这在各个软件项目迭代和应用程序的生命周期中重复多次。如果我们操作的是已经存在的数据库，我们需要配置`DBContext`及其模型以反映我们想要映射的所有表的现有结构。然后，我们使用`IgnoreChanges`选项调用设计工具，以便它们生成一个空迁移。此外，这个空迁移必须传递到物理数据库，以便它可以同步与物理数据库关联的数据库结构版本与数据库快照中记录的版本。这个版本很重要，因为它决定了哪些迁移必须应用到数据库以及哪些已经应用。

整个设计过程需要一个测试/设计数据库，如果我们操作的是现有数据库，则该测试/设计数据库的结构必须反映实际数据库——至少在我们想要映射的表方面。为了使设计工具能够与数据库交互，我们必须定义它们传递给`DBContext`构造函数的`DbContextOptions`选项。这些选项在设计时很重要，因为它们包含测试/设计数据库的连接字符串。如果我们创建一个实现`IDesignTimeDbContextFactory<T>`接口的类，其中`T`是我们的`DBContext`子类，设计工具就可以了解我们的`DbContextOptions`选项：

```cs
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Design;

namespace WWTravelClubDB
{
    public class LibraryDesignTimeDbContextFactory
        : IDesignTimeDbContextFactory<MainDBContext>
    {
        private const string connectionString =
            @"Server=(localdb)\mssqllocaldb;Database=wwtravelclub;
                Trusted_Connection=True;MultipleActiveResultSets=true";
        public MainDBContext CreateDbContext(params string[] args)
        {
            var builder = new DbContextOptionsBuilder<MainDBContext>();

            builder.UseSqlServer(connectionString);
            return new MainDBContext(builder.Options);
        }
    }
}
```

`connectionString`将由 Entity Framework 用于在开发机器上安装的本地 SQL Server 实例中创建一个新的数据库，并通过 Windows 凭据进行连接。你可以自由地更改它以反映你的需求。

现在，我们准备好创建我们的第一个迁移了！让我们开始吧：

1.  让我们转到包管理控制台，并确保 WWTravelClubDB 被选为我们的默认项目。

1.  现在，输入`Add-Migration initial`并按*Enter*键来执行此命令：

![图片](img/d81b11a6-78ee-4f9f-b286-002c0b88ddb1.png)

`initial`是我们给第一个迁移取的名字。所以，一般来说，命令是`Add-Migration <迁移名称>`。当我们操作现有数据库时，我们必须将`-IgnoreChanges`选项添加到第一个迁移（仅添加到该迁移）以创建一个空迁移。有关所有命令的引用可以在*进一步阅读*部分找到。

1.  如果在创建迁移之后但在将迁移应用到数据库之前，我们意识到我们犯了错误，我们可以使用`Remove-Migration`命令撤销我们的操作。如果迁移已经应用到数据库，纠正我们的错误的最简单方法是对代码进行所有必要的更改，然后应用另一个迁移。

1.  一旦执行了`Add-Migration`命令，我们的项目中就会出现一个新的文件夹：

![图片](img/bb865a32-ee95-460d-ab24-c9b86ee779a6.png)

`20190205102637_initial.cs`是我们用易于理解的语言表达的迁移。

您可以审查代码以验证一切是否正常，并且您也可以修改迁移内容（只有当您足够专业才能可靠地完成时）。每个迁移都包含一个`Up`方法和一个`Down`方法。`Up`方法表示迁移，而`Down`方法则撤销其更改。因此，`Down`方法包含`Up`方法中包含的所有操作的逆序操作。

`20190205102637_initial.Designer.cs`是您*必须不*修改的 Visual Studio 设计器代码，而`MainDBContextModelSnapshot.cs`是整体数据库结构快照。如果您添加了更多的迁移，新的迁移文件及其设计器对应文件将出现，并且唯一的`MainDBContextModelSnapshot.cs`数据库结构快照将被更新以反映数据库的整体结构。

您可以在 Windows 控制台中通过输入`dotnet ef migrations add initial`来发出相同的命令。然而，此命令必须从项目的根目录（而不是从解决方案的根目录）发出。

您可以通过在包管理器控制台中输入`Update-Database`来将迁移应用到数据库。等效的 Windows 控制台命令是`dotnet ef database update`。让我们尝试使用这个命令来创建物理数据库！

下一个子节解释了如何创建 Entity Framework 无法自动创建的数据库内容。之后，在下一节中，我们将使用 Entity Framework 的配置以及我们使用`dotnet ef database update`生成的数据库来创建、查询和更新数据。

# 理解存储过程和直接 SQL 命令

一些数据库结构不能通过我们之前描述的 Entity Framework Core 命令和声明自动生成。例如，Entity Framework Core 不能自动生成存储过程。如通用 SQL 字符串之类的存储过程可以通过`migrationBuilder.Sql("<sql scommand>")`方法手动包含在`Up`和`Down`方法中。

做这件事最安全的方式是添加一个不执行任何配置更改的迁移，这样在创建时迁移是空的。然后，我们可以将必要的 SQL 命令添加到这个迁移的空 `Up` 方法中，以及它们的逆命令在空 `Down` 方法中。将所有 SQL 字符串放在资源文件（`.resx` 文件）的属性中是一种良好的做法。

现在，你已经准备好通过 Entity Framework Core 与数据库进行交互了。

# 使用 Entity Framework Core 查询和更新数据

为了测试我们的数据库层，我们需要将一个基于与我们的库相同 .NET Core 版本的控制台项目添加到解决方案中。让我们开始吧：

1.  让我们将新的控制台项目命名为 `WWTravelClubDBTest`。

1.  现在，我们需要通过在控制台项目的 *References* 节点右键单击并选择 *Add reference* 来将我们的数据层作为控制台项目的依赖项添加。

1.  从 `program.cs` 文件中的 `Main` 静态方法中删除内容，并开始编写以下内容：

```cs
Console.WriteLine("program start: populate database");
Console.ReadKey();
```

1.  然后，在文件顶部添加以下命名空间：

```cs
using WWTravelClubDB;
using WWTravelClubDB.Models;
using Microsoft.EntityFrameworkCore;
using System.Linq;
```

现在我们已经完成了测试项目的准备工作，我们可以尝试查询和数据更新。让我们首先创建一些数据库对象，即一些目的地和包。按照以下步骤操作：

1.  首先，我们必须使用适当的连接字符串创建我们的 `DBContext` 子类的一个实例。我们可以使用设计工具使用的相同 `LibraryDesignTimeDbContextFactory` 类来获取它：

```cs
var context = new LibraryDesignTimeDbContextFactory()
    .CreateDbContext(); 
```

1.  可以通过简单地将类实例添加到我们 `DBContext` 子类的映射集合中来创建新行。如果一个 `Destination` 实例与包相关联，我们可以简单地将其添加到其 `Packages` 属性中：

```cs
var firstDestination= new Destination
{
    Name = "Florence",
    Country = "Italy",
    Packages = new List<Package>()
    {
        new Package
        {
            Name = "Summer in Florence",
            StartValidityDate = new DateTime(2019, 6, 1),
            EndValidityDate = new DateTime(2019, 10, 1),
            DuratioInDays=7,
            Price=1000
        },
        new Package
        {
            Name = "Winter in Florence",
            StartValidityDate = new DateTime(2019, 12, 1),
            EndValidityDate = new DateTime(2020, 2, 1),
            DuratioInDays=7,
            Price=500
        }
    }
};
context.Destinations.Add(firstDestination);
context.SaveChanges();
Console.WriteLine(
    "DB populated: first destination id is "+
    firstDestination.Id);
Console.ReadKey();
```

没有必要指定主键，因为它们是自动生成的，将由数据库填充。实际上，在 `SaveChanges()` 操作同步我们的上下文与实际数据库后，`firstDestination.Id` 属性有一个非零值。对于 `Package` 的主键也是如此。

当我们通过将其插入父实体集合（在我们的例子中是 `Packages` 集合）来声明一个实体（在我们的例子中是 `Package`）是另一个实体（在我们的例子中是 `Destination`）的子实体时，没有必要显式设置其外键（在我们的例子中是 `DestinationId`），因为 Entity Framework Core 会自动推断它。一旦创建并与 `firstDestination` 数据库同步，我们可以通过两种不同的方式添加更多包：

+   创建一个 `Package` 类实例，将其 `DestinationId` 外键设置为 `firstDestination.Id`，并将其添加到 `context.Packages`

+   创建一个不需要设置外键的 `Package` 类实例，然后将其添加到其父 `Destination` 实例的 `Packages` 集合中。

后一种选项是唯一的选择，当添加具有其父实体（`Destination`）的子实体（`Package`）且父实体具有自动生成的主键时，在这种情况下，外部键在我们执行添加操作时不可用。在大多数其他情况下，前一种选项更简单，因为第二种选项需要将父`Destination`实体及其`Packages`集合加载到内存中，即与所有与`Destination`对象关联的套餐一起（默认情况下，通过查询不会加载连接的实体）。

现在，假设我们想修改*佛罗伦萨*目的地，并将所有`佛罗伦萨`套餐的价格提高 10%。我们该如何操作？按照以下步骤了解如何进行：

1.  首先，我们需要通过查询将实体加载到内存中，修改它，并调用`SaveChanges()`来同步我们的更改与数据库。如果我们只想修改，比如描述，以下查询就足够了：

```cs
var toModify = context.Destinations
    .Where(m => m.Name == "Florence").FirstOrDefault();
```

1.  我们需要加载所有默认未加载的相关目的地套餐。这可以通过`Include`子句完成，如下所示：

```cs
var toModify = context.Destinations
    .Where(m => m.Name == "Florence")
    .Include(m => m.Packages)
    .FirstOrDefault();
```

1.  之后，我们可以修改描述和套餐价格，如下所示：

```cs
toModify.Description = 
  "Florence is a famous historical Italian town";
foreach (var package in toModify.Packages)
   package.Price = package.Price * 1.1m;
context.SaveChanges();

var verifyChanges= context.Destinations
    .Where(m => m.Name == "Florence")
    .FirstOrDefault();

Console.WriteLine(
    "New Florence description: " +
    verifyChanges.Description);
Console.ReadKey();
```

到目前为止，我们执行了查询，其唯一目的是更新检索到的实体。接下来，我们将解释如何检索将显示给用户以及/或用于复杂业务操作的信息。

# 将数据返回到表示层

为了保持层之间的分离，并使查询适应每个*用例*实际需要的数据，数据库实体不会直接发送到表示层。相反，数据被投影到包含*用例*所需信息的更小的类中。这些类由表示层的调用方法实现。在将数据从一个层移动到另一个层的对象被称为**数据传输对象**（**DTOs**）。例如，让我们创建一个包含当向用户返回套餐列表时值得显示的摘要信息的 DTO（我们假设如果需要，用户可以通过点击他们感兴趣的套餐来获取更多详细信息）：

1.  让我们在我们的 WWTravelClubDBTest 项目中添加一个 DTO，其中包含需要在套餐列表中显示的所有信息：

```cs
namespace WWTravelClubDBTest
{
    public class PackagesListDTO
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
        public int DuratioInDays { get; set; }
        public DateTime? StartValidityDate { get; set; }
        public DateTime? EndValidityDate { get; set; }
        public string DestinationName { get; set; }
        public int DestinationId { get; set; }
        public override string ToString()
        {
            return string.Format("{0}. {1} days in {2}, price: 
            {3}", Name, DuratioInDays, DestinationName, Price);
        }
    }
}
```

我们不需要在内存中加载实体并将它们的数据复制到 DTO 中，但数据库数据可以直接投影到 DTO 中，这得益于 LINQ 的`Select`子句。这最小化了与数据库交换的数据量。

1.  例如，我们可以使用查询来填充我们的 DTOs，该查询检查所有在 8 月 10 日左右可用的套餐：

```cs
var period = new DateTime(2019, 8, 10);
var list = context.Packages
    .Where(m => period >= m.StartValidityDate
    && period <= m.EndValidityDate)
    .Select(m => new PackagesListDTO
    {
        StartValidityDate=m.StartValidityDate,
        EndValidityDate=m.EndValidityDate,
        Name=m.Name,
        DuratioInDays=m.DuratioInDays,
        Id=m.Id,
        Price=m.Price,
        DestinationName=m.MyDestination.Name,
        DestinationId = m.DestinationId
    })
    .ToList();
foreach (var result in list)
    Console.WriteLine(result.ToString());
Console.ReadKey();
```

1.  在`Select`子句中，我们还可以导航到任何相关实体以获取所需的数据。例如，前面的查询导航到相关的`Destination`实体以获取`Package`的目的地名称。

1.  现在，在解决方案资源管理器中右键单击 WWTravelClubDBTest 项目，将其设置为启动项目。然后，运行解决方案。

1.  程序在每个 `Console.ReadKey()` 方法处停止，等待您按任意键。这样，您就有时间分析所有代码片段产生的输出，这些代码片段都已添加到 `Main` 方法中。

现在，我们将学习如何处理无法有效地映射到表示数据库表的内存集合的即时操作的运算。

# 发出直接 SQL 命令

并非所有数据库操作都可以通过使用 LINQ 查询数据库并在内存中更新实体来高效执行。例如，计数器增加可以通过单个 SQL 指令更高效地执行。此外，如果我们定义了适当的存储过程/SQL 命令，一些操作可以以可接受的性能执行。在这些情况下，我们被迫直接向数据库发出 SQL 命令或从我们的 Entity Framework 代码中调用数据库存储过程。有两种可能性：执行数据库操作但不返回实体的 SQL 语句，以及返回实体的 SQL 语句。

不返回实体的 SQL 命令可以使用 `DBContext` 方法执行，如下所示：

```cs
int DBContext.Database.ExecuteSqlRaw(string sql, params object[] parameters)
```

参数可以在字符串中以 `{0}, {1}, ..., {n}` 的形式引用。每个 `{m}` 都会被填充到 `parameters` 数组的 `m` 索引处的对象，该对象从 .NET 类型转换为相应的 SQL 类型。该方法返回受影响的行数。

返回实体集合的 SQL 命令必须通过与这些实体关联的映射集合的 `FromSqlRaw` 方法发出：

```cs
context.<mapped collection>.FromSqlRaw(string sql, params object[] parameters)
```

因此，例如，返回 `Package` 实例的命令可能看起来像这样：

```cs
var results = context.Packages.FromSqlRaw("<some sql>", par1, par2, ...).ToList();
```

在 `ExecuteSqlRaw` 方法中，SQL 字符串和参数是这样工作的。以下是一个简单的示例：

```cs
var allPackages =context.Packages.FromSqlRaw(
    "SELECT * FROM Products WHERE Name = {0}",
    myPackageName)
```

将所有 SQL 字符串放入资源文件中，并将所有 `ExecuteSqlRaw` 和 `FromSqlRaw` 调用封装在您在 `DBContext` 子类中定义的公共方法内，是一种良好的实践，这样可以保持您的 Entity Framework Core 数据层内部对特定数据库的依赖。

# 处理事务

对 `DBContext` 实例所做的所有更改都在第一次 `SaveChanges` 调用中以单个事务传递。然而，有时有必要在同一个事务中包含查询和更新。在这些情况下，我们必须显式处理事务。如果我们将几个实体 Framework Core 命令放入与事务对象关联的 `using` 块中，那么这些命令可以包含在一个事务中：

```cs
using (var dbContextTransaction = context.Database.BeginTransaction())
{
    try{
        ...
        ...
        dbContextTransaction.Commit();
    }
    catch
    {
        dbContextTransaction.Rollback();
    }
}
```

在前面的代码中，`context`是我们`DBContext`子类的一个实例。在`using`块内部，可以通过调用其`Rollback`和`Commit`方法来中止和提交事务。任何包含在事务块中的`SaveChanges`调用都将使用它们已经所在的交易，而不是创建新的交易。

# 部署你的数据层

当你的数据库层在生产或预发布环境中部署时，通常已经存在一个空数据库，因此你必须应用所有迁移以创建所有数据库对象。这可以通过调用`context.Database.Migrate()`来完成。`Migrate`方法应用尚未应用到数据库中的迁移，因此它可以在应用程序的生命周期中安全地多次调用。`context`是我们`DBContext`类的一个实例，必须通过一个具有足够权限创建表和执行我们迁移中包含的所有操作的连接字符串来传递。因此，通常，此连接字符串与我们在正常应用程序操作中使用的字符串不同。

在将 Web 应用程序部署到 Azure 的过程中，我们有机会使用我们提供的连接字符串来检查迁移。我们还可以在应用程序启动时通过调用`context.Database.Migrate()`方法手动检查迁移。这将在第十三章展示 ASP.NET Core MVC 中详细讨论，该章节专门介绍 ASP.NET MVC Web 应用程序。

对于桌面应用程序，我们可以在应用程序安装及其后续更新期间应用迁移。

在第一次应用程序安装和/或后续应用程序更新时，我们可能需要用初始数据填充一些表。对于 Web 应用程序，此操作可以在应用程序启动时执行，而对于桌面应用程序，此操作可以包含在安装过程中。

可以使用 Entity Framework Core 命令填充数据库表。首先，我们需要验证表是否为空，以避免多次添加相同的表行。这可以通过以下代码中的`Any()` LINQ 方法来完成：

```cs
if(!context.Destinations.Any())
{
    //populate here the Destinations table
}
```

让我们看看 Entity Framework Core 要分享的一些高级功能。

# 理解 Entity Framework Core 高级功能——全局过滤器

全局过滤器是在 2017 年底引入的。它们使诸如软删除和多租户表等技术成为可能，这些技术被多个用户共享，其中每个用户只需*看到*自己的记录。

全局过滤器是通过`modelBuilder`对象定义的，该对象在`DBContext.OnModelCreating`方法中可用。此方法的语法如下：

```cs
modelBuilder.Entity<MyEntity>().HasQueryFilter(m => <define filter condition here>);

```

例如，如果我们向我们的`Package`类添加一个`IsDeleted`属性，我们可以通过定义以下过滤器来对 Package 进行软删除，而不从数据库中删除它：

```cs
modelBuilder.Entity<Package>().HasQueryFilter(m => !m.IsDeleted);
```

然而，过滤器包含`DBContext`属性。因此，例如，如果我们向我们的`DBContext`子类（其值在创建`DBContext`实例时设置）添加一个`CurrentUserID`属性，那么我们可以向所有引用用户 ID 的实体添加如下过滤器：

```cs
modelBuilder.Entity<Document>().HasQueryFilter(m => m.UserId == CurrentUserId);
```

在设置上述过滤器后，当前登录的用户只能访问他们拥有的文档（具有其`UserId`的文档）。类似的技术在多租户应用程序的实现中非常有用。

# 摘要

在本章中，我们探讨了 ORM 基础知识的要点以及为什么它们如此有用。然后，我们描述了 Entity Framework Core。特别是，我们讨论了如何使用类注解和其他包含在`DBContext`子类中的声明和命令来配置数据库映射。

然后，我们讨论了如何使用迁移自动创建和更新物理数据库结构，以及如何通过 Entity Framework Core 查询和传递数据库更新。最后，我们学习了如何通过 Entity Framework Core 传递直接 SQL 命令和事务，以及如何基于 Entity Framework Core 部署数据层。

本章还回顾了最新 Entity Framework Core 版本中引入的一些高级功能。

在下一章中，我们将讨论如何使用 Entity Framework Core 与 NoSQL 数据模型结合，以及云中（特别是 Azure 中）可用的各种存储选项。

# 问题

1.  Entity Framework Core 如何适应几个不同的数据库引擎？

1.  在 Entity Framework Core 中如何声明主键？

1.  在 Entity Framework Core 中如何声明字符串字段的长度？

1.  在 Entity Framework Core 中如何声明索引？

1.  在 Entity Framework Core 中如何声明关系？

1.  两个重要的迁移命令是什么？

1.  默认情况下，LINQ 查询是否加载相关实体？

1.  是否可以在一个不是数据库实体的类实例中返回数据库数据？如果是，如何操作？

1.  迁移如何在生产环境和预览环境中应用？

# 进一步阅读

+   更多关于迁移命令的详细信息可以在[`docs.microsoft.com/en-US/ef/core/miscellaneous/cli/index`](https://docs.microsoft.com/en-US/ef/core/miscellaneous/cli/index)和其他链接中找到。

+   更多关于 Entity Framework Core 的详细信息可以在官方 Microsoft 文档中找到：[`docs.microsoft.com/en-us/ef/core/`](https://docs.microsoft.com/en-us/ef/core/).

+   可以在这里找到复杂 LINQ 查询的详尽示例：[`code.msdn.microsoft.com/101-LINQ-Samples-3fb9811b.`](https://code.msdn.microsoft.com/101-LINQ-Samples-3fb9811b)
