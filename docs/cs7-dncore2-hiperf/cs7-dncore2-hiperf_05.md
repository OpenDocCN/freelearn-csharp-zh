# 第五章：.NET Core 应用程序性能设计指南

架构和设计是任何应用程序的核心基础。遵循最佳实践和指南使应用程序具有高可维护性、高性能和可扩展性。应用程序可以是基于 Web 的应用程序、Web API、服务器/客户端基于 TCP 的消息传递应用程序、关键任务应用程序等等。然而，所有这些应用程序都应该遵循一定的实践，从而在各种方面获益。在本章中，我们将学习几种几乎所有应用程序中常见的实践。

以下是本章将学习的一些原则：

+   编码原则：

+   命名约定

+   代码注释

+   每个文件一个类

+   每个方法一个逻辑

+   设计原则：

+   KISS（保持简单，愚蠢）

+   YAGNI（你不会需要它）

+   DRY（不要重复自己）

+   关注点分离

+   SOLID 原则

+   缓存

+   数据结构

+   通信

+   资源管理

+   并发

# 编码原则

在本节中，我们将介绍一些基本的编码原则，这些原则有助于编写提高应用程序整体性能和可扩展性的优质代码。

# 命名约定

在每个应用程序中始终使用适当的命名约定，从解决方案名称开始，解决方案名称应提供有关您正在工作的项目的有意义的信息。项目名称指定应用程序的层或组件部分。最后，类应该是名词或名词短语，方法应该代表动作。

当我们在 Visual Studio 中创建一个新项目时，默认的解决方案名称设置为您为项目名称指定的内容。解决方案名称应始终与项目名称不同，因为一个解决方案可能包含多个项目。项目名称应始终代表系统的特定部分。例如，假设我们正在开发一个消息网关，该网关向不同的方发送不同类型的消息，并包含三个组件，即监听器、处理器和调度器；监听器监听传入的请求，处理器处理传入的消息，调度器将消息发送到目的地。命名约定可以如下：

+   解决方案名称：`MessagingGateway`（或任何代码词）

+   监听器项目名称：`ListenerApp`

+   处理器项目名称：`ProcessorAPI`（如果是 API）

+   调度项目名称：`DispatcherApp`

在.NET 中，我们通常遵循的命名约定是类和方法名称使用帕斯卡命名法。在帕斯卡命名法中，每个单词的第一个字符都是大写字母，而参数和其他变量则使用骆驼命名法。以下是一些示例代码，显示了在.NET 中应如何使用命名法。

```cs
public class MessageDispatcher 
{ 
  public const string SmtpAddress = "smpt.office365.com"; 

  public void SendEmail(string fromAddress, string toAddress, 
  string subject, string body) 
  { 

  } 
}
```

在上述代码中，我们有一个常量字段`SmtpAddress`和一个使用帕斯卡命名法的`SendEmail`方法，而参数则使用骆驼命名法。

以下表格总结了.NET 中不同工件的命名约定：

| **属性** | **命名约定** | **示例** |
| --- | --- | --- |
| 类 | 帕斯卡命名法 | `class PersonManager {}` |
| 方法 | 帕斯卡命名法 | `void SaveRecord(Person person) {}` |
| 参数/成员变量 | 骆驼命名法 | `bool isActive;` |
| 接口 | 帕斯卡命名法；以字母 I 开头 | `IPerson` |
| 枚举 | 帕斯卡命名法 | `enum Status {InProgress, New, Completed}` |

# 代码注释

任何包含适当注释的代码都可以在许多方面帮助开发人员。它不仅减少了理解代码的时间，还可以利用诸如*Sandcastle*或*DocFx*之类的工具，在生成完整的代码文档时即时共享给团队中的其他开发人员。此外，在谈论 API 时，Swagger 在开发人员社区中被广泛使用和受欢迎。Swagger 通过提供有关 API 的完整信息，可用方法，每个方法所需的参数等，来赋予 API 使用者权力。Swagger 还读取这些注释，以提供完整的文档和接口，以测试任何 API。

# 每个文件一个类

与许多其他语言不同，在.NET 中，我们不受限于为每个类创建单独的文件。我们可以创建一个单独的`.cs`文件，并在其中创建多个类。相反，这是一种不好的做法，当处理大型应用程序时会很痛苦。

# 每个方法一个逻辑

