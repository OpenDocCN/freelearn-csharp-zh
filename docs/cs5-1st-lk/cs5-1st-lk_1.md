# 第一章：入门 C#

在这一章中，我们将讨论 C#首次推出时行业的一般状况，以及它成为一种优秀语言的一些原因。在本章结束时，你将拥有一个完全可用的开发环境，准备好运行本书中的所有示例。

# 起源

每个漫画超级英雄都有一个起源故事，每个行业的专业人士也是如此。与同事分享起源故事很棒，因为它可以作为对过去情况的反思，以及对事物如何演变和未来可能走向的思考。我的个人故事起源于上世纪 90 年代末的高中时期，当时我看着比我大五岁、正在上大学的哥哥学习 C++。通过一些神秘的指令，复杂的程序变得生动起来，准备好行动。我着迷了。

这种力量的初次体验只是开始。大约在同一时间，我班上的一个朋友开始制作一款游戏，同样是用 C++编写，风格类似于 NES 游戏《塞尔达传说》。虽然我曾经短暂地瞥见过旧的 QBasic 游戏，比如《Gorillas》，但我对他在小型演示中所取得的质量感到惊讶。我决定认真学习如何编程，考虑到我认识的每个人都在使用 C++，这成为了我第一门编程语言的默认选择。

我写的第一个程序是一个非常简单的财务预算程序。当时我刚刚开始在高中的第一份工作，我非常清楚管理金钱所涉及的新责任，所以我想写一个程序来帮助我更好地管理我的资金。首先，它要求输入我的工资金额，然后输入我必须支付的账单清单。

经过一些基本的计算，它给我提供了一个报告，告诉我在照顾好我的责任之后还剩下多少可支配收入。就程序而言，它并不是最复杂的软件，但它帮助我学会了一些基础知识，比如循环、条件语句、存储不确定数量项目的列表，以及对数组执行聚合操作。

这是一个巨大的个人胜利，但在初步探索 C++后，我发现自己遇到了一些困难。对于一个刚接触编程的人（还在上高中），C++很难完全掌握。我不仅需要学习软件的基础知识，还必须时刻注意我使用的内存。最终，我发现了当时对我来说更简单的网页开发工具。我从复杂性谱的一端移动到了另一端。

当时的软件领域大部分被计算机语言所主导，这些语言可以分为三大类：低级系统语言，比如 C++，在性能和灵活性方面提供了最多的功能，但也很难和复杂；解释性语言，比如 JavaScript 和 VBScript，其指令在运行时被评估，非常易于使用和学习，但无法与低级语言的性能相匹敌；最后是一组介于两者之间的语言。

这条中间路线涵盖了诸如 Java 和 Visual Basic 等语言，既提供了最好的一面，也带来了最糟糕的一面。在这些语言中，你有一个**垃圾收集器**，这意味着当你创建一个对象时，你不必在完成后显式释放已使用的内存。它们还被编译成中间语言（例如，VB 的 p 代码和 Java 的字节码），然后在目标平台上本地运行的虚拟机中执行。由于这种中间语言类似于机器代码，它能够比纯解释语言执行得更快。然而，这种性能仍然不是真正与经过适当调整的 C++程序相匹敌的，因此与 C++相比，Java 和 Visual Basic 程序通常被认为是慢语言。

尽管存在一些缺点，但对于微软来说，拥有一个受管理的内存环境的好处是显而易见的。因为程序员不必担心指针和手动内存管理等复杂概念，程序可以更快地编写，并且出现更少的错误。**快速应用程序开发**（简称**RAD**）似乎是微软平台的未来方向。

在 90 年代末，他们开发了一个 Java 虚拟机的版本，据许多人说比市场上其他实现更快。不幸的是，由于他们包含了一些专有扩展，并且他们没有完全实现 Java 1.1 标准，他们在 1997 年遇到了一些法律问题。这最终导致微软停止了他们对 Java 实现的开发，并最终在 2001 年将其从他们的平台中移除。

虽然我们无法知道接下来发生的事情是否是对微软 Java 虚拟机的法律行动的直接结果，但我们知道的是，1999 年微软开始研发一种名为**Cool**（**类 C 对象导向语言**）的新编程语言。

## C#诞生

然后发生了这件事；在 2000 年，微软宣布他们正在开发一种新的编程语言。这种最初被称为 Cool 的语言在 2000 年奥兰多的专业开发者大会上作为**C#**公布。这种新语言的一些亮点是：

+   它基于 C 系列编程语言的语法，因此对于有 C++、Java 或 JavaScript 经验的人来说，语法非常熟悉。

