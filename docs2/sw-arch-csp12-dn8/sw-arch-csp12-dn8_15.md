

# 第十五章：使用.NET 应用服务导向架构

**服务导向架构**（**SOA**）这个术语指的是一种模块化架构，其中系统组件之间的交互是通过通信实现的。这种方法已经发展多年，现在已成为互联网上所有系统间通信的基础。SOA 允许来自不同组织的应用程序自动交换数据和事务。除此之外，它还允许组织在互联网上提供服务。例如，在银行应用程序中，SOA 可以允许账户管理、交易处理和客户支持等独立服务无缝通信。更重要的是，它还可以使供应商直接访问客户支持。

此外，正如我们在第十一章中讨论的，*将微服务架构应用于企业应用程序*，基于通信的交互解决了由共享相同地址空间的模块组成的复杂系统中不可避免出现的二进制兼容性和版本不匹配问题。此外，使用 SOA 及其通信模式，你不需要在所有使用它的各种系统/子系统中部署同一组件的不同副本——每个组件只需在一个地方部署即可，即使它们是用不同的编程语言编写的，这也简化了**持续集成/持续交付**（**CI/CD**）的整体周期。

如果更新的版本符合客户端声明的外部通信接口，则不会出现不兼容性。例如，如果你有一个后端服务，该服务根据特定的规则和销售数据的输入来计算税费，如果特定的规则发生变化，但销售数据没有变化，你将能够更新服务而无需更改客户端中的应用程序。另一方面，使用 DLLs/包时，即使保持相同的接口，由于库模块可能与其客户端共有的其他 DLLs/包的依赖项可能存在版本不匹配，因此可能会出现不兼容性。

在第十一章*将微服务架构应用于企业应用程序*中讨论了组织协作服务的集群/网络。在本章中，我们将重点关注全球范围内使用的两个主要通信接口。更具体地说，我们将讨论以下主题：

+   理解 SOA 方法的原则

+   SOAP 和 REST Web 服务

+   .NET 8 如何处理 SOA？

到本章结束时，你将了解如何通过 ASP.NET Core 服务公开暴露应用程序中的数据。

# 技术要求

本章需要安装所有数据库工具的 Visual Studio 2022 免费社区版或更高版本。

