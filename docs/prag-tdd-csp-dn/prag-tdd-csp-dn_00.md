# 前言

作为一名顾问，我与多个组织中的许多团队合作过。我见过进行 TDD 的团队，也见过没有进行 TDD 但进行单元测试的团队。我还见过认为自己在进行单元测试但实际上在进行集成测试的团队，以及没有进行任何测试的团队！作为一个普通人，我开始根据经验证据形成信念，认为进行 TDD 的团队是最成功的，但这并不是因为他们使用了 TDD！TDD 的结果来自于拥有激情。

*TDD 是单元测试加上激情*。在一些团队中，单元测试是强制的，因此开发者必须执行它，但 TDD 很少被强制执行，这取决于开发者自己来强制执行。不用说，充满激情的开发者会创造出高质量的项目，而高质量的项目更有可能成功。

TDD 通常与**领域驱动设计**（**DDD**）架构的某些或全部方面相结合。因此，我确保我涵盖了 TDD 和 DDD 的结合，以便能够给出现实生活中的例子。我还想反映今天的市场，它被关系型数据库和文档数据库这两个数据库类别所分割，所以我自由地包括了每个类别的示例章节，并展示了单元测试实现中的差异，目的是使本书保持实用。

不要被本书的篇幅所欺骗，因为图表和代码片段确实增加了本书的篇幅。我努力避免过时和不实用的理论，以缩短本书的篇幅并保持重点突出。

TDD 和单元测试在大多数现代职位说明中都是要求，是面试测试项目的必备条件，也是热门面试问题的主题。如果你想要了解更多关于这些主题的信息，并成为一名 TDD 开发者，那么你来到了正确的地点。

关于 TDD 的许多其他优秀书籍也针对.NET 开发者，那么*为什么这本书？* 在这本书中，我通过进入 DDD 世界、关系型数据库和文档数据库，展示了真正的实际实现。我展示了实践者在进行 TDD 时使用的思维模式的决策树。我展示了 SOLID 与 TDD 之间的关系，并介绍了一套被称为 TDD 的 FIRSTHAND 指南的易于记忆的最佳实践。

我写这本书的目的是让你成为一个自信的 TDD 实践者，或者至少是一个单元测试实践者，我希望我已经实现了我的目标。

# 本书面向对象

测试驱动开发是从第一天开始设计、记录和测试你的应用程序的主流方式。作为一名希望攀登技术阶梯到更高职位的高级开发者，TDD 及其相关的单元测试、测试替身和依赖注入主题是必须学习的。

本书面向中级至高级.NET 开发者，旨在利用 TDD 的潜力来开发高质量的软件。假设读者具备 OOP 和 C#编程概念的基本知识，但不需要了解 TDD 或单元测试。本书全面覆盖了 TDD 和单元测试的所有概念，并作为开发者从零开始构建基于 TDD 的应用或计划将单元测试引入其组织的优秀指南。

# 本书涵盖的内容

本书涵盖了 TDD 及其.NET 生态系统中的 IDE 和库，并介绍了环境设置。本书首先涵盖了构成 TDD 先决条件的话题，即依赖注入、单元测试和测试替身。然后，在介绍 TDD 及其最佳实践之后，本书深入探讨了使用领域驱动设计作为架构从头开始构建应用。

本书还涵盖了构建持续集成管道的基础、处理未考虑可测试性的遗留代码，并以将 TDD 引入组织的想法结束。

*第一章*, *编写您的第一个 TDD 实现*，没有长篇介绍或理论，而是直接进入 IDE 选择和编写您的第一个 TDD 实现，以体验本书内容的味道。

*第二章*, *通过示例理解依赖注入*，复习了理解依赖注入概念所需的先进 OOP 原则，并提供了多个示例。

*第三章*, *开始使用单元测试*，提供了对 xUnit 和单元测试基础简单介绍。

*第四章*, *使用测试替身进行真实单元测试*，介绍了存根、模拟和 NSubstitute，然后讨论了更多测试类别。

*第五章*, *测试驱动开发解析*，展示了如何以 TDD 风格编写单元测试，并讨论了其优缺点。

*第六章*, *TDD 的 FIRSTHAND 指南*，详细介绍了单元测试和 TDD 的最佳实践。

*第七章*, *对领域驱动设计的实用看法*，介绍了 DDD、服务和存储库。

*第八章*, *设计预约应用*，概述了将使用 DDD 架构和 TDD 风格在以后实现的现实应用规范。

*第九章*, *使用 Entity Framework 和关系型数据库构建预约应用*，演示了使用关系型数据库后端的一个 TDD 应用示例。

*第十章*，*使用仓储和文档数据库构建应用程序*，演示了使用文档数据库和仓储模式的一个 TDD 应用程序示例。

*第十一章*，*使用 GitHub Actions 实现持续集成*，展示了如何使用 GitHub Actions 为 *第十章* 中的应用程序构建 CI 管道。

