# 第二章：说 C#

本章全是关于 C#编程语言基础的。在这一章中，你将学习如何使用 C#的语法编写语句，并介绍一些你每天都会用到的常用词汇。此外，到本章结束时，你将自信地知道如何暂时存储和处理计算机内存中的信息。

本章涵盖以下主题：

+   介绍 C#语言

+   理解 C#语法和词汇

+   使用变量

+   深入探讨控制台应用程序

# 介绍 C#语言

本书的这一部分是关于 C#语言的——你每天用来编写应用程序源代码的语法和词汇。

编程语言与人类语言有许多相似之处，不同之处在于编程语言中，你可以创造自己的词汇，就像苏斯博士那样！

在 1950 年苏斯博士所著的《如果我经营动物园》一书中，他这样说道：

> "然后，只是为了展示给他们看，我将航行到卡特鲁，并带回一个伊特卡奇、一个普里普、一个普鲁，一个内克尔、一个书呆子和一个条纹薄棉布！"

## 理解语言版本和特性

本书的这一部分涵盖了 C#编程语言，主要面向初学者，因此涵盖了所有开发者需要了解的基础主题，从声明变量到存储数据，再到如何定义自己的自定义数据类型。

本书涵盖了 C#语言从 1.0 版本到最新 10.0 版本的所有特性。

如果你已经对旧版本的 C#有所了解，并且对最新版本中的新特性感到兴奋，我通过列出语言版本及其重要的新特性，以及学习它们的章节号和主题标题，使你更容易跳转。

### C# 1.0

C# 1.0 于 2002 年发布，包含了静态类型、面向对象现代语言的所有重要特性，正如您将在*第二章*至*第六章*中所见。

### C# 2.0

C# 2.0 于 2005 年发布，重点是使用泛型实现强类型化，以提高代码性能并减少类型错误，包括以下表格中列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 可空值类型 | 6 | 使值类型可空 |
| 泛型 | 6 | 通过泛型使类型更可重用 |

### C# 3.0

C# 3.0 于 2007 年发布，重点是启用声明式编码，包括**语言集成查询**（**LINQ**）及相关特性，如匿名类型和 lambda 表达式，包括以下表格中列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 隐式类型局部变量 | 2 | 推断局部变量的类型 |
| LINQ | 11 | *第十一章*，*使用 LINQ 查询和操作数据*中的所有主题 |

### C# 4.0

C# 4.0 于 2010 年发布，专注于提高与 F#和 Python 等动态语言的互操作性，包括下表所列主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 动态类型 | 2 | 存储动态类型 |
| 命名/可选参数 | 5 | 可选参数和命名参数 |

### C# 5.0

C# 5.0 于 2012 年发布，专注于通过自动实现复杂的状态机来简化异步操作支持，同时编写看似同步的语句，包括下表所列主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 简化的异步任务 | 12 | 理解异步和等待 |

### C# 6.0

C# 6.0 于 2015 年发布，专注于对语言进行小幅优化，包括下表所列主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| `static` 导入 | 2 | 简化控制台使用 |
| 内插字符串 | 2 | 向用户显示输出 |
| 表达式主体成员 | 5 | 定义只读属性 |

### C# 7.0

C# 7.0 于 2017 年 3 月发布，专注于添加元组和模式匹配等函数式语言特性，以及对语言进行小幅优化，包括下表所列主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 二进制字面量和数字分隔符 | 2 | 存储整数 |
| 模式匹配 | 3 | 使用`if`语句进行模式匹配 |
| `out` 变量 | 5 | 控制参数传递方式 |
| 元组 | 5 | 使用元组组合多个值 |
| 局部函数 | 6 | 定义局部函数 |

### C# 7.1

C# 7.1 于 2017 年 8 月发布，专注于对语言进行小幅优化，包括下表所列主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 默认字面量表达式 | 5 | 使用默认字面量设置字段 |
| 推断元组元素名称 | 5 | 推断元组名称 |
| `async` 主方法 | 12 | 提高控制台应用的响应性 |

### C# 7.2

C# 7.2 于 2017 年 11 月发布，专注于对语言进行小幅优化，包括下表所列主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 数值字面量中的前导下划线 | 2 | 存储整数 |
| 非尾随命名参数 | 5 | 可选参数和命名参数 |
| `private protected` 访问修饰符 | 5 | 理解访问修饰符 |
| 可对元组类型进行`==`和`!=`测试 | 5 | 元组比较 |

### C# 7.3

C# 7.3 于 2018 年 5 月发布，专注于提高`ref`变量、指针和`stackalloc`的性能导向安全代码。这些特性对于大多数开发者来说较为高级且不常用，因此本书未涉及。

### C# 8

C# 8 于 2019 年 9 月发布，专注于与空处理相关的主要语言变更，包括下表所列主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 可空引用类型 | 6 | 使引用类型可空 |
| 开关表达式 | 3 | 使用开关表达式简化`switch`语句 |
| 默认接口方法 | 6 | 理解默认接口方法 |

### C# 9

C# 9 于 2020 年 11 月发布，专注于记录类型、模式匹配的改进以及最小代码控制台应用，包括下表列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 最小代码控制台应用 | 1 | 顶层程序 |
| 目标类型的新 | 2 | 使用目标类型的新实例化对象 |
| 增强的模式匹配 | 5 | 对象的模式匹配 |
| 记录 | 5 | 使用记录 |

### C# 10

C# 10 于 2021 年 11 月发布，专注于减少常见场景中所需代码量的特性，包括下表列出的主题：

| 特性 | 章节 | 主题 |
| --- | --- | --- |
| 全局命名空间导入 | 2 | 导入命名空间 |
| 常量字符串字面量 | 2 | 使用插值字符串格式化 |
| 文件作用域命名空间 | 5 | 简化命名空间声明 |
| 必需属性 | 5 | 实例化时要求属性设置 |
| 记录结构 | 6 | 使用记录结构类型 |
| 空参数检查 | 6 | 方法参数中的空值检查 |

## 理解 C#标准

多年来，微软已向标准机构提交了几个版本的 C#，如下表所示：

| C#版本 | ECMA 标准 | ISO/IEC 标准 |
| --- | --- | --- |
| 1.0 | ECMA-334:2003 | ISO/IEC 23270:2003 |
| 2.0 | ECMA-334:2006 | ISO/IEC 23270:2006 |
| 5.0 | ECMA-334:2017 | ISO/IEC 23270:2018 |

C# 6 的标准仍处于草案阶段，而添加 C# 7 特性的工作正在推进中。微软于 2014 年将 C#开源。

目前，为了尽可能开放地进行 C#及相关技术的工作，有三个公开的 GitHub 仓库，如下表所示：

