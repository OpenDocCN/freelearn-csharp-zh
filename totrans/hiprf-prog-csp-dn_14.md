# *第 12 章*：响应式用户界面

在本章中，你将学习如何编写响应式用户界面。你将编写响应式的 **Windows Forms**（**WinForms**）、**Windows Presentation Foundation**（**WPF**）、ASP.NET、.NET MAUI 和 WinUI 应用程序。通过使用后台工作线程，你将了解如何在后台运行长时间运行的任务，从而实时更新和与 **用户界面**（**UI**）交互。

在本章中，我们将探讨以下主题：

+   **使用 WinForms 构建响应式 UI**：在本节中，你将编写一个简单的 WinForms 应用程序，该应用程序在执行多项任务的同时保持对用户交互的响应性。

+   **使用 WPF 构建响应式 UI**：在本节中，你将编写一个简单的 WPF 应用程序，该应用程序在执行多项任务的同时保持对用户交互的响应性。

+   **使用 ASP.NET 构建响应式 UI**：在本节中，你将编写一个简单的 ASP.NET 应用程序，该应用程序在执行多项任务的同时保持对用户交互的响应性。

+   **使用 .NET MAUI 构建响应式 UI**：在本节中，你将编写一个简单的 Xamarin.Forms 应用程序，该应用程序在执行多项任务的同时保持对用户交互的响应性。然后，你将通过更新库引用将项目从 Xamarin.Forms 迁移到 .NET MAUI。

+   **使用 WinUI 构建响应式 UI**：在本节中，你将编写一个简单的 WinUI 应用程序，该应用程序在执行多项任务的同时保持对用户交互的响应性。

通过学习本章，你将获得以下技能：

+   使用后台工作线程保持 UI 响应

+   使用等待屏幕在用户需要等待时提供更新

+   使用 AJAX、WebSockets、SignalR 和 gRPC/gRPC-Web 发送和接收数据以及传输资产

+   编写响应式桌面、Web 和移动 UI

    注意

    为了澄清，当在本章中讨论响应式 UI 时，我们不是在谈论 UI 布局适应设备大小或屏幕可用空间。相反，我们专注于使忙碌的 UI 对用户输入做出响应，而不是在任务执行期间阻止用户工作。

# 技术要求

+   Visual Studio 2022 或更高版本。

