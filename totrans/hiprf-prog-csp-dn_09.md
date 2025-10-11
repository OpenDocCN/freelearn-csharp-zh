# *第7章*: LINQ 性能

LINQ 以其速度慢而闻名。但与人们的观点相反，有方法可以确保使用 LINQ 的最佳性能。

在本章中，你将学习如何以性能为导向执行 LINQ 查询。根据你如何使用 LINQ，返回相同结果的不同方法可能会有不同的行为和性能。因此，在本章中，你将学习如何最好地执行 LINQ 查询以提高你应用程序的性能。

在这里，你将对确定 LINQ 查询中最后一个元素的最有效方法进行基准测试。你将了解在 LINQ 语句中使用 `let` 关键字的性能惩罚，以及为什么你应该避免使用它。通过基准测试不同的 `Group By` 方法，你将深入了解使用 LINQ 执行 GroupBy 查询的最有效方式。在执行查询和数据操作时，你可能会需要使用闭包。通过编写参数化和非参数化闭包，你会发现参数化闭包的性能远优于非参数化闭包。

在本章中，我们将涵盖以下主题：

+   设置我们的示例数据库

+   设置我们的内存中示例数据

+   使用 LINQ 查询数据库

+   获取集合的最后一个值

+   避免在 LINQ 查询中使用 let 关键字

+   提高 LINQ 查询中 Group By 的性能

+   过滤列表

+   理解闭包

到本章结束时，你将具备使用高效的 LINQ 安全存储秘密、查询数据库和内存中数据所需的技能。你还将能够理解在查询中使用 `let` 关键字对性能的影响，以及如何使用 LINQ 进行有效的过滤和分组数据。

# 技术要求

为了跟上本章的内容，你需要访问以下工具：

+   Visual Studio 2022

+   SQL Server 2019

+   SQL Server Management Studio

