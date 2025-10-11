# 第八章：8. 创建和使用 Web API 客户端

概述

在本章中，你将通过调用 Web API 来步入 HTTP 实践的世界。你将使用网络浏览器、你自己的 HTTP 客户端和 NuGet 包以各种方式与 Web API 交互。你将学习 Web API 中涉及的安全基础，使用 PayPal 进行沙盒支付，并探索 Azure 文本分析服务和 Azure Blob 存储等云服务。

到本章结束时，你将能够阅读 HTTP 请求和响应消息，调用任何 Web API，并创建自己的 HTTP 客户端以简化与复杂 API 的工作。你还将能够分析并学习任何形式的传入 HTTP 请求和传出 HTTP 响应，并使用 Chrome 浏览器中的开发工具检查浏览你喜欢的网站时来回移动的流量。

# 简介

万维网（WWW）（或简称网络）是一个包含各种文档（XML、JSON、HTML、MP3、JPG 等）的大仓库，这些文档可以通过统一资源定位符（URL）访问。在网络的上下文中，一个文档通常被称为资源。有些资源不会改变。它们存储在某处，并且每次请求时，都会返回相同的资源。这类资源被称为静态资源。其他资源是动态的，这意味着它们将在需要时生成。

网络上的通信通过协议进行。在检索文档的上下文中，你使用超文本传输协议（HTTP）。超文本是一种特殊的文本，它包含指向网络中资源的链接。点击它将打开它指向的资源。HTTP 基于客户端-服务器架构。简单来说，客户端发送请求，服务器响应。实践中一个例子是浏览器（客户端）和网站（托管在服务器上）之间的通信。通常，单个服务器为许多客户端提供服务：

![图 8.1：客户端-服务器架构](img/B16835_08_01.jpg)

图 8.1：客户端-服务器架构

当你导航到网站时，你发送一个 `HTTP GET` 请求，服务器通过在浏览器中显示相关网站内容来响应。`GET` 是一个 HTTP 动词——一个标识请求应该如何被处理的方法。常见的 HTTP 动词如下：

+   `GET`：获取一个资源。

+   `POST`：创建一个资源或发送一个复杂的查询。

+   `PUT`：更新所有资源字段。

+   `PATCH`：更新单个字段。

+   `DELETE`：删除一个资源。

# 浏览器

现代浏览器不仅仅是一个访问互联网内容的工具。它包括分解网站元素、检查流量甚至执行代码的工具。浏览器的这一部分被称为开发者工具。确切的快捷键可能有所不同，但按 `F12` 或 `Control + Shift + I` 应该会调出开发者工具标签。执行以下步骤以更好地了解它：

1.  打开 Google Chrome 或任何其他浏览器。