+   C#的内存管理类似于 Java 和 Visual Basic，具有非常强大的垃圾收集器。这意味着用户可以专注于他们应用程序的内容，而不必担心样板式的内存管理代码。

+   C#编译器以及静态类型系统意味着某些类别的错误可以在编译时捕获，而不必像在 JavaScript 中那样在运行时处理它们。这是一个**即时编译器**，这意味着代码将在运行时编译为本机可执行文件，并针对执行代码的操作系统进行优化。性能是新平台的一个重要目标。

+   这种语言具有强大且广泛的**基类库**，这意味着许多功能块将直接内置到框架中。除了一些行业标准库，如 Boost，没有很多常见的 C/C++库，这导致人们经常重写常见功能。另一方面，Java 有很多库，但它们是由不同的开发人员编写的，这意味着功能和风格的一致性是一个问题。

+   它还与其他在**公共语言运行时**（**CLR**）上运行的语言具有互操作性。因此，一个程序可以使用用不同语言编写的功能，从而为每种语言使用其最擅长的功能。

+   微软向 ISO 工作组提交了规范。这为框架周围的充满活力的开源社区打开了大门，因为这意味着总会有一个标准可供参考。一个名为**Mono**的流行开源实现了.NET Framework 和 C#，让你可以在不同的平台上运行你的代码。

尽管这个列表中描述的元素都不是特别新的，但 C#的目标是吸收以前的编程语言的最佳特点，包括 C++的强大和功能、JavaScript 的简单性以及 VBScript/ASP 的易于托管等。

任何语言（C、C++或 Java）的人都可以很轻松地在 C#中提高生产力。C#找到了生产力、功能和学习曲线的完美交汇点。

在接下来的十年里，这种语言将继续发展出一套非常有吸引力的功能，使编写优秀的程序变得更加容易和快速。现在在其第五个版本中，C#语言已经变得更加表达力和强大，具有**语言集成查询**（**LINQ**）、**任务并行库**（**TPL**）、**动态语言运行时**（**DLR**）和异步编程功能等特性。更重要的是，通过 Mono 框架，你不仅可以针对 Windows，还可以针对其他主流平台，如 Linux、Mac OS、Android、iOS，甚至游戏主机，如 Playstation Vita。

无论你过去十年是否一直在写 C#，还是刚刚开始学习，这本书都会带你了解最新版本 5.0 的所有功能。我们还将探讨 C#的演变和历史，以便你了解为什么某些功能会以这种方式发展，以及如何充分利用它们。

不过，在我们开始之前，我们需要配置你的计算机，以便能够编译所有的示例。本章将指导你安装所有你需要的东西，以便你能够完成本书中的每个示例。

# 工具

每当你接触一个新的编程语言或工具时，你可以问自己几个问题，以便快速熟练掌握这个环境，比如：

+   你如何构建一个程序，或者为部署做好准备？

+   你如何调试一个程序？快速找出问题所在以及问题出现的位置。这和一开始编写程序一样重要。

在接下来的章节中，我们将回顾一些可用的工具，以便在你的本地计算机上建立一个开发环境。这些选项在许多不同的许可条款和成本结构之间变化。无论你的情况或偏好如何，你都能够建立一个开发环境，并且在本章结束时你将能够回答之前的问题。

## Visual Studio

微软为 C#语言提供了事实上的编译器和开发环境。尽管自.NET Framework 首次发布以来，编译器就作为一个命令行可执行文件可用，但大多数开发人员会留在 Visual Studio 的范围内，这是微软的**集成开发环境**（**IDE**）。

### Visual Studio 的完整版本

微软的 Visual Studio 完整商业版本有几个不同的版本，每个版本都有累积的功能数量，随着你的提升而增加。

+   **专业版**：这是基本的商业套餐。它允许你在所有可用的语言中构建所有可用的项目。在 C#的上下文中，一些可用的项目类型包括 ASP.NET WebForms、ASP.NET MVC、Windows 8 应用、Windows Phone、Silverlight、库、控制台，以及一个强大的测试框架。

+   **高级版**：在这个版本中，除了包括所有专业功能外，还包括了代码度量、扩展测试工具、架构图、实验室管理和项目管理功能。

+   **Ultimate**：此版本包括代码克隆分析、更多测试工具（包括 Microsoft Fakes）和 IntelliTrace，以及所有先前级别的功能。

