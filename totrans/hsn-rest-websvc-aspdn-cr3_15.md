# 服务的容器化

上一章重点介绍了使用 ASP.NET Core 构建网络服务的一些高级主题。本章快速介绍了容器以及它们如何在本地沙盒环境中运行应用程序方面非常有用。本章的设计并不是要涵盖所有关于容器的内容；相反，它更多的是对它们的简要介绍。我们将学习如何在容器上运行目录服务。

本章将涵盖以下主题：

+   容器简介

+   如何在 Docker 上运行目录服务

+   .NET Core Docker 镜像概述

+   优化 Docker 镜像

# 容器简介

现在，分布式系统构成了每个应用程序的基础。反过来，分布式系统的基础是容器。容器化的目标是资源在隔离环境中运行。容器定义了分布式系统组件之间的边界和关注点的分离。这种关注点的分离是我们通过提供参数化配置来重用容器的方式。网络服务和网络应用的另一个重要特性是 *可伸缩性*。应该很容易扩展容器并创建新的实例。

Docker 是一个旨在使用容器创建、运行和部署应用程序的工具。Docker 也被称为推广这项技术的平台。在过去的几年里，Docker 已经成为了一个热门词汇，并且被相当多的公司、初创企业和开源项目所采用。许多项目都与 Docker 有关。

