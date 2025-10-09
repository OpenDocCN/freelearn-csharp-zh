# 8

# 使用 SOLID 原则避免代码反模式

正确的设计原则可以防止你的代码迅速过时。虽然编写代码有许多正确的方法，但也有一些反模式和代码异味构成了错误的编写方式。

此外，社区已经确定了一些原则，在构建软件时应牢记，这有助于你的代码尽可能长时间地抵抗技术债务的积累。在本章中，我们将介绍这些原则中的许多，包括著名的 SOLID 缩写，并探讨它们如何帮助你构建能够积极抵抗逐渐走向过时代码的软件。

在本章中，我们将介绍以下主题：

+   识别 C#代码中的反模式

+   编写 SOLID 代码

+   考虑其他架构原则

# 识别 C#代码中的反模式

我经常发现自己告诉新程序员，要构建好的软件，你必须首先构建大量的非常糟糕的软件，并从中学习。

虽然这个说法有些玩笑成分，但其中确实有一些真理：几乎每个开发者都能识别出编写错误的方式，并发现使其难以工作的因素，这样做有助于你在下一次编写更好的代码。

当你的代码质量不佳时，你内心通常会有所察觉。你会看到一些你不喜欢的细节：重复的代码片段、命名或参数顺序的不一致性、传递过多的参数、方法，甚至是一些太大而难以有效管理的类。

这些症状就是我们通常所说的**代码异味**，我们将在本节稍后重新讨论。

代码异味之外，还有一种称为**反模式**的东西，即与社区建议显著偏离的代码。不幸的是，并非所有反模式都容易察觉或自行发现，有些甚至在完全探索之前似乎对个人或团队来说是好主意。

我看到的一些常见的 C#反模式包括抛出和捕获一个`Exception`错误而不是特定类型的`Exception`错误，没有释放实现`IDisposable`接口的资源，以及效率低下的**语言集成查询**（**LINQ**）语句。关于这些反模式的更多细节，请参阅本章的*进一步阅读*部分。

在这本书中，要涵盖的反模式太多了，而且.NET 开发的既定实践随着时间的推移而演变。正因为这种持续的变化，Visual Studio 提供了代码分析工具，以帮助发现和修复违反社区标准的行为。这些工具包括代码分析规则集和内置的**Roslyn 分析器**，我们将在*第十二章*，*Visual Studio 中的代码分析*中更详细地介绍。

代码中的问题并不都是针对 C# 代码的。许多问题源于类之间的交互、相互传递数据、管理变量以及一般的结构。这些问题甚至在你开始看到系统规模随着新功能的添加而扩大时，也会出现在你打算“结构良好”的代码中。

幸运的是，即使是新开发者也天生具有识别难以遵循的代码、需要比应有的更多工作来维护和扩展的代码，或者涉及过度重复的代码的能力。这些类型的代码问题通常被称为 **代码异味**。

代码异味是什么？

代码异味是当前架构存在一些缺陷且需要进行重构的强烈指标。当你遇到这些症状时，包括你编写的代码，请注意这些症状。了解使代码难以工作的原因将帮助你编写更好的代码，并将现有代码重构为更好的形式。

现在，让我们继续讨论编写 **SOLID 代码**，这可以帮助你避免一些常见的代码异味，并构建健壮、可维护和可测试的代码。

# 编写 SOLID 代码

**SOLID** 是由迈克尔·费瑟斯（Michael Feathers）引入的一个缩写，总结了罗伯特·C·马丁（Robert C. Martin）的话。SOLID 的目的是为开发者提供一套原则，引导他们编写更易于维护且能抵抗技术债务的代码。

SOLID 代码的五个原则是：

+   **单一职责** **原则** （**SRP**）

+   **开闭** **原则** （**OCP**）

+   **里氏替换** **原则** （**LSP**）

+   **接口隔离** **原则** （**ISP**）

+   **依赖倒置** **原则** （**DIP**）

在本节中，我们将涵盖这五个原则。

## 单一职责原则

