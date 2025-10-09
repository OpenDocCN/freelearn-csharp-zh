

# 从现有网站迁移或与之结合

在本章中，我们将探讨如何结合不同的技术和框架使用 Blazor。

如果我们已经有了一个网站呢？

当涉及到从现有网站迁移时，有多种选择；第一个问题是，我们是否想要从现有网站迁移，还是想要将其与新技术结合？

微软有让技术共存的历史，这正是本章的主题。

我们如何在 Blazor 网站中使用 Angular 和 React，或者如何将 Blazor 引入现有的 Angular 和 React 网站？

在本章中，我们将涵盖以下内容：

+   介绍 Web 组件

+   探索自定义元素

+   探索 Blazor 组件

+   将 Blazor 添加到 Angular 网站

+   将 Blazor 添加到 React 网站

+   将 Blazor 添加到 MVC/Razor Pages

+   将 Web 组件添加到 Blazor 网站

+   从 Web 表单迁移

结合技术可以非常有用，要么是因为我们一次无法转换整个网站，要么是因为其他技术更适合我们想要实现的目标。

话虽如此，我更喜欢在我的网站上使用一种技术，而不是将 Blazor 与 Angular 或 React 混合。但在迁移期间或如果我们的团队是混合的，混合也有其好处。

混合技术是有成本的，我们将在本章中探讨这一点。

在撰写本章、回顾 Angular 和 React 的过程中，我必须抓住机会说，我有多么喜欢 Razor 语法。React 是带有 HTML 标签的 JavaScript，Angular 有模板，我觉得很棒，让人联想到 Razor 语法的样子。

然而，涉及到的内容很多：几乎 300 MB 的 Node.js 模块、npm、TypeScript 和 webpack。好吧，列表很长。

我喜欢使用 Blazor，因为我不需要处理我刚才提到的所有内容。在我看来，Blazor 在这三个选项中拥有最好的语法。

# 技术要求

本章是一个参考章节，并且与本书的其他章节没有任何关联。

