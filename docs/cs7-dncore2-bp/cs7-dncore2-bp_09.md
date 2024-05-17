# 第九章：使用 Docker 和 ASP.NET Core

在本章中，我们将看看 Docker 是如何工作的。你可能以前听说过 Docker，但还没有机会去尝试。特别是，我们将看以下内容：

+   Docker 是什么

+   图像和容器

+   Docker 如何使 web 开发人员受益

+   在 Windows 10 Pro 上安装 Docker

+   运行 Docker 并选择一些共享驱动器

+   在 Windows 防火墙似乎是问题时排除共享驱动器的故障

+   Visual Studio 2017 如何与 Docker 集成

+   创建一个 ASP.NET Core MVC 应用程序并在容器内运行它

+   使用 Docker Hub 与 GitHub 并设置自动构建

Docker 将为你打开一个全新的世界。

# Docker 是什么？

在我们开始使用 Docker 之前，让我们来看看 Docker 到底是什么。如果你访问 [`www.docker.com`](https://www.docker.com) 并查看 What is Docker? 页面，你会看到他们说 Docker 是一个容器化平台。从第一眼看来，这并没有太多意义。不过，深入挖掘一下，你会发现 Docker 简化了应用构建过程，并允许你在不同的环境中运行和部署这些应用。这些不同的环境可能是开发、测试、用户验收测试和生产环境。

Docker 使用图像和容器，如果你看一下 Docker 的标志，你会看到他们的标志中代表着容器的概念：

![](img/489db9e4-fde8-4041-8de7-8d9e548cb856.png)

货物规划员经常需要非常小心地堆放货船上的集装箱。他们在规划集装箱在船上的位置时需要考虑集装箱的目的地。

例如，前往中东的集装箱不能被装载在前往日本东京的集装箱下面。这意味着他们必须先移除顶部的集装箱，然后卸载底部的集装箱，然后再重新装载顶部的集装箱。集装箱的位置必须非常小心地规划，以优化货运物流的效率。

Docker 在使用容器方面类似。因此，让我们进一步澄清 **容器** 和 **图像** 这两个术语。

# 图像和容器

Docker 镜像只是用来创建 Docker 容器的文件。把它想象成 Docker 需要创建运行容器的蓝图。图像是只读模板，可以理解为创建容器实例所使用的共享文件的分层文件系统。

另一方面，容器是从这些图像创建的实例。容器是隔离和安全的，可以启动、停止、移动或删除。

# Docker 运行在哪里？

如前所述，使用货船的类比，货船代表你的开发环境、测试环境或生产环境。

Docker 可以原生运行在以下系统上：

+   Linux

+   Windows Server 2016

+   Windows 10

Docker 也可以在以下云上运行：

+   Amazon EC2

+   Google Compute Engine

+   Azure

+   Rackspace

从前面的观点可以看出，Docker 是非常灵活的，使用 Docker 可以为开发人员提供巨大的好处。让我们看看 Docker 如何特别有益于 web 开发人员。

# Docker 如何使 web 开发人员受益

Docker 为 web 开发人员提供了几个好处。如果你在开发人员、测试人员、设计师等混合环境中工作，你可能希望他们使用实际的应用程序而不是原型。你可以在服务器上设置应用程序，并将其连接到 SQL 数据库，然后管理每个用户从服务器访问站点所需的权限。另一方面，Docker 允许我们创建可以轻松在各个开发人员或设计师的机器上运行的容器。

我之前提到过，Docker 容器是隔离和安全的。因此，容器消除了应用程序冲突。我相信如果你已经开发了一段时间，你一定会遇到这样的情况：应用程序部署在生产服务器上。如果你想（或需要）升级应用程序的框架（例如），你可能会因升级而遇到其他应用程序冲突。有了 Docker，隔离的容器可以在不影响环境中的其他系统的情况下进行升级。

你有多少次听到开发人员说，“但是我的系统上应用程序运行良好”，而部署的应用程序失败了？这是因为开发人员的计算机、暂存服务器或生产服务器的设置可能存在差异。有了 Docker，你只需将镜像从一个环境移动到另一个环境，并让容器运行起来。这意味着，如果你的应用程序在开发机器上的容器内运行良好，它肯定也应该在暂存或生产机器上运行良好。

由于 Docker 容器的可预测性和稳定性，你能够比以前更快地发布代码。这将提高生产率。

# 在 Windows 10 专业版上安装 Docker

对于 Windows 10 专业版和 Windows 10 企业版，Docker **Community Edition** (**CE**)是免费提供的。

你可以从[`www.docker.com/docker-windows`](https://www.docker.com/docker-windows)下载 Docker CE。

Docker CE 需要 Hyper-V，因此你需要运行 Windows 10 专业版或更高版本。要查看你的 Windows 版本，以管理员身份打开命令提示符，并在提示符处输入以下命令： 

```cs
systeminfo
```

你会看到以下信息显示：

![](img/0f2a90cc-d069-4a19-934b-be1ecf222c06.png)

要检查 Hyper-V 是否已启用，请向下滚动一小段距离：

![](img/f87eb65c-b913-4de0-b8cb-3135c7738593.png)

早期版本的 Windows 没有 Hyper-V，因此 Docker CE 无法运行。根据 Docker 文档（[`docs.docker.com/v17.09/docker-for-windows/faqs/#questions-about-stable-and-edge-channels`](https://docs.docker.com/v17.09/docker-for-windows/faqs/#questions-about-stable-and-edge-channels)），Windows 10 家庭版也不受支持。

对于较旧的 Mac 和 Windows 系统，可以安装 Docker Toolbox。它使用免费的 Oracle VM VirtualBox。有关更多信息，请查看[`docs.docker.com/toolbox/toolbox_install_windows/`](https://docs.docker.com/toolbox/toolbox_install_windows/)。

如前所述，Docker CE 可在 Windows 10 专业版和 Windows 10 企业版上下载。你可以从 Docker 商店下载安装程序：[`store.docker.com/editions/community/docker-ce-desktop-windows`](https://store.docker.com/editions/community/docker-ce-desktop-windows)。

在撰写本文时，商店上的下载页面如下所示：

![](img/76541aed-10d6-4ee5-bd5d-e5e8492c5e6d.png)

点击“获取 Docker”按钮，将 Docker 安装程序下载到你的计算机上。

安装程序将要求你注销 Windows 以完成安装。它不会自动执行此操作，但在执行安装之前，最好关闭其他正在运行的应用程序。

实际上，这个安装程序是我近年来见过的最友好的安装程序之一。而且安装起来也非常简单：

![](img/1871e2f6-5cc8-491d-9356-ea91c5ed25a3.png)

通常情况下，我总是以管理员身份运行安装程序。Docker 的安装过程非常简单。安装完成后，它会提示您注销 Windows：

![](img/2399db35-e2c3-4f03-bd5f-02d30470f177.png)

重新登录 Windows 后，你可能会看到一条消息，要求你打开 Hyper-V 以使用 Docker 容器。选择打开 Hyper-V 选项。此时，你的计算机可能会再次重启。计算机重新启动后，你将看到 Docker 正在运行的通知：

![](img/73e316ce-2eaf-4d34-b34a-b2720a80b19e.png)

您已成功安装了 Docker。我告诉过你，这真的很容易。

# 理解 Docker

要开始使用 Docker，请查找 Docker for Windows 桌面应用程序：

![](img/8d2b33ab-b33c-4773-8fd7-dd45d5c50eb3.png)

这将在您的计算机上启动 Docker。当 Docker 运行时，您将在任务栏中看到它：

![](img/7c76ec28-90c1-444b-a42c-ff22667c23a8.png)

默认情况下，安装后应启动 Docker，因此首先从任务栏检查它是否正在运行。让我们看看 Docker 提供给我们的各种设置。右键单击任务栏中的 Docker 图标，然后从上下文菜单中选择“设置”。打开屏幕后，单击“共享驱动器”选项卡：

![](img/2324abcf-d808-405e-9194-b6bdd9f82d0c.png)

重要的是您选择要提供给容器的本地驱动器。检查共享驱动器可以支持卷。卷是 Docker 容器生成的数据持久存在的机制。您可以在官方 Docker 文档中阅读有关卷的更多信息[`docs.docker.com/engine/admin/volumes/volumes/`](https://docs.docker.com/engine/admin/volumes/volumes/)。

但是，我想指出文档中的以下要点：

+   卷可以很容易地进行备份

+   卷在 Linux 和 Windows 容器上工作

+   您可以在多个容器之间共享卷

+   您可以使用卷驱动程序将卷存储在远程计算机或云中

+   您可以加密卷的内容

因为卷存在于容器之外，所以它是持久保存数据的首选选择。Docker 还需要端口`445`打开以在主机机器和容器之间共享驱动器。如果 Docker 检测到端口`445`关闭，您将看到以下屏幕：

![](img/9f5007bc-555a-4261-8605-e83c058c1486.png)

您可以单击链接阅读有关此错误的文档。

有关共享驱动器的更多信息，请参阅 Docker 文档[`docs.docker.com/docker-for-windows/#shared-drives`](https://docs.docker.com/docker-for-windows/#shared-drives)。

有一些在线推荐的方法可以解决此问题。首先是卸载并重新安装“Microsoft 网络的文件和打印机共享”。

1.  要做到这一点，请从 Windows 设置中打开“网络和共享中心”。然后单击“vEthernet（DockerNAT）”连接：

![](img/bdd2714f-ce9d-4cfe-8d65-3a88929a69b7.png)

1.  在 vEthernet（DocketNAT）状态窗口中，单击“属性”按钮：

![](img/4a31733f-7913-4bc5-b40e-6875260fedb1.png)

1.  在这里，您将看到“文件和打印机共享”用于 Microsoft 网络。您的第一步是单击“卸载”按钮。这将从列表中删除条目。接下来，您需要单击“安装”按钮：

![](img/8c66efd6-a6d0-43c5-acfb-273bef665565.png)

1.  在“选择网络功能类型”屏幕上，单击“服务功能”，然后单击“添加”按钮：

![](img/8f424b5b-8afc-40f9-87b8-b6b4df3ed34e.png)

1.  在“选择网络服务”屏幕上，选择 Microsoft 作为制造商，然后单击“Microsoft 网络的文件和打印机共享”服务：

![](img/32fc82cc-f680-4f1e-9eaf-37cd244b737d.png)

1.  单击“确定”并关闭所有屏幕后，通过右键单击任务栏中的图标并单击“退出 Docker”来停止 Docker。然后，您可以通过再次单击“Windows 应用程序”来重新启动 Docker。

此时，您应该能够从设置屏幕中选择要与 Docker 一起使用的共享驱动器。如果仍然看到防火墙检测到的消息，则很可能是您的防病毒软件正在阻止它。

在我的情况下，是 ESET Endpoint Security 阻止了通信。您可能使用不同的防病毒软件，因此请查看它最近阻止的特定应用程序列表。在我的情况下，我启动了 ESET Endpoint Security 并选择了 SETUP，然后选择了 Network：

![](img/11591cb9-ac3c-4ebd-bf4e-f41c688d1141.png)

1.  接下来，我选择了“最近阻止的应用程序或设备”列表：

![](img/1c3c1f7a-1b1a-464e-bbef-ee4c9158915c.png)

在浏览列表时，我发现 ESET 阻止了`10.0.75.2`。根据 Docker 文档，这是要通过防火墙允许的 IP 地址：

“要共享驱动器，请允许 Windows 主机机器和 Windows 防火墙或第三方防火墙软件之间的连接。您不需要在任何其他网络上打开 445 端口。默认情况下，允许从 10.0.75.2（虚拟机）到 10.0.75.1 端口 445（Windows 主机）的连接。如果防火墙规则似乎是开放的，请考虑在虚拟网络适配器上重新安装文件和打印共享服务。”

1.  单击“解除阻止”按钮会显示一个确认屏幕：

![](img/d6a1c1ad-11b1-4cb8-a79b-7468da16d302.png)

当您这样做时，您已解除了`10.0.75.2`的阻止：

![](img/ef684023-d7bd-41c4-b984-5e419f0cd385.png)

1.  最后，单击“完成”，返回到 Docker 设置，并选择要共享的驱动器。

现在，您应该能够选择要供 Docker 使用的共享驱动器。如果您仍然无法共享驱动器，请查看以下 Stack Overflow 文章，获取额外的故障排除提示：[`stackoverflow.com/questions/42203488/settings-to-windows-firewall-to-allow-docker-for-windows-to-share-drive/43904051#43904051`](https://stackoverflow.com/questions/42203488/settings-to-windows-firewall-to-allow-docker-for-windows-to-share-drive/43904051#43904051)。

接下来，我们将看看 Docker 如何集成到 Visual Studio 2017 中，以及您可以为 ASP.NET Core 应用程序启用 Docker 支持的方法。我们还将看看如何为现有的 ASP.NET Core 应用程序添加 Docker 支持（或 Dockerize）。

Docker 拥有一个庞大的开发者社区，也有大量的帮助文档可用。花些时间浏览这些文档，并研究您可能遇到的任何问题。

# 在 Visual Studio 2017 中在 Docker 内运行 ASP.NET Core 应用程序

那么，这一切对我们意味着什么呢？我们已经看过如何在 Windows 10 上设置 Docker，以及如何解决围绕这一设置的一些问题。现在让我们看看如何创建一个 ASP.NET Core 应用程序并为新应用程序添加 Docker 支持。

1.  在 Visual Studio 2017 中创建一个新的 ASP.NET Core Web 应用程序，然后单击“确定”：

![](img/56d75944-f66e-4307-938a-991eb3e24ce1.png)

1.  在下一个屏幕上，选择 Web 应用程序（模型-视图-控制器）或您喜欢的任何类型，同时确保从下拉列表中选择了 ASP.NET Core 2.0。然后勾选“启用 Docker 支持”复选框。这将启用操作系统下拉列表。在这里选择 Windows，然后单击“确定”按钮：

![](img/3522e171-82c9-4f00-a6b5-db3c4e8b52b8.png)

如果您看到以下消息，您需要切换到 Windows 容器。这是因为您可能已将 Docker 的默认容器设置为 Linux：

![](img/281112c2-1ca9-4c3e-9784-9e958e82c89c.png)

如果您在任务栏中的 Docker 图标上右键单击，您将看到您有一个选项在那里启用 Windows 容器。您可以通过单击任务栏中的 Docker 图标上的“切换到 Windows 容器”选项切换到 Windows 容器：

![](img/a31d6e29-9bcb-4a0f-ad57-b32230726971.png)

切换到 Windows 容器可能需要几分钟的时间才能完成，这取决于您的线路速度和 PC 的硬件配置。

但是，如果您没有单击此选项，当选择操作系统平台为 Windows 时，Visual Studio 将要求您切换到 Windows 容器。

我选择 Windows 容器作为目标操作系统是有充分理由的。在本章中，当使用 Docker Hub 和自动构建时，这个理由将变得更加清晰。

创建完 ASP.NET Core 应用程序后，您将在解决方案资源管理器中看到以下项目设置：

![](img/be1d936d-3dcd-422f-87d8-35ea8f1e7b05.png)

添加到 Visual Studio 的 Docker 支持不仅以 Dockerfile 的形式添加，还以 Docker 配置信息的形式添加。这些信息包含在解决方案级别的全局 `docker-compose.yml` 文件中：

![](img/7a4a2b60-616d-4d23-8438-e5c1223d37e6.png)

1.  单击“解决方案资源管理器”中的 Dockerfile，您会发现它看起来一点也不复杂。请记住，Dockerfile 是创建图像的文件。图像是一个只读模板，概述了如何创建 Docker 容器。因此，Dockerfile 包含生成图像并运行它所需的步骤。Dockerfile 中的指令在图像中创建层。这意味着如果 Dockerfile 中有任何更改，只有更改的层在重新构建图像时才会被重建。Dockerfile 如下所示：

```cs
FROM microsoft/aspnetcore:2.0-nanoserver-1709 AS base 
WORKDIR /app 
EXPOSE 80 

FROM microsoft/aspnetcore-build:2.0-nanoserver-1709 AS build 
WORKDIR /src 
COPY *.sln ./ 
COPY DockerApp/DockerApp.csproj DockerApp/ 
RUN dotnet restore 
COPY . . 
WORKDIR /src/DockerApp 
RUN dotnet build -c Release -o /app 

FROM build AS publish 
RUN dotnet publish -c Release -o /app 

FROM base AS final 
WORKDIR /app 
COPY --from=publish /app . 
ENTRYPOINT ["dotnet", "DockerApp.dll"] 
```

当您查看 Visual Studio 2017 中的菜单时，您会注意到“运行”按钮已更改为“Docker”：

![](img/2138e1e5-4f04-49e9-aaca-8536e87d1906.png)

1.  单击“Docker”按钮调试 ASP.NET Core 应用程序时，您会注意到“输出”窗口中会弹出一些内容。特别感兴趣的是最后的 IP 地址。在我的情况下，它显示 Launching http://172.24.12.112 (您的将不同)：

![](img/cb41f852-e3b2-49a5-a426-04afafc9b495.png)

启动浏览器后，您将看到 ASP.NET Core 应用程序正在以前在“输出”窗口中列出的 IP 地址上运行。您的 ASP.NET Core 应用程序现在正在 Windows Docker 容器中运行：

![](img/c725afa8-0fe4-482b-a476-24d8c642e639.png)

这非常好，而且非常容易上手。但是，要将现有的 ASP.NET Core 应用程序 Docker 化，您需要做些什么呢？事实证明，这并不像您想象的那么困难。

# 为现有的 ASP.NET Core 应用程序添加 Docker 支持

假设您有一个不支持 Docker 的 ASP.NET Core 应用程序。要为此现有应用程序添加 Docker 支持，只需从上下文菜单中添加即可：

![](img/64a64c14-60a7-4988-b714-d5e1bf29d3eb.png)

要为现有的 ASP.NET Core 应用程序添加 Docker 支持，需要执行以下操作：

1.  在“解决方案资源管理器”中右键单击项目

1.  点击“添加”菜单项

1.  单击“Docker 支持”菜单中的“Docker 支持”：

![](img/c19e437f-7ae2-46eb-b76f-e07cc035afb4.png)

1.  Visual Studio 2017 现在会询问您的目标操作系统是什么。在我们的情况下，我们将以 Windows 为目标：

![](img/3a7ba04b-3d6a-4ca8-bc8e-48a65a3e8bc6.png)

1.  单击“确定”按钮后，Visual Studio 2017 将开始为您的项目添加 Docker 支持：

![](img/7982e375-fd2a-4f95-9d17-20dd50384d8c.png)

创建具有 Docker 支持的 ASP.NET Core 应用程序非常容易，甚至可以更轻松地为现有的 ASP.NET Core 应用程序添加 Docker 支持。

最后，如果遇到任何问题，如文件访问问题，请确保您的防病毒软件已将 Dockerfile 排除在扫描范围之外。还要确保以管理员身份运行 Visual Studio。

# 使用 GitHub 的 Docker Hub

接下来的部分将说明如何设置 Docker Hub 以从 GitHub 存储库中的项目进行自动构建。

在此示例中，我不会介绍如何将代码检入 GitHub。

1.  使用前几节创建的 DockerApp 项目，将其检入新的 GitHub 存储库。检入代码后，转到 Docker Hub [`hub.docker.com/`](https://hub.docker.com/) 并登录，如果还没有帐户，可以创建一个：

![](img/6c06ea26-3203-4497-a361-7af6ea985b2a.png)

1.  注册过程非常快速简单。您只需确认您的电子邮件地址，然后就可以使用了。确认电子邮件地址后，您将被提示再次登录。这将带您到 Docker Hub 仪表板。

在此页面上，您有几个选项可用。您可以创建存储库和组织，并浏览存储库：

![](img/81522c86-c32f-4321-951b-c8c5c58bedc4.png)

1.  要开始使用 GitHub，我们首先需要将 Docker Hub 与 GitHub 链接起来。点击页面右上角选择的用户名，然后点击“设置”菜单选项：

![](img/0aaf31c0-c55e-4573-a415-b34df6b71483.png)

1.  在设置下，找到“链接的帐户和服务”选项卡，然后点击它。现在您需要点击“链接 Github”选项继续：

![](img/cc496846-3255-4330-a6a8-289300f640f7.png)

1.  为了简单起见（也是建议的），我直接点击了“公共和私有访问”设置：

![](img/c3af8ca7-b625-48d4-a99b-fd83c3c07090.png)

1.  Docker Hub 现在会重定向您到授权页面，以允许 Docker Hub 访问您的 GitHub 存储库。在这里，您需要使用您的 GitHub 凭据登录：

![](img/a9013b54-e767-4678-bf54-19f892e6a7af.png)

请注意，如果您启用了双因素身份验证，您需要输入智能手机应用生成的身份验证代码。所以请保持手机附近。

1.  要授权 Docker Hub，点击“授权 docker”按钮：

![](img/7506ade2-0cfe-4af6-98dc-c020f17297dd.png)

现在您将被带回“链接的帐户和服务”页面，在那里您将看到您 Docker Hub 配置文件上的已链接帐户：

![](img/d6b0ef2f-fdf3-4304-ac9e-76de9356398d.png)

1.  接下来，我们需要去创建一个自动构建。从菜单中，点击“创建”菜单项，然后从下面的选项中选择“创建自动构建”：

![](img/14550eb3-11e9-430c-88f2-a320a4df177c.png)

1.  然后，您需要点击“创建自动构建 Github”选项：

![](img/56835390-96cb-4712-a22f-10f916fa8208.png)

这将显示您 GitHub 帐户中所有可用存储库的列表。我之前将 DockerApp 项目检入到了我的 GitHub 帐户，所以这就是我们要选择的：

![](img/706d8496-54a0-4c3c-9336-9104c9038efa.png)

1.  现在您可以根据需要在这里定义其他信息，或者保持默认设置。由您决定。完成后，点击“创建”按钮：

![](img/b9630b8e-c499-4e80-bb68-8d8ba859da69.png)

我们的自动构建现在已经创建并准备就绪。那么，这个自动构建是如何工作的呢？每当您将代码提交到 GitHub 存储库时，Docker Hub 都会构建您的项目：

![](img/de472987-7d85-4f15-80bb-5ff4abbb771a.png)

1.  要测试这一点，打开 Visual Studio 2017 中的 ASP.NET Core 应用程序并进行一些更改。然后将这些更改提交到您的 GitHub 存储库。然后点击 Docker Hub 中的“构建详情”链接。您会看到构建已排队，并将在几分钟内完成。要查看构建结果，稍等一会后刷新此页面：

![](img/a1f57fb4-fba3-4485-9233-fbba04336f93.png)

1.  刷新页面后，您会看到发生了错误。Docker Hub 将为您显示构建结果，您可以点击构建结果查看失败的详细信息。

我将举例说明一些常见的自动构建错误。我还将展示我发现的解决方法。我不确定这些方法在此期间是否有变化，但在撰写本文时，这些问题确实存在。

当我们查看失败的原因时，我们发现 Docker Hub 找不到项目根目录中的 Dockerfile。为什么会出现这个问题，我不知道。我本来期望 Docker Hub 会递归地遍历项目的树形结构，以找到 Dockerfile 的位置。不过，这很容易解决：

![](img/f514bdf0-50c6-431e-bb8e-add6e2314a9d.png)

我只是复制了我的 Dockerfile 并将其复制到解决方案的根目录。然后我再次将我的代码检入 GitHub：

![](img/5bbe7665-4b8f-4a9c-89ca-d875ceb1d2c2.png)

1.  如果刷新您的自动构建页面，您会看到它正在重新构建项目：

![](img/b81716cc-8f41-4b77-a42c-6e7284824fbb.png)

1.  这一次，又出现了另一个错误。再次点击错误条目会带您到错误详细信息：

![](img/672b1c39-3f47-4b4b-b827-70b96cb865b8.png)

这次，它显示错误的原因是我们的项目针对的是 Windows OS 而不是 Linux：

![](img/37330ff9-8d5d-4de0-99e9-5e814590b629.png)

错误列如下：

```cs
Build failed: image operating system "windows" cannot be used on this platform 
```

1.  要解决此问题，我们需要修改 Dockerfile。 Windows 的 Dockerfile 如下所示：

```cs
FROM microsoft/aspnetcore:2.0-nanoserver-1709 AS base 
WORKDIR /app 
EXPOSE 80 

FROM microsoft/aspnetcore-build:2.0-nanoserver-1709 AS build 
WORKDIR /src 
COPY *.sln ./ 
COPY DockerApp/DockerApp.csproj DockerApp/ 
RUN dotnet restore 
COPY . . 
WORKDIR /src/DockerApp 
RUN dotnet build -c Release -o /app 

FROM build AS publish 
RUN dotnet publish -c Release -o /app 

FROM base AS final 
WORKDIR /app 
COPY --from=publish /app . 
ENTRYPOINT ["dotnet", "DockerApp.dll"] 
```

1.  将其修改为使用`aspnetcore:2.0`而不是`aspnetcore:2.0-nanoserver`：

```cs
FROM microsoft/aspnetcore:2.0 AS base 
WORKDIR /app 
EXPOSE 80 

FROM microsoft/aspnetcore-build:2.0 AS build 
WORKDIR /src 
COPY *.sln ./ 
COPY DockerApp/DockerApp.csproj DockerApp/ 
RUN dotnet restore 
COPY . . 
WORKDIR /src/DockerApp 
RUN dotnet build -c Release -o /app 

FROM build AS publish 
RUN dotnet publish -c Release -o /app 

FROM base AS final 
WORKDIR /app 
COPY --from=publish /app . 
ENTRYPOINT ["dotnet", "DockerApp.dll"]
```

1.  再次将代码提交到 GitHub 以启动自动构建：

![](img/a80b5125-a920-4c7f-abdf-11a1d2747594.png)

这一次，您将看到构建成功。

有关.NET 容器要针对哪个操作系统的更多信息，请参阅以下 Microsoft 文档：[`docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/net-core-net-framework-containers/net-container-os-targets`](https://docs.microsoft.com/en-us/dotnet/standard/microservices-architecture/net-core-net-framework-containers/net-container-os-targets)。

1.  我们现在成功自动构建了 GitHub 项目。切换回 Repo Info 选项卡并记下 Docker pull 命令：

![](img/0e7cdf2a-8051-4316-8679-32126084070f.png)

您的图像的 Docker 存储库位于`dirkstrauss/dockerapp`，Docker pull 命令为`docker pull dirkstrauss/dockerapp`。

1.  以管理员身份运行 Windows 命令提示符，输入 Docker pull 命令，然后按*Enter*键：

![](img/13991978-bc1f-4220-a962-37207f72dde6.png)

您将看到您将开始将图像下载到本地计算机。

如果在拉取 Docker 镜像时收到错误消息 Error response from daemon: Get https://registry-1.docker.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)，只需通过右键单击任务栏中的 Docker 图标，单击设置，然后单击重置，然后重新启动 Docker。如果收到类似于 Image operating system "linux" cannot be used on this platform 的错误，您需要切换回 Linux 容器。有关更多信息，请参阅以下 URL：[`github.com/docker/kitematic/issues/2828`](https://github.com/docker/kitematic/issues/2828)。

1.  现在，我们需要通过输入`docker run -d -p 5000:80` `[image-repository]`来运行容器，这将将容器绑定到端口`5000`：

![](img/c2f1b0ab-af0e-466c-97bc-c61679e07b8a.png)

1.  如果要查看容器是否已启动，请运行以下命令：

```cs
Docker container ls 
```

现在您可以看到容器 ID，以及有关正在运行的容器的其他信息：

![](img/396f9412-d90d-4a78-ab7f-bb9f806582eb.png)

1.  我们现在要做的是在浏览器中运行我们在 GitHub 中检查的 ASP.NET Core 应用程序。为此，我们需要找到 IP 地址。在 Windows 10 上，我们需要查找 DockerNAT 的 IP 地址，为此我们需要运行以下命令：

```cs
ipconfig 
```

您将看到定义的 IP 地址是`10.0.75.1`，这是我们的容器将运行的 IP 地址：

![](img/7063e9b5-7dbd-4cfd-bbc0-62a279085475.png)

1.  打开浏览器，输入 IP 地址和端口号`10.0.75.1:5000`，然后点击*Enter*。您的 ASP.NET Core 应用程序将在浏览器窗口中以其全部荣耀出现：

![](img/5c595518-18bc-49e0-a905-832389e4a2ca.png)

使用 Docker Hub 设置 GitHub 以执行自动构建可能一开始看起来有点麻烦，但对开发团队来说好处多多。它允许您始终使用项目的最新构建。

# 摘要

在本章中，我们看了如何在 Windows 10 Pro 机器上安装 Docker。我们还了解了 Docker 是什么，以及对开发人员的好处。然后，我们看了一下当防火墙似乎是阻碍问题时，如何解决在本地机器上安装 Docker 的设置。然后，我们使用 Docker 创建了一个从头开始就具有 Docker 支持的 ASP.NET Core MVC 应用程序。我们还看了如何将 Docker 支持添加到现有应用程序中。最后，我们设置了 Docker 与 GitHub 集成并执行自动构建。我们还看了如何从 Docker Hub 拉取容器并在本地机器上运行它。

Docker 容器和 Docker Hub 是开发人员可以使用的工具，可以使他们的工作更加轻松。与 GitHub 和 Docker 等流行平台合作的力量将带来增加生产力和盈利能力的好处。Docker 消除了在多台机器上部署应用程序时的所有兼容性问题。

关于 Docker 还有很多要学习的，远远超过一个章节可以说明的。继续探索 Docker 的力量。
