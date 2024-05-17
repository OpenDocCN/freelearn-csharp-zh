C#中的编码标准和原则

C#中编码标准和原则的主要目标是让程序员通过编写性能更好、更易于维护的代码来提高他们的技能。在本章中，我们将看一些好代码的例子，并对比一些坏代码的例子。这将很好地引出我们为什么需要编码标准、原则和方法的讨论。然后，我们将继续考虑命名、注释和格式化源代码的约定，包括类、方法和变量。

一个大型程序可能相当难以理解和维护。对于初级程序员来说，了解代码及其功能可能是一个令人望而却步的任务。团队可能会发现很难在这样的项目上共同工作。从测试的角度来看，这可能会使事情变得相当困难。因此，我们将看一下如何使用模块化将程序分解为更小的模块，这些模块共同工作以产生一个完全可测试的解决方案，可以同时由多个团队进行开发，并且更容易阅读、理解和文档化。

我们将通过查看一些编程设计准则来结束本章，主要是 KISS、YAGNI、DRY、SOLID 和奥卡姆剃刀。

本章将涵盖以下主题：

+   编码标准、原则和方法的必要性

+   命名约定和方法

+   注释和格式化

+   模块化

+   KISS

+   YAGNI

+   DRY

+   SOLID

+   奥卡姆剃刀

本章的学习目标是让您做到以下几点：

+   了解为什么坏代码会对项目产生负面影响。

+   了解好代码如何积极影响项目。

+   了解编码标准如何改进代码以及如何强制执行它们。

+   了解编码原则如何提高软件质量。

+   了解方法论如何促进清洁代码的开发。

+   实施编码标准。

+   选择假设最少的解决方案。

+   减少代码重复，编写 SOLID 代码。

# 第二章：技术要求

