# 第六章：*第六章*：显示图表和自定义 2D 图形中的数据

本章将介绍需要显示图形、报告和复杂图形的应用程序。应用程序通常包括某种图形或图表。还越来越常见的是在 UI 中包含无法轻松使用标准控件制作的元素。

随着我们在本章的进展，我们将为我们虚构的业务构建一个仪表板应用程序，显示适合业务不同部分的信息。这样的应用程序在管理报告工具中很常见。您可以想象不同的屏幕显示在每个部门墙上安装的监视器上。这使员工可以立即看到他们所在业务部门的情况。

在本章中，我们将涵盖以下主题：

+   显示图形和图表

+   使用 SkiaSharp 创建自定义图形

+   使 UI 布局对屏幕尺寸的变化做出响应

在本章结束时，您将创建一个仪表板应用程序，显示在 UWP 和 Web 上运行的财务、运营和网络信息。它还将适应不同的屏幕比例，因此每个页面的内容都会考虑不同的屏幕尺寸和纵横比。

# 技术要求

本章假设您已经设置好了开发环境，包括安装了项目模板，就像在*第一章*中介绍的那样，*介绍 Uno Platform*。本章的源代码位于[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/Chapter06`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/Chapter06)。

本章的代码使用了来自[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/SharedLibrary`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/SharedLibrary)的库。

