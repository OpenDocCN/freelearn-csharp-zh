# *第 15 章*：使用模块化

让我在本章的开头声明——模块化应用程序开发是一项艰巨的工作！我们希望将大型系统拆分成更小的模块并将它们彼此隔离。然而，在集成这些模块并使它们相互通信时，我们将会遇到困难。

ABP 框架的一个基本设计目标是模块化。它提供了构建真正模块化系统的必要基础设施。

本章将从模块化的含义和 .NET 平台的模块化级别开始。在章节的大部分内容中，我们将探讨我为 EventHub 参考解决方案构建的 Payment 模块。我们将学习模块的结构、应用程序模块开发的关键点以及如何将模块安装到主应用程序中。

本章包括以下主要主题：

+   理解模块化

+   构建 Payment 模块

+   将 Payment 模块安装到 EventHub

# 技术要求

您可以从 GitHub 上克隆或下载 EventHub 项目的源代码：[https://github.com/volosoft/eventhub](https://github.com/volosoft/eventhub).

如果你想跟随本章的示例，你需要有一个支持 ASP.NET Core 开发的 IDE/编辑器。

最后，如果你想使用 ABP CLI 创建模块，你应该按照 [*第 2 章*](B17287_02_Epub_AM.xhtml#_idTextAnchor026) 中 *安装 ABP CLI* 部分的说明，在你的计算机上安装它。

# 理解模块化

**可重用性**：构建一个模块并在多个应用程序中重用它，可以减少代码重复并节省时间。

模块化是一种软件设计技术，用于将大型解决方案的代码库分解成更小、独立的模块，然后可以独立开发。模块化应用程序开发背后的两个主要原因是：

+   **降低复杂性**：将大型代码库拆分成更小、独立的模块集合，使得开发和维护解决方案变得容易。

+   模块是软件行业中用得最多、最复杂的概念之一。在本节中，我想解释在 .NET 和 ABP 框架中我所说的模块化是什么。

在接下来的几节中，我将从技术和设计角度讨论两种不同的模块化级别：类库（NuGet 包）和应用模块。让我们从类库开始。

## 类库和 NuGet 包

大多数编程语言和框架都有模块的概念。一般来说，模块是一组代码文件（类和其他资源），它们被开发和一起分发（部署）。

一个模块为更大的应用程序提供一些组件和服务。一个模块可能依赖于其他模块，并可以使用依赖模块提供的组件和服务。

在.NET中，组件（assembly）是创建模块的一种典型方式。我们可以创建一个**类库项目**，然后在其他库和应用程序中使用它。我们可以为类库创建NuGet包，并在[NuGet.org](http://NuGet.org)上公开发布。如果库不是公开的，我们可以在自己的公司中托管一个私有NuGet服务器。NuGet包系统使得将库添加到项目中变得极其容易。在[NuGet.org](http://NuGet.org)上已经发布了成千上万的包。

ABP框架本身被设计成模块化的。它由数百个NuGet包组成；每个包都为您的应用程序提供不同的基础设施功能。一些示例包包括`Volo.Abp.Validation`、`Volo.Abp.Authorization`、`Volo.Abp.Caching`、`Volo.Abp.EntityFrameworkCore`、`Volo.Abp.BlobStoring`、`Volo.Abp.Auditing`和`Volo.Abp.Emailing`。您可以在应用程序中使用您需要的任何包。

您可以参考[*第5章*](B17287_05_Epub_AM.xhtml#_idTextAnchor146)的*理解模块化*部分，*探索ASP.NET Core和ABP基础设施*，以了解基于包的ABP模块。下一节将讨论应用程序模块，通常由多个包（类库项目）组成。

## 应用程序模块

我们可以将应用程序模块视为应用程序的一个垂直切片。应用程序模块具有以下属性：

+   定义一些业务对象（例如，聚合、实体和值对象）

+   实现它定义的业务对象所用的业务逻辑

+   为业务对象提供数据库集成和映射

+   包含应用程序服务、数据传输对象和HTTP API（控制器）

+   可以包含与它提供的功能相关的用户界面组件和页面

+   可能需要在UI的应用程序菜单、布局或工具栏中添加新项目

+   发布和消费分布式事件

+   可能具有更多功能和您期望从常规应用程序中获得的其他详细信息

根据您的需求和目标，应用程序模块有几种隔离级别。以下列出了四个常见示例：

+   **紧密耦合的模块**：一个模块可以是具有单个数据库的大型单体应用程序的一部分。您可以使用该模块的实体和服务在其他模块中使用，并通过连接该模块的表执行数据库查询。这样，您的模块就紧密耦合在一起了。

+   **边界上下文**：一个模块可以是大型单体应用程序的一部分，但它隐藏其内部领域对象和数据库表，使其对其他模块不可见。其他模块只能使用其集成服务并订阅该模块发布的事件。它们不能在SQL查询中使用该模块的数据库表。该模块甚至可能使用不同类型的DBMS来满足其特定需求。这就是领域驱动设计中的边界上下文模式。如果将来您想将单体应用程序转换为微服务解决方案，这样的模块是转换为微服务的良好候选。 

+   **通用模块**：通用模块被设计为与应用程序无关。它们可以集成到不同类型的应用程序中。使用通用模块的应用程序可能依赖于该模块的一些功能，并且可能需要一些集成代码。通用模块可能提供一些选项和自定义点，但不对最终应用程序做出假设。例如，身份管理和多语言模块等基础设施模块属于此类。此外，在*构建支付模块*部分中解释的支付模块也是一个通用模块。

+   **插件模块**：插件模块是一个完全隔离且可重用的应用程序模块。其他模块对该模块没有直接依赖。您可以轻松地将此模块添加到现有解决方案中或从现有解决方案中移除，而不会影响其他模块和您的应用程序。如果其他模块需要使用该模块，它们将使用共享库中提供的某些标准抽象。在这种情况下，该模块实现了这些抽象，可以被实现相同抽象的另一个模块所替换。即使其他模块以某种方式使用该模块，在移除该模块时，它们也可以继续按预期工作。这意味着该模块应该是可选的并且可以移除的，以便于应用程序。

ABP框架的主要目标之一是提供一个方便的基础设施来开发任何类型的应用程序模块。它提供了构建真正模块化系统所需的基础设施详细信息。它还提供了一些预构建的应用程序模块，您可以直接在您的应用程序中使用。以下是一些示例：

+   **账户模块**提供认证功能，例如登录、注册、忘记密码和社交登录集成。

+   **身份模块**管理您系统中的用户、角色及其权限。

+   **租户管理**模块允许您在SaaS/多租户系统中创建和管理租户。

+   **CMS Kit**模块可用于将基本**内容管理系统（CMS**）功能添加到您的应用程序中，例如页面、标签、评论和博客。

账户、身份、租户管理和一些其他模块在创建新的ABP解决方案时预安装（作为NuGet包）。

所有预构建的模块都设计为可扩展和可定制的。然而，如果你需要根据你的需求完全更改一个模块，总是可以下载模块的源代码并将其包含在你的解决方案中。

在下一节中，我们将看到如何使用自己的实体、服务和页面构建一个新的应用程序模块。

# 构建`Payment`模块

你已经可以调查预构建的ABP模块的源代码，看看它们是如何构建和在你的应用程序中使用的。我建议这样做，因为你可以看到模块化开发的不同的实现细节。然而，本节将探讨`Payment`模块，它已被创建为本书的一个简单但真实的示例。它被EventHub解决方案用于当组织想要升级到高级账户时接收支付。在书中不可能展示该模块的逐步开发过程。我们将研究基本点，以便你能够理解模块结构并构建你自己的模块。让我们从创建一个新的应用程序模块开始。

## 创建一个新的应用程序模块

ABP CLI的`new`命令提供了一个选项来创建一个新的解决方案以构建可重用的应用程序模块。请看以下示例：

[PRE0]

我指定使用模块模板（`-t module`），模块名称为`Payment`。如果你打开解决方案，你会看到解决方案结构，如下面的图所示：

![图15.1 – 由ABP CLI创建的新应用程序模块

](img/Figure_15.1_B17287.jpg)

图15.1 – 由ABP CLI创建的新应用程序模块

模块启动模板包含太多的项目，因为它支持多个UI和数据库选项，并包含一些测试/演示项目。让我们删除一些项目：

+   `host`文件夹中的项目是一些演示应用程序，用于在不同的架构选项中运行模块。这些项目不是模块的一部分，只是用于手动测试。我们将把这个模块安装到EventHub解决方案中并在那里测试它，所以我删除了所有主机项目。

+   我删除了`Blazor.*`项目，因为我的主要UI将是MVC/Razor Pages。

+   我删除了与MongoDB相关的项目，因为我只想用我的模块支持EF Core。

+   最后，我删除了`angular`文件夹（在*图15.1*中没有显示），因为我不想为这个模块使用Angular UI。

清理后，模块解决方案中有12个项目。其中四个用于单元和集成测试，因此该模块由八个将部署的项目组成。这八个项目是类库项目，因此它们不能单独运行。它们需要由一个可执行应用程序使用，例如EventHub：

![图15.2 – 清理后的`Payment`模块

](img/Figure_15.2_B17287.jpg)

图15.2 – 清理后的`Payment`模块

这种解决方案结构和层在[*第9章*](B17287_09_Epub_AM.xhtml#_idTextAnchor300)，*理解领域驱动设计*的*根据DDD结构化.NET解决方案*部分中已经解释过了。所以，这里我不会重复全部内容。然而，我们将改变这个结构，因为我们想为支付模块提供多个应用层。

## 重新构建支付模块解决方案

我们将把这个支付模块安装到EventHub解决方案中。记得从[*第4章*](B17287_04_Epub_AM.xhtml#_idTextAnchor130)，*理解参考解决方案*，中了解到EventHub解决方案有两个UI应用程序：

+   一个面向最终用户的公共网站，用户可以通过它创建和参加活动。该应用程序具有MVC/Razor Pages UI。

+   一个由EventHub系统的管理用户使用的管理Web应用程序。该应用程序是一个Blazor WebAssembly应用程序。

为了支持相同的架构，我们将为支付模块提供两个UI应用层：

+   一个包含MVC/Razor Pages UI的应用层，该UI被EventHub公共网站使用。最终用户将通过该UI进行支付。

+   一个包含Blazor WebAssembly UI的应用层，该UI被EventHub管理应用程序使用。管理用户将通过该UI查看支付报告。

以下图显示了我在添加了管理端层并组织了解决方案文件夹后的最终支付解决方案结构：

![图15.3 – 带有管理端面的支付模块解决方案

](img/Figure_15.3_B17287.jpg)

图15.3 – 带有管理端面的支付模块解决方案

我添加了`Payment.Admin.Application`、`Payment.Admin.Application.Contracts`、`Payment.Admin.Blazor`、`Payment.Admin.HttpApi`和`Payment.Admin.HttpApi.Client`项目。我还添加了`Payment.BackgroundServices`项目以执行一些周期性后台工作。

解决方案文件夹反映了整体结构——`admin`应用程序（带有Blazor UI）和`www`（公共）应用程序（带有MVC/Razor Pages UI）。`common`文件夹在两个应用程序中都被使用，所以我们共享相同的领域层和数据库集成代码。

我们已经了解了支付解决方案的整体结构。在下一节中，你将了解支付过程的细节。

## 理解支付过程

支付模块的唯一责任是从用户那里收取支付。它内部使用PayPal作为支付网关。支付模块是通用的，可以被任何类型的应用程序使用。使用支付模块的应用程序应包含一些集成逻辑，以启动支付过程并处理支付结果。在本节中，我将基于EventHub集成来解释这个过程。

EventHub应用程序使用支付模块从用户那里获取支付，以将免费组织账户升级为高级组织账户。正如你所猜到的，高级组织在应用程序中拥有更多权利。

如果您是组织的所有者并访问组织详情页面，您将在页面上看到一个**升级至高级版**按钮，如图所示：

![图15.4 – EventHub组织详情页面

![图片](img/Figure_15.4_B17287.jpg)

图15.4 – EventHub组织详情页面

当您点击**升级至高级版**按钮时，您将被重定向到定价页面：

![图15.5 – EventHub定价页面

![图片](img/Figure_15.5_B17287.png)

图15.5 – EventHub定价页面

在这里，我们可以看到账户类型及其差异。当我们在这里点击**升级至高级版**按钮时，我们将被重定向到预结账页面，该页面由支付模块定义：

![图15.6 – 预结账页面

![图片](img/Figure_15.6_B17287.jpg)

图15.6 – 预结账页面

预结账页面通常位于支付模块内部，并开发成与应用程序无关。我们可以通过一个如`/Payment/PreCheckout?paymentRequestId=3a002186-cb04-eb46-7310-251e45fc6aed`的URL将用户重定向到预结账页面。然而，我们首先需要使用`IPaymentRequestAppService`服务的`CreateAsync`方法获取一个支付请求ID。这是在`EventHub.Web`项目的`Pages/Pricing.cshtml.cs`文件中完成的。

EventHub应用程序覆盖视图（UI）部分以更好地适应EventHub的UI设计。这是在最终应用程序中自定义模块的一个示例。EventHub应用程序在`Pages/Payment`文件夹下定义了`PreCheckout.cshtml`和`PostCheckout.cshtml`文件，如图所示：

![图15.7 – 覆盖支付模块的结账视图

![图片](img/Figure_15.7_B17287.jpg)

图15.7 – 覆盖支付模块的结账视图

它们自动覆盖相应的支付页面（因为它们正好位于支付模块定义的相同路径中）。这些页面没有`.cshtml.cs`文件，因为我们不想改变页面的行为；我们只想改变视图方面。

下图显示了支付过程中使用的主要组件和流程：

![图15.8 – 支付流程

![图片](img/Figure_15.8_B17287.jpg)

图15.8 – 支付流程

当我们在定价页面（如图15.5所示）上点击**升级至高级版**按钮时，它将重定向到支付模块的结账页面。当我们点击该页面上的**结账**按钮（如图15.6所示）时，我们将被重定向到PayPal，这是支付模块使用和集成的支付系统。一旦我们在PayPal上完成支付，我们将被重定向回应用程序的结账后页面，该页面会向用户显示感谢信息。

当支付过程成功时，支付模块发布一个名为 `PaymentRequestCompletedEto` 的分布式事件（在 `Payment.Domain.Shared` 项目中定义）。EventHub 应用程序订阅此事件（在 `EventHub.Domain` 项目中的 `PaymentRequestEventHandler` 类内部）。它找到与完成支付相关的用户和组织，升级组织，并发送电子邮件感谢用户升级账户。

在某些罕见情况下，从 PayPal 返回到我们的应用程序时可能会出现错误，我们无法知道支付过程是否成功。对于此类情况，支付模块提供了一个 `PaymentRequestController`（位于 `Payment.HttpApi` 项目中）。如果操作成功，将发布相同的 `PaymentRequestCompletedEto` 事件，以便 EventHub 应用程序可以异步升级组织账户。

在下一节中，我们将看到支付模块如何提供配置选项。

## 提供配置选项

支付模块使用 PayPal，因此它需要应用程序必须配置的 PayPal 账户信息。它遵循选项模式（见 [*第 5 章*](B17287_05_Epub_AM.xhtml#_idTextAnchor146)，*探索 ASP.NET Core 和 ABP 基础设施）中的 *实现选项模式* 部分），并提供了一个 `PayPalOptions` 类，该类可以被应用程序配置，如下例所示：

[PRE1]

我们通常从配置（`appsettings.json` 文件）中获取值。如果你已经定义了它，支付模块可以从 `Payment:PayPal` 键获取选项值，如下例所示：

[PRE2]

这可以通过以下代码实现，该代码位于 `Payment.Domain` 项目的 `PaymentDomainModule` 类中：

[PRE3]

默认情况下从配置中获取值是一种良好的做法。

我已经介绍了支付模块结构的主要要点。其代码库与典型的 ABP 应用程序没有太大区别。你可以探索其源代码来了解其内部工作原理。在下一节中，我们将看到它如何在 EventHub 应用程序中安装。

# 将支付模块安装到 EventHub

模块本身不是一个可运行的项目。它应该安装到更大的应用程序中，并作为其一部分工作。在本节中，我们将看到支付模块是如何在 EventHub 解决方案中安装的。

## 设置项目依赖项

支付模块在其解决方案中包含超过 10 个项目（见 *图 15.3*）。同样，EventHub 解决方案包含许多项目，有三个应用程序——管理端、公共端和账户（`IdentityServer`）应用程序。

我想将支付模块集成到 EventHub 解决方案的所有层中。通常，EventHub 解决方案中的每一层都应该依赖于（使用）支付模块的相应层。以下表格显示了 EventHub 项目对支付模块项目的所有依赖关系：

![Figure 15.9 – EventHub and Payment module project dependencies]

![img/Figure_15.9_B17287.jpg]

图15.9 – EventHub和支付模块项目依赖

因此，我们应该逐个添加项目引用。例如，我们将`Payment.Domain`项目依赖项添加到`EventHub.Domain`项目中。这样，我们就可以在我们的应用程序域层中使用支付模块的实体。

Visual Studio不支持从解决方案外部（我称之为外部项目依赖）向项目添加本地项目依赖项。然而，我们可以手动将`ProjectReference`元素添加到目标项目的`csproj`文件中。因此，我们可以将以下行添加到`EventHub.Domain.csproj`文件中：

[PRE4]

当我们添加这样的外部项目依赖项时，Visual Studio无法自动解析它。我们应该打开命令行终端并运行`dotnet restore`命令。此命令仅在您添加新依赖项或删除现有依赖项时需要。此外，如果您想使用支付模块构建EventHub解决方案，可以使用`dotnet build /graphBuild`命令。虽然这很少需要，但它可以在Visual Studio无法解析依赖模块中的某些类型时救命。

一旦添加了项目引用，我们也应该添加ABP模块依赖。以下代码块显示了`EventHubDomainModule`类的`PaymentDomainModule`依赖项：

[PRE5]

我们应手动设置所有项目依赖项，如这里所述。下一步是配置支付模块的数据库表格。

## 配置数据库集成

支付模块需要一些数据库表格才能正常工作。我们可以使用主EventHub数据库来存储支付模块的表格。采用这种方法，我们将为系统拥有一个单一数据库。或者，我们可以为支付模块创建一个单独的数据库，这样我们就有两个数据库。EventHub解决方案更喜欢第一种方法，因为它更容易实现和管理。然而，我还会展示我们如何实现单独的数据库方法。让我们从单一数据库方法开始。

### 使用单个数据库

在本节中，我将向您展示我们如何在EventHub应用程序的主数据库中创建支付表格。

EventHub解决方案在`EventHub.EntityFrameworkCore`项目中包含一个`EventHubDbContext`类，这是将实体映射到数据库表格的主要类。支付模块定义了一个`ConfigurePayment`扩展方法，我们从`DbContext`类的`OnModelCreating`方法中调用它，以将支付数据库映射模型包含到我们的主数据库模型中（请参阅`EventHub.EntityFrameworkCore`项目中的`EventHubDbContext`类）：

[PRE6]

在这里，`builder.ConfigurePayment()` 由支付模块定义（在 `Payment.EntityFrameworkCore` 项目的 `PaymentDbContextModelCreatingExtensions` 类中）。在 `OnModelCreating` 方法内添加此行后，我们可以使用以下命令行终端中的命令将新的数据库迁移添加到 EventHub 解决方案中（我们在 `EventHub.EntityFrameworkCore` 项目的 `root` 文件夹中运行此命令）：

[PRE7]

此命令创建一个新的迁移文件。然后，我们可以使用以下命令对数据库应用新的迁移：

[PRE8]

那就结束了。支付模块将使用主 EventHub 数据库来存储其数据。这样，我们将有一个包含应用程序所有表的单一数据库。在下一节中，我们将讨论单独的数据库方法。

### 使用单独的数据库

在本节中，我们将看到如何将 EventHub 解决方案更改为为支付模块使用单独的数据库。EventHub 解决方案使用 PostgreSQL 作为数据库提供者。我们将使用 Microsoft 的 SQL Server 作为支付模块。这样，你将学习如何在单个应用程序中与多个数据库提供者一起工作。

我在一个单独的分支中做出了这些更改，并在 GitHub 上创建了一个草稿 **Pull Request**（**PR**），这样你就可以看到所有更改：

+   GitHub 分支 URL: [https://github.com/volosoft/eventhub/tree/payment-sepr-db](https://github.com/volosoft/eventhub/tree/payment-sepr-db)

+   PR URL: [https://github.com/volosoft/eventhub/pull/74](https://github.com/volosoft/eventhub/pull/74)

在这里，我将指出我做出的基本更改。你可以在 GitHub 上的 PR 中查看所有更改。

首先，我在 `EventHub.EntityFrameworkCore` 项目中创建了一个名为 `EventHubPaymentDbContext` 的第二个 `DbContext` 类，以管理数据库迁移：

[PRE9]

这个类使用 `[ReplaceDbContext]` 属性替换了由支付模块定义的 `IPaymentDbContext`，并实现了 `IPaymentDbContext` 接口。它还声明了 `[ConnectionStringName]` 属性，在 `appsettings.json` 文件中使用 `Payment` 连接字符串名称而不是 `Default`。最后，它调用了支付模块的 `modelBuilder.ConfigurePayment()` 扩展方法来配置数据库映射。

支付模块被设计成独立于任何特定的 `Volo.Abp.EntityFrameworkCore` 包，它是数据库管理系统无关的。由于我想使用 SQL Server 数据库，我已经将 `Volo.Abp.EntityFrameworkCore.SqlServer` 包依赖项添加到 `EventHub.EntityFrameworkCore` 项目中。我还将 `AbpEntityFrameworkCoreSqlServerModule` 添加到 `EventHubEntityFrameworkCoreModule` 类的 `DependsOn` 属性中，因为 ABP 需要它。

EF Core 的命令行工具需要创建一个 `DbContext` 工厂类，以便在运行其命令时创建相关 `DbContext` 类的实例。您可以在源代码中看到 `EventHubPaymentDbContextFactory` 类。它使用 `Payment` 连接字符串和 `UseSqlServer` 扩展方法来配置 SQL Server 作为数据库提供程序。通过这个更改，我们应该在 `EventHub.DbMigrator` 项目的 `appsettings.json` 文件中添加 `Payment` 连接字符串：

[PRE10]

ABP 将自动获取新 `DbContext` 类的 `Payment` 连接字符串，因为它具有 `ConnectionStringName` 属性（`PaymentDbProperties.ConnectionStringName` 的值是 `Payment`）。我还需要将 `Payment` 连接字符串添加到所有我已经定义了 `Default` 连接字符串的 `appsettings.json` 文件中。

我应该将新的 `EventHubPaymentDbContext` 类注册到依赖注入系统中并配置它。为此，我已更改 `EventHubEntityFrameworkCoreModule` 类的 `ConfigureServices` 方法，如下所示：

[PRE11]

`AddAbpDbContext<EventHubPaymentDbContext>()` 调用注册了新的 `DbContext` 类。我还添加了 `Configure<EventHubPaymentDbContext>(…)` 块来为这个 `DbContext` 类使用 SQL Server。其他 `DbContext` 类将继续使用 PostgreSQL（全局配置所有 `DbContext` 类的 `UseNpgsql()` 调用）。

`EventHub.DbMigrator` 应用程序为主数据库执行数据库迁移。现在，我们有了第二个数据库，我们想要更改 `EventHub.DbMigrator` 应用程序，使其也执行支付模块数据库的数据库迁移。更改很简单；我在 `EntityFrameworkCoreEventHubDbSchemaMigrator` 类的 `MigrateAsync` 方法内部添加了以下代码块：

[PRE12]

这个类在 `EventHub.DbMigrator` 应用程序迁移数据库时使用。因此，通过添加这个代码块，当运行 `EventHub.DbMigrator` 应用程序时，新的数据库也会被迁移。

作为最后的更改，我将从主 EventHub 数据库中移除支付表格，并从 `EventHubDbContext` 类中移除以下行：

[PRE13]

然后，我可以使用 EF Core 的命令行工具创建数据库迁移（在 `EventHub.EntityFrameworkCore` 项目的 `root` 文件夹中）：

[PRE14]

与标准用法不同，我添加了 `--context EventHubDbContext` 参数。我指定 `DbContext` 类型，因为 `EventHub.EntityFrameworkCore` 项目中有两个 `DbContext` 类。一旦创建迁移（删除支付表格），我就可以使用以下命令对数据库应用更改：

[PRE15]

现在，主数据库没有支付表格。但我们还没有创建支付数据库。为此，我可以使用 EF Core 的命令行工具为支付数据库创建数据库迁移（在 `EventHub.EntityFrameworkCore` 项目的 `root` 文件夹中）：

[PRE16]

这次，除了指定`EventHubPaymentDbContext`类型的`context`参数之外，我还设置了`output-dir`参数以指定创建迁移类的文件夹。默认文件夹名称是`Migrations`，但这个名称被`EventHubDbContext`类使用，所以我不能使用它。我将文件夹名称指定为`MigrationsPayment`。以下图显示了`EventHub.EntityFrameworkCore`项目中的新迁移文件夹：

![图15.10 – 支付模块的迁移文件夹

](img/Figure_15.10_B17287.jpg)

图15.10 – 支付模块的迁移文件夹

现在，我可以在`EventHub.EntityFrameworkCore`项目的`root`文件夹中使用以下命令：

[PRE17]

如果我检查数据库，我可以看到支付模块的表（它只有一个数据库表）：

![图15.11 – 支付模块在其自己的数据库中的表

](img/Figure_15.11_B17287.jpg)

图15.11 – 支付模块在其自己的数据库中的表

单独的数据库配置已完成。现在，支付模块将使用新的SQL Server数据库，而应用程序的其他部分将继续与主PostgreSQL数据库一起工作。

使用DbMigrator

我已经使用`dotnet ef`命令行工具更新了数据库模式。然而，我们也可以运行`DbMigrator`应用程序以将更改应用到数据库中。由于我们还将`EntityFrameworkCoreEventHubDbSchemaMigrator`类更改为支持第二个数据库，因此`DbMigrator`可以迁移两个数据库的模式。

通过创建新的迁移`DbContext`类，如这里所述，您可以设置其他模块使用它们自己的数据库。我在同一个`EventHub.EntityFrameworkCore`项目中添加了新的`DbContext`类；然而，我们也可以为新的`DbContext`类创建一个新的项目，并在其中管理迁移。在这种情况下，我们不需要为EF Core命令指定上下文和`output-dir`参数。然而，我建议使用单个项目以最小化解决方案中的项目数量，因为它已经有很多了。

# 摘要

在本章中，我首先解释了模块化的含义以及边界上下文、紧密耦合、通用和插件模块是什么。我们学习了如何使用ABP CLI创建一个新的模块。

然后，我们探讨了支付模块的结构，并了解了它是如何集成到EventHub解决方案中的。我们学习了通过设置项目依赖关系手动将支付模块安装到EventHub解决方案中的步骤。

最后，我们看到了两种使用支付数据库表的方法。单一数据库方法简单，EventHub应用程序和支付模块共享相同的数据库。另一方面，单独的数据库方法允许我们为支付表使用专用数据库，使得可以在支付模块中使用与主应用程序不同的DBMS。

我建议您检查支付模块和EventHub解决方案的源代码，以了解它们结构的所有细节。我还建议您查阅ABP文档，以更好地理解模块化，并学习构建可重用、通用应用模块的最佳实践：[https://docs.abp.io/en/abp/latest/Best-Practices/Index](https://docs.abp.io/en/abp/latest/Best-Practices/Index).

在下一章中，我们将探讨多租户的概念，这是构建SaaS应用的关键。
