# 集成外部组件和处理

到目前为止，我们一直在开发我们的 FlixOneStore。在上一章中，我们添加了购物车和发货功能。然而，某些组织可能不需要这样的设施，因为某些组织已经拥有一切。例如，我们的 FlixOneStore 需要一个外部组件来帮助我们跟踪分配和支付管理系统。

在本章中，我们将通过代码示例来讨论外部组件。我们将主要涵盖以下主题：

+   理解中间件

+   在中间件中添加日志记录到我们的 API

+   通过构建自己的中间件来拦截 HTTP 请求和响应

+   JSON-RPC 用于 RPC 通信

# 理解中间件

如其名所示，中间件是一种连接两个不同或相似位置的软件。在软件工程的世界里，中间件是一个软件组件，它被组装到应用程序管道中，以处理请求和响应。

这些组件还可以检查请求是否应该传递到下一个组件，或者请求是否应该在下一个组件触发/调用之前或之后由组件处理。这个请求管道是通过使用请求代表构建的。这个请求代表与每个 HTTP 请求交互。

查看以下来自 ASP.NET Core 文档的引用（[https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/))：

“中间件是组装到应用程序管道中以处理请求和响应的软件。”

查看以下图表，展示了一个简单的中间件组件的示例：

![图片](img/9bce228e-7b7b-4e9c-8088-5adfcb279a1f.png)

# 请求代表

请求由 `Use`、`Run`、`Map` 和 `MapWhen` 扩展方法处理。这些方法配置请求代表。

为了更详细地了解这一点，让我们使用 `ASP.NET Core` 创建一个模拟项目。按照以下步骤操作：

1.  打开 Visual Studio。

1.  转到“文件”|“新建”|“项目”，或按 *Ctrl* + *Shift* + *N*。参考以下截图：

![图片](img/c0b2f086-c173-4d34-bd96-36a9f72eb2aa.png)

使用 Visual Studio 2017 创建新项目

1.  从“新建项目”屏幕中选择 ASP.NET Core Web 应用程序。

1.  命名您的新项目（例如 `Chap05_01`），选择位置，然后单击“确定”，如下截图所示：

![图片](img/230053ba-e018-446e-bc6d-e6fd627bb5b1.png)

选择新项目模板

1.  从新的 ASP.NET Core Web 应用程序模板屏幕中选择 API 模板。确保您选择了 .NET Core 和 ASP.NET Core 2.0。

1.  单击“确定”，如下截图所示：

![图片](img/f27c348c-5b5b-49e9-9f5b-149c6bae1232.png)

1.  打开解决方案资源管理器。您将看到文件/文件夹结构，如下截图所示：

![图片](img/59fe708a-495b-4023-b55d-7743d92329ce.png)

显示 Chap05_01 项目的文件/文件夹结构

从我们刚刚创建的模拟项目中打开 `Startup.cs` 文件，查看包含以下代码的 `Configure` 方法：

[PRE0]

上述代码是自我解释的：它告诉系统通过启动 `Microsoft.AspNetCore.Builder.IApplicationBuilder` 的 `app.UseMvc()` 扩展方法将 `Mvc` 添加到请求管道中。

你可以在 [https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.iapplicationbuilder?view=aspnetcore-2.0](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.iapplicationbuilder?view=aspnetcore-2.0) 获取有关 `IApplicationBuilder` 的更多信息。

它还指示系统在开发环境中使用特定的异常页面。前面的方法配置了应用程序。

在下一节中，我们将详细讨论四个重要的 `IApplicationBuilder` 方法。

# 使用

`Use` 方法向应用程序请求管道中添加一个委托。请查看以下截图以了解此方法的签名：

![图片](img/291321c1-da74-4bc7-85da-9ab284eee7ff.png)

`Use` 方法的签名

正如我们在上一节中讨论的，中间件方法可以短路请求管道或将请求传递给下一个委托。

短路请求不过是结束请求。

请查看以下 `Use` 方法的代码：

[PRE1]

在前面的代码中，我试图通过使用局部函数来解释 `Use` 方法的模拟实现。在这里，你可以看到 `Middleware` 在 `await next.Invoke();` 之前或之后调用或传递请求给下一个委托。你可以编写/实现其他代码片段，但这些代码片段不应该向客户端发送响应，例如那些写入输出、产生 404 状态等的代码片段。

