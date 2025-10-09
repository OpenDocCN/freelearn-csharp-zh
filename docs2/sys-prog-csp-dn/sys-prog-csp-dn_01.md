# 1

# 拥有底层秘密的那一个

*理解* *底层 API*

编写软件可能是一项艰巨的任务。当您试图将您的想法转化为机器上可以运行的东西时，您需要考虑许多因素。毕竟，在计算机执行任何有用的操作之前，您需要告诉计算机很多事情。

但我们很幸运。我们需要提供给 CPU 的许多指令都被封装在框架、工具、包和其他软件组件中。这些构建块使我们能够专注于我们想要构建的内容，而不是 CPU 如何解释我们的指令。这使得生活变得更加容易！

本章探讨了这些构建块，它们如何帮助我们，以及我们如何最好地使用它们。本章还涵盖了.NET 的工作原理及其来源。这很重要：大多数开发者都理所当然地认为.NET 具有优势。这是可以接受的，因为框架隐藏了许多复杂性。然而，在编写底层系统软件时，了解.NET 中的事物为何如此工作以及如何在需要时使用其他解决方案至关重要。此外，偶尔提醒一些基本的事情也无妨，尤其是在您可能需要偏离面向用户的软件开发者所走的道路时。 

因此，我们将涵盖以下主题：

+   什么是底层 API？

+   基类库（BCL）如何帮助我们.NET 开发者？

+   什么是公共语言运行时（CLR）？

+   Win32 API 是什么，我们如何调用它们？

总而言之，我们将深入探讨并全面了解技术。

但在我们深入探讨.NET 生态系统为我们提供的构建块之前，我们需要讨论 API——更准确地说，是底层 API 和高级 API 之间的区别。

# 技术要求

您可以访问以下链接查看本章中所有代码：[`github.com/PacktPublishing/Systems-Programming-with-C-Sharp-and-.NET/tree/main/SystemsProgrammingWithCSharpAndNet/Chapter01`](https://github.com/PacktPublishing/Systems-Programming-with-C-Sharp-and-.NET/tree/main/SystemsProgrammingWithCSharpAndNet/Chapter01)。

# 底层 API 是什么，它们与高级抽象有何不同？

嗯，也许我们走得有点快了。在我们能够查看底层和高级 API 之前，我们需要就 API 的含义达成一致。

API 是应用程序编程接口的缩写。虽然技术上正确，但它并没有告诉我们太多。我们需要一个更好的 API 定义。

接口是什么？

让我们从术语**接口**开始。仅凭这一点，根据您询问的人的不同，就可以完全不同地定义。

接口可以是一个**软件接口**，它是两块软件之间的边界。例如，SQL Server 这样的数据库允许用户通过接受 SQL 查询来访问数据。这就是该数据库系统的主要接口。

接口的一个另一种定义将是**硬件接口**。您电脑上的 USB 端口以及您通过它们连接到机器的外设都是硬件接口。

当然，在 C#中，我们也有接口。大多数面向对象编程语言以某种方式支持接口。例如，C++有纯虚类的概念。Python 支持抽象基类，它们具有相同的目的。

API 是软件与程序员使用的其他软件之间的接口。这定义了给定代码集的边界以及如何与之交互。

因此，可以创建一个充满奇妙且高度复杂代码的巨大库。作为库用户，你将获得一系列方法、类、接口（是的，C#类型的）、枚举以及其他与该库交互的方式。

这很棒，因为你可以在不担心编写代码的情况下使用那个库。

底层和高级 API

API 的级别是一个任意的区分，用来给你一个 API 实际硬件接近程度的想法。

没有任何度量标准告诉我们何时某个东西是底层或高级 API。一切都是相对的，并且可以公开讨论。然而，这里我们不会这样做。

通常，底层 API 比高级 API 提供对硬件更细粒度的控制。然而，高级 API 通常更易于移植，并且可以更快地实现目标。

如果这一切听起来有点抽象，不要担心。让我通过一些例子来澄清这一点。例如，想象你想通过网络发送一些数据。当我说网络时，我的意思是我们将它发送到 IP 地址`127.0.0.1`。换句话说，我们发送到本地主机；我们在和自己说话。

要做到这一点，我们需要调用 Windows SDK 为我们提供的许多底层 API。代码看起来是这样的：

```cs
static void UseLowLevelAPI()
{
    WSAData wsaData;
    if (WSAStartup(0x0202, out wsaData) != 0)
    {
        Console.WriteLine("WSAStartup failed");
        return;
    }
    IntPtr sock = socket(2 /* AF_INET */, 1 /* SOCK_STREAM */, 0);
    if (sock == new IntPtr(-1))
    {
        Console.WriteLine("socket() failed");
        WSACleanup();
        return;
    }
    sockaddr_in sin = new sockaddr_in();
    sin.sin_family = 2; // AF_INET
    sin.sin_port =(ushort)IPAddress.HostToNetworkOrder((short)8000);     // Port 8000
    sin.sin_addr = BitConverter.ToUInt32(IPAddress.Parse("127.0.0.1")    .GetAddressBytes(), 0);
    if (connect(sock, ref sin, Marshal.SizeOf(typeof(        sockaddr_in))) != 0)
    {
        Console.WriteLine("connect() failed");
        closesocket(sock);
        WSACleanup();
        return;
    }
    byte[] data = Encoding.ASCII.GetBytes("Hello, server!");
    if (send(sock, data, data.Length, 0) == -1)
    {
        Console.WriteLine("send() failed");
    }
    closesocket(sock);
    WSACleanup();
}
```

正如你所见，为了完成这样一个相对简单的任务，必须发生许多事情。我已经省略了所有我们需要访问 API 以及类和结构体（如`WSAData`）定义的代码。我还简化了这个示例，没有使用很多错误处理或内存管理。

我不会解释前面代码中发生的事情，因为它不是我要展示的内容的一部分。我们将在本书讨论网络时再次回到这个话题。我提供这段代码是为了展示底层 API 的样子。在这里，我希望你注意对`WSAStartup()`、`WSACleanup()`、`socket()`、`connect(`)、`send()`和`closesocket()`的调用。这些来自 Windows SDK 的 API。它们是 Windows 中帮助我们设置网络接口连接、转换地址、打开和关闭套接字以及发送数据的部分。

注意

记住 Windows SDK 是一个包装器是很好的。SDK 内部的代码，主要用 C 语言编写，部分用 C++编写，执行了繁重的任务并调用硬件。我们不必担心这一点：微软的人已经想出了如何完成所有这些。

就像我说的一样，低级和高级术语取决于你如何看待它们。这完全是相对的。当你从 C 程序员的视角来看时，他们必须做所有繁重的工作，你可以将 Windows SDK API 视为高级 API。

但作为.NET 开发者，我们将其视为相当低级。这是因为，作为.NET 开发者，我们有更易于使用的工具。前面的代码不是大多数开发者会写的。相反，他们会写以下内容：

