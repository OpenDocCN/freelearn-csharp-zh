# 第三章：编写可测试的代码

在第一章中，*探索测试驱动开发*，解释了编写代码以防止代码异味的陷阱。编写良好的代码本身就是一种艺术，而编写可以有效测试的代码的过程需要开发人员额外的努力和承诺，以编写可以反复测试而不费吹灰之力的干净代码。

练习 TDD 可以提高代码生产效率，鼓励编写健壮且易于维护的良好代码是事实。然而，如果参与软件项目的开发人员编写不可测试的代码，那么花在 TDD 上的时间可能是浪费的，该技术的投资回报可能无法实现。这通常可以追溯到使用糟糕的代码设计架构，以及未充分或有效地使用面向对象设计原则。

编写测试和编写主要代码一样重要。为不可测试的代码编写测试非常累人且非常困难，这就是为什么首先应该避免不可测试的代码的原因。代码之所以不可测试，可能有不同的原因，比如代码做得太多（**怪兽代码**），违反了单一职责原则，架构使用错误，或者面向对象设计有缺陷。

在本章中，我们将涵盖以下主题：

+   编写不可测试代码的警告信号

+   迪米特法则

+   SOLID 架构原则

+   为 ASP.NET Core MVC 设置 DI 容器

# 编写不可测试代码的警告信号

有效和持续的 TDD 实践可以改善编写代码的过程，使测试变得更容易，从而提高代码质量和软件应用的健壮性。然而，当项目的代码库包含不可测试的代码部分时，编写单元测试或集成测试变得极其困难，甚至几乎不可能。

当软件项目的代码库中存在不可测试的代码时，软件开发团队无法明确验证应用程序功能和特性的一致行为。为了避免这种可预防的情况，编写可测试的代码不是一个选择，而是每个重视质量软件的严肃开发团队的必须。

不可测试的代码是由于违反了已被证明和测试可以提高代码质量的常见标准、实践和原则而产生的。虽然专业素养随着良好实践和经验的反复使用而来，但有一些常见的糟糕代码设计和编写方法即使对于初学者来说也是常识，比如在不需要时使用全局变量、代码的紧耦合、硬编码依赖关系或可能在代码中发生变化的值。

在本节中，我们将讨论一些常见的反模式和陷阱，当编写代码时应该注意，因为它们可能会使为生产代码编写测试变得困难。

# 紧耦合

**耦合**是对象相互依赖或密切相关的程度。进一步解释，当`LoanProcessor`类与`EligibilityChecker`紧密耦合时，更改后者可能会影响前者的行为或修改其状态。

大多数不可测试的代码通常是由于不同部分的代码中存在的固有依赖关系造成的，通常是通过使用依赖关系的具体实现，导致了本应在应用程序边界上分离的关注点混合在一起。

具有紧密耦合依赖关系的单元测试代码将导致测试紧密耦合的不同对象。在单元测试期间，应该在构造函数中注入的依赖关系理想情况下应该很容易模拟，但这将是不可能的。这通常会减慢整体测试过程，因为所有依赖关系都必须在受测试的代码中构建。

在以下代码片段中，`LoanProcessor` 与 `EligibilityChecker` 紧密耦合。这是因为 `EligibilityChecker` 在 `LoanProcessor` 构造函数中使用了 new 关键字进行实例化。对 `EligibilityChecker` 的更改将影响 `LoanProcessor`，可能导致其出现故障。此外，对 `LoanProcessor` 中包含的任何方法进行单元测试都将导致 `EligibilityChecker` 被构造：

```cs
public class LoanProcessor
{
    private EligibilityChecker eligibilityChecker;

    public LoanProcessor()
    {
       eligibilityChecker= new EligibilityChecker();
    }        

    public void ProcessCustomerLoan(Loan loan)
    {
       throw new NotImplementedException();
    }    
}
```

解决 `LoanProcessor` 中紧密耦合的一种方法是使用**依赖注入**（**DI**）。由于 `LoanProcessor` 无法在隔离环境中进行测试，因为 `EligibilityChecker` 对象将必须在构造函数中实例化，所以可以通过构造函数将 `EligibilityChecker` 注入到 `LoanProcessor` 中：

```cs
public class LoanProcessor
{
    private EligibilityChecker eligibilityChecker;

    public LoanProcessor(EligibilityChecker eligibilityChecker)
    {
       this.eligibilityChecker= eligibilityChecker;
    }        

    public void ProcessCustomerLoan(Loan loan)
    {
       bool isEligible=eligibilityChecker.CheckLoan(loan);
       throw new NotImplementedException();
    }    
}
```

通过注入 `EligibilityChecker`，测试 `LoanProcessor` 变得更容易，因为这使您可以编写一个测试，其中模拟 `EligibilityChecker` 的实现，从而允许您在隔离环境中测试 `LoanProcessor`。

另外，可以通过 `LoanProcessor` 类的属性或成员注入 `EligibilityChecker`，而不是通过 `LoanProcessor` 构造函数传递依赖项：

```cs
public class LoanProcessor
{
    private EligibilityChecker eligibilityChecker;

    public EligibilityChecker EligibilityCheckerObject 
    {
        set { eligibilityChecker = value; }
    }     

    public void ProcessCustomerLoan(Loan loan)
    {
       bool isEligible=eligibilityChecker.CheckLoan(eligibilityChecker);
       throw new NotImplementedException();
    }    
}
```

通过构造函数或属性注入依赖后，`LoanProcessor` 和 `EligibilityChecker` 现在变得松散耦合，从而使得编写单元测试和模拟 `EligibilityChecker` 变得容易。

要使类松散耦合且可测试，必须确保该类不实例化其他类和对象。在类的构造函数或方法中实例化对象可能会导致无法注入模拟或虚拟对象，从而使代码无法进行测试。

# 怪物构造函数

要测试一个方法，您必须实例化或构造包含该方法的类。开发人员最常见的错误之一是创建我所谓的**怪物构造函数**，它只是一个做了太多工作或真正工作的构造函数，比如执行 I/O 操作、数据库调用、静态初始化、读取一些大文件或与外部服务建立通信。