本章中的所有概念都将通过基于 WWTravelClub 书籍用例的实用示例进行阐明，该用例位于*第二十一章，案例研究*。您可以在[`github.com/PacktPublishing/Software-Architecture-with-C-Sharp-12-and-.NET-8-4E`](https://github.com/PacktPublishing/Software-Architecture-with-C-Sharp-12-and-.NET-8-4E)找到本章的代码。

# 理解 SOA 方法的原则

就像面向对象架构中的类一样，服务是实现接口的实现，而这些接口又来自系统的功能规范。因此，*服务*设计的第一步是定义其*抽象接口*。在这个初始阶段，您可能会有两种方法：

+   将所有服务操作定义为操作您喜欢的语言（C#、Java、C++、JavaScript 等）类型的接口方法，并决定哪些操作使用同步通信实现，哪些操作使用异步通信实现。

+   首先以互操作格式创建合同。在这种方法中，您可以使用定义文件，例如 OpenAPI、Protobuf、WSDL 和 AsyncAPI 的模式，而不必接触将用于开发服务的编程语言，使用一些工具来帮助。

在这个初始阶段定义的接口不一定会在实际的服务实现中使用，它们只是有用的设计工具。一旦我们决定了服务的架构，这些接口通常会被重新定义，以便我们可以根据架构的特定性来调整它们。

值得指出的是，SOA 消息必须保持与方法调用/响应相同的语义。此外，SOA 遵循无状态开发；也就是说，对消息的反应必须不依赖于任何之前收到的消息，因为服务器不会保存先前请求的信息，这意味着消息必须相互独立。

例如，如果消息的目的是创建一个新的数据库条目，这种语义必须不因其他消息的上下文而改变，并且数据库条目的创建方式必须依赖于当前消息的内容，而不是其他之前收到的消息。因此，客户端不能创建会话，也不能登录到服务，执行一些操作，然后注销。必须在每条消息中重复使用身份验证令牌。

这种约束的原因是模块化、可测试性和可维护性。事实上，由于会话数据中隐藏的交互，基于会话的服务将非常难以测试和修改。

一旦您决定了一个服务将要实现的服务接口，您必须决定采用哪种通信堆栈/SOA。通信堆栈必须是某些官方或*事实上的*标准的一部分，以确保服务的互操作性。

互操作性是 SOA 规定的最主要约束：服务必须提供一个不依赖于特定库、实现语言或部署平台的通信接口。

考虑到你已经决定了通信栈/架构，你需要将之前的接口适应架构的特定性（有关更多详细信息，请参阅本章的*REST Web 服务*小节）。然后，你必须将这些接口转换为所选的通信语言。这意味着你必须将所有编程语言类型映射到所选通信语言中可用的类型。

数据的实际翻译通常由开发环境使用的 SOA 库自动执行。然而，可能需要一些配置，在任何情况下，我们必须了解在每次通信之前我们的编程语言类型是如何转换的。例如，某些数值类型可能被转换为精度较低或具有不同值范围的类型。例如，在.NET 8 中，你应该意识到浮点数值类型之间有所不同：**float** (~6-9 位数字)，**double** (~15-17 位数字)和**decimal** (~28-29 位数字)。你可以考虑使用字符串变量作为替代方案，以降低在传输数值类型时的精度风险。

对于无法访问其集群之外的微服务，互操作性约束可以以较轻的形式解释，因为这些微服务需要与其他属于同一集群的微服务进行通信。在这种情况下，这意味着通信栈可能是平台特定的，以便提高性能，但它必须是标准的，以避免与其他微服务兼容性问题，这些微服务可能会随着应用程序的发展而添加到集群中。

我们讨论的是*通信栈*而不是*通信协议*，因为 SOA 通信标准通常定义消息内容的格式，并为嵌入这些消息的特定协议提供不同的可能性。例如，REST 服务通常基于 JSON 消息在 HTTP/HTTPS 上运行，而 SOAP 协议仅定义了基于 XML 的各种消息格式，但 SOAP 消息可以通过各种协议传递。通常，用于 SOAP 的最常见协议也是 HTTP，但你可能决定跳到 HTTP 级别，并通过 TCP/IP 直接发送 SOAP 消息以获得更好的性能。

你应该采用的通信栈的选择取决于以下描述的几个因素。当涉及到访问数据时，通信栈可能是强制性的，由提供商决定，但你在提供服务时也应关注这些因素：

+   **兼容性约束**：如果你的服务必须向商业客户公开在互联网上，那么你必须遵守最常见的选择，这意味着使用基于 HTTP 或 REST 服务的 SOAP。如果您的客户不是商业客户而是**物联网**（**IoT**）客户，那么最常见的选择可能会有所不同。在物联网内部，不同应用领域使用的协议也可能不同。例如，海洋车辆状态数据通常使用*Signal K*进行交换。尽管这个协议过于具体，这里仅作为示例，但作为一名软件架构师，你必须理解你可能会在特定领域遇到这种标准。

+   **开发/部署平台**：并非所有通信堆栈都可在所有开发框架和所有部署平台上使用，但幸运的是，所有在公共商业服务中使用的最常见通信堆栈，如基于 SOAP 和 JSON 的 REST 通信，都可在所有主要开发/部署平台上使用。

+   **性能**：如果您的系统没有暴露给外界，而是您微服务集群的私有部分，那么性能考虑的优先级更高。在这种情况下，我们将在本章稍后讨论的 gRPC 可以被视为一个好的选择。

+   **团队/组织中工具和知识的可用性**：在考虑选择可接受的通信堆栈时，了解您的团队/组织中工具的可用性很重要。

    然而，这种类型的约束总是比兼容性约束的优先级低，因为设计一个对您的团队来说容易实现但对几乎没有人有用的系统是没有意义的。

+   **灵活性与可用功能**：一些通信解决方案虽然不够完整，但提供了更高的灵活性，而其他解决方案虽然更完整，但提供的灵活性较低。对灵活性的需求在过去的几年中引发了一场从基于 SOAP 的服务到更灵活的 REST 服务的运动。当我们在本节剩余部分描述 SOAP 和 REST 服务时，这一点将更详细地讨论。

+   **服务描述**：当服务必须在互联网上公开时，客户端应用程序需要公开的服务规范描述来设计它们的通信客户端。一些通信堆栈包括用于描述服务规范的编程语言和约定。以这种方式公开的形式化服务规范可以被处理，以便自动创建通信客户端。SOAP 更进一步，通过包含有关每个 Web 服务可以执行的任务信息的公共 XML 目录，允许服务可发现性。

一旦您选择了希望使用的通信栈，您就必须使用开发环境中可用的工具以符合所选通信栈的方式实现服务。有时，开发工具会自动确保通信栈的合规性，但有时可能需要一些开发工作。例如，在 .NET 世界中，如果您使用 WCF，开发工具会自动确保 SOAP 服务的合规性，而 REST 服务的合规性则落在开发者的责任之下，尽管从 .NET 5 开始，Swagger 提供了对 OpenAPI 标准的自动支持。以下是一些 SOA 解决方案的基本功能：

+   **身份验证**：允许客户端进行身份验证以访问服务操作。

+   **授权**：处理客户端的权限。

+   **安全**：这是如何保持通信安全，即如何防止未经授权的系统读取和/或修改通信内容。通常，加密可以防止未经授权的修改和读取，而电子签名算法可以防止仅修改。

+   **异常**：向客户端返回异常。

+   **消息可靠性**：确保在可能的基础设施故障的情况下，消息可靠地到达其目的地。

虽然有时是可取的，但以下功能并不总是必要的：

+   **分布式事务**：处理分布式事务的能力，因此当分布式事务失败或被中止时，可以撤销您所做的所有更改。

+   **支持发布/订阅模式**：是否以及如何支持事件和通知。

+   **寻址**：是否以及如何支持对其他服务和/或方法的引用。

+   **路由**：是否以及如何将消息通过服务网络进行路由。

本节的剩余部分将简要描述 SOAP 服务。然而，重点将是 REST 服务，因为今天它们是公开其集群/服务器之外的商业服务的*事实标准*。出于性能原因，微服务使用其他协议，这些协议在*第十一章*、*将微服务架构应用于您的企业应用*、*第十四章*、*使用 .NET 实现微服务*中进行了讨论。对于集群间通信，使用**高级消息队列协议**（**AMQP**），并在*进一步阅读*部分提供了链接。

# SOAP 网络服务

**简单对象访问协议**（**SOAP**）允许单向消息和请求/回复消息。通信可以是同步的，也可以是异步的，如第一章“理解软件架构的重要性”和第二章“非功能性需求”中所述，但如果底层协议是同步的，例如在 HTTP 的情况下，发送者会收到一个确认消息，表明消息已被接收（但不一定已处理）。当使用异步通信时，发送者必须监听传入的通信。通常，异步通信使用发布/订阅模式实现，我们在第六章“设计模式和.NET 8 实现”中描述了该模式。

消息表示为称为**信封**的 XML 文档。每个信封包含`header`（头部）、`body`（主体）和`fault`（错误）元素。`body`是放置实际消息内容的地方。`fault`元素包含可能的错误，因此它是通信发生时交换异常的方式。最后，`header`包含任何丰富协议但不包含域数据的辅助信息。例如，`header`可能包含一个身份验证令牌和/或签名，如果消息被签名的话。

您可以在[`www.w3.org/2003/05/soap-envelope/`](https://www.w3.org/2003/05/soap-envelope/)找到 SOAP 信封的默认命名空间。

用于发送 XML 信封的底层协议通常是 HTTP，因为这是互联网的协议，但 SOAP 规范允许使用任何协议，因此我们可以直接使用 TCP/IP 或 SMTP。事实上，更广泛使用的底层协议是 HTTP，所以如果没有充分的理由选择其他协议，应该使用 HTTP 以最大化服务的互操作性。

## SOAP 规范

SOAP 规范包含消息交换的基本内容，而其他辅助功能则在单独的规范文档中描述，称为`WS-` `*`，通常通过在 SOAP 头部添加额外信息来处理。`WS-*`规范处理我们之前列出的 SOA 的基本和期望功能。以下是一些例子：

| **WS-*** | **主要目标** |
| --- | --- |
| `WS-Security` | 负责安全，包括身份验证、授权和加密/签名 |
| `WS-Eventing` / `WS-Notification` | 两种实现发布/订阅模式的替代方法 |
| `WS-ReliableMessaging` | 关注在可能出现故障的情况下消息的可靠传递 |
| `WS-Transaction` | 关注分布式事务 |

表 15.1：关键 WS-*规范及其在基于 SOAP 的 SOA 中的主要目标总结

之前的 `WS-*` 规范并不全面，但它们是更相关和支持的功能。实际上，在各个环境（如 Java 和 .NET）中的实际实现提供了更相关的 `WS-*` 服务，但没有一种实现支持所有 `WS-*` 规范。

所有参与 SOAP 协议的 XML 文档/文档部分都在 XSD 文档中正式定义，如下例所示，这些是特殊的 XML 文档，其内容提供了 XML 结构的描述。

```cs
<?xml version="1.0"?>
<xs:schema id="sample" targetNamespace="http://tempuri.org/sample.xsd" elementFormDefault="qualified" xmlns="http://tempuri.org/sample.xsd" xmlns:xs="http://www.w3.org/2001/XMLSchema">
<xs:element name='mySample'>
<xs:complexType>
<xs:simpleContent>
<xs:extension base='xs:decimal'>
<xs:attribute name='sizing' type='xs:string' />
</xs:extension>
</xs:simpleContent>
</xs:complexType>
</xs:element>
</xs:schema> 
```

此外，如果您的自定义数据结构（面向对象语言中的类和接口）要成为 SOAP 封装的一部分，它们必须被转换为 XSD。

每个 XSD 规范都有一个相关的 `namespace`，用于标识规范及其物理位置，该位置可以找到。命名空间和物理位置都是 URI。如果 Web 服务仅从内部网络访问，则位置 URI 不需要公开可访问。

服务整体定义是一个 XSD 规范，可能包含对其他命名空间的引用，即对其他 XSD 文档的引用。简单来说，所有通过 SOAP 通信的消息都必须在 XSD 规范中定义。然后，如果服务器和客户端引用相同的 XSD 规范，它们就可以进行通信。这意味着，例如，每次向消息添加另一个字段时，都需要创建一个新的 XSD 规范。之后，需要通过创建新版本来更新所有引用旧消息定义的 XSD 文件，以指向新消息定义。反过来，这些修改需要为其他 XSD 文件创建其他版本，依此类推。因此，保持与先前行为兼容的简单修改（客户端可以简单地忽略添加的字段）可能导致版本变化的指数级链式反应。

## 与标准相关的问题

在过去几年中，处理修改的难度，以及处理所有 `WS-*` 规范的配置复杂性和性能问题，导致逐渐转向我们将在后续章节中描述的更简单的 REST 服务。这种转变始于由于实现能够在浏览器中高效运行的完整 SOAP 客户端的困难，而调用 JavaScript 中的服务。此外，复杂的 SOAP 机制对于在浏览器中运行的典型客户端的简单需求来说过大，可能导致了开发时间的完全浪费。

因此，针对非 JavaScript 客户端的服务开始大规模转向 REST 服务，如今，首选的选择是 REST 服务，SOAP 仅用于与遗留系统兼容或当需要 REST 服务不支持的功能时。

今天，我们可以考虑使用 JSON 进行数据传输的 REST 服务，这是全球最常用的方法。安全性、支持事务的设计模式、性能以及甚至文档等方面在过去的几年里都有所改进，因此这无疑是现在应用 SOA 的最佳选择。让我们在下一个主题中看看 REST 网络服务。

# REST 网络服务

REST 服务最初是为了避免在简单情况下（例如从网页的 JavaScript 代码调用服务）使用 SOAP 的复杂机制而设计的。然后，它们逐渐成为复杂系统的首选选择。REST 服务使用 HTTP 以 JSON 或较少情况下以 XML 格式交换数据。简单来说，它们用 HTTP 体替换 SOAP 体，用 HTTP 头替换 SOAP 头，用 HTTP 响应代码替换错误元素，并提供了关于所执行操作的其他辅助信息。

REST 服务成功的主要原因在于 HTTP 已经提供了 SOAP 的大部分功能，这意味着我们可以避免在 HTTP 之上构建 SOAP 层。此外，整个 HTTP 机制比 SOAP 简单：编程更简单，配置更简单，实现效率更高。

此外，REST 服务对客户端的约束较少。服务器和客户端之间的类型兼容性符合更灵活的 JavaScript 类型兼容性模型，因为 JSON 是 JavaScript 的一个子集。此外，当使用 XML 代替 JSON 时，它保持相同的 JavaScript 类型兼容性规则。不需要指定 XML 命名空间。

当使用 JSON 和 XML 时，如果服务器在保持所有其他字段与先前客户端兼容的相同语义的同时添加了一些更多字段，它们可以简单地忽略这些新字段。相应地，对 REST 服务定义所做的更改只需在导致服务器实际不兼容行为的破坏性更改的情况下传播给先前客户端。

此外，变化很可能是自我限制的，不会导致指数级的变化链，因为类型兼容性不需要在唯一共享的位置定义特定类型的引用，只需确保类型形状兼容即可。

## 服务类型兼容性规则

让我们通过一个例子来明确 REST 服务类型兼容性规则。想象一下，几个服务使用一个包含`Name`、`Surname`和`Address`字符串字段的`Person`对象。这个对象由**S1**提供服务：

```cs
{
    Name: string,
    Surname: string,
    Address: string
} 
```

如果服务和客户端引用的是先前定义的不同副本，则类型兼容性得到保证。客户端使用具有较少字段的定义也是可以接受的，因为它可以简单地忽略所有其他字段：

```cs
{
    Name: string,
    Surname: string,
} 
```

你只能在“自己的”代码中使用具有较少字段的定义。尝试在没有预期字段的情况下将信息发送回服务器可能会导致问题。

现在，想象一下这样的场景：你有一个**S2**服务，它从**S1**服务中获取`Person`对象并将它们添加到其某些方法返回的响应中。假设处理`Person`对象的**S1**服务将`Address`字符串替换为一个复杂对象：

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

在破坏性变更之后，**S2**服务将不得不调整其调用**S1**服务的通信客户端，以适应新的格式。然后，它可以将新的`Person`格式转换为旧格式，在响应中使用`Person`对象之前。这样，**S2**服务就可以避免传播**S1**的破坏性变更。

通常，基于对象形状（嵌套属性的树）而不是对同一正式类型定义的引用来建立类型兼容性，可以增加灵活性和可修改性。我们为此增加的灵活性所付出的代价是，类型兼容性不能通过比较服务器和客户端接口的正式定义来自动计算。事实上，在没有统一规范的情况下，每次发布服务的新版本时，开发者都必须验证客户端和服务器共有的所有字段的语义与上一个版本保持不变。

REST 服务背后的基本思想是放弃严格的检查和复杂的协议，以获得更大的灵活性和简单性，而 SOAP 则恰恰相反。

## REST 和原生 HTTP 功能

REST 服务宣言指出，REST 使用原生 HTTP 功能来实现所有所需的服务功能。例如，认证将通过 HTTP 的`Authorization`字段直接执行，加密将通过 HTTPS 实现，异常将通过 HTTP 错误状态码处理，而路由和可靠消息传递将由 HTTP 协议依赖的机制处理。通过使用 URL 来引用服务、其方法和其他资源来实现寻址。

由于 HTTP 是同步协议，因此没有原生对异步通信的支持。也没有原生对发布者/订阅者模式的支持，但两个服务可以通过各自向对方暴露端点来交互使用发布者/订阅者模式。更具体地说，第一个服务暴露一个订阅端点，而第二个服务暴露一个接收其通知的端点，这些通知通过在订阅期间交换的通用密钥进行授权。这种模式相当常见。GitHub 还允许我们将仓库事件发送到我们的 REST 服务。

REST 服务在实现分布式事务时没有提供简单的选项，因为 HTTP 是无状态的。然而，像在 *第十四章，使用 .NET 实现微服务* 中描述的 SAGA 模式，以及在 *第七章，理解软件解决方案中的不同领域* 中描述的事件溯源，在过去几年中极大地帮助解决了这个难题。此外，幸运的是，大多数应用领域不需要分布式事务确保的强一致性形式。对于它们，更轻量级的一致性形式，如 *最终一致性*，就足够了，并且出于性能原因更受欢迎。请参阅 *第十二章*，*在云中选择您的数据存储*，以了解各种一致性类型的讨论。

REST 宣言不仅规定了使用 HTTP 中已可用的预定义解决方案，还规定了使用类似网络的语义。一般来说，服务操作可以被视为 CRUD 操作，但不仅限于它们在由 URL 标识的资源上（同一资源可能由多个 URL 标识）。实际上，REST 是 **Representational State Transfer** 的缩写，意味着每个 URL 都是某种实体的表示。作为最佳实践，每种类型的服务请求都需要采用适当的 HTTP 动词，并按以下方式返回状态码：

+   `GET` (读取操作): URL 表示由读取操作返回的资源。因此，`GET` 操作模拟指针解引用。在操作成功的情况下，返回 200 (OK) 状态码。

+   `POST` (创建操作): 请求体中包含的 JSON/XML 对象作为新资源添加到由操作 URL 表示的对象中。如果新资源立即成功创建，则返回 201 (已创建) 状态码，以及一个依赖于操作和指示从何处检索创建的资源响应对象。响应对象应包含标识创建资源的最具体 URL。如果创建被延迟到以后的时间，则返回 202 (已接受) 状态码。

+   `PUT` (编辑操作): 请求体中包含的 JSON/XML 对象替换了由请求 URL 引用的对象。在操作成功的情况下，返回 200 (OK) 状态码。此操作是幂等的，意味着重复相同的请求两次会导致相同的修改。204 (无内容) 也是一个可能的返回值。

+   `PATCH`: 请求体中包含的 JSON/XML 对象包含如何修改由请求 URL 引用的对象的说明。由于修改可能是一个数值字段的增量，此操作不是幂等的。在操作成功的情况下，返回 200 (OK) 状态码。

+   `DELETE`: 删除由请求 URL 引用的资源。在操作成功的情况下，返回 200 (OK) 状态码。

如果资源已从请求 URL 移动到另一个 URL，则返回重定向代码：

+   `301`（永久移动），以及我们可以找到资源的新的 URL

+   `307`（临时移动），以及我们可以找到资源的新的 URL

如果操作失败，则返回一个依赖于失败类型的状态码。以下是一些失败代码的示例：

+   `400`（请求错误）：发送到服务器的请求格式不正确。

+   `404`（未找到）：当请求 URL 不指向任何已知对象时。

+   `405`（方法不允许）：当请求动词不受 URL 引用的资源支持时。

+   `401`（未授权）：操作需要身份验证，但客户端未提供任何有效的授权头。

+   `403`（禁止）：客户端正确认证，但没有权利执行操作。

+   `409`（冲突）：由于与服务器当前状态存在冲突，操作失败。

+   `412`（先决条件失败）：由于某些先决条件未满足，操作失败。

+   `422`（不可处理的内容）：请求格式良好，但其中存在语义错误。

上述状态码列表并不完整。在*进一步阅读*部分将提供完整的列表。

需要强调的是，`POST`/`PUT`/`PATCH`/`DELETE`操作可能——并且通常——对其他资源有副作用。否则，将无法编码同时作用于多个资源的操作。

换句话说，HTTP 动词必须符合由请求 URL 引用并执行的操作，但该操作可能影响其他资源。相同的操作可能使用不同的 HTTP 动词在涉及的其他资源上执行。选择以哪种方式执行相同的操作以在服务接口中实现它是开发者的责任。

由于 HTTP 动词的副作用，REST 服务可以将所有这些操作编码为对由 URL 表示的资源进行的 CRUD 操作。

通常，将现有服务迁移到 REST 需要我们在请求 URL 和请求体之间分割各种输入。更具体地说，我们提取那些唯一定义方法执行中涉及的某个对象的输入字段，并使用它们来创建一个唯一标识该对象的 URL。然后，我们根据对所选对象执行的操作来决定使用哪个 HTTP 动词。最后，我们将输入的其余部分放在请求体中。

如果我们的服务是以面向对象架构设计的，该架构专注于业务域对象（例如，如*第七章，理解软件解决方案中的不同域*中描述的 DDD），那么所有服务方法的 REST 转换应该相当直接，因为服务应该已经围绕域资源组织。否则，迁移到 REST 可能需要重新定义一些服务接口。

采用完整的 REST 语义具有这样的优势：服务可以在不修改现有操作定义的情况下进行扩展，也可以在修改现有操作定义的情况下进行扩展。实际上，扩展应主要表现为某些对象的新属性以及一些相关操作的新资源 URL。因此，现有的客户端可以简单地忽略它们。

## REST 语言中方法的示例

现在，让我们通过一个简单的银行内部转账示例来学习如何在 REST 语言中表达方法。我们将在这里介绍两种方法。

在第一种情况下，一个银行账户可以如下通过 URL 表示：

`https://mybank.com/accounts/{银行账户号}`

如果我们想象一个银行转账，我们可以将其表示为一个`PATCH`请求，其体包含一个对象，该对象具有表示金额、转账时间、描述以及接收资金的账户的属性。

该操作修改了 URL 中提到的账户以及接收账户作为*副作用*。如果账户没有足够的钱，则返回`409`（冲突）状态码，以及包含所有错误详情的对象（错误描述、可用资金等）。

然而，由于所有银行操作都记录在账户报表中，因此为与银行账户关联的“银行账户操作”集合创建和添加一个新的转账对象来表示转账是一种更好的方式。在这种第二种方法中，URL 可能如下所示：

`https://mybank.com/accounts/{银行账户号}/transactions`

在这里，由于我们正在创建一个新的对象，所以 HTTP 动词是`POST`。正文内容相同，如果资金不足，则返回`422`状态码。

两种传输表示都会在数据库中引起相同的变化。此外，一旦从不同的 URL 和可能不同的请求体中提取了输入，后续的处理就是相同的。在两种情况下，我们都有相同的输入和相同的处理——只是两个请求的外部表现不同。

然而，虚拟*交易*集合的引入使我们能够通过几个更多*交易*集合特定的方法来扩展服务。值得注意的是，*交易*集合不需要与数据库表或任何物理对象相关联：它存在于 URL 的世界中，为我们提供了一个方便的方式来模拟转账。

REST 服务的使用增加导致需要创建 REST 服务接口的描述，就像为 SOAP 开发的那些一样。这个标准被称为**OpenAPI**。我们将在下一个子节中讨论这个问题。

## OpenAPI 标准

OpenAPI 是一个用于描述 REST API 的全球性规范。OpenAPI 倡议成立于 2015 年 11 月，作为一个在 Linux 基金会下的开源项目，由 SmartBear、Google、IBM 和 Microsoft 等公司支持。该规范目前版本为 3.1。整个服务通过一个 JSON 或 YAML 端点进行描述，即一个使用 JSON 对象描述服务的端点。这个 JSON 对象有一个通用部分，适用于整个服务，并包含服务的一般特性，如其版本和描述，以及共享定义。

您可以在[`github.com/OAI/OpenAPI-Specification/`](https://github.com/OAI/OpenAPI-Specification/)找到 OpenAPI 规范示例。

然后，每个服务端点都有一个特定部分，描述端点 URL 或 URL 格式（如果 URL 中包含一些输入），所有输入，所有可能的输出类型和状态码，以及所有授权协议。每个端点特定部分可以引用通用部分中包含的定义。

本书中不包含 OpenAPI 语法的完整描述，但您可以在互联网上找到可视化编辑器，这些编辑器可以帮助您澄清之前提到的规范。SmartBear 公司提供了一个很好的例子，该公司是这项倡议的发起者之一，它被称为 Swagger Editor。在在线工具的测试版中，您可以使用 OpenAPI 3.1.0 版本加载一个示例。这有助于公司在决定 API 的编程语言之前就创建 API 合同。

不同的开发框架通过处理 REST API 代码自动生成 OpenAPI 文档，并且开发者提供了更多信息，因此您的团队不需要深入了解 OpenAPI 语法。本章节中我们将介绍的`Swashbuckle.AspNetCore` NuGet 包就是一个例子。

“.NET 8 如何处理 SOA？”这一节解释了如何在 ASP.NET Core REST API 项目中自动生成 OpenAPI 文档，而*第二十一章，案例研究*中提出的用例将提供一个其实际应用的例子。

我们将通过讨论如何在 REST 服务中处理认证和授权来结束本小节。

## REST 服务授权和认证

由于 REST 服务是无状态的，当需要认证时，客户端必须在每个请求中发送一个认证令牌。该令牌通常放置在 HTTP 授权头部，但这取决于您所使用的认证协议类型。最简单的认证方式是通过显式传输共享密钥。这可以通过以下代码实现：

```cs
Authorization: Api-Key <string known by both server and client> 
```

共享密钥被称为 API 密钥。由于在撰写本文时，尚无关于如何发送它的标准，因此 API 密钥也可以通过其他头部发送，如下面的代码所示：

```cs
X-API-Key: <string known by both server and client> 
```

值得注意的是，基于 API 密钥的认证需要 HTTPS 来防止共享密钥被盗。API 密钥非常易于使用，但它们不传达有关用户授权的信息，因此当客户端允许的操作相当标准且没有复杂的授权模式时，可以采用它们。此外，当在请求中交换时，API 密钥容易受到服务器或客户端端的攻击。一种常见的缓解方法是创建一个“服务帐户”用户，并限制其授权仅限于所需权限，并在与 API 交互时使用该特定帐户的 API 密钥。

如果您需要一个更复杂的认证服务，您可以考虑使用 OAuth 2.0 协议。例如，当您实现“使用[某些特定的社交媒体]登录”时，您可能正在使用此协议。当然，为了使用它，您必须定义一个认证服务提供商。

更安全的技巧使用有效的长期共享密钥，只需用户登录即可。然后，登录返回一个短期令牌，该令牌在所有后续请求中用作共享密钥。当短期密钥即将到期时，可以通过调用续订端点来更新它。

整个逻辑与基于短期令牌的授权逻辑完全解耦。登录通常基于接收长期凭证并返回短期令牌的登录端点。登录凭证可以是作为登录方法输入的常规用户名-密码对，也可以是其他类型的授权令牌，这些令牌被转换为由登录端点提供的短期令牌。登录还可以通过基于 X.509 证书的各种认证协议实现。

最常见的短期令牌类型是所谓的持证人令牌。每个持证人令牌都编码了有关其持续时间的详细信息以及一系列称为声明的断言列表，这些声明可用于授权目的。持证人令牌由登录操作或续订操作返回。它们的特征是它们不与接收它们的客户端或任何其他特定客户端绑定，而是识别客户端，该客户端可以简单地在其调用中使用它们。

无论客户端如何获得持证人令牌，这都是客户端需要获得的所有内容，包括其声明中隐含的所有权利。只需将持证人令牌转移到另一个客户端，就可以赋予该客户端由所有持证人令牌声明隐含的所有权利，因为基于持证人令牌的授权不需要身份证明。

因此，一旦客户端获得持证人令牌，它就可以通过将其持证人令牌转移到第三方来委托一些操作。通常，当必须使用持证人令牌进行委托时，在登录阶段，客户端指定要包含的声明以限制令牌可以授权的操作。

与 API 密钥认证相比，基于令牌的认证受到标准的约束。它们必须使用以下`Authorization`头：

```cs
Authorization: Bearer <bearer token string> 
```

携带令牌可以以多种方式实现。REST 服务通常使用 JWT，这些 JWT 是具有 Base64 URL 编码的 JSON 对象的字符串。更具体地说，JWT 的创建从 JSON 头开始，以及一个 JSON 有效载荷。JSON 头指定了令牌的类型及其签名方式，而有效载荷由一个包含所有声明作为属性/值对的 JSON 对象组成。以下是一个示例头：

```cs
{
  "alg": "RS256",
  "typ": "JWT"
} 
```

以下是一个示例有效载荷：

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

然后，将头和有效载荷进行 Base64 URL 编码，并按以下方式连接相应的字符串：

```cs
<header BASE64 string>.<payload base64 string> 
```

然后使用头中指定的算法对前面的字符串进行签名，在我们的例子中是`RSA + SHA256`，然后将签名字符串按以下方式与原始字符串连接：

```cs
<header BASE64 string>.<payload base64 string>.<signature string> 
```

前面的代码是最终的令牌字符串。可以使用对称签名代替 RSA，但在此情况下，JWT 发行者和所有使用它进行授权的服务必须共享一个共同的秘密，而使用 RSA 时，JWT 发行者的私钥不需要与任何人共享，因为可以使用发行者的公钥来验证签名。

一些有效载荷属性是标准的，如下所示：

+   `iss`：JWT 的发行者。

+   `aud`：受众，即可以使用令牌进行授权的服务和/或操作。如果一个服务没有在这个列表中看到其标识符，它应该拒绝该令牌。

+   `sub`：一个字符串，用于标识 JWT 发行的*主体*（即用户）。

+   `iat`、`exp`和`nbf`：这些分别代表 JWT 的发行时间、过期时间以及如果设置了，则代表令牌有效的起始时间。所有时间都是以 1970 年 1 月 1 日午夜 UTC 为起点的秒数表示。在这里，所有天都被认为是包含 86,400 秒。

如果我们用唯一的 URI 来表示其他声明，那么这些声明可以定义为公开的；否则，它们被认为是发行者及其已知服务的私有信息。

## API 版本控制

考虑到这样一个自然场景，即你的应用程序中 API 的数量将会增加，而且，更重要的是，业务逻辑显然会演变，作为一个软件架构师，你必须决定你将如何对 API 进行版本控制，以确保你的服务和消费这些服务的客户端之间的兼容性。

重要的是要提到，为此有几种版本控制选项：

+   **URI**：它包括在 URI 中定义 API 的版本，例如，`https://wwtravelclub.com/v1/trips`。

+   **参数**：你可以在请求中定义一个参数来定义版本，例如，`https://wwtravelclub.com/trips?version=2`。

+   **媒体类型**：在这种情况下，所需的 API 版本将通过 HTTP `Accept`头呈现。

+   **自定义请求头**：类似于媒体类型版本控制技术，但在此情况下，HTTP 头将由您自定义。

前两种方案是最常用的，但这里的重要一点是，你必须认为版本控制技术的实现至关重要。

# .NET 8 如何处理 SOA？

WCF 技术尚未移植到 .NET 5+，并且没有计划进行完整的移植。部分源代码已被捐赠，并由此启动了一个开源项目。您可以在 [`github.com/CoreWCF/CoreWCF`](https://github.com/CoreWCF/CoreWCF) 找到关于此项目的更多信息。相反，微软正在投资于 Google 的开源技术 gRPC。此外，.NET 8 通过 ASP.NET Core 对 REST 服务提供了出色的支持。

微软开发了一个工具来帮助您将 WCF 应用程序迁移到最新的 .NET。您可以在 [`devblogs.microsoft.com/dotnet/migration-wcf-to-corewcf-upgrade-assistant/`](https://devblogs.microsoft.com/dotnet/migration-wcf-to-corewcf-upgrade-assistant/) 找到它。

放弃 WCF 的主要原因是以下几方面：

+   正如我们已经讨论过的，在大多数应用领域，SOAP 技术已被 REST 技术取代。

+   WCF 技术与 Windows 紧密相关，因此从头开始重新实现其所有功能在 .NET 5+ 中将非常昂贵。由于对全 .NET 的支持将继续，需要 WCF 的用户仍然可以依赖它。

+   作为一种一般策略，从 .NET 5+ 开始，微软更倾向于投资于可以与其他竞争对手共享的开源技术。这就是为什么微软没有投资于 WCF，而是从 .NET Core 3.0 开始提供了 gRPC 实现。

下一个子节将涵盖 Visual Studio 为我们提到的每种技术提供的支持。

## SOAP 客户端支持

在 WCF 中，服务规范是通过 .NET 接口定义的，实际的服务代码是在实现这些接口的类中提供的。端点、底层协议（HTTP 和 TCP/IP）以及任何其他功能都在一个配置文件中定义。反过来，配置文件可以使用一个易于使用的配置工具进行编辑。因此，开发者只需提供作为标准 .NET 类的服务行为，并以声明性方式配置所有服务功能。这样，服务配置与实际服务行为完全解耦，每个服务都可以重新配置，以便能够适应不同的环境，而无需修改其代码。

虽然.NET 8 不支持 SOAP 技术来创建新服务，但它支持在存在许多 SOAP 服务作为遗留技术的情况下使用 SOAP 客户端。更具体地说，在 Visual Studio 中为现有的 SOAP 服务创建 SOAP 服务代理非常简单（请参阅*第六章*，*设计模式和.NET 8 实现*，以讨论代理是什么以及代理模式）。

在服务的情况下，代理是一个实现服务接口的类，其方法通过调用远程服务的类似方法来完成其工作。

要创建服务代理，在**解决方案资源管理器**中右键单击项目中的**依赖项**，然后选择**添加连接的服务**。然后，在出现的表单中，选择**Microsoft WCF 服务引用提供程序**。在那里，您可以指定服务的 URL（其中包含 WSDL 服务描述），您希望添加代理类的命名空间，以及更多。在向导结束时，Visual Studio 自动添加所有必要的 NuGet 包并构建代理类。这足以创建此类的一个实例并调用其方法，以便我们可以与远程 SOAP 服务交互。

此外，还有一些第三方工具，例如 NuGet 包，为 SOAP 服务提供有限的支持，但当前它们并不非常实用，因为这种有限的支持不包括 REST 服务中不可用的功能。

## gRPC 支持

.NET SDK 支持 gRPC 项目模板，该模板可以构建一个 gRPC 服务器和一个 gRPC 客户端。gRPC 实现了一种远程过程调用模式，提供同步和异步调用，从而减少了客户端和服务器之间消息的流量。

使用 gRPC 非常简单，因为 Visual Studio 的 gRPC 项目模板自动构建了一切，使得 gRPC 服务和其客户端可以正常工作。开发者只需定义特定于应用程序的 C#服务接口及其实现类。

对于配置，服务是通过 Protobuf 文件中编写的接口定义的，并且它们的代码由实现这些接口的 C#类提供，而客户端通过实现相同服务接口的代理与这些服务交互。

gRPC 是微服务集群内部通信的良好选择。由于所有主要语言和开发框架都有 gRPC 库，因此它可以在基于 Kubernetes 的集群中使用。此外，由于 gRPC 具有更紧凑的数据表示方式，并且由于其使用起来更简单，因为与协议相关的一切都由开发框架处理，因此 gRPC 比 REST 服务协议更高效。

因此，我们新增了一章专门讨论这种实现，即第十四章“使用.NET 实现微服务”，您可以在[`docs.microsoft.com/en-us/aspnet/core/tutorials/grpc/grpc-start?view=aspnetcore-8.0`](https://docs.microsoft.com/en-us/aspnet/core/tutorials/grpc/grpc-start?view=aspnetcore-8.0)查看有关技术的详细信息。

本节剩余部分将致力于介绍.NET 对 REST 服务的服务器和客户端支持。

## ASP.NET Core 简介简述

ASP.NET Core 应用程序是基于我们在第十一章“将微服务架构应用于您的企业应用程序”的“使用通用宿主”子节中描述的**Host**概念的.NET 应用程序。

使用 C# 12 和.NET 8，创建 ASP.NET Core 应用程序的模板有所改变。主要目的是简化我们的设置方式。每个 ASP.NET 应用程序的`Program.cs`文件现在创建一个宿主，构建它，并运行它，不再需要`Startup`类，如下面的代码所示：

```cs
var builder = WebApplication.CreateBuilder(args);
// Add services to the container.
builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
var app = builder.Build();
// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run(); 
```

`Environment`变量来自`ASPNETCORE_ENVIRONMENT`环境变量。反过来，当应用程序在 Visual Studio 的**解决方案资源管理器**中运行时，它定义在`Properties\launchSettings.json`文件中。在这个文件中，你可以定义几个环境，可以通过 Visual Studio 运行按钮旁边的下拉菜单选择，**IIS Express**。默认情况下，**IIS Express**设置将`ASPNETCORE_ENVIRONMENT`设置为`Development`。以下是一个典型的`launchSettings.json`文件：

```cs
{
"iisSettings": {
"windowsAuthentication": false,
"anonymousAuthentication": true,
"iisExpress": {
"applicationUrl": "http://localhost:48638",
"sslPort": 44367
}
},
"profiles": {
"http": {
"commandName": "Project",
"dotnetRunMessages": true,
"launchBrowser": true,
"launchUrl": "swagger",
"applicationUrl": "http://localhost:5085",
"environmentVariables": {
"ASPNETCORE_ENVIRONMENT": "Development"
}
},
"https": {
"commandName": "Project",
"dotnetRunMessages": true,
"launchBrowser": true,
"launchUrl": "swagger",
"applicationUrl": "https://localhost:7214;http://localhost:5085",
"environmentVariables": {
"ASPNETCORE_ENVIRONMENT": "Development"
}
},
"IIS Express": {
"commandName": "IISExpress",
"launchBrowser": true,
"launchUrl": "swagger",
"environmentVariables": {
"ASPNETCORE_ENVIRONMENT": "Development"
}
}
}
} 
```

当应用程序发布时，用于`ASPNETCORE_ENVIRONMENT`的值可以在 Visual Studio 创建后添加到发布的 XML 文件中。此值为`<EnvironmentName>Staging</EnvironmentName>`。它也可以在您的 Visual Studio ASP.NET Core 项目文件（`.csproj`）中指定：

```cs
<PropertyGroup>
<EnvironmentName>Staging</EnvironmentName>
</PropertyGroup> 
```

管道中的每个中间件都由一个`app.Use<something>`方法定义，它通常接受一些选项。每个都会处理请求，然后将修改后的请求转发到管道中的下一个，或者返回一个 HTTP 响应。当返回 HTTP 响应时，它将按相反顺序处理所有之前的中间件。

模块按照`app.Use<something>`方法定义的顺序插入到管道中。前面的代码在`ASPNETCORE_ENVIRONMENT`为`Development`时添加一个错误页面。关于 ASP.NET Core 管道的完整描述将在第十七章“理解 Web 应用程序的表现层”部分中给出。

在下一小节中，我们将解释 MVC 框架如何让您实现 REST 服务。

## 使用 ASP.NET Core 实现 REST 服务

今天，我们可以保证 MVC 和 Web API 的使用已经得到巩固。在 MVC 框架中，HTTP 请求由称为控制器的类处理。每个请求都映射到控制器公共方法的调用。所选控制器及其控制器方法取决于请求路径的形状，它们由路由规则定义，对于 REST API，这些规则通常通过与 `Controller` 类及其方法关联的属性提供。

ASP.NET Core 6 引入了最小 API 以简化使用 C# 实现 API 的机制。您可以在[`docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis`](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)找到关于它的良好解释。

处理 HTTP 请求的 `Controller` 方法称为操作方法。当控制器和操作方法被选中时，MVC 框架创建一个控制器实例来处理请求。控制器构造函数的所有参数都通过依赖注入解决。

请参阅第十一章，*将微服务架构应用于您的企业应用程序*中的*使用通用宿主*子节，了解如何使用 .NET 宿主进行依赖注入的描述，以及第六章，*设计模式和 .NET 8 实现*中的*依赖注入模式*子节，了解依赖注入的一般讨论。

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

`[ApiController]` 属性声明该控制器是一个 REST API 控制器。`[Route("api/[controller]")]` 声明控制器必须在以 `api/<controller name>` 开头的路径上被选中。控制器的名称是控制器类名称，不带 `Controller` 后缀。这比硬编码控制器名称更节省重构时间。因此，在这种情况下，我们有 `api/values`。

`[HttpGet("{id}")]` 声明该方法必须在 `api/values/<id>` 类型的 GET 请求上被调用，其中 `id` 必须是一个作为方法调用参数传递的数字。这可以通过 `Get(int id)` 实现。对于每个 HTTP 动词，也存在一个 `Http<verb>` 属性：`HttpPost` 和 `HttpPatch`。

我们还可以定义另一个方法，如下所示：

```cs
[HttpGet]
public ... Get() 
```

此方法在 `api/values` 类型的 `GET` 请求上被调用，即在没有控制器名称后跟 `id` 的 `GET` 请求上。

几个操作方法可以具有相同的名称，但每个请求路径只能有一个与之兼容；否则，会抛出异常。换句话说，路由规则和 `Http<verb>` 属性必须唯一地定义每个请求应选择哪个控制器及其哪个操作方法。

默认情况下，参数根据以下规则传递给 API 控制器的操作方法。

简单类型（`整数`、`浮点数`和 `DateTimes`）如果路由规则指定它们为参数，则从请求路径中获取，例如上一个示例的 `[HttpGet("{id}")]` 属性。如果它们在路由规则中找不到，ASP.NET Core 框架将寻找具有相同名称的查询字符串参数。因此，如果我们用 `[HttpGet("{id}")]` 替换 `[HttpGet]`，ASP.NET Core 框架将寻找类似 `api/values?id=<id type>` 或 `api/values/{id}` 的内容。

复杂类型通过格式化程序从请求体中提取。根据请求的 `Content-Type` 头部的值选择正确的格式化程序。如果没有指定 `Content-Type` 头部，则使用 JSON 格式化程序。JSON 格式化程序尝试将请求体解析为 JSON 对象，然后尝试将此 JSON 对象转换为 .NET 复杂类型的实例。如果 JSON 提取或后续转换失败，则抛出异常。如 *第二章，非功能性需求* 中所述，由于异常的计算成本远高于正常代码流，因此请小心处理异常。

如果不可避免地出现异常，请考虑使用 *第四章，C# 编码最佳实践 12* 中描述的日志记录建议。默认情况下，仅支持 JSON 输入格式化程序，但您也可以添加一个 XML 格式化程序，当 `Content-Type` 指定 XML 内容时可以使用。

您可以通过在参数前加上适当的属性来自定义用于填充动作方法参数的源。以下代码显示了此示例的一些示例：

```cs
...MyActionMethod(....[FromHeader] string myHeader....)
// x is taken from a request header named myHeader
...MyActionMethod(....[FromServices] MyType x....)
// x is filled with an instance of MyType through dependency injection 
```

动作方法的返回类型可以是 `IActionResult` 接口、实现该接口的类型或 DTO。反过来，`IActionResult` 只有以下方法：

```cs
Task ExecuteResultAsync(ActionContext context); 
```

这种方法由 MVC 框架在正确的时间调用以创建实际的响应和响应头。当传递给方法时，`ActionContext` 对象包含整个 HTTP 请求的上下文，其中包括一个包含有关原始 HTTP 请求（头部、主体和 Cookie）所有必要信息的请求对象，以及一个收集正在构建的响应所有片段的响应对象。

您不必手动创建 `IActionResult` 的实现，因为 `ControllerBase` 已经有创建 `IActionResult` 实现的方法，以便生成所有必要的 HTTP 响应。以下是一些这些方法：

+   `OK`：这返回一个 200 状态码，以及一个可选的结果对象。它用作 `return OK()` 或 `return OK(myResult)`。

+   `BadRequest`：这返回一个 400 状态码，以及一个可选的响应对象。

+   `Created(string uri, object o)`: 这返回一个 201 状态码，以及一个结果对象和创建资源的 URI。

+   `Accepted`：这返回一个 202 状态结果，以及一个可选的结果对象和资源 URI。

+   `Unauthorized`：这返回一个 401 状态结果，以及一个可选的结果对象。

+   `Forbid`：这返回一个 403 状态结果，以及一个可选的失败权限列表。

+   `StatusCode(int statusCode, object o = null)`：这返回一个自定义状态码，以及一个可选的结果对象。

动作方法可以直接使用`return myObject`返回结果对象。这相当于返回`OK(myObject)`。

当所有结果路径返回相同类型的对象，例如`MyType`，动作方法可以声明为返回`ActionResult<MyType>`。你也可以返回类似`NotFound`的响应，但无疑，这种方法可以得到更好的类型检查。

默认情况下，结果对象在响应体中以 JSON 格式序列化。然而，如果已将 XML 格式化程序添加到 ASP.NET Core 框架处理管道中，如前所述，结果的序列化方式取决于 HTTP 请求的`Accept`头。更具体地说，如果客户端明确要求使用`Accept`头以 XML 格式，对象将以 XML 格式序列化；否则，将以 JSON 格式序列化。

将作为输入传递给动作方法的复杂对象可以使用以下验证属性进行验证：

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

如果控制器已用`[ApiController]`属性装饰，并且验证失败，ASP.NET Core 框架会自动创建一个包含所有检测到的验证错误的`BadRequest`响应，而不执行动作方法。因此，你不需要添加额外的代码来处理验证错误。

动作方法也可以声明为`async`方法，如下所示：

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

实际的控制器/动作方法示例将在*用例 - 暴露 WWTravelClub 包*中展示，该用例在*第二十一章，案例研究*中介绍。在下一个小节中，我们将解释如何使用 JWT 处理授权和身份验证。

### ASP.NET Core 服务授权

当使用 JWT 时，授权基于 JWT 中包含的声明。任何动作方法中的所有令牌声明都可以通过`User.Claims`控制器属性访问。由于`User.Claims`是`IEnumerable<Claim>`，它可以使用 LINQ 进行处理，以验证声明上的复杂条件。

如果基于*角色*声明进行授权，你可以简单地使用`User.IsInRole`函数，如下面的代码所示：

```cs
If(User.IsInRole("Administrators") || User.IsInRole("SuperUsers"))
{
    ...
}
else return Forbid(); 
```

然而，通常不会在动作方法内部检查权限，而是由 MVC 框架根据装饰整个控制器或单个动作方法的授权属性自动检查。如果动作方法或整个控制器被`[Authorize]`装饰，则只有当请求具有有效的身份验证令牌时，才能访问动作方法，这意味着我们不需要对令牌声明进行检查。也可以使用以下代码检查令牌是否包含一组角色：

```cs
[Authorize(Roles = "Administrators,SuperUsers")] 
```

对于更复杂的声明条件，需要在构建应用程序时在 `Program.cs` 中定义授权策略，如下面的代码所示：

```cs
var builder = WebApplication.CreateBuilder(args);
// Add services to the container.
builder.Services.AddControllers();
...
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("CanDrive", policy =>
       policy.RequireAssertion(context =>
       context.User.HasClaim(c => c.Type == "HasDrivingLicense")));
}); 
```

之后，您可以对动作方法或控制器使用 `[Authorize(Policy = "Father")]` 装饰器。

在使用基于 JWT 的授权之前，您必须在 `Program.cs` 中进行配置。首先，您必须添加处理 ASP.NET Core 中身份验证令牌的中间件，如下所示：

```cs
var app = builder.Build();
...
app.UseAuthorization();
app.MapControllers();
app.Run(); 
```

然后，您必须配置身份验证服务。在那里，您定义将通过依赖注入注入到身份验证中间件的身份验证选项：

```cs
var builder = WebApplication.CreateBuilder(args);
// Add services to the container.
builder.Services.AddControllers();
...
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
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
                  SymmetricSecurityKey(Encoding.ASCII.GetBytes("MySecret"))
          };
    }); 
```

上述代码为身份验证方案提供了一个名称，即默认名称。然后，它指定 JWT 身份验证选项。通常，我们要求身份验证中间件验证 JWT 是否未过期（`ValidateLifetime = true`），它是否有正确的发行者和受众（参见本章的 *REST 服务授权和身份验证* 部分），以及其签名是否有效。

上述示例使用从字符串生成的对称签名密钥。这意味着相同的密钥用于签名和验证签名。如果 JWT 由使用它们的同一网站创建，这是一个可接受的选择，但如果有一个独特的 JWT 发行者控制对多个 Web API 站点的访问，则这不是一个可接受的选择。

在这里，我们应该使用非对称密钥（通常是 `RsaSecurityKey`），因此 JWT 验证只需要知道与实际私有签名密钥关联的公钥。可以使用 IdentityServer 4 快速创建一个作为身份验证服务器的网站。它可以使用常规的用户名/密码凭据发出 JWT 或转换其他身份验证令牌。如果您使用像 IdentityServer 4 这样的身份验证服务器，则不需要指定 `IssuerSigningKey` 选项，因为授权中间件能够自动从授权服务器检索所需的公钥。

只需提供身份验证服务器 URL 即可，如下所示：

```cs
.AddJwtBearer(options => {
options.Authority = "https://www.MyAuthorizationserver.com";
options.TokenValidationParameters =...
        ... 
```

另一方面，如果您决定在您的 Web API 站点发出 JWT，您可以定义一个 `Login` 动作方法，该方法接受一个包含用户名和密码的对象，并且依赖于数据库信息，使用类似于以下代码的方式构建 JWT：

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

在这里，`JwtSecurityTokenHandler().WriteToken(token)` 从包含在 `JwtSecurityToken` 实例中的令牌属性生成实际的令牌字符串。

在下一个子节中，我们将学习如何通过 OpenAPI 文档端点增强我们的 Web API，以便可以自动生成与我们的服务通信的代理类。

### ASP.NET Core 对 OpenAPI 的支持

填写 OpenAPI JSON 文档所需的大部分信息可以通过反射从 Web API 控制器中提取，即输入类型和来源（路径、请求体和头部）以及端点路径（这些可以从路由规则中提取）。返回输出类型和状态码通常难以计算，因为它们可以动态生成。

因此，MVC 框架提供了 `ProducesResponseType` 属性，以便我们可以声明可能的返回类型——状态码对。只需用尽可能多的 `ProducesResponseType` 属性装饰每个操作方法，即可能的类型，也就是可能的状态码对，如下所示：

```cs
[HttpGet("{id}")]
[ProducesResponseType(typeof(MyReturnType), StatusCodes.Status200OK)]
[ProducesResponseType(typeof(MyErrorReturnType), StatusCodes.Status404NotFound)]
public IActionResult GetById(int id)... 
```

如果路径上没有返回任何对象，我们只需声明状态码如下：

```cs
 [ProducesResponseType(StatusCodes.Status403Forbidden)] 
```

我们也可以在所有路径返回相同类型且该类型已在操作方法返回类型中指定为 `ActionResult<CommonReturnType>` 时，仅指定状态码。

一旦所有操作方法都已记录，要生成 JSON 端点的任何实际文档，我们必须安装 `Swashbuckle.AspNetCore` NuGet 包并在 `Program.cs` 文件中放置一些代码：

在 .NET 5+ 中，您可以通过在创建项目时勾选 **OpenAPI 支持** 来自动包含它。

```cs
var builder = WebApplication.CreateBuilder(args);
...
//open api middleware
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new() { Title = "WWTravelClubREST60", Version = "v1" });
});
var app = builder.Build();
...
app.UseSwagger();
app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "WWTravelClubREST60 v1"));
...
app.Run(); 
```

`SwaggerDoc` 方法的第一个参数是文档端点名称。默认情况下，文档端点可通过 `<webroot>//swagger/<endpoint name>/swagger.json` 路径访问，但可以通过多种方式更改。`Info` 类中包含的其他信息是自解释的。

我们可以添加多个 `SwaggerDoc` 调用来定义多个文档端点。然而，默认情况下，所有文档端点都将包含相同的文档，其中包括项目中包含的所有 REST 服务的描述。此默认值可以通过在 `services.AddSwaggerGen(c => {...})` 中调用 `c.DocInclusionPredicate(Func<string, ApiDescription> predicate)` 方法来更改。

`DocInclusionPredicate` 必须传递一个函数，该函数接收一个 JSON 文档名称和一个操作方法描述，并且如果操作方法的文档必须包含在该 JSON 文档中，则必须返回 `true`。

要声明您的 REST API 需要 JWT，您必须在 `services.AddSwaggerGen(c => {...})` 内部添加以下代码：

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

您可以通过从三斜杠注释中提取信息来丰富 JSON 文档端点，这些注释通常用于生成自动代码文档。以下代码展示了这一点的示例。以下代码片段展示了我们如何添加方法描述和参数信息：

```cs
/// <summary>
/// Deletes a specific TodoItem.
/// </summary>
/// <param name="id">id to delete</param>
[HttpDelete("{id}")]
public IActionResultDelete(long id) 
 we can add an example of usage:
```

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

以下代码片段展示了我们如何为每个 HTTP 状态码添加参数描述和返回类型描述：

```cs
/// <param name="item">item to be created</param>
/// <returns>A newly created TodoItem</returns>
/// <response code="201">Returns the newly created item</response>
/// <response code="400">If the item is null</response> 
```

要启用从三斜杠注释中提取，我们必须通过将以下代码添加到我们的项目文件（`.csproj`）中来启用代码文档创建：

```cs
<PropertyGroup>
<GenerateDocumentationFile>true</GenerateDocumentationFile>
<NoWarn>$(NoWarn);1591</NoWarn>
</PropertyGroup> 
```

然后，我们必须在 `services.AddSwaggerGen(c => {...})` 内启用代码文档处理，通过添加以下代码：

```cs
var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
c.IncludeXmlComments(xmlPath); 
```

一旦我们的文档端点准备就绪，我们可以添加一些包含在同一个 `Swashbuckle.AspNetCore` NuGet 包中的中间件，以生成一个友好的用户界面，我们可以在其上测试我们的 REST API：

```cs
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/<documentation name>/swagger.json", "
    <api name that appears in dropdown>");
}); 
```

如果你拥有多个文档端点，你需要为每个端点添加一个 `SwaggerEndpoint` 调用。我们将使用此接口来测试书中用例中定义的 REST API，该用例在 *第二十一章，案例研究* 中介绍。在那里你还将了解到如何使用 Postman，一个用于构建和使用的 API 平台。

一旦你有了工作的 JSON 文档端点，你可以使用以下方法之一自动生成代理类的 C# 或 TypeScript 代码，该代理类在 *第六章，设计模式和 .NET 8 实现* 中介绍：

+   可在 [`github.com/RicoSuter/NSwag/wiki/NSwagStudio`](https://github.com/RicoSuter/NSwag/wiki/NSwagStudio) 找到的 NSwagStudio Windows 程序。

+   如果你想要自定义代码生成，可以使用 `NSwag.CodeGeneration.CSharp` 或 `NSwag.CodeGeneration.TypeScript` NuGet 包。

+   如果你想要将代码生成与 Visual Studio 构建操作关联起来，可以使用 `NSwag.MSBuild` NuGet 包。有关此包的文档可以在 [`github.com/RicoSuter/NSwag/wiki/NSwag.MSBuild`](https://github.com/RicoSuter/NSwag/wiki/NSwag.MSBuild) 找到。

在下一小节中，你将学习如何从一个 REST API 或从 .NET 客户端调用 REST API。

### .NET HTTP 客户端

`System.Net.Http` 命名空间中的 `HttpClient` 类是一个 .NET Standard 2.0 内置的 HTTP 客户端类。虽然它可以直接在任何需要与 REST 服务交互时使用，但反复创建和释放 `HttpClient` 实例存在一些问题，如下所述：

+   它们的创建成本很高。

+   当 `HttpClient` 被释放时，例如在 using 语句中，底层的连接不会立即关闭，而是在第一次垃圾回收会话时关闭。因此，重复的创建和释放操作会迅速耗尽操作系统可以处理的连接最大数量。

因此，可以重用单个 `HttpClient` 实例，例如 Singleton，或者以某种方式池化 `HttpClient` 实例。从 .NET Core 2.1 版本开始，引入了 `HttpClientFactory` 类来池化 HTTP 客户端。更具体地说，每当需要为 `HttpClientFactory` 对象创建新的 `HttpClient` 实例时，就会创建一个新的 `HttpClient`。然而，底层的 `HttpClientMessageHandler` 实例，创建成本较高，会被池化直到其最大生命周期到期。

`HttpClientMessageHandler` 实例必须有一个有限的生命周期，因为它们缓存了可能随时间变化的 DNS 解析信息。`HttpClientMessageHandler` 的默认生命周期为 2 分钟，但可以被开发者重新定义。

使用`HttpClientFactory`允许我们自动将所有 HTTP 操作与其他操作进行管道化。例如，我们可以添加 Polly 重试策略来自动处理所有 HTTP 操作的所有失败。有关 Polly 的介绍，请参阅第五章的*实现 C# 12 中的代码重用*的*弹性任务执行*子节。

利用`HttpClientFactory`类提供的优势的最简单方法是添加`Microsoft.Extensions.Http` NuGet 包，然后按照以下步骤操作：

1.  定义一个代理类，例如`MyProxy`，以与所需的 REST 服务交互。

1.  让`MyProxy`在其构造函数中接受一个`HttpClient`实例。

1.  使用构造函数中注入的`HttpClient`来实现所有必要的操作。

1.  在宿主的服务配置方法中声明你的代理，在 ASP.NET Core 应用程序的情况下，这个方法位于`Program.cs`类中。在最简单的情况下，声明类似于`builder.Services.AddHttpClient<MyProxy>()`。

这样，`MyProxy`将自动添加到可用于依赖注入的服务中，因此你可以轻松地在控制器构造函数中注入它。此外，每次创建`MyProxy`的实例时，`HttpClientFactory`都会返回一个`HttpClient`实例，并将其自动注入到其构造函数中。

在需要与 REST 服务交互的类构造函数中，我们可能也需要一个接口，而不是具有类型声明的特定代理实现：

```cs
builder.Services.AddHttpClient<IMyProxy, MyProxy>() 
```

这样，每个传递给代理的客户端都会预先配置，以便它需要一个 JSON 响应，并且必须与特定服务一起工作。一旦定义了基本地址，每个 HTTP 请求都需要指定要调用的服务方法的相对路径。

以下代码展示了如何向服务执行`POST`操作。这需要额外的包`System.Net.Http.Json`，因为使用了`PostAsJsonAsync`。在这里，我们声明注入到代理构造函数中的`HttpClient`已经被存储在`webClient`私有字段中：

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

如果你使用 Polly，你不需要拦截和处理通信错误，因为这个任务由 Polly 执行。首先，你需要验证状态码以决定下一步要做什么。然后，你可以解析响应体中包含的 JSON 字符串，以获取一个.NET 类型的实例，这通常取决于状态码。执行解析的代码基于`System.Text.Json` NuGet 包的`JsonSerializer`类，如下所示：

```cs
var result =
  JsonSerializer.Deserialize<MyResultClass>(stringResult); 
```

执行`GET`请求类似，但需要调用`GetAsync`而不是`PostAsJsonAsync`，如下所示。其他 HTTP 动词的使用完全类似：

```cs
var response =
  await webClient.GetAsync("my/getmethod/relative/path"); 
```

如您从本节中看到的那样，访问 HTTP API 非常简单，需要实现一些 .NET 6 库。自 .NET Core 以来，Microsoft 一直在努力改进框架的这一部分的性能和简单性。您需要确保自己了解他们持续实施的文档和设施。

# 摘要

在本章中，我们介绍了 SOA、其设计原则和其约束。其中，互操作性值得记住。

然后，我们关注了适用于业务应用的标准，这些标准实现了公开服务所需的互操作性。因此，详细讨论了 SOAP 和 REST 服务，以及在过去几年中大多数应用领域发生的从 SOAP 服务到 REST 服务的转变。然后，更详细地描述了 REST 服务原则、认证/授权和文档。

最后，我们探讨了 .NET 8 中可用的工具，我们可以使用这些工具来实现和交互服务。我们探讨了用于集群内通信的各种框架，如 .NET 远程过程调用和 gRPC，以及基于 SOAP 和 REST 的公共服务的工具。

在这里，我们主要关注 REST 服务。它们的 ASP.NET Core 实现被详细描述，包括我们可以用来认证/授权它们的技巧以及它们的文档。我们还关注了如何实现高效的 .NET 代理，以便我们可以与 REST 服务交互。

在下一章中，我们将学习如何使用 .NET 8 实现 ASP.NET Core 微服务。

# 问题

1.  服务可以使用基于 cookie 的会话吗？

1.  实现一个具有自定义通信协议的服务是否是好的实践？为什么或为什么不？

1.  向 REST 服务发送的 `POST` 请求会导致删除吗？

1.  JWT 持有者令牌中包含多少个点分隔的部分？

1.  默认情况下，REST 服务操作方法的复杂类型参数是从哪里获取的？

1.  如何声明控制器作为 REST 服务？

1.  ASP.NET Core 服务的最主要文档属性是什么？

1.  ASP.NET Core REST 服务路由规则是如何声明的？

1.  如何声明代理，以便我们可以利用 .NET `HttpClientFactory` 类的功能？

# 进一步阅读

+   本章主要关注更常用的 REST 服务。如果您对 SOAP 服务感兴趣，可以从关于 SOAP 规范的维基百科页面开始了解：[`en.wikipedia.org/wiki/List_of_web_service_specifications`](https://en.wikipedia.org/wiki/List_of_web_service_specifications)。另一方面，如果您对实现 SOAP 服务的 Microsoft .NET WCF 技术感兴趣，可以在此处参考 WCF 的官方文档：[`docs.microsoft.com/en-us/dotnet/framework/wcf/`](https://docs.microsoft.com/en-us/dotnet/framework/wcf/).

+   本章提到了 AMQP 协议作为集群内部通信的选项，但没有对其进行描述。关于此协议的详细信息可在 AMQP 的官方网站找到：[`www.amqp.org/`](https://www.amqp.org/).

+   关于 gRPC 的更多信息可在 Google gRPC 的官方网站找到：[`grpc.io/`](https://grpc.io/). 关于 Visual Studio gRPC 项目模板的更多信息请在此处查看：[`docs.microsoft.com/en-US/aspnet/core/grpc/`](https://docs.microsoft.com/en-US/aspnet/core/grpc/). 您还可以查看 gRPC-Web，详情请访问：[`devblogs.microsoft.com/aspnet/grpc-web-for-net-now-available/`](https://devblogs.microsoft.com/aspnet/grpc-web-for-net-now-available/).

+   关于 ASP.NET Core 服务的更多详细信息可在官方文档中找到：[`docs.microsoft.com/en-US/aspnet/core/web-api/`](https://docs.microsoft.com/en-US/aspnet/core/web-api/). 关于 .NET HTTP 客户端的更多信息请在此处查看：[`docs.microsoft.com/en-US/aspnet/core/fundamentals/http-requests`](https://docs.microsoft.com/en-US/aspnet/core/fundamentals/http-requests).

+   最小 API 的描述可在 [`docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis`](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis) 找到。

+   关于 JWT 验证的更多信息请在此处查看：[`jwt.io/`](https://jwt.io/). 如果您想使用 IdentityServer 生成 JWT，可以参考其官方文档页面：[`docs.duendesoftware.com/identityserver/v7`](https://docs.duendesoftware.com/identityserver/v7).

+   关于 OpenAPI 的更多信息可在 [`swagger.io/docs/specification/about/`](https://swagger.io/docs/specification/about/) 找到，而关于 Swashbuckle 的更多信息可在其 GitHub 仓库页面找到：[`github.com/domaindrivendev/Swashbuckle`](https://github.com/domaindrivendev/Swashbuckle).

# 留下评论！

喜欢这本书吗？通过留下亚马逊评论来帮助像您这样的读者。扫描下面的二维码以获取 20% 的折扣码。

![](img/Leave_a_review_QR.png)

**限时优惠**
