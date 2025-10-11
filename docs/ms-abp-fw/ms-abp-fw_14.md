# 第十一章：*第十一章*：DDD – 应用层

前一章详细解释了领域层构建块。领域层用于实现解决方案的核心、与应用程序无关的领域逻辑。然而，我们还需要一些应用程序与该领域逻辑进行交互，例如一个 Web 或移动应用程序。应用层负责实现这些应用程序的业务逻辑，而不依赖于表示层中使用的**用户界面**（**UI**）技术。我们通过将应用服务封装起来，将领域层与表示技术隔离开来。

在本章中，我们将学习如何使用 ABP 框架设计和实现应用服务和**数据传输对象**（**DTOs**）。我们还将了解领域层和应用层职责之间的区别。

本章涵盖了以下主题：

+   实现应用服务

+   设计 DTOs

+   理解各层的职责

# 技术要求

您可以从 GitHub 克隆或下载*EventHub*项目的源代码：[`github.com/volosoft/eventhub`](https://github.com/volosoft/eventhub)。

如果您想在本地开发环境中运行解决方案，您需要一个**集成开发环境**（**IDE**）/编辑器（例如 Visual Studio）来构建和运行 ASP.NET Core 解决方案。此外，如果您想创建 ABP 解决方案，您需要安装 ABP **命令行界面**（**CLI**），如*第二章*，*开始使用 ABP 框架*中所述。

# 实现应用服务

应用服务是一个无状态的类，由表示层使用以执行应用程序的用例。它协调领域对象以实现业务操作。应用服务获取和返回 DTO 而不是实体。

应用服务方法被视为一个工作单元（意味着所有数据库操作——要么全部成功，要么全部失败作为一个组，如在第*第六章*，*与数据访问基础设施一起工作*中所述），ABP 框架会自动处理。一个典型应用服务方法的流程包括以下步骤：

1.  使用输入参数和当前上下文从仓储获取必要的聚合。

1.  通过协调聚合、领域服务和其他领域对象，并委托工作给它们来实现用例。

1.  使用仓储更新数据库中已更改的聚合。

1.  可选地，向客户端返回一个结果 DTO（通常，返回给表示层）。

    关于更新更改对象

    实际上，如果你使用 **Entity Framework Core** (**EF Core**)，则 *步骤 3* 是不必要的，因为 EF Core 有一个更改跟踪系统，可以自动确定更改的对象，并在 **工作单元** (**UoW**) 的末尾将它们更新到数据库中。所以，如果你没有问题依赖 EF Core 的功能，你可以跳过 *步骤 3*。

让我们看看以下示例中的 `AddSessionAsync` 应用服务方法：

```cs
public class EventAppService : ApplicationService,
    IEventAppService
{
    ...
    [Authorize]
    public async Task AddSessionAsync(Guid id,
                                      AddSessionDto input)
    {
        var @event = await _eventRepository.GetAsync(id);
        @event.AddSession(
            input.TrackId, GuidGenerator.Create(),
            input.Title,
            input.StartTime, input.EndTime,
            input.Description, input.Language
        );
        await _eventRepository.UpdateAsync(@event);
    }
}
```

此方法是一个简单的应用服务方法。它用于向事件添加一个新的会话。它首先从数据库中获取相关的 `Event` 聚合。然后，它使用 `Event` 类的 `AddSession` 方法将实际的业务操作委派给域层。最后，它更新数据库中的更改后的 `Event` 对象。我们将在本章的 *设计 DTOs* 部分看到 `AddSessionDto` 类。

让我们看看一个更复杂的例子，如下创建一个新事件：

```cs
[Authorize]
public async Task<EventDto> CreateAsync(CreateEventDto 
                                        input)
{
    var organization = await _organizationRepository
        .GetAsync(input.OrganizationId);
    if (organization.OwnerUserId != CurrentUser.GetId())
    {
        throw new AbpAuthorizationException(
        L[“EventHub:
          NotAuthorizedToCreateEventInThisOrganization”,
          organization.DisplayName]
        );
    }
    var @event = await _eventManager.CreateAsync(
        organization, input.Title,
        input.StartTime, input.EndTime, input.Description);
    await _eventManager.SetLocationAsync(@event,
        input.IsOnline, input.OnlineLink, input.CountryId,
        input.City);
    await _eventManager.SetCapacityAsync(@event,
                                         input.Capacity);
    @event.Language = input.Language;
    if (input.CoverImageContent != null &&
        input.CoverImageContent.Length > 0)
    {
        await SaveCoverImageAsync(
            @event.Id, input.CoverImageContent);
    }
    await _eventRepository.InsertAsync(@event);
    return ObjectMapper.Map<Event, EventDto>(@event);
}
```

`CreateAsync` 方法从 UI 层获取一个 `CreateEventDto` 对象，该对象携带新事件数据，是创建新实体的一个好例子。让我们调查它是如何实现的。

它首先从数据库中获取 `organization` 对象，并比较其所有者的 `AbpAuthorizationException` 异常，如果用户不满足条件。授权是应用层的责任，不同的应用中授权规则可能不同。例如，在管理应用中，管理员用户可以代表任何用户创建事件，而不检查组织的所有权。

`CreateAsync` 方法随后使用 `eventManager` 域服务创建一个新的 `Event` 对象，并带有所需的最小属性。

我们已经创建了一个 `Event` 对象，但我们的工作还没有完成。`CreateEventDto` 有一些可选属性，用户可能设置。我们再次使用 `eventManager` 域服务来设置事件的地点。然后，我们直接设置 `Event` 的 `Language` 属性，因为没有业务规则来设置它；它有一个公共设置器，其值甚至可以是 `null`。

`CreateAsync` 方法继续使用 `eventManager` 类通过检查核心域规则来设置事件容量。如果提供了事件封面图像，它也会保存该图像。

到那时，`Event` 对象还没有保存到数据库中。所有操作都是在内存对象上执行的。域服务不会将更改保存到数据库中，因为这是应用层的责任。如果域服务方法保存了它们的更改，我们最终会在数据库中得到一个插入操作和三个更新操作。根据当前实现，`CreateAsync` 方法使用存储库的 `InsertAsync` 方法，并在方法末尾通过单个数据库操作保存对象。

如您在示例应用程序服务定义中看到的，应用程序服务方法使用 DTO 类从上层（通常是表示层）获取数据，并将数据返回到上层。下一节介绍了 DTO 设计考虑因素和最佳实践。

# 设计 DTO

DTO 是一个简单的对象，用于在表示层和应用层之间传输数据。让我们先看看设计 DTO 类的基本原则。

## 设计 DTO 类

在定义 DTO 类时，有一些基本的原则需要遵循，如下所述：

+   DTOs 不应该包含任何业务逻辑；它们只是用于数据传输。

+   DTO 对象应该是可序列化的，因为它们大多数时候都是通过网络传输的。通常，它们有一个无参构造函数，并且所有属性都有公共的 getter 和 setter。

+   DTO 类不应该从实体继承，也不应该使用实体类型作为它们的属性。

以下 DTO 类用于在现有事件的轨迹中添加新会话时存储数据：

```cs
public class AddSessionDto
{
    [Required]
    [StringLength(SessionConsts.MaxTitleLength,
        MinimumLength = SessionConsts.MinTitleLength)]
    public string Title { get; set; }
    [Required]
    [StringLength(SessionConsts.MaxDescriptionLength,
        MinimumLength = 
            SessionConsts.MinDescriptionLength)]
    public string Description { get; set; }
    public Guid TrackId { get; set; }
    public DateTime StartTime { get; set; }
    public DateTime EndTime { get; set; }
    public string Language { get; set; }
}
```

`AddSessionDto`类没有方法，因此没有业务逻辑。它所有的属性都有公共的 getter 和 setter。`AddSessionDto`类没有定义任何构造函数，因此它有一个隐式的公共无参构造函数。

`AddSessionDto`类的`Title`和`Description`属性有验证属性，例如`Required`和`StringLength`。

下一节讨论了验证输入 DTO。

## 验证输入 DTO

当 DTO 对象作为应用程序服务方法参数使用时，有几种方式可以验证 DTO 对象，如下所述：

+   我们可以使用数据注释属性，例如`Required`、`StringLength`和`Range`。

+   我们可以为 DTO 类实现`IValidatableObject`接口，并在`Validate`方法中执行额外的验证逻辑。

+   我们可以使用第三方库来验证 DTO 对象。例如，ABP 集成了`FluentValidation`库，将验证逻辑从 DTO 类中分离出来，并执行高级验证逻辑。

无论您采用哪种方法（您可以为 DTO 类同时使用所有方法），ABP 都会自动检查这些验证规则，并在值无效的情况下抛出验证异常。因此，您的应用程序服务方法始终使用有效的 DTO 对象执行。有关 ABP 验证基础设施的所有详细信息，请参阅*第七章*的*验证用户输入*部分，*探索横切关注点*。

在 DTO 类（或 `FluentValidation` 验证器类中）上的验证逻辑应该是正式验证。这意味着你可以检查给定的输入是否提供并且格式正确。然而，它不应包含领域验证。例如，不要尝试检查给定的开始和结束日期是否与同一轨道上的另一个会话冲突。此类验证逻辑应在领域层实现，通常在实体或领域服务类中。

与 DTO 相关的另一个常见任务是将其映射到其他对象。

## 对象到对象映射

我们在领域和应用层内部使用实体，并使用 DTO 与上层进行通信。这种方法使我们创建与实体类相似的 DTO 类，并将实体对象转换为 DTO 对象。如果实体类只有少量属性，则可以通过逐个复制属性手动创建相应的 DTO 对象。然而，实体类会随着时间的推移而增长，编写和维护手动映射代码变得繁琐且容易出错。

ABP 框架提供了一个 `IObjectMapper` 服务，用于将相似的对象相互转换。请参阅以下应用程序服务方法：

```cs
public async Task<EventDto> GetAsync(Guid id)
{
    Event eventEntity = 
        await _eventRepository.GetAsync(id);
    return ObjectMapper.Map<Event, EventDto>(eventEntity);
}
```

此方法简单地通过使用 `IObjectMapper` 服务将 `Event` 对象转换为 `EventDto` 对象来返回一个 `EventDto` 对象。`EventDto` 有很多属性，手动创建它会导致代码块变长。`IObjectMapper` 是一个抽象，在创建新的 ABP 解决方案时使用 `AutoMapper` 库实现。如果你想要使用前面的代码，你应该首先定义 `AutoMapper` 映射配置。

对象到对象映射文档

对象到对象映射这一主题并未包含在本书中。然而，在创建示例应用程序时，我们确实在 *第三章*，*逐步应用开发* 中使用了它。请参考 ABP 的文档以全面了解对象到对象映射系统：[`docs.abp.io/en/abp/latest/Object-To-Object-Mapping`](https://docs.abp.io/en/abp/latest/Object-To-Object-Mapping)。

虽然使用对象映射器相当简单，但我们仍应谨慎使用。对象映射库主要依赖于命名约定。它们会自动映射同名属性，而我们可以手动配置映射。

当你重构实体但未更新相应的 DTO 或映射代码时，可能会出现一个问题。`AutoMapper` 库有一个名为配置验证的概念。它在应用程序启动时验证映射配置，并在检测到映射配置问题时抛出异常。我建议为你的应用程序启用它。有关配置验证的更多信息，请参阅 `AutoMapper` 文档：[`docs.automapper.org/en/stable/Configuration-validation.html`](https://docs.automapper.org/en/stable/Configuration-validation.html)。

当你将实体映射到 DTO 时，对象到对象的映射非常有用。然而，不要将输入 DTO 映射到实体。这个建议背后有一些技术和设计原因，如下所述：

+   你还记得上一章的 *实现实体构造函数* 部分吗？实体类通常具有主构造函数来获取所需的属性并创建一个有效的实体。自动映射操作通常需要在目标类上有一个空构造函数，因此映射会失败。

+   实体上的一些属性设计为具有私有设置器。你应该使用实体方法来更改这些属性值以应用一些业务规则。直接从 DTO 对象复制它们的值可能会违反业务规则。

+   你应该仔细验证和处理用户输入，而不是盲目地将它们映射到实体上。

在 *实现应用程序服务* 节中解释的 `CreateAsync` 方法是使用输入 DTO 创建实体的一个好例子。它不将 DTO 映射到实体，而是使用领域服务创建一个有效的实体并设置可选属性。

在下一节中，我们将讨论一些 DTO 的设计实践。

## DTO 设计最佳实践

创建 DTO 在一开始看起来很简单——它们确实是简单的。然而，一旦应用程序增长，你将有许多 DTO 类，了解如何组织这些类变得很重要。在接下来的几节中，我将提供一些关于 DTO 的建议，以使你的代码库更易于维护和更少出错。

### 不要在输入 DTO 中定义未使用的属性

谁会在输入 DTO 中定义一个未使用的属性，对吧？不幸的是，事实并非如此。我见过很多代码库存在这个问题。

在应用程序服务方法中未使用的 DTO 类属性是混淆使用该应用程序服务方法的开发者的完美方式，并可能导致构建有缺陷的代码库。

在 DTO 类中未使用的属性的一个可能原因是，它曾经被使用过，但应用程序服务方法已更改，开发者忘记将其删除。我们应该关注这一点，并始终删除未使用的属性。如果你关心向后兼容性，因为你不是构建客户端应用程序的人，那么请在该属性上声明 `[过时]` 属性，记录破坏性更改，并在提供值的情况下尝试保留旧行为。

如果你违反了下一节中的规则，未使用的属性可能是不可避免的。

### 不要重用输入 DTO

当你有太多的应用程序服务方法时，你可能会认为使用一些 DTO 为多个应用程序服务方法服务是一个好主意，以减少 DTO 的数量。然而，如果你这样做，一些属性将在某些方法中使用，但在其他方法中则不会使用。

为每个应用服务方法定义一个专门的输入 DTO 是一个好的实践。有时，由于两个方法几乎相同，似乎重用相同的 DTO 类是实用的。然而，应用服务方法会随着时间的推移而变化，需求也会不同。从另一个 DTO 继承 DTO 是重用 DTO 的另一种方式，但问题仍然是相同的。

在许多场景中，代码重复比耦合用例是一个更好的实践。请看以下示例：

```cs
public interface IEventAppService : IApplicationService
{
    Task<EventDto> GetAsync(Guid id);
    Task CreateAsync(EventDto input);
    Task UpdateEventTimeAsync(EventDto input);
}
```

在这个例子中，`GetAsync`方法返回一个存储几乎所有事件属性的`EventDto`对象。`CreateAsync`方法重用了相同的`EventDto`类。一般来说，将输出 DTO 作为输入 DTO 重用并不好，因为一些`EventDto`属性（如`Id`、`UrlCode`、`RegisteredUserCount`和`CreationTime`）在事件创建时不应由客户端应用程序发送，而是在服务器端计算。最后，在`UpdateEventTimeAsync`方法中重用相同的`EventDto`类更糟糕，因为此方法仅使用`Id`、`StartTime`和`EndTime`属性。

真正的 DTO 设计在以下示例中展示：

```cs
public interface IEventAppService : IApplicationService
{
    Task<EventDto> GetAsync(Guid id);
    Task CreateAsync(EventCreationDto input);
    Task UpdateEventTimeAsync(EventTimeUpdateDto input);
}
```

我们为`CreateAsync`和`UpdateEventTimeAsync`方法定义了单独的 DTO 类。这样，一个 DTO 的任何变化都不会影响其他方法。

在前两个部分中的设计建议是针对输入 DTO 的。下一部分将解释输出 DTO 的情况。

### 关于输出 DTO

在实践中，输出 DTO 与输入 DTO 不同。输入 DTO 的*未使用属性*问题在输出 DTO 中不存在。让我们尝试理解为什么未使用的属性对输入 DTO 是一个问题。想象一下，我们正在调用一个方法并在输入 DTO 上设置一个属性。我们期望它正在被方法处理并改变行为。如果方法没有使用该属性，并且我们没有看到任何行为上的差异，无论我们为该属性设置了什么，我们都会感到困惑。如果一个方法参数（或参数的属性）存在，它应该像预期的那样工作。

然而，对于输出 DTO，情况并非如此。一个应用服务可能返回比客户端当前需要的更多属性。因此，我的意思是应用服务方法填充了输出 DTO 的所有属性；如果 DTO 中存在某个属性，它应该始终被填充，无论客户端是否使用它。

这种方法有一个优点——当客户端稍后需要那些属性时，我们不需要更改服务类，这使得扩展 UI 更容易。如果客户端不是你的应用程序或者你希望将你的**应用程序编程接口**（**API**）向第三方客户端开放，他们的需求可能不同，这尤其有用。

由于输出 DTO 可能包含一些客户端尚未使用的属性，我们可以减少输出 DTO 的数量，并在多个情况下重用一些 DTO 类。

请看以下示例应用程序服务定义：

```cs
public interface IEventAppService : IApplicationService
{
    Task<EventDto> CreateAsync(CreateEventDto input);
    Task<EventDto> GetAsync(Guid id);
    Task<List<EventDto>> GetListAsync(PagedResultRequestDto 
                                      input);
    Task<EventDto> AddSessionAsync(Guid id, 
                                  AddSessionDto input);
}
```

在此示例中，所有方法都使用相同的输出 DTO 类 `EventDto` 来表示事件，而它们获取不同的输入 DTO 对象。

性能考虑

返回经过微调和最小输出的 DTO 对象可能因性能要求而需要，尤其是在返回大量结果集时。在这种情况下，你可以定义不同的 DTO 类，只包含相关用例所需的属性。

我们已经介绍了使用 ABP 框架实现 DDD 的基本构建块。下一节将演示一些示例，以了解各层的角色和职责。

# 理解各层的职责

将你的业务逻辑分离到应用层和领域层，允许你在同一领域上创建多个应用程序，如在第九章的 *处理多个应用程序* 部分中解释的，*理解领域驱动设计*。大型系统通常有多个应用程序，将核心领域与应用特定逻辑隔离是避免不同应用程序逻辑混合的关键原则。为每个应用程序创建一个单独的应用层，使我们能够设计最适合不同应用程序要求的应用服务方法。

为了成功地将应用层和领域层分离，我们应该对每一层的职责有良好的理解。在最后三章中，我已经在解释 DDD 构建块时提到了这些职责。在下一节中，我将总结这些职责，以便更好地理解它们。

## 授权用户

授权用于允许或阻止用户（在 UI 上）或客户端应用程序（在机器到机器通信中）使用某些应用程序功能。

ABP 提供了使用 `[Authorize]` 属性进行声明性检查用户权限和通过 `IAuthorizationService` 服务进行命令性检查的方式。你可以使用这些功能来限制对应用程序中所需功能的访问。

授权是应用层和上层（如表示层）的责任，因为它高度依赖于客户端和应用程序的用户。

例如，在 *EventHub* 项目中，公共网络应用中的用户只能编辑自己的事件。然而，在管理应用中的管理员用户，如果他们拥有所需的权限，可以编辑任何事件而无需检查所有权。另一方面，后台服务可能在没有任何授权规则的情况下更改事件的状态。这些应用程序使用相同的领域层，因此相同的领域规则，但实现了不同的授权规则。因此，最好不要在领域层中包含授权逻辑，以便在不同情况下可重用。

## 控制事务

UoW 系统的责任是为用例（通常是 Web 请求）创建事务范围，并确保在该用例中进行的所有更改都一起提交。UoW 系统在*第六章*中*与数据访问基础设施一起工作*的*理解 UoW 系统*部分中进行了介绍。

用例的范围是应用程序服务方法。应用程序服务方法可能与多个领域服务和聚合一起工作，并可能在聚合上做出更改。确保所有更改一起提交的唯一方法是控制应用程序层中的应用程序服务级别的 UoW 系统。因此，UoW 是一个应用层概念。

## 验证用户输入

如*第七章*中*探索横切关注点*部分的*验证用户输入*所述，一个典型应用程序有三个验证级别，如下所示：

+   *客户端验证*用于在将数据发送到服务器之前预先验证用户输入。此类验证是表示层的责任。

+   *服务器端验证*由服务器执行，以防止不完整、格式错误或恶意请求。我们通常使用数据注释属性和其他功能来验证 DTO 对象。此类验证是应用层的责任。

+   *业务验证*也在服务器上执行，实现您的业务规则，并保持业务数据的一致性。业务验证通常在领域层执行，以确保每个应用程序中都有相同的业务规则。

ABP 提供了良好的基础设施，并优雅地集成到 ASP.NET Core 服务中，以便轻松执行正式验证逻辑。您可以在聚合构造函数、方法和领域服务中实现业务验证规则和约束，如*第十章*中所述，*领域驱动设计 – 领域层*。

## 与当前用户一起工作

ABP 的`ICurrentUser`服务用于获取当前用户的信息。当前用户逻辑需要一个有状态的会话系统，该系统存储用户信息并在每个 Web 请求中使其可用。

对于 ASP.NET Core 应用程序，ABP 使用基于身份验证票据的当前原则。当用户登录应用程序时创建身份验证票据。它保存在 cookie 中，以便在后续请求中读取。对于**单页应用程序**（**SPA**），它可以存储在本地存储中，并在每个请求的**超文本传输协议**（**HTTP**）头中发送到服务器。

会话/当前用户是一个通常在表示层中实现的概念。它可以在应用层中使用，因为应用层是为了被表示层使用而设计的，所以它可以假设在当前上下文中存在一个 *当前用户*。然而，领域层应该设计为独立于任何应用，因此它不应该与当前用户一起工作。在某些应用程序中，例如后台服务或集成应用程序，可能根本不存在用户。

以下来自 *EventHub* 项目的应用程序服务方法，用于当前用户加入指定的组织：

```cs
[Authorize]
public async Task JoinAsync(Guid organizationId)
{
    var organization = await _organizationRepository
        .GetAsync(organizationId);
    var user = await
        _userRepository.GetAsync(CurrentUser.GetId());    
    await _organizationMembershipManager.JoinAsync(
        organization, user);
}
```

首先，这种方法是经过授权的。因此，可以保证该方法是由已经登录到应用程序的用户调用的，并且 `CurrentUser.GetId()` 返回一个有效的用户 ID。

我们不接受用户的 ID 作为方法参数；否则，任何经过身份验证的用户都可以使任何用户成为任何组织的成员。但我们希望每个用户都能自己加入组织。

我们从存储库中获取组织和用户聚合，并将工作委托给领域服务（`organizationMembershipManager`）。这样，领域服务就独立于当前用户概念，并且更具可重用性：它可以与任何用户一起工作，而不仅仅是当前用户。

# 摘要

在本章中，你学习了如何正确实现应用程序服务和设计 DTO。我详细介绍了 DTO 设计，例如验证输入 DTO 和将实体映射到 DTO，并基于最佳实践和我的经验提供了建议。

我们已经了解到，混合层的责任会使分层变得没有意义。我们已经调查了一些基本责任，以了解我们应该在哪个层实现这些责任。

随着本章的结束，我们已经完成了本书的第三部分。这一部分的目的在于展示如何使用 ABP 框架实现 **领域驱动设计**（**DDD**）的构建块。我提供了规则、最佳实践和建议，以便你在遵循它们时可以使代码库更易于维护。

本书下一部分将探讨使用 ABP 框架进行 UI 和 API 开发。在下一章中，我们将学习 ABP 框架 MVC/Razor Pages UI 的架构结构和基本功能。
