# 前言

欢迎阅读《C# 7 和.NET Core 2.0 蓝图》。通过采用*蓝图*方法来展示.NET Core 2.0 的强大之处，您将学习如何在创建可用的令人兴奋的应用程序时使用.NET Core 2.0。

# 这本书适合谁

本书旨在面向那些对 C#编程语言有很好掌握但可能需要更多了解.NET Core 的开发人员。

# 本书涵盖的内容

第一章，*电子书管理和目录应用*，介绍了 C# 7 引入的新功能，使开发人员能够编写更少的代码并提高生产力。我们将创建一个电子书管理应用程序。如果你和我一样，在硬盘和外部驱动器上都散落着电子书，这个应用程序将提供一个机制将所有这些不同的位置汇聚到一个虚拟存储空间中。该应用程序已经具备功能，但可以进一步增强以满足您的需求。

第二章，*板球比分计算器和跟踪器*，指出面向对象编程（OOP）是编写.NET 应用程序的关键要素。适当的 OOP 确保开发人员可以轻松地在项目之间共享代码。在本章中，我们将创建一个 ASP.NET Bootstrap Web 应用程序，用于跟踪您两支最喜欢的板球队的比分。也正是通过这个应用程序，面向对象编程的原则将变得明显。

第三章，*跨平台.NET Core 系统信息管理器*，介绍了.NET Core 是什么；.NET Core 允许我们创建在 Windows、macOS 和 Linux 上运行的应用程序。为了在本章中加以说明，我们将创建一个简单的信息仪表板应用程序，显示我们正在运行的计算机的信息以及该计算机位置的天气情况。

第四章，*使用 MongoDB 的任务错误记录 ASP .NET Core MVC 应用程序*，通过创建一个任务/错误记录应用程序，介绍了在 ASP.NET Core MVC 中使用 MongoDB。MongoDB 可以让开发人员更加高效，并且可以轻松地添加到.NET Core 中。

第五章，*ASP.NET SignalR 聊天应用程序*，开始让你想象具有服务器端代码实时推送数据到网页的能力，而无需用户刷新页面。ASP.NET SignalR 库为开发人员提供了一种简化的方法，以向应用程序添加实时网络功能。当阅读第八章，*使用 OAuth 的 Twitter 克隆*时，请记住这一点。这是一个完美的应用程序，可以集成 SignalR。

第六章，*使用 Entity Framework Core 的 Web 研究工具*，讨论了 Entity Framework Core，这是我们.NET Core 教育中的一个重要组成部分。开发应用程序中最令人沮丧的部分之一是尝试建立代码与数据库之间的通信层。Entity Framework Core 可以轻松解决这个问题，并且本章向您展示了如何实现。

第七章，*无服务器电子邮件验证 Azure 函数*，向您展示如何创建 Azure 函数以及如何从 ASP.NET Core MVC 应用程序调用该函数。Azure 函数将只验证电子邮件地址。本章介绍了无服务器计算，并在阅读本章时将清楚地了解其好处。

第八章，*使用 OAuth 创建 Twitter 克隆*，表达了我有时希望能够调整 Twitter 以满足自己的需求，例如保存喜爱的推文。在本章中，我们将看看使用 ASP.NET Core MVC 创建基本 Twitter 克隆有多容易。然后，您可以轻松地向应用程序添加功能，以定制满足您特定需求。

第九章，*使用 Docker 和 ASP.NET Core*，探讨了当今非常流行的 Docker，以及其非常重要的原因。本章说明了 Docker 如何使开发人员受益。我还将向您展示如何创建 ASP.NET Core MVC 应用程序并在 Docker 容器中运行它。在本章的最后部分，我们将看到如何使用 Docker Hub 和 GitHub 设置自动构建。

# 充分利用本书

假设您至少对 C# 6.0 有很好的理解。本书中的所有示例将在相关的地方使用 C# 7。

您需要安装最新补丁的 Visual Studio 2017。如果您没有 Visual Studio 2017，可以免费从[`www.visualstudio.com/downloads/`](https://www.visualstudio.com/downloads/)安装 Visual Studio Community 2017。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

您可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的解压缩软件解压缩文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/CSharp7-and-.NET-Core-2.0-Blueprints`](https://github.com/PacktPublishing/CSharp7-and-.NET-Core-2.0-Blueprints)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有其他代码包，来自我们丰富的图书和视频目录，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。例如：“您可以随意命名应用程序，但我将我的称为`eBookManager`。”

代码块设置如下：

```cs
namespace eBookManager.Engine 
{ 
    public class DeweyDecimal 
    { 
        public string ComputerScience { get; set; } = "000"; 
        public string DataProcessing { get; set; } = "004"; 
        public string ComputerProgramming { get; set; } = "005"; 
    } 
} 
```

任何命令行输入或输出都是这样写的：

```cs
    mongod -dbpath D:MongoTask 
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会在文本中以这种方式出现。例如：“在添加了所有存储空间和电子书之后，您将看到列出的虚拟存储空间。”

警告或重要说明会出现在这样。

提示和技巧会出现在这样。
