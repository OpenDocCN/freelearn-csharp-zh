# 第四章：*第四章*：使您的应用程序移动化

本章将向您展示如何使用 Uno 平台为移动设备开发应用程序。这样的应用程序可能与在桌面设备或 Web 上运行的应用程序有很大的不同，并带来了您必须考虑的挑战。

在本章中，我们将涵盖以下主题：

+   为运行 iOS 和 Android 的移动设备构建

+   在偶尔连接的环境中使用远程数据

+   为其运行的平台设计应用程序的样式

+   利用应用程序所在设备的功能

在本章结束时，您将创建一个在 Android 和 iOS 设备上运行的移动应用程序，每个平台上的外观都不同，并与远程服务器通信以检索和发送数据。

# 技术要求

本章假设您已经设置好了开发环境，并安装了必要的项目模板，就像我们在*第一章* *介绍 Uno 平台*中所介绍的那样。本章的源代码可以在[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/Chapter04`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/Chapter04)找到。

本章的代码使用以下库：[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/SharedLibrary`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/SharedLibrary)。

本章还从远程 Web 服务器检索数据，您可以使用[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/WebApi`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/WebApi)的代码重新创建。

查看以下视频以查看代码的运行情况：[`bit.ly/3jKGRkI`](https://bit.ly/3jKGRkI)

# 介绍应用程序

我们将在本章中构建的应用程序称为**Network Assist**。这是一个将提供给所有员工使用的应用程序。对于在公共场合工作的人来说，这是特别有用的。这个应用程序的真实版本将有许多功能，但我们只会实现两个：

+   显示下一班火车将到达每个车站的时间

+   记录和报告发生在网络周围的事件的细节。

由于这个应用程序将被员工在整个网络上执行工作时使用，它将被构建为在 Android 和 iOS 设备上运行。

“移动”是什么意思？

很容易认为“移动”只是关于应用程序所在的设备，但这样做是有限制的。“移动”可以是“Android 和 iOS 设备”的一个有用的简称。然而，重要的是要记住，移动不仅仅是指手机（或平板电脑）。使用设备的人也是移动的。考虑将使用应用程序的人通常比运行应用程序的设备更重要。设备只是要考虑的一个因素。一个人可能在过程中使用多个设备，因此需要体验在他们在设备之间移动时也是移动的 - 也许在一个设备上开始一个任务，然后在另一个设备上完成它。

我们构建 Network Assist 应用程序为移动应用程序的主要原因是因为将使用它的人将整天四处旅行。正因为人是移动的，我们才构建了一个在“移动”设备上运行的“移动”应用程序。

与其花费大量时间事先解释功能，不如开始构建应用程序。我们将在编写代码时扩展需求。

## 创建应用程序

我们将从创建应用程序的解决方案开始：

1.  在 Visual Studio 中，使用**多平台应用程序（Uno 平台）**模板创建一个新项目。

1.  将项目命名为`NetworkAssist`。你可以使用不同的名称，但需要相应地调整所有后续的代码片段。

1.  删除所有平台头项目，*除了* **Android**，**iOS**和**UWP**。

始终保留 UWP 头在解决方案中

即使您不打算发布应用程序的 UWP 版本，保留 UWP 头在解决方案中也有两个原因。首先，当诊断任何编译错误时，这可能是有帮助的，以检查代码是否存在基本问题，或者问题是否与 Uno 特定的工具有关。其次，更重要的是，当选择 UWP 头时，Visual Studio 可以提供额外的工具和智能感知。通过在项目中添加 UWP 头，您的 Uno 平台开发体验将更加简单。

1.  为了避免写更多的代码，我们将添加对共享库项目的引用。在`UnoBookRail.Common.csproj`文件中，右键单击解决方案节点，然后点击**打开**。

1.  对于每个特定平台的项目，我们需要添加对通用库项目的引用。在**解决方案资源管理器**中右键单击**Android**项目节点，然后选择**添加 > 引用... > 项目**。然后，选中**UnoBookRail.Common**的条目，然后点击**确定**。现在，*重复此过程用于 iOS 和 UWP 项目*。

基本解决方案结构现在已经准备就绪，我们可以向主页添加一些功能。

## 创建主页

由于这将是一个简单的应用程序，我们将把所有功能放在一个页面上。设计要求是应用程序在屏幕底部有选项卡或按钮，以便在不同功能区域之间进行切换。我们将把不同的功能放在单独的控件中，并根据用户按下的按钮（或选项卡）来更改显示的控件。

这是合适的，因为用户不需要通过他们已经查看过的选项卡后退。

### 允许相机凹口、切口和安全区域

在添加任何自己的内容之前，您可能希望运行应用程序，以检查是否一切都可以编译和调试。根据您运行应用程序的设备或模拟器，您可能会看到*图 4.1*左侧的内容，显示了在 iPhone 12 模拟器上运行的默认应用程序。在这个图中，您可以看到**Hello, World!**文本重叠（或撞到）时间，并且在相机凹口后面。

如果您没有设备可以测试这个功能，一些模拟器可以模拟这个凹口。其他模拟器将有一个可配置的选项，允许在有或没有切口的情况下进行测试。在**设置 > 系统 > 开发人员选项 > 模拟具有切口的显示**下查找：

![图 4.1 - 显示允许状态栏和相机凹口的内容的前后截图](img/Author_Figure_4.01_B17132.jpg)

图 4.1 - 显示允许状态栏和相机凹口的内容的前后截图

我们的应用程序不会有**Hello, World!**文本，但我们不希望我们的内容被遮挡。幸运的是，Uno 平台带有一个辅助类，可以为相机凹口留出空间，无论它们在哪种设备上或者它们的位置如何。

要使用这个辅助类，我们需要做以下几步：

1.  在`MainPage.xaml`的根元素`Page`中添加`xmlns:toolkit="using:Uno.UI.Toolkit"`。

1.  在`Page`元素内部的`Grid`元素中添加`toolkit:VisibleBoundsPadding.PaddingMask="All"`。通过设置`All`的值，如果设备横向旋转，辅助类将提供适当的空间，并且凹口将显示在屏幕的侧面。

现在运行应用程序，你会看到类似于*图 4.1*右侧图像的东西，它展示了布局已经添加了足够的空间。这样可以防止状态栏或相机凹口遮挡我们的内容。

现在我们已经处理了屏幕上的切口，我们可以实现应用程序所需的功能。

### 实现主页面的内容

由于应用程序中只有一个页面，我们现在将实现它：

1.  用以下内容替换`Grid`的现有内容：

```cs
<Grid.RowDefinitions>
    <RowDefinition Height="*" />
    <RowDefinition Height="Auto" />
</Grid.RowDefinitions>
<CommandBar VerticalAlignment="Bottom" Grid.Row="1">
    <CommandBar.PrimaryCommands>
        <AppBarButton Icon="Clock" Label="Arrivals" 
            Click="ShowArrivals" />
        <AppBarButton Label="Quick Report" 
            Click="ShowQuickReport">
            <AppBarButton.Icon>
                <FontIcon Glyph="&#xE724;" />
            </AppBarButton.Icon>
        </AppBarButton>
    </CommandBar.PrimaryCommands>
</CommandBar>
```

网格的顶行将包含不同功能元素的控件。底行将承载选择不同控件的按钮。

我们使用`CommandBar`，因为这是最适合在应用程序中提供选择功能区域按钮的 UWP 控件。这只是我们希望在 iOS 和 Android 上看到的外观的近似值，我们将很快解决这些问题。

注意

XAML 提供了多种方法来实现相似的结果。在本章的代码中，我们使用了最简单的方法来在所有平台上提供一致的输出。

1.  现在我们需要自定义控件来显示不同的功能。首先右键单击`Views`，以匹配存储 UI 相关控件的约定。

如果您愿意，可以将`MainPage`文件移入`Views`文件夹，但这对应用程序的功能并不重要。

1.  在新文件夹中，右键单击并选择`ArrivalsControl`。重复此操作以添加名为`QuickReportControl`的控件。

1.  现在我们将控件添加到`MainPage.xaml`。在页面级别声明一个新的 XML 命名空间别名，值为`xmlns:views="using:Network Assist.Views"`。在`Grid`标签的开头和`CommandBar`之前，添加以下内容以创建我们新控件的实例：

```cs
<views:ArrivalsControl x:Name="Arrivals" Visibility="Visible" />
<views:QuickReportControl x:Name="QuickReport" Visibility="Collapsed" />
```

1.  在代码后台文件（`MainPage.xaml.cs`）中，我们需要添加处理 XAML 中`AppBarButtons`引用的`Click`事件的方法：

```cs
public void ShowArrivals(object sender, RoutedEventArgs args) 
{
    Arrivals.Visibility = Visibility.Visible; 
    QuickReport.Visibility = Visibility.Collapsed;
}
public void ShowQuickReport(object sender, RoutedEventArgs args) 
{
    Arrivals.Visibility = Visibility.Collapsed; 
    QuickReport.Visibility = Visibility.Visible;
}
```

我们将在这里使用点击事件和代码后台，因为逻辑与 UI 紧密耦合，并且不会受益于编写的测试。可以使用`ICommand`实现和绑定来控制每个控件何时显示，但如果您希望这样实现，可以自行实现。

MVVM 和代码后台

在本章中，我们将使用代码后台文件和**Model-View-ViewModel**（**MVVM**）模式的组合。有三个原因。首先，它使我们可以使代码更短，更简单，这样您就更容易跟随。其次，它避免了解释特定的 MVVM 框架或实现的需要，而我们可以专注于与应用程序相关的代码。最后，它表明 Uno 平台不会强迫您以特定方式工作。您可以使用您喜欢的编码风格、模式或框架。

主页面已经运行，现在我们可以添加显示即将到达的详细信息的功能。

## 显示即将到达的详细信息

显示即将到达的要求如下：

+   显示站点列表，并在选择一个站点时，显示每个方向的下三列火车的到达时间。

+   数据可以刷新以确保始终有最新的信息可用。

+   显示检索到最后一条数据的时间。

+   如果未选择站点或检索数据时出现问题，则会显示提示。

+   应用程序指示正在检索数据时。

您可以在本章结束时创建的最终功能示例中看到以下图示：

![图 4.2 - iPhone 上显示的即将到达的详细信息（左）和 Android 设备上（右）](img/Figure_4.02_B17132.jpg)

图 4.2 - iPhone 上显示的即将到达的详细信息（左）和 Android 设备上（右）

用于显示即将到达的用户控件将是应用程序中最复杂的 UI 部分。看起来可能有很多步骤，但每一步都很简单：

1.  首先在`ArrivalsControl.xaml`中的`Grid`中添加两个列定义和四个行定义：

```cs
<Grid.ColumnDefinitions>
    <ColumnDefinition Width="*" />
    <ColumnDefinition Width="Auto" />
</Grid.ColumnDefinitions>
<Grid.RowDefinitions>
    <RowDefinition Height="Auto" />
    <RowDefinition Height="Auto" />
    <RowDefinition Height="Auto" />
    <RowDefinition Height="*" />
</Grid.RowDefinitions>
```

1.  顶部行将包含一个用于选择车站的`ComboBox`控件和一个用于请求刷新数据的`Button`元素：

```cs
<ComboBox x:Name="StationList"
    HorizontalAlignment="Stretch" 
    VerticalAlignment="Stretch"
    ItemsSource="{x:Bind VM.ListOfStations}"
    SelectedItem="{x:Bind VM.SelectedStation, 
        Mode=TwoWay}"
    SelectionChanged="OnStationListSelectionChanged"
    SelectionChangedTrigger="Always">
    <ComboBox.ItemTemplate>
        <DataTemplate xmlns:network="using:UnoBookRail.Common.Network".