当一个类设计有一个构造函数，用于初始化或实例化除值对象（列表、数组和字典）之外的对象时，该类在技术上具有非灵活的结构。这是糟糕的类设计，因为该类自动与其实例化的类紧密耦合，使得单元测试变得困难。具有这种设计的任何类也违反了单一责任原则，因为对象图的创建是可以委托给另一个类的责任。

在具有做大量工作的构造函数的类中测试方法会带来巨大的成本。实质上，要测试具有上述设计的类中的方法，您被迫要经历在构造函数中创建依赖对象的痛苦。如果依赖对象在构造时进行数据库调用，那么每次测试该类中的方法时，这个调用都会被重复，使得测试变得缓慢和痛苦：

```cs
public class LoanProcessor
{
    private EligibilityChecker eligibilityChecker;
    private CurrencyConverter currencyConverter;

    public LoanProcessor()
    {
       eligibilityChecker= new EligibilityChecker();
       currencyConverter = new CurrencyConverter();
       currencyConverter.DownloadCurrentRates();
       eligibilityChecker.CurrentRates= currencyConverter.Rates;
    }
}
```

在上述代码片段中，对象图的构建是在 `LoanProcessor` 构造函数中完成的，这肯定会使得该类难以测试。最好的做法是拥有一个精简的构造函数，它做很少的工作，并且对其他对象的了解很少，特别是它们能做什么，但不知道它们是如何做到的。

有时开发人员使用一种测试技巧，即为一个类创建多个构造函数。其中一个构造函数将被指定为仅用于测试的构造函数。虽然使用这种方法可以使类在隔离环境中进行测试，但也存在不好的一面。例如，使用多个构造函数创建的类可能会被其他类引用，并使用做大量工作的构造函数进行实例化。这可能会使得测试这些依赖类变得非常困难。

以下代码片段说明了为了测试类而创建单独构造函数的糟糕设计：

```cs
public class LoanProcessor
{
    private EligibilityChecker eligibilityChecker;
    private CurrencyConverter currencyConverter;

    public LoanProcessor()
    {
       eligibilityChecker= new EligibilityChecker();
       currencyConverter = new CurrencyConverter();
       currencyConverter.DownloadCurrentRates();
       eligibilityChecker.CurrentRates= currencyConverter.Rates;
    } 

    // constructor for testing
    public LoanProcessor(EligibilityChecker eligibilityChecker,CurrencyConverter currencyConverter)
    {
       this.eligibilityChecker= eligibilityChecker;
       this.currencyConverter = currencyConverter;
    }
}
```

有一些重要的警告信号可以帮助您设计一个构造函数工作量较小的松散耦合类。避免在构造函数中使用`new`操作符，以允许注入依赖对象。您应该初始化并分配通过构造函数注入的所有对象到适当的字段中。轻量级值对象的实例化也应该在构造函数中完成。

此外，应避免静态方法调用，因为静态调用无法被注入或模拟。此外，应避免在构造函数中使用迭代或条件逻辑；每次测试类时，逻辑或循环都将被执行，导致过多的开销。

在设计类时要考虑测试，不要在构造函数中创建依赖对象或协作者。当您的类需要依赖其他类时，请注入依赖项。确保只创建值对象。在代码中创建对象图时，使用*工厂方法*来实现。工厂方法用于创建对象。

# 具有多个责任的类

理想情况下，一个类应该只有一个责任。当您设计的类具有多个责任时，可能会在类之间产生交互，使得代码修改变得困难，并且几乎不可能对交互进行隔离测试。

有一些指标可以清楚地表明一个类做了太多事情并且具有多个责任。例如，当您在为一个类命名时感到困难，最终可能会在类名中使用`and`这个词，这表明该类做了太多事情。

一个具有多个责任的类的另一个标志是，类中的字段仅在某些方法中使用，或者类具有仅对参数而不是类字段进行操作的静态方法。此外，当一个类具有长列表的字段或方法以及许多依赖对象传递到类构造函数中时，表示该类做了太多事情。

在以下片段中，`LoanProcessor`类的依赖项已经整洁地注入到构造函数中，使其与依赖项松散耦合。然而，该类有多个改变的原因；该类既包含用于数据检索的代码，又包含业务规则处理的代码：

```cs
public class LoanProcessor
{
    private EligibilityChecker eligibilityChecker;
    private DbContext dbContext;

    public LoanProcessor(EligibilityChecker eligibilityChecker, DbContext dbContext)
    {
       this.eligibilityChecker= eligibilityChecker;
       this.dbContext= dbContext;
    }

    public double CalculateCarLoanRate(Loan loan)
    {
        double rate=12.5F;
        bool isEligible=eligibilityChecker.IsApplicantEligible(loan);
        if(isEligible)
          rate=rate-loan.DiscountFactor; 
        return rate;
    }

    public List<CarLoan> GetCarLoans()
    {
        return dbContext.CarLoan;
    }          
}
```

为了使类易于维护并且易于测试，`GetCarLoans`方法不应该在`LoanProcessor`中。应该将`LoanProcessor`与`GetCarLoans`一起重构到数据访问层类中。

具有本节描述的特征的类可能很难进行调试和测试。新团队成员可能很难快速理解类的内部工作原理。如果您的代码库中有具有这些属性的类，建议通过识别责任并将其分离到不同的类中，并根据其责任命名类来进行重构。

# 静态对象

在代码中使用**静态变量**、**方法**和**对象**可能是有用的，因为这些允许对象在所有实例中具有相同的值，因为只创建了一个对象的副本并放入内存中。然而，测试包含静态内容的代码，特别是静态方法的代码，可能会产生测试问题，因为您无法在子类中覆盖静态方法，并且使用模拟框架来模拟静态方法是一项非常艰巨的任务：

```cs
public static class LoanProcessor
{
    private static EligibilityChecker eligibilityChecker= new EligibilityChecker();

    public static double CalculateCarLoanRate(Loan loan)
    {
        double rate=12.5F;
        bool isEligible=eligibilityChecker.IsApplicantEligible(loan);
        if(isEligible)
          rate=rate-loan.DiscountFactor; 
        return rate;
    }     
}
```

