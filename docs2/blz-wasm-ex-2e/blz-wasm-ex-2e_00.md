# 前言

Blazor WebAssembly 是一个框架，它使您能够构建使用 C# 而不是 JavaScript 的单页网页应用。它建立在流行的、健壮的 ASP.NET 框架之上。Blazor WebAssembly 不需要插件或附加组件来在浏览器中使用 C#。它只要求浏览器支持 WebAssembly，而所有现代浏览器都支持。

在本书中，您将完成实际项目，这些项目将教会您 Blazor WebAssembly 框架的基础知识。每一章都包含一个独立的项目，并提供详细的逐步说明。每个项目都旨在突出 Blazor WebAssembly 的一些或多个重要概念。

在本书结束时，您将拥有构建简单独立网页应用和具有 SQL Server 后端的主机网页应用的经验。

这是本书的第二版。在本版中，我们增加了有关调试、部署到 Microsoft Azure 以及使用 Microsoft Azure Active Directory 保护应用程序的章节。此外，本书中的所有项目都已更新，以使用 Blazor WebAssembly 框架的最新版本。

# 本书面向对象

本书面向那些厌倦了不断涌现的新 JavaScript 框架，并希望利用他们在 .NET 和 C# 方面的经验来构建可在任何地方运行的网页应用的经验丰富的网页开发者。

本书面向任何希望通过强调实践而非理论来快速学习 Blazor WebAssembly 的人。它使用完整、逐步的示例项目，这些项目易于跟随，以教授您使用 Blazor WebAssembly 框架开发网页应用所需的概念。

您不需要成为专业开发者就能从本书的项目中受益，但您需要一些 C# 和 HTML 的经验。

# 本书涵盖内容

*第一章*，*Blazor WebAssembly 简介*，介绍了 Blazor WebAssembly 框架。它解释了使用 Blazor 框架的好处，并描述了三种托管模型之间的区别：Blazor Server、Blazor Hybrid 和 Blazor WebAssembly。在强调使用 Blazor WebAssembly 框架的优势后，讨论了 WebAssembly 的目标和支持选项。最后，它指导您完成设置计算机以完成本书中项目的步骤。在本章结束时，您将能够继续阅读本书的任何其他章节。

*第二章*，*构建您的第一个 Blazor WebAssembly 应用程序*，通过创建一个简单的项目介绍了 Razor 组件。本章分为两个部分。第一部分解释了 Razor 组件、路由、Razor 语法以及在开发应用程序时如何使用`热重载`。第二部分逐步引导您使用 Microsoft 提供的 Blazor WebAssembly App 项目模板创建您的第一个 Blazor WebAssembly 应用程序。到本章结束时，您将能够创建一个演示 Blazor WebAssembly 项目。

*第三章*，*调试和部署 Blazor WebAssembly 应用程序*，通过创建一个简单的项目向您展示了如何调试和部署 Blazor WebAssembly 应用程序。本章分为两个部分。第一部分解释了调试、日志记录、异常处理、使用**即时编译**（**AOT**）以及将应用程序部署到 Microsoft Azure。第二部分逐步引导您调试和部署 Blazor WebAssembly 应用程序。到本章结束时，您将能够调试一个简单的 Blazor WebAssembly 应用程序并将其部署到 Microsoft Azure。

*第四章*，*使用模板组件构建模态对话框*，通过创建模态对话框组件向您介绍了模板组件。本章分为两个部分。第一部分解释了`RenderFragment`参数、`EventCallback`参数和 CSS 隔离。第二部分逐步引导您创建模态对话框组件并将其移动到您自己的自定义 Razor 类库中。到本章结束时，您将能够创建一个模态对话框组件并通过 Razor 类库与多个项目共享。

*第五章*，*使用 JavaScript 互操作性（JS Interop）构建本地存储服务*，通过创建本地存储服务向您展示了如何在 Blazor WebAssembly 中使用 JavaScript。本章分为两个部分。第一部分解释了为什么您偶尔还需要使用 JavaScript 以及如何从.NET 中调用 JavaScript 函数。为了完整性，它还涵盖了如何从 JavaScript 中调用.NET 方法。最后，它介绍了项目中使用的 Web Storage API。在第二部分中，它逐步引导您创建和测试一个写入和读取浏览器本地存储的服务。到本章结束时，您将能够通过使用 JS Interop 从 Blazor WebAssembly 应用程序中调用 JavaScript 函数来创建本地存储服务。

