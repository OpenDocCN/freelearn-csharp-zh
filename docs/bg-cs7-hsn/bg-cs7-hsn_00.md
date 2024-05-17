# 前言

*Beginning C# 7 Hands-On – Advanced Language Features* 假设您已经掌握了 C#语言的基本要素，并且现在准备在工作的 Visual Studio 环境中逐行学习更高级的 C#语言和语法。您将学习如何编写高级的 C#语言主题，包括泛型、lambda 表达式和匿名方法。您将学习使用查询语法构建查询和部署执行聚合函数的查询。您将使用 C# 7 和 SQL Server 2017 执行复杂的连接和存储过程。探索高级文件访问方法，并了解如何序列化和反序列化对象——所有这些都是通过编写可以在 Visual Studio 中运行的工作代码来完成的。您还将通过 Web 表单查看 C#的 Web 编程。完成本书后，您将了解 C#语言的所有关键高级要素，以及如何从 C#泛型到 XML、LINQ 和您的第一个完整的 MVC Web 应用程序进行编程。这些都是您可以逐行组合利用 C#编程语言全部功能的高级构建块。本书适用于已经掌握基础知识的初学者 C#开发人员，以及需要快速参考实际编码示例中使用高级 C#语言功能的任何人。

# 本书需要什么

建议安装和运行在 Windows 7 或以上的 Visual Studio 2017，并推荐使用 2GB 或 4GB 的 RAM。典型安装至少需要 20-50GB 的硬盘空间。

# 这本书是为谁准备的

这本书将吸引任何对学习如何在 C#中编程感兴趣的人。先前的编程经验将帮助您轻松地通过初始部分，尽管不一定需要具备任何经验。

# 约定

在本书中，您将找到一些区分不同类型信息的文本样式。以下是这些样式的一些示例及其含义解释。文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下："具体来说，`Default.aspx` 是一个包含网页上元素标记的文件。"

代码块设置如下：

```cs
<asp:DropDownList ID="DropDownList1" runat="server" AutoPostBack="True">
    <asp:ListItem>Monday</asp:ListItem>
    <asp:ListItem>Tuesday</asp:ListItem>
    <asp:ListItem>Wednesday</asp:ListItem>
</asp:DropDownList>
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```cs
<asp:DropDownList ID="DropDownList1" runat="server" AutoPostBack="True">
    <asp:ListItem>Monday</asp:ListItem>
    <asp:ListItem>Tuesday</asp:ListItem>
    <asp:ListItem>Wednesday</asp:ListItem>
</asp:DropDownList>
```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词，例如菜单或对话框中的单词，会以这样的形式出现在文本中："如果愿意，点击 浏览 并将文件保存到您选择的位置，然后点击 确定。"

警告或重要说明会出现在这样的形式。

提示和技巧会以这样的形式出现。

# 读者反馈

我们非常欢迎读者的反馈。请告诉我们您对本书的看法——您喜欢或不喜欢的地方。读者的反馈对我们很重要，因为它可以帮助我们开发出您真正能够充分利用的标题。要发送一般反馈，只需发送电子邮件至`feedback@packtpub.com`，并在主题中提及书名。如果您在某个专题上有专业知识，并且有兴趣编写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的自豪所有者，我们有一些事情可以帮助您充分利用您的购买。

# 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。您可以按照以下步骤下载代码文件：

1.  登录或注册到我们的网站，使用您的电子邮件地址和密码。

1.  将鼠标指针悬停在顶部的支持选项卡上。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名。

1.  选择您要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的地点。

1.  点击“代码下载”。

文件下载完成后，请确保您使用最新版本的解压软件解压文件夹：

+   Windows 系统可选择 WinRAR / 7-Zip

+   Mac 系统可选择 Zipeg / iZip / UnRarX

+   Linux 系统可选择 7-Zip / PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Beginning-CSharp-7-Hands-On-Advanced-Language-Features`](https://github.com/PacktPublishing/Beginning-CSharp-7-Hands-On-Advanced-Language-Features)。我们还有来自丰富书籍和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。欢迎查阅！

# 下载本书的彩色图片

我们还为您提供了一个 PDF 文件，其中包含本书中使用的截图/图表的彩色图片。彩色图片将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/BeginningCSharp7HandsOnAdvancedLanguageFeatures_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/BeginningCSharp7HandsOnAdvancedLanguageFeatures_ColorImages.pdf)下载此文件。

# 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误是难免的。如果您在我们的书籍中发现错误——可能是文字或代码上的错误——我们将不胜感激地接受您的报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)进行报告，选择您的书籍，点击“勘误提交表格”链接，并输入您的勘误详情。一旦您的勘误经过验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该书籍的勘误部分的现有勘误列表中。要查看以前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将出现在勘误部分下方。

# 盗版

互联网上的版权盗版问题是所有媒体的持续问题。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。请通过`copyright@packtpub.com`与我们联系，并提供涉嫌盗版材料的链接。感谢您帮助我们保护作者和我们提供有价值内容的能力。

# 问题

如果您对本书的任何方面有问题，可以通过`questions@packtpub.com`与我们联系，我们将尽力解决问题。