始终编写一次只执行一件事的方法。假设我们有一个方法，它从数据库中读取用户 ID，然后调用 API 来检索用户上传的文档列表。在这种情况下，最好的方法是有两个单独的方法，`GetUserID`和`GetUserDocuments`，分别首先检索用户 ID，然后检索文档：

```cs
public int GetUserId(string userName) 
{ 
  //Get user ID from database by passing the username 
} 

public List<Document> GetUserDocuments(int userID) 
{ 
  //Get list of documents by calling some API 
} 
```

这种方法的好处在于减少了代码重复。将来，如果我们想要更改任一方法的逻辑，我们只需在一个地方进行更改，而不是在所有地方复制它并增加错误的机会。

# 设计原则

遵循最佳实践开发清晰的架构会带来多种好处，应用程序性能就是其中之一。我们经常看到，应用程序背后使用的技术是强大而有效的，但应用程序的性能仍然不尽人意或不佳，这通常是因为糟糕的架构设计和在应用程序设计上投入较少的时间。

在这一部分，我们将讨论一些在.NET Core 中设计和开发应用程序时应该解决的常见设计原则：

+   KISS（保持简单，愚蠢）

+   YAGNI（你不会需要它）

+   DRY（不要重复自己）

+   关注点分离

+   SOLID 原则

+   缓存

+   数据结构

+   通信

+   资源管理

+   并发

# KISS（保持简单，愚蠢）

编写更清洁的代码并始终保持简单有助于开发人员在长期内理解和维护它。在代码中添加不必要的复杂性不仅使其难以理解，而且在需要时也难以维护和更改。这就是 KISS 所说的。在软件上下文中，KISS 可以在设计软件架构时考虑，使用**面向对象原则**（**OOP**），设计数据库，用户界面，集成等。添加不必要的复杂性会使软件的设计复杂化，并可能影响应用程序的可维护性和性能。

# YAGNI（你不会需要它）

YAGNI 是 XP（极限编程）的核心原则之一。XP 是一种软件方法，包含短期迭代，以满足客户需求，并在需要或由客户发起时欢迎变更。主要目标是满足客户的期望，并保持客户所需的质量和响应能力。它涉及成对编程和代码审查，以保持质量完整，并满足客户的期望。

YAGNI 最适合极限编程方法，该方法帮助开发人员专注于应用程序功能或客户需求的特性。做一些额外的事情，如果没有告知客户或不是迭代或需求的一部分，最终可能需要重新工作，并且会浪费时间。

# DRY（不要重复自己）

DRY（不要重复自己）也是编写更清晰代码的核心原则之一。它解决了开发人员在大型应用程序中不断更改或扩展功能或基础逻辑时所面临的挑战。根据该原则，它规定“系统中的每个知识片段必须有一个可靠的表示”。

在编写应用程序时，我们可以使用抽象来避免代码的重复，以避免冗余。这有助于适应变化，并让开发人员专注于需要更改的一个领域。如果相同的代码在多个地方重复，那么在一个地方进行更改需要在其他地方进行更改，这会消除良好的架构实践，从而引发更高的错误风险，并使应用程序代码更加错误。

# 关注点分离（SoC）

开发清晰架构的核心原则之一是**关注点分离**（**SoC**）。这种模式规定，每种不同类型的应用程序应该作为一个独立的组件单独构建，与其他组件几乎没有或没有紧密耦合。例如，如果一个程序将用户消息保存到数据库，然后一个服务随机选择消息并选择获胜者，你可以看到这是两个独立的操作，这就是所谓的关注点分离。通过关注点分离，代码被视为一个独立的组件，如果需要，任何定制都可以在一个地方完成。可重用性是另一个因素，它帮助开发人员在一个地方更改代码，以便在多个地方使用。然而，测试要容易得多，而且在出现问题的情况下，错误可以被隔离和延后修复。

# SOLID 原则

SOLID 是 5 个原则的集合，列举如下。这些是在开发软件设计时经常使用的常见设计原则：

+   **单一责任原则**（**SRP**）

+   **开闭原则**（**OCP**）

+   **里氏替换原则**（**LSP**）

+   **接口隔离原则**（**ISP**）

+   **依赖倒置原则**（**DIP**）

# 单一责任原则

单一责任原则规定类应该只有一个特定的目标，并且该责任应该完全封装在类中。如果有任何更改或需要适应新目标，应创建一个新的类或接口。

