# 第五章：在 Azure Service Fabric 上创建微服务

本章涉及令人兴奋的微服务世界和**Azure Service Fabric**。在本章中，我们将涵盖以下食谱：

+   下载和安装 Service Fabric

+   使用无状态 actor 服务创建 Service Fabric 应用程序

+   使用 Service Fabric Explorer

# 简介

传统上，开发者以单体方式编写应用程序。这意味着一个单一的可执行文件，通过类等组件被分割。单体应用程序需要大量的测试，并且由于单体应用程序的庞大，部署过程繁琐。即使你有多个开发团队，他们也需要对整个应用程序有一个稳固的理解。

微服务是一种旨在解决单体应用程序和传统应用程序开发方式问题的技术。使用微服务，你可以将应用程序分解成更小的部分（服务），这些服务可以独立运行，不依赖于其他任何服务。这些较小的服务可以是无状态的或是有状态的，在功能规模上也比较小，这使得它们更容易开发、测试和部署。你还可以独立于其他服务对每个微服务进行版本控制。如果一个微服务比其他微服务接收更多的负载，你可以仅将该服务扩展以满足其需求。对于单体应用程序，你必须尝试扩展整个应用程序以满足应用程序中单个组件的需求。

以一个流行的在线网店为例，其运作可能包括购物车、购物者资料、订单管理、后端登录、库存管理、计费、退货等更多功能。传统上，会创建一个单独的 Web 应用程序来提供所有这些服务。使用微服务，你可以将每个服务隔离为独立的、自包含的功能和代码库。你也可以指派一个开发团队专注于网店的一个部分。如果这个团队负责库存管理微服务，他们将处理其所有方面。例如，这意味着从编写代码和增强功能到测试和部署的每一个环节。

微服务的另一个优秀副作用是它允许你轻松隔离可能遇到的任何故障。最后，你还可以使用任何你想要的任何技术（C#、Java、VB.NET）创建微服务，因为它们是语言无关的。

Azure Service Fabric 允许您轻松扩展微服务并提高应用程序的可用性，因为它实现了故障转移。当与 Fabric 一起使用时，微服务成为一项非常强大的技术。将 Azure Service Fabric 视为**平台即服务**（**PaaS**）解决方案，您的微服务位于其上。我们称微服务所在的集合为 Service Fabric 集群。每个微服务都位于一个虚拟机上，在 Service Fabric 集群中被称为节点。这个 Service Fabric 集群可以存在于云中或本地机器上。如果某个节点因任何原因变得不可用，Service Fabric 集群将自动将微服务重新分配到其他节点，以确保应用程序保持可用。

最后，关于有状态和无状态微服务的区别，这里有一句话要说。您可以将微服务创建为有状态或无状态。当一个微服务依赖于外部数据存储来持久化数据时，它本质上是无状态的。这仅仅意味着微服务不内部维护其状态。另一方面，有状态的微服务通过在它所在的本地服务器上存储来维护其状态。

# 下载和安装 Service Fabric

在您能够创建和测试 Service Fabric 应用程序之前，您必须在您的 PC 上安装和设置一个本地 Service Fabric 集群。

## 准备工作

我们将从 Azure 网站下载并安装**软件开发工具包**（**SDK**），这将允许我们在您的本地开发机器上创建一个本地 Service Fabric 集群。

## 如何操作…