要在本章中处理代码，您需要下载并安装 Visual Studio 2019 社区版或更高版本。可以从[`visualstudio.microsoft.com/`](https://visualstudio.microsoft.com/)下载这个集成开发环境。

您可以在[`github.com/PacktPublishing/Clean-Code-in-C-`](https://github.com/PacktPublishing/Clean-Code-in-C-)[.]找到本书的代码。我已将它们全部放在一个单一的解决方案中，每个章节都是一个解决方案文件夹。您将在相关的章节文件夹中找到每个章节的代码。如果要运行项目，请记得将其分配为启动项目。

# 好代码与坏代码

好代码和坏代码都可以编译。这是要理解的第一件事。要理解的下一件事是，坏代码之所以糟糕是有原因的，同样，好代码之所以好也是有原因的。让我们在下面的比较表中看一些原因：

| **好代码** | **坏代码** |
| --- | --- |
| 适当的缩进。 | 不正确的缩进。 |
| 有意义的注释。 | 陈述显而易见的注释。 |
| API 文档注释。 | 为糟糕的代码辩解的注释。被注释掉的代码行。 |
| 使用命名空间进行适当的组织。 | 使用命名空间进行不适当的组织。 |
| 良好的命名约定。 | 糟糕的命名约定。 |
| 只做一件工作的类。 | 做多个工作的类。 |
| 只做一件事的方法。 | 做很多事情的方法。 |
| 不超过 10 行的方法，最好不超过 4 行。 | 超过 10 行的方法。 |
| 方法不超过两个参数。 | 方法超过两个参数。 |
| 适当使用异常。 | 使用异常来控制程序流程。 |
| 可读性代码。 | 难以阅读的代码。 |
| 松散耦合的代码。 | 紧密耦合的代码。 |
| 高内聚性。 | 低内聚性。 |
| 对象被清理干净。 | 对象被搁置不管。 |
| 避免使用`Finalize()`方法。 | 使用`Finalize()`方法。 |
| 正确的抽象级别。 | 过度工程。 |
| 在大类中使用区域。 | 在大类中缺乏区域。 |
| 封装和信息隐藏。 | 直接暴露信息。 |
| 面向对象的代码。 | 意大利面代码。 |
| 设计模式。 | 设计反模式。 |

这是一个相当详尽的列表，不是吗？在接下来的部分中，我们将看看这些特性以及好代码和坏代码之间的差异如何影响你的代码的性能。

## 糟糕的代码

现在我们将简要介绍我们之前列出的每个不良编码实践，具体说明这些实践如何影响你的代码。

### 不正确的缩进

不正确的缩进可能导致代码变得非常难读，特别是如果方法很大的话。为了让代码易于人类阅读，我们需要正确的缩进。如果代码缺乏正确的缩进，很难看出代码的哪一部分属于哪个块。

默认情况下，Visual Studio 2019 在括号和大括号关闭时会正确格式化和缩进你的代码。但有时，它会错误地格式化代码，以提醒你你写的代码中包含异常。但如果你使用简单的文本编辑器，那么你就必须手动进行格式化。

错误缩进的代码也很耗时，当它本可以很容易避免时，这也是对编程时间的一种沮丧的浪费。让我们看一个简单的代码例子：

```cs
public void DoSomething()
{
for (var i = 0; i < 1000; i++)
{
var productCode = $"PRC000{i}";
//...implementation
}
}
```

前面的代码看起来并不那么好，但它仍然是可读的。但是你添加的代码行数越多，代码就变得越难读。

很容易错过一个闭合括号。如果你的代码没有正确缩进，那么找到缺失的括号就会变得更加困难，因为你很难看出哪个代码块缺少了闭合括号。

### 显而易见的注释

我见过程序员对显而易见的注释感到非常不满，因为他们觉得这些注释是居高临下的。在我参与的编程讨论中，程序员们表示他们不喜欢注释，认为代码应该是自解释的。

我能理解他们的情绪。如果你能像读书一样读懂没有注释的代码，那么这就是一段非常好的代码。如果你已经声明了一个变量是字符串，那为什么还要添加`// string`这样的注释呢？让我们看一个例子：

```cs
public int _value; // This is used for storing integer values.
```

我们知道值通过其`int`类型来保存整数。所以真的没有必要说明显而易见的事情。你所做的只是浪费时间和精力，以及使代码变得混乱。

### 借口糟糕的注释

你可能有一个紧迫的截止日期要满足，但是像`// 我知道这段代码很糟糕，但至少它能工作！`这样的注释真的很糟糕。不要这样做。这显示了缺乏专业精神，可能会让其他程序员感到不满。

如果你真的被迫让某些东西快速运行，那就提出一个重构的工单，并将其作为`// TODO: PBI23154 重构代码以符合公司编码规范`这样的 TODO 注释的一部分。然后你或者其他被分配处理技术债务的开发人员可以接手**产品待办事项**（**PBI**）并重构代码。

这里有另一个例子：

```cs
...
int value = GetDataValue(); // This sometimes causes a divide by zero error. Don't know why!
...
```

这真的很糟糕。好吧，谢谢你告诉我们这里会发生除零错误。但你提出了一个 bug 工单吗？你尝试找出问题并修复它了吗？如果所有正在项目中积极工作的人都不碰那段代码，他们怎么会知道有错误的代码存在呢？

至少你应该在代码中加上一个`// TODO:`注释。这样至少这个注释会出现在任务列表中，开发人员可以收到通知并进行处理。

### 注释掉的代码行

如果你注释掉一些代码来尝试一些东西，那没问题。但是如果你要使用替换代码而不是注释掉的代码，那么在提交之前删除注释掉的代码。一两行注释掉的代码并不那么糟糕。但是当你有很多行注释掉的代码时，它会分散注意力，使代码难以维护；甚至会导致混乱：

```cs
/* No longer used as has been replaced by DoSomethinElse().
public void DoSomething()
{
    // ...implementation...
}
*/
```

为什么？为什么？如果它已经被替换并且不再需要，那就删除它。如果你的代码在版本控制中，并且你需要恢复这个方法，那么你可以随时查看文件的历史记录并恢复这个方法。

### 命名空间的不当组织

在使用命名空间时，不要包含应该放在其他地方的代码。这样会使找到正确的代码变得非常困难甚至不可能，特别是在大型代码库中。让我们看看这个例子：

```cs
namespace MyProject.TextFileMonitor
{
    + public class Program { ... }
    + public class DateTime { ... }
    + public class FileMonitorService { ... }
    + public class Cryptography { ... }
}
```

我们可以看到前面的代码中所有的类都在一个命名空间下。然而，我们有机会添加三个更好地组织这些代码的命名空间：

+   `MyProject.TextFileMonitor.Core`：定义常用成员的核心类将放置在这里，比如我们的`DateTime`类。

+   `MyProject.TextFileMonitor.Services`：所有充当服务的类都将放置在这个命名空间中，比如`FileMonitorService`。

+   `MyProject.TextFileMonitor.Security`：所有与安全相关的类都将放置在这个命名空间中，包括我们示例中的`Cryptography`类。

### 糟糕的命名约定

在 Visual Basic 6 编程时代，我们曾经使用匈牙利命名法。我记得我第一次转到 Visual Basic 1.0 时使用它。现在不再需要使用匈牙利命名法。而且，它会让你的代码看起来很丑。所以，现代的做法是使用`NameLabel`、`NameTextBox`和`SaveButton`，而不是使用`lblName`、`txtName`或`btnSave`这样的名称。

使用晦涩的名称和与代码意图不符的名称会使阅读代码变得相当困难。**ihridx**是什么意思？它的意思是**Human Resources Index**，是一个*整数*。真的！避免使用`mystring`、`myint`和`mymethod`这样的名称。这样的名称真的没有任何意义。

在名称中也不要使用下划线，比如`Bad_Programmer`。这会给开发人员造成视觉压力，并且使代码难以阅读。只需删除下划线。

不要在类级别和方法级别使用相同的代码约定。这会使变量的范围难以确定。变量名称的一个好的约定是对变量名称使用驼峰命名法，比如`alienSpawn`，对方法、类、结构和接口名称使用帕斯卡命名法，比如`EnemySpawnGenerator`。

遵循良好的变量命名约定，你应该通过在成员变量前加下划线来区分局部变量（在构造函数或方法中包含的变量）和成员变量（在构造函数和方法之外的类顶部放置的变量）。我在工作中使用过这种编码约定，它确实非常有效，程序员似乎也喜欢这种约定。

### 做多项工作的类

一个好的类应该只做一件事。一个类连接到数据库，获取数据，操作数据，加载报告，将数据分配给报告，显示报告，保存报告，打印报告和导出报告，这样做的工作太多了。它需要重构为更小、更有组织的类。这样的全面类很难阅读。我个人觉得它们令人望而生畏。如果你遇到这样的类，将功能组织成区域。然后将这些区域中的代码移动到执行一个工作的新类中。

让我们来看一个做多件事情的类的例子：

```cs
public class DbAndFileManager
{
 #region Database Operations

 public void OpenDatabaseConnection() { throw new 
  NotImplementedException(); }
 public void CloseDatabaseConnection() { throw new 
  NotImplementedException(); }
 public int ExecuteSql(string sql) { throw new 
  NotImplementedException(); }
 public SqlDataReader SelectSql(string sql) { throw new 
  NotImplementedException(); }
 public int UpdateSql(string sql) { throw new 
  NotImplementedException(); }
 public int DeleteSql(string sql) { throw new 
  NotImplementedException(); }
 public int InsertSql(string sql) { throw new 
  NotImplementedException(); }

 #endregion

 #region File Operations

 public string ReadText(string filename) { throw new 
  NotImplementedException(); }
 public void WriteText(string filename, string text) { throw new 
  NotImplementedException(); }
 public byte[] ReadFile(string filename) { throw new 
  NotImplementedException(); }
 public void WriteFile(string filename, byte[] binaryData) { throw new 
  NotImplementedException(); }

 #endregion
}
```

正如你在前面的代码中所看到的，这个类做了两件主要的事情：它执行数据库操作和文件操作。现在代码被整齐地组织在正确命名的区域内，用于在类内逻辑上分离代码。但是**单一职责原则**（**SRP**）被打破了。我们需要从重构这段代码开始，将数据库操作分离出来，放到一个名为`DatabaseManager`的自己的类中。

然后，我们将数据库操作从`DbAndFileManager`类中移除，只留下文件操作，然后将`DbAndFileManager`类重命名为`FileManager`。我们还需要考虑每个文件的命名空间，以及是否应该修改它们，使得`DatabaseManager`放在`Data`命名空间中，`FileManager`放在`FileSystem`命名空间中，或者在你的程序中的等价位置。

以下代码是将`DbAndFileManager`类中的数据库代码提取到自己的类中，并放在正确的命名空间中的结果：

```cs
using System;
using System.Data.SqlClient;

namespace CH01_CodingStandardsAndPrinciples.GoodCode.Data
{
    public class DatabaseManager
    {
        #region Database Operations

        public void OpenDatabaseConnection() { throw new 
         NotImplementedException(); }
        public void CloseDatabaseConnection() { throw new 
         NotImplementedException(); }
        public int ExecuteSql(string sql) { throw new 
         NotImplementedException(); }
        public SqlDataReader SelectSql(string sql) { throw new 
         NotImplementedException(); }
        public int UpdateSql(string sql) { throw new 
         NotImplementedException(); }
        public int DeleteSql(string sql) { throw new 
         NotImplementedException(); }
        public int InsertSql(string sql) { throw new 
         NotImplementedException(); }

        #endregion
    }
}
```

文件系统代码的重构结果是`FileSystem`命名空间中的`FileManager`类，如下面的代码所示：

```cs
using System;

namespace CH01_CodingStandardsAndPrinciples.GoodCode.FileSystem
{
    public class FileManager
    {
         #region File Operations

         public string ReadText(string filename) { throw new 
          NotImplementedException(); }
         public void WriteText(string filename, string text) { throw new 
          NotImplementedException(); }
         public byte[] ReadFile(string filename) { throw new 
          NotImplementedException(); }
         public void WriteFile(string filename, byte[] binaryData) { throw 
          new NotImplementedException(); }

         #endregion
    }
}
```

我们已经看到了如何识别做太多事情的类，以及如何将它们重构为只做一件事。现在让我们重复这个过程，看看做很多事情的方法。

### 做很多事情的方法

我发现自己在许多层级的缩进中迷失，这些缩进中做了很多事情。排列组合令人费解。我想重构代码以使维护更容易，但我的前辈禁止了。我清楚地看到，通过将代码分配给不同的方法，该方法可以变得更小。

举个例子。在这个例子中，该方法接受一个字符串。然后对该字符串进行加密和解密。它也很长，这样你就可以看到为什么方法应该保持简短：

```cs
public string security(string plainText)
{
    try
    {
        byte[] encrypted;
        using (AesManaged aes = new AesManaged())
        {
            ICryptoTransform encryptor = aes.CreateEncryptor(Key, IV);
            using (MemoryStream ms = new MemoryStream())
                using (CryptoStream cs = new CryptoStream(ms, encryptor, 
                 CryptoStreamMode.Write))
                {
                    using (StreamWriter sw = new StreamWriter(cs))
                        sw.Write(plainText);
                    encrypted = ms.ToArray();
                }
        }
        Console.WriteLine($"Encrypted data: 
         {System.Text.Encoding.UTF8.GetString(encrypted)}");
        using (AesManaged aesm = new AesManaged())
        {
            ICryptoTransform decryptor = aesm.CreateDecryptor(Key, IV);
            using (MemoryStream ms = new MemoryStream(encrypted))
            {
                using (CryptoStream cs = new CryptoStream(ms, decryptor, 
                 CryptoStreamMode.Read))
                {
                    using (StreamReader reader = new StreamReader(cs))
                        plainText = reader.ReadToEnd();
                }
            }
        }
        Console.WriteLine($"Decrypted data: {plainText}");
    }
    catch (Exception exp)
    {
        Console.WriteLine(exp.Message);
    }
    Console.ReadKey();
    return plainText;
}
```

如你在前面的方法中所看到的，它有 10 行代码，很难阅读。此外，它做了不止一件事。这段代码可以分解为两个分别执行单个任务的方法。一个方法会对字符串进行加密，另一个方法会解密字符串。这很好地说明了为什么方法不应该超过 10 行代码。

### 超过 10 行代码的方法

大方法不易阅读和理解。它们也可能导致非常难以找到的错误。大方法的另一个问题是它们可能会失去原始意图。当你遇到由注释分隔和代码包裹在区域中的大方法时，情况会变得更糟。

如果你必须滚动阅读一个方法，那么它就太长了，可能会导致程序员的压力和误解。这反过来可能会导致修改破坏代码或意图，或者两者都会。方法应该尽可能小。但是需要行使常识，因为你可以将小方法的问题推到*第 n*度，直到它变得过分。获得正确平衡的关键是确保方法的意图非常清晰和简洁地实现。

前面的代码是为什么你应该保持方法简短的一个很好的例子。小方法易于阅读和理解。通常，如果你的代码超过 10 行，它可能会做得比预期的更多。确保你的方法命名它们的意图，比如`OpenDatabaseConnection()`和`CloseDatabaseConnection()`，并且它们要坚持它们的意图，不要偏离它们。

现在我们要看一下方法参数。

### 具有两个以上参数的方法

具有许多参数的方法往往变得有些难以控制。除了难以阅读之外，很容易将一个值传递给错误的参数并破坏类型安全。

随着参数数量的增加，测试方法变得越来越复杂，主要原因是你有更多的排列组合要应用到你的测试用例上。可能会错过一个在生产中会导致问题的用例。

### 使用异常来控制程序流程

用异常来控制程序流程可能会隐藏代码的意图。它们也可能导致意外和意想不到的结果。你的代码已经被编程成期望一个或多个异常，这表明你的设计是错误的。在第五章中更详细地介绍了一个典型情况，*异常处理*。

典型情况是当企业使用**业务规则异常**（**BREs**）时。一个方法将执行一个动作，预期会抛出一个异常。程序流程将根据异常是否被抛出来确定。一个更好的方法是使用可用的语言结构来执行返回布尔值的验证检查。

以下代码显示了使用 BRE 来控制程序流程：

```cs
public void BreFlowControlExample(BusinessRuleException bre)
{
    switch (bre.Message)
    {
        case "OutOfAcceptableRange":
            DoOutOfAcceptableRangeWork();
            break;
        default:
            DoInAcceptableRangeWork();
            break;
    }
}
```

该方法接受`BusinessRuleException`。根据异常中的消息，`BreFlowControlExample()`要么调用`DoOutOfAcceptableRangeWork()`方法，要么调用`DoInAcceptableRangeWork()`方法。

通过布尔逻辑来控制流程是一个更好的方法。让我们看一下以下`BetterFlowControlExample()`方法：

```cs
public void BetterFlowControlExample(bool isInAcceptableRange)
{
    if (isInAcceptableRange)
        DoInAcceptableRangeWork();
    else
        DoOutOfAcceptableRangeWork();
}
```

在`BetterFlowControlExample()`方法中，一个布尔值被传递到方法中。这个布尔值用于确定要执行哪条路径。如果条件在可接受范围内，那么将调用`DoInAcceptableRangeWork()`。否则，将调用`DoOutOfAcceptableRangeWork()`方法。

接下来，我们将考虑难以阅读的代码。

### 难以阅读的代码

像千层饼和意大利面代码这样的代码真的很难阅读或跟踪。糟糕命名的方法也可能是一个痛点，因为它们可能会掩盖方法的意图。如果方法很大，并且链接的方法被一些不相关的方法分开，那么方法会进一步被混淆。

千层饼代码，也更常见地称为间接引用，指的是抽象层次，其中某物是按名称而不是按动作来引用的。分层在**面向对象编程**（**OOP**）中被广泛使用，并且效果很好。然而，使用的间接引用越多，代码就会变得越复杂。这可能会让项目中的新程序员很难理解代码。因此，必须在间接引用和易理解性之间取得平衡。

意大利面代码指的是紧密耦合、内聚性低的一团乱麻。这样的代码很难维护、重构、扩展和重新设计。但好的一面是，它在编程上更加程序化，因此阅读和跟踪起来会更容易。我记得曾经在一个 VB6 GIS 程序上作为初级程序员工作，这个程序被公司购买并用于营销目的。我的技术总监和他的高级程序员之前曾试图重新设计软件，但失败了。所以他们把这个任务交给了我，让我重新设计这个程序。但当时我并不擅长软件分析和设计，所以我也失败了。

代码太复杂，难以理解和分组到相关项目中，而且太大了。事后看来，我最好是列出程序所做的一切，按功能对列表进行分组，然后在甚至不看代码的情况下列出一系列要求。

所以我在重新设计软件时学到的教训是，无论如何都要避免看代码。写下程序的所有功能，以及它应该包括的新功能。将列表转化为一组软件需求，附带任务、测试和验收标准，然后按照规格进行编程。

### 紧密耦合的代码

紧密耦合的代码很难测试，也很难扩展或修改。依赖于系统内其他代码的代码也很难重用。

紧密耦合的一个例子是在参数中引用具体类类型而不是引用接口。当引用具体类时，对具体类的任何更改直接影响引用它的类。因此，如果您为连接到 SQL Server 的客户端创建了一个数据库连接类，然后接受需要 Oracle 数据库的另一个客户端，那么具体类将必须针对该特定客户端及其 Oracle 数据库进行修改。这将导致代码的两个版本。

客户越多，所需的代码版本就越多。这很快变得难以维护，而且在财务上非常昂贵。想象一下，您的数据库连接类有 100,000 个不同的客户使用类的 30 个变体中的 1 个，并且它们都存在已经确定并影响它们所有的相同错误。这是 30 个类必须具有相同的修复措施，经过测试，打包和部署。这是很多维护开销，而且在财务上非常昂贵。

通过引用接口类型并使用数据库工厂构建所需的连接对象，可以克服这种特定情况。然后，客户可以在配置文件中设置连接字符串，并将其传递给工厂。工厂将为指定连接字符串中指定的特定数据库类型生成实现连接接口的具体连接类。

以下是紧密耦合代码的糟糕示例：

```cs
public class Database
{
    private SqlServerConnection _databaseConnection;

    public Database(SqlServerConnection databaseConnection)
    {
        _databaseConnection = databaseConnection;
    }
}
```

从示例中可以看出，我们的数据库类与使用 SQL Server 绑定，并且需要硬编码更改才能接受任何其他类型的数据库。我们将在后面的章节中涵盖代码重构，包括实际的代码示例。

### 低内聚

低内聚由执行各种不同任务的不相关代码组成。例如，一个实用程序类包含许多不同的实用程序方法，用于处理日期，文本，数字，进行文件输入和输出，数据验证以及加密和解密。

### 对象挂在那里

当对象挂在内存中时，它们可能导致内存泄漏。

静态变量可能以几种方式导致内存泄漏。如果您没有使用`DependencyObject`或`INotifyPropertyChanged`，那么您实际上是在订阅事件。**公共语言运行时**（**CLR**）通过`PropertyDescriptors AddValueChanged`事件使用`ValueChanged`事件创建强引用，这导致存储引用绑定到的对象的`PropertyDescriptor`。

除非取消订阅绑定，否则会导致内存泄漏。使用静态变量引用不会被释放的对象也会导致内存泄漏。静态变量引用的任何对象都被垃圾收集器标记为不可收集。这是因为引用对象的静态变量是**垃圾收集**（**GC**）根，任何是 GC 根的东西都被垃圾收集器标记为*不要收集*。

当您使用捕获类成员的匿名方法时，会引用类实例。这会导致类实例的引用在匿名方法保持活动的同时保持活动。

在使用**非托管代码**（**COM**）时，如果不释放任何托管和非托管对象并显式释放任何内存，那么会导致内存泄漏。

在不使用弱引用、删除未使用的缓存或限制缓存大小的情况下，无限期缓存的代码最终会耗尽内存。

如果在永远不终止的线程中创建对象引用，也会导致内存泄漏。

不是匿名引用类的事件订阅。当这些事件保持订阅状态时，对象将继续存在于内存中。因此，除非在不需要时取消订阅事件，否则可能会导致内存泄漏。

### 使用 Finalize()方法

虽然终结器可以帮助释放未正确处理的对象的资源，并有助于防止内存泄漏，但它们也有许多缺点。

您不知道何时会调用终结器。它们将与图上所有依赖项一起被垃圾收集器提升到下一代，并且直到垃圾收集器决定这样做之前，它们不会被垃圾收集。这意味着对象可能会长时间停留在内存中。使用终结器可能会导致内存不足异常，因为您可能会比垃圾收集速度更快地创建对象。

### 过度设计

过度设计可能是一场噩梦。最大的原因是，作为一个普通人，浏览一个庞大的系统，试图理解它，如何使用它，以及各个部分的功能是一个耗时的过程。当没有文档时，您对系统还很陌生，甚至使用它比您长时间的人也无法回答您的问题时，情况就更加如此。

当您被要求在设定的截止日期内进行工作时，这可能是一个主要的压力原因。

#### 学会保持简单，愚蠢

一个很好的例子是我曾经工作过的一个地方。我必须为一个接受来自服务的 JSON 的 Web 应用编写一个测试，允许一个子类进行测试，然后将结果的评分传递给另一个服务。根据公司政策，我没有按照 OOP、SOLID 或 DRY 的要求进行操作。但是我通过在非常短的时间内使用 KISS 和过程式编程与事件完成了工作。我因此受到了惩罚，并被迫使用他们自己开发的测试播放器进行重写。

因此，我开始学习他们的测试播放器。没有文档，也没有遵循 DRY 原则，很少有人真正理解它。与我的受罚系统相比，我的新版本需要使用他们的系统，因此花了几周的时间来构建，因为它没有做我需要它做的事情，而且我也不被允许修改它来做我需要它做的事情。因此，我在等待有人做所需的工作时被拖慢了速度。

我的第一个解决方案满足了业务需求，并且是一个独立的代码片段，不关心其他任何事情。第二个解决方案满足了开发团队的技术要求。项目的持续时间超过了截止日期。任何超过截止日期的项目都会比计划的成本更高。

我想要用我的受罚系统表达的另一点是，它比被重写为使用通用测试播放器的新系统要简单得多，更容易理解。

您并不总是需要遵循 OOP、SOILD 和 DRY。有时候不遵循反而更好。毕竟，您可以编写最美丽的 OOP 系统。但在底层，您的代码被转换为更接近计算机理解的过程式代码！

### 大类中缺乏区域

大量区域的大类很难阅读和跟踪，特别是当相关方法没有分组在一起时。区域对于在大类中对类似成员进行分组非常有用。但是如果您不使用它们，它们就没有用处！

### 失去意图的代码

如果您正在查看一个类，并且它正在做几件事情，那么您如何知道它的原始意图是什么？例如，如果您正在寻找一个日期方法，并且在代码的输入/输出命名空间的文件类中找到它，那么日期方法是否在正确的位置？不是。其他不了解您的代码的开发人员会很难找到该方法吗？当然会。看看这段代码：

```cs
public class MyClass 
{
    public void MyMethod()
    {
        // ...implementation...
    }

    public DateTime AddDates(DateTime date1, DateTime date2)
    {
        //...implementation...
    }

    public Product GetData(int id)
    {
        //...implementation...
    }
}
```

类的目的是什么？名称没有给出任何指示，`MyMethod` 做什么？该类似乎还在进行日期操作和获取产品数据。`AddDates` 方法应该在专门管理日期的类中。`GetData` 方法应该在产品的视图模型中。

### 直接暴露信息

直接暴露信息的类是不好的。除了产生可能导致错误的紧密耦合之外，如果要更改信息类型，就必须在使用的每个地方更改类型。另外，如果要在赋值之前执行数据验证怎么办？举个例子：

```cs
public class Product
{
    public int Id;
    public int Name;
    public int Description;
    public string ProductCode;
    public decimal Price;
    public long UnitsInStock
}
```

在上述代码中，如果要将 `UnitsInStock` 从类型 `long` 更改为类型 `int`，则必须更改 *每个* 引用它的代码。对 `ProductCode` 也是一样。如果新的产品代码必须遵循严格的格式，如果字符串可以直接由调用类分配，您将无法验证产品代码。

## 良好的代码

既然您知道不应该做什么，现在是时候简要了解一些良好的编码实践，以便编写令人愉悦、高性能的代码。

### 适当的缩进

当您使用适当的缩进时，阅读代码会变得更加容易。您可以通过缩进看出代码块的开始和结束位置，以及哪些代码属于这些代码块：

```cs
public void DoSomething()
{
    for (var i = 0; i < 1000; i++)
    {
        var productCode = $"PRC000{i}";
        //...implementation
    }
}
```

在上述简单示例中，代码看起来很好，易于阅读。您可以清楚地看到每个代码块的开始和结束位置。

### 有意义的注释

有意义的注释是表达程序员意图的注释。当代码正确但可能不容易被新手理解，甚至在几周后也是如此时，这样的注释是有用的。这样的注释可以真正有帮助。

### API 文档注释

一个好的 API 是具有易于遵循的良好文档的 API。API 注释是 XML 注释，可用于生成 HTML 文档。HTML 文档对于想要使用您的 API 的开发人员很重要。文档越好，开发人员越有可能想要使用您的 API。举个例子：

```cs
/// <summary>
/// Create a new <see cref="KustoCode"/> instance from the text and globals. Does not perform 
/// semantic analysis.
/// </summary>
/// <param name="text">The code text</param>
/// <param name="globals">
///   The globals to use for parsing and semantic analysis. Defaults to <see cref="GlobalState.Default"/>
/// </param>.
 public static KustoCode Parse(string text, GlobalState globals = null) { ... }
```

Kusto 查询语言项目的这段摘录是 API 文档注释的一个很好的例子。

### 使用命名空间进行适当的组织

适当组织并放置在适当的命名空间中的代码可以在寻找特定代码片段时为开发人员节省大量时间。例如，如果您正在寻找与日期和时间相关的类和方法，最好有一个名为 `DateTime` 的命名空间，一个名为 `Time` 的类用于与时间相关的方法，以及一个名为 `Date` 的类用于与日期相关的方法。

以下是命名空间的适当组织的示例：

| **名称** | **描述** |
| --- | --- |
| `CompanyName.IO.FileSystem` | 该命名空间包含定义文件和目录操作的类。 |
| `CompanyName.Converters` | 该命名空间包含执行各种转换操作的类。 |
| `CompanyName.IO.Streams` | 该命名空间包含用于管理流输入和输出的类型。 |

### 良好的命名约定

遵循 Microsoft C# 命名约定是很好的。对于命名空间、类、接口、枚举和方法，请使用帕斯卡命名法。对于变量名和参数名，请使用驼峰命名法，并确保使用下划线前缀来命名成员变量。

看看这个示例代码：

```cs
using System;
using System.Text.RegularExpressions;

namespace CompanyName.ProductName.RegEx
{
  /// <summary>
  /// An extension class for providing regular expression extensions 
  /// methods.
  /// </summary>
  public static class RegularExpressions
  {
    private static string _preprocessed;

    public static string RegularExpression { get; set; }

    public static bool IsValidEmail(this string email)
    {
      // Email address: RFC 2822 Format. 
      // Matches a normal email address. Does not check the 
      // top-level domain.
      // Requires the "case insensitive" option to be ON.
      var exp = @"\A(?:[a-z0-9!#$%&'*+/=?^_`{|}~-]+(?:\.
       [a-z0-9!#$%&'*+/=?^_`{|}~-]+)*@(?:a-z0-9?\.)+a-z0-9?)\Z";
      bool isEmail = Regex.IsMatch(email, exp, RegexOptions.IgnoreCase);
      return isEmail;
    }

    // ... rest of the implementation ...

  }
}

