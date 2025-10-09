# 2

# 同步和异步数据绑定

在本章中，我们将探讨 Blazor 中数据绑定的各个方面。**绑定**是现代网络开发的一个基石。我们将从绑定值与 DOM 元素的基本原理开始。然后，我们将进一步探讨绑定特定的 DOM 事件，确保你的 Blazor 应用程序高度交互和响应。大多数商业应用程序都需要与外部数据提供者集成。因此，你必须执行异步操作。我们将探讨如何将它们与绑定配对。

此外，我们将介绍自定义获取器和设置器，以提供更大的数据处理灵活性。我们还将介绍**bind-Value**绑定模式，它应该简化你大多数的绑定场景。最后，我们将实现一个与外部数据提供者无缝绑定的商业场景。这在实现搜索模块或数据持久化机制时非常有用。

然而，我们将完全跳过构建表单或使用可以简化绑定的 Blazor 原生组件——我们将在*第六章*中介绍，该章节涵盖*构建* *交互式表单*。

到本章结束时，你将深入理解 Blazor 中的数据绑定。这将使你能够构建更动态和用户友好的网络应用程序。你将要实现的全部交互性都将归结为绑定。你将手头有食谱来处理那些同步和异步场景。

下面是我们将在本章中介绍的食谱列表：

+   使用标记元素绑定值

+   绑定到特定的 DOM 事件

+   绑定后执行异步操作

+   自定义获取和设置绑定逻辑

+   使用**bind-Value**模式简化绑定

+   使用外部数据提供者进行绑定

# 技术要求

由于我们将探讨 Blazor 和 Web 开发的基本概念，你不需要任何付费插件或额外工具。然而，你需要以下内容：

+   一个现代 IDE（你选择的）

+   一个现代网络浏览器（支持**WebAssembly**）

+   一个 Blazor 项目

你接下来将看到的全部代码示例（和数据样本）可以在以下 GitHub 仓库中找到：

