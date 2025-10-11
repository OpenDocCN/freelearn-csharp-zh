# 在 Azure 上部署服务

在本章中，你将学习如何在 Microsoft Azure 上部署目录 Web 服务。尽管我们将重点关注 Microsoft Azure 云提供商，并且大部分说明将紧密关联该平台，但一些概念可以应用于多个云提供商：容器正在成为在云上构建和运行应用程序和 Web 服务的常见方式，因此，每个云提供商都提供略微不同的服务和产品来托管容器。本章不会深入探讨 Microsoft Azure；它将概述 Azure 云的**Azure 容器实例**（**ACI**）和 Azure 应用服务功能。

在本章中，我们将涵盖以下主题：

+   Azure 入门

+   将容器推送到 Azure 容器注册库

+   配置 ACI

+   配置应用服务

# Azure 入门

如我们之前提到的，Microsoft Azure 是由微软构建的云平台。Azure 提供了广泛的 IT 产品、技术和集成工具。虚拟机、无服务器技术、数据库和机器学习管道只是它提供的产品中的一部分。在本章中，我们将重点关注 Azure 提供的一些服务，例如**容器实例**、**应用服务**和**Azure SQL 数据库**。

让我们从创建订阅开始。当新用户首次创建 Azure 账户时，微软允许我们尝试 Azure 服务。您可以在 [https://azure.microsoft.com/free/](https://azure.microsoft.com/free/) 注册新的 Microsoft Azure 账户。

注册过程将要求您提供一些个人详细信息，以及有效的电话号码和有效的信用卡。默认情况下，微软提供 12 个月的流行免费服务，加上 30 天的 170 欧元 Azure 服务和 25+ 永久免费的服务。这使得新开发者或工程师能够轻松测试/学习如何使用 Azure 的一些新服务。

完成注册过程后，您将能够使用您刚刚创建的账户登录到 Azure 门户 ([https://portal.azure.com](https://portal.azure.com/))。

Azure CLI 是微软官方用于管理 Azure 资源的 CLI；它几乎适用于所有操作系统，并且是 Azure SDK 的一部分。Azure SDK 是跨平台的；因此，您可以通过三种不同的方式在 Windows、macOS 或 Linux 上安装它：

| **平台** | **命令** | **要求** |
| --- | --- | --- |
| Linux | `curl -L https://aka.ms/InstallAzureCli &#124; bash` | 您需要一些预先安装的软件，即Python 2.7或Python 3.x([https://www.python.org/downloads/](https://www.python.org/downloads/))、libffi([https://sourceware.org/libffi/](https://sourceware.org/libffi/))和OpenSSL 1.0.2([https://www.openssl.org/source/](https://www.openssl.org/source/))。有关更多信息，请访问[https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux?view=azure-cli-latest](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux?view=azure-cli-latest)。 |
| macOS | `brew update && brew install azure-cli` | 您需要`brew`，它应该已经安装到您的机器上。有关`brew`的更多信息，请访问[https://brew.sh/](https://brew.sh/)。 |
| Windows | [https://aka.ms/installazurecliwindows](https://aka.ms/installazurecliwindows) | Microsoft为Windows平台提供了一个MSI。 |

Azure SDK及其CLI提供了您管理Azure服务所需的所有命令行工具。让我们首先使用CLI进行登录。我们可以通过执行以下命令来完成此操作：

[PRE0]

上述命令将打开浏览器窗口，并将您带到Microsoft Azure门户的登录页面，并将会话保存到您的本地环境中。在下一节中，我们将看到如何使用CLI将我们在前几章中构建的容器镜像推送到Azure的容器注册表服务。

# 将容器推送到Azure容器注册表

在本节中，我们将专注于将我们的容器部署到Microsoft Azure。这个过程涉及到一些云提供商提供的开箱即用的资源和服务。以下是我们将要构建的架构方案的概述图：

![](img/d32f4269-5723-4e15-92fc-fa0b1d67eddc.png)

让我们看看参与此架构的不同组件：

+   **Azure容器注册表**是一个基于开源Docker Registry 2.0的托管Docker注册表服务([https://docs.docker.com/registry/](https://docs.docker.com/registry/))。您可以使用Azure容器注册表来存储、管理和使用您的私有Docker容器镜像。我们将使用它来保存与我们的自定义镜像相关的图像，例如`catalog_api`镜像，并使它们可供其他云服务使用。

+   容器Web应用允许我们使用我们的容器，并将它们作为Web应用部署到**App Service**。此外，它消除了耗时的基础设施管理任务，如更新、扩展以及在一般情况下管理基础设施。

+   应用程序的其他依赖项，如`catalog_esb`和`catalog_cache`，将从公共Docker Hub(*[https://hub.docker.com](https://hub.docker.com))*获取镜像。

让我们继续创建一个Azure容器注册表。在本章的后续内容中，我们将使用该注册表来推送和拉取`catalog_api`的镜像。

# 创建 Azure 容器注册表

要创建一个新的 Azure 容器注册表，我们应该首先创建一个新的资源组。资源组是 Azure 资源管理中的一个基本概念：它们允许我们根据管理、部署和计费原因将一组资源分组在一起。一般来说，所有具有相同生命周期的资源都应该被分组到同一个资源组中。让我们开始吧：

1.  首先，在 Azure CLI 中使用以下命令创建一个资源组：

[PRE1]

之前的命令在我们的账户中创建了一个名为 `handsOn` 的新资源组，该资源组存储在西欧地区。

1.  接下来，我们将通过执行以下命令来创建 Azure 容器注册表：

[PRE2]

之前的命令在名为 `handsOn` 的资源组下创建了一个新的 Azure 容器注册表，并使用了我们选择的名称。它还定义了该资源的 **库存单位** (**SKU**)——在我们的例子中，是基本版本。

SKU 通常指代产品的特定变体以及识别该类型产品的所有属性。同样，Microsoft Azure 使用这个术语来识别特定的可购买商品或服务。

现在我们已经创建了一个 Azure 容器注册表，让我们将 `catalog_api` 镜像推送到注册表中。为了解决我们容器的其他依赖项，我们将创建另一个 `appsettings.json` 文件，专门用于 `Stage` 环境。因此，我们将设置 `ASPNETCORE_ENVIRONMENT` 变量为 `Stage` 以应用容器所需的连接字符串。我们可以通过以下方式创建 `appsettings.Stage.json` 文件：

[PRE3]

之前的 `appsettings.json` 文件定义了 `catalog_db`、`catalog-esb` 和 `catalog-cache` 容器的端点。每个端点都由我们将要创建的容器的名称，后跟语法——`<region_name>.azurecontainer.io` 组成。第一部分代表地区，接着是使用服务的子域，在我们的例子中是 `azurecontainer.io`。让我们继续定义将我们的本地镜像推送到之前创建的容器注册表的步骤：

1.  让我们先使用以下命令在容器注册表中验证 Azure CLI：

[PRE4]

这应该在 CLI 中返回一个登录成功的消息。

1.  之后，我们可以通过准备我们的服务 Docker 镜像并触发以下命令在 `Catalog.API` 文件夹中构建镜像来继续：

[PRE5]

这个指令基于我们在项目文件夹中已有的 Dockerfile 创建一个新的 Docker 镜像。镜像的名称将取决于 `docker-compose.yml` 文件中指定的名称以及 `.env` 文件中指定的 `COMPOSE_PROJECT_NAME`：如果 `COMPOSE_PROJECT_NAME` 是 `store`，那么命令将创建一个名为 `store_catalog_api` 的镜像。

1.  可以通过执行 `docker images` 命令来验证生成的镜像：

[PRE6]

1.  必须获取Azure容器注册表服务器的地址，以便我们可以将本地镜像推送到注册表。我们可以通过将我们刚刚创建的容器标记为之前创建的Azure容器注册表的地址来继续操作：

[PRE7]

在标记镜像并使用`docker push`命令后，Docker将开始将容器上传到我们的Azure容器存储库。因此，我们可以在Azure提供的所有服务中使用我们的容器镜像。此上传通常需要一些时间，具体取决于镜像的大小和您的互联网连接质量。上传完成后，可以通过浏览Azure门户的容器注册表部分来检查结果（[https://portal.azure.com/](https://portal.azure.com/))：

![图片](img/4ddbea9e-5b47-42b7-81ef-2d5ba99cd8df.png)

在这种情况下，我们可以看到，在`handsOn`资源组下创建了一个名为`handsonaspnetcoreacr`的容器注册表。最终，我们可以选择直接从门户创建或管理容器注册表。现在我们已经推送了容器，我们可以继续配置ACI。

# 配置Azure容器实例

微软Azure的ACI服务为我们提供了一种快速简便的方式在云中运行容器，无需担心虚拟机的管理部分，也不必学习新的工具。此服务旨在尽可能快速，简化在云中启动和运行容器的过程。此外，可以通过执行简单的Azure CLI命令来启动容器，例如以下命令：

[PRE8]

ACI服务是测试和运行Azure中容器的理想服务。因此，ACI服务允许我们通过利用每秒计费来降低基础设施成本。因此，ACI服务也是持续集成和持续管道目的的首选服务。以下步骤显示了如何在ACI上部署目录服务*：

1.  让我们先创建一个新的资源组，以便我们可以分组我们的容器。使用以下命令来完成此操作：

[PRE9]

1.  我们可以通过以下命令获取容器注册表的注册表用户名和密码，以继续操作：

[PRE10]

1.  创建组后，我们需要使用GitHub仓库中名为`aci-deploy.sh`的Bash脚本来执行Azure CLI命令：

[PRE11]

脚本主要运行五个不同的指令来创建这些容器的新的实例：

+   +   它声明有关容器的信息，例如资源组、分配给容器的名称以及一些额外的环境变量。

    +   它执行`az container create`命令来创建和运行`microsoft/mssql-server-linux`。

    +   它执行 `az container create` 指令来创建和运行 `rabbitmq:3-management-alpine` 镜像，并使用 `rabbitmq_user` 和 `rabbitmq_pass` 环境变量来设置 RabbitMQ 实例的默认用户。

    +   它使用 `redis:alpine` 部署 Redis 缓存实例。

    +   最后，它通过指定注册表 URL 来创建和部署已存在于 Azure 容器注册表存储库中的 `catalog_api` 镜像。

请注意，执行顺序遵循这些容器依赖项的逻辑；因此，API 容器是最后一个运行的。

注意，为了使演示尽可能简单，`aci-deploy.sh` 脚本使用 `--ip-address public` 创建目录服务容器，这意味着任何人都可以访问我们的容器。在生产环境中，强烈不建议直接暴露 API 而没有任何反向代理和 API 网关，以避免将容器暴露给外部世界。

现在我们已经执行了脚本，我们可以通过登录到 Azure 门户 ([https://portal.azure.com/#](https://portal.azure.com/#)) 并在容器实例部分检查我们的容器来查看结果：

![图片](img/69924149-046e-4903-b729-14b57491cd2b.png)

如您所见，有四个容器实例正在运行。它们都在使用 `--dns-name-label` 参数在 DNS 上运行，并且可以通过它们的地址相互访问。因此，可以使用由我们的 shell 脚本生成的地址调用容器 API。我们还可以通过单击容器的名称来检查与容器相关的统计信息和属性：

![图片](img/4a71ce45-9376-4faf-be99-ef3ee41a0a4b.png)

最后，我们可以从浏览器中调用健康检查的 HTTP 路由来验证所有依赖项是否正确：

[PRE12]

前面的过程描述了如何将目录服务部署到 ACI 产品中。尽管 ACI 功能强大且易于部署，但它们缺少一些最小化的开箱即用功能，例如 SSH、监控和配置管理。因此，在生产环境中管理容器实例变得困难。在下一节中，我们将关注不同的托管过程，该过程使用应用服务技术来托管名为应用服务的应用程序。这种方式更专注于 Web 应用程序和 Web 服务的托管；因此，它提供了一套用于监控和配置应用程序的工具和功能。

# 配置应用服务

ACI的替代方案是应用服务。微软Azure最近发布了一个新功能，使我们能够使用**应用服务**部署Docker镜像。当你想要保持开发机器和生产环境相同的运行环境时，这种方法非常有用。与ACI*相比，应用服务为我们提供了一个管理容器运行的方式。它自带一些开箱即用的功能，例如SSL加密、监控、配置管理、远程调试以及应用扩展设置。除此之外，应用服务与Azure的其他产品紧密集成。因此，将其他服务轻松地连接到`catalog-srv`成为可能。例如，我们可能会选择运行我们的Azure SQL数据库解决方案，为目录服务设置一个完全管理的SQL数据库。Azure SQL提供了最广泛的SQL Server引擎兼容性；它使用你偏好的SQL工具简化了维护过程。

正如我们之前提到的，在不使用持久卷的情况下使用Docker集成持久数据存储并不容易。因此，在本节中，我们将探讨一种存储数据的替代方法。

让我们通过使用项目根目录中的`azuresql-deploy.sh`脚本创建一个新的Azure SQL数据库来继续操作：

[PRE13]

在前面的脚本中，`azuresql-deploy.sh`文件创建了托管`store`数据库和包含目录信息的有效数据库的逻辑服务器。首先，脚本通过创建资源组开始；然后继续创建Azure SQL元素。由于防火墙规则中指定的`start-ip`和`end-ip`都是`0.0.0.0`，因此该账户的所有Azure服务都可以连接到数据库。

通过这种方式，可以将之前创建的`catalog-srv`应用服务连接到数据库。

# 使用容器镜像创建应用服务

让我们逐步了解如何使用已发布并存在于Azure容器注册表中的镜像创建应用服务，即`<registry_name>.azurecr.io/catalog_api:v1`。作为第一步，我们需要使用以下命令创建一个应用服务计划：

[PRE14]

创建应用服务需要应用服务计划：它定义了一组用于运行同一计划中所有应用服务的计算资源。在这个例子中，我们将使用最基本的服务计划，可以使用以下标志指定：`--sku FREE`。此计划支持最多10个实例，并且不提供任何额外的自动扩展功能。

现在我们已经创建了所有必需的要求，我们可以通过执行位于项目根目录的`appservice-deploy.sh`文件来继续操作：

[PRE15]

前面的脚本使用 `az webapp create` 指令创建了 Web 应用，并在应用服务创建完成后，通过执行 `az webapp config appsettings set` 命令来设置正确的 ASP.NET Core 环境值。一旦脚本执行完毕，我们可以在门户中检查应用服务的状态：

![图片](img/59b4576b-52d0-43ac-94ab-20c0bd57bd3a.png)

此外，我们还可以通过调用健康检查 URL 来验证服务状态：`http://catalog-api.westeurope.azurecontainer.io/health`。

# 摘要

在本章中，我们看到了如何在 Microsoft Azure 中托管和运行目录服务项目。我们还学习了如何创建私有 Azure 容器注册库以及如何存储目录服务的 Docker 镜像。然后，我们向您展示了您可以使用的一些模式将自定义容器部署到云端，以及如何使用 Microsoft Azure 云提供商提供的服务来运行它们。最后，我们探讨了两种托管目录服务的方法：使用 ACI 产品和 Azure App Service，以及使用 Azure SQL 服务来存储数据。

在下一章中，我们将学习如何通过实现 Swagger 框架来使用 OpenAPI 规范来记录 API。
