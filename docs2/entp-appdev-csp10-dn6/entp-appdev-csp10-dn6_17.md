# *第十三章*: 在.NET 6 中实现授权

构建安全应用程序的一个重要方面是确保用户只能访问他们需要的资源。在现实生活中，当你入住酒店时，前台员工会验证你的身份证和信用卡，并分配一张房卡以进入你的房间。根据你选择的房型，你可能享有一些特权，例如进入休息室、游泳池或健身房等。在这里，验证你的身份证和信用卡以及分配房卡被称为**认证**，而允许你访问各种资源被称为**授权**。所以，为了进一步解释，使用房卡，我们无法识别你是谁，但可以确定你能做什么。

授权是一种机制，通过它你可以确定用户可以做什么，并授予或拒绝对应用程序资源的访问权限。例如，我们电子商务应用程序的用户应该能够浏览产品、将它们添加到购物车并结账购买，而只有管理员或后台办公室用户应该能够添加或更新产品信息、更新产品价格以及批准或拒绝订单。

在本章中，我们将学习什么是授权，以及使用 ASP.NET Core 框架实现授权的各种方法。本章涵盖了以下主题：

+   理解.NET 6 中的授权

+   简单授权

+   基于角色的授权

+   基于声明（Claims）的授权

+   基于策略的授权

+   自定义授权

+   客户端和服务器应用程序中的授权

# 技术要求

对于本章，你需要具备 Azure、Azure AD B2C、C#、.NET Core 和 Visual Studio 2022 的基本知识。

# 回到一些基础知识

在我们深入更多细节之前，让我们了解认证和授权之间的区别。

认证和授权看起来可能相似，并且可以互换使用，但本质上它们是不同的。以下表格说明了这些差异：

![表 13.1](img/Table_13.1.jpg)

表 13.1

注意

请参阅*第十二章*，*理解认证*，以获取更多关于 ASP.NET 6 中认证工作原理的详细信息。

总结一下，认证和授权是相辅相成的。授权只有在用户的身份确定之后才会生效，当用户尝试访问受保护资源时，授权会触发认证挑战。在本章的后续部分，我们将了解如何在 ASP.NET 6 应用程序中实现授权。

# 理解授权

ASP.NET Core 中的授权由 **中间件** 处理。当你的应用程序收到来自未经身份验证用户的第一个请求到受保护资源时，中间件将触发身份验证挑战，根据身份验证方案，用户要么被重定向到登录页面，要么访问被禁止。一旦在身份验证后确定了用户的身份，授权中间件会检查用户是否可以访问该资源。在后续请求中，授权中间件使用用户的身份来确定是否允许访问。

要在项目中配置授权中间件，你需要在 `Program.cs` 中调用 `UseAuthorization()`。在注册授权中间件之前，必须先注册身份验证中间件，因为授权只能在确定用户身份后执行。请参考以下代码：

```cs
var builder = WebApplication.CreateBuilder(args);
```

```cs
..
```

```cs
var app = builder.Build();
```

```cs
app.UseAuthentication();
```

```cs
app.UseAuthorization();
```

```cs
app.UseEndpoints(endpoints =>
```

```cs
{
```

```cs
endpoints.MapControllerRoute(
```

```cs
    name: "default",
```

```cs
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

```cs
    endpoints.MapRazorPages();
