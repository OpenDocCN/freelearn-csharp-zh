# 第二十章：软件测试自动化

在前面的章节中，我们讨论了单元测试和集成测试在软件开发中的重要性，以及它们如何确保代码库的可靠性。我们还讨论了单元和集成测试是所有软件生产阶段的组成部分，并且每次修改代码库时都会运行。

还有其他重要的测试，称为**功能**/**验收**测试。它们只在每个冲刺结束时运行，以验证冲刺的输出实际上是否满足与利益相关者商定的规范。

本章专门介绍功能/验收测试以及定义和执行它们的技巧。更具体地说，本章涵盖了以下主题：

+   理解功能测试的目的

+   使用单元测试工具自动化 C#中的功能测试

+   用例 – 自动化功能测试

到本章结束时，你将能够设计手动和自动测试，以验证冲刺产生的代码是否符合其规范。

# 技术要求

鼓励读者在继续本章之前阅读第十五章，*使用单元测试用例和 TDD 测试您的代码*。

本章需要 Visual Studio 2017 或 2019 免费社区版或更高版本，并安装所有数据库工具。在这里，我们将修改第十五章的代码，*使用单元测试用例和 TDD 测试您的代码*，该代码可在[`github.com/PacktPublishing/Hands-On-Software-Architecture-with-CSharp-8/tree/master/ch20`](https://github.com/PacktPublishing/Hands-On-Software-Architecture-with-CSharp-8/tree/master/ch20)找到[.](https://github.com/PacktPublishing/Hands-On-Software-Architecture-with-CSharp-8/tree/master/ch20)

# 理解功能测试的目的

功能/验收测试使用与单元和集成测试类似的技术，但它们的不同之处在于它们只在每个冲刺结束时运行。它们的基本作用是验证当前版本的整个软件是否符合其规范。这种验证被转化为以下目的的正式流程：

+   功能测试代表了利益相关者和开发团队之间合同的最重要部分，另一部分是非功能规格的验证。这种合同的形式化方式取决于开发团队和利益相关者之间关系的本质。在供应商-客户关系的案例中，它们成为每个冲刺的供应商-客户商业合同的一部分，并由为客户工作的团队编写。如果测试失败，则冲刺将被拒绝，供应商必须运行一个补充冲刺来解决所有问题。如果没有正式的商业合同，测试的结果通常用于驱动下一个冲刺的规格。然而，在这种情况下，如果失败率很高，冲刺可能被拒绝并需要重新进行。

+   在每个冲刺结束时运行的正式功能测试可以避免之前冲刺中取得的结果被新代码破坏。

+   当使用敏捷开发方法时，维护一个最新的功能测试库是获取最终系统规格正式表示的最佳方式，因为在敏捷开发过程中，最终系统的规格并不是在开发开始之前就确定的，而是系统演化的结果。

由于早期阶段的第一轮冲刺的输出可能与最终系统有很大差异，因此不值得花费太多时间编写详细的手动测试和/或自动化测试。因此，你可以限制只添加一些示例到用户故事中，这些示例将同时作为软件开发输入和手动测试。

随着系统功能始终更加稳定，投入时间编写详细和正式的功能测试是值得的。对于每个功能规格，我们必须编写测试来验证它们在极端情况下的正确操作。例如，在一个支付用例中，我们必须编写测试来验证所有可能性：

+   资金不足

+   各种数字化错误

+   卡片过期

+   错误的凭证和重复的错误凭证

在手动测试的情况下，对于上述每个场景，我们必须提供每个操作中涉及的每个步骤的所有详细信息，以及每个步骤的预期结果。

一个重要的决定是是否要自动化所有或部分验收/功能测试，因为编写模拟与系统用户界面交互的人类操作员的自动化测试非常昂贵。最终的决定取决于测试实现的成本除以预期使用的次数。

在 CI/CD 的情况下，相同的功能测试可以执行多次，但不幸的是，功能/验收测试严格依赖于用户界面的实现方式，而在现代系统中，用户界面经常发生变化。因此，在这种情况下，相同的测试最多只能与完全相同的用户界面执行几次。

为了克服与用户界面相关的所有问题，功能测试可以实施为**皮下测试**，即作为绕过用户界面的测试。然而，皮下测试由于其本质是不完整的，因为它们无法检测用户界面本身的错误。此外，在 Web 应用的情况下，皮下测试通常还受到其他限制，因为它们绕过了整个 HTTP 协议。在 ASP.NET Core 应用的情况下，这意味着必须绕过整个 ASP.NET Core 管道，并将请求直接传递到 ASP.NET 控制器。因此，身份验证、授权、CORS 以及 ASP.NET Core 管道中其他模块的行为将不会被测试分析。

一个完整的 Web 应用功能自动测试应该执行以下操作：

1.  在要测试的 URL 上启动一个实际的浏览器。

1.  等待页面上的任何 JavaScript 完成其执行。

1.  然后，向浏览器发送模拟人类操作员行为的命令。

1.  最后，在每次与浏览器的交互之后，自动测试应该等待，以便任何由交互触发的 JavaScript 完成其执行。

虽然存在浏览器自动化工具，但如前所述，使用浏览器自动化实现的测试非常昂贵且难以实现。因此，ASP.NET Core MVC 建议的方法是使用.NET HTTP 客户端向实际的 Web 应用副本发送实际的 HTTP 请求，而不是使用浏览器。一旦 HTTP 客户端收到 HTTP 响应，它就会将其解析为 DOM 树，并验证它是否收到了正确的响应。

与浏览器自动化工具的唯一区别是 HTTP 客户端无法运行任何 JavaScript。然而，可以添加其他测试来测试 JavaScript 代码。这些测试基于特定的 JavaScript 测试工具，例如**Jasmine**和**Karma**。

下一个部分将解释如何使用.NET HTTP 客户端自动化 Web 应用的功能测试，而功能测试自动化的实际示例将在最后一部分展示。

# 使用单元测试工具来自动化 C#中的功能测试。

自动化的功能/验收测试使用与单元和集成测试相同的测试工具。也就是说，这些测试可以嵌入到我们在第十五章中描述的同一 xUnit、NUnit 或 MSTests 项目中，即使用单元测试用例和 TDD 测试您的代码。然而，在这种情况下，我们必须添加能够与用户界面交互和检查的进一步工具。

在本章剩余部分，我们将专注于网络应用程序，因为它们是本书的主要焦点。因此，如果我们正在测试网络 API，我们只需要`HTTPClient`实例，因为它们可以轻松地与 XML 和 JSON 格式的网络 API 端点进行交互。

在 ASP.NET Core MVC 应用程序返回 HTML 页面的情况下，交互更为复杂，因为我们还需要工具来解析和与 HTML 页面 DOM 树进行交互。`AngleSharp` NuGet 包是一个很好的解决方案，因为它支持最先进的 HTML 和最小化的 CSS，并为外部提供的 JavaScript 引擎（如 Node.js）提供了扩展点。然而，我们不建议你在测试中包含 JavaScript 和 CSS，因为它们严格绑定到目标浏览器，所以对于它们来说，最佳选项是使用可以在目标浏览器中直接运行的 JavaScript 特定测试工具。

使用`HTTPClient`类测试网络应用程序有两个基本选项：

+   一个`HTTPClient`实例通过互联网/内网与实际的*预发布*网络应用程序以及所有正在测试软件的 beta 测试人员连接。这种方法的优势在于你正在测试*真实的东西*，但测试的构思更困难，因为你无法控制每个测试之前应用程序的初始状态。

+   一个`HTTPClient`实例连接到一个在每次测试之前都配置、初始化和启动的本地应用程序。这种情况与单元测试场景完全类似。测试结果可以在每个测试的初始状态固定后重现，测试设计更容易，并且实际数据库可以被一个更快、更容易初始化的内存数据库所替代。然而，在这种情况下，你离实际系统的操作还相当远。

一个好的策略是使用第二种方法，即完全控制初始状态，用于测试所有极端情况，然后使用第一种方法测试真实环境中的随机平均情况。

下面的两个部分描述了两种方法。这两种方法的不同之处仅在于你定义测试固定值的方式。

# 测试预发布应用程序

在这种情况下，你的测试只需要一个`HTTPClient`实例，因此你必须定义一个高效的固定值，它提供`HTTPClient`实例，避免出现窗口连接耗尽的风险。我们在第十二章的“*.NET Core HTTP 客户端*”部分遇到了这个问题，*使用.NET Core 应用服务架构*。可以通过使用`IHTTPClientFactory`管理`HTTPClient`实例并将它们通过依赖注入注入来解决。

一旦我们有一个依赖注入容器，我们可以通过以下代码片段来丰富其处理`HTTPClient`实例的能力：

```cs
services.AddHTTPClient();
```

在这里，`AddHTTPClient` 扩展属于 `Microsoft.Extensions.DependencyInjection` 命名空间，并在 `Microsoft.Extensions.HTTP` NuGet 包中定义。因此，我们的测试固定项必须创建一个依赖注入容器，必须调用 `AddHTTPClient`，最后必须构建容器。以下固定类执行此操作（如果您不记得固定类，请参阅 第十五章 的 *高级测试准备/测试清理场景* 部分，*使用单元测试用例和 TDD 测试您的代码*，如果您不记得固定类）：

```cs
public class HTTPClientFixture
{
    public HTTPClientFixture()
    {
        var serviceCollection = new ServiceCollection();
        serviceCollection
            .AddHTTPClient();
         ServiceProvider = serviceCollection.BuildServiceProvider();
    }

    public ServiceProvider ServiceProvider { get; private set; }
}
```

在前面的定义之后，您的测试应该如下所示：

```cs
public class UnitTest1:IClassFixture<HTTPClientFixture>
{
    private ServiceProvider _serviceProvider;

    public UnitTest1(HTTPClientFixture fixture)
    {
        _serviceProvider = fixture.ServiceProvider;
    }

    [Fact]
    public void Test1()
    {
        using (var factory = 
            _serviceProvider.GetService<IHTTPClientFactory>())
        {
            HTTPClient client = factory.CreateClient();
            //use client to interact with application here
        }
    }
}
```

在 `Test1` 中，一旦您获取了 HTTP 客户端，您可以通过发出 HTTP 请求并分析应用程序返回的响应来测试应用程序。有关如何处理服务器返回的响应的更多详细信息将在 *用例* 部分提供。

下一节解释如何测试在受控环境中运行的应用程序。

# 测试受控应用程序

在这种情况下，我们在测试应用程序中创建一个 ASP.NET Core 服务器，并使用 `HTTPClient` 实例对其进行测试。`Microsoft.AspNetCore.Mvc.Testing` NuGet 包包含我们创建 HTTP 客户端和运行应用程序的服务器所需的所有内容。我们还需要通过引用 `Microsoft.AspNetCore.App` NuGet 包来引用整个 Web 框架。

最后，我们必须通过以下步骤将测试项目转换为 Web 项目：

1.  在 Visual Studio 解决方案资源管理器中单击测试项目图标，然后从上下文菜单中选择编辑项目项。

1.  将根 XML 节点替换为 `<Project Sdk="Microsoft.NET.Sdk">`，改为 `<Project Sdk="Microsoft.NET.Sdk.web">`。

`Microsoft.AspNetCore.Mvc.Testing` 包含一个固定类，该类负责启动本地 Web 服务器并为与之交互提供客户端。预定义的固定类是 `WebApplicationFactory<T>`。泛型 `T` 参数必须使用您的 Web 项目的 `Startup` 类进行实例化。

测试看起来像以下类：

```cs
public class UnitTest1 
    : IClassFixture<WebApplicationFactory<MyProject.Startup>>
{
    private readonly 
        WebApplicationFactory<RazorPagesProject.Startup> _factory;

    public UnitTest1 (WebApplicationFactory<MyProject.Startup> factory)
    {
        _factory = factory;
    }

    [Theory]
    [InlineData("/")]
    [InlineData("/Index")]
    [InlineData("/About")]
    ....

    public async Task MustReturnOK(string url)
    {
        var client = _factory.CreateClient();
        // here both client and server are ready

        var response = await client.GetAsync(url);
        //get the response

        response.EnsureSuccessStatusCode(); 
        // verify we got a success return code.

    }
    ...
    ---
}
```

如果您想分析返回页面的 HTML，您还必须引用 `AngleSharp` NuGet 包。我们将在下一节的示例中看到如何使用它。在这种类型的测试中处理数据库的最简单方法是使用内存数据库，这些数据库更快，并且每当本地服务器关闭和重新启动时都会自动清除。

这可以通过创建一个新的部署环境，例如`AutomaticStaging`，以及一个针对测试的特定配置文件来完成。在创建了这个新的部署环境之后，前往你的应用程序的`Startup`类中的`ConfigureServices`方法，找到添加你的`DBContext`配置的地方。一旦找到这个地方，就在那里添加一个`if`语句，如果应用程序正在`AutomaticStaging`环境中运行，就用类似以下的内容替换你的`DBContext`配置：

```cs
services.AddDbContext<MyDBContext>(options =>  options.UseInMemoryDatabase(databaseName: "MyDatabase"));
```

作为一种替代方案，你还可以在继承自`WebApplicationFactory<T>`的自定义固定工具的构造函数中添加所有必要的指令来清除标准数据库。

# 用例 - 自动化功能测试

在本节中，我们将向第十五章的 ASP.NET Core 测试项目添加一个简单的验收测试，该章节是*使用单元测试用例和 TDD 测试你的代码*。我们的测试方法基于`Microsoft.AspNetCore.Mvc.Testing`和`AngleSharp` NuGet 包。请创建整个解决方案的新副本。

作为第一步，我们必须将测试项目转换成一个 Web 项目，通过替换其项目文件根节点的`sdk`属性来实现，即`Sdk="Microsoft.NET.Sdk.web"`。

测试项目已经引用了位于`test`目录下的 ASP.NET Core 项目以及所有必需的 xUnit NuGet 包，因此我们只需要添加`Microsoft.AspNetCore.Mvc.Testing`和`AngleSharp` NuGet 包。

现在，让我们添加一个名为`UIExampleTestcs.cs`的新类文件。我们需要`using`语句来引用所有必要的命名空间。更具体地说，我们需要以下内容：

+   `using PackagesManagement;`: 这是为了引用你的应用程序类所必需的。

+   `using Microsoft.AspNetCore.Mvc.Testing;`: 这是为了引用客户端和服务器类所必需的。

+   `using AngleSharp;`和`using AngleSharp.Html.Parser;`: 这些是为了引用`AngleSharp`类所必需的。

+   `System.IO`: 这是为了从 HTTP 响应中提取 HTML 所必需的。

+   `using Xunit`: 这是为了引用所有`xUnit`类所必需的。

总结起来，整个`using`块如下：

```cs
using PackagesManagement;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Xunit;
using Microsoft.AspNetCore.Mvc.Testing;
using AngleSharp;
using AngleSharp.Html.Parser;
using System.IO;
```

我们将使用之前在*测试受控应用程序*部分中引入的标准固定工具类，即以下内容：

```cs
public class UIExampleTestcs:
         IClassFixture<WebApplicationFactory<Startup>>
{
    private readonly
       WebApplicationFactory<Startup> _factory;
    public UIExampleTestcs(WebApplicationFactory<Startup> factory)
    {
       _factory = factory;
    }
}
```

现在，我们已经准备好编写主页的测试了！这个测试验证了主页 URL 返回成功的 HTTP 结果，并且主页包含指向包管理页面的链接，这是一个`/ManagePackages`的相对链接。

理解这一点是基本的，即自动测试不应依赖于 HTML 的细节，而应仅验证逻辑事实，以避免在每次对应用程序 HTML 进行微小修改后频繁更改。这就是为什么我们只验证所需的链接存在，而不对其位置施加约束。

让我们将`TestMenu`称为主页测试：

```cs
[Fact]
public async Task TestMenu()
{
    var client = _factory.CreateClient();
    ...
    ...         
}

```

每个测试的第一步是创建一个客户端。然后，如果测试需要分析一些 HTML，我们必须准备所谓的`AngleSharp`浏览上下文：

```cs
//Create an angleSharp default configuration
var config = Configuration.Default;

//Create a new context for evaluating webpages 
//with the given config
var context = BrowsingContext.New(config);
```

配置对象指定了诸如 cookie 处理和其他浏览器相关属性等选项。到目前为止，我们已经准备好要求主页：

```cs
var response = await client.GetAsync("/");
```

作为第一步，我们验证收到的响应包含成功状态码，如下所示：

```cs
response.EnsureSuccessStatusCode(); 
```

前面的方法调用在状态码不成功时抛出异常，从而导致测试失败。HTML 分析需要从响应中提取。以下代码展示了如何简单地进行操作：

```cs
var stream = await response.Content.ReadAsStreamAsync();
string source;
using (StreamReader responseReader = new StreamReader(stream))
{
    source = await responseReader.ReadToEndAsync();
} 
```

`ReadAsStreamAsync`返回`Stream`，我们可以使用它来构建`StreamReader`（一个专门用于读取文本的流），它可以读取整个响应体。

现在，我们必须将提取的 HTML 传递给之前的`AngleSharp`浏览上下文对象，以便它可以构建 DOM 树。以下代码展示了如何操作：

```cs
var document = await context.OpenAsync(req => req.Content(source));
```

`OpenAsync`方法使用`context`中包含的设置执行 DOM 构建活动。构建 DOM 文档的输入由传递给`OpenAsync`方法的 lambda 函数指定。在我们的情况下，`req.Content(...)`从传递给`Content`方法的 HTML 字符串构建 DOM 树，这是客户端收到的响应中包含的 HTML。

一旦获得`document`对象，我们就可以像在 JavaScript 中使用它一样使用它。特别是，我们可以使用`QuerySelector`来查找具有所需链接的锚点：

```cs
var node = document.QuerySelector("a[href=\"/ManagePackages\"]");
```

剩下的就是验证`node`不是 null：

```cs
Assert.NotNull(node);
```

我们做到了！如果你想要分析需要用户登录或其他更复杂场景的页面，你需要启用 HTTP 客户端中的 cookie 和自动 URL 重定向。这样，客户端将表现得像通常的浏览器，它会存储和发送 cookie，并在接收到`Redirect` HTTP 响应时移动到另一个 URL。这可以通过将选项对象传递给`CreateClient`方法来实现，如下所示：

```cs
var client = _factory.CreateClient(
    new WebApplicationFactoryClientOptions
    {
        AllowAutoRedirect=true,
        HandleCookies=true
    });
```

使用前面的设置，你的测试可以做到一个普通浏览器能做的所有事情。例如，你可以设计测试，其中 HTTP 客户端登录并访问需要身份验证的页面，因为`HandleCookies=true`允许客户端存储身份验证 cookie 并在所有后续请求中发送它。

# 摘要

本章解释了验收/功能测试的重要性，以及如何定义详细的手动测试，以便在每个冲刺的输出上运行。到目前为止，你应该能够定义自动和/或手动测试，以验证在每个冲刺结束时，你的应用程序符合其规范。

然后，本章分析了何时值得自动化某些或所有验收/功能测试，并描述了如何在 ASP.NET Core 应用程序中自动化它们。

一个最终的例子展示了如何使用`AngleSharp`编写 ASP.NET Core 验收/功能测试，以检查应用程序返回的响应。

# 问题

1.  在快速 CI/CD 循环的情况下，自动化用户界面验收测试是否总是值得？

1.  对 ASP.NET Core 应用程序的皮下测试有什么缺点？

1.  建议用于编写 ASP.NET Core 验收测试的技术是什么？

1.  建议如何检查服务器返回的 HTML？

# 进一步阅读

关于 `Microsoft.AspNetCore.Mvc.Testing` NuGet 包和 `AngleSharp` 的更多详细信息可以在它们各自的官方文档中找到：[`docs.microsoft.com/en-US/aspnet/core/test/integration-tests?view=aspnetcore-3.0`](https://docs.microsoft.com/en-US/aspnet/core/test/integration-tests?view=aspnetcore-3.0) 和 [`anglesharp.github.io/`](https://anglesharp.github.io/).

对 JavaScript 测试感兴趣的读者可以参考 Jasmine 文档：[`jasmine.github.io/`](https://jasmine.github.io/).
