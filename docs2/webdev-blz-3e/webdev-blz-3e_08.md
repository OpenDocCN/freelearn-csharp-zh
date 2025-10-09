# 8

# 身份验证和授权

在本章中，我们将学习如何将 **身份验证** 和 **授权** 添加到我们的博客中，因为我们不希望任何人都能创建或编辑博客文章。

覆盖身份验证和授权可能需要一整本书，所以在这里我们将保持简单。本章的目标是让内置的身份验证和授权功能正常工作，基于已经内置到 ASP.NET 中的功能。这意味着这里没有太多 Blazor 魔法；已经存在许多我们可以利用的资源。

几乎每个系统今天都有一种登录方式，无论是管理员界面（如我们的）还是成员登录门户。有许多不同的登录提供者，如 Google、Twitter 和 Microsoft。我们可以使用所有这些提供者，因为我们只是在现有的架构上构建。

一些网站可能已经有一个用于存储登录凭证的数据库，但对我们博客来说，我们将使用名为 Auth0 的服务来管理我们的用户。这是一种非常强大的方式来添加许多不同的社交提供者（如果我们想的话），而且我们不必自己管理用户。

我们可以在创建项目时选择添加身份验证的选项。当涉及到 Blazor 服务器、Blazor WebAssembly 和 API 时，身份验证的工作方式不同，我们将在本章中更详细地探讨。

在本章中，我们将涵盖以下主题：

+   设置身份验证

+   保护 Blazor 服务器

+   保护 Blazor WebAssembly

+   保护 API

+   添加授权

# 技术要求

确保你已经遵循了前面的章节，或者以 `Chapter07` 文件夹作为起点。

