# 12

# 使用GraphQL组合数据源

在本章中，你将了解GraphQL，这是一种服务技术，它提供了一种更现代的方法来组合来自各种来源的数据，并提供了一种查询这些数据的标准方式。

本章将涵盖以下主题：

+   理解GraphQL

+   构建支持GraphQL的服务

+   定义EF Core模型的GraphQL查询

+   为GraphQL服务构建.NET客户端

+   实现GraphQL突变

+   实现GraphQL订阅

# 理解GraphQL

在*第8章*，*使用最小API构建和保障Web服务*中，你学习了如何通过将请求路径端点映射到lambda表达式或返回响应的方法来定义Web API服务。任何参数和响应格式都由服务控制。客户端无法请求他们确切需要的内容或使用更有效的数据格式。

如果你完成了仅在网络上提供的部分，*通过OData在Web上公开数据*，那么你知道OData有一个内置的查询语言，客户端可以用来控制他们想要返回的数据。然而，OData有一个相当过时的方法，并且与HTTP标准相关联，例如，在HTTP请求中使用查询字符串。

如果你希望使用一种更现代且灵活的技术来组合并公开你的数据作为服务，那么一个不错的选择是**GraphQL**。

与OData一样，GraphQL是一个描述你的数据并查询它的标准，它给了客户端控制权，让他们确切知道他们需要什么。它于2012年由Facebook内部开发，并于2015年开源，现在由GraphQL基金会管理。

与OData相比，GraphQL的一些关键优势包括：

+   GraphQL不需要HTTP，因为它与传输无关，因此你可以使用替代的传输协议，如WebSockets或TCP。

+   GraphQL比OData拥有更多针对不同平台的客户端库。

+   GraphQL有一个单一端点，通常简单地是`/graphql`。

## GraphQL查询文档格式

GraphQL使用自己的文档格式进行查询，这与JSON有点相似，但GraphQL查询不需要在字段名之间使用逗号，如下面的查询所示，该查询请求了ID为`23`的产品的一些字段和相关数据：

[PRE0]

GraphQL查询文档的官方媒体类型是`application/graphql`。

### 请求字段

最基本的GraphQL查询从一个类型请求一个或多个字段，例如，为每个`customer`实体请求三个字段，如下面的代码所示：

[PRE1]

响应以JSON格式，例如，一个`customer`对象的数组，如下面的文档所示：

[PRE2]

### 指定过滤器和参数

使用HTTP或REST风格的API时，调用者仅限于在API预定义的情况下传递参数。在GraphQL中，你可以在查询的任何位置设置参数，例如，通过订单日期和下订单的客户的国籍来过滤订单，如下面的代码所示：

[PRE3]

注意，尽管GraphQL使用camelCase作为实体、属性和参数名称，但您应该使用PascalCase作为查询名称。

您可能希望传递命名参数的值而不是将它们硬编码，如下面的代码所示：

[PRE4]