在软件设计中应用这一原则使我们的代码易于维护和理解。架构师通常在设计软件架构时遵循这一原则，但随着时间的推移，当许多开发人员在该代码/类中工作并进行更改时，它变得臃肿，并且违反了单一责任原则，最终使我们的代码难以维护。

这也涉及到内聚性和耦合的概念。内聚性指的是类中责任之间的关联程度，而耦合指的是每个类相互依赖的程度。我们应该始终专注于保持类之间的低耦合和类内的高内聚。

这是一个基本的`PersonManager`类，包含四个方法，即`GetPerson`、`SavePerson`、`LogError`和`LogInformation`：

![](img/00056.gif)

所有这些方法都使用数据库持久性管理器来读取/写入数据库中的记录。正如你可能已经注意到的那样，`LogError`和`LogInformation`与`PersonManager`类的内聚性不高，并且与`PersonManager`类紧密耦合。如果我们想在其他类中重用这些方法，我们必须使用`PersonManager`类，并且更改内部日志记录的逻辑也需要更改`PersonManager`类。因此，`PersonManager`违反了单一责任原则。

为了修复这个设计，我们可以创建一个单独的`LogManager`类，可以被`PersonManager`使用来记录执行操作时的信息或错误。下面是更新后的类图，表示关联关系：

![](img/00057.gif)

# 开闭原则

根据定义，开闭原则规定，类、方法、接口等软件实体应该对修改封闭，对扩展开放。这意味着我们不能修改现有代码，并通过添加额外的类、接口、方法等来扩展功能，以应对任何变化。

在任何应用程序中使用这个原则可以解决各种问题，列举如下：

+   在不改变现有代码的情况下添加新功能会产生更少的错误，并且不需要彻底测试

+   更少的涟漪效应通常在更改现有代码以添加或更新功能时经历

+   扩展通常使用新接口或抽象类来实现，其中现有代码是不必要的，而且破坏现有功能的可能性较小

为了实现开闭原则，我们应该使用抽象化，这是通过参数、继承和组合方法实现的。

# 参数

方法中可以设置特殊参数，用于控制该方法中编写的代码的行为。假设有一个`LogException`方法，它将异常保存到数据库，并发送电子邮件。现在，每当调用这个方法时，两个任务都会执行。没有办法从代码中停止发送电子邮件来处理特定的异常。然而，如果以某种方式表达，并使用一些参数来决定是否发送电子邮件，就可以控制。然而，如果现有代码不支持这个参数，那么就需要定制，但是在设计时，我们可以采用这种方法来暴露某些参数，以便处理方法的内部行为：

```cs
public void LogException(Exception ex) 
{ 
  SendEmail(ex); 
  LogToDatabase(ex); 
} 
```

推荐的实现如下：

```cs
public void LogException(Exception ex, bool sendEmail, bool logToDb) 
{ 
  if (sendEmail) 
  { 
    SendEmail(ex); 
  } 

  if (logToDb) 
  { 
    LogToDatabase(ex); 
  } 
}
```

# 继承

使用继承方法，我们可以使用模板方法模式。使用模板方法模式，我们可以在根类中创建默认行为，然后创建子类来覆盖默认行为并实现新功能。

例如，这里有一个`Logger`类，它将信息记录到文件系统中：

```cs
public class Logger 
{ 
  public virtual void LogMessage(string message) 
  { 
    //This method logs information into file system 
    LogToFileSystem(message); 
  } 

  private void LogtoFileSystem(string message) { 
    //Log to file system 
  } 
} 
```

我们有一个`LogMessage`方法，通过调用`LogToFileSystem`方法将消息记录到文件系统中。这个方法一直工作得很好，直到我们想要扩展功能。假设，以后我们提出了将这些信息也记录到数据库的要求。我们必须更改现有的`LogMessage`方法，并将代码编写到同一个类中。以后，如果出现其他要求，我们必须一遍又一遍地添加功能并修改这个类。根据开闭原则，这是一种违反。

使用模板方法模式，我们可以重新设计这段代码，遵循开闭原则，使其对扩展开放，对定制封闭。

遵循 OCP，这里是新设计，我们有一个包含`LogMessage`抽象方法的抽象类，以及两个具有自己实现的子类：

![](img/00058.gif)

有了这个设计，我们可以在不改变现有`Logger`类的情况下添加第 n 个扩展：

