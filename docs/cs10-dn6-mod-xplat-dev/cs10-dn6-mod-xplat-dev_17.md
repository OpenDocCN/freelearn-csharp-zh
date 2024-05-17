# 第十七章：使用 Blazor 构建用户界面

本章介绍了使用 Blazor 构建用户界面。我将描述 Blazor 的不同版本及其优缺点。

您将学习如何构建可以在 Web 服务器上或 Web 浏览器中执行其代码的 Blazor 组件。当使用 Blazor Server 托管时，它使用 SignalR 将所需的更新传达到浏览器中的用户界面。当使用 Blazor WebAssembly 托管时，组件在客户端中执行其代码，并必须进行 HTTP 调用以与服务器交互。

在本章中，我们将涵盖以下主题：

+   理解 Blazor

+   比较 Blazor 项目模板

+   使用 Blazor Server 构建组件

+   为 Blazor 组件抽象服务

+   使用 Blazor WebAssembly 构建组件

+   改进 Blazor WebAssembly 应用程序

# 理解 Blazor

Blazor 允许您使用 C#而不是 JavaScript 构建共享组件和交互式 Web 用户界面。2019 年 4 月，微软宣布 Blazor“不再是实验性的，我们承诺将其作为受支持的 Web UI 框架进行发布，包括在 WebAssembly 上在浏览器中运行客户端的支持。” Blazor 在所有现代浏览器上都受支持。

## JavaScript 和朋友

传统上，需要在 Web 浏览器中执行的任何代码都是使用 JavaScript 编程语言编写的，或者使用**转译**（转换或编译）为 JavaScript 的更高级技术。这是因为所有浏览器在大约二十年的时间里都支持 JavaScript，因此它已成为在客户端实现业务逻辑的最低公共分母。

然而，JavaScript 确实存在一些问题。尽管它与 C#和 Java 等 C 风格语言有表面上的相似之处，但一旦深入挖掘，它实际上是非常不同的。它是一种动态类型的伪函数式语言，使用原型而不是类继承来实现对象重用。它可能看起来像人类，但当它被揭示为斯克鲁尔时，你会感到惊讶。

如果我们能在 Web 浏览器中使用与服务器端相同的语言和库，那不是很好吗？

## Silverlight - 使用插件的 C#和.NET

微软曾尝试使用一种名为 Silverlight 的技术来实现这一目标。当 Silverlight 2.0 于 2008 年发布时，C#和.NET 开发人员可以利用他们的技能构建在 Web 浏览器中由 Silverlight 插件执行的库和可视组件。

到 2011 年和 Silverlight 5.0，苹果在 iPhone 上取得了成功，史蒂夫·乔布斯对 Flash 等浏览器插件的厌恶最终导致微软放弃了 Silverlight，因为与 Flash 一样，Silverlight 在 iPhone 和 iPad 上被禁止使用。

## WebAssembly - Blazor 的目标

浏览器的最新发展为微软提供了另一次尝试的机会。2017 年，**WebAssembly 共识**完成，所有主要浏览器现在都支持它：Chromium（Chrome，Edge，Opera，Brave），Firefox 和 WebKit（Safari）。Blazor 不受微软的 Internet Explorer 支持，因为它是一个传统的 Web 浏览器。

**WebAssembly**（**Wasm**）是用于虚拟机的二进制指令格式，它提供了一种在 Web 上以接近本机速度运行多种语言编写的代码的方法。Wasm 被设计为高级语言（如 C#）编译的可移植目标。

## 理解 Blazor 托管模型

Blazor 是一个单一的编程或应用模型，具有多个托管模型：

+   **Blazor Server**在服务器端运行，因此您编写的 C#代码可以完全访问业务逻辑可能需要的所有资源，而无需进行身份验证。然后，它使用 SignalR 将用户界面更新传达到客户端。

+   服务器必须保持与每个客户端的实时 SignalR 连接，并跟踪每个客户端的当前状态，因此如果需要支持大量客户端，Blazor Server 的扩展性就不太好。它首次作为 ASP.NET Core 3.0 的一部分于 2019 年 9 月发布，并包含在.NET 5.0 及以后的版本中。

+   **Blazor WebAssembly** 在客户端运行，因此您编写的 C# 代码只能访问浏览器中的资源，并且必须在访问服务器上的资源之前进行 HTTP 调用（可能需要身份验证）。它最初是作为 ASP.NET Core 3.1 的扩展于 2020 年 5 月发布的，并且被版本化为 3.2，因为它是一个当前版本，因此不受 ASP.NET Core 3.1 的长期支持的覆盖。Blazor WebAssembly 3.2 版本使用了 Mono 运行时和 Mono 库；.NET 5 及更高版本使用了 Mono 运行时和 .NET 5 库。"*Blazor WebAssembly 在 .NET IL 解释器上运行，没有任何 JIT，因此不会赢得任何速度比赛。不过，在 .NET 5 中我们已经做了一些重大的速度改进，我们期望在 .NET 6 中进一步改进。*"—Daniel Roth