当您创建维护状态的静态方法时，例如在前面片段中的`LoanProcessor`中的`CalculateCarLoanRate`方法，静态方法无法通过多态进行子类化或扩展。此外，静态方法无法使用接口进行定义，因此使得模拟变得不可能，因为大多数模拟框架都有效地使用接口。

# 迪米特法则

软件应用程序是由不同组件组成的复杂系统，这些组件进行通信以实现解决现实生活问题和业务流程自动化的整体目的。实际上，这些组件必须共存、互动，并在组件边界之间共享信息，而不会混淆不同的关注点，以促进组件的可重用性和整体系统的灵活性。

在软件编程中，技术上没有严格遵循的硬性法律。然而，已经制定了各种原则和法律，作为指导方针，可以帮助软件开发人员和从业者，促进构建具有高内聚性和松耦合性的组件的软件应用程序，以充分封装数据，并确保产生易于理解和扩展的高质量源代码，从而降低软件的维护成本。其中之一就是**迪米特法则**（**LoD**）。

LoD，也称为**最少知识原则**，是开发面向对象软件应用程序的重要设计方法或规则。该规则于 1987 年由 Ian Holland 在东北大学制定。通过正确理解这一原则，软件开发人员可以编写易于测试的代码，并构建具有更少或没有错误的软件应用程序。该法则的制定是：

+   每个单元只应对当前单元“密切”相关的单元有限了解。

+   每个单元只能与其朋友交谈；不要与陌生人交谈。

LoD 强调低耦合，这实际上意味着一个对象对另一个对象的了解应该很少或非常有限。将 LoD 与典型的类对象联系起来，类中的方法只应对密切相关对象的其他方法有限了解。

LoD 作为软件开发人员的启发式，以促进软件模块和组件中的信息隐藏。LoD 有两种形式——**对象或动态形式**和**类或静态形式**。

LoD 的类形式被公式化为：

**类**（**C**）的**方法**（**M**）只能向以下类的对象发送消息：

+   M 的参数类，包括 C

+   C 的实例变量

+   在 M 中创建的实例的类

+   C 的属性或字段

LoD 的对象形式被公式化为：

在 M 中，消息只能发送到以下对象：

+   M 的参数，包括封闭对象。

+   M 调用封闭对象返回的即时部分对象，包括封闭对象的属性，或者封闭对象的属性集合的元素：

```cs
public class LoanProcessor
{
    private CurrencyConverter currencyConverter;

    public LoanProcessor(LoanCalculator loanCalculator)
    {
       currencyConverter = loanCalculator.GetCurrencyConverter();
    }
}
```

前面的代码明显违反了 LoD，这是因为`LoanProcessor`实际上并不关心`LoanCalculator`，因为它没有保留任何对它的引用。在代码中，`LoanProcessor`已经在与`LoanCalculator`进行交流，一个陌生人。这段代码实际上并不可重用，因为任何试图重用它们的类或代码都将需要`CurrencyConverter`和`LoanProcessor`，尽管从技术上讲，`LoanCalculator`在构造函数之外并未被使用。

为了对`LoanProcessor`编写单元测试，需要创建对象图。必须创建`LoanCalculator`以便`CurrencyConverter`可用。这会在系统中创建耦合，如果`LoanCalculator`被重构，这是可能的，那么可能会导致`LoanProcessor`出现故障，导致单元测试停止运行。

`LoanCalculator`类可以被模拟，以便单独测试`LoanProcessor`，但这有时会使测试变得难以阅读，最好避免耦合，这样可以编写灵活且易于测试的代码。

要重构前面的代码片段，并使其符合 LoD 并从类构造函数中获取其依赖项，从而消除对`LoanCalculator`的额外依赖，并减少代码的耦合：

```cs
public class LoanProcessor
{
    private CurrencyConverter currencyConverter;

    public LoanProcessor(CurrencyConverter currencyConverter)
    {
       this.currencyConverter = currencyConverter;
    }     
}
```

# 火车失事

另一个违反 LoD 的反模式是所谓的**火车失事**或**链式调用**。这是一系列函数的链，并且当你在一行代码中追加了一系列 C#方法时就会发生。当你花时间试图弄清楚一行代码的作用时，你就会知道你写了一个火车失事的代码：

```cs
loanCalculator.
    CalculateHouseLoan(loanDTO).
        GetPaymentRate().
            GetMaximumYearsToPay();
```

你可能想知道这种现象如何违反了 LoD。首先，代码缺乏可读性，不易维护。此外，代码行不可重用，因为一行代码中有三个方法调用。

这行代码可以通过减少交互和消除方法链来进行重构，以使其符合“不要和陌生人说话”的原则。这个原则解释了调用点或方法应该一次只与一个对象交互。通过消除方法链，生成的代码可以在其他地方重复使用，而不必费力理解代码的作用：

```cs
var houseLoan=loanCalculator.CalculateHouseLoan(loanDTO);
var paymentRate=houseLoan.GetPaymentRate();
var maximumYears=paymentRate.GetMaximumYearsToPay();
```

一个对象应该对其他对象的知识和信息有限。此外，对象中的方法应该对应用程序的对象图具有很少的认识。通过有意识的努力，使用 LoD，你可以构建松散耦合且易于维护的软件应用程序。

# SOLID 架构原则

软件应用程序开发的程序和方法，从第一步到最后一步，应该简单易懂，无论是新手还是专家都能理解。这些程序，当与正确的原则结合使用时，使开发和维护软件应用程序的过程变得简单和无缝。

开发人员不时采用和使用不同的开发原则和模式，以简化复杂性并使软件应用程序代码库易于维护。其中一个原则就是 SOLID 原则。这个原则已经被证明非常有用，是每个面向对象系统的严肃程序员必须了解的。

SOLID 是开发面向对象系统的五个基本原则的首字母缩写。这五个原则是用于类设计的，表示为：

+   **S**：单一职责原则

+   **O**：开闭原则

+   **L**：里氏替换原则

+   **I**：接口隔离原则

+   **D**：依赖反转原则

这些原则首次被整合成 SOLID 的首字母缩写，并在 2000 年代初由*罗伯特·C·马丁*（通常被称为**鲍勃叔叔**）推广。这五个原则是用于类设计的，遵守这些原则可以帮助管理依赖关系，避免创建混乱的、到处都是依赖的僵化代码库。