```

```cs
});
```

在前面的代码块中，你会注意到 `app.UseAuthorization()` 在 `app.UseAuthentication()` 之后和 `app.UseEndpoints()` 之前被调用。

ASP.NET 6 提供了简单、基于角色和声明的授权模型以及丰富的基于策略的模型。在接下来的章节中，我们将学习更多关于这些模型的具体细节。

## 简单授权

在 ASP.NET Core 中，授权是通过 `AuthorizationAttribute` 配置的。你可以在控制器、操作或 Razor 页面上应用 `[Authorize]` 属性。当你添加这个属性时，对该组件的访问权限将仅限于经过身份验证的用户。请参考以下代码块：

```cs
public class HomeController : Controller
```

```cs
{
```

```cs
[Authorize]
```

```cs
public IActionResult Index()
```

```cs
{
```

```cs
    return View();
```

```cs
}
```

```cs
public IActionResult Privacy()
```

```cs
{
```

```cs
    return View();
```

```cs
}
```

```cs
}
```

在前面的代码中，你会注意到 `[Authorize]` 属性被添加到了 `Index` 操作上。当用户尝试从浏览器访问 `/Home/Index` 时，中间件会检查用户是否已经经过身份验证。如果没有，用户将被重定向到登录页面。

如果我们将 `[Authorize]` 属性添加到一个控制器中，那么该控制器下所有操作的访问权限将仅限于经过身份验证的用户。在下面的代码中，你会注意到 `[Authorize]` 属性被添加到了 `HomeController` 中，使得其下的所有操作都变得安全：

```cs
[Authorize]
```

```cs
public class HomeController : Controller
```

```cs
{
```

```cs
public IActionResult Index()
```

```cs
{
```

```cs
    return View();
```

```cs
}
```

```cs
[AllowAnonymous]
```

```cs
public IActionResult Privacy()
```

```cs
{
```

```cs
    return View();
```

```cs
}
```

```cs
}
```

有时，你可能希望允许应用程序的某些区域对任何用户都开放；例如，登录或重置密码页面应该对所有用户开放，无论用户是否经过身份验证。为了满足这样的要求，你可以在控制器或操作上添加 `[AllowAnonymous]` 属性，使它们对未经身份验证的用户也开放。

在前面的代码中，你会注意到 `[AllowAnonymous]` 属性被添加到了 `Privacy` 操作上，尽管控制器上已经有了 `[Authorize]` 属性。这个要求被操作方法上的 `[AllowAnonymous]` 属性覆盖，因此 `Privacy` 操作对所有用户都是可访问的。

注意

`[AllowAnonymous]` 属性会覆盖所有授权配置。如果你在一个控制器上设置了 `[AllowAnonymous]`，那么在它下面的任何操作方法上设置 `[Authorize]` 属性将没有任何影响。在这种情况下，操作方法上的 `Authorize` 属性将被完全忽略。

到目前为止，我们已经看到了如何保护控制器或操作方法。在下一节中，我们将看到如何在 ASP.NET Core 应用程序中全局启用授权。

## 全局启用授权

到目前为止，我们已经看到了如何使用 `[Authorize]` 属性来保护控制器或操作方法。在大型项目中，为每个控制器或操作设置 `authorize` 属性是不可持续的；你可能会错过为新添加的控制器或操作方法配置它，这可能导致安全漏洞。

ASP.NET Core 允许你通过在应用程序中添加回退策略来全局启用授权。你可以在 `Program.cs` 中定义回退策略。回退策略将应用于所有未定义显式授权要求的请求：

```cs
var builder = WebApplication.CreateBuilder(args);
```

```cs
builder.Services.AddAuthorization(options =>
```

```cs
{
```

```cs
options.FallbackPolicy = new AuthorizationPolicyBuilder()
```

```cs
        .RequireAuthenticatedUser()
```

```cs
        .Build();
```

```cs
});
```

在全局范围内添加策略强制用户进行身份验证才能访问应用程序中的任何操作方法。这个选项很有用，因为你不需要为应用程序中的每个控制器/操作指定 `[Authorize]` 属性。

你仍然可以在控制器或操作方法上设置 `[AllowAnonymous]` 属性来覆盖默认行为，使其匿名访问。

现在我们已经了解了如何实现简单的授权，在下一节中，让我们了解基于角色的授权是什么以及它是如何简化实现的。

# 基于角色的授权

在你的应用程序的某些区域仅对特定用户可用是很常见的。而不是在用户级别授予访问权限，通常的做法是将用户分组到角色中，并授予角色访问权限。让我们考虑一个典型的电子商务应用程序，其中 *用户* 可以下订单，*支持* 员工可以查看、更新或取消订单并解决用户查询，而 *管理员* 角色则批准或拒绝订单，管理库存等。

基于角色的授权可以解决这样的需求。当你创建一个用户时，你可以将其分配到一个或多个角色中，当我们配置 `[Authorize]` 属性时，我们可以将一个或多个角色名称传递给 `Authorize` 属性的 `Roles` 属性。

以下代码限制了 `Admin` 控制器下所有操作方法的访问权限，仅限于属于 `Admin` 角色的用户：

```cs
[Authorize(Roles ="Admin")]
```

```cs
public class AdminController : Controller
```

```cs
{
```

```cs
public IActionResult Index()
```

```cs
{
```

```cs
    return View();
```

```cs
}
```

```cs
}
```

类似地，你可以在 `Authorize` 属性的 `Roles` 属性中指定以逗号分隔的角色名称，这样属于配置的任一角色的用户都将能够访问该控制器下的操作方法。

在以下代码中，你会注意到 `User,Support` 被用作 `[Authorize]` 属性 `Roles` 属性的值；属于 `User` 或 `Support` 角色的用户可以访问 `OrdersController` 的操作方法：

```cs
[Authorize(Roles ="User,Support")]
```

```cs
public class OrdersController : Controller
```

```cs
{
```

```cs
public IActionResult Index()
```

```cs
{
```

```cs
  return View();
```

```cs
}
```

```cs
}
```

您也可以指定多个授权属性。如果您这样做，用户必须是所有指定角色的成员才能访问它。

在以下代码中，对 `InventoryController` 配置了多个 `[Authorize]` 属性，用于 `InventoryManager` 和 `Admin` 角色。要访问 `Inventory` 控制器，用户必须具有 `InventoryManager` 和 `Admin` 角色：

```cs
[Authorize(Roles ="InventoryManager")]
```

```cs
[Authorize(Roles ="Admin")]
```

```cs
public class InventoryController : Controller
```

```cs
{
```

```cs
public IActionResult Index()
```

```cs
{
```

```cs
  return View();
```

```cs
}
```

```cs
[Authorize(Roles ="Admin")]
```

```cs
public IActionResult Approve()
```

```cs
{
```

```cs
  return View();
```

```cs
}
```

```cs
}
```

您可以通过指定授权属性进一步限制对 `Inventory` 控制器下操作方法的访问。在前面的代码中，用户必须具有 `InventoryManager` 和 `Admin` 角色才能访问 `Approve` 操作。

通过编程方式，如果您想检查用户是否属于某个角色，您可以使用 `ClaimsPrinciple` 的 `IsInRole` 方法。在以下示例中，您会注意到 `User.IsInRole` 接受 `roleName`，并根据用户的角色返回 `true` 或 `false`：

```cs
public ActionResult Index()
```

```cs
{
```

```cs
if (User.IsInRole("Admin"))
```

```cs
{
```

```cs
    // Handle your logic
```

```cs
}
```

```cs
return View();
```

```cs
}
```

到目前为止，我们已经看到了如何通过在授权属性中指定角色名称来通过指定角色名称来保护控制器或操作。在下一节中，我们将看到如何使用基于策略的角色授权将这些配置集中在一个地方。

## 基于策略的角色授权

我们也可以在 `Program.cs` 中将角色要求定义为策略。这种方法非常有用，因为您可以在一个地方创建和管理基于角色的访问要求，并使用策略名称而不是角色名称来控制访问。为了定义基于策略的角色授权，我们需要在 `Program.cs` 中将一个或多个角色要求注册为授权策略，并将策略名称提供给 `Authorize` 属性的 `Policy` 属性。

在以下代码中，通过添加具有 `Admin` 角色的要求创建 `AdminAccessPolicy`：

```cs
var builder = WebApplication.CreateBuilder(args);
```

```cs
builder.Services.AddAuthorization(options =>
```

```cs
{
```

```cs
    options.AddPolicy("AdminAccessPolicy",
```

```cs
        policy => policy.RequireRole("Admin"));
```

```cs
});
```

在您的控制器中，您可以指定要应用的政策如下，并且对 `AdminController` 的访问被限制为具有 `Admin` 角色的用户：

```cs
[Authorize(Policy ="AdminAccessPolicy")]
```

```cs
public class AdminController : Controller
```

```cs
{
```

```cs
public IActionResult Index()
```

```cs
{
```

```cs
    return View();
```

```cs
}
```

```cs
}
```

在定义策略时，您可以指定多个角色。当使用该策略授权用户时，属于任何角色的用户都可以访问资源。例如，以下代码将允许具有 `User` 或 `Support` 角色的用户访问资源：

```cs
options.AddPolicy("OrderAccessPolicy",
```

```cs
        policy => policy.RequireRole("User","Support"));
