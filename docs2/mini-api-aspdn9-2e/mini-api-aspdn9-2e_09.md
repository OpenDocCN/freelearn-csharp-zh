

# 使用 Entity Framework Core 和 Dapper 进行对象关系映射

在上一章中，我们使用了直接连接到 SQL 和 NoSQL 数据库来创建和检索数据，这是大多数 API 的基本功能。

实际上，大量基于.NET 的 API 倾向于使用**对象关系映射**（**ORM**）与数据库进行交互，而不是直接连接方法。这是因为 ORM 在底层数据之上提供了另一层抽象，促进了 SOLID 设计原则，同时有利于可扩展性和易于长期维护。

在本章中，我们将探讨两个主流 ORM 框架——Entity Framework Core 和 Dapper。使用这两种技术，我们将能够映射数据库中的实体，并将它们作为如果数据包含在我们的最小 API 项目中的类来管理。

我们将涵盖以下主要内容：

+   ORMs 简介

+   在最小 API 项目中配置 Dapper

+   使用 Dapper 执行 CRUD 操作

+   在最小 API 项目中配置 Entity Framework

+   使用 Entity Framework 执行 CRUD 操作

# 技术要求

您需要在您的机器上安装以下软件：

+   Visual Studio 2022 或 Visual Studio Code

+   Microsoft SQL Server 2022 开发者版

+   Microsoft SQL Server Management Studio

您需要在 SQL Server 中创建一个数据库（**MyCompany**）。创建所需**Employees**表的 SQL 语句在上一章中已提供，但我也将包括在本章中。

