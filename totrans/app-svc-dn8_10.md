# 10

# 使用 Azure 函数构建无服务器纳米服务

在本章中，你将了解 Azure 函数，它可以配置为在执行时仅需要服务器端资源。它们在触发活动时执行，例如向队列发送消息或将文件上传到 Azure 存储时，或者在定期的时间间隔内。

本章将涵盖以下主题：

+   理解 Azure 函数

+   构建 Azure 函数项目

+   响应定时器和资源触发器

+   将 Azure 函数项目发布到云端

+   清理 Azure 函数资源

# 理解 Azure 函数

**Azure 函数**是一个事件驱动的无服务器计算平台。您可以在本地构建和调试，然后部署到微软 Azure 云。Azure 函数可以使用多种语言实现，而不仅仅是 C# 和 .NET。它支持 Visual Studio 2022 和 Visual Studio Code 的扩展，以及一个命令行工具。

但首先，你可能想知道，“没有服务器如何提供服务？”

*无服务器*字面上并不意味着没有服务器。无服务器真正意味着的是一个没有*永久运行服务器*的服务，通常这意味着大部分时间不运行或资源使用低，并在需要时动态扩展。这可以节省很多成本。

例如，组织通常有一些只需要每小时运行一次、每月运行一次或按需运行的业务功能。也许组织在月底打印支票（在英国称为cheques）来支付员工的工资。这些支票可能需要将工资金额转换为文字以打印在支票上。可以将将数字转换为文字的功能实现为一个无服务器服务。

例如，使用内容管理系统，编辑器可能会上传新的图片，这些图片可能需要以各种方式处理，如生成缩略图和其他优化。这项工作可以添加到队列中，或者当文件上传到 Blob 存储时触发 Azure 函数。

Azure 函数可以远不止是一个单一的功能。它们支持复杂的状态化工作流和事件驱动的解决方案，使用**Durable Functions**。

我在这本书中不涉及 Durable Functions，如果您感兴趣，可以在此链接中了解更多关于使用 C# 和 .NET 实现它们的信息：[https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=csharp](https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=csharp)。

Azure 函数基于触发器和绑定的编程模型，使你的无服务器服务能够响应事件并连接到其他服务，如数据存储。

## Azure 函数触发器和绑定

**触发器**和**绑定**是 Azure 函数的关键概念。

触发器是导致函数执行的原因。每个函数必须有一个，且只有一个触发器。以下列出了最常见的触发器：

+   **HTTP**: 此触发器响应传入的 HTTP 请求，通常是 `GET` 或 `POST`。

+   **Azure SQL**: 此触发器在检测到 SQL 表上的更改时响应。

+   **Cosmos DB**: 此触发器使用 Cosmos DB 变更流来监听插入和更新。

+   **Timer**: 此触发器响应预定时间的到来。如果函数失败，则不会重试。直到下一次预定时间，函数不会被再次调用。

+   **SignalR**: 此触发器响应来自 Azure SignalR 服务的消息。

+   **队列**和**RabbitMQ**: 这些触发器响应队列中到达的消息，准备进行处理。

+   **Blob Storage**: 此触发器响应新的或更新的 **二进制大对象** (**Blob**)。

+   **Event Grid**和**Event Hub**: 这些触发器在预定义的事件发生时响应。

绑定允许函数有输入和输出。每个函数可以有零个、一个或多个绑定。以下列出了一些常见的绑定：

+   **Azure SQL**: 在 SQL Server 数据库中读取或写入表。

+   **Blob Storage**: 读取或写入存储为 BLOB 的任何文件。

+   **Cosmos DB**: 读取或写入云规模数据存储中的文档。

+   **SignalR**: 接收或执行远程方法调用。

+   **HTTP**: 发起 HTTP 请求并接收响应。

+   **队列**和**RabbitMQ**: 向队列写入消息或从队列读取消息。

+   **SendGrid**: 发送电子邮件消息。

+   **Twilio**: 发送短信消息。

+   **IoT hub**: 向互联网连接的设备写入。