对 SOLID 原则的正确理解和运用可以使软件开发人员实现非常高的内聚度，并编写易于理解和维护的高质量代码。有了 SOLID 原则，你可以编写干净的代码，构建健壮且可扩展的软件应用程序。

事实上，鲍勃叔叔澄清了 SOLID 原则不是法律或规则，而是已经观察到在几种情况下起作用的启发式。要有效地使用这些原则，你必须搜索你的代码，检查违反原则的部分，然后进行重构。

# 单一职责原则

**单一职责原则**（**SRP**）是五个 SOLID 原则中的第一个。该原则规定一个类在任何时候只能有一个改变的理由。这简单地意味着一个类一次只能执行一个职责或有一个责任。

软件项目的业务需求通常不是固定的。在软件项目发布之前，甚至在软件的整个生命周期中，需求会不时地发生变化，开发人员必须根据变化调整代码库。为了使软件应用程序满足其业务需求并适应变化，必须使用灵活的设计模式，并且类始终只有一个责任。

此外，重要的是要理解，当一个类有多个责任时，即使进行最微小的更改也会对整个代码库产生巨大影响。对类的更改可能会导致连锁反应，导致之前工作的功能或其他方法出现故障。例如，如果你有一个解析`.csv`文件的类，同时它还调用一个 Web 服务来检索与`.csv`文件解析无关的信息，那么这个类就有多个改变的原因。对 Web 服务调用的更改将影响该类，尽管这些更改与`.csv`文件解析无关。

以下代码片段中的`LoanCalculator`类的设计明显违反了 SRP。`LoanCalculator`有两个责任——第一个是计算房屋和汽车贷款，第二个是从 XML 文件和 XML 字符串中解析贷款利率：

```cs
public class LoanCalculator
{
    public CarLoan CalculateCarLoan(LoanDTO loanDTO)
    {
        throw new NotImplementedException();
    }

    public HouseLoan CalculateHouseLoan(LoanDTO loanDTO)
    {
        throw new NotImplementedException();
    }

    public List<Rate> ParseRatesFromXmlString(string xmlString)
    {
        throw new NotImplementedException();
    }

    public List<Rate> ParseRatesFromXmlFile(string xmlFile)
    {
        throw new NotImplementedException();
    }
}
```

`LoanCalculator`类的双重责任状态会产生一些问题。首先，该类变得非常不稳定，因为对一个责任的更改可能会影响另一个责任。例如，对要解析的 XML 内容结构的更改可能需要重写、测试和重新部署该类；尽管如此，对第二个关注点——贷款计算——并没有进行更改。

`LoanCalculator`类中的混乱代码可以通过重新设计类并分离责任来修复。新设计将是将 XML 利率解析的责任移交给一个新的`RateParser`类，并将贷款计算的关注点留在现有类中：

```cs
public class RateParser : IRateParser
{    
    public List<Rate> ParseRatesFromXml(string xmlString)
    {
        throw new NotImplementedException();
    }
    public List<Rate> ParseRatesFromXmlFile(string xmlFile)
    {
        throw new NotImplementedException();
    }
}
```

通过从`LoanCalculator`中提取`RateParser`类，`RateParser`现在可以作为`LoanCalculator`中的一个依赖使用。对`RateParser`中的任何方法的更改不会影响`LoanCalculator`，因为它们现在处理不同的关注点，每个类只有一个改变的原因：

```cs
public class LoanCalculator
{
    private IRateParser rateParser;

    public LoanCalculator(IRateParser rateParser)
    {
        this.rateParser=rateParser;
    }

    public CarLoan CalculateCarLoan(LoanDTO loanDTO)
    {
        throw new NotImplementedException();
    }

    public HouseLoan CalculateCarLoan(LoanDTO loanDTO)
    {
        throw new NotImplementedException();
    }  
}
```

将关注点分开在代码库中创造了很大的灵活性，并允许轻松测试这两个类。通过新的设计，对`RateParser`的更改不会影响`LoanCalculator`，这两个类可以独立进行单元测试。

责任不应该混在一个类中。你应该避免在一个类中混淆责任，这会导致做太多事情的怪兽类。相反，如果你能想到一个改变类的理由或动机，那么它已经有了多个责任；将类分成每个只包含单一责任的类。

类似地，对以下代码片段中的`LoanRepository`类的第一印象可能不会直接表明关注点混淆。但是，如果仔细检查该类，数据访问和业务逻辑代码都混在一起，这使得它违反了 SRP：

```cs
public class LoanRepository
{
    private DbContext dbContext;
    private IEligibilityChecker eligibilityChecker;

    public LoanRepository(DbContext dbContext,IEligibilityChecker eligibilityChecker)
    {
        this.dbContext=dbContext;
        this.eligibilityChecker= eligibilityChecker;
    }

    public List<CarLoan> GetCarLoans()
    {
        return dbContext.CarLoan;
    }

    public List<HouseLoan> GetHouseLoans()
    {
        return dbContext.HouseLoan;
    }

    public double CalculateCarLoanRate(CarLoan carLoan)
    {
        double rate=12.5F;
        bool isEligible=eligibilityChecker.IsApplicantEligible(carLoan);
        if(isEligible)
          rate=rate-carLoan.DiscountFactor; 
        return rate;
    }
}
```

这个类可以通过将计算汽车贷款利率的业务逻辑代码分离到一个新的类——`LoanService`中来重构，这将允许`LoanRepository`类只包含与数据层相关的代码，从而使其符合 SRP：

```cs
public class LoanService
{
    private IEligibilityChecker eligibilityChecker;

    public LoanService(IEligibilityChecker eligibilityChecker)
    {
        this.eligibilityChecker= eligibilityChecker;
    }    

    public double CalculateCarLoanRate(CarLoan carLoan)
    {
        double rate=12.5F;
        bool isEligible=eligibilityChecker.IsApplicantEligible(carLoan);
        if(isEligible)
          rate=rate-carLoan.DiscountFactor; 
        return rate;
    }
}
```

通过将业务逻辑代码分离到`LoanService`类中，`LoanRepository`类现在只有一个依赖，即`DbContext`实体框架。未来，`LoanRepository`可以很容易地进行维护和测试。新的`LoanService`类也符合 SRP：