*第十二章*，*处理遗留项目*，概述了在考虑遗留项目的 TDD 和单元测试时的思考过程。

*第十三章*，*推广 TDD 的复杂性*，解释了在组织采用 TDD 时的思维过程。

*附录 1*，*常用库与单元测试*，展示了 MSTest、NUnit、Moq、Fluent Assertions 和 Auto Fixture 的快速示例。

*附录 2*，*高级模拟场景*，展示了使用 NSubstitute 的更复杂的模拟场景。

# 为了充分利用这本书

本书假设您熟悉 C# 语法，并且至少有一年的使用 Visual Studio 或类似 IDE 环境的经验。虽然本书将复习 OOP 原则的高级概念，但本书假设您熟悉基础知识。

| **本书涵盖的软件** | **操作系统要求** |
| --- | --- |
| Visual Studio 2022 | Windows 或 macOS |
| 精细代码覆盖率 | Windows |
| SQL Server | Windows, macOS (Docker), 或 Linux |
| Cosmos DB | Windows, macOS (Docker) 或 Linux (Docker) |
| **库和框架** | **操作系统要求** |
| .NET Core 6, C# 10 | Windows, macOS 或 Linux |
| xUnit | Windows, macOS 或 Linux |
| NSubstitute | Windows, macOS 或 Linux |
| Entity Framework | Windows, macOS 或 Linux |

为了充分利用这本书，你需要有一个 C# 集成开发环境。本书使用 Visual Studio 2022 社区版，并在 *第一章* 的开头介绍了替代方案。

**如果您正在使用本书的数字版，我们建议您亲自输入代码或从本书的 GitHub 仓库（下一节中有一个链接）获取代码。这样做将帮助您避免与代码复制粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件[`github.com/PacktPublishing/Pragmatic-Test-Driven-Development-in-C-Sharp-and-.NET`](https://github.com/PacktPublishing/Pragmatic-Test-Driven-Development-in-C-Sharp-and-.NET)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有来自我们丰富的书籍和视频目录中的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图和图表彩色图像的 PDF 文件。您可以从这里下载：[`packt.link/OzRlM`](https://packt.link/OzRlM)。

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“前面的代码违反了此规则，因为先运行`UnitTest2`再运行`UnitTest1`将导致测试失败。”

代码块设置如下：

```cs
public class SampleTests 
{
    private static int _staticField = 0;
    [Fact]
    public void UnitTest1()
    {
        _staticField += 1;
        Assert.Equal(1, _staticField);
    }
    [Fact]
    public void UnitTest2()
    {
        _staticField += 5;
        Assert.Equal(6, _staticField);
    }
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```cs
public class SampleTests 
{
    private static int _staticField = 0;
    [Fact]
    public void UnitTest1()
    {
        _staticField += 1;
        Assert.Equal(1, _staticField);
    }
    [Fact]
    public void UnitTest2()
    {
        _staticField += 5;
        Assert.Equal(6, _staticField);
    }
}
```

任何命令行输入或输出都按以下方式编写：

```cs
GET https://webapidomain/services
```

**粗体**：表示新术语、重要词汇或您在屏幕上看到的词汇。例如，菜单或对话框中的词汇以**粗体**显示。以下是一个示例：“安装本地模拟器后，您需要获取连接字符串，您可以通过浏览到[`localhost:8081/_explorer/index.xhtml`](https://localhost:8081/_explorer/index.xhtml)并从**主连接字符串**字段复制连接字符串来完成此操作。”

小贴士或重要注意事项

看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请通过 customercare@packtpub.com 给我们发邮件，并在邮件主题中提及书名。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告此错误。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，我们将不胜感激，如果您能提供位置地址或网站名称。请通过 copyright@packt.com 与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个主题上具有专业知识，并且您有兴趣撰写或为本书做出贡献，请访问[authors.packtpub.com](https://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了《Pragmatic Test-Driven Development in C# and .NET》，我们很乐意听到您的想法！请[点击此处直接访问此书的亚马逊评论页面](https://packt.link/r/1803230193)并分享您的反馈。

您的评论对我们和科技社区都至关重要，并将帮助我们确保我们提供高质量的内容。

# 第一部分：入门和 TDD 的基本知识

在这部分，我们将逐步介绍构成测试驱动开发的所有概念——从依赖注入开始，经过测试替身，最后到 TDD 指南和最佳实践。

在本部分结束时，你将具备使用 TDD 参与应用程序开发的必要知识。本部分包括以下章节：

+   *第一章*, *编写你的第一个 TDD 实现*

+   *第二章*, *通过示例理解依赖注入*

+   *第三章*, *单元测试入门*

+   *第四章*, *使用测试替身进行真实单元测试*

+   *第五章*, *测试驱动开发详解*

+   *第六章*, *TDD 的亲身体验指南*