*第六章*，*构建作为渐进式 Web 应用程序（PWA）的天气应用程序*，通过创建一个简单的天气 Web 应用程序，为您介绍了渐进式 Web 应用程序。本章分为两个部分。第一部分解释了什么是 PWA 以及如何创建它。它涵盖了清单文件和各种类型的服务工作者。还描述了如何使用项目所需的 CacheStorage API、地理位置 API 和 OpenWeather One Call API。第二部分逐步引导您创建一个 5 天的天气预报应用程序，并将其转换为 PWA，通过添加标志、清单文件和服务工作者。最后，它展示了如何安装和卸载 PWA。到本章结束时，您将能够通过添加标志、清单文件和服务工作者将 Blazor WebAssembly 应用程序转换为 PWA。

*第七章*，*使用应用程序状态构建购物车*，通过创建购物车 Web 应用程序，解释了如何使用应用程序状态。本章分为两个部分。第一部分解释了应用程序状态和依赖注入。第二部分逐步引导您创建购物车应用程序。为了在您的应用程序中维护状态，您将创建一个服务，并将其注册到 DI 容器中，然后注入到您的组件中。到本章结束时，您将能够使用依赖注入在 Blazor WebAssembly 应用程序内维护应用程序状态。

*第八章*，*使用事件构建看板板*，通过创建看板板 Web 应用程序，为您介绍了事件处理。本章分为两个部分。第一部分讨论了事件处理、属性展开和任意参数。第二部分逐步引导您创建一个使用`DragEventArgs`类来启用您在投放区域之间拖放任务的看板板应用程序。到本章结束时，您将能够处理您的 Blazor WebAssembly 应用程序中的事件，并且会舒适地使用属性展开和任意参数。

*第九章*，*上传和读取 Excel 文件*，解释了如何上传各种类型的文件以及如何使用**Open XML SDK**读取 Excel 文件。本章分为两个部分。第一部分解释了上传一个或多个文件和调整图像大小。它还解释了如何使用虚拟化来渲染数据以及如何从 Excel 文件中读取数据。第二部分逐步引导您创建一个可以上传和读取 Excel 文件并使用虚拟化渲染 Excel 文件数据的应用程序。到本章结束时，您将能够将文件上传到 Blazor WebAssembly 应用程序中，使用 Open XML SDK 读取 Excel 文件，并使用虚拟化渲染数据。

*第十章*，*使用 Azure Active Directory 保护 Blazor WebAssembly 应用程序*，通过创建一个简单的应用程序来显示声明的内容，教您如何通过创建一个简单的应用程序来保护 Blazor WebAssembly 应用程序。本章分为两个部分。第一部分解释了身份验证和授权之间的区别。它还教您如何处理身份验证以及如何使用`Authorize`属性和`AuthorizeView`组件。第二部分逐步引导您将应用程序添加到 Azure AD，并使用它进行身份验证和授权。到本章结束时，您将能够使用 Azure AD 保护 Blazor WebAssembly 应用程序。

*第十一章*，*使用 ASP.NET Web API 构建任务管理器*，通过创建任务管理器 Web 应用程序，为您介绍了托管 Blazor WebAssembly 应用程序。这是第一章节使用 SQL Server。它分为两个部分。第一部分描述了托管 Blazor WebAssembly 应用程序的组件。它还解释了如何使用`HttpClient`服务和各种 JSON 辅助方法来操作数据。最后一部分逐步引导您创建一个将数据存储在 SQL Server 数据库中的任务管理器应用程序。您将使用 Entity Framework 创建一个 API 控制器和操作。到本章结束时，您将能够创建一个使用 ASP.NET Web API 更新 SQL Server 数据库数据的托管 Blazor WebAssembly 应用程序。

