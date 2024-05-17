# 前言

欢迎阅读《C#中的清晰代码》。你将学习如何识别问题代码，尽管它可以编译，但不利于可读性、可维护性和可扩展性。你还将了解各种工具和模式，以及重构代码使其更清晰的方法。

# 本书适合对象

这本书面向对 C#编程语言有一定了解的计算机程序员，他们希望在 C#中识别问题代码并编写清晰的代码时得到指导。主要读者群将从研究生到中级程序员，但即使是高级程序员也可能会发现这本书有价值。

# 本书涵盖内容

第一章《C#中的编码标准和原则》探讨了一些良好的代码与糟糕的代码。当你阅读本章时，你将了解为什么需要编码标准、原则、方法和代码约定。你将学习模块化和设计准则 KISS、YAGNI、DRY、SOLID 和奥卡姆剃刀。

第二章《代码审查-流程和重要性》带领你了解代码审查的流程，并提供其重要性的原因。在本章中，你将了解准备代码进行审查的流程，领导代码审查，知道什么需要审查，知道何时发送代码进行审查，以及如何提供和回应审查反馈。

第三章《类、对象和数据结构》涵盖了类组织、文档注释、内聚性、耦合性、迪米特法则和不可变对象和数据结构等广泛主题。在本章结束时，你将能够编写良好组织且只有单一职责的代码，为代码的使用者提供相关文档，并使代码具有可扩展性。

第四章《编写清晰的函数》帮助你了解函数式编程，如何保持方法的小型化，以及如何避免代码重复和多个参数。在完成本章之后，你将能够描述函数式编程，编写函数式代码，避免编写超过两个参数的代码，编写不可变的数据对象和结构，保持方法的小型化，并编写符合单一职责原则的代码。

第五章《异常处理》涵盖了已检查和未检查的异常，空指针异常以及如何避免它们，同时还涵盖了业务规则异常，提供有意义的数据，以及构建自定义异常。

第六章《单元测试》带领你使用 SpecFlow 使用行为驱动开发（BDD）软件方法和使用 MSTest 和 NUnit 使用测试驱动开发（TDD）。你将学习如何使用 Moq 编写模拟（伪造）对象，以及如何使用 TDD 软件方法编写失败的测试，使测试通过，然后在通过后重构代码。

第七章《端到端系统测试》指导你通过一个示例项目手动进行端到端测试的过程。在本章中，你将进行端到端（E2E）测试，代码和测试工厂，代码和测试依赖注入，以及测试模块化。你还将学习如何利用模块化。

第八章《线程和并发》侧重于理解线程生命周期；向线程添加参数；使用`ThreadPool`、互斥体和同步线程；使用信号量处理并行线程；限制`ThreadPool`使用的线程和处理器数量；防止死锁和竞争条件；静态方法和构造函数；可变性和不可变性；以及线程安全。

第九章，*设计和开发 API*，帮助您了解 API 是什么，API 代理，API 设计指南，使用 RAML 进行 API 设计以及 Swagger API 开发。在本章中，您将使用 RAML 设计一个与语言无关的 API，并在 C#中开发它，并使用 Swagger 记录您的 API。

第十章，*使用 API 密钥和 Azure Key Vault 保护 API*，向您展示如何获取第三方 API 密钥，将该密钥存储在 Azure Key Vault 中，并通过您将构建和部署到 Azure 的 API 检索它。然后，您将实现 API 密钥身份验证和授权以保护您自己的 API。

第十一章，*解决横切关注点*，介绍了使用 PostSharp 来解决横切关注点的方法，使用方面和属性构成了面向方面的开发的基础。您还将学习如何使用代理和装饰器。

第十二章，*使用工具改善代码质量*，向您介绍了各种工具，这些工具将帮助您编写高质量的代码并改善现有代码的质量。您将接触到代码度量和代码分析，快速操作，JetBrains 工具 dotTrace Profiler 和 Resharper，以及 Telerik JustDecompile。

