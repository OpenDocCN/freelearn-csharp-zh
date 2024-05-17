# 第十章：使用 Entity Framework Core 处理数据

本章介绍了如何使用名为**Entity Framework Core**（**EF Core**）的对象到数据存储映射技术，读取和写入数据存储，如 Microsoft SQL Server、SQLite 和 Azure Cosmos DB。

本章将涵盖以下主题：

+   理解现代数据库

+   设置 EF Core

+   定义 EF Core 模型

+   查询 EF Core 模型

+   使用 EF Core 加载模式

+   使用 EF Core 操作数据

+   处理事务

+   Code First EF Core 模型

# 理解现代数据库

存储数据的两个最常见的地方是**关系数据库管理系统**（**RDBMS**），如 Microsoft SQL Server、PostgreSQL、MySQL 和 SQLite，或者**NoSQL**数据库，如 Microsoft Azure Cosmos DB、Redis、MongoDB 和 Apache Cassandra。

## 理解传统的 Entity Framework

**Entity Framework**（**EF**）最初是作为.NET Framework 3.5 与 Service Pack 1 的一部分发布的，那是在 2008 年末。自那时起，Entity Framework 已经发展，因为微软观察到程序员在现实世界中如何使用**对象关系映射**（**ORM**）工具。

ORM 使用映射定义将表中的列与类中的属性关联起来。然后，程序员可以以熟悉的方式与不同类型的对象交互，而不必知道如何将值存储在关系表或 NoSQL 数据存储提供的其他结构中。

包含在.NET Framework 中的 EF 版本是**Entity Framework 6**（**EF6**）。它是成熟、稳定的，支持使用 EDMX（XML 文件）定义模型以及复杂的继承模型等一些其他高级功能。

EF 6.3 及更高版本已从.NET Framework 中提取为一个单独的包，因此可以在.NET Core 3.0 及更高版本上得到支持。这使得像 Web 应用程序和服务这样的现有项目可以被移植并跨平台运行。但是，EF6 应该被视为传统技术，因为在跨平台运行时存在一些限制，并且不会添加新功能。

### 使用传统的 Entity Framework 6.3 或更高版本

要在.NET Core 3.0 或更高版本项目中使用传统的 Entity Framework，必须在项目文件中添加对它的包引用，如下面的标记所示：

```cs

<PackageReference Include="EntityFramework"

Version="6.4.4"

/>

```

**最佳实践**：只有在必要时才使用传统的 EF6，例如在迁移使用它的 WPF 应用程序时。本书是关于现代跨平台开发的，因此在本章的其余部分中，我将只涵盖现代的 Entity Framework Core。您不需要像上面所示的项目中引用传统的 EF6 包。

## 理解 Entity Framework Core

真正跨平台的版本**EF Core**与传统的 Entity Framework 不同。尽管 EF Core 的名称相似，但您应该了解它与 EF6 有何不同。最新的 EF Core 版本是 6.0，以匹配.NET 6.0。

EF Core 5 及更高版本仅支持.NET 5 及更高版本。EF Core 3.0 及更高版本仅在支持.NET Standard 2.1 的平台上运行，这意味着.NET Core 3.0 及更高版本。它不支持.NET Standard 2.0 平台，如.NET Framework 4.8。

除了传统的关系型数据库管理系统，EF Core 还支持现代基于云的非关系型无模式数据存储，如 Microsoft Azure Cosmos DB 和 MongoDB，有时还包括第三方提供程序。

EF Core 有很多改进，本章无法涵盖所有内容。我将重点关注所有.NET 开发人员都应该了解的基础知识以及一些更酷的新功能。

使用 EF Core 有两种方法：

1.  **Database First**：数据库已经存在，因此您可以构建一个与其结构和特性匹配的模型。

1.  **Code First**：没有数据库存在，因此您可以构建一个模型，然后使用 EF Core 创建与其结构和特性匹配的数据库。

我们将首先使用 EF Core 与现有数据库。

## 创建一个用于使用 EF Core 的控制台应用程序

首先，我们将为本章创建一个控制台应用程序项目：

1.  使用您喜欢的代码编辑器创建一个名为`Chapter10`的新解决方案/工作空间。

1.  添加一个控制台应用程序项目，如下列表所定义：

1.  项目模板：**控制台应用程序** / `console`

1.  工作空间/解决方案文件和文件夹：`Chapter10`

1.  项目文件和文件夹：`WorkingWithEFCore`

## 使用样本关系数据库

要学习如何使用.NET 管理 RDBMS，有一个样本数据库会很有用，这样您就可以在一个中等复杂度和一定数量的样本记录上进行练习。微软提供了几个样本数据库，其中大多数对我们的需求来说都太复杂了，因此我们将使用一个在上世纪 90 年代初首次创建的数据库，即**Northwind**。

让我们花一分钟时间看一下 Northwind 数据库的图表。您可以在本书的编写代码和查询过程中使用以下图表作为参考：

![自动生成的图表描述](img/Image00089.jpg)

图 10.1：Northwind 数据库表和关系

您将在本章后面的部分编写代码来处理`Categories`和`Products`表，以及后面章节中的其他表。但在我们开始之前，请注意：

+   每个类别都有一个唯一的标识符、名称、描述和图片。

+   每个产品都有一个唯一的标识符、名称、单价、库存单位和其他字段。

+   每个产品都通过存储类别的唯一标识符与一个类别相关联。

+   `Categories`和`Products`之间的关系是一对多的，意味着每个类别可以有零个或多个产品。

## 使用 Microsoft SQL Server for Windows

微软为 Windows、Linux 和 Docker 容器提供了其流行和功能强大的 SQL Server 产品的各种版本。我们将使用一个可以独立运行的免费版本，称为 SQL Server 开发人员版。您还可以使用 Express 版或可以与 Visual Studio for Windows 一起安装的免费 SQL Server LocalDB 版。

如果您没有 Windows 计算机，或者想要使用跨平台数据库系统，那么您可以直接跳到*使用 SQLite*主题。

### 下载并安装 SQL Server

您可以从以下链接下载 SQL Server 版本：

[`www.microsoft.com/en-us/sql-server/sql-server-downloads`](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)

1.  下载**开发人员**版。

1.  运行安装程序。

1.  选择**自定义**安装类型。

1.  选择一个安装文件的文件夹，然后点击**安装**。

1.  等待 1.5GB 的安装程序文件下载完成。

1.  在**SQL Server 安装中心**中，点击**安装**，然后点击**新建 SQL Server 独立安装或向现有安装添加功能**。

1.  选择**开发人员**作为免费版本，然后点击**下一步**。

1.  接受许可条款，然后点击**下一步**。

1.  查看安装规则，修复任何问题，然后点击**下一步**。

1.  在**功能选择**中，选择**数据库引擎服务**，然后点击**下一步**。

1.  在**实例配置**中，选择**默认实例**，然后点击**下一步**。如果您已经配置了默认实例，那么您可以创建一个命名实例，可能称为`cs10dotnet6`。

1.  在**服务器配置**中，注意**SQL Server** **数据库引擎**已配置为自动启动。将**SQL Server Browser**设置为自动启动，然后点击**下一步**。

1.  在**数据库引擎配置**中，在**服务器配置**选项卡上，将**身份验证模式**设置为**混合**，将**sa**账户密码设置为强密码，点击**添加当前用户**，然后点击**下一步**。

1.  在**准备安装**中，查看将要采取的操作，然后点击**安装**。

1.  在**完成**中，注意已经采取的成功操作，然后点击**关闭**。

1.  在**SQL Server 安装中心**中，在**安装**中，点击**安装 SQL Server 管理工具**。

1.  在浏览器窗口中，单击下载 SSMS 的最新版本。

1.  运行安装程序，然后单击**安装**。

1.  安装程序完成后，如果需要，单击**重新启动**或**关闭**。

## 为 SQL Server 创建 Northwind 示例数据库

现在我们可以运行数据库脚本来创建 Northwind 示例数据库：

