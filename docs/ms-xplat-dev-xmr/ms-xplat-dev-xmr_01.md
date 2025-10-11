# 第一章. 使用 Xamarin 进行开发

本章探讨了 Xamarin 框架和架构在不同目标平台上的情况，并确定了差异和相似之处。它还包括有关为 Xamarin 准备开发环境的入门信息和技巧，并涵盖了 Xamarin 开发的一些基本要素。本章分为以下部分：

+   使用 Xamarin 的跨平台项目

+   目标平台

+   设置开发环境

+   模拟器选项

+   典型的 Xamarin 解决方案结构

+   跨平台开发的质量

# 使用 Xamarin 的跨平台项目

开发者正在享受一个新时代，在这个时代中，开发不再局限于单一的应用程序平台，而是跨越各种媒体，如手机、平板电脑、个人电脑，甚至可穿戴设备。开发项目之间的共享代码和资源提高了工作的优雅性和质量。此外，健壮性、维护多平台应用程序所需的努力以及可重用模块之间存在直接相关性。

通用应用程序是一个以前用来识别针对运行 iOS 操作系统（iPhone 和 iPad）的设备的应用程序的术语。然而，现在这个术语也用来描述 Windows Runtime 应用程序（Windows Store 和 Windows Phone 8.1 - WinRT）以及针对手机和平板电脑的 Android 应用程序。随着 Xamarin 的发布，一个真正的通用应用程序概念诞生了。当考虑 Xamarin 应用程序时，术语“通用”指的是在所有三个平台上运行并适应系统资源的应用程序。

在这个通用应用程序的背景下，开发者现在发现很难获取所有三个平台上的常见任务的必要解决方案。此外，将每个平台视为一个独立的项目进行开发会导致开发者的时间浪费，尽管这种应用程序的主要驱动因素，即数据、业务逻辑和 UI，在所有平台上在概念上几乎是相同的。

Xamarin 平台的开发策略和模式，其中一些在本书的其余部分有所描述，试图解决一些这些问题，并为开发者提供生产跨平台、可管理和高质量产品的工具和策略。

## Xamarin 作为一个平台

Xamarin 最初诞生于一个社区努力，旨在将.NET 库和通用语言运行时编译器移植到不同的操作系统。最初的尝试旨在创建一组二进制文件，用于在 Unix 平台上的 C#（.NET 的本土语言）编写应用程序的开发、编译和运行。这个项目，Mono，后来被移植到许多其他操作系统，包括 iOS（Mono-Touch）和 Android（Mono for Android）。

Xamarin 开发平台的兴起创造了一个新的开发细分市场，即在同一时间针对三个不同的平台创建产品，同时允许用户将现有的.NET 开发技能适应这些新平台，并为更广泛的设备和操作系统生成应用程序。

### 注意

自从早期阶段开始，微软一直是 Xamarin 平台和工具集的强力支持者。正如你将在本章剩余部分看到的那样，Xamarin 工具已完全集成到 Visual Studio 中，并最终包含在 Visual Studio 2015 的设置中。这种合作关系一直持续到 Xamarin 最终被微软收购，这一消息于 2016 年 3 月公开宣布。

Xamarin 为每个提到的平台都提供了编译器，以便在.NET 框架（类似）中编写的代码编译成原生应用程序。这个过程提供了高度高效的应用程序，与解释型移动 HTML 应用程序大相径庭。

除了原生编译，Xamarin 还提供了对强类型平台特定功能的访问。这些功能以健壮的方式使用，在编译时绑定到底层平台。平台特定的执行也可以通过原生调用进行扩展，这是通过互操作库实现的。

## 作为一款产品，Xamarin

作为一套开发工具，Xamarin 有不同的版本。具有不同知识和经验集合的开发者可以使用这些工具根据他们的需求设置自己的开发环境。Xamarin 开发环境可以在不同的操作系统上配置。然而，目前还不能在同一个操作系统上为所有三个平台进行开发。

