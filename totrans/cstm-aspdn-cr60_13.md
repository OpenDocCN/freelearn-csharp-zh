# *第 13 章*：使用自定义 ModelBinder 管理输入

在上一章关于 `OutputFormatter` 的内容中，我们学习了如何以不同的格式向客户端发送数据。在本章中，我们将反向操作。本章是关于从外部获取的数据；例如，如果你以特殊格式接收到数据，或者如果你需要以特殊方式验证接收到的数据时，**Model Binder** 将帮助你处理这种情况。

在本章中，我们将涵盖以下主题：

+   介绍 `ModelBinder`

+   准备测试项目

+   创建 `PersonsCsvBinder`

+   使用 `ModelBinder`

+   测试 `ModelBinder`

本章中的主题涉及 ASP.NET Core 架构的 WebAPI 层：

![图 13.1 – ASP.NET Core 架构

](img/Figure_13.1_B17996.jpg)

图 13.1 – ASP.NET Core 架构

# 技术要求

要遵循本章的描述，你需要创建一个 ASP.NET Core MVC 应用程序。打开你的控制台、shell 或 Bash 终端，切换到你的工作目录。使用以下命令创建一个新的 MVC 应用程序：

[PRE0]

现在，通过双击项目文件或在 VS Code 中输入以下命令来在 Visual Studio 中打开项目：

[PRE1]

本章中所有的代码示例都可以在本书的 GitHub 仓库中找到：[https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/tree/main/Chapter13](https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/tree/main/Chapter13)。

# 介绍 ModelBinder

Model Binder 负责将传入的数据绑定到特定的操作方法参数。它们将请求中发送的数据绑定到参数。默认绑定器能够绑定通过 `QueryString` 发送或请求正文中发送的数据。在正文中，数据可以以 URL 或 JSON 格式发送。

模型绑定尝试通过参数名称在请求中查找值。表单值、路由数据和查询字符串值存储为一个键值对集合，绑定尝试在集合的键中查找参数名称。

让我们通过一个测试项目来演示这是如何工作的。

# 准备测试数据

在本节中，我们将了解如何将 CSV 数据发送到 Web API 方法。我们将重用我们在 [*第 12 章*](B17996_12_ePub.xhtml#_idTextAnchor172)，*使用自定义 OutputFormatter 进行内容协商* 中创建的 CSV 数据。

这是我们想要使用的测试数据片段：

[PRE2]

你可以在 GitHub 上找到完整的 CSV 测试数据：[https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/blob/main/Chapter13/testdata.csv](https://github.com/PacktPublishing/Customizing-ASP.NET-Core-6.0-Second-Edition/blob/main/Chapter13/testdata.csv)。

# 准备测试项目

让我们按照以下步骤准备项目：

1.  在已经创建的项目（参考*技术要求*部分），我们现在将创建一个新的空API控制器，其中包含一个小操作：

    [PRE3]

    这个看起来基本上和任何其他操作一样。它接受人员列表并返回一个包含人员数量以及人员列表的匿名对象。这个操作相当无用，但帮助我们使用Postman调试`ModelBinder`。

1.  我们还需要`Person`类：

    [PRE4]

    如果我们想向该操作发送基于JSON的数据，这将实际上工作得很好。

1.  作为最后的准备步骤，我们需要添加`CsvHelper` NuGet包以更轻松地解析CSV数据。.NET CLI在这里也很有用：

    [PRE5]

现在所有这些都设置好了，我们可以在下一节尝试它并创建`PersonsCsvBinder`。

# 创建`PersonsCsvBinder`

让我们构建一个绑定器。

要创建`ModelBinder`，添加一个名为`PersonsCsvBinder`的新类，该类实现`IModelBinder`接口。在`BindModelAsync`方法中，我们获取包含所有所需信息的`ModelBindingContext`，以便获取数据和反序列化。以下代码片段显示了一个通用的绑定器，它应该适用于任何模型列表。我们将其分为几个部分，以便您可以清楚地看到绑定器的每个部分是如何工作的：

[PRE6]

如您从前面的代码块中看到的，首先检查上下文是否为null。之后，如果尚未指定，我们将设置默认参数名称给模型。如果这样做，我们就可以通过之前设置的名称获取值：

[PRE7]

在下一部分，如果没有值，在这种情况下我们不应该抛出异常。原因是下一个配置的`ModelBinder`可能负责。如果我们抛出异常，当前请求的执行将被取消，下一个配置的`ModelBinder`就没有机会被执行：

[PRE8]

如果我们有值，我们可以实例化一个新的`StringReader`，需要将其传递给`CsvReader`：

[PRE9]

使用`CsvReader`，我们可以将CSV字符串值反序列化为`Persons`列表。如果我们有列表，我们需要创建一个新的、成功的`ModelBindingResult`，并将其分配给`ModelBindingContext`的`Result`属性：

[PRE10]

你可能需要在文件开头添加以下`using`语句：

[PRE11]

接下来，我们将使用`ModelBinder`。

# 使用模型绑定器

绑定器不会自动使用，因为它没有在依赖注入容器中注册，并且没有配置在MVC框架中使用。

使用此模型绑定器的最简单方法是，在模型应绑定到的操作的参数上使用`ModelBinderAttribute`：

[PRE12]

在这里，我们将我们的`PersonsCsvBinder`的类型设置为`binderType`属性。

注意

使用`ModelBinderProvider`将`ModelBinder`添加到现有列表中。

我个人更喜欢显式声明，因为大多数自定义`ModelBinder`将特定于某个操作或特定类型，并且可以防止在后台隐藏魔法。

现在，让我们测试一下我们所构建的内容。

# 测试ModelBinder

要测试它，我们需要在Postman中创建一个新的请求：

1.  通过在控制台运行`dotnet run`或按*F5*在Visual Studio或VS Code中启动应用程序来启动应用程序。

1.  在Postman中，我们将在地址栏中将请求类型设置为`https://localhost:5001/api/persons`。

    端口号可能因您的环境而异。

1.  接下来，我们需要将CSV数据添加到请求体中。选择`form-data`作为请求体类型，添加`persons`键，并在值字段中粘贴以下值行：

    [PRE13]

1.  按下**发送**后，我们得到的结果，如图*图13.2*所示：

![图13.2 – Postman中的CSV数据截图

![图13.2 – Postman中的CSV数据截图](img/Figure_13.2_B17996.jpg)

图13.2 – Postman中的CSV数据截图

现在，客户端将能够向服务器发送基于CSV的数据。

# 摘要

这是一种将输入转换为动作所需的方式的好方法。您也可以使用`ModelBinder`对数据库或您需要在模型传递给动作之前完成的任何操作进行一些自定义验证。

在下一章中，我们将看到您可以使用`ActionFilter`做什么。

# 进一步阅读

要了解更多关于`ModelBinder`的信息，您应该查看以下相对详细的文档：

+   Steve Gordon，*ASP.NET MVC Core中的自定义模型绑定*: https://www.stevejgordon.co.uk/html-encode-string-aspnet-core-model-binding/

+   *ASP.NET Core中的模型绑定*: https://docs.microsoft.com/en-us/aspnet/core/mvc/models/model-binding

+   *ASP.NET Core中的自定义模型绑定*: https://docs.microsoft.com/en-us/aspnet/core/mvc/advanced/custom-model-binding