在[`www.microsoft.com/visualstudio/11/enus/products/visualstudio`](http://www.microsoft.com/visualstudio/11/enus/products/visualstudio)上查看这些版本。

### 许可证

完整版本的 Visual Studio 有几种不同的许可证选项。

+   **MSDN 订阅**：微软开发人员网络提供订阅服务，您可以支付年费以获得 Visual Studio 的各个版本。此外，您可以作为微软 MVP 计划的一部分获得 MSDN 订阅，该计划奖励开发社区中的活跃成员。您可以在[`msdn.microsoft.com/en-us/subscriptions/buy/buy.aspx`](https://msdn.microsoft.com/en-us/subscriptions/buy/buy.aspx)找到有关购买 MSDN 订阅的更多信息。

+   **BizSpark**：如果您正在创建一家初创公司，微软提供 BizSpark 计划，让您在三年内免费获得 Microsoft 软件（包括 Visual Studio）的访问权限。毕业后，您可以保留在计划过程中下载的许可证，并获得 MSDN 订阅的折扣，以及其他校友福利。BizSpark 是任何想要使用微软技术堆栈的企业家的绝佳选择。在[`www.microsoft.com/bizspark`](http://www.microsoft.com/bizspark)上查看您是否符合 BizSpark 计划的资格。

+   **DreamSpark**：学生可以参加 DreamSpark 计划，该计划允许您下载 Visual Studio Professional（以及其他应用程序和服务器）。只要您是有效学术机构的学生，您就可以访问使用 C#编写应用程序所需的一切。立即在[`www.dreamspark.com/`](https://www.dreamspark.com/)注册。

+   **个人和批量许可**：如果之前的商业版 Visual Studio 的选项都不合适，那么您可以直接从微软或各种经销商购买许可证，网址为[`www.microsoft.com/visualstudio/en-us/buy/small-midsize-business`](http://www.microsoft.com/visualstudio/en-us/buy/small-midsize-business)。

### Express

**Visual Studio Express**产品系列是一个几乎完整功能的 Visual Studio 版本，而且是免费的。任何人都可以下载这些产品并开始学习和开发，而无需付费。

可用的版本如下：

+   **Visual Studio Express 2012 for Windows 8**：用于创建 Windows 8 的 Metro 样式应用程序

+   **Visual Studio Express 2012 for Windows Phone**：这让您为微软的 Windows Phone 设备编写程序。

+   **Visual Studio Express 2012 for Web**：所有 Web 应用程序都可以使用这个版本的 Visual Studio 构建，从 ASP.NET（表单和 MVC）到 Azure 托管项目

+   **Visual Studio Express 2012 for Desktop**：可以使用这个版本构建针对*经典*Windows 8 桌面环境的应用程序。

人们普遍误解 Visual Studio Express 只能用于非商业项目，但事实并非如此。您完全可以在遵守最终用户许可协议的情况下开发和发布商业产品。唯一的限制是技术上的，如下所示：

+   Visual Studio 的 Express 版本受垂直堆栈的限制，这意味着您必须为每种受支持的项目类型（Web、桌面、手机等）安装单独的产品。不过，这几乎不是一个巨大的限制，在最复杂的解决方案中才会成为负担。

+   没有插件。对于全版本的 Visual Studio，有许多提高生产力的插件可用，因此对于一些用户来说，这种排除可能是一个大问题。然而，好消息是，最近记忆中最受欢迎的插件之一，**NuGet**，现在已经随 Visual Studio 2012 的所有版本一起发布。NuGet 帮助您管理项目的库依赖关系。您可以浏览 NuGet 目录，并添加开源第三方库，以及来自 Microsoft 的库。

Visual Studio 的 Express 版本可以从[`www.microsoft.com/visualstudio/11/en-us/products/express`](http://www.microsoft.com/visualstudio/11/en-us/products/express)下载。

### 使用 Visual Studio

无论您决定使用哪个版本的 Visual Studio，一旦产品安装完成，入门都非常简单。以下是步骤：

1.  启动 Visual Studio，或者如果您使用 Express 版本，则启动 Visual Studio Express for Desktop。

1.  通过点击**文件** | **新建项目...**来创建一个新项目。

1.  从**已安装** | **模板** | **Visual C#**中选择**控制台应用程序**。

1.  给项目取一个名称，比如`program`，然后点击**确定**。

1.  在`Main`方法中添加一行代码如下：

```cs
Console.WriteLine("Hello World");
```

1.  通过选择**调试** | **无调试运行**来运行程序。

您将看到预期的**Hello World**输出，现在可以开始使用 Visual Studio 了。

#### 命令行编译器

如果您喜欢以比 Visual Studio 这样的 IDE 更低的级别工作，您总是可以选择直接使用命令行编译器。微软通过从[`www.microsoft.com/en-us/download/details.aspx?id=8483`](http://www.microsoft.com/en-us/download/details.aspx?id=8483)下载和安装.NET 4.5 可再发行包，为您提供了免费编译 C#代码所需的一切。

一旦下载并安装了，您可以在`C:\windows\microsoft.net\Framework\v4.0.30319\csc.exe`找到编译器，假设您保留了所有默认安装选项：

### 注意

请注意，如果您安装了.NET Framework 4.5 版本，它实际上会替换 4.0 框架。这就是为什么之前提到的路径显示为`v4.0.30319`。你不会是第一个被.NET Framework 版本搞糊涂的人。

使命令行编译器更容易使用的一个小技巧是将其添加到环境的`Path`变量中。如果您使用 PowerShell（我强烈鼓励），您可以通过运行以下命令轻松实现：

```cs
PS ~> $env:Path += ";C:\Windows\Microsoft.NET\Framework\v4.0.30319"

```

这样你就可以只输入`csc`而不是整个路径。命令行编译器的使用非常简单，看下面的类：

```cs
using System;

namespace program
{
    class MainClass
    {
        static void Main (string[] args)
        {
            Console.WriteLine("Hello, World");
        }
    }
}
```

将此类保存为名为`program.cs`的文件，使用您喜欢的文本编辑器。保存后，您可以使用以下命令从命令行编译它：

```cs
PS ~\book\code\ch1> csc .\ch1_hello.cs

```

这将生成一个名为`ch1_hello.exe`的可执行文件，当执行时，将产生一个熟悉的问候，如下所示：

```cs
PS ~\book\code\ch1> .\ch1_hello.exe
Hello, World

```

默认情况下，`csc`将输出一个可执行文件。但是，您也可以使用目标参数来生成库。考虑以下类：

```cs
using System;

namespace program
{
    public class Greeter
    {
        public void Greet(string name)
        {
            Console.WriteLine("Hello, " + name);
        }
    }
}
```

这个类封装了前一个程序的功能，并且通过让您定义要问候的名称，甚至使其可重用。尽管这是一个有点陈词滥调的例子，但重点是要展示如何创建一个`.dll`文件，您可以从多个程序中使用。

```cs
PS ~\dev\book\code\ch1> csc /target:library .\ch1_greeter.cs

```

将生成一个名为`ch1_greeter.dll`的程序集，然后您可以从稍微修改过的前一个程序中使用它，如下所示：

```cs
using System;

namespace program
{
    class MainClass
    {
        static void Main (string[] args)
        {
            Greeter greeter = new Greeter();
            greeter.Greet("Componentized World");
        }
    }
}
```

如果您尝试像以前一样编译前一个程序，编译器将正确地抱怨不知道`Greeter`类的任何信息，如下所示：

```cs
PS ~\book\code\ch1> csc .\ch1_greeter_program.cs
Microsoft (R) Visual C# Compiler version 4.0.30319.17626
for Microsoft (R) .NET Framework 4.5
Copyright (C) Microsoft Corporation. All rights reserved.

ch1_greeter_program.cs(9,13): error CS0246: The type or
        namespace name 'Greeter' could not be found (are you
        missing a using directive or an assembly reference?)
ch1_greeter_program.cs(9,35): error CS0246: The type or
        namespace name 'Greeter' could not be found (are you
        missing a using directive or an assembly reference?)
```

每当程序出现错误时，它将显示在输出中，并提供有关发现错误的文件和行的信息，因此您可以轻松找到它。为了使其工作，您将不得不告诉编译器使用使用`/r:`参数创建的`ch1_greeter.dll`文件，如下所示：

```cs
PS ~\book\code\ch1> csc /r:ch1_greeter.dll .\ch1_greeter_program.cs

```

现在当您运行生成的`ch1_greeter_program.exe`程序时，您将看到输出显示**Hello, Componentized World**。

尽管大多数开发人员现在不会直接使用命令行编译器，但了解它的可用性以及如何使用它是很好的，特别是如果您必须支持高级场景，例如将多个模块合并为单个程序集。

#### SharpDevelop

当您启动 SharpDevelop 时，加载屏幕上的标语**The Open Source .NET IDE**是一个简洁的描述。自.NET Framework 的早期，它为开发人员提供了一个免费的选项来编写 C#，在 Microsoft 发布 Express 版本之前。自那时起，它一直在不断成熟和增加功能，截至 4.2 版本，SharpDevelop 支持针对.NET 4.5 的目标，更具体地说，支持 C# 5.0 的编译和调试。虽然 Visual Studio Express 是一个引人注目的选择，但缺乏源代码控制插件可能会成为一些用户的断档因素。幸运的是，SharpDevelop 将乐意让您在 IDE 中集成源代码控制服务器。此外，一些更为专业的项目类型，如创建 Windows 服务（Express 不支持的少数项目类型之一），在 SharpDevelop 中得到了充分支持。

项目使用与 Visual Studio 相同的格式（.sln，.csproj），因此项目的可移植性很高。通常可以将在 Visual Studio 中编写的项目打开在 SharpDevelop 中。

从[`www.icsharpcode.net/OpenSource/SD/`](http://www.icsharpcode.net/OpenSource/SD/)下载应用程序。

安装非常简单，您可以通过创建以下示例程序来验证正确的安装：

1.  启动 SharpDevelop。

1.  通过单击**文件** | **新建** | **解决方案**来创建一个新项目。

1.  从**C#** | **Windows 应用程序**中选择**控制台应用程序**。

1.  给项目取一个名称，如`program`，然后单击**创建**。

1.  右键单击**项目**窗口中的项目节点，选择**属性**菜单项；检查**编译**选项卡，看看**目标框架**是否说**.NET Framework 4.0 Client Profile**。

1.  如果是这样，只需单击**更改**按钮，在**更改目标框架**下拉菜单中选择**.NET Framework 4.5**，最后单击**转换**按钮。

1.  通过选择**调试** | **无调试运行**来运行程序。

您将看到预期的**Hello World**输出。

#### MonoDevelop

**Mono**框架是 Common Language Runtime 和 C#的开源版本。它经过了十多年的积极开发，因此非常成熟和稳定。Mono 有适用于几乎任何平台的版本，包括 Windows，OS X，Unix/Linux，Android，iOS，PlayStation Vita，Wii 和 Xbox 360。

MonoDevelop 基于 SharpDevelop，但在一段时间前分叉，专门作为运行在多个平台上的 Mono 的开发环境。它可以在 Windows，OS X，Ubuntu，Debian，SLE 和 openSUSE 上运行；因此，作为开发人员，您可以真正选择要在哪个平台上工作。

您可以通过从[`www.go-mono.com/mono-downloads/download.html`](http://www.go-mono.com/mono-downloads/download.html)安装 Mono Development Kit 2.11 或更高版本来开始。

安装了适用于您平台的软件后，您可以继续从[`monodevelop.com/`](http://monodevelop.com/)安装最新版本的 MonoDevelop。

使用 C# 5.0 编译器只需几个简单的步骤：

1.  启动 MonoDevelop。

1.  通过单击**文件** | **新建** | **解决方案…**来创建一个新项目。

1.  从**C#**菜单中选择**控制台应用程序**。

1.  给项目命名为`program`，然后单击“前进”，然后单击“确定”。

1.  在“解决方案”窗口中右键单击项目节点，然后选择“选项”菜单项。现在转到“生成”|“常规”以查看“目标框架”是否为“Mono/.NET 4.0”。

1.  如果是这样，那么只需从下拉菜单中选择“.NET Framework 4.5”，然后单击“确定”按钮。

1.  通过选择“运行”|“开始调试”来运行程序。

如果一切顺利，您将看到一个终端窗口（例如在 OS X 上运行），显示“Hello World”文本。

# 总结

C#语言的引入正好是在合适的时候，它是一种现代的面向对象的语言，汲取了之前许多语言的优点。具有低级别的强大功能和即时编译，垃圾回收环境的简单性，以及运行时的灵活性，可以轻松与其他语言进行互操作，还有一个出色的基类库、蓬勃发展的开源社区，以及能够针对多个平台进行开发的能力，这些都使 C#成为一个引人注目的选择。

在本章中，我们讨论了设置开发环境并下载所有相关工具和运行时的步骤。

+   Visual Studio 有商业和免费选项。

+   命令行对于直接插入使用 shell 命令的自动化工具非常有用。

+   SharpDevelop 是 Visual Studio 的开源替代品。

+   MonoDevelop 是.NET Framework 和 C#的开源实现的官方 IDE。这使您可以针对多个平台进行开发。

一旦您选择了首选的开发环境，并按照本章详细介绍的步骤进行操作，您就可以继续阅读本书的其余部分以及其中包含的所有示例。

在下一章中，我们将讨论语言的演变，这将帮助您了解沿途引入的功能，并有助于我们如今拥有 C# 5.0 的地位。
