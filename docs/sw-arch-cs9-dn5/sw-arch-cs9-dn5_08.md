# 第八章：在 C#中与数据交互-Entity Framework Core

正如我们在*第五章*中提到的，*将微服务架构应用于企业应用程序*，软件系统被组织成层，每一层通过接口与前后层通信，这些接口不依赖于层的实现方式。当软件是一个商业/企业系统时，通常至少包含三层：数据层、业务层和表示层。一般来说，每一层提供的接口以及层的实现方式取决于应用程序。

然而，事实证明，数据层提供的功能非常标准，因为它们只是将数据从数据存储子系统映射到对象，反之亦然。这导致了以一种实质性的声明方式实现数据层的通用框架的构想。这些工具被称为**对象关系映射**（**ORM**）工具，因为它们是基于关系数据库的数据存储子系统。然而，它们也可以很好地与现代的非关系存储（如 MongoDB 和 Azure Cosmos DB）一起使用，因为它们的数据模型更接近目标对象模型，而不是纯粹的关系模型。

在本章中，我们将涵盖以下主题：

+   理解 ORM 基础知识

+   配置 Entity Framework Core

+   Entity Framework Core 迁移

+   使用 Entity Framework Core 查询和更新数据

+   部署您的数据层

+   理解 Entity Framework Core 高级功能-全局过滤器

本章描述了 ORM 以及如何配置它们，然后重点介绍了 Entity Framework Core，这是.NET 5 中包含的 ORM。

# 技术要求

本章需要免费的 Visual Studio 2019 社区版或更高版本，并安装了所有数据库工具。

