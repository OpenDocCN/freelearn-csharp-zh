# 第五章：*第五章*：使您的应用程序准备好面向现实世界

在上一章中，我们介绍了使用 Uno Platform 编写面向 UnoBookRail 员工的第一个移动应用程序。在本章中，我们也将编写一个移动应用程序；但是，我们将专注于使其准备好供客户使用。在本章中，您将编写一个在设备上持久保存用户偏好和更大数据集的应用程序。此外，您还将学习如何通过自定义应用程序图标使您的应用程序对用户更具吸引力，以及如何编写可以供使用辅助技术的人使用的应用程序。

为了做到这一点，我们将在本章中涵盖以下主题：

+   介绍应用程序

+   使用`ApplicationData` API 和 SQLite 在本地持久化数据

+   使您的应用程序准备好供客户使用

+   本地化您的应用程序

+   使用自定义应用程序图标和启动画面

+   使您的应用程序适用于所有用户

在本章结束时，您将创建一个在 iOS 和 Android 上运行的移动应用程序，该应用程序已准备好供客户使用，并且已进行本地化和可访问。

# 技术要求

本章假设您已经设置好了开发环境，包括安装了项目模板，就像在*第一章*中介绍的那样，*介绍 Uno Platform*。本章的源代码位于[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/Chapter05`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/Chapter05)。

本章的代码使用了来自[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/SharedLibrary`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/SharedLibrary)的库。