+   本章源代码可在 [https://github.com/PacktPublishing/High-Performance-Programming-in-CSharp-and-.NET/tree/master/CH12](https://github.com/PacktPublishing/High-Performance-Programming-in-CSharp-and-.NET/tree/master/CH12) 获取。

# 使用 WinForms 构建响应式 UI

在本节中，我们将构建一个非常简单的 WinForms 应用程序，该应用程序对 **每英寸点数**（**DPI**）敏感，并允许用户在长时间运行的操作期间继续工作。应用程序具有带有进度条和更新标签的启动画面，为用户提供视觉反馈，表明应用程序正在忙于加载。一旦加载进度完成，启动画面关闭，主窗口显示。

在主窗口中，有一个标签，每次点击增加计数按钮时都会更新，一个可以通过提供的按钮进行导航的分页表格，以及一个用于长时间运行任务的进度指示器，它还有一个取消按钮。

在长时间运行的任务执行期间，您可以移动窗口，通过点击增加计数按钮来增加标签，并且您可以浏览数据。如果您选择的话，您还可以取消长时间运行的任务。

当长时间运行的任务完成、取消或遇到错误时，任务进度面板将被隐藏。

## 启用DPI感知和长文件路径感知

在本节中，我们将配置一个WinForms应用程序，使其在高DPI屏幕和普通DPI大屏幕上看起来很好。我们还将其配置为能够识别长文件路径。请按照以下步骤操作：

1.  启动一个新的.NET 6 WinForms应用程序，并将其命名为`CH12_ResponsiveWinForms`。

1.  添加一个新的*应用程序清单*文件。

1.  打开`app.manifest`文件并更新`compatibility`部分如下：

    [PRE0]

此XML代码使Windows Vista及更高版本的WinForms应用程序具有DPI感知能力。

1.  取消以下`application`部分的注释：

    [PRE1]

此代码通知编译器应用程序具有对长路径和DPI设置的感知能力。有了这些设置，应用程序现在将根据不同的屏幕DPI设置进行缩放，并且能够处理长达256个字符的长路径。

在下一节中，我们将添加一个带有加载进度反馈的启动画面。

## 添加一个随着加载进度更新的启动画面

应用程序可以非常快速地加载，或者加载相当缓慢。当它们正在加载时，用户并不知道应用程序正在做什么。您可以选择显示启动画面作为您应用程序品牌的一部分。如果您的应用程序加载速度快，那么您可能需要添加一个短暂的延迟，例如3秒，以便用户能够看到启动画面。否则，用户可能看到的只是屏幕快速闪烁。

如果应用程序有一些需要花费时间处理的繁重加载操作，用户可能会认为有问题，程序已崩溃。因此，提供提供视觉反馈的启动画面是一种良好的做法。这样，用户就知道应用程序正在忙于处理，并没有崩溃。当用户看到这样的反馈时，他们会更有耐心，并等待应用程序加载完成。

在本节中，我们添加了一个带有视觉反馈的启动画面。主窗口通过延迟模拟几个加载操作。然后，关闭启动画面并显示主窗口。现在，我们将开始添加必要的代码：

1.  添加一个名为`SplashScreenForm`的新表单，并将其**FormBorderStyle**属性更改为**None**，**StartPosition**属性更改为**CentreScreen**。将**BackColor**属性更改为**ActiveCaptionText**。

1.  在表单中添加一个`LoadingProgressBar`并将其停靠到表单底部。

1.  向 `LoadingProgressLabel` 添加一个标签并将其停靠到表单底部，使其出现在进度条上方。将 **Text** 属性设置为 **Loading. Please wait…**，并将 **Font** | **Size** 设置为 **12**。将 **ForeColor** 属性更改为 **HighlightWhite**。将 **Margin** | **All** 和 **Padding** | **All** 设置为 **8**。

1.  向 `TitleLabel` 添加另一个标签，其 **Text** 属性设置为 **Responsive WinForms Example**，**ForeColor** 设置为 **HighlightText**，**Font** | **Size** 设置为 **32**，并将 **Location** 设置为 **29, 126**。

1.  重命名 `MainForm` 并打开表单。双击 WindowsForm。这将打开代码窗口。

1.  向 `MainForm` 类添加以下 `using` 语句：

    [PRE2]

这些 `using` 语句为我们启动画面的代码提供了所需的所有功能。

1.  向 `MainForm` 类添加以下成员变量：

    [PRE3]

这些成员变量将在我们的 `MainForm` 类的各个方法中被引用，以提供分页、内存数据存储以及存储正在处理操作的点击计数和操作编号。

1.  按照以下方式更新 `MainForm_Load` 方法：

    [PRE4]

此代码创建我们的启动画面，然后迭代 100 次，模拟许多加载操作。每次迭代都会使 UI 线程休眠半秒，更新启动画面进度，并通过调用 `Application.DoEvents()` 释放线程，以便其他线程可以通过调用 `Application.DoEvents()` 来执行它们的工作。

1.  打开 **SplashScreenForm** 并查看其代码。添加以下方法：

    [PRE5]

此代码从 `MainForm` 类获取输入，并更新启动画面的标签和进度条，向用户提供应用程序正在加载和进度的反馈。

我们现在已经完成了进度条。如果您运行代码，您将看到以下启动画面：

![图 12.1 – WinForms 启动画面

](img/Figure_12.01_B16617.jpg)

图 12.1 – WinForms 启动画面

现在我们的启动画面已经工作，让我们添加显示按钮点击增量计数的标签和按钮。

## 添加增量计数按钮和标签

为了演示在执行长时间操作时 UI 不会被阻塞，我们将有一个标签，每次用户点击按钮时都会更新其文本。在我们的代码中，我们需要执行以下任务：

1.  向 `MainForm` 添加一个名为 `ClickCounterLabel` 的标签并将其停靠到顶部。将其文本设置为空字符串，文本属性设置为 **Segoe UI** 和 **36pt**，并将 **TextAlign** 设置为 **MiddleCenter**。

1.  向表单添加一个名为 `IncrementCountButton` 的按钮并将其停靠到表单顶部。将其文本设置为 **&Increment Text**。

1.  *双击* 按钮，生成其点击事件。更新点击事件的代码如下：

    [PRE6]

每次用户点击按钮，`_clickCounter` 变量就会增加一。然后更新 `ClickCounterLabel` 文本，通知用户他们点击按钮的次数。

接下来，我们将添加一个带有分页导航的表格。我们将在下一节中完成这项工作。

## 添加具有分页数据的表格

在本节中，我们将添加一个带有分页导航的表格。这将演示，即使在后台运行长时间操作时，用户仍然可以通过 WinForms 应用程序中的数据与页面进行交互。让我们开始：

1.  添加 `DataTable`，并设置其 **Dock** 属性为 **Fill**。

1.  添加 `DataPagingPanel`，将其 **Dock** 属性设置为 **Bottom**。

1.  在 `FirstButton` 上添加一个按钮，文本设置为 **|<<**。*双击* 按钮以生成点击事件。然后，返回到设计窗口。

1.  在 `PreviousButton` 上添加一个按钮，文本设置为 **<<**。*双击* 按钮以生成点击事件。然后，返回到设计窗口。

1.  在 **FlowLayoutPanel** 中添加一个名为 `PageTextBox` 的文本框。

1.  在 **FlowLayoutPanel** 中添加一个名为 `NextButton` 的按钮，文本设置为 **>>**。*双击* 按钮以生成其点击事件。然后，返回到设计窗口。

1.  在 **FlowLayoutPanel** 中添加一个名为 `LastButton` 的按钮，文本设置为 **>>|**。*双击* 按钮以生成其点击事件。这次，保持代码视图，因为我们已经完成了本节 UI 需要完成的工作。

1.  添加 `BuildCollection` 方法：

    [PRE7]

此方法构建一个包含 `100` 个产品的集合。

1.  在 `SplashScreenForm` 实例化行之前，将 `BuildCollection` 方法的调用添加到 `MainForm_Load` 方法中。

1.  在关闭启动屏幕的行之后，添加以下两行代码：

    [PRE8]

此代码设置 `PagedProducts` 方法的数据源。

1.  添加 `PagedProducts` 方法：

    [PRE9]

此方法从 `_products` 集合返回一个范围。`_offset` 变量存储构成返回集合起始点的索引值，`_pageSize` 变量存储每页要返回的记录数。

1.  添加 `PageCount` 方法：

    [PRE10]

此方法获取 `_products` 集合中包含的产品数量，将该数量除以 `_pageSize` 变量，然后返回结果。结果是我们可以导航的数据页数。

1.  按如下方式更新 `FirstButton_Click` 方法：

    [PRE11]

此代码将数据集的第一页移动到当前页，并相应地更新 UI。

1.  使用以下代码更新 `PreviousButton_Click` 方法：

    [PRE12]

此代码将数据集的前一页移动到当前页，并相应地更新 UI。

1.  添加 `NextButton_Click` 方法代码：

    [PRE13]

此代码将数据集的下一页移动到当前页，并相应地更新 UI。

1.  添加 `LastButton_Click` 方法代码：

    [PRE14]

此方法将数据集的最后页移动到当前页，并相应地更新 UI。

1.  最后，添加 `Product` 类：

    [PRE15]

此类是 `Product` 类，我们的 `MainForm` 在其 `BuildCollection` 方法中使用它来构建其产品列表。

我们现在已经构建了我们的分页数据表，并且我们的增量按钮和标签已经就位。我们表单的最后一件事是添加我们的长时间运行的任务，以表明用户交互仍然可能，而不会因为长时间运行的任务而被阻塞。这将是下一节的主题。

## 在后台运行长时间运行的任务

在本节中，我们将升级我们的 UI 以显示在后台运行的长运行任务的进度。用户可以在任何时候取消长时间运行的任务。当任务完成时，无论其状态如何，长时间运行的任务更新进度控件将从用户隐藏。让我们开始添加代码：

1.  添加一个 `LongRunningOperationCancelButton` 并将其文本设置为 `&取消长时间运行操作`。

1.  添加一个 `StatusBar`。

1.  添加一个 `TaskProgressBar`。

1.  添加一个 `StatusLabel` 并确保其文本属性为空。

1.  添加一个 `CollectionBuilderBackgroundWorker`。

1.  添加一个 `LongRunningProcessBackgroundWorker`。

1.  在 `MainForm` 类的构造函数中，添加以下三行：

    [PRE16]

此代码为我们的 `BackgroundWorker` 添加了处理程序，该处理程序将负责执行长时间运行的任务。

1.  在 `MainForm_Load` 方法的最后行（在闭合括号之前）添加以下方法调用：`LongRunningProcess();`。

1.  添加以下 `LongRunningProcess` 方法：

    [PRE17]

如果 `LongRunningProcessBackgroundWorker` 没有忙碌，则调用 `RunWorkerAsync` 方法的 `LongRunningProcessBackground Worker_DoWork` 将被执行。

1.  将 `LongRunningProcessBackgroundWorker_DoWork` 添加到 `MainForm` 类：

    [PRE18]

我们将发送者强制转换为 `BackgroundWorker` 并将其分配给我们的本地工作变量。然后，我们迭代 100 次。每次迭代时，我们将 `_operationNumber` 变量设置为循环计数变量的值，休眠 100 毫秒，然后调用工作者的 `ReportProgress` 方法，传入已完成的工作百分比。

1.  将 `LongRunningProcessBackgroundWorker_ProgressChanged` 方法添加到 `MainForm` 类：

    [PRE19]

此代码使用长运行任务的进度更新 UI。如果所有操作都已完成，则隐藏任务取消按钮和状态栏。

1.  将 `LongRunningProcessBackgroundWorker_RunWorkerCompleted` 方法添加到 `MainForm` 类：

    [PRE20]

当长时间运行的任务完成时，此方法将 `StatusLabel.Text` 执行到方法的输出，输出结果为 `Cancelled`、`Error` 或 `Done`。

1.  在完成并运行我们的 WinForms 应用程序之前，我们需要编写的最后一部分代码是向 `MainClass` 的 `LongRunningOperationButton_Click` 方法添加代码，如下所示：

    [PRE21]

此代码检查任务是否支持取消。如果支持，则取消任务，并将取消按钮和状态栏从用户隐藏。

1.  运行代码。您应该看到*图12.1*中显示的启动画面。然后，您应该看到类似于*图12.2*所示的主窗口。移动窗口并点击递增计数按钮。此外，点击翻页按钮在数据集的不同数据页之间移动，并取消任务。您应该看到窗口完全响应您的输入，如下所示：

![图12.2 – Windows Forms主应用程序窗口

](img/Figure_12.02_B16617.jpg)

图12.2 – Windows Forms主应用程序窗口

如您所见，我们编写了一个功能丰富的WinForms应用程序。我们有一个启动画面，它向用户提供视觉反馈，这样他们就不会认为应用程序以任何方式崩溃，并且我们有一个在长时间运行的任务期间对用户输入保持响应的UI。

现在我们已经有一个工作的WinForms应用程序，让我们将注意力转向WPF。在下一节中，我们将把我们在WinForms应用程序中学到的知识应用到WPF应用程序中。

# 使用WPF构建响应式UI

在本节中，我们将构建与WinForms应用程序相同的界面，但这次将使用WPF。我们现在将开始编写我们的代码：

1.  创建一个名为`CH12_ResponsiveWPF`的新WPF应用程序，并确保选择**.NET 6.0**作为目标框架。

1.  将`Product`类添加到项目中。它与我们在WinForms应用程序中使用的代码相同。

1.  添加一个名为`SplashWindow`的新窗口。

1.  按如下方式修改**SplashWindow** XAML：

    [PRE22]

我们刚刚更新的XAML声明了一个包含两个标签和进度条的堆叠面板。第一个标签显示标题，第二个标签显示与进度条一起的加载进度。

1.  将以下方法添加到`SplashWindow`类中：

    [PRE23]

这段代码将由`MainWindow`类调用，并负责更新**SplashWindow**上的进度指示器。

1.  打开`MainWindow.xaml`文件，并用以下内容替换现有的XAML：

    [PRE24]

此XAML提供了一个状态面板，将显示任何后台任务的进度，一个递增标签和一个递增按钮，一个数据网格，以及用于翻页不同数据页面的导航面板。

1.  将以下`using`语句添加到`MainWindow.xaml.cs`文件中：

    [PRE25]

这些`using`语句对于我们的WPF窗口正常工作是必需的。

1.  将以下成员变量添加到`MainWindow`类中：

    [PRE26]

在这里，我们有与WinForms应用程序相同的变量，除了我们声明了一个后台工作线程。

1.  使用以下代码更新`MainWindow`构造函数：

    [PRE27]

这段代码基本上与我们的WinForms加载方法相同。唯一的真正区别是我们所有的初始化代码都在构造函数中。

1.  添加`Worker_DoWork`方法：

    [PRE28]

这段代码模拟了100个操作的工作，每个操作之间有短暂的延迟。

1.  添加`Worker_ProgressChanged`方法代码：

    [PRE29]

这段代码更新了长时间运行任务的进度指示器。

1.  添加 `Worker_RunWorkerCompleted` 方法：

    [PRE30]

此方法报告长时间运行任务的结果，然后从最终用户那里隐藏状态面板。

1.  添加 `PagedData` 方法：

    [PRE31]

此方法返回一个数据页，其索引从 `_offset` 开始，返回的行数由 `_pageSize` 定义。

1.  添加 `DoEvents` 方法：

    [PRE32]

此代码的行为类似于 WinForms 的 `Application.DoEvents()` 代码。您可以将非 UI 阻塞代码放在这里，并更新 UI。

1.  添加 `BuildCollection` 方法：

    [PRE33]

`BuildCollection` 方法构建我们的 100 个产品数据集。

1.  添加 `PageCount` 方法：

    [PRE34]

`PageCount` 方法根据数据集大小和页面大小计算数据页数，然后返回结果。

1.  添加 `FirstButton_Click` 方法：

    [PRE35]

当执行时，此方法将导航到我们数据集的第一条记录，并相应地升级 UI。

1.  添加 `PreviousButton_Click` 方法：

    [PRE36]

此方法将移动到数据集的前一页，并相应地更新 UI。

1.  添加 `NextButton_Click` 代码：

    [PRE37]

此方法移动到数据集的下一页，并相应地更新 UI。

1.  添加 `LastButton_Click` 方法：

    [PRE38]

此方法移动到最后一个数据集页面，并相应地更新 UI。

1.  添加 `IncrementCounterButton_Click` 方法：

    [PRE39]

每次您点击 `IncrementCounterButton`，此方法将增加 `_clickCounter` 变量，并在屏幕上报告您点击按钮的次数。

1.  添加最终名为 `CancelTaskButton_Click` 的 WPF 方法：

    [PRE40]

如果支持取消，此方法将取消长时间运行的任务。

1.  运行 WPF 应用程序。您将看到显示加载进度的启动屏幕，如下所示：

![Figure 12.3 – WPF 应用程序的启动屏幕

![img/Figure_12.03_B16617.jpg]

图 12.3 – WPF 应用程序的启动屏幕

加载完成后，启动屏幕关闭，您将看到主窗口。在长时间运行的任务进行时，您可以移动窗口，点击增加计数器按钮，浏览分页数据，并取消长时间运行的任务。

如以下截图所示，我们已准备好一切，为最终用户提供进度视觉反馈，并在长时间运行的任务期间保持对用户输入的响应性 UI：

![Figure 12.4 – WPF 应用程序的主窗口

![img/Figure_12.04_B16617.jpg]

图 12.4 – WPF 应用程序的主窗口

在下一节中，我们将探讨如何保持 ASP.NET UI 对用户输入的响应性。

# 使用 ASP.NET 构建响应式 UI

在本节中，我们将探讨如何帮助 ASP.NET 应用程序快速响应。我们将首先查看内存和分布式缓存。然后，我们将探讨如何使用 AJAX 更新页面的一部分。接下来，我们将编写一个实时聊天应用程序，使用 SignalR。然后，我们将探讨在 ASP.NET 应用程序中使用 WebSockets。

注意

在本章中，我们将不会介绍 gRPC-Web，因为我们已经在 [*第 9 章*](B16617_09_Final_SB_Epub.xhtml#_idTextAnchor168) 中通过示例代码介绍了该主题，即 *增强网络应用程序的性能*，其中我们探讨了 gRPC 用于非 Web 应用程序和 gRPC-Web 用于 Web 应用程序。在本章中，我们还使用 gRPC-Web 实现了一个简单的 Blazor Web 应用程序，因此你可以参考本章以了解 gRPC/gRPC-Web。

让我们通过关注缓存来开始查看一个响应式 ASP.NET 应用程序。我们将查看两种类型的缓存。这些是 **内存缓存** 和 **分布式缓存**。在下一节中，我们将实现内存缓存。

## 实现内存缓存

Web 应用程序通过网络（我们所有人都知道的网络）加载资源。从互联网访问、下载和渲染资源需要不同程度的时间。时间可能会因网络流量、网络质量以及计算机系统资源而变化。我们有没有办法加快这个过程？嗯，是的。我们可以实现缓存。但缓存究竟是什么呢？

**缓存**是将频繁访问的资源本地存储以提高访问和处理速度的一种方式。

在本节中，你将看到我们如何在 ASP.NET 中轻松实现内存缓存。要实现内存缓存，请按照以下步骤操作：

1.  开始一个新的 ASP.NET Core Web 应用程序（模型-视图-控制器）项目，并将其命名为 `CH12_ResponsiveASPNET`。

1.  添加 `Microsoft.Extensions.Caching.Memory` NuGet 包。如果 Visual Studio 无法安装它，请在包管理器中运行以下命令：

    [PRE41]

1.  在 `HomeController` 类中，添加语句 `using Microsoft.Extensions.Caching.Memory`。

1.  添加以下成员变量：

    [PRE42]

此代码声明了将存储我们的日志记录器和内存缓存对象的变量。

1.  如下更新 `HomeController` 构造函数：

    [PRE43]

在此代码中，我们将注入我们将要使用的日志记录器和内存缓存对象，并传递变量以设置我们的成员变量。

1.  添加 `GetMemoryCacheTime` 方法：

    [PRE44]

在这里，我们检查我们的 `CachedTime` 变量是否存在于内存缓存中。如果它存在，则将 `currentTime` 变量设置并返回缓存的时长。否则，我们获取当前时间并将其存储在内存缓存中，并带有滑动过期值，然后返回缓存的时长。

1.  使用以下代码更新 `Index` 方法：

    [PRE45]

`Index` 控制器方法返回一个字符串。返回的字符串是缓存的时长。

1.  运行项目并导航到 `https://localhost:5001/Home`。你应该会看到以下类似的输出：

**当前时间：2021年12月7日 20:18:25**

**内存缓存时间：2021年12月7日 20:18:25**

如你所见，时间不存在于缓存中，因此在返回之前被添加到缓存中。

注意

端口号的设置取决于端口的可用性。无论你选择哪个端口，如果它已被其他程序占用，则将无法工作。

1.  现在，刷新页面，你应该会看到当前时间和内存缓存时间的不同值：

**当前时间：12/07/2021 20:21:21**

**内存缓存时间：12/07/2021 20:18:25**

你可以清楚地看到内存缓存时间早于当前时间。这表明我们已经将时间存储在内存缓存中并成功检索。

在 ASP.NET 中实现内存缓存非常简单，你可以通过存储和检索内存缓存中的数据来提高页面加载和渲染时间。现在我们已经了解了内存缓存，我们将转向分布式缓存。

## 实现分布式缓存

在本节中，我们将使用相同的 ASP.NET Web 项目和控制器来实现分布式缓存。我们所说的分布式缓存是什么意思？分布式缓存扩展了本地缓存的概念，包括跨多台计算机的缓存。这种缓存使得事务数据的扩展成为可能。你主要会使用分布式缓存来存储位于数据库中的应用程序数据，以及与 Web 会话相关的数据。在本节中，我们使用 Redis 进行缓存。Redis 是一个内存数据结构存储，用作分布式、内存中的键值数据库、缓存和消息代理，具有可选的持久性。要实现分布式缓存，执行以下操作：

1.  将 `Microsoft.Extensions.Caching.Redis` NuGet 包添加到 Web 包中。你可以使用以下命令：

    [PRE46]

1.  在 `HomeController` 类中，添加 `using Microsoft.Extensions.Caching.Distributed` 语句。

1.  添加以下成员变量：

    [PRE47]

这个变量将保存我们的分布式缓存对象，该对象通过构造函数注入。

1.  现在，更新构造函数代码：

    [PRE48]

我们正在注入分布式缓存对象并设置我们的成员变量。

1.  要使用我们的分布式缓存，我们需要对 Base64 字符串进行编码和解码。添加以下两个方法：

    [PRE49]

在这两个方法中，我们将字符串编码为 Base64 编码的字符串，同时也将字符串从 Base64 解码为 UTF8。

1.  添加 `GetDistriutedCacheString` 方法：

    [PRE50]

在此代码中，我们从缓存中获取字符串数据。如果存在，则返回它。如果不存在，则将字符串的 Base64 编码版本保存到缓存中，并设置绝对过期时间，然后以 UTF 编码的字符串形式返回字符串的 Base64 解码版本。

1.  更新 `HomeController.Index` 方法，如下所示：

    [PRE51]

此代码获取内存缓存和分布式缓存存储的数据，并将其输出给用户，显示当前时间、内存缓存时间和分布式缓存中的数据。

1.  运行程序并导航到 `https://localhost:5001`。你应该会看到以下输出：

    [PRE52]

我们可以看到，内存缓存时间和分布式缓存字符串都刚刚被添加到缓存中，因为它们与当前时间相同。现在，刷新你的浏览器。你应该会看到这两个缓存值都早于当前时间，如下所示：

[PRE53]

在本节和上一节中，您已经看到如何轻松地将内存和分布式缓存添加到我们的应用程序中。这两种缓存形式都可以在提高您的ASP.NET Web应用程序性能方面非常有用。在下一节中，我们将探讨如何使用AJAX更新当前显示页面的一个小部分。

## 使用AJAX更新当前显示页面的部分

在本节中，我们将使用AJAX来更新当前正在显示的页面的一部分。这样我们就不必加载整个页面。让我们开始编写我们的AJAX示例：

1.  右键单击 `Controllers` 文件夹。从上下文菜单中选择 **添加** | **控制器…**。然后，选择 **MVC 控制器 – 空的**。

1.  调用新的控制器`AjaxController`并打开类。

1.  通过添加以下方法来更新控制器：

    [PRE54]

当调用此方法时，将返回一个JSON结果，在我们的例子中是一个简单的字符串。

1.  *右键单击* `Index` 方法并选择 `index.cshtml`。

1.  使用以下HTML和JavaScript代码更新 `Views/Ajax/index.cshtml` 文件：

    [PRE55]

我们有一个HTML表单。该表单有一个按钮，当按下时，将执行JavaScript，通过执行我们的`AjaxDemo`操作方法来检索AJAX数据。这将导致我们的JSON字符串在页面上显示。

1.  运行项目并导航到 `http://localhost:5001/Ajax`。您应该看到以下内容：

![Figure 12.5 – 在AJAX检索之前AJAX演示](img/Figure_12.05_B16617.jpg)

![img/Figure_12.05_B16617.jpg](img/Figure_12.05_B16617.jpg)

![Figure 12.5 – 在AJAX检索之前AJAX演示](img/Figure_12.05_B16617.jpg)

如您所见，我们的页面在没有我们的JSON字符串的情况下加载。现在，点击 **Ajax 演示** 按钮。您现在看到以下内容：

![Figure 12.6 – 使用AJAX检索并显示的JSON字符串](img/Figure_12.06_B16617.jpg)

![img/Figure_12.06_B16617.jpg](img/Figure_12.06_B16617.jpg)

![Figure 12.6 – 使用AJAX检索并显示的JSON字符串](img/Figure_12.06_B16617.jpg)

点击按钮后，我们可以看到AJAX操作检索了我们的JSON字符串，并在不进行完整页面加载的情况下将其显示在页面上。

我们已经看到如何使用AJAX更新页面的一部分，在此之前，我们看到了如何实现内存和分布式缓存。在下一节中，我们将探讨如何实现WebSockets。

## 实现WebSockets

在本节中，我们将实现 **WebSockets**。您可能已经听说过WebSockets，但它们是什么？WebSocket是一个全双工通信协议，用于通过单个TCP连接进行通信。要了解更多关于WebSocket规范的信息，您可以查阅2011年的IETF RFC 6455（[https://www.rfc-editor.org/rfc/rfc6455.txt](https://www.rfc-editor.org/rfc/rfc6455.txt)）。

我们为什么要使用WebSockets？嗯，我们可以使用它们在浏览器和服务器之间打开一个单一的双向交互会话。这样，我们可以取消服务器轮询，向服务器发送消息，并通过事件接收响应。从而使我们的应用程序成为事件驱动的。

在我们的WebSocket演示中，我们将点击一个按钮。它将打开一个WebSocket，发送消息，接收响应，然后关闭连接。我们浏览器和服务器之间的通信将输出到我们的网页上。因此，让我们开始编写我们的WebSocket示例：

1.  添加一个名为`WebSocketsController`的新控制器。

1.  右键单击`Index`方法并从上下文菜单中选择**添加视图**。

1.  按照以下方式更新`Views/WebSockets/Index.cshtml`文件：

    [PRE56]

当通过按钮点击打开WebSocket时，`messages`段落会更新消息，然后向服务器发送一条消息。当服务器响应时，`messages`段落随后更新以通知用户服务器已响应。如果发生错误，则向用户显示一条消息。然后关闭WebSocket并在页面上显示一条消息。

1.  运行代码并导航到`http://localhost:5001/WebSockets`。点击按钮，你应该会得到以下结果：

![图12.7 – 点击按钮并执行我们的WebSocket示例的最终结果]

![img/Figure_12.07_B16617.jpg](img/Figure_12.07_B16617.jpg)

图12.7 – 点击按钮并执行我们的WebSocket示例的最终结果

WebSocket的代码并不多。在这个示例中，我们发送了一条简单的消息并收到了响应。我们用于执行此操作的所有代码都存在于视图的`CSHTML`文件中。在下一节中，我们将探讨如何使用SignalR编写实时聊天程序。

## 使用SignalR实现实时聊天应用程序

在本节中，我们将学习如何使用SignalR在ASP.NET web应用程序中编写实时功能。我们将通过编写一个简单的聊天应用程序来演示SignalR的实际应用。现在，我们将开始编写应用程序：

1.  右键单击项目，并从上下文菜单中选择**添加** | **客户端库**，然后填写如图12.8所示的详细信息。然后，点击**安装**按钮：

![图12.8 – 添加客户端库以配置安装SignalR]

![img/Figure_12.08_B16617.jpg](img/Figure_12.08_B16617.jpg)

图12.8 – 添加客户端库以配置安装SignalR

1.  将`wwwroot/lib/microsoft/signalr`库复制并粘贴到`wwwroot/js`文件夹中。

1.  添加一个名为`SignalRController`的新控制器。

1.  在主项目根目录下添加一个名为`Hubs`的文件夹。

1.  在`Hubs`文件夹中添加一个名为`ChatHub`的类。然后，更新`ChatHub`类，如下所示：

    [PRE57]

我们已经有了SignalR hub类，并且`SendMessage`方法异步地向指定用户发送消息。

1.  在`SignalRController`类的`Index`方法上右键单击，并从上下文菜单中选择**添加视图**。

1.  在`Views/SignalR/Index.cshtml`文件中，将现有内容替换为以下代码：

    [PRE58]

我们已经构建了一个聊天UI。脚本使用SignalR。我们现在需要添加使我们的UI交互的JavaScript。

1.  在`wwwroot/js`文件夹中，添加一个名为`chat.js`的文件，包含以下代码：

    [PRE59]

我们添加了使我们的 UI 交互式的 JavaScript。此代码管理用户之间的聊天消息发送。

1.  在 `Program` 类中，添加以下服务：

    [PRE60]

此代码将 SignalR 添加到我们的可用服务中，以便我们可以将 SignalR 请求传递给 SignalR。

注意事项

如果使用新的最小模板，代码是 `builder.Services.AddRazorPages(); builder.Services.AddSignalR();`。

1.  更新 `Program` 类以包含映射到我们的 `ChatHub` 的路由：

    [PRE61]

我们已经包含了到我们的 `ChatHub` 的路由，这样我们的聊天应用程序就知道如何处理传入的请求。

1.  运行代码并导航到 `https://localhost:5001/SignalR`。你需要两个并排的浏览器实例。在每个浏览器中输入用户名和消息，然后点击**发送消息**按钮。每次输入文本时，它都会出现在接收者的聊天页面上，就像这里所示：

![Figure 12.9 – Our SignalR application in action]

![img/Figure_12.09_B16617.jpg]

![Figure 12.9 – Our SignalR application in action]

设置和运行我们的 SignalR 比较直接。正如你所见，SignalR 是实时通信的一个优秀选择，我相信你将能够在你编写的网络应用程序中进一步运用这些知识。这就结束了本章关于 ASP.NET 的内容。现在，让我们继续下一节，看看 .NET MAUI。

# 使用 .NET MAUI 构建响应式 UI

Microsoft .NET MAUI 是 Xamarin.Forms 的新版本。在 Xamarin.Forms 5.0 版本和 .NET MAUI（Xamarin.Forms 6.0 版本）之间有一些重大变化。MAUI 最大的变化是将 Android、iOS 和 macOS 项目合并为一个主项目。虽然针对 Windows 的特定代码仍然位于其自己的项目中，但 Microsoft 正在努力将 Windows 代码包含到主项目中。这将使我们能够使用 C# 和 XAML 编写跨平台应用程序时拥有一个单一的项目。让我们看看使用 .NET MAUI 构建跨平台应用程序的其他一些改进。

注意事项

如果你使用的是 MAUI 的早期版本，要运行 Windows 项目，你需要将 Windows 项目设置为启动项目并部署项目。一旦项目部署完成，你就可以从 Windows 启动菜单中运行应用程序。

## 布局

在 .NET MAUI 中做出的另一个重大更改是，Xamarin.Forms 项目原本使用的布局已被移动到 `Microsoft.Maui.Controls.Compatibility` 命名空间。默认情况下，MAUI 将使用新的布局。这些布局基于一个新编写的 `LayoutManager`，它针对性能、一致性和可维护性进行了优化。新的布局包括 `Grid`、`FlexLayout` 和 `StackLayout`（`HorizontalStackLayout` 和 `VerticalStackLayout`）。Microsoft 鼓励你选择最适合你需求的堆叠布局。同时，也鼓励你用新的布局替换旧布局。

新布局的默认间距值已标准化为0。将这些值设置为0设定了您将设置自己的首选值以满足设计要求的预期。最好在全局样式设置这些值，如下所示：

[PRE62]

[PRE63]

[PRE64]

[PRE65]

[PRE66]

[PRE67]

[PRE68]

[PRE69]

[PRE70]

让我们继续看看可访问性的改进。

## 可访问性

微软定期与那些致力于开发最高可访问性评级应用程序的开发者会面。这促使微软移除了`TabIndex`和`IsTabStop`属性，因为它们最终变得令人困惑，并且没有满足可访问性需求。为了提高可访问性，您可以通过实施深思熟虑的设计来提高屏幕阅读器识别UI阅读顺序的能力。如果您需要控制UI组件的顺序，微软建议您使用`SemanticOrderView`组件。

### SetSemanticFocus和Announce

屏幕阅读器是可访问且友好的应用程序的重要组成部分。为了帮助这些应用程序在读取正确组件方面的性能，有一个新的`SemanticExtensions`类。作为此类的一部分，有一个名为`SetSematicFocus`的新方法。此方法允许将屏幕阅读器的焦点设置到特定元素。

注意

在撰写本文时，`SetSemanticFocus`和`Announce`仅适用于iOS、Android和Mac Catalyst。

这里是一个设置语义焦点的XAML示例：

[PRE71]

[PRE72]

[PRE73]

[PRE74]

[PRE75]

[PRE76]

[PRE77]

[PRE78]

[PRE79]

[PRE80]

[PRE81]

[PRE82]

[PRE83]

[PRE84]

[PRE85]

[PRE86]

[PRE87]

在这个XAML中，我们有一个指令标签和一个用户可以按下的按钮。当按钮被按下时，点击事件会将语义焦点设置到`semanticFocusLabel`。以下是点击事件代码：

[PRE88]

[PRE89]

[PRE90]

[PRE91]

[PRE92]

以下代码使屏幕阅读器能够发出公告：

[PRE93]

[PRE94]

[PRE95]

另一个可访问性新增功能是自动字体缩放。

### 字体缩放

默认情况下，所有组件现在都具有自动字体缩放，并且默认启用。这意味着当您的用户在各个平台上更改文本缩放时，您的应用程序的文本将自动缩放到他们选择的设置。您可以使用以下标记关闭自动字体缩放：`FontAutoScalingEnabled="False"`。将属性更改为`True`或删除它将重新启用字体自动缩放。

## BlazorWebView

使用BlazorWebView，您可以在Microsoft MAUI应用程序中托管Blazor网站。这使得您的Blazor网站能够利用原生平台功能和各种用户控件。您可以将`BlazorWebView`添加到XAML页面，并将其指向Blazor应用程序的根：

[PRE96]

[PRE97]

[PRE98]

[PRE99]

[PRE100]

[PRE101]

[PRE102]

如您从XAML中看到的，我们Blazor应用程序的根是`wwwroot/index.html`。在下一节中，我们将探讨WinUI 3。

注意

截至2022年6月20日，MAUI已普遍可用，但为了开发MAUI应用程序，您需要安装.NET 2022预览版。

# 使用MAUI构建响应式UI

在本节中，我们将使用MAUI构建一个简单的响应式UI。在MAUI包含在Visual Studio 2022之前，您需要确保您使用Visual Studio 2022预览版：

1.  启动一个新的.NET MAUI应用程序，命名为`CH12_ResponsiveMAUI`。

1.  添加一个名为`Api`的新文件夹。

1.  在`Api`文件夹中，添加一个名为`PropertyChangedNotifier`的类，并用以下代码替换其内容：

    [PRE103]

此代码是一个实现`INotifyPropertyChanged`接口的基类。

1.  添加一个名为`Data`的新文件夹。

1.  在`Data`文件夹中添加一个名为`BaseEntity`的新类，并具有以下属性：

    [PRE104]

这些是我们实体的基本属性，将继承此类。

1.  在`Data`文件夹中添加一个名为`IRepository`的新接口，并用以下代码替换类：

    [PRE105]

此接口将由所有我们的存储库实现。

1.  在`Data`文件夹中添加一个名为`BaseRepository`的类，并用以下代码更新类：

    [PRE106]

此类是一个通用的基本存储库，实现了`IRepository`接口。存储数据上下文为`ICollection`类型，我们将`Context`设置为作为参数传入的集合。

1.  添加`Add`方法：

    [PRE107]

此代码将一个实体添加到我们的集合中。

1.  添加`Count`方法：

    [PRE108]

此代码返回我们集合中所有实体的数量。

1.  添加`Filter`方法：

    [PRE109]

此代码接收一个查询并返回一个过滤后的项目列表。

1.  添加`FilteredCount`方法：

    [PRE110]

此代码返回我们过滤列表中的项目。

1.  添加`FirstOrDefault`方法：

    [PRE111]

此方法返回与我们的查询匹配的第一条记录。如果没有匹配项，则返回默认值。

1.  添加`GetAll`方法：

    [PRE112]

此方法返回我们列表中的所有项。

1.  添加`GetById`方法：

    [PRE113]

此方法根据其ID号从列表中获取一个项。

1.  添加`Remove`方法：

    [PRE114]

此方法从集合中删除一个实体。

1.  添加`Update`方法：

    [PRE115]

此方法更新集合中的一个实体。

1.  在`Data`文件夹中添加一个名为`PeopleRepository`的新类，并按以下方式更新类定义：

    [PRE116]

此类创建了一个新的`Person`类型存储库。

1.  在`Data`文件夹中添加一个名为`Person`的新类。然后，按以下方式更新类：

    [PRE117]

此类继承我们的`BaseEntity`类并添加了`FirstName`和`LastName`属性。

1.  添加一个名为`ViewModels`的新文件夹和一个名为`ViewModelBase`的新类。按以下所示更新类定义：

    [PRE118]

此类是所有视图模型的基本视图模型类。它可以被转换为任何类型，并实现了`PropertyChangeNotifer`。

1.  添加`PeopleViewModel`：

    [PRE119]

此代码用人员填充我们的集合。

1.  在项目根目录中添加一个名为`SplashPage`的新页面：

    [PRE120]

我们的`SplashPage`是一个加载页面，将以进度条和标签的形式向用户显示进度。该类继承自`Content`页面并实现了`INotifyPropertyChanged`事件。我们有一个计时器，其回调是一个报告加载进度的方法。

1.  添加`ReportProgress`方法：

    [PRE121]

此方法停止计时器并运行代码以更新应用程序加载进度状态。它使用一个安全调用方法来更新启动屏幕。

1.  添加`LoadMainPage`方法：

    [PRE122]

此方法将应用程序的`MainPage`设置为`AppShell`，并传递一个类型为`BaseEntity`的参数。

1.  添加`SaveInvokeInMaInThread`方法：

    [PRE123]

此代码在主线程上执行安全调用以更新UI。该方法在调用设备对应的方法之前会检查应用程序正在运行的设备。

1.  添加`UpdateProgress`方法：

    [PRE124]

此方法更新进度条和标签。

1.  更新`SplashPage` XAML，如下所示：

    [PRE125]

此标记包含我们的UI定义，当它运行时将由代码更新。

1.  通过替换以下XAML来更新`MainPage`：

    [PRE126]

此代码通过添加人员表来更新原始源代码。

1.  添加`PeopleRepository`类变量，并更新`MainPage`类的构造函数，如下所示：

    [PRE127]

此代码通过将`MainPage`的`BindingContext`设置为`PeopleViewModel`来修改我们的`MainPage`。

1.  运行代码，你应该会看到以下屏幕：

![Figure 12.10 – 启动页面

![Figure 12.10 – B16617.jpg](img/Figure_12.10_B16617.jpg)

Figure 12.10 – 启动页面

下一个屏幕是你将看到的：

![Figure 12.11 – 带有滚动视图中的表格和响应点击按钮的主表单

![Figure 12.11 – B16617.jpg](img/Figure_12.11_B16617.jpg)

Figure 12.11 – 带有滚动视图中的表格和响应点击按钮的主表单

我们成功构建了一个响应式启动屏幕，它还会填充表格并响应用户点击按钮。这标志着我们对MAUI的探讨结束。现在我们将转向WinUI 3。

# 使用WinUI 3构建响应式UI

在本节中，我们将探讨如何在WinUI 3应用程序中执行长时间运行的操作时使用`ProgressRing`组件来提供用户反馈。当用户触发一个长时间运行的操作并阻止UI时，在操作完成之前提供用户反馈是一个好主意。让我们编写一个简单的应用程序，使用以下步骤来模拟长时间运行的操作：

1.  启动一个新的WinUI3应用程序，并将其命名为`CH12_ResponsiveWinUI3`。

1.  打开`MainWindow.xaml`并替换窗口标签之间的现有XAML，如下所示：

    [PRE128]

我们已经使用`OneWay`绑定将`ProgressRing`类的`IsActive`和`Visibility`属性绑定到`IsWorking`属性。

1.  在类的代码后面实现`INotifyPropertyChanged`接口。

1.  将以下成员添加到类中：

    [PRE129]

`_dispatcherTimer`将被用来模拟长时间运行的操作。`PropertyChanged`事件将被用来通知`ProgressRing`，`IsWorking`属性已更改，`_isWorking`变量将被更新，让`ProgressRing`知道是显示还是隐藏自己。

1.  如果`PropertyChanged`事件不是`null`，则添加一个方法来引发该事件：

    [PRE130]

当我们设置 `IsWorking` 属性时，我们调用此方法以引发 `PropertyChanged` 事件。

1.  在构造函数中添加以下三行代码：

    [PRE131]

这三行代码实例化了我们的 `DispatcherTimer`，将其间隔设置为 `10` 秒，并添加了 `Tick` 事件处理程序。

1.  我们现在将添加 `DispatcherTimer_Tick` 事件处理程序：

    [PRE132]

我们停止计时器并移除事件处理程序，以防止它再次触发并被保留在内存中。然后，我们将 `IsWorking` 属性设置为 `false`，这将导致 `ProgressRing` 被隐藏并变为不活动状态。然后，我们向 `MessageTextBlock` 添加一条消息。

1.  现在，添加 `IsWorking` 属性：

    [PRE133]

1.  当设置我们的属性时，我们调用 `NotifyPropertyChanged` 方法来引发 `PropertyChanged` 事件，让 `ProgressRing` 知道属性已更改。

1.  现在，添加按钮点击的代码：

    [PRE134]

我们折叠我们的按钮，因为它不再需要。将 `IsWorking` 属性设置为 `true`，并启动我们的 `DispatcherTimer`。

1.  运行代码。你应该看到一个显示 `ProgressRing` 的单个按钮，持续 10 秒。然后，`ProgressRing` 应该消失，并被文本 **工作完成** 替换。

现在我们已经结束了对响应式 UI 的探讨，让我们总结一下我们学到了什么。

# 摘要

在本章中，你学习了如何使用各种 UI 框架来使 UI 响应。首先，我们看了 WinForms。使用 WinForms，我们启用了 DPI 和长文件路径感知。我们还确保即使在运行长时间的后台任务时，我们也能在表格中翻页并执行其他 UI 操作，我们还添加了一个更新加载进度的启动画面。

使用 WPF，我们成功地创建了一个具有长时间运行任务（可以取消并显示进度指示）的窗口。它还有一个分页的数据表和按钮，当点击时，会更新点击计数标签。

然后，我们看了 ASP.NET 中的内存缓存和分布式缓存。我们还使用了 AJAX 来更新当前显示页面的部分，并研究了 WebSockets 和 SignalR。我们使用 SignalR 实现了一个实时 ASP.NET 聊天应用程序。

然后，我们继续研究 MAUI。特别是，我们研究了布局、可访问性和 `BlazorWebView`。最后，我们研究了 WinUI 3 以及如何在长时间运行的过程中提供用户反馈。

在下一章中，我们将探讨分布式系统。但首先，尝试回答下一节中的问题，然后进行一些进一步阅读，以增强你对响应式 UI 的了解。

# 问题

1.  你如何使 WinForms 应用程序在高清屏幕或普通 DPI 的大屏幕上正确缩放？

1.  你如何在 Windows 上处理长文件路径？

1.  当你的应用程序启动时间较长时，你如何保持用户的参与度？

1.  当你有一个长时间运行的过程正在运行时，你如何保持应用程序对用户输入的响应性？

1.  你可以使用哪些缓存方法来加速对资源的访问？

1.  你如何只加载网页的一部分？

1.  列出两个用于执行网络数据传输和实时网络通信的框架？

1.  列出MAUI中可用的三种无障碍方法。

1.  如何将现有的Blazor Web应用包含在MAUI项目中？

1.  当您的应用程序已经加载，并且用户启动一个长时间运行的操作时，您可以使用哪些控件来提供用户反馈，以便用户不会认为您的WinUI 3应用程序已崩溃？

# 进一步阅读

+   *哪个更好？WebSockets还是SignalR*: [https://dotnetplaybook.com/which-is-best-websockets-or-signalr/](https://dotnetplaybook.com/which-is-best-websockets-or-signalr/ )

+   *为什么SignalR/messagepack比gRPC/protobuf快两倍？*: [https://github.com/grpc/grpc-dotnet/issues/812](https://github.com/grpc/grpc-dotnet/issues/812 )

+   *教程：开始使用ASP.NET Core SignalR*: [https://docs.microsoft.com/aspnet/core/tutorials/signalr?view=aspnetcore-5.0&tabs=visual-studio](https://docs.microsoft.com/aspnet/core/tutorials/signalr?view=aspnetcore-5.0&tabs=visual-studio )

+   *WebSocket*: [https://javascript.info/websocket](https://javascript.info/websocket)

+   *将您的应用从Xamarin.Forms迁移*: https://docs.microsoft.com/dotnet/maui/get-started/migrate

+   *Xamarin.Forms Made Easy*: [https://winstongubantes.blogspot.com/2018/09/backgrounding-with-xamarinforms-easy-way.html](https://winstongubantes.blogspot.com/2018/09/backgrounding-with-xamarinforms-easy-way.html)

+   *Xamarin – 与线程一起工作*: [https://lukealderton.com/blog/posts/2016/october/xamarin-forms-working-with-threads/](https://lukealderton.com/blog/posts/2016/october/xamarin-forms-working-with-threads/ )

+   在Windows上创建Android模拟器: [https://docs.microsoft.com/xamarin/android/get-started/installation/android-emulator/device-manager?tabs=windows&pivots=windows](https://docs.microsoft.com/xamarin/android/get-started/installation/android-emulator/device-manager?tabs=windows&pivots=windows)

+   安装Microsoft OpenJDK: [https://docs.microsoft.com/xamarin/android/get-started/installation/openjdk](https://docs.microsoft.com/xamarin/android/get-started/installation/openjdk )

+   *为VS 2022的单项目MSIX打包工具*: https://marketplace.visualstudio.com/items?itemName=ProjectReunion.MicrosoftSingleProjectMSIXPackagingToolsDev17

+   *使用Blazor组件虚拟化提高渲染性能*: [https://www.daveabrock.com/2020/10/20/blazor-component-virtualization/#:~:text=Improve%20rendering%20performance%20with%20Blazor%20component%20virtualization%20Use,the%20entire%20HTML%20tree%20loads%20from%20the%20server](https://www.daveabrock.com/2020/10/20/blazor-component-virtualization/#:~:text=Improve%20rendering%20performance%20with%20Blazor%20component%20virtualization%20Use,the%20entire%20HTML%20tree%20loads%20from%20the%20server).

+   *如何在 .NET MAUI 中重用 Xamarin.Forms 自定义渲染器*: [https://www.syncfusion.com/blogs/post/how-to-reuse-xamarin-forms-custom-renderers-in-net-maui.aspx](https://www.syncfusion.com/blogs/post/how-to-reuse-xamarin-forms-custom-renderers-in-net-maui.aspx)

+   *宣布 .NET MAUI 预览版 7*: [https://devblogs.microsoft.com/dotnet/announcing-net-maui-preview-7/](https://devblogs.microsoft.com/dotnet/announcing-net-maui-preview-7/)

+   *.NET 多平台应用程序用户界面*: [https://dotnet.microsoft.com/en-us/apps/maui](https://dotnet.microsoft.com/en-us/apps/maui)
