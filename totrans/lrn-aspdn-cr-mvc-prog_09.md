# 第9章。ASP.NET Core应用程序的部署

一旦我们完成了我们的ASP.NET core应用程序的开发，我们需要部署应用程序，以便用户可以访问它。

在任何应用程序中，无论它是Web、桌面还是移动应用程序，并不是所有的功能都是通过代码实现的。实际上，你不应该试图通过代码实现所有功能。

在本章中，你将学习以下主题：

+   ASP.NET Core应用程序的配置

+   在Microsoft Azure平台上注册

+   将ASP.NET Core应用程序部署到Azure云平台

如果你使用任何之前的ASP.NET MVC版本构建了Web应用程序，将有一个名为`Web.config`（一个XML文件）的文件，你可以在此配置应用程序的所有依赖项。但在ASP.NET Core中，你的解决方案中将没有`Web.config`文件：

![ASP.NET Core应用程序的部署](img/Image00131.jpg)

相反，我们有`project.json`（一个JSON文件），其中我们将配置应用程序的依赖项。在讨论`project.json`的内容之前，让我们先简单谈谈JSON。

JSON是**JavaScript对象表示法**的缩写。它是一个开放标准的数据交换格式。它将以人类可读的文本形式存在，并包含属性/值对。考虑以下JSON，让我们分析一下它代表什么：

[PRE0]

每个数据项都是一个属性值对，由冒号分隔。例如，`"DoorNo": 16`表示第一条记录中`DoorNo`变量的值为16。每个属性值对（有时称为属性）由逗号分隔。例如，考虑以下三个属性：

[PRE1]

每条记录或对象都包含在一对花括号内。例如，以下JSON数据代表一条记录或一个对象：

[PRE2]

相似的记录可以分组在一起，并可以形成一个数组（对象数组）。在以下示例中，使用方括号表示JSON格式中的数组：

[PRE3]

如果我们需要以XML格式表示相同的数据，可以这样做。请注意，对于每条信息，我们都应该有一个开始标签和一个结束标签（以"`/`"结尾）：

[PRE4]

# `project.json`文件

所有项目配置都应该放入ASP.NET Core应用程序的`project.json`文件中。以下是在使用预定义的ASP.NET Core Web应用程序模板时创建的`project.json`文件：

![`project.json`文件](img/Image00132.jpg)

在这个JSON文件中，针对不同的功能有不同的预定义节点。让我们讨论一下`project.json`文件中的一些重要节点。

## `dependencies`节点

`dependencies`节点列出了你的ASP.NET Core应用程序的所有依赖项。

以下是在ASP.NET Core应用程序中的`dependencies`节点的片段。每个依赖项都是一个属性值对，其中属性代表依赖项，值代表依赖项的版本。如果您需要为依赖项提供更多信息，您可以使用嵌套JSON配置，就像在`Microsoft.NETCore.App`中那样：

[PRE5]

## 框架节点

在此节点中，我们提到了我们依赖的ASP.NET Core应用程序的框架。`dotnet5.6`代表完整的.NET框架，而`dnxcore50`代表包含完整.NET框架子集功能的.NET Core框架：

[PRE6]

# 微软Azure

微软Azure是微软提供的云计算平台和基础设施，用于构建、部署和管理应用程序和服务。它支持不同的编程语言和服务数组。

您可以将应用程序部署到任何具有您网络中**互联网信息服务**（**IIS**）的服务器上。但这将限制您的应用程序只能从您的网络内部访问，假设您的服务器只能从您的网络内部访问（如大多数网络设置）。在本节中，我们将部署ASP.NET Core应用程序到微软Azure，以便您的全球用户可以访问您的应用程序。

## 注册微软Azure