| 描述 | 链接 |
| --- | --- |
| C#语言设计 | [`github.com/dotnet/csharplang`](https://github.com/dotnet/csharplang) |
| 编译器实现 | [`github.com/dotnet/roslyn`](https://github.com/dotnet/roslyn) |
| 描述语言的标准 | [`github.com/dotnet/csharpstandard`](https://github.com/dotnet/csharpstandard) |

## 发现您的 C#编译器版本

.NET 语言编译器，包括 C#和 Visual Basic（也称为 Roslyn），以及一个独立的 F#编译器，作为.NET SDK 的一部分分发。要使用特定版本的 C#，您必须安装至少该版本的.NET SDK，如下表所示：

| .NET SDK | Roslyn 编译器 | 默认 C#语言 |
| --- | --- | --- |
| 1.0.4 | 2.0 - 2.2 | 7.0 |
| 1.1.4 | 2.3 - 2.4 | 7.1 |
| 2.1.2 | 2.6 - 2.7 | 7.2 |
| 2.1.200 | 2.8 - 2.10 | 7.3 |
| 3.0 | 3.0 - 3.4 | 8.0 |
| 5.0 | 3.8 | 9.0 |
| 6.0 | 3.9 - 3.10 | 10.0 |

当您创建类库时，可以选择面向.NET Standard 以及现代.NET 的版本。它们有默认的 C#语言版本，如下表所示：

| .NET Standard | C# |
| --- | --- |
| 2.0 | 7.3 |
| 2.1 | 8.0 |

### 如何输出 SDK 版本

让我们看看您可用的.NET SDK 和 C#语言编译器版本：

1.  在 macOS 上，启动 **终端**。在 Windows 上，启动 **命令提示符**。

1.  要确定您可用的.NET SDK 版本，请输入以下命令：

    ```cs
    dotnet --version 
    ```

1.  注意，撰写本文时的版本是 6.0.100，表明这是 SDK 的初始版本，尚未有任何错误修复或新功能，如下输出所示：

    ```cs
    6.0.100 
    ```

### 启用特定语言版本编译器

像 Visual Studio 和 `dotnet` 命令行接口这样的开发工具默认假设您想使用 C#语言编译器的最新主版本。在 C# 8.0 发布之前，C# 7.0 是默认使用的最新主版本。要使用 C#点版本（如 7.1、7.2 或 7.3）的改进，您必须在项目文件中添加 `<LangVersion>` 配置元素，如下所示：

```cs
<LangVersion>7.3</LangVersion> 
```

C# 10.0 随.NET 6.0 发布后，如果微软发布了 C# 10.1 编译器，并且您想使用其新语言特性，则必须在项目文件中添加配置元素，如下所示：

```cs
<LangVersion>10.1</LangVersion> 
```

以下表格展示了 `<LangVersion>` 的可能值：

| LangVersion | 描述 |
| --- | --- |
| 7, 7.1, 7.2, 7.38, 9, 10 | 输入特定版本号将使用已安装的该编译器。 |
| latestmajor | 使用最高的主版本号，例如，2019 年 8 月的 7.0，2019 年 10 月的 8.0，2020 年 11 月的 9.0，2021 年 11 月的 10.0。 |
| `latest` | 使用最高的主版本和次版本号，例如，2017 年的 7.2，2018 年的 7.3，2019 年的 8，以及 2022 年初可能的 10.1。 |
| `preview` | 使用可用的最高预览版本，例如，2021 年 7 月安装了.NET 6.0 Preview 6 的 10.0。 |

创建新项目后，您可以编辑 `.csproj` 文件并添加 `<LangVersion>` 元素，如下所示高亮显示：

```cs
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
 **<LangVersion>preview</LangVersion>**
  </PropertyGroup>
</Project> 
```

您的项目必须针对 `net6.0` 以使用 C# 10 的全部功能。

**良好实践**：如果您正在使用 Visual Studio Code 且尚未安装，请安装名为 **MSBuild 项目工具** 的 Visual Studio Code 扩展。这将为您在编辑 `.csproj` 文件时提供 IntelliSense，包括轻松添加带有适当值的 `<LangVersion>` 元素。

# 理解 C#语法和词汇

要学习简单的 C#语言特性，您可以使用.NET 交互式笔记本，这消除了创建任何类型应用程序的需要。

要学习其他一些 C#语言特性，您需要创建一个应用程序。最简单的应用程序类型是控制台应用程序。

让我们从 C#语法和词汇的基础开始。在本章中，您将创建多个控制台应用程序，每个都展示 C#语言的相关特性。

## 显示编译器版本

我们将首先编写显示编译器版本的代码：

1.  如果你已经完成了*第一章*，*你好，C#！欢迎，.NET！*，那么你将已经拥有一个`Code`文件夹。如果没有，那么你需要创建它。

1.  使用你喜欢的代码编辑器创建一个新的控制台应用，如下表所示：

    1.  项目模板：**控制台应用程序[C#]** / `console`

    1.  工作区/解决方案文件和文件夹：`Chapter02`

    1.  项目文件和文件夹：`Vocabulary`

        **最佳实践**：如果你忘记了如何操作，或者没有完成前一章，那么*第一章*，*你好，C#！欢迎，.NET！*中给出了创建包含多个项目的工作区/解决方案的分步说明。

1.  打开`Program.cs`文件，在注释下方，添加一个语句以显示 C#版本作为错误，如下面的代码所示：

    ```cs
    #error version 
    ```

1.  运行控制台应用程序：

    1.  在 Visual Studio Code 中，在终端输入命令`dotnet run`。

    1.  在 Visual Studio 中，导航到**调试** | **开始不调试**。当提示继续并运行上次成功的构建时，点击**否**。

1.  注意编译器版本和语言版本显示为编译器错误消息编号`CS8304`，如图 2.1 所示：![](img/B17442_02_01.png)

    图 2.1：显示 C#语言版本的编译器错误

1.  在 Visual Studio Code 的**问题**窗口或 Visual Studio 的**错误列表**窗口中的错误消息显示`编译器版本：'4.0.0...'`，语言版本为`10.0`。

1.  注释掉导致错误的语句，如下面的代码所示：

    ```cs
    // #error version 
    ```

1.  注意编译器错误消息消失了。

## 理解 C#语法

C#的语法包括语句和块。要记录你的代码，你可以使用注释。

**最佳实践**：注释不应该是你唯一用来记录代码的方式。为变量和函数选择合理的名称、编写单元测试以及创建实际文档是其他记录代码的方式。

## 语句

在英语中，我们用句号表示句子的结束。一个句子可以由多个单词和短语组成，单词的顺序是语法的一部分。例如，在英语中，我们说“the black cat”。

形容词*black*位于名词*cat*之前。而法语语法则不同；形容词位于名词之后：“le chat noir”。重要的是要记住，顺序很重要。

C#使用分号表示**语句**的结束。一个语句可以由多个**变量**和**表达式**组成。例如，在以下语句中，`totalPrice`是一个变量，`subtotal + salesTax`是一个表达式：

```cs
var totalPrice = subtotal + salesTax; 
```

该表达式由一个名为`subtotal`的操作数、一个运算符`+`和另一个名为`salesTax`的操作数组成。操作数和运算符的顺序很重要。

## 注释

编写代码时，你可以使用双斜杠`//`添加注释来解释你的代码。通过插入`//`，编译器将忽略`//`之后的所有内容，直到该行结束，如下面的代码所示：

```cs
// sales tax must be added to the subtotal
var totalPrice = subtotal + salesTax; 
```

要编写多行注释，请在注释的开头使用`/*`，在结尾使用`*/`，如下列代码所示：

```cs
/*
This is a multi-line comment.
*/ 
```

**最佳实践**：设计良好的代码，包括具有良好命名参数的函数签名和类封装，可以一定程度上自我说明。当你发现自己需要在代码中添加过多注释和解释时，问问自己：我能否通过重写（即重构）这段代码，使其在不依赖长篇注释的情况下更易于理解？

您的代码编辑器具有命令，使得添加和删除注释字符更加容易，如下表所示：

+   **Windows 版 Visual Studio 2022**：导航至**编辑** | **高级** | **注释选择** 或 **取消注释选择**

+   **Visual Studio Code**：导航至**编辑** | **切换行注释** 或 **切换块注释**

    **最佳实践**：您通过在代码语句上方或后方添加描述性文本来**注释**代码。您通过在语句前或周围添加注释字符来**注释掉**代码，使其失效。**取消注释**意味着移除注释字符。

## 块

在英语中，我们通过换行来表示新段落的开始。C# 使用花括号`{ }`来表示**代码块**。

块以声明开始，用以指示定义的内容。例如，一个块可以定义包括命名空间、类、方法或`foreach`等语句在内的多种语言结构的开始和结束。

您将在本章及后续章节中了解更多关于命名空间、类和方法的知识，但现在简要介绍一些概念：

+   **命名空间**包含类等类型，用于将它们组合在一起。

+   **类**包含对象的成员，包括方法。

+   **方法**包含实现对象可执行动作的语句。

## 语句和块的示例

在面向 .NET 5.0 的控制台应用程序项目模板中，请注意，项目模板已为您编写了 C# 语法的示例。我已对语句和块添加了一些注释，如下列代码所示：

```cs
using System; // a semicolon indicates the end of a statement
namespace Basics
{ // an open brace indicates the start of a block
  class Program
  {
    static void Main(string[] args)
    {
      Console.WriteLine("Hello World!"); // a statement
    }
  }
} // a close brace indicates the end of a block 
```

## 理解 C# 词汇

C# 词汇由**关键字**、**符号字符**和**类型**组成。

本书中您将看到的一些预定义保留关键字包括`using`、`namespace`、`class`、`static`、`int`、`string`、`double`、`bool`、`if`、`switch`、`break`、`while`、`do`、`for`、`foreach`、`and`、`or`、`not`、`record`和`init`。

您将看到的一些符号字符包括`"`, `'`, `+`, `-`, `*`, `/`, `%`, `@`, 和 `$`。

还有其他只在特定上下文中具有特殊意义的上下文关键字。

然而，这仍然意味着 C# 语言中只有大约 100 个实际的关键字。

## 将编程语言与人类语言进行比较

英语有超过 25 万个不同的单词，那么 C#是如何仅用大约 100 个关键字就做到的呢？此外，如果 C#只有英语单词数量的 0.0416%，为什么它还这么难学呢？

人类语言和编程语言之间的一个关键区别是，开发者需要能够定义具有新含义的新“单词”。除了 C#语言中的大约 100 个关键字外，本书还将教你了解其他开发者定义的数十万个“单词”，同时你还将学习如何定义自己的“单词”。

世界各地的程序员都必须学习英语，因为大多数编程语言使用英语单词，如 namespace 和 class。有些编程语言使用其他人类语言，如阿拉伯语，但这些语言较为罕见。如果你对此感兴趣，这个 YouTube 视频展示了一种阿拉伯编程语言的演示：[`youtu.be/dkO8cdwf6v8`](https://youtu.be/dkO8cdwf6v8)。

## 更改 C#语法的颜色方案

默认情况下，Visual Studio Code 和 Visual Studio 将 C#关键字显示为蓝色，以便更容易与其他代码区分开来。这两个工具都允许您自定义颜色方案：

1.  在 Visual Studio Code 中，导航至**代码** | **首选项** | **颜色主题**（在 Windows 上的**文件**菜单中）。

1.  选择一个颜色主题。作为参考，我将使用**Light+（默认浅色）**颜色主题，以便截图在印刷书籍中看起来效果良好。

1.  在 Visual Studio 中，导航至**工具** | **选项**。

1.  在**选项**对话框中，选择**字体和颜色**，然后选择您希望自定义的显示项。

## 帮助编写正确的代码

像记事本这样的纯文本编辑器不会帮助你书写正确的英语。同样，记事本也不会帮助你编写正确的 C#代码。

Microsoft Word 可以通过用红色波浪线标记拼写错误来帮助你书写英语，例如 Word 会提示"icecream"应为 ice-cream 或 ice cream，并用蓝色波浪线标记语法错误，例如句子应以大写字母开头。

同样，Visual Studio Code 的 C#扩展程序和 Visual Studio 通过标记拼写错误（例如方法名应为`WriteLine`，其中 L 为大写）和语法错误（例如语句必须以分号结尾）来帮助你编写 C#代码。

C#扩展程序会不断监视你输入的内容，并通过用彩色波浪线标记问题来给予你反馈，类似于 Microsoft Word。

让我们看看它的实际应用：

1.  在`Program.cs`中，将`WriteLine`方法中的`L`改为小写。

1.  删除语句末尾的分号。

1.  在 Visual Studio Code 中，导航至**视图** | **问题**，或在 Visual Studio 中导航至**视图** | **错误列表**，并注意代码错误下方会出现红色波浪线，并且会显示详细信息，如*图 2.2*所示：![](img/B17442_02_02.png)

    图 2.2：错误列表窗口显示两个编译错误

1.  修复这两个编码错误。

## 导入命名空间

`System`是一个命名空间，类似于类型的地址。为了精确地引用某人的位置，您可能会使用`Oxford.HighStreet.BobSmith`，这告诉我们需要在牛津市的高街上寻找一个名叫 Bob Smith 的人。

`System.Console.WriteLine`指示编译器在一个名为`Console`的类型中查找名为`WriteLine`的方法，该类型位于名为`System`的命名空间中。为了简化我们的代码，在.NET 6.0 之前的每个版本的**控制台应用程序**项目模板中，都会在代码文件顶部添加一个语句，告诉编译器始终在`System`命名空间中查找未带命名空间前缀的类型，如下列代码所示：

```cs
using System; // import the System namespace 
```

我们称之为*导入命名空间*。导入命名空间的效果是，该命名空间中的所有可用类型将无需输入命名空间前缀即可供您的程序使用，并在编写代码时在 IntelliSense 中可见。

.NET Interactive 笔记本会自动导入大多数命名空间。

### 隐式和全局导入命名空间

传统上，每个需要导入命名空间的`.cs`文件都必须以`using`语句开始导入那些命名空间。几乎所有`.cs`文件都需要命名空间，如`System`和`System.Linq`，因此每个`.cs`文件的前几行通常至少包含几个`using`语句，如下列代码所示：

```cs
using System;
using System.Linq;
using System.Collections.Generic; 
```

在使用 ASP.NET Core 创建网站和服务时，每个文件通常需要导入数十个命名空间。

C# 10 引入了一些简化导入命名空间的新特性。

首先，`global using`语句意味着您只需在一个`.cs`文件中导入一个命名空间，它将在所有`.cs`文件中可用。您可以将`global using`语句放在`Program.cs`文件中，但我建议为这些语句创建一个单独的文件，命名为类似`GlobalUsings.cs`或`GlobalNamespaces.cs`，如下列代码所示：

```cs
global using System;
global using System.Linq;
global using System.Collections.Generic; 
```

**最佳实践**：随着开发者逐渐习惯这一新的 C#特性，我预计该文件的一种命名约定将成为标准。

其次，任何面向.NET 6.0 的项目，因此使用 C# 10 编译器，会在`obj`文件夹中生成一个`.cs`文件，以隐式全局导入一些常见命名空间，如`System`。隐式导入的命名空间列表取决于您所针对的 SDK，如下表所示：

| SDK | 隐式导入的命名空间 |
| --- | --- |
| `Microsoft.NET.Sdk` | `System``System.Collections.Generic``System.IO``System.Linq``System.Net.Http``System.Threading``System.Threading.Tasks` |
| `Microsoft.NET.Sdk.Web` | 与`Microsoft.NET.Sdk`相同，并包括：`System.Net.Http.Json``Microsoft.AspNetCore.Builder``Microsoft.AspNetCore.Hosting``Microsoft.AspNetCore.Http``Microsoft.AspNetCore.Routing``Microsoft.Extensions.Configuration``Microsoft.Extensions.DependencyInjection``Microsoft.Extensions.Hosting``Microsoft.Extensions.Logging` |
| `Microsoft.NET.Sdk.Worker` | 与`Microsoft.NET.Sdk`相同，并包括：`Microsoft.Extensions.Configuration``Microsoft.Extensions.DependencyInjection``Microsoft.Extensions.Hosting``Microsoft.Extensions.Logging` |

让我们看看当前自动生成的隐式导入文件：

1.  在**解决方案资源管理器**中，选择`词汇`项目，打开**显示所有文件**按钮，并注意编译器生成的`bin`和`obj`文件夹可见。

1.  展开`obj`文件夹，展开`Debug`文件夹，展开`net6.0`文件夹，并打开名为`Vocabulary.GlobalUsings.g.cs`的文件。

1.  注意此文件是由编译器为面向.NET 6.0 的项目自动创建的，并且它导入一些常用命名空间，包括`System.Threading`，如下所示的代码：

    ```cs
    // <autogenerated />
    global using global::System;
    global using global::System.Collections.Generic;
    global using global::System.IO;
    global using global::System.Linq;
    global using global::System.Net.Http;
    global using global::System.Threading;
    global using global::System.Threading.Tasks; 
    ```

1.  关闭`Vocabulary.GlobalUsings.g.cs`文件。

1.  在**解决方案资源管理器**中，选择项目，然后向项目文件添加额外的条目以控制哪些命名空间被隐式导入，如下所示高亮显示的标记：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
      </PropertyGroup>
     **<ItemGroup>**
     **<Using Remove=****"System.Threading"** **/>**
     **<Using Include=****"System.Numerics"** **/>**
     **</ItemGroup>**
    </Project> 
    ```

1.  保存对项目文件的更改。

1.  展开`obj`文件夹，展开`Debug`文件夹，展开`net6.0`文件夹，并打开名为`Vocabulary.GlobalUsings.g.cs`的文件。

1.  注意此文件现在导入`System.Numerics`而不是`System.Threading`，如下所示高亮显示的代码：

    ```cs
    // <autogenerated />
    global using global::System;
    global using global::System.Collections.Generic;
    global using global::System.IO;
    global using global::System.Linq;
    global using global::System.Net.Http;
    global using global::System.Threading.Tasks;
    **global****using****global****::System.Numerics;** 
    ```

1.  关闭`Vocabulary.GlobalUsings.g.cs`文件。

您可以通过从项目文件中删除一个条目来禁用所有 SDK 的隐式导入命名空间功能，如下所示的标记：

```cs
<ImplicitUsings>enable</ImplicitUsings> 
```

## 动词是方法

在英语中，动词是表示动作或行为的词，如跑和跳。在 C#中，表示动作或行为的词被称为**方法**。C#中有数十万个方法可供使用。在英语中，动词根据动作发生的时间改变其书写方式。例如，Amir *过去在跳*，Beth *现在跳*，他们*过去跳*，Charlie *将来会跳*。

在 C#中，像`WriteLine`这样的方法会根据具体操作的细节改变其调用或执行方式。这称为重载，我们将在*第五章*，*使用面向对象编程构建自己的类型*中详细介绍。但现在，请考虑以下示例：

```cs
// outputs the current line terminator string
// by default, this is a carriage-return and line feed
Console.WriteLine();
// outputs the greeting and the current line terminator string
Console.WriteLine("Hello Ahmed");
// outputs a formatted number and date and the current line terminator string
Console.WriteLine("Temperature on {0:D} is {1}°C.", 
  DateTime.Today, 23.4); 
```

一个不同的比喻是，有些单词拼写相同，但根据上下文有不同的含义。

## 名词是类型、变量、字段和属性

在英语中，名词是用来指代事物的名称。例如，Fido 是一条狗的名字。单词“狗”告诉我们 Fido 是什么类型的事物，因此为了让 Fido 去取球，我们会使用他的名字。

在 C#中，它们的等价物是**类型**、**变量**、**字段**和**属性**。例如：

+   `Animal`和`Car`是类型；它们是用于分类事物的名词。

+   `Head`和`Engine`可能是字段或属性；属于`Animal`和`Car`的名词。

+   `Fido`和`Bob`是变量；用于指代特定对象的名词。

C#可用的类型有数以万计，尽管你注意到我没有说“C#中有数以万计的类型”吗？这种区别微妙但重要。C#语言只有几个类型的关键字，如`string`和`int`，严格来说，C#并没有定义任何类型。看起来像类型的关键字，如`string`，是**别名**，它们代表 C#运行的平台上提供的类型。

重要的是要知道 C#不能独立存在；毕竟，它是一种运行在.NET 变体上的语言。理论上，有人可以为 C#编写一个使用不同平台的编译器，具有不同的底层类型。实际上，C#的平台是.NET，它为 C#提供了数以万计的类型，包括`System.Int32`，这是 C#关键字别名`int`映射到的，以及许多更复杂的类型，如`System.Xml.Linq.XDocument`。

值得注意的是，术语**类型**经常与**类**混淆。你玩过*二十个问题*这个聚会游戏吗，也称为*动物、植物或矿物*？在游戏中，一切都可以归类为动物、植物或矿物。在 C#中，每个**类型**都可以归类为`class`、`struct`、`enum`、`interface`或`delegate`。您将在*第六章*，*实现接口和继承类*中学习这些含义。例如，C#关键字`string`是一个`class`，但`int`是一个`struct`。因此，最好使用术语**类型**来指代两者。

## 揭示 C#词汇的范围

我们知道 C#中有超过 100 个关键字，但有多少种类型呢？让我们编写一些代码来找出在我们的简单控制台应用程序中 C#可用的类型（及其方法）的数量。

现在不必担心这段代码是如何工作的，但要知道它使用了一种称为**反射**的技术：

1.  我们将首先在`Program.cs`文件顶部导入`System.Reflection`命名空间，如下所示：

    ```cs
    using System.Reflection; 
    ```

1.  删除写入`Hello World!`的语句，并用以下代码替换：

    ```cs
    Assembly? assembly = Assembly.GetEntryAssembly();
    if (assembly == null) return;
    // loop through the assemblies that this app references
    foreach (AssemblyName name in assembly.GetReferencedAssemblies())
    {
      // load the assembly so we can read its details
      Assembly a = Assembly.Load(name);
      // declare a variable to count the number of methods
      int methodCount = 0;
      // loop through all the types in the assembly
      foreach (TypeInfo t in a.DefinedTypes)
      {
        // add up the counts of methods
        methodCount += t.GetMethods().Count();
      }
      // output the count of types and their methods
      Console.WriteLine(
        "{0:N0} types with {1:N0} methods in {2} assembly.",
        arg0: a.DefinedTypes.Count(),
        arg1: methodCount, arg2: name.Name);
    } 
    ```

1.  运行代码。您将看到在您的操作系统上运行最简单的应用程序时，实际可用的类型和方法的数量。显示的类型和方法的数量将根据您使用的操作系统而有所不同，如下所示：

    ```cs
    // Output on Windows
    0 types with 0 methods in System.Runtime assembly.
    106 types with 1,126 methods in System.Linq assembly.
    44 types with 645 methods in System.Console assembly.
    // Output on macOS
    0 types with 0 methods in System.Runtime assembly.
    103 types with 1,094 methods in System.Linq assembly.
    57 types with 701 methods in System.Console assembly. 
    ```

    为什么`System.Runtime`程序集中不包含任何类型？这个程序集很特殊，因为它只包含**类型转发器**而不是实际类型。类型转发器表示已在外部.NET 或其他高级原因中实现的一种类型。

1.  在导入命名空间后，在文件顶部添加语句以声明一些变量，如下面的代码中突出显示的那样：

    ```cs
    using System.Reflection;
    **// declare some unused variables using types**
    **// in additional assemblies**
    **System.Data.DataSet ds;**
    **HttpClient client;** 
    ```

    通过声明使用其他程序集中的类型的变量，这些程序集会随我们的应用程序一起加载，这使得我们的代码能够看到其中的所有类型和方法。编译器会警告你有未使用的变量，但这不会阻止你的代码运行。

1.  再次运行控制台应用程序并查看结果，结果应该类似于以下输出：

    ```cs
    // Output on Windows
    0 types with 0 methods in System.Runtime assembly.
    383 types with 6,854 methods in System.Data.Common assembly.
    456 types with 4,590 methods in System.Net.Http assembly.
    106 types with 1,126 methods in System.Linq assembly.
    44 types with 645 methods in System.Console assembly.
    // Output on macOS
    0 types with 0 methods in System.Runtime assembly.
    376 types with 6,763 methods in System.Data.Common assembly.
    522 types with 5,141 methods in System.Net.Http assembly.
    103 types with 1,094 methods in System.Linq assembly.
    57 types with 701 methods in System.Console assembly. 
    ```

现在，你更清楚为什么学习 C#是一项挑战，因为有如此多的类型和方法需要学习。方法仅是类型可以拥有的成员类别之一，而你和其他程序员不断定义新的类型和成员！

# 处理变量

所有应用程序都处理数据。数据进来，数据被处理，然后数据出去。

数据通常从文件、数据库或用户输入进入我们的程序，并可以暂时存储在变量中，这些变量将存储在运行程序的内存中。当程序结束时，内存中的数据就会丢失。数据通常输出到文件和数据库，或输出到屏幕或打印机。使用变量时，首先应考虑变量在内存中占用多少空间，其次应考虑处理速度有多快。

我们通过选择适当的类型来控制这一点。你可以将`int`和`double`等简单常见类型视为不同大小的存储箱，其中较小的箱子占用较少的内存，但处理速度可能不那么快；例如，在 64 位操作系统上，添加 16 位数字可能不如添加 64 位数字快。其中一些箱子可能堆放在附近，而有些可能被扔进更远的堆中。

## 命名事物和赋值

事物有命名规范，遵循这些规范是良好的实践，如下表所示：

| 命名规范 | 示例 | 用途 |
| --- | --- | --- |
| 驼峰式 | `cost`, `orderDetail`, `dateOfBirth` | 局部变量，私有字段 |
| 标题式（也称为帕斯卡式） | `String`, `Int32`, `Cost`, `DateOfBirth`, `Run` | 类型，非私有字段，以及其他成员如方法 |

**良好实践**：遵循一致的命名规范将使你的代码易于被其他开发者（以及未来的你自己）理解。

下面的代码块展示了一个声明命名局部变量并使用`=`符号为其赋值的示例。你应该注意到，可以使用 C# 6.0 引入的关键字`nameof`输出变量的名称：

```cs
// let the heightInMetres variable become equal to the value 1.88
double heightInMetres = 1.88;
Console.WriteLine($"The variable {nameof(heightInMetres)} has the value
{heightInMetres}."); 
```

前面代码中双引号内的消息因为打印页面的宽度太窄而换到第二行。在你的代码编辑器中输入这样的语句时，请将其全部输入在同一行。

## 字面值

当你给变量赋值时，你通常，但并非总是，赋一个**字面**值。但什么是字面值呢？字面值是一种表示固定值的符号。数据类型有不同的字面值表示法，在接下来的几节中，你将看到使用字面值表示法给变量赋值的示例。

## 存储文本

对于文本，单个字母，如`A`，存储为`char`类型。

**最佳实践**：实际上，这可能比那更复杂。埃及象形文字 A002（U+13001）需要两个`System.Char`值（称为代理对）来表示它：`\uD80C`和`\uDC01`。不要总是假设一个`char`等于一个字母，否则你可能在你的代码中引入奇怪的错误。

使用单引号将字面值括起来，或赋值给一个虚构函数调用的返回值，来给`char`赋值，如下代码所示：

```cs
char letter = 'A'; // assigning literal characters
char digit = '1'; 
char symbol = '$';
char userChoice = GetSomeKeystroke(); // assigning from a fictitious function 
```

对于文本，多个字母，如`Bob`，存储为`string`类型，并使用双引号将字面值括起来，或赋值给函数调用的返回值，如下代码所示：

```cs
string firstName = "Bob"; // assigning literal strings
string lastName = "Smith";
string phoneNumber = "(215) 555-4256";
// assigning a string returned from a fictitious function
string address = GetAddressFromDatabase(id: 563); 
```

### 理解逐字字符串

当在`string`变量中存储文本时，你可以包含转义序列，这些序列使用反斜杠表示特殊字符，如制表符和新行，如下代码所示：

```cs
string fullNameWithTabSeparator = "Bob\tSmith"; 
```

但如果你要存储 Windows 上的文件路径，其中一个文件夹名以`T`开头，如下代码所示，该怎么办？

```cs
string filePath = "C:\televisions\sony\bravia.txt"; 
```

编译器会将`\t`转换为制表符，你将会得到错误！

你必须以前缀`@`符号使用逐字字面`string`，如下代码所示：

```cs
string filePath = @"C:\televisions\sony\bravia.txt"; 
```

总结如下：

+   **字面字符串**：用双引号括起来的字符。它们可以使用转义字符，如`\t`表示制表符。要表示反斜杠，使用两个：`\\`。

+   **逐字字符串**：以`@`为前缀的字面字符串，用于禁用转义字符，使得反斜杠就是反斜杠。它还允许`string`值跨越多行，因为空白字符被视为其本身，而不是编译器的指令。

+   **内插字符串**：以`$`为前缀的字面字符串，用于启用嵌入格式化变量。你将在本章后面了解更多关于这方面的内容。

## 存储数字

数字是我们想要进行算术计算的数据，例如乘法。电话号码不是数字。要决定一个变量是否应存储为数字，请问自己是否需要对该数字执行算术运算，或者该数字是否包含非数字字符，如括号或连字符来格式化数字，例如(414) 555-1234。在这种情况下，该数字是一串字符，因此应将其存储为`string`。

数字可以是自然数，如 42，用于计数（也称为整数）；它们也可以是负数，如-42（称为整数）；或者，它们可以是实数，如 3.9（带有小数部分），在计算中称为单精度或双精度浮点数。

让我们探索数字：

1.  使用您偏好的代码编辑器，在`Chapter02`工作区/解决方案中添加一个名为`Numbers`的**控制台应用程序**。

    1.  在 Visual Studio Code 中，选择`Numbers`作为活动的 OmniSharp 项目。当看到弹出警告消息提示缺少必需资产时，点击**是**以添加它们。

    1.  在 Visual Studio 中，将启动项目设置为当前选择。

1.  在`Program.cs`中，删除现有代码，然后输入语句以声明一些使用各种数据类型的数字变量，如下列代码所示：

    ```cs
    // unsigned integer means positive whole number or 0
    uint naturalNumber = 23;
    // integer means negative or positive whole number or 0
    int integerNumber = -23;
    // float means single-precision floating point
    // F suffix makes it a float literal
    float realNumber = 2.3F;
    // double means double-precision floating point
    double anotherRealNumber = 2.3; // double literal 
    ```

### 存储整数

您可能知道计算机将所有内容存储为位。位的值要么是 0，要么是 1。这称为**二进制数系统**。人类使用**十进制数系统**。

十进制数系统，又称基数 10，其**基数**为 10，意味着有十个数字，从 0 到 9。尽管它是人类文明中最常用的数基，但科学、工程和计算领域中其他数基系统也很流行。二进制数系统，又称基数 2，其基数为 2，意味着有两个数字，0 和 1。

下表展示了计算机如何存储十进制数 10。请注意 8 和 2 列中值为 1 的位；8 + 2 = 10：

| 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 0 | 0 | 0 | 1 | 0 | 1 | 0 |

因此，十进制中的`10`在二进制中是`00001010`。

#### 通过使用数字分隔符提高可读性

C# 7.0 及更高版本中的两项改进是使用下划线字符`_`作为数字分隔符，以及支持二进制字面量。

您可以在数字字面量的任何位置插入下划线，包括十进制、二进制或十六进制表示法，以提高可读性。

例如，您可以将 100 万在十进制表示法（即基数 10）中写为`1_000_000`。

您甚至可以使用印度常见的 2/3 分组：`10_00_000`。

#### 使用二进制表示法

要使用二进制表示法，即基数 2，仅使用 1 和 0，请以`0b`开始数字字面量。要使用十六进制表示法，即基数 16，使用 0 到 9 和 A 到 F，请以`0x`开始数字字面量。

### 探索整数

让我们输入一些代码来查看一些示例：

1.  在`Program.cs`中，输入语句以声明一些使用下划线分隔符的数字变量，如下列代码所示：

    ```cs
    // three variables that store the number 2 million
    int decimalNotation = 2_000_000;
    int binaryNotation = 0b_0001_1110_1000_0100_1000_0000; 
    int hexadecimalNotation = 0x_001E_8480;
    // check the three variables have the same value
    // both statements output true 
    Console.WriteLine($"{decimalNotation == binaryNotation}"); 
    Console.WriteLine(
      $"{decimalNotation == hexadecimalNotation}"); 
    ```

1.  运行代码并注意结果是所有三个数字都相同，如下列输出所示：

    ```cs
    True
    True 
    ```

计算机总能使用`int`类型或其同类类型（如`long`和`short`）精确表示整数。

## 存储实数

计算机不能总是精确地表示实数，即小数或非整数。`float`和`double`类型使用单精度和双精度浮点来存储实数。

大多数编程语言都实现了 IEEE 浮点算术标准。IEEE 754 是由**电气和电子工程师协会**（**IEEE**）于 1985 年制定的浮点算术技术标准。

下表简化了计算机如何用二进制表示数字`12.75`。注意 8、4、½和¼列中值为`1`的位。

8 + 4 + ½ + ¼ = 12¾ = 12.75。

| 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 | . | ½ | ¼ | 1/8 | 1/16 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 0 | 0 | 0 | 1 | 1 | 0 | 0 | . | 1 | 1 | 0 | 0 |

因此，十进制的`12.75`在二进制中是`00001100.1100`。如你所见，数字`12.75`可以用位精确表示。然而，有些数字则不能，我们很快就会探讨这一点。

### 编写代码以探索数字大小

C# 有一个名为`sizeof()`的运算符，它返回一个类型在内存中使用的字节数。某些类型具有名为`MinValue`和`MaxValue`的成员，这些成员返回可以存储在该类型的变量中的最小和最大值。我们现在将使用这些特性来创建一个控制台应用程序以探索数字类型：

1.  在`Program.cs`中，输入语句以显示三种数字数据类型的大小，如下面的代码所示：

    ```cs
    Console.WriteLine($"int uses {sizeof(int)} bytes and can store numbers in the range {int.MinValue:N0} to {int.MaxValue:N0}."); 
    Console.WriteLine($"double uses {sizeof(double)} bytes and can store numbers in the range {double.MinValue:N0} to {double.MaxValue:N0}."); 
    Console.WriteLine($"decimal uses {sizeof(decimal)} bytes and can store numbers in the range {decimal.MinValue:N0} to {decimal.MaxValue:N0}."); 
    ```

    本书中打印页面的宽度使得`string`值（用双引号括起来）跨越多行。你必须在一行内输入它们，否则会遇到编译错误。

1.  运行代码并查看输出，如*图 2.3*所示：![](img/B17442_02_03.png)

    *图 2.3*：常见数字数据类型的大小和范围信息

`int`变量使用四个字节的内存，并可以存储大约 20 亿以内的正负数。`double`变量使用八个字节的内存，可以存储更大的值！`decimal`变量使用 16 个字节的内存，可以存储大数字，但不如`double`类型那么大。

但你可能在想，为什么`double`变量能够存储比`decimal`变量更大的数字，而在内存中只占用一半的空间？那么，现在就让我们来找出答案吧！

### 比较`double`和`decimal`类型

接下来，你将编写一些代码来比较`double`和`decimal`值。尽管不难理解，但不必担心现在就掌握语法：

1.  输入语句以声明两个`double`变量，将它们相加并与预期结果进行比较，然后将结果写入控制台，如下面的代码所示：

    ```cs
    Console.WriteLine("Using doubles:"); 
    double a = 0.1;
    double b = 0.2;
    if (a + b == 0.3)
    {
      Console.WriteLine($"{a} + {b} equals {0.3}");
    }
    else
    {
      Console.WriteLine($"{a} + {b} does NOT equal {0.3}");
    } 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Using doubles:
    0.1 + 0.2 does NOT equal 0.3 
    ```

在使用逗号作为小数分隔符的地区，结果会略有不同，如下面的输出所示：

```cs
0,1 + 0,2 does NOT equal 0,3 
```

`double`类型不能保证精确，因为像`0.1`这样的数字实际上无法用浮点值表示。

一般来说，只有当精确度，尤其是比较两个数的相等性不重要时，才应使用`double`。例如，当你测量一个人的身高，并且只会使用大于或小于进行比较，而永远不会使用等于时，就可能属于这种情况。

前面代码的问题在于计算机如何存储数字`0.1`，或其倍数。为了在二进制中表示`0.1`，计算机在 1/16 列存储 1，在 1/32 列存储 1，在 1/256 列存储 1，在 1/512 列存储 1，等等。

`0.1`在十进制中是二进制的`0.00011001100110011`…，无限重复：

| 4 | 2 | 1 | . | ½ | ¼ | 1/8 | 1/16 | 1/32 | 1/64 | 1/128 | 1/256 | 1/512 | 1/1024 | 1/2048 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 0 | 0 | . | 0 | 0 | 0 | 1 | 1 | 0 | 0 | 1 | 1 | 0 | 0 |

**良好实践**：切勿使用`==`比较`double`值。在第一次海湾战争期间，美国爱国者导弹电池在其计算中使用了`double`值。这种不准确性导致它未能跟踪并拦截来袭的伊拉克飞毛腿导弹，导致 28 名士兵丧生；你可以在[`www.ima.umn.edu/~arnold/disasters/patriot.html`](https://www.ima.umn.edu/~arnold/disasters/patriot.html)阅读有关此事件的信息。

1.  复制并粘贴你之前写的（使用了`double`变量的）语句。

1.  修改语句以使用`decimal`，并将变量重命名为`c`和`d`，如下所示：

    ```cs
    Console.WriteLine("Using decimals:");
    decimal c = 0.1M; // M suffix means a decimal literal value
    decimal d = 0.2M;
    if (c + d == 0.3M)
    {
      Console.WriteLine($"{c} + {d} equals {0.3M}");
    }
    else
    {
      Console.WriteLine($"{c} + {d} does NOT equal {0.3M}");
    } 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Using decimals:
    0.1 + 0.2 equals 0.3 
    ```

`decimal`类型之所以精确，是因为它将数字存储为一个大整数并移动小数点。例如，`0.1`存储为`1`，并注明将小数点向左移动一位。`12.75`存储为`1275`，并注明将小数点向左移动两位。

**良好实践**：使用`int`表示整数。使用`double`表示不会与其他值比较相等性的实数；比较`double`值是否小于或大于等是可以的。使用`decimal`表示货币、CAD 图纸、通用工程以及任何对实数的精确性很重要的地方。

`double`类型有一些有用的特殊值：`double.NaN`表示非数字（例如，除以零的结果），`double.Epsilon`表示`double`中可以存储的最小的正数，`double.PositiveInfinity`和`double.NegativeInfinity`表示无限大的正负值。

## 存储布尔值

布尔值只能包含`true`或`false`这两个文字值之一，如下所示：

```cs
bool happy = true; 
bool sad = false; 
```

它们最常用于分支和循环。你不需要完全理解它们，因为它们在*第三章*，*控制流程、转换类型和处理异常*中会有更详细的介绍。

## 存储任何类型的对象

有一个名为`object`的特殊类型，可以存储任何类型的数据，但其灵活性是以代码更混乱和可能的性能下降为代价的。由于这两个原因，应尽可能避免使用它。以下步骤展示了如果需要使用对象类型时如何操作：

1.  使用您喜欢的代码编辑器，在`Chapter02`工作区/解决方案中添加一个新的**控制台应用程序**，命名为`Variables`。

1.  在 Visual Studio Code 中，选择`Variables`作为活动 OmniSharp 项目。当看到弹出警告消息提示缺少必需资产时，点击**是**以添加它们。

1.  在`Program.cs`中，键入语句以声明和使用一些使用`object`类型的变量，如下所示：

    ```cs
    object height = 1.88; // storing a double in an object 
    object name = "Amir"; // storing a string in an object
    Console.WriteLine($"{name} is {height} metres tall.");
    int length1 = name.Length; // gives compile error!
    int length2 = ((string)name).Length; // tell compiler it is a string
    Console.WriteLine($"{name} has {length2} characters."); 
    ```

1.  运行代码并注意第四条语句无法编译，因为`name`变量的数据类型编译器未知，如图 2.4 所示：![](img/B17442_02_04.png)

    图 2.4：对象类型没有 Length 属性

1.  在语句开头添加双斜杠注释，以“注释掉”无法编译的语句，使其无效。

1.  再次运行代码并注意，如果程序员明确告诉编译器`object`变量包含一个`string`，通过前缀加上类型转换表达式如`(string)`，编译器可以访问`string`的长度，如下所示：

    ```cs
    Amir is 1.88 metres tall. 
    Amir has 4 characters. 
    ```

`object`类型自 C#的第一个版本起就可用，但 C# 2.0 及更高版本有一个更好的替代方案，称为**泛型**，我们将在*第六章*，*实现接口和继承类*中介绍，它将为我们提供所需的灵活性，而不会带来性能开销。

## 存储动态类型

还有一个名为`dynamic`的特殊类型，也可以存储任何类型的数据，但其灵活性甚至超过了`object`，同样以性能为代价。`dynamic`关键字是在 C# 4.0 中引入的。然而，与`object`不同，存储在变量中的值可以在没有显式类型转换的情况下调用其成员。让我们利用`dynamic`类型：

1.  添加语句以声明一个`dynamic`变量，然后分配一个`string`字面值，接着是一个整数值，最后是一个整数数组，如下所示：

    ```cs
    // storing a string in a dynamic object
    // string has a Length property
    dynamic something = "Ahmed";
    // int does not have a Length property
    // something = 12;
    // an array of any type has a Length property
    // something = new[] { 3, 5, 7 }; 
    ```

1.  添加一个语句以输出`dynamic`变量的长度，如下所示：

    ```cs
    // this compiles but would throw an exception at run-time
    // if you later store a data type that does not have a
    // property named Length
    Console.WriteLine($"Length is {something.Length}"); 
    ```

1.  运行代码并注意它有效，因为`string`值确实具有`Length`属性，如下所示：

    ```cs
    Length is 5 
    ```

1.  取消注释分配`int`值的语句。

1.  运行代码并注意运行时错误，因为`int`没有`Length`属性，如下所示：

    ```cs
    Unhandled exception. Microsoft.CSharp.RuntimeBinder.RuntimeBinderException: 'int' does not contain a definition for 'Length' 
    ```

1.  取消注释分配数组的语句。

1.  运行代码并注意输出，因为一个包含三个`int`值的数组确实具有`Length`属性，如下所示：

    ```cs
    Length is 3 
    ```

`动态`的一个限制是代码编辑器无法显示 IntelliSense 来帮助你编写代码。这是因为编译器在构建时无法检查类型是什么。相反，CLR 在运行时检查成员，并在缺失时抛出异常。

异常是一种指示运行时出现问题的方式。你将在*第三章*，*控制流程、转换类型和处理异常*中了解更多关于它们以及如何处理它们的信息。

## 声明局部变量

局部变量在方法内部声明，它们仅在方法执行期间存在，一旦方法返回，分配给任何局部变量的内存就会被释放。

严格来说，值类型会被立即释放，而引用类型必须等待垃圾回收。你将在*第六章*，*实现接口和继承类*中了解值类型和引用类型之间的区别。

### 指定局部变量的类型

让我们探讨使用特定类型和类型推断声明的局部变量：

1.  使用特定类型声明并赋值给一些局部变量的类型语句，如下列代码所示：

    ```cs
    int population = 66_000_000; // 66 million in UK
    double weight = 1.88; // in kilograms
    decimal price = 4.99M; // in pounds sterling
    string fruit = "Apples"; // strings use double-quotes
    char letter = 'Z'; // chars use single-quotes
    bool happy = true; // Booleans have value of true or false 
    ```

根据你的代码编辑器和配色方案，它会在每个变量名下显示绿色波浪线，并将其文本颜色变浅，以警告你该变量已被赋值但从未使用过。

### 推断局部变量的类型

你可以使用`var`关键字来声明局部变量。编译器将从赋值运算符`=`后分配的值推断出类型。

没有小数点的字面数字默认推断为`整数`变量，除非你添加后缀，如下列列表所述：

+   `L`：推断为`长整型`

+   `UL`：推断为`无符号长整型`

+   `M`：推断为`小数`

+   `D`：推断为`双精度浮点数`

+   `F`：推断为`单精度浮点数`

带有小数点的字面数字默认推断为`双精度浮点数`，除非你添加`M`后缀，在这种情况下，它推断为`小数`变量，或者添加`F`后缀，在这种情况下，它推断为`单精度浮点数`变量。

双引号表示`字符串`变量，单引号表示`字符`变量，而`真`和`假`值则暗示了`布尔`类型：

1.  修改前述语句以使用`var`，如下列代码所示：

    ```cs
    var population = 66_000_000; // 66 million in UK
    var weight = 1.88; // in kilograms
    var price = 4.99M; // in pounds sterling
    var fruit = "Apples"; // strings use double-quotes
    var letter = 'Z'; // chars use single-quotes
    var happy = true; // Booleans have value of true or false 
    ```

1.  将鼠标悬停在每个`var`关键字上，并注意你的代码编辑器会显示一个带有关于已推断类型信息的工具提示。

1.  在类文件顶部，导入用于处理 XML 的命名空间，以便我们能够声明一些使用该命名空间中类型的变量，如下列代码所示：

    ```cs
    using System.Xml; 
    ```

    **良好实践**：如果你正在使用.NET 交互式笔记本，那么请在上层代码单元格中添加`using`语句，并在编写主代码的代码单元格上方。然后点击**执行单元格**以确保命名空间被导入。它们随后将在后续代码单元格中可用。

1.  在前述语句下方，添加语句以创建一些新对象，如下列代码所示：

    ```cs
    // good use of var because it avoids the repeated type
    // as shown in the more verbose second statement
    var xml1 = new XmlDocument(); 
    XmlDocument xml2 = new XmlDocument();
    // bad use of var because we cannot tell the type, so we
    // should use a specific type declaration as shown in
    // the second statement
    var file1 = File.CreateText("something1.txt"); 
    StreamWriter file2 = File.CreateText("something2.txt"); 
    ```

    **最佳实践**：尽管使用`var`很方便，但一些开发者避免使用它，以便代码读者更容易理解正在使用的类型。就我个人而言，我只在使用类型明显时使用它。例如，在前面的代码语句中，第一条语句与第二条一样清晰地说明了`xml`变量的类型，但更短。然而，第三条语句并不清楚地显示`file`变量的类型，所以第四条更好，因为它显示了类型是`StreamWriter`。如果有疑问，就明确写出类型！

### 使用目标类型的新来实例化对象

使用 C# 9，微软引入了一种称为**目标类型的新**的实例化对象的语法。在实例化对象时，你可以先指定类型，然后使用`new`而不重复类型，如下面的代码所示：

```cs
XmlDocument xml3 = new(); // target-typed new in C# 9 or later 
```

如果你有一个需要设置字段或属性的类型，则可以推断类型，如下面的代码所示：

```cs
class Person
{
  public DateTime BirthDate;
}
Person kim = new();
kim.BirthDate = new(1967, 12, 26); // instead of: new DateTime(1967, 12, 26) 
```

**最佳实践**：除非必须使用版本 9 之前的 C#编译器，否则请使用目标类型的新来实例化对象。我在本书的其余部分都使用了目标类型的新。如果你发现我遗漏了任何情况，请告诉我！

## 获取和设置类型的默认值

除了`string`之外的大多数基本类型都是**值类型**，这意味着它们必须有一个值。你可以通过使用`default()`运算符并传递类型作为参数来确定类型的默认值。你可以使用`default`关键字来赋予类型默认值。

`string`类型是**引用类型**。这意味着`string`变量包含值的内存地址，而不是值本身。引用类型变量可以具有`null`值，这是一个指示变量未引用任何内容（尚未）的字面量。`null`是所有引用类型的默认值。

你将在*第六章*，*实现接口和继承类*中了解更多关于值类型和引用类型的信息。

让我们探索默认值：

1.  添加语句以显示`int`、`bool`、`DateTime`和`string`的默认值，如下面的代码所示：

    ```cs
    Console.WriteLine($"default(int) = {default(int)}"); 
    Console.WriteLine($"default(bool) = {default(bool)}"); 
    Console.WriteLine($"default(DateTime) = {default(DateTime)}"); 
    Console.WriteLine($"default(string) = {default(string)}"); 
    ```

1.  运行代码并查看结果，注意如果你的输出日期和时间格式与英国不同，以及`null`值输出为空`string`，如下面的输出所示：

    ```cs
    default(int) = 0 
    default(bool) = False
    default(DateTime) = 01/01/0001 00:00:00 
    default(string) = 
    ```

1.  添加语句以声明一个数字，赋予一个值，然后将其重置为其默认值，如下面的代码所示：

    ```cs
    int number = 13;
    Console.WriteLine($"number has been set to: {number}");
    number = default;
    Console.WriteLine($"number has been reset to its default: {number}"); 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    number has been set to: 13
    number has been reset to its default: 0 
    ```

## 在数组中存储多个值

当你需要存储同一类型的多个值时，你可以声明一个**数组**。例如，当你需要在`string`数组中存储四个名字时，你可能会这样做。

接下来你将编写的代码将为存储四个`string`值的数组分配内存。然后，它将在索引位置 0 到 3 处存储`string`值（数组通常的下界为零，因此最后一项的索引比数组长度小一）。

**良好实践**：不要假设所有数组都从零开始计数。.NET 中最常见的数组类型是**szArray**，一种单维零索引数组，它们使用正常的`[]`语法。但.NET 也有**mdArray**，多维数组，它们不必具有零的下界。这些很少使用，但你应该知道它们存在。

最后，它将使用`for`语句遍历数组中的每个项，我们将在*第三章*，*控制流程、转换类型和处理异常*中更详细地介绍这一点。

让我们看看如何使用数组：

1.  输入语句以声明和使用`string`值的数组，如下面的代码所示：

    ```cs
    string[] names; // can reference any size array of strings
    // allocating memory for four strings in an array
    names = new string[4];
    // storing items at index positions
    names[0] = "Kate";
    names[1] = "Jack"; 
    names[2] = "Rebecca"; 
    names[3] = "Tom";
    // looping through the names
    for (int i = 0; i < names.Length; i++)
    {
      // output the item at index position i
      Console.WriteLine(names[i]);
    } 
    ```

1.  运行代码并记录结果，如下面的输出所示：

    ```cs
    Kate 
    Jack 
    Rebecca 
    Tom 
    ```

数组在内存分配时总是具有固定大小，因此在实例化之前，你需要决定要存储多少项。

除了上述三个步骤定义数组的替代方法是使用数组初始化器语法，如下面的代码所示：

```cs
string[] names2 = new[] { "Kate", "Jack", "Rebecca", "Tom" }; 
```

当你使用`new[]`语法为数组分配内存时，你必须在大括号中至少包含一个项，以便编译器可以推断数据类型。

数组适用于临时存储多个项，但当需要动态添加和删除项时，集合是更灵活的选择。目前你不需要担心集合，因为我们在*第八章*，*使用常见的.NET 类型*中会涉及它们。

# 深入探索控制台应用程序

我们已经创建并使用了基本的控制台应用程序，但现在我们应该更深入地研究它们。

控制台应用程序是基于文本的，并在命令行上运行。它们通常执行需要脚本的简单任务，例如编译文件或加密配置文件的一部分。

同样，它们也可以接受参数来控制其行为。

一个例子是使用指定的名称而不是当前文件夹的名称创建一个新的控制台应用程序，如下面的命令行所示：

```cs
dotnet new console -lang "F#" --name "ExploringConsole" 
```

## 向用户显示输出

控制台应用程序最常见的两个任务是写入和读取数据。我们已经一直在使用`WriteLine`方法输出，但如果我们不希望在行尾有回车，我们可以使用`Write`方法。

### 使用编号的位置参数进行格式化

生成格式化字符串的一种方法是使用编号的位置参数。

此功能受到`Write`和`WriteLine`等方法的支持，对于不支持此功能的方法，可以使用`string`的`Format`方法对`string`参数进行格式化。

本节的前几个代码示例将与.NET Interactive 笔记本一起工作，因为它们是关于输出到控制台的。在本节后面，你将学习通过控制台获取输入，遗憾的是笔记本不支持这一点。

让我们开始格式化：

1.  使用你偏好的代码编辑器，在`Chapter02`工作区/解决方案中添加一个新的**控制台应用程序**，命名为`Formatting`。

1.  在 Visual Studio Code 中，选择`Formatting`作为活动的 OmniSharp 项目。

1.  在`Program.cs`中，输入语句以声明一些数字变量并将它们写入控制台，如下所示：

    ```cs
    int numberOfApples = 12; 
    decimal pricePerApple = 0.35M;
    Console.WriteLine(
      format: "{0} apples costs {1:C}", 
      arg0: numberOfApples,
      arg1: pricePerApple * numberOfApples);
    string formatted = string.Format(
      format: "{0} apples costs {1:C}",
      arg0: numberOfApples,
      arg1: pricePerApple * numberOfApples);
    //WriteToFile(formatted); // writes the string into a file 
    ```

`WriteToFile`方法是一个不存在的用于说明概念的方法。

**良好实践**：一旦你对格式化字符串更加熟悉，你应该停止命名参数，例如，停止使用`format:`、`arg0:`和`arg1:`。前面的代码使用了非规范的风格来显示`0`和`1`的来源，而你正在学习。

### 使用插值字符串进行格式化

C# 6.0 及更高版本有一个名为**插值字符串**的便捷功能。以`$`为前缀的`字符串`可以使用大括号包围变量或表达式的名称，以在该`字符串`中的该位置输出该变量或表达式的当前值，如下所示：

1.  在`Program.cs`文件底部输入如下所示的语句：

    ```cs
    Console.WriteLine($"{numberOfApples} apples costs {pricePerApple * numberOfApples:C}"); 
    ```

1.  运行代码并查看结果，如下面的部分输出所示：

    ```cs
     12 apples costs £4.20 
    ```

对于简短的格式化`字符串`值，插值`字符串`可能更容易让人阅读。但在书籍中的代码示例中，由于行需要跨越多行，这可能很棘手。对于本书中的许多代码示例，我将使用编号的位置参数。

避免使用插值字符串的另一个原因是它们不能从资源文件中读取以进行本地化。

在 C# 10 之前，字符串常量只能通过连接来组合，如下所示：

```cs
private const string firstname = "Omar";
private const string lastname = "Rudberg";
private const string fullname = firstname + " " + lastname; 
```

使用 C# 10，现在可以使用插值字符串，如下所示：

```cs
private const string fullname = "{firstname} {lastname}"; 
```

这只适用于组合字符串常量值。它不能与其他类型（如数字）一起工作，这些类型需要运行时数据类型转换。

### 理解格式字符串

变量或表达式可以在逗号或冒号后使用格式字符串进行格式化。

`N0`格式字符串表示带有千位分隔符且没有小数位的数字，而`C`格式字符串表示货币。货币格式将由当前线程决定。

例如，如果你在英国的 PC 上运行这段代码，你会得到以逗号作为千位分隔符的英镑，但如果你在德国的 PC 上运行这段代码，你将得到以点作为千位分隔符的欧元。

格式项的完整语法是：

```cs
{ index [, alignment ] [ : formatString ] } 
```

每个格式项都可以有一个对齐方式，这在输出值表时很有用，其中一些可能需要在字符宽度内左对齐或右对齐。对齐值是整数。正整数表示右对齐，负整数表示左对齐。

例如，要输出一个水果及其数量的表格，我们可能希望在 10 个字符宽的列内左对齐名称，并在 6 个字符宽的列内右对齐格式化为无小数点的数字计数：

1.  在`Program.cs`底部，输入以下语句：

    ```cs
    string applesText = "Apples"; 
    int applesCount = 1234;
    string bananasText = "Bananas"; 
    int bananasCount = 56789;
    Console.WriteLine(
      format: "{0,-10} {1,6:N0}",
      arg0: "Name",
      arg1: "Count");
    Console.WriteLine(
      format: "{0,-10} {1,6:N0}",
      arg0: applesText,
      arg1: applesCount);
    Console.WriteLine(
      format: "{0,-10} {1,6:N0}",
      arg0: bananasText,
      arg1: bananasCount); 
    ```

1.  运行代码并注意对齐和数字格式的效果，如下所示：

    ```cs
    Name          Count
    Apples        1,234
    Bananas      56,789 
    ```

## 从用户获取文本输入

我们可以使用`ReadLine`方法从用户获取文本输入。该方法等待用户输入一些文本，一旦用户按下 Enter 键，用户输入的任何内容都会作为`string`值返回。

**良好实践**：如果你在本节使用.NET Interactive 笔记本，请注意它不支持使用`Console.ReadLine()`从控制台读取输入。相反，你必须设置文字值，如下所示：`string? firstName = "Gary";`。这通常更快，因为你可以简单地更改文字`string`值并点击**执行单元格**按钮，而不必每次想输入不同`string`值时都重启控制台应用。

让我们获取用户输入：

1.  输入语句以询问用户的姓名和年龄，然后输出他们输入的内容，如下所示：

    ```cs
    Console.Write("Type your first name and press ENTER: "); 
    string? firstName = Console.ReadLine();
    Console.Write("Type your age and press ENTER: "); 
    string? age = Console.ReadLine();
    Console.WriteLine(
      $"Hello {firstName}, you look good for {age}."); 
    ```

1.  运行代码，然后输入姓名和年龄，如下所示：

    ```cs
    Type your name and press ENTER: Gary 
    Type your age and press ENTER: 34 
    Hello Gary, you look good for 34. 
    ```

在`string?`数据类型声明末尾的问号表示我们承认从`ReadLine`调用可能返回`null`（空）值。你将在*第六章*，*实现接口和继承类*中了解更多关于这方面的内容。

## 简化控制台的使用

在 C# 6.0 及更高版本中，`using`语句不仅可以用于导入命名空间，还可以通过导入静态类进一步简化我们的代码。然后，我们就不需要在代码中输入`Console`类型名称。你可以使用代码编辑器的查找和替换功能来删除我们之前写过的`Console`：

1.  在`Program.cs`文件顶部，添加一个语句以**静态导入**`System.Console`类，如下所示：

    ```cs
    using static System.Console; 
    ```

1.  选择代码中的第一个`Console.`，确保也选中了`Console`单词后的点。

1.  在 Visual Studio 中，导航至**编辑** | **查找和替换** | **快速替换**，或在 Visual Studio Code 中，导航至**编辑** | **替换**，并注意一个覆盖对话框出现，准备让你输入你想替换**Console.**的内容，如*图 2.5*所示：![](img/B17442_02_05.png)

    图 2.5：在 Visual Studio 中使用替换功能简化代码

1.  将替换框留空，点击**全部替换**按钮（替换框右侧两个按钮中的第二个），然后通过点击替换框右上角的叉号关闭替换框。

## 从用户获取按键输入

我们可以使用`ReadKey`方法从用户获取按键输入。此方法等待用户按下键或键组合，然后将其作为`ConsoleKeyInfo`值返回。

在.NET Interactive 笔记本中，你将无法执行对`ReadKey`方法的调用，但如果你创建了一个控制台应用程序，那么让我们来探索读取按键操作：

1.  输入语句以要求用户按下任何键组合，然后输出有关它的信息，如下列代码所示：

    ```cs
    Write("Press any key combination: "); 
    ConsoleKeyInfo key = ReadKey(); 
    WriteLine();
    WriteLine("Key: {0}, Char: {1}, Modifiers: {2}",
      arg0: key.Key, 
      arg1: key.KeyChar,
      arg2: key.Modifiers); 
    ```

1.  运行代码，按下 K 键，注意结果，如下列输出所示：

    ```cs
    Press any key combination: k 
    Key: K, Char: k, Modifiers: 0 
    ```

1.  运行代码，按住 Shift 键并按下 K 键，注意结果，如下列输出所示：

    ```cs
    Press any key combination: K  
    Key: K, Char: K, Modifiers: Shift 
    ```

1.  运行代码，按下 F12 键，注意结果，如下列输出所示：

    ```cs
    Press any key combination: 
    Key: F12, Char: , Modifiers: 0 
    ```

在 Visual Studio Code 内的终端中运行控制台应用程序时，某些键盘组合会在你的应用程序处理之前被代码编辑器或操作系统捕获。

## 向控制台应用程序传递参数

你可能一直在思考如何获取可能传递给控制台应用程序的任何参数。

在.NET 的每个版本中，直到 6.0 之前，控制台应用程序项目模板都显而易见，如下列代码所示：

```cs
using System;
namespace Arguments
{
  class Program
  {
    static void Main(string[] args)
    {
      Console.WriteLine("Hello World!");
    }
  }
} 
```

`string[] args`参数在`Program`类的`Main`方法中声明并传递。它们是一个数组，用于向控制台应用程序传递参数。但在.NET 6.0 及更高版本中使用的顶级程序中，`Program`类及其`Main`方法，连同`args`字符串数组的声明都被隐藏了。诀窍在于你必须知道它仍然存在。

命令行参数由空格分隔。其他字符如连字符和冒号被视为参数值的一部分。

要在参数值中包含空格，请用单引号或双引号将参数值括起来。

假设我们希望能够在命令行输入一些前景色和背景色的名称，以及终端窗口的尺寸。我们可以通过从始终传递给`Main`方法（即控制台应用程序的入口点）的`args`数组中读取这些颜色和数字来实现。

1.  使用你偏好的代码编辑器，在`Chapter02`工作区/解决方案中添加一个新的**控制台应用程序**，命名为`Arguments`。由于无法向笔记本传递参数，因此你不能使用.NET Interactive 笔记本。

1.  在 Visual Studio Code 中，选择`Arguments`作为活动的 OmniSharp 项目。

1.  添加一条语句以静态导入`System.Console`类型，并添加一条语句以输出传递给应用程序的参数数量，如下列代码所示：

    ```cs
    using static System.Console;
    WriteLine($"There are {args.Length} arguments."); 
    ```

    **良好实践**：记住在所有未来的项目中静态导入`System.Console`类型，以简化您的代码，因为这些说明不会每次都重复。

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    There are 0 arguments. 
    ```

1.  如果您使用的是 Visual Studio，那么导航到**项目** | **属性** **参数**，选择**调试**选项卡，在**应用程序参数**框中输入一些参数，保存更改，然后运行控制台应用程序，如图*2.6*所示：![图形用户界面，文本，应用程序 自动生成的描述](img/B17442_02_06.png)

    图 2.6：在 Visual Studio 项目属性中输入应用程序参数

1.  如果您使用的是 Visual Studio Code，那么在终端中，在`dotnet run`命令后输入一些参数，如下面的命令行所示：

    ```cs
    dotnet run firstarg second-arg third:arg "fourth arg" 
    ```

1.  注意结果显示了四个参数，如下面的输出所示：

    ```cs
    There are 4 arguments. 
    ```

1.  要枚举或迭代（即循环遍历）这四个参数的值，请在输出数组长度后添加以下语句：

    ```cs
    foreach (string arg in args)
    {
      WriteLine(arg);
    } 
    ```

1.  再次运行代码，并注意结果显示了四个参数的详细信息，如下面的输出所示：

    ```cs
    There are 4 arguments. 
    firstarg
    second-arg 
    third:arg 
    fourth arg 
    ```

## 使用参数设置选项

现在我们将使用这些参数让用户为输出窗口的背景、前景和光标大小选择颜色。光标大小可以是 1 到 100 的整数值，1 表示光标单元格底部的线条，100 表示光标单元格高度的百分比。

`System`命名空间已经导入，以便编译器知道`ConsoleColor`和`Enum`类型：

1.  添加语句以警告用户，如果他们没有输入三个参数，然后解析这些参数并使用它们来设置控制台窗口的颜色和尺寸，如下面的代码所示：

    ```cs
    if (args.Length < 3)
    {
      WriteLine("You must specify two colors and cursor size, e.g.");
      WriteLine("dotnet run red yellow 50");
      return; // stop running
    }
    ForegroundColor = (ConsoleColor)Enum.Parse(
      enumType: typeof(ConsoleColor),
      value: args[0],
      ignoreCase: true);
    BackgroundColor = (ConsoleColor)Enum.Parse(
      enumType: typeof(ConsoleColor),
      value: args[1],
      ignoreCase: true);
    CursorSize = int.Parse(args[2]); 
    ```

    设置`CursorSize`仅在 Windows 上支持。

1.  在 Visual Studio 中，导航到**项目** | **属性参数**，并将参数更改为：`red yellow 50`，运行控制台应用程序，并注意光标大小减半，窗口中的颜色已更改，如图*2.7*所示：![图形用户界面，应用程序，网站 自动生成的描述](img/B17442_02_07.png)

    图 2.7：在 Windows 上设置颜色和光标大小

1.  在 Visual Studio Code 中，使用参数运行代码，将前景色设置为红色，背景色设置为黄色，光标大小设置为 50%，如下面的命令所示：

    ```cs
    dotnet run red yellow 50 
    ```

    在 macOS 上，您会看到一个未处理的异常，如图*2.8*所示：

    ![图形用户界面，文本，应用程序 自动生成的描述](img/B17442_02_08.png)

图 2.8：在不受支持的 macOS 上出现未处理的异常

尽管编译器没有给出错误或警告，但在某些平台上运行时，某些 API 调用可能会失败。虽然 Windows 上的控制台应用程序可以更改光标大小，但在 macOS 上却无法实现，并且尝试时会报错。

## 处理不支持 API 的平台

那么我们该如何解决这个问题呢？我们可以通过使用异常处理器来解决。你将在*第三章*，*控制流程、类型转换和异常处理*中了解更多关于`try-catch`语句的细节，所以现在只需输入代码：

1.  修改代码，将更改光标大小的行包裹在`try`语句中，如下所示：

    ```cs
    try
    {
      CursorSize = int.Parse(args[2]);
    }
    catch (PlatformNotSupportedException)
    {
      WriteLine("The current platform does not support changing the size of the cursor.");
    } 
    ```

1.  如果你在 macOS 上运行这段代码，你会看到异常被捕获，并向用户显示一个更友好的消息。

另一种处理操作系统差异的方法是使用`System`命名空间中的`OperatingSystem`类，如下所示：

```cs
if (OperatingSystem.IsWindows())
{
  // execute code that only works on Windows
}
else if (OperatingSystem.IsWindowsVersionAtLeast(major: 10))
{
  // execute code that only works on Windows 10 or later
}
else if (OperatingSystem.IsIOSVersionAtLeast(major: 14, minor: 5))
{
  // execute code that only works on iOS 14.5 or later
}
else if (OperatingSystem.IsBrowser())
{
  // execute code that only works in the browser with Blazor
} 
```

`OperatingSystem`类为其他常见操作系统（如 Android、iOS、Linux、macOS，甚至是浏览器）提供了等效方法，这对于 Blazor Web 组件非常有用。

处理不同平台的第三种方法是使用条件编译语句。

有四个预处理器指令控制条件编译：`#if`、`#elif`、`#else`和`#endif`。

你使用`#define`定义符号，如下所示：

```cs
#define MYSYMBOL 
```

许多符号会自动为你定义，如下表所示：

| 目标框架 | 符号 |
| --- | --- |
| .NET 标准 | `NETSTANDARD2_0`、`NETSTANDARD2_1`等 |
| 现代.NET | `NET6_0`、`NET6_0_ANDROID`、`NET6_0_IOS`、`NET6_0_WINDOWS`等 |

然后你可以编写仅针对指定平台编译的语句，如下所示：

```cs
#if NET6_0_ANDROID
// compile statements that only works on Android
#elif NET6_0_IOS
// compile statements that only works on iOS
#else
// compile statements that work everywhere else
#endif 
```

# 实践与探索

通过回答一些问题来测试你的知识和理解，进行一些实践练习，并深入研究本章涵盖的主题。

## 练习 2.1 – 测试你的知识

为了得到这些问题的最佳答案，你需要进行自己的研究。我希望你能“跳出书本思考”，因此我故意没有在书中提供所有答案。

我想鼓励你养成在其他地方寻求帮助的好习惯，遵循“授人以渔”的原则。

1.  你可以在 C#文件中输入什么语句来发现编译器和语言版本？

1.  在 C#中有哪两种类型的注释？

1.  逐字字符串和插值字符串之间有什么区别？

1.  为什么在使用`float`和`double`值时要小心？

1.  如何确定像`double`这样的类型在内存中占用多少字节？

1.  何时应该使用`var`关键字？

1.  创建`XmlDocument`类实例的最新方法是什么？

1.  为什么在使用`dynamic`类型时要小心？

1.  如何右对齐格式字符串？

1.  控制台应用程序的参数之间用什么字符分隔？

    *附录*，*测试你的知识问题的答案*可从 GitHub 仓库的 README 中的链接下载：[`github.com/markjprice/cs10dotnet6`](https://github.com/markjprice/cs10dotnet6)。

## 练习 2.2 – 测试你对数字类型的知识

你会为以下“数字”选择什么类型？

1.  一个人的电话号码

1.  一个人的身高

1.  一个人的年龄

1.  一个人的薪水

1.  一本书的 ISBN

1.  一本书的价格

1.  一本书的运送重量

1.  一个国家的人口

1.  宇宙中的恒星数量

1.  英国中小型企业中每个企业的员工数量（每个企业最多约 50,000 名员工）

## 练习 2.3 – 实践数字大小和范围

在`Chapter02`解决方案/工作区中，创建一个名为`Exercise02`的控制台应用程序项目，输出以下每种数字类型在内存中占用的字节数及其最小和最大值：`sbyte`、`byte`、`short`、`ushort`、`int`、`uint`、`long`、`ulong`、`float`、`double`和`decimal`。

运行你的控制台应用程序的结果应该类似于*图 2.9*：

![自动生成的文本描述](img/B17442_02_09.png)

图 2.9：输出数字类型大小的结果

所有练习的代码解决方案都可以从以下链接下载或克隆 GitHub 仓库：[`github.com/markjprice/cs10dotnet6`](https://github.com/markjprice/cs10dotnet6)。

## 练习 2.4 – 探索主题

使用以下页面上的链接来了解本章涵盖的主题的更多细节：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#第二章-使用 C#进行编程`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-2---speaking-c)

# 总结

在本章中，你学会了如何：

+   声明具有指定或推断类型的变量。

+   使用一些内置的数字、文本和布尔类型。

+   选择数字类型

+   控制控制台应用程序的输出格式。

在下一章中，你将学习运算符、分支、循环、类型转换以及如何处理异常。