```

您可以在控制器或操作方法上使用带有 `Authorize` 属性的 `OrderAccessPolicy` 策略来控制访问。

现在我们已经了解了如何使用基于角色的授权，在下一节中，我们将创建一个简单的应用程序并将其配置为使用基于角色的授权。

## 实现基于角色的授权

让我们创建一个示例应用程序，使用 ASP.NET Core Identity 实现基于角色的授权：

1.  创建一个新的 ASP.NET Core 项目。您可以使用以下 `dotnet` `Individual` 账户作为 `Authentication` 模式，并使用 `SQLite` 作为数据库存储：

    ```cs
    dotnet new mvc --auth Individual -o AuthSample
    ```

1.  你需要在 `Program.cs` 中调用 `AddRoles<IdentityRole>()` 来启用角色服务。你可以参考以下代码来启用它。你还会注意到 `RequireConfirmedAccount` 被设置为 `false`。这对于本示例是必需的，因为我们以编程方式创建用户：

    ```cs
    {
    var builder = WebApplication.CreateBuilder(args);
    var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
    builder.Services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlite(connectionString));
    builder.Services.AddDatabaseDeveloperPageExceptionFilter();
    builder.Services.AddDefaultIdentity<IdentityUser>(options =>
        options.SignIn.RequireConfirmedAccount = false)
        .AddRoles<IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>();
    builder.Services.AddControllersWithViews();   
    }
    ```

1.  接下来，我们需要创建角色和用户。为此，我们将在 `Program.cs` 中添加两个方法，`SetupRoles` 和 `SetupUsers`。我们可以使用 `RoleManager` 和 `UserManager` 服务来创建角色和用户。在以下代码中，我们创建了三个角色。使用 `IServiceProvider`，我们获取 `roleManager` 服务的实例，然后我们使用 `RoleExistsAsync` 和 `CreateAsync` 方法来创建它：

    ```cs
    //Add this method to Program.cs
    async Task SetupRoles(IServiceProvider serviceProvider)
    {
    var rolemanager = serviceProvider
        .GetRequiredService<RoleManager<IdentityRole>>();
    string[] roles = { "Admin", "Support", "User" };
    foreach (var role in roles)
    {
        var roleExist = await rolemanager.RoleExistsAsync(role);
        if (!roleExist)
        {
            await rolemanager.CreateAsync(new 
              IdentityRole(role));
        }
       }
    }
    ```

1.  同样，我们使用 `userManager` 服务创建用户并分配一个角色。在以下代码中，我们创建了两个用户 – `admin@abc.com`，分配了 `admin` 角色，以及 `support@abc.com`，分配了 `support` 角色：

    ```cs
    //Add this method to Program.cs
    async Task SetupUsers(IServiceProvider serviceProvider)
    {
    var userManager = serviceProvider
        .GetRequiredService<UserManager<IdentityUser>>();
    var adminUser = await userManager.FindByEmailAsync("admin@abc.com");
    if (adminUser == null)
    {
        var newAdminUser = new IdentityUser
        {
            UserName = "admin@abc.com",
            Email = "admin@abc.com",
        };
    var result = await userManager
        .CreateAsync(newAdminUser, "Password@123");
    if (result.Succeeded)
        await userManager.AddToRoleAsync(newAdminUser, 
          "Admin");
    }
    var supportUser = await userManager
        .FindByEmailAsync("support@abc.com");
    if (supportUser == null)
    {
        var newSupportUser = new IdentityUser
        {
            UserName = "support@abc.com",
            Email = "support@abc.com",
        };
    var result = await userManager
        .CreateAsync(newSupportUser, "Password@123");
    if (result.Succeeded)
        await userManager.AddToRoleAsync(newSupportUser, 
          "Support");
    }
    }
    ```

1.  要调用这两个方法，我们需要一个 `IServiceProvider` 的实例。以下代码获取 `IServiceProvider` 的实例来设置角色和用户数据：

    ```cs
    //
    //
    var app = builder.Build();
    using (var scope = app.Services.CreateScope())
    {
        var services = scope.ServiceProvider;
        await SetupRoles(services);
        await SetupUsers(services);
    }
    ```

1.  在 `Home` 控制器内部，添加以下代码。为了简化实现，我们正在使用 `Index` 视图。在实际场景中，你需要返回为相应操作方法创建的视图：

    ```cs
    [Authorize(Roles = "Admin")]
    public IActionResult Admin()
    {
    return View("Index");
    }
    [Authorize(Roles = "Support")]
    public IActionResult Support()
    {
    return View("Index");
    }
    ```

1.  可选地，我们可以在 `Layout.cshtml` 中添加逻辑来根据登录用户的角色显示导航链接。以下示例使用 `IsInRole` 来检查用户的角色并显示一个链接：

    ```cs
    <li class="nav-item">
    <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Index">Home</a>
    </li>
    @if (User.IsInRole("Admin"))
    {
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Admin">Admin</a>
    </li>
    }
    @if (User.IsInRole("Support"))
    {
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Support">Support</a>
    </li>
    }
    ```

通过前面的步骤，示例实现完成，你可以运行应用程序来查看其工作方式。

运行应用程序，使用 `admin@abc.com` 登录，你会注意到 **管理员** 菜单项是可见的，而 **支持** 是隐藏的：

![图 13.1 – 管理员用户登录视图](img/Figure_13.1_B18507.jpg)

图 13.1 – 管理员用户登录视图

当你使用 `support@abc.com` 登录时，你会注意到 **支持** 菜单项是可见的，而 **管理员** 项是隐藏的：

![图 13.2 – 支持用户登录视图](img/Figure_13.2_B18507.jpg)

图 13.2 – 支持用户登录视图

在下一节中，我们将看到如何使用声明进行授权。

# 基于声明的授权

**声明**是在身份验证成功后与身份相关联的关键值对。声明可以是出生日期、性别或邮政编码等。一个或多个声明可以分配给一个用户。基于声明的授权使用声明的值来确定是否可以授予对资源的访问权限。你可以使用两种方法来验证声明；一种方法只是检查声明是否存在，另一种方法是检查声明是否存在并具有特定的值。

要使用基于声明的授权，我们需要在 `Program.cs` 中注册一个策略。你需要传递一个声明名称和可选值到 `RequireClaim` 方法来注册。例如，以下代码注册了 `PremiumContentPolicy` 并要求 `PremiumUser` 声明：

```cs
var builder = WebApplication.CreateBuilder(args);
```

```cs
builder.Services.AddAuthorization(options =>
```

```cs
{
```

```cs
    options.AddPolicy("PremiumContentPolicy",
```

```cs
        policy => policy.RequireClaim("PremiumUser"));
```

```cs
});
```

在以下代码中，`PremiumContentPolicy`授权策略被用于`PremiumContentController`。它检查用户声明中是否存在`PremiumUser`声明来授权用户的请求；它不关心声明中的值是什么：

```cs
[Authorize(Policy ="PremiumContentPolicy")]
```

```cs
public class PremiumContentController : Controller
```

```cs
{
```

```cs
public IActionResult Index()
```

```cs
{
```

```cs
        return View();
```

```cs
}
```

```cs
}
```

你也可以在定义声明时指定一个值列表。它们将被验证以授予对资源的访问权限。例如，根据以下代码，如果用户具有`Country`声明，其值为`US`、`UK`或`IN`，则用户请求被授权：

```cs
var builder = WebApplication.CreateBuilder(args);
```

```cs
builder.Services.AddAuthorization(options =>
```

```cs
{
```

```cs
    options.AddPolicy("ExpressShippingPolicy",
```

```cs
        policy => policy.RequireClaim(ClaimTypes.Country, 
```

```cs
          "US", "UK", "IN"));
```

```cs
});
```

在程序上，如果你想检查一个用户是否有声明，你可以通过指定一个匹配条件来使用`ClaimsPrincipal`的`HasClaim`方法。

要获取一个声明值，你可以使用`FindFirst`方法。以下代码展示了示例：

```cs
@if (User.HasClaim(x => x.Type == "PremiumUser"))
```

```cs
{
```

```cs
    <h1>Yay, you are Premium User!!!, @User.FindFirst(x => 
```

```cs
        x.Type == ClaimTypes.Country)?.Value</h1>
```

```cs
}
```

如在*实现基于角色的授权*部分所见，在向应用程序添加用户时，你也可以使用`UserManager`服务向用户添加一个声明。在以下代码中，你会注意到`AddClaimAsync`方法与`IdentityUser`和`Claim`一起被调用：

```cs
var user = await userManager.FindByEmailAsync("user@abc.com");
```

```cs
if (user == null)
```

```cs
{
```

```cs
var newUser = new IdentityUser
```

```cs
{
```

```cs
    UserName = "user@abc.com",
```

```cs
    Email = "user@abc.com",
```

```cs
};
```

```cs
var result = await userManager.CreateAsync(newUser, "Password@123");
```

```cs
if (result.Succeeded)
```

```cs
{
```

```cs
await userManager
```

```cs
    .AddToRoleAsync(newUser, "User");
```

```cs
await userManager
```

```cs
    .AddClaimAsync(newUser, new Claim("PremiumUser", 
```

```cs
       "true"));
```

```cs
await userManager
```

```cs
    .AddClaimAsync(newUser, new Claim(ClaimTypes.Country, 
```

```cs
       "US"));
```

```cs
await userManager
```

```cs
                .AddClaimAsync(newUser, new Claim(ClaimTypes.DateOfBirth, "1-5-2003"));
```

```cs
}
```

```cs
}
```

在前面的代码中，你会注意到创建了两个声明，并使用`AddClaimAsync`方法与用户关联。在下一节中，我们将看到如何使用基于策略的授权。

# 基于策略的授权

基于策略的授权允许你编写自己的逻辑来处理满足你需求的授权需求。例如，你有一个验证用户年龄的需求，并且只有当用户年龄超过 14 岁时才授权下单。你可以使用基于策略的授权模型来处理这样的需求。

要配置基于策略的授权，我们需要定义一个需求和处理器，然后使用需求注册策略。让我们了解这些组件：

+   **策略**是通过一个或多个需求定义的。

+   **需求**是一组数据参数的集合，政策使用这些参数来评估用户的身份。

+   **处理器**负责评估需求中的数据与上下文之间的数据，并确定是否可以授予访问权限。

在下一节中，我们将看到如何创建一个需求和处理器，并注册一个授权策略。

## 需求

要创建一个需求，你需要实现`IAuthorizationRequirement`接口。这是一个标记接口，所以你不需要实现任何成员。例如，以下代码创建了一个`MinimumAgeRequirement`，其中`MinimumAge`是一个数据参数：

```cs
public class MinimumAgeRequirement: IAuthorizationRequirement
```

```cs
{
```

```cs
public int MinimumAge { get; set; }
```

```cs
public MinimumAgeRequirement(int minimumAge)
```

```cs
{
```

```cs
    this.MinimumAge = minimumAge;
```

```cs
}
```

```cs
}
```

## 需求处理器

需求处理器封装了允许或拒绝请求的逻辑。它们使用需求属性与`AuthorizationHandlerContext`进行比较以确定访问权限。处理器可以继承`Authorizationhandler<TRequirement>`，其中`TRequirement`是`IauthorizationRequirement`类型，或者实现`IAuthorizationHandler`。

在以下示例中，通过从 `AuthorizationHandler` 继承并使用 `MinimumAgeRequirement` 作为 `TRequirement` 来创建 `MinimumAgeAuthorizationHandler`。我们需要重写 `HandleRequirementAsync` 来编写自定义授权逻辑，其中用户的年龄是从 `DateOfBirth` 声明中计算出来的。如果用户的年龄大于或等于 `MinimumAge`，我们调用 `context.Succeed` 以授予访问权限。如果声明不存在或不符合年龄标准，则禁止访问：

```cs
public class MinimumAgeAuthorizationHandler
```

```cs
: AuthorizationHandler<MinimumAgeRequirement>
```

```cs
{
```

```cs
protected override Task HandleRequirementAsync(
```

```cs
AuthorizationHandlerContext context,
```

```cs
MinimumAgeRequirement requirement)
```

```cs
{
```

```cs
    if (context.User.HasClaim(
```

```cs
        c => c.Type == ClaimTypes.DateOfBirth))
```

```cs
    {
```

```cs
        var dateOfBirth = Convert.ToDateTime(
```

```cs
            context.User.FindFirst(x =>
```

```cs
            x.Type == ClaimTypes.DateOfBirth).Value);
```

```cs
        var age = DateTime.Today.Year - dateOfBirth.Year;
```

```cs
        if (dateOfBirth > DateTime.Today.AddYears(-age)) 
```

```cs
          age--;
```

```cs
        if (age >= requirement.MinimumAge)
```

```cs
        {
```

```cs
            context.Succeed(requirement);
```

```cs
        }
```

```cs
       else
```

```cs
        {
```

```cs
            context.Fail();
```

```cs
        }
```

```cs
    }
```

```cs
            return Task.CompletedTask;
```

```cs
}
```

```cs
}
```

要标记一个需求为成功，您需要通过传递一个需求作为参数来调用 `context.Succeed`。您不需要处理失败，因为可能还有另一个处理程序会成功处理相同的需求。如果您想禁止请求，可以调用 `context.Fail`。

注意

必须在 `Program.cs` 中为服务集合注册处理程序。

## 注册策略

在 `Program.cs` 中，使用名称和需求注册策略。在定义策略时，您可以注册一个或多个需求。

在以下示例中，通过调用 `policy.Requirements.Add()` 并传递 `MinimumAgeRequirement` 的新实例来创建一个包含需求的策略。您还会注意到 `MinimumAgeAuthorizationHandler` 被添加到服务集合中，具有单例作用域：

```cs
var builder = WebApplication.CreateBuilder(args);
```

```cs
builder.Services.AddAuthorization(options =>
```

```cs
{
```

```cs
options.AddPolicy("Over14", policy =>
```

```cs
    policy.Requirements.Add(new 
```

```cs
      MinimumAgeRequirement(14)));
```

```cs
});
```

```cs
builder.Services.AddSingleton<IAuthorizationHandler,
```

```cs
    MinimumAgeAuthorizationHandler>();