**单一职责原则** （**SRP**）表示一个类应该只负责一件事情。以下是一些遵循 SRP 的类的例子：

+   负责将应用程序数据保存到特定文件格式的类

+   专门用于对数据库表或一组表执行查询的数据库访问类

+   提供 REST 方法以与飞行数据交互的 API 控制器

+   代表你应用程序特定部分的用户界面的类

类通过在同一个类中尝试做多种类型的事情来违反 SRP。更正式地说，如果修改一个类的原因超过一个，那么这个类就违反了 SRP。

例如，如果一个类负责跟踪用户界面中一组项目的状态、响应用户按钮点击、解析用户输入以及异步获取数据，那么这个类很可能违反了 SRP。

违反单一职责原则（SRP）的类往往会被频繁修改，随着时间的推移复杂性增加，并且与其他系统中的类相比，它们会变得非常大。这些类可能难以完全理解或充分测试，并且随着复杂性的增加，可能会变得脆弱和充满错误。

我用来帮助检测 SRP 违反的一种方法是添加一个类级别注释，讨论类的责任。例如，以下 XML 注释描述了本书第一部分的`FlightScheduler`类：

```cs
/// <summary>
/// This class is responsible for tracking information
/// about current and pending flights
/// </summary>
public class FlightScheduler {
  // Details omitted
}
```

在这里，`FlightScheduler`类的责任是明确的：它存在是为了跟踪系统中的活跃和待处理的航班。修改此类的原因应与这些航班的跟踪相关，而不是与其他主题相关。

因此，每当我在定义一个新类时，我倾向于在所有类中添加类级别注释，以帮助该类在其生命周期中保持对其任务的专注。

但如果你有一个已经存在且违反 SRP（单一责任原则）的类怎么办？

当你有一个负责多项事物的类时，我喜欢查看该类目前负责的所有内容，并将它们分组到相关的成员组中。例如，如果一个类有 10 个字段，25 个方法，和 6 个属性，我可能会逐一检查它们，并试图找到那些事物共同解决的问题。

例如，如果`FlightScheduler`类违反了 SRP，它可能包含以下成员：

+   航班调度和取消

+   为航班分配机组人员

+   为乘客预订航班

+   更改乘客的座位分配

+   将乘客转移到不同的航班

+   为管理层生成航班调度文档

显然，这个类负责多种类型的事物。在一个生产系统中，这个类可能长达 2,000 行或更多，难以完全理解和充分测试。此外，对类的一个区域的更改可能会以意想不到的方式影响其他区域。

通过查看一个类处理的事物的组，你通常可以识别出几个关键组。我喜欢这样做，然后专注于那些与类的核心目标不明确相关的最大相关责任组。一旦确定了这些分组，你就可以提取一个新的类来管理这些方面。你的原始类可以引用这个类，或者如果需要，将其存储为字段，或者新类可以完全独立于旧类运行。

在`FlightScheduler`的情况下，我会说航班调度和取消是类的核心部分，而类中目前的其他方面可能更适合放在别处。查看那些其他区域，有几个与为乘客管理航班预订相关的事物，因此在这种情况下，可能需要引入一个`FlightBookingManager`类来包含这些相关的逻辑片段。

通过迭代地引入与类核心责任无关的新类，你可以将大型类缩小到可管理的规模，并抵抗那些在忽略 SRP（单一责任原则）的类中发现的复杂性、质量和可测试性问题。

单一职责原则不仅适用于类，也适用于方法。一个方法应该有一个它负责的单个核心任务，并且这个目的应该通过方法名来传达。当一个方法负责多个事物或开始变得太大时，这可能是一个很好的迹象，表明你可能需要提取一个方法并将一些逻辑从原始方法中拉出来，以保持方法的大小可维护。

个人而言，如果我能向年轻时的自己——或者大多数早期/中级开发者传授一个编程原则，那将是单一职责原则（SRP）在保持代码易于理解、测试、扩展和维护方面的重要性。