*第十二章*，*使用 EditForm 组件构建支出跟踪器*，通过创建支出跟踪器 Web 应用程序，教您如何使用`EditForm`组件。本章使用 SQL Server。它分为两个部分。第一部分介绍了`EditForm`组件、内置输入组件和内置验证组件。它还解释了如何使用导航锁定来防止用户在保存编辑之前导航到另一个页面。最后一部分逐步引导您创建一个使用`EditForm`组件和一些内置组件来添加和编辑存储在 SQL Server 数据库中的支出的支出跟踪器应用程序。到本章结束时，您将能够使用`EditForm`组件与内置组件一起输入和验证存储在 SQL Server 数据库中的数据。

# 为了充分利用本书

我们建议您阅读本书的前两章，以了解如何设置您的计算机以及如何使用 Blazor WebAssembly 项目模板。之后，您可以按任何顺序完成剩余的章节。随着您在书中的进展，每个章节的项目变得越来越复杂。最后两章需要 SQL Server 数据库来完成项目。*第三章*和*第十章*需要 Microsoft Azure 订阅。

您将需要本书中涵盖的以下软件/硬件：

+   Visual Studio 2022 Community Edition

+   SQL Server 2022 Express Edition

+   Microsoft Azure

*如果您使用的是本书的数字版，我们建议您自己输入代码或通过 GitHub 仓库（下一节中提供链接）访问代码。这样做将帮助您避免与代码复制和粘贴相关的任何潜在错误。*

本书假设您是一位经验丰富的 Web 开发者。您应该有一些 C#和 HTML 的经验。

有些项目使用 JavaScript 和 CSS，但所有代码都提供。此外，还有两个项目使用 Entity Framework，但同样，所有代码都提供。

## 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件：[`github.com/PacktPublishing/Blazor-WebAssembly-by-Example-Second-Edition`](https://github.com/PacktPublishing/Blazor-WebAssembly-by-Example-Second-Edition)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还提供其他丰富的书籍和视频的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

## 代码实战

本书的相关代码实战视频可以在([`packt.link/CodeinAction`](https://packt.link/CodeinAction))查看。

## 下载彩色图像

我们还提供包含本书中使用的截图/图表的彩色 PDF 文件。您可以从这里下载：[`packt.link/Q27px`](https://packt.link/Q27px)。

## 使用的约定

本书使用了几个文本约定。

`CodeInText`: 表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。例如：“将`DeleteProduct`方法添加到`@code`块中。”

代码块设置如下：

```cs
private void DeleteProduct(Product product) 
{ 
    cart.Remove(product); 
    total -= product.Price; 
} 
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```cs
public class CartService : ICartService 
{ 
    **public** **IList<Product> Cart {** **get****;** **private****set****; }**
    **public****int** **Total {** **get****;** **set****; }**
    public event Action OnChange; 
} 
```

任何命令行输入或输出都按以下方式编写：

```cs
Add-Migration Init 
Update-Database 
```

**粗体**: 表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。例如：“从**构建**菜单中选择**构建解决方案**选项。”

警告或重要提示看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并给我们发送电子邮件至 [customercare@packtpub.com](http://customercare@packtpub.com)。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告，我们将非常感谢。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过[版权@packt.com](http://copyright@packt.com)与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了《Blazor WebAssembly By Example，第二版》，我们很乐意听到您的想法！请[点击此处直接访问此书的亚马逊评论页面](https://packt.link/r/1803241853)并分享您的反馈。

您的评论对我们和科技社区都非常重要，它将帮助我们确保我们提供高质量的内容。

# 下载此书的免费 PDF 副本

感谢您购买此书！

您喜欢在移动中阅读，但无法携带您的印刷书籍到任何地方吗？您的电子书购买是否与您选择的设备不兼容？

不要担心，现在，随着每本 Packt 书籍，您都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何设备上阅读。直接从您最喜欢的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

优惠不会就此停止，您还可以获得独家折扣、时事通讯和每日免费内容的每日电子邮件。

按照以下简单步骤获取好处：

1.  扫描二维码或访问以下链接

*![二维码描述自动生成](img/B18471_QR_Free_PDF.png)*

[`packt.link/free-ebook/9781803241852`](https://packt.link/free-ebook/9781803241852)

1.  提交您的购买证明

1.  就这些！我们将直接将您的免费 PDF 和其他好处发送到您的电子邮件。
