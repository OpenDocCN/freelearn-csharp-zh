# 第十章：使用 Entity Framework Core 处理数据

本章是关于使用名为**实体框架核心**（**EF Core**）的对象到数据存储映射技术，对数据存储（如 Microsoft SQL Server、SQLite 和 Azure Cosmos DB）进行读写。

本章将涵盖以下主题：

+   理解现代数据库

+   设置 EF Core

+   定义 EF Core 模型

+   EF Core 模型的查询

+   EF Core 中的加载模式

+   EF Core 中的数据操作

+   事务处理

+   Code First EF Core 模型

# 理解现代数据库

存储数据最常见的两种地方是**关系数据库管理系统**（**RDBMS**），如 Microsoft SQL Server、PostgreSQL、MySQL 和 SQLite，或者**NoSQL**数据库，如 Microsoft Azure Cosmos DB、Redis、MongoDB 和 Apache Cassandra。

## 理解遗留的实体框架

**实体框架**（**EF**）最初作为.NET Framework 3.5 的一部分，在 2008 年底随 Service Pack 1 发布。自那时起，微软观察到程序员如何在现实世界中使用**对象关系映射**（**ORM**）工具，实体框架也随之演进。

ORMs 利用映射定义将表中的列关联到类的属性上。这样，程序员就可以用他们熟悉的方式与不同类型的对象进行交互，而不必处理如何将值存储在关系表或 NoSQL 数据存储提供的其他结构中。

.NET Framework 中包含的 EF 版本是**实体框架 6**（**EF6**）。它成熟、稳定，并支持 EDMX（XML 文件）方式定义模型以及复杂的继承模型，以及其他一些高级功能。

EF 6.3 及更高版本已从.NET Framework 中提取出来，作为一个独立包，以便支持.NET Core 3.0 及更高版本。这使得现有的项目，如 Web 应用程序和服务，能够移植并在跨平台上运行。然而，EF6 应被视为一种遗留技术，因为它在跨平台运行时存在一些限制，并且不会再添加新功能。

### 使用遗留的实体框架 6.3 或更高版本

要在.NET Core 3.0 或更高版本的项目中使用遗留的实体框架，你必须在你的项目文件中添加对该包的引用，如下面的标记所示：

```cs
<PackageReference Include="EntityFramework" Version="6.4.4" /> 
```

**最佳实践**：仅在必要时使用遗留的 EF6，例如，当迁移使用它的 WPF 应用程序时。本书是关于现代跨平台开发的，因此在本章的其余部分，我将只介绍现代的实体框架核心。你不需要像上面那样在为本章项目引用遗留的 EF6 包。

## 理解实体框架核心

真正的跨平台版本，**EF Core**，与遗留的实体框架不同。尽管 EF Core 名称相似，但你应该意识到它与 EF6 的差异。最新的 EF Core 版本是 6.0，以匹配.NET 6.0。

EF Core 5 及更高版本仅支持 .NET 5 及更高版本。EF Core 3.0 及更高版本仅在支持 .NET Standard 2.1 的平台（即 .NET Core 3.0 及更高版本）上运行。它不支持 .NET Standard 2.0 平台，如 .NET Framework 4.8。

除了传统的关系数据库管理系统，EF Core 还支持现代基于云的、非关系型的、无模式的数据存储，如 Microsoft Azure Cosmos DB 和 MongoDB，有时通过第三方提供商支持。

EF Core 有许多改进，本章无法涵盖所有内容。我将重点介绍所有 .NET 开发者应该了解的基础知识以及一些较新的酷炫功能。

与 EF Core 工作有两种方法：

1.  **数据库先行**：数据库已经存在，因此你构建一个与数据库结构和特性相匹配的模型。

1.  **代码先行**：不存在数据库，因此你先构建一个模型，然后使用 EF Core 创建一个与该模型结构和特性相匹配的数据库。

我们将从使用 EF Core 与现有数据库开始。

## 创建一个用于与 EF Core 工作的控制台应用

首先，我们将为本章创建一个控制台应用项目：

1.  使用你偏好的代码编辑器创建一个名为 `Chapter10` 的新解决方案/工作区。

1.  添加一个控制台应用项目，如下表所示：

    1.  项目模板：**控制台应用程序** / `console`

    1.  工作区/解决方案文件和文件夹：`Chapter10`

    1.  项目文件和文件夹：`WorkingWithEFCore`

## 使用示例关系数据库

为了学习如何使用 .NET 管理关系数据库管理系统，拥有一个示例数据库会很有帮助，这样你就可以在一个中等复杂度和适当数量的示例记录上练习。微软提供了几个示例数据库，其中大多数对于我们的需求来说太复杂了，因此我们将使用一个在 20 世纪 90 年代初首次创建的数据库，称为 **Northwind**。

让我们花一分钟时间查看 Northwind 数据库的图表。你可以使用以下图表作为参考，在我们编写本书中的代码和查询时：

![图表描述自动生成](img/B17442_11_01.png)

图 10.1：Northwind 数据库表及其关系

你将在本章后面编写代码以与 `Categories` 和 `Products` 表交互，并在后续章节中与其他表交互。但在我们开始之前，请注意：

+   每个类别都有一个唯一的标识符、名称、描述和图片。

+   每个产品都有一个唯一的标识符、名称、单价、库存单位以及其他字段。

+   每个产品通过存储类别的唯一标识符与一个类别关联。

+   `Categories` 和 `Products` 之间的关系是一对多，意味着每个类别可以有零个或多个产品。

## 使用 Microsoft SQL Server for Windows

微软为其流行且功能强大的 SQL Server 产品提供了多种版本，适用于 Windows、Linux 和 Docker 容器。我们将使用一个可以独立运行的免费版本，称为 SQL Server 开发者版。你也可以使用 Express 版或与 Windows 上的 Visual Studio 一起安装的免费 SQL Server LocalDB 版。

如果您没有 Windows 电脑或希望使用跨平台数据库系统，则可以跳至主题*使用 SQLite*。

### 下载并安装 SQL Server。

您可以从以下链接下载 SQL Server 版本：

[`www.microsoft.com/en-us/sql-server/sql-server-downloads`](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)

1.  下载**开发者**版本。

1.  运行安装程序。

1.  选择**自定义**安装类型。

1.  选择安装文件夹，然后点击**安装**。

1.  等待 1.5 GB 的安装文件下载完成。

1.  在**SQL Server 安装中心**中，点击**安装**，然后点击**新 SQL Server 独立安装或向现有安装添加功能**。

1.  选择**开发者**作为免费版本，然后点击**下一步**。

1.  接受许可条款，然后点击**下一步**。

1.  审查安装规则，解决任何问题，然后点击**下一步**。

1.  在**功能选择**中，选择**数据库引擎服务**，然后点击**下一步**。

1.  在**实例配置**中，选择**默认实例**，然后点击**下一步**。如果您已有默认实例配置，则可以创建一个命名实例，可能名为`cs10dotnet6`。

1.  在**服务器配置**中，注意**SQL Server 数据库引擎**已设置为自动启动。将**SQL Server 浏览器**设置为自动启动，然后点击**下一步**。

1.  在**数据库引擎配置**中，在**服务器配置**标签页，设置**认证模式**为**混合**，设置**sa**账户密码为强密码，点击**添加当前用户**，然后点击**下一步**。

1.  在**准备安装**中，审查将要执行的操作，然后点击**安装**。

1.  在**完成**中，注意成功执行的操作，然后点击**关闭**。

1.  在**SQL Server 安装中心**中，点击**安装**，然后选择**安装 SQL Server 管理工具**。

1.  在浏览器窗口中，点击下载最新版本的 SSMS。

1.  运行安装程序并点击**安装**。

1.  安装程序完成后，如有需要点击**重启**或点击**关闭**。

## 为 SQL Server 创建 Northwind 示例数据库。

现在我们可以运行一个数据库脚本来创建 Northwind 示例数据库：

