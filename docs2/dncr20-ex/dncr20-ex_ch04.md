# Let's Chat Web 应用

有句老话说：

"与智者的一次交谈胜过一个月的书本学习。"

带着这个想法，让我们通过开发一个名为 Let's Chat 的 ASP.NET Core 2.0 聊天应用来与我们的在线朋友进行对话。在接下来的三个章节中，我们将开发和部署这个聊天应用。在这个过程中，我们将深入研究 ASP.NET Core 2.0 的功能，了解它们的复杂性，并学习如何对 ASP.NET Core 2.0 应用进行单元测试。我们还将了解容器并将应用部署到云端。在三个章节的结尾，我们将快速开发和演示一个使用 Microsoft Bot Framework 的 Chatbot，这个 Chatbot 可以在几分钟内轻松创建，并可以添加到您的任何 Web 应用中，也可以与社交网络集成。在本章中，我们将涵盖以下主题：

+   让我们来看看 Let's Chat 网络应用的要求规范

+   Let's Chat 网络应用设计

+   项目设置

参考 ASP.NET Core 2.0，我们还将学习以下内容：

+   认证

+   授权

+   ASP.NET Core 管道

+   中间件

+   依赖项注册

+   读取配置值

+   记录

# Let's Chat 网络应用要求规范

在本节中，我们将讨论在 ASP.NET Core 2.0 中开发我们的 Let's Chat 网络应用的要求

作为用户，我应该能够满足以下对聊天室应用的要求：

1.  在网络应用中注册自己

1.  登录应用

1.  如果我忘记了旧密码，重置密码

1.  查看聊天室中所有当前登录用户的列表

1.  查看用户加入或离开聊天室

1.  如果存在，查看已登录用户的显示图片

1.  与所有在线用户同时聊天，就像聊天室一样

1.  通过互联网从浏览器访问应用并与任何数量的在线用户聊天