```cs
public abstract class Logger 
{ 
  public abstract void LogMessage(string message); 

} 

public class FileLogger : Logger 
{ 
  public override void LogMessage(string message) 
  { 
    //Log to file system 
  } 
} 

public class DatabaseLogger : Logger 
{ 
  public override void LogMessage(string message) 
  { 
    //Log to database 
  } 
} 

```

# 组合

第三种方法是组合，这可以通过策略模式实现。通过这种方法，客户端代码依赖于抽象，实际实现封装在一个单独的类中，该类被注入到暴露给客户端的类中。

让我们看一个实现策略模式的例子。基本要求是发送可能是电子邮件或短信的消息，并且我们需要以一种方式构造它，以便将来可以添加新的消息类型而不对主类进行任何修改：

![](img/00059.gif)

根据策略模式，我们有一个`MessageStrategy`抽象类，它公开一个抽象方法。每种工作类型都封装到继承`MessageStrategy`基本抽象类的单独类中。

这是`MessageStrategy`抽象类的代码：

```cs
public abstract class MessageStrategy 
{ 
  public abstract void SendMessage(Message message); 
}
```

我们有两个`MessageStrategy`的具体实现；一个用于发送电子邮件，另一个用于发送短信，如下所示：

```cs
public class EmailMessage : MessageStrategy 
{ 
  public override void SendMessage(Message message) 
  { 
    //Send Email 
  } 
} 

public class SMSMessage : MessageStrategy 
{ 
  public override void SendMessage(Message message) 
  { 
    //Send SMS  
  } 
} 
```

最后，我们有`MessageSender`类，客户端将使用它。在这个类中，客户端可以设置消息策略并调用`SendMessage`方法，该方法调用特定的具体实现类型来发送消息：

```cs
public class MessageSender 
{ 
  private MessageStrategy _messageStrategy; 
  public void SetMessageStrategy(MessageStrategy messageStrategy) 
  { 
    _messageStrategy = messageStrategy; 
  } 

  public void SendMessage(Message message) 
  { 
    _messageStrategy.SendMessage(message); 
  } 

} 

```

从主程序中，我们可以使用`MessageSender`，如下所示：

```cs
static void Main(string[] args) 
{ 
  MessageSender sender = new MessageSender(); 
  sender.SetMessageStrategy(new EmailMessage()); 
  sender.SendMessage(new Message { MessageID = 1, MessageTo = "jason@tfx.com", 
  MessageFrom = "donotreply@tfx.com", MessageBody = "Hello readers", 
  MessageSubject = "Chapter 5" }); 
}
```

# Liskov 原则

根据 Liskov 原则，通过基类对象使用派生类引用的函数必须符合基类的行为。

这意味着子类不应该删除基类的行为，因为这违反了它的不变性。通常，调用代码应该完全依赖于基类中公开的方法，而不知道其派生实现。

让我们举一个例子，首先违反 Liskov 原则的定义，然后修复它以了解它特别设计用于什么：

![](img/00060.gif)

`IMultiFunctionPrinter`接口公开了两种方法，如下所示：

```cs
public interface IMultiFunctionPrinter 
{ 
  void Print(); 
  void Scan(); 
}
```

这是一个可以由不同类型的打印机实现的接口。以下是实现`IMultiFunctionPrinter`接口的两种打印机，它们分别是：

```cs
public class OfficePrinter: IMultiFunctionPrinter 
{ 
  //Office printer can print the page 
  public void Print() { } 
  //Office printer can scan the page 
  public void Scan() { } 
} 

public class DeskjetPrinter : IMultiFunctionPrinter 
{ 
  //Deskjet printer print the page 
  public void Print() { } 
  //Deskjet printer does not contain this feature 
  public void Scan() => throw new NotImplementedException(); 
}
```

在前面的实现中，我们有一个提供打印和扫描功能的`OfficePrinter`，而另一个家用`DeskjetPrinter`只提供打印功能。当调用`Scan`方法时，`DeskjetPrinter`实际上违反了 Liskov 原则，因为它会抛出`NotImplementedException`。

作为对前面问题的补救，我们可以将`IMultiFunctionPrinter`拆分为两个接口，即`IPrinter`和`IScanner`，而`IMultiFunctionPrinter`也可以实现这两个接口以支持两种功能。`DeskjetPrinter`只实现了`IPrinter`接口，因为它不支持扫描：

