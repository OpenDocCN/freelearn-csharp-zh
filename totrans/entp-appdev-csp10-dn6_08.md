# *第 6 章*：.NET 6 中的配置

.NET 6 中的配置包括默认设置以及应用程序的运行时设置；配置是一个非常强大的功能。我们可以更新设置，如功能标志以启用或禁用功能、依赖服务端点、数据库连接字符串、日志级别等，并在不重新编译的情况下控制应用程序的运行时行为。

在本章中，我们将涵盖以下主题：

+   理解配置

+   利用内置配置提供程序

+   构建自定义配置提供程序

到本章结束时，你将很好地掌握配置概念、配置提供程序以及如何在项目中利用它们，以及能够识别适合你应用程序的配置和配置源。

# 技术要求

你需要对 .NET 和 Azure 有基本的了解。本章的代码可以在以下位置找到：https://github.com/PacktPublishing/Enterprise-Application-Development-with-C-10-and-.NET-6-Second-Edition/tree/main/Chapter06。

# 理解配置

配置通常存储为 `Program.cs`（），你将获得 .NET 6 提供的默认配置。此外，你可以配置不同的内置和自定义配置源，并在需要时使用不同的配置提供程序读取它们：

![图 6.1 – 应用和配置

](img/Figure_6.1_B18507.jpg)

图 6.1 – 应用和配置

上述图示显示了应用程序、配置提供程序和配置文件之间的高级关系。应用程序使用配置提供程序从配置源读取配置；配置可以是环境特定的。**Env A** 可能是你的开发环境，而 **Env B** 可能是你的生产环境。在运行时，应用程序将根据其运行时的上下文和环境读取正确的配置。

在下一节中，我们将了解默认配置的工作原理以及如何从 `appsettings.json` 文件中添加和读取配置。

## 默认配置

要了解默认配置的工作原理，让我们创建一个新的 .NET 6 Web API，将项目名称设置为 `TestConfiguration`，并打开 `Program.cs` 文件。以下是从 `Program.cs` 文件中的代码片段：

[PRE0]

[PRE1]

[PRE2]

[PRE3]

[PRE4]

[PRE5]

[PRE6]

[PRE7]

[PRE8]

[PRE9]

[PRE10]

[PRE11]

[PRE12]

[PRE13]

[PRE14]

[PRE15]

从前面的代码中，我们看到 `WebApplication.CreateBuilder` 负责为应用程序提供默认配置。

配置的加载顺序如下：

1.  `MemoryConfigurationProvider`：此提供程序从内存集合中加载配置，作为配置键值对。

1.  `ChainedConfigurationProvider`: 这添加主机配置并将其设置为第一个来源。有关主机配置的更多详细信息，您可以参考此链接：[https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0)。

1.  `JsonConfigurationProvider`: 这将从 `appsettings.json` 文件中加载配置。

1.  `JsonConfigurationProvider`: 这将从 `appsettings.Environment.json` 文件中加载配置；在 `appsettings.Environment.json` 中的 `Environment` 可以设置为开发、测试或生产。

1.  `EnvironmentVariablesConfigurationProvider`: 这将加载环境变量配置。

如本节开头所述，配置在来源中指定为键值对。后来添加的配置提供程序（按顺序）覆盖了之前的键值对设置。例如，如果您在 `MemoryConfigurationProvider` 和 `JsonConfigurationProvider` 中都有 `DbConnectionString` 键，则 `JsonConfigurationProvider` 中的 `DbConnectionString` 键的值将覆盖 `MemoryConfigurationProvider` 的键值对设置。

当您调试 `Program.cs` 代码时，您可以看到由 `CreateDefaultBuilder` 提供的默认配置被注入到配置中，如下所示：

![图 6.2 – 默认配置来源

](img/Figure_6.2_B18507.jpg)

图 6.2 – 默认配置来源

在下一节中，我们将看看如何添加我们应用程序所需的配置。

## 添加配置

如前一小节所示，有多种配置来源可用。在现实世界的项目中，`appsettings.json` 文件是最广泛使用的，用于添加应用程序所需的配置，除非它是秘密信息，不能以纯文本形式存储。

让我们看看在需要配置的几个常见场景：