第十三章，*重构 C#代码-识别代码异味*，是两章中的第一章，带您了解不同类型的问题代码，并向您展示如何修改它以成为易于阅读，维护和扩展的清洁代码。每章都按字母顺序列出代码问题。在这里，您将涵盖类依赖关系，无法修改的代码，集合和组合爆炸等主题。

第十四章，*重构 C#代码-实现设计模式*，带您了解创建和结构设计模式的实现。在这里，简要介绍了行为设计模式。然后，您将对清洁代码和重构进行一些最终思考。

# 为了充分利用本书

大多数章节可以独立阅读，顺序不限。但为了充分利用本书，建议按照提供的顺序阅读章节。在阅读章节时，请按照说明执行任务。然后，在完成章节时，回答问题并进行推荐的进一步阅读，以加强所学知识。为了充分利用本书的内容，建议您满足以下要求：

| **本书涵盖的软件/硬件** | **要求** |
| --- | --- |
| Visual Studio 2019 | Windows 10, macOS |
| Atom | Windows 10, macOS, Linux: [`atom.io/`](https://atom.io/) |
| Azure 资源 | Azure 订阅：[`azure.microsoft.com/en-gb/`](https://azure.microsoft.com/en-gb/) |
| Azure Key Vault | Azure 订阅：[`azure.microsoft.com/en-gb/`](https://azure.microsoft.com/en-gb/) |
| Morningstar API | 从[`rapidapi.com/integraatio/api/morningstar1`](https://rapidapi.com/integraatio/api/morningstar1)获取您自己的 API 密钥 |
| Postman | Windows 10, macOS, Linux: [`www.postman.com/`](https://www.postman.com/) |

在开始阅读和逐章阅读的过程中，如果您已经具备以下条件，将会很有帮助。

**如果您使用的是本书的数字版本，我们建议您自己输入代码或通过 GitHub 存储库（链接在下一节中提供）访问代码。这样做将帮助您避免与复制和粘贴代码相关的任何潜在错误。**

您应该具有使用 Visual Studio 2019 社区版或更高版本的基本经验，以及基本的 C#编程技能，包括编写控制台应用程序。许多示例将以 C#控制台应用程序的形式呈现。主要项目将使用 ASP.NET。如果您能够使用框架和核心编写 ASP.NET 网站，那将会很有帮助。但不用担心-您将被引导完成所需的步骤。

## 下载示例代码文件

您可以从您的帐户在[www.packt.com](http://www.packt.com)下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](https://www.packtpub.com/support)注册，以便文件直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packt.com](http://www.packt.com)。

1.  选择“支持”选项卡。

1.  点击“代码下载”。

1.  在“搜索”框中输入书名，然后按照屏幕上的说明操作。

文件下载完成后，请确保使用最新版本的解压缩软件解压缩文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Clean-Code-in-C-`](https://github.com/PacktPublishing/Clean-Code-in-C-)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有其他代码包，来自我们丰富的图书和视频目录，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。快去看看吧！

## 下载彩色图像

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。您可以在这里下载：[`static.packt-cdn.com/downloads/9781838982973_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781838982973_ColorImages.pdf)。

## 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。例如：“`InMemoryRepository`类实现了`IRepository`的`GetApiKey()`方法。这将返回一个 API 密钥字典。这些密钥将存储在我们的`_apiKeys`字典成员变量中。”

代码块设置如下：

```cs
using CH10_DividendCalendar.Security.Authentication;
using System.Threading.Tasks;

namespace CH10_DividendCalendar.Repository
{
    public interface IRepository
    {
        Task<ApiKey> GetApiKey(string providedApiKey);
    }
}
```

任何命令行输入或输出都将被写成如下形式：

```cs
az group create --name "<YourResourceGroupName>" --location "East US"
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会以这种形式出现在文本中。例如：“要创建应用服务，请右键单击您创建的项目，然后从菜单中选择发布。”

警告或重要说明会出现在这样。

技巧和窍门会出现在这样。
