# 6

# 使用流行的第三方库

本章介绍了几个流行的 .NET 第三方库，这些库允许你执行一些操作，这些操作要么无法使用核心 .NET 库完成，要么比内置功能更好。这些操作包括使用 **ImageSharp** 处理图像、使用 **Serilog** 记录、使用 **AutoMapper** 将对象映射到其他对象、使用 **FluentAssertions** 进行单元测试断言、使用 **FluentValidation** 验证数据以及使用 **QuestPDF** 生成 PDF。

本章涵盖以下主题：

+   哪些第三方库最受欢迎？

+   处理图像

+   使用 Serilog 记录

+   对象之间的映射

+   在单元测试中进行流畅断言

+   验证数据

+   生成 PDF

# 哪些第三方库最受欢迎？

为了帮助我决定在本书中包含哪些第三方库，我研究了在 [https://www.nuget.org/stats/packages](https://www.nuget.org/stats/packages) 上下载频率最高的库，如 *表 6.1* 所示，它们是：

| **排名** | **包** | **下载量** |
| --- | --- | --- |
| 1 | `newtonsoft.json` | 167,927,712 |
| 2 | `serilog` | 42,436,567 |
| 3 | `awssdk.core` | 36,423,449 |
| 4 | `castle.core` | 28,383,411 |
| 5 | `newtonsoft.json.bson` | 26,547,661 |
| 6 | `swashbuckle.aspnetcore.swagger` | 25,828,940 |
| 7 | `swashbuckle.aspnetcore.swaggergen` | 25,823,941 |
| 8 | `polly` | 22,487,368 |
| 9 | `automapper` | 21,679,921 |
| 10 | `swashbuckle.aspnetcore.swaggerui` | 21,373,873 |
| 12 | `moq` | 19,408,440 |
| 15 | `fluentvalidation` | 17,739,259 |
| 16 | `humanizer.core` | 17,602,598 |
| 23 | `stackexchange.redis` | 15,771,377 |
| 36 | `fluentassertions` | 12,244,097 |
| 40 | `dapper` | 10,819,569 |
| 52 | `rabbitmq.client` | 8,591,362 |
| 83 | `hangfire.core` | 5,479,381 |
| 94 | `nodatime` | 4,944,830 |

表 6.1：最受欢迎的 NuGet 包

## 我书中涵盖的内容

我的书《C# 12 和 .NET 8 – 现代跨平台开发基础》介绍了使用 `newtonsoft.json` 处理 JSON 以及使用 `swashbuckle` 记录 Web 服务。

目前，使用 Castle Core 生成动态代理和类型化字典，或将应用程序部署到并与 **Amazon Web Services**（**AWS**）集成，超出了本书的范围。

除了原始下载量外，读者的提问和库的有用性也影响了我将库包含在本章中的决定，如下列总结所示：

+   最受欢迎的图像处理库：**ImageSharp**

+   最受欢迎的文本处理库：**Humanizer**

+   最受欢迎的日志库：**Serilog**

+   最受欢迎的对象映射库：**AutoMapper**

+   最受欢迎的单元测试断言库：**FluentAssertions**

+   最受欢迎的数据验证库：**FluentValidation**

+   生成 PDF 的开源库：**QuestPDF**

在第 7 章，*处理日期、时间和国际化* 中，我介绍了处理日期和时间的最流行库：**Noda Time**。

在第 9 章，*缓存、队列和弹性后台服务* 中，我介绍了一些更受欢迎的库，如下列列表中总结：

+   最受欢迎的弹性和短暂故障处理库：**Polly**

+   最受欢迎的作业调度和实现后台服务的库：**Hangfire**

+   最受欢迎的分布式缓存库：**Redis**

+   最受欢迎的队列库：**RabbitMQ**

# 处理图像

**ImageSharp** 是一个第三方跨平台 2D 图形库。当 .NET Core 1.0 开发时，社区对缺少用于处理 2D 图像的 `System.Drawing` 命名空间提出了负面反馈。ImageSharp 项目开始是为了填补现代 .NET 应用程序的这一空白。

在 Microsoft 的 `System.Drawing` 的官方文档中，微软表示，“`System.Drawing` 命名空间不建议用于新开发，因为它不支持在 Windows 或 ASP.NET 服务中，并且它不是跨平台的。ImageSharp 和 SkiaSharp 被推荐作为替代方案。”

六个劳动在 2023 年 3 月发布了 ImageSharp 3.0。现在它需要 .NET 6 或更高版本，未来的主要版本将针对 .NET 的 LTS 版本，如 .NET 8。您可以在以下链接中阅读公告：[https://sixlabors.com/posts/announcing-imagesharp-300/](https://sixlabors.com/posts/announcing-imagesharp-300/)。

## 生成灰度缩略图

让我们看看 ImageSharp 能实现什么：

1.  使用您首选的代码编辑器创建一个控制台应用程序项目，如下列列表中定义：

    +   项目模板：**控制台应用程序** / `console`

    +   解决方案文件和文件夹：`Chapter06`

    +   项目文件和文件夹：`WorkingWithImages`

    +   **不要使用顶级语句**：已清除

    +   **启用原生 AOT 发布**：已清除

1.  在 `WorkingWithImages` 项目中，创建一个 `images` 文件夹，并从以下链接下载九张图片到该文件夹：[https://github.com/markjprice/apps-services-net8/tree/master/images/Categories](https://github.com/markjprice/apps-services-net8/tree/master/images/Categories)。

1.  如果你正在使用 Visual Studio 2022，那么 `images` 文件夹及其文件必须复制到 `WorkingWithImages\bin\Debug\net8` 文件夹，编译的控制台应用程序将在该文件夹中运行。我们可以配置 Visual Studio 为我们完成此操作，如下所示的操作步骤：

    1.  在 **解决方案资源管理器** 中，选择所有九张图片。

    1.  在 **属性** 中，将 **复制到输出目录** 设置为 **始终复制**，如图 6.1 所示：![包含文本、截图、软件、计算机图标  自动生成的描述](img/B19587_06_01.png)

    图 6.1：设置图像始终复制到输出目录

    1.  打开项目文件，注意 `<ItemGroup>` 条目，它们将复制九张图片到正确的文件夹，如下所示的部分标记：

        [PRE0]

1.  在 `WorkingWithImages` 项目中，将警告视为错误，全局和静态导入 `System.Console` 类，并为 `SixLabors.ImageSharp` 添加包引用，如下面的标记所示：

    [PRE1]

    为了节省空间，在本章的其他类似步骤中，我将不会显示将警告视为错误或全局和静态导入 `System.Console` 类的标记。我将只显示针对特定任务的库的 `ItemGroup` 和 `PackageReference`。

1.  构建名为 `WorkingWithImages` 的项目。

1.  如果你正在使用 Visual Studio 2022，那么在**解决方案资源管理器**中，切换**显示所有文件**。

1.  在 `obj\Debug\net8.0` 文件夹中，打开 `WorkingWithImages.GlobalUsings.g.cs`，并注意引用 `SixLabors.ImageSharp` 包会在 .NET SDK 添加的常规命名空间导入旁边添加三个全局命名空间导入，如下面的代码所示：

    [PRE2]

    如果你引用了 `SixLabors.ImageSharp` 的旧版本，如 2.0.0，那么它不会这样做，因此你必须在每个代码文件中手动导入这三个命名空间。这是为什么 3.0 及以后的版本对 .NET 6 有最低要求的一个原因。

1.  在 `Program.cs` 中，删除现有的语句，然后添加语句将 `images` 文件夹中的所有文件转换为十分之一大小的灰度缩略图，如下面的代码所示：

    [PRE3]

1.  运行控制台应用程序，并注意图像应该被转换为灰度缩略图，如下面的部分输出所示：

    [PRE4]

1.  在文件系统中，打开适当的 `images` 文件夹，并注意比之前小得多的字节灰度缩略图，如图 *6.2* 所示：

![图片](img/B19587_06_02.png)

图 6.2：处理后的图像

## 用于绘制和网络的 ImageSharp 包

ImageSharp 还提供了用于程序化绘制图像和在网络中处理图像的 NuGet 包，如下面的列表所示：

+   `SixLabors.ImageSharp.Drawing`

+   `SixLabors.ImageSharp.Web`

    **更多信息**：了解更多详情，请访问以下链接：[https://docs.sixlabors.com/](https://docs.sixlabors.com/)。

# 使用 Humanizer 处理文本和数字

Humanizer 在 `string` 值、`enum` 值的名称、日期、时间、数字和数量中操作文本。

## 处理文本

内置的 `string` 类型有 `Substring` 和 `Trim` 等方法来操作文本。但还有许多其他常见的文本操作，我们可能想要执行。例如：

+   你可能有一些代码生成的丑陋字符串，你希望使其看起来更友好，以便向用户显示。这在多词值不能使用空格的 `enum` 类型中很常见。

+   你可能正在构建一个内容管理系统，当用户输入文章标题时，你需要将他们输入的内容转换为适合 URL 路径的格式。

+   你可能有一个需要截断以在移动用户界面有限空间中显示的长 `string` 值。

## Humanizer 的案例转换

可以通过将多个 Humanizer 转换传递给 `Transform` 方法来按顺序执行复杂转换。转换实现 `IStringTransformer` 或 `ICulturedStringTransformer` 接口，因此您可以实现自己的自定义转换。

内置的转换都是大小写转换，它们列在 *表 6.2* 中，以及扩展 `string` 类型的便捷替代方法：

| **Transform** | **描述** | **示例** |
| --- | --- | --- |
| `To.LowerCase` | 将 `string` 中的所有字符转换为小写。 | the cat sat on the mat |
| `To.UpperCase` | 将 `string` 中的所有字符转换为大写。 | THE CAT SAT ON THE MAT |
| `To.TitleCase` | 将 `string` 中每个单词的第一个字符转换为大写。 | The Cat Sat on the Mat |
| `To.SentenceCase` | 将 `string` 中的第一个字符转换为大写。它忽略句点（句号），因此不识别句子！ | The cat sat on the mat |

表 6.2：Humanizer 大小写转换

**良好实践**：考虑原始文本的大小写非常重要。如果它已经是大写，标题和句子大小写选项将 *不会* 转换为小写！您可能需要先转换为小写，然后再转换为标题或句子大小写。

除了使用 `To.TitleCase` 这样的转换对象调用 `Transform` 方法外，还有便捷的方法来操作文本的大小写，如 *表 6.3* 所示：

| **字符串扩展方法** | **描述** |
| --- | --- |
| `Titleize` | 等同于使用 `To.TitleCase` 进行转换 |
| `Pascalize` | 将字符串转换为 upper camel case，同时删除下划线 |
| `Camelize` | 与 `Pascalize` 相同，除了第一个字符是小写 |

表 6.3：Humanizer 文本扩展方法

## Humanizer 空格转换

有便捷的方法来操作文本的空格，通过添加下划线和破折号，如 *表 6.4* 所示：

| **字符串扩展方法** | **描述** |
| --- | --- |
| `Underscore` | 使用下划线分隔输入单词 |
| `Dasherize`、`Hyphenate` | 在字符串中将下划线替换为破折号（连字符） |
| `Kebaberize` | 使用破折号（连字符）分隔输入单词 |

表 6.4：Humanizer 空格转换方法

## Humanizer 的单复数转换方法

Humanizer 为 `string` 类提供了两个扩展方法，用于自动在单词的单数和复数版本之间转换，如 *表 6.5* 所示：

| **字符串扩展方法** | **描述** |
| --- | --- |
| `Singularize` | 如果 `string` 包含复数词，则将其转换为单数等价词。 |
| `Pluralize` | 如果 `string` 包含单数词，则将其转换为复数等价词。 |

表 6.5：Humanizer 的单复数转换方法

这些方法被 Microsoft Entity Framework Core 用于将实体类及其成员的名称单复数化。

## 使用控制台应用程序探索文本操作

让我们探索一些使用Humanizer进行文本操作示例：

1.  使用你喜欢的代码编辑器向`Chapter06`解决方案中添加一个名为`HumanizingData`的新**Console App** / `console`项目。在Visual Studio 2022中，将启动项目设置为当前选择。

1.  在`HumanizingData`项目中，将警告视为错误，全局和静态导入`System.Console`类，并为`Humanizer`添加包引用，如下所示标记：

    [PRE5]

    我们正在引用一个包含所有语言的`Humanizer`包。如果你只需要英语，那么你可以引用`Humanizer.Core`。如果你还需要语言子集，请使用模式`Humanizer.Core.<lang>`引用特定的语言包，例如，`Humanizer.Core.fr`用于法语。

1.  构建项目`HumanizingData`以恢复包。

1.  向项目中添加一个名为`Program.Functions.cs`的新类文件。

1.  在`Program.Functions.cs`中，添加导入用于全球化操作的命名空间的语句，并定义一个方法来配置控制台以启用当前文化的轻松切换并启用特殊字符的使用，如下所示代码：

    [PRE6]

1.  在`Program.cs`中，删除现有的语句，然后调用`ConfigureConsole`方法，如下所示代码：

    [PRE7]

1.  在`Program.Functions.cs`中，添加导入用于处理Humanizer提供的扩展方法的命名空间的语句，如下所示代码：

    [PRE8]

1.  在`Program.Functions.cs`中，添加定义一个方法来输出原始的`string`，然后是使用内置的大小写转换转换的结果，如下所示代码：

    [PRE9]

1.  在`Program.cs`中，使用三个不同的`string`值调用`OutputCasings`方法，如下所示代码：

    [PRE10]

1.  运行代码并查看结果，如下所示输出：

    [PRE11]

1.  在`Program.Functions.cs`中，添加定义一个方法，该方法使用各种Humanizer扩展方法输出一个丑陋的`string`值，如下所示代码：

    [PRE12]

1.  在`Program.cs`中，注释掉之前的调用方法，然后添加调用新方法的语句，如下所示高亮显示的代码：

    [PRE13]

1.  运行代码并查看结果，如下所示输出：

    [PRE14]

1.  向项目中添加一个名为`WondersOfTheAncientWorld.cs`的新类文件。

1.  修改`WondersOfTheAncientWorld.cs`文件，如下所示代码：

    [PRE15]

1.  在`Program.Functions.cs`中，导入用于使用我们刚刚定义的`enum`的命名空间，如下所示代码：

    [PRE16]

1.  在`Program.Functions.cs`中，定义一个方法来创建`WondersOfTheWorld`变量并使用各种Humanizer扩展方法输出其名称，如下所示代码：

    [PRE17]

1.  在`Program.cs`中，注释掉之前的调用方法，然后添加对`OutputEnumNames`的调用，如下所示高亮显示的代码：

    [PRE18]

1.  运行代码并查看结果，如下所示输出：

    [PRE19]

注意`Truncate`方法默认使用单字符`…`（省略号）。如果你要求截断到8个字符的长度，它可以返回前七个字符后跟省略号字符。你可以使用`Truncate`方法的重载指定不同的字符。

## 处理数字

现在让我们看看Humanizer如何帮助我们处理数字：

1.  在`Program.Functions.cs`中，定义一个方法以创建一些数字，然后使用各种Humanizer扩展方法输出它们，如下所示代码：

    [PRE20]

1.  在`Program.cs`中，注释掉之前的方法调用并添加对`NumberFormatting`的调用，如下所示高亮显示的代码：

    [PRE21]

1.  运行代码并查看结果，如下所示输出：

    [PRE22]

    Humanizer的默认词汇表相当不错，但它并没有正确地将“总检察长”（复数形式为*attorneys general*）或“二头肌”（单数为*biceps*，复数为*bicepses*）进行复数化。

1.  在`Program.Functions.cs`中，导入用于处理Humanizer词汇的命名空间，如下所示代码：

    [PRE23]

1.  在`Program.Functions.cs`中，在`NumberFormatting`方法的顶部添加语句以注册两个不规则单词，如下所示代码：

    [PRE24]

1.  运行代码，查看结果，并注意现在两个不规则单词的输出是正确的。

## 处理日期和时间

现在让我们看看Humanizer如何帮助我们处理日期和时间：

1.  在`Program.Functions.cs`中，定义一个方法以获取当前日期和时间以及一些天数，然后使用各种Humanizer扩展方法输出它们，如下所示代码：

    [PRE25]

1.  在`Program.cs`中，注释掉之前的方法调用并添加对`DateTimeFormatting`的调用，如下所示高亮显示的代码：

    [PRE26]

1.  运行代码并查看结果，如下所示输出：

    [PRE27]

1.  在`Program.cs`中，配置控制台时指定法语语言和区域，如下所示高亮显示的代码：

    [PRE28]

1.  运行代码，查看结果，并注意文本已本地化为法语。

# 使用Serilog进行日志记录

虽然.NET包括日志记录框架，但第三方日志提供程序通过使用**结构化事件数据**提供了更多的功能和灵活性。Serilog是最受欢迎的。

## 结构化事件数据

大多数系统将纯文本消息写入其日志。

Serilog可以指示将序列化的结构化数据写入日志。在参数前加上`@`符号会告诉Serilog序列化传入的对象，而不是仅调用`ToString`方法的结果。

之后，那个复杂对象可以在日志中进行查询，以提供改进的搜索和排序功能。

例如：

[PRE29]

你可以在以下链接中了解更多关于Serilog如何处理结构化数据的信息：[https://github.com/serilog/serilog/wiki/Structured-Data](https://github.com/serilog/serilog/wiki/Structured-Data)。

## Serilog接收器

所有日志系统都需要将日志条目记录到某个地方。这可能是在控制台输出、文件或更复杂的数据存储，如关系数据库或云数据存储。Serilog 将这些称为 **sinks**。

Serilog 为您可能想要记录日志的所有可能位置提供了数百个官方和第三方 sink 包。要使用它们，只需包含适当的包。以下是最受欢迎的列表：

+   `serilog.sinks.file`

+   `serilog.sinks.console`

+   `serilog.sinks.periodicbatching`

+   `serilog.sinks.debug`

+   `serilog.sinks.rollingfile`（已弃用；请使用 `serilog.sinks.file` 代替）

+   `serilog.sinks.applicationinsights`

+   `serilog.sinks.mssqlserver`

目前在微软的公共 NuGet 资源库中列出了 470 多个包：[https://www.nuget.org/packages?q=serilog.sinks](https://www.nuget.org/packages?q=serilog.sinks)。

## 使用 Serilog 将日志记录到控制台和滚动文件

让我们开始：

1.  使用您首选的代码编辑器，向 `Chapter06` 解决方案中添加一个名为 `Serilogging` 的新 **Console App** / `console` 项目。

1.  在 `Serilogging` 项目中，将警告视为错误，全局和静态导入 `System.Console` 类，并为 `Serilog` 添加包引用，包括 `console` 和 `file`（也支持滚动文件）的存储，如下面的标记所示：

    [PRE30]

1.  构建项目 `Serilogging`。

1.  在 `Serilogging` 项目中，添加一个名为 `Models` 的新文件夹。

1.  在 `Serilogging` 项目中，在 `Models` 文件夹中，添加一个名为 `ProductPageView.cs` 的新类文件，并修改其内容，如下面的代码所示：

    [PRE31]

1.  在 `Program.cs` 中，删除现有的语句，然后导入一些用于处理 Serilog 的命名空间，如下面的代码所示：

    [PRE32]

1.  在 `Program.cs` 中，创建一个将日志写入控制台并配置滚动间隔的日志配置器，这意味着每天创建一个新文件，并写入各种级别的日志条目，如下面的代码所示：

    [PRE33]

1.  运行控制台应用程序，并注意消息，如下面的输出所示：

    [PRE34]

1.  打开 `logYYYYMMDD.txt` 文件，其中 `YYYY` 是年份，`MM` 是月份，`DD` 是日期，并注意它包含相同的消息：

    +   对于 Visual Studio 2022，日志文件将被写入到 `Serilogging\bin\Debug\net8.0` 文件夹。

    +   对于 Visual Studio Code 和 `dotnet run`，日志文件将被写入到 `Serilogging` 文件夹。

        **更多信息**：在以下链接中了解更多详情：[https://serilog.net/](https://serilog.net/)。

**良好实践**：在不必要时禁用日志记录。日志记录可能会迅速变得昂贵。例如，一个组织每月在云资源上花费10,000美元，远超过预期，而且他们不知道原因。结果发现，他们在生产中记录了每个执行的SQL语句。通过“切换”停止该日志记录，他们每月节省了8,500美元！您可以在以下链接中阅读Milan Jovanović的故事：[https://www.linkedin.com/posts/milan-jovanovic_i-helped-a-team-save-100k-in-azure-cloud-activity-7109887614664474625-YDiU/](https://www.linkedin.com/posts/milan-jovanovic_i-helped-a-team-save-100k-in-azure-cloud-activity-7109887614664474625-YDiU/).

# 对象之间的映射

作为程序员最无聊的部分之一就是对象之间的映射。通常需要集成具有概念上相似对象但结构不同的系统或组件。

应用程序的不同部分可能有不同的数据模型。表示存储中数据的模型通常被称为**实体模型**。表示必须在层之间传递的数据的模型通常称为**数据传输对象**（**DTOs**）。仅表示必须向用户展示的数据的模型通常称为**视图模型**。所有这些模型都可能具有共性但结构不同。

**AutoMapper**是一个流行的映射对象包，因为它具有使工作尽可能简单的约定。例如，如果您有一个名为`CompanyName`的源成员，它将被映射到具有相同名称`CompanyName`的目标成员。

AutoMapper的创造者Jimmy Bogard写了一篇关于其设计哲学的文章，值得一读，可在以下链接中找到：[https://jimmybogard.com/automappers-design-philosophy/](https://jimmybogard.com/automappers-design-philosophy/).

让我们看看AutoMapper的实际应用示例。您将创建四个项目：

+   实体和视图模型的类库。

+   一个类库，用于创建可在单元测试和实际项目中重用的映射配置。

+   一个单元测试项目来测试映射。

+   一个用于执行实时映射的控制台应用程序。

我们将构建一个示例对象模型，它代表一个电子商务网站客户及其包含几个项目的购物车，然后将其映射到摘要视图模型以向用户展示。

## 定义AutoMapper配置的模型

为了测试映射，我们将定义一些`record`类型。作为提醒，`record`（或`record class`）是一个基于值相等性的引用类型。`class`是一个基于内存地址相等性的引用类型（除了`string`，它覆盖了这种行为）。

在使用之前始终验证映射配置是一个好习惯，因此我们将首先定义一些模型和它们之间的映射，然后为映射创建一个单元测试：

1.  使用您首选的代码编辑器将名为`MappingObjects.Models`的新**类库**/ `classlib`项目添加到`Chapter06`解决方案中。

1.  在`MappingObjects.Models`项目中，删除名为`Class1.cs`的文件。

1.  在`MappingObjects.Models`项目中，添加一个名为`Customer.cs`的新类文件，并修改其内容以使用构造函数语法定义一个不可变的记录类型`Customer`，如下面的代码所示：

    [PRE35]

1.  在`MappingObjects.Models`项目中，添加一个名为`LineItem.cs`的新类文件，并修改其内容，如下面的代码所示：

    [PRE36]

1.  在`MappingObjects.Models`项目中，添加一个名为`Cart.cs`的新类文件，并修改其内容，如下面的代码所示：

    [PRE37]

1.  在`MappingObjects.Models`项目中，在`Northwind.ViewModels`命名空间（不是`Northwind.EntityModels`）中添加一个名为`Summary.cs`的新类文件，删除任何现有语句，然后定义一个记录类型，该类型可以使用默认的无参数构造函数实例化，并在之后设置其属性，如下面的代码所示：

    [PRE38]

对于实体模型，我们使用了使用构造函数语法定义的`record class`类型来使它们不可变。但`Summary`的一个实例需要使用默认的无参数构造函数创建，然后通过 AutoMapper 设置其成员。因此，它必须是一个`record class`，具有可以在初始化时设置但在之后不能设置的公共属性。为此，我们使用`init`关键字。

## 定义 AutoMapper 配置的映射器

现在我们可以定义模型之间的映射：

1.  使用您首选的代码编辑器将名为`MappingObjects.Mappers`的新**类库**/ `classlib`项目添加到`Chapter06`解决方案中。

1.  在`MappingObjects.Mappers`项目中，将警告视为错误，添加对最新`AutoMapper`包的引用，并添加对`Models`项目的引用，如下面的标记所示：

    [PRE39]

1.  构建`MappingObjects.Mappers`项目以恢复包并编译引用的项目。

1.  在`MappingObjects.Mappers`项目中，删除名为`Class1.cs`的文件。

1.  在`MappingObjects.Mappers`项目中，添加一个名为`CartToSummaryMapper.cs`的新类文件，并修改其内容以创建一个映射配置，将`Summary`的`FullName`映射到`Customer`的`FirstName`和`LastName`的组合，如下面的代码所示：

    [PRE40]

## 对 AutoMapper 配置进行测试

现在我们可以为映射器定义单元测试：

1.  使用您首选的代码编辑器将名为`MappingObjects.Tests`的新**xUnit 测试项目**/ `xunit`添加到`Chapter06`解决方案中。

1.  在`MappingObjects.Tests`项目文件中，添加对`AutoMapper`的包引用，如下面的标记所示：

    [PRE41]

1.  在`MappingObjects.Tests`项目文件中，添加对`MappingObjects.Models`和`MappingObjects.Mappers`的项目引用，如下面的标记所示：

    [PRE42]

1.  构建MappingObjects.Tests项目以恢复包并构建引用的项目。

1.  在`MappingObjects.Tests`项目中，将`UnitTest1.cs`重命名为`TestAutoMapperConfig.cs`。

1.  修改`TestAutoMapperConfig.cs`的内容以获取映射器，然后断言映射已完成，如下所示代码：

    [PRE43]

1.  运行测试：

    +   在Visual Studio 2022中，导航到**测试** | **运行所有测试**。

    +   在Visual Studio Code的**终端**中，输入`dotnet test`。

1.  注意测试失败是因为`Summary`视图模型的`Total`成员未映射，如图6.3所示：

![](img/B19587_06_03.png)

图6.3：测试失败是因为Total成员未映射

1.  在`MappingObjects.Mappers`项目中，在映射配置中，在`FullName`成员的映射之后，添加对`Total`成员的映射，如下所示代码中突出显示：

    [PRE44]

1.  运行测试并注意，这次它通过了。

## 在模型之间执行实时映射

现在我们已经验证了映射配置，我们可以在实时控制台应用程序中使用它：

1.  使用您首选的代码编辑器将一个新的**控制台应用程序**/ `console`项目命名为`MappingObjects.Console`并添加到`Chapter06`解决方案中。

1.  在`MappingObjects.Console`项目中，将警告视为错误，全局和静态导入`System.Console`类，为两个类库添加项目引用，并为`AutoMapper`添加包引用，如下所示标记：

    [PRE45]

1.  构建项目`MappingObjects.Console`。

1.  在`Program.cs`中，删除现有的语句，添加一些语句来构建一个表示客户及其购物车中几个项目的示例对象模型，并将其映射到摘要视图模型以向用户展示，如下所示代码：

    [PRE46]

1.  运行控制台应用程序并注意成功的结果，如下所示代码：

    [PRE47]

1.  可选地，编写一个单元测试以执行与前面代码类似的检查，断言`Summary`具有正确的全名和总计。

**良好实践**：关于何时使用AutoMapper的讨论，您可以在以下链接的文章中阅读（文章底部有更多链接）：[https://www.anthonysteele.co.uk/AgainstAutoMapper.html](https://www.anthonysteele.co.uk/AgainstAutoMapper.html)。

**更多信息**：了解更多详情，请访问以下链接：[https://automapper.org/](https://automapper.org/)。

# 在单元测试中进行流畅的断言

**FluentAssertions**是一组扩展方法，可以使在单元测试中编写和阅读代码以及失败的测试的错误消息更接近自然语言，如英语。

它与大多数单元测试框架兼容，包括xUnit。当你为测试框架添加包引用时，FluentAssertions会自动找到该包并将其用于抛出异常。

在导入 `FluentAssertions` 命名空间后，对一个变量调用 `Should()` 扩展方法，然后调用数百种其他扩展方法之一以以人类可读的方式做出断言。您可以使用 `And()` 扩展方法链式调用多个断言，或者有单独的语句，每个语句都调用 `Should()`。

## 对字符串做出断言

让我们从对单个 `string` 值做出断言开始：

1.  使用您首选的代码编辑器将一个新的 **xUnit 测试项目** / `xunit` 命名为 `FluentTests` 添加到 `Chapter06` 解决方案中。

1.  在 `FluentTests` 项目中，添加对 `FluentAssertions` 的包引用，如下所示：

    [PRE48]

    `FluentAssertions` `7.0` 应在本书出版时可用。您可以在以下链接中检查：[https://www.nuget.org/packages/FluentAssertions/](https://www.nuget.org/packages/FluentAssertions/)。

1.  构建项目 `FluentTests`。

1.  将 `UnitTest1.cs` 重命名为 `FluentExamples.cs`。

1.  在 `FluentExamples.cs` 中，导入命名空间以使 `FluentAssertions` 扩展方法可用，并编写一个针对 `string` 值的测试方法，如下所示：

    [PRE49]

1.  运行测试：

    +   在 Visual Studio 2022 中，导航到 **测试** | **运行所有测试**。

    +   在 Visual Studio Code 中，在 **终端** 中输入 `dotnet test`。

1.  注意测试通过。

1.  在 `TestString` 方法中，对于 `city` 变量，删除 `London` 中的最后一个 `n`。

1.  运行测试并注意它失败，如下所示输出：

    [PRE50]

1.  在 `London` 中重新添加 `n`。

1.  再次运行测试以确认修复。

## 对集合和数组做出断言

现在让我们继续对集合和数组做出断言：

1.  在 `FluentExamples.cs` 中，添加一个测试方法来探索集合断言，如下所示代码：

    [PRE51]

1.  运行测试并注意集合测试失败，如下所示输出：

    [PRE52]

1.  将 `Charlie` 改为 `Charly`。

1.  运行测试并注意它们成功。

## 对日期和时间做出断言

让我们从对日期和时间值做出断言开始：

1.  在 `FluentExamples.cs` 中，导入命名空间以添加更多针对命名月份和其他有用的日期/时间相关功能的扩展方法，如下所示：

    [PRE53]

1.  添加一个测试方法来探索日期/时间断言，如下所示：

    [PRE54]

1.  运行测试并注意日期/时间测试失败，如下所示输出：

    [PRE55]

1.  对于 `due` 变量，将小时从 `11` 改为 `13`。

1.  运行测试并注意日期/时间测试成功。

**更多信息**：您可以在以下链接中了解更多关于 FluentAssertions 的详细信息：[https://fluentassertions.com/](https://fluentassertions.com/)。

# 验证数据

**FluentValidation** 允许您以人类可读的方式定义强类型验证规则。

您通过从 `AbstractValidator<T>` 继承来为类型创建验证器，其中 `T` 是您想要验证的类型。在构造函数中，您调用 `RuleFor` 方法来定义一个或多个规则。如果规则应在指定的场景中运行，则调用 `When` 方法。

## 理解内置验证器

FluentValidation 随带了许多有用的内置验证器扩展方法，用于定义规则，如下面的部分列表所示，其中一些您将在本节的编码任务中探索：

+   `Null`, `NotNull`, `Empty`, `NotEmpty`

+   `Equal`, `NotEqual`

+   `Length`, `MaxLength`, `MinLength`

+   `LessThan`, `LessThanOrEqualTo`, `GreaterThan`, `GreaterThanOrEqualTo`

+   `InclusiveBetween`, `ExclusiveBetween`

+   `ScalePrecision`

+   `Must`（即谓词）

+   `Matches`（即正则表达式），`EmailAddress`, `CreditCard`

+   `IsInEnum`, `IsEnumName`

## 执行自定义验证

创建自定义规则的最简单方法是使用 `Predicate` 编写自定义验证函数。您还可以调用 `Custom` 方法以获得最大控制。

## 自定义验证消息

有一些扩展方法用于在数据未通过规则时自定义验证消息的输出：

+   `WithName`: 更改消息中使用的属性名称。

+   `WithSeverity`: 将默认严重性从 `Error` 更改为 `Warning` 或其他级别。

+   `WithErrorCode`: 分配一个错误代码，可以在消息中输出。

+   `WithState`: 添加一些可以在消息中使用的状态。

+   `WithMessage`: 自定义默认消息的格式。

## 定义模型和验证器

让我们看看 FluentValidation 的实际应用示例。您将创建三个项目：

+   一个用于验证代表客户下单的模型的类库。

+   用于验证模型的验证器类库。

+   一个用于执行实时验证的控制台应用程序。

让我们开始：

1.  使用您首选的代码编辑器，将一个名为 `FluentValidation.Models` 的新 **类库**/ `classlib` 项目添加到 `Chapter06` 解决方案中。

1.  在 `FluentValidation.Models` 项目中，删除名为 `Class1.cs` 的文件。

1.  在 `FluentValidation.Models` 项目中，添加一个名为 `CustomerLevel.cs` 的新类文件，并修改其内容以定义一个包含三个客户级别 `Bronze`、`Silver` 和 `Gold` 的 `enum`，如下面的代码所示：

    [PRE56]

1.  在 `FluentValidation.Models` 项目中，添加一个名为 `Order.cs` 的新类文件，并修改其内容，如下面的代码所示：

    [PRE57]

1.  使用您首选的代码编辑器，将一个名为 `FluentValidation.Validators` 的新 **类库**/ `classlib` 项目添加到 `Chapter06` 解决方案中。

1.  在 `FluentValidation.Validators` 项目中，将 `Models` 项目添加为项目引用，并将 `FluentValidation` 包添加为包引用，如下面的标记所示：

    [PRE58]

1.  构建的 `FluentValidation.Validators` 项目。

1.  在 `FluentValidation.Validators` 项目中，删除名为 `Class1.cs` 的文件。

1.  在`FluentValidation.Validators`项目中，添加一个名为`OrderValidator.cs`的新类文件，并修改其内容，如下所示：

    [PRE59]

## 测试验证器

现在我们已经准备好创建一个控制台应用程序来测试模型上的验证器：

1.  使用您喜欢的代码编辑器，向`Chapter06`解决方案中添加一个名为`FluentValidation.Console`的新**控制台应用程序**/`console`项目。

1.  在`FluentValidation.Console`项目中，将警告视为错误，全局和静态导入`System.Console`类，并为`FluentValidation.Validators`和`FluentValidation.Models`添加项目引用，如下所示：

    [PRE60]

1.  构建项目`FluentValidation.Console`以构建引用的项目。

1.  在`Program.cs`中删除现有语句，然后添加创建订单并验证的语句，如下所示：

    [PRE61]

1.  运行控制台应用程序，并注意以下输出中显示的失败规则：

    [PRE62]

1.  取消注释设置文化的两个语句，以查看您本地语言和区域的输出。例如，如果您在法国（`fr-FR`），它将如下所示：

    [PRE63]

1.  设置订单的一些属性值，如下所示（高亮显示的代码）：

    [PRE64]

1.  将当前文化设置为美国英语，以确保您看到的输出与本书中的输出相同。您可以在以后尝试自己的文化设置。

1.  运行控制台应用程序，并注意以下输出中显示的失败规则：

    [PRE65]

1.  修改订单的一些属性值，如下所示（高亮显示的代码）：

    [PRE66]

1.  运行控制台应用程序，并注意订单现在有效，如下所示：

    [PRE67]

## 使用ASP.NET Core验证数据

对于使用ASP.NET Core进行自动数据验证，FluentValidation支持.NET Core 3.1及更高版本。

**更多信息**：在以下链接中了解更多详细信息：[https://cecilphillip.com/fluent-validation-rules-with-asp-net-core/](https://cecilphillip.com/fluent-validation-rules-with-asp-net-core/)。

# 生成PDF文件

在教授C#和.NET时，我经常被问到的一个最常见问题是：“有什么开源库可以用来生成PDF文件？”

生成PDF文件的授权库有很多，但多年来，找到跨平台的开源库一直很困难。

QuestPDF表示：“*如果您作为年收入超过100万美元的营利性公司/个人，以直接包依赖项的方式使用QuestPDF库在闭源软件中，您必须根据软件开发人员的数量购买QuestPDF专业版或企业版许可证。请参阅QuestPDF许可和定价网页以获取更多详细信息。（[https://www.questpdf.com/pricing.html](https://www.questpdf.com/pricing.html)）*”

较旧的2022.12.X版本将始终在MIT许可下可用，免费用于商业用途。如果您想支持库开发，请考虑购买2023.1.X或更高版本的Professional许可证。

## 在Apple硅Mac上使用QuestPDF

QuestPDF 使用 SkiaSharp，它为 Windows、Mac 和 Linux 操作系统提供了实现。因此，您在本节中创建的用于生成 PDF 的控制台应用程序是跨平台的。但在苹果硅 Mac（如我的 Mac mini M1）上，我必须安装 x64 版本的 .NET SDK，并使用 `dotnet new -a x64` 启动项目。这告诉 .NET SDK 使用 x64 架构，否则 SkiaSharp 库会报错，因为它们尚未构建以针对 ARM64。

## 创建用于生成 PDF 文档的类库

让我们看看 QuestPDF 的一个实际例子。您将创建三个项目：

+   用于表示具有名称和图像的产品类别目录的模型的类库。

+   用于文档模板的类库。

+   用于执行实时生成 PDF 文件的控制台应用程序。

让我们开始：

1.  使用您喜欢的代码编辑器，将一个名为 `GeneratingPdf.Models` 的新 **类库** / `classlib` 项目添加到 `Chapter06` 解决方案中。

1.  在 `GeneratingPdf.Models` 项目中，删除名为 `Class1.cs` 的文件。

1.  在 `GeneratingPdf.Models` 项目中，添加一个名为 `Category.cs` 的新类文件，并修改其内容以定义一个包含类别名称和标识符的两个属性的类，如下面的代码所示：

    [PRE68]

    之后，您将创建一个 `images` 文件夹，其文件名使用 `categoryN.jpeg` 的模式，其中 `N` 是从 1 到 8 的数字，与 `CategoryId` 值匹配。

1.  在 `GeneratingPdf.Models` 项目中，添加一个名为 `Catalog.cs` 的新类文件，并修改其内容以定义一个包含存储八个类别的属性的类，如下面的代码所示：

    [PRE69]

1.  使用您喜欢的代码编辑器，将一个名为 `GeneratingPdf.Document` 的新 **类库** / `classlib` 项目添加到 `Chapter06` 解决方案中。

1.  在 `GeneratingPdf.Document` 项目中，添加对 `QuestPDF` 的包引用和对 `Models` 类库的项目引用，如下面的标记所示：

    [PRE70]

1.  构建 `GeneratingPdf.Document` 项目。

1.  在 `GeneratingPdf.Document` 项目中，删除名为 `Class1.cs` 的文件。

1.  在 `GeneratingPdf.Document` 项目中，添加一个名为 `CatalogDocument.cs` 的新类文件。

1.  在 `CatalogDocument.cs` 中，定义一个实现 `IDocument` 接口的类，以定义一个包含页眉和页脚的模板，然后输出八个类别，包括名称和图像，如下面的代码所示：

    [PRE71]

## 创建用于生成 PDF 文档的控制台应用程序

现在我们可以创建一个控制台应用程序项目，它将使用类库生成 PDF 文档：

1.  使用您喜欢的代码编辑器，将一个名为 `GeneratingPdf.Console` 的新 **控制台应用程序** / `console` 项目添加到 `Chapter06` 解决方案中。

1.  在 `GeneratingPdf.Console` 项目中，创建一个 `images` 文件夹，并从以下链接下载 1 到 8 的八个类别图像到该文件夹：[https://github.com/markjprice/apps-services-net8/tree/master/images/Categories](https://github.com/markjprice/apps-services-net8/tree/master/images/Categories)。

1.  如果你使用的是 Visual Studio 2022 或 JetBrains Rider，那么 `images` 文件夹及其文件必须复制到 `GeneratingPdf.Console\bin\Debug\net8` 文件夹：

    1.  在 **解决方案资源管理器** 中，选择所有图像。

    1.  在 **属性** 中，将 **复制到输出目录** 设置为 **始终复制**。

    1.  打开项目文件，注意将八张图片复制到正确文件夹的 `<ItemGroup>` 条目，如下所示的部分标记：

    [PRE72]

1.  在 `GeneratingPdf.Console` 项目中，将警告视为错误，全局和静态导入 `System.Console` 类，并为 `Document` 模板类库添加项目引用，如下所示的部分标记：

    [PRE73]

1.  构建 `GeneratingPdf.Console` 项目。

1.  在 `Program.cs` 中，删除现有的语句，然后添加语句来创建目录模型，将其传递给目录文档，生成 PDF 文件，然后尝试使用适当的操作系统命令打开文件，如下面的代码所示：

    [PRE74]

    `Process` 类及其 `Start` 方法也应该能够在 Mac 和 Linux 上启动进程，但正确获取路径可能很棘手，所以我将其留作读者的可选练习。你可以在以下链接中了解更多关于 `Process` 类及其 `Start` 方法的知识：[https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process.start](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process.start).

1.  运行控制台应用程序，并注意生成的 PDF 文件，如图 *6.4* 所示：

![](img/B19587_06_04.png)

图 6.4：从 C# 代码生成的 PDF 文件

**更多信息**: 在以下链接中了解更多详情：[https://www.questpdf.com/](https://www.questpdf.com/).

# 练习和探索

通过回答一些问题、进行一些实际操作练习，并对本章中的主题进行深入研究来测试你的知识和理解。

## 练习 6.1 – 测试你的知识

使用网络回答以下问题：

1.  所有时间中最受欢迎的第三方 `NuGet` 包是什么？

1.  你在 `ImageSharp Image` 类上调用哪个方法来执行像调整大小或用灰度替换颜色这样的更改？

1.  使用 `Serilog` 进行日志记录的主要好处是什么？

1.  Serilog 沉淀器是什么？

1.  你是否应该始终使用像 `AutoMapper` 这样的包来在对象之间进行映射？

1.  你应该调用哪个 `FluentAssertions` 方法来开始对值进行流畅断言？

1.  你应该调用哪个 `FluentAssertions` 方法来断言序列中的所有项目都符合某个条件，例如一个 `string` 项目必须少于六个字符？

1.  你应该从哪个 `FluentValidation` 类继承来定义自定义验证器？

1.  使用 `FluentValidation`，你如何设置仅在特定条件下应用的规则？

1.  使用 `QuestPDF`，你必须实现哪个接口来定义 PDF 文档，以及该接口必须实现哪些方法？

## 练习 6.2 – 探索主题

使用下一页上的链接了解本章涵盖主题的更多详细信息：

[https://github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-6---implementing-popular-third-party-libraries](https://github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-6---implementing-popular-third-party-libraries)

# 摘要

在本章中，您探索了一些受.NET开发者欢迎的第三方库，用于执行以下功能：

+   使用微软推荐的第三方库ImageSharp来操作图像。

+   使用Humanizer使文本、数字、日期和时间更友好。

+   使用Serilog记录结构化数据。

+   对象之间的映射，例如，实体模型到视图模型。

+   在单元测试中进行流畅的断言。

+   以本地文化语言可读的方式验证数据。

+   生成PDF文件。

在下一章中，我们将学习如何处理日期和时间的国际化以及本地化，包括.NET 8中的一个新类型，它使得对依赖于当前时间的组件进行单元测试更容易。

# 在Discord上了解更多

要加入本书的Discord社区——在那里您可以分享反馈、向作者提问，并了解新版本——请扫描下面的二维码：

[https://packt.link/apps_and_services_dotnet8](https://packt.link/apps_and_services_dotnet8)

![二维码](img/QR_Code3048220001028652625.png)
