# 第十一章. 在 Visual Studio 中创建移动应用程序

Visual Studio 是**集成开发环境**（**IDE**）中的佼佼者。这一点毫无疑问。作为开发者，你可以通过为广泛的平台创建应用程序来变得非常灵活。其中之一就是移动开发。开发者开始创建移动应用程序，但不想使用不同的 IDE。在 Visual Studio 2015 中，你不必这样做。它将允许你创建 Android 和（现在通过**Xamarin**）iOS 应用程序。因此，本章将探讨以下概念：

+   安装 Xamarin 和其他必需组件

+   使用 Apache Cordova 创建 Android Visual Studio 项目

+   使用 Xamarin Forms 创建 iOS 应用程序

# 简介

如果你还没有听说过 Xamarin，我们鼓励你通过 Google 搜索这个工具。传统上，开发者需要使用**Xcode**或**NetBeans**来创建 iOS 和 Android 应用程序。对于开发者来说，这意味着需要学习一门新的编程语言。例如，如果你创建了一个你希望部署到 iOS、Android 和 Windows 的应用程序，你需要了解 Objective-C 或 Swift、Java 和.NET 语言。

这也给开发带来了额外的挑战，因为它意味着需要维护多个代码库。如果需要在应用程序的 Windows 版本中进行更改，也必须在 iOS 和 Android 代码库中进行更改。有时公司会为每个平台管理不同的开发团队。你可以想象在多个平台上的多个团队之间管理更改的复杂性。这尤其适用于你处理的是大型代码库。

Xamarin 通过允许.NET 开发者使用标准的.NET 库，在 Visual Studio 中创建 iOS 和 Android 应用程序来解决这一问题。作为.NET 开发者，你现在可以使用你已有的技能来完成这项任务。简而言之，你会为你的应用程序创建一个共享库，然后为不同的平台创建不同的外观。第二个选项是使用 Xamarin Forms 来创建一个 Visual Studio 项目，并针对所有三个平台。这使得开发者针对多个平台变得非常容易。

# 安装 Xamarin 和其他必需组件

Xamarin 可以在自定义 Visual Studio 安装过程中安装。现在，让我们假设 Xamarin 尚未安装，并且在你安装 Visual Studio 之后需要现在安装它。

## 准备工作

如果你想针对 iOS 进行开发，需要注意的一点是你将需要使用 Mac 来构建你的 iOS 应用程序。

## 如何操作…

1.  在**控制面板**中，点击**程序和功能**。右键单击你的 Visual Studio 安装，然后点击**更改**：![如何操作…](img/B05391_11_01.jpg)

1.  这将显示 Visual Studio 安装程序。在这里，您可以随意添加和删除组件来修改您当前的 Visual Studio 安装。请注意，我们已选择**C#/.NET (Xamarin v4.0.3)**和**HTML/JavaScript (Apache Cordova) Update 8.1**进行安装。如果您对使用 Xamarin 没有兴趣，则可以不安装 Xamarin 组件，只保留 Apache Cordova 选项选中。这将仍然允许您使用 Apache Cordova 而不是 Xamarin 来创建 Android 应用程序。同样，如果您对 Apache Cordova 没有兴趣，只想使用 Visual Studio 创建 Android 应用程序和 iOS 应用程序，请选择安装 Xamarin 组件。其余的安装过程很简单：![如何操作…](img/B05391_11_02.jpg)

