# 第 12 章。ASP.NET Core Identity

安全性对所有类型的应用程序都是必不可少的，包括 Web 应用程序。如果任何人都可以通过冒充您来更新您的状态，您会使用 Facebook 吗？如果这是可能的，那么没有人会再回到 Facebook。从这个例子中，我们可以看到安全性与其说是一个特性，不如说是一个对所有应用程序的必需品。

在本章中，我们将学习以下主题：

+   认证和授权

+   ASP.NET Identity

+   如何在 ASP.NET Core 应用程序中使用 ASP.NET Identity 和 Entity Framework 实现安全性

当我们谈论应用程序的安全性时，我们主要希望防止任何未经授权的访问，这意味着只有有权访问信息的人才能访问它——不多也不少。

在继续之前，我想澄清一些关于安全性的核心概念。

# 认证

认证是验证用户是否有权访问系统的过程。在任何应用程序中，用户首先都会进行认证。这可以通过要求用户输入他们的用户 ID 和密码来实现。

# 授权

授权是验证用户是否有权访问请求的资源的过程。他们可能对系统有合法的访问权限，但他们可能没有访问请求资源的权限，因为他们没有所需的访问权限。例如，只有管理员用户可以访问应用程序的配置页面，而普通用户不应被允许使用此页面。

ASP.NET Identity 为保护应用程序提供了几个功能。

让我们考虑以下简单场景，其中用户试图访问 **受保护页面**，这是一个只有授权人员才能访问的页面。由于用户未登录，他们将被重定向到 **登录页面**，以便我们可以对用户进行认证和授权。认证成功后，用户将被重定向到 **受保护页面**。如果由于任何原因我们无法对用户进行认证和授权，我们可以将他们重定向到 **“访问被拒绝”页面**：

![授权](img/Image00230.jpg)

ASP.NET Core Identity 是一个会员系统，它使您能够轻松地保护应用程序，并具有添加登录功能到应用程序等特性。以下是我们需要遵循的步骤，以便在我们的应用程序中使用 **ASP.NET Identity**（与 Entity Framework 结合）：

1.  将相关依赖项添加到 `project.json` 文件中。

1.  创建一个 `appsettings.json` 文件并存储数据库连接字符串。

1.  创建 `ApplicationUser` 类和 `ApplicationDbContext` 类。

1.  配置应用程序以使用 ASP.NET Identity。

1.  创建注册和登录的 ViewModels。

1.  创建必要的控制器和相关动作方法和视图。

# 将相关依赖项添加到 project.json 文件中

如果您想在应用程序中使用 ASP.NET Identity 和 Entity Framework，您需要添加以下依赖项：

[PRE0]

创建一个 `appsettings.json` 文件并存储数据库连接字符串。

在项目根级别创建一个名为 `appsettings.json` 的文件，如以下截图所示：

![将相关依赖项添加到 project.json 文件中](img/Image00231.jpg)

将以下连接字符串存储在 `appsettings.json` 中。此连接字符串将由 ASP.NET Identity 用于在相关表中存储数据：

[PRE1]

## 添加 ApplicationUser 和 ApplicationDbContext 类

创建一个 **Models** 文件夹和几个文件——**ApplicationDbContext.cs** 和 **ApplicationUser.cs**——如以下截图所示：

![添加 ApplicationUser 和 ApplicationDbContext 类](img/Image00232.jpg)

`ApplicationUser` 类从 `AspNet.Identity.EntityFramework6` 命名空间中的 `IdentityUser` 类继承，如下所示：

[PRE2]

您可以根据应用程序的需求向用户添加属性。我没有添加任何属性，因为我希望保持简单，以展示 ASP.NET Identity 的功能。

`ApplicationDbContext` 类从 `ApplicationUser` 的 `IdentityDbContext` 类继承。在构造函数方法中，我们传递 `connectionstring`，它最终传递到基类。

甚至 `OnModelCreating` 方法也被重写。如果您想更改任何表名（由 Identity 生成），可以按以下方式操作：

[PRE3]

一旦我们创建了 `Models` 文件，我们需要配置应用程序和服务。您可以在 `Startup` 类中的 `Configure` 和 `ConfigureServices` 中配置这些。

# 配置应用程序以使用 Identity

为了使用 Identity，我们只需在 `Startup` 类的 `Configure` 方法中添加以下行：

[PRE4]

以下代码显示了完整的 `Configure` 方法，包括对 `UseIdentity` 方法的调用，即 `app.UseIdentity()` :

[PRE5]

在 `ConfigureServices` 方法中，我们将进行以下更改：

+   我们将添加 `ApplicationDbContext` 类，其连接字符串来自 `appsettings.json` 文件

+   我们将添加 `UserStore` 和 `RoleStore` 以使用 Identity

+   最后，我们将要求 ASP.NET Core 在我们请求 `IEmailSender` 和 `ISMSSender` 类时返回 `AuthMessageSender`

    [PRE6]

# 创建 ViewModels

接下来，我们将创建几个 ViewModels，我们将在我们的视图模型中使用它们。

