# 第十六章：Blazor WebAssembly

在本章中，您将学习如何使用 Blazor WebAssembly 实现演示层。Blazor WebAssembly 应用程序是 C#应用程序，可以在支持 WebAssembly 技术的任何浏览器中运行。它们可以通过导航到特定的 URL 进行访问，并以标准静态内容的形式在浏览器中下载，由 HTML 页面和可下载文件组成。

Blazor 应用程序使用了我们在第十五章《介绍 ASP.NET Core MVC》中已经分析过的许多技术，比如依赖注入和 Razor。因此，我们强烈建议在阅读本章之前先学习第十五章《介绍 ASP.NET Core MVC》。

更具体地说，在本章中，您将学习以下主题：

+   Blazor WebAssembly 架构

+   Blazor 页面和组件

+   Blazor 表单和验证

+   Blazor 高级特性，如全球化、身份验证和 JavaScript 互操作性

+   Blazor WebAssembly 的第三方工具

+   用例：在 Blazor WebAssembly 中实现一个简单的应用程序

虽然也有运行在服务器上的 Blazor，就像 ASP.NET Core MVC 一样，但本章仅讨论 Blazor WebAssembly，它完全在用户的浏览器中运行，因为本章的主要目的是提供一个相关的示例，展示如何使用客户端技术实现演示层。此外，作为一种服务器端技术，Blazor 无法提供与其他服务器端技术（如 ASP.NET Core MVC）相媲美的性能，我们已经在第十五章《介绍 ASP.NET Core MVC》中进行了分析。

第一节概述了 Blazor WebAssembly 的总体架构，而其余部分描述了具体特性。在需要时，通过分析和修改 Visual Studio 在选择 Blazor WebAssembly 项目模板时自动生成的示例代码来澄清概念。最后一节展示了如何将学到的所有概念应用到实践中，实现一个基于 WWTravelClub 书籍用例的简单应用程序。

# 技术要求

