# 9

# 分享代码和资源

在整本书中，我们一直在构建一个可以在许多不同的托管模型中运行的项目。如果我们想在将来进一步切换技术，或者像我们在工作中做的那样，在客户门户和我们的内部**客户关系管理**（**CRM**）系统之间共享组件，这是一个构建项目的绝佳方式。

总是考虑我们正在构建的组件中是否可能有一个可共享的部分；这样，我们可以重用它，如果我们向组件中添加了某些内容，我们就可以为所有组件获得这种好处。

但这不仅仅是我们自己项目内部共享组件的问题。如果我们想创建一个可以与其他部门共享的库，或者甚至是一个与世界共享组件的开源项目，那会怎样？

在本章中，我们将探讨一些我们在共享组件时已经使用的内容，以及共享 CSS 和其他静态文件。

在本章中，我们将涵盖以下主题：

+   添加静态文件

+   CSS 隔离

# 技术要求

确保您已经遵循了前面的章节，或者以`Chapter08`文件夹作为起点。

您可以在[`github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter09`](https://github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter09)找到本章结果的源代码。

如果您使用 GitHub 上的代码跳转到本章，请确保您已在设置文件中添加了`Auth0`账户信息。您可以在*第八章*，*身份验证和授权*中找到说明。

# 添加静态文件

Blazor 可以使用静态文件，如图片、CSS 和 JavaScript。如果我们把我们的文件放在`wwwroot`文件夹中，它们将自动暴露给互联网，并可以从我们网站的根目录访问。Blazor 的好处在于，我们也可以用库来做同样的事情；在库内分发静态文件非常简单。

在工作中，我们在所有的 Blazor 项目中共享组件，共享库也可以依赖于其他库。通过共享组件和构建我们自己的组件（有时是在其他库之上），我们确保整个网站具有相同的视觉和感觉。我们还共享静态内容，如图片和 CSS，这使得如果我们需要更改某些内容并且希望所有网站都受到影响时，变得简单快捷。

要链接到另一个库/程序集的资源，我们可以使用`_content`文件夹。

看一下这个例子：

```cs
<link rel="stylesheet" href="_content/SharedComponents/MyBlogStyle.min.css" /> 
```

HTML 的`link`标签、`rel`和`href`是普通的 HTML 标签和属性，但添加以`_content`开头的 URL 告诉我们，我们想要访问的内容位于另一个库中。在我们的例子中，库（程序集名称）是`SharedComponents`，后面是我们想要访问的文件，该文件存储在我们的库中的`wwwroot`文件夹中。

Blazor 最终只是 HTML，而 HTML 可以使用 CSS 进行样式化。正如之前提到的，Blazor 模板默认使用 Bootstrap，我们也将继续使用它。

有一个优秀的网站，提供了易于使用的 Bootstrap 主题，可供下载，网址为[`bootswatch.com/`](https://bootswatch.com/)。

我喜欢 Darkly 主题，所以我们将使用这个，但你可以自由地稍后尝试其他主题。

## 选择框架

我经常被问到如何样式化 Blazor 应用程序，事实是你可以使用你习惯的所有工具。最终，Blazor 会输出 HTML。我们可以使用许多语言和框架来编写我们的 CSS。

我们可以使用 CSS、**语法优美的样式表**（**SASS**）和**更简洁的 CSS**（**LESS**）。只要输出是 CSS，我们就可以使用它。

在本章中，我们将继续使用 Bootstrap 和 CSS。SASS 和 LESS 超出了本书的范围。

Tailwind 是 Blazor 的一个流行框架，绝对可以与 Blazor 一起使用。Tailwind 非常注重组件，并且需要一些配置才能开始，但如果它是你熟悉并喜欢的，你可以与 Blazor 一起使用它。

## 添加新的样式

许多模板以 Bootstrap 为基础，所以如果你在寻找网站的设计，使用基于 Bootstrap 的模板将是一个易于实现的方案。

Bootstrap 的问题（以及为什么有些人不喜欢它）在于许多网站使用 Bootstrap，导致“所有网站看起来都一样”。如果我们正在构建一个**业务线**（**LOB**），这可能是个好事，但如果我们试图创新，这可能是个坏事。Bootstrap 在下载时也相当大，这也是反对它的一个论点。

本章是关于让我们的博客看起来更美观，所以我们将继续使用 Bootstrap，但我们应该知道，如果我们使用其他东西来处理我们的 CSS，它也会与 Blazor 一起工作。

这些模板网站之一是`Bootswatch`，它为我们提供了从传统 Bootstrap 主题中的一些美好变化：

1.  导航到[`bootswatch.com/darkly/`](https://bootswatch.com/darkly/)。

1.  在顶部菜单`Darkly`中，有一些链接。下载`bootstrap.min.css`。

1.  在`SharedComponents`项目中，在`wwwroot`文件夹中，添加`bootstrap.min.css`文件。

我们可以为我们的网站添加所有必要的 CSS。

## 添加 CSS

现在，是时候为我们网站添加新的样式了：

1.  在`BlazorWebApp`项目中，打开`Components/App.razor`文件。

1.  定位到这一行：

    ```cs
    <link rel="stylesheet" href="bootstrap/bootstrap.min.css" /> 
    ```

    将前面的行替换为以下内容：

    ```cs
    <link rel="stylesheet" href="_content/SharedComponents/bootstrap.min.css" /> 
    ```

1.  通过按*Ctrl* + *F5*来运行项目。

太好了！我们的 Blazor 项目现在已更新为使用新的样式。主色调现在应该是深色，但仍有一些工作要做。

## 使管理界面更易于使用

让我们现在进一步清理。我们刚刚开始实现管理功能，所以让我们让它更容易访问。左侧的菜单不再需要，所以让我们将其改为仅在您是管理员时才可见：

1.  打开`Components/Layout/MainLayout.razor`文件，并将`AuthorizeView`放在`sidebar` `div`周围，如下所示：

    ```cs
    <AuthorizeView Roles="Administrator">
        <div class="sidebar">
            <NavMenu />
        </div>
    </AuthorizeView> 
    ```

在这种情况下，我们没有指定`Authorized`或`NotAuthorized`。默认行为是`Authorized`，所以如果我们只寻找授权状态，我们不需要通过名称指定它。

启动项目以查看其效果。如果我们未登录，菜单不应显示。

现在，我们需要让菜单看起来更好。尽管计数器点击起来很有趣，但它对我们博客来说意义不大。

## 使菜单更实用

我们应该将链接替换为指向我们管理页面的链接：

1.  在`BlazorWebApp`项目中，打开`Components/Layout/Navmenu.razor`文件。

    编辑代码，使其看起来像这样：

    ```cs
    <div class="top-row ps-3 navbar navbar-dark">
        <div class="container-fluid">
            <a class="navbar-brand" href="">MyBlog Admin</a>
        </div>
    </div>
    <input type="checkbox" title="Navigation menu" class="navbar-toggler" />
    <div class="nav-scrollable" onclick="document.querySelector('.navbar-toggler').click()">
        <nav class="flex-column">
            <div class="nav-item px-3">
                <NavLink class="nav-link" href="" Match="NavLinkMatch.All">
                    <span class="bi bi-house-door-fill-nav-menu" aria-hidden="true"></span> Home
                </NavLink>
            </div>
            <div class="nav-item px-3">
                <NavLink class="nav-link" href="Admin/Blogposts">
                    <span class="bi bi-plus-square-fill-nav-menu" aria-hidden="true"></span> Blog posts
                </NavLink>
            </div>
            <div class="nav-item px-3">
                <NavLink class="nav-link" href="Admin/Tags">
                    <span class="bi bi-plus-square-fill-nav-menu" aria-hidden="true"></span> Tags
                </NavLink>
            </div>
            <div class="nav-item px-3">
                <NavLink class="nav-link" href="Admin/Categories">
                    <span class="bi bi-plus-square-fill-nav-menu" aria-hidden="true"></span> Categories
                </NavLink>
            </div>
        </nav>
    </div> 
    ```

1.  如果我们启动项目并调整屏幕大小，我们会注意到菜单在大屏幕上显示，但在小屏幕上隐藏。值得注意的是，`NavMenu`不包含代码，所以菜单的隐藏和显示完全依赖于 CSS。

太好了！我们的博客看起来更像一个博客了，但我们还可以做得更多！

## 使博客看起来更像一个博客

管理界面已经完成（至少，目前是这样），我们应该专注于我们博客的首页。首页应该包含博客文章的标题和一些描述：

1.  在`SharedComponents`项目中，打开`Pages/Home.razor`文件。

1.  在文件顶部添加一个`using`语句用于`Markdig`：

    ```cs
    @using Markdig; 
    ```

1.  添加一个`OnInitializedAsync`方法来处理`Markdig`管道的实例化（这是我们在`Post.razor`文件中有的相同代码）：

    ```cs
    MarkdownPipeline pipeline;
    protected override Task OnInitializedAsync()
    {
        pipeline = new MarkdownPipelineBuilder()
                    .UseEmojiAndSmiley()
                    .Build();
        return base.OnInitializedAsync();
    } 
    ```

1.  在`Virtualize`组件内部，将内容（`RenderFragment`）更改为以下内容：

    ```cs
     <article>
            <h2>@p.Title</h2>
            @((MarkupString)Markdig.Markdown.ToHtml(new string(p.Text.Take(100).ToArray()), pipeline))
            <a href="/Post/@p.Id">Read more</a>
        </article> 
    ```

1.  此外，删除`<ul>`标签。

现在，使用*Ctrl* + *F5*运行项目，查看我们新的首页。我们的博客开始成形，但我们还有工作要做。

# CSS 隔离

在.NET 5 中，Microsoft 添加了一个名为隔离 CSS 的功能。许多其他框架也有类似的功能。这个想法是为一个组件编写特定的 CSS。当然，好处是我们创建的 CSS 不会影响其他任何组件。

Blazor 的模板使用隔离的 CSS 为`Components/Layout/MainLayout.razor`和`NavMenu.Razor`。如果我们展开`MainLayout.razor`，我们会看到一个名为`MainLayout.razor.css`的文件。这个树状结构是通过使用相同的命名方式实现的。

我们还可以通过添加一个名为`MainLayout.razor.scss`的文件来使用 SASS。重要的是，我们添加的文件应该生成一个名为`MainLayout.razor.css`的文件，以便编译器可以识别。

这个命名约定将确保重写 CSS 和 HTML 输出。

CSS 具有以下命名约定：

```cs
main {
    flex: 1;
} 
```

它将被重写如下：

```cs
main[b-bfl5h5967n] {
    flex: 1;
} 
```

这意味着元素需要有一个名为 `b-bfl5h5967n` 的属性（在这个例子中），以便应用该样式。这是一个为该组件随机生成的字符串。

在 `MainLayout` 组件内部包含 `CSS` 标签的 `div` 标签将被输出如下：

```cs
<main b-bfl5h5967n> 
```

为了让所有这些发生，我们还需要有一个指向 CSS 的链接（由模板提供），它看起来是这样的：

```cs
<link href="{Assemblyname}.styles.css" rel="stylesheet"> 
```

这对于组件库来说很有用。在我们的共享库（`NavMenu` 和 `MainLayout`）中，组件的 CSS 被包含在 `{Assemblyname}.styles.css` 文件中。

我们不需要做任何额外的事情来包含我们的共享 CSS。如果我们为任何人创建库，我们应该考虑在组件需要一些 CSS 才能正确工作的情况下使用隔离的 CSS 方法。

如果我们从空模板开始创建 Blazor 项目，我们需要添加一个指向隔离 CSS 的链接。

这样，我们的用户就不需要添加对 CSS 的引用，并且没有风险在我们的 CSS 会破坏用户的 app（因为它被隔离）。重要的是，我们在需要的时候使用正确的方法。

假设我们正在创建一个具有非常特定样式的组件，这个样式只由该组件使用。在这种情况下，使用隔离的 CSS 是一个很好的选择，因为它更容易找到（紧挨着组件），并且我们可以使用 CSS 变量来设置颜色等。

我们在隔离 CSS 中对类似事物进行样式设计时应该小心，以免最终出现一堆不同的 CSS 文件来设计按钮等。

如前所述，隔离的 CSS 只影响组件内部的 HTML 标签，但如果我们组件内部有其他组件呢？

如果我们打开 `Component/Layout/NavMenu.razor.css`，我们可以看到，对于 `.nav-item` 样式，其中一些使用了 `::deep` 关键字；这意味着即使是子组件也应该受到这种样式的影响。

看看以下代码：

```cs
.nav-item ::deep a {…} 
```

它针对的是 `<a>` 标签，但 Razor 代码看起来是这样的：

```cs
<li class="nav-item px-3">
     <NavLink class="nav-link" href="Admin/Blogposts">
         <span class="oi oi-signpost" aria-hidden="true"></span> Blog posts
     </NavLink>
</li> 
```

是 `NavLink` 组件渲染了 `<a>` 标签；通过添加 `::deep`，我们表示我们希望将此样式应用于具有 `.nav-item` 类的所有元素以及该元素内部的所有 `<a>` 标签。

我们还需要了解的另一件事是 `::deep`；它确保共享属性的 ID（例如 `b-bfl5h5967n`），并且需要一个 HTML 标签来实现这一点。因此，如果我们有一个由其他组件组成的组件（完全不添加任何 HTML 标签），我们需要在内容周围添加一个 HTML 标签，以便 `::deep` 能够工作。

在我们总结本章内容之前，让我们再做一些事情。

让我们修复菜单的背景颜色：

1.  打开 `Components/Layout/MainLayout.razor.css`。

1.  查找 `.sidebar` 样式并将其替换为以下代码：

    ```cs
    .sidebar {
        background-image: linear-gradient(180deg, var(--bs-body-bg) 0%,var(--bs-gray-800) 70%);
    } 
    ```

1.  将 `.top-row` 样式替换为以下代码：

    ```cs
    .top-row {
        background-color: var(--bs-primary);
        justify-content: flex-end;
        height: 3.5rem;
        display: flex;
        align-items: center;
    } 
    ```

    我们替换了背景颜色并移除了边框。

1.  在 `.top-row ::deep a, .top-row ::deep .btn-link` 样式下，添加以下内容：

    ```cs
    color:white; 
    ```

现在，我们能够更好地看到登录/登出链接。

我们现在有一个工作的管理界面和一个看起来不错的网站。

# 摘要

在本章中，我们添加了共享 CSS。

我们看到了如何创建共享库（供他人使用）。这也是结构化我们内部项目的一个很好的方法（这样就可以轻松地从 Blazor Server 转换到 Blazor WebAssembly，或者反过来）。

如果你已经有了网站，你可以在共享库中构建你的 Blazor 组件，我们在整本书中都这样做过。

将组件作为你网站的一部分（使用 Blazor Server），你可以逐步开始使用 Blazor，直到你将整个网站转换完成。当这一切都完成时，你可以决定是否继续使用 Blazor Server（如我提到的，我们在我的工作场所使用 Blazor Server）或者转向 Blazor WebAssembly，或者像我们的项目一样两者都使用。

我们讨论了如何在我们的网站上使用 SASS 和 CSS，包括*常规* CSS 和*隔离* CSS。

在下一章中，我们将了解我们试图避免的一件事（至少，我是这样）——作为 Blazor 开发者——JavaScript。