**局部函数**是在方法内部声明的函数，可以在方法的范围内调用。这些方法只能由另一个方法使用。

# 运行

`Run` 方法与 `Use` 方法以相同的方式向请求管道中添加一个委托，但此方法会终止请求管道。请查看以下截图以了解此方法的签名：

![图片](img/6a5a0166-c10b-490c-8ccc-a4454aabc2a0.png)

请查看以下代码：

[PRE2]

在前面的代码中，我试图通过使用局部函数 `RequestDelegate` 来展示 `Run` 终止请求管道。在这里，我使用了局部函数。

你可以看到我在此之前添加了一个控制台记录器，并且有空间添加更多的代码片段，但不是那些将响应发送回客户端的代码片段。在这里，`Run` 通过返回一个字符串来终止。运行 Visual Studio 或按 *F5* 键，你将得到类似于以下截图的输出：

![图片](img/47ec67d8-9b78-4440-8bb0-223ef2a17710.png)

# 映射

`Map` 方法有助于当你想要连接多个中间件实例时。为此，`Map` 调用另一个请求委托。请查看以下截图以了解此方法的签名：

![](img/9e767d6f-3229-463b-9373-8c8ec0ac298a.png)^(Map方法的签名)

看看下面的代码：

[PRE3]

在此代码中，我添加了一个`Map`，它只是映射`<url>/testroute`。接下来是之前讨论过的相同的`Run`方法。`TestRoutehandler`是一个私有方法。看看下面的代码：

[PRE4]

在`app.Run(Handler);`之前是一个正常的委托。现在，运行代码并查看结果。它们应该类似于以下截图：

![](img/93907e52-1b91-46c5-9e62-7b3ab8e77ac8.png)

你可以看到，Web应用程序的根目录显示了在`Run`委托方法中提到的字符串。你将得到以下截图所示的输出：

![](img/a5cb5a2a-d3dd-417c-8f89-d9d3ddb0df73.png)

# 在中间件中为我们的API添加日志记录

简而言之，日志记录就是将日志文件集中在一起的过程或行为，以获取API在通信期间发生的事件或其他操作。在本节中，我们将为我们的产品API实现日志记录。

在我们开始查看如何记录API的事件之前，让我们先快速看一下我们现有的产品API。

参考以下 *请求委托* 部分，以刷新你对如何创建一个新的ASP.NET Core项目的记忆。

以下截图显示了我们的产品API的项目结构：

![](img/84ad0efb-4932-4e26-82e1-460cbc488d84.png)

这里是我们的`Product`模型：

[PRE5]

`Product`模型是一个表示产品的类，包含属性。

这里是我们的存储库接口：

[PRE6]

`IProductRepository`接口有我们API开始时对产品进行操作所需的方法。

让我们看看我们的`ProductRepository`类：

[PRE7]

`ProductRepository`类实现了`IProductRepository`接口。前面的代码是自我解释的。

打开 `Startup.cs` 文件并添加以下代码：

[PRE8]

为了支持我们的产品API的Swagger，你需要添加`Swashbuckle.ASPNETCore` NuGet包。

现在，打开`appsettings.json`文件并添加以下代码：

[PRE9]

让我们看看我们的`ProductController`包含什么：

[PRE10]

上述代码是我们产品API的`GET`资源。它调用我们的`ProductRepository`的`GetAll()`方法，转换响应，并返回它。在之前的代码中，我们已经指示系统使用`ProductRepository`类解析`IProductRepository`接口。参考`Startup`类。

这里是转换响应的方法：

[PRE11]

前面的代码接受一个`Product`类型的参数，然后返回一个`ProductViewModel`类型的对象。

以下代码显示了我们的控制器构造函数是如何注入的：

[PRE12]

在前面的代码中，我们注入了我们的`ProductRepository`，并且每当有人调用产品API的任何资源时，它将自动初始化。

现在，你已经准备好玩这个应用程序了。从菜单中运行应用程序或点击*F5*。在网页浏览器中，你可以在地址的URL后使用后缀`/swagger`。