```

1.  接下来的两行将使用`TextBlocks`来显示上次检索数据的时间以及检索数据时是否出现问题：

```cs
<TextBlock 
    Grid.Row="1" 
    Grid.ColumnSpan="2" 
    Margin="4"
    HorizontalAlignment="Stretch"
    HorizontalTextAlignment="Right"
    Text="{x:Bind VM.DataTimestamp, Mode=OneWay}" />
<TextBlock 
    Grid.Row="2" 
    Grid.ColumnSpan="2"
    Margin="4"
    HorizontalAlignment="Stretch"
    HorizontalTextAlignment="Right"
    Foreground="Red" 
    TextWrapping="WrapWholeWords"
    Text="Connectivity issues: data may not be up to 
          date!"
    Visibility="{x:Bind VM.ShowErrorMsg, 
        Mode=OneWay}"/>
```

1.  `ListView`将使用我们在控件级别定义的一些数据模板。在打开的`UserControl`标签之后添加以下内容：

```cs
<UserControl.Resources>
  <DataTemplate x:Key="HeaderTemplate">
       <Grid HorizontalAlignment="Stretch" 
           Background="{ThemeResource 
               ApplicationPageBackgroundThemeBrush}">
      <TextBlock 
          Margin="0" 
          FontWeight="Bold"
          Style="{StaticResource 
                  SubheaderTextBlockStyle}"
          Text="{Binding Platform}" />
    </Grid>
  </DataTemplate>
  <DataTemplate x:Key="ItemTemplate">
    <Grid Margin="0,10">
      <Grid.ColumnDefinitions>
        <ColumnDefinition Width="100" />
        <ColumnDefinition Width="*" />
      </Grid.ColumnDefinitions>
      <TextBlock 
          Margin="0,10"
          Style="{StaticResource TitleTextBlockStyle}"
          Text="{Binding DisplayedTime}" />
      <TextBlock 
          Grid.Column="1" 
          Margin="0,10"
          Style="{StaticResource TitleTextBlockStyle}"
          Text="{Binding Destination}" />
    </Grid>
  </DataTemplate>