我个人的指导原则是努力使类不超过 200 行代码，方法不超过 20 行代码，但这都是挑战，根据你维护的代码的性质，这些指导原则可能会有例外——记住，这些是原则和指导原则，而不是严格规则或戒律。

*如果你只记得 SOLID 原则中的一个，那就记住单一职责原则（SRP）；它对你的应用程序的健康至关重要。* 然而，还有四个原则需要探索。

## 开放封闭原则

当类*开放于扩展*但*封闭于修改*时，我们说它们遵循**开放封闭原则（OCP）**。

这个原则最初是为 C++模块编写的，它不像其他 SOLID 原则那样干净地转换为 C#，但这本质上是一个关于在设计类时遵循**面向对象编程（OOP）**原则的原则。

如果你构建了遵循开放封闭原则的东西，你就是在设计一个类，使其行为可以通过继承它的其他类、可定制的属性或参数，或者通过组合（将你的类组合成其他对象，这些对象会改变其行为）来扩展。

在*第五章*中，我们讨论了使用组合的例子：面向对象重构，并涉及到为航班提供不同的货物项目。

本节剩余部分将专注于使用继承来实现开放封闭原则。

在 C#中，默认情况下方法不允许被重写。这意味着你需要明确选择允许他人通过将它们声明为`virtual`来重写你的方法。

反驳观点

我听说一些开发者认为，在没有类重写的情况下将方法声明为`virtual`会让人困惑，给代码添加了不必要的关键字，甚至略微损害了代码在运行时的性能。所有这些事情都可能成立，但如果你处于一个无法预测他人如何使用你的代码，并且你知道他们无法修改你的源代码的场景中，将关键方法标记为`virtual`通常是一个好主意。在这些场景中，`virtual`提供了额外的灵活性。

记住，SOLID 原则是在构建软件时需要记住的*指导原则*，而不是你需要始终遵循的严格规则。

作为 OCP 的一个具体示例，让我们看看一个代表乘客通过 Cloudy Skies Airlines 旅行的航班行程信息的`ItineraryManager`类样本：

```cs
public class ItineraryManager {
  public int MilesAccumulated {get; private set;}
  public FlightInfo? Flight {get; private set;}
  public virtual void FlightCompleted(FlightInfo? next) {
    if (Flight != null) {
      AccumulateMiles(Flight.Miles);
    }
    Flight = next;
  }
  public virtual void ChangeFlight(FlightInfo newFlight,
    bool isInvoluntary) =>
    Flight = newFlight;
  public void AccumulateMiles(int miles) =>
    MilesAccumulated += miles;
}
```

在这里，我们有一个跟踪乘客累积的总里程以及乘客计划乘坐的下一航班（当他们的旅行完成时，这可能是`null`）的类。该类有两个与处理完成的航班以及取消的航班相关的`virtual`方法。此外，该类还有一个非`virtual`方法`AccumulateMiles`，它更新乘客在本次旅行中累积的里程。

虽然这个类满足了航空公司的需求，但假设航空公司想引入一种新的逻辑来奖励客户，每当客户完成一次航班时，他们可以获得 100 额外里程，并且当乘客被强制转移到新的航班时，奖励该航班的预定里程。

在 OCP 的指导下，如果我们假设类是可修改的，我们应该能够这样做而不必修改我们的基类。结果是我们可以用以下`RewardsItineraryManager`类来实现这一点：

```cs
public class RewardsItineraryManager : ItineraryManager {
  private const int BonusMilesPerFlight = 100;
  public override void FlightCompleted(FlightInfo? next) {
    base.FlightCompleted(next);
    AccumulateMiles(BonusMilesPerFlight);
  }
  public override void ChangeFlight(FlightInfo newFlight,       bool isInvoluntary) {
    if (isInvoluntary && Flight != null) {
       AccumulateMiles(Flight.Miles);
    }
    base.ChangeFlight(newFlight, isInvoluntary);
  }
}
```