完整的源代码，请参考GitHub仓库链接：[https://github.com/PacktPublishing/Building-RESTful-Web-services-with-DOTNET-Core](https://github.com/PacktPublishing/Building-RESTful-Web-services-with-DOTNET-Core)。

它将显示Swagger API文档，如下所示：

![图片](img/81c11e65-4875-466e-a068-7a62e931b066.png)

点击`GET /api/Product/productlist`资源。它将返回产品列表，如下所示：

![图片](img/2d825f3e-e331-434a-8d31-ad740b96efc1.png)

让我们为我们的API实现日志记录功能。请注意，为了使我们的演示简短简单，我没有添加复杂场景来跟踪一切。我只是添加了简单的日志来展示日志记录功能。

要开始为我们的产品API实现日志记录，在名为`Logging`的新文件夹中添加一个名为`LogAction`的新类。以下是`LogAction`类的代码：

[PRE13]

上述代码包含的常量只是我们应用程序的操作，也称为**事件**。

更新我们的`ProductController`；现在它应该看起来像以下代码：

[PRE14]

在上述代码中，我们添加了一个`ILogger`接口，它来自依赖注入容器（更多详情请见[https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-2.0 ](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-2.0)）。

让我们在产品API的`GET`资源中添加日志记录功能：

[PRE15]

上述代码返回产品列表并记录信息。

要测试此场景，我们需要一个客户端或API工具，以便我们可以看到输出。为此，我们将使用`Postman`扩展（更多详情请见[ https://www.getpostman.com/ ](https://www.getpostman.com/)）。

首先，我们需要运行应用程序。为此，打开Visual Studio命令提示符，移动到您的项目文件夹，然后输入命令`dotnet run`。您将看到如下所示的类似消息：

![图片](img/adeff017-ecbf-4077-bd70-3c2e468bb408.png)

现在，启动Postman并调用`GET /api/product/productlist`资源：

![图片](img/9f77c310-49f6-4d9c-b833-0d264193c9bd.png)

点击发送按钮，你期望返回产品列表，但情况并非如此，如下所示：

![图片](img/99026730-2294-43ba-85c6-535d9e4cac95.png)

上述异常发生是因为我们在`ProductController`中使用了一个非泛型类型，该类型不可注入。

因此，我们需要对我们的`ProductController`进行一些小的修改。看看以下代码片段：

[PRE16]

在上述代码中，我添加了一个泛型`ILogger<ProductController>`类型。由于它是可注入的，它将自动解析。

与其早期版本相比，.NET Core 2.0 中的日志记录略有不同。默认情况下，非泛型 `ILogger` 的实现不可用，但 `ILogger<T>` 可用。如果您想使用非泛型实现，请使用 `ILoggerFactory` 而不是 `ILogger`。

在这种情况下，我们的 `ProductController` 构造函数将如下所示：

`private readonly IProductRepository _productRepository;`

`private readonly ILogger _logger;`

`public ProductController(IProductRepository productRepository, ILoggerFactory logger)`

`{`

`_productRepository = productRepository;`

`_logger = logger.CreateLogger("Product logger");`

`}`

打开 `Program` 类并更新它。它应该看起来像以下代码片段：

[PRE17]

您还需要更新 `appsettings.json` 文件并为记录器编写更多代码，以便您的文件看起来像以下片段：

[PRE18]

现在，再次打开 Visual Studio 命令提示符并写入 `dotnet build` 命令。它将构建项目，您将收到类似于以下截图的消息：

![](img/712f7568-3d05-4989-87ea-78badf111be9.png)

从这一点开始，如果您运行 Postman，它将给出以下截图所示的结果：

![](img/955ca74a-b21f-4f99-8816-ddfe33cb32e5.png)

上述代码添加了记录操作的能力。您将收到类似于以下截图所示的类似日志操作：

![](img/4f57ea89-3a05-4cc4-841f-72f8095a8bf3.png)

在这里，我们编写了一些使用默认 `ILogger` 的代码。我们使用了默认方法来调用记录器；然而，在某些场景中，我们需要一个定制的记录器。在下一节中，我们将讨论如何编写用于自定义记录器的中间件。

# 通过构建自己的中间件来拦截 HTTP 请求和响应

在本节中，我们将为现有应用程序创建自己的中间件。在这个中间件中，我们将记录所有请求和响应。让我们按以下步骤进行：

1.  打开 Visual Studio。

1.  通过单击文件 | 打开 | 项目/解决方案（或按 *Ctrl* + *Shift* + *O*）打开 Product APIs 的现有项目，如图以下截图所示：

![](img/c7d1d3df-b9e8-4033-b1ef-d18001ade638.png)

1.  定位到您的解决方案文件夹并单击打开，如图以下截图所示：

![](img/61661159-326b-4c8a-81da-d7b943efb169.png)

1.  打开解决方案资源管理器，通过右键单击项目名称添加一个新文件夹，并将其命名为 `Middleware`，如图以下截图所示：

![](img/01dd79d4-1500-4331-a7e8-5935fb70d39d.png)

1.  右键单击 `Middleware` 文件夹并选择添加 | 新项。

1.  从网络模板中，选择中间件类并将新文件命名为 `FlixOneStoreLoggerMiddleware`。然后单击添加，如图以下截图所示：

![](img/bbc65dd4-b76a-4fd4-9cee-0f5b13150f20.png)

您的文件夹层次结构应类似于以下截图所示：

![](img/da2e2fcb-191b-4c64-91ae-cf6b04d6f02d.png)

感谢Justin Williams提供的POST资源的解决方案；他的解决方案可在[https://github.com/JustinJohnWilliams/RequestLogging](https://github.com/JustinJohnWilliams/RequestLogging)找到。

查看以下我们的`FlixOneStoreLoggerMiddleware`类的代码片段：

[PRE19]

在前面的代码中，我们只是利用内置的DI通过`RequestDelegate`来创建我们的自定义中间件。

以下代码展示了我们如何为日志配置所有请求和响应：

[PRE20]

参考本章中的“*请求委托*”部分，其中我们探讨了中间件。在前面的代码中，我们只是通过`ILogger`泛型类型帮助记录请求和响应。`await _next(httpContext);`行继续请求管道。

打开`Setup.cs`文件，并在`Configure`方法中添加以下代码：

[PRE21]

在前面的代码中，我们利用`ILoggerFactory`并添加`Console`和`Debug`来记录请求和响应。`UseFlixOneLoggerMiddleware`方法实际上是一个扩展方法。为此，将以下代码添加到`FlixOneStoreLoggerExtension`类中：

[PRE22]

现在，每当任何请求到达我们的产品API时，日志应该会显示，如下面的截图所示：

![](img/a7b36f72-6571-4284-860f-f2c2f0ed525b.png)

在本节中，我们创建了一个自定义中间件，并记录了所有请求和响应。

# JSON-RPC用于RPC通信

JSON-RPC是一个无状态的、轻量级的远程过程调用（RPC）协议。规范（即，JSON-RPC 2.0规范（见[http://www.jsonrpc.org/specification](http://www.jsonrpc.org/specification)获取更多详细信息））定义了各种数据结构和它们的处理规则。

主要对象按规范在以下部分中展示。

# 请求对象

`Request`对象代表发送到服务器的任何调用/请求。`Request`对象具有以下成员：

+   **jsonrpc**：一个表示JSON-RPC协议版本的字符串。它**必须**准确（在这种情况下，版本2.0）。

+   **method**：一个字符串，包含要调用的方法名称。以单词`rpc`开头并后跟一个点字符（U+002E或ASCII 46）的方法名称被限制为rpc-内部方法和扩展，**不得**用于其他任何目的。

+   **params**：一个结构化值，主导参数值。在整个方法调用过程中都要使用此成员。此成员**可能**被删除。

+   **id**：客户端固定的一个标识符，如果构成，必须有字符串、数字或**null**值。

# 响应对象

根据规范，每当对服务器进行调用时，服务器必须有一个响应。`Response`以一个包含以下成员的单个JSON对象表示：

+   **jsonrpc**：一个表示JSON-RPC协议版本的字符串

+   **result**：一个必需的成员，如果请求成功

+   **error**：如果发生错误，一个必需的成员

+   **id**：一个必需的成员

在本节中，我们概述了JSON-RPC规范2.0。

# 摘要

在本章中，我们讨论了与支付网关、订单跟踪、通知服务等相关的外部API/组件的集成。我们还通过实际代码实现了它们的功能。

测试是我们帮助代码无错误的唯一过程。它也是所有希望使代码整洁和可维护的开发者的实践。在下一章中，我们将涵盖日常开发活动中的测试范式。我们将讨论与测试范式相关的一些重要术语。我们还将涵盖这些术语的理论，然后我们将涵盖代码示例，查看存根和模拟，并了解集成、安全和性能测试。
