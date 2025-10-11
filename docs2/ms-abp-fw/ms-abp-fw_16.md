# *第十二章*：与 MVC/Razor Pages 一起工作

ABP 框架被设计成模块化、分层且与 UI 框架无关。它非常适合服务器-客户端架构，从理论上讲，它可以与任何类型的 UI 技术一起工作。服务器端使用标准的身份验证协议并提供符合标准的 HTTP API。您可以使用您喜欢的 SPA 框架并轻松地消费服务器端 API。这样，您就可以利用 ABP 框架的整个服务器端基础设施。

然而，ABP 框架也帮助您进行 UI 开发。它提供系统，使您可以构建模块化用户界面、UI 主题、布局、导航菜单和工具栏。它使您在处理数据表、模态框和表单或与服务器进行身份验证和通信时的工作过程更加容易。

ABP 框架与以下 UI 框架良好集成，并提供启动解决方案模板：

+   ASP.NET Core MVC/Razor Pages

+   Blazor

+   Angular

在本书的第四部分，我将介绍如何使用 MVC/Razor Pages 和 Blazor UI 选项。在本章中，您将学习 ABP 框架的 MVC/Razor Page 基础设施是如何设计的，以及它如何帮助您完成常规的 UI 开发周期。

我将这种 UI 类型称为 MVC/Razor Pages，因为它支持 MVC 和 Razor Pages 两种方法。你甚至可以在单个应用程序中使用两者。然而，由于 Razor Pages（自 ASP.NET Core 2.0 以来引入）是微软推荐的新应用程序的方法，所有预构建的 ABP 模块、示例和文档都使用 Razor Pages 方法。

本章涵盖了以下主题：

+   理解主题系统

+   使用打包和压缩

+   与菜单一起工作

+   与 Bootstrap 标签助手一起工作

+   创建表单并实现验证

+   与模态框一起工作

+   使用 JavaScript API

+   消费 HTTP API

# 技术要求

如果您想跟随本章中的示例，您将需要一个支持 ASP.NET Core 开发的 IDE/编辑器。在某些时候，我们将使用 ABP CLI，因此您需要安装 ABP CLI，如*第二章*中所述，*ABP 框架入门*。最后，您需要安装 Node.js v14+ 以能够安装 NPM 包。