+   如果我们需要一个 `ApplicationInsights` 仪表化密钥来添加应用程序遥测，这可以是配置的一部分

+   如果我们有需要调用的依赖服务，这些服务可以是配置的一部分

这些配置可能因环境而异（开发环境和生产环境中的值不同）。

您可以将以下配置添加到 `appsettings.json` 文件中，以便在发生更改时直接更新它，并在无需重新编译和部署的情况下开始使用它：

[PRE16]

[PRE17]

[PRE18]

[PRE19]

[PRE20]

[PRE21]

[PRE22]

[PRE23]

[PRE24]

[PRE25]

[PRE26]

[PRE27]

[PRE28]

[PRE29]

[PRE30]

[PRE31]

[PRE32]

[PRE33]

[PRE34]

[PRE35]

[PRE36]

[PRE37]

[PRE38]

[PRE39]

[PRE40]

[PRE41]

[PRE42]

[PRE43]

[PRE44]

[PRE45]

[PRE46]

[PRE47]

[PRE48]

从前面的代码中，我们看到我们为 `ApplicationInsights` 仪表化密钥添加了一个键值对，其中键是 `InstrumentationKey` 字符串，值是应用程序需要用于在 `ApplicationInsights` 中仪表化遥测的实际仪表化密钥。在 `ApiConfigs` 部分，我们以分层顺序添加了多个键值对，包括调用我们的依赖服务所需的配置。

在下一节中，我们将看到如何读取我们已添加的配置。

## 读取配置

我们已经看到了如何将配置添加到 `appsettings.json`。在本节中，我们将看到如何使用不同的选项在我们的项目中读取它们。

在 `Program.cs` 中由 `WebApplication.CreateBuilder` 提供的 `builder.Configuration` 对象实现了 `Microsoft.Extensions.Configuration.IConfiguration` 类型，您有以下选项可用于读取 `IConfiguration`：

[PRE49]

[PRE50]

[PRE51]

[PRE52]

[PRE53]

[PRE54]

[PRE55]

[PRE56]

[PRE57]

[PRE58]

[PRE59]

[PRE60]

[PRE61]

[PRE62]

[PRE63]

[PRE64]

[PRE65]

[PRE66]

[PRE67]

[PRE68]

[PRE69]

[PRE70]

[PRE71]

[PRE72]

[PRE73]

[PRE74]

[PRE75]

[PRE76]

[PRE77]

[PRE78]

让我们看看如何利用 `Iconfiguration` 中的这些选项来读取我们在上一节中添加的配置，*添加配置*。

要从 `appsettings.json` 中读取 `ApplicationInsights` 仪表化密钥，我们可以在 `Program.cs` 中使用以下代码使用 `string this[string key] { get; set; }` 选项：

[PRE79]

要读取 `ApiConfigs`，我们可以使用以下代码。我们可以使用分隔符在配置键中读取层次化配置：

[PRE80]

注意

使用分隔符这种方式读取容易出错且难以维护。首选的方法是使用 ASP.NET Core 提供的 **options** 模式。而不是逐个读取每个键/设置值，选项模式使用类，这也会为您提供对相关设置的强类型访问。

当配置设置通过场景隔离到强类型类中时，应用程序遵循两个重要的设计原则：

+   **接口隔离原则**（**ISP**）或封装原则

+   关注点分离

使用 ISP 或封装，您通过一个定义良好的接口或契约读取配置，并且只依赖于您需要的配置设置。此外，如果有一个非常大的配置文件，这将有助于关注点分离，因为应用程序的不同部分不会依赖于相同的配置，从而允许它们解耦。让我们看看我们如何利用代码中的选项模式。

您可以创建以下 `ApiConfig` 和 `ApiUrl` 类并将它们添加到您的项目中：

[PRE81]

[PRE82]

[PRE83]

[PRE84]

[PRE85]

[PRE86]

[PRE87]

[PRE88]

[PRE89]

[PRE90]

[PRE91]

在 `Program.cs` 中添加以下代码以使用 `GetSection` 方法读取配置，然后调用 `Bind` 以将配置绑定到我们已有的强类型类：

[PRE92]

[PRE93]