```cs
static void UseHighLevelAPI()
{
    try
    {
        // Connect to server at 127.0.0.1:8000
        using (TcpClient client = new TcpClient("127.0.0.1", 8000))
        using (NetworkStream stream = client.GetStream())
        {
            // Prepare the message
            byte[] data = Encoding.ASCII.GetBytes("Hello, server!");
            // Send the message
            stream.Write(data, 0, data.Length);
            Console.WriteLine("Sent: Hello, server!");
        }
    }
    catch (SocketException e)
    {
        Console.WriteLine($"SocketException: {e}");
    }
    catch (IOException e)
    {
        Console.WriteLine($"IOException: {e}");
    }
    catch (Exception e)
    {
        Console.WriteLine($"Exception: {e}");
    }
}
```

这段代码更容易且更小。前面的大部分代码都是捕获异常。

`TcpClient`类正在做艰苦的工作。我们实例化它的一个实例，给它我们想要连接的地址，从它那里获取一个`NetworkStream`实例，然后写入一些字节。很简单。它工作得非常出色。

那么，你为什么要关心这些低级的东西呢？

尽管低级代码工作量更大且更复杂，但它给你一个显著的优势：更多的控制。

我们在这里使用 TCP/IP。但如果你想要通信的设备没有 TCP 怎么办？在你还没来得及说“现在所有东西都是基于 IP 的”之前，我非常确信你家里的电脑可能还在使用较旧的技术进行通信。你可能每天都在使用没有 TCP 的设备。我指的是大多数电视机的遥控器。它们使用红外线。许多设备仍然使用红外线。它成本低，易于理解，安装快速，在用例方面稳健。它也不受.NET 支持。

但当涉及到低级 API 时，它相当简单。在设置连接的方式上存在一些差异：没有 IP 地址，所以你必须使用设备 ID，但连接本身并不难使用。看看我们设置`socket()`调用的那行代码。我们使用`2`作为第一个参数，它代表`AF_INET`，意味着 TCP。将其改为`26 (AF_IRDA)`，底层库就会切换到红外设备。

这不能用可用的.NET 库来完成。

高级 API 非常出色，帮助我们快速编写易于理解的代码。然而，作为系统程序员，我们必须处理硬件和其他低级系统。那时我们就必须使用低级 API。

在我们深入探讨如何使用这些 API 之前，让我们看看.NET 库本身。在此过程中，我们将检查 CLR，以便你对.NET 能给我们带来什么有一个清晰的了解。

# .NET Core 运行时组件概述（CLR、BCL）

之前，我们探讨了低级和高级编程语言之间的区别。就像 API 一样，低级和高级意味着你离实际机器有多近或有多远。用 C 语言编程意味着你非常接近硬件；用 C#编程意味着你远离。当然，距离越远意味着你在抽象层面工作。优点是许多事情都简化了，正如本章前面所看到的。此外，由于许多抽象，将你的代码迁移到其他平台更容易管理。

使这一切成为可能的是.NET 运行时。从第一个版本开始，设计者就始终致力于尽可能多地屏蔽低级内容。这让你可以快速编写代码，专注于功能而不是样板代码。

.NET 是一个复杂的话题。但简而言之，它是一套以多种形式存在的工具，帮助你实现目标。

有趣的事实

在其最初发布之前，该项目的代码名称是 Project 42。42 是来自科幻作家道格拉斯·亚当斯的书、电视剧和主要科幻电影《银河系漫游指南》中，对生命、宇宙和万物之答案。亚当斯写道，42 是万物的答案；因此，.NET 设计者认为将所有开发者问题的解决方案命名为 Project 42 是合适的。

.NET 并不能解决所有问题，但它使生活变得更加容易。让我们看看它是如何做到这一点的。

我们可以确定.NET 帮助我们解决三个主要领域：

+   开发工具

+   CLR

+   BCL

我不会在这里花费时间讨论开发工具。相反，我想讨论 CLR 和 BCL。这两个构成了.NET 生态系统的核心。在后面的章节中，我将介绍.NET 生态系统的其他重要部分，例如**公共类型系统**（**CTS**）。

## CLR

CLR 是我们代码运行的运行时环境。

编译器（本书后面将详细介绍）编译我们编写的代码。目前，我们可以想象编译器将我们人类可读的文本转换为计算机可以理解和使用的某种形式。

嗯，并不完全是这样。我需要在这里澄清一些事情。虽然我在讨论编译器时写的内容在技术上是对的，但这只适用于真正的编译器，例如使用 C 或 C++找到的编译器。这并不适用于.NET 世界。

.NET 编译器在针对通用运行时而不是我们运行的硬件进行编译。

编译器的输出不是针对硬件的。相反，它输出一种称为中间语言（IL）的东西。这是一种“中间”形式。它不是人类可读的，但对于计算机来说又过于抽象。它介于这两种形式之间。

让我通过一个例子来澄清这一点。

我编写了一个没有任何顶层语句的.NET 控制台应用程序。换句话说，这是我们使用.NET 可以编写的最简单的代码片段。整个程序由一行代码组成：

```cs
Console.WriteLine("Hello, System Programmers!");
```

我不需要解释我在这里做什么，对吧？

如果我们使用 Visual Studio 来编译代码，它将获取所有我们的文件，将它们交给编译器，并指示它构建一个二进制文件。这看起来是这样的：

```cs
.method private hidebysig static void  '<Main>$'(string[] args) cil managed
{
  .entrypoint
  // Code size       12 (0xc)
  .maxstack  8
  IL_0000:  ldstr      "Hello, System Programmers!"
  IL_0005:  call       void
  [System.Console]System.Console::WriteLine(string)
  IL_000a:  nop
  IL_000b:  ret
} // end of method Program::'<Main>$'
```

这有点难以理解，但并不太难。首先，有一些代码用于设置环境（`.maxstack 8`）。我们通过调用`ldstr`函数来加载字符串，然后调用`System.Console::WriteLine(string)`方法，然后我们就完成了。

再次强调，这并不是机器代码。那看起来要复杂得多，我不会向您展示。如果编译成 CPU 可以理解和执行的代码，这段代码会有几页长。

然而，我将向您展示其中的一部分，以便您先尝尝味道：

```cs
00007FF9558C06B0  push        rbp
00007FF9558C06B1  push        rdi
00007FF9558C06B2  push        rsi
00007FF9558C06B3  sub         rsp,20h
00007FF9558C06B7  mov         rbp,rsp
00007FF9558C06BA  mov         qword ptr [rbp+40h],rcx
00007FF9558C06BE  cmp         dword ptr [7FF95597CFA8h],0
00007FF9558C06C5  je
Program.<Main>$(System.String[])+01Ch (07FF9558C06CCh)
00007FF9558C06C7  call        00007FF9B54D7A10
00007FF9558C06CC  mov         rcx,1A871002068h
00007FF9558C06D6  mov         rcx,qword ptr [rcx]
00007FF9558C06D9  call        qword ptr
[CLRStub[MethodDescPrestub]@00007FF9559C17E0
(07FF9559C17E0h)]
00007FF9558C06DF  nop
00007FF9558C06E0  nop
00007FF9558C06E1  lea         rsp,[rbp+20h]
00007FF9558C06E5  pop         rsi
00007FF9558C06E6  pop         rdi
00007FF9558C06E7  pop         rbp
```

