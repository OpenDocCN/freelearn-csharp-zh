# 7

# 依赖注入、服务和消息传递

随着我们继续使用.NET MAUI 构建我们的*Recipes!*应用程序，我们希望充分利用 MVVM 设计模式。MVVM 非常适合保持我们的代码整洁，并促进行业标准实践，使我们的代码库更易于维护和测试。在本章中，我们将重点关注两个对坚实的 MVVM 架构至关重要的关键概念：**依赖注入**（**DI**）和**消息传递**。DI 促进了关注点的分离，并使我们的代码更容易进行测试。消息传递帮助我们保持代码的不同部分不会相互纠缠。它允许应用程序的不同区域以松耦合的方式相互通信。这两个概念对于确保我们的 MVVM 架构真正突出至关重要。

让我们看看本章涵盖了哪些内容：

+   通过依赖注入实现控制反转

+   注册、解析和注入服务

+   消息传递

到本章结束时，您将对我们如何在*Recipes!*应用程序中实现这些概念有一个很好的理解。那么，让我们继续深入探讨。

# 技术要求

在本章的整个过程中，我们将增强*Recipes!*应用程序的功能。包括本章涵盖的主题所需的所有资源，包括额外的类和代码，都可以在 GitHub 上找到：[`github.com/PacktPublishing/MVVM-pattern-.NET-MAUI/tree/main/Chapter07`](https://github.com/PacktPublishing/MVVM-pattern-.NET-MAUI/tree/main/Chapter07)。要跟随本章的内容，您可以从提供的`Start`文件夹开始。它包含本章的初始代码和必要的特定类。此代码作为基础，建立在之前章节学到的内容之上。如果您想参考或比较完成的代码，包括本章编写的所有代码，可以在`Finish`文件夹中找到。

# 通过依赖注入实现控制反转

**控制反转**（**IoC**）是一种编程原则，其中程序流程的某些方面的控制权从主代码转移到框架或容器。简单来说，组件不再负责管理其依赖项和生命周期，这些责任被反转或委托给外部控制器。这种方法对于创建模块化和灵活的系统特别有用。

在典型的无 IoC 的软件设计中，一个需要从其他类获取特定功能的类会在其内部创建或管理这些依赖对象。使用 IoC 时，这种创建和管理由外部组件处理，因此实现了控制反转。

IoC 可以通过多种方法实现，例如 DI、工厂模式、服务定位器等。在这些方法中，DI 是 MVVM 上下文中最常用的方法。

## 依赖注入

依赖注入（DI）是 IoC 的一种特定形式，它涉及通过外部实体将对象的依赖项（如服务或组件）注入到对象中，而不是由对象创建它们。这通常是通过构造函数完成的。

使用 DI 带来了许多优势。让我们看看：

+   **关注点分离**：每个组件或类都专注于其核心责任。其依赖项的创建和生命周期管理由外部处理。

+   **可测试性**：DI 通过允许注入模拟依赖项，使得测试组件变得更加容易。这对于单元测试至关重要，因为在单元测试中，你希望隔离正在测试的组件，并且不必担心依赖项。例如，如果 ViewModel 依赖于一个获取数据的服务，你可以注入一个模拟数据服务来模拟数据检索，而不必实际击中数据库或 API。这使得测试更快、可重复且更可靠。

+   **可重用性和可维护性**：组件因为不与其依赖项紧密耦合，所以变得更加可重用和可维护。

+   **灵活性**：在不改变依赖类的情况下，更容易更改或交换依赖项的实现。

通过允许从外部注入依赖项，依赖注入（DI）支持创建更松散耦合的代码。这不仅导致系统更易于维护和扩展，而且对于测试也非常有利。通过在测试期间注入模拟或存根实现，你可以专注于单独测试单个组件的功能，而不必担心整个系统的复杂性和不可预测性。

注入不同实现的能力是 DI 的一个强大方面，并且对于创建健壮和灵活的架构至关重要。

记得，在*第一章*，“什么是 MVVM 设计模式？”中，我们有一个`MainPageViewModel`，其构造函数接受一个名为`IQuoteService`的接口？让我们看看：

```cs
public class MainPageViewModel : INotifyPropertyChanged
{
    private readonly IQuoteService quoteService;
    public MainPageViewModel(IQuoteService quoteService)
    {
        this.quoteService = quoteService;
    }
...
}
```

这是一个依赖注入的典型示例。`MainPageViewModel`不应该负责检索“每日名言”。相反，这应该是另一个类的责任。`MainPageViewModel`依赖于一个实现`IQuoteService`接口的类的实例以实现该功能。然而，而不是直接创建或管理该服务的实例，它通过其构造函数接收该实例。这就是所谓的通过构造函数的`IQuoteService`依赖，`MainPageViewModel`遵循的原则是`MainPageViewModel`类的责任是提供视图数据，而获取实际数据的责任则委托给`IQuoteService`。

在这里，`MainPageViewModel`对实现`IQuoteService`接口的类的来源、实例化方式或生命周期管理一无所知。它只是接收一个实例并使用它。这使得 ViewModel 独立于`IQuoteService`接口的具体实现，并且可以是实现此接口的任何类。

对于这个`IQuoteService`接口，我们只需创建一个新的类来实现它，并从 API 获取数据。然后我们可以将这个新类注入到我们的 ViewModel 中，而无需更改`MainPageViewModel`本身的任何代码。ViewModel 只关心它有一个实现`IQuoteService`接口的类来与之交互，而不关心它完成任务的具体细节。

注意

虽然在依赖注入（DI）中使用接口很常见，但这并不是一个严格的要求。接口在 DI 中很受欢迎，因为它们促进了松耦合。然而，抽象类甚至具体类也可以被注入。选择取决于您应用程序的具体需求和设计目标。

当涉及到测试时，这种解耦的优势变得更加明显。假设我们想要为`MainPageViewModel`编写单元测试。为了使这些测试可靠，我们需要确保它们不受外部依赖不可预测性的影响。使用依赖注入（DI），我们可以通过创建一个返回受控数据的`IQuoteService`的模拟实现来轻松实现这一点，这对于测试场景来说非常完美。这样，我们就可以在隔离的情况下测试`MainPageViewModel`的所有方面，而不会受到外部依赖不可预测行为的影响。

让我们看看 DI 的实际应用，并看看我们如何注册、解析和注入依赖项。

# 注册、解析和注入服务

.NET MAUI 自带对 DI 的支持。它已经考虑到了 DI，这使得配置和管理应用程序所依赖的服务变得更加容易。通过提供开箱即用的 DI 支持，.NET MAUI 使开发者能够利用 DI 和 IoC 的概念来使他们的代码更易于维护和更松耦合。由于 MVVM 模式从 DI 和 IoC 中获得了巨大的好处，这再次表明 MVVM 和.NET MAUI 是完美匹配的！

`Microsoft.Extensions.DependencyInjection`命名空间是.NET MAUI 获取其默认 DI 实现的来源。然而，需要注意的是，.NET MAUI 对 DI 容器是无关的，这意味着您不受默认容器的限制。如果您更喜欢第三方 DI 容器，您可以自由地将默认容器替换为您首选的选择。让我们看看如何使用.NET MAUI 的默认 DI 实现来注册服务。

## 注册服务

.NET MAUI 托管一个依赖注入容器，并使其在整个应用程序中可用。当应用程序启动时，你有机会配置将可用于注入的服务。这是通过`MauiAppBuilder`实例的`Services`属性完成的。如果你之前使用过 ASP.NET，那么你已经知道这是如何工作的。通过`Services`属性，我们可以在整个应用程序中设置服务。

`MauiAppBuilder`上的`Services`属性是`IServiceCollection`类型，这是一个框架提供的用于服务描述符集合的接口。它提供了在容器中注册服务的方法。在下面的示例中，使用`AddSingleton`方法将`QuoteService`注册为`IQuoteService`接口的单例服务：

```cs
public static MauiApp CreateMauiApp()
{
    var builder = MauiApp.CreateBuilder();
    ...
    builder.Services.AddTransient<IQuoteService,
      QuoteService>();
    ...
    return builder.Build();
}
```

前面的代码展示了我们如何将一个具体的实现（`QuoteService`）与一个接口（`IQuoteService`）关联起来。因此，当应用的`IServiceProvider`需要解析实现`IQuoteService`的类的实例时，它将创建（或重用）`QuoteService`类的实例。`IServiceProvider`代表一个服务容器，它是一组服务注册的集合。它本质上是一个负责在需要时解析和提供通过关联的`IServiceCollection`注册的服务实例的对象。为特定接口注册具体的实现使我们能够将实现与代码中的实际使用解耦。面向接口编程而不是面向实现编程是一种良好的编程实践，这正是我们在这里能够实现的目标。

然而，对于 ViewModel 或其他不一定有相关接口的类，我们可以直接注册它们，如下面的代码片段所示：

```cs
builder.Services.AddTransient<MainPageViewModel>();
```

当我们这样做时，每次从依赖注入容器请求`MainPageViewModel`的实例时，它将提供一个类的实例并管理其生命周期。

根据预期的生命周期，服务可以以不同的方式注册：

+   **瞬态（Transient）**：瞬态服务每次从容器请求时都会创建。这种生命周期最适合轻量级、无状态的服务。从理论上讲，瞬态服务因为每次请求服务时都会创建一个新的实例，所以会占用更多的内存。然而，因为每个实例一旦不再使用就可以被垃圾回收，所以这些内存可以快速回收。

+   **单例（Singleton）**: 单例服务只创建一次，并在整个应用程序的生命周期内使用相同的实例。这对于需要在整个应用程序中保持一致性的有状态服务，或者对于创建多次成本较高的重服务来说，效果最好。单例服务占用的内存最少，因为只创建了一个实例。然而，由于整个应用程序都使用相同的实例，它将保留在内存中，直到应用程序运行结束。

+   **作用域（Scoped）**: 作用域服务在每个作用域内创建一次。然而，在.NET MAUI 应用程序中，这类似于单例，因为通常只有一个作用域。

选择这些生命周期完全取决于服务的具体需求。如果你的服务是无状态的且轻量级，那么瞬态（Transient）生命周期可能很合适。如果你的服务需要在整个应用程序中维护状态，或者创建成本较高，那么单例（Singleton）生命周期可能是最佳选择。为你的服务选择正确的生命周期非常重要。它可能会影响你的应用程序的行为和性能，所以请仔细考虑。根据我的经验，我通常尽可能选择瞬态（Transient）作为我的服务。我这样做是因为我旨在保持我的服务简单且无状态。这样，它们更有效地使用内存，因为一旦不再需要，它们就会被移除。此外，这有助于避免在不同地方更改共享状态时可能出现的问题，尤其是在多线程情况下。但请记住，这里没有一刀切的方法。每个服务都是独特的，你的应用程序也是如此。根据你的服务做什么以及你的应用程序需要什么，你可能需要选择不同的生命周期。这就是为什么理解这些概念并为你的特定应用程序做出明智的选择至关重要。

让我们看看我们如何解决和注入这些已注册的服务。

## 解决和注入服务

DI 容器，如.NET MAUI 中内置的容器，能够解决不仅包括直接依赖项，还包括嵌套依赖项。

从本质上讲，当容器解决类或服务的实例时，所有其依赖项以及这些依赖项的依赖项都会自动解决并注入。这形成了一个完整的对象图，其中每个类都有其依赖项得到满足。

如果一个特定的类有一个需要注入的依赖项，只需将其定义为类构造函数的参数即可。这正是我们对`MainPageViewModel`所做的那样：它有一个需要实现`IQuoteService`接口的类的实例的构造函数。

DI 容器使用哪个构造函数？

由于 DI 容器可以为我们实例化类，你可能会想知道当类有多个构造函数时，它使用哪个构造函数。答案是相当简单的：它使用可以解析的参数最多的构造函数。如果有两个或更多具有相同参数数量的构造函数可以被 DI 容器解析，或者当找不到所有依赖项都可以解析的构造函数时，将抛出异常。

我们还能够动态解析服务，只要我们能够获取到应用的`IServiceProvider`容器。此接口公开了一个`GetService<T>`方法，我们可以调用它来获取与提供的泛型类型参数关联的类的实例。以下代码块显示了如何创建一个静态的`ServiceProvider`类，我们可以用它从应用的任何地方访问服务容器：

```cs
public static class ServiceProvider
{
    public static TService GetService<TService>()
        => Current.GetService<TService>();
    public static IServiceProvider Current
        =>
#if WINDOWS10_0_17763_0_OR_GREATER
        MauiWinUIApplication.Current.Services;
#elif ANDROID
        MauiApplication.Current.Services;
#elif IOS || MACCATALYST
       MauiUIApplicationDelegate.Current.Services;
#else
      null;
#endif
}
```

因此，我们不需要在`MainPage_MVVM`类的代码背后实例化`MainPageViewModel`，也不需要手动提供实现 IQuoteService 接口的类的实例，我们可以这样做：

```cs
public MainPage_MVVM()
{
    InitializeComponent();
    BindingContext = ServiceProvider
        .GetService<MainPageViewModel>();
}
```

由于我们已经将`MainPageViewModel`注册为服务，并将`QuoteService`与之关联到`IQuoteService`接口，`GetService`方法将返回一个`MainPageViewModel`的实例，该实例通过其构造函数注入了`QuoteService`类的实例。这个`ServiceProvider`类可以在动态地从 DI 容器中解析类实例时非常有帮助。然而，对于这个特定的例子，我们可以更进一步，避免手动解析`MainPageViewModel`实例的需求。我们可以通过将`MainPageViewModel`的实例作为依赖项添加到`MainPage_MVVM`类中来实现这一点，如下所示：

```cs
public MainPage_MVVM(MainPageViewModel vm)
{
    InitializeComponent();
    BindingContext = vm;
}
```

以下代码片段显示了如何注册`MainPage_MVVM`类：

```cs
builder.Services.AddTransient<MainPage_MVVM>();
```

在这个示例应用中，我们使用.NET MAUI Shell 进行导航。这允许在导航过程中动态解析页面的实例。因此，当我们导航到`MainPage_MVVM`页面时，DI 容器会立即行动。它解析`MainPage_MVVM`页面的实例及其所有依赖项。

什么是.NET MAUI Shell？

在第八章“MVVM 中的导航”中，我们将更深入地探讨导航和.NET MAUI Shell 的各个方面。

现在我们已经通过“每日名言”应用体验了依赖注入（DI），让我们将所学应用到功能丰富的“食谱”应用中，进一步提升我们的技能。这将使我们能够更深入地了解 DI，并看到它如何在更复杂的项目中巧妙地被利用。所以，卷起袖子，让我们在“食谱”应用中使用 DI 大显身手吧。

## 应用依赖注入

到目前为止，在我们的 *Recipes!* 应用中的 ViewModels 我们一直在使用硬编码的数据。现在，是时候通过引入可以动态获取和管理数据的服务来给我们的应用程序注入更多活力了。GitHub 仓库中本章的 `Begin` 目录展示了某些更新和额外的代码，包括新的服务接口，如 `IRecipeService`、`IFavoritesService` 和 `IRatingsService`，以及它们各自的实现。这些服务将在我们的应用程序中扮演关键角色：`IRecipeService` 接口定义了一个将加载和管理食谱数据的服务合同。同样，`IFavoritesService` 概述了处理用户喜欢的食谱的服务规则，而 `IRatingsService` 接口对管理食谱评分的服务做了同样的事情。随着我们继续前进，我们将探讨如何在 MVVM 架构中使用这些服务，以及 DI 如何以干净、可管理的方式将它们全部整合在一起。

为了在我们的 *Recipes!* 应用中引入依赖注入（DI），我们需要确保，与我们现在所做的不一样，我们不再在代码背后初始化 ViewModels。相反，这些 ViewModels 需要被注入并分配给页面的 `BindingContext`。在我们更新 ViewModels 之前，让我们先看看这个。

### 向页面添加依赖项

要将特定 ViewModel 的依赖项添加到页面中，我们只需将 ViewModel 的类型作为参数添加到构造函数中。此外，我们还需要确保页面和 ViewModel 都已在 DI 容器中注册：

1.  前往 `RecipesOverviewPage` 的代码背后，并将 `RecipeOverviewViewModel` 类型的参数添加到页面的构造函数中，如下面的代码片段所示：

    ```cs
    public RecipesOverviewPage(RecipesOverviewViewModel
      viewModel)
    {
        InitializeComponent();
        BindingContext = viewModel;
    }
    ```

1.  接下来，我们需要确保 `RecipesOverviewPage` 和 `RecipeOverviewViewModel` 类已在 DI 容器中注册。只有这样，DI 容器才能解析 `RecipesOverviewPage` 并解析其依赖项，即 `RecipesOverviewViewModel` 的一个实例。前往 `MauiProgram` 并添加以下代码行：

    ```cs
    public static MauiApp CreateMauiApp()
    {
        var builder = MauiApp.CreateBuilder();
        builder
            .UseMauiApp<App>()
        ...
        builder.Services
            .AddTransient<RecipesOverviewPage>();
        builder.Services
            .AddTransient<RecipesOverviewViewModel>();
        ...
    }
    ```

1.  同样，我们还需要对 `RecipeDetailPage` 和 `RecipeRatingDetailPage` 做同样的处理：通过将它们作为参数包含进来，将它们各自的 ViewModels 作为依赖项添加。以下是 `RecipesOverviewPage` 的样子：

    ```cs
    public RecipesOverviewPage(
        RecipesOverviewViewModel viewModel)
    {
        InitializeComponent();
        BindingContext = viewModel;
    }
    ```

    同样，对于 `RecipeRatingDetailPage`，我们必须做以下操作，其中我们想要注入 `RecipeRatingsDetailViewModel`：

    ```cs
    public RecipeRatingDetailPage(
        RecipeRatingsDetailViewModel viewModel)
    {
        InitializeComponent();
        BindingContext = viewModel;
    }
    ```

1.  现在，就像我们之前做的那样，让我们在 `MauiProgram` 类中注册这些额外的页面和它们的 ViewModels：

    ```cs
    public static MauiApp CreateMauiApp()
    {
        var builder = MauiApp.CreateBuilder();
        builder
            .UseMauiApp<App>()
        ...
        builder.Services
            .AddTransient<RecipesOverviewPage>();
        builder.Services
            .AddTransient<RecipesOverviewViewModel>();
        builder.Services
            .AddTransient<RecipeDetailPage>();
        builder.Services
            .AddTransient<RecipeDetailViewModel>();
        builder.Services
            .AddTransient<RecipeRatingDetailPage>();
        builder.Services
            .AddTransient<RecipeRatingsDetailViewModel>();
    ...
    }
    ```

通过这些修改，我们已经在我们的*Recipes!*应用中成功实现了 DI 的基础元素。通过将页面及其对应的 ViewModel 注册到 DI 容器中，我们确保了每当需要这些组件时，它们可以很容易地由 DI 容器解决并提供。此外，通过通过构造函数将 ViewModel 注入到我们的页面中，我们将创建和管理 ViewModel 实例的责任从页面本身转移到 DI 容器。这为我们应用程序设置了一个更灵活、更易于维护的结构，为我们进一步通过额外的服务和功能增强它铺平了道路。

现在，让我们更仔细地看看我们需要对 ViewModel 进行的具体更改，以完全实现 DI。

### 向 ViewModel 添加依赖项

我们不再希望我们的 ViewModel 包含硬编码的数据，也不希望它们负责检索数据。因此，让我们向我们的 ViewModel 中引入一些依赖项：

1.  在`RecipesOverviewViewModel`类的顶部，我们可以先移除`items`字段。我们正在远离硬编码的数据，并将使用服务来获取数据。

1.  以下代码片段展示了我们如何向这个类中引入两个字段：`recipeService`，其类型为`IRecipeService`，以及`favoritesService`，其类型为`IFavoritesService`：

    ```cs
    private readonly IRecipeService recipeService;
    private readonly IFavoritesService favoritesService;
    ```

    这些服务将负责加载此页面上的食谱，并显示用户是否收藏了它们。这两个服务都是将被注入到 ViewModel 中的依赖项。

1.  此代码块展示了这些依赖项如何通过 ViewModel 的构造函数进行注入：

    ```cs
    public RecipesOverviewViewModel(
        IRecipeService recipeService,
        IFavoritesService favoritesService)
    {
        this.recipeService = recipeService;
        this.favoritesService = favoritesService;
        Recipes = new ();
        TryLoadMoreItemsCommand =
            new AsyncRelayCommand(TryLoadMoreItems);
        NavigateToSelectedDetailCommand =
            new AsyncRelayCommand
              (NavigateToSelectedDetail);
        LoadRecipes(7, 0);
    }
    ```

    通过在`RecipesOverviewViewModel`的构造函数中定义这两个参数，DI 容器将尝试在创建`RecipesOverviewViewModel`时解决这两个实例。然后，解决的实例作为参数传递给构造函数，在那里我们可以将它们分配给之前创建的字段。

1.  当我们检查`LoadRecipes`方法时，我们可以看到我们如何利用这些服务来加载数据：

    ```cs
    private async Task LoadRecipes(int pageSize, int page)
    {
        var loadRecipesTask =
            recipeService.LoadRecipes(pageSize, page);
        var loadFavoritesTask =
            favoritesService.LoadFavorites();
       ...
    }
    ```

    ViewModel 不关心这些食谱或收藏来自哪里。它只信任注入的服务——`recipeService`和`favoritesService`——遵循指定的接口并交付所需的功能。具体的实现被抽象化，从 ViewModel 中突出显示了 DI 的主要好处之一。

1.  转到 `RecipeDetailViewModel`。在这个类中，我们同样希望移除所有硬编码的数据：`Title`、`Allergens`、`Calories` 等等。并且在我们做这件事的同时，我们应该更新属性为“完整”属性，这些属性会触发 `PropertyChanged` 事件。这是必要的，因为 ViewModel 上的数据将异步加载，因此在页面渲染时可能不存在。因此，每个属性的 `PropertyChanged` 事件需要在数据加载时触发，以便在 UI 上反映加载的值。以下代码片段显示了部分更新的属性：

    ```cs
    string _title;
    public string Title
    {
        get => _title;
        set => SetProperty(ref _title, value);
    }
    string[] _allergens = new string[0];
    public string[] Allergens
    {
        get => _allergens;
        set => SetProperty(ref _allergens, value);
    }
    int? _calories;
    public int? Calories
    {
        get => _calories;
        set => SetProperty(ref _calories, value);
    }
    ```

1.  现在也是时候更新标签的绑定模式了，该标签显示菜谱的标题。到目前为止，这被定义为 `OneTime` 绑定，但因为我们现在在将 ViewModel 设置为 `RecipeDetailPage` 的 `BindingContext` 之后加载菜谱的数据，我们需要确保更新的值也显示在屏幕上。让我们将绑定模式更新为 `OneWay`，这样当 `Title` 属性的值设置后触发 `PropertyChanged` 事件时，绑定引擎会更新 UI 上的值。以下代码片段显示了更新的标签：

    ```cs
    <Label
        FontAttributes="Bold"
        FontSize="22"
        Text="{Binding Path=Title, Mode=OneWay}"
        VerticalOptions="Center" />
    ```

1.  让我们更新 `RecipeDetailViewModel` 的构造函数，使其接受 `IRecipeService`、`IFavoritesService` 和 `IRatingsService` 的实例，如下所示：

    ```cs
    public RecipeDetailViewModel(
        IRecipeService recipeService,
        IFavoritesService favoritesService,
        IRatingsService ratingsService)
    {
        this.recipeService = recipeService;
        this.favoritesService  = favoritesService;
        this.ratingsService = ratingsService;
    ...
    }
    ```

1.  这些服务（`recipeService`、`favoritesService` 和 `ratingsService`）是 ViewModel 中的 `readonly` 字段，如以下代码片段所示：

    ```cs
    private readonly IRecipeService recipeService;
    private readonly IFavoritesService favoritesService;
    private readonly IRatingsService ratingsService;
    ```

1.  以下代码块显示了 `LoadRecipe` 方法，该方法接受要加载的菜谱 ID 作为参数。此方法使用注入的服务来加载此 ViewModel 所需的所有相关数据：

    ```cs
    private async Task LoadRecipe(string recipeId)
    {
        var loadRecipeTask =
            recipeService.LoadRecipe(recipeId);
        var loadIsFavoriteTask =
            favoritesService.IsFavorite(recipeId);
        var loadRatingsTask =
            ratingsService.LoadRatingsSummary(recipeId);
        await Task.WhenAll(loadRecipeTask,
            loadRecipeTask, loadRatingsTask);
        if(loadRecipeTask.Result is not null)
            MapRecipeData(
                loadRecipeTask.Result,
                loadRatingsTask.Result,
                loadIsFavoriteTask.Result);
    }
    ```

    获取 `RecipeDetailDto`、`RatingsSummaryDto` 和表示菜谱是否为收藏夹的 `bool` 值的三个异步任务并行启动。通过 `Task.WhenAll` 方法，我们等待所有三个任务完成。在此点之后，任务的 `Result` 属性包含检索到的数据。然后，通过 `MapRecipeData` 方法将这些数据映射到 ViewModel。

1.  在下一章，*第八章*，*MVVM 中的导航*，我们将探讨如何将选中菜谱的 ID 从 `RecipesOverviewViewModel` 传递到 `RecipeDetailViewModel` 以加载选中菜谱的详细信息。现在，让我们在 ViewModel 的构造函数末尾添加以下代码片段以加载 ID 为 `3` 的菜谱的详细信息：

    ```cs
    LoadRecipe("3");
    ```

1.  最后，我们还需要更新 `RecipeRatingsDetailViewModel`。和之前一样，我们希望移除所有硬编码的数据，并更新构造函数，使其接受一个实现了 `IRecipeService` 接口和实现了 `IRatingsService` 接口类的实例。以下代码片段显示了更新的构造函数，其中我们还删除了 `Reviews` 和 `GroupedReviews` 属性的初始化：

    ```cs
    public RecipeRatingsDetailViewModel(
        IRecipeService recipeService,
        IRatingsService ratingsService)
    {
        this.recipeService = recipeService;
        this.ratingsService = ratingsService;
        ...
    }
    ```

1.  可以移除`Reviews`属性，并确保当`RecipeTitle`属性被更新时调用`PropertyChanged`事件。同样，我们必须这样做，因为数据是异步加载的，我们必须通知 UI 关于更新值的消息。下面的代码块显示了更新的`RecipeTitle`属性，它使用`ObservableObject`类的`SetProperty`方法来分配值并触发`PropertyChanged`事件。它还显示了我们在构造函数中分配注入依赖关系的字段：

    ```cs
    public class RecipeRatingsDetailViewModel :
      ObservableObject
    {
        private readonly IRatingsService ratingsService;
        private readonly IRecipeService recipeService;
        string _recipeTitle = string.Empty;
        public string RecipeTitle
        {
            get => _recipeTitle;
            set => SetProperty(ref _recipeTitle, value);
        }
        …
    }
    ```

1.  让我们再添加一个`LoadData`方法，该方法接受我们想要加载评分的食谱 ID。它使用注入的服务动态加载 ViewModel 中所需的数据。让我们看看：

    ```cs
    private async Task LoadData(string recipeId)
    {
        var recipeTask =
            recipeService.LoadRecipe(recipeId);
        var ratingsTask =
            ratingsService.LoadRatings(recipeId);
        await Task.WhenAll(recipeTask, ratingsTask);
        RecipeTitle =
            recipeTask.Result?.Name ?? string.Empty;
        GroupedReviews = ratingsTask.Result
            ...
            .ToList();
    }
    ```

1.  现在，让我们在构造函数中调用`LoadData`方法，以便在初始化 ViewModel 时加载一些数据：

    ```cs
    LoadData("3");
    ```

在对 ViewModel 的所有更新完成后，让我们通过注册我们的更新后的 ViewModel 现在作为依赖项的服务来完成。

### 注册服务

现在我们 ViewModel 有一些依赖项，我们必须确保这些依赖项被注册，以便 DI 容器可以解析它们。

在`MauiProgram`类中再次执行注册所需依赖项的操作。以下代码片段显示了如何注册`FavoritesService`，这非常直接：

```cs
builder.Services.AddSingleton<IFavoritesService,
  FavoritesService>();
```

我们故意将`FavoritesService`注册为 Singleton，因为这种特定的实现将用户的收藏存储在内存中。如果我们将其注册为 Transient，每次将其作为依赖项注入时都会创建一个新的实例，这将导致收藏在页面导航之间不会持久化。然而，值得注意的是，将收藏保留在内存中并不是理想的，但为了这个示例，它将满足我们的目的。在现实场景中，我们希望收藏能够持久化存储在（在线）数据存储中。

注册`RecipeService`涉及一个稍微复杂的过程。原因在于`RecipeService`的构造函数需要一个`Task`属性，该属性返回一个指向包含所有食谱信息的 JSON 文件的流。这可以在`RecipeService`的构造函数中看到：

```cs
public RecipeService(Task<Stream> recipesJsonStreamTask)
{
    this.recipesJsonStreamTask = recipesJsonStreamTask;
}
```

我们不能像之前在`FavoritesService`或其他服务中注册那样注册`RecipeService`。这是因为 DI 容器需要知道传递给构造函数的参数是什么。在之前的示例中，这很简单：我们只需指定我们想要与接口或基类或类型本身关联的具体类型。然后容器可以通过调用其默认构造函数或注入其他已解析的依赖关系来创建具体类的实例。

然而，对于 `RecipeService` 来说，创建实例所需的参数并不是我们计划注册的，这意味着它不能由依赖注入容器解析。为了应对此类场景，`AddTransient`、`AddSingleton` 和 `AddScoped` 方法提供了重载功能。这个重载功能允许我们传递一个函数，该函数返回我们想要与给定基类型关联的类型实例。每当关联类型需要被解析时，这个函数就会被调用。更重要的是，该函数的参数是 `IServiceProvider` 本身，这允许我们在必要时解析任何额外的依赖项。以下代码块展示了我们如何使用重载函数注册 `RecipeService`，同时传递一个创建此类实例的函数：

```cs
builder.Services.AddTransient<IRecipeService>(
    serviceProvider => new RecipeService( FileSystem.
      OpenAppPackageFileAsync("recipedetails.json")));
```

每当需要通过依赖注入容器解析 `IRecipeService` 对象时，传入的函数都会被调用。然而，由于使用了 `AddSingleton` 方法，该函数只会被调用一次。这意味着在这个特定用例中，将服务注册为 Singleton 可能是一个明智的决定，因为它将确保 JSON 文件只被读取一次，从而保持食谱在内存中，并优化应用程序的性能。

对于 `RatingsService` 的注册也是如此。就像 `RecipeService` 一样，这个类也将从本地文件中读取评分。因此，就像之前一样，我们希望使用重载的 `AddTransient` 方法来注册此服务，如下所示：

```cs
builder.Services.AddSingleton<IRatingsService>(
    serviceProvider => new RatingsService( FileSystem.
       OpenAppPackageFileAsync("ratings.json")));
```

一旦所有这些服务都已注册，我们就可以继续运行 *Recipes!* 应用程序。我们的代码现在利用了依赖注入，这是一种极大地增强了我们应用程序模块化和可测试性的实践。通过注入依赖项，我们解耦了具体类与接口或基类，允许我们更改或替换底层实现，而不会影响依赖类。在 MVVM 模式下，依赖注入允许我们为 ViewModels 提供处理其任务所需的服务，例如数据获取或业务逻辑，而不需要硬编码这些依赖项，从而促进关注点的清晰分离。此外，我们甚至通过将 ViewModels 直接注入到视图中，进一步强调了这种实践在我们应用程序开发过程中的灵活性和多功能性。

注意

尽管我们主要讨论了在 .NET MAUI 中基于构造函数的依赖注入，但值得提及的是，在更广泛的环境中，依赖也可以通过属性或方法注入。然而，这些方法在 .NET MAUI 中并不是原生支持的。依赖注入的本质是为类提供其依赖项，无论使用何种方法。构造函数注入通常因为其清晰性而被优先考虑，但所使用的具体技术可能会根据平台和设计目标的不同而有所变化。

DI 在保持我们应用程序组件解耦方面发挥着至关重要的作用。接下来，我们将深入了解另一种促进应用程序解耦的机制。

# 消息传递

消息传递是一种软件架构模式，它促进了应用程序不同部分之间的通信。在.NET MAUI 和 MVVM 架构的背景下，消息传递通常用于在松散耦合的组件之间发送通知，例如在 ViewModel 之间，或者从模型到 ViewModel。这解耦了组件，并促进了更模块化和可维护的代码库。

当需要在不同部分之间传递数据或通信事件，而这些部分之间没有直接关系时，消息传递的概念特别有用。而不是通过直接调用对方来紧密耦合这些部分，你可以使用一个消息系统，其中一个部分发送的消息可以被应用程序中任何感兴趣的任何部分接收并做出反应。

这种模式是**观察者模式**的一种形式，其中名为**主题**的对象维护其依赖者（称为**观察者**）的列表，并自动通知它们任何状态变化，通常是通过调用它们的方法之一。同样，在 MVVM 中，消息传递用于在应用程序的解耦组件之间进行通信：应用程序中的任何对象，包括 ViewModel、服务、模型类或服务都可以发送消息，任何订阅了该特定类型消息的任何其他类都将被通知并可以相应地做出反应。

关于 MessagingCenter

`MessagingCenter`最初在 Xamarin.Forms 中作为组件之间松散耦合通信的机制被引入，但在.NET MAUI 中被标记为已弃用。虽然它在.NET 8 中保留以用于过渡场景，但其使用是不被推荐的！

通常，在 MVVM 的背景下，如以下图所示，消息系统本身维护一个观察者列表，并处理从发送者到适当接收者的消息传递：

![图 7.1：消息概述](img/B20941_07_01.jpg)

图 7.1：消息概述

消息模式的一个显著挑战是其固有的不透明性：很难确定应用程序的哪些部分订阅了特定的消息。这种缺乏透明度在修改代码时可能导致不可预见的结果，使得代码库更难以导航和调试。当我确实使用消息时，我会非常谨慎。保持消息最小化和专注于特定任务可以帮助减轻这一挑战。使用消息的另一个潜在风险是无意中造成内存泄漏。这可能会发生在对象订阅了消息但从未取消订阅的情况下。如果发生这种情况，消息系统会继续持有订阅者对象的引用，即使应用程序中没有其他引用，也会阻止它被垃圾回收。随着时间的推移，这可能会导致内存使用量增加，并最终降低应用程序的性能。

在 MVVM 的上下文中，这个问题尤为重要，因为 ViewModel 可能会在初始化期间订阅消息，然后在用户在应用程序中导航时被新的 ViewModel 所取代。如果这些 ViewModel 在不再使用时没有取消订阅消息，它们将无限期地留在内存中。

正是这里，`WeakReferenceMessenger` 发挥了作用。

## WeakReferenceMessenger

我们之前讨论过的 MVVM Toolkit 为我们提供了一个强大的消息传递实现，名为`WeakReferenceMessenger`。考虑到 MVVM 应用程序的需求，这个消息传递者确保我们可以在不担心潜在内存泄漏的情况下享受消息传递的好处。

与传统的消息传递者不同，后者对其订阅者持有强引用，`WeakReferenceMessenger` 则持有弱引用。这意味着它不会阻止其订阅者被垃圾回收。因此，即使你忘记取消订阅，垃圾回收器仍然可以在不再使用时清理你的 ViewModel，从而防止内存泄漏。

注意

在本节中，我们将使用 MVVM Toolkit 中的`WeakReferenceMessenger`作为我们的消息系统。然而，重要的是要注意，还有其他消息系统可供选择。虽然我们专注于`WeakReferenceMessenger`，但我们在这里讨论的核心概念——例如发送和接收消息——适用于大多数消息系统。始终记得研究和理解你正在使用的特定消息系统，以充分利用其功能并避免潜在的风险。

`WeakReferenceMessenger`使用基于类型的消息系统。这意味着当你发送消息时，你指定一个消息类型，只有订阅了该特定类型的接收者才会收到消息。消息类型通常定义为类，而消息数据则存储为该类的属性。

要发送消息，你必须使用 `Send` 方法，传入消息对象，可选地传入发送者和目标。然后，信使会将消息传递给所有已注册的指定消息类型的接收者。要接收消息，一个类需要通过调用 `Register` 方法与信使注册，指定它希望接收的消息类型，并提供一个回调方法，当发送该类型消息时将被调用。

让我们看看我们如何通过使用 `WeakReferenceMessenger` 更新我们的代码，并使其更加松耦合。

### 更新份数数量

通过消息传递，ViewModels 可以以松耦合的方式相互通信。例如，让我们看看 `IngredientsListViewModel`。当更新 `NumberOfServings` 属性的值时，我们会遍历 `Ingredients` 集合中的所有元素（它们是 `RecipeIngredientViewModel` 对象），并调用它们的 `UpdateServings` 方法，传入更新的值：

```cs
public int NumberOfServings
{
    get => _numberOfServings;
    set
    {
        if (SetProperty(ref _numberOfServings, value))
        {
            Ingredients
                .ForEach(i => i.UpdateServings(value));
        }
    }
}
```

这种方法紧密耦合，因为属性了解其他对象的实现细节，特别是 `RecipeIngredientViewModel`。此外，它并不很好地遵循 **单一职责原则**：*属性不仅关注自身；它还负责更新其他属性的值。*

因此，让我们在这里引入消息传递！

+   由于 `WeakReferenceMessenger` 是基于类型的消息系统，我们必须创建一个新类型，我们可以在用户更新份数时发送和订阅。在 `Messages` 上右键单击。

+   在 `ServingsChangedMessage` 上右键单击。

虽然 `WeakReferenceMessenger` 可以发送任何类型的消息，但你可能希望从某些基消息类继承。在这种情况下，我们可以从通用的 `CommunityToolkit.Mvvm.Messaging.Messages.ValueChanged` 类继承，因为 `ServingsChangedMessage` 正是为了这个目的。

下面的代码块显示了该类的实现：

```cs
using CommunityToolkit.Mvvm.Messaging.Messages;
namespace Recipes.Client.Core.Messages;
public class ServingsChangedMessage :
    ValueChangedMessage<int>
{
    public ServingsChangedMessage(int value)
        : base(value)
    { }
}
```

现在，我们可以更新 `IngredientsListViewModel` 类中的 `NumberOfServings` 方法。我们不再需要遍历配料列表中的每个项目并调用其 `UpdateServings` 方法，现在我们可以发送 `ServingsChangedMessage`，如下所示：

```cs
public int NumberOfServings
{
    get => _numberOfServings;
    set
    {
        if (SetProperty(ref _numberOfServings, value))
        {
            WeakReferenceMessenger.Default.Send(
                new ServingsChangedMessage(value));
        }
    }
}
```

向 `WeakReferenceMessenger` 的实例调用 `Send` 方法发送消息就像发送一个你想发送的消息一样简单。

最后，我们需要更新 `RecipeIngredientViewModel`。这个类需要订阅 `ServingsChangedMessage` 以便能够对其做出反应。通过在 `WeakReferenceMessenger` 上调用通用的 `Register` 方法来注册消息类型。作为类型参数，你需要传入你想要监听的消息类型。以下是如何注册 `ServingsChangedMessage` 的一种方法：

```cs
public RecipeIngredientViewModel(...)
{
...
    WeakReferenceMessenger.Default
    .Register<ServingsChangedMessage>(this, (r, m) =>
    ((RecipeIngredientViewModel)r)
    .UpdateServings(m.Value));
}
```

`Register`方法的第一参数是消息的接收者，在我们的情况下将是这个类本身。第二个参数是当接收到消息时被调用的处理程序。处理程序的第一个参数是接收者，第二个参数是消息本身。传入的接收者允许 Lambda 表达式不捕获`this`，这可以提高性能。也许将`UpdateServings`方法的访问修饰符更新为`private`也是一个好主意，因为不再需要公开访问这个方法。

使用这个更新后的实现，`IngredientsListViewModel`中的`NumberOfServings`属性不再需要了解`RecipeIngredientViewModel`对象。相反，当其值发生变化时，它只需发送一条消息。订阅这些消息的`RecipeIngredientViewModel`对象可以相应地更新其状态。这种方式解耦了这两个类，并确保每个类只负责管理自己的状态，遵循单一职责原则和关注点分离。

在以下示例中，我们将探讨消息传递不仅对 ViewModel 之间有价值。服务也可能发送 ViewModel 可以响应的消息。

### 保持收藏同步

在*Recipes!*应用程序中，`RecipesOverviewPage`显示所有食谱，用户可以在`RecipeDetailPage`上标记收藏。然而，如果没有重新加载`RecipesOverviewPage`，新收藏的食谱不会被突出显示。鉴于食谱数据库不经常更新，频繁的页面重新加载将是过度行为，可能会影响用户体验。

一种更有效的策略是使用消息传递。当一个食谱被收藏时，会发送一条消息。`RecipesOverviewViewModel`中包含的`RecipeListItemViewModel`个体订阅了这个消息，并在接收到消息后实时更新其收藏状态。这种方法防止了不必要的请求数据，从而提高了应用程序的性能和响应速度。让我们看看我们需要做什么来实现这一点：

1.  首先，让我们添加一个新的消息类型。右键单击`FavoriteUpdateMessage`作为类名。

1.  将以下代码添加到`FavoriteUpdateMessage`类中：

    ```cs
    public class FavoriteUpdateMessage
    {
        public string RecipeId { get; }
        public bool IsFavorite { get; }
        public FavoriteUpdateMessage(string recipeId,
            bool isFavorite)
        {
            RecipeId = recipeId;
            IsFavorite = isFavorite;
        }
    }
    ```

    这个类包含两个属性，`RecipeId`和`IsFavorite`，这样我们就可以通过这条消息来指示哪个食谱被标记或移除收藏。

1.  以下代码块展示了我们如何从`FavoritesService`发送`FavoriteUpdateMessage`，每当一个食谱被添加为收藏时：

    ```cs
    public Task Add(string id)
    {
        if(!favorites.Contains(id))
        {
            favorites.Add(id);
            WeakReferenceMessenger.Default.Send(
                new FavoriteUpdateMessage(id, true));
        }
        return Task.CompletedTask;
    }
    ```

1.  同样，当一个食谱被移除收藏时，也可以发送`FavoriteUpdateMessage`，如下所示：

    ```cs
    public Task Remove(string id)
    {
        if (favorites.Contains(id))
        {
            favorites.Remove(id);
            WeakReferenceMessenger.Default.Send(
                new FavoriteUpdateMessage(id, false));
        }
        return Task.CompletedTask;
    }
    ```

1.  最后一步是在`RecipeListItemViewModel`中订阅此消息。这确保了当`FavoriteUpdateMessage`到达时，`IsFavorite`属性可以相应地更新。与之前的示例不同，当时我们使用`Register`方法定义了一个消息处理器，这次我们将采用不同的方法，通过实现`IRecipient`接口。下面是如何做到这一点的示例：

    ```cs
    public class RecipeListItemViewModel :
        ObservableObject,
        IRecipient<FavoriteUpdateMessage>
    {
    ...
        public RecipeListItemViewModel(...)
        {
            ...
            WeakReferenceMessenger.Default.Register(this);
        }
        void IRecipient<FavoriteUpdateMessage>
            .Receive(FavoriteUpdateMessage message)
        {
            if (message.RecipeId == Id)
            {
                IsFavorite = message.IsFavorite;
            }
        }
    }
    ```

    通过实现`CommunityToolkit.Mvvm.Messaging.IRecipient<TMessage>`接口，其中`TMessage`在这种情况下是`FavoriteUpdateMessage`，我们指定了我们想要处理的消息类型。实现此接口允许我们调用`WeakReferenceMessenger`的`Register`方法，并将类本身作为唯一参数传递。该接口要求我们实现`Receive`方法，当接收到指定类型的消息时，该方法会被调用。

通过更新的代码，每当用户将食谱添加或删除为收藏时，都会发送一条消息。`RecipeListItemViewModel`的实例被设置为监听此消息并相应地更新它们的`IsFavorite`属性。因此，当用户从更新收藏状态的详细页面返回时，刷新的状态会立即在概览页上可见——而无需重新加载数据。

注意

虽然`WeakReferenceMessenger`为许多消息传递场景提供了强大的解决方案，但在处理大量监听者时，使用它时需要谨慎。始终监控应用程序的性能和行为，尤其是在向数千个监听者发送消息时，如果需要，考虑优化或重新评估你的设计。

# 摘要

在本章中，我们探讨了现代应用程序架构中的两个关键主题：依赖注入（DI）和消息传递。首先，我们探讨了依赖注入，这是一种在对象及其依赖项之间实现松耦合的技术。在 MVVM 模式中，我们利用这项技术将服务和其它依赖项注入到我们的 ViewModel 中，从而增强了它们的可测试性和可维护性。

本章的后半部分专注于消息传递，这是 MVVM 应用程序中另一个重要的组件，用于在组件之间促进解耦通信。我们考察了 MVVM Toolkit 提供的`WeakReferenceMessenger`，它有助于在应用程序中实现松耦合。

本质上，本章旨在强调软件设计中松耦合的重要性，展示了依赖注入和消息传递如何对创建可维护和可测试的应用程序做出重大贡献。

在接下来的章节中，我们将深入探讨.NET MAUI 中导航的复杂性以及如何将导航集成到我们的 MVVM 架构中。

# 进一步阅读

要了解更多关于本章所涉及的主题，请查看以下资源：

+   依赖注入: [`learn.microsoft.com/dotnet/architecture/maui/dependency-injection`](https://learn.microsoft.com/dotnet/architecture/maui/dependency-injection)

+   MVVM Toolkit 消息传递器: [`learn.microsoft.com/dotnet/communitytoolkit/mvvm/messenger`](https://learn.microsoft.com/dotnet/communitytoolkit/mvvm/messenger)