</UserControl.Resources>
```

1.  第四行，也是最后一行，包含一个`ListView`，显示即将到达的到站时间：

```cs
<ListView Grid.Row="3" 
    Grid.ColumnSpan="2"
    ItemTemplate="{StaticResource ItemTemplate}"
    ItemsSource="{x:Bind VM.ArrivalsViewSource}"
    SelectionMode="None">
    <ListView.GroupStyle>
        <GroupStyle HeaderTemplate="{StaticResource 
            HeaderTemplate}" />
    </ListView.GroupStyle>
</ListView>
```

1.  第四行还包含一个`Grid`，其中包含其他信息控件，根据需要显示在`ListView`上或替代`ListView`：

```cs
<Grid Grid.Row="3" Grid.ColumnSpan="2">
    <TextBlock HorizontalAlignment="Stretch"
        VerticalAlignment="Center"
        HorizontalTextAlignment="Center"
        Style="{StaticResource 
                SubheaderTextBlockStyle}"
        Text="Select a station" TextWrapping="NoWrap"
        Visibility="{x:Bind VM.ShowNoStnMsg,
            Mode=OneWay}" />
    <ProgressRing Width="100" Height="100"
        IsActive="True" IsEnabled="True"
        Visibility="{x:Bind VM.IsBusy, Mode=OneWay}"
    />
</Grid>
```

1.  我们在这里添加了相当多的 XAML。看看它的外观的第一步是连接 ViewModel，以便我们可以访问相关属性和命令。将`ArrivalsControlxaml.cs`的内容更改为以下内容：

```cs
public sealed partial class ArrivalsControl : UserControl {
    private ArrivalsViewModel VM to help keep the code concise) in the constructor, and it's this class that contains most of the logic.The code-behind also includes a method to handle the `SelectionChanged` event on the `ComboBox`. This is currently necessary as a workaround for a bug due to the order that `ComboBox` events are raised in. The bug is logged at [`github.com/unoplatform/uno/issues/5792`](https://github.com/unoplatform/uno/issues/5792). Once fixed, it should be possible to bind to a `Command` on the ViewModel to perform the equivalent functionality.