例如，我们正在查看类似[`gitter.im/dotnet/cli`](https://gitter.im/dotnet/cli)的高度精简版本，这是一个.NET Core **命令行界面**（CLI）工具的聊天室。

现在我们已经确定了要求规范，让我们看看应用的活动流程，并制定一个流程图。以下是一个粗略的应用工作流程流程图：

![](img/e8aeb145-101d-449f-8dde-6419c16c2cde.png)

让我们来看看流程：

1.  **开始**：这是流程的开始。用户浏览聊天 URL。它被构想为一个网站，因为我们希望从浏览器访问它并与任何数量的在线用户聊天（要求步骤 7 和 8）。

1.  **用户是否已登录**？：此时，应用检查用户是否已认证，即用户是否已登录。如果用户已登录，则将其重定向到聊天室页面；否则，用户将被重定向到登录页面。还有重置密码的选项（要求步骤 2 和 3）。

1.  **用户是否已注册**？当用户未登录时，他们将被重定向到登录页面，在那里他们需要提供正确的用户名和密码以获得认证并登录。但这仅发生在用户有有效凭证的情况下，即用户已经注册（需求 *步骤 1*）。

1.  **注册用户**：如果用户未注册，应用应提供注册用户的选项（需求 *步骤 1*）。

1.  **登录**：这是任何未经认证用户的登录页面。已经认证的用户将被直接导航到聊天室页面。这向其他在线用户表明用户已加入房间（需求 *步骤 2*）。

1.  **聊天室**：在成功认证后，用户将被重定向到聊天室，在那里他们可以看到在线用户列表并与他们所有人聊天（需求 *步骤 4*、*步骤 5* 和 *步骤 6*）。

1.  **与其他用户聊天**：一旦用户进入聊天室，他们可以与所有在线用户聊天，并可以在聊天中看到他们的名字和显示图片（需求 *步骤 6* 和 *步骤 7*）。

1.  **完成**：一旦用户与朋友聊天完毕，他们可以退出页面。这向其他在线用户表明用户已离开房间（需求 *步骤 5*）。

如我们从流程图中所见和推断，这正是每个现代在线应用工作流程的样子。我们还看到聊天仅在流程图的 *步骤 6* 和 *步骤 7* 中发生。所有其他步骤都是用于认证和设置。我们还知道这个流程涵盖了所有需求。

# Let's Chat 网页应用 - 高级设计

在本节中，我们将查看我们创建的流程图，并使用 ASP.NET Core 2.0 构成应用程序的细粒度模块。我们需要开发以下内容：

+   一个负责登录和用户认证的认证和授权模块

+   一个负责用户注册的注册模块

+   一个聊天室页面

+   一个跟踪所有已登录用户并通知客户端聊天室页面用户加入和离开的模块

+   一个用户信息模块，用于跟踪所有用户信息，例如显示名称、显示图片等

+   一个聊天中心模块，允许每个用户向其他在线用户广播消息

看起来很简单？让我们看看我们如何使用 ASP.NET Core 2.0 设计和实现上述每个需求：

+   **认证和授权模块**：这是核心功能，在大多数现代应用程序中都是必需的。使用已经开发、测试和安全的解决方案是有意义的，这样我们就不必浪费时间在重新发现轮子上了。我们可以通过多种方式实现这一点，例如创建和使用自定义身份提供者，或者使用 Facebook、Twitter、Google 或 Microsoft 认证提供者。在我们的应用程序中，我们将使用 Facebook 认证，因为用户在 Facebook 上更有可能有一个个人资料图片，这将在聊天时提供良好的用户体验。这是通过 OAuth（发音为*oh-auth*）实现的，代表开放授权，是互联网上基于令牌的认证和授权的开放标准。当我们在应用程序中实现它时，我们将详细讨论这个问题。这将处理我们在上一节中讨论的流程图中的*步骤 1*至*步骤 5*。

+   **聊天中心模块**：此模块将负责启用所有在线用户之间的实时聊天，并跟踪用户加入和离开房间的情况。中心将接收来自客户端的消息，然后将接收到的消息广播给所有客户端。这将是基于 SignalR 中心的实现，并负责处理用户连接、断开连接以及所有实时消息功能。我们在上一个应用程序中已经看到并编写了 SignalR 的基本知识。

现在我们已经有了高级设计，让我们深入细节来实现这些设计，并了解更多相关信息。

# 项目设置

在本节中，我们将开始开发 Let's Chat 网络应用程序，并在过程中学习所有必需的基本知识和 ASP.NET Core 2.0 功能。

创建一个新的 ASP.NET Core 2.0 MVC 应用程序，命名为 Let's Chat，就像我们在第一章中“*入门*”部分的*创建简单运行代码*部分所做的那样。

由于我们还需要 SignalR 来实现所有实时消息传递，因此我们需要在我们的应用程序中安装 SignalR 包。为此，请按照第三章中“*项目设置*”部分所述的步骤操作，该部分是关于*构建我们的第一个.NET Core 游戏 - 费茨*-*塔克*。

在安装 SignalR 之后，我们就准备好开始编码应用程序了。我们将首先从认证和授权模块开始，然后继续到聊天中心模块。

认证和授权紧密相关，但它们是不同的，并且是分布式应用安全性的基本概念。我们必须理解这些概念，以便我们可以开发一个安全的应用程序。安全性的一个最定义性的原则是**信任**。当我们把我们的 Web 应用程序发布到互联网上时，就像把我们的资源放在公共场所一样，数百甚至数千人都可以看到它。你相信你的资源会安全吗？有句俗话说，“我们信仰上帝，其余的必须提供数据”，所以，这是一个大大的**不**！就像放在公共场所的东西不安全一样，我们在互联网上提供的内容也不安全，这就是为什么需要认证和授权。让我们理解认证和授权，然后深入到模块的编码中。

认证涉及从用户那里获取某种凭据的过程，并基于这些凭据验证用户身份。授权是指指定资源对已认证资源的权限/访问权限的过程。认证总是在授权之前发生，即使应用程序允许匿名访问，因为应用程序首先会将用户识别为匿名用户，然后授予访问权限。授权通常基于最小权限原则。

在研究、调查和新闻业中，提问的“五 W 和如何”或 5W1H 方法论被认为是收集信息的一种非常有效的方法，因此让我们采用相同的方法来理解认证和授权：

| **问题** | **认证** | **授权** |
| --- | --- | --- |
| **什么/谁？** | 根据牛津词典，计算机世界中“认证”一词的含义是**验证用户或进程身份的过程或行为。它的目的是识别用户/进程，并确定它是否有效**。 | 根据牛津词典，“授权”一词的字面意义是**授权的行为**。 |
| **为什么？** | 基于凭据验证用户/进程的身份。 | 控制或限制对您的设备/服务/资源的访问。 |
| **在哪里/什么时候？** | 无论何时何地我需要限制或控制对设备/服务的访问。但安全专家们可能会有不同的答案，他们可能会说在所有地方和所有时间。 | 无论何时何地我需要限制或控制对设备/服务的访问。但安全专家们可能会有不同的答案，他们可能会说在所有地方和所有时间。 |
| **如何？** | 验证提供的凭据与存储的凭据是否一致。 | 检查用户/进程的角色并授予或拒绝对资源的访问权限。 |
| **示例** | 有人在敲门。你向外看，通过识别他/她来确认你是否认识这个人。这是身份验证。脑海中浮现的问题是“你是谁？” | 在同样的例子中，在执行身份验证（识别这个人）之后，如果你认识这个人，无论是你的朋友还是亲戚，你会让他/她进入你的房子，否则如果你不认识他们，你不会让他们进入。这是授权。脑海中浮现的问题是“你有什么权限？” |

下图说明了身份验证和授权流程：

![](img/388dd761-aaa8-4c49-ba1c-69852fbb1eff.png)

身份验证和授权是广泛的话题，如果我们深入探讨，可能会有关于它们的整本书。因此，我们将简要地看看各种不同的实现和相关术语，以了解其精髓。热心的读者可能想了解更多关于身份验证和授权的内容。我可以推荐一些优秀且可靠的资源：

+   [`docs.microsoft.com/en-us/aspnet/core/security/authorization/`](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/)

+   [`channel9.msdn.com/Search?term=Advanced%20ASP.NET%20Core%20Authorization%20with%20Barry%20Dorrans#pubDate=year&ch9Search&lang-en=en`](https://channel9.msdn.com/Search?term=Advanced%20ASP.NET%20Core%20Authorization%20with%20Barry%20Dorrans#pubDate=year&ch9Search&lang-en=en)

# 身份验证

在 ASP.NET 中，身份验证的实现方式多种多样。其中一些最著名的包括：

+   Windows 身份验证

+   表单身份验证

+   基于令牌的身份验证

以下部分将进一步讨论这些内容。

# Windows 身份验证

这使用本地 Windows 用户和组进行身份验证。它可以进一步细分为：

+   **基本身份验证**：用户名和密码以 Base64 编码的字符串形式发送，因此很容易被破解，所以这是一种非常弱的身份验证形式，不应使用。

+   **摘要身份验证**：摘要身份验证解决了基本身份验证的问题，发送的数据是 MD5 散列的。这个散列消息不容易被解读。然而，一些浏览器不支持它。

+   **集成身份验证**：Kerberos 身份验证或**NT 局域网管理器**（NTLM）身份验证。在安全性和支持方面，这是最好的选择。

Windows 身份验证在 ASP.NET Core 2.0 中仍然得到支持，但由于它基于 Windows，因此它不会在 Linux 或 Macintosh 上可用。应用程序必须使用 IIS 或 HTTP.SYS（一个仅适用于 Windows 的 ASP.NET Core 网络服务器）来托管。这适用于服务器、客户端和用户都属于同一 Windows 域的内部网络。

# 表单身份验证

这是一种基于 cookie/URL 的身份验证，其中用户名和密码以 cookie 的形式存储在客户端机器上，并在用户关闭 cookie 支持的情况下，在每次请求中以加密的形式发送到 URL。我们可以在 ASP.NET Core 2.0 中实现这种 cookie 身份验证，如下面的图所示：

![](img/d7291395-2811-4010-9d09-2f94150ef3db.png)

如我们所见，一旦我们在初始调用中进行身份验证后设置了 auth-key，我们只需在每次请求中传递 cookie 中的会话 ID/令牌。它可以与 ASP.NET Core Identity 一起使用，也可以不使用它。ASP.NET Core Identity 是一个成员系统，它允许我们在我们的 Web 应用程序中添加登录功能。用户可以使用用户名和密码创建账户并登录，或者他们可以使用外部社交登录提供者，如 Facebook、Twitter、Google、Microsoft 等。我们可以使用 SQL Server 或其他持久化存储机制来存储用户详细信息。由于这需要非常详细的理解，建议您阅读 Microsoft 官方文档中的 ASP.NET Identity，网址为[`docs.microsoft.com/en-us/aspnet/core/security/authentication/identity?tabs=visual-studio%2Caspnetcore2x`](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/identity?tabs=visual-studio%2Caspnetcore2x)。

如果你之前在 ASP.NET Core 之前使用过 ASP.NET，你会发现基本类型`IPrincipal`基本上是相同的。它过去是在`HTTPContext`上的`user`实现，代表请求的用户。现在，我们还有一个名为`user`的属性，但其类型是`ClaimsPrincipal`，它反过来又实现了`IPrincipal`。这是 ASP.NET Core 2.0 从 ASP.NET 的先前版本中做出的转变：即，它从基于角色的模型转变为基于声明的模型，声明是角色的超集。因此，问题产生了，什么是声明？

声明是一组以键值对形式存储的信息，用于存储用户信息，如姓名、地址、电子邮件地址、电话号码等。我们可以将声明用作角色的替代品，因为我们可以将角色转移到声明中。例如，我说“我是 Rishabh Verma。我住在印度。我是这本书的作者”，在这里我声称我的名字是 Rishabh Verma，我声称我住在印度，我声称我是这本书的作者。这就是我说声明是角色超集的含义，因为在先前的陈述中，我已经将我的作者角色转换成了声明。

以下代码应该可以很好地解释声明、身份和主体之间的关系：

```cs
IList&lt;Claim&gt; claimCollection = new List&lt;Claim&gt;
{
     new Claim(ClaimTypes.Name, "Rishabh Verma")
     ,new Claim(ClaimTypes.Country, "India")
     ,new Claim(ClaimTypes.Role, "Author")
}

ClaimsIdentity identity = new ClaimsIdentity(claimCollection);
ClaimsPrincipal principal = new ClaimsPrincipal(identity);
```

# 基于令牌的身份验证

基于令牌的身份验证系统的基本概念很简单。它允许用户输入他们的用户名和密码以获取令牌，然后在每个请求中使用此令牌来获取特定资源——而无需再次使用他们的用户名和密码。一旦获得令牌，用户就可以在令牌有效期间提供令牌以访问资源。这如图所示：

![图片](img/fafb9d4f-52ca-48a2-b372-25f4994a90f7.png)

它是无状态的且可扩展的。服务器无需在会话或内存中存储令牌（无状态），因此它是可扩展的。它适用于移动应用程序，安全，并且可以用于将身份验证传递到其他系统。由于**单页应用程序**（**SPAs**）、**物联网**（**IoT**）和 Web API 的兴起，基于令牌的身份验证变得重要。尽管有不同方式来实现令牌，但**JSON Web Token**（**JWT**）被更广泛地使用。JWT 是一个开放标准，并已成为事实上的标准令牌，它定义了一种紧凑且自包含的方法，用于安全地在各方之间传输信息，编码为 JSON 对象。JWT 因其紧凑性而获得了巨大的人气，这使得令牌可以轻松地通过查询字符串、头部属性和请求体内部进行传输。

JWT 由三部分组成：

+   **头部**：包含令牌的类型和散列算法。

+   **有效载荷**：包含身份声明。

+   **签名**：包含使用密钥和组合的头部和有效载荷创建的字符串。它用于识别令牌的完整性。

它的格式为`xxxxx.yyyyy.zzzzz`，一个示例令牌可能看起来像这样：

```cs
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoicmlzaGFiaCIsInN1cm5hbWUiOiJ2ZXJtYSIsInNpdGUiOiJodHRwOi8vd3d3LnJpc2hhYmh2ZXJtYS5uZXQiLCJjb2F1dGhvciI6Ik5laGEifQ.Nxhxs024YUyPMVHlsQQWvZ3QN8oXAwL0OAzR6FN62_E
```

您可以在[`jwt.io/`](https://jwt.io/)创建 JWT，也可以粘贴 JWT 并检查其数据。

让我们通过快速讨论 OAuth 和 OpenID Connect 来结束我们对身份验证的讨论。

+   **OAuth 2.0**：它是一个开放标准的授权协议。它通常用于提供一种方式，让用户可以使用第三方账户（例如 Facebook）登录到网站（比如我们的 Let's Chat 应用在[`packtletschat.azurewebsites.net`](http://packtletschat.azurewebsites.net)），而无需向 Let's Chat 应用提供其 Facebook 账户的密码。虽然我们将用它进行身份验证，但实际上它是一个授权协议。

+   **OpenID Connect (OIDC)**：它是一个基于 HTTP 的协议，使用身份提供者来验证用户是否真的是他们所说的那个人。它是一个非常简单的协议，并为密码提供了保护。它是一个简单的层，位于 OAuth 2.0 之上，并在 OAuth 协议之上增加了额外的安全性。由于其简单性，它已经得到了广泛的应用。Google、Facebook 和 Stack Exchange 是一些最知名的标识提供者。重要的是要注意，OIDC 与 OpenID 非常不同。后者是一个基于 XML 的协议，它遵循与 OIDC 相似的方法和目标，但以一种对开发者不太友好的方式。

# 授权

如前所述，授权是正交且独立于认证的。让我们快速浏览一下不同类型的授权：

+   简单授权

+   基于角色的授权

+   基于声明的授权

让我们在下一节中讨论它们。

# 简单授权

如果您之前使用过 ASP.NET MVC，您可能已经熟悉授权。`[Authorize]` 和 `[AllowAnonymous]` 属性是框架中内置的授权组件。在最简单的层面上，在 `Controller` 或操作上应用 `[Authorize]` 属性仅将 `Controller` 或操作的访问限制为已认证的用户。如果您将 `[Authorize]` 属性应用于 `Controller`，它将应用于所有操作：

```cs
[Authorize]
public class AccountController : Controller
{
    public ActionResult Login()
    {
    }

    public ActionResult Logout()
    {
    }
}
```

在前面的代码中，只有已认证的用户可以访问 `Login` 操作。这没有意义，因为我希望在未登录时登录，而不是在已经登录时登录。因此，如果您只想将其应用于少数操作，那么只需将这些属性应用于这些操作即可。防止这种情况的另一种方法是，在 `Login` 操作上使用 `[AllowAnnonymous]` 属性，这将使其对未认证的用户也开放。

将 `[AllowAnonymous]` 视为 `0`，将 `[Authorize]` 视为 `1`。因此，如果您在 `Controller` 上应用 `[Authorize]` 并在操作上应用 `[AllowAnonymous]`，则该操作将可以匿名访问（`1 x 0 = 0`），而其他操作将需要认证。相反，如果您在 `Controller` 上应用 `[AllowAnonymous]` 并在操作上应用 `[Authorize]`，则所有操作都将可以匿名访问。这是因为任何乘以 `0` 的结果都是 `0`，所以如果您在 `Controller` 上应用 `[AllowAnonymous]`，则所有其他操作级别的属性都将被绕过。

# 基于角色的授权

当创建 `ClaimsPrincipal` 对象时，就像我们之前做的那样，有一个名为 `IsInRole` 的属性。这个属性为我们提供了访问用户 `Roles` 的权限。基于角色的授权检查是声明性的。它们可以使用我们之前看到的 `[Authorize]` 属性以相同的方式使用，通过传递 `Roles` 参数：

```cs
[Authorize(Roles = "Administrator,ITAdminsitrator")]
public class UserAdministrationController : Controller
{ 
    ….
}
```

在前面的例子中，`UserAdministrationController` 的所有操作都对具有 `Adminsitrator` 或 `ITAdministrator` 角色的认证用户是可访问的。因此，我们可以在 `Roles` 参数中提供多个角色，并以逗号分隔的值形式，它们将被视为一个 *或* 条件。您还可以通过在操作级别提供属性来仅限制访问一个角色。

如果我需要在角色之间有一个 *与* 条件怎么办？以下代码片段将确保只有当认证用户同时具有 `ITAdministrator` 和 `Administrator` 角色时，操作才是可访问的：

```cs
[Authorize(Roles = "ITAdminsitrator")]
[Authorize(Roles = "Administrator")]
public class UserAdministrationController : Controller
{ 
    ….
}
```

如果我们查看 `[Authorize]` 属性的重载，有一个重载可以接受以下策略：

![图片](img/5b645a68-54b9-4014-bd8c-b3a0650ee07d.jpg)

基于策略的角色检查也受到支持，可以在配置授权时在 `Startup` 的 `ConfigureServices()` 方法中完成：

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.AddAuthorization(options =&gt;
    {
        options.AddPolicy("RequireAdminRole", policy =&gt; 
        policy.RequireRole("Administrator"));
    });
} 

[Authorize(Policy="RequireAdminRole")]
public IActionResult HighPreviligeAction()
{
    return View();
}
```

# 基于声明的授权

当您想根据用户声明进行授权时，我们可以使用基于声明的授权。一个现实世界的例子是当您在驾驶车辆时，交通警察拦住您，怀疑您未满 18 岁。然后，您拿出您的驾驶执照并声称您是完全合法的驾驶年龄。警察接受您的声明（因为它是由有效机构签发的）并让您继续驾驶。这是基于声明的授权。基于声明的授权检查也是声明性的，并且可以装饰在控制器或操作上。声明要求是基于策略的，因此，就像在前面章节中一样，我们需要在启动时注册策略，表达声明要求。在前面的例子中，代码看起来是这样的：

```cs
services.AddAuthorization(options =&gt;
    {
        options.AddPolicy("RequireClaim", policy =&gt; 
        policy.RequireClaim("&lt;&lt;Claim Needed&gt;&gt;"));
    });
```

这次讨论应该已经给了您一个相当清晰的认证和授权的视图，现在您应该能够深入探讨这些主题，并扩展和深化您对这些基本概念的知识。

我们还注意到，所有身份验证和授权都是在 `Startup` 类的 `ConfigureServices()` 方法中实现的，在那里配置了管道并添加了中间件。为了完成讨论，让我们快速了解 ASP.NET Core 管道以及它是如何处理请求的。以下图表逐步说明了 ASP.NET Core 如何服务请求：

![图片](img/73d351ed-4c3e-45bd-a18a-f41454504115.png)

这里是流程：

1.  浏览器将 HTTP 请求发送到服务器。请求被反向代理接收。

1.  请求被反向代理转发到 ASP.NET Core。

1.  ASP.NET Core 网络服务器接收请求并通过其管道路由，通过中间件。在通过中间件后，请求由 ASP.NET Core 应用程序处理，生成响应并将其返回。

1.  ASP.NET Core 网络服务器将响应发送给反向代理。

1.  HTTP 响应发送到浏览器。

反向代理是一种代理服务器，代表客户端从一台或多台服务器检索资源。它可以定义为一种负责接收请求并将它们转发到适当的 Web 服务器的软件组件。反向代理直接暴露给互联网，而底层的 Web 服务器仅暴露给代理。这种设置有几个好处，主要是对 Web 服务器的安全和性能。

让我们看看*步骤 3*的详细信息，因为它讨论了 ASP.NET Core Web 服务器及其管道，这是我们感兴趣的区域。首先，什么是中间件？

# 中间件

根据 ASP.NET Core 文档（可在[`docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?tabs=aspnetcore2x`](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?tabs=aspnetcore2x)找到），中间件是一种软件，它被组装到应用程序管道中以处理请求和响应。每个组件：

+   选择是否将请求传递给管道中的下一个组件

+   在调用管道中的下一个组件之前和之后都可以执行工作

请求委托用于构建请求管道。请求委托处理每个 HTTP 请求。请求委托使用`Run`、`Map`和`Use`扩展方法进行配置。单个请求委托可以内联指定为匿名方法（称为**内联中间件**），或者它可以定义在可重用的类中。这些可重用的类和内联匿名方法是**中间件**。请求管道中的每个中间件组件都负责调用管道中的下一个组件，或者根据需要短路链。

管道中中间件的经典图示，可在微软官方 ASP.NET Core 文档网站上找到，此处展示了带有步骤的详细信息，使其非常清晰。以下图示展示了*步骤 3*发生的情况：

![](img/d65abcc3-891e-47c6-a41d-10dd40a89e51.png)

让我们一步一步地来看：

1.  **步骤 1**：请求被**中间件 1**接收。在`Startup.cs`中的`Configure(IApplicationBuilder app, IHostingEnvironment env)`方法中，**中间件 1**的伪代码会是`UseMiddleware1();`，因为它处理请求并调用下一个中间件。回想一下，我们在第三章，*构建我们的第一个 .NET Core 游戏“井字棋”*中简要讨论了`Use`、`Map`和`Run`方法。

1.  **步骤 2**：执行**中间件 1**的代码/逻辑。

1.  **步骤 3**：执行`RequestDelegate`—`next();`，这会调用下一个中间件。

1.  **步骤 4**：执行**中间件 2**的代码/逻辑。在`Startup.cs`中的`Configure(IApplicationBuilder app, IHostingEnvironment env)`方法中，**中间件 2**的伪代码会是`UseMiddleware2();`。

1.  **步骤 5**：执行`RequestDelegate`—`next();`，这会调用下一个中间件。

1.  **步骤 6**：**中间件 3**的代码/逻辑被执行。根据这个图表，请求在这里被处理，但我们可以根据需要链入尽可能多的中间件。在`Startup.cs`的`Configure(IApplicationBuilder app, IHostingEnvironment env)`方法中，**中间件 3**的伪代码可以是`UseMiddleware3();`，它不调用`next();`，或者它可以是简单的`RunMiddleware3();`，这会短路管道。通常情况下，这会是 MVC 的`Routing`中间件，它会将请求路由到 MVC 的`Controller`，处理请求并返回响应。

1.  **步骤 7**：由于请求在最后一步被处理，响应被返回到**中间件 2**，它可以进一步处理响应并将其返回。

1.  **步骤 8 和 9**：同样，响应在**中间件 2**和**中间件 1**中被处理。

1.  **步骤 10**：所有中间件处理后的响应被返回。

如果我们查看`Startup.cs`类中的`Configure(IApplicationBuilder app, IHostingEnvironment env)`方法，我们会看到以下代码，这是默认 MVC 模板自带的内容：

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {            
        app.UseDeveloperExceptionPage(); 
        app.UseBrowserLink();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
    }

    app.UseStaticFiles();
    app.UseMvc(routes =&gt;
    {
        routes.MapRoute(name: "default",template: " 
        {controller=Home}/{action=Index}/{id?}");
    });
}
```

他们常说“一图胜千言”，为了在管道的上下文中理解这段代码，如果我们为非开发者环境（即`env.IsDevelopment() == false;`）绘制图表，大致会是这个样子：

![](img/4fec7bc6-63d4-44e4-b1b2-7ce058dee3bb.png)

我们可以看到，根据`else`情况的代码，请求首先通过`app.UseExceptionHandler("/Home/Error");`经过异常中间件。这样做是为了确保如果管道中的任何代码遇到异常，它将被异常处理器处理，因此它被放在第一位。从图表中，我们可以注意到在请求处理过程中，中间件不做任何事情，只是调用链中的下一个中间件。这可以从 ASP.NET Core 的源代码中看到，这些源代码在 GitHub 上免费提供给每个人查看和学习（[`github.com/aspnet`](https://github.com/aspnet)）。我强烈建议读者浏览代码，并查看 ASP.NET Core 团队实现的方式，以更好地理解这些实现。

接下来是静态文件中间件，它服务于放置在 `wwwroot` 文件夹中的静态文件，如 `css`、`js`、`images` 等。这些应该快速提供服务，因为将此中间件放在管道早期是有意义的。如果收到对静态文件的请求，静态文件中间件将处理请求并 *短路* 管道，这样在服务静态文件内容时就不会执行其他不必要的代码。这是通过 `app.UseStaticFiles();` 配置的。然后是 MVC 路由，它将传入的请求路由到其所需的控制器操作。我们看到在这个中间件中执行了许多项。我们将在后面的章节中详细讨论这些项的每个细节，但这里要注意的重要事情是请求是从控制器操作服务，然后通过相同的管道路由回。当服务非静态文件请求时，静态文件中间件没有作用，这从图中可以明显看出。如果有异常，它将由异常处理器处理，并将响应返回。

第一个 `app.Run` 代理终止了管道。也就是说，在此之后，即使你添加任何进一步的中间件或代码，它们也不会被执行。可以使用 `app.Use` 将多个请求代理或中间件链接起来。`next` 参数代表管道中的下一个代理。我们可以通过不调用 `next` 动作来短路管道。响应发送到客户端后，切勿调用 `next.Invoke`。响应开始后对 `HttpResponse` 的更改将抛出异常。在 `Configure` 方法中添加中间件组件的顺序定义了它们在请求上调用的顺序，以及响应的逆序。

希望到现在为止，我们已经对 ASP.NET Core 管道和中间件有了相对较好的理解。在我们编写自己的中间件并学习如何将其连接到我们的管道之前，让我们首先了解依赖注入，因为它将在我们编写的任何 ASP.NET Core 应用程序中得到广泛使用。

# 依赖注入

**依赖注入**（**DI**）是一种软件设计模式，它使我们能够开发松耦合的代码，并且是减少软件组件之间紧密耦合的绝佳方式。**SOLID** 设计原则中的 **D** 代表 **依赖倒置原则**（**DIP**），它指出“**高层模块不应该依赖于低层模块。两者都应依赖于抽象**”。也就是说，它依赖于抽象，而不是具体的实现。如果我们用代码的语言来说，类不应该使用 new 或静态辅助方法来填充其依赖项；它应该更倾向于注入它们。如果你曾经听到你的架构师或领导说过“**面向接口编程，而不是面向实现**”，那是因为他们想让你使用 DI。DI 是 DIP 的实现。**

DI 使以下方面变得更好：

+   可测试性

+   可维护性

+   可重用性

我在提供例子方面不是很好，但让我尝试用一个现实世界的场景来试试。假设有一个男孩想结婚。为了结婚，他需要一个女孩，因此他依赖于女孩来结婚。这个场景的伪代码大致如下：

```cs
public class Boy : IBoy
{
    //// Get married. First get the girl friend and then marry her.
    public void GetMarried()
    {
        IGirl girl = this.GetGirl(); //// Returns null, if boy doesn't 
        have a girl-friend.
        //// Marry with girl.
    }
}
```

我们注意到`Boy`类与`girl`类存在硬耦合，因为它在`marriage`方法中获取`girl`类的特定实例并与之结婚。这种硬耦合表示男孩已经有了女朋友，因此男孩和女孩紧密相连。嗯，大多数男孩（或不幸，取决于你怎么看）都不够幸运（或不幸运）有女朋友。此外，前面的代码可能不适用于所有场景，例如如果男孩没有女朋友或如果父母为男孩找到一个女孩。

实现上述代码的更好方式是移除通过从外部注入`girl`对象而导致的硬耦合。这样，`GetMarried`方法就变得通用，因为它移除了`girl`类的硬耦合，并且将适用于所有男孩，取决于传递的`girl`对象：

```cs
public class Boy : IBoy
{
    //// Get married. The girl object passed can be boy's girlfriend or 
         any girl selected by his parents
    public void GetMarried(IGirl girl)
    {
        //// Marry with girl.
    }
}
```

这是 DI（依赖注入）最简单的例子，其中不是创建一个依赖对象的新的实例，而是从外部注入依赖。依赖项可以以多种方式注入。在这里，我们将仅讨论其中的一些，并参考类和接口，如下所示。

假设我们有两个类：

+   `CoreClass`是实现`ICore`接口的主要类

+   `DependencyClass`是实现`IDependency`接口的类，这样`CoreClass`就依赖于`DependencyClass`：

```cs
public class CoreClass : ICore
{
    public CoreClass()
    {
    }

    //// Class Methods. Hidden for brevity
}

public class DependencyClass : IDependency
{
    public DependencyClass()
    {
    }

    //// Class methods. Hidden for brevity
}
```

以前面的类和接口为参考，我们可以以下列方式注入依赖项：

+   **构造函数注入**：在构造函数注入中，依赖对象在对象构造时注入，即在类的构造函数中。这样，依赖项在对象创建时就明确指出了。缺点是，一旦对象被创建，依赖项就不能更改，并且由于依赖项是在构造函数中注入的，因此类没有默认的无参数构造函数。前面类的构造函数注入代码如下：

```cs
public class CoreClass : ICore
{
    private readonly IDependency dependency;

    public CoreClass(IDependency dependency)
    {
        this.dependency = dependency;
    }

    //// Class Methods. Hidden for brevity
}
```

+   **属性设置器注入**：在属性注入中，依赖对象作为属性设置器被注入。这样，类的默认构造函数仍然存在，并且可以在对象创建后更改依赖项，只需设置属性即可。缺点是，由于依赖项不是在对象创建时注入的，因此可能错过要设置的依赖项，从而导致运行时错误。注入属性的代码看起来像这样：

```cs
public class CoreClass : ICore
{
    public CoreClass()
    {
    }

    public IDependency Dependency {get;set;}
    //// Class Methods. Hidden for brevity
}
```

在前面的`CoreClass`类中，有一个名为`Dependency`的属性，其类型为`IDependency`，这是类正常工作所必需的。然而，如果开发人员只是通过创建其新实例来初始化`CoreClass`，并忘记设置`Dependency`属性，我们可能会遇到运行时异常。相比之下，在构造函数注入中，由于依赖项是在对象创建时传递的，所以我们永远不会面临此类问题。

我们已经看到了如何进行依赖注入。想象一个系统中存在许多类，这些类都需要进行依赖注入。如果我们遵循上述任何一种注入方法并在需要的地方创建对象，代码将变得混乱不堪。因此，有意义的是拥有创建这些类并按需注入依赖项的类。这些类被称为容器，依赖注入容器，**控制反转**（**IoC**）容器。容器可以被视为一个工厂，负责提供请求的类型实例。如果给定类型已声明它有依赖项，并且容器已配置为提供依赖类型，它将在创建请求实例的过程中创建依赖项。这样，可以无需任何硬编码的对象构造，将复杂的依赖关系图提供给类。此外，除了创建具有其依赖项的对象外，容器通常还管理应用程序中的对象生命周期。

ASP.NET Core 附带一个非常简单的内置容器（`IServiceProvider`接口），默认支持构造函数注入。ASP.NET 的容器将其管理的类型称为服务。这些服务可以在 ASP.NET Core 应用程序的`Startup`类的`ConfigureServices`方法中注入到容器中。

我们需要了解有关内置 ASP.NET Core 容器的某些事实，以便正确地在应用程序中使用依赖注入。

应该解析的依赖项的构造函数必须具有`public`访问修饰符。

应只有一个构造函数，该构造函数与一组给定的参数相匹配。支持构造函数重载，但只能存在一个重载，其参数都可以通过依赖注入（DI）来满足。如果存在多个构造函数，可能会遇到`InvalidOperationException`。

注意`Startup`类中`ConfigureServices`方法的签名：

```cs
// This method gets called by the runtime. Use this method to add services to the container. 
public void ConfigureServices(IServiceCollection services)
{
    //// Code excluded for brevity.
}
```

此方法暴露了`IServicesCollection`，我们可以根据需要添加自己的服务/类型。这可以通过`Microsoft.Extensions.DependencyInjection`命名空间中的以下扩展方法来完成：

+   `AddTransient`：使用此扩展方法来实例化每次请求时都应该创建的类型。这应该用于轻量级、无状态的服务。

+   `AddScoped`：使用这个扩展方法来实例化每次请求都应创建一次的类型。基于请求的类型应根据请求进行实例化，因此应由该方法创建。

+   `AddSingleton`：使用这个扩展方法来创建只应实例化一次并在应用程序的生命周期中使用的类型。这种场景的一个常见例子是缓存。一个实例对于应用程序来说就足够了。

我的架构师将这些分别称为*每次、有时和一次*。瞬态每次请求时都会创建；作用域每次请求时创建，所以有时。单例在整个应用程序的生命周期中只创建一次，所以一次。这是一个很好的记忆方式！

在轻松的气氛中消化这个概念：一个国家的**总理**（**PM**）可以作为一个单例的例子。无论何时何地以及需要多少次，都只有一个总理，因此总理是一个单例。同样，可以形成瞬态和作用域的例子。这留作读者练习和悬念，以提供瞬态和作用域的真实世界例子。

这些扩展方法都有七个或更多的重载，以提供灵活性并满足不同的需求，例如传递具体实例或提供工厂方法来创建对象。读者应该仔细查看重载，并有效地使用这些方法。这样做最简单的方法是，在 Visual Studio 中，对其中一个扩展方法使用*F12*来阅读方法文档，或者使用 Visual Studio 的对象浏览器，或者浏览 GitHub 上的源代码（[`github.com/PacktPublishing/.NET-Core-2.0-By-Example`](https://github.com/PacktPublishing/.NET-Core-2.0-By-Example)）。

我将列出几种我们可以注入类型/服务的方法：

```cs
services.AddSingleton&lt;IHttpContextAccessor, HttpContextAccessor&gt;();    
//// Will inject HttpContextAccessor where IHttpContextAccessor is used  
     but only one instance.
services.AddSingleton&lt;HttpContextAccessor&gt;(); 

//// Directly inject HttpContextAccessor as a singleton
services.AddSingleton&lt;Connection&gt;((serviceProvider) =&gt; this.CreateConnection(serviceProvider)); 

//// Singleton using factory method overload using service provider.   
     Same overloads apply to transient and scoped as well.
services.AddTransient&lt;IUserRepository, UserRepository&gt;(); 

//// A new instance of UserRepository would be created and passed, 
     whenever it is requested.
services.AddScoped&lt;IScopedService, ScopedService&gt;(); 
//// Would be created and passed, once per request.
```

这标志着我们对依赖注入的讨论结束。有关 ASP.NET Core 2.0 中依赖注入的更详细和广泛的探讨，请访问[`docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection`](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)。

# 你如何编写自定义中间件？

现在是创建我们自己的简单中间件并将其插入管道的合适时机，这样我们就可以通过一个实际例子来结束我们所学到的所有理论。

我们可以编写内联中间件或整洁的类，该类可以在启动时进行配置。我们将编写两者：

```cs
 public void Configure(IApplicationBuilder app) 
 { 
    app.Run(async context =&gt; 
    { 
        await context.Response.WriteAsync(".NET Core 2.0 By Example"); 
    });
 }
```

这是最简单的中间件之一。这是`Run`中间件，在执行后短路管道，并仅发送带有消息`".NET Core 2.0 By Example"`的响应。它已经通过以下代码插入到`Startup.cs/`的`Configure`方法中。

在看到了内联中间件之后，让我们编写你会在应用程序中编写的更严肃的中间件。

让我们从创建一个名为`MyFirstMiddleware`的类开始。

由于中间件只是一个处理请求/响应并调用链中下一个代理的请求代理，以下是如何创建中间件类的示例。注意，下一个请求代理预期将由 DI 解决。这是一个相当简单的类，由以下内容组成：

+   一个 `readonly` 字段，用于保存对下一个 `RequestDelegate` 的引用。

+   一个构造函数，它期望将下一个 `RequestDelegate` 作为参数执行。在构造过程中，它设置字段以引用下一个请求代理。

+   一个 `Invoke` 方法，它接受一个 `HttpContext` 对象进行操作，然后将它传递给下一个中间件。

让我们看看代码：

```cs
public class MyFirstMiddleware
{ 
    //// To hold the next middleware in the pipeline.
    private readonly RequestDelegate next;

    public MyFirstMiddleware(RequestDelegate next) 
    { 
        this.next = next; 
    }

    public async Task Invoke(HttpContext httpContext) 
    { 
        // Execute the logic, that doesn't write to response.

        // Call the next delegate in the pipeline
        await this.next(httpContext); 
    }
}
```

通过这种方式，我们已经准备好了简单的中间件代码。然而，我们还没有将其连接到我们的应用程序管道。为此，我们将创建一个 `IApplicationBuilder` 的扩展方法，它内部使用 `UseMiddleware<T>` 方法来注册中间件，如下所示：

```cs
public static class MyFirstMiddlewareExtensions 
{ 
    public static IApplicationBuilder UseMyFirstMiddleware(this 
    IApplicationBuilder builder) 
    { 
        return builder.UseMiddleware&lt;MyFirstMiddlewareExtensions&gt;(); 
    }
}
```

最后，要将它连接到我们的管道，请转到 `Startup` 类的 `Configure` 方法，并使用简单的一行代码添加中间件，`app.UseMyFirstMiddleware();`。

通过这种方式，我们已经创建了我们的第一个中间件并将其连接到我们的应用程序管道。太棒了，简单又快捷！

接下来，我们将探讨 ASP.NET Core 中的配置提供程序以及我们如何读取配置并在我们的应用程序中使用它。

C# 中的扩展方法允许我们通过新功能扩展现有类型，并向其添加新方法，而无需派生或重新编译旧类型。扩展方法是一种特殊的静态方法，但它们被调用时就像它们是扩展类型的实例方法一样。最常用的扩展方法是 `LINQ` 标准查询运算符。ASP.NET Core 广泛使用扩展方法来注册服务。

# 配置

当我们开发 ASP.NET Core 网络应用程序时，我们会很快意识到我们需要在运行时更改一些设置。以一个向服务发出 API 调用的网络应用程序为例。当你开发网络应用程序时，服务部署在一些开发服务器上，当你将其部署到生产环境中时，服务 URL 是不同的，因此 URL 不应该硬编码在应用程序中。相反，它应该从配置中读取，这样就可以在不重新编译代码的情况下进行更改。另一个常用的例子是数据库连接字符串，如果应用程序使用它们。本节将探讨我们如何在 ASP.NET Core 中实现这一点。

ASP.NET Core 配置模型有三个主要的感兴趣的结构：

+   `ConfigurationProvider`

+   `ConfigurationRoot`

+   `ConfigurationBuilder`

如以下截图所示，映射图是由 `Microsoft.Extensions.Configuration` 构建的：

![](img/5f65ea3a-6b35-406a-8610-583dfe10861d.png)

`ConfigurationProvider`被开发者抽象化，因此我们不会看到它，但了解存在对多个配置提供者的支持是好的。其中一些重要的提供者包括：

+   **JSON 文件**：读取应用程序`Startup`文件夹中的 JSON 文件并配置应用程序

+   **命令行参数**：在启动应用程序时，可以将命令行参数作为键值传递来配置应用程序

+   **环境变量**：设置环境变量，提供者会读取环境变量并处理配置

+   **Azure 密钥保管库**：我们将在讨论 Azure 的后续章节中介绍这个内容，但正如其名称所暗示的，它是一个密钥保管库。

因此，在一个单一的应用程序中，我们可以选择从 JSON 文件中读取一些配置，从命令行读取一些，等等。当我们添加多个提供者时，考虑添加它们的顺序很重要，因为这定义了配置值添加到基础字典中的顺序。较晚提供者的配置值将覆盖较早提供者具有相同键的值。此外，我们还可以在图中注意到`ConfigurationProvider`中有一个`GetReloadToken`方法，在`ConfigurationRoot`中也有一个`Reload`方法。如果我们阅读文档（在 Visual Studio 中使用*F12*），我们知道配置系统支持在不重新启动 Web 应用程序的情况下重新加载配置，这是非常棒的。让我们看看我们如何读取配置。默认的`Startup.cs`包含以下代码：

```cs
public Startup(IConfiguration configuration)
{
    Configuration = configuration;
}

public IConfiguration Configuration { get; }
```

注意到我们已经在`Startup`构造函数中注入了`IConfiguration`接口，这设置了`Startup`的`Configuration`属性。另外，请注意，当我们从模板创建应用程序时，默认包含`appsettings.json`。本质上，默认代码已经为我们连接了基于 JSON 的配置，我们无需编写额外的代码即可利用其优点。

为了有效地使用`Configuration`属性，让我们在`IConfiguration`上执行*F12*（转到定义）来查看其属性。以下是在 Visual Studio 中出现的代码：

```cs
    //
    // Summary:
    // Represents a set of key/value application configuration 
       properties.
    [DefaultMember("Item")]
    public interface IConfiguration
    {
        //
        // Summary:
        // Gets or sets a configuration value. //
        // Parameters:
        // key:
        // The configuration key. //
        // Returns:
        // The configuration value.
        string this[string key] { get; set; }

        //
        // Summary:
        // Gets the immediate descendant configuration sub-sections. //
        // Returns:
        // The configuration sub-sections.
        IEnumerable&lt;IConfigurationSection&gt; GetChildren();

        //
        // Summary:
        // Returns a Microsoft.Extensions.Primitives.IChangeToken that 
           can be used to observe
        // when this configuration is reloaded. //
        // Returns:
        // A Microsoft.Extensions.Primitives.IChangeToken.
        IChangeToken GetReloadToken();
        //
        // Summary:
        // Gets a configuration sub-section with the specified key. //
        // Parameters:
        // key:
        // The key of the configuration section. //
        // Returns:
        // The 
           Microsoft.Extensions.Configuration.IConfigurationSection. //
        // Remarks:
        // This method will never return null. If no matching sub-
           section is found with
        // the specified key, an empty 
           Microsoft.Extensions.Configuration.IConfigurationSection
        // will be returned.
        IConfigurationSection GetSection(string key);
    }
```

因此，我们看到我们可以通过索引器属性获取键的值，通过传递要查找的键。我们可以获取子项，获取一个重新加载令牌，或者通过提供键来获取特定的部分。这份文档使我们的配置使用任务变得极其简单。让我们看看一个例子。首先，定义我们想要读取的配置，如下所示：

```cs
{
"Book": ".NET Core 2.0 By Example",
  "Genere": {
    "Name": "Technical level 200"
  },
  "Authors": [
    {
      "Name": "Rishabh Verma",
      "Experience": "10"
    },
    {
      "Name": "Neha Shrivastava",
      "Experience": "7"
    }
  ]
}
```

读取值的代码如下：

```cs
var book = Configuration["Book"];
var genereName = Configuration["Genere:Name"];
var author1Name = Configuration["Authors:0:Name"];
var author1Experience = Configuration["Authors:0: Experience "];
var author2Name = Configuration["Authors:1:Name"];
var author2Experience = Configuration["Authors:1:Experience"];
```

注意到我们使用零基索引来访问数组的元素。其余的键可以通过传递正确的键到`Configuration`索引器来读取，正如我们刚才看到的。

我们通常为不同的环境使用不同的配置设置。例如，开发、测试、预发布和生产可能都有不同的配置设置。在 ASP.NET Core 2.0 应用中，`CreateDefaultBuilder` 扩展方法添加了以下顺序的配置提供程序来读取 JSON 文件和环境变量：

1.  `appsettings.json`

1.  `appsettings.&lt;EnvironmentName&gt;.json`

1.  环境变量

在这里，`appsettings.&lt;EnvironmentName&gt;.json` 将覆盖 `appsettings.json` 中定义的键值，而环境变量将覆盖之前定义的键值。

这不需要在我们的 `Startup` 类中进行任何代码更改。然而，如果我们希望使用任何其他命名的配置文件，那么我们需要更改代码。以下示例使用 `config.json` 作为配置文件，然后是 `config.Development.json`：

```cs
public Startup(IHostingEnvironment env)
{
    var builder = new ConfigurationBuilder()
       .SetBasePath(env.ContentRootPath)
       .AddJsonFile("config.json")
       .AddJsonFile($"config.{env.EnvironmentName}.json");
       Configuration = builder.Build();
}
```

一个深思熟虑的程序员会说，在我的 `appsettings.json` 中，可能会有各种用于缓存、数据库连接字符串、服务 URL 等的章节。为什么我要将这些值传递到每个地方呢？例如，缓存与数据库连接字符串章节或服务 URL 有什么关系？看起来这里没有职责分离，明显违反了 **SOLID** 的 **I**（接口隔离原则）：**接口隔离原则**。为什么缓存需要依赖于 `IConfiguration`，而它只需要它的一个子集？为了防止这些违规行为，如果配置要在 `Startup.cs` 之外读取，则不建议使用前面提到的方式来访问设置。

我们应该使用所谓的 `Options` 模式。使用它很简单：首先创建一个简单的 **Plain Old CLR Object**（**POCO**）类，然后通过将其注册为服务并在需要的地方消费它来使用它。这是一个简单的、无脑的类，具有属性但没有逻辑或智能。

上述示例的 POCO 类将是：

```cs
public class BookDetails
{
     public string Book { get; set; }
     public Genere Genere { get; set; }
     public Author[] Authors { get; set; }
 }

 public class Genere
 {
     public string Name { get; set; }
 }

 public class Author
 {
     public string Name { get; set; }
     public string Experience { get; set; }
 }
```

要注册它，我们进入 `Startup.cs` 中的 `ConfigureServices` 方法并添加以下行：

```cs
services.Configure&lt;BookDetails&gt;(Configuration);
```

`BookDetails` 将会被填充并添加到容器中。要在控制器中使用它，以下代码就足够了：

```cs
private readonly BookDetails details;

public HomeController(IOptions&lt;BookDetails&gt; options)
{
     this.details = options.Value;
}
```

现在可以从强类型对象 `details` 中使用属性。如果你希望在配置绑定后覆盖某些属性，你可以在 `ConfigureService` 方法中执行以下操作：

```cs
services.Configure&lt;BookDetails&gt;(Configuration);
services.Configure&lt;BookDetails&gt;(opt =&gt; {opt.Name = “Roslyn via C#”;});
```

这将书的名称更改为 `"Roslyn Via C#"`。这是一个“后发制人”的方法，即最后执行的操作占上风。

另一个问题浮现在脑海中：如果我的配置值发生变化，我是否需要像过去修改 `web.config` 时那样重启我的应用程序？不，在 ASP.NET Core 中我们已经考虑了这一点。为此，我们需要使用 `IOptionsSnapshot`，它被设计用来支持配置文件更改时配置数据的重新加载。使用带有 `reloadOnChange` 标志设置为 `true` 的 `IOptionsSnapshot`，当文件更改时，选项会被绑定到配置并重新加载。

从前面的示例中，我们只需要在 `AddJsonFile` 方法中添加 JSON 文件时，将 `IOptions` 更改为 `IOptionsSnapshot` 并添加一个标志 `reloadOnChange` 设置为 `true`。

让我们退一步，看看我们的 `Program.cs` 文件，它是应用程序的入口点：

```cs
public static void Main(string[] args)
{
    BuildWebHost(args).Run();
}

public static IWebHost BuildWebHost(string[] args) =&gt;
         WebHost.CreateDefaultBuilder(args)
         .UseStartup&lt;Startup&gt;()
         .Build();
}
```

`CreateDefaultBuilder` 加载来自 `appsettings.json`、`appsettings.{Environment}.json`、用户机密（在开发环境中）、环境变量和命令行参数的可选配置。最后调用 `CommandLine` 配置提供程序。将提供程序放在最后调用允许在运行时传递的命令行参数通过“后到先得”的方法覆盖之前调用的其他配置提供程序设置的配置。此外，重要的是要注意，`reloadOnChange` 对 `appsettings` 文件是启用的，因此如果应用程序启动后 `appsettings` 文件中的匹配配置值发生变化，则命令行参数将被覆盖。

当我们在企业应用程序中工作时，我们会意识到存在各种配置设置，例如机密、密码等，这些不应该保存在配置文件中，因此应该从应用程序代码中排除。此外，作为一个最佳实践，我们不应该在我们的开发环境中使用生产机密。在生产环境中，这些也应该从 Azure Key Vault 中读取，我们将在后面的章节中探讨这一点。在开发环境中，我们可以使用 Secret Manager 工具来保护机密。

在看到如何使用用户机密之前，一个问题浮现在脑海中：这是否意味着我们不使用环境变量来将机密与应用程序隔离开，因为 ASP.NET Core 支持，并且在模板代码中默认存在？是的，我们可以使用环境变量。然而，环境变量以纯文本形式存储，可以被任何第三方代码访问，因此我们可以在本地开发中使用它们，但不应该依赖它们进行生产部署。

秘密管理器工具只是给在开发期间将 .NET Core 项目的秘密存储在代码库之外的一个花哨的名字。存储的数据*不是*加密的。这种方法提供的唯一优势是秘密不会成为代码的一部分，因此不会在源控制中检查，因此在开发期间秘密将保持秘密。秘密存储在一个 JSON 文件中，该文件保存在用户配置文件文件夹中。要设置用户秘密工具，编辑 `.csproj` 文件，并在 `ItemGroup` 节点中添加以下行：

```cs
&lt;DotNetCliToolReference Include="Microsoft.Extensions.SecretManager.Tools" Version="2.0.0" /&gt;
```

因此，最终的代码看起来像这样：

```cs
&lt;ItemGroup&gt;
    &lt;DotNetCliToolReference 
    Include="Microsoft.VisualStudio.Web.CodeGeneration.Tools" 
    Version="2.0.0" /&gt;
    &lt;DotNetCliToolReference 
    Include="Microsoft.Extensions.SecretManager.Tools" Version="2.0.0" 
    /&gt;
&lt;/ItemGroup&gt;
```

保存 `.csproj` 文件。这将恢复我们刚刚添加的包。现在，在解决方案资源管理器中右键单击项目，并选择管理用户秘密。这将向 `PropertyGroup` 中添加一个名为 `UserSecretId` 的新节点。保存文件。Visual Studio 将打开一个名为 `secrets.json` 的文件。将鼠标悬停在其上，查看路径。它将是 `%AppData%\microsoft\UserSecrets\<userSecretsId>\secrets.json`。

现在我们可以像配置文件一样在 JSON 文件中添加键值对。要读取它们，我们需要在我们的启动代码中添加几行代码，如下所示：

```cs
public Startup(IHostingEnvironment env)
{
    var builder = new ConfigurationBuilder()
       .SetBasePath(env.ContentRootPath)
       .AddJsonFile("config.json")
       .AddJsonFile($"config.{env.EnvironmentName}.json");

       Configuration = builder.Build();
}
```

之后，我们可以像访问其他任何配置项一样访问秘密：

```cs
var secretValue = Configuration["SecretKey"];
```

这就结束了我们的配置讨论。让我们看看我们如何进行日志记录。

# 记录日志

ASP.NET Core 支持日志记录 API，它可以与各种日志记录提供者一起工作。我们可以将日志写入一个或多个位置，也可以集成第三方日志记录框架，如 NLog、Serilog 等。在本节中，我们将查看开箱即用的日志记录，因为它足以满足大多数应用程序的日志记录需求。

让我们看看日志记录基础设施的架构。日志记录的代码映射图如下所示：

![](img/ea861236-f127-49cb-814c-50edc17cce06.png)

如我们所见，它由以下主要组件组成：

+   `ILogger/ ILogger<TCategoryName>`：我们将在我们的应用程序中使用这个来记录消息。我们可以看到它有一个 `IsEnabled()` 方法来检查日志记录是否启用，以及一个 `Log` 方法来写入日志。

+   `ILoggerFactory`：它有一个添加提供者和日志记录器的方法。

+   `ILoggerProvider`：它有一个创建日志记录器的方法，它将控制日志的输出位置。

我们还注意到 `EventId`、`LogLevel` 枚举以及用于注册日志的扩展。

我们可以在 `Program.cs` 文件中构建 `WebHost` 的时候配置日志记录，如下所示：

```cs
  public static IWebHost BuildWebHost(string[] args) =&gt;
            WebHost.CreateDefaultBuilder(args)
                .UseStartup&lt;Startup&gt;()
                .ConfigureLogging((hostingContext, logging) =&gt; { logging.AddConfiguration(hostingContext. 
                 Configuration.GetSection("Logging"));
 logging.AddConsole();
 logging.AddDebug();
            })
                .Build();
```

此代码配置了日志记录以读取 `appsettings.json` 中提供的日志记录配置部分，并添加了控制台和调试日志记录。或者，我们可以将 `ILoggerFactory` 作为参数注入到 `Startup.cs` 的 `Configure` 方法中，并配置 `loggerFactory`，如下所示：

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole();
    loggerFactory.AddDebug();
    //// Other methods omitted for brevity.
}
```

就像 ASP.NET Core 应用程序中的任何其他类型或服务一样，日志记录器也是通过依赖注入 (DI) 注入到类或控制器中的。在这里，我们注入 `ILogger<T>`，其中 `T` 是类的名称。类型参数 `T` 用于定义 *类别*，因为它遵循 `ILogger<TCategoryName>` 接口。

例如，要在 ASP.NET Core 控制器 `HomeController` 中编写日志消息，我们需要注入 `ILogger<HomeController>` 并在 `ILogger` 上调用其中一个日志扩展方法：

```cs
public class HomeController: Controller 
{
    private readonly ILogger&lt;HomeController&gt; logger;

    public HomeController(ILogger&lt;HomeController&gt; logger)
    {
         this.logger = logger;
    }

    public IActionResult Index()
    {
        logger.LogInformation($"Calling {nameof(this.Index)}");
        return View();
    }
}
```

这会将日志消息写入配置的日志提供程序的每个输出。以下是在控制台中的样子：

![图片](img/d61c6be4-43d1-4d95-a8be-65da5cd28beb.jpg)

ASP.NET Core 默认包含许多日志提供程序，我们可以使用它们将日志消息写入各种位置：

+   **控制台提供程序**：将日志消息写入控制台

+   **调试提供程序**：在 Visual Studio 调试时将日志消息写入调试窗口

+   **EventSource 提供程序**：使用 Windows 事件跟踪写入日志消息

+   **事件日志提供程序**：将日志消息写入 Windows 事件日志

同样支持第三方结构化日志，这非常有用，因为它使得在生产环境中查找和诊断问题变得更加容易。结构化日志涉及将键值对与每个日志条目关联起来，而不是简单的消息字符串。我们不会深入讨论这个问题，但了解这一点是好的，读者可以将它作为练习来探索和实现使用 ASP.NET Core 应用程序的结构化日志。

每个创建的日志条目都包含一个 *类别*。我们在创建 `ILogger` 对象时指定类别。类别可以是任何字符串，但惯例是使用写入日志的类的限定名，就像我们在之前的示例中所做的那样。我们还可以指定日志级别，它表示日志的严重程度或重要性。例如，通常在方法正常执行时使用信息日志，在方法返回 404 返回代码时使用警告日志，在捕获到意外异常时使用错误日志。包含级别的日志方法都是 `ILogger` 的扩展方法，例如 `LogError`、`LogWarning`、`LogInformation` 和 `LogTrace`。在幕后，这些方法调用带有 `LogLevel` 参数的 `Log` 方法。ASP.NET Core 定义了以下日志级别，按严重程度从低到高排序。代码的文档使其直观易懂：

```cs
    //
    // Summary:
    // Defines logging severity levels.
    public enum LogLevel
    {
        //
        // Summary:
        // Logs that contain the most detailed messages. These messages 
           may contain sensitive
        // application data. These messages are disabled by default and 
           should never be
        // enabled in a production environment.
        Trace = 0,
        //
        // Summary:
        // Logs that are used for interactive investigation during 
           development. These logs
        // should primarily contain information useful for debugging 
           and have no long-term
        // value.
        Debug = 1,
        //
        // Summary:
        // Logs that track the general flow of the application. These 
           logs should have long-term
        // value.
        Information = 2,
        //
        // Summary:
        // Logs that highlight an abnormal or unexpected event in the 
           application flow,
        // but do not otherwise cause the application execution to 
           stop.
        Warning = 3,
        //
        // Summary:
        // Logs that highlight when the current flow of execution is 
           stopped due to a failure.
        // These should indicate a failure in the current activity, not 
           an application-wide
        // failure.
        Error = 4,
        //
        // Summary:
        // Logs that describe an unrecoverable application or system 
           crash, or a catastrophic
        // failure that requires immediate attention.
        Critical = 5,
        //
        // Summary:
        // Not used for writing log messages. Specifies that a logging 
           category should not
        // write any messages.
        None = 6
    }
```

我们还可以在配置日志记录器时设置最小跟踪级别，以下代码在 `Program.cs` 文件中创建 `WebHost`：

```cs
.ConfigureLogging(logging =&gt; logging.SetMinimumLevel(LogLevel.Warning))
```

我们还可以进行基于范围的日志记录；也就是说，在范围内的逻辑操作组中进行日志记录，以便将相同的数据附加到每个日志条目。`BeginScope` 方法专门为此目的而设计，以下示例展示了其用法：

```cs
 using (this.logger.BeginScope($"Logging scope demo"))
 {
      this.logger.LogInformation($"Calling {nameof(this.Index)}");
      return View();
 }
```

在我们总结之前，让我们看一下 `appSettings.json` 文件中 `Logging` 的配置。考虑以下 `Logging` 配置：

```cs
{
  "Logging": {
    "IncludeScopes": false,
    "Debug": {
      "LogLevel": {
        "Default": "Information"
      }
    },
    "Console": {
      "LogLevel": {       
        "Microsoft.AspNetCore.Mvc.Razor": "Error",
        "Default": "Information"
      }
    },
    "LogLevel": {
      "Default": "Debug"
    }
  }
}
```

上述配置定义了四个日志过滤器。以下是对配置的解释：

1.  基于范围的日志记录已被禁用，因为我们已将 `IncludeScopes` 设置为 `false`。基于范围的日志记录的主要用例之一是在事务性数据访问或操作中，您可能希望将相同的标识符附加到事务中发生的所有操作上。

1.  默认的 `Debug` 日志提供程序的日志级别对所有类别都是 `Information`。

1.  默认的 `Console` 日志提供程序的日志级别对所有类别都是 `Information`。

1.  以 `Microsoft.AspNetCore.Mvc.Razor` 开头的类别的日志级别对于 `Console` 日志提供程序是 `Error`。

1.  所有日志提供程序对所有类别的默认日志级别是 `Debug`。

对于对 ASP.NET Core 2.0 中的日志记录进行详细深入的了解，读者应访问 [`docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?tabs=aspnetcore2x`](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?tabs=aspnetcore2x)。[`docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?tabs=aspnetcore2x`](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?tabs=aspnetcore2x)

通过这一点，我们结束了对日志记录的讨论，以及本章的内容。

# 摘要

在本章中，我们讨论了 Let's Chat 网络应用程序，其需求和高层次设计，并完成了项目设置以开始编码。完成项目设置后，我们学习了 ASP.NET Core 的基础知识，例如身份验证、授权、中间件、依赖注入、配置和日志记录。

在下一章中，我们将完成基础知识的学习，然后深入编码 Let's Chat 应用程序。