本章的代码可在 GitHub 仓库中找到：[`github.com/PacktPublishing/Minimal-APIs-in-ASP.NET-9`](https://github.com/PacktPublishing/Minimal-APIs-in-ASP.NET-9)。

# ORMs 简介

ORMs 首次在 20 世纪 90 年代推出，旨在解决关系数据库（如 SQL）中数据建模方式与面向对象编程（**OOP**）语言之间的不匹配，通常被称为**阻抗不匹配**。

关系型数据库中的数据以一系列表的形式排列，每个表都有多个列定义每条记录的属性，这些属性反过来由表中的一行表示。关系型数据库中实体之间的关系由**外键**和查询期间发生的**连接**操作表示。

相比之下，在面向对象的语言中，数据以具有字段、属性和操作逻辑（如方法和函数，可以作用于数据）的对象形式表示。面向对象语言中对象之间的关系更为抽象，通过指针引用和继承、多态等概念表示。

ORM 通过提供一种映射数据的方法来弥合这两种范式之间的差距，使我们能够将数据库记录作为对象来处理。这提供了一层抽象，简化了 SQL 的复杂性，使得执行**创建**、**读取**、**更新**、**删除**（**CRUD**）操作变得更加容易。

对于各种面向对象的编程语言和框架，有许多广泛使用的 ORM 可供选择。第一个广泛使用的 ORM 是**TopLink**。它在 1994 年开发，旨在为 Java 应用程序提供映射，影响了今天我们视为理所当然的许多 ORM 技术，包括我们将在本章中探讨的两个——Dapper 和 Entity Framework。

ORM 是任何项目的显著加速器，尤其是对最小 API 项目。它们减少了 ASP.NET 中的样板代码，因为它们可以作为包轻松安装，并且配置可以集中完成，查询和命令比使用**SqlConnection**和**SqlCommand**等类的直接 SQL 连接需要更少的仪式。

ORM 最强大的方面之一是它们能够管理底层数据库的模式。当然，对象可以从现有数据库映射，但 ORM 提供了以与代码中配置类相同的方式管理数据库结构的能力。这使得模式管理变得非常简单和高效，因为开发者无论如何都会更改类结构。它防止了*双重键入*，利用代码中已经完成的工作来自动化数据库中的相关更改。

如果你阅读了上一章，你会记得我们创建了一个名为**MyCompany**的数据库，作为从我们的 API 直接连接的示例。我们将继续使用这个数据库，但这次我们将使用 ORM 来映射其内的对象。让我们首先使用 ORM，Dapper 来实现这一点。

## 在最小 API 项目中配置 Dapper

我选择从 Dapper 开始，因为它与 Entity Framework 相比，通常被称为*微型 ORM*。这是因为它去掉了许多 ORM 功能，如结果缓存、更改跟踪、延迟加载和数据库迁移。相反，Dapper 专注于简单性和性能。作为一个更轻量级的解决方案，它可能更适合你的最小 API 需求。

Dapper 是学习 ORM 的一个好起点，因为它仍然使用 SQL 查询，这使得它在上一章中展示的直接连接方法和 Entity Framework 提供的更冗长的 ORM 功能集之间成为完美的中间地带。

数据库迁移和 Dapper

最后一个区分性功能（数据库迁移）非常重要，因为它与更改数据库模式的能力相关。当我们探索 Entity Framework 时，我们将在本章后面讨论迁移，但在此之前，要知道数据库迁移是通过代码更改数据库模式，而 Dapper 不支持通过迁移来实现这一点。然而，你仍然可以通过 Dapper 发送 SQL 命令来更改表结构。这超出了本章的范围。

Dapper 与许多数据库提供程序兼容，包括 SQL Server、Oracle、SQLite、MySQL 和 PostgreSQL。它使用 ADO.NET，这意味着它将与任何具有使用 ADO.NET 提供程序的数据库平台一起工作。因为我们正在使用上一章中创建的数据库，所以我们将连接到 SQL Server 数据库。

让我们在 Visual Studio 中创建一个新的（空白的）ASP.NET 项目，以便我们有一个干净的平台来工作。

一旦创建了项目，导航到**Program.cs**，它应该有一个**Hello World**模板示例。

首先，我们需要一个数据库来连接。如果你跟随上一章的内容，你将已经安装了一个 SQL Server 实例并创建了一个名为**MyCompany**的数据库。如果你还没有这样做，请安装 SQL Server 并在 SQL Server Management Studio 中创建数据库。

一旦有了数据库，如果你还没有**Employees**表，你可以使用以下 SQL 来创建一个：

```cs
CREATE TABLE dbo.Employees
    (
    Id int NOT NULL IDENTITY (1, 1),
    Name varchar(MAX) NOT NULL,
    Salary decimal(10, 2) NOT NULL,
    Address varchar(MAX) NOT NULL,
    City varchar(50) NOT NULL,
    Region varchar(50) NOT NULL,
    Country varchar(50) NOT NULL,
    Phone varchar(200) NOT NULL
    PostalCode varchar(50) NOT NULL,
)
```

Dapper 使用提供程序来简化与目标数据库平台的连接。对于 SQL Server，所需的提供程序与从 C#直接连接到 SQL Server 的要求相同：**Microsoft.Data.SqlClient**。

安装**Microsoft.Data.SqlClient**，无论是从**NuGet**包管理器 GUI 还是通过在包管理器控制台运行以下命令：

```cs
Install-Package Microsoft.Data.SqlClient
```

当我们在安装**NuGet**包时，我们还需要安装**Dapper**包。你可以在包管理器控制台运行以下命令来完成此操作：

```cs
Install-Package Dapper
```

从技术上讲，我们现在可以直接在我们的最小 API 端点中编写查询，但为了保持一致性和良好的实践，我们应该为 Dapper 创建一个新的服务并为其注册依赖注入。创建一个名为**DapperService**的类。正如我们在上一章中为**SQLService**和**MongoDbService**所做的那样，我们将在这个类中注册单例到**Program.cs**中。确保在**app.Run();**之前注册服务：

```cs
builder.Services.AddSingleton<DapperService>();
```

我们现在已经配置了项目，使其使用依赖注入和相关的提供程序在 SQL Server 数据库上使用 Dapper，并且我们已经创建了一个可以工作的数据库，这意味着我们可以从我们的最小 API 开始执行我们的第一个 CRUD 操作。

## 使用 Dapper 执行 CRUD 操作

让我们通过一些示例来逐步了解 Dapper 中 CRUD 的各个方面。首先，让我们创建一个员工。

### 创建员工记录

首先，我们将创建一个创建员工的端点。这意味着我们将使用**POST**方法：

1.  前往**Program.cs**并将 POST 端点映射到**employees**路由。它应该接受一个**Employee**作为参数，并且应该注入**DapperService**：

    ```cs
    app.MapPost(
        "/employees",
        (Employee employee,
        [FromServices] DapperService dapperService) =>
    {
    });
    ```

1.  接下来，我们可以在**DapperService**内部创建一个方法来处理在数据库中创建员工（这就是我们使用 Dapper 的地方）。

1.  打开**DapperService.cs**并创建一个名为**AddEmployee**的方法：

    ```cs
    public async Task AddEmployee(Employee employee)
    {
    }
    ```

    如前一章中的直接连接示例，Dapper 使用 **SqlConnection** 连接到 SQL Server。在该连接的作用域内，您需要编写执行所需操作的适当查询。因为我们正在创建一个 **employee**，所以我们将向数据库添加一条新记录，因此我们将编写一个 **INSERT** 语句。

1.  使用 **using** 语句来保持对数据库的连接（记住，**using** 语句允许自动释放连接）并将连接字符串添加到数据库中。

1.  紧接着，定义一个表示 **INSERT** 语句的字符串：

    ```cs
    using (var sqlConnection = new
        SqlConnection("YOURCONNECTIONSTRING"))
    {
        var sql = "INSERT INTO Employees " +
                   (Name, Salary, Address, City, " +
                   Region, "Country, Phone) VALUES " +
                   (@Name, @Salary, @Address, @City, " +
                   @Region, @Country, @Phone)";
    }
    ```

    注意到 **INSERT** 语句必须包含所有相关列及其值作为参数。

1.  接下来，我们将使用 Dapper 将数据库事务提交给实例化的 **SqlConnection**。我们可以使用 **ExecuteAsync()**，这是一个 Dapper 扩展方法，它将在映射从我们传递到这个 **AddEmployee()** 方法的 **Employee** 对象属性时执行语句：

    ```cs
    await sqlConnection.ExecuteAsync(sql, new
    {
        employee.PostalCode,
        employee.Name,
        employee.Salary,
        employee.Address,
        employee.City,
        employee.Region,
        employee.Country,
        employee.Phone
    });
    ```

    现在我们有一个在 **DapperService** 上的方法，可以接收一个 **Employee** 参数并通过 Dapper 将其提交到数据库，如下所示：

    ```cs
    public class DapperService
    {
        public async Task AddEmployee(Employee employee)
        {
            using (var sqlConnection = new
                SqlConnection("YOURCONNECTIONSTRING"))
            {
                var sql = "INSERT INTO Employees " +
                    (Name, Salary, Address, City, " +
                    Region, Country, Phone) VALUES " +
                    (@Name, @Salary, @Address, " +
                    @City, @Region, @Country, @Phone)";
                await sqlConnection.ExecuteAsync(sql, new
                {
                    employee.PostalCode,
                    employee.Name,
                    employee.Salary,
                    employee.Address,
                    employee.City,
                    employee.Region,
                    employee.Country,
                    employee.Phone
                });
            }
        }
    }
    ```

1.  现在剩下的只是调用我们在 **Program.cs** 中开始编写的 **POST** 端点上的此方法，在返回 **HTTP 201 CREATED** 状态码给客户端之前：

    ```cs
    app.MapPost(
        "/employees",
        async (Employee employee,
               [FromServices] DapperService dapperService)
               =>
    {
        await dapperService.AddEmployee(employee);
        return Results.Created();
    });
    ```

这样就处理了 CRUD 的 *Create* 部分。

存储和引用连接字符串

在上一章中，我演示了如何通过将连接字符串存储在配置文件中并通过 **IConfiguration** 引用来遵循最佳实践。

如果您还没有这样做，请参考上一章，以便您可以为 Dapper 和 Entity Framework 的使用实现它。

现在让我们继续到 *Read* 部分。

### 读取员工记录

*Read* 部分将包括在数据库上执行一个 **SELECT** 查询的 **GET** 端点：

1.  首先在 **Program.cs** 中添加一个 **GET** 端点，映射到 **Employees** 路由。它应该有一个名为 **id** 的路由参数，该参数通过端点内部 lambda 表达式的主体传递，并且应该注入 **DapperService**：

    ```cs
    app.MapGet(
        "/employees/{id}",
        (int id,
         [FromServices] DapperService dapperService) =>
    {
    });
    ```

1.  接下来，我们将采取与上一个示例相同的方法，在 **DapperService** 中创建一个新的函数，但这次，而不是插入，它将运行一个 **SELECT** 查询以获取指定 **id** 的员工，然后在返回到端点之前：

    ```cs
            public async Task<Employee>
                GetEmployeeById(int id)
            {
                using (var sqlConnection = new
                    SqlConnection("YOURCONNECTIONSTRING"))
                {
                    var sql = "SELECT * FROM Employees
                        WHERE Id = @employeeId";
                    var result = await sqlConnection
                        .QuerySingleAsync<Employee>(
                            sql, new { employeeId = id });
                    return result;
                }
            }
    ```

    **QuerySingleAsync<Employee>()** 是 Dapper 代码介入的地方。和之前一样，这是一个 Dapper 扩展方法，允许我们执行一个 SQL 查询，预期返回一条记录，即一个员工记录。

1.  注意发送的参数。我们传递了 SQL 查询，然后传递了一个新的对象数组实例。这是我们在 SQL 查询中声明的**@employeeId**参数的值。Dapper 期望我们以对象数组的形式传递参数值，以便它们的值可以映射到查询中的相关参数。

1.  我们还调用了一个特定的扩展方法——**QuerySingleAsync()**。原因很明显——我们只想得到一个记录。如果你需要更多，有如**Query()**这样的函数将返回包含多个记录的**IEnumerable**。

1.  最后，我们再次简单地从端点调用**DapperService**中的函数，将结果返回给客户端：

    ```cs
    app.MapGet(
        "/employees/{id}",
        async (int id,
               [FromServices] DapperService dapperService)
               =>
    {
         return Results.Ok(
             await dapperService.GetEmployeeById(id));
    });
    ```

接下来，让我们看看我们如何更新员工记录。

### 更新员工记录

我们现在已经进入了 CRUD 的第二个字母。让我们现在通过采取相同的方法来查看**更新**——创建一个**PUT**端点并将其连接到**DapperService**：

```cs
app.MapPut(
    "/employees",
     async (Employee employee,
            [FromServices] DapperService dapperService) =>
{
    await dapperService.UpdateEmployee(employee);
    return Results.Ok();
});
```

然后，我们创建一个函数来对数据库进行更新操作：

```cs
        public async Task UpdateEmployee (
            Employee employee)
        {
            using (var sqlConnection = new
                SqlConnection("YOURCONNECTIONSTRING"))
            {
                var sql = "UPDATE Employees SET Name = " +
                @Name, Salary = @Salary, Address = " +
                @Address, City = @City, " +
                Region = @Region WHERE Id = @id";
                var parameters = new
                {
                    employee.Id,
                    employee.Name,
                    employee.Salary,
                    employee.Address,
                    employee.City,
                    employee.Region
                };
                await sqlConnection.ExecuteAsync(
                    sql, parameters);
        }
        }
```

注意我们添加到**DapperService**中更新记录的代码与我们添加到创建记录的代码相似。主要区别是 SQL 字符串。否则，我们仍然传递一个**Employee**对象，并通过 Dapper 将其属性映射到数据库中的记录。

### 删除员工记录

最后，我们到达了 CRUD 的最后一部分——**删除**。

我们将遵循到目前为止在所有操作中应用的原则，但这次我将首先在**DapperService**中添加通过 ID 删除记录的功能：

```cs
        public async Task DeleteEmployeeById(int id)
        {
            using (var sqlConnection = new
                SqlConnection("YOURCONNECTIONSTRING"))
            {
                var sql = "DELETE FROM Employees WHERE Id =
                    @employeeId";
                await sqlConnection.ExecuteAsync(
                    sql,
                    new { employeeId = id }
                );
            }
        }
```

现在，我们可以添加一个端点来使用该服务进行记录删除，在成功时向客户端返回**无内容**的结果：

```cs
            app.MapDelete(
                "/employees/{id}",
                async (int id,
                       [FromServices] DapperService
                       dapperService) =>
            {
                await dapperService.DeleteEmployeeById(id);
                return Results.NoContent();
            });
```

我们已经介绍了足够的 Dapper 知识，你现在应该能够使用它通过这个强大而轻量级的（微）ORM 在 SQL 数据库上执行基本的简单 CRUD 操作。你也可以尝试用支持其他数据库的 SQL 提供程序替换 SQL 提供程序，例如 MySQL 或 PostgreSQL，以改善你使用 Dapper 管理 SQL 数据在最小 API 上的请求体验。

Dapper 有其位置，但 Entity Framework 是一个功能更丰富的替代品，不仅用于交易数据，还可以通过代码管理 SQL 数据库结构。

让我们看看我们如何使用 Entity Framework 构建我们在该部分中探索的相同功能。

## 在最小 API 项目中配置 Entity Framework

首先，我们需要使用 Microsoft 的 Entity Framework 包来配置连接，以下称为**上下文**。这种说法已经创建了一个从数据库的抽象层，因为我们开始将数据视为该上下文的成员。

首先，通过 Visual Studio 中的包管理器控制台安装以下包：

```cs
Install-Package Microsoft.EntityFrameworkCore
Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.EntityFrameworkCore.Tools
```

这确保了所有通过 Entity Framework 与 SQL Server 交互所需的库都已就绪。

接下来，我们将从包管理控制台再次使用数据库的连接字符串来**scaffold**现有的**MyCompany**数据库：

```cs
Scaffold-DbContext "Your_Connection_String" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Context MyCompanyContext
```

**Scaffolding**意味着 Entity Framework 会查看数据库的模式并将表映射到你的代码中的对象。结果是生成一个**DbContext**，它封装了上下文中的所有不同实体。还会创建一系列模型，这些模型是代表每个实体的类。这些模型被放置在由控制台命令使用的**OutputDir**开关指定的文件夹中。

因为我说了我想将上下文称为**MyCompanyContext**，所以在**Models**文件夹中生成了一个名为**MyCompanyContext**的新类。

在这个类中，你可以看到添加了一个名为**Employees**的**DbSet<Employee>**。一个**DbSet**代表给定数据库表中所有当前记录。这是一个表示**Employees**中每个记录的集合，通过向、从、更新或从这个集合中删除，我们可以间接更改相应的 SQL 表。让我们看看生成的代码，以进一步了解这里发生了什么。

**MyCompanyContext**是一个从**DbContext**派生的类。它代表了正在使用的数据源，并提供了一种以更高层次抽象与该源交互的方式。当类被实例化时，**DbContextOptions**会被注入其中，然后传递给基类**DbContext**：

```cs
public partial class MyCompanyContext : DbContext
{
    public MyCompanyContext()
    {
    }
    public MyCompanyContext(
        DbContextOptions<MyCompanyContext> options)
        : base(options)
    {
    }
    public virtual DbSet<Employee> Employees { get; set; }
```

Entity Framework 随后生成构建上下文所需的代码，将业务对象映射到数据库中的表，从而允许创建模型：

```cs
    protected override void OnConfiguring(
        DbContextOptionsBuilder optionsBuilder) =>
# warning To protect potentially sensitive information in
# your connection string, you should move it out of source
# code. You can avoid scaffolding the connection string by
# using the Name= syntax to read it from configuration –
# see https://go.microsoft.com/fwlink/?linkid=2131148\. For
# more guidance on storing connection strings, see
# https://go.microsoft.com/fwlink/?LinkId=723263.
```

在这个例子中，Entity Framework 被指示使用指定的连接字符串来使用 SQL Server 来根据数据库模式建模实体：

```cs
       optionsBuilder.UseSqlServer("YOURCONNECTIONSTRING");
    protected override void OnModelCreating(
        ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Employee>(entity =>
        {
            entity.HasNoKey();
            entity.Property(e => e.Address)
                .IsUnicode(false);
            entity.Property(e => e.City)
                .HasMaxLength(50)
                .IsUnicode(false);
            entity.Property(e => e.Country)
                .HasMaxLength(50)
                .IsUnicode(false);
            entity.Property(e => e.Id)
                .ValueGeneratedOnAdd();
            entity.Property(e => e.Name).IsUnicode(false);
            entity.Property(e => e.Phone)
                .HasMaxLength(200)
                .IsUnicode(false);
            entity.Property(e => e.PostalCode)
                .HasMaxLength(50)
                .IsUnicode(false);
            entity.Property(e => e.Region)
                .HasMaxLength(50)
                .IsUnicode(false);
            entity.Property(e => e.Salary)
                .HasColumnType("decimal(10, 2)");
        });
        OnModelCreatingPartial(modelBuilder);
    }
    partial void OnModelCreatingPartial(
        ModelBuilder modelBuilder);
}
```

在我们开始通过 API 端点使用 Entity Framework 与数据库通信之前，我们还有另一项配置要探索。

理解 Entity Framework 如何使用迁移来更改数据库模式是很重要的。迁移是一组指令，用于对数据库进行特定的更改。无论是向表中添加列、添加新表还是删除列，通常都可以通过迁移来完成。

除了数据之外，迁移还概述了如何管理数据库的结构变化，从而保持 Entity Framework 作为你数据的单一真相来源。

让我们通过修改我们的**Employees**表来探索迁移。我们将在表中添加一个**Title**列。首先，打开在**Models**文件夹中生成的**Employee**模型，并将**Title**添加为一个**string**属性：

```cs
  public string Title { get; set; }
```

然后，我们回到包管理控制台，输入一个命令来添加迁移。此命令要求 Entity Framework 查找**DbContext**中任何**DbSet**对象的变化。然后，它将生成最终会在数据库上运行 SQL 以提交更改的 C#代码。

在包管理控制台中使用 **Add-Migration** 命令，然后跟一个字符串，为迁移提供一个名称：

```cs
PM> Add-Migration "Add_title_to_employees_table"
```

迁移名称

当命名迁移时，提供这些迁移所做的更改的摘要是一个好的做法——例如，**changed_datatype_of_salary_column_to_decimal**。

Entity Framework 将构建项目，并在 **Migrations** 文件夹中添加一个新的迁移类，如果尚不存在，则创建该文件夹：

```cs
public partial class add_title_field_to_employees :
    Migration
{
    /// <inheritdoc />
    protected override void Up(
        MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<string>(
            name: "Title",
            table: "Employees",
            type: "nvarchar(max)",
            nullable: false,
            defaultValue: "");
    }
    /// <inheritdoc />
    protected override void Down(
        MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropColumn(
            name: "Title",
            table: "Employees");
    }
}
```

你可以看到迁移指定要添加一个名为 **Title** 的列。

要将此提交到数据库，我们可以在包管理控制台编写另一个简单的命令：

```cs
Update-Database
```

如果一切如预期进行，你将看到数据库中的 **Employees** 表有一个 **Title** 列。

现在我们已经介绍了其配置的基本知识，让我们更新 **Program.cs** 中的端点以使用 Entity Framework。

# 使用 Entity Framework 执行 CRUD 操作

我们有一个新的依赖项，形式为 **DbContext**。我们应该在 **Program.cs** 中的 **Main** 方法中注册它，以便在请求期间使用：

```cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddScoped<MyCompanyContext>();
var app = builder.Build();
```

就像我们在之前的 Dapper 示例中做的那样，我们将创建一个新的服务来管理 **Employee** 对象的 CRUD 操作。这次，我们将对命名更加具体，并将其称为 **EmployeeService**。

在 **EmployeeService** 中，首先添加一个构造函数，我们可以向其中传递已注册的上下文：

```cs
public class EmployeeService
{
    private MyCompanyContext _companyContext;
    public EmployeeService(
        MyCompanyContext myCompanyContext)
    {
        _companyContext = myCompanyContext;
    }
```

然后，定义所有需要的 CRUD 操作函数，使用 **MyCompanyContext**：

```cs
    public async Task AddEmployee(Employee employee)
    {
        await _companyContext.Employees
            .AddAsync(employee);
        await _companyContext
            .SaveChangesAsync();
    }
    public async Task<Employee> GetEmployeeById(int id)
    {
        var result =  await _companyContext.Employees
            .FirstOrDefaultAsync(x => x.Id == id);
        if(result == null)
        {
            throw new EmployeeNotFoundException(id);
        }
        return result;
    }
    public async Task UpdateEmployee(Employee employee)
    {
        var employeeToUpdate = await GetEmployeeById(
            employee.Id);
        _companyContext.Employees.Update(employeeToUpdate);
        await _companyContext.SaveChangesAsync();
    }
    public async Task DeleteEmployee(Employee employee)
    {
        _companyContext.Remove(employee);
        await _companyContext.SaveChangesAsync();
    }
}
```

最后，我们将创建一个名为 **EmployeeNotFoundException** 的客户异常，当请求的员工不在数据源中时，我们可以抛出这个异常：

```cs
public class EmployeeNotFoundException : Exception
{
    public EmployeeNotFoundException(int id)
        : base(
            $"Employee with id {id} could not be found")
    {
    }
}
```

在 **EmployeeService** 中的每个新函数中，我们通过位于 **MyCompanyContext** 中的 **Employees** 集合与数据库进行交互。然后，我们使用 **SaveChangesAsync();** 将我们对该集合所做的更改提交到数据库。

注意其中一个函数的可重用性，**GetEmployeeById**。在大多数 CRUD 操作中，我们需要能够定位受影响的 **Employee** 对象，我们可以重用这个函数。为了防止在集合中找不到员工的可能性，我们可以抛出一个自定义异常。这对于 API 端点很有用，因为它意味着它可以在发生时针对特定的异常进行响应，在这种情况下，如果需要，返回 **404 NOT FOUND** 状态码。

我们已经确定了在 Entity Framework 中，与数据库表交互意味着与 C# 中的集合交互。这在 **EmployeeService** 中很明显，其中使用了 LINQ 查询，而不是在直接 SQL 连接或像 Dapper 这样的微 ORM 中使用的 SQL 查询。

随着 **EmployeeService** 的建立，**Program.cs** 中的 API 端点可以修改为使用 Entity Framework 而不是之前使用的 Dapper。但它们需要改变到什么程度呢？

这个问题的答案是——并不多，多亏了我们以**EmployeeService**的形式创建的抽象。我们使用注入的依赖项来管理数据库交互，其中函数具有相同的名称或**签名**，这意味着我们可以简单地用新的使用 Entity Framework 的**EmployeeService**替换注入的**DapperService**。

返回到**Program.cs**并注册**EmployeeService**以进行依赖注入：

```cs
    var builder = WebApplication.CreateBuilder(args);
    builder.Services.AddScoped<MyCompanyContext>();
    builder.Services.AddScoped<EmployeeService>();
    var app = builder.Build();
```

双击当前传递给任何映射端点的**DapperService**参数。在 Visual Studio 中，您可以通过按两次*Ctrl*和*R*键来重命名此对象为**EmployeeService**。

重命名后，所有其他出现的地方也将被更新。只要每个端点调用的函数签名相同，无论您是使用**DapperService**还是**EmployeeService**，您都不应该有任何错误。

以下代码展示了创建**员工**后更新为使用**EmployeeService**的**POST**端点：

```cs
    app.MapPost(
        "/employees",
        async (Employee employee,
               [FromServices] EmployeeService
               employeeService) =>
    {
        await employeeService.AddEmployee(employee);
        return Results.Created();
    });
```

这是对 Entity Framework 的一个相当快速的介绍，它有更多功能。然而，主要重点是抽象方式下最小 API 与数据库的交互，这个例子足以让您开始 ORM 之旅。

让我们回顾一下本章关于 Dapper 和 Entity Framework 所学到的东西。

# 摘要

本章介绍了使用 Dapper 和 Entity Framework 来在最小 API 端点和关系型数据库之间提供抽象层。

我们从 ORMs 的介绍开始，定义了它们在简化最小 API 数据库交互中的作用，然后概述了与数据源交互的各种功能。

然后，我们逐步配置了 Dapper，添加了相关库，并提供了专门的**DapperService**，该服务可以在具有依赖注入的最小 API 端点上使用。一旦我们配置了 Dapper，就在**DapperService**中创建了 SQL 查询，并将端点链接到服务，通过 Dapper 提供 API 和数据库之间的端到端链接。

在使用 Dapper 在数据库上建立 CRUD 操作后，我们通过配置 Entity Framework 进行了对比，然后执行了完成 CRUD 操作的服务等效设置。

最后，使用 Entity Framework 的**EmployeeService**替换了原始的**DapperService**，展示了将抽象作为数据管理依赖项注入的灵活性。

毫无疑问，通过 ORMs 集成数据源是构建最小 API 的一个重要方面。在管理数据时，根据数据请求的方式，性能瓶颈的可能性可能很大。我们将在下一章探讨这个概念以及缓解这些瓶颈的方法。

# 第三部分 - 最佳最小 API

要构建高性能、可扩展的 API，对系统性能进行微调和利用高级编程技术至关重要。本部分涵盖了如何识别瓶颈、采用异步编程以及实施缓存策略来提高效率和用户体验。

本部分包含以下章节：

+   *第十章* ，*分析和识别瓶颈*

+   *第十一章* ，*利用异步编程实现可扩展性*

+   *第十二章* ，*增强性能的缓存策略*
