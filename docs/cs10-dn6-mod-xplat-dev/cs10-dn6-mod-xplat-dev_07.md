# 第七章：打包和分发.NET 类型

本章介绍了 C#关键字与.NET 类型的关系，以及命名空间和程序集之间的关系。您还将熟悉如何打包和发布您的.NET 应用程序和库以供跨平台使用，如何在.NET 库中使用传统的.NET Framework 库，以及将传统的.NET Framework 代码库移植到现代.NET 的可能性。

本章涵盖以下主题：

+   通往.NET 6 的道路

+   理解.NET 组件

+   发布您的应用程序以进行部署

+   反编译.NET 程序集

+   为 NuGet 分发打包您的库

+   从.NET Framework 迁移到现代.NET

+   使用预览功能工作

# 通往.NET 6 的道路

本书的这一部分介绍了.NET 提供的**基类库**（**BCL**）API 中的功能，以及如何使用.NET Standard 在所有不同的.NET 平台上重用功能。

首先，我们将回顾到达这一点的路线以及了解过去的重要性。

.NET Core 2.0 及更高版本对于最低.NET Standard 2.0 的支持非常重要，因为它提供了许多在第一个版本的.NET Core 中缺失的 API。.NET Framework 开发人员在过去 15 年中可用的库和应用程序现在已经迁移到.NET，并且可以在 macOS 和 Linux 变体上跨平台运行，以及在 Windows 上运行。

.NET Standard 2.1 增加了大约 3,000 个新的 API。其中一些 API 需要运行时更改，这将破坏向后兼容性，因此.NET Framework 4.8 只实现.NET Standard 2.0。.NET Core 3.0、Xamarin、Mono 和 Unity 实现了.NET Standard 2.1。

如果您的所有项目都可以使用.NET 6，.NET 6 将不再需要.NET Standard。由于您可能仍然需要为传统的.NET Framework 项目或传统的 Xamarin 移动应用程序创建类库，因此仍然需要创建.NET Standard 2.0 和 2.1 类库。在 2021 年 3 月，我对专业开发人员进行了调查，一半的人仍然需要创建符合.NET Standard 2.0 的类库。

现在.NET 6 已经发布，支持使用.NET MAUI 构建移动和桌面应用程序的预览，因此进一步减少了对.NET Standard 的需求。

为了总结过去五年中.NET 的进展，我已经将主要的.NET Core 和现代.NET 版本与等效的.NET Framework 版本进行了比较，具体如下：

+   **.NET Core 1.x**：与 2016 年 3 月的当前版本.NET Framework 4.6.1 相比，API 要小得多。

+   **.NET Core 2.x**：与.NET Framework 4.7.1 相比，现代 API 达到了 API 的平等，因为它们都实现了.NET Standard 2.0。

+   **.NET Core 3.x**：与.NET Framework 相比，现代 API 更大，因为.NET Framework 4.8 没有实现.NET Standard 2.1。

+   **.NET 5**：与现代 API 相比，与.NET Framework 4.8 相比，API 更大，性能得到了大幅提升。

+   **.NET 6**：预计将在 2022 年 5 月之前最终统一，并支持.NET MAUI 中的移动应用程序。

## .NET Core 1.0

.NET Core 1.0 于 2016 年 6 月发布，重点是实现适用于构建现代跨平台应用程序的 API，包括使用 ASP.NET Core 为 Linux 构建的 Web 和云应用程序和服务。

## .NET Core 1.1

.NET Core 1.1 于 2016 年 11 月发布，重点是修复错误，增加支持的 Linux 发行版数量，支持.NET Standard 1.6，并提高性能，特别是对于 Web 应用程序和服务的 ASP.NET Core。

## .NET Core 2.0

.NET Core 2.0 于 2017 年 8 月发布，重点是实现.NET Standard 2.0，引用.NET Framework 库的能力，以及更多的性能改进。

这本书的第三版于 2017 年 11 月出版，因此涵盖了.NET Core 2.0 和**通用 Windows 平台**（**UWP**）应用程序。

## .NET Core 2.1

.NET Core 2.1 于 2018 年 5 月发布，重点是可扩展的工具系统，添加了新类型如`Span<T>`，新的加密和压缩 API，一个 Windows 兼容包，其中包含额外的 20,000 个 API，以帮助移植旧的 Windows 应用程序，Entity Framework Core 值转换，LINQ `GroupBy`转换，数据种子，查询类型，以及更多性能改进，包括以下表中列出的主题：

| 功能 | 章节 | 主题 |
| --- | --- | --- |
| Spans | 8 | 使用 spans、索引和范围 |
| Brotli 压缩 | 9 | 使用 Brotli 算法进行压缩 |
| 加密 | 20 | 加密的新功能是什么？ |
| EF Core 懒加载 | 10 | 启用延迟加载 |
| EF Core 数据种子 | 10 | 理解数据种子 |

## .NET Core 2.2

.NET Core 2.2 于 2018 年 12 月发布，重点是对运行时的诊断改进，可选的分层编译，并为 ASP.NET Core 和 Entity Framework Core 添加了新功能，如使用**NetTopologySuite**（**NTS**）库中的类型支持空间数据，查询标签和拥有实体的集合。

## .NET Core 3.0

.NET Core 3.0 于 2019 年 9 月发布，重点是添加对使用 Windows 窗体（2001）、**Windows Presentation Foundation**（**WPF**；2006）和 Entity Framework 6.3 构建 Windows 桌面应用程序的支持，同时支持应用程序本地部署，快速 JSON 读取器，串口访问和其他用于**物联网**（**IoT**）解决方案的引脚访问，以及默认情况下的分层编译，包括以下表中列出的主题：

| 功能 | 章节 | 主题 |
| --- | --- | --- |
| 将.NET 嵌入应用程序中 | 7 | 发布您的应用程序以进行部署 |
| `Index` 和 `Range` | 8 | 使用 spans、索引和范围 |
| `System.Text.Json` | 9 | 高性能 JSON 处理 |
| 异步流 | 12 | 使用异步流 |

该书的第四版于 2019 年 10 月出版，因此涵盖了直到.NET Core 3.0 版本后添加的一些新 API。

## .NET Core 3.1

.NET Core 3.1 于 2019 年 12 月发布，重点是修复错误和改进，以便成为**长期支持**（**LTS**）版本，直到 2022 年 12 月才停止支持。

## .NET 5.0

.NET 5.0 于 2020 年 11 月发布，重点是统一各种.NET 平台（除移动平台），完善平台，并提高性能，包括以下表中列出的主题：

| 功能 | 章节 | 主题 |
| --- | --- | --- |
| `Half`类型 | 8 | 处理数字 |
| 正则表达式性能改进 | 8 | 正则表达式性能改进 |
| `System.Text.Json` 改进 | 9 | 高性能 JSON 处理 |
| EF Core 生成的 SQL | 10 | 获取生成的 SQL |
| EF Core Filtered Include | 10 | 过滤包含的实体 |
| EF Core Scaffold-DbContext 现在使用 Humanizer 进行单数化 | 10 | 使用现有数据库搭建模型 |

## .NET 6.0

.NET 6.0 于 2021 年 11 月发布，重点是与移动平台统一，为 EF Core 添加更多数据管理功能，并提高性能，包括以下表中列出的主题：

| 功能 | 章节 | 主题 |
| --- | --- | --- |
| 检查.NET SDK 状态 | 7 | 检查.NET SDK 是否有更新 |
| Apple Silicon 支持 | 7 | 创建用于发布的控制台应用程序 |
| 将链接修剪模式设置为默认 | 7 | 使用应用修剪减小应用程序的大小 |
| `DateOnly` 和 `TimeOnly` | 8 | 指定日期和时间值 |
| `List<T>` 的 `EnsureCapacity` | 8 | 通过确保集合的容量来提高性能 |
| EF Core 配置约定 | 10 | 配置预约定模型 |
| 新的 LINQ 方法 | 11 | 使用 Enumerable 类构建 LINQ 表达式 |

## 从.NET Core 2.0 到.NET 5 的性能改进

在过去几年中，微软在性能方面取得了显著的改进。您可以在以下链接阅读详细的博客文章：[`devblogs.microsoft.com/dotnet/performance-improvements-in-net-5/`](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-5/)。

## 检查您的.NET SDK 是否有更新

在.NET 6 中，微软添加了一个命令来检查您已安装的.NET SDK 和运行时的版本，并在需要更新时提醒您。例如，您输入以下命令：

```cs

dotnet sdk check

```

然后，您将看到结果，包括可用更新的状态，如下面的部分输出所示：

```cs

.NET SDK：

版本                         状态

-----------------------------------------------------------------------------

3.1.412                         最新。

5.0.202                         补丁 5.0.206 可用。

...

```

# 理解.NET 组件

.NET 由几个部分组成，如下列表所示：

+   **语言编译器**：这些将您用 C#、F#和 Visual Basic 等语言编写的源代码转换为存储在程序集中的**中间语言**（**IL**）代码。从 C# 6.0 开始，微软切换到了一个名为 Roslyn 的开源重写编译器，它也被 Visual Basic 使用。

+   **公共语言运行时（CoreCLR）**：这个运行时加载程序集，将它们中存储的 IL 代码编译成本地代码指令，然后在管理资源（如线程和内存）的环境中执行代码。

+   **基础类库（BCL 或 CoreFX）**：这些是使用 NuGet 打包和分发的预构建类型程序集，用于构建应用程序时执行常见任务。您可以使用它们快速构建任何您想要的东西，就像组合 LEGO™积木一样。.NET Core 2.0 实现了.NET Standard 2.0，它是所有以前版本的.NET Standard 的超集，并将.NET Core 提升到了与.NET Framework 和 Xamarin 的平等。.NET Core 3.0 实现了.NET Standard 2.1，它增加了新的功能，并使性能优于.NET Framework。.NET 6 实现了跨所有类型应用程序的统一 BCL，包括移动应用程序。

## 理解程序集、NuGet 包和命名空间