```

然后，我们可以在控制器或操作上配置授权策略，根据用户的年龄来限制访问：

```cs
[Authorize(Policy ="Over14")]
```

```cs
public class OrdersController : Controller
```

```cs
{
```

```cs
public IActionResult Index()
```

```cs
{
```

```cs
        return View();
```

```cs
}
```

```cs
}
```

如果我们注册了一个包含多个需求的策略，那么所有需求都必须得到满足才能成功授权。

在下一节中，我们将学习如何进一步自定义授权。

# 自定义授权

在上一节中，我们学习了如何使用基于策略的授权并实现自定义逻辑来处理授权需求。但，并不总是可以在 `Program.cs` 中像那样注册授权策略。在本节中，我们将看到如何使用 `IAuthorizationPolicyProvider` 在您的应用程序中动态构建策略配置。

`IAuthorizationPolicyProvider` 接口有三个方法需要实现：

+   `GetDefaultPolicyAsync`：此方法返回要使用的默认授权策略。

+   `GetFallbackPolicyAsync`：此方法返回回退授权策略。当没有定义明确的授权需求时使用。

+   `GetPolicyAsync`：此方法用于构建并返回提供的策略名称的授权策略。

让我们来看一个例子，您可能想要根据不同的年龄标准授权对多个控制器/操作的请求，比如 `Over14`、`Over18`、`Over21`、`Over60` 等。实现它的一个方法是将所有这些需求注册为策略并在您的控制器或操作中使用它们。但是，使用这种方法，代码的可维护性较低，在具有许多策略的大型应用程序中不可持续。让我们看看我们如何利用授权策略提供者。

我们需要创建一个实现 `IAuthorizationPolicyProvider` 的类，并实现 `GetPolicy` 和其他方法。

在以下示例中，`MinimumAgePolicyProvider` 类实现了 `GetPolicyAsync` 方法。此方法的输入是策略名称。由于我们的策略名称类似于 `Over14` 或 `Over18`，我们可以使用字符串函数从中提取年龄，并使用所需的年龄初始化一个要求，并将其注册为新的策略：

```cs
public class MinimumAgePolicyProvider : IAuthorizationPolicyProvider
```

```cs
{
```

```cs
        const string PREFIX = "Over";
```

```cs
        public Task<AuthorizationPolicy?> 
```

```cs
          GetPolicyAsync(string policyName)
```

```cs
        {
```

```cs
            if (policyName.StartsWith(PREFIX, 
```

```cs
              StringComparison.OrdinalIgnoreCase) &&
```

```cs
            int.TryParse(policyName.Substring(
```

```cs
              PREFIX.Length), out var age))
```

```cs
            {
```

```cs
                var policy = new 
```

```cs
                  AuthorizationPolicyBuilder();
```

```cs
                policy.AddRequirements(new 
```

```cs
                  MinimumAgeRequirement(age));
```

```cs
                return Task.FromResult 
```

```cs
                  <AuthorizationPolicy?>(policy.Build());
```

```cs
            }
```

```cs
            return 
```

```cs
              Task.FromResult<AuthorizationPolicy?>(null);
```

```cs
        }
```

```cs
}
```

注意

对于 `MinimumAgeRequirement` 的实现，请参阅 *基于策略的授权* 部分。

ASP.NET Core 只使用一个 `IAuthorizationPolicyProvider` 实例。因此，您应该自定义一个 `Default` 和 `Fallback` 授权策略，或者使用备用提供程序。

在以下代码中，您将看到 `MinimumAgePolicyProvider` 类中 `GetDefaultPolicyAsync` 和 `GetFallbackPolicyAsync` 方法的示例实现。

`AuthorizationOptions` 注入到构造函数中，并用于初始化 `DefaultAuthorizationPolicyProvider`。`BackupPolicyProvider` 对象用于实现 `GetDefaultPolicyAsync` 和 `GetFallbackPolicyAsync` 方法：

```cs
public MinimumAgePolicyProvider(IOptions<AuthorizationOptions> options)
```

```cs
{
```

```cs
this.BackupPolicyProvider =
```

```cs
    new DefaultAuthorizationPolicyProvider(options);
```

```cs
}
```

```cs
Private DefaultAuthorizationPolicyProvider BackupPolicyProvider { get; }
```

```cs
public Task<AuthorizationPolicy> GetDefaultPolicyAsync()
```

```cs
=> this.BackupPolicyProvider.GetDefaultPolicyAsync();
```

```cs
public Task<AuthorizationPolicy?> GetFallbackPolicyAsync()
```

```cs
=> this.BackupPolicyProvider.GetFallbackPolicyAsync();
```

这就完成了 `MinimumAgePolicyProvider` 的实现。现在您可以在控制器或操作方法上使用授权策略。在以下代码中，您会注意到使用了两个策略，一个在控制器上使用 `Over14`，另一个在 `Index` 操作方法上使用 `Over18`：

```cs
[Authorize(Policy ="Over14")]
```

```cs
public class OrdersController : Controller
```

```cs
{
```

```cs
    [Authorize(Policy ="Over18")]
```

```cs
public IActionResult Index()
```

```cs
{
```

```cs
        return View();
```

```cs
}
```

```cs
}
```

年龄超过 14 岁的用户将能够访问 `OrdersController` 下的任何操作方法，而年龄超过 18 岁的用户只能访问 `Index` 操作。

在下一节中，我们将学习如何创建和使用自定义授权属性。

## 自定义授权属性

在前面的示例中，一个包含年龄的策略名称作为字符串传递，但这种方式代码不够整洁。如果能够将 `age` 作为参数传递给授权属性会更好。为此，您需要创建一个继承自 `AuthorizeAttribute` 类的自定义授权属性。

在以下示例代码中，`AuthorizeAgeOverAttribute` 类继承自 `AuthorizeAttribute` 类。该类的构造函数接受 `age` 作为输入。在设置器中，我们通过连接 `PREFIX` 和 `Age` 来构造并设置一个策略名称：

```cs
public class AuthorizeAgeOverAttribute : AuthorizeAttribute
```

```cs
{
```

```cs
const string PREFIX = "Over";
```

```cs
public AuthorizeAgeOverAttribute(int age) => Age = age;
```

```cs
public int Age
```

```cs
{
```

```cs
    get
```

```cs
            {
```

```cs
                if 
```

```cs
            (int.TryParse(Policy.Substring(PREFIX.Length), 
```

```cs
            out var age))
```

```cs
                {
```

```cs
                    return age;
```

```cs
                }
```

```cs
                return default(int);
```

```cs
            }
```

```cs
            set
```

```cs
            {
```

```cs
                Policy = $"{PREFIX}{value.ToString()}";
```

```cs
            }
```

```cs
        }
```

```cs
}
```

要使用 `AuthorizeAgeOver` 属性，我们必须在 `Program.cs` 中注册 `AuthorizationHandler` 和 `AuthorizationPolicyProvider` 服务。在以下代码中，`MinimumAgeAuthorizationHandler` 和 `MinimumAgePolicyProvider` 类型分别注册为 `Singleton` 用于 `IAuthorizationHandler` 和 `IauthorizationPolicyProvider`：

```cs
builder.Services.AddSingleton<IAuthorizationHandler,
```

```cs
        MinimumAgeAuthorizationHandler>();
