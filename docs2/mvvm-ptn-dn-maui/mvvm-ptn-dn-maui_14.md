# 14

# 故障排除和调试技巧

恭喜你在掌握.NET MAUI 中 MVVM 模式的旅程中走这么远！到目前为止，你已经了解了数据绑定、依赖注入、转换器以及构成你的*Recipes!*应用的各个组件的复杂性。然而，正如任何经验丰富的开发者都会告诉你的，即使是经验最丰富的专家有时也会遇到障碍。

尽管 MVVM 拥有所有这些好处，但它有时可能感觉像是在一个复杂的迷宫中导航。当你遇到问题时，并不总是明显知道要找到根本原因或如何修复它。这正是本章的作用所在。在本章这个简短但非常有价值的章节中，我们将揭示你在 MVVM 之旅中可能遇到的常见陷阱和挑战。

我们将探讨三个经常出现问题的领域：

+   常见的数据绑定问题

+   服务和依赖注入陷阱

+   频繁的自定义控件和转换器问题

让我们开始解决这些常见障碍的旅程。到本章结束时，你将更好地装备自己，以解决.NET MAUI 中 MVVM 可能带来的挑战。

# 技术要求

为了确保你与即将到来的内容保持同步，请前往我们的 GitHub 仓库[`github.com/PacktPublishing/MVVM-pattern-.NET-MAUI/tree/main/Chapter14`](https://github.com/PacktPublishing/MVVM-pattern-.NET-MAUI/tree/main/Chapter14)。从`Start`文件夹中的材料开始。记住，如果你需要综合参考，`Finish`文件夹在章节结束时包含了最终的、精炼的代码。

# 常见的数据绑定问题

MVVM 模式的一个基石是数据绑定。它形成了你的 View 和 ViewModel 之间的联系，确保它们之间无缝的通信。虽然数据绑定提供了强大的功能，但它也是开发者经常面临挑战的领域。本节旨在阐明常见的数据绑定和 ViewModel 问题以及如何解决它们：

+   **错别字和名称不匹配**：开发者遇到的最简单但意外常见的问题之一是错别字或属性名称不匹配。你 XAML 标记或 ViewModel 代码中的一个小错误可能会破坏整个数据绑定过程。

+   `OneWay`、`TwoWay`和`OneTime`，每个都有其自己的用途，正如我们在*第四章*中看到的，*在.NET MAUI 中的数据绑定*。使用错误的模式可能导致你的应用出现意外的行为。

+   尽管 XAML 在某些情况下支持隐式类型转换，但将`int`转换为 UI 控件上的`Color`属性类型将不起作用。

+   `PropertyChanged`事件。当这些事件没有按预期触发时会发生什么？View 将不会更新以反映 ViewModel 数据中的更改。

+   `ObservableCollection`。确保你的集合正确更新，并避免无意中分配新的`ObservableCollection`。

+   `ListView`或`CollectionView`，请注意，集合中的每个项目都会创建自己的数据绑定作用域。这意味着当尝试绑定位于 ViewModel 而不是单个项目中的属性或命令时，您需要使用相对或元素绑定等技术来正确引用所需上下文。

+   **在行为中的数据绑定**：行为存在于视觉树之外，这意味着它们没有与其他 UI 元素相同的定位祖先的能力。

注意

通过利用编译绑定，如*第四章*中所述的，*在 .NET MAUI 中的数据绑定*，可以避免一些陷阱，例如属性名中的拼写错误和绑定不兼容的数据类型。

如您所见，与数据绑定相关的问题可能很多。幸运的是，其中一些问题并不难发现和修复，只要您知道该往哪里看。让我们从查看 Visual Studio 中的某些工具开始。

## 检查输出和 XAML 绑定失败窗口

在 Visual Studio 中，使用**输出**窗口和**XAML 绑定失败**窗口，可以轻松地发现拼写错误或属性名不匹配。以下是方法：

1.  让我们先在我们的代码中引入两个错误的数据绑定语句。在`RecipesOverviewPage`中，更新`CollectionView`的`ItemTemplate`中的`Image`和`Label`元素，如下所示：

    ```cs
    <CollectionView.ItemTemplate>
        <DataTemplate>
            ...
                <Image
                    Aspect="AspectFill"
                    HorizontalOptions="Fill"
                    Source="{Binding IsFavorite}"
                    VerticalOptions="Fill" />
                ...
                <Label
                    Grid.Row="1"
                    Margin="20,5,20,40"
                    FontSize="16"
                    HorizontalOptions="Fill"
                    HorizontalTextAlignment="Start"
                    MaxLines="2"
                    Text="{Binding Titel}"
                    TextColor="Black"
                    VerticalOptions="Center" />
            ...
        </DataTemplate>
    </CollectionView.ItemTemplate>
    ```

1.  运行应用。当应用正在运行时，前往 Visual Studio 并打开**输出**窗口。如果尚未打开，您可以通过**调试**|**窗口**|**输出**来打开它。注意以下内容：

    ```cs
    [0:] Microsoft.Maui.Controls.Xaml.Diagnostics.Binding
    Diagnostics: Warning: 'False' cannot be converted to
    type 'Microsoft.Maui.Controls.ImageSource'
    [0:] Microsoft.Maui.Controls.Xaml.Diagnostics.Binding
    Diagnostics: Warning: 'Titel' property not found on
    'Recipes.Client.Core.ViewModels.RecipeListItemView
    Model', target property: 'Microsoft.Maui
    bool value cannot be converted to an ImageSource type. The second warning signals that a property named Titel cannot be found on the RecipeListItemViewModel. That, of course, is a typo!
    ```

1.  或者，让我们看看 Visual Studio 中的**XAML 绑定失败**窗口。可以通过**调试**|**窗口**|**XAML 绑定失败**来打开。*图 14*.1 展示了此窗口的外观：

![图 14.1：XAML 绑定失败窗口](img/B20941_14_01.jpg)

图 14.1：XAML 绑定失败窗口

此窗口提供的信息与**输出**窗口相同。此外，它还显示了额外信息，例如失败的绑定语句所在位置以及此问题发生次数。这个窗口最好的地方是什么？当点击此列表中的项目时，Visual Studio 将打开包含错误绑定语句的 XAML 文件，并将指针放在包含错误的确切数据绑定语句上。

无论何时您的应用程序中出现绑定失败，Visual Studio 中的**XAML 绑定失败**窗口和**输出**窗口都会提供有关错误的信息。特别是**XAML 绑定失败**窗口，可以立即提供关于拼写错误、缺失属性或数据类型问题的洞察。在开发视图时，始终关注此窗口。

另一种解决或调试数据绑定问题的方法是创建并利用专门的转换器。让我们看看吧！

## 使用 DoNothingConverter 进行调试

`DoNothingConverter` 是一个宝贵的调试工具。通过将其放置在绑定管道中，您可以检查绑定过程中传递的值。如果您看到意外的值或根本没有值，这可以帮助确定故障发生的位置。以下是 `DoNothingConverter` 的实现：

```cs
public class DoNothingConverter : IValueConverter
{
    public object Convert(object value,
        Type targetType, object parameter,
        CultureInfo culture)
    {
        // Break here to inspect value during debugging
        return value;
    }
    public object ConvertBack(object value,
        Type targetType, object parameter,
        CultureInfo culture)
    {
        // Break here to inspect value during debugging
        return value;
    }
}
```

要添加和使用此转换器到您的绑定语句中，请按照以下步骤操作：

1.  将 `DoNothingConverter` 添加到您想要调试绑定语句的页面的 `Resources` 中。以下是将其添加到 `RecipesOverviewPage` 的方法：

    ```cs
    <ContentPage
        x:Class="Recipes.Mobile.RecipesOverviewPage"
        ...
        xmlns:conv="clr-namespace:
          Recipes.Mobile.Converters"
        ... >
        <ContentPage.Resources>
            ...
            <conv:DoNothingConverter
                x:Key="doNothingConverter" />
        </ContentPage.Resources>
        ...
    </ContentPage>
    ```

1.  将转换器添加到您想要调试的绑定语句中，如下所示：

    ```cs
    <Image
        ...
        Source="{Binding IsFavorite,
        Converter={StaticResource doNothingConverter}}"
        VerticalOptions="Fill" />
    ...
    <Label
        ...
        Text="{Binding Titel,
        Converter={StaticResource doNothingConverter}}"
        TextColor="Black"
        VerticalOptions="Center" />
    ```

1.  在 `DoNothingConverter` 的 `Convert` 或 `ConvertBack` 方法中插入断点。

1.  如果在运行时遇到断点，这表明成功绑定到 ViewModel 上的现有属性。您会注意到，在 `Convert` 方法中的断点不会因为 `Titel` 绑定而触发，因为这个属性不存在。

1.  如果在属性值后续更新时没有遇到断点，请检查语句的绑定模式，并确保当属性更新时触发 `PropertyChanged` 方法。

1.  当断点被触发时，您可以轻松检查绑定的值并将其与您的预期进行比较。

1.  您还可以检查 `targetType` 参数，它表示目标属性的类型。请注意，虽然 XAML 在某些情况下支持隐式类型转换，但了解支持的特定转换是至关重要的。

1.  当 UI 控件上的属性更新且绑定模式设置为 `TwoWay` 或 `OneWayToSource` 时，应调用 `ConvertBack` 方法。如果您期望这能正常工作，但 `ConvertBack` 方法没有调用，请检查绑定语句的绑定模式。

通过遵循这些步骤并使用 `DoNothingConverter` 工具，您可以有效地解决 MVVM 应用中的数据绑定问题。

让我们讨论另一个可能导致数据绑定问题的原因：集合。

## 解决集合问题

当与集合一起工作时，尤其是 `ObservableCollection`，开发者经常会遇到与更新和绑定相关的挑战。

如果您使用 `ObservableCollection` 或任何实现 `INotifyCollectionChanged` 接口的集合，它通常在 ViewModel 的初始化过程中分配一次。这里有一个需要记住的重要细节：这个属性的设置器不会触发 `PropertyChanged` 事件。相反，当您向集合中添加或删除项目时，它会在集合实例上触发 `CollectionChanged` 事件。此事件随后更新绑定的控件，假设它支持绑定到 `ObservableCollection`。要验证特定控件是否与 `INotifyCollectionChanged` 接口良好协作，请查阅控件文档。

然而，有一个关键点需要注意：如果重新分配 `ObservableCollection`，绑定将实际上会丢失，除非当然，`PropertyChanged` 事件被正确触发。这意味着如果您使用 `ObservableCollection` 的新实例重新分配整个集合，您需要确保 `PropertyChanged` 事件被正确触发。为了检查此事件是否有效触发，您可以使用 `DoNothingConverter`。

相比之下，当您处理一个没有实现 `INotifyCollectionChanged` 的集合（例如标准 `List` 或类似的集合）时，添加或删除项目将不会被 UI 层自动检测。在这种情况下，当向集合中添加或从集合中删除项目时，必须显式触发 `PropertyChanged` 事件。因此，当您进行更改时，整个列表将在 UI 中重新渲染。

在处理与集合相关的问题时，请密切关注您是否正在使用 `ObservableCollection` 或非可观察集合，并确保您触发了适当的事件以保持 ViewModel 和 UI 保持同步。理解这些动态将帮助您更有效地导航 MVVM 应用程序中集合的复杂性，并防止潜在的问题。

当与集合一起工作时，请记住，当集合中某个项目的属性发生变化时，您不需要在集合本身上触发 `PropertyChanged` 事件。相反，关键在于在经过修改的特定项目的实例上引发 `PropertyChanged` 事件。这确保了 UI 被通知到项目级别的更改，并准确地反映了更新后的状态。本质上，您正在将更新事件精确地集中在需要的地方，最小化对整个集合的不必要更新。

## 在 Behaviors 上的数据绑定陷阱

在编写 XAML 时很容易忽略这一点，但相对源绑定在 Behaviors 上不起作用。这是因为 Behaviors 存在于视觉树之外。实际上，一个 Behavior 可以被多个 UI 元素重用，因此相对源绑定将无法检索父对象。当将相对源绑定应用于 Behavior 时，您的应用程序将崩溃，并在崩溃之前出现类型为 `System.InvalidOperationException` 的异常。该异常指出以下内容：**由于对象当前的状态，操作无效**。这个异常以及此消息应该是一个迹象，表明在 Behavior 上定义了一个错误的数据绑定语句。在异常或 **输出** 窗口中将没有任何进一步的指示。您唯一能做的就是系统地遍历代码中的 Behaviors 并查看它们的绑定语句。

在许多情况下，相对源绑定可以被元素绑定所替代，如下所示：

```cs
<!-- RelativeSource binding fails on Behaviors! -->
<toolkit:IconTintColorBehavior
    TintColor="{Binding IsFavorite,
    Source={RelativeSource AncestorType={x:Type local:
RelativeSource binding in the IcontTintColorBehavior. This can be bypassed by leveraging element binding, as shown in the next code block:

```

<ContentView

...

x:Name="root">

...

<toolkit:IconTintColorBehavior

TintColor="{Binding IsFavorite,

源={x:Reference root},

转换器=...}" />