在不修改我们的基类的情况下，我们可以通过我们的新类扩展`ItineraryManager`的实现，这个新类遵循略微不同的逻辑。多态的神奇之处在于，我们可以在接受`ItineraryManager`类的地方使用`RewardsItineraryManager`类，进一步支持 OCP 的封闭性原则。

## Liskov 替换原则

**Liskov 替换原则**（**LSP**）表示，多态代码不应该需要知道它正在处理的具体类型的对象。

这仍然是一个相当模糊的描述，所以让我们再次看看之前的`FlightCompleted`方法：

```cs
public virtual void FlightCompleted(FlightInfo? next) {
  if (Flight != null) {
    AccumulateMiles(Flight.Miles);
  }
  Flight = next;
}
```

此方法接收一个航班并将其存储在`Flight`属性中。如果之前在该`Flight`属性中存储了航班，代码将调用该航班的`Miles`属性调用`AccumulateMiles`方法。

应用程序有多个从`FlightInfo`继承的类：`PassengerFlightInfo`和`CargoFlightInfo`。这意味着我们的`next`参数可能是这三个类中的任何一个——或者是从它们继承的任何其他类。

LSP 表示，任何有效的`FlightInfo`实例在调用其`Miles`属性（或任何其他方法）时都不应该出错。例如，这个版本的`CargoFlightInfo`会违反 LSP，因为当调用其`Miles`属性时会出错：

```cs
public class CargoFlightInfo : FlightInfo {
  public decimal TonsOfCargo { get; set; }
  public override int RewardMiles =>
    throw new NotSupportedException();
}
```

实质上，遵循 LSP 时，方法不应该有任何理由需要知道它正在处理`FlightInfo`的哪个子类。

因为 LSP 关注多态性，所以它适用于.NET 代码中的类继承和接口实现。

说到接口，让我们继续讨论 ISP。

## 接口隔离原则

**接口分离原则**（**ISP**）是一种优雅的说法，意味着你应该优先选择许多专注于相关功能的小型专业接口，而不是一个包含你类所做所有事情的庞大接口。

例如，假设我们有一个`FlightRepository`类，该类管理对单个航班的数据库访问。在许多系统中，这个类可能实现了一个`IFlightRepository`接口，该接口可以定义如下，其中类中的所有公共成员都是接口的一部分：

```cs
public interface IFlightRepository {
  FlightInfo AddFlight(FlightInfo flight);
  FlightInfo UpdateFlight(FlightInfo flight);
  void CancelFlight(FlightInfo flight);
  FlightInfo? FindFlight(string id);
  IEnumerable<FlightInfo> GetActiveFlights();
  IEnumerable<FlightInfo> GetPendingFlights();
  IEnumerable<FlightInfo> GetCompletedFlights();
}
```

如您所见，这管理了与航班相关的常见操作，并提供了一些查找许多航班信息的方式。在一个更真实世界的例子中，可能需要多年内添加许多额外的功能来支持新特性。

在我的.NET 代码经验中，每个主要类通常都有一个包含该类所有公共方法的庞大接口。这个接口通常以它所基于的类命名，并且主要存在是为了通过**依赖注入**（**DI**）来支持可测试性，我们将在下一章中涉及。

然而，这种方法通常违反了 ISP。因为我们的接口是围绕类而不是离散的功能集设计的，所以引入一个满足某些但不是所有这些功能的新类变得更加困难。

例如，假设 Cloudy Skies Airlines 想要与另一家子公司航空公司的系统集成。它不需要添加、更新或删除航班，但它确实想要一种搜索航班的方式。在`IFlightRepository`接口下，`AddFlight`、`UpdateFlight`和`CancelFlight`方法可能需要什么也不做，或者在调用时抛出`NotSupportedException`错误。顺便说一句，在更大的接口中抛出不支持的方法调用异常，将违反之前提到的 LSP。

与每个主要类型对应一个庞大接口的做法不同，ISP 提倡为紧密相关的功能使用小型接口。在`FlightRepository`的例子中，它本质上做两件事：

+   添加、编辑和删除航班

+   搜索现有航班

