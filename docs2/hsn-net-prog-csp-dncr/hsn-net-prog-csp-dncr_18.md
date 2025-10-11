# 第十四章：网络上的身份验证和授权

在上一章中，我们认真考虑了确保安全连接的传输层解决方案。有了这些知识，我们将在本章中探讨需要传输层安全的主机到主机交互类型。我们将退回到网络堆栈的层级，进入应用层，看看.NET Core 中是如何处理身份验证和授权的。我们将查看 HTTP 授权头支持的各种标准。然后，我们将查看一些广泛使用和广泛支持的开放源代码工具，用于身份验证。最后，我们将探讨如何在 C#应用程序中管理访问控制。

本章将涵盖以下主题：

+   HTTP 请求中有效授权头值支持的多种身份验证方案

+   理解 OAuth 令牌以及如何利用它们进行用户身份验证和授权

+   在.NET Core 应用程序中实现授权方案的策略和设置

# 技术要求

在本章中，就像在前几章中一样，你需要你的 IDE 或你选择的源代码编辑器，以及本章的示例代码 [`github.com/PacktPublishing/Hands-On-Network-Programming-with-C-and-.NET-Core/tree/master/Chapter%2014`](https://github.com/PacktPublishing/Hands-On-Network-Programming-with-C-and-.NET-Core/tree/master/Chapter%2014).

查看以下视频以查看代码的实际运行情况： [`bit.ly/2HY64XC`](http://bit.ly/2HY64XC)

我们还将大量依赖 REST 客户端来对我们的演示 API 发起请求，所以请确保你已经安装了一个。我的两个推荐是 Postman，可以在以下网址找到： [`www.getpostman.com/downloads/`](https://www.getpostman.com/downloads/)，或者 Insomnia REST 客户端，可以在以下网址找到： [`insomnia.rest/`](https://insomnia.rest/)

# 授权头

如果你曾经使用浏览器工具检查过登录到网站的外出请求，你可能会在你的网络检查器的请求头部分注意到一个标题为`Authorization`的头。这个 HTTP 协议中的标准头可以用来指定用于在请求的 URL 上验证和授权用户访问内容的各种方案。如果你不熟悉它，你可能会对你的软件中提供这些基本功能所拥有的选项的多样性感到惊讶。所以，让我们看看这些授权方案是什么，以及我们如何在我们的项目中使用它们。

# 授权与身份验证的比较

在我们探索`Authorization`头时，首先需要考虑的是它能够启用的网络安全中的两个功能。虽然它明确地被命名为`Authorization`头，但在实践中，这通常是一个误称。实际上，它既可以是一个`Authorization`头，也可以是一个`Authentication`头，并且通常它同时具备这两种功能。

现在，如果您对这两个操作之间的区别不太清楚，这听起来可能就像我在吹毛求疵。然而，它们各自提供了对健壮访问控制系统基本不同且基本必要的功能。当我们谈论`Authorization`头时，我们明确地是在谈论控制对软件资源的访问。那么，这些操作究竟是什么，它们如何促进受控访问架构的实现呢？

# 认证

简单来说，认证是验证用户对其身份声明的过程。您的用户正在对其身份做出声明，而您想要确保这个身份是真实的。这通常是通过让用户提供一些只有他们才能合理预期能够拥有的信息来完成的。在受控访问的软件中，这些信息通常是凭证集，例如用户名和密码组合，但它也可能是任何数量的事物，例如对以前地址的了解或家族关系。

认证凭证可以是任何用户应该合理预期能够知道，而其他人则不太可能知道的信息。一旦提供了这些凭证，就可以验证用户声明的真实性。因此，认证是关于验证用户身份的***。***

# 授权

在访问控制硬币的另一面是授权。这是关于确定用户在系统中应该被允许做什么，从修改或添加系统存储的信息到最初访问它。正如其名所示，授权是一种预防措施，不允许用户执行操作，直到系统知道用户有权这样做。

在几乎所有受控访问系统中，授权依赖于认证。在你首先不能验证用户是他们所声称的人之前，你不能确定用户是否有权执行某个操作。尽管认证步骤相当直接，只有两种可能的结果：要么用户是他们所声称的人，在这种情况下认证成功，要么他们不是，在这种情况下认证失败。然而，在授权方面，为特定用户返回的权限可能由任何数量的底层规则、程序或条件决定。因此，虽然认证是关于验证用户的身份，但授权是关于确定用户被允许执行的操作。

因此，现在我们了解了认证和授权在访问控制中扮演的角色，让我们来看看使这些角色得以实现的 HTTP 机制。

# 授权头信息值

当用户尝试访问受控访问的网站时，他们可能会被提示从服务器提供`Authorization`头信息。为了使客户端能够使用服务器准备处理的认证机制进行授权，他们必须这样做。如果服务器仅设置了基本认证，但客户端尝试传递一个携带令牌，服务器将无法解析`Authorization`头信息，并将返回 401-未授权的状态码，无论客户端发送的令牌是否有效。

客户端（在这种情况下，不知道服务器支持的合法授权方案）必须首先被告知它应该使用哪种授权机制来与服务器进行认证。这是通过一个`WWW-Authenticate`响应头信息来完成的，该头信息应用于服务器的初始响应。此头信息用于向客户端指示服务器期望的`Authorization`协议。那么，HTTP 定义的`Authorization`协议究竟是什么，它们是如何工作的呢？

# 基本认证

这个特定的方案是我们第一个例子，展示了`Authorization`头信息在认证任务中的应用。在 HTTP 交互中，**基本认证**（**BA**）方案用于从客户端简单传输用户名和密码到服务器进行认证和授权。它被称为基本认证，因为当服务器请求时，客户端只需传递凭据，无需额外的会话密钥或 cookie，也不需要在服务之间进行额外的握手来适当地设置头信息。

拥有 BA（基本认证），服务器通过返回一个具有以下结构的`WWW-Authenticate`头信息来指示认证方案：

```cs
WWW-Authenticate: Basic realm="{description of the access-controlled area}", charset="UTF-8"
```

在这里，`realm`和`charset`参数在技术上不是必需的，但可以为客户端提供有关如何或为什么他们必须传递其凭据的有用指导。在接收到此头后，客户端负责通过后续请求中的授权头传递其凭据。

# 用户信息 URL 段

我们实际上在第二章*，DNS 和资源定位*的*URL、域名和设备地址*部分中见过第一种方法。如果一个服务器支持基本认证机制，客户端可以完全绕过`Authorization`头，并在 URL 本身中直接传输其凭据。

如您所记得，URL 的第一个段，在方案指定之后，实际上是一个可选段，用于访问凭据。所以，让我们想象有一个用户有权访问远程资源。为了这个例子，让我们假设他们的用户名是`aesop_rock`，密码是`A3h4s9f0cjeC`。在接收到指定基本认证的`WWW-Authenticate`方法后，客户端可以简单地将自己重定向到以下带有凭据的前缀的受控访问 URL：

```cs
 https://aesop_rock:A3hw4s9f0cjeC@test-domain.com/test/url
```

虽然这种格式符合有效 URL 的标准，并且允许在不传递认证头的情况下使用基本认证机制，但应不惜一切代价避免使用。将密码作为目的地 URL 的一部分以纯文本形式传输会带来重大的安全风险。因此，使用此`username:password`格式被认为是过时的，并且通常不被现代网络浏览器支持。然而，大量继续支持基于 URI 的认证技术的服务使得理解并考虑它是值得的。然而，按照规则，您永远不应该支持这种凭据机制的访问。在可能的情况下，您应该在通过电线传输请求之前，从用户信息段中清除任何位于第一个冒号之后的内容，以防止意外持久化用户密码的纯文本记录。

# 带有头值的基本认证

然而，当在请求头中传输凭据时，客户端会使用以下配置的`Authorization`头传输所有后续请求：

```cs
Authorization: Basic <base64-encoded-credentials>
```

在这里，凭据首先以用户名和密码通过单个冒号分隔，就像在 URL 格式中一样，然后进行`base64`字符编码。

对于使用此基本认证格式传递的凭据，第一个冒号始终被解析为用户名和密码字段之间的分隔符。因此，在基本认证系统中，用户名中永远不能包含冒号。

因此，如果我们有之前例子中的用户，用户名为`aesop_rock`，密码为`A3h4s9f0cjeC`，那么我们首先会按照以下方式格式化凭据：

```cs
j_public:A3r9f0cjeC
```

然后我们将字符进行`base-64`编码。将此应用于`Authorization`头，我们将有一个如下所示的头部值：

```cs
Authorization: Basic al9wdWJsaWM6QTNyOWYwY2plQw==
```

现在，我们有一个基本的认证`Authorization`头，当传输到服务器时，服务器将能够通过解码`base-64`编码的访问凭证的值来验证。

# 加密与编码

重要的是要注意，当我们修改我们的凭证时，我们只是在编码它们，而不是真正地加密它们。这里的区别在于，用户可以从`base-64`编码转换为纯文本，然后再转换回`base-64`编码，这一切都不需要任何类型的加密密钥来从一个格式转换到另一个格式。编码仅仅是字符表示的问题。这就像将一个基本的名词从英语翻译成西班牙语，然后再从西班牙语翻译回英语一样。意义并没有被掩盖；只是单词的表示方式不同。与此同时，加密是根据额外的、秘密的输入推导出一个新的、秘密的输入字符串，以完全掩盖原始消息的意义。

到目前为止，我只想指出一个显而易见的事实——任何涉及`Authorization`头的交互都必须通过使用 HTTPS 的安全连接进行。尽管这可能并不总是可行的，但强制执行这种更安全行为的简单最佳实践就是简单地不要配置您的应用程序来监听 HTTP 请求。我们很快就会看到我们还有哪些其他选项可以防止特权信息在网络上未加密、未安全地传输。现在，如果您能够完全避免支持 HTTP，我强烈建议这样做。

# 携带令牌授权

虽然基本认证只是提供了一个原始的用户名和密码凭证，但携带认证处理的是所谓的**携带令牌**。携带令牌只是一个安全令牌，它通知服务器，展示该令牌的用户（携带者）拥有令牌所赋予的凭证和权限。

由于从服务器的角度来看，此令牌用于验证特定用户，因此可以认为携带令牌仅仅是一种不同类型的访问凭证。然而，为了使这种情况成立，服务器必须基于这样的假设：持有特定安全令牌的人除非被授权持有，否则不会拥有它。因此，保护特定携带令牌的价值免受窃听或未经授权的访问与保护用户密码的价值同样重要。应格外小心，确保令牌永远不会以未加密的格式保存在您的服务器上。

在`Authorization`头的上下文中，载体认证方案专门用于支持所谓的**OAuth 令牌**。**开放认证**（**OAuth**）是一个旨在明确支持基于令牌的认证机制，用于基于 Web 资源的访问控制的标准。它在互联网上极其常见。

# OAuth 基础知识

如果你曾经使用谷歌账户、Facebook 或 Twitter 登录过网站，那么你之前已经使用过 OAuth。它是一种访问控制机制，旨在允许代理服务对受控资源进行用户身份验证和授权。因此，当你第一次访问一个新网站时，如果它允许你使用谷歌账户访问，那么这个新网站就是在将验证你的责任委托给谷歌。谷歌请求你的凭证并验证你，你隐式地通知谷歌为新网站提供一个令牌，新网站将认为这个令牌足以进行*授权*。

这与我们在第十三章中学习的**受信任证书机构**（**CA**）系统并没有太大的不同，即**传输层安全性**。在这个背景下，谷歌可以类比为 CA，而用户或令牌的持有者实际上是安全证书的持有者。因此，支持 OAuth 访问的系统是信任 OAuth 提供者（如谷歌、Facebook 等）作为认证的可靠来源。

# 使用载体令牌进行授权

当支持载体令牌时，在`Authorization`头中传递令牌的语法几乎与基本认证相同。这样做的方式如下：

```cs
Authorization: Bearer <token>
```

这里的主要区别在于，虽然基本认证凭证只是`base64`编码，但在载体认证方案中的令牌可以经过加密保护，如果客户端尝试通过不安全的传输机制传递令牌，这提供了一定程度的保护。此外，基本认证凭证将仅包含用户的访问凭证，而载体令牌可以包含更多或更丰富的信息或上下文。

虽然在基本认证方案中仅传递访问凭证就足以验证用户身份，但它将确定该用户可能拥有的任何权限的任务留给了服务器。然而，使用载体令牌时，令牌可能包含有关用户身份验证以及授予用户的权限的信息。因此，如果正确实现，载体认证机制可以更加灵活。

# 摘要认证

使用摘要认证方案，设计者明确寻求提供一种比未加密的基本认证方案更安全的认证实现。然而，实际上，它只提供了一套安全权衡，并且依赖于过时的哈希算法进行加密。尽管如此，它仍然具有许多优点，在某些情况下值得考虑实施。为了了解它引入的优点和缺点，以及何时可能想要使用它，让我们看看它是如何工作的。

# 摘要认证的 WWW-Authenticate 头

当用户尝试访问采用摘要认证的系统时，他们仍然会被提示使用带有`WWW-Authenticate`响应头的`Authorization`头。然而，与基本和载体认证方案返回的简单方案规范不同，摘要认证方案中的`WWW-Authenticate`头包含一系列客户端将用于生成其认证请求的值。

这些头值包括我们在基本认证方案中看到的作为可选参数的`realm`参数，它具有相同的作用，描述了认证空间。还有一个必需的`nonce`值（nonce 的意思是只打算使用一次；在这种情况下，用于一个认证会话），客户端使用它来生成其摘要凭据。这本质上充当了一个共享密钥或会话密钥。除了这两个必需的头参数外，还有几个额外的可选参数。

这些可选参数中的第一个是`qop`，即保护质量——一个如果包含，则规定用户在构建其认证响应时必须采取的额外步骤的规范。此可选字段指定的值可以确保客户端传输的凭据具有更高的安全性。当服务器未指定时，客户端使用的默认操作对于传输来说是最不安全的。有趣的是，**远程过程调用**（**RFC**），该 RFC 标准化了摘要认证（RFC 7616），指定`qop`字段为必需，但其支持和实现因不同的应用服务器而异。

服务器还可以指定一个`domain`字段，该字段可以包含一个由空格分隔的 URI 列表，用于定义受保护空间。当提供此字段时，客户端可以使用该值来确定所有将被视为有效的相同认证信息的 URI。这在分布式系统中非常有用，其中一种资源负责认证，但用户必须将认证信息转发到另一个服务器以访问其受限制的内容。

服务器可以使用`algorithm`参数指定客户端必须用于其摘要响应的特定哈希算法。如果没有指定，则预期算法默认为 MD5，这在许多情况下被认为是不可靠的。因此，如果您在实现摘要认证方案时，我强烈建议您使用响应头中的`algorithm`参数强制执行安全的哈希算法。

此外，服务器还可以提供一个`opaque`值，其目的是由客户端精确地回显。这是一个有用的机制，通过它服务器可以将状态信息从资源传输到另一个通过认证的客户端。例如，如果服务器 A 负责为访问服务器 B 上的资源对客户端进行认证，那么服务器 A 可以通过`opaque`字段将访问详情传输到服务器 B。当正确实现时，客户端只需在其随后的请求中回显服务器 A 发送的任何值即可。

最后，还有一些可选参数用于认证交互的细微细节，例如`charset`，它用于指定客户端可能使用的支持的编码方案。还有一个`userhash`参数，它通知客户端服务器支持对凭据的密码组件进行哈希处理，同时也可以对用户名进行哈希处理。

综合来看，每个必需的和可选的参数都将生成一个类似于以下示例的`WWW-Authenticate`头，正如原始 RFC 中摘要认证所展示的：

```cs
WWW-Authenticate: Digest
    realm="http-auth@example.org",
    qop="auth, auth-int",
    algorithm=SHA-256,
    nonce="7ypf/xlj9XXwfDPEoM4URrv/xwf94BcCAzFZH4GiTo0v",
    opaque="FQhe/qaU925kfnzjCev0ciny7QMkPqMAFRtzCUYo5tdS"
```

现在，为了了解这些头值是如何用来创建摘要凭据的，让我们看看客户端在这个过程中所扮演的角色。

# 摘要认证的授权头

一旦客户端从服务器接收到`WWW-Authenticate`头，他们就有责任使用服务器指定的哈希算法（如果没有指定，则为默认的 MD5）构建他们的摘要响应。为此，他们遵循一系列程序，使用哈希算法对密码进行单向哈希处理，然后对他们的用户名、服务器返回的 nonce 值和他们的密码的组合进行哈希处理。

用户使用他们的用户名、域和密码创建一个哈希值，每个值之间用冒号分隔。假设服务器指定了`SHA-256`，这将创建一个称为`HA1`的值，如下所示：

```cs
HA1 = SHA256(username:realm:password)
```

然后他们生成一个二级哈希值，称为`HA2`，它由 HTTP 方法（用于访问受限制的资源）和指定的 URI 组成，用冒号分隔并用指定的算法进行哈希处理：

```cs
HA2 = SHA256(method:resourceURI)
```

最后，这两个值都会与服务器发送的`nonce`值一起再次进行哈希处理，以生成`Authorization`头中的`response`参数：

```cs
response = SHA256(HA1:nonce:HA2)
```

根据服务器指定的`qop`值，这三个值的具体输入可能会有所不同，但总体交互是相同的。一旦完成所有这些操作，客户端就会向服务器返回一个包含响应值以及它自己的可选参数数组（以及如果服务器最初发送了的话，回声的透明参数）的`Authorization`头，如下所示：

```cs
Authorization: Digest username="Mufasa",
   realm="testrealm@host.com",
   uri="/dir/secured.html",
   response="6629fae49393a05397450978507c4ef1",
   nonce="7ypf/xlj9XXwfDPEoM4URrv/xwf94BcCAzFZH4GiTo0v",
   opaque="FQhe/qaU925kfnzjCev0ciny7QMkPqMAFRtzCUYo5tdS"
```

这种方法的优点是用户永远不会以明文形式发送他们的密码。原因是，由于服务器应该知道用户的密码，它可以使用相同的散列算法，使用相同的输入，并对照`Authorization`头中的`response`参数来确认结果。

当然，你可能已经看到了这种实现固有的风险。为了服务器能够重现相同的`response`参数，它必须能够访问存储在某个地方的用户的明文密码。由于这通常是一个糟糕的想法，更现代的实现通常会首次用户创建一组新凭证时，将整个 HA1 值存储在安全的数据库中。这项技术仍然允许服务器产生`SHA256(HA1:nonce:HA2)`的响应计算，同时消除了对 HA1 的明文输入的需要。然而，这也意味着每次更改域值时，都需要为系统上的每个用户重新计算 HA1。

此外，即使存储`HA1`值也应该被认为相对不安全，因为它被客户端用来创建新的响应并访问数据库仍然会危害底层系统。恶意行为者仍然能够生成一个欺诈请求，即使没有受害者的明文密码。使用`HA1`的最大好处是，在服务器被入侵的情况下，它至少可以保护用户的原始密码不被泄露，从而在安全漏洞事件中最大限度地减少对用户的风险。

尽管摘要认证方案旨在为系统访问的认证步骤提供更高的安全性，但它引入了自己的复杂性和安全担忧。因此，它使用得较少。

# HTTP 源绑定访问

虽然不是一个完全实现的标准，但**HTTP 源绑定访问**（**HOBA**）认证方案代表了访问控制机制设计中的一个令人兴奋的范式转变。我们之前讨论的所有先前形式的认证都是围绕用户提供凭证（通常是用户名和密码的形式）以获取系统访问权限。在 HOBA 中，永远不会传输凭证。相反，客户端（通常是特定的网络浏览器）会持续一个数字签名，该签名在挑战-响应方案中提供给服务器。

在 HOBA 机制下，当客户端第一次尝试访问系统时，该特定客户端必须向服务器注册。在注册过程中，一旦客户端通过传统方式被认证，客户端就会创建一个**客户端公钥**（**CPK**）和一个注册到特定源/域的私钥，用于受限制的资源。然后，CPK 被提供给服务器，服务器将 CPK 分配为客户端的数字签名。正是这个 CPK 用于验证和授权用户随后的请求，以便访问指定域内的受限制资源。

这提供了主要的安全优势，即无需在服务器上存储用户凭据的记录，即使是以安全散列格式存储。然而，这种做法的缺点是，CPK 通常由浏览器在客户端机器的本地存储中存储。这意味着每次用户从不同的机器（或者甚至同一台机器上的不同网络浏览器）访问服务器时，客户端都必须再次将新的用户代理注册到服务器。幸运的是，RFC 草案，该草案定义了 HOBA 规范，明确允许服务器将多个 CPK 注册到单个客户端账户。随着对这种身份验证方案的支持增长以及规范的正式标准化，我预计其他身份验证模式将很快被 HOBA 所取代。

# 授权令牌

最常用的授权机制之一是携带令牌。发行这些令牌最常见的方式是使用 OAuth。然而，尽管携带令牌通常被称为 OAuth 令牌，但实际上，它们只是由 OAuth 提供的。令牌本身可以由任何标准定义，或者根本不定义标准。让我们看看这种令牌与令牌发行者之间的关系如何展开。

# OAuth 令牌提供

如我之前提到的，OAuth 是提供客户端有效身份验证令牌的标准。这种做法的正式标准实际上相对简短且高度概括，允许在具体实现中具有很大的灵活性。它最初被设计为允许第三方应用程序代表用户访问目标应用程序。

在 OAuth 交互方案中，客户端必须首先注册为资源服务器的消费者。在这个上下文中，资源服务器简单地是指任何包含受限制资源的服务器，并依赖于代理服务器来处理其访问控制。在注册过程中，资源服务器授予客户端未来访问受限制资源的相关权限。

一旦客户端注册，每当该客户端尝试在未来访问资源服务器时，它都会被提示获取一个 **访问令牌**。然后，客户端必须从 **授权服务器** 请求访问令牌。授权服务器是资源服务器已注册为其代表的服务器，并实现了 OAuth 交互标准。客户端提供在注册步骤中由资源服务器给予的任何访问凭证。

在验证访问凭证后，授权服务器返回一个访问令牌。OAuth 2.0 的访问令牌响应构成了大多数用户日常与标准交互的基础。每当一个有效用户成功请求令牌时，他们都会收到包含以下属性的响应体：

+   **access_token**：此属性是必需的，原因很明显。它是将被返回给资源服务器的令牌字符串的值。

+   **token_type**：这表示资源服务器将期望的具体令牌授权机制。在几乎所有情况下，此值将简单地是 bearer。

+   **expires_in**：虽然此属性不是必需的，但强烈建议使用。访问令牌应该是短暂的，OAuth 规范建议大多数令牌的最大生存期仅为 10 分钟。这是一个安全预防措施，以减少暴露的访问令牌被恶意行为者使用的风险。

+   **refresh_token**：此属性仅在访问令牌有到期时间时使用，即使在这种情况下也是可选的。它指定了一个刷新令牌，客户端可以使用它从授权服务器请求新的访问令牌。

+   **作用域**：此字段用于通知用户，如果他们所授予的权限集合比他们最初请求的权限集合更少，则使用。例如，如果我请求一个具有读取、写入和更新权限的访问令牌，但服务器只限制我的访问权限为只读，则此字段将用于指定读取权限。

最后，客户端使用访问令牌作为其 `Authorization` 携带令牌向资源服务器发送请求。

# 令牌生成

虽然 OAuth 规定了客户端与相应资源及授权服务器之间的交互，但它并未说明 `access_token` 字段是如何实际生成的。相反，这一细节留给了每个实现或依赖 OAuth 的服务器。那么，这些令牌是如何生成和验证的呢？

# 持久令牌

令牌仅仅是随机生成的字符串的情况并不少见。在这种令牌生成机制中，授权服务器和资源服务器共享一个数据库，其中包含所有相关的用户访问凭证和权限。在预定的时间表上，随机生成令牌并将其与数据库中的每个用户关联。然后，在成功认证后，授权服务器查找给定用户的当前令牌并将其作为访问令牌返回。在授权后，资源服务器随后查找数据库中具有相同令牌值的用户并验证其访问权限，并可以查找他们可能拥有的任何权限。

这种方法的明显缺点是需要共享数据库，并且在认证服务器和资源服务器之间在该数据库中存在并发性。这种方法并不特别适合可能同时运行数十个资源或授权服务器实例的云部署系统，这些实例具有多个数据库实例。它也不特别适合极高流量的应用程序，因为多次数据库查找可能会极大地影响应用程序的响应速度。然而，如果你的系统相对集中且处理量合理，那么在实现访问令牌系统时，这绝对不是一种糟糕的选择。那么，替代方案是什么呢？

# 自编码令牌

可能最稳健且最有用的令牌生成机制之一是自编码令牌。自编码令牌是指其主体包含所有必要信息，以便资源服务器授权持有者的令牌。因此，当用户通过授权服务器认证并授予令牌后，资源服务器可以简单地检查令牌的主体以确定授权请求的成功以及用户拥有的任何权限或声明。由于上下文完全包含在令牌中，资源服务器无需访问共享数据库来验证用户或其权限，从而在分布式环境中节省了资源编排。

可能最广泛使用且广泛支持的自我编码令牌方案是**JSON Web Token**（JWT）。JWT 通过在编码的 JSON 对象中提供一组作为属性声明的持有者声明来实现自编码令牌机制。JWT 令牌的结构由三个独立的部分组成，这些部分连接在一起，并由句点分隔。

这些组件中的第一个是头部，它是一个`base-64`编码的字符串，其中包含作为 JSON 对象表示的已知头部参数。这指定了用于签名的算法（我们稍后将讨论）和令牌类型，通常是 JWT。

接下来是正文，它是一个`base-64`编码的 JSON 对象，包含令牌持有者拥有的完整声明列表。在这个对象中有一些必需的参数，例如 sub（主题，或持有者的名称）和 iat（发行时间）。然而，由于令牌正文的规范完全由资源服务器自行决定，因此可以添加任何数量的声明，并赋予任何给定的值。

JWT 的最后一部分是验证签名。这是资源服务器如何知道令牌来自其关联的授权服务器的方式。验证签名是一个加密的**散列消息认证码**（**HMAC**），它是由前两个部分的`base-64`编码字符串以及一个密钥组合而成的。如果令牌的正文或头部的任何部分被修改，那么令牌授权的验证步骤将不会产生相同的 HMAC，资源服务器可以找出令牌是否被篡改并且无效。然而，为了使这起作用，资源服务器和授权服务器必须在两者之间共享密钥，并使用它来签署和随后验证在两者之间传递的任何令牌。

无论令牌是否被确定为有效，令牌的前两个组件仅仅是`base-64`编码的。因此，它们可以很容易地被任何感兴趣的一方解码和读取。出于这个原因，永远不要在 JWT 的正文以明文形式发送私人信息。如果必须发送机密信息或访问凭证，它们应该以安全加密格式发送，当可能时使用单向哈希，否则使用安全的可逆加密。

# .NET Core 中的授权

现在我们已经探讨了 OSI 堆栈应用程序层授权的方方面面，是时候看看我们如何可以利用这些特性在我们的应用程序中。幸运的是，正如强制执行 SSL 一样，在我们的 Web API 上启用授权方案主要是配置问题，而不是其他。为了这个演示的目的，我们将创建一个应用程序，它将作为我们的资源服务器和认证服务器。而且，一如既往地，我们将为了简单起见在错误处理和健壮的应用程序设计方面采取一些捷径。

# `AuthorizeAttribute`

我们首先需要确定哪些资源需要授权访问。稍后我们将处理授权用户的实际过程。为了指定受限资源，我们应用`AuthorizeAttribute`。这个属性被指定为适用于方法或类。这意味着我们可以直接将其应用于我们应用程序的受限端点或整个控制器。因此，让我们看看每种方法的影响。首先，让我们使用 CLI 命令创建我们的应用程序：

```cs
dotnet new webapi -n AuthSample
```

然后，我们将导航到`ValuesController.cs`文件，将其重命名为`AuthController.cs`，然后修改如下：

```cs
[Route("api/[controller]")]
[ApiController]
public class AuthController : ControllerBase {

  [HttpGet("secret")]
  public ActionResult<string> GetRestrictedResource() {
    return "This message is top secret!";
  }

  [HttpPost("authenticate")]
  public void AuthenticateUser([FromBody] Credentials creds) {
  }
}
```

我们希望允许用户在认证时`POST`他们的凭证，但他们将简单地`GET`我们的绝密信息。在这里，我创建了一个简单的`Credentials`类，作为我们`POST`请求的消息体。这只是为了方便将`username`和`password`字符串封装在单个容器类中。现在，让我们看看应用`Authorize`属性的不同方法。一种方法是将特定的端点显式标记为需要授权。在这种情况下，这将是我们`GetRestrictedResource()`方法。因此，我们可以在`[HttpGet]`属性之上或之下应用该属性，然后就这样！让我们看看以下代码：

```cs
[Authorize]
[HttpGet("secret")]
public ActionResult<string> GetRestrictedResource() {
    return "This message is top secret!";
}
```

然而，我相信你可以想象一个场景，其中控制器有数十个端点，每个端点都需要授权。在这种情况下，你可以简单地将`Authorize`属性应用于控制器类本身，如下所示：

```cs
[Route("api/[controller]")]
[ApiController]
[Authorize]
public class AuthController : ControllerBase {
```

通过将属性应用于控制器，它将自动应用于该控制器中定义的每个端点。当然，在我们的当前控制器中，我们需要某种方式来指定我们的`AuthenticateUser()`方法不需要授权（毕竟，如果用户必须首先被授权，他们如何成为授权用户呢？）。为此，我们可以用方法级别的`AllowAnonymous`属性覆盖控制器级别应用的`Authorize`属性：

```cs
[HttpPost]
[AllowAnonymous]
public void AuthenticateUser([FromBody] Credentials creds) {
}
```

这个属性将始终覆盖应用于当前方法的任何`Authorize`属性。因此，始终确保你只在绝对必要时应用`AllowAnonymous`属性是非常重要的。

# 授权中间件

现在我们已经指定了哪些资源需要使用授权来访问，是时候看看在我们的代码中这个授权步骤看起来是什么样子了。对于这个应用程序，我们将利用 JWT 令牌认证。为了利用这一点，我们将进入我们的`Startup.cs`文件，并修改我们配置的服务以使用适当的认证方案。

由于其开源分布和广泛的支持，`Microsoft.AspNetCore`和`Microsoft.IdentityModel`命名空间直接支持 JWT 令牌库。这将使定义我们的认证行为变得更加容易。我们将在`IServicesCollection`上调用`AddAuthentication()`方法，并使用`JwtBeareDefaults`库类应用默认的 JWT 令牌认证方案。让我们看看以下代码：

```cs
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
```

这个`AddAuthentication()`方法是你可以在给定的`Authorize`端点上定义任何自定义访问策略的地方。例如，如果你想为基于角色的认证方案定义一个策略，你可以通过将其作为选项添加到`AddAuthentication()`中间件来定义一个需要`Manager`权限的策略：

```cs
services.AddAuthorization(options => {
  options.AddPolicy("RequireManagerRole", 
    policy => policy.RequireRole("Manager"));
});
```

然后，对于任何你想要限制管理员访问的端点，只需在`Authorize`属性中定义该访问策略，如下所示：

```cs
[Authorize(Policy = "RequireManagerRole")]
```

当你有许多不同的访问策略并且需要一次性应用许多策略时，这可能会变得很麻烦。在这种情况下，你通常会发现自己正在编写自己的`IServicesCollection`类的扩展方法，并直接调用它们。然而，对于这个例子，我们已经有一个来自`Microsoft.AspNetCore.Authentication.JwtBearer`命名空间的扩展方法来定制我们的授权。

只需将我们的认证方案定义为 JWT 默认值，这意味着任何带有`Authorize`指令的端点都会调用.NET 的 JWT 验证代码。我们不需要为我们的应用程序编写任何其他代码来从`Authorization`头中提取令牌值，将其签名与我们的私钥记录进行验证，然后批准请求继续到方法，或者以 401 响应拒绝它。所有这些操作都是通过在`AddAuthentication()`方法中定义`JwtBearerDefaults.AuthenticationScheme`来完成的。

我们要采取的唯一剩余步骤是定义什么应该被视为有效的令牌。我们将通过使用`AddJwtBearer()`扩展方法来完成这项工作。此方法允许我们定义一个动作委托来配置我们希望我们的`Authentication`代码在验证令牌时使用的选项。对于这个示例代码，我已经将签名密钥、令牌发行者和令牌受众的定义移动到了一个名为`SecurityService`的静态实用工具类中。这样做只是为了使获取相同的值以供我们的令牌验证选项和令牌生成代码更容易，我们将在稍后查看这些代码。但如果你好奇，这个类所做的只是为我们的令牌的一些关键组件返回一致的价值：

```cs
public static class SecurityService {
  public static SymmetricSecurityKey GetSecurityKey() {
      string key = "0125eb1b-0251-4a86-8d43-8ebeeeb39d9a";
      return new SymmetricSecurityKey(Encoding.ASCII.GetBytes(key));
  }

  public static string GetIssuer() {
      return "https://our-issuer.com/oauth";
  }

  public static string GetAudience() {
      return "we_the_audience";
  }
}
```

因此，通过使用这个类来生成我们的共享对称密钥、发行者和受众，我们可以配置令牌的这些方面，使其成为授权有效用户所必需的。只需将它们应用到`JwtBearerOptions`的`TokenValidationParameters`类中，如下所示：

```cs
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
  .AddJwtBearer(options => {
    options.TokenValidationParameters = new TokenValidationParameters {
      IssuerSigningKey = SecurityService.GetSecurityKey(),
      RequireSignedTokens = true,
      ValidateActor = false,
      ValidateAudience = true,
      ValidAudience = SecurityService.GetAudience(),
      ValidateIssuer = true,
      ValidIssuer = SecurityService.GetIssuer()
    };
  });
```

通过这种配置，你可以看到我们的应用程序负责根据它包含的声明来确定携带令牌的有效性。但是，多亏了 JWT 实用工具库中可用的扩展，定义这些参数变得非常简单！

最后，就像在`ConfigureServices`方法中配置的任何中间件一样，我们需要告诉我们的应用程序通过在`Configure`方法中添加以下行来利用它：

```cs
app.UseAuthentication();
```

现在，如果你运行应用程序并尝试访问`/auth/secret`端点，你会收到一个 401 响应，其中`WWW-Authenticate`消息指示预期的认证方案：

![](img/b565513a-6916-49b9-b58b-86b798ddb0cb.png)

现在，剩下的只是给我们的用户提供一个令牌。让我们看看如何在我们的`AuthController`类中使用 JWT 库来做到这一点。

# 生成令牌

在我们的`/auth/secret`端点安全地锁在`authorize`属性后面之后，我们需要一种方法来验证和授权用户。为此，我们将使用`AuthenticateUser()`方法，该方法配置了`AllowAnonymous`属性以允许任何人尝试登录。我们首先需要的是一个用户列表。为此，我创建了一个简单的`username:password`组合字典，存储在`user_vault.json`文件中，我们可以通过静态的`UserVault`类来访问它。然后，`UserVault`类公开了一个简单的方法来检查我们的用户数据库中是否存在`username:password`组合。所以，让我们让`user_vault.json`定义如下：

```cs
{
    "aladdin": "open_sesame",
    "dr_suess": "green_eggs_and_ham",
    "jack_skellington": "halloween"
}
```

这为我们提供了三个有效的用户，我们可以用它们进行测试。我们的`UserVault`类允许我们通过首先使用静态构造函数初始化来检查这一点：

```cs
private static Dictionary<string, string> _users { get; set; }
static UserVault() {
  try {
    using (var sr = new StreamReader("user_vault.json")) {
      var json = sr.ReadToEnd();
      _users = JsonConvert.DeserializeObject<Dictionary<string, string>>(json);
    }
  } catch (Exception e) {
    throw e;
  }
}
```

然后，使用我们`user_vault.json`文件的内存表示，我们可以通过`ContainsCredentials()`方法检查我们的私有字典中是否存在任何`username:password`对：

```cs
public static bool ContainsCredentials(string userName, string password) {
  if (_users.ContainsKey(userName)) {
    string storedPassword;
    if(_users.TryGetValue(userName, out storedPassword)){
      return storedPassword.Equals(password);
    }
  }
  return false;
}
```

因此，现在我们能够检查用户是否在我们的用户数据库中，让我们继续定义我们应用程序的认证。我们首先需要做的是构建我们的授权代码将期望从任何我们签发的令牌中获取的关键细节。因此，我们需要使用我们配置认证中间件期望的相同安全密钥。我们还将使用`ClaimsIdentity`类给用户一个基本的`Identity`：

```cs
[HttpPost]
[AllowAnonymous]
public ActionResult<string> AuthenticateUser([FromBody] Credentials creds) {
  if (UserVault.ContainsCredentials(creds.UserName, creds.Password)) {
    var key = SecurityService.GetSecurityKey();
    var signingCredentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
    var identity = new ClaimsIdentity(new GenericIdentity(creds.UserName, "username"));
```

在此代码中，`SigningCredentials`类是一个简单的包装类，它处理使用指定的哈希算法（在这种情况下为 HMAC SHA-256）对给定的安全密钥进行哈希处理的细节。然后，这个类被用来生成结果 JWT 令牌的签名密钥组件。一旦我们有了这个，我们就可以构建我们的 JWT 令牌。为此，我们将使用`JwtSecurityTokenHandler`类的实例。这本质上是一个工厂类，用于使用传递给它的配置详细信息生成格式良好的 JWT 令牌。

# 应用用户声明

为了演示的目的，我们将对我们的令牌应用一些任意的声明，这样我们就可以检查结果并看到这些声明是如何在格式良好的令牌中应用和显示的。为此，我们有一个简单的辅助类，它返回一个基本的`Claim`实例列表：

```cs
private IEnumerable<Claim> GetClaims() {
  return new List<Claim>() {
    new Claim("secret_access", "true"),
    new Claim("excellent_code", "true")
  };
}
```

因此，现在我们已经定义了所有的声明，我们可以将这些声明应用到我们为用户创建的`ClaimsIdentity`上，如下代码所示：

```cs
identity.AddClaims(GetClaims());
```

这将使我们能够根据用户在特定情况下的权限来限制对单个资源的访问。对于任何给定的请求，`HttpContext`的`User`属性是一个`ClaimsPrinciple`的实例。因此，我们可以通过`HttpContext.User.Claims`属性检查用户的声明。

当我们使用授权过滤器保护端点时，由认证机制（在这种情况下，JWT）指定的任何用户声明都将应用于`HttpContext.User.Claims`属性。考虑到这一点，我们可以通过检查用户的`Claims`属性并确认它们包含我们正在寻找的声明来限制对我们的秘密端点的访问。只需修改你的`GetRestrictedResource`方法，检查用户声明的`Type`参数，并确认用户至少有一个由`GetClaims`返回的有效声明类型，如下所示：

```cs
[HttpGet("secret")]
public ActionResult<string> GetRestrictedResource() {
    var validClaims = GetClaims().Select(x => x.Type);
    var userClaims = HttpContext.User.Claims.Select(x => x.Type);
    if (validClaims.Intersect(userClaims).Count() < 1) {
        return StatusCode(403);
    }
    return "This message is top secret!";
}
```

有了这些，我们已经定义了用户，指定了他们的声明，并将我们的资源限制为具有那些声明的用户。由于我们的签名凭证配置为使用与资源服务器相同的对称密钥，我们准备好构建我们的令牌。这可以通过`JwtSecurityTokenHandler`和`SecurityTokenDescriptor`类轻松完成：

```cs
var handler = new JwtSecurityTokenHandler();
var token = handler.CreateToken(new SecurityTokenDescriptor() {
  Issuer = SecurityService.GetIssuer(),
  Audience = SecurityService.GetAudience(),
  SigningCredentials = signingCredentials,
  Subject = identity,
  Expires = DateTime.Now.AddMinutes(10),
  NotBefore = DateTime.Now
});
```

现在，我们只剩下将令牌写入我们的输出，供用户在后续请求中应用。如果你想实现一个完全符合 OAuth 标准的服务器，你的响应体和异常处理需要遵循 OAuth 2.0 定义的标准，你的令牌作为更大响应体的一部分返回。由于我们只是在演示高级工作流程，我将这项额外的研究留给你。相反，我们的输出将只是原始的 JWT 令牌。所以，通过我们刚刚完成的所有操作来填补空白，我们的认证方法外壳应该看起来像这样：

```cs
[HttpPost]
[AllowAnonymous]
public ActionResult<string> AuthenticateUser([FromBody] Credentials creds) {
  if (UserVault.ContainsCredentials(creds.UserName, creds.Password)) {
    ... // Build and generate JWT token
    return handler.WriteToken(token);
  } else {
    return StatusCode(401);
}
```

现在，我们准备将所有这些整合起来。运行应用程序，向认证端点发送一个请求，使用`user_vault.json`中的任何凭证，你应该会收到一个令牌：

![](img/e5ee7a7b-ee40-46af-8656-75a4fc31d624.png)

接下来，向秘密端点发送一个请求，提供你新检索到的令牌。为此，转到 Postman（或 Insomnia）的授权选项卡，并将下拉列表中的授权类型设置为 Bearer Token 选项。然后，将你新获得的令牌粘贴到输入框中，并发送请求。如果你遵循了这里的所有步骤，你应该会收到带有 200 状态码的秘密消息：

![](img/e54e94ad-ed86-4c89-bf03-f8ef20c82954.png)

就这样，你已经在.NET core 中实现了并应用了一个完整的认证和授权方案，这需要你最少量的自定义代码。

在我们继续之前，然而，我想明确指出，这里使用的编码实践仅用于演示目的**。**我的目标仅仅是展示.NET Core 中认证/授权框架的概念流程和基本模式。如果你发现自己正在任何应用程序中实现用户访问，你有责任确保他们的私有访问凭证的安全，这一点绝不能轻视。

话虽如此，我们现在可以探讨其他方法来提高我们应用层网络代码的性能和灵活性。在下一章中，我们将仔细研究.NET Core 中的缓存策略和模式。

# 摘要

在本章中，我们涵盖了应用层认证和授权的各个方面。首先，我们学习了认证和授权之间关键但细微的区别。我们研究了允许应用程序执行这两项任务以控制对受限资源访问的 HTTP 标准头。然后，我们学习了标准`Authorization`头支持的每个有效认证方案。

我们看到了基本认证带来的安全风险和实现上的便捷。我们研究了载体令牌认证如何减轻与基本认证相关的某些安全风险，而无需增加太多复杂性。最后，我们学习了摘要认证机制的复杂性和细微差别。在继续之前，我们还花时间考虑了未来可能通过类似 HOBA 方案来处理授权的方式。

接下来，我们深入探讨了载体令牌。我们了解了 OAuth 标准如何定义一个用于访问和提供令牌的交互机制。然后，我们研究了这些令牌如何由资源服务器生成和利用。最后，我们通过学习如何在.NET Core 中实现这些功能，将所有这些内容串联起来。现在，我们准备通过缓存来查看性能改进和弹性策略，这些内容将在下一章中进行探讨。

# 问题

1.  认证和授权之间有什么区别？它们是如何应用于基于 Web 资源的访问控制的？

1.  对于授权 HTTP 头，哪些是有效的认证机制？

1.  HOBA 是什么？它相对于其他认证方案的主要优势是什么？

1.  载体令牌是什么？它是如何用于认证或授权的？

1.  OAuth 是什么？它与标准授权技术有何关联？

1.  什么是自编码令牌？

1.  JWT 是什么？它是如何使用的？

# 进一步阅读

关于现代认证技术，我强烈推荐阅读*Mastering OAuth 2.0*一书，作者是*Charles Bihis*。这是一本相对简短的读物，但提供了对标准的广泛探索。您可以通过 Packt 在这里找到它：[`www.packtpub.com/application-development/mastering-oauth-2`](https://www.packtpub.com/application-development/mastering-oauth-2)

或者，如果您打算在自己的软件中利用 OAuth 系统，并希望对这个主题有更实际的了解，我推荐阅读由**Adolfo Eloy Nascimento**所著的《*OAuth 2.0 烹饪书*》。您也可以通过 Packt 购买，链接如下： [`www.packtpub.com/virtualization-and-cloud/oauth-20-cookbook.`](https://www.packtpub.com/virtualization-and-cloud/oauth-20-cookbook)