您可以在[`github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter15`](https://github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter15)找到本章示例的源代码。

# 介绍 Web 组件

要与 JavaScript 一起工作，无论是将 JavaScript 带入 Blazor 还是将 Blazor 带入 JavaScript，我们可以使用一种称为 Web 组件的技术。

Web 组件是一组 Web 平台 API，允许我们创建新的、自定义的、可重用的 HTML 标签。它们以封装的方式打包，我们可以像在 Blazor 中使用组件一样使用它们。

真正的好处是，我们可以在支持 HTML 的任何 JavaScript 库或框架中使用它们。

Web 组件建立在现有的 Web 标准（如 shadow DOM、ES 模块、HTML 模板和自定义元素）之上。

我们也将在 Blazor 中识别一些这些技术或它们的变体。Shadow DOM 与 Blazor 的渲染树相同，ES 模块是我们第十章中讨论的*JavaScript 互操作*中查看的 JavaScript 模块类型。

我们将在本章中探讨的技术是**自定义元素**。

# 探索自定义元素

要将 Blazor 引入现有的 Angular 或 React 网站，我们使用一个名为`CustomElements`的功能。它是在.NET 6 中作为一个实验性功能引入的，并从.NET 7 开始成为框架的一部分。

理想的情况是在 Blazor 中创建网站的部分，而不必完全迁移到 Blazor。

为了使这个功能正常工作，我们需要有一个 ASP.NET 后端或手动确保`_framework`文件可用。这样我们就可以提供 Blazor 框架文件。

运行`CustomElements`有两种方式；我们可以将其作为 Blazor WebAssembly 或作为 Blazor Server 运行。由于我们正在将 Blazor 添加到像 React 或 Angular 这样的客户端框架中，最相关的方法是将它作为 Blazor WebAssembly 运行。因此，这些第一部分中的示例将是针对 Blazor WebAssembly 的。

在 GitHub 仓库中，有一个名为`CustomElements`的文件夹，其中包含项目的代码，我们将从本章中看到示例代码。

值得注意的是，由于组件是在客户端提供和使用的，因此没有任何东西阻止我们（或对我们造成伤害的人）反编译代码（如果我们使用 WebAssembly）。这是所有框架的客户端开发者一直在处理的事情，但再次提一下也是值得的。

# 探索 Blazor 组件

我们需要尝试的第一件事是一个 Blazor 组件。我在一个名为`BlazorCustomElements`的 Blazor WebAssembly 项目中创建了一个`counter`组件。

默认模板包含了很多东西，而 repo 项目被简化到最基本，因此很容易理解。

组件与我们之前在书中看到的不同，它是一个带有设置计数器应增加多少的参数的`counter`组件。它看起来像这样：

```cs
<h1>Blazor counter</h1>
<p role="status">Current count: @currentCount</p>
<p>Increment amount: @IncrementAmount</p>
<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>
@code {
    private int currentCount = 0;
    [Parameter] public int IncrementAmount { get; set; } = 1;
    private void IncrementCount()
    {
        currentCount += IncrementAmount;
    }
} 
```

该项目还需要对 NuGet 包有一个引用：

```cs
Microsoft.AspNetCore.Components.CustomElements 
```

在`Program.cs`中，我们需要像这样注册组件/自定义元素：

```cs
builder.RootComponents.RegisterCustomElement<Counter>("my-blazor-counter"); 
```

Blazor 项目就到这里了。

现在是时候使用我们的自定义元素了。

# 将 Blazor 添加到 Angular 网站

让我们看看如何将 Blazor 添加到现有的 Angular 网站中。这个演示基于 Visual Studio 中的 Angular 和 ASP.NET Core 模板。

该文件夹被命名为`Angular`。

首先，我们需要对我们的 Blazor 库有一个引用。我将`BlazorCustomElement`项目作为引用添加到服务器项目中。

我们需要一个对`Microsoft.AspNetCore.Components.WebAssembly.Server NuGet`包的引用；这样我们就可以提供框架文件。

为了使我们的网站提供框架文件，我们需要将以下内容添加到`Program.cs`中：

```cs
app.UseBlazorFrameworkFiles(); 
```

默认情况下，当我们添加自定义元素时，Angular 会感到不满，因为它不识别该标签。为了解决这个问题，我们需要告诉 Angular 我们正在使用自定义元素。在`angularproject.client/src/app/app.module.ts`中添加以下内容：

```cs
import { CUSTOM_ELEMENTS_SCHEMA, NgModule } from '@angular/core'; 
```

确保替换掉已经包含对`NgModule`导入的行。

在同一文件的下方一点，添加以下内容：

```cs
 schemas: [
    CUSTOM_ELEMENTS_SCHEMA // Tells Angular we will have custom tags in our templates
  ] 
```

现在，Angular 对使用自定义元素表示满意。

接下来，是时候添加我们的组件了。在`angularproject.client/src/app/app.component.html`中，我们添加我们的自定义标签：

```cs
<my-blazor-counter increment-amount="10"></my-blazor-counter> 
```

在这种情况下，我们将`increment-amount`参数设置为`10`，每次点击时计数器将增加`10`。

要使这一切都能正常工作，我们需要加载一些 JavaScript 脚本。在`angularproject.client/src/index.html`中，我们需要添加以下内容：

```cs
<script src="img/Microsoft.AspNetCore.Components.CustomElements.lib.module.js"></script>
<script src="img/blazor.webassembly.js"></script> 
```

我们还有最后一件事需要修复。当运行 Angular 项目时，它会启动一个开发者服务器。实际上，它会启动两个：一个用于 ASP.NET 后端，一个用于 Angular 前端。我们需要让 Angular 服务器将所有框架请求发送到 ASP.NET 后端。

在默认项目模板中，这已经为`/weatherforecast`路径完成了。将以下代码添加到`angularproject.client/proxy.conf.js`文件中：

```cs
 context: [
  "/weatherforecast",
  "/_framework",
  "/_content",
], 
```

我们告诉开发者服务器，如果请求是发送到`weatherforecast`、`_framework`或`_content`，我们希望将该请求重定向到 ASP.NET 后端。

现在我们有一个工作的 Angular/Blazor WebAssembly 混合应用。我第一次尝试时，真的对它有多么简单和直接感到惊讶。这使得在 Angular 网站上包含一些 Blazor 组件变得非常容易，因此您可以逐步将其转换为 Blazor，组件一个接一个。

接下来，我们将使用 React 网站执行相同的操作。

# 将 Blazor 添加到 React 网站

将 Blazor 添加到 React 网站与添加到 Angular 非常相似。这个演示基于 Visual Studio 中的 React 和 ASP.NET Core 模板。项目名为`ReactProject`。

首先，我们需要对 Blazor 库有一个引用，我添加了`BlazorCustomElement`项目作为引用。

我们需要一个对`Microsoft.AspNetCore.Components.WebAssembly.Server NuGet`包的引用；这样我们就可以提供框架文件。

要使我们的网站能够提供框架文件，我们需要在`Program.cs`中添加以下内容：

```cs
app.UseBlazorFrameworkFiles(); 
```

接下来，是时候添加我们的组件了。在`reactproject.client/src/App.tsx`中，我们添加我们的自定义标签：

```cs
<my-blazor-counter increment-amount="10"></my-blazor-counter> 
```

在这种情况下，我们将`increment-amount`参数设置为`10`，每次点击时计数器将增加`10`。

要使这一切都能正常工作，我们需要加载一些 JavaScript。在`reactproject.client/index.html`中，我们需要添加以下内容：

```cs
 <script src="img/Microsoft.AspNetCore.Components.CustomElements.lib.module.js"></script>
<script src="img/blazor.webassembly.js"></script> 
```

这些脚本将确保我们的组件加载。

我们还有最后一件事需要修复。当运行 React 项目时，它会启动一个开发者服务器。实际上，它会启动两个：一个用于 ASP.NET 后端，一个用于 React 前端。

我们需要让 React 服务器将所有框架请求发送到 ASP.NET 后端。在默认项目模板中，这已经为 `/weatherforecast` 路径完成了。

将以下代码添加到 `reactproject.client/vite.config.ts` 文件中：

```cs
'^/_framework': {
    target,
    secure: false
},
'^/_content': {
    target,
    secure: false
}, 
```

我们告诉开发者服务器，如果请求发送到 `weatherforecast`、`_framework` 或 `_content`，我们希望将那个请求重定向到 ASP.NET 后端。

现在我们有一个工作的 React/Blazor WebAssembly 混合。这非常类似于 Angular，我对它如何简单直接感到惊讶。这使得在 React 网站上包含一些 Blazor 组件变得非常容易，这样你可以逐步将它们转换为 Blazor。

接下来，我们将使用一个 Razor Pages 网站做同样的事情。

# 将 Blazor 添加到 MVC/Razor Pages

当我开始使用 Blazor 时，这正是我们想要解决的问题。我们有一个 MVC/Razor Pages 混合，是时候升级了。

我们通过实现引用了 Razor 组件的 Razor Pages 解决了这个问题。现在回想起来，这并不是一个很漂亮的解决方案，至少一开始不是，直到我们到达大多数代码都在 Blazor 中重写的点。

挑战在于，如果我们导航到一个包含 Blazor 组件（一个 Razor 组件）的页面，该页面会连接到服务器并建立 WebSocket。如果我们从 Blazor 页面导航到一个 MVC 页面，例如，我们会重新加载整个页面，脚本也会重新加载。新的连接被建立，而旧的连接在服务器上保持 3 分钟。

我们的用户不多，对我们来说，这种技术足以让我们完成迁移并推出网站的新的 Blazor 版本。

但我有好消息！

我们还可以使用相同的自定义元素在 Razor Pages 网站上运行。

让我们看看吧！

该项目被称为 `RazorPagesProject`。

在之前的 Angular 和 React 示例中，那些技术是客户端的；因此，我们使用了 WebAssembly。Razor Pages 是服务器端的，尽管我们也可以在这里使用 WebAssembly，但这是一个很好的机会来看看如何使 **自定义组件** 使用 Blazor 服务器。

首先，我们需要引用我们的 Blazor 库。我添加了 `BlazorCustomElement` 项目作为引用。

然后，我们需要通过在 `Program.cs` 中添加以下代码来在我们的 Razor Pages 中启用 Blazor 服务器。

```cs
builder.Services.AddRazorComponents()
    .AddInteractiveServerComponents(options =>
    {
        options.RootComponents.RegisterCustomElement<Counter>("my-counter");
    }); 
```

还有：

```cs
app.MapBlazorHub(); 
```

在 `Pages/Shared/_Layout.cshtml` 中，我们需要添加以下 JavaScript：

```cs
 <script src="img/Microsoft.AspNetCore.Components.CustomElements.lib.module.js"></script>
<script src="img/blazor.server.js"></script> 
```

在这种情况下，我们添加 Blazor 服务器端的脚本。

最后但同样重要的是，我们需要添加我们的组件。在 `Pages/Index.cshtml` 中，我们添加：

```cs
<my-counter increment-amount="10"></my-counter> 
```

我们完成了；自定义组件现在正在我们的 Razor Pages 网站内运行（当然，这是一个启用了 Razor Pages 的 ASP.NET 网站）。

最酷的部分是，只需进行一些更改，我们就可以将此实现切换为使用 WebAssembly 而不是 Blazor 服务器运行 Blazor 组件。

再次，我对这个项目印象深刻；它使得将现有网站迁移到 Blazor 变得非常简单。

接下来，我们将看看如何在我们的 Blazor 网站上使用 Angular 或 React 控件。

# 将 Web 组件添加到 Blazor 网站

我们已经探讨了如何将 Blazor 添加到现有的 Angular、React 以及 MVC/Razor Pages 网站。

但有时，你可能非常喜欢使用的那个完美的库可能没有 Blazor 对应版本。我们知道我们可以创建一个 JavaScript 互操作并自己构建它，但我们是否也可以从 Blazor 使用 Angular 和 React 库？

这里我们有两个选择；要么我们可以将我们的网站转换为 Angular/React 网站，并使用那些示例，要么我们可以将 JavaScript 库转换为 Web 组件，并在 Blazor 中使用它。

到目前为止，我们还没有使用 npm 或类似的东西，因为在大多数情况下，我们不需要它。但现在我们在混合技术，为此，npm 是最简单的方式。`npm` 不在本书的范围之内，所以我就不会深入介绍它了。

如何将 Angular/React 或其他任何东西转换为 Web 组件也不在本书的范围之内。

该项目被称为 `BlazorProject`。

我们可以浏览这个网站上的一些 Web 组件：[`www.webcomponents.org/`](https://www.webcomponents.org/)。

我在 GitHub 上找到了一个 Markdown 编辑器。即使我们不会在我们的博客上实现它，如果你愿意，也可以随时回去实现。

我们可以在这里了解编辑器：

`www.webcomponents.org/element/@github/markdown-toolbar-element`

要获取所需的 JavaScript 文件，我们需要设置 `npm`。在项目文件夹（`BlazorProject.Client`）中，运行以下命令：

```cs
npm init
npm install –save @github/markdown-toolbar-element 
```

这将下载我们需要的 JavaScript。

接下来，将 `BlazorProject\node_modules\@github\markdown-toolbar-element\` 文件夹复制到 `wwwroot` 文件夹（在服务器项目中），并将其包含在项目中。

现在，JavaScript 将可以从我们的项目中访问。

在 `app.razor` 中，我们需要添加对 JavaScript 的引用，并将其放在 Blazor JavaScript 下方：

```cs
<script type="module" src="img/index.js"></script> 
```

这个组件是一个 ES6 模块，所以我们将其类型设置为 `"module"`。

现在，剩下的事情就是添加我们的组件。在演示项目中，我将它添加到了 `MarkdownDemo` 组件中。

首先，是组件：

```cs
<markdown-toolbar for="textarea" role="toolbar">
<md-bold class="btn btn-sm" tabindex="0">bold</md-bold>
<md-header class="btn btn-sm" tabindex="-1">header</md-header>
<md-italic class="btn btn-sm" tabindex="-1">italic</md-italic>
<md-quote class="btn btn-sm" tabindex="-1">quote</md-quote>
<md-code class="btn btn-sm" tabindex="-1">code</md-code>
<md-link class="btn btn-sm" tabindex="-1">link</md-link>
<md-image class="btn btn-sm" tabindex="-1">image</md-image>
<md-unordered-list class="btn btn-sm" tabindex="-1">unordered-list</md-unordered-list>
<md-ordered-list class="btn btn-sm" tabindex="-1">ordered-list</md-ordered-list>
<md-task-list class="btn btn-sm" tabindex="-1">task-list</md-task-list>
<md-mention class="btn btn-sm" tabindex="-1">mention</md-mention>
<md-ref class="btn btn-sm" tabindex="-1">ref</md-ref>
<md-strikethrough class="btn btn-sm" tabindex="-1">strikethrough</md-strikethrough>
</markdown-toolbar> 
```

然后是绑定到 C# 变量 `markdown` 的文本区域：

```cs
<textarea @bind="markdown" @bind:event="oninput" rows="6" class="mt-3 d-block width-full" id="textarea" contenteditable="false" spellcheck="false"></textarea>
@markdown
@code
{
    private string markdown = "Hello, **world**!";
} 
```

当我们编辑文本框时，C# 变量会立即改变，无论是通过使用工具栏还是输入一些文本。

我们已经将一个 Web 组件集成到我们的 Blazor 项目中，它绑定到一个 C# 变量。

这非常强大，为我们提供了将现有功能添加到我们的 Blazor 网站的新可能性。

现在我们知道了如何处理像 React 和 Angular 这样的 SPA 框架。但关于像 Web 表单这样的服务器框架怎么办？这就是我们接下来要看的。

# 从 Web 表单迁移

最后但同样重要的是，我们有 **Web 表单**。

实际上，对于 Web 表单并没有一个好的升级路径；曾经有一个项目旨在将代码重用到 Blazor 迁移中，但现在它并没有被积极开发。

我们首先应该知道的是，Blazor 在许多方面与 Web 表单非常相似，因此学习 Blazor 的曲线几乎不存在，因为我们既有 Web 表单也有 Blazor 中的状态管理。

有一些迁移策略，你可能会使用**另一个反向代理**（**YARP**）。然而，我的建议是将网站的一部分迁移到 Blazor，并让两个网站同时运行，直到我们达到功能完整的点。迁移到 Blazor 相对较快，最终我相信这将为您节省时间。

当我们将我们的网站从 MVC 迁移到 Blazor 时，我们意识到在某些情况下，将组件重写为 Blazor 比在 MVC 中解决问题要快。

由于后端代码与 Blazor 比 MVC 更相似，Web 表单的转换应该更快。

那么，我们应该怎么做呢？是升级还是继续使用 Web 表单？升级——你不会失望的！

# 摘要

在本章中，我们讨论了将 Blazor 添加到其他技术中，如 Angular、React 和 Razor Pages，使用 Web 组件。我们探讨了如何将 Web 组件添加到 Blazor 项目中，并在我们的 Blazor 应用程序中利用 JavaScript 库。

将现有网站升级到 Blazor 可能是一项大量工作。在我以前雇主那里，我们 4 年前就经历了这个过程。在我们的案例中，我们希望更新我们的 MVC 网站以使其更具交互性。我们选择了 Blazor，我认为这挽救了我们的项目并提高了我们的生产力，从而带来了更丰富的用户体验。

在下一章中，我们将更深入地探讨 Blazor WebAssembly。

# 加入我们的 Discord 社区

加入我们社区的 Discord 空间，与作者和其他读者进行讨论：

[`packt.link/WebDevBlazor3e`](https://packt.link/WebDevBlazor3e)

![](img/QR_Code2668029180838459906.png)