[`github.com/PacktPublishing/Blazor-Web-Development-Cookbook/tree/main/BlazorCookbook.App.Client/Chapters/Chapter02`](https://github.com/PacktPublishing/Blazor-Web-Development-Cookbook/tree/main/BlazorCookbook.App.Client/Chapters/Chapter02)

# 使用标记元素绑定值

在本食谱中，我们介绍了 Blazor 应用程序中数据绑定的基础概念。这个特性连接了用户界面与应用程序的数据或状态之间的差距。对这个概念有深入的理解将使你能够提升你项目的交互性。我们首先从掌握基础知识开始。

让我们绑定一个简单的文本字段到一个后端变量，以查看数据从用户界面流向后端。

## 准备工作

在你开始绑定之前，创建一个 **Recipe01** 目录 – 这将是你的工作目录。

## 如何做到这一点…

按照以下步骤绑定 C# 值与标记元素：

1.  添加一个可路由的 **IntroduceYourself** 组件，以 **InteractiveWebAssembly** 模式渲染：

    ```cs
    @page "/ch02r01"
    @rendermode InteractiveWebAssembly
    ```

1.  在 **IntroduceYourself** 的 **@code** 块中，初始化一个 **User** 变量以存储用户输入的值：

    ```cs
    @code {
        protected string User = string.Empty;
    }
    ```

1.  在 **IntroduceYourself** 标记中，添加一个号召性用语和一个带有 **@bind** 属性的输入字段，该属性分配给 **User** 变量：

    ```cs
    <h3>What's your name?</h3>
    <input class="form-control w-50" @bind="@User" />
    ```

1.  作为 **IntroduceYourself** 标记的一部分，构建一些逻辑以在用户填写输入字段后动态显示问候语：

    ```cs
    @if (string.IsNullOrWhiteSpace(User)) return;
    <hr />
    <h1>Hello @User!</h1>
    ```

## 它是如何工作的…

我们以 Blazor 开发中的一个常规步骤开始这个食谱 – 创建一个新的组件。在 *步骤 1* 中，我们执行 Blazor 开发中的一个常规步骤 – 创建一个新的 **IntroduceYourself** 组件并声明其渲染模式。在我们的案例中，我们选择 **InteractiveWebAssembly**。在 *步骤 2* 中，我们跳转到 **IntroduceYourself** 的 **@code** 块并初始化一个 **User** 后备字段 – 这对于我们的数据绑定操作至关重要。

实际的绑定魔法从 *步骤 3* 开始。我们转向 **IntroduceYourself** 标记，并在号召性用语旁边嵌入一个 **input** 元素，并首次利用 Blazor 的原生 **@bind** 属性。**@bind** 属性启用双向数据绑定 – 不仅将输入值分配给 **User** 后备字段，还确保 UI 更新以反映这一变化。在 *步骤 4* 中，我们添加 **IntroduceYourself** 标记的另一个部分 – 当用户填写输入时，我们显示当前的 **User** 值。这将帮助我们可视化绑定的行为。

由 **@bind** 属性促进的双向绑定简化了用户交互的实现。对于大多数简单的绑定情况，这是一个首选方法，其中不需要执行额外的逻辑。值得注意的是，默认情况下，此绑定发生在用户退出输入框时，但这完全可定制。我们将在下一个食谱中探索和阐明这种行为及其背后的机制。

# 绑定到特定的 DOM 事件

现在，我们将深入探讨在 Blazor 应用程序中针对用户交互的精确和高效处理。虽然一般数据绑定至关重要，但将特定操作绑定到特定的 **DOM 事件** 可以将应用程序的交互性提升到下一个层次。这种方法允许更受控和响应的用户体验。你可以直接将事件（如点击或按键）链接到相应的 C# 方法或操作。你将学习如何识别和绑定到这些事件以及哪些事件是可以绑定的。

让我们实现一个简单的文本字段，但触发绑定是在用户输入时，而不是他们退出字段时，这是默认行为。

## 准备工作

在探索特定事件的绑定之前，请执行以下操作：

+   创建一个 **Recipe02** 目录 – 这将是你的工作目录

+   从*绑定值与标记元素*菜谱复制您的**IntroduceYourself**组件，或者从 GitHub 仓库的**Chapter02** / **Recipe01**目录复制其实现。

## 如何实现...

要绑定到特定的 DOM 事件，请遵循以下说明：

1.  导航到**IntroduceYourself**组件中的**@code**块。

1.  除了现有的**用户**变量外，初始化一个新的**问候**变量。我们将使用它来保存对用户的问候：

    ```cs
    protected string User = string.Empty,
                     Greeting = string.Empty;
    ```

1.  实现了**IsGreetingReady**和**IsUserFilled**方法，这些方法允许您评估**用户**和**问候**变量的状态：

    ```cs
    private bool IsGreetingReady
        => !string.IsNullOrWhiteSpace(Greeting);
    private bool IsUserFilled
        => !string.IsNullOrWhiteSpace(User);
    ```

1.  添加一个**SayHello()**方法来准备问候信息：

    ```cs
    private void SayHello() => Greeting = $"Hello {User}";
    ```

1.  在**IntroduceYourself**标记中，通过添加**@** **bind:event="oninput"**来扩展输入字段绑定：

    ```cs
    <input class="form-control w-50"
           @bind="@User"
           @bind:event="oninput" />
    ```

1.  通过将**SayHello()**行为附加到**@** **onfocusout**事件来进一步扩展输入字段绑定：

    ```cs
    <input class="form-control w-50" @bind="@User"
           @bind:event="oninput"
           @onfocusout="@SayHello" />
    ```

1.  从之前的实现中移除对**用户**值的任何现有检查，并基于状态检查方法实现条件问候渲染：

    ```cs
    @if (IsGreetingReady)
    {
        <h1>@Greeting</h1>
        return;
    }
    @if (IsUserFilled)
    {
        <h1>Introducing @User...</h1>
    }
    ```

## 它是如何工作的...

在这个菜谱中，我们正在增强**IntroduceYourself**组件，使其更具交互性和吸引力。我们的目标是使组件能够问候用户并显示用户名输入的进度。

在*第 1 步*中，我们导航到**IntroduceYourself**组件，在*第 2 步*中，我们直接进入**@code**块并初始化一个额外的**问候**变量来存储生成的问候信息。这为我们的动态用户交互奠定了基础。在*第 3 步*中，我们引入了**IsUserFilled**和**IsGreetingReady**这两个无参方法，它们检查**用户**和**问候**变量的状态，并允许我们简化标记代码并使我们的逻辑更易于阅读。继续到*第 4 步*，我们添加了另一个方法——**SayHello()**。当用户完成他们的名字输入时，我们将调用**SayHello()**来生成问候并将其分配给**问候**变量。这为用户体验增添了个性化元素。

*第 5 步*是我们控制绑定逻辑的地方。您可以在适用于脚本框架的任何事件上触发绑定，例如**oncopy**、**onpaste**或**onblur**。我们选择了**oninput**事件作为此示例。我们使用**@bind:event="oninput"**覆盖默认绑定事件，并启用**用户**值在用户输入时实时更新。注意**@bind:event**属性的语法是区分大小写的，因为 Razor 是一个区分大小写的语言框架。在*第 6 步*中，我们将交互性提升到一个新的层次。我们使用原生的 Blazor 引用到**@onfocusout**事件，当用户从输入字段导航离开时，我们调用**SayHello()**方法，从而生成问候信息。

最后，在 *步骤 7* 中，我们实现了一些渲染逻辑，以确保只有当用户完成输入时，问候语才会显示。在此之前，将生成 **Introducing...** 消息。这样，我们提供了一个清晰的 UI 交互和进度指示器。

## 还有更多...

重要的是要知道，Blazor 允许您拦截所有事件参数。您可以通过添加一个与触发事件对应的事件参数类型匹配的参数来实现这一点。

例如，让我们考虑 **DragEventArgs**。当用户将元素拖入或拖动到您的应用程序中时，您可以拦截光标位置和任何活动的键盘组合。

拦截拖拽事件的简单方法如下：

```cs
public void OnDragging(DragEventArgs args) { /*...*/ }
```

类似地，使用 **InputFileChangeEventArgs**，您可以访问用户已上传的文件的详细信息。它还公开了 **IBrowserFile** 对象，您可以使用它将内容流式传输到您的服务器。

下面是如何拦截用户上传文件细节的方法：

```cs
public void OnFileInput(InputFileChangeEventArgs args)
{
    var droppedFile = args.File;
    // ...
}
```

拦截特定事件及其细节的能力为在 Blazor 应用程序中实现更高级和细致的事件处理提供了可能性。

# 绑定后执行异步操作

异步操作在现代网络应用中至关重要，尤其是在处理如从 **API** 或数据库获取数据等数据密集型操作时。同时，当您的应用程序正在执行长时间运行的任务时，保持其响应性也非常关键。在本食谱中，我将指导您在处理异步任务时进行数据绑定。您将学习如何集成异步操作，以实现非阻塞的 UI 更新和更流畅的用户体验。

让我们在一个简单的输入字段上启用 **自动完成** 功能，并在用户输入他们的名字时生成一个建议列表。

## 准备工作

在深入到自动完成实现之前，请执行以下操作：

+   创建一个 **Recipe03** 目录——这将作为您的工作目录。

+   从 *绑定到特定的 DOM 事件* 食谱复制您的 **IntroduceYourself** 组件，或者从 GitHub 仓库的 **Chapter02**/ **Recipe02** 目录复制其实现。

+   在 **Recipe03** 旁边，从 GitHub 仓库复制 **Chapter02**/ **Data** 目录，其中包含本食谱所需的 **SuggestionsApi** 类。

## 如何实现...

按照以下步骤在数据绑定后触发异步操作：

1.  打开您的应用程序的 **Program** 文件，并注册 **SuggestionsApi** 服务以启用与 API 的通信：

    ```cs
    builder.Services.AddTransient<SuggestionsApi>();
    ```

1.  导航到 **IntroduceYourself** 组件的 **@code** 块，清理该部分，以确保只有 **User** 变量仍然存在：

    ```cs
    @code {
        protected string User = string.Empty;
    }
    ```

1.  在 **@code** 块内部，注入 **SuggestionsApi** 服务并初始化一个 **Suggestions** 集合，该集合将保存自动完成的结果：

    ```cs
    [Inject] private SuggestionsApi Api { get; init; }
    protected IList<string> Suggestions = [];
    ```

1.  仍然在**@code**块中，实现一个新的异步**AutocompleteAsync()**方法，负责调用 API 并根据用户输入更新**建议**集合：

    ```cs
    private async Task AutocompleteAsync()
    {
        Suggestions = string.IsNullOrWhiteSpace(User) ?
            [] : await Api.FindAsync(User);
        await InvokeAsync(StateHasChanged);
    }
    ```

1.  在**自我介绍**标记中，找到**输入**字段，并将**@onfocusout**赋值替换为触发**AutocompleteAsync()**方法的**@bind:after**属性：

    ```cs
    <input class="form-control w-50" @bind=@User
           @bind:event="oninput"
           @bind:after="@AutocompleteAsync" />
    ```

1.  在**输入**下方，在**<hr />**分隔符下，清除现有的问候和介绍部分，并在建议当前不可用时构建快速返回：

    ```cs
    <hr />
    @if (!Suggestions.Any()) return;
    ```

1.  最后，在**自我介绍**标记的末尾，渲染从 API 响应中接收到的**建议**集合。

    ```cs
    <h5>Did you mean?</h5>
    @foreach (var name in Suggestions)
    {
        <div>@name</div>
    }
    ```

## 它是如何工作的…

在**步骤 1**中，我们将**SuggestionsApi**服务添加到我们应用程序的**依赖注入**容器中。这使得**SuggestionsApi**在整个应用程序中都可以用于注入，确保我们可以在需要的地方使用它。类似于其他.NET Web 框架，Blazor 可以管理具有三种生命周期——单例、作用域和瞬态——的服务：

+   **单例**服务在每个应用程序中只创建一次，并在所有组件和请求之间共享。

+   **瞬态**服务每次请求时都会被重新创建，这使得它们非常适合轻量级、无状态的服务（例如 API 集成）。

+   **作用域**服务稍微复杂一些。在客户端应用程序中，作用域服务通常表现得像单例服务，因为没有连接上下文来区分会话。然而，当**OwningComponentBase**组件介入时，情况就改变了。**OwningComponentBase**是一个基类，确保组件及其依赖项在 Blazor 销毁组件实例时能够优雅地释放。

在**步骤 2**中，我们继续到**自我介绍**组件，并清除问候实现，因此只剩下**用户**变量。在**步骤 3**中，我们将**SuggestionsApi**注入到我们的**自我介绍**组件中，并初始化一个**建议**集合来存储我们从 API 调用中获取的结果。进入**步骤 4**，我们实现**AutocompleteAsync()**异步方法，在简化标记逻辑中起着至关重要的作用。**AutocompleteAsync()**检查用户是否已输入任何内容，并调用 API 的**FindAsync()**方法来获取建议；否则，它将短路操作，返回一个空数组。

在**步骤 5**中，我们增强了**IntroduceYourself**的标记。我们通过移除**@onfocusout**事件并替换为**@bind:after**指令来细化输入字段的绑定逻辑，该指令在绑定完成后立即调用**AutocompleteAsync()**方法。通过这种设置，我们确保每次用户修改输入时，都会从 API 请求一组新的自动完成建议。最后，在**步骤 6**和**步骤 7**中，我们引入了额外的标记来显示自动完成操作的结果。我们渲染一个找到的名称列表，以用户输入的相同字符集开始，如果 API 返回任何结果。

## 更多内容…

在我们的示例中，我们使用了**[Inject]**属性将**SuggestionsApi**注入到组件中。然而，Blazor 还提供了其他服务注入方法。让我们回顾一下所有这些方法：

+   当你在**.** **razor**文件的**@code**块中注入服务时，你通常会使用**[Inject]**属性：

    ```cs
    @code {
        [Inject] private SuggestionsApi Api { get; init; }
    }
    ```

+   如果你采用代码后置（code-behind）方法，你也会使用**[Inject]**属性，其中你将 Blazor 组件的逻辑分离到一个**.** **cs**文件中：

    ```cs
    public partial class IntroduceYourself
    {
        [Inject] private SuggestionsApi Api { get; init; }
    }
    ```

+   当以代码后置方式工作时，你还可以利用构造函数注入模式，并且根本不需要使用任何属性：

    ```cs
    public partial class IntroduceYourself(
        SuggestionsApi Api) { }
    ```

+   最后，Blazor 允许在组件的标记中直接注入服务 - 在这种情况下，你会使用**@** **inject**指令：

    ```cs
    @page "/ch02r01"
    @inject SuggestionsApi Api
    ```

所有这些方法都服务于相同的目的，但适应了不同的编码风格和偏好。没有哪一个比另一个更好，所以选择最适合你代码库结构和组织的选项。

# 自定义获取和设置绑定逻辑

数据绑定不仅仅是将 UI 元素连接到数据源。它还涉及到数据的检索和更新方式。自定义这些**get**和**set**操作可以使状态管理和数据流更加灵活。在本食谱中，我将指导你了解显式使用**get**和**set**的注意事项，以及在设置值时执行异步逻辑。这些机制将允许你简化代码，并更好地控制 Blazor 应用程序的交互性。

让我们实现一个简单的文本字段，具有显式的**get**和**set**操作，这样我们就可以在绑定开始时执行额外的异步逻辑。

## 准备工作

这次，我们将通过简化准备工作来为即将到来的食谱做准备：

+   创建一个**Recipe04**目录 - 这将是你的工作目录

+   从*执行绑定后的异步操作*食谱中复制你的**IntroduceYourself**组件，或者从 GitHub 仓库的**Chapter02**/ **Recipe03**目录中复制其实现。

+   在**Recipe04**旁边，从 GitHub 仓库复制**Chapter02**/ **Data**目录，其中包含本食谱所需的对象

## 如何做到这一点…

要自定义**get**和**set**绑定逻辑，请遵循以下说明：

1.  在**IntroduceYourself**组件的标记中定位**input**字段。保留**@bind:event**指令，但将其他指令替换为自定义的**@bind:get**和**@** **bind:set**逻辑：

    ```cs
    <input class="form-control w-50" @bind:event="oninput"
           @bind:get="@User"
           @bind:set="@AutocompleteAsync" />
    ```

1.  导航到**IntroduceYourself**的**@code**块，并调整**AutocompleteAsync()**方法以符合由**@** **bind:set**指令强制执行的 setter 模式：

    ```cs
    private async Task AutocompleteAsync(string value)
    {
        User = value;
        Suggestions = string.IsNullOrWhiteSpace(User) ?
            [] : await Api.FindAsync(User);
        await InvokeAsync(StateHasChanged);
    }
    ```

## 它是如何工作的…

在*步骤 1*中，我们更新了**input**字段的绑定逻辑。我们保留**oninput**事件，因为我们希望我们的方法在用户输入时执行，但我们用**@bind:get**和**@bind:set**替换了其他指令。我们使用**@bind:get**来指定 Blazor 应从哪里检索输入值——在我们的例子中，这只是对**User**变量的引用。通过**@bind:set**指令，我们定义了当 Blazor 将输入值绑定到组件状态时要执行的逻辑。那就是我们触发**AutocompleteAsync()**方法的时候。此时，你的 IDE 应该会突出显示一个编译错误。

通常，C#中的**set**方法默认接收一个**value**对象：

```cs
private string _userName;
public string UserName
{
    get => _userName;
    set => _userName = value;
}
```

在 Blazor 中，用于**@bind:set**的方法必须遵循类似的模式。区别在于我们可以定义传入的**value**对象的名称，从而提供更大的控制和清晰度。因此，在*步骤 2*中，我们通过添加**value**参数扩展**AutocompleteAsync()**，并保持与传统的**get** - **set**模式一致。请特别注意参数类型的一致性。由于我们将**input**绑定到类型为**string**的**User**对象，因此**AutocompleteAsync()**方法也期望一个字符串。例如，如果你将绑定到类型为**int**的**Age**变量，你的绑定方法需要接受一个**int**参数。

## 还有更多…

那么，为什么**@bind:after**和**@bind:set**都存在，尽管它们几乎做的是同一件事？每个事件的执行时间至关重要。当你需要使用默认的绑定操作**@bind:after**执行额外的逻辑，通常是异步的，你应该选择它，因为它可以避免你手动持久化传入值的复杂性。另一方面，**@bind:set**提供了更多的灵活性，因为它在设置绑定值时执行。它允许将验证逻辑（包括异步操作）集成到绑定过程中。关键的是，**@bind:set**允许你在任何验证结果的基础上评估传入的值，并在它成为组件状态的一部分之前决定是否丢弃或接受它。这两个功能对于确保数据完整性和在 Blazor 应用程序中实现复杂的验证机制都非常有价值。

# 使用 bind-Value 模式简化绑定

**bind-Value**模式是简化 UI 元素与数据属性链接过程的一个变革性工具。它通过减少与双向数据绑定相关的样板代码，提高了代码的清晰性和简洁性。在本食谱中，我将通过一个实际示例指导您，展示该模式在创建更易于维护和简单的代码中的实用性，从而最终提升您的开发工作流程。

让我们实现一个组件，它允许直接绑定到其参数，其结构类似于标准绑定到 HTML 元素。

## 准备工作

在开始实现启用**bind-Value**模式的组件之前，执行以下操作：

+   创建一个**Recipe05**目录——这将成为您的工作目录

+   在**Recipe05**旁边，从 GitHub 仓库复制**Chapter02**/ **Data**目录，其中包含**SkillLevel**和**DataSeed**，这是本食谱所需的。

## 如何做到这一点…

按照以下步骤应用**bind-Value**绑定模式：

1.  创建一个**IntroductionForm**组件并在顶部添加所需的程序集引用：

    ```cs
    @using BlazorCookbook.App.Client.Chapters.Chapter02.Data
    ```

1.  在**IntroductionForm**的**@code**块中，声明一个**string**参数用于**Name**。在其旁边，添加**EventCallback<string>**以将**Name**值的更改通知：

    ```cs
    [Parameter]
    public string Name { get; set; }
    [Parameter]
    public EventCallback<string> NameChanged { get; set; }
    ```

1.  类似地，在**Name**和**NameChanged**下方，添加一个**Skill**和一个**EventCallback<SkillLevel>**参数来管理技能级别状态：

    ```cs
    [Parameter]
    public SkillLevel Skill { get; set; }
    [Parameter]
    public EventCallback<SkillLevel> SkillChanged { get; set; }
    ```

1.  仍然在**@code**块中，实现一个调用**NameChanged**回调并传播**Name**值更改的方法：

    ```cs
    private Task OnNameChanged()
        => NameChanged.InvokeAsync(Name);
    ```

1.  最后，在**@code**块中添加另一个方法，处理**Skill**值的更改，该方法检索**ChangeEventArgs**参数并根据解析的值和底层的**DataSeed.SkillLevels**数据源设置技能级别：

    ```cs
    private Task OnSkillChanged(ChangeEventArgs args)
    {
        var id = int.Parse(args.Value.ToString());
        var skill = DataSeed.SkillLevels
            .SingleOrDefault (it => it.Id == id);
        return SkillChanged.InvokeAsync(skill);
    }
    ```

1.  在**IntroductionForm**标记中，在**@using**下方添加一个用户可以输入其姓名的部分。声明绑定将在**oninput**事件上发生，并在绑定完成后触发**OnNameChanged**方法：

    ```cs
    <h5>What's your name?</h5>
    <input class="form-control w-50 mb-1" @bind="@Name"
           @bind:event="oninput"
           @bind:after=@OnNameChanged />
    ```

1.  在下方添加另一个标记部分，允许您选择用户的技能级别。利用**DataSeed.SkillLevels**数据源来填充选择选项，并确保选择更改的绑定与**onchange**事件一起发生：

    ```cs
    <h5>What's your skill level?</h5>
    <select class="form-control w-50 mb-1"
            @onchange="@OnSkillChanged">
        <option value="0">-</option>
        @foreach (var level in DataSeed.SkillLevels)
        {
            <option value="@level.Id">
                @level.Title
            </option>
        }
    </select>
    ```

1.  创建一个新的**IntroduceYourself**可路由组件，以**InteractiveWebAssembly**模式渲染，并引用数据对象程序集：

    ```cs
    @using BlazorCookbook.App.Client.Chapters.Chapter02.Data
    @page "/ch02r05"
    @rendermode InteractiveWebAssembly
    ```

1.  在**@code**块中，初始化**Name**和**Skill**变量以捕获用户输入并生成问候语：

    ```cs
    protected string Name { get; set; }
    protected SkillLevel Skill { get; set; }
    ```

1.  仍然在**@code**块中，实现一个**IsGreetingReady**方法，允许您检查问候语是否准备好渲染：

    ```cs
    private bool IsGreetingReady
        => !string.IsNullOrWhiteSpace(Name)
        && Skill is not null;
    ```

1.  在**IntroduceYourself**标记中，嵌入**IntroductionForm**并利用**bind-Value**模式动态绑定**Name**和**Skill**参数到相应的目标变量：

    ```cs
    <IntroductionForm @bind-Name="@Name"
                      @bind-Skill="@Skill" />
    ```

1.  通过添加一个部分分隔符和条件渲染用户问候，当它准备好时，完成**IntroduceYourself**标记：

    ```cs
    <hr />
    @if (!IsGreetingReady) return;
    <h5>Welcome @Name on level @Skill.Title!</h5>
    ```

## 它是如何工作的…

在*第 1 步*中，我们创建了一个**IntroductionForm**组件，并在顶部引用了包含样本数据的汇编。在*第 2 步*中，我们定义了一对类型为**string**和**EventCallback<string>**的参数，我们将它们绑定到表单上，并且故意将它们命名为**Name**和**NameChanged**。在*第 3 步*中，我们添加了一对类似的参数来处理技能水平，并将它们命名为**Skill**和**SkillChanged**。类型为**T**的值与其匹配的**EventCallback<T>**处理器的配对构成了**bind-Value**模式的基础。类似于隐式识别**ChildContent**，当使用**RenderFragment**时，Blazor 的代码生成器会识别**T**和**EventCallback<T>**并编译**@bind-Value**指令。在*第 4 步*中，我们声明了一个**OnNameChanged()**方法来调用预期的**EventCallback**并传播**Name**的变化。为了处理**Skill**参数的值变化，我们在*第 5 步*中实现了一些更复杂的逻辑。我们拦截了一个携带所选选项值的**ChangeEventArgs**对象，我们使用它从**DataSeed.SkillLevels**集合中检索相应的**SkillLevel**对象。然后我们将此值传递给预期的**EventCallback**处理器。

在逻辑设置到位后，在*第 6 步*中，我们继续进行到**IntroductionForm**标记。我们添加了一个简单的**输入**字段并将其绑定到**Name**参数。我们还连接了**OnNameChanged()**方法，以便在绑定完成后触发。在*第 7 步*中，我们构建了一个**选择**字段，允许用户选择他们的技能水平。我们渲染了一个中性的选项，显示**-**，以及来自**DataSeed.SkillLevels**集合的技能选项。我们将**选择**字段的**@onchanged**事件连接到**OnSkillChanged()**方法。

在*第 8 步*中，我们创建了一个可路由的**IntroduceYourself**组件，以**InteractiveWebAssembly**模式渲染。我们还引用了示例数据汇编，利用了**@using**指令。在*第 9 步*中，我们在**IntroduceYourself**组件内初始化一个**@code**块，并声明**Name**和**Skill**后端属性以进行绑定和生成用户的问候。在*第 10 步*中，我们实现了一个简单的**IsGreetingReady**方法，该方法检查**Name**和**Skill**是否具有有意义的值，并且可以安全地生成问候。

在*步骤 11*中，我们跳转到**IntroduceYourself**标记并见证**bind-Value**模式的作用。由于**IntroductionForm**公开了具有模式匹配事件回调的**Name**和**Skill**参数，我们可以使用**@bind-Name**和**@bind-Skill**动态绑定它们。您的 IDE 将自动识别此模式，甚至可能建议这些指令。我们在*步骤 12*中通过添加基于当前**Name**和**Skill**值的问候消息的条件渲染来最终确定标记。

**bind-Value**模式封装了组件内的绑定逻辑和验证，极大地简化了单元测试，并增强了父组件的整洁性和健壮性。它只需要父组件提供后端变量，从而简化了开发过程。它甚至更强大，因为我们所涵盖的所有绑定指令（**@bind:after**，**@bind:get**，**@bind:set**），都可以与**bind-Value**模式搭配使用。

# 与外部数据提供者绑定

当构建与外部数据源交互的 Web 应用程序时，通常会在用户输入时触发 API 调用。然而，这可能导致请求洪水，使 API 过载并降低用户体验。为了应对这一挑战，我们将实现输入**节流**——一种根据用户输入调节请求发送速率的技术。在本食谱中，我将指导您在 Blazor 组件中设置输入节流，确保高效且负责任地使用外部 API。您将创建更健壮、用户友好的应用程序，能够处理大量用户交互而不会压垮您的数据提供者。

让我们实现一个简单的文本字段，该字段使用节流来限制对外部 API 的调用，并无缝等待用户完成输入。

## 准备工作

在开始节流实现之前，请执行以下操作：

+   创建一个**Recipe06**目录——这将作为您的工作目录

+   在**Recipe06**旁边，从 GitHub 仓库复制**Chapter02** / **Data**目录，其中包含本食谱所需的**SuggestionsApi**类

+   从 GitHub 仓库的**Chapter02** / **Recipe03**目录复制**IntroduceYourself**组件

## 如何操作...

在调用外部 API 时实现节流，请按照以下步骤操作：

1.  打开您应用程序的**程序**文件，并将**SuggestionsApi**服务注册以启用与 API 的通信：

    ```cs
    builder.Services.AddTransient<SuggestionsApi>();
    ```

1.  使用**@implements**指令在**@** **rendermode**指令下方增强**IntroduceYourself**组件的实现**IDisposable**接口：

    ```cs
    @rendermode InteractiveWebAssembly
    @implements IDisposable
    ```

1.  在**IntroduceYourself**组件的**@code**块内部，声明**Timer**变量和两个**TimeSpan**变量——用于节流和整体超时：

    ```cs
    private Timer _debounceTimer;
    private readonly TimeSpan
        _throttle = TimeSpan.FromMilliseconds(500),
        _timeout = TimeSpan.FromMinutes(1);
    ```

1.  仍然在**@code**块内，实现一个使用**Timer**和**TimeSpan**变量进行节流的**OnUserInput()**方法，该方法使用代理逻辑来封装在**AutocompleteAsync()**方法内的 API 请求：

    ```cs
    private void OnUserInput()
    {
        _debounceTimer?.Dispose();
        _debounceTimer = new Timer(
            _ => InvokeAsync(AutocompleteAsync),
            null, _throttle, _timeout);
    }
    ```

1.  为了完成**@code**块，实现**IDisposable**接口所需的**Dispose()**生命周期方法，并显式地销毁**_debounceTimer**实例：

    ```cs
    public void Dispose() => _debounceTimer?.Dispose();
    ```

1.  在**IntroduceYourself**标记中，更新**@bind:after**指令以调用**OnUserInput()**方法，其中我们添加了节流逻辑：

    ```cs
    <input class="form-control w-50" @bind=@User
           @bind:event="oninput"
           @bind:after="@OnUserInput" />
    ```

## 它是如何工作的…

在**步骤 1**中，我们在应用程序的依赖注入容器中注册了**SuggestionsApi**服务。如果你正在跟随整个章节，你可能已经有了**SuggestionsApi**。

在**步骤 2**中，借助**@implements**指令，我们使用**IDisposable**模式增强了**IntroduceYourself**组件。在 Blazor 中，**IDisposable**接口用于在销毁组件时释放未管理资源或断开事件处理器。**IDisposable**需要实现一个**Dispose()**方法，Blazor 将在组件从 UI 中移除时自动调用该方法，确保适当的清理并防止内存泄漏。如果没有实现**Dispose()**方法，你的 IDE 将突出显示编译错误。我们将在后续步骤中解决这个问题。

在**步骤 3**中，我们为节流逻辑打下基础。在**IntroduceYourself**组件的**@code**块中，我们初始化了两个关键变量：**_throttle**，它定义了用户交互和 API 调用之间的空闲时间（在我们的示例中为 500 毫秒），以及**_timeout**，它为外部通信设置了一个总超时时间。我们还声明了一个**_debounceTimer**变量，其类型为**Timer**，这是管理 API 调用频率的骨干。**Timer**类封装了一个调度器，它将方法的执行延迟指定的时间，这使得它非常适合节流。在**步骤 4**中，仍然在**@code**块内，我们实现了一个具有节流代理逻辑的**OnUserInput()**方法。首先，我们通过销毁**_debounceTimer**实例来停止当前计划的操作，以避免任何重叠执行。接下来，我们实例化一个新的**Timer**对象，将现有的**AutocompleteAsync()**方法包装在**InvokeAsync()**方法中以确保线程安全。由于我们不需要在定时器调用之间维护任何状态，我们传递**null**作为状态对象，并使用**_throttle**和**_timeout**变量完成**_debounceTimer**的初始化。在**步骤 5**中，我们通过实现缺失的**Dispose()**方法来完成**@code**块，以优雅地销毁**_debounceTimer**实例。现在应该没有编译错误了。

最后，在*第 6 步*中，我们稍微更新了**IntroduceYourself**标记中的**输入**字段。而不是在绑定完成后调用**AutocompleteAsync()**，我们将**@bind:after**属性更新为调用**OnUserInput()**方法，其中包含我们的节流逻辑。现在，每个按键都通过节流机制，优化应用程序的响应速度并减少对外部 API 的负载。
