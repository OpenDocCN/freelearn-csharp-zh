

# .NET MAUI 中的数据绑定构建块

在前面的章节中，我们熟悉了 MVVM 模式的核心概念，并探讨了.NET MAUI 的基础知识。在了解了 MVVM 原则和.NET MAUI 的能力之后，我们现在可以开始探讨如何将 MVVM 应用于.NET MAUI。

数据绑定是.NET MAUI 中的关键组件，它是 MVVM 模式的关键推动者。在本章中，我们将重点关注促进.NET MAUI 中数据绑定的基本概念、组件和技术。这些关键元素连接了应用程序的视图和 ViewModel 层，实现了高效的通信并确保了关注点的清晰分离。

在本章中，我们将涵盖以下主题：

+   数据绑定的关键组件

+   绑定模式和**INotifyPropertyChanged**接口

+   处理与**ICommand**接口的交互

到本章结束时，你将深入理解.NET MAUI 附带的基本数据绑定构建块。这将帮助你理解.NET MAUI 中数据绑定的内部工作原理以及每个组件的作用。有了这个基础，你将准备好探索下一章中更高级的主题和技术。

# 技术要求

在本章中，我们将向**Recipes!**应用添加功能。所有必要的资源，包括本章中使用的所有代码，都可以在 GitHub 上找到，网址为[`github.com/PacktPublishing/MVVM-pattern-.NET-MAUI/tree/main/Chapter03`](https://github.com/PacktPublishing/MVVM-pattern-.NET-MAUI/tree/main/Chapter03)。

# 数据绑定的关键组件

让我们首先关注使.NET MAUI 中的数据绑定成为可能的核心组件：`BindableObject`、`BindableProperty`和`BindingContext`。这些组件协同工作，建立并管理您的视图和 ViewModel 之间的连接。理解这些元素的作用和功能至关重要，因为它们构成了.NET MAUI 中数据绑定的骨架。

让我们快速讨论在数据绑定中起作用的元素。

## 数据绑定元素

在我们深入关键组件之前，让我们回顾一下为了在.NET MAUI 应用程序中有效地使用数据绑定，我们需要理解哪些元素。这些元素在促进视图和 ViewModel 层之间的通信中发挥着至关重要的作用，使得数据和使用者交互的无缝同步成为可能：

+   `INotifyPropertyChanged`接口。此接口确保每当 ViewModel 中的数据发生变化时，视图都会收到通知，从而允许 UI 相应地更新。重要的是要理解，实现`INotifyPropertyChanged`接口并非使属性作为绑定源所必需的。实际上，任何属性，无论其封装类是否实现了`INotifyPropertyChanged`，都可以作为绑定源。

+   你可能想要连接到绑定源的 UI 元素（或另一个`BindableObject`）上的`BindableProperty`。在.NET MAUI 中，大多数 UI 元素，如标签、文本框和按钮，都从`Microsoft.Maui.Controls.BindableObject`类派生，这使得它们能够作为绑定目标。与绑定源不同，并非每个属性都可以作为绑定目标，只有从`BindableObject`派生的类上的`BindableProperty`类型属性才能作为绑定目标。

+   **绑定上下文**：绑定上下文建立了视图模型和视图之间的关系。它作为数据绑定引擎的参考点，提供对视图模型实例的连接。绑定上下文通常在页面级别或单个 UI 元素上设置。默认情况下，子元素从其父元素继承上下文。

+   **绑定路径**：绑定路径是一个表达式，指定了需要绑定到绑定目标的绑定源属性。在最简单的情况下，绑定路径指的是视图模型中的单个属性名，但它也可以包括更复杂的表达式，例如属性链或索引器。绑定路径和绑定上下文的组合构成了绑定源。

+   **绑定模式**：绑定模式决定了绑定源和目标之间数据流的方向，正如我们在*第一章*“什么是 MVVM 设计模式？”中看到的，它决定了数据流的流向。

+   **值转换器**：值转换器修改 ViewModel 和视图之间的数据，反之亦然。它允许我们转换正在绑定的数据值，当 ViewModel 的数据类型与视图中的 UI 元素期望的数据类型不匹配时，特别有用。

在本节和*第四章*“.NET MAUI 中的数据绑定”中，将全面讨论数据绑定的各个方面。

让我们先看看使.NET MAUI 中的数据绑定成为可能的核心组件。

## BindableObject

在.NET MAUI 中，`Microsoft.Maui.Controls.BindableObject`类是利用数据绑定的对象的基类。它通过实现与绑定过程相关的必要属性、方法和事件，为通过 UI 元素和其他对象启用数据绑定提供了基础。实际上，它是.NET MAUI 中数据绑定功能的核心，也是应用 MVVM 模式的关键组件。它使我们能够连接视图和视图模型，使它们能够相互通信并保持同步，而无需直接耦合。

.NET MAUI 中的大多数 UI 元素，如标签、按钮和文本框，都从`BindableObject`类派生。这种继承使得这些元素能够参与数据绑定。

从本质上讲，`BindableObject`存储了`Microsoft.Maui.Controls.BindableProperty`的实例并管理`BindingContext`。这些是启用有效数据绑定的重要方面，从而促进了视图和视图模型之间的无缝通信，促进了 MVVM。

## 可绑定属性

`BindableProperty`是一种特殊的属性，可以作为绑定目标。它基本上是一个高级 CLR 属性，如果你愿意的话，就是一个“强化”的属性。最终，可绑定属性充当存储与属性相关的数据的容器，例如其默认值和验证逻辑。它还提供了一个机制来启用数据绑定和属性更改通知。它与一个公共实例属性相关联，该属性作为获取和设置可绑定属性值的接口。在.NET MAUI 中，许多 UI 元素的属性都是可绑定属性，这使得我们可以通过数据绑定或使用样式来设置它们的值。

例如，让我们看看`Label`的`Text`属性：

```cs
public partial class Label : View ...
{
    ....
    public static readonly BindableProperty TextProperty =
        BindableProperty.Create(nameof(Text),
            typeof(string), typeof(Label), default(string),
            propertyChanged: OnTextPropertyChanged);
    ...
    public string Text
    {
        get { return (string)GetValue(TextProperty); }
        set { SetValue(TextProperty, value); }
    }
    static void OnTextPropertyChanged(
        BindableObject bindable,
        object oldvalue, object newvalue)
    {
        ...
    }
    ...
}
```

让我们从查看`Text`属性开始。这只是一个 CLR 属性，但它不是从后端字段存储和检索值，而是执行以下操作：

+   获取器通过调用`GetValue`方法并传入`TextProperty`来检索值。这个`GetValue`方法是在`Label`类继承的`BindableObject`基类上的一个方法。

+   设置器调用`BindableObject`的`SetValue`方法，传入`TextProperty`和给定的值。

在`GetValue`和`SetValue`方法中使用的`TextProperty`是一个`BindableProperty`。它是静态的，属于`Label`类。如前所述，`BindableProperty`是一种“容器”，它包含有关属性的各种信息，包括其值。

让我们分解这个`TextProperty`。这个属性被定义为`BindableProperty`类型的公共静态字段。它通过调用`BindableProperty.Create`方法并传入以下值来实例化：

+   `BindableProperty`字段。在这个特定的案例中，`nameof(Text)`用于引用`Label`类的`Text`属性。

+   `Type`定义了可绑定属性所引用的属性的数据类型。由于`Text`属性是`string`类型，因此传递了`typeof(string)`作为值。

+   `Type`。它指的是属性的拥有者类型，即属性所属的类。在这个特定的例子中，`Text`属性是在`Label`类中定义的，因此这个类型作为值传递。

+   `defaultValue`参数，其类型为`object`。这是一个可选参数，允许我们为可绑定属性提供一个默认值。在这种情况下，传递了`default(string)`，这意味着默认值将是`NULL`。

+   **OnTextpropertyChanged**：可选地，我们可以提供一个在属性值更改时需要调用的委托。

`BindableProperty.Create`方法有很多可选参数，如果需要的话，你可以提供这些参数。

更多关于可绑定属性的信息

在 *第十一章*，*创建 MVVM 友好的控件*，我们将探讨如何构建我们自己的控件。理解 `BindableProperty` 的概念对于创建 MVVM 友好的控件至关重要。因此，你可以期待在那个章节中更深入地了解这一点。

`BindableProperty` 的概念一开始可能会让人感到有些令人困惑或难以理解。目前，只需记住所有 UI 元素都继承自 `BindableObject`，并且它们中的许多属性都是可绑定属性，这使得我们可以将它们用作绑定目标。

## BindingContext

`BindableObject` 有一个类型为 `object` 的 `BindingContext` 属性，它充当绑定源和绑定目标之间的粘合剂。它将数据绑定引擎指向一个充当绑定源的类的实例。当你在一个 `BindableObject` 上设置 `BindingContext` 属性，例如一个 .NET MAUI 页面或 UI 元素时，你指定了该对象作用域内数据绑定表达式的源对象。该作用域内的子元素将默认继承 `BindingContext`，除非为它们显式设置了不同的 `BindingContext`。

当在 `BindableObject` 上设置 `BindingContext` 时，数据绑定引擎将解析所有将 `Source` 设置为 `BindingContext` 的绑定。此外，当 `BindingContext` 属性的值发生变化时，这些绑定将使用更新的 `BindingContext` 重新评估。

既然我们已经讨论了所有允许我们在 .NET MAUI 中进行数据绑定的核心组件，让我们看看如何定义和使用它们。

## 实践中的数据绑定

在我们开始编写数据绑定之前，我们首先需要将我们的 ViewModels 添加到我们的解决方案中。为了完全接受关注点的分离，让我们将它们放入一个单独的项目中：

1.  在 **Solution Explorer** 中，右键单击你的解决方案，选择 **Add** | **New Project…**。

1.  从项目模板列表中选择 **Class Library** 并点击 **Next**。

1.  在 **Project name** 下输入 `Recipes.Client.Core` 并点击 **Next**。

1.  选择 `.NET 8.0` 作为 **Framework** 并点击 **Create**。

一旦创建了项目，让我们删除默认创建的 `Class1.cs`。接下来，我们想要将我们的 ViewModels 添加到这个项目中。为了保持一切井井有条，让我们首先创建一个 `ViewModels` 文件夹来放置所有的 ViewModels：

1.  在 `ViewModels` 中右键单击 `Recipes.Client.Core` 项目。

1.  右键单击新添加的文件夹，选择 `RecipeDetailViewModel`，并添加以下代码：

    ```cs
    namespace Recipes.Client.Core.ViewModels;
    public class RecipeDetailViewModel
    {
        public string Title { get; set; } = "Classic Caesar Salad";
    }
    ```

    `RecipeDetailViewModel` 表示菜谱的详细信息。目前，它只包含一个 `Title` 属性，我们现在给它一个硬编码的值 `"Classic` `Caesar Salad"`。

接下来，我们需要从 `Recipes.Mobile` 项目添加对 `Recipes.Client.Core` 项目的引用。为此，只需在 `Recipes.Mobile` 项目上右键单击，选择 `Recipes.Client.Core` 项目。

在我们深入数据绑定之前，作为最后一步，我们需要向我们的`Recipes.Mobile`项目添加一个新页面：

1.  右键单击项目名称，选择**添加** | **新建项…**。

1.  选择`RecipeDetailPage.xaml`。

1.  打开`App.xaml.cs`，并在构造函数中将`RecipeDetailPage`的一个实例分配给`MainPage`属性：

    ```cs
    public App()
    {
        InitializeComponent();
        MainPage = new RecipeDetailPage();
    }
    ```

    这确保了在启动时，移动应用将显示我们新创建的`RecipeDetailPage`。

1.  最后，在`RecipeDetailPage`的代码背后，我们需要将其`BindingContext`属性分配给`RecipeDetailViewModel`的一个实例：

    ```cs
    using Recipes.Client.Core.ViewModels;
    namespace Recipes.Mobile;
    public partial class RecipeDetailPage : ContentPage
    {
        public RecipeDetailPage()
        {
            InitializeComponent();
            BindingContext = new RecipeDetailViewModel();
        }
    }
    ```

现在，我们可以专注于`RecipeDetailPage`并开始实现一些数据绑定。数据绑定可以使用 C#和 XAML 在代码中定义。两者都有其优缺点，但本质上取决于个人喜好。如果你想的话，甚至可以混合使用这两种方法，尽管我不建议这么做。最常见的是在 XAML 中定义数据绑定，以最小化代码背后的代码量。不过，让我们首先看看如何使用 C#定义数据绑定。

### C#中的数据绑定

让我们转到我们的新`RecipeDetailPage.xaml`文件，并开始更新一些 XAML：

1.  从`ContentPage`标签内移除任何默认的 XAML 元素。

1.  在`ContentPage`内添加以下元素：

    ```cs
    <ScrollView>
        <VerticalStackLayout Padding="10">
        </VerticalStackLayout>
    </ScrollView>
    ```

    目前我们保持 UI 非常简单直接。这就是为什么我们将一切放入一个`VerticalStackLayout`中，它以垂直堆叠的方式组织子元素。我们用`ScrollView`包围`VerticalStackLayout`，以确保当屏幕上无法显示所有内容时，我们得到滚动条。

1.  在`VerticalStackLayout`中添加一个名为`lblTitle`的`Label`：

    ```cs
    <Label x:Name="lblTitle"
        FontAttributes="Bold" FontSize="22" />
    ```

    这个标签将显示菜谱的标题。

1.  在代码背后，在`RecipeDetailPage`的构造函数中，我们现在可以添加数据绑定代码，以便在`lblTitle`标签中显示`RecipeDetailViewModel`的`Title`属性值：

    ```cs
    lblTitle.SetBinding(
        Label.TextProperty,
        nameof(RecipeDetailViewModel.Title),
        BindingMode.OneWay);
    ```

    因为我们已经给`Label`命名为`lblTitle`，所以会生成一个具有此名称的字段，这允许我们从代码背后访问这个标签。通过调用`SetBinding`方法，我们可以定义并应用一个绑定。

1.  运行应用，你应该能在屏幕上看到`RecipeDetailViewModel`的`Title`，目前它是硬编码为`"Classic Caesar Salad"`。

通过查看我们刚刚实现的第一条数据绑定，你应该能够识别出我们之前讨论的大多数数据绑定元素：

+   `SetBinding`方法是在`BindableObject`类上的一个方法。`SetBinding`方法需要的第一个参数是一个`BindableProperty`。在这种情况下，我们传递的是静态的`Label.TextProperty`，它对应于`Label`的`Text`实例属性。我们调用`SetBinding`方法的`BindableObject`实例，以及我们作为第一个参数指定的`BindableProperty`，共同构成了绑定目标。

+   `SetBinding`方法期望的是一个要绑定的属性路径。这，加上`BindingContext`，形成了绑定源——但在这个情况下，`BindingContext`是什么？由于我们没有在`Label`上显式指定`BindingContext`，`BindingContext`是从`lblTitle`的父元素继承的，即`RecipeDetailPage`。因此，标签的`BindingContext`也是`RecipeDetailViewModel`的实例。

+   `BindingMode`是本例中的第三个参数。然而，这个参数是可选的。在这种情况下，我们将`BindingMode`设置为`OneTime`。在*第一章*，“什么是 MVVM 设计模式？”中，我们简要讨论了不同的绑定模式，我们将在本章稍后更深入地讨论绑定模式。

在代码中定义数据绑定既简单又直接。然而，大多数情况下，数据绑定是在 XAML 中定义的。我个人认为在 XAML 中定义它们感觉更自然，并且在使用 XAML 创建 UI 时需要更少的上下文切换。那么，让我们看看吧！

### XAML 中的数据绑定

由于我们将要在 XAML 中编写数据绑定，我们可以删除或注释掉在上一节*步骤 4*中添加到`RecipeDetailPage`构造函数中的绑定代码。

我们现在可以切换到`RecipeDetailPage.xaml`并更新标签，通过删除`x:Name`属性并添加包含**绑定****标记扩展**的`Text`属性：

```cs
<Label
    FontAttributes="Bold"
    FontSize="22"
    x:Name property. Of course, if you need a reference to the label for any other reason (for example, you have some animation logic in the code-behind that animates the label), you need to keep the x:Name property.
But more importantly, let’s take a look at the `Binding` markup extension that we’ve set to the `Text` property of the label.
XAML markup extensions
XAML markup extensions are a feature of XAML that allows you to provide values for properties during the parsing of the XAML markup more dynamically and flexibly. Markup extensions use curly braces (`{}`) to enclose their syntax and enable you to add more complex logic or functionality to the XAML itself.
Markup extensions can be used to reference resources, create bindings, or even instantiate objects, among other things. They allow you to extend the capabilities of the XAML language.
A lot more about markup extensions can be found in the docs: [`learn.microsoft.com/dotnet/maui/xaml/fundamentals/markup-extensions`](https://learn.microsoft.com/dotnet/maui/xaml/fundamentals/markup-extensions).
The `Binding` markup extension is used to create a data binding between the `Text` property of the label and the `Title` property of the binding source. The binding source, in this case, is an instance of `RecipeDetailViewModel`, which is set as the `BindingContext` of the control’s parent, `RecipeDetailPage`.
The `Path=Title` part of the binding expression specifies that the `Title` property from the binding source should be used as the source of the binding. In this scenario, you can omit `Path=` and simply use `Title`, as the binding expression is smart enough to recognize it as a property path. So, the binding expression can be written as follows:

```

<Label

FontAttributes="Bold"

FontSize="22"

Mode=, 我们可以指示绑定模式，就像我们在上一个示例中所做的那样；在这个示例中，我们将它设置为 OneTime。

除了`Path`和`Mode`之外，绑定标记扩展还有更多属性，例如`Source`，它允许我们指向另一个绑定源，`Converter`，`TargetNullValue`等。*第四章*，“.NET MAUI 中的数据绑定”将更详细地介绍所有这些内容。

现在，让我们更详细地看看不同的绑定模式以及如何自动在绑定目标中反映绑定源的变化。

绑定模式和 INotifyPropertyChanged 接口

在*第一章*，“什么是 MVVM 设计模式”中，我们已经讨论了数据如何流动：

+   **单向**：从 ViewModel 到 View

+   **单向到源**：从 View 到 ViewModel

+   **一次性**：仅从 ViewModel 到 View 一次

+   **双向**：从 ViewModel 到 View 和从 View 到 ViewModel

现在，让我们看看在.NET MAUI 中是如何处理这个问题的。

.NET MAUI 中的绑定模式

.NET MAUI 支持所有这些数据流，通过 `Microsoft.Maui.Controls.BindingMode` 枚举表示：`OneWay`、`OneWayToSource`、`OneTime` 和 `TwoWay`。实际上还有一个第五个值：`Default`。记得我们在本章前面讨论可绑定属性时提到的吗？在创建可绑定属性时，我们可以设置一些可选值。其中之一是 `defaultBindingMode`。这允许我们在可绑定属性上设置默认绑定模式。例如，在 `Entry` 上，将 `Text` 属性的默认绑定模式设置为 `TwoWay` 是有意义的，因为它显示了绑定源的价值，并且用户能够更新该值。另一方面，`Label` 上的 `Text` 属性是只读的，因此那里的默认绑定模式是 `OneWay`。现在，回到 `BindingMode` 枚举，特别是 `Default` 值，当我们没有指定绑定模式或使用 `Default` 时，数据绑定引擎将使用绑定目标的可绑定属性上指定的默认绑定模式。

在数据绑定和 .NET MAUI 中的各种绑定模式背景下，有一个高效的方法来通知 UI 关于 ViewModel 属性的更改至关重要。`INotifyPropertyChanged` 接口通过提供一个机制来满足这一需求，允许 ViewModel 在属性值更新时通知 UI。

INofityPropertyChanged

`INotifyPropertyChanged` 接口，作为 `System.ComponentModel` 命名空间的一部分，允许您的绑定源将属性更改通知给绑定目标。此接口并不仅限于 .NET MAUI；当绑定到对象实现 `INotifyPropertyChanged` 接口时，它是 .NET `PropertyChanged` 事件的一部分。当事件被触发时，引擎将相应地更新 UI。

要实现 `INotifyPropertyChanged` 接口，您的 ViewModel 必须包含一个 `PropertyChanged` 事件，该事件可以在属性值更改时触发。此外，通常创建一个方法，通常称为 `OnPropertyChanged`，该方法触发此事件并提供更改属性的名称作为参数。此方法应在属性设置器中调用，在属性值更新后立即调用：

```cs
public class SampleViewModel : INotifyPropertyChanged
{
    private string _title = string.Emtpty;
    public string Title
    {
        get => _title;
        set
        {
            if(_title != value)
            {
                _title = value;
                OnPropertyChanged(nameof(Title));
            }
        }
    }
    public void OnPropertyChanged(string propertyName)
        => PropertyChanged?.Invoke(this,
            new PropertyChangedEventArgs(propertyName));
    public event PropertyChangedEventHandler? PropertyChanged;
}
```

如前所述的示例所示，此 `SampleViewModel` 实现了 `INotifyPropertyChanged` 接口，这要求我们实现 `PropertyChanged` 事件的 `PropertyChangedEventHandler` 类型。可以通过调用 `OnPropertyChanged` 方法并传递已更改属性的名称来轻松调用 `PropertyChanged` 事件。在查看代码时，在 `Title` 属性的设置器中，当将新值分配给属性的备份字段时，会调用 `OnPropertyChanged` 方法，并传递属性的名称。在调用 `PropertyChanged` 事件时，通过 `this` 关键字将 `SampleViewModel` 的当前实例作为发送者传递，后跟一个 `PropertyChangedEventArgs` 实例，该实例需要更新的属性的名称。通过此事件，数据绑定引擎会收到已更新属性的通知，并且根据数据绑定模式，可以自动更新绑定目标。

`CallerMemberNameAttribute` 是在 `OnPropertyChanged` 方法的 `propertyName` 参数上非常常见的一种属性。此属性自动获取调用具有属性的方法或属性的名称。这使从属性的设置器调用 `PropertyChanged` 事件变得更加容易，因为不需要显式地作为参数传递属性的名称：

```cs
public class SampleViewModel : INotifyPropertyChanged
{
    ...
    public string Title
    {
        get => _title;
        set
        {
            if(_title != value)
            {
                _title = value;
                OnPropertyChanged();
            }
        }
    }
    public void OnPropertyChanged([CallerMemberName]string? propertyName = null)
        => PropertyChanged?.Invoke(this,
            new PropertyChangedEventArgs(propertyName));
...
}
```

由于 `Title` 属性调用了 `OnPropertyChanged` 方法，但没有提供作为参数的显式值，因此调用者的名称（在这种情况下为 `Title`）将被作为值传递。在 *第五章* 的 *社区工具包* 中，我们将看到如何在触发 `PropertyChanged` 事件时消除这里的大部分仪式。

不同的绑定模式在实际应用

在本章前面的示例中，我们将 `RecipeDetailViewModel` 的 `Title` 属性绑定了一次到 `Label` 的 `Text` 属性。这意味着仅在设置绑定目标的 `BindingContext` 或将 `BindingContext` 分配为新实例时，才设置绑定源。在我们的当前设置中，`BindingContext`，`Title` 属性的初始值立即绑定。然而，由于一次性数据绑定的性质，对 `Title` 属性的任何后续更改（例如在 ViewModel 加载数据后更新它）都不会反映在 UI 中。由于我们目前使用的是静态数据来演示，所以这不会成为问题，但在实际应用中，这是一个需要注意的问题。随着我们继续阅读本书，我们将修订此绑定模式，以更有效地处理此类情况。

让我们添加更多的代码并讨论其他绑定模式。首先，让我们添加一个额外的 `IngredientsListViewModel` 并将我们的 `RecipeDetailViewModel` 扩展为具有该类型的属性：

1.  在`Recipes.Client.Core`项目的`ViewModels`文件夹中，选择**添加** | **类…**。

1.  输入`IngredientsListViewModel.cs`作为名称并点击**添加**。

1.  让类实现`INotifyPropertyChanged`接口并添加`OnPropertyChanged`方法。将以下代码添加到新创建的类中：

    ```cs
    public class IngredientsListViewModel : INotifyPropertyChanged
    {
        public void OnPropertyChanged([CallerMemberName]string?  propertyName = null)
            => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        public event PropertyChangedEventHandler? PropertyChanged;}
    ```

    这是对`INotifyPropertyChanged`接口的典型实现，包括`PropertyChangedEvent`和一个触发已实现`PropertyChanged`事件的`OnPropertyChanged`方法。

    4.  接下来，我们可以添加`NumberOfServings`属性，如下面的代码片段所示。为了简洁，省略了`INotifyPropertyChanged`接口的实现：

    ```cs
    public class IngredientsListViewModel : INotifyPropertyChanged
    {
        private int _numberOfServings = 4;
        public int NumberOfServings
        {
            get => _numberOfServings;
            set
            {
                if(_numberOfServings != value)
                {
                    _numberOfServings = value;
                    OnPropertyChanged();
                }
            }
        }
    ...
        //ToDo: add list of Ingredients
    }
    ```

    `IngredientsListViewModel`包含所有食材及其所需数量的列表，以及一个`NumberOfServings`属性，该属性表示列表中食材及其数量所针对的份量数。用户应该能够在 UI 中调整份量数并看到根据所选数量更新食材数量的情况。我们将在稍后添加食材列表。

    5.  转到`RecipeDetailViewModel`并添加一个类型为`IngredientsListViewModel`的`IngredientsList`属性，并默认分配一个新实例：

    ```cs
    public IngredientsListViewModel
            IngredientsList { get; set; } = new ();
    ```

    在此更新后的代码到位后，我们现在可以转到`RecipeDetailPage`并再次关注 XAML。我们希望更新 UI 以允许用户选择所需的份量数。对于此用例，我们将创建一个`OneWay`和一个`TwoWay`数据绑定。

    前往`RecipeDetail`并在显示食谱标题的`Label`下方添加以下 XAML：

    ```cs
    <HorizontalStackLayout Padding="10">
        <Label Text="Number of servings:"
            VerticalOptions="Center" />
        <Label
            Margin="10,0" FontAttributes="Bold"
            Text="{Binding IngredientsList.NumberOfServings, Mode=OneWay}"
            VerticalOptions="Center" />
        <Stepper
            BackgroundColor="{OnPlatform WinUI={StaticResource Primary}}"
            Maximum="8" Minimum="1"
            Value="{Binding IngredientsList.NumberOfServings, Mode=TwoWay}" />
    </HorizontalStackLayout>
    ```

让我们看看通过这段代码添加的绑定语句：

+   标签的`Text`属性使用以下绑定语句绑定到`NumberOfServings`属性：`{Binding IngredientsList.NumberOfServings, Mode=OneWay}`。标签显示当前选定的份量数。`NumberOfServings`属性是`RecipeDetailViewModel`上的`IngredientsList`属性的一部分，因此绑定路径设置为`IngredientsList.NumberOfServings`。

    由于用户可以更改份量数，我们使用`OneWay`绑定模式。这确保了当`NumberOfServings`的值更新时，UI 会相应地反映变化。

+   `Stepper`控件的`Value`属性使用以下绑定语句绑定到`NumberOfServings`属性：`{Binding IngredientsList.NumberOfServings, Mode=TwoWay}`。`Stepper`允许用户调整当前选定的份量数。

    由于用户可以更改份量数，并且 ViewModel 应该相应更新，我们使用`TwoWay`绑定模式。这确保了不仅 UI 反映了`NumberOfServings`属性值，而且当用户通过`Stepper`控件修改份量数时，ViewModel 也会更新。

    使用`Stepper`对`NumberOfServings`属性进行的增加或减少将由于`OneWay`绑定而在之前讨论的标签中反映出来，并且该属性会触发`PropertyChanged`事件。

在 XAML 代码中，这两个绑定都针对`IngredientsList`属性内的属性。在这个例子中，`HorizontalStackLayout`的所有子元素都绑定到`RecipeDetailViewModel`中`IngredientsList`属性的属性。由于子元素从其父元素继承绑定上下文，如果需要，可以将绑定移动到`IngredientsList`属性的一个级别以上：

```cs
<HorizontalStackLayout Padding="10" BindingContext="{Binding IngredientsList}">
    <Label Text="Number of servings:"
        VerticalOptions="Center" />
    <Label
        Margin="10,0" FontAttributes="Bold"
        Text="{Binding NumberOfServings, Mode=OneWay}"
        VerticalOptions="Center" />
    <Stepper
        BackgroundColor="{OnPlatform WinUI={StaticResource Primary}}"
        Maximum="8" Minimum="1"
        Value="{Binding NumberOfServings, Mode=TwoWay}"/>
</HorizontalStackLayout>
```

在前面的例子中，我们将`HorizontalStackLayout`的`BindingContext`绑定到`RecipeDetailViewModel`的`IngredientsList`属性。这允许我们简化子元素上的绑定语句。当然，只有当所有子元素都具有相同的绑定源时，你才能这样做：

作为最后的例子，让我们在我们的**Recipes!**应用中实现一个`OneWayToSource`数据绑定。在这里，我们的目标是显示每个菜谱的过敏信息。由于并非每个人都对查看此信息感兴趣，我们不会默认显示它。然而，我们希望允许用户勾选复选框以获取此信息：

1.  前往`RecipesDetailViewModel`并让它实现`INotifyPropertyChanged`：

    ```cs
    public class RecipeDetailViewModel : INotifyPropertyChanged
    {
        ...
        public event PropertyChangedEventHandler? PropertyChanged;
    }
    ```

    2. 创建一个接受`propertyName`参数并调用`PropertyChanged`事件的`OnPropertyChanged`方法，就像我们在`IngredientsListViewModel`中做的那样：

    ```cs
    public void OnPropertyChanged([CallerMemberName] string? propertyName = null)
                => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    ```

    3. 现在，让我们添加一个类型为`bool`的`ShowAllergenInformation`属性。这个属性将负责在`RecipeDetailPage`上显示过敏信息的可见性：

    ```cs
    private bool _showAllergenInformation;
    public bool ShowAllergenInformation
    {
        get => _showAllergenInformation;
        set
        {
            if (_showAllergenInformation!= value)
            {
                _showAllergenInformation = value;
                OnPropertyChanged();
            }
        }
    }
    ```

    4. 当这个属性的值发生变化时，它会触发`PropertyChanged`事件。这允许我们将它绑定到`VisualElement`的`IsVisible`属性，这样每当 ViewModel 上的值发生变化时，UI 元素的可见性就会自动更新。

1.  最后，我们可以前往`RecipeDetailPage`并在显示菜谱标题的标签和绑定到`IngredientsList`的`HorizontalStackLayout`之间添加以下 XAML：

    ```cs
    <VerticalStackLayout Padding="10">
        <HorizontalStackLayout>
            <Label
                FontAttributes="Italic"
                Text="Show Allergen information"
                VerticalOptions="Center" />
            <CheckBox IsChecked="{Binding ShowAllergenInformation, Mode=OneWayToSource}" />
        </HorizontalStackLayout>
        <Label IsVisible="{Binding ShowAllergenInformation, Mode=OneWay}"
            Text="ToDo: add allergen information" />
    </VerticalStackLayout>
    "Show Allergen Information", followed by a checkbox that the user can toggle if they want to view the recipe’s allergen information. The IsChecked property is bound using the OneWayToSource mode to the ShowAllergenInformation property in the ViewModel. This means that when the user checks the box, the property will update accordingly.
    ```

    此外，我们使用`OneWay`模式将相同的`ShowAllergenInformation`属性绑定到标签的`IsVisible`属性。这个标签最终将显示过敏信息。因此，当用户切换复选框时，它会更新 ViewModel 中的`ShowAllergenInformation`属性，这反过来又触发了`PropertyChangedEvent`。这个事件将被绑定引擎捕获，它将更新标签的`IsVisible`属性，根据用户的偏好显示或隐藏过敏信息。

`INotifyPropertyChanged`不是绑定源的必要条件

重要的一点是要注意，绑定目标始终应该是继承自`BindableObject`的类的`BindableProperty`。另一方面，绑定源可以是任何类的任何属性。大多数情况下，绑定源类的实现`INotifyPropertyChanged`接口，但这不是必需的。只有在使用`OneWay`和`TwoWay`绑定模式，并且绑定源上的值可以更新并需要在 UI 中反映的情况下，才需要实现`INotifyPropertyChanged`接口。或者，重新设置`BindingContext`也会更新所有值，但这意味着需要重新评估一切，这可能不是一个好主意。

既然我们已经看到了.NET MAUI 如何支持数据绑定和不同的绑定模式，让我们看看我们如何处理用户交互。

使用 ICommand 接口处理交互

在大多数应用中，用户交互起着至关重要的作用。常见的交互包括点击按钮、从列表中选择项目、切换开关等。为了在遵循 MVVM 模式的同时有效地处理这些交互，利用一个封装必要逻辑在 ViewModel 中的强大机制是至关重要的。`ICommand`接口正是为此目的而设计的，它允许您以干净和可维护的方式管理用户交互，同时确保视图和 ViewModel 之间有明确的关注点分离。在本节中，我们将探讨如何实现和使用`ICommand`来处理.NET MAUI 应用程序中的用户交互。

ICommand 接口

在.NET MAUI 应用程序的 MVVM 模式上下文中，`ICommand`接口在处理用户交互方面发挥着关键作用。`ICommand`是`System.Windows.Input`命名空间的一部分，允许您在 ViewModel 中封装执行特定操作和确定该操作是否可以执行的逻辑。再次强调，此接口并不仅限于.NET MAUI；它是.NET BCL 的一个组成部分。

`ICommand`有两个主要成员：`Execute`和`CanExecute`：

+   **Execute (object 参数)**：当命令被调用时，将执行此方法。如果需要，可以通过可选参数传递额外的数据。

+   **CanExecute (object 参数)**：此方法指示命令在其当前状态下是否可以执行。如果需要，可以传递一个参数。

`ICommand`还公开了一个名为`CanExecute`方法更改的事件，该事件会通知 UI 重新评估命令是否可以执行。这使 UI 元素（如按钮）的自动启用/禁用成为可能，这些元素的状态基于应用程序的当前状态。

要在 ViewModel 中使用 `ICommand`，你可以创建一个实现 `ICommand` 接口的自定义命令类，或者使用内置的命令类，如 `Microsoft.Maui.Controls.Command`。还有来自 MVVM Toolkit 的第三方实现，例如 `CommunityToolkit.Mvvm.Input.RelayCommand`，我们将在 *第五章*，*社区工具包* 中更详细地探讨。通常，这些实现包含一个名为 `ChangeCanExecute` 或 `NotifyCanExecuteChanged` 的方法，这被称为 `CanExecuteChanged` 事件。

现在，你已经熟悉了 `ICommand` 接口及其在 ViewModel 中的作用，是时候看看它在实际中的应用了。

将其付诸实践

作为简单的演示，我们希望允许用户添加或删除菜谱作为收藏。为此，让我们向 `RecipeDetailPage` 添加两个按钮，位于包含显示过敏信息 `CheckBox` 元素的 `VerticalStackLayout` 下方：

```cs
<VerticalStackLayout>
    <Button Command="{Binding AddAsFavoriteCommand}" Text="Add as favorite" />
    <Button Command="{Binding RemoveAsFavoriteCommand}" Text="Remove as favorite" />
</VerticalStackLayout>
```

第一个按钮的 `Command` 属性绑定到我们的绑定源 `RecipeDetailViewModel` 上的 `AddAsFavoriteCommand` 属性。这个按钮应该允许用户在菜谱尚未收藏时将其标记为收藏。第二个按钮则正好相反：它应该允许用户取消收藏菜谱。让我们看看这两个命令的实现：

1.  在 `RecipeDetailViewModel` 中，我们可以添加一个 `IsFavorite` 属性：

    ```cs
    private bool _isFavorite = false;
    public bool IsFavorite
    {
        get => _isFavorite;
        private set
        {
            if (_isFavorite != value)
            {
                _isFavorite = value;
                OnPropertyChanged();
            }
        }
    }
    ```

    这个属性包含一个 `bool` 值，用来指示用户是否已将此菜谱标记为收藏。

    2.  接下来，我们需要添加两个命令，`AddAsFavoriteCommand` 和 `RemoveAsFavoriteCommand`：

    ```cs
    public ICommand AddAsFavoriteCommand
    {
        get;
    }
    public ICommand RemoveAsFavoriteCommand
    {
        get;
    }
    ```

    这些是按钮的 `Command` 属性绑定到的两个类型为 `ICommand` 的属性。

    3.  现在，我们需要实例化这两个命令。虽然 .NET 包含一个 `ICommand` 接口，但它没有具体的实现。另一方面，.NET MAUI 确实有一个实现！为了从 `Recipes.Client.Core` 项目访问这个实现，我们需要配置项目以使用 .NET MAUI 框架。

    在 `Recipes.Client.Core` 项目中。这应该会打开相关的 `csproj` 文件，其中你需要添加 `<UseMaui>true</UseMaui>`：

    ```cs
    <PropertyGroup>
      <TargetFramework>net8.0</TargetFramework>
      <ImplicitUsings>enable</ImplicitUsings>
      <UseMaui>true</UseMaui>
      <Nullable>enable</Nullable>
    </PropertyGroup>
    ```

    这使我们能够从我们的 `Core` 项目访问 .NET MAUI 特定的库，例如 `Microsoft.Maui.Controls.Command`，它实现了 `ICommand` 接口。

    4.  在 `RecipeDetailViewModel` 构造函数中，我们现在可以实例化 `AddAsFavoriteCommand` 和 `RemoveAsFavoriteCommand`：

    ```cs
    private void AddAsFavorite() => IsFavorite = true;
    private void RemoveAsFavorite() => IsFavorite = false;
    private bool CanAddAsFavorite()
        => !IsFavorite;
    private bool CanRemoveAsFavorite()
        => IsFavorite;
    public RecipeDetailViewModel()
    {
        AddAsFavoriteCommand =
            new Command(AddAsFavorite, CanAddAsFavorite);
        RemoveAsFavoriteCommand =
            new Command(RemoveAsFavorite, CanRemoveAsFavorite);
    }
    ```

    在这个例子中，我们在 ViewModel 中有两个命令：`AddAsFavoriteCommand` 和 `RemoveAsFavoriteCommand`。每个命令都使用一个相关的 `Action` 和一个 `Func<bool>` 来确定其可执行性。

    `AddAsFavoriteCommand` 有一个 `AddAsFavorite` 方法作为其操作，它只是将 `IsFavorite` 属性设置为 `true`。它的 `CanExecute` 方法由 `CanAddAsFavorite` 方法确定，当 `IsFavorite` 属性的值为 `false` 时返回 `true`。

    另一方面，`RemoveAsFavoriteCommand`有一个`RemoveAsFavorite`方法作为其操作，它将`IsFavorite`属性设置为`false`。`CanRemoveAsFavorite`方法提供为此命令的`CanExecute`检查，并且当`IsFavorite`属性值为`true`时返回`true`。

注意

需要注意的是，检查并遵守`CanExecute`方法的责任在于控件。它不应该是一个盲目依赖的东西，因为它可能没有实现或者没有按照你预期的样子工作。确保阅读控件的文档并彻底测试它。

总结来说，当`IsFavorite`属性为`true`时，只有`RemoveAsFavoriteCommand`可以执行，而`AddAsFavoriteCommand`不能执行。相反，当`IsFavorite`属性为`false`时，`AddAsFavoriteCommand`可以执行，而`RemoveAsFavoriteCommand`不能执行。这确保了根据`IsFavorite`属性当前的状态，可以执行适当的命令。

1.  只缺少一个拼图碎片：每当`IsFavorite`属性的值发生变化时，两个命令的`CanExecute`方法都需要重新评估。为了做到这一点，我们需要更新`IsFavorite`属性的设置器：

    ```cs
    if (_isFavorite != value)
    {
        _isFavorite = value;
        OnPropertyChanged();
        ((Command)AddAsFavoriteCommand).ChangeCanExecute();
        ((Command)RemoveAsFavoriteCommand).ChangeCanExecute();
    }
    ```

在所有这些代码就绪后，我们可以再次运行应用程序。默认情况下，当食谱不是用户喜欢的时，只有第一个按钮（绑定到`AddAsFavoriteCommand`）是启用的。点击此按钮后，`IsFavorite`属性被更新，并且两个命令的`ChangeCanExecute`方法被调用。结果，第一个按钮变为禁用，而第二个按钮（绑定到`RemoveAsFavoriteCommand`）自动启用。这确保了根据`IsFavorite`属性正确地启用或禁用按钮。

或者，我们也可以使用单个命令并使用`CommandParameter`来处理`IsFavorite`属性的切换：

1.  为了做到这一点，让我们添加一个新的`SetFavoriteCommand`属性，其类型为`ICommand`，并在`RecipeDetailViewModel`的构造函数中初始化它：

    ```cs
    public RecipeDetailViewModel()
    {
        ...
        SetFavoriteCommand =
            new Command<bool>(SetFavorite, CanSetFavorite);
    }
    private bool CanSetFavorite(bool isFavorite)
        => IsFavorite != isFavorite;
    private void SetFavorite(bool isFavorite)
        => IsFavorite = isFavorite;
    ```

    通过将命令分配给`Command<bool>`的实例，我们指定在`Execute`和`CanExecute`方法中我们期望一个布尔类型的参数。

    2. 在`IsFavorite`属性的设置器中，我们现在需要调用`SetFavoriteCommand`的`CanExecuteChanged`方法。

1.  最后，我们可以更新两个按钮的绑定语句：

    ```cs
    <Button
        Command="{Binding SetFavoriteCommand}"
        CommandParameter="{x:Boolean true}"
        Text="Add as favorite" />
    <Button
        Command="{Binding SetFavoriteCommand}"
        CommandParameter="{x:Boolean false}"
        Text="Remove as favorite" />
    ```

    两个按钮调用相同的命令，但它们各自传递不同的参数，这些参数被传递到`Execute`和`CanExecute`方法中。

使用 Maui 和 MVVM 最佳实践

在本章中，我们在`Core`项目中引入了`UseMaui`属性，这可能会与我们对 ViewModels 框架无关的先前声明相矛盾。虽然遵循 MVVM 模式，但建议保持 ViewModels 不受任何框架特定依赖的影响。然而，在这个特定的情况下，我们选择了更实用的方法来演示通过`Microsoft.Maui.Controls.Command`类实现`ICommand`的实例。在一个严格遵循 MVVM 的场景中，你希望在 ViewModels 中避免这样的依赖，以确保最大的灵活性和可维护性。

在*第五章*，*社区工具包*中，我们将探讨如何改进这段代码，使其更接近这些最佳实践。

作为额外的功能，我们可能还想显示一个图标来指示一个菜谱是否是收藏的。这个图标的可见性可以绑定到`IsFavorite`属性。当我们通过我们刚刚创建的按钮和命令切换这个属性时，图标的可见性也会更新：

1.  首先，让我们将`Chapter 03``/Assets/favorite.png`文件添加到`Resources/Images`文件夹中的`favorite.png`文件。你可能需要调整文件选择器弹出窗口中的文件类型过滤器，以包括`.png`文件。

1.  在`RecipeDetailPage.xaml`中，在显示菜谱标题的`Label`下方直接添加以下 XAML：

    ```cs
    <Image
        Margin="5" HeightRequest="35"
        IsVisible="{Binding IsFavorite}"
        Source="favorite.png" WidthRequest="35" />
    ```

    现在我们点击**添加为收藏**或**从收藏中移除**按钮时，你应该看到收藏图标出现或消失。

`ICommand`接口在实现 MVVM 中扮演着至关重要的角色。通过在命令中封装用户交互，它促进了视图和 ViewModel 之间关注点的清晰分离。这允许代码更加可维护、可测试和模块化。正如所展示的，`ICommand`及其`CanExecute`功能确保了根据应用程序的状态，用户可以获得适当的操作，从而进一步增强用户体验。通过利用`ICommand`，开发者可以有效地实现 MVVM 模式。

随着我们探索`ICommand`接口，你可能想知道，还有额外的工具和资源可用于在.NET MAUI 中实现 MVVM。多亏了出色的.NET 和.NET MAUI 社区，各种社区工具包为你的项目提供了更多对 MVVM 模式的支持。

摘要

在本章中，我们深入探讨了使 .NET MAUI 中的 MVVM 模式得以实现的核心理念。我们讨论了基本构建块，包括 `BindableObject`、`BindableProperty` 和 `BindingContext`，以及它们如何促进视图和视图模型之间的无缝通信。此外，我们还考察了 `INotifyPropertyChanged` 接口在通知 UI 视图模型属性变化中的重要性，并展示了 `ICommand` 接口如何以解耦的方式处理用户交互。通过理解这些基本概念，我们可以清楚地看到为什么 .NET MAUI 和 MVVM 模式如此和谐地匹配。

当我们继续进入第 *4 章* 数据绑定在 .NET MAUI 中，我们将更深入地研究数据绑定，包括值转换器、回退值、元素和相对绑定、多绑定以及编译绑定等主题。这将使你能够充分利用数据绑定的全部功能。让我们继续我们的旅程！

进一步阅读

要了解更多关于本章所涉及的主题，请查看以下资源：

+   可绑定属性：[`learn.microsoft.com/dotnet/maui/fundamentals/bindable-properties`](https://learn.microsoft.com/dotnet/maui/fundamentals/bindable-properties)

+   `BindableObject` 类：[`learn.microsoft.com/dotnet/api/microsoft.maui.controls.bindableobject`](https://learn.microsoft.com/dotnet/api/microsoft.maui.controls.bindableobject)

+   `INotifyPropertyChanged` 接口：[`learn.microsoft.com/dotnet/api/system.componentmodel.inotifypropertychanged`](https://learn.microsoft.com/dotnet/api/system.componentmodel.inotifypropertychanged)

```cs

```
