# 14

# 创建您自己的约定

相关性是可观察性最重要的部分之一。分布式跟踪通过传播跟踪上下文带来相关性，使我们能够跟踪单个操作，而一致的属性使跟踪和其他遥测信号之间的相关性成为可能。

在 *第九章* 的 *最佳实践* 中，我们讨论了重用标准属性和遵循 OpenTelemetry 语义约定的重要性。有时我们需要更进一步，定义我们自己的约定。在这里，我们将探讨如何定义自定义属性和约定，并在整个系统中使用它们。

首先，我们将列出应在整个系统中标准化的属性，然后我们将探讨如何使用共享代码来填充它们。最后，我们将查看 OpenTelemetry 的语义约定模式，并了解它如何简化自定义约定的文档和验证。

在本章中，你将学习以下内容：

+   识别和记录常见的属性和约定

+   在整个系统中共享仪器和自定义约定

+   使用 OpenTelemetry 工具创建约定

通过这种方式，你应该能够创建易于使用的流程和工具，以保持自定义遥测和属性的一致性和稳定性。

# 技术要求

本章的代码可在 GitHub 上的书籍存储库中找到：[`github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/tree/main/chapter14`](https://github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/tree/main/chapter14)。

要运行示例并执行分析，我们需要以下工具：

+   .NET SDK 7.0 或更高版本

+   Docker 和 `docker-compose`

# 定义自定义约定

表达最基本的事情也有多种方式。如果我们以 *第五章* 的 meme 应用程序示例，*配置和控制平面* 为例，我们为所有跨度添加了 meme 名称属性，以便我们可以在 meme 上传时找到它或了解它被访问的频率。

我们选择了那种方法，但我们可以先记录一次 meme 名称的日志，然后使用稍微复杂一些的查询来找到与该 meme 相关的所有跟踪。我们可以想出其他方法，但重要的是在整个系统中保持方法的一致性。

即使为每个跨度添加了自定义属性，在记录这样的属性时仍有许多事情需要考虑：

+   `meme_name`、`meme.name` 和 `memeName` 是不同的属性。除非我们记录确切的名称并将其定义为某个地方的一个常量，否则最终有人会使用错误的变体。

+   **类型**：主题名称只是一个字符串。如果我们想捕获大小或图像格式呢？我们需要记录类型，并可能提供辅助方法来记录属性，以便更容易地正确设置它们。

+   `image/png` 或作为一个枚举？

当记录 meme 名称时，我们从上传的文件名中提取它，并需要记录在属性值上应该记录什么（绝对路径或相对路径、文件名、带或不带扩展名）。如果在编写业务逻辑时，我们对 meme 名称进行清理或转义，或生成一个唯一的名称，我们可能想要捕获业务逻辑使用的那个。我们也可能只记录一次原始名称，用于调试目的。

+   **何时填充属性**：记录在哪些指标、跨度和对数上应该记录属性。例如，meme 名称具有高基数，不属于指标。meme 名称属性可以记录在跨度和对数上，但哪些可以？在我们的例子中，在*第五章*的*配置和控制平面*中，我们在所有跨度上记录了 meme 名称，并在几个特定的日志记录上记录了它。

你可能还想记录其他方面：跨度之间的关系、事件名称、是否在跨度上记录异常、属性基数、稳定性等等。

我们将在本章的*使用 OpenTelemetry 模式和工具*部分中看到如何正式定义属性。现在，让我们专注于命名。

## 命名属性

命名被认为是计算机科学中最难的问题之一。

如果我们将 meme 名称属性命名为`document.id`，它可能与数据库模式中的一个属性精确匹配。然而，它将非常通用，并且与系统中另一个类似概念发生冲突的概率很高。不熟悉内部结构但分析业务数据的人可能很难理解`document.id`代表什么。

`meme_name`看起来直观、简短且描述性强，与其他事物的冲突概率很低。这似乎是可行的，但我们可能还有其他属性，应该将它们放入特定于应用程序或公司的命名空间中。

### 命名空间

命名空间允许我们设置唯一、具体、描述性和一致的名称。

由于我们称我们的系统为`memes`，让我们将其用作根命名空间。这将帮助我们理解这个空间中的所有属性都来自我们的自定义，而不是由某些自动工具设置。

这有助于我们在遥测之间导航，并使得在遥测管道中进行过滤、编辑和其他后处理成为可能。例如，如果我们想从日志中删除未知属性，我们可以考虑`memes`命名空间（http、`db`等）中的所有内容都是已知的。

我们可以有嵌套的命名空间。由于我们希望记录其他 meme 属性，例如大小和类型，我们最终可能会得到以下一组属性：`memes.meme.name`、`memes.meme.size`和`memes.meme.type`。我们可以根据需要轻松添加其他属性，例如`memes.meme.author`或`memes.meme.description`。

虽然`memes.meme`可能看起来重复，但一旦我们添加了诸如`memes.user.name`或`memes.tag.description`之类的元素，它就会开始更有意义。

OpenTelemetry 命名约定（可在[`github.com/open-telemetry/opentelemetry-specification/blob/main/specification/common/attribute-naming.md`](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/common/attribute-naming.md)找到）使用点（`.`）作为命名空间之间的分隔符。

对于多词命名空间或属性，OpenTelemetry 建议使用`snake_case`：例如，我们可以引入`memes.meme.creation_date`。

遵循此约定为我们自定义属性，使我们能够在所有属性中保持一致性。它还将减少在编写查询时的错误机会。

在某种形式下定义和记录模式是一个基本步骤。但如何使其与代码保持同步并确保所有服务都遵循它？做到这一点的一种方法是将它捕获在代码中并在整个系统中重用它。

# 共享公共模式和代码

一致的遥测报告适用于遥测收集配置。首先，我们需要在所有服务上启用基本层级的仪器，这应该包括资源利用率指标、跟踪和 HTTP、gRPC 或系统使用的任何其他 RPC 协议的指标。

我们还应该配置采样和资源属性，添加丰富处理程序，并设置上下文传播器。

单个服务应该能够在一定程度上自定义配置：添加更多仪器、启用自定义活动源和仪表、或控制日志详细程度。

统一配置的最简单方法是将相应的代码作为公共库（或一组库）分发，这些库在系统中的所有服务之间共享。这样的库将定义配置选项，提供辅助方法以启用遥测收集，实现常见的丰富处理程序，声明跨服务事件等。让我们继续实现这样的配置助手。

## 共享设置代码

在*第五章*“配置和控制平面”以及其他章节中，我们使用了 meme 应用程序，我们在每个服务中单独应用了 OpenTelemetry 配置。

我们永远不会在生产代码中这样做——很难保持我们的配置、仪器选项、OpenTelemetry 包版本和其他一切同步。

为了解决这个问题，我们可以开始将常见的仪器代码片段提取到一个共享库中。由于配置可能从服务到服务略有不同，我们需要定义一些配置选项。

我们需要一些选项来帮助我们设置服务名称，指定采样率或策略，启用额外的仪器，等等。你可以在书籍仓库中的`MemesTelemetryConfiguration`类中找到一个这样的选项示例。

然后，我们可以声明一个处理 OpenTelemetry 配置的辅助方法。以下是这个方法的示例：

OpenTelemetryExtensions.cs

```cs
public static void ConfigureTelemetry(
  this WebApplicationBuilder builder,
  MemesTelemetryConfiguration config)
{
  var resourceBuilder = GetResourceBuilder(config);
  var sampler = GetSampler(config.SamplingStrategy,
    config.SamplingProbability);
  builder.Services.AddOpenTelemetry()
    .WithTracing(builder => builder
      .SetSampler(sampler)
      .AddProcessor<MemeNameEnrichingProcessor>()
      .SetResourceBuilder(resourceBuilder)
      .AddHttpClientInstrumentation(o =>
        o.ConfigureHttpClientCollection(
          config.RecordHttpExceptions))
      .AddAspNetCoreInstrumentation(o =>
         o.ConfigureAspNetCoreCollection(
           config.RecordHttpExceptions,
           config.AspNetCoreRequestFilter))
      .AddCustomInstrumentations(config.ConfigureTracing)
      .AddOtlpExporter())
  ...
}
```

[`github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/Memes.OpenTelemetry.Common/OpenTelemetryExtensions.cs`](https://github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/Memes.OpenTelemetry.Common/OpenTelemetryExtensions.cs)

使用此方法的所有服务都将应用和丰富相同的基本级别的仪表。 

这里是一个在**存储**服务中使用此方法的示例：

storage/Program.cs

```cs
var config = new MemesTelemetryConfiguration();
builder.Configuration.GetSection("Telemetry").Bind(config);
config.ConfigureTracing = o => o
  .AddEntityFrameworkCoreInstrumentation();
builder.ConfigureTelemetry(config);
```

[`github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/storage/Program.cs`](https://github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/storage/Program.cs)

在这里，我们从 ASP.NET Core 配置的 `Telemetry` 部分读取遥测选项，您可以根据自己的需要以任何方式填充它。

然后，我们添加了 Entity Framework 仪表。只有**存储**服务需要它。

注意

拥有一个中央库，它能够实现收集功能，有助于减少版本混乱。通过依赖它，各个服务包会获得对 OpenTelemetry 包的传递依赖，并且不应将其作为直接依赖添加。需要不常见仪表库的服务仍然需要安装相应的 NuGet 包。

现在我们有了共同的设置，让我们看看我们可以做些什么来帮助服务遵循我们的自定义语义约定。

## 约定法典化

在上一个示例中，我们开始记录 meme 名称属性——我们启用了 `MemeNameEnrichingProcessor`，它将 `memes.meme.name` 属性设置在每个跨度上。各个服务不需要做任何事情来启用它，也不能设置错误的属性名称。

尽管如此，我们可能还需要在其他某些部分（例如日志）中直接使用属性，因此声明属性名称为常量并永远不在代码中使用字符串字面量是很重要的。以下是一个演示如何声明属性名称的示例：

SemanticConventions.cs

```cs
public class SemanticConventions
{
    public const string MemeNameKey = "memes.meme.name";
    public const string MemeSizeKey = "memes.meme.size";
    public const string MemeTypeKey = "memes.meme.type";
    …
}
```

[`github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/Memes.OpenTelemetry.Common/SemanticConventions.cs`](https://github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/Memes.OpenTelemetry.Common/SemanticConventions.cs)

OpenTelemetry 还提供了 `OpenTelemetry.SemanticConventions` NuGet 包，它声明了规范中定义的常见属性。在重用常见属性时添加对其的依赖可能是有意义的。

因此，我们现在已经为属性名称定义了常量，并且有一个填充 meme 名称的处理器。我们能做更多吗？

我们可以提供助手以高效且一致地报告常见事件。让我们看看我们如何使用我们在*第八章*中探索的高性能日志记录，*编写结构和关联日志*，来填充我们的属性：

EventService.cs

```cs
private static readonly Action<ILogger, string, string?,
  long?, string, string, Exception> LogDownload =
    LoggerMessage.Define<string, string?,
      long?, string, string>(
    LogLevel.Information,
    new EventId(1),
    $"download {{{SemanticConventions.MemeNameKey}}}
    {{{SemanticConventions.MemeTypeKey}}}
    {{{SemanticConventions.MemeSizeKey}}}
    {{{SemanticConventions.EventNameKey}}}
    {{{SemanticConventions.EventDomainKey}}}");
  …
  public void DownloadMemeEvent(string memeName,
    MemeContentType type,
    long? memeSize) =>
  LogDownload(_logger,
    memeName,
    ContentTypeToString(type),
    memeSize,
    SemanticConventions.DownloadMemeEventName,
    SemanticConventions.MemesEventDomain,
    default!);
```

[`github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/Memes.OpenTelemetry.Common/EventService.cs`](https://github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/Memes.OpenTelemetry.Common/EventService.cs)

在这里，我们有一些难以阅读的代码。它通过遵循 OpenTelemetry 约定和我们的约定，定义了一个表示表情包下载事件的日志记录。我们并不希望每次需要记录某些内容时都编写此代码。

在公共库中实现此事件一次，并使其易于重用，是记录事件一致性和低性能开销的最佳方式。

现在，任何人都可以使用`DownloadMemeEvent`方法，如下面的示例所示：

StorageService.cs

```cs
_events.DownloadMemeEvent(name, MemeContentType.PNG,
    response.Content.Headers.ContentLength);
```

[`github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/frontend/StorageService.cs`](https://github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/frontend/StorageService.cs)

这很容易使用且性能良好，无需担心属性、它们的类型或任何约定。如果属性被重命名，无需更新服务代码——所有这些都隐藏在共享库中。

如果我们遵循这种方法，我们可以定义其他事件并为指标和跟踪添加辅助方法来填充属性组。

如果我们需要任何自定义的仪器，就像我们为 gRPC 或消息传递所做的那样，我们应该将它们放入共享库中，并在那里应用所有属性，而不是在服务代码中。

将与遥测相关的代码从业务逻辑中分离出来，使它们都更容易阅读和维护。这也使得与遥测相关的代码可测试，并帮助我们保持它与文档的一致性。这也使得通过测试强制执行约定变得容易，并在代码审查期间，当共享代码更改由语义约定控制的某些内容时，可以注意到这些更改。

在代码中定义语义约定对于某些应用来说是足够的。其他可能使用不同语言或有一些其他约束的应用，不能仅仅依赖共享代码。无论如何，公司中的每个人都可以使用遥测数据来进行业务报告和满足任何非技术需求。因此，将它们与代码分开单独记录是很重要的。

让我们看看我们如何使用 OpenTelemetry 工具来实现这一点。

# 使用 OpenTelemetry 模式和工具

我们实际上并不关心如何记录自定义语义约定。目标是有一个一致且具体的约定，易于阅读和遵循。让我们看看 OpenTelemetry 语义约定模式如何帮助解决这个问题。

## 语义约定模式

到目前为止，当我们谈论语义约定时，我们提到了如 [`github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/semantic_conventions/http.md`](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/semantic_conventions/http.md) 这样的 Markdown 文件。这些文件是真相的来源，但在这里，我们将看看其背后的实现细节。

这些文件中描述属性的表格通常是自动生成的。属性定义在遵循 OpenTelemetry 语义约定架构的 YAML 文件中。

YAML 文件可以在不同的语义约定和信号之间共享，然后使用脚本一致地写入所有 Markdown 文件。

让我们看看如何在 YAML 文件中定义我们的 meme 属性，以了解其架构：

memes-common.yaml

```cs
groups:
  - id: memes.meme
    type: attribute_group
    brief: "Describes memes attributes."
    prefix: memes.meme
    attributes:
      - id: name
        type: string
        requirement_level: required
        brief: 'Unique and sanitized meme name'
        examples: ["this is fine"]
```

[`github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/semconv/memes-common.yaml`](https://github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/semconv/memes-common.yaml)

在这里，我们在 `memes.meme` 命名空间（由 `prefix` 属性定义）内定义了 `name` 属性，其类型为 `string`。这是一个必需属性，因为在 *第五章*，*配置和控制平面* 中，我们决定在所有跨度上记录 meme 名称属性。

OpenTelemetry 支持多个需求级别：

+   `required`：遵循此约定的任何遥测项都必须设置该属性。

+   `conditionally_required`：当满足条件时，必须填充该属性。例如，http.route 仅在启用路由且存在路由时填充。

+   `recommended`：应填充该属性，但可能被删除或禁用。可观察性后端和工具不应依赖于其可用性。这是默认级别。

+   `opt_in`：默认情况下不填充该属性，但已知并已记录，并且可以在明确启用时添加。

让我们看看如何定义 `size` 属性：

memes-common.yaml

```cs
- id: size
  type: int
  requirement_level: opt_in
  brief: 'Meme size in bytes.'
  examples: [49335, 12345]
```

[`github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/semconv/memes-common.yaml`](https://github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/semconv/memes-common.yaml)

`size` 属性具有 `int` 类型（并映射到 `int64` 或 `long`），并且具有 `opt-in` 级别，因为我们不希望在默认情况下记录所有遥测数据。

最后，我们可以定义 `type` 属性：

memes-common.yaml

```cs
- id: type
  type:
  members:
    - id: png
      value: "png"
      brief: 'PNG image type.'
    - id: jpg
      value: "jpg"
      brief: 'JPG image type.'
    - id: unknown
      value: "unknown"
      brief: 'unknown type.'
  requirement_level: opt_in
  brief: 'Meme image type.'
  examples: ['png', 'jpg']
```

[`github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/semconv/memes-common.yaml`](https://github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/semconv/memes-common.yaml)

在这里，我们将 `type` 定义为一个枚举。仪器必须使用这里定义的值之一或将 `type` 设置为 `unknown`。

希望你的团队不会在 JPEG 和 JPG 之间花费太多时间决定——两者都可以。重要的是要选择并记录一个选项。

你可以在 GitHub 上 OpenTelemetry 构建工具仓库中找到完整的模式定义（https://github.com/open-telemetry/build-tools/blob/main/semantic-conventions/syntax.md）。它还包含你的 IDE 可能用于自动完成和验证模式文件的模式定义。

现在我们已经定义了一些属性，让我们使用另一个 OpenTelemetry 工具来验证模式文件，并在 Markdown 文件中生成内容。

如果你查看书籍仓库中的原始 `memes.md` 文件，它包含以下注释的文档：

memes.md

```cs
<!-- semconv memes.meme -->
...
<!-- endsemconv -->
```

[`github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/semconv/memes.md`](https://github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/semconv/memes.md)

这些行之间的内容是从具有 `memes.meme` 标识符的 YAML 组自动生成的。我们可以使用以下命令重新生成此内容（确保指定路径）：

```cs
chapter14$ docker run \
  -v /path/to/chapter14/semconv:/source \
  -v /path/to/chapter14/semconv:/destination \
  otel/semconvgen:latest \
  -f /source markdown -md /destination
```

在这里，我们使用 `otel/semconvgen` 图像中的 Markdown 生成器。我们挂载了 `source` 和 `destination` 卷。生成器递归地解析 `source` 文件夹中找到的所有 YAML 文件，然后根据我们之前看到的 `semconv` 注释，在 `destination` 文件夹中生成 Markdown 文件中的属性表。

生成是蛋糕上的樱桃，你可能一开始不需要它。不过，如果你决定使用 OpenTelemetry 语义约定模式，请确保使用 `otel/semconvgen` 工具来 *验证* YAML 文件，你可以在 CI 运行过程中这样做；只需将 `–md-check` 标志添加到之前的命令中。

工具还支持使用 Jinja 模板（https://jinja.palletsprojects.com）在代码中生成属性定义。我们只需要为 `SemanticConventions.cs` 文件创建一个 Jinja 模板，然后运行 `otel/semconvgen` 生成器。

我们还可以定义跟踪、指标或特定事件的约定。让我们为事件做这件事。

## 定义事件约定

段子上传和下载事件对于业务报告非常重要。我们实际上不能将它们作为指标暴露——段子名称具有高基数，我们感兴趣的是找到最受欢迎的或者测量每个段子的其他事物。

为了避免破坏业务报告，我们需要确保事件定义得足够精确，并且有良好的文档记录。为了实现这一点，我们可以以下述方式声明一个事件：

memes-events.yaml

```cs
- id: meme.download.event
  type: event
  prefix: download_meme
  brief: "Describes meme download event."
  Attributes:
    - ref: memes.meme.name
    - ref: memes.meme.size
      requirement_level: required
    - ref: memes.meme.type
      requirement_level: required
```

[`github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/semconv/memes-events.yaml`](https://github.com/PacktPublishing/Modern-Distributed-Tracing-in-.NET/blob/main/chapter14/semconv/memes-events.yaml)

在这里，我们声明了一个具有 `event` 类型的组（之前，我们使用 `attribute_group`，它是信号无关的）。我们提供了一个前缀（`download_name`），用于记录事件名称。我们添加了对先前定义的属性的引用，但现在仅在这些事件上需要 `size` 和 `type` 属性的存在。

您可能已经注意到事件名称不包含命名空间——在这里，我们遵循 `event` 语义约定。如果您查看 `EventService` 类的相应代码片段，我们也会记录 `event.domain` 属性，该属性作为命名空间。

采用这种方法，我们可以定义跨度的或度量约定，这些约定重用了相同的公共属性。

这些模式或相应的 Markdown 文件将定义和记录遥测生产者和消费者之间的合同，无论他们使用什么语言。

# 摘要

在本章中，我们讨论了在不同系统内保持自定义遥测和属性一致性的不同方法。我们确定了需要记录的属性属性，并了解了属性命名约定。

保持遥测的一致性是一个挑战。我们探讨了如何通过共享公共仪表化代码来使其更容易，包括 OpenTelemetry 设置和实用方法，这些方法报告具有正确名称和类型的属性。

最后，我们了解了 OpenTelemetry 语义约定模式和工具，这可能有助于您定义、验证和自动化自定义约定的文档过程。

在项目的早期阶段为遥测定义一个公共模式将为您组织节省大量时间，现在您拥有了知识和工具来实现这一点。在下一章中，我们将讨论棕色地带系统，其中新解决方案与旧解决方案共存，我们将看到对齐不同标准和约定有多么困难。

# 问题

1.  警报、仪表板和用法报告很可能依赖于自定义遥测并依赖于约定。你将如何处理演变约定以防止破坏关键部分？

1.  是否可以验证来自某些服务的遥测是否符合定义的语义约定？