本章中的所有概念都将通过基于 WWTravelClub 书籍用例的实际示例进行澄清。您可以在[`github.com/PacktPublishing/Software-Architecture-with-C-9-and-.NET-5`](https://github.com/PacktPublishing/Software-Architecture-with-C-9-and-.NET-5)找到本章的代码。

# 理解 ORM 基础知识

ORM 将关系数据库表映射为内存中的对象集合，其中对象属性对应于数据库表字段。来自 C#的类型，如布尔值、数字类型和字符串，都有对应的数据库类型。如果映射的数据库中没有 GUID，则诸如 GUID 之类的类型将映射到它们的等效字符串表示。所有日期和时间类型都映射到 C#的`DateTime`，当日期/时间不包含时区信息时，或者映射到`DateTimeOffset`，当日期/时间还包含显式时区信息时。任何数据库时间持续时间都映射到`TimeSpan`。最后，单个字符根本不应该映射到数据库字段。

由于大多数面向对象语言的字符串属性没有与之关联的长度限制（而数据库字符串字段通常有长度限制），因此在数据库映射配置中考虑了数据库限制。一般来说，当需要指定数据库类型和面向对象语言类型之间的映射时，这些选项都在映射配置中声明。

整个配置的定义方式取决于具体的 ORM。Entity Framework Core 提供了三种选项：

+   数据注释（属性注释）

+   名称约定

+   基于配置对象和方法的流畅配置接口

虽然流畅接口可以用于指定任何配置选项，但数据注释和名称约定只能用于其中的一小部分。

就个人而言，我更喜欢对大多数设置使用流畅的接口。我仅在指定具有 ID 属性名称的主键时使用名称约定，因为我发现仅依赖名称约定进行更复杂的设置也是非常危险的。实际上，名称约定上没有编译时检查，因此重新工程操作可能会错误地更改或破坏一些 ORM 设置。

我主要使用数据注释来指定属性可能值的约束，例如值的最大长度，或者属性是必填的且不能为空。实际上，这些约束限制了每个属性中指定的类型，因此将它们放在应用的属性旁边可以增加代码的可读性。

为了增加代码的可读性和可维护性，所有其他设置最好通过使用流畅的接口进行分组和组织。

每个 ORM 都适应于特定的 DB 类型（Oracle、MySQL、SQL Server 等），具有称为**提供程序**或**连接器**的特定于 DB 的适配器。Entity Framework Core 具有大多数可用 DB 引擎的提供程序。

可以在[`docs.microsoft.com/en-US/ef/core/providers/`](https://docs.microsoft.com/en-US/ef/core/providers/)找到完整的提供程序列表。

适配器对于 DB 类型的差异、事务处理方式以及 SQL 语言未标准化的所有其他特性都是必需的。

表之间的关系用对象指针表示。例如，在一对多关系中，映射到关系*一*方的类包含一个集合，该集合由关系*多*方上的相关对象填充。另一方面，映射到关系*多*方的类具有一个简单的属性，该属性由关系*一*方上的唯一相关对象填充。

整个数据库（或其中的一部分）由一个内存缓存类表示，该类包含映射到 DB 表的每个集合的属性。首先，在内存缓存类的实例上执行查询和更新操作，然后将此实例与数据库同步。

Entity Framework Core 使用的内存缓存类称为`DbContext`，它还包含映射配置。更具体地说，通过继承`DbContext`并将其添加到所有映射集合和所有必要的配置信息中，可以获得特定于应用程序的内存缓存类。

总之，`DbContext`子类实例包含与数据库同步以获取/更新实际数据的 DB 的部分快照。

使用在内存缓存类的集合上进行方法调用的查询语言执行 DB 查询。实际的 SQL 是在同步阶段创建和执行的。例如，Entity Framework Core 在映射到 DB 表的集合上执行**语言集成查询**（**LINQ**）。

一般来说，LINQ 查询会产生`IEnumerable`实例，也就是说，在查询结束时创建`IEnumerable`时，集合的元素并不会被计算，而是当您尝试从`IEnumerable`中实际检索集合元素时才会计算。这称为延迟评估或延迟执行。它的工作方式如下：

+   从`DbContext`的映射集合开始的 LINQ 查询会创建`IQueryable`的特定子类型。

+   `IQueryable`包含发出对数据库查询所需的所有信息，但是当检索到`IQueryable`的第一个元素时，实际的 SQL 才会被生成和执行。

+   通常，每个 Entity Framework 查询都以`ToList`或`ToArray`操作结束，将`IQueryable`转换为列表或数组，从而导致在数据库上实际执行查询。

+   如果查询预计只返回单个元素或根本没有元素，通常我们会执行一个`SingleOrDefault`操作，该操作返回一个元素（如果有的话）或`null`。

此外，通过在表示数据库表的`DbContext`集合属性上模拟这些操作，也可以对 DB 表执行更新、删除和添加新实体。但是，只有在通过查询加载到内存集合中后，才能以这种方式更新或删除实体。更新查询需要根据需要修改实体的内存表示，而删除查询需要从其内存映射集合中删除实体的内存表示。在 Entity Framework Core 中，通过调用集合的`Remove(entity)`方法执行删除操作。

添加新实体没有进一步的要求。只需将新实体添加到内存集合中即可。对各种内存集合进行的更新、删除和添加实际上是通过显式调用 DB 同步方法传递到数据库的。

例如，当您调用`DbContext.SaveChanges()`方法时，Entity Framework Core 会将在`DbContext`实例上执行的所有更改传递到数据库。

在同步操作期间传递到数据库的更改是在单个事务中执行的。此外，对于具有事务的显式表示的 ORM（如 Entity Framework Core），同步操作是在事务范围内执行的，因为它使用该事务而不是创建新事务。

本章的其余部分将解释如何使用 Entity Framework Core，以及基于本书的 WWTravelClub 用例的一些示例代码。

# 配置 Entity Framework Core

由于数据库处理被限制在专用应用程序层中，因此最好的做法是在一个单独的库中定义您的 Entity Framework Core（`DbContext`）。因此，我们需要定义一个.NET Core 类库项目。正如我们在*第二章*的*书籍用例-理解.NET Core 项目的主要类型*部分中讨论的那样，我们有两种不同类型的库项目：**.NET Standard**和**.NET (Core)**。

虽然.NET Core 库与特定的.NET Core 版本相关联，但.NET Standard 2.0 库具有广泛的应用范围，因为它们可以与大于 2.0 的任何.NET 版本以及经典的.NET Framework 4.7.2 及以上版本一起使用。

然而，`Microsoft.EntityFrameworkCore`包的第 5 版，也就是随.NET 5 一起发布的版本，仅依赖于.NET Standard 2.1。这意味着它不是设计用于特定的.NET（Core）版本，而是只需要支持.NET Standard 2.1 的.NET Core 版本。因此，Entity Framework 5 可以与.NET 5 以及高于或等于 2.1 的任何.NET Core 版本正常工作。

由于我们的库不是通用库（它只是特定.NET 5 应用程序的一个组件），所以我们可以选择.NET 5 库而不是选择.NET Standard 库项目。我们的.NET 5 库项目可以按以下方式创建和准备：

1.  打开 Visual Studio 并定义一个名为`WWTravelClubDB`的新解决方案，然后选择可用的最新.NET Core 版本的**类库（.NET Core）**。

1.  我们必须安装所有与 Entity Framework Core 相关的依赖项。安装所有必要的依赖项的最简单方法是添加我们将要使用的数据库引擎提供程序的 NuGet 包 - 在我们的情况下是 SQL Server - 正如我们在*第四章*，*决定最佳基于云的解决方案*中提到的。实际上，任何提供程序都将安装所有所需的包，因为它们都作为依赖项。因此，让我们添加最新稳定版本的`Microsoft.EntityFrameworkCore.SqlServer`。如果您计划使用多个数据库引擎，还可以添加其他提供程序，因为它们可以并存。在本章的后面，我们将安装其他包含我们需要处理 Entity Framework Core 的工具的 NuGet 包。然后，我们将解释如何安装进一步需要处理 Entity Framework Core 配置的工具。

1.  让我们将默认的`Class1`类重命名为`MainDbContext`。这是自动添加到类库中的。

1.  现在，让我们用以下代码替换其内容：

```cs
using System;
using Microsoft.EntityFrameworkCore;
namespace WWTravelClubDB
{
    public class MainDbContext: DbContext
    {
        public MainDbContext(DbContextOptions options)
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

1.  我们继承自`DbContext`，并且需要将`DbContextOptions`传递给`DbContext`构造函数。`DbContextOptions`包含创建选项，如数据库连接字符串，这取决于目标数据库引擎。

1.  所有映射到数据库表的集合将作为`MainDbContext`的属性添加。映射配置将在重写的`OnModelCreating`方法中使用传递的`ModelBuilder`对象来定义。

下一步是创建表示所有数据库表行的所有类。这些称为**实体**。我们需要为要映射的每个数据库表创建一个实体类。让我们在项目根目录下创建一个`Models`文件夹。下一小节将解释如何定义所有所需的实体。

## 定义数据库实体

数据库设计，就像整个应用程序设计一样，是按迭代进行的。假设在第一次迭代中，我们需要一个包含两个数据库表的原型：一个用于所有旅行套餐，另一个用于所有套餐引用的位置。每个套餐只涵盖一个位置，而单个位置可能被多个套餐涵盖，因此这两个表通过一对多的关系相连。

因此，让我们从位置数据库表开始。正如我们在上一节末尾提到的，我们需要一个实体类来表示这个表的行。让我们称实体类为`Destination`：

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

所有数据库字段必须由可读/写的 C#属性表示。假设每个目的地都类似于一个城镇或地区，可以通过其名称和所在国家来定义，并且所有相关信息都包含在其`Description`中。在将来的迭代中，我们可能会添加几个字段。`Id`是自动生成的键。

然而，现在，我们需要添加关于如何将所有字段映射到数据库字段的信息。在 Entity Framework Core 中，所有基本类型都会自动映射到数据库类型，由所使用的数据库引擎特定提供程序（在我们的情况下是 SQL Server 提供程序）。

我们唯一的担忧是：

+   字符串的长度限制：可以通过为每个字符串属性应用适当的`MaxLength`和`MinLength`属性来考虑。所有对实体配置有用的属性都包含在`System.ComponentModel.DataAnnotations`和`System.ComponentModel.DataAnnotations.Schema`命名空间中。因此，最好将它们都添加到所有实体定义中。

+   **指定哪些字段是必填的，哪些是可选的**：如果项目没有使用新的可空引用类型功能，默认情况下，所有引用类型（例如所有字符串）都被假定为可选的，而所有值类型（例如数字和 GUID）都被假定为必填的。如果我们希望引用类型是必填的，那么我们必须用`Required`属性进行修饰。另一方面，如果我们希望`T`类型的属性是可选的，并且`T`是值类型或者可空引用类型功能已经开启，那么我们必须用`T?`替换`T`。

+   **指定哪个属性代表主键**：可以通过用`Key`属性修饰属性来指定主键。然而，如果没有找到`Key`属性，那么名为`Id`的属性（如果有的话）将被视为主键。在我们的情况下，不需要`Key`属性。

由于每个目的地都在一对多关系的*一*侧，它必须包含一个与相关包实体相关的集合；否则，我们将无法在 LINQ 查询的子句中引用相关实体。

将所有内容放在一起，`Destination`类的最终版本如下：

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

由于`Description`属性没有长度限制，它将以 SQL Server `nvarchar(MAX)`字段的无限长度实现。我们可以以类似的方式编写`Package`类的代码：

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
        public int DurationInDays { get; set; }
        public DateTime? StartValidityDate { get; set; }
        public DateTime? EndValidityDate { get; set; }
        public Destination MyDestination { get; set; }
        public int DestinationId { get; set; }
    }
} 
```

每个包都有一个持续时间（以天为单位），以及可选的开始和结束日期，其中包的优惠有效。`MyDestination`将包与它们与`Destination`实体的多对一关系连接起来，而`DestinationId`是同一关系的外键。

虽然不是必须指定外键，但这是一个好习惯，因为这是唯一指定关系的一些属性的方法。例如，在我们的情况下，由于`DestinationId`是一个`int`（值类型），它是必填的。因此，这里的关系是一对多，而不是（0,1）-对多。将`DestinationId`定义为`int?`，而不是`int`，会将一对多关系转变为（0,1）-对多关系。此外，正如我们将在本章后面看到的那样，有一个外键的显式表示大大简化了更新操作和一些查询。

在下一节中，我们将解释如何定义表示数据库表的内存集合。

## 定义映射集合

一旦我们定义了所有的实体，它们就是数据库行的面向对象表示，我们需要定义表示数据库表本身的内存集合。正如我们在*理解 ORM 基础*部分中提到的，所有数据库操作都映射到这些集合上的操作（本章的*使用 Entity Framework Core 查询和更新数据*部分将解释如何）。对于每个实体`T`，只需在我们的`DbContext`中添加一个`DbSet<T>`集合属性即可。通常，每个属性的名称是通过将实体名称变为复数形式得到的。因此，我们需要将以下两个属性添加到我们的`MainDbContext`中：

```cs
public DbSet<Package> Packages { get; set; }
public DbSet<Destination> Destinations { get; set; } 
```

到目前为止，我们已经将数据库内容翻译成属性、类和数据注释。然而，Entity Framework 需要更多信息来与数据库交互。下一小节将解释如何提供这些信息。

## 完成映射配置

我们无法在实体定义中指定的映射配置信息必须在`OnModelCreating DbContext`方法中添加。每个与实体`T`相关的配置信息都以`builder.Entity<T>()`开头，并继续调用指定该约束类型的方法。进一步嵌套调用指定约束的更多属性。例如，我们的一对多关系可以配置如下：

```cs
builder.Entity<Destination>()
    .HasMany(m => m.Packages)
    .WithOne(m => m.MyDestination)
    .HasForeignKey(m => m.DestinationId)
    .OnDelete(DeleteBehavior.Cascade); 
```

关系的两侧是通过我们添加到实体的导航属性来指定的。`HasForeignKey`指定外部键。最后，`OnDelete`指定了在删除目标时要执行的操作。在我们的情况下，它执行了与该目的地相关的所有包的级联删除。

可以通过从关系的另一侧开始定义相同的配置，也就是从`builder.Entity<Package>()`开始：

```cs
builder.Entity<Package>()
    .HasOne(m => m.MyDestination)
    .WithMany(m => m.Packages)
    .HasForeignKey(m => m.DestinationId)
    .OnDelete(DeleteBehavior.Cascade); 
```

唯一的区别是前面语句的`HasMany-WithOne`方法被`HasOne-WithMany`方法替换，因为我们是从关系的另一侧开始的。在这里，我们还可以选择每个小数属性在其映射的数据库字段中表示的精度。默认情况下，小数由 18 位和 2 位小数表示。您可以使用类似以下内容为每个属性更改此设置：

```cs
...
.Property(m => m.Price)
        .HasPrecision(10, 3); 
```

`ModelBuilder builder`对象允许我们使用以下内容指定数据库索引：

```cs
builder.Entity<T>()
   .HasIndex(m => m.PropertyName); 
```

多属性索引定义如下：

```cs
builder.Entity<T>()
    .HasIndex("propertyName1", "propertyName2", ...); 
```

从版本 5 开始，索引也可以通过应用于类的属性来定义。以下是单属性索引的情况：

```cs
[Index(nameof(Property), IsUnique = true)]
public class MyClass
{
    public int Id { get; set; }
    [MaxLength(128)]
    public string Property { get; set; }
} 
```

以下是多属性索引的情况：

```cs
[Index(nameof(Property1), nameof(Property2), IsUnique = false)]
public class MyComplexIndexClass
{
    public int Id { get; set; }
    [MaxLength(64)]
    public string Property1 { get; set; }
    [MaxLength(64)]
    public string Property2 { get; set; }
} 
```

如果我们添加了所有必要的配置信息，那么我们的`OnModelCreating`方法将如下所示：

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
        .HasIndex(nameof(Package.StartValidityDate),
                  nameof(Package.EndValidityDate));
} 
```

前面的示例展示了一对多关系，但 Entity Framework Core 5 也支持多对多关系：

```cs
 modelBuilder
        .Entity<Teacher>()
        .HasMany(e => e.Classrooms)
        .WithMany(e => e.Teachers) 
```

在前面的情况下，联接实体和数据库联接表是自动创建的，但您也可以指定现有实体作为联接实体。在前面的示例中，联接实体可能是老师在每个教室教授的课程：

```cs
modelBuilder
  Entity<Teacher>()
  .HasMany(e => e.Classrooms)
  .WithMany(e => e.Teachers)
      .UsingEntity<Course>(
           b => b.HasOne(e => e.Teacher).WithMany()
           .HasForeignKey(e => e.TeacherId),
           b => b.HasOne(e => e.Classroom).WithMany()
           .HasForeignKey(e => e.ClassroomId)); 
```

一旦配置了 Entity Framework Core，我们可以使用所有配置信息来创建实际的数据库，并在应用程序发展过程中放置所有需要的工具，以便更新数据库的结构。下一节将解释如何进行。

# Entity Framework Core 迁移

现在我们已经配置了 Entity Framework 并定义了特定于应用程序的`DbContext`子类，我们可以使用 Entity Framework Core 设计工具来生成物理数据库，并创建 Entity Framework Core 与数据库交互所需的数据库结构快照。

每个需要它们的项目中必须安装 Entity Framework Core 设计工具作为 NuGet 包。有两个等效的选项：

+   **适用于任何 Windows 控制台的工具**：这些工具通过`Microsoft.EntityFrameworkCore.Design` NuGet 包提供。所有 Entity Framework Core 命令都以`dotnet ef .....`格式，因为它们包含在`ef`命令行的.NET Core 应用程序中。

+   **专门用于 Visual Studio Package Manager 控制台的工具**：这些工具包含在`Microsoft.EntityFrameworkCore.Tools` NuGet 包中。它们不需要`dotnet ef`前缀，因为它们只能从 Visual Studio 内的**Package Manager Console**中启动。

Entity Framework Core 的设计工具在设计/更新过程中使用。该过程如下：

1.  根据需要修改`DbContext`和实体的定义。

1.  我们启动设计工具，要求 Entity Framework Core 检测和处理我们所做的所有更改。

1.  一旦启动，设计工具将更新数据库结构快照并生成一个新的*迁移*，即一个包含我们需要的所有指令的文件，以便修改物理数据库以反映我们所做的所有更改。

1.  我们启动另一个工具来使用新创建的迁移更新数据库。

1.  我们测试新配置的 DB 层，如果需要新的更改，我们回到*步骤 1*。

1.  当数据层准备就绪时，它被部署到暂存或生产环境中，所有迁移再次应用到实际的暂存/生产数据库。

这在各种软件项目迭代和应用程序的生命周期中会重复多次。

如果我们操作的是已经存在的数据库，我们需要配置`DbContext`及其模型，以反映我们想要映射的所有表的现有结构。然后，如果我们想要开始使用迁移而不是继续进行直接的数据库更改，我们可以调用设计工具，并使用`IgnoreChanges`选项，以便它们生成一个空迁移。此外，这个空迁移必须传递给物理数据库，以便它可以将与物理数据库关联的数据库结构版本与数据库快照中记录的版本进行同步。这个版本很重要，因为它决定了哪些迁移必须应用到数据库，哪些已经应用了。

整个设计过程需要一个测试/设计数据库，如果我们操作的是已经存在的数据库，那么这个测试/设计数据库的结构必须反映实际数据库的结构 - 至少在我们想要映射的表方面。为了使设计工具能够与数据库交互，我们必须定义它们传递给`DbContext`构造函数的`DbContextOptions`选项。这些选项在设计时很重要，因为它们包含测试/设计数据库的连接字符串。如果我们创建一个实现`IDesignTimeDbContextFactory<T>`接口的类，其中`T`是我们的`DbContext`子类，设计工具可以了解我们的`DbContextOptions`选项：

```cs
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Design;
namespace WWTravelClubDB
{
    public class LibraryDesignTimeDbContextFactory
        : IDesignTimeDbContextFactory<MainDbContext>
    {
        private const string connectionString =
            @"Server=(localdb)\mssqllocaldb;Database=wwtravelclub;
                Trusted_Connection=True;MultipleActiveResultSets=true";
        public MainDbContext CreateDbContext(params string[] args)
        {
            var builder = new DbContextOptionsBuilder<MainDbContext>();

            builder.UseSqlServer(connectionString);
            return new MainDbContext(builder.Options);
        }
    }
} 
```

`connectionString`将被 Entity Framework 用于在开发机器上安装的本地 SQL Server 实例中创建一个新数据库，并使用 Windows 凭据进行连接。您可以自由更改它以反映您的需求。

现在，我们准备创建我们的第一个迁移！让我们开始吧：

1.  让我们转到**程序包管理器控制台**，确保**WWTravelClubDB**被选为我们的默认项目。

1.  现在，输入`Add-Migration initial`并按 Enter 键发出此命令。在发出此命令之前，请验证是否已添加了`Microsoft.EntityFrameworkCore.Tools` NuGet 包，否则可能会出现“未识别的命令”错误！[](img/B16756_08_01.png)

图 8.1：添加第一个迁移

`initial`是我们给第一个迁移的名称。因此，一般来说，命令是`Add-Migration <迁移名称>`。当我们操作现有数据库时，必须在第一个迁移（仅在第一个迁移）中添加`-IgnoreChanges`选项，以便创建一个空迁移。有关整套命令的参考可以在*进一步阅读*部分找到。

1.  如果在创建迁移之后，但在将迁移应用到数据库之前，我们意识到我们犯了一些错误，我们可以使用`Remove-Migration`命令撤消我们的操作。如果迁移已经应用到数据库，纠正错误的最简单方法是对代码进行所有必要的更改，然后应用另一个迁移。

1.  一旦执行`Add-Migration`命令，我们的项目中会出现一个新文件夹！[](img/B16756_08_02.png)

图 8.2：Add-Migration 命令创建的文件

`20201008150827_initial.cs`是我们用易于理解的语言表达的迁移。

您可以查看代码以验证一切是否正常，您也可以修改迁移内容（只有当您足够专业时才能可靠地这样做）。每个迁移都包含一个`Up`方法和一个`Down`方法。`Up`方法表示迁移，而`Down`方法撤消其更改。因此，`Down`方法包含与`Up`方法中包含的所有操作的相反操作，按照相反的顺序。

`20201008150827_initial.Designer.cs`是 Visual Studio 的设计器代码，*不得*修改，而`MainDBContextModelSnapshot.cs`是整体数据库结构快照。如果添加了进一步的迁移，新的迁移文件及其设计器对应文件将出现，并且唯一的`MainDBContextModelSnapshot.cs`数据库结构快照将被更新以反映数据库的整体结构。

在 Windows 控制台中输入`dotnet ef migrations add initial`可以发出相同的命令。但是，此命令必须在项目的根文件夹中发出（而不是在解决方案的根文件夹中）。

可以通过在包管理器控制台中键入`Update-Database`来将迁移应用到数据库。相应的 Windows 控制台命令是`dotnet ef database update`。让我们尝试使用这个命令来创建物理数据库！

下一小节将解释如何创建 Entity Framework 无法自动创建的数据库内容。之后，在下一节中，我们将使用 Entity Framework 的配置和我们使用`dotnet ef database update`生成的数据库来创建、查询和更新数据。

## 理解存储过程和直接 SQL 命令

一些数据库结构，例如存储过程，无法通过我们之前描述的 Entity Framework Core 命令和声明自动生成。例如，可以通过`migrationBuilder.Sql("<sql scommand>")`方法在`Up`和`Down`方法中手动包含存储过程和通用 SQL 字符串。

最安全的方法是添加一个迁移而不进行任何配置更改，以便在创建时迁移为空。然后，我们可以将必要的 SQL 命令添加到此迁移的空`Up`方法中，以及在空的`Down`方法中添加它们的相反命令。将所有 SQL 字符串放在资源文件（`.resx`文件）的属性中是一个好的做法。

现在，您已经准备好通过 Entity Framework Core 与数据库进行交互了。

# 使用 Entity Framework Core 查询和更新数据

为了测试我们的 DB 层，我们需要根据与我们的库相同的.NET Core 版本向解决方案中添加一个基于控制台的项目。让我们开始吧：

1.  让我们将新的控制台项目命名为`WWTravelClubDBTest`。

1.  现在，我们需要将数据层作为控制台项目的依赖项添加到**References**节点中，然后选择**Add reference**。

1.  删除`program.cs`文件中`Main`静态方法的内容，并开始编写以下内容：

```cs
Console.WriteLine("program start: populate database, press a key to continue");
Console.ReadKey(); 
```

1.  然后，在文件顶部添加以下命名空间：

```cs
using WWTravelClubDB;
using WWTravelClubDB.Models;
using Microsoft.EntityFrameworkCore;
using System.Linq; 
```

现在，我们已经完成了准备测试项目的工作，可以尝试查询和更新数据。让我们开始创建一些数据库对象，即一些目的地和包。按照以下步骤进行：

1.  首先，我们必须创建一个适当的连接字符串的`DbContext`子类的实例。我们可以使用相同的`LibraryDesignTimeDbContextFactory`类，该类被设计工具用于获取它：

```cs
var context = new LibraryDesignTimeDbContextFactory()
    .CreateDbContext(); 
```

1.  可以通过简单地将类实例添加到我们`DbContext`子类的映射集合中来创建新行。如果`Destination`实例与其关联的包相关联，我们可以简单地将它们添加到其`Packages`属性中：

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
            DurationInDays=7,
            Price=1000
        },
        new Package
        {
            Name = "Winter in Florence",
            StartValidityDate = new DateTime(2019, 12, 1),
            EndValidityDate = new DateTime(2020, 2, 1),
            DurationInDays=7,
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

无需指定主键，因为它们是自动生成的，并将由数据库填充。事实上，在`SaveChanges()`操作后，我们的上下文与实际数据库同步后，`firstDestination.Id`属性具有非零值。对于`Package`的主键也是如此。

当我们声明一个实体（在我们的情况下是`Package`）是另一个实体（在我们的情况下是`Destination`）的子实体，通过将其插入到父实体集合（在我们的情况下是`Packages`集合）中时，由于 Entity Framework Core 会自动推断外键（在我们的情况下是`DestinationId`），因此无需显式设置外键。创建并与`firstDestination`数据库同步后，我们可以以两种不同的方式添加更多的套餐：

+   创建一个`Package`类实例，将其`DestinationId`外键设置为`firstDestinatination.Id`，并将其添加到`context.Packages`

+   创建一个`Package`类实例，无需设置其外键，然后将其添加到其父`Destination`实例的`Packages`集合中。

后一种选项是唯一的可能性，当子实体（`Package`）与其父实体（`Destination`）一起添加，并且父实体具有自动生成的主键时，因为在这种情况下，外键在执行添加时不可用。在大多数其他情况下，前一种选项更简单，因为第二种选项要求在内存中加载父`Destination`实体，以及其`Packages`集合，即与`Destination`对象相关联的所有套餐（默认情况下，连接的实体不会通过查询加载）。

现在，假设我们想修改*佛罗伦萨*目的地，并为所有`佛罗伦萨`套餐价格增加 10%。我们该如何操作？按照以下步骤找出答案：

1.  首先，注释掉所有以前用于填充数据库的指令，但保留`DbContext`创建指令。

1.  然后，我们需要使用查询将实体加载到内存中，修改它，并调用`SaveChanges()`将我们的更改与数据库同步。

如果我们只想修改其描述，那么以下查询就足够了：

```cs
var toModify = context.Destinations
    .Where(m => m.Name == "Florence").FirstOrDefault(); 
```

1.  我们需要加载所有相关的目的地套餐，这些套餐默认情况下未加载。可以使用`Include`子句来完成，如下所示：

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

如果使用`Include`方法包含的实体本身包含我们想要包含的嵌套集合，我们可以使用`ThenInclude`，如下所示：

```cs
.Include(m => m.NestedCollection)
.ThenInclude(m => m.NestedNestedCollection) 
```

由于 Entity Framework 始终尝试将每个 LINQ 翻译为单个 SQL 查询，有时生成的查询可能过于复杂和缓慢。在这种情况下，从第 5 版开始，我们可以允许 Entity Framework 将 LinQ 查询拆分为多个 SQL 查询，如下所示：

```cs
.AsSplitQuery().Include(m => m.NestedCollection)
.ThenInclude(m => m.NestedNestedCollection) 
```

通过检查`ToQueryString`方法生成的 LinQ 查询的 SQL，可以解决性能问题：

```cs
var mySQL = myLinQQuery.ToQueryString (); 
```

从第 5 版开始，包含的嵌套集合也可以使用`Where`进行过滤，如下所示：

```cs
.Include(m => m.Packages.Where(l-> l.Price < x)) 
```

到目前为止，我们执行的查询的唯一目的是更新检索到的实体。接下来，我们将解释如何检索将向用户显示和/或由复杂业务操作使用的信息。

## 将数据返回到表示层

为了保持层之间的分离，并根据每个*用例*实际需要的数据调整查询，DB 实体不会按原样发送到表示层。相反，数据将投影到包含*用例*所需信息的较小类中，这些类由表示层的调用方法实现。将数据从一层移动到另一层的对象称为**数据传输对象**（**DTOs**）。例如，让我们创建一个 DTO，其中包含在向用户返回套餐列表时值得显示的摘要信息（我们假设如果需要，用户可以通过单击他们感兴趣的套餐来获取更多详细信息）：

1.  让我们在 WWTravelClubDBTest 项目中添加一个 DTO，其中包含需要在套餐列表中显示的所有信息：

```cs
namespace WWTravelClubDBTest
{
    public class PackagesListDTO
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
        public int DurationInDays { get; set; }
        public DateTime? StartValidityDate { get; set; }
        public DateTime? EndValidityDate { get; set; }
        public string DestinationName { get; set; }
        public int DestinationId { get; set; }
        public override string ToString()
        {
            return string.Format("{0}. {1} days in {2}, price: 
            {3}", Name, DurationInDays, DestinationName, Price);
        }
    }
} 
```

我们不需要将实体加载到内存中，然后将其数据复制到 DTO 中，而是可以直接将数据库数据投影到 DTO 中，这要归功于 LINQ 的`Select`子句。这样可以最大程度地减少与数据库交换的数据量。

1.  例如，我们可以使用查询填充我们的 DTO，该查询检查所有在 8 月 10 日左右可用的包裹：

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
        DurationInDays=m.DurationInDays,
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

1.  在`Select`子句中，我们还可以导航到任何相关实体以获取所需的数据。例如，前面的查询导航到相关的`Destination`实体以获取`Package`目的地名称。

1.  程序在每个`Console.ReadKey()`方法处停止，等待您按任意键。这样，您就有时间分析由我们添加到`Main`方法的所有代码片段产生的输出。

1.  现在，在解决方案资源管理器中右键单击 WWTravelClubDBTest 项目，并将其设置为启动项目。然后，运行解决方案。

现在，我们将学习如何处理不能有效映射到表示数据库表的内存集合中的即时操作的操作。

## 发出直接的 SQL 命令

并非所有的数据库操作都可以通过使用 LINQ 查询数据库并更新内存实体来高效执行。例如，计数器增量可以通过单个 SQL 指令更有效地执行。此外，如果我们定义了适当的存储过程/SQL 命令，一些操作可以以可接受的性能执行。在这些情况下，我们不得不直接向数据库发出 SQL 命令或从 Entity Framework 代码中调用数据库存储过程。有两种可能性：执行数据库操作但不返回实体的 SQL 语句，以及返回实体的 SQL 语句。

不返回实体的 SQL 命令可以通过`DbContext`方法执行，如下所示：

```cs
int DbContext.Database.ExecuteSqlRaw(string sql, params object[] parameters) 
```

参数可以在字符串中作为`{0}，{1}，...，{n}`进行引用。每个`{m}`都填充了`parameters`数组中`m`索引处包含的对象，该对象从.NET 类型转换为相应的 SQL 类型。该方法返回受影响的行数。

必须通过与这些实体相关联的映射集合的`FromSqlRaw`方法发出返回实体集合的 SQL 命令：

```cs
context.<mapped collection>.FromSqlRaw(string sql, params object[] parameters) 
```

因此，例如，返回`Package`实例的命令看起来像这样：

```cs
var results = context.Packages.FromSqlRaw("<some sql>", par1, par2, ...).ToList(); 
```

SQL 字符串和参数在`ExecuteSqlRaw`方法中的工作方式如下。以下是一个简单的例子：

```cs
var allPackages =context.Packages.FromSqlRaw(
    "SELECT * FROM Products WHERE Name = {0}",
    myPackageName) 
```

将所有 SQL 字符串放入资源文件中，并将所有`ExecuteSqlRaw`和`FromSqlRaw`调用封装在您在基于 Entity Framework Core 的数据层中定义的公共方法中，以便将特定数据库的依赖性保持在内部。

## 处理事务

对`DbContext`实例所做的所有更改都在第一次`SaveChanges`调用时作为单个事务传递。然而，有时需要在同一个事务中包含查询和更新。在这些情况下，我们必须显式处理事务。如果我们将它们放在与事务对象关联的`using`块中，那么几个 Entity Framework Core 命令可以包含在事务中：

```cs
using (var dbContextTransaction = context.Database.BeginTransaction())
try{
   ...
   ...
   dbContextTransaction.Commit();
 }
 catch
 {
   dbContextTransaction.Rollback();
 } 
```

在上述代码中，`context`是我们`DbContext`子类的一个实例。在`using`块内，可以通过调用其`Rollback`和`Commit`方法来中止和提交事务。包含在事务块中的任何`SaveChanges`调用都使用它们已经存在的事务，而不是创建新的事务。

# 部署数据层

当数据库层部署到生产环境或暂存环境时，通常已经存在一个空数据库，因此必须应用所有迁移以创建所有数据库对象。这可以通过调用`context.Database.Migrate()`来完成。`Migrate`方法应用尚未应用到数据库的迁移，因此在应用程序的生命周期中可以安全地多次调用。`context`是我们的`DbContext`类的一个实例，必须通过具有足够权限来创建表和执行迁移中包含的所有操作的连接字符串进行传递。因此，通常，此连接字符串与我们在正常应用程序操作期间使用的字符串不同。

在 Azure 上部署 Web 应用程序时，我们有机会使用我们提供的连接字符串来检查迁移。我们还可以在应用程序启动时通过调用`context.Database.Migrate()`方法来手动检查迁移。这将在*第十五章*“介绍 ASP.NET Core MVC”中详细讨论，该章节专门讨论 ASP.NET MVC Web 应用程序。

对于桌面应用程序，我们可以在应用程序安装和后续更新期间应用迁移。

在首次安装应用程序和/或后续应用程序更新时，我们可能需要使用初始数据填充一些表。对于 Web 应用程序，此操作可以在应用程序启动时执行，而对于桌面应用程序，此操作可以包含在安装中。

数据库表可以使用 Entity Framework Core 命令进行填充。但首先，我们需要验证表是否为空，以避免多次添加相同的表行。这可以使用`Any()` LINQ 方法来完成，如下面的代码所示：

```cs
if(!context.Destinations.Any())
{
    //populate here the Destinations table
} 
```

让我们来看看 Entity Framework Core 有哪些高级特性可以分享。

# 理解 Entity Framework Core 的高级特性

值得一提的一个有趣的 Entity Framework 高级特性是全局过滤器，这是在 2017 年底引入的。它们可以实现软删除和多租户表等技术，这些表由多个用户共享，每个用户只能*看到*自己的记录。

全局过滤器是使用`modelBuilder`对象定义的，该对象在`DbContext`的`OnModelCreating`方法中可用。此方法的语法如下：

```cs
modelBuilder.Entity<MyEntity>().HasQueryFilter(m => <define filter condition here>); 
```

例如，如果我们向我们的`Package`类添加一个`IsDeleted`属性，我们可以通过定义以下过滤器软删除`Package`而不从数据库中删除它：

```cs
modelBuilder.Entity<Package>().HasQueryFilter(m => !m.IsDeleted); 
```

但是，过滤器包含`DbContext`属性。因此，例如，如果我们向我们的`DbContext`子类添加一个`CurrentUserID`属性（其值在创建`DbContext`实例时设置），那么我们可以向所有引用用户 ID 的实体添加以下过滤器：

```cs
modelBuilder.Entity<Document>().HasQueryFilter(m => m.UserId == CurrentUserId); 
```

通过上述过滤器，当前登录的用户只能访问他们拥有的文档（具有他们的`UserId`的文档）。类似的技术在多租户应用程序的实现中非常有用。

另一个值得一提的有趣特性是将实体映射到不可更新的数据库查询，这是在版本 5 中引入的。

当您定义一个实体时，您可以明确定义映射的数据库表的名称或映射的可更新视图的名称：

```cs
 modelBuilder.Entity<MyEntity1>().ToTable("MyTable");
 modelBuilder.Entity<MyEntity2>().ToView("MyView"); 
```

当实体映射到视图时，数据库迁移不会生成表，因此必须由开发人员手动定义数据库视图。

如果我们想要映射实体的视图不可更新，LinQ 无法使用它将更新传递给数据库。在这种情况下，我们可以同时将相同实体映射到视图和表：

```cs
modelBuilder.Entity<MyEntity>().ToTable("MyTable").ToView("MyView"); 
```

Entity Framework 将使用视图进行查询和表进行更新。当我们创建数据库表的新版本，但又希望在所有查询中同时从旧版本的表中获取数据时，这是非常有用的。在这种情况下，我们可以定义一个视图，该视图从旧表和新表中获取数据，但只在新表上传递所有更新。

# 摘要

在本章中，我们讨论了 ORM 基础知识的基本要点以及它们为何如此有用。然后，我们描述了 Entity Framework Core。特别是，我们讨论了如何使用类注释和其他声明以及包含在`DbContext`子类中的命令来配置数据库映射。

然后，我们讨论了如何通过迁移自动创建和更新物理数据库结构，以及如何通过 Entity Framework Core 查询和传递更新到数据库。最后，我们学习了如何通过 Entity Framework Core 传递直接的 SQL 命令和事务，以及如何基于 Entity Framework Core 部署数据层。

本章还回顾了最新的 Entity Framework Core 版本中引入的一些高级功能。

在下一章中，我们将讨论 Entity Framework Core 如何与 NoSQL 数据模型一起使用，以及云中和特别是 Azure 中可用的各种存储选项。

# 问题

1.  Entity Framework Core 如何适应多种不同的数据库引擎？

1.  Entity Framework Core 中如何声明主键？

1.  Entity Framework Core 中如何声明字符串字段的长度？

1.  Entity Framework Core 中如何声明索引？

1.  Entity Framework Core 中如何声明关系？

1.  什么是两个重要的迁移命令？

1.  默认情况下，LINQ 查询是否加载相关实体？

1.  是否可能在不是数据库实体的类实例中返回数据库数据？如果是，如何？

1.  在生产和分段中如何应用迁移？

# 进一步阅读

+   有关迁移命令的更多详细信息，请参阅[`docs.microsoft.com/en-US/ef/core/miscellaneous/cli/index`](https://docs.microsoft.com/en-US/ef/core/miscellaneous/cli/index)以及其中包含的其他链接。

+   有关 Entity Framework Core 的更多详细信息，请参阅官方 Microsoft 文档：[`docs.microsoft.com/en-us/ef/core/`](https://docs.microsoft.com/en-us/ef/core/)。

+   这里可以找到一组复杂 LINQ 查询的详尽示例：[`code.msdn.microsoft.com/101-LINQ-Samples-3fb9811b`](https://code.msdn.microsoft.com/101-LINQ-Samples-3fb9811b)。
