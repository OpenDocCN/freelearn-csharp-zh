# 16

# 使用.NET MAUI 构建移动和桌面应用程序

本章是关于通过构建适用于 iOS 和 Android、macOS Catalyst 和 Windows 的跨平台移动和桌面应用程序来学习如何制作图形用户界面（GUI）应用程序。根据 MAUI 团队的说法，.NET 7 和.NET 8 之间没有破坏性 API 更改。他们主要专注于修复错误和改进性能。

您将看到可扩展应用程序标记语言（**XAML**）如何使定义图形应用程序的用户界面（UI）变得容易。XAML 发音为“zamel”。

跨平台 GUI 开发不能仅在一百多页中学会，但我希望向您介绍一些可能的内容。想想这个.NET MAUI 章节和仅在线提供的附加部分，它们将为您提供一个入门介绍，以激发您的兴趣，然后您可以从专门针对移动或桌面开发的书籍中学习更多。

该应用程序将允许列出和管理 Northwind 数据库中的客户。您创建的移动应用程序将调用 ASP.NET Core Minimal APIs Web 服务。我们将从本章开始构建它，然后在仅在线提供的部分，即“实现.NET MAUI 的模型-视图-视图模型（MVVM）”，继续构建应用程序，您可以在本章末尾的练习中找到它。

拥有 Visual Studio 2022 版本 17.8 或更高版本的 Windows 计算机，或者任何具有 Visual Studio Code 和`dotnet` CLI 或 JetBrains Rider 的操作系统，都可以用来创建.NET MAUI 项目。但您需要一个装有 Windows 的计算机来编译 WinUI 3 应用程序，并且您需要一个装有 macOS 和 Xcode 的计算机来编译 macOS Catalyst 和 iOS。

在本章中，我们将涵盖以下主题：

+   理解 XAML

+   理解.NET MAUI

+   使用.NET MAUI 构建移动和桌面应用程序

+   使用共享资源

+   使用数据绑定

# 理解 XAML

让我们从.NET MAUI 使用的标记语言开始看起。

在 2006 年，微软发布了**Windows Presentation Foundation**（**WPF**），这是第一个使用 XAML 的技术。随后，用于网页和移动应用的 Silverlight 也迅速推出，但微软已不再支持它。WPF 至今仍被用于创建 Windows 桌面应用程序；例如，Visual Studio 2022 就是部分使用 WPF 构建的。

XAML 可以用于构建以下应用的某些部分：

+   **.NET MAUI 应用程序**适用于移动和桌面设备，包括 Android、iOS、Windows 和 macOS。它是名为**Xamarin.Forms**的技术的一种演变。

+   **WinUI 3 应用程序**适用于 Windows 10 和 11。

+   **通用 Windows 平台（UWP）应用**适用于 Windows 10 和 11、Xbox、混合现实和 Meta Quest VR 头戴式设备。

+   **WPF 应用程序**用于 Windows 桌面，包括 Windows 7 及更高版本。

+   使用跨平台第三方技术的**Avalonia**和**Uno Platform**应用。

## 使用 XAML 简化代码

XAML 简化了 C#代码，尤其是在构建用户界面（UI）时。

假设你需要两个或更多水平排列的粉色按钮来创建工具栏，当点击时执行它们的实现方法。

在 C# 中，你可能编写以下代码：

```cs
HorizontalStackPanel toolbar = new();
Button newButton = new();
newButton.Content = "New";
newButton.Background = new SolidColorBrush(Colors.Pink);
newButton.Clicked += NewButton_Clicked;
toolbar.Children.Add(newButton);
Button openButton = new();
openButton.Content = "Open";
openButton.Background = new SolidColorBrush(Colors.Pink);
openButton.Clicked += OpenButton_Clicked;
toolbar.Children.Add(openButton); 
```

在 XAML 中，这可以简化为以下几行代码。当处理此 XAML 时，等效的属性会被设置，并调用方法以实现与前面 C# 代码相同的目标：

```cs
<HorizontalStackPanel x:Name="toolbar">
  <Button x:Name="newButton" Background="Pink" 
          Clicked="NewButton_Clicked">New</Button>
  <Button x:Name="openButton" Background="Pink" 
          Clicked="OpenButton_Clicked">Open</Button>
</StackPanel> 
```

你可以将 XAML 视为一个替代且更简单的声明和实例化 .NET 类型的途径，尤其是在定义 UI 及其使用的资源时。

XAML 允许在不同级别声明资源，如 UI 元素或页面，或全局应用于应用程序以实现资源共享。

XAML 允许在 UI 元素之间或 UI 元素与对象和集合之间进行数据绑定。

如果你选择在编译时使用 XAML 定义你的 UI 和相关资源，那么代码后文件必须在页面构造函数中调用 `InitializeComponent` 方法，如下面高亮显示的代码所示：

```cs
public partial class MainPage : ContentPage
{
**public****MainPage****()**
 **{**
 **InitializeComponent();** **// Process the XAML markup.**
 **}**
  private void NewButton_Clicked(object sender, EventArgs e)
  {
    ...
  }
  private void OpenButton_Clicked(object sender, EventArgs e)
  {
    ...
  }
} 
```

调用 `InitializeComponent` 方法告诉页面读取其 XAML，创建其中定义的控件，并设置它们的属性和事件处理器。

## .NET MAUI 命名空间

.NET MAUI 有几个重要的命名空间，其中定义了其类型，如 *表 16.1* 所示：

| **命名空间** | **描述** |
| --- | --- |
| `Microsoft.Maui` | 如 `FlowDirection`、`IButton`、`IImage` 和 `Thickness` 之类的实用类型。 |
| `Microsoft.Maui.Controls` | 常用控件、页面和相关类型，如 `Application`、`Brush`、`Button`、`CheckBox`、`ContentPage`、`Image` 和 `VerticalStackPanel`。 |
| `Microsoft.Maui.Graphics` | 用于图形的类型，如 `Color`、`Font`、`ImageFormat`、`PathBuilder`、`Point` 和 `Size`。 |

表 16.1：重要的 MAUI 命名空间

要使用 XAML 导入命名空间，在根元素中添加 `xmlns` 属性。一个命名空间被导入为默认值，其他命名空间必须使用前缀命名。

例如，.NET MAUI 类型默认导入，因此元素名称不需要前缀；通用 XAML 语法使用 `x` 前缀导入，用于执行命名控件或 XAML 将编译成的类名等常见操作。你的项目类型通常使用 `local` 前缀导入，如下面的标记所示：

```cs
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:MyMauiApp.Controls"
             x:Class="MyMauiApp.MainPage"
             ...>
  <Button x:Name="NewFileButton" ...>New File</Button>
  <local:CustomerList x:Name="CustomerList" ... />
  ...
</ContentPage> 
```

在上面的示例中，项目名为 `MyMauiApp`，其控件如 `CustomerList` 控件定义在名为 `MyMauiApp.Controls` 的命名空间中。此命名空间已使用前缀 `local` 进行注册，因此当需要 `CustomerList` 控件的实例时，它使用 `<local:CustomerList>` 进行声明。

你可以根据需要导入带有不同前缀的命名空间。

## 类型转换器

类型转换器将必须设置为 `string` 类型的 XAML 属性值转换为其他类型。例如，以下按钮的 `Background` 属性设置为 `string` 值 `"Pink"`：

```cs
<Button x:Name="newButton" Background="Pink" ... 
```

这通过类型转换器转换为 `SolidColorBrush` 实例，如下面的等效代码所示：

```cs
newButton.Background = new SolidColorBrush(Colors.Pink); 
```

.NET MAUI 提供了许多类型转换器，你可以创建并注册自己的。这些对于自定义数据可视化特别有用。

## 在 .NET MAUI 控件之间进行选择

有许多预定义的控件可供选择，用于常见的 UI 场景。.NET MAUI（以及大多数 XAML 方言）支持这些控件，如 *表 16.2* 所示：