如果我们想要引入接口，我们可以为这些相关的独立功能集引入接口，如下所示：

```cs
public interface IFlightUpdater {
  FlightInfo AddFlight(FlightInfo flight);
  FlightInfo UpdateFlight(FlightInfo flight);
  void CancelFlight(FlightInfo flight);
}
public interface IFlightProvider {
  FlightInfo? FindFlight(string id);
  IEnumerable<FlightInfo> GetActiveFlights();
  IEnumerable<FlightInfo> GetPendingFlights();
  IEnumerable<FlightInfo> GetCompletedFlights();
}
```

在这个例子中，我们的`FlightRepository`类将实现`IFlightUpdater`接口和`IFlightProvider`接口。如果我们想要与另一家航空公司的系统集成，但没有修改其航班的能力，那么`IFlightProvider`接口可以不实现`IFlightUpdater`接口。

通过将我们的接口分割成表示不同功能集的小接口，我们使得提供这些功能的替代实现以及之后测试我们的代码变得更加容易。

我们已经几次提到了依赖注入（DI）；让我们通过涵盖 DIP 来更详细地探讨这个主题，并完善我们的 SOLID 原则。

## 依赖倒置原则

**依赖倒置原则**（**DIP**）指出，你的代码通常应该依赖于抽象而不是依赖于具体的实现。

为了说明这一点，让我们看看一个`FlightBookingManager`类，它帮助乘客预订航班。这个类需要注册预订请求并发送预订确认消息。这是它的当前代码：

```cs
public class FlightBookingManager {
  private readonly SpecificMailClient _email;
  public FlightBookingManager(string connectionString) {
    _email = new SpecificMailClient(connectionString);
  }
  public bool BookFlight(Passenger passenger,
    PassengerFlightInfo flight, string seat) {
    if (!flight.IsSeatAvailable(seat)) {
      return false;
    }
    flight.AssignSeat(passenger, seat);
    string message = "Your seat is confirmed";
    _email.SendMessage(passenger.Email, message);
    return true;
  }
}
```

这段代码允许乘客通过检查座位是否可用，然后预订该座位并使用`_email`字段发送消息来预订航班。这个字段在构造函数中设置为`SpecificMailClient`的新实例，这是一个虚构的类，代表电子邮件客户端的一些非常具体的实现。构造函数需要获取连接字符串来实例化这个类。

这段代码违反了依赖倒置原则（DIP），因为我们的`FlightBookingManager`类与一个特定的电子邮件客户端紧密耦合。如果我们想对这个类编写单元测试，这个类总是会尝试向那个电子邮件客户端发送消息，这在测试时通常不是你想要的。

此外，如果组织想要更换电子邮件提供商，你需要切换到不同的电子邮件客户端，`FlightBookingManager`类将需要随着系统中任何其他与`SpecificMailClient`类紧密耦合的地方一起改变。

依赖倒置通过让我们的类依赖它们所依赖的具体事物的抽象来实现这一点。这通常是通过依赖一个基类，如`EmailClientBase`，然后继承它，或者通过接受一个接口，如`IEmailClient`，具体客户端可以实现它。

我们通常将这些依赖作为构造函数参数接受。我们`FlightBookingManager`类的这个版本看起来会是这样：

```cs
public class FlightBookingManager {
  private readonly IEmailClient _email;
  public FlightBookingManager(IEmailClient email) {
    _email = email;
  }
  public bool BookFlight(Passenger passenger,
    PassengerFlightInfo flight, string seat) {
    if (!flight.IsSeatAvailable(seat)) {
      return false;
    }
    flight.AssignSeat(passenger, seat);
    string message = "Your seat is confirmed";
    _email.SendMessage(passenger.Email, message);
    return true;
  }
}
```

在这里，我们不再接受连接字符串，而是接受一个`IEmailClient`类。这意味着我们的类不需要知道它正在处理哪种实现，也不需要知道如何实例化该类的实例，不需要连接字符串，不需要在特定的电子邮件提供商更改时进行更改，并且可以通过传递一个模拟电子邮件客户端而不是真实的一个来更容易地进行测试（我们将在下一章讨论 Moq 时更多地讨论这一点）。