```

它展示了命名空间、类、成员变量、类、参数和局部变量的命名约定的合适示例。

### 只做一件事的类

一个好的类是一个只做一件事的类。当您阅读类时，其意图是清晰的。只有应该在该类中的代码才在该类中，没有其他东西。

### 只做一件事的方法

方法应该只做一件事。你不应该有一个做多件事的方法，比如解密字符串和执行字符串替换。方法的意图应该是清晰的。只做一件事的方法更容易小、易读和有意义。

### 方法不超过 10 行，最好不超过 4 行

理想情况下，你应该有不超过 4 行代码的方法。然而，这并不总是可能的，所以你应该努力使方法的长度不超过 10 行，以便它们易于阅读和维护。

### 方法不超过两个参数

最好是有没有参数的方法，但有一个或两个也可以。如果开始有超过两个参数，你需要考虑你的类和方法的责任：它们是否承担了太多？如果你确实需要超过两个参数，那么最好传递一个对象。

任何超过两个参数的方法都可能变得难以阅读和理解。最多只有两个参数使得代码更易读，而一个对象作为单个参数比具有多个参数的方法更易读。

### 异常的正确使用

永远不要使用异常来控制程序流程。以一种不会引发异常的方式处理可能触发异常的常见条件。一个好的类设计应该能够避免异常。

通过使用`try`/`catch`/`finally`异常来恢复异常和/或释放资源。在捕获异常时，使用可能在你的代码中抛出的特定异常，这样你就可以获得更详细的信息来记录或帮助处理异常。

有时，使用预定义的.NET 异常类型并不总是可能的。在这种情况下，将需要生成自定义异常。用单词`Exception`作为自定义异常类的后缀，并确保包括以下三个构造函数：

+   `Exception()`: 使用默认值

+   `Exception(string)`: 接受一个字符串消息

+   `Exception(string, exception)`: 接受一个字符串消息和一个内部异常

如果必须抛出异常，不要返回错误代码，而是返回带有有意义信息的异常。

### 可读的代码

代码越易读，开发者就越喜欢使用它。这样的代码更容易学习和使用。随着开发者在项目中的进出，新手将能够轻松阅读、扩展和维护代码。易读的代码也不太容易出错和不安全。

### 松散耦合的代码

松散耦合的代码更容易测试和重构。如果需要，你也可以更容易地交换和更改松散耦合的代码。代码重用是松散耦合代码的另一个好处。

让我们使用一个糟糕的例子，一个数据库被传递了一个 SQL Server 连接。我们可以通过引用一个接口而不是具体类型，使得相同的类松散耦合。让我们看一下之前重构的糟糕例子的好例子：

```cs
public class Database
{
    private IDatabaseConnection _databaseConnection;