1.  从 Microsoft Azure 网站下载 SDK，并通过 Service Fabric 学习路径访问其他资源，例如文档，请访问[`azure.microsoft.com/en-us/documentation/learning-paths/service-fabric/`](https://azure.microsoft.com/en-us/documentation/learning-paths/service-fabric/)：![如何操作…](img/B05391_05_01.jpg)

1.  在安装开始之前，您需要接受许可条款：![如何操作…](img/B05391_05_02.jpg)

1.  然后 Web 平台安装程序开始下载 Microsoft Azure Service Fabric 运行时。请允许此过程完成：![如何操作…](img/B05391_05_03.jpg)

1.  下载完成后，安装过程将开始：![如何操作…](img/B05391_05_04.jpg)

1.  安装完成后，以下产品将被安装，这也在下面的屏幕截图中显而易见：

    +   Microsoft Azure Service Fabric 运行时

    +   Microsoft Azure Service Fabric Core SDK 预览版

    +   Microsoft Azure Service Fabric Visual Studio 2015 工具预览版

    +   Microsoft Azure Service Fabric SDK 预览版

    ![如何操作…](img/B05391_05_05.jpg)

1.  下一个任务是作为管理员打开 PowerShell。在 Windows 10 开始菜单中输入单词 `PowerShell`，搜索将立即返回桌面应用程序作为结果。右键单击桌面应用程序，并在上下文菜单中选择 **以管理员身份运行**：![如何操作…](img/B05391_05_06.jpg)

1.  一旦 Windows PowerShell 打开，运行命令 `Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force -Scope CurrentUser`。这样做的原因是 Service Fabric 使用 PowerShell 脚本创建本地开发集群。它还用于部署 Visual Studio 开发的应用程序。运行此命令可以防止 Windows 阻止这些脚本：![如何操作…](img/B05391_05_07.jpg)

1.  接下来，创建本地 Service Fabric 集群。输入命令 `& "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\ClusterSetup\DevClusterSetup.ps1"`。

    这将创建托管 Service Fabric 应用程序所需的本地集群：

    ![如何操作…](img/B05391_05_08.jpg)

1.  集群创建后，PowerShell 将启动服务：![如何操作…](img/B05391_05_09.jpg)

1.  此过程可能需要几分钟。请确保让它完成：![如何操作…](img/B05391_05_10.jpg)

1.  一旦命名服务就绪，您可以关闭 PowerShell：![如何操作…](img/B05391_05_11.jpg)

1.  要查看创建的集群，您可以在本地计算机上导航到 `http://localhost:19080/Explorer`。

    这将为您提供集群的健康状况和状态的快照。它还将显示集群中运行的应用程序：

    ![如何操作…](img/B05391_05_12.jpg)

## 它是如何工作的…

如您所见，Service Fabric 集群对于创建和运行在 Visual Studio 中创建的应用程序至关重要。这将允许我们在将它们发布到云之前，直接在您的本地计算机上测试应用程序。

# 创建具有无状态演员服务的状态化 Service Fabric 应用程序

作为本章介绍的组成部分，我们探讨了有状态和无状态微服务的区别。可用的 Service Fabric 应用程序模板进一步分为 **可靠服务**（有状态/无状态）和 **可靠演员**（有状态/无状态）。何时使用哪一个将取决于您应用程序的具体业务需求。

简单来说，如果您想创建一个应该向任何时间点上的许多应用程序用户公开的服务，一个可靠服务可能是一个不错的选择。想象一下，一个公开最新汇率的服务，可以被许多用户或应用程序同时消费。

再次，回顾本章的引言部分，我们使用了在线网店和购物车的例子。可靠的演员可以很好地适应每个购买商品的客户，因此你可以有一个购物车演员。作为服务框架框架的一部分，可靠的演员基于虚拟演员模式。查看有关虚拟演员模式的文章，请访问[`research.microsoft.com/en-us/projects/orleans/`](http://research.microsoft.com/en-us/projects/orleans/)。

为了展示使用无状态演员服务作为示例创建微服务是多么容易，我们将使用 Visual Studio 将服务发布到服务框架集群，并从控制台（客户端）应用程序调用该服务。

## 准备工作

要完成这个配方，你必须确保你在本地机器上安装了你的本地服务框架集群。

## 如何做…

1.  在 Visual Studio 中，通过转到**文件** | **新建** | **项目**来创建一个新的项目。![如何做…](img/B05391_05_13.jpg)

1.  从**Visual C#**节点展开节点，直到你看到**云**节点。当你点击它时，你会看到 Visual Studio 现在列出了一个名为**服务框架应用程序**的新模板。选择**服务框架应用程序**模板，命名为`sfApp`，然后点击**确定**。![如何做…](img/B05391_05_14.jpg)

1.  接下来，从弹出的**创建服务**窗口中选择**无状态可靠** **演员**。我们将其命名为`UtilitiesActor`：![如何做…](img/B05391_05_15.jpg)

1.  一旦你的解决方案创建完成，你会注意到它由三个项目组成。这些是：

    +   `sfApp`

    +   `UtilitiesActor`

    +   `UtilitiesActor.Interfaces`

    ![如何做…](img/B05391_05_22.jpg)

1.  我们将首先修改`IUtilitiesActor`接口。这个接口将简单地要求`UtilitiesActor`实现一个名为`ValidateEmailAsync`的方法，该方法接受一个电子邮件地址作为参数，并返回一个布尔值，表示它是否是一个有效的电子邮件地址：

    ```cs
    namespace UtilitiesActor.Interfaces
    {
        public interface IUtilitiesActor : IActor
        {
            Task<bool> ValidateEmailAsync(string emailToValidate);
        }
    }
    ```

1.  接下来，打开你的`UtilitiesActor`项目并查看类。它将用红色波浪线标记，因为它没有实现接口成员`ValidateEmailAsync()`：![如何做…](img/B05391_05_16.jpg)

1.  使用*Ctrl* + *.*（句号），实现接口。删除所有其他不必要的默认代码（如果有）：![如何做…](img/B05391_05_17.jpg)

1.  为您插入的实现接口代码应如下所示。目前，它只包含`NotImplementedException`。这就是我们将实现代码以验证电子邮件地址的地方：

    ```cs
    namespace UtilitiesActor
    {
        internal class UtilitiesActor : StatelessActor, IUtilitiesActor
        {
            public Task<bool> ValidateEmailAsync(string emailToValidate)
            {
                throw new NotImplementedException();
            }        
        }
    }
    ```

1.  我们将使用正则表达式来验证通过参数传递给此方法的电子邮件地址。正则表达式非常强大。然而，在我的编程生涯中，我从未自己编写过表达式。这些在互联网上很容易找到，并且您可以为您的项目创建一个实用工具类（或扩展方法类）以供重用。您可以使用正则表达式和其他常用代码。

    最后，您将注意到 `ActorEventSource` 代码。这只是为了创建 **Windows 事件跟踪** (**ETW**) 事件，这些事件将帮助您从 Visual Studio 的诊断事件窗口中查看应用程序中的情况。要打开诊断事件窗口，请转到 **视图**，**其他窗口**，然后单击 **诊断事件查看器**：

    ```cs
    internal class UtilitiesActor : StatelessActor, IUtilitiesActor
    {
        public async Task<bool> ValidateEmailAsync(string emailToValidate)
        {
            ActorEventSource.Current.ActorMessage(this, "Email Validation");

            return await Task.FromResult(Regex.IsMatch(emailToValidate, @"\A(?:[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.[a-z0- 9!#$%&'*+/=?^_`{|}~-]+)*@(?:a-z0-9?\.)+a-z0-9?)\Z", RegexOptions.IgnoreCase));
        }        
    }
    ```

1.  一定要添加对 `System.Text.RegularExpressions` 命名空间的引用。没有它，您将无法使用正则表达式。如果您在代码中添加了正则表达式而没有添加引用，Visual Studio 将在 `Regex` 方法下显示一条红色的波浪线：![如何操作…](img/B05391_05_18.jpg)

1.  使用 *Ctrl* + *.* (句点)，将 `using` 语句添加到您的项目中。这将把正则表达式命名空间引入作用域：![如何操作…](img/B05391_05_19.jpg)

1.  现在我们已经创建了接口，并添加了该接口的实现，是时候添加一个我们将用于测试的客户端应用程序了。在您的解决方案上右键单击，然后添加一个新项目：![如何操作…](img/B05391_05_20.jpg)

1.  最简单的方法是添加一个简单的控制台应用程序。将您的客户端应用程序命名为 `sfApp.Client` 并单击 **确定** 按钮：![如何操作…](img/B05391_05_21.jpg)

1.  在您将控制台应用程序添加到解决方案后，您的解决方案应如下所示：![如何操作…](img/B05391_05_23.jpg)

1.  现在，您需要添加对您的客户端应用程序的引用。在 `sfApp.Client` 项目的 **引用** 节点处右键单击，并从上下文菜单中选择 **添加引用**：![如何操作…](img/B05391_05_24.jpg)

1.  首先，添加对 `UtilitiesActor.Interfaces` 项目的引用：![如何操作…](img/B05391_05_25.jpg)

1.  您还需要添加对几个 Service Fabric **动态链接库** (**DLL**) 的引用。当您创建 Service Fabric 应用程序时，它应该在项目文件夹结构中添加一个名为 `packages` 的文件夹。浏览到该文件夹，并从那里添加您的 Service Fabric DLL。在添加了所需的 DLL 之后，您的项目应如下所示：![如何操作…](img/B05391_05_29.jpg)

1.  在您的控制台应用程序的 `Program.cs` 文件中，您需要在 `Main` 方法中添加以下代码：

    ```cs
    namespace sfApp.Client
    {
        class Program
        {
            static void Main(string[] args)
            {
                var actProxy = ActorProxy.Create<IUtilitiesActor> (ActorId.NewId(), "fabric:/sfApp");

                WriteLine("Utilities Actor {0} - Valid Email?: {1}", actProxy.GetActorId(), actProxy.ValidateEmailAsync ("validemail@gmail.com").Result);
                WriteLine("Utilities Actor {0} - Valid Email?: {1}", actProxy.GetActorId(), actProxy.ValidateEmailAsync ("invalid@email@gmail.com").Result);
                ReadLine();
            }
        }
    }
    ```

    我们所做的一切只是为我们的演员创建一个代理，并将电子邮件验证的输出写入控制台窗口。您的客户端应用程序现在已准备就绪。

1.  在我们能够运行客户端应用程序之前，然而，我们首先需要发布我们的服务。在**解决方案资源管理器**中，右键单击`sfApp`服务，然后从上下文菜单中选择**发布**：![如何操作…](img/B05391_05_26.jpg)

1.  现在，将显示**发布 Service Fabric 应用程序**窗口。单击**连接端点**文本框旁边的**选择…**按钮：![如何操作…](img/B05391_05_27.jpg)

1.  将**连接端点**选择为**本地集群**，然后单击**确定**：![如何操作…](img/B05391_05_28.jpg)

1.  将**目标配置文件**和**应用程序参数文件**更改为`Local.xml`。完成后，单击**发布**按钮：![如何操作…](img/B05391_05_30.jpg)

1.  如果你导航到`http://localhost:19080/Explorer`，你会注意到你创建的服务已经发布到你的本地 Service Fabric 集群中：![如何操作…](img/B05391_05_31.jpg)