```

```cs
builder.Services.AddSingleton<IAuthorizationPolicyProvider,
```

```cs
        MinimumAgePolicyProvider>();
```

现在自定义属性实现完成后，我们可以在控制器或操作方法上使用它。在以下示例中，您可以看到一个示例实现，其中年龄作为参数传递给我们的自定义授权属性 `AuthorizeAgeOver`：

```cs
[AuthorizeAgeOver(14)]
```

```cs
public class OrdersController : Controller
```

```cs
{
```

```cs
[AuthorizeAgeOver(18)]
```

```cs
public IActionResult Index()
```

```cs
{
```

```cs
        return View();
```

```cs
}
```

```cs
}
```

在下一节中，我们将学习如何在 Azure AD 应用程序中配置角色并使用基于角色的身份验证。

# 客户端和服务器应用程序的授权

在前面的章节中，我们学习了如何使用**Azure Active Directory**（**AAD**）作为身份服务来验证用户，但为了使用基于角色的授权，我们需要在 Azure AD 中进行一些配置更改。在本节中，我们将了解如何在 Azure AD 应用程序中启用和创建自定义角色，并在我们的电子商务应用程序中授权用户。

当用户登录到应用程序时，Azure AD 会将分配的角色和声明添加到用户的身份中。

前提条件

您应该已经设置了 Azure AD 和 AD 应用程序。如果没有，您可以参考*第十二章*的*Azure Active Directory 简介*部分，了解*身份验证*以进行设置。

让我们看看在 Azure AD 应用程序中启用角色需要执行的步骤：

1.  在 Azure 门户中，导航到您的**Active Directory**租户。

1.  在左侧菜单下，在**管理**中，选择**应用注册**：

![图 13.3 – Azure AD 应用程序](img/Figure_13.3_B18507.jpg)

图 13.3 – Azure AD 应用程序

1.  在**应用注册**页面搜索并选择您的 AD 应用程序。请参考以下截图：

![图 13.4 – Azure AD 应用程序](img/Figure_13.4_B18507.jpg)

图 13.4 – Azure AD 应用程序

1.  点击左侧菜单中的**清单**进行编辑，如图中所示。

![图 13.5 – 编辑清单](img/Figure_13.5_B18507.jpg)

图 13.5 – 编辑清单

1.  定位`appRoles`以配置多个角色。请参考以下代码以添加角色：

    ```cs
    {
    "allowedMemberTypes": [
            "User"
        ],
        "description": "Admin Users",
        "displayName": "Admin",
        "id": "6ef9b400-0219-463c-a542-5f4693c4e286",
        "isEnabled": true,
        "lang": null,
        "origin": "Application",
        "value": "Admin"
    }
    ```

您需要为`displayName`、`value`、`description`和`id`提供值。`id`的值是`Guid`，并且对于您添加的每个角色都必须是唯一的。同样，对于`value`，您需要提供在代码中引用的角色名称，并且它应该是唯一的。

1.  保存清单以完成它。

保存包含所需详细信息的清单将启用 Azure AD 应用程序中的自定义角色。在下一节中，我们将学习如何将这些自定义角色分配给用户。

## 为用户分配角色

下一步是为用户分配角色。用户角色的分配可以通过 Azure 门户或使用 Graph API 编程方式完成。在本节中，我们将使用 Azure 门户来分配角色，同样也可以使用 Graph API 实现。更多信息，请参阅[`docs.microsoft.com/en-us/graph/azuread-identity-access-management-concept-overview`](https://docs.microsoft.com/en-us/graph/azuread-identity-access-management-concept-overview)：

1.  在 Azure 门户中，导航到**Azure Active Directory**租户。

1.  点击左侧菜单中的**企业应用程序**并搜索并选择您的 AD 应用程序。

1.  前往**管理** | **用户和组** | **添加用户**。

1.  搜索并选择用户，然后点击**确定**。

1.  点击**选择角色**以选择您想要分配的角色。

1.  点击**分配**以保存选择。

您可以继续这些步骤将角色分配给多个用户。

要保护控制器或操作，您可以在角色一起添加 `Authorize` 属性。在以下代码中，`Admin` 控制器仅对具有 `Admin` 角色的用户可访问：

```cs
[Authorize(Roles ="Admin")]
```

```cs
public class AdminController : Controller
```

```cs
{
```

```cs
public IActionResult Index()
```

```cs
{
```

```cs
    return View();
```

```cs
}
```

```cs
}
```

到目前为止，我们学习了如何在 Azure AD 中启用角色并使用基于角色的模型进行授权。在下一节中，我们将看到如何使用视图中的用户身份访问角色和声明。

## 视图中的用户身份

用户声明原则可用于视图，根据需要有条件地显示或隐藏数据。例如，以下代码检查用户身份的 `IsAuthenticated` 属性以确定用户是否已认证。如果用户未认证，则显示 `Sign in` 链接；否则，显示带有 `Sign out` 链接的用户名：

```cs
<ul class="navbar-nav">
```

```cs
    @if (User.Identity.IsAuthenticated)