    public Database(IDatabaseConnection databaseConnection)
    {
        _databaseConnection = datbaseConnection;
    }
}
```

正如你在这个相当基本的例子中所看到的，只要传入的类实现了`IDatabaseConnection`接口，我们就可以为任何类型的数据库连接传入任何类。因此，如果我们在 SQL Server 连接类中发现了一个 bug，只有 SQL Server 客户端会受到影响。这意味着具有不同数据库的客户端将继续工作，我们只需要在一个类中修复 SQL Server 客户端的代码。这减少了维护开销，从而降低了总体维护成本。

### 高内聚

正确分组的常见功能被认为是高度内聚的。这样的代码很容易找到。例如，如果你查看`Microsoft System.Diagnostics`命名空间，你会发现它只包含与诊断相关的代码。在`Diagnostics`命名空间中包含集合和文件系统代码是没有意义的。

### 对象被清理干净

在使用可处理类时，您应该始终调用`Dispose()`方法，以清理处于使用中的任何资源。这有助于消除内存泄漏的可能性。

有时您可能需要将对象设置为`null`以使其超出范围。一个例子是一个静态变量，它保存对您不再需要的对象的引用。

`using`语句也是一种很好的清洁方式来使用可处理对象，因为当对象不再在范围内时，它会自动被处理，所以你不需要显式调用`Dispose()`方法。让我们来看一下接下来的代码：

```cs
using (var unitOfWork = new UnitOfWork())
{
 // Perform unit of work here.
}
// At this point the unit of work object has been disposed of.
```

代码在`using`语句中定义了一个可处理对象，并在打开和关闭大括号之间执行所需的操作。在大括号退出之前，对象会自动被处理。因此，无需手动调用`Dispose()`方法，因为它会自动调用。

### 避免 Finalize()方法

在使用不受管理的资源时，最好实现`IDisposable`接口，并避免使用`Finalize()`方法。不能保证最终器何时运行。它们可能不会按您期望的顺序或时间运行。相反，在`Dispose()`方法中处理不受管理的资源更好且更可靠。

### 正确的抽象级别

当您仅向更高级别公开需要公开的内容，并且不在实现中迷失时，您就具有了正确的抽象级别。

如果您发现自己在实现细节中迷失了方向，那么您已经过度抽象了。如果您发现多个人不得不同时在同一个类中工作，那么您就没有足够的抽象。在这两种情况下，都需要重构以使抽象达到正确的水平。

### 在大类中使用区域

区域对于在大类中对项目进行分组非常有用，因为它们可以被折叠起来。阅读大类并不得不在方法之间来回跳转可能会令人望而生畏，因此在类中对相互调用的方法进行分组是一种很好的方法。在处理代码时，可以根据需要折叠和展开这些方法。

从迄今为止我们所看到的内容可以看出，良好的编码实践使得代码更易读和更易维护。我们现在将看一下编码标准和原则的必要性，以及一些软件方法论，如 SOLID 和 DRY。

# 编码标准、原则和方法论的必要性

大多数软件今天都是由多个团队的程序员编写的。正如您所知，我们都有自己独特的编码方式，我们都有某种形式的编程思想。您可以很容易地找到关于各种软件开发范式的编程辩论。但共识是，如果我们都遵守一组给定的编码标准、原则和方法论，那么作为程序员，这确实会让我们的生活更轻松。

让我们更详细地回顾一下这些意思。

## 编码标准

编码标准规定了必须遵守的几个要点和禁忌。这些标准可以通过诸如 FxCop 之类的工具或通过同行代码审查手动执行。所有公司都有自己的编码标准必须遵守。但在现实世界中，您会发现，当企业期望满足截止日期时，这些编码标准可能会被抛到一边，因为截止日期可能比实际代码质量更重要。这通常通过将任何所需的重构添加到错误列表作为技术债务来解决，以便在发布后解决。

微软有自己的编码标准，大多数情况下这些是被采纳的标准，可以根据每个企业的需求进行修改。以下是一些在线找到的编码标准的例子：

+   [`www.c-sharpcorner.com/UploadFile/ankurmalik123/C-Sharp-coding-standards/`](https://www.c-sharpcorner.com/UploadFile/ankurmalik123/C-Sharp-coding-standards/)

+   [`www.dofactory.com/reference/csharp-coding-standards`](https://www.dofactory.com/reference/csharp-coding-standards)

+   [`blog.submain.com/coding-standards-c-developers-need/`](https://blog.submain.com/coding-standards-c-developers-need/)

当跨团队或同一团队的人遵守编码标准时，您的代码库将变得统一。统一的代码库更容易阅读、扩展和维护。它也更不容易出错。如果存在错误，也更容易找到，因为代码遵循一套所有开发人员都遵守的标准准则。

## 编码原则

编码原则是一组编写高质量代码、测试和调试代码以及对代码进行维护的准则。原则可能因程序员和编程团队而异。

即使您是一个孤独的程序员，也可以通过定义自己的编码原则并坚持它们来为自己提供光荣的服务。如果您在一个团队中工作，那么达成一套编码标准对于所有人都是非常有益的，可以使共享代码的工作更加容易。

在本书中，您将看到诸如 SOLID、YAGNI、KISS 和 DRY 等编码原则的示例，所有这些都将被详细解释。但现在，**SOLID**代表**单一职责原则、开闭原则、里氏替换原则、接口隔离原则**和**依赖反转原则**。**YAGNI**代表**你不会需要它**。**KISS**代表**保持简单，愚蠢**，**DRY**代表**不要重复自己**。

## 编码方法论

编码方法论将软件开发过程分解为许多预定义阶段。每个阶段都将与之相关的一些步骤。不同的开发人员和开发团队将遵循自己的编码方法论。编码方法论的主要目的是从最初的概念、编码阶段到部署和维护阶段的流程。

在本书中，您将习惯于使用 SpecFlow 进行**测试驱动开发**（**TDD**）和**行为驱动开发**（**BDD**），以及使用 PostSharp 进行**面向方面的编程**（**AOP**）。

## 编码约定

最好实施微软的 C#编码约定。您可以在[`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/inside-a-program/coding-conventions`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/inside-a-program/coding-conventions)上查看它们。

通过采用微软的编码约定，您可以确保以正式接受和商定的格式编写代码。这些 C#编码约定帮助人们专注于阅读您的代码，而不是专注于布局。基本上，微软的编码标准促进了最佳实践。

## 模块化

将大型程序分解为较小的模块是非常有意义的。小模块易于测试，更容易重用，并且可以独立于其他模块进行操作。小模块也更容易扩展和维护。

模块化程序可以分为不同的程序集和程序集内的不同命名空间。模块化程序在团队环境中也更容易操作，因为不同的模块可以由不同的团队进行操作。

在同一个项目中，通过添加反映命名空间的文件夹来将代码模块化。命名空间必须只包含与其名称相关的代码。因此，例如，如果您有一个名为`FileSystem`的命名空间，则与文件和目录相关的类型应放置在该文件夹中。同样，如果您有一个名为`Data`的命名空间，则只有与数据和数据源相关的类型应放置在该命名空间中。

正确模块化的另一个美好之处是，如果你保持模块小而简单，它们就更容易阅读。除了编码之外，大部分程序员的生活都花在阅读和理解代码上。因此，代码越小、正确模块化，就越容易阅读和理解。这会导致对代码的更深入理解，并提高开发人员对代码的接受和使用。

## KISS

你可能是计算机编程世界的超级天才。你可能能够编写出让其他程序员只能惊叹地盯着它并流口水的代码。但其他程序员只看代码就知道它是什么吗？如果你在 10 周后发现了这段代码，当时你深陷于不同代码的海洋中，需要满足截止日期，你能清楚地解释你的代码做了什么以及你选择编码方法的理由吗？你有没有考虑过你可能需要在将来进一步处理这段代码？

你是否曾经编写过一些代码，然后离开，几天后再看它，然后对自己说，“我没写这种垃圾，是吗？我当时在想什么！？”我知道我曾经有过这种经历，我的一些前同事也有。

在编写代码时，保持代码简单且易于阅读，即使新手初级程序员也能理解。通常，初级程序员需要阅读、理解和维护代码。代码越复杂，初级程序员需要花费的时间就越长。甚至高级程序员也可能在复杂系统中遇到困难，以至于他们离开寻找其他工作，这样对大脑和身心的负担就会减轻。

例如，如果你正在开发一个简单的网站，问问自己几个问题。它真的需要使用微服务吗？你正在处理的旧项目真的很复杂吗？有可能简化它以便更容易维护吗？在开发新系统时，你需要写一个健壮、可维护和可扩展的解决方案，需要的最少的移动部件是什么？

## YAGNI

YAGNI 是编程敏捷世界中的一种纪律，规定程序员在绝对需要之前不应添加任何代码。一个诚实的程序员会根据设计编写失败的测试，然后只编写足够的生产代码使测试工作，最后重构代码以消除任何重复。使用 YAGNI 软件开发方法，你将你的类、方法和总代码行数保持在绝对最低限度。

YAGNI 的主要目标是防止计算机程序员过度设计软件系统。如果不需要，就不要增加复杂性。你必须记住只编写你需要的代码。不要编写你不需要的代码，也不要为了实验和学习而编写代码。将实验和学习代码保留在专门用于这些目的的沙盒项目中。

## DRY

我说*不要重复自己！* 如果你发现自己在多个地方写了相同的代码，那么这绝对是重构的候选。你应该查看代码，看看它是否可以变成通用的，并放在一个辅助类中供整个系统使用，或者放在一个库中供其他项目使用。

如果你在多个地方有相同的代码，并且发现代码有错误需要修改，那么你必须在其他地方修改代码。在这种情况下，很容易忽视需要修改的代码。结果就是发布的代码在一些地方修复了问题，但在其他地方仍然存在。

这就是为什么在遇到重复代码时，尽快删除它是个好主意，因为如果不这样做，它可能会在将来造成更多问题。

## SOLID

SOLID 是一组旨在使软件更易于理解和维护的五个设计原则。软件代码应该易于阅读和扩展，而无需修改现有代码的部分。五个 SOLID 设计原则如下：

+   **单一责任原则**：类和方法应该只执行单一职责。组成单一责任的所有元素应该被分组在一起并封装起来。

+   **开闭原则**：类和方法应该对扩展开放，对修改关闭。当需要对软件进行更改时，您应该能够扩展软件而不修改任何代码。

+   **里氏替换原则**：您的函数有一个指向基类的指针。它必须能够使用任何从基类派生的类而不知道它。

+   **接口隔离原则**：当您有大型接口时，使用它们的客户端可能不需要所有的方法。因此，使用**接口隔离原则**（**ISP**），您将方法提取到不同的接口中。这意味着您不再有一个大接口，而是有许多小接口。类可以实现只有它们需要的方法的接口。

+   **依赖反转原则**：当您有一个高级模块时，它不应该依赖于任何低级模块。您应该能够在不影响使用它们的高级模块的情况下自由切换低级模块。高级和低级模块都应该依赖于抽象。

抽象不应该依赖于细节，但细节应该依赖于抽象。

当声明变量时，您应该始终使用静态类型，如接口或抽象类。然后可以将实现接口或继承自抽象类的具体类分配给变量。

## 奥卡姆剃刀

奥卡姆剃刀陈述如下：*实体不应该被无必要地增加*。换句话说，这基本上意味着*最简单的解决方案很可能是正确的*。因此，在软件开发中，违反奥卡姆剃刀原则是通过进行不必要的假设并采用最不简单的解决方案来实现的。

软件项目通常建立在一系列事实和假设之上。事实很容易处理，但假设是另一回事。在解决软件项目问题时，通常作为团队讨论问题和潜在解决方案。在选择解决方案时，您应该始终选择假设最少的项目，因为这将是最准确的实施选择。如果有一些公平的假设，您需要做的假设越多，您的设计解决方案就越有可能存在缺陷。

移动部件较少的项目出现问题的可能性较小。因此，通过保持项目小，尽可能少地做出假设，除非有必要，并且只处理事实，您遵守了奥卡姆剃刀原则。

# 总结

在本章中，您已经对好代码和坏代码有了介绍，希望您现在明白了为什么好代码很重要。您还提供了微软 C#编码约定的链接，以便您可以遵循微软的最佳编码实践（如果您还没有这样做的话）。

您还简要介绍了各种软件方法，包括 DRY、KISS、SOLID、YAGNI 和奥卡姆剃刀。

使用模块化，您已经看到了使用命名空间和程序集模块化代码的好处。这些好处包括独立团队能够独立工作在独立模块上，以及代码的可重用性和可维护性。

在下一章中，我们将看一下同行代码审查。有时可能会令人不快，但同行代码审查有助于通过确保他们遵守公司编码程序来使程序员受到约束。

# 问题

1.  坏代码的一些结果是什么？

1.  好代码的一些结果是什么？

1.  写模块化代码的一些好处是什么？

1.  DRY 代码是什么？

1.  写代码时为什么要 KISS？

1.  SOLID 的首字母缩写代表什么？

1.  解释 YAGNI。

1.  奥卡姆剃刀是什么？

# 进一步阅读

+   *自适应代码：使用设计模式和 SOLID 原则进行敏捷编码，第二版*，作者是 Gary McLean Hall。

+   *使用 C#和.NET Core 的设计模式实践*，作者是 Jeffrey Chilberto 和 Gaurav Aroraa。

+   *可维护软件构建，C#版*，作者是 Rob can der Leek，Pascal can Eck，Gijs Wijnholds，Sylvan Rigal 和 Joost Visser。

+   关于软件反模式的良好信息，包括一个反模式的长列表，可以在[`en.wikibooks.org/wiki/Introduction_to_Software_Engineering/Architecture/Anti-Patterns`](https://en.wikibooks.org/wiki/Introduction_to_Software_Engineering/Architecture/Anti-Patterns)找到。

+   关于设计模式的良好信息，包括一个链接到图表和实现源代码的设计模式列表，可以在[`en.wikipedia.org/wiki/Software_design_pattern`](https://en.wikipedia.org/wiki/Software_design_pattern)找到。