为了将您的应用程序部署到Azure，您需要有一个Azure账户。您可以免费创建一个Azure账户，并在前30天内有足够的信用额度免费部署您的应用程序（[https://azure.microsoft.com/en-in/](https://azure.microsoft.com/en-in/)）：

![注册微软Azure](img/Image00133.jpg)

点击**免费试用**按钮或右上角的**免费账户**链接，您将被重定向到以下页面：

![注册微软Azure](img/Image00134.jpg)

点击**立即开始**按钮，您将被重定向到以下页面。输入您的微软账户凭据并点击**登录**按钮。如果您没有微软账户，您可以通过点击页面底部的**立即注册**链接来创建一个：

![注册微软Azure](img/Image00135.jpg)

由于我已经有一个微软账户，我已经用我的凭据登录。一旦您登录，您将需要提供有关您的国家、名字、姓氏和您的办公电话的详细信息，如下所示：

![注册微软Azure](img/Image00136.jpg)

一旦您输入了所有必要的详细信息，您将需要提供您的国家代码和电话号码，以便Azure可以通过短信或电话验证您是一个真实的人，而不是机器人 ![注册微软Azure](img/Image00071.jpg) 。如果您选择**短信给我**选项，您将收到一个手机短信中的验证码；您需要在最后一个字段中输入它并点击**验证码**：

![注册微软Azure](img/Image00137.jpg)

一旦通过电话验证，您需要在以下表单中输入您的信用卡信息。您将被收取大约 1 美元的费用，并在五到六个工作日内退还到您的账户。收集此信息是为了识别用户的身份，除非用户明确选择了付费服务，否则不会向用户收费：

![注册 Microsoft Azure](img/Image00138.jpg)

一旦输入您的信用卡信息并点击**下一步**，您将需要在注册过程的最后一步同意订阅协议：

![注册 Microsoft Azure](img/Image00139.jpg)

一旦点击**注册**按钮，完成过程将需要另外五分钟。您将看到以下屏幕，直到过程完成：

![注册 Microsoft Azure](img/Image00140.jpg)

一旦完成注册过程，您将看到以下屏幕。您还将收到一封确认电子邮件（发送到您在第一步中提供的电子邮件地址），其中包含订阅详情：

![注册 Microsoft Azure](img/Image00141.jpg)

## Azure 部署先决条件

为了从 Visual Studio 2015 社区版将 ASP.NET Core 应用程序发布到 Azure，您应该安装（至少）Visual Studio 2015 更新 2，并且应该安装/启用 SQL Server 数据工具。

### 注意

如果您有 VS 2015 的最新版本，则无需安装更新 2。

您可以从以下网址下载 Visual Studio 2015 更新 2：[https://www.visualstudio.com/en-us/news/vs2015-update2-vs.aspx](https://www.visualstudio.com/en-us/news/vs2015-update2-vs.aspx) 并安装它。

要安装 SQL Server 数据工具，请转到**控制面板** | **程序和功能**。右键单击**Microsoft Visual Studio Community 2015**并选择**更改**选项，如图下截图所示：

![Azure 部署先决条件](img/Image00142.jpg)

一旦点击**更改**选项，您将看到以下窗口——在这里您需要选择**修改**按钮。一旦点击**修改**按钮，您将获得一个选项，可以修改 Visual Studio 安装选项。我已选择**Microsoft SQL Server 数据工具**，如图下截图所示：

![Azure 部署先决条件](img/Image00143.jpg)

一旦点击**下一步**，Visual Studio 将安装 SQL Server 数据工具，一旦完成，您将看到以下屏幕，显示设置完成状态：

![Azure 部署先决条件](img/Image00144.jpg)

# 在 Azure 中部署 ASP.NET Core 应用程序

让我们创建一个可以部署到 Microsoft Azure 的 ASP.NET Core 应用程序：

![在 Azure 中部署 ASP.NET Core 应用程序](img/Image00145.jpg)

一旦点击**确定**按钮，ASP.NET Core 应用程序将被创建：

![在 Azure 中部署 ASP.NET Core 应用程序](img/Image00146.jpg)

由于默认的ASP.NET Core Web应用程序模板使用Entity Framework，我们需要执行以下命令来创建数据库迁移：

[PRE7]

一旦您在**命令提示符**（在项目路径中）中输入命令，就会创建迁移文件。此迁移文件将包含对数据库的所有更改。此迁移将在Azure部署时应用，以便Azure可以创建必要的Azure部署数据库脚本：

![在Azure中部署ASP.NET Core应用程序](img/Image00147.jpg)

数据库迁移完成后，右键单击创建的Core应用程序，并选择**发布**选项，如下截图所示：

![在Azure中部署ASP.NET Core应用程序](img/Image00148.jpg)

当您点击**发布**选项时，您将看到以下屏幕，展示了您可用的各种发布选项：

![在Azure中部署ASP.NET Core应用程序](img/Image00149.jpg)

请选择  **Microsoft Azure App Service** 选项，以在Microsoft Azure平台上发布Web应用程序：

![在Azure中部署ASP.NET Core应用程序](img/Image00150.jpg)

点击**新建**按钮，您将看到以下屏幕：

![在Azure中部署ASP.NET Core应用程序](img/Image00151.jpg)

您可以将Web应用程序的名称更改为您想要的任何名称。我已经将Web应用程序的名称更改为**learningmvc6**。

点击**资源组**旁边的**新建**按钮，并输入资源组的名称。资源组只是一个标签，您可以在其中分组所有计算资源，以便如果您想删除所有资源，只需删除资源组即可。例如，资源组可能包括一个Web服务器和一个数据库服务器——您可以将它视为资源集合。

现在，点击**App Service Plan**旁边的**新建**按钮。您将看到以下窗口，您可以在其中选择Web应用程序容器的位置和大小。您的位置可以从美国南部中心到欧洲，从日本到加拿大。您的应用程序容器可以是免费的，也可以是具有7 GB RAM的机器。我选择了免费选项，因为我们的目标是部署ASP.NET Core应用程序到云环境，而不是部署一个将被数百万用户访问的应用程序。当然，您可以使用ASP.NET Core和Microsoft Azure实现相同的目标：

![在Azure中部署ASP.NET Core应用程序](img/Image00152.jpg)

现在，我们可以配置作为附加Azure服务提供的SQL数据库。

![在Azure中部署ASP.NET Core应用程序](img/Image00153.jpg)

点击顶部区域可用的+按钮，这将带我们到SQL数据库的配置。

![在Azure中部署ASP.NET Core应用程序](img/Image00154.jpg)

如果你已经在 Azure 环境中有一个现有的 SQL 服务器，你可以使用它。由于我没有这样的服务器，我将通过点击 SQL 服务器旁边的 **新建** 按钮来创建一个 SQL 服务器：

![在 Azure 中部署 ASP.NET Core 应用程序](img/Image00155.jpg)

请输入 SQL 服务器管理员用户名和密码，然后点击 **确定**。你将看到以下屏幕：

![在 Azure 中部署 ASP.NET Core 应用程序](img/Image00156.jpg)

一旦你点击 **确定**，你将看到以下屏幕：

![在 Azure 中部署 ASP.NET Core 应用程序](img/Image00157.jpg)

在上述屏幕上点击 **确定**，我们将看到 **创建应用服务** 屏幕：

![在 Azure 中部署 ASP.NET Core 应用程序](img/Image00158.jpg)

一旦我们配置了所有必需的 Azure 服务，就点击 **创建**：

![在 Azure 中部署 ASP.NET Core 应用程序](img/Image00159.jpg)

上述屏幕显示了部署配置选项，例如我们应用程序的 **站点名称** 和 **目标 URL**。在上述屏幕上点击 **下一步**：

![在 Azure 中部署 ASP.NET Core 应用程序](img/Image00160.jpg)

需要注意的是，你需要展开 **数据库** 选项和 **Entity Framework 迁移** 选项，并选择两个复选框。第一个复选框代表运行时应该使用的连接字符串，第二个复选框代表在发布时应应用的数据库迁移。

![在 Azure 中部署 ASP.NET Core 应用程序](img/Image00161.jpg)

上述屏幕是预览屏幕，其中你可以看到发布时将要部署的文件。这是一个可选步骤——如果你想查看文件，可以点击 **开始预览** 按钮。否则，你可以点击 **发布** 按钮在 Azure 平台上发布网络应用程序。

一旦你点击 **发布** 按钮，我们的 ASP.NET Core 应用程序将在 Azure 中部署，并且你的应用程序 URL 将在成功发布后打开。你将看到以下屏幕：

![在 Azure 中部署 ASP.NET Core 应用程序](img/Image00162.jpg)

# 在 Linux 环境中部署 ASP.NET Core 网络应用程序

在本章的这部分内容中，我们将学习如何在 Linux 平台上创建和部署 ASP.NET Core 网络应用程序。我将使用 **Amazon Web Services** ( **AWS** ) 在云端部署该应用程序。显然，你不需要 AWS 就能在 Linux 上部署 ASP.NET Core 应用程序。我只是用它来避免在我的本地机器上安装 Linux。并且，使用 AWS（或任何其他公共云服务提供商或任何托管提供商）托管的一个优点是，我可以从任何地方访问网络应用程序，因为它将是公开可用的。

我们有以下先决条件来创建和部署在 Linux 环境中：

+   Linux 机器

+   Putty 客户端（如果你正在使用远程 Linux 机器）

## 创建 Linux 机器

我们将使用AWS来创建一个Linux机器。使用AWS或其他任何云服务提供商的优势在于，我们只需在我们需要的时候使用他们的服务，并且当您完成使用后可以关闭机器。您只需为使用的时间付费。对于第一年，AWS有一个免费层，您可以免费托管机器（如果符合免费层条件），无需支付任何费用。我已经使用AWS超过几年时间，在云中尝试了许多事情，因此我不再符合免费层的条件。

然而，您可以通过使用任何虚拟化软件在Windows PC上安装Linux。Ubuntu Linux有从USB驱动器启动的选项，这样您就不需要打扰您的本地PC上的任何东西。

一旦您注册了AWS账户，您就可以进入**EC2仪表板**，在那里您可以创建**EC2实例**：

![创建Linux机器](img/Image00163.jpg)

在上一屏幕中点击**启动实例**。将启动一个向导，它将帮助您选择和配置实例。在这一步，我们选择Ubuntu Linux服务器，因为它易于使用。

![创建Linux机器](img/Image00164.jpg)

AWS中有不同类型的实例可供选择，从小型实例（0.5 GB RAM）到大机器（1952 GB RAM）。我们将选择**微型**实例，因为它符合免费层的条件：

![创建Linux机器](img/Image00165.jpg)

在上一步中，我们可以配置云中的实例。我们可以创建一个自动扩展组，当负载高时，AWS云会自动启动实例。由于我们的目标是创建和部署ASP.NET Core Web应用程序，我们将保留默认值，并点击**下一步：添加存储**以进入下一屏幕：

![创建Linux机器](img/Image00166.jpg)

**微型**实例不附带任何外部存储。因此，我们需要添加存储才能使用它。我们有三种存储选项可供选择：**通用型SSD**、**预配SSD**和**磁SSD**。在这三种中，**通用型SSD**是通常会被使用的存储。

当您的应用程序进行高输入输出操作时，吞吐量可能会下降。但在预配SSD中，您可以从存储设备中维持所需的吞吐量。磁存储只是旧式存储。我们将使用通用型8 GB **固态硬盘**（**SSD**），因为它很好地满足了我们的需求。

![创建Linux机器](img/Image00167.jpg)

如果您正在使用多个实例，您可以给它们打标签，这样您就可以通过标签名称来控制实例。由于我们只将启动一个实例，我将直接跳到下一步：

![创建Linux机器](img/Image00168.jpg)

在此步骤中，我们可以为实例配置安全组——应该打开哪些端口以供传入流量使用。在任何配置中的通用规则是只打开你需要的端口，而不是其他端口。你还需要告诉IP（或其范围），从哪里可以访问该机器。由于这是一个演示应用程序，我们将打开端口`22`，用于**安全外壳**（**SSH**）；使用PuTTY，以及`80`，用于访问核心Web应用程序。

一旦你配置了安全组，点击**审查和启动**。

![创建Linux机器](img/Image00169.jpg)

在下一个屏幕上，你可以查看所选选项：

![创建Linux机器](img/Image00170.jpg)

当你对所选选项满意时，可以点击**启动**。否则，你可以返回上一步重新配置它们为正确的值。

当你点击**启动**时，它将要求你选择一个密钥对，你将使用它登录到任何AWS服务器。如果你没有，你可以创建一个。由于我已经创建了一个，我将使用现有的一个，如下截图所示：

![创建Linux机器](img/Image00171.jpg)

选择密钥对并点击**启动实例**。AWS将为我们启动新的实例，状态将显示（如下截图所示）。实例ID也将可用（在截图中已框出）：

![创建Linux机器](img/Image00172.jpg)

点击蓝色链接将获取状态（如下截图所示）。**公共DNS**和**公共IP**是重要的值，你将使用它们来连接到该服务器。因此，我在截图中将它们框了起来：

![创建Linux机器](img/Image00173.jpg)

## 安装PuTTY客户端

在创建了一个新的Linux服务器，我们可以在这里创建ASP.NET 5 Web应用程序并托管它之后，我们需要安装PuTTY客户端，这是一个可以将命令发送到Linux服务器并接收响应的小应用程序。由于我们将在Linux服务器上安装应用程序，我们需要一种方法从你的Windows PC连接到Linux服务器。PuTTY客户端应用程序正是这样做的。

你可以通过访问 [http://www.chiark.greenend.org.uk/~sgtatham/putty/](http://www.chiark.greenend.org.uk/~sgtatham/putty/)  下载PuTTY客户端。

![安装PuTTY客户端](img/Image00174.jpg)

点击**下载**链接，并在下一个屏幕中选择截图中的链接（已框出）：

![安装PuTTY客户端](img/Image00175.jpg)

它将下载MSI文件。下载完成后，启动安装程序，你将看到以下欢迎屏幕：

![安装PuTTY客户端](img/Image00176.jpg)

点击**下一步**，你将看到以下屏幕：

![安装PuTTY客户端](img/Image00177.jpg)

选择安装文件夹——你可以保持默认设置并点击**下一步**：

![安装PuTTY客户端](img/Image00178.jpg)

选择您想要安装的产品功能。您可以保留默认选择并点击 **安装**。安装完成后，您将看到以下屏幕：

![安装 PuTTY 客户端](img/Image00179.jpg)

点击 **完成** 并启动 PuTTY 应用程序。它将打开 PuTTY 配置窗口，我们将在此输入主机名和认证详情。主机名是 `<username>@<public DNS>`。在我们的例子中，它是 **ubuntu@ec2-107-22-121-81.compute-1.amazonaws.com**。Ubuntu 是我们选择的 Ubuntu AMI 的默认用户。我们可以在之前显示的状态窗口中获取公共 DNS 值：

![安装 PuTTY 客户端](img/Image00180.jpg)

对于认证，在左侧面板中选择 **连接** | **SSH** | **认证**，并选择我们之前创建的 `PPK` 文件（私钥文件）：

![安装 PuTTY 客户端](img/Image00181.jpg)

点击 **打开**。您将收到一个警告，询问您是否信任此主机。点击 **是**，您将看到 Linux 屏幕的命令提示符。

![安装 PuTTY 客户端](img/Image00182.jpg)

接下来，在创建 ASP.NET 5 应用程序并最终托管它们之前，我们需要先安装 .NET Core。

## Linux机器上安装.NET Core

为了在 Ubuntu 上安装 .NET Core，我们首先需要设置 apt 和获取包含我们所需包的源的 feed。输入以下命令：

[PRE8]

您将看到以下屏幕：

![Linux机器上安装.NET Core的截图](img/Image00183.jpg)

然后通过以下命令更新它，该命令将下载所需的包并安装它们：

[PRE9]

对于此命令，您将看到以下屏幕：

![Linux机器上安装.NET Core的截图](img/Image00184.jpg)

使用以下命令安装 .NET Core：

[PRE10]

将会显示以下屏幕：

![Linux机器上安装.NET Core的截图](img/Image00185.jpg)

# 创建一个新的 ASP.NET 5 项目

通过以下命令创建一个新的目录，我们将在此目录中创建 ASP.NET 5 应用程序。第一个命令（`**mkdir**` - 创建目录）是在 Linux 中创建目录，第二个命令（`**cd**` - 切换目录）是进入文件夹。最后一个命令是创建 .NET Core 应用程序的命令行：

[PRE11]

将会显示以下屏幕：

![创建一个新的 ASP.NET 5 项目](img/Image00186.jpg)

这将创建一个 .NET Core 应用程序，其中包含几个文件——`Program.cs` 和 `project.json`。这是一个最小化应用程序，甚至没有 `Startup` 文件。

我们需要在 `project.json` 中添加 `Kestrel HTTP Server` 包作为依赖项。您可以通过执行命令 `**vi project.json**` 来编辑该文件。默认情况下，vi 编辑器将以只读模式打开文件。您需要按下 *Esc* + *I* 才能进入编辑模式。添加如下所示的行 **"Microsoft.AspNetCore.Server.Kestrel": "1.0.0"**，如图所示：

![创建一个新的 ASP.NET 5 项目](img/Image00187.jpg)

按下 Escape 键和 " *:"  * 并输入 `wq`  以写入并退出 `vi` 编辑器。

由于我们添加了依赖项，我们需要通过执行以下命令来恢复包：

[PRE12]

当您输入此命令时，所有包都将如以下截图所示恢复：

![创建新的 ASP.NET 5 项目](img/Image00188.jpg)

创建一个新文件，`Startup.cs`，内容如下。您可以通过输入命令 `**vi Startup.cs**` 来创建新文件。像往常一样，我们需要按下 *Esc* + *I* 使文件处于可写和可读模式。粘贴以下内容（您可以通过在此处复制后右键单击鼠标粘贴）：

[PRE13]

按下 *Esc* + *:* 并输入 `wq` 以保存文件。更新 `Program.cs` 文件，内容如下：

[PRE14]

您将看到以下屏幕：

![创建新的 ASP.NET 5 项目](img/Image00189.jpg)

我们已创建 ASP.NET Core 网络应用程序。现在我们需要安装 **Nginx**，一个反向代理服务器，它使您能够卸载诸如服务静态内容、缓存和压缩请求等工作。您可以通过输入以下命令来配置 Nginx 以监听特定端口（我们将在本章后面讨论细节）。您可以通过以下命令安装 Nginx：

[PRE15]

安装完成后，您可以输入以下命令来启动服务：

[PRE16]

当您运行命令时，您将看到以下屏幕：

![创建新的 ASP.NET 5 项目](img/Image00190.jpg)

# 配置 Nginx 服务器

通过修改文件（`/etc/nginx/sites-available/default`）以包含以下内容来配置 Nginx 服务器，以便 Nginx 将请求转发到 ASP.NET。为了修改此文件，您需要足够的权限——尝试切换到超级用户。`**Sudo su**` 是切换到超级用户的命令。请参见以下代码：

[PRE17]

代码看起来如下：

![配置 Nginx 服务器](img/Image00191.jpg)

通过执行以下命令来运行应用程序：

[PRE18]

您将看到以下屏幕：

![配置 Nginx 服务器](img/Image00192.jpg)

现在，使用公共 DNS（实例启动时 AWS 创建了公共 DNS）从浏览器访问应用程序：

![配置 Nginx 服务器](img/Image00193.jpg)

哇！我们已经在 Linux 箱子中创建了 ASP.NET Core 网络应用程序并启动了它。我们甚至通过 **Amazon Web Services**（**AWS**）使用了云服务。

# 摘要

在本章中，您学习了 `project.json` 文件中可用的不同组件，其中包含您所有 ASP.NET Core 的配置。我们讨论了如何注册 Microsoft Azure 云平台并在 Azure 平台上部署 ASP.NET Core 应用程序。我们还学习了如何在云中使用 Amazon Web Services 在 Linux 中创建和部署 ASP.NET Core 网络应用程序。