```cs
public class LoanRepository
{
    private DbContext dbContext;

    public LoanRepository(DbContext dbContext)
    {
        this.dbContext=dbContext;
    }

    public List<CarLoan> GetCarLoans()
    {
        return dbContext.CarLoan;
    }

    public List<HouseLoan> GetHouseLoans()
    {
        return dbContext.HouseLoan;
    }    
}
```

当您的代码中的问题得到很好的管理时，代码库将具有高内聚性，并且将来会更加灵活、易于测试和维护。有了高内聚性，类将松散耦合，对类的更改将很少可能破坏整个系统。

# 开闭原则

设计和最终编写生产代码的方法应该是允许向项目的代码库添加新功能，而无需进行多次更改、更改代码库的几个部分或类，或破坏已经正常工作且状态良好的现有功能。

如果由于对类中的方法进行更改而导致必须对多个部分或模块进行更改，这表明代码设计存在问题。这就是**开闭原则**（OCP）所解决的问题，允许您的代码库设计灵活，以便您可以轻松进行修改和增强。

OCP 规定软件实体（如类、方法和模块）应设计为对扩展开放，但对修改关闭。这个原则可以通过继承或设计模式（如工厂、观察者和策略模式）来实现。这是指类和方法可以被设计为允许添加新功能，以供现有代码使用，而无需实际修改或更改现有代码，而是通过扩展现有代码的行为。

在 C#中，通过正确使用对象抽象，您可以拥有封闭的类，这些类对修改关闭，而类的行为可以通过派生类进行扩展。派生类是封闭类的子类。使用继承，您可以创建通过扩展其基类添加更多功能的类，而无需修改基类。

考虑以下代码片段中的`LoanCalculator`类，它具有一个`CalculateLoan`方法，必须能够计算传递给它的任何类型的贷款的详细信息。在不使用 OCP 的情况下，可以使用`if..else if`语句来计算要求。

`LoanCalculator`类具有严格的结构，当需要支持新类型时需要进行大量工作。例如，如果您打算添加更多类型的客户贷款，您必须修改`CalculateLoan`方法并添加额外的`else if`语句以适应新类型的贷款。`LoanCalculator`违反了 OCP，因为该类不是封闭的以进行修改：

```cs
public class LoanCalculator
{
    private IRateParser rateParser;

    public LoanCalculator(IRateParser rateParser)
    {
        this.rateParser=rateParser;
    }

    public Loan CalculateLoan(LoanDTO loanDTO)
    {
        Loan loan = new Loan();
        if(loanDTO.LoanType==LoanType.CarLoan)
        {
            loan.LoanType=LoanType.CarLoan;
            loan.InterestRate=rateParser.GetRateByLoanType(LoanType.CarLoan);
            // do other processing
        }
        else if(loanDTO.LoanType==LoanType.HouseLoan)
        {
            loan.LoanType=LoanType.HouseLoan;
            loan.InterestRate=rateParser.GetRateByLoanType(LoanType.HouseLoan);
            // do other processing
        }        
        return loan;
    }   
}
```

为了使`LoanCalculator`类对扩展开放而对修改关闭，我们可以使用继承来简化重构。 `LoanCalculator`将被重构以允许从中创建子类。将`LoanCalculator`作为基类将有助于创建两个派生类，`HouseLoanCalculator`和`CarLoanCalulator`。计算不同类型贷款的业务逻辑已从`CalculateLoan`方法中移除，并在两个派生类中实现，如下面的代码片段所示：

```cs
public class LoanCalculator
{
    protected IRateParser rateParser;

    public LoanCalculator(IRateParser rateParser)
    {
        this.rateParser=rateParser;
    }

    public Loan CalculateLoan(LoanDTO loanDTO)
    {
        Loan loan = new Loan(); 
        // do some base processing
        return loan;
    }   
}
```

`LoanCalculator`类中的`If`条件已从`CalculateLoan`方法中移除。现在，新的`CarLoanCaculator`类包含了获取汽车贷款计算的逻辑：

```cs
public class CarLoanCalculator : LoanCalculator
{    
    public CarLoanCalculator(IRateParser rateParser) :base(rateParser)
    {
        base.rateParser=rateParser;
    }

    public override Loan CalculateLoan(LoanDTO loanDTO)
    {
        Loan loan = new Loan();
        loan.LoanType=loanDTO.LoanType;
        loan.InterestRate=rateParser.GetRateByLoanType(loanDTO.LoanType);
        // do other processing
        return loan
    }   
}
```

`HouseLoanCalculator`类是从`LoanCalculator`创建的，具有覆盖基类`LoanCalculator`中的`CalculateLoan`方法的`CalculateLoan`方法。对`HouseLoanCalculator`进行的任何更改都不会影响其基类的`CalculateLoan`方法：

```cs
public class HouseLoanCalculator : LoanCalculator
{    
    public HouseLoanCalculator(IRateParser rateParser) :base(rateParser)
    {
        base.rateParser=rateParser;
    }

    public override Loan CalculateLoan(LoanDTO loanDTO)
    {
        Loan loan = new Loan();
        loan.LoanType=LoanType.HouseLoan;
        loan.InterestRate=rateParser.GetRateByLoanType(LoanType.HouseLoan);
        // do other processing
        return loan;
    }    
}
```

如果引入了新类型的贷款，比如研究生学习贷款，可以创建一个新类`PostGraduateStudyLoan`来扩展`LoanCalculator`并实现`CalculateLoan`方法，而无需对`LoanCalculator`类进行任何修改。

从技术上讲，观察 OCP 意味着您的代码中的类和方法应该对扩展开放，这意味着可以扩展类和方法以添加新的行为来支持新的或不断变化的应用程序需求。而且类和方法对于修改是封闭的，这意味着您不能对源代码进行更改。

为了使`LoanCalculator`对更改开放，我们将其作为其他类型的基类派生。或者，我们可以创建一个`ILoanCalculator`抽象，而不是使用经典的对象继承：

```cs
public interface ILoanCalculator
{
    Loan CalculateLoan(LoanDTO loanDTO);
}
```

`CarLoanCalculator`类现在可以被创建来实现`ILoanCalculator`接口。这将需要`CarLoanCalculator`类明确实现接口中定义的方法和属性。