**程序集**是类型存储在文件系统中的地方。程序集是部署代码的一种机制。例如，`System.Data.dll`程序集包含了管理数据的类型。要使用其他程序集中的类型，必须引用它们。程序集可以是静态的（预先创建的）或动态的（在运行时生成的）。动态程序集是我们在本书中不会涉及的高级功能。程序集可以编译成一个文件，作为 DLL（类库）或 EXE（控制台应用）。

程序集以**NuGet 包**的形式分发，这些文件可以从公共在线源下载，并且可以包含多个程序集和其他资源。您还会听到**项目 SDK**、**工作负载**和**平台**，它们是 NuGet 包的组合。

微软的 NuGet 源可以在这里找到：[`www.nuget.org/`](https://www.nuget.org/)。

### 什么是命名空间？

命名空间是类型的地址。命名空间是一种通过需要完整地址而不仅仅是一个简短名称来唯一标识类型的机制。在现实世界中，*34 Sycamore Street 的 Bob*和*12 Willow Drive 的 Bob*是不同的。

在.NET 中，`System.Web.Mvc`命名空间的`IActionFilter`接口与`System.Web.Http.Filters`命名空间的`IActionFilter`接口是不同的。

### 理解依赖程序集

如果一个程序集被编译为类库并提供其他程序集使用的类型，那么它的文件扩展名是`.dll`（**动态链接库**），它不能独立执行。

同样，如果一个程序集被编译为应用程序，那么它的文件扩展名是`.exe`（**可执行文件**），可以独立执行。在.NET Core 3.0 之前，控制台应用程序被编译为`.dll`文件，并且必须通过`dotnet run`命令或主机可执行文件来执行。

任何程序集都可以引用一个或多个类库程序集作为依赖项，但不能有循环引用。因此，如果程序集*A*已经引用了程序集*B*，那么程序集*B*就不能引用程序集*A*。如果您尝试添加一个会导致循环引用的依赖引用，编译器会警告您。循环引用通常是糟糕代码设计的警告信号。如果您确定需要循环引用，那么使用接口来解决它。

## 理解 Microsoft .NET 项目 SDK

默认情况下，控制台应用程序依赖于 Microsoft .NET 项目 SDK。该平台包含了几乎所有应用程序都需要的 NuGet 包中的成千上万的类型，例如`System.Int32`和`System.String`类型。

在使用.NET 时，您需要在项目文件中引用依赖程序集、NuGet 包和应用程序所需的平台。

让我们来探讨程序集和命名空间之间的关系：

1.  使用您喜欢的代码编辑器创建一个名为`Chapter07`的新解决方案/工作区。

1.  添加一个控制台应用程序项目，如下列表所定义的：

1.  项目模板：**控制台应用程序** / `console`

1.  工作区/解决方案文件和文件夹：`Chapter07`

1.  项目文件和文件夹：`AssembliesAndNamespaces`

1.  打开`AssembliesAndNamespaces.csproj`，注意它是一个典型的.NET 6 应用程序的项目文件，如下标记所示：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

<OutputType>Exe</OutputType>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<ImplicitUsings>enable</ImplicitUsings>

</PropertyGroup>

</Project>

```

## 理解程序集中的命名空间和类型

许多常见的.NET 类型都在`System.Runtime.dll`程序集中。程序集和命名空间之间并不总是一一对应。一个程序集可以包含多个命名空间，一个命名空间可以在多个程序集中定义。您可以看到一些程序集与它们提供类型的命名空间之间的关系，如下表所示：

| 程序集 | 示例命名空间 | 示例类型 |
| --- | --- | --- |
| `System.Runtime.dll` | `System` , `System.Collections` , `System.Collections.Generic` | `Int32` , `String` , `IEnumerable<T>` |
| `System.Console.dll` | `System` | `Console` |
| `System.Threading.dll` | `System.Threading` | `Interlocked` , `Monitor` , `Mutex` |
| `System.Xml.XDocument.dll` | `System.Xml.Linq` | `XDocument` , `XElement` , `XNode` |

## 理解 NuGet 包

.NET 被分成一组包，使用名为 NuGet 的 Microsoft 支持的包管理技术进行分发。这些包中的每一个代表着同名的单个程序集。例如，`System.Collections`包包含了`System.Collections.dll`程序集。

以下是包的好处：

+   包可以轻松地在公共源上分发。

+   包可以被重用。

+   包可以按照自己的时间表发布。

+   包可以独立于其他包进行测试。

+   包可以通过包含为不同操作系统和 CPU 构建的同一程序集的多个版本来支持不同的操作系统和 CPU。

+   包可以具有特定于一个库的依赖项。

+   应用程序更小，因为未引用的包不会成为分发的一部分。以下表列出了一些更重要的包及其重要的类型：

| 包 | 重要类型 |
| --- | --- |
| `System.Runtime` | `Object` , `String` , `Int32` , `Array` |
| `System.Collections` | `List<T>` , `Dictionary<TKey, TValue>` |
| `System.Net.Http` | `HttpClient` , `HttpResponseMessage` |
| `System.IO.FileSystem` | `File` , `Directory` |
| `System.Reflection` | `Assembly` , `TypeInfo` , `MethodInfo` |

## 理解框架

框架和包之间存在双向关系。包定义 API，而框架将包分组。没有任何包的框架将不定义任何 API。

.NET 包各自支持一组框架。例如，`System.IO.FileSystem` 包版本 4.3.0 支持以下框架：

+   .NET Standard，1.3 版或更高版本。

+   .NET Framework，4.6 版或更高版本。

+   六个 Mono 和 Xamarin 平台（例如，Xamarin.iOS 1.0）。

**更多信息**：您可以在以下链接阅读详细信息：[`www.nuget.org/packages/System.IO.FileSystem/`](https://www.nuget.org/packages/System.IO.FileSystem/)。

## 导入命名空间以使用类型

让我们探讨命名空间与程序集和类型的关系：

1.  在 `AssembliesAndNamespaces` 项目的 `Program.cs` 中，输入以下代码：

```cs

XDocument doc = new

();

```

1.  构建项目并注意编译器错误消息，如下所示：

```cs

类型或命名空间名称 'XDocument' 无法找到（您是否缺少 using 指令或程序集引用？）

```

`XDocument` 类型未被识别，因为我们没有告诉编译器该类型的命名空间是什么。尽管该项目已经引用了包含该类型的程序集，但我们还需要使用其命名空间前缀类型名称，或者导入命名空间。

1.  点击 `XDocument` 类名。您的代码编辑器会显示一个灯泡，表明它识别到了该类型，并可以自动为您修复问题。

1.  点击灯泡，并从菜单中选择 `using System.Xml.Linq;`。

这将通过在文件顶部添加 `using` 语句*导入命名空间*。一旦命名空间在代码文件顶部被导入，那么命名空间中的所有类型都可以在该代码文件中使用，只需输入它们的名称，而不需要通过在其前缀中加上其命名空间来完全限定类型名称。

有时我喜欢在导入命名空间后添加一个带有类型名称的注释，以提醒我为什么需要导入该命名空间，如下所示：

```cs

使用

System.Xml.Linq; // XDocument

```

## 将 C# 关键字与 .NET 类型相关联

我从新手 C# 程序员那里经常得到的一个常见问题是，“`string` 和大写 `String` 之间有什么区别？”

简短的答案很简单：没有。长答案是，所有 C# 类型关键字，如 `string` 或 `int`，都是类库程序集中 .NET 类型的别名。

当您使用 `string` 关键字时，编译器将其识别为 `System.String` 类型。当您使用 `int` 类型时，编译器将其识别为 `System.Int32` 类型。

让我们通过一些代码来看看这一点：

1.  在 `Program.cs` 中，声明两个变量来保存 `string` 值，一个使用小写的 `string`，一个使用大写的 `String`，如下所示：

```cs

string

s1 = "Hello"

;

String s2 = "World"

;

WriteLine($"

{s1}

{s2}

"

);

```

1.  运行代码，并注意目前它们都同样有效，并且字面上意思相同。

1.  在 `AssembliesAndNamespaces.csproj` 中，添加条目以防止全局导入 `System` 命名空间，如下所示：

```cs

<ItemGroup>

<使用 Remove="System"

/>

</ItemGroup>

```

1.  在 `Program.cs` 中注意编译器错误消息，如下所示：

```cs

类型或命名空间名称 'String' 无法找到（您是否缺少 using 指令或程序集引用？）

```

1.  在 `Program.cs` 的顶部，使用 `using` 语句导入 `System` 命名空间，以修复错误，如下所示：

```cs

使用

System; // String

```

**良好实践**：当有选择时，使用 C# 关键字而不是实际类型，因为关键字不需要导入命名空间。

### 将 C# 别名映射到 .NET 类型

以下表格显示了 18 个 C# 类型关键字以及它们的实际 .NET 类型：

| 关键字 | .NET 类型 | 关键字 | .NET 类型 |
| --- | --- | --- | --- |
| `string` | `System.String` | `char` | `System.Char` |
| `sbyte` | `System.SByte` | `byte` | `System.Byte` |
| `short` | `System.Int16` | `ushort` | `System.UInt16` |
| `int` | `System.Int32` | `uint` | `System.UInt32` |
| `long` | `System.Int64` | `ulong` | `System.UInt64` |
| `nint` | `System.IntPtr` | `nuint` | `System.UIntPtr` |
| `float` | `System.Single` | `double` | `System.Double` |
| `decimal` | `System.Decimal` | `bool` | `System.Boolean` |
| `object` | `System.Object` | `dynamic` | `System.Dynamic.DynamicObject` |

其他.NET 编程语言编译器也可以做同样的事情。例如，Visual Basic .NET 语言有一个名为`Integer`的类型，它是`System.Int32`的别名。

#### 理解本机大小的整数

C# 9 引入了`nint`和`nuint`关键字别名，表示**本机大小的整数**，这意味着整数值的存储大小是特定于平台的。它们在 32 位进程中存储 32 位整数，并且`sizeof()`返回 4 个字节；它们在 64 位进程中存储 64 位整数，并且`sizeof()`返回 8 个字节。这些别名表示内存中整数值的指针，这就是它们的.NET 名称为`IntPtr`和`UIntPtr`的原因。实际的存储类型将根据进程是`System.Int32`还是`System.Int64`。

在 64 位进程中，以下代码：

```cs

WriteLine($"int.MaxValue =

{

int

.MaxValue:N0}

"

);

WriteLine($"nint.MaxValue =

{

nint

.MaxValue:N0}

"

);

```

产生以下输出：

```cs

int.MaxValue = 2,147,483,647

nint.MaxValue = 9,223,372,036,854,775,807

```

### 揭示类型的位置

代码编辑器为.NET 类型提供了内置文档。让我们来探索一下：

1.  右键单击`XDocument`内部，然后选择**转到定义**。

1.  导航到代码文件的顶部，注意程序集文件名是`System.Xml.XDocument.dll`，但该类在`System.Xml.Linq`命名空间中，如*图 7.1*所示：![图形用户界面，文本，应用程序，电子邮件描述自动生成](img/Image00076.jpg)

图 7.1：包含 XDocument 类型的程序集和命名空间

1.  关闭**XDocument [from metadata]**标签。

1.  右键单击`string`或`String`内部，然后选择**转到定义**。

1.  导航到代码文件的顶部，注意程序集文件名是`System.Runtime.dll`，但该类在`System`命名空间中。

实际上，您的代码编辑器在技术上是在欺骗您。如果您还记得我们在*第二章*，*说 C#*中编写代码时，当我们揭示了 C#词汇量的范围时，我们发现`System.Runtime.dll`程序集中不包含任何类型。

它包含的是类型转发器。这些是特殊类型，看起来存在于一个程序集中，但实际上是在其他地方实现的。在这种情况下，它们是使用高度优化的代码在.NET 运行时深处实现的。

## 使用.NET Standard 与旧平台共享代码

在.NET Standard 之前，有**可移植类库**（**PCLs**）。使用 PCLs，您可以创建一个代码库，并明确指定要支持该库的平台，例如 Xamarin、Silverlight 和 Windows 8。然后，您的库可以使用指定平台支持的 API 的交集。

微软意识到这是不可持续的，所以他们创建了.NET Standard——一个所有未来.NET 平台都将支持的单一 API。有较早版本的.NET Standard，但.NET Standard 2.0 是统一所有重要的最近.NET 平台的尝试。.NET Standard 2.1 在 2019 年底发布，但只有.NET Core 3.0 和当年的 Xamarin 版本支持其新功能。在本书的其余部分，我将使用术语.NET Standard 来指代.NET Standard 2.0。

.NET Standard 类似于 HTML5，因为它们都是平台应该支持的标准。就像谷歌的 Chrome 浏览器和微软的 Edge 浏览器实现 HTML5 标准一样，.NET Core、.NET Framework 和 Xamarin 都实现.NET Standard。如果您想要创建一个可以在各种旧版.NET 上工作的类型库，那么最容易使用.NET Standard。

**良好实践**：由于.NET Standard 2.1 中的许多 API 添加需要运行时更改，而.NET Framework 是微软的遗留平台，需要尽可能保持不变，因此.NET Framework 4.8 仍然保持在.NET Standard 2.0 上，而不是实现.NET Standard 2.1。如果您需要支持.NET Framework 客户端，则应该在.NET Standard 2.0 上创建类库，即使它不是最新的，也不支持所有最近的语言和 BCL 新功能。

您选择要针对的.NET Standard 版本取决于最大化平台支持和可用功能之间的平衡。较低版本支持更多平台，但具有较小的 API 集。较高版本支持较少的平台，但具有更大的 API 集。通常，您应该选择支持您需要的所有 API 的最低版本。

## 理解不同 SDK 的类库默认值

使用`dotnet` SDK 工具创建类库时，了解默认使用的目标框架可能是有用的，如下表所示：

| SDK | 新类库的默认目标框架 |
| --- | --- |
| .NET Core 3.1 | `netstandard2.0` |
| .NET 5 | `net5.0` |
| .NET 6 | `net6.0` |

当然，类库默认针对特定版本的.NET 并不意味着您在使用默认模板创建类库项目后就不能更改它。

您可以手动设置目标框架的值，以支持需要引用该库的项目，如下表所示：

| 类库目标框架 | 可被目标项目使用 |
| --- | --- |
| `netstandard2.0` | .NET Framework 4.6.1 或更高版本，.NET Core 2.0 或更高版本，.NET 5.0 或更高版本，Mono 5.4 或更高版本，Xamarin.Android 8.0 或更高版本，Xamarin.iOS 10.14 或更高版本 |
| `netstandard2.1` | .NET Core 3.0 或更高版本，.NET 5.0 或更高版本，Mono 6.4 或更高版本，Xamarin.Android 10.0 或更高版本，Xamarin.iOS 12.16 或更高版本 |
| `net5.0` | .NET 5.0 或更高版本 |
| `net6.0` | .NET 6.0 或更高版本 |

**良好实践**：始终检查类库的目标框架，然后根据需要手动更改它为更合适的内容。做出有意识的决定，而不是接受默认值。

## 创建一个.NET Standard 2.0 类库

我们将使用.NET Standard 2.0 创建一个类库，以便它可以跨所有重要的.NET 旧版平台和 Windows、macOS 和 Linux 操作系统上进行跨平台使用，同时还可以访问广泛的.NET API 集：

1.  使用您喜欢的代码编辑器向`Chapter07`解决方案/工作区添加一个名为`SharedLibrary`的新类库。

1.  如果您使用 Visual Studio 2022，在提示**目标框架**时，选择**.NET Standard 2.0**，然后将解决方案的启动项目设置为当前选择。

1.  如果您使用 Visual Studio Code，包括一个切换以针对.NET Standard 2.0，如下命令所示：

```cs

dotnet new classlib -f netstandard2.0

```

1.  如果您使用 Visual Studio Code，请将`SharedLibrary`选择为活动的 OmniSharp 项目。

**良好实践**：如果您需要创建使用.NET 6.0 新功能的类型，以及仅使用.NET Standard 2.0 功能的类型，那么可以创建两个单独的类库：一个针对.NET Standard 2.0，一个针对.NET 6.0。您将在*第十章*，*使用 Entity Framework Core 处理数据*中看到这一点。

手动创建两个类库的替代方法是创建一个支持多目标的类库。如果您希望我在下一版中添加一个关于多目标的部分，请告诉我。您可以在这里阅读有关多目标的信息：[`docs.microsoft.com/en-us/dotnet/standard/library-guidance/cross-platform-targeting#multi-targeting`](https://docs.microsoft.com/en-us/dotnet/standard/library-guidance/cross-platform-targeting#multi-targeting)。

## 控制.NET SDK

默认情况下，执行`dotnet`命令会使用最近安装的.NET SDK。有时您可能希望控制使用哪个 SDK。

例如，第四版的一位读者希望他们的体验与使用.NET Core 3.1 SDK 的书中步骤相匹配。但他们还安装了.NET 5.0 SDK，并且默认情况下正在使用。如前一节所述，创建新类库时的行为已更改为以.NET 5.0 为目标，而不是.NET Standard 2.0，这让读者感到困惑。

您可以使用`global.json`文件来控制默认使用的.NET SDK。`dotnet`命令会在当前文件夹和祖先文件夹中搜索`global.json`文件。

1.  在`Chapter07`文件夹中创建一个名为`ControlSDK`的子目录/文件夹。

1.  在 Windows 上，启动**命令提示符**或**Windows 终端**。在 macOS 上，启动**终端**。如果您使用 Visual Studio Code，则可以使用集成终端。

1.  在`ControlSDK`文件夹中，在命令提示符或终端中，输入一个命令来创建一个`global.json`文件，强制使用最新的.NET Core 3.1 SDK，如下所示的命令：

```cs

dotnet new globaljson --sdk-version 3.1.412

```

1.  打开`global.json`文件并查看其内容，如下标记所示：

```cs

{

"sdk"

: {

"版本"

: "3.1.412"

}

```

```

您可以在以下链接的表格中发现最新.NET SDK 的版本号：[`dotnet.microsoft.com/download/visual-studio-sdks`](https://dotnet.microsoft.com/download/visual-studio-sdks)

1.  在`ControlSDK`文件夹中，在命令提示符或终端中，输入一个命令来创建一个类库项目，如下所示的命令：

```cs

dotnet new classlib

```

1.  如果您尚未安装.NET Core 3.1 SDK，则将看到一个错误，如下所示的输出：

```cs

无法执行，因为未找到应用程序或未安装兼容的.NET SDK。

```

1.  如果您已经安装了.NET Core 3.1 SDK，则将默认创建一个以.NET Standard 2.0 为目标的类库项目。

您不需要完成上述步骤，但如果您想尝试并且尚未安装.NET Core 3.1 SDK，则可以从以下链接安装：

[`dotnet.microsoft.com/download/dotnet/3.1`](https://dotnet.microsoft.com/download/dotnet/3.1)

# 发布您的代码以进行部署

如果您写了一本小说，希望其他人阅读，您必须将其发布。

大多数开发人员编写代码供其他开发人员在其自己的代码中使用，或供用户作为应用程序运行。为此，您必须将代码发布为打包的类库或可执行应用程序。

有三种方法可以发布和部署.NET 应用程序。它们是：

1.  **Framework-dependent deployment**（**FDD**）。

1.  **Framework-dependent executables**（**FDEs**）。

1.  Self-contained.

如果选择部署应用程序及其包依赖项，但不包括.NET 本身，则依赖于目标计算机上已经安装了.NET。这对于部署到服务器的 Web 应用程序非常有效，因为.NET 和许多其他 Web 应用程序很可能已经在服务器上。

**Framework-dependent deployment**（**FDD**）意味着您部署一个必须由`dotnet`命令行工具执行的 DLL。**Framework-dependent executables**（**FDE**）意味着您部署一个可以直接从命令行运行的 EXE。两者都需要系统上已经安装了.NET。

有时，您希望能够给某人一个包含您的应用程序的 USB 硬盘，并知道它可以在他们的计算机上执行。您希望执行自包含部署。虽然部署文件的大小会更大，但您会知道它可以工作。

## 创建一个控制台应用程序进行发布

让我们探索如何发布一个控制台应用程序：

1.  使用您喜欢的代码编辑器将一个名为`DotNetEverywhere`的新控制台应用添加到`Chapter07`解决方案/工作区中。

1.  在 Visual Studio Code 中，选择`DotNetEverywhere`作为活动的 OmniSharp 项目。当您看到弹出的警告消息说缺少所需的资源时，点击**是**进行添加。

1.  在`Program.cs`中，删除注释并静态导入`Console`类。

1.  在`Program.cs`中，添加一个语句输出一个消息，表明控制台应用可以在任何地方运行，并提供一些关于操作系统的信息，如下所示的代码：

```cs

WriteLine("我可以在任何地方运行！"

);

WriteLine($"操作系统版本为

{Environment.OSVersion}

。"

);

if

(OperatingSystem.IsMacOS())

{

WriteLine("我是 macOS。"

);

}

else

if

(OperatingSystem.IsWindowsVersionAtLeast(major: 10

))

{

WriteLine("我是 Windows 10 或 11。"

);

}

else

{

WriteLine("我是其他神秘的操作系统。"

);

}

WriteLine("按 ENTER 停止我。"

);

ReadLine();

```

1.  打开`DotNetEverywhere.csproj`，并在`<PropertyGroup>`元素中添加运行时标识，如下所示：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

<OutputType>Exe</OutputType>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<ImplicitUsings>enable</ImplicitUsings>

**<RuntimeIdentifiers>**

**win10-x64;osx-x64;osx**

**.11.0**

**-arm64;linux-x64;linux-arm64**

**</RuntimeIdentifiers>**

</PropertyGroup>

</Project>

```

+   `win10-x64` RID 值表示 Windows 10 或 Windows Server 2016 64 位。您还可以使用`win10-arm64` RID 值部署到 Microsoft Surface Pro X。

+   `osx-x64` RID 值表示 macOS Sierra 10.12 或更高版本。您还可以指定特定版本的 RID 值，如`osx.10.15-x64`（Catalina）、`osx.11.0-x64`（Big Sur on Intel）或`osx.11.0-arm64`（Big Sur on Apple Silicon）。

+   `linux-x64` RID 值表示大多数桌面 Linux 发行版，如 Ubuntu、CentOS、Debian 或 Fedora。对于 Raspbian 或 Raspberry Pi OS 32 位，请使用`linux-arm`。对于运行 Ubuntu 64 位的 Raspberry Pi，请使用`linux-arm64`。

## 了解 dotnet 命令

当您安装.NET SDK 时，它包括一个名为`dotnet`的**命令行界面（CLI）**。

### 创建新项目

.NET CLI 有命令可以在当前文件夹上工作，使用模板创建新项目：

1.  在 Windows 上，启动**命令提示符**或**Windows 终端**。在 macOS 上，启动**终端**。如果您正在使用 Visual Studio Code，则可以使用集成终端。

1.  输入`dotnet new --list`或`dotnet new -l`命令，列出当前安装的模板，如*图 7.2*所示：![包含文本描述的图片](img/Image00077.jpg)

图 7.2：已安装的 dotnet new 项目模板列表

大多数`dotnet`命令行开关都有长版本和短版本。例如，`--list`或`-l`。短的更快，但更容易被您或其他人误解。有时候更多的输入更清晰。

## 获取有关.NET 及其环境的信息

查看当前安装的.NET SDK 和运行时，以及操作系统的信息，如下所示的命令非常有用：

```cs

dotnet --info

```

请注意结果，如下所示的部分输出：

```cs

.NET SDK（反映任何 global.json）：

版本：   6.0.100

Commit:    22d70b47bc

运行时环境：

操作系统名称：Windows

操作系统版本：10.0.19043

操作系统平台：Windows

RID:         win10-x64

基本路径：C:\Program Files\dotnet\sdk\6.0.100\

主机（用于支持）：

版本：6.0.0

Commit:  91ba01788d

已安装的.NET SDK：

3.1.412 [C:\Program Files\dotnet\sdk]

5.0.400 [C:\Program Files\dotnet\sdk]

6.0.100 [C:\Program Files\dotnet\sdk]

已安装.NET 运行时：

Microsoft.AspNetCore.All 2.1.29 [...\dotnet\shared\Microsoft.AspNetCore.All]

...

```

## 管理项目

.NET CLI 具有以下命令，可在当前文件夹中管理项目：

+   `dotnet restore`：这将为项目下载依赖项。

+   `dotnet build`：这将构建，也就是编译，项目。

+   `dotnet test`：这将为项目构建然后运行单元测试。

+   `dotnet run`：这将构建然后运行项目。

+   `dotnet pack`：为项目创建一个 NuGet 包。

+   `dotnet publish`：这将构建然后发布项目，可以是带有依赖项的，也可以是独立的应用程序。

+   `dotnet add`：向项目添加对包或类库的引用。

+   `dotnet remove`：这将从项目中删除对包或类库的引用。

+   `dotnet list`：列出项目的包或类库引用。

## 发布一个独立的应用程序

现在您已经看到了一些示例`dotnet`工具命令，我们可以发布我们的跨平台控制台应用程序：

1.  在命令行中，确保您在`DotNetEverywhere`文件夹中。

1.  输入一个命令来构建和发布 Windows 10 的控制台应用程序的发布版本，如下命令所示：

```cs

dotnet publish -c Release -r win10-x64

```

1.  请注意，构建引擎会还原所需的包，将项目源代码编译成程序集 DLL，并创建一个`publish`文件夹，如下输出所示：

```cs

Microsoft（R）构建引擎版本 17.0.0+073022eb4 for .NET

版权所有（C）Microsoft Corporation。保留所有权利。

确定要还原的项目...

已恢复 C:\Code\Chapter07\DotNetEverywhere\DotNetEverywhere.csproj（46.89 秒）。

DotNetEverywhere -> C:\Code\Chapter07\DotNetEverywhere\bin\Release\net6.0\win10-x64\DotNetEverywhere.dll

DotNetEverywhere -> C:\Code\Chapter07\DotNetEverywhere\bin\Release\net6.0\win10-x64\publish\

```

1.  输入命令以构建和发布 macOS 和 Linux 变体的发布版本，如下命令所示：

```cs

dotnet publish -c Release -r osx-x64

dotnet publish -c Release -r osx.11.0-arm64

dotnet publish -c Release -r linux-x64

dotnet publish -c Release -r linux-arm64

```

**良好实践**：您可以使用脚本语言（如 PowerShell）自动化这些命令，并在任何操作系统上使用跨平台 PowerShell Core 执行它。只需创建一个扩展名为`.ps1`的文件，其中包含这五个命令。然后执行该文件。在以下链接了解有关 PowerShell 的更多信息：[`github.com/markjprice/cs10dotnet6/tree/main/docs/powershell`](https://github.com/markjprice/cs10dotnet6/tree/main/docs/powershell)

1.  打开 macOS **Finder**窗口或 Windows **文件资源管理器**，导航到`DotNetEverywhere\bin\Release\net6.0`，并注意各种操作系统的输出文件夹。

1.  在`win10-x64`文件夹中，选择`publish`文件夹，注意所有支持的程序集，如`Microsoft.CSharp.dll`。

1.  选择`DotNetEverywhere`可执行文件，并注意它的大小为 161 KB，如*图 7.3*所示：![自动生成的图形用户界面描述](img/Image00078.jpg)

图 7.3：Windows 10 64 位的 DotNetEverywhere 可执行文件

1.  如果您在 Windows 上，则双击执行该程序，并注意结果，如下输出所示：

```cs

我可以在任何地方运行！

操作系统版本为 Microsoft Windows NT 10.0.19042.0。

我是 Windows 10。

按 ENTER 键停止我。

```

1.  请注意`publish`文件夹及其所有文件的总大小为 64.8 MB。

1.  在`osx.11.0-arm64`文件夹中，选择`publish`文件夹，注意所有支持的程序集，然后选择`DotNetEverywhere`可执行文件，并注意可执行文件为 126 KB，`publish`文件夹为 71.8 MB。

如果您将这些`publish`文件夹中的任何一个复制到适当的操作系统上，控制台应用程序将运行；这是因为它是一个自包含的可部署.NET 应用程序。例如，在具有英特尔处理器的 macOS 上，如下输出所示：

```cs

我可以在任何地方运行！

操作系统版本是 Unix 11.2.3

我是 macOS。

按 ENTER 键停止我。

```

这个例子使用了一个控制台应用程序，但您也可以很容易地创建一个 ASP.NET Core 网站或 Web 服务，或者一个 Windows Forms 或 WPF 应用程序。当然，您只能将 Windows 桌面应用程序部署到 Windows 计算机上，而不能部署到 Linux 或 macOS。

## 发布单文件应用程序

要发布为“单一”文件，您可以在发布时指定标志。在.NET 5 中，单文件应用程序主要集中在 Linux 上，因为在 Windows 和 macOS 上都存在限制，这意味着真正的单文件发布在技术上不可能。在.NET 6 中，您现在可以在 Windows 上创建真正的单文件应用程序。

如果您可以假设.NET 6 已经安装在您要运行应用程序的计算机上，那么您可以在发布应用程序时使用额外的标志，以表示它不需要是自包含的，并且您希望将其发布为单个文件（如果可能的话），如下面的命令所示（必须在一行上输入）：

```cs

dotnet publish -r win10-x64 -c Release --self-contained=false

/p:PublishSingleFile=true

```

这将生成两个文件：`DotNetEverywhere.exe`和`DotNetEverywhere.pdb`。`.exe`是可执行文件。`.pdb`文件是一个**程序调试数据库**文件，用于存储调试信息。

在 macOS 上，发布的应用程序没有`.exe`文件扩展名，因此，如果您在上面的命令中使用`osx-x64`，文件名将没有扩展名。

如果您希望将`.pdb`文件嵌入`.exe`文件中，那么请在`.csproj`文件的`<PropertyGroup>`元素中添加一个`<DebugType>`元素，并将其设置为`embedded`，如下标记所示：

```cs

<PropertyGroup>

<OutputType>Exe</OutputType>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<ImplicitUsings>enable</ImplicitUsings>

<RuntimeIdentifiers>

win10-x64;osx-x64;osx.11.0

-arm64;linux-x64;linux-arm64

</RuntimeIdentifiers>

**<DebugType>embedded</DebugType>**

</PropertyGroup>

```

如果您不能假设.NET 6 已经安装在计算机上，那么尽管 Linux 也只生成两个文件，但是对于 Windows，还会有以下额外的文件：`coreclr.dll`，`clrjit.dll`，`clrcompression.dll`和`mscordaccore.dll`。

让我们看一个 Windows 的例子：

1.  在命令行中，输入命令以构建 Windows 10 的控制台应用程序的发布版本，如下命令所示：

```cs

dotnet publish -c Release -r win10-x64 /p:PublishSingleFile=true```

```

1.  导航到`DotNetEverywhere\bin\Release\net6.0\win10-x64\publish`文件夹，选择`DotNetEverywhere`可执行文件，并注意可执行文件现在为 58.3 MB，还有一个大小为 10 KB 的`.pdb`文件。您的系统上的大小会有所不同。

## 使用应用程序修剪减小应用程序的大小

将.NET 应用程序部署为独立应用程序的问题之一是.NET 库占用了大量空间。减小尺寸的最大需求之一是 Blazor WebAssembly 组件，因为所有.NET 库都需要下载到浏览器中。

幸运的是，您可以通过不将未使用的程序集与部署一起打包来减小这个大小。从.NET Core 3.0 开始，应用程序修剪系统可以识别代码所需的程序集，并删除不需要的程序集。

在.NET 5 中，修剪进一步去除了单个类型，甚至是程序集中的方法等成员，如果它们没有被使用。例如，对于一个 Hello World 控制台应用程序，`System.Console.dll`程序集从 61.5 KB 减小到 31.5 KB。对于.NET 5，这是一个实验性功能，因此默认情况下是禁用的。

在.NET 6 中，微软为其库添加了注释，指示它们如何安全地修剪，因此类型和成员的修剪成为默认设置。这被称为**链接修剪模式**。

关键是修剪如何识别未使用的程序集、类型和成员。如果您的代码是动态的，可能使用反射，那么它可能无法正常工作，因此微软还允许手动控制。

### 启用程序集级修剪

有两种启用程序集级修剪的方法。

第一种方法是在项目文件中添加一个元素，如下标记所示：

```cs

<PublishTrimmed>true

</PublishTrimmed>

```

第二种方法是在发布时添加一个标志，如下命令中突出显示的那样：

```cs

dotnet publish ...

**-p:PublishTrimmed=True**

```

### 启用类型级和成员级修剪

有两种启用类型级和成员级修剪的方法。

第一种方法是在项目文件中添加两个元素，如下标记所示：

```cs

<PublishTrimmed>true

</PublishTrimmed>

<TrimMode>Link</TrimMode>

```

第二种方法是在发布时添加两个标志，如下命令中突出显示的那样：

```cs

dotnet publish ...

**-p:PublishTrimmed=True -p:TrimMode=Link**

```

对于.NET 6，链接修剪模式是默认的，因此您只需要指定开关，如果要设置替代修剪模式，如`copyused`，这意味着程序集级修剪。

# 反编译.NET 程序集

学习如何为.NET 编写代码的最佳方法之一是看专业人士是如何做的。

**良好的做法**：您可以为非学习目的反编译他人的程序集，例如复制他们的代码以在自己的生产库或应用程序中使用，但请记住您正在查看他们的知识产权，所以请尊重。

## 使用 Visual Studio 2022 的 ILSpy 扩展进行反编译

出于学习目的，您可以使用 ILSpy 等工具反编译任何.NET 程序集。

1.  在 Windows 的 Visual Studio 2022 中，导航到**扩展** | **管理扩展**。

1.  在搜索框中输入`ilspy`。

1.  对于**ILSpy**扩展，点击**下载**。

1.  点击**关闭**。

1.  关闭 Visual Studio 以允许扩展安装。

1.  重新启动 Visual Studio 并重新打开`Chapter07`解决方案。

1.  在**解决方案资源管理器**中，右键单击**DotNetEverywhere**项目，然后选择**在 ILSpy 中打开输出**。

1.  导航到**文件** | **打开…**。

1.  导航到以下文件夹：

```cs

Code/Chapter07/DotNetEverywhere/bin/Release/net6.0

/linux-x64

```

1.  选择`System.IO.FileSystem.dll`程序集，然后点击**打开**。

1.  在**程序集**树中，展开**System.IO.FileSystem**程序集，展开**System.IO**命名空间，选择**Directory**类，等待其反编译。

1.  在`Directory`类中，点击**[+]**展开`GetParent`方法，如*图 7.4*所示：![图形用户界面，文本，应用程序描述自动生成](img/Image00079.jpg)

图 7.4：在 Windows 上反编译的 Directory 类的 GetParent 方法

1.  注意检查`path`参数并在其为`null`时抛出`ArgumentNullException`，在其长度为零时抛出`ArgumentException`的良好做法。

1.  关闭 ILSpy。

## 使用 Visual Studio Code 的 ILSpy 扩展进行反编译

类似的功能作为 Visual Studio Code 的扩展跨平台可用。

1.  如果您还没有为 Visual Studio Code 安装**ILSpy .NET 反编译器**扩展，请立即搜索并安装。

1.  在 macOS 或 Linux 上，该扩展依赖于 Mono，因此您还需要从以下链接安装 Mono：[`www.mono-project.com/download/stable/`](https://www.mono-project.com/download/stable/)。

1.  在 Visual Studio Code 中，导航到**视图** | **命令面板…**。

1.  键入`ilspy`，然后选择**ILSpy:反编译 IL 程序集（选择文件）**。

1.  导航到以下文件夹：

```cs

Code/Chapter07/DotNetEverywhere/bin/Release/net6.0

/linux-x64

```

1.  选择`System.IO.FileSystem.dll`程序集，点击**选择程序集**。看起来没有任何反应，但可以通过查看**输出**窗口来确认 ILSpy 正在工作，选择下拉列表中的**ilspy-vscode**，并查看处理过程，如*图 7.5*所示：![图形用户界面，文本，应用程序，电子邮件描述自动生成](img/Image00080.jpg)

图 7.5：选择要反编译的程序集时的 ILSpy 扩展输出

1.  在**资源管理器**中，展开**ILSPY 反编译成员**，选择程序集，关闭**输出**窗口，注意打开的两个编辑窗口，显示使用 C#代码的程序集属性和使用 IL 代码的外部 DLL 和程序集引用，如*图 7.6*所示：![图形用户界面，文本，应用程序描述自动生成](img/Image00081.jpg)

图 7.6：扩展 ILSPY 反编译成员

1.  在右侧的 IL 代码中，注意到对`System.Runtime`程序集的引用，包括版本号，如下所示：

```cs

.module extern

libSystem.Native

.assembly extern

System.Runtime

{

.publickeytoken = (

b0 3f

5f

7f

11

d5 0

a 3

a

)

.ver 6

:0

:0

:0

}

```

`.module extern libSystem.Native`表示该程序集调用 Linux 系统 API，这与与文件系统交互的代码所期望的一样。如果我们反编译了该程序集的 Windows 等效版本，它将使用`.module extern kernel32.dll`，这是 Win32 API。

1.  在**资源管理器**中，在**ILSPY 反编译成员**中，展开程序集，展开**System.IO**命名空间，选择**Directory**，注意打开的两个编辑窗口，左边显示用 C#代码反编译的`Directory`类，右边显示 IL 代码，如*图 7.7*所示：![](img/Image00082.jpg)

图 7.7：C#和 IL 代码中的反编译 Directory 类

1.  比较`GetParent`方法的 C#源代码，如下所示：

```cs

public

static

DirectoryInfo? GetParent(string

path)

{

if

(path == null

)

{

throw

new

ArgumentNullException("path"

);

}

if

(path.Length == 0

)

{

throw

new

ArgumentException(SR.Argument_PathEmpty, "path"

);

}

string

fullPath = Path.GetFullPath(path);

string

directoryName = Path.GetDirectoryName(fullPath);

if

(directoryName == null

)

{

return

null

;

}

返回

new

DirectoryInfo(directoryName);

}

```

1.  具有`GetParent`方法的等效 IL 源代码，如下所示：

```cs

.method /* 06000067 */

public

hidebysig static

class

System.IO.DirectoryInfo

GetParent

(

string

path

) cil

managed

{

.param [0

]

.custom instance void

System.Runtime.CompilerServices

.NullableAttribute::.ctor(uint8) = (

01

00

02

00

00

)

// 方法开始于 RVA 0x62d4

// 代码大小 64（0x40）

.maxstack 2

.locals /* 1100000E */

(

[0

] string

,

[1

] string

)

IL_0000: ldarg.0

IL_0001: brtrue.s IL_000e

IL_0003: ldstr "path"

/* 700005CB */

IL_0008: newobj instance void

[System.Runtime]

System.ArgumentNullException::.ctor(string

) /* 0A000035 */

IL_000d: throw

IL_000e: ldarg.0

IL_000f: callvirt instance int32 [System.Runtime]

System.String::get_Length() /* 0A000022 */

IL_0014: brtrue.s IL_0026

IL_0016: call string

System.SR::get_Argument_PathEmpty() /* 0600004C */

IL_001b: ldstr "path"

/* 700005CB */

IL_0020: newobj instance void

[System.Runtime]

System.ArgumentException::.ctor(string

, string

) /* 0A000036 */

IL_0025: throw

IL_0026: ldarg.0

IL_0027: call string

[System.Runtime.Extensions]

System.IO.Path::GetFullPath(string

) /* 0A000037 */

IL_002c: stloc.0

IL_002d: ldloc.0

IL_002e: call string

[System.Runtime.Extensions]

System.IO.Path::GetDirectoryName(string

) /* 0A000038 */

IL_0033: stloc.1

IL_0034: ldloc.1

IL_0035: brtrue.s IL_0039 IL_0037: ldnull

IL_0038: ret IL_0039: ldloc.1

IL_003a: newobj instance void

System.IO.DirectoryInfo::.ctor(string

) /* 06000097 */

IL_003f: ret

} // end of method Directory::GetParent

```

**良好实践**：IL 代码编辑窗口在你非常精通 C#和.NET 开发时才会有用，因为了解 C#编译器如何将你的源代码转换为 IL 代码可能很重要。更有用的编辑窗口包含了由微软专家编写的等效 C#源代码。你可以从专业人士如何实现类型中学到很多良好的实践。例如，`GetParent` 方法展示了如何检查参数是否为 `null` 和其他参数异常。

1.  关闭不保存更改的编辑窗口。

1.  在 **EXPLORER** 中的 **ILSPY DECOMPILED MEMBERS** 中，右键单击程序集，然后选择 **Unload Assembly** 。

## 不，从技术上讲，你无法阻止反编译

有时候有人问我是否有办法保护编译后的代码以防止反编译。简短的答案是否定的，如果你仔细想想，你就会明白为什么必须这样。你可以使用诸如 **Dotfuscator** 这样的混淆工具来增加难度，但最终你无法完全阻止它。

所有编译后的应用程序都包含对平台、操作系统和硬件的指令。这些指令必须在功能上与原始源代码相同，但对人类来说更难阅读。这些指令必须是可读的才能执行你的代码；因此它们也必须是可读的才能被反编译。如果你使用某种自定义技术保护你的代码免受反编译，那么你也会阻止你的代码运行！

虚拟机模拟硬件，因此可以捕获你的运行应用程序与它认为正在运行的软件和硬件之间的所有交互。

如果你能保护你的代码，那么你也会阻止用调试器附加到它并逐步执行它。如果编译后的应用程序有一个 `pdb` 文件，那么你可以附加调试器并逐行执行语句。即使没有 `pdb` 文件，你仍然可以附加调试器并大致了解代码的工作原理。

这对所有编程语言都是适用的。不仅仅是.NET 语言，如 C#、Visual Basic 和 F#，还有 C、C++、Delphi、汇编语言：所有这些都可以用于调试或者被反汇编或反编译。一些专业人员使用的工具如下表所示：

| 类型 | 产品 | 描述 |
| --- | --- | --- |
| 虚拟机 | VMware | 专业的恶意软件分析师总是在虚拟机中运行软件。 |
| 调试器 | SoftICE | 通常在虚拟机中运行在操作系统下方。 |
| 调试器 | WinDbg | 用于理解 Windows 内部的工具，因为它对 Windows 数据结构的了解比其他调试器更多。 |
| 反汇编器 | IDA Pro | 由专业恶意软件分析师使用。 |
| 反编译器 | HexRays | 反编译 C 应用程序。IDA Pro 的插件。 |
| 反编译器 | DeDe | 反编译 Delphi 应用程序。 |
| 反编译器 | dotPeek | 来自 JetBrains 的.NET 反编译器。 |

**良好实践**：调试、反汇编和反编译他人的软件可能违反其许可协议，在许多司法管辖区都是非法的。与其试图用技术解决方案保护你的知识产权，法律有时是你唯一的救济。

# 为 NuGet 分发打包你的库

在学习如何创建和打包我们自己的库之前，我们将回顾项目如何使用现有的包。

## 引用 NuGet 包

假设你想要添加一个由第三方开发者创建的包，例如 `Newtonsoft.Json`，这是一个用于处理 JavaScript 对象表示（JSON）格式的流行包：

1.  在 `AssembliesAndNamespaces` 项目中，使用 Visual Studio 2022 的 GUI 或 Visual Studio Code 的 `dotnet add package` 命令添加对 `Newtonsoft.Json` NuGet 包的引用。

1.  打开 `AssembliesAndNamespaces.csproj` 文件，并注意已添加了一个包引用，如下标记所示：

```cs

<ItemGroup>

<PackageReference Include="newtonsoft.json"

Version="13.0.1"

/>

</ItemGroup>

```

如果您有一个更新版本的`newtonsoft.json`软件包，那么自本章编写以来它已经更新。

### 修复依赖项

为了始终恢复软件包并编写可靠的代码，重要的是**修复依赖项**。修复依赖项意味着您正在使用针对特定版本的.NET 发布的相同软件包系列，例如.NET 6.0 的 SQLite，如下面标记中突出显示的那样：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

<OutputType>Exe</OutputType>

<TargetFramework>net6.0

</TargetFramework>

</PropertyGroup>

<ItemGroup>

**<PackageReference**

**包括=**

**"Microsoft.EntityFrameworkCore.Sqlite"**

**版本=**

**"6.0.0"**

**/>**

</ItemGroup>

</Project>

```

为了修复依赖项，每个软件包都应该有一个单独的版本，没有额外的修饰符。额外的修饰符包括 beta（`beta1`），发布候选版（`rc4`）和通配符（`*`）。

通配符允许将来的版本自动引用和使用，因为它们始终代表最新版本。但是通配符因此是危险的，因为它们可能导致使用未来不兼容的软件包，从而破坏您的代码。

这在编写一本书时可能是值得冒的风险，因为每个月都会发布新的预览版本，您不希望不断更新软件包引用，就像我在 2021 年所做的那样，如下面的标记所示：

```cs

<PackageReference

Include="Microsoft.EntityFrameworkCore.Sqlite"

Version="6.0.0-preview.*"

/>

```

如果您使用`dotnet add package`命令，或者使用 Visual Studio 的**管理 NuGet 软件包**，它将默认使用软件包的最新特定版本。但是，如果您从博客文章中复制并粘贴配置，或者手动添加引用，可能会包含通配符修饰符。

以下依赖项是 NuGet 软件包引用的示例，这些软件包引用*不*是固定的，因此应该避免使用，除非您知道其影响：

```cs

<PackageReference Include="System.Net.Http"

Version="4.1.0-*"

/>

<PackageReference Include="Newtonsoft.Json"

Version="12.0.3-beta1"

/>

```

**良好的实践**：微软保证，如果您将依赖项固定到特定版本的.NET，例如 6.0.0，这些软件包将一起工作。几乎总是要修复您的依赖项。

## 为 NuGet 打包一个库

现在，让我们打包之前创建的`SharedLibrary`项目：

1.  在`SharedLibrary`项目中，将`Class1.cs`文件重命名为`StringExtensions.cs`。

1.  修改其内容以提供一些有用的扩展方法，用于使用正则表达式验证各种文本值，如下面的代码所示：

```cs

使用

System.Text.RegularExpressions;

命名空间

Packt.Shared

{

公共

静态

类

StringExtensions

{

公共

静态

布尔

IsValidXmlTag

(

这

字符串

输入

)

{

返回

Regex.IsMatch(input,

@"^<([a-z]+)([^<]+)*(?:>(.*)<\/\1>|\s+\/>)$"

);

}

公共

静态

布尔

IsValidPassword

(

这

字符串

输入

)

{

// 最少八个有效字符

返回

Regex.IsMatch(input, "^[a-zA-Z0-9_-]{8,}$"

);

}

公共

静态

布尔

IsValidHex

(

这

字符串

输入

)

{

// 三个或六个有效的十六进制数字字符

返回

Regex.IsMatch(input,

"^#?([a-fA-F0-9]{3}|[a-fA-F0-9]{6})$"

);

}

}

}

```

您将学习如何在*第八章*，*使用常见的.NET 类型*中编写正则表达式。

1.  在`SharedLibrary.csproj`中，修改其内容，如下面标记中突出显示的那样，并注意以下内容：

+   `PackageId`必须是全局唯一的，因此如果要将此 NuGet 软件包发布到[`www.nuget.org/`](https://www.nuget.org/)公共源以供其他人引用和下载，则必须使用不同的值。

+   `PackageLicenseExpression`必须是以下链接中的值：[`spdx.org/licenses/`](https://spdx.org/licenses/)，或者您可以指定自定义许可证。

+   所有其他元素都是不言自明的：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

<TargetFramework>netstandard2.0

</TargetFramework>

**<GeneratePackageOnBuild>**

**true**

**</GeneratePackageOnBuild>**

**<PackageId>Packt.CSdotnet.SharedLibrary</PackageId>**

**<PackageVersion>**

**6.0.0.0**

**</PackageVersion>**

**<Title>C**

**# 10 and .NET 6 Shared Library</Title>**

**<Authors>Mark J Price</Authors>**

**<PackageLicenseExpression>**

**MS-PL**

**</PackageLicenseExpression>**

**<PackageProjectUrl>**

**https:**

**//github.com/markjprice/cs10dotnet6**

**</PackageProjectUrl>**

**<PackageIcon>packt-csdotnet-sharedlibrary.png</PackageIcon>**

**<PackageRequireLicenseAcceptance>**

**true**

**</PackageRequireLicenseAcceptance>**

**<PackageReleaseNotes>**

**示例共享库打包**

**for**

**NuGet.**

**</PackageReleaseNotes>**

**<Description>**

**验证三个扩展方法**

**string**

**value**

**.**

**</Description>**

**<Copyright>**

**版权所有 ©**

**2016-2021**

**Packt Publishing Limited**

**</Copyright>**

**<PackageTags>**

**string**

**扩展包 csharp dotnet packt**

</PropertyGroup>

**<ItemGroup>**

**<None Include=**

**"packt-csdotnet-sharedlibrary.png"**

**>**

**<Pack>True</Pack>**

**<PackagePath></PackagePath>**

**</None>**

**</ItemGroup>**

</Project>

```

**最佳实践**：配置属性值为`true`或`false`值不能有任何空格，因此`<PackageRequireLicenseAcceptance>`条目不能有换行和缩进，如前面的标记所示。

1.  下载图标文件，并将其保存在以下链接的`SharedLibrary`文件夹中：[`github.com/markjprice/cs10dotnet6/blob/main/vs4win/Chapter07/SharedLibrary/packt-csdotnet-sharedlibrary.png`](https://github.com/markjprice/cs10dotnet6/blob/main/vs4win/Chapter07/SharedLibrary/packt-csdotnet-sharedlibrary.png) 。

1.  构建发布程序集：

1.  在 Visual Studio 中，选择工具栏中的**Release**，然后转到**生成** | **生成 SharedLibrary** 。

1.  在 Visual Studio Code 中，在**终端**中，输入`dotnet build -c Release`

1.  如果我们没有在项目文件中将`<GeneratePackageOnBuild>`设置为`true`，那么我们将不得不手动创建 NuGet 包，使用以下附加步骤：

1.  在 Visual Studio 中，转到**生成** | **打包 SharedLibrary**。

1.  在 Visual Studio Code 中，在**终端**中，输入`dotnet pack -c Release`。

### 将包发布到公共 NuGet feed

如果您希望每个人都能够下载和使用您的 NuGet 包，则必须将其上传到公共 NuGet feed，如 Microsoft 的：

1.  启动您最喜欢的浏览器，转到以下链接：[`www.nuget.org/packages/manage/upload`](https://www.nuget.org/packages/manage/upload) 。

1.  如果您希望为其他开发人员上传 NuGet 包作为依赖包的引用，则需要使用 Microsoft 帐户登录[`www.nuget.org/`](https://www.nuget.org/)。

1.  单击**浏览...**，选择通过生成 NuGet 包创建的`.nupkg`文件。文件夹路径应为`Code\Chapter07\SharedLibrary\bin\Release`，文件名为`Packt.CSdotnet.SharedLibrary.6.0.0.nupkg`。

1.  验证`SharedLibrary.csproj`文件中输入的信息是否填写正确，然后单击**提交**。

1.  等待几秒钟，您将看到一个成功消息，显示您的包已上传，如*图 7.8*所示：![](img/Image00083.jpg)

图 7.8：NuGet 包上传消息

**最佳实践**：如果出现错误，请检查项目文件中的错误，或者阅读有关`PackageReference`格式的更多信息，网址为[`docs.microsoft.com/en-us/nuget/reference/msbuild-targets`](https://docs.microsoft.com/en-us/nuget/reference/msbuild-targets)。

### 将包发布到私有 NuGet feed

组织可以托管自己的私有 NuGet feed。这对于许多开发团队共享工作可能是一个方便的方式。您可以在以下链接阅读更多信息：

[`docs.microsoft.com/en-us/nuget/hosting-packages/overview`](https://docs.microsoft.com/en-us/nuget/hosting-packages/overview)

## 使用工具探索 NuGet 包

Uno Platform 创建了一个名为**NuGet Package Explorer**的方便工具，用于打开和查看有关 NuGet 包的更多详细信息。除了作为网站外，它还可以作为跨平台应用程序安装。让我们看看它能做什么：

1.  启动您喜欢的浏览器，然后导航到以下链接：[`nuget.info`](https://nuget.info)。

1.  在搜索框中输入`Packt.CSdotnet.SharedLibrary`。

1.  选择由 Mark J Price 发布的 v6.0.0 包，然后点击“打开”按钮。

1.  在**内容**部分，展开**lib**文件夹和**netstandard2.0**文件夹。

1.  选择**SharedLibrary.dll**，并注意详细信息，如*图 7.9*所示：![](img/Image00084.jpg)

图 7.9：使用 Uno Platform 的 NuGet Package Explorer 探索我的包

1.  如果您将来想在本地使用此工具，请单击浏览器中的安装按钮。

1.  关闭您的浏览器。

并非所有浏览器都支持像这样安装 Web 应用程序。我建议使用 Chrome 进行测试和开发。

## 测试您的类库包

现在，您将通过在 AssembliesAndNamespaces 项目中引用它来测试您上传的包：

1.  在`AssembliesAndNamespaces`项目中，添加对您（或我的）包的引用，如下面标记中所示：

```cs

<ItemGroup>

<PackageReference Include="newtonsoft.json"

Version="13.0.1"

/>

<PackageReference Include=

**"packt.csdotnet.sharedlibrary"**

**Version=**

**"6.0.0"**

**/>**

</ItemGroup>

```

1.  构建控制台应用程序。

1.  在`Program.cs`中，导入`Packt.Shared`命名空间。

1.  在`Program.cs`中，提示用户输入一些`string`值，然后使用包中的扩展方法进行验证，如下面的代码所示：

```cs

Write("输入十六进制颜色值："

）；

字符串

? hex = ReadLine(); // 或 "00ffc8"

WriteLine("Is {0} a valid color value? {1}"

，

arg0: hex, arg1: hex.IsValidHex());

Write("输入 XML 元素："

）；

字符串

? xmlTag = ReadLine(); // 或 "<h1 class=\"<\" />"

WriteLine("Is {0} a valid XML element? {1}"

，

arg0: xmlTag, arg1: xmlTag.IsValidXmlTag());

Write("输入密码："

）；

字符串

? password = ReadLine(); // 或 "secretsauce"

WriteLine("Is {0} a valid password? {1}"

，

arg0: password, arg1: password.IsValidPassword());

```

1.  运行代码，按提示输入一些值，并查看结果，如下面的输出所示：

```cs

输入十六进制颜色值：00ffc8

00ffc8 是有效的颜色值吗？True

输入一个 XML 元素：<h1 class="<" />

<h1 class="<" />是有效的 XML 元素吗？False

输入密码：secretsauce

secretsauce 是有效的密码吗？True

```

# 从.NET Framework 迁移到现代.NET

如果您是现有的.NET Framework 开发人员，则可能有现有的应用程序，您认为应该将其迁移到现代.NET。但是，您应该仔细考虑迁移是否是您代码的正确选择，因为有时，最好的选择并不是迁移。

例如，您可能有一个复杂的网站项目，运行在.NET Framework 4.8 上，但只被少数用户访问。如果它能够在最低硬件上运行并处理访问者流量，那么花费几个月的时间将其迁移到.NET 6 可能是浪费时间。但是，如果该网站目前需要许多昂贵的 Windows 服务器，那么如果您可以迁移到更少、成本更低的 Linux 服务器，迁移的成本最终可能会得到回报。

## 您能够迁移吗？

现代.NET 在 Windows、macOS 和 Linux 上对以下类型的应用程序有很好的支持，因此它们是迁移的良好候选者：

+   ASP.NET Core MVC 网站。

+   ASP.NET Core Web API Web 服务（REST/HTTP）。

+   ASP.NET Core SignalR 服务。

+   **控制台应用程序**命令行界面。

现代.NET 在 Windows 上对以下类型的应用程序有良好的支持，因此它们是迁移的潜在候选者：

+   **Windows Forms**应用程序。

+   **Windows Presentation Foundation**（**WPF**）应用程序。

现代.NET 对跨平台桌面和移动设备上的以下类型的应用程序有很好的支持：

+   **Xamarin**应用程序适用于移动 iOS 和 Android。

+   **.NET MAUI** 用于桌面 Windows 和 macOS，或移动 iOS 和 Android。

现代.NET 不支持以下类型的传统 Microsoft 项目：

+   **ASP.NET Web Forms**网站。这些可能最好使用**ASP.NET Core Razor Pages**或**Blazor**重新实现。

+   **Windows Communication Foundation**（**WCF**）服务（但有一个名为**CoreWCF**的开源项目，根据需求可能可以使用）。WCF 服务可能最好使用**ASP.NET Core gRPC**服务重新实现。

+   **Silverlight**应用程序。这些可能最好使用**.NET MAUI**重新实现。

Silverlight 和 ASP.NET Web Forms 应用程序永远无法迁移到现代.NET，但现有的 Windows Forms 和 WPF 应用程序可以迁移到 Windows 上的.NET，以从新的 API 和更快的性能中受益。

目前在.NET Framework 上的传统 ASP.NET MVC Web 应用程序和 ASP.NET Web API Web 服务可以迁移到现代.NET，然后托管在 Windows、Linux 或 macOS 上。

## 是否应该迁移？

即使您*可以*迁移，*应该*吗？您能获得什么好处？一些常见的好处包括以下内容：

+   **部署到 Linux、Docker 或 Kubernetes 的网站和 Web 服务**：这些操作系统作为网站和 Web 服务平台轻量且具有成本效益，特别是与更昂贵的 Windows Server 相比。

+   不依赖于 IIS 和 System.Web.dll 的移除：即使您继续部署到 Windows Server，ASP.NET Core 也可以托管在轻量级、性能更高的 Kestrel（或其他）Web 服务器上。

+   **命令行工具**：开发人员和管理员用来自动化其任务的工具通常构建为控制台应用程序。跨平台运行单个工具的能力非常有用。

## .NET Framework 和现代.NET 之间的区别

如下表所示，有三个关键区别：

| 现代.NET | .NET Framework |
| --- | --- |
| 分发为 NuGet 包，因此每个应用程序都可以使用其自己的应用程序本地副本部署所需的.NET 版本。 | 分发为系统范围的共享程序集（实际上是在全局程序集缓存（GAC）中）。 |
| 拆分为小型、分层组件，以便执行最小部署。 | 单一的、单块的部署。 |
| 删除旧技术，如 ASP.NET Web Forms，以及非跨平台功能，如 AppDomains、.NET Remoting 和二进制序列化。 | 以及现代.NET 中类似的一些技术，如 ASP.NET Core MVC，它还保留了一些旧技术，如 ASP.NET Web Forms。 |

## 了解.NET 可移植性分析器

微软有一个有用的工具，可以针对您现有的应用程序运行，生成一个迁移报告。您可以在以下链接观看该工具的演示：[`channel9.msdn.com/Blogs/Seth-Juarez/A-Brief-Look-at-the-NET-Portability-Analyzer`](https://channel9.msdn.com/Blogs/Seth-Juarez/A-Brief-Look-at-the-NET-Portability-Analyzer)。

## 了解.NET 升级助手

微软最新的用于将传统项目升级到现代.NET 的工具是.NET 升级助手。

对于我的日常工作，我在一家名为 Optimizely 的公司工作。我们有一个基于.NET Framework 的企业级数字体验平台（DXP），包括内容管理系统（CMS）和用于构建数字商务网站。微软需要一个具有挑战性的迁移项目来设计和测试.NET 升级助手，因此我们与他们合作构建了一个很棒的工具。

目前，它支持以下.NET Framework 项目类型，以后将添加更多：

+   ASP.NET MVC

+   Windows Forms

+   WPF

+   控制台应用程序

+   类库

它安装为全局`dotnet`工具，如下命令所示：

```cs

dotnet tool install -g upgrade-assistant

```

您可以在以下链接中了解有关此工具及其使用方法的更多信息：

[`docs.microsoft.com/en-us/dotnet/core/porting/upgrade-assistant-overview`](https://docs.microsoft.com/en-us/dotnet/core/porting/upgrade-assistant-overview)

## 使用非.NET 标准库

大多数现有的 NuGet 软件包都可以与现代.NET 一起使用，即使它们没有为.NET 标准或.NET 6 等现代版本编译。如果您发现一个软件包在其[nuget.org](https://www.nuget.org/)网页上没有正式支持.NET 标准，您不必放弃。您应该尝试一下，看看它是否有效。

例如，有一个由 Dialect Software LLC 创建的处理矩阵的自定义集合软件包，文档在以下链接中：

[`www.nuget.org/packages/DialectSoftware.Collections.Matrix/`](https://www.nuget.org/packages/DialectSoftware.Collections.Matrix/)

这个软件包是在 2013 年最后更新的，那时.NET Core 或.NET 6 还不存在，因此这个软件包是为.NET Framework 构建的。只要像这样的程序包只使用.NET 标准中可用的 API，它就可以在现代.NET 项目中使用。

让我们尝试使用它，看看它是否有效：

1.  在`AssembliesAndNamespaces`项目中，添加 Dialect Software 软件包的包引用，如下面的标记所示：

```cs

<PackageReference

Include="dialectsoftware.collections.matrix"

Version="1.0.0"

/>

```

1.  构建`AssembliesAndNamespaces`项目以恢复包。

1.  在`Program.cs`中，添加语句以导入`DialectSoftware.Collections`和`DialectSoftware.Collections.Generics`命名空间。

1.  添加语句以创建`Axis`和`Matrix<T>`的实例，用值填充它们，并输出它们，如下面的代码所示：

```cs

Axis x = new

("x"

, 0

, 10

, 1

);

Axis y = new

("y"

, 0

, 4

, 1

);

Matrix<long

> matrix = new

（新

[] { x, y });

for

（int

i = 0

; i < matrix.Axes[0

].Points.Length; i++)

{

matrix.Axes[0

].Points[i].Label = "x"

+ i.ToString();

}

for

（int

i = 0

; i < matrix.Axes[1

].Points.Length; i++)

{

matrix.Axes[1

].Points[i].Label = "y"

+ i.ToString();

}

foreach

（长

[] c in

matrix)

{

matrix[c] = c[0

] + c[1

];

}

foreach

（长

[] c in

matrix)

{

WriteLine("{0},{1} ({2},{3}) = {4}"

,

matrix.Axes[0

].Points[c[0

]].Label，

matrix.Axes[1

].Points[c[1

]].Label，

c[0

]，c[1

]，matrix[c]);

}

```

1.  运行代码，注意警告消息和结果，如下面的输出所示：

```cs

警告 NU1701：软件包'DialectSoftware.Collections.Matrix

1.0.0'已使用'.NETFramework，Version=v4.6.1，

.NETFramework，Version=v4.6.2，.NETFramework，Version=v4.7，

.NETFramework，Version=v4.7.1，.NETFramework，Version=v4.7.2，

.NETFramework，Version=v4.8'而不是项目目标框架'net6.0'。此软件包可能与您的项目不完全兼容。

x0，y0（0,0）= 0

x0，y1（0,1）= 1

x0，y2（0,2）= 2

x0，y3（0,3）= 3

...

```

尽管这个软件包是在.NET 6 存在之前创建的，编译器和运行时无法知道它是否有效，因此会显示警告，但因为它只调用了.NET 标准兼容的 API，所以它有效。

# 使用预览功能

对于微软来说，要交付一些具有跨越.NET 许多部分的影响的新功能是一个挑战，比如运行时、语言编译器和 API 库。这是一个经典的鸡和蛋问题。你先做什么？

从实际角度来看，这意味着尽管微软可能已经完成了大部分需要的功能，但整个功能可能直到他们现在的.NET 发布周期的最后阶段才准备好，对于“野外”中的适当测试来说太晚了。

因此，从.NET 6 开始，微软将在**通用可用性**（**GA**）发布中包含预览功能。开发人员可以选择这些预览功能，并向微软提供反馈。在以后的 GA 发布中，它们可以为所有人启用。

**良好实践**：预览功能不支持生产代码。预览功能在最终发布之前可能会有重大变化。启用预览功能需自担风险。

## 需要预览功能

`[RequiresPreviewFeatures]`属性用于指示使用并因此需要关于预览功能的警告的程序集、类型或成员。然后，代码分析器会扫描此程序集，并在需要时生成警告。如果您的代码不使用任何预览功能，则不会看到任何警告。如果您使用任何预览功能，则您的代码应该警告代码的使用者您使用了预览功能。

## 启用预览功能

让我们来看一个在.NET 6 中可用的预览功能的例子，即定义一个带有静态抽象方法的接口的能力：

1.  使用您喜欢的代码编辑器将一个名为`UsingPreviewFeatures`的新控制台应用程序添加到`Chapter07`解决方案/工作空间中。

1.  在 Visual Studio Code 中，将`UsingPreviewFeatures`选择为活动的 OmniSharp 项目。当看到弹出的警告消息说需要的资产丢失时，点击**是**来添加它们。

1.  在项目文件中，添加一个元素来启用预览功能，并添加一个元素来启用预览语言功能，如下面的标记中所示：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

<OutputType>Exe</OutputType>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<ImplicitUsings>enable</ImplicitUsings>

**<EnablePreviewFeatures>**

**true**

**</EnablePreviewFeatures>**

**<LangVersion>preview</LangVersion>**

</PropertyGroup>

</Project>

```

1.  在`Program.cs`中，删除注释并静态导入`Console`类。

1.  添加语句来定义一个带有静态抽象方法的接口，一个实现它的类，然后在顶层程序中调用该方法，如下面的代码所示：

```cs

using

static

System.Console;

Doer.DoSomething();

public

接口

IWithStaticAbstract

{

static

抽象

void

DoSomething

()

;

}

public

类

Doer

: IWithStaticAbstract

{

public

static

void

DoSomething

()

{

WriteLine("我是一个静态抽象方法的实现。"

);

}

}

```

1.  运行控制台应用程序，并注意它是否正确输出。

## 通用数学

为什么微软添加了定义静态抽象方法的能力？它们有什么用处？

很长一段时间以来，开发人员一直在要求微软能够在通用类型上使用像*这样的运算符。这将使开发人员能够定义数学方法来执行诸如加法、平均值等操作，而不是必须为他们想要支持的所有数值类型创建大量的重载方法。接口中对静态抽象方法的支持是一项基础功能，它将使通用数学成为可能。

如果您感兴趣，可以在以下链接中阅读更多：

[`devblogs.microsoft.com/dotnet/preview-features-in-net-6-generic-math/`](https://devblogs.microsoft.com/dotnet/preview-features-in-net-6-generic-math/)

# 练习和探索

通过回答一些问题，进行一些动手实践，并深入研究本章主题，来测试你的知识和理解。

## 练习 7.1-测试你的知识

回答以下问题：

1.  命名空间和程序集之间有什么区别？

1.  如何在`.csproj`文件中引用另一个项目？

1.  像 ILSpy 这样的工具有什么好处？

1.  C#中的`float`别名代表哪种.NET 类型？

1.  将应用程序从.NET Framework 移植到.NET 6 时，应该在移植之前运行什么工具，以及可以运行什么工具来执行大部分移植工作？

1.  .NET 应用程序的依赖框架和自包含部署有什么区别？

1.  什么是 RID？

1.  `dotnet pack`和`dotnet publish`命令之间有什么区别？

1.  为.NET Framework 编写的哪些类型的应用程序可以移植到现代.NET？

1.  您能否使用为.NET Framework 编写的软件包与现代.NET 兼容？

## 练习 7.2 – 探索主题

使用以下页面上的链接，了解本章涵盖的主题的更多细节：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-7---understanding-and-packaging-net-types`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-7---understanding-and-packaging-net-types)

## 练习 7.3 – 探索 PowerShell

PowerShell 是微软用于在每个操作系统上自动执行任务的脚本语言。微软建议使用带有 PowerShell 扩展的 Visual Studio Code 来编写 PowerShell 脚本。

由于 PowerShell 是一种独立的广泛语言，本书中没有空间来涵盖它。相反，我在书的 GitHub 存储库上创建了一些补充页面，介绍了一些关键概念并展示了一些示例：

[`github.com/markjprice/cs10dotnet6/tree/main/docs/powershell`](https://github.com/markjprice/cs10dotnet6/tree/main/docs/powershell)

# 总结

在本章中，我们回顾了通往.NET 6 的旅程，探讨了程序集和命名空间之间的关系，看到了发布应用程序以分发到多个操作系统的选项，打包和分发了一个类库，并讨论了迁移现有.NET Framework 代码库的选项。

在下一章中，您将了解一些包含在现代.NET 中的常见基类库类型。