对于希望使用熟悉的 Visual Studio 界面并利用现有技能的开发者，Xamarin 的 Visual Studio 扩展提供了合适的选项。一旦安装了扩展，环境就准备好开发 Android 和 Windows Phone 应用程序。这个扩展让开发者能够充分利用 Visual Studio，其中包括这两个平台的设计师。为了开发 iOS 应用程序，你需要通过所谓的 Visual Studio 与 Apple OS X 构建机器的配对过程。构建机器反过来用于在开发环境（Visual Studio）中可视化故事板、编译 iOS 代码和调试应用程序。

第二种选择是使用 Xamarin Studio。Xamarin Studio 是一个完整的 IDE，它具有一些你从 Visual Studio 熟悉的特性，例如智能代码补全（IntelliSense）、代码分析和代码格式化。如果你在 Apple OS X 上运行 Xamarin Studio，你可以使用这个 IDE 为 Android 和 iOS 平台开发。然而，在 Windows 上使用 Xamarin Studio 时，你只能针对 Android 平台。

这个开发套件的一个重要部分是实时监控工具，称为 Xamarin Insights。Xamarin Insights 允许开发者监控他们的实时应用程序，以帮助检测和诊断性能问题和异常，并了解应用程序的使用情况。Xamarin Insights 还可以连接到其他应用程序，例如，应用程序错误可以直接推送到错误跟踪系统。

# 目标平台

如前所述，Xamarin 创建了一个新的平台，其中开发工作针对多个操作系统和各种设备。最重要的是，编译的应用程序不是运行解释序列，而是拥有原生代码库（如 Xamarin.iOS）或集成的.NET 应用程序运行时（如 Xamarin.Android）。本质上，Xamarin 用编译的二进制文件和执行上下文替换了.NET 应用程序的公共语言运行时（CLR）和 IL，所谓的**mono 运行时**。

## Xamarin on Android

在 Android 应用程序中，mono 运行时直接放置在 Linux 内核之上。这创建了一个与 Android 运行时的并行执行上下文。然后，Xamarin 代码被编译成 IL，并由 mono 运行时访问。另一方面，Android 运行时通过所谓的**托管可调用包装器**（**MCW**）访问，这是一个在两个运行时之间的打包包装器。MCW 层负责将托管类型转换为 Android 运行时类型，并在执行时间调用 Android 代码。每当.NET 代码需要调用 Java 代码时，都会使用这个 JNI（Java 互操作）桥。MCW 提供了一系列应用，包括继承 Java 类型、重写方法和实现 Java 接口。

下面的图像显示了 Xamarin.Android 架构：

![Xamarin on Android](img/B04693_01_01.jpg)

图 1：Xamarin.Android 架构

`Android.*`和`Java.*`命名空间在 MCWs 中用于访问 Android 运行时和 Java API 中特定于设备和平台的功能，例如音频、图形、OpenGL 和电话等功能。

使用互操作库，也可以在 Xamarin.Android 的执行上下文中加载原生库并执行原生代码。在这种情况下，反向回调执行通过**Android Callable Wrappers**（**ACW**）处理。ACW 是一个 JNI 桥，允许 Android 运行时访问.NET 域。对于每个直接或间接与 Java 类型（继承自`Java.Lang.Object`的类型）相关的托管类，在编译时生成一个 ACW。

## Xamarin on iOS

在 iOS 应用程序中，根据 iOS SDK 协议，使用集成的并行运行时是不允许的（遗憾的是）。根据 iOS SDK 协议，只有当所有脚本和代码都由 Apple 的 WebKit 框架下载和运行时，才能使用解释代码。

在此限制条件下，开发者仍然可以在.NET 平台上开发应用程序并在其他三个平台上共享代码。在编译时，项目首先被编译成 IL 代码，然后（使用 Mono Touch Ahead-Of-Time 编译器—mtouch）编译成静态原生 iOS 位代码。这意味着使用 Xamarin 开发的 iOS 应用程序是完全的原生应用程序。

![Xamarin on iOS](img/B04693_01_02.jpg)

图 2：Xamarin.iOS 编译

