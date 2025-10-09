# 6

# 进程低语篇

*进程间* *通信 (IPC)*

在上一章中，我们讨论了输入/输出。我们的大部分注意力都集中在文件上。文件是人们想到与其他系统共享数据时首先想到的事情之一。另一种常用的方法是网络。然而，系统之间还有其他通信方式。如果你想要长时间保留数据，文件是很好的选择。网络连接是连接不同机器上系统之间更直接连接的绝佳方式。但文件和网络更多地关于传输数据的基础技术。我们还必须决定如何使用这些方法连接到系统。简而言之，这就是**进程间通信**（IPC）的全部内容。我们如何让两个系统相互交谈？

在本章中，我们将涵盖以下主题：

+   什么是 IPC？

+   设计 IPC 时，我们需要关注哪些考虑因素？

+   Windows 消息 - 一种 Windows 原生的消息方式

+   管道 - 命名和无名

+   套接字 - 一种基于网络的 messaging 系统

+   共享内存 - 一种快速简单的本地 messaging 系统

+   **远程过程调用**（RPC）——控制其他机器

+   **Google 远程过程调用**（gRPC）——新加入的成员

欢迎来到迷人的低语系统世界！

# 技术要求

你可以在以下链接中找到本章中所有的代码：[`github.com/PacktPublishing/Systems-Programming-with-C-Sharp-and-.NET/tree/main/SystemsProgrammingWithCSharpAndNet/Chapter06`](https://github.com/PacktPublishing/Systems-Programming-with-C-Sharp-and-.NET/tree/main/SystemsProgrammingWithCSharpAndNet/Chapter06).

# IPC 概述及其在现代计算中的重要性

大多数软件都有用户界面。毕竟，用户应该通过这种方式与应用程序交互。用户点击按钮，输入文本，并在屏幕上读取响应。屏幕是数据、用户和应用程序交换数据和指令的方式。

人们不使用系统软件。其他软件会使用。因此，它需要不同的交互方式。我想技术上可能可以编写一个常规的用户界面并使用技巧来读取或输入数据，但这并不真正高效。

应用程序在相互交谈时会有不同的通信方式。它们有自己的语言和自己的协议。这就是进程间通信（IPC）的全部内容——进程之间的通信。

由于系统的性质，在设计系统之间的接口时，我们必须考虑几个关键点。在设计这个接口时，我们做出的选择与设计面向人的用户界面不同。这里有许多因素需要考虑。让我们逐一来看。

+   **明智地选择你的语言**：系统可以使用许多不同的方式相互通信，就像人类的对话一样，如果所有参与方都说同一种语言，这会极大地帮助。本章描述了我们可以如何让系统相互通信，但还有更多方法。某些方法可能比其他方法更适合特定的环境或用例，所以你必须仔细思考。不要选择你感到最舒适的那个，因为你已经知道那个解决方案。考虑所有的用例场景，然后选择合适的协议。

+   **安全性**：安全性是一个巨大的话题，尤其是在系统编程中。我们正在处理数据，系统隐藏在我们的计算机深处。大多数人不知道他们的机器上正在运行多个进程，因此他们不太可能检查它们并评估其安全性水平。

+   **数据格式和序列化**：当你的数据从一个系统转移到另一个系统时，你必须考虑最佳的数据转换方式。数据必须是包、信封或其他传输方式的一部分。有众多不同的格式和序列化方式，但你选择哪一种取决于许多因素。例如，如果你在同一个 Windows 机器上的两个 64 位进程之间使用直接内存连接，你可以使用一个非常高效、轻量级的二进制表示。然而，如果你必须与地球上另一侧运行不同操作系统的机器通信，那么你必须设计出两个系统都能理解的序列化机制。

+   **错误处理和健壮性**：软件可能会出错，我们都知道这一点。如果你在谈论多个独立的系统，那么错误和可用性的问题会呈指数级增长。因此，你必须对此保持警觉。你还必须考虑你的需求。你是否需要保证交付？你是否需要错误恢复？这两者可能很有用，但它们是有代价的。毕竟，没有什么是免费的。你需要考虑这些场景。通常，你必须设计一个可以应对的解决方案，而不是过度依赖错误纠正方案。

+   **性能和可扩展性**：在进程内部传输内存块是非常快的。在进程之间移动数据可能非常慢，甚至难以想象地慢。通过**传输控制协议**（**TCP**）连接将一个内存块移动到另一台机器比在内存中这样做慢数千倍。将数据写入磁盘，即使是快速的固态硬盘，也比这还要慢。

+   这意味着你必须确保这些用例的最佳 I/O 策略。建立连接或创建文件可能很慢，但你必须为每次传输只做一次。一旦你有了这个，你就可以写入数据。如果你有很多小数据包，你可能想将它们捆绑在一起，这样你只需要启动一次。正如我们之前所述，在传输之前压缩数据可能是个好主意。是的，压缩会占用 CPU 周期，但考虑到将数据传输到另一个系统要慢得多，这可能值得。

+   **同步和死锁**：一旦你的数据离开你的系统，你就不再知道它发生了什么。其他进程可能也在请求接收方的注意，或者接收方可能已经没有数据了。你必须非常小心，以确保数据同步。或者不。这当然取决于你的用例。此外，死锁也可能发生。你可能会等待接收方完成某个操作，但如果那个操作在等待你的系统，你就遇到了问题。要留意那些问题区域。

+   **文档和维护性**：与其他系统共享数据意味着与其他开发者共享你的数据结构。别忘了，“其他开发者”可能是六个月后的你自己，当你回顾你所做的一切并想知道你在想什么时。记录你的工作、你的想法和你的数据结构可以为你和你的同伴节省很多麻烦。做件好事，记录你的数据及其结构、你为了满足所有限制所做的工作，以及你的假设。这使得你的代码更容易维护。当然，这适用于数据共享场景和所有软件开发，但在你需要与其他系统共享数据时，这一点尤为重要。不要跳过这一步！

+   **平台和环境限制**：你未必总是清楚你的数据将会与哪种硬件共享。如果你不知道这一点，你必须考虑所有可用的选项。假设最坏的情况，并为此做好准备。例如，如果你传输几个吉字节的数据包，加密并使用压缩算法包装，你可能会收到投诉，说接收方是一个非常低端的 IOT 设备，内存和 CPU 处理能力有限。

并非所有平台都支持我在本章中概述的所有策略。例如，我们接下来要讨论的 Windows 消息，仅在 Windows 上可用。名字多少有点暗示，不是吗？要意识到平台和环境限制，并围绕这些限制设计你的数据共享。

因此，现在你已经知道了在选择通信方法时要考虑的因素，让我们看看我们有哪些可用方法。我们从一个经典的方法开始：Windows 消息。

# Windows 消息

**Windows 消息**是 Windows 中最老类型的 IPC。当编写系统软件时，它们可能不是最佳选择，但它们可能很有帮助。更重要的是，它们非常快且轻量级。然而，正如其名所示，它们是 Windows 特有的功能。

消息与窗口一起工作。我说的不是操作系统；我指的是您监视器上的屏幕。Windows 中的几乎所有 GUI 元素都是一个窗口。窗口显然是，但按钮、编辑框、文本框、滑块等等也都是。操作系统通过向窗口发送消息与您的应用程序通信。您的应用程序至少有一个主窗口，然后它会将消息分发到*子窗口*或处理那些子窗口的消息。然而，每个窗口都可以有自己的消息处理逻辑。

由于消息与图形屏幕元素（如按钮、标签和列表框）一起工作，你可能认为它们不能用于控制台应用程序或 Windows 服务。这在技术上是对的，但我们可以绕过这一点。我们可以创建一个隐藏的窗口来接收这些消息。

消息很简单。它只是一个包含四个数字参数的结构。这就是参数是什么以及它们用于什么。

| **名称** | **类型 /** **C# 类型** | **描述** |
| --- | --- | --- |
| `hWnd` | HWND / IntPtr | 要接收消息的窗口的唯一句柄 |
| `Msg` | UINT / uint | 消息的 ID |
| `wParam` | WPARAM / IntPtr | 一个附加参数，或数据结构的指针 |
| `lParam` | LPARAM / IntPtr | 一个附加参数，或数据结构的指针 |

表 6.1：Windows 消息中的参数

消息就这么多。`wParam` 和 `lParam` 指针指向包含有效载荷的内存。如果只想发送数字，它们也可以只是一个数字。在 16 位 Windows 中，`wParam` 是 16 位，`lParam` 是 32 位。在 Windows 的 32 位版本中，它们都是 32 位长，在 64 位版本中，它们都是 64 位长。因此，在长度方面，`wParam` 和 `lParam` 没有真正的区别了。

这些消息都是操作系统向您的应用程序发送的通信。如果用户将鼠标移到您的窗口上，您会收到通知。好吧，在鼠标移动的情况下，您会收到数百个通知。如果用户按下一个键，您会收到一个消息。如果用户调整窗口大小，您会收到另一个消息。操作系统上发生的任何可能对您的应用程序有趣的事情都会以消息的形式发送给您。您的应用程序会一直接收到数百甚至数千条消息。您的应用程序需要监听这些消息。我们很快就会看到它是如何工作的。

消息标识符可以是预定义的；它可以是您选择的数字，或者操作系统可以生成它。

让我解释一下我的意思。

如果 Windows 发送一个消息，它就是预定义的其中之一。例如，如果鼠标移动了，你会收到 `WM_MOUSEMOVE` 消息。`WM_MOUSEMOVE` 是一个值为 `0x0200` 的常量。`wParam` 包含有关鼠标按钮和键的状态的信息，例如你键盘上的 *Ctrl* 键。你可以解码这些标志来查看鼠标移动时是否按下了按钮。`lParam` 包含鼠标相对于接收消息的窗口（左上角）的 *X* 和 *Y* 位置。`lParam` 的前半部分包含 *Y* 坐标，后半部分包含 *X* 坐标。

一个有趣的消息是 `WM_CLOSE`。它的值是 `0x0010`。如果一个窗口收到这个消息，用户想要关闭它。如果这发生在你的主窗口上，应用程序就会结束。

你也可以定义自己的消息。有一个名为 `WM_USER` 的常量（值为 `0x0400`）。你可以在你的应用程序中自由使用 `WM_USER` 和 `0x7FFF` 之间的任何值来定义你的消息。有一个注意事项：你只能在你向应用程序的其他窗口发送这些消息时使用它们。你不能用它们与其他应用程序通信。原因很简单：你不知道系统外谁使用这些值。

如果你想要向其他应用程序发送消息，你需要向 Windows 注册。你可以调用一个 API 来保留一个在计算机运行期间唯一的保留号码。如果两个应用程序保留了相同的消息名称，它们将获得相同的 ID。这可以用来在进程之间进行通信，这正是我们现在要做的。

## 一个示例

要处理消息，我们需要使用大量的 Win32 API。逻辑并不复杂，但这个示例需要大量的设置。

我们可以将其分解如下：

1.  `Window` 类。它就像面向对象编程一样：首先定义一个类，然后创建实例。窗口就是这样。

1.  **定义消息循环方法**：这个方法在消息可用时立即被调用。

1.  **创建窗口**：一旦发生，消息就开始流动。

1.  `WM_CLOSE`，关闭应用程序。如果你想处理这个消息，请这样做。如果不处理，就将其传递给所有应用程序都有的默认处理器。

就这些了。

本书 GitHub 仓库中的源代码包含一个示例。我没有在这里包含它，因为该示例需要大量的样板代码，占据了数页。我决定将其从本章中省略，因为除了某些特定情况外，Windows 消息并未使用。然而，如果你感兴趣，只需查看示例代码。有了前面的解释，你可以很好地跟随。

现在你已经了解了 Windows 消息的工作原理，我们可以继续下一步，看看其他的方法。我们从一个简单的方法开始：管道。

# 在本地 IPC 中使用管道

**管道**最初来自 Unix，但也已经出现在其他平台上。管道就像两个系统之间的直接连接。它非常轻量级且易于设置。您可以使用它们在同一台机器上的进程之间以及通过网络在不同机器之间进行通信。从理论上讲，您可以使用管道在 Linux 和 Windows 之间进行通信。我说理论上是因为由于两个平台上的管道实现差异很大，您必须跳过许多步骤才能使其工作。实际上，您必须做的这项工作非常繁重，您可能更愿意使用其他方式，例如套接字，来实现相同的结果。这将更容易实现。

有两种类型的管道：**命名管道**和**匿名管道**。命名管道是最简单的一种。

## 命名管道

**命名管道**是如果您想在同一台机器上的一个进程与另一个进程之间进行通信时的一个很好的解决方案。通过网络进行通信并不复杂，但需要更多关于安全和访问权限的思考。

在.NET 中，您可以使用`NamedPipeServerStream`和`NamedPipeClientStream`类来实现这一点。

代码很简单。例如，让我们看看一个等待连接的服务器。我们还添加了一个连接到该服务器的客户端。一旦建立连接，服务器就会向客户端发送一条消息，该消息将在屏幕上显示。

这里是服务器代码：

```cs
using System.IO.Pipes;
"Starting the server".Dump(ConsoleColor.Cyan);
await using var server = new
    NamedPipeServerStream("SystemsProgrammersPipe");
"Waiting for connection".Dump(ConsoleColor.Cyan);
await server.WaitForConnectionAsync();
await using var writer = new StreamWriter(server);
writer.AutoFlush = true;
writer.WriteLine("Hello from the server!");
```

再次，我使用我的`Dump()`扩展方法在这里快速着色屏幕上的消息。

首先，我创建了一个`NamedPipeServerStream`的实例。作为一个参数，我给它提供了一个唯一的名称。如果我使用一个已经注册的名称，我将能够访问那个其他命名管道。这些名称在您的机器上是唯一的，但一旦`NamedPipeServerStream`被销毁，它们就会消失。

然后，我们等待连接。当客户端连接时，我们创建`StreamWriter`，将其命名为管道服务器流，并将数据写入流。

我们在写入器上使用`AutoFlush`：我们不希望数据悬挂在那里。

让我们看看客户端代码：

```cs
using System.IO.Pipes;
await using var client = new NamedPipeClientStream(".",     "SystemsProgrammersPipe");
"Connecting to the server".Dump(ConsoleColor.Yellow);
await client.ConnectAsync();
using var reader = new StreamReader(client);
string? message = await reader.ReadLineAsync();
message.Dump(ConsoleColor.Yellow);
```

这段代码看起来应该很熟悉。我们创建了一个`NamedPipeClientStream`（而不是服务器）的实例，并给它提供了两个参数。第一个参数是网络上计算机的名称（在我们的例子中，是我们自己的计算机，由点指定）。第二个参数是管道的名称。显然，这应该与我们用于服务器流的名称相同。

我们将客户端连接到管道，使用该客户端创建一个`StreamReader`实例，并读取数据。最后，我们显示来自服务器的数据。

## 匿名管道

**匿名管道**的工作方式与命名管道大致相同。它们提供了一种轻量级的方式将进程连接起来。然而，命名管道和匿名管道之间存在一些差异。以下表格突出了其中最重要的几个：

| **特性** | **命名管道** | **匿名管道** |
| --- | --- | --- |
| **识别** | 命名的。你可以通过名称找到它们。 | 未命名的。你必须知道运行时句柄才能连接。 |
| **通信** | 本地和网络通信。 | 只有本地通信。 |
| **对等方** | 每个服务器可以有多个客户端。可以设置为处理双向对话。 | 只有一对一。也只有单向。 |
| **复杂性** | 更复杂。允许进行异步通信，也能处理“发送后即忘”的场景。 | 更简单。直接的一对一父-子通信。 |
| **安全性** | 支持 ACL 以启用安全通信。 | 没有安全功能可用。 |
| **速度** | 由于控制更多而较慢。 | 快速。几乎没有开销。 |

表 6.2：命名管道和匿名管道功能比较。

设置匿名管道的代码实际上非常简单。让我们从服务器代码开始：

```cs
using System.IO.Pipes;
await using var pipeServer = new     AnonymousPipeServerStream(PipeDirection.Out, HandleInheritability.    Inheritable);
$"The pipe handle is: {pipeServer.GetClientHandleAsString()}".    Dump(ConsoleColor.Cyan);
pipeServer.DisposeLocalCopyOfClientHandle();
await using var sw = new StreamWriter(pipeServer);
sw.AutoFlush = true;
sw.WriteLine("From server");
pipeServer.WaitForPipeDrain();
```

让我带你了解这个过程。

首先，我创建了一个 `AnonymousPipeServerStream` 的实例。这个类处理所有通信的设置。我们可以看出它既可以发送也可以接收代码。我们不能使用提供的 `PipeDirection.InOut` 枚举：这将抛出异常。记住：匿名管道是单向的。

我们需要确保句柄可以被继承。这是因为客户端需要“继承”这个句柄。毕竟，这是我们唯一能够识别管道的方式。没有名称；它是匿名的！

我们调用 `GetClientHandleAsString` 以确定客户端应使用的内容。

当你创建 `AnonymousPipeServerStream` 时，它也会自动创建一个客户端。如果你想在进程内部进行通信，这会很有用。然而，如果另一个进程需要与这个服务器通信，你将遇到问题。匿名管道仅支持单连接。调用 `DisposeLocalCopyOfClientHandle` 会移除本地客户端，这样我们就有空间为另一个客户端服务。

然后，我们创建一个流，将其与管道关联，并向其写入。

最后，我们调用 `WaitForPipeDrain`，这是一个阻塞调用，只有当客户端读取完所有数据时才会继续。

客户端甚至更简单：

```cs
"Enter the pipeHandle".Dump(ConsoleColor.Yellow);
var pipeHandle = Console.ReadLine();
using var pipeClient = new AnonymousPipeClientStream(PipeDirection.In, pipeHandle);
using var sr = new StreamReader(pipeClient);
while (sr.ReadLine() is { } temp)
    temp?.Dump(ConsoleColor.Yellow);
```

我们首先从控制台读取句柄。这是服务器的输出，所以我们有可用的内容。然后，我们通过创建 `AnonymousPipeClientStream` 的实例来创建客户端，告诉它准备接收数据，并给它句柄。

然后，我们创建 `stream` 并从其读取。就这样！

有一个很大的注意事项。假设你编写了这两个控制台应用程序并运行它们。在这种情况下，你会看到，当你尝试创建`AnonymousPipeClientStream`的实例时，你会得到一个`InvalidHandle`异常。原因是 Windows 将进程分开，确保安全性尽可能高。如果你运行两个进程，它们无法相互访问句柄。因此，它无法访问管道，这意味着你无法通信。恐怕我们对此无能为力。如果你仔细想想，这确实是有道理的。你只能进行一对一的通信。所以，如果多个控制台应用程序连接到服务器，你怎么确保这种一对一的行为呢？答案是：你不能。

如果你想要独立的控制台应用程序，你应该使用命名管道。

然而，如果你想使用我提供的示例，你可以确保客户端和服务器在同一个地址空间中运行。你可以通过从服务器启动客户端来实现这一点。看起来是这样的：

```cs
Process pipeClient = new Process();
pipeClient.StartInfo.FileName = @"pipeClient.exe";
// Pass the client process a handle to the server.
pipeClient.StartInfo.Arguments =
    pipeServer.GetClientHandleAsString();
pipeClient.StartInfo.UseShellExecute = false;
pipeClient.Start();
```

不要忘记将客户端更改为从`Main`方法提供的`args`参数中获取句柄，而不是通过控制台从用户那里获取。

这里的秘密在于我们设置`UseShellExecute`为`False`的那一行。如果它是`True`，客户端将在另一个 shell 中启动，从而将其与服务器隔离开来。通过将其设置为`False`，我们防止了这种情况，并可以访问句柄，从而可以访问管道。

如果它们适合你的场景，匿名管道是通信工具箱中的绝佳补充。它们速度快且轻量级，这正是我们作为系统程序员所喜欢的。然而，还有其他可能更好的通信方式，尽管它们并不那么简单。让我们来谈谈套接字…

# 使用套接字建立基于网络的 IPC

套接字很棒。它们有点像通信领域的瑞士军刀。当你转向套接字时，管道和 Windows 消息的缺点就消失了。当然，没有免费的午餐，所以请准备好花大量时间思考错误处理和内存管理。尽管如此，一旦你掌握了这个概念，套接字并不难使用。

**套接字**是两个系统之间通过网络连接的端点。当然，系统可以位于同一台机器上，但它们也可以位于世界的两端。多亏了自 20 世纪 60 年代以来人们为构建网络所做的大量工作，我们现在可以接触到全球的各种机器。

## 网络基础

计算机网络已经存在很长时间了。然而，每个供应商都有自己的方式让机器相互通信。随着时间的推移，标准出现了。正如标准所做的那样，有好多可供选择。如今，我们在设置网络方面已经或多或少实现了标准化，所以你再也不必担心这个问题了。

但在我们深入具体细节之前，我们需要先谈谈**开放系统互联**（**OSI**）。

OSI 是一个分层架构，你可以用它来描述网络的工作方式。每一层都建立在上一层之上（第一层似乎是一个例外）。

共有七层，以下是它们的描述：

+   **第 1 层 – 物理层**：这是描述硬件的部分。例如，电缆的外观，开关的工作方式，施加的电压等。

+   **第 2 层 – 数据链路层**：这一层描述了系统如何在物理层上连接。在这里，我们描述了以太网或 Wi-Fi 的工作方式。MAC 地址（每个网络设备的唯一编号）在这里定义。

+   **第 3 层 – 网络层**：这一层完全是关于路由和寻址。在第 3 层定义了几个协议，例如**互联网控制消息协议**（**ICMP**），用于网络诊断和错误报告，**地址解析协议**（**ARP**），用于地址解析，蓝牙，当然还有**互联网协议**（**IP**），包括 v4 和 v6。

+   **第 4 层 – 传输层**：这一层负责端到端通信和可靠性。TCP 和**用户数据报协议**（**UDP**），本章的主题，位于这一层。

+   **第 5 层 – 会话层**：这一层管理应用程序之间的会话。

+   **第 6 层 – 表示层**：这一层确保数据以其他系统可以理解的形式呈现。

+   **第 7 层 – 应用层**：这里列出了使用网络的程序。

硬件和操作系统处理第 1 层至第 4 层。第 5 层至第 7 层由我们来负责。

几乎所有系统都使用 TCP 作为传输层，但有时人们会选择 UDP。我先解释 TCP 及其使用方法，然后在本文这部分结束时转向 UDP。IP 几乎是既定的。我们可以选择其他网络层协议，但这会使生活变得不必要地复杂。

设置会话（第 5 层）是我们编写代码在客户端或服务器上建立连接的地方。表示层（第 6 层）是关于我们如何打包数据：我们如何序列化，使用什么编码等。我们已经对此进行了广泛的讨论。第 7 层只是我们的应用程序；我将这一层留给你。

那么，让我们编写一些第 5 层的代码！

## 基于 TCP 的聊天应用程序

网络的“hello-world”应用程序是一个聊天应用程序。这类应用程序使我们能够研究系统如何连接并交换数据，而不必处理它们之间传递的数据类型的技术细节。数据类型是应用程序的一部分，我们在 OSI 模型中了解到它是第 7 层。我们对此不感兴趣。第 6 层是表示层，但对于一个简单的聊天应用程序，我们可以简单地处理：我们取一个字符串并将其编码为 UTF8 字节（当然，也可以反向操作）。由于操作系统负责第 1 层至第 3 层，我们只需处理第 4 层和第 5 层。

让我们让它成为现实。

我想在这里使用 TCP，这是一个非常优秀的协议，它为我们提供了可靠性并保证了数据到达的顺序。它也非常容易设置。

服务器看起来是这样的：

```cs
01: using System.Net;
02: using System.Net.Sockets;
03: using System.Text;
04:
05: "Server is starting up.".Dump();
06:
07: var server = new TcpListener(IPAddress.Loopback, 8080);
08: server.Start();
09:
10: "Waiting for a connection.".Dump();
11:
12: var client = await server.AcceptTcpClientAsync();
13: "Client connected".Dump();
14:
15: var stream = client.GetStream();
16: while (true)
17: {
18:     var buffer = new byte[1024];
19:     var bytes = await stream.ReadAsync(buffer, 0, buffer.Length);
20:     var message = Encoding.UTF8.GetString(buffer, 0, bytes);
21:     $"Received message: {message}".Dump();
22:
23:     if (message.ToLower() == "bye")
24:         break;
25:
26:     "Say something back".Dump();
27:     var response = Console.ReadLine();
28:     var responseBytes = Encoding.UTF8.GetBytes(response);
29:     await stream.WriteAsync(responseBytes, 0, responseBytes.           Length);
30:
31:     if (response.ToLower() == "bye")
32:         break;
33: }
34:
35: client.Close();
36: server.Stop();
37: "Connection closed.".Dump();
```

由于发生了很多事情，我决定在这里使用行号。这使得引用我所解释的内容变得容易一些。

在第 7 行，我们创建了一个新的`TcpListener`类实例。这个类处理所有关于通信的细节，但它需要我们从它那里获取一些信息。我们向构造函数提供了两个参数，告诉它所有需要知道的信息。第一个是我们使用的地址。地址是网络适配器的唯一标识符，例如您的以太网或 Wi-Fi 适配器。这个 IP 地址是 OSI 模型的第 3 层，即网络层的一部分。它是 IP 规范的一部分。然而，多个应用程序可以同时在一个计算机中使用网络适配器。我们可以指定端口号以确保所有应用程序都能获得它们所需的数据，并将其发送到线路另一端的正确应用程序。这个或多或少是任意选择的数字决定了连接到该 IP 地址的应用程序将获得哪些数据。这个端口号是 OSI 模型的第 4 层。我说这个数字或多或少是任意的。技术上，你可以选择你想要的任何数字，但关于这些数字有一些约定。由于端口号决定了哪个应用程序获取或发送数据，因此标准有助于确保我们所有人都为相同的应用程序使用相同的端口。例如，Web 服务器默认监听端口`80`，除非它们使用安全的 HTTPS 协议。后者使用端口`443`。有很多“保留”的数字，但技术上，没有任何东西阻止你为你的聊天应用程序使用端口`80`。尽管如此，我不建议这样做：这会混淆其他人。

我想确保我们的聊天服务器监听端口`8080`，这是一个“免费使用”的数字。

我在这里多次使用了“监听”这个词。监听意味着应用程序等待另一个进程连接，无论是我们机器上的还是外部机器上的。将其与等待电话响铃进行比较：你在等待你的铃声响起，并准备好接听。

由于您的机器可以有多个网络适配器，您必须指定您想要监听哪一个。在这种情况下，我选择了一个固定的 IP 地址，`IPAddress.Loopback`，它对应于`127.0.0.1` IPv4 地址。这个地址是本地机器，没有连接到任何实际的适配器。换句话说，我们只监听来自同一物理机器的连接。

第 8 行很简单：我们启动服务器。在第 14 行的`AcceptTcpClientAsync`调用中，我们告诉服务器接受任何传入的连接。

多个客户端可以同时连接到同一个服务器。这里的客户端代表的是已连接的客户端。我们只期望有一个客户端，所以我们不需要处理会话。记住：会话管理是 OSI 模型的第 5 层。我们假设只有一个客户端，并将它存储在变量`client`中。客户端的类型是`TcpClient`，以防你有所疑问。

这个调用是阻塞的，并且只有当客户端连接时才会继续，我们通过第 15 行的消息告诉用户这一点。

一旦我们建立了连接，我们就打开一个流来访问客户端的数据或使我们能够向该客户端发送数据。这个流是`NetworkStream`类型，它是双向的。我们在第 17 行将这个流存储在变量`stream`中。

数据以二进制形式传入。因此，我们使用`ReadAsync`来读取一个数据缓冲区。我假设没有传入的数据超过 1,024 字节。在现实世界的应用中，你可能无法做出这个假设，所以你必须继续读取，直到你有了所有数据。在这里，我们将这些数据存储在一个长度为 1,024 字节的字节数组中（第 20 和 21 行），并将其转换为 UTF8 字符串（第 22 行）。这就是我们的数据呈现方式，它是 OSI 模型的第 6 层。一旦我们有了这个字符串，我们就显示它。如果字符串是“bye”，我们就认为客户端想要断开连接。否则，我们允许服务器端的用户输入一个响应，并在将其转换为另一个字节数组后将其发送给客户端。我们在这里使用相同的流。

如果流中没有更多数据，或者有人在对话中使用“bye”这个词，我们将在第 37 行关闭连接，并在第 38 行停止监听。

客户端的代码非常相似。下面是代码：

```cs
01: using System.Net.Sockets;
02: using System.Text;
03:
04: "Client is starting up.".Dump(ConsoleColor.Yellow);
05:
06: var client = new TcpClient("127.0.0.1", 8080);
07: "Connected to the server. Let's chat!".Dump(ConsoleColor.Yellow);
08: var stream = client.GetStream();
09:
10: while (true)
11: {
12:     "Say something".Dump(ConsoleColor.Yellow);
13:     var message = Console.ReadLine();
14:     var data = Encoding.UTF8.GetBytes(message);
15:     await stream.WriteAsync(data, 0, data.Length);
16:     if (message.ToLower() == "bye")
17:         break;
18:
19:     var buffer = new byte[1024];
20:     var bytesRead = await stream.ReadAsync(buffer, 0, buffer.            Length);
21:     var response = Encoding.UTF8.GetString(buffer, 0, bytesRead);
22:     $"Server says: {response}".Dump(ConsoleColor.Yellow);
23:     if (response.ToLower() == "bye")
24:         break;
25: }
26:
27: client.Close();
28: "Connection closed.".Dump(ConsoleColor.Yellow);
```

在第 6 行，我们创建了一个新的`TcpClient`类实例。同样，我们必须给它一个 IP 地址和一个端口号。这次，我们必须使用一个实际的数字。我们使用`127.0.0.1`，这意味着我们正在寻找同一台机器上的服务器。端口号仍然是`8080`；否则，我们的服务器将看不到任何进入的连接。

这个调用又是阻塞的，所以它不会继续，直到建立了一个连接。一旦我们有了连接，我们就可以访问流，就像第 8 行那样。这个流，再次，是`NetworkStream`类型，所以我们有一个双向连接。

我们为服务器所做的同样的事情。我们假设消息大小为 1,024 字节或更少。我们使用 UTF8 作为编码将字符串转换为字节数组，并从字节数组转换回字符串。我们使用“bye”这个词来表示停止交谈的愿望，并使用`client.Close()`来最终化连接。

如你所见，代码与服务器非常相似。我们在这里简化了许多事情：我们不考虑多个客户端连接到单个服务器的情况。我们对消息大小做了许多假设，并在出错时必须回退或重试机制。当跨机器工作连接时，事情出错是常有的事，所以你必须意识到这一点，并相应地编写代码。然而，由于这与实际的网络代码无关，正如我向你展示的那样，我可以安全地将这留给你去解决。

## UDP

TCP 是一个优秀的协议，但它并非唯一。**UDP** 更直接且更轻量。当然，这也伴随着一些缺点。以下表格中，我概述了这两种协议之间的差异：

| **考虑因素** | **TCP** | **UDP** |
| --- | --- | --- |
| 主要目标 | 可靠性 | 速度 |
| 顺序 | 保证顺序 | 不保证消息顺序 |
| 握手 | 是 | 否 |
| 错误检查 | 是 | 否 |
| 拥塞控制 | 是 | 否 |
| 用例 | 网络浏览、聊天、文件传输、电子邮件 | 视频流、在线游戏、VOIP |

表 6.3：TCP 和 UDP 对比

TCP 是可靠的。消息几乎总是到达。当事情出错时，TCP 会尝试重新发送数据，直到数据被成功交付。UDP 不关心这一点。它只是尽可能快地将数据发送出去。

TCP 确保消息按照发送的顺序到达。然而，UDP 并不保证：消息可能会以不同于离开源地的顺序到达目的地。

TCP 确保另一端准备好通信。UDP 只是开始发送数据。

TCP 检查数据以查看在传输过程中是否发生错误，甚至可以修复一些错误。UDP 并不关心：只要数据被发送，它就对此感到满意。

如果网络拥塞，TCP 可以减慢传输速度以帮助缓解。UDP 会尽可能快地丢弃数据，而不考虑网络条件。

当你必须有一个可靠、无错误的传输数据方式时，TCP 是最佳选择。例如，在聊天中，消息必须以预期的顺序正确传达。然而，UDP 完全是关于速度的。视频流就是一个例子：如果数据流中有时丢失了一部分，那并不是什么大问题。但是，缓慢的流会毁掉体验。

UDP 不常被使用，但它可以成为你工具箱中的一个宝贵工具。

# 使用共享内存在进程间交换数据

到目前为止，我们一直在向同一台计算机上的其他进程发送消息。使用命名管道和套接字，我们也可以使用其他机器。这正是这些协议的美丽之处：它们对网络是透明的。然而，如果你确定你只想留在同一台机器上，使用管道或套接字可能会成为一种负担。这些方法并不是最快的通信方式。在这种情况下，你可能更愿意使用 **共享内存**。

共享内存的设置非常简单。当然，这也伴随着一些缺点。几乎没有任何方法可以保证数据的安全或防止冲突。然而，它的速度非常快；真的，非常快。所以，让我们看看一个示例。

首先，我们来看看如何将数据写入共享内存：

```cs
using System.IO.MemoryMappedFiles;
"Ready to write data to share memory.\nPress Enter to do     so.".Dump(ConsoleColor.Cyan);
Console.ReadLine();
using var mmf = MemoryMappedFile.CreateNew("SharedData", 1024);
// Create a view accessor to write data
using MemoryMappedViewAccessor accessor = mmf.CreateViewAccessor();
byte[] data = System.Text.Encoding.UTF8.GetBytes("Hello from Process     1");
accessor.WriteArray(0, data, 0, data.Length);
"Data written to shared memory. Press any key to     exit.".Dump(ConsoleColor.Cyan);
Console.ReadKey();
```

共享内存就像有一个只存在于内存中的文件。它是在内存中预留的一块区域。它有一个你可以用来识别它的名称。再次强调，它就像一个文件。在这里，我们创建了一个新的`MemoryMappedFile`类的实例，给它一个名称和大小。（在我们的例子中，1,024 字节）。如果你想使用那个文件，你必须获取`MemoryMappedViewAccessor`。你可以通过在`MemoryMappedFile`实例上调用`CreateViewAccessor`来获取它。

然后，你可以从这个访问器中读取和写入数据。

从那个共享文件中读取也同样简单。以下是代码：

```cs
using System.IO.MemoryMappedFiles;
"Wait for the server to finish. \nPress Enter to read the shared     data.".Dump(ConsoleColor.Yellow);
Console.ReadLine();
using var mmf = MemoryMappedFile.OpenExisting("SharedData");
// Create a view accessor to read data
using MemoryMappedViewAccessor accessor = mmf.CreateViewAccessor();
byte[] data = new byte[1024];
accessor.ReadArray(0, data, 0, data.Length);
$"Received message: {System.Text.Encoding.UTF8.GetString(data)}".    Dump(ConsoleColor.Yellow);
```

我们使用几乎与写入者相同的代码。然而，我们不是在内存中创建一个新的文件，而是打开一个现有的文件。我们不需要指定大小，但必须知道名称。

一旦我们有了那个文件，我们就可以使用相同的代码来获取一个访问器。有了它，我们可以读取数据并显示它。简单，不是吗？

再次强调，这是一种在相同机器上的进程之间快速共享数据的方法。然而，也有一些缺点需要注意。例如，任何知道共享内存块名称的进程都可以访问它。没有任何安全性。当然，你可以通过使用加密来规避这一点。

另一个缺点是没有内置的机制来通知进程新的或更改的数据。你必须使用像信号量（semaphores）和互斥锁（mutexes）这样的东西来做这件事。你可以使用实际的文件来设置`FileSystemWatcher`以获得通知，但这对内存中的这些共享文件不可用。

另一个潜在的缺点是它仅适用于 Windows。这可能会限制你以后部署的选择。

但总的来说，共享内存是快速在相同 Windows 机器上的进程之间共享大量数据的好方法。利用它的优势！

# RPC 概述及其在 IPC 中的应用

到目前为止，我们探讨了我们可以共享数据的方法。在大多数情况下，开发者使用这种方法仅仅是为了数据：从一个系统向另一个系统发送有效载荷。然而，有效载荷也可以是其他东西。它们可以是指令，用来指示软件执行某些操作。我们不是在系统中存储、转换和使用数据，而是告诉其他系统执行操作。在这种情况下，我们谈论的是 RPC。

要从外部控制系统，建立通信线路，确保你的安全措施到位，并定义一个协议。

有很多种方法可以做到这一点。在以前，我们曾经使用过 SOAP、DCOM、WCF 和其他技术来做到这一点。

RESTful 服务与 RPC

你可以将 RESTful 服务视为某种 RPC。然而，它们并不相同，我不想在这里深入讨论 RESTful 服务。它们有很多相似之处，但 RESTful 服务背后的基本理念是它们都是关于资源的。调用网络服务通常用于从服务器检索数据。技术上，你可以设置 RESTful 服务仅接受命令，这样它们就是 RPC。这就像把 calzone 当作披萨。技术上，这是正确的，但在实践中存在足够的差异，需要采取不同的方法。因此，我决定不将 RESTful 服务包含在这本书中。如果你选择使用 RESTful 服务与你的系统通信，请随意。

基本上，这非常简单。你想出一个方法来结构和发送命令。只要双方都理解发生了什么，这就可以正常工作。当然，你不必重新发明轮子：存在几个经过良好建立的标准来做这件事。在本章的后面部分，我会向你展示如何使用 gRPC 来做这件事。然而，就像所有标准一样，它们都有代价。有时，你不需要一个已建立框架提供的额外复杂性。有时，你只想向系统发送一个简单的命令。假设你的场景允许一个不太安全和未知的协议。在这种情况下，你可以通过拥有自己的协议来提高速度和内存。

JSON RPC 是实现这一点的最常用方法之一。让我们看看。

## JSON RPC

JSON RPC 就是将你的命令封装在 JSON 结构中，通过网络发送它们，在另一端拦截它们，并执行命令告诉系统执行的操作。

让我们从定义我们想要发送的命令开始：

```cs
[Serializable]
internal class ShowDateCommand
{
    public bool IncludeTime { get; set; }
}
```

我想让客户端通知服务器它需要打印当前的日期。我可能还想包括当前的时间。所以，这是我们创建的命令：带有 `IncludeTime` 字段的 `ShowDateCommand`。

在我的示例中，我将客户端和服务器放在了同一个应用程序中，每个都在不同的任务上运行。我这样做是为了简化。当然，如果你想向同一应用程序的另一个部分发送命令，RPC 就过于冗余了。这甚至是不正确的：它根本不是远程的。然而，对于这个演示，它运行得很好。

对于通信，我选择了一个命名管道。它很容易设置，可以用来在网络中发送消息。除了这些考虑因素，我实际上没有选择这个选项的真正原因，所以你可以做任何你想做的事情。

服务器部分看起来是这样的：

```cs
internal class Server(CancellationToken cancellationToken)
{
    public async Task StartServer()
    {
        "Starting the server".Dump(ConsoleColor.Cyan);
        await using var server = new             NamedPipeServerStream("CommandsPipe");
        "Waiting for connection".Dump(ConsoleColor.Cyan);
        await server.WaitForConnectionAsync(cancellationToken);
        using var reader = new StreamReader(server);
        while (!cancellationToken.IsCancellationRequested)
        {
            var line = await reader.ReadLineAsync();
            if (line == null) break;
            $"Received this command: {line}".Dump(ConsoleColor.Cyan);
            var command = JsonSerializer.                Deserialize<ShowDateCommand>(line);
            if (command is { IncludeTime: true })
                DateTime.Now.ToString("yyyy-MM-dd
                    HH:mm:ss").Dump(ConsoleColor.Cyan);
            else
                DateTime.Now.ToString("yyyy-MM-dd").Dump(ConsoleColor.                    Cyan);
        }
    }
}
```

这个名为 `Server` 的类有一个名为 `StartServer` 的方法。它使用 `CommandsPipe` 名称创建一个 `NamePipeServerStream` 实例。然后，它等待客户端连接。一旦发生这种情况，我们就读取传入的数据。一旦我们得到一个字符串，我们就将其反序列化为正确的格式并执行它告诉的任务：打印当前的日期，并可选地包括时间。

客户端看起来像这样：

```cs
internal class Client(CancellationToken cancellationToken)
{
    public async Task StartClient()
    {
        var newCommand = new ShowDateCommand
        {
            IncludeTime = true
        };
        var newCommandAsJson = JsonSerializer.Serialize(newCommand);
        "Starting the client".Dump(ConsoleColor.Yellow);
        await using var client = new             NamedPipeClientStream("CommandsPipe");
        await client.ConnectAsync(cancellationToken);
        await using var writer = new StreamWriter(client);
        $"Sending this command: {newCommandAsJson}".Dump(ConsoleColor.            Yellow);
        await writer.WriteLineAsync(newCommandAsJson);
        await writer.FlushAsync();
    }
}
```

客户端创建了一个`ShowDateCommand`的实例，并将`IncludeTime`设置为`true`。然后，它使用正确的名称创建了`NamedPipeClientStream`并连接到服务器。最后，它通过电线发送 JSON。这就是全部内容。

为了完整性，我在程序的`Main`方法中给出了初始化服务器和客户端的代码：

```cs
var cancellationTokenSource = new CancellationTokenSource();
"Starting the server".Dump(ConsoleColor.Green);
var server = new Server(cancellationTokenSource.Token);
Task.Run(() => server.StartServer(), cancellationTokenSource.Token);
var client = new Client(cancellationTokenSource.Token);
    Task.Run(() => client.StartClient(),
    cancellationTokenSource.Token);
"Server and client are running, press a key to stop".    Dump(ConsoleColor.Green);
var input = Console.ReadKey();
"Stopping all".Dump(ConsoleColor.Green);
```

我创建了`Server`和`Client`的实例，在`Task.Run()`中启动它们，并等待用户按下键。在后台，`Server`和`Client`执行它们的工作，通过调用`Dump()`告诉你所有关于它的事情。请注意`Dump`中的线程 ID——它们对于了解线程（或刷新你的记忆）非常有信息量。

这种技术简单且非常快。然而，它仅在你知道等式的两端：服务器和客户端必须遵循你的专有协议。如果不是这样，你最好使用一个标准。这些标准之一就是 gRPC。让我们看看下一个。

# gRPC 概述及其用于 IPC 的使用方法

目前建立进程间直接通信的领先方式之一是 gRPC。缩写**gRPC**代表**Google 远程过程调用**或递归命名的**gRPC 远程过程调用**。你可以选择你喜欢的。谷歌将其开发为一个公开版本和改进其内部框架 Stubby 的版本。

gRPC 使用**协议缓冲区**（**Protobufs**）。这是一种描述可用命令、消息和可以传递的参数的格式。Protobufs 被编译成二进制形式，从而实现更快的数据传输。该系统建立在 HTTP/2 之上，因此我们可以使用多路复用（多个请求通过相同的 TCP 连接）。HTTP/2 比旧的 HTTP/1.x 具有更多优势，其中大多数涉及效率。

跨语言和平台支持也是主要的驱动因素之一。因此，你可以确信 gRPC 可以在许多设备上使用。

假设我们想要重新构建一个示例系统，该系统可以远程指示显示当前日期（带或不带时间）。在这种情况下，我们首先必须定义消息结构。然而，在我们这样做之前，我们需要向我们的服务器应用程序添加几个 NuGet 包：

| **包** | **描述** |
| --- | --- |
| `Google.Protobuf` | 处理 proto 文件 |
| `Grpc.Core` | gRPC 的核心实现 |
| `Grpc.Tools` | 包含，例如，proto 文件的编译器 |
| `Grpc.AspNetCore` | 需要在我们的应用程序中托管服务器 |

表 6.4：我们的 gRPC 服务器的 NuGet 包

在 C#控制台应用程序中，添加一个名为`displayer.proto`的新文件。这只是一个文本文件。我喜欢将它们放在一个单独的文件夹中，我称之为`Protos`。编译器会处理这个文件，为我们生成大量的 C#代码。

文件看起来像这样：

```cs
syntax = "proto3";
option csharp_namespace = "_02_GRPC_Server";
service TimeDisplayer {
    rpc DisplayTime (DisplayTimeRequest) returns (DisplayTimeReply);
}
message DisplayTimeRequest{
    string name = 1;
    bool wantsTime = 2;
}
message DisplayTimeReply{
    string message = 1;
}
```

让我们分析一下。

首先，我们告诉系统这是什么格式。我们使用`proto3`，这是最新也是推荐的版本。

然后，我们告诉系统在生成 C#文件时将它们放在哪个命名空间中。正如你所想象的，这个选项仅限于 C#。这是一个辅助选项，帮助我们保持代码的整洁。

然后，我们定义服务。我们有一个名为`TimeDisplayer`的服务。它有一个名为`DisplayTime`的 RPC 方法。它以`DisplayTimeRequest`作为参数，并返回`DisplayTimeReply`类型的某个东西。

`DisplayTimeRequest`和`DisplayTimeReply`类型定义在下面。它们是消息，并且可以包含参数。我添加了一个名字来展示如何添加一个字符串。对于请求，我还添加了一个布尔值，表示我们是否想要显示时间。

参数需要按顺序和编号。这样，如果消息以某种方式被打乱，两个系统仍然知道数据最初看起来是什么样子。

Visual Studio 通常知道如何处理这个，如果你在你的应用程序中添加了一个`.proto`文件。然而，如果这没有发生（我偶尔也看到它出错），你必须指导编译器如何处理这个文件。在你的`csproj`文件中，只需添加以下部分：

```cs
<ItemGroup>
  <ProtoBuf Include="Protos\displayer.proto" GrpcServices="Server" />
</ItemGroup>
```

这应该足以让编译器开始工作了。

让我们构建服务器！

我在我的控制台应用程序中添加了服务器的代码。由于编译器会为我们编译所有必要的代码，我们可以使用以下代码：

```cs
internal class TimeDisplayerService : TimeDisplayer.TimeDisplayerBase
{
    public override Task<DisplayTimeReply> DisplayTime(
        DisplayTimeRequest request,
        ServerCallContext context)
    {
        var result = request.WantsTime
            ? DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")
            : DateTime.Now.ToString("yyyy-MM-dd");
        result.Dump();
        return Task.FromResult(new DisplayTimeReply
        {
            Message = $"I printed {result}"
        });
    }
}
```

我们的`TimeDisplayerService`类是从`TimeDisplayer.TimeDisplayerBase`基类派生的。这个基类是由我们的`.proto`文件生成的。正如你所看到的，`TimeDisplayer`的名字与我们在那个`.proto`文件中的名字匹配。

我们这里有一个名为`DisplayTime`的方法。再次，这与我们在`.proto`文件中的内容匹配。代码很简单；它只接受一个`DisplayTimeRequest`实例，查看`WantsTime`参数，并返回结果。

通常，gRPC 服务器运行在一些类型的 web 服务器上，将此代码添加到 ASP.NET 应用程序中是直接的。但当然，你可以在任何你想运行它的地方运行它，这是我们作为系统程序员真正可以利用的事情。所以，如果你打算在控制台应用程序中运行此代码，你可以按照以下方式设置。在你的程序的主要方法中，添加以下内容：

```cs
"Starting gRPC server...".Dump();
var port = 50051;
var server = new Server
{
    Services = {TimeDisplayer.BindService(new         TimeDisplayerService())},
    Ports = {new ServerPort("localhost", port, ServerCredentials.        Insecure)}
};
server.Start();
Console.WriteLine("Greeter server listening on port " + port);
Console.WriteLine("Press any key to stop the server...");
Console.ReadKey();
await server.ShutdownAsync();
```

我们创建了一个新的`Server`类实例。这来自于我们安装的`gRPC.Core` NuGet 包。我们给它提供了我们想要使用的服务（在我们的例子中，是`TimeDisplayerService`）并定义了我们决定使用的网络地址和端口。在这里我不关心凭证，但你可以使用 SSL、TLS 和其他安全方式。

我们启动服务器并等待用户按下任意键。然后，我们再次停止服务器。

接下来：客户端。

再次，我们需要向我们的控制台应用程序添加一些 NuGet 包。这些是你需要的：

| **包** | **描述** |
| --- | --- |
| `Google.Protobuf` | 处理 proto 文件 |
| `Grpc.Net.Client` | gRPC 的客户端实现 |
| `Grpc.Tools` | 包含编译器，用于处理 proto 文件等 |

表 6.5：我们的 gRPC 客户端所需的 NuGet 包

首先，我们需要一个 `.prot`o 文件。更准确地说，我们需要与服务器上使用的相同的 `.proto` 文件。因此，最好链接到该文件而不是重新创建它。然而，如果你喜欢键入，请随意创建一个新的。只需确保在做出更改时这些文件保持同步即可。

我们不需要特定的客户端类；我们只需在我们的程序 `Main` 方法中添加以下代码即可：

```cs
"Starting gRPC client... Press ENTER to connect.".Dump(ConsoleColor.Yellow);
Console.ReadLine();
var channel = GrpcChannel.ForAddress("http://localhost:50051");
var client = new
TimeDisplayer.TimeDisplayerClient(channel);
var reply =
    await client.DisplayTimeAsync(
        new DisplayTimeRequest
        {
            Name = "World",
            WantsTime = false
        });
Console.WriteLine("From server: " + reply.Message);
```

我们从等待用户按下一个键开始。由于我在解决方案中同时启动服务器和客户端，如果客户端在设置连接时比服务器快一点，我可能会遇到时序问题。

然后，我们使用正确的参数调用 `GrpcChannel.ForAddress()` 来设置连接。有了这个连接，我们使用正确的 `DisplayTimeRequest` 设置调用 `DisplayTimeAsync` 方法。结果应该返回并显示服务器执行的操作。

就这些了！我们现在有一个完全功能的服务器和客户端应用程序，它们通过 gRPC 进行通信。

## JSON RPC 和 gRPC 的差异

正如你所见，设置 gRPC 服务器和客户端并不太复杂。但仍然，它会给你的代码增加一些复杂性。如果你不需要 gRPC 的优势，你可以使用 JSON RPC。但你在什么时候选择哪一个呢？

如果你的消息变得很大，gRPC 是更好的选择。记得我之前说过 I/O 很慢吗？嗯，JSON 文件通常比它们的二进制等效文件大得多。gRPC 使用较小的二进制格式，因此使用该格式进行数据传输要快得多。

然而，JSON 更易于阅读，更易于调试，也更易于人类解释。代码也更容易设置。`.proto` 文件是你必须习惯的东西。除此之外，编译器需要将 `.proto` 文件转换为 C# 类，这会使你的系统更加复杂。

总的来说，这取决于你的场景。然而，为了便于参考，我在下表中概述了 JSON RPC 和 gRPC 之间的差异：

| **特性** | **gRPC** | **使用 JSON 的 RPC** |
| --- | --- | --- |
| **序列化格式** | Protobufs（二进制格式） | JSON（文本格式） |
| **性能** | 由于二进制序列化，通常更高，初始设置和连接可能较慢 | 低于二进制格式，但设置更快（取决于通信设置） |
| **协议** | HTTP/2 | 通常为 HTTP/1.1 |
| **流式传输** | 支持双向流 | 有限支持，通常是请求-响应 |
| **类型安全** | 强类型合约（Protobuf） | 松散类型，容易在运行时出现错误 |
| **语言互操作性** | 高（原生支持许多语言） | 高（JSON 被普遍支持） |
| **网络效率** | 更高效（更小的负载，HTTP/2 特性） | 更低效（更大的负载，HTTP/1.1） |
| **错误处理** | 丰富的错误处理，具有明确的错误代码 | 通常依赖于 HTTP 状态代码 |
| **截止日期/超时** | 原生支持指定调用截止日期 | 通常在应用层管理 |
| **安全性** | 支持各种身份验证机制 | 通常在应用层添加 |

表 6.6：gRPC 和 JSON RPC 之间的差异

如您所见，尽管 gRPC 和使用 JSON 的 RPC 具有许多共同特性，但每个都有其特定的使用场景。选择最适合您场景的那个。

# 下一步

*每个人都需要有人陪伴*。这个真理甚至成为了一首歌的标题。对于系统来说，尤其是那些不是为人类使用而设计的系统，也是如此。它们需要某种东西来告诉它们该做什么，以及使用什么数据来做。它们需要相互通信。您现在已经看到了许多可以用来设置通信的方法。

我们已经探讨了 Windows 消息，这种传统的通信风格（尽管 Windows 仍然用于内部通信）。我们研究了命名管道和无名管道。然后，我们探讨了计算机之间最常用的通信方式：套接字。在这个过程中，我们还对 OSI 模型进行了研究，以了解我们需要在哪里编写代码，以及我们可以将哪些留给他人。

我们还探讨了在相同机器上使用共享内存快速共享数据的方法。

最后，我们研究了如何通过使用 JSON RPC 和 gRPC 来发布命令。

现在，我们应该准备好迈出下一步。毕竟，除了与我们的代码进行交流外，我们还可以使用操作系统来帮助我们。Windows 提供了许多我们可能需要或可以利用的服务，这是下一章的主题。
