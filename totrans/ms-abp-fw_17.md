# *第13章*：使用Blazor WebAssembly UI

Blazor是一个相对较新的使用C#而不是JavaScript构建交互式网页应用的**单页应用程序（SPA**）框架。Blazor是ABP框架提供的内置UI选项之一。

在本章中，我将简要讨论Blazor是什么以及使用这个新框架的主要优缺点。然后，我将继续解释如何使用Blazor UI选项创建新的ABP解决方案。到本章结束时，你将了解ABP Blazor集成的架构和设计，并了解你将在应用程序中使用的基本ABP服务。

本章包括以下主题：

+   什么是Blazor？

+   开始使用ABP Blazor UI

+   验证用户身份

+   理解主题系统

+   使用菜单

+   使用基本服务

+   使用UI服务

+   消费HTTP API

+   使用全局脚本和样式

# 技术要求

如果你想跟随本章中的示例，你需要一个支持ASP.NET Core开发的IDE/编辑器。在某些地方，我们将使用ABP CLI，因此你需要安装ABP CLI，具体请参考[*第2章*](B17287_02_Epub_AM.xhtml#_idTextAnchor026)，“ABP框架入门”。

您可以从以下GitHub仓库下载示例应用程序：[https://github.com/PacktPublishing/Mastering-ABP-Framework](https://github.com/PacktPublishing/Mastering-ABP-Framework)。它包含本章中给出的一些示例。

# 什么是Blazor？

如我在引言中指出的，Blazor是一个用于构建交互式网页应用的SPA框架，就像其他SPA框架（如Angular、React和Vue.js）一样。然而，它有一个重要的区别——我们可以使用C#来构建应用而不是JavaScript，这意味着我们可以在浏览器中运行.NET。Blazor使用.NET核心运行时在浏览器中执行.NET代码（对于Blazor WebAssembly）。

在浏览器中运行.NET并不是一个新想法。微软之前已经通过Silverlight做到了这一点。要运行Silverlight应用程序，我们不得不在浏览器上安装一个插件。另一方面，Blazor通过**WebAssembly**技术原生地在浏览器上运行，该技术定义如下[https://webassembly.org](https://webassembly.org)：

"WebAssembly（简称Wasm）是一种基于栈的虚拟机的二进制指令格式。Wasm被设计为编程语言的便携式编译目标，使得客户端和服务器应用程序能够在网络上部署。"

一种高级语言，如C#，可以被编译成WebAssembly并在浏览器中本地运行。WebAssembly被所有主流的网页浏览器支持，因此我们不需要安装任何自定义插件。如果你想知道Blazor是否是新的Silverlight，我可以简单地说，不是的。

作为.NET开发者，Blazor为我们带来了难以置信的机会：

+   我们可以利用语言和运行时的全部功能，使用我们现有的C#技能来开发应用程序。

+   我们可以使用现有的.NET库，例如我们最喜欢的NuGet包。

+   我们可以在服务器和客户端之间共享代码（例如DTO类、应用程序服务合约、本地化和验证代码）。

+   我们可以使用熟悉的Razor语法来构建UI页面和组件。

除了使用C#，Blazor还提供了JavaScript互操作性，可以从C#调用JavaScript代码，反之亦然。这意味着我们可以在需要时使用现有的JavaScript库并编写我们的JavaScript代码。

在服务器和客户端应用程序之间编写C#和共享代码对于.NET开发者来说是一个巨大的优势。ABP也利用了这一点，尽可能地在MVC/Razor Pages UI和Blazor UI之间共享基础设施。您会发现许多服务与MVC/Razor Pages UI非常相似。

作为.NET开发者和软件公司经理，我对Blazor印象深刻，并将将其用于未来的项目。然而，这并不意味着它没有缺点：

+   打包大小、初始加载时间和运行时性能比其JavaScript竞争对手，如Angular和React，更差。然而，微软正在投资Blazor并努力提高其性能。例如，.NET 6.0引入了**即时编译**（**AOT**）。

+   由于Blazor还处于早期阶段，UI组件和生态系统尚未成熟。

+   调试目前还不是那么直接。

如果这些缺点对您的项目可以容忍，您今天就可以开始使用Blazor。

有趣的是，Blazor有两种运行时模型。到目前为止，我主要谈论的是**Blazor WebAssembly**。第二种模型称为**Blazor Server**。虽然组件开发模型是相同的，但托管逻辑和运行时模型完全不同。

使用Blazor WebAssembly，.NET代码在浏览器上的Mono运行时中运行，我们不必在服务器端运行.NET。一小段初始化JavaScript代码下载标准的.NET **动态链接库**（**DLL**）并在浏览器中运行。这种模式类似于，并且是Angular和React的直接竞争对手，因为它在浏览器中完全运行客户端逻辑。

另一方面，Blazor Server在服务器上完全运行.NET代码。它建立了客户端和服务器之间的实时SignalR连接。浏览器运行JavaScript并通过该SignalR连接与服务器通信。它向服务器发送事件，服务器执行必要的.NET代码并将**文档对象模型**（**DOM**）更改发送到浏览器。最后，浏览器将DOM更改应用到UI上。

与Blazor WebAssembly相比，Blazor Server模型具有更快的初始加载时间。然而，它需要与服务器通信以处理所有事件和DOM更改，因此我们需要服务器和客户端之间有一个良好且稳定的连接。

我在这本书中的目的不是提供Blazor的完整介绍、概述和使用案例，而是提供一个足够简短的介绍，以便理解它是什么。此外，本章将重点介绍Blazor WebAssembly，但大多数主题都适用于Blazor Server。

现在，我们可以开始ABP的Blazor集成。

# 开始使用ABP Blazor UI

使用ABP的启动解决方案模板启动新项目有两种方式。您可以从[https://abp.io/get-started](https://abp.io/get-started)下载它，或者使用ABP CLI创建它。本书中我将使用CLI方法。如果您尚未安装它，请打开命令行终端并执行以下命令：

[PRE0]

现在，我们可以使用`abp new`命令创建一个新的解决方案：

[PRE1]

`DemoApp`是本例中的解决方案名称。我已通过传递`-u blazor`参数来指定Blazor WebAssembly。如果您想使用Blazor Server，可以将参数指定为`-u blazor-server`。

我尚未指定数据库提供程序，因此默认使用Entity Framework Core（如果想要使用MongoDB，请指定`-d mongodb`参数）。在创建解决方案后，我们需要创建初始数据库迁移。作为第一步，我们应该在`src/DemoApp.DbMigrator`目录中执行以下命令：

[PRE2]

此命令创建初始的代码优先迁移并将其应用于数据库。

该解决方案包含两个应用程序：

+   第一个是一个服务器（后端）应用程序，它托管HTTP API并提供身份验证UI。

+   第二个应用程序是包含应用程序UI并与服务器通信的前端Blazor WebAssembly应用程序。

因此，我们首先运行此示例的`DemoApp.HttpApi.Host`服务器应用程序。然后，我们可以运行`DemoApp.Blazor` Blazor应用程序来运行UI。您可以使用`admin`作为用户名和`1q2w3E*`作为密码登录到应用程序。

我不会深入探讨应用程序的细节，因为我们已经在[*第2章*](B17287_02_Epub_AM.xhtml#_idTextAnchor026)中做了介绍，即*使用ABP框架入门*。下一节将解释用户是如何进行身份验证的。

# 用户身份验证

**OpenID Connect**（**OIDC**）是Microsoft建议用于身份验证Blazor WebAssembly应用程序的方法。ABP遵循这一建议，并在启动解决方案中预先配置了它。

Blazor应用程序不包含登录、注册或其他与身份验证相关的UI页面。它使用启用了**证明密钥用于代码交换**（**PKCE**）的**授权代码**流将用户重定向到服务器应用程序。服务器处理所有身份验证逻辑并将用户重定向回Blazor应用程序。

身份验证配置存储在Blazor应用程序的`wwwroot/appsettings.json`文件中。请参阅以下示例配置：

[PRE3]

在这里，`Authority`是后端服务器应用程序的根URL。`ClientId`是服务器所知的Blazor应用程序的名称。最后，`ResponseType`指定了授权代码流。

此配置在模块类中使用，例如本例中的`DemoAppBlazorModule`类，如下面的代码块所示：

[PRE4]

`AuthServer`是匹配配置密钥的关键。如果您想自定义身份验证选项，这些就是您需要从这些点开始的地方。例如，您可以修改请求的作用域或更改OIDC配置。有关Blazor WebAssembly身份验证的更多信息，请参阅Microsoft的文档：[https://docs.microsoft.com/en-us/aspnet/core/blazor/security/webassembly/](https://docs.microsoft.com/en-us/aspnet/core/blazor/security/webassembly/)。

在下一节中，我将介绍Blazor UI的主题系统。

# 理解主题系统

ABP为Blazor UI提供了一套主题系统，正如我们在介绍MVC/Razor Pages UI时所述[*第12章*](B17287_12_Epub_AM.xhtml#_idTextAnchor356)，*使用MVC/Razor Pages*。主题系统带来了灵活性，因此我们可以开发我们的应用程序和模块，而无需依赖于特定的UI主题/样式。

所有ABP Blazor UI主题都使用一组基础库。基本基础库是Bootstrap，其组件旨在与JavaScript一起使用。幸运的是，一些组件库封装了Bootstrap组件，并提供了一个更简单的.NET API，这更适合在Blazor应用程序中使用。

这些组件库之一是**Blazorise**。实际上，它是一个抽象库，可以与多个提供者（如Bootstrap、Bulma和Ant Design）一起工作。ABP启动模板使用Blazorise库的Bootstrap提供者。

您可以在其网站上了解更多关于Blazorise的信息，并查看组件的实际应用：[https://blazorise.com](https://blazorise.com)。以下图是表单组件演示的截图：

![图13.1 – Blazorise演示：表单组件

](img/Figure_13.01_B17287.jpg)

图13.1 – Blazorise演示：表单组件

除了Blazorise库之外，ABP Blazor UI还使用**Font Awesome**作为CSS字体图标库。因此，任何模块或应用程序都可以在其页面上使用这些库，而无需显式依赖。

UI主题负责渲染布局，包括页眉、菜单、工具栏、页面警报和页脚。

在下一节中，我们将看到如何向主菜单添加新项目。

# 使用菜单

ABP Blazor UI中的菜单管理与ABP MVC/Razor Pages UI中的菜单管理非常相似，这在[*第12章*](B17287_12_Epub_AM.xhtml#_idTextAnchor356)中有所介绍，即*使用MVC/Razor Pages*。

我们使用`AbpNavigationOptions`向菜单系统添加贡献者。ABP执行所有贡献者以动态构建菜单。启动解决方案包括一个菜单贡献者，并按照以下示例添加到`AbpNavigationOptions`中：

[PRE5]

`DemoAppMenuContributor`是一个实现`IMenuContributor`接口的类。`IMenuContributor`接口定义了`ConfigureMenuAsync`方法，我们应该按照以下示例实现：

[PRE6]

在`StandardMenus`类（在`Volo.Abp.UI.Navigation`命名空间中）中定义了两个标准菜单名称作为常量：

+   `Main`：应用程序的主菜单。

+   `User`：用户上下文菜单。当您在页眉上点击您的用户名时，它会打开。

因此，前面的示例检查菜单名称，并且只向主菜单添加项目。以下代码块向主菜单添加一个新菜单项：

[PRE7]

您可以使用`context.ServiceProvider`对象从依赖注入中解析服务。`context.GetLocalizer`方法是一个用于解析`IStringLocalizer<T>`实例的快捷方式。同样，我们可以使用`context.IsGrantedAsync`快捷方法来检查当前用户的权限，如以下代码块所示：

[PRE8]

菜单项可以嵌套。以下示例在`Crm`菜单项下添加了一个`Orders`菜单项：

[PRE9]

我在第一个`ApplicationMenuItem`对象上调用`AddItem`以添加子项。您可以为`Orders`菜单项做同样的事情以构建更深的菜单。

我们在创建菜单项时使用了本地化和授权服务。在下一节中，我们将看到如何在Blazor应用程序的其他部分中使用这些服务。

# 使用基本服务

在本节中，我将向您展示如何在Blazor应用程序中使用一些基本服务。正如您将看到的，它们几乎与我们在早期章节中介绍的服务器端服务相同。让我们从授权服务开始。

## 授权用户

我们通常在Blazor应用程序中使用授权来隐藏/禁用一些页面、组件和功能。虽然服务器始终检查相同的授权规则以确保安全，但客户端授权检查提供了更好的用户体验。

`IAuthorizationService`用于以编程方式检查权限/策略，就像在服务器端一样。您可以根据以下示例注入和使用其方法：

[PRE10]

`AuthorizationService`有不同的工作方式。请参阅[*第7章*](B17287_07_Epub_AM.xhtml#_idTextAnchor213)中关于*与授权和权限系统一起工作*的部分，以了解更多关于授权系统的信息。

在前面的示例中，该组件是从`AbpComponentBase`类继承的。由于`AbpComponentBase`类为我们预先注入了它，我们可以直接使用`AuthorizationService`属性而无需手动注入。`AuthorizationService`属性类型是`IAuthorizationService`。

如果你没有从`AbpComponentBase`类继承，你可以使用`[Inject]`属性来注入它：

[PRE11]

当你需要时，你可以在Razor组件的视图侧使用相同的`IAuthorizationService`。然而，有一些替代方法可以使你的应用程序代码更简洁。例如，你可以在组件上使用`[Authorize]`属性，使其仅对认证用户可用：

[PRE12]

`[Authorize]`属性与服务器端类似。你可以在以下示例中传递策略/权限名称来检查特定权限：

[PRE13]

如果用户具有特定的权限，通常会显示UI的一部分。以下示例使用`AuthorizeView`元素，如果当前用户有权限编辑订单，则显示消息：

[PRE14]

这样，你可以有条件地渲染操作按钮或其他UI部分。

ABP与Blazor的授权系统100%兼容，因此你可以参考Microsoft的文档来查看更多示例和详细信息：[https://docs.microsoft.com/en-us/aspnet/core/blazor/security](https://docs.microsoft.com/en-us/aspnet/core/blazor/security)。

在下一节中，我们将学习如何使用本地化系统，这是另一个常见的UI服务。

## 本地化用户界面

Blazor应用程序使用相同的API进行文本本地化。我们可以注入并使用`IStringLocalizer<T>`服务来获取当前语言的本地化文本。

以下Razor组件使用了`IStringLocalizer<T>`服务：

[PRE15]

我们使用标准的`@inject`指令，并在泛型`IStringLocalizer<T>`接口中指定本地化资源类型。相同的接口也可以注入并用于应用程序中的任何服务。请参阅[*第8章*](B17287_08_Epub_AM.xhtml#_idTextAnchor249)的*本地化用户界面*部分，*使用ABP的功能和服务*，了解如何与本地化系统一起工作。

接下来，我们将在下一节学习如何获取当前用户的信息。

## 访问当前用户

你有时可能需要在应用程序中知道当前用户的用户名、电子邮件地址和其他详细信息。我们使用`ICurrentUser`服务来访问当前用户，就像在服务器端一样。以下示例组件通过当前用户的名称渲染欢迎信息：

[PRE16]

除了`Name`、`Surname`、`UserName`和`Email`等标准属性外，你还可以使用`ICurrentUser.FindClaimValue(...)`方法来获取服务器颁发的自定义声明。

我已经介绍了所有应用程序通常使用的ABP Blazor基本服务。我使它们保持简短，因为API几乎与服务器端相同，而且我们已经在之前的章节中详细介绍了它们。在下一节中，我将继续介绍用于通知用户的UI服务。

# 使用UI服务

在每个应用程序中，向用户显示消息、通知和警报以通知或警告他们是很常见的。在接下来的部分中，我将介绍ABP为这些服务内置的API。

## 显示消息框

消息框用于向用户显示阻塞消息或确认对话框。用户点击**确定**按钮来禁用消息，或者点击**是**或**取消**按钮来对配置对话框做出决定。

有五种类型的消息 – `信息`、`成功`、`警告`、`错误`和`确认`。以下示例显示了发送给用户的`成功`消息：

[PRE17]

在此示例中，`Message`属性来自`AbpComponentBase`类（`DemoAppComponentBase`继承自它），其类型为`IUiMessageService`。或者，您可以为您的组件、页面或服务手动注入`IUiMessageService`。所有`IUiMessageService`方法都可以接受额外的`title`参数和`options`操作来自定义对话框。

以下图显示了前面示例的结果：

![图13.2 – 一个没有标题的简单成功消息

](img/Figure_13.02_B17287.jpg)

图13.2 – 一个没有标题的简单成功消息

以下示例显示了发送给用户的确认对话框，如果用户点击**是**按钮，则采取行动：

[PRE18]

`Confirm`方法返回一个`bool`值，因此您可以看到用户是否接受了对话框消息。以下图显示了此示例的结果：

![图13.3 – 确认对话框

](img/Figure_13.03_B17287.jpg)

图13.3 – 确认对话框

下一个部分将解释如何向用户显示非阻塞的信息消息。

## 显示通知

消息框将用户的注意力集中在消息上。他们应该点击**确定**按钮返回到应用程序UI。另一方面，通知是非阻塞的信息消息。它们显示在屏幕的右下角，并在几秒钟后自动消失。

有四种类型的通知 – `信息`、`成功`、`警告`和`错误`。以下示例显示了一个确认对话框，如果用户接受确认消息，则显示`成功`通知：

[PRE19]

`Notify`属性来自`AbpComponentBase`基类。您可以将`IUiNotificationService`接口注入并用于任何地方来在UI上显示通知。所有通知方法都可以接受额外的`title`参数和`options`操作来自定义对话框。以下图显示了在前面代码块中使用的`Notify.Success`方法的结果：

![图13.4 – 一个示例通知消息

](img/Figure_13.04_B17287.jpg)

图13.4 – 一个示例通知消息

下一个部分介绍了警报，这是向用户显示消息的另一种方式。

## 显示警报

使用警报是一种粘性的方式来向用户显示非阻塞消息。用户可以选择忽略警报。

有四种类型的提示信息 – `Info`、`Success`、`Warning`和`Danger`。以下示例展示了发送给用户的`Success`提示信息：

[PRE20]

在这个例子中，我使用了来自基类的`Alerts`属性。你总是可以注入`IAlertManager`服务并像`IAlertManager.Alerts.Success(…)`一样使用它。

所有的提示方法都接受`text`（必需）、`title`（可选）和`dismissible`（可选，默认为`true`）参数。如果一个提示信息是可关闭的，那么用户可以通过点击**X**按钮使其消失。以下图显示了前面例子中创建的提示信息：

![图13.5 – 成功提示消息

](img/Figure_13.05_B17287.jpg)

图13.5 – 成功提示消息

提示信息是通过主题在页面内容上方渲染的。除了标准的`Info`、`Success`、`Warning`和`Danger`方法外，你可以通过指定`AlertType`来使用所有Bootstrap样式，如`Primary`、`Secondary`或`Dark`。

你现在已经学会了三种向用户展示信息消息的方法。在下一节中，我们将探讨Blazor应用程序如何消费服务器的HTTP API。

# 消费HTTP API

你可以使用标准的`HttpClient`手动设置和执行对服务器的HTTP请求。然而，ABP提供了C#客户端代理，可以轻松调用HTTP API端点。你可以直接从Blazor UI消费你的应用服务，并让ABP框架为你处理HTTP API调用。

假设我们有一个应用服务接口，如下例所示：

[PRE21]

应用服务接口定义在`Application.Contracts`项目中（对于我创建的示例解决方案，是`DemoApp.Application.Contracts`项目）。Blazor应用程序引用了这个项目。这样，我们就可以在客户端使用`ITestAppService`接口。

应用服务在`Application`项目中实现（对于我创建的示例解决方案，是`DemoApp.Application`项目）。我们可以简单地实现`ITestAppService`接口，如下面的代码块所示：

[PRE22]

现在，我们可以直接将`ITestAppService`注入到任何页面/组件中，就像注入任何其他本地服务一样，并调用其方法，就像标准的函数调用一样：

[PRE23]

在这个例子中，我在`TestAppService`属性上方使用了标准的`[Inject]`属性来告诉Blazor为我注入它。然后，我在`OnInitializedAsync`方法中重写了它来调用`GetDataAsync`方法。正如我们所知，`OnInitializedAsync`方法是在组件/页面最初渲染并准备好工作后立即被调用的。

简单到这种程度。当我们调用 `GetDataAsync` 方法时，ABP 实际上通过处理所有复杂性（包括身份验证、错误处理和 JSON 序列化）向服务器发出 HTTP API 调用。它从 Blazor 项目的 `wwwroot/appsettings.json` 文件中的 `RemoteServices` 配置中读取服务器的根 URL。以下是一个示例配置代码块：

[PRE24]

在本节中，我使用了 ABP 的动态 C# 客户端代理方法来从 Blazor 应用程序中消费 HTTP API。我们将在[*第 14 章*](B17287_14_Epub_AM.xhtml#_idTextAnchor429) *构建 HTTP API 和实时服务*中回到这个话题，同时介绍静态 C# 客户端代理。

下一节将探讨我们如何将脚本和样式文件添加到我们的 Blazor 应用程序中。

# 处理全局脚本和样式

导入 Blazor Server UI 的脚本和样式文件与 ABP 框架的 MVC/Razor Pages UI 相同。您可以参考[*第 12 章*](B17287_12_Epub_AM.xhtml#_idTextAnchor356) *使用 MVC/Razor Pages*来了解如何使用它。本节基于 Blazor WebAssembly。

Blazor WebAssembly 是一个单页应用程序，默认情况下只有一个入口点。`index.html` 文件位于 `wwwroot` 文件夹中，如下所示：

![图 13.6 – wwwroot 文件夹中的 index.html 文件

![图片](img/Figure_13.06_B17287.jpg)

图 13.6 – wwwroot 文件夹中的 index.html 文件

`index.html` 是一个纯 HTML 文件。服务器在未经任何处理的情况下将其发送到浏览器。请记住，一个简单的静态文件服务器可以提供 Blazor WebAssembly 应用程序。浏览器首先加载 `index.html` 文档，然后加载此文档导入的样式和脚本。

如果您打开 `index.html` 文档，您将看到 `ABP:Styles` 注释内的一部分，如下所示：

[PRE25]

这段代码（包括注释）是由 ABP CLI 在您在 Blazor 项目的根目录中执行以下命令时自动创建（并更新）的：

[PRE26]

当您执行此命令时，它创建（或重新生成）全局样式包。这个包包含所有必要的样式，包括 .NET 运行时、Blazor 和其他使用的库，以压缩格式。每次您将新的与 Blazor 相关的 ABP NuGet 包/模块添加到您的应用程序中时，您都需要重新运行 `abp bundle` 命令，并包含必要的依赖项重新生成包。

ABP 的 `bundle` 命令做得很好。当安装一个模块时，您不需要知道它的全局脚本文件或额外依赖项。只需运行此命令，您就有了一个更新后的、生产就绪的全局包文件。每个模块都会将其自己的依赖项贡献到该包中，然后 ABP 通过尊重模块依赖顺序来生成包。要操作包，您应该定义一个实现 `IBundleContributor` 接口的类。启动解决方案模板中的 Blazor 项目已经包含了一个包贡献者，如下面的代码块所示：

[PRE27]

`AddScripts` 和 `AddStyles` 方法用于向全局包添加 JavaScript 和 CSS 文件。您还可以使用 `context.BundleDefinitions` 集合删除或更改现有文件（由您的应用程序依赖的包添加），但这很少需要。在这里，`excludeFromBundle` 参数将 `main.css` 文件单独添加到全局包之外。您可以移除该参数以将其包含在 `global.css` 包文件中。

类似于样式包，`index.html` 文件包含一个 `ABP:Scripts` 部分，如下面的代码块所示：

[PRE28]

再次强调，这段代码部分是由 ABP CLI 使用 `abp bundle` 命令创建（并更新）的。如果您想包含文件，可以在您的包贡献者类的 `AddScripts` 方法中完成。文件的路径被认为是相对于 `wwwroot` 文件夹的相对路径。

# 摘要

本章简要介绍了 ABP 框架的 Blazor UI，以了解其架构以及您将在应用程序中频繁使用的服务。

认证是应用程序中最具挑战性的方面之一，ABP 提供了一个行业标准解决方案，您可以直接在您的应用程序中使用。

我们已经学习了获取当前用户身份信息的服务、检查用户的权限以及本地化用户界面的服务。我们还探索了向用户显示消息框、通知和警报的服务。

ABP 的动态 C# 客户端代理系统使得消费服务器端 HTTP API 变得非常容易。最后，您已经学习了如何使用全局打包系统来处理 Blazor 应用程序中的打包和压缩。

我故意在本章中没有涵盖两个主题。第一个是 Blazor 本身。在本书的单章中涵盖这个非常详细的主题是不够的。如果您是 Blazor 框架的新手，我建议您阅读微软的文档（[https://docs.microsoft.com/en-us/aspnet/core/blazor](https://docs.microsoft.com/en-us/aspnet/core/blazor)）或购买一本专门的书籍。您可以查看由 Packt Publishing 出版的由 Jimmy Engström 编写的《Web Development with Blazor》一书。

本章中我尚未涉及的第二主题是复杂的用户界面组件，例如数据表、模态框和标签页。这些组件非常特定于你使用的UI工具包。ABP自带Blazorise库，你可以参考其文档来学习其组件：[https://blazorise.com/docs](https://blazorise.com/docs)。我还建议你通过ABP框架的Blazor UI教程来了解使用最常用组件（数据表和模态框）的基本开发模型：[https://docs.abp.io/en/abp/latest/Getting-Started](https://docs.abp.io/en/abp/latest/Getting-Started)。

在下一章中，我们将专注于构建HTTP API并在客户端应用程序中消费它们。我们还将探讨使用SignalR库来实现客户端和服务器之间的实时通信。