...

</ContentView>

```cs

 Next, let’s discuss the things to look out for when working with Dependency Injection.
Services and Dependency Injection pitfalls
In your MVVM journey, DI plays a crucial role in providing essential functionality to your application. However, even in the world of DI, there can be pitfalls waiting to catch you off guard. This section is dedicated to unveiling the most common pitfalls and equipping you with the knowledge to navigate them effectively.
Unable to resolve service for type
A `System.InvalidOperationException` stating `RecipesOverviewViewModel` in the DI container:
![Figure 14.2: InvalidOperationException thrown](img/B20941_14_02.jpg)

Figure 14.2: InvalidOperationException thrown
The exception gives you all the information you need as it clearly states what type is missing while trying to create a particular type.
Registering the missing dependency in the `MauiProgram` class (or anywhere you do your registrations) should fix the issue.
Let’s have a look at another common exception in the context of DI.
No parameterless constructor defined for type
A `System.MissingMethodException` can be thrown in the following scenario:

*   Shell is used to perform navigation
*   The BindingContext (a ViewModel) of a page is injected through the page’s constructor
*   The page isn’t registered in the DI container

As long as a page doesn’t have any dependencies that need to be injected through the constructor, it doesn’t need to be registered in the DI container, as its default constructor is being used by Shell to instantiate the page. However, when the page has one or more dependencies, we need to register it in the DI container. That way, Shell can ask the container to resolve an instance of the needed page.
Registering the page in the DI container solves this issue.
A much more subtle pitfall when it comes to DI is not registering the services appropriately. Let’s have a look.
Incorrect service registration
In the context of DI, one common pitfall stems from improperly registering services, leading to issues that can affect your application’s functionality:

*   **Resource intensiveness**: If you register a service as transient when it should be a singleton, you may encounter resource-intensive behavior. This occurs because a new instance of the service is created every time it’s requested. For services that involve resource-intensive operations, such as establishing database connections or managing file handles, this frequent creation can lead to performance bottlenecks and resource exhaustion. Such issues can significantly impact your application’s performance and stability.
*   **Unintended shared state**: Conversely, if you mistakenly register a service as a singleton when it should be transient, you may inadvertently introduce an unintended shared state. In this scenario, changes made to the service’s state or properties affect all parts of your application that depend on that service. This shared state can lead to unpredictable behavior and make debugging challenging, as the source of the problem may not be immediately apparent. It’s crucial to align the service’s registration with its intended usage to avoid such pitfalls.

To mitigate these issues, carefully consider the intended scope and usage of each service during registration. Ensure that services requiring a single shared instance across your application are registered as singletons, while services that should have unique instances for each request are registered as transient. By making informed decisions about service registration, you can prevent these common pitfalls and ensure your application functions as intended.
In the final section, let’s have a look at common problems around custom controls and value converters.
Frequent custom control and converter problems
Most of the issues that arise when working with custom controls regularly have to do with bindable properties. Often, a small typo or a little oversight might cause your custom control to not react as expected or to display the wrong data.
Troubleshooting bindable properties
On a custom control, there is a lot of ceremony needed to define bindable properties. It’s very easy to make a mistake that is very hard to spot when troubleshooting. Here are a couple of things to look out for:

*   The `propertyName` parameter in the `Create` method: Make sure the `propertyName` parameter matches the exact naming of the property:

    ```

    public static readonly BindableProperty

    IsFavoriteProperty =

    可绑定属性创建(nameof(IsFavorite), …);

    public bool IsFavorite

    {

    ...

    }

    ```cs

    As this code sample shows, it is advised to use the `nameof` expression to prevent typos!

     *   The `returnType` parameter in the `Create` method: The second parameter of the `BindableProperty`’s `Create` method is the `returnType`, which must match the type of the property:

    ```

    public static readonly BindableProperty

    IsFavoriteProperty =

    可绑定属性创建(nameof(IsFavorite),

    typeof(bool), …);

    public bool IsFavorite

    {

    ...

    }

    ```cs

     *   The `declaringType` parameter in the `Create` method: This parameter should be the type of the class where the property is defined:

    ```

    public partial class FavoriteControl : ContentView

    {

    public static readonly BindableProperty

    IsFavoriteProperty =

    BindableProperty.Create(...

    typeof(FavoriteControl),...);

    public bool IsFavorite

    {

    ...

    }

    ```cs

     *   It’s also important to make sure the getter and setter of the property call the `GetValue` and `SetValue`, passing in the correct `BindableProperty`:

    ```

    public static readonly BindableProperty IsFavoriteProperty = ...

    public bool IsFavorite

    {

    get => (bool)GetValue(IsFavoriteProperty);

    set => SetValue(IsFavoriteProperty, value);

    }

    ```cs

Whenever there is a discrepancy between the provided values in the `BindableProperty`’s `Create` method and the values on the control itself, or when the property doesn’t get or set the value correctly on the `BindableProperty`, the bindable property will not work as expected. So, it’s crucial to double-check these values!
Binding to the BindingContext
As already stipulated in *Chapter 11*, *Creating MVVM-Friendly Controls,* it is crucial that custom controls don’t depend on their `BindingContext`! The reason is that you can’t control that, as it is inherited from the parent the custom control is used on. Instead, you should only bind to the (bindable) properties that you’ve defined on the control itself. This can easily be achieved by leveraging relative or element binding, just like we did with the `FavoriteControl`:

```

<Image

HeightRequest="{Binding HeightRequest,

源={x:Reference icon}}"