```

1.  将以下`using`声明添加到文件顶部，以便编译器可以找到我们刚刚添加的类型：

```cs
using NetworkAssist.ViewModels;
using UnoBookRail.Common.Network;
```

1.  现在我们准备创建一个包含剩余功能逻辑的 ViewModel。我们将首先创建一个名为`ViewModels`的文件夹。在该文件夹中，创建一个名为`ArrivalsViewModel`的类。

1.  为了避免在遵循 MVVM 模式时编写常见的代码，需要在每个平台头项目中添加对`Microsoft.Toolkit.Mvvm` NuGet 包的引用*：

```cs
Install-Package Microsoft.Toolkit.Mvvm -Version 7.0.2
```

1.  更新`ArrivalsViewModel`类，使其继承自`Microsoft.Toolkit.Mvvm.ComponentModel.ObservableObject`。

1.  `ArrivalsViewModel`将使用来自不同位置的类型，因此我们需要引用以下命名空间：

```cs
using Microsoft.Toolkit.Mvvm.Input;
using System.Collections.ObjectModel;
using System.Threading.Tasks;
using System.Windows.Input;
using UnoBookRail.Common.Network;
using Windows.UI.Xaml.Data;
```

1.  首先，在类中添加以下字段：

```cs
private static DataService _data = DataService.Instance;
private List<Station> _listOfStations;
private ObservableCollection<StationArrivalDetails> 
_arrivals = 
    new ObservableCollection<StationArrivalDetails>();