1.  如果我们想使用 Xamarin 来针对 iOS 应用程序，我们还需要采取第二个步骤。我们必须在 Mac 上安装所需的软件。在您的 Mac 上打开 Xamarin 的网站。网址是[`www.xamarin.com/`](https://www.xamarin.com/)。点击**产品**下拉菜单，从列表中选择**Xamarin Platform**：![如何操作…](img/B05391_11_03.jpg)

1.  您也可以通过访问[`www.xamarin.com/platform`](https://www.xamarin.com/platform)来获取所需页面。点击**免费下载**按钮将在您的 Mac 上安装一个名为 Xamarin Studio 的程序。您需要知道，当安装在 Mac 上时，Xamarin Studio 无法创建 Windows 应用程序。它只会允许您在 Mac 上创建 iOS 和 Android 应用程序。除了 Xamarin Studio，您还将获得 Xamarin Mac Agent（之前称为 Xamarin Build Host）。这是一个必需的组件，以便您可以将您的 PC 连接到您的 Mac 以构建您的 iOS 应用程序。最后，PC 和 Mac 也必须能够通过网络相互连接（关于这一点稍后会有更多说明）：![如何操作…](img/B05391_11_04.jpg)

1.  在 Mac 上下载安装程序后，安装过程很简单。只需按照屏幕提示完成安装：![如何操作…](img/B05391_11_05.jpg)

## 它是如何工作的…

我们之前在安装 Xamarin 和 Apache Cordova 时采取的步骤将使我们能够做到以下事情：

+   **安装 Apache Cordova**：如果您只想针对 Android、iOS 和 Windows，但不想使用 Xamarin

+   **安装 Xamarin**：如果您想针对 Android、iOS、Windows 或三者都进行目标定位，并使用单个解决方案来实现

Visual Studio 非常灵活，为开发者提供了广泛的选择。

# 使用 Apache Cordova 创建 Android Visual Studio 项目

使用 Apache Cordova 创建 Android 应用程序非常简单。然而，这个配方只会向您展示如何开始。

## 准备工作

在 Visual Studio 设置过程中，您需要将 Apache Cordova 作为自定义安装选项的一部分进行安装。要了解如何操作，请参阅本章中的*安装 Xamarin 和其他必需组件*配方。

## 如何操作…

1.  在**新建项目**对话框屏幕中，选择**Apache Cordova 应用程序**，并将**空白应用（Apache Cordova）**作为要使用的模板。选择项目位置，然后点击**确定**按钮：![如何操作……](img/B05391_11_06.jpg)

1.  一旦 Visual Studio 创建了你的应用程序，你会注意到它有一个非常具体的结构。从项目来看，你会注意到你可以针对 Android、iOS、Windows 或 Windows Phone 8.1。这是你将用来创建 Android 应用程序的框架：![如何操作……](img/B05391_11_07.jpg)

1.  当你准备好调试时，你可以从**调试**菜单中选择一个模拟器。这将部署你的应用程序到所选模拟器，并允许你测试你的应用程序：![如何操作……](img/B05391_11_08.jpg)

## 它是如何工作的……

能够使用 Visual Studio 从一个解决方案中针对不同的移动设备，这给了开发者自由去实验，找到最适合他们和发展风格的解决方案。

# 使用 Xamarin Forms 创建 iOS 应用程序

许多开发者都想尝试编写 iOS 应用程序。一直以来，最大的缺点就是学习一门新的编程语言和新的 IDE。对一些人来说，这可能不是问题，因为他们想学习新东西。但对许多.NET 开发者来说，能够坚持使用他们熟悉的 IDE 和编程语言是非常有利的。嗯，这正是 Xamarin Forms 和 Visual Studio 所实现的。它为.NET 开发者提供了使用 Visual Studio 编写可以在多个平台上轻松运行的应用程序的能力，而不需要为每个平台编写单独的代码库。

## 准备工作

你需要有一台运行 OS X 的 Mac。你只需要这个来调试 iOS 应用程序。

## 如何操作……

1.  在 Visual Studio 2015 中创建一个新的项目。从已安装的模板中选择**跨平台**，然后选择**空白应用（Xamarin.Forms.Portable）**。这将使我们能够创建一个跨平台的应用程序，而不是针对单一平台（例如 Android 或 iOS）：![如何操作……](img/B05391_11_09.jpg)

1.  项目创建可能需要几分钟才能完成。在这个过程中，你可能会看到一个消息告诉你 Windows 10（假设你在运行 Windows 10）的**开发者模式**尚未启用：![如何操作……](img/B05391_11_10.jpg)

1.  启用这个功能很简单。你可以在弹出的消息中点击**开发者设置**链接，或者你可以在 Windows 10 **设置**中的**查找设置**搜索框中输入`开发者模式`：![如何操作……](img/B05391_11_11.jpg)

1.  点击**开发者模式**选项将显示**使用开发者功能**确认对话框。只需点击**是**继续：![如何操作……](img/B05391_11_12.jpg)

1.  项目创建完成后，你将看到一个**开始使用 Xamarin.Forms**的屏幕：![如何操作……](img/B05391_11_13.jpg)

1.  查看你的**解决方案资源管理器**，你会注意到已经创建了几个项目。我们只关注 iOS 项目：![如何操作…](img/B05391_11_14.jpg)

1.  查看调试目标时，你会注意到，当你将目标更改为 Droid，例如，Android 项目被设置为启动项目。如果你将其设置为 iOS，也会发生相同的情况：![如何操作…](img/B05391_11_15.jpg)

1.  目前为止，在你开始调试 iOS 应用程序之前，你需要将 Visual Studio 连接到 Mac 上的 Xamarin Mac Agent。在 Visual Studio 中，将鼠标悬停在 iOS 工具栏上的**Xamarin Mac Agent**按钮上。它将显示为未连接：![如何操作…](img/B05391_11_16.jpg)

    ### 注意

    参考本章前面的*安装 Xamarin 和其他必需组件*部分，了解如何安装 Xamarin Mac Agent。

1.  要连接到 Xamarin Mac Agent，点击此按钮。将显示**Xamarin Mac Agent 说明**窗口。你可以按照此屏幕上的说明操作，说明如下：![如何操作…](img/B05391_11_17.jpg)

1.  在你的 Mac 上，打开**系统偏好设置**。查找并点击**共享**图标：![如何操作…](img/B05391_11_18.jpg)

1.  这将显示**共享**窗口。从左侧菜单中选择**远程登录**，然后在**仅这些用户**下选择或添加你当前 Mac 用户到这个列表中：![如何操作…](img/B05391_11_19.jpg)

1.  当你将你的当前 Mac 用户添加到**远程登录**列表中后，点击后退按钮返回到上一个屏幕。然后查找并点击**网络**：![如何操作…](img/B05391_11_18.jpg)

1.  这将打开**网络**屏幕。查看显示当前状态为**已连接**的地方。下面，你会看到一个 IP 地址。记下显示的 IP 地址，因为你将需要使用它将 Visual Studio 连接到 Xamarin Mac Agent：![如何操作…](img/B05391_11_20.jpg)

    ### 注意

    只要注意，我已经故意在截图上隐藏了我的 IP 地址。

1.  在 Windows 中，在 Visual Studio 中点击**确定**以关闭**Xamarin Mac Agent 说明**屏幕。现在，**Xamarin Mac Agent**屏幕将可见。在此屏幕底部，点击**添加 Mac…**按钮：![如何操作…](img/B05391_11_21.jpg)

1.  这将显示**添加 Mac**屏幕，在这里你需要输入从 Mac 的**系统偏好设置**中的**网络**屏幕上记下的 IP 地址。点击**添加**按钮：![如何操作…](img/B05391_11_22.jpg)

    ### 注意

    只要注意，我已经故意在截图上隐藏了我的 IP 地址。

1.  现在，你将被要求提供之前在**远程登录**屏幕上添加的 Mac 用户的用户名和密码。点击**登录**按钮：![如何操作…](img/B05391_11_23.jpg)

    ### 注意

    只要注意，我已经故意在截图上隐藏了 IP 地址和 GUID。

1.  点击**登录**后，你应该会自动从 Visual Studio 连接到你的 Xamarin Mac Agent：![如何操作…](img/B05391_11_24.jpg)

1.  你现在可以选中你想要调试的 iOS 设备。正如你所见，有各种各样的 iOS 设备可供选择：![如何操作…](img/B05391_11_25.jpg)

1.  为了本菜谱的目的，我们只是选择了一个**iPhone 4S iOS 9.3**。点击**调试**按钮以启动应用：![如何操作…](img/B05391_11_26.jpg)

1.  这将现在构建你的应用程序，并通过网络连接将信息发送到 Xamarin Mac 代理。然后它将在你的 Mac 上启动模拟器。第一次这样做时，启动模拟器可能需要几分钟，但一旦完成，后续的调试会快得多：![如何操作…](img/B05391_11_27.jpg)

1.  在 Mac 上启动模拟器后，Xamarin 应用程序将被启动：![如何操作…](img/B05391_11_28.jpg)

1.  当 Xamarin 启动画面关闭时，你会看到**欢迎使用 Xamarin Forms！**文本：![如何操作…](img/B05391_11_29.jpg)

1.  在 Visual Studio 中返回，停止调试。你会注意到应用在 Mac 的模拟器中关闭，并且调试在 Visual Studio 中停止。然而，模拟器在你的 Mac 上仍然保持打开状态。

    现在让我们更改一些文本。查看你的 Visual Studio 解决方案中的可移植项目。这是所有其他解决方案中项目将使用的共享项目。在可移植项目中，点击 `App.cs` 文件：

    ![如何操作…](img/B05391_11_30.jpg)

1.  默认代码显示。在这里，你可以看到我们在之前调试的应用程序中看到的 `Welcome to Xamarin Forms!` 文本：![如何操作…](img/B05391_11_31.jpg)

1.  将代码更改为如下所示。我们正在做的是添加日期和时间。这里有几个需要注意的地方：

    +   我们在这里使用标准的 .NET `DateTime` 库。

    +   我们使用字符串插值来创建在表单上显示的文本：

        ```cs
        MainPage = new ContentPage
        {
            Content = new StackLayout
            {
                VerticalOptions = LayoutOptions.Center,
                Children = {
                    new Label { HorizontalTextAlignment = TextAlignment.Center,
                        Text = $"Welcome to Xamarin Forms! The date is {DateTime.Now}"
                    }
                }
            }
        };
        ```

1.  完成这些后，再次调试你的应用程序。当模拟器在 Mac 上显示你的 iOS 应用程序时，你会看到日期和时间被显示出来：![如何操作…](img/B05391_11_32.jpg)

## 它是如何工作的…

需要注意的一点是，我们在这里做的并没有比在其他任何标准 .NET 应用程序中做的更多。我们正在编写 C# 代码并将其编译以在 iOS 操作系统上运行。我们也可以轻松地将应用程序更改为在任何 iOS 设备上调试。我们不需要学习 Objective-C 或 Swift（尽管 Swift 是一种很棒的语言，值得学习）。我们也不需要熟悉新的 IDE（用于开发 iOS 和 Mac 应用的 Xcode）。我们不需要调整任何约束，修改任何游乐场元素，或学习如何使用任何新的控件。Xamarin Forms 和 Visual Studio 会为我们处理所有这些。最好的是，Xamarin 现在免费与 Visual Studio 一起使用。没有理由你不应该尝试编写 iOS 应用程序。