1.  如果您以前没有下载或克隆过本书的 GitHub 存储库，请使用以下链接进行下载：[`github.com/markjprice/cs10dotnet6/`](https://github.com/markjprice/cs10dotnet6/)。

1.  将从本地 Git 存储库中的以下路径`/sql-scripts/Northwind4SQLServer.sql`中的脚本复制到`WorkingWithEFCore`文件夹中。

1.  启动**SQL Server 管理工具**。

1.  在**连接到服务器**对话框中，对于**服务器名称**，输入`。`（一个点），表示本地计算机名称，然后单击**连接**。

如果您必须创建一个命名实例，比如`cs10dotnet6`，那么输入`.\cs10dotnet6`

1.  导航到**文件** | **打开** | **文件...**。

1.  浏览以选择`Northwind4SQLServer.sql`文件，然后单击**打开**。

1.  在工具栏中，单击**执行**，注意**命令已成功完成**的消息。

1.  在**对象资源管理器**中，展开**Northwind**数据库，然后展开**表**。

1.  右键单击**产品**，单击**选择前 1000 行**，注意返回的结果，如*图 10.2*所示：![](img/Image00090.jpg)

图 10.2：SQL Server 管理工具中的产品表

1.  在**对象资源管理器**工具栏中，单击**断开**按钮。

1.  退出 SQL Server 管理工具。

## 使用服务器资源管理器管理 Northwind 示例数据库

我们不必使用 SQL Server 管理工具来执行数据库脚本。我们还可以使用 Visual Studio 中的工具，包括**SQL Server 对象资源管理器**和**服务器资源管理器**：

1.  在 Visual Studio 中，选择**查看** | **服务器资源管理器**。

1.  在**服务器资源管理器**窗口中，右键单击**数据连接**，然后选择**添加连接...**。

1.  如果您看到**选择数据源**对话框，如*图 10.3*所示，请选择**Microsoft SQL Server**，然后单击**继续**：![图形用户界面，应用程序描述自动生成](img/Image00091.jpg)

图 10.3：选择 SQL Server 作为数据源

1.  在**添加连接**对话框中，将服务器名称输入为`。`，将数据库名称输入为`Northwind`，然后单击**确定**。

1.  在**服务器资源管理器**中，展开数据连接及其表。您应该看到 13 个表，包括**类别**和**产品**表。

1.  右键单击**产品**表，选择**显示表数据**，注意返回了 77 行产品。

1.  要查看**产品**表的列和类型的详细信息，请右键单击**产品**，然后选择**打开表定义**，或者在**服务器资源管理器**中双击表。

## 使用 SQLite

SQLite 是一个小型、跨平台、自包含的关系型数据库管理系统，可在公共领域中使用。它是移动平台（如 iOS（iPhone 和 iPad）和 Android）最常见的关系型数据库管理系统。即使您使用 Windows 并在上一节中设置了 SQL Server，您可能也想设置 SQLite。我们编写的代码将适用于两者，并且看到微妙的差异可能会很有趣。

### 为 macOS 设置 SQLite

在 macOS 中，SQLite 包含在`/usr/bin/`目录中，作为一个名为`sqlite3`的命令行应用程序。

### 为 Windows 设置 SQLite

在 Windows 上，我们需要将 SQLite 的文件夹添加到系统路径中，这样当我们在命令提示符或终端中输入命令时，它就会被找到：

1.  启动您喜欢的浏览器，并导航到以下链接：[`www.sqlite.org/download.html`](https://www.sqlite.org/download.html)。

1.  向下滚动页面到**Windows 的预编译二进制文件**部分。

1.  单击**sqlite-tools-win32-x86-3360000.zip**。请注意，该文件在本书出版后可能具有更高的版本号。

1.  将 ZIP 文件解压缩到名为`C:\Sqlite\`的文件夹中。

1.  导航到**Windows 设置**。

1.  搜索`environment`并选择**编辑系统环境变量**。在 Windows 的非英文版本中，请搜索本地语言中的等效词以找到设置。

1.  点击**环境变量**按钮。

1.  在**系统变量**中，选择列表中的**Path**，然后点击**编辑…**。

1.  点击**新建**，输入`C:\Sqlite`，然后按 Enter。

1.  点击**确定**。

1.  点击**确定**。

1.  点击**确定**。

1.  关闭**Windows 设置**。

### 为其他操作系统设置 SQLite

SQLite 可以从以下链接下载并安装到其他操作系统：[`www.sqlite.org/download.html`](https://www.sqlite.org/download.html)。

## 为 SQLite 创建 Northwind 示例数据库

现在我们可以使用 SQL 脚本为 SQLite 创建 Northwind 示例数据库：

1.  如果您之前没有克隆过本书的 GitHub 存储库，请使用以下链接进行克隆：[`github.com/markjprice/cs10dotnet6/`](https://github.com/markjprice/cs10dotnet6/)。

1.  从本地 Git 存储库中的以下路径复制用于 SQLite 创建 Northwind 数据库的脚本：`/sql-scripts/Northwind4SQLite.sql`，并将其粘贴到`WorkingWithEFCore`文件夹中。

1.  在`WorkingWithEFCore`文件夹中启动命令行：

1.  在 Windows 上，启动**文件资源管理器**，右键单击`WorkingWithEFCore`文件夹，然后选择**在文件夹中新建命令提示符**或**在 Windows 终端中打开**。

1.  在 macOS 上，启动**Finder**，右键单击`WorkingWithEFCore`文件夹，然后选择**在文件夹中新建终端**。

1.  输入以下命令来执行使用 SQLite 创建`Northwind.db`数据库的 SQL 脚本，如下命令所示：

```cs

sqlite3 Northwind.db -init Northwind4SQLite.sql

```

1.  请耐心等待，因为这个命令可能需要一段时间来创建数据库结构。最终，您将看到 SQLite 命令提示符，如下输出所示：

```cs

-- 从 Northwind4SQLite.sql 加载资源

SQLite 版本 3.36.0 2021-08-24 15:20:15

输入".help"以获取使用提示。

sqlite>

```

1.  在 Windows 上按 Ctrl + C 或在 macOS 上按 Ctrl + D 退出 SQLite 命令模式。

1.  保持终端或命令提示符窗口打开，因为您很快将再次使用它。

## 使用 SQLiteStudio 管理 Northwind 示例数据库

您可以使用名为**SQLiteStudio**的跨平台图形数据库管理器轻松管理 SQLite 数据库：

1.  导航到以下链接，[`sqlitestudio.pl`](https://sqlitestudio.pl)，并下载并解压应用程序到您喜欢的位置。

1.  启动**SQLiteStudio**。

1.  在**数据库**菜单中，选择**添加数据库**。

1.  在**数据库**对话框中，在**文件**部分，点击黄色文件夹按钮以浏览本地计算机上现有数据库文件，选择`WorkingWithEFCore`文件夹中的`Northwind.db`文件，然后点击**确定**。

1.  右键单击**Northwind**数据库，然后选择**连接到数据库**。您将看到脚本创建的 10 个表。（与 SQL Server 的脚本相比，SQLite 的脚本更简单，它不会创建太多的表或其他数据库对象。）

1.  右键单击**Products**表，然后选择**编辑表**。

1.  在表编辑器窗口中，注意`Products`表的结构，包括列名、数据类型、键和约束，如*图 10.4*所示：![图形用户界面，文本，应用程序描述自动生成](img/Image00092.jpg)

图 10.4：SQLiteStudio 中的表编辑器显示了 Products 表的结构

1.  在表编辑器窗口中，点击**数据**选项卡，您将看到 77 个产品，如*图 10.5*所示：![图形用户界面，文本，应用程序描述自动生成](img/Image00093.jpg)

图 10.5：数据选项卡显示了 Products 表中的行

1.  在**数据库**窗口中，右键单击**Northwind**，然后选择**断开与数据库的连接**。

1.  退出 SQLiteStudio。

# 设置 EF Core

在我们深入讨论使用 EF Core 管理数据的实际操作之前，让我们简要讨论一下在 EF Core 数据提供程序之间进行选择。

## 选择 EF Core 数据库提供程序

要管理特定数据库中的数据，我们需要知道如何有效地与该数据库通信的类。

EF Core 数据库提供程序是一组针对特定数据存储优化的类。甚至有一个提供程序用于将数据存储在当前进程的内存中，这对于高性能单元测试可能很有用，因为它避免了对外部系统的访问。

它们以 NuGet 软件包的形式分发，如下表所示：

| 要管理此数据存储 | 安装此 NuGet 软件包 |
| --- | --- |
| Microsoft SQL Server 2012 或更高版本 | `Microsoft.EntityFrameworkCore.SqlServer` |
| SQLite 3.7 或更高版本 | `Microsoft.EntityFrameworkCore.SQLite` |
| MySQL | `MySQL.Data.EntityFrameworkCore` |
| 内存中 | `Microsoft.EntityFrameworkCore.InMemory` |
| Azure Cosmos DB SQL API | `Microsoft.EntityFrameworkCore.Cosmos` |
| Oracle DB 11.2 | `Oracle.EntityFrameworkCore` |

您可以在同一个项目中安装尽可能多的 EF Core 数据库提供程序。每个软件包都包括共享类型和特定于提供程序的类型。

## 连接到数据库

要连接到 SQLite 数据库，我们只需要知道数据库文件名，使用参数`Filename`设置。

要连接到 SQL Server 数据库，我们需要知道多个信息，如下列表所示：

+   服务器的名称（如果有实例，则包括实例）。

+   数据库的名称。

+   安全信息，例如用户名和密码，或者是否应自动传递当前登录用户的凭据。

我们在**连接字符串**中指定这些信息。

为了向后兼容，我们可以在 SQL Server 连接字符串中使用多个可能的关键字来表示各种参数，如下列表所示：

+   `Data Source`或`server`或`addr`：这些关键字是服务器的名称（和可选实例）。您可以使用点`.`表示本地服务器。

+   `Initial Catalog`或`database`：这些关键字是数据库的名称。

+   `集成安全性`或`trusted_connection`：这些关键字设置为`true`或`SSPI`以传递线程的当前用户凭据。

+   `MultipleActiveResultSets`：此关键字设置为`true`以允许单个连接同时与多个表一起工作，以提高效率。它用于从相关表中延迟加载行。

如上列表所述，当您编写连接到 SQL Server 数据库的代码时，您需要知道其服务器名称。服务器名称取决于您将连接到的 SQL Server 的版本和版本，如下表所示：

| SQL Server 版本 | 服务器名称\实例名称 |
| --- | --- |
| LocalDB 2012 | `(localdb)\v11.0` |
| LocalDB 2016 或更高版本 | `(localdb)\mssqllocaldb` |
| Express | `.\sqlexpress` |
| 完整/开发人员（默认实例） | `.` |
| 完整/开发人员（命名实例） | `.\cs10dotnet6` |

**良好实践**：使用点`.`作为本地计算机名称的缩写。请记住，SQL Server 的服务器名称由两部分组成：计算机的名称和 SQL Server 实例的名称。您可以在自定义安装期间提供实例名称。

## 定义 Northwind 数据库上下文类

`Northwind`类将用于表示数据库。要使用 EF Core，该类必须继承自`DbContext`。该类了解如何与数据库通信，并动态生成 SQL 语句以查询和操作数据。

您的`DbContext`-派生类应该有一个名为`OnConfiguring`的重写方法，该方法将设置数据库连接字符串。

为了让您轻松尝试 SQLite 和 SQL Server，我们将创建一个支持两者的项目，其中包含一个`string`字段，用于在运行时控制使用哪个：

1.  在`WorkingWithEFCore`项目中，添加对 SQL Server 和 SQLite 的 EF Core 数据提供程序的包引用，如下标记所示：

```cs

<ItemGroup>

<PackageReference

Include="Microsoft.EntityFrameworkCore.Sqlite"

Version="6.0.0"

/>

<PackageReference

Include="Microsoft.EntityFrameworkCore.SqlServer"

Version="6.0.0"

/>

</ItemGroup>

```

1.  构建项目以恢复包。

1.  添加一个名为`ProjectConstants.cs`的类文件。

1.  在`ProjectConstants.cs`中，定义一个带有公共字符串常量的类，用于存储要使用的数据库提供程序名称，如下所示：

```cs

命名空间

Packt.Shared

;

public

类

ProjectConstants

{

public

const

字符串

DatabaseProvider = "SQLite"

; // 或"SQLServer"

}

```

1.  在`Program.cs`中，导入`Packt.Shared`命名空间，并输出数据库提供程序，如下所示：

```cs

使用

{ProjectConstants.DatabaseProvider}

数据库提供程序。"

);

```

1.  添加一个名为`Northwind.cs`的类文件。

1.  在`Northwind.cs`中，定义一个名为`Northwind`的类，导入 EF Core 的主要命名空间，使该类继承自`DbContext`，并在`OnConfiguring`方法中，检查`provider`字段以使用 SQLite 或 SQL Server，如下所示：

```cs

使用

Microsoft.EntityFrameworkCore; // DbContext, DbContextOptionsBuilder

使用

静态

System.Console;

命名空间

Packt.Shared

;

// 这管理对数据库的连接

public

类

Northwind

: DbContext

{

protected

覆盖

void

OnConfiguring

(

DbContextOptionsBuilder optionsBuilder

)

{

if

(ProjectConstants.DatabaseProvider == "SQLite"

)

{

字符串

路径 = Path.Combine(

Environment.CurrentDirectory, "Northwind.db"

);

使用

{路径}

数据库文件。"

);

optionsBuilder.UseSqlite($"Filename=

{路径}

"

);

}

else

{

字符串

连接 = "Data Source=.;"

+

"Initial Catalog=Northwind;"

+

"Integrated Security=true;"

+

"MultipleActiveResultSets=true;"

;

optionsBuilder.UseSqlServer(connection);

}

}

}

```

如果您正在使用 Windows 的 Visual Studio，则编译的应用程序将在`WorkingWithEFCore\bin\Debug\net6.0`文件夹中执行，因此它将无法找到数据库文件。

1.  在**解决方案资源管理器**中，右键单击`Northwind.db`文件，然后选择**属性**。

1.  在**属性**中，将**复制到输出目录**设置为**始终复制**。

1.  打开`WorkingWithEFCore.csproj`，注意新元素，如下标记所示：

```cs

<ItemGroup>

<None Update="Northwind.db"

<CopyToOutputDirectory>Always</CopyToOutputDirectory>

</None>

</ItemGroup>

```

如果您正在使用 Visual Studio Code，则编译的应用程序将在`WorkingWithEFCore`文件夹中执行，因此它将在不复制的情况下找到数据库文件。

1.  运行控制台应用程序，并注意输出显示您选择使用的数据库提供程序。

# 定义 EF Core 模型

EF Core 使用**约定**、**注释属性**和**Fluent API**语句的组合在运行时构建实体模型，以便对类执行的任何操作后来可以自动转换为对实际数据库执行的操作。实体类表示表的结构，类的实例表示该表中的一行。

首先，我们将回顾三种定义模型的方法，附带代码示例，然后我们将创建一些实现这些技术的类。

## 使用 EF Core 约定来定义模型

我们将编写的代码将使用以下约定：

+   表的名称假定与`DbContext`类中的`DbSet<T>`属性的名称匹配，例如`Products`。

+   列的名称假定与实体模型类中的属性名称匹配，例如`ProductId`。

+   假定`string` .NET 类型在数据库中是`nvarchar`类型。

+   假定`int` .NET 类型在数据库中是`int`类型。

+   主键被假定为命名为`Id`或`ID`的属性，或者当实体模型类命名为`Product`时，属性可以命名为`ProductId`或`ProductID`。如果此属性是整数类型或`Guid`类型，则还假定它是一个`IDENTITY`列（在插入时自动分配值的列类型）。

**良好实践**：还有许多其他约定，您可以甚至定义自己的约定，但这超出了本书的范围。您可以在以下链接阅读有关它们的信息：[`docs.microsoft.com/en-us/ef/core/modeling/`](https://docs.microsoft.com/en-us/ef/core/modeling/)

## 使用 EF Core 注释属性定义模型

常规通常不足以完全将类映射到数据库对象。向模型添加更多智能的简单方法是应用注释属性。

一些常见的属性如下表所示：

| 属性 | 描述 |
| --- | --- |
| `[Required]` | 确保值不为`null`。 |
| `[StringLength(50)]` | 确保值的长度最多为 50 个字符。 |
| `[RegularExpression(expression)]` | 确保值与指定的正则表达式匹配。 |
| `[Column(TypeName = "money", Name = "UnitPrice")]` | 指定表中使用的列类型和列名。 |

例如，在数据库中，产品名称的最大长度为 40，且值不能为空，如下所示在以下**数据定义语言**（**DDL**）代码中突出显示的定义了如何创建名为`Products`的表及其列、数据类型、键和其他约束：

```cs

