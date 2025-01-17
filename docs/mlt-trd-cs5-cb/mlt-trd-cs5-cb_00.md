# 前言

不久以前，典型的个人电脑 CPU 只有一个计算核心，功耗足以在上面煎鸡蛋。2005 年，英特尔推出了其第一款多核 CPU，自那时起，计算机开始朝着不同的方向发展。低功耗和多个计算核心变得比单个计算核心的性能更重要。这也导致了编程范式的变化。现在我们需要学会如何有效地使用所有 CPU 核心以实现最佳性能，同时通过仅在特定时间运行所需的程序来节省电池电量。此外，我们还需要以尽可能高效的方式编写服务器应用程序，以利用多个 CPU 核心甚至多台计算机来支持尽可能多的用户。

要能够创建这样的应用程序，您必须学会有效地在程序中使用多个 CPU 核心。如果您使用 Microsoft .NET 开发平台和 C#编程语言，这本书将是编写性能良好且响应迅速的应用程序的完美起点。

本书的目的是为您提供 C#中多线程和并行编程的逐步指南。我们将从基本概念开始，根据前几章的信息逐渐深入更高级的主题，并以真实世界的并行编程模式和 Windows Store 应用程序示例结束。

# **本书内容**

第一章，“线程基础”，介绍了 C#中线程的基本操作。它解释了线程是什么，使用线程的利弊以及其他重要的线程方面。

第二章，“线程同步”，描述了线程交互的细节。您将了解为什么我们需要协调线程以及组织线程协调的不同方式。

第三章，“使用线程池”，解释了线程池的概念。它展示了如何使用线程池，如何处理异步操作，以及使用线程池的良好和不良实践。

第四章，“使用任务并行库”，深入探讨了任务并行库框架。本章概述了 TPL 的每个重要方面，包括任务组合、异常管理和操作取消。

第五章，“使用 C# 5.0”，详细解释了新的 C# 5.0 特性 - 异步方法。您将了解 async 和 await 关键字的含义，以及如何在不同场景中使用它们，以及 await 在幕后的工作原理。

第六章，“使用并发集合”，描述了.NET Framework 中包含的用于并行算法的标准数据结构。它涵盖了每种数据结构的示例编程场景。

第七章，“使用 PLINQ”，是对并行 LINQ 基础设施的深入探讨。本章描述了任务和数据并行性，对 LINQ 查询进行并行化，调整并行性选项，对查询进行分区，并聚合并行查询结果。

第八章，“响应式扩展”，解释了何时以及如何使用响应式扩展框架。您将学习如何组合事件以及如何针对事件序列执行 LINQ 查询。

第九章，“使用异步 I/O”，详细介绍了包括文件、网络和数据库场景在内的异步 I/O 过程。

第十章，“并行编程模式”，概述了常见的并行编程问题解决方案。

第十一章，*更多内容*，涵盖了为 Windows 8 编写异步应用程序的方面。您将学习如何使用 Windows 8 异步 API，并在 Windows Store 应用程序中执行后台工作。

# 本书所需内容

对于大多数的示例，您将需要 Microsoft Visual Studio Express 2012 for Windows Desktop。第十一章的示例将需要 Windows 8 和 Microsoft Visual Studio Express 2012 for Windows 8 来编译 Windows Store 应用程序。

# 本书适合对象

*Multithreading in C# 5.0 Cookbook* 是为现有的 C# 开发人员编写的，他们在多线程、异步和并行编程方面几乎没有背景。本书涵盖了从基本概念到使用 C# 和 .NET 生态系统的复杂编程模式和算法的这些主题。

# 约定

在本书中，您将找到一些区分不同信息类型的文本样式。以下是一些这些样式的示例，以及它们含义的解释。

文本中的代码单词显示如下：“当我们构造一个线程时，`ThreadStart` 或 `ParameterizedThreadStart` 委托的实例被传递给构造函数。”

代码块设置如下：

```cs
static void PrintNumbers()
{
  Console.WriteLine("Starting...");
  for (int i = 1; i < 10; i++)
  {
    Console.WriteLine(i);
  }
}
```

**新术语** 和 **重要单词** 以粗体显示。例如，屏幕上看到的单词，例如菜单或对话框中的单词，会在文本中显示为：“启动 Visual Studio 2012。创建一个新的 C# **控制台应用程序** 项目。”

### 注意

警告或重要提示会以这样的方式显示在一个框中。

### 提示

提示和技巧看起来像这样。