1.  导航到 [google.com](http://google.com)。按 `Control + Shift + I` 键。

1.  转到 `网络` (`1`)。以下窗口应该会显示：

![图 8.2：Chrome 浏览器中已打开开发者工具并加载了 google.com](img/B16835_08_02.jpg)

图 8.2：Chrome 浏览器中已打开开发者工具并加载了 google.com

1.  选择第一个条目，[www.google.com](http://www.google.com) (`2`)。

1.  点击`头部`(`3`)。

1.  在`通用`(`4`)部分，你可以观察到导航到[google.com](http://google.com)时的效果。首先发生的事情是发送了`HTTP GET`请求到[`www.google.com/`](https://www.google.com/)。

1.  在`请求头部`部分(`5`)，你可以看到随请求发送的元数据。

1.  要查看 Google 的响应，请点击`响应`部分(`6`)。

这种流程被称为客户端-服务器架构，以下适用：

+   客户端是发送请求到[google.com](http://google.com)的 Chrome 浏览器。

+   服务器是一台（或多台）托管[google.com](http://google.com)的机器，它以[google.com](http://google.com)网站内容作为响应。

# Web API

应用程序编程接口（API）是通过代码调用某些功能的一个接口。它可以是 C#中的类或接口，或者是一个浏览器（你可以通过其提供的接口与之交互），但在 HTTP 的上下文中，它是一个网络服务。网络服务是托管在远程机器上的 API，可以通过 HTTP 访问。用于在 Web API 上调用单个功能点的访问点称为端点。最常用的 Web API 类型是 RESTful。

## RESTful API

表示性状态转移（REST）API 是基于以下六个原则构建的 API。其中四个原则是无论你使用什么框架实现 RESTful API，作为客户端都应该预期的：

+   客户端-服务器：在客户端和服务器之间建立连接。客户端发送请求以从服务器获取响应。

+   无状态：服务器将能够处理请求，而不管之前的请求。这意味着每个请求都应该包含所有信息，而不是依赖于服务器记住之前发生的事情。

+   可缓存：使用 HTTP 方法或头部指定哪些请求可以被缓存的特性。

+   按需代码（可选）：REST 允许脚本在客户端下载并执行。在互联网主要由静态页面组成的时候，这很有用，但现在要么不再需要，要么被视为安全风险。

然而，其他两个原则（客户端-服务器和无状态）取决于你，因此你将需要更加关注它们。分层系统是由层组成的系统，每一层只与直接下方的层进行通信。一个典型的例子是三层架构，其中你将表示层、业务逻辑和数据存储分开。从实际的角度来看，这意味着 RESTful API（业务逻辑层）不应发送 HTML 作为响应，因为渲染输出的责任在于客户端（表示层）。

最后一个原则被称为统一接口。它定义了 API 的一组规则：

+   资源识别：

这些示例包括获取资源的所有实例 (`/resource`)、创建资源 (`/resource`)、获取单个资源 (`/resource/id`) 以及获取资源中所有子资源的实例 (`/resource/subresource/`)。

+   通过这些表示来操作资源：

使用表示创建、读取、更新和删除（CRUD）的 HTTP 动词来操作资源——`GET`、`UPDATE`、`PUT`、`PATCH`、`DELETE`。

+   自描述消息：

包含所有必要信息且没有额外文档的响应，并指示如何处理消息（头、MIME 类型等）。

+   超媒体作为应用程序状态引擎（HATEOAS）：

在响应中包含超链接，以便您可以导航到相关资源。此指南通常被忽略。

REST 与 HTTP 不同。REST 是一组指南，而 HTTP 是一种协议。两者可能会混淆，因为 HTTP 约束与 REST 约束（方法、头等）高度重叠。然而，RESTful API 不必使用 HTTP 才是 RESTful，同时 HTTP 可以通过使用会话或查询参数来执行操作而违反 REST 约束。RESTful API 可以与 XML 和 JSON 数据格式一起工作。然而，几乎所有场景都涉及 JSON。

## Postman

Postman 是用于测试不同类型 Web API 的最受欢迎的工具之一。它易于设置和使用。Postman 就像浏览器一样，充当 HTTP 客户端。为了下载 Postman，请访问 [`www.postman.com/`](https://www.postman.com/)。你需要注册并下载安装程序。安装 Postman 后，执行以下步骤：

1.  打开 Postman。

1.  通过点击 `Workspaces` 创建你的工作空间，然后点击 `Create Workspace`。

1.  在新窗口中，转到 `Collections` 选项卡 (`2`)，然后点击 `Create new Collection` (`+`) 按钮 (`3`)。

1.  创建 `New Collection` (`4`)。

1.  点击 `Add a request` (`5`)：

![图 8.3：没有请求的新 Postman 收藏夹](img/B16835_08_03.jpg)

图 8.3：没有请求的新 Postman 收藏夹

将会打开一个新的请求窗口。

1.  点击 `New Request` 旁边的编辑符号，并将新请求命名为 `Users` (`6`)。

1.  选择 `GET` HTTP 动词，并将 URL [`api.github.com/users/github-user`](https://api.github.com/users/github-user) (`7`) 复制粘贴。

    注意

    在此处以及随后的所有地方，将 `github-user` 替换为您的 GitHub 用户名。

1.  点击 `Send` 按钮 (`8`)。

1.  现在向下滚动查看返回的响应结果 (`9`)：

![图 8.4：Postman 中的 GET GitHub 用户请求](img/B16835_08_04.jpg)

图 8.4：Postman 中的 GET GitHub 用户请求

Postman 在充当 HTTP 客户端方面优于浏览器。它专注于构建 HTTP 请求，并以紧凑的方式显示响应信息，提供多种输出格式。在 Postman 中，您可以使用多个环境，为请求设置预条件和后条件，自动调用等，但这些高级功能超出了本章的范围。目前，只需知道 Postman 是手动测试 Web API 的首选工具即可。

## 客户端

REST 需要在客户端和服务器之间进行通信。在先前的示例中，客户端角色由浏览器或 Postman 扮演。然而，浏览器或 Postman 都不能替代您代码中的客户端。相反，您需要使用 C# 创建 HTTP 请求。

流行的 Web API 通常为您创建了客户端（在大多数常用语言中也是如此）。Web API 客户端的目的在于简化与底层 API 的交互。例如，您不需要在不支持 `DELETE` 请求的端点上发送请求并得到 `Method Not Allowed` 的响应，在自定义客户端上甚至不会有这样的选项。

## Octokit

Octokit 是一个 GitHub API 客户端。它通过一个 C# 类暴露接口，您可以通过它传递对象来调用 GitHub。此类客户端的好处是您无需担心传递哪些头信息或如何命名以便它们被正确序列化。API 客户端为您处理所有这些。

您可以通过在 VS Code 终端或命令提示符中运行以下命令来在项目中安装 Octokit 客户端：

```cs
dotnet add package Octokit
```

一旦安装了 Octokit 客户端，您就可以使用它来创建 GitHub 客户端，如下所示：

```cs
var github = new GitHubClient(new ProductHeaderValue("Packt"));
```

在前面的代码片段中，您需要一个新的 `ProductHeaderValue`，因为 GitHub 期望一个 `UserAgent` 头信息。如前所述，自定义 HTTP 客户端可以防止在您甚至可以发出请求之前发生错误。在这种情况下，不提供 `UserAgent` 头信息（通过 `ProductHeaderValue`）不是一个选项。

要检查客户端是否工作，尝试获取用户名 `github-user` 的信息：

```cs
const string username = "github-user";
var user = await github.User.Get(username);
```

注意

在 GitHub 上，`github-user` 显示为 `Almantask`。最好将其更改为您的个人 GitHub 用户名，以便代码能够正常工作。

要打印用户创建的日期，请输入以下代码：

```cs
Console.WriteLine($"{username} created profile at {user.CreatedAt}");
```

您将看到以下输出：

```cs
github-user created profile at 2018-06-22 07:51:56 +00:00
```

GitHub API 上可用的每种方法在 **GitHub 客户端 Octokit** 上也同样可用。您无需担心端点、必填头信息、响应或请求格式；这一切都由强类型客户端定义。

注意

您可以在 [`packt.link/DK2n2`](https://packt.link/DK2n2) 找到此示例使用的代码。

## API 密钥

在许多公共免费 API 中，您可能会遇到以下担忧：

+   如何控制大量的请求？

+   应在何时对哪个客户端收费？

如果所有这些公共 API 只提供匿名访问，您将无法识别客户端或确定每个客户端各自调用了多少次。API 密钥是基本认证（识别客户端）和授权（授予他们使用 API 进行某些操作的权限）的最基本手段。简单来说，API 密钥允许您调用 API。没有它，您将几乎没有访问 API 的权限。

为了帮助您更好地掌握 API 密钥的使用，下一节将探讨需要 API 密钥的 Web API，即 Azure 文本分析。

## Azure 文本分析

Azure 文本分析是 Azure API，用于以下方式分析文本：

+   识别命名实体（人物、事件、组织）

+   解释文本的情绪（积极、消极、中性）

+   撰写文档的摘要或突出显示关键短语

+   处理非结构化医疗数据，例如识别人员、分类诊断等

为了演示 Azure 文本分析 API，您将专注于情感分析。这是根据积极、消极或中立的置信度分数评估文本的过程：

+   得分为 1，即 100%，表示预测（消极、积极、中性）的正确概率。

+   得分为 0，即 0%，表示不可能的预测。

    注意

    使用 Azure 文本分析免费，直到您在 30 天内分析超过 5,000 个单词。

在开始编码之前，您需要在 Azure 云上设置 Azure 文本分析。毕竟，您需要一个端点和 API 密钥来调用此 API。

注意

确保您已设置 Azure 订阅。如果您没有，请访问[`azure.microsoft.com/en-gb/free/search`](https://azure.microsoft.com/en-gb/free/search)，并按照那里的说明创建一个**免费**订阅。Azure 免费试用提供许多免费服务。其中一些服务在一年后仍将免费。学生订阅是获取 Azure 信用额和更长时间免费服务的选项。创建 Azure 订阅需要信用卡或借记卡；然而，除非您超出免费服务的限制，否则您不会收费。

Azure 文本分析可以用来对积极和消极反馈进行排序的一种方式是确定您所写的内容听起来是被动攻击性的还是友好的。要查看此功能如何运作，请按照以下步骤创建一个小型应用程序，该应用程序可以分析您输入控制台中的任何文本：

1.  首先，访问[`portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics`](https://portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics)。

1.  点击“继续创建资源”而不使用任何附加功能：

![图 8.5：Azure 文本分析资源创建](img/B16835_08_05.jpg)

图 8.5：Azure 文本分析资源创建

1.  在创建文本分析窗口中，点击“基础”选项卡。这是创建新资源时首次打开的第一个选项卡。

1.  在`订阅`和`资源组`字段中选择一个选项：

![图 8.6：输入新资源创建的项目详情](img/B16835_08_06.jpg)

图 8.6：输入新资源创建的项目详情

1.  然后，选择区域，例如，`北欧`。

1.  输入名称，例如，`Packt-Test`。

1.  之后，选择`免费 F0`定价层并点击`审查 + 创建`按钮：

![图 8.7：Azure 文本分析定价层](img/B16835_08_07.jpg)

图 8.7：Azure 文本分析定价层

新窗口将显示以确认您的输入。

1.  点击`创建`选项。文本分析 API 将开始部署。服务部署完成后，将打开一个新窗口，显示`您的部署已完成`。

1.  点击`转到资源`按钮：

![图 8.8：显示部署完成的文本分析 API](img/B16835_08_08.jpg)

图 8.8：显示部署完成的文本分析 API

显示文本分析资源窗口。

1.  点击`密钥和端点`选项。您将看到`端点`选项以及`KEY 1`和`KEY 2`以调用此 API。您可以从任一密钥中选择：

![图 8.9：带有 API 密钥超链接的 Azure 文本分析快速入门窗口](img/B16835_08_09.jpg)

图 8.9：带有 API 密钥超链接的 Azure 文本分析快速入门窗口

1.  跟踪`KEY 1`（一个 API 密钥）。API 密钥是秘密的，不应以纯文本形式公开。您将再次使用环境变量来存储它。

创建一个键值对环境变量。值将是连接到 Azure 文本分析所需的端点 API 密钥。为了帮助识别缺失的环境变量，请使用辅助类。`GetOrThrow`方法将获取用户环境变量，如果不存在，将抛出异常：

```cs
    public static class EnvironmentVariable
    {
        public static string GetOrThrow(string environmentVariable)
        {
            var variable = Environment.GetEnvironmentVariable(environmentVariable, EnvironmentVariableTarget.User);
            if (string.IsNullOrWhiteSpace(variable))
            {
                throw new ArgumentException($"Environment variable {environmentVariable} not found.");
            }
            return variable;
        }
    }
```

1.  跟踪`端点`选项。您将在接下来的练习中使用它来调用您刚刚部署的 API。

本节帮助您在 Azure 云上设置 Azure 文本分析，同时设置端点和 API 密钥以调用 API。在接下来的练习中，您将使用 Azure 文本分析客户端调用 API。

## 练习 8.01：对任何文本执行情感分析

Azure 文本分析只是另一个 REST API。再次，您向它发送 HTTP 请求并获取响应。这次，您将发送一段文本以获取其情感分析。再次练习使用强类型客户端从 C#调用 RESTful API。

使用最近部署的 Azure 文本分析服务（在本例中为`Pack-Test`），对任何您想要的文本执行情感分析。按照以下步骤完成此练习：

1.  按照以下步骤安装`Azure.AI.TextAnalytics` NuGet 包以获取 Azure 文本分析 API 客户端：

    ```cs
    dotnet add package Azure.AI.TextAnalytics
    ```

1.  添加`TextAnalysisApiKey`环境变量。

1.  然后添加`TextAnalysisEndpoint`环境变量。

1.  创建一个 `Demo` 类，并添加对最近添加的两个环境变量的引用：

    ```cs
    public class Demo
    {
        private static string TextAnalysisApiKey { get; } = EnvironmentVariable.GetOrThrow("TextAnalysisApiKey");
        private static string TextAnalysisEndpoint { get; } = EnvironmentVariable.GetOrThrow("TextAnalysisEndpoint");
    ```

这些属性用于隐藏 API 密钥和端点的敏感值。

1.  创建一个新的 `BuildClient` 方法来构建 API 客户端：

    ```cs
    static TextAnalyticsClient BuildClient()
    {
        var credentials = new AzureKeyCredential(TextAnalysisApiKey);
        var endpoint = new Uri(TextAnalysisEndpoint);
        var client = new TextAnalyticsClient(endpoint, credentials);
        return client;
    }
    ```

API 客户端需要两个操作参数：一个基础 URL——一种统一资源标识符 (URI)——和一个 API 密钥，这两个参数在初始化时传递给它。

1.  使用客户端，创建 `PerformSentimentalAnalysis` 方法来分析文本：

    ```cs
    private static async Task<DocumentSentiment> PerformSentimentalAnalysis(TextAnalyticsClient client, string text)
    {
        var options = new AnalyzeSentimentOptions { IncludeOpinionMining = true };
        DocumentSentiment documentSentiment = await client.AnalyzeSentimentAsync(text, options: options);
        return documentSentiment;
    }
    ```

在这里，你正在使用配置对象 `AnalyzeSentimentOptions` 提取目标和对其的意见。客户端既有 `AnalyzeSentimentAsync` 方法，也有 `AnalyzeSentiment` 方法。对于公共客户端库，公开相同方法的异步和非异步版本是一个非常常见的场景。毕竟，并不是每个人都会对异步 API 感到舒适。然而，当调用另一台机器（数据库、API 等）时，最好使用异步 API。这是因为异步调用在等待 API 响应时不会阻塞调用线程。

1.  现在创建一个 `DisplaySentenceSymmary` 函数来显示句子的整体评估：

    ```cs
    private static void DisplaySentenceSummary(SentenceSentiment sentence)
    {
        Console.WriteLine($"Text: \"{sentence.Text}\"");
        Console.WriteLine($"Sentence sentiment: {sentence.Sentiment}");
        Console.WriteLine($"Positive score: {sentence.ConfidenceScores.Positive:0.00}");
        Console.WriteLine($"Negative score: {sentence.ConfidenceScores.Negative:0.00}");
        Console.WriteLine($"Neutral score: {sentence.ConfidenceScores.Neutral:0.00}{Environment.NewLine}");
    }
    ```

1.  创建一个 `DisplaySentenceOpinions` 函数，用于在句子中的每个目标上显示消息 `Opinions`：

    ```cs
    private static void DisplaySentenceOpinions(SentenceSentiment sentence)
    {
        if (sentence.Opinions.Any())
        {
            Console.WriteLine("Opinions: ");
            foreach (var sentenceOpinion in sentence.Opinions)
            {
                Console.Write($"{sentenceOpinion.Target.Text}");
                var assessments = sentenceOpinion
                    .Assessments
                    .Select(a => a.Text);
                Console.WriteLine($" is {string.Join(',', assessments)}");
                Console.WriteLine();
            }
        }
    }
    ```

句子的目标是应用了意见（语法修饰符）的主题。例如，在句子 **a beautiful day** 中，**day** 是目标，**beautiful** 是意见。

1.  要在控制台中输入的文本上执行情感分析，创建 `SentimentAnalysisExample` 方法：

    ```cs
    static async Task SentimentAnalysisExample(TextAnalyticsClient client, string text)
    {
        DocumentSentiment documentSentiment = await PerformSentimentalAnalysis(client, text);
        Console.WriteLine($"Document sentiment: {documentSentiment.Sentiment}\n");
        foreach (var sentence in documentSentiment.Sentences)
        {
            DisplaySentenceSummary(sentence);
            DisplaySentenceOpinions(sentence);
        }
    }
    ```

在前面的代码片段中，分析文本评估整体文本的情感，然后将其分解成句子，并对每个句子进行评估。

1.  为了演示你的代码如何工作，创建一个静态的 `Demo.Run` 方法：

    ```cs
    public static Task Run()
    {
        var client = BuildClient();
        string text = "Today is a great day. " +
                           "I had a wonderful dinner with my family!";
        return SentimentAnalysisExample(client, text);
    }
    ```

在设置正确的环境变量后，以下输出应该被显示：

```cs
Document sentiment: Positive
Text: "Today is a great day."
Sentence sentiment: Positive
Positive score: 1,00
Negative score: 0,00
Neutral score: 0,00
Text: "I had a wonderful dinner with my family!"
Sentence sentiment: Positive
Positive score: 1,00
Negative score: 0,00
Neutral score: 0,00
Opinions:
dinner is wonderful
```

你在这里没有硬编码 API 密钥的值，因为公开的 API 密钥可能会被不当使用，如果被盗用，可能会产生灾难性的后果（例如，过度使用、创建虚假资源、数据泄露、删除数据等）。这就是为什么在处理机密信息时，应使用尽可能少的防护措施，即环境变量。

环境变量的另一个好处是能够在不同的环境中具有不同的值（本地、集成、系统测试、生产等）。不同的环境通常使用不同的资源。因此，通过环境变量指向这些资源不需要对代码进行任何更改。

为了运行这个练习，请访问 [`packt.link/GR27A`](https://packt.link/GR27A) 并注释掉 `static void Main(string[] args)` 方法体内的所有行，除了 `await Exercises.Exercise01.Demo.Run();`。同样，在执行每个练习/示例/活动之前，取消注释 `Program.cs` 中的相应练习/示例/活动代码行。

注意

你可以在[`packt.link/y1Bqy`](https://packt.link/y1Bqy)找到用于此练习的代码。

这项练习只是您消费公共 Web API 的许多练习之一。Azure 充满了这样的服务。使用强类型客户端调用 API 很简单；然而，并非所有 API 都有。在下一节中，您将学习如何创建自己的 Web API 客户端。

# 您自己的客户端

到目前为止，您只使用预制的客户端来消费 Web API。然而，对于不太受欢迎的 API，可能没有可用的客户端。在这种情况下，您将不得不自己进行 HTTP 调用。在.NET 中，进行调用的方式已经发生了很大的变化。如果您不想使用任何第三方库，您可以使用`HttpClient`类。

## HttpClient

在本节中，您将重复 GitHub `Users`示例（来自*Postman*部分），但这次使用`HttpClient`。这个流程相当简单，以下示例将为您详细描述：

1.  在`GitHttp`静态类中，创建`GetUser`方法：

    ```cs
    public static async Task GetUser()
    ```

1.  在`GitExamples`方法中，首先创建一个客户端：

    ```cs
    client = new HttpClient { BaseAddress = new Uri("https://api.github.com") };
    client.DefaultRequestHeaders.Add("User-Agent", "Packt");
    ```

创建客户端几乎总是涉及指定特定的基本 URL。通常，Web API 需要传递强制性的头信息，否则它们将使请求无效（`400 Bad Request`）。对于 GitHub，您需要发送标识调用 API 的客户端的`User-Agent`头信息。将`Packt`用户代理头信息添加到默认头信息中，将使该头信息随每个请求发送到客户端。

1.  然后按照以下方式创建一个请求：

    ```cs
    const string username = "github-user"; //replace with your own
    var request = new HttpRequestMessage(HttpMethod.Get, new Uri($"users/{username}", UriKind.Relative));
    ```

记得将`github-user`替换为您自己的 GitHub 用户名。在这里，您指定了您想要创建一个`GET`请求。您没有指定完整的路径，而是只指定了您想要触发的端点；因此，您必须将`UriKind`标记为`Relative`。

1.  接下来，使用客户端发送请求：

    ```cs
    var response = await client.SendAsync(request);
    ```

发送 HTTP 请求消息只有一个异步版本，因此您需要等待它。发送`HttpRequestMessage`的结果是`HttpResponseMessage`。

1.  然后，按照以下方式将内容反序列化为可用的对象：

    ```cs
    var content = await response.Content.ReadAsStringAsync();
    var user = JsonConvert.DeserializeObject<User>(content);
    ```

反序列化是将结构化文本（如 JSON）转换为内存中对象的过程。为此，您需要将内容转换为字符串，然后进行反序列化。您可以使用来自 Octokit NuGet 的用户模型。由于您已经在进行自定义调用，您也可以使用自定义模型。对于最基本的情况（仅使用到的字段），您的模型可能看起来像这样：

```cs
public class User
{
    public string Name { get; set; }
    [JsonProperty("created_at")]
    public DateTime CreatedAt { get; set; }
}
```

在`public DateTime CreatedAt { get; set; }`之上，行`[JsonProperty("created_at")]`将 JSON 字段绑定到 C#属性。这种绑定是必要的，因为名称不匹配。

如果您想创建自己的客户端（用于进行 GitHub 调用），您有责任公开 API 返回的所有数据，而不仅仅是特定场景下可能需要的数据，让消费者进行选择。

1.  使用 Postman 之前的调用消息来获取 GitHub 用户响应体以生成要反序列化的模型。在这种情况下，响应消息如下（消息为了清晰而截断）：

    ```cs
    {
       "login":"github-user",
       "id":40486932,
       "node_id":"MDQ6VXNlcjQwNDg2OTMy",
       "name":"Kaisinel",
       "created_at":"2018-06-22T07:51:56Z",
       "updated_at":"2021-08-12T14:55:29Z"
    }
    ```

有许多工具可以将 JSON 转换为 C#模型。

1.  在这种情况下，使用[`json2csharp.com/`](https://json2csharp.com/)将 JSON 转换为 C#模型代码。

1.  复制响应（`GET github/user`）并转到[`json2csharp.com/`](https://json2csharp.com/)。

1.  将响应粘贴到左侧的文本框中，然后点击`Convert`按钮：

![图 8.10：将 JSON 转换为 C#模型代码](img/B16835_08_10.jpg)

图 8.10：将 JSON 转换为 C#模型代码

左侧显示 JSON 的模型，而右侧显示从 JSON 生成的代码（C#类）。

1.  将右侧的内容复制并粘贴到你的代码中：

    ```cs
    public class Root
    {
        public string login { get; set; }
        public int id { get; set; }
        public string node_id { get; set; }
        public string name { get; set; }
        public DateTime created_at { get; set; }
        public DateTime updated_at { get; set; }
    }
    ```

这就是你的模型。在前面的代码中观察，`Root`是一个不可读的类名。这是因为转换器没有方法知道 JSON 代表什么类。`Root`类代表一个用户；因此，将其重命名为`User`。

最后，转换器可能是在.NET 5 之前创建的，这就是为什么它没有记录功能。记录是一个用于序列化的优秀类，也是数据传输对象（DTO）的优秀候选者。DTO 是一个没有逻辑但仅包含数据，有时还包含用于绑定序列化的属性的类。你获得的好处如下：

+   值相等

+   `ToString`将返回属性及其值

+   能够用更简洁的语法定义它们

因此，尽可能在你的应用程序中使用记录来定义 DTO。

1.  将（`Root`重命名为`User`）并将类型从`class`更改为`record`。代码行看起来如下，不需要对属性进行任何更改：

    ```cs
    public record User
    ```

1.  最后，运行以下代码行：

    ```cs
    Console.WriteLine($"{user.Name} created profile at {user.CreatedAt}");
    ```

输出显示如下：

```cs
Kaisinel created profile at 2018-06-22 07:51:56
```

为了运行此练习，请访问[`packt.link/GR27A`](https://packt.link/GR27A)并注释掉`static void Main(string[] args)`体内的所有行，除了`await Examples.GitHttp.Demo.Run();`。同样，在执行之前，取消注释`Program.cs`中的相应练习/示例/活动的代码行。

注意

你可以在[`packt.link/UPxmW`](https://packt.link/UPxmW)找到用于此示例的代码。

现在你已经看到了使用`HttpClient`类代替第三方库的好处，你可以在下一节探索`IDisposable`模式。

### HttpClient 和 IDisposable

`HttpClient` 实现了 `IDisposable` 模式。一般来说，在你完成使用实现 `IDisposable` 的对象后，你应该清理并调用 `Dispose` 方法，或者将调用封装在一个 `using` 块中。然而，`HttpClient` 是特殊的，你不应该频繁地创建和销毁它。销毁和重新初始化 `HttpClient` 的问题在于 `HttpClient` 管理它与其他 API 建立的连接，销毁 `HttpClient` 并没有正确关闭这些连接（或套接字）。

最危险的部分是，由于可用的连接数量巨大，你在本地开发应用程序时不会注意到任何差异。然而，当将应用程序部署到生产环境时，你可能会耗尽免费的套接字连接。再一次，避免调用 `Dispose` 方法并重新初始化 `HttpClient`。如果你必须这样做，请使用 `HttpClientFactory`。不仅 `HttpClientFactory` 通过管理 `HttpClientMessageHandler`（负责发送 HTTP 请求和接收响应的组件）来管理套接字连接的生存期，它还提供了日志记录功能，允许集中管理客户端的配置，支持向客户端注入中间件等。如果你在企业环境中使用 `HttpClient`，这些提到的优势很重要。你可以在 *第九章*，*创建 API 服务* 中了解更多关于 `HttpClientFactory` 的信息。

理想情况下，你应该有一个静态的 `HttpClient`，你可以在整个应用程序中重复使用它来调用 Web API。然而，你不应该为所有事情使用单个 `HttpClient`。关于不销毁 `HttpClient` 并拥有一个静态的并不是一个硬性规则。如果你调用许多不同的 API，它们将有自己的基本地址、强制头等信息。为所有这些拥有一个单一的对象不是一个可行的场景。

你迄今为止处理过的请求是公开可访问的，并且没有安全性。然而，Web API 中的昂贵或私有操作通常受到保护。通常，保护是通过设置一个授权头来实现的。在许多情况下，授权头涉及某种形式的 ID 和密钥。在 GitHub API 的情况下，它涉及客户端 ID 和客户端密钥。但是，要获取它们，你需要创建一个 OAuth 应用。

在你能够这样做之前，你需要熟悉 OAuth。

## OAuth

OAuth 是一种开放标准的授权协议，允许代表用户委托访问。本节将探讨两个示例：

+   生活中的类比

+   API 类比

### 生活中的类比

想象一个在学校的孩子。那位孩子的老师正在组织去另一个城市的旅行。需要家长提供一份同意书。家长写了一张便条：*我的孩子可以去地方 X，没有问题。* 孩子把便条交给老师，并获得前往目的地 X 的郊游许可。

### API 类比

许多应用程序相互连接，彼此之间有集成。例如，著名的社交平台 Discord 允许您显示您在其他社交媒体上的任何账户。但要做到这一点，您需要连接到您想要显示的社交媒体平台。例如，当您在 Discord 上并尝试链接一个 Twitter 账户时，您将需要在 Twitter 上登录。登录需要一定的访问范围（在这种情况下是您的个人资料名称）。成功的登录是授予访问权限的证明，Discord 将能够代表您在 Twitter 上显示您的个人资料信息。

## GitHub OAuth 应用程序

返回到 GitHub 的主题，OAuth 应用程序是什么？它是一个单一安全点的注册。它充当您的应用程序身份。GitHub 用户可能有零个或多个应用程序。如前所述，OAuth 应用程序包括客户端 ID 和密钥。通过它们，您可以使用 GitHub API。换句话说，您可以设置它来请求访问 GitHub 的安全功能，例如更改您在 GitHub 上的个人数据。

GitHub 有一个有趣的 API 限制。如果来自同一 IP 的未认证请求超过 60 个，它将阻止长达一小时的后续请求。然而，通过授权请求可以移除速率限制。这就是您将使用授权来访问其他情况下公开端点的主要原因。

OAuth 通常涉及两个客户端应用程序：

+   代表某人请求权限的应用

+   另一个授予该权限的应用

因此，在设置 OAuth 时，您很可能会被要求创建一个 URL，在客户端授予访问权限后返回。在 GitHub 上设置 OAuth 应用程序涉及以下步骤：

1.  在右上角，点击您的个人资料图片并点击`设置`：

![图 8.11：GitHub 中的账户设置](img/B16835_08_11.jpg)

图 8.11：GitHub 中的账户设置

1.  在左侧，向下滚动菜单直到接近底部，然后点击`开发者设置`选项：

![图 8.12：GitHub 中的开发者设置](img/B16835_08_12.jpg)

图 8.12：GitHub 中的开发者设置

1.  现在选择`OAuth 应用程序`选项：

![图 8.13：在 GitHub 开发者设置中选择 OAuth 应用程序](img/B16835_08_13.jpg)

图 8.13：在 GitHub 开发者设置中选择 OAuth 应用程序

1.  然后点击`注册新应用程序`按钮：

![图 8.14：在 GitHub 中创建新的 OAuth 应用程序](img/B16835_08_14.jpg)

图 8.14：在 GitHub 中创建新的 OAuth 应用程序

注意

如果您之前已创建 OAuth 应用程序，则此窗口将显示所有列出的应用程序。为了创建一个新的，您将必须点击`新建 OAuth 应用程序`。

1.  在下一个窗口中，您将完成表格。首先填写`应用程序名称`（`5`）。避免使用特殊字符。

1.  接下来，填写`主页 URL`（`6`）。

这个 URL 通常指向一个网站，该网站描述了特定情况下 OAuth 的使用以及为什么需要它。即使你没有描述此类情况的网站，你也可以输入一个占位符 URL（在这个例子中，是`myapp.com`）。只要是一个有效的 URL，这个字段就接受任何内容。

1.  填写`授权回调 URL`（`7`）字段。这可以是任何你想要的内容。这里使用的是`myapp.com/home`。使用一个有效的回调 URL。

1.  点击`注册应用`（`8`）![图 8.15：GitHub 中的新 OAuth 应用窗口](img/B16835_08_15.jpg)

图 8.15：GitHub 中的新 OAuth 应用窗口

1.  在新窗口中，你会看到`Client ID`和`Client secrets`：

![图 8.16：GitHub 上新的 OAuth 应用详情，包括应用凭据—Client ID 和 Client secrets](img/B16835_08_16.jpg)

图 8.16：GitHub 上新的 OAuth 应用详情，包括应用凭据—Client ID 和 Client secrets

最好将客户端密钥存储在一个安全的地方以供将来参考，因为你只会在 GitHub 上看到它一次。如果你忘记了它，你将不得不创建一个新的密钥并删除旧的密钥。

现在你已经在 GitHub 上成功创建了一个 OAuth 应用。客户端密钥在这张截图中被部分隐藏，这是出于一个原因。你永远不应该公开它。为了在演示中使用它，你将首先使用环境变量来隐藏它们。

1.  因此，将这些值存储在环境变量`GithubClientId`和`GithubSecret`中。

1.  然后在`Demo.cs`中的静态属性中公开这两个属性（前面已解释）如下：

    ```cs
    private static string GitHubClientId { get; } = Environment.GetEnvironmentVariable("GithubClientId", EnvironmentVariableTarget.User);
    private static string GitHubSecret { get; } = Environment.GetEnvironmentVariable("GithubSecret", EnvironmentVariableTarget.User);
    ```

本节介绍了在 GitHub 上设置 OAuth 应用的步骤，该应用可以用来请求访问 GitHub 的安全功能，例如更改你的个人数据。有了这些知识，你现在可以使用客户端 ID 和客户端密钥在 GitHub API 上创建授权调用，如下节所示。

## 授权头

授权头有三种形式——基本、API 密钥（或个人访问令牌）和第三方身份验证。GitHub API 不允许来自同一来源的无限制调用。就像 Azure Text Analytics 客户端一样，它也使用 API 密钥。然而，在这种情况下，API 密钥用于速率限制（你每小时可以调用多少次）。对于匿名调用，它每小时只允许 60 次调用。但是，通过使用有效的授权头，这个数量可以增加到 5,000。

在下面的示例中，你将比速率限制允许的次数多调用一次（60 + 1 = 61）。这样，你将获取 61 次用户信息。为了实现这一点，你还需要确保将`CacheControl`头设置为`NoCache`，因为你不希望在 60 次连续调用后忽略请求：

```cs
public static async Task GetUser61Times()
{
    const int rateLimit = 60;
    for (int i = 0; i < rateLimit + 1; i++)
    {
        const string username = "github-user";
        var request = new HttpRequestMessage(HttpMethod.Get, new Uri($"users/{username}", UriKind.Relative));
        request.Headers.CacheControl = new CacheControlHeaderValue(){NoCache = true};

        var response = await client.SendAsync(request);
        if (!response.IsSuccessStatusCode)
        {
            throw new Exception(response.ReasonPhrase);
        }
```

这段代码是对*HttpClient*部分中的`GetUser`方法的改编。这里主要有三个调整：

+   第一个调整是循环中的所有内容都运行 61 次。

+   你还添加了一个错误处理器，这意味着如果响应不是成功的，你将打印出 API 返回的错误信息。

+   最后，你添加一个 `CacheControl` 头来忽略缓存（因为你确实希望有 61 次调用服务器）。

运行此代码在第 61 次调用时会产生错误信息，这证明了 API 速率限制（错误信息已被截断以清晰显示）：

```cs
60) Kaisinel created profile at 2018-06-22 07:51:56
Unhandled exception. System.Exception: rate limit exceeded
```

为了修复这个问题，你需要添加一个 `Authorization` 头（你将在 `CacheControl` 头下面添加它）：

```cs
GitHttp.cs
public static async Task GetUser61Times(string authHeader)
{
    const int rateLimit = 60;
            for (int i = 0; i < rateLimit + 1; i++)
            {
                const string username = "github-user"; // replace with your own
                var request = new HttpRequestMessage(HttpMethod.Get, new Uri($"users/{username}", UriKind.Relative));
               request.Headers.CacheControl = new CacheControlHeaderValue(){NoCache = true};
               request.Headers.Add("Authorization", authHeader);
               var response = await client.SendAsync(request);
                if (!response.IsSuccessStatusCode)
                {
                    throw new Exception(response.ReasonPhrase);
                }
The complete code can be found here: https://packt.link/1C5wb.
```

由于 GitHub 对匿名调用有限制（例如，每小时只能进行 60 次请求以获取用户配置文件信息），你会发现提供授权头以识别自己并因此摆脱这种严格限制会更有效率。在接下来的示例中，你将获取一个授权令牌，并将其提供给此方法，从而展示授权如何帮助你克服速率限制。

当运行放置在 [`packt.link/Uz2BL`](https://packt.link/Uz2BL) 的演示代码时，建议你一次运行一个示例（即，取消注释一行，并在 `Run` 方法中注释掉其余行）。这是因为 `Demo.cs` 文件是授权和匿名调用的混合，你可能会得到意外的结果。但是，保留获取令牌的行，因为它可能被个别示例所需要。

在本节结束时，你应该已经掌握了授权头背后的逻辑及其三种形式——基本认证、API 密钥（或个人访问令牌）和第三方认证——并且了解到，与 Azure 文本分析客户端一样，GitHub API 使用 API 密钥。现在你可以继续学习基本认证。

## 基本认证

基本认证涉及用户名和密码。这两个通常组合成一个字符串，并使用以下格式进行编码：

```cs
Basic username:password
```

这里是用于生成基本认证授权令牌的代码：

```cs
public static string GetBasicToken()
{
    var id = GitHubClientId;
    var secret = GitHubSecret;
    var tokenRaw = $"{id}:{secret}";
    var tokenBytes = Encoding.UTF8.GetBytes(tokenRaw);
    var token = Convert.ToBase64String(tokenBytes);
    return "Basic " + token;
}
```

使用用户名和密码获取基本令牌。然后将它传递给 `GetUser61Times` 方法：

```cs
var basicToken = GitExamples.GetBasicToken();
await GitExamples.GetUser61Times(basicToken);
```

调用 `GetUser61Times` 时不再显示错误，因为通过提供授权头避免了速率限制。

注意

你可以在 [`packt.link/Uz2BL`](https://packt.link/Uz2BL) 和 [`packt.link/UPxmW`](https://packt.link/UPxmW) 找到用于此示例的代码。

下一节将介绍更专业的 API 密钥和个人访问令牌，它们很相似，因为它们都提供了访问受保护数据的权限。

### API 密钥和个人访问令牌

个人访问令牌仅限于个人数据。然而，API 密钥可以用于整个 API。除了可以访问的范围外，两者在使用方式上没有区别。你可以像这样将 API 密钥或个人访问令牌添加到授权头中。

但是，当然，要使用某个 API 的访问令牌，你首先需要创建它。你可以通过以下步骤完成：

1.  在“设置”窗口下，前往 GitHub 的“开发者设置”选项。

1.  导航到“个人访问令牌”（`1`）。

1.  选择“生成新令牌”按钮（`2`）：

![图 8.17：创建新的个人访问令牌](img/B16835_08_17.jpg)

图 8.17：创建新的个人访问令牌

1.  接下来，输入您的 GitHub 密码。

1.  添加一个备注（这可以是任何内容）并向下滚动。此屏幕将帮助您修改用户数据，因此请勾选`user`复选框（`4`）以获取访问权限。

1.  点击`Generate token`按钮（`5`）：

![图 8.18：为个人访问令牌配置的访问范围](img/B16835_08_18.jpg)

图 8.18：为个人访问令牌配置的访问范围

在新窗口中，您将看到所有个人访问令牌，包括新添加的：

![图 8.19：在 GitHub 上创建的新个人访问令牌](img/B16835_08_19.jpg)

图 8.19：在 GitHub 上创建的新个人访问令牌

注意

请记住，您只能看到令牌值一次。因此，请确保您将其安全地复制并存储。此外，请注意，个人访问令牌在一个月后过期，此时您需要重新生成它。

1.  创建一个名为`GitHubPersonalAccess`的环境变量。

1.  将个人访问令牌添加到`Demo.cs`中：

    ```cs
    private static string GitHubPersonAccessToken { get; } = Environment.GetEnvironmentVariable("GitHubPersonalAccess", EnvironmentVariableTarget.User);
    ```

1.  运行以下代码：

    ```cs
    await GetUser61Times(GitHubPersonAccessToken);
    ```

您将观察到调用`GetUser61Times`方法不会失败。

访问令牌、授权令牌、API 密钥和 JWT（将在以下章节中进一步介绍）是向 API 证明您已被授予访问权限并有权访问您想要资源的不同方式。但无论您使用哪种具体的授权方式，它们通常都会指向同一个地方——即授权头。

下一节将详细介绍一个名为 OAuth2 的授权协议。

## 第三方身份验证——OAuth2

GitHub 是一个授权服务器的示例。它允许以所有者的名义访问资源或功能。例如，更新用户的就业状态仅对已登录用户可用。但是，如果用户已被授予执行此操作的访问权限，则可以直接这样做。OAuth2 就是关于代表某人获取访问权限的程序。

执行以下步骤以修改用户的就业状态：

1.  导航到此 URL 或发送一个 HTTP `GET`请求：

    ```cs
    https://github.com/login/oauth/authorize?client_id={{ClientId}}&redirect_uri={{RedirectUrl}}
    ```

在这里，`{{ClientId}}`和`{{RedirectUrl}}`是您在 OAuth2 GitHub 应用中设置的值。

注意

将占位符`{{ClientId}}`和`{{RedirectUrl}}`替换为您 GitHub OAuth 应用中的相应值。

以下屏幕提示您登录到您的 GitHub 应用：

![图 8.20：登录 OAuth2 GitHub 应用](img/B16835_08_20.jpg)

图 8.20：登录 OAuth2 GitHub 应用

1.  完成用户名和密码。

1.  接下来，点击`Sign in`按钮进行登录。

登录成功后，您将被重定向到 OAuth2 应用中指定的 URL。

1.  通过向以下格式的 URI 发送 HTTP `POST`请求来创建令牌请求：

    ```cs
    {tokenUrl}?client_id={clientId}&redirect_uri={redirectUri}&client_secret={secret}&code={code}:
    ```

其代码如下：

```cs
private static HttpRequestMessage CreateGetAccessTokenRequest()
{
    const string tokenUrl = "https://github.com/login/oauth/access_token";
    const string code = "2ecab6ecf412f28f7d4d";
    const string redirectUri = "https://www.google.com/";
    var uri = new Uri($"{tokenUrl}?client_id={GitHubClientId}&redirect_uri={redirectUri}&client_secret={GitHubSecret}&code={code}");
    var request = new HttpRequestMessage(HttpMethod.Post, uri);
    return request;
}
```

在这种情况下，重定向 URL 是[`www.google.com`](https://www.google.com)。你最终得到的 URI 是[`www.google.com/?code=a681b5126b4d0ba160ba`](https://www.google.com/?code=a681b5126b4d0ba160ba)。`code=`部分是获取`OAuth`访问令牌所需的代码。令牌以以下格式返回：

```cs
access_token=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&scope=user&token_type=bearer
```

1.  在此令牌可以使用之前，你需要从响应中解析它。因此，创建一个解析令牌响应的函数：

    ```cs
    private static Dictionary<string, string> ConvertToDictionary(string content)
    {
        return content
            .Split('&')
            .Select(kvp => kvp.Split('='))
            .Where(kvp => kvp.Length > 1)
            .ToDictionary(kvp => kvp[0], kvp => kvp[1]);
    }
    ```

这会将每个`=`属性放入一个字典中。`=`之前的是键，`=`之后的是值。

1.  使用`GetToken`函数创建并发送请求，然后解析响应，然后格式化令牌并返回它：

    ```cs
    private static async Task<string> GetToken()
    {
        HttpRequestMessage request = CreateGetAccessTokenRequest();
        var response = await client.SendAsync(request);
        var content = await response.Content.ReadAsStringAsync();
        Dictionary<string, string> tokenResponse = ConvertToDictionary(content);
        // ValidateNoError(tokenResponse);
        var token = $"{tokenResponse["token_type"]} {tokenResponse["access_token"]}";
        return token;
    }
    ```

在这里，你创建了一个请求，将其发送到客户端，将响应解析为令牌，然后返回。`ValidateNoError`目前被注释掉了。你稍后会回到它。返回的令牌应该看起来像这样：

```cs
bearer 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
```

这个令牌是一个持票令牌，它是由授权服务器（在这种情况下，是 GitHub）生成的，代表你（或任何用于登录 GitHub 的其它用户名）访问 GitHub。你可以用它来发送需要特殊访问权限的请求。例如，更新用户的就业状态。

1.  要更新用户的就业状态，请使用`UpdateEmploymentStatus`函数：

    ```cs
    public static async Task UpdateEmploymentStatus(bool isHireable, string authToken)
    {
        var user = new UserFromWeb
        {
            hireable = isHireable
        };
        var request = new HttpRequestMessage(HttpMethod.Patch, new Uri("/user", UriKind.Relative));
        request.Headers.Add("Authorization", authToken);
        var requestContent = JsonConvert.SerializeObject(user, new JsonSerializerSettings { NullValueHandling = NullValueHandling.Ignore });
        request.Content = new StringContent(requestContent, Encoding.UTF8, "application/json");
        var response = await client.SendAsync(request);
        var responseContent = await response.Content.ReadAsStringAsync();
        Console.WriteLine(responseContent);
    }
    ```

这段代码将用户的属性`isHireable`设置为`true`并打印更新后的用户信息。这里重要的是内容；在发送`PUT`、`PATCH`或`POST`请求时，你通常需要一个带有请求（或换句话说，内容）的主体。

将内存中的对象转换为结构化文本（例如，JSON）的行为称为序列化。在这种情况下，主体是一个用户更新。你发送一个`PATCH`请求，因为你只想更改更新的值。如果内容中没有提供值，则不应更改。这是`PATCH`请求与`POST`请求之间的关键区别——成功的请求会覆盖所有值（即使你没有提供它们）。

你使用了`new JsonSerializerSettings { NullValueHandling = NullValueHandling.Ignore }`来避免提供`null`值。这是因为你不想更新所有字段，只更新你提供的那些字段。

在创建 HTTP 内容时，你还需要提供一个 MIME 类型（请求中发送的一种媒体类型）。这是必要的，以便服务器可以了解它应该如何处理请求。MIME 类型遵循以下格式：

```cs
type/subtype
```

在这种情况下，`application/json`意味着客户端应该期望从服务器接收 JSON。`application`是最常见的 MIME 类型，意味着二进制数据。

此外，还有`StringContent`，它是一种序列化内容，通常是 JSON 或 XML。或者，你也可以使用`StreamContent`或`ByteContent`，但它们稍微少用一些，并且在使用性能或数据量是关注点时使用。

以下代码展示了完整的演示：

```cs
public static async Task Run()
{
    var oathAccessToken = await GitExamples.GetToken();
    await GitExamples.UpdateEmploymentStatus(true, oathAccessToken);
}
```

在 `GetToken` 方法（用于 *第三部分认证（OAuth2）* 部分的 *步骤 6*）中，有一行被注释掉的代码，`ValidateNoError`。取消注释它并实现 `GetToken` 方法，因为你不总是能得到成功的响应，在这种情况下解析令牌将失败（即它不存在）。因此，始终验证服务器响应并在意外发生时抛出异常是一个好主意。查看以下 GitHub 错误格式：

```cs
error=bad_verification_code&error_description=The+code+passed+is+incorrect+or+expired.&error_uri=https%3A%2F%2Fdocs.github.com%2Fapps%2Fmanaging-oauth-apps%2Ftroubleshooting-oauth-app-access-token-request-errors%2F%23bad-verification-code
```

这不是很易读。`ValidateNoError` 将格式化响应并将其作为异常抛出，而不是让它静默失败：

```cs
private static void ValidateNoError(Dictionary<string, string> tokenResponse)
{
    if (tokenResponse.ContainsKey("error"))
    {
        throw new Exception(
            $"{tokenResponse["error"].Replace("_", " ")}. " +
            $"{tokenResponse["error_description"].Replace("+", " ")}");
    }
}
```

如果你再次运行代码并且因为相同的原因失败，错误信息现在将如下所示：

```cs
bad verification code. The code passed is incorrect or expired.
```

本节介绍了如何在有安全措施的情况下发送 HTTP 请求的基础知识。在接下来的部分（*Restsharp* 和 *Refit*）中，你将使用第三方库创建客户端，以减少 `HttpClient` 所需的样板代码。

注意

你可以在 [`packt.link/UPxmW`](https://packt.link/UPxmW) 找到用于此示例的代码。

## 请求幂等性

幂等 HTTP 请求是指总是产生相同结果的请求。只有 `GET`、`PUT` 和 `PATCH` 请求是幂等的，因为它们要么不改变任何内容，要么重复相同的改变，但这种改变永远不会导致错误并产生相同的数据。`DELETE` 不是幂等的，因为删除已删除的项目将产生错误。`POST` 可能是或不是幂等的，这完全取决于实现。

### PUT、PATCH 或 POST

`PUT`、`PATCH` 和 `POST` 之间的区别可以总结如下：

+   `PUT` 用于覆盖模型中的字段。即使明确提供了单个值，整个模型也将包含未提供的值（或者至少这是预期）。例如，如果你想通过首先获取旧详情然后发送修改后的版本来更新用户详情，你会使用 `PUT`。

+   `PATCH` 用于更新明确提供的单个值。例如，如果你想更新用户名，使用 `PATCH` 而不是 `PUT` 请求会更合理。

+   `POST` 用于创建项目或发送复杂查询。无论哪种方式，此动词的默认预期都有副作用。例如，如果你想创建用户，你会使用 `POST` 请求。

## 练习 8.02：HttpClient 调用星球大战 Web API

你可能熟悉星球大战。有电影、游戏和电视剧。然而，你知道它还有多个 API 来检索数据吗？即将到来的练习将介绍 API 的不同格式，并使你熟悉更复杂的响应的反序列化。

在这个练习中，你将创建一个强类型 API 客户端，它将底层使用`HttpClient`。该客户端将用于返回星球大战电影。你将使用星球大战 API（SWAPI）([`swapi.dev/`](https://swapi.dev/))。所需的端点是[`swapi.dev/api/films/`](https://swapi.dev/api/films/)。执行以下步骤以完成此练习：

1.  创建一个新的类来持有`HttpClient`和基本 URL：

    ```cs
    public class StarWarsClient
        {
            private readonly HttpClient _client;
            public StarWarsClient()
            {
                _client = new HttpClient {BaseAddress = new Uri("https://swapi.dev/api/")};
            }
    ```

这将作为一个强类型 API 客户端。

注意

URI 末尾的`/`表示将在 URI（在`api`之后而不是在`dev`之后）之后附加更多文本。

1.  创建一个表示电影的类型：

    ```cs
    Film.cs
    public record Film
    {
        public string Title { get; set; }
        public int EpisodeId { get; set; }
        public string OpeningCrawl { get; set; }
        public string Director { get; set; }
        public string Producer { get; set; }
        [JsonProperty("release_date")]
        public string ReleaseDate { get; set; }
        public string[] Characters { get; set; }
        public string[] Planets { get; set; }
        public string[] Starships { get; set; }
        public string[] Vehicles { get; set; }
        public string[] Species { get; set; }
        public DateTime Created { get; set; }
    ```

```cs
The complete code can be found here: https://packt.link/tjHLa.
```

这是一个你将用于在响应中反序列化电影的类。`ReleaseDate`属性上方有`[JsonProperty("release_date")]`来指定`"release_date"` JSON 字段将映射到`ReleaseDate` C#属性。

1.  创建一个用于存储结果的类型：

    ```cs
    public record ApiResult<T>
    {
        public int Count { get; set; }
        public string Next { get; set; }
        public string Previous { get; set; }
        [JsonProperty("results")]
        public T Data { get; set; }
    }
    ```

这也是一个用于反序列化电影响应的类型；然而，星球大战 API 以分页格式返回结果。它包含`Previous`和`Next`属性，分别指向上一页和下一页。例如，如果你不提供想要的页码，它将返回`null`的值。然而，如果还有剩余元素，下一个属性将指向下一页（否则它也将是`null`）。使用下一个或上一个作为 URI 查询 API 将返回该页的资源。你使用`JsonProperty`属性在`T Data`上方提供 JSON 到属性的映射，因为属性和 JSON 名称不匹配（JSON 字段名是`results`，而`Data`是属性名）。

注意

你可以将`ApiResult`更改为具有`Results`属性而不是`Data`。然而，`ApiResult.Results`有点令人困惑。在编写代码时，与其选择自动化（在这种情况下，序列化）的便利性，不如选择可维护性和可读性。因此，*步骤 3*中选择的名称不同但更清晰。

1.  现在，创建一个获取多个电影的方法：

    ```cs
    public async Task<ApiResult<IEnumerable<Film>>> GetFilms()
    {
    ```

你返回了一个任务，以便其他人可以等待此方法。几乎所有 HTTP 调用都将使用`async Task`。

1.  创建一个 HTTP 请求以获取所有电影：

    ```cs
    var request = new HttpRequestMessage(HttpMethod.Get, new Uri("films", UriKind.Relative));
    ```

URI 是相对的，因为你是从已经设置了基本 URI 的`HttpClient`中调用的。

1.  要查询星球大战 API 中的电影，发送此请求：

    ```cs
    var response = await _client.SendAsync(request);
    ```

1.  它返回`HttpResponseMessage`。这个有两个重要部分：状态码和响应体。C#有一个方法可以根据状态码确定是否有错误。要处理错误，请使用以下代码：

    ```cs
    if (!response.IsSuccessStatusCode)
    {
          throw new HttpRequestException(response.ReasonPhrase);
    }
    ```

错误处理很重要，因为失败的 HTTP 请求通常会返回错误状态码而不是异常。在尝试反序列化响应体之前，建议你做类似的事情，因为如果失败，你可能会得到意外的体。

1.  现在，调用`ReadAsStringAsync`方法：

    ```cs
    var content = await response.Content.ReadAsStringAsync();
    var films = JsonConvert.DeserializeObject<ApiResult<Film>>(content);
        return films;
    }
    ```

响应的内容更有可能是某种流。要将 `HttpContent` 转换为字符串，调用 `ReadAsStringAsync` 方法。这返回一个字符串（JSON），允许你将 JSON 转换为 C# 对象并反序列化结果。最后，通过反序列化响应内容体并将所有内容转换为 `ApiResult<Film>` 来获取结果。

1.  为了演示，创建客户端并使用它获取所有星球大战电影，然后打印它们：

    ```cs
    public static class Demo
    {
        public static async Task Run()
        {
            var client = new StarWarsClient();
            var filmsResponse = await client.GetFilms();
            var films = filmsResponse.Data;
            foreach (var film in films)
            {
                Console.WriteLine($"{film.ReleaseDate} {film.Title}");
            }
        }
    }
    ```

如果一切正常，你应该看到以下结果：

```cs
1977-05-25 A New Hope
1980-05-17 The Empire Strikes Back
1983-05-25 Return of the Jedi
1999-05-19 The Phantom Menace
2002-05-16 Attack of the Clones
2005-05-19 Revenge of the Sith
```

这个练习说明了如何创建强类型 HTTP 客户端以简化操作。

备注

你可以在 [`packt.link/2CHpb`](https://packt.link/2CHpb) 找到这个练习使用的代码。

你可能已经注意到，发送 HTTP 请求和使用 HTTP 客户端的方式非常类似于将简单的文本文件发送到 GitHub API 的方式。即使它们不同，同一个 API 中的端点通常具有相同的要求。然而，如果你每次需要调用 API 时都手动构建 HTTP 请求，你并不高效。更好的方法是创建可重用的东西。一个常见的方法是创建 `BaseHttpClient`。你将在以下活动中将其付诸实践。

## 活动 8.01：通过重用 HttpClient 快速创建 API 客户端

`HttpClient` 的问题是你仍然需要自己管理很多事情：

+   错误处理

+   序列化和反序列化

+   必须的头部信息

+   授权

当你在团队中工作或在更大的项目中工作时，你很可能会进行不止一次的 HTTP 调用。不同调用之间的一致性和相同要求需要被管理。

这个活动的目的是展示你可以简化重复 HTTP 调用的多种方法之一。你将使用 `BaseHttpClient` 类，你首先需要创建这个类。这个类将通用错误处理和反序列化响应和请求，这将显著简化你做出的不同 HTTP 调用。在这里，你将学习如何通过重写 `StarWarsClient` 使用 `BaseHttpClient` 来实现基础客户端。

执行以下步骤以完成此活动：

1.  创建一个基础的 `HttpClient` 类。基础客户端封装 `HttpClient`。因此，你将保留对其的私有引用，并允许它从一个 URL 创建。内部的 `HttpClient` 通常也包含基础头部，但在这个例子中不是必需的。

1.  定义一种为每个方法创建请求的方式。为了简洁，我们坚持使用 `GET` 请求。在 `GET` 请求中，定义默认头部是一个常见的做法，但在这个例子中，它不是强制的。

1.  创建一个发送请求的方法，并包括错误处理和反序列化。

1.  在 SWAPI 中，如果你正在查询多个结果，你会得到 `ApiResult<IEnumerable<T>>` 用于分页。创建一个 `SendGetManyRequest` 方法。

1.  使用你创建的基础客户端，简化 *练习 8.02* 中的客户端。

1.  使用新的 `StarWarsClient` 版本通过相同的演示代码运行代码。

1.  如果您再次使用新的 `StarWarsClient` 运行演示，您应该看到相同的电影返回：

    ```cs
    1977-05-25 A New Hope
    1980-05-17 The Empire Strikes Back
    1983-05-25 Return of the Jedi
    1999-05-19 The Phantom Menace
    2002-05-16 Attack of the Clones
    2005-05-19 Revenge of the Sith
    ```

为了运行此活动，请转到 [`packt.link/GR27A`](https://packt.link/GR27A) 并在 `static void Main(string[] args)` 体中注释掉所有行，除了 `await Activities.Activity01.Demo.Run();`。

注意

此活动的解决方案可以在 [`packt.link/qclbF`](https://packt.link/qclbF) 找到。

以这种方式重用 `HttpClient` 非常有用，因为它消除了代码重复。然而，调用 Web API 并消除重复代码是一个常见问题，并且可能通过某些库以某种方式解决。下一节将探讨如何使用两个流行的 NuGet 包简化对 Web API 的调用：

+   RestSharp

+   Refit

# RestSharp

RestSharp 的理念与基 `HttpClient` 非常相似——减少代码重复。它简化了请求的创建，并为进行 HTTP 调用提供了很多实用工具。使用 `RestSharp` 重新做 `StarWarsClient`，但首先，您需要安装 `RestSharp` NuGet：

```cs
dotnet add package RestSharp
```

现在创建一个与您在 *活动 8.01* 中创建的非常相似的客户端：

```cs
    public class StarWarsClient
    {
        private readonly RestClient _client;
        public StarWarsClient()
        {
            _client = new RestClient("https://swapi.dev/api/");
        }
```

创建 `RestSharp` 后，您将获得一个内置的响应序列化。它还能够猜测您将使用哪个 HTTP 方法：

```cs
        public async Task<ApiResult<IEnumerable<Film>>> GetFilms()
        {
            var request = new RestRequest("films");
            var films = await _client.GetAsync<ApiResult<IEnumerable<Film>>>(request);
            return films;
        }
    }
```

您已经通过了制作 HTTP 请求所需的最小信息（调用电影，返回 `ApiResult<IEnumerable<Film>>`），其余的工作已经完成。这非常类似于您之前编写的基客户端。

注意

`ApiResult` 是与 *练习 8.02* 中相同的类型。

然而，如果您在您的演示中运行此代码，您会注意到 JSON 的 `Data` 属性返回为 `null`。这是因为您在 `response` 和 `film` 类上有一个 `JsonProperty` 属性。RestSharp 使用不同的序列化器，它不知道这些属性。为了使其工作，您可以将所有属性更改为 RestSharp 可以理解的形式，或者使用之前的相同序列化器。您正在使用 `Newtonsoft.Json`，为了在 RestSharp 中使用它，您需要调用 `UseSerializer` 方法，选择 `JsonNetSerializer`：

```cs
        public StarWarsClient()
        {
            _client = new RestClient("https://swapi.dev/api/");
            _client.UseSerializer(() => new JsonNetSerializer());
        }
```

在运行演示时，以下输出会显示：

```cs
1977-05-25 A New Hope
1980-05-17 The Empire Strikes Back
1983-05-25 Return of the Jedi
1999-05-19 The Phantom Menace
2002-05-16 Attack of the Clones
2005-05-19 Revenge of the Sith
```

结果与 *练习 8.02* 中的结果相同；然而，区别在于前一个示例中使用了 `Newtonsoft` 序列化器。`RestSharp` 可能是 `HttpClient` 的最佳抽象，因为它最小化了您需要编写的代码量来执行 HTTP 调用，同时保持了与 `HttpClient` 的相似性。

注意

您可以在 [`packt.link/f5vVG`](https://packt.link/f5vVG) 找到用于此示例的代码。

此示例旨在通过 HTTP 请求与 Web API 进行通信。尽管演示文件看起来相同，但它们使用的是不同的库或设计模式。在接下来的活动中，您将练习使用 RestSharp 消费更多 API。

## 活动八.02：使用 RestSharp 列出所有国家的国家 API

地址[`restcountries.com/v3/`](https://restcountries.com/v3/)是一个公开的 Web API，它提供了所有现有国家的列表。假设您需要使用该 API 获取所有国家的列表，通过首都（例如，维尔纽斯）查找国家，以及查找说给定语言（例如，立陶宛语）的所有国家。您需要只打印前两个国家的名称、它们所在的地区和首都，并实现一个强类型客户端来使用`RestSharp`访问此 API。

本活动的目的是让您在使用第三方库（`RestSharp`）进行 HTTP 调用时感到更加舒适。使用第三方库通常可以节省大量时间。它允许您重用已经存在的东西。

执行以下步骤以完成此活动：

1.  使用[`restcountries.com/v3/`](https://restcountries.com/v3/)的 URL 创建一个基本客户端类。

    注意

    导航到[`restcountries.com/v3/`](https://restcountries.com/v3/)将返回 HTTP 状态码`404`，并显示“页面未找到”消息。这是因为基本 API URI 不包含任何关于资源的信息；它尚未完成，只是资源完整 URI 的开始。

1.  创建用于序列化的模型。

1.  使用示例[`restcountries.com/v3/name/peru`](https://restcountries.com/v3/name/peru)来获取响应。

1.  复制响应，然后使用类生成器，例如[`json2csharp.com/`](https://json2csharp.com/)，将 JSON（响应）转换为模型。

1.  在客户端中创建以下方法：`Get`、`GetByCapital`和`GetByLanguage`。

1.  创建一个调用所有三个方法的演示。

1.  打印每个响应中的国家。

结果应该是这样的：

```cs
All:
Aruba Americas Oranjestad
Afghanistan Asia Kabul
Lithuanian:
Lithuania Europe Vilnius
Vilnius:
Lithuania Europe Vilnius
```

注意

本活动的解决方案可以在[`packt.link/qclbF`](https://packt.link/qclbF)找到。

您现在知道 RestSharp 简化了请求的创建，并为进行 HTTP 调用提供了许多实用工具。下一节将帮助您练习使用 Refit，这是另一种消费 API 的方式。

# Refit

Refit 是最智能的客户端抽象，因为它从一个接口生成客户端。您需要做的只是提供一个抽象：

1.  要使用`Refit`库，首先安装`Refit` NuGet：

    ```cs
    dotnet add package Refit
    ```

1.  要在 Refit 中创建客户端，首先创建一个具有 HTTP 方法的接口：

    ```cs
    public interface IStarWarsClient
    {
        [Get("/films")]
        public Task<ApiResult<IEnumerable<Film>>> GetFilms();
    }
    ```

请注意，这里的端点是`/films`而不是`films`。如果您用`films`运行代码，您将得到一个异常，建议您使用前面的`/`更改端点。

1.  要解析客户端，只需运行以下代码：

    ```cs
    var client = RestService.For<IStarWarsClient>("https://swapi.dev/api/");
    ```

在运行演示时，以下输出将显示：

```cs
1977-05-25 A New Hope
1980-05-17 The Empire Strikes Back
1983-05-25 Return of the Jedi
1999-05-19 The Phantom Menace
2002-05-16 Attack of the Clones
2005-05-19 Revenge of the Sith
```

结果与您在*练习 8.02*中看到的结果相同；然而，区别在于实现方式。

注意

您可以在[`packt.link/cqkH5`](https://packt.link/cqkH5)找到用于此示例的代码。

仅当你的场景非常简单时才使用 Refit。尽管 Refit 可能看起来是最简单的解决方案，但在需要为更复杂的场景进行自定义授权时，它也会带来自己的复杂性。你将在接下来的活动中进一步简化解决方案。

## 活动 8.03：使用 Refit 列出所有国家的国家 API

你知道执行同一任务的不同方法越多，你做出选择并挑选最适合的工具就越容易。不同的团队可能会使用不同的工具，Refit 是一种相当独特、简约的方法，你可能会遇到。其他人可能会说它使工作复杂化，因为客户端界面中隐藏了太多东西（代码越少并不意味着你可以轻松掌握代码）。无论你是否支持 Refit 或反对它，亲自实践并形成自己的观点都是好的。这个活动将帮助你做到这一点。在这里，你将访问国家 API 来显示所有国家、按语言划分的国家以及按首都划分的国家。

这个活动的目的是展示 Refit 在消费简单 API 时如何适用于快速原型设计。以下是这些步骤：

1.  创建用于序列化的模型。为此，使用示例 [`restcountries.com/v3/name/peru`](https://restcountries.com/v3/name/peru) 获取响应。

1.  现在复制响应。

1.  然后使用一个类生成器，例如 [`json2csharp.com/`](https://json2csharp.com/)，从 JSON（响应）中创建模型。

1.  定义一个具有以下方法的接口：`Get`、`GetByCapital` 和 `GetByLanguage`。

1.  创建一个打印国家名称、地区和国家状态的演示。

结果将如下显示：

```cs
All:
Aruba Americas Oranjestad
Afghanistan Asia Kabul
Lithuanian:
Lithuania Europe Vilnius
Vilnius:
Lithuania Europe Vilnius
```

注意

这个活动的解决方案可以在 [`packt.link/qclbF`](https://packt.link/qclbF) 找到。

.NET 有几种创建 HTTP 请求的本地方法，为此，你可以使用 `HttpWebRequest` 或 `WebClient`。这两个方法尚未被弃用，使用它们是可以的，但与较新的 `HttpClient` 相比，它们是较旧的替代方案。下一节将涵盖所有这些内容。

在以下部分，你将了解到其他一些库，它们可以解决在使用 `HttpClient` 时代码重复的问题。

# 其他创建 HTTP 请求的方法

Refit 和 RestSharp 只是众多解决在使用 `HttpClient` 时代码重复问题的库中的两个。Flurl 和 TinyRest 是另外两个流行的替代方案。每年都会创建新的库，并且它们在不断发展。没有一种适合所有场景的最佳方法。为了确保你做出正确的选择，你首先需要进行一些研究，因为对这些替代方案有一些潜在的问题需要考虑。

`HttpClient` 是为 .NET 中的最低级 HTTP 调用而设计的。它是最安全的选项，因为它有良好的文档记录，经过测试，并允许最大的自由度。尽管有许多库比 `HttpClient` 更容易使用，但它们通常针对基本场景（无授权，无动态设置的头）。当涉及到创建高级 HTTP 调用时，它们往往变得相当复杂。

当涉及到选择使用哪个客户端时，首先选择 API 本地提供的客户端。如果没有为 API 提供客户端，考虑你项目的复杂性和范围。对于简单、范围较小的项目，使用你找到的最方便的 NuGet `HttpClient` 替代方案。但如果项目的范围很大且调用复杂，请使用框架提供的本地 `HttpClient`。

在下一个练习中，你将实现一个示例，使用 Refit 将其转变为一个复杂问题。为了解决这个问题，你将同时使用 `HttpClient` 和 RestSharp。

## 练习 8.03：用于在 PayPal 沙盒中测试支付的强类型 HTTP 客户端

编程中常见的场景是进行支付。然而，在开发阶段，你不想使用真实的银行账户，因此寻找在测试环境中处理支付的方法——即沙盒。在这个练习中，你将学习如何调用支付沙盒 API。你将使用 PayPal 的沙盒 API ([`developer.paypal.com/docs/api/orders/v2/`](https://developer.paypal.com/docs/api/orders/v2/)) 创建订单并获取你创建的订单。

这个练习将使用 `Refit` 作为客户端接口和实现解析。它还将使用 `HttpClient` 为 Refit 提供获取 `auth` 标头的途径。最后，你将使用 RestSharp 从 `HttpClient` 内部获取访问令牌。按照以下步骤完成此练习：

1.  前往 [`www.paypal.com/tt/webapps/mpp/account-selection`](https://www.paypal.com/tt/webapps/mpp/account-selection)。

1.  创建一个 PayPal 账户（个人或商业）。

1.  选择你的位置并点击 `Get Started` 按钮。

1.  提供你的手机号码。

1.  点击 `Next` 按钮，并输入代码。

1.  通过输入电子邮件地址和密码设置你的个人资料。

1.  提供你的地址详情。

1.  现在链接你的信用卡或借记卡。你也可以通过遵循 [`www.paypal.com/tt/webapps/mpp/account-selection`](https://www.paypal.com/tt/webapps/mpp/account-selection) 提供的说明免费完成此操作。

    注意

    在 PayPal 上创建账户是免费的。链接信用卡（或借记卡）的要求只是账户创建的一部分，并且不会收取费用。一旦身份验证确认，支付就会退款。

1.  现在退出账户并前往 [`developer.paypal.com/developer/accounts/`](https://developer.paypal.com/developer/accounts/)。

1.  点击 `Log in to Dashboard` 按钮，继续操作：

![图 8.21：登录 PayPal 控制面板以管理沙盒和实时环境](img/B16835_08_21.jpg)

图 8.21：登录 PayPal 控制面板以管理沙盒和实时环境

1.  然后输入所需的凭据，并转到下一屏幕。

1.  在`Sandbox`选项下点击`Accounts`选项。您将看到为您创建的两个测试账户：

![图 8.22：用于测试的沙盒 PayPal 账户](img/B16835_08_22.jpg)

图 8.22：用于测试的沙盒 PayPal 账户

您将使用这些账户在下一步中进行测试。

注意

PayPal 沙盒是免费的。

1.  前往[`developer.paypal.com/developer/applications`](https://developer.paypal.com/developer/applications)获取您的客户端 ID 和密钥。就像 GitHub 示例一样，PayPal 使用 OAuth 应用为您提供客户端 ID 和密钥。

1.  对于默认账户之一，PayPal 还会生成一个默认的 OAuth 应用。因此，点击`Sandbox`选项卡并选择`Default Application`：

![图 8.23：为 PayPal 创建 OAuth 应用](img/B16835_08_23.jpg)

图 8.23：为 PayPal 创建 OAuth 应用

1.  在新窗口中，检查`Client ID`和`Secret`。

1.  注意这两者并将它们存储在环境变量中：

![图 8.24：显示 Client ID 和 Secret 的默认应用程序详情](img/B16835_08_24.jpg)

图 8.24：显示 Client ID 和 Secret 的默认应用程序详情

1.  在一个新空类`Exercise03.AuthHeaderHandler.cs`中创建用于访问 PayPal 客户端 ID 和密钥的属性：

    ```cs
    public static string PayPalClientId { get; } = EnvironmentVariable.GetOrThrow("PayPalClientId");
    public static string PayPalSecret { get; } = EnvironmentVariable.GetOrThrow("PayPalSecret");
    ```

这里使用了`EnvironmentVariable.GetOrThrow`辅助方法来获取用户的环镜变量，如果不存在则抛出异常。您将使用这些属性来连接到沙盒 PayPal API。

注意

您可以在[`packt.link/y2MCy`](https://packt.link/y2MCy)找到用于环境变量的代码。

1.  在`Demo.cs`类中，添加一个`const`变量用于 PayPal 沙盒的`BaseAddress`：

    ```cs
    public const string BaseAddress = "https://api.sandbox.paypal.com/";
    ```

`BaseAddress`将用于初始化具有 PayPal URL 的不同客户端（RestSharp 和 Refit）。

1.  使用`Refit`创建一个具有`CreateOrder`和`GetOrder`方法的客户端：

    ```cs
    public interface IPayPalClient
    {
        [Post("/v2/checkout/orders")]
        public Task<CreatedOrderResponse> CreateOrder(Order order);
        [Get("/v2/checkout/orders/{id}")]
        public Task<Order> GetOrder(string id);
    }
    ```

要获取示例请求，请参考您想要调用的 API 的文档。通常，它们都有一个示例请求。在这种情况下，PayPal 的 `CreateOrder` 请求可以在[`developer.paypal.com/docs/api/orders/v2/`](https://developer.paypal.com/docs/api/orders/v2/)找到：

```cs
{
   "intent":"CAPTURE",
   "purchase_units":[
      {
         "amount":{
            "currency_code":"USD",
            "value":"100.00"
         }
      }
   ]
}
```

![图 8.25：带有突出显示的 PayPal CreateOrder 请求体的示例请求](img/B16835_08_25.jpg)

图 8.25：带有突出显示的 PayPal CreateOrder 请求体的示例请求

在*图 8.25*中，`-d`是一个参数，不属于请求体。

1.  使用[`json2csharp.com/`](https://json2csharp.com/)生成 C# 类，从 JSON 中生成相应的 C# 类。

1.  将`RootObject`重命名为`Order`，并将所有类更改为`record`类型，因为这对于 DTO 是一个更合适的类型：

    ```cs
    IPayPalClient.cs
    public record Order
    {
        public string intent { get; set; }
        public Purchase_Units[] purchase_units { get; set; }
    }
    public record Name
    {
        public string name { get; set; }
    }
    public record Purchase_Units
    {
        public Amount amount { get; set; }
        public Payee payee { get; set; }
    ```

```cs
The complete code can be found here: https://packt.link/GvEZ8.
```

1.  使用相同的 PayPal 文档([`developer.paypal.com/docs/api/orders/v2/`](https://developer.paypal.com/docs/api/orders/v2/))，复制示例响应：

    ```cs
    {
        "id": "7XS70547FW3652617",
        "intent": "CAPTURE",
        "status": "CREATED",
        "purchase_units": [
            {
                "reference_id": "default",
                "amount": {
                    "currency_code": "USD",
                    "value": "100.00"
                },
                "payee": {
                    "email_address": "sb-emttb7510335@business.example.com",
                    "merchant_id": "7LSF4RYZLRB96"
                }
            }
        ],
        "create_time": "2021-09-04T13:01:34Z",
        "links": [
            {
                "href": "https://api.sandbox.paypal.com/v2/checkout/orders/7XS70547FW3652617",
                "rel": "self",
                "method": "GET"
            }
         ]
    }
    ```

1.  使用[`json2csharp.com/`](https://json2csharp.com/)从 JSON 生成 C#类。在这里，你会得到与请求 JSON 非常相似的类。唯一的区别是响应（为了简洁而简化）：

    ```cs
    public class CreateOrderResponse
    {
        public string id { get; set; }
    }
    ```

1.  使用`AuthHeaderHandler`在请求时获取访问令牌，并确保它继承自`DelegatingHandler`：

    ```cs
    public class AuthHeaderHandler : DelegatingHandler
    {
    ```

要调用 PayPal，你需要在每个请求中包含`auth`头。`auth`头的值是从另一个端点检索的。Refit 不能随意添加头。然而，你可以使用自定义`HttpClient`和自定义`HttpMessageHandler`设置 Refit，该处理器在每次请求时都会获取访问令牌。`AuthHeaderHandler`就是为此而使用的。

`DelegatingHandler`是一个允许在发送`HttpRequest`时拦截它，并在发送前后执行某些操作的类。在这种情况下，在发送 HTTP 请求之前，你将获取`auth`头并将其添加到发送的请求中。

1.  现在，通过向`AuthenticationHeader`添加 bearer 令牌来覆盖`SendRequest`：

    ```cs
    protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
    {
                    var accessToken = await GetAccessToken(CreateBasicAuthToken());
                    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
                    return await base.SendAsync(request, cancellationToken).ConfigureAwait(false);
    }
    ```

1.  要获取访问令牌，你首先需要使用基本的`auth`（客户端 ID 和密钥）获取 OAuth 令牌：

    ```cs
     private static string CreateBasicAuthToken()
          {
                    var credentials = Encoding.GetEncoding("ISO-8859-1").GetBytes(PayPalClientId + ":" + PayPalSecret);
                    var authHeader = Convert.ToBase64String(credentials);
                    return "Basic " + authHeader;
          }
    ```

1.  获取访问令牌需要`auth`令牌。使用`RestSharp`客户端并在请求中添加`Authorization`头。

1.  接下来，根据 PayPal API 规范将`content-type`设置为`application/x-www-form-urlencoded`。

1.  按如下方式添加正文内容`grant_type=client_credentials`：

    ```cs
                private static async Task<string> GetAccessToken(string authToken)
                {
                    var request = new RestRequest("v1/oauth2/token");
                    request.AddHeader("Authorization", authToken);
                    request.AddHeader("content-type", "application/x-www-form-urlencoded");
                    request.AddParameter("application/x-www-form-urlencoded", "grant_type=client_credentials", ParameterType.RequestBody);
    ```

1.  使用私有嵌套类`Response`执行前面的请求并返回响应，以简化你的工作：

    ```cs
                    var response = await RestClient.ExecuteAsync<Response>(request, Method.POST);
                    return response.Data.access_token;
                }
            private class Response
            {
                public string access_token { get; set; }
            }
          }
    ```

为什么需要嵌套类？在这里，访问令牌嵌套在响应中。它返回的不仅仅是一个字符串，而是一个对象。自己从 JSON 解析它可能会稍微复杂一些。然而，你已经知道如何反序列化对象。所以，即使只有一个属性，反序列化仍然有帮助。

1.  现在，为`GetAccessToken`方法创建`RestClient`。在`AuthHandler`类中这样做：

    ```cs
    private static readonly RestClient RestClient = new RestClient(baseAddress);
    ```

1.  在`Demo`类中创建`Run`方法：

    ```cs
    public static async Task Run()
            	{
    ```

1.  使用自定义`AuthHeaderHandler`提供程序解析`Refit`客户端：

    ```cs
                var authHandler = new AuthHeaderHandler {InnerHandler = new HttpClientHandler() };
                var payPalClient = RestService.For<IPayPalClient>(new HttpClient(authHandler)
                    {
                        BaseAddress = new Uri(baseAddress)
                    });
    ```

1.  假设通过创建`Order`对象进行了支付，运行以下代码：

    ```cs
    var order = new Order
                {
                    intent = "CAPTURE",
                    purchase_units = new[]
                    {
                        new Purchase_Units
                        {
                            amount = new Amount
                            {
                                currency_code = "EUR", value = "100.00"
                            }
                        }
                    }
                };
    ```

1.  现在，调用 PayPal API 并创建一个带有你刚刚创建的订单的订单端点。

1.  获取创建的订单以查看它是否工作，并打印检索到的订单支付信息：

    ```cs
    var createOrderResponse = await payPalClient.CreateOrder(order);
    var payment = await payPalClient.GetOrder(createOrderResponse.id);
    var pay = payment.purchase_units.First();
    Console.WriteLine($"{pay.payee.email_address} - " +
                                  $"{pay.amount.value}" +
                                  $"{pay.amount.currency_code}");
    ```

在设置正确的环境变量后，你应该看到以下输出：

```cs
sb-emttb7510335@business.example.com - 100.00EUR
```

如前所述，这是一个沙盒 API。然而，切换到使用真实货币的实时环境只需在该环境中设置新的 PayPal 账户并调用不同的端点：[`api-m.paypal.com`](https://api-m.paypal.com)。

注意

您将无法访问 [`api-m.paypal.com`](https://api-m.paypal.com)，因为它用于生产 PayPal，并且是付费的。然而，当您准备好进行与 PayPal 的实际集成时，代码中应该只有一处变化（不同的基本 URI）。

请确保您已设置环境变量，并使用您自己的客户端和密钥。否则，可能会显示一些未处理的异常错误。

注意

你可以在 [`packt.link/cFRq6`](https://packt.link/cFRq6) 找到用于此练习的代码。

你现在已经知道如何使用 Web API 进行简单的 CRUD 操作。然而，到目前为止你只处理过文本。那么，调用带有图像的 API 会有所不同吗？在下一个活动中找出答案。

## 活动 8.04：使用 Azure Blob 存储客户端上传和下载文件

Azure Blob Storage 是 Azure 上的一个云服务，用于存储不同的文件（日志、图像、音乐和整个驱动器）。在您可以使用任何 Azure 存储服务之前，您将需要一个存储帐户。Blob 只是文件，但它们不能直接存储在帐户中；相反，它们需要一个容器。

Azure 存储容器就像一个目录，其中存储着其他文件。然而，与目录不同的是，容器不能包含其他容器。使用 Azure 存储帐户创建两个容器，上传一个图像和一个文本文件，然后在本地上传的文件。所有这些都将在你自己的客户端中完成，该客户端围绕 Azure Blob 存储客户端。

本活动的目的是熟悉通过云存储处理文件，同时将您迄今为止所学的一切应用到实践中。执行以下步骤以完成此活动：

1.  导航到 `Azure 存储帐户`。

1.  创建一个新的 Azure 存储帐户。

1.  将 blob 存储访问密钥存储在名为 `BlobStorageKey` 的环境变量中。

1.  安装 `Azure Blob Storage` 客户端。

1.  创建 `FilesClient` 类以存储 blob 客户端和默认容器客户端的字段（blob 将默认存储在此容器中）。

1.  创建一个构造函数来初始化两个客户端（以支持访问不同的容器）。

1.  添加一个方法来创建容器或获取已存在的容器。

1.  创建一个方法来将文件上传到特定的容器。

1.  创建一个方法来从特定的容器中下载文件。

1.  创建一个 `Demo` 类，包含下载和上传目录的路径。

1.  添加测试数据，即两个文件——即一个图像和一个文本文件（*图 8.26*、*图 8.27* 和 *图 8.28*）：

![图 8.26：您的存储帐户中的两个 Azure 存储容器，exercise04 和 exercise04b](img/B16835_08_26.jpg)

图 8.26：您的存储帐户中的两个 Azure 存储容器，exercise04 和 exercise04b

文本文件：

![图 8.27：练习 04 容器中上传的 Test1.txt 文件](img/B16835_08_27.jpg)

图 8.27：练习 04 容器中上传的 Test1.txt 文件

图像文件：

![图 8.28：exercise04b 容器中上传的 Morning.jpg 文件](img/B16835_08_28.jpg)

图 8.28：exercise04b 容器中上传的 Morning.jpg 文件

1.  创建名为 `Run` 的方法以上传文本文件，然后将其本地下载。

1.  运行代码。如果你一切都做对了，你应该会看到以下输出，两个文件都已本地下载：

![图 8.29：演示代码执行后从两个容器中下载的 Morning.jpg 和 Test1.txt 文件](img/B16835_08_29.jpg)

图 8.29：演示代码执行后从两个容器中下载的 Morning.jpg 和 Test1.txt 文件

注意

该活动的解决方案可在 [`packt.link/qclbF`](https://packt.link/qclbF) 找到。

几乎不可能创建一个适合所有人的完美客户端。因此，即使有人给你提供了一个问题的解决方案，你通常仍然需要进一步抽象它，以适应解决你确切的问题。你遇到的问题是上传和下载特定文件夹中的文件。为了解决这个问题，你抽象出多层客户端，只暴露两个函数——一个用于上传文件，另一个用于下载文件。

# 摘要

无论你是什么类型的程序员，都会有许多你必须消费网络服务的场景。在线上有不同种类的服务，但最常见的一种是 RESTful。REST 只是一组指南，因此不应与 HTTP 混淆。REST API 简单、自文档化、结构良好，目前是 Web API 的黄金标准。然而，在大多数情况下，在 RESTful API 的上下文中，请求是通过 HTTP 发送的，你的消息包含 JSON。

使用 C# 进行 HTTP 调用的主要工具是 `HttpClient`，然而，在你尝试自己实现 HTTP 调用之前，你应该寻找你试图消费的 Web API 的 NuGet 包。Azure Blob 存储空间、Azure 文本分析、PayPal 和 GitHub 只是 Web API 的几个例子。

在本章中，你了解了许多为你完成的网络功能。消费起来并不困难；你现在需要知道的是如何与第三方 RESTful 网络 API 进行通信。在下一章中，你将学习如何使用 ASP.NET Core Web API 模板创建自己的 RESTful 网络服务，同时还将介绍 Azure Functions 和特殊的工具 Swagger 以及 NuGet。