创建

表

产品 (

产品编号 整数

主键

,

产品名称 NVARCHAR

(40

) 不

空

,

供应商编号 "INT"

,

类别编号 "INT"

,

每单位数量 NVARCHAR

(20

),

单价 "货币"

约束

DF_Products_UnitPrice 默认

(0

),

库存 "SMALLINT"

约束

DF_Products_UnitsInStock 默认

(0

),

订购单位 "SMALLINT"

约束

DF_Products_UnitsOnOrder 默认

(0

),

订货水平 "SMALLINT"

约束

DF_Products_ReorderLevel 默认

(0

),

已停产 "位"

不

空

约束

DF_Products_Discontinued 默认

(0

),

约束

FK_Products_Categories 外键

键

(

类别编号

)

引用

类别 (类别编号),

约束

FK_Products_Suppliers 外键

键

(

供应商编号

)

引用

供应商 (供应商编号),

约束

CK_Products_UnitPrice 检查

(单价 >= 0

),

约束

CK_ReorderLevel 检查

(订货水平 >= 0

),

约束

CK_UnitsInStock 检查

(库存单位 >= 0

),

约束

CK_UnitsOnOrder 检查

(订购单位 >= 0

)

);

```

在`Product`类中，我们可以应用属性来指定这一点，如下代码所示：

```cs

必需

]

[StringLength(40)

]

公共

字符串

产品名称 { 获取

; 设置

; }

```

当.NET 类型和数据库类型之间没有明显的映射时，可以使用属性。

例如，在数据库中，`Products`表的`UnitPrice`列类型为`money`。.NET 没有`money`类型，因此应改用`decimal`，如下代码所示：

```cs

[Column(TypeName =

"money"

)

]

公共

小数

? 单价 { 获取

; 设置

; }

```

另一个例子是`Categories`表，如下 DDL 代码所示：

```cs

创建

表

类别 (

类别 ID 整数

主键

,

类别名称 NVARCHAR

(15

) 不

空

,

描述 "NTEXT"

,

图片 "图像"

);

```

“描述”列的长度可以超过存储在`nvarchar`变量中的最大 8000 个字符，因此需要映射到`ntext`，如下代码所示：

```cs

[Column(TypeName =

"ntext"

)

]

公共

字符串

描述 { 获取

; 设置

; }

```

## 使用 EF Core Fluent API 定义模型

定义模型的最后一种方法是使用 Fluent API。 此 API 可以用于替代属性，也可以与属性一起使用。 例如，要定义`ProductName`属性，可以在数据库上下文类的`OnModelCreating`方法中编写等效的 Fluent API 语句，而不是用两个属性装饰属性，如下所示的代码：

```cs

modelBuilder.Entity<Product>()

.Property(product => product.ProductName)

.IsRequired()

.HasMaxLength(40

);

```

这使实体模型类更简单。

### 使用 Fluent API 理解数据种植

Fluent API 的另一个好处是提供初始数据以填充数据库。 EF Core 会自动计算必须执行的插入，更新或删除操作。

例如，如果我们想确保新数据库中的`Product`表至少有一行，则我们将调用`HasData`方法，如下所示的代码：

```cs

modelBuilder.Entity<Product>()

.HasData(new

产品

{

ProductId = 1

，

ProductName = "Chai"

，

UnitPrice = 8.99

M

});

```

我们的模型将映射到已经填充了数据的现有数据库，因此我们不需要在我们的代码中使用这种技术。

## 为`Northwind`表构建 EF Core 模型

现在您已经了解了定义 EF Core 模型的方法，让我们构建一个模型来表示`Northwind`数据库中的两个表。

这两个实体类将相互引用，因此为了避免编译器错误，我们将首先创建没有任何成员的类：

1.  在`WorkingWithEFCore`项目中，添加两个名为`Category.cs`和`Product.cs`的类文件。

1.  在`Category.cs`中，定义一个名为`Category`的类，如下所示的代码：

```cs

命名空间

Packt.Shared

;

公共

类

类别

{

}

```

1.  在`Product.cs`中，定义一个名为`Product`的类，如下所示的代码：

```cs

命名空间

Packt.Shared

;

公共

类

产品

{

}

```

### 定义类别和产品实体类

`Category`类，也称为实体模型，将用于表示`Categories`表中的一行。 此表有四列，如下所示的 DDL：

```cs

创建

表

类别 (

CategoryId   INTEGER

主键

，

CategoryName NVARCHAR

（15

）NOT

NULL

，

描述  "NTEXT"

，

图片      "图像"

);

```

我们将使用约定来定义：

+   四个属性中的三个（我们不会映射`Picture`列）。

+   主键。

+   到`Products`表的一对多关系。

为了将`Description`列映射到正确的数据库类型，我们需要使用`Column`属性装饰`字符串`属性。

在本章的后面，我们将使用 Fluent API 来定义`CategoryName`不能为空，并且最多为 15 个字符。

让我们开始：

1.  修改`Category`实体模型类，如下所示的代码：

```cs

使用

System.ComponentModel.DataAnnotations.Schema; // [Column]

命名空间

Packt.Shared

;

公共

类

Category

{

//这些属性映射到数据库中的列

公共

int

CategoryId {获取

; set

; }

公共

字符串

？CategoryName {获取

; set

; }

[Column(TypeName =

"ntext"

）

]

公共

字符串

？描述 {获取

; set

; }

//为相关行定义导航属性

公共

虚拟

ICollection<Product> Products {获取

; set

; }

公共

类别

（）

{

// 以便开发人员可以将产品添加到类别，我们必须

//将导航属性初始化为空集合

Products = new

HashSet<Product>();

}

}

```

`Product`类将用于表示`Products`表中的一行，该表有十列。

您不需要将表中的所有列包括在类的属性中。 我们只会映射六个属性：`ProductId`，`ProductName`，`UnitPrice`，`UnitsInStock`，`Discontinued`和`CategoryId`。

15

我们可以通过定义具有不同名称的属性（如`Cost`）来重命名列，然后使用`[Column]`属性装饰该属性并指定其列名（如`UnitPrice`）。

最后一个属性`CategoryId`与将用于将每个产品映射到其父类别的`Category`属性相关联。

1.  根据以下代码修改`Product`类：

```cs

using

System.ComponentModel.DataAnnotations; // [Required], [StringLength]

using

System.ComponentModel.DataAnnotations.Schema; // [Column]

namespace

Packt.Shared

;

public

class

Product

{

public

int

ProductId { get

; set

; } // 主键

...

]

[StringLength(40)

]

public

string

ProductName { get

; set

; } = null

!;

[Column(

"UnitPrice"

, TypeName =

"money"

)

]

public

decimal

? Cost { get

; set

; } // 属性名称！=列名称

[Column(

"UnitsInStock"

)

]

public

short

? Stock { get

; set

; }

public

bool

Discontinued { get

; set

; }

// 这两个定义了外键关系

// 到 Categories 表

public

int

CategoryId { get

; set

; }

不映射到属性的列不能使用类实例读取或设置。如果使用该类创建新对象，则表中的新行将具有该行中未映射列的`NULL`或其他默认值。您必须确保这些缺失的列是可选的，或者由数据库设置默认值，否则将在运行时引发异常。在这种情况下，行已经具有数据值，我决定在此应用程序中不需要读取这些值。

virtual

Category Category { get

; set

; } = null

!;

}

```

两个实体相关的两个属性，`Category.Products`和`Product.Category`，都标记为`virtual`。这允许 EF Core 继承和重写属性，以提供额外的功能，如延迟加载。

## 添加表到 Northwind 数据库上下文类

在派生自`DbContext`的类中，您必须至少定义一个`DbSet<T>`类型的属性。这些属性表示表。为了告诉 EF Core 每个表有哪些列，`DbSet<T>`属性使用泛型来指定表示表中行的类。该实体模型类具有表示其列的属性。

派生自`DbContext`的类可以选择具有名为`OnModelCreating`的重写方法。这是您可以编写 Fluent API 语句的地方，作为在实体类上使用属性的替代方法。

让我们写一些代码：

1.  根据以下代码的高亮部分修改`Northwind`类，以定义两个表的两个属性和一个`OnModelCreating`方法：

```cs

public

class

Northwind

: DbContext