1.  如果您之前未下载或克隆本书的 GitHub 仓库，请使用以下链接进行操作：[`github.com/markjprice/cs10dotnet6/`](https://github.com/markjprice/cs10dotnet6/)。

1.  从本地 Git 仓库的以下路径复制创建 Northwind 数据库的脚本：`/sql-scripts/Northwind4SQLServer.sql`到`WorkingWithEFCore`文件夹。

1.  启动**SQL Server Management Studio**。

1.  在**连接到服务器**对话框中，对于**服务器名称**，输入`.`（一个点），表示本地计算机名称，然后点击**连接**。

    如果您需要创建一个命名实例，如`cs10dotnet6`，则输入`.\cs10dotnet6`。

1.  导航至**文件** | **打开** | **文件...**。

1.  浏览并选择`Northwind4SQLServer.sql`文件，然后点击**打开**。

1.  在工具栏上，点击**执行**，并注意显示的**命令已成功完成**消息。

1.  在**对象资源管理器**中，展开**Northwind**数据库，然后展开**表**。

1.  右键点击**产品**，点击**选择前 1000 行**，并注意返回的结果，如图*10.2*所示：![](img/B17442_11_04.png)

    图 10.2：SQL Server Management Studio 中的产品表

1.  在**对象资源管理器**工具栏上，点击**断开连接**按钮。

1.  退出 SQL Server Management Studio。

## 使用服务器资源管理器管理 Northwind 示例数据库

我们无需使用 SQL Server Management Studio 来执行数据库脚本。我们还可以使用 Visual Studio 中的工具，包括**SQL Server 对象资源管理器**和**服务器资源管理器**：

1.  在 Visual Studio 中，选择**视图** | **服务器资源管理器**。

1.  在**服务器资源管理器**窗口中，右键点击**数据连接**并选择**添加连接...**。

1.  如果您看到如图*10.3*所示的**选择数据源**对话框，请选择**Microsoft SQL Server**，然后点击**继续**：![图形用户界面，应用程序 自动生成描述](img/B17442_11_05.png)

    图 10.3：选择 SQL Server 作为数据源

1.  在**添加连接**对话框中，将服务器名称输入为`.`，将数据库名称输入为`Northwind`，然后点击**确定**。

1.  在**服务器资源管理器**中，展开数据连接及其表。您应该能看到 13 个表，包括**类别**和**产品**表。

1.  右键点击**产品**表，选择**显示表数据**，并注意返回的 77 行产品数据。

1.  要查看**产品**表的列和类型详细信息，右键点击**产品**并选择**打开表定义**，或在**服务器资源管理器**中双击该表。

## 使用 SQLite

SQLite 是一个小巧、跨平台、自包含的 RDBMS，属于公共领域。它是移动平台如 iOS（iPhone 和 iPad）和 Android 上最常用的 RDBMS。即使您使用 Windows 并在前一节中设置了 SQL Server，您可能也想设置 SQLite。我们编写的代码将与两者兼容，观察它们之间的细微差别也颇有趣味。

### 在 macOS 上设置 SQLite

SQLite 作为命令行应用程序`sqlite3`包含在 macOS 的`/usr/bin/`目录中。

### 在 Windows 上设置 SQLite

在 Windows 上，我们需要将 SQLite 文件夹添加到系统路径中，以便在命令提示符或终端输入命令时能够找到它：

1.  打开您喜欢的浏览器并导航至以下链接：[`www.sqlite.org/download.html`](https://www.sqlite.org/download.html)。

1.  向下滚动页面至**Windows 预编译二进制文件**部分。

1.  点击**sqlite-tools-win32-x86-3360000.zip**。请注意，文件版本号可能在此书出版后有所更新。

1.  将 ZIP 文件解压到一个名为`C:\Sqlite\`的文件夹中。

1.  导航至**Windows 设置**。

1.  搜索`环境`并选择**编辑系统环境变量**。在非英语版本的 Windows 上，请搜索您本地语言中的等效词汇以找到该设置。

1.  点击**环境变量**按钮。

1.  在**系统变量**中，从列表中选择**路径**，然后点击**编辑…**。

1.  点击**新建**，输入`C:\Sqlite`，然后按回车键。

1.  点击**确定**。

1.  点击**确定**。

1.  点击**确定**。

1.  关闭**Windows 设置**。

### 为其他操作系统设置 SQLite

SQLite 可以从以下链接下载并安装在其他操作系统上：[`www.sqlite.org/download.html`](https://www.sqlite.org/download.html)。

## 创建 SQLite 的 Northwind 示例数据库

现在我们可以使用 SQL 脚本为 SQLite 创建 Northwind 示例数据库：

1.  如果您之前未克隆本书的 GitHub 仓库，请现在使用以下链接进行克隆：[`github.com/markjprice/cs10dotnet6/`](https://github.com/markjprice/cs10dotnet6/)。

1.  从本地 Git 仓库的以下路径复制创建 Northwind 数据库的脚本：`/sql-scripts/Northwind4SQLite.sql`，将其粘贴到`WorkingWithEFCore`文件夹中。

1.  在`WorkingWithEFCore`文件夹中启动命令行：

    1.  在 Windows 上，启动**文件资源管理器**，右键点击`WorkingWithEFCore`文件夹，选择**在此处打开命令提示符**或**在 Windows 终端中打开**。

    1.  在 macOS 上，启动**Finder**，右键点击`WorkingWithEFCore`文件夹，选择**在此处新建终端**。

1.  执行以下命令，使用 SQLite 运行 SQL 脚本并创建`Northwind.db`数据库：

    ```cs
    sqlite3 Northwind.db -init Northwind4SQLite.sql 
    ```

1.  请耐心等待，因为此命令可能需要一段时间来创建数据库结构。最终，您将看到 SQLite 命令提示符，如下所示：

    ```cs
    -- Loading resources from Northwind4SQLite.sql 
    SQLite version 3.36.0 2021-08-24 15:20:15
    Enter ".help" for usage hints.
    sqlite> 
    ```

1.  在 Windows 上按 Ctrl + C 或在 macOS 上按 Ctrl + D 退出 SQLite 命令模式。

1.  保持终端或命令提示符窗口打开，因为您很快会再次用到它。

## 使用 SQLiteStudio 管理 Northwind 示例数据库

您可以使用跨平台的图形数据库管理器**SQLiteStudio**轻松管理 SQLite 数据库：

1.  访问以下链接：[`sqlitestudio.pl`](https://sqlitestudio.pl)，下载并解压应用程序至您偏好的位置。

1.  启动**SQLiteStudio**。

1.  在**数据库**菜单中，选择**添加数据库**。

1.  在**数据库**对话框中，在**文件**部分，点击黄色文件夹按钮浏览本地计算机上的现有数据库文件，选择`WorkingWithEFCore`文件夹中的`Northwind.db`文件，然后点击**确定**。

1.  右键点击**Northwind**数据库，选择**连接到数据库**。您将看到由脚本创建的 10 个表。（SQLite 的脚本比 SQL Server 的简单；它没有创建那么多表或其他数据库对象。）

1.  右键点击**产品**表，选择**编辑表**。

1.  在表编辑器窗口中，注意 `Products` 表的结构，包括列名、数据类型、键和约束，如图 *10.4* 所示：![图形用户界面，文本，应用程序 描述自动生成](img/B17442_11_07.png)

    图 10.4：SQLiteStudio 中的表编辑器显示产品表的结构

1.  在表编辑器窗口中，点击**数据**选项卡，您将看到 77 种产品，如图 *10.5* 所示：![图形用户界面，文本，应用程序 描述自动生成](img/B17442_11_08.png)

    图 10.5：数据选项卡显示产品表中的行

1.  在**数据库**窗口中，右键点击**Northwind**并选择**断开与数据库的连接**。

1.  退出 SQLiteStudio。

# 设置 EF Core

在我们深入探讨使用 EF Core 管理数据的实际操作之前，让我们简要讨论一下在 EF Core 数据提供者之间进行选择的问题。

## 选择 EF Core 数据库提供者

为了管理特定数据库中的数据，我们需要知道如何高效地与该数据库通信的类。

EF Core 数据库提供者是一组针对特定数据存储优化的类。甚至还有一个提供者用于在当前进程的内存中存储数据，这对于高性能单元测试非常有用，因为它避免了访问外部系统。

它们作为 NuGet 包分发，如下表所示：

| 要管理此数据存储 | 安装此 NuGet 包 |
| --- | --- |
| Microsoft SQL Server 2012 或更高版本 | `Microsoft.EntityFrameworkCore.SqlServer` |
| SQLite 3.7 或更高版本 | `Microsoft.EntityFrameworkCore.SQLite` |
| MySQL | `MySQL.Data.EntityFrameworkCore` |
| 内存中 | `Microsoft.EntityFrameworkCore.InMemory` |
| Azure Cosmos DB SQL API | `Microsoft.EntityFrameworkCore.Cosmos` |
| Oracle DB 11.2 | `Oracle.EntityFrameworkCore` |

您可以在同一项目中安装所需数量的 EF Core 数据库提供者。每个包都包括共享类型以及提供者特定的类型。

## 连接到数据库

要连接到 SQLite 数据库，我们只需要知道数据库文件名，使用参数 `Filename` 设置。

要连接到 SQL Server 数据库，我们需要知道以下列表中的多项信息：

+   服务器名称（如果有实例，则包括实例）。

+   数据库名称。

+   安全信息，例如用户名和密码，或者我们是否应该自动传递当前登录用户的凭据。

我们在**连接字符串**中指定这些信息。

为了向后兼容，我们可以在 SQL Server 连接字符串中使用多种可能的关键字来表示各种参数，如下表所示：

+   `Data Source` 或 `server` 或 `addr`：这些关键字是服务器名称（以及可选的实例）。您可以使用点 `.` 表示本地服务器。

+   `Initial Catalog` 或 `database`：这些关键字是数据库名称。

+   `Integrated Security`或`trusted_connection`：这些关键字设置为`true`或`SSPI`，以传递线程当前用户凭据。

+   `MultipleActiveResultSets`：此关键字设置为`true`，以启用单个连接同时处理多个表以提高效率。它用于延迟加载相关表中的行。

如上表所述，编写代码连接到 SQL Server 数据库时，您需要知道其服务器名称。服务器名称取决于您将连接的 SQL Server 版本和版本，如下表所示：

| SQL Server 版本 | 服务器名称\实例名称 |
| --- | --- |
| LocalDB 2012 | `(localdb)\v11.0` |
| LocalDB 2016 或更高版本 | `(localdb)\mssqllocaldb` |
| Express | `.\sqlexpress` |
| Full/Developer（默认实例） | `.` |
| Full/Developer（命名实例） | `.\cs10dotnet6` |

**最佳实践**：使用点`.`作为本地计算机名称的简写。请记住，SQL Server 服务器名称由两部分组成：计算机名称和 SQL Server 实例名称。您在自定义安装期间提供实例名称。

## 定义 Northwind 数据库上下文类

`Northwind`类将用于表示数据库。要使用 EF Core，该类必须继承自`DbContext`。此类知道如何与数据库通信并动态生成 SQL 语句以查询和操作数据。

您的`DbContext`派生类应有一个名为`OnConfiguring`的重写方法，该方法将设置数据库连接字符串。

为了方便您尝试 SQLite 和 SQL Server，我们将创建一个支持两者的项目，并使用一个`string`字段在运行时控制使用哪一个：

1.  在`WorkingWithEFCore`项目中，添加对 EF Core 数据提供程序的包引用，包括 SQL Server 和 SQLite，如下面的标记所示：

    ```cs
    <ItemGroup>
      <PackageReference
        Include="Microsoft.EntityFrameworkCore.Sqlite" 
        Version="6.0.0" />
      <PackageReference
        Include="Microsoft.EntityFrameworkCore.SqlServer" 
        Version="6.0.0" />
    </ItemGroup> 
    ```

1.  构建项目以恢复包。

1.  添加一个名为`ProjectConstants.cs`的类文件。

1.  在`ProjectConstants.cs`中，定义一个具有公共字符串常量的类，以存储您想要使用的数据库提供程序名称，如下面的代码所示：

    ```cs
    namespace Packt.Shared;
    public class ProjectConstants
    {
      public const string DatabaseProvider = "SQLite"; // or "SQLServer"
    } 
    ```

1.  在`Program.cs`中，导入`Packt.Shared`命名空间并输出数据库提供程序，如下面的代码所示：

    ```cs
    WriteLine($"Using {ProjectConstants.DatabaseProvider} database provider."); 
    ```

1.  添加一个名为`Northwind.cs`的类文件。

1.  在`Northwind.cs`中，定义一个名为`Northwind`的类，导入 EF Core 的主命名空间，使该类继承自`DbContext`，并在`OnConfiguring`方法中，检查`provider`字段以使用 SQLite 或 SQL Server，如下面的代码所示：

    ```cs
    using Microsoft.EntityFrameworkCore; // DbContext, DbContextOptionsBuilder
    using static System.Console;
    namespace Packt.Shared;
    // this manages the connection to the database
    public class Northwind : DbContext
    {
      protected override void OnConfiguring(
        DbContextOptionsBuilder optionsBuilder)
      {
        if (ProjectConstants.DatabaseProvider == "SQLite")
        {
          string path = Path.Combine(
            Environment.CurrentDirectory, "Northwind.db");
          WriteLine($"Using {path} database file.");
          optionsBuilder.UseSqlite($"Filename={path}");
        }
        else
        {
          string connection = "Data Source=.;" + 
            "Initial Catalog=Northwind;" + 
            "Integrated Security=true;" +
            "MultipleActiveResultSets=true;";
          optionsBuilder.UseSqlServer(connection);
        }
      }
    } 
    ```

    如果您使用的是 Windows 上的 Visual Studio，则编译后的应用程序在`WorkingWithEFCore\bin\Debug\net6.0`文件夹中执行，因此它将找不到数据库文件。

1.  在**解决方案资源管理器**中，右键单击`Northwind.db`文件并选择**属性**。

1.  在**属性**中，将**复制到输出目录**设置为**始终复制**。

1.  打开`WorkingWithEFCore.csproj`并注意新元素，如下面的标记所示：

    ```cs
    <ItemGroup>
      <None Update="Northwind.db">
        <CopyToOutputDirectory>Always</CopyToOutputDirectory>
      </None>
    </ItemGroup> 
    ```

    如果你使用的是 Visual Studio Code，那么编译后的应用程序将在 `WorkingWithEFCore` 文件夹中执行，因此无需复制即可找到数据库文件。

1.  运行控制台应用程序并注意输出，显示你选择的哪个数据库提供程序。

# 定义 EF Core 模型

EF Core 结合使用**约定**、**注解属性**和**Fluent API** 语句在运行时构建实体模型，以便对类执行的任何操作都可以自动转换为对实际数据库执行的操作。实体类表示表的结构，类的实例表示该表中的一行。

首先，我们将回顾定义模型的三种方式，并附上代码示例，随后我们将创建一些实现这些技术的类。

## 使用 EF Core 约定定义模型

我们将编写的代码将使用以下约定：

+   表名默认与 `DbContext` 类中 `DbSet<T>` 属性的名称相匹配，例如 `Products`。

+   列名默认与实体模型类中的属性名称相匹配，例如 `ProductId`。

+   `.NET` 中的 `string` 类型默认在数据库中为 `nvarchar` 类型。

+   `.NET` 中的 `int` 类型默认在数据库中为 `int` 类型。

+   主键默认是名为 `Id` 或 `ID` 的属性，或者当实体模型类名为 `Product` 时，属性可以名为 `ProductId` 或 `ProductID`。如果此属性是整数类型或 `Guid` 类型，则还假定它为 `IDENTITY` 列（插入时自动赋值的列类型）。

**良好实践**：还有许多其他约定你应该了解，你甚至可以定义自己的约定，但这超出了本书的范围。你可以在以下链接中阅读相关内容：[`docs.microsoft.com/en-us/ef/core/modeling/`](https://docs.microsoft.com/en-us/ef/core/modeling/)

## 使用 EF Core 注解属性定义模型

常规往往不足以完全映射类到数据库对象。为模型添加更多智能的一种简单方法是应用注解属性。

下表展示了一些常见属性：

| 属性 | 描述 |
| --- | --- |
| `[Required]` | 确保值不为 `null`。 |
| `[StringLength(50)]` | 确保值长度最多为 50 个字符。 |
| `[RegularExpression(expression)]` | 确保值与指定的正则表达式匹配。 |
| `[Column(TypeName = "money", Name = "UnitPrice")]` | 指定表中使用的列类型和列名称。 |

例如，在数据库中，产品名称的最大长度为 40，且值不能为空，如下所示，**数据定义语言**（**DDL**）代码高亮显示了如何创建名为 `Products` 的表及其列、数据类型、键和其他约束：

```cs
CREATE TABLE Products (
    ProductId       INTEGER       PRIMARY KEY,
    ProductName     NVARCHAR (40) NOT NULL,
    SupplierId      "INT",
    CategoryId      "INT",
    QuantityPerUnit NVARCHAR (20),
    UnitPrice       "MONEY"       CONSTRAINT DF_Products_UnitPrice DEFAULT (0),
    UnitsInStock    "SMALLINT"    CONSTRAINT DF_Products_UnitsInStock DEFAULT (0),
    UnitsOnOrder    "SMALLINT"    CONSTRAINT DF_Products_UnitsOnOrder DEFAULT (0),
    ReorderLevel    "SMALLINT"    CONSTRAINT DF_Products_ReorderLevel DEFAULT (0),
    Discontinued    "BIT"         NOT NULL
                                  CONSTRAINT DF_Products_Discontinued DEFAULT (0),
    CONSTRAINT FK_Products_Categories FOREIGN KEY (
        CategoryId
    )
    REFERENCES Categories (CategoryId),
    CONSTRAINT FK_Products_Suppliers FOREIGN KEY (
        SupplierId
    )
    REFERENCES Suppliers (SupplierId),
    CONSTRAINT CK_Products_UnitPrice CHECK (UnitPrice >= 0),
    CONSTRAINT CK_ReorderLevel CHECK (ReorderLevel >= 0),
    CONSTRAINT CK_UnitsInStock CHECK (UnitsInStock >= 0),
    CONSTRAINT CK_UnitsOnOrder CHECK (UnitsOnOrder >= 0) 
); 
```

在`Product`类中，我们可以应用属性来指定这一点，如下面的代码所示：

```cs
[Required] 
[StringLength(40)]
public string ProductName { get; set; } 
```

当.NET 类型和数据库类型之间没有明显的映射时，可以使用属性。

例如，在数据库中，`Products`表的`UnitPrice`列的类型是`money`。.NET 没有`money`类型，因此应使用`decimal`代替，如下面的代码所示：

```cs
[Column(TypeName = "money")]
public decimal? UnitPrice { get; set; } 
```

另一个例子是针对`Categories`表的，如下面的 DDL 代码所示：

```cs
CREATE TABLE Categories (
    CategoryId   INTEGER       PRIMARY KEY,
    CategoryName NVARCHAR (15) NOT NULL,
    Description  "NTEXT",
    Picture      "IMAGE"
); 
```

`Description`列可能比`nvarchar`变量可以存储的最大 8,000 个字符更长，因此需要映射到`ntext`，如下面的代码所示：

```cs
[Column(TypeName = "ntext")]
public string Description { get; set; } 
```

## 使用 EF Core Fluent API 定义模型

定义模型的最后一种方式是使用 Fluent API。此 API 可以替代属性使用，也可以与属性一起使用。例如，为了定义`ProductName`属性，而不是用两个属性装饰该属性，可以在数据库上下文类的`OnModelCreating`方法中编写等效的 Fluent API 语句，如下面的代码所示：

```cs
modelBuilder.Entity<Product>()
  .Property(product => product.ProductName)
  .IsRequired()
  .HasMaxLength(40); 
```

这使得实体模型类更简单。

### 理解使用 Fluent API 进行数据播种

使用 Fluent API 的另一个好处是提供初始数据以填充数据库。EF Core 会自动计算出必须执行哪些插入、更新或删除操作。

例如，如果我们想确保新数据库的`Product`表中至少有一行，那么我们将调用`HasData`方法，如下面的代码所示：

```cs
modelBuilder.Entity<Product>()
  .HasData(new Product
  {
    ProductId = 1,
    ProductName = "Chai",
    UnitPrice = 8.99M
  }); 
```

我们的模型将映射到一个已填充数据的现有数据库，因此我们不需要在我们的代码中使用这种技术。

## 为 Northwind 表构建 EF Core 模型

现在你已经了解了定义 EF Core 模型的方法，让我们构建一个模型来表示`Northwind`数据库中的两个表。

两个实体类将相互引用，为了避免编译器错误，我们将首先创建不包含任何成员的类：

1.  在`WorkingWithEFCore`项目中，添加两个名为`Category.cs`和`Product.cs`的类文件。

1.  在`Category.cs`中，定义一个名为`Category`的类，如下面的代码所示：

    ```cs
    namespace Packt.Shared;
    public class Category
    {
    } 
    ```

1.  在`Product.cs`中，定义一个名为`Product`的类，如下面的代码所示：

    ```cs
    namespace Packt.Shared;
    public class Product
    {
    } 
    ```

### 定义 Category 和 Product 实体类

`Category`类，也称为实体模型，将用于表示`Categories`表中的一行。该表有四列，如下面的 DDL 所示：

```cs
CREATE TABLE Categories (
    CategoryId   INTEGER       PRIMARY KEY,
    CategoryName NVARCHAR (15) NOT NULL,
    Description  "NTEXT",
    Picture      "IMAGE"
); 
```

我们将使用约定来定义：

+   四个属性中的三个（我们将不映射`Picture`列）。

+   主键。

+   与`Products`表的一对多关系。

为了将`Description`列映射到正确的数据库类型，我们需要用`Column`属性装饰`string`属性。

本章后面，我们将使用 Fluent API 定义`CategoryName`不能为空，且最多只能有 15 个字符。

开始吧：

1.  修改`Category`实体模型类，如下所示：

    ```cs
    using System.ComponentModel.DataAnnotations.Schema; // [Column]
    namespace Packt.Shared;
    public class Category
    {
      // these properties map to columns in the database
      public int CategoryId { get; set; }
      public string? CategoryName { get; set; }
      [Column(TypeName = "ntext")]
      public string? Description { get; set; }
      // defines a navigation property for related rows
      public virtual ICollection<Product> Products { get; set; }
      public Category()
      {
        // to enable developers to add products to a Category we must
        // initialize the navigation property to an empty collection
        Products = new HashSet<Product>();
      }
    } 
    ```

    `Product`类将用于表示`Products`表中的一行，该表有十列。

    你不需要将表中的所有列都作为类的属性包含在内。我们只会映射六个属性：`ProductId`、`ProductName`、`UnitPrice`、`UnitsInStock`、`Discontinued`和`CategoryId`。

    未映射到属性的列不能通过类实例读取或设置。如果你使用该类创建一个新对象，那么表中新行的未映射列值将为`NULL`或其他默认值。你必须确保这些缺失的列是可选的，或者由数据库设置了默认值，否则在运行时会抛出异常。在这种情况下，行中已有数据值，我已决定在本应用程序中不需要读取这些值。

    我们可以通过定义一个不同名称的属性，如`Cost`，然后使用`[Column]`属性装饰该属性并指定其列名称，如`UnitPrice`，来重命名一列。

    最后一个属性`CategoryId`与一个`Category`属性关联，该属性将用于将每个产品映射到其父类别。

1.  修改`Product`类，如下所示：

    ```cs
    using System.ComponentModel.DataAnnotations; // [Required], [StringLength]
    using System.ComponentModel.DataAnnotations.Schema; // [Column]
    namespace Packt.Shared;
    public class Product
    {
      public int ProductId { get; set; } // primary key
      [Required]
      [StringLength(40)]
      public string ProductName { get; set; } = null!;
      [Column("UnitPrice", TypeName = "money")]
      public decimal? Cost { get; set; } // property name != column name
      [Column("UnitsInStock")]
      public short? Stock { get; set; }
      public bool Discontinued { get; set; }
      // these two define the foreign key relationship
      // to the Categories table
      public int CategoryId { get; set; }
      public virtual Category Category { get; set; } = null!;
    } 
    ```

关联两个实体的两个属性，`Category.Products`和`Product.Category`，都被标记为`virtual`。这使得 EF Core 能够继承并重写这些属性，以提供额外功能，如延迟加载。

## 向 Northwind 数据库上下文类添加表

在你的`DbContext`派生类中，你必须至少定义一个`DbSet<T>`类型的属性。这些属性代表表。为了告诉 EF Core 每个表有哪些列，`DbSet<T>`属性使用泛型来指定一个代表表中一行的类。该实体模型类具有代表其列的属性。

`DbContext`派生类可以选择性地重写名为`OnModelCreating`的方法。在这里，你可以编写 Fluent API 语句，作为用属性装饰实体类的一种替代方法。

让我们来写一些代码：

1.  修改`Northwind`类，添加语句以定义两个表的两个属性及一个`OnModelCreating`方法，如下所示，高亮部分：

    ```cs
    public class Northwind : DbContext
    {
    **// these properties map to tables in the database**
    **public** **DbSet<Category>? Categories {** **get****;** **set****; }**
    **public** **DbSet<Product>? Products {** **get****;** **set****; }**
      protected override void OnConfiguring(
        DbContextOptionsBuilder optionsBuilder)
      {
        ...
      }
    **protected****override****void****OnModelCreating****(**
     **ModelBuilder modelBuilder****)**
     **{**
    **// example of using Fluent API instead of attributes**
    **// to limit the length of a category name to 15**
     **modelBuilder.Entity<Category>()**
     **.Property(category => category.CategoryName)**
     **.IsRequired()** **// NOT NULL**
     **.HasMaxLength(****15****);**
    **if** **(ProjectConstants.DatabaseProvider ==** **"SQLite"****)**
     **{**
    **// added to "fix" the lack of decimal support in SQLite**
     **modelBuilder.Entity<Product>()**
     **.Property(product => product.Cost)**
     **.HasConversion<****double****>();**
     **}**
     **}**
    } 
    ```

EF Core 3.0 及更高版本中，SQLite 数据库提供程序不支持`decimal`类型进行排序和其他操作。我们可以通过告诉模型在使用 SQLite 数据库提供程序时`decimal`值可以转换为`double`值来解决这个问题。这实际上在运行时不会执行任何转换。

既然你已经看到了手动定义实体模型的一些示例，让我们来看一个能为你完成部分工作的工具。

## 设置 dotnet-ef 工具

.NET 有一个名为`dotnet`的命令行工具。它可以扩展用于与 EF Core 工作的有用功能。它可以执行设计时任务，如从旧模型到新模型创建和应用迁移，以及从现有数据库为模型生成代码。

`dotnet` `ef`命令行工具不是自动安装的。您必须将此包作为**全局**或**本地**工具安装。如果您已经安装了该工具的旧版本，则应卸载任何现有版本：

1.  在命令提示符或终端中，检查是否已将`dotnet-ef`作为全局工具安装，如以下命令所示：

    ```cs
    dotnet tool list --global 
    ```

1.  检查列表中是否已安装了旧版本的工具，例如 .NET Core 3.1 的版本，如以下输出所示：

    ```cs
    Package Id      Version     Commands
    -------------------------------------
    dotnet-ef       3.1.0       dotnet-ef 
    ```

1.  如果已安装旧版本，则卸载该工具，如以下命令所示：

    ```cs
    dotnet tool uninstall --global dotnet-ef 
    ```

1.  安装最新版本，如以下命令所示：

    ```cs
    dotnet tool install --global dotnet-ef --version 6.0.0 
    ```

1.  如有必要，请按照任何特定于操作系统的说明，将`dotnet tools`目录添加到您的 PATH 环境变量中，如安装`dotnet-ef`工具的输出所述。

## 使用现有数据库脚手架模型

脚手架是指使用工具创建表示现有数据库模型的类的过程，使用逆向工程。一个好的脚手架工具允许您扩展自动生成的类，然后重新生成这些类而不会丢失您的扩展类。

如果你确定永远不会使用工具重新生成这些类，那么请随意修改自动生成的类的代码。工具生成的代码只是最佳近似。

**良好实践**：当你更了解情况时，不要害怕推翻工具的建议。

让我们看看工具是否生成了与我们手动创建相同的模型：

1.  将`Microsoft.EntityFrameworkCore.Design`包添加到`WorkingWithEFCore`项目中。

1.  在`WorkingWithEFCore`文件夹中的命令提示符或终端中，为`Categories`和`Products`表在新文件夹`AutoGenModels`中生成模型，如以下命令所示：

    ```cs
    dotnet ef dbcontext scaffold "Filename=Northwind.db" Microsoft.EntityFrameworkCore.Sqlite --table Categories --table Products --output-dir AutoGenModels --namespace WorkingWithEFCore.AutoGen --data-annotations --context Northwind 
    ```

    注意以下事项：

    +   命令操作：`dbcontext scaffold`

    +   连接字符串：`"Filename=Northwind.db"`

    +   数据库提供程序：`Microsoft.EntityFrameworkCore.Sqlite`

    +   生成模型的表：`--table Categories --table Products`

    +   输出文件夹：`--output-dir AutoGenModels`

    +   命名空间：`--namespace WorkingWithEFCore.AutoGen`

    +   同时使用数据注释和 Fluent API：`--data-annotations`

    +   将上下文从[数据库名称]Context 重命名为：`--context Northwind`

    对于 SQL Server，更改数据库提供程序和连接字符串，如以下命令所示：

    ```cs
    dotnet ef dbcontext scaffold "Data Source=.;Initial Catalog=Northwind;Integrated Security=true;" Microsoft.EntityFrameworkCore.SqlServer --table Categories --table Products --output-dir AutoGenModels --namespace WorkingWithEFCore.AutoGen --data-annotations --context Northwind 
    ```

1.  注意构建消息和警告，如以下输出所示：

    ```cs
    Build started...
    Build succeeded.
    To protect potentially sensitive information in your connection string, you should move it out of source code. You can avoid scaffolding the connection string by using the Name= syntax to read it from configuration - see https://go.microsoft.com/fwlink/?linkid=2131148\. For more guidance on storing connection strings, see http://go.microsoft.com/fwlink/?LinkId=723263.
    Skipping foreign key with identity '0' on table 'Products' since principal table 'Suppliers' was not found in the model. This usually happens when the principal table was not included in the selection set. 
    ```

1.  打开`AutoGenModels`文件夹，并注意自动生成的三个类文件：`Category.cs`、`Northwind.cs`和`Product.cs`。

1.  打开`Category.cs`，注意与您手动创建的版本相比的差异，如下面的代码所示：

    ```cs
    using System;
    using System.Collections.Generic;
    using System.ComponentModel.DataAnnotations;
    using System.ComponentModel.DataAnnotations.Schema; 
    using Microsoft.EntityFrameworkCore;
    namespace WorkingWithEFCore.AutoGen
    {
      [Index(nameof(CategoryName), Name = "CategoryName")]
      public partial class Category
      {
        public Category()
        {
          Products = new HashSet<Product>();
        }
        [Key]
        public long CategoryId { get; set; }
        [Required]
        [Column(TypeName = "nvarchar (15)")] // SQLite
        [StringLength(15)] // SQL Server
        public string CategoryName { get; set; }
        [Column(TypeName = "ntext")]
        public string? Description { get; set; }
        [Column(TypeName = "image")]
        public byte[]? Picture { get; set; }
        [InverseProperty(nameof(Product.Category))]
        public virtual ICollection<Product> Products { get; set; }
      }
    } 
    ```

    注意以下内容：

    +   它用 EF Core 5.0 中引入的`[Index]`属性装饰实体类。这表示应为哪些属性创建索引。在早期版本中，仅支持 Fluent API 定义索引。由于我们正在使用现有数据库，因此不需要这样做。但如果我们想从代码重新创建一个新的空数据库，则需要这些信息。

    +   数据库中的表名为`Categories`，但`dotnet-ef`工具使用第三方库**Humanizer**自动将类名单数化，变为`Category`，这在创建单个实体时是一个更自然的名称。

    +   实体类使用`partial`关键字声明，以便您可以创建匹配的部分类来添加额外的代码。这样，您可以重新运行工具并重新生成实体类，而不会丢失那些额外的代码。

    +   `CategoryId`属性用`[Key]`属性装饰，表示它是此实体的主键。该属性的数据类型对于 SQL Server 是`int`，对于 SQLite 是`long`。

    +   `Products`属性使用`[InverseProperty]`属性定义与`Product`实体类上的`Category`属性的外键关系。

1.  打开`Product.cs`，注意与您手动创建的版本相比的差异。

1.  打开`Northwind.cs`，注意与您手动创建的版本相比的差异，如下面的编辑后代码所示：

    ```cs
    using Microsoft.EntityFrameworkCore; 
    namespace WorkingWithEFCore.AutoGen
    {
      public partial class Northwind : DbContext
      {
        public Northwind()
        {
        }
        public Northwind(DbContextOptions<Northwind> options)
          : base(options)
        {
        }
        public virtual DbSet<Category> Categories { get; set; } = null!;
        public virtual DbSet<Product> Products { get; set; } = null!;
        protected override void OnConfiguring(
          DbContextOptionsBuilder optionsBuilder)
        {
          if (!optionsBuilder.IsConfigured)
          {
    #warning To protect potentially sensitive information in your connection string, you should move it out of source code. You can avoid scaffolding the connection string by using the Name= syntax to read it from configuration - see https://go.microsoft.com/fwlink/?linkid=2131148\. For more guidance on storing connection strings, see http://go.microsoft.com/fwlink/?LinkId=723263.
            optionsBuilder.UseSqlite("Filename=Northwind.db");
          }
        }
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
          modelBuilder.Entity<Category>(entity =>
          {
            ...
          });
          modelBuilder.Entity<Product>(entity =>
          {
            ...
          });
          OnModelCreatingPartial(modelBuilder);
        }
        partial void OnModelCreatingPartial(ModelBuilder modelBuilder);
      }
    } 
    ```

    注意以下内容：

    +   北风数据上下文类被声明为`partial`，以便您可以扩展它并在将来重新生成它。

    +   它有两个构造函数：一个无参数的默认构造函数和一个允许传递选项的构造函数。这在您希望在运行时指定连接字符串的应用程序中非常有用。

    +   代表`Categories`和`Products`表的两个`DbSet<T>`属性被设置为`null`-forgiving 值，以防止在编译时出现静态编译器分析警告。这在运行时没有影响。

    +   在`OnConfiguring`方法中，如果在构造函数中未指定选项，则默认使用在当前文件夹中查找数据库文件的连接字符串。它有一个编译器警告，提醒您不应在此连接字符串中硬编码安全信息。

    +   在`OnModelCreating`方法中，使用 Fluent API 配置了两个实体类，然后调用了名为`OnModelCreatingPartial`的部分方法。这允许您在自己的部分`Northwind`类中实现该部分方法，添加自己的 Fluent API 配置，这样在重新生成模型类时不会丢失这些配置。

1.  关闭自动生成的类文件。

## 配置预设模型

除了支持`DateOnly`和`TimeOnly`类型用于 SQLite 数据库提供程序外，EF Core 6 引入的新特性之一是配置预约定模型。

随着模型变得更为复杂，依赖约定来发现实体类型及其属性和成功地将它们映射到表和列变得更加困难。如果在分析和构建模型之前能够配置这些约定，将会非常有用。

例如，你可能想要定义一个约定，规定所有`string`属性默认应有一个最大长度为 50 个字符，或者任何实现自定义接口的属性类型不应被映射，如下所示代码：

```cs
protected override void ConfigureConventions(
  ModelConfigurationBuilder configurationBuilder)
{
  configurationBuilder.Properties<string>().HaveMaxLength(50);
  configurationBuilder.IgnoreAny<IDoNotMap>();
} 
```

在本章的其余部分，我们将使用你手动创建的类。

# 查询 EF Core 模型

现在我们有了一个映射到 Northwind 数据库及其两个表的模型，我们可以编写一些简单的 LINQ 查询来获取数据。在*第十一章*，*使用 LINQ 查询和操作数据*中，你将学习更多关于编写 LINQ 查询的知识。

目前，只需编写代码并查看结果：

1.  在`Program.cs`顶部，导入主要的 EF Core 命名空间，以启用使用`Include`扩展方法从相关表预先加载：

    ```cs
    using Microsoft.EntityFrameworkCore; // Include extension method 
    ```

1.  在`Program.cs`底部，定义一个`QueryingCategories`方法，并添加执行这些任务的语句，如下所示代码中所示：

    +   创建一个`Northwind`类的实例来管理数据库。数据库上下文实例设计为在单元工作中的短期生命周期。它们应尽快被释放，因此我们将它包裹在一个`using`语句中。在*第十四章*，*使用 ASP.NET Core Razor Pages 构建网站*中，你将学习如何通过依赖注入获取数据库上下文。

    +   创建一个查询，获取所有包含相关产品的类别。

    +   遍历类别，输出每个类别的名称和产品数量：

    ```cs
    static void QueryingCategories()
    {
      using (Northwind db = new())
      {
        WriteLine("Categories and how many products they have:");
        // a query to get all categories and their related products
        IQueryable<Category>? categories = db.Categories?
          .Include(c => c.Products);
        if (categories is null)
        {
          WriteLine("No categories found.");
          return;
        }
        // execute query and enumerate results
        foreach (Category c in categories)
        {
          WriteLine($"{c.CategoryName} has {c.Products.Count} products.");
        }
      }
    } 
    ```

1.  在`Program.cs`顶部，在输出数据库提供程序名称后，调用`QueryingCategories`方法，如下所示代码中高亮显示的部分：

    ```cs
    WriteLine($"Using {ProjectConstants.DatabaseProvider} database provider.");
    **QueryingCategories();** 
    ```

1.  运行代码并查看结果（如果使用 Visual Studio 2022 for Windows 并使用 SQLite 数据库提供程序运行），如下所示输出：

    ```cs
    Using SQLite database provider.
    Categories and how many products they have: 
    Using C:\Code\Chapter10\WorkingWithEFCore\bin\Debug\net6.0\Northwind.db database file.
    Beverages has 12 products.
    Condiments has 12 products. 
    Confections has 13 products. 
    Dairy Products has 10 products. 
    Grains/Cereals has 7 products. 
    Meat/Poultry has 6 products.
    Produce has 5 products. 
    Seafood has 12 products. 
    ```

如果你使用 Visual Studio Code 运行并使用 SQLite 数据库提供程序，那么路径将是`WorkingWithEFCore`文件夹。如果你使用 SQL Server 数据库提供程序运行，则不会有数据库文件路径输出。

**警告！** 如果你在使用 Visual Studio 2022 中的 SQLite 时看到以下异常，最可能的问题是`Northwind.db`文件没有被复制到输出目录。请确保**复制到输出目录**设置为**总是复制**：

`未处理的异常。Microsoft.Data.Sqlite.SqliteException (0x80004005): SQLite 错误 1: '没有这样的表: Categories'。`

## 过滤包含的实体

EF Core 5.0 引入了**过滤包含**，这意味着您可以在`Include`方法调用中指定一个 lambda 表达式，以过滤结果中返回哪些实体：

1.  在`Program.cs`底部，定义一个`FilteredIncludes`方法，并添加语句执行这些任务，如下所示：

    +   创建一个`Northwind`类的实例来管理数据库。

    +   提示用户输入库存单位的最小值。

    +   创建一个查询，查找具有该最小库存单位数量的产品的类别。

    +   遍历类别和产品，输出每个的名称和库存单位：

    ```cs
    static void FilteredIncludes()
    {
      using (Northwind db = new())
      {
        Write("Enter a minimum for units in stock: ");
        string unitsInStock = ReadLine() ?? "10";
        int stock = int.Parse(unitsInStock);
        IQueryable<Category>? categories = db.Categories?
          .Include(c => c.Products.Where(p => p.Stock >= stock));
        if (categories is null)
        {
          WriteLine("No categories found.");
          return;
        }
        foreach (Category c in categories)
        {
          WriteLine($"{c.CategoryName} has {c.Products.Count} products with a minimum of {stock} units in stock.");
          foreach(Product p in c.Products)
          {
            WriteLine($"  {p.ProductName} has {p.Stock} units in stock.");
          }
        }
      }
    } 
    ```

1.  在`Program.cs`中，注释掉`QueryingCategories`方法，并调用`FilteredIncludes`方法，如以下高亮代码所示：

    ```cs
    WriteLine($"Using {ProjectConstants.DatabaseProvider} database provider.");
    **// QueryingCategories();**
    **FilteredIncludes();** 
    ```

1.  运行代码，输入库存单位的最小值，如`100`，并查看结果，如下所示：

    ```cs
    Enter a minimum for units in stock: 100
    Beverages has 2 products with a minimum of 100 units in stock.
      Sasquatch Ale has 111 units in stock.
      Rhönbräu Klosterbier has 125 units in stock.
    Condiments has 2 products with a minimum of 100 units in stock.
      Grandma's Boysenberry Spread has 120 units in stock.
      Sirop d'érable has 113 units in stock.
    Confections has 0 products with a minimum of 100 units in stock. 
    Dairy Products has 1 products with a minimum of 100 units in stock.
      Geitost has 112 units in stock.
    Grains/Cereals has 1 products with a minimum of 100 units in stock.
      Gustaf's Knäckebröd has 104 units in stock.
    Meat/Poultry has 1 products with a minimum of 100 units in stock.
      Pâté chinois has 115 units in stock.
    Produce has 0 products with a minimum of 100 units in stock. 
    Seafood has 3 products with a minimum of 100 units in stock.
      Inlagd Sill has 112 units in stock.
      Boston Crab Meat has 123 units in stock. 
      Röd Kaviar has 101 units in stock. 
    ```

### Windows 控制台中的 Unicode 字符

在 Windows 10 Fall Creators Update 之前的 Windows 版本中，微软提供的控制台存在限制。默认情况下，控制台无法显示 Unicode 字符，例如 Rhönbräu 的名称。

如果您遇到此问题，则可以通过在运行应用之前在提示符下输入以下命令来临时更改控制台中的代码页（也称为字符集）为 Unicode UTF-8：

```cs
chcp 65001 
```

## 过滤和排序产品

让我们探索一个更复杂的查询，它将过滤和排序数据：

1.  在`Program.cs`底部，定义一个`QueryingProducts`方法，并添加语句执行以下操作，如下所示：

    +   创建一个`Northwind`类的实例来管理数据库。

    +   提示用户输入产品价格。与之前的代码示例不同，我们将循环直到输入有效价格。

    +   使用 LINQ 创建一个查询，查找价格高于指定价格的产品。

    +   遍历结果，输出 Id、名称、成本（以美元格式化）和库存单位数量：

    ```cs
    static void QueryingProducts()
    {
      using (Northwind db = new())
      {
        WriteLine("Products that cost more than a price, highest at top."); 
        string? input;
        decimal price;
        do
        {
          Write("Enter a product price: ");
          input = ReadLine();
        } while (!decimal.TryParse(input, out price));
        IQueryable<Product>? products = db.Products?
          .Where(product => product.Cost > price)
          .OrderByDescending(product => product.Cost);
        if (products is null)
        {
          WriteLine("No products found.");
          return;
        }
        foreach (Product p in products)
        {
          WriteLine(
            "{0}: {1} costs {2:$#,##0.00} and has {3} in stock.",
            p.ProductId, p.ProductName, p.Cost, p.Stock);
        }
      }
    } 
    ```

1.  在`Program.cs`中，注释掉之前的方法，并调用`QueryingProducts`方法

1.  运行代码，当提示输入产品价格时，输入`50`，并查看结果，如下所示：

    ```cs
    Products that cost more than a price, highest at top. 
    Enter a product price: 50
    38: Côte de Blaye costs $263.50 and has 17 in stock.
    29: Thüringer Rostbratwurst costs $123.79 and has 0 in stock. 
    9: Mishi Kobe Niku costs $97.00 and has 29 in stock.
    20: Sir Rodney's Marmalade costs $81.00 and has 40 in stock. 
    18: Carnarvon Tigers costs $62.50 and has 42 in stock.
    59: Raclette Courdavault costs $55.00 and has 79 in stock. 
    51: Manjimup Dried Apples costs $53.00 and has 20 in stock. 
    ```

## 获取生成的 SQL

您可能会好奇我们编写的 C#查询生成的 SQL 语句写得如何。EF Core 5.0 引入了一种快速简便的方法来查看生成的 SQL：

1.  在`FilteredIncludes`方法中，在使用`foreach`语句遍历查询之前，添加一条语句以输出生成的 SQL，如下所示：

    ```cs
    **WriteLine(****$"ToQueryString:** **{categories.ToQueryString()}****"****);**
    foreach (Category c in categories) 
    ```

1.  在`Program.cs`中，注释掉对`QueryingProducts`方法的调用，并取消对`FilteredIncludes`方法的调用。

1.  运行代码，输入库存单位的最小值，如`99`，并查看结果（使用 SQLite 运行时），如下所示：

    ```cs
    Enter a minimum for units in stock: 99 
    Using SQLite database provider.
    ToQueryString: .param set @_stock_0 99
    SELECT "c"."CategoryId", "c"."CategoryName", "c"."Description", 
    "t"."ProductId", "t"."CategoryId", "t"."UnitPrice", "t"."Discontinued", 
    "t"."ProductName", "t"."UnitsInStock"
    FROM "Categories" AS "c" 
    LEFT JOIN (
        SELECT "p"."ProductId", "p"."CategoryId", "p"."UnitPrice",
    "p"."Discontinued", "p"."ProductName", "p"."UnitsInStock" 
        FROM "Products" AS "p"
        WHERE ("p"."UnitsInStock" >= @_stock_0)
    ) AS "t" ON "c"."CategoryId" = "t"."CategoryId" 
    ORDER BY "c"."CategoryId", "t"."ProductId"
    Beverages has 2 products with a minimum of 99 units in stock.
      Sasquatch Ale has 111 units in stock. 
      Rhönbräu Klosterbier has 125 units in stock.
    ... 
    ```

注意名为`@_stock_0`的 SQL 参数已设置为最小库存值`99`。

对于 SQL Server，生成的 SQL 略有不同，例如，它使用方括号而不是双引号围绕对象名称，如下所示输出：

```cs
Enter a minimum for units in stock: 99
Using SqlServer database provider.
ToQueryString: DECLARE @__stock_0 smallint = CAST(99 AS smallint);
SELECT [c].[CategoryId], [c].[CategoryName], [c].[Description], [t].[ProductId], [t].[CategoryId], [t].[UnitPrice], [t].[Discontinued], [t].[ProductName], [t].[UnitsInStock]
FROM [Categories] AS [c]
LEFT JOIN (
    SELECT [p].[ProductId], [p].[CategoryId], [p].[UnitPrice], [p].[Discontinued], [p].[ProductName], [p].[UnitsInStock]
    FROM [Products] AS [p]
    WHERE [p].[UnitsInStock] >= @__stock_0
) AS [t] ON [c].[CategoryId] = [t].[CategoryId]
ORDER BY [c].[CategoryId], [t].[ProductId] 
```

## 使用自定义日志记录提供程序记录 EF Core

为了监控 EF Core 与数据库之间的交互，我们可以启用日志记录。这需要完成以下两个任务：

+   注册**日志记录提供程序**。

+   **日志记录器**的实现。

让我们看一个实际操作的示例：

1.  向项目中添加一个名为`ConsoleLogger.cs`的文件。

1.  修改文件以定义两个类，一个实现`ILoggerProvider`，另一个实现`ILogger`，如下所示代码，并注意以下事项：

    +   `ConsoleLoggerProvider`返回一个`ConsoleLogger`实例。它不需要任何非托管资源，因此`Dispose`方法无需执行任何操作，但必须存在。

    +   `ConsoleLogger`对于日志级别`None`、`Trace`和`Information`被禁用。对于所有其他日志级别均启用。

    +   `ConsoleLogger`通过向`Console`写入内容来实现其`Log`方法：

    ```cs
    using Microsoft.Extensions.Logging; // ILoggerProvider, ILogger, LogLevel
    using static System.Console;
    namespace Packt.Shared;
    public class ConsoleLoggerProvider : ILoggerProvider
    {
      public ILogger CreateLogger(string categoryName)
      {
        // we could have different logger implementations for
        // different categoryName values but we only have one
        return new ConsoleLogger();
      }
      // if your logger uses unmanaged resources,
      // then you can release them here
      public void Dispose() { }
    }
    public class ConsoleLogger : ILogger
    {
      // if your logger uses unmanaged resources, you can
      // return the class that implements IDisposable here
      public IDisposable BeginScope<TState>(TState state)
      {
        return null;
      }
      public bool IsEnabled(LogLevel logLevel)
      {
        // to avoid overlogging, you can filter on the log level
        switch(logLevel)
        {
          case LogLevel.Trace:
          case LogLevel.Information:
          case LogLevel.None:
            return false;
          case LogLevel.Debug:
          case LogLevel.Warning:
          case LogLevel.Error:
          case LogLevel.Critical:
          default:
            return true;
        };
      }
      public void Log<TState>(LogLevel logLevel,
        EventId eventId, TState state, Exception? exception,
        Func<TState, Exception, string> formatter)
      {
        // log the level and event identifier
        Write($"Level: {logLevel}, Event Id: {eventId.Id}");
        // only output the state or exception if it exists
        if (state != null)
        {
          Write($", State: {state}");
        }
        if (exception != null)
        {
          Write($", Exception: {exception.Message}");
        }
        WriteLine();
      }
    } 
    ```

1.  在`Program.cs`顶部，添加用于日志记录所需的命名空间导入语句，如下所示：

    ```cs
    using Microsoft.EntityFrameworkCore.Infrastructure;
    using Microsoft.Extensions.DependencyInjection; 
    using Microsoft.Extensions.Logging; 
    ```

1.  我们已经使用`ToQueryString`方法获取了`FilteredIncludes`的 SQL，因此无需向该方法添加日志记录。在`QueryingCategories`和`QueryingProducts`方法中，立即在`Northwind`数据库上下文的`using`块内添加语句以获取日志记录工厂并注册您的自定义控制台日志记录器，如下所示突出显示：

    ```cs
    using (Northwind db = new())
    {
     **ILoggerFactory loggerFactory = db.GetService<ILoggerFactory>();** 
     **loggerFactory.AddProvider(****new** **ConsoleLoggerProvider());** 
    ```

1.  在`Program.cs`顶部，注释掉对`FilteredIncludes`方法的调用，并取消注释对`QueryingProducts`方法的调用。

1.  运行代码并查看日志，部分输出如下所示：

    ```cs
    ...
    Level: Debug, Event Id: 20000, State: Opening connection to database 'main' on server '/Users/markjprice/Code/Chapter10/WorkingWithEFCore/Northwind.db'.
    Level: Debug, Event Id: 20001, State: Opened connection to database 'main' on server '/Users/markjprice/Code/Chapter10/WorkingWithEFCore/Northwind.db'.
    Level: Debug, Event Id: 20100, State: Executing DbCommand [Parameters=[@__price_0='?'], CommandType='Text', CommandTimeout='30']
    SELECT "p"."ProductId", "p"."CategoryId", "p"."UnitPrice", "p"."Discontinued", "p"."ProductName", "p"."UnitsInStock"
    FROM "Products" AS "p"
    WHERE "p"."UnitPrice" > @__price_0
    ORDER BY "product"."UnitPrice" DESC
    ... 
    ```

您的日志可能与上述显示的有所不同，这取决于您选择的数据库提供程序和代码编辑器，以及 EF Core 未来的改进。目前，请注意不同事件（如打开连接或执行命令）具有不同的事件 ID。

### 根据提供程序特定值过滤日志

事件 ID 值及其含义将特定于.NET 数据提供程序。如果我们想了解 LINQ 查询如何被转换为 SQL 语句并执行，则要输出的事件 ID 具有`Id`值`20100`：

1.  修改`ConsoleLogger`中的`Log`方法，仅输出具有`Id`为`20100`的事件，如下所示突出显示：

    ```cs
    public void Log<TState>(LogLevel logLevel, EventId eventId,
      TState state, Exception? exception,
      Func<TState, Exception, string> formatter)
    {
    **if** **(eventId.Id ==** **20100****)**
     **{**
        // log the level and event identifier
        Write("Level: {0}, Event Id: {1}, Event: {2}",
          logLevel, eventId.Id, eventId.Name);
        // only output the state or exception if it exists
        if (state != null)
        {
          Write($", State: {state}");
        }
        if (exception != null)
        {
          Write($", Exception: {exception.Message}");
        }
        WriteLine();
     **}**
    } 
    ```

1.  在`Program.cs`中，取消注释`QueryingCategories`方法并注释掉其他方法，以便我们可以监视在连接两个表时生成的 SQL 语句。

1.  运行代码，并注意已记录的以下 SQL 语句，如下所示输出已为空间编辑：

    ```cs
    Using SQLServer database provider.
    Categories and how many products they have:
    Level: Debug, Event Id: 20100, State: Executing DbCommand [Parameters=[], CommandType='Text', CommandTimeout='30']
    SELECT [c].[CategoryId], [c].[CategoryName], [c].[Description], [p].[ProductId], [p].[CategoryId], [p].[UnitPrice], [p].[Discontinued], [p].[ProductName], [p].[UnitsInStock]
    FROM [Categories] AS [c]
    LEFT JOIN [Products] AS [p] ON [c].[CategoryId] = [p].[CategoryId]
    ORDER BY [c].[CategoryId], [p].[ProductId]
    Beverages has 12 products.
    Condiments has 12 products.
    Confections has 13 products.
    Dairy Products has 10 products.
    Grains/Cereals has 7 products.
    Meat/Poultry has 6 products.
    Produce has 5 products.
    Seafood has 12 products. 
    ```

### 使用查询标签进行日志记录

记录 LINQ 查询时，在复杂场景中关联日志消息可能较为棘手。EF Core 2.2 引入了查询标签功能，通过允许你向日志添加 SQL 注释来提供帮助。

您可以使用`TagWith`方法注释 LINQ 查询，如下所示：

```cs
IQueryable<Product>? products = db.Products?
  .TagWith("Products filtered by price and sorted.")
  .Where(product => product.Cost > price)
  .OrderByDescending(product => product.Cost); 
```

这将在日志中添加一个 SQL 注释，如下所示：

```cs
-- Products filtered by price and sorted. 
```

## 使用 Like 进行模式匹配

EF Core 支持包括`Like`在内的常用 SQL 语句进行模式匹配：

1.  在`Program.cs`底部，添加一个名为`QueryingWithLike`的方法，如下所示，并注意：

    +   我们已启用日志记录。

    +   我们提示用户输入产品名称的一部分，然后使用`EF.Functions.Like`方法在`ProductName`属性中任意位置进行搜索。

    +   对于每个匹配的产品，我们输出其名称、库存以及是否已停产：

    ```cs
    static void QueryingWithLike()
    {
      using (Northwind db = new())
      {
        ILoggerFactory loggerFactory = db.GetService<ILoggerFactory>();
        loggerFactory.AddProvider(new ConsoleLoggerProvider());
        Write("Enter part of a product name: ");
        string? input = ReadLine();
        IQueryable<Product>? products = db.Products?
          .Where(p => EF.Functions.Like(p.ProductName, $"%{input}%"));
        if (products is null)
        {
          WriteLine("No products found.");
          return;
        }
        foreach (Product p in products)
        {
          WriteLine("{0} has {1} units in stock. Discontinued? {2}", 
            p.ProductName, p.Stock, p.Discontinued);
        }
      }
    } 
    ```

1.  在`Program.cs`中，注释掉现有方法，并调用`QueryingWithLike`。

1.  运行代码，输入部分产品名称如`che`，并查看结果，如下所示：

    ```cs
    Using SQLServer database provider.
    Enter part of a product name: che
    Level: Debug, Event Id: 20100, State: Executing DbCommand [Parameters=[@__Format_1='?' (Size = 40)], CommandType='Text', CommandTimeout='30']
    SELECT "p"."ProductId", "p"."CategoryId", "p"."UnitPrice",
    "p"."Discontinued", "p"."ProductName", "p"."UnitsInStock" FROM "Products" AS "p"
    WHERE "p"."ProductName" LIKE @__Format_1
    Chef Anton's Cajun Seasoning has 53 units in stock. Discontinued? False 
    Chef Anton's Gumbo Mix has 0 units in stock. Discontinued? True
    Queso Manchego La Pastora has 86 units in stock. Discontinued? False 
    Gumbär Gummibärchen has 15 units in stock. Discontinued? False 
    ```

EF Core 6.0 引入了另一个有用的函数`EF.Functions.Random`，它映射到数据库函数，返回 0 和 1 之间（不包括 1）的伪随机数。例如，您可以将随机数乘以表中的行数，以从该表中选择一行随机行。

## 定义全局过滤器

北风产品可能会停产，因此即使程序员不在查询中使用`Where`过滤它们，确保停产产品永远不会在结果中返回可能很有用：

1.  在`Northwind.cs`中，修改`OnModelCreating`方法，添加一个全局过滤器以移除停产产品，如下所示高亮部分：

    ```cs
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
      ...
    **// global filter to remove discontinued products**
     **modelBuilder.Entity<Product>()**
     **.HasQueryFilter(p => !p.Discontinued);**
    } 
    ```

1.  运行代码，输入部分产品名称`che`，查看结果，并注意**Chef Anton's Gumbo Mix**现已缺失，因为生成的 SQL 语句中包含了对`Discontinued`列的过滤，如下所示高亮部分：

    ```cs
    SELECT "p"."ProductId", "p"."CategoryId", "p"."UnitPrice",
    "p"."Discontinued", "p"."ProductName", "p"."UnitsInStock" 
    FROM "Products" AS "p"
    WHERE **("p"."Discontinued" = 0)** AND "p"."ProductName" LIKE @__Format_1 
    Chef Anton's Cajun Seasoning has 53 units in stock. Discontinued? False 
    Queso Manchego La Pastora has 86 units in stock. Discontinued? False 
    Gumbär Gummibärchen has 15 units in stock. Discontinued? False 
    ```

# EF Core 的加载模式

与 EF Core 一起常用的有三种加载模式：

+   **预先加载**：提前加载数据。

+   **延迟加载**：在数据即将被使用前自动加载。

+   **显式加载**：手动加载数据。

在本节中，我们将逐一介绍它们。

## 实体的预先加载

在`QueryingCategories`方法中，当前代码使用`Categories`属性遍历每个类别，输出类别名称以及该类别中的产品数量。

这样做是因为在编写查询时，我们通过调用`Include`方法为相关产品启用了预先加载。

让我们看看如果我们不调用`Include`会发生什么：

1.  修改查询，注释掉`Include`方法的调用，如下所示：

    ```cs
    IQueryable<Category>? categories =
      db.Categories; //.Include(c => c.Products); 
    ```

1.  在`Program.cs`中，注释掉除`QueryingCategories`之外的所有方法。

1.  运行代码并查看结果，如下所示部分输出：

    ```cs
    Beverages has 0 products. 
    Condiments has 0 products. 
    Confections has 0 products.
    Dairy Products has 0 products. 
    Grains/Cereals has 0 products. 
    Meat/Poultry has 0 products.
    Produce has 0 products. 
    Seafood has 0 products. 
    ```

`foreach`循环中的每一项都是`Category`类的一个实例，该类具有一个名为`Products`的属性，即该类别中的产品列表。由于原始查询仅从`Categories`表中选择，因此每个类别的此属性为空。

## 启用延迟加载

延迟加载是在 EF Core 2.1 中引入的，它可以自动加载缺失的相关数据。要启用延迟加载，开发者必须：

+   引用一个 NuGet 包用于代理。

+   配置延迟加载以使用代理。

让我们看看这是如何运作的：

1.  在`WorkingWithEFCore`项目中，添加一个 EF Core 代理的包引用，如下面的标记所示：

    ```cs
    <PackageReference
      Include="Microsoft.EntityFrameworkCore.Proxies" 
      Version="6.0.0" /> 
    ```

1.  构建项目以恢复包。

1.  打开`Northwind.cs`，并在`OnConfiguring`方法的顶部调用一个扩展方法以使用延迟加载代理，如下面的高亮代码所示：

    ```cs
    protected override void OnConfiguring(
      DbContextOptionsBuilder optionsBuilder)
    {
     **optionsBuilder.UseLazyLoadingProxies();** 
    ```

    现在，每当循环枚举时，尝试读取`Products`属性，延迟加载代理将检查它们是否已加载。如果没有，它将通过执行一个`SELECT`语句为我们“懒惰地”加载当前类别的那组产品，然后正确的计数将返回到输出。

1.  运行代码并注意产品计数现在已正确。但你会发现延迟加载的问题在于，为了最终获取所有数据，需要多次往返数据库服务器，如下面的部分输出所示：

    ```cs
    Categories and how many products they have:
    Level: Debug, Event Id: 20100, State: Executing DbCommand [Parameters=[], CommandType='Text', CommandTimeout='30']
    SELECT "c"."CategoryId", "c"."CategoryName", "c"."Description" FROM "Categories" AS "c"
    Level: Debug, Event Id: 20100, State: Executing DbCommand [Parameters=[@ p_0='?'], CommandType='Text', CommandTimeout='30'] 
    SELECT "p"."ProductId", "p"."CategoryId", "p"."UnitPrice",
    "p"."Discontinued", "p"."ProductName", "p"."UnitsInStock"
    FROM "Products" AS "p"
    WHERE ("p"."Discontinued" = 0) AND ("p"."CategoryId" = @ p_0) 
    Beverages has 11 products.
    Level: Debug, Event ID: 20100, State: Executing DbCommand [Parameters=[@ p_0='?'], CommandType='Text', CommandTimeout='30'] 
    SELECT "p"."ProductId", "p"."CategoryId", "p"."UnitPrice",
    "p"."Discontinued", "p"."ProductName", "p"."UnitsInStock"
    FROM "Products" AS "p"
    WHERE ("p"."Discontinued" = 0) AND ("p"."CategoryId" = @ p_0) 
    Condiments has 11 products. 
    ```

## 显式加载实体

另一种加载类型是显式加载。它的工作方式与延迟加载类似，区别在于你可以控制确切的相关数据何时加载：

1.  在`Program.cs`的顶部，导入更改跟踪命名空间，以便我们能够使用`CollectionEntry`类手动加载相关实体，如下面的代码所示：

    ```cs
    using Microsoft.EntityFrameworkCore.ChangeTracking; // CollectionEntry 
    ```

1.  在`QueryingCategories`方法中，修改语句以禁用延迟加载，然后提示用户是否希望启用预先加载和显式加载，如下面的代码所示：

    ```cs
    IQueryable<Category>? categories;
      // = db.Categories;
      // .Include(c => c.Products);
    db.ChangeTracker.LazyLoadingEnabled = false; 
    Write("Enable eager loading? (Y/N): ");
    bool eagerloading = (ReadKey().Key == ConsoleKey.Y); 
    bool explicitloading = false;
    WriteLine();
    if (eagerloading)
    {
      categories = db.Categories?.Include(c => c.Products);
    }
    else
    {
      categories = db.Categories;
      Write("Enable explicit loading? (Y/N): ");
      explicitloading = (ReadKey().Key == ConsoleKey.Y);
      WriteLine();
    } 
    ```

1.  在`foreach`循环中，在`WriteLine`方法调用之前，添加语句以检查是否启用了显式加载，如果是，则提示用户是否希望显式加载每个单独的类别，如下面的代码所示：

    ```cs
    if (explicitloading)
    {
      Write($"Explicitly load products for {c.CategoryName}? (Y/N): "); 
      ConsoleKeyInfo key = ReadKey();
      WriteLine();
      if (key.Key == ConsoleKey.Y)
      {
        CollectionEntry<Category, Product> products =
          db.Entry(c).Collection(c2 => c2.Products);
        if (!products.IsLoaded) products.Load();
      }
    }
    WriteLine($"{c.CategoryName} has {c.Products.Count} products."); 
    ```

1.  运行代码：

    1.  按下`N`以禁用预先加载。

    1.  然后按下`Y`以启用显式加载。

    1.  对于每个类别，按`Y`或`N`以按你的意愿加载其产品。

我选择只为八个类别中的两个——饮料和海鲜——加载产品，如下面的输出所示，为了节省空间已进行了编辑：

```cs
Categories and how many products they have:
Enable eager loading? (Y/N): n 
Enable explicit loading? (Y/N): y
Level: Debug, Event Id: 20100, State: Executing DbCommand [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT "c"."CategoryId", "c"."CategoryName", "c"."Description" FROM "Categories" AS "c"
Explicitly load products for Beverages? (Y/N): y
Level: Debug, Event Id: 20100, State: Executing DbCommand [Parameters=[@ p_0='?'], CommandType='Text', CommandTimeout='30'] 
SELECT "p"."ProductId", "p"."CategoryId", "p"."UnitPrice",
"p"."Discontinued", "p"."ProductName", "p"."UnitsInStock"
FROM "Products" AS "p"
WHERE ("p"."Discontinued" = 0) AND ("p"."CategoryId" = @ p_0)
Beverages has 11 products.
Explicitly load products for Condiments? (Y/N): n 
Condiments has 0 products.
Explicitly load products for Confections? (Y/N): n 
Confections has 0 products.
Explicitly load products for Dairy Products? (Y/N): n 
Dairy Products has 0 products.
Explicitly load products for Grains/Cereals? (Y/N): n 
Grains/Cereals has 0 products.
Explicitly load products for Meat/Poultry? (Y/N): n 
Meat/Poultry has 0 products.
Explicitly load products for Produce? (Y/N): n 
Produce has 0 products.
Explicitly load products for Seafood? (Y/N): y
Level: Debug, Event ID: 20100, State: Executing DbCommand [Parameters=[@ p_0='?'], CommandType='Text', CommandTimeout='30'] 
SELECT "p"."ProductId", "p"."CategoryId", "p"."UnitPrice",
"p"."Discontinued", "p"."ProductName", "p"."UnitsInStock"
FROM "Products" AS "p"
WHERE ("p"."Discontinued" = 0) AND ("p"."CategoryId" = @ p_0) 
Seafood has 12 products. 
```

**最佳实践**：仔细考虑哪种加载模式最适合你的代码。延迟加载可能会让你成为懒惰的数据库开发者！更多关于加载模式的信息，请访问以下链接：[`docs.microsoft.com/en-us/ef/core/querying/related-data`](https://docs.microsoft.com/en-us/ef/core/querying/related-data)

# 使用 EF Core 操作数据

使用 EF Core 插入、更新和删除实体是一个容易完成的任务。

`DbContext` 自动维护变更跟踪，因此本地实体可以有多个变更被跟踪，包括添加新实体、修改现有实体和删除实体。当你准备好将这些变更发送到底层数据库时，调用 `SaveChanges` 方法。成功变更的实体数量将被返回。

## 插入实体

让我们先来看看如何向表中添加新行：

1.  在 `Program.cs` 中，创建一个名为 `AddProduct` 的新方法，如下所示：

    ```cs
    static bool AddProduct(
      int categoryId, string productName, decimal? price)
    {
      using (Northwind db = new())
      {
        Product p = new()
        {
          CategoryId = categoryId,
          ProductName = productName,
          Cost = price
        };
        // mark product as added in change tracking
        db.Products.Add(p);
        // save tracked change to database
        int affected = db.SaveChanges();
        return (affected == 1);
      }
    } 
    ```

1.  在 `Program.cs` 中，创建一个名为 `ListProducts` 的新方法，该方法输出每个产品的 Id、名称、成本、库存和停产属性，并按成本最高排序，如下所示：

    ```cs
    static void ListProducts()
    {
      using (Northwind db = new())
      {
        WriteLine("{0,-3} {1,-35} {2,8} {3,5} {4}",
          "Id", "Product Name", "Cost", "Stock", "Disc.");
        foreach (Product p in db.Products
          .OrderByDescending(product => product.Cost))
        {
          WriteLine("{0:000} {1,-35} {2,8:$#,##0.00} {3,5} {4}",
            p.ProductId, p.ProductName, p.Cost, p.Stock, p.Discontinued);
        }
      }
    } 
    ```

    记住，`1,-35` 表示将参数 1 左对齐于一个 35 字符宽的列中，而 `3,5` 表示将参数 3 右对齐于一个 5 字符宽的列中。

1.  在 `Program.cs` 中，注释掉之前的方法调用，然后调用 `AddProduct` 和 `ListProducts`，如下所示：

    ```cs
    // QueryingCategories();
    // FilteredIncludes();
    // QueryingProducts();
    // QueryingWithLike();
    if (AddProduct(categoryId: 6, 
      productName: "Bob's Burgers", price: 500M))
    {
      WriteLine("Add product successful.");
    }
    ListProducts(); 
    ```

1.  运行代码，查看结果，并注意新添加的产品，如下所示：

    ```cs
    Add product successful.
    Id  Product Name              Cost Stock Disc.
    078 Bob's Burgers          $500.00       False
    038 Côte de Blaye          $263.50    17 False
    020 Sir Rodney's Marmalade  $81.00    40 False
    ... 
    ```

## 更新实体

现在，让我们修改表中的现有行：

1.  在 `Program.cs` 中，添加一个方法，将名称以指定值（在我们的例子中使用 Bob）开头的第一种产品的价格提高一个指定数额，比如 $20，如下所示：

    ```cs
    static bool IncreaseProductPrice(
      string productNameStartsWith, decimal amount)
    {
      using (Northwind db = new())
      {
        // get first product whose name starts with name
        Product updateProduct = db.Products.First(
          p => p.ProductName.StartsWith(productNameStartsWith));
        updateProduct.Cost += amount;
        int affected = db.SaveChanges();
        return (affected == 1);
      }
    } 
    ```

1.  在 `Program.cs` 中，注释掉整个调用 `AddProduct` 的 `if` 块，并在列出产品之前添加对 `IncreaseProductPrice` 的调用，如下所示：

    ```cs
    **/***
    if (AddProduct(categoryId: 6, 
      productName: "Bob's Burgers", price: 500M))
    {
      WriteLine("Add product successful.");
    }
    ***/**
    **if** **(IncreaseProductPrice(**
     **productNameStartsWith:** **"Bob"****, amount:** **20****M))**
    **{**
     **WriteLine(****"Update product price successful."****);**
    **}**
    ListProducts(); 
    ```

1.  运行代码，查看结果，并注意 Bob's Burgers 的现有实体价格已增加 $20，如下所示：

    ```cs
    Update product price successful.
    Id  Product Name              Cost Stock Disc.
    078 Bob's Burgers          $520.00       False
    038 Côte de Blaye          $263.50    17 False
    020 Sir Rodney's Marmalade  $81.00    40 False
    ... 
    ```

## 删除实体

你可以使用 `Remove` 方法移除单个实体。当你想要删除多个实体时，`RemoveRange` 更为高效。

现在让我们看看如何从表中删除行：

1.  在 `Program.cs` 底部，添加一个方法，删除名称以指定值（在我们的例子中是 Bob）开头的产品，如下所示：

    ```cs
    static int DeleteProducts(string productNameStartsWith)
    {
      using (Northwind db = new())
      {
        IQueryable<Product>? products = db.Products?.Where(
          p => p.ProductName.StartsWith(productNameStartsWith));
        if (products is null)
        {
          WriteLine("No products found to delete.");
          return 0;
        }
        else
        {
          db.Products.RemoveRange(products);
        }
        int affected = db.SaveChanges();
        return affected;
      }
    } 
    ```

1.  在 `Program.cs` 中，注释掉整个调用 `IncreaseProductPrice` 的 `if` 语句块，并添加对 `DeleteProducts` 的调用，如下所示：

    ```cs
    int deleted = DeleteProducts(productNameStartsWith: "Bob");
    WriteLine($"{deleted} product(s) were deleted."); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    1 product(s) were deleted. 
    ```

如果有多个产品名称以 Bob 开头，那么它们都会被删除。作为可选挑战，修改语句以添加三个以 Bob 开头的新产品，然后删除它们。

## 数据库上下文池化

`DbContext` 类是可释放的，并且是按照单一工作单元原则设计的。在前面的代码示例中，我们在一个 `using` 块中创建了所有派生自 `DbContext` 的 Northwind 实例，以便在工作单元结束时适当地调用 `Dispose`。

ASP.NET Core 的一个与 EF Core 相关的特性是，在构建网站和服务时，它通过池化数据库上下文使你的代码更高效。这允许你创建和销毁任意数量的`DbContext`派生对象，同时确保代码尽可能高效。

# **处理事务**

每次调用`SaveChanges`方法时，都会启动一个**隐式** **事务**，以便在出现问题时自动回滚所有更改。如果事务内的多个更改成功，则提交事务和所有更改。

事务通过应用锁来防止在更改序列发生时进行读写，从而维护数据库的完整性。

事务遵循**ACID**原则，这是一个缩写，其含义如下：

+   **原子性**意味着事务中的所有操作要么全部提交，要么全部不提交。

+   **一致性**意味着数据库在事务前后保持一致状态。这取决于你的代码逻辑；例如，在银行账户之间转账时，你的业务逻辑必须确保如果在一个账户中扣除$100，则在另一个账户中增加$100。

+   **隔离性**意味着在事务执行期间，其更改对其他进程是隐藏的。你可以从多个隔离级别中选择（参见下表）。隔离级别越高，数据完整性越好，但需要应用更多的锁，这可能会对其他进程产生负面影响。快照是一个特殊情况，因为它创建了多行副本以避免锁，但这会增加事务发生时数据库的大小。

+   **持久性**意味着如果在事务执行过程中发生故障，可以进行恢复。这通常通过两阶段提交和事务日志来实现。一旦事务被提交，即使后续出现错误，也能保证其持久性。与持久性相对的是易失性。

## **控制事务使用隔离级别**

开发人员可以通过设置**隔离级别**来控制事务，如下表所述：

| 隔离级别 | 锁 | 允许的完整性问题 |
| --- | --- | --- |
| `未提交读` | 无 | 脏读、不可重复读和幻像数据 |
| `已提交读` | 在编辑时，它应用读锁以阻止其他用户在事务结束前读取记录 | 不可重复读和幻像数据 |
| `可重复读` | 在读取时，它应用编辑锁以阻止其他用户在事务结束前编辑记录 | 幻像数据 |
| `可串行化` | 应用键范围锁以防止任何影响结果的操作，包括插入和删除 | 无 |
| `快照` | 无 | 无 |

## **定义显式事务**

你可以使用数据库上下文的`Database`属性来控制显式事务：

1.  在`Program.cs`中，导入 EF Core 存储命名空间以使用`IDbContextTransaction`接口，如下面的代码所示：

    ```cs
    using Microsoft.EntityFrameworkCore.Storage; // IDbContextTransaction 
    ```

1.  在`DeleteProducts`方法中，在`db`变量实例化后，添加语句以启动显式事务并输出其隔离级别。在方法底部，提交事务并关闭大括号，如下面的代码所示：

    ```cs
    static int DeleteProducts(string name)
    {
      using (Northwind db = new())
      {
    **using** **(IDbContextTransaction t = db.Database.BeginTransaction())**
     **{**
     **WriteLine(****"Transaction isolation level: {0}"****,**
     **arg0: t.GetDbTransaction().IsolationLevel);**
          IQueryable<Product>? products = db.Products?.Where(
            p => p.ProductName.StartsWith(name));
          if (products is null)
          {
            WriteLine("No products found to delete.");
            return 0;
          }
          else
          {
            db.Products.RemoveRange(products);
          }
          int affected = db.SaveChanges();
     **t.Commit();**
          return affected;
     **}**
      }
    } 
    ```

1.  运行代码并在 SQL Server 中查看结果，如下面的输出所示：

    ```cs
    Transaction isolation level: ReadCommitted 
    ```

1.  运行代码并在 SQLite 中查看结果，如下面的输出所示：

    ```cs
    Transaction isolation level: Serializable 
    ```

# Code First EF Core 模型

有时您可能没有现有数据库。相反，您将 EF Core 模型定义为 Code First，然后 EF Core 可以使用创建和删除 API 生成匹配的数据库。

**最佳实践**：创建和删除 API 仅应在开发期间使用。一旦发布应用程序，您不希望它删除生产数据库！

例如，我们可能需要为学院创建一个管理学生和课程的应用程序。一个学生可以注册参加多个课程。一个课程可以由多个学生参加。这是学生和课程之间多对多关系的一个例子。

让我们模拟这个示例：

1.  使用您喜欢的代码编辑器，在`Chapter10`解决方案/工作区中添加一个名为`CoursesAndStudents`的新控制台应用程序。

1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择。

1.  在 Visual Studio Code 中，选择`CoursesAndStudents`作为活动 OmniSharp 项目。

1.  在`CoursesAndStudents`项目中，添加以下包的包引用：

    +   `Microsoft.EntityFrameworkCore.Sqlite`

    +   `Microsoft.EntityFrameworkCore.SqlServer`

    +   `Microsoft.EntityFrameworkCore.Design`

1.  构建`CoursesAndStudents`项目以恢复包。

1.  添加名为`Academy.cs`、`Student.cs`和`Course.cs`的类。

1.  修改`Student.cs`，并注意它是一个没有属性装饰类的 POCO（普通旧 CLR 对象），如下面的代码所示：

    ```cs
    namespace CoursesAndStudents;
    public class Student
    {
      public int StudentId { get; set; }
      public string? FirstName { get; set; }
      public string? LastName { get; set; }
      public ICollection<Course>? Courses { get; set; }
    } 
    ```

1.  修改`Course.cs`，并注意我们已经用一些属性装饰了`Title`属性，以向模型提供更多信息，如下面的代码所示：

    ```cs
    using System.ComponentModel.DataAnnotations;
    namespace CoursesAndStudents;
    public class Course
    {
      public int CourseId { get; set; }
      [Required]
      [StringLength(60)]
      public string? Title { get; set; }
      public ICollection<Student>? Students { get; set; }
    } 
    ```

1.  修改`Academy.cs`，如下面的代码所示：

    ```cs
    using Microsoft.EntityFrameworkCore;
    using static System.Console;
    namespace CoursesAndStudents;
    public class Academy : DbContext
    {
      public DbSet<Student>? Students { get; set; }
      public DbSet<Course>? Courses { get; set; }
      protected override void OnConfiguring(
        DbContextOptionsBuilder optionsBuilder)
      {
        string path = Path.Combine(
          Environment.CurrentDirectory, "Academy.db");
        WriteLine($"Using {path} database file.");
        optionsBuilder.UseSqlite($"Filename={path}");
        // optionsBuilder.UseSqlServer(@"Data Source=.;Initial Catalog=Academy;Integrated Security=true;MultipleActiveResultSets=true;");
      }
      protected override void OnModelCreating(ModelBuilder modelBuilder)
      {
        // Fluent API validation rules
        modelBuilder.Entity<Student>()
            .Property(s => s.LastName).HasMaxLength(30).IsRequired();
          // populate database with sample data
          Student alice = new() { StudentId = 1, 
            FirstName = "Alice", LastName = "Jones" };
          Student bob = new() { StudentId = 2, 
            FirstName = "Bob", LastName = "Smith" };
          Student cecilia = new() { StudentId = 3, 
            FirstName = "Cecilia", LastName = "Ramirez" };
          Course csharp = new() 
          { 
            CourseId = 1,
            Title = "C# 10 and .NET 6", 
          };
          Course webdev = new()
          {
            CourseId = 2,
            Title = "Web Development",
          };
          Course python = new()
          {
            CourseId = 3,
            Title = "Python for Beginners",
          };
          modelBuilder.Entity<Student>()
            .HasData(alice, bob, cecilia);
          modelBuilder.Entity<Course>()
            .HasData(csharp, webdev, python);
          modelBuilder.Entity<Course>()
            .HasMany(c => c.Students)
            .WithMany(s => s.Courses)
            .UsingEntity(e => e.HasData(
              // all students signed up for C# course
              new { CoursesCourseId = 1, StudentsStudentId = 1 },
              new { CoursesCourseId = 1, StudentsStudentId = 2 },
              new { CoursesCourseId = 1, StudentsStudentId = 3 },
              // only Bob signed up for Web Dev
              new { CoursesCourseId = 2, StudentsStudentId = 2 },
              // only Cecilia signed up for Python
              new { CoursesCourseId = 3, StudentsStudentId = 3 }
            ));
      }
    } 
    ```

    **最佳实践**：使用匿名类型为多对多关系中的中间表提供数据。属性名称遵循命名约定`NavigationPropertyNamePropertyName`，例如，`Courses`是导航属性名称，`CourseId`是属性名称，因此`CoursesCourseId`将是匿名类型的属性名称。

1.  在`Program.cs`中，在文件顶部，导入 EF Core 和处理任务的命名空间，并静态导入`Console`，如下面的代码所示：

    ```cs
    using Microsoft.EntityFrameworkCore; // for GenerateCreateScript()
    using CoursesAndStudents; // Academy
    using static System.Console; 
    ```

1.  在`Program.cs`中，添加语句以创建`Academy`数据库上下文的实例，并使用它来删除现有数据库，根据模型创建数据库并输出它使用的 SQL 脚本，然后枚举学生及其课程，如下所示：

    ```cs
    using (Academy a = new())
    {
      bool deleted = await a.Database.EnsureDeletedAsync();
      WriteLine($"Database deleted: {deleted}");
      bool created = await a.Database.EnsureCreatedAsync();
      WriteLine($"Database created: {created}");
      WriteLine("SQL script used to create database:");
      WriteLine(a.Database.GenerateCreateScript());
      foreach (Student s in a.Students.Include(s => s.Courses))
      {
        WriteLine("{0} {1} attends the following {2} courses:",
          s.FirstName, s.LastName, s.Courses.Count);
        foreach (Course c in s.Courses)
        {
          WriteLine($"  {c.Title}");
        }
      }
    } 
    ```

1.  运行代码，并注意首次运行代码时无需删除数据库，因为它尚不存在，如下所示：

    ```cs
    Using C:\Code\Chapter10\CoursesAndStudents\bin\Debug\net6.0\Academy.db database file.
    Database deleted: False
    Database created: True
    SQL script used to create database:
    CREATE TABLE "Courses" (
        "CourseId" INTEGER NOT NULL CONSTRAINT "PK_Courses" PRIMARY KEY AUTOINCREMENT,
        "Title" TEXT NOT NULL
    );
    CREATE TABLE "Students" (
        "StudentId" INTEGER NOT NULL CONSTRAINT "PK_Students" PRIMARY KEY AUTOINCREMENT,
        "FirstName" TEXT NULL,
        "LastName" TEXT NOT NULL
    );
    CREATE TABLE "CourseStudent" (
        "CoursesCourseId" INTEGER NOT NULL,
        "StudentsStudentId" INTEGER NOT NULL,
        CONSTRAINT "PK_CourseStudent" PRIMARY KEY ("CoursesCourseId", "StudentsStudentId"),
        CONSTRAINT "FK_CourseStudent_Courses_CoursesCourseId" FOREIGN KEY ("CoursesCourseId") REFERENCES "Courses" ("CourseId") ON DELETE CASCADE,
        CONSTRAINT "FK_CourseStudent_Students_StudentsStudentId" FOREIGN KEY ("StudentsStudentId") REFERENCES "Students" ("StudentId") ON DELETE CASCADE
    );
    INSERT INTO "Courses" ("CourseId", "Title")
    VALUES (1, 'C# 10 and .NET 6');
    INSERT INTO "Courses" ("CourseId", "Title")
    VALUES (2, 'Web Development');
    INSERT INTO "Courses" ("CourseId", "Title")
    VALUES (3, 'Python for Beginners');
    INSERT INTO "Students" ("StudentId", "FirstName", "LastName")
    VALUES (1, 'Alice', 'Jones');
    INSERT INTO "Students" ("StudentId", "FirstName", "LastName")
    VALUES (2, 'Bob', 'Smith');
    INSERT INTO "Students" ("StudentId", "FirstName", "LastName")
    VALUES (3, 'Cecilia', 'Ramirez');
    INSERT INTO "CourseStudent" ("CoursesCourseId", "StudentsStudentId")
    VALUES (1, 1);
    INSERT INTO "CourseStudent" ("CoursesCourseId", "StudentsStudentId")
    VALUES (1, 2);
    INSERT INTO "CourseStudent" ("CoursesCourseId", "StudentsStudentId")
    VALUES (2, 2);
    INSERT INTO "CourseStudent" ("CoursesCourseId", "StudentsStudentId")
    VALUES (1, 3);
    INSERT INTO "CourseStudent" ("CoursesCourseId", "StudentsStudentId")
    VALUES (3, 3);
    CREATE INDEX "IX_CourseStudent_StudentsStudentId" ON "CourseStudent" ("StudentsStudentId");
    Alice Jones attends the following 1 course(s):
      C# 10 and .NET 6
    Bob Smith attends the following 2 course(s):
      C# 10 and .NET 6
      Web Development
    Cecilia Ramirez attends the following 2 course(s):
      C# 10 and .NET 6
      Python for Beginners 
    ```

    注意以下事项：

    +   `Title`列不可为空，因为模型被装饰了`[Required]`。

    +   `LastName`列不可为空，因为模型使用了`IsRequired()`。

    +   创建了一个名为`CourseStudent`的中间表，用于存储哪些学生参加了哪些课程的信息。

1.  使用 Visual Studio Server Explorer 或 SQLiteStudio 连接到`Academy`数据库并查看表格，如图 10.6 所示：![](img/B17442_11_09.png)

图 10.6：使用 Visual Studio 2022 Server Explorer 在 SQL Server 中查看 Academy 数据库

## 理解迁移

发布使用数据库的项目后，很可能稍后需要更改实体数据模型，从而改变数据库结构。届时，不应使用`Ensure`方法。相反，你需要使用一个系统，该系统允许你在保留数据库中任何现有数据的同时逐步更新数据库架构。EF Core 迁移就是这样的系统。

迁移很快会变得复杂，因此超出了本书的范围。你可以在以下链接中了解更多信息：[`docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/`](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/)

# 实践与探索

通过回答一些问题来测试你的知识和理解，进行一些实践练习，并深入研究本章的主题。

## 练习 10.1 – 测试你的知识

回答以下问题：

1.  对于表示表的属性，例如数据库上下文的`Products`属性，应使用哪种类型？

1.  对于表示一对多关系的属性，例如`Category`实体的`Products`属性，应使用哪种类型？

1.  EF Core 对主键的约定是什么？

1.  何时可能在实体类中使用注解属性？

1.  为何你可能更倾向于选择 Fluent API 而不是注解属性？

1.  事务隔离级别为`Serializable`意味着什么？

1.  `DbContext.SaveChanges()`方法返回什么？

1.  急切加载与显式加载之间有何区别？

1.  如何定义一个 EF Core 实体类以匹配以下表格？

    ```cs
    CREATE TABLE Employees(
      EmpId INT IDENTITY,
      FirstName NVARCHAR(40) NOT NULL,
      Salary MONEY
    ) 
    ```

1.  将实体导航属性声明为`virtual`有何好处？

## 练习 10.2 – 实践使用不同的序列化格式导出数据

在`Chapter10`解决方案/工作区中，创建一个名为`Exercise02`的控制台应用程序，该程序查询 Northwind 数据库中的所有类别和产品，并使用.NET 提供的至少三种序列化格式对数据进行序列化。哪种序列化格式使用的字节数最少？

## 练习 10.3 – 探索主题

使用以下页面上的链接，了解更多关于本章涵盖主题的详细信息：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-10---working-with-data-using-entity-framework-core`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-10---working-with-data-using-entity-framework-core)

## 练习 10.4 – 探索 NoSQL 数据库

本章重点介绍了 SQL Server 和 SQLite 等 RDBMS。如果你想了解更多关于 Cosmos DB 和 MongoDB 等 NoSQL 数据库的信息，以及如何使用它们与 EF Core，那么我推荐以下链接：

+   **欢迎来到 Azure Cosmos DB**：[`docs.microsoft.com/en-us/azure/cosmos-db/introduction`](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction)

+   **将 NoSQL 数据库用作持久性基础设施**：[`docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/nosql-database-persistence-infrastructure`](https://docs.microsoft.com/en-us/dotnet.standard/microservices-architecture/microservice-ddd-cqrs-patterns/nosql-database-persistence-infrastructure)

+   **Entity Framework Core 的文档数据库提供程序**：[`github.com/BlueshiftSoftware/EntityFrameworkCore`](https://github.com/BlueshiftSoftware/EntityFrameworkCore)

# 总结

在本章中，你学习了如何连接到现有数据库，如何执行简单的 LINQ 查询并处理结果，如何使用过滤的包含，如何添加、修改和删除数据，以及如何为现有数据库（如 Northwind）构建实体数据模型。你还学习了如何定义 Code First 模型，并使用它创建新数据库并填充数据。

在下一章中，你将学习如何编写更高级的 LINQ 查询，以进行选择、过滤、排序、连接和分组。
