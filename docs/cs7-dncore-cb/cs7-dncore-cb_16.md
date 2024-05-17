# 第十六章：Azure 和无服务器计算

现在，我敢打赌，有些人来到这一章，问道：“无服务器计算到底是什么意思？”名字很令人困惑，我同意。对我来说毫无意义，但当你理解这个概念时，它有点意义。在这一章中，我们将看看无服务器计算这个术语的含义。我们还将看一下：

+   创建 Azure 函数

+   使用 DocRaptor 提供打印功能

+   使用 AWS 和 S3

+   使用 AWS 和 S3 创建 C# lambda 函数

# 介绍

无服务器并不意味着没有服务器，而是你（或应用程序）不知道用于为应用程序提供某些功能的服务器是哪个。因此，无服务器描述了一个依赖于云中的某些第三方应用程序或服务来为应用程序提供一些逻辑或功能的应用程序。

让我们以学生研究门户的例子来说明。学生研究某个主题并在门户中创建相关的文档。然后他们可以加载打印信用到他们的个人资料中，并打印他们需要的保存的文档。在打印一页后，打印信用将从他们的个人资料中扣除。

虽然这是一个非常简单的例子，但我用它来说明无服务器计算的概念。我们可以将应用程序分成各种组件。具体如下：

1.  登录认证

1.  购买打印信用

1.  更新剩余的打印信用

1.  打印文档

这里可能需要其他未提及的组件，但这不是现实世界。我们只是创建这个假设的应用程序来说明无服务器计算的概念。

当已经有第三方服务提供登录认证时，为什么还要在您的应用程序中编写代码来提供登录认证呢？同样，当有提供打印文档的服务时，为什么还要编写代码来打印文档呢？任何特定的功能，比如购买和加载学生打印信用，都可以使用 Azure 函数来创建。无服务器计算的主题是广泛的，而且还处于起步阶段。还有很多东西要学习和体验。让我们迈出第一步，探索这对开发人员有什么好处。

# 创建 Azure 函数

为什么选择 Azure Functions？想象一下，您有一个应用程序需要提供一些特定的功能，但当对函数的调用率增加时，它仍然会扩展。这就是 Azure Functions 提供的好处所在。使用 Azure Functions，您只支付函数在特定时间点所需的计算，而且它立即可用。

