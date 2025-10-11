# 第十章：*第十章*：设置我们的数据库项目

在本章和接下来的两章中，我们将提高基于数据库的应用程序的性能。在本章中，我们将设置我们的关系数据库和访问该数据库的代码。在下一章中，我们将编写基准测试来测试不同框架的性能，这些框架包括 Entity Framework、Dapper 和 ADO.NET。最后，在*第十二章* *响应式用户界面*中，我们将学习如何提高 SQL Server 和 Cosmos DB 的性能。

数据在我们的日常生活中被广泛使用。在当今大数据的世界中，收集和存储用于各种分析的数据量是巨大的。当处理数据时，随着数据量的增长，性能会呈指数级下降。而且，根据你需要处理的数据量，时间往往是关键因素。

本章中，我们将创建一个数据库并填充它，并编写代码以访问数据库并执行插入、更新、选择和删除操作。我们的数据库访问代码将包括 Entity Framework、Dapper.NET 和 ADO.NET。

注意

本章将不讨论代码性能改进。我们只关注为下一章将要进行的基准测试设置数据库和源代码。

本章将涵盖以下主题：

+   创建和填充 SQL Server 数据库

+   编写使用 Entity Framework 访问数据库的代码

+   编写使用 Dapper.NET 访问数据库的代码

+   编写使用 ADO.NET 访问数据库的代码

完成本章后，你将能够做到以下：

+   登录 SQL Server Management Studio 并执行数据库创建和初始化脚本

+   在开发时将秘密存储在 `secrets.json` 中，以确保秘密不会存储在版本控制中

+   使用 Entity Framework 访问 SQL Server 数据库并执行 **创建/插入、读取/选择、更新和删除** （**CRUD**）操作

+   使用 Dapper.NET 访问 SQL Server 数据库并执行 CRUD 操作

+   使用 ADO.NET 访问 SQL Server 数据库并执行 CRUD 操作

# 技术要求

要跟随本章内容，你需要确保以下条件：

+   SQL Server 2019 Express Edition 或更高版本

+   SQL Server Management Studio

+   Visual Studio 2022