`GetSection` 将会读取 `appsettings.json` 中指定的特定部分。`Bind` 将尝试通过匹配属性名到配置键来将给定的对象实例绑定到配置值。`GetSection(string sectionName)` 如果请求的部分不存在，将返回 `null`。在实际的程序中，请确保您添加了空值检查。

在本节中，我们看到了如何通过使用配置 API 添加和读取 `appsettings.json` 中的数据。我还提到我们应该使用 `appsettings.json` 用于纯文本，而不是用于机密。在下一节中，我们将探讨内置配置提供程序以及如何使用 Azure Key Vault 配置提供程序添加和读取机密。

# 利用内置配置提供程序

除了 `appsettings.json` 之外，还有多个配置源可用，.NET 6 提供了多个内置配置提供程序来读取它们。以下是为 .NET 6 可用的内置提供程序：

+   Azure Key Vault 配置提供程序从 Azure Key Vault 中读取配置。

+   文件配置提供程序从 INI、JSON 和 XML 文件中读取配置。

+   命令行配置提供程序从命令行参数中读取配置。

+   环境变量配置提供程序从环境变量中读取配置。

+   内存配置提供程序从内存集合中读取配置。

+   Azure 应用配置提供程序从 Azure 应用配置中读取配置。

+   按文件密钥配置提供程序从目录的文件中读取配置。

让我们看看如何利用 Azure Key Vault 配置提供程序和文件配置提供程序，因为与其它提供程序相比，它们更为重要且更广泛地被使用。

注意

您可以使用以下链接了解我们在此处未详细介绍的其它配置提供程序：https://docs.microsoft.com/en-us/dotnet/core/extensions/configuration-providers。

## Azure Key Vault 配置提供程序

Azure Key Vault 是一种基于云的服务，它提供了一个集中的配置源，用于安全地存储密码、证书、API 密钥和其他机密。这有助于确保我们的应用程序安全并符合安全漏洞的合规性。让我们看看如何创建一个密钥库，向其中添加一个机密，并使用 Azure Key Vault 配置提供程序从应用程序中访问它。

### 创建密钥库并添加机密

在本节中，我们将使用 Azure Cloud Shell 创建一个密钥库并添加一个机密。Azure Cloud Shell 是基于浏览器的，可以用来管理 Azure 资源。以下是需要您采取的步骤列表：