```cs
public class CarLoanCalculator : ILoanCalculator
{    
    private IRateParser rateParser;

    public CarLoanCalculator(IRateParser rateParser)
    {
        this.rateParser=rateParser;
    }

    public Loan CalculateLoan(LoanDTO loanDTO)
    {
        Loan loan = new Loan();
        loan.LoanType=loanDTO.LoanType;
        loan.InterestRate=rateParser.GetRateByLoanType(loanDTO.LoanType);
        // do other processing
        return loan
    }   
}
```

`HouseLoanCalculator`类也可以被创建来实现`ILoanCalculator`，通过构造函数将`IRateParser`对象注入其中，类似于`CarLoanCalculator`。`CalculateLoan`方法可以被实现为具有计算房屋贷款所需的特定代码。通过简单地创建类并使其实现`ILoanCalculator`接口，可以添加任何其他类型的贷款：

```cs
public class HouseLoanCalculator  : ILoanCalculator
{    
    private IRateParser rateParser;

    public HouseLoanCalculator (IRateParser rateParser)
    {
        this.rateParser=rateParser;
    }

    public Loan CalculateLoan(LoanDTO loanDTO)
    {
        Loan loan = new Loan();
        loan.LoanType=loanDTO.LoanType;
        loan.InterestRate=rateParser.GetRateByLoanType(loanDTO.LoanType);
        // do other processing
        return loan
    }   
}
```

使用 OCP，您可以创建灵活的软件应用程序，其行为可以轻松扩展，从而避免代码基础僵化且缺乏可重用性。通过适当使用 OCP，通过有效使用代码抽象和对象多态性，可以对代码基础进行更改，而无需更改许多部分，并且付出很少的努力。您真的不必重新编译代码基础来实现这一点。

# Liskov 替换原则

**Liskov 替换原则**（LSP），有时也称为**按合同设计**，是 SOLID 原则中的第三个原则，最初由*Barbara Liskov*提出。LSP 规定，派生类或子类应该可以替换基类或超类，而无需对基类进行修改或在系统中生成任何运行时错误。

LSP 可以通过以下数学符号进一步解释——假设 S 是 T 的子集，T 的对象可以替换 S 的对象，而不会破坏系统的现有工作功能或引起任何类型的错误。

为了说明 LSP 的概念，让我们考虑一个带有`Drive`方法的`Car`超类。如果`Car`有两个派生类，`SalonCar`和`JeepCar`，它们都有`Drive`方法的重写实现，那么无论何时需要`Car`，都应该可以使用`SalonCar`和`JeepCar`来替代`Car`类。派生类与`Car`有一个*是一个*的关系，因为`SalonCar`是`Car`，`JeepCar`是`Car`。

为了设计您的类并实现它们以符合 LSP，您应该确保派生类的元素是按照合同设计的。派生类的方法定义应该与基类的相似，尽管实现可能会有所不同，因为不同的业务需求。

此外，重要的是派生类的实现不违反基类或接口中实现的任何约束。当您部分实现接口或基类时，通过具有未实现的方法，您正在违反 LSP。

以下代码片段具有`LoanCalculator`基类，具有`CalculateLoan`方法和两个派生类，`HouseLoanCalculator`和`CarLoanCalculator`，它们具有`CalculateLoan`方法并且可以具有不同的实现：

```cs
public class LoanCalculator
{
    public Loan CalculateLoan(LoanDTO loanDTO)
    {
        throw new NotImplementedException();
    }   
}

public class HouseLoanCalculator  : LoanCalculator
{     
    public override Loan CalculateLoan(LoanDTO loanDTO)
    {
        throw new NotImplementedException();   
    }   
}

public class CarLoanCalculator  : LoanCalculator
{     
    public override Loan CalculateLoan(LoanDTO loanDTO)
    {
        throw new NotImplementedException();   
    }   
}
```

如果在前面的代码片段中没有违反 LSP，那么`HouseLoanCalculator`和`CarLoanCalculator`派生类可以在需要`LoanCalculator`引用的任何地方使用。这在以下代码片段中的`Main`方法中得到了证明：

```cs
public static void Main(string [] args)
{
    //substituting CarLoanCalulator for LoanCalculator
    RateParser rateParser = new RateParser();
    LoanCalculator loanCalculator= new CarLoanCalculator(rateParser);
    Loan carLoan= loanCalulator.CalculateLoan();

    //substituting HouseLoanCalculator for LoanCalculator
    loanCalculator= new HouseLoanCalculator(rateParser);
    Loan houseLoan= loanCalulator.CalculateLoan();

    Console.WriteLine($"Car Loan Interest Rate - {carLoan.InterestRate}");
    Console.WriteLine($"House Loan Interest Rate - {houseLoan.InterestRate}");
}

```

# 接口隔离原则

接口是一种面向对象的编程构造，被对象用来定义它们公开的方法和属性，并促进与其他对象的交互。接口包含相关方法，具有空的方法体但没有实现。接口是面向对象编程和设计中的有用构造；它允许创建灵活且松耦合的软件应用程序。

接口隔离原则（ISP）规定接口应该是适度的，只包含所需的属性和方法的定义，客户端不应被强制实现他们不使用的接口，或依赖他们不需要的方法。

要有效地在代码库中实现 ISP，您应该倾向于创建简单而薄的接口，这些接口具有逻辑上分组在一起以解决特定业务案例的方法。通过创建薄接口，类代码中包含的方法可以轻松实现，同时保持代码库的清晰和优雅。

另一方面，如果您的接口臃肿或臃肿，其中包含类不需要的功能的方法，您更有可能违反 ISP 并在代码中创建耦合，这将导致代码库无法轻松测试。

与其拥有臃肿或臃肿的接口，不如创建两个或更多个薄接口，将方法逻辑地分组，并让您的类实现多个接口，或让接口继承其他薄接口，这种现象被称为多重继承，在 C#中得到支持。

以下片段中的`IRateCalculator`接口违反了 ISP。它可以被视为一个污染的接口，因为唯一实现它的类不需要`FindLender`方法，因为`RateCalculator`类不需要它：