![](img/00061.gif)

这是三个接口`IPrinter`，`IScanner`和`IMultiFunctionPrinter`的代码：

```cs
public interface IPrinter 
{ 
  void Print(); 
} 

public interface IScanner 
{ 
  void Scanner(); 
} 

public interface MultiFunctionPrinter : IPrinter, IScanner 
{  

} 
```

最后，具体实现将如下所示：

```cs
public class DeskjetPrinter : IPrinter 
{ 
  //Deskjet printer print the page 
  public void Print() { } 
} 

public class OfficePrinter: IMultiFunctionPrinter 
{ 
  //Office printer can print the page 
  public void Print() { } 
  //Office printer can scan the page 
  public void Scan() { } 
}
```

# 接口隔离原则

接口隔离原则规定，客户端代码只应依赖于客户端使用的东西，不应依赖于他们不使用的任何东西。这意味着你不能强迫客户端代码依赖于不需要的某些方法。

让我们举一个首先违反接口隔离原则的例子：

![](img/00062.gif)

在前面的图表中，我们有一个包含两种方法`WriteLog`和`GetLogs`的 ILogger 接口。`ConsoleLogger`类将消息写入应用程序控制台窗口，而`DatabaseLogger`类将消息存储到数据库中。`ConsoleLogger`在控制台窗口上打印消息并不持久化它；对于`GetLogs`方法，它抛出`NotImplementedException`，因此违反了接口隔离原则。

这是前面问题的代码：

```cs
public interface ILogger 
{ 
  void WriteLog(string message); 
  List<string> GetLogs(); 
} 

/// <summary> 
/// Logger that prints the information on application console window 
/// </summary> 
public class ConsoleLogger : ILogger 
{ 
  public List<string> GetLogs() => throw new NotImplementedException(); 
  public void WriteLog(string message) 
  { 
    Console.WriteLine(message); 
  } 
} 

/// <summary> 
/// Logger that writes the log into database and persist them 
/// </summary> 
public class DatabaseLogger : ILogger 
{ 
  public List<string> GetLogs() 
  { 
    //do some work to get logs stored in database, as the actual code 
    //in not written so returning null 
    return null;  
  } 
  public void WriteLog(string message) 
  { 
    //do some work to write log into database 
  } 
}
```

为了遵守**接口隔离原则**（**ISP**），我们分割了 ILogger 接口，并使其更精确和相关于其他实现者。ILogger 接口将仅包含`WriteLog`方法，并引入了一个新的`IPersistenceLogger`接口，它继承了 ILogger 接口并提供了`GetLogs`方法：

![](img/00063.gif)

以下是修改后的示例，如下所示：

```cs
public interface ILogger 
{ 
  void WriteLog(string message); 

} 

public interface PersistenceLogger: ILogger 
{ 
  List<string> GetLogs(); 
} 

/// <summary> 
/// Logger that prints the information on application console window 
/// </summary> 
public class ConsoleLogger : ILogger 
{ 
  public void WriteLog(string message) 
  { 
    Console.WriteLine(message); 
  } 
} 

/// <summary> 
/// Logger that writes the log into database and persist them 
/// </summary> 
public class DatabaseLogger : PersistenceLogger 
{ 
  public List<string> GetLogs() 
  { 
    //do some work to get logs stored in database, as the actual code 
    //in not written so returning null 
    return null; 
  } 
  public void WriteLog(string message) 
  { 
    //do some work to write log into database 
  } 
}
```

# 依赖倒置原则

依赖倒置原则规定，高级模块不应依赖于低级模块，它们两者都应该依赖于抽象。

软件应用程序包含许多类型的依赖关系。依赖关系可以是框架依赖关系、第三方库依赖关系、Web 服务依赖关系、数据库依赖关系、类依赖关系等。根据依赖倒置原则，这些依赖关系不应该紧密耦合在一起。

例如，在分层架构方法中，我们有一个表示层，其中定义了所有视图；服务层公开了表示层使用的某些方法；业务层包含系统的核心业务逻辑；数据库层定义了后端数据库连接器和存储库类。将其视为 ASP.NET MVC 应用程序，其中控制器调用服务，服务引用业务层，业务层包含系统的核心业务逻辑，并使用数据库层对数据库执行 CRUD（创建、读取、更新和删除）操作。依赖树将如下所示：

![](img/00064.gif)