1.  现在，你已准备好运行你的客户端应用程序。右键单击`sfApp.Client`项目，然后从上下文菜单中选择**调试**和**启动新实例**：![如何操作…](img/B05391_05_32.jpg)

1.  控制台应用程序调用验证方法来检查电子邮件地址，并将结果显示在控制台窗口中。结果符合预期：![如何操作…](img/B05391_05_33.jpg)

1.  然而，在创建 actor ID 时，我们可以更加具体。我们可以给它一个特定的名称。修改你的代理代码，并创建一个新的`ActorId`方法，并给它任何字符串值：

    ```cs
    var actProxy = ActorProxy.Create<IUtilitiesActor>(new ActorId("Utilities"), "fabric:/sfApp");

    WriteLine("Utilities Actor {0} - Valid Email?: {1}", actProxy.GetActorId(), actProxy.ValidateEmailAsync("validemail@gmail.com").Result) ;
    WriteLine("Utilities Actor {0} - Valid Email?: {1}", actProxy.GetActorId(), actProxy.ValidateEmailAsync("invalid@email@gmail.com").Resu lt);
    ReadLine();
    ```

    ### 注意

    `ActorId`方法可以接受类型为`Guid`、`long`或`string`的参数。

1.  当你再次调试你的客户端应用程序时，你会注意到`Utilities Actor`现在有一个逻辑名称（当你创建新的`ActorId`方法时传递的字符串值相同的名称）：![如何操作…](img/B05391_05_34.jpg)