IsVisible="{Binding IsInteractive,

源={RelativeSource AncestorType={x:Type

local:FavoriteControl}}}"

... />

```cs

 Any binding statements on a custom control that don’t have an explicit source set will bind to the `BindingContext` of the parent, which we don’t control. When a custom control works in one place but not in the other, chances are high that there is some binding going on that is not relative to the control itself. So, always double-check the binding statements in your custom control!
Finally, let’s have a quick look at the issues that might arise when working with value converters.
Value converter issues
Converters play a crucial role in data transformation within your app. However, their logic might not always behave as expected. It’s a seemingly trivial issue, but one that is frequently underestimated. The solution? Simple yet powerful: write unit tests! In *Chapter 13*, *Unit Testing,* we’ve highlighted how easy it is to unit test value converters. Paying attention to the logic within converters, testing them rigorously, and handling special cases will ensure that your converters perform reliably.
Summary
Now that we’ve reached the end of this short chapter, I hope you’ve gained valuable insights and tips for effectively troubleshooting issues that can arise in an MVVM context. Remember, the road to mastering MVVM is an ongoing journey, and troubleshooting and debugging are indispensable companions on this path. These challenges, though sometimes frustrating, are valuable teachers that will deepen your understanding and proficiency in MVVM. Embrace them as opportunities to grow, and in doing so, you’ll become a more proficient and confident MVVM developer. Your journey doesn’t end here; it evolves with each issue you resolve.
As we wrap up this final chapter, I want to extend my heartfelt congratulations to you for completing this book’s journey into the world of MVVM in .NET MAUI. Throughout this book, you’ve delved into the intricacies of the MVVM pattern, explored the capabilities of .NET MAUI, and built your very own *Recipes!* app.
Once again, congratulations on your accomplishment, and may your MVVM and .NET MAUI journey continue to be rewarding and filled with exciting projects!
Further reading
To learn more about the topics that were covered in this chapter, take a look at the following resource:
*XAML data binding* *diagnostics*: [`learn.microsoft.com/en-us/visualstudio/xaml-tools/xaml-data-binding-diagnostics?view=vs-2022`](https://learn.microsoft.com/en-us/visualstudio/xaml-tools/xaml-data-binding-diagnostics?view=vs-2022)

```