从其他东西中接受依赖的过程被称为**依赖倒置**，对于新和中级开发者来说，这通常是一个令人畏惧的话题，但就其核心而言，依赖倒置完全是关于类从外部获取它们的依赖，而不是必须自己创建特定的实例。

遵循 DIP 会导致代码更加易于维护、灵活和可测试。

这就结束了 SOLID 中的五个原则，但在结束这一章之前，我们还有几个设计原则要介绍。

# 考虑其他架构原则

在我们结束这一章之前，让我分享三个在我自己的软件开发之旅中帮助我的简要原则。

## 学习 DRY 原则

**不要重复自己**（**DRY**）是软件开发中的一个重要原则。DRY 原则的核心是确保你不会在应用程序的代码中重复相同的模式。编写代码需要花费时间，阅读和维护也需要时间，而且错误不可避免地会在每行代码中以一定的速率发生。因此，你希望努力在一个集中的地方一次性解决问题，然后重用那个解决方案。

让我们看看一些违反 DRY 原则的示例代码。这段代码接受一个`"CSA1234,CMH,ORD"`并将其转换为`FlightInfo`对象：

```cs
public FlightInfo ReadFlightFromCsv(string csvLine) {
  string[] parts = csvLine.Split(',');
  const string fallback = "Unknown";
  FlightInfo flight = new();
  if (parts.Length > 0) {
    flight.Id = parts[0]?.Trim() ?? fallback;
  } else {
    flight.Id = fallback;
  }
  if (parts.Length > 1) {
    flight.DepartureAirport = parts[1]?.Trim() ?? fallback;
  } else {
    flight.DepartureAirport = fallback;
  }
  if (parts.Length > 2) {
    flight.ArrivalAirport = parts[2]?.Trim() ?? fallback;
  } else {
    flight.ArrivalAirport = fallback;
  }
  // Other parsing logic omitted
  return flight;
}
```

注意到解析 CSV 字符串每一部分的逻辑是如何被包裹在针对`null`值和部分数组为空的检查中的。这段代码非常重复，很容易想象如果 CSV 数据中添加了新字段，进行更改的开发者会直接复制和粘贴这五行代码。

重复代码模式，如这个问题，有几个问题：

+   它鼓励复制和粘贴，这往往会产生糟糕的代码或由于在粘贴时应该更改而未更改的事情而导致错误

+   如果解析单个字段的逻辑需要改变（例如，为了防止空字符串），现在需要在很多地方进行更改

我们可以通过提取包含解析字段逻辑的方法来解决这个问题：

```cs
private string ReadFromCsv(string[] parts, int index,
  string fallback = "Unknown") {
  if (parts.Length > index) {
    return parts[index]?.Trim() ?? fallback;
  } else {
    return fallback;
  }
}
public FlightInfo ReadFlightFromCsv(string csvLine) {
  string[] parts = csvLine.Split(',');
  FlightInfo flight = new();
  flight.Id = ReadFromCsv(parts, 0);
  flight.DepartureAirport = ReadFromCsv(parts, 1);
  flight.ArrivalAirport = ReadFromCsv(parts, 2);
  // Other parsing logic omitted
  return flight;
}
```

不仅这个新版本更容易维护，而且它还减少了代码量，并有助于将注意力集中在不同部分之间逻辑不同的部分。这提高了代码的可读性，同时减少了你犯错误的可能性。

## KISS 原则

“**保持简单，傻瓜**”（缩写为**KISS**，有时也称为“**保持简单，愚蠢**”）是一个关注软件系统复杂性的原则。作为软件工程师，我们有时会过度思考，使事情变得极其复杂，而实际上并不需要这样。KISS 原则鼓励你尽可能保持你的代码和类简单，只有在真正必要时才扩展复杂性。