根据依赖倒置原则，不建议直接从每个层实例化对象。这会在层之间创建紧密耦合。为了打破这种耦合，我们可以通过接口或抽象类实现抽象化。我们可以使用一些实例化模式，如工厂或依赖注入来实例化对象。此外，我们应该始终使用接口而不是类。假设在我们的服务层中，我们引用了我们的业务层，并且我们的服务契约正在使用`EmployeeManager`来执行一些 CRUD 操作。`EmployeeManager`包含以下方法：

```cs
public class EmployeeManager 
{ 

  public List<Employee> GetEmployees(int id) 
  { 
    //logic to Get employees 
    return null; 
  } 
  public void SaveEmployee(Employee emp) 
  { 
    //logic to Save employee 
  } 
  public void DeleteEmployee(int id) 
  { 
    //Logic to delete employee 
  } 

} 
```

在服务层中，我们可以使用 new 关键字实例化业务层`EmployeeManager`对象。在`EmployeeManager`类中添加更多方法将直接基于访问修饰符在服务层中使用。此外，对现有方法的任何更改都将破坏服务层代码。如果我们将接口暴露给服务层并使用一些工厂或**依赖注入**（**DI**）模式，它将封装底层实现并仅暴露所需的方法。

以下代码显示了从`EmployeeManager`类中提取出`IEmployeeManager`接口：

```cs
public interface IEmployeeManager 
{ 
  void DeleteEmployee(int id); 
  System.Collections.Generic.List<Employee> GetEmployees(int id); 
  void SaveEmployee(Employee emp); 
}
```

考虑到上述示例，我们可以使用依赖注入来注入类型，因此每当服务管理器被调用时，业务管理器实例将被初始化。

# 缓存

缓存是可以用来提高应用程序性能的最佳实践之一。它通常与数据一起使用，其中更改不太频繁。有许多可用的缓存提供程序，我们可以考虑使用它们来保存数据并在需要时检索数据。它比数据库操作更快。在 ASP.NET Core 中，我们可以使用内存缓存，它将数据存储在服务器的内存中，但对于部署到多个地方的 Web 农场或负载平衡场景，建议使用分布式缓存。Microsoft Azure 还提供了 Redis 缓存，它是一个分布式缓存，提供了一个端点，可以用来在云上存储值，并在需要时检索。

要在 ASP.NET Core 项目中使用内存缓存，我们可以简单地在`ConfigureServices`方法中添加内存缓存，如下所示：

```cs
public void ConfigureServices(IServiceCollection services) 
{ 
  services.AddMvc(); 
  services.AddMemoryCache(); 
}
```

然后，我们可以通过依赖注入在我们的控制器或页面模型中注入`IMemoryCache`，并使用`Set`和`Get`方法设置或获取值。

# 数据结构

选择正确的数据结构在应用程序性能中起着至关重要的作用。在选择任何数据结构之前，强烈建议考虑它是否是一种负担，或者它是否真正解决了特定的用例。在选择适当的数据结构时需要考虑的一些关键因素如下：

+   了解您需要存储的数据类型

+   了解数据增长的方式以及在增长时是否存在任何缺点

+   了解是否需要通过索引或键/值对访问数据，并选择适当的数据结构

+   了解是否需要同步访问，并选择线程安全的集合

选择正确的数据结构时还有许多其他因素，这些因素已经在第四章中涵盖，*C#中的数据结构和编写优化代码。*

# 通信

如今，通信已经成为任何应用程序中的重要缩影，主要因素是技术的快速发展。诸如基于 Web 的应用程序、移动应用程序、物联网应用程序和其他分布式应用程序在网络上执行不同类型的通信。我们可以以一个应用程序为例，该应用程序在某个云实例上部署了 Web 前端，调用了云中另一个实例上部署的某个服务，并对本地托管的数据库执行一些后端连接。此外，我们可以有一个物联网应用程序，通过互联网调用某个服务发送室温，等等。设计分布式应用程序时需要考虑的某些因素如下：

# 使用轻量级接口

避免多次往返服务器造成更多的网络延迟，降低应用程序性能。使用工作单元模式避免向服务器发送冗余操作，并执行一次单一操作以与后端服务通信。工作单元将所有消息分组为一个单元并将它们作为一个单元进行处理。

# 最小化消息大小