首先，我们将创建一个包含三个属性——`Email`、`Password` 和 `ConfirmPassword` 的 `RegisterViewModel` 类。我们使用适当的属性装饰这些属性，以便我们可以使用无障碍 jQuery 验证进行客户端验证。我们将所有字段都设置为必填项，如下所示：

[PRE7]

现在，我们可以创建`LoginViewModel`模型，用户可以使用它来登录到你的应用程序。还有一个额外的属性`RememberMe`，当勾选时，将允许你登录而无需再次输入密码：

[PRE8]

# 创建控制器和相关操作方法

现在我们需要创建一个`AccountController`类，我们将在这里定义认证和授权的操作方法：

[PRE9]

在前面的代码中，我们正在使用不同组件提供的服务。`UserManager`和`SignInManager`由ASP.NET Identity提供。`IEmailSender`和`ISmsSender`是我们编写的自定义类，将用于发送电子邮件和短信。我们将在本章后面更详细地了解电子邮件和短信。日志记录由Microsoft Logging扩展提供。以下是一个简单的登录`HTTPGET`方法。它只是存储访问`Login`方法的URL并返回登录页面：

[PRE10]

# 创建视图

现在，我们将创建相应的登录视图页面。在这个视图页面中，我们只是显示了以下详细信息：

[PRE11]

![创建视图](img/Image00233.jpg)

当用户首次登录应用程序时，他们可能没有任何登录凭证，因此我们的应用程序应该提供一个功能，让他们可以创建自己的登录账户。我们将创建一个简单的`Register`操作方法，该方法将仅返回一个视图，用户可以通过该视图注册自己：

[PRE12]

我们还将创建相应的视图，其中包含电子邮件、密码、密码确认和`Register`按钮的输入控件：

[PRE13]

以下是对应的注册`POST`操作方法。在这里，程序检查模型是否有效，如果有效，它将使用模型数据创建一个`ApplicationUser`对象，并调用身份验证API（`CreateAsync`方法）。如果可以创建`user`变量，用户将使用该用户ID登录并重定向到`Home`页面：

[PRE14]

登出功能相当简单。只需调用身份验证API的`SignoutAsync`方法，然后重定向到`Index`页面：

[PRE15]

回到登录功能，以下是对应的操作方法。我们调用身份验证API的`PasswordSignInAsync`方法。登录成功后，我们将重定向到登录功能访问的URL：

[PRE16]

# 电子邮件和短信服务

如果你想将电子邮件和短信服务添加到应用程序的认证功能中，你可以通过创建这里显示的接口和类来实现：

[PRE17]

# 在控制器中保护操作方法

为了解释方便，让我们假设**关于**页面是一个受保护的页面，只有经过认证的用户才能访问它。

我们只需要在`Home`控制器中的`About`操作方法上装饰一个`[Authorize]`属性：

[PRE18]

进行上述更改后，当用户尝试访问登录页面而未登录应用程序时，将重定向用户到登录页面：

![在控制器中保护操作方法](img/Image00234.jpg)

在下面的屏幕截图中，您将注意到URL中有一个额外的查询参数，`ReturnURL`。这个`ReturnURL`参数将把应用程序重定向到特定的页面（在我们的例子中，是`ReturnURL`参数传递的值——**首页** / **关于**）。

一旦登录，您将被重定向到您之前请求的页面：

![在控制器中保护操作方法](img/Image00235.jpg)

当您注册新用户时，用户的详细信息将被存储在ASP.NET Identity创建的相关表中。

通过选择**视图** | **SQL Server对象资源管理器**选项打开SQL Server对象资源管理器窗口，如下面的屏幕截图所示：

![在控制器中保护操作方法](img/Image00236.jpg)

一旦选择**SQL Server对象资源管理器**选项，您将看到一个类似于以下屏幕截图的窗口。ASP.NET Identity使用Entity Framework和我们之前在`appsettings.json`包中提供的连接字符串为我们创建数据库。

ASP.NET Identity创建多个表来维护与身份相关的信息和Entity Framework的数据库迁移历史。由于我们在基本级别使用ASP.NET Identity，除了  **dbo.AspNetUsers** 表外，没有任何与身份相关的表会被填充：

![在控制器中保护操作方法](img/Image00237.jpg)

您可以右键单击**dbo.AspNetUsers**表并选择**查看数据**来查看数据：

![在控制器中保护操作方法](img/Image00238.jpg)

由于在我们的应用程序中只注册了一个用户，因此只创建了一行。请注意，哈希密码（由ASP.NET Identity为我们标记）和空白密码将不会存储在表中。

# 读累了记得休息一会哦~

**公众号：古德猫宁李**

+   电子书搜索下载

+   书单分享

+   书友学习交流

**网站：**[沉金书屋 https://www.chenjin5.com](https://www.chenjin5.com)

+   电子书搜索下载

+   电子书打包资源分享

+   学习资源分享

# 摘要

在本章中，我们学习了身份验证和授权。我们还学习了如何通过逐步过程在ASP.NET Core应用程序中实现ASP.NET Identity。我们还讨论了ASP.NET Identity中涉及的表，并学习了如何查看ASP.NET Identity创建的数据。