通常，你的系统中复杂性越高，添加新功能、诊断问题、 onboard 新团队成员和解决面向客户的问题所需的时间就越长。随着应用程序中移动部件的增加，也有更多可能出错的事情，这意味着复杂性有真正的机会创建面向客户的问题——所有这些都是为了解决你组织几年内可能不会遇到的问题的潜在解决方案。

复杂性往往会随着时间的推移而增长，很少会减少（尤其是在数据库模式中）。保持简单，直到你看到迫切和有说服力的理由来增加复杂性。

## 理解高内聚和低耦合

最后，让我们通过回顾你在软件工程中偶尔会听到的两个术语来结束本章：**内聚**和**耦合**。

内聚与类的不同部分如何与同一事物相关联有关。在一个高内聚的类中，几乎所有的类部分都面向同一类型的功能。让我们再次看看之前提到的`IFlightUpdater`接口作为例子：

```cs
public interface IFlightUpdater {
  FlightInfo AddFlight(FlightInfo flight);
  FlightInfo UpdateFlight(FlightInfo flight);
  void CancelFlight(FlightInfo flight);
}
```

一个实现了这个接口中的所有内容且没有添加其他成员的类是一个很好的**高内聚**例子，因为该接口中的所有成员都与处理同一类型的项目相关。一个**低内聚**的类会从这些方法开始，但也会添加许多与预订航班、生成报告、搜索数据或其他功能相关的方法。

具有低内聚的类通常也违反了 SRP。

耦合指的是单个类与其他类紧密配对的程度。一个类需要了解的其他类越多，它的耦合度就越高。具有更高耦合度的类由于依赖项较多，测试起来更困难，并且随着相关类随时间演变而需要更频繁地修改。

DIP 为类提供了一个很好的方法来减少它们的耦合。

因此，当你听到人们谈论希望有高内聚和低耦合时，他们是在提倡具有非常专注于特定领域并且尽可能少地依赖其他类的类。当这种组合满足时，类往往非常专注且易于维护。

# 摘要

在本章中，我们讨论了代码异味和反模式。正确的设计原则可以帮助保持你的代码集中和最小化，并减缓它自然积累复杂性的速度。这有助于保持你的代码良好形态并抵抗技术债务的积累。

质量编程最常见的原则是 SOLID，遵循单一职责原则（SRP），使代码易于扩展而难以修改，Liskov 替换原则（LSP）提倡使用多态代码实现低耦合，接口隔离原则关注多个较小的接口而不是一个较大的接口，以及依赖倒置原则（DIP），它讨论通过让类从类外获取它们所需的东西来减少耦合。

现在我们已经确定了如何编写 SOLID 代码，我们将探讨一些高级测试技术，这些技术可以帮助测试使用这些原则构建的代码。

# 问题

1.  SRP 如何影响内聚性？

1.  你的代码中哪些区域违反了 SRP 或 DRY？

1.  DI 的优势是什么？它是如何影响耦合的？

# 进一步阅读

你可以在以下 URL 中找到有关本章讨论的材料更多信息：

+   *C#中的 SOLID 原则及示例*：[`www.c-sharpcorner.com/UploadFile/damubetha/solid-principles-in-C-Sharp/`](https://www.c-sharpcorner.com/UploadFile/damubetha/solid-principles-in-C-Sharp/)

+   *开发者持续使用的 15 个最糟糕的 C#反模式（以及如何避免它们）*：[`methodpoet.com/worst-anti-patterns/`](https://methodpoet.com/worst-anti-patterns/)

+   *C#中的 Top 10 Dotnet 异常反模式*：[`newdevsguide.com/2022/11/06/exception-anti-patterns-in-csharp/`](https://newdevsguide.com/2022/11/06/exception-anti-patterns-in-csharp/)

+   *使用实现*IDisposable*的对象*：[`learn.microsoft.com/en-us/dotnet/standard/garbage-collection/using-objects`](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/using-objects)

+   *LINQ 的注意事项和陷阱*：[`dev.to/samfieldscc/linq-37k3`](https://dev.to/samfieldscc/linq-37k3)