{

// 这些属性映射到数据库中的表

public

DbSet<Category>? Categories {

get

;

set

**

public

DbSet<Product>? Products {

get

;

set

; }

protected

override

void

OnConfiguring

(

DbContextOptionsBuilder optionsBuilder

)

{

OnModelCreating

}

保护

override

void

; }

（

ModelBuilder modelBuilder

）**

{

// 使用 Fluent API 而不是属性的示例

// 限制类别名称的长度为 15

modelBuilder.Entity<Category>()

.Property(category => category.CategoryName)

.IsRequired()

// 非空

.HasMaxLength(

.HasConversion<

);

如果

(ProjectConstants.DatabaseProvider ==

"SQLite"

**）**

{

// 添加以“修复”SQLite 中对 decimal 的支持不足

modelBuilder.Entity<Product>()

.Property(product => product.Cost)

[Required

double

**>();**

}

**}**

}

```

在 EF Core 3.0 及更高版本中，SQLite 数据库提供程序不支持`decimal`类型进行排序和其他操作。我们可以通过告诉模型，在使用 SQLite 数据库提供程序时，`decimal`值可以转换为`double`值来解决这个问题。这实际上并不会在运行时执行任何转换。

现在您已经看到了一些手动定义实体模型的示例，让我们看看一个可以为您完成部分工作的工具。

## 设置 dotnet-ef 工具

.NET 有一个名为`dotnet`的命令行工具。它可以扩展为对 EF Core 有用的功能。它可以执行设计时任务，如从旧模型创建和应用迁移到新模型，并从现有数据库生成模型的代码。

`dotnet` `ef`命令行工具不会自动安装。您必须将此包安装为**全局**或**本地工具**。如果您已安装了旧版本的工具，则应卸载任何现有版本：

1.  在命令提示符或终端中，检查是否已将`dotnet-ef`作为全局工具安装，如下所示：

```cs

dotnet tool list --global

```

1.  检查列表中是否已安装了旧版本的工具，例如.NET Core 3.1 的版本，如下所示：

```cs

包 Id 版本 命令

-------------------------------------

dotnet-ef       3.1.0       dotnet-ef

```

1.  如果已安装旧版本，请使用以下命令卸载该工具：

```cs

dotnet tool uninstall --global dotnet-ef

```

1.  安装最新版本，如下所示：

```cs

dotnet tool install --global dotnet-ef --version 6.0.0

```

1.  如有必要，请按照任何特定于操作系统的说明将`dotnet tools`目录添加到您的 PATH 环境变量中，如在安装`dotnet-ef`工具的输出中所述。

## 使用现有数据库搭建模型

脚手架是使用工具进行反向工程创建表示现有数据库模型的类的过程。一个好的脚手架工具允许您扩展自动生成的类，然后重新生成这些类而不会丢失您的扩展类。

如果您知道永远不会使用该工具重新生成类，则可以随意更改自动生成的类的代码。工具生成的代码只是最佳近似值。

**良好实践**：当您知道得更好时，不要害怕否决工具。

让我们看看该工具是否生成了与我们手动生成的模型相同的模型：

1.  将`Microsoft.EntityFrameworkCore.Design`包添加到`WorkingWithEFCore`项目中。

1.  在`WorkingWithEFCore`文件夹的命令提示符或终端中，生成一个名为`AutoGenModels`的新文件夹中的`Categories`和`Products`表的模型，如下命令所示：

```cs

dotnet ef dbcontext scaffold "Filename=Northwind.db" Microsoft.EntityFrameworkCore.Sqlite --table Categories --table Products --output-dir AutoGenModels --namespace WorkingWithEFCore.AutoGen --data-annotations --context Northwind

```

注意以下内容：

+   命令操作：`dbcontext scaffold`

+   连接字符串：`"Filename=Northwind.db"`

+   数据库提供程序：`Microsoft.EntityFrameworkCore.Sqlite`

+   要生成模型的表：`--table Categories --table Products`

+   输出文件夹：`--output-dir AutoGenModels`

+   命名空间：`--namespace WorkingWithEFCore.AutoGen`

+   要使用数据注释以及 Fluent API：`--data-annotations`

+   要将上下文从[database_name]Context 重命名为：`--context Northwind`

对于 SQL Server，请更改数据库提供程序和连接字符串，如下所示：

```cs

dotnet ef dbcontext scaffold "Data Source=.;Initial Catalog=Northwind;Integrated Security=true;" Microsoft.EntityFrameworkCore.SqlServer --table Categories --table Products --output-dir AutoGenModels --namespace WorkingWithEFCore.AutoGen --data-annotations --context Northwind

```

1.  注意构建消息和警告，如下所示：

```cs

构建开始...

构建成功。

为了保护可能包含敏感信息的连接字符串，您应该将其移出源代码。您可以通过使用 Name=语法从配置中读取它来避免脚手架连接字符串 - 请参阅 https://go.microsoft.com/fwlink/?linkid=2131148。有关存储连接字符串的更多指导，请参阅 http://go.microsoft.com/fwlink/?LinkId=723263。

在模型中找不到主表“Suppliers”，因此跳过了表“Products”上的标识为“0”的外键。这通常发生在未在选择集中包含主表时。

```

1.  打开`AutoGenModels`文件夹，注意自动生成的三个类文件：`Category.cs`，`Northwind.cs`和`Product.cs`。

1.  打开`Category.cs`，注意与手动创建的文件的区别，如下面的代码所示：

```cs

using

System;

using

System.Collections.Generic;

using

System.ComponentModel.DataAnnotations;

using

System.ComponentModel.DataAnnotations.Schema;

using

Microsoft.EntityFrameworkCore;

namespace

WorkingWithEFCore.AutoGen

{

[Index(nameof(CategoryName), Name =

"CategoryName"

)

]

public

partial

class

Category

{

public

Category

()

{

Products = new

HashSet<Product>();

}

[Key

]

public

long

CategoryId { get

; set

; }

[Required

]

[Column(TypeName =

"nvarchar (15)"

)

] // SQLite

[StringLength(15)

] // SQL Server

public

string

CategoryName { get

; set

; }

[Column(TypeName =

"ntext"

)

]

public

string

? Description { get

; set

; }

[Column(TypeName =

"image"

)

]

public

byte

[]? Picture { get

; set

; }

[InverseProperty(nameof(Product.Category))

]

public

virtual

ICollection<Product> Products { get

; set

; }

}

}

```

请注意以下内容：

+   它使用在 EF Core 5.0 中引入的`[Index]`属性装饰实体类。这表示应该具有索引的属性。在早期版本中，只支持使用 Fluent API 来定义索引。由于我们正在使用现有数据库，因此不需要这个。但是，如果我们想从我们的代码重新创建一个新的空数据库，那么这些信息将是必要的。

+   数据库中的表名是`Categories`，但`dotnet-ef`工具使用**Humanizer**第三方库自动将类名单数化为`Category`，这在创建单个实体时是更自然的名称。

+   实体类使用`partial`关键字声明，以便您可以创建匹配的`partial`类以添加额外的代码。这允许您重新运行工具并重新生成实体类，而不会丢失额外的代码。

+   `CategoryId`属性使用`[Key]`属性装饰，表示它是此实体的主键。此属性的数据类型对于 SQL Server 是`int`，对于 SQLite 是`long`。

+   `Products`属性使用`[InverseProperty]`属性来定义与`Product`实体类上的`Category`属性的外键关系。

1.  打开`Product.cs`，注意与手动创建的文件的区别。

1.  打开`Northwind.cs`，注意与手动创建的文件的区别，如下面编辑过的代码所示：

```cs

using

Microsoft.EntityFrameworkCore;

namespace

WorkingWithEFCore.AutoGen

{

public

partial

class

Northwind

: DbContext

{

public

Northwind

()

{

}

public

Northwind

(

DbContextOptions<Northwind> options

)

:

base

(

options

)

{

}

public

virtual

DbSet<Category> Categories { get

; set

; } = null

！;

public

virtual

DbSet<Product> Products { get

; set

; } = null

！;

protected

override

void

OnConfiguring

(

DbContextOptionsBuilder optionsBuilder

)

{

if

(!optionsBuilder.IsConfigured)

{

#

warning

为了保护可能包含敏感信息的连接字符串，您应该将其移出源代码。您可以通过使用 Name=语法从配置中读取它来避免脚手架连接字符串 - 请参阅 https://go.microsoft.com/fwlink/?linkid=2131148。有关存储连接字符串的更多指导，请参阅 http://go.microsoft.com/fwlink/?LinkId=723263。

optionsBuilder.UseSqlite("Filename=Northwind.db"

);

}

}

受保护的

覆盖

空

OnModelCreating

(

ModelBuilder modelBuilder

)

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

部分

空

OnModelCreatingPartial

(

ModelBuilder modelBuilder

)

;

}

}

```

注意以下内容：

+   `Northwind`数据上下文类是`partial`的，允许您扩展它并在将来重新生成它。

+   它有两个构造函数：一个默认的无参数构造函数，一个允许传入选项的构造函数。这在应用程序中很有用，您可以在运行时指定连接字符串。

+   代表`Categories`和`Products`表的两个`DbSet<T>`属性被设置为`null`-forgiving 值，以防止在编译时出现静态编译器分析警告。在运行时没有影响。

+   在`OnConfiguring`方法中，如果在构造函数中没有指定选项，则默认使用连接字符串，在当前文件夹中查找数据库文件。它有一个编译器警告，提醒您不要在此连接字符串中硬编码安全信息。

+   在`OnModelCreating`方法中，使用 Fluent API 配置了两个实体类，然后调用了一个名为`OnModelCreatingPartial`的部分方法。这允许您在自己的部分`Northwind`类中实现该部分方法，以添加您自己的 Fluent API 配置，如果重新生成模型类，这些配置将不会丢失。

1.  关闭自动生成的类文件。

## 配置预约定模型

除了支持与 SQLite 数据库提供程序一起使用的`DateOnly`和`TimeOnly`类型之外，EF Core 6 引入的新功能之一是配置预约定模型。

随着模型变得更加复杂，依靠约定来发现实体类型及其属性，并成功地将它们映射到表和列变得更加困难。如果您能在使用约定分析和构建模型之前配置约定本身，那将是非常有用的。

例如，您可能希望定义一个约定，即所有`string`属性应默认具有最大长度为 50 个字符，或者实现自定义接口的任何属性类型不应被映射，如下面的代码所示：

```cs

受保护的

覆盖

空

ConfigureConventions

(

ModelConfigurationBuilder configurationBuilder

)

{

configurationBuilder.Properties<string

>().HaveMaxLength(50

);

configurationBuilder.IgnoreAny<IDoNotMap>();

}

```

在本章的其余部分，我们将使用您手动创建的类。

# 查询 EF Core 模型

现在我们有了一个映射到 Northwind 数据库及其两个表的模型，我们可以编写一些简单的 LINQ 查询来获取数据。您将在*第十一章*，*使用 LINQ 查询和操作数据*中学到更多关于编写 LINQ 查询的知识。

现在，只需编写代码并查看结果：

1.  在`Program.cs`的顶部，导入主要的 EF Core 命名空间，以启用`Include`扩展方法从相关表中预取数据：

```cs

使用

Microsoft.EntityFrameworkCore; // 包括扩展方法

```

1.  在`Program.cs`的底部，定义一个`QueryingCategories`方法，并添加语句来执行这些任务，如下面的代码所示：

+   创建一个将管理数据库的`Northwind`类的实例。数据库上下文实例设计为短暂的工作单元。它们应尽快被处理，因此我们将它包装在`using`语句中。在*第十四章*，*使用 ASP.NET Core Razor Pages 构建网站*中，您将学习如何使用依赖注入获取数据库上下文。

+   创建一个包含其相关产品的所有类别的查询。

+   枚举类别，输出每个类别的名称和产品数量：

```cs

静态

空

查询类别

()

{

使用

(Northwind db = new

())

{

WriteLine("类别及其拥有的产品数量："

）；

//一个查询，获取所有类别及其相关产品

IQueryable<Category>?类别= db.Categories？

.Include（c => c.Products）;

如果

（类别是

null

）

{

WriteLine（“找不到类别。”

）；

返回

；

}

//执行查询并枚举结果

对于

（类别 c 在

类别）

{

WriteLine（$“

{c.CategoryName}

有

{c.Products.Count}

产品。”

）；

}

}

}

```

1.  在`Program.cs`的顶部，在输出数据库提供程序名称之后，调用`QueryingCategories`方法，如下面的代码中突出显示的那样：

```cs

WriteLine（$“使用

{ProjectConstants.DatabaseProvider}

数据库提供程序。”

）；

**QueryingCategories（）;**

```

1.  运行代码并查看结果（如果在 Windows 上使用 SQLite 数据库提供程序的 Visual Studio 2022 运行），如下面的输出所示：

```cs

使用 SQLite 数据库提供程序。

类别及其拥有的产品数量：

使用 C：\ Code \ Chapter10 \ WorkingWithEFCore \ bin \ Debug \ net6.0 \ Northwind.db 数据库文件。

饮料有 12 种产品。

调味品有 12 种产品。

糖果有 13 种产品。

乳制品有 10 种产品。

谷物/谷物有 7 种产品。

肉类/家禽有 6 种产品。

生产有 5 种产品。

海鲜有 12 种产品。

```

如果您在使用 SQLite 数据库提供程序的 Visual Studio Code 中运行，则路径将是`WorkingWithEFCore`文件夹。如果您使用 SQL Server 数据库提供程序运行，则不会输出数据库文件路径。

**警告！**如果在使用 Visual Studio 2022 的 SQLite 时看到以下异常，最可能的问题是`Northwind.db`文件没有复制到输出目录。确保**始终复制**设置为**始终复制**：

`未处理的异常。Microsoft.Data.Sqlite.SqliteException（0x80004005）：SQLite 错误 1：'没有表：类别'。`

## 包括实体的过滤

EF Core 5.0 引入了**过滤包含**，这意味着您可以在`Include`方法调用中指定 lambda 表达式，以过滤结果中返回的实体：

1.  在`Program.cs`的底部，定义一个`FilteredIncludes`方法，并添加语句来执行这些任务，如下面的代码所示：

+   创建一个将管理数据库的`Northwind`类的实例。

+   提示用户输入库存单位的最小值。

+   为具有库存单位最小数量的产品创建一个查询。

+   枚举类别和产品，输出每个类别的名称和库存单位：

```cs

静态

空

FilteredIncludes

（）

{

使用

（新风 db =新

（）

{

Write（“输入库存单位的最小值：”

）；

字符串

unitsInStock = ReadLine（）？？“10”

；

int

库存= int

.Parse（unitsInStock）;

IQueryable<Category>?类别= db.Categories？

.Include（c => c.Products.Where（p => p.Stock> = stock））;

如果

（类别是

null

）

{

WriteLine（“找不到类别。”

）；

返回

；

}

对于

（类别 c 在

类别）

{

WriteLine（$“

{c.CategoryName}

有

{c.Products.Count}

最少有

{stock}

库存单位。”

）；

对于

（产品 p 在

c.Products）

{

WriteLine（$“

{p.ProductName}

有

{p.Stock}

库存单位。”

）；

}

}

}

}

```

1.  在`Program.cs`中，注释掉`QueryingCategories`方法，并调用`FilteredIncludes`方法，如下面的代码中突出显示的那样：

```cs

WriteLine（$“使用

{ProjectConstants.DatabaseProvider}

数据库提供程序。”

）；

**// QueryingCategories（）;**

**FilteredIncludes（）;**

```

1.  运行代码，输入库存单位的最小值，如`100`，并查看结果，如下面的输出所示：

```cs

输入库存单位的最小值：100

饮料有 2 种产品，最少 100 个库存单位。

Sasquatch Ale 有 111 个库存单位。

Rhönbräu Klosterbier 有 125 个库存单位。

调味品有 2 种产品，最少 100 个库存单位。

Grandma's Boysenberry Spread 有 120 个库存单位。

Sirop d'érable 有 113 个库存单位。

糖果有 0 种产品，最少 100 个库存单位。

乳制品有 1 种产品，最少 100 个库存单位。

Geitost 库存中有 112 个单位。

谷物/谷物库存中有 1 种产品，最少 100 个单位。

Gustaf 的 Knäckebröd 库存中有 104 个单位。

肉类/家禽库存中有 1 种产品，最少 100 个单位。

Pâté chinois 库存中有 115 个单位。

生产中有 0 种产品，库存中至少有 100 个单位。

海鲜有 3 种产品，库存中至少有 100 个单位。

Inlagd Sill 库存中有 112 个单位。

Boston Crab Meat 库存中有 123 个单位。

Röd Kaviar 库存中有 101 个单位。

```

### Windows 控制台中的 Unicode 字符

在 Windows 10 秋季创作者更新之前的 Windows 版本上，Microsoft 提供的控制台存在限制。默认情况下，控制台无法显示 Unicode 字符，例如 Rhönbräu 的名称。

如果您遇到此问题，可以在运行应用程序之前在控制台中输入以下命令，临时更改代码页（也称为字符集）为 Unicode UTF-8：

```cs

chcp 65001

```

## 过滤和排序产品

让我们探索一个更复杂的查询，将过滤和排序数据：

1.  在`Program.cs`的底部，定义一个`QueryingProducts`方法，并添加以下语句，如下面的代码所示：

+   创建一个将管理数据库的`Northwind`类的实例。

+   提示用户输入产品价格。与前一个代码示例不同，我们将循环直到输入有效价格。

+   使用 LINQ 创建一个成本高于价格的产品查询。

+   循环遍历结果，输出 Id，名称，成本（以美元格式化），以及库存中的单位数：

```cs

静态

void

查询产品

（）

{

使用

（Northwind db =新

（）

{

WriteLine（“成本高于价格的产品，最高在顶部。”

）；

字符串

？输入；

十进制

价格；

做

{

Write（“输入产品价格：”

）；

输入= ReadLine（）;

} while

（！十进制

.TryParse（输入，out

价格）；

IQueryable<Product>？产品= db.Products？

.Where（product => product.Cost > price）

.OrderByDescending（product => product.Cost）;

如果

（产品是

null

）

{

WriteLine（“未找到产品。”

）；

返回

；

}

foreach

（Product p in

产品）

{

WriteLine（

“{0}：{1}的成本为{2：$ #，## 0.00}，库存为{3}。”

，

p.ProductId，p.ProductName，p.Cost，p.Stock）;

}

}

}

```

1.  在`Program.cs`中，注释掉上一个方法，并调用`QueryingProducts`方法

1.  运行代码，当提示输入产品价格时输入`50`，并查看结果，如下面的输出所示：

```cs

成本高于价格的产品，最高在顶部。

输入产品价格：50

38：Côte de Blaye 售价 263.50 美元，库存 17。

29：Thüringer Rostbratwurst 售价 123.79 美元，库存为 0。

9：Mishi Kobe Niku 售价 97.00 美元，库存 29。

20：Sir Rodney 的橙子酱售价 81.00 美元，库存 40。

18：Carnarvon Tigers 售价 62.50 美元，库存 42。

59：Raclette Courdavault 售价 55.00 美元，库存 79。

51：Manjimup Dried Apples 售价 53.00 美元，库存 20。

```

## 获取生成的 SQL

您可能想知道我们编写的 C#查询生成的 SQL 语句有多好。EF Core 5.0 引入了一种快速简便的方法来查看生成的 SQL：

1.  在`FilteredIncludes`方法中，在使用`foreach`语句枚举查询之前，添加一个语句来输出生成的 SQL，如下面的代码中所突出显示的那样：

```cs

**WriteLine（**

**$“ToQueryString：**

**{categories.ToQueryString()}**

**“**

**）；**

foreach

（类别 c 中

类别）

```

1.  在`Program.cs`中，注释掉对`QueryingProducts`方法的调用，并取消对`FilteredIncludes`方法的调用。

1.  运行代码，输入库存中的最小值，如`99`，并查看结果（在 SQLite 中运行时），如下面的输出所示：

```cs

输入库存的最小值：99

使用 SQLite 数据库提供程序。

ToQueryString：.param set @_stock_0 99

选择“c”。“CategoryId”，“c”。“CategoryName”，“c”。“Description”，

“t”。“ProductId”，“t”。“CategoryId”，“t”。“UnitPrice”，“t”。“Discontinued”，

"t"."ProductName"，"t"."UnitsInStock"

FROM "Categories" AS "c"

LEFT JOIN (

SELECT "p"."ProductId"，"p"."CategoryId"，"p"."UnitPrice"，

"p"."Discontinued", "p"."ProductName", "p"."UnitsInStock"

FROM "Products" AS "p"

WHERE（"p"."UnitsInStock">= @_stock_0）

）AS "t" ON "c"."CategoryId" = "t"."CategoryId"

ORDER BY "c"."CategoryId", "t"."ProductId"

饮料有 2 种产品，最低库存为 99 个单位。

Sasquatch Ale 有 111 个库存单位。

Rhönbräu Klosterbier 有 125 个库存单位。

...

```

注意 SQL 参数命名为`@_stock_0`已设置为最小库存值`99`。

对于 SQL Server，生成的 SQL 略有不同，例如，它使用方括号而不是双引号来表示对象名称，如下面的输出所示：

```cs

输入最低库存单位：99

使用 SqlServer 数据库提供程序。

ToQueryString：DECLARE @__stock_0 smallint = CAST(99 AS smallint);

SELECT [c].[CategoryId]，[c].[CategoryName]，[c].[Description]，[t].[ProductId]，[t].[CategoryId]，[t].[UnitPrice]，[t].[Discontinued]，[t].[ProductName]，[t].[UnitsInStock]

FROM [Categories] AS [c]

LEFT JOIN (

SELECT [p].[ProductId]，[p].[CategoryId]，[p].[UnitPrice]，[p].[Discontinued]，[p].[ProductName]，[p].[UnitsInStock]

FROM [Products] AS [p]

WHERE [p].[UnitsInStock] >= @__stock_0

）AS [t] ON [c].[CategoryId] = [t].[CategoryId]

ORDER BY [c].[CategoryId]，[t].[ProductId]

```

## 使用自定义日志提供程序记录 EF Core

要监视 EF Core 与数据库之间的交互，我们可以启用日志记录。这需要以下两个任务：

+   **日志提供程序**的注册。

+   **记录器**的实现。

让我们看一个示例：

1.  向项目添加名为`ConsoleLogger.cs`的文件。

1.  修改文件以定义两个类，一个用于实现`ILoggerProvider`，一个用于实现`ILogger`，如下面的代码所示，并注意以下内容：

+   `ConsoleLoggerProvider`返回`ConsoleLogger`的实例。它不需要任何非托管资源，因此`Dispose`方法不执行任何操作，但必须存在。

+   `ConsoleLogger`对于日志级别`None`，`Trace`和`Information`被禁用。它对所有其他日志级别启用。

+   `ConsoleLogger`通过写入`Console`实现其`Log`方法：

```cs

使用

Microsoft.Extensions.Logging; // ILoggerProvider，ILogger，LogLevel

使用

静态

System.Console;

命名空间

Packt.Shared

;

public

类

ConsoleLoggerProvider

：ILoggerProvider

{

public

ILogger

CreateLogger

(

string

categoryName

）

{

//我们可以有不同的记录器实现

//不同的 categoryName 值，但我们只有一个

return

new

ConsoleLogger();

}

//如果您的记录器使用非托管资源，

// 然后您可以在这里释放它们

public

void

Dispose

()

{ }

}

public

类

ConsoleLogger

：ILogger

{

// 如果您的记录器使用非托管资源，您可以

//在这里返回实现 IDisposable 的类

public

IDisposable

BeginScope

<

TState

>(

TState state

）

{

return

null

;

}

public

布尔

IsEnabled

(

LogLevel logLevel

)

{

//为了避免过度记录，您可以根据日志级别进行过滤

switch

（logLevel）

{

case

LogLevel.Trace：

case

LogLevel.Information：

case

LogLevel.None：

return

假

;

case

LogLevel.Debug：

case

LogLevel.Warning：

case

LogLevel.Error：

case

LogLevel.Critical：

默认

:

return

真

;

};

}

public

void

日志

<

TState

>(

LogLevel logLevel，

EventId eventId，TState state，Exception？exception，

Func<TState，Exception，

string

> 格式化程序

）

{

// 记录级别和事件标识

Write($"级别：

{logLevel}

，事件 ID：

{eventId.Id}

"

）;

//仅在状态或异常存在时输出

if

（状态！= null

）

{

Write($",状态：

{state}

"

）;

}

if

（异常！= null

）

{

Write($",异常：

{exception.Message}

"

）;

}

WriteLine();

}

}

```

1.  在`Program.cs`的顶部，添加语句以导入日志记录所需的命名空间，如下面的代码所示：

```cs

使用

Microsoft.EntityFrameworkCore.Infrastructure;

使用

Microsoft.Extensions.DependencyInjection;

使用

Microsoft.Extensions.Logging;

```

1.  我们已经使用`ToQueryString`方法获取了`FilteredIncludes`的 SQL，因此我们不需要向该方法添加日志记录。对于`QueryingCategories`和`QueryingProducts`方法，立即在`Northwind`数据库上下文的`using`块内添加语句，以获取日志工厂并注册您的自定义控制台记录器，如下面的代码所示：

```cs

使用

（Northwind db = new

())

{

**ILoggerFactory loggerFactory = db.GetService<ILoggerFactory>();**

**loggerFactory.AddProvider(**

**new**

**ConsoleLoggerProvider());**

```

1.  在`Program.cs`的顶部，注释掉对`FilteredIncludes`方法的调用，并取消注释对`QueryingProducts`方法的调用。

1.  运行代码并查看日志，部分日志显示在以下输出中：

```cs

...

级别：调试，事件 ID：20000，状态：打开到服务器'/Users/markjprice/Code/Chapter10/WorkingWithEFCore/Northwind.db'上的数据库'main'的连接。

级别：调试，事件 ID：20001，状态：已打开到服务器'/Users/markjprice/Code/Chapter10/WorkingWithEFCore/Northwind.db'上的数据库'main'的连接。

级别：调试，事件 ID：20100，状态：执行 DbCommand [Parameters=[@__price_0='?']，CommandType='Text'，CommandTimeout='30']

选择“p”。“ProductId”，“p”。“CategoryId”，“p”。“UnitPrice”，“p”。“Discontinued”，“p”。“ProductName”，“p”。“UnitsInStock”

FROM "Products" AS "p"

WHERE "p"。"UnitPrice" > @__price_0

ORDER BY "product"。"UnitPrice" DESC

...

```

您的日志可能会因您选择的数据库提供程序和代码编辑器以及 EF Core 的未来改进而有所不同。现在，请注意不同的事件（如打开连接或执行命令）具有不同的事件 ID。

### 通过提供程序特定值过滤日志

事件 ID 值及其含义将特定于.NET 数据提供程序。如果我们想知道 LINQ 查询如何被转换为 SQL 语句并执行，则要输出的事件 ID 具有`Id`值为`20100`：

1.  修改`ConsoleLogger`中的`Log`方法，仅输出`Id`为`20100`的事件，如下面的代码所示：

```cs

public

void

日志

<

TState

>(

LogLevel logLevel，EventId eventId，

TState state, Exception? exception,？

Func<TState, Exception，

字符串

> 格式化程序

)

{

**if**

**(eventId.Id ==**

**20100**

**)**

**{**

// 记录级别和事件标识符

Write("Level: {0}, Event Id: {1}, Event: {2}"

，

logLevel，eventId.Id，eventId.Name);

// 仅在存在时输出状态或异常

如果

（state！= null

)

{

Write($", 状态：

{state}

"

);

}

如果

（异常！= null

)

{

Write($", 异常：

{exception.Message}

"

);

}

WriteLine();

**}**

}

```

1.  在`Program.cs`中，取消注释`QueryingCategories`方法，并注释掉其他方法，以便我们可以监视连接两个表时生成的 SQL 语句。

1.  运行代码，并注意以下 SQL 语句的日志记录，如下面编辑过的输出所示：

```cs

使用 SQLServer 数据库提供程序。

类别及其产品数量：

级别：调试，事件 ID：20100，状态：执行 DbCommand [Parameters=[]，CommandType='Text'，CommandTimeout='30']

选择[c]。[CategoryId]，[c]。[CategoryName]，[c]。[Description]，[p]。[ProductId]，[p]。[CategoryId]，[p]。[UnitPrice]，[p]。[Discontinued]，[p]。[ProductName]，[p]。[UnitsInStock]

FROM [Categories] AS [c]

LEFT JOIN [Products] AS [p] ON [c]。[CategoryId] = [p]。[CategoryId]

ORDER BY [c]。[CategoryId]，[p]。[ProductId]

饮料有 12 种产品。

调味品有 12 种产品。

糖果有 13 种产品。

乳制品有 10 种产品。

谷物/谷物有 7 种产品。

肉类/家禽有 6 种产品。

生产有 5 种产品。

海鲜有 12 种产品。

```

### 使用查询标签记录

在记录 LINQ 查询时，在复杂的情况下，很难将日志消息相关联。EF Core 2.2 引入了查询标签功能，通过允许您向日志中添加 SQL 注释来帮助。

您可以使用`TagWith`方法注释 LINQ 查询，如下面的代码所示：

```cs

IQueryable<Product>？产品= db.Products？

.TagWith("按价格和排序过滤的产品。"

)

。Where（product => product.Cost>价格）

.OrderByDescending（product => product.Cost）;

```

这将在日志中添加一个 SQL 注释，如下面的输出所示：

```cs

--按价格和排序过滤的产品。

```

## 使用 Like 进行模式匹配

EF Core 支持常见的 SQL 语句，包括用于模式匹配的`Like`：

1.  在`Program.cs`的底部，添加一个名为`QueryingWithLike`的方法，如下面的代码所示，并注意：

+   我们已启用日志记录。

+   我们提示用户输入产品名称的一部分，然后使用`EF.Functions.Like`方法在`ProductName`属性的任何位置进行搜索。

+   对于每个匹配的产品，我们输出其名称，库存以及是否已停产：

```cs

静态

void

QueryingWithLike

()

{

使用

（Northwind db = new

（）

{

ILoggerFactory loggerFactory = db.GetService<ILoggerFactory>();

loggerFactory.AddProvider(new

ConsoleLoggerProvider());

Write("输入产品名称的一部分："

）;

字符串

？ 输入= ReadLine（）;

IQueryable<Product>？产品= db.Products？

。Where（p => EF.Functions.Like（p.ProductName，$“％

{输入}

％”

）;

如果

（产品是

null

）

{

WriteLine("找不到产品。"

）;

返回

;

}

对于

（在

产品）

{

WriteLine（“{0}有{1}个单位的库存。已停产？{2}”

，

p.ProductName，p.Stock，p.Discontinued);

}

}

}

```

1.  在`Program.cs`中，注释掉现有的方法，并调用`QueryingWithLike`。

1.  运行代码，输入部分产品名称，例如`che`，并查看结果，如下面的输出所示：

```cs

使用 SQLServer 数据库提供程序。

输入产品名称的一部分：che

级别：调试，事件 ID：20100，状态：执行 DbCommand [参数=[@__Format_1='?'（大小=40）]，命令类型='文本'，命令超时='30']

选择“p”。 “ProductId”，“p”。 “CategoryId”，“p”。 “UnitPrice”，

“p”。 “已停产”，“p”。 “ProductName”，“p”。 “UnitsInStock” FROM“Products” AS“p”

WHERE“p”。 “ProductName”LIKE @__Format_1

Chef Anton's Cajun Seasoning 有 53 个单位的库存。已停产？假

Chef Anton's Gumbo Mix 有 0 个单位的库存。已停产？真

Queso Manchego La Pastora 有 86 个单位的库存。已停产？假

Gumbär Gummibärchen 有 15 个单位的库存。已停产？假

```

EF Core 6.0 引入了另一个有用的函数`EF.Functions.Random`，它映射到返回介于 0 和 1 之间的伪随机数的数据库函数。例如，您可以将随机数乘以表中的行数来选择该表中的一个随机行。

## 定义全局过滤器

Northwind 产品可能已停产，因此确保即使程序员在其查询中不使用`Where`来过滤它们，也可以确保永远不会返回已停产的产品可能是有用的：

1.  在`Northwind.cs`中，修改`OnModelCreating`方法以添加全局过滤器以删除已停产的产品，如下面代码中突出显示的那样：

```cs

受保护的

覆盖

void

OnModelCreating

(

ModelBuilder modelBuilder

)

{

...

**//全局过滤器以删除已停产的产品**

**modelBuilder.Entity<Product>()**

**.HasQueryFilter（p =>！p.Discontinued）;**

}

```

1.  运行代码，输入部分产品名称`che`，查看结果，并注意到**Chef Anton's Gumbo Mix**现在已经消失，因为生成的 SQL 语句包括对`Discontinued`列的过滤，如下面的输出所示：

```cs

选择“p”。 “ProductId”，“p”。 “CategoryId”，“p”。 “UnitPrice”，

“p”。 “已停产”，“p”。 “ProductName”，“p”。 “UnitsInStock”

FROM“Products” AS“p”

在哪里

**（“p”。 “已停产”= 0）**

AND“p”。 “ProductName”LIKE @__Format_1

Chef Anton's Cajun Seasoning 有 53 个单位的库存。已停产？假

Queso Manchego La Pastora 有 86 个单位的库存。已停产？假

Gumbär Gummibärchen 有 15 个单位的库存。已停产？假

```

# 使用 EF Core 加载模式

通常在 EF Core 中使用三种加载模式：

+   **急加载**：提前加载数据。

+   **懒加载**：在需要之前自动加载数据。

+   **显式加载**：手动加载数据。

在本节中，我们将介绍每一个。

## 急加载实体

在 `QueryingCategories` 方法中，代码当前使用 `Categories` 属性循环遍历每个类别，输出该类别的名称和该类别中的产品数量。

这是因为当我们编写查询时，通过调用 `Include` 方法启用了急加载。

让我们看看如果不调用 `Include` 会发生什么：

1.  修改查询以注释掉 `Include` 方法调用，如下所示：

```cs

IQueryable<Category>? categories =

db.Categories; //.Include(c => c.Products);

```

1.  在 `Program.cs` 中，注释掉除 `QueryingCategories` 之外的所有方法。

1.  运行代码并查看结果，如下部分输出所示：

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

`foreach` 中的每个项目都是 `Category` 类的实例，该类具有名为 `Products` 的属性，即该类别中的产品列表。由于原始查询仅从 `Categories` 表中选择，因此该属性对于每个类别都为空。

## 启用懒加载

懒加载是在 EF Core 2.1 中引入的，它可以自动加载丢失的相关数据。要启用懒加载，开发人员必须：

+   引用代理的 NuGet 包。

+   配置懒加载以使用代理。

让我们看看这个实例：

1.  在 `WorkingWithEFCore` 项目中，添加 EF Core 代理的包引用，如下标记所示：

```cs

<PackageReference

Include="Microsoft.EntityFrameworkCore.Proxies"

Version="6.0.0"

/>

```

1.  构建项目以恢复包。

1.  打开 `Northwind.cs`，并在 `OnConfiguring` 方法的顶部调用一个扩展方法来使用懒加载代理，如下所示在以下代码中突出显示：

```cs

protected

覆盖

void

OnConfiguring

(

DbContextOptionsBuilder optionsBuilder

)

{

**optionsBuilder.UseLazyLoadingProxies();**

```

现在，每次循环枚举时，尝试读取 `Products` 属性时，懒加载代理将检查它们是否已加载。如果没有，它将通过执行 `SELECT` 语句来“懒惰”地加载当前类别的产品集，然后正确的计数将返回到输出。

1.  运行代码并注意产品计数现在是正确的。但是您会看到懒加载的问题是需要多次往返到数据库服务器才能最终获取所有数据，如下部分输出所示：

```cs

类别及其拥有的产品数量：

级别：调试，事件 ID：20100，状态：执行 DbCommand [参数= []，命令类型='Text'，命令超时='30']

SELECT "c"."CategoryId", "c"."CategoryName", "c"."Description" FROM "Categories" AS "c"

级别：调试，事件 ID：20100，状态：执行 DbCommand [参数= [@ p_0='?']，命令类型='Text'，命令超时='30']

SELECT "p"."ProductId", "p"."CategoryId", "p"."UnitPrice",

"p"."Discontinued", "p"."ProductName", "p"."UnitsInStock"

FROM "Products" AS "p"

WHERE ("p"."Discontinued" = 0) AND ("p"."CategoryId" = @ p_0)

Beverages has 11 products.

级别：调试，事件 ID：20100，状态：执行 DbCommand [参数= [@ p_0='?']，命令类型='Text'，命令超时='30']

SELECT "p"."ProductId", "p"."CategoryId", "p"."UnitPrice",

"p"."Discontinued", "p"."ProductName", "p"."UnitsInStock"

FROM "Products" AS "p"

WHERE ("p"."Discontinued" = 0) AND ("p"."CategoryId" = @ p_0)

Condiments has 11 products.

```

## 显式加载实体

另一种加载类型是显式加载。它的工作方式类似于延迟加载，不同之处在于您可以控制加载的相关数据以及何时加载：

1.  在`Program.cs`的顶部，导入更改跟踪命名空间，以便我们可以使用`CollectionEntry`类手动加载相关实体，如下所示的代码：

```cs

使用

Microsoft.EntityFrameworkCore.ChangeTracking; // CollectionEntry

```

1.  在`QueryingCategories`方法中，修改语句以禁用延迟加载，然后提示用户是否要启用急切加载和显式加载，如下所示的代码：

```cs

IQueryable<Category>? categories;

// = db.Categories;

// .Include(c => c.Products);

db.ChangeTracker.LazyLoadingEnabled = false

;

Write("启用急切加载？（Y/N）："

）;

布尔值

eagerloading = (ReadKey().Key == ConsoleKey.Y);

布尔值

explicitloading = false

;

WriteLine();

如果

（急切加载）

{

categories = db.Categories?.Include(c => c.Products);

}

否则

{

categories = db.Categories;

Write("启用显式加载？（Y/N）："

);

explicitloading = (ReadKey().Key == ConsoleKey.Y);

WriteLine();

}

```

1.  在`foreach`循环中，在`WriteLine`方法调用之前，添加语句来检查是否启用了显式加载，如果是，则提示用户是否要显式加载每个单独的类别，如下所示的代码：

```cs

如果

（显式加载）

{

Write($"显式加载产品

{c.CategoryName}

？（Y/N）："

）;

ConsoleKeyInfo key = ReadKey();

WriteLine();

如果

（key.Key == ConsoleKey.Y）

{

CollectionEntry<Category, Product> products =

db.Entry(c).Collection(c2 => c2.Products);

如果

（！products.IsLoaded）products.Load();

}

}

WriteLine($"

{c.CategoryName}

有

{c.Products.Count}

产品。

）;

```

1.  运行代码：

1.  按`N`键禁用急切加载。

1.  然后按`Y`键启用显式加载。

1.  对于每个类别，按`Y`或`N`加载其产品，如您所愿。

我选择只为八个类别中的两个加载产品，饮料和海鲜，如下所示的输出已经被编辑以节省空间：

```cs

类别及其产品数量：

启用急切加载？（Y/N）：n

启用显式加载？（Y/N）：y

级别：调试，事件 ID：20100，状态：执行 DbCommand [参数=[]，命令类型='文本'，命令超时='30']

SELECT "c"."CategoryId", "c"."CategoryName", "c"."Description" FROM "Categories" AS "c"

显式加载饮料的产品？（Y/N）：y

级别：调试，事件 ID：20100，状态：执行 DbCommand [参数=[@ p_0='?']，命令类型='文本'，命令超时='30']

选择“p”。"ProductId"，“p"。"CategoryId"，“p"。"UnitPrice"，

"p"."Discontinued"，“p"。"ProductName"，“p"。"UnitsInStock"

FROM "Products" AS "p"

WHERE ("p"."Discontinued" = 0) AND ("p"."CategoryId" = @ p_0)

饮料有 11 种产品。

显式加载调味品的产品？（Y/N）：n

调味品没有产品。

显式加载糖果的产品？（Y/N）：n

糖果没有产品。

显式加载乳制品的产品？（Y/N）：n

乳制品没有产品。

显式加载谷物/谷物的产品？（Y/N）：n

谷物/谷物没有产品。

显式加载肉类/家禽的产品？（Y/N）：n

肉类/家禽没有产品。

显式加载生产的产品？（Y/N）：n

生产没有产品。

显式加载海鲜的产品？（Y/N）：y

调试级别：事件 ID：20100，状态：执行 DbCommand [参数=[@ p_0='?']，命令类型='文本'，命令超时='30']

SELECT "p"."ProductId", "p"."CategoryId", "p"."UnitPrice",

"p"。"Discontinued"，“p"。"ProductName"，“p"。"UnitsInStock"

FROM "Products" AS "p"

WHERE ("p"."Discontinued" = 0) AND ("p"."CategoryId" = @ p_0)

海鲜有 12 种产品。

```

**良好的实践**：仔细考虑哪种加载模式最适合您的代码。懒加载可能会让您成为一个懒惰的数据库开发人员！在以下链接中阅读有关加载模式的更多信息：[`docs.microsoft.com/en-us/ef/core/querying/related-data`](https://docs.microsoft.com/en-us/ef/core/querying/related-data)

# 使用 EF Core 操纵数据

使用 EF Core 插入、更新和删除实体是一项易于完成的任务。

`DbContext` 自动维护更改跟踪，因此本地实体可以跟踪多个更改，包括添加新实体、修改现有实体和删除实体。当您准备将这些更改发送到底层数据库时，请调用 `SaveChanges` 方法。将返回成功更改的实体数。

## 插入实体

让我们首先看看如何向表中添加新行：

1.  在 `Program.cs` 中，创建一个名为 `AddProduct` 的新方法，如下面的代码所示：

```cs

静态

布尔

AddProduct

(

int

categoryId，

字符串

productName，

decimal

？价格

）

{

使用

(Northwind db = new

（）

{

Product p = new

()

{

CategoryId = categoryId,

ProductName = productName,

成本 = 价格

};

// 标记产品为更改跟踪中的已添加

db.Products.Add(p);

// 保存跟踪的更改到数据库

int

affected = db.SaveChanges();

返回

(affected == 1

);

}

}

```

1.  在 `Program.cs` 中，创建一个名为 `ListProducts` 的新方法，该方法输出每个产品的 Id、名称、成本、库存和停产属性，按成本最高的产品排序，如下面的代码所示：

```cs

静态

void

列出产品

()

{

使用

(Northwind db = new

（）

{

WriteLine("{0,-3} {1,-35} {2,8} {3,5} {4}"

，

“Id”

，“产品名称”

，成本

，"库存"

，“折扣”

);

foreach

（Product p in

db.Products

.OrderByDescending(product => product.Cost))

{

WriteLine("{0:000} {1,-35} {2,8:$#,##0.00} {3,5} {4}"

，

p.ProductId, p.ProductName, p.Cost, p.Stock, p.Discontinued);

}

}

}

```

请记住，`1,-35` 表示左对齐参数 1 在 35 个字符宽的列中，`3,5` 表示右对齐参数 3 在 5 个字符宽的列中。

1.  在 `Program.cs` 中，注释掉以前的方法调用，然后调用 `AddProduct` 和 `ListProducts`，如下面的代码所示：

```cs

// QueryingCategories();

// FilteredIncludes();

// QueryingProducts();

// QueryingWithLike();

if

(AddProduct(categoryId: 6

，

productName: "Bob's Burgers"

，价格：500

M))

{

WriteLine("添加产品成功。"

);

}

ListProducts();

```

1.  运行代码，查看结果，并注意新产品已添加，如下面的部分输出所示：

```cs

添加产品成功。

Id 产品名称 成本 库存 折扣。

078 Bob's Burgers          $500.00       False

038 Côte de Blaye          $263.50    17 False

020 Sir Rodney's Marmalade  $81.00    40 False```

...

```

## 更新实体

现在，让我们修改表中的现有行：

1.  在 `Program.cs` 中，添加一个方法，通过指定的值（我们的示例中使用 Bob）增加以指定值开头的第一个产品的价格，如 $20，如下面的代码所示：

```cs

静态

布尔

IncreaseProductPrice

(

字符串

productNameStartsWith,

decimal

金额

)

{

使用

(Northwind db = new

（）

{

// 获取名称以 name 开头的第一个产品

Product updateProduct = db.Products.First(

p => p.ProductName.StartsWith(productNameStartsWith));

updateProduct.Cost += amount;

int

affected = db.SaveChanges();

返回

(affected == 1

);

}

}

```

1.  在 `Program.cs` 中，注释掉整个调用 `AddProduct` 的 `if` 块，并在调用列出产品之前添加对 `IncreaseProductPrice` 的调用，如下面的代码中所突出显示的那样：

```cs

**/***

if (AddProduct(categoryId: 6,

productName: "Bob's Burgers", price: 500M))

{

WriteLine("添加产品成功。");

}

***/**

**如果**

**(IncreaseProductPrice(**

**productNameStartsWith:**

**"Bob"**

**，金额：**

**20**

**M))**

**{**

**WriteLine(**

**"更新产品价格成功。"**

**）；**

**}**

ListProducts();

```

1.  运行代码，查看结果，并注意 Bob's Burgers 的现有实体的价格增加了$20，如下面的部分输出所示：

```cs

更新产品价格成功。

Id 产品名称 成本 库存 折扣

078 Bob's Burgers          $520.00       False

038 Côte de Blaye          $263.50    17 False

020 Sir Rodney's Marmalade  $81.00    40 False

...

```

## 删除实体

您可以使用`Remove`方法删除单个实体。当您想要删除多个实体时，`RemoveRange`更有效。

现在让我们看看如何从表中删除行：

1.  在`Program.cs`的底部，添加一个方法来删除所有以指定值开头的产品（在我们的示例中为 Bob），如下面的代码所示：

```cs

static

int

DeleteProducts

(

string

productNameStartsWith

)

{

using

（Northwind db = new

())

{

IQueryable<Product>? products = db.Products?.Where(

p => p.ProductName.StartsWith(productNameStartsWith));

if

（products 是

null

)

{

WriteLine("未找到要删除的产品。"

);

return

0

;

}

else

{

db.Products.RemoveRange(products);

}

int

affected = db.SaveChanges();

return

affected;

}

}

```

1.  在`Program.cs`中，注释掉调用`IncreaseProductPrice`的整个`if`语句块，并添加一个调用`DeleteProducts`，如下面的代码所示：

```cs

int

deleted = DeleteProducts(productNameStartsWith: "Bob"

);

WriteLine($"

{deleted}

已删除个产品。"

);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

1 个产品已被删除。

```

如果有多个产品名称以 Bob 开头，则它们都将被删除。作为一个可选的挑战，修改语句以添加三个以 Bob 开头的新产品，然后将它们删除。

## 池化数据库上下文

`DbContext`类是可处置的，并且遵循单工作单元原则。在前面的代码示例中，我们在`using`块中创建了所有`DbContext` -派生的 Northwind 实例，以便在每个工作单元结束时正确调用`Dispose`。

与 EF Core 相关的 ASP.NET Core 的一个特性是，在构建网站和服务时，它通过池化数据库上下文使您的代码更加高效。这允许您创建和处置尽可能多的`DbContext` -派生对象，而知道您的代码仍然尽可能高效。

# 处理事务

每次调用`SaveChanges`方法时，都会启动一个**隐式** **事务**，以便如果出现问题，它将自动回滚所有更改。如果事务内的多个更改成功，则提交事务和所有更改。

事务通过应用锁来维护数据库的完整性，以防止在一系列更改发生时进行读取和写入。

事务是**ACID**，这是一个在下面列表中解释的首字母缩写：

+   **A 代表原子**。事务中的所有操作都提交，或者都不提交。

+   **C 代表一致**。事务之前和之后数据库的状态是一致的。这取决于您的代码逻辑；例如，在银行账户之间转账时，您的业务逻辑要确保如果在一个账户中借记$100，则在另一个账户中贷记$100。

+   **I 代表隔离**。在事务期间，更改对其他进程隐藏。您可以从多个隔离级别中选择（请参阅下表）。级别越高，数据的完整性就越好。但是，必须应用更多的锁，这将对其他进程产生负面影响。快照是一个特殊情况，因为它创建多个行的副本以避免锁定，但这将增加数据库的大小，同时发生事务。

+   **D 是持久的**。如果在事务期间发生故障，可以进行恢复。这通常是作为两阶段提交和事务日志实现的。一旦事务提交，即使有后续错误，它也保证会持久。持久的相反是易失性的。

## 使用隔离级别控制事务

开发人员可以通过设置**隔离级别**来控制事务，如下表所述：

| 隔离级别 | 锁 | 允许的完整性问题 |
| --- | --- | --- |
| `ReadUncommitted` | 无 | 脏读、不可重复读和幻读 |
| `ReadCommitted` | 在编辑时，它会对记录应用读锁，阻止其他用户在事务结束之前读取记录 | 可重复读和幻读 |
| `RepeatableRead` | 在读取时，它会对记录应用编辑锁，阻止其他用户在事务结束之前编辑记录 | 幻读 |
| `Serializable` | 应用键范围锁，以防止任何可能影响结果的操作，包括插入和删除 | 无 |
| `Snapshot` | 无 | 无 |

## 定义显式事务

您可以使用数据库上下文的`Database`属性来控制显式事务：

1.  在`Program.cs`中，导入 EF Core 存储命名空间以使用`IDbContextTransaction`接口，如下面的代码所示：

```cs

using

Microsoft.EntityFrameworkCore.Storage; // IDbContextTransaction

```

1.  在`DeleteProducts`方法中，在`db`变量实例化之后，添加语句以启动显式事务并输出其隔离级别。在方法底部，提交事务，并关闭括号，如下面代码中所示：

```cs

static

int

DeleteProducts

(

string

name

)

{

using

(Northwind db = new

())

{

**using**

**(IDbContextTransaction t = db.Database.BeginTransaction())**

**{**

**WriteLine(**

**"事务隔离级别：{0}"**

**，**

**arg0: t.GetDbTransaction().IsolationLevel);**

IQueryable<Product>? products = db.Products?.Where(

p => p.ProductName.StartsWith(name));

if

(products is

null

)

{

WriteLine("No products found to delete."

);

return

0

;

}

else

{

db.Products.RemoveRange(products);

}

int

affected = db.SaveChanges();

**t.Commit();**

return

affected;

**}**

}

}

```

1.  运行代码并使用 SQL Server 查看结果，如下面的输出所示：

```cs

事务隔离级别：ReadCommitted

```

1.  运行代码并使用 SQLite 查看结果，如下面的输出所示：

```cs

事务隔离级别：Serializable

```

# Code First EF Core models

有时您可能没有现有的数据库。相反，您将 EF Core 模型定义为 Code First，然后 EF Core 可以使用 create 和 drop API 生成匹配的数据库。

**良好实践**：create 和 drop API 应该只在开发过程中使用。一旦发布应用程序，您不希望它删除生产数据库！

例如，我们可能需要创建一个用于管理学生和学院课程的应用程序。一个学生可以报名参加多门课程。一门课程可以由多名学生参加。这是学生和课程之间的多对多关系的一个例子。

让我们对这个例子建模：

1.  使用您喜欢的代码编辑器向`Chapter10`解决方案/工作区添加一个名为`CoursesAndStudents`的新控制台应用程序。

1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择。

1.  在 Visual Studio Code 中，选择`CoursesAndStudents`作为活动的 OmniSharp 项目。

1.  在`CoursesAndStudents`项目中，为以下包添加包引用：

+   `Microsoft.EntityFrameworkCore.Sqlite`

+   `Microsoft.EntityFrameworkCore.SqlServer`

+   `Microsoft.EntityFrameworkCore.Design`

1.  构建`CoursesAndStudents`项目以恢复包。

1.  添加名为`Academy.cs`，`Student.cs`和`Course.cs`的类。

1.  修改`Student.cs`，注意它是一个没有属性装饰类的 POCO（普通的 CLR 对象），如下面的代码所示：

```cs

namespace

CoursesAndStudents

;

public

class

Student

{

public

int

StudentId { get

; set

; }

public

string

? FirstName { get

; set

; }

public

string

? LastName { get

; set

; }

public

ICollection<Course>? Courses { get

; set

; }

}

```

1.  Modify `Course.cs` , and note that we have decorated the `Title` property with some attributes to provide more information to the model, as shown in the following code:

```cs

using

System.ComponentModel.DataAnnotations;

namespace

CoursesAndStudents

;

public

class

Course

{

public

int

CourseId { get

; set

; }

[Required

]

[StringLength(60)

]

public

string

? Title { get

; set

; }

public

ICollection<Student>? Students { get

; set

; }

}

```

1.  Modify `Academy.cs` , as shown in the following code:

```cs

using

Microsoft.EntityFrameworkCore;

using

static

System.Console;

namespace

CoursesAndStudents

;

public

class

Academy

: DbContext

{

public

DbSet<Student>? Students { get

; set

; }

public

DbSet<Course>? Courses { get

; set

; }

protected

override

void

OnConfiguring

(

DbContextOptionsBuilder optionsBuilder

)

{

string

path = Path.Combine(

Environment.CurrentDirectory, "Academy.db"

);

WriteLine($"Using

{path}

database file."

);

optionsBuilder.UseSqlite($"Filename=

{path}

"

);

// optionsBuilder.UseSqlServer(@"Data Source=.;Initial Catalog=Academy;Integrated Security=true;MultipleActiveResultSets=true;");

}

protected

override

void```

OnModelCreating

(

ModelBuilder modelBuilder

)

{

// Fluent API validation rules

modelBuilder.Entity<Student>()

.Property(s => s.LastName).HasMaxLength(30

).IsRequired();

// populate database with sample data

Student alice = new

() { StudentId = 1

,

FirstName = "Alice"

, LastName = "Jones"

};

Student bob = new

() { StudentId = 2

,

FirstName = "Bob"

, LastName = "Smith"

};

Student cecilia = new

() { StudentId = 3

,

FirstName = "Cecilia"

, LastName = "Ramirez"

};

Course csharp = new

()

{

CourseId = 1

,

Title = "C# 10 and .NET 6"

,

};

Course webdev = new

()

{

CourseId = 2

,

Title = "Web Development"

,

};

Course python = new

()

{

CourseId = 3

,

Title = "Python for Beginners"

,

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

new

{ CoursesCourseId = 1

, StudentsStudentId = 1

},

new

{ CoursesCourseId = 1

, StudentsStudentId = 2

},

new

{ CoursesCourseId = 1

, StudentsStudentId = 3

},

// only Bob signed up for Web Dev

new

{ CoursesCourseId = 2

, StudentsStudentId = 2

},

// only Cecilia signed up for Python

new

{ CoursesCourseId = 3

, StudentsStudentId = 3

}

));

}

}

```

**Good Practice** : Use an anonymous type to supply data for the intermediate table in a many-to-many relationship. The property names follow the naming convention `NavigationPropertyNamePropertyName` , for example, `Courses` is the navigation property name and `CourseId` is the property name so `CoursesCourseId` will be the property name of the anonymous type.

1.  In `Program.cs` , at the top of the file, import the namespace for EF Core and working with tasks, and statically import `Console` , as shown in the following code:

```cs

using

Microsoft.EntityFrameworkCore; // for GenerateCreateScript()

using

CoursesAndStudents; // Academy

using

static

System.Console;

```cs

1.  In `Program.cs` , add statements to create an instance of the `Academy` database context and use it to delete the database if it exists, create the database from the model and output the SQL script it uses, and then enumerate the students and their courses, as shown in the following code:

```cs

using

(Academy a = new

())

{

bool

deleted = await

a.Database.EnsureDeletedAsync();

WriteLine($"Database deleted:

{deleted}

"

);

bool

created = await

a.Database.EnsureCreatedAsync();

WriteLine($"Database created:

{created}

"

);

WriteLine("SQL script used to create database:"

);

WriteLine(a.Database.GenerateCreateScript());

foreach

(Student s in

a.Students.Include(s => s.Courses))

{

WriteLine("{0} {1}参加以下{2}课程："

,

s.FirstName, s.LastName, s.Courses.Count);

foreach

(Course c in

s.Courses)

{

WriteLine($"

{c.Title}

"

);

}

}

}

```

1.  运行代码，并注意第一次运行代码时，它不需要删除数据库，因为数据库尚不存在，如下所示：

```cs

使用 C:\Code\Chapter10\CoursesAndStudents\bin\Debug\net6.0\Academy.db 数据库文件。

数据库已删除：False

数据库已创建：True

用于创建数据库的 SQL 脚本：

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

Alice Jones 参加以下 1 门课程：

C#

10 and .NET 6

Bob Smith 参加以下 2 门课程：

C#

10 and .NET 6

Web Development

Cecilia Ramirez 参加以下 2 门课程：

C#

10 and .NET 6

Python for Beginners

```

注意以下内容：

+   `Title`列是`NOT NULL`，因为模型被装饰为`[Required]`。

+   `LastName`列是`NOT NULL`，因为模型使用了`IsRequired()`。

+   创建了一个名为`CourseStudent`的中间表，用于保存哪些学生参加了哪些课程。

1.  使用 Visual Studio Server Explorer 或 SQLiteStudio 连接到`Academy`数据库并查看表，如*图 10.6*所示：![](img/Image00094.jpg)

图 10.6：使用 Visual Studio 2022 Server Explorer 在 SQL Server 中查看 Academy 数据库

## 理解迁移

在发布使用数据库的项目后，您很可能需要稍后更改实体数据模型，因此需要更改数据库结构。在那时，您不应该使用`Ensure`方法。相反，您需要使用一种允许您逐步更新数据库架构并保留数据库中任何现有数据的系统。EF Core 迁移就是这样的系统。

迁移变得复杂很快，因此超出了本书的范围。您可以在以下链接阅读有关它们的信息：[`docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/`](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/)

# 练习和探索

通过回答一些问题来测试您的知识和理解，进行一些实践，并深入研究本章的主题。

## 练习 10.1 – 测试您的知识

回答以下问题：

1.  例如，数据库上下文的`Products`属性，您会使用什么类型来表示一个表？

1.  例如，`Category`实体的`Products`属性，您会使用什么类型来表示一对多的关系？

1.  主键的 EF Core 约定是什么？

1.  何时可能在实体类中使用注释属性？

1.  为什么您可能会选择使用 Fluent API 而不是注释属性？

1.  `Serializable`事务隔离级别是什么意思？

1.  `DbContext.SaveChanges()`方法返回什么类型？

1.  贪婪加载和显式加载之间有什么区别？

1.  您应该如何定义 EF Core 实体类以匹配以下表？

```cs

CREATE

表

Employees(

EmpId INT

IDENTITY

，

FirstName NVARCHAR

(40

）NOT

NULL

，

薪水 MONEY

)

```

1.  从将实体导航属性声明为“虚拟”中获得的好处是什么？

## 练习 10.2 – 使用不同的序列化格式练习导出数据

在`Chapter10`解决方案/工作区中，创建一个名为`Exercise02`的控制台应用程序，该应用程序查询 Northwind 数据库的所有类别和产品，然后使用.NET 提供的至少三种序列化格式对数据进行序列化。哪种序列化格式使用的字节数最少？

## 练习 10.3 – 探索主题

使用以下页面上的链接详细了解本章涵盖的主题：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-10---working-with-data-using-entity-framework-core`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-10---working-with-data-using-entity-framework-core)

## 练习 10.4 – 探索 NoSQL 数据库

本章重点介绍了诸如 SQL Server 和 SQLite 之类的 RDBMS。如果您希望了解有关 NoSQL 数据库（如 Cosmos DB 和 MongoDB）以及如何在 EF Core 中使用它们的更多信息，则我建议查看以下链接：

+   **欢迎使用 Azure Cosmos DB**：[`docs.microsoft.com/en-us/azure/cosmos-db/introduction`](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction)

+   **使用 NoSQL 数据库作为持久性基础设施**：[`docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/nosql-database-persistence-infrastructure`](https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/nosql-database-persistence-infrastructure)

+   **Entity Framework Core 的文档数据库提供程序**：[`github.com/BlueshiftSoftware/EntityFrameworkCore`](https://github.com/BlueshiftSoftware/EntityFrameworkCore)

# 摘要

在本章中，您学习了如何连接到现有数据库，如何执行简单的 LINQ 查询并处理结果，如何使用过滤包含，如何添加，修改和删除数据，以及如何为现有数据库（如 Northwind）构建实体数据模型。您还学习了如何定义 Code First 模型并使用它来创建新数据库并用数据填充它。

在下一章中，您将学习如何编写更高级的 LINQ 查询，以选择、过滤、排序、连接和分组。