## 它是如何工作的…

创建你的 Service Fabric 应用程序并在本地发布是一个在将其发布到云之前测试应用程序的完美解决方案。创建小型独立的微服务可以让开发者获得许多与测试、调试和部署高效且健壮的代码相关的优势，这些代码可以让你的应用程序利用以确保最大可用性。

# 使用 Service Fabric Explorer

你还可以使用另一个工具来可视化 Service Fabric 集群。这是一个独立的工具，你可以通过导航到本地安装路径`%Program Files%\Microsoft SDKs\Service Fabric\Tools\ServiceFabricExplorer`并单击`ServiceFabricExplorer.exe`来找到它。当你运行应用程序时，它将自动连接到你的本地 Service Fabric 集群。它可以显示有关集群中的应用程序、集群节点、应用程序和节点的健康状态以及集群中应用程序的任何负载的丰富信息。

## 准备工作

您必须已经在本机完成了 Service Fabric 的安装，以便 Service Fabric Explorer 能够正常工作。如果您还没有这样做，请遵循本章中的 *下载和安装 Service Fabric* 菜单。

## 如何操作…

1.  当您启动 Service Fabric Explorer 时，将出现以下窗口：![如何操作…](img/B05391_05_35.jpg)

1.  注意左侧的树视图显示 **应用视图** 和 **节点视图**：![如何操作…](img/B05391_05_36.jpg)

1.  右侧面板将显示有关本地集群的信息。这使得您很容易看到本地服务集群的整体健康状况：![如何操作…](img/B05391_05_37.jpg)

1.  当您展开 **应用视图** 时，您会注意到我们的 `sfApp` 服务已经发布。进一步展开，您会看到 `sfApp` 服务已经在 **Node.2** 上发布。展开 **节点视图** 和 **Node.2** 以查看在该节点上活动的服务：![如何操作…](img/B05391_05_38.jpg)

1.  为了说明微服务的可伸缩性，右键单击 **Node.2**，然后在上下文菜单中停止该节点。然后，点击窗口顶部的刷新按钮以刷新节点和应用。

1.  如果您现在继续展开 **应用视图** 并再次查看服务，您会注意到 Service Fabric 集群已经注意到 **Node.2** 已关闭。然后它自动将服务推送到一个新的、健康的节点（在本例中为 **Node.5**）：![如何操作…](img/B05391_05_40.jpg)

1.  Service Fabric Explorer 右侧面板中的本地集群节点视图也报告说 **Node.2** 已关闭：![如何操作…](img/B05391_05_41.jpg)

## 工作原理…

Service Fabric Explorer 将允许您查看所选节点的信息，并且您将能够深入查看有关 Service Fabric 集群应用的丰富信息。