1.  使用 [https://portal.azure.com](https://portal.azure.com) 登录到 Azure 门户。在门户页面上选择云 Shell 图标：

![图 6.3 – Azure 云 Shell

](img/Figure_6.3_B18507.jpg)

图 6.3 – Azure 云 Shell

1.  您将获得选择 **Bash** 或 **PowerShell** 的选项。选择 **PowerShell**。您可以在任何时候更改外壳：

![图 6.4 – Azure 云 Shell 选项 – PowerShell 和 Bash

](img/Figure_6.4_B18507.jpg)

图 6.4 – Azure 云 Shell 选项 – PowerShell 和 Bash

1.  使用以下命令创建一个资源组：

    [PRE94]

我为这个演示实际运行的命令如下：

[PRE95]

`{RESOURCE GROUP NAME}` 代表新资源组的资源组名称，而`{LOCATION}`代表Azure区域（数据中心）。

1.  使用以下命令在资源组中创建密钥保管库：

    [PRE96]

这里是我在此演示中实际运行的命令：

[PRE97]

`{KEY VAULT NAME}` 是新密钥保管库的唯一名称。

`{RESOURCE GROUP NAME}` 是在上一步骤中创建的新资源组的资源组名称。

`{LOCATION}` 是Azure区域（数据中心）。

1.  使用以下命令在密钥保管库中以名称-值对的形式创建密钥：

    [PRE98]

这里是我在此演示中实际运行的命令：

[PRE99]

`{KEY VAULT NAME}` 与您在上一步骤中创建的密钥保管库名称相同。

`SecretName` 是您的密钥名称。

`SecretValue` 是您的密钥值。

我们现在已成功创建了一个名为`TestKeyVaultForConfig`的密钥保管库，并使用Azure Cloud Shell添加了一个密钥为`TestKey`、值为`TestValue`的密钥：

![图6.5 – Azure密钥保管库密钥

![图6.5 – 图6.5_B18507.jpg](img/Figure_6.5_B18507.jpg)

图6.5 – Azure密钥保管库密钥

您也可以使用Azure **命令行界面**（**CLI**）创建和管理Azure资源。您可以在以下位置了解更多关于Azure CLI的信息：https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest。

在下一节中，我们将看到如何让我们的应用程序访问密钥保管库。

### 授予应用程序访问密钥保管库的权限

在本节中，让我们看看我们的`TestConfiguration` Web API如何通过以下步骤访问密钥保管库：

1.  在**Azure Active Directory**（**AAD**）中注册`TestConfiguration`应用程序并创建一个身份。使用https://portal.azure.com登录Azure门户。

1.  导航到**Azure Active Directory** | **应用程序注册**。点击**新建注册**：

![图6.6 – AAD新应用程序注册

![图6.6 – 图6.6_B18507.jpg](img/Figure_6.6_B18507.jpg)

图6.6 – AAD新应用程序注册

1.  填写默认值并点击**注册**，如图下所示截图，并记下**应用程序（客户端）ID**值。稍后访问密钥保管库时需要使用此值：

![图6.7 – AAD注册完成

![图6.7 – 图6.7_B18507.jpg](img/Figure_6.7_B18507.jpg)

图6.7 – AAD注册完成

1.  点击**证书和密钥**（**1**） | **新建客户端密钥**（**2**），输入**描述**（**3**）值，然后点击**添加**（**4**），如图下所示截图。记下在**新客户端密钥**下显示的**AppClientSecret**值，这是应用程序在请求令牌时用来证明其身份的：

![图6.8 – 为其身份创建AAD新应用程序密钥

![图6.8 – 图6.8_B18507.jpg](img/Figure_6.8_B18507.jpg)

图6.8 – 为其身份创建AAD新应用程序密钥

1.  使用访问策略授予应用程序访问密钥保管库的权限。搜索您刚刚创建的密钥保管库并选择它：

![图6.9 – 密钥保管库搜索

![图6.9 – 图6.9_B18507.jpg](img/Figure_6.9_B18507.jpg)

图6.9 – 密钥保管库搜索

1.  在密钥保管库属性中，选择 **设置** 下的 **访问策略**，然后点击 **添加访问策略**：

![Figure 6.10 – Key Vault 访问策略

![img/Figure_6.10_B18507.jpg]

Figure 6.10 – Key Vault 访问策略

1.  在 **选择主体** 字段中，搜索您的应用程序并选择您的应用程序访问 Key Vault 所需的权限，然后点击 **添加**：

![Figure 6.11 – 添加访问策略

![img/Figure_6.11_B18507.jpg]

Figure 6.11 – 添加访问策略

1.  添加策略后，您必须保存它。这将完成授予您的应用程序访问 Key Vault 的过程。

我们现在已授予我们的应用程序访问 Key Vault 的权限。在下一节中，我们将看到如何使用 Azure Key Vault 配置提供程序从我们的应用程序访问 Key Vault。

### 利用 Azure Key Vault 配置提供程序

在本节中，我们将对我们的应用程序进行配置和代码更改，以利用 Azure Key Vault 配置提供程序并从 Key Vault 获取密钥，如下所示：

![Figure 6.12 – 开发期间访问 Key Vault

![img/Figure_6.12_B18507.jpg]

Figure 6.12 – 开发期间访问 Key Vault

以下是要更改的列表：

1.  将密钥保管库名称、从 *Figure 6.7* 记下的 `AppClientId` 值和从 AAD 的 *Figure 6.8* 记下的 `AppClientSecret` 值添加到您的 `TestConfiguration` Web API 的 `appsettings.json` 文件中：

![Figure 6.13 – appsettings.json 中的 Key Vault 部分

![img/Figure_6.13_B18507.jpg]

Figure 6.13 – appsettings.json 中的 Key Vault 部分

1.  安装以下 NuGet 包：

    +   `Microsoft.Azure.KeyVault`

    +   `Microsoft.Extensions.Configuration.AzureKeyVault`

    +   `Microsoft.Azure.Services.AppAuthentication`

1.  将 `Program.cs` 更新为利用 Azure Key Vault 配置提供程序来使用您的密钥保管库。以下代码将 Azure Key Vault 添加为另一个配置源，并使用 Azure Key Vault 配置提供程序获取所有配置：

    [PRE100]

1.  将 `WeatherForecastController.cs` 更新为从 Key Vault 读取密钥，如下所示：

    [PRE101]

按照此处共享的代码示例包含所有引用。您可以运行应用程序并查看结果：

![Figure 6.14 – Key Vault 结果

![img/Figure_6.14_B18507.jpg]

Figure 6.14 – Key Vault 结果

应用程序将能够使用 Azure Key Vault 配置提供程序访问密钥保管库并获取密钥。这非常简单，因为所有繁重的工作都由 .NET 6 完成，我们只需要安装 NuGet 包并添加几行代码。然而，你现在可能正在思考 `AppClientId` 和 `AppClientSecret` 是如何添加到 `appsettings.json` 配置文件中，以及这种方式并不十分安全。你完全正确。

我们可以通过两种方式解决这个问题：

+   一种选择是将这些值加密并存储在 `appsettings.json` 中；然后可以在代码中读取和解密。

    参考

    对于开发中应用程序机密的安全存储，请参阅 https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-6.0&tabs=windows。

+   另一个选项是使用托管身份访问 Azure 资源，这允许应用程序使用 AAD 认证对 Azure Key Vault 进行身份验证，而无需凭证（应用程序 ID 和密码/客户端密钥）。

您的应用程序可以使用其身份通过支持 AAD 认证的服务进行身份验证，例如 Azure Key Vault，这将帮助我们从代码中移除凭证：

![图 6.15 – 应用程序部署后生产环境中访问密钥保管库

](img/Figure_6.15_B18507.jpg)

图 6.15 – 应用程序部署后生产环境中访问密钥保管库

注意

这是我们将应用于已部署到生产环境的应用程序的最佳实践。在代码中管理凭证是一个常见的挑战，保持凭证的安全和保密是一个重要的安全要求。Azure 资源在 Azure Active Directory (AAD) 中的托管身份有助于解决这个挑战。托管身份为 Azure 服务提供在 AAD 中自动管理的身份。您可以使用此身份对支持 AAD 认证的任何服务进行身份验证，包括密钥保管库，而无需在您的代码中包含任何凭证。

您可以在此处了解更多关于托管身份的信息：https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview。

在本节中，我们学习了如何创建密钥保管库，如何将机密添加到密钥保管库，如何将我们的`TestConfiguration` Web API 注册到 Azure Active Directory (AAD)，如何创建机密或身份，如何让`TestConfiguration` Web API 获取对密钥保管库的访问权限，以及如何使用 Azure Key Vault 配置提供程序从我们的代码中访问密钥保管库。您还可以通过使用 Visual Studio 连接服务将密钥保管库添加到您的 Web 应用程序中，具体操作请参阅 https://docs.microsoft.com/en-us/azure/key-vault/general/vs-key-vault-add-connected-service:

![图 6.16 – Azure Key Vault 作为连接服务

](img/Figure_6.16_B18507.jpg)

图 6.16 – Azure Key Vault 作为连接服务

在下一节中，我们将了解如何利用文件配置提供程序。

## 文件配置提供程序

文件配置提供程序帮助我们从文件系统中加载配置。JSON 配置提供程序和 XML 配置提供程序分别从文件配置提供程序类继承，用于从 JSON 文件和 XML 文件中读取键值对配置。让我们看看如何将它们作为`CreateHostBuilder`的一部分添加到配置源中。

### JSON 配置提供程序

可以使用以下代码在`Program.cs`中配置 JSON 配置提供程序：

[PRE102]

[PRE103]

[PRE104]

[PRE105]

[PRE106]

在这种情况下，JSON 配置提供者将加载 `AdditionalConfig.json` 文件，`AddJsonFile` 方法的三个参数为我们提供了指定文件名、文件是否可选以及文件在文件被修改时是否必须重新加载的选项。

以下是一个 `AdditionalConfig.json` 示例文件：

[PRE107]

然后，我们将更新 `WeatherForecastController.cs` 以从加载自 `AdditionalConfig.json` 配置文件的配置中读取键值对，如下所示：

[PRE108]

[PRE109]

[PRE110]

[PRE111]

[PRE112]

[PRE113]

[PRE114]

[PRE115]

[PRE116]

您可以运行应用程序并查看结果。应用程序将能够访问 `AdditionalConfig.json` 文件并读取配置。在下一节中，我们将探讨 XML 配置提供者。

### XML 配置提供者

我们将在项目中添加一个名为 `AdditionalXMLConfig.xml` 的新文件和所需的配置。然后，可以使用以下代码在 `Program.cs` 中配置 XML 配置提供者，以从我们添加的文件中读取：

[PRE117]

[PRE118]

[PRE119]

[PRE120]

[PRE121]

在这种情况下，XML 配置提供者将加载 `AdditionalXMLConfig.xml` 文件，三个参数为我们提供了指定 XML 文件、文件是否可选以及文件在发生任何更改时是否必须重新加载的选项。

以下是一个 `AdditionalXMLConfig.xml` 示例文件：

[PRE122]

[PRE123]

[PRE124]

[PRE125]

[PRE126]

接下来，我们将更新 `WeatherForecastController.cs` 以从 `AdditionalXMLConfig.xml` 加载的配置中读取键值对，如下所示：

[PRE127]

[PRE128]

[PRE129]

[PRE130]

[PRE131]

[PRE132]

[PRE133]

您可以运行应用程序并查看结果。应用程序将能够访问 `AdditionalXMLConfig.xml` 并读取配置。在 .NET 6 中，JSON 配置文件和 JSON 配置提供者可用，因此您不需要 XML 配置文件和 XML 配置提供者。话虽如此，我们刚才讨论的内容是针对那些喜欢 XML 文件和开闭标签的人来说的，例如。

在下一节中，我们将探讨为什么需要自定义配置提供者以及如何构建一个。

# 构建自定义配置提供者

在上一节中，我们讨论了 .NET 6 中的内置或现有配置提供者。存在一些场景，许多系统在数据库中维护应用程序配置设置。这些设置可以由管理员通过门户管理，或者由支持工程师通过运行数据库脚本来创建/更新/删除应用程序配置设置。

.NET 6 并没有内置的提供者来从数据库中读取配置。让我们看看如何通过以下步骤构建一个自定义配置提供者来从数据库中读取：

+   **实现配置源**：为了创建配置提供者的实例

+   **实现配置提供者**：为了从适当的源加载配置

+   **实现配置扩展**：为了将配置源添加到配置构建器中

让我们从配置源开始。

## 配置源

配置源的责任是创建配置提供者的一个实例并将其返回到源。它需要继承自`IConfigurationSource`接口，这要求我们实现`ConfigurationProvider Build(IConfigurationBuilder builder)`方法。

在`Build`方法实现中，我们需要创建自定义配置提供者的一个实例并返回它。还应该有构建构建器所需的参数。在这种情况下，因为我们正在构建自定义SQL配置提供者，所以重要的参数是连接字符串和SQL查询。以下代码片段展示了`SqlConfigurationSource`类的一个示例实现：

[PRE134]

[PRE135]

[PRE136]

[PRE137]

[PRE138]

[PRE139]

[PRE140]

[PRE141]

[PRE142]

[PRE143]

[PRE144]

[PRE145]

[PRE146]

[PRE147]

[PRE148]

[PRE149]

如您所见，实现这个方法非常简单且容易。您获取构建提供者所需的参数，然后创建提供者的新实例，然后返回这些参数。让我们看看如何在下一节中构建一个SQL配置提供者。

## 配置提供者

配置提供者的责任是从适当的位置加载所需的配置并返回相同的配置。它需要继承自`IConfigurationProvider`接口，这要求我们实现`Load()`方法。配置提供者类可以继承自`ConfigurationProvider`基类，因为它已经实现了`IConfigurationProvider`接口中的所有方法。这将帮助我们节省时间，因为我们不需要实现未使用的方法，而只需实现`Load`方法即可。

在`Load`方法实现中，我们需要有从源获取配置数据的逻辑。在这种情况下，我们将执行一个查询从SQL存储中获取数据。以下代码片段展示了`SqlConfigurationProvider`类的一个示例实现：

[PRE150]

[PRE151]

[PRE152]

[PRE153]

[PRE154]

[PRE155]

[PRE156]

[PRE157]

[PRE158]

[PRE159]

[PRE160]

[PRE161]

[PRE162]

[PRE163]

[PRE164]

[PRE165]

[PRE166]

[PRE167]

[PRE168]

[PRE169]

[PRE170]

[PRE171]

[PRE172]

[PRE173]

[PRE174]

[PRE175]

[PRE176]

[PRE177]

[PRE178]

[PRE179]

[PRE180]

[PRE181]

[PRE182]

[PRE183]

让我们看看如何在下一节中构建配置扩展。

## 配置扩展

与其他提供者一样，我们可以使用扩展方法将配置源添加到配置构建器中。

注意

扩展方法是静态方法，您可以在不修改或重新编译原始类的情况下向现有类中添加方法。

以下代码片段展示了在配置构建器中`SqlConfigurationExtensions`类的一个示例实现：

[PRE184]

[PRE185]

[PRE186]

[PRE187]

[PRE188]

[PRE189]

[PRE190]

[PRE191]

[PRE192]

[PRE193]

[PRE194]

[PRE195]

[PRE196]

扩展方法将减少我们应用程序启动时的代码量。

我们可以在`Program.cs`中添加引导代码，就像我们为其他配置提供者添加的那样，如下所示：

[PRE197]

以下截图显示了数据库中的一些示例配置设置。您可以在`config.AddSql()`中传递适当的连接字符串和SQL查询，并从数据库加载以下配置。SQL查询可能是一个简单的`select`语句，用于读取所有键值对，就像以下截图所示：

![图6.17 – 数据库配置设置

![图片](img/Figure_6.17_B18507.jpg)

图 6.17 – 数据库配置设置

按照以下方式更新 `WeatherForecastController.cs` 以从 SQL 配置提供程序加载的配置中读取键值对：

[PRE198]

[PRE199]

[PRE200]

[PRE201]

[PRE202]

[PRE203]

[PRE204]

你可以运行应用程序并查看结果。应用程序将能够访问 SQL 配置并读取配置。

这只是一个自定义配置提供程序的例子。你可能能够想到不同的场景，在这些场景中你会构建其他不同的自定义配置提供程序，例如从 CSV 文件中读取或从 JSON 或 XML 文件中读取加密值并解密它们。

# 摘要

在本章中，我们看到了 .NET 6 中配置的工作方式，如何向应用程序提供默认配置，如何以分层顺序添加键值对配置，如何读取配置，如何利用 Azure Key Vault 配置提供程序和文件配置提供程序，以及如何构建自定义配置提供程序以从 SQL 数据库中读取。你现在拥有了根据具体需求在项目中实现不同配置所需的知识。

在下一章中，我们将学习关于日志以及它在 .NET 6 中的工作方式。

# 问题

在阅读本章后，你应该能够回答以下问题：

1.  在 .NET 6 中，负责提供应用程序默认配置的是什么？

a. `CreateDefaultBuilder`

b. `ChainedConfigurationProvider`

c. `JsonConfigurationProvider`

d. 所有上述选项

**答案：a**

1.  以下哪个说法是不正确的？

a. Azure Key Vault 配置提供程序从 Azure Key Vault 读取配置。

b. 文件配置提供程序从 INI、JSON 和 XML 文件中读取配置。

c. 命令行配置提供程序从数据库中读取配置。

d. 内存配置提供程序从内存集合中读取配置。

**答案：c**

1.  用于在运行时访问配置并通过依赖注入注入的接口是什么？

a. `IConfig`

b. `IConfiguration`

c. `IConfigurationSource`

d. `IConfigurationProvider`

**答案：b**

1.  在生产中存储密钥推荐使用哪个提供程序/源？

a. 从 `appsettings.json` 中的 JSON

b. 从 XML 文件中的 `FileConfiguration`

c. 从 `AzureKeyVault` 的 `AzureKeyVaultProvider`

d. 从命令行来的命令行配置提供程序

**答案：c**

# 进一步阅读

+   https://docs.microsoft.com/en-us/aspnet/core/blazor/fundamentals/configuration?view=aspnetcore-6.0

+   https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options?view=aspnetcore-6.0