尽量减少与服务通信的数据量。例如，有一个 Person API 提供一些`GET`、`POST`、`PUT`和`DELETE`方法来对后端数据库执行 CRUD 操作。要删除一个人的记录，我们可以只传递该人的`ID`（主键）作为参数传递给服务，而不是将整个对象作为参数传递。此外，使用少量属性或方法的对象，提供最小的工件集。最好的情况是使用**POCO**（**Plain Old CLR object**）实体，它们对其他对象的依赖性很小，只包含必须发送到网络的属性。

# 排队通信

对于较大的对象或复杂操作，将单一的请求/响应通道与分布式消息通道解耦会提高应用程序的性能。对于大型、笨重的操作，我们可以将通信设计和分发到多个组件中。例如，有一个网站调用一个服务来上传图像，一旦上传完成，它会进行一些处理以提取缩略图并将其保存在数据库中。一种方法是在单个调用中同时进行上传和处理，但有时当用户上传较大的图像或图像处理需要更长时间时，用户可能会遇到请求超时异常，请求将终止。

通过排队架构，我们可以将这两个操作分开进行。用户上传图像，该图像将保存在文件系统中，并且图像路径将保存到存储中。后台运行的服务将获取该文件并异步进行处理。与此同时，当后端服务在处理时，控制权将返回给用户，用户可以看到一些正在进行的通知。最后，当缩略图生成时，用户将收到通知：

![](img/00065.jpeg)

# 资源管理

每台服务器都有一组有限的资源。无论服务器规格多么好，如果应用程序没有设计成以高效的方式利用资源，就会导致性能问题。在设计.NET Core 应用程序时，有一些需要注意的最佳实践来最大程度地利用服务器资源。

# 避免线程的不当使用

为每个任务创建一个新线程，而不监视或中止线程的生命周期是一种不好的做法。线程适合执行多任务和利用服务器的多个资源并行运行。然而，如果设计是为每个请求创建线程，这会减慢应用程序的性能，因为 CPU 在线程之间切换的上下文中花费的时间比执行实际工作更多。

每当使用线程时，我们应该尽量保持一个共享的线程池，任何需要执行的新项目都会在队列中等待，如果线程忙碌，则在可用时获取。这样，线程管理就变得简单，服务器资源也会被有效利用。

# 及时释放对象

**CLR**（**公共语言运行时**）提供自动内存管理，使用 new 关键字实例化的对象不需要显式进行垃圾回收；**GC**（**垃圾回收**）会处理。然而，非托管资源不会被 GC 自动释放，应该通过实现`IDisposable`接口来显式进行回收。这些资源可能是数据库连接、文件处理程序、套接字等。要了解更多关于在.NET Core 中处理非托管资源的信息，请参考第六章，*在.NET Core 中的内存管理技术*。

# 在需要时获取资源

只有在需要时才获取资源。提前实例化对象不是一个好的做法。这会占用不必要的内存并利用系统资源。此外，使用*try*、*catch*和*finally*来阻塞和释放*finally*块中的对象。这样，如果发生任何异常，方法内部实例化的对象将被释放。

# 并发

在并发编程中，许多对象可能同时访问同一资源，保持它们线程安全是主要目标。在.NET Core 中，我们可以使用锁来提供同步访问。然而，有些情况下，线程必须等待较长时间才能访问资源，这会导致应用程序无响应。

最佳实践是仅对那些需要线程安全的特定代码行应用同步访问，例如可以使用锁的地方，这些是数据库操作、文件处理、银行账户访问以及应用程序中许多其他关键部分。这些需要同步访问，以便一次处理一个线程。

# 总结

编写更清洁的代码，遵循架构和设计原则，并遵循最佳实践在应用程序性能中起着重要作用。如果代码臃肿和重复，会增加错误的机会，增加复杂性，并影响性能。

在本章中，我们学习了一些编码原则，使应用程序代码看起来更清晰，更容易理解。如果代码干净，它可以让其他开发人员完全理解，并在许多其他方面提供帮助。随后，我们学习了一些被认为是设计应用程序时的核心原则的基本设计原则。诸如 KISS、YAGNI、DRY、关注分离和 SOLID 等原则在软件设计中非常重要，缓存和选择正确的数据结构对性能有重大影响，如果使用得当可以提高性能。最后，我们学习了一些在处理通信、资源管理和并发时应考虑的最佳实践。

下一章是对内存管理的详细介绍，在这里我们将探讨.NET Core 中的一些内存管理技术。
