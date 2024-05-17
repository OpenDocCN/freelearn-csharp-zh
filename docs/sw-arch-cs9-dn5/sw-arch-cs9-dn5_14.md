# 第十四章：使用.NET Core 应用面向服务的架构

“面向服务的架构”（SOA）一词指的是通过通信实现系统组件之间交互的模块化架构。SOA 允许来自不同组织的应用程序自动交换数据和交易，并允许组织在互联网上提供服务。

此外，正如我们在《微服务和模块概念的演变》章节中讨论的那样，《将微服务架构应用于企业应用程序》第五章，基于通信的交互解决了复杂系统中不可避免出现的模块共享相同地址空间的二进制兼容性和版本不匹配问题。此外，使用 SOA，您无需在使用它的各个系统/子系统中部署相同组件的不同副本-每个组件只需在一个地方部署。这可以是单个服务器，位于单个数据中心的集群，或地理分布的集群。在这里，您的组件的每个版本只部署一次，服务器/集群逻辑会自动创建所有必要的副本，从而简化整个持续集成/持续交付（CI/CD）周期。

如果新版本符合向客户端声明的通信接口，则不会发生不兼容性。另一方面，对于 DLLs/软件包，当保持相同接口时，可能会出现不兼容性，因为库模块可能与其客户端共享的其他 DLLs/软件包的依赖关系可能存在版本不匹配的情况。

在《将微服务架构应用于企业应用程序》第五章中讨论了组织协作服务的集群/网络。在本章中，我们将主要关注每个服务提供的通信接口。更具体地，我们将讨论以下主题：

+   理解 SOA 方法的原则

+   SOAP 和 REST Web 服务

+   .NET 5 如何处理 SOA？

+   用例-公开 WWTravelClub 套餐

通过本章结束时，您将了解如何通过 ASP.NET Core 服务公开 WWTravelClub 书籍用例中的数据。

# 技术要求

本章需要安装了所有数据库工具的 Visual Studio 2019 免费社区版或更高版本。