这段微小的汇编代码段指示 CPU 获取字符串所在内存的指针，然后调用`WriteLine`方法的第一部分。

再次强调，完整的代码会有几页长。

我希望您开始欣赏.NET 系统的简洁性和简单性。但我也想让您知道幕后发生了什么。在编写系统软件时，我们有时需要做一些在.NET 中不可能完成的事情。然后，我们必须依赖其他方式来实现我们的目标。我们不会在这本书中编写纯汇编代码：那会太复杂了。但我确实想让您知道正在发生的事情，这将在以后给您带来巨大的好处。

好的。在我向您展示的 IL 代码和我向您展示的汇编代码之间，是 CLR 存在的地方。

如[`learn.microsoft.com/en-us/dotnet/standard/clr`](https://learn.microsoft.com/en-us/dotnet/standard/clr)所述，CLR 为我们提供了相当多的事物：

+   性能改进

+   能够轻松使用其他语言开发的组件

+   由类库提供的可扩展类型

+   面向对象编程的语言特性，如继承、接口和重载

+   支持显式多线程，允许创建多线程和可扩展的应用程序

+   支持结构化异常处理

+   支持自定义属性

+   垃圾回收

这条信息直接来自微软的文档，所以如果您想了解更多，我强烈建议您查找并阅读更多关于它的内容。后面的章节将讨论一些这些项目，例如线程、异常处理和垃圾回收。现在，了解当我们编译我们的代码时，我们为 CLR 使用和运行它做准备，CLR 将负责其余部分，并在实际硬件上运行得很好就足够了。

在 CLR 上运行的代码就是我们所说的托管代码。所有其他代码（因此不在 CLR 控制下的代码）都是非托管的。您将大部分时间都在处理托管代码，但编写系统软件时，您会经常遇到非托管代码。但别担心：我会引导您通过这一过程！

## BCL

.NET 的设计者心中设定的一个目标就是消除开发者所说的 DLL 地狱。

这个想法是，当编写软件时，开发者很快意识到反复编写相同的代码会感到乏味，并且难以维护。相反，他们创建了包含可重用函数的库。这些库将在需要时加载，并与调用代码链接。这就是**动态链接库**（**DLL**）这个名字的由来。

当然，开发者作为开发者，并不满足于他们或其他人之前写的 DLL，并对它们进行了修改。这些修改并不总是向后兼容的。这意味着作为一个 DLL 的用户，你必须确保你使用了正确的版本。如果没有测试这个特定版本的 DLL 是否与你的代码兼容，你就不能轻易升级。

DLL（动态链接库）有两种类型。一种专属于你的代码。你将这些 DLL 放在与你的应用程序相同的目录中，因此你只需要在你的应用程序目录中加载这些 DLL。如果推出了新的应用程序版本，它将附带它自己的 DLL 集合。

由于大多数 DLL 没有太多变化（如果它们甚至有所变化），因此有许多重复和重复的 DLL。因此，我们不是在复制代码，而是在复制 DLL。

幸运的是，有一个解决方案：你可以将 DLL 放在共享空间中。在 Windows 上，那就是`C:\Windows\System32`目录。运行时知道，如果它需要加载一个 DLL，但在`applications`目录中找不到它，它可以在`System32`目录中查找它。

在做这件事的时候，你需要确保你保持了向后兼容性。

自然，事情出了问题。更新会替换 DLL 为更新的、不兼容的版本。有时，应用程序更新 DLL 时没有意识到其他东西依赖于它。有时，更新会删除 DLL，从而破坏应用程序。在许多情况下，开发者部署了错误的版本。问题层出不穷。这给许多开发者带来了许多挫折，并导致他们称之为 DLL 地狱。项目 42 被设立来解决这个问题。从某种意义上说，它确实做到了。

几十年前，`String`类是新 C++程序员会写的第一件事。C 和 C++没有这样的东西：字符串不是语言的原生部分（它们现在还不是，但现在包含它们的辅助类是标准的一部分）。字符串可以非常简单：它只是一个指向内存中某个位置的指针，所有后续的字节形成一个长字符串。字符串在系统看到值为 0 的字节时结束（零，不是字符 o）。就是这样。`String`类将包含该字节数组的地址，一些分配和清除内存的辅助方法，以及如`Length()`这样的附加函数。就是这样。

很快，每个人都编写了不同的版本，它们都会略有不同。.NET 通过提供一个`String`类来解决这一问题。这个类是框架附带的一个 DLL 的一部分。系统注册了这个 DLL 及其版本号。因此，所有开发者需要做的只是告诉系统它正在使用哪个版本的框架，然后通过魔法，诸如字符串之类的功能就可用。我在这里简化了事情，但基本上就是这样工作的。

作为.NET 开发者，你可以使用一个庞大的库。你可以在`C:\Windows\assembly`中看到它。如果你使用 Windows 资源管理器，你会看到一个过滤后的内容视图。你可以使用终端或命令行查看实际内容。

这些 DLL 是 BCL 的一部分。BCL 是一组辅助类、函数、方法和枚举，可以帮助你完成工作。你不需要自己弄清楚所有代码，它是安装的一部分，可以直接使用。

构成 BCL 的 DLL 中的类和其他代码结构被组织到命名空间中。BCL 包含大量有用的代码，其中一些如下：

+   `System`命名空间，其中包含`Object`、`String`、`Array`等类。

+   `System.IO`命名空间，用于处理文件、流等。

+   `System.Net`用于处理网络。

+   `System.Threading`用于处理——你猜对了——多线程。

+   `System.Data`用于处理数据库中的数据存储和其他持久化数据的方式。

+   `System.Xml`在这里，你可以用它来处理 XML 文件和数据。

+   `System.Diagnostics`帮助你识别代码中的问题。我们稍后会深入探讨这个问题。

+   `System.Security`命名空间，以及所有与安全和加密相关的内容。

还有许多其他命名空间，但这些都是最常用的几个。我们稍后会重新访问它们。

然而，请记住，这些类是为了帮助你。它们以良好的方式封装了复杂和广泛的代码，对大多数开发者来说都是如此。但是，如果你发现 BCL 代码无法让你达到想要的目标，没有任何阻止你亲自编写代码。正如我们之前看到的，如果你想要设置 TCP/IP 连接，BCL 代码是出色的，但如果你想要使用红外连接，你必须自己完成。

好消息是你可以混合使用。在可能的地方使用 BCL，在需要的地方使用低级 API。

# 使用 P/Invoke 调用低级 API

我们已经确定.NET 为你提供了许多快速开发工具。它还通过屏蔽底层操作系统的低级细节来帮助你。但它也允许你在需要时使用低级 API。

但我们如何访问这些 API 呢？答案是**平台调用**，或（**P/Invoke**）。我们可以使用这个工具直接访问 Win32 API。P/Invoke 在两个平台之间架起桥梁，这样我们就可以随心所欲地混合使用。

注意

Win32 是这个 SDK 和可用的 API 的名称。没有 Win64 API 这样的东西。如果你在 64 位 Windows 平台上运行代码，我们的代码是针对 64 位 Windows 编译的，但我们（以及微软）仍然称之为 Win32 API。

P/Invoke 是如何工作的？

P/Invoke 涉及几个步骤。这些是你必须遵循的步骤，以便在.NET 应用程序中使用 Win32 API：

1.  找到你想要使用的 API。

1.  找到 API 所在的 DLL。

1.  在你的程序集加载该 DLL。

1.  声明一个存根，告诉你的应用程序如何调用该 API。

1.  将.NET 数据类型转换为 Win32 API 可以理解的形式。

1.  使用 API。

警告！

你已经离开了.NET Framework 和 CLR 的关爱之手。你不再受到错误的保护。你现在处于一个非托管的世界。在旧时代，他们可能会在这个文档的部分标记上警告“此处有龙”。你现在需要负责比以往更多的事情，比如内存管理和错误处理。你现在对系统的控制力更强，但请记住：大权在握伴随着巨大的责任！

让我从一个例子开始。这展示了.NET Framework 的强大功能，但也展示了之前提到的步骤在实际中是如何工作的。我们将进行一个简单的“Hello World”。

为了确保我们处于同一页面上，让我给你展示这个程序的.NET 版本：

`Console.WriteLine(“Hello,` `System Programmers!”);`

是的，这是我们之前看到的同一个示例。嘿，我们必须从某个地方开始，对吧？

现在，`Console`是 BCL 中的一个类。它有一个静态方法`WriteLine`，可以将字符串输出到输出。但如果我们假设我们不想使用这个类呢？那么我们应该如何处理这个问题？换一种方式来提出这个问题，`WriteLine`是如何在内部工作的？毕竟，在执行过程中，代码必须调用 Win32 API。这可以通过 CLR 或我们来实现，但必须有人或某物来调用它。

让我们使用 P/Invoke 重写代码。我会先展示整个程序，然后逐行剖析并解释它是如何工作的：

```cs
01: using System.Runtime.InteropServices;
02:
03: [DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError =         true)]
04: static extern bool WriteConsole(
05:     IntPtr hConsoleOutput,
06:     string lpBuffer,
07:     uint nNumberOfCharsToWrite,
08:     out uint lpNumberOfCharsWritten,
09:     IntPtr lpReserved);
10:
11: [DllImport("kernel32.dll", SetLastError = true)]
12: static extern IntPtr GetStdHandle(int nStdHandle);
13:
14: const int STD_OUTPUT_HANDLE = -11;
15:
16: IntPtr stdHandle = GetStdHandle(STD_OUTPUT_HANDLE);
17: if (stdHandle == IntPtr.Zero)
18: {
19:     Console.WriteLine("Could not retrieve standard output           handle.");
20:     return;
21: }
22:
23: string output = "Hello, System Programmers!";
24: uint charsWritten;
25:
26: if (!WriteConsole(
27:     stdHandle,
28:     output,
29:     (uint)output.Length,
30:     out charsWritten,
31:     IntPtr.Zero))
32: {
33:     Console.WriteLine("Failed to write using Win32 API.");
34: }
```

这段代码很多，但让我们逐行分析。

在*第 1 行*，我们导入了一个命名空间，它允许我们使用 P/Invoke。.NET 使用`InteropServices`这个名字，所以导入它是合理的。

在*第 3 行*，我们看到魔法发生了。还记得我们必须采取的步骤吗？*步骤 1*是“找到你想要使用的 API”。由于我们想在屏幕上打印一些东西，`WriteConsole` API 听起来像是一个不错的选择。

微软的官方文档指出，`WriteConsole` API“*从当前光标位置开始将字符字符串写入控制台屏幕缓冲区。*”这听起来不错。

文档接着给出了这个 API 的签名：

```cs
BOOL WINAPI WriteConsole(
  _In_             HANDLE  hConsoleOutput,
  _In_       const VOID    *lpBuffer,
  _In_             DWORD   nNumberOfCharsToWrite,
  _Out_opt_        LPDWORD lpNumberOfCharsWritten,
  _Reserved_       LPVOID  lpReserved
);
```

如果你是一个 .NET 开发者，这可能会看起来很奇怪。有很多我们不知道或理解的事情在发生。我们需要将这些类型转换为 CLR 能够理解的东西。幸运的是，有人已经为我们解决了这个问题。为了使生活更加简单，他们还为我们做了 *步骤 2*（找到 DLL）和 *步骤 4*（声明存根）。给定正确的参数，CLR 会处理 *步骤 3*（加载 DLL）。

“弄清楚这一点的人”是 [`pinvoke.net`](https://pinvoke.net) 背后的人。你可以在那里搜索 API 并学习如何使用它们。

官方文档有一个名为 `Kernel32.dll` 的部分（`Pinvoke.Net` 也提供了这个信息）。

*第 3 行* 是告诉 `InteropServices` 加载 DLL 的部分。让我们深入探讨一下：

```cs
[DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)]
```

这一行告诉 CLR 加载 `kernel32.dll`。然后指定如何处理字符。字符和字符串可能很复杂。表示单个字符有几种不同的方式。它可以是一个 ASCII 字符，也可以是 Unicode，或者可以是 ANSI。它们在内存中都有不同的表示形式。在这里，我们说我们想要使用 `Auto`。当这样做时，系统会查看我们使用的完整字符串，找出它可以用来表示完整字符串的集合，并使用它找到的第一个。由于它首先尝试将其适应到 ASCII 字符串中，然后“向上移动”到更复杂、更慢、更占用内存的方式，这保证了我们得到表示这个字符串的最佳方式。

接下来，我们可以看到 `SetLastError = true`。这指示系统在发生错误时通知我们。在出现错误的情况下，它调用 `GetLastError` API 来获取错误并将其返回给我们。我们将在以后大量使用这个。现在，我建议你始终将 `SetLastError` 设置为 `true`。

我们的运行时现在知道要加载 `kernel32.dll`。但我们必须告诉它我们想要使用哪个特定的 API。这发生在下一行。函数的签名必须始终紧跟在 `DllImport` 之后：它们总是属于一起。如果你想要从同一个 DLL 加载多个函数，你仍然必须为每个函数使用 `DllImport`。

下一行是函数的存根：

```cs
static extern bool WriteConsole(
    IntPtr hConsoleOutput,
    string lpBuffer,
    uint nNumberOfCharsToWrite,
    out uint lpNumberOfCharsWritten,
    IntPtr lpReserved);
```

这看起来像我们从官方文档中看到的代码，但现在，类型已经被转换成了它们的 .NET 等价物。再次强调，`Pinvoke.Net` 是你的好朋友！

参数基本上是自我解释的，除了第一个。让我们跳过那个，看看其他的：

| **String lpBuffer** | **我们想要打印的字符串** |
| --- | --- |
| nNumberOfCharsToWrite | 我们想要从给定字符串中打印的字符数 |
| lpNumberOfCharsWritten | 写入系统的字符数 |
| lpReserved | 这个参数没有被使用，所以可以忽略 |

表 1.1：WriteConsole 的参数

这里需要注意的一点是，Win32 API 使用匈牙利命名法为其参数命名。这种风格规定你必须使用类型的缩写前缀来修饰每个参数，这样当你以后阅读代码时，就能知道这个参数代表什么类型。在当前现代和快速的 IDE 出现之前，这非常有帮助：你不能将鼠标悬停在变量上以查看其类型；你必须滚动代码以找到其声明。通过前缀，你可以立即看到它。如今，我们不再需要这样做，但 C 和 C++ 开发者仍然使用这个标准。

因此，正如你所看到的，要打印的字符串是一个长指针（lp），要写入的字符数是一个数字（n）。

但让我们看看 `hConsoleOutput`。它是一个句柄（以 h 开头），在 .NET 中将其转换为 `IntPtr`。

指针只是内存中某个东西的地址。在我们的例子中，这个内存属于控制台所在的位置。但我们如何获取它？控制 `Console` 的代码在哪里？

答案是，我们不知道。没有固定的位置；这可以，并且每次你重启程序时都会改变。因此，我们需要去寻找它。

幸运的是，这并不难做，因为有一个我们可以使用的 API 来完成这个任务。这个 API 被称为 `GetStdHandle`，它位于 `kernel32.dll` 中。我们知道如何导入它，我们可以在我们的代码的第 *11* 和 *12* 行中看到它：

```cs
[DllImport("kernel32.dll", SetLastError = true)]
static extern IntPtr GetStdHandle(int nStdHandle);
```

没有字符串，所以我们不需要设置 `CharSet`。然而，我们需要设置 `SetLastError`。

查找地址的方法被称为 `GetStdHandle`，它接受一个参数：`nStdHandle`。这告诉此 API 我们正在寻找哪种类型的控制台。有三种类型可供选择：`STD_INPUT_HANDLE, STD_OUTPUT_HANDLE` 和 `STD_ERROR_HANDLE`。这三个常量的值分别为 -10、-11 和 -12。如果你认为它们是负值很奇怪，那么你是正确的。这很奇怪。然而，在 Win32 中，这些值是无符号的。它们是整数范围的终点，因此不会妨碍你定义的其他任何类型的控制台。将无符号整型的高值转换为有符号整型会导致负值。

在 *第 14 行*，我们定义了 `STD_OUTPUT_HANDLE` 常量，并给它赋值为 `-11`。这类事情很常见：Win32 API 中充满了魔法数字和常量。

在 *第 16 行*，我们使用 `GetStdHandle` 来获取内存中 `Console` 的指针，并给它传递 `STD_OUTPUT_HANDLE`。如果出错，我们会得到一个 0（零）。但由于 .NET 是强类型的，我们不能使用这个数字。相反，我们必须使用 `IntPtr.Zero` 常量，它在正确的类型中做同样的事情。

每次从 Win32 API 获取一个 0，都表示存在错误情况。我们需要处理这种情况，但这将是后续讨论的主题。

假设一切顺利，我们可以定义我们的字符串，`out` 变量会告诉我们写入了多少个字符（*第 23* 和 *24* 行）。

然后，在 *第 26 行*，我们调用实际的 API：

```cs
if (!WriteConsole(
    stdHandle,
    output,
    (uint)output.Length,
    out charsWritten,
    IntPtr.Zero))
{
    Console.WriteLine("Failed to write using Win32 API.");
}
```

现在应该很清楚了。我们调用 API，给它正确的参数，并检查结果是否为`0`（`IntPtr.Zero`）。

CLR 将复杂的.NET `String`类型转换为以 0 结尾的简单字节数组。我们不必担心这一点。我们可以给这个 API 提供一个 C#字符串，一切都会顺利。

就这样。我们已经使用 Win32 API 向控制台写入了一些内容！

# 处理错误

在前面的例子中，我们做了一些错误检查。如果我们无法获取句柄，我们会显示一个消息。如果我们无法写入控制台，我们也会这样做。我明白写入无法写入的系统控制台听起来很滑稽（例如，看看*第 33 行*），但你应该明白我的意思。

但如果你想知道真正的情况，这还不够。我们需要一种更彻底的方式来处理错误。

在.NET 中，我们习惯于在出现问题时获取异常。我们知道如何处理它。在低级世界中，事情有所不同。当出现问题时，我们会得到`0`，然后我们必须处理它。我们可以继续编写代码而不会被错误打扰。我们甚至可以忽略 API 调用的结果。然而，这将导致灾难。你应该始终检查并处理 API 调用的结果。如何处理这个问题是我们将在本节中讨论的内容。

有一个名为`GetLastError`的低级 API 可以帮助我们。P/Invoke 的签名如下：

```cs
[DllImport("kernel32.dll")]
static extern uint GetLastError();
```

这看起来相当简单。没有需要担心的参数，我们也不必在这里设置那个`SetLastError`值。由于`SetLastError`确保任何错误都保存在注册表中，以便`GetLastError`可以读取它，所以在这里设置它没有意义。如果设置了，并且将其设置为 false，那么`GetLastError`将如何工作呢？

这个函数返回一个无符号整数。这个数字对应一个错误；你可以在文档中查找这个数字的含义。

但这里有个问题：它不起作用。嗯，它确实起作用，但结果没有保证。

BCL 和`CLR`始终与低级 Win32 API 一起工作。这是显而易见的：BCL 是 Win32 API 的包装器，CLR 使用这个包装器调用操作系统的核心系统。我们可以自己调用 API，就像我们刚才做的那样，但 CLR 也会调用它。有时，它在同一个线程上调用。有时，它在另一个线程上调用。在 CLR 调用 API 的过程中也可能出错。这可能导致`GetLastError`返回没有错误或错误的错误。嗯，技术上，它们不是错误的错误，但它们可能与我们正在做的事情无关。

幸运的是，.NET 的设计者考虑到了这一点，并在 `System.Runtime.InteropServices` 命名空间中添加了一个名为 `Marshal` 的类。该类用于在托管和非托管代码之间进行封送处理——或者，用我们在这里所做的事情来说，就是在 Win32 API 和我们的 C# .NET 代码之间。

让我们假设我们犯了一个错误。我知道这很难想象，但请在这里忍受一下。我们不是将 `-11` 分配给 `STD_OUTPUT_HANDLE`，而是将其设置为 `11`。我们都会犯错误，对吧？

我们随后调用 `GetStdHandle` 并传入 `11`。这是不正确的；我们都知道这一点。文档说明，如果发生任何错误，该函数返回 `0`（或在 C# 中为 `IntPtr.Zero`）。但在我们的情况下，它返回了其他东西：`0xffffffffffffffff`。这是有符号值 `-1` 的无符号版本。换句话说，对 API 的调用返回 `-1`，这不是一个有效的句柄。

然而，我们没有检查这一点。我们只检查 `0` 值。如果你这样想，这是有道理的。毕竟，`0` 表示在调用该函数时出了问题。这种情况没有发生：该函数工作得完美无缺。它只是没有找到与我们所提供的 ID 匹配的内容（`11` 而不是 `-11`）。所以，从 API 的角度来看，没有错误。

但然后我们来到了调用 `WriteConsole` 的地方。我们给它传递控制台的句柄——或者更确切地说，我们认为我们这样做。相反，我们传递了一个值为 `-1`（`0xffffffffffffffff`）。这不是 `WriteConsole` 可以处理的有效的句柄。

在 .NET 中，你会得到一个异常，但这里没有发生这种情况。代码继续愉快地运行，没有任何抱怨。它只是没有输出任何内容。

这些错误可能很难找到和解决。在这种情况下，它相当直接，但想象一下，当你尝试设置与红外接收器的连接并且出了问题的情况。然而，我们继续进行，因为我们没有检查那个结果。当我们准备发送数据时，什么也没有发生——或者更糟，系统崩溃了。我们开始查看实际发送数据的代码，但那里没有问题。需要花费很多时间和仔细的调试才能看到错误发生在我们设置连接时。让我重复一下我之前说过的话：你应该始终检查所有 API 调用的结果。这个责任在你身上。.NET 运行时在这些情况下生成异常，但如果你处于非托管环境中，你必须负责这样做。

让我们改进一下我们的代码。

首先，我们将我们的 `WriteConsole` 调用包裹在一个 `try-catch` 块中，并捕获 `Exception`，尽管这通常是一个糟糕的想法。然而，在这里，这已经足够好了。

如果 `WriteConsole` 返回 `IntPtr.Zero`，我们就遇到了问题，并且出了错。在非托管环境中，你会调用 `GetLastError` 来查看发生了什么，但在这里不起作用。相反，我们使用我之前提到的那个 `Marshal` 类：

```cs
if(!WriteConsole(stdHandle, output, (uint)output.Length, out charsWritten, IntPtr.Zero))
{
    var lastError = Marshal.GetLastWin32Error();
    Console.WriteLine($ something went wrong. Error code:
        {lastError}");
}
```

当将`STD_OUTPUT_HANDLE`设置为`11`时运行此代码，系统会报告出错了。它甚至告诉我们错误代码是`6`。

在官方文档中查找这个信息，结果如下：

ERROR_INVALID_HANDLE

6 (0x6)

这个句柄是无效的。

这正是正在发生的事情。

“等一下，”我几乎能听到你这么说。“我不能要求我的用户每次出问题时都去查阅官方文档来查看错误消息的含义！”

嗯，你说的没错。.NET 设计团队也同意这一点。他们增加了一些获取错误消息的方法。有两种方法可以获取它，你可以选择你想要的任意一种。

首先，如果你想获取那个错误消息，你可以用以下代码来获取：

```cs
if(!WriteConsole(stdHandle, output, (uint)output.Length, out charsWritten, IntPtr.Zero))
{
    var lastError = Marshal.GetLastWin32Error();
    var errorMessage = Marshal.GetPInvokeErrorMessage(lastError);
    Console.WriteLine($"Something went wrong. Error message:         {errorMessage}");
}
```

再次，我们首先获取错误代码。我们总是必须这样做，而且我们应该尽可能快地这样做，以免其他地方的另一个错误破坏了事情。

然后，我们调用`Marshal.GetPInvokeErrorMessage`方法，并给它那个`lastError`代码。它返回一个字符串告诉我们`The handle` `is invalid.`。

很好。但如果这个错误的影响如此之大以至于我们无法继续呢？.NET 告诉我们在这种情况下使用异常。良好的实践教导我们永远不要抛出异常，而应该使用一个专门的派生异常。我们正好有这样一个东西：`Win32Exception`。

我们可以将这个错误消息抛出，并将消息设置为从`GetPInvokeErrorMessage`获取的消息，但鉴于这是一个非常常见的场景，.NET Framework 为我们提供了一个快捷方式来做这件事。看看下面的代码：

```cs
try
{
    if(!WriteConsole(stdHandle, output, (uint)output.Length, out         charsWritten, IntPtr.Zero))
    {
        var lastError = Marshal.GetLastWin32Error();
        throw new Win32Exception(lastError);
    }
}
catch(Win32Exception e)
{
    Console.WriteLine($"Error: {e.Message}");
};
```

这看起来要好得多。这段代码会在我们的屏幕上显示一条消息，内容为`Error: The handle is invalid`。好吧，既然这只是一个简单的例子，我未能正确处理这个问题（在这里重新抛出是个好主意）。在出现此类错误后如何继续取决于你的编码风格、用例以及你想要达到的目标。

获取错误消息还有另一种方法。这个方法相当不错，但不如我们之前讨论的其他方法直接：`FormatMessage`。

`FormatMessage`函数来自 Win32 API。其声明如下：

```cs
[DllImport("kernel32.dll")]
static extern uint FormatMessage(
    uint dwFlags,
    IntPtr lpSource,
    uint dwMessageId,
    uint dwLanguageId,
    [Out] StringBuilder lpBuffer,
    uint nSize,
    IntPtr Arguments);
```

如果我们有错误代码，我们可以这样使用它：

```cs
var lastError = Marshal.GetLastWin32Error();
int bufferSize = 256;
var errorBuffer = new StringBuilder(bufferSize);
var res = FormatMessage(
    0x00001000,
    IntPtr.Zero,
    (uint)lastError,
    0,
    errorBuffer,
    (uint)bufferSize,
    IntPtr.Zero);
if(res != IntPtr.Zero)
{
    var formattedError = errorBuffer.ToString();
    Console.WriteLine(formattedError);
}
```

首先，我们创建`StringBuilder`。API 使用它来构建带有错误消息的字符串。我们给它分配了`256`个字符的大小。这个大小对于大多数，如果不是所有错误来说应该足够了。我们需要给出这个大小，因为在 C 和 C++中，你需要事先分配一个缓冲区；它不能动态扩展（嗯，它可以，但如果你想要高性能，你不会这样做）。我们使用 0x00001000 标志调用`FormatMessage`。这个标志意味着“使用提供的错误代码。”我们可以使用其他标志，但这个标志是最常用的。我们没有想要格式化的消息，所以第二个参数是`IntPtr.Zero`。然后，我们给它`lastError`，语言为 0（即系统默认，通常是英语），缓冲区，缓冲区的大小，以及另一个`IntPtr.Zero`参数。最后一个参数意味着我们不使用参数。在这里，参数与我们在 C#中想要格式化字符串时的参数相同：

```cs
Console.WriteLine("Hello {0}", 42);
```

在这里，`42`是参数。

当我们运行这段代码时，我们会得到相同的“句柄无效”消息。

你可能想使用这个 API，因为它可以做一些很酷的技巧。例如，将`languageId`代码 0 替换为代码 0x0413。这个`languageId`是荷兰的 Windows 语言 ID（请使用你想要的任何语言。）

结果是`De ingang is ongeldig`，这基本上是对原始错误的良好翻译。

这样，你可以有格式良好、翻译过的错误消息！

这里还有最后一件事要说明：许多在线示例使用以下代码：

```cs
if(!WriteConsole(stdHandle, output, (uint)output.Length, out charsWritten, IntPtr.Zero))
{
    var lastError = Marshal.GetLastWin32Error();
    var errorMessage = new Win32Exception(lastError).Message;
    Console.WriteLine($"Error: {errorMessage}");
}
```

从技术上讲，这并没有什么问题。但这并不是异常存在的原因。仅仅为了得到消息就创建一个异常是错误的。然而，我见过这种情况很多次，所以我认为我应该警告你。如果你不想抛出异常，就不要创建它。在这种情况下，调用`Marshal.GetPInvokeErrorMessage`代替。这将对你和那些维护你的代码的人大有裨益。

# 使用低级 API 调试代码时的问题

使用如 Win32 API 这样的低级 API 可以打开一个充满新且强大工具的宝库。然而，这也带来了一些缺点。调试你的代码突然变得困难得多，而且它也变得更加关键。

当你想使用低级 API 调试代码时，有几个区域你需要注意：

+   错误处理

+   互操作性

+   调试工具

+   兼容性和可移植性

+   文档和社区支持

这些都可能构成挑战，要求你在开始编码之前考虑你的调试策略。让我们看看潜在的问题。

## 错误处理

如前所述，使用低级 API 时，错误处理由你负责。当调用函数出错时，你不会从函数中得到异常。你必须始终小心地检查调用代码的返回码，看它是否为 0。即使如此，也不能保证事情会按照你预期的方向发展。例如，当我们给`GetStdHandle`函数一个无效的`ConsoleId`类型时，调用是正常的，但结果仍然不是我们预期的。你必须对这些类型的调用非常小心。理想情况下，我们会立即捕捉到这个问题，并通知系统出现了错误。

即使你捕捉到了所有的错误代码，这也并不意味着你可以确定发生了什么错误。有时，错误信息如此晦涩，你必须阅读文档才能了解发生了什么。

API 中有一个名为`CoCreateInstance`的方法。它用于创建 COM 对象，这些对象用于连接到其他系统，例如 Word 或 Excel。为了建立这种连接，给它你想要连接的对象的 ID。这些 ID 的形式是 GUID，你必须手动输入。如果曾经有过容易出错的情况，那可能就是这种情况。

使用不存在的`ClassID`会返回错误代码`0x80004005`。如果我们使用之前描述的方法来获取错误信息，你可能会读到类似`Invalid ClassId`或`COM 对象未找到`的内容。不幸的是，你得到的是`E_FAIL:` `未指定错误`。

唉。

这完全无助于解决问题，对吧？失败了。好吧，我们知道了。但是为什么？是什么失败了？我们不知道。系统在这里根本不帮助你。你必须知道你在做什么，系统期望什么，并且逐行检查代码以找出错误。这并不容易。

## 互操作性

正如我们讨论的，在调用 Win32 API 时，你必须采取的步骤之一是将 C#中使用的类型转换为它们的 Win32 等效类型，反之亦然。有时，这很简单；有时，这可能相当具有挑战性。

框架设计者做了很多工作来帮助我们：当 Win32 API 期望一个字符串时，你通常可以给它一个 C#字符串，CLR 会自动在两者之间进行类型转换，即使你不知道。但是，仍然有一些转换在进行。C 风格字符串是内存中字符位置的指针。下一个字符紧挨着它，以此类推，直到系统找到一个值为 0 的值。这是字符串的结束标记。这与我们在 C#中拥有的`String`类完全不同（内部，`String`类仍然有以 0 结尾的字符列表，但我们从未看到过，所以我们可以假装它根本不存在）。

C#中的大多数类型在 Win32 中都有一个兄弟类型。以下是最常用类型的列表：

| **C# 类型** | **Win32 类型** | **描述** |
| --- | --- | --- |
| `byte` | `BYTE` | 8-bit 无符号整数。 |
| `sbyte` | `CHAR` | 8-bit 有符号整数，通常用于 ASCII 字符。 |
| `short` | `SHORT` | 16 位有符号整数。 |
| `ushort` | `WORD` | 16 位无符号整数。 |
| `int` | `INT` 或 `LONG` | 32 位有符号整数 |
| `uint` | `UINT` 或 `DWORD` | 32 位无符号整数。也用于标志和枚举。 |
| `long` | `LONGLONG` | 64 位有符号整数。 |
| `ulong` | `ULONGLONG` | 64 位无符号整数。 |
| `float` | `FLOAT` | 32 位浮点数。 |
| `double` | `DOUBLE` | 64 位浮点数。 |
| `char` | `WCHAR` 或 `TCHAR` | 在 C# 中为 16 位 Unicode 字符，而在 Win32 中 `WCHAR`/`TCHAR` 则不同。 |
| `bool` | `BOOL` | 布尔类型。在 C# 中为 `True` 或 `False`，在 Win32 中通常为 `TRUE` 或 `FALSE`。在这里，`FALSE` 被定义为 `0`，而 `TRUE` 被定义为 `NOT FALSE`，即任何其他值，但通常为 `1`。 |
| `IntPtr` | `HANDLE`、`HINSTANCE`、`HWND` 等 | 表示指针或句柄。类型根据上下文而变化。 |
| `UIntPtr` | 在 Win32 中很少使用 | 无符号指针或句柄。 |
| `T[]` | `T*` 或 `SAFEARRAY` | `T` 类型的数组。其在 Win32 中的表示取决于上下文。 |
| `DateTime` | `FILETIME` 或 `SYSTEMTIME` | 表示日期和时间。在 Win32 中的表示不同。 |
| `Guid` | `GUID` 或 `UUID` | GUID。128 位数字。（GUID 通常与 Windows 平台相关联，而 UUID 在其他平台上找到。尽管如此，它们基本上是相同的。） |
| `TimeSpan` | 通常由 `DWORDs` 的组合表示 | 时间间隔。在 Win32 上不可用。 |

表 1.2：C# 和 Win32 类型

如你所见，大多数类型可以轻松地在平台之间转换。当我们深入研究更复杂的类型时，事情会变得有点复杂，因为许多类型都依赖于上下文或实现。这使得在平台之间传输类型具有挑战性。

另一个需要考虑的是所谓的调用约定。调用约定定义了在调用函数时如何处理参数。最常见的是 `stdcall` 和 `cdecl`。Win32 API 通常使用 `stdcall`，而大多数其他 C 库期望 `cdecl`。

我不会深入探讨这两种调用约定。然而，让我们总结一下最重要的区别：

+   `stdcall`：被调用者清理堆栈。它具有固定数量的参数，并且通常用于 Windows API。在这里，函数名称通常会被装饰。

+   `cdecl`：调用者清理堆栈并允许可变长度的参数列表。它通常用于 C 标准库。在这里，函数名称不会被装饰。

如你所见，了解如何调用函数是至关重要的。错误的约定可能会搞乱堆栈，并将参数传递给函数或返回错误的结果。你甚至可能会搞乱内存，这在编写托管代码时几乎是不可能的。

当你没有指定调用约定时，默认为 `stdcall`。如果你需要调用另一个库，你应该提供正确的调用约定。

可能一个例子会在这里有所帮助。我们之前使用 `WriteConsole` 将内容写入控制台，但有一个更简单的方法：`printf` 函数。这个函数是 Microsoft `msvcrt.dll` 库中的 C 运行时的一部分。如果你想使用这个函数，可以使用现在众所周知的 `DllImport` 声明来导入它：

```cs
[DllImport("msvcrt.dll", CallingConvention = CallingConvention.Cdecl)]
static extern int printf(string format, int i, double d);
printf("Hello, System Programmers!\n", 1, 2.0);
```

由于这个函数不是 Win32 API 的一部分，而是位于一个单独的 DLL 中，你必须小心并指定正确的调用约定。在这里，我们需要使用 `cdecl`，我们可以通过设置 `CallingConvention =` `CallingConvention.Cdecl` 来指定它。

其他类型包括 `WinAPI`、`StdCall`（它们基本上是相同的）、`ThisCall` 和 `FastCall`。你几乎不会遇到后两种，但至少你现在已经听说过它们了。

当你调用 API 并遇到奇怪的错误或意外行为时，你可能想要了解一下如何进行类型打包或调用约定。系统在这里并没有通过提供良好的错误信息来帮助你。

## 调试工具

Visual Studio 调试器很棒。然而，当混合托管代码和非托管代码时，事情可能会变得复杂。如果出现问题，系统可能会停止并显示一个断点。但由于被调用的代码不是 C#，调试器可能不会显示你需要看到的内容。它会尽力而为，所以它可能会反汇编代码并显示出错的汇编代码。

我在本章开头展示了些汇编代码。如果你想要在代码中查找错误，这并不是你想要看到的东西。好吧，我不知道这对你是否适用，但我知道我肯定不想看到那种。

如果发生这种情况，你可能想要使用其他调试器，例如 WinDbg。在本书的后续章节中，当我们介绍调试时，我们会更详细地探讨这个工具。但请相信我，调试混合代码可不是件轻松的事情。

## 兼容性和可移植性

Windows 会发生变化。有时，变化很大；有时，变化很微妙。尽管微软以尽可能保持向后兼容而闻名，但有时 API 会发生变化。签名可能会改变，或者行为可能会改变。而且你只有在事情变得非常糟糕时才会发现这一点。再次强调，你很少看到异常或错误信息，所以你只能自己调试代码并逐步执行。

一旦你开始使用 Win32 API，你就是在将自己绑定到一组有限的设备和平台。

更不要考虑将前面的代码部署到 Linux 平台。当然，.NET 在 Linux 上运行得很好，但当你开始使用 P/Invoke 时就不一样了。而且可能你的代码在一个 Windows 版本上运行得很好，但在下一个来自雷德蒙德的版本上却完全崩溃。我们可以称之为“工作保障”，因为这将要求我们不时更新我们的代码，但我不认为这很有趣。

## 文档和社区支持

Win32 API 文档的主要受众是 C 和 C++开发者。作为一个 C#开发者，很难找到所需的信息。像[`pinvoke.net`](https://pinvoke.net)这样的网站有所帮助，但前提是你知道它们是如何工作的。

作为.NET 开发者，你可能想要使用的第三方 DLL 的文档甚至更难找到。有时，你必须检查 DLL，了解其内部工作方式，然后将其转换为正确的 DLL 导入语句。如果你这样做，确保你有正确的调用约定和数据类型！

在混合托管和非托管代码时，社区支持也是一个挑战。大多数开发者会落入两个阵营之一：他们在非托管世界中工作，或者他们在托管世界中工作。两者兼顾是非常罕见的。

能够同时做到这两点的优秀开发者很少。好消息是，通过阅读这本书，你正走在成为那个非常精英群体中的一员的正确道路上！

# 下一步

本章探讨了低级 API 和高级 API 之间的区别。我们通过检查 BCL 和 CLR 来深入研究.NET 的基础。然后，我们探讨了如何调用低级 API，例如 Win32 API。我们通过重新实现无处不在的`Console.WriteLine`到 Windows 操作系统可以运行的代码中，而没有使用 BCL 来实现这一点。这导致我们讨论了错误发现和错误处理，以及如何最好地处理它们。

我们还讨论了你开始进行那种编码时可能会遇到的问题。我们提到了类型系统的差异以及你在处理调试器时可能遇到的问题。

希望这一章让你更加欣赏.NET 框架，以及 BCL 和 CLR 作为开发者为你所做的大量工作。但我也希望你意识到，当你使用 Win32 API 或其他用 C 或 C++编写的第三方库时，你获得的强大能力。

系统编程在很大程度上依赖于这些技术。尽管使用这些 API 将你绑定到正在为其开发的操作系统或该系统的特定版本，但这通常是实现你结果唯一的方法。坦白说，我认为与这些 API 一起工作很有趣。这完全是为了回归基础。

与低级 API 一起工作可能会很具挑战性。它们可能导致许多难以解决的错误。但是，当正确使用时，它们可以提高你的代码性能。在编写系统软件时，这一点非常重要。正如之前讨论的，系统软件不应该妨碍用户或用户直接与之交互的系统。相反，它应该尽可能快。因此，使用正确的 API 可能会给你带来所需的额外性能。我认为这一点非常重要，所以我专门写了一章关于性能的内容，恰好是下一章。我们生来就是为了奔跑，所以让我们尽可能快地奔向下一部分！