与 Xamarin.Android 类似，Xamarin.iOS 也包含一个互操作引擎，它将.NET 世界与 Objective-C 世界连接起来。通过这个桥梁，在`ObjCRuntime`命名空间下，用户能够访问 iOS 基于 C 的 API，以及使用 Foundation 命名空间，并且可以使用和从原生类型派生，以及访问 Objective-C 属性。例如，Objective-C 类型如`NSObject`、`NSString`和`NSArray`在 C#中暴露并提供对底层类型的绑定。这些类型可以作为内存引用或作为强类型对象使用。这提高了开发体验，并增加了类型安全性。

这种静态编译是使用 Windows 平台上的 Xamarin 开发 iOS 应用程序时使用构建机器的主要原因。因此，在 Xamarin.iOS 中没有反向回调功能，即从.NET 代码到原生运行时的调用得到支持，但原生代码回.NET 域的调用则不支持。由于 Xamarin.iOS 应用程序的编译方式，还有其他一些功能被禁用。例如，不允许泛型类型从`NSObject`继承。另一个重要的限制是，在运行时不允许创建动态类型，这反过来又禁用了在 Xamarin.iOS 应用程序中使用动态关键字。

### 注意

与其他平台相比，如果 Xamarin.iOS 应用程序包是在调试配置中构建的，它们的大小比它们的发布版本要大得多。这些包被工具化了，但没有被链接器优化。在 Xamarin.iOS 应用程序中不允许对这些包进行性能分析。

与 Xamarin.Android 开发类似，使用 Xamarin.iOS，也可以从托管代码中重用原生代码和库。为此，Xamarin 提供了一个名为**绑定库**的项目模板。绑定库帮助开发者创建原生 Objective-C 代码的托管包装器。

## Windows Runtime 应用程序

尽管 Xamarin 没有将 Windows Runtime 作为目标平台，也没有为它提供专门的工具（除了 Xamarin.Forms 之外），但涉及 Xamarin 的跨平台项目可以，并且通常包括 Windows Runtime 项目。由于.NET 和 C#是 Windows Runtime 的本地语言，大多数共享项目（如可移植库、共享项目和 Xamarin.Forms 项目）可以在 Windows Runtime 中重用，无需进一步修改。

使用 Windows Runtime，开发者可以创建 Windows Phone 8.1 和 Windows Store 应用程序。Windows Phone 8 和 Windows Phone 8.1 Silverlight 也可以作为目标并包含在 PCL 描述中。

# 设置开发环境

Xamarin 项目可以在各种开发环境中进行。由于此类项目涉及多个平台，操作系统、IDE 选择和配置都是准备过程中的关键部分。

### 注意

