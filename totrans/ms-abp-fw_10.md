# *第8章*：使用ABP的功能和服务

ABP框架是一个全栈应用程序开发框架，因此它为企业的每个方面提供了许多构建块。在前三章中，我们已经探讨了ABP框架提供的核心服务、数据访问基础设施和横切关注点解决方案。

在本节“第2部分”的最后一章，“ABP框架基础”中，我们将继续探讨在业务应用程序中经常使用的ABP功能，顺序如下：

+   获取当前用户

+   使用数据过滤系统

+   控制审计日志系统

+   缓存数据

+   本地化用户界面（**UI**）

# 技术要求

如果您想跟随并尝试这些示例，您需要安装一个**集成开发环境**（**IDE**）/编辑器（例如Visual Studio）来构建ASP.NET Core项目。

您可以从以下GitHub仓库下载代码示例：[https://github.com/PacktPublishing/Mastering-ABP-Framework](https://github.com/PacktPublishing/Mastering-ABP-Framework).

# 获取当前用户

如果您的应用程序需要某些功能进行用户身份验证，通常您需要获取当前用户的信息。ABP提供了`ICurrentUser`服务来获取当前登录用户的详细信息。对于Web应用程序，`ICurrentUser`的实现完全集成到ASP.NET Core的认证系统中，因此您可以轻松获取当前用户的声明。

以下代码块展示了`ICurrentUser`服务的简单用法：

[PRE0]

在此示例中，`MyService`构造函数注入了`ICurrentUser`服务，然后获取当前用户的唯一`Id`、`Username`和`Email`值。

下面是`ICurrentUser`接口的属性：

+   `IsAuthenticated`（`bool`）：如果当前用户已登录（认证），则返回`true`。

+   `Id`（`Guid?`）：如果当前用户未登录，则为`null`。

+   `UserName`（`string`）：当前用户的用户名。如果当前用户未登录，则返回`null`。

+   `TenantId`（`Guid?`）：当前用户的租户ID。对于多租户应用程序是可用的。如果当前用户与租户无关，则返回`null`。

+   `Email`（`string`）：当前用户的电子邮件地址。如果当前用户未登录或未设置电子邮件地址，则返回`null`。

+   `EmailVerified`（`bool`）：如果当前用户的电子邮件地址已验证，则返回`true`。

+   `PhoneNumber`（`string`）：当前用户的电话号码。如果当前用户未登录或未设置电话号码，则返回`null`。

+   `PhoneNumberVerified`（`bool`）：如果当前用户的电话号码已验证，则返回`true`。

+   `Roles`（`string[]`）：当前用户的所有角色作为一个字符串数组。

    注入ICurrentUser服务

    `ICurrentUser`是一个广泛使用的服务。因此，一些基础ABP类（如`ApplicationService`和`AbpController`）预先注入了它。在这些类中，你可以直接使用`CurrentUser`属性，而不是手动注入此服务。

ABP可以与任何身份验证提供者一起工作，因为它使用ASP.NET Core提供的当前声明。**声明**是在用户登录时发放的键值对，并存储在身份验证票据中。如果你使用基于cookie的身份验证，它们存储在cookie中，并在每次请求中发送到服务器。如果你使用基于令牌的身份验证，它们由客户端在每次请求中发送，通常在**超文本传输协议**（**HTTP**）头中。

`ICurrentUser`服务从当前声明中获取所有信息。如果你想直接查询当前声明，可以使用`FindClaim`、`FindClaims`和`GetAllClaims`方法。如果你创建了自定义声明，这些方法特别有用。

## 定义自定义声明

ABP提供了一种简单的方法将你的自定义声明添加到身份验证票据中，以便你可以在同一用户的下一个请求中安全地获取这些自定义值。你可以实现`IAbpClaimsPrincipalContributor`接口，将自定义声明添加到身份验证票据中。

在以下示例中，我们正在将社会保险号码信息——一个自定义声明——添加到身份验证票据中：

[PRE1]

在此示例中，我们首先获取`ClaimsIdentity`并找到当前用户的ID。然后，我们从`IUserService`获取社会保险号码，这是一个你应该自己开发的自定义服务。你可以从`ServiceProvider`获取任何服务来查询所需的数据。最后，我们向`identity`添加一个新的`Claim`。`SocialSecurityNumberClaimsPrincipalContributor`随后在用户登录应用程序时使用。

你可以使用自定义声明来授权当前用户满足特定业务需求、过滤数据或仅显示在UI上。请注意，除非你使身份验证票据无效并强制用户重新认证，否则身份验证票据的声明不能更改，因此不要在声明中存储频繁更改的数据。如果你的目的是存储可以在以后快速访问的用户数据，你可以使用缓存系统（将在*数据缓存*部分介绍）。

`ICurrentUser`是在你的应用程序代码中频繁使用的核心服务。下一节将介绍数据过滤系统，它在大多数情况下可以无缝工作。

# 使用数据过滤系统

在数据库操作中过滤数据是非常常见的。如果你使用`WHERE`子句。如果你在C#中使用`Where`扩展方法。虽然这些过滤条件在查询中可能有所不同，但如果你实现了如软删除和多租户等模式，一些表达式将应用于你运行的每个查询。

ABP 自动化数据过滤过程，帮助你避免在应用程序代码的每个地方重复相同的过滤逻辑。

在本节中，我们将首先了解 ABP 框架的预构建数据过滤器，然后学习如何在需要时禁用这些过滤器。最后，我们将看到如何实现我们自己的自定义数据过滤器。

我们通常使用简单的接口来为实体启用过滤功能。ABP 定义了两个预定义的数据过滤器来实现软删除和多租户模式。

## 软删除数据过滤器

如果你为实体使用软删除模式，你永远不会在数据库中物理删除该实体。相反，你将其标记为 *已删除*。

ABP 定义了 `ISoftDelete` 接口，以标准化标记实体为软删除的属性。你可以为实体实现该接口，如下面的代码块所示：

[PRE2]

在这个例子中，`Order` 实体有一个由 `ISoftDelete` 接口定义的 `IsDeleted` 属性。一旦你实现了该接口，ABP 会为你自动完成以下任务：

+   当你删除一个订单时，ABP 会识别出 `Order` 实体实现了软删除模式，阻止删除操作，并将 `IsDeleted` 设置为 `true`。因此，订单在数据库中不会被物理删除。

+   当你查询订单时，ABP 会自动过滤已删除的实体（通过在查询中添加 `IsDeleted == false` 条件），以避免意外从数据库中检索已删除的订单。

数据过滤与查询相关，因此，第一个任务与数据过滤没有直接关系，而是由 ABP 框架实现的支持逻辑。

数据过滤限制

数据过滤自动化仅在您使用存储库或 `DbContext`（对于 `DELETE` 或 `SELECT` 命令，您应该自己处理，因为 ABP 在这些情况下无法拦截您的操作）时才有效。

软删除过滤器是 ABP 数据过滤器中内置的一种。另一种内置过滤器用于多租户。

## 多租户数据过滤器

多租户是一种广泛使用的模式，用于在软件即服务（**SaaS**）解决方案中共享租户之间的资源。在多租户应用程序中隔离不同租户之间的数据至关重要。一个租户不能读取或写入另一个租户的数据，即使它们位于同一个物理数据库中。

ABP 拥有一个完整的多租户系统，这将在[*第16章*](B17287_16_Epub_AM.xhtml#_idTextAnchor457)中详细解释，*实现多租户*。然而，在这里提及多租户过滤器也是好的，因为它与数据过滤系统相关。

ABP 定义了 `IMultiTenant` 接口，以启用实体的多租户数据过滤器。我们可以为实体实现该接口，如下面的代码块所示：

[PRE3]

`IMultiTenant` 接口定义了 `TenantId` 属性，如下面的示例所示。ABP 使用 `Guid` 值作为租户 ID。

一旦我们实现了`IMultiTenant`接口，ABP将自动使用当前租户的ID对所有`Order`实体的查询进行过滤。当前租户的ID是从`ICurrentTenant`服务中获取的，这将在[*第16章*](B17287_16_Epub_AM.xhtml#_idTextAnchor457)中解释，*实现多租户*。

多个数据过滤器的使用

对于同一实体，可以启用多个数据过滤器。例如，本节中定义的`Order`实体可以同时实现`ISoftDelete`和`IMultiTenant`接口。

如您所见，为实体实现数据过滤器相当简单——只需实现与数据过滤器相关的接口。除非您明确禁用它们，否则所有数据过滤器默认启用。

## 禁用数据过滤器

在某些情况下，禁用自动过滤器可能是必要的——例如，您可能想要禁用软删除过滤器以从数据库中读取已删除的实体，或者您可能希望允许用户恢复已删除的实体。您可能想要禁用多租户过滤器以查询多租户系统中的所有租户的数据。无论出于何种原因，ABP都提供了一个简单且安全的方法来禁用数据过滤器。

以下示例展示了如何通过使用`IDataFilter`服务禁用`ISoftDelete`数据过滤器来从数据库中获取所有订单，包括已删除的订单：

[PRE4]

在本例中，`OrderService`注入了`Order`存储库和`IdataFilter`服务。然后它使用`_dataFilter.Disable<IsoftDelete>()`表达式来禁用软删除过滤器。在`using`语句中，过滤器被禁用，我们也可以查询已删除的订单。

总是使用using语句

`Disable`方法返回一个可处置的对象，这样我们就可以在`using`语句中使用它。一旦`using`块结束，过滤器将自动恢复到之前的状态，这意味着如果它在`using`块之前已启用，它将返回到启用状态。如果它在`using`语句之前已经禁用，则`Disable`方法不会影响它，并且它在`using`语句之后保持禁用状态。这个系统允许我们安全地禁用过滤器，而不会影响调用`GetAllOrders`方法的任何逻辑。始终建议在`using`语句中禁用过滤器。

`IdataFilter`服务提供了另外两个方法：

+   `Enable<Tfilter>`：启用数据过滤器。您可以使用此方法在禁用过滤器的范围内临时启用数据过滤器。如果过滤器已经启用，则此方法没有效果。始终建议在`using`语句中启用过滤器，就像使用`Disable`方法一样。

+   `IsEnabled<Tfilter>`：如果给定的过滤器当前已启用，则返回`true`。您通常不需要此方法，因为`Enable`和`Disable`在两种情况下都按预期工作。

我们已经学习了如何使用预构建的`Disable`和`Enable`数据过滤器。下一节将展示如何创建自定义数据过滤器。

## 定义自定义数据过滤器

正如与预构建的数据过滤器一样，你可能想定义自己的过滤器。数据过滤器由一个接口表示，因此第一步是为你的过滤器定义一个接口。

假设你想要存档你的实体，并自动过滤存档数据，以便默认情况下不将它们检索到应用程序中。对于这个例子，我们可以定义这样一个简单的接口（你可以在你的领域层中定义这个接口），如下所示：

[PRE5]

`IsArchived` 属性将用于过滤实体。默认情况下，具有 `IsArchived` 为 `true` 的实体将被排除。一旦我们定义了这样的接口，我们就可以为可以存档的实体实现它。请看以下示例：

[PRE6]

在这个例子中，`Order` 实体实现了 `Iarchivable` 接口，这使得可以在该实体上应用数据过滤器。

注意，`Iarchivable` 接口没有为 `IsArchived` 定义设置器，但 `Order` 实体定义了它。这是我的设计决策；我们不需要在接口上设置 `IsArchived`，但需要在实体上设置它。

由于数据过滤是在数据库提供程序级别完成的，因此自定义过滤器实现也取决于数据库提供程序。本节将展示如何为 EF Core 提供程序实现 `Iarchivable` 过滤器。如果你在寻找 MongoDB，请参阅 ABP 的文档：[https://docs.abp.io/en/abp/latest/Data-Filtering](https://docs.abp.io/en/abp/latest/Data-Filtering)。

ABP 使用 EF Core 的 `DbContext` 类。

第一步是在你的 `DbContext` 类中定义一个属性，该属性将用于过滤表达式，如下所示：

[PRE7]

此属性直接使用 `IdataFilter` 服务来获取过滤状态。`DataFilter` 属性来自基 `AbpDbContext` 类，如果 `DbContext` 实例未从 `null` 检查中解析出来，则它可以是 `null`。

下一步是重写 `ShouldFilterEntity` 方法以决定是否应该过滤给定的实体类型：

[PRE8]

ABP 框架为这个 `DbContext` 类中的每个实体类型调用此方法（它只调用一次——在应用程序启动后第一次使用 `DbContext` 类时）。如果此方法返回 `true`，则启用该实体的 EF Core 全局过滤器。在这里，我只是检查了给定的实体是否实现了 `IArchivable` 接口，并在该情况下返回 `true`。否则，调用 `base` 方法，以便它检查其他数据过滤器。

`ShouldFilterEntity` 只决定是否启用过滤。实际的过滤逻辑应该通过重写 `CreateFilterExpression` 方法来实现：

[PRE9]

实现似乎有点复杂，因为它创建了并组合了表达式。重要的是如何定义`archiveFilter`表达式。`!IsArchiveFilterEnabled`检查过滤器是否已禁用。如果过滤器已禁用，则不会评估其他条件，并且将检索所有实体而不进行过滤。`!EF.Property<bool>(e, "IsArchived")`检查该实体的`IsArchived`值是否为`false`，因此消除了`IsArchived`值为`true`的实体。

如您从前面的代码块中看到的，我未在过滤器实现中使用`Order`实体。这意味着实现是通用的，可以与任何实体类型一起工作——您只需要为要应用过滤器的实体实现`IArchivable`接口。

总结来说，ABP使我们能够轻松创建和控制全局查询过滤器。它还使用该系统实现两种流行的模式——软删除和多租户。下一节将介绍审计日志系统，这是ABP的另一个在企业级软件解决方案中非常常见的功能。

# 控制审计日志系统

ABP的审计日志系统跟踪所有请求和实体更改，并将它们写入数据库。然后，您可以获取关于在您的应用程序中做了什么、何时做的以及谁做的报告。

当您从启动模板创建新解决方案时，审计日志系统已安装并正确配置。大多数时候，您无需任何配置即可使用它。然而，ABP允许您控制、自定义和扩展审计日志系统。但首先，让我们了解审计日志对象是什么。

## 审计日志对象

审计日志对象是一组在有限的作用域内（通常是在Web应用的HTTP请求中）一起执行的动作和相关实体更改。我们将在下一节中更多地讨论审计日志作用域。

*图8.1* 中的图表示审计日志对象：

![图8.1 – 审计日志对象

](img/Figure_8.1_B17287.jpg)

图8.1 – 审计日志对象

让我们从根对象开始解释该图，如下所示：

+   `AuditLogInfo`: 在每个作用域（通常是一个Web请求）中，都有一个包含有关当前用户、当前租户、HTTP请求、客户端和浏览器详细信息以及操作执行时间和持续时间的`AuditLogInfo`对象。

+   `AuditLogActionInfo`: 在每个审计日志中，可能有零个或多个动作。动作通常是控制器动作调用、页面处理程序调用或应用程序服务方法调用。它包括调用中的类名、方法名和方法参数。

+   `EntityChangeInfo`: 审计日志对象可能包含零个或多个对数据库中实体的更改。每个实体更改包含更改类型（创建、更新或删除）、实体类型（完整类名）以及更改实体的ID。

+   `EntityPropertyChangeInfo`：对于每个实体变化，它会在属性（数据库中的字段）上记录变化。此对象包含受影响属性的名字、类型、旧值和新值。

+   `Exception`：在这次审计日志作用域中发生的异常列表。

+   `Comment`：与这个审计日志相关的附加评论/日志。

审计日志对象被保存到关系数据库的多个表中：`AbpAuditLogs`、`AbpAuditLogActions`、`AbpEntityChanges` 和 `AbpEntityPropertyChanges`。我在前面的列表中已经写出了审计日志对象的基本属性。你可以检查这些数据库表或调查 `AuditLogInfo` 对象以查看所有详细信息。

MongoDB 限制

由于 ABP 使用 EF Core 的更改跟踪系统来获取实体更改信息，而 MongoDB 驱动程序没有这样的更改跟踪系统，因此不会为 MongoDB 记录实体更改。

如本节开头所述，每个审计日志作用域都会创建一个审计日志对象。

## 审计日志作用域

审计日志作用域使用 **环境上下文模式**。当你创建一个新的审计日志作用域时，在这个作用域中进行的所有操作和更改都会保存为一个单独的审计日志对象。

有几种方法可以建立审计日志作用域。

### 审计日志中间件

创建审计日志作用域的第一种和最常见的方式是在 ASP.NET Core 管道配置中使用审计日志中间件：

[PRE10]

这通常放置在 `app.UseEndpoints()` 或 `app.UseConfiguredEndpoints()` 端点配置之前。当你使用这个中间件时，每个HTTP请求都会写入一个单独的审计日志记录，这在大多数情况下是期望的行为，并且默认情况下已经在启动模板中配置好了。

### 审计日志拦截器

如果你没有使用审计日志中间件，或者如果你的应用程序不是请求/回复风格的 ASP.NET Core 应用程序（例如，桌面或 Blazor Server 应用程序），那么 ABP 会为每个应用程序服务方法创建一个新的审计日志作用域。

### 手动创建审计作用域

你通常不需要这样做，但如果你想手动创建审计作用域，可以使用 `IAuditingManager` 服务，如下面的代码块所示：

[PRE11]

一旦注入了 `IAuditingManager` 服务，你可以使用 `BeginScope` 方法创建一个新的作用域。然后，创建一个 `try`-`catch` 块来保存审计日志，包括异常情况。在 `try` 部分中，你只需执行你的逻辑，调用其他服务等。所有这些操作以及这些操作中的更改都作为单个审计日志对象保存在 `finally` 块中。

在审计日志作用域内（无论是由 ABP 创建还是您手动创建），可以使用 `_auditingManager.Current.Log` 来获取当前的审计日志对象以进行调查或操作它（例如，添加注释行或附加信息）。如果您不在审计日志作用域内，则 `_auditingManager.Current` 返回 `null`，因此如果您不确定是否存在周围的审计日志作用域，请检查 `null`。

我已经介绍了审计日志对象和审计日志作用域，它们默认情况下可以无缝工作。现在，让我们看看选项，以了解默认值和审计日志系统的全局配置可能性。

## 审计选项

`AbpAuditingOptions` 类用于配置审计系统的默认选项。它可以使用标准的 `options` 模式进行配置，如下面的示例所示：

[PRE12]

您可以在模块的 `ConfigureServices` 方法中配置 `options`。以下列表显示了审计系统的主要选项：

+   `IsEnabled` (`bool`; 默认: `true`): 完全禁用审计系统的主要点。

+   `IsEnabledForGetRequests` (`bool`; 默认: `false`): ABP 默认不保存 HTTP `GET` 请求的审计日志，因为 `GET` 请求不应该更改数据库。但是，您可以将其设置为 `true`，这样也可以为 `GET` 请求启用审计日志。

+   `IsEnabledForAnonymousUsers` (`bool`; 默认: `true`): 如果您只想为认证用户写入审计日志，请将其设置为 `false`。如果您为匿名用户保存审计日志，您将看到这些用户的 `UserId` 值为 `null`。

+   `AlwaysLogOnException` (`bool`; 默认: `true`): 如果您的应用程序代码中发生异常，ABP 默认会保存审计日志，而不考虑 `IsEnabledForGetRequests` 和 `IsEnabledForAnonymousUsers` 选项。将此设置为 `false` 以禁用该行为。

+   `hideErrors` (`bool`; 默认: `true`): ABP 在将审计日志对象保存到数据库时忽略异常。将此设置为 `false` 以抛出异常而不是隐藏它们。

+   `ApplicationName` (`string`; 默认: `null`): 如果多个应用程序使用相同的数据库来保存审计日志，您可以在每个应用程序中设置此选项，以便可以根据应用程序名称过滤日志。

+   `IgnoredTypes` (`List<Type>`): 您可以在审计日志系统中忽略一些特定的类型，包括实体类型。

除了这些简单的全局选项之外，您还可以为实体启用/禁用更改跟踪。

### 启用实体历史记录

审计日志对象包含具有属性详细信息的实体更改。然而，默认情况下它对所有实体都是禁用的，因为它可能会将过多的日志写入数据库，这可能会迅速增加数据库的大小。建议以受控的方式为要跟踪的实体启用它。

为实体启用实体历史记录有两种方式，如下所述：

+   使用 `[Auditing]` 属性为单个实体启用它。它将在下一节中解释。

+   `EntityHistorySelectors` 选项用于为多个实体启用它。

在以下示例中，我为所有实体启用了 `EntityHistorySelectors` 选项：

[PRE13]

`AddAllEntities` 方法是一个快捷方式。`EntityHistorySelectors` 是一个命名选择器的列表，你可以添加一个 lambda 表达式来选择你想要的实体。以下代码与前面的配置代码等效：

[PRE14]

`NamedTypeSelector` 的第一个参数是选择器名称——在这个例子中是 `MySelectorName`。选择器名称是任意的，并且可以在以后用来在选择器列表中查找或删除选择器。通常你不会使用它；只需给它一个独特的名称。`NamedTypeSelector` 的第二个参数接受一个表达式。它为你提供一个实体 `type` 并等待 `true` 或 `false`。如果你想为给定的实体类型启用实体历史记录，则返回 `true`。因此，你可以传递一个如 `type => type.Namespace.StartsWith("MyRootNamespace")` 的表达式来选择所有具有命名空间的实体。你可以添加你需要的任意数量的选择器。所有选择器都会被测试。如果其中任何一个返回 `true`，则实体将被选中以记录属性更改。

除了这些全局选项和选择器之外，还有方法可以按类、方法和属性级别启用/禁用审计日志。

## 详细禁用和启用审计日志

当你使用审计日志系统时，通常你想要记录每一次访问。然而，在某些情况下，你可能想要禁用某些特定操作或实体的审计日志。以下是一些可能的原因：操作参数可能对写入日志是危险的（例如，它可能包含用户的密码），操作调用或实体更改可能超出用户控制，因此不值得为了审计目的记录，或者操作可能是一个大量操作，它写入过多的审计日志并降低性能。

ABP 定义了 `[DisableAuditing]` 和 `[Audited]` 属性来声明性地控制记录的对象。你可以控制审计日志的两个目标：服务调用和实体历史记录。

### 控制服务调用的审计日志

应用服务方法、Razor 页面处理程序以及类或方法级别的 `[DisableAuditing]` 属性。

以下示例在应用服务类上使用了 `[DisableAuditing]` 属性：

[PRE15]

使用这种用法，ABP 不会将这些方法的执行包括在审计日志对象中。如果你只想禁用其中一个方法，你可以在方法级别使用它：

[PRE16]

在这种情况下，`CreateAsync` 方法调用不包括在审计日志中，而 `DeleteAsync` 方法调用被写入审计日志对象。同样的行为可以使用以下代码实现：

[PRE17]

我禁用了所有方法（除了 `DeleteAsync` 方法），因为 `DeleteAsync` 方法声明了 `[Audited]` 属性。

`[Audited]`属性可以用于任何类（与DI系统一起使用）以在该类上启用审计日志，即使该类默认不进行审计日志记录。此外，您可以在任何类的任何方法中使用它，只为该特定方法调用启用它。如果您在类上使用`[Audited]`属性，然后可以使用`[DisableAuditing]`属性禁用特定方法。

当ABP在审计日志对象中包含方法调用信息时，它也会包含执行方法的所有参数。这对于了解您的系统中哪些更改非常有用；然而，在某些情况下，您可能想排除输入的一些属性。考虑一个场景，您从用户那里获取信用卡信息。您可能不想将其包含在审计日志中。在这种情况下，您可以在输入对象的任何属性上使用`[DisableAuditing]`属性。请参阅以下示例，它从`Dto`输入的属性中排除了审计日志：

[PRE18]

对于此示例，ABP不会将`CreditCardNumber`值写入审计日志。

禁用方法调用审计日志不会影响实体历史记录。如果一个实体被更改并且它被选中进行审计日志记录，更改仍然会被记录。下一节将解释如何控制实体历史记录的审计日志系统。

### 控制实体历史记录的审计日志

在*启用实体历史记录*部分，我们看到了如何通过定义选择器来为一个或多个实体启用实体历史记录。然而，如果您只想为单个实体启用实体历史记录，有一个替代且更简单的方法：只需在您的实体类上方添加`[Audited]`属性：

[PRE19]

在此示例中，我向`Order`实体添加了`[Audited]`属性，以配置审计日志系统为该实体启用实体历史记录。

假设您已使用选择器为许多或所有实体启用了实体历史记录，但想为特定实体禁用它们。在这种情况下，您可以使用该实体类的`[DisableAuditing]`属性。

`[DisableAuditing]`属性也可以用于实体的属性，以排除此属性从审计日志中，如下例所示：

[PRE20]

对于那个例子，ABP不会将`CreditCardNumber`值写入审计日志。

### 存储审计日志

ABP框架的核心设计是通过在需要接触数据源的地方引入抽象，不假设任何数据存储。审计日志系统也不例外。它定义了`IAuditingStore`接口来抽象审计日志对象保存的位置。该接口只有一个方法：

[PRE21]

您可以实现此接口以将审计日志保存到您想要的位置。如果您使用ABP的启动模板创建新解决方案，它已配置为将审计日志保存到应用程序的主要数据库中，因此您通常不需要手动实现`IAuditingStore`接口。

我们已经看到了控制和管理审计日志系统的方法。审计日志是企业系统跟踪和记录系统更改的必要系统。下一节将介绍缓存系统，这是Web应用程序的另一个基本功能。

# 缓存数据

缓存是提高应用程序性能和可伸缩性的最基本系统之一。ABP扩展了ASP.NET Core的**分布式缓存**系统，并使其与ABP框架的其他功能兼容，如多租户。

如果您运行多个应用程序实例或拥有分布式系统，如微服务解决方案，分布式缓存是必不可少的。它提供了不同应用程序之间的一致性，并允许共享缓存值。分布式缓存通常是一个外部独立的应用程序，例如Redis和Memcached。

即使您的应用程序只有一个运行实例，也建议使用分布式缓存系统。无需担心性能，因为分布式缓存的默认实现是在内存中工作的。这意味着除非您明确配置一个真实的分布式缓存提供者，如Redis，否则它不是分布式的。

ASP.NET Core中的分布式缓存

本节重点介绍ABP的缓存功能，并不涵盖所有ASP.NET Core的分布式缓存系统功能。您可以参考Microsoft的文档来了解有关ASP.NET Core中分布式缓存的更多信息：[https://docs.microsoft.com/en-us/aspnet/core/performance/caching/distributed](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/distributed)。

在本节中，我将向您展示如何使用`IDistributedCache<T>`接口，配置选项，并处理错误处理和批量操作。我们还将了解如何使用Redis作为分布式缓存提供者。最后，我将讨论如何使缓存值无效。

让我们从基础开始——`IDistributedCache<T>`接口。

## 使用IDistributedCache<T>接口

ASP.NET Core定义了一个`IDistributedCache`接口，但它不是类型安全的。它设置和获取`byte`数组而不是对象。ABP的`IDistributedCache<T>`接口，另一方面，被设计为泛型，具有类型安全的参数方法（`T`代表存储在缓存中的项的类型）。它内部使用标准的`IDistributedCache`接口，以确保与ASP.NET Core的缓存系统100%兼容。ABP的`IDistributedCache<T>`接口有两个主要优势，如下所示：

+   自动将对象序列化和反序列化为`byte`数组。因此，您无需处理序列化和反序列化。

+   它自动将缓存名称前缀添加到缓存键中，以便可以使用相同的键用于不同类型的缓存对象。

使用`IDistributedCache<T>`接口的第一步是定义一个类来表示缓存中的项。我已经定义了以下类来在缓存中存储用户信息：

[PRE22]

这是一个普通的 C# 类。唯一的限制是它应该是可序列化的，因为它在保存到缓存时被序列化为 JSON，在从缓存读取时被反序列化（例如，不要添加引用到其他对象，这些对象不应该或不能存储在缓存中；保持简单）。

一旦我们定义了缓存项类，我们就可以注入 `IDistributedCache<T>` 接口，如下面的代码块所示：

[PRE23]

我已注入了 `IDistributedCache<UserCacheItem>` 服务以处理 `UserCacheItem` 对象的分布式缓存。以下代码块显示了我们可以如何使用它来获取缓存的用户信息，如果给定的用户未在缓存中找到，则回退到数据库查询：

[PRE24]

我已向 `GetOrAddAsync` 方法传递了三个参数：

+   第一个参数是缓存键，它应该是一个字符串值，因此我将 `Guid` `userId` 值转换为字符串值。

+   第二个参数是一个工厂方法，如果给定的键未在缓存中找到，则会执行。我在这里传递了 `GetUserFromDatabaseAsync` 方法。在该方法中，您应从其数据源构建缓存项。

+   最后一个参数是一个工厂方法，它返回一个 `DistributedCacheEntryOptions` 对象。这是可选的，并配置了缓存项的过期时间。只有当 `GetOrAddAsync` 方法添加条目时，才会调用工厂方法。

缓存键默认为 `string` 数据类型。然而，ABP 定义了另一个接口 `IDistributedCache<TCacheItem, TCacheKey>`，允许您指定缓存键，这样您就不需要手动将您的键转换为 `string` 数据类型。我们可以注入 `IDistributedCache<UserCacheItem, Guid>` 服务，并移除此示例中第一个参数的 `ToString()` 使用。

`DistributedCacheEntryOptions` 提供以下选项来控制缓存项的生存周期：

+   `AbsoluteExpiration`：您可以设置一个绝对时间，就像我们在本例中所做的那样。在该时间，项目将从缓存中自动删除。

+   `AbsoluteExpirationRelativeToNow`：设置绝对过期时间的另一种方法。我们可以将本例中的选项重写为 `AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1)`。结果将是相同的。

+   `SlidingExpiration`：设置缓存项在移除之前可以不活跃（未访问）多长时间。这意味着如果您继续访问缓存项，则过期时间会自动延长。

如果您未传递过期时间参数，则使用默认值。您可以使用下一节中解释的 `AbpDistributedCacheOptions` 类配置默认值和一些其他全局选项。在此之前，让我们看看 `IDistributedCache<UserCacheItem>` 服务的其他方法，如下所示：

+   `GetAsync` 用于使用缓存键从缓存中读取数据。

+   `SetAsync` 用于将项保存到缓存中。如果可用，它将覆盖现有值。

+   `RefreshAsync` 用于重置给定键的滑动过期时间。

+   `RemoveAsync` 用于从缓存中删除一个项。

    关于同步缓存方法

    所有方法也有同步版本，例如 `GetAsync` 方法的 `GET` 方法。然而，建议尽可能使用异步版本。

这些方法是 ASP.NET Core 的标准方法。ABP 为每个方法添加了处理多个项的方法，例如 `GetManyAsync` 对应 `GetAsync`。如果你有很多项需要读取或写入，使用 `Many` 方法会有显著的性能提升。`GetOrAddAsync` 方法（在本节中 `GetUserInfoAsync` 示例中使用）也是由 ABP 框架定义的，用于安全地读取缓存值，回退到原始数据源，并在单个方法调用中设置缓存值。

## 配置缓存选项

`AbpDistributedCacheOptions` 是配置缓存系统的主要选项类。你可以在模块类的 `ConfigureServices` 方法中配置它（你可以在领域层或应用层中这样做），如下所示：

[PRE25]

我已在此代码块中将 `GlobalCacheEntryOptions` 属性配置为将默认缓存过期时间设置为 `2` 小时。

`AbpDistributedCacheOptions` 还有一些其他属性，如下所述：

+   `KeyPrefix` (`string`; 默认: `null`): 添加到该应用程序所有缓存键开头的前缀值。此选项可用于在使用多个应用程序共享的分布式缓存时隔离你的应用程序的缓存项。

+   `hideErrors` (`bool`; 默认: `true`): 用于控制缓存服务方法上错误处理的默认值。

正如你在前面的示例中所看到的，这些选项可以通过传递参数到 `IDistributedCache` 服务的 `GetAsync` 方法的参数来覆盖。

## 错误处理

当我们使用外部进程（如 Redis）进行分布式缓存时，在从缓存中读取数据或写入数据时可能会遇到问题。缓存服务器可能离线，或者我们可能遇到暂时的网络问题。这些临时问题通常可以忽略，尤其是在尝试从缓存中读取数据时。如果缓存服务当前不可用，你可以安全地尝试从原始数据源读取。这可能会慢一些，但比抛出异常并使当前请求失败要好。

所有 `IDistributedCache<T>` 方法都有一个可选的 `hideErrors` 参数来控制异常处理行为。如果你传递 `false`，则所有异常都会抛出。如果你传递 `true`，则 ABP 隐藏与缓存相关的错误。如果你没有指定值，则使用上一节中解释的默认值。

## 在多租户应用程序中使用缓存

如果你的应用程序是多租户的，ABP 会自动将当前租户的 ID 添加到缓存键中，以区分不同租户的缓存值。这样，它提供了租户之间的隔离。

如果您想创建一个在租户之间共享的缓存，您可以使用 `[IgnoreMultiTenancy]` 属性为缓存项类，如下面的代码块所示：

[PRE26]

对于这个例子，`MyCacheItem` 的值可以被不同的租户访问。

## 使用 Redis 作为分布式缓存提供者

Redis 是一个流行的工具，用作分布式缓存。ASP.NET Core 为 Redis 提供了一个缓存集成包。您可以通过遵循 Microsoft 的文档（[https://docs.microsoft.com/en-us/aspnet/core/performance/caching/distributed](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/distributed)）来使用它，并且它运行得非常好。

ABP 还提供了一个 Redis 集成包，它扩展了 Microsoft 的集成以支持批量操作（如 *Using the IDistributedCache<T> interface* 部分中提到的 `GetManyAsync`）。因此，建议使用 ABP 的集成 `Volo.Abp.Caching.StackExchangeRedis` NuGet 包来使用 Redis 作为缓存提供者。您可以使用以下命令在您想要使用的项目的目录中使用 ABP **命令行界面**（**CLI**）来安装它：

[PRE27]

安装完成后，您只需将配置添加到 `appsettings.json` 文件中，以连接到 Redis 服务器，如下所示：

[PRE28]

您将服务器地址和端口（一个连接字符串）写入到 `Configuration` 选项中。请参阅 Microsoft 的文档以获取配置的详细信息：[https://docs.microsoft.com/en-us/aspnet/core/performance/caching/distributed](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/distributed)。

## 缓存值失效

有一个流行的说法，缓存失效是计算机科学中两个难题之一（另一个是命名事物）。缓存的值通常是原始数据的一个副本，原始数据位于经常需要读取且成本较高的位置，或者是一个计算成本较高的值。在这种情况下，它可以提高性能和可伸缩性，但当原始数据发生变化并使缓存值过时时，问题就开始了。我们应该仔细观察这些变化，并从缓存中删除或刷新相关数据。这被称为缓存失效。

缓存失效过程在很大程度上取决于缓存数据和您的应用程序逻辑。然而，有一些特定的情况，ABP 可以帮助您失效缓存数据。

一个具体的情况是我们可能希望在实体发生变化（被更新或删除）时使缓存项失效。对于这种情况，我们可以注册 ABP 框架发布的事件。以下代码在相关用户实体发生变化时使用户缓存项失效：

[PRE29]

`MyUserService`注册了一个`EntityChangedEventData<IdentityUser>`本地事件。当创建一个新的`IdentityUser`实体或更新或删除现有的`IdentityUser`实体时，将触发此事件。在这种情况下，会调用`HandleEventAsync`方法，并将相关实体在`data.Entity`属性中。此方法简单地从缓存中删除具有更改实体`Id`值的用户。

本地事件在当前进程中工作。这意味着处理类（这里为`MyUserService`）应该与实体变更在同一个进程中。

关于事件总线系统

本地和分布式事件是ABP框架的有趣特性，但本书中没有包括。如果你想了解更多关于它们的信息，请参阅ABP文档：[https://docs.abp.io/en/abp/latest/Event-Bus](https://docs.abp.io/en/abp/latest/Event-Bus)。

在本节中，我们学习了如何与分布式缓存系统协同工作，配置选项，并处理错误处理。我们还介绍了Redis缓存提供程序的安装。最后，我们介绍了可以帮助我们使缓存值无效的自动ABP事件。

下一个部分将涉及UI本地化，这是我在本章中将要介绍的ABP的最后一个功能。

# 本地化用户界面

如果你正在构建一个全球产品，你可能希望根据当前用户的语言显示本地化的UI。ASP.NET Core提供了一个系统来本地化你的应用程序的UI。ABP增加了一些有用的功能和约定，使其更加容易和灵活。

本节解释了如何定义你想要支持的语言，为不同的语言创建文本，并获取当前用户的正确文本。你将了解本地化资源概念和嵌入式本地化资源文件。

我们可以先定义你的应用程序支持的语言。

## 配置支持的语言

关于本地化的第一个问题是这样的：*你希望在UI上支持哪些语言？* ABP提供了一个简单的配置来定义语言，使用`AbpLocalizationOptions`，如下面的代码块所示：

[PRE30]

你可以将这段代码写入你的模块类的`ConfigureServices`方法中。实际上，当你使用ABP应用程序启动模板创建新解决方案时，这个配置（以及许多语言）已经完成了。你只需根据需要编辑列表即可。

`LanguageInfo`构造函数接受几个参数：

+   `cultureName`：语言的文化名称（代码），在运行时设置为`CultureInfo.CurrentCulture`。

+   `uiCultureName`：语言的UI文化名称（代码），在运行时设置为`CultureInfo.CurrentUICulture`。

+   `displayName`：在用户选择此语言时显示的语言名称。建议用其原始语言书写该名称。

+   `flagIcon`：一个字符串值，UI可以使用它来在语言名称附近显示国家国旗。

ABP 根据当前的 HTTP 请求确定这些语言之一。

## 确定当前语言

ABP 通过使用 `AbpRequestLocalizationMiddleware` 类来确定当前语言。这是一个 ASP.NET Core 中间件，通过以下代码行添加到 ASP.NET Core 请求管道中：

[PRE31]

当请求通过此中间件时，将选择配置的语言之一并将其设置为 `CultureInfo.CurrentCulture` 和 `CultureInfo.CurrentUICulture`。这是 .NET 中设置和获取当前文化定位的标准系统。

当前语言的选择基于以下 HTTP 请求参数，按照以下优先级顺序：

1.  如果设置了 `culture` 查询字符串参数，它将用于确定当前语言。一个例子是 `http://localhost:5000/?culture=en-US`。

1.  如果设置了 `.AspNetCore.Culture` cookie 的值，则它将用作当前语言。

1.  如果设置了 `Accept-Language` HTTP 头，它将用作当前语言。浏览器通常默认发送最后一个。

    关于 ASP.NET Core 的本地化系统

    本节中解释的行为是默认行为。然而，ASP.NET Core 的语言确定系统更加灵活和可定制。请参阅 Microsoft 的文档以获取更多信息：[https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization).

在定义我们想要支持的语言之后，我们可以定义我们的本地化资源。

## 定义本地化资源

ABP 与 ASP.NET Core 的本地化系统 100% 兼容。因此，你可以通过遵循 Microsoft 的文档使用 `.resx` 文件作为本地化资源：[https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization)。然而，ABP 提供了一种轻量级、灵活且可扩展的方式来定义本地化文本，使用简单的 JSON 文件。

当你使用 ABP 启动模板创建一个新的解决方案时，`Domain.Shared` 项目包含应用程序的本地化资源以及本地化 JSON 文件：

![图 8.2 – 本地化资源和本地化 JSON 文件

](img/Figure_8.2_B17287.jpg)

图 8.2 – 本地化资源和本地化 JSON 文件

对于此示例，`DemoAppResource` 类代表本地化资源。一个应用程序可以拥有多个本地化资源，每个资源定义其自己的 JSON 文件。你可以将本地化资源视为一组本地化文本。它有助于构建模块化系统，其中每个模块都有自己的本地化资源。

本地化资源类是一个空类，如下面的代码所示：

[PRE32]

当您想要在该本地化资源中使用文本时，此类引用相关资源。`LocalizationResourceName`属性为资源设置一个字符串名称。每个本地化资源都有一个唯一的名称，该名称在客户端代码中用于引用资源。我们将在*在客户端使用本地化*部分中探讨客户端本地化。

应用程序的默认本地化资源

在您的应用程序中，通常只有一个（默认）本地化资源，该资源在创建新的ABP解决方案时随启动模板一起提供。默认本地化资源类的名称以项目名称开头——例如，如果您将项目名称指定为`ProductManagement`，则类名为`ProductManagementResource`。

一旦我们有了本地化资源，我们就可以为每个我们支持的语言创建一个JSON文件。

## 与本地化JSON文件一起工作

本地化文件是一个简单的JSON格式文件，如下面的代码块所示：

[PRE33]

该文件中有两个主要的根元素，如下所述：

+   `culture`：相关语言的区域代码。它与在*配置支持的语言*部分中引入的区域代码相匹配。

+   `texts`：包含本地化文本的键值对。键用于访问本地化文本，应在所有不同语言的JSON文件中相同。值是当前文化（语言）的本地化文本。

在为每种语言定义了本地化文本之后，我们可以在运行时请求本地化文本。

## 获取本地化文本

ASP.NET Core定义了一个`IStringLocalizer<T>`接口，用于获取当前文化的本地化文本，其中`T`代表本地化资源类。您可以将该接口注入到您的类中，如下面的代码块所示：

[PRE34]

在前面的代码块中，`LocalizationDemoService`类注入了`IStringLocalizer<DemoAppResource>`服务，该服务用于访问`DemoAppResource`类的本地化文本。在`GetWelcomeMessage`方法中，我们简单地获取`WelcomeMessage`键的本地化文本。如果当前语言是英语，它将返回我们在上一节中定义的JSON文件中的`Welcome to the application.`。

我们可以在本地化文本时传递参数。

### 参数化文本

本地化文本可以包含参数，如下面的示例所示：

[PRE35]

可以将参数传递给本地化器，如下面的代码块所示：

[PRE36]

本例中给出的名称替换了`{0}`占位符。

### 回退逻辑

当请求的文本在当前文化的JSON文件中找不到时，本地化系统会使用父级或默认文化进行回退。

例如，假设您请求获取`WelcomeMessage`文本，而当前区域代码（`CultureInfo.CurrentUICulture`）为`de-DE`（德国-德国）。在这种情况下，以下情况之一会发生：

+   如果您没有定义具有 `"culture": "de-DE"` 的 JSON 文件，或者您已经定义了一个 JSON 文件但它不包含 `WelcomeMessage` 键，那么它将回退到父文化（`"de"`），尝试在该文化中找到给定的键，如果可用则返回它。

+   如果在父文化中找不到，它将回退到本地化资源的默认文化（请参阅下一节以配置默认文化）。

+   如果在默认文化中找不到，则返回给定的键（例如本例中的 `WelcomeMessage`）作为响应。

## 配置本地化资源

在使用之前，应将本地化资源添加到 `AbpLocalizationOptions` 中。此配置已在启动模板中完成，如下所示：

[PRE37]

本地化 JSON 文件通常定义为嵌入式资源。我们正在配置 ABP 的虚拟文件系统（使用 `AbpVirtualFileSystemOptions`），将此程序集中的所有嵌入式文件添加到虚拟文件系统中，以便本地化文件也被添加。

然后，在第二部分中，我们将 `DemoAppResource` 添加到 `Resources` 字典中，以便 ABP 能够识别它。在这里，`"en"` 参数设置了该本地化资源的默认文化。

ABP 的本地化系统相当先进。它允许您通过从另一个本地化资源继承本地化资源来重用本地化资源的文本。在本例中，我们正在继承 `AbpValidationResource`，它由 ABP 框架定义，并包含标准的验证错误消息。

`AddVirtualJson` 方法用于通过虚拟文件系统设置与该资源相关的 JSON 文件。

最后，`DefaultResourceType` 设置了该应用程序的默认本地化资源。您可以在未指定本地化资源的地方使用默认资源。下一节将解释此配置的主要使用点。

## 在特殊服务中进行本地化

在每个地方注入 `IStringLocalizer<T>` 服务可能会很繁琐。ABP 预先将这些本地化器注入到一些特殊的基类中。当您从这些类继承时，您可以直接使用 `L` 短路属性来本地化文本。

以下示例展示了如何在应用程序服务方法中本地化文本：

[PRE38]

在本例中，`L` 属性由 `ApplicationService` 基类定义，因此您不需要手动注入 `IStringLocalizer<T>` 服务。您可能会想知道，因为我们没有指定本地化资源，这里使用的是哪一个。答案是上一节中解释的 `DefaultResourceType` 选项。

如果您想为特定应用程序服务指定另一个本地化资源，则请在服务的构造函数中设置 `LocalizationResource` 属性：

[PRE39]

除了 `ApplicationService` 类之外，一些其他常见的基类，如 `AbpController` 和 `AbpPageModel`，也提供了相同的 `L` 属性，作为注入 `IStringLocalizer<T>` 服务的快捷方式。

## 在客户端使用本地化

ABP 的本地化系统的一个优点是所有本地化资源都可以直接在客户端代码中使用。

例如，以下代码在 ASP.NET Core MVC/Razor Pages 应用程序的 JavaScript 代码中将 `WelcomeMessage` 键本地化：

[PRE40]

`DemoApp` 是本地化资源名称，而 `WelcomeMessage` 是这里的本地化键。客户端本地化将在本书的 *第 4 部分*，*用户界面和 API 开发* 中介绍。

# 摘要

在本章中，我们学习了您几乎在所有 Web 应用程序中都需要的一些基本功能。

`ICurrentUser` 服务允许您获取您应用程序中当前用户的信息。您可以使用标准声明（例如用户名和 ID）并根据您的需求定义自定义声明。

我们探讨了数据过滤系统，该系统在从数据库查询数据时自动过滤数据。这样，我们可以轻松实现一些模式，如软删除和多租户。我们还学习了如何定义自定义数据过滤器，并在必要时禁用过滤器。

我们已经了解了审计日志系统是如何跟踪和保存用户执行的所有操作的。我们可以通过属性和选项声明性和约定性地控制审计日志系统。

缓存数据是提高系统性能和可扩展性的另一个基本概念。我们学习了 ABP 的 `IDistributedCache<T>` 服务，它提供了一种类型安全的方式与缓存提供程序交互，并自动化了一些常见任务，例如序列化和异常处理。

最后，我们探讨了 ASP.NET Core 和 ABP 框架的本地化基础设施，以便我们可以在应用程序中轻松定义和消费本地化文本。

现在我们已经到达本章的结尾，我们已经完成了本书的 *第 2 部分*，*ABP 框架基础*，涵盖了 ABP 框架和 ASP.NET Core 基础设施的基本知识。下一部分是使用 ABP 框架实现 **领域驱动设计**（**DDD**）的实用指南。DDD 是 ABP 基于的核心概念之一。它包括构建可维护业务解决方案的原则、模式和最佳实践。