```cs
public interface IRateCalculator
{
    Rate GetYearlyCarLoanRate();
    Rate GetYearlyHouseLoanRate();
    Lender FindLender(LoanType loanType);
}
```

`RateCalculator`类具有`GetYearlyCarLoanRate`和`GetYearlyHouseLoanRate`方法，这些方法是必需的以满足类的要求。通过实现`IRateCalculator`，`RateCalculator`被迫为`FindLender`方法实现，而这并不需要：

```cs
public class RateCalculator :IRateCalculator
{
    public Rate GetYearlyCarLoanRate()
    {
        throw new NotImplementedException();
    }

    public Rate GetYearlyHouseLoanRate()
    {
        throw new NotImplementedException();
    }

    public Lender FindLender(LoanType loanType)
    {
        throw new NotImplementedException();
    }
}
```

前述的`IRateCalculator`可以重构为两个具有可以逻辑分组在一起的方法的连贯接口。通过小接口，可以以极大的灵活性编写代码，并且易于对实现接口的类进行单元测试：

```cs
public interface IRateCalculator
{
    Rate GetYearlyCarLoanRate();
    Rate GetYearlyHouseLonaRate();
}

public interface ILenderManager
{
    Lender FindLender(LoanType loanType);
}
```

通过将`IRateCalculator`重构为两个接口，`RateCalculator`可以被重构以删除不需要的`FindLender`方法：

```cs
public class RateCalculator :IRateCalculator
{
    public Rate GetYearlyCarLoanRate()
    {
        throw new NotImplementedException();
    }

    public Rate GetYearlyHouseLonaRate()
    {
        throw new NotImplementedException();
    }    
}
```

在实现符合 ISP 的接口时要注意的反模式是为每个方法创建一个接口，试图创建薄接口；这可能导致创建多个接口，从而导致难以维护的代码库。

# 依赖反转原则

刚性或糟糕的设计可能会使软件应用程序的组件或模块的更改变得非常困难，并创建维护问题。这些不灵活的设计通常会破坏先前正常工作的功能。这可能以原则和模式的错误使用、糟糕的代码和不同组件或层的耦合形式出现，从而使维护过程变得非常困难。

当应用程序代码库中存在严格的设计时，仔细检查代码将会发现模块之间紧密耦合，使得更改变得困难。对任何模块的更改可能会导致破坏先前正常工作的另一个模块的风险。观察 SOLID 原则中的最后一个——依赖反转原则（DIP）可以消除模块之间的任何耦合，使代码库灵活且易于维护。

DIP 有两种形式，都旨在实现代码的灵活性和对象及其依赖项之间的松耦合：

+   高级模块不应依赖于低级模块；两者都应依赖于抽象

+   抽象不应依赖于细节；细节应依赖于抽象

当高级模块或实体直接耦合到低级模块时，对低级模块进行更改通常会直接影响高级模块，导致它们发生变化，产生连锁反应。在实际情况下，当对高级模块进行更改时，低级模块应该发生变化。

此外，您可以在需要类与其他类通信或发送消息的任何地方应用 DIP。DIP 倡导应用程序开发中众所周知的分层原则或关注点分离原则：

```cs
public class AuthenticationManager
{
    private DbContext dbContext;

    public AuthenticationManager(DbContext dbContext)
    {
        this.dbContext=dbContext;
    }
}
```

在上面的代码片段中，`AuthenticationManager`类代表了一个高级模块，而传递给类构造函数的`DbContext` Entity Framework 是一个负责 CRUD 和数据层活动的低级模块。虽然非专业的开发人员可能不会在代码结构中看到任何问题，但它违反了 DIP。这是因为`AuthenticationManager`类依赖于`DbContext`类，并且对`DbContext`内部代码进行更改的尝试将会传播到`AuthenticationManager`，导致它发生变化，从而违反 OCP。

我们可以重构`AuthenticationManager`类，使其具有良好的设计并符合 DIP。这将需要创建一个`IDbContext`接口，并使`DbContext`实现该接口。

```cs
public interface IDbContext
{
    int SaveChanges();
    void Dispose();
}

public class DbContext : IDbContext
{
    public int SaveChanges()
    {
        throw new NotImplementedException();
    }

    public void Dispose()
    {
        throw new NotImplementedException();
    }
}
```

`AuthenticationManager`可以根据接口编码，从而打破与`DbContext`的耦合或直接依赖，并且依赖于抽象。对`AuthenticationManager`进行编码，使其针对`IDbContext`意味着接口将被注入到`AuthenticationManager`的构造函数中，或者使用*属性注入*：

```cs
public class AuthenticationManager
{
    private IDbContext dbContext;

    public AuthenticationManager(IDbContext dbContext)
    {
        this.dbContext=dbContext;
    }
}
```

重构完成后，`AuthenticationManager`现在使用依赖反转，并依赖于抽象—`IDbContext`。将来，如果对`DbContext`类进行更改，将不再影响`AuthenticationManager`类，并且不会违反 OCP。

虽然通过构造函数将`IDbContext`注入到`AutheticationManager`中非常优雅，但`IDbcontext`也可以通过公共属性注入到`AuthenticationManager`中：

```cs
public class AuthenticationManager
{
    private IDbContext dbContext;

    private IDbContext DbContext
    {
        set
        {
            dbContext=value;
        }
    }
}
```

此外，DI 也可以通过*接口注入*来实现，其中对象引用是使用接口操作传递的。这简单地意味着使用接口来注入依赖项。以下代码片段解释了使用接口注入来实现依赖的概念。

`IRateParser`是使用`ParseRate`方法定义创建的。第二个接口`IRepository`包含`InjectRateParser`方法，该方法接受`IRateParser`作为参数，并将注入依赖项。

```cs
public interface IRateParser
{
    Rate ParseRate();
}

public interface IRepository
{
    void InjectRateParser(IRateParser rateParser);
}
```

现在，让我们创建`LoanRepository`类来实现`IRepository`接口，并为`InjectRateParser`创建一个代码实现，以将`IRateParser`存储库作为依赖项注入到`LoanRepository`类中以供代码使用：

