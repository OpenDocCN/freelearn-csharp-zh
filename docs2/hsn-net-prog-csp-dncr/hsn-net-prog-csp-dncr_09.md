# 线上错误处理

本章将探讨分布式应用的许多可能的故障点，以及故障的影响如何被你的应用的下游消费者感受到。我们将检查不同错误根据严重性、上下文和网络流量生命周期的阶段如何被报告或检测。我们将探讨 C#中实现的多种错误处理策略，并展示如何利用约定和标准确保你的应用对任何潜在的下游消费者都表现出预期的行为。最后，我们将探讨如何在请求无法合理服务时为你的应用消费者生成有意义的错误。

本章将涵盖以下主题：

+   不同的故障点应生成不同的错误消息，以及如何从中恢复

+   已正确实现各自通信协议的服务返回的常见错误代码和消息

+   根据应用必须满足的需求处理不同类型错误的策略

+   使用状态码、错误、日志和消息为下游消费者生成和报告自己的错误

# 技术要求

我们将编写大量的示例代码，这些代码可以在 GitHub 上找到：[`github.com/PacktPublishing/Hands-On-Network-Programming-with-CSharp-and-.NET-Core/tree/master/Chapter%207`](https://github.com/PacktPublishing/Hands-On-Network-Programming-with-CSharp-and-.NET-Core/tree/master/Chapter%207)

查看以下视频以查看代码的实际应用：[`bit.ly/2HT1l9z`](http://bit.ly/2HT1l9z)

本章将介绍一个名为**Polly**的弹性网络客户端来演示常见的错误恢复策略。我建议在这里了解该特定库的一些功能：[`github.com/App-vNext/Polly.`](https://github.com/App-vNext/Polly)

# 多设备，多故障点

当你引入网络交互的不确定性时，即使是简单的软件也可能出现无数的问题。上游服务中一个简单的索引错误可能导致 JSON 字符串中缺少闭合花括号，使得整个有效负载无法解析。**互联网服务提供商**（**ISP**）服务中断或弱无线信号可能导致超时和有效负载不完整交付。同时，你请求资源的远程系统的稳定性完全超出你的控制。由于所有这些因素都引入了错误的可能性，我们不能简单地希望避免软件中的错误或异常。我们必须假设它们会发生，并围绕这种可能性进行设计。

# 外部依赖

在我们作为专业工程师的时间里，我们可以用一只手数的过来，我们写的应用程序既没有作为下游消费者的网络依赖，也没有对上游网络资源有依赖。每次你的软件必须进行网络跳转以访问必要的资源时，你都在引入失败的风险。

通常情况下，每次从外部依赖中读取数据时，你必须实现适当的异常处理。始终假设可能会出错。我们在上一章没有这样做，是因为我们不想在试图完全阐明数据流的概念和使用方法时引入不必要的复杂性。然而，在这一章中，我们将专门探讨错误处理策略。首先的策略是**始终**假设访问外部依赖最终会失败。

当你处理另一个外部依赖的响应时，这相当直接，但如果你自己的软件是另一个应用程序的依赖呢？对于具有弹性应用程序行为的下一个策略是始终假设你自己的软件最终会失败。这将鼓励你考虑到这一点，并为在失败时刻使用你软件的任何人提供容错性和有用的错误信息。考虑到这一点，让我们从上一章的网络访问代码开始，并对其进行修改以增强其弹性。

回顾我们的方法，我们有以下内容：

```cs
public async Task<ResultObject> AsyncMethodDemo() {
  ResultObject result = new ResultObject();
  WebRequest request = WebRequest.Create("http://test-domain.com");
  request.Method = "POST";
  Stream reqStream = request.GetRequestStream();

  using (StreamWriter sw = new StreamWriter(reqStream)) {
    sw.Write("Our test data query");
  }
  var responseTask = request.GetResponseAsync();

  result.LocalResult = LongRunningSlowMethod();

  var webResponse = await responseTask;

  using (StreamReader sr = new StreamReader(webResponse.GetResponseStream())) {
    result.RequestResult = await sr.ReadToEndAsync();
  }

  return result;
}
```

在这个方法中，我们有一个外部依赖。当我们尝试访问和处理从服务器收到的响应时，可能会遇到失败。由于任何原因，都可能在这里出现任何数量的问题，因此我们希望将这段代码包裹在`try`/`catch`块中，或者在代码中应用异常过滤器（稍后会更详细地介绍）。我们将从一个简单的`try`/`catch`块开始，查看对我们目的非常有用的内置`Exception`类，即`WebException`类。所以让我们捕捉它，看看我们能从中获得什么样的效用：

```cs
try {
  var webResponse = await responseTask;

  using (StreamReader sr = new StreamReader(webResponse.GetResponseStream())) {
    result.RequestResult = await sr.ReadToEndAsync();
  }
} catch (WebException ex) {
  Console.WriteLine(ex.Status);
  Console.WriteLine(ex.Message);
}
```

这里，你会注意到我们不必在阻塞我们的代码并等待响应返回之前寻找异常。如果我们启动一个异步任务，在执行过程中该任务抛出错误，那么它不会到达我们的代码，直到我们`await`该任务的结果。当我们捕获我们知道的错误（因为[`test-domain.com/`](http://test-domain.com/)资源实际上不存在）时，我们将其捕获为`WebException`。这个类是你从代码中遇到的任何特定于网络的异常中接收到的基异常类。与`catchall Exception`类相比，使其特别有用的是网络错误特定的`Status`属性可用。

在这个示例中，我们只是在控制台记录状态和异常消息。然而，如果这段代码存在于我们编写的 API 中，并且通过网络暴露给下游实体，我们就需要负责返回一个有意义的自定义状态码。这样做可以确保如果我们的特定应用程序代码是流程中的主要故障点，我们将尽可能提供信息，以便可靠地响应和恢复异常。

# 解析异常状态以获取上下文

当在`WebException`中返回错误状态时，该属性的值可以告诉我们很多关于失败原因的信息。`Status`属性是`WebExceptionStatus`枚举的一个实例，返回的值可以告诉我们很多关于导致我们的外部依赖失败的条件。这可能是一个路由问题，或者无法解析缓存查找或保持活跃连接。

仅通过检查状态码的值，你可以了解到很多关于具体失败原因和最有可能产生积极结果的恢复策略。例如，如果你的异常的`Status`是`NameResolutionFailure`，你可以安全地假设重试请求不会是一个有效的策略。如果在第一次尝试中 DNS 未能根据提供的名称识别主机，那么使用相同主机名的后续尝试不太可能取得成效。然而，如果你收到一个`Timeout`类型的错误状态，你可能会增加请求客户端的超时阈值并提交一系列重试，直到达到预定的最大超时长度。

`WebException`状态的文档是免费可用的，你需要自己确定可能遇到的哪些可能的异常状态。此外，一旦你知道请求链中失败的位置或原因，你就可以确定最适合你应用程序代码的最佳恢复策略。然而，这里的主要启示是，你应该在任何请求传输的应用程序点检查并尝试从`WebException`事件中恢复。

# 状态码和错误信息

现在我们知道了我们应该在哪里检查潜在的异常（或为我们消费者提供它们），了解这些异常可能最终看起来是什么样子是很重要的。确定可能的异常的完整范围并为每种可能性编写恢复解决方案将使我们的代码几乎对分布式资源获取的不可靠性具有防弹效果。虽然我们已经看到，我们只需检查由任何失败的网路请求抛出的`WebException`异常，就可以访问到极其有用的信息，但关于互联网状态码规范和异常响应处理的标准还有很多需要了解。

# 状态信息和状态码

首先，让我们看看在处理上游依赖项错误的不同状态响应时的一个可靠方法。让我们使用我们之前示例中的相同代码片段，但更稳健地响应我们可能收到的各种状态。为了使我们的请求代码更短，我们将异常处理代码委托给一个名为`ProcessException(WebException ex)`的不同方法。这个方法的两参数将是生成的异常，以及触发我们代码中异常状态的原请求。这将给异常处理方法足够的上下文，以便尝试优雅地从错误中恢复。因此，在我们的早期示例的`catch`块中，我们将相应地替换我们的两个`Console.WriteLine()`语句：

```cs
} catch (WebException ex) {
    ProcessException(ex);
}
```

然后，在`ProcessException(WebException ex)`方法内部，我们将根据异常的`Status`属性的可能值进行切换，根据接收到的状态或消息执行有用的恢复逻辑：

```cs
public void ProcessException(WebException ex) {
    switch(ex.Status) {
      case WebExceptionStatus.ConnectFailure:
      case WebExceptionStatus.ConnectionClosed:
      case WebExceptionStatus.RequestCanceled:
      case WebExceptionStatus.PipelineFailure:
      case WebExceptionStatus.SendFailure:
      case WebExceptionStatus.KeepAliveFailure:
      case WebExceptionStatus.Timeout:
        Console.WriteLine("We should retry connection attempts");
        break;
      case WebExceptionStatus.NameResolutionFailure:
      case WebExceptionStatus.ProxyNameResolutionFailure:
      case WebExceptionStatus.ServerProtocolViolation:
      case WebExceptionStatus.ProtocolError:
        Console.WriteLine("Prevent further attempts and notify consumers to check URL configurations");
        break;
      case WebExceptionStatus.SecureChannelFailure:
      case WebExceptionStatus.TrustFailure:
        Console.WriteLine("Authentication or security issue. Prompt for credentials and perhaps try again");
        break;
      default:
        Console.WriteLine("We don't know how to handle this. We should post the error message and terminate our current workflow.");
        break;     
    }
}
```

通过使用`WebException`返回的描述性和可靠的状态代码，我们可以将类似的错误分组在一起，并针对它们提供解决方法，这些解决方法可能会解决每个问题的共同问题。如果有连接或超时问题，可能只是你的 ISP 有问题，或者远程主机没有从缓存中加载资源，因此处理请求花费了太多时间。在这种情况下，简单地重试可能确实是一个持续可靠的解决方案。然而，如果异常是由于无法解析目标主机名，那么后续请求可能会以相同的方式失败。它们都会由相同的 DNS 处理，除非请求 URI 更新为有效的域名，否则重试请求没有好处。同时，安全问题可能通过刷新身份验证或授权凭据来解决。

你会注意到默认建议发布与`WebException`类返回的内部消息。这是因为，在没有服务器返回通用状态代码的情况下，该类本身将有一些关于可能发生什么问题的默认消息。因此，即使我们得到`WebExceptionStatus.UnknownError`的实例，错误消息中也可能返回一些有用的信息。

# 有用的错误消息

如果我们发现自己处于一个场景中，无法优雅地从尝试从上游依赖项请求资源失败中恢复，并且无法继续提供我们的应用程序提供的服务，那么是时候发送我们自己的错误消息了。我们有责任尽可能提供关于失败状态的信息，以便用户理解出了什么问题，同时避免发送任何可能使我们的应用程序容易受到漏洞攻击的潜在敏感细节。

这就是状态码成为你最好朋友的地方。当你处理针对你应用的 HTTP 请求时，你应该尽可能具体地指定你返回的状态码。每次出问题时返回一个 500 状态码可能看起来非常简单，因为 5XX 是服务器错误的通用代码标识。然而，如果你想让人们愿意使用你的服务，我建议你不要这样做。你越具体地使用状态码，你的消费者在理解和从他们自己那一侧的问题中恢复时需要付出的努力就越少。

使用尽可能具体的状态码也给我们提供了一个风险免费的方式来传达足够关于出了什么问题的信息，同时不传达任何可能使我们的软件处于风险中的信息。如果你用一个 401 状态码（表示“未授权”）响应一个错误的认证请求，用户就会知道他们需要调整他们的认证机制。然而，如果你只是返回一个通用的 400 状态码，以及一个表示密码最小字符要求为八个的消息，那么你刚刚给了潜在的恶意行为者比你之前尝试失败之前更多的关于你的认证方案的信息。而且，对于恶意软件来说，关于你系统具体信息的任何信息都是危险的。

理解多少信息是足够的，与过多信息，可能是一种微妙的平衡行为。了解其他人将从你的软件中看到什么，很大程度上来自于经验。你看到的越多外部服务发送无用的“出了问题。哎呀！”错误消息，你就越会知道你想要知道什么，以及你如何在未来的代码中做得更好。然而，当你刚开始时，一个很好的经验法则是状态码应该尽可能具体，错误消息应该尽可能模糊。在 HTTP 中，错误状态码都集中在 4XX 和 5XX 值上。这两个分组分别指定请求服务错误和服务器错误。

请求服务错误与导致失败发生的请求的结构、来源或性质有关。因此，从请求一个在请求位置实际上不存在的资源（404 - 未找到），到请求一个由监听服务器不支持 HTTP 动词的资源（405 - 方法不允许），再到请求客户端未授权访问的资源（401 - 未授权）。

相反，服务器错误与在接收到正确且格式良好的地址之后发生的问题有关。这些问题的数量要少得多，因为大多数格式良好的请求被认为是格式良好的，特别是因为已经配置了服务器来处理它们。5XX 响应的原因范围从上游服务器无法处理客户端请求的某些方面（502 - 网关错误/504 - 网关超时），到目标服务器在请求时根本无法使用或不可用（503 - 服务不可用）。

如果你发送的是正确的错误代码，那么你几乎不可能在自己的代码中明确返回 5XX 错误代码。如果你的软件编写得很好，导致它抛出错误的问题几乎总是可以追溯到某个入站请求的某个方面。然而，当这种情况发生时，你有绝对的义务尽你所能找出是什么具体关于请求导致了错误，并迅速报告。

# 错误处理策略

现在我们已经看到了在遇到来自网络资源的错误消息（以及发送自己的错误消息）时你可以使用的工具，让我们来看看我们应该如何处理它们。

我们应该如何最好地响应`RequestCancelled`异常状态？哪些故障状态可能有一个共同的根源，因此有一个共同的共享解决方案？当我们无法从更上游的错误中恢复时，我们的软件应该如何对我们的用户做出响应？在本节中，我们将探讨这些问题中的每一个，并留下一些可以适应和扩展到几乎所有情况的具体方法。

# 使用 Polly 实现弹性请求

正如我们在之前的代码示例中已经看到的，用相同的一般恢复解决方案来响应多个类似错误状态并不罕见。这是一种简化代码库的绝佳实践，并且可以在各种常见情况下提供持久的异常处理。

将常见的网络问题分组，以便可以使用类似策略解决的行为，正是 Polly 库为弹性 HTTP 客户端背后的理念。虽然我们现在不是专门研究 HTTP，但它是最健壮的库之一，用于处理最常见的网络协议之一，因此我认为在我们继续研究错误恢复策略时，它值得检查。

首要任务是将在我们的项目中包含该包，无论是通过在 NuGet 包管理器中明确包含，还是使用以下命令行输入：

```cs
dotnet add package Polly
```

一旦安装完成，我们可以声明一个`Policy`类，用于使用 Polly 的声明性处理程序和恢复方法来处理各种网络异常。`Policy`类是一个健壮的、短暂的容器，用于特定的任务。我们定义一个委托，然后将该委托提供给`Policy`以执行。然后`Policy`类使用活动定义的错误处理程序及其恢复定义来执行任务，并相应地监听和响应异常状态。

我们希望 Polly 响应的异常使用通用的`Handle<T>()`方法设置，其中`T`是`Exception`类型的某个子类。`Handle<T>()`方法还接受可选的条件参数，指定我们想要使用相应的恢复规范响应的`Exception`类型的状态。这使我们能够为不同的状态定义特定的恢复策略。让我们看看这个动作，以了解我的意思。

首先，我们将定义一个用于请求某些远程资源的方法。这只是为了演示目的，所以我们希望它偶尔失败，偶尔成功。为此，我们只需生成一个随机数，如果数字是偶数，我们将抛出一个异常；否则，我们将返回一个有效的响应。然而，重要的是，我们希望在屏幕上记录我们的失败尝试，这样我们就可以看到重试的效果：

```cs
public static HttpResponseMessage ExecuteRemoteLookup() {
    if (new Random().Next() % 2 == 0) {
        Console.WriteLine("Retrying connections...");
        throw new WebException("Connection Failure", WebExceptionStatus.ConnectFailure);
    }
    return new HttpResponseMessage();
}
```

这将是我们在定义了恢复策略并想要尝试执行它之后传递给我们的`Policy`对象的委托。接下来，我们将定义我们之前在简单的错误处理代码中定义的几个错误状态的行为。为了提高可读性，我们将定义一些私有类变量来保存我们逻辑上合并的`WebExceptionStatus`值组：

```cs
private List<WebExceptionStatus> connectionFailure = new List<WebExceptionStatus>() {
 WebExceptionStatus.ConnectFailure,
 WebExceptionStatus.ConnectionClosed,
 WebExceptionStatus.RequestCanceled,
 WebExceptionStatus.PipelineFailure,
 WebExceptionStatus.SendFailure,
 WebExceptionStatus.KeepAliveFailure,
 WebExceptionStatus.Timeout
};

private List<WebExceptionStatus> resourceAccessFailure = new List<WebExceptionStatus>() {
 WebExceptionStatus.NameResolutionFailure,
 WebExceptionStatus.ProxyNameResolutionFailure,
 WebExceptionStatus.ServerProtocolViolation
};

private List<WebExceptionStatus> securityFailure = new List<WebExceptionStatus>() {
 WebExceptionStatus.SecureChannelFailure,
 WebExceptionStatus.TrustFailure
};
```

这样，我们可以轻松地定义一个`Policy`对象，该对象对具有我们分组中状态之一的`WebException`做出响应，如下所示：

```cs
public static void ExecuteRemoteLookupWithPolly() {
    Policy connFailurePolicy = Policy
        .Handle<WebException>(x => connectionFailure.Contains(x.Status))
        .RetryForever();

    HttpResponseMessage resp = connFailurePolicy.Execute(() => ExecuteRemoteLookup());
    if (resp.IsSuccessStatusCode) {
        Console.WriteLine("Success!");
    }
}
```

注意，当我们执行`Policy`时，我们指定只要我们得到在`connectionFailure`分组中定义的`WebExceptionStatus`，我们希望重试请求。所以，现在让我们从这个驱动程序程序中多次调用它，看看每次运行后我们的控制台看起来像什么。假设由于伪随机数生成器的足够随机性，应该至少有几次运行在返回有效响应之前多次失败。（注意，为了演示的目的，所有的 Polly 代码都存在于一个`static PollyDemo`类中）。让我们看一下以下代码：

```cs
using System.Threading;

namespace ErrorHandling {
    public class Program {
        static void Main(string[] args) {
            PollyDemo.ExecuteRemoteLookupWithPolly();
            Thread.Sleep(10000);
        }
    }
}
```

如果你已经将你的 IDE 配置为在错误时中断，那么每次你的代码失败时，你都会在执行中暂停。然而，我自己运行这段代码时，我看到了立即的成功，然后是五次连续的重试，我的代码才成功执行。我能够在不到 10 行代码中定义这一点是令人难以置信的，这也说明了 Polly 在为应用程序提供弹性方面的价值。

然而，如果我们想要真正地反映我们在之前简单错误处理代码中建立的行为，我们希望根据达到的状态对各种异常状态响应以特定的恢复代码。为此，Polly 允许你定义多个状态处理器，然后将它们包装在`PolicyWrap`类中，这正是它所做的事情。它将允许你为所需的任何条件状态定义恢复策略，然后将它们包装在一个单一的共同策略中，当调用`PolicyWrap`实例的`Execute(delegate)`方法时，将遵守这个策略。

为了演示这一点，我们将为我们的代理定义几个额外的异常状态，这样如果生成的随机数能被`3`整除，我们将抛出一个名称解析错误；如果数字能被`4`整除，我们将抛出一个安全错误：

```cs
public static HttpResponseMessage ExecuteRemoteLookup() {
    var num = new Random().Next();
    if (num % 3 == 0) {
        Console.WriteLine("Breaking the circuit");
        throw new WebException("Name Resolution Failure", WebExceptionStatus.NameResolutionFailure);
    } else if (num % 4 == 0) {
        Console.WriteLine("Falling Back");
        throw new WebException("Security Failure", WebExceptionStatus.TrustFailure);
    } else if (num % 2 == 0) {
        Console.WriteLine("Retrying connections...");
        throw new WebException("Connection Failure", WebExceptionStatus.ConnectFailure);
    }
    return new HttpResponseMessage();
}
```

既然我们现在有随机机会满足我们的至少一个异常条件，让我们定义每种情况下的行为。正如你可能从我的控制台消息中注意到的，我们将为每个特定的错误情况使用不同的策略。Polly 默认定义了一些策略，你可以在最需要的情况下调用每个策略。现在我不会详细介绍所有这些策略，但我会花点时间鼓励你阅读 Polly 的文档([`github.com/App-vNext/Polly`](https://github.com/App-vNext/Polly))。它比这一整章都要长，但写得很好，对于任何希望为生产应用程序提供更可靠稳定性的开发者来说，它都是极其有用的。不过，现在我们只关注`Circuit-breaker`策略和`Fallback`策略。这两个策略似乎对我们用例最有用，因为它们与我们先前简单方法中确定的战略最为接近。

`Fallback`策略是两个中较简单的一个。它仅仅允许你在处理指定的异常时返回一个替代响应。在我们的例子中，由于我们将使用`Fallback`来处理我们的安全异常，我们将简单地返回一个新的`HttpResponseMessage`实例，并将状态码设置为 401，以通知我们的下游消费者存在需要解决的问题，即授权问题。

断路器策略指定，在多次尝试解决请求失败后，应该打开电路到请求的资源，并在请求开始之前停止后续请求。这在像我们为名称解析失败定义的场景中很有用，在这种情况下，基于错误消息，后续尝试成功的可能性并不比原始请求更高。打开电路（从而停止通过该电路的请求流）给上游系统一个机会，在没有被一系列重试尝试轰炸的情况下恢复。你可以配置电路在指定数量的失败尝试或指定超时后打开，并且你可以设置它保持打开状态，直到你认为这可能是必要的，以便允许上游系统恢复。

与重试策略及其变体不同，然而，断路器在响应抛出的错误时实际上并不做任何事情。事实上，它总是会重新抛出任何捕获到的错误；即使电路已经断开。如果你想在开放电路指定的重置周期后重试请求，你可以自由地实现这种行为，但默认情况下，Polly 的`spec`不会在其断路器实现中这样做。因此，在我们的示例中，我们将在只有一次失败尝试后断开电路，我们仍然需要在调用代码的`try`/`catch`中查找适当的错误消息。

考虑到这一点，让我们更新我们之前的示例。我们首先要做的是为我们的`Fallback`策略添加一个方法，返回`HttpResponseMessage`中的 401 状态码：

```cs
private static HttpResponseMessage GetAuthorizationErrorResponse() {
    return new HttpResponseMessage(HttpStatusCode.Unauthorized);
}
```

然后我们将为我们的两种替代错误状态设置策略，并相应地包装它们：

```cs
public static void ExecuteRemoteLookupWithPolly() {
    Policy connFailurePolicy = Policy
        .Handle<WebException>(x => connectionFailure.Contains(x.Status))
        .RetryForever();

    Policy<HttpResponseMessage> authFailurePolicy = Policy<HttpResponseMessage>
        .Handle<WebException>(x => securityFailure.Contains(x.Status))
        .Fallback(() => GetAuthorizationErrorResponse());

    Policy nameResolutionPolicy = Policy
        .Handle<WebException>(x => resourceAccessFailure.Contains(x.Status))
        .CircuitBreaker(1, TimeSpan.FromMinutes(2));

    Policy intermediatePolicy = Policy
        .Wrap(connFailurePolicy, nameResolutionPolicy);

    Policy<HttpResponseMessage> combinedPolicies = intermediatePolicy
        .Wrap(authFailurePolicy);

    try {
        HttpResponseMessage resp = combinedPolicies.Execute(() => ExecuteRemoteLookup());
        if (resp.IsSuccessStatusCode) {
            Console.WriteLine("Success!");
        } else if (resp.StatusCode.Equals(HttpStatusCode.Unauthorized)) {
            Console.WriteLine("We have fallen back!");
        }
    } catch (WebException ex) {
        if (resourceAccessFailure.Contains(ex.Status)) {
            Console.WriteLine("We should expect to see a broken circuit.");
        }
    }
}
```

因此，在我们的改进方法中，我们为每个我们想要响应的可能场景定义一个策略，包括应该处理的特定异常状态和我们要实施的恢复过程。值得注意的是，我们调用了`Policy.Wrap()`方法两次。这样做的原因是，在`Policy<HttpResponseMessage>`的强类型实例上使用`Fallback()`方法是唯一能够指定我们传递给`Fallback()`的委托方法返回对象类型的方式。然而，通过强类型化策略，我们无法在单个调用中将它`Wrap()`到其他弱类型策略中。强类型策略的`Wrap()`方法最多只能接受一个参数。因此，这个问题的解决方案是首先将我们定义的所有弱类型策略包装起来，然后使用那个包装好的`Policy`实例作为输入到我们强类型`Policy`的`Wrap()`调用。我明白这最初可能会让人困惑，但随着你使用 Polly、阅读他们出色的文档以及最重要的是，在编写的任何网络软件中实施这些错误处理策略，这将会变得更加清晰。

为了使我们的驱动程序程序在测试目的上使用起来更简单（而不是手动运行程序二十多次来查看所有可能的结果），我们也将更新它。让我们看一下以下代码：

```cs
public static void Main(string[] args) {
    for (var i = 0; i < 24; i++) {
        Console.WriteLine($"Polly Demo Attempt {i}");
        Console.WriteLine("-------------");
        PollyDemo.ExecuteRemoteLookupWithPolly();
        Console.WriteLine("-------------");
        Thread.Sleep(5000);
    }
}
```

运行该程序一次（也许两次，因为毕竟随机性是随机的）你应该会看到我们每个可能场景的适当日志语句。我指望你能理解我们程序中控制流的本质，以了解为什么屏幕上的结果展示了我们定义的`Policy`对象所承诺的功能。

随着我们继续前进并查看不同网络协议的具体实现，我们将大量依赖 Polly 来定义我们的恢复策略。这个库有很多深度，你可以从中得到你选择投入学习的任何东西。然而，有了这个基础，你将准备好继续阅读本书的其余部分。

# 摘要

在本章中，我们详细探讨了.NET Core `WebException`类如何为工程师提供一个稳定、可靠且信息丰富的接口，以便在出现网络错误时理解它们。我们讨论了何时以及如何预期和解释网络异常，以及如何检查这些异常的`Status`属性以确定其根本原因。我们还考虑了提供有意义的异常消息的责任，以及为我们的软件的任何消费者提供尽可能具体的状态代码的价值。最后，我们探讨了常见的策略，以及一个极其有用的库 Polly，用于持续地从网络异常中恢复，以最大化我们的应用程序的运行时间和增加消费者对我们软件的信任。在未来的工作中，保持这些弹性和优化的想法将非常重要。

在下一章中，我们将进入低级数据传输和主机间通信的世界。

# 问题

1.  我们应该始终假设关于外部依赖性的一件事是什么？

1.  HTTP 中的错误状态代码分为哪两类？

1.  我们可以使用`WebException`类的哪个属性来确定我们收到的异常的性质？

1.  Polly 的回退策略为异常状态提供了什么？

1.  电路断路器策略规范与`retry`策略规范有何不同？

1.  编写一个示例方法，该方法基于从示例请求的响应中的`StatusCode`结合`fallback`和`retry`策略。

1.  在哪些情况下，你不想在初始失败后重试网络请求？

# 进一步阅读

如需了解有关各种网络软件架构的具体错误处理策略的更多信息，请参阅*Jason De Oliveira*和*Michel Bruche**t*合著的《*学习 ASP.NET Core 2.0*》。这本书将为基于 ASP.NET Core 的 HTTP 应用场景中的错误处理提供更深入的指导。本书电子版或印刷版均可购买，链接如下：[`www.packtpub.com/application-development/learning-aspnet-core-20.`](https://www.packtpub.com/application-development/learning-aspnet-core-20)

或者，我再次推荐由*Mark J. Price*和*Ovais Mehboob Ahmed Khan*合著的《*C# 7 和.NET：设计现代跨平台应用*》。关于异常处理的内容对许多常见用例来说很有用，并且聚焦良好。再次提醒，购买电子书或印刷版的链接在这里：[`www.packtpub.com/application-development/learning-path-c-7-and-net-designing-modern-cross-platform-applications.`](https://www.packtpub.com/application-development/learning-path-c-7-and-net-designing-modern-cross-platform-applications)