要开始，请访问[`azure.microsoft.com/en-us/services/functions`](https://azure.microsoft.com/en-us/services/functions)并创建一个免费账户。

因为在运行 Azure Functions 时，您只支付实际使用的计算时间，所以您的代码尽可能优化是至关重要的。如果您重构 Azure Function 代码并获得了 40%的代码执行改进，那么您直接节省了 40%的月度费用。您重构和改进代码的越多，您就能节省更多的钱。

# 准备工作

您需要设置一个 Azure 账户。如果您还没有账户，可以免费设置一个。从 Azure 门户，在左侧菜单中，点击“新建”开始：

![](img/B06434_17_01.jpg)

在搜索框中，输入“函数应用程序”并点击“Enter”按钮。第一个结果应该是函数应用程序。选择它。

![](img/B06434_17_02.jpg)

当您选择函数应用程序时，您将看到右侧弹出此屏幕。描述完美地描述了 Azure Functions 的功能。在此表单底部点击“创建”按钮。

![](img/B06434_17_03.jpg)

现在您看到一个表单，允许您为函数命名并选择资源组和其他设置。完成后，点击“创建”按钮。 

![](img/B06434_17_04.jpg)

# 如何做...

1.  Azure 创建新的函数应用程序后，您将能够创建 Azure 函数。我们要做的就是创建一个 Azure 函数，每当 GitHub 存储库上发生某些事情时就会触发。单击创建自定义函数链接。

![](img/B06434_17_05.jpg)

根据 Microsoft Azure 网站，编写 Azure Functions 时支持以下内容：JavaScript、C#、F#以及 Python、PHP、Bash、Batch 和 PowerShell 等脚本选项。

1.  现在您将看到可以在几个模板之间进行选择。从语言选择中选择 C#，从场景选择中选择 API 和 Webhooks，然后选择 GitHubWebHook-CSharp 模板。Azure 现在会要求您为函数命名。我将我的命名为`GithubAzureFunctionWebHook`。单击创建按钮创建函数。

![](img/B06434_17_06.jpg)

1.  创建函数后，您将看到在线代码编辑器中为您添加了一些默认代码。

```cs
        using System.Net;

        public static async Task<HttpResponseMessage> Run
                              (HttpRequestMessage req, TraceWriter log)
        {
          log.Info("C# HTTP trigger function processed a request.");

          // Get request body
          dynamic data = await req.Content.ReadAsAsync<object>();

          // Extract github comment from request body
          string gitHubComment = data?.comment?.body;

          return req.CreateResponse(HttpStatusCode.OK, "From Github:" +
                                    gitHubComment);
        }

```

1.  在`return`语句之前，添加以下代码行：`log.Info($"来自 GitHub 的消息：{gitHubComment}");`。这样我们就可以看到从 GitHub 发送的内容。

1.  您的代码现在应如下所示。请注意，有两个链接可让您获取函数 URL 和 GitHub 秘钥。单击这些链接，然后将每个值复制到记事本中。单击保存并运行按钮。

您的 Azure 函数 URL 应类似于：`https://funccredits.azurewebsites.net/api/GithubAzureFunctionWebHook`

![](img/B06434_17_07.jpg)

1.  转到 GitHub 网站[`github.com/`](https://github.com/)。如果您没有帐户，请创建一个并创建一个存储库（GitHub 对开源项目免费）。转到您创建的存储库，然后单击设置选项卡。在左侧，您将看到一个名为 Webhooks 的链接。单击该链接。

![](img/B06434_17_08.jpg)

1.  现在您将看到右侧有一个名为添加 webhook 的按钮。单击该按钮。

![](img/B06434_17_09.jpg)

1.  将之前复制的 Azure Function URL 添加到 Payload URL 字段。将内容类型更改为 application/json，并将之前复制的 GitHub 秘钥添加到秘钥字段。选择 Send me everything，然后单击添加 webhook 按钮。

![](img/B06434_17_10.jpg)

# 工作原理...

在您的 GitHub 存储库中，打开一个文件并向其添加评论。单击此提交上的评论按钮。

![](img/B06434_17_11.jpg)

返回到 Azure Function 并查看日志窗口。此窗口直接位于代码窗口下方。您将看到我们在 GitHub 中发布的评论出现在 Azure Function 的日志输出中。

![](img/B06434_17_12.jpg)

如果日志窗口中没有显示任何内容，请确保您已单击 Azure Function 的运行按钮。如果一切都失败了，请单击测试窗口底部的运行按钮。

虽然这只是一个非常简单的例子，但 Azure Functions 的实用性应该变得明显。您还会注意到函数具有`.csx`扩展名。重要的是要注意，无论您选择使用哪种编程语言编写代码，Azure Functions 都共享一些核心概念和组件。归根结底，函数是这里的主要概念。您还有一个包含 JSON 配置数据的`function.json`文件。您可以通过单击右侧的查看文件链接来查看此文件和其他文件。

![](img/B06434_17_13.jpg)

单击`function.json`文件，您将看到 JSON 文件的内容。将`disabled`属性更改为`true`将有效地阻止函数在调用时执行。您还会注意到`bindings`属性。在这里，您可以配置您的 web hook。所有这些设置都可以在 Azure Function 的集成和其他部分中设置。

```cs
        {
          "bindings": [
            {
              "type": "httpTrigger",
              "direction": "in",
              "webHookType": "github",
              "name": "req"
            },
            {
              "type": "http",
              "direction": "out",
              "name": "res"
            }
          ],
          "disabled": false
        }

```

Azure Functions 和向开发人员提供的好处是一个令人兴奋的概念。这是您编程技能中的一个领域，肯定会让您忙碌很多小时，因为您将探索更复杂和复杂的任务。

# 使用 DocRaptor 提供打印功能

从 Web 应用程序中打印一直是棘手的。如今，由于提供打印功能的众多第三方控件的可用性，这变得更加容易。然而，现实情况是，我遇到过许多项目，在开发时使用了第三方控件来提供打印功能。当时，第三方控件很好，确实满足了他们的需求。

使应用程序具有这种功能意味着购买这些第三方控件的公司很少继续续订他们的许可证。然而，几年后，这将导致 Web 应用程序包含旧的和过时的打印技术。虽然这没有什么问题，但它确实有一些缺点。

开发人员通常被困在维护老化的代码库中，这些代码库被锁定在这个第三方控件中。一旦需求发生变化，你会发现开发人员不得不使代码在第三方控件的限制内工作。或者，他们需要向管理层提出建议，建议更新第三方控件到最新版本。这意味着在打印模块中需要的小改动，结果比任何人的预算都要昂贵。

现实世界：我曾经在一家公司工作，这家公司会让顾问为客户报价某个应用功能的更改。一旦报价得到接受，就交给开发人员在规定的时间和预算内使其工作。这导致开发人员不得不修改代码以使其工作，并满足预算和截止日期，因为缺乏适当的项目管理技能。替换第三方控件几乎是不可能的，因为预算已经在没有开发人员参与的情况下确定了。

我同意有一些开发人员在维护老化的代码库中提供和维护功能方面做得非常好。我也非常喜欢第三方控件和它们提供的功能。开发人员可以选择一些大型的供应商。但问题是：当你只需要打印发票时，为什么要购买一套第三方控件？按照这种逻辑，在许多情况下（包括本例），无服务器更有意义。

# 准备工作

这个示例将介绍一个名为 DocRaptor 的服务。这项服务并不是免费的，但考虑一下在您的 Web 应用程序中编写和维护提供打印功能的代码的成本。考虑购买第三方控件以提供相同的功能的成本。最终取决于作为开发人员的您选择什么才是最合理的。

创建一个基本的 Web 应用程序，然后转到工具，NuGet 包管理器，包管理器控制台。在控制台中键入以下命令以安装 DocRaptor NuGet 包。

```cs
Install-Package DocRaptor

```

安装了 DocRaptor 后，您可以访问他们的网页（[`docraptor.com/`](http://docraptor.com/)）阅读一些 API 文档，或者您也可以访问 GitHub 页面（[`github.com/DocRaptor/docraptor-csharp`](https://github.com/DocRaptor/docraptor-csharp)）获取更多信息。

最好查看本书附带的源代码，以便复制本示例的代码。

# 如何做到这一点...

1.  添加一个包含发票详细信息的 aspx 网页。我只是从 DocRaptor 网站的示例中简单地提取并稍作修改。将此页面命名为`InvoicePrint.aspx`。

我已经在名为`invoice.css`的样式表中包含了 CSS。一定要从本书附带的源代码中获取这个。

有几种方法可以处理这段代码。这并不一定是创建 Web 页面的唯一方法。如果您使用.NET Core MVC，您的方法可能会有所不同。但是，如果您这样做，请记住，这段代码只是为了说明这里的概念。

```cs
    <%@ Page Language="C#" AutoEventWireup="true" CodeBehind="InvoicePrint.aspx.cs" Inherits="Serverless.InvoicePrint" %>

    <!DOCTYPE html>

    <html >
      <head runat="server">
         <title>Invoice</title>
         <meta http-equiv="content-type" content="text/html;
          charset=utf-8"/>
        <link href="css/invoice.css" rel="stylesheet" />
        <script type="text/javascript">
          function ToggleErrorDisplay()
          {
            if ($("#errorDetails").is(":visible")) {
              $("#errorDetails").hide();
            } else {
              $("#errorDetails").show();
            }
          }

          function TogglePrintResult() {
            if ($("#printDetails").is(":visible")) {
              $("#printDetails").hide();
            } else {
              $("#printDetails").show();
            }
          }
        </script>
      </head>
      <body>
        <form runat="server">
          <div id="container"> 
            <div id="main">
              <div id="header">
                <div id="header_info black">The Software Company
                  <span class="black">|</span> (072)-412-5920 
                  <span class="black">|</span> software.com</div>
              </div>
              <h1 class="black" id="quote_name">Invoice INV00015</h1>
              <div id="client" style="float: right">
                <div id="client_header">client:</div>
                <p class="address black">
                  Mr. Wyle E. Coyote
                </p>
              </div>
              <table id="phase_details">
                <thead>
                  <tr>
                    <th class="title">Stock Code</th>
                    <th class="description">Item Description</th>
                    <th class="price">price</th>
                  </tr>
                </thead>
                <tr class="first black">
                  <td>BCR902I45</td>
                  <td>Acme Company Roadrunner Catch'em Kit</td>
                  <td class="price">
                    <div class="price_container">$300</div>
                  </td>
                </tr>
                <tr>
                  <td></td>
                  <td>Booster Skates</td>
                  <td class="price">
                    <div class="price_container">$200</div>
                  </td>
                </tr>
                <tr>
                  <td></td>
                  <td>Emergency Parachute</td>
                  <td class="price">
                     <div class="price_container">$100</div>
                  </td>
                </tr>
                <tr class="last">
                  <td></td>
                  <td></td>
                  <td></td>
                </tr>
                <tr class="first black">
                  <td>BFT547J78</td>
                  <td>Very Sneaky Trick Seed Kit</td>
                  <td class="price">
                    <div class="price_container">$800</div>
                  </td>
                </tr>
                <tr>
                  <td></td>
                  <td>Giant Magnet and Lead Roadrunner Seeds</td>
                  <td class="price">
                    <div class="price_container">$500</div>
                  </td>
                </tr>
                <tr>
                  <td></td>
                  <td>Rollerblades</td>
                  <td class="price">
                    <div class="price_container">$300</div>
                  </td>
                </tr>
                <tr class="last">
                  <td></td>
                  <td></td>
                  <td></td>
                </tr>
              </table>
            </div>
            <div id="total_price">
              <h2>TOTAL: <span class="price black">$1100</span></h2>
            </div>
            <div id="print_link">
              <asp:LinkButton ID="lnkPrintInvoice" runat="server"
                Text="Print this invoice" OnClick="lnkPrintInvoice_Click">
              </asp:LinkButton> 
            </div>
            <div id="errorDetails">
              <asp:Label ID="lblErrorDetails" runat="server">
              </asp:Label>
            </div>
            <div id="printDetails">
              <asp:Label ID="lblPrintDetails" runat="server">
              </asp:Label>
            </div>
          </div>
        </form>

      </body>
    </html>

```

1.  我还创建了一个名为`invoice.html`的发票页面的打印友好版本。

1.  下一步是为链接按钮创建一个单击事件。将以下代码添加到单击事件。您会注意到，我只是将生成 PDF 文档的路径硬编码为：`C:tempinvoiceDownloads`。如果您想要输出到不同的路径（或者获取相对于您所在服务器的路径），请确保更改此路径。

```cs
        Configuration.Default.Username = "YOUR_API_KEY_HERE";
        DocApi docraptor = new DocApi();

         Doc doc = new Doc(
           Test: true,
           Name: "docraptor-csharp.pdf",
           DocumentType: Doc.DocumentTypeEnum.Pdf,
           DocumentContent: GetInvoiceContent()
        );

        byte[] create_response = docraptor.CreateDoc(doc);
        File.WriteAllBytes(@"C:tempinvoiceDownloadsinvoice.pdf",
                           create_response);

```

1.  确保在您的网页中包含以下命名空间：

```cs
        using System;
        using System.Web.UI;
        using DocRaptor.Client;
        using DocRaptor.Model;
        using DocRaptor.Api;
        using System.IO;
        using System.Net;
        using System.Text;

```

1.  最后，获取名为`invoice.html`的打印友好页面的 HTML 内容。下面代码中的 URL 在您的机器上会有所不同，因为您的端口号可能不同。

```cs
        private string GetInvoiceContent()
        {
          WebRequest req = WebRequest.Create
                              ("http://localhost:37464/invoice.html");
          WebResponse resp = req.GetResponse();
          Stream st = resp.GetResponseStream();
          StreamReader sr = new StreamReader(st, Encoding.ASCII);
          return sr.ReadToEnd();
        }

```

# 它是如何工作的...

运行您的 Web 应用程序并查看在 Web 页面上显示的基本发票。确保您已将`InvoicePrint.aspx`页面设置为 Web 应用程序的起始页面。单击“打印此发票”链接。

![](img/B06434_17_14.jpg)

您将看到发票已创建在您指定的输出路径中。

![](img/B06434_17_15.jpg)

单击 PDF 文档以打开发票。

![](img/B06434_17_16.jpg)

DocRaptor 为开发人员创建 Web 解决方案提供了一个非常有用的服务。如果您需要从您的应用程序创建 PDF 或 Excel 文档，DocRaptor 可以使您的团队受益。本例中使用的测试文档可免费使用，不会从您的月配额中扣除（如果您是付费计划用户）。

从真正无服务器的意义上讲，DocRaptor 为您提供功能，而无需您编写大量额外的代码。它非常容易实现，也非常容易维护。前面的例子非常基本，但您可以传递给 DocRaptor 一个 URL，而不是`DocumentContent`，以打印您想要的页面。从开发人员的角度来看，他们不关心 DocRaptor 是如何做到的。它只是有效。这就是无服务器计算背后的理念。

开发人员可以轻松、毫不费力地在他们的应用程序中实现解决方案，并在记录时间内使用最少的代码为他们正在开发的应用程序增加了很多价值。随着需求的增加，实施的功能也可以轻松扩展。但是，专业计划会有超额费用。最后，创建几个 PDF 文档可能不会对服务器计算能力产生太大影响。然后考虑到 DocRaptor 被一些大公司使用，这些公司可能每个月生成数千份文档。所有这些文档生成请求都不是由使用 DocRaptor 的客户处理的，而是由 DocRaptor 服务器自己处理的。

然后，您可以开发一个轻量级、简化的 Web 应用程序，随着访问量的增加，不会对您的服务器造成巨大的需求。

# 使用 AWS 和 S3

没有看到 Amazon Web Services（AWS）这一章就不能算完整。AWS 的主题非常广泛。该平台提供了许多功能。开发人员可以在他们的应用程序中利用这一点，并在他们自己的部分上使用最少的代码提供丰富的功能。AWS 还有非常好的文档，开发人员可以快速查看以迅速掌握。S3 是亚马逊的简单存储服务，允许您在云中存储和检索数据。

我喜欢和我的孩子们一起玩 Minecraft。他们创造的一些东西令人难以置信，尤其是因为我的女儿（以 CupcakeSparkle 的身份玩耍）只有 7 岁，而我的儿子（以 Cheetah 的身份玩耍）只有 4 岁。我的女儿从 5 岁开始玩 Minecraft，可以想象，她已经创造了相当多令人难以置信的结构。Joseph Garrett 绝对是我孩子们最喜欢的 YouTuber，他以 Stampy Cat 的身份玩耍。他们经常（包括与 Squid Nugget 一起建造时间）看他的游戏视频。我们经常举行自己的建造时间比赛，而 Stampy Cat 和他美丽的世界则成为我的孩子们在 Minecraft 中所做的一切的灵感来源。

这是我女儿建造的 Stampy Cat 的图片。

![](img/B06434_17_23.png)

这是我儿子建造的 Squid Nugget 的图片。

![](img/B06434_17_24.png)

因此，我想创建一个地方来上传一些他们的 Minecraft 图片、截图和与我们的 Minecraft 冒险相关的其他文档。为此，我们将使用 S3。

# 准备工作

本章假设您已经注册了 AWS 账户并使用了免费套餐。有关免费套餐的更多详细信息，请转到[`aws.amazon.com/free/`](https://aws.amazon.com/free/)。不过，我想要强调的一个部分是：

<q>亚马逊网络服务（AWS）免费套餐旨在让您能够亲身体验 AWS 云服务。AWS 免费套餐包括在您注册 AWS 后的 12 个月内提供免费套餐的服务，以及在您的 12 个月 AWS 免费套餐期满后不会自动到期的其他服务提供。</q>

为了注册，您需要提供您的信用卡信息。免费套餐期满后（或者如果您的应用程序超出了使用限制），您将按照按使用量付费的服务费率收费。特别是关于 S3，免费套餐允许 5GB 的存储空间，20,000 个获取请求和 2,000 个放置请求。首先，您需要创建一个 S3 存储桶。从服务选择中，找到存储组，然后点击 S3。

![](img/B06434_17_21.jpg)

创建您的第一个存储桶。我将其命名为`familyvaultdocs`并选择了 EU（法兰克福）地区。点击下一步，直到完成存储桶的创建。 

![](img/B06434_17_17.jpg)

创建存储桶后，您可以查看存储桶的权限。为简单起见，我已选择让所有人对对象访问和权限访问具有读取和写入权限。

![](img/B06434_17_18.jpg)

最后，您还需要为您的应用程序创建访问密钥和秘密密钥。从服务中查找安全、身份和合规性组，然后点击 IAM（身份和访问管理）。添加一个访问类型为程序访问的用户。这将为您提供所需的访问密钥 ID 和秘密访问密钥。

![](img/B06434_17_22.jpg)

创建了您的存储桶，用户权限设置为所有人，并创建了访问密钥，让我们写一些代码。

# 如何做...

1.  我们将创建一个控制台应用程序，将图片上传到之前创建的 S3 存储桶中。首先打开 NuGet 包管理器，并将 AWSSDK NuGet 包添加到您的控制台应用程序中。

您可能值得查看以下链接中的.NET 的 AWS SDK[`aws.amazon.com/sdk-for-net/`](https://aws.amazon.com/sdk-for-net/)。这有助于开发人员快速掌握 SDK。

1.  接下来，创建一个名为`StampysLovelyWorld`的类和一个名为`SaveStampy()`的方法。代码真的没有什么复杂的地方。创建一个指定存储桶区域的客户端对象，创建一个指定要上传的文件、存储桶名称和目录的`TransferUtilityUploadRequest`对象，最后，通过`TransferUtility`将文件上传到存储桶。

AWS 的`RegionEndpoint`枚举为 EU（法兰克福）是`EUCentral1`。请参考 AWS 区域和端点的以下链接[`docs.aws.amazon.com/general/latest/gr/rande.html.`](https://docs.aws.amazon.com/general/latest/gr/rande.html.)

实际上，我们可能会枚举文件夹的内容，甚至允许用户选择多个文件。这个类只是为了说明将文件上传到我们的存储桶的概念。正如您将看到的，这段代码真的很简单。

```cs
        internal static class StampysLovelyWorld
        { 
          public static void SaveStampy(string fileToSave,
                                        string bucket,
                                        string bucketDirectory,
                                        string bucketFilename)
          {
            IAmazonS3 client = AWSClientFactory.CreateAmazonS3Client
                                        (RegionEndpoint.EUCentral1);

            TransferUtility utility = new TransferUtility(client); 
            TransferUtilityUploadRequest request = new 
                                    TransferUtilityUploadRequest();

            request.BucketName = bucket + "/" + bucketDirectory;
            request.Key = bucketFilename; 
            request.FilePath = fileToSave; 
            utility.Upload(request); 
          }
        }

```

1.  在控制台应用程序的`static void Main`方法中，指定您之前创建的存储桶名称，要在存储桶中创建的文件夹以及您想要在 S3 文件夹中的文件名。将这些与文件的路径一起传递给`StampysLovelyWorld`类中的`SaveStampy()`方法。

```cs
        static void Main(string[] args)
        {
          string uploadFile = "C:UsersdirkPicturesSaved 
                               PicturesStampyCat.png";
          string S3Bucket = "familyvaultdocs"; 
          string S3Folder = "MinecraftPictures";
          string uploadedFilename = $"{DateTime.Now.ToString("yyyymmdd")}
                                      - StampyCat.png";
          StampysLovelyWorld.SaveStampy(uploadFile, S3Bucket, S3Folder,
                                        uploadedFilename);
          WriteLine("uploaded");
          ReadLine();
        }

```

1.  我们需要做的最后一件事是将访问密钥和秘密密钥添加到我们控制台应用程序的 App.config 文件中。只需添加一个`<appSettings>`部分，并添加此处列出的密钥。您显然会使用之前在 IAM 中生成的访问密钥和秘密密钥。

```cs
        <?xml version="1.0" encoding="utf-8" ?>
        <configuration>
          <appSettings>
            <add key="AWSProfileName" value="profile1"/>
            <add key="AWSAccessKey" value="AKIAJ6Q2Q77IHJX7STWA"/>
            <add key="AWSSecretKey" value="uFBN6xtuWCSf9zR9WzQKrh1vk
                                           zU2PEuosTTy5qhc"/>
          </appSettings>
          <startup>
            <supportedRuntime version="v4.0" sku=".NETFramework,
                Version=v4.6.2" />
          </startup>
        </configuration>

```

1.  运行您的控制台应用程序。文件上传后，您的控制台应用程序将在输出中显示上传的文本。

# 它是如何工作的...

返回到 AWS 中的`familyvaultdocs`存储桶，并单击欧盟（法兰克福）区域旁边的刷新图标。您将看到您在代码中指定的`MinecraftPictures`文件夹。

![](img/B06434_17_19.jpg)

单击文件夹，您将看到列出的内容。我之前上传了`SquidNugget.png`图像，但我们在代码示例中上传的`StampyCat.png`图像已经根据代码中指定的日期前缀。

![](img/B06434_17_20.jpg)

代码运行并且文件几乎立即被添加。诚然，这些文件并不是很大，但这表明了在 AWS 中添加简单存储服务并将其与.NET 应用程序集成是多么容易。

# 使用 AWS 创建 C# Lambda 函数

2016 年 12 月 1 日，亚马逊宣布 C#现在是 AWS Lambda 支持的语言。因此，这实际上是最新的消息，开发人员可以尝试在.NET 应用程序中使用 AWS Lambda。AWS Lambda 允许您将代码部署到 AWS，而无需担心代码运行的机器，甚至无需担心需求增加时这些机器的扩展。您的代码将正常工作。这对移动开发人员来说非常棒。直到 12 月，AWS Lambda 只支持 Node.js、Pythos 和 Java。让我们看看如何在 Visual Studio 2017 中使用 C#创建 Lambda 函数。

# 准备工作

您需要确保已下载并安装了 Visual Studio 2017 的 AWS Toolkit 预览版。在撰写本文时，工具包可以在以下链接找到：[`aws.amazon.com/blogs/developer/preview-of-the-aws-toolkit-for-visual-studio-2017/`](https://aws.amazon.com/blogs/developer/preview-of-the-aws-toolkit-for-visual-studio-2017/)。

如果您使用的是较早版本的 Visual Studio，请从此链接下载 AWS Toolkit：[`aws.amazon.com/visualstudio/`](https://aws.amazon.com/visualstudio/)。该工具包支持 Visual Studio 2015，并允许您下载 Visual Studio 2010-2012 和 Visual Studio 2008 的旧版本。下载并安装工具包后，您就可以创建您的第一个 AWS Lambda 函数了。

# 如何操作...

1.  启动 Visual Studio 并创建一个新项目。在 Visual C#模板下，您将看到一个名为 AWS Lambda 的新类型。单击 AWS Lambda 项目(.NET Core)模板。没错，这些是.NET Core 应用程序。

![](img/B06434_17_25.jpg)

1.  下一个屏幕将允许我们选择一个蓝图。对于我们的目的，我们将选择一个简单的 S3 函数蓝图，用于响应 S3 事件通知。

![](img/B06434_17_26.jpg)

1.  函数已创建，您的 Visual Studio 中的解决方案资源管理器将如下所示。

![](img/B06434_17_27.jpg)

1.  添加到`Function.cs`文件的代码只是一个具有名为`FunctionHandler()`的方法的类。您还会注意到类顶部的程序集属性如下：`[assembly: LambdaSerializerAttribute(typeof(Amazon.Lambda.Serialization.Json.JsonSerializer))]`。这是必需的，并注册了使用`Newtonsoft.Json`创建我们的类型化类的 Lambda JSON 序列化程序。由于这段代码只是起作用，我不会花太多时间来解释它。

```cs
        public async Task<string> FunctionHandler(S3Event evnt,
                                                ILambdaContext context)
        {
          var s3Event = evnt.Records?[0].S3;
          if(s3Event == null)
          {
            return null;
          }

          try
          {
            var response = await this.S3Client.GetObjectMetadataAsync
                           (s3Event.Bucket.Name, s3Event.Object.Key);
            return response.Headers.ContentType;
          }
          catch(Exception e)
          {
            context.Logger.LogLine($"Error getting object
              {s3Event.Object.Key} from bucket {s3Event.Bucket.Name}.
              Make sure they exist and your bucket is in the same
              region as this function.");
            context.Logger.LogLine(e.Message);
            context.Logger.LogLine(e.StackTrace);
            throw;
          }
        }

```

1.  现在，您可以直接从 Visual Studio 中发布函数到 AWS。右键单击您创建的项目，从上下文菜单中选择发布到 AWS Lambda....

![](img/B06434_17_28.jpg)

1.  现在，您需要完成部署向导。为您的函数命名，如果您没有选择帐户配置文件，请添加一个。

![](img/B06434_17_29.jpg)

对于您的 AWS Lambda 函数，请确保选择与上一篇文章中创建的 S3 存储桶相同的区域。

1.  添加账户配置文件非常简单。这是您在 IAM 中配置的帐户。

![](img/B06434_17_30.jpg)

1.  单击“下一步”将允许您选择为 S3 和我们的函数提供访问权限的 IAM 角色名称。这是在**IAM**（身份和访问管理）中配置的。

![](img/B06434_17_31.jpg)

1.  单击“上传”将函数上传到 AWS。

![](img/B06434_17_32.jpg)

1.  请注意，在这一步可能会遇到几个权限问题。您可能会遇到以下内容：

```cs
Error creating Lambda function: User: arn:aws:iam::932141661806:user/S3Lambda is not authorized to perform: lambda:CreateFunction on resource: arn:aws:lambda:eu-central-1:932141661806:function:S3LambdaFunction

```

实际上，在尝试将函数上传到 AWS 时，您可能会收到几个此类错误。AWS 中的身份和访问管理区域在这里是您的朋友。您应该查看您正在使用的用户（在本例中是 S3Lambda）并审查分配给用户的权限。在这里，错误通知我们，用户 S3Lambda 没有权限在 AWS 上为 S3LambdaFunction 资源创建函数。修改您的权限，然后尝试重新上传。

# 工作原理...

将函数上传到 AWS 后，在 Visual Studio 中单击“查看”菜单，然后选择 AWS 资源管理器。展开 AWS Lambda 节点将显示我们之前上传的函数。如果在展开节点时看到错误，可能需要为您的用户提供 ListFunctions 权限。展开 AWS 身份和访问管理节点还将显示您配置的用户、组和角色。您可以通过选择一个示例请求并单击“调用”按钮在 Visual Studio 中轻松测试 AWS Lambda 函数。

![](img/B06434_17_33.jpg)

然而，我们想要做的是将存储文件的 S3 连接到我们的函数以发送事件。单击“事件源”选项卡，然后单击“添加”按钮。选择 Amazon S3 作为源类型，并选择我们在上一篇文章中创建的`familyvaultdocs`存储桶。完成后，单击“确定”按钮。

![](img/B06434_17_34.jpg)

运行上一篇文章中的控制台应用程序以将新文件上传到我们的 S3 存储桶将触发我们的 Lambda 函数。我们可以通过查看函数视图中的日志部分来确认这一点。

![](img/B06434_17_35.jpg)

您还可以从 AWS 资源管理器上传文件。展开 Amazon S3 节点，然后单击“上传文件”按钮到存储桶。

![](img/B06434_17_36.jpg)

您的文件已上传，并且进度显示在底部的状态窗口中。

![](img/B06434_17_37.jpg)

虽然这个例子并不太复杂（除了权限设置可能有点复杂），但它确实说明了 AWS Lambda 函数的概念。我们可以使用该函数在触发来自 S3 存储桶中的事件等简单事件时执行一系列操作。开始结合功能，您可以创建一个非常强大的无服务器模块，以支持和增强您的应用程序。

无论您使用 AWS、Azure 还是诸如 DocRaptor（或任何其他第三方服务），无服务器计算都将长存下去，C# Lambda 函数将以一种重大的方式改变开发的面貌。
