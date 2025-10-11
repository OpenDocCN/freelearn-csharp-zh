# *第 11 章*：配置身份管理

在上一章中，我们学习了如何添加和自定义 ASP.NET Core Identity UI，以便用户可以注册、登录和管理他们的个人资料。不幸的是，ASP.NET Core Identity 默认不提供身份管理。

在本章中，我们将学习如何通过使用 IdentityManager2 创建用户和角色来管理 ASP.NET Core Identity。

我们将涵盖以下部分：

+   介绍 IdentityManager2

+   设置 IdentityManager2

本章的主题与 ASP.NET Core 架构的 MVC 层相关：

![图 11.1 – ASP.NET Core 架构](img/Figure_11.1_B17996.jpg)

图 11.1 – ASP.NET Core 架构

# 技术要求

要跟随本章的示例，你需要创建一个 ASP.NET Core MVC 应用程序。打开你的控制台、shell 或 Bash 终端，切换到你的工作目录。使用以下命令创建一个新的 MVC 应用程序：

[PRE0]

现在，通过双击项目文件在 Visual Studio 中打开项目，或者在 VS Code 中通过在已打开的控制台中输入以下命令来打开项目：

[PRE1]

本章的所有代码示例都可以在本书的 GitHub 仓库中找到：[https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/tree/main/Chapter11](https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/tree/main/Chapter11)。

重要提示

本章假设你已经完成了上一章中的步骤。作为替代方案，你可以重用上一章的项目，可能只需要调整项目名称。

# 介绍 IdentityManager2

IdentityManager 是一个最初由 Brock Allen 创建并拥有的项目，他同时也与 Dominick Baier 一起创建了 IdentityServer。Scott Brady ([https://www.scottbrady91.com/aspnet-identity](https://www.scottbrady91.com/aspnet-identity)) 及其雇主接管了该项目，将其移植到 ASP.NET Core，并作为 IdentityManager2 ([https://brockallen.com/2018/07/09/identitymanager2/](https://brockallen.com/2018/07/09/identitymanager2/)) 发布。

它通过 NuGet 提供 ([https://www.nuget.org/packages/identitymanager2](https://www.nuget.org/packages/identitymanager2))。

# 设置 IdentityManager2

第一步是加载包。使用已经打开的命令行或 VS Code 或 Visual Studio 中的终端：

[PRE2]

如果包已加载，打开 `Program.cs` 并将 IdentityManager2 添加到服务集合中：

[PRE3]

将 ASP.NET Identity 的服务注册从以下内容更改：

[PRE4]

更改为以下内容：

[PRE5]

这向服务集合中添加了一些相关的服务。

此外，还需要添加 `DefaultTokenProvider`：

[PRE6]

至此，服务配置就完成了。

然后，需要将 `IdentityServer` 添加到管道中。在身份验证和授权中间件之后添加它：

[PRE7]

现在我们需要将 IdentityManager2 与已经配置好的 ASP.NET Identity 数据库连接连接起来。

这需要安装以下包：

[PRE8]

现在，你可以将IdentityManager2连接到已经存在的`ApplicationDbContext`，它是`IdentityDbContext`，处理`IdentityUsers`和`IdentityRoles`。别忘了添加`using`到`IdentityManager2.AspNetIdentity`。在代码中，需要使用已经存在的`ApplicationUser`：

[PRE9]

运行`IdentityManager`就到这里。在命令提示符中输入`dotnet watch`以启动应用程序，或在VS Code或VS中按*F5*。如果你现在在浏览器中调用应用程序，你将看到管理数据的UI：

![图11.2 - IdentityManager2](img/Figure_11.2_B17996.jpg)

图11.2 - IdentityManager2

现在你可以创建角色和用户：

1.  创建一个**管理员**角色和一个**用户**角色。之后，为自己创建一个用户角色。

1.  用户角色创建后，转到**所有用户**并编辑新创建的用户。在这里，你可以更改用户属性并为他们分配两个角色：

![图11.3 - 编辑角色](img/Figure_11.3_B17996.jpg)

图11.3 - 编辑角色

通过使用IdentityManager，你获得了一个管理用户和角色的完整工具。它还可以与自定义用户和自定义用户属性一起工作。

# 保护IdentityManager2

我相信你已经注意到IdentityManager2可以在不登录的情况下访问。这是设计的一部分。你需要限制对其的访问。

Scott Brady描述了一种使用IdentityServer来实现这一功能的方法（[https://www.scottbrady91.com/aspnet-identity/getting-started-with-identitymanager2](https://www.scottbrady91.com/aspnet-identity/getting-started-with-identitymanager2)）。我们也会建议这样做。设置IdentityServer并不直接，且本书没有涵盖。不幸的是，无法使用默认的ASP.NET Core个人认证来保护IdentityManager2。看起来创建IdentityManager2 UI的中间件不支持个人认证，并重定向到ASP.NET Core Identity UI。

创建一个单独的ASP.NET Core应用程序来托管IdentityManager2是有意义的。这样，这种类型的行政UI将完全与公开可用的应用程序分离，你将能够使用OAuth或Azure Active Directory认证来保护应用程序。

# 摘要

在本章中，你学习了如何为你的应用程序添加用户界面来管理用户和角色。IdentityManager2是管理你的身份的最佳和最完整的解决方案。

在下一章中，你将学习如何使用内容协商，仅通过单个HTTP端点创建不同类型的输出。