| **Controls** | **描述** |
| --- | --- |
| `Button`, `ImageButton`, `MenuItem`, `ToolbarItem` | 执行操作 |
| `CheckBox`, `RadioButton`, `Switch` | 选择选项 |
| `DatePicker`, `TimePicker` | 选择日期和时间 |
| `CollectionView`, `ListView`, `Picker`, `TableView` | 从列表和表中选择项目 |
| `CarouselView`, `IndicatorView` | 每次显示一个项目的滚动动画视图 |
| `AbsoluteLayout`, `BindableLayout`, `FlexLayout`, `Grid`, `HorizontalStackLayout`, `StackLayout`, `VerticalStackLayout` | 以不同方式影响其子项的布局容器 |
| `Border`, `BoxView`, `Frame`, `ScrollView` | 视觉元素 |
| `Ellipse`, `Line`, `Path`, `Polygon`, `Polyline`, `Rectangle`, `RoundRectangle` | 图形元素 |
| `ActivityIndicator`, `Label`, `ProgressBar`, `RefreshView` | 显示只读文本和其他只读显示 |
| `Editor`, `Entry` | 编辑文本 |
| `GraphicsView`, `Image` | 嵌入图像、视频和音频文件 |
| `Slider`, `Stepper` | 在数字范围内选择 |
| `SearchBar` | 添加搜索功能 |
| `BlazorWebView`, `WebView` | 嵌入 Blazor 和 Web 组件 |
| `ContentView` | 构建自定义控件 |

表 16.2：MAUI 用户界面控件

.NET MAUI 在 `Microsoft.Maui.Controls` 命名空间中定义其控件。它还有一些专门的控件：

+   `Application`: 表示一个跨平台的图形应用程序。它设置根页面，管理窗口、主题和资源，并提供应用级别的事件，如 `PageAppearing`、`ModalPushing` 和 `RequestedThemeChanged`。它还具有你可以重写的方法来挂钩应用事件，如 `OnStart`、`OnSleep`、`OnResume` 和 `CleanUp`。

+   `Shell`: 一个提供大多数应用程序所需的 UI 功能的 `Page` 控件，如飞出或标签栏导航、导航跟踪和管理以及导航事件。

大多数 .NET MAUI 控件都从 `View` 派生。`View` 派生类型的一个重要特性是它们可以进行嵌套。这允许你构建复杂的自定义用户界面。

## 标记扩展

为了支持一些高级功能，XAML 使用标记扩展。其中一些最重要的使元素和数据绑定以及资源重用成为可能，如下列所示：

+   `{Binding}` 将一个元素链接到另一个元素或数据源中的值。

+   `{OnPlatform}` 根据当前平台设置不同的属性值。

+   `{StaticResource}` 和 `{DynamicResource}` 将元素链接到共享资源。

+   `{AppThemeBinding}` 将元素链接到在主题中定义的共享资源。

.NET MAUI 提供了 `OnPlatform` 标记扩展，允许您根据平台设置不同的标记。例如，iPhone X 及以后的型号在手机显示屏顶部引入了缺口，占据了额外的空间。我们可以为适用于所有设备的应用程序添加额外的填充，但如果我们能只为 iOS 添加额外的填充，那就更好了，如下面的标记所示：

```cs
<VerticalStackLayout>
  <VerticalStackLayout.Padding>
    <OnPlatform x:TypeArguments="Thickness">
      <On Platform="iOS" Value="30,60,30,30" />
      <On Platform="Android" Value="30" />
      <On Platform="WinUI" Value="30" />
    </OnPlatform>
  </VerticalStackLayout.Padding> 
```

也有简化的语法，如下面的标记所示：

```cs
<VerticalStackLayout Padding"{OnPlatform iOS='30,60,30,30', Default='30'}"> 
```

# 理解 .NET MAUI

要创建一个仅需要在 iPhone 上运行的应用程序，您可能会选择使用 Objective-C 或 Swift 语言以及 UIKit 库，并通过 Xcode 开发工具来构建它。

要创建一个仅需要在 Android 手机上运行的应用程序，您可能会选择使用 Java 或 Kotlin 语言以及 Android SDK 库，并通过 Android Studio 开发工具来构建它。

但如果您需要创建一个可以在 iPhone 和 Android 手机上运行的应用程序呢？如果您只想使用您已经熟悉的编程语言和开发平台创建一次该移动应用程序怎么办？还有，如果您意识到通过稍微增加一些代码来适应桌面尺寸的 UI，您还可以针对 macOS 和 Windows 桌面呢？

.NET MAUI 使开发者能够使用 Catalyst 为 Apple iOS（iPhone）、iPadOS、macOS 构建跨平台移动应用程序，使用 WinUI 3 为 Windows 构建应用程序，以及使用 C# 和 .NET 为 Google Android 构建应用程序，这些应用程序随后被编译成原生 API 并在原生手机和桌面平台上执行。

业务逻辑层代码可以一次编写并在所有平台上共享。UI 交互和 API 在各种移动和桌面平台上有所不同，因此 UI 层有时需要针对每个平台进行自定义。

与 WPF 和 UWP 应用程序一样，.NET MAUI 使用 XAML 为所有平台定义一次 UI，使用特定于平台的 UI 组件的抽象。使用 .NET MAUI 构建的应用程序使用原生平台小部件绘制 UI，因此应用程序的外观和感觉与目标移动平台自然契合。

使用 .NET MAUI 构建的用户体验可能不会像使用针对该平台原生工具自定义构建的应用程序那样完美地适应特定平台，但对于不会拥有数百万用户的移动和桌面应用程序来说，这已经足够好了。通过一些努力，您可以构建出美丽的应用程序，正如微软挑战所展示的那样，您可以在以下链接中了解更多信息：