查看以下视频以查看代码的实际操作：[`bit.ly/3AywuqQ`](https://bit.ly/3AywuqQ)

# 介绍应用程序

在本章中，我们将构建 UnoBookRail DigitalTicket 应用程序，这是一个面向想要使用 UnoBookRail 从 A 到 B 的 UnoBookRail 客户的应用程序。虽然这个应用程序的真实版本可能有很多功能，但在本章中，我们只会开发以下功能：

+   预订 UnoBookRail 网络两个站点之间的行程车票

+   查看所有预订的车票以及车票的 QR 码

+   本地化应用程序，并允许用户选择用于应用程序的语言

作为其中的一部分，我们还将确保我们的应用程序是可访问的，并允许不同能力水平的更多人使用我们的应用程序。现在让我们开始创建应用程序并添加第一部分内容。

## 创建应用程序

首先，我们需要为我们的应用程序设置解决方案：

1.  首先，使用**Multi-Platform App (Uno Platform)** 模板创建一个新的应用程序。

1.  将项目命名为`DigitalTicket`。当然，您也可以使用不同的名称；但是，在本章中，我们将假设该应用程序被命名为 DigitalTicket，并使用相应的命名空间。

1.  删除除**Android**、**iOS**和**UWP**之外的所有平台头。请注意，即使在网络上提供此功能可能会有好处，我们也会删除 WASM 头。虽然 WASM 在移动设备上运行得相当不错，但并不理想，为了简单起见，我们将继续不使用应用程序的 WASM 版本。

1.  将 UnoBookRail 共享库添加到解决方案中，因为我们稍后将需要其功能。为此，请右键单击解决方案文件，选择`UnoBookRail.Common.csproj`文件，然后单击**打开**。

1.  在每个头项目中引用共享库项目。为此，请右键单击头项目，选择**添加** | **引用…** | **项目**，选中**UnoBookRail.Common**，然后单击**确定**。由于我们需要在每个头中引用该库，请为每个头重复此过程，即 Android、iOS 和 UWP。

由于我们的应用程序还将遵循`Microsoft.Toolkit.MVVM`包，您还需要添加对其的引用：

1.  在解决方案视图中右键单击解决方案节点，然后选择**管理解决方案的 NuGet 包…**。

1.  搜索`Microsoft.Toolkit.MVVM`并选择**NuGet**包。

1.  在项目列表中选择 Android、iOS 和 UWP 头部，然后点击**安装**。

与上一章类似，我们还需要修改我们的应用程序以留出相机刘海的空间，以避免应用程序的内容被遮挡：

1.  为此，在`MainPage.xaml`文件中添加以下命名空间：`xmlns:toolkit="using:Uno.UI.Toolkit"`。

1.  之后，在我们的`MainPage.xaml`文件内的网格中添加`toolkit:VisibleBoundsPadding.PaddingMask="All"`。

## 创建主导航和预订流程

由于我们的应用程序将包含不同的功能，我们将把应用程序的功能拆分成不同的页面，我们将导航到这些页面。在`MainPage`内，我们将有我们的导航和相关代码：

1.  首先，通过右键单击`Views`创建一个 views 文件夹。

1.  现在，在`JourneyBookingPage.xaml`、`OwnedTicketsPage.xaml`和`SettingsPage.xaml`内添加以下三个页面。

1.  由于我们以后会需要它，创建一个`Utils`文件夹，并添加一个`LocalizedResources`类，其中包含以下代码：

```cs
public static class LocalizedResources
{
    public static string GetString(string key) {
        return key;
    }
}
```

目前，这个类只会返回字符串，这样我们就可以引用该类，而不必以后更新代码。不过，在本章的后面，我们将更新实现以返回提供的键的本地化版本。

1.  之后，在共享项目中创建一个`ViewModels`文件夹，并创建一个`NavigationViewModel`类。

1.  将以下内容添加到您的`NavigationViewModel`类：

```cs
using DigitalTicket.Views;
using Microsoft.Toolkit.Mvvm.ComponentModel;
using Microsoft.UI.Xaml.Controls;
using System;
namespace DigitalTicket.ViewModels
{
    public class NavigationViewModel : 
        ObservableObject
    {
        private Type pageType;
        public Type PageType
        {
            get
            {
                return pageType;
            }
            set
            {
                SetProperty(ref pageType, value);
            }
        }
        public void NavigationView_SelectionChanged(
          NavigationView navigationView, 
            NavigationViewSelectionChangedEventArgs
              args)
        {
            if (args.IsSettingsSelected)
            {
                PageType = typeof(SettingsPage);
            }
            else
            {
                switch ((args.SelectedItem as 
                   NavigationViewItem).Tag.ToString())
                {
                    case "JourneyPlanner":
                        PageType = 
                          typeof(JourneyBookingPage);
                        break;
                    case "OwnedTickets":
                        PageType = 
                          typeof(OwnedTicketsPage);
                        break;
                }
            }
        }
    }
}
```

此代码将公开`MainPage`应该导航到的页面类型，并提供选择更改侦听器以在应用程序导航更改时更新。为了确定正确的页面类型，我们将使用所选项的`Tag`属性。

1.  现在，用以下内容替换`MainPage`的内容：

```cs
    ...
    xmlns:muxc="using:Microsoft.UI.Xaml.Controls">
    <Grid toolkit:VisibleBoundsPadding.PaddingMask=
        "All">
        <muxc:NavigationView x:Name="AppNavigation"
            PaneDisplayMode="LeftMinimal"             
            IsBackButtonVisible="Collapsed" 
            Background="{ThemeResource 
                ApplicationPageBackgroundThemeBrush}"
            SelectionChanged="{x:Bind 
                navigationVM.NavigationView_
                     SelectionChanged, Mode=OneTime}">
            <muxc:NavigationView.MenuItems>
                <muxc:NavigationViewItem 
                    x:Name="JourneyBookingItem" 
                    Content="Journey Booking"
                    Tag="JourneyPlanner"/>
                <muxc:NavigationViewItem 
                    Content="Owned tickets"
                    Tag="OwnedTickets"/>
                <muxc:NavigationViewItem Content="All 
                    day tickets - soon" 
                    Tag="AllDayTickets" 
                    IsEnabled="False"/>
                <muxc:NavigationViewItem 
                    Content="Network plan - soon" 
                    IsEnabled="False"/>
                <muxc:NavigationViewItem 
                    Content="Line overview - soon"
                    IsEnabled="False"/>
            </muxc:NavigationView.MenuItems>
            <Frame x:Name="ContentFrame" 
                Padding="0,40,0,0"/>
             </muxc:NavigationView>
    </Grid>
```

这是我们应用程序的主要导航。我们使用`NavigationView`控件来实现这一点，它允许我们轻松地拥有一个可以使用汉堡按钮打开的侧边窗格。在其中，我们提供不同的导航选项，并将`Tag`属性设置为`NavigationViewModel`使用。由于在本章中我们只允许预订行程和拥有的票证列表，我们暂时禁用了其他选项。

1.  用以下内容替换您的`MainPage`类：

```cs
using DigitalTicket.ViewModels;
using DigitalTicket.Views;
using System;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Navigation;
namespace DigitalTicket
{
    public sealed partial class MainPage : Page
    {
        public NavigationViewModel navigationVM = new 
            NavigationViewModel();
        public MainPage()
        {
            InitializeComponent();
            if (navigationVM.PageType is null)
            {
                AppNavigation.SelectedItem = 
                    JourneyBookingItem;
                navigationVM.PageType = 
                    typeof(JourneyBookingPage);
                navigationVM.PageTypeChanged += 
                    NavigationVM_PageTypeChanged;
            }
        }
        protected override void OnNavigatedTo(
            NavigationEventArgs e)
        {
            base.OnNavigatedTo(e);
            if (e.Parameter is Type navigateToType)
            {
                if (navigateToType == 
                    typeof(SettingsPage))
                {
                    AppNavigation.SelectedItem = 
                        AppNavigation.SettingsItem;
                }
                navigationVM.PageType = 
                    navigateToType;
                ContentFrame.Navigate(navigateToType);
            }
        }
        private void NavigationVM_PageTypeChanged(
           object sender, EventArgs e)
        {
            ContentFrame.Navigate(
                navigationVM.PageType);
        }
    }
}
```

通过这样，`MainPage`在创建时将创建必要的视图模型，并根据此更新显示的内容。`MainPage`还监听`OnNavigatedTo`事件，以根据传递给它的参数更新显示的项目。最后，我们还监听`NavigationViewModels`属性更改事件。

请注意，我们重写了`OnNavigatedTo`函数，以便允许导航到`MainPage`，以及在`MainPage`内导航到特定页面。虽然我们现在不需要这个，但以后我们会用到。让我们继续填充行程预订页面的内容：

1.  在`ViewModels`文件夹内创建`JourneyBookingOption`类。

1.  将以下代码添加到`JourneyBookingOption`类：

```cs
using DigitalTicket.Utils;
using UnoBookRail.Common.Tickets;
namespace DigitalTicket.ViewModels
{
    public class JourneyBookingOption
    {
        public readonly string Title;
        public readonly string Price;
        public readonly PricingOption Option;
        public JourneyBookingOption(PricingOption 
            option)
        {
            Title = LocalizedResources.GetString(
              option.OptionType.ToString() + "Label");
            Price = option.Price;
            Option = option;
        }
    }
}
```

由于这是一个用于显示选项的数据对象，它只包含属性。由于标题将显示在应用程序内并且需要本地化，我们使用`LocalizedResources.GetString`函数来确定正确的值。

1.  现在在`ViewModels`文件夹中创建`JourneyBookingViewModel`类，并添加 GitHub 上看到的代码（[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/blob/main/Chapter05/DigitalTicket.Shared/ViewModels/JourneyBookingViewModel.cs`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/blob/main/Chapter05/DigitalTicket.Shared/ViewModels/JourneyBookingViewModel.cs)）。请注意，有几行被注释掉，那是因为我们稍后会需要这些行；但是，现在我们还没有添加必要的代码。

1.  更新`JourneyBookingPage.xaml.cs`和`JourneyBookingPage.xaml`，使它们与 GitHub 上看到的一样。

1.  将以下条目复制到`Strings/en`文件夹中的`Strings.resw`文件中。请注意，您不必逐字复制`Comments`列，因为它只是为其他两列提供指导和上下文：

![表 5.1](img/Table_01.jpg)

您可能会注意到，一些控件设置了`x:Uid`属性，这就是为什么需要`Strings.resw`文件中的条目。我们将在*本地化您的应用程序*部分介绍这些工作原理；现在，我们只会添加代码和相应的条目到我们的资源文件中。现在，如果您启动应用程序，您应该会看到*图 5.1*中显示的内容：

![图 5.1 - Android 上的旅程预订页面](img/Author_Figure_5.01_B17132.jpg)

图 5.1 - Android 上的旅程预订页面

现在您的用户可以配置他们的旅程，选择车票并预订，尽管车票名称不够理想。我们将在*本地化您的应用程序*部分中解决这个问题。为简单起见，我们将不处理实际付款，并假设付款信息与用户帐户关联。

在本节中，我们添加了应用程序的初始代码和导航。我们还添加了旅程预订页面，尽管目前实际上还没有预订车票，但我们稍后会更改。在下一节中，我们将介绍如何使用两种不同的方法在用户设备上本地持久化数据，即`ApplicationData` API 和 SQLite。

# 使用 ApplicationData API 和 SQLite 在本地持久化数据

虽然在许多情况下，数据可以从互联网上获取，就像我们在*第四章*中看到的那样，*移动化您的应用程序*，通常需要在用户设备上持久化数据。这可能是需要在没有互联网连接时可用的数据，或者是设备特定的数据，例如设置。我们将首先使用`ApplicationData` API 持久化小块数据。

## 使用 ApplicationData API 存储数据

由于我们将本地化我们的应用程序，我们还希望用户能够选择应用程序的语言。为此，首先在我们的共享项目中创建一个`Models`文件夹，并添加一个`SettingsStore`类。现在，将以下代码添加到`SettingsStore`类中：

```cs
using Windows.Storage;
public static class SettingsStore
{
    private const string AppLanguageKey = 
        "Settings.AppLanguage";
    public static void StoreAppLanguageOption(string 
         appTheme)
    {
        ApplicationData.Current.LocalSettings.Values[
            AppLanguageKey] = appTheme.ToString();
    }
    public static string GetAppLanguageOption()
    {
        if (ApplicationData.Current.LocalSettings.Values.
            Keys.Contains(AppLanguageKey))
        {
            return ApplicationData.Current.LocalSettings.
                Values[AppLanguageKey].ToString();
        }
        return "SystemDefault";
    }
}
```

访问应用程序的默认本地应用程序存储，我们使用`ApplicationData.Current.LocalSettings`对象。`ApplicationData` API 还允许您访问存储数据的不同方式，例如，您可以使用它来访问应用程序的本地文件夹，使用`ApplicationData.Current.LocalFolder`。在我们的情况下，我们将使用`ApplicationData.Current.LocalSettings`来持久化数据。`LocalSettings`对象是一个`ApplicationDataContainer`对象，您可以像使用字典一样使用它。请注意，`LocalSettings`对象仅支持字符串和数字等简单数据类型。现在我们已经添加了一种存储要显示应用程序语言的方法，我们需要让用户更改语言：

1.  首先，在我们的`ViewModels`文件夹中创建一个名为`SettingsViewModel`的新类。您可以在此处找到此类的代码：[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/blob/main/Chapter05/DigitalTicket.Shared/ViewModels/SettingsViewModel.cs`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/blob/main/Chapter05/DigitalTicket.Shared/ViewModels/SettingsViewModel.cs)。

1.  现在，我们更新我们的设置页面，以包括更改应用程序语言的 UI。为此，请将`SettingsPage.xaml`中的`Grid`元素替换为以下内容：

```cs
<StackPanel Padding="10,0,10,10">
    <ComboBox x:Name="LanguagesComboBox"
        Header="Choose the app's language"
        SelectedIndex="{x:Bind 
            settingsVM.SelectedLanguageIndex,
                Mode=TwoWay}"/>
</StackPanel>
```

1.  除此之外，我们还需要更新`SettingsPage.xaml.cs`。请注意，我们将在代码后台设置`ComboBox`的`ItemsSource`，以确保在`ComboBox`创建并准备就绪后设置`ItemsSource`，以便`ComboBox`能够正确更新。为此，请添加以下代码：

```cs
using DigitalTicket.ViewModels;
...
private SettingsViewModel settingsVM = new SettingsViewModel();
public SettingsPage()
{
    InitializeComponent();
    LanguagesComboBox.ItemsSource = 
        settingsVM.LanguageOptions;
}
```

1.  最后，为了确保在应用程序启动时将尊重所选的语言，将以下代码添加到`App.xaml.cs`的`OnLaunched`函数中，并为`DigitalTicket.Models`和`DigitalTicket.ViewModels`添加导入：

```cs
ApplicationLanguages.PrimaryLanguageOverride = 
SettingsViewModel.GetPrimaryLanguageOverrideFromLanguage(
SettingsStore.GetAppLanguageOption());
```

现在我们已经添加了语言选项，让我们试一下。如果您现在启动应用程序并使用左侧的导航转到设置页面，您应该会看到类似于*图 5.2*左侧的内容。现在，如果您选择`SettingsViewModel`重新加载`MainPage`和所有其他页面，并设置`ApplicationLanguages.PrimaryLanguageOverride`属性，我们将在*本地化您的应用程序*部分更多地讨论此属性，并且还将更新应用程序，以便所有当前可见的文本也根据所选择的语言进行更新：

![图 5.2–左：设置页面；右：切换语言为德语后的导航](img/Figure_5.02_B17132.jpg)

图 5.2–左：设置页面；右：切换语言为德语后的导航

## 使用 SQLite 存储数据

虽然`ApplicationData` API 适用于存储小数据块，但是如果要持久化更大的数据集，则`ApplicationData` API 并不理想，因为使用`ApplicationData.Current.LocalSettings`对象存储的条目存在空间限制。换句话说，对象键的长度只能为 255 个字符，UWP 上的条目大小只能为 8 千字节。当然，这并不意味着您不能在应用程序中存储更大或更复杂的数据集。这就是`sqlite-net-pcl`库的作用，因为该库适用于我们应用程序支持的每个平台。`sqlite-net-pcl`包括 SQLite 的跨平台实现，并允许我们轻松地将对象序列化为 SQLite 数据库。

让我们首先向我们的应用程序添加对`sqlite-net-pcl`的引用。为此，请在解决方案视图中右键单击解决方案，单击`sqlite-net-pcl`。由于在编写本书时，最新的稳定版本是**1.7.335**，请选择该版本并在项目列表中选择 Android、iOS 和 UWP 头。然后，单击**安装**。现在，我们需要添加代码来创建、加载和写入 SQLite 数据库：

1.  首先，我们需要添加一个类，我们希望使用 SQLite 持久化其对象。为此，在`ViewModels`文件夹中添加一个名为`OwnedTicket`的新类。您可以在 GitHub 上找到此类的源代码：[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/blob/main/Chapter05/DigitalTicket.Shared/ViewModels/OwnedTicket.cs`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/blob/main/Chapter05/DigitalTicket.Shared/ViewModels/OwnedTicket.cs)。

有两件重要的事情需要知道：

由于每个 SQLite 表都需要一个主键，我们添加了带有 PrimaryKey 和 AutoIncrement 属性的`DBId`属性。使用这些属性，我们让`sqlite-net-pcl`为我们管理主键，而无需自己处理。

将对象传递给`sqlite-net-pcl`以将它们持久化到 SQLite 数据库中，只有属性将被持久化。由于我们不想持久化`ShowQRCodeCommand`（实际上也不能），这只是一个字段，而不是属性。

1.  现在在`Models`文件夹中创建`OwnedTicketsRepository`类，并向其中添加以下代码：

```cs
using DigitalTicket.ViewModel;
using SQLite;
using System;
using System.IO;
using System.Threading.Tasks;
using Windows.Storage;
namespace DigitalTicket.Models
{
    public class OwnedTicketsRepository
    {
        const string DBFileName = "ownedTickets.db";
        private static SQLiteAsyncConnection database;
        public async static Task InitializeDatabase()
        {
            if(database != null)
            {
                return;
            }
            await ApplicationData.Current.LocalFolder.
                CreateFileAsync(DBFileName, 
                CreationCollisionOption.OpenIfExists);
            string dbPath = Path.Combine(
                ApplicationData.Current.LocalFolder
                    .Path, DBFileName);
            database = 
                new SQLiteAsyncConnection(dbPath);
            database.CreateTableAsync<
                OwnedTicket>().Wait();
        }
        public static Task<int> SaveTicketAsync(
            OwnedTicket ticket)
        {
            if (ticket.DBId != 0)
            {
                // Update an existing ticket.
                return database.UpdateAsync(ticket);
            }
            else
            {
                // Save a new ticket.
                return database.InsertAsync(ticket);
            }
        }
    }
}
```

`InitializeDatabase`函数处理创建我们的 SQLite 数据库文件和创建表（如果不存在），但也会在文件已经存在时加载现有数据库。在`SaveTicketsAsync`函数中，我们更新并保存传递的车票到数据库，或者如果数据库中已存在该车票，则更新该车票。

1.  更新`App.xaml.cs`以在`OnLaunched`函数的开头包含以下代码，并将`OnLaunched`函数更改为异步：

```cs
await OwnedTicketsRepository.InitializeDatabase();
```

这将在应用程序启动时初始化 SQLite 连接，因为按需创建连接并不理想，特别是在加载所拥有的车票页面时。

1.  现在更新`JourneyBookingViewModel`以将车票保存到`OwnedTicketsRepository`。为此，请删除当前创建`BookJourney`并取消注释文件顶部的`using`语句以及`JourneyBookingViewModel`构造函数中的代码。

现在让我们谈谈我们刚刚做的步骤。首先，我们创建了我们的`OwnedTicket`对象，我们将在下一节中将其写入 SQLite 并从 SQLite 中加载。

然后我们添加了`OwnedTicketsRepository`，我们用它来与我们的 SQLite 数据库交互。在可以向 SQLite 数据库发出任何请求之前，我们首先需要初始化它，为此我们需要一个文件来将 SQLite 数据库写入其中。使用以下代码，我们确保我们要将数据库写入的文件存在：

```cs
await ApplicationData.Current.LocalFolder.CreateFileAsync(DBFileName, CreationCollisionOption.OpenIfExists);
```

之后，我们为我们的数据库创建了一个`SQLiteAsyncConnection`对象。`SQLiteAsyncConnection`对象将处理与 SQLite 的所有通信，包括创建表和保存和加载数据。由于我们还需要一个表来写入我们的数据，我们使用`SQLiteAsyncConnection`为我们的`OwnedTickets`对象创建一个表，如果该表在我们的 SQLite 数据库中不存在。为了确保在对数据库进行任何请求之前执行这些步骤，我们在我们的应用程序构造函数中调用`OwnedTicketsRepository.InitializeDatabase()`。

最后一步是更新我们的`JourneyBookingViewModel`类，以便将数据持久化到 SQLite 数据库中。虽然我们只向数据库中添加新项目，但我们仍然需要注意是否正在更新现有条目或添加新条目，这就是为什么`SavedTicketAsync`函数确保我们只在没有 ID 存在时才创建项目。

## 从 SQLite 加载数据

现在我们已经介绍了如何持久化数据，当然，我们也需要加载数据；否则，我们就不需要首先持久化数据。让我们通过添加用户预订的所有车票的概述来改变这一点。由于 UnoBookRail 的客户需要在登上火车或检查车票时出示他们的车票，我们还希望能够为每张车票显示 QR 码。由于我们将使用`ZXing.Net.Mobile`来实现这一点，请立即将该**NuGet**包添加到您的解决方案中，即 Android、iOS 和 UWP 头。请注意，在撰写本文时，版本**2.4.1**是最新的稳定版本，我们将在本章中使用该版本。

在我们想要显示所有车票之前，我们首先需要从 SQLite 数据库中加载它们。为此，向我们的`OwnedTicketsRepository`类添加以下方法：

```cs
using System.Collections.Generic;
...
static Task<List<OwnedTicket>> LoadTicketsAsync()
{
    //Get all tickets.
    return database.Table<OwnedTicket>().ToListAsync();
}
```

由于`sqlite-net-pcl`，这就是我们需要做的一切。该库为我们处理了其余部分，包括读取表并将行转换为`OwnedTicket`对象。

现在我们也可以加载票了，我们可以更新本章开头创建的`OwnedTicketsPage`类，以显示用户预订的所有票。在我们的应用程序中，这意味着我们只会显示在此设备上预订的票。在真实的应用程序中，我们还会从远程服务器访问票务并将其下载到设备上；但是，由于这超出了本章的范围，我们不会这样做：

1.  在更新拥有的票页面之前，首先在`ViewModels`文件夹中添加一个`OwnedTicketsViewModel`类。该类的源代码在此处可用：[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/blob/main/Chapter05/DigitalTicket.Shared/ViewModels/OwnedTicketsViewModel.cs`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/blob/main/Chapter05/DigitalTicket.Shared/ViewModels/OwnedTicketsViewModel.cs)。

1.  现在，更新`OwnedTicketsPage.xaml`和`OwnedTicketsPage.xaml.cs`。您可以在 GitHub 上找到这两个文件的源代码：[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/Chapter05/DigitalTicket.Shared/Views`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/tree/main/Chapter05/DigitalTicket.Shared/Views)。

现在，如果启动应用程序并导航到拥有的票页面，您应该会看到一个空页面。如果您已经预订了一张票，您应该会看到*图 5.3*左侧的内容。如果您点击票下方的小、宽、灰色框，您应该会看到*图 5.3*右侧的内容：

![图 5.3 - 左：拥有单张票的票务列表；右：拥有的票和已预订票的 QR 码](img/Figure_5.03_B17132.jpg)

图 5.3 - 左：拥有单张票的票务列表；右：拥有的票和已预订票的 QR 码

当然，这还不是最终的 UI；用户应该看到指示他们尚未预订票的文本，而不是空白屏幕。不过，目前预计文本缺失，按钮也没有标签，因为它们使用的是`x:Uid`，而不是设置了`Text`或`Content`属性。在下一节中，我们将看看`x:Uid`是什么，并更新我们的应用程序，以便所有标签都能正确显示。

# 使您的应用程序准备好迎接客户

在本节中，我们将更新我们的应用程序，以便为我们的客户做好准备，包括本地化支持，使应用程序更易于客户使用。添加本地化支持后，我们将更新应用程序的图标和启动画面，以便用户更容易识别。

## 本地化您的应用程序

如果您正在开发一个面向客户的应用程序，能够以客户的母语提供翻译非常重要，特别是针对来自不同国家的客户的应用程序。在上一节中，我们已经添加了`x:Uid`属性并向`Strings.resw`文件添加了条目；但是，还有其他本地化资源的方法，我们将在后面介绍。我们将从`x:Uid`开始本地化文本。

### 使用 x:Uid 本地化您的 UI

使用`x:Uid`和资源文件（.resw 文件）是本地化应用程序的最简单方法，特别是因为添加新的翻译（例如，为新语言）非常容易。但是如何使用`x:Uid`和.resw 文件本地化您的应用程序呢？

`x:Uid`属性可以添加到你的 XAML 代码的任何元素上。除了在你想要提供翻译的控件上设置`x:Uid`属性之外，你还需要添加这些翻译。这就是`.resw`文件发挥作用的地方。简而言之，`resw`文件是包含必要条目的 XML 文档。然而，最容易想到的方法是将它们视为一张包含三个属性的条目列表，通常表示为表格。这些属性（或列）如下：

+   **Name**：你可以用来查找资源的名称。这个路径也将用于确定要设置哪个控件上的哪个属性。

+   **Value**：设置的文本或查找此资源时返回的文本。

+   **Comment**：你可以使用这一列提供解释该行的注释。当将应用程序翻译成新语言时，这是特别有用的，因为你可以使用注释找出最佳翻译是什么。查看*图 5.4*中的**Comment**列，了解它们可能如何使用。

在 Visual Studio 中打开`.resw`文件时，显示将如*图 5.4*所示：

![图 5.4 - 在 Visual Studio 中查看.resw 文件](img/Figure_5.04_B17132.jpg)

图 5.4 - 在 Visual Studio 中查看.resw 文件

当在 XAML 代码中使用`x:Uid`属性与`.resw`文件结合使用时，你需要注意如何编写资源的名称条目。名称条目需要以控件的`x:Uid`值开头，后跟一个点（`.`）和应该设置的属性的名称。因此，在前面的示例中，如果我们想要本地化`TextBlock`元素的文本，我们将添加一个名称值为`ButtonTextBlock.Text`的条目，因为我们想设置`TextBlock`元素的`Text`属性。

“但是本地化是如何运作的呢？”你可能会问。毕竟，我们只添加了一个条目；它怎么知道选择哪种语言呢？这就是为什么你放置`.resw`文件的文件夹很重要。在你的项目中，你需要有一个`Strings`文件夹。在该文件夹中，对于你想要将应用本地化的每种语言，你需要创建一个文件夹，比如`en-GB`，而对于*德语（德国）*，你需要创建一个名为`de-DE`的文件夹。在你为每种想要支持的语言创建的文件夹中，你需要放置`.resw`文件，以便本地化能够正常工作。请注意，如果某种语言不可用，资源查找将尝试找到下一个最佳匹配。你可以在这里了解更多关于这个过程的信息，因为你的 Uno 平台应用在每个平台上的行为都是相同的：[`docs.microsoft.com/windows/uwp/app-resources/how-rms-matches-lang-tags`](https://docs.microsoft.com/windows/uwp/app-resources/how-rms-matches-lang-tags)。

重要提示

要小心命名这些文件夹。资源查找将根据文件夹的名称进行。如果文件夹的名称有拼写错误或不符合 IETF BCP 47 标准，资源查找可能会失败，你的用户将看到缺少标签和文本，或者资源查找将退回到已翻译文本的语言混合。

我们已经有一个用于英文文本资源的文件夹；然而，我们也想支持德语翻译。为此，在`Strings`文件夹内创建一个名为`de-DE`的新文件夹。现在，添加一个名为`Resources.resw`的新`.resw`文件，并添加以下条目：

![表 5.2](img/Table_02.jpg)

如果现在启动应用并将应用的语言切换为德语，你会看到预订页面现在已本地化。如果你的设备语言已设置为德语，而不是以英语显示页面，现在应该显示为德语，即使你现在不切换到德语选项。

### 从代码后台访问资源

使用`x:Uid`并不是本地化应用程序的唯一方法；我们现在将看到如何可以从代码后台访问资源。例如，当您想要本地化集合中的项目时，例如我们应用程序中拥有的票证列表。要访问字符串资源，您可以使用`ResourceLoader`类。我们在本章开头添加了`LocalizedResources`类；然而，直到现在，它还没有访问任何资源。现在通过添加以下导入并替换`GetString`函数来更新`LocalizedResources`：

```cs
using Windows.ApplicationModel.Resources;
...
private static ResourceLoader cachedResourceLoader;
public static string GetString(string name)
{
    if (cachedResourceLoader == null)
    {
        cachedResourceLoader = 
            ResourceLoader.GetForViewIndependentUse();
    }
    if (cachedResourceLoader != null)
    {
        return cachedResourceLoader.GetString(name);
    }
    return null;
}
```

由于我们将经常使用加载的资源，我们正在缓存该值，以避免调用`GetForViewIndependentUse`，因为这是昂贵的。

现在我们已经介绍了`x:Uid`的工作原理以及如何从代码后台访问本地化资源，让我们更新应用程序的其余部分以进行本地化。首先，通过向我们的`.resw`文件添加必要的条目来开始。以下是您需要为`MainPage.xaml`文件及其英语和德语条目的条目表：

![Table 5.3](img/Table_03.jpg)

现在，请将`MainPage.xaml`文件中的`NavigationViewItems`属性替换为以下内容：

```cs
<muxc:NavigationViewItem x:Name="JourneyBookingItem" x:Uid="JourneyBookingItem" Tag="JourneyPlanner"/>
<muxc:NavigationViewItem x:Uid="OwnedTicketsItem" Tag="OwnedTickets"/>
<muxc:NavigationViewItem x:Uid="AllDayTicketsItem" Tag="AllDayTickets" IsEnabled="False"/>
<muxc:NavigationViewItem x:Uid="NetworkPlanItem" IsEnabled="False"/>
<muxc:NavigationViewItem x:Uid="LineOverViewItemItem" IsEnabled="False"/>
```

要将应用程序的其余部分本地化，请查看 GitHub 上的源代码。您还可以在那里找到英语和德语的更新的`Resources.resw`文件。请注意，我们选择不本地化车站名称，因为本地化街道和地名可能会让客户感到困惑。

重要提示

您还可以本地化其他资源，如图像或音频文件。为此，您需要将它们放在正确命名的文件夹中。例如，如果您想本地化名为`Recipe.png`的图像，您需要将该图像的本地化版本放在`Assets/[语言标识符]`文件夹中，其中`语言标识符`是图像所属语言的 IETF BCP 47 标识符。您可以在这里了解有关自定义和本地化资源的更多信息：[`docs.microsoft.com/windows/uwp/app-resources/images-tailored-for-scale-theme-contrast`](https://docs.microsoft.com/windows/uwp/app-resources/images-tailored-for-scale-theme-contrast)。

在本节中，我们介绍了如何使用`x:Uid`和资源文件本地化您的应用程序。随着您的应用程序变得更大并提供更多语言，使用多语言应用程序工具包可能会有所帮助。它使您更容易地检查哪些语言键未被翻译，并集成到 Visual Studio 中。您可以在这里了解更多信息：[`developer.microsoft.com/en-us/windows/downloads/multilingual-app-toolkit/`](https://developer.microsoft.com/en-us/windows/downloads/multilingual-app-toolkit/)。

## 自定义应用程序的外观

在将应用程序发布到商店时，您希望您的应用程序能够被用户识别并传达您的品牌。然而，到目前为止，我们开发的所有应用程序都使用了标准的 Uno 平台应用程序图标。幸运的是，Uno 平台允许我们更改应用程序的图标，并允许我们为应用程序设置启动图像。

### 更新应用程序的图标

让您的应用程序被用户识别的最重要的事情之一就是为您的应用程序添加图标。更新应用程序的图标很容易。您可以在这里找到我们将使用的图像：[`github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/blob/main/Chapter05/DigitalTicket.Shared/Assets/AppIcon.png`](https://github.com/PacktPublishing/Creating-Cross-Platform-C-Sharp-Applications-with-Uno-Platform/blob/main/Chapter05/DigitalTicket.Shared/Assets/AppIcon.png)。

#### 更新 Android 应用程序的图标

要更新 Android 应用程序的应用程序图标，您只需将 Android 项目的 drawable 文件夹中的`Icon.png`文件替换为所需的应用程序徽标。请注意，您还需要在项目属性中选择正确的图像。为此，请双击`Appicon`，您将在**Properties**节点内的`AndroidManifest.xml`文件中选择`android:icon`条目。

#### 更新 iOS 应用程序的图标

更新我们的 iOS 应用程序图标需要更多的工作。对于 iOS 应用程序，您需要根据应用程序安装的设备不同的尺寸来设置应用程序图标。要查看尺寸列表并更新 iOS 应用程序的应用图标，只需展开 iOS 项目的**Assets Catalog**节点，然后双击其中的**Media**条目。在**AppIcons**选项卡中，您可以选择不同设备和类别的图像和尺寸。并不需要为每个尺寸提供图像；但是，您至少应该为每个类别提供一个图标。

#### 更新 UWP 应用程序的图标

更新 UWP 头部的应用程序图标的最简单方法是使用`Package.appxmanifest`文件。为此，请双击`Package.appxmanifest`，然后在**Visual Assets**选项卡中选择**App icon**选项。要更新应用程序的图标，请选择源图像，选择目标文件夹，然后单击**Generate**。这将生成不同尺寸的应用程序图标，并因此将您的应用程序图标更新为指定的图像。

#### 更新其他项目的图标

虽然我们的应用程序将不会在其他平台上可用，并且我们已经删除了相应平台的头部，但您可能希望在其他项目中更新其他平台的图标：

+   `Assets/xcassets/AppIcon.appiconset`文件夹。如果重命名图像，请确保还更新`Contents.json`文件。

+   **基于 Skia 的项目**：在 Visual Studio 中右键单击项目并选择**属性**。在**应用程序**选项卡中，您可以使用**资源**部分中的**浏览**按钮选择一个新图标。

+   `favicon.ico`在项目的**Assets**文件夹中。

### 自定义您的应用程序启动画面

更新您的应用程序图标并不是使您的应用程序更具识别性的唯一方法。除了应用程序图标之外，您还可以自定义应用程序的启动画面。请注意，目前只有 Android、iOS、UWP 和 WASM 应用程序支持设置启动画面。与图标一样，您可以在 GitHub 上找到此类图像资源。

#### 更新 Android 启动画面

要向 Android 应用程序添加启动画面，您首先需要添加您的启动画面图像。在我们的情况下，我们将其命名为`SplashScreen.png`。之后，将以下条目添加到`Resource/values/Styles.xml`文件中：

```cs
<item name="android:windowBackground">@drawable/splash</item>
```

然后，您需要在`Resources/drawable`文件夹中添加`splash.xml`文件，并添加以下代码：

```cs
<?xml version="1.0" encoding="utf-8"?>
    <layer-list xmlns:android=
        "http://schemas.android.com/apk/res/android">
    <item>
        <!-- background color -->
        <color android:color="#008cff"/>
    </item>
    <item>
    <!-- splash image -->
        <bitmap android:src="img/splashscreen"
                android:tileMode="disabled"
                android:gravity="center" />
    </item>
</layer-list>
```

#### 更新 iOS 应用程序的启动画面

与任何 iOS 应用程序一样，启动画面需要是一个故事板。Uno Platform 使得很容易显示一个单一图像作为启动画面。只需这些简单的步骤：

1.  在解决方案资源管理器中，选择 iOS 项目并按下**显示所有文件**按钮。

1.  现在您将能够看到一个名为**LaunchScreeen.storyboard**的文件。右键单击此文件并选择**包含在项目中**。这将在启动应用程序时自动使用。

如果运行应用程序，您将看到 Uno Platform 标志在启动应用程序时显示。您可以通过替换图像来轻松更改此内容。

1.  在`SplashScreen@2x.png`和`SplashScreen@3x.png`中。这些是故事板使用的文件。用您想要的图像替换它们。

1.  要更改用于背景的颜色，您可以在 Xcode Interface Builder 中打开故事板并更改颜色。或者，您可以在 XML 编辑器中打开故事板文件，并更改颜色的`red`、`green`和`blue`属性，使用`backgroundColor`键。

可以使用任何您希望的内容作为启动屏幕的故事板文件。要做到这一点，您需要使用 Xcode Interface Builder。在版本**16.9**之前，Visual Studio 包括一个 iOS 故事板编辑器，但现在不再可用。要现在编辑故事板，您需要在 Visual Studio for Mac 中打开项目，右键单击文件，然后选择**打开方式** | **Xcode Interface Builder**。

#### 更新 UWP 应用程序的启动画面

与更新 UWP 应用程序的应用程序图标类似，使用`Package.appxmanifest`文件和`#008CFF`。现在，单击**生成**以生成 UWP 应用程序的启动画面图像。

#### 更新 WASM 应用程序的启动画面

要更新 WASM 头的启动画面，请将新的启动画面图像添加到 WASM 项目的`AppManifest.js`文件中的`WasmScripts`文件夹中，以引用该图像，并在必要时更新启动画面颜色。

如果您已成功按照我们的应用程序步骤进行操作，您应该能够在 Android 应用程序列表中看到应用程序，就像*图 5.5*左侧所示。一旦启动应用程序，您的应用程序在显示旅程预订页面之前应该看起来像*图 5.5*右侧所示。请注意，此处提供的图标和启动画面仅为示例。在真实的应用程序中，您应该确保您的应用程序图标即使很小也看起来不错：

![图 5.5 – 左：应用程序列表中的 DigitalTicket；右：DigitalTicket 的启动画面](img/Author_Figure_5.05_B17132.jpg)

图 5.5 – 左：应用程序列表中的 DigitalTicket；右：DigitalTicket 的启动画面

## 确保每个人都能使用您的应用程序

为确保每个人都能使用您的应用程序，您需要使其具有无障碍性。在开发应用程序时，无障碍性至关重要。所有能力水平的人都会使用您的应用程序；如果您的应用程序不具有无障碍性，将使您的客户生活更加困难，甚至可能使他们无法使用您的应用程序。

在考虑无障碍性时，大多数人首先想到的是通过为屏幕阅读器添加标签和替代文本来使应用程序对盲人无障碍。然而，无障碍性涉及的远不止这些。例如，视力较低但不是盲人的人可能不使用屏幕阅读器，而是选择使用高对比度主题使应用程序更易于使用，或者选择增大字体大小以便更容易阅读文本。提供暗色主题通常被视为纯粹的美学方面；然而，它在无障碍性方面也很重要。一些人可能能够更好地阅读文本，而某些残障人士可能会更难使用您的应用程序。

如果您已经熟悉 UWP 中可用的 API 来制作应用程序，那么在使您的 Uno 平台应用程序具有无障碍性时会有一些不同之处。由于您的应用程序将在不同的平台上运行，而这些平台都有不同的 API 来提供无障碍应用程序，Uno 平台在无障碍性方面只提供了一部分可用属性。在撰写本文时，只支持以下属性，并且在每个平台上都可以使用：

+   `AutomationProperties.AutomationId`: 您可以设置此属性以便使用辅助技术更轻松地导航到控件。

+   `AutomationProperties.Name`: 辅助技术将使用此属性向用户宣布控件。

+   `AutomationProperties.LabeledBy`: 设置此属性时，将使用此属性指定的控件来宣布设置了此属性的控件。

+   `AutomationProperties.AccessibilityView`: 使用此属性，您可以指示控件不应该被辅助技术向用户宣布，或者您想要包括通常不会被宣布的控件。

除了之前列出的属性外，Uno 平台还在每个平台上支持高对比度主题。由于我们使用 Uno 平台提供的标准控件，我们不需要特别关注这一点，因为 Uno 平台已经为我们的应用程序提供了正确的高对比度外观。但是，如果您编写自己的控件，您还应该检查您的应用程序的高对比度版本，以确保其可接受。

重要提示

您应该始终本地化辅助技术将使用的资源。不这样做可能会使您的应用程序无法访问，因为用户可能会遇到语言障碍，特别是如果辅助技术期望从一种语言中读出单词，而实际上却是另一种语言的单词。

为了确保您的应用程序对使用辅助技术的人员可访问，您需要使用辅助技术测试您的应用程序。在下一节中，您可以找到启动平台默认屏幕阅读器的说明。

### 在不同平台上启动屏幕阅读器

由于激活系统辅助技术的步骤因平台而异，我们将逐一进行介绍，从 Android 开始。

#### Android 上的 TalkBack

启动**设置**应用程序，打开**辅助功能**页面。按下**TalkBack**，并点击开关以启用 TalkBack。最后，按下**确定**关闭对话框。

#### iOS 上的 VoiceOver

打开**设置**应用程序，打开**通用**下的**辅助功能**选项。然后，在**视觉**类别中点击**VoiceOver**，并点击开关以启用它。

#### macOS 上的 VoiceOver

启动**系统偏好设置**，点击**辅助功能**。然后，在**视觉**类别中点击**VoiceOver**。勾选**启用 VoiceOver**以使用**VoiceOver**。

#### Windows 上的 Narrator（适用于 UWP 和 WASM）

要在 Windows 上启动**Narrator**屏幕阅读器，您只需同时按下 Windows 徽标键、*Ctrl*和*Enter*。

### 更新我们的应用程序以实现可访问性

在本章中，我们尚未确保我们的应用程序是可访问的。虽然许多控件已经可以自行访问，例如，按钮控件将宣布其内容，但仍然有一些控件需要我们在可访问性方面进行改进。如果用户使用辅助技术使用应用程序，不是所有内容都会以有意义的方式进行宣布。让我们通过更新应用程序的 UI 来改变这一点，以设置所有必要的属性。为此，我们将首先更新我们的旅程预订页面。

我们旅程预订页面上的两个`ComboBox`控件目前只会被宣布为`ComboBox`控件，因此使用辅助技术的用户不知道`ComboBox`控件实际用途。由于我们已经添加了描述其目的的`TextBlock`元素，我们将更新它们以使用`AutomationProperties.LabeledBy`属性：

```cs
<TextBlock x:Name="StartPointLabel" x:Uid="StartPointLabel" FontSize="20"/>
<ComboBox ItemsSource="{x:Bind journeyBookingVM.AllStations}" x:Uid="StartPointComboBox"
    AutomationProperties.LabeledBy="{x:Bind 
        StartPointLabel}"
    SelectedItem="{x:Bind 
        journeyBookingVM.SelectedStartpoint,Mode=TwoWay}"
    HorizontalAlignment="Stretch" 
        DisplayMemberPath="Name"/>
<TextBlock x:Name="EndPointLabel" x:Uid="EndPointLabel" FontSize="20"/>
<ComboBox ItemsSource="{x:Bind journeyBookingVM.AvailableDestinations, Mode=OneWay}" x:Uid="EndPointComboBox"
    AutomationProperties.LabeledBy="{x:Bind EndPointLabel}"
    SelectedItem="{x:Bind 
        journeyBookingVM.SelectedEndpoint,Mode=TwoWay}"
    HorizontalAlignment="Stretch" 
    DisplayMemberPath="Name"/>
```

现在，当用户使用辅助技术导航到`ComboBox`控件时，`ComboBox`控件将使用由`AutomationProperties.LabeledBy`引用的`TextBlock`元素的文本进行宣布。由于该页面上的其余控件已经为我们处理了可访问性，让我们继续进行所拥有的车票页面。

在所拥有的车票页面上，存在两个潜在问题：

+   车站名称旁边的图标将被宣布为空白图标。

+   QR 码将只被宣布为一个图像。

由于图标仅用于视觉表示，我们指示辅助技术不应使用`AutomationProperties.AccessibilityView`属性进行通告，并将其设置为`Raw`。如果您想为辅助技术包括一个控件，可以将该属性设置为`Content`。

为了确保 QR 码图像能够以有意义的方式进行通告，我们将为其添加一个描述性名称。为简单起见，我们将宣布它是当前选定车票的 QR 码。首先，您需要按照以下方式更新图像元素：

```cs
<Image x:Name="QRCodeDisplay" x:Uid="QRCodeDisplay"
    Source="{x:Bind ownedTicketsVM.CurrentQRCode,
             Mode=OneWay}"
    Grid.Row="4" MaxWidth="300" MaxHeight="300" 
        Grid.ColumnSpan="2"/>
```

之后，将以下条目添加到`Resources.resw`文件中：

**英语**：

![表格 5.4](img/Table_04.jpg)

**德语**：

![表格 5.5](img/Table_05.jpg)

通过添加这些条目，我们现在为显示的 QR 码提供了一个描述性名称，同时确保此文本将被本地化。

最后，我们还需要更新设置页面。由于它只包含一个单独的`ComboBox`控件，缺少名称，因此将以下条目添加到`Resources.resw`文件中：

**英语**：

![表格 5.6](img/Table_06.jpg)

**德语**：

![表格 5.7](img/Table_07.jpg)

在本节中，我们简要介绍了 Uno 平台中的可访问性；但是，我们没有提到还有一些限制和需要注意的事项。您可以在官方文档中阅读更多关于这些限制的信息：[`platform.uno/docs/articles/features/working-with-accessibility.html`](https://platform.uno/docs/articles/features/working-with-accessibility.html)。如果您希望了解更多关于一般可访问性的信息，您可以查看以下资源：

+   https://docs.microsoft.com/en-us/learn/paths/accessibility-fundamentals/

+   [`developer.mozilla.org/en-US/docs/Learn/Accessibility/What_is_accessibility`](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/What_is_accessibility)

+   [`developers.google.com/web/fundamentals/accessibility`](https://developers.google.com/web/fundamentals/accessibility)

# 摘要

在本章中，我们构建了一个在 iOS 和 Android 上运行的面向客户的应用程序。我们介绍了如何使用 SQLite 存储数据，如何使您的应用程序具有可访问性，并使其为客户准备好。作为其中的一部分，我们介绍了如何本地化您的应用程序，让用户选择应用程序的语言，并为您的应用程序提供自定义启动画面。

在下一章中，我们将为 UnoBookRail 编写一个信息仪表板。该应用程序将面向 UnoBookRail 的员工，并在桌面和 Web 上运行。
