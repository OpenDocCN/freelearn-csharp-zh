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