[`devblogs.microsoft.com/dotnet/announcing-dotnet-maui-beautiful-ui-challenge/`](https://devblogs.microsoft.com/dotnet/announcing-dotnet-maui-beautiful-ui-challenge/)

## .NET MAUI 和 Xamarin 支持

.NET MAUI 的主要版本从.NET 7 开始与.NET 一起发货，但作为可选工作负载。这意味着.NET MAUI 不遵循与主要.NET 平台相同的**短期支持**（**STS**）/**长期支持**（**LTS**）。每个.NET MAUI 版本只有 18 个月的支持，因此.NET MAUI 实际上始终是一个 STS 版本，这包括随.NET 8 一起作为工作负载发货的.NET MAUI 版本。

.NET MAUI 依赖于其他操作系统，如 iOS 和 macOS，因此会变得复杂。iOS 的主要版本通常在 9 月发布，而 iPadOS 和 macOS 的主要版本通常在 10 月或 11 月稍晚发布。这并不给.NET MAUI 团队太多时间在 11 月初.NET 发布前确保其平台与这些操作系统良好兼容。

**警告！**Xamarin 将于 2024 年 5 月 1 日达到其**生命终结**（**EOL**），因此任何 Xamarin 和 Xamarin.Forms 项目都应该在此之前迁移到.NET MAUI 或 Avalonia 或 Uno 等替代方案。

## 移动优先、云优先的开发工具

移动应用程序通常由云中的服务支持。

微软首席执行官萨蒂亚·纳德拉（Satya Nadella）曾著名地说以下内容：

> 对我来说，当我们说“移动优先”时，并不是指设备的移动性，而是指个人体验的移动性。[...] 你唯一能够协调这些应用程序和数据移动性的方式是通过云。

在安装 Visual Studio 2022 时，你必须选择位于**桌面和移动**部分的**.NET 多平台应用程序 UI 开发**工作负载，如图*图 16.1*所示：

![](img/B19587_16_01.png)

图 16.1：为 Visual Studio 2022 选择.NET MAUI 工作负载

## 手动安装.NET MAUI 工作负载

安装 Visual Studio 2022 应该会安装你选择的所需.NET MAUI 工作负载。如果没有，那么你可以手动确保工作负载已安装。

要查看当前安装的工作负载，请输入以下命令：

```cs
dotnet workload list 
```

当前安装的工作负载将显示在表格中，如下所示输出：

```cs
Installed Workload Ids      Manifest Version      Installation Source
-----------------------------------------------------------------------
maui-maccatalyst            6.0.486/6.0.400       SDK 7.0.100 
```

要查看可安装的工作负载，请输入以下命令：

```cs
dotnet workload search 
```

当前可用的工作负载将显示在表格中，如下所示输出：

```cs
Workload ID                 Description
------------------------------------------------------------------------------------
android                     .NET SDK Workload for building Android applications.
ios                         .NET SDK Workload for building iOS applications.
maccatalyst                 .NET SDK Workload for building MacCatalyst applications.
macos                       .NET SDK Workload for building macOS applications.
maui                        .NET MAUI SDK for all platforms
maui-android                .NET MAUI SDK for Android
maui-desktop                .NET MAUI SDK for Desktop
maui-ios                    .NET MAUI SDK for iOS
maui-maccatalyst            .NET MAUI SDK for Mac Catalyst
maui-mobile                 .NET MAUI SDK for Mobile
maui-tizen                  .NET MAUI SDK for Tizen
maui-windows                .NET MAUI SDK for Windows
tvos                        .NET SDK Workload for building tvOS applications.
wasi-experimental           workloads/wasi-experimental/description
wasm-experimental           workloads/wasm-experimental/description
wasm-experimental-net7      .NET WebAssembly experimental tooling for net7.0
wasm-tools                  .NET WebAssembly build tools
wasm-tools-net6             .NET WebAssembly build tools for net6.0
wasm-tools-net7             .NET WebAssembly build tools for net7.0 
```

要安装所有平台的.NET MAUI 工作负载，请在命令行或终端中输入以下命令：

```cs
dotnet workload install maui 
```

要更新所有现有工作负载安装，请输入以下命令：

```cs
dotnet workload update 
```

要添加项目中缺少的工作负载安装，请在包含项目文件的文件夹中输入以下命令：

```cs
dotnet workload restore <projectname> 
```

随着时间的推移，你可能会安装与不同版本的.NET SDK 相关的多个工作负载版本。在.NET 8 之前，开发者尝试手动删除工作负载文件夹，这可能会引起问题。随着.NET 8 的引入，有一个新功能可以删除遗留和不需要的工作负载，如下所示命令：

```cs
dotnet workload clean 
```

**更多信息**：如果您想使用 Visual Studio 2022 创建 iOS 移动应用程序或 macOS Catalyst 桌面应用程序，那么您可以通过网络连接到**Mac 构建主机**。有关说明，请参阅以下链接：[`learn.microsoft.com/en-us/dotnet/maui/ios/pair-to-mac`](https://learn.microsoft.com/en-us/dotnet/maui/ios/pair-to-mac)。

## .NET MAUI 用户界面组件类别

.NET MAUI 包括一些用于构建用户界面的常见控件。它们可以分为四个类别：

+   **页面**代表跨平台应用程序屏幕，例如`Shell`、`ContentPage`、`NavigationPage`、`FlyoutPage`和`TabbedPage`。

+   **布局**代表其他 UI 组件组合的结构，例如`Grid`、`StackLayout`和`FlexLayout`。

+   **视图**代表单个用户界面组件，例如`CarouselView`、`CollectionView`、`Label`、`Entry`、`Editor`和`Button`。

+   **单元格**代表列表或表格视图中的单个项目，例如`TextCell`、`ImageCell`、`SwitchCell`和`EntryCell`。

### Shell 控件

`Shell`控件旨在通过提供标准化的导航和搜索功能来简化应用程序开发。在您的项目中，您将创建一个从`Shell`控件类继承的类。您的派生类定义了如`TabBar`之类的组件，它包含`Tab`项、`FlyoutItem`实例和`ShellContent`，它们包含每个页面的`ContentPage`实例。当只有四到五页需要导航时，应使用`TabBar`。当有更多项时，应使用`FlyoutItem`导航，因为它们可以表示为垂直可滚动的列表。您可以使用两者，其中`TabBar`显示项目的一个子集。`Shell`将它们保持同步。

Flyout 导航是指从移动设备屏幕的左侧或桌面应用程序主窗口的左侧飞出（或滑动）项目列表。用户通过点击一个带有三个水平线堆叠的“汉堡”图标来调用它。当用户点击飞出项时，其页面在需要时实例化，因为用户在 UI 中导航。

顶部栏在需要时自动显示**返回**按钮，允许用户导航回上一页。

### 列表视图控件

`ListView`控件用于长列表的数据绑定值，这些值类型相同。它可以有标题和页脚，其列表项可以分组。

它包含用于容纳每个列表项的单元格。有两种内置的单元格类型：文本和图像。开发者可以定义自定义单元格类型。

单元格可以具有上下文操作，当在 iPhone 上滑动单元格、在 Android 上长按或桌面操作系统上右键单击时会出现。破坏性的上下文操作可以以红色显示，如下面的标记所示：

```cs
<TextCell Text="{Binding CompanyName}" Detail="{Binding Location}">
  <TextCell.ContextActions>
    <MenuItem Clicked="Customer_Phoned" Text="Phone" />
    <MenuItem Clicked="Customer_Deleted" Text="Delete" IsDestructive="True" />
  </TextCell.ContextActions>
</TextCell> 
```

### Entry 和 Editor 控件

`Entry`和`Editor`控件用于编辑文本值，通常与实体模型属性数据绑定，如下面的标记所示：

```cs
<Editor Text="{Binding CompanyName, Mode=TwoWay}" /> 
```

**良好实践**：对于单行文本使用`Entry`。对于多行文本使用`Editor`。

## .NET MAUI 处理器

在.NET MAUI 中，XAML 控件在`Microsoft.Maui.Controls`命名空间中定义。称为**处理器**的组件将这些常用控件映射到每个平台的原生控件。在 iOS 上，处理器将.NET MAUI 的`Button`映射到由 UIkit 定义的 iOS 原生`UIButton`。在 macOS 上，`Button`映射到由 AppKit 定义的`NSButton`。在 Android 上，`Button`映射到 Android 原生`AppCompatButton`。

处理器有一个`NativeView`属性，它公开了底层的原生控件。这允许你使用平台特定的功能，如属性、方法和事件，并自定义原生控件的所有实例。

## 编写特定平台的代码

如果你需要编写仅针对特定平台（如 Android）执行的代码语句，则可以使用编译器指令。

例如，默认情况下，Android 上的`Entry`控件显示下划线字符。如果你想隐藏下划线，你可以编写一些 Android 特定的代码来获取`Entry`控件的处理器，使用其`NativeView`属性来访问底层的原生控件，然后将控制该功能的属性设置为`false`，如下面的代码所示：

```cs
#if __ANDROID__
  Handlers.EntryHandler.EntryMapper[nameof(IEntry.BackgroundColor)] = (h, v) =>
  {
    (h.NativeView as global::Android.Views.Entry).UnderlineVisible = false;
  };
#endif 
```

预定义的编译器常量包括以下内容：

+   `__ANDROID__`

+   `__IOS__`

+   `WINDOWS`

编译器的`#if`语句语法与 C#的`if`语句语法略有不同，如下面的代码所示：

```cs
#if __IOS__
  // iOS-specific statements
#elif __ANDROID__
  // Android-specific statements
#elif WINDOWS
  // Windows-specific statements
#endif 
```

现在你已经了解了 MAUI 应用程序的一些重要概念，并且已经设置了 MAUI 所需的附加组件，让我们来实际操作并构建一个 MAUI 项目。

# 使用.NET MAUI 构建移动和桌面应用程序

我们将为 Northwind 的客户管理构建一个移动和桌面应用程序。

**良好实践**：如果你有一台 Mac，并且你从未在上面运行过 Xcode，那么现在就运行它，直到你看到*开始*窗口。这将确保所有必需的组件都已安装并注册。如果你不这样做，那么你的项目可能会出现错误。

## 创建用于本地应用程序测试的虚拟 Android 设备

要针对 Android，你必须安装至少一个 Android SDK。Visual Studio 2022 的默认安装（已包含移动开发工作负载）已经包含了一个 Android SDK，但它通常是针对尽可能多的 Android 设备设计的较旧版本。

要使用.NET MAUI 的最新功能，你必须配置一个更近期的 Android 虚拟设备：

1.  在 Windows 中，启动**Visual Studio 2022**。如果你看到模态对话框**欢迎体验**，则单击**无代码继续**。

1.  导航到**工具** | **Android** | **Android 设备管理器**。如果你被**用户账户控制**提示允许此应用程序对你的设备进行更改，请单击**是**。

1.  在**Android 设备管理器**中，单击**+ 新建**按钮以创建一个新设备。

1.  在对话框中，按照*图 16.2*所示进行以下选择：

    +   **基础设备**: **Pixel 5**

    +   **处理器**: **x86_64**

    +   **操作系统**: **Android 13.0 – API 33**

    +   **Google APIs**: 已选择

    +   **Google Play Store**: 已清除

![](img/B19587_16_02.png)

图 16.2：选择虚拟 Android 设备的硬件和操作系统

1.  点击 **创建**。

1.  接受任何许可协议。

1.  等待任何必要的下载。

1.  在 **Android 设备管理器** 中，在设备列表中，在您刚刚创建的设备的行中，点击 **启动**。

1.  请耐心等待！模拟器启动可能需要几分钟。

1.  当 Android 设备启动完成后，点击 Chrome 浏览器，通过导航到 [`www.bbc.co.uk/news`](https://www.bbc.co.uk/news) 测试它是否有网络访问权限。

1.  关闭模拟器。

1.  关闭 **Android 设备管理器**。

1.  重新启动 Visual Studio 2022 以确保它知道新的模拟器。

## 启用 Windows 开发者模式

要为 Windows 创建应用程序，您必须启用开发者模式：

1.  导航到 **开始** **|** **设置** **|** **隐私和安全** **|** **开发者**，然后开启 **开发者模式**。（您也可以搜索“开发者”。）

1.  接受有关它“**可能使您的设备和个人信息面临安全风险或损害您的设备**”的警告。

1.  关闭 **设置** 应用。

## 创建 .NET MAUI 项目

现在我们将创建一个跨平台移动和桌面应用程序的项目：

1.  在 Visual Studio 2022 中，添加一个新项目，如下列表中定义：

    +   项目模板：**.NET MAUI App** / `maui`

    您可以选择 **C#** 作为语言，并选择 **MAUI** 作为项目类型以过滤并仅显示适当的模板。

    +   项目文件和文件夹：`Northwind.Maui.Client`

    +   解决方案文件和文件夹：`Chapter16`

1.  在 Windows 上，如果您看到 Windows 安全警报，表明 **Windows Defender 防火墙已阻止所有公共和私人网络上的 Broker 的某些功能**，那么选择 **私人网络** 并清除 **公共网络**，然后点击 **允许访问** 按钮。

1.  在项目文件中，注意针对 iOS、Android 和 Mac Catalyst 的元素，以及如果操作系统是 Windows，则启用 Windows 目标的元素，以及将项目设置为单个 MAUI 项目的元素，如下部分标记所示：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
     **<TargetFrameworks>net8****.0****-ios;net8****.0****-android;net8****.0****-maccatalyst</TargetFrameworks>**
     **<TargetFrameworks Condition=****"$([MSBuild]::IsOSPlatform('windows'))"****>$(TargetFrameworks);net8****.0****-windows10****.0.19041.0****</TargetFrameworks>**
        ...
        <OutputType>Exe</OutputType>
        <RootNamespace>Northwind.Maui.Client</RootNamespace>
     **<UseMaui>****true****</UseMaui>**
     **<SingleProject>****true****</SingleProject>** 
    ```

    如果您看到错误 `Error NU1012 Platform version is not present for one or more target frameworks, even though they have specified a platform: net8.0-ios, net8.0-maccatalyst`，那么在命令提示符或终端中，在项目文件夹中，如以下命令所示恢复项目工作负载：

    ```cs
    dotnet workload restore 
    ```

1.  在工具栏中 **运行** 按钮的右侧，将 **框架** 设置为 **net8.0-android**，并选择您之前创建的 **Pixel 5 - API 33 (Android 13.0 - API 33)** 模拟器映像，如图 *16.3* 所示：

![](img/B19587_16_03.png)

图 16.3：选择 Android 设备作为启动目标

1.  在工具栏中点击 **运行** 按钮并等待设备模拟器启动 Android 操作系统，然后部署并启动您的移动应用程序。这可能需要超过五分钟，尤其是第一次构建新的 MAUI 项目时。请关注 Visual Studio 2022 的状态栏，如下所示：![](img/B19587_16_04.png)

    图 16.4：状态栏显示 .NET MAUI 应用程序部署的进度

    如果您是第一次这样做，可能会有另一个 Google 许可协议需要确认。

1.  在 .NET MAUI 应用程序中，点击 **点击我** 按钮三次以增加计数器，如下所示：*图 16.5*：

![](img/B19587_16_05.png)

图 16.5：在 Android 的 .NET MAUI 应用程序中增加计数器三次

1.  关闭 Android 设备模拟器。您不需要关闭模拟器电源。

1.  在工具栏中 **运行** 按钮的右侧，将 **框架** 设置为 **net8.0-windows10.0.19041.0**。

1.  确保已选择 **调试** 配置，然后点击标有 **Windows Machine** 的实心绿色三角形 **启动** 按钮。您可能会看到一个关于缺少应安装的包的警告。只需再次点击 **启动** 按钮即可，它们现在应该已安装并正常工作。

1.  几分钟后，注意 Windows 应用程序显示与 *图 16.6* 中相同的 **点击我** 按钮和计数器功能：

![](img/B19587_16_06.png)

图 16.6：Windows 上的相同 .NET MAUI 应用程序

1.  关闭 Windows 应用程序。

**良好实践**：您应该在所有可能运行的设备上测试您的 .NET MAUI 应用程序。在本章中，即使我没有明确要求您这样做，我也建议您在添加新功能后，通过在模拟的 Android 设备和 Windows 上运行应用程序来尝试应用程序。这样，您至少可以看到它在主要为高而窄的纵向尺寸的移动设备上的外观，以及在更大横向尺寸的桌面设备上的外观。如果您使用的是 Mac，那么我建议您在 iOS 模拟器、Android 模拟器和作为 Mac Catalyst 桌面应用程序中测试它。

## 添加外壳导航和更多内容页面

现在，让我们回顾现有的 .NET MAUI 应用程序结构，然后向项目中添加一些新页面和导航：

1.  在 `Northwind.Maui.Client` 项目中，在 `MauiProgram.cs` 中，注意 `builder` 对象调用 `UseMauiApp` 并指定 `App` 作为其泛型类型，如下面的代码所示，高亮显示：

    ```cs
    using Microsoft.Extensions.Logging;
    namespace Northwind.Maui.Client;
    public static class MauiProgram
    {
      public static MauiApp CreateMauiApp()
      {
        var builder = MauiApp.CreateBuilder();
        builder
     **.UseMauiApp<App>()**
          .ConfigureFonts(fonts =>
          {
            fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
            fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");
          });
    #if DEBUG
        builder.Logging.AddDebug();
    #endif
        return builder.Build();
      }
    } 
    ```

1.  在 **解决方案资源管理器** 中，展开 `App.xaml`，打开 `App.xaml.cs`，并注意 `App` 的 `MainPage` 属性被设置为 `AppShell` 的一个实例，如下面的代码所示，高亮显示：

    ```cs
    namespace Northwind.Maui.Client;
    public partial class App : Application
    {
      public App()
      {
        InitializeComponent();
     **MainPage =** **new** **AppShell();**
      }
    } 
    ```

1.  在 `AppShell.xaml` 中，注意外壳禁用了飞出模式，并且只有一个名为 `MainPage` 的内容页面，如下面的代码所示，高亮显示：

    ```cs
    <?xml version="1.0" encoding="UTF-8" ?>
    <Shell
      x:Class="Northwind.Maui.Client.AppShell"
      xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
      xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
      xmlns:local="clr-namespace:Northwind.Maui.Client"
      **Shell.FlyoutBehavior****=****"Disabled"**
      Title="Northwind.Maui.Client">
      <ShellContent
        Title="Home"
        **ContentTemplate****=****"{DataTemplate local:MainPage}"**
      Route="MainPage" />
    </Shell> 
    ```

    只包含一个内容页面的外壳不会显示任何导航。您至少需要两个外壳内容项。

1.  在`Resources`文件夹中的`Images`文件夹中，添加我们将用于即将添加的导航中飞出项的一些图标图片。

    你可以从以下链接下载图片：[`github.com/markjprice/apps-services-net8/tree/main/code/Chapter16/Northwind.Maui.Client/Resources/Images`](https://github.com/markjprice/apps-services-net8/tree/main/code/Chapter16/Northwind.Maui.Client/Resources/Images)。

1.  在`AppShell.xaml`中，启用飞出模式，将背景设置为浅蓝色，添加`MainPage`内容的图标，添加飞出标题，然后添加一些包含更多外壳内容的飞出项，如下面的标记所示：

    ```cs
    <?xml version="1.0" encoding="UTF-8" ?>
    <Shell
      x:Class="Northwind.Maui.Client.AppShell"
      xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
      xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
      xmlns:local="clr-namespace:Northwind.Maui.Client"
      Shell.FlyoutBehavior="**Flyout"**
      Title="Northwind.Maui.Client"
      **FlyoutBackgroundColor****=****"AliceBlue"****>**
    ** <Shell.FlyoutHeader****>**
    **<****HorizontalStackLayout****Spacing****=****"10"****HorizontalOptions****=****"Start"****>**
    **<****Image****Source****=****"wind_face_3d.png"**
    **WidthRequest****=****"80"****HeightRequest****=****"80"** **/>**
    **<****Label****Text****=****"Northwind"****FontFamily****=****"OpenSansSemibold"**
    **FontSize****=****"32"****VerticalOptions****=****"Center"** **/>**
    **</****HorizontalStackLayout****>**
    **</****Shell.FlyoutHeader****>**
      <ShellContent Title="Home"
     **Icon****=****"file_cabinet_3d.png"**
        ContentTemplate="{DataTemplate local:MainPage}"
        Route="MainPage" />
     **<ShellContent****Title****=****"Categories"**
    **Icon****=****"delivery_truck_3d.png"**
    **ContentTemplate****=****"{DataTemplate local:CategoriesPage}"**
    **Route****=****"Categories"** **/>**
    **<****ShellContent****Title****=****"Products"**
    **Icon****=****"cityscape_3d.png"**
    **ContentTemplate****=****"{DataTemplate local:ProductsPage}"**
    **Route****=****"Products"** **/>**
    **<****ShellContent****Title****=****"Customers"**
    **Icon****=****"card_index_3d.png"**
    **ContentTemplate****=****"{DataTemplate local:CustomersPage}"**
    **Route****=****"Customers"** **/>**
    **<****ShellContent****Title****=****"Employees"**
    **Icon****=****"identification_card_3d.png"**
    **ContentTemplate****=****"{DataTemplate local:EmployeesPage}"**
    **Route****=****"Employees"** **/>**
    **<****ShellContent****Title****=****"Settings"**
    **Icon****=****"gear_3d.png"**
    **ContentTemplate****=****"{DataTemplate local:SettingsPage}"**
    **Route****=****"Settings"** **/>**

    </Shell> 
    ```

    你会在一些`ContentTemplate`行上看到关于缺失页面的警告，因为我们还没有创建它们。在浅色模式下`AliceBlue`看起来不错，但如果你的操作系统使用深色模式，你可能更喜欢像`#75858a`这样的替代颜色。

1.  在 Visual Studio 2022 中，右键单击`Northwind.Maui.Client`项目文件夹，选择**添加** | **新建项...**或按*Ctrl* + *Shift* + *A*，在模板类型树中选择**.NET MAUI**，选择**.NET MAUI ContentPage (XAML)**，输入名称`SettingsPage`，然后点击**添加**。

    Visual Studio Code 和 JetBrains Rider 没有 MAUI 的项目项模板。你可以使用 CLI 创建此项目，如下面的命令所示：

    ```cs
    dotnet new maui-page-xaml --name SettingsPage.xaml 
    ```

1.  重复之前的步骤以添加以下命名的内容页面：

    +   `CategoriesPage`

    +   `CustomersPage`

    +   `CustomerDetailPage`

    +   `EmployeesPage`

    +   `ProductsPage`

1.  在**解决方案资源管理器**中，双击`CategoriesPage.xaml`文件以打开它进行编辑。请注意，Visual Studio 2022 还没有 XAML 的图形设计视图。

1.  在`<ContentPage>`元素中，将`Title`更改为`Categories`，在`<Label>`元素中，将`Text`更改为`Categories`，如下面的标记所示：

    ```cs
    <?xml version="1.0" encoding="utf-8" ?>
    <ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                 xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                 x:Class="Northwind.Maui.Client.CategoriesPage"
                 Title="**Categories**">
        <VerticalStackLayout>
            <Label 
                Text="**Categories**"
                VerticalOptions="Center" 
                HorizontalOptions="Center" />
        </VerticalStackLayout>
    </ContentPage> 
    ```

1.  导航到**视图** | **工具箱**或按*Ctrl* + *W*，*X*。请注意，工具箱有**控件**、**布局**、**单元格**和**通用**等部分。如果你使用的是没有工具箱的代码编辑器，你可以直接输入标记而不是使用工具箱。

1.  在工具箱的顶部是一个搜索框。输入字母`b`，然后注意控件列表被过滤以显示**Button**、**ProgressBar**和**AbsoluteLayout**等控件。

1.  将工具箱中的**Button**控件拖放到现有的`<Label>`控件之后，在`VerticalStackLayout`的关闭元素之前，并将它的`Text`属性更改为`Hello!`，如下面的标记所示：

    ```cs
    <Button Text="Hello!" /> 
    ```

1.  将启动设置为**Windows Machine**，然后以调试模式启动`Northwind.Maui.Client`项目。请注意，Visual Studio 的状态栏显示**XAML 热重载**已连接。

1.  在应用左上角，点击飞出菜单（“汉堡”图标），并注意飞出项中使用的标题和图标图片，如图 16.7 所示：

![图片](img/B19587_16_07.png)

图 16.7：带有图像图标的飞出菜单

1.  在飞出菜单中，点击**类别**，注意按钮上的文本为**Hello**！并且它横跨应用程序窗口的宽度。

1.  保持应用程序运行，然后在 Visual Studio 2022 中，将 `Text` 属性更改为 `Click Me`，添加一个属性来设置 `WidthRequest` 属性为 `100`，并注意**XAML 热重载**功能会自动在应用程序本身中反映更改，如图 *图 16.8* 所示：

![图片](img/B19587_16_08.png)

图 16.8：XAML 热重载自动更新实时应用程序中的 XAML 更改

1.  关闭应用程序。

1.  将 `Button` 元素修改为名为 `ClickMeButton`，并为它的 `Clicked` 事件添加一个新的事件处理器，如图 *图 16.9* 所示：

![图片](img/B19587_16_09.png)

图 16.9：向控件添加事件处理器

1.  右键单击事件处理器名称，选择**转到定义**或按 *F12*。

1.  在事件处理器方法中添加一个语句，将按钮的内容设置为当前时间，如图中高亮显示的以下代码所示：

    ```cs
    private void ClickMeButton_Click(object sender, EventArgs e)
    {
      **ClickMeButton.Text = DateTime.Now.ToString(****"hh:mm:ss"****);**
    } 
    ```

1.  在至少一个移动设备和至少一个桌面设备上启动 `Northwind.Maui.Client` 项目。

    **良好实践**：当部署到 Android 模拟器或 iOS 模拟器时，旧版本的应用程序可能仍在运行。在与新版本的应用程序交互之前，请确保等待新版本的应用程序部署完成。您可以通过查看 Visual Studio 状态栏来跟踪部署进度，或者只需等待您看到消息**XAML 热重载已连接**。

1.  导航到**类别**，点击按钮，注意其文本标签变为当前时间。

1.  关闭应用程序。

## 实现更多内容页面

现在，让我们实现一些新页面：

1.  在 `EmployeesPage.xaml` 中，将 `Title` 修改为 `Employees`，并添加标记来定义简单计算器的 UI，如图中高亮显示的以下标记所示：

    ```cs
    <?xml version="1.0" encoding="utf-8" ?>
    <ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                 xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                 x:Class="Northwind.Maui.Client.EmployeesPage"
                 Title="**Employees**">

      <VerticalStackLayout>
    **<****Grid****Background****=****"DarkGray"****Margin****=****"10"**
    **Padding****=****"5"****x:Name****=****"GridCalculator"**
    **ColumnDefinitions****=****"****Auto,Auto,Auto,Auto"**
    **RowDefinitions****=****"Auto,Auto,Auto,Auto"****>**
    **<****Button****Grid.Row****=****"0"****Grid.Column****=****"0"****Text****=****"****X"** **/>**
    **<****Button****Grid.Row****=****"0"****Grid.Column****=****"1"****Text****=****"/"** **/>**
    **<****Button****Grid.Row****=****"0"****Grid.Column****=****"2"****Text****=****"+"** **/>**
    **<****Button****Grid.Row****=****"****0"****Grid.Column****=****"3"****Text****=****"-"** **/>**
    **<****Button****Grid.Row****=****"1"****Grid.Column****=****"****0"****Text****=****"7"** **/>**
    **<****Button****Grid.Row****=****"1"****Grid.Column****=****"1"****Text****=****"****8"** **/>**
    **<****Button****Grid.Row****=****"1"****Grid.Column****=****"2"****Text****=****"9"** **/>**
    **<****Button****Grid.Row****=****"1"****Grid.Column****=****"3"****Text****=****"0"** **/>**
    **<****Button****Grid.Row****=****"****2"****Grid.Column****=****"0"****Text****=****"4"** **/>**
    **<****Button****Grid.Row****=****"2"****Grid.Column****=****"****1"****Text****=****"5"** **/>**
    **<****Button****Grid.Row****=****"2"****Grid.Column****=****"2"****Text****=****"****6"** **/>**
    **<****Button****Grid.Row****=****"2"****Grid.Column****=****"3"****Text****=****"."** **/>**
    **<****Button****Grid.Row****=****"3"****Grid.Column****=****"0"****Text****=****"1"** **/>**
    **<****Button****Grid.Row****=****"****3"****Grid.Column****=****"1"****Text****=****"2"** **/>**
    **<****Button****Grid.Row****=****"3"****Grid.Column****=****"****2"****Text****=****"3"** **/>**
    **<****Button****Grid.Row****=****"3"****Grid.Column****=****"3"****Text****=****"****="** **/>**
    **</****Grid****>**
    **<****Label****x:Name****=****"Output"****FontSize****=****"24"**
    **VerticalOptions****=****"****Center"**
    **HorizontalOptions****=****"Start"** **/>**
      </VerticalStackLayout>
    </ContentPage> 
    ```

1.  为页面的 `Loaded` 事件添加一个语句，如图中高亮显示的以下标记所示：

    ```cs
    Title="Employees"
    **Loaded=****"ContentPage_Loaded"**> 
    ```

1.  在 `EmployeesPage.xaml.cs` 中，添加语句来调整网格中每个按钮的大小，并为 `Clicked` 事件连接事件处理器，如图中以下代码所示：

    ```cs
    private void ContentPage_Loaded(object sender, EventArgs e)
    {
      foreach (Button button in GridCalculator.Children.OfType<Button>())
      {
        button.FontSize = 24;
        button.WidthRequest = 54;
        button.HeightRequest = 54;
        button.Clicked += Button_Clicked;
      }
    } 
    ```

1.  添加一个 `Button_Clicked` 方法，包含处理点击按钮的语句，将按钮的文本连接到输出标签，如图中以下代码所示：

    ```cs
    private void Button_Clicked(object sender, EventArgs e)
    {
      string operationChars = "+-/X=";
      Button button = (Button)sender;
      if (operationChars.Contains(button.Text))
      {
        Output.Text = string.Empty;
      }
      else
      {
        Output.Text += button.Text;
      }
    } 
    ```

    这不是一个合适的计算器实现，因为操作尚未实现。我们现在只是模拟一个，因为我们专注于如何使用.NET MAUI 构建 UI。您可以通过 Google 搜索如何实现一个简单的计算器作为可选练习。

1.  使用至少一个桌面设备和至少一个移动设备启动 `Northwind.Maui.Client` 项目。

1.  导航到**员工**，点击一些按钮，注意标签更新以显示所点击的内容，如图 *图 16.10* 所示：

![图片](img/B19587_16_10.png)

图 16.10：模拟器上的模拟计算器

1.  关闭应用程序。

到目前为止，我们已经使用 XAML 和 MAUI 构建了一些简单的 UI。接下来，让我们看看一些改进应用的技术，比如定义和共享资源。

# 使用共享资源

当构建图形用户界面时，你通常会想要使用资源，例如画笔来绘制控件背景或类的实例以执行自定义转换。资源可以在以下级别定义，并与该级别或以下级别的所有内容共享：

+   应用程序

+   页面

+   控件

## 定义跨应用共享的资源

定义共享资源的好地方是在应用级别，那么让我们看看如何做到这一点：

1.  在 `Resources` 文件夹中，在 `Styles` 文件夹中，添加一个名为 `Northwind.xaml` 的新 **.NET MAUI 资源字典 (XAML**) 项目项。

    Visual Studio Code 和 JetBrains Rider 没有 MAUI 的项目项模板。你可以使用 CLI 创建此项目项，如下所示：

    ```cs
    dotnet new maui-dict-xaml --name Northwind.xaml 
    ```

1.  在现有的 `ResourceDictionary` 元素内部添加标记，以定义具有 `Rainbow` 键的线性渐变画笔，如下所示，高亮显示的标记：

    ```cs
    <?xml version="1.0" encoding="utf-8" ?>
    <ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                 xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                 x:Class="Northwind.Maui.Client.Resources.Styles.Northwind">
    **<****LinearGradientBrush****x:Key****=****"Rainbow"****>**
    **<****GradientStop****Color****=****"Red"****Offset****=****"0"** **/>**
    **<****GradientStop****Color****=****"Orange"****Offset****=****"****0.1"** **/>**
    **<****GradientStop****Color****=****"Yellow"****Offset****=****"0.3"** **/>**
    **<****GradientStop****Color****=****"****Green"****Offset****=****"0.5"** **/>**
    **<****GradientStop****Color****=****"Blue"****Offset****=****"0.7"** **/>**
    **<****GradientStop****Color****=****"Indigo"****Offset****=****"0.9"** **/>**
    **<****GradientStop****Color****=****"Violet"****Offset****=****"****1"** **/>**
    **</****LinearGradientBrush****>**
    </ResourceDictionary> 
    ```

1.  在 `App.xaml` 中，向合并的资源字典中添加一个条目以引用 `Styles` 文件夹中名为 `Northwind.xaml` 的资源文件，如下所示，高亮显示的标记：

    ```cs
    <?xml version = "1.0" encoding = "UTF-8" ?>
    <Application xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                 xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                 xmlns:local="clr-namespace:Northwind.Maui.Client"
                 x:Class="Northwind.Maui.Client.App">
      <Application.Resources>
        <ResourceDictionary>
          <ResourceDictionary.MergedDictionaries>
            <ResourceDictionary Source="Resources/Styles/Colors.xaml" />
            <ResourceDictionary Source="Resources/Styles/Styles.xaml" />
    **<****ResourceDictionary****Source****=****"Resources/Styles/Northwind.xaml"** **/>**
          </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
      </Application.Resources>
    </Application> 
    ```

## 引用共享资源

现在我们可以引用共享资源：

1.  在 `CategoriesPage.xaml` 中，修改 `ContentPage` 以将其背景设置为具有 `Rainbow` 键的画笔资源，如下所示，高亮显示的标记：

    ```cs
    <ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                 xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                 x:Class="Northwind.Maui.Client.CategoriesPage"
    **Background****=****"{StaticResource Rainbow}"**
               Title="Categories"> 
    ```

    `StaticResource` 表示资源在应用首次启动时只读取一次。如果之后资源发生变化，引用它的任何元素都不会更新。

1.  以调试模式启动 `Northwind.Maui.Client` 项目。

1.  导航到 **类别** 并注意页面的背景是一个彩虹。

1.  关闭应用。

## 动态更改共享资源

现在我们可以实现一个设置页面，允许用户在运行时在浅色模式、深色模式或 UI 中使用的系统模式之间切换：

1.  在 `Resources` 文件夹中，在 `Styles` 文件夹中，添加一个名为 `LightDarkModeColors.xaml` 的新 **.NET MAUI 资源字典 (XAML**) 项目项。

1.  在现有的 `ResourceDictionary` 元素内部添加标记，以定义浅色模式和深色模式适用的颜色集，如下所示，高亮显示的标记：

    ```cs
    <?xml version="1.0" encoding="utf-8" ?>
    <ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
      xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
      x:Class="Northwind.Maui.Client.Resources.Styles.LightDarkModeColors">
      <Color x:Key="LightPageBackgroundColor">White</Color>
      <Color x:Key="LightNavigationBarColor">AliceBlue</Color>
      <Color x:Key="LightPrimaryColor">WhiteSmoke</Color>
      <Color x:Key="LightSecondaryColor">Black</Color>
      <Color x:Key="LightPrimaryTextColor">Black</Color>
      <Color x:Key="LightSecondaryTextColor">White</Color>
      <Color x:Key="LightTertiaryTextColor">Gray</Color>
      <Color x:Key="DarkPageBackgroundColor">Black</Color>
      <Color x:Key="DarkNavigationBarColor">Teal</Color>
      <Color x:Key="DarkPrimaryColor">Teal</Color>
      <Color x:Key="DarkSecondaryColor">White</Color>
      <Color x:Key="DarkPrimaryTextColor">White</Color>
      <Color x:Key="DarkSecondaryTextColor">White</Color>
      <Color x:Key="DarkTertiaryTextColor">WhiteSmoke</Color>
    </ResourceDictionary> 
    ```

1.  在 `Resources` 文件夹中，在 `Styles` 文件夹中，添加一个名为 `DarkModeTheme.xaml` 的新 **.NET MAUI 资源字典 (XAML**) 项目项。

1.  在现有的 `ResourceDictionary` 元素内部添加标记，以定义用于深色模式的样式，如下所示，高亮显示的标记：

    ```cs
    <?xml version="1.0" encoding="utf-8" ?>
    <ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
      xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
      x:Class="Northwind.Maui.Client.Resources.Styles.DarkModeTheme">
      <Style TargetType="Shell">
        <Setter Property="FlyoutBackgroundColor" 
                Value="{StaticResource DarkNavigationBarColor}" />
      </Style>
      <Style TargetType="ContentPage">
        <Setter Property="BackgroundColor" 
                Value="{StaticResource DarkPageBackgroundColor}" />
      </Style>
      <Style TargetType="Button">
        <Setter Property="BackgroundColor"
                Value="{StaticResource DarkPrimaryColor}" />
        <Setter Property="TextColor"
                Value="{StaticResource DarkSecondaryColor}" />
        <Setter Property="HeightRequest" Value="45" />
        <Setter Property="WidthRequest" Value="190" />
        <Setter Property="CornerRadius" Value="18" />
      </Style>
    </ResourceDictionary> 
    ```

1.  在 `Resources` 文件夹中，在 `Styles` 文件夹中，添加一个名为 `LightModeTheme.xaml` 的新 **.NET MAUI 资源字典 (XAML**) 项目项。

1.  在现有的 `ResourceDictionary` 元素内部添加标记，以定义用于浅色模式的样式，如下所示，高亮显示的标记：

    ```cs
    <?xml version="1.0" encoding="utf-8" ?>
    <ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
      xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
      x:Class="Northwind.Maui.Client.Resources.Styles.LightModeTheme">
      <Style TargetType="Shell">
        <Setter Property="FlyoutBackgroundColor" 
                Value="{StaticResource LightNavigationBarColor}" />
      </Style>
      <Style TargetType="ContentPage">
        <Setter Property="BackgroundColor" 
                Value="{StaticResource LightPageBackgroundColor}" />
      </Style>
      <Style TargetType="Button">
        <Setter Property="BackgroundColor"
                Value="{StaticResource LightPrimaryColor}" />
        <Setter Property="TextColor"
                Value="{StaticResource LightSecondaryColor}" />
        <Setter Property="HeightRequest" Value="45" />
        <Setter Property="WidthRequest" Value="190" />
        <Setter Property="CornerRadius" Value="18" />
      </Style>
    </ResourceDictionary> 
    ```

1.  在 `Resources` 文件夹中，在 `Styles` 文件夹中，添加一个名为 `SystemModeTheme.xaml` 的新 **.NET MAUI 资源字典 (XAML**) 项目项。

1.  在现有的`ResourceDictionary`元素内添加标记，以定义根据操作系统选项设置来使用的样式，如下面的标记所示：

    ```cs
    <?xml version="1.0" encoding="utf-8" ?>
    <ResourceDictionary xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
      xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
      x:Class="Northwind.Maui.Client.Resources.Styles.SystemModeTheme">
      <Style TargetType="Shell">
        <Setter Property="FlyoutBackgroundColor" 
                Value="{AppThemeBinding Light={StaticResource LightNavigationBarColor}, Dark={StaticResource DarkNavigationBarColor}}" />
      </Style>
      <Style TargetType="ContentPage">
        <Setter Property="BackgroundColor" 
                Value="{AppThemeBinding Light={StaticResource LightPageBackgroundColor}, Dark={StaticResource DarkPageBackgroundColor}}" />
      </Style>
      <Style TargetType="Button">
        <Setter Property="BackgroundColor"
                Value="{AppThemeBinding Light={StaticResource LightPrimaryColor}, Dark={StaticResource DarkPrimaryColor}}" />
        <Setter Property="TextColor"
                Value="{AppThemeBinding Light={StaticResource LightSecondaryColor}, Dark={StaticResource DarkSecondaryColor}}" />
        <Setter Property="HeightRequest" Value="45" />
        <Setter Property="WidthRequest" Value="190" />
        <Setter Property="CornerRadius" Value="18" />
      </Style>
    </ResourceDictionary> 
    ```

    注意使用`AppThemeBinding`扩展来动态绑定到两个预定义的特殊过滤器`Light`和`Dark`。这些绑定到系统模式。

1.  在`App.xaml`中添加浅色和暗色模式颜色以及系统主题资源，如下面的标记所示：

    ```cs
    <ResourceDictionary.MergedDictionaries>
      <ResourceDictionary Source="Resources/Styles/Colors.xaml" />
      <ResourceDictionary Source="Resources/Styles/Styles.xaml" />
      <ResourceDictionary Source="Resources/Styles/Northwind.xaml" />
    **<****ResourceDictionary****Source****=****"Resources/Styles/LightDarkModeColors.xaml"** **/>**
    **<****ResourceDictionary****Source****=****"Resources/Styles/SystemModeTheme.xaml"** **/>**
    </ResourceDictionary.MergedDictionaries> 
    ```

    **良好实践**：`SystemModeTheme.xaml`资源文件引用了在`LightDarkModelColors.xaml`文件中定义的颜色，因此顺序很重要。

1.  在项目中创建一个名为`Controls`的文件夹。

1.  在`Controls`文件夹中，添加一个名为`ThemeEnum.cs`的类文件，并定义一个包含三个值`System`、`Light`和`Dark`的`enum`类型，用于选择主题，如下面的代码所示：

    ```cs
    namespace Northwind.Maui.Client.Controls;
    public enum Theme
    {
      System,
      Light,
      Dark
    } 
    ```

1.  在`Controls`文件夹中，添加一个名为`EnumPicker.cs`的类文件，并定义一个从`Picker`控件继承的类，该类可以绑定到任何`enum`类型并显示其值的下拉列表，如下面的代码所示：

    ```cs
    using System.Reflection; // To use GetTypeInfo method.
    namespace Northwind.Maui.Client.Controls;
    public class EnumPicker : Picker
    {
      public Type EnumType
      {
        set => SetValue(EnumTypeProperty, value);
        get => (Type)GetValue(EnumTypeProperty);
      }
      public static readonly BindableProperty EnumTypeProperty =
        BindableProperty.Create(
          propertyName: nameof(EnumType), 
          returnType: typeof(Type), 
          declaringType: typeof(EnumPicker),
          propertyChanged: (bindable, oldValue, newValue) =>
        {
          EnumPicker picker = (EnumPicker)bindable;
          if (oldValue != null)
          {
            picker.ItemsSource = null;
          }
          if (newValue != null)
          {
            if (!((Type)newValue).GetTypeInfo().IsEnum)
              throw new ArgumentException(
                "EnumPicker: EnumType property must be enumeration type");
            picker.ItemsSource = Enum.GetValues((Type)newValue);
          }
        });
    } 
    ```

1.  在`SettingsPage.xaml`中，导入一个`local`命名空间以使用我们的自定义`EnumPicker`控件，一个`ios`命名空间以添加仅适用于 iOS 应用程序的特殊属性，将`Title`更改为`设置`，并创建一个`EnumPicker`实例以选择主题，如下面的标记所示：

    ```cs
    <?xml version="1.0" encoding="utf-8" ?>
    <ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                 xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    **xmlns:local****=****"clr-namespace:Northwind.Maui.Client.Controls"**
    **xmlns:ios****=****"clr-namespace:Microsoft.Maui.Controls.PlatformConfiguration.iOSSpecific;assembly=Microsoft.Maui.Controls"**
                 x:Class="Northwind.Maui.Client.SettingsPage"
                 Title="**Settings**">
      <VerticalStackLayout **HorizontalOptions****=****"Center"**>
      **<****local:EnumPicker****ios:Picker.UpdateMode****=****"WhenFinished"**
    **EnumType****=****"{x:Type local:Theme}"**
    **Title****=****"Select Theme"**
    **SelectedIndexChanged****=****"ThemePicker_SelectionChanged"**
    **Loaded****=****"ThemePicker_Loaded"**
    **x:Name****=****"ThemePicker"** **/>**
    </VerticalStackLayout>
    </ContentPage> 
    ```

1.  在`SettingsPage.xaml.cs`中添加处理选择器事件的语句，如下面的代码所示：

    ```cs
    **using** **Northwind.Maui.Client.Controls;** **// To use enum Theme.**
    **using** **Northwind.Maui.Client.Resources.Styles;** **// To use DarkModeTheme.**
    namespace Northwind.Maui.Client;
    public partial class SettingsPage : ContentPage
    {
      public SettingsPage()
      {
        InitializeComponent();
      }
    **private****void****ThemePicker_SelectionChanged****(****object** **sender, EventArgs e****)**
     **{**
     **Picker picker = sender** **as** **Picker;**
     **Theme theme = (Theme)picker.SelectedItem;**
     **ICollection<ResourceDictionary> resources =** 
     **Application.Current.Resources.MergedDictionaries;**
    **if** **(resources** **is****not****null****)**
     **{**
     **resources.Clear();**
     **resources.Add(****new** **Resources.Styles.Northwind());**
     **resources.Add(****new** **LightDarkModeColors());**
     **ResourceDictionary themeResource = theme** **switch**
     **{**
     **Theme.Dark  =>** **new** **DarkModeTheme(),**
     **Theme.Light =>** **new** **LightModeTheme(),**
     **_           =>** **new** **SystemModeTheme()**
     **};**
     **resources.Add(themeResource);**
     **}**
     **}**
    **private****void****ThemePicker_Loaded****(****object** **sender, EventArgs e****)**
     **{**
     **ThemePicker.SelectedItem = Theme.System;**
     **}**
    **}** 
    ```

1.  使用至少一个桌面和移动设备启动`Northwind.Maui.Client`项目。

1.  注意，主页上按钮的颜色和形状处于浅色模式，如图 16.11 中的**Window**机器所示：

![](img/B19587_16_11.png)

图 16.11：Windows 上的浅色模式按钮

1.  保持应用程序运行，启动 Windows**设置**应用程序，导航到**个性化**|**颜色**，在**选择你的模式**部分选择**暗色**，如图 16.12 所示：

![](img/B19587_16_12.png)

图 16.12：在 Windows 设置中切换系统颜色模式

1.  保持**设置**打开，切换回应用程序，并注意它已动态切换到暗色模式颜色，如图 16.13 所示：

![](img/B19587_16_13.png)

图 16.13：我们应用程序中的暗色模式

1.  关闭应用程序。

1.  在**设置**中，切换回**浅色**模式。

**良好实践**：资源可以是任何对象的实例。为了在应用程序中共享它，请在`App.xaml`文件中定义它并给它一个唯一的键。为了在应用程序首次启动时一次性设置元素的属性，请使用`{StaticResource key}`。为了在应用程序的生命周期内资源值变化时设置元素的属性，请使用`{DynamicResource key}`。为了使用代码加载资源，请使用`Resources`属性的`TryGetValue`方法。如果你将`Resources`属性视为字典并使用数组样式语法，如`Resources[key]`，它将只找到在字典中直接定义的资源，而不是在任何合并的字典中。

资源可以在 XAML 的任何元素中定义和存储，而不仅仅是应用程序级别。例如，如果一个资源只在`MainPage`上需要，那么它可以在那里定义。您还可以在运行时动态加载 XAML 文件。

**更多信息**：您可以在以下链接中了解更多关于.NET MAUI 资源字典的信息：[`learn.microsoft.com/en-us/dotnet/maui/fundamentals/resource-dictionaries`](https://learn.microsoft.com/en-us/dotnet/maui/fundamentals/resource-dictionaries)。特别是注意关于资源查找行为的章节。

# 使用数据绑定

当构建图形用户界面时，你通常会想要将一个控件的一个属性绑定到另一个控件，或者绑定到某些数据。

## 绑定到元素

最简单的绑定类型是两个元素之间的绑定。一个元素作为值的源，另一个元素作为目标。

让我们采取以下步骤：

1.  在`CategoriesPage.xaml`中，在现有的垂直堆叠布局中的按钮下方，添加一个用于说明的标签，另一个用于显示当前旋转角度的标签，一个用于选择旋转的滑动条，以及一个彩虹方块用于旋转，如下面的标记所示：

    ```cs
    <?xml version="1.0" encoding="utf-8" ?>
    <ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
                 xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                 x:Class="Northwind.Maui.Client.CategoriesPage"
                 Background="{StaticResource Rainbow}"
                 Title="Categories">

      <VerticalStackLayout>

        <Button Text="Click Me" WidthRequest="100" 
                x:Name="ClickMeButton" 
                Clicked="ClickMeButton_Clicked" />

    **<****Label****Margin****=****"10"****>**
     **Use the slider to rotate the square:**
    **</****Label****>**
    **<****Label****BindingContext****=****"{x:Reference Name=SliderRotation}"**
    **Text****=****"{Binding Path=Value, StringFormat='{0:N0} degrees'}"**
    **FontSize****=****"30"****HorizontalTextAlignment****=****"****Center"** **/>**
    **<****Slider****Value****=****"0"****Minimum****=****"0"****Maximum****=****"180"**
    **x:Name****=****"****SliderRotation"****Margin****=****"10,0"** **/>**

    **<****Rectangle****HeightRequest****=****"200"****WidthRequest****=****"200"**
    **Fill****=****"****{StaticResource Rainbow}"**
    **BindingContext****=****"{x:Reference Name=SliderRotation}"**
    **Rotation****=****"{Binding Path=Value}"** **/>**
      </VerticalStackLayout>

    </ContentPage> 
    ```

    注意，标签的文本和矩形的旋转角度都是通过绑定上下文和`{Binding}`标记扩展绑定到滑动条的值。

1.  启动`Northwind.Maui.Client`项目。

1.  导航到**类别**页面。

1.  点击并拖动滑动条以更改彩虹方块的旋转，如图*图 16.14*所示：

![图片](img/B19587_16_14.png)

图 16.14：一个与标签绑定并旋转的矩形在 Windows 上的滑动数据

1.  关闭应用程序。

# 练习和探索

通过回答一些问题，进行一些实际操作练习，并更深入地研究本章的主题来测试你的知识和理解。

## 练习 16.1 – 测试你的知识

回答以下问题：

1.  .NET MAUI UI 组件的四个类别是什么，它们分别代表什么？

1.  `Shell`组件的好处是什么，它实现了哪些类型的 UI？

1.  你如何让用户能够在列表视图中的一个单元格上执行操作？

1.  你会在什么情况下使用`Entry`而不是`Editor`？

1.  将菜单项的`IsDestructive`设置为`true`对单元格上下文操作中的菜单项有什么影响？

1.  你已经定义了一个包含内容页的 `Shell`，但没有显示导航。这可能是为什么？

1.  对于像 `Button` 这样的元素，`Margin` 和 `Padding` 之间的区别是什么？

1.  如何使用 XAML 将事件处理器附加到对象？

1.  XAML 样式的作用是什么？

1.  你在哪里可以定义资源？

## 练习 16.2 – 探索主题

使用下一页上的链接了解更多关于本章涵盖主题的详细信息：

[`github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-16---building-mobile-and-desktop-apps-using-net-maui`](https://github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-16---building-mobile-and-desktop-apps-using-net-maui)

## 练习 16.3 – 实现 .NET MAUI 的模型-视图-视图模型

在这个仅在线的部分，你将学习如何使用 .NET MAUI 应用实现 MVVM 设计模式：

[`github.com/markjprice/apps-services-net8/blob/main/docs/ch16-mvvm.md`](https://github.com/markjprice/apps-services-net8/blob/main/docs/ch16-mvvm.md)

## 练习 16.4 – 集成 .NET MAUI 应用与 Blazor 和原生平台

在这个仅在线的部分，你将学习如何将 .NET MAUI 应用与原生移动功能集成：

[`github.com/markjprice/apps-services-net8/blob/main/docs/ch16-maui-blazor.md`](https://github.com/markjprice/apps-services-net8/blob/main/docs/ch16-maui-blazor.md)

# 摘要

在本章中，你学习了：

+   如何使用 .NET MAUI 构建跨平台的移动和桌面应用程序。

+   如何定义共享资源并引用它们。

+   如何使用常见控件进行数据绑定。

在 *结语* 中，你将学习如何通过 .NET 的应用程序和服务继续你的学习之旅。