你可以在 [`github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter08`](https://github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter08) 找到本章最终结果的源代码。

# 设置身份验证

在身份验证方面有许多内置功能。添加身份验证的最简单方法是创建项目时选择身份验证选项。

我们需要分别对 Blazor 服务器项目和 Blazor WebAssembly 项目实现身份验证，因为它们的工作方式不同。

但我们仍然可以在这两个项目之间共享一些东西。首先，我们需要设置 Auth0。

**Auth0** 是一种服务，可以帮助我们处理用户。有许多不同的服务，但 Auth0 对我们来说是一个很好的选择。我们可以连接一个或多个社交连接器，这将允许我们的用户使用 Facebook、Twitter、Twitch 或我们在网站上添加的任何其他服务进行登录。

尽管所有这些都可以通过我们自己编写代码来实现，但这种集成方式是快速添加身份验证并获得非常强大解决方案的绝佳方式。此外，身份验证很复杂，所以除非你确定自己在做什么，否则不要编写它。Auth0 对最多 7,000 个用户免费（我们博客可能达不到这个数字，尤其是管理员界面）。

它还具有添加我们有权访问的用户数据的强大功能。我们将在本章后面添加用户角色时进行此操作。你需要执行以下步骤：

1.  访问 [`auth0.com`](https://auth0.com) 并创建一个账户。

1.  点击**创建应用程序**按钮。

1.  现在，是时候给我们的应用程序命名了。例如使用 `MyBlog`。然后，是时候选择我们正在使用哪种应用程序类型了。是原生应用程序吗？是**单页 Web 应用程序**、**常规 Web 应用程序**还是**机器到机器应用程序**？

这取决于我们将要运行的主机模型。

目前项目设置的美好之处在于服务器将处理所有身份验证并将这些操作交给 WebAssembly（如果我们有一个在 InteractiveAuto 或 InteractiveWebAssembly 中运行的可组件）。但这不会限制功能，只会限制我们在设置应用程序时需要配置的内容。

如果我们打算只以 Blazor 服务器（InteractiveServer）运行，我们应该使用常规的 Web 应用程序。但我们可能希望将所有内容都改为在 InteractiveWebAssembly 中运行，所以在这里不要限制自己。

选择**单页应用程序**，这样我们就可以在任何主机模型中使用我们的身份验证。

接下来，我们将选择我们项目使用的技术。我们有 Apache、.NET、Django、Go 以及许多其他选择，但我们没有针对 Blazor 的特定选择，至少在撰写本文时没有。

只需跳过这一步并点击**设置**选项卡。

现在，我们将设置我们的应用程序。我们需要保存并稍后使用一些值。你需要确保你记下了`域名`、`客户端 ID`和`客户端密钥`，因为我们将在稍后使用这些。

如果我们向下滚动，我们可以更改徽标，但我们将跳过这一步。

1.  保持`应用程序登录 URI`为空。从 .NET 6 开始，端口号是随机的，所以请确保添加你应用程序的端口号：

    1.  `允许的回调 URL`：`https://localhost:PORTNUMBER/callback`

    1.  `允许的注销 URL`：`https://localhost:PORTNUMBER/`

允许的回调 URL 是 Auth0 在用户身份验证后将要调用的 URL，而允许的注销 URL 是用户注销后应重定向到的位置。

现在，在页面底部点击**保存更改**。

## 配置我们的 Blazor 应用程序

我们已经完成了 Auth0 的配置。接下来，我们将配置我们的 Blazor 应用程序。

在 .NET 中存储密钥的方式有很多（一个未签入的文件、Azure Key Vault 等）。你可以使用你最熟悉的一种。

我们将保持非常简单，并将机密信息存储在 `appsettings.json` 中。确保在提交时记住排除该文件。您不要将机密信息存入源代码控制。您可以在文件上右键单击并选择 **Git**，**忽略和取消跟踪项**。

要配置我们的 Blazor 项目，请按照以下步骤操作：

1.  在 `BlazorWebApp.Client` 项目的根目录下，添加一个名为 `UserInfo.cs` 的新类，并添加以下内容：

    ```cs
    namespace BlazorWebApp.Client;
    public class UserInfo
    {
        public required string UserId { get; set; }
        public required string Email { get; set; }    public required string[] Roles { get; set; }
    } 
    ```

这是从 Blazor 模板中提取的。

1.  在 `BlazorWebApp` 项目中，打开 `appsettings.json` 并将以下代码添加到现有应用程序设置对象的根目录：

    ```cs
     "Auth0": {
        "Authority": "Get this from the domain for your application at Auth0",
        "ClientId": "Get this from Auth0 setting"
      } 
    ```

这些是我们之前章节中记录的值。用我们自己的 `Auth0` 中的值替换这些值。

由于我们的网站是一个带有一些附加 Blazor 功能的 ASP.NET 网站，这意味着我们可以使用 NuGet 包来获得一些开箱即用的功能。

1.  在 `BlazorWebApp` 项目中，添加对 `Auth0.AspNetCore.Authentication NuGet` 包的引用。

1.  在项目的根目录下，创建一个名为 `PersistingServerAuthenticationStateProvider.cs` 的新类，并添加以下代码：

    ```cs
    using BlazorWebApp.Client;
    using Microsoft.AspNetCore.Components;
    using Microsoft.AspNetCore.Components.Authorization;
    using Microsoft.AspNetCore.Components.Server;
    using Microsoft.AspNetCore.Components.Web;
    using Microsoft.AspNetCore.Identity;
    using Microsoft.Extensions.Options;
    using System.Diagnostics;
    namespace BlazorWebApp;
    internal sealed class PersistingServerAuthenticationStateProvider : ServerAuthenticationStateProvider, IDisposable
    {
        private readonly PersistentComponentState state;
        private readonly IdentityOptions options;
        private readonly PersistingComponentStateSubscription subscription;
        private Task<AuthenticationState>? authenticationStateTask;
        public PersistingServerAuthenticationStateProvider(
            PersistentComponentState persistentComponentState,
            IOptions<IdentityOptions> optionsAccessor)
        {
            state = persistentComponentState;
            options = optionsAccessor.Value;
            AuthenticationStateChanged += OnAuthenticationStateChanged;
            subscription = state.RegisterOnPersisting(OnPersistingAsync, RenderMode.InteractiveWebAssembly);
        }
        private void OnAuthenticationStateChanged(Task<AuthenticationState> task)
        {
            authenticationStateTask = task;
        }
        private async Task OnPersistingAsync()
        {
            if (authenticationStateTask is null)
            {
                throw new UnreachableException($"Authentication state not set in {nameof(OnPersistingAsync)}().");
            }
            var authenticationState = await authenticationStateTask;
            var principal = authenticationState.User;
            if (principal.Identity?.IsAuthenticated == true)
            {
                var userId = principal.FindFirst(options.ClaimsIdentity.UserIdClaimType)?.Value;
                var email = principal.FindFirst(options.ClaimsIdentity.EmailClaimType)?.Value;
                var roles = principal.FindAll(options.ClaimsIdentity.RoleClaimType);

                if (userId != null)
                {
                    state.PersistAsJson(nameof(UserInfo), new UserInfo
                    {
                        UserId = userId,
                        Email = email,
                        Roles=roles.Select(r=>r.Value).ToArray()
                    });
                }
            }
        }
        public void Dispose()
        {
            subscription.Dispose();
            AuthenticationStateChanged -= OnAuthenticationStateChanged;
        }
    } 
    ```

1.  我已经从 Blazor 模板（当我们选择立即添加身份验证时）中提取了这个文件。我添加了角色，以便如果 `Auth0` 提供任何角色，它们也会存储在状态中。目前，`Auth0` 不会给我们任何角色，所以我们稍后再回到角色。正在发生的事情是，当我们登录时，它将在 `PersistentComponentState` 中保存已登录用户，这样我们就可以轻松地将用户转移到 WebAssembly。服务器将在 DOM 上渲染数据，然后 WebAssembly 将获取这些数据。

1.  打开 `Program.cs` 并在文件顶部添加以下代码：

    ```cs
    using BlazorWebApp;
    using Auth0.AspNetCore.Authentication;
    using Microsoft.AspNetCore.Authentication;
    using Microsoft.AspNetCore.Authentication.Cookies; 
    ```

1.  在 `WebApplication app = builder.Build();` 之前添加以下代码：

    ```cs
    builder.Services.AddScoped<AuthenticationStateProvider, PersistingServerAuthenticationStateProvider>();
    builder.Services.AddCascadingAuthenticationState();
    builder.Services
        .AddAuth0WebAppAuthentication(options =>
        {
            options.Domain = builder.Configuration["Auth0:Authority"]??"";;
            options.ClientId = builder.Configuration["Auth0:ClientId"]??"";;
        }); 
    ```

1.  在 Blazor 的早期版本中，我们必须首先确保当我们的组件请求 `AuthenticationStateProvider` 时，我们返回一个 `PersistingServerAuthenticationStateProvider` 的实例。我们还添加了对 `AddCascadingAuthenticationState` 的调用，这将确保无论托管方法如何，都会向所有组件发送 `AuthenticationState`。

1.  此外，在 `app.UseAntiforgery();` 之后添加以下代码。这段代码将允许我们保护我们的网站：

    ```cs
    app.UseAuthentication();
    app.UseAuthorization(); 
    ```

1.  在 `Program.cs` 中，在 `app.Run()` 之前添加以下代码：

    ```cs
    app.MapGet("account/login", async (string redirectUri, HttpContext
     context) =>
    {
        var authenticationProperties = new LoginAuthenticationPropertiesBuilder()
             .WithRedirectUri(redirectUri)
             .Build();
        await context.ChallengeAsync(Auth0Constants.AuthenticationScheme, authenticationProperties);
    }); 
    ```

当我们的网站重定向到 `authentication/login` 时，Minimal API 端点将启动登录功能。

1.  我们需要添加类似的注销功能。在 *步骤 7* 中的上一个端点下方添加以下代码：

    ```cs
    app.MapGet("authentication/logout", async (HttpContext context) =>
    {
        var authenticationProperties = new LogoutAuthenticationPropertiesBuilder()
             .WithRedirectUri("/")
             .Build();
        await context.SignOutAsync(Auth0Constants.AuthenticationScheme, authenticationProperties);
        await context.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
    }); 
    ```

需要注销两次，一次用于 `Auth0` 认证方案，一次用于 cookie 认证方案。配置全部完成。现在，我们需要一些东西来保护。

# 保护我们的 Blazor 应用

Blazor 使用 `App.razor` 进行路由。为了启用 Blazor 的保护，我们需要在应用程序组件中添加一些组件。

我们需要添加`CascadingAuthenticationState`，这将把身份验证状态发送到所有监听它的组件。我们还需要将路由视图更改为`AuthorizeRouteView`，它可以根据您是否已进行身份验证显示不同的视图：

1.  在`BlazorWebApp`项目中，打开`Components/_Imports.razor`并添加以下命名空间：

    ```cs
    @using Microsoft.AspNetCore.Components.Authorization
    @using BlazorWebApp.Components.Layout
    @using BlazorWebApp.Components 
    ```

1.  打开`Components/Routes.razor`组件，并将`Router`组件内的所有内容替换为以下内容：

    ```cs
    <Found Context="routeData">
         <AuthorizeRouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)">
             <Authorizing>
                 <p>Determining session state, please wait...</p>
             </Authorizing>
             <NotAuthorized>
                 <h1>Sorry</h1>
                 <p>You're not authorized to reach this page. You need to log in.</p>
             </NotAuthorized>
         </AuthorizeRouteView>
         <FocusOnNavigate RouteData="@routeData" Selector="h1" />
     </Found>
     <NotFound>
         <PageTitle>Not found</PageTitle>
         <LayoutView Layout="@typeof(MainLayout)">
             <p role="alert">Sorry, there's nothing at this address.</p>
         </LayoutView>
     </NotFound> 
    ```

在 Blazor 的早期版本中，我们不得不将代码包裹在`<CascadingAuthenticationState>`标签内，但使用.NET 8 后，这可以通过添加对`AddCascadingAuthenticationState`的调用来自动处理。

现在，只剩下两件事：一个我们可以保护页面和一个登录链接显示。

1.  在`Components`文件夹中，添加一个名为`LoginStatus.razor`的新 Razor 组件。

将内容替换为以下内容：

```cs
<AuthorizeView>
    <Authorized>
        <a href="authentication/logout">Log out</a>
    </Authorized>
    <NotAuthorized>
        <a href="account/login?returnUrl=/">Log in</a>
    </NotAuthorized>
</AuthorizeView> 
```

`LoginStatus`是一个组件，如果未进行身份验证，将显示登录链接；如果已进行身份验证，将显示注销链接。

1.  打开`Components/Layout/MainLayout.razor`：

    ```cs
    <AuthorizeView Roles="Administrator">
            <div class="sidebar">
                <NavMenu />
            </div>
    </AuthorizeView> 
    ```

将关于链接替换为以下内容：

```cs
<LoginStatus /> 
```

现在，我们的布局页面将显示我们是否已登录，并给我们提供登录或注销的机会。

1.  在`SharedComponents`和`BlazorWebApp.Client`项目中，在每个项目的`_Imports`文件中添加以下内容：

    ```cs
    @using Microsoft.AspNetCore.Authorization 
    ```

1.  将`authorize`属性添加到我们希望保护的组件中。

1.  在`SharedComponents`项目中，我们有以下组件：

`Pages/Admin/BlogPostEdit.razor`

`Pages/Admin/BlogPostList.razor`

`Pages/Admin/CategoryList.razor`

`Pages/Admin/TagList.razor`（在`BlazorWebApp.Client`项目中）

在上述每个组件中添加以下属性：

```cs
@attribute [Authorize] 
```

这就是全部所需，一些配置，然后我们就可以设置好了。

现在，启动我们的`BlazorWebApp`并查看您是否可以访问`/admin/blogposts`页面（剧透：您不应该能够访问）；登录（创建用户）并查看您现在是否可以访问该页面。

我们的管理界面已经得到了保护。

在下一节中，我们将确保我们的 Blazor WebAssembly 版本博客和 API 的安全性。

# 保护 Blazor WebAssembly

在本书的前几版中，我们构建了两个版本的博客，一个用于 Blazor 服务器，一个用于 Blazor WebAssembly。在本版中，整个重点是我们不必在两者之间做出选择。如前所述，有两个项目，`BlazorWebApp` 和 `BlazorWebApp.Client`。在客户端项目中，我们添加了所有我们希望作为 WebAssembly 运行的组件。这里有一个真正酷的部分。我们在客户端项目中有一个 `TagList` 组件。如果我们以 InteractiveAuto 运行它，它将首先使用 `BlazorWebApp` 项目的配置在服务器上使用 SignalR 进行渲染。但下次网站运行时，它将加载 WebAssembly 版本并使用 `BlazorWebApp.Client` 项目的配置。因此，同一个组件可以使用不同的依赖注入。在这种情况下，它将使用直接数据访问，而在另一种情况下，它将使用我们在上一章中创建的 API 客户端。

为了我们能够访问我们的 API，我们需要设置 `HttpClient`：

但首先，我们需要从服务器获取身份验证信息。WebAssembly 并不真正登录；它从服务器获取信息并使用身份验证 cookie 进行对服务器的额外调用。

在服务器项目中，我们添加了一个 `PersistingServerAuthenticationStateProvider` 来存储有关登录用户的信息。在客户端，我们需要获取这些信息。

1.  在 `BlazorWebApp.Client` 项目中，在根目录下添加一个名为 `PersistentAuthenticationStateProvider.cs` 的新类。

1.  将代码替换为以下内容：

    ```cs
    using BlazorWebApp.Client;
    using Microsoft.AspNetCore.Components;
    using Microsoft.AspNetCore.Components.Authorization;
    using System.Security.Claims;
    namespace BlazorApp1.Client;
    internal class PersistentAuthenticationStateProvider : AuthenticationStateProvider
    {
        private static readonly Task<AuthenticationState> defaultUnauthenticatedTask =
            Task.FromResult(new AuthenticationState(new ClaimsPrincipal(new ClaimsIdentity())));
        private readonly Task<AuthenticationState> authenticationStateTask = defaultUnauthenticatedTask;
        public PersistentAuthenticationStateProvider(PersistentComponentState state)
        {
            if (!state.TryTakeFromJson<UserInfo>(nameof(UserInfo), out var userInfo) || userInfo is null)
            {
                return;
            }
            List<Claim> claims = new();
            claims.Add(new Claim(ClaimTypes.NameIdentifier, userInfo.UserId));
            claims.Add(new Claim(ClaimTypes.Name, userInfo.Email??""));
            claims.Add(new Claim(ClaimTypes.Email, userInfo.Email??""));
            foreach (var role in userInfo.Roles)
            {
                claims.Add(new Claim(ClaimTypes.Role, role));
            }
            authenticationStateTask = Task.FromResult(
                new AuthenticationState(new ClaimsPrincipal(new ClaimsIdentity(claims,
                    authenticationType: nameof(PersistentAuthenticationStateProvider)))));
        }
        public override Task<AuthenticationState> GetAuthenticationStateAsync() => authenticationStateTask;
    } 
    ```

1.  这也是从 Blazor 模板（带有身份验证）中提取的。这里只进行了少量修改，比如添加角色。

1.  在 `BlazorWebApp.Client` 项目中的 `Program.cs` 文件中，在 `builder.Build().RunAsync()` 之上添加以下行；

    ```cs
    builder.Services.AddAuthorizationCore();
    builder.Services.AddCascadingAuthenticationState();
    builder.Services.AddSingleton<AuthenticationStateProvider, PersistentAuthenticationStateProvider>();
    builder.Services.AddHttpClient("Api",client => client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress)); 
    ```

这将启用身份验证，将级联 `身份验证状态` 添加到我们的组件中，并从 `Persistent Component State` 中获取登录用户。`HttpClient` 的名称是 “`Api`”；这是我们 *第七章*，*创建一个 API* 中使用的名称。

我们还需要设置依赖注入，以便当我们请求一个 `IBlogAPI` 时，我们将得到我们在 *第七章*，*创建一个 API* 中创建的 `BlogApiWebClient`。 

1.  在 `Program.cs` 文件中，在 `builder.Services.AddSingleton<AuthenticationStateProvider, PersistentAuthenticationStateProvider>();` 行之下添加以下代码：

    ```cs
    builder.Services.AddTransient<IBlogApi, BlogApiWebClient>(); 
    ```

现在，当我们请求一个 `IBlogApi` 时，我们将得到通过 API 访问数据的 API 客户端。这里真正酷的地方是，根据组件是在服务器（静态、InteractiveServer）上渲染、客户端（InteractiveWebAssembly）上渲染，还是组合（InteractiveAuto）渲染，它将选择适合该场景的正确客户端。我们有选择最适合的那个的能力。

1.  确保添加所需的命名空间。

现在，一切准备就绪，我们可以为在 WebAssembly 模式下运行进行安全设置了。

这个示例是关于在运行 ASP.NET 后端时保护 WebAssembly。在*第十六章*，*深入 WebAssembly*中，我们将探讨如何保护 Blazor WebAssembly 应用程序。

让我们试一试：

1.  在`BlazorWebApp.Client`项目中，打开`Pages/Admin/TagList.razor`。到目前为止，我们一直使用 InteractiveServer 渲染模式运行我们的组件。现在，让我们将其更改为 InteractiveWebAssembly 并运行它。

将`@rendermode InteractiveServer`更改为`@rendermode InteractiveWebAssembly`。就这样！现在，我们的组件将首先在服务器上渲染（因为我们正在运行服务器预渲染），然后 WebAssembly 将接管并再次渲染组件。它将拾取存储在组件状态中的认证信息，并使用我们的 Web API 从我们的 API 检索数据。这是因为当请求`IBlogApi`的实例时，WebAssembly 应用程序被配置为使用`BlogApiWebClient`。因此，相同的组件首先使用直接数据访问在服务器上预渲染，然后再次使用 Web API。非常酷！

现在，运行项目，导航到`/Admin/Tags`，并尝试编辑一些标签。这个组件现在正在 WebAssembly 上运行。您可以尝试将其更改为`@rendermode InteractiveAuto`。为了看到行为，这将首先连接 SignalR，然后在下一次加载时切换到 WebAssembly。

但如果不同的用户有不同的权限怎么办？

正是这里，角色派上用场。

# 添加角色

Blazor Server 和 Blazor WebAssembly 处理角色的方式略有不同；这不是什么大问题，但我们需要进行不同的实现。在本章中，我们将探讨为我们的当前项目（每个组件）实现它，并在*第十六章*，*深入 WebAssembly*中返回角色。

## 通过添加角色配置 Auth0

让我们从在 Auth0 中添加角色开始：

1.  登录到`Auth0`，导航到**用户管理** | **角色**，然后点击**创建角色**。

1.  输入名称`管理员`和描述`可以做任何事情`，然后按**创建**。

1.  前往**用户**选项卡，点击**添加用户**，搜索您的用户，然后点击**分配**。您也可以从左侧的**用户**菜单管理角色。

1.  默认情况下，角色不会发送到客户端，因此我们需要丰富数据以包括角色。

我们通过添加一个操作来实现这一点。

1.  前往**操作**，然后**流程**。

流程是在特定流程中执行代码的一种方式。

我们希望`Auth0`在我们登录时添加我们的角色。

1.  选择**登录**，在那里我们将看到流程；在我们的情况下，我们还没有任何内容。

1.  在右侧，点击**自定义**和加号。当一个小弹出菜单出现时，选择**从头开始构建**。

1.  将操作命名为`添加角色`，保持**触发器**和**运行时**不变，然后按**创建**。

我们将看到一个窗口，我们可以在这里编写我们的操作。

1.  将所有代码替换为以下内容：

    ```cs
    /**
     * @param {Event} event - Details about the user and the context in which they are logging in.
     * @param {PostLoginAPI} api - Interface whose methods can be used to change the behavior of the login.
     */
    exports.onExecutePostLogin = async (event, api) => {
      const claimName  = 'http://schemas.microsoft.com/ws/2008/06/identity/claims/role'
      if (event.authorization) {
        api.idToken.setCustomClaim(claimName, event.authorization.roles);
        api.accessToken.setCustomClaim(claimName, event.authorization.roles);
      }
    } 
    ```

1.  点击**部署**然后**返回流程**。

1.  再次点击**自定义**，我们将看到我们刚刚创建的操作。

1.  将**添加角色**操作拖到**开始**和**完成**之间的箭头上。

1.  点击**应用**。

现在，我们有一个将角色添加到我们的登录令牌中的操作。

我们的用户现在是一个管理员。值得注意的是，角色是 Auth0 的付费功能，并且仅在试用期间免费。

现在，让我们设置 Blazor 以使用这个新角色。

## 将角色添加到 Blazor

由于我们正在使用 Auth0 库，Blazor 的设置几乎已经完成。

让我们修改一个组件以显示用户是否是管理员：

1.  在 `Components` 项目中，打开 `Shared/NavMenu.razor`。

1.  在组件顶部添加以下内容：

    ```cs
    <AuthorizeView Roles="Administrator">
        <Authorized>
            Hi admin!
        </Authorized>
        <NotAuthorized>
            You are not an admin =(
        </NotAuthorized>
    </AuthorizeView> 
    ```

1.  现在，运行我们的 BlazorWebApp 项目。

如果我们登录，我们应该能够看到左侧的文字，在深蓝色背景上显示黑色文字“**Hi Admin!**”，这可能不是很明显。我们将在**第九章**，**共享代码和资源**中解决这个问题。

# 摘要

在本章中，我们学习了如何将身份验证添加到我们现有的网站上。在创建项目时添加身份验证更容易。然而，现在我们对底层发生了什么以及如何处理添加外部身份验证源有了更好的理解。

在整本书中，我们已经在不同的托管模型之间共享了组件。

在下一章中，我们将探讨共享更多内容，例如静态文件和 CSS，并尝试使一切看起来都很漂亮。