您可以从本书的 GitHub 仓库下载示例应用程序：[`github.com/PacktPublishing/Mastering-ABP-Framework`](https://github.com/PacktPublishing/Mastering-ABP-Framework)。它包含本章提供的一些示例。

# 理解主题系统

UI 样式是应用程序中最具定制性的部分，您有很多选择。您可以从 Bootstrap、Tailwind CSS 或 Bulma 等 UI 工具包之一作为您应用程序 UI 的基础开始。然后，您可以构建一个设计语言或从主题市场购买一个预构建的、价格低廉的 UI 主题。如果您正在构建一个独立的应用程序，您可以根据这些选择进行选择，并基于这些选择创建您的 UI 页面和组件。您的页面和样式不需要与另一个应用程序兼容。

另一方面，如果您想构建一个模块化应用程序，其中每个模块的 UI 都是独立开发的（可能由不同的团队完成），模块在运行时作为一个单一的应用程序组合在一起，您需要确定一个设计标准，所有模块开发者都应该实施，以便您有一个一致的用户界面。

由于 ABP 框架提供了一个模块化基础设施，它提供了一个主题系统，该系统确定了一组基础库和标准。这有助于确保应用程序和模块开发者可以构建 UI 页面和组件，而无需依赖于特定的主题或样式集。一旦模块/应用程序代码与主题无关且主题标准明确，您就可以构建不同的主题，并可以通过简单的配置轻松地为应用程序使用该主题。

ABP 框架提供了两个免费的预构建 UI 主题：

+   **基本**主题是一个基于纯 Bootstrap 样式的简约主题。如果您想从头开始构建样式，它是最理想的。

+   **LeptonX**主题是由 ABP 框架团队构建的现代且适用于生产的 UI 主题。

本书在所有示例中都使用基本主题。以下是 LeptonX 主题的截图：

![图 12.1 – LeptonX 主题和应用程序布局](img/Figure_12.01_B17287.jpg)

图 12.1 – LeptonX 主题和应用程序布局

预构建的 UI 主题作为 NuGet 和 NPM 包部署，因此您可以轻松安装和切换它们。

接下来的两节将介绍这些主题共享的基本基础库和布局。

## 基础库

为了使模块/应用程序独立于特定的主题，ABP 确定了一些基础 CSS 和 JavaScript 库，我们的模块/应用程序可以依赖这些库。

ABP 框架 MVC/Razor Pages UI 的第一个和最基本依赖是*Twitter Bootstrap*框架。从 ABP 框架版本 5.0 开始，使用 Bootstrap 5.x。

除了 Bootstrap 之外，还有一些其他的核心库依赖，例如 Datatables.Net、JQuery、JQuery Validation、FontAwesome、Sweetalert、Toastr、Lodash 等。如果您想在模块或应用程序中使用这些标准库，不需要额外的设置。

下一节将解释所需的布局系统，以便理解网页是如何构建的。

## 布局

一个典型的网络页面由两部分组成——布局和页面内容。布局塑造了整个页面的形状，通常包括主要标题、公司/产品标志、主要导航菜单、页脚和其他标准组件。以下截图显示了示例布局中的这些部分：

![图 12.2 – 页面布局的组成部分](img/Figure_12.02_B17287.jpg)

图 12.2 – 页面布局的组成部分

在现代网络应用中，布局被设计成响应式的，这意味着它们会根据当前用户使用的设备改变形状和位置，以适应该设备。

布局的内容在不同页面之间几乎保持不变——只有页面内容会变化。页面内容通常是布局的主要部分，如果内容大于屏幕高度，则可能会滚动。

一个网络应用在不同的部分/页面可能会有不同的布局要求。在 ABP 框架中，一个主题可以定义一个或多个布局。每个布局都有一个唯一的 `string` 名称，ABP 框架定义了四个标准布局名称：

+   `Application`：为具有标题、菜单、工具栏、页脚等后台办公风格网络应用设计。*图 12.1* 展示了一个示例。

+   `Account`：为登录、注册和其他与账户相关的页面设计。

+   `Public`：为面向公众的网站设计，例如你产品的着陆页。

+   `Empty`：一个没有实际布局的布局。页面内容覆盖整个屏幕。

这些字符串定义在 `Volo.Abp.AspNetCore.Mvc.UI.Theming.StandardLayouts` 类中。每个主题都必须定义 `Application`、`Account` 和 `Empty` 布局，因为它们对大多数应用来说是通用的。`Public` 布局是可选的，如果没有实现，将回退到 `Application` 布局。一个主题可以定义更多具有不同名称的布局。

`Application` 布局是默认的，除非你更改它。你可以按页面/视图或文件夹更改它。如果你为文件夹更改它，该文件夹下的所有页面/视图都将使用所选布局。

要更改页面/视图的布局，请注入 `IThemeManager` 服务并使用带有布局名称的 `CurrentTheme.GetLayout` 方法：

```cs
@inject IThemeManager ThemeManager
@{
    Layout = ThemeManager.CurrentTheme
             .GetLayout(StandardLayouts.Empty);
}
```

在这里，你可以使用 `StandardLayouts` 类来获取标准布局名称。对于这个例子，我们可以使用 `GetLayout("Empty")`，因为 `StandardLayouts.Empty` 的值是一个常量 `string`，其值为 `Empty`。这样，你可以通过它们的 `string` 名称获取你主题的非标准布局。

如果你想更改文件夹中所有页面/视图的布局，你可以在该文件夹中创建一个 `_ViewStart.cshtml` 文件，并将以下代码放入其中：

```cs
@using Volo.Abp.AspNetCore.Mvc.UI.Theming
@inject IThemeManager ThemeManager
@{
    Layout = ThemeManager.CurrentTheme
             .GetLayout(StandardLayouts.Account);
}
```

如果你将那个 `_ViewStart.cshtml` 文件放在 `Pages` 文件夹中（或 MVC 视图的 `Views` 中），除非你为子文件夹或特定页面/视图选择了另一个布局，否则所有页面都将使用所选布局。

我们可以轻松选择一个布局来放置页面内容。下一节将解释如何将脚本/样式文件导入到我们的页面中，并利用打包和压缩系统。

# 使用打包和压缩系统

ABP 提供了一站式解决方案，用于安装客户端包、将脚本/样式文件添加到页面中，并在开发和生产环境中打包和压缩这些文件。

让我们从安装应用程序的客户端包开始。

## 安装 NPM 包

NPM 是 JavaScript/CSS 库的事实上的包管理器。当您使用 MVC/Razor Pages UI 创建新解决方案时，您将在 Web 项目的根文件夹中看到一个`package.json`文件。`package.json`文件的初始内容可能看起来像这样：

```cs
{
  ...
  "dependencies": {
    "@abp/aspnetcore.mvc.ui.theme.basic": "⁵.0.0"
  }
}
```

初始时，我们有一个名为`@abp/aspnetcore.mvc.ui.theme.basic`的单个 NPM 包依赖项。此包依赖于所有必要的基 CSS/JavaScript 库，以支持基本主题。如果我们想安装另一个 NPM 包，我们可以使用标准的`npm install`（或`yarn add`）命令。

假设我们想在应用程序中使用*Vue.js*库。我们可以在 Web 项目的根目录下运行以下命令：

```cs
npm install vue
```

此命令将`vue` NPM 包安装到`node_modules/vue`文件夹中。然而，我们无法使用`node_modules`文件夹下的文件。我们应该将必要的文件复制到 Web 项目的`wwwroot`文件夹中，以便将它们导入到页面中。

您可以手动复制必要的文件，但这不是最佳方式。ABP 提供了一个`install-libs`命令，通过映射文件来自动化此过程。在 Web 项目下打开`abp.resourcemapping.js`文件，并将以下代码添加到`mappings`字典中：

```cs
"@node_modules/vue/dist/vue.min.js": "@libs/vue/"
```

`abp.resourcemapping.js`文件的最终内容应如下所示：

```cs
module.exports = {
    aliases: { },
    mappings: {
        "@node_modules/vue/dist/vue.min.js": "@libs/vue/"
    }
};
```

现在，我们可以在命令行终端中，在 Web 项目的根目录下运行以下命令：

```cs
abp install-libs
```

应将`vue.min.js`文件复制到`wwwroot/libs/vue`文件夹下：

![图 12.3 – 将 Vue.js 库添加到 Web 项目中

![img/Figure_12.03_B17287.jpg]

图 12.3 – 将 Vue.js 库添加到 Web 项目中

映射支持 glob 通配符模式。例如，您可以使用以下映射复制`vue`包中的所有文件：

```cs
"@node_modules/vue/dist/*": "@libs/vue/"
```

默认情况下，`libs`文件夹被提交到源代码控制系统（如 Git）。这意味着如果您的队友从您的源代码控制系统中获取代码，他们不需要运行`npm install`或`abp install-libs`命令。如果您愿意，可以将`libs`文件夹添加到源代码控制系统的忽略文件中（如 Git 的`.gitignore`）。在这种情况下，您需要在运行应用程序之前运行`npm install`和`abp install-libs`命令。

下一节将解释标准 ABP NPM 包。

## 使用标准包

构建模块化系统还有一个挑战——所有模块都应该使用相同（或兼容）版本的同一 NPM 包。ABP 框架提供了一套标准 NPM 包，以便 ABP 生态系统可以使用这些 NPM 包的相同版本，并自动映射到将资源复制到`libs`文件夹。

`@abp/vue`是这些标准包之一，可以用来在你的项目中安装 Vue.js 库。你可以安装这个包而不是`vue`包：

```cs
npm install @abp/vue
```

现在，你可以运行`abp install-libs`命令将`vue.min.js`文件复制到`wwwroot/libs/vue`文件夹。注意，你不需要在`abp.resourcemapping.js`文件中定义映射，因为`@abp/vue`包已经包含了必要的映射配置。

建议当它们可用时使用标准的`@abp/*`包。这样，你可以依赖相关库的标准版本，而且你不需要手动配置`abp.resourcemapping.js`文件。

然而，当你在项目中安装库时，你需要将其导入页面才能在应用程序中使用它。

## 导入脚本和样式文件

一旦我们安装了 JavaScript 或 CSS 库，我们就可以将其包含在任何页面或捆绑包中。让我们从最简单的情况开始——你可以使用以下代码将`vue.min.js`导入 Razor 页面或查看它：

```cs
@section scripts {
    <abp-script src=»/libs/vue/vue.min.js» />
}
```

在这里，我们正在将 JavaScript 文件导入到`scripts`部分，因此主题将它们放置在 HTML 文档的末尾，在全局脚本之后。`abp-script`是 ABP 框架定义的一个标签助手，用于将脚本包含到页面/视图中。它被渲染如下：

```cs
<script src=»/libs/vue/vue.min.js?_v=637653840922970000»></script>
```

我们可以使用标准的`script`标签，但`abp-script`有以下优点：

+   如果给定的文件尚未压缩，它会在生产（或预发布）环境中自动压缩该文件。如果文件未压缩且 ABP 在原始文件附近找到压缩文件，它将使用预压缩文件而不是在运行时动态压缩。

+   它添加了一个查询字符串参数来添加版本信息，这样当文件更改时，浏览器不会缓存它。这意味着当你重新部署应用程序时，浏览器不会意外地缓存你的脚本文件的老版本。

+   ABP 确保文件只添加到页面一次，即使你多次包含它。如果你希望构建模块化系统，这是一个很好的特性，因为不同的模块组件可能包含独立的相同库，而 ABP 框架消除了这种重复。

一旦我们在页面中包含了 Vue.js，我们就可以利用其力量创建高度动态的页面。以下是一个名为`VueDemo.cshtml`的 Razor 页面示例：

```cs
@page
@model MvcDemo.Web.Pages.VueDemoModel
@section scripts {
    <abp-script src=»/libs/vue/vue.min.js» />
    <script>
        var app = new Vue({
            el: '#app',
            data: {
                message: 'Hello Vue!'
            }
        })
    </script>
}
<div id="app">
    <h2>{{ message }}</h2>
</div>
```

如果你运行这个页面，UI 上会显示一个**Hello Vue!**消息。我建议在你需要构建复杂和动态用户界面的 MVC/Razor Pages 应用程序的一些页面中使用 Vue.js。

让我们进一步分析这个例子，并将自定义 JavaScript 代码移动到一个单独的文件中。在同一文件夹中创建一个名为 `VueDemo.cshtml.js` 的 JavaScript 文件：

![Figure 12.4 – 添加 JavaScript 文件

![Figure 12.04_B17287.jpg]

图 12.4 – 添加 JavaScript 文件

我更喜欢这种命名约定，但你可以为 JavaScript 文件设置任何名称。

Pages 文件夹下的 JavaScript/CSS 文件

在常规 ASP.NET Core 应用程序中，你应该将所有 JavaScript/CSS 文件放置在 `wwwroot` 文件夹下。ABP 允许你将 JavaScript/CSS 文件添加到 `Pages` 或 `Views` 文件夹，靠近相应的 `.cshtml` 文件。我发现这种方法非常实用，因为我们把相关的文件放在一起。

新的 JavaScript 文件的内容如下所示：

```cs
var app = new Vue({
    el: '#app',
    data: {
        message: 'Hello Vue!'
    }
});
```

现在，我们可以更新 `VueDemo.cshtml` 文件的内容，如下所示：

```cs
@page
@model MvcDemo.Web.Pages.VueDemoModel
@section scripts {
    <abp-script src=»/libs/vue/vue.min.js» />
    <abp-script src=»/Pages/VueDemo.cshtml.js» />
}
<div id="app">
    <h2>{{ message }}</h2>
</div>
```

将 JavaScript 代码保存在单独的文件中，并将其作为外部文件包含在页面上，就像前面的例子一样，这是一个好习惯。

与样式（CSS）文件一起工作与脚本文件一起工作非常相似。以下示例使用 `styles` 部分 和 `abp-style` 标签助手在页面上导入一个样式文件：

```cs
@section styles {
    <abp-style src="img/VueDemo.cshtml.css" />
}
```

我们可以将多个脚本或样式文件导入到页面中。下一节将展示如何在生产中将这些文件捆绑成一个单一的、压缩的文件。

## 创建页面 bundle

当我们在页面上使用多个 `abp-script`（或 `abp-style`）标签时，ABP 会单独包含页面上的文件，并在生产中包含压缩版本。然而，我们通常希望在生产中创建一个单一的捆绑和压缩文件。我们可以使用 `abp-script-bundle` 和 `abp-style-bundle` 标签助手为页面创建 bundle，如下例所示：

```cs
@section scripts {
    <abp-script-bundle>
        <abp-script src=»/libs/vue/vue.min.js» />
        <abp-script src=»/Pages/VueDemo.cshtml.js» />
    </abp-script-bundle>
}
```

在这里，我们正在创建一个包含两个文件的 bundle。ABP 会自动压缩这些文件，并将它们捆绑成一个文件，然后在生产环境中对这个单一文件进行版本控制。ABP 在第一次请求页面时执行捆绑操作，并将捆绑的文件缓存到内存中。它使用缓存的捆绑文件来处理后续请求。

你可以在 bundle 标签内使用条件逻辑或动态代码，如下例所示：

```cs
<abp-script-bundle>
    <abp-script src="img/validator.js" />
    @if (System.Globalization.CultureInfo
         .CurrentUICulture.Name == "tr")
    {
        <abp-script src="img/validator.tr.js" />
    }
    <abp-script src="img/some-other.js" />
</abp-script-bundle>
```

此示例向 bundle 添加了一个示例验证库，并条件性地添加了土耳其本地化脚本。如果用户的语言是土耳其语，则将土耳其本地化添加到 bundle 中。否则，不会添加。ABP 可以理解这种差异——它创建并缓存两个单独的 bundle，一个用于土耳其用户，另一个用于其他人。

有了这些，我们已经学会了如何为单个页面创建 bundle。在下一节中，我们将解释如何配置全局 bundle。

## 配置全局 bundle

Bundling 标签助手对于页面 bundle 非常有用。如果你正在创建自定义布局，你也可以使用它们。然而，当我们使用主题时，布局由主题控制。

假设我们已经决定在所有页面上使用 Vue.js 库，并希望将其添加到全局捆绑中，而不是逐页添加。为此，我们可以在模块的`ConfigureServices`中配置`AbpBundlingOptions`（在 Web 项目中），如下面的代码块所示：

```cs
Configure<AbpBundlingOptions>(options =>
{
    options.ScriptBundles.Configure(
        StandardBundles.Scripts.Global,
        bundle =>
        {
            bundle.AddFiles(«/libs/vue/vue.min.js»);
        }
    );
    options.StyleBundles.Configure(
        StandardBundles.Styles.Global,
        bundle =>
        {
            bundle.AddFiles("/global-styles.css");
        }
    );
});
```

`options.ScriptBundles.Configure`方法用于操作具有给定名称的捆绑。第一个参数是捆绑的名称。`StandardBundles.Scripts.Global`是一个`常量字符串`，其值是全局脚本捆绑的名称，由所有布局导入。前面的示例还向全局样式捆绑添加了一个 CSS 文件。

全局捆绑只是命名的捆绑。我们将在下一节中解释这些。

## 创建命名的捆绑

基于页面的捆绑是创建单个页面捆绑的简单方法。然而，在某些情况下，您将需要定义一个捆绑并在多个页面上重用它。如前所述，全局样式和脚本捆绑是命名的捆绑。我们也可以定义自定义命名的捆绑，并在任何页面或布局中导入捆绑。

以下示例定义了一个命名的捆绑，并在其中添加了三个 JavaScript 文件：

```cs
Configure<AbpBundlingOptions>(options =>
{
    options
        .ScriptBundles
        .Add("MyGlobalScripts", bundle => {
            bundle.AddFiles(
                "/libs/jquery/jquery.js",
                "/libs/bootstrap/js/bootstrap.js",
                "/scripts/my-global-scripts.js"
            );
        });                
});
```

我们可以在模块类（通常是 Web 层的模块类）的`ConfigureServices`中编写此代码。`options.ScriptBundles`和`options.StyleBundles`是两种捆绑类型。在这个例子中，我们使用了`ScriptBundles`属性来创建一个包含一些 JavaScript 文件的捆绑。

一旦我们创建了一个命名的捆绑，我们就可以使用`abp-script-bundle`和`abp-style-bundle`标签助手在页面/视图中使用它，如下面的示例所示：

```cs
<abp-script-bundle name="MyGlobalScripts" />
```

当我们在页面或视图中使用此代码时，所有脚本文件在开发时间单独添加到页面中。默认情况下，它们在生产环境中自动捆绑和压缩。下一节将解释如何更改此默认行为。

## 控制捆绑和压缩行为

我们可以使用`AbpBundlingOptions`选项类来更改捆绑和压缩系统的默认行为。请参见以下配置：

```cs
Configure<AbpBundlingOptions>(options =>
{
    options.Mode = BundlingMode.None;
});
```

此配置代码禁用了捆绑和压缩逻辑。这意味着即使在生产环境中，所有脚本/样式文件也是单独添加到页面中，而不进行捆绑和压缩。`options.Mode`可以取以下值之一：

+   `Auto`（默认）：在生产环境和预发布环境中捆绑和压缩，但在开发时间禁用捆绑和压缩。

+   `Bundle`: 将文件捆绑（为每个捆绑创建一个文件）但不会压缩样式/脚本。

+   `BundleAndMinify`：始终捆绑和压缩文件，即使在开发时间也是如此。

+   `None`：禁用捆绑和压缩过程。

在这本书中，我已经解释了捆绑和最小化系统的基本用法。然而，它具有高级功能，例如创建捆绑贡献者类、从另一个捆绑继承、扩展和操作捆绑等。这些功能在你想创建可重用 UI 模块时特别有帮助。请参阅 ABP 框架文档以了解所有功能：[`docs.abp.io/en/abp/latest/UI/AspNetCore/Bundling-Minification`](https://docs.abp.io/en/abp/latest/UI/AspNetCore/Bundling-Minification)。

在下一节中，你将学习如何与导航菜单一起工作。

# 与菜单一起工作

菜单由当前主题渲染，因此最终的应用程序或模块不能直接更改菜单项。你可以在 *图 12.1* 的左侧看到主菜单。ABP 提供了一个菜单系统，因此模块和最终应用程序可以动态添加新的菜单项或删除/更改由这些模块添加的项目。

我们可以使用 `AbpNavigationOptions` 来向菜单系统添加贡献者。ABP 会执行所有贡献者以动态构建菜单，如下面的示例所示：

```cs
Configure<AbpNavigationOptions>(options =>
{
    options.MenuContributors.Add(new MyMenuContributor());
});
```

在这里，`MyMenuContributor` 应该是一个实现了 `IMenuContributor` 接口的类。ABP 启动解决方案模板已经包含了一个可以直接使用的菜单贡献者类。`IMenuContributor` 定义了 `ConfigureMenuAsync` 方法，我们应该像这样实现它：

```cs
public class MvcDemoMenuContributor : IMenuContributor
{
    public async Task ConfigureMenuAsync(
        MenuConfigurationContext context)
    {
        if (context.Menu.Name == StandardMenus.Main)
        {
            //TODO: Configure the main menu
        }
    }
}
```

我们首先应该考虑的是菜单的名称。在 `StandardMenus` 类（在 `Volo.Abp.UI.Navigation` 命名空间中）中定义了两个标准菜单名称作为常量：

+   `Main`：应用程序的主菜单。它在 *图 12.1* 的左侧显示。

+   `User`：用户上下文菜单。当你点击页眉上的用户名时，它会打开。

因此，前面的示例检查了菜单的名称，并且只向主菜单添加了项目。下面的示例代码块添加了一个 **客户关系管理**（**CRM**）菜单项和两个子菜单项：

```cs
var l = context.GetLocalizer<MvcDemoResource>();
context.Menu.AddItem(
    new ApplicationMenuItem("MyProject.Crm", l["CRM"])
        .AddItem(new ApplicationMenuItem(
            name: "MyProject.Crm.Customers", 
            displayName: l["Customers"], 
            url: "/crm/customers")
        ).AddItem(new ApplicationMenuItem(
            name: "MyProject.Crm.Orders", 
            displayName: l["Orders"],
            url: "/crm/orders")
        )
);
```

在这个示例中，我们获取一个 `IStringLocalizer` 实例（`l`）以本地化菜单项的显示名称。`context.GetLocalizer` 是获取本地化服务的一个快捷方式。你可以使用 `context.ServiceProvider` 来解析任何服务并将你的自定义逻辑应用于构建菜单。

每个菜单项都应该有一个唯一的 `name`（例如本例中的 `MyProject.Crm.Customers`）和一个 `displayName`。有 `url`、`icon`、`order` 以及一些其他选项可用于控制菜单项的外观和行为。

基本主题渲染了示例菜单，如下面的屏幕截图所示：

![图 12.5 – 由基本主题渲染的菜单项

![图 12.05_B17287.jpg]

图 12.5 – 由基本主题渲染的菜单项

重要的是要理解，每次我们渲染菜单时都会调用 `ConfigureMenuAsync` 方法。对于一个典型的 MVC/Razor Pages 应用程序，此方法在每次页面请求时都会被调用。这样，你可以动态地塑造菜单，并条件性地添加或删除项目。你通常需要在添加菜单项时检查权限，如下面的代码块所示：

```cs
if (await context.IsGrantedAsync("MyPermissionName"))
{
    context.Menu.AddItem(...);
}
```

`context.IsGrantedAsync` 是检查当前用户权限名称的快捷方式。如果我们想手动解析和使用 `IAuthorizationService`，我们可以重写相同的代码，如下面的代码块所示：

```cs
var authorizationService = context
    .ServiceProvider.GetRequiredService<IAuthorizationService>();
if (await authorizationService.IsGrantedAsync(
    "MyPermissionName"))
{
    context.Menu.AddItem()
}
```

在此示例中，我使用了 `context.ServiceProvider` 来解析 `IauthorizationService`。然后，我像上一个示例一样使用了它的 `IsGrantedAsync` 方法。你可以安全地从 `context.ServiceProvider` 解析服务，并让 ABP 框架在菜单构建过程的末尾释放这些服务。

你也可以在 `context.Menu.Items` 集合中找到现有的菜单项（由依赖模块添加），以修改或删除它们。

在下一节中，我们将继续探讨 Bootstrap 标签辅助器，并学习如何以类型安全的方式渲染常见的 Bootstrap 组件。

# 使用 Bootstrap 标签辅助器

Bootstrap 是世界上最受欢迎的 UI（HTML/CSS/JS）库之一，它是所有 ABP 主题使用的根本 UI 框架。作为使用此类库作为标准库的好处，我们可以基于 Bootstrap 构建我们的 UI 页面和组件，并让主题为其添加样式。这样，我们的模块甚至应用程序都可以与主题无关，并与任何 ABP 兼容的 UI 主题一起工作。

Bootstrap 是一个文档齐全且易于使用的库。然而，在编写基于 Bootstrap 的 UI 代码时存在两个问题：

+   一些组件需要大量的样板代码。这些代码的大部分都是重复的，编写和维护都很繁琐。

+   在 MVC/Razor Pages 网络应用程序中编写纯 Bootstrap 代码并不非常类型安全。我们可能会在类名和 HTML 结构中犯错误，这些错误在编译时无法捕获。

ASP.NET Core MVC/Razor Pages 有一个 *标签* *辅助器* 系统来定义可重用的组件，并将它们用作我们页面/视图中的其他 HTML 标签。ABP 利用标签辅助器的力量，并为 Bootstrap 库提供了一套标签辅助器组件。这样，我们可以用更少的代码并以类型安全的方式构建基于 Bootstrap 的 UI 页面和组件。

使用 ABP 框架仍然可以编写原生 Bootstrap HTML 代码，并且 ABP 的 Bootstrap 标签辅助器并不涵盖 Bootstrap 100%。然而，我们建议尽可能使用 Bootstrap 标签辅助器。请参见以下示例：

```cs
<abp-button button-type="Primary" text="Click me!" />
```

在这里，我使用了 `abp-button` 标签辅助器来渲染 Bootstrap 按钮。我使用了具有编译时检查支持的 `button-type` 和 `text` 属性。此示例代码在运行时渲染如下：

```cs
<button class="btn btn-primary" type="button">
    Click me!
</button>
```

ABP 框架中有很多 Bootstrap 标签辅助工具，所以在这里我不会解释它们全部。请参考 ABP 的文档来学习如何使用它们：[`docs.abp.io/en/abp/latest/UI/AspNetCore/Tag-Helpers/Index`](https://docs.abp.io/en/abp/latest/UI/AspNetCore/Tag-Helpers/Index)。

在接下来的两个部分中，我们将使用一些这些 Bootstrap 标签辅助工具来构建表单项和打开模态。

# 创建表单和实现验证

ASP.NET Core 提供了良好的基础设施来准备表单，并在服务器端提交、验证和处理它们。然而，它仍然需要编写一些样板代码和重复的代码。ABP 框架通过提供标签辅助工具并在可能的情况下自动化验证和本地化来简化与表单的工作。让我们从如何使用 ABP 的标签辅助工具渲染表单元素开始。

## 渲染表单元素

`abp-input` 标签辅助工具用于渲染给定属性的适当 HTML 输入元素。最好通过一个完整的示例来展示其用法。

假设我们需要构建一个表单来创建一个新的 *电影* 实体，并且已经创建了一个名为 `CreateMovie.cshtml` 的新 Razor 页面。首先，让我们看看代码后置文件：

```cs
public class CreateMovieModel : AbpPageModel
{
    [BindProperty]
    public MovieViewModel Movie { get; set; }

    public void OnGet()
    {
        Movie = new MovieViewModel();
    }
    public async Task OnPostAsync()
    {
        // TODO: process the form (using the Movie object)
    }
}
```

页面模型通常是从 `PageModel` 类派生的。然而，我们是从 ABP 的 `AbpPageModel` 基类派生的，因为它提供了一些预注入的服务和辅助方法。这是一个简单的页面模型类。在这里，我们在 `OnGet` 方法中创建一个新的 `MovieViewModel` 实例，并将其绑定到表单元素。我们还有一个 `OnPostAsync` 方法，我们可以用它来处理提交的表单数据。`[BindProperty]` 告诉 ASP.NET Core 将请求数据绑定到 `Movie` 对象。

为了探索这个示例，让我们看看 `MovieViewModel` 类：

```cs
public class MovieViewModel
{
    [Required]
    [StringLength(256)]
    public string Name { get; set; }
    [Required]
    [DataType(DataType.Date)]
    public DateTime ReleaseDate { get; set; }
    [Required]
    [TextArea]
    [StringLength(1000)]
    public string Description { get; set; }
    public Genre Genre { get; set; }
    public float? Price { get; set; }
    public bool PreOrder { get; set; }
}
```

此对象用于渲染表单元素并在用户提交表单时绑定请求数据。请注意，一些属性具有数据注释验证属性，可以自动验证这些属性的值。在这里，`Genre` 属性是一个 `enum`，如下所示：

```cs
public enum Genre
{
    Classic, Action, Fiction, Fantasy, Animation
}
```

现在，我们可以切换到视图部分，尝试渲染一个表单来获取用户的电影信息。

首先，我将向您展示在没有 ABP 框架的情况下如何做到这一点，以便理解使用 ABP 框架的好处。首先，我们必须打开一个 `form` 元素，如下面的代码块所示：

```cs
<form method="post">
    <-- TODO: FORM ELEMENTS -->
    <button class="btn btn-primary" type="submit">
        Submit
    </button>
</form>
```

在 `form` 块中，我们为每个 `form` 元素编写代码，然后添加一个 `submit` 按钮来提交表单。展示完整的 `form` 代码对于这本书来说会太长，所以我会只展示渲染 `Movie.Name` 属性输入元素的必要代码：

```cs
<div class="form-group">
    <label asp-for="Movie.Name" class="control-label">
    </label>
    <input asp-for="Movie.Name" class="form-control"/>
    <span asp-validation-for="Movie.Name" 
        class="text-danger"></span>
</div>
```

如果您曾经使用 ASP.NET Core Razor Pages/MVC 和 Bootstrap 创建过表单，前面的代码块应该非常熟悉。它通过将它们包裹在 `form-group` 中来放置 `label`、实际的输入元素和验证消息区域。以下截图显示了渲染的表单：

![图 12.6 – 带有一个单行文本输入的简单表单![图 12.06 – B17287.jpg](img/Figure_12.06_B17287.jpg)

图 12.6 – 带有一个文本输入的简单表单

当前表单只为 `Name` 属性包含一个单独的文本输入。你可以为 `Movie` 类的每个属性编写类似的代码，这将导致代码庞大且重复。让我们看看如何使用 ABP 框架的 `abp-input` 标签助手渲染相同的输入：

```cs
<abp-input asp-for="Movie.Name" />
```

这很简单。现在，我们可以渲染所有表单元素。以下是将最终代码：

```cs
<form method="post">
    <abp-input asp-for="Movie.Name" />
    <abp-select asp-for="Movie.Genre" />
    <abp-input asp-for="Movie.Description" />
    <abp-input asp-for="Movie.Price" />
    <abp-input asp-for="Movie.ReleaseDate" />
    <abp-input asp-for="Movie.PreOrder" />
    <abp-button type="submit" button-type="Primary"
        text="Submit"/>
</form>
```

与标准的 Bootstrap 表单代码相比，前面的代码块显著更短。我使用了 `abp-select` 标签助手来处理 `Genre` 属性。它理解 `Genre` 是一个 `enum`，并使用 `enum` 成员创建下拉元素。以下是被渲染的表单：

![图 12.7 – 创建新电影的完整表单![图 12.07 – B17287.jpg](img/Figure_12.07_B17287.jpg)

图 12.7 – 创建新电影的完整表单

ABP 自动在必填表单字段的标签附近添加 *****。它读取类属性的类型和属性，并确定表单字段。

如果你只想按顺序渲染输入元素，你可以将最后一个代码块替换为以下代码：

```cs
<abp-dynamic-form abp-model="Movie" submit-button="true" />
```

`abp-dynamic-form` 标签助手获取一个模型并自动创建整个表单！

`abp-input`、`abp-select` 和 `abp-radio` 标签助手映射到类属性并渲染相应的 UI 元素。如果你想控制表单布局并在表单控件之间放置自定义 HTML 元素，可以使用它们。另一方面，`abp-dynamic-form` 在你较少控制表单布局的情况下使创建表单变得非常简单。无论你如何创建表单，ABP 都会为你自动化验证和本地化过程，我将在接下来的几节中解释。

## 验证用户输入

如果你尝试不填写必填字段提交表单，表单将不会提交到服务器，并且每个无效表单元素都会显示错误消息。以下截图显示了当你留空 `Name` 属性并提交表单时的错误消息：

![图 12.8 – 无效的用户输入![图 12.08 – B17287.jpg](img/Figure_12.08_B17287.jpg)

图 12.8 – 无效的用户输入

客户端验证是自动基于 `MovieViewModel.Name` 属性中的数据注释属性完成的。因此，你不需要为标准检查编写任何验证代码。用户必须确保所有字段有效后才能提交表单。

客户端验证只是为了用户体验。很容易绕过客户端验证并将无效的表单提交到服务器（通过在浏览器的开发者工具中操作或禁用 JavaScript 代码）。因此，你应该始终在服务器端验证用户输入，这应该在页面模型类的 `OnPostAsync` 方法中完成。以下代码块显示了处理表单提交时使用的常见模式：

```cs
public async Task OnPostAsync()
{
    if (ModelState.IsValid)
    {
        //TODO: Create a new movie
    }
    else
    {
        Alerts.Danger("Please correct the form fields!");
    }
}
```

如果任何表单字段无效，`ModelState.IsValid` 返回 `false`。这是 ASP.NET Core 的一个标准功能。你应该始终在 `if` 语句中处理输入。可选地，你可以在 `else` 语句中添加逻辑。在这个例子中，我使用了 ABP 的 `Alerts` 功能向用户显示客户端警告消息。以下截图显示了提交无效表单的结果：

![图 12.9 – 服务器端无效表单结果![图片](img/Figure_12.09_B17287.jpg)

图 12.9 – 服务器端无效表单结果

如果你查看 `MovieViewModel` 类的 `IValidatableObject` 接口下的验证错误消息，如下面的代码块所示：

```cs
public class MovieViewModel : IValidatableObject
{
    // ... properties omitted
    public IEnumerable<ValidationResult> Validate(
        ValidationContext validationContext)
    {
        if (PreOrder && Price > 999)
        {
            yield return new ValidationResult(
                "Price should be lower than 999 for 
                 pre-order movies!",
                new[] { nameof(Price) }
            );
        }
    }
}
```

我在 `Validate` 方法中执行复杂的自定义验证逻辑。你可以参考 *第七章*，*探索横切关注点*中的 *验证用户输入* 部分，了解更多关于服务器端验证的信息。在这里，我们应该理解我们可以在服务器端使用自定义逻辑，并在客户端显示验证消息。

在下一节中，我们将学习如何本地化验证错误以及表单标签。

## 本地化表单

ABP 框架根据当前语言自动本地化验证错误消息。尝试切换到另一种语言，并提交表单而不提供电影名称。以下截图显示了土耳其语言的情况：

![图 12.10 – 自动本地化的验证错误消息![图片](img/Figure_12.10_B17287.jpg)

图 12.10 – 自动本地化的验证错误消息

错误文本已更改。然而，你仍然可以看到 **名称** 作为字段名称，因为这是我们自定义的字段名称，我们还没有对其进行本地化。

ABP 为表单字段提供了一个基于约定的本地化系统。你只需在你的本地化 JSON 文件中定义一个本地化条目，键的格式为 `DisplayName:<属性名>`。我可以在 *Domain.Shared* 项目中的 `en.json` 文件中添加以下行以本地化电影创建表单的所有字段：

```cs
"DisplayName:Name": "Name",
"DisplayName:ReleaseDate": "Release date",
«DisplayName:Description»: «Description»,
«DisplayName:Genre»: «Genre»,
"DisplayName:Price": "Price",
"DisplayName:PreOrder": "Pre-order"
```

然后，我可以在 `tr.json` 文件中使用以下条目将这些内容本地化为土耳其语：

```cs
"DisplayName:Name": "İsim",
"DisplayName:ReleaseDate": "Yayınlanma tarihi",
"DisplayName:Description": "Açıklama",
"DisplayName:Genre": "Tür",
"DisplayName:Price": "Ücret",
"DisplayName:PreOrder": "Ön sipariş"
```

现在，我们有一个本地化的标签和一个更本地化的验证错误消息：

![图 12.11 – 完全本地化的验证错误消息和字段标签![图片](img/Figure_12.11_B17287.jpg)

图 12.11 – 完全本地化的验证错误消息和字段标签

将 `DisplayName:` 前缀添加到属性名是 `form` 字段的建议约定，但实际上并非必需。如果 ABP 找不到 `DisplayName:Price` 条目，它将搜索不带前缀的 `Price` 键的条目。如果你想指定属性的本地化键，你可以在属性顶部添加 `[DisplayName]` 属性，如下面的示例所示：

```cs
[DisplayName("MoviePrice")]
public float? Price { get; set; }
```

使用这种设置，ABP 将尝试使用 `"MoviePrice"` 键来本地化字段名称。

`abp-select` 标签根据约定将 `enum` 类型的下拉列表项本地化。你可以在本地化文件中添加条目，例如 `<enum-type>.<enum-member>`。对于 `Genre` 枚举类型的 `Action` 成员，我们可以添加一个带有 `Genre.Action` 键的本地化条目。如果找不到 `Genre.Action` 键，它将回退到 `Action` 键。

在下一节中，我们将讨论如何将标准表单转换为完全的 AJAX 表单。

## 实现 AJAX 表单

当用户提交标准表单时，会执行全页面的 POST 操作，服务器重新渲染整个页面。另一种方法是将表单作为 AJAX 请求发送，并在 JavaScript 代码中处理响应。这种方法比常规的 POST 请求快得多，因为浏览器不需要重新加载整个页面和页面的所有资源。在许多情况下，这也能提供更好的用户体验，因为你可以显示一些等待时的动画。此外，这样你不会丢失页面的状态，可以在 JavaScript 代码中执行智能操作。

你可以手动处理所有的 AJAX 事务，但 ABP 框架提供了内置的方式来处理这些常见模式。你可以在任何 `form` 元素（包括 `abp-dynamic-form` 元素）上添加 `data-ajaxForm="true"` 属性，使其通过 AJAX 请求发送。

以下示例为 `abp-dynamic-form` 添加了 AJAX 功能：

```cs
<abp-dynamic-form abp-model="Movie"
                  submit-button="true" 
                  data-ajaxForm="true"
                  id="MovieForm" />
```

当我们将表单转换为 AJAX 表单时，服务器端应该正确实现 POST 处理器。以下代码块展示了实现 POST 处理器的常见模式：

```cs
public async Task<IActionResult> OnPostAsync()
{
    ValidateModel();    
    //TODO: Create a new movie
    return NoContent();
}
```

第一行验证用户输入，如果输入模型无效，则抛出 `AbpValidationException`。`ValidateModel` 方法来自基类 `AbpPageModel`。如果你不想使用它，你可以检查 `if (ModelState.IsValid)` 并执行所需的任何操作。如果表单有效，你通常会将新电影保存到数据库中。最后，你可以将结果数据返回给客户端。对于这个例子，我们不需要返回响应，所以 `NoContent` 结果是合适的。

当你将表单转换为 AJAX 表单时，你通常希望在表单成功提交时采取行动。以下示例处理了表单的 `abp-ajax-success` 事件：

```cs
$(function (){
    $('#MovieForm').on('abp-ajax-success', function(){
        $('#MovieForm').slideUp();
        abp.message.success('Successfully saved, thanks
            :)');
    });
});
```

在这个例子中，我为表单的 `abp-ajax-success` 事件注册了一个回调函数。在这个回调中，你可以执行任何需要的操作。例如，我使用了 `slideUp` JQuery 函数来隐藏表单，然后使用了 ABP 的成功 UI 消息。我们将在本章的 *使用 JavaScript API* 部分回到 `abp.message` API。

对于 AJAX 请求，异常处理逻辑是不同的。ABP 处理所有异常，向客户端返回适当的 JSON 响应，然后自动在客户端处理错误。例如，假设表单有一个在服务器端确定的验证错误。在这种情况下，服务器返回一个验证错误消息，客户端显示一个消息框，如下面的截图所示：

![图 12.12 – AJAX 表单提交时的服务器端验证错误![图 12.12_B17287.jpg](img/Figure_12.12_B17287.jpg)

图 12.12 – AJAX 表单提交时的服务器端验证错误

消息框在任何异常中都会显示，包括你的自定义异常和`UserFriendlyException`。前往*第七章*的*异常处理*部分，*探索横切关注点*，以了解更多关于异常处理系统信息。

除了将表单转换为 AJAX 表单和处理异常之外，ABP 还防止在**提交**按钮的`data-busy-text`属性上双击以使用另一段文本。

在下一节中，我们将学习 ABP 框架如何帮助我们处理模态对话框。

# 与模态对话框一起工作

当你想要创建交互式用户界面时，模态对话框是基本组件之一。它提供了一种方便的方式，可以在不改变当前页面布局的情况下获取用户的响应或显示一些信息。

Bootstrap 有一个模态组件，但它需要一些样板代码。ABP 框架提供了`abp-modal`标签助手来渲染模态组件，这简化了在大多数用例中模态的使用。模态的另一个问题是将模态代码放在打开模态的页面中，这使得模态难以重用。ABP 在 JavaScript 端提供了一个模态 API，用于动态加载和控制这些模态。它也与模态内的表单很好地协同工作。让我们从最简单的用法开始。

## 理解模态的基础知识

ABP 建议将模态定义为独立的 Razor 页面（或者如果你使用 MVC 模式，则是视图）。因此，作为第一步，我们应该创建一个新的 Razor 页面。假设我们在`Pages`文件夹下创建了一个名为`MySimpleModal.cshtml`的新 Razor 页面。后端代码很简单：

```cs
public class MySimpleModalModel : AbpPageModel
{
    public string Message { get; set; }

    public void OnGet()
    {
        Message = "Hello modals!";
    }
}
```

我们在模态对话框中只显示一个`Message`属性。让我们看看视图方面：

```cs
@page
@model MvcDemo.Web.Pages.MySimpleModalModel
@{
    Layout = null;
}
<abp-modal>
    <abp-modal-header title="My header"></abp-modal-header>
    <abp-modal-body>
        @Model.Message
    </abp-modal-body>
    <abp-modal-footer buttons="Close"></abp-modal-footer>
</abp-modal>
```

这里的`Layout = null`语句是关键的。因为这个页面是通过 AJAX 请求加载的，结果应该只包含模态的内容，而不是标准布局。`abp-modal`是渲染模态对话框 HTML 的主要标签助手。`abp-modal-header`、`abp-modal-body`和`abp-modal-footer`是模态的主要部分，并且有不同的选项。在这个例子中，模态体非常简单；它只是在模型上显示`Message`。

我们已经创建了模态，但我们应该创建一种打开它的方法。ABP 在 JavaScript 端提供了 `ModalManager` API 来控制模态。在这里，我们需要在想要打开模态的页面上创建一个 `ModalManager` 对象：

```cs
var simpleModal = new abp.ModalManager({
    viewUrl: '/MySimpleModal'
});
```

`abp.ModalManager` 有几个选项，但最基本的是 `viewUrl`，它指示模态内容将被加载的 URL。一旦我们有一个 `ModalManager` 实例，我们就可以调用它的 `open` 方法来打开模态：

```cs
$(function (){
    $('#Button1').click(function (){
        simpleModal.open();
    });
});
```

此示例假设页面上有一个 ID 为 `Button1` 的按钮。当用户点击按钮时，我们将打开模态。以下截图显示了打开的模态：

![图 12.13 – 一个简单的模态对话框](img/Figure_12.13_B17287.jpg)

![图 12.13 – 一个简单的模态对话框](img/Figure_12.13_B17287.jpg)

图 12.13 – 一个简单的模态对话框

通常，我们在模态中创建动态内容，因此需要在打开模态对话框时传递一些参数。为此，您可以将一个包含模态参数的对象传递给 `open` 方法，如下例所示：

```cs
simpleModal.open({
    productId: 42
});
```

在这里，我们向模态传递了一个 `productId` 参数，因此它可能会显示给定产品的详细信息。您可以将相同的参数添加到 `MySimpleModalModel` 类的 `OnGet` 方法中，以获取值并在方法内部进行处理：

```cs
public void OnGet(int productId)
{
    ...
}
```

您可以从数据库中获取产品信息，并在模态体中渲染产品详情。

在下一节中，我们将学习如何将表单放置在模态中，以从用户那里获取数据。

## 在模态中处理表单

模态被广泛用于向用户显示表单。ABP 的 `ModalManager` API 优雅地为您处理一些常见任务：

+   它将焦点放在表单的第一个输入上。

+   当您按下 *Enter* 键或点击 **保存** 按钮时，它会触发验证检查。除非表单完全有效，否则不允许提交表单。

+   它通过 AJAX 请求提交表单，禁用模态按钮，并在保存操作完成前显示进度图标。

+   如果您已输入一些数据并点击 **取消** 按钮或关闭模态，它会警告您有关未保存的更改。

假设我们想要显示一个用于创建新电影的模态对话框，并且我们已经创建了一个名为 `ModalWithForm.cshtml` 的新 Razor 页面。代码隐藏文件与我们在 *实现 AJAX 表单* 部分中看到的内容类似：

```cs
public class ModalWithForm : AbpPageModel
{
    [BindProperty]
    public MovieViewModel Movie { get; set; }

    public void OnGet()
    {
        Movie = new MovieViewModel();
    }
    public async Task<IActionResult> OnPostAsync()
    {
        ValidateModel();
        //TODO: Create a new movie
        return NoContent();
    }
}
```

`OnPostAsync` 方法首先验证用户输入。如果表单无效，将抛出异常，并由 ABP 框架在服务器端和客户端处理。您可以向客户端返回一个响应，但在此示例中，我们返回一个 `NoContent` 响应。

由于我们混合了表单和模态，模态的视图侧略有不同：

```cs
@page
@using Volo.Abp.AspNetCore.Mvc.UI.Bootstrap.TagHelpers.Modal
@model MvcDemo.Web.Pages.ModalWithForm
@{
    Layout = null;
}
<form method="post" asp-page="/ModalWithForm">
    <abp-modal>
        <abp-modal-header title="Create new movie">
        </abp-modal-header>
        <abp-modal-body>
            <abp-input asp-for="Movie.Name" />
            <abp-select asp-for="Movie.Genre" />
            <abp-input asp-for="Movie.Description" />
            <abp-input asp-for="Movie.Price" />
            <abp-input asp-for="Movie.ReleaseDate" />
            <abp-input asp-for="Movie.PreOrder" />
        </abp-modal-body>
        <abp-modal-footer buttons="@(
            AbpModalButtons.Cancel|AbpModalButtons.Save)">
        </abp-modal-footer>
    </abp-modal>
</form>
```

`abp-modal` 标签被 `form` 元素包裹。我们不会将 `form` 标签放在 `abp-modal-body` 元素内部，因为 `form`。因此，作为解决方案，我们将 `form` 作为此视图的最高级元素放置。其余的代码块应该很熟悉；我们使用 ABP 输入标签助手来渲染表单元素。

现在，我们可以在我们的 JavaScript 代码中打开模态：

```cs
var newMovieModal = new abp.ModalManager({
    viewUrl: '/ModalWithForm'
});
$(function (){
    $('#Button2').click(function (){
        newMovieModal.open();
    });
});
```

打开的对话框如下截图所示：

![图 12.14 – 模态内的表单](img/Figure_12.14_B17287.jpg)

图 12.14 – 模态内的表单

在模态内也可以使用 `abp-dynamic-form` 标签助手。我们可以像这样重写模态的视图：

```cs
<abp-dynamic-form abp-model="Movie" 
    asp-page="ModalWithForm">
    <abp-modal>
        <abp-modal-header title="Create new movie!">
        </abp-modal-header>
        <abp-modal-body>
            <abp-form-content/>
        </abp-modal-body>
        <abp-modal-footer buttons="@(
            AbpModalButtons.Cancel|AbpModalButtons.Save)">
        </abp-modal-footer>
    </abp-modal>
</abp-dynamic-form>
```

在这里，我像在上一节中一样将 `abp-modal` 包裹在 `abp-dynamic-form` 元素中。这个示例的主要点是我在 `abp-modal-body` 元素中使用了 `<abp-form-content/>` 标签助手。`abp-form-content` 是一个可选的标签助手，用于将 `abp-dynamic-form` 标签助手的表单输入放置在所需的位置。

通常，你希望在模态表单保存后采取行动。为此，你可以将回调函数注册到 `ModalManager` 的 `onResult` 事件，如下面的代码块所示：

```cs
newMovieModal.onResult(function (e, data){
    console.log(data.responseText);
});
```

如果服务器发送任何结果，`data.responseText` 将是数据。例如，你可以从 `OnPostAsync` 方法返回一个 `Content` 响应，如下面的示例所示：

```cs
public async Task<IActionResult> OnPostAsync()
{
    ...
    return Content("42");
}
```

ABP 简化了所有这些常见任务。否则，你将需要编写大量的样板代码。

在下一节中，我们将学习如何向我们的模态对话框添加客户端逻辑。

## 为模态添加 JavaScript

如果你的模态需要一些高级客户端逻辑，你可能需要为你的模态编写一些自定义 JavaScript 代码。你可以在打开模态的页面中编写你的 JavaScript 代码，但这不是非常模块化和可重用的。最好将你的模态 JavaScript 代码写在单独的文件中，理想情况下靠近模态的 `.cshtml` 文件（记住 ABP 允许你将 JavaScript 文件放在 `Pages` 文件夹下）。

为了此，我们可以创建一个新的 JavaScript 文件并在 `abp.modals` 命名空间中定义一个函数，如下面的代码所示：

```cs
abp.modals.MovieCreation = function () {
     this.initModal = function(modalManager, args) {
        var $modal = modalManager.getModal();
        var preOrderCheckbox =
            $modal.find('input[name="Movie.PreOrder"]');
        preOrderCheckbox.change(function(){
            if (this.checked){
               alert('checked pre-order!'); 
            }
        });
        console.log('initialized the modal...');
    }
};
```

一旦我们创建了这样的 JavaScript 类，我们可以在创建 `ModalManager` 实例时将其与模态关联起来：

```cs
var newMovieModal = new abp.ModalManager({
    viewUrl: '/ModalWithForm',
    modalClass: 'MovieCreation'
});
```

`ModalManager` 在每次打开模态时都会创建 `abp.modals.MovieCreation` 类的新实例，如果你定义了 `initModal` 函数，它将调用该函数。`initModal` 函数接受两个参数。第一个参数是与模态关联的 `ModalManager` 实例，这样你就可以使用它的函数。第二个参数是你打开模态时传递给 `open` 函数的参数。

`initModal` 函数是准备模态内容和将一些回调注册到模态组件事件的完美位置。在先前的示例中，我获取了模态实例和一个 JQuery 对象，找到了 `Movie.PreOrder` 复选框，并注册了其 `change` 回调，以便在用户勾选它时得到通知。

这个示例仍然不起作用，因为我们还没有将 JavaScript 文件添加到页面中。有两种方法可以将它添加到页面中：

+   我们可以使用 `abp-script` 标签将模态的 JavaScript 文件包含在我们打开模态的页面中。

+   我们可以设置 `ModalManager` 以使其懒加载 JavaScript 文件。

第一个选项很简单——只需在你想使用模态的页面中包含以下行：

```cs
<abp-script src="img/ModalWithForm.cshtml.js" />
```

如果我们想懒加载模态的脚本，我们可以这样配置`ModalManager`：

```cs
var newMovieModal = new abp.ModalManager({
    viewUrl: '/ModalWithForm',
    scriptUrl: '/Pages/ModalWithForm.cshtml.js',
    modalClass: 'MovieCreation'
});
```

在这里，我添加了`scriptUrl`选项作为模态的 JavaScript 文件的 URL。`ModalManager`在第一次打开模态时懒加载 JavaScript 文件。如果你第二次打开模态（不刷新整个页面），则不会再次加载脚本。

在本节中，我们学习了如何处理表单、验证和模态。它们是典型 Web 应用的必要部分。在下一节中，我们将了解一些在每项应用中都需要的有用 JavaScript API。

# 使用 JavaScript API

在本节中，我们将探索 ABP 框架的一些有用的客户端 API。其中一些 API 提供了使用服务器端定义的功能（如身份验证和本地化）的简单方法，而其他 API 则提供了常见 UI 模式（如消息框和通知）的解决方案。

所有客户端 JavaScript API 都是声明在`abp`命名空间下的全局对象和函数。让我们从在你的 JavaScript 代码中访问当前用户信息开始。

## 访问当前用户

我们在服务器端使用`ICurrentUser`服务来获取当前登录用户的信息。在 JavaScript 代码中，我们可以使用全局的`abp.currentUser`对象，如下所示：

```cs
var userId = abp.currentUser.id;
var userName = abp.currentUser. userName;
```

通过这样做，我们可以获取用户的 ID 和用户名。以下 JSON 对象是`abp.currentUser`对象的示例：

```cs
{
  isAuthenticated: true,
  id: "813108d7-7108-4ab2-b828-f3c28bbcd8e0",
  tenantId: null,
  userName: "john",
  name: "John",
  surName: "Nash",
  email: "john.nash@abp.io",
  emailVerified: true,
  phoneNumber: "+901112223342",
  phoneNumberVerified: true,
  roles: ["moderator","manager"]
}
```

如果当前用户尚未登录，所有这些值都将为`null`或`false`，正如你所期望的那样。`abp.currentUser`对象提供了一种简单的方法来获取有关当前用户的信息。在下一节中，我们将学习如何检查当前用户的权限。

## 检查用户权限

ABP 的授权和权限管理系统是一种强大的方式，可以在运行时定义权限并检查当前用户的权限。使用`abp.auth` API 在你的 JavaScript 代码中检查这些权限非常容易。

以下示例检查当前用户是否有`DeleteProduct`权限：

```cs
if (abp.auth.isGranted('DeleteProduct')) {
  // TODO: Delete the product
} else {
  abp.message.warn("You don't have permission to delete
                    products!");
}
```

`abp.auth.isGranted`如果当前用户已授予权限或策略，则返回`true`。如果用户没有权限，我们将使用 ABP 消息 API 显示警告消息，这将在本章后面的*显示消息框*部分进行解释。

虽然这些 API 很少需要，但在你需要获取所有可用权限/策略的列表时，可以使用`abp.auth.policies`对象；如果你需要获取当前用户所有已授予权限/策略的列表，可以使用`abp.auth.grantedPolicies`对象。

基于权限隐藏 UI 部分

客户端权限检查的一个典型用例是根据用户的权限隐藏一些 UI 部分（例如操作按钮）。虽然`abp.auth` API 提供了动态的方式来做到这一点，但我建议尽可能在你的 Razor Pages/views 中使用标准的`IAuthorizationService`来条件性地渲染 UI 元素。

注意，在客户端检查权限只是为了用户体验，并不能保证安全性。你应该始终在服务器端检查相同的权限。

在下一节中，我们将学习如何在多租户应用程序中检查当前租户的功能权限。

## 检查租户功能

功能系统用于根据当前租户限制应用程序的功能/特性。我们将在*第十六章*中探索 ABP 的多租户基础设施，*实现多租户*。然而，我们将在这里介绍如何检查 ASP.NET Core MVC/Razor Pages UI 的租户功能。

`abp.features` API 用于检查当前租户的功能值。假设我们有一个从 Mailchimp（一个云电子邮件营销平台）导入电子邮件列表的功能，并且我们已经定义了一个名为`MailchimpImport`的功能。我们可以轻松地检查当前租户是否启用了该功能：

```cs
if (abp.features.isEnabled('MailchimpImport'))
{
  // TODO: Import from Mailchimp
}
```

`abp.features.isEnabled`仅在给定的功能值是`true`时返回`true`。ABP 的功能系统还允许你定义非布尔功能。在这种情况下，你可以使用`abp.features.get(…)`函数来获取当前租户给定功能的值。

在客户端检查功能使得执行动态客户端逻辑变得容易，但请记住，为了确保应用程序的安全，也要在服务器端检查功能。

在下一节中，我们将继续在你的 JavaScript 代码中使用本地化系统。

## 本地化字符串

ABP 本地化系统的一个强大之处在于你可以在客户端重用相同的本地化字符串。这样，你就不必在 JavaScript 代码中处理另一种本地化库。

`abp.localization` API 在你的 JavaScript 代码中可用，以帮助你利用本地化系统。让我们从最简单的情况开始：

```cs
var str = abp.localization.localize('HelloWorld');
```

在这种用法中，`localize`函数接受一个本地化键，并根据当前语言返回本地化值。它使用默认的本地化资源。如果你需要，你可以将本地化资源作为第二个参数指定：

```cs
var str = abp.localization.localize('HelloWorld', 'MyResource');
```

在这里，我们已将`MyResource`指定为本地化资源。如果你想从同一资源中本地化大量字符串，有一个更简短的方法来做这件事：

```cs
var localizer = abp.localization.getResource('MyResource');
var str = localizer('HelloWorld');
```

在这里，你可以使用`localizer`对象从相同的资源获取文本。

JavaScript 本地化 API 将相同的回退逻辑应用于服务器端 API；如果找不到本地化值，它将返回给定的键。

如果本地化字符串包含占位符，你可以将占位符值作为参数传递。假设我们在本地化 JSON 文件中有以下条目：

```cs
"GreetingMessage": "Hello {0}!"
```

我们可以将参数传递给 `localizer` 或 `abp.localization.localize` 函数，如下例所示：

```cs
var str = abp.localization.localize('GreetingMessage', 'John');
```

对于此示例，结果 `str` 值将是 `Hello John!`。如果你有多个占位符，你可以按相同顺序将值传递给 `localizer` 函数。

除了本地化文本外，你可能还需要知道当前的文化和语言，以便你可以采取额外的行动。`abp.localization.currentCulture` 对象包含有关当前语言和文化的详细信息。除了当前语言外，`abp.localization.languages` 值是当前应用程序中所有可用语言的数组。大多数时候，你不会直接使用这些 API，因为你所使用的主题负责向用户显示语言列表并允许你在它们之间切换。然而，了解你可以在需要时访问语言数据是很好的。

到目前为止，你已经学习了如何在客户端使用一些 ABP 服务器端功能。在下一节中，你将学习如何向用户显示消息和确认框。

## 显示消息框

在应用程序中向用户显示阻止消息框以通知他们发生的重要事情是非常常见的。在本节中，你将学习如何在应用程序中显示漂亮的消息框和确认对话框。

`abp.message` API 用于显示消息框，以便轻松通知用户。有四种类型的消息框：

+   `abp.message.info`：显示信息消息

+   `abp.message.success`：显示成功消息

+   `abp.message.warn`：显示警告消息

+   `abp.message.error`：显示错误消息

让我们看看以下示例：

```cs
abp.message.success('Your changes have been successfully
                     saved!', 'Congratulations');
```

在此示例中，我使用了 `success` 函数来显示成功消息。第一个参数是消息文本，而可选的第二个参数是消息标题。此示例的结果如下截图所示：

![图 12.15 – 成功消息框![图片](img/Figure_12.15_B17287.jpg)

图 12.15 – 成功消息框

消息框被阻止，这意味着页面被阻止（不可点击），直到用户点击**确定**按钮。

另一种类型的消息框用于确认目的。`abp.message.confirm` 函数显示一些对话框以从用户那里获取响应：

```cs
abp.message.confirm('Are you sure to delete this product?')
.then(function(confirmed){
  if(confirmed){
    // TODO: Delete the product!
  }
});
```

`confirm` 函数返回一个承诺，因此我们可以将其与 `then` 回调链式调用，以便在用户通过接受或取消对话框关闭后执行一些代码。以下截图显示了为此示例创建的确认对话框：

![图 12.16 – 确认对话框![图片](img/Figure_12.16_B17287.jpg)

图 12.16 – 确认对话框

消息框是吸引用户注意的好方法。然而，还有另一种方法可以做到这一点，我们将在下一节中看到。

## 显示通知

通知是非阻塞的方式，用于通知用户某些事件。它们显示在屏幕的右下角，并在几秒钟后自动消失。就像消息框一样，有四种类型的通知：

+   `abp.notify.info`: 显示信息通知

+   `abp.notify.success`: 显示成功通知

+   `abp.notify.warn`: 显示警告通知

+   `abp.notify.error`: 显示错误通知

以下示例展示了信息通知：

```cs
abp.notify.info(
    'The product has been successfully deleted.',
    'Deleted the Product'
);
```

第二个参数是通知标题，是可选的。以下截图显示了此示例代码的结果：

![图 12.17 – 通知消息](img/Figure_12.17_B17287.jpg)

图 12.17 – 通知消息

使用通知 API，我们正在关闭 JavaScript API。

在这里，我介绍了最常用的 API。然而，您可以在 JavaScript 代码中使用更多 API，所有这些您都可以通过阅读 ABP 框架文档来了解：[`docs.abp.io/en/abp/latest/UI/AspNetCore/JavaScript-API/Index`](https://docs.abp.io/en/abp/latest/UI/AspNetCore/JavaScript-API/Index)。在下一节中，我们将学习如何从 JavaScript 代码中消费服务器端 API。

# 消费 HTTP API

您可以使用任何工具或技术从 JavaScript 代码中消费 HTTP API。然而，ABP 提供了以下作为完全集成解决方案的方法：

+   您可以将`abp.ajax` API 作为`jQuery.ajax` API 的扩展使用。

+   您可以使用动态 JavaScript 客户端代理来调用服务器端 API，就像使用 JavaScript 函数一样。

+   您可以在开发时生成静态 JavaScript 客户端代理。

让我们从第一个开始 – `abp.ajax` API。

## 使用 abp.ajax API

`abp.ajax` API 是标准`jQuery.ajax` API 的包装器。它在发生错误的情况下自动处理所有错误，并向用户显示本地化消息。它还向 HTTP 头中添加了反伪造令牌，以满足服务器端的**跨站请求伪造**（**CSRF**）保护。

以下示例使用`abp.ajax` API 从服务器获取用户列表：

```cs
abp.ajax({
  type: 'GET',
  url: '/api/identity/users'
}).then(function(result){
  // TODO: process the result
});
```

在此示例中，我们已将`GET`指定为请求的`type`。您可以指定所有`jQuery.ajax`（或`$.ajax`）的标准选项来覆盖默认值。`abp.ajax`返回一个承诺对象，因此我们可以添加`then`回调来处理服务器发送的结果。我们还可以使用`catch`回调来处理错误，以及使用`always`回调在请求结束时执行操作。

以下示例展示了如何手动处理错误：

```cs
abp.ajax({
  type: 'GET',
  url: '/api/identity/users',
  abpHandleError: false
}).then(function(result){
  // TODO: process the result
}).catch(function(){
  abp.message.error("request failed :(");
});
```

在这里，我在 `then` 函数之后添加了一个 `catch` 回调函数。你可以在那里执行你的错误逻辑。我还指定了 `abpHandleError: false` 选项来禁用 ABP 的自动错误处理逻辑。否则，ABP 将处理错误并向用户显示错误消息。

`abp.ajax` 是一个低级 API。你通常使用动态或静态客户端代理来消费自己的 HTTP API。

## 使用动态客户端代理

如果你应用了示例应用程序的 *第三章* 步骤-by-步骤应用程序开发，你应该已经使用了动态 JavaScript 客户端代理系统。ABP 框架在运行时生成 JavaScript 函数，以便轻松消费应用程序的所有 HTTP API。

下面的代码块显示了在 *第三章* 步骤-by-步骤应用程序开发 中定义的两个 `IProductAppService` 样式方法：

```cs
namespace ProductManagement.Products
{
    public interface IProductAppService :
        IApplicationService
    {
        Task CreateAsync(CreateUpdateProductDto input);
        Task<ProductDto> GetAsync(Guid id);
    }
}
```

所有这些方法都在客户端的相同命名空间中可用。例如，我们可以通过其 ID 获取一个产品，如下面的代码块所示：

```cs
productManagement.products.product
  .get('1b8517c8-2c08-5016-bca8-39fef5c4f817')
  .then(function (result) {
    console.log(result);
  });
```

`productManagement.products` 是 C# 代码中 `ProductManagement.Products` 命名空间的驼峰式等效。`product` 是 `IProductAppService` 的传统名称。`I` 前缀和 `AppService` 后缀已被移除，剩余的名称被转换为驼峰式。然后，我们可以使用不带 `Async` 后缀的驼峰式方法名称。因此，`GetAsync` 方法在 JavaScript 代码中用作 `get` 函数。`get` 函数接受与 C# 方法相同的参数。它返回一个 `Deferred` 对象，这样我们就可以使用 `then`、`catch` 或 `always` 回调函数来链式调用，类似于 `abp.ajax` API 可以做到的那样。它内部使用 `abp.ajax` API。在这个例子中，`then` 函数的 `result` 参数是服务器发送的 `ProductDto` 对象。

其他方法以类似的方式进行使用。例如，我们可以使用以下代码创建一个新产品：

```cs
productManagement.products.product.create({
  categoryId: '5f568193-91b2-17de-21f3-39fef5c4f808',
  name: 'My product',
  price: 42,
  isFreeCargo: true,
  releaseDate: '2023-05-24',
  stockState: 'PreOrder'
});
```

在这里，我们使用 JSON 对象格式传递了 `CreateUpdateProductDto` 对象。

在某些情况下，我们可能需要为 HTTP API 调用传递额外的 AJAX 选项。你可以将一个对象作为每个代理函数的最后一个参数传递：

```cs
productManagement.products.product.create({
  categoryId: '5f568193-91b2-17de-21f3-39fef5c4f808',
  name: 'My product',
  //...other values
}, {
  url: 'https://localhost:21322/api/my-custom-url'
  headers: {
    'MyHeader': 'MyValue'
  }
});
```

在这里，我传递了一个对象来更改 URL 并向请求添加一个自定义头。你可以参考 jQuery 的文档 ([`api.jquery.com/jquery.ajax/`](https://api.jquery.com/jquery.ajax/)) 了解所有可用选项。

动态 JavaScript 客户端代理函数由应用程序的 `/Abp/ServiceProxyScript` 端点在运行时生成。这个 URL 由主题添加到布局中，这样你就可以直接在你的页面中使用任何代理函数，而无需导入任何脚本。

在下一节中，你将了解消费 HTTP API 的另一种替代方法。

## 使用静态客户端代理

与在运行时生成的动态客户端代理不同，静态代理是在开发时生成的。我们可以使用 ABP CLI 来生成代理脚本文件。

首先，我们需要运行提供 HTTP API 的应用程序，因为 API 端点数据是从服务器请求的。然后，我们可以使用 `generate-proxy` 命令，如下例所示：

```cs
abp generate-proxy -t js -u https://localhost:44349
```

`generate-proxy` 命令可以接受以下参数：

+   `-t`（必需）：代理的类型。在这里我们使用 `js` 表示 JavaScript。

+   `-u`（必需）：API 端点的根 URL。

+   `-m`（可选）：生成代理的模块名称。默认值为 `app`，用于生成应用程序的代理。在模块化应用程序中，你可以在此处指定模块名称。

静态 JavaScript 代理在 `wwwroot/client-proxies` 文件夹下生成，如下截图所示：

![图 12.18 – 静态 JavaScript 代理文件![img/Figure_12.18_B17287.jpg](img/Figure_12.18_B17287.jpg)

图 12.18 – 静态 JavaScript 代理文件

然后，你可以将代理脚本文件导入任何页面，并像使用动态代理一样使用静态代理函数。

当你使用静态代理时，不需要动态代理。默认情况下，ABP 为你的应用程序创建动态代理。你可以配置 `DynamicJavaScriptProxyOptions` 来禁用它，如下例所示：

```cs
Configure<DynamicJavaScriptProxyOptions>(options => {
    options.EnabledModules.Remove("app");
});
```

`EnabledModules` 列表默认包含 `app`。如果你正在构建模块化应用程序并希望为你的模块启用动态 JavaScript 代理，你需要将其显式添加到 `EnabledModules` 列表中。

# 摘要

在本章中，我们介绍了 ABP 框架 MVC/Razor Pages UI 的基本设计要点和基本功能。

主题系统允许你构建与主题/样式无关的模块和应用程序，并轻松地在 UI 主题之间切换。它是通过定义一组基本库和标准布局来实现这一点的。

你还学习了捆绑和压缩系统，它涵盖了在应用程序中导入和使用客户端依赖项的整个开发周期，并在生产环境中优化资源使用。

ABP 使得使用标签辅助器和预定义约定创建表单以及实现验证和本地化变得简单。你还学习了如何将标准表单转换为 AJAX 提交的表单。

我们还介绍了一些可以用于客户端并利用 ABP 功能的 JavaScript API，例如授权和本地化，以及轻松显示美观的消息框和通知。

最后，你了解了从 JavaScript 代码中消费 HTTP API 的替代方法。

在下一章中，你将学习关于 ABP 框架的 Blazor UI，使用 C# 而不是 JavaScript 来构建交互式 Web UI。
