类、对象和数据结构

在本章中，我们将讨论组织、格式化和注释类。我们还将讨论编写符合迪米特法则的干净的 C#对象和数据结构。此外，我们还将讨论不可变对象和数据结构，以及在`System.Collections.Immutable`命名空间中定义不可变集合的接口和类。

我们将涵盖以下广泛的主题：

+   组织类

+   用于文档生成的注释

+   内聚性和耦合性

+   迪米特法则

+   不可变对象和数据结构

在本章中，你将学到以下技能：

+   如何有效地使用命名空间组织你的类。

+   当你学会用单一职责来编程时，你的类会变得更小更有意义。

+   在编写自己的 API 时，你可以通过提供注释来帮助文档生成工具，从而提供良好的开发者文档。

+   由于高内聚性和低耦合性，你编写的任何程序都将易于修改和扩展。

+   最后，你将能够应用迪米特法则并编写和使用不可变的数据结构。

因此，让我们开始看看如何通过使用命名空间有效地组织我们的类。

# 第四章：技术要求

你可以在 GitHub 上访问本章的代码，网址为[`github.com/PacktPublishing/Clean-Code-in-C-/tree/master/CH03`](https://github.com/PacktPublishing/Clean-Code-in-C-/tree/master/CH03)。

# 组织类

你会注意到一个干净项目的标志是它会有组织良好的类。文件夹将用于将属于一起的类分组。此外，文件夹中的类将被封装在与程序集名称和文件夹结构匹配的命名空间中。

每个接口、类、结构和枚举都应该在正确的命名空间中有自己的源文件。源文件应该在适当的文件夹中逻辑分组，源文件的命名空间应该与程序集名称和文件夹结构匹配。以下截图展示了一个干净的文件夹和文件结构：

![](img/1dc5cf04-4924-414d-88a9-62c6305c9757.png)

在实际源文件中，不要有多个接口、类、结构或枚举。原因是这样会使定位项变得困难，尽管我们有智能感知来帮助我们。

在考虑你的命名空间时，遵循公司名称、产品名称、技术名称的帕斯卡命名规则，然后是由空格分隔的组件的复数名称。以下是一个示例：

```cs
FakeCompany.Product.Wpf.Feature.Subnamespace {} // Product, technology and feature specific.
```

以公司名称开头的原因是它有助于避免命名空间类。因此，如果微软和 FakeCompany 都有一个名为`System`的命名空间，你想要使用哪个`System`可以通过公司名称来区分。

接下来，任何能够在多个项目中重用的代码项最好放在可以被多个项目访问的单独的程序集中：

```cs
FakeCompany.Wpf.Feature.Subnamespace {} /* Technology and feature specific. Can be used across multiple products. */
```

在代码中使用测试时，比如进行**测试驱动开发**（**TDD**），最好将测试类放在单独的程序集中。测试程序集应该始终以被测试的程序集名称结尾的`Tests`命名空间。

```cs
FakeCompany.Core.Feature {} /* Technology agnostic and feature specific. Can be used across multiple products. */
```

永远不要将不同程序集的测试放在同一个测试程序集中。始终保持它们分开。

此外，命名空间和类型不应该使用相同的名称，因为这可能会产生编译器冲突。在为公司名称、产品名称和缩写形式命名空间时，可以省略复数形式。

总结一下，组织类时要牢记以下规则：

+   遵循公司名称、产品名称、技术名称的帕斯卡命名规则，然后是由空格分隔的组件的复数名称。

+   将可重用的代码项放在单独的程序集中。

+   不要使用相同的名称作为命名空间和类型。

+   不要将公司和产品名称以及缩写形式变为复数。

我们将继续讨论类的责任。

# 一个类应该只有一个责任

责任是分配给类的工作。在 SOLID 原则集中，S 代表**单一责任原则**（**SRP**）。当应用于类时，SRP 规定类必须只处理正在实现的功能的一个方面。该单个方面的责任应完全封装在类内。因此，您不应该将超过一个责任应用于一个类。

让我们看一个例子来理解为什么：

```cs
public class MultipleResponsibilities() 
{
    public string DecryptString(string text, 
     SecurityAlgorithm algorithm) 
    { 
        // ...implementation... 
    }

    public string EncryptString(string text, 
     SecurityAlgorithm algorithm) 
    { 
        // ...implementation... 
    }

    public string ReadTextFromFile(string filename) 
    { 
        // ...implementation... 
    }

    public string SaveTextToFile(string text, string filename) 
    { 
        // ...implementation... 
    }
}
```

正如您在前面的代码中所看到的，对于`MultipleResponsibilities`类，我们已经实现了我们的加密功能，包括`DecryptString`和`EncryptString`方法。我们还实现了文件访问，包括`ReadTextFromFile`和`SaveTextToFile`方法。这个类违反了 SRP 原则。

因此，我们需要将这个类分成两个类，一个用于加密和另一个用于文件访问：

```cs
namespace FakeCompany.Core.Security
{
    public class Cryptography
    {    
        public string DecryptString(string text, 
         SecurityAlgorithm algorithm) 
        { 
            // ...implementation... 
        }

        public string EncryptString(string text, 
         SecurityAlgorithm algorithm) 
        { 
            // ...implementation... 
        }  
    }
}
```

正如我们现在从前面的代码中所看到的，通过将`EncryptString`和`DecryptString`方法移动到核心安全命名空间中的自己的`Cryptography`类中，我们已经使得在不同产品和技术组中重用代码来加密和解密字符串变得容易。`Cryptography`类也符合 SRP。

在下面的代码中，我们可以看到`Cryptography`类的`SecurityAlgorithm`参数是一个枚举，并已放置在自己的源文件中。这有助于保持代码整洁、最小化和良好组织：

```cs
using System;

namespace FakeCompany.Core.Security
{
    [Flags]
    public enum SecurityAlgorithm
    {
        Aes,
        AesCng,
        MD5,
        SHA5
    }
}

```

现在，在下面的`TextFile`类中，我们再次遵守 SRP，并且有一个很好的可重用的类，位于适当的核心文件系统命名空间中。`TextFile`类可以在不同产品和技术组中重复使用：

```cs
namespace FakeCompany.Core.FileSystem
{
    public class TextFile
    {
        public string ReadTextFromFile(string filename) 
        { 
            // ...implementation... 
        }

        public string SaveTextToFile(string text, string filename) 
        { 
            // ...implementation... 
        }
    }
}
```

我们已经看过了类的组织和责任。现在让我们来看看为了其他开发人员的利益而对类进行注释。

# 用于文档生成的注释

始终为您的源代码编写文档是一个好主意，无论是内部项目还是将由其他开发人员使用的外部软件。内部项目因为开发人员的流失而受到影响，通常缺乏或几乎没有可用于帮助新开发人员快速上手的文档。许多第三方 API 由于开发人员文档的糟糕状态而难以起步或接受速度低于预期，通常由于采用者因开发人员文档的糟糕状态而感到沮丧而放弃 API。

在每个源代码文件的顶部包括版权声明并对您的命名空间、接口、类、枚举、结构、方法和属性进行注释始终是一个好主意。您的版权注释应该在源文件中首先出现，在`using`语句之上，并采用以`/*`开头和以`*/`结尾的多行注释形式：

```cs
/**********************************************************************************
 * Copyright 2019 PacktPub
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy of 
 * this software and associated documentation files (the "Software"), to deal in 
 * the Software without restriction, including without limitation the rights to use, 
 * copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the 
 * Software, and to permit persons to whom the Software is furnished to do so, 
 * subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in all 
 * copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR 
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, 
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE 
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER 
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, 
 * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE 
 * SOFTWARE. 
 *********************************************************************************/

using System;

/// <summary>
/// The CH3.Core.Security namespace contains fundamental types used 
/// for the purpose of implementing application security.
/// </summary>
namespace CH3.Core.Security
{
    /// <summary>
    /// Encrypts and decrypts provided strings based on the selected 
    /// algorithm.
    /// </summary>
    public class Cryptography
    {
        /// <summary>
        /// Decrypts a string using the selected algorithm.
        /// </summary>
        /// <param name="text">The string to be decrypted.</param>
        /// <param name="algorithm">
        /// The cryptographic algorithm used to decrypt the string.
        /// </param>
        /// <returns>Decrypted string</returns>
        public string DecryptString(string text, 
         SecurityAlgorithm algorithm)
        {
            // ...implementation... 
            throw new NotImplementedException();
        }

        /// <summary>
        /// Encrypts a string using the selected algorithm.
        /// </summary>
        /// <param name="text">The string to encrypt.</param>
        /// <param name="algorithm">
        /// The cryptographic algorithm used to encrypt the string.
        /// </param>
        /// <returns>Encrypted string</returns>
        public string EncryptString(string text, 
         SecurityAlgorithm algorithm)
        {
            // ...implementation... 
            throw new NotImplementedException();
        }
    }
}
```

前面的代码示例提供了一个带有文档化的命名空间和类以及文档化方法的示例。您将看到命名空间和包含的成员的文档注释以`///`开头，并直接位于被评论的项目上方。当您键入三个正斜杠时，Visual Studio 会根据下面的行自动生成 XML 标签。

例如，在前面的代码中，命名空间只有一个摘要，类也是如此，但两个方法都包含一个摘要，一些参数注释和一个返回注释。

下表包含了您可以在文档注释中使用的不同 XML 标签。

| **标签** | **部分** | **目的** |
| --- | --- | --- |
| `<c>` | `<c>` | 将文本格式化为代码 |
| `<code>` | `<code>` | 作为输出提供源代码 |
| `<example>` | `<example>` | 提供示例 |
| `<exception>` | `<exception>` | 描述方法可能抛出的异常 |
| `<include>` | `<include>` | 包含来自外部文件的 XML |
| `<list>` | `<list>` | 添加列表或表格 |
| `<para>` | `<para>` | 为文本添加结构 |
| `<param>` | `<param>` | 描述构造函数或方法的参数 |
| `<paramref>` | `<paramref>` | 标记一个词以识别它是一个参数 |
| `<permission>` | `<permission>` | 描述成员的安全可访问性 |
| `<remarks>` | `<remarks>` | 提供额外信息 |
| `<returns>` | `<returns>` | 描述返回类型 |
| `<see>` | `<see>` | 添加超链接 |
| `<seealso>` | `<seealso>` | 添加一个*参见*条目 |
| `<summary>` | `<summary>` | 总结类型或成员 |
| `<value>` | `<value>` | 描述值 |
| `<typeparam>` |  | 描述类型参数 |
| `<typeparamref>` |  | 标记一个词以识别它是一个类型参数 |

从上表可以清楚地看出，您有很多空间来记录您的源代码。因此，充分利用可用的标签来记录您的代码是一个好主意。文档越好，其他开发人员就能更快更容易地掌握使用代码。

现在是时候看看内聚性和耦合性了。

# 内聚性和耦合性

在设计良好的 C#程序集中，代码将被正确地分组在一起。这就是**高内聚性**。**低内聚性**是指将不相关的代码分组在一起。

您希望相关的类尽可能独立。一个类对另一个类的依赖性越高，耦合性就越高。这就是**紧密耦合**。类之间相互独立程度越高，内聚性就越低。这就是低内聚。

因此，在一个定义良好的类中，您希望有高内聚性和低耦合性。我们现在将看一些紧密耦合的例子，然后是低耦合。

## 紧密耦合的例子

在下面的代码示例中，`TightCouplingA`类打破了封装性，并直接访问了`_name`变量。`_name`变量应该是私有的，并且只能由其封闭类中的属性或方法修改。`Name`属性提供了`get`和`set`方法来验证`_name`变量，但这是毫无意义的，因为这些检查可以被绕过，属性也不会被调用：

```cs
using System.Diagnostics;

namespace CH3.Coupling
{
    public class TightCouplingA
    {
        public string _name;

        public string Name
        {
            get
            {
                if (!_name.Equals(string.Empty))
                    return _name;
                else
                    return "String is empty!";
            }
            set
            {
                if (value.Equals(string.Empty))
                    Debug.WriteLine("String is empty!");
            }
        }
    }
}
```

另一方面，在下面的代码中，`TightCouplingB`类创建了`TightCouplingA`的一个实例。然后，它通过直接访问`_name`成员变量并将其设置为`null`，然后直接访问并将其值打印到调试输出窗口，直接在这两个类之间引入了紧密耦合：

```cs
using System.Diagnostics;

namespace CH3.Coupling
{
    public class TightCouplingB
    {
        public TightCouplingB()
        {
            TightCouplingA tca = new TightCouplingA();
            tca._name = null;
            Debug.WriteLine("Name is " + tca._name);
        }
    }
}
```

现在让我们看一下使用低耦合的相同简单示例。

## 低耦合的例子

在这个例子中，我们有两个类，`LooseCouplingA`和`LooseCouplingB`。`LooseCouplingA`声明了一个名为`_name`的私有实例变量，并通过一个公共属性设置这个变量。

`LooseCouplingB`创建了`LooseCouplingA`的一个实例，并获取和设置`Name`的值。因为无法直接设置`_name`数据成员，所以对该数据成员的设置和获取值的检查是通过属性进行的。

因此，我们有一个松散耦合的例子。让我们看一下名为`LooseCouplingA`和`LooseCouplingB`的两个类，展示了这一点：

```cs
using System.Diagnostics;

namespace CH3.Coupling
{
    public class LooseCouplingA
    {
        private string _name;
        private readonly string _stringIsEmpty = "String is empty";

        public string Name
        {
            get
            {
                if (_name.Equals(string.Empty))
                    return _stringIsEmpty;
                else
                    return _name;
            }

            set
            {
                if (value.Equals(string.Empty))
                    Debug.WriteLine("Exception: String length must be 
                     greater than zero.");
            }
        }
    }
}
```

在`LooseCouplingA`类中，我们将`_name`字段声明为私有，因此阻止直接修改数据。`_name`数据通过`Name`属性间接访问：

```cs

using System.Diagnostics;

namespace CH3.Coupling
{
    public class LooseCouplingB
    {
        public LooseCouplingB()
        {
            LooseCouplingA lca = new LooseCouplingA();
            lca = null;
            Debug.WriteLine($"Name is {lca.Name}");
        }
    }
}
```

`LooseCouplingB`类无法直接访问`LooseCouplingB`类的`_name`变量，因此通过属性修改数据成员。

好吧，我们已经看过耦合性，现在知道如何避免紧密耦合的代码并实现松散耦合的代码。所以现在，是时候让我们看一些低内聚性和高内聚性的例子了。

## 低内聚性的例子

当一个类具有多个职责时，就说它是一个低内聚的类。看一下下面的代码：

```cs
namespace CH3.Cohesion
{
    public class LowCohesion
    {
        public void ConnectToDatasource() { }
        public void ExtractDataFromDataSource() { }
        public void TransformDataForReport() { }
        public void AssignDataAndGenerateReport() { }
        public void PrintReport() { }
        public void CloseConnectionToDataSource() { }
    }
}
```

正如我们所看到的，前面的类至少有三个职责：

+   连接到数据源和断开连接

+   提取数据并将其转换为报告插入准备好

+   生成报告并打印输出

你会清楚地看到这是如何违反 SRP 的。接下来，我们将把这个类分解为三个遵守 SRP 的类。

## 高内聚的例子

在这个例子中，我们将把`LowCohesion`类分解为三个遵守 SRP 的类。这些将被称为`Connection`，`DataProcessor`和`ReportGenerator`。让我们看看在实现这三个类之后代码变得多么清晰。

在以下类中，你可以看到该类中的方法只与连接到数据源相关：

```cs
namespace CH3.Cohesion
{
     public class Connection
     {
         public void ConnectToDatasource() { }
         public void CloseConnectionToDataSource() { }
     }
}
```

类本身被命名为`Connection`，所以这是一个高内聚的类的例子。

在以下代码中，`DataProcessor`类包含两个方法，通过从数据源中提取数据并将数据转换为报告插入而处理数据：

```cs
namespace CH3.Cohesion
{
     public class DataProcessor
     {
         public void ExtractDataFromDataSource() { }
         public void TransformDataForReport() { }
     }
}
```

因此，这是另一个高内聚类的例子。

在以下代码中，`ReportGenerator`类只有与生成和输出报告相关的方法：

```cs
namespace CH3.Cohesion
{
    public class ReportGenerator
    {
        public void AssignDataAndGenerateReport() { }
        public void PrintReport() { }
    }
}
```

同样，这是另一个高内聚类的例子。

查看这三个类的每一个，我们可以看到它们只包含与其单一职责相关的方法。因此，这三个类都是高内聚的。

现在是时候看看我们如何通过使用接口而不是类来设计我们的代码，以便可以使用依赖注入和控制反转将代码注入到构造函数和方法中。

# 为变更设计

在设计变更时，你应该将*what*改变为*how*。

*what*是业务的需求。任何经验丰富的参与软件开发角色的人都会告诉你，需求经常变化。因此，软件必须能够适应这些变化。业务不关心软件和基础设施团队如何实现需求，只关心需求准确地按时和按预算完成。

另一方面，软件和基础设施团队更关注如何满足业务需求。无论采用何种技术和流程来实现需求，软件和目标环境必须能够适应不断变化的需求。

但这还不是全部。你会发现，软件版本经常因为错误修复和新功能而改变。随着新功能的实施和重构的进行，软件代码变得过时并最终过时。此外，软件供应商有软件路线图，这是他们应用生命周期管理的一部分。最终，软件版本会被淘汰，供应商不再支持。这可能会迫使从当前不再受支持的版本迁移到新支持的版本，这可能会带来必须解决的破坏性变化。

## 面向接口的编程

**面向接口的编程**（**IOP**）帮助我们编写多态代码。在面向对象编程中，多态性被定义为不同类具有相同接口的不同实现。因此，通过使用接口，我们可以改变软件以满足业务需求。

让我们考虑一个数据库连接的例子。一个应用程序可能需要连接到不同的数据源。但无论使用何种数据库，数据库代码如何保持不变呢？答案在于使用接口。

你有不同的数据库连接类，它们实现了相同的数据库连接接口，但它们各自有自己版本的实现方法。这就是多态。然后数据库接受一个数据库连接参数，该参数是数据库连接接口类型。然后你可以将任何实现数据库连接接口的数据库连接类型传递给数据库。让我们编写这个示例，以便更清楚地说明这些事情。

首先创建一个简单的.NET Framework 控制台应用程序。然后按照以下方式更新`Program`类：

```cs
static void Main(string[] args)
{
    var program = new Program();
    program.InterfaceOrientedProgrammingExample();
}

private void InterfaceOrientedProgrammingExample()
{
    var mongoDb = new MongoDbConnection();
    var sqlServer = new SqlServerConnection();
    var db = new Database(mongoDb);
    db.OpenConnection();
    db.CloseConnection();
    db = new Database(sqlServer);
    db.OpenConnection();
    db.CloseConnection();
}
```

在这段代码中，`Main()`方法创建了`Program`类的一个新实例，然后调用了`InterfaceOrientedProgrammingExample()`方法。在该方法中，我们实例化了两个不同的数据库连接，一个是 MongoDB，一个是 SQL Server。然后我们使用 MongoDB 连接实例化数据库，打开数据库连接，然后关闭它。然后我们使用相同的变量实例化一个新的数据库，并传入一个 SQL Server 连接，然后打开连接并关闭连接。正如你所看到的，我们只有一个`Database`类和一个构造函数，但`Database`类可以与实现所需接口的任何数据库连接一起工作。因此，让我们添加`IConnection`接口：

```cs
public interface IConnection
{
    void Open();
    void Close();
}
```

该接口只有两个名为`Open()`和`Close()`的方法。添加实现该接口的 MongoDB 类：

```cs
public class MongoDbConnection : IConnection
{
    public void Close()
    {
        Console.WriteLine("Closed MongoDB connection.");
    }

    public void Open()
    {
        Console.WriteLine("Opened MongoDB connection.");
    }
}
```

我们可以看到该类实现了`IConnection`接口。每个方法都会在控制台打印一条消息。现在添加`SQLServerConnection`类：

```cs
public class SqlServerConnection : IConnection
{
    public void Close()
    {
        Console.WriteLine("Closed SQL Server Connection.");
    }

    public void Open()
    {
        Console.WriteLine("Opened SQL Server Connection.");
    }
}
```

`Database`类也是一样。它实现了`IConnection`接口，对于每个方法调用，都会在控制台打印一条消息。现在来看`Database`类，如下所示：

```cs
public class Database
{
    private readonly IConnection _connection;

    public Database(IConnection connection)
    {
        _connection = connection;
    }

    public void OpenConnection()
    {
        _connection.Open();
    }

    public void CloseConnection()
    {
        _connection.Close();
    }
}
```

`Database`类接受一个`IConnection`参数。这设置了`_connection`成员变量。`OpenConnection()`方法打开数据库连接，`CloseConnection()`方法关闭数据库连接。现在是运行程序的时候了。你应该在控制台窗口中看到以下输出：

```cs
Opened MongoDB connection.
Closed MongoDB connection.
Opened SQL Server Connection.
Closed SQL Server Connection.
```

现在，你可以看到编程接口的优势。你可以看到它们如何使我们能够扩展程序，而无需修改现有的代码。这意味着如果我们需要支持更多的数据库，那么我们只需要编写更多实现`IConnection`接口的连接对象。

现在你知道了接口的工作原理，我们可以看看如何将它们应用到依赖注入和控制反转中。依赖注入帮助我们编写干净的、松耦合且易于测试的代码，而控制反转使得根据需要可以互换软件实现，只要这些实现实现了相同的接口。

## 依赖注入和控制反转

在 C#中，我们有能力使用**依赖注入**（**DI**）和**控制反转**（**IoC**）来应对不断变化的软件需求。这两个术语确实有不同的含义，但通常可以互换使用来表示相同的事物。

使用 IoC，你可以编写一个通过调用模块来完成任务的框架。IoC 容器用于保持模块的注册。这些模块在用户请求或配置请求它们时加载。

DI 将类的内部依赖项移除。依赖对象然后由外部调用者注入。IoC 容器使用 DI 将依赖对象注入到对象或方法中。

在本章中，你将找到一些有用的资源，这些资源将帮助你理解 IoC 和 DI。然后你将能够在你的程序中使用这些技术。

让我们看看如何在没有任何第三方框架的情况下实现我们自己的简单 DI 和 IoC。

## 依赖注入的示例

在这个例子中，我们将自己编写一个简单的 DI。我们将有一个`ILogger`接口，它将有一个带有字符串参数的单一方法。然后，我们将产生一个名为`TextFileLogger`的类，它实现了`ILogger`接口，并将一个字符串输出到文本文件。最后，我们将有一个`Worker`类，它将演示构造函数注入和方法注入。让我们看看代码。

以下接口有一个方法，将用于实现类根据方法的实现输出消息：

```cs
namespace CH3.DependencyInjection
{
     public interface ILogger
     {
         void OutputMessage(string message);
     }
}
```

`TexFileLogger`类实现了`ILogger`接口，并将消息输出到文本文件：

```cs
using System;

namespace CH3.DependencyInjection
{
    public class TextFileLogger : ILogger
    {
        public void OutputMessage(string message)
        {
            System.IO.File.WriteAllText(FileName(), message);
        }

        private string FileName()
        {
            var timestamp = DateTime.Now.ToFileTimeUtc().ToString();
            var path = Environment.GetFolderPath(Environment
             .SpecialFolder.MyDocuments);
            return $"{path}_{timestamp}";
        }
    }
}
```

`Worker`类提供了构造函数 DI 和方法 DI 的示例。请注意参数是一个接口。因此，任何实现该接口的类都可以在运行时注入：

```cs
namespace CH3.DependencyInjection
{
     public class Worker
     {
         private ILogger _logger;

         public Worker(ILogger logger)
         {
             _logger = logger;
             _logger.OutputMessage("This constructor has been injected 
              with a logger!");
         }

         public void DoSomeWork(ILogger logger)
         {
             logger.OutputMessage("This methods has been injected 
              with a logger!");
         }
     }
}
```

`DependencyInject`方法运行示例以展示 DI 的工作原理：

```cs
        private void DependencyInject()
        {
            var logger = new TextFileLogger();
            var di = new Worker(logger);
            di.DoSomeWork(logger);
        }
```

正如你在刚才看到的代码中所看到的，我们首先生成了`TextFileLogger`类的一个新实例。然后将这个对象注入到工作者的构造函数中。然后我们调用`DoSomeWork`方法并传入`TextFileLogger`实例。在这个简单的例子中，我们看到了如何通过构造函数和方法将代码注入到一个类中。

这段代码的好处在于它消除了工作者和`TextFileLogger`实例之间的依赖关系。这使得我们可以很容易地用实现`ILogger`接口的任何其他类型的记录器替换`TextFileLogger`实例。因此，我们可以使用，例如，事件查看器记录器或甚至数据库记录器。使用 DI 是减少代码耦合的好方法。

现在我们已经看到了 DI 的工作，我们也应该看看 IoC。我们现在就来看看。

## IoC 的一个例子

在这个例子中，我们将使用 IoC 容器注册依赖项。然后我们将使用 DI 来注入必要的依赖项。

在下面的代码中，我们有一个 IoC 容器。容器将依赖项注册到字典中，并从配置元数据中读取值：

```cs
using System;
using System.Collections.Generic;

namespace CH3.InversionOfControl
{
    public class Container
    {
        public delegate object Creator(Container container);

        private readonly Dictionary<string, object> configuration = new 
         Dictionary<string, object>();
        private readonly Dictionary<Type, Creator> typeToCreator = new 
         Dictionary<Type, Creator>();

        public Dictionary<string, object> Configuration
        {
            get { return configuration; }
        }

        public void Register<T>(Creator creator)
        {
            typeToCreator.Add(typeof(T), creator);
        }

        public T Create<T>()
        {
            return (T)typeToCreatortypeof(T);
        }

        public T GetConfiguration<T>(string name)
        {
            return (T)configuration[name];
        }
    }
}
```

然后，我们创建一个容器，并使用容器来配置元数据，注册类型，并创建依赖项的实例：

```cs
private void InversionOfControl()
{
    Container container = new Container();
    container.Configuration["message"] = "Hello World!";
    container.Register<ILogger>(delegate
    {
        return new TextFileLogger();
    });
    container.Register<Worker>(delegate
    {
        return new Worker(container.Create<ILogger>());
    });
}
```

接下来，我们将看看如何使用迪米特法则将对象的知识限制在只知道它的近亲。这将帮助我们编写一个干净的 C#代码，避免使用导航列车。

# 迪米特法则

迪米特法则旨在消除导航列车（点计数），并且还旨在提供松散耦合的良好封装代码。

理解导航列车的方法违反了迪米特法则。例如，看一下下面的代码：

```cs
report.Database.Connection.Open(); // Breaks the Law of Demeter.
```

代码的每个单元应该具有有限的知识量。这些知识应该只涉及相关的代码。根据迪米特法则，你必须告诉而不是询问。使用这个法则，你只能调用一个或多个以下对象的方法：

+   作为参数传递

+   本地创建

+   实例变量

+   全局变量

实施迪米特法则可能很困难，但告诉而不是询问有其优势。这样做的一个好处是解耦你的代码。

看到违反迪米特法则的坏例子以及遵守迪米特法则的例子是很好的，所以我们将在接下来的部分中看到这一点。

## 迪米特法则的好例子和坏例子（链接）

在好的例子中，我们有报告的实例变量。在报告变量对象实例上，调用了打开连接的方法。这不违反法律。

以下代码是一个`Connection`类，其中有一个打开连接的方法：

```cs
namespace CH3.LawOfDemeter
{
    public class Connection
    {
        public void Open()
        {
            // ... implementation ...
        }
    }
}
```

`Database`类创建一个新的`Connection`对象并打开连接：

```cs
namespace CH3.LawOfDemeter
{
    public class Database
    {
        public Database()
        {
            Connection = new Connection();
        }

        public Connection Connection { get; set; }

        public void OpenConnection()
        {
            Connection.Open();
        }
    }
}
```

在`Report`类中，实例化了一个`Database`对象，然后打开了与数据库的连接：

```cs
namespace CH3.LawOfDemeter
{
    public class Report
    {
        public Report()
        {
            Database = new Database();
        }

        public Database Database { get; set; }

        public void OpenConnection()
        {
            Database.OpenConnection();
        }
    }
}
```

到目前为止，我们已经看到了遵守迪米特法则的好代码。但以下是违反这一法则的代码。

在`Example`类中，迪米特法则被打破，因为我们引入了方法链，如`report.Database.Connection.Open()`：

```cs
namespace CH3.LawOfDemeter
{
    public class Example
    {
        public void BadExample_Chaining()
        {
            var report = new Report();
            report.Database.Connection.Open();
        }

        public void GoodExample()
        {
            var report = new Report();
            report.OpenConnection();
        }
    }
}
```

在这个糟糕的例子中，对报告实例变量调用了`Database` getter。这是可以接受的。但然后调用了返回不同对象的`Connection` getter。这违反了迪米特法则，最后调用打开连接也是如此。

# 不可变对象和数据结构

不可变类型通常被认为只是值类型。对于值类型，当它们被设置时，不希望它们发生变化是有道理的。但是您也可以有不可变对象类型和不可变数据结构类型。不可变类型是一种在初始化后其内部状态不会改变的类型。

不可变类型的行为不会使其他程序员感到惊讶，因此符合**最小惊讶原则**（**POLA**）。不可变类型的 POLA 符合度遵守与客户之间达成的任何合同，并且因为它是可预测的，程序员会发现很容易推断其行为。

由于不可变类型是可预测且不会改变，您不会遇到任何令人不快的惊喜。因此，您不必担心由于以某种方式被更改而导致的任何不良影响。这使得不可变类型非常适合在线程之间共享，因为它们是线程安全的，无需进行防御性编程。

当您创建一个不可变类型并使用对象验证时，您将获得一个在该对象的生命周期内有效的对象。

让我们看一个 C#中不可变类型的例子。

## 不可变类型的例子

现在我们将看一个不可变对象。以下代码中的`Person`对象有三个私有成员变量。这些变量只能在构造函数中设置。一旦设置，它们在对象的其余生命周期内将无法修改。每个变量只能通过只读属性进行读取：

```cs
namespace CH3.ImmutableObjectsAndDataStructures
{
    public class Person
    {
        private readonly int _id;
        private readonly string _firstName;
        private readonly string _lastName;

        public int Id => _id;
        public string FirstName => _firstName;
        public string LastName => _lastName;
        public string FullName => $"{_firstName} {_lastName}";
        public string FullNameReversed => $"{_lastName}, {_firstName}";

        public Person(int id, string firstName, string lastName)
        {
            _id = id;
            _firstName = firstName;
            _lastName = lastName;
        }
    }
}
```

现在我们已经看到编写不可变对象和数据结构有多么容易，我们将看看对象中的数据和方法。

# 对象应该隐藏数据并公开方法

对象的状态存储在成员变量中。这些成员变量是数据。数据不应直接可访问。您应该只通过公开的方法和属性提供对数据的访问。

为什么要隐藏数据并公开方法？

隐藏数据并公开方法在面向对象编程世界中被称为封装。封装将类的内部工作隐藏在外部世界之外。这使得更改值类型而不破坏依赖于该类的现有实现变得容易。数据可以被设置为可读/可写、可写或只读，这样可以更灵活地访问和使用数据。您还可以验证输入，从而防止数据接收无效值。封装还使得测试类变得更容易，并且可以使类更具可重用性和可扩展性。

让我们看一个例子。

## 封装的例子

以下代码示例显示了一个封装的类。`Car`对象是可变的。它具有在构造函数初始化后获取和设置数据值的属性。构造函数和设置属性执行参数的验证。如果值无效，则抛出无效参数异常，否则将传回值并设置数据值：

```cs
using System;

namespace CH3.Encapsulation
{
    public class Car
    {
        private string _make;
        private string _model;
        private int _year;

        public Car(string make, string model, int year)
        {
            _make = ValidateMake(make);
            _model = ValidateModel(model);
            _year = ValidateYear(year);
        }

        private string ValidateMake(string make)
        {
            if (make.Length >= 3)
                return make;
            throw new ArgumentException("Make must be three 
             characters or more.");
        }

        public string Make
        {
            get { return _make; }
            set { _make = ValidateMake(value); }
        }

        // Other methods and properties omitted for brevity.
    }
}
```

前面代码的好处是，如果您需要更改获取或设置数据值的代码的验证，您可以这样做而不会破坏实现。

# 数据结构应该公开数据并且没有方法

结构与类的不同之处在于它们使用值相等而不是引用相等。除此之外，结构和类之间没有太大的区别。

关于数据结构是否应该将变量公开还是隐藏在 get 和 set 属性后，存在争论。选择权完全取决于你，但我个人认为即使在结构中也最好隐藏数据，并且只通过属性和方法提供访问。在拥有干净的数据结构并且安全的情况下，有一个例外，那就是一旦创建，结构体不应允许方法和 get 属性对其进行改变。这样做的原因是对临时数据结构的更改将被丢弃。

现在让我们看一个简单的数据结构示例。

## 数据结构的一个例子

以下代码是一个简单的数据结构：

```cs
namespace CH3.Encapsulation
{
    public struct Person
    {
        public int Id { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }

        public Person(int id, string firstName, string lastName)
        {
            Id = id;
            FirstName = firstName;
            LastName = lastName;
        }
    }
}
```

正如你所看到的，数据结构与类并没有太大的不同，它有构造函数和属性。

通过这一章的学习，我们将回顾我们所学到的知识。

# 总结

在本章中，我们学习了如何在文件夹和包中组织我们的命名空间，以及良好的组织如何帮助防止命名空间类。然后我们转向类和责任，并探讨了为什么类应该只有一个责任。我们还研究了内聚性和耦合性，以及为什么具有高内聚性和低耦合性是重要的。

良好的文档需要对公共成员进行正确的注释，并且我们学习了如何使用 XML 注释来实现这一点。还讨论了为什么应该为更改而设计的重要性，并提供了 DI 和 IoC 的基本示例。

德米特法则告诉你不要与陌生人交谈，只与直接朋友交谈，以及如何避免链式调用。最后，我们研究了对象和数据结构，以及它们应该隐藏和公开的内容。

在下一章中，我们将简要介绍 C#中的函数式编程以及如何编写简洁的方法。我们还将学习避免在方法中使用超过两个参数，因为参数过多的方法会变得难以管理。此外，我们还将学习避免重复，因为重复可能是一个麻烦的错误源，即使在一个地方修复了，但在代码的其他地方仍然存在。

# 问题

1.  我们如何在 C#中组织我们的类？

1.  一个类应该有多少个责任？

1.  如何在代码中为文档生成器添加注释？

1.  内聚性是什么意思？

1.  耦合是什么意思？

1.  内聚性应该高还是低？

1.  耦合应该是紧密的还是松散的？

1.  有哪些机制可以帮助你设计以便进行更改？

1.  什么是 DI？

1.  什么是 IoC？

1.  使用不可变对象的一个好处是什么？

1.  对象应该隐藏和显示什么？

1.  结构应该隐藏和显示什么？

# 进一步阅读

+   要了解不同类型的内聚性和耦合性的更多细节，请查看[`www.geeksforgeeks.org/software-engineering-coupling-and-cohesion/`](https://www.geeksforgeeks.org/software-engineering-coupling-and-cohesion/)。

+   [可以在](https://www.tutorialsteacher.com/ioc/) [`www.tutorialsteacher.com/ioc/`](https://www.tutorialsteacher.com/ioc/) [找到许多关于 IoC 的教程。](https://www.tutorialsteacher.com/ioc/)