```cs
public class LoanRepository : IRepository
{
    IRateParser rateParser;

    public void InjectRateParser(IRateParser rateParser)
    {
        this.rateParser = rateParser;
    }

     public float GetCheapestRate(LoanType loanType)
     {
         return rateParser.GetRateByLoanType(loanType);
     }
}
```

接下来，我们可以创建`IRateParser`依赖的具体实现，`XmlRateParser`和`RestServiceRateParser`，它们都包含了从 XML 和 REST 源解析贷款利率的`ParseRate`方法的实现：

```cs
public class XmlRateParser : IRateParser
{
    public Rate ParseRate()
    {
        // Parse rate available from xml file
        throw new NotImplementedException();
    }
}

public class RestServiceRateParser : IRateParser
{
    public Rate ParseRate()
    {
        // Parse rate available from REST service
        throw new NotImplementedException();
    }
}
```

总之，我们可以使用在前面的代码片段中创建的接口和类来测试*接口注入*概念。创建了`IRateParser`的具体对象，它被注入到`LoanRepository`类中，通过`IRepository`接口，并且可以使用`IRateParser`接口的两种实现之一来构造它。

```cs
IRateParser rateParser = new XmlRateParser();           
LoanRepository loanRepository = new LoanRepository();
((IRepository)loanRepository).InjectRateParser(rateParser);
var rate= loanRepository.GetCheapestRate();

rateParser = new RestServiceRateParser();       
((IRepository)loanRepository).InjectRateParser(rateParser);
rate= loanRepository.GetCheapestRate();

```

在本节中描述的任何三种技术都可以有效地用于在需要时将依赖项注入到代码中。适当有效地使用 DIP 可以促进创建易于维护的松散耦合的应用程序。

# 为 ASP.NET Core MVC 设置 DI 容器

ASP.NET Core 的核心是 DI。该框架提供了内置的 DI 服务，允许开发人员创建松散耦合的应用程序，并防止依赖关系的实例化或构造。使用内置的 DI 服务，您的应用程序代码可以设置为使用 DI，并且依赖项可以被注入到`Startup`类中的方法中。虽然默认的 DI 容器具有一些很酷的功能，但您仍然可以在 ASP.NET Core 应用程序中使用其他已知的成熟的 DI 容器。

您可以将代码配置为以两种模式使用 DI：

+   **构造函数注入**：类所需的接口通过类的公共构造函数传递或注入。使用私有构造函数无法进行构造函数注入，当尝试这样做时，将引发`InvalidOperationException`。在具有重载构造函数的类中，只能使用一个构造函数进行 DI。

+   **属性注入**：通过在类中使用公共接口属性将依赖项注入到类中。可以使用这两种模式之一来请求依赖项，这些依赖项将由 DI 容器注入。

DI 容器，也称为**控制反转**（**IoC**）容器，通常是一个可以创建具有其关联依赖项的类的类或工厂。在成功构造具有注入依赖项的类之前，项目必须设计或设置为使用 DI，并且 DI 容器必须已配置为具有依赖类型。实质上，DI 将具有包含接口到其具体类的映射的配置，并将使用此配置来解析所需依赖项的类。

ASP.NET Core 内置的 IoC 容器由`IServiceProvider`接口表示，您可以使用`Startup`类中的`ConfigureService`方法对其进行配置。容器默认支持构造函数注入。在`ConfigureService`方法中，可以定义服务和平台功能，例如 Entity Framework Core 和 ASP.NET MVC Core：

```cs
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddDbContext<ApplicationDbContext>(options => options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
    services.AddIdentity<ApplicationUser, IdentityRole>().AddEntityFrameworkStores<ApplicationDbContext>().AddDefaultTokenProviders();

    services.AddMvc();

    // Configured DI
    services.AddTransient<ILenderManager, LenderManager >();
    services.AddTransient<IRateCalculator, RateCalculator>();
}
```

ASP.NET Core 内置容器具有一些扩展方法，例如`AddDbContext`、`AddIdentity`和`AddMvc`，您可以使用这些方法添加其他服务。可以使用`AddTransient`方法配置应用程序依赖项，该方法接受两个泛型类型参数，第一个是接口，第二个是具体类。`AddTransient`方法将接口映射到具体类，因此每次请求时都会创建服务。容器使用此配置为在 ASP.NET MVC 项目中需要它的每个对象注入接口。

用于配置服务的其他扩展方法是`AddScoped`和`AddSingleton`方法。`AddScoped`每次请求只创建一次服务：

```cs
services.AddScoped<ILenderManager, LenderManager >();
```

`AddSingleton`方法只在首次请求时创建服务，并将其保存在内存中，使其可供后续请求使用。您可以自行实例化单例，也可以让容器来处理：

```cs
// instantiating singleton 
services.AddSingleton<ILenderManager>(new LenderManager()); 

// alternative way of configuring singleton service
services.AddSingleton<IRateCalculator, RateCalculator>();
```

ASP.NET Core 的内置 IoC 容器轻量级且功能有限，但基本上您可以在应用程序中使用它进行 DI 配置。但是，您可以将其替换为.NET 中可用的其他 IoC 容器，例如**Ninject**或**Autofac**。

使用 DI 将简化应用程序开发体验，并使您能够编写松散耦合且易于测试的代码。在典型的 ASP.NET Core MVC 应用程序中，您应该使用 DI 来处理依赖项，例如**存储库**、**控制器**、**适配器**和**服务**，并避免对服务或`HttpContext`进行静态访问。

# 摘要

本章中使用的面向对象设计原则将帮助您掌握编写清晰、灵活、易于维护和易于测试代码所需的技能。本章中解释的 LoD 和 SOLID 原则可以作为创建松散耦合的面向对象软件应用程序的指导原则。

为了获得 TDD 周期的好处，您必须编写可测试的代码。所涵盖的 SOLID 原则描述了适当的实践，可以促进编写可轻松维护并在需要时进行增强的可测试代码。本章的最后一节着重介绍了为 ASP.NET Core MVC 应用程序设置和使用依赖注入容器。

在下一章中，我们将讨论良好单元测试的属性，.NET 生态系统中可用于创建测试的单元测试框架，以及在单元测试 ASP.NET MVC Core 项目时需要考虑的内容，我们还将深入探讨在.NET Core 平台上使用 xUnit 库进行单元测试的属性。