您可以在以下链接了解更多关于GraphQL查询语言的信息：[https://graphql.org/learn/queries/](https://graphql.org/learn/queries/)。

## 理解其他GraphQL能力

除了查询之外，其他标准的GraphQL特性还包括突变和订阅：

+   **突变**使您能够创建、更新和删除资源。

+   **订阅**允许客户端在资源发生变化时收到通知。它们与WebSocket等附加通信技术配合使用效果最佳。

## 理解ChilliCream GraphQL平台

GraphQL.NET是实施GraphQL的.NET中最受欢迎的平台之一。在我看来，即使是基本示例，GraphQL.NET也需要太多的配置，而且文档令人沮丧。我有一种感觉，尽管它功能强大，但它以一对一的方式实现了GraphQL规范，而不是重新思考.NET平台如何使其更容易上手。

您可以在以下链接了解GraphQL.NET：[https://graphql-dotnet.github.io/](https://graphql-dotnet.github.io/)。

对于这本书，我寻找了替代方案，并找到了我想要的。ChilliCream是一家创建了一个整个平台来与GraphQL一起工作的公司：

+   **热巧克力**使您能够为.NET创建GraphQL服务。

+   **草莓奶昔**使您能够为.NET创建GraphQL客户端。

+   **香蕉蛋糕棒**使您能够使用基于Monaco的GraphQL IDE运行查询并探索GraphQL端点。

+   **绿色甜甜圈**在加载数据时提供更好的性能。

与一些其他可以用来添加GraphQL支持的包不同，ChilliCream包旨在尽可能容易实现，使用约定和简单的POCO类而不是复杂类型和特殊模式。它的工作方式类似于微软可能为.NET构建的GraphQL平台，有合理的默认值和约定，而不是大量的样板代码和配置。

正如ChilliCream在其主页上所说：“我们在ChilliCream构建终极GraphQL平台。我们的大部分代码是开源的，并将永远保持开源。”

Hot Chocolate的GitHub仓库链接如下：[https://github.com/ChilliCream/hotchocolate](https://github.com/ChilliCream/hotchocolate)。

# 构建支持GraphQL的服务

由于GraphQL不需要托管在Web服务器上，因为它不依赖于HTTP，因此选择**ASP.NET Core Empty**项目模板作为起点是合理的。然后我们将添加一个用于GraphQL支持的包引用：

1.  使用您首选的代码编辑器添加一个新项目，如下列所示：

    +   项目模板：**ASP.NET Core Empty** / `web`

    +   解决方案文件和文件夹：`Chapter12`

    +   项目文件和文件夹：`Northwind.GraphQL.Service`

    +   其他 Visual Studio 2022 选项：

        +   **配置 HTTPS**：已选择

        +   **启用 Docker**：已清除

        +   **不要使用顶级语句**：已清除

1.  在项目文件中，添加一个对托管在 ASP.NET Core 中的 Hot Chocolate 的包引用，如下面的标记所示：

    [PRE5]

    写作时，版本 13.6.0 正在预览中。要使用最新的预览版本，您可以将其设置为 `13.6-*`。但我建议您访问以下链接，然后引用最新的 GA 版本：[https://www.nuget.org/packages/HotChocolate.AspNetCore/](https://www.nuget.org/packages/HotChocolate.AspNetCore/)。

1.  在项目文件中，将警告视为错误并禁用警告 `AD0001`，如下面的标记所示：

    [PRE6]

1.  在 `Properties` 文件夹中，在 `launchSettings.json` 中，对于 `https` 配置文件，将 `applicationUrl` 修改为使用 `https` 的端口 `5121` 和 `http` 的端口 `5122`。然后，添加一个 `launchUrl` 为 `graphql`，如下面的配置所示：

    [PRE7]

1.  构建 `Northwind.GraphQL.Service` 项目。

## 定义 Hello World 的 GraphQL 模式

第一项任务是定义我们希望在 Web 服务中公开的 GraphQL 模型。

让我们定义一个 GraphQL 查询，用于最基本的 `Hello World` 示例，当请求问候时将返回纯文本：

1.  在 `Northwind.GraphQL.Service` 项目/文件夹中，添加一个名为 `Query.cs` 的类文件。

1.  修改类以包含一个名为 `GetGreeting` 的方法，该方法返回纯文本 `"Hello, World!"`，如下面的代码所示：

    [PRE8]

1.  在 `Program.cs` 中，导入我们定义 `Query` 类的命名空间，如下面的代码所示：

    [PRE9]

1.  在配置服务的部分，在调用 `CreateBuilder` 之后，添加一个语句以添加 GraphQL 服务器端支持，并将查询类型添加到已注册的服务集合中，如下面的代码所示：

    [PRE10]

1.  修改将 `GET` 请求映射为返回更有用的纯文本消息的语句，如下面的代码所示：

    [PRE11]

1.  在配置 HTTP 管道的部分，在调用 `Run` 之前，添加一个语句将 GraphQL 映射为一个端点，如下面的代码所示：

    [PRE12]

1.  启动 `Northwind.GraphQL.Service` 网络项目，使用 `https` 配置文件且不进行调试：

    +   如果你正在使用 Visual Studio 2022，请选择 **https** 配置文件，不进行调试启动项目，并注意浏览器会自动启动。

    +   如果你正在使用 Visual Studio Code，请输入命令 `dotnet run --launch-profile https`，手动启动 Chrome，并导航到 `https://localhost:5121/graphql`。

1.  注意 **BananaCakePop** 用户界面，然后点击 **创建文档** 按钮。

1.  在右上角，点击 **连接设置** 按钮，如图 *12.1* 所示：

![](img/B19587_12_01.png)

图 12.1：一个新的 BananaCakePop 文档和连接设置按钮

1.  在 **连接设置** 中，确认 **模式端点** 是正确的，然后点击 **取消**，如图 *图 12.2* 所示：

![图片](img/B19587_12_02.png)

图 12.2：审查 BananaCakePop 连接设置

1.  在 **untitled 1** 文档的顶部，点击 **模式引用** 选项卡。

1.  在 **模式引用** 选项卡中，注意“`Query` 类型是一个特殊类型，它定义了每个 GraphQL 查询的入口点”，它有一个名为 `greeting` 的字段，该字段返回一个 `String!` 值。感叹号表示该值将 *不会* 为 `null`。

1.  点击 **模式定义** 选项卡，并注意只有一个类型被定义，即特殊的 `Query` 对象，它有一个 `greeting` 字段，该字段是一个非空 `String` 值，如下面的代码所示：

    [PRE13]

## 编写和执行 GraphQL 查询

现在我们知道了模式，我们可以编写并运行一个查询：

1.  在 **Banana Cake Pop** 的 **untitled 1** 文档中，点击 **操作** 选项卡。

1.  在左侧，输入一个开括号 `{`，并注意为你自动写入了闭括号 `}`。

1.  输入字母 `g`，并注意自动完成显示它识别了 `greeting` 字段，如图 *图 12.3* 所示：

![图片](img/B19587_12_03.png)

图 12.3：`greeting` 字段的自动完成

1.  按 *Enter* 键接受自动完成建议。

1.  点击 **运行** 按钮并注意响应，如图下面的输出所示：

    [PRE14]

1.  关闭 Chrome 浏览器，并关闭 web 服务器。

## 命名 GraphQL 查询即操作

我们编写的查询没有命名。我们也可以将其创建为命名查询，如下面的代码所示：

[PRE15]

命名查询允许客户端识别用于遥测目的的查询和响应，例如，当在 Microsoft Azure 云服务中托管并使用 Application Insights 进行监控时。

## 理解字段约定

我们在 `Query` 类中创建的 C# 方法名为 `GetGreeting`，但在查询它时，我们使用了 `greeting`。表示 GraphQL 中字段的命名方法名上的 `Get` 前缀是可选的。让我们看看更多示例：

1.  在 `Query.cs` 中添加两个不带 `Get` 前缀的方法，如下面的代码所示：

    [PRE16]

1.  使用 `https` 配置文件启动 `Northwind.GraphQL.Service` 项目，不进行调试。

1.  点击 **模式定义** 选项卡，并注意更新的模式，如下面的代码所示：

    [PRE17]

    C# 方法使用 PascalCase。GraphQL 字段使用 camelCase。

1.  点击 **操作** 选项卡，并将查询修改为指定名称并请求 `rollTheDie` 字段，如下面的代码所示：

    [PRE18]

1.  多次点击 **运行** 按钮。注意，响应包含介于 1 和 6 之间的随机数字，以及当前浏览器会话中请求和响应的历史记录，如图 *图 12.4* 所示：

![图片](img/B19587_12_04.png)

图 12.4：执行命名查询和请求/响应历史记录

1.  关闭 Chrome 浏览器，并关闭 web 服务器。

# 定义 EF Core 模型的 GraphQL 查询

现在我们已经成功运行了一个基本的 GraphQL 服务，让我们扩展它以支持查询 Northwind 数据库。

## 添加对 EF Core 的支持

我们必须添加另一个 Hot Chocolate 包，以便允许轻松地将我们的 EF Core 数据库上下文与 GraphQL 查询类集成依赖项服务：

1.  添加对 Hot Chocolate 与 EF Core 集成的包引用，以及 Northwind 数据库上下文项目的项目引用，如下面的标记所示：

    [PRE19]

    项目的路径不能有换行符。所有 Hot Chocolate 包应具有相同的版本号。

1.  在命令提示符或终端中使用 `dotnet build` 命令构建 `Northwind.GraphQLService` 项目。

    当你引用当前解决方案之外的项目时，你必须在命令提示符或终端中至少构建一次项目，然后才能使用 Visual Studio 2022 **构建**菜单来编译它。

1.  在 `Program.cs` 中，导入命名空间以使用 Northwind 数据库的 EF Core 模型，如下面的代码所示：

    [PRE20]

1.  在 `CreateBuilder` 方法之后添加一个语句以注册 `Northwind` 数据库上下文类，并在添加 GraphQL 服务器支持之后添加一个语句以注册 `NorthwindContent` 类进行依赖注入，如下面的代码所示：

    [PRE21]

1.  在 `Query.cs` 中添加语句以定义一个对象图类型，该类型具有一些查询类型，可以返回类别列表、单个类别、类别的产品、具有最低单位价格的产品和所有产品，如下面的代码所示：

    [PRE22]

## 探索使用 Northwind 的 GraphQL 查询

现在我们可以测试为 Northwind 数据库中的类别和产品编写 GraphQL 查询：

1.  如果你的数据库服务器没有运行，例如，因为你正在 Docker、虚拟机或云端托管它，那么请确保启动它。

1.  使用 `https` 配置文件（无调试）启动 `Northwind.GraphQL.Service` 项目。

1.  在 **Banana Cake Pop** 中，点击 **+** 打开一个新标签页。

1.  点击 **Schema Definition** 选项卡，并注意 `Category` 的查询和类型定义，部分内容如图 12.5 所示：

![](img/B19587_12_05.png)

图 12.5：使用 GraphQL 查询 Northwind 类别和产品的模式

1.  注意以下代码中的完整定义：

    [PRE23]

1.  点击 **Operations** 选项卡，编写一个命名查询以请求所有类别的 ID、名称和描述字段，如下面的标记所示：

    [PRE24]

1.  点击 **Run**，并注意响应，如图 12.6 和以下部分输出所示：

    [PRE25]

![](img/B19587_12_06.png)

图 12.6：获取所有类别

1.  点击 **+** 打开一个名为 **untitled 2** 的新文档标签页，并编写一个查询以请求 ID 为 `2` 的类别，包括其产品的 ID、名称和价格，如下面的标记所示：

    [PRE26]

    确保 `categoryId` 中的 `I` 是大写。

1.  点击 **Run**，并注意响应，如下面的部分输出所示：

    [PRE27]

1.  在GraphQL web服务命令提示符或终端中，注意为此查询执行的SQL语句，如下所示的部分输出：

    [PRE28]

    虽然GraphQL查询不需要每个类别的图片，只需要ID、名称和单价，但EF Core动态生成的查询返回了所有属性。

1.  点击**+**标签打开一个新标签页，并编写一个查询以请求ID、名称和库存单位的产品，其类别ID为`1`，如下所示：

    [PRE29]

1.  点击**运行**，并注意响应，如下所示的部分输出：

    [PRE30]

1.  点击**+**标签打开一个新标签页，并编写一个查询以请求产品的ID、名称、库存单位和其类别名称，如下所示：

    [PRE31]

1.  点击**运行**，并注意响应，如下所示的部分输出：

    [PRE32]

1.  点击**+**标签打开一个新标签页，并编写一个查询以请求类别的ID和名称，通过指定类别ID选择该类别，并包括其每个产品的ID和名称。类别ID将使用变量设置，如下所示：

    [PRE33]

1.  在**变量**部分，定义一个变量的值，如下所示，并在*图12.7*中：

    [PRE34]

![图片](img/B19587_12_07.png)

图12.7：执行带有变量的GraphQL查询

1.  点击**运行**，并注意响应，如下所示的部分输出：

    [PRE35]

1.  点击**+**标签打开一个新标签页，编写一个查询以请求类别的ID和名称，通过指定类别ID选择该类别，并包括其每个产品的ID和名称。类别ID将使用变量设置，如下所示：

    [PRE36]

1.  在**变量**部分，定义一个变量的值，如下所示：

    [PRE37]

1.  点击**运行**，并注意响应，如下所示的部分输出：

    [PRE38]

1.  关闭Chrome，并关闭Web服务器。

## 实现分页支持

当我们使用`GetProducts`方法（`products`查询）请求产品时，返回所有77个产品。让我们添加分页支持：

1.  在`Query.cs`中添加语句以定义一个查询，用于返回所有产品，使用分页，并注意其实现与不带分页的产品查询相同，但它被装饰了`[UsePaging]`属性，如下所示：

    [PRE39]

    **良好实践**：`[UsePaging]`、`[UseFiltering]`和`[UseSorting]`属性必须装饰到返回`IQueryable<T>`的查询方法上，允许GraphQL在执行数据存储之前动态配置LINQ查询。

1.  使用`https`配置文件（无调试）启动`Northwind.GraphQL.Service`项目。

1.  在**香蕉蛋糕棒**中，点击**+**以打开一个新标签页。

1.  点击**模式定义**标签，并注意名为`products`（不带分页）和`productsWithPaging`的查询，这些查询说明了如何使用查询请求产品的一页，如下所示的部分输出：

    [PRE40]

1.  点击 **操作**选项卡，并编写一个命名查询以请求第 1 页的 10 个商品，如下所示的部分标记：

    [PRE41]

1.  点击 **运行**，注意响应，包括 `pageInfo` 部分，它告诉我们还有另一页的产品，并且此页的游标范围从 `MA==` 到 `OQ==`，如下所示的部分输出：

    [PRE42]

1.  点击 **+** 打开一个新标签页，并编写一个命名查询以请求第 2 页的 10 个商品，指定我们想要一个从 `OQ==` 开始的游标，如下所示的部分标记：

    [PRE43]

1.  点击 **运行**，注意响应，包括 `pageInfo` 部分，它告诉我们还有另一页的产品，并且此页的游标范围从 `MTA=` 到 `MTk=`，如下所示的部分输出：

    [PRE44]

1.  关闭 Chrome，并关闭 web 服务器。

## 实现过滤支持

当我们在本章前面探索查询时，我们预先定义了一些带有参数的查询，例如，通过传递 `categoryId` 参数返回一个类别中所有商品的查询。

然而，如果你事先不知道要执行什么过滤操作怎么办？

让我们向我们的 GraphQL 查询添加过滤支持：

1.  在 `Program.cs` 文件中，在调用 `AddGraphQLServer` 之后添加对 `AddFiltering` 的调用，如下代码所示：

    [PRE45]

1.  在 `Query.cs` 文件中，使用 `[UseFiltering]` 属性装饰 `GetProducts` 方法，如下代码所示：

    [PRE46]

1.  使用不带调试的 `https` 配置启动 `Northwind.GraphQL.Service` 项目。

1.  在 **香蕉蛋糕棒** 中，点击 **+** 打开一个新标签页。

1.  点击 **方案定义**，注意 `products` 查询现在接受一个过滤输入，如下所示的部分标记：

    [PRE47]

1.  将方案定义向下滚动以找到 `ProductFilterInput`，并注意过滤选项包括布尔运算符如 `and` 和 `or`，以及字段过滤器如 `IntOperationFilterInput` 和 `StringOperationFilterInput`，如下所示的部分标记：

    [PRE48]

1.  将方案定义向下滚动以找到 `IntOperationFilterInput` 和 `StringOperationFilterInput`，并注意你可以与它们一起使用的操作，如等于 (`eq`)、不等于 (`neq`)、在数组中 (`in`)、大于 (`gt`)、包含 (`contains`) 和以...开头 (`startsWith`)，如下所示的部分标记：

    [PRE49]

1.  点击 **操作**，然后编写一个命名查询以请求库存超过 `120` 单位的商品，如下所示的部分标记：

    [PRE50]

1.  点击 **运行**，注意响应，如下所示的部分输出：

    [PRE51]

1.  点击 **+** 打开一个新标签页，点击 **操作**，然后编写一个命名查询以请求名称以 `Cha` 开头的商品，如下所示的部分标记：

    [PRE52]

1.  点击 **运行**，注意响应，如下所示的部分输出：

    [PRE53]

1.  在 GraphQL 服务命令提示符或终端中，注意 EF Core 生成的使用参数的 SQL 过滤器，如下所示的部分输出：

    [PRE54]

1.  关闭 Chrome，并关闭 web 服务器。

## 实现排序支持

要启用 GraphQL 服务的排序，调用 `AddSorting` 方法，如下面的代码所示：

[PRE55]

然后，使用 `[UseSorting]` 属性装饰返回 `IQueryable<T>` 的查询方法，如下面的代码所示：

[PRE56]

在查询中应用一个或多个排序顺序，如下面的代码所示：

[PRE57]

`SortEnumType` 有两个值，如下面的代码所示：

[PRE58]

我将把添加排序功能到你的 GraphQL 服务中留给你。

# 为 GraphQL 服务构建 .NET 客户端

现在我们已经使用 **Banana Cake Pop** 工具探索了一些查询，让我们看看客户端如何调用 GraphQL 服务。虽然 **Banana Cake Pop** 工具很方便，但它运行在与服务相同的域中，因此一些问题可能直到我们创建一个单独的客户端时才变得明显。

## 选择 GraphQL 请求格式

大多数 GraphQL 服务以 `application/graphql` 或 `application/json` 媒体格式处理 `GET` 和 `POST` 请求。一个 `application/graphql` 请求将只包含一个查询文档。使用 `application/json` 的好处是，除了查询文档外，你还可以在有多种操作时指定操作，并定义和设置变量，如下面的代码所示：

[PRE59]

我们将使用 `application/json` 媒体格式，这样我们就可以传递变量及其值。

## 理解 GraphQL 响应格式

一个 GraphQL 服务应该返回一个包含预期数据对象和可能包含一些错误数组的 JSON 文档，其结构如下：

[PRE60]

`errors` 数组只有在文档中有错误时才应出现在文档中。

## 使用 REST Client 作为 GraphQL 客户端

在我们编写客户端代码向 GraphQL 服务发送请求之前，最好使用你的代码编辑器的 `.http` 文件支持对其进行测试。这样，如果我们的 .NET 客户端应用不起作用，我们就知道问题出在我们的客户端代码而不是服务上：

1.  如果你正在使用 Visual Studio Code 并且尚未安装 Huachao Mao 的 REST Client (`humao.rest-client`)，那么现在就安装它。

1.  在你偏好的代码编辑器中，启动 `Northwind.GraphQL.Service` 项目网络服务，使用 `https` 配置文件，不进行调试，并保持运行。

1.  在你的代码编辑器中，在 `HttpRequests` 文件夹中，创建一个名为 `graphql-queries.http` 的文件，并修改其内容以包含获取海鲜类别产品的请求，如下面的代码所示：

    [PRE61]

1.  发送查询请求，并注意响应，如图 *图 12.8* 所示：

![](img/B19587_12_08.png)

图 12.8：使用 REST Client 请求海鲜产品

1.  添加一个查询来获取所有类别的 ID、名称和描述，如下面的代码所示：

    [PRE62]

1.  发送查询请求，并注意响应包含 `data` 属性中的八个类别。

1.  在查询文档中，将 `categoryId` 改为 `id`。

1.  发送查询请求，并注意响应包含一个 `errors` 数组，如下面的响应所示：

    [PRE63]

1.  在查询文档中，将 `id` 重新改为 `categoryId`。

1.  添加一个查询以请求获取通过参数指定的类别ID及其名称，以及每个产品的ID和名称，如下面的代码所示：

    [PRE64]

1.  发送查询请求，并注意响应包含类别`1`，`Beverages`，以及其产品在`data`属性中。

1.  将ID更改为`4`，发送请求，并注意响应包含类别`4`，`Dairy Products`，以及其产品在`data`属性中。

现在我们已经对服务及其对我们想要运行的查询的响应进行了一些基本的测试，我们可以构建一个客户端来执行这些查询并处理JSON响应。

## 使用ASP.NET Core MVC项目作为GraphQL客户端

我们将创建一个模型类，以便轻松反序列化响应：

1.  使用您首选的代码编辑器添加一个新项目，如下列列表中定义的：

    1.  项目模板：**ASP.NET Core Web App (Model-View-Controller**) / `mvc`

    1.  解决方案文件和文件夹：`Chapter12`

    1.  项目文件和文件夹：`Northwind.GraphQL.Client.Mvc`

    1.  其他Visual Studio 2022选项：

        +   **身份验证类型**：无

        +   **配置为HTTPS**：已选择

        +   **启用Docker**：已清除

        +   **不要使用顶级语句**：已清除

1.  在Visual Studio 2022中，设置启动项目为当前选择。

1.  在`Northwind.GraphQL.Client.Mvc`项目中，添加对Northwind实体模型项目的项目引用，如下面的标记所示：

    [PRE65]

    项目的路径不得包含换行符。

1.  在命令提示符或终端中构建`Northwind.GraphQL.Client.Mvc`项目。

1.  在`Properties`文件夹中的`launchSettings.json`中，修改`applicationUrl`以使用端口`5123`进行`https`和端口`5124`进行`http`，如下面的配置中突出显示：

    [PRE66]

1.  在`Northwind.GraphQL.Client.Mvc`项目的`Models`文件夹中，添加一个名为`ResponseErrors.cs`的新类文件，如下面的代码所示：

    [PRE67]

1.  在`Models`文件夹中，添加一个名为`ResponseProducts.cs`的新类文件，如下面的代码所示：

    [PRE68]

1.  在`Models`文件夹中，添加一个名为`ResponseCategories.cs`的新类文件，如下面的代码所示：

    [PRE69]

1.  在`Models`文件夹中，添加一个名为`IndexViewModel.cs`的新类文件，它将具有存储我们可能在视图中显示的所有数据的属性，如下面的代码所示：

    [PRE70]

1.  在`Program.cs`中，导入命名空间以设置HTTP头，如下面的代码所示：

    [PRE71]

1.  在`Program.cs`中，在`CreateBuilder`方法调用之后，添加注册GraphQL服务的HTTP客户端的语句，如下面的代码所示：

    [PRE72]

1.  在`Controllers`文件夹中的`HomeController.cs`中，导入命名空间以处理文本编码以及本地项目模型，如下面的代码所示：

    [PRE73]

1.  定义一个字段以存储已注册的HTTP客户端工厂，并在构造函数中设置它，如下面的代码所示：

    [PRE74]

1.  在 `Index` 动作方法中，将方法修改为异步。然后，添加调用 GraphQL 服务的语句，并注意 HTTP 请求是 `POST`，媒体类型是包含 GraphQL 查询的 `application/json` 文档，该查询请求给定类别中所有产品的 ID、名称和库存数量，通过名为 `id` 的参数传递，如下面的代码所示：

    [PRE75]

    **良好实践**：为了设置我们请求的内容，我们将使用 C# 11 或更高版本的三个美元符号和三个双引号的原始插值字符串字面量语法。这允许我们使用三个大括号嵌入 `id` 变量，不应与 `unitsInStock` 后面的两个大括号混淆，后者结束查询本身。

1.  在 `Views/Home` 文件夹中的 `Index.cshtml` 文件中，删除其现有的标记，然后添加标记以渲染海鲜产品，如下面的标记所示：

    [PRE76]

## 测试 .NET 客户端

现在，我们可以测试我们的 .NET 客户端：

1.  如果您的数据库服务器没有运行，例如，因为您正在 Docker、虚拟机或云中托管它，那么请确保启动它。

1.  使用不带调试的 `https` 配置启动 `Northwind.GraphQL.Service` 项目。

1.  使用不带调试的 `https` 配置启动 `Northwind.GraphQL.Client.Mvc` 项目。

1.  注意，使用 GraphQL 成功检索了产品，如图 *图 12.9* 所示：

![图片](img/B19587_12_09.png)

图 12.9：来自 GraphQL 服务的饮料类别产品

1.  输入另一个存在的类别 ID，例如 `4`。

1.  输入一个超出范围的类别 ID，例如 `13`，并注意返回了 0 个产品。

1.  关闭 Chrome，并关闭 `Northwind.GraphQL.Client.Mvc` 项目的 Web 服务器。

1.  在 `HomeController.cs` 中，修改查询以故意犯一个错误，例如将 `productId` 改为 `productid`。

1.  使用不带调试的 `https` 配置启动 `Northwind.GraphQL.Client.Mvc` 项目。

1.  点击 **显示/隐藏详细信息** 按钮，并注意错误信息和响应详细信息，如下面的输出所示：

    [PRE77]

1.  关闭 Chrome，并关闭两个 Web 服务器。

1.  修复查询中的错误！

## 使用 Strawberry Shake 创建控制台应用程序客户端

与使用普通 HTTP 客户端不同，ChilliCream 有一个 GraphQL 客户端库，可以更轻松地构建用于 GraphQL 服务的 .NET 客户端。

**更多信息**：您可以在以下链接中了解更多关于 Strawberry Shake 的信息：[https://chillicream.com/docs/strawberryshake](https://chillicream.com/docs/strawberryshake)

现在，让我们使用 Strawberry Shake 创建另一个客户端，以便您可以看到其好处：

1.  使用您首选的代码编辑器添加一个新的 **控制台应用程序** / `console` 项目，命名为 `Northwind.GraphQL.Client.Console`。

1.  在项目文件夹的命令提示符或终端中，创建一个工具清单文件，如下面的命令所示：

    [PRE78]

1.  在命令行或终端中，安装 Strawberry Shake 工具，如下面的命令所示：

    [PRE79]

1.  注意Strawberry Shake已安装，如下所示：

    [PRE80]

1.  在项目中，将警告视为错误，添加对Microsoft扩展依赖注入、处理HTTP和Strawberry Shake代码生成的NuGet包的引用，然后全局和静态导入`Console`类，如下所示：

    [PRE81]

    你需要为不同类型的.NET项目使用不同的Strawberry Shake包。对于控制台应用程序和ASP.NET Core应用程序，引用`StrawberryShake.Server`。对于Blazor WebAssembly应用程序，引用`StrawberryShake.Blazor`。对于.NET MAUI应用程序，引用`StrawberryShake.Maui`。

1.  构建项目`Northwind.GraphQL.Client.Console`以还原包。

1.  启动`Northwind.GraphQL.Service`项目，使用`https`配置文件且不进行调试，并让它运行，以便Strawberry Shake工具可以与其通信。

1.  在`Northwind.GraphQL.Client.Console`项目中，在命令提示符或终端中添加一个用于GraphQL服务的客户端，如下所示：

    [PRE82]

1.  注意以下输出结果：

    [PRE83]

1.  在`Northwind.GraphQL.Client.Console`项目中，在`.graphqlrc.json`文件中添加一个条目来控制代码生成期间使用的C#命名空间，如下所示：

    [PRE84]

1.  在`Northwind.GraphQL.Client.Console`项目中，添加一个名为`seafoodProducts.graphql`的新文件，该文件定义了一个获取海鲜产品的查询，如下所示：

    [PRE85]

    Strawberry Shake使用的GraphQL查询必须命名。

1.  如果你正在使用Visual Studio 2022，它可能会自动修改项目文件以显式地从构建过程中删除此文件，因为它不认识它。如果是这样，则删除或注释掉该元素，如下所示：

    [PRE86]

    **良好实践**：必须至少有一个`.graphql`文件，以便Strawberry Shake工具能够自动生成其代码。像下面这样的元素将阻止Strawberry Shake工具生成代码，你稍后将会遇到编译错误。你应该删除或注释掉该元素。

1.  构建项目`Northwind.GraphQL.Client.Console`，以便Strawberry Shake处理GraphQL查询文件并生成代理类。

1.  注意自动生成的`obj\Debug\net8.0\berry`文件夹，名为`NorthwindClient.Client.cs`的文件，以及它定义的十几个类型，包括`INorthwindClient`接口，如图*12.10*所示：

![图片](img/B19587_12_10.png)

图12.10：Northwind GraphQL服务生成的类文件

1.  在`Program.cs`中，删除现有的语句。添加语句以创建一个新的服务集合，将自动生成的`NorthwindClient`添加到其中，并使用服务正确的URL，然后获取并使用依赖服务来获取海鲜产品，如下所示：

    [PRE87]

1.  运行控制台应用程序并注意以下输出结果：

    [PRE88]

# 实现GraphQL突变

大多数服务都需要修改数据以及查询数据。GraphQL 将这些称为 **突变**。一个突变有三个相关组件：

+   突变本身，它定义了对图所做的更改。它应该使用动词、名词并用驼峰式命名，例如，`addProduct`。

+   **输入** 是突变的输入，并且应该与突变具有相同的名称，后缀为 `Input`，例如，`AddProductInput`。尽管只有一个输入，但它是一个对象图，因此可以像你需要的那样复杂。

+   **负载** 是突变的返回文档，并且应该与突变具有相同的名称，后缀为 `Payload`，例如，`AddProductPayload`。尽管只有一个负载，但它是一个对象图，因此可以像你需要的那样复杂。

## 将突变添加到 GraphQL 服务中

让我们定义添加的突变，稍后，我们将定义一些用于更新和删除产品的突变：

1.  在 `Northwind.GraphQL.Service` 项目/文件夹中，添加一个名为 `Mutation.cs` 的类文件。

1.  在类文件中，定义一个记录和两个类来表示执行 `addProduct` 突变所需的三个类型，如下面的代码所示：

    [PRE89]

1.  在 `Program.cs` 中，添加对 `AddMutationType<T>` 方法的调用以注册你的 `Mutation` 类，如下面的代码所示：

    [PRE90]

## 探索添加产品突变

现在，我们可以使用 Banana Cake Pop 探索突变：

1.  使用 `https` 配置文件启动 `Northwind.GraphQL.Service` 项目，不进行调试。

1.  在 **Banana Cake Pop** 中，点击 **+** 打开一个新的标签页。

1.  点击 **模式定义** 选项卡，并注意突变类型，如图 12.11 部分所示：

![](img/B19587_12_11.png)

图 12.11：使用 GraphQL 突变修改产品模式

1.  注意 `addProduct` 突变及其相关类型的完整模式定义，如下面的代码所示：

    [PRE91]

1.  点击 **操作** 选项卡，如果需要，创建一个新的空白文档，然后输入一个突变来添加一个名为 `Tasty Burgers` 的新产品。然后，从返回的 `product` 对象中，只需选择 ID 和名称，如下面的代码所示：

    [PRE92]

1.  点击 **运行**，注意新产品已成功添加，并由 SQL Server 数据库分配了下一个连续编号，这可能是任何大于 77 的数字，具体取决于你是否已经添加了一些其他产品，如下面的输出和 *图 12.12* 所示：

    [PRE93]

    **警告！** 请注意分配给新添加产品的 ID。在下一节中，你将更新此产品然后删除它。你不能删除任何 ID 在 1 到 77 之间的现有产品，因为它们与其他表相关联，这样做会引发引用完整性异常！

![](img/B19587_12_12.png)

图 12.12：使用 GraphQL 突变添加新产品

1.  关闭浏览器，并关闭 web 服务器。

## 将更新和删除作为突变实现

接下来，我们将定义突变来更新产品的单价，所有“单位”字段，以及删除一个产品：

1.  在`Mutation.cs`中，定义三个`record`类型来表示执行两个`updateProduct`和一个`deleteProduct`突变所需的输入，如下所示代码：

    [PRE94]

1.  在`Mutation.cs`中，定义两个类类型来表示从`update`或`delete`突变返回结果所需的类型，包括操作是否成功，如下所示代码：

    [PRE95]

1.  在`Mutation.cs`文件中，在`Mutation`类中，定义三个方法来实现两个`updateProduct`和一个`deleteProduct`突变，如下所示代码：

    [PRE96]

1.  如果你的数据库服务器没有运行，例如，因为你正在Docker、虚拟机或云中托管它，那么请确保启动它。

1.  启动`Northwind.GraphQL.Service`项目，使用`https`配置文件而不进行调试。

1.  在**香蕉蛋糕棒**中，点击**+**来打开一个新标签页。

1.  编写一个命名查询来请求你添加的产品，例如，使用`productId`大于`77`，如下所示标记：

    [PRE97]

1.  点击**运行**，并注意响应包括你之前添加的新产品，单价为`40`，如下所示输出：

    [PRE98]

1.  记录你添加的产品的`productId`。在我的情况下，它是`79`。

1.  在**香蕉蛋糕棒**中，点击**+**来打开一个新标签页。

1.  输入一个突变来更新你新产品的单价为`75`，然后从返回的`product`对象中，仅选择ID、名称、单价和库存单位，如下所示代码：

    [PRE99]

1.  点击**运行**，并注意响应，如下所示输出：

    [PRE100]

1.  点击**+**来打开一个新标签页，输入一个突变来更新现有产品的单位，然后从返回的`product`对象中仅选择ID、名称、单价和库存单位，如下所示代码：

    [PRE101]

1.  点击**运行**，并注意响应，如下所示输出：

    [PRE102]

1.  在查询标签页中请求新产品，点击**运行**，并注意响应，如下所示输出：

    [PRE103]

1.  点击**+**来打开一个新标签页，并输入一个突变来删除产品并显示是否成功，如下所示代码：

    [PRE104]

    **警告！**你将无法删除在其他表中引用的产品。ID 1到77将抛出引用完整性异常。

1.  点击**运行**，并注意响应，如下所示输出：

    [PRE105]

1.  确认产品已被删除，可以通过重新运行查询新产品的查询来验证，并注意你将得到一个空数组，如下所示输出：

    [PRE106]

1.  关闭浏览器，并关闭Web服务器。

# 实现GraphQL订阅

GraphQL订阅默认通过WebSockets工作，但也可以通过**服务器发送事件**（**SSE**）、SignalR或甚至gRPC工作。

想象一下，一个客户端应用程序希望在产品单价降低时收到通知。如果客户端能够订阅一个在单价降低时被触发的事件，而不是必须查询单价的变化，那就太好了。

## 向 GraphQL 服务添加订阅和主题

让我们将此功能添加到我们的 GraphQL 服务中，使用订阅：

1.  添加一个名为 `ProductDiscount.cs` 的新类文件。

1.  修改内容以定义一个模型，通知客户端关于产品单价降低的信息，如下面的代码所示：

    [PRE107]

1.  添加一个名为 `Subscription.cs` 的新类文件。

1.  修改内容以定义一个名为 `OnProductDiscounted` 的订阅事件（也称为主题），客户端可以订阅，如下面的代码所示：

    [PRE108]

1.  在 `Mutation.cs` 的 `UpdateProductPriceAsync` 方法中，添加语句，在产品单价降低时通过主题发送消息，如下面的代码所示：

    [PRE109]

1.  在 `Program.cs` 中，配置 GraphQL 服务以注册 `Subscription` 类，并将活动订阅存储在内存中，如下面的代码所示：

    [PRE110]

    除了内存中，您还可以使用 Redis 和其他数据存储来跟踪活动订阅。

1.  可选地，在构建**app**后，配置 WebSocket 的使用，如下面的代码所示：

    [PRE111]

    这是个可选步骤，因为 GraphQL 服务可以回退到使用 `https` 上的 SSE。

## 探索订阅主题

让我们订阅一个主题并查看结果：

1.  使用**https**配置文件启动**Northwind.GraphQL.Service**项目，不进行调试。

1.  在**香蕉蛋糕棒**中，点击**+**以打开新标签页。

1.  点击**模式参考**标签页，点击**订阅**类型，并注意名为**onProductDiscounted**的主题，如图*12.13*所示：

![](img/B19587_12_13.png)

图12.13：带有主题的订阅

1.  点击**操作**，输入对主题的订阅，并选择要在结果中显示的所有字段，如下面的代码所示：

    [PRE112]

1.  点击**运行**，并注意订阅开始但尚未显示任何结果。

1.  点击**+**以打开新标签页，并输入一个更新现有产品 `1` 的单价为 `8.99` 的变异。然后从返回的 `product` 对象中选择 ID 和单价，并显示更新是否成功，如下面的代码所示：

    [PRE113]

1.  点击**运行**，并注意以下输出中的响应：

    [PRE114]

1.  切换回包含订阅的标签页，并注意响应以及订阅仍然处于活动状态，这可以通过标签页上的旋转器和**取消**按钮来指示，如图*12.14*所示：

![](img/B19587_12_14.png)

图12.14：活动订阅显示旋转器

1.  切换回包含更新变异的标签页，将单价更改为 `7.99`，然后点击**运行**。然后切换回包含订阅的标签页，并注意它也收到了该更新通知。

1.  切换回更新变体的标签页，将单价更改为 `9.99`，然后点击 **运行**。然后，切换回订阅的标签页，并注意它没有收到任何通知，因为单价是增加的，而不是减少的。

1.  关闭浏览器，并关闭 web 服务器。

# 练习和探索

通过回答一些问题、进行一些实际操作练习，以及通过深入研究本章主题来测试你的知识和理解。

## 练习 12.1 – 测试你的知识

回答以下问题：

1.  GraphQL 服务使用什么传输协议？

1.  GraphQL 使用什么媒体类型进行其查询？

1.  你如何参数化 GraphQL 查询？

1.  使用 Strawberry Shake 而不是常规 HTTP 客户端进行 GraphQL 查询有哪些好处？

1.  你如何将新产品插入 Northwind 数据库？

## 练习 12.2 – 探索主题

使用下一页上的链接了解本章涵盖主题的更多详细信息：

[https://github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-12---combining-data-sources-using-graphql](https://github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-12---combining-data-sources-using-graphql)

## 练习 12.3 – 练习构建 .NET 客户端

在 `HomeController.cs` 中，添加一个名为 `Categories` 的操作方法，并实现它以查询 `categories` 字段，其中包含一个用于 `id` 的变量。在页面上，允许访问者提交 `id`，并注意类别信息和其产品列表。

# 摘要

在本章中，你学习了以下内容：

+   一些 GraphQL 的概念。

+   如何构建一个 `Query` 类，其中包含表示可查询实体的字段。

+   如何使用 Banana Cake Pop 工具探索 GraphQL 服务架构。

+   如何使用 REST 客户端扩展向 GraphQL 服务 POST 数据。

+   如何为 GraphQL 服务创建 .NET 客户端。

+   如何实现 GraphQL 变更。

+   如何实现 GraphQL 订阅。

在下一章中，你将了解可以用来实现高效微服务的 gRPC 服务技术。

# 在 Discord 上了解更多

要加入本书的 Discord 社区——在那里您可以分享反馈、向作者提问，并了解新书发布——请扫描下面的二维码：

[https://packt.link/apps_and_services_dotnet8](https://packt.link/apps_and_services_dotnet8)

![二维码](img/QR_Code3048220001028652625.png)