本章中的所有概念都将通过本书的 WWTravelClub 书籍用例的实际示例加以阐明。您可以在以下网址找到本章的代码：[`github.com/PacktPublishing/Software-Architecture-with-C-9-and-.NET-5`](https://github.com/PacktPublishing/Software-Architecture-with-C-9-and-.NET-5)。

# 理解 SOA 方法的原则

与面向对象架构中的类一样，服务是接口的实现，而接口又来自系统的功能规范。因此，“服务”设计的第一步是定义其“抽象接口”。在此阶段，您将定义所有服务操作，作为操作您喜欢的语言类型（C＃，Java，C ++，JavaScript 等）的接口方法，并决定使用同步通信实现哪些操作，使用异步通信实现哪些操作。

在这个初始阶段定义的接口不一定会在实际服务实现中使用，它们只是有用的设计工具。一旦我们决定了服务的架构，通常会重新定义这些接口，以便我们可以将它们适应架构的特殊性。

值得指出的是，SOA 消息必须保持与方法调用/回答相同类型的语义；也就是说，对消息的反应不能依赖于先前接收到的任何消息。在这里，消息必须彼此独立，服务*不能记住*任何先前接收到的消息。这就是我们所说的无状态开发。

例如，如果消息的目的是创建新的数据库条目，这种语义不能随着其他消息的上下文而改变，创建数据库条目的方式必须取决于当前消息的内容，而不是先前接收到的其他消息。因此，客户端不能创建会话，也不能登录到服务，执行一些操作，然后注销。认证令牌必须在每条消息中重复。

这种约束的原因是模块化、可测试性和可维护性。事实上，基于会话的服务将非常难以测试和修改，因为这些交互是*隐藏*在会话数据中的。

一旦您决定要由服务实现的接口，您必须决定采用哪种通信堆栈/SOA 架构。通信堆栈必须是某些官方或*事实*标准的一部分，以确保服务的互操作性。互操作性是 SOA 规定的主要约束：服务必须提供一个通信接口，不依赖于特定的通信库、实现语言或部署平台。

考虑到您已经决定了通信堆栈/架构，您需要根据架构的特点调整先前的接口（有关更多详细信息，请参阅本章的*REST Web 服务*子部分）。然后，您必须将这些接口翻译成所选择的通信语言。这意味着您必须将所有编程语言类型映射到所选择的通信语言中可用的类型。

数据的实际翻译通常由您的开发环境使用的 SOA 库自动执行。然而，可能需要一些配置，无论如何，我们必须意识到在每次通信之前我们的编程语言类型是如何转换的。例如，一些数字类型可能会被转换为精度较低或具有不同值范围的类型。

互操作性约束在微服务的情况下可以以较轻的形式解释，因为这些微服务无法在其集群之外访问，因此它们需要与属于同一集群的其他微服务进行通信。在这种情况下，这意味着通信堆栈可能是特定于平台的，以提高性能，但必须是标准的，以避免与随着应用程序发展可能添加到集群中的其他微服务的兼容性问题。

我们谈论的是*通信堆栈*而不是*通信协议*，因为 SOA 通信标准通常定义了消息内容的格式，并为用于嵌入这些消息的特定协议提供了不同的可能性。例如，SOAP 协议只是定义了各种消息的基于 XML 的格式，但 SOAP 消息可以通过各种协议传递。通常，用于 SOAP 的最常见协议是 HTTP，但您可以决定跳转到 HTTP 级别，并直接通过 TCP/IP 发送 SOAP 消息以获得更好的性能。

您应该采用的通信堆栈的选择取决于几个因素：

+   **兼容性约束**：如果您的服务必须对业务客户在互联网上公开可用，那么您必须遵守最常见的选择，这意味着使用 SOAP over HTTP 或 JSON REST 服务。如果您的客户不是业务客户而是**物联网**（**IoT**）客户，则最常见的选择是不同的。此外，在物联网中，不同应用领域使用的协议可能不同。例如，海洋车辆状态数据通常不与*Signal K*交换。

+   **开发/部署平台**：并非所有通信堆栈都适用于所有开发框架和所有部署平台，但幸运的是，所有主要的开发/部署平台上都可以使用在公共业务服务中使用的所有最常见的通信堆栈，例如 SOAP 和基于 JSON 的 REST 通信。

+   **性能**：如果您的系统不向外部公开，而是微服务集群的私有部分，性能考虑将具有更高的优先级。在这种情况下，我们将在本章后面讨论的 gRPC 可以被提及为一个不错的选择。

+   **团队中工具和知识的可用性**：在选择可接受的通信堆栈时，团队/组织中的知识和工具的可用性具有重要的影响。然而，这种约束始终比兼容性约束的优先级低，因为构想一个对团队易于实现但几乎没有人能使用的系统是毫无意义的。

+   **灵活性与可用功能**：一些通信解决方案虽然不够完整，但提供了更高程度的灵活性，而另一些解决方案虽然更完整，但提供的灵活性较少。在过去几年中，对灵活性的需求促使从基于 SOAP 的服务转向更灵活的 REST 服务。在本节的其余部分中，我们将更详细地讨论 SOAP 和 REST 服务。

+   **服务描述**：当服务必须在互联网上公开时，客户端应用程序需要一个公开可用的服务规范描述，以设计其通信客户端。一些通信堆栈包括用于描述服务规范的语言和约定。以这种方式公开的正式服务规范可以被处理，以便它们自动创建通信客户端。SOAP 进一步允许通过包含每个 Web 服务可以执行的任务信息的公共基于 XML 的目录来发现服务。

选择要使用的通信堆栈后，必须使用开发环境中可用的工具来实现符合所选通信堆栈的服务。有时，开发工具会自动确保通信堆栈的符合性，但有时可能需要一些开发工作。例如，在.NET 世界中，如果使用 WCF，则开发工具会自动确保 SOAP 服务的符合性，而 REST 服务的符合性则由开发人员负责。SOA 解决方案的一些基本特性如下：

+   **身份验证**：允许客户端进行身份验证以访问服务操作。

+   **授权**：处理客户端的权限。

+   **安全性**：这是通信如何保持安全的方式，即如何防止未经授权的系统读取和/或修改通信内容。通常，加密可以防止未经授权的修改和阅读，而电子签名算法只能防止修改。

+   **异常**：向客户返回异常。

+   **消息可靠性**：确保在可能的基础设施故障的情况下，消息能够可靠地到达目的地。

虽然有时是可取的，但以下特性并不总是必要的：

+   **分布式事务**：处理分布式事务的能力，因此当分布式事务失败或中止时，撤消您所做的所有更改。

+   **支持发布者/订阅者模式**：事件和通知的支持方式。

+   **寻址**：引用其他服务和/或服务/方法的支持方式。

+   **路由**：消息如何通过服务网络进行路由。

本节的其余部分致力于描述 SOAP 和 REST 服务，因为它们是在集群/服务器之外公开的业务服务的*事实*标准。出于性能原因，微服务使用其他协议，在*第五章*，*将微服务架构应用于企业应用程序*；*第六章*，*Azure 服务布局*；和*第七章*，*Azure Kubernetes 服务*中进行了讨论。对于集群间通信，使用**高级消息队列协议**（**AMQP**），在*进一步阅读*部分给出了链接。

# SOAP Web 服务

**简单对象访问协议**（**SOAP**）允许单向消息和答复/响应消息。通信可以是同步的也可以是异步的，但是，如果底层协议是同步的，比如在 HTTP 的情况下，发送者会收到一个确认消息，说明消息已经被接收（但不一定被处理）。当使用异步通信时，发送者必须监听传入的通信。通常，异步通信是使用我们在*第十一章*，*设计模式和.NET 5 实现*中描述的发布者/订阅者模式来实现的。

消息被表示为称为**信封**的 XML 文档。每个信封包含一个`头部`，一个`主体`和一个`故障`元素。`主体`是消息的实际内容所在。`故障`元素包含可能的错误，因此在通信发生时，这是交换异常的方式。最后，`头部`包含丰富协议的任何辅助信息，但不包含域数据。例如，`头部`可能包含身份验证令牌，和/或如果消息被签名的话。

用于发送 XML 信封的底层协议通常是 HTTP，但 SOAP 规范允许任何协议，因此我们可以直接使用 TCP/IP 或 SMTP。事实上，更为普遍的底层协议是 HTTP，因此，如果您没有选择其他协议的充分理由，应该使用 HTTP 以最大化服务的互操作性。

SOAP 规范包含消息交换的基础知识，而其他辅助功能则在称为`WS- *`的单独规范文档中描述，并通常通过在 SOAP 头部添加额外信息来处理。`WS-*`规范处理我们之前列出的 SOA 的所有基本和理想特性。例如，`WS-Security`负责安全性，包括身份验证、授权和加密/签名；`WS-Eventing`和`WS-Notification`是实现发布者/订阅者模式的两种替代方式；`WS-ReliableMessaging`关注在可能出现故障时消息的可靠传递；`WS-Transaction`关注分布式事务。

前述的`WS-*`规范并不是详尽无遗的，但是它们是更相关和受支持的特性。实际上，各种环境中的实际实现（如 Java 和.NET）提供了更相关的`WS-*`服务，但没有一种实现支持所有的`WS-*`规范。

SOAP 协议涉及的所有 XML 文档/文档部分在 XSD 文档中得到正式定义，这些文档是特殊的 XML 文档，其内容提供了 XML 结构的描述。此外，所有自定义数据结构（面向对象语言中的类和接口）必须被转换为 XSD，如果它们将成为 SOAP 信封的一部分。

每个 XSD 规范都有一个关联的`命名空间`，用于标识规范和可以找到规范的物理位置。命名空间和物理位置都是 URI。如果 Web 服务只能从内部网络访问，位置 URI 不需要公开访问。

服务的整个定义是一个可能包含对其他命名空间的引用的 XSD 规范，也就是说，对其他 XSD 文档的引用。简而言之，SOAP 通信的所有消息必须在 XSD 规范中定义。然后，如果服务器和客户端引用相同的 XSD 规范，它们可以进行通信。这意味着，例如，每次向消息添加另一个字段时，您需要创建一个新的 XSD 规范。之后，您需要更新所有引用旧消息定义的 XSD 文件到新消息定义，通过创建它们的新版本。反过来，这些修改需要为其他 XSD 文件创建其他版本，依此类推。因此，保持与先前行为兼容的简单修改（客户端可以简单地忽略添加的字段）可能会导致指数级的版本更改链。

在过去几年中，处理修改的困难以及处理所有`WS-*`规范的配置和性能问题，导致了逐渐向我们将在接下来的部分中描述的更简单的 REST 服务的转变。这一转变始于从 JavaScript 调用的服务，因为在 Web 浏览器中实现完整的 SOAP 客户端的困难。此外，复杂的 SOAP 机制对于运行在浏览器中的典型客户端的简单需求来说过于庞大，可能导致开发时间的完全浪费。

因此，面向非 JavaScript 客户端的服务开始大规模转向 REST 服务，如今首选的选择是 REST 服务，而 SOAP 则用于与遗留系统兼容或者在需要 REST 服务不支持的功能时使用。继续偏好 SOAP 系统的典型应用领域是支付/银行系统，因为这些系统需要`WS-Transaction` SOAP 规范提供的事务支持。在 REST 服务领域没有相应的功能。

# REST 网络服务

REST 服务最初被构想为避免在简单情况下使用 SOAP 的复杂机制，比如从网页的 JavaScript 代码调用服务。然后，它们逐渐成为复杂系统的首选。REST 服务使用 HTTP 以 JSON 或者更少见的 XML 格式交换数据。简而言之，它们用 HTTP 主体替换 SOAP 主体，用 HTTP 头部替换 SOAP 头部，HTTP 响应代码替换故障元素，并提供有关执行的操作的进一步辅助信息。

REST 服务成功的主要原因是，HTTP 本身已经提供了大部分 SOAP 功能，这意味着我们可以避免在 HTTP 之上构建 SOAP 级别。此外，整个 HTTP 机制比 SOAP 更简单：编程更简单，配置更简单，而且更容易高效地实现。

此外，REST 服务对客户端的约束更少。服务器和客户端之间的类型兼容性符合更灵活的 JavaScript 类型兼容性模型，因为 JSON 是 JavaScript 的子集。此外，当使用 XML 代替 JSON 时，它保持相同的 JavaScript 类型兼容性规则。不需要指定 XML 命名空间。

在使用 JSON 和 XML 时，如果服务器在响应中添加了一些新字段，同时保持与先前客户端的所有其他字段兼容的语义，他们可以简单地忽略新字段。因此，在服务器中引起实际不兼容行为的破坏性更改的情况下，对 REST 服务定义所做的更改只需要传播到先前的客户端。

此外，由于类型兼容性不要求引用特定类型在唯一共享的位置定义，并且只需要类型的形状是兼容的，因此更改可能是自限制的，不会导致指数级的更改链。

## 服务类型兼容性规则

让我们通过一个例子来澄清 REST 服务类型兼容性规则。假设有几个服务使用包含“姓名”、“姓氏”和“地址”字符串字段的`Person`对象。这个对象由**S1**提供：

```cs
{
    Name: string,
    Surname: string,
    Address: string
} 
```

确保类型兼容性，如果服务和客户端引用前面定义的不同副本。客户端使用字段较少的定义也是可以接受的，因为它可以简单地忽略所有其他字段：

```cs
{
    Name: string,
    Surname: string,
} 
```

您只能在“自己”的代码中使用字段较少的定义。尝试在没有预期字段的情况下将信息发送回服务器可能会导致问题。

现在，想象一下这样的情景：您有一个**S2**服务，它从**S1**获取`Person`对象，并将它们添加到其某些方法返回的响应中。假设处理`Person`对象的**S1**服务用复杂对象替换了“地址”字符串：

```cs
{
    Name: string,
    Surname: string,
    Address:
        {
            Country: string,
            Town: string,
            Location: string
        }
} 
```

在破坏性更改之后，**S2**服务将不得不调整其调用**S1**服务的通信客户端以适应新格式。然后，它可以在使用`Person`对象在其响应中之前将新的`Person`格式转换为旧的格式。这样，**S2**服务避免了传播**S1**的破坏性更改。

总的来说，基于对象形状（嵌套属性树）而不是对相同正式类型定义的引用来确定类型兼容性，增加了灵活性和可修改性。我们为这种增加的灵活性付出的代价是，类型兼容性不能通过比较服务器和客户端接口的正式定义来自动计算。事实上，在没有明确规范的情况下，每次发布服务的新版本时，开发人员必须验证客户端和服务器共有的所有字段的语义是否与上一个版本保持不变。

REST 服务背后的基本思想是放弃严格的检查和复杂的协议，以换取更大的灵活性和简单性，而 SOAP 恰恰相反。

## Rest 和本机 HTTP 功能

REST 服务宣言指出，REST 使用本机 HTTP 功能来实现所有所需的服务功能。因此，例如，身份验证将直接使用 HTTP 的`Authorization`字段进行，加密将通过 HTTPS 实现，异常将使用 HTTP 错误状态代码处理，路由和可靠的消息传递将由 HTTP 协议依赖的机制处理。通过使用 URL 来引用服务、它们的方法和其他资源来实现寻址。

由于 HTTP 是同步协议，因此没有原生支持异步通信。也没有原生支持发布者/订阅者模式，但两个服务可以通过各自公开一个端点来与发布者/订阅者模式进行交互。更具体地，第一个服务公开一个订阅端点，而第二个服务公开一个端点，用于接收其通知，这些通知通过在订阅期间交换的共同秘密进行授权。这种模式非常常见。GitHub 还允许我们将我们的 REST 服务发送到存储库事件。

REST 服务在实现分布式事务时没有简单的选项，这就是为什么支付/银行系统仍然更喜欢 SOAP。幸运的是，大多数应用领域不需要分布式事务所确保的强一致性。对于它们来说，轻量级的一致性形式，如*最终一致性*，已经足够，并且出于性能原因更受欢迎。请参阅*第九章*，*如何在云中选择您的数据存储*，讨论各种一致性类型。

REST 宣言不仅规定了在 HTTP 中使用预定义的解决方案，还规定了使用类似 Web 的语义。更具体地说，所有服务操作必须被构想为对由 URL 标识的资源进行 CRUD 操作（同一资源可以由多个 URL 标识）。事实上，REST 是**表现状态转移**的缩写，意味着每个 URL 都是某种对象的表示。每种服务请求都需要采用适当的 HTTP 动词，如下所示：

+   `GET`（读取操作）：URL 代表读取操作返回的资源。因此，`GET`操作模拟指针解引用。在操作成功的情况下，将返回 200（OK）状态码。

+   `POST`（创建操作）：包含在请求体中的 JSON/XML 对象被添加为操作 URL 所代表的对象的新资源。如果新资源立即成功创建，将返回 201（已创建）状态码，以及取决于操作的响应对象和关于可以从哪里检索到创建的资源的指示。响应对象应包含标识创建的资源的最具体 URL。如果创建推迟到以后的时间，将返回 202（已接受）状态码。

+   `PUT`（编辑操作）：请求体中包含的 JSON/XML 对象替换了请求 URL 引用的对象。在操作成功的情况下，将返回 200（OK）状态码。这个操作是幂等的，也就是说重复相同的请求两次会导致相同的修改。

+   `PATCH`：请求体中包含的 JSON/XML 对象包含如何修改请求 URL 引用的对象的指令。这个操作不是幂等的，因为修改可能是对数值字段的增量。在操作成功的情况下，将返回 200（OK）状态码。

+   `DELETE`：删除请求 URL 引用的资源。在操作成功的情况下，将返回 200（OK）状态码。

如果资源已从请求 URL 移动到另一个 URL，将返回重定向代码：

+   `301`（永久移动），以及我们可以找到资源的新 URL

+   `307`（临时移动），以及我们可以找到资源的新 URL

如果操作失败，将返回取决于失败类型的状态码。一些失败代码的示例如下：

+   `400`（错误的请求）：发送到服务器的请求格式不正确。

+   `404`（未找到）：当请求 URL 不引用任何已知对象时。

+   `405`（方法不允许）：当 URL 引用的资源不支持请求动词时。

+   `401`（未经授权）：操作需要身份验证，但客户端未提供任何有效的授权头。

+   `403`（禁止）：客户端已正确进行身份验证，但没有执行操作的权限。

上述状态码列表并非详尽无遗。详尽列表的参考将在*进一步阅读*部分提供。

需要指出的是，`POST`/`PUT`/`PATCH`/`DELETE`操作可能具有对其他资源的副作用，通常也会有副作用。否则，将无法编写同时对多个资源进行操作的操作。

换句话说，HTTP 动词必须符合在请求 URL 引用的资源上执行的操作，但该操作可能会影响其他资源。同一操作可能会使用不同的 HTTP 动词在其他涉及的资源中执行。开发人员有责任选择以哪种方式执行相同的操作来实现服务接口。

由于 HTTP 动词的副作用，REST 服务可以将所有这些操作编码为 URL 表示的资源上的 CRUD 操作。

通常，将现有服务转换为 REST 需要我们在请求 URL 和请求主体之间分割各种输入。更具体地说，我们提取唯一定义方法执行中涉及的对象之一的输入字段，并使用它们创建一个唯一标识该对象的 URL。然后，我们根据在所选对象上执行的操作选择要使用的 HTTP 动词。最后，我们将其余的输入放在请求主体中。

如果我们的服务是以面向业务域对象的面向对象架构设计的（例如 DDD，如*第十二章*中所述，*理解软件解决方案中的不同领域*），那么所有服务方法的 REST 翻译应该是相当直接的，因为服务应该已经围绕领域资源组织起来。否则，转移到 REST 可能需要重新定义一些服务接口。

采用完整的 REST 语义的优势在于服务可以在不对现有操作定义进行小修改的情况下进行扩展。事实上，扩展主要表现为某些对象的附加属性和一些相关操作的附加资源 URL。因此，现有客户端可以简单地忽略它们。

## REST 语言中方法的示例

现在，让我们通过一个简单的银行内部转账的示例来学习如何在 REST 语言中表达方法。银行账户可以通过以下 URL 表示：

```cs
https://mybank.com/bankaccounts/{bank account number} 
```

转账可以表示为一个`PATCH`请求，其主体包含一个表示金额、转账时间、描述和接收资金的账户的属性对象。

该操作修改了 URL 中提到的账户，同时也会影响到接收账户。如果账户没有足够的资金，将返回 403（禁止）状态码，以及一个包含所有错误细节的对象（错误描述、可用资金等）。

然而，由于所有的银行操作都记录在账单中，为与银行账户相关的*bank account operations*集合创建和添加一个新的转账对象是更好的表示转账的方式。在这种情况下，URL 可能是以下内容：

```cs
https://mybank.com/bankaccounts/{bank account number}/transactions 
```

在这里，HTTP 动词是`POST`，因为我们正在创建一个新对象。主体内容是相同的，如果资金不足，则返回`422`状态码。

转账的两种表示都会导致数据库中的相同更改。此外，一旦输入从不同的 URL 和可能不同的请求主体中提取出来，后续处理是相同的。在这两种情况下，我们有相同的输入和相同的处理 - 只是这两个请求的外观不同。

然而，引入虚拟的*operations*集合使我们能够通过几种特定于*operations*集合的方法来扩展服务。值得指出的是*operations*集合不需要与数据库表或任何物理对象连接：它存在于 URL 的世界中，并为我们建模转账提供了便利的方式。

REST 服务的增加使用导致了 REST 服务接口的描述被创建，就像为 SOAP 开发的那样。这个标准称为 OpenAPI。我们将在下一小节中讨论这个问题。

## OpenAPI 标准

OpenAPI 是用于描述 REST API 的标准。目前是版本 3。整个服务由 JSON 端点描述，即用 JSON 对象描述服务的端点。这个 JSON 对象有一个适用于整个服务的一般部分，包含服务的一般特性，如其版本和描述，以及共享定义。

然后，每个服务端点都有一个特定部分，描述端点的 URL 或 URL 格式（如果 URL 中包含一些输入），所有的输入，所有可能的输出类型和状态码，以及所有的授权协议。每个特定端点部分可以引用一般部分中包含的定义。

本书不涵盖 OpenAPI 语法的描述，但在“进一步阅读”部分提供了参考资料。各种开发框架通过处理 REST API 代码自动生成 OpenAPI 文档，并由开发人员提供更多信息，因此您的团队不需要深入了解 OpenAPI 语法。其中一个例子是我们将在本章中介绍的`Swashbuckle.AspNetCore` NuGet 包。

“.NET 5 如何处理 SOA？”部分解释了我们如何在 ASP.NET Core REST API 项目中自动生成 OpenAPI 文档，而本章末尾的用例提供了其使用的实际示例。

我们将通过讨论如何处理 REST 服务中的身份验证和授权来结束本小节。

## REST 服务授权和身份验证

由于 REST 服务是无状态的，当需要身份验证时，客户端必须在每个请求中发送身份验证令牌。该令牌通常放在 HTTP 授权标头中，但这取决于您使用的身份验证协议的类型。通过显式传输共享密钥是进行身份验证的最简单方式。可以使用以下代码来实现：

```cs
Authorization: Api-Key <string known by both server and client> 
```

共享密钥称为 API 密钥。由于在撰写本文时，尚无关于如何发送 API 密钥的标准，因此 API 密钥也可以在其他标头中发送，如下面的代码所示：

```cs
X-API-Key: <string known by both server and client> 
```

值得一提的是，基于 API 密钥的身份验证需要使用 HTTPS 来阻止共享密钥被窃取。API 密钥非常简单易用，但它们不传达有关用户授权的信息，因此当客户端允许的操作非常标准且没有复杂的授权模式时，可以采用它们。此外，在请求中交换 API 密钥时，API 密钥容易受到服务器或客户端的攻击。缓解这种情况的常见模式是创建一个“服务账户”用户，并将其授权限制在所需的范围内，并在与 API 交互时使用该特定账户的 API 密钥。

更安全的技术使用长期有效的共享密钥，用户登录后即可使用。然后，登录返回一个短期令牌，该令牌在所有后续请求中用作共享密钥。当短期密钥即将过期时，可以通过调用续订端点来进行续订。

整个逻辑与短期令牌身份验证逻辑完全解耦。登录通常基于接收长期凭据并返回短期令牌的登录端点。登录凭据可以是传递给登录方法的常规用户名密码对，也可以是其他类型的授权令牌，这些令牌被转换为由登录端点提供的短期令牌。登录还可以通过基于 X.509 证书的各种身份验证协议实现。

最常见的短寿命令牌类型是所谓的持有者令牌。每个持有者令牌都编码了它的持续时间以及一系列断言，称为声明，可用于授权目的。持有者令牌由登录操作或续订操作返回。它们的特征是它们不与接收它们的客户端或任何其他特定客户端绑定。

无论客户端如何获得持有者令牌，这都是客户端需要被其声明隐含的所有权利授予的。只需将持有者令牌转让给另一个客户端，即可授予该客户端所有持有者令牌声明隐含的所有权利，因为基于持有者令牌的授权不需要身份的证明。

因此，一旦客户端获得了一个持有者令牌，它可以通过将其持有者令牌转让给第三方来委托一些操作。通常，在必须使用持有者令牌进行委托时，在登录阶段，客户端指定要包含的声明以限制令牌可以授权的操作。

与 API 密钥身份验证相比，基于持有者令牌的身份验证受到标准的约束。它们必须使用以下`Authorization`标头：

```cs
Authorization: Bearer <bearer token string> 
```

持有者令牌可以以多种方式实现。REST 服务通常使用 JWT 令牌，该令牌是由 JSON 对象的 Base64URL 编码串联而成。更具体地说，JWT 的创建始于 JSON 标头，以及 JSON 负载。JSON 标头指定了令牌的类型以及如何签名，而负载由一个包含所有声明的 JSON 对象作为属性/值对组成。以下是一个示例标头：

```cs
{
  "alg": "RS256",
  "typ": "JWT"
} 
```

以下是一个示例负载：

```cs
{
  "iss": "wwtravelclub.com"
  "sub": "example",
  "aud": ["S1", "S2"],
  "roles": [
    "ADMIN",
    "USER"
  ],
  "exp": 1512975450,
  "iat": 1512968250230
} 
```

然后，标头和负载被 Base64URL 编码，并且相应的字符串连接如下：

```cs
<header BASE64 string>.<payload base64 string> 
```

然后，使用标头中指定的算法对前面的字符串进行签名，例如，在我们的示例中，是 RSA + SHA256，并将签名字符串与原始字符串连接如下：

```cs
<header BASE64 string>.<payload base64 string>.<signature string> 
```

前面的代码是最终的持有者令牌字符串。可以使用对称签名代替 RSA，但在这种情况下，JWT 颁发者和所有使用它进行授权的服务必须共享一个公共密钥，而在 RSA 中，JWT 颁发者的私钥不需要与任何人共享，因为签名可以仅通过颁发者公钥进行验证。

一些负载属性是标准的，比如以下内容：

+   `iss`：JWT 的颁发者。

+   `aud`：受众，即可以使用令牌进行授权的服务和/或操作。如果服务在此列表中看不到其标识符，则应拒绝令牌。

+   `sub`：标识 JWT 颁发给的*主体*（即用户）的字符串。

+   `iat`，`exp`和`nbf`：这些是 JWT 颁发的时间，其过期时间，以及如果设置了，令牌有效的时间之后。所有时间都表示为从 1970 年 1 月 1 日 UTC 午夜开始的秒数。在这里，所有天都被认为是确切地有 86400 秒。

如果我们用唯一的 URI 表示，其他声明可以被定义为公共的；否则，它们被认为是颁发者和已知服务的私有声明。

# .NET 5 如何处理 SOA？

WCF 技术尚未移植到.NET 5，并且没有计划对其进行完全移植。相反，微软正在投资于 gRPC，谷歌的开源技术。此外，.NET 5 通过 ASP.NET Core 对 REST 服务有出色的支持。

.NET 5 放弃 WCF 的主要原因如下：

+   正如我们已经讨论过的，SOAP 技术在大多数应用领域已被 REST 技术取代。

+   WCF 技术严格绑定在 Windows 上，因此在.NET 5 中从头开始重新实现其所有功能将非常昂贵。由于对完整.NET 的支持将继续，需要 WCF 的用户仍然可以依赖它。

+   作为一般策略，微软更倾向于投资于可以与其他竞争对手共享的开源技术。这就是为什么微软在.NET Core 3.0 开始提供了 gRPC 实现，而不是投资于 WCF。

下面的小节将介绍 Visual Studio 为我们提到的每种技术提供的支持。

## SOAP 客户端支持

在 WCF 中，服务规范是通过.NET 接口定义的，实际的服务代码是在实现这些接口的类中提供的。端点、底层协议（HTTP 和 TCP/IP）以及任何其他特性都在配置文件中定义。配置文件可以通过易于使用的配置工具进行编辑。因此，开发人员只需提供标准的.NET 类作为服务行为，并以声明方式配置所有服务特性。这样，服务配置完全与实际的服务行为解耦，每个服务都可以重新配置，以适应不同的环境，而无需修改其代码。

虽然.NET 5 不支持 SOAP 技术，但它支持 SOAP 客户端。更具体地说，在 Visual Studio 中为现有的 SOAP 服务创建 SOAP 服务代理非常容易（请参阅*第十一章*，*设计模式和.NET 5 实现*，讨论代理是什么以及代理模式）。

在服务的情况下，代理是实现服务接口的类，其方法通过调用远程服务的类似方法来执行它们的工作。

要创建服务代理，在**解决方案资源管理器**中的项目中右键单击**依赖项**，然后选择**添加连接的服务**。然后，在出现的表单中，选择**Microsoft WCF 服务引用提供程序**。在那里，您可以指定服务的 URL（包含 WSDL 服务描述的位置）、要添加代理类的命名空间等。在向导结束时，Visual Studio 会自动添加所有必要的 NuGet 包并生成代理类。这就足以创建此类的实例并调用其方法，以便与远程 SOAP 服务进行交互。

还有第三方，如 NuGet 包，提供了对 SOAP 服务的有限支持，但目前它们并不是很有用，因为这种有限支持不包括在 REST 服务中不可用的功能。

## gRPC 支持

Visual Studio 2019 支持 gRPC 项目模板，可以为 gRPC 服务器和 gRPC 客户端生成脚手架。gRPC 实现了远程过程调用模式，提供了同步和异步调用，减少了客户端和服务器之间的消息流量。

尽管在撰写本书时，gRPC 在 Azure 的 IIS 和应用服务中不可用，但与此相关的伟大倡议。其中之一是 gRPC-Web（[`devblogs.microsoft.com/aspnet/grpc-web-for-net-now-available/`](https://devblogs.microsoft.com/aspnet/grpc-web-for-net-now-available/)）。

它的配置方式类似于 WCF 和.NET 远程调用，正如我们在*第六章*，*Azure Service Fabric*末尾所描述的那样。也就是说，服务是通过接口定义的，它们的代码是在实现这些接口的类中提供的，而客户端通过实现相同的服务接口的代理与这些服务进行交互。

gRPC 是微服务集群内部通信的一个很好的选择，特别是如果集群不完全基于 Service Fabric 技术，并且不能依赖.NET 远程调用。由于所有主要语言和开发框架都有 gRPC 库，因此它可以在基于 Kubernetes 的集群中使用，以及在托管了在其他框架中实现的 Docker 镜像的 Service Fabric 集群中使用。

由于 gRPC 对数据的更紧凑表示以及更易于使用，因为协议的所有内容都由开发框架处理，所以 gRPC 比 REST 服务协议更有效。然而，在撰写本文时，它的特性都不依赖于成熟的标准，因此不能用于公开的端点 - 它只能用于集群内部通信。因此，我们不会详细描述 gRPC，但本章的*进一步阅读*部分包含了对 gRPC 的一般参考以及其.NET Core 实现的引用。

使用 gRPC 非常简单，因为 Visual Studio 的 gRPC 项目模板会自动生成 gRPC 服务和其客户端的所有内容。开发人员只需定义特定于应用程序的 C#服务接口和实现它的类。

您可以在[`docs.microsoft.com/en-us/aspnet/core/tutorials/grpc/grpc-start?view=aspnetcore-5.0`](https://docs.microsoft.com/en-us/aspnet/core/tutorials/grpc/grpc-start?view=aspnetcore-5.0)上查看有关此实现的详细信息。

本节的其余部分专门介绍了.NET Core 对 REST 服务的支持，包括服务器端和客户端。

## ASP.NET Core 简介

ASP.NET Core 应用程序是基于我们在“使用通用主机”子章节中描述的*主机*概念的.NET Core 应用程序，该子章节位于*第五章*的*将微服务架构应用于企业应用程序*中。每个 ASP.NET 应用程序的`program.cs`文件都会创建一个主机，构建它，并使用以下代码运行它：

```cs
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }
    public static IHostBuilder CreateHostBuilder(string[] args) => 
        Host
        .CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
} 
```

`CreateDefaultBuilder`设置了一个标准主机，而`ConfigureWebHostDefaults`配置了它，以便它可以处理 HTTP 管道。更具体地说，它为当前目录设置了`IWebHostEnvironment`接口的`ContentRootPath`属性。

然后，它从`appsettings.json`和`appsettings.[EnvironmentName].json`中加载配置信息。一旦加载，JSON 对象属性中包含的配置信息可以使用 ASP.NET Core 选项框架映射到.NET 对象属性。更具体地说，`appsettings.json`和`appsettings.[EnvironmentName].json`被合并，文件的特定于环境的信息会覆盖相应的`appsettings.json`设置。

`EnvironmentName`取自`ASPNETCORE_ENVIRONMENT`环境变量。反过来，当应用程序在 Visual Studio 中运行时，它在`Properties\launchSettings.json`文件中定义，位于**解决方案资源管理器**上方。在此文件中，您可以定义几个可以使用下拉菜单选择的环境，该下拉菜单位于 Visual Studio 运行按钮**IIS Express**旁边。默认情况下，**IIS Express**设置将`ASPNETCORE_ENVIRONMENT`设置为`Development`。以下是一个典型的`launchSettings.json`文件：

```cs
{
  "iisSettings": {
    "windowsAuthentication": false, 
    "anonymousAuthentication": true, 
    "iisExpress": {
      "applicationUrl": "http://localhost:2575",
      "sslPort": 44393
    }
  },
  "profiles": {
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    ...
    ...
    }
  }
} 
```

在应用程序发布时，可以在 Visual Studio 创建的发布 XML 文件中添加要使用的`ASPNETCORE_ENVIRONMENT`值。该值为`<EnvironmentName>Staging</EnvironmentName>`。它也可以在您的 Visual Studio ASP.NET Core 项目文件（`.csproj`）中指定：

```cs
<PropertyGroup>
<EnvironmentName>Staging</EnvironmentName>
</PropertyGroup> 
```

随后，应用程序配置主机日志记录，以便可以将日志写入控制台和调试输出。此设置可以通过进一步的配置进行更改。然后，它设置/连接 Web 服务器到 ASP.NET Core 管道。

当应用程序在 Linux 上运行时，ASP.NET Core 管道连接到.NET Core Kestrel Web 服务器。由于 Kestrel 是一个最小的 Web 服务器，您需要负责从完整的 Web 服务器（如 Apache 或 NGINX）进行反向代理请求，以添加 Kestrel 没有的功能。当应用程序在 Windows 上运行时，默认情况下，`ConfigureWebHostDefaults`将 ASP.NET Core 管道直接连接到**Internet Information Services**（**IIS**）。但是，您也可以在 Windows 中使用 Kestrel，并且可以通过更改 Visual Studio 项目文件的`AspNetCoreHostingModel`设置将 IIS 请求反向代理到 Kestrel。

```cs
<PropertyGroup>
    ...
<AspNetCoreHostingModel>OutOfProcess</AspNetCoreHostingModel>
</PropertyGroup> 
```

`UseStartup<Startup>()`允许主机服务（参见*第五章*的*使用通用主机*子章节，将 ASP.NET Core 管道的定义从项目的`Startup.cs`类的方法中获取。更具体地说，服务在其`ConfigureServices(IServiceCollection services)`方法中定义，而 ASP.NET Core 管道在`Configure`方法中定义。以下代码显示了使用 API REST 项目生成的标准`Configure`方法：

```cs
public void Configure(IApplicationBuilder app, 
    IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    app.UseHsts();
    app.UseHttpsRedirection();
    app.UseRouting();
    app.UseAuthorization();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
} 
```

管道中的每个中间件都由一个`app.Use<something>`方法定义，通常接受一些选项。它们中的每一个都处理请求，然后要么将修改后的请求转发到管道中的下一个中间件，要么返回 HTTP 响应。当返回 HTTP 响应时，它将按相反的顺序由所有先前的中间件处理。

模块按照它们被`app.Use<something>`方法调用定义的顺序插入到管道中。前面的代码在`ASPNETCORE_ENVIRONMENT`为`Development`时添加了一个错误页面；否则，`UseHsts`与客户端协商安全协议。最后，`UseEndpoints`添加了创建实际 HTTP 响应的 MVC 控制器。ASP.NET Core 管道的完整描述将在*第十五章*的*理解 Web 应用程序的表示层*部分中给出。

在下一小节中，我们将解释 MVC 框架如何让您实现 REST 服务。

## 使用 ASP.NET Core 实现 REST 服务

今天，我们可以保证 MVC 和 Web API 的使用已经得到巩固。在 MVC 框架中，HTTP 请求由称为控制器的类处理。每个请求都映射到调用控制器公共方法。所选的控制器和控制器方法取决于请求路径的形状，并且由路由规则定义，对于 REST API，通常通过与`Controller`类及其方法相关联的属性提供。

处理 HTTP 请求的`Controller`方法称为操作方法。当选择控制器和操作方法时，MVC 框架会创建一个控制器实例来处理请求。控制器构造函数的所有参数都通过`Startup.cs`类的`ConfigureServices`方法中定义的类型进行依赖注入解析。

有关如何在.NET Core 主机中使用依赖注入的描述，请参阅*第五章*的*应用微服务架构到企业应用程序*的*使用通用主机*子章节，并参阅*第十一章*的*设计模式和.NET 5 实现*的*依赖注入模式*子章节，以获取有关依赖注入的一般讨论。

以下是一个典型的 REST API 控制器及其控制器方法定义：

```cs
[Route("api/[controller]")]
    [ApiController]
    public class ValuesController : ControllerBase
    {
        // GET api/values/5
        [HttpGet("{id}")]
        public ActionResult<string> Get(int id)
        {
            ... 
```

`[ApiController]`属性声明控制器是一个 REST API 控制器。`[Route("api/[controller]")]`声明控制器必须在以`api/<controller name>`开头的路径上进行选择。控制器名称是控制器类的名称，不包括`Controller`后缀。因此，在这种情况下，我们有`api/values`。

`[HttpGet("{id}")]`声明该方法必须在`api/values/<id>`类型的 GET 请求上调用，其中`id`必须是作为方法调用参数传递的数字。可以使用`Get(int id)`来实现。每个 HTTP 动词也有一个`Http<verb>`属性：`HttpPost`和`HttpPatch`。

我们还可以定义另一个方法如下：

```cs
[HttpGet]
public ... Get() 
```

这种方法是在`api/values`类型的`GET`请求上调用的，也就是在控制器名称后没有`id`的`GET`请求上调用的。

多个操作方法可以具有相同的名称，但只有一个应与每个请求路径兼容；否则，将抛出异常。换句话说，路由规则和`Http<verb>`属性必须明确定义每个请求的哪个控制器及其哪个操作方法。

默认情况下，参数根据以下规则传递给 API 控制器的操作方法：

+   如果路由规则将简单类型（`整数`、`浮点数`和`日期时间`）指定为参数，它们将从请求路径中获取，就像前面示例的`[HttpGet("{id}")]`属性一样。如果它们在路由规则中找不到，MVC 框架将查找具有相同名称的查询字符串参数。因此，例如，如果我们用`[HttpGet]`替换`[HttpGet("{id}")]`，MVC 框架将查找类似`api/values?id=<整数>`的内容。

+   复杂类型是由格式化程序从请求正文中提取的。根据请求的`Content-Type`标头的值选择正确的格式化程序。如果未指定`Content-Type`标头，则采用 JSON 格式化程序。JSON 格式化程序尝试解析请求正文作为 JSON 对象，然后尝试将此 JSON 对象转换为.NET Core 复杂类型的实例。如果 JSON 提取或随后的转换失败，将抛出异常。默认情况下，只支持 JSON 输入格式化程序，但您还可以添加一个 XML 格式化程序，当`Content-Type`指定 XML 内容时可以使用。只需添加`Microsoft.AspNetCore.Mvc.Formatters.Xml` NuGet 软件包，并在`Startup.cs`的`ConfigureServices`方法中用`services.AddControllers().AddXmlSerializerFormatters()`替换`services.AddControllers()`即可。

您可以通过使用适当的属性为操作方法参数添加前缀来自定义用于填充操作方法参数的源。以下代码显示了一些示例：

```cs
...MyActionMethod(....[FromHeader] string myHeader....)
// x is taken from a request header named myHeader
...MyActionMethod(....[FromServices] MyType x....)
// x is filled with an instance of MyType through dependency injection 
```

`Action`方法的返回类型必须是`IActionResult`接口或实现该接口的类型。反过来，`IActionResult`只有以下方法：

```cs
Task ExecuteResultAsync(ActionContext context); 
```

这种方法在正确的时间由 MVC 框架调用，以创建实际的响应和响应头。当传递给方法时，`ActionContext`对象包含 HTTP 请求的整个上下文，其中包括一个包含有关原始 HTTP 请求的所有必要信息的请求对象（标头、正文和 cookie），以及一个收集正在构建的响应的所有部分的响应对象。

您不必手动创建`IActionResult`的实现，因为`ControllerBase`已经有了创建`IActionResult`实现的方法，以便生成所有必要的 HTTP 响应。其中一些方法如下。

+   `OK`：这返回一个 200 状态代码，以及一个可选的结果对象。它可以作为`return OK()`或`return OK(myResult)`使用。

+   `BadRequest`：这返回一个 400 状态代码，以及一个可选的响应对象。

+   `Created(string uri, object o)`：这返回一个 201 状态代码，以及一个结果对象和创建的资源的 URI。

+   `Accepted`：这返回一个 202 状态结果，以及一个可选的结果对象和资源 URI。

+   `Unauthorized`：这返回一个 401 状态结果，以及一个可选的结果对象。

+   `Forbid`：这返回一个 403 状态结果，以及一个可选的失败权限列表。

+   `StatusCode(int statusCode, object o = null)`：这返回一个自定义状态码，以及一个可选的结果对象。

操作方法可以直接返回一个结果对象，如 `return myObject`。这相当于返回 `OK(myObject)`。

当所有结果路径返回相同类型的结果对象，比如 `MyType`，操作方法可以声明为返回 `ActionResult<MyType>`。您也可以返回诸如 `NotFound` 的响应，但是通过这种方法肯定会得到更好的类型检查。

默认情况下，结果对象以 JSON 格式序列化在响应主体中。然而，如果在 MVC 框架处理管道中添加了 XML 格式化程序，如前所示，结果的序列化方式取决于 HTTP 请求的 `Accept` 标头。更具体地说，如果客户端明确要求 XML 格式的 `Accept` 标头，对象将以 XML 格式序列化；否则，它将以 JSON 格式序列化。

作为输入传递给操作方法的复杂对象可以使用验证属性进行验证，如下所示：

```cs
public record MyType
{
   [Required]
    public string Name{get; set;}
    ...
    [MaxLength(64)]
    public string Description{get; set;}
} 
```

如果控制器已经使用了 `[ApiController]` 属性进行装饰，并且验证失败，MVC 框架会自动创建一个包含所有检测到的验证错误的字典的 `BadRequest` 响应，而不执行操作方法。因此，您无需添加进一步的代码来处理验证错误。

操作方法也可以声明为异步方法，如下所示：

```cs
public async Task<IActionResult>MyMethod(......)
{
    await MyBusinessObject.MyBusinessMethod();
    ...
}
public async Task<ActionResult<MyType>>MyMethod(......)
{
    ... 
```

本章的*用例*部分将展示控制器/操作方法的实际示例。在下一小节中，我们将解释如何处理 JWT 令牌的授权和身份验证。

### ASP.NET Core 服务授权

在使用 JWT 令牌时，授权是基于 JWT 令牌中包含的声明。任何操作方法中的所有令牌声明都可以通过 `User.Claims` 控制器属性访问。由于 `User.Claims` 是一个 `IEnumerable<Claim>`，它可以使用 `LINQ` 处理以验证声明的复杂条件。如果授权基于 *角色* 声明，您可以简单地使用 `User.IsInRole` 函数，如下所示：

```cs
If(User.IsInRole("Administrators") || User.IsInRole("SuperUsers"))
{
    ...
}
else return Forbid(); 
```

然而，权限通常不是从操作方法内部检查的，而是由 MVC 框架自动检查，根据装饰整个控制器或单个操作方法的授权属性。如果操作方法或整个控制器使用 `[Authorize]` 装饰，那么只有在请求具有有效的身份验证令牌时才能访问操作方法，这意味着我们不必对令牌声明进行检查。还可以使用以下代码检查令牌是否包含一组角色：

```cs
[Authorize(Roles = "Administrators,SuperUsers")] 
```

对声明的复杂条件需要在 `Startup.cs` 的 `ConfigureServices` 方法中定义授权策略，如下所示：

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    ...
    services.AddAuthorization(options =>
   {
      options.AddPolicy("CanDrive", policy =>
         policy.RequireAssertion(context =>
         context.User.HasClaim(c =>c.Type == "HasDrivingLicense"));
   });
} 
```

之后，您可以使用 `[Authorize(Policy = "Father")]` 装饰操作方法或控制器。

在使用基于 JWT 的授权之前，您必须在 `Startup.cs` 中进行配置。首先，您必须在 `Configure` 方法中定义的 ASP.NET Core 处理管道中添加处理身份验证令牌的中间件，如下所示：

```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
   ...
   app.UseAuthorization();
   ...
   app.UseEndpoints(endpoints =>
   {
   endpoints.MapControllers();
   });

} 
```

然后，您必须在 `ConfigureServices` 部分配置身份验证服务。在这里，您定义将通过依赖注入注入到身份验证中间件中的身份验证选项：

```cs
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => {
      options.TokenValidationParameters =
        new TokenValidationParameters
        {
            ValidateIssuer = true, 
            ValidateAudience = true, 
            ValidateLifetime = true, 
            ValidateIssuerSigningKey = true,
            ValidIssuer = "My.Issuer",
            ValidAudience = "This.Website.Audience",
            IssuerSigningKey = new
                SymmetricSecurityKey(Encoding.ASCII.GetByte
                ("MySecret"))
        };
}); 
```

上述代码为身份验证方案提供了一个名称，即默认名称。然后，它指定了 JWT 身份验证选项。通常，我们要求身份验证中间件验证 JWT 令牌是否未过期（`ValidateLifetime = true`），它具有正确的发行者和受众（请参阅本章的*REST 服务授权和身份验证*部分），以及其签名是否有效。

前面的示例使用了从字符串生成的对称签名密钥。这意味着相同的密钥用于签名和验证签名。如果 JWT 令牌是由使用它们的同一个网站创建的，这是一个可以接受的选择，但如果有一个唯一的 JWT 发行者控制对多个 Web API 站点的访问，这是一个不可接受的选择。

在这里，我们应该使用非对称密钥（通常是`RsaSecurityKey`），因此 JWT 验证只需要知道与实际私钥相关联的公钥。Identity Server 4 可以用于快速创建一个作为身份验证服务器的网站。它发出带有通常的用户名/密码凭据或转换其他身份验证令牌的 JWT 令牌。如果您使用诸如 Identity Server 4 之类的身份验证服务器，您不需要指定`IssuerSigningKey`选项，因为授权中间件能够自动从授权服务器检索所需的公钥。

只需提供身份验证服务器的 URL，如下所示：

```cs
.AddJwtBearer(options => {
options.Authority = "https://www.MyAuthorizationserver.com";
options.TokenValidationParameters =...
        ... 
```

另一方面，如果决定在 Web API 站点中发出 JWT，可以定义一个接受包含用户名和密码的对象的`Login`操作方法，并且在依赖于数据库信息的同时，使用类似以下代码构建 JWT 令牌：

```cs
var claims = new List<Claim>
{ 
   new Claim(...), 
   new Claim(...) ,
   ...
};
var token = new JwtSecurityToken( 
          issuer: "MyIssuer", 
          audience: ..., 
          claims: claims, 
          expires: DateTime.UtcNow.AddMinutes(expiryInMinutes), 
signingCredentials: 
new SymmetricSecurityKey(Encoding.ASCII.GetBytes("MySecret"));
return OK(new JwtSecurityTokenHandler().WriteToken(token)); 
```

在这里，`JwtSecurityTokenHandler().WriteToken(token)`从`JwtSecurityToken`实例中包含的令牌属性生成实际的令牌字符串。

在下一小节中，我们将学习如何通过 OpenAPI 文档端点增强我们的 Web API，以便可以自动生成用于与我们的服务通信的代理类。

### ASP.NET Core 对 OpenAPI 的支持

通过反射，大部分填写 OpenAPI JSON 文档所需的信息可以从 Web API 控制器中提取，即输入类型和来源（路径、请求体和标头）以及端点路径（这些可以从路由规则中提取）。一般来说，返回的输出类型和状态码不能轻松计算，因为它们可以动态生成。

因此，MVC 框架提供了`ProducesResponseType`属性，以便我们可以声明可能的返回类型 - 状态码对。只需为每个操作方法装饰上与可能的类型相同数量的`ProducesResponseType`属性，即可能的状态码对，如下面的代码所示：

```cs
[HttpGet("{id}")] 
[ProducesResponseType(typeof(MyReturnType), StatusCodes.Status200OK)]
[ProducesResponseType(typeof(MyErrorReturnType), StatusCodes.Status404NotFound)]
public IActionResult GetById(int id)... 
```

如果沿着某个路径没有返回对象，我们可以只声明状态码，如下所示：

```cs
 [ProducesResponseType(StatusCodes.Status403Forbidden)] 
```

当所有路径返回相同类型且该类型在操作方法返回类型中指定为`ActionResult<CommonReturnType>`时，我们也可以只指定状态码。

一旦所有操作方法都已记录，要为 JSON 端点生成任何实际的文档，我们必须安装`Swashbuckle.AspNetCore` NuGet 软件包，并在`Startup.cs`文件中放置一些代码。更具体地说，我们必须在`Configure`方法中添加一些中间件，如下所示：

```cs
app.UseSwagger(); //open api middleware
...
app.UseEndpoints(endpoints =>
{
   endpoints.MapControllers();
}); 
```

然后，我们必须在`ConfigureServices`方法中添加一些配置选项，如下所示：

```cs
services.AddSwaggerGen(c =>
{
c.SwaggerDoc("MyServiceName", new OpenApiInfo
   {
        Version = "v1",
        Title = "ToDo API",
        Description = "My service description",
   });
}); 
```

`SwaggerDoc`方法的第一个参数是文档端点名称。默认情况下，文档端点可以通过`<webroot>//swagger/<endpoint name>/swagger.json`路径访问，但可以通过多种方式进行更改。`Info`类中包含的其余信息是不言自明的。

我们可以添加多个`SwaggerDoc`调用来定义多个文档端点。然而，默认情况下，所有文档端点将包含相同的文档，其中包括项目中包含的所有 REST 服务的描述。可以通过在`services.AddSwaggerGen(c => {...})`中调用`c.DocInclusionPredicate(Func<string, ApiDescription> predicate)`方法来更改此默认设置。

`DocInclusionPredicate`必须传递一个函数，该函数接收一个 JSON 文档名称和一个操作方法描述，并且必须在该 JSON 文档中包含操作的文档时返回`true`。

要声明您的 REST API 需要 JWT 令牌，您必须在`services.AddSwaggerGen(c => {...})`中添加以下代码：

```cs
var security = new Dictionary<string, IEnumerable<string>>
{
    {"Bearer", new string[] { }},
};
c.AddSecurityDefinition("Bearer", new ApiKeyScheme
{
    Description = "JWT Authorization header using the Bearer scheme. 
    Example: \"Authorization: Bearer {token}\"",
    Name = "Authorization",
    In = "header",
    Type = "apiKey"
});
c.AddSecurityRequirement(security); 
```

您可以使用从三斜杠注释中提取的信息来丰富 JSON 文档端点，这些注释通常用于生成自动代码文档。以下代码显示了一些示例。以下代码片段显示了如何添加方法描述和参数信息：

```cs
/// <summary>
/// Deletes a specific TodoItem.
/// </summary>
/// <param name="id">id to delete</param>
[HttpDelete("{id}")]
public IActionResultDelete(long id) 
```

以下代码片段显示了如何添加使用示例：

```cs
/// <summary>
/// Creates an item.
/// </summary>
/// <remarks>
/// Sample request:
///
/// POST /MyItem
/// {
/// "id": 1,
/// "name": "Item1"
/// }
///
/// </remarks> 
```

以下代码片段显示了如何为每个 HTTP 状态代码添加参数描述和返回类型描述：

```cs
/// <param name="item">item to be created</param>
/// <returns>A newly created TodoItem</returns>
/// <response code="201">Returns the newly created item</response>
/// <response code="400">If the item is null</response> 
```

要从三斜杠注释中提取信息，我们必须通过在项目文件（`.csproj`）中添加以下代码来启用代码文档创建：

```cs
<PropertyGroup>
<GenerateDocumentationFile>true</GenerateDocumentationFile>
<NoWarn>$(NoWarn);1591</NoWarn>
</PropertyGroup> 
```

然后，我们必须通过添加以下代码来从`services.AddSwaggerGen(c => {...})`中启用代码文档处理：

```cs
var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
c.IncludeXmlComments(xmlPath); 
```

一旦我们的文档端点准备就绪，我们可以添加一些中间件，该中间件包含在相同的`Swashbuckle.AspNetCore` NuGet 包中，以生成一个友好的用户界面，我们可以在其上测试我们的 REST API：

```cs
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/<documentation name>/swagger.json", "
    <api name that appears in dropdown>");
}); 
```

如果您有多个文档端点，您需要为每个端点添加一个`SwaggerEndpoint`调用。我们将使用此接口来测试本章中定义的 REST API。

一旦您有一个可用的 JSON 文档端点，您可以使用以下方法之一自动生成代理类的 C#或 TypeScript 代码：

+   NSwagStudio Windows 程序，可在[`github.com/RicoSuter/NSwag/wiki/NSwagStudio`](https://github.com/RicoSuter/NSwag/wiki/NSwagStudio)上获得。

+   如果您想自定义代码生成，可以使用`NSwag.CodeGeneration.CSharp`或`NSwag.CodeGeneration.TypeScript` NuGet 包。

+   如果您想将代码生成与 Visual Studio 构建操作绑定在一起，可以使用`NSwag.MSBuild` NuGet 包。有关此的文档可以在[`github.com/RicoSuter/NSwag/wiki/MSBuild`](https://github.com/RicoSuter/NSwag/wiki/MSBuild)找到。

在下一小节中，您将学习如何从另一个 REST API 或.NET Core 客户端调用 REST API。

### .Net Core HTTP 客户端

`System.Net.Http`命名空间中的`HttpClient`类是一个.NET 标准 2.0 内置的 HTTP 客户端类。虽然它可以直接使用，但在重复创建和释放`HttpClient`实例时会出现一些问题，如下所示：

+   它们的创建是昂贵的。

+   例如，当`HttpClient`在`using`语句中被释放时，底层连接不会立即关闭，而是在第一次垃圾回收会话时关闭，这是一个重复的创建。释放操作会迅速耗尽操作系统可以处理的最大连接数。

因此，要么重用单个`HttpClient`实例，比如单例，要么以某种方式对`HttpClient`实例进行池化。从.NET Core 2.1 版本开始，引入了`HttpClientFactory`类来对 HTTP 客户端进行池化。更具体地说，每当需要为`HttpClientFactory`对象创建新的`HttpClient`实例时，都会创建一个新的`HttpClient`。但是，昂贵创建的底层`HttpClientMessageHandler`实例会在其最大生命周期到期之前被池化。

`HttpClientMessageHandler`实例必须具有有限的持续时间，因为它们缓存可能随时间变化的 DNS 解析信息。`HttpClientMessageHandler`的默认生命周期为 2 分钟，但可以由开发人员重新定义。

使用`HttpClientFactory`允许我们自动将所有 HTTP 操作与其他操作进行管道化。例如，我们可以添加 Polly 重试策略来处理所有 HTTP 操作的失败。有关 Polly 的介绍，请参阅*第五章*的*将微服务架构应用于企业应用程序*的*弹性任务执行*子部分。

利用`HttpClientFactory`类提供的优势的最简单方法是添加`Microsoft.Extensions.Http` NuGet 包，然后按照以下步骤操作：

1.  定义一个代理类，比如`MyProxy`，以与所需的 REST 服务进行交互。

1.  让`MyProxy`在其构造函数中接受一个`HttpClient`实例。

1.  使用注入到构造函数中的`HttpClient`来实现所有必要的操作。

1.  在主机的服务配置方法中声明您的代理，在 ASP.NET Core 应用程序的情况下，这是`Startup.cs`类的`ConfigureServices`方法，而在客户端应用程序的情况下，这是`HostBuilder`实例的`ConfigureServices`方法。在最简单的情况下，声明类似于`services.AddHttpClient<MyProxy>()`。这将自动将`MyProxy`添加到可用于依赖注入的服务中，因此您可以轻松地将其注入到控制器的构造函数中。此外，每次创建`MyProxy`的实例时，`HttpClientFactory`都会返回一个`HttpClient`并自动注入到其构造函数中。

在需要与 REST 服务进行交互的类的构造函数中，我们可能还需要一个接口，而不是具体的代理实现类型的声明：

```cs
services.AddHttpClient<IMyProxy, MyProxy>() 
```

可以应用 Polly 弹性策略（请参阅*第五章*的*将微服务架构应用于企业应用程序*的*弹性任务执行*子部分）到我们代理类发出的所有 HTTP 调用，如下所示：

```cs
var myRetryPolicy = Policy.Handle<HttpRequestException>()
    ...//policy definition
    ...;
services.AddHttpClient<IMyProxy, MyProxy>()
    .AddPolicyHandler(myRetryPolicy ); 
```

最后，我们可以预先配置传递给我们代理的所有`HttpClient`实例的一些属性，如下所示：

```cs
services.AddHttpClient<IMyProxy, MyProxy>(clientFactory =>
{
  clientFactory.DefaultRequestHeaders.Add("Accept", "application/json");
  clientFactory.BaseAddress = new Uri("https://www.myService.com/");
})
 .AddPolicyHandler(myRetryPolicy ); 
```

这样，传递给代理的每个客户端都预先配置，以便它们需要 JSON 响应并且必须与特定服务一起工作。一旦定义了基本地址，每个 HTTP 请求都需要指定要调用的服务方法的相对路径。

以下代码显示了如何执行对服务的`POST`。这需要一个额外的包`System.Net.Http.Json`。在这里，我们声明注入到代理构造函数中的`HttpClient`已存储在`webClient`私有字段中：

```cs
//Add a bearer token to authenticate the call
webClient.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);
...
//Call service method with a POST verb and get response
var response = await webClient.PostAsJsonAsync<MyPostModel>("my/method/relative/path",
    new MyPostModel
    {
        //fill model here
        ...
    });
//extract response status code
var status = response.StatusCode;
...
//extract body content from response
string stringResult = await response.Content.ReadAsStringAsync(); 
```

如果您使用 Polly，则无需拦截和处理通信错误，因为 Polly 会执行此任务。首先，您需要验证状态代码以决定下一步该做什么。然后，您可以解析响应主体中包含的 JSON 字符串，以获得一个.NET 类型的实例，通常取决于状态代码。执行解析的代码基于`System.Text.Json` NuGet 包的`JsonSerializer`类，如下所示：

```cs
var result = 
  JsonSerializer.Deserialize<MyResultClass>(stringResult); 
```

执行 GET 请求类似，但是，而不是调用`PostAsJsonAsync`，您需要调用`GetAsync`，如下所示。使用其他 HTTP 动词完全类似：

```cs
var response = 
  await webClient.GetAsync("my/getmethod/relative/path"); 
```

正如您可以在本主题中检查的那样，访问 HTTP API 非常简单，并且需要实现一些.NET 5 库。自.NET Core 开始，微软一直在努力改进框架的性能和简单性。您需要与他们不断实施的文档和设施保持联系。

# 使用情况 - 暴露 WWTravelClub 套餐

在本节中，我们将实现一个 ASP.NET REST 服务，列出给定假期开始和结束日期可用的所有套餐。出于教学目的，我们不会按照*第十二章*中描述的最佳实践结构化应用程序；相反，我们将简单地使用 LINQ 查询生成结果，并直接放置在控制器操作方法中。一个良好结构化的 ASP.NET Core 应用程序将在*第十五章*中介绍，*介绍 ASP.NET Core MVC*，该章节专门介绍 MVC 框架。

让我们复制`WWTravelClubDB`解决方案文件夹，并将新文件夹重命名为`WWTravelClubREST`。WWTravelClubDB 项目是在*第八章*的各个部分逐步构建的，*在 C#中与实体框架核心交互*。让我们打开新解决方案，并向其中添加一个名为`WWTravelClubREST`的新 ASP.NET Core API 项目（与新解决方案文件夹同名）。为简单起见，选择不进行身份验证。右键单击新创建的项目，选择**设置为启动项目**，使其成为运行解决方案时启动的默认项目。

最后，我们需要向 WWTravelClubDB 项目添加引用。

ASP.NET Core 项目将配置常量存储在`appsettings.json`文件中。让我们打开这个文件，并向其中添加我们在 WWTravelClubDB 项目中创建的数据库连接字符串，如下所示：

```cs
{
    "ConnectionStrings": {
        "DefaultConnection": "Server=
   (localdb)\\mssqllocaldb;Database=wwtravelclub;
Trusted_Connection=True;MultipleActiveResultSets=true"
    },
    ...
    ...
} 
```

现在，我们必须在`Startup.cs`的`ConfigureServices`方法中添加 WWTravelClubDB 实体框架数据库上下文，如下所示：

```cs
services.AddDbContext<WWTravelClubDB.MainDBContext>(options =>
options.UseSqlServer(
Configuration.GetConnectionString("DefaultConnection"),
            b =>b.MigrationsAssembly("WWTravelClubDB"))); 
```

传递给`AddDbContext`的选项对象设置指定了使用从`appsettings.json`配置文件的`ConnectionStrings`部分提取的连接字符串的 SQL 服务器，使用`Configuration.GetConnectionString("DefaultConnection")`方法。`b =>b.MigrationsAssembly("WWTravelClubDB")` lambda 函数声明了包含数据库迁移的程序集的名称（参见*第八章*，*在 C#中与实体框架核心交互*），在我们的情况下，这是由 WWTravelClubDB 项目生成的 DLL。为了使前面的代码编译，您应该添加`Microsoft.EntityFrameworkCore`。

由于我们希望使用 OpenAPI 文档丰富我们的 REST 服务，让我们添加对`Swashbuckle.AspNetCore` NuGet 包的引用。现在，我们可以向`ConfigureServices`方法添加以下非常基本的配置：

```cs
services.AddSwaggerGen(c =>
{
c.SwaggerDoc("WWWTravelClub", new OpenAPIInfo
    {
        Version = "WWWTravelClub 1.0.0",
        Title = "WWWTravelClub",
        Description = "WWWTravelClub Api",
TermsOfService = null
    });
}); 
```

然后，我们可以添加 OpenAPI 端点的中间件，并为 API 文档添加用户界面，如下所示：

```cs
app.UseSwagger();
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint(
        "/swagger/WWWTravelClub/swagger.json", 
        "WWWTravelClub Api");
});
app.UseEndpoints(endpoints => //preexisting code//
{
    endpoints.MapControllers();
}); 
```

现在，我们准备编码我们的服务。让我们删除 Visual Studio 自动生成的`ValuesController`。然后，右键单击`Controller`文件夹，选择**添加** | **控制器**。现在，选择一个名为`PackagesController`的空 API 控制器。首先，让我们修改代码，如下所示：

```cs
[Route("api/packages")]
[ApiController]
public class PackagesController : ControllerBase
{
    [HttpGet("bydate/{start}/{stop}")]
    [ProducesResponseType(typeof(IEnumerable<PackagesListDTO>), 200)]
    [ProducesResponseType(400)]
    [ProducesResponseType(500)]
    public async Task<IActionResult> GetPackagesByDate(
        [FromServices] WWTravelClubDB.MainDBContext ctx, 
        DateTime start, DateTime stop)
    {
    }
} 
```

`Route`属性声明了我们服务的基本路径将是`api/packages`。我们实现的唯一操作方法是`GetPackagesByDate`，它在`HttpGet`请求的路径上调用`bydate/{start}/{stop}`类型的路径，其中`start`和`stop`是作为输入传递给`GetPackagesByDate`的`DateTime`参数。`ProduceResponseType`属性声明如下：

+   当请求成功时，将返回 200 代码，并且响应体包含`PackagesListDTO`（我们将很快定义）类型的`IEnumerable`，其中包含所需的包信息。

+   当请求格式不正确时，将返回 400 代码。我们不指定返回的类型，因为坏请求会通过`ApiController`属性自动由 MVC 框架处理。

+   在出现意外错误的情况下，将返回 500 代码并带有空的响应体。

现在，让我们在一个新的`DTOs`文件夹中定义`PackagesListDTO`类：

```cs
namespace WWTravelClubREST.DTOs
{
    public record PackagesListDTO
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
        public int DurationInDays { get; set; }
        public DateTime? StartValidityDate { get; set; }
        public DateTime? EndValidityDate { get; set; }
        public string DestinationName { get; set; }
        public int DestinationId { get; set; }
    }
} 
```

最后，让我们将以下`using`子句添加到我们的控制器代码中，以便我们可以轻松地引用我们的 DTO 和 Entity Framework LINQ 方法：

```cs
using Microsoft.EntityFrameworkCore;
using WWTravelClubREST.DTOs; 
```

现在，我们准备使用以下代码填充`GetPackagesByDate`方法的主体：

```cs
try
{
    var res = await ctx.Packages
        .Where(m => start >= m.StartValidityDate
        && stop <= m.EndValidityDate)
        .Select(m => new PackagesListDTO
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
        .ToListAsync();
    return Ok(res);
}
catch (Exception err)
{
    return StatusCode(500, err);
} 
```

LINQ 查询类似于我们在*第八章*中测试的`WWTravelClubDBTest`项目中包含的查询，即*在 C#中与数据交互 - Entity Framework Core*。一旦结果计算完成，就会通过`OK`调用返回。该方法的代码通过捕获异常并返回 500 状态代码来处理内部服务器错误，因为坏请求会在`Controller`方法被`ApiController`属性调用之前自动处理。

让我们运行解决方案。当浏览器打开时，无法从我们的 ASP.NET Core 网站接收任何结果。让我们修改浏览器 URL，使其为`https://localhost:<previous port>/swagger`。OpenAPI 文档的用户界面将如下所示：

![](img/B16756_14_01.png)

图 14.1：Swagger 输出

`PackagesListDTO`是我们定义的用于列出包的模型，而`ProblemDetails`是在发生坏请求时用于报告错误的模型。通过单击**GET**按钮，我们可以获取有关我们的`GET`方法的更多详细信息，并且还可以测试它，如下面的屏幕截图所示：

![](img/B16756_14_02.png)

图 14.2：GET 方法详细信息

在插入数据库中由包覆盖的日期时要注意；否则，将返回一个空列表。在前面的屏幕截图中显示的应该可以工作。

日期必须以正确的 JSON 格式输入；否则，将返回 400 Bad Request 错误，就像下面的代码中所示的那样：

```cs
{
  "errors": {
    "start": [
      "The value '2019' is not valid."
    ]
  },
  "title": "One or more validation errors occurred.",
  "status": 400,
  "traceId": "80000008-0000-f900-b63f-84710c7967bb"
} 
```

如果插入正确的输入参数，Swagger UI 将以 JSON 格式返回满足查询的包。

就是这样！您已经实现了您的第一个带有 OpenAPI 文档的 API！

# 总结

在本章中，我们介绍了 SOA 及其设计原则和约束。其中，值得记住的是互操作性。

然后，我们专注于业务应用程序的成熟标准，以实现公开服务所需的互操作性。因此，SOAP 和 REST 服务以及从 SOAP 服务过渡到 REST 服务的细节被详细讨论，这在过去几年中在大多数应用领域都已经发生。然后，更详细地描述了 REST 服务原则、验证/授权和其文档。

最后，我们看了一下在.NET 5 中可用的工具，我们可以使用这些工具来实现和交互服务。我们看了一系列用于集群内通信的框架，如.NET remoting 和 gRPC，以及用于 SOAP 和基于 REST 的公共服务的工具。

在这里，我们主要关注 REST 服务。它们的 ASP.NET Core 实现被详细描述，以及我们可以用来验证/授权它们和它们的文档的技术。我们还专注于如何实现高效的.NET Core 代理，以便我们可以与 REST 服务交互。

在下一章中，我们将学习如何在 ASP .NET Core MVC 上构建应用程序时使用.NET 5。

# 问题

1.  服务可以使用基于 cookie 的会话吗？

1.  使用自定义通信协议实现服务是一个好的做法吗？为什么是或者为什么不是？

1.  `POST`请求到 REST 服务会导致删除吗？

1.  JWT 承载令牌中包含多少个点分隔的部分？

1.  默认情况下，REST 服务的操作方法的复杂类型参数来自哪里？

1.  如何声明控制器作为 REST 服务？

1.  ASP.NET Core 服务的主要文档属性是什么？

1.  ASP.NET Core REST 服务路由规则如何声明？

1.  如何声明代理以便我们可以利用.NET Core 的`HttpClientFactory`类的特性？

# 进一步阅读

本章主要关注更常用的 REST 服务。如果您对 SOAP 服务感兴趣，可以从维基百科关于 SOAP 规范的页面开始：[`en.wikipedia.org/wiki/List_of_web_service_specifications`](https://en.wikipedia.org/wiki/List_of_web_service_specifications)。另外，如果您对用于实现 SOAP 服务的 Microsoft .NET WCF 技术感兴趣，可以参考 WCF 的官方文档：[`docs.microsoft.com/en-us/dotnet/framework/wcf/`](https://docs.microsoft.com/en-us/dotnet/framework/wcf/)。

本章提到了 AMQP 协议作为集群内通信的一种选择，但没有进行描述。有关该协议的详细信息可在 AMQP 的官方网站上找到：[`www.amqp.org/`](https://www.amqp.org/)。

gRPC 的更多信息可在 Google gRPC 的官方网站上找到：[`grpc.io/`](https://grpc.io/)。有关 Visual Studio gRPC 项目模板的更多信息可以在这里找到：[`docs.microsoft.com/en-US/aspnet/core/grpc/`](https://docs.microsoft.com/en-US/aspnet/core/grpc/)。您还可以查看[`devblogs.microsoft.com/aspnet/grpc-web-for-net-now-available/`](https://devblogs.microsoft.com/aspnet/grpc-web-for-net-now-available/)上的 gRPC-Web。

ASP.NET Core 服务的更多详细信息可在官方文档中找到：[`docs.microsoft.com/en-US/aspnet/core/web-api/`](https://docs.microsoft.com/en-US/aspnet/core/web-api/)。有关.NET Core 的 HTTP 客户端的更多信息，请访问这里：[`docs.microsoft.com/en-US/aspnet/core/fundamentals/http-requests`](https://docs.microsoft.com/en-US/aspnet/core/fundamentals/http-requests)。

有关 JWT 令牌认证的更多信息，请访问这里：[`jwt.io/`](https://jwt.io/)。如果您想要使用 Identity Server 4 生成 JWT 令牌，可以参考其官方文档页面：[`docs.identityserver.io/en/latest/`](http://docs.identityserver.io/en/latest/)。

有关 OpenAPI 的更多信息，请访问[`swagger.io/docs/specification/about/`](https://swagger.io/docs/specification/about/)，而有关 Swashbuckle 的更多信息可以在其 GitHub 存储库页面上找到：[`github.com/domaindrivendev/Swashbuckle`](https://github.com/domaindrivendev/Swashbuckle)。