环境设置不仅取决于目标应用平台，还取决于 Xamarin 许可证。不同许可选项和定价信息可以在 Xamarin 网站上找到（[`store.xamarin.com/`](https://store.xamarin.com/))。

## 选择合适的开发操作系统

Android 应用程序可以使用安装了 Xamarin 扩展的 Xamarin Studio 和 Visual Studio 在 Windows 上进行开发和编译，也可以在安装了 Xamarin Studio for Mac 的 Apple OS X 操作系统上进行。

对于 iOS 应用程序开发，无论是在 Windows 上的 Visual Studio 还是 Apple OS X 上的 Xamarin Studio，都需要一台至少运行 OS X Mountain Lion 的 Apple Macintosh 计算机。构建机器应安装 Xcode 开发工具和 iOS SDK，以及安装了 Xamarin.iOS 套件。

另一方面，Windows Store 应用程序只能在 Windows 平台上进行开发。

|   | Apple OS X | Microsoft Windows |
| --- | --- | --- |
|   | **Xamarin Studio** | **Xamarin Studio** | **Visual Studio** |
| **iOS 应用** | 是 |   | 是（带 OS X 构建机器） |
| **Android 应用** | 是 | 是 | 是 |
| **Windows Store 应用** |   |   | 是 |

> *图 3：OS X 和 Windows 上的开发 IDE*

在虚拟化方面，开发者也受到限制。根据最终用户协议，OS X 不能安装在非 Apple 品牌的机器上并运行，也不能进行虚拟化。另一方面，你可以在 OS X 开发机器上设置一个虚拟机，用于 Microsoft Windows 和 Visual Studio。然而，在这种情况下，系统应运行嵌套虚拟化以运行 Windows Phone 和 Android 模拟器的 Visual Studio for Windows。尽管 Parallels 和 VMWare Fusion 支持嵌套虚拟化，但 Microsoft 不支持嵌套 Hyper-V，因此此类机器可能不稳定。

### Xamarin Studio 设置和配置

Xamarin Studio 可以在 Windows 和 OS X 操作系统上设置。开发者可以从 [www.xamarin.com](http://www.xamarin.com) 下载它并遵循安装说明。对于目标平台（例如，Xamarin.iOS、Xamarin.Android 等）以及这些平台的依赖项（例如，Android SDK）应下载并安装到开发机器上。对于 OS X，必须单独安装并配置的一个必需组件是带有 Xcode 开发环境的 iOS SDK。

![Xamarin Studio 设置和配置](img/B04693_01_03.jpg)

图 4：Mavericks（OS X 10.9）上的 Xamarin 设置

在 Microsoft Windows 上，重要的是要提到，Xamarin Studio 仅支持 Android 应用程序的开发。Windows Phone 或 iOS 应用程序（即使是远程构建机器）的项目都不能在 Windows 上的 Xamarin Studio 中查看、修改或编译。

![Xamarin Studio 设置和配置](img/B04693_01_04.jpg)

图 5：在 OS X 上设置 Xamarin 开发环境

在 OS X 上开发时，与 iOS 和 Android 一起开发 Windows Phone 应用程序的唯一选项是使用 Windows 虚拟机，并使 Visual Studio 与 Xamarin Studio 并行运行。这种设置对于使用 Team Foundation Server 作为源控制的开发者也有帮助，因为他们可以使用 Visual Studio 客户端提供的增强集成，而不是独立的 TFS Everywhere。还可以设置，使操作系统主机机可以与 Visual Studio 配对，成为 iOS 应用程序的构建主机。

### Visual Studio 设置和配置

对于 Xamarin 项目，典型的 Windows 开发平台配置包括 Visual Studio 2013 或 2015、Apple OS X 构建主机以及 Hyper-V 和/或 VirtualBox，以便能够使用 Android 和 Windows Phone 模拟器。Xamarin.iOS 应用程序随后在 Apple OS X 构建主机上编译和模拟。

![Visual Studio 设置和配置](img/B04693_01_05.jpg)

图 6：Windows 平台 Xamarin 开发环境

### 注意

尽管在 Microsoft Windows 环境中使用虚拟机运行 OS X 在技术上可行，但 Apple 的许可协议不允许这样做：

*"2.H. 其他使用限制：本许可证中规定的授予并不允许您，您同意不，在任何非 Apple 品牌的计算机上安装、使用或运行 Apple 软件，或允许他人这样做。"*

在 Microsoft Windows 上，Xamarin 的安装与在 Apple OS X 上的 Xamarin Studio 设置类似。所有 Xamarin 开发的先决条件都包含在 Xamarin for Windows 软件包中，以及 Visual Studio 扩展程序。

![Visual Studio 设置和配置](img/B04693_01_06.jpg)

图 7：Visual Studio 2015 设置

OS X 和 Microsoft Windows 之间的一个主要区别是，Visual Studio 2015 现在包括跨平台开发工具，如 Android SDK、开发套件和 Xamarin 项目模板。因此，Xamarin 的安装仅负责安装请求平台的扩展（即 Xamarin.iOS 和/或 Xamarin.Android）。

为了使用 Visual Studio 开发和测试 iOS 应用，以及可视化并编辑故事板，必须将 Apple OS X 机器连接到 Visual Studio 作为构建主机。Xamarin 4.0 引入了 Xamarin Mac Agent 的概念，这是在 OS X 机器上运行的后台进程，它为 Visual Studio 提供所需的 SSH 连接（通过端口 22 的安全连接）。在 Xamarin 4.0 之前，构建主机机器需要运行所谓的 Mac **构建主机**，该主机用于将 Mac 主机与 Visual Studio 配对。Xamarin Mac Agent 的唯一先决条件是在 Windows 工作站和 OS X 构建主机上安装了 Xamarin.iOS，并且构建主机为当前用户启用了远程登录。在 Visual Studio 中，**查找 Xamarin Mac Agent** 对话框有助于建立远程连接。

![Visual Studio 设置和配置](img/b04693_01_07.jpg)

图 8：Xamarin.iOS 构建主机

需要记住的是，与 Visual Studio 配对的 Mac 机器必须安装带有 iOS SDK 的 Xcode。还必须在 Xcode 的账户配置部分添加一个开发者账户（无论是否已注册到应用开发者计划）。

### 注意

如果与 Xcode 关联的账户没有为开发者计划支付订阅，iOS 项目的平台只能设置为模拟器和调试选择中的一个模拟器选项，而不是实际设备。否则，用户将看到一个错误消息，例如，**在密钥链中找不到有效的 iOS 签名密钥**。

# 模拟器选项

对于目标平台和开发环境，有许多编译的 Xamarin 项目的模拟器。开发者在使用 Android 平台的模拟器时最具灵活性，而 iOS 和 Windows 商店应用的选项仅限于 SDK 提供的模拟器。

## Android 模拟器

Android 应用可以在 Microsoft Windows 和 Apple OS X 操作系统上的多个模拟器上运行和测试。

Android SDK 随带默认的模拟器，该模拟器安装在开发机器上。这种仿真选项在 OS X 和 Windows 操作系统上均可用。

![Android 模拟器](img/B04693_01_09.jpg)

图 9：AVD 和 Genymotion 模拟器

此 Android 模拟器使用 **Android 虚拟设备**（**AVD**）来模拟 Linux 内核和 Android 运行时。它不需要任何额外的虚拟化软件来运行，然而，缺乏虚拟化支持使得 AVD 的响应速度大大降低，并且启动时间相对较长。它还提供了广泛的仿真选项供开发者使用，从短信和电话到硬件、外围设备和电源事件。

Genymotion 模拟器([`www.genymotion.com/`](https://www.genymotion.com/))是 Xamarin 和 Android 开发者最受欢迎的模拟选项之一。尽管它提供免费许可证，但免费版本仅允许 GPS 和摄像头模拟。Genymotion 模拟器在 VirtualBox 虚拟化软件上运行（并与其一起安装）。

### 小贴士

**VirtualBox 与 Hyper-V 一起使用**

VirtualBox 软件不能与 Hyper-V 虚拟化软件同时运行，后者是 Windows Phone 开发和 Windows 操作系统上模拟所必需的。为了使用 Windows Phone 模拟器和 Genymotion Android 模拟器，您可以在 Windows 启动时创建一个双启动选项来禁用和启用 Hyper-V。

```cs
bcdedit /copy {current} /d "No Hyper-V"
bcdedit /set {<identifier from previous command>} hypervisorlaunchtype off

```

这将创建一个第二个启动选项，以便在没有 Hyper-V 功能的情况下启动 Windows，这样虚拟化就可以由 VirtualBox 使用。

最后也是最新的 Android 模拟选项是 Visual Studio Android 模拟器。这个 Android 模拟器在 Hyper-V 上运行，并为开发者提供各种设备 API 版本和模拟选项。

![Android 模拟器](img/B04693_01_10.jpg)

图 10：Visual Studio Android 模拟器

Visual Studio Android 模拟器是 Visual Studio 2015 安装的一部分，也可以稍后作为扩展安装。该模拟器提供了与 Windows Phone 模拟器相似的经验，并允许开发者和测试人员使用几乎相同的模拟选项，包括不同的设备配置文件以及不同的 API 级别。

## iOS 模拟

iOS 模拟仅适用于 Xcode 工具和 iOS SDK。iOS 模拟器可以直接在开发 Xamarin Studio 时在 Apple OS X 上启动，或者通过将构建机器与在 Microsoft Windows 上运行的 Visual Studio Xamarin 扩展配对来启动。它还可以用来测试 iPhone 和 iPad 应用程序。

# 典型的 Xamarin 解决方案结构

一个 Xamarin 解决方案可以由不同类型的项目组成。其中一些项目是特定平台的项目，而其他项目是共享项目类型或模块，这使得跨平台重用代码成为可能。

![典型的 Xamarin 解决方案结构](img/B04693_01_11.jpg)

图 11：Visual Studio 和 Xamarin Studio 上的 Xamarin 项目解决方案结构

## 便携式类库

便携式类库是跨平台项目之间共享代码最常见的方式。PCLs 提供了一套公共参考程序集，使得.NET 库和二进制文件可以在任何基于.NET 的运行时或与 Xamarin 编译器中使用——从手机到客户端，再到服务器和云。PCL 模块设计为仅使用.NET 框架的特定子集，并且可以设置为针对不同的平台。

![便携式类库](img/B04693_01_12.jpg)

图 12：便携式类库目标

微软为每个目标组合指定了一个名称，每个配置文件也都有一个 NuGet 目标。通过 NuGet 发布了适用于可移植类库的 .NET 库子集，这是在 Visual Studio 2013 发布时实现的。这使得开发者能够通过 NuGet 包发布他们的工作，针对广泛的移动平台（有关更多信息，请参阅 *NuGet 包* 部分）。

### 注意

目前首选的配置文件和 Xamarin 项目的最大公约数子集是所谓的 Profile 259。微软对该配置文件的支持指定为 .NET 可移植子集 (.NET Framework 4.5, Windows 8, Windows Phone 8.1, Windows Phone Silverlight 8)，而 NuGet 目标框架配置文件是 `portable-net45+netcore45+wpa81+wp8`。

在创建 PCL 时，最大的缺点是项目不能包含或引用任何平台特定代码。通常，通过抽象平台特定需求或使用依赖注入或类似方法在平台特定项目中引入实现来解决这个限制。

例如，在下面的设备特定外设示例中，公共可移植类库有一个接受两个单独接口的构造函数，这些接口可以通过依赖注入容器注入，或者可以通过设备特定的实现进行初始化。作为回报，公共库创建了一个业务逻辑实现，如下所示：

```cs
namespace Master.Xamarin.Portable
{
    public class MyPhotoViewer
    {
        private readonly IStorageManager m_StorageManager;

        private readonly ICameraManager m_CameraManager;
        public MyPhotoViewer(IStorageManager storageManager, ICameraManager cameraManager)
        {
            m_StorageManager = storageManager;
            m_CameraManager = cameraManager;
        }

        public async Task TakePhotoAsync()
        {
            var photoFileIdentifier = await m_CameraManager.TakePhotoAndStoreAsync();

            var photoData = await m_StorageManager.RetrieveFileAsync(photoFileIdentifier);

            // TODO: Do something with the photo buffer
        }
    }

    /// <summary>
    /// Should be implemented in Platform Specific Library
    /// </summary>
    public interface IStorageManager
    {
        Task<string> StoreFileAsync(byte[] buffer);

        Task<byte[]> RetrieveFileAsync(string fileIdentifier);
    }

    /// <summary>
    /// Should be implemented in Platform Specific Library
    /// </summary>
    public interface ICameraManager
    {
        Task<string> TakePhotoAndStoreAsync();
    }
}
```

## 共享项目

“共享项目”这个术语最初是由微软团队在发布适用于 Windows Phone 和 Windows Runtime 的通用应用时提出的（即 Visual Studio 2013）。随着 Xamarin 的到来，共享项目也可以被 Android 和 iOS 项目引用。这类项目本质上是对共享代码和资源文件的包装或容器，这些代码和资源文件链接到多个项目和平台。共享文件资源将在引用项目中稍后包含，并作为这些模块的一部分进行编译。

![共享项目](img/B04693_01_13.jpg)

图 13：共享项目

在使用共享项目时，开发者应小心包含平台特定代码，因为共享元素将被包含在所有引用项目中并编译。可以在共享项目中引入编译器指令（例如，`#if __ANDROID__`）来表示代码的某些部分仅适用于特定平台。

### 小贴士

**在共享项目中可视化平台特定代码**

使用 Visual Studio (2013 或更高版本) 时，可以根据条件编译常量的组合可视化不同的执行路径。

![共享项目](img/B04693_01_14.jpg)

图 14：Visual Studio 共享项目编辑器

Visual Studio 在编辑器窗口的右上角提供了一个下拉菜单，用于确定引用共享项目的平台特定项目。通过选择项目，你可以看到根据目标平台禁用的代码部分。

如果我们使用相同的示例来拍照，我们需要为同一操作创建两个完全不同的实现，如下所示：

```cs
private async Task<string> TakePhotoAsync()
{
    string resultingFilePath = "";

    var fileName = String.Format("testPhoto_{0}.jpg", Guid.NewGuid());

#if __ANDROID__

    Intent intent = new Intent(MediaStore.ActionImageCapture);
    var file = new File(m_Directory, fileName);
    intent.PutExtra(MediaStore.ExtraOutput, Net.Uri.FromFile(_file));

    // TODO: Need an event handler with TaskCompletionSource for
    // Intent's result
    m_CurrentActivity.StartActivityForResult(intent, 0);

    resultingFile = file.AbsolutePath;

#elif WINDOWS_PHONE_APP

    ImageEncodingProperties imgFormat = ImageEncodingProperties.CreateJpeg();

    // create storage file in local app storage   
    var file = await LocalStore.CreateFileAsync(fileName);

    resultingFilePath = file.Path;

    // take photo   
    await capture.CapturePhotoToStorageFileAsync(imgFormat, file);

#endif

    return resultingFile;
}
```

## Xamarin.Forms

Xamarin.Forms 是用于创建目标平台 UI 实现的统一库，这些 UI 实现将使用原生控件进行渲染。Xamarin.Forms 项目通常作为 PCL 项目创建，并且可以被 Xamarin.iOS、Xamarin.Android 和 Windows Phone 开发项目引用。Xamarin.Forms 组件也可以包含在共享项目中，并可以利用平台特定的功能。

开发者可以使用这些表单有效地创建常见的 UI 实现，无论是声明式（使用 XAML），还是通过使用提供的 API。这些由 Xamarin.Forms 组件构建的视图，在运行时使用特定平台的控件进行渲染。

开发项目可以通过创建数据访问模型直到 UI 组件的共享实现来实现，从而将平台之间的共享代码量提高到 90% 或更多。

## NuGet 包

NuGet 最初是微软的一个开源倡议，旨在在开发者之间共享代码，现在已经成为一个更大的生态系统。虽然 NuGet 服务器可以用作开源库共享平台，但许多开发团队将 NuGet 用作私有公司仓库，用于存储编译后的库。

NuGet 打包是适用于 Xamarin 项目的可行代码共享和重用策略，因为它得到了 Xamarin Studio 和 Visual Studio（在安装 Visual Studio 2012 之后无需进一步安装）的支持。

Xamarin 项目的 NuGet 目标框架名称为 mono，还有进一步的分组，例如 MonoAndroid10，它指的是目标框架为 MonoAndroid 1.0 或更高版本的项目。其他平台目标包括：

+   MonoAndroid: Xamarin.Android

+   Xamarin.iOS: Xamarin.iOS 统一 API（支持 64 位）

+   Xamarin.Mac: Xamarin.Mac 的移动配置文件

+   MonoTouch: iOS 经典 API

开发者可以自由地重用公开可用的 NuGet 包或创建自己的仓库来存储编译包，以便包含在 Xamarin 项目中。

### 小贴士

**在 Visual Studio 2015 中创建 NuGet 包**

随着 Visual Studio 2015 的发布，有一个新的项目模板可以帮助开发者创建和重用 NuGet 包。

![NuGet 包](img/B04693_01_15.jpg)

图 15：Visual Studio NuGet 包项目模板

关于创建 NuGet 包和发布的更多信息可以在 NuGet 文档中心找到：([`docs.nuget.org/create/creating-and-publishing-a-package`](http://docs.nuget.org/create/creating-and-publishing-a-package))

## 组件

组件是另一种在 Xamarin 项目中重用编译好的库和模块的方法。组件存储库集成在 Xamarin Studio 和 Visual Studio 中，并且自 2013 年发布以来已经收集了来自开发者的许多可重用提交。组件可以通过使用 Xamarin 组件存储库以与 NuGet 包相同的方式下载并安装到项目中。Xamarin 组件存储库可以在[`components.xamarin.com`](https://components.xamarin.com)找到。

# 跨平台开发中的质量

一些开发术语有助于开发者在为多个平台开发时创建健壮、可维护、高质量的代码。这些代码描述符帮助开发团队识别架构问题、软件问题和随机错误。

## 可重用性

*"在整个项目中可以重用多少代码？"*

可重用性是跨平台开发项目中关键的质量标识之一。Xamarin，特别是随着 Xamarin.Forms 的发布，为开发者提供了丰富的资源来创建平台无关的组件，这可以减少冗余并减少复杂项目中的开发时间。由 Visual Studio 生成的代码质量矩阵和单元测试覆盖率结果可以将这个描述符转化为可衡量的度量。

## 抽象

*"共享组件对平台了解多少？"*

在跨平台解决方案中，几乎不可避免地要包含特定平台的代码片段。这些模块抽象化的程度提高了共享组件的鲁棒性，并且与实现逻辑与底层平台耦合的松紧程度密切相关。通过这种方式，共享组件可以很容易地使用模拟或伪造的库进行测试，而无需创建特定平台的测试框架。单元测试代码覆盖率结果有助于确定应用程序的可测试性。

## 松耦合

*"将项目移植到另一个平台有多容易？"*

在特定平台的抽象实现之上，一个自主的共享实现层创建了灵活的解决方案，这些解决方案可以轻松地适应其他平台。减少共享逻辑对底层平台的依赖不仅本质上增加了可重用性，也提高了开发项目的敏捷性。共享项目中针对底层平台的条件编译块或`if`或`else`循环的数量确定了根据平台执行的代码量。

## 原生性

*"你的应用程序有多少程度与平台融合在一起？"*

尽管使用 Xamarin 开发时的最终目标是创建一个可以轻松编译到多个目标的应用程序，但使用 Xamarin 创建的应用程序应该看起来、感觉和表现如同为该特定平台设计。在创建共同基础的同时，应尊重每个平台的 UI 范式和用户交互机制。与上述代码描述相比，原生性更多的是一个名义上的和主观的衡量标准。

# 摘要

在本章中，我们讨论了 Xamarin 开发套件的一些关键特性，以及在这些先前描述的平台上的开发，并探讨了开发移动应用时 Xamarin 的基本要素。接下来的章节将参考这些关键特性以及平台之间的差异，以识别创建 Xamarin 跨平台应用的有价值模式和策略。

我们还讨论了目标平台的架构概述以及在这些平台上如何开发和编译 Xamarin 应用程序。这些平台之间最重要的区别是，Xamarin.Android（以及 Windows Phone）使用.NET 二进制文件和 Mono（以及.NET）运行时来执行代码，而 Xamarin.iOS 应用程序具有完全不同的设置和双编译（编译时提前）来利用.NET 二进制文件，但不是直接运行它们。

在使用 Xamarin 为 Android 和 iOS 平台开发时，开发者还被迫在不同的操作系统平台和开发 IDE 之间做出选择。开发环境的选取和配置取决于目标平台。IDE 功能、模拟器和仿真器选项在此选择中扮演着重要角色。虽然 OS X 操作系统与 Xamarin Studio 提供了一个熟悉的界面，并允许开发者转移他们的.NET 相关技能和知识，但目前它们对于开发 iOS 应用来说是一个更可行的选择。

另一个重要的更新是 Xamarin 解决方案结构。我们讨论了开发者如何在不同的平台之间共享代码，以及如何重用公共或私有存储库来包含共享模块。共享项目是大多数跨平台开发模式和策略的基础，与可移植类库一起。

总体而言，当使用 Xamarin 规范和功能时，开发者的主要目标应该是创建松散耦合、平台无关的模块，以提高生产力和提升跨平台开发项目的质量。