本章需要免费的 Visual Studio 2019 社区版或更高版本，并安装了所有数据库工具。所有概念都将通过一个简单的示例应用程序进行澄清，该应用程序基于 WWTravelClub 书籍用例。本章的代码可在[`github.com/PacktPublishing/Software-Architecture-with-C-9-and-.NET-5`](https://github.com/PacktPublishing/Software-Architecture-with-C-9-and-.NET-5)上找到。

# Blazor WebAssembly 架构

Blazor WebAssembly 利用了新的 WebAssembly 浏览器功能，在浏览器中执行.NET 运行时。这样，它使所有开发人员都能够在任何支持 WebAssembly 的浏览器中运行的应用程序的实现中使用整个.NET 代码库和生态系统。WebAssembly 被构想为 JavaScript 的高性能替代品。它是一种能够在浏览器中运行并遵守与 JavaScript 代码相同限制的汇编。这意味着 WebAssembly 代码，就像 JavaScript 代码一样，运行在一个具有非常有限访问所有机器资源的隔离执行环境中。

WebAssembly 与过去的类似选项（如 Flash 和 Silverlight）不同，因为它是 W3C 的官方标准。更具体地说，它于 2019 年 12 月 5 日成为官方标准，因此预计将有很长的寿命。事实上，所有主流浏览器已经支持它。

然而，WebAssembly 不仅带来了性能！它还为现代和先进的面向对象语言（如 C++（直接编译）、Java（字节码）和 C#（.NET））在浏览器中运行整个代码库创造了机会。

微软建议使用 Unity 3D 图形框架和 Blazor 在浏览器中运行.NET 代码。

在 WebAssembly 之前，浏览器中运行的演示层只能用 JavaScript 实现，这带来了语言维护所带来的所有问题。

现在，使用 Blazor，我们可以使用现代和先进的 C#来实现复杂的应用程序，利用 C#编译器和 Visual Studio 为这种语言提供的所有便利。

此外，使用 Blazor，所有.NET 开发人员都可以利用.NET 框架的全部功能来实现在浏览器中运行的表示层，并与在服务器端运行的所有其他层共享库和类。

接下来的小节描述了 Blazor 架构的整体情况。第一小节探讨了单页应用程序的一般概念，并指出了 Blazor 的特点。

## 什么是单页应用程序？

**单页应用程序**（**SPA**）是一个基于 HTML 的应用程序，其中 HTML 由在浏览器中运行的代码更改，而不是向服务器发出新请求并从头开始呈现新的 HTML 页面。SPA 能够通过用新的 HTML 替换完整的页面区域来模拟多页面体验。

SPA 框架是专门设计用于实现单页应用程序的框架。在 WebAssembly 出现之前，所有的 SPA 框架都是基于 JavaScript 的。最著名的基于 JavaScript 的 SPA 框架是 Angular、React.js 和 Vue.js。

所有的 SPA 框架都提供了将数据转换为 HTML 以显示给用户的方法，并依赖一个称为*router*的模块来模拟页面更改。通常，数据填充到 HTML 模板的占位符中，并选择要呈现的模板部分（类似 if 的结构），以及呈现的次数（类似 for 的结构）。

Blazor 的模板语言是 Razor，我们已经在*第十五章*中描述过。

为了增加模块化，代码被组织成组件，这些组件是一种虚拟的 HTML 标记，一旦呈现，就会生成实际的 HTML 标记。像 HTML 标记一样，组件有它们的属性，通常被称为参数，以及它们的自定义事件。开发人员需要确保每个组件使用它的参数来创建适当的 HTML，并确保它生成足够的事件。组件可以以分层的方式嵌套在其他组件中。

应用程序路由器通过选择组件来执行其工作，充当页面，并将它们放置在预定义的区域。每个页面组件都有一个与之相关联的 Web 地址路径。这个路径与 Web 应用程序域连接在一起，成为一个唯一标识页面的 URL。与通常的 Web 应用程序一样，页面 URL 用于与路由器通信，以确定要加载哪个页面，可以使用常规链接或路由方法/函数。

一些 SPA 框架还提供了预定义的依赖注入引擎，以确保组件与在浏览器中运行的通用服务和业务代码之间有更好的分离。在本小节列出的框架中，只有 Blazor 和 Angular 具有开箱即用的依赖注入引擎。

基于 JavaScript 的 SPA 框架通常会将所有 JavaScript 代码编译成几个 JavaScript 文件，然后执行所谓的摇树操作，即删除所有未使用的代码。

目前，Blazor 将主应用程序引用的所有 DLL 分开，并对每个 DLL 执行摇树操作。

下一小节开始描述 Blazor 架构。鼓励您创建一个名为`BlazorReview`的 Blazor WebAssembly 项目，这样您就可以检查整个章节中解释的代码和构造。请选择**个人用户帐户**作为身份验证，以及**ASP.NET Core hosted**。这样，Visual Studio 还将创建一个与 Blazor 客户端应用程序通信的 ASP.NET Core 项目，其中包含所有身份验证和授权逻辑。

![](img/B16756_16_01.png)

图 16.1：创建 BlazorReview 应用程序

如果启动应用程序并尝试登录或尝试访问需要登录的页面，则应该出现一个错误，指出数据库迁移尚未应用。只需单击消息旁边的链接即可应用待处理的迁移。否则，如*第八章*的*使用 C#与数据交互-Entity Framework Core*部分中所解释的那样，转到 Visual Studio 包管理器控制台并运行`Update-Database`命令。

## 加载和启动应用程序

Blazor WebAssembly 应用程序的 URL 始终包括一个`index.html`静态 HTML 页面。在我们的`BlazorReview`项目中，`index.html`位于`BlazorReview.Client->wwwroot->index.html`。此页面是 Blazor 应用程序将创建其 HTML 的容器。它包含一个带有`viewport meta`声明、标题和整个应用程序 CSS 的 HTML 头。Visual Studio 默认项目模板添加了一个特定于应用程序的 CSS 文件和 Bootstrap CSS，具有中性样式。您可以使用具有自定义样式的默认 Bootstrap CSS 或完全不同的 CSS 框架来替换默认的 Bootstrap CSS。

正文包含以下代码：

```cs
<body>
<div id="app">Loading...</div>
<div id="blazor-error-ui">
        An unhandled error has occurred.
<a href="" class="reload">Reload</a>
<a class="dismiss">![](img/B16756_16_001.png)</a>
</div>
<script 
src="img/AuthenticationService.js">
</script>
<script src="img/blazor.webassembly.js"></script>
</body> 
```

初始的`div`是应用程序将放置其生成的代码的地方。放置在此`div`中的任何标记都将在 Blazor 应用程序加载和启动时出现，然后将被应用程序生成的 HTML 替换。第二个`div`通常是不可见的，只有在 Blazor 拦截到未处理的异常时才会出现。

`blazor.webassembly.js`包含 Blazor 框架的 JavaScript 部分。除其他外，它负责下载.NET 运行时以及所有应用程序 DLL。更具体地说，`blazor.webassembly.js`下载列出所有应用程序文件及其哈希值的`blazor.boot.json`文件。然后，`blazor.webassembly.js`下载此文件中列出的所有资源并验证它们的哈希值。`blazor.webassembly.js`下载的所有资源都是在构建或发布应用程序时创建的。

只有在项目启用身份验证时才会添加`AuthenticationService.js`，它负责 Blazor 利用其他身份验证凭据（如 cookie）来获取承载令牌的`OpenID Connect`协议。承载令牌是客户端通过 Web API 与服务器交互的首选身份验证凭据。身份验证将在本章后面的*身份验证和授权*子章节中更详细地讨论，而承载令牌将在*第十四章*的*应用 Service-Oriented Architectures with .NET Core*部分中讨论。

Blazor 应用程序的入口点在`BlazorReview.Client->Program.cs`文件中。它具有以下结构：

```cs
public class Program
{
    public static async Task Main(string[] args)
        {
            var builder = WebAssemblyHostBuilder.CreateDefault(args);
            builder.RootComponents.Add<App>("#app");
            // Services added to the application 
            // Dependency Injection engine declared with statements like:
            // builder.Services.Add...
            await builder.Build().RunAsync();
        }
    } 
```

`WebAssemblyHostBuilder`是用于创建`WebAssemblyHost`的构建器，它是在*第五章*的*将微服务架构应用于企业应用程序*中讨论的通用主机的 WebAssembly 特定实现（鼓励您查看该子章节）。第一个构建器配置指令声明了 Blazor 根组件（`App`），它将包含整个组件树，并在`Index.html`页面的哪个 HTML 标记中放置它（`#app`）。更具体地说，`RootComponents.Add`添加了一个托管服务，负责处理整个 Blazor 组件树。我们可以通过多次调用`RootComponents.Add`在同一个 HTML 页面中运行多个 Blazor WebAssembly 用户界面，每次使用不同的 HTML 标记引用。

`builder.Services`包含了所有通常的方法和扩展方法，用于向 Blazor 应用程序的依赖引擎添加服务：`AddScoped`、`AddTransient`、`AddSingleton`等等。就像在 ASP.NET Core MVC 应用程序中一样（*第十五章*，*介绍 ASP.NET Core MVC*），服务是实现业务逻辑和存储共享状态的首选位置。在 ASP.NET Core MVC 中，服务通常传递给控制器，而在 Blazor WebAssembly 中，它们被注入到组件中。

下一小节将解释根`App`组件如何模拟页面更改。

## 路由

由主机构建代码引用的根`App`类在`BlazorReview.Client->App.razor`文件中定义。`App`是一个 Blazor 组件，像所有 Blazor 组件一样，它是在具有`.razor`扩展名的文件中定义的，并且使用富有组件标记的 Razor 语法，即用表示其他 Blazor 组件的类似 HTML 的标签。它包含了处理应用程序页面的全部逻辑：

```cs
<CascadingAuthenticationState>
<Router AppAssembly="@typeof(Program).Assembly">
<Found Context="routeData">
<AuthorizeRouteView RouteData="@routeData"
                    DefaultLayout="@typeof(MainLayout)">
<NotAuthorized>
@*Template that specifies what to show 
when user is not authorized *@
</NotAuthorized>
</AuthorizeRouteView>
</Found>
<NotFound>
<LayoutView Layout="@typeof(MainLayout)">
<p>Sorry, there's nothing at this address.</p>
</LayoutView>
</NotFound>
</Router>
</CascadingAuthenticationState> 
```

前面代码中的所有标记都代表组件或特定的组件参数，称为模板。组件将在本章中详细讨论。暂时想象它们是一种我们可以用 C#和 Razor 代码定义的自定义 HTML 标记。模板则是接受 Razor 标记作为值的参数。模板将在本节的*模板和级联参数*小节中讨论。

`CascadingAuthenticationState`组件的唯一功能是将身份验证和授权信息传递给其内部组件树中的所有组件。只有在项目创建过程中选择添加授权时，Visual Studio 才会生成它。

`Router`组件是实际的应用程序路由器。它扫描`AppAssembly`参数中传递的程序集，寻找包含路由信息的组件，即可以作为页面工作的组件。Visual Studio 将包含`Program`类的程序集传递给它，即主应用程序。其他程序集中包含的页面可以通过`AdditionalAssemblies`参数添加，该参数接受一个程序集的`IEnumerable`。

之后，路由器拦截所有通过代码或通过通常的`<a>` HTML 标签执行的页面更改，这些标签指向应用程序基地址内的地址。导航可以通过代码处理，通过从依赖注入中要求`NavigationManager`实例来处理。

`Router`组件有两个模板，一个用于找到请求的 URI 的页面（`Found`），另一个用于找不到请求的页面（`NotFound`）。当应用程序使用授权时，`Found`模板由`AuthorizeRouteView`组件组成，进一步区分用户是否有权访问所选页面。当应用程序不使用授权时，`Found`模板由`RouteView`组件组成：

```cs
<RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" /> 
```

`RouteView`接受所选页面，并在由`DefaultLayout`参数指定的布局页面内呈现它。这个规范只是一个默认值，因为每个页面都可以通过指定不同的布局页面来覆盖它。Blazor 布局页面的工作方式类似于*第十五章*，*介绍 ASP.NET Core MVC*中描述的 ASP.NET Core MVC 布局页面，唯一的区别是指定页面标记的位置是用`@Body`指定的：

```cs
<div class="content px-4">
      @Body
</div> 
```

在 Visual Studio 模板中，默认的布局页面位于`BlazorReview.Client->Shared->MainLayout.razor`文件中。

如果应用程序使用授权，`AuthorizeRouteView`的工作方式类似于`RouteView`，但它还允许指定一个用户未经授权的情况下的模板：

```cs
<NotAuthorized>
@if (!context.User.Identity.IsAuthenticated)
{
<RedirectToLogin />
}
else
{
<p>You are not authorized to access this resource.</p>
}
</NotAuthorized> 
```

如果用户未经过身份验证，`RedirectToLogin`组件将使用`NavigationManager`实例来转到登录逻辑页面，否则，它会通知用户他们没有足够的权限来访问所选页面。

Blazor WebAssembly 还允许程序集延迟加载以减少初始应用程序加载时间，但由于篇幅有限，我们将不在此讨论。*进一步阅读*部分包含了官方 Blazor 文档的参考资料。

# Blazor 页面和组件

在本节中，您将学习 Blazor 组件的基础知识，如何定义组件，其结构，如何将事件附加到 HTML 标记，如何定义它们的属性，以及如何在组件内部使用其他组件。我们已将所有内容组织成不同的子节。第一个子节描述了组件结构的基础知识。

## 组件结构

组件是在扩展名为`.razor`的文件中定义的。一旦编译，它们就变成了从`ComponentBase`继承的类。与所有其他 Visual Studio 项目元素一样，Blazor 组件可以通过**添加新项**菜单获得。通常，要用作页面的组件是在`Pages`文件夹中定义的，或者在其子文件夹中定义，而其他组件则组织在不同的文件夹中。默认的 Blazor 项目将所有非页面组件添加到`Shared`文件夹中，但您可以以不同的方式组织它们。

默认情况下，页面被分配一个与它们所在文件夹的路径相对应的命名空间。因此，例如，在我们的示例项目中，所有在`BlazorReview.Client->Pages`路径中的页面都被分配到`BlazorReview.Client.Pages`命名空间。但是，您可以通过在文件顶部的声明区域中放置一个`@namespace`声明来更改此默认命名空间。此区域还可以包含其他重要的声明。以下是一个显示所有声明的示例：

```cs
@page "/counter"
@layout MyCustomLayout
@namespace BlazorApp2.Client.Pages
@using Microsoft.AspNetCore.Authorization
@implements MyInterface
@inherits MyParentComponent
@typeparam T
@attribute [Authorize]
@inject NavigationManager navigation 
```

前两个指令只对必须作为页面工作的组件有意义。更具体地说，`@layout`指令用另一个组件覆盖默认的布局页面，而`@page`指令定义了页面的路径（**路由**）在应用程序的基本 URL 内。因此，例如，如果我们的应用程序在`https://localhost:5001`上运行，那么上述页面的 URL 将是`https://localhost:5001/counter`。页面路由也可以包含参数，就像在这个例子中一样：`/orderitem/{customer}/{order}`。参数名称必须与组件定义的参数的公共属性匹配。匹配不区分大小写，并且参数将在本小节后面进行解释。

实例化每个参数的字符串被转换为参数类型，如果此转换失败，则会抛出异常。可以通过将每个参数与类型关联来防止这种行为，在这种情况下，如果转换为指定类型失败，则与页面 URL 的匹配失败。只支持基本类型：`/orderitem/{customer:int}/{order:int}`。参数是强制性的，也就是说，如果找不到它们，匹配失败，路由器会尝试其他页面。但是，您可以通过指定两个`@page`指令使参数变为可选，一个带参数，另一个不带参数。

`@namespace`覆盖了组件的默认命名空间，而`@using`等同于通常的 C# `using`。在特殊的`{project folder}->_Imports.razor`文件夹中声明的`@using`会自动应用于所有组件。

`@inherits`声明组件是另一个组件的子类，而`@implements`声明它实现了一个接口。

如果组件是一个泛型类，则使用`@typeparam`，并声明泛型参数的名称，而`@attribute`声明应用于组件类的任何属性。属性级别的属性直接应用于代码区域中定义的属性，因此它们不需要特殊的标记。`[Authorize]`属性应用于作为页面使用的组件类，防止未经授权的用户访问页面。它的工作方式与在 ASP.NET Core MVC 中应用于控制器或操作方法时完全相同。

最后，`@inject`指令需要一个类型实例来注入依赖注入引擎，并将其插入到类型名称后声明的字段中；在前面的示例中，在`navigation`参数中。

组件文件的中间部分包含了将由 Razor 标记呈现的 HTML，其中可能包含对子组件的调用。

文件的底部由`@code`构造包围，并包含实现组件的类的字段、属性和方法：

```cs
@code{
 ...
 private string myField="0";
 [Parameter]
 public int Quantity {get; set;}=0;
 private void IncrementQuantity ()
 {
         Quantity++;
 }
 private void DecrementQuantity ()
 {
        Quantity--;
        if (Quantity<0) Quantity=0;
 }
 ... 
} 
```

用`[Parameter]`属性修饰的公共属性作为组件参数工作；也就是说，当组件实例化到另一个组件中时，它们用于将值传递给修饰的属性，就像在 HTML 标记中将值传递给 HTML 元素一样：

```cs
<OrderItem Quantity ="2" Id="123"/> 
```

值也可以通过页面路由参数传递给组件参数，这些参数与属性名称进行不区分大小写的匹配：

```cs
OrderItem/{id}/{quantity} 
```

组件参数也可以接受复杂类型和函数：

```cs
<modal title='() => "Test title" ' ...../> 
```

如果组件是通用的，它们必须为每个使用`typeparam`声明的通用参数传递类型值：

```cs
<myGeneric T= "string"……/> 
```

然而，通常编译器能够从其他参数的类型中推断出通用类型。

最后，`@code`指令包围的代码也可以在与组件相同的名称和命名空间的部分类中声明：

```cs
public partial class Counter
{
  [Parameter] 
public int CurrentCounter {get; set;}=0;
  ...
  ...
} 
```

通常，这些部分类被声明在与组件相同的文件夹中，并且文件名等于组件文件名加上`.cs`后缀。因此，例如，与`counter.razor`组件关联的部分类将是`counter.razor.cs`。

每个组件也可以有一个关联的 CSS 文件，其名称必须是组件文件名加上`.css`后缀。因此，例如，与`counter.razor`组件关联的 CSS 文件将是`counter.razor.css`。此文件中包含的 CSS 仅应用于该组件，对页面的其余部分没有影响。这称为 CSS 隔离，目前是通过向所有组件 HTML 根添加唯一属性来实现的。然后，组件 CSS 文件的所有选择器都被限定为此属性，以便它们不能影响其他 HTML。

每当一个组件用`[Parameter(CaptureUnmatchedValues = true)]`修饰一个`IDictionary<string, object>`参数时，那么所有未匹配的参数插入到标签中，也就是所有没有匹配组件属性的参数，都会作为键值对添加到`IDictionary`中。

此功能提供了一种简单的方法，将参数转发给组件标记中包含的 HTML 元素或其他子组件。例如，如果我们有一个`Detail`组件，它显示传递给其`Value`参数的对象的详细视图，我们可以使用此功能将所有常规 HTML 属性转发到组件的根 HTML 标记，如下例所示：

```cs
<div  @attributes="AdditionalAttributes">
...
</div>
@code{
[Parameter(CaptureUnmatchedValues = true)]
public Dictionary<string, object>
AdditionalAttributes { get; set; }
 [Parameter]
 Public T Value {get; set;}
} 
```

这样，添加到组件标记的常规 HTML 属性，例如 class，将被转发到组件的根`div`，并以某种方式用于样式化组件：

```cs
<Detail Value="myObject" class="my-css-class"/> 
```

下一小节解释了如何将生成标记的函数传递给组件。

## 模板和级联参数

Blazor 通过构建称为**渲染树**的数据结构来工作，该结构在 UI 更改时进行更新。在每次更改时，Blazor 会定位必须呈现的 HTML 部分，并使用**渲染树**中包含的信息来更新它。

`RenderFragment`委托定义了一个能够向**渲染树**的特定位置添加更多标记的函数。还有一个`RenderFragment<T>`，它接受一个进一步的参数，您可以使用它来驱动标记生成。例如，您可以将`Customer`对象传递给`RenderFragment<T>`，以便它可以呈现该特定客户的所有数据。

您可以使用 C#代码定义`RenderFragment`或`RenderFragment<T>`，但最简单的方法是在组件中使用 Razor 标记进行定义。Razor 编译器将负责为您生成适当的 C#代码：

```cs
RenderFragment myRenderFragment = @<p>The time is @DateTime.Now.</p>;
RenderFragment<Customer> customerRenderFragment = 
(item) => @<p>Customer name is @item.Name.</p>; 
```

有关添加标记的位置的信息是通过其接收的`RenderTreeBuilder`参数传递的。您可以通过简单调用它来在组件 Razor 标记中使用`RenderFragment`，如下例所示：

```cs
RenderFragment myRenderFragment = ...
  ...
<div>
  ...
  @myRenderFragment
  ...
</div>
  ... 
```

调用`RenderFragment`的位置定义了它将添加其标记的位置，因为组件编译器能够生成正确的`RenderTreeBuilder`参数传递给它。`RenderFragment<T>`委托的调用如下所示：

```cs
Customer myCustomer = ...
  ...
<div>
  ...
  @myRenderFragment(myCustomer)
  ...
</div>
  ... 
```

作为函数，渲染片段可以像所有其他类型一样传递给组件参数。但是，Blazor 有一种特定的语法，使同时定义和传递渲染片段到组件变得更容易，即**模板**语法。首先，在组件中定义参数：

```cs
[Parameter]
Public RenderFragment<Customer>CustomerTemplate {get; set;}
[Parameter]
Public RenderFragment Title {get; set;} 
```

然后，当您调用客户时，可以执行以下操作：

```cs
<Detail>
<Title>
<h5>This is a title</h5>
</Title>
<CustomerTemplate Context=customer>
<p>Customer name is @customer.Name.</p>
</CustomerTemplate >
</Detail> 
```

每个渲染片段参数都由与参数同名的标记表示。您可以将定义渲染片段的标记放在其中。对于具有参数的`CustomerTemplate`，`Context`关键字在标记内定义了参数名称。在我们的示例中，选择的参数名称是`customer`。

当组件只有一个渲染片段参数时，如果它的名称为`ChildContent`，则模板标记可以直接封闭在组件的开始和结束标记之间：

```cs
[Parameter]
Public RenderFragment<Customer> ChildContent {get; set;}
……………
……………
<IHaveJustOneRenderFragment Context=customer>
<p>Customer name is @customer.Name.</p>
</IHaveJustOneRenderFragment> 
```

为了熟悉组件模板，让我们修改`Pages->FetchData.razor`页面，以便不再使用`foreach`，而是使用`Repeater`组件。

让我们右键单击`Shared`文件夹，选择**添加**，然后**Razor 组件**，并添加一个新的**Repeater.razor**组件。然后，用以下内容替换现有代码：

```cs
@typeparam T
@foreach(var item in Values)
{
@ChildContent(item)
}
@code {
    [Parameter]
public RenderFragment<T> ChildContent { get; set; }
    [Parameter]
public IEnumerable<T> Values { get; set; }
} 
```

该组件使用泛型参数进行定义，以便可以与任何`IEnumerable`一起使用。现在让我们用这个替换**FetchData.razor**组件的`tbody`中的标记：

```cs
<Repeater Values="forecasts" Context="forecast">
<tr>
<td>@forecast.Date.ToShortDateString()</td>
<td>@forecast.TemperatureC</td>
<td>@forecast.TemperatureF</td>
<td>@forecast.Summary</td>
</tr>
</Repeater> 
```

由于`Repeater`组件只有一个模板，并且我们将其命名为`ChildContent`，因此我们可以直接在组件的开始和结束标记中放置我们的模板标记。运行它并验证页面是否正常工作。您已经学会了如何使用模板，以及放置在组件内部的标记定义了一个模板。

一个重要的预定义模板化 Blazor 组件是`CascadingValue`组件。它以不进行任何更改地呈现放置在其中的内容，但将类型实例传递给其所有后代组件：

```cs
<CascadingValue  Value="new MyOptionsInstance{...}">
……
</CascadingValue > 
```

现在，放置在`CascadingValue`标记内以及所有后代组件中的所有组件都可以捕获传递给`CascadingValueValue`参数的`MyOptionsInstance`实例。只需组件声明一个与`MyOptionsInstance`兼容的类型的公共或私有属性，并使用`CascadingParameter`属性进行修饰即可：

```cs
[CascadingParameter]
privateMyOptionsInstance options {get; set;} 
```

匹配是通过类型兼容性执行的。在与其他具有兼容类型的级联参数存在歧义的情况下，我们可以指定`CascadingValue`组件的`Name`可选参数，并将相同的名称传递给`CascadingParameter`属性：`[CascadingParameter("myUnique name")]`。

`CascadingValue`标签还有一个`IsFixed`参数，出于性能原因，应尽可能设置为`true`。实际上，传播级联值非常有用，用于传递选项和设置，但计算成本非常高。

当`IsFixed`设置为`true`时，传播仅在每个涉及内容的第一次呈现时执行，然后在内容的生命周期内不尝试更新级联值。因此，只要级联对象的指针在内容的生命周期内没有更改，就可以使用`IsFixed`。

级联值的一个例子是我们在*路由*小节中遇到的`CascadingAuthenticationState`组件，它将认证和授权信息级联到所有渲染的组件中。

## 事件

HTML 标记和 Blazor 组件都使用属性/参数来获取输入。HTML 标记通过事件向页面的其余部分提供输出，Blazor 允许将 C#函数附加到 HTML 的`on{event name}`属性。语法显示在`Pages->Counter.razor`组件中：

```cs
<p>Current count: @currentCount</p>
<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>
@code {
private int currentCount = 0;
private void IncrementCount()
    {
        currentCount++;
    }
} 
```

该函数也可以作为 lambda 内联传递。此外，它接受通常的`event`参数的 C#等价物。*进一步阅读*部分包含了指向 Blazor 官方文档页面的链接，列出了所有支持的事件及其参数。

Blazor 还允许组件中的事件，因此它们也可以返回输出。组件事件是类型为`EventCallBack`或`EventCallBack<T>`的参数。`EventCallBack`是没有参数的组件事件类型，而`EventCallBack<T>`是带有类型为`T`的参数的组件事件类型。为了触发一个事件，比如`MyEvent`，组件调用：

```cs
awaitMyEvent.InvokeAsync() 
```

或者

```cs
awaitMyIntEvent.InvokeAsync(arg) 
```

这些调用执行与事件绑定的处理程序，如果没有绑定处理程序，则不执行任何操作。

一旦定义，组件事件可以与 HTML 元素事件完全相同的方式使用，唯一的区别在于不需要使用`@`前缀来命名事件，因为在 HTML 事件中，`@`是需要区分 HTML 属性和 Blazor 添加的具有相同名称的参数之间的区别：

```cs
[Parameter]
publicEventCallback MyEvent {get; set;}
[Parameter]
publicEventCallback<int> MyIntEvent {get; set;}
...
...
<ExampleComponent 
MyEvent="() => ..." 
MyIntEvent = "(i) =>..." /> 
```

实际上，HTML 元素事件也是`EventCallBack<T>`，这就是为什么这两种事件类型的行为完全相同。`EventCallBack`和`EventCallBack<T>`都是结构体，而不是委托，因为它们包含一个委托，以及一个指向必须被通知事件已被触发的实体的指针。从形式上讲，这个实体由`Microsoft.AspNetCore.Components.IHandleEvent`接口表示。不用说，所有组件都实现了这个接口。通知`IHandleEvent`发生了状态变化。状态变化在 Blazor 更新页面 HTML 的方式中起着基本作用。我们将在下一小节中详细分析它们。

对于 HTML 元素，Blazor 还提供了通过向指定事件的属性添加`:preventDefault`和`:stopPropagation`指令来阻止事件的默认操作和事件冒泡的可能性，就像这些例子中一样：

```cs
@onkeypress="KeyHandler" @onkeypress:preventDefault="true"
@onkeypress="KeyHandler" @onkeypress:preventDefault="true" @onkeypress:stopPropagation  ="true" 
```

## 绑定

通常，组件参数值必须与外部变量、属性或字段保持同步。这种同步的典型应用是在输入组件或 HTML 标记中编辑的对象属性。每当用户更改输入值时，对象属性必须一致更新，反之亦然。对象属性值必须在组件渲染时立即复制到组件中，以便用户可以编辑它。

类似的情况由参数-事件对处理。具体来说，一方面，属性被复制到输入组件参数中。另一方面，每当输入更改值时，都会触发一个更新属性的组件事件。这样，属性和输入值保持同步。

这种情况非常常见和有用，以至于 Blazor 有一个特定的语法，可以同时定义事件和将属性值复制到参数中。这种简化的语法要求事件与交互中涉及的参数具有相同的名称，但带有`Changed`后缀。

例如，假设一个组件有一个`Value`参数。那么相应的事件必须是`ValueChanged`。此外，每当用户更改组件值时，组件必须通过调用`await ValueChanged.InvokeAsync(arg)`来调用`ValueChanged`事件。有了这个设置，可以使用这里显示的语法将属性`MyObject.MyProperty`与`Value`属性同步：

```cs
<MyComponent @bind-Value="MyObject.MyProperty"/> 
```

上述语法称为**绑定**。Blazor 会自动附加一个更新`MyObject.MyProperty`属性的事件处理程序到`ValueChanged`事件。

HTML 元素的绑定方式类似，但由于开发人员无法决定参数和事件的名称，因此必须使用略有不同的约定。首先，无需在绑定中指定参数名称，因为它始终是 HTML 输入`value`属性。因此，绑定简单地写为`@bind="object.MyProperty"`。默认情况下，对象属性在`change`事件上更新，但您可以通过添加`@bind-event: @bind-event="oninput"`属性来指定不同的事件。

此外，HTML 输入的绑定尝试自动将输入字符串转换为目标类型。如果转换失败，输入将恢复到其初始值。这种行为相当原始，因为在出现错误时，不会向用户提供错误消息，并且文化设置没有得到正确的考虑（HTML5 输入使用不变的文化，但文本输入必须使用当前文化）。我们建议只将输入绑定到字符串目标类型。Blazor 具有专门用于处理日期和数字的组件，应该在目标类型不是字符串时使用。我们将在*Blazor 表单和验证*部分中对它们进行描述。

为了熟悉事件，让我们编写一个组件，当用户单击确认按钮时，同步输入文本类型的内容。右键单击`Shared`文件夹，然后添加一个新的**ConfirmedText.razor**组件。然后用以下代码替换其代码：

```cs
<input type="text" @bind="Value" @attributes="AdditionalAttributes"/>
<button class="btn btn-secondary" @onclick="Confirmed">@ButtonText</button>
@code {
    [Parameter(CaptureUnmatchedValues = true)]
public Dictionary<string, object> AdditionalAttributes { get; set; }
    [Parameter]
public string Value {get; set;}
    [Parameter]
public EventCallback<string> ValueChanged { get; set; }
    [Parameter]
public string ButtonText { get; set; }
async Task Confirmed()
    {
        await ValueChanged.InvokeAsync(Value);
    }
} 
```

`ConfirmedText`组件利用按钮点击事件来触发`ValueChanged`事件。此外，组件本身使用`@bind`将其`Value`参数与 HTML 输入同步。值得指出的是，组件使用`CaptureUnmatchedValues`将应用于其标记的所有 HTML 属性转发到 HTML 输入。这样，`ConfirmedText`组件的用户可以通过简单地向组件标记添加`class`和/或`style`属性来设置输入字段的样式。

现在让我们在`Pages->Index.razor`页面中使用此组件，方法是将以下代码放在`Index.razor`的末尾：

```cs
<ConfirmedText @bind-Value="textValue" ButtonText="Confirm" />
<p>
    Confirmed value is: @textValue
</p>
@code{
private string textValue = null;
} 
```

如果运行项目并与输入及其**确认**按钮进行交互，您会发现每次单击**确认**按钮时，不仅会将输入值复制到`textValue`页面属性中，而且组件后面段落的内容也会得到一致的更新。

我们明确使用`@bind-Value`将`textValue`与组件同步，但是谁负责保持`textValue`与段落内容同步？答案在下一小节中。

## Blazor 如何更新 HTML

当我们在 Razor 标记中写入变量、属性或字段的内容时，例如`@model.property`，Blazor 不仅在组件呈现时呈现变量、属性或字段的实际值，而且还尝试在该值每次更改时更新 HTML，这个过程称为**变更检测**。变更检测是所有主要 SPA 框架的特性，但 Blazor 实现它的方式非常简单和优雅。

基本思想是，一旦所有 HTML 都被呈现，更改只能因为在事件内执行的代码而发生。这就是为什么`EventCallBack`和`EventCallBack<T>`包含对`IHandleEvent`的引用。当组件将处理程序绑定到事件时，Razor 编译器创建一个`EventCallBack`或`EventCallBack<T>`，并在其`struct`构造函数中传递绑定到事件的函数以及定义该函数的组件（`IHandleEvent`）。

处理程序的代码执行后，Blazor 运行时会通知`IHandleEvent`可能已更改。实际上，处理程序代码只能更改组件中定义处理程序的变量、属性或字段的值。反过来，这会触发组件中的变更检测。Blazor 验证了组件 Razor 标记中使用的变量、属性或字段的更改，并更新相关的 HTML。

如果更改的变量、属性或字段是另一个组件的输入参数，则该组件生成的 HTML 可能也需要更新。因此，会递归触发另一个根据该组件触发的变更检测过程。

先前概述的算法仅在满足以下列出的条件时才发现所有相关更改：

1.  在事件处理程序中，没有组件引用其他组件的数据结构。

1.  所有组件的输入都通过其参数而不是通过方法调用或其他公共成员到达。

如果由于前述条件之一的失败而未检测到更改，则开发人员必须手动声明组件可能的更改。这可以通过调用`StateHasChanged()`组件方法来实现。由于此调用可能会导致页面 HTML 的更改，因此其执行不能异步进行，而必须在 HTML 页面 UI 线程中排队。这是通过将要执行的函数传递给`InvokeAsync`组件方法来完成的。

总结一下，要执行的指令是`await InvokeAsync(StateHasChanged)`。

下一小节总结了组件的生命周期及相关的生命周期方法的描述。

## 组件生命周期

每个组件生命周期事件都有一个关联的方法。一些方法既有同步版本又有异步版本，有些只有异步版本，而有些只有同步版本。

组件生命周期始于传递给组件的参数被复制到相关的组件属性中。您可以通过覆盖以下方法来自定义此步骤：

```cs
public override async Task SetParametersAsync(ParameterView parameters)
{
await ...
await base.SetParametersAsync(parameters);
} 
```

通常，定制包括修改其他数据结构，因此调用基本方法也执行将参数复制到相关属性的默认操作。

之后，与这两种方法相关联的组件初始化如下：

```cs
protected override void OnInitialized()
{
    ...
}
protected override async Task OnInitializedAsync()
{
await ...
} 
```

它们在组件生命周期中只被调用一次，即在组件创建并添加到渲染树后立即调用。请将任何初始化代码放在那里，而不是在组件构造函数中，因为这将提高组件的可测试性，因为在那里，您已经设置了所有参数，并且未来的 Blazor 版本可能会池化和重用组件实例。

如果初始化代码订阅了某些事件或执行需要在组件销毁时进行清理的操作，请实现`IDisposable`，并将所有清理代码放在其`Dispose`方法中。实际上，每当组件实现`IDisposable`时，Blazor 在销毁组件之前都会调用其`Dispose`方法。

组件初始化后，每次组件参数更改时，都会调用以下两种方法：

```cs
protected override async Task OnParametersSetAsync()
{
await ...
}
protected override void OnParametersSet()
{
    ...
} 
```

它们是更新依赖于组件参数值的数据结构的正确位置。

之后，组件被渲染或重新渲染。您可以通过覆盖`ShouldRender`方法来防止更新后组件重新渲染：

```cs
protected override bool ShouldRender()
{
...
} 
```

只有在确定其 HTML 代码将更改时，才让组件重新渲染是一种高级优化技术，用于组件库的实现中。

组件渲染阶段还涉及调用其子组件。因此，只有在所有后代组件完成渲染后，组件渲染才被认为是完整的。渲染完成后，将调用以下方法：

```cs
protected override void OnAfterRender(bool firstRender)
{
if (firstRender)
    {
    }
...
}
protected override async Task OnAfterRenderAsync(bool firstRender)
{
if (firstRender)
    {
    await...
        ...
    }
    await ...
} 
```

由于在调用上述方法时，所有组件 HTML 都已更新，并且所有子组件都已执行完其生命周期方法，因此上述方法是执行以下操作的正确位置：

+   调用操纵生成的 HTML 的 JavaScript 函数。JavaScript 调用在*JavaScript 互操作性*子部分中描述。

+   处理附加到参数或级联参数的信息由后代组件。事实上，类似标签的组件和其他组件可能需要在根组件中注册一些子部件，因此根组件通常会级联一个数据结构，其中一些子组件可以注册。在`AfterRender`和`AfterRenderAsync`中编写的代码可以依赖于所有子部件已完成其注册的事实。

下一节描述了 Blazor 用于收集用户输入的工具。

# Blazor 表单和验证

与所有主要的 SPA 框架类似，Blazor 还提供了特定的工具来处理用户输入，同时通过错误消息和即时视觉提示向用户提供有效的反馈。整个工具集被称为**Blazor Forms**，包括一个名为`EditForm`的表单组件，各种输入组件，数据注释验证器，验证错误摘要和验证错误标签。

`EditForm`负责编排所有输入组件的状态，通过表单内级联的`EditContext`类的实例。编排来自输入组件和数据注释验证器与此`EditContext`实例的交互。验证摘要和错误消息标签不参与编排，但会注册一些`EditContext`事件以便了解错误。

`EditForm`必须在其`Model`参数中传递其属性必须呈现的对象。值得指出的是，绑定到嵌套属性的输入组件不会被验证，因此`EditForm`必须传递一个扁平化的 ViewModel。`EditForm`创建一个新的`EditContext`实例，将其接收到的对象传递给其构造函数中的`Model`参数，并级联它以便它可以与表单内容交互。

您还可以直接在`EditForm`的`EditContext`参数中传递一个`EditContext`自定义实例，而不是在其`Model`参数中传递对象，这种情况下，`EditForm`将使用您的自定义副本而不是创建一个新实例。通常，当您需要订阅`EditContextOnValidationStateChanged`和`OnFieldChanged`事件时，可以这样做。

当使用**提交**按钮提交`EditForm`且没有错误时，表单会调用其`OnValidSubmit`回调，在这里您可以放置使用和处理用户输入的代码。如果有验证错误，表单会调用其`OnInvalidSubmit`回调。

每个输入的状态反映在自动添加到其中的一些 CSS 类中，即：`valid`，`invalid`和`modified`。您可以使用这些类为用户提供适当的视觉反馈。默认的 Blazor Visual Studio 模板已经为它们提供了一些 CSS。

以下是一个典型的表单：

```cs
<EditForm Model="FixedInteger"OnValidSubmit="@HandleValidSubmit" >
<DataAnnotationsValidator />
<ValidationSummary />
<div class="form-group">
<label for="integerfixed">Integer value</label>
<InputNumber @bind-Value="FixedInteger.Value"
id="integerfixed" class="form-control" />
<ValidationMessage For="@(() => FixedInteger.Value)" />
</div>
<button type="submit" class="btn btn-primary"> Submit</button>
</EditForm> 
```

标签是标准的 HTML 标签，而`InputNumber`是一个专门用于数字属性的 Blazor 组件。`ValidationMessage`是仅在验证错误发生时出现的错误标签。默认情况下，它以`validation-message` CSS 类呈现。与错误消息相关联的属性通过无参数的 lambda 传递给`for`参数，如示例所示。

`DataAnnotationsValidator`组件基于通常的.NET 验证属性（如`RangeAttribute`，`RequiredAttribute`等）添加了验证。您还可以通过继承`ValidationAttribute`类来编写自定义验证属性。

您可以在验证属性中提供自定义错误消息。如果它们包含`{0}`占位符，如果找到`DisplayAttribute`，则将填充为属性显示名称，否则将填充为属性名称。

除了`InputNumber`组件外，Blazor 还支持用于`string`属性的`InputText`组件，用于在 HTML`textarea`中编辑`string`属性的`InputTextArea`组件，用于`bool`属性的`InputCheckbox`组件，以及用于呈现`DateTime`和`DateTimeOffset`的`InputDate`组件。它们的工作方式与`InputNumber`组件完全相同。没有其他 HTML5 输入类型的组件可用。特别是，没有用于呈现时间或日期和时间，或用于使用`range`小部件呈现数字的组件。

您可以通过继承`InputBase<TValue>`类并重写`BuildRenderTree`、`FormatValueAsString`和`TryParseValueFromString`方法来实现渲染时间或日期和时间。`InputNumber`组件的源代码显示了如何做到这一点：[`github.com/dotnet/aspnetcore/blob/15f341f8ee556fa0c2825cdddfe59a88b35a87e2/src/Components/Web/src/Forms/InputNumber.cs`](https://github.com/dotnet/aspnetcore/blob/15f341f8ee556fa0c2825cdddfe59a88b35a87e2/src/Components/We)。您还可以使用*Blazor WebAssembly 的第三方工具*部分中描述的第三方库。

Blazor 还有一个专门用于呈现`select`的组件，其工作方式如下例所示：

```cs
<InputSelect @bind-Value="order.ProductColor">
<option value="">Select a color ...</option>
<option value="Red">Red</option>
<option value="Blue">Blue</option>
<option value="White">White</option>
</InputSelect> 
```

您还可以使用`InputRadioGroup`和`InputRadio`组件将枚举呈现为单选按钮组，如下例所示：

```cs
<InputRadioGroup Name="color" @bind-Value="order.Color">
<InputRadio Name="color" Value="AllColors.Red" /> Red<br>
<InputRadio Name="color" Value="AllColors.Blue" /> Blue<br>
<InputRadio Name="color" Value="AllColors.White" /> White<br>
</InputRadioGroup> 
```

最后，Blazor 还提供了一个`InputFile`组件以及处理和上传文件的所有工具。我们不会在这里介绍，但*进一步阅读*部分包含指向官方文档的链接。

本小节结束了对 Blazor 基础知识的描述；下一节将分析一些高级功能。

# Blazor 高级功能

本节收集了各种 Blazor 高级功能的简短描述，分为子节。由于篇幅有限，我们无法提供每个功能的所有细节，但缺少的细节在*进一步阅读*部分的链接中有所涵盖。我们从如何引用 Razor 标记中定义的组件和 HTML 元素开始。

## 组件和 HTML 元素的引用

有时我们可能需要引用组件以便调用其一些方法。例如，对于实现模态窗口的组件就是这种情况：

```cs
<Modal @ref="myModal">
...
</Modal>
...
<button type="button" class="btn btn-primary" 
@onclick="() => myModal.Show()">
Open modal
</button>
...
@code{
private Modal  myModal {get; set;}
 ...
} 
```

正如前面的例子所示，引用是使用`@ref`指令捕获的。相同的`@ref`指令也可以用于捕获对 HTML 元素的引用。HTML 引用具有`ElementReference`类型，并且通常用于在 HTML 元素上调用 JavaScript 函数，如下一小节所述。

## JavaScript 互操作性

由于 Blazor 不会将所有 JavaScript 功能暴露给 C#代码，并且由于方便利用可用的大量 JavaScript 代码库，有时需要调用 JavaScript 函数。Blazor 通过`IJSRuntime`接口允许这样做，该接口可以通过依赖注入注入到组件中。

一旦有了`IJSRuntime`实例，就可以调用返回值的 JavaScript 函数，如下所示：

```cs
T result = await jsRuntime.InvokeAsync<T>(
"<name of JavaScript function or method>", arg1, arg2....); 
```

不返回任何参数的函数可以像这样被调用：

```cs
awaitjsRuntime.InvokeAsync(
"<name of JavaScript function or method>", arg1, arg2....); 
```

参数可以是基本类型或可以在 JSON 中序列化的对象，而 JavaScript 函数的名称是一个字符串，可以包含表示属性、子属性和方法名称的点，例如`"myJavaScriptObject.myProperty.myMethod"`字符串。

参数也可以是使用`@ref`指令捕获的`ElementReference`实例，在这种情况下，它们在 JavaScript 端作为 HTML 元素接收。

调用的 JavaScript 函数必须在`Index.html`文件中定义，或者在`Index.html`中引用的 JavaScript 文件中定义。

如果您正在编写一个带有 Razor 库项目的组件库，JavaScript 文件可以作为 DLL 库中的资源与 CSS 文件一起嵌入。只需在项目根目录中添加一个`wwwroot`文件夹，并将所需的 CSS 和 JavaScript 文件放在该文件夹或其子文件夹中。之后，这些文件可以被引用为：

```cs
_content/<dll name>/<file path in wwwroot> 
```

因此，如果文件名为`myJsFile.js`，dll 名称为`MyCompany.MyLibrary`，并且文件放在`wwwroot`内的`js`文件夹中，则其引用将是：

```cs
_content/MyCompany.MyLibrary/js/myJsFile.js 
```

如果您的 JavaScript 文件组织为 ES6 模块，您可以避免在`Index.html`中引用它们，并可以直接加载模块，如下所示：

```cs
// _content/MyCompany.MyLibrary/js/myJsFile.js  JavaScript file 
export function myFunction ()
{
...
}
...
//C# code
var module = await jsRuntime.InvokeAsync<JSObjectReference>(
    "import", "./_content/MyCompany.MyLibrary/js/myJsFile.js");
...
T res= await module.InvokeAsync<T>("myFunction") 
```

此外，可以从 JavaScript 代码中调用 C#对象的实例方法，采取以下步骤：

1.  假设 C#方法名为`MyMethod`。请使用`[JSInvokable]`属性装饰`MyMethod`方法。

1.  将 C#对象封装在`DotNetObjectReference`实例中，并通过 JavaScript 调用将其传递给 JavaScript：

```cs
var objRef = DotNetObjectReference.Create(myObjectInstance);
//pass objRef to JavaScript
....
//dispose the DotNetObjectReference
objRef.Dispose() 
```

1.  在 JavaScript 方面，假设 C#对象在名为`dotnetObject`的变量中。然后只需调用：

```cs
dotnetObject.invokeMethodAsync("<dll name>", "MyMethod", arg1, ...).
then(result => {...}) 
```

下一节将解释如何处理内容和数字/日期本地化。

## 全球化和本地化

一旦 Blazor 应用程序启动，应用程序文化和应用程序 UI 文化都将设置为浏览器文化。但是，开发人员可以通过将所选文化分配给`CultureInfo.DefaultThreadCurrentCulture`和`CultureInfo.DefaultThreadCurrentUICulture`来更改它们。通常，应用程序允许用户选择其支持的文化之一，或者仅在支持的情况下接受浏览器文化，否则将回退到支持的文化。实际上，只能支持合理数量的文化，因为所有应用程序字符串必须在所有支持的文化中进行翻译。

一旦设置了`CurrentCulture`，日期和数字将根据所选文化的惯例自动格式化。对于 UI 文化，开发人员必须手动提供包含所有支持的文化中所有应用程序字符串翻译的资源文件。

有两种使用资源文件的方法。使用第一种选项，您创建一个资源文件，比如`myResource.resx`，然后添加所有特定语言的文件：`myResource.it.resx`，`myResource.pt.resx`等。在这种情况下，Visual Studio 会创建一个名为`myResource`的静态类，其静态属性是每个资源文件的键。这些属性将自动包含与当前 UI 文化对应的本地化字符串。您可以在任何地方使用这些静态属性，并且您可以使用由资源类型和资源名称组成的对来设置验证属性的`ErrorMessageResourceType`和`ErrorMessageResourceName`属性，或其他属性的类似属性。这样，属性将使用自动本地化的字符串。

使用第二种选项，您只添加特定语言的资源文件（`myResource.it.resx`，`myResource.pt.resx`等）。在这种情况下，Visual Studio 不会创建与资源文件关联的任何类，您可以将资源文件与在组件中注入的`IStringLocalizer`和`IStringLocalizer<T>`一起使用，就像在 ASP.NET Core MVC 视图中使用它们一样（请参阅*第十五章*的*ASP.NET Core 全球化*部分，*展示 ASP.NET Core MVC*）。

## 认证和授权

在*Routing*子部分中，我们概述了`CascadingAuthenticationState`和`AuthorizeRouteView`组件如何阻止未经授权的用户访问受`[Authorize]`属性保护的页面。让我们深入了解页面授权的工作原理。

在.NET 应用程序中，身份验证和授权信息通常包含在`ClaimsPrincipal`实例中。在服务器应用程序中，当用户登录时，将构建此实例，并从数据库中获取所需的信息。在 Blazor WebAssembly 中，此类信息必须由负责 SPA 身份验证的远程服务器提供。由于有几种方法可以为 Blazor WebAssembly 应用程序提供身份验证和授权，因此 Blazor 定义了`AuthenticationStateProvider`抽象。

身份验证和授权提供程序继承自`AuthenticationStateProvider`抽象类，并覆盖其`GetAuthenticationStateAsync`方法，该方法返回一个`Task<AuthenticationState>`，其中`AuthenticationState`包含身份验证和授权信息。实际上，`AuthenticationState`只包含一个具有`ClaimsPrincipal`的`User`属性。

一旦我们定义了`AuthenticationStateProvider`的具体实现，我们必须在应用程序的`program.cs`文件中将其注册到依赖引擎容器中。

```cs
services.AddScoped<AuthenticationStateProvider, MyAuthStateProvider>(); 
```

在描述了 Blazor 如何使用由注册的`AuthenticationStateProvider`提供的身份验证和授权信息后，我们将回到 Blazor 提供的`AuthenticationStateProvider`的预定义实现。

`CascadingAuthenticationState`组件调用注册的`AuthenticationStateProvider`的`GetAuthenticationStateAsync`方法，并级联返回的`Task<AuthenticationState>`。您可以使用以下方式在组件中定义`[CascadingParameter]`来拦截此级联值：

```cs
[CascadingParameter]
private Task<AuthenticationState>myAuthenticationStateTask { get; set; }
……
ClaimsPrincipal user = (await myAuthenticationStateTask).User; 
```

然而，Blazor 应用程序通常使用`AuthorizeRouteView`和`AuthorizeView`组件来控制用户对内容的访问。

`AuthorizeRouteView`如果用户不满足页面`[Authorize]`属性的要求，则阻止访问页面，否则将呈现`NotAuthorized`模板中的内容。`AuthorizeRouteView`还有一个`Authorizing`模板，当正在检索用户信息时会显示该模板。

`AuthorizeView`可以在组件内部使用，仅向经过授权的用户显示其包含的标记。它包含与`[Authorize]`属性相同的`Roles`和`Policy`参数，您可以使用这些参数来指定用户必须满足的约束以访问内容。

```cs
<AuthorizeView Roles="Admin,SuperUser">
//authorized content
</AuthorizeView> 
```

`AuthorizeView`还可以指定`NotAuthorized`和`Authorizing`模板：

```cs
<AuthorizeView>
<Authorized>
...
</Authorized>
<Authorizing>
        ...
</Authorizing>
<NotAuthorized>
        ...
</NotAuthorized>
</AuthorizeView> 
```

如果在创建 Blazor WebAssembly 项目时添加了授权，将向应用程序的依赖引擎添加以下方法调用：

```cs
builder.Services.AddApiAuthorization(); 
```

此方法添加了一个`AuthenticationStateProvider`，该提取用户信息的方式是从通常的 ASP.NET Core 身份验证 cookie 中提取。由于身份验证 cookie 是加密的，因此必须通过联系服务器公开的端点来执行此操作。此操作是通过本章的*加载和启动应用程序*子章节中看到的`AuthenticationService.js` JavaScript 文件来执行的。服务器端点以 bearer token 的形式返回用户信息，该 token 也可用于验证与服务器的 WEB API 的通信。有关 bearer token 的详细信息，请参见*第十四章*，*使用.NET Core 应用服务导向架构*中的*REST 服务授权和身份验证*和*ASP.NET Core 服务授权*部分。Blazor WebAssembly 通信将在下一子章节中描述。

如果找不到有效的身份验证 cookie，提供程序将创建一个未经身份验证的`ClaimsPrincipal`。这样，当用户尝试访问由`[Authorize]`属性保护的页面时，`AuthorizeRouteView`组件会调用`RedirectToLogin`组件，后者又会导航到`Authentication.razor`页面，并在其`action`路由参数中传递一个登录请求。

```cs
@page "/authentication/{action}"
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
<RemoteAuthenticatorView Action="@Action"  />
@code{
    [Parameter] public string Action { get; set; }
} 
```

`RemoteAuthenticatorView`充当与通常的 ASP.NET Core 用户登录/注册系统的接口，每当它接收要执行的“操作”时，都会将用户从 Blazor 应用程序重定向到适当的 ASP.NET Core 服务器页面（登录、注册、注销、用户资料）。

与服务器通信所需的所有信息都基于名称约定，但可以使用`AddApiAuthorization`方法的`options`参数进行自定义。例如，在那里，您可以更改用户可以注册的 URL，以及 Blazor 用于收集有关服务器设置的端点的地址。此端点位于`BlazorReview.Server->Controller->OidcConfigurationController.cs`文件中。

用户登录后，将被重定向到引起登录请求的 Blazor 应用程序页面。重定向 URL 由`BlazorReview.Client->Shared->RedirectToLogin.razor`组件计算，该组件从`NavigationManager`中提取 URL 并将其传递给`RemoteAuthenticatorView`组件。这次，`AuthenticationStateProvider`能够从登录操作创建的身份验证 cookie 中获取用户信息。

有关身份验证过程的更多详细信息，请参阅*Further reading*部分中的官方文档参考。

下一小节描述了`HttpClient`类和相关类型的 Blazor WebAssembly 特定实现。

## 与服务器的通信

Blazor WebAssembly 支持与*第十四章*，*应用.NET Core 的面向服务的架构*中描述的相同的.NET `HttpClient`和`HttpClientFactory`类。但是，由于浏览器的通信限制，它们的实现是不同的，并依赖于浏览器的**fetch API**。

在*第十四章*，*应用.NET Core 的面向服务的架构*中，我们分析了如何利用`HttpClientFactory`来定义类型化的客户端。您也可以使用完全相同的语法在 Blazor 中定义类型化的客户端。

然而，由于 Blazor 需要在每个请求中发送在身份验证过程中创建的令牌到应用程序服务器，因此通常会定义一个命名客户端，如下所示：

```cs
builder.Services.AddHttpClient("BlazorReview.ServerAPI", client =>
    client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress)
.AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>(); 
```

`AddHttpMessageHandler`添加了一个`DelegatingHandler`，即`DelegatingHandler`抽象类的子类。`DelegatingHandler`的实现重写了其`SendAsync`方法，以处理每个请求和每个相关响应：

```cs
protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request,
        CancellationToken cancellationToken)
{
//modify request 
   ...
HttpResponseMessage= response = await base.SendAsync(
request, cancellationToken);
//modify response
   ...
return response;
} 
```

`BaseAddressAuthorizationMessageHandler`是通过我们在前一节中看到的`AddApiAuthorization`调用添加到依赖注入引擎中的。它将由授权过程生成的令牌添加到每个发送到应用程序服务器域的请求中。如果此令牌已过期或根本找不到，则它会尝试从用户身份验证 cookie 中获取新的令牌。如果此尝试也失败，则会抛出`AccessTokenNotAvailableException`。通常，类似的异常会被捕获并触发重定向到登录页面（默认情况下为`/authentication/{action}`）：

```cs
try
    {
        //server call here
    }
catch (AccessTokenNotAvailableException exception)
    {
        exception.Redirect();
    } 
```

由于大多数请求都是针对应用程序服务器的，并且只有少数调用可能会与 CORS 联系其他服务器，因此`BlazorReview.ServerAPI`命名为`client`也被定义为默认的`HttpClient`实例：

```cs
builder.Services.AddScoped(sp =>
                sp.GetRequiredService<IHttpClientFactory>()
                    .CreateClient("BlazorReview.ServerAPI")); 
```

可以通过向依赖注入引擎请求`HttpClient`实例来获取默认客户端。可以通过定义使用其他令牌的其他命名客户端来处理对其他服务器的 CORS 请求。可以通过首先从依赖注入中获取`IHttpClientFactory`实例，然后调用其`CreateClient("<named client name>")`方法来获取命名客户端。Blazor 提供了用于获取令牌和连接到知名服务的包。它们在*Further reading*部分中的授权文档中有描述。

接下来的部分简要讨论了一些最相关的第三方工具和库，这些工具和库完善了 Blazor 的官方功能，并帮助提高 Blazor 项目的生产力。

# Blazor WebAssembly 的第三方工具

尽管 Blazor 是一个年轻的产品，但其第三方工具和产品生态系统已经相当丰富。在开源、免费产品中，值得一提的是**Blazorise**项目（[`github.com/stsrki/Blazorise`](https://github.com/stsrki/Blazorise)），其中包含各种免费的基本 Blazor 组件（输入、选项卡、模态框等），可以使用各种 CSS 框架（如 Bootstrap 和 Material）进行样式设置。它还包含一个简单的可编辑网格和一个简单的树视图。

另外值得一提的是**BlazorStrap**（[`github.com/chanan/BlazorStrap`](https://github.com/chanan/BlazorStrap)），其中包含了所有 Bootstrap 4 组件和小部件的纯 Blazor 实现。

在所有商业产品中，值得一提的是**Blazor Controls Toolkit**（[`blazor.mvc-controls.com/`](http://blazor.mvc-controls.com/)），这是一个用于实现商业应用程序的完整工具集。它包含了所有输入类型及其在浏览器不支持时的回退；所有 Bootstrap 组件；其他基本组件；以及一个完整的、高级的拖放框架；高级可定制和可编辑的组件，如详细视图、详细列表、网格、树重复器（树视图的泛化）。所有组件都基于一个复杂的元数据表示系统，使用户能够使用数据注释和内联 Razor 声明以声明方式设计标记。

此外，它还包含了额外复杂的验证属性，撤消用户输入的工具，计算发送到服务器的更改的工具，基于 OData 协议的复杂客户端和服务器端查询工具，以及用于维护和保存整个应用程序状态的工具。

还值得一提的是**bUnit**开源项目（[`github.com/egil/bUnit`](https://github.com/egil/bUnit)），它提供了测试 Blazor 组件的所有工具。

接下来的部分将展示如何将所学知识付诸实践，实现一个简单的应用程序。

# 用例 - 在 Blazor WebAssembly 中实现一个简单的应用程序

在本节中，我们将为*WWTravelClub*书籍使用案例实现一个包搜索应用程序。第一小节解释了如何利用我们在*第十五章* *介绍 ASP.NET Core MVC*中已经实现的域层和数据层来设置解决方案。

## 准备解决方案

首先，创建一个**PackagesManagement**解决方案文件夹的副本，我们在*第十五章* *介绍 ASP.NET Core MVC*中创建，并将其重命名为**PackagesManagementBlazor**。

打开解决方案，右键单击 Web 项目（名为**PackagesManagement**）并删除它。然后，转到解决方案文件夹并删除整个 Web 项目文件夹（名为**PackagesManagement**）。

现在右键单击解决方案，然后选择**添加新项目**。添加一个名为**PackagesManagementBlazor**的新的 Blazor WebAssembly 项目。选择**无身份验证**和**ASP.NET Core 托管**。我们不需要身份验证，因为我们将要实现的按位置搜索功能也必须对未注册用户可用。

确保**PackagesManagementBlazor.Server**项目是启动项目（其名称应为粗体）。如果不是，请右键单击它，然后单击**设置为启动项目**。

服务器项目需要引用数据（**PackagesManagementDB**）和域（**PackagesManagementDomain**）项目，请将它们添加为引用。

让我们也将旧 Web 项目的相同连接字符串复制到`PackagesManagementBlazor.Serverappsettings.json`文件中：

```cs
"ConnectionStrings": {
        "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=package-management;Trusted_Connection=True;MultipleActiveResultSets=true"
 }, 
```

这样，我们可以重用我们已经创建的数据库。我们还需要添加与旧 Web 项目添加的相同的 DDD 工具。在项目根目录中添加一个名为`Tools`的文件夹，并将与该书籍关联的 GitHub 存储库的`ch12->ApplicationLayer`文件夹的内容复制到其中。

为了完成解决方案设置，我们只需要通过在`Startup.cs`文件的`ConfigureServices`方法的末尾添加以下代码来将**PackagesManagementBlazor.Server**与域层连接起来：

```cs
services.AddDbLayer(Configuration
                .GetConnectionString("DefaultConnection"),
                "PackagesManagementDB"); 
```

这是我们在旧的 Web 项目中添加的相同方法。最后，我们还可以添加`AddAllQueries`扩展方法，它会发现 Web 项目中的所有查询：

```cs
services.AddAllQueries(this.GetType().Assembly); 
```

由于这是一个仅查询的应用程序，我们不需要其他自动发现工具。

下一小节将解释如何设计服务器端的 REST API。

## 实现所需的 ASP.NET Core REST API

作为第一步，让我们定义在服务器和客户端应用程序之间通信中使用的 ViewModels。它们必须在被两个应用程序引用的**PackagesManagementBlazor.Shared**项目中定义。

让我们从`PackageInfosViewModel` ViewModel 开始：

```cs
using System;
namespace PackagesManagementBlazor.Shared
{
    public class PackageInfosViewModel
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
        public int DurationInDays { get; set; }
        public DateTime? StartValidityDate { get; set; }
        public DateTime? EndValidityDate { get; set; }
        public string DestinationName { get; set; }
        public int DestinationId { get; set; }
        public override string ToString()
        {
            return string.Format("{0}. {1} days in {2}, price: {3}",
                Name, DurationInDays, DestinationName, Price);
        }
    }
} 
```

然后，还要添加一个 ViewModel，它包含要返回给 Blazor 应用程序的所有软件包：

```cs
using System.Collections.Generic;
namespace PackagesManagementBlazor.Shared
{
    public class PackagesListViewModel
    {
        public IEnumerable<PackageInfosViewModel>
            Items { get; set; }
    }
} 
```

现在我们还可以添加我们的查询，通过位置搜索软件包。让我们在**PackagesManagementBlazor.Server**项目的根目录中添加一个`Queries`文件夹，然后添加定义我们查询的接口`IPackagesListByLocationQuery`：

```cs
using DDD.ApplicationLayer;
using PackagesManagementBlazor.Shared;
using System.Collections.Generic;
using System.Threading.Tasks;
namespace PackagesManagementBlazor.Server.Queries
{
    public interface IPackagesListByLocationQuery: IQuery
    {
        Task<IEnumerable<PackageInfosViewModel>>
            GetPackagesOf(string location); 
    }
} 
```

最后，让我们也添加查询实现：

```cs
public class PackagesListByLocationQuery:IPackagesListByLocationQuery
    {
        private readonly MainDbContext ctx;
        public PackagesListByLocationQuery(MainDbContext ctx)
        {
            this.ctx = ctx;
        }
        public async Task<IEnumerable<PackageInfosViewModel>> GetPackagesOf(string location)
        {
            return await ctx.Packages
                .Where(m => m.MyDestination.Name.StartsWith(location))
                .Select(m => new PackageInfosViewModel
            {
                StartValidityDate = m.StartValidityDate,
                EndValidityDate = m.EndValidityDate,
                Name = m.Name,
                DurationInDays = m.DurationInDays,
                Id = m.Id,
                Price = m.Price,
                DestinationName = m.MyDestination.Name,
                DestinationId = m.DestinationId
            })
                .OrderByDescending(m=> m.EndValidityDate)
                .ToListAsync();
        }
    } 
```

我们终于准备好定义我们的`PackagesController`：

```cs
using Microsoft.AspNetCore.Mvc;
using PackagesManagementBlazor.Server.Queries;
using PackagesManagementBlazor.Shared;
using System.Threading.Tasks;
namespace PackagesManagementBlazor.Server.Controllers
{
    [Route("[controller]")]
    [ApiController]
    public class PackagesController : ControllerBase
    {
        // GET api/<PackagesController>/Flor
        [HttpGet("{location}")]
        public async Task<PackagesListViewModel> Get(string location, 
            [FromServices] IPackagesListByLocationQuery query )
        {
            return new PackagesListViewModel
            {
                Items = await query.GetPackagesOf(location)
            };
        }  
    }
} 
```

服务器端代码已经完成！让我们继续定义与服务器通信的 Blazor 服务。

## 在服务中实现业务逻辑

让我们在**PackagesManagementBlazor.Client**项目中添加一个`ViewModels`和一个`Services`文件夹。我们需要的大多数 ViewModel 已经在**PackagesManagementBlazor.Shared**项目中定义。我们只需要一个用于搜索表单的 ViewModel。让我们将其添加到`ViewModels`文件夹中：

```cs
using System.ComponentModel.DataAnnotations;
namespace PackagesManagementBlazor.Client.ViewModels
{
    public class SearchViewModel
    {
        [Required]
        public string Location { get; set; }
    }
} 
```

让我们称我们的服务为`PackagesClient`，并将其添加到`Services`文件夹中：

```cs
namespace PackagesManagementBlazor.Client.Services
{
    public class PackagesClient
    {
        private HttpClient client;
        public PackagesClient(HttpClient client)
        {
            this.client = client;
        }
        public async Task<IEnumerable<PackageInfosViewModel>>
            GetByLocation(string location)
        {
            var result =
                await client.GetFromJsonAsync<PackagesListViewModel>
                    ("Packages/" + Uri.EscapeDataString(location));
            return result.Items;
        }
    }
} 
```

代码很简单！`Uri.EscapeDataString`方法对参数进行 url 编码，以便可以安全地附加到 URL 上。

最后，让我们在依赖注入中注册服务：

```cs
builder.Services.AddScoped<PackagesClient>(); 
```

值得指出的是，在商业应用程序中，我们应该通过`IPackagesClient`接口注册服务，以便能够在测试中模拟它（`.AddScoped<IPackagesClient, PackagesClient>()`）。

一切就绪；我们只需要构建 UI。

## 实现用户界面

作为第一步，让我们删除我们不需要的应用页面，即`Pages->Counter.razor`和`Pages->FetchData.razor`。让我们还从`Shared->NavMenu.razor`中的侧边菜单中删除它们的链接。

我们将把我们的代码放在`Pages->Index.razor`页面中。让我们用以下代码替换此页面的代码：

```cs
@using PackagesManagementBlazor.Client.ViewModels
@using PackagesManagementBlazor.Shared
@using PackagesManagementBlazor.Client.Services
@inject PackagesClient client
@page "/"
<h1>Search packages by location</h1>
<EditForm Model="search"
          OnValidSubmit="Search">
<DataAnnotationsValidator />
<div class="form-group">
<label for="integerfixed">Insert location starting chars</label>
<InputText @bind-Value="search.Location" />
<ValidationMessage For="@(() => search.Location)" />
</div>
<button type="submit" class="btn btn-primary">
        Search
</button>
</EditForm>
@code{
    SearchViewModel search { get; set; } 
= new SearchViewModel();
    async Task Search()
    {
        ...
    }
} 
```

前面的代码添加了所需的`@using`，在页面中注入了我们的`PackagesClient`服务，并定义了搜索表单。当表单成功提交时，它会调用`Search`回调，我们将在其中放置检索所有结果的代码。

现在是时候添加显示所有结果的逻辑并完成`@code`块了。以下代码必须立即放在搜索表单之后：

```cs
@if (packages != null)
{
...
}
else if (loading)
{
    <p><em>Loading...</em></p>
}
@code{
    SearchViewModel search { get; set; } = new SearchViewModel();
    private IEnumerable<PackageInfosViewModel> packages;
    bool loading;
    async Task Search()
    {
        packages = null;
        loading = true;
        await InvokeAsync(StateHasChanged);
        packages = await client.GetByLocation(search.Location);
        loading = false;
    }
} 
```

`if`块中省略的代码负责渲染带有所有结果的表格。在注释了前面的代码之后，我们将显示它。

在使用`PackagesClient`服务检索结果之前，我们删除所有先前的结果并设置`loading`字段，因此 Razor 代码选择`else if`路径，用加载消息替换先前的表。一旦我们设置了这些变量，就必须调用`StateHasChanged`来触发变化检测并刷新页面。在检索到所有结果并且回调返回后，不需要再次调用`StateHasChanged`，因为回调本身的终止会触发变化检测并导致所需的页面刷新。

以下是呈现包含所有结果的表的代码：

```cs
<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th scope="col">Destination</th>
        <th scope="col">Name</th>
        <th scope="col">Duration/days</th>
        <th scope="col">Price</th>
        <th scope="col">Available from</th>
        <th scope="col">Available to</th>
      </tr>
    </thead>
    <tbody>
      @foreach (var package in packages)
      {
        <tr>
          <td>
            @package.DestinationName
          </td>
          <td>
            @package.Name
          </td>
          <td>
            @package.DurationInDays
          </td>
          <td>
            @package.Price
          </td>
          <td>
            @(package.StartValidityDate.HasValue ?
              package.StartValidityDate.Value.ToString("d")
              :
              String.Empty)
          </td>
          <td>
            @(package.EndValidityDate.HasValue ?
              package.EndValidityDate.Value.ToString("d")
              :
              String.Empty)
          </td>
        </tr>
      }
    </tbody>
  </table>
</div> 
```

运行项目并编写 Florence 的初始字符。由于在之前的章节中，我们在数据库中插入了 Florence 作为一个位置，所以应该会出现一些结果！

# 总结

在本章中，您了解了 SPA 是什么，并学习了如何基于 Blazor WebAssembly 框架构建 SPA。本章的第一部分描述了 Blazor WebAssembly 架构，然后解释了如何与 Blazor 组件交换输入/输出以及绑定的概念。

在解释了 Blazor 的一般原则之后，本章重点介绍了如何在提供用户输入的同时，在出现错误时为用户提供足够的反馈和视觉线索。然后，本章简要介绍了高级功能，如 JavaScript 互操作性，全球化，授权认证和客户端-服务器通信。

最后，从书中用户案例中提取的实际示例展示了如何在实践中使用 Blazor 来实现一个简单的旅游套餐搜索应用程序。

# 问题

1.  WebAssembly 是什么？

1.  SPA 是什么？

1.  Blazor `router`组件的目的是什么？

1.  Blazor 页面是什么？

1.  `@namespace`指令的目的是什么？

1.  `EditContext`是什么？

1.  初始化组件的正确位置是什么？

1.  处理用户输入的正确位置是什么？

1.  `IJSRuntime`接口是什么？

1.  `@ref`的目的是什么？

# 进一步阅读

+   Blazor 官方文档可在此处找到：[`docs.microsoft.com/en-US/aspnet/core/blazor/webassembly-lazy-load-assemblies`](https://docs.microsoft.com/en-US/aspnet/core/blazor/webassembly-lazy-load-assemblies)。

+   有关程序集的延迟加载的描述在此处：[`docs.microsoft.com/en-US/aspnet/core/blazor/webassembly-lazy-load-assemblies`](https://docs.microsoft.com/en-US/aspnet/core/blazor/webassembly-lazy-load-assemblies)。

+   Blazor 支持的所有 HTML 事件及其事件参数均列在：[`docs.microsoft.com/en-US/aspnet/core/blazor/components/event-handling?#event-argument-types`](https://docs.microsoft.com/en-US/aspnet/core/blazor/components/event-handling?#event-argument-types)。

+   Blazor 支持与 ASP.NET MVC 相同的验证属性，但不包括`RemoteAttribute`：[`docs.microsoft.com/en-us/aspnet/core/mvc/models/validation#built-in-attributes`](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/validation#built-in-attributes)。

+   `InputFile`组件的描述以及如何使用它可以在这里找到：[`docs.microsoft.com/en-US/aspnet/core/blazor/file-uploads`](https://docs.microsoft.com/en-US/aspnet/core/blazor/file-uploads)。

+   有关 Blazor 本地化和全球化的更多详细信息可在此处找到：[`docs.microsoft.com/en-US/aspnet/core/blazor/globalization-localization`](https://docs.microsoft.com/en-US/aspnet/core/blazor/globalization-localization)。

+   有关 Blazor 身份验证的更多详细信息可在此处找到，以及所有相关 URL：[`docs.microsoft.com/en-US/aspnet/core/blazor/security/webassembly/`](https://docs.microsoft.com/en-US/aspnet/core/blazor/security/webassembly/)。