首先，让我们谈谈 **Moby** ([https://github.com/moby/moby](https://github.com/moby/moby))，这是一个由 Docker 创建的开源项目。随着大量的人和社区开始为该项目做出贡献，Docker 决定开发 Moby。Moby 项目的所有贡献，可以被视为 Docker 的一个研发部门，都是开源的。

Moby 是一个用于构建特定容器系统的框架。它提供了一组组件，这些组件是容器系统的基本要素：

+   操作系统和容器运行时

+   编排

+   基础设施管理

+   网络

+   存储

+   安全

+   构建

+   镜像分发

它还提供了构建这些组件所需的工具，以创建适用于每个平台和架构的可运行工件。

如果我们寻找更多的下游解决方案，我们有 Docker **社区版**（**CE**）和 Docker **企业版**（**EE**）版本，这些是使用 Moby 项目的产品。Docker CE 被小型团队和开发者用来构建他们的系统，而商业客户使用 Docker EE。两者都是推荐解决方案，允许我们在开发和企业环境中使用容器化。

本章将使用Docker CE对目录服务进行容器化。Docker CE和Docker EE可在Docker网站上找到：[https://www.docker.com/](https://www.docker.com/)。您可以通过遵循提供的步骤下载并安装社区版：[https://www.docker.com/get-started](https://www.docker.com/get-started)。

# Docker术语

在设置我们的服务使用Docker之前，让我们快速了解一下这项技术背后的术语。

让我们从定义容器镜像开始，这是创建容器实例所需的所有依赖项和工件列表。术语*容器镜像*不应与术语*容器实例*混淆，后者指的是容器镜像的单个实例*。

容器镜像的核心部分是Dockerfile。Dockerfile提供了构建容器的指令。例如，对于一个运行.NET Core解决方案的容器，它提供了恢复包和构建解决方案的命令。Docker还提供了一种对容器镜像进行标记并将它们分组到称为**仓库**的集合中的方法。仓库通常由注册表覆盖，允许访问特定的仓库。仓库可以是公共的或私有的，具体取决于它们的使用目的。例如，一家私人公司可以使用私有仓库为内部团队提供不同版本的容器。这种集中式思考方式非常强大，它为我们提供了重用容器的方法。公共仓库的主要例子是[https://hub.docker.com/](https://hub.docker.com/)，这是世界上最大的库，提供容器镜像：

![图片](img/c3f3196b-bd1e-44b6-b602-8c37057cdda5.png)

上述图表描述了本节中描述的组件之间的典型交互。客户端通常会提供一些Docker命令，这些命令在**Docker HOST**上被解析和执行。**Docker HOST**包含我们在本地机器上运行的**容器**以及这些容器使用的镜像。这些镜像通常来自Docker公共或私有仓库，这通常是Docker Hub网站或私人公司仓库。

与Docker相关的另一个重要术语是compose工具。Compose工具用于定义和运行一个多容器应用程序或服务。通常，composer会与一个不同格式的定义文件结合使用，例如一个YAML文件，该文件定义了容器组的结构。

Compose工具通常使用`docker-compose` CLI命令运行。在本章中，我们将使用`docker-compose`命令使我们的服务在本地环境中运行。

容器系统的一个基本部分是*编排器*。编排器简化了多容器系统的使用。此外，编排器通常应用于复杂系统。容器编排器的例子包括Kubernetes、Azure Service Fabric和Docker Swarm。

本书不会涵盖编排器的使用。本章提供了 Docker 和更广泛的容器化能力的高级概述。更复杂的 Docker 主题需要 DevOps 和系统工程技能。

让我们继续探讨容器化的力量以及如何快速使用它来构建我们的应用程序和服务。下一节将应用上一章中构建的目录服务的容器原则。下一节也将广泛使用容器技术来使我们的示例运行起来。

# 使用 Docker 运行目录服务

本节解释了如何将我们的目录服务与 Docker 结合起来，使其在本地运行。让我们首先检查目录服务背后的系统：

![图片](img/70b28f16-f147-46c6-9efd-3a732844e21c.png)

如前图所示，Web 服务部分运行在 `microsoft/dotnet` Docker 镜像上，数据源部分运行在 Microsoft SQL Server 实例上，使用的是 `microsoft/mssql-server-linux` Docker 镜像（我们已经在第 8 章[84b281bd-11a2-4703-81ae-ca080e2a267a.xhtml]，*构建数据访问层*中处理了 MSSQL 的容器化）。这两个镜像都已从 Docker Hub 中现有的公共 Microsoft 仓库下载；让我们看看如何使用 `docker-compose` 定义整个服务的基础设施。

首先，让我们在项目的根目录下创建一个包含以下内容的 `docker-compose.yml` 文件：

[PRE0]

前面的文件使用 YAML 语法定义了两个容器。第一个是 `catalog_api` 容器：它将用于托管基于 ASP.NET Core 框架构建的服务核心部分。第二个容器是 `catalog_db`，它使用 `microsoft/mssql-server-linux` 镜像（与我们在第 8 章[84b281bd-11a2-4703-81ae-ca080e2a267a.xhtml]，*构建数据访问层*中使用的一样）来设置 MSSQL 数据库。

此外，我们应在项目的根目录下创建以下文件夹结构：

[PRE1]

前面的文件夹将包含 `docker-compose.yml` 文件中指定的 API 和数据库容器的相关文件。让我们继续通过检查 `catalog_api` 容器的定义来深入了解：

[PRE2]

上述代码片段指定当前文件夹为构建上下文，并引用 `containers/api/Dockerfile` 文件来构建 Docker 镜像。它还通过以下语法引用环境变量文件：

[PRE3]

最后，它声明了一个名为 `my_network` 的网络下的容器，并使用 `ports:` 指令将端口 `5000` 暴露给宿主系统。

同样地，`catalog_db` 容器声明了与 `catalog_api` 容器相同的网络，并使用之前看到的方法指定了一个不同的环境变量文件。

在`docker-compose.yml`文件末尾，定义了`my_network`，它使用桥接驱动程序。桥接驱动程序是网络的默认选项。在同一桥接网络下的两个容器可以共享流量。

更多关于开箱即用的驱动程序类型的信息，请参阅以下链接：[https://docs.docker.com/network/](https://docs.docker.com/network/)。

# 定义环境变量

Docker提供了许多指定容器环境变量的方法。在前面指定的`docker-compose.yml`文件中，我们使用了`env_file`方法。此外，我们还可以通过在`catalog_api`容器的定义中指定的路径创建`api/api.env`文件来继续：

[PRE4]

文件语法期望文件中的每一行都遵循以下格式：`VAR=VAL`。在前面的例子中，我们正在定义ASP.NET Core运行服务所使用的环境变量：`ASPNETCORE_URLS`变量指定了由网络服务使用的URL，`ASPNETCORE_ENVIRONMENT`指定了应用程序使用的环境名称。同样，我们也应该在相应的文件夹中定义`db/db.env`文件：

[PRE5]

在这种情况下，文件定义了SQL Server容器的相应变量：`SA_PASSWORD`指定系统管理员账户密码，`ACCEPT_EULA`是SQL Server启动过程所需的。

此外，`docker-compose`支持在`.env`文件中声明默认环境变量。该文件必须放置在`docker-compose.yml`文件相同的目录中。该文件包含一些定义环境变量的简单规则。让我们在目录服务目录的根文件夹中创建一个新的`.env`文件：

[PRE6]

`COMPOSE_PROJECT_NAME`变量是Docker提供的`docker-compose`命令的保留变量。它指定了运行容器时要使用的项目名称。因此，`catalog_api`和`catalog_db`容器都将运行在同一个名为`store`的项目下。

# 定义Dockerfile

让我们通过描述我们的compose项目的Dockerfile来继续。如前所述，Dockerfile是一个简单的文本文件，其中包含用户可以调用的命令来构建镜像。让我们检查`containers/api/`文件夹中可能包含的Dockerfile的定义：

[PRE7]

此代码定义了构建Docker镜像的特定步骤。`FROM`指令指明了构建过程中要使用的基镜像。此指令是强制性的，并且必须是文件的第一条指令。`COPY`指令将项目复制到`/app`文件夹，`WORKDIR`命令将`/app`文件夹设置为默认工作目录。之后，构建脚本通过执行`dotnet restore`和`dotnet build`命令继续进行。最后，Dockerfile在`PATH`变量中添加了`/root/.dotnet/tools`路径，并执行了`containers/api/entrypoint.sh`Bash文件，其内容如下：

[PRE8]

`entrypoint.sh`入口点文件存储在与Dockerfile相同的级别。它通过执行`dotnet run`命令运行目录服务的主要项目，一旦数据库容器准备就绪，它将继续执行`dotnet ef database update`命令以创建数据库模式。

# 执行docker-compose命令

要在本地运行目录服务，我们必须使用`docker-compose`命令从CLI完成组合过程。可以通过运行`docker-compose --help`来获取命令的概述。

与多容器应用组合相关的命令如下：

+   `docker-compose build`: 这构建服务。它使用与镜像关联的Dockerfile执行构建。

+   `docker-compose images`: 这列出了当前容器镜像。

+   `docker-compose up`: 这创建并运行容器。

+   `docker-compose config`: 这验证并查看`docker-compose.yml`文件。

我们可以通过以下命令运行目录服务：

[PRE9]

通过指定`--build`标志，可以在运行容器之前触发构建。一旦构建完成，我们只需运行`docker-compose up`命令，直到我们的代码更改并需要重新构建项目。

尽管我们现在能够使用容器运行我们的服务，但构建和运行过程尚未优化：我们将解决方案中的所有文件都复制到容器中；除此之外，我们还在整个.NET Core SDK上运行容器，如果我们只想运行项目，这是不必要的。在下一节中，我们将看到如何优化容器化过程以使其更轻量。

# 优化Docker镜像

微软提供了不同的Docker镜像来运行ASP.NET Core以及使用Docker的.NET Core应用程序。重要的是要理解，执行ASP.NET Core服务的容器不需要提供SDK。

本节概述了Docker Hub上可用的不同Docker镜像以及如何使用合适的Docker镜像优化我们的部署。

# 不同Docker镜像概述

微软根据您尝试通过应用程序实现的目标提供各种镜像。让我们看看 Docker Hub 中 `microsoft/dotnet` 仓库提供的不同 Docker 镜像（[https://hub.docker.com/u/microsoft/](https://hub.docker.com/r/microsoft/dotnet/))）：

| **图像** | **描述** | **大小** |
| --- | --- | --- |
| `mcr.microsoft.com/dotnet/core/sdk:3.1` | 此镜像包含整个 .NET Core SDK。它提供了所有开发、运行和构建应用程序的工具。可以使用开发命令，如 `dotnet run`、`dotnet ef` 以及 SDK 提供的整套命令。 | 690 MB |
| `mcr.microsoft.com/dotnet/core/runtime:3.1` | 此镜像包含 .NET Core 运行时。它提供了一种运行 .NET Core 应用程序（如控制台应用程序）的方式。由于镜像仅包含运行时，因此无法构建应用程序。该镜像仅暴露运行时 CLI 命令 `dotnet`。 | 190 MB |
| `mcr.microsoft.com/dotnet/core/aspnet:3.1` | 此镜像包含 .NET Core 运行时和 ASP.NET Core 运行时。可以执行 .NET Core 应用程序和 ASP.NET Core 应用程序。作为一个运行时镜像，无法构建应用程序。 | 205 MB |
| `mcr.microsoft.com/dotnet/core/runtime-deps:3.1` | 此镜像非常轻量。它仅包含 .NET Core 运行所需的底层依赖项（[https://github.com/dotnet/core/blob/master/Documentation/prereqs.md](https://github.com/dotnet/core/blob/master/Documentation/prereqs.md)）。它不包含 .NET Core 运行时或 ASP.NET Core 运行时。它旨在用于自托管应用程序。 | 110 MB |

Docker 镜像通常以三种模式可用：*debian:stretch-slim*、*ubuntu:bionic* 和 *alpine*，具体取决于镜像运行的操作系统。默认情况下，镜像运行在 DebianOS 上。然而，也可以使用其他操作系统，例如 *Alpine* 来节省一些存储空间。例如，基于 Alpine 的镜像将 `aspnetcore-runtime` 镜像的大小从 ~260 MB 减少到 ~160 MB。

# 目录服务上的多阶段构建

让我们将 *多阶段* 构建的概念应用到之前定义的 Docker 镜像上。多阶段构建是 Docker 17.05 或更高版本的新功能。多阶段构建对于在保持 Dockerfile 易读和易于维护的同时优化 Dockerfile 的人来说非常有用。

让我们通过查看目录服务 Dockerfile 来探索如何应用多阶段构建过程：

[PRE10]

可以以下方式更改之前定义的文件：

[PRE11]

如您所见，Dockerfile 现在执行了三个不同的步骤（前两个步骤一起描述，因为它们使用相同的 Docker 镜像）：

+   `builder` 步骤使用 `mcr.microsoft.com/dotnet/core/sdk` 镜像来复制文件，并使用 `dotnet build` 命令触发项目的构建。

+   `publish` 步骤使用相同的镜像来触发 `dotnet publish` 命令，用于 `Catalog.API` 项目。

+   如其名所示，`final` 步骤使用 `mcr.microsoft.com/dotnet/core/aspnet:3.1` 镜像在运行时环境中执行已发布的包。最后，它使用 `ENTRYPOINT` 命令运行服务。

需要注意的是，这个更改优化了由 Dockerfile 生成的最终镜像。在此更改之后，我们不再需要 `entrypoint.sh` 文件，因为 Dockerfile 直接使用 `dotnet Catalog.API.dll` 命令触发服务的执行。

使用多阶段构建方法时，我们还应该注意到，我们不能触发数据库迁移的执行，因为运行时 Docker 镜像不使用 **Entity Framework Core** （**EF Core**） 工具。因此，我们需要找到另一种触发迁移的方法。一个可能的选择是从我们服务的 `Startup` 文件中释放迁移。此外，EF Core 提供了一种在 `Configure` 方法中使用以下语法在运行时应用迁移的方法：

[PRE12]

之前的代码确保通过执行 `app.ApplicationServices.GetService<CatalogContext>().Database.Migrate()` 指令来执行数据库迁移。由于 `msssql` 容器的启动时间，我们需要通过处理 `SqlException` 并采用指数时间方法重试来实现重试策略。前面的实现使用 `Polly` 来定义和执行重试策略。此外，我们还需要通过在 `Catalog.API` 项目中执行以下命令来添加 `Polly` 依赖项：

[PRE13]

重试策略在分布式系统世界中非常有用，可以成功处理失败。`Polly` 和在 Web 服务中实现弹性策略将在下一章中作为多个服务之间通过 HTTP 通信的一部分进行讨论。

在实际应用中，在服务的部署阶段执行迁移是非常不寻常的。数据库模式很少改变，将与服务相关的实现与数据库模式中的更改分开是至关重要的。出于演示目的，我们通过覆盖数据库更改以采取最直接的方法，在每次部署中都执行数据库的迁移。

# 摘要

在本章中，我们快速概述了 Docker 的功能以及如何使用它来提高隔离性、可维护性和服务可靠性。我们探讨了 Microsoft 提供的不同 Docker 镜像以及如何结合多步骤构建方法使用它们。

本章涵盖的主题提供了一个简单的方法来在本地、隔离的环境中运行我们的服务，并将相同的环境配置传播到预发布和生产环境中。

在下一章中，我们将提升如何在多个服务之间共享信息的知识。我们还将探讨一些与多服务系统相关的模式。本章中我们探讨的与Docker相关的概念也将在下一章中应用，以提供顺畅的部署体验。