+   **.NET MAUI Blazor App**，又名 **Blazor Hybrid**，在 .NET 进程中运行，使用本地互操作通道将其 Web UI 渲染到 Web 视图控件，并托管在 .NET MAUI 应用程序中。这在概念上类似于使用 Node.js 的 Electron 应用程序。我们将在在线章节 *第十九章* *使用 .NET MAUI 构建移动和桌面应用程序* 中看到这种托管模型（可在 [`github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf`](https://github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf) 上找到）。

这种多主机模型意味着，通过仔细规划，开发人员可以编写一次 Blazor 组件，然后在 Web 服务器端、Web 客户端端或桌面应用程序中运行它们。

尽管 Blazor Server 支持 Internet Explorer 11，但 Blazor WebAssembly 不支持。

Blazor WebAssembly 可选支持 **渐进式 Web 应用程序**（**PWA**），这意味着网站访问者可以使用浏览器菜单将应用程序添加到其桌面并在离线状态下运行应用程序。

## 理解 Blazor 组件

重要的是要理解 Blazor 用于创建 **用户界面组件**。组件定义了如何渲染用户界面，如何响应用户事件，并且可以被组合和嵌套，并编译成 NuGet Razor 类库以进行打包和分发。

例如，您可以创建一个名为 `Rating.razor` 的组件，如下面的标记所示：

```cs

<div

@for (int

i = 0

; i < Maximum; i++)

{

如果

（i < 值）

{

<span

类

="oi oi-star-filled"

/>

}

否则

{

<span

类

="oi oi-star-empty"

/>

}

}

</div

@code {

[Parameter

]

公共

字节

最大 { get

; set

; }

[Parameter

]

公共

字节

值 { get

; set

; }

}

```

代码可以存储在名为 `Rating.razor.cs` 的单独的代码后台文件中，而不是在一个包含标记和 `@code` 块的单个文件中。此文件中的类必须是 `partial`，并且与组件的名称相同。

然后，您可以在网页上使用组件，如下面的标记所示：

```cs

<

h1

回顾</

h1

<

评分

id

=

"rating"

最大

=

"5"

值

=

"3"

/>

<

textarea

id

=

"comment"

/>

```

有许多内置的 Blazor 组件，包括用于在网页的 `<head>` 部分中设置 `<title>` 等元素的组件，以及许多第三方公司出售用于常见目的的组件。

将来，Blazor 可能不仅限于使用 Web 技术创建用户界面组件。微软有一项名为 **Blazor Mobile Bindings** 的实验性技术，允许开发人员使用 Blazor 构建移动用户界面组件。它不使用 HTML 和 CSS 来构建 Web 用户界面，而是使用 XAML 和 .NET MAUI 来构建跨平台图形用户界面。

## Blazor 和 Razor 有什么区别？

您可能想知道为什么 Blazor 组件使用 `.razor` 作为文件扩展名。Razor 是一种模板标记语法，允许混合 HTML 和 C#。支持 Razor 语法的旧技术使用 `.cshtml` 文件扩展名来指示 C# 和 HTML 的混合。

Razor 语法用于：

+   ASP.NET Core MVC **视图** 和 **部分视图** 使用 `.cshtml` 文件扩展名。业务逻辑分离到一个控制器类中，该类将视图视为一个模板，然后将视图模型推送到该模板，然后将其输出到一个网页。

+   **Razor Pages** 使用 `.cshtml` 文件扩展名。业务逻辑可以嵌入或分离到使用 `.cshtml.cs` 文件扩展名的文件中。输出是一个网页。

+   **Blazor 组件** 使用 `.razor` 文件扩展名。输出不是网页，尽管可以使用布局将组件包装起来，使其输出为网页，并且可以使用 `@page` 指令来分配一个路由，定义检索组件作为页面的 URL 路径。

# 比较 Blazor 项目模板

了解选择 Blazor Server 和 Blazor WebAssembly 托管模型之间的选择的一种方法是查看它们默认项目模板中的差异。

## 审查 Blazor Server 项目模板

让我们看看 Blazor Server 项目的默认模板。大多数情况下，您会发现它与 ASP.NET Core Razor Pages 模板相同，只是增加了一些关键内容：

1.  使用您喜欢的代码编辑器添加一个新项目，如下列表所示：

1.  项目模板：**Blazor Server App** / `blazorserver`

1.  工作区/解决方案文件和文件夹：`PracticalApps`

1.  项目文件和文件夹：`Northwind.BlazorServer`

1.  其他 Visual Studio 选项：**身份验证类型**：**无**；**配置 HTTPS**：已选中；**启用 Docker**：已清除

1.  在 Visual Studio Code 中，选择 `Northwind.BlazorServer` 作为活动的 OmniSharp 项目。

1.  构建 `Northwind.BlazorServer` 项目。

1.  在 `Northwind.BlazorServer` 项目/文件夹中，打开 `Northwind.BlazorServer.csproj`，注意它与使用 Web SDK 并针对 .NET 6.0 的 ASP.NET Core 项目相同。

1.  打开 `Program.cs`，注意它几乎与 ASP.NET Core 项目相同。不同之处包括配置服务的部分，其中调用 `AddServerSideBlazor` 方法，如下面的代码中所示：

```cs

builder.Services.AddRazorPages();

**builder.Services.AddServerSideBlazor();**

builder.Services.AddSingleton<WeatherForecastService>();

```

1.  还要注意配置 HTTP 管道的部分，其中添加了对 `MapBlazorHub` 和 `MapFallbackToPage` 方法的调用，这些方法配置 ASP.NET Core 应用程序接受 Blazor 组件的传入 SignalR 连接，而其他请求则回退到名为 `_Host.cshtml` 的 Razor Page，如下面的代码所示：

```cs

app.UseRouting();

**app.MapBlazorHub();**

**app.MapFallbackToPage(**

**"/_Host"**

**);**

app.Run();

```

1.  在 `Pages` 文件夹中，打开 `_Host.cshtml`，注意它设置了一个名为 `_Layout` 的共享布局，并呈现了一个类型为 `App` 的 Blazor 组件，该组件在服务器上进行了预呈现，如下面的标记所示：

```cs

@page "/"

@namespace  Northwind.BlazorServer.Pages

@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

@{

Layout = "_Layout"

;

}

<

component

type

=

"typeof(App)"

render-mode

=

"ServerPrerendered"

/>

```

1.  在 `Pages` 文件夹中，打开名为 `_Layout.cshtml` 的共享布局文件，如下面的标记所示：

```cs

@using Microsoft.AspNetCore.Components.Web

@namespace Northwind.BlazorServer.Pages

@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

<!DOCTYPE

html

<

html

lang

=

"en"

<

head

<

meta

charset

=

"utf-8"

/>

<

meta

name

=

"viewport"

content

=

"width=device-width, initial-scale=1.0"

/>

<

base

href

=

"~/"

/>

<

link

rel

=

"stylesheet"

href

=

"css/bootstrap/bootstrap.min.css"

/>

<

link

href

=

"css/site.css"

rel

=

"stylesheet"

/>

<

link

href

=

"Northwind.BlazorServer.styles.css"

rel

=

"stylesheet"

/>

<

component

type

=

"typeof(HeadOutlet)"

render-mode

=

"ServerPrerendered"

/>

</

head

<

body

@RenderBody()

<

div

id

=

"blazor-error-ui"

<

environment

include

=

"Staging,Production"

发生了错误。该应用程序可能不再响应，直到重新加载。

</

environment

<

environment

include

=

"Development"

发生了一个未处理的异常。请查看浏览器开发工具以获取详细信息。

</

environment

<

a

href

=

""

class

=

"reload"

Reload</

a

<

a

class

=

"dismiss"

![](img/Image00149.jpg)

</

a

</

div

<

script

src

=

"_framework/blazor.server.js"

></

script

</

body

</

html

```

While reviewing the preceding markup, note the following:

+   `<div id="blazor-error-ui">`用于显示 Blazor 错误，当发生错误时，它将显示为网页底部的黄色条

+   `blazor.server.js`的脚本块管理到服务器的 SignalR 连接

1.  在`Northwind.BlazorServer`文件夹中，打开`App.razor`，注意它为当前程序集中找到的所有组件定义了一个`Router`，如下面的代码所示：

```cs

<

Router

AppAssembly

=

"@typeof(App).Assembly"

<

找到

Context

=

"routeData"

<

RouteView

RouteData

=

"@routeData"

DefaultLayout

=

"@typeof(MainLayout)"

/>

<

FocusOnNavigate

RouteData

=

"@routeData"

Selector

=

"h1"

/>

</

Found

<

NotFound

<

PageTitle

Not found</

PageTitle

<

LayoutView

Layout

=

"@typeof(MainLayout)"

<

p

抱歉，这个地址上没有任何内容。</

p

</

LayoutView

</

NotFound

</

Router

```

在审查前面的标记时，请注意以下内容：

+   如果找到匹配的路由，则执行`RouteView`，将组件的默认布局设置为`MainLayout`，并将任何路由数据参数传递给组件。

+   如果找不到匹配的路由，则执行`LayoutView`，渲染`MainLayout`中的内部标记（在本例中，是一个简单的段落元素，其中包含一条消息，告诉访问者在这个地址上没有任何内容）。

1.  在`Shared`文件夹中，打开`MainLayout.razor`，注意它定义了一个包含由此项目中的`NavMenu.razor`组件文件实现的导航菜单的侧边栏的`<div>`，以及用于内容的 HTML5 元素，如`<main>`和`<article>`，如下面的代码所示：

```cs

@inherits LayoutComponentBase

<

PageTitle

Northwind.BlazorServer</

PageTitle

<

div

class

=

"page"

<

div

class

=

"sidebar"

<

NavMenu

/>

</

div

<

main

<

div

class

=

"top-row px-4"

<

a

href

=

"https://docs.microsoft.com/aspnet/"

target

=

"_blank"

About</

a

</

div

<

article

class

=

"content px-4"

@Body

</

article

</

main

</

div

```

1.  在`Shared`文件夹中，打开`MainLayout.razor.css`，注意它包含了组件的隔离 CSS 样式。

1.  在`Shared`文件夹中，打开`NavMenu.razor`，注意它有三个菜单项**Home**，**Counter**和**Fetch data**。这些是使用名为`NavLink`的由 Microsoft 提供的 Blazor 组件创建的，如下面的标记所示：

```cs

<

div

class

=

"top-row ps-3 navbar navbar-dark"

<

div

class

=

"container-fluid"

<

a

class

=

"navbar-brand"

href

=

""

Northwind.BlazorServer</

a

<

button

title

=

"Navigation menu"

class

=

"navbar-toggler"

@

onclick

=

"ToggleNavMenu"

<

span

class

=

"navbar-toggler-icon"

></

span

</

button

</

div

</

div

<

div

class

=

"@NavMenuCssClass"

@

onclick

=

"ToggleNavMenu"

<

nav

class

=

"flex-column"

<

div

class

=

"nav-item px-3"

<

NavLink

class

=

"nav-link"

href

=

""

Match

=

"NavLinkMatch.All"

<

span

class

=

"oi oi-home"

aria-hidden

=

"true"

></

span

Home

</

NavLink

</

div

<

div

class

=

"nav-item px-3"

<

NavLink

class

=

"nav-link"

href

=

"counter"

<

span

class

=

"oi oi-plus"

aria-hidden

=

"true"

></

span

Counter

</

NavLink

</

div

<

div

class

=

"nav-item px-3"

<

NavLink

class

=

"nav-link"

href

=

"fetchdata"

<

span

class

=

"oi oi-list-rich"

aria-hidden

=

"true"

></

span

Fetch data

</

NavLink

</

div

</

nav

</

div

@code {

private

bool

collapseNavMenu = true

;

private

string

? NavMenuCssClass => collapseNavMenu ? "collapse"

: null

;

private

void

ToggleNavMenu

()

{

collapseNavMenu = !collapseNavMenu;

}

}

```

1.  在 `Pages` 文件夹中，打开 `FetchData.razor` 并注意它定义了一个从注入的天气服务中获取天气预报并在表中呈现它们的组件，如下面的代码所示：

```cs

@page "/fetchdata"

<

页面标题

天气预报</

页面标题

@using Northwind.BlazorServer.Data

@inject WeatherForecastService ForecastService

<

h1

天气预报</

h1

<

p

此组件演示了从服务获取数据。

p

@if (forecasts == null)

{

<

p

><

em

加载中...</

em

></

p

}

否则

{

<

表

类

=

"表"

<

表头

<

行

<

表头

日期</

表头

<

表头

温度（C）</

表头

<

表头

温度（F）</

表头

<

表头

摘要</

表头

</

行

</

表头

<

tbody

@foreach (var forecast in forecasts)

{

<

行

<

单元格

@forecast.Date.ToShortDateString()</

单元格

<

单元格

@forecast.TemperatureC</

单元格

<

单元格

@forecast.TemperatureF</

单元格

<

单元格

@forecast.Summary</

单元格

</

行

}

</

tbody

</

表

}

@code {

私有的

WeatherForecast[]? forecasts;

受保护的

覆盖

异步的

Task OnInitializedAsync

()

{

forecasts = await

ForecastService.GetForecastAsync(DateTime.Now);

}

}

```

1.  在 `Data` 文件夹中，打开 `WeatherForecastService.cs` 并注意它*不是*一个 Web API 控制器类；它只是一个返回随机天气数据的普通类，如下面的代码所示：

```cs

命名空间

Northwind.BlazorServer.Data

{

公共的

类

WeatherForecastService

{

私有的

静态的

只读的

字符串

[] Summaries = 新的

[]

{

"冰冷的"

, "清凉的"

, "寒冷的"

, "凉爽的"

, "温和的"

, "温暖的"

,

"温暖的"

, "炎热的"

, "酷热的"

, "酷热的"

};

公共的

Task<WeatherForecast[]> GetForecastAsync(DateTime startDate)

{

返回

Task.FromResult(Enumerable.Range(1

, 5

)

选择(index => 新的

WeatherForecast

{

日期 = startDate.AddDays(index),

温度 C = Random.Shared.Next(-20

, 55

),

摘要 = Summaries[Random.Shared.Next(Summaries.Length)]

}).ToArray());

}

}

}

```

### 了解 CSS 和 JavaScript 隔离

Blazor 组件经常需要提供自己的 CSS 来应用样式或 JavaScript 用于无法纯粹在 C# 中执行的活动，比如访问浏览器 API。为了确保这不会与站点级别的 CSS 和 JavaScript 冲突，Blazor 支持 CSS 和 JavaScript 隔离。如果有一个名为 `Index.razor` 的组件，只需创建一个名为 `Index.razor.css` 的 CSS 文件。在此文件中定义的样式将覆盖项目中的任何其他样式。

## 了解 Blazor 路由到页面组件

我们在 `App.razor` 文件中看到的 `Router` 组件使路由到组件成为可能。创建组件实例的标记看起来像 HTML 标记，其中标记的名称是组件类型。组件可以使用元素嵌入到网页中，例如，`<Rating Stars="5" />`，也可以像 Razor Page 或 MVC 控制器一样路由到。

### 如何定义可路由的页面组件

要创建可路由的页面组件，请在组件的 `.razor` 文件顶部添加 `@page` 指令，如下面的标记所示：

```cs

@page "customers"

```

前面的代码相当于一个带有 `[Route]` 属性的 MVC 控制器，如下面的代码所示：

```cs

[Route(

"顾客"

)

]

公共的

类

CustomersController

{

```

`Router` 组件扫描特定的程序集，其 `AppAssembly` 参数用于注册带有 `[Route]` 属性的组件的 URL 路径。

任何单页组件都可以有多个 `@page` 指令来注册多个路由。

在运行时，页面组件与您指定的任何特定布局合并，就像 MVC 视图或 Razor Page 一样。默认情况下，Blazor Server 项目模板将 `MainLayout.razor` 定义为页面组件的布局。

**良好实践**：按照惯例，将可路由的页面组件放在 `Pages` 文件夹中。

### 如何导航 Blazor 路由

Microsoft 提供了一个名为 `NavigationManager` 的依赖服务，它了解 Blazor 路由和 `NavLink` 组件。

`NavigateTo`方法用于转到指定的 URL。

### 如何传递路由参数

Blazor 路由可以包括不区分大小写的命名参数，您的代码可以通过将参数绑定到代码块中的属性来最容易地访问传递的值，使用`[Parameter]`属性，如下面的标记所示：

```cs

@page "/customers/{country}"

<div

>作为值的国家参数：@Country</div

@code {

[Parameter

]

public

字符串

Country { get

; 设置

; }

}

```

处理应该具有默认值的参数的推荐方法是在`OnParametersSet`方法中使用`?`后缀参数，并使用空值合并运算符，如下面的标记所示：

```cs

@page "/customers/{country?}"

<div

>作为值的国家参数：@Country</div

@code {

[Parameter

]

public

字符串

Country { get

; 设置

; }

protected

重写

void

OnParametersSet

()

{

// 如果自动设置的属性为空

// 将其值设置为美国

Country = Country ?? "USA"

;

}

}

```

### 理解基本组件类

`OnParametersSet`方法是由默认继承的基类`ComponentBase`定义的，如下面的代码所示：

```cs

using

Microsoft.AspNetCore.Components;

public

抽象

类

ComponentBase

: IComponent

, IHandleAfterRender

, IHandleEvent

{

// 未显示成员

}

```

`ComponentBase`有一些有用的方法，您可以调用和重写，如下表所示：

| 方法 | 描述 |
| --- | --- |
| `InvokeAsync` | 调用此方法以在关联的渲染器同步上下文中执行函数。 |
| `OnAfterRender`，`OnAfterRenderAsync` | 重写这些方法以在每次组件呈现后调用代码。 |
| `OnInitialized`，`OnInitializedAsync` | 重写这些方法以在组件从呈现树中的父级接收到其初始参数后调用代码。 |
| `OnParametersSet`，`OnParametersSetAsync` | 重写这些方法以在组件接收到参数并且值已分配给属性后调用代码。 |
| `ShouldRender` | 重写此方法以指示组件是否应呈现。 |
| `StateHasChanged` | 调用此方法以导致组件重新呈现。 |

Blazor 组件可以以类似于 MVC 视图和 Razor 页面的方式具有共享布局。

创建一个`.razor`组件文件，但要明确地继承自`LayoutComponentBase`，如下面的标记所示：

```cs

@inherits LayoutComponentBase

<

div

...

@Body

...

</

div

```

基类有一个名为`Body`的属性，您可以在标记中呈现该属性，以便在布局中的正确位置呈现。

在`App.razor`文件及其`Router`组件中为组件设置默认布局。要为组件明确设置布局，请使用`@layout`指令，如下面的标记所示：

```cs

@page "/customers"

@layout AlternativeLayout

<

div

...

</

div

```

### 如何使用导航链接组件与路由

在 HTML 中，您使用`<a>`元素来定义导航链接，如下面的标记所示：

```cs

<a

href="/customers"

>客户</a

```

在 Blazor 中，使用`<NavLink>`组件，如下面的标记所示：

```cs

<

NavLink

href

=

"/customers"

Customers</

NavLink

```

`NavLink`组件比锚元素更好，因为如果其`href`与当前位置 URL 匹配，它会自动将其类设置为`active`。如果您的 CSS 使用不同的类名，则可以在`NavLink.ActiveClass`属性中设置类名。

默认情况下，在匹配算法中，`href`是路径*前缀*，因此如果`NavLink`的`href`为`/customers`，如前面的代码示例所示，那么它将匹配所有以下路径并将它们全部设置为`active`类样式：

```cs

/客户

/客户/美国

/客户/德国/柏林

```

为了确保匹配算法只对*所有*路径执行匹配，将 `Match` 参数设置为 `NavLinkMatch.All`，如下面的代码所示：

```cs

<

NavLink

href

=

"/customers"

匹配

=

"NavLinkMatch.All"

Customers</

NavLink

```

如果设置了其他属性，例如 `target`，它们将传递给生成的底层 `<a>` 元素。

## 运行 Blazor Server 项目模板

现在我们已经审查了项目模板和与 Blazor Server 特定的重要部分，我们可以启动网站并审查其行为：

1.  在 `Properties` 文件夹中，打开 `launchSettings.json`。

1.  修改 `applicationUrl` 以使用端口 `5000` 用于 `HTTP` 和端口 `5001` 用于 `HTTPS`，如下面的标记所示：

```cs

"profiles"

: {

"Northwind.BlazorServer"

: {

"commandName"

: "Project"

,

"dotnetRunMessages"

: true

,

"launchBrowser"

: true

,

**"applicationUrl"**

**:**

**"https://localhost:5001;http://localhost:5000"**

**,**

"environmentVariables"

: {

"ASPNETCORE_ENVIRONMENT"

: "Development"

}

},

```

1.  启动网站。

1.  启动 Chrome。

1.  导航到 `https://localhost:5001/` 。

1.  在左侧导航菜单中，单击 **Fetch data**，如 *图 17.1* 所示：![](img/Image00150.jpg)

图 17.1：将天气数据获取到 Blazor Server 应用程序

1.  在浏览器地址栏中，将路由更改为 `/apples` 并注意缺少的消息，如 *图 17.2* 所示：![](img/Image00151.jpg)

图 17.2：缺少组件消息

1.  关闭 Chrome 并关闭 Web 服务器。

## 审查 Blazor WebAssembly 项目模板

现在我们将创建一个 Blazor WebAssembly 项目。如果代码与 Blazor Server 项目中的代码相同，我将不在书中显示代码：

1.  使用您喜欢的代码编辑器向 `PracticalApps` 解决方案或工作区中添加一个新项目，如下列表所定义的：

1.  项目模板：**Blazor WebAssembly App** / `blazorwasm`

1.  Switches: `--pwa --hosted`

1.  工作区/解决方案文件和文件夹：`PracticalApps`

1.  项目文件和文件夹：`Northwind.BlazorWasm`

1.  **身份验证类型**：**无**

1.  **配置为 HTTPS**：checked

1.  **ASP.NET Core hosted** : checked

1.  **渐进式 Web 应用程序** : checked

在审查生成的文件夹和文件时，请注意生成了三个项目，如下列表所述：

+   `Northwind.BlazorWasm.Client` 是 `Northwind.BlazorWasm\Client` 文件夹中的 Blazor WebAssembly 项目。

+   `Northwind.BlazorWasm.Server` 是 `Northwind.BlazorWasm\Server` 文件夹中的 ASP.NET Core 项目网站，用于托管天气服务，其实现方式与以前一样返回随机天气预报，但是实现为适当的 Web API 控制器类。项目文件具有对 `Shared` 和 `Client` 的项目引用，并具有支持服务器端 Blazor WebAssembly 的包引用。

+   `Northwind.BlazorWasm.Shared` 是 `Northwind.BlazorWasm\Shared` 文件夹中的类库，其中包含天气服务的模型。

文件夹结构简化，如 *图 17.3* 所示：

![](img/Image00152.jpg)

图 17.3：Blazor WebAssembly 项目模板的文件夹结构

有两种部署 Blazor WebAssembly 应用程序的方法。您可以通过将其发布文件放在任何静态托管 Web 服务器中来部署 `Client` 项目。它可以配置为调用您在 *第十六章*，*构建和使用 Web 服务* 中创建的天气服务，或者您可以部署引用 `Client` 应用程序并托管天气服务和 Blazor WebAssembly 应用程序的 `Server` 项目。该应用程序放置在服务器网站 `wwwroot` 文件夹中，以及任何其他静态资产。您可以在以下链接中阅读有关这些选择的更多信息：[`docs.microsoft.com/en-us/aspnet/core/blazor/host-and-deploy/webassembly`](https://docs.microsoft.com/en-us/aspnet/core/blazor/host-and-deploy/webassembly)

1.  在`Client`文件夹中，打开`Northwind.BlazorWasm.Client.csproj`并注意它使用 Blazor WebAssembly SDK 并引用两个 WebAssembly 包和`Shared`项目，以及 PWA 支持所需的服务工作者，如下面的标记所示：

```cs

<Project Sdk="Microsoft.NET.Sdk.BlazorWebAssembly"

<PropertyGroup>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<ImplicitUsings>enable</ImplicitUsings>

<ServiceWorkerAssetsManifest>service-worker-assets.js

</ServiceWorkerAssetsManifest>

</PropertyGroup>

<ItemGroup>

<PackageReference Include=

“Microsoft.AspNetCore.Components.WebAssembly”

Version="6.0.0"

/>

<PackageReference Include=

“Microsoft.AspNetCore.Components.WebAssembly.DevServer”

Version="6.0.0"

PrivateAssets="all"

/>

</ItemGroup>

<ItemGroup>

<ProjectReference Include=

“..\Shared\Northwind.BlazorWasm.Shared.csproj”

/>

</ItemGroup>

<ItemGroup>

<ServiceWorker Include="wwwroot\service-worker.js"

PublishedContent="wwwroot\service-worker.published.js"

/>

</ItemGroup>

</Project>

```

1.  在`Client`文件夹中，打开`Program.cs`并注意主机生成器是用于`WebAssembly`，而不是服务器端 ASP.NET Core，并且它注册了一个用于进行 HTTP 请求的依赖服务，这是 Blazor WebAssembly 应用程序的一个极其常见的要求，如下面的代码所示：

```cs

使用

Microsoft.AspNetCore.Components.Web;

使用

Microsoft.AspNetCore.Components.WebAssembly.Hosting;

使用

Northwind.BlazorWasm.Client;

var

builder = WebAssemblyHostBuilder.CreateDefault(args);

builder.RootComponents.Add<App>("#app"

);

builder.RootComponents.Add<HeadOutlet>("head::after"

);

builder.Services.AddScoped(sp => new

HttpClient

{ BaseAddress = new

Uri(builder.HostEnvironment.BaseAddress) });

等待

builder.Build().RunAsync();

```

1.  在`wwwroot`文件夹中，打开`index.html`并注意支持离线工作的`manifest.json`和`service-worker.js`文件，以及下载所有 NuGet 包的`blazor.webassembly.js`脚本，如下面的标记所示：

```cs

<!DOCTYPE

html

<

html

<

头

<

元

字符集

=

“utf-8”

/>

<

元

名称

=

“视口”

内容

=

“width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no”

/>

<

标题

Northwind.BlazorWasm</

标题

<

基础

href

=

“/”

/>

<

链接

href

=

“css/bootstrap/bootstrap.min.css”

关系

=

“样式表”

/>

<

链接

href

=

“css/app.css”

关系

=

“样式表”

/>

<

链接

href

=

“Northwind.BlazorWasm.Client.styles.css”

关系

=

“样式表”

/>

<

链接

href

=

“manifest.json”

关系

=

“清单”

/>

<

链接

关系

=

“apple-touch-icon”

尺寸

=

“512x512”

href

=

“图标-512.png”

/>

<

链接

关系

=

“apple-touch-icon”

尺寸

=

“192x192”

href

=

“图标-192.png”

/>

</

头

<

身体

<

div

标识

=

“应用”

加载中...</

div

<

div

id

=

“blazor-error-ui”

发生了一个未处理的错误。

<

a

href

=

""

类

=

“重新加载”

重新加载</

a

<

a

类

=

“解散”

！[](img/Image00149.jpg)

</

a

</

div

<

脚本

src

=

“_framework/blazor.webassembly.js”

></

脚本

<

脚本

navigator.serviceWorker.register('service-worker.js'

);</

脚本

</

身体

</

html

```

1.  请注意，以下`.razor`文件与 Blazor Server 项目中的文件相同：

+   `App.razor`

+   `Shared\MainLayout.razor`

+   `Shared\NavMenu.razor`

+   `Shared\SurveyPrompt.razor`

+   `Pages\Counter.razor`

+   `Pages\Index.razor`

1.  在“页面”文件夹中，打开`FetchData.razor`并注意标记与 Blazor Server 相似，除了用于进行 HTTP 请求的注入依赖服务，如下面的部分标记所示：

```cs

@page "/fetchdata"

@using Northwind.BlazorWasm.Shared

**@inject HttpClient Http**

<h1

>天气预报</h1

...

@code {

私人的

WeatherForecast[]? forecasts;

受保护的

覆盖

async

任务

OnInitializedAsync

（）

{

**预测=**

**await**

**Http.GetFromJsonAsync<WeatherForecast[]>(**

**“WeatherForecast”**

**);**

}

}

```

1.  启动`Northwind.BlazorWasm.Server`项目。

1.  注意应用程序具有与以前相同的功能。Blazor 组件代码在浏览器内执行，而不是在服务器上执行。天气服务正在 Web 服务器上运行。

1.  关闭 Chrome 并关闭 Web 服务器。

# 使用 Blazor Server 构建组件

在本节中，我们将构建一个组件，用于列出、创建和编辑 Northwind 数据库中的客户。我们将首先为 Blazor Server 原生构建它，然后重构为同时与 Blazor Server 和 Blazor WebAssembly 一起使用。

## 定义和测试一个简单的组件

我们将新组件添加到现有的 Blazor Server 项目中：

1.  在`Northwind.BlazorServer`项目（*不是*`Northwind.BlazorWasm.Server`项目）中，在`Pages`文件夹中，添加一个名为`Customers.razor`的新文件。在 Visual Studio 中，项目项名为**Razor Component**。

**良好实践**：组件文件名必须以大写字母开头，否则会出现编译错误！

1.  添加语句以输出`Customers`组件的标题，并定义一个代码块，其中定义一个属性来存储国家的名称，如下面的标记所示：

```cs

<h3

>Customers@(string

.IsNullOrWhiteSpace(Country) ? " 全球"

："在"

+ Country)</h3

@code {

[Parameter

]

公共

string

? Country { get

; 设置

; }

}

```

1.  在`Pages`文件夹中，在`Index.razor`组件中，添加语句到文件底部，实例化`Customers`组件两次，一次将`Germany`作为国家参数传递，一次不设置国家，如下面的标记所示：

```cs

<

顾客

国家

=

"德国"

/>

<

Customers

/>

```

1.  启动`Northwind.BlazorServer`网站项目。

1.  启动 Chrome。

1.  导航到`https://localhost:5001/`，注意`Customers`组件，如*图 17.4*所示：![](img/Image00153.jpg)

图 17.4：Country 参数设置为德国而不设置的 Customers 组件

1.  关闭 Chrome 并关闭 Web 服务器。

## 使组件成为可路由的页面组件

将此组件简单地转换为可路由的页面组件，带有国家的路由参数：

1.  在`Pages`文件夹中，在`Customers.razor`组件中，添加语句到文件顶部，将`/customers`注册为其路由，带有可选的国家路由参数，如下面的标记所示：

```cs

@page "/customers/{country?}"

```

1.  在`Shared`文件夹中，打开`NavMenu.razor`，并添加两个列表项元素，用于显示全球和德国客户的可路由页面组件，两者都使用人的图标，如下面的标记所示：

```cs

<

div

类

=

"nav-item px-3"

<

NavLink

类

=

"nav-link"

href

=

"customers"

匹配

=

"NavLinkMatch.All"

<

span

类

=

"oi oi-people"

aria-hidden

=

"true"

></

span

全球客户

</

NavLink

</

div

<

div

类

=

"nav-item px-3"

<

NavLink

类

=

"nav-link"

href

=

"customers/Germany"

<

span

class

=

"oi oi-people"

aria-hidden

=

"true"

></

span

德国的客户

</

NavLink

</

div

```

我们在顾客菜单项中使用了一个人的图标。您可以在以下链接查看其他可用的图标：[`iconify.design/icon-sets/oi/`](https://iconify.design/icon-sets/oi/)

1.  启动网站项目。

1.  启动 Chrome。

1.  导航到`https://localhost:5001/`。

1.  在左侧导航菜单中，单击**德国的客户**，注意国家名称正确传递到页面组件，并且组件使用与其他页面组件相同的共享布局，如`Index.razor`。

1.  关闭 Chrome 并关闭 Web 服务器。

## 将实体引入组件

现在您已经看到了组件的最小实现，我们可以为其添加一些有用的功能。在这种情况下，我们将使用 Northwind 数据库上下文从数据库中获取客户：

1.  在`Northwind.BlazorServer.csproj`中，添加对 Northwind 数据库上下文项目的引用，用于 SQL Server 或 SQLite，如下面的标记所示：

```cs

<ItemGroup>

<!-- 如果您喜欢，将 Sqlite 更改为 SqlServer -->

<ProjectReference Include="..\Northwind.Common.DataContext.Sqlite

\Northwind.Common.DataContext.Sqlite.csproj"

/>

</ItemGroup>

```

1.  构建`Northwind.BlazorServer`项目。

1.  在`Program.cs`中，导入用于处理 Northwind 数据库上下文的命名空间，如下面的代码所示：

```cs

using

Packt.Shared; // 添加 NorthwindContext 扩展方法

```

1.  在配置服务的部分，添加一个语句来注册 Northwind 数据库上下文到依赖服务集合中，如下面的代码所示：

```cs

builder.Services.AddNorthwindContext();

```

1.  打开`_Imports.razor`并导入用于处理 Northwind 实体的命名空间，以便我们构建的 Blazor 组件不需要单独导入命名空间，如下面的标记所示：

```cs

@using Packt.Shared  @* Northwind entities *@

```

`_Imports.razor`文件仅适用于`.razor`文件。如果您使用代码后端的`.cs`文件来实现组件代码，那么它们必须单独导入命名空间，或者使用全局导入来隐式导入命名空间。

1.  在`Pages`文件夹中，在`Customers.razor`中，添加语句来注入 Northwind 数据库上下文，然后使用它来输出所有客户的表格，如下面的代码所示：

```cs

@using Microsoft.EntityFrameworkCore  @* ToListAsync 扩展方法 *@

@page "/customers/{country?}"

@inject NorthwindContext db

<

h3

Customers @(string.IsNullOrWhiteSpace(Country)

? "全球" : "在" + Country)</

h3

@if (customers == null)

{

<

p

><

em

加载中...

em

></

p

}

else

{

<

table

class

=

"table"

<

thead

<

tr

<

th

Id</

th

<

th

公司名称</

th

<

th

地址</

th

<

th

电话</

th

<

th

></

th

</

tr

</

thead

<

tbody

@foreach (Customer c in customers)

{

<

tr

<

td

@c.CustomerId</

td

<

td

@c.CompanyName</

td

<

td

@c.Address<

br

/>

@c.City<

br

/>

@c.PostalCode<

br

/>

@c.Country

</

td

<

td

@c.Phone</

td

<

td

<

a

class

=

"btn btn-info"

href

=

编辑客户/@c.CustomerId

<

i

class

=

"oi oi-pencil"

></

i

></

a

<

a

class

=

"btn btn-danger"

href

=

"deletecustomer/@c.CustomerId"

<

i

class

=

"oi oi-trash"

></

i

></

a

</

td

</

tr

}

</

tbody

</

table

}

@code {

[Parameter

]

public

string

? Country { get

; set

; }

private

IEnumerable<

客户

? customers;

protected

覆盖

async

Task OnParametersSetAsync

()

{

if

(string

.IsNullOrWhiteSpace(Country))

{

customers = await

db.Customers.ToListAsync();

}

else

{

customers = await db.Customers

.Where(c => c.Country == Country).ToListAsync();

}

}

}

```

1.  启动`Northwind.BlazorServer`项目网站。

1.  启动 Chrome。

1.  导航到`https://localhost:5001/`。

1.  在左侧导航菜单中，点击**全球的客户**，注意客户表从数据库加载并在网页中呈现，如*图 17.5*所示：![](img/Image00154.jpg)

图 17.5：全球客户列表

1.  在左侧导航菜单中，点击**德国的客户**，注意客户表被过滤，只显示德国客户。

1.  在浏览器地址栏中，将`德国`改为`英国`，注意客户表被过滤，只显示英国客户。

1.  在左侧导航菜单中，点击**主页**，注意当作为嵌入式组件在页面上使用时，客户组件也能正常工作。

1.  点击任何编辑或删除按钮，并注意它们返回一个消息，说`抱歉，这个地址上没有任何内容`。因为我们还没有实现该功能。

1.  关闭浏览器。

1.  关闭 Web 服务器。

# 为 Blazor 组件抽象出一个服务

目前，Blazor 组件直接调用 Northwind 数据库上下文来获取客户。这在 Blazor Server 中可以正常工作，因为组件在服务器上执行。但是当在 Blazor WebAssembly 中托管时，该组件将无法工作。

现在我们将创建一个本地依赖服务，以便更好地重用组件：

1.  在`Northwind.BlazorServer`项目中，在`Data`文件夹中，添加一个名为`INorthwindService.cs`的新文件。(Visual Studio 项目项模板名为**Interface**。)

1.  修改其内容以定义一个本地服务的契约，该契约抽象了 CRUD 操作，如下面的代码所示：

```cs

命名空间

Packt.Shared

;

公共

接口

INorthwindService

{

任务<List<Customer>> GetCustomersAsync();

任务<List<Customer>> GetCustomersAsync(string

国家);

任务<Customer?> GetCustomerAsync(string

id);

任务<Customer>

CreateCustomerAsync

(

Customer c

)

;

任务<Customer>

UpdateCustomerAsync

(

Customer c

)

;

任务

DeleteCustomerAsync

(

字符串

id

)

;

}

```

1.  在`Data`文件夹中，添加一个名为`NorthwindService.cs`的新文件，并修改其内容，以使用 Northwind 数据库上下文实现`INorthwindService`接口，如下面的代码所示：

```cs

使用

Microsoft.EntityFrameworkCore;

命名空间

Packt.Shared

;

公共

类

NorthwindService

: INorthwindService

{

私人的

只读的

NorthwindContext db;

公共

NorthwindService

(

NorthwindContext db

)

{

这

.db = db;

}

公共

任务<List<Customer>> GetCustomersAsync()

{

返回

db.Customers.ToListAsync();

}

公共

任务<List<Customer>> GetCustomersAsync(string

国家)

{

返回

db.Customers.Where(c => c.Country == country).ToListAsync();

}

公共

任务<Customer?> GetCustomerAsync(string

id)

{

返回

db.Customers.FirstOrDefaultAsync

(c => c.CustomerId == id);

}

公共

任务<Customer>

CreateCustomerAsync

(

Customer c

)

{

db.Customers.Add(c);

db.SaveChangesAsync();

返回

Task.FromResult(c);

}

公共

任务<Customer>

UpdateCustomerAsync

(

Customer c

)

{

db.Entry(c).State = EntityState.Modified;

db.SaveChangesAsync();

返回

Task.FromResult(c);

}

公共

任务

DeleteCustomerAsync

(

字符串

id

)

{

Customer? customer = db.Customers.FirstOrDefaultAsync

(c => c.CustomerId == id).Result;

如果

(customer == null

)

{

返回

Task.CompletedTask;

}

否则

{

db.Customers.Remove(customer);

返回

db.SaveChangesAsync();

}

}

}

```

1.  在`Program.cs`中，配置服务的部分中，添加一条语句，将`NorthwindService`注册为实现`INorthwindService`接口的瞬态服务，如下面的代码所示：

```cs

builder.Services.AddTransient<INorthwindService, NorthwindService>();

```

1.  在`Pages`文件夹中，打开`Customers.razor`并用指令替换 Northwind 数据库上下文的指令，用指令替换注册的 Northwind 服务，如下面的代码所示：

```cs

@inject INorthwindService service

```

1.  修改`OnParametersSetAsync`方法，调用服务，如下面代码中突出显示的那样：

```cs

受保护的

覆盖

async

任务

OnParametersSetAsync

()

{

如果

(string

.IsNullOrWhiteSpace(Country))

{

**customers =**

**等待**

**service.GetCustomersAsync();**

}

否则

{

**customers =**

**等待**

**service.GetCustomersAsync(Country);**

}

}

```

1.  启动`Northwind.BlazorServer`网站项目，并确认其保留与以前相同的功能。

## 使用 EditForm 组件定义表单

Microsoft 提供了用于构建表单的现成组件。我们将使用它们为客户提供创建和编辑功能。

Microsoft 提供了`EditForm`组件和几个表单元素，如`InputText`，使得在 Blazor 中使用表单更加容易。

`EditForm`可以设置模型，将其绑定到具有属性和事件处理程序的对象，用于自定义验证，以及识别模型类上的标准 Microsoft 验证属性，如下面的代码所示：

```cs

<

EditForm

模型

=

"@customer"

OnSubmit

=

"ExtraValidation"

<

DataAnnotationsValidator

/>

<

ValidationSummary

/>

<

InputText

id

=

"名称"

@

bind-Value

=

"customer.CompanyName"

/>

<

按钮

类型

=

"提交"

提交</

按钮

</

EditForm

@code {

私人的

Customer customer = new

();

私人的

空

ExtraValidation()

{

//执行任何额外的验证

}

}

```

作为`ValidationSummary`组件的替代方案，您可以使用`ValidationMessage`组件来在单个表单元素旁边显示消息。

## Building and using a customer form component

现在我们可以创建一个共享组件来创建或编辑客户：

1.  在`Shared`文件夹中，创建一个名为`CustomerDetail.razor`的新文件。（Visual Studio 项目项模板名为**Razor Component**。）此组件将在多个页面组件上重用。

1.  Modify its contents to define a form to edit the properties of a customer, as shown in the following code:

```cs

<

EditForm

Model

=

"@Customer"

OnValidSubmit

=

"@OnValidSubmit"

<

DataAnnotationsValidator

/>

<

div

class

=

"form-group"

<

div

<

label

Customer Id</

label

<

div

<

InputText

@

bind-Value

=

"@Customer.CustomerId"

/>

<

ValidationMessage

For

=

"@(() => Customer.CustomerId)"

/>

</

div

</

div

</

div

<

div

class

=

"form-group "

<

div

<

label

Company Name</

label

<

div

<

InputText

@

bind-Value

=

"@Customer.CompanyName"

/>

<

ValidationMessage

For

=

"@(() => Customer.CompanyName)"

/>

</

div

</

div

</

div

<

div

class

=

"form-group "

<

div

<

label

Address</

label

<

div

<

InputText

@

bind-Value

=

"@Customer.Address"

/>

<

ValidationMessage

For

=

"@(() => Customer.Address)"

/>

</

div

</

div

</

div

<

div

class

=

"form-group "

<

div

<

label

Country</

label

<

div

<

InputText

@

bind-Value

=

"@Customer.Country"

/>

<

ValidationMessage

For

=

"@(() => Customer.Country)"

/>

</

div

</

div

</

div

<

button

type

=

"submit"

class

=

"btn btn-@ButtonStyle"

@ButtonText

</

button

</

EditForm

@code {

[Parameter

]

public

Customer Customer { get

; set

; } = null!;

[Parameter

]

public

string

ButtonText { get

; set

; } = "Save Changes";

[Parameter

]

public

string

ButtonStyle { get

; set

; } = "info";

[Parameter

]

public

EventCallback OnValidSubmit { get

; set

; }

}

```

1.  In the `Pages` folder, create a new file named `CreateCustomer.razor` . This will be a routable page component.

1.  Modify its contents to use the customer detail component to create a new customer, as shown in the following code:

```cs

@page "/createcustomer"

@inject INorthwindService service

@inject NavigationManager navigation

<h3

>Create Customer</h3

<CustomerDetail

ButtonText

="Create Customer"

Customer

="@customer"

OnValidSubmit

="@Create"

/>

@code {

private

Customer customer = new

();

private

async

Task

Create

()

{

await

service.CreateCustomerAsync(customer);

navigation.NavigateTo("customers"

);

}

}

```

1.  在`Pages`文件夹中，打开名为`Customers.razor`的文件，并在`<h3>`元素之后，添加一个`<div>`元素，其中包含一个按钮，用于导航到`createcustomer`页面组件，如下面的标记所示：

```cs

<

div

class

=

"form-group"

<

a

class

=

"btn btn-info"

href

=

"createcustomer"

<

i

class

=

"oi oi-plus"

></

i

Create New</

a

</

div

```

1.  在`Pages`文件夹中，创建一个名为`EditCustomer.razor`的新文件，并修改其内容以使用客户详细组件来编辑并保存对现有客户的更改，如下面的代码所示：

```cs

@page "/editcustomer/{customerid}"

@inject INorthwindService service

@inject NavigationManager navigation

<h3

>Edit Customer</h3

<CustomerDetail

ButtonText

="Update"

Customer

="@customer"

OnValidSubmit

"@Update"

/>

@code {

[Parameter

]

public

string

CustomerId { get

; set

; }

private

Customer? customer = new

();

protected

async

override

Task

OnParametersSetAsync

()

{

customer = await

service.GetCustomerAsync(CustomerId);

}

private

async

Task

Update

()

{

if

(customer is

not

null

)

{

await

service.UpdateCustomerAsync(customer);

}

navigation.NavigateTo("customers"

);

}

}

```

1.  在`Pages`文件夹中，创建一个名为`DeleteCustomer.razor`的新文件，并修改其内容以使用客户详细组件来显示即将删除的客户，如下面的代码所示：

```cs

@page "/deletecustomer/{customerid}"

@inject INorthwindService service

@inject NavigationManager navigation

<h3

>删除客户</h3

<div

类

="警报警报-危险"

警告！此操作无法撤消！

</div

<CustomerDetail

按钮文本

="删除客户"

按钮样式

="危险"

客户

="@customer"

OnValidSubmit

="@Delete"

/>

@code {

[参数

]

公共

字符串

CustomerId { get

; 设置

; }

私人的

客户？客户 =新

();

受保护的

async

覆盖

任务

OnParametersSetAsync

()

{

customer = await

service.GetCustomerAsync(CustomerId);

}

私人的

async

任务

删除

()

{

如果

（客户是

不

空

)

{

等待

service.DeleteCustomerAsync(CustomerId);

}

navigation.NavigateTo("customers"

);

}

}

```

## 测试客户表单组件

现在我们可以测试客户表单组件以及如何使用它来创建、编辑和删除客户：

1.  启动`Northwind.BlazorServer`网站项目。

1.  启动 Chrome。

1.  转到`https://localhost:5001/`。

1.  转到**全球客户**，然后单击**+创建新**按钮。

1.  输入无效的**客户 ID**，如`ABCDEF`，离开文本框，然后注意验证消息，如*图 17.6*所示：![](img/Image00155.jpg)

图 17.6：创建新客户并输入无效的客户 ID

1.  将**客户 ID**更改为`ABCDE`，为其他文本框输入值，然后单击**创建客户**按钮。

1.  当客户列表出现时，向下滚动页面底部以查看新客户。

1.  在**ABCDE**客户行上，单击**编辑**图标按钮，更改地址，单击**更新**按钮，注意客户记录已更新。

1.  在**ABCDE**客户行上，单击**删除**图标按钮，注意警告，然后单击**删除客户**按钮，注意客户记录已删除。

1.  关闭 Chrome 并关闭 Web 服务器。

# 使用 Blazor WebAssembly 构建组件

现在我们将在 Blazor WebAssembly 项目中重用相同的功能，以便您可以清楚地看到关键区别。

由于我们在`INorthwindService`接口中抽象了本地依赖服务，我们将能够重用所有组件和该接口，以及实体模型类。唯一需要重写的部分是`NorthwindService`类的实现。它将不再直接调用`NorthwindContext`类，而是在服务器端调用一个客户 Web API 控制器，如*图 17.7*所示：

![](img/Image00156.jpg)

图 17.7：比较使用 Blazor Server 和 Blazor WebAssembly 的实现

## 为 Blazor WebAssembly 配置服务器

首先，我们需要一个 Web 服务，客户端应用程序可以调用该服务来获取和管理客户。如果您完成了*第十六章*，*构建和使用 Web 服务*，那么您在`Northwind.WebApi`服务项目中可能已经有一个客户服务可供使用。但是，为了使本章更加自包含，让我们在`Northwind.BlazorWasm.Server`项目中构建一个客户 Web API 控制器：

**警告！**与以前的项目不同，共享项目的相对路径引用，如实体模型和数据库，是向上两级，例如，`"..\.."`。

1.  在`Server`项目/文件夹中，打开`Northwind.BlazorWasm.Server.csproj`并添加语句以引用 Northwind 数据库上下文项目，无论是 SQL Server 还是 SQLite，如下标记所示：

```cs

<ItemGroup>

<!-- 如果您更喜欢，则将 Sqlite 更改为 SqlServer

您更喜欢的话 -->

<ProjectReference Include="..\..\Northwind.Common.DataContext.Sqlite

\Northwind.Common.DataContext.Sqlite.csproj"

/>

</ItemGroup>

```

1.  构建`Northwind.BlazorWasm.Server`项目。

1.  在`Server`项目/文件夹中，打开`Program.cs`并添加一条语句以导入与 Northwind 数据库上下文一起使用的命名空间，如下所示：

```cs

使用

Packt.Shared;

```

1.  在配置服务的部分，添加一条语句来注册 Northwind 数据库上下文，无论是 SQL Server 还是 SQLite，如下所示：

```cs

// 如果使用 SQL Server

builder.Services.AddNorthwindContext();

// 如果使用 SQLite

builder.Services.AddNorthwindContext(

relativePath: Path.Combine(".."

, ".."

));

```

1.  在`Server`项目中，在`Controllers`文件夹中，创建一个名为`CustomersController.cs`的文件，并添加语句来定义一个 Web API 控制器类，其中包含与之前类似的 CRUD 方法，如下面的代码所示：

```cs

使用

Microsoft.AspNetCore.Mvc; // [ApiController], [Route]

使用

Microsoft.EntityFrameworkCore; // ToListAsync, FirstOrDefaultAsync

使用

Packt.Shared; // NorthwindContext, Customer

命名空间

Northwind.BlazorWasm.Server.Controllers

;

[ApiController

]

[Route(

"api/[controller]"

)

]

公共

类

CustomersController

: ControllerBase

{

私人的

只读

NorthwindContext db;

公共

CustomersController

(

NorthwindContext db

)

{

这

.db = db;

}

[HttpGet

]

公共

异步

Task<List<Customer>> GetCustomersAsync()

{

返回

await

db.Customers.ToListAsync();

}

[HttpGet(

"in/{country}"

)

] // 不同的路径以消除歧义

公共

异步

Task<List<Customer>> GetCustomersAsync(string

国家)

{

返回

await

db.Customers

.Where(c => c.Country == country).ToListAsync();

}

[HttpGet(

"{id}"

)

]

公共

异步

Task<Customer?> GetCustomerAsync(string

id)

{

返回

await

db.Customers

.FirstOrDefaultAsync(c => c.CustomerId == id);

}

[HttpPost

]

公共

异步

Task<Customer?> CreateCustomerAsync

(Customer customerToAdd)

{

Customer? existing = await

db.Customers.FirstOrDefaultAsync

(c => c.CustomerId == customerToAdd.CustomerId);

如果

(现有 == null

)

{

db.Customers.Add(customerToAdd);

int

受影响 = await

db.SaveChangesAsync();

如果

(受影响 == 1

)

{

返回

customerToAdd;

}

}

返回

现有;

}

[HttpPut

]

公共

异步

Task<Customer?> UpdateCustomerAsync(Customer c)

{

db.Entry(c).State = EntityState.Modified;

int

受影响 = await

db.SaveChangesAsync();

如果

(受影响 == 1

)

{

返回

c;

}

返回

空

;

}

[HttpDelete(

"{id}"

)

]

公共

异步

Task<

int

DeleteCustomerAsync

(

string

id

)

{

Customer? c = await

db.Customers.FirstOrDefaultAsync

(c => c.CustomerId == id);

如果

(c != null

)

{

db.Customers.Remove(c);

int

受影响 = await

db.SaveChangesAsync();

返回

受影响;

}

返回

0

;

}

}

```

## 为 Blazor WebAssembly 配置客户端

其次，我们可以重用 Blazor Server 项目中的组件。由于组件将是相同的，我们可以复制它们，只需要对抽象的 Northwind 服务的本地实现进行更改：

1.  在`Client`项目中，打开`Northwind.BlazorWasm.Client.csproj`并添加语句来引用 Northwind 实体模型库项目（而不是数据库上下文项目），无论是针对 SQL Server 还是 SQLite，如下面的标记所示：

```cs

<ItemGroup>

<!-- 如果您喜欢，将 Sqlite 更改为 SqlServer -->

<ProjectReference Include="..\..\Northwind.Common.EntityModels.Sqlite\

Northwind.Common.EntityModels.Sqlite.csproj"

/>

</ItemGroup>

```

1.  构建`Northwind.BlazorWasm.Client`项目。

1.  在`Client`项目中，打开`_Imports.razor`并导入`Packt.Shared`命名空间，以便在所有 Blazor 组件中使用 Northwind 实体模型类型，如下面的代码所示：

```cs

@using Packt.Shared

```

1.  在`Client`项目中，在`Shared`文件夹中，打开`NavMenu.razor`并添加一个`NavLink`元素，用于全球和法国的客户，如下面的标记所示：

```cs

<

div

类

=

"nav-item px-3"

<

NavLink

类

=

"nav-link"

href

=

"customers"

匹配

=

"NavLinkMatch.All"

<

跨度

类

=

"oi oi-people"

aria-hidden

=

"true"

></

跨度

全球客户

</

NavLink

</

div

<

div

类

=

"nav-item px-3"

<

NavLink

类

=

"nav-link"

href

=

"customers/France"

<

跨度

类

=

"oi oi-people"

aria-hidden

=

"true"

></

跨度

法国的客户

</

NavLink

</

div

```

1.  从`Northwind.BlazorServer`项目的`Shared`文件夹复制`CustomerDetail.razor`组件到`Northwind.BlazorWasm` `Client`项目的`Shared`文件夹。

1.  从`Northwind.BlazorServer`项目的`Pages`文件夹复制以下可路由页面组件到`Northwind.BlazorWasm` `Client`项目的`Pages`文件夹：

+   `CreateCustomer.razor`

+   `Customers.razor`

+   `DeleteCustomer.razor`

+   `EditCustomer.razor`

1.  在`Client`项目中，创建一个`Data`文件夹。

1.  将`INorthwindService.cs`文件从`Northwind.BlazorServer`项目的`Data`文件夹复制到`Client`项目的`Data`文件夹。

1.  在`Data`文件夹中，添加一个名为`NorthwindService.cs`的新文件。

1.  修改其内容以通过使用`HttpClient`实现`INorthwindService`接口来调用客户 Web API 服务，如下面的代码所示：

```cs

使用

System.Net.Http.Json; // GetFromJsonAsync, ReadFromJsonAsync

使用

Packt.Shared; // Customer

命名空间

Northwind.BlazorWasm.Client.Data

{

公共

类

NorthwindService

：INorthwindService

{

私人的

只读

HttpClient http;

公共

NorthwindService

(

HttpClient http

)

{

这个

.http = http;

}

公共

任务<List<Customer>> GetCustomersAsync()

{

返回

http.GetFromJsonAsync

<List<Customer>>("api/customers"

);

}

public

任务<List<Customer>> GetCustomersAsync(string

国家)

{

返回

http.GetFromJsonAsync

<List<Customer>>($"api/customers/in/

{国家}

"

);

}

公共

任务<Customer>

GetCustomerAsync

(

字符串

id

)

{

返回

http.GetFromJsonAsync

<Customer>($"api/customers/

{id}

"

);

}

公共

async

任务<Customer>

CreateCustomerAsync

(

Customer c

)

{

HttpResponseMessage response = await

http.PostAsJsonAsync("api/customers"

，c);

返回

await

response.Content

.ReadFromJsonAsync<Customer>();

}

公共

async

任务<Customer>

UpdateCustomerAsync

(

客户 c

)

{

HttpResponseMessage response = await

http.PutAsJsonAsync("api/customers"

，c);

返回

await

response.Content

.ReadFromJsonAsync<Customer>();

}

公共

async

任务

DeleteCustomerAsync

(

字符串

id

)

{

HttpResponseMessage response = await

http.DeleteAsync($"api/customers/

{id}

"

);

}

}

}

```

1.  在`Program.cs`中，导入`Packt.Shared`和`Northwind.BlazorWasm.Client.Data`命名空间。

1.  在配置服务的部分，添加一个语句来注册 Northwind 依赖服务，如下面的代码所示：

```cs

builder.Services.AddTransient<INorthwindService, NorthwindService>();

```

## 测试 Blazor WebAssembly 组件和服务

现在我们可以启动 Blazor WebAssembly 服务器托管项目，以测试组件是否与调用客户 Web API 服务的抽象 Northwind 服务一起工作：

1.  在`Server`项目/文件夹中，启动`Northwind.BlazorWasm.Server`网站项目。

1.  启动 Chrome，显示**开发人员工具**，并选择**网络**选项卡。

1.  导航到`https://localhost:5001/`。您的端口号将不同，因为它是随机分配的。查看控制台输出以发现它是什么。

1.  选择**控制台**选项卡，并注意 Blazor WebAssembly 已将.NET 程序集加载到浏览器缓存中，并且它们占用约 10MB 的空间，如*图 17.8*所示：![](img/Image00157.jpg)

图 17.8：Blazor WebAssembly 将.NET 程序集加载到浏览器缓存中

1.  选择**网络**选项卡。

1.  在左侧导航菜单中，单击**全球客户**，注意包含所有客户的 JSON 响应的 HTTP `GET`请求，如*图 17.9*所示：![](img/Image00158.jpg)

图 17.9：包含所有客户的 JSON 响应的 HTTP GET 请求

1.  单击**+创建新**按钮，填写表单以添加新客户，然后注意所做的 HTTP `POST`请求，如*图 17.10*所示：![](img/Image00159.jpg)

图 17.10：创建新客户的 HTTP POST 请求

1.  重复之前的步骤来编辑然后删除新创建的客户。

1.  关闭 Chrome 并关闭 Web 服务器。

# 改进 Blazor WebAssembly 应用程序

有一些常见的方法可以改进 Blazor WebAssembly 应用程序。我们现在将看一些最受欢迎的方法。

## 启用 Blazor WebAssembly AOT

默认情况下，Blazor WebAssembly 使用的.NET 运行时是使用 WebAssembly 编写的解释器进行 IL 解释。与其他.NET 应用程序不同，它不使用即时（JIT）编译器，因此 CPU 密集型工作负载的性能低于您可能希望的水平。

在.NET 6 中，微软已经添加了对**提前编译**（**AOT**）的支持，但您必须明确选择，因为尽管它可以显著提高运行时性能，AOT 编译可能需要几分钟，例如在本书中的小型项目上，对于更大的项目可能需要更长的时间。编译后的应用程序大小也比没有 AOT 时要大，通常是两倍大小。因此，使用 AOT 的决定是基于增加的编译和浏览器下载时间的平衡，可能会有更快的运行时。

AOT 是微软调查中最受欢迎的功能，并且缺乏 AOT 被认为是一些开发人员尚未采用.NET 开发**单页应用程序（SPAs）**的主要原因。

让我们安装 Blazor AOT 所需的附加工作负载，名为**.NET WebAssembly 构建工具**，然后为我们的 Blazor WebAssembly 项目启用 AOT：

1.  在具有管理员权限的命令提示符或终端中，安装 Blazor AOT 工作负载，如下命令所示：

```cs

dotnet workload install wasm-tools

```

1.  请注意以下部分输出中显示的消息：

```cs

...

安装包 Microsoft.NET.Runtime.MonoAOTCompiler.Task 版本 6.0.0...

安装包 Microsoft.NETCore.App.Runtime.AOT.Cross.browser-wasm 版本 6.0.0...

成功安装了工作负载 wasm-tools。

```

1.  修改`Northwind.BlazorWasm.Client`项目文件以启用 AOT，如下标记所示：

```cs

<PropertyGroup>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<ImplicitUsings>enable</ImplicitUsings>

<ServiceWorkerAssetsManifest>service-worker-assets.js

</ServiceWorkerAssetsManifest>

**<RunAOTCompilation>**

**true**

**</RunAOTCompilation>**

</PropertyGroup>

```

1.  发布`Northwind.BlazorWasm.Client`项目，如下命令所示：

```cs

dotnet publish -c Release

```

1.  请注意，75 个程序集已经应用了 AOT，如下部分输出所示：

```cs

Northwind.BlazorWasm.Client -> C:\Code\PracticalApps\Northwind.BlazorWasm\Client\bin\Release\net6.0\Northwind.BlazorWasm.Client.dll

Northwind.BlazorWasm.Client（Blazor 输出）-> C:\Code\PracticalApps\Northwind.BlazorWasm\Client\bin\Release\net6.0\wwwroot

优化大小的程序集，这可能会改变应用程序的行为。发布后一定要进行测试。请参阅：https://aka.ms/dotnet-illink

AOT'ing 75 个程序集

[1/75] Microsoft.Extensions.Caching.Abstractions.dll -> Microsoft.Extensions.Caching.Abstractions.dll.bc

...

[75/75] Microsoft.EntityFrameworkCore.Sqlite.dll -> Microsoft.EntityFrameworkCore.Sqlite.dll.bc

使用 emcc 编译本机资源。这可能需要一段时间...

...

使用 emcc 进行链接。这可能需要一段时间...

...

优化 dotnet.wasm...

压缩 Blazor WebAssembly 发布工件。这可能需要一段时间...

```

1.  等待进程完成。即使在现代多核 CPU 上，该过程也可能需要大约 20 分钟。

1.  转到`Northwind.BlazorWasm\Client\bin\release\net6.0\publish`文件夹，并注意从 10 MB 增加到 112 MB 的下载大小。

没有 AOT 时，下载的 Blazor WebAssembly 应用程序大约占用 10 MB 的空间。有了 AOT，大约占用 112 MB。这种大小的增加会影响网站访问者的体验。

使用 AOT 是在较慢的初始下载和更快的潜在执行之间取得平衡。根据您的应用程序的具体情况，AOT 可能并不值得。

## 探索渐进式 Web 应用程序支持

Blazor WebAssembly 项目中的**渐进式 Web 应用程序（PWA）**支持意味着 Web 应用程序具有以下好处：

+   直到访问者明确决定进入完整的应用程序体验之前，它都会像普通网页一样运行。

+   安装应用程序后，从操作系统的开始菜单或桌面启动它。

+   它在自己的应用程序窗口中可视化显示，而不是在浏览器选项卡中。

+   它可以离线工作。

+   它会自动更新。

让我们看看 PWA 支持的实际效果：

1.  启动`Northwind.BlazorWasm.Server` web 主机项目。

1.  转到`https://localhost:5001/`或您的端口号。

1.  在 Chrome 中，在地址栏的右侧，单击带有工具提示**安装 Northwind.BlazorWasm**的图标，如*图 17.11*所示：![](img/Image00160.jpg)

图 17.11：将 Northwind.BlazorWasm 安装为应用程序

1.  单击**安装**按钮。

1.  关闭 Chrome。如果应用程序自动运行，您可能还需要关闭应用程序。

1.  从 Windows 开始菜单或 macOS Launchpad 启动**Northwind.BlazorWasm**应用程序，并注意它具有完整的应用程序体验。

1.  在标题栏的右侧，单击三个点的菜单，注意您可以卸载该应用，但现在不要这样做。

1.  转到**开发人员工具**。在 Windows 上，按 F12 或 Ctrl + Shift + I。在 macOS 上，按 Cmd + Shift + I。

1.  选择**网络**选项卡，然后在**限制**下拉菜单中选择**离线**预设。

1.  在左侧导航菜单中，单击**主页**，然后单击**全球客户**，注意无法加载任何客户和应用程序窗口底部的错误消息，如*图 17.12*所示：![](img/Image00161.jpg)

图 17.12：当网络离线时无法加载任何客户

1.  在**开发人员工具**中，将**限制**设置回**禁用：无限制**。

1.  单击应用程序底部的黄色错误栏中的**重新加载**链接，并注意功能是否恢复。

1.  您现在可以卸载 PWA 应用程序，或者只是关闭它。

### 为 PWA 实现离线支持

我们可以通过在本地缓存来自 Web API 服务的 HTTP `GET`响应，将新的、修改的或已删除的客户端存储在本地，然后在网络连接恢复后通过执行存储的 HTTP 请求与服务器进行同步来改善体验。但是，这需要大量的工作来实现良好，因此超出了本书的范围。

## 了解 Blazor WebAssembly 的浏览器兼容性分析器

使用.NET 6，Microsoft 已统一了所有工作负载的.NET 库。但是，尽管从理论上讲，这意味着 Blazor WebAssembly 应用程序可以完全访问所有.NET API，但实际上，它在浏览器沙箱中运行，因此存在一些限制。如果调用不受支持的 API，将会抛出`PlatformNotSupportedException`。

为了预先警告不支持的 API，您可以添加一个平台兼容性分析器，当您的代码使用浏览器不支持的 API 时，它会警告您。

**Blazor WebAssembly 应用程序**和**Razor 类库**项目模板会自动启用浏览器兼容性检查。

要手动激活浏览器兼容性检查，例如在**类库**项目中，添加一个条目到项目文件中，如下面的标记所示：

```cs

<ItemGroup>

<SupportedPlatform Include="browser"

/>

</ItemGroup>

```

Microsoft 装饰不支持的 API，如下面的代码所示：

```cs

[UnsupportedOSPlatform(

"浏览器"

)

]

public

void

DoSomethingOutsideTheBrowserSandbox

()

{

...

}

```

**良好实践**：如果您创建的库不应在 Blazor WebAssembly 应用程序中使用，则应以相同的方式装饰您的 API。

## 在一个类库中共享 Blazor 组件

我们目前在 Blazor Server 项目和 Blazor WebAssembly 项目中有重复的组件。最好将它们定义一次在一个类库项目中，并从另外两个 Blazor 项目中引用它们。

让我们创建一个新的 Razor 类库：

1.  使用您喜欢的代码编辑器添加一个新项目，如下面的列表中所定义的那样：

1.  项目模板：**Razor 类库** / `razorclasslib`

1.  工作区/解决方案文件和文件夹：`PracticalApps`

1.  项目文件和文件夹：`Northwind.Blazor.Customers`

1.  支持页面和视图：已检查

1.  在`Northwind.Blazor.Customers`项目中，添加对`Northwind.Common.EntityModels.Sqlite`或`SqlServer`项目的项目引用。

1.  在`Northwind.Blazor.Customers`项目中，添加一个条目来检查浏览器兼容性，如下所示的标记中突出显示的那样：

```cs

<Project Sdk="Microsoft.NET.Sdk.Razor"

<PropertyGroup>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<ImplicitUsings>enable</ImplicitUsings>

<AddRazorSupportForMvc>true

</AddRazorSupportForMvc>

</PropertyGroup>

<ItemGroup>

<FrameworkReference Include="Microsoft.AspNetCore.App"

/>

</ItemGroup>

<ItemGroup>

<ProjectReference Include="..\Northwind.Common.EntityModels.Sqlite

\Northwind.Common.EntityModels.Sqlite.csproj"

/>

</ItemGroup>

**<ItemGroup>**

**<SupportedPlatform Include=**

**"browser"**

**/>**

**</ItemGroup>**

</Project>

```

1.  在`Northwind.BlazorServer`项目中，添加对`Northwind.Blazor.Customers`项目的项目引用。

1.  构建`Northwind.BlazorServer`项目。

1.  在`Northwind.Blazor.Customers`项目中，删除`Areas`文件夹及其所有内容。

1.  将`_Imports.razor`文件从`Northwind.BlazorServer`项目的根目录复制到`Northwind.Blazor.Customers`项目的根目录。

1.  在`_Imports.razor`中，删除对`Northwind.BlazorServer`命名空间的两个导入，并添加一个语句来导入包含我们共享的 Blazor 组件的命名空间，如下所示的代码中突出显示的那样：

```cs

@using Northwind.Blazor.Customers.Shared

```

1.  创建三个名为`Data`，`Pages`和`Shared`的文件夹。

1.  将`INorthwindService.cs`从`Northwind.BlazorServer`项目的`Data`文件夹移动到`Northwind.Blazor.Customers`项目的`Data`文件夹。

1.  将`Northwind.BlazorServer`项目的`Shared`文件夹中的所有组件移动到`Northwind.Blazor.Customers`项目的`Shared`文件夹中。

1.  将`CreateCustomer.razor`，`Customers.razor`，`EditCustomer.razor`和`DeleteCustomer.razor`组件从`Northwind.BlazorServer`项目的`Pages`文件夹移动到`Northwind.Blazor.Customers`项目的`Pages`文件夹。

我们将保留其他页面组件，因为它们依赖于尚未正确重构的天气服务。

1.  在`Northwind.BlazorServer`项目中，在`_Imports.razor`中，删除对`Northwind.BlazorServer.Shared`的`using`语句，并添加导入类库中页面和共享组件的语句，如下所示的代码：

```cs

@using Northwind.Blazor.Customers.Pages

@using Northwind.Blazor.Customers.Shared

```

1.  在`Northwind.BlazorServer`项目中，在`App.razor`中，添加一个参数来告诉`Router`组件扫描附加的程序集，以设置类库中页面组件的路由，如下所示的代码中突出显示的那样：

```cs

<Router AppAssembly="@typeof(App).Assembly"

**AdditionalAssemblies=**

**"new[] { typeof(Customers).Assembly }"**

```

**良好实践**：只要指定的类在外部程序集中，就无所谓。我选择了`Customers`，因为它是最重要和明显的组件类。

1.  启动`Northwind.BlazorServer`项目，并注意它与以前的行为相同。

**良好实践**：现在您可以在其他 Blazor Server 项目中重用 Blazor 组件。但是，您不能在 Blazor WebAssembly 项目中使用该类库，因为它依赖于完整的 ASP.NET Core 工作负载。创建适用于两种托管模型的 Blazor 组件库超出了本书的范围。

## 与 JavaScript 的互操作

默认情况下，Blazor 组件无法访问浏览器功能，如本地存储、地理位置和媒体捕获，也无法访问任何 JavaScript 库，如 React 或 Vue。如果需要与它们交互，可以使用 JavaScript Interop。

让我们看一个示例，它使用浏览器窗口的警报框和本地存储，可以永久保存每个访问者的最多 5MB 的数据：

1.  在`Northwind.BlazorServer`项目中，在`wwwroot`文件夹中，添加一个名为`scripts`的文件夹。

1.  在`scripts`文件夹中，添加一个名为`interop.js`的文件。

1.  修改其内容，如下面的代码所示：

```cs

功能

messageBox

(

消息

）

{

窗口

.alert(message);

}

功能

setColorInStorage

（）

{

如果

（类型

（存储）！== "未定义"

）{

localStorage

.setItem("color"

，

文件

.getElementById("colorBox"

）。value);

}

}

功能

getColorFromStorage

（）

{

如果

（类型

（存储）！== "未定义"

）{

文件

.getElementById("colorBox"

）。value =

localStorage

.getItem("color"

);

}

}

```

1.  在`Pages`文件夹中，在`_Layout.cshtml`文件中，在添加 Blazor 服务器支持的`script`元素之后，添加一个引用刚刚创建的 JavaScript 文件的`script`元素，如下面的代码所示：

```cs

<

脚本

src

=

"scripts/interop.js"

></

脚本

```

1.  在`Pages`文件夹中，在`Index.razor`文件中，删除两个`Customers`组件实例，然后添加一个按钮和一个代码块，使用 Blazor JavaScript 运行时依赖服务调用 JavaScript 函数，如下面的代码所示：

```cs

<button

类型

="button"

类

="btn btn-info"

@onclick

="AlertBrowser"

戳浏览器</button

<hr

/>

<input

id

="colorBox"

/>

<button

类型

="button"

类

="btn btn-info"

@onclick

="SetColor"

设置颜色</button

<button

类型

="button"

类

="btn btn-info"

@onclick

="GetColor"

Get Color</button

@code {

[Inject

]

公共

IJSRuntime JSRuntime { get

; set

; } = null

！;

公共

async

任务

AlertBrowser

（）

{

await

JSRuntime.InvokeVoidAsync(

"messageBox"

，"Blazor 戳浏览器"

）;

}

公共

async

任务

SetColor

（）

{

await

JSRuntime.InvokeVoidAsync("setColorInStorage"

）;

}

公共

async

任务

GetColor

（）

{

await

JSRuntime.InvokeVoidAsync("getColorFromStorage"

）;

}

}

```

1.  启动`Northwind.BlazorServer`项目。

1.  启动 Chrome 并导航到`https://localhost:5001/`。

1.  在主页上，在文本框中输入`red`，然后单击**设置颜色**按钮。

1.  显示**开发者工具**，选择**应用程序**选项卡，展开**本地存储**，选择`https://localhost:5001`，并注意键值对`color-red`，如*图 17.13*所示：！[](img/Image00162.jpg)

图 17.13：使用 JavaScript 互操作在浏览器本地存储中存储颜色

1.  关闭 Chrome 并关闭 Web 服务器。

1.  启动`Northwind.BlazorServer`项目。

1.  启动 Chrome 并导航到`https://localhost:5001/`。

1.  在主页上，单击**获取颜色**按钮，并注意从本地存储中检索到的值`red`在文本框中显示，这是在访客会话之间的本地存储中检索到的。

1.  关闭 Chrome 并关闭 Web 服务器。

## Blazor 组件库

Blazor 有许多组件库。付费组件库来自 Telerik、DevExpress 和 Syncfusion 等公司。开源 Blazor 组件库包括以下内容：

+   Radzen Blazor 组件：[`blazor.radzen.com/`](https://blazor.radzen.com/)

+   Blazor 项目的一些很棒的开源项目：[`awesomeopensource.com/projects/blazor`](https://awesomeopensource.com/projects/blazor)

# 练习和探索

通过回答一些问题来测试您的知识和理解，进行一些实践，并深入研究本章的主题。

## 练习 17.1-测试您的知识

回答以下问题：

1.  Blazor 的两种主要托管模型是什么，它们有什么不同？

1.  在 Blazor 服务器网站项目中，与 ASP.NET Core MVC 网站项目相比，启动类需要额外的配置吗？

1.  Blazor 的好处之一是能够使用 C#和.NET 实现客户端组件，而不是 JavaScript。Blazor 组件需要任何 JavaScript 吗？

1.  在 Blazor 项目中，`App.razor`文件是做什么的？

1.  使用`<NavLink>`组件的好处是什么？

1.  如何将值传递给组件？

1.  使用`<EditForm>`组件的好处是什么？

1.  当参数设置时如何执行一些语句？

1.  当组件出现时，如何执行一些语句？

1.  在 Blazor Server 和 Blazor WebAssembly 项目的 `Program` 类中有两个关键区别是什么？

## 练习 17.2 – 通过创建乘法表组件进行练习

创建一个根据名为 `Number` 的参数呈现乘法表的组件，然后以两种方式测试您的组件。

首先，通过将组件的实例添加到 `Index.razor` 文件中，如下面的标记所示：

```cs

<

timestable

数字

=

"6"

/>

```

其次，通过在浏览器地址栏中输入路径，如下面的链接所示：

`https://localhost:5001/timestable/6`

## 练习 17.3 – 通过创建国家导航项进行练习

向 `CustomersController` 类添加一个操作方法，返回一个国家名称列表。

在共享的 `NavMenu` 组件中，调用客户的 Web 服务以获取国家名称列表，并循环遍历它们，为每个国家创建一个菜单项。

## 练习 17.4 – 探索主题

使用以下页面上的链接，了解本章涵盖的主题的更多细节：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-17---building-user-interfaces-using-blazor`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-17---building-user-interfaces-using-blazor)

# 摘要

在本章中，您学习了如何为服务器和 WebAssembly 托管 Blazor 组件。您看到了两种托管模型之间的一些关键区别，比如如何使用依赖服务来管理数据。