查看以下视频，以查看代码的实际运行情况：[`bit.ly/3iDchtK`](https://bit.ly/3iDchtK)

# 介绍应用程序

本章中我们将构建的应用程序名为**Dashboard**。这是一个显示业务部门内当前活动的应用程序。这不是所有员工都可以使用的东西，但为了让我们专注于本章的特性和兴趣领域，我们不会关心访问权限是如何控制的。这个应用程序的真实版本将有许多功能，但我们只会实现三个：

+   显示当前的财务信息

+   显示实时的运营信息

+   显示火车目前在网络中的位置

由于这个应用程序将被办公室工作人员使用，它将在桌面上（通过 UWP）和在 Web 浏览器上（使用 WASM 版本）可用。

## 创建应用程序

我们将从创建应用程序的解决方案开始。

1.  在 Visual Studio 中，使用**多平台应用程序（Uno Platform）**模板创建一个新项目。

1.  给项目命名为`Dashboard`。您可以使用不同的名称，但需要相应调整所有后续的代码片段。

1.  删除所有平台头项目，**除了** **UWP** 和 **WASM**。

1.  为了避免写更多的代码，我们现在将添加对共享库项目的引用。右键单击`UnoBookRail.Common.csproj`文件中的解决方案节点，然后单击**打开**。

1.  对于每个特定于平台的项目，我们需要添加对共享库项目的引用。右键单击`UnoBookRail.Common`，然后单击**确定**。现在*重复此过程以进行 WASM 项目*。

现在基本的解决方案结构已经准备好，我们可以向主页面添加一些功能。

## 创建各个页面

我们将为要显示的每个功能区域使用单独的页面：

1.  在`Views`中创建一个新文件夹。

1.  在`Views`文件夹中，添加名为`FinancePage.xaml`、`OperationsPage.xaml`和`NetworkPage.xaml`的*三个*新页面。

现在我们将更新主页面以在这些新页面之间进行导航。

## 创建主页面

该应用程序已经包含了文件`MainPage.xaml`，我们将使用它作为在其他页面之间导航的容器：

1.  用包含每个我们将实现的单独页面选项的以下`NavigationView`控件替换`MainPage.xaml`中的网格：

```cs
<NavigationView
    PaneDisplayMode="Top"
    SelectionChanged="NavItemSelected"
    IsBackEnabled="{Binding Path=CanGoBack, 
                    ElementName=InnerFrame}"
    BackRequested="NavBackRequested"
    IsSettingsVisible="False">
    <NavigationView.MenuItems>
        <NavigationViewItem Content="Finance" />
        <NavigationViewItem Content="Operations" />
        <NavigationViewItem Content="Network" />
    </NavigationView.MenuItems>
    <Frame x:Name="InnerFrame" />
</NavigationView>
```

1.  我们现在需要添加`NavItemSelected`事件的处理程序，以执行页面之间的实际导航。在`MainPage.xaml.cs`中添加以下内容：

```cs
using Dashboard.Views;
private void NavItemSelected(NavigationView sender, NavigationViewSelectionChangedEventArgs args) 
{
  var item = (args.SelectedItem as 
              NavigationViewItem).Content.ToString();
  Type page = null;
  switch (item) {
    case "Finance":
      page = typeof(FinancePage);
      break;
    case "Operations":
      page = typeof(OperationsPage);
      break;
    case "Network":
      page = typeof(NetworkPage);
      break;
  }
  if (page != null && InnerFrame.CurrentSourcePageType
      != page) {
    InnerFrame.Navigate(page);
  }
}
```

1.  我们还需要实现`NavBackRequested`方法来处理用户按下返回按钮导航回页面。添加以下内容来实现这一点：

```cs
private void NavBackRequested(object sender, NavigationViewBackRequestedEventArgs e) 
{
    InnerFrame.GoBack();
}
```

导航

该应用程序使用自定义定义的框架和基于堆栈的导航样式。这允许用户按下内置的返回按钮返回到上一页。虽然这可能不被认为是这个应用程序最合适的方式之一，但这是开发人员在 UWP 应用程序中实现导航的最流行方式之一。因此，我们认为将其包含在本书中并展示它可以被整合到 Uno 平台应用程序中是合适的。

1.  前面的内容将允许我们在菜单中选择项目时在页面之间进行导航，但我们也希望在应用程序首次打开时显示一个页面。为此，在`MainPage`构造函数的*末尾*添加以下调用：

```cs
InnerFrame.Navigate(typeof(FinancePage));
```

重要提示

本节中的代码显示了在`NavigationView`控件中启用页面之间导航的最简单方法。这当然不是唯一的方法，也不是应该总是这样做的建议。

现在所有基础都已就绪，我们现在可以向财务页面添加一个图表。

# 使用来自 SyncFusion 的控件显示图表

SyncFusion 是一家为 Web、桌面和移动开发制作 UI 组件的公司。他们的 Uno 平台控件在撰写本文时处于测试阶段，并且在预览期间可以免费使用，通过他们的社区许可证([`www.syncfusion.com/products/communitylicense`](https://www.syncfusion.com/products/communitylicense))。有许多不同的图表类型可用，但我们将使用线图来创建一个类似于*图 6.1*所示的页面。图表显示在一些箭头旁边，提供一些一般趋势数据，以便查看它们的人可以快速了解数据的摘要。想象它们代表数据与上周、上个月和去年同一天相比较的情况：

![图 6.1-包括来自 SyncFusion 的图表的财务信息](img/Figure_6.01_B17132.jpg)

图 6.1-包括来自 SyncFusion 的图表的财务信息

## 更新引用以包括 SyncFusion 控件

SyncFusion Uno 图表控件的测试版本可在 GitHub 上获得完整的源代码：

1.  从[`github.com/syncfusion/Uno.SfChart`](https://github.com/syncfusion/Uno.SfChart)下载或克隆代码。

1.  通过右键单击解决方案并选择**添加** | **现有项目…**，将**Syncfusion.SfChart.Uno.csproj**项目添加到解决方案中。

1.  更新**Syncfusion.SfChart.Uno**项目以使用最新版本的**Uno.UI**包。这是为了避免在解决方案中的不同项目中使用不同版本的库时出现任何问题。

1.  从*UWP*和*WASM*项目中引用**Syncfusion.SfChart.Uno**项目。

我们现在可以在应用程序中使用这些控件。

重要提示

由于 SyncFusion 控件仅从源代码中获取，虽然不太可能，但当您阅读本文时它们可能已经发生了变化。希望可以获得编译版本的控件，但如果您需要达到与本文撰写时相当的状态，请使用提交**43cd434**。

## 绘制线图

我们可以通过以下步骤绘制一个简单的线图：

1.  首先将此命名空间添加到`FinancePage.xaml`中：

```cs
xmlns:sf="using:Syncfusion.UI.Xaml.Charts"
```

1.  现在用以下内容替换网格：

```cs
<RelativePanel HorizontalAlignment="Center">
  <sf:SfChart class we can specify. We define a PrimaryAxis class (for the X-axis), which reflects the hours of the day, with a SecondaryAxis class (for the Y-axis) representing the numeric values and a set of data as a LineSeries class.We also specify a `TextBlock` element to appear below the chart but be horizontally aligned. This will display arrows indicating trend information relating to the graph.
```

1.  为了提供数据，我们需要在`FinancePage.xaml.cs`中的类中添加以下内容：

```cs
public List<HourlySales> DailySales
    => FinanceInfo.DailySales
       .Select(s => new HourlySales(s.Hour, 
            s.Sales)).ToList();
public string TrendArrows => FinanceInfo.TrendArrows;
```

1.  这些属性需要您添加此`using`声明：

```cs
using UnoBookRail.Common.DashboardData;
```

1.  我们还必须创建以下类，`SfChart`对象将使用它来查找我们在 XAML 中引用的命名属性：

```cs
public class HourlySales
{
    public HourlySales(string hour, double totalSales) 
    {
        Hour = hour;
        TotalSales = totalSales;
    }
    public string Hour { get; set; }
    public double TotalSales { get; set; }
}
```

显然，我们在这里只创建了一个简单的图表，但关键是要注意它是多么容易。一个真正的仪表板可能会显示不止一个图表。您可以在存储库中包含的示例应用程序中看到您可以包含的图表的示例[`github.com/syncfusion/Uno.SfChart`](https://github.com/syncfusion/Uno.SfChart)。

我们已经看到了如何轻松地包含来自一个供应商的图表来显示财务信息。现在让我们添加另一个供应商的图表，以显示一些不同的信息。

# 使用 Infragistics 控件显示图表

Infragistics 是一家为各种平台提供 UI 和 UX 工具的公司。他们还有一系列控件可供 Uno 平台应用程序使用，在预览期间免费使用。

您可以在[`www.infragistics.com/products/uno-platform`](https://www.infragistics.com/products/uno-platform)了解更多关于这些控件的信息，或者跟随我们为应用程序添加图表，以显示与 UnoBookRail 业务的当前操作相关的信息，并创建一个看起来像*图 6.2*的页面：

![图 6.2 - 来自 Infragistics 的图表上显示的网络操作详细信息](img/Figure_6.02_B17132.jpg)

图 6.2 - 来自 Infragistics 的图表上显示的网络操作详细信息

## 更新引用

为了能够在我们的应用程序中使用这些控件，我们必须首先进行以下修改：

1.  在**UWP**项目中引用`Infragistics.Uno.Charts` NuGet 包：

```cs
Install-Package Infragistics.Uno.Charts -Version 20.2.59-alpha
```

1.  在**WASM**项目中引用`Infragistics.Uno.Wasm.Charts` NuGet 包：

```cs
Install-Package Infragistics.Uno.Wasm.Charts -Version 20.2.59-alpha
```

1.  在**WASM**项目中引用`Uno.SkiaSharp.Views`和`Uno.SkiaSharp.Wasm` NuGet 包。这是必要的，因为 Infragistics 控件使用 SkiaSharp 来绘制控件。这与我们之前使用的 SyncFusion 控件不同，后者使用 XAML：

```cs
Install-Package Uno.SkiaSharp.Views -Version 2.80.0-uno.493
Install-Package Uno.SkiaSharp.Wasm -Version 2.80.0-uno.493
```

通过这些简单的修改，我们现在可以将图表添加到我们的应用程序中。

重要提示

如果在进行上述更改后注意到任何奇怪的编译行为，请尝试清理解决方案，关闭所有打开的 Visual Studio 实例，然后重新打开解决方案。这不应该是必要的，但我们发现在某些情况下需要这样做。

您可能还会在 SyncFusion 项目的错误列表中看到条目，尽管它成功编译。这些错误可以安全地忽略。

## 绘制柱状图

现在我们将为应用程序的**Operations**页面添加内容。为简单起见，我们只添加两条信息。我们将添加一个图表，显示今天每小时使用了多少张票的类型。此外，我们将根据持票进入车站但随后没有出站的人数，显示目前在火车上或车站中的人数：

1.  将以下命名空间添加到`OperationsPage.xaml`的`Page`元素中：

```cs
xmlns:ig="using:Infragistics.Controls.Charts"
```

1.  现在将以下 XAML 添加为页面的内容：

```cs
<Grid>
  <Grid.RowDefinitions>
    <RowDefinition Height="*" />
    <RowDefinition Height="*" />
  </Grid.RowDefinitions>
  <Grid.ColumnDefinitions>
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="*" />
  </Grid.ColumnDefinitions>
  <ig:XamDataChart class. Within this, we specify the *x* and *y* axes and the data to display as a StackedColumnSeries element. Within the series, we detail the paths to the data for each fragment of the stack.Finally, we added the `TextBlock` element that displays the current passenger count.
```

1.  在`OperationsPage.xaml.cs`中添加以下`using`指令：

```cs
using UnoBookRail.Common.DashboardData;
```

这些是我们将添加到此文件的属性所需的。

1.  将以下内容添加到`OperationsPage`类中，提供图表中显示的数据：

```cs
public string PsngrCount => OperationsInfo.CurrentPassengers;
private List<PersonCount> Passengers
   => OperationsInfo.Passengers.Select(p 
       => new PersonCount(p.Hour, p.Children,
           p.Adults, p.Seniors)).ToList();
```

1.  现在我们需要添加刚刚引用的`PersonCount`类：

```cs
public class PersonCount 
{
    public PersonCount(string hour, double child,
        double adult, double senior) 
    {
        Hour = hour;
        Children = child;
        Adults = adult;
        Seniors = senior;
    }
    public string Hour { get; set; }
    public double Children { get; set; }
    public double Adults { get; set; }
    public double Seniors { get; set; }
}
```

有了这个，我们现在有一个简单的页面图表，显示每小时旅行的乘客数量。

与 SyncFusion 图表一样，Infragistics 还有许多其他图表和控件可用。您可以在[`github.com/Infragistics/uno-samples`](https://github.com/Infragistics/uno-samples)找到这些示例。

现在我们已经看到了使用第三方库显示更复杂控件的不同方法，让我们来看看如何自己绘制更复杂的东西。

# 使用 SkiaSharp 绘制自定义图形

UWP 和 Uno 平台包括支持创建形状并提供基本绘图功能。然而，有时您需要在应用程序中显示一些无法轻松使用标准控件完成的东西，您需要精细的控制，或者在操作大量 XAML 控件时遇到性能问题。在这些情况下，可能需要直接在 UI 上进行绘制。其中一种方法是使用 SkiaSharp。SkiaSharp 是一个基于 Google 的 Skia 图形库的跨平台 2D 图形 API，我们可以在 Uno 平台应用程序中使用。为了展示使用起来有多简单，我们将创建我们应用程序的最后一部分，显示网络中火车当前位置的地图。只需几行代码，我们就可以创建出类似*图 6.3*中显示的屏幕截图的东西：

![图 6.3-在浏览器中运行时应用程序中显示的网络地图](img/Author_Figure_6.03_B17132.jpg)

图 6.3-在浏览器中运行时应用程序中显示的网络地图

现在你已经看到我们要创建的东西了，让我们开始做吧。

## 更新项目引用

我们在应用程序中需要使用 SkiaSharp 的引用已经作为我们添加到使用 Infragistics 控件的引用的一部分添加。如果您已经进行了这些更改，这里就没有什么要做的了。

如果您在上一节中跟着做，并且*没有*添加 Infragistics 控件，您需要对解决方案进行以下更改：

+   在**WASM**项目中引用`Uno.SkiaSharp.Views`和`Uno.SkiaSharp.Wasm` NuGet 包：

```cs
Install-Package Uno.SkiaSharp.Views -Version 2.80.0-uno.493
Install-Package Uno.SkiaSharp.Wasm -Version 2.80.0-uno.493
```

在添加相关引用之后，我们现在准备绘制网络地图。

## 绘制网络地图

要在应用程序中绘制网络地图，我们需要采取以下步骤：

1.  在`NetworkPage.xaml`中，添加以下作为唯一的内容。这是将显示我们绘制的控件：

```cs
<skia:SKXamlCanvas xmlns:skia="using:SkiaSharp.Views.UWP" PaintSurface="OnPaintSurface" />
```

1.  要在`SKXamlCanvas`控件上绘制地图，我们需要在`NetworkPage.xaml.cs`中添加以下使用声明：

```cs
using SkiaSharp;
using SkiaSharp.Views.UWP;
using UnoBookRail.Common.Mapping;
using UnoBookRail.Common.Network;
```

1.  接下来，我们必须添加我们在 XAML 中引用的`OnPaintSurface`方法。每当控件需要重新绘制图像时，该方法将被控件调用。这将在控件首次加载时以及控件的渲染大小发生变化时发生：

```cs
private void OnPaintSurface(object sender, SKPaintSurfaceEventArgs e) 
{
    var canvas = SetUpCanvas(e);
    DrawLines(canvas);
    DrawStations(canvas);
    DrawTrains(canvas);
}
```

1.  添加`SetUpCanvas`方法来正确初始化和定位图像：

```cs
private SKCanvas SetUpCanvas(SKPaintSurfaceEventArgs e) 
{
  var canvas = e.Surface.Canvas;
  var relativeWidth = e.Info.Width / ImageMap.Width;
  var relativeHeight = 
      e.Info.Height / ImageMap.Height;
  canvas.Scale(Math.Min(relativeWidth, 
      relativeHeight));
  var x = 0f;
  var y = 0f;
  if (relativeWidth > relativeHeight) 
  {
    x = (e.Info.Width - (ImageMap.Width * 
         relativeHeight)) / 2f / relativeHeight;
  }
  else {
    y = (e.Info.Height - (ImageMap.Height * 
         relativeWidth)) / 2f / relativeWidth;
  }
  canvas.Translate(x, y);
  canvas.Clear();
  return canvas;
}
```

`SetUpCanvas`方法调整我们的绘图区域尽可能大，而不会扭曲或拉伸它，并确保它始终水平和垂直居中。最后，它清除画布并返回它，准备让其他方法在其上绘制。

1.  添加`DrawLines`方法来在画布上绘制支线：

```cs
void DrawLines(SKCanvas canvas) 
{
    var paint = new SKPaint 
    {
        Color = SKColors.Black, 
        StrokeWidth = 1,
    };
    var northPnts = 
        ImageMap.GetStations(Branch.NorthBranch);
    var mainPnts = 
        ImageMap.GetStations(Branch.MainLine);
    var southPnts = 
        ImageMap.GetStations(Branch.SouthBranch);
    SKPoint[] ToSKPointArray(List<(float X, float Y)> 
        list)
        => list.Select(p => new SKPoint(p.X, 
            p.Y)).ToArray();
    void DrawBranch(SKPoint[] stnPoints)
        => canvas.DrawPoints(SKPointMode.Polygon, 
            stnPoints, paint);
    DrawBranch(ToSKPointArray(northPnts));
    DrawBranch(ToSKPointArray(mainPnts));
    DrawBranch(ToSKPointArray(southPnts));
}
```

在上面的代码中，库返回的站点位置被转换为 Skia 特定的数组，用于绘制连接所有点的多边形。

1.  添加`DrawStations`方法来在支线上绘制站点位置：

```cs
void DrawStations(SKCanvas canvas) 
{
    var paint = new SKPaint 
    {
        Color = SKColors.Black,
        Style = SKPaintStyle.Fill,
    };
    foreach (var (X, Y) in ImageMap.Stations) 
    {
        canvas.DrawCircle(new SKPoint(X, Y), 2, 
            paint);
    }
}
```

`DrawStations`方法很简单，因为它只是为每个站点绘制一个圆圈。

1.  将`DrawTrains`方法添加到地图上显示火车当前位置的方法：

```cs
void DrawTrains(SKCanvas canvas) 
{
    var trainPaint = new SKPaint 
    {
        Color = SKColors.Cyan,
        Style = SKPaintStyle.Fill,
    };
    foreach (var train in ImageMap.GetTrainsInNetwork()) 
    {
        canvas.DrawCircle(new SKPoint(
            train.MapPosition.X, train.MapPosition.Y),
                1.8f, trainPaint);
    }
}
```

`DrawTrains`方法同样简单，因为它循环遍历提供的数据，并在每个位置绘制一个青色的圆圈。因为这是在站点圆圈之后绘制的，所以当火车在站点时，它会出现在站点上方。

重要提示

在本章中，我们只使用了一些圆圈和线条来创建我们的地图。然而，SkiaSharp 能够做的远不止我们在这里介绍的。您可能希望通过扩展我们刚刚创建的地图来探索其他可用的功能，包括包括站点名称或添加显示火车行驶方向或是否在站点的其他细节。

现在我们已经实现了应用程序的所有页面，但我们可以通过根据屏幕或窗口的大小调整内容来进一步改进。

# 响应 UI 的变化

您的应用程序将需要在不同大小的屏幕和窗口上运行。其中一些差异是由应用程序运行的不同设备引起的，但您可能还需要考虑用户可以调整大小的窗口。

可以设计页面的多个版本，并在运行时加载适当的版本。但通常更容易创建一个根据可用尺寸调整的单个页面。我们将看看如何使用可用的功能来实现这一点。

## 更改页面布局

Uno 平台允许您通过在`VisualStates`之间切换来创建响应式 UI。

可以创建`AdaptiveTrigger`元素，根据其附加的控件的大小触发。现在我们将使用自适应触发器来调整**财务**和**运营**页面，以更好地根据可用宽度布置其内容：

1.  将以下内容添加为`FinancePage.xaml`中`RelativePanel`的第一个子元素：

```cs
<VisualStateManager.VisualStateGroups>
  <VisualStateGroup>
    <VisualState>
      <VisualState.StateTriggers>
        <AdaptiveTrigger element that's applied when the panel is at least 1,200 relative pixels wide. When this visual state is triggered, the TextBlock element is set to the right of the chart and has its alignment adjusted accordingly. The left-hand side of *Figure 6.4* shows how this looks.
```

1.  现在我们可以在`OperationsPage.xaml`页面的网格中做类似的事情。在行和列定义下方立即添加以下内容：

```cs
<VisualStateManager.VisualStateGroups>
  <VisualStateGroup>
    <VisualState>
      <VisualState.StateTriggers>
        <AdaptiveTrigger MinWindowWidth="1200" />
      </VisualState.StateTriggers>
      <VisualState.Setters>
        <Setter Target="PassengerChart.
            (Grid.ColumnSpan)" Value="1"/>
        <Setter Target="PassengerChart.(Grid.RowSpan)"
             Value="2"/>
        <Setter Target="CurrentCount.(Grid.Row)" 
             Value="0"/>
        <Setter Target="CurrentCount.(Grid.Column)" 
            Value="1"/>
        <Setter Target="CurrentCount.
            (Grid.ColumnSpan)" Value="1"/>
        <Setter Target="CurrentCount.
            (Grid.RowSpan)" Value="2"/>
      </VisualState.Setters>
    </VisualState>
  </VisualStateGroup>
</VisualStateManager.VisualStateGroups>
```

通过这些设置器，我们正在利用之前创建的行和列定义。初始代码将控件放在不同的行中，而在这里我们正在更改控件，使它们位于不同的列中，并在窗口更宽时跨越行。如*图 6.4*所示，这意味着当前在火车上的人数显示在图表旁边，而不是下方：

![图 6.4-以横向布局显示的财务和运营页面](img/Figure_6.04_B17132.jpg)

图 6.4-以横向布局显示的财务和运营页面

通过这两个示例，我们已经看到了改变页面上元素重新定位以更改布局的不同方法。没有一种适合所有不同可用空间量的页面的正确方法。状态触发器可用于更改元素上的任何属性，还可以使用多个触发器，因此您可以为小型、中型和大型屏幕制定不同的布局，例如。

更改屏幕上元素的布局不是调整显示内容的唯一方法。控件本身也可以调整、调整大小和重绘以适应空间。

## 拉伸和缩放内容以适应可用空间

XAML 的一个优点是其能够动态布局控件，而不依赖于为每个元素提供特定大小。可以通过设置`HorizontalAlignment`和`VerticalAlignment`属性来调整单个 XAML 控件的大小，以控制它们如何利用可用空间。将这些属性的值设置为`Stretch`将允许它们占用其父元素中的所有可用空间。对于更复杂的情况，还可以使用`ViewBox`元素以不同的方式和方向拉伸控件。

如果您想了解如何使用 XAML 元素创建布局的更多信息，您可以在[`platform.uno/docs/articles/winui-doc-links-development.html#layouting`](https://platform.uno/docs/articles/winui-doc-links-development.html#layouting)找到一些有用的链接。

许多控件也会自动调整以使用所有或尽可能多的可用空间。我们在 SkiaSharp 绘制的地图上做到了这一点。地图被绘制得尽可能大，而不会扭曲。它被放置在可用空间的中心，无论窗口是纵向还是横向纵横比。

现在所有页面都已调整到可用空间，我们的应用程序和本章已经完成。

# 总结

在这一章中，我们构建了一个可以在 UWP 和 Web 浏览器上运行的应用程序。该应用程序使用了 SyncFusion 和 Infragistics 的图形控件。我们还使用 SkiaSharp 创建了一个自定义地图。最后，我们看了如何根据不同和变化的屏幕尺寸调整 UI 布局。

这一章是本书的这一部分的最后一章。在接下来的部分中，我们将从构建应用程序转向如何测试和部署它们。在下一章中，我们将看看如何在更广泛的测试策略中使用`Uno.UITest`库。在构建可以在多个平台上运行的应用程序时，自动化跨平台的测试可以节省大量时间并提高生产率。