+   书籍的源代码：[https://github.com/PacktPublishing/High-Performance-Programming-in-CSharp-and-.NET/tree/master/CH07](https://github.com/PacktPublishing/High-Performance-Programming-in-CSharp-and-.NET/tree/master/CH07)

# 设置示例数据库

在本章中，我们将演示不同的集合接口如何处理数据，为了演示，你需要访问数据库数据。为此，你将创建一个数据库，添加一个表，并用数据填充它。你将使用 SQL Server 作为你的数据库引擎，并使用 SQL Server Management Studio 来开发你的示例数据库。

注意

在 `CH07_LinqPerformance.Data` 源代码文件夹中，你可以找到一个名为 `SampleData.Product.sql` 的数据库创建脚本，该脚本创建数据库并用数据填充。你可以在 SQL Server Management Studio 中运行此脚本。这将让你免于在本节中设置数据库。但如果你是 SQL Server 新手，你可能想运行这一节。

要添加你的数据库，请按照以下步骤操作：

1.  打开 SQL Server Management Studio 并连接到你的数据库引擎。

1.  在**对象资源管理器**中右键单击**数据库**文件夹，如图 7.1 所示：

![图 7.1：SQL Server Management Studio 对象资源管理器选项卡

](img/B16617_07_01.jpg)

图 7.1：SQL Server Management Studio 对象资源管理器选项卡

1.  从上下文菜单中选择**新建数据库**。这将显示如图 7.2 所示的**新建数据库**对话框：

![图 7.2：SQL Server Management Studio 新数据库对话框

](img/B16617_07_02.jpg)

图 7.2：SQL Server Management Studio 新数据库对话框

1.  一旦你为数据库名称输入了`SampleData`，点击**确定**按钮来创建数据库。

1.  通过以下图示展开 `Products` 来定位数据库：

![表 7.1：产品表设计

](img/Table_7.1.jpg)

表 7.1：产品表设计

1.  保存表，然后展开**表**文件夹。右键单击**产品**表，选择**编辑前 n 条记录**，其中 *n* 将是配置的记录数，默认为 *200*。

1.  将以下图中的数据添加到**产品**表中：

![表 7.2：产品表行数据

](img/Table_7.2.jpg)

表 7.2：产品表行数据

我们现在有一个数据库，其中只有一个表，表里填充了我们将要在本章后面使用的数据。在下一节中，我们将添加我们的内存样本数据。

# 设置我们的内存样本数据

由于你将研究 LINQ 性能，因此你需要一个集合来工作。你将使用一个`Person`对象的集合。每个人将被命名为希腊字母。一个`Person`对象将包含一个`FirstName`、`LastName`和`FullName`属性。`FullName`属性将是一个插值字符串，它结合了人的名字和姓氏。

让我们现在开始编写我们的 LINQ 代码，并结合**基准测试**，以便我们可以测量我们的 LINQ 语句的性能：

1.  创建一个名为 `CH07_LinqPerformance` 的新 .NET 6.0 控制台应用程序。

1.  安装 NuGet 包 `BenchmarkDotNet`。

1.  添加以下 `Person` 结构体：

    [PRE0]

此结构定义了具有 `FirstName`、`LastName` 和计算出的 `FullName` 的 `Person`。

1.  现在，添加一个名为 `LinqPerformance` 的新类，并包含以下 `using` 语句：

    [PRE1]

这些 `using` 语句为你提供了访问基准测试、泛型集合和 LINQ 类的权限。

1.  将以下代码添加到类的顶部：

    [PRE2]

你已声明了一个人员列表和两个数组。这两个数组都包含属于那些组的人员的姓氏，且均为小写。

1.  现在，添加一个全局设置类，该类将为基准测试各种 LINQ 查询准备你的集合：

    [PRE3]

现在，你已经为我们将在本章中讨论的主题设置了示例数据库和内存中的示例数据。因此，让我们首先调查查询数据库的各种方法及其对 LINQ 查询性能的影响。

# 数据库查询性能

在 [*第 6 章*](B16617_06_Final_SB_Epub.xhtml#_idTextAnchor117)，“.NET 集合”中，我们看到了 `IEnumerator` 与 `IEnumerable` 的区别，以及当遍历内存中的集合时，`IEnumerator` 的性能如何优于 `IEnumerable`。现在，我们将查询数据库，并使用各种基准技术遍历结果集合。为此，我们将遵循以下步骤：

1.  添加一个名为 `IEnumeratorVsIQueryable` 的新类。

1.  你将连接到 SQL Server 数据库，并将需要保持秘密的信息。你的 `secret.json` 文件不会提交到版本控制。因此，右键单击项目，并从上下文菜单中选择 **管理用户秘密**。

1.  将弹出一个对话框，提示需要安装额外的包。单击 **是**。

![图 7.3：提示需要安装额外包以管理用户秘密的对话框

](img/B16617_07_03.jpg)

图 7.3：提示需要安装额外包以管理用户秘密的对话框

1.  Visual Studio 将在新的标签页中打开 `secrets.json` 文件。这是你添加用户秘密的地方。

1.  打开 **包管理控制台** 并添加以下包：

    [PRE4]

这些包使你能够连接到并从 SQL Server 数据库中提取数据。

1.  使用以下代码更新你的 `secrets.json` 文件，其中包含你在本章开头创建的数据库的连接字符串：

    [PRE5]

此连接字符串将用于连接到你的数据库，执行返回一些数据的查询，并允许你遍历这些数据并对其执行操作。

1.  添加一个名为 `Configuration` 的文件夹，并在该文件夹中添加一个名为 `SecretsManager` 的类，该类具有空的静态构造函数和以下 `using` 语句：

    [PRE6]

你需要这些 `using` 语句来处理文件 I/O 和系统配置，例如从 `secrets.json` 文件中获取秘密。

1.  在 `SecretsManager` 类的顶部添加以下行：

    [PRE7]

此行声明了你的静态配置属性，该属性用于在应用程序中获取配置数据。

1.  现在添加以下代码：

    [PRE8]

此代码获取 .NET Core 环境的环境变量。然后获取代码以查看它是否在软件开发环境中运行或在生产环境中运行。然后为将要运行的环境构建配置。因此，如果我们处于调试模式，配置将为开发环境构建。如果我们处于发布模式，配置将为生产环境构建。如果我们处于开发模式，则添加由 `T` 变量定义的 `secrets` 类。

1.  创建一个新的文件夹，命名为 `Models`，并使用以下代码添加 `Product` 类：

    [PRE9]

我们的 `Product` 类通过 `Id`、`Name` 和 `Description` 属性提供产品数据模型，这些属性通过构造函数设置。我们还重写了 `ToString` 方法以返回属性值的文本表示。

1.  为 `System.ComponentModel.DataAnnotations` 添加一个 `using` 语句。将结构体更改为类，并将 `[Key]` 属性添加到 `Id` 属性。我们需要这些更改，因为我们正在使用 Entity Framework 连接到数据库并提取数据。

1.  在 `CH07_LinqPerformance.Data` 文件夹中添加 `DatabaseContext` 类：

    [PRE10]

我们已经声明了我们的 `DatabaseContext` 类，它继承自 `DbContext` 类。现在我们需要添加其内部实现。

1.  将以下项添加到 `DatabaseContext` 类中：

    [PRE11]

在此代码中，我们声明了我们的产品 `DbSet` 属性，它将包含我们的 `Product` 类的集合，以及一个连接字符串成员变量，它将包含连接到我们的数据库的字符串。然后声明构造函数，它接受一个连接字符串，我们将其传递给 `GetOptions` 方法，然后传递给基类构造函数。

1.  将 `GetOptions` 方法添加到 `DatabaseContext` 类中：

    [PRE12]

此方法返回用于我们的 SQL Server 数据库连接的 `DbContextOptions`。所使用的连接字符串是在开发时存储在我们的 `secrets.json` 文件中，在生产时存储在 `appsettings.json` 中。

1.  添加 `OnModelCreating` 方法：

    [PRE13]

在这里，我们正在配置将在我们的 `DbSet` 中使用的 `Product` 类。我们声明 `Id` 字段是主键，而 `Name` 字段的最大长度为 50，`Description` 字段的最大长度为 255。

1.  将 `DatabaseSettings` 类添加到 `Configuration` 文件夹中：

    [PRE14]

此类有一个名为 `ConnectionString` 的单个属性，它将保存到我们的 `SampleData` 数据库的连接字符串。请注意，类的名称和属性的名称与 JSON 部分和属性的名称相匹配！

1.  现在，将 `appsettings.json` 添加到项目的根目录，并包含以下内容：

    [PRE15]

此文件与 `secrets.json` 文件和 `DatabaseSettings` 类具有相同的布局。此文件用于存储您的连接字符串。在开发中，它设置在秘密文件中，在生产中设置在 Azure 中。现在您已经设置了数据库配置，可以添加基准测试代码。

1.  在项目的根目录中添加一个名为 `DatabaseQueryAndIteration` 的新类，该类实现 `IDisposable` 并具有以下代码：

    [PRE16]

此代码声明了我们的类并定义了它实现了 `IDisposable` 接口。它也被配置为可进行基准测试。

1.  在我们的类中实现 `IDisposable` 接口：

    [PRE17]

此代码释放了我们的托管资源并抑制了对类终结器的调用。

1.  我们已经为在这个类中的方法进行基准测试、访问数据库资源以及清理工作做好了准备。将以下代码添加到类中：

    [PRE18]

`_context` 变量为我们提供了数据库访问权限。`GlobalSetup()` 方法从我们的密钥文件中获取连接字符串，并使用安全存储的连接字符串创建一个新的 `DatabaseContext`。`GlobalSetup()` 方法将在我们的基准测试之前运行。`GlobalCleanup()` 方法在基准测试完成后调用 `Dispose(disposing)` 方法来清理我们的托管资源。

1.  接下来，添加 `QueryDb()` 方法：

    [PRE19]

`QueryDb()` 方法通过选择具有大于 `1` 的 ID 的产品来在数据库上执行一个简单的 LINQ 查询。然后它遍历 `IQueryable<Product>` 列表中的每个产品，并将产品名称写入调试窗口。

1.  现在，添加 `QueryDbAsList()` 方法：

    [PRE20]

`QueryDbAsList()` 执行与 `QueryDb()` 相同的查询，但处理的是 `List<Product>` 类型。

1.  添加 `QueryDbAsIEnumerable()` 方法：

    [PRE21]

`QueryDbAsIEnumerable()` 方法执行与 `QueryDbAsList` 相同的查询，但处理的是 `IEnumerable<Product>` 类型。

1.  添加 `QueryDbAsIEnumerator()` 方法：

    [PRE22]

`QueryDbAsIEnumerator()` 与前述方法执行相同，但操作的是 `IEnumerator<Product>` 类型，并使用 `while` 循环而不是 `foreach` 循环进行迭代。

1.  我们需要添加的这个类中的最后一个方法是 `QueryDbAsIQueryable()` 方法：

    [PRE23]

这种方法与 `QueryDb` 相同，但明确地操作 `IQueryable<Product>` 类型。

1.  将 `Program` 类中的 `Main` 方法中的代码替换为以下内容：

    [PRE24]

此代码运行您的基准测试。进行代码的发布构建，并从命令行运行可执行文件。您应该看到类似于以下摘要报告：

![图 7.4：使用 LINQ 的各种数据库查询类型的不同时间和内存分配]

![图片](img/B16617_07_04.jpg)

图 7.4：使用 LINQ 的各种数据库查询类型的不同时间和内存分配

让我们总结一下，在运行查询基准测试后我们从摘要报告中学习到的东西：

+   在内存使用方面，性能最差的是 `QueryDb()` 方法，其次是 `QueryDbAsList()` 方法。`QueryDbAsIEnumerable()` 和 `QueryDbAsIQueryable()` 都比前两种方法略好。但所有五种方法中，内存分配性能最好的方法是 `QueryDbAsIEnumerator()` 方法。

+   在速度方面，`QueryDb()` 方法再次表现最差。其次是 `QueryDbAsIEnumerable()`，然后是 `QueryDbAsList()`，然后是 `QueryDbAsIQueryable()`。再次，速度方面的最佳表现者是 `QueryDbAsIEnumerator()` 方法。

因此，我们可以看到，在速度和内存使用方面，查询和迭代数据库的最佳方法是我们在选择调查的所有方法中性能最好的 `QueryDbAsIEnumerator()` 方法。

在下一节中，我们将研究获取集合中最后一个项的最快方法。

# 获取集合的最后一个值

现在您将看到，与直接通过索引访问项目相比，LINQ方法获取集合中的最后一个元素实际上非常慢。这将通过基准测试来测量不同方法的性能来完成：

1.  更新`Main`方法如下：

    [PRE25]

1.  打开`LinqPerformance`类。

1.  添加`GetLastPersonVersion1()`方法：

    [PRE26]

此方法使用LINQ提供的`Last()`方法获取集合中的最后一个人员。

1.  添加`GetLastPersonVersion2()`方法：

    [PRE27]

1.  在这里，我们使用列表的索引来提取列表中的最后一个人员。此时，值得注意的是，两种方法之间的区别在于，在第一种方法中，这个`Last()`方法调用实际上是在`System.Linq.Enumerable`中声明的。方法签名如下：

    [PRE28]

因此，`GetLastPersonVersion1()`方法在返回最后一个值之前执行了各种检查。但`GetLastPersonVersion2()`方法不执行这些检查，并立即返回最后一个位置上的值。这解释了为什么在`GetLastPersonVersion1()`中使用的方法比在`GetLastPersonVersion2()`中通过索引访问元素的方法慢得多，您将在下面的屏幕截图中看到：

![图7.5：使用Last()方法和直接索引访问获取最后一个人员的示例性能](img/B16617_07_05.jpg)

![img/B16617_07_05.jpg](img/B16617_07_05.jpg)

图7.5：使用Last()方法和直接索引访问获取最后一个人员的示例性能

查看我们刚刚运行的基准测试的总结报告，很明显，使用索引进行直接访问比使用`Last()`方法调用在性能提升方面更好。

我们已经看到如何快速访问集合中的最后一个元素。现在让我们考虑为什么我们应该避免在LINQ查询中使用`let`关键字。

# 避免在LINQ查询中使用`let`关键字

如果在查询中要多次使用某个值，您可以使用`let`关键字声明一个变量并为其赋值以在您的LINQ查询中使用。乍一看，这似乎表明您正在提高性能，因为您只执行一次赋值，然后在查询中多次使用相同的变量。但实际上并非如此。在您的LINQ查询中使用`let`关键字实际上可能会降低您的LINQ查询性能。

让我们通过一些基准测试示例来分析。在`LinqPerformance`类中，执行以下操作：

1.  添加`ReadingDataWithoutUsingLet()`方法：

    [PRE29]

在这个方法中，我们使用LINQ而没有使用`let`关键字从`_people`列表中选择具有姓氏*Omega*和名字*Upsilon*的人员。

1.  现在，添加`ReadingDataUsingLet()`方法：

    [PRE30]

在这个方法中，我们也在选择具有姓氏*Omega*和名字*Upsilon*的人员从`_people`列表中。但这次，我们使用`let`关键字对两个过滤器进行操作，并在`where`子句中使用它们。

1.  构建项目并从命令行运行可执行文件。您应该看到与*图7.6*中显示的结果类似：

![Figure 7.6: BenchmarkDotNet results for reading data with and without using the let keyword](img/B16617_07_06.jpg)

![img/B16617_07_06.jpg](img/B16617_07_06.jpg)

图7.6：使用和不使用`let`关键字读取数据的BenchmarkDotNet结果

如您从这些结果中可以看到，在我们的查询中使用`let`关键字降低了性能。处理时间增加，内存分配也是如此。

注意

您会看到一些网站宣传在LINQ查询中使用`let`关键字以提高性能和可读性。但正如我们在例子中所看到的，使用`let`关键字会严重减慢查询的性能并增加内存使用。因此，作为一个经验法则，请测量您特定查询的性能，并选择最适合您查询任务的执行方法。

在本节中，我们看到了使用`let`关键字如何增加使用LINQ执行简单`select`查询所需的时间和内存。当处理大量数据时，这种性能下降可能成为一个真正的问题。在下一节中，我们将探讨几种分组数据的方法，并查看哪种方法表现最好。

# 提高LINQ查询中的Group By性能

在本节中，我们将探讨执行相同的`Group By`操作的三种不同方式。每种方式都提供不同的性能级别。您将在本节结束时看到哪种方法最适合执行快速的`Group By`查询。在本节中添加的方法将被添加到`LinqPerformance`类中。

对于我们的场景，我们想要从一个所有人员都拥有相同名字的集合中获取人员列表。为了提取这些人，我们将执行一个`Group By`操作。然后，我们将提取那些组计数大于一的所有人员，并将他们添加到人员列表中。

让我们添加使用`GroupBy`子句返回人员列表的三个方法：

1.  添加`GroupByVersion1()`方法：

    [PRE31]

如您所见，我们是根据人的姓氏进行分组的。然后我们过滤这些组，只包括那些计数大于*1*的组。然后选择这些组，并将它们作为人员列表返回。

1.  现在，添加`GroupByVersion2()`方法：

    [PRE32]

在这种方法中，我们通过按姓氏分组人群，然后过滤这些组，只包括那些计数为*2*或更多的组，来获得一个枚举器。然后我们声明一个新的人员列表。接着我们遍历枚举器，获取当前的`IGrouping<string, Person>`。然后遍历分组，并将组中的每个人添加到人员列表中。

1.  添加`GroupByVersion3()`方法：

    [PRE33]

`GroupByVersion3()`方法与`GroupByVersion2()`方法相同，行为也相同，但有一个主要区别。我们在执行`Group By`之前将人员列表转换为数组。

1.  在`LinqPerformance`类的顶部添加以下注释：

    [PRE34]

这些注释将扩展总结报告中的数据，您很快就会看到。进行项目的发布构建，然后从命令行运行项目以基准测试这三种方法。您应该会看到以下基准测试总结报告：

![图7.7：BenchmarkDotNet Group By总结报告](img/B16617_07_07.jpg)

![img/B16617_07_07.jpg](img/B16617_07_07.jpg)

图7.7：BenchmarkDotNet Group By总结报告

如我们所见，我们执行`Group By`操作的第一次尝试需要*2.204*微秒，第二次尝试需要*2.011*微秒，第三次和最后一次尝试需要*2.204*微秒。因此，我们可以看到在执行`Group By`之前将列表转换为数组可以加快速度。我们的最终版本比原始版本快*0.243*微秒，尽管涉及更多的代码！

下一个部分将带您了解五种不同的提供列表过滤方式。您将看到不同的方法如何影响LINQ查询的性能。

# 过滤列表

在本节中，我们将探讨使用LINQ过滤列表的各种方法。我们将看到各种方法的表现都不相同。在本节结束时，您将知道如何过滤列表以获得更高的性能。您将编写两个不同的基准测试，以展示使用和不使用`let`关键字时查询性能的差异。让我们开始编写我们的基准测试：

1.  添加`FilterGroupsVersion1()`方法：

    [PRE35]

我们的第一个基准测试过滤属于`_group1`和`_group2`的人。由于数组是小写的，因此`LastName`也被转换为小写。然后，过滤的人作为人的列表返回。

1.  添加`FilterGroupsVersion2()`基准测试：

    [PRE36]

这与我们的第一个基准测试做的是同样的事情。主要区别在于我们使用`let`关键字引入了`lastName`变量，并将其分配给人的小写`LastName`。

1.  以发布模式编译项目并从命令行运行。将生成基准测试，您应该会看到一个类似于*图7.8*的基准测试报告：

![图7.8：使用和不使用let关键字的LINQ基准测试报告](img/B16617_07_08.jpg)

![img/B16617_07_08.jpg](img/B16617_07_08.jpg)

图7.8：使用和不使用let关键字的LINQ基准测试报告

我们可以在总结报告中看到，使用`let`关键字会显著减慢速度。因此，我们现在将调查为什么`let`关键字会减慢速度。

1.  打开`CH07_LinqPerformance.dll`。

1.  展开`FilterGroupsVersion1`和`FilterGroupsVersion2`。

1.  双击`FilterGroupsVersion1`方法以查看编译器生成的中间语言。

1.  现在，使用`FilterGroupsVersion2`方法做同样的操作。当您比较两种方法的IL时，您将清楚地看到`FilterGroupsVersion2`的IL比`FilterGroupsVersion1`的IL包含更多的代码行。

这就解释了为什么使用`let`关键字的代码版本比不使用`let`关键字的原始代码版本执行速度慢。但我们在性能方面能否做得比`FilterGroupsVersion1`更好？答案是，是的，我们可以。

1.  添加`FilterGroupsVersion3`方法：

    [PRE37]

如您所见，我们创建了一个新的人员列表。然后我们遍历`_people`列表。对于每个人，我们从`_people`列表中获取他们。然后我们将他们名字的小写形式赋值给一个局部变量。使用这个变量，我们检查`_group1`或`_group2`是否包含这些名字。如果包含，则将这个人添加到`_people`列表中。一旦迭代完成，`_people`集合将被返回。

1.  再次构建和运行代码。您应该看到以下报告：

![图7.9：BenchmarkDotNet性能摘要报告显示FilterGroupsVersion3的性能

![图7.10：BenchmarkDotNet性能摘要报告显示FilterGroupsVersion4的性能

![图7.9：BenchmarkDotNet性能摘要报告显示FilterGroupsVersion3的性能

如您所见，我们有三种不同的代码版本产生相同的输出，但每个版本的执行时间不同。在这三种不同的方法中，`FilterGroupsVersion3`是达到预期结果最快的方法。

1.  我们将再次尝试改进LINQ过滤器查询的性能。添加`FilterGroupsVersion4`方法：

    [PRE38]

可以看出，`FilterGroupsVersion3`和`FilterGroupsVersion4`之间的唯一区别是`if`条件检查的顺序。

1.  构建项目并运行基准测试。*图7.10*显示了性能摘要：

![图7.10：BenchmarkDotNet性能摘要报告显示FilterGroupsVersion4的性能

![图7.10：BenchmarkDotNet性能摘要报告显示FilterGroupsVersion4的性能

![图7.10：BenchmarkDotNet性能摘要报告显示FilterGroupsVersion4的性能

从基准报告中可以看出，我们过滤器的第4版在性能方面是获胜的方法。那么，为什么第4版比第3版更好？`_group2`数组包含的项目比`_group1`少。如果您理解业务领域，您将能够以这种方式排序过滤检查，即首先检查项目较少的数组。

您已经看到使用`let`关键字会减慢速度。但您也看到了条件语句中检查的顺序如何影响性能。在条件检查语句中将具有最少元素的检查放在第一位将提高性能。

在下一节中，我们将探讨LINQ语句中的闭包以及它们如何影响查询性能。

# 理解闭包

在本节中，我们将从C#的角度理解闭包，并将其应用于LINQ查询。让我们从维基百科上关于计算机编程闭包的定义开始。

维基百科：“在编程语言中，闭包，也称为词法闭包或函数闭包，是一种在具有一等函数的语言中实现词法作用域名称绑定技术。”

在操作上，闭包是一个记录，它存储了一个函数及其环境。

环境是一个映射，它将函数的每个自由变量（在局部使用但在封装作用域中定义的变量）与在创建闭包时名称所绑定到的值或引用关联起来。

与普通函数不同，闭包允许函数通过闭包对其值的副本或引用的捕获变量进行访问，即使函数在其作用域之外被调用。

为了理解这里所说的内容，我们将首先理解一等函数是什么。

一等函数是 C# 将其视为一等数据类型的函数。这意味着你可以将方法分配给变量并传递它，你可以像调用普通方法一样调用它。一等函数可以使用匿名方法和 lambda 表达式创建。

自由变量是那些不是方法参数变量的变量，它们也不是那个方法局部变量，换句话说，它们是存在于方法之外的变量，但在方法的作用域内被引用。

我们将把闭包应用到 LINQ 表达式并对其进行基准测试。第一个将使用带参数的闭包的 LINQ，第二个将使用使用自由变量的闭包的 LINQ。按照以下步骤进行：

1.  在 `LinqPerformance` 类中，注释掉当前 `[Benchmark]` 注解的方法。

1.  添加 `LinqClosureUsingParameters` 方法：

    [PRE39]

在 `LinqClosureUsingParameters` 方法中，我们使用带有参数的委托声明闭包。我们声明一个名为 `IsBetween` 的变量并将 `Between` 方法分配给它。然后我们执行 LINQ 查询并通过调用 `IsBetween` 来过滤结果。结果是，我们只会得到那些姓氏首字母在 A 和 G 之间的人。

1.  我们也可以使用自由变量。因此，现在让我们看看一个使用自由变量的不同示例。添加 `LinqClosureUsingVariables` 方法：

    [PRE40]

在 `LinqClosureUsingVariables` 方法中，我们使用自由变量来声明用于过滤数据集的第一个和最后一个字符。然后，我们将 `Between` 方法分配给 `IsBetweenAG` 变量。然后，我们执行 LINQ 查询并通过将每个个人的姓氏传递给 `IsBetweenAG` 方法来过滤结果。

1.  添加一个名为 `NonLinqFilter` 的方法：

    [PRE41]

在这个方法中，我们只是使用自己的 `FindAll` 方法过滤列表。

1.  确保你处于发布模式，然后运行你的项目。你应该得到以下截图中的类似结果：

![Figure 7.11: Closure benchmarks with and without parameters](img/B16617_07_11.jpg)

![img/B16617_07_11.jpg](img/B16617_07_11.jpg)

图 7.11：带参数和不带参数的闭包基准测试

如*图7.11*的基准测试所示，我们可以清楚地看到，带有参数的闭包比不带参数的闭包更快，并且分配的内存更少。但使用列表自己的`FindAll`方法进行过滤更好，因为它比LINQ和闭包更快，并且使用的分配内存更少。

当你需要在自己的LINQ查询中使用自定义闭包时，可能的情况是，你有复杂的数据操作和查询生成，这些操作无法用正常的LINQ轻松处理。在这种情况下，闭包将对你有所帮助。在进行了闭包的基准测试后，你现在知道在使用LINQ时，为了获得最佳性能，应该使用带有参数的闭包。但如果你不需要使用LINQ，那么使用列表自己的方法可能更有利。而且如果你确实需要在列表上工作，那么首先使用非LINQ方法过滤数据集可能是有益的，然后在对过滤后的列表执行LINQ查询。

本章现在已完成。但在我们继续进入[*第8章*](B16617_08_Final_SB_Epub.xhtml#_idTextAnchor152)，*文件和流I/O*之前，让我们总结一下本章学到的内容。

# 摘要

在本章中，我们通过基准测试了查询、分组、过滤和迭代从数据库和内存集合中获取数据的各种方法，研究了LINQ的性能。发现查询数据库的最有效方法是使用`IEnumerator`接口。通过反汇编代码，我们看到`let`关键字可能会由于编译器产生的额外IL代码行而降低性能。我们还看到，使用索引访问集合中的最后一个元素比调用`Last()`方法更快。我们还了解到，首先过滤具有最少项的对象来过滤列表可以提高过滤操作的性能。与不传递参数相比，传递参数的闭包提供了更好的整体性能。

在下一章中，我们将探讨文件和流I/O性能。但到目前为止，看看你是否能回答以下问题，并查看进一步阅读材料，以巩固本章学到的内容。

# 问题

1.  提出一些提高LINQ性能的方法。

1.  在LINQ查询中使用`let`关键字有什么问题？

1.  提高分组查询性能的最佳方法是什么？

1.  带参数的闭包和不带参数的闭包哪个性能更好？

# 进一步阅读

+   **控制台用户秘密**：[https://github.com/jasonshave/ConsoleSecrets](https://github.com/jasonshave/ConsoleSecrets).

+   **优化LINQ**：[https://mattwarren.org/2016/09/29/Optimising-LINQ/](https://mattwarren.org/2016/09/29/Optimising-LINQ/)

)

+   **提高LINQ to SQL性能的五个技巧**：[https://visualstudiomagazine.com/articles/2010/06/24/five-tips-linq-to-sql.aspx](https://visualstudiomagazine.com/articles/2010/06/24/five-tips-linq-to-sql.aspx).

+   **使用LINQ连接使您的C#应用程序更快**：[https://timdeschryver.dev/blog/make-your-csharp-applications-faster-with-linq-joins](https://timdeschryver.dev/blog/make-your-csharp-applications-faster-with-linq-joins).

+   **LINQ很糟糕 – 您LINQ中的代码异味**：[https://markheath.net/post/linq-stinks](https://markheath.net/post/linq-stinks).

+   **如何使用LINQ表达式树从Span<T>中获取值？**：[https://stackoverflow.com/questions/52112628/how-to-get-a-value-out-of-a-spant-with-linq-expression-trees](https://stackoverflow.com/questions/52112628/how-to-get-a-value-out-of-a-spant-with-linq-expression-trees).

+   **C#中的Linq ToLookup方法**：[https://dotnettutorials.net/lesson/linq-tolookup-operator/](https://dotnettutorials.net/lesson/linq-tolookup-operator/).

+   **LINQ (C#) – ToLookup运算符示例和教程**：[https://www.completecsharptutorial.com/linqtutorial/tolookup-operator-example-csharp-linq-tutorial.php](https://www.completecsharptutorial.com/linqtutorial/tolookup-operator-example-csharp-linq-tutorial.php).

+   **C#闭包的简单解释**：[https://www.simplethread.com/c-closures-explained/](https://www.simplethread.com/c-closures-explained/).
