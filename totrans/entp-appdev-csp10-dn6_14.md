# *第11章*：创建ASP.NET Core 6网络应用程序

到目前为止，我们已经构建了应用程序的所有核心组件，如数据访问层和服务层，所有这些组件主要是服务器端组件，也称为**后端组件**。

在本章中，我们将构建我们的电子商务应用程序的呈现层/用户界面（**UI**），也称为**客户端组件**。UI是应用程序的界面；一个好的呈现层不仅有助于保持用户对应用程序的参与度，而且鼓励用户返回应用程序。这在企业应用程序中尤为重要，一个好的呈现层有助于用户轻松地浏览应用程序，并帮助他们轻松地执行依赖于应用程序的日常活动。

我们将专注于理解ASP.NET Core MVC，并使用ASP.NET Core MVC开发一个网络应用程序。主要涵盖以下主题：

+   前端网页开发简介

+   将API与服务层集成

+   创建控制器和操作

+   使用ASP.NET Core MVC创建用户界面

+   理解Blazor

# 技术要求

对于本章，你需要具备基本的C#、.NET Core、HTML和CSS知识。本章的代码示例可以在这里找到：[https://github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition/tree/main/Chapter11/RazorSample](https://github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition/tree/main/Chapter11/RazorSample).

你可以在这里找到更多代码示例：[https://github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition/tree/main/Enterprise%20Application](https://github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition/tree/main/Enterprise%20Application).

# 前端网页开发简介

呈现层主要是指浏览器可以渲染并显示给用户的代码。每当一个页面在浏览器中加载时，它会创建一个包含各种元素（如文本框和标签）的层次结构，这些元素都存在于页面上。这个层次结构被称为**文档对象模型**（**DOM**）。

一个好的前端主要在于能够根据需要操作DOM，并且有许多技术/库支持使用网络事实上的语言JavaScript动态地操作DOM和加载数据。无论是简化JavaScript使用的jQuery，还是支持完整客户端渲染的完整客户端框架，如Angular、React或Vue，或者是ASP.NET Core框架，如ASP.NET Core MVC、Razor Pages或Blazor，它们都归结为处理网络的三个主要构建块：HTML、CSS和JavaScript。让我们来看看这三个构建块：

+   **超文本标记语言**（**HTML**）：正如全称所述，HTML 是一种浏览器可以理解和显示内容的标记语言。它主要包含一系列称为 **HTML 元素** 的标签，允许开发者定义页面的结构。例如，如果你想创建一个需要允许用户输入他们的名字和姓氏的表单，可以使用输入 HTML 元素来定义它。

+   **层叠样式表**（**CSS**）：表示层全部关于以吸引用户的方式展示数据，并确保无论用户尝试在哪种设备/分辨率上加载应用程序，应用程序都是可用的。这正是 CSS 在定义浏览器上内容显示方面发挥关键作用的地方。它控制各种事情，如页面的样式、应用程序的主题和调色板，更重要的是，它使它们具有响应性，这样用户在使用应用程序时，无论是在移动设备还是桌面设备上，都能获得相同的体验。

现代网络开发的优点是我们不需要从头开始编写一切，许多库都可以直接选择并用于应用程序中。我们将在 *使用 ASP.NET Core MVC 创建 UI* 部分中使用这样一个库，其解释如下。

+   **JavaScript**: JavaScript 是一种脚本语言，有助于执行各种高级动态操作，例如，验证在表单中输入的文本或类似启用/禁用 HTML 元素条件性或从 API 获取数据等操作。JavaScript 为网页提供了更多功能，并添加了许多开发者可以用来在客户端执行高级操作的编程特性。就像 HTML 和 CSS 一样，所有浏览器都能理解 JavaScript，它构成了表示层的一个重要部分。所有这些组件都可以相互链接，如下面的图所示：

![图 11.1 – HTML、CSS 和 JavaScript

](img/Figure_11.1_B18507.jpg)

图 11.1 – HTML、CSS 和 JavaScript

注意

HTML、CSS 和 JavaScript 互相依存，在开发客户端/前端应用程序中发挥着重要作用，需要专门的书籍来全面解释。一些相关链接可以在 *进一步阅读* 部分找到。

现在我们已经了解了 HTML、CSS 和 JavaScript 的重要性，我们需要知道如何使用它们来构建网页应用程序的表示层，以便它能够支持具有不同分辨率的多个浏览器，并且能够管理状态（HTTP 是无状态的）。

一种技术可能是创建所有 HTML 页面并在 Web 服务器上托管它们；然而，虽然这对于静态站点工作得很好，并且也涉及从头开始构建一切，但如果我们希望内容更加动态并且想要丰富的 UI，我们需要使用能够动态生成 HTML 页面并提供与后端无缝交互支持的技术。让我们在下一节中看看可以用来生成动态 HTML 的各种技术。

## Razor 语法

在我们开始理解 ASP.NET Core 提供的各种可能框架之前，让我们首先了解什么是 Razor 语法。它是一种标记语法，用于将服务器端组件嵌入到 HTML 中。我们可以使用 Razor 语法绑定任何动态数据以进行显示，或者将其从视图/页面发送回服务器以进行进一步处理。Razor 语法主要编写在 Razor 文件/Razor 视图页面上，这些文件不过是 C# 用来生成动态 HTML 的文件。它们带有 `.cshtml` 扩展名并支持 Razor 语法。Razor 语法由一个名为 **视图引擎** 的引擎处理，默认视图引擎被称为 **Razor 引擎**。

要嵌入 Razor 语法，我们通常使用 `@`，这告诉 Razor 引擎解析并生成 HTML。`@` 后可以跟任何 C# 内置方法来生成 HTML。例如，`<b>@DateTime.Now</b>` 可以用于在 Razor 视图/页面上显示当前日期和时间。除此之外，就像 C# 一样，Razor 语法也支持代码块、控制结构和变量等。以下图展示了通过 Razor 引擎的一些示例 Razor 语法：

![图 11.2 – Razor 语法

](img/Figure_11.2_B18507.jpg)

图 11.2 – Razor 语法

Razor 语法还支持定义 HTML 控件；例如，要定义一个文本框，我们可以使用以下 Razor 语法：

[PRE0]

上述代码被称为 `input` 标记助手，Razor 语法负责处理称为 **指令** 标记助手的绑定数据到 HTML 控件并生成丰富、动态的 HTML。让我们简要讨论一下：

+   **指令**：在底层，每个 Razor 视图/页面都由 Razor 引擎解析，并使用一个 C# 类来生成动态 HTML，然后将其发送回浏览器。指令可以用来控制这个类的行为，进而控制生成的动态 HTML。

例如，`@using` 指令可以用于在 Razor 视图/页面上包含任何命名空间，或者 `@code` 指令可以用于包含任何 C# 成员。

最常用的指令之一是 `@model`，它允许你将模型绑定到视图，这有助于验证视图的类型以及帮助 Intellisense。将视图绑定到特定类/模型的过程称为 **强类型** 视图。在我们的电子商务应用程序中，我们将对所有视图进行强类型处理，这将在 *使用 ASP.NET Core MVC 创建 UI* 部分中看到。

+   **标签助手**：如果你在 ASP.NET Core 之前使用过 ASP.NET MVC，你可能会遇到 HTML 助手，这些是帮助绑定数据和生成 HTML 控件的类。

然而，在 ASP.NET Core 中，我们有标签助手，它们帮助我们将数据绑定到 HTML 控件。与 HTML 助手相比，标签助手的优点是它们使用与 HTML 相同的语法，并为标准 HTML 控件分配了额外的属性，这些属性可以由动态数据生成。例如，要生成一个 HTML 文本框控件，通常我们会编写以下代码：

[PRE1]

使用标签助手，这将被重写为以下代码，其中 `@Name` 是视图强类型关联的模型属性：

[PRE2]

因此，正如你所看到的，这完全是关于编写 HTML，但利用 Razor 标记来生成动态 HTML。ASP.NET Core 内置了许多标签助手，更多关于它们的信息可以在以下链接中找到：[https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/?view=aspnetcore-6.0](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/?view=aspnetcore-6.0)。

不需要了解/记住每个标签助手，我们将在开发应用程序 UI 时使用此参考文档。

注意

由于 Razor 语法是标记，因此不需要了解所有语法。以下链接可以用作 Razor 语法的参考：[https://docs.microsoft.com/en-us/aspnet/core/mvc/views/razor?view=aspnetcore-6.0](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/razor?view=aspnetcore-6.0)。

因此，让我们来看看 ASP.NET Core 以及其他常见框架中用于开发表示层的各种选项。

## 探索 Razor Pages

Razor Pages 是使用 ASP.NET Core 实现网络应用程序的默认方式。Razor Pages 依赖于拥有一个可以直接处理请求的 Razor 文件的概念，以及与该 Razor 文件相关联的可选 C# 文件，用于任何额外的处理。可以使用以下命令创建一个典型的 Razor 应用程序：

[PRE3]

正如你所看到的，项目包含 Razor 页面及其对应的 C# 文件。在打开任何 Razor 视图时，我们会看到一个名为 `@page` 的指令，它有助于浏览页面。例如，`/index` 将被路由到 `index.cshtml`。重要的是，所有 Razor 页面都必须在页面顶部包含 `@page` 指令，并且放置在 `Pages` 文件夹中，因为 ASP.NET Core 运行时会在此文件夹中查找所有 Razor 页面。

可以通过使用另一个名为 `@model` 的指令将 Razor 页面进一步关联到一个 C# 类，也称为 `PageModel` 类。以下是 `index.cshtml` 页面的代码：

[PRE4]

[PRE5]

[PRE6]

[PRE7]

[PRE8]

[PRE9]

[PRE10]

[PRE11]

[PRE12]

[PRE13]

[PRE14]

[PRE15]

[PRE16]

[PRE17]

`PageModel` 类不过是一个可以具有特定 `GET` 和 `POST` 调用方法的 C# 类，以便从 API 等动态获取 Razor 页面上的数据。这个类需要继承自 `Microsoft.AspNetCore.Mvc.RazorPages.PageModel`，并且是一个标准的 C# 类。`index.cshtml` 的 `PageModel`，它是 `index.cshtml.cs` 的一部分，如下代码所示：

[PRE18]

[PRE19]

[PRE20]

[PRE21]

[PRE22]

[PRE23]

[PRE24]

[PRE25]

[PRE26]

[PRE27]

[PRE28]

[PRE29]

[PRE30]

[PRE31]

[PRE32]

[PRE33]

[PRE34]

[PRE35]

[PRE36]

[PRE37]

[PRE38]

在这里，你可以看到我们正在通过 `OnGet` 方法填充在 Razor 页面上使用的数据，这同样也被称为 `PageModel` 处理器，可以用于初始化 Razor 页面。像 `OnGet` 一样，我们可以添加一个 `OnPost` 处理器，它可以用于将数据从 Razor 页面提交回 `PageModel` 并进一步处理。

如果 `OnPost` 方法满足以下两个条件，它将自动绑定 `PageModel` 类中的所有属性：

+   属性被注解为 `BindProperty` 属性。

+   Razor 页面有一个与属性同名的 HTML 控件。

例如，如果我们想要绑定前面代码中 `select` 控件的值，我们需要首先在 `PageModel` 类中添加一个属性，如下代码所示：

[PRE39]

[PRE40]

然后，使用 `select` 控件的属性名，如这里所示，Razor Pages 将自动将选中的值绑定到这个属性：

[PRE41]

我们可以使用异步命名约定为 `OnGet` 和 `OnPost` 方法命名，如果我们在使用异步编程，它们可以命名为 `OnGetAsync/OnPostAsync`。

Razor Pages 还支持根据动词调用方法。方法名的模式应遵循 `OnPost[handler]/OnPost[handler]Async` 规范，其中 `[handler]` 是设置在任何标签助手 `asp-page-handler` 属性上的值。

例如，以下代码将调用对应 `PageModel` 类中的 `OnPostDelete/OnPostDeleteAsync` 方法：

[PRE42]

对于服务配置部分，Razor Pages 可以通过在 `Program` 类中使用 `AddRazorPages` 方法来配置，通过将服务添加到 ASP.NET Core 的 `MapRazorPages` 中间件，并在 `Program` 类中注入，如下代码所示。这样做是为了使所有 Razor 页面都可以使用页面名称进行请求：

[PRE43]

这完成了简单的 Razor 页面应用程序设置；我们在 [*第 9 章*](B18507_09_Epub.xhtml#_idTextAnchor860)，*在 .NET 6 中使用数据*，中看到了另一个示例，它使用了 Razor Pages 从数据库中检索数据，使用了 **Entity Framework Core** （**EF Core**）。

Razor Pages 是在 ASP.NET Core 中开发 Web 应用程序的最简单形式；然而，对于一种更结构化的开发 Web 应用程序的形式，可以处理复杂功能，我们可以选择 ASP.NET Core MVC。让我们在下一节中探索使用 ASP.NET Core MVC 开发 Web 应用程序。

## 探索 ASP.NET Core MVC 网站

如其名所示，ASP.NET Core MVC 基于第 10 章*创建 ASP.NET Core 6 Web API*中讨论的 MVC 模式，是 ASP.NET Core 中用于构建 Web 应用程序的框架。我们在第 10 章*创建 ASP.NET Core 6 Web API*中看到，ASP.NET Core Web API 也使用 MVC 模式；然而，ASP.NET Core MVC 还支持用于显示数据的视图。底层设计模式是相同的，其中我们有一个模型来存储数据，一个控制器来传输数据，以及视图来渲染和显示数据。

ASP.NET Core MVC 支持在第 10 章*创建 ASP.NET Core 6 Web API*中讨论的所有功能，例如路由、依赖注入、模型绑定和模型验证，并使用与 Web API 相同的启动技术，即使用 `Program` 类。就像 Web API 一样，.NET 6 应用程序服务和中间件在 `Program` 类中进行配置。

与 MVC 的一个主要区别是还需要加载视图，因此，我们不是在 `Program` 类中使用 `AddControllers`，而是需要使用 `AddControllersWithViews`。以下图示了一个示例：

![Figure 11.3 – MVC 请求生命周期

![img/Figure_11.3_B18507.jpg](img/Figure_11.3_B18507.jpg)

Figure 11.3 – MVC 请求生命周期

`AddControllersWithViews` 主要负责加载视图和处理控制器发送的数据，但最重要的是，它负责配置用于在视图中处理 Razor 语法并生成动态 HTML 的 Razor 引擎服务。

在 ASP.NET Core MVC 中，控制器操作需要根据 URL 中传递的动作名称进行路由，因此在路由部分，我们不是调用 `MapController`，而是配置 `MapControllerRoute` 并向其传递一个模式。因此，`UseEndpoints` 中间件中的默认路由配置看起来如下代码片段所示：

[PRE44]

[PRE45]

[PRE46]

[PRE47]

[PRE48]

[PRE49]

在这个模式中，我们正在告诉中间件，URL 的第一部分应该是 `controller` 名称，后面跟着动作名称和可选的 `id` 参数。如果 URL 中没有传递任何内容，则默认路由是 `ProductsController` 的 `Index` 动作方法。因此，这主要是我们在第 10 章*创建 ASP.NET Core 6 Web API*中讨论的基于约定的路由。

就像 Razor Pages 一样，ASP.NET Core MVC 应用程序中的视图支持 Razor 语法，并允许强类型视图；也就是说，一个视图可以绑定到一个模型进行类型检查，并且模型属性可以与具有编译时智能感知支持的 HTML 控件相关联。

由于 ASP.NET Core MVC 为应用程序提供了更多的结构，因此我们将使用 ASP.NET Core MVC 进行我们的表示层开发，这将在后续章节中详细讨论。

注意

总是有一个关于选择前端开发技术的常见问题。以下链接提供了一些关于这个主题的建议：[https://docs.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/choose-between-traditional-web-and-single-page-apps](https://docs.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/choose-between-traditional-web-and-single-page-apps)。在选择前端技术之前，应该评估所有优点和缺点，因为没有一种适合所有情况的要求。

在这个基础上，让我们继续到下一节，我们将开始将到目前为止开发的后端 API 与我们的表示层集成。

# 将 API 与服务层集成

在本节中，我们将开发 `Packt.Ecommerce.Web` ASP.NET Core MVC 应用程序，该应用程序是通过添加 `ASP.NET Core web application(Model-View-Controller)` 模板创建的。由于我们已经为表示层开发了各种所需的 API，因此我们首先将构建一个用于与这些 API 通信的包装类。

这是一个用于与各种 API 通信的单个包装类，因此让我们为这个类创建一个合约。为了简单起见，我们将要求限制到我们电子商务应用中最重要的工作流程，如下所示：

+   登录页面检索系统中的所有产品，并允许用户搜索/过滤产品。

+   查看产品的详细信息，将它们添加到购物车中，并能够添加更多产品到购物车。

+   完成订单并查看发票。

为了采用更结构化的方法，我们将将各种类和接口分离到不同的文件夹中。让我们在以下步骤中看看如何操作：

1.  首先，让我们向 `Packt.Ecommerce.Web` 项目添加一个名为 `Contracts` 的文件夹，并添加一个名为 `IECommerceService` 的接口。这个接口将包含以下方法：

    [PRE50]

1.  现在，让我们添加一个名为 `Services` 的文件夹，并添加一个名为 `EcommerceService` 的类。这个类将继承 `IECommerceService` 并实现所有方法。

1.  由于我们需要调用各种 API，我们需要使用 `Packt.Ecommerce.Common.Options.ApplicationSettings` 通过 `options` 模式来使用它。

`Program` 类将为我们的 MVC 应用程序配置以下服务：

+   `AddControllersWithViews`: 这将为 ASP.NET Core MVC 注入使用控制器和视图所需的服务。

+   `ApplicationSettings`: 这将使用以下代码使用 `IOptions` 模式配置 `ApplicationSettings` 类：

    [PRE51]

+   `AddHttpClient`: 这将注入 `System.Net.Http.IHttpClientFactory` 和相关类，这将允许我们创建一个 `HttpClient` 对象。此外，我们将配置重试策略和断路器策略，如在第 [*第10章*](B18507_10_Epub.xhtml#_idTextAnchor1040) 中所述，*创建 ASP.NET Core 6 Web API*。

+   使用 .NET Core DI 容器将 `EcommerceService` 映射到 `IECommerceService`。

1.  使用以下代码配置应用洞察：

    [PRE52]

接下来，关于中间件，我们将使用`Program`类注入以下中间件，除了默认的路由中间件之外：

+   `UseStatusCodePagesWithReExecute`: 这个中间件用于将请求重定向到除`500`错误代码外的自定义页面。在下一节中，我们将在`ProductController`中添加一个方法，该方法将被执行并基于错误代码加载相关视图。这个中间件接受一个字符串作为输入参数，这实际上是在出现错误时应执行的路由，并且为了传递错误代码，它允许使用占位符`{0}`。因此，中间件配置看起来如下所示：

    [PRE53]

+   **错误处理**：至于表示层，与API不同，我们需要将用户重定向到自定义页面，在运行时失败的情况下，该页面包含相关信息，例如用户友好的错误消息和可以用于稍后检索实际失败的相关的日志ID。然而，在开发环境中，我们可以显示完整的错误以及堆栈。因此，我们将配置两个中间件，如下面的代码所示：

    [PRE54]

在这里，我们可以看到，对于开发环境，我们使用的是`UseDeveloperExceptionPage`中间件，它将加载完整的异常堆栈跟踪，而对于非开发环境，我们使用的是`UseExceptionHandler`中间件，它接受需要执行的错误操作方法的路径。此外，在这里，我们不需要我们的自定义错误处理中间件，因为ASP.NET Core中间件负责将详细的错误记录到日志提供程序，在我们的例子中是应用洞察。

+   `UseStaticFiles`: 为了允许各种静态文件，如CSS、JavaScript、图像以及任何其他静态文件，我们不需要通过整个请求管道，这就是这个中间件发挥作用的地方，它允许服务静态文件并支持为静态文件短路其余的管道。

回到`EcommerceService`类，让我们首先定义这个类的局部变量和构造函数，它将使用以下代码注入`HTTPClient`工厂和`ApplicationSettings`：

[PRE55]

[PRE56]

[PRE57]

[PRE58]

[PRE59]

[PRE60]

[PRE61]

[PRE62]

[PRE63]

现在，为了按照我们的`IECommerceService`接口实现方法，我们将使用以下步骤来处理Get API：

![图11.4 – API的Get调用

](img/Figure_11.4_B18507.jpg)

图11.4 – API的Get调用

根据前面图中的步骤，`GetProductsAsync`的实现，主要用于检索用于着陆页的产品并在进行产品搜索时应用任何过滤器，将如下所示：

[PRE64]

[PRE65]

[PRE66]

[PRE67]

[PRE68]

[PRE69]

[PRE70]

[PRE71]

[PRE72]

[PRE73]

[PRE74]

[PRE75]

[PRE76]

[PRE77]

[PRE78]

[PRE79]

[PRE80]

[PRE81]

[PRE82]

[PRE83]

[PRE84]

[PRE85]

对于`POST`/`PUT` API，我们将有类似的步骤，但略有修改，如下面的图所示：

![图11.5 – API调用后的情况

](img/Figure_11.5_B18507.jpg)

图11.5 – API的Post调用

基于此，`CreateOrUpdateOrder`的策略实现，主要用于创建购物车，其代码如下所示：

[PRE86]

[PRE87]

[PRE88]

[PRE89]

[PRE90]

[PRE91]

[PRE92]

[PRE93]

[PRE94]

[PRE95]

[PRE96]

[PRE97]

[PRE98]

[PRE99]

[PRE100]

[PRE101]

[PRE102]

[PRE103]

[PRE104]

同样，我们将使用前面提到的一种策略和相关的API端点来实现`GetProductByIdAsync`、`GetOrderByIdAsync`、`GetInvoiceByIdAsync`和`SubmitOrder`。

现在，让我们创建将与`EcommerceService`通信并加载相关视图的控制器和操作方法。

# 创建控制器和操作

我们已经看到，路由负责将请求URI映射到控制器中的操作方法，因此让我们进一步了解操作方法是如何加载相应视图的。正如您所注意到的，ASP.NET Core MVC项目中的所有视图都是`Views`文件夹的一部分，当操作方法执行完成后，它只需简单地查找`Views/<ControllerName>/<Action>.cshtml`。

例如，将操作方法映射到`Products/Index`路由将加载`Views/Products/Index.cshtml`视图。这是通过在每个操作方法结束时调用`Microsoft.AspNetCore.Mvc.Controller.View`方法来处理的。

除了可以覆盖此行为并按需路由到不同视图的额外重载和辅助方法外，我们将在讨论这些辅助方法之前，就像Web API一样，MVC控制器中的每个操作方法也可以返回`IActionResult`，这意味着我们可以利用辅助方法来重定向到视图。在ASP.NET Core MVC中，每个控制器都继承自基类`Microsoft.AspNetCore.Mvc.Controller`，该类包含一些辅助方法，并通过以下`Microsoft.AspNetCore.Mvc.Controller`类中的辅助方法来处理通过操作方法加载视图：

+   `View`：此方法具有多种重载，主要根据控制器名称从`Views`文件夹下的文件夹加载视图。例如，在`ProductsController`中调用此方法可以加载`Views/Products`文件夹中的任何`.cshtml`文件。此外，它还可以接受视图名称，如果需要，可以加载，并支持通过强类型化视图来传递可以由视图检索的对象。

+   `RedirectToAction`：尽管`View`方法处理了大多数场景，但仍然会有需要在同一控制器或另一个控制器中调用另一个操作方法的场景，这就是`RedirectToAction`发挥作用的地方。此方法具有多种重载，允许我们指定操作方法、控制器方法以及操作方法可以接收作为路由值的对象。

简而言之，为了加载视图并从控制器传递数据，我们将通过`View`方法传递相应的模型，并在需要时使用`RedirectToAction`来调用另一个操作方法。

现在，问题是如何处理数据检索（`GET`调用）与数据提交（`POST`调用），在ASP.NET Core MVC中，所有动作方法都支持使用`HttpGet`和`HttpPost`属性来标记HTTP动词。以下是一些可以用来标记方法的规则：

+   如果我们想要检索数据，则动作方法应使用`HttpGet`进行标记。

+   如果我们想要向动作方法提交数据，则应使用`HttpPost`进行标记，并将相关对象作为动作方法的输入参数。

通常，需要从控制器向视图发送数据的函数应使用`[HttpGet]`进行标记，而需要从视图接收数据以进一步提交到数据库的函数应使用`[HttpPost]`进行标记。

现在，让我们继续添加所需的控制器并实现它们。当我们添加`Packt.Ecommerce.Web`时，它将创建一个包含默认创建的`HomeController`的`Controllers`文件夹，我们需要将其删除。然后，我们需要通过右键单击`Controllers`文件夹，然后选择`ProductsController`、`CartController`和`OrdersController`来添加三个控制器。

所有这些控制器都将具有以下两个共同属性，一个用于日志记录，另一个用于调用`EcommerceService`的方法。它们将通过构造函数注入进一步初始化，如下所示：

[PRE105]

[PRE106]

让我们现在讨论每个控制器中定义的内容：

+   `ProductsController`：此控制器将包含一个`public async Task<IActionResult> Index(string searchString, string category)`动作方法，用于加载列出所有产品的默认视图，并进一步支持过滤。将会有另一个方法，`public async Task<IActionResult> Details(string productId, string productName)`，它接受产品的ID和名称，并加载指定产品的详细信息。由于这两个方法都用于检索，因此它们将使用`[HttpGet]`进行标记。此外，此控制器将具有之前讨论过的`Error`方法。由于它可以从`UseStatusCodePagesWithReExecute`中间件接收错误代码作为输入参数，因此我们将有简单的逻辑来相应地加载视图：

    [PRE107]

+   `CartController`：此控制器包含一个`public async Task<IActionResult> Index(ProductListViewModel product)`动作方法，用于将产品添加到购物车中。我们将创建一个订单，并将订单状态设置为`'Cart'`，因为这需要接收数据并将其进一步传递到API，该API将被标记为`[HttpPost]`。为了简化，这里将其留为匿名，但可以限制对已登录用户的访问。一旦创建订单，此方法将使用`RedirectToAction`辅助方法，并将重定向到控制器内的`public async Task<IActionResult> Index(string orderId)`动作方法，该方法进一步加载包含所有产品的购物车和结账表单。此方法也可以直接导航到购物车。

+   `OrdersController`: 这是流程中的最后一个控制器，其中包含一个`public async Task<IActionResult> Create(OrderDetailsViewModel order)`动作方法，用于在填写支付详情后提交订单。此方法将订单状态更新为`Submitted`，然后为订单创建发票，最后重定向到另一个动作方法`public async Task<IActionResult> Index(string invoiceId)`，该方法加载订单的最终发票并完成交易。

下面的图表示了控制器之间完成购物流程的方法之间的流程：

![图11.6 – 控制器动作方法之间的流程

](img/Figure_11.6_B18507.jpg)

图11.6 – 控制器动作方法之间的流程

基于这些知识，让我们在下一节中设计视图。

# 使用ASP.NET Core MVC创建UI

到目前为止，我们已经定义了一个服务来与后端API通信，并进一步定义了将使用模型将数据传递给视图的控制器。现在，让我们构建各种视图，这些视图将渲染数据并向用户展示。

首先，让我们看看参与渲染视图的各种组件：

+   `Views`文件夹：所有视图都是这个文件夹的一部分，每个控制器特定的视图都由一个子文件夹分隔，最后，每个动作方法都由一个`.cshtml`文件表示。

要添加一个视图，我们可以在动作方法上右键单击并选择**添加视图**，这将自动创建一个（如果尚未存在）以控制器命名的文件夹，并添加视图。此外，在执行此操作时，我们可以指定视图将绑定到的模型。

+   `Layout`页面：在Web应用程序中，我们通常有一个跨应用共用的部分，例如带有菜单或左侧导航的页眉。为了使我们的页面具有模块化结构并避免重复，ASP.Net Core MVC提供了一个名为`_Layout.cshtml`的布局页面，它是`Views/Shared`文件夹的一部分。这个页面可以用作我们MVC项目中所有视图的父页面。一个典型的布局页面如下所示：

    [PRE108]

在这里，你可以看到它允许我们定义应用程序的骨架布局，然后最后，有一个名为`@RenderBody()`的Razor方法，它实际上加载了子视图。要指定任何视图中的布局页面，我们可以使用以下语法，它将`_Layout.cshtml`添加为视图的父页面：

[PRE109]

然而，没有必要在所有视图中重复此代码，这正是`_ViewStart.cshtml`发挥作用的地方。让我们看看它是如何帮助我们在视图之间重用一些代码的：

+   `_ViewStart.cshtml`：这是一个通用的视图，位于 `Views` 文件夹下，并由 Razor 引擎执行需要在视图代码之前执行的任何代码。因此，通常情况下，它用于定义布局页面，所以前面的代码可以添加到这个文件中，以便它在整个应用程序中应用。

+   `_ViewImports.cshtml`：这是另一个可以用于在应用程序中导入任何公共指令或命名空间的页面。就像 `_ViewStart` 一样，它也位于根文件夹下；然而，`_ViewStart` 和 `_ViewImport` 可以在一个（或多个）文件夹中，并且它们是按照从根视图文件夹中的开始，到任何子文件夹中的低级别文件夹的顺序执行的。要启用使用 Application Insights 的客户端遥测，我们按照以下代码注入 `JavaScriptSnippet`。我们在 [*第 5 章*](B18507_05_Epub.xhtml#_idTextAnchor445) 中学习了如何将依赖服务注入到视图中，*在 .NET 6 中的依赖注入*。在以下代码中，`JavaScriptSnippet` 被注入到视图中：

    [PRE110]

+   `wwwroot`：这是应用程序的根文件夹，所有静态资源，如 JavaScript、CSS 以及任何图像文件，都放置在这里。这里还可以存放我们希望在应用程序中使用的任何 HTML 插件。由于我们已经在应用程序中配置了 `UseStaticFiles` 中间件，因此文件夹中的内容可以直接提供服务，无需任何处理。ASP.NET Core MVC 的默认模板根据类型对文件夹进行了隔离；例如，所有 JavaScript 文件都放置在 `js` 文件夹中，CSS 文件放置在 `css` 文件夹中，等等。我们将坚持使用这种文件夹结构来构建我们的应用程序。

    注意

    通过右键单击操作方法并使用内置模板自动生成视图的过程称为 **scaffolding**，如果您对 Razor 语法不熟悉，可以使用它。然而，使用 scaffolding 或手动将其放置在相应的文件夹中并强类型化，会产生相同的行为。

## 设置 AdminLTE、布局页面和视图

要在整个应用程序中保持相同的视觉效果和感觉，一个重要的事情是选择正确的样式框架。这样做不仅提供了统一的布局，还简化了响应式设计，有助于在不同分辨率下正确渲染页面。我们为 `Packt.Ecommerce.Web` 使用的 ASP.NET Core MVC 项目模板默认包含 Bootstrap 作为其样式框架。我们将进一步扩展到名为 `AdminLTE` 的主题，它包含一些有趣的布局和仪表板，可以插入到我们的表示层中。

让我们执行以下步骤以将 `AdminLTE` 集成到我们的应用程序中：

1.  从这里下载 `AdminLTE` 的最新版本：[https://github.com/ColorlibHQ/AdminLTE/releases](https://github.com/ColorlibHQ/AdminLTE/releases)。

1.  提取之前下载的ZIP文件，并导航到`AdminLTE-3.0.5\dist\css`。复制`adminlte.min.css`，并将其粘贴到`Packt.Ecommerce.Web`的`wwwroot/css`文件夹内。

1.  导航到`AdminLTE-3.0.5\dist\js`。复制`adminlte.min.js`，并将其粘贴到`Packt.Ecommerce.Web`的`wwwroot/js`文件夹内。

1.  导航到`AdminLTE-3.0.5\dist\img`。复制所需的图片，并将它们粘贴到`Packt.Ecommerce.Web`的`wwwroot/img`文件夹内。

1.  复制`AdminLTE-3.0.5\plugins`文件夹，并将其粘贴到`Packt.Ecommerce.Web`的`wwwroot`文件夹内。

更多关于`AdminLTE`的信息可以在[https://adminlte.io/docs/2.4/installation](https://adminlte.io/docs/2.4/installation)找到。

现在，导航到`Views/_Layout.cshtml`页面，删除所有现有代码，并用`Packt.Ecommerce.Web\Views\Shared_Layout.cshtml`中的代码替换它。从高层次上讲，布局分为以下几部分：

+   在左侧页眉中的导航到主页

+   在页眉中添加一个搜索框，并在中间添加一个带有搜索类别的下拉菜单

+   在右侧页眉中的购物车

+   一个用于显示导航的面包屑路径

+   一个用于使用`@RenderBody()`渲染子视图的部分

完成集成`AdminLTE`模板所需的一些其他关键事项如下：

+   在`<head>`标签中添加以下样式定义：

    [PRE111]

+   在`<body>`标签的末尾之前添加以下JavaScript文件：

    [PRE112]

通过这种方式，我们已经将`AdminLTE`主题集成到我们的应用程序中。要使用Application Insights启用客户端遥测所需的JavaScript，请在`_Layout.cshtml`的`head`标签内添加以下代码：

[PRE113]

之前的代码注入了发送遥测数据所需的JavaScript，以及仪表化密钥。与服务器端或客户端不同，仪表化密钥是公开的。任何人都可以从浏览器开发者工具中看到仪表化密钥。但是，这就是客户端遥测的设置方式。此时，这种风险是恶意用户或攻击者可以通过仪表化密钥的只写访问权限推送不希望的数据。如果您希望使客户端遥测更安全，您可以从您的服务中公开一个安全的REST API，并从那里记录遥测事件。您将在[*第14章*](B18507_14_Epub.xhtml#_idTextAnchor1674)中了解更多关于Application Insights功能的信息，*健康和诊断*。

现在，应用程序布局已准备就绪。让我们现在继续定义应用程序中的各种视图。

### 创建Products/Index视图

此视图将用于列出我们电子商务应用程序上所有可用的产品，并且使用`IEnumerable<Packt.Ecommerce.DTO.Models.ProductListViewModel>`模型进行强类型化。它使用`ProductsController`的`Index`操作方法检索数据。

在此视图中，我们将使用简单的 Razor `@foreach (var item in Model)` 循环，并为每个产品显示产品的图片、名称和价格。以下截图展示了此视图的一个示例：

![图11.7 – 产品视图

](img/Figure_11.7_B18507.jpg)

图11.7 – 产品视图

在这里，您可以看到有一个搜索栏和一个来自布局页面的分类下拉菜单。点击产品图片将导航到 `Products/Details` 视图。为了支持此导航，我们将使用 `AnchorTagHelper` 并将产品ID和名称传递给 `ProductsController` 的 `Details` 动作方法，以便进一步在 `Products/Details` 视图中加载产品的详细信息。

### 创建 Products/Details 视图

此视图将根据从 `Products/Index` 视图传递的产品ID和名称加载产品的详细信息。我们将使用以下示例页面：[https://adminlte.io/themes/dev/AdminLTE/pages/examples/e_commerce.html](https://adminlte.io/themes/dev/AdminLTE/pages/examples/e_commerce.html)。

此页面将使用 `Packt.Ecommerce.DTO.Models.ProductDetailsViewModel` 进行强类型定义，并显示产品的所有详细信息。以下截图展示了此页面的一个示例：

![图11.8 – 产品详情视图

](img/Figure_11.8_B18507.jpg)

图11.8 – 产品详情视图

如您所见，有一个 `'Cart'`，这将调用 `CartController` 的 `Index` 动作方法以创建购物车。

要将数据传递回动作方法，我们将借助 `FormTagHelper`，它允许我们将页面包裹在一个HTML表单中，并使用以下代码指定页面可以提交到的动作和控制器：

[PRE114]

使用此代码，一旦提交为 `Submit` 类型，页面就会提交到 `CartController` 的 `Index` 动作方法，以便进一步将其保存到数据库。然而，我们仍然需要将产品详情传递回 `Index` 动作方法，为此，我们将借助 `InputTagHelper` 并为所有需要传递回动作方法的值创建隐藏字段。

这里最重要的是隐藏变量的名称应与模型中属性的名称匹配，因此我们将在表单内添加以下代码，以将产品值传递回控制器：

[PRE115]

[PRE116]

[PRE117]

[PRE118]

ASP.NET Core MVC 的模型绑定系统读取这些值，并为 `CartController` 的 `Index` 方法创建所需的产品对象，然后进一步调用后端系统以创建订单。

### 创建购物车/索引视图

此视图将加载购物车详情，并将有一个填写所有详细信息并完成订单的结账表单。在此，我们可以导航回主页以添加更多产品或完成订单。

此视图使用 `Packt.Ecommerce.DTO.Models.OrderDetailsViewModel` 进行强类型化，并使用 `OrdersController` 的 `Index` 动作方法加载数据。在这里，我们使用了来自 [https://getbootstrap.com/docs/4.5/examples/checkout/](https://getbootstrap.com/docs/4.5/examples/checkout/) 的 Bootstrap 结账表单示例。

此表单利用模型验证和 HTML 属性对必填字段进行验证，我们借助 ASP.NET Core MVC 标签辅助器和一些 HTML 辅助器来渲染表单。以下代码展示了具有模型验证的示例属性：

[PRE119]

[PRE120]

[PRE121]

[PRE122]

[PRE123]

[PRE124]

[PRE125]

[PRE126]

[PRE127]

此模型用于 [https://github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition/blob/main/Enterprise%20Application/src/platform-apis/data/Packt.Ecommerce.DTO.Models/OrderDetailsViewModel.cs](https://github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition/blob/main/Enterprise%20Application/src/platform-apis/data/Packt.Ecommerce.DTO.Models/OrderDetailsViewModel.cs) 并在下单时触发必要的验证。

由于此表单也需要提交，整个表单被包裹在 `FormTagHelper` 中，如下面的代码所示：

[PRE128]

要在 UI 上显示这些验证，请将以下脚本添加到 `_layout.cshtml` 中，在添加了所有其他脚本之后：

[PRE129]

[PRE130]

要显示错误消息，我们可以使用验证消息标签辅助器，如下面的代码片段所示。在服务器端，这可以通过 `ModelState.IsValid` 进一步评估：

[PRE131]

[PRE132]

此页面的示例截图如下：

![图 11.9 – 购物车和结账页面

](img/Figure_11.9_B18507.jpg)

图 11.9 – 购物车和结账页面

我们将使用 `InputTagHelper` 作为隐藏字段和文本框，以将任何附加信息传递回动作方法。文本框的好处是，如果文本框的 `id` 属性与属性名称匹配，那么数据将自动传递回动作方法，ASP.NET Core MVC 的模型绑定系统将负责将其映射到所需的对象，在这种情况下，是 `Packt.Ecommerce.DTO.Models.OrderDetailsViewModel` 类型的对象，最终提交订单、生成发票并重定向到 `Orders/Index` 动作方法。

注意

在前面的屏幕截图中，尽管在生产应用中我们有一个包含支付信息的结账表单，但我们将与第三方支付网关集成，通常，整个表单都位于应用程序的支付网关侧，出于各种安全原因。[https://razorpay.com/docs/payment-gateway/server-integration/dot-net/](https://razorpay.com/docs/payment-gateway/server-integration/dot-net/) 和 [https://stripe.com/docs/api](https://stripe.com/docs/api) 是这类第三方提供商的几个例子，它们有助于支付网关集成。

### 创建 Orders/Index 视图

最后，我们将有一个查看订单发票的视图，这是一个简单的只读视图，显示从 `OrdersController` 的 `Index` 动作方法发送的发票信息。以下截图显示了此页面的一个示例：

![图 11.10 – 最终发票

](img/Figure_11.10_B18507.jpg)

图 11.10 – 最终发票

这完成了各种视图的集成，如您所见，我们已经将视图限制在电子商务应用程序中最重要的流程中。然而，您可以使用相同的原理进一步添加更多功能。

# 理解 Blazor

Blazor 是从 .NET Core 3.1 开始可用的新框架，用于开发应用程序的前端层。它是 MVC 和 Razor Pages 的替代方案之一，并且应用程序模型非常接近 SPA；然而，我们可以用 C# 和 Razor 语法编写逻辑，而不是 JavaScript。

所有的 Blazor 编写代码都放在一个名为 `.Razor` 的地方，用于表示应用程序；无论是整个网页还是一个小对话框弹出窗口，所有内容都在 Blazor 应用程序中作为一个组件创建。一个典型的 Razor 组件如下代码片段所示：

[PRE133]

[PRE134]

[PRE135]

[PRE136]

[PRE137]

[PRE138]

[PRE139]

[PRE140]

[PRE141]

在此代码中，我们创建了一个页面，该页面在按钮点击时增加计数器，点击事件的逻辑由 C# 代码处理，该代码更新 HTML 中的值。此页面可以通过 `/counter` 相对 URL 访问。

Blazor 与其他 MVC/Razor Pages 之间的主要区别在于，与请求-响应模型不同，其中每个请求都发送到服务器，并将 HTML 发送回浏览器，Blazor 将所有组件打包（就像 SPA 一样）并在客户端加载。当应用程序首次请求时，任何后续对服务器的调用都是为了检索/提交任何 API 数据或更新 DOM。Blazor 支持以下两种托管模型：

+   **Blazor WebAssembly**（**WASM**）：WASM 是可以在现代浏览器上运行的低级指令，这有助于在浏览器上运行用高级语言（如 C#）编写的代码，而无需任何附加插件。Blazor WASM 托管模型利用 WASM 提供的开放网络标准，在浏览器上的沙盒环境中运行任何 Blazor WASM 应用程序的 C# 代码。在较高层次上，所有 Blazor 组件都编译成 .NET 程序集，并下载到浏览器，WASM 加载 .NET Core 运行时和所有程序集。它进一步使用 JavaScript 互操作来刷新 DOM；唯一的服务器调用将是任何后端 API。架构如下所示：

![图 11.11 – Blazor WASM 托管

](img/Figure_11.11_B18507.jpg)

图 11.11 – Blazor WASM 托管

+   `blazor.server.js` 并使用 SignalR 接收所有 DOM 更新，这意味着每个用户交互都会有一个服务器调用（尽管非常轻量）。架构如下所示：

![图 11.12 – Blazor Server 托管

](img/Figure_11.12_B18507.jpg)

图 11.12 – Blazor Server 托管

.NET 6 为两种托管模型提供完整的工具支持，包括它们自己的项目模板，并且各有优缺点，这些将在以下内容中进一步解释：[https://docs.microsoft.com/en-us/aspnet/core/blazor/hosting-models?view=aspnetcore-6.0](https://docs.microsoft.com/en-us/aspnet/core/blazor/hosting-models?view=aspnetcore-6.0)。

现在让我们按照以下步骤使用 Blazor Server 应用程序创建一个前端应用程序，这允许我们为我们的电子商务应用程序添加/修改产品详情：

1.  将名为 `Packt.Ecommerce.Blazorweb` 的新 Blazor Server 应用程序添加到企业解决方案中，并将 `Products.razor`、`AddProduct.Razor` 和 `EditProduct.razor` Razor 组件添加到 `Pages` 文件夹。

1.  此项目包含 `Program` 类，它与其他任何 ASP.NET Core 应用程序完全相同，只是增加了一些额外的 Blazor 服务。`_Host.cshtml` 是应用程序的根，应用程序的初始调用由该页面接收，并以 HTML 进行响应。此页面进一步引用 `blazor.server.js` 脚本文件以建立 SignalR 连接。另一个重要组件是 `App.Razor` 组件，它负责基于 URL 的路由。在 Blazor 中，任何需要映射到特定 URL 的组件都将在其开头包含 `@page` 指令，该指令指定应用程序的相对 URL。`App.Razor` 拦截 URL 并将它们路由到指定的组件。所有 Razor 组件都是 `Pages` 文件夹的一部分，而 `Data` 文件夹包含用于 `FetchData.razor` 组件的示例模型和服务。

1.  让我们在 `NavMenu.razor` 中添加以下代码以将 `Products` 导航添加到左侧菜单。在这个阶段，如果你运行应用程序，你应该能够看到带有 `Products` 导航的左侧菜单；然而，它不会导航到任何页面：

    [PRE142]

1.  由于我们将从 API 获取数据，我们需要将 `HTTPClient` 注入到我们的 `Program` 类中，就像在 ASP.NET Core 应用程序中所做的那样。因此，将以下代码添加到 `Program` 类中：

    [PRE143]

1.  将以下 `ApplicationSettings:ProductsApiEndpoint` 设置添加到 `appsettings.json`：

    [PRE144]

1.  由于我们将要绑定 `products` 数据，所以让我们将 `Packt.Ecommerce.DTO.Models` 添加为 `Packt.Ecommerce.Blazorweb` 项目的引用。在 `Pages` 文件夹中，将以下代码添加到 `Products.razor` 页面的 `@code` 块内，我们在其中使用 `IHttpClientFactory` 创建一个 `HttpClient` 对象，该对象将在下一步注入，并在 `OnInitializedAsync` 方法中检索 `products` 数据：

    [PRE145]

1.  接下来，在`Products.Razor`页面的开头添加以下代码（在`@code`块外部）。在这里，我们通过`@page`指令设置此组件的相对路由为`/products`。接下来，我们注入`IHttpClientFactory`和其他所需的命名空间，然后添加渲染产品列表的HTML部分。如您所见，它是HTML和Razor语法的混合体：

    [PRE146]

在这一点上，如果您运行应用程序，应该会看到以下截图所示的输出：

![图11.13 – 产品列表Blazor UI

](img/Figure_11.13_B18507.jpg)

图11.13 – 产品列表Blazor UI

1.  接下来，让我们创建`Add/Edit`页面，我们将使用Blazor表单。表单中可用的一些重要工具/组件如下。

Blazor表单是使用Blazor中称为`EditForm`的内置模板创建的，并且可以直接使用模型属性绑定到任何C#对象。一个典型的`EditForm`如下面的代码片段所示。在这里，我们定义在表单提交时调用`OnSubmit`方法。让我们将其添加到`AddProduct.razor`中：

[PRE147]

在这里，`product`是我们想要使用的模型对象，在我们的案例中是`Packt.Ecommerce.DTO.Models.ProductDetailsViewModel`。要将数据绑定到任何控件，我们可以使用HTML和Razor语法的组合，如下面的代码所示。在这里，我们将产品对象的`Name`属性绑定到文本框，同样，将`Category`属性绑定到下拉列表。一旦你在文本框中输入任何值或在下拉列表中选择一个值，它就会自动出现在这些属性中，以便将其传递回任何后端API或数据库。让我们以类似的方式将所有必需的属性添加到HTML元素中：

[PRE148]

Blazor表单支持使用数据注解进行数据验证，因此任何我们想要绑定到UI的模型都可以有数据注解，Blazor会自动将这些验证应用到绑定的控件上。要应用验证，我们添加`DataAnnotationsValidator`组件，并可以使用`ValidationSummary`组件来显示所有验证失败的摘要。更多详细信息请参阅[https://docs.microsoft.com/en-us/aspnet/core/blazor/forms-validation?view=aspnetcore-6.0](https://docs.microsoft.com/en-us/aspnet/core/blazor/forms-validation?view=aspnetcore-6.0)。我们还可以在控件级别使用`ValidationMessage`组件，如下面的代码片段所示：

[PRE149]

1.  在`code`组件中，添加一个`ProductDetailsViewModel`对象，并将其命名为product，即如`EditForm`的`Model`属性中定义的那样，并进一步实现`OnSubmit`方法。

`AddProduct.Razor`和`EditProduct.Razor`的完整代码可以在GitHub仓库中找到，一旦运行应用程序，我们就可以看到以下页面：

![图11.14 – 添加产品Blazor UI

](img/Figure_11.14_B18507.jpg)

图11.14 – 添加产品Blazor UI

这是一个使用 Blazor 构建前端的基本示例，它执行列表、创建和更新操作。然而，Blazor 中还有许多概念可以进一步探索，请参阅[https://docs.microsoft.com/en-us/aspnet/core/blazor/?view=aspnetcore-6.0](https://docs.microsoft.com/en-us/aspnet/core/blazor/?view=aspnetcore-6.0)。

# 摘要

在本章中，我们了解了表示层和 UI 设计的各个方面。此外，我们还学习了使用 ASP.NET Core MVC 和 Razor Pages 开发表示层的各种技能，最后，我们使用 ASP.NET Core MVC 和 Blazor 为我们的企业应用程序实现了表示层。

使用这些技能，你应该能够使用 ASP.NET Core MVC、Razor Pages 和 Blazor 构建表示层，并将其与后端 API 集成。

在下一章中，我们将看到如何将身份验证集成到我们系统的各个应用层中。

# 问题

1.  以下哪个是推荐用于定义在整个 Web 应用程序中需要出现的左侧导航的页面？

a. `_ViewStart.cshtml`

b. `_ViewImports.cshtml`

c. `_Layout.cshtml`

d. `Error.cshtml`

**答案：c**

1.  以下哪个页面可以用来配置整个应用程序的 `Layout` 页面？

a. `_ViewStart.cshtml`

b. `_ViewImports.cshtml`

c. `_Layout.cshtml`

d. `Error.cshtml`

**答案：a**

1.  以下哪个特殊字符用于在 `.cshtml` 页面中编写 Razor 语法？

a. `@`

b. `#`

c. `<% %>`

d. 以上皆非

**答案：a**

1.  在以下 Razor 页面应用程序的标签助手代码中，哪个方法会在按钮点击时被调用？

    [PRE150]

a. `OnGet()`

b. `onDelete()`

c. `OnPostDelete()`

d. `OnDeleteAsync()`

**答案：c**

# 进一步阅读

+   [https://www.packtpub.com/web-development/html5-and-css3-building-responsive-websites](https://www.packtpub.com/web-development/html5-and-css3-building-responsive-websites)

+   [https://www.packtpub.com/product/bootstrap-for-asp-net-mvc-second-edition/9781785889479](https://www.packtpub.com/product/bootstrap-for-asp-net-mvc-second-edition/9781785889479)

+   [https://developer.mozilla.org/en-US/docs/WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly)

+   [https://docs.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-6.0](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-6.0)

+   [https://docs.microsoft.com/en-us/aspnet/core/mvc/views/partial?view=aspnetcore-6.0](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/partial?view=aspnetcore-6.0)

+   [https://developer.mozilla.org/en-US/docs/Learn/Accessibility](https://developer.mozilla.org/en-US/docs/Learn/Accessibility)