你可以在以下链接中查看支持的触发器和绑定的完整列表：[https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings?tabs=csharp#supported-bindings](https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings?tabs=csharp#supported-bindings)。

触发器和绑定对于不同的语言配置方式不同。对于 C# 和 Java，你需要在方法和参数上使用属性进行装饰。对于其他语言，你需要配置一个名为 `function.json` 的文件。

## NCRONTAB 表达式

**Timer** 触发器使用 **NCRONTAB 表达式**来定义计时器的频率。默认时区是 **协调世界时** (**UTC**)。这可以被覆盖，但你真的应该使用 UTC，原因你已经在 *第7章*，*处理日期、时间和国际化* 中了解到了。

如果你是在 App Service 计划中托管，那么你可以使用 `TimeSpan` 作为替代，但我推荐学习 NCRONTAB 表达式以获得灵活性。

NCRONTAB 表达式由五个或六个部分组成（如果包含秒数）：

[PRE0]

上面的值字段中的星号 `*` 表示该列的所有合法值，就像括号中的那样。你可以使用破折号指定范围，并使用 `/` 指定步长值。以下是一些如何以这种格式指定值的示例：

+   `0` 表示该值。例如，对于小时，表示午夜。

+   `0,6,12,18` 表示那些列出的值。例如，对于小时，表示午夜、上午6点、中午12点和下午6点。

+   `3-7` 表示该值的包含范围。例如，对于小时，表示上午3点、上午4点、上午5点、上午6点和上午7点。

+   `4/3` 表示起始值为 `4`，步长值为 `3`。例如，对于小时，表示上午4点、上午7点、上午10点、下午1点、下午4点、下午7点和晚上10点。

*表10.1* 显示了一些更多示例：

| **表达式** | **描述** |
| --- | --- |
| `0 5 * * * *` | 每小时一次，在每小时的第5分钟。 |
| `0 0,10,30,40 * * * *` | 每小时四次 – 在每个小时的第0分钟、第10分钟、第30分钟和第40分钟。 |
| `* * */2 * * *` | 每2小时一次。 |
| `0,15 * * * * *` | 每分钟的第0秒和第15秒。 |
| `0/15 * * * * *` | 每分钟的第0秒、第15秒、第30秒和第45秒，也称为每15秒一次。 |
| `0-15 * * * * *` | 在每分钟的0秒、1秒、2秒、3秒，等等，直到15秒之后，但不是每分钟的16秒到59秒。 |
| `0 30 9-16 * * *` | 每天八次 – 在上午9:30、上午10:30，等等，直到下午4:30。 |
| `0 */5 * * * *` | 每小时12次 – 在每小时的第5分钟的每0秒。 |
| `0 0 */4 * * *` | 每天六次 – 在每天每4小时的第0分钟。 |
| `0 30 9 * * *` | 每天上午9:30。 |
| `0 30 9 * * 1-5` | 每个工作日上午9:30。 |
| `0 30 9 * * Mon-Fri` | 每个工作日上午9:30。 |
| `0 30 9 * Jan Mon` | 1月每周一上午9:30。 |

表10.1：NCRONTAB表达式的示例

现在让我们构建一个简单的控制台应用程序来测试您对NCRONTAB表达式的理解：

1.  使用您首选的代码编辑器将名为 `NCrontab.Console` 的新控制台应用程序添加到 `Chapter10` 解决方案中。

1.  在 `NCrontab.Console` 项目中，将警告视为错误，全局和静态导入 `System.Console` 类，并为 `NCrontab.Signed` 添加包引用，如下面的标记所示：

    [PRE1]

    NCRONTAB库仅用于解析表达式。它本身不是一个调度器。您可以在以下链接的GitHub仓库中了解更多信息：[https://github.com/atifaziz/NCrontab](https://github.com/atifaziz/NCrontab)。

1.  构建项目 `NCrontab.Console` 以恢复包。

1.  在 `Program.cs` 文件中，删除现有的语句。添加语句以定义2023年的日期范围，输出NCRONTAB语法的摘要，并构建一个NCRONTAB计划，然后使用它来输出2023年将发生的下一个40个事件，如下面的代码所示：

    [PRE2]

    注意以下内容：

    +   发生事件的默认潜在时间跨度是2023年全年。

    +   默认表达式是 `0,30 * * * * *`，意味着在每个月每个工作日每天每小时的每分钟的第0秒和第30秒。

    +   语法帮助的格式化假设每个组件将占用三个字符宽，因为用于输出格式的 `-3`。你可以编写一个更聪明的算法来动态调整箭头指向可变宽度的组件，但我很懒惰。我将把这个留给你作为练习。

    +   我们的表达式包括秒，因此在解析时必须将其设置为附加选项。

    +   定义计划后，计划调用其 `GetNextOccurrences` 方法以返回所有计算出的发生序列。

    +   循环只输出前 40 个发生次数。这应该足以理解大多数表达式的工作方式。

1.  不带调试启动控制台应用程序，并注意发生次数为每 30 秒一次，如下部分输出所示：

    [PRE3]

    注意，尽管开始时间是 `Sun, 01 Jan 2023 00:00:00`，但该值不包括在发生次数中，因为它不是一个“下一个”发生。

1.  关闭控制台应用程序。

1.  在 `Program.cs` 中，修改表达式的组件以测试表中的某些示例，或者创建自己的示例。尝试表达式 `0 0 */4 * * *`，并注意它应该有如下部分输出：

    [PRE4]

注意，尽管开始时间是 `Sun, 01 Jan 2023 00:00:00`，但该值不包括在发生次数中，因为它不是一个“下一个”发生。因此，星期日只有五个发生次数。从星期一开始，每天应有预期的六个发生次数。

## Azure Functions 版本和语言

运行时主机 Azure Functions 的版本 4 是唯一仍然普遍可用的版本。所有旧版本都已达到生命周期的终点。

**良好实践**：Microsoft 建议在所有语言中使用 v4。v1、v2 和 v3 处于维护模式，应避免使用。

Azure Functions v4 支持的语言和平台包括：

+   **C#** 和 **F#**：.NET 8, .NET 7, .NET 6, 和 .NET Framework 4.8。注意，.NET 6 和 .NET 7（以及未来的 .NET 9）仅支持在隔离工作模型中，因为它们是 **标准支持期**（**STS**）版本，或者，在 .NET 6 的情况下，它们是较旧的长期支持（**LTS**）版本。.NET 8 支持隔离和进程内工作模型，因为它是一个 **长期支持**（**LTS**）版本。

+   **JavaScript**：Node 14, 16, 和 18。TypeScript 通过转换（转换/编译）到 JavaScript 支持。

+   **Java** 8, 11, 和 17。

+   **PowerShell** 7.2。

+   **Python** 3.7, 3.8, 3.9, 和 3.10。

**更多信息**：您可以在以下链接中查看整个语言表：[https://learn.microsoft.com/en-us/azure/azure-functions/functions-versions?tabs=v4&pivots=programming-language-csharp#languages](https://learn.microsoft.com/en-us/azure/azure-functions/functions-versions?tabs=v4&pivots=programming-language-csharp#languages)。

在这本书中，我们将只查看使用 C# 和 .NET 8 实现 Azure Functions，这样我们就可以使用进程内和隔离工作模型并获得 LTS。

对于高级用途，您甚至可以注册一个自定义处理程序，这样您就可以使用您喜欢的任何语言来实现 Azure 函数。您可以在以下链接中了解更多关于 Azure Functions 自定义处理程序的信息：[https://learn.microsoft.com/en-us/azure/azure-functions/functions-custom-handlers](https://learn.microsoft.com/en-us/azure/azure-functions/functions-custom-handlers)。

## Azure Functions 工作模型

Azure Functions 有两种工作模型，进程内和隔离，如下列表所述：

+   **进程内**：您的函数在一个类库中实现，必须在与宿主相同的进程中运行，这意味着您的函数必须在 .NET 最新的 LTS 版本上运行。最新的 LTS 版本是 .NET 8。下一个 LTS 版本将在 2025 年 11 月的 .NET 10 中发布，但 .NET 8 将是支持进程内托管的最后一个版本。在 .NET 8 之后，将只支持所有版本的隔离工作模型。

+   **隔离型**：您的函数在一个控制台应用程序中实现，该应用程序在其自己的进程中运行。因此，您的函数可以在任何支持的 .NET 版本上执行，完全控制其 `Main` 入口点，并具有调用中间件等附加功能。从 .NET 9 开始，这将是唯一的工作模型。您可以在以下链接中了解更多关于此决定的信息：[https://techcommunity.microsoft.com/t5/apps-on-azure-blog/net-on-azure-functions-august-2023-roadmap-update/ba-p/3910098](https://techcommunity.microsoft.com/t5/apps-on-azure-blog/net-on-azure-functions-august-2023-roadmap-update/ba-p/3910098)。

**最佳实践**：新项目应使用隔离工作模型。

## Azure Functions 托管计划

在本地测试后，您必须将 Azure Functions 项目部署到 Azure 托管计划。您可以从以下列表中选择三种 Azure Functions 计划：

+   **消费型**：在此方案中，根据活动动态添加和删除托管实例。此方案与 *无服务器* 方案最为接近。在负载高峰期间自动扩展。成本仅针对运行时的计算资源。您可以配置函数执行时间的超时，以确保您的函数不会永远运行。

+   **高级**：此方案支持弹性上下文扩展，始终处于预热状态的实例以避免冷启动，无限制的执行时长，多核实例大小最多可达四个核心，成本可能更具可预测性，并为多个 Azure Functions 项目提供高密度应用分配。成本基于实例间分配的核心秒数和内存。必须始终分配至少一个实例，因此即使该月没有执行，也始终会有一个最低的每月成本。

+   **专用**：在云中的服务器农场等效环境中执行。托管由控制分配服务器资源的 Azure App Service 计划提供。Azure App Service 计划包括基本、标准、高级和隔离。如果你已经有一个用于其他项目（如 ASP.NET Core MVC 网站、gRPC、OData 和 GraphQL 服务等）的 App Service 计划，那么这个计划可能是一个特别好的选择。费用仅限于 App Service 计划。你可以在这个计划中托管尽可能多的 Azure Functions 和其他 Web 应用。

    **警告！**高级和专用计划都运行在 Azure App Service 计划上。你必须仔细选择与你的 Azure Functions 托管计划兼容的正确 App Service 计划。例如，对于高级计划，你应该选择一个弹性高级计划，如 `EP1`。如果你选择一个像 `P1V1` 这样的 App Service 计划，那么你选择的是一个不会弹性扩展的专用计划！

你可以在以下链接中了解更多关于你的选择：[https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale)。

## Azure 存储要求

Azure Functions 需要一个 Azure 存储账户来存储某些绑定和触发器的信息。这些 Azure 存储服务也可以由你的代码用于其实施：

+   **Azure 文件存储**：在消费或高级计划中存储和运行你的函数应用代码。

+   **Azure Blob 存储**：存储绑定和函数键的状态。

+   **Azure 队列存储**：某些触发器用于故障和重试处理。

+   **Azure 表存储**：Durable Functions 中的任务中心使用 Blob、队列和表存储。

## 使用 Azurite 本地测试

Azurite 是一个开源的本地环境，用于测试 Azure Functions 及其相关的 Blob、队列和表存储。Azurite 在 Windows、Linux 和 macOS 上都是跨平台的。Azurite 取代了旧的 Azure 存储模拟器。

要安装 Azurite：

+   对于 Visual Studio 2022，Azurite 已包含在内。

+   对于 Visual Studio Code，搜索并安装 Azurite 扩展。

+   对于 JetBrains Rider，安装 Azure Toolkit for Rider 插件，该插件包括 Azurite。

+   对于在命令提示符下的安装，你必须安装 Node.js 8 或更高版本，然后可以输入以下命令：

    [PRE5]

一旦你在本地测试了 Azure 函数，你可以切换到云中的 Azure 存储账户。

你可以在以下链接中了解更多关于 Azurite 的信息：[https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azurite](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azurite)。

## Azure Functions 授权级别

Azure Functions 有三个授权级别，用于控制是否需要 API 密钥：

+   **匿名**：不需要 API 密钥。

+   **函数**：需要函数级别的密钥。

+   **管理员**：需要主密钥。

API 密钥可通过 Azure 门户获取。

## Azure Functions 支持依赖注入

Azure Functions 中的依赖注入建立在标准的 .NET 依赖注入功能之上，但具体实现方式取决于你选择的工人模型。

要注册依赖服务，创建一个继承自 `FunctionsStartup` 类的类，并重写其 `Configure` 方法。添加 `[FunctionsStartup]` 程序集属性以指定注册的启动类名。将服务添加到传递给方法的 `IFunctionsHostBuilder` 实例中。你将在本章后面的任务中这样做。

通常，实现 Azure Functions 函数的类是 `static` 的，并且有一个 `static` 方法。`static` 类不使用构造函数进行实例化。依赖注入使用构造函数注入，这意味着你必须使用实例类来注入服务以及你的函数类实现。你将在编码任务中看到如何做到这一点。

## 安装 Azure Functions 核心工具

**Azure Functions 核心工具**提供了创建函数的核心运行时和模板，这使你能够在 Windows、macOS 和 Linux 上使用任何代码编辑器进行本地开发。

Azure Functions 核心工具包含在 Visual Studio 2022 的 **Azure 开发** 工作负载中，因此你可能已经安装了它。

你可以从以下链接安装最新版本的 **Azure Functions 核心工具**：

[https://www.npmjs.com/package/azure-functions-core-tools](https://www.npmjs.com/package/azure-functions-core-tools)

前一个链接中找到的页面提供了在 Windows 上使用 **Microsoft 软件安装程序 (MSI)** 和 `winget`、在 Mac 上使用 Homebrew、在任何操作系统上使用 `npm` 以及常见 Linux 发行版上安装的说明。

如果你使用 JetBrains Rider，那么通过 Rider 安装 Azure Functions 核心工具。

# 构建 Azure Functions 项目

现在，我们可以创建一个 Azure Functions 项目。虽然它们可以通过 Azure 门户在云端创建，但开发者最好先在本地创建和运行它们，以便获得更好的体验。一旦你在自己的电脑上测试了函数，你就可以将其部署到云端。

每个代码编辑器在开始 Azure Functions 项目时都有略微不同的体验，因此让我们逐一查看，从 Visual Studio 2022 开始。

## 使用 Visual Studio 2022

如果你更喜欢使用 Visual Studio 2022，以下是如何创建 Azure Functions 项目的步骤：

1.  在 Visual Studio 2022 中，创建一个新项目，如下列所示：

    +   项目模板：**Azure Functions**

    +   解决方案文件和文件夹：`Chapter10`

    +   项目文件和文件夹：`Northwind.AzureFunctions.Service`

    +   **函数工作者**：**.NET 8.0 Isolated (长期支持)**

    +   **函数**：**Http 触发器**

    +   **使用 Azurite 作为运行时存储账户 (AzureWebJobsStorage)**: 已选择

    +   **禁用** **Docker**：已清除

    +   **授权级别**：**匿名**

1.  点击 **创建**。

1.  配置解决方案的启动项目为当前选择。

## 使用 Visual Studio Code

如果您更喜欢使用 Visual Studio Code，以下是如何创建 Azure Functions 项目的步骤：

1.  在 Visual Studio Code 中，导航到 **扩展** 并搜索 Azure Functions (`ms-azuretools.vscode-azurefunctions`)。它依赖于两个其他扩展：Azure 账户 (`ms-vscode.azure-account`) 和 Azure 资源 (`ms-azuretools.vscode-azureresourcegroups`)，因此这些也将被安装。点击 **安装** 按钮来安装扩展。

1.  创建一个名为 `Northwind.AzureFunctions.Service` 的文件夹。（如果您之前使用 Visual Studio 2022 创建了相同的项目，那么请创建一个名为 `Chapter10-vscode` 的新文件夹，并将此新项目文件夹放在那里。）

1.  在 Visual Studio Code 中，打开 `Northwind.AzureFunctions.Service` 文件夹。

1.  导航到 **视图** | **命令面板**，输入 `azure f`，然后在 Azure Functions 命令列表中，点击 **Azure Functions：创建新项目…**。

1.  选择包含您的函数项目的 `Northwind.AzureFunctions.Service` 文件夹。

1.  在提示时，选择以下选项：

    +   选择语言：**C#**

    +   选择一个 .NET 运行时：**.NET 8 Isolated LTS**。

    +   选择项目第一个函数的模板：**HTTP trigger**。

    +   提供一个函数名称：`NumbersToWordsFunction`

    +   提供一个命名空间：`Northwind.AzureFunctions.Service`

    +   选择授权级别：**匿名**

1.  导航到 **终端** | **新建终端**。

1.  在命令提示符下，构建项目，如下所示：

    [PRE6]

## 使用 func CLI

如果您更喜欢使用命令行和一些其他代码编辑器，以下是如何创建和启动 Azure Functions 项目的步骤：

1.  创建一个名为 `Chapter10-cli` 的文件夹，并在其中创建一个名为 `Northwind.AzureFunctions.Service` 的子文件夹。

1.  在命令提示符或终端中，在 `Northwind.AzureFunctions.Service` 文件夹中，使用以下命令创建一个新的 Azure Functions 项目，使用 C#：

    [PRE7]

1.  在命令提示符或终端中，在 `Northwind.AzureFunctions.Service` 文件夹中，使用以下命令创建一个新的 Azure Functions 函数，使用 `HTTP trigger` 可以匿名调用：

    [PRE8]

1.  可选地，您可以在本地启动函数，如下所示：

    [PRE9]

如果您在命令提示符或终端中找不到 `func`，那么请尝试使用 Chocolatey 安装 Azure Functions Core Tools，如以下链接所述：[https://community.chocolatey.org/packages/azure-functions-core-tools](https://community.chocolatey.org/packages/azure-functions-core-tools)。

## 检查 Azure Functions 项目

在我们编写函数之前，让我们回顾一下构成 Azure Functions 项目的要素：

1.  打开项目文件，并注意 Azure Functions 版本和实现响应 HTTP 请求的 Azure 函数所需的包引用，如下所示：

    [PRE10]

1.  在 `host.json` 中，请注意已启用将日志记录到 Application Insights，但排除了 `Request` 类型，如下所示：

    [PRE11]

    Application Insights是Azure的监控和日志服务。我们将在本章中不使用它。

1.  在`local.settings.json`中，确认在本地开发期间，你的项目将使用本地开发存储和隔离的工作模型，如下所示：

    [PRE12]

1.  如果`AzureWebJobsStorage`设置是空的或缺失的，这可能发生在你使用Visual Studio Code的情况下，那么添加它，将其设置为`UseDevelopmentStorage=true`，然后保存更改。

    `FUNCTIONS_WORKER_RUNTIME`是你项目使用的语言。`dotnet`表示.NET类库；`dotnet-isolated`表示.NET控制台应用程序。其他值包括`java`、`node`、`powershell`和`python`。

1.  在`Properties`文件夹中，在`launchSettings.json`中，注意为网络服务随机分配的端口号，如下面的配置所示：

    [PRE13]

1.  将端口号更改为`5101`并将更改保存到文件中。

## 实现一个简单的函数

让我们使用`Humanizer`包实现将数字转换为文字的函数：

1.  在项目文件中，添加对Humanizer的包引用，如下所示：

    [PRE14]

1.  构建项目以恢复包。

    如果你使用Visual Studio 2022，在`Northwind.AzureFunctions.Service`项目中，右键单击`Function1.cs`并将其重命名为`NumbersToWordsFunction.cs`。

1.  在`NumbersToWordsFunction.cs`中，修改内容以实现将金额作为数字转换为文字的Azure函数，如下面的代码所示：

    [PRE15]

## 测试一个简单的函数

现在我们可以在我们本地开发环境中测试这个函数：

1.  启动`Northwind.AzureFunctions.Service`项目：

    +   如果你使用Visual Studio Code，你需要导航到**运行和调试**面板，确保**附加到.NET函数**被选中，然后点击**运行**按钮。

    +   在Windows上，如果你看到来自**Windows Defender防火墙**的**Windows安全警报**，那么点击**允许访问**。

1.  注意**Azure Functions Core Tools**托管你的函数，如下面的输出和*图10.1*所示：

    [PRE16]

    ![](img/B19587_10_01.png)

    图10.1：Azure Functions Core Tools托管一个函数

    `Host` `lock` `lease`消息可能需要几分钟才能出现，所以如果它没有立即显示，请不要担心。

1.  选择你的函数的URL并将其复制到剪贴板。

1.  启动Chrome。

1.  将URL粘贴到地址框中，追加查询字符串`?amount=123456`，并在浏览器的**十二万三千四百五十六**中注意成功的响应，如图*图10.2*所示：

![](img/B19587_10_02.png)

图10.2：对本地运行的Azure函数的成功调用

1.  在命令提示符或终端中，注意函数调用成功，如下面的输出所示：

    [PRE17]

1.  尝试调用函数时在查询字符串中不包含金额，或者金额为非整数值，如`apples`，并注意函数返回一个`400`状态码，表示有自定义消息的无效请求，`Failed to parse: apples`。

1.  关闭Chrome并关闭Web服务器（或在Visual Studio Code中停止调试）。

# 响应定时器和资源触发

现在你已经看到了一个响应HTTP请求的Azure Functions函数，让我们构建一些响应其他类型触发器的函数。

内置了对HTTP和定时器触发的支持。其他绑定的支持作为扩展包实现。

## 实现定时器触发的函数

首先，我们将创建一个每小时运行一次的函数，请求[amazon.com](https://www.amazon.com/)上我书籍第八版（*C# 12和.NET 8 – 现代跨平台开发基础*）的页面，以便我可以跟踪其在美国的最佳卖家排名。

函数需要执行HTTP `GET`请求，因此我们应该注入HTTP客户端工厂。为此，我们需要添加一些额外的包引用并创建一个特殊的启动类：

1.  在`Northwind.AzureFunctions.Service`项目中，添加用于处理Azure Functions扩展和计时器的包引用，如下面的标记所示：

    [PRE18]

1.  构建项目`Northwind.AzureFunctions.Service`以恢复包。

1.  在`Program.cs`中，导入用于处理依赖注入和HTTP媒体头的命名空间，如下面的代码所示：

    [PRE19]

1.  在`Program.cs`中添加语句以配置一个HTTP客户端工厂，用于向亚马逊发送请求，就像它是Chrome浏览器作为依赖服务一样，如下面的高亮代码所示：

    [PRE20]

    2023年6月5日的Chrome版本是`114.0.5735.91`。主要版本号通常每月递增，因此到2023年11月，它可能为`119`，到2024年11月，它可能为`131`。

1.  添加一个名为`ScrapeAmazonFunction.cs`的类文件。

1.  修改其内容以实现一个函数，该函数请求亚马逊网站上我第七版书的页面，并处理使用GZIP压缩的响应，以提取书籍的最佳卖家排名，如下面的代码所示：

    [PRE21]

## 测试定时器触发的函数

可以通过以下格式的HTTP `GET`请求检索函数信息：

`http://locahost:5101/admin/functions/<functionname>`

现在我们可以测试本地开发环境中的定时器函数：

1.  启动`Northwind.AzureFunctions.Service`项目：

    +   如果你使用Visual Studio Code，你需要确保已安装Azurite扩展并且Azurite服务正在运行。导航到**运行和调试**面板，确保已选择**附加到.NET Functions**，然后点击**运行**按钮。

1.  注意现在有两个函数，如下面的部分输出所示：

    [PRE22]

1.  在`HttpRequests`文件夹中，添加一个名为`azurefunctions-scrapeamazon.http`的新文件。

1.  修改其内容以定义一个全局变量和两个请求到本地托管的Azure Functions服务，如下代码所示：

    [PRE23]

1.  发送第一个请求并注意返回了一个包含`NumbersToWordsFunction`函数信息的JSON文档，如下所示：

    [PRE24]

1.  发送第二个请求并注意返回了一个包含`ScrapeAmazonFunction`函数信息的JSON文档。对于这个函数，最有趣的信息是绑定类型和计划，如下部分响应所示：

    [PRE25]

1.  添加一个第三个请求，通过发送一个包含空JSON文档的`POST`请求到其`admin`端点，手动触发计时器函数，而无需等待小时标记，如下代码所示：

    [PRE26]

1.  发送第三个请求并注意它被成功接受，如下所示：

    [PRE27]

1.  在请求体中移除`{}`，再次发送，并注意从客户端错误响应中我们可以推断出需要一个空JSON文档，如下响应所示：

    [PRE28]

1.  将空JSON文档添加回去。

1.  在Azure Functions服务的命令提示符或终端中，注意函数是由我们的调用触发的。它输出了触发时间（下午1:49）和其正常计时计划中的下一次发生时间（如果服务继续运行，时间为下午2点），如下所示：

    [PRE29]

1.  可选地，等待直到小时标记，并注意下一次触发，如下所示：

    [PRE30]

1.  如果我停止运行服务，等待超过一小时，然后启动服务，它将立即运行函数，因为已经过期，如下所示（高亮显示）：

    [PRE31]

1.  关闭Azure Functions服务。

## 实现与队列和Blob一起工作的函数

HTTP触发的函数直接以纯文本形式响应了`GET`请求。我们现在将定义一个类似的功能，将其绑定到队列存储，并向队列中添加一条消息，表示需要生成并上传到Blob存储的图像。然后可以将其打印出来作为检查。

当在本地运行服务时，我们希望在本地文件系统中生成支票BLOB的图像，以便更容易确保其正确工作。我们将在本地设置中设置一个自定义环境变量来检测该条件。

我们需要一个看起来像手写的字体。谷歌有一个有用的网站，你可以在这里搜索、预览和下载字体。我们将使用的是Caveat字体，如下链接所示：

[https://fonts.google.com/specimen/Caveat?category=Handwriting&preview.text=one%20hundred%20and%20twenty%20three%20thousand,%20four%20hundred%20and%20fifty%20six&preview.text_type=custom#standard-styles](https://fonts.google.com/specimen/Caveat?category=Handwriting&preview.text=one%20hundred%20and%20twenty%20three%20thousand,%20four%20hundred%20and%20fifty%20six&preview.text_type=custom#standard-styles)

让我们开始：

1.  从上面的链接下载字体，解压 ZIP 文件，并将文件复制到名为 `fonts` 的文件夹中，如图 10.3 所示的 Visual Studio 2022：

![](img/B19587_10_03.png)

图 10.3：Visual Studio 2022 中带有 Caveat 字体文件的字体文件夹

1.  选择 `Caveat-Regular.ttf` 字体文件。

1.  在 **属性** 中，将 **复制到输出目录** 设置为 **始终复制**，如图 10.3 所示。这将在项目文件中添加一个条目，如下所示高亮显示的标记：

    [PRE32]

    如果你使用的是 Visual Studio Code，请手动将前面的条目添加到项目文件中。

1.  在 `Northwind.AzureFunctions.Service` 项目中，添加用于处理 Azure 队列和 Blob 存储扩展以及使用 `ImageSharp` 绘图的包引用，如下所示标记：

    [PRE33]

1.  构建项目以还原包。

1.  在 `Northwind.AzureFunctions.Service` 项目中，添加一个名为 `NumbersToChecksFunction.cs` 的新类。

1.  在 `NumbersToChecksFunction.cs` 中添加语句以将函数与用于队列存储的输出绑定注册，以便它可以写入命名队列，并且当金额成功解析后返回到队列，如下所示代码：

    [PRE34]

1.  在 `local.settings.json` 中，添加一个名为 `IS_LOCAL` 的环境变量，其值为 `true`，如图所示高亮显示的配置：

    [PRE35]

1.  添加一个名为 `CheckGeneratorFunction.cs` 的类文件。

1.  修改其内容，如下所示代码：

    [PRE36]

注意以下事项：

+   `[QueueTrigger("checksQueue")] string message` 参数表示函数由添加到 `checksQueue` 的消息触发，并且队列项自动传递给名为 `message` 的参数。

+   我们使用 ImageSharp 创建一个 1200x600 的支票图像。

+   我们使用当前的 UTC 日期和时间来命名 BLOB 以避免重复。在实际实现中，你可能需要更健壮的机制，如 GUID。

+   如果将 `IS_LOCAL` 环境变量设置为 `true`，则我们将图像作为 PNG 格式保存到本地文件系统的 `blobs` 子文件夹中。

+   我们将图像保存为 PNG 格式到内存流中，然后作为字节数组返回并上传到由 `[BlobOutput("checks-blob-container/check.png")]` 属性定义的 BLOB 容器。

## 测试与队列和 BLOB 一起工作的函数

现在我们可以测试在本地开发环境中与队列和 BLOB 一起工作的函数：

1.  启动 `Northwind.AzureFunctions.Service` 项目：

    如果你使用的是 Visual Studio Code，你需要导航到 **运行和调试** 面板，确保已选择 **附加到 .NET Functions**，然后点击 **运行** 按钮。

1.  注意现在有四个函数，如下所示部分输出：

    [PRE37]

1.  在 `HttpRequests` 文件夹中，添加一个名为 `azurefunctions-numberstochecks.http` 的新文件。

1.  修改其内容，如下所示代码：

    [PRE38]

1.  发送请求并注意返回了一个包含 `NumbersToWordsFunction` 函数信息的 JSON 文档，如下所示响应：

    [PRE39]

1.  在命令提示符或终端中，注意函数调用成功，并将消息发送到队列中，从而触发了 `CheckGeneratorFunction`，如下所示输出：

    [PRE40]

1.  在 `Northwind.AzureFunctions.Service\bin\Debug\net8.0\blobs` 文件夹中，注意在 `blobs` 文件夹中创建的本地图像，如图 *图 10.4* 所示：

![图片](img/B19587_10_04.png)

图 10.4：在项目 blob 文件夹中生成的验证图像

1.  注意验证图像，如图 *图 10.5* 所示：

![图片](img/B19587_10_05.png)

图 10.5：在 Windows Paint 中打开的验证图像

# 将 Azure Functions 项目发布到云端

现在，让我们在 Azure 订阅中创建一个函数应用和相关资源，然后将您的函数部署到云端并运行。

如果您还没有 Azure 账户，那么您可以在以下链接处免费注册：[https://azure.microsoft.com/en-us/free/](https://azure.microsoft.com/en-us/free/)。

## 使用 Visual Studio 2022 发布

Visual Studio 2022 提供了一个用于发布到 Azure 的图形用户界面：

1.  在 **解决方案资源管理器** 中，右键单击 `Northwind.AzureFunctions.Service` 项目并选择 **发布**。

1.  选择 **Azure** 然后点击 **下一步**。

1.  选择 **Azure Function App (Linux)** 并点击 **下一步**。

1.  登录并输入您的 Azure 凭据。

1.  选择您的订阅；例如，我选择了名为 **Pay-As-You-Go** 的订阅。

1.  在 **函数实例** 部分，点击 **+ 创建新** 按钮。

1.  完成对话框，如图 *图 10.6* 所示：

    +   **名称**：这必须是全局唯一的。建议根据项目名称和当前日期时间命名。

    +   **订阅名称**：选择您的订阅。

    +   **资源组**：选择或创建一个新的资源组，以便稍后更容易删除所有内容。我选择了 `apps-services-book`。

    +   **计划类型**：**消费**（仅为您使用的付费）。

    +   **位置**：您最近的数据中心。我选择了 **UK South**。

    +   **Azure 存储**：在您最近的数据中心中创建一个名为 `northwindazurefunctions`（或任何其他全局唯一的名称）的新账户，并为账户类型选择 **Standard – 本地冗余存储**。

    +   **应用程序洞察**：**无**。

![图片](img/B19587_10_06.png)

图 10.6：创建新的 Azure 函数应用

1.  点击 **创建**。此过程可能需要一分钟或更长时间。

1.  在 **发布** 对话框中，点击 **完成** 然后点击 **关闭**。

1.  在 **发布** 窗口中，点击 **发布** 按钮，如图 *图 10.7* 所示：

![图片](img/B19587_10_07.png)

图 10.7：使用 Visual Studio 2022 准备发布的 Azure Function App

1.  查看输出窗口，如图以下发布输出所示：

    [PRE41]

1.  在 **发布** 窗口中，点击 **打开站点** 并注意您的 Azure Functions v4 主站已准备就绪。

1.  通过将以下相对 URL 添加到地址框中，在浏览器中测试函数，如图 *图 10.8* 所示：

    `/api/NumbersToWordsFunction?amount=987654321`

    ![图片](img/B19587_10_08.png)

    图 10.8：调用 Azure 云中托管的功能

    ## 使用 Visual Studio Code 进行发布

    您可以在以下链接中学习如何使用 Visual Studio Code 进行发布：

[https://learn.microsoft.com/en-us/azure/azure-functions/functions-develop-vs-code?tabs=csharp#sign-in-to-azure](https://learn.microsoft.com/en-us/azure/azure-functions/functions-develop-vs-code?tabs=csharp#sign-in-to-azure)

现在您已成功将 Azure Functions 项目发布到云端，了解如何高效地管理资源变得很重要。让我们探讨如何清理我们的 Azure Functions 资源，以避免不必要的成本并确保资源管理整洁。

# 清理 Azure Functions 资源

您可以使用以下步骤删除函数应用及其相关资源，以避免产生更多成本：

1.  在您的浏览器中，导航到 [https://portal.azure.com/](https://portal.azure.com/)

1.  在 Azure 门户中，在您的函数应用的 **概览** 选项卡中，选择 **资源组**。

1.  确认它只包含您想要删除的资源；例如，应该有一个 **存储账户**、一个 **函数应用** 和一个 **应用服务计划**。

1.  如果您确定要删除组中的所有资源，请单击 **删除资源组** 并接受任何其他确认。或者，您可以逐个删除每个资源。

# 练习和探索

通过回答一些问题、进行一些动手实践，以及更深入地研究本章的主题来测试您的知识和理解。

## 练习 10.1 – 测试您的知识

回答以下问题：

1.  Azure Functions 的进程内和隔离工作模型之间有什么区别？

1.  您使用什么属性来使函数在队列中收到消息时触发？

1.  您使用什么属性来使队列可用于发送消息？

1.  以下 NCRONTAB 表达式定义了什么计划？

`0 0 */6 * 6 6`

1.  您如何配置依赖服务以在函数中使用？

## 练习 10.2 – 探索主题

使用以下页面上的链接获取本章涵盖主题的更多详细信息：

[https://github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-10---building-serverless-nanoservices-using-azure-functions](https://github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-10---building-serverless-nanoservices-using-azure-functions)

# 摘要

在本章中，您学习了：

+   Azure Functions 的一些概念

+   如何使用 Azure Functions 构建无服务器服务

+   如何响应 HTTP、定时器和队列触发器

+   如何绑定到队列和 Blob 存储

+   如何将 Azure Functions 项目部署到云端

在下一章中，您将了解 SignalR，这是一种在客户端和服务器之间进行实时通信的技术。