private Station _selectedStation = null;
private string _dataTimestamp;
private bool _isBusy;
private bool _showErrorMsg;
```

1.  我们的`ViewModel`需要以下属性，因为它们在我们之前定义的 XAML 绑定中被引用。它们将使用我们刚刚添加的后备字段：

```cs
public List<Station> ListOfStations 
{
    get => _listOfStations;
    set => SetProperty(ref _listOfStations, value);
}
public bool ShowErrorMsg 
{
    get => _showErrorMsg;
    set => SetProperty(ref _showErrorMsg, value);
}
public Station SelectedStation 
{
    get => _selectedStation;
    set {
        if (SetProperty(ref _selectedStation, value)) 
        {
            OnPropertyChanged(nameof(ShowNoStnMsg));
        }
    }
}
public ObservableCollection<StationArrivalDetails> Arrivals 
{
    get => _arrivals;
    set => SetProperty(ref _arrivals, value);
}
public string DataTimestamp 
{
    get => _dataTimestamp;
    set => SetProperty(ref _dataTimestamp, value);
}
public bool IsBusy 
{
    get => _isBusy;
    set => SetProperty(ref _isBusy, value);
}
public IEnumerable<object> ArrivalsViewSource => new CollectionViewSource() 
{
    Source = Arrivals,
    IsSourceGrouped = true
}.View;
public bool ShowNoStnMsg => SelectedStation == null;
public ICommand RefreshCommand { get; }
public ICommand SelectionChangedCommand { get; }
```

1.  我们将使用构造函数来初始化车站列表和命令：

```cs
public ArrivalsViewModel() 
{
    ListOfStations = _data.GetAllStations();
    RefreshCommand = new AsyncRelayCommand(async () =>
        { await LoadArrivalsDataAsync(); });
    SelectionChangedCommand = new AsyncRelayCommand(
        async () => { await LoadArrivalsDataAsync(); 
            });
}
```

1.  现在，添加处理检索和显示数据的方法：

```cs
public async Task LoadArrivalsDataAsync(int stationId = 0)
{
  if (stationId < 1) 
  {
    // if no value passed use the previously selected 
    // Id.
    stationId = SelectedStation?.Id ?? 0;
  }
  else 
  { 
    // We've changed station so clear current details
    Arrivals.Clear();
    DataTimestamp = string.Empty;
    ShowErrorMsg = false;
  }
  if (stationId > 0) 
  {
    IsBusy = true;
    try {
      var arr = await 
          _data.GetArrivalsForStationAsync(stationId);
      ShowErrorMsg = false;
      if (arr.ForStationId == stationId) 
      {
        DataTimestamp = 
            $"Updated at {arr.Timestamp:t}";
        Arrivals.Clear();
        if (!string.IsNullOrEmpty(
            arr.DirectionOneName)) 
        {
          var d1details = new StationArrivalDetails
              (arr.DirectionOneName);
          d1details.AddRange(arr.DirectionOneDetails);
          Arrivals.Add(d1details);
        }
        if (!string.IsNullOrEmpty(
            arr.DirectionTwoName)) 
        {
          var d2details = new StationArrivalDetails(
              arr.DirectionTwoName);
          d2details.AddRange(arr.DirectionTwoDetails);
          Arrivals.Add(d2details);
        }
      }
    }
    catch (Exception exc) {
      // Log this or take other appropriate action
      ShowErrorMsg = true;
    }
    finally {
      IsBusy = false;
    }
  }
}
```

1.  您可能已经注意到数据是从单例`DataService`类中检索的。我们将首先创建一个简单版本，稍后再扩展。通常约定将此类放在名为`Services`的目录中，尽管您也可以将其放在`ViewModels`文件夹中：

```cs
using System.Linq;
using System.Threading.Tasks;
using UnoBookRail.Common.Network;
public class DataService 
{
    private static readonly Lazy<DataService> ds =
        new Lazy<DataService>(() => new
            DataService());
    private static readonly Lazy<Stations> stations =
        new Lazy<Stations>(() => new Stations());
    public static DataService Instance => ds.Value;
    private DataService() { }
    public List<Station> GetAllStations() => 
        stations.Value.GetAll().OrderBy(s => 
            s.Name).ToList();
    public async Task<Arrivals> 
        GetArrivalsForStationAsync method may seem overly complex.
```

1.  现在我们有了`DataService`类，可以检索到达详情，但是我们需要做更多工作来显示它们。我们还需要另一个类。这是`StationArrivalDetails`，它允许我们按站台和列车行驶方向对信息进行分组。在`ViewModels`目录中创建这个类：

```cs
using UnoBookRail.Common.Network;
public class StationArrivalDetails : 
List<ArrivalDetail> 
{
    public StationArrivalDetails(string platform) 
    {
        Platform = platform;
    }
    public string Platform { get; set; }
}
```

Uno 中使用分组数据的 CollectionViewSource

在 Uno 平台上显示分组列表比在 UWP 上更复杂。如果您以前在 UWP 应用程序中使用过`CollectionViewSource`，那么您可能已经在 XAML 中定义了它，而不是作为`IEnumerable<object>`。不幸的是，为了 Uno 平台能够正确渲染 Android 和 iOS 上的所有组和标题，我们需要将我们的`CollectionViewSource`定义为`IEnumerable<IEnumerable>`。如果不这样做，我们将在 iOS 上看到缺少组标题，而在 Android 上只能看到第一组的内容。

现在我们有一个可用的应用程序，但在接下来的两个部分中，我们将进行两项改进。在那里，我们将改善应用程序的外观并使用一些本机控件，但在此之前，我们将切换到使用来自远程源的“实时”数据，而不是应用程序自带的数据。

# 检索远程数据

很少有应用程序仅使用其自带的数据。**网络辅助**提供的价值是基于提供实时信息。知道火车实际到达的时间比知道计划到达时间更有价值。为了收集这些信息，应用程序必须连接到远程实时数据源。

大多数移动应用程序连接到外部数据源，最常见的方式是通过 HTTP(S)。如果您只开发运行在桌面上的应用程序，您可能可以假设始终有可用的连接。对于移动应用程序，必须考虑设备为**偶尔连接**。

由于不可能假设应用程序始终可用连接或连接速度很快，因此在设计应用程序时必须考虑这一点。这些问题适用于所有移动应用程序，并不是 Uno 平台开发中的独特问题。正确处理偶尔的连接性和数据可用性的方式因应用程序而异。这个问题太大，我们无法在这里完全覆盖，但重要的是提出来。至少，考虑偶尔的连接性意味着需要考虑重试失败的连接请求和管理数据。我们之前在`LoadArrivalsDataAsync`方法中编写的代码已经以一种粗糙的缓存形式，通过在刷新数据时不丢弃当前信息，直到成功请求并有新数据可用于显示。虽然应用程序中显示的信息可能会很快过时，但相对于不显示任何内容，显示应用程序承认为几分钟前的内容更为合适。

在另一个应用程序中，将数据保存在文件或数据库中可能更合适，以便在远程数据不可用时检索和显示。*第五章*，*使您的应用程序准备好面对现实*，展示了如何使用 SQLite 数据库来实现这一点。

我们将很快看到应用程序如何处理连接到远程数据的失败，但首先，我们将看看如何连接到远程数据。

## 连接到远程数据源

本书的 GitHub 存储库位于[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform)，其中包括一个**WebAPI**项目，该项目将为应用程序返回火车到站数据。

您可以选择运行代码并通过本地机器访问，或者您可以连接到[`unobookrail.azurewebsites.net/`](https://unobookrail.azurewebsites.net/)上提供的版本。如果连接到托管版本，请注意它基于服务器的本地时间，并且这可能与您所在的地方不同。如果服务器不断表示下一班火车还有很长时间，因为服务器所在地的凌晨，如果您自己运行项目，您将看到更多不同的数据：

1.  我们将使用`System.Net.Http.HttpClient`连接到服务器。为了能够做到这一点，我们必须在*Android 和 iOS*项目中添加对`System.Net.Http`的包引用：

```cs
Install-Package System.Net.Http -Version 4.3.4
```

1.  由于 API 返回的数据是 JSON 格式，因此我们还将在*所有平台项目*中添加对`Newtonsoft.Json`库的引用，以便我们可以对响应进行反序列化：

```cs
Install-Package Newtonsoft.Json -Version 12.0.3
```

1.  我们现在准备检索远程数据。所有更改都将在`DataService.cs`文件中进行。首先添加一个`HttpClient`的实例。我们将使用这个实例进行所有请求：

```cs
using System.Net.Http;
private static readonly HttpClient _http = new HttpClient();
```

1.  要连接到服务器，我们需要指定它的位置。由于我们最终将进行多个请求，因此在一个地方定义服务器域是明智的。我们将通过`__ANDROID__`常量来实现这一点，该常量可用于`#if`预处理指令。有关更多信息，请参见*第二章**，编写您的第一个 Uno 平台应用程序*。

如果您从 Android 模拟器连接到本地托管的 WebAPI 实例，则需要使用 IP 地址`10.0.2.2`进行连接。这是模拟器用来指代主机机器的特殊 IP 地址。您可以使用条件编译来指定这一点，就像前面的代码片段中所示。如果您连接到外部服务器，您可以直接设置地址，不需要任何条件代码。

1.  现在我们可以更新`GetArrivalsForStationAsync`方法以获取实时数据。*用以下内容替换*当前的实现：

```cs
using Newtonsoft.Json;
public async Task<Arrivals> GetArrivalsForStationAsync(int stationId) 
{
  var url = $"{WebApiDomain}/stations/?stationid=
      {stationId}";
  var rawJson = await _http.GetStringAsync(url);
  return JsonConvert.DeserializeObject<Arrivals>
      (rawJson);
}
```

如果现在运行应用程序，数据将来自远程位置。您可能会注意到数据检索不再是瞬间完成的，等待时会显示一个忙指示器。我们在应用程序的原始版本中添加了显示进度指示器的代码，但直到现在才看到它显示出来。这突显了在处理需要时间检索的数据时可能出现的另一个潜在问题。*在发生某事时让用户了解情况至关重要*。我们在这里使用`ProgressRing`来指示发生了某事。如果没有这个，用户可能会想知道是否有任何事情发生，并变得沮丧或反复按刷新按钮。

到目前为止，我们已经从远程源检索到数据，并在此过程中让用户了解情况，但是当事情出错时，我们需要做更多。所以，我们接下来会看看这一点。

## 使用 Polly 处理异常并重试请求

处理异常并重试失败的请求的需求几乎适用于所有应用程序。幸运的是，有许多解决方案可以帮助我们处理一些复杂性。**Polly** ([`github.com/App-vNext/Polly`](https://github.com/App-vNext/Polly))是一个流行的开源库，用于处理瞬态错误，我们将在我们的应用程序中使用。让我们来看一下：

1.  我们将首先向*所有平台项目*添加对`Polly.Extensions.Http`包的引用：

```cs
Install-Package Polly.Extensions.Http -Version 3.0.0
```

这扩展了标准的 Polly 功能，并简化了处理与 HTTP 相关的故障。

1.  我们现在将再次更新`GetArrivalsForStationAsync`方法，使其使用 Polly 的`HandleTransientHttpError`。这告诉 Polly 如果 HTTP 响应是服务器错误（HTTP 5xx）或超时错误（HTTP 408），则重试请求。

对`WaitAndRetryAsync`的调用告诉 Polly 最多重试三次。我们还使用`policy.ExecuteAsync`指定每个请求之间的延迟，并将其传递给我们希望应用策略的操作。

1.  如果请求因我们策略未覆盖的原因而失败，我们之前创建的代码会导致屏幕顶部显示一条消息，如下面的屏幕截图所示，指示问题所在。其他应用可能需要以不同方式记录或报告此类问题，但通常不适合什么都不做：

![图 4.3 - 应用程序显示连接问题的消息](img/Figure_4.03_B17132.jpg)

图 4.3 - 应用程序显示连接问题的消息

现在我们有了一个可以可靠地从远程源提供有用数据的应用程序。我们想要做的最后一件事是改善它在不同平台上的外观。

# 使您的应用程序看起来像属于每个平台

到目前为止，应用程序中的所有内容都使用了 Uno Platform 提供的默认样式。因为 Uno Platform 基于 UWP 和 WinUI，我们的应用程序的样式是基于 Fluent Design 系统的，因为这是 Windows 的默认样式。如果我们希望我们的应用程序看起来这样，这是可以的，但是如果我们希望我们的应用程序使用 Android 或 iOS 的默认样式怎么办？幸运的是，Uno Platform 为我们提供了解决方案。它为我们提供了**Material**和**Cupertino**样式的库，我们可以应用到我们的应用程序中。虽然这些库分别是为 Android 和 iOS 设备本地化的，但它们可以在任何地方使用。

现在，我们将使用这些库提供的资源，将 Material Design 样式应用于我们应用程序的 Android 版本，将 Cupertino 样式应用于 iOS 版本。

## 将 Material 样式应用于应用程序的 Android 版本

让我们开始吧：

1.  我们将首先向*Android 项目*添加对`Uno.Material`软件包的引用。请注意，这是一个预发布软件包，因此如果您通过 UI 搜索，请启用此软件包：

```cs
Install-Package Uno.Material -Version 1.0.0-dev.790
```

1.  虽然`Uno.Material`库知道如何为控件设置样式，但它并不包含所有资产和引用以使用它们。为此，*在 Android 项目*中*添加*`Xamarin.AndroidX.Lifecycle.LiveData`和`Xamarin.AndroidX.AppCompat.AppCompatResources`软件包：

```cs
Install-Package Xamarin.AndroidX.AppCompat.AppCompatResources -Version 1.2.0.5
Install-Package Xamarin.AndroidX.Lifecycle.LiveData -Version 2.3.1
```

1.  要在 Android 库中使用样式，我们必须通过在`App.xaml`中引用它们来将它们添加到应用程序中可用的样式中：

```cs
<Application
    x:Class="NetworkAssist.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/
           xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/
             xaml"
    xmlns:android="http://uno.ui/android"
    xmlns:local="using:NetworkAssist"
    xmlns:mc="http://schemas.openxmlformats.org/
             markup-compatibility/2006"
    mc:Ignorable="android">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <XamlControlsResources xmlns=
                "using:Microsoft.UI.Xaml.Controls" />
                <android:MaterialColors xmlns=
                    "using:Uno.Material" />
                <android:MaterialResources xmlns=
                     "using:Uno.Material" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

1.  一些控件将自动应用 Material 样式，而其他控件将需要直接应用样式。为了展示这一点，我们将为刷新`Button`应用特定样式。

在`ArrivalsControl.xaml`中，*在文件顶部添加 Android 命名空间别名*。我们只在 Android 上运行时才会使用这个。然后，将样式应用于`Button`元素：

```cs
Button control looks on the arrivals control, but it hasn't improved the buttons in CommandBar at the bottom of the shell page. Let's address this now.
```

1.  与使用 Windows `CommandBar`不同，Material Design 系统具有一个单独的控件，更适合在屏幕底部显示与导航相关的按钮。这称为`BottomNavigationBar`。我们将首先将其添加到`MainPage.xaml`中，并将现有的`CommandBar`包装在一个`Grid`中，该`Grid`仅在 Windows 上显示：

```cs
Click events as before. It's only the control that's displaying them that we're changing.NoteAfter adding the `Xamarin.AndroidX` packages, you may get a compilation error related to a file called `abc_vector_test.xml`. This error is due to compatibility inconsistencies between different preview versions of the packages and Visual Studio. This error can be addressed by opening the **Properties** section of the **Android** project, selecting **Android Options**, and unchecking the **Use incremental Android packaging system (aap2)** option. This may lead to a separate build warning and slightly slower builds, but the code will now compile. Hopefully, future updates that are made to these packages will help us avoid this issue.
```

1.  如果现在运行应用程序，您会看到按钮和导航栏是紫色的。这是`Uno.Material`库中定义的颜色方案的一部分。您可以通过包含提供预定义 Material 颜色的不同值的`ResourceDictionary`来使用自己的颜色方案。然后，当您添加*步骤 2*中显示的资源时，您可以引用它。有关如何执行此操作的指南，请参阅[`platform.uno/docs/articles/features/uno-material.html#getting-started`](https://platform.uno/docs/articles/features/uno-material.html#getting-started)。

现在我们已经改善了 Android 上应用程序的外观，让我们为 iOS 做同样的事情。

## 将 Cupertino 样式应用于应用程序的 iOS 版本

让我们开始吧：

1.  单独的软件包包含 Cupertino 样式，因此我们必须在 iOS 项目中添加对`Uno.Cupertino`的引用：

```cs
Install-Package Uno.Cupertino -Version 1.0.0-dev.790
```

与上一节中的 Material 软件包一样，我们需要通过添加以下内容在`App.xaml`中加载此软件包的资源：

```cs
xmlns:ios="http://uno.ui/ios"
mc:Ignorable="android ios">
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <XamlControlsResources xmlns=
                "using:Microsoft.UI.Xaml.Controls" />
            <android:MaterialColors xmlns=
                "using:Uno.Material" />
            <android:MaterialResources xmlns=
                "using:Uno.Material" />
            <ios:CupertinoColors xmlns=
                "using:Uno.Cupertino" />
            <ios:CupertinoResources xmlns=
               "using:Uno.Cupertino" />
       </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

1.  此软件包尚未包含原生选项卡栏控件（`UITabBar`），但我们可以轻松创建与苹果的人机界面指南相匹配的内容。

*在`MainPage.xaml`中*添加*以下内容，添加到`win:Grid`元素之后：

```cs
Click events that we did previously, but we're using a new converter for ForegroundColor of the Buttons. For this, you'll need to *create a folder* called Converters and *create a file* called CupertinoButtonColorConverter.cs containing the following code:

```

使用 Windows.UI.Xaml.Data;

public class CupertinoButtonColorConverter：IValueConverter

{

public object Convert(object value, Type targetType，

对象参数，字符串语言）

{

如果（value？.ToString() == parameter？.ToString()）

{

return App.Current.Resources[

"CupertinoBlueBrush"];

}

否则

{

return App.Current.Resources[

"CupertinoSecondaryGrayBrush"];

}

}

public object ConvertBack(object value, Type

targetType，对象参数，字符串语言）

=> 抛出未实现的异常();

}

```cs

```

1.  与 Android 项目一样，Cupertino 样式不会自动应用于应用程序中的按钮。但是，我们可以创建一个*隐式样式*，将其应用于整个应用程序中的所有`Button`元素，而不是直接将样式应用于每个`Button`元素。要做到这一点，*修改* `App.xaml`以添加样式，如下所示：

```cs
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <XamlControlsResources xmlns=
                "using:Microsoft.UI.Xaml.Controls" />
            <android:MaterialColors xmlns=
                "using:Uno.Material" />
            <android:MaterialResources xmlns=
                "using:Uno.Material" />
            <ios:CupertinoColors xmlns=
                "using:Uno.Cupertino"  />
            <ios:CupertinoResources xmlns=
                "using:Uno.Cupertino" />
        </ResourceDictionary.MergedDictionaries>
        <ios:Style TargetType="Button"
BasedOn="{StaticResource 
                CupertinoButtonStyle}" />
    </ResourceDictionary>
</Application.Resources>
```

隐式样式可以用于任何平台，因此，如果您愿意，您可以在应用程序的 Android 版本中执行类似的操作。

现在我们有一个看起来属于每个平台的应用程序，并且它可以显示我们从外部服务器检索的内容。现在，让我们看看如何使用设备的功能来创建数据并将其发送到远程源。

# 访问设备功能

我们将向应用程序添加的最后一个功能与我们迄今为止所做的不同。到目前为止，我们已经研究了消耗数据，但现在我们将研究如何创建数据。

公司对应用程序的要求是，它提供了一种让员工在发生事故时捕获信息的方式。所谓的“事故”可以是企业可能需要记录或了解的任何事情。它可能是一些小事，比如顾客在公司财产上绊倒，也可能是一起重大事故。所有这些事件都有一个共同点：捕获详细信息比依靠人们以后记住细节更有益。目标是让员工尽可能快速、简单地捕获图像或一些文本，以增加捕获的信息量。软件将使用事件发生的时间和位置以及记录者的信息来增强捕获的信息。这些信息将被汇总并在一个单独的后端系统中进一步记录。

让我们创建一种简单的方式来满足这些要求，以演示 Uno 平台如何提供一种在不同平台上使用 UWP API 的方式：

1.  使用相机并获取设备位置，我们需要指示应用程序将需要必要的权限来执行此操作。我们在每个平台上指定权限的方式略有不同。

在 Android 上，打开项目的`info.plist`并使用`Package.appxmanfiest`打开它，转到`CameraCaptureUI`。

1.  我们可以通过在`QuickReportControl.xaml`的`Grid`中添加以下内容来创建 UI：

```cs
Button elements on Android. This is to highlight the importance of each button.
```

1.  在`QuickReportControl.xaml.cs`中，让我们添加处理用户单击按钮添加照片时发生的情况的代码：

```cs
using Windows.Media.Capture;
using Windows.UI.Xaml.Media.Imaging;
Windows.Storage.StorageFile capturedPhoto;
private async void CaptureImageClicked(object sender, RoutedEventArgs e) 
{
    try 
    {
         var captureUI = new CameraCaptureUI and call CaptureFileAsync to ask it to capture a photograph. When that returns successfully (it isn't canceled by the user), we display the image on the screen and store it in a field to send it to the server later.
```

1.  现在我们将创建一个方法来封装检索设备位置的逻辑：

```cs
using Windows.Devices.Geolocation;
using System.Threading.Tasks;
private async Task<string> GetLocationAsync() 
{
    try 
    {
        var accessStatus = await 
            Geolocator.RequestAccessAsync();
        switch (accessStatus) 
        {
            case GeolocationAccessStatus.Allowed:
                 var geolocator = new Geolocator();
                 var pos = await 
                     geolocator.GetGeopositionAsync();
                 return $"{pos.Coordinate.Latitude},
                    {pos.Coordinate.Longitude},
                        {pos.Coordinate.Altitude}";
            case GeolocationAccessStatus.Denied:
                return "Location access denied";
            case GeolocationAccessStatus.Unspecified:
                return "Location Error";
        }
    }
    catch (Exception ex) 
    {
        // Log the exception as appropriate
    }
    return string.Empty;
}
```

1.  最后一步是为“成功”提交有效数据时添加事件处理程序。应用程序会检查这一点，并向用户显示适当的消息。

注意

您可能认为允许用户与应用程序交谈并记录他们的声音会更方便。这是一个明智的建议，也是可以很容易在将来添加的内容。我们在这里没有包括它，因为大多数设备都具有内置功能，可以使用语音转文字来输入详细信息。使用设备的现有功能可能比复制已有功能更快捷、更容易。

现在，我们的应用程序已经完成了这最后一部分功能。您可以在下图中看到它的运行效果：

![图 4.4-快速报告屏幕在 iPhone 上运行（左）并显示所选图像，以及 Android 设备（右）显示输入的一些口述文本](img/Figure_4.04_B17132.jpg)

图 4.4-快速报告屏幕在 iPhone 上运行（左）并显示所选图像，以及 Android 设备（右）显示输入的一些口述文本

# 总结

在本章中，我们构建了一个可以在 iOS 和 Android 设备上运行的应用程序。这使您了解了创建“移动”应用程序的含义，处理远程数据，将本机平台主题应用于应用程序，并使用本机设备功能。

在下一章中，我们将构建另一个移动应用程序。这将与迄今为止制作的应用程序不同，因为它旨在供客户使用，而不是公司员工使用。除其他事项外，我们将利用这个应用程序来研究可访问性、本地化和使用 SQLite 数据库。