+   本书源代码：[`github.com/PacktPublishing/High-Performance-Programming-in-CSharp-and-.NET/tree/master/CH10`](https://github.com/PacktPublishing/High-Performance-Programming-in-CSharp-and-.NET/tree/master/CH10)

# 设置我们的数据库

在本节中，我们将设置我们的数据库并使我们的项目准备好进行基准测试。我们将基准测试不同的插入、更新、选择和删除数据的方法。让我们从设置数据库开始：

1.  访问 [`github.com/Microsoft/sql-server-samples/tree/master/samples/databases/northwind-pubs`](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/northwind-pubs)。

1.  下载 `instnwnd.sql` 文件。

1.  一旦文件下载完成，在 SQL Server Management Studio 中打开它。

1.  执行文件。这将安装数据库。

1.  打开一个新的查询窗口，并输入以下 SQL 代码：

    ```cs
    USE [Northwind]
    GO
    SET ANSI_NULLS ON
    GO
    SET QUOTED_IDENTIFIER ON
    GO
    CREATE PROCEDURE [dbo].[InsertProduct] 
        @ProductName NVARCHAR(40),
        @CategoryID INT,
        @SupplierID INT,
        @Discontinued BIT
    AS
    BEGIN
    SET NOCOUNT ON;
    INSERT INTO 
            Products (
                ProductName,
                CategoryID,
                SupplierID,
                Discontinued,
                 QuantityPerUnit
            )
        VALUES (
            @ProductName,
            @CategoryID,
            @SupplierID,
            @Discontinued,
             '1'
        )
    END
    GO
    ```

一旦输入代码，执行脚本。此代码生成 `InsertProduct` 存储过程。此存储过程将产品插入到 `Northwind` 数据库的 `Products` 表中。

1.  将现有的 SQL 替换为以下 SQL：

    ```cs
    USE [Northwind]
    GO
    SET ANSI_NULLS ON
    GO
    SET QUOTED_IDENTIFIER ON
    GO
    CREATE PROCEDURE [dbo].[GetProductName]
        @ProductName NVARCHAR(40)
    AS
    BEGIN
        SET NOCOUNT ON;
        SELECT 
            Top 1 ProductName 
        FROM 
            Products
        WHERE
            ProductName LIKE @ProductName
    END
    GO
    ```

执行 SQL 生成 `GetProductName` 存储过程。产品名称可能有不同的变体。此存储过程获取给定产品的顶级 1 个名称。

1.  将现有的 SQL 代码替换为以下 SQL：

    ```cs
    USE [Northwind]
    GO
    SET ANSI_NULLS ON
    GO
    SET QUOTED_IDENTIFIER ON
    GO
    CREATE PROCEDURE [dbo].[FilterProducts]
        @ProductName NVARCHAR(40)
    AS
    BEGIN
            SET NOCOUNT ON;
            SELECT 
                * 
            FROM 
                Products
            WHERE
                ProductName LIKE @ProductName
    END
    GO
    ```

执行 SQL 生成 `FilterProducts` 存储过程。该存储过程返回所有名称包含搜索词的产品。

1.  现在，将现有的 SQL 替换为以下 SQL：

    ```cs
    USE [Northwind]
    GO
    SET ANSI_NULLS ON
    GO
    SET QUOTED_IDENTIFIER ON
    GO
    CREATE PROCEDURE [dbo].[UpdateProductName]
            @OldProductName NVARCHAR(40),
            @NewProductName NVARCHAR(40)
    AS
    BEGIN
        SET NOCOUNT ON;
         UPDATE 
             Products
             SET 
                ProductName = @NewProductName
             WHERE
                ProductName = @OldProductName
    END
    GO
    ```

执行此 SQL 生成 `UpdateProductName` 存储过程。此过程将产品名称从当前名称更新为新名称。

1.  将现有的 SQL 替换为以下内容：

    ```cs
    USE [Northwind]
    GO
    SET ANSI_NULLS ON
    GO
    SET QUOTED_IDENTIFIER ON
    GO
    CREATE PROCEDURE [dbo].[DeleteProduct]
        @ProductName NVARCHAR(40)
    AS
    BEGIN
        SET NOCOUNT ON;
         DELETE FROM 
                Products
         WHERE
                ProductName = @ProductName
    END
    GO
    ```

执行此代码以生成 `DeleteProduct` 存储过程。此过程从数据库中删除与给定产品名称匹配的产品。

1.  一旦数据库安装完成，所有存储过程都已编写并执行，您可以关闭 SQL Server Management Studio。

现在我们已经设置了数据库，我们将设置数据库访问项目。

# 设置我们的数据库访问项目

在本节中，我们将创建我们的数据库访问项目和类。在下一章中，我们将编写一些基准测试，这些基准测试将引用我们在本章中编写的类。按照以下方式创建项目：

1.  打开 Visual Studio 并创建一个名为 `CH10_DataAccessBenchmarks` 的 .NET 6.0 控制台应用程序。

1.  添加 `Microsoft.EntityFrameworkCore.SqlServer` NuGet 包的最新版本。

1.  添加 `Dapper` NuGet 包的最新版本。

1.  添加 `System.Data.SqlClient` NuGet 包的最新版本。

1.  添加一个名为 `Configuration` 的新文件夹，并添加两个类名为 `DatabaseSettings` 和 `SecretsManager`。

1.  添加一个名为 `Data` 的文件夹，并添加三个类名为 `AdoDotNetData`、`DapperDotNet` 和 `EntityFrameworkCoreData`。

1.  添加一个名为 `Models` 的文件夹，并添加三个类名为 `Product`、`SqlCommandModel` 和 `SqlCommandParameterModel`。

1.  添加一个名为 `Reflection` 的文件夹，并添加一个名为 `Properties` 的类。

1.  在主根目录下添加一个名为 `BenchmarkTests` 的类。

1.  保存项目。

因此，我们已经创建并更新了我们的数据库，其中包含了我们将要调用的存储过程，我们还建立了项目、文件夹和类文件，我们将使用这些文件来基准测试我们在数据库上从代码中通常执行的各种数据操作。让我们先从编写`Properties`类开始。

## 编写`Properties`类

作为基准测试的一部分，我们需要获取`DbDataRecord`的`FieldCount`值。但是，没有使用反射，该属性无法直接访问。因此，为了使我们的工作更简单，我们将编写一个名为`Properties`的类，帮助我们通过反射轻松获取属性的值。按照以下步骤操作：

1.  打开`Properties`类并添加以下`using`语句：

    ```cs
    using System.Data.Common;
    using System.Reflection;
    internal class Properties 
    {
    }
    ```

我们需要导入这两个命名空间，因为我们使用反射并需要访问`DbDataRecord`类。

1.  添加`GetProperty`方法：

    ```cs
    public static PropertyInfo GetProperty<T>(string name)
    {
          return typeof(T).GetProperty(name);
    }
    ```

此方法接受一个泛型类型和一个属性名称。然后，它获取该属性并将其作为`PropertyInfo`实例返回。

1.  现在，添加`GetValue`方法：

    ```cs
    public static T GetValue<T, U>(U source, string name)
    {
          return (T)GetProperty<U>(name).GetValue(source);
    }
    ```

此方法接受一个泛型对象类型、返回类型和属性名称。然后，它通过传递泛型对象类型和属性名称调用`GetProperty`方法。然后调用`GetValue`方法，传递源对象。结果被转换为泛型返回类型并返回给调用者。

1.  添加`GetFieldCount`方法：

    ```cs
        public static int GetFieldCount(DbDataRecord 
            record)
        {
            return GetValue<int, DbDataRecord>(
            record, "FieldCount"
        );
    }
    ```

此方法接受一个`DbDataRecord`对象。它通过传递返回类型、我们的`DbDataRecord`和我们的`FieldCount`属性名称调用我们的`GetValue`方法。返回一个整数，包含我们的`DbDataRecord`对象具有的字段数。

因此，我们已经创建了我们的`Properties`类。作为基准测试的一部分，我们将从 SQL Server 数据库中插入、读取、编辑和删除数据。因此，在下一节中，我们将更新我们的`DatabaseSettings`类。

## 编写`DatabaseSettings`类

我们的`DatabaseSettings`类非常简单：它包含一个单一属性。打开数据库并添加以下属性：

```cs
public string ConnectionString { get; set; }
```

此属性持有我们的 SQL Server 数据库的连接字符串。我们将在每个基准测试方法中设置此属性。然后，它将被传递到我们的数据访问类的构造函数中。

由于数据库连接字符串是一种敏感的数据形式，应该非常私密地保存，我们在开发过程中将数据库连接字符串存储在`secrets.json`文件中。但在生产中，我们将从`appsettings.json`文件中获取连接字符串。因此，在下一节中，我们将编写一个`SecretsManager`类。

# 编写`SecretsManager`

在本节中，我们将更新我们的`SecretsManager`类，以便我们可以安全地获取秘密。

注意

我们的开发环境将使用`secrets.json`文件。这非常重要，因为之前已经在源代码托管网站上发现了并访问了私有凭证，我们不希望成为检查包含应保持私密的密钥的代码的责任人。

按照以下步骤操作：

1.  添加以下 NuGet 包：

    ```cs
    Microsoft.Extensions.Configuration
    Microsoft.Extensions.Configuration.JsonFile
    Microsoft.Extensions.Configuration.EnvironmentVariables
    Microsoft.Extensions.Configuration.UserSecrets
    ```

我们需要这些包，以便我们可以为用户密钥和`appsettings.json`配置项目。

1.  打开`SecretsManager`类，并添加以下`using`语句：

    ```cs
    using Microsoft.Extensions.Configuration;
    using System;
    using System.IO;
    ```

我们需要这些`using`语句来访问我们的属性、文件系统、环境变量以及访问 Microsoft 的`IConfiguration`接口。

1.  添加`Configuration`属性：

    ```cs
    public static IConfiguration Configuration 
    {
          get; private set; 
    }
    ```

此属性将保存正确的配置对象，这取决于我们是在开发模式还是生产模式。

1.  现在，添加`GetSecrets`方法：

    ```cs
    public static string GetSecrets<T>(string sectionName)
    where T : class
    {
    var devEnvironmentVariable = 
        Environment
            .GetEnvironmentVariable("NETCORE_ENVIRONMENT");
    var isDevelopment = 
        string.IsNullOrEmpty(devEnvironmentVariable) 
        || devEnvironmentVariable.ToLower() == "development";
    var builder = new ConfigurationBuilder()
        .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile(
                "appsettings.json", 
                optional: true, 
                reloadOnChange: true
        )
        .AddEnvironmentVariables();
    //only add secrets in development
    if (isDevelopment) 
    {
        builder.AddUserSecrets<T>();
    }
    Configuration = builder.Build();
    return Configuration.GetSection($"{typeof(T).Name}
          :{sectionName}").Value;
    }
    ```

此方法确定我们是在开发模式还是非开发模式。如果我们处于开发模式，则使用密钥配置模式。否则，我们从`appsettings.json`文件中获取密钥。该方法接受一个部分名称，这是我们想要检索的密钥的名称，并返回该密钥的值。

这样，我们已经完成了`secrets`类的编写。对于我们的数据操作基准，我们将专注于单个表——`Northwind`数据库的`Products`表。我们需要一个充当数据模型的类。因此，在下一节中，我们将编写`Product`类。

# 编写 Product 类

在本节中，我们将更新我们的`Product`类。它是一个用于数据操作基准的简单对象，包含与`Northwind`数据库中`Products`表相对应的属性。按照以下步骤操作：

1.  打开`Product`类，并按以下方式更新它：

    ```cs
    using System;
    using System.ComponentModel.DataAnnotations;
    using System.ComponentModel.DataAnnotations.Schema;
    [Table("Products")]
    public class Product
    {
    }
    ```

在这里，我们使用`Table`注解注解了我们的类，将此类映射到`Northwind`数据库中表的名称传递给注解。

1.  添加以下属性和注解：

    ```cs
    [Key]
    public int ProductID { get; set; }
    public string ProductName { get; set; }
    [ForeignKey("Suppliers")]
    public int SupplierID { get; set; }
    [ForeignKey("Categories")]
    public int CategoryID { get; set; }
    public string QuantityPerUnit { get; set; } = "1"
    public decimal UnitPrice { get; set; }
    public Int16 UnitsInStock { get; set; }
    public Int16 UnitsOnOrder { get; set; }
    public Int16 ReorderLevel { get; set; }
    public bool Discontinued { get; set; }
    ```

这些属性与`Northwind`数据库中`Product`表的列相对应。`[Key]`注解将`ProductID`属性标识为表的主键。两个外键通过`[ForeignKey]`注解进行标识。我们将表名传递给此注解，其中包含主键。

就这样——我们已经完成了`Product`类的编写。在访问数据时，我们将使用多个命令和参数。为了简化操作，我们将有一个`SqlCommandModel`类来定义我们的命令，以及一个`SqlCommandParameterModel`类来定义我们的命令参数。让我们先编写`SqlCommandModel`类。

# 编写 SqlCommandModel 类

在本节中，我们将编写一个简单的类来模拟 SQL 命令。按照以下步骤操作：

1.  打开`SqlCommandModel`类，将其定义为公共类，并添加`System.Data`命名空间。

1.  现在，添加以下三个属性：

    ```cs
    public string CommandText { get; set; }
    public CommandType CommandType { get; set; }
    public SqlCommandParameterModel[] CommandParameters { 
        get; set; }
    ```

`CommandText` 属性包含我们的 SQL 命令。这可能是一个存储过程的名称或 SQL 语句。`CommandType` 属性确定命令是 `Text` 命令还是 `StoredProcedure` 命令，而 `CommandParameters` 属性包含 SQL 命令参数的数组。

现在我们已经编写了 `SqlCommandModel`，接下来让我们编写 `SqlCommandParameterModel` 类。

# 编写 SqlCommandParameterModel 类

在本节中，我们将编写我们的 `SqlCommandParameterModel` 类。这个类仅仅是一个 SQL 参数定义模型。

打开 `SqlCommandParameterModel` 类，将其设置为公共类，并添加 `System.Data` 命名空间。

现在，添加以下三个参数：

```cs
public string ParameterName { get; set; }
```

```cs
public DbType DataType { get; set; }
```

```cs
public dynamic Value { get; set; }
```

此类模拟了一个标准参数，包括参数的名称、其数据库类型和其值。

通过这样，我们已经创建了数据访问类中所需的核心功能。在接下来的章节中，我们将编写数据访问类，以使用 Entity Framework、Dapper 和 ADO.NET 访问数据。

选择 SQL Server 作为数据库服务器的原因是它是最常见的数据库服务器之一，并且在全球许多商业场景中被使用。在采用 SQL Server 的专业环境中，最常用的三种数据访问方法是 Entity Framework、Dapper 和 ADO.NET。这就是为什么我们将在本章中对其进行基准测试。让我们先编写我们的 ADO.NET 数据访问类。

# 编写 AdoDotNet 类

在本节中，我们将编写我们的数据插入方法。但是，我们不会运行基准测试，这些基准测试将在下一章中进行分析时进行。请按照以下步骤操作：

1.  更新 `AdoDotNetData` 类，如下所示：

    ```cs
    using CH10_DataAccessBenchmarks.Models;
    using CH10_DataAccessBenchmarks.Reflection;
    using System;
    using System.Collections;
    using System.Collections.Generic;
    using System.Data.Common;
    using System.Data.SqlClient;
    using System.Reflection;
    internal class AdoDotNetData : IDisposable
    {
    private readonly SqlConnection _sqlConnection;
    private bool _isDisposed;
    public AdoDotNetData(string connectionString)
    {
              _sqlConnection = 
               new SqlConnection(connectionString);
    }
    public void Dispose()
    {
        Dispose(_isDisposed);
    }
    public void Dispose(bool disposing)
    {
            if (disposing)
            {
            _sqlConnection.Dispose();
            _isDisposed = true;
        }
    }
    }
    ```

在前面的代码中，我们实现了 `IDisposable` 模式。当我们完成我们的类时，我们销毁我们的类，这也销毁了它所持有的内存中的可处置对象。

1.  添加 `ExecuteNonQuery` 方法：

    ```cs
        internal void ExecuteNonQuery(SqlCommandModel 
            model)
        {
            SqlCommand sqlCommand 
             = new (model.CommandText, _sqlConnection);
         sqlCommand.CommandType = model.CommandType;
         foreach (SqlCommandParameterModel parameter in 
            model.CommandParameters)
         sqlCommand.Parameters.Add(new SqlParameter()
            {
                ParameterName = parameter.ParameterName,
                DbType = parameter.DataType,
                Value = parameter.Value
         });
         _sqlConnection.Open();
         sqlCommand.ExecuteNonQuery();
         _sqlConnection.Close();
    }
    ```

此方法接受一个 `SqlCommandModel` 对象。在实例化过程中创建了一个新的 `SqlCommand` 对象实例。我们将 SQL 命令和 SQL 连接传递给构造函数。然后，我们遍历命令参数，为每个 `model.CommandParameter` 实例化并添加一个 `SqlParameter` 到 `sqlCommand` 对象中。接下来，我们打开数据库连接，执行查询，并关闭连接。

1.  添加以下代码：

    ```cs
    internal int ExecuteNonQuery(string sql)
    {
    try
    {
    _sqlConnection.Open();
    return new SqlCommand(sql, _sqlConnection)
        .ExecuteNonQuery();
    }
    finally
    {
    _sqlConnection.Close();
    }
    }
    ```

上述代码执行通过 `sql` 字符串传入的非查询 SQL 代码。

1.  添加以下通用标量方法：

    ```cs
    internal T ExecuteScalar<T>(string sql)
    {
         try
         {
            _sqlConnection.Open();
            return (T)new SqlCommand(sql, _sqlConnection)
                .ExecuteScalar();
    }
        finally
        {
            _sqlConnection.Close();
    }
    }
    ```

此方法接受一个 SQL 命令作为字符串。打开数据库连接，并实例化一个新的 `SqlCommand`。执行 `ExecuteScalar` 命令，从数据库返回单个值。在返回值之前，将其转换为调用者指定的泛型类型，并以该类型返回。然后关闭连接。

1.  添加以下标量方法：

    ```cs
    internal T ExecuteScalar<T>(SqlCommandModel model)
    {
    SqlCommand sqlCommand = new(
         model.CommandText, _sqlConnection);
    sqlCommand.CommandType = model.CommandType;
        foreach (SqlCommandParameterModel parameter in 
            model.CommandParameters)
            sqlCommand.Parameters.Add(new SqlParameter()
            {
                ParameterName = parameter.ParameterName,
                DbType = parameter.DataType,
                Value = parameter.Value
         });
      _sqlConnection.Open();
        T data = (T)sqlCommand.ExecuteScalar();
        _sqlConnection.Close();
        return data;
    }
    ```

此方法接收一个 `SqlCommandModel` 并使用它来构建一个 `SqlCommand`。通过调用 `ExecuteScalar` 方法执行 `SqlCommand` 类，并在返回之前将其转换为泛型类型。

1.  添加以下读取方法：

    ```cs
    internal IEnumerator<T> ExecuteReader<T>(string sql)
    {
        Type TypeT = typeof(T);
        ConstructorInfo ctor = 
          TypeT.GetConstructor(Type.EmptyTypes);
    if (ctor == null)
        {
    throw new InvalidOperationException($"Type 
        {TypeT.Name} does not have a default
            constructor.");
    }
        _sqlConnection.Open();
    IEnumerator data = new SqlCommand(sql, _sqlConnection)
     .ExecuteReader().GetEnumerator();    
    while (data.MoveNext())
        {
            T newInst = (T)ctor.Invoke(null);
            DbDataRecord record = (DbDataRecord)
                    data.Current;
            int fieldCount = Properties
                 .GetFieldCount((DbDataRecord)
                    data.Current);
         for (int i = 0; i < fieldCount; i++)
            {
                string propertyName = record.GetName(i);
                PropertyInfo propertyInfo = TypeT
                     .GetProperty(propertyName);
                 if (propertyInfo != null)
                {
                    object value = record[i];
                    if (value == DBNull.Value)
                        propertyInfo
                             .SetValue(newInst, null);
                     else
                        propertyInfo
                             .SetValue(newInst, value);
             }
         }
            yield return newInst;
    }
    }
    ```

此方法接收一个 SQL 语句并通过调用 `ExecuteReader` 方法来执行它。一旦方法执行完毕，我们就获得读者的枚举器。然后，我们遍历枚举器并构建当前迭代的对象，并产生结果。

1.  添加以下读取方法：

    ```cs
    internal IEnumerator<T> ExecuteReader<T>
         (SqlCommandModel model) {
    Type TypeT = typeof(T);
    ConstructorInfo ctor 
             = TypeT.GetConstructor(Type.EmptyTypes);
    if (ctor == null) {
    throw new InvalidOperationException($"Type 
        {TypeT.Name} does not have a default
             constructor.");
    }
    SqlCommand sqlCommand 
        = new(model.CommandText, _sqlConnection);
    sqlCommand.CommandType = model.CommandType;
    foreach (SqlCommandParameterModel parameter in 
        model.CommandParameters)
    sqlCommand.Parameters.Add(new SqlParameter() {
    ParameterName = parameter.ParameterName,
    DbType = parameter.DataType, Value = 
        parameter.Value});
    _sqlConnection.Open();
    SqlDataReader reader = sqlCommand.ExecuteReader();
    if (reader.HasRows) {
    while (reader.Read()) {
    T newInst = (T)ctor.Invoke(null);
    for (int i = 0; i < reader.FieldCount; i++) {
         string propertyName = reader.GetName(i);
         PropertyInfo propertyInfo 
             = TypeT.GetProperty(propertyName);
         if (propertyInfo != null) {
             object value = reader[i];
             if (value == DBNull.Value)
                   propertyInfo.SetValue(newInst, null);
             else  
                   propertyInfo.SetValue(newInst, value);
         }
    }        
        yield return newInst;
    }
    }
      _sqlConnection.Close();
    }
    ```

此读取方法接收一个 `SqlCommandModel` 并构建一个 `SqlCommand`。它执行读取操作并获取 `SqlDataReader`。它遍历读取器并构建一个泛型类型的实例，然后将其传递给用户。

这样，我们的 ADO.NET 数据访问类就完成了。现在，让我们学习如何编写 Entity Framework 数据访问类。

# 编写 EntityFrameworkCoreData 类

在本节中，我们将编写我们的 Entity Framework 数据访问类的相关方法。在本节中编写的代码将在下一章中执行。请按照以下步骤操作：

1.  打开 `EntityFrameworkCoreData` 类并按以下方式编辑它：

    ```cs
    using CH10_DataAccessBenchmarks.Models;
    using Microsoft.EntityFrameworkCore;
    using System.Collections.Generic;
    using Microsoft.Data.SqlClient;
    using System.Linq;
    using Microsoft.EntityFrameworkCore.SqlServer
        .Infrastructure.Internal;
    public class EntityFrameworkCoreData : DbContext
    {
        private string _connectionString = string.Empty;
        public DbSet<Product> Products { get; set; }
        public EntityFrameworkCoreData(string 
            connectionString) : base(GetOptions
                 (connectionString))
        {
            _connectionString = connectionString;
        }
        private static DbContextOptions GetOptions(string 
            connectionString)
        {
        return SqlServerDbContextOptionsExtensions
            .UseSqlServer(new DbContextOptionsBuilder(), 
                connectionString).Options;
        }
    ```

我们的类继承自 `Microsoft.EntityFrameworkCore` 库中的 `DbContext` 类。我们声明一个变量来保存我们的数据库连接字符串，以及一个变量来保存 `Products` 集合。在我们的构造函数中，我们设置连接字符串并调用基类构造函数。

1.  添加 `OnConfiguring` 方法：

    ```cs
    protected override void OnConfiguring
         (DbContextOptionsBuilder optionsBuilder)
    {     
          optionsBuilder.UseSqlServer(_connectionString);
    }
    ```

此方法确定我们将使用 SQL Server，并传递我们将使用的 SQL Server 连接字符串。

1.  添加以下方法，它执行原始 SQL：

    ```cs
    public int ExecuteSQL(string sql)
        {
            return Database.ExecuteSqlRaw(sql, null);
    }
    ```

此方法接收一个 SQL 语句并将其作为原始 SQL 在数据库中执行。返回值是受该语句执行影响的记录数。

1.  添加以下方法以执行存储过程作为非查询：

    ```cs
    public int ExecuteNonQuerySP(SqlCommandModel model)
        {
            SqlParameter[] parameters 
            = new SqlParameter[model.CommandParameters
                .Length];
            for (int i = 0; i < parameters.Length; i++)
            {
                parameters[i] = new SqlParameter(
                model.CommandParameters[i].ParameterName,
                 model.CommandParameters[i].Value
                 );
         }
            if (parameters.Length == 4)
                return Database.ExecuteSqlRaw(
                 model.CommandText, parameters[0], 
                 parameters[1], parameters[2], 
                 parameters[3]
             );
         else if (parameters.Length == 2)
             return Database.ExecuteSqlRaw(
                 model.CommandText, parameters[0], 
                 parameters[1]
             );
         else
             return Database.ExecuteSqlRaw(
                 model.CommandText, parameters[0]
             );
    }
    ```

在此方法中，我们从 `SqlCommandModel` 构建一个 `SqlParameter` 数组。然后，通过将每个参数传递给存储过程来执行原始 SQL。此执行是非查询操作，并返回运行该过程影响的行数。

1.  以下方法将执行并返回一个 `string` 类型的标量值：

    ```cs
    public string ExecuteScalarSP(string productName)
        {
            return Products.FromSqlRaw(
                "EXEC FilterProducts @ProductName={0}",
                new SqlParameter() { 
            ParameterName = "@ProductName", Value = 
                    productName })
                .AsEnumerable().FirstOrDefault()
                  .ProductName;
        }
    ```

此方法执行一个带有单个参数的存储过程。我们获取可枚举的返回对象并过滤它以获取第一条记录。然后返回产品名称作为字符串。

1.  向我们的类中添加最终的方法，该方法返回一个枚举器：

    ```cs
    public IEnumerator<Product> ExecuteReaderSP(string 
        productName)
    {
    return Products.FromSqlRaw(
            "EXEC FilterProducts @ProductName={0}", 
            new SqlParameter() { 
                 ParameterName = "@ProductName", 
                 Value = productName 
             }
         ).GetEnumerator();
    }
    ```

此操作执行一个带有单个参数的存储过程，并返回一个充满过滤产品的枚举器。

有了这些，我们已经编写了所有的 Entity Framework 类。现在，是时候编写我们的 Dapper.NET 方法了。

# 编写 DapperDotNet 类

在本节中，我们将编写我们的 Dapper.NET 方法。这是在编写基准测试方法之前的最后一节。我们将运行本节中编写的代码。请按照以下步骤操作：

1.  打开 `DapperDotNet` 类，添加 `SimpleCRUD` 包，并按如下方式修改：

    ```cs
    public class DapperDotNet : IDisposable
    {
        private bool isDisposed = false;
         private IDbConnection _dbConnection;
         public DapperDotNet(string connection)
        {
         SimpleCRUD
                .SetDialect(SimpleCRUD.Dialect.SQLServer);
             _dbConnection = new SqlConnection
                 (connection);
         }
         public void Dispose()
        {
             Dispose(true);
            GC.SuppressFinalize(this);
         }
         protected virtual void Dispose(bool disposing)
        {
            if (isDisposed)
                return;
             if (disposing)
                 _dbConnection.Dispose();
             isDisposed = true;
    }
    }
    ```

我们在这个类中实现了 `IDisposable` 模式，并将 SQL 方言设置为 SQL Server。

1.  添加以下非查询方法：

    ```cs
        public int ExecuteNonQuery(string sql)
        {
            try
            {
                _dbConnection.Open();
                return _dbConnection.Execute(sql);
         }
            finally
            {
                _dbConnection.Close();
         }
        }
    ```

此方法执行原始 SQL 并返回受 SQL 语句影响的记录数。

1.  添加以下方法以执行非查询操作：

    ```cs
    public void ExecuteNonQuery(SqlCommandModel model)
        {
            try
            {
                _dbConnection.Open();
             var parameters = new DynamicParameters();
                foreach (
                 SqlCommandParameterModel parameter in  
                 model.CommandParameters
             ) 
                 parameters.Add(
                     parameter.ParameterName, 
                     parameter.Value
                 );
                 _dbConnection.Query(
                 model.CommandText, 
                 parameters, 
                 commandType: CommandType.StoredProcedure
             );
            }
            finally
         {
                _dbConnection.Close();
            }
    }
    ```

此方法接受一个 `SqlCommandModel` 实例，并构建一个 `DynamicParameter` 包。然后，它执行由模型 `CommandText` 定义的存储过程。

1.  添加以下泛型标量方法：

    ```cs
        public T ExecuteScalar<T>(string sql)
        {
            try
            {
                _dbConnection.Open();
                return _dbConnection.ExecuteScalar
                    <T>(sql);
         }
            finally
            {
                if (_dbConnection != null 
                 && _dbConnection.State 
                     == ConnectionState.Open)
                 _dbConnection.Close();
         }
    }
    ```

此方法接受一个 SQL 语句并执行它，返回所需类型的单个值。

1.  添加以下方法，该方法执行存储过程并返回字符串：

    ```cs
    public string ExecuteScalarSP(SqlCommandModel model)
    {
            try
            {
                _dbConnection.Open();
                var parameters = new DynamicParameters();
                    parameters.Add(
                    model.CommandParameters[0]
                     .ParameterName,   
                 model.CommandParameters[0].Value
             );
                return _dbConnection.Query<Product>(
                 model.CommandText, 
                 parameters, 
                 commandType: CommandType.StoredProcedure
             ).First().ProductName;
         }
            finally
            {
                if (
                 _dbConnection != null 
                 && _dbConnection.State 
                     == ConnectionState.Open)
                 _dbConnection.Close();
         }
    }
    ```

此方法接受一个 `SqlCommandModel` 实例，并使用它来执行存储过程。请记住将缺少的 `using` 语句 `SqlCommandModel` 添加到类中。存储过程执行返回 `IEnumerable<Product>` 类型的数据。因此，我们获取列表中的第一个产品并返回其 `ProductName`。

1.  添加以下方法，该方法执行原始 SQL 并返回 `IEnumerator<T>` 类型的数据：

    ```cs
    public IEnumerator<T> ExecuteReader<T>(string sql) 
        where T : class
    {
             try
            {
                _dbConnection.Open();
                return _dbConnection.Query<T>(sql)
                 .GetEnumerator();
        }
            finally
            {
                if (_dbConnection != null 
                 && _dbConnection.State 
                     == ConnectionState.Open)
                 _dbConnection.Close();
        }
    }
    ```

此方法执行原始 SQL 字符串并返回 `IEnumerable<T>` 类型的数据。

1.  添加以下方法，该方法执行存储过程并返回 `IEnumerator<Product>` 类型的数据：

    ```cs
        public IEnumerator<Product> ExecuteReaderSP
            <Product>(
        SqlCommandModel model
    )
        {
            try
            { 
                _dbConnection.Open();
                var parameters = new DynamicParameters();
                foreach (SqlCommandParameterModel 
                    parameter in model.CommandParameters)
                 parameters.Add(
                     parameter.ParameterName, 
                     parameter.Value
                 );
             return _dbConnection.Query<Product>(
                 model.CommandText, 
                 parameters, 
                 commandType: CommandType.StoredProcedure
             ).GetEnumerator();
         }
            finally
            {
                if (_dbConnection != null 
                 && _dbConnection.State 
                 == ConnectionState.Open)
             _dbConnection.Close();
         }
    }
    ```

此方法接受一个 `SqlCommandModel` 实例，并构建一个要执行的参数化存储过程。返回 `IEnumerator<Product>` 类型的数据。

1.  添加我们的最终 dapper 方法，该方法将获取与 `productName` 参数匹配的第一个产品名称：

    ```cs
    public string GetProductNameSP(string productName)
        {
            try
            {
                _dbConnection.Open();
                var parameters = new DynamicParameters();
                parameters.Add("@ProductName", 
                    productName);
                return _dbConnection.Query<Product>(
                 $"GetProductName", parameters, 
                 commandType: CommandType.StoredProcedure
                 ).First().ProductName;
         }
            finally
            {
                if (_dbConnection != null 
                 && _dbConnection.State 
                 == ConnectionState.Open)
                 _dbConnection.Close();
            }
    }
    ```

此方法接受一个产品名称并执行 `GetProductName` 存储过程。存储过程匹配所有数据库中产品名称与产品名称参数相似的产品。然后，它获取返回列表中的第一个产品并返回其产品名称。

这就完成了我们为下一章将要进行的基准测试工作所做的数据库和数据访问项目设置。让我们回顾一下本章我们取得了哪些成果。

# 摘要

在本章中，我们下载了 `Products` 表。

在确保我们已设置好所需的存储过程后，我们启动了一个 .NET 6.0 控制台应用程序。我们添加了我们的模型类和数据访问类，以在 Entity Framework、Dapper 和 ADO.NET 中执行数据访问操作。

在下一章中，我们将对每个框架的数据访问方法进行基准测试。在 *进一步阅读* 部分中，您可以通过提供的链接进一步了解 Entity Framework、Dapper 和 ADO.NET。

# 进一步阅读

要了解更多关于本章所涉及的主题，请查看以下资源：

+   Entity Framework Core: [`docs.microsoft.com/ef/core/`](https://docs.microsoft.com/ef/core/)

)

+   Dapper: [`dapper-tutorial.net/dapper`](https://dapper-tutorial.net/dapper)

)

+   ADO.NET: [`dotnettutorials.net/course/ado-net-tutorial-for-beginners-and-professionals/`](https://dotnettutorials.net/course/ado-net-tutorial-for-beginners-and-professionals/)