```

```cs
    {
```

```cs
        //// HTML code goes here
```

```cs
    }
```

```cs
    else
```

```cs
    {
```

```cs
        ////
```

```cs
    }
```

```cs
</ul>
```

类似地，我们可以使用 `IsInRole` 或 `HasClaim` 并编写我们的逻辑来向用户显示内容或从用户隐藏内容：

```cs
@if (User.HasClaim(x => x.Type == "PremiumUser"))
```

```cs
{
```

```cs
    <h1>Yay, you are Premium User!!!, @User.FindFirst(x => x.Type == ClaimTypes.Country)?.Value</h1>
```

```cs
}
```

更多详细信息，请参阅[`docs.microsoft.com/en-us/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps`](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps)。

# 摘要

在本章中，我们学习了什么是授权以及使用 ASP.NET Core 框架的不同实现方式。我们学习了如何使用简单、声明性的基于角色和声明的模型限制或匿名允许用户访问资源，我们还学习了如何使用丰富的基于策略的授权模型实现自定义逻辑以授权用户请求。

我们学习了如何使用授权策略提供程序动态添加授权策略，构建自定义授权属性。我们还学习了如何在 Azure AD 中配置自定义角色并在 ASP.NET Core 应用程序中使用它们。根据您的授权需求，您可以使用一个或多个授权模型来保护您的应用程序。

在下一章中，我们将学习如何监控 ASP.NET Core 应用程序的健康状况和性能。

# 问题

阅读本章后，您应该能够回答以下问题：

1.  以下哪个是确定授权是否成功的主要服务？

a. `IAuthorizationHandler`

b. `IAuthorizationRequirement`

c. `IAuthorizationService`

d. `IAuthorizationPolicyProvider`

**答案：c**

1.  在以下代码中，对 `Support` 动作的访问仅限于 `Support` 角色：

    ```cs
    [AllowAnonymous]
    public class HomeController : Controller
    {
          public IactionResult Index()
    {
        return View();
    }
    [Authorize(Roles ="Support")]
    public IactionResult Support()
    {
            return View();
    }
    }
    ```

a. 正确

b. 错误

**答案：b**

# 进一步阅读

要了解更多关于授权的信息，您可以参考[`docs.microsoft.com/en-us/aspnet/core/security/authorization/introduction?view=aspnetcore-6.0`](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/introduction?view=aspnetcore-6.0)。
