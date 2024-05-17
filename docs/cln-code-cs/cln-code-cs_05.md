异常处理

在上一章中，我们看了函数。尽管程序员尽力编写健壮的代码，但函数最终会产生异常。这可能是由于许多原因，例如缺少文件或文件夹，空值或空值，无法写入位置，或者用户被拒绝访问。因此，在本章中，您将学习使用异常处理产生清晰的 C#代码的适当方法。首先，我们将从算术`OverflowExceptions`的检查和未经检查的异常开始。我们将看看它们是什么，为什么使用它们，以及它们在代码中的一些示例。

然后，我们将看看如何避免`NullPointerReference`异常。之后，我们将研究为特定类型的异常实现特定业务规则。在对异常和异常业务规则有了新的理解之后，我们将开始构建自己的自定义异常，然后最后看看为什么我们不应该使用异常来控制计算机程序的流程。

在本章中，我们将涵盖以下主题：

+   检查和未经检查的异常

+   避免`NullPointerExceptions`

+   业务规则异常

+   异常应提供有意义的信息

+   构建自定义异常

在本章结束时，您将具备以下技能：

+   您将能够理解 C#中的检查和未经检查的异常，以及它们的原因。

+   您将能够理解什么是`OverflowException`以及如何在编译时捕获它们。

+   您将了解什么是`NullPointerExceptions`以及如何避免它们。

+   您将能够编写自己的自定义异常，为客户提供有意义的信息，并帮助您和其他程序员轻松识别和解决引发的任何问题。

+   您将能够理解为什么不应该使用异常来控制程序流程。

+   您将知道如何使用 C#语句和布尔检查来替换业务规则异常，以控制程序流程。

# 第六章：检查和未经检查的异常

在未经检查的模式下，算术溢出会被*忽略*。在这种情况下，无法分配给目标类型的高阶位将从结果中丢弃。

默认情况下，C#在运行时执行非常量表达式时处于未经检查的上下文中。但是编译时常量表达式*始终*默认进行检查。在检查模式下遇到算术溢出时，会引发`OverflowException`。未经检查异常被使用的一个原因是为了提高性能。检查异常可能会稍微降低方法的性能。

经验法则是确保在检查上下文中执行算术运算。任何算术溢出异常都将被视为编译时错误，然后您可以在发布代码之前修复它们。这比发布代码然后不得不修复客户运行时错误要好得多。

在未经检查的模式下运行代码是危险的，因为您对代码进行了假设。假设并非事实，它们可能导致在运行时引发异常。运行时异常会导致客户满意度降低，并可能产生严重的后续异常，以某种方式对客户产生负面影响。

允许应用程序继续运行，即使发生了溢出异常，从商业角度来看是非常危险的。原因在于数据可能会处于不可逆转的无效状态。如果数据是关键的客户数据，那么这对企业来说可能会非常昂贵，你不希望承担这样的责任。

考虑以下代码。这段代码演示了在客户银行业务中未经检查的溢出有多糟糕：

```cs
private static void UncheckedBankAccountException()
{
    var currentBalance = int.MaxValue;
    Console.WriteLine($"Current Balance: {currentBalance}");
    currentBalance = unchecked(currentBalance + 1);
    Console.WriteLine($"Current Balance + 1 = {currentBalance}");
    Console.ReadKey();
}
```

想象一下，当客户看到将 1 英镑加到他们的银行余额 2,147,483,647 英镑时，他们的脸上会有多么恐慌！

![](img/a108b152-4768-43c9-acf0-164699f74f0b.png)

现在，是时候用一些代码示例演示检查和未检查异常了。首先，启动一个新的**控制台应用程序**并声明一些变量：

```cs
static byte y, z;
```

前面的代码声明了两个字节，我们将在算术代码示例中使用。现在，添加`CheckedAdd()`方法。如果在添加两个数字时遇到算术溢出导致的结果太大无法存储为字节，此方法将引发一个检查过的`OverflowException`：

```cs
private static void CheckedAdd()
{
    try
    {
        Console.WriteLine("### Checked Add ###");
        Console.WriteLine($"x = {y} + {z}");
        Console.WriteLine($"x = {checked((byte)(y + z))}");
    }
    catch (OverflowException oex)
    {
        Console.WriteLine($"CheckedAdd: {oex.Message}");
    }
}
```

然后，编写`CheckedMultiplication()`方法。如果在乘法过程中检测到算术溢出，导致的数字大于一个字节，将引发检查过的`OverflowException`：

```cs
private static void CheckedMultiplication()
{
    try
    {
        Console.WriteLine("### Checked Multiplication ###");
        Console.WriteLine($"x = {y} x {z}");
        Console.WriteLine($"x = {checked((byte)(y * z))}");
    }
    catch (OverflowException oex)
    {
        Console.WriteLine($"CheckedMultiplication: {oex.Message}");
    }
}
```

接下来，添加`UncheckedAdd()`方法。此方法将忽略由于加法而发生的任何溢出，因此不会引发`OverflowException`。溢出的结果将存储为一个字节，但值将是不正确的：

```cs
private static void UncheckedAdd()
{
    try
    {
         Console.WriteLine("### Unchecked Add ###");
         Console.WriteLine($"x = {y} + {z}");
         Console.WriteLine($"x = {unchecked((byte)(y + z))}");
    }
    catch (OverflowException oex)
    {
         Console.WriteLine($"CheckedAdd: {oex.Message}");
    }
}
```

现在，我们添加`UncheckedMultiplication()`方法。当遇到溢出时，此方法不会抛出`OverflowException`。异常将被简单地忽略。这将导致一个不正确的数字被存储为字节：

```cs
private static void UncheckedMultiplication()
{
    try
    {
         Console.WriteLine("### Unchecked Multiplication ###");
         Console.WriteLine($"x = {y} x {z}");
         Console.WriteLine($"x = {unchecked((byte)(y * z))}");
    }
    catch (OverflowException oex)
    {
        Console.WriteLine($"CheckedMultiplication: {oex.Message}");
    }
}
```

最后，是时候修改我们的`Main(string[] args)`方法，以便我们可以初始化变量并执行方法。在这里，我们将最大值添加到`y`变量和`2`添加到`z`变量。然后，我们运行`CheckedAdd()`和`CheckedMultiplication()`方法，这两个方法都会生成`OverflowException()`。这是因为`y`变量包含了一个字节的最大值。

因此，通过添加或乘以`2`，您超出了存储变量所需的地址空间。接下来，我们将运行`UncheckedAdd()`和`UncheckedMultiplication()`方法。这两种方法都忽略溢出异常，将结果分配给`x`变量，并忽略任何溢出的位。最后，我们在用户按下任意键时打印一条消息，然后退出：

```cs
static void Main(string[] args)
{
    y = byte.MaxValue;
    z = 2;
    CheckedAdd();
    CheckedMultiplication();
    UncheckedAdd();
    UncheckedMultiplication();
    Console.WriteLine("Press any key to exit.");
    Console.ReadLine();
}
```

当我们运行前面的代码时，我们得到以下输出：

![](img/575def5c-092b-4eda-b195-6fc08d6d56bd.png)

如您所见，当我们使用检查异常时，当遇到`OverflowException`时会引发异常。但当我们使用未检查异常时，不会引发异常。

从前面的截图可以看出，意外值可能导致问题，并且使用未检查异常可能导致某些行为。因此，在执行算术运算时的经验法则必须始终使用检查异常。

现在，让我们继续看一个程序员经常遇到的非常常见的异常，称为`NullPointerException`。

# 避免 NullPointerExceptions

`NullReferenceException`是大多数程序员经历过的常见异常。当尝试访问`null`对象的属性或方法时，会引发此异常。

为了防止计算机程序崩溃，程序员们常用的做法是使用`try{...}catch (NullReferenceExceptionre){...}`块。这是防御性编程的一部分。但问题是，很多时候错误只是*记录*和*重新抛出*。此外，还进行了很多不必要的计算。

处理`ArgumentNullExceptions`的一个更好的方法是实现`ArgumentNullValidator`。方法的参数通常是`null`对象的来源。在使用参数之前测试方法的参数并且如果发现它们因任何原因无效，则抛出适当的`Exception`是有意义的。在`ArgumentNullValidator`的情况下，您将把此验证器放在方法的顶部，然后测试每个参数。如果发现任何参数为`null`，则会抛出`NullReferenceException`。这将节省计算并消除了将方法代码包装在`try...catch`块中的需要。

为了明确事物，我们将编写`ArgumentNullValidator`并在一个方法中使用它来测试方法的参数：

```cs
public class Person
{
    public string Name { get; }
    public Person(string name)
    {
         Name = name;
    }
}
```

在上面的代码中，我们创建了一个名为`Name`的只读属性的`Person`类。这将是我们将用于传递到示例方法中以引发`NullReferenceException`的对象。接下来，我们将为验证器创建我们的`Attribute`，称为`ValidatedNotNullAttribibute`：

```cs
[AttributeUsage(AttributeTargets.All, Inherited = false, AllowMultiple = true)]
internal sealed class ValidatedNotNullAttribute : Attribute { }
```

现在我们有了我们的`Attribute`，是时候编写验证器了：

```cs
internal static class ArgumentNullValidator
{
    public static void NotNull(string name, 
     [ValidatedNotNull] object value)
    {
        if (value == null)
        {
            throw new ArgumentNullException(name);
        }
    }
}
```

`ArgumentNullValidator`接受两个参数：

+   对象的名称

+   对象本身

检查对象是否为`null`。如果是`null`，则抛出`ArgumentNullException`，并传入对象的名称。

以下方法是我们的`try/catch`示例方法。请注意，我们记录了一条消息并抛出了异常。然而，我们没有使用声明的异常参数，因此按理说应该将其删除。您会经常在代码中看到这种情况。这是不必要的，应该删除以整理代码：

```cs
private void TryCatchExample(Person person)
{
    try
    {
        Console.WriteLine($"Person's Name: {person.Name}");
    }
    catch (NullReferenceException nre)
    {
        Console.WriteLine("Error: The person argument cannot be null.");
        throw;
    }
}
```

接下来，我们将编写一个将使用`ArgumentNullValidator`的示例方法。我们将其称为`ArgumentNullValidatorExample`：

```cs
private void ArgumentNullValidatorExample(Person person)
{
    ArgumentNullValidator.NotNull("Person", person);
    Console.WriteLine($"Person's Name: {person.Name}");
    Console.ReadKey();
}
```

请注意，我们已经从包括大括号在内的九行代码减少到了只有两行。我们也不会在验证之前尝试使用该值。现在我们需要做的就是修改我们的`Main`方法来运行这些方法。通过注释掉其中一个方法并运行程序来测试每个方法。这样做时，最好逐步执行代码以查看发生了什么。

以下是运行`TryCatchExample`方法的输出：

![](img/af29322d-9c55-4bf5-919a-2d18d325f98e.png)

以下是运行`ArgumentNullValidatorExample`的输出：

![](img/c1e41646-e39e-46c3-a8ea-43d813d5e302.png)

如果您仔细研究前面的屏幕截图，您会发现在使用`ArgumentNullValidatorExample`时我们只记录了一次错误。当使用`TryCatchExample`抛出异常时，异常被记录了两次。

第一次，我们有一个有意义的消息，但第二次，消息是*神秘的*。然而，由调用方法`Main`记录的异常并不神秘。事实上，它非常有帮助，因为它向我们显示了`Person`参数的值不能为`null`。

希望这一部分向您展示了在使用构造函数和方法之前检查参数的价值。通过这样做，您可以看到参数验证器如何减少您的代码，从而使其更易读。

现在，我们将看看如何为特定异常实现业务规则。

# 业务规则异常

技术异常是由计算机程序由于程序员的错误和/或环境问题（例如磁盘空间不足）而抛出的异常。

但是业务规则异常是不同的。业务规则异常意味着这种行为是预期的，并且用于控制程序流程，而实际上，异常应该是程序的正常流程的例外，而不是方法的预期输出。

例如，想象一个在 ATM 机上从账户中取出 100 英镑的人，账户里没有钱，也没有透支的能力。ATM 接受用户的 100 英镑取款请求，因此发出`Withdraw(100);`命令。`Withdraw`方法检查余额，发现账户资金不足，因此抛出`InsufficientFundsException()`。

您可能认为拥有这样的异常是一个好主意，因为它们是明确的，并有助于识别问题，以便在收到这样的异常时执行非常具体的操作——但不是！这不是一个好主意。

在这种情况下，当用户提交请求时，应检查所请求的金额是否可以取款。如果可以，那么交易应该继续进行，如用户所请求的那样。但是，如果验证检查确定无法继续进行交易，那么程序应该按照正常的程序流程取消交易，并通知发出请求的用户，而不引发异常。

我们刚刚看到的取款情景表明，程序员已经正确考虑了程序的正常流程和不同的结果。程序流程已经适当地使用布尔检查编码，以允许成功的取款交易并防止不允许的取款交易。

让我们看看如何使用**业务规则异常**（**BREs**）来实现不允许透支的银行账户的取款。然后，我们将看看如何实现相同的场景，但是使用正常的程序流程而不是使用 BREs。

启动一个新的控制台应用程序，并添加两个名为`BankAccountUsingExceptions`和`BankAccountUsingProgramFlow`的文件夹。使用以下代码更新您的`void Main(string[] args)`方法：

```cs
private static void Main(string[] args)
{
    var usingBrExceptions = new UsingBusinessRuleExceptions();
    usingBrExceptions.Run();
    var usingPflow = new UsingProgramFlow();
    usingPflow.Run();
}
```

前面的代码运行每个情景。`UsingBusinessRuleExceptions()`演示了异常作为控制程序流程的预期输出的使用，而`UsingProgramFlow()`演示了在不使用异常条件的情况下控制程序流程的干净方式。

现在我们需要一个类来保存我们的活期账户信息。因此，在您的 Visual Studio 控制台项目中添加一个名为`CurrentAccount`的类，如下所示：

```cs
internal class CurrentAccount
{
    public long CustomerId { get; }
    public decimal AgreedOverdraft { get; }
    public bool IsAllowedToGoOverdrawn { get; }
    public decimal CurrentBalance { get; }
    public decimal AvailableBalance { get; private set; }
    public int AtmDailyLimit { get; }
    public int AtmWithdrawalAmountToday { get; private set; }
}
```

该类的属性只能通过构造函数内部或外部设置。现在，添加一个以客户标识符作为唯一参数的构造函数：

```cs
public CurrentAccount(long customerId)
{
    CustomerId = customerId;
    AgreedOverdraft = GetAgreedOverdraftLimit();
    IsAllowedToGoOverdrawn = GetIsAllowedToGoOverdrawn();
    CurrentBalance = GetCurrentBalance();
    AvailableBalance = GetAvailableBalance();
    AtmDailyLimit = GetAtmDailyLimit();
    AtmWithdrawalAmountToday = 0;
}
```

当前账户构造函数初始化所有属性。如前面的代码所示，一些属性是使用方法初始化的。让我们依次实现每个方法：

```cs
private static decimal GetAgreedOverdraftLimit()
{
    return 0;
}
```

`GetAgreedOverdraftLimit()`返回账户上约定的透支限额的值。在本例中，它被硬编码为零。但在实际情况中，它将从配置文件或其他数据存储中提取实际数字。这将允许非技术用户更新约定的透支限额，而无需开发人员更改代码。

`GetIsAllowedToGoOverdrawn()`确定账户是否可以透支，即使没有经过同意，有些银行是允许的。在这种情况下，我们只需返回`false`来确定账户无法透支：

```cs
private static bool GetIsAllowedToGoOverdrawn()
{
    return false;
}
```

为了本例的目的，我们将在`GetCurrentBalance()`方法中将用户的账户余额设置为 250 英镑：

```cs
private static decimal GetCurrentBalance()
{
    return 250.00M;
}
```

作为我们示例的一部分，我们需要确保即使用户的账户余额为 250 英镑，但其可用余额小于该金额，他们也无法取出超过可用余额的金额，因为这将导致透支。为此，我们将在`GetAvailableBalance()`方法中将可用余额设置为 173.64 英镑：

```cs
private static decimal GetAvailableBalance()
{
    return 173.64M;
}
```

在英国，ATM 机要么允许您最多取款 200 英镑，要么允许您最多取款 250 英镑。因此，在`GetAtmDailyLimit()`方法中，我们将将 ATM 每日限额设置为 250 英镑：

```cs
private static int GetAtmDailyLimit()
{
    return 250;
}
```

让我们通过使用业务规则异常和正常程序流程来处理程序中的不同条件，编写我们两种情景的代码。

## 示例 1 - 使用业务规则异常处理条件

向项目添加一个名为 `UsingBusinessRuleExceptions` 的新类，然后添加以下 `Run()` 方法：

```cs
public class UsingBusinessRuleExceptions
{
    public void Run()
    {
        ExceedAtmDailyLimit();
        ExceedAvailableBalance();
    }
}
```

`Run()` 方法调用两个方法：

+   第一个方法称为 `ExceedAtmDailyLimit()`。该方法故意超出了允许从 ATM 提取的每日金额。`ExceedAtmDailyLimit()` 导致 `ExceededAtmDailyLimitException`。

+   其次，调用 `ExceedAvailableBalance()` 方法，该方法故意引发 `InsufficientFundsException`。添加 `ExceedAtmDailyLimit()` 方法：

```cs
private void ExceedAtmDailyLimit()
{
     try
     {
            var customerAccount = new CurrentAccount(1);
            customerAccount.Withdraw(300);
            Console.WriteLine("Request accepted. Take cash and card.");
      }
      catch (ExceededAtmDailyLimitException eadlex)
      {
            Console.WriteLine(eadlex.Message);
      }
}
```

`ExceedAtmDailyLimit()` 方法创建一个新的 `CustomerAccount` 方法，并传入客户的标识符，表示为数字 `1`。然后，尝试提取 £300。如果请求成功，那么将在控制台窗口打印消息 `Request accepted. Take cash and card.`。如果请求失败，那么该方法会捕获 `ExceededAtmLimitException` 并将异常消息打印到控制台窗口：

```cs
private void ExceedAvailableBalance()
{
    try
    {
        var customerAccount = new CurrentAccount(1);
        customerAccount.Withdraw(180);
        Console.WriteLine("Request accepted. Take cash and card.");
    }
    catch (InsufficientFundsException ifex)
    {
        Console.WriteLine(ifex.Message);
    }
}
```

`ExceedAvailableBalance()` 方法创建一个新的 `CurrentAccount` 并传入客户标识符，表示为数字 `1`。然后尝试提取 £180。由于 `GetAvailableMethod()` 返回 £173.64，该方法导致 `InsufficientFundsException`。

通过这样，我们已经看到了如何使用业务规则异常来管理不同的条件。现在，让我们看看如何以正常的程序流程管理相同的条件，而不使用异常。

## 示例 2 - 使用正常程序流程处理条件

添加一个名为 `UsingProgramFlow` 的类，然后向其中添加以下代码：

```cs
public class UsingProgramFlow
{
    private int _requestedAmount;
    private readonly CurrentAccount _currentAccount;

    public UsingProgramFlow()
    {
        _currentAccount = new CurrentAccount(1);
    }
}
```

在 `UsingProgramFlow` 类的构造函数中，我们将创建一个新的 `CurrentAccount` 类并传入客户标识符。接下来，我们将添加 `Run()` 方法：

```cs
public void Run()
{
    _requestedAmount = 300;
    Console.WriteLine($"Request: Withdraw {_requestedAmount}");
    WithdrawMoney();
    _requestedAmount = 180;
    Console.WriteLine($"Request: Withdraw {_requestedAmount}");
    WithdrawMoney();
    _requestedAmount = 20;
    Console.WriteLine($"Request: Withdraw {_requestedAmount}");
    WithdrawMoney();
}
```

`Run()` 方法三次设置 `_requestedAmount` 变量。每次这样做时，在调用 `WithdrawMoney()` 方法之前，将在控制台窗口上打印提取的金额的消息。现在，添加 `ExceedsDailyLimit()` 方法：

```cs
private bool ExceedsDailyLimit()
{
    return (_requestedAmount > _currentAccount.AtmDailyLimit)
        || (_requestedAmount + _currentAccount.AtmWithdrawalAmountToday > _currentAccount.AtmDailyLimit);
}
```

`ExceedDailyLimit()` 方法如果 `_requestedAmount` 超过每日 ATM 提款限额，则返回 `true`。否则，返回 false。现在，添加 `ExceedsAvailableBalance()` 方法：

```cs
private bool ExceedsAvailableBalance()
{
    return _requestedAmount > _currentAccount.AvailableBalance;
}
```

`ExceedsAvailableBalance()` 方法如果请求的金额超过了可提取的金额，则返回 `true`。最后，我们来到最后一个方法，称为 `WithdrawMoney()`：

```cs
private void WithdrawMoney()
{
    if (ExceedsDailyLimit())
        Console.WriteLine("Cannot exceed ATM Daily Limit. Request denied.");
    else if (ExceedsAvailableBalance())
        Console.WriteLine("Cannot exceed available balance. You have no agreed 
         overdraft facility. Request denied.");
    else
        Console.WriteLine("Request granted. Take card and cash.");
}
```

`WithdrawMoney()` 方法不使用 BREs 来控制程序流程。相反，该方法调用布尔验证方法来确定程序流程。如果 `_requestedAmount` 超过了由调用 `ExceedsDailyLimit()` 确定的 ATM 每日限额，则请求被拒绝。否则，将进行下一个检查，以查看 `_requestedAmount` 是否超过了 `AvailableBalance`。如果是，则拒绝请求。如果不是，则执行授予请求的代码。

我希望您能看到，使用可用逻辑控制程序的流程比期望抛出异常更有意义。代码更清晰，更正确。异常应该保留给不属于业务需求的特殊情况。

当正确引发适当的异常时，对它们提供有意义的信息非常重要。晦涩的错误消息对任何人都没有好处，实际上可能会给最终用户或开发人员增加不必要的压力。现在，我们将看看如何在计算机程序引发的任何异常中提供有意义的信息。

# 异常应该提供有意义的信息

声明“没有错误”并终止程序的关键错误根本没有用。我亲身经历过实际的“没有错误”关键异常。这是一个阻止应用程序工作的关键异常。然而，消息告诉我们没有错误。好吧，如果没有错误，那么为什么屏幕上会出现关键异常警告？为什么我无法继续使用应用程序？显然，要引发关键异常，必须在某个地方发生了关键异常。但是在哪里，为什么？

当这些异常深植于你正在使用的框架或库中（你无法控制），并且你无法访问源代码时，这样的异常会变得更加恼人。这些异常导致程序员因沮丧而说出负面的话。我曾经有过这样的经历，也见过同事有同样的情况。沮丧的主要原因之一是代码引发了错误，用户或程序员已经被通知，但没有有用的信息来建议问题所在或查找位置，甚至采取什么补救措施。

异常必须提供对技术挑战者尤其友好的信息。在开发阅读障碍测试和评估软件的时候，我和许多教师和 IT 技术人员一起工作过。

可以说，许多各种能力水平的 IT 技术人员和教师在回应软件异常消息时经常一无所知。

我支持的软件的许多最终用户一直困惑的一个错误是**错误 76：路径未找到**。这是一个古老的微软异常，早在 Windows 95 时代就存在，今天仍然存在。对于引发此异常的软件的最终用户来说，错误消息是完全无用的。最终用户知道哪个文件和位置找不到，并知道应采取什么步骤来解决问题将是有用的。

一个潜在的解决方案是实施以下步骤：

1.  检查位置是否存在。

1.  如果位置不存在或访问被拒绝，则根据需要显示文件保存或打开对话框。

1.  将用户选择的位置保存到配置文件以供将来使用。

1.  在同一段代码的后续运行中，使用用户设置的位置。

但是，如果你要保留错误消息，那么你至少应该提供缺失的位置和/或文件的名称。

有了这些说法，现在是时候看看我们如何构建自己的异常，以提供对最终用户和程序员有用的信息了。但请注意：你必须小心，不要透露敏感信息或数据。

# 构建自定义异常

Microsoft .NET Framework 已经有许多可以引发的异常，你可以捕获。但可能会有一些情况，你需要一个提供更详细信息或在术语上更加用户友好的自定义异常。

因此，我们现在将看看构建自定义异常的要求是什么。构建自定义异常其实非常简单。你只需要给你的类一个以`Exception`结尾的名称，并继承自`System.Exception`。然后，你需要添加三个构造函数，如下面的代码示例所示：

```cs
    public class TickerListNotFoundException : Exception
    {
        public TickerListNotFoundException() : base()
        {
        }

        public TickerListNotFoundException(string message)
            : base(message)
        {
        }

        public TickerListNotFoundException(
            string message, 
            Exception innerException
        )
            : base(message, innerException)
        {
        }
    }
```

`TickerListNotFoundException`继承自`System.Exception`类。它包含三个必需的构造函数：

+   一个默认构造函数

+   一个接受异常消息文本字符串的构造函数

+   一个接受异常消息文本字符串和`Exception`对象的构造函数

现在，我们将编写并执行三种方法，这些方法将使用我们自定义异常的每个构造函数。您将能够清楚地看到使用自定义异常来创建更有意义的异常的好处：

```cs
static void Main(string[] args)
{
    ThrowCustomExceptionA();
    ThrowCustomExceptionB();
    ThrowCustomExceptionC();
}
```

上述代码显示了我们更新的`Main(string[] args)`方法，该方法已更新以依次执行我们的三种方法。这将测试每个自定义异常的构造函数：

```cs
private static void ThrowCustomExceptionA()
{
    try
    {
        Console.WriteLine("throw new TickerListNotFoundException();");
        throw new TickerListNotFoundException();
    }
    catch (Exception tlnfex)
    {
        Console.WriteLine(tlnfex.Message);
    }
}
```

`ThrowCustomExceptionA()`方法通过使用默认构造函数抛出一个新的`TickerListNotFoundException`。当您运行代码时，打印到控制台窗口的消息会通知用户已抛出`CH05_CustomExceptions.TickerListNotFoundException`：

```cs
private static void ThrowCustomExceptionB()
{
    try
    {
        Console.WriteLine("throw new 
         TickerListNotFoundException(Message);");
        throw new TickerListNotFoundException("Ticker list not found.");
    }
    catch (Exception tlnfex)
    {
        Console.WriteLine(tlnfex.Message);
    }
}
```

`ThrowCustomExceptionB()`通过使用接受文本消息的构造函数抛出一个新的`TickerListNotFoundException`。在这种情况下，最终用户被告知找不到股票列表：

```cs
private static void ThrowCustomExceptionC()
{
    try
    {
        Console.WriteLine("throw new TickerListNotFoundException(Message, 
         InnerException);");
        throw new TickerListNotFoundException(
            "Ticker list not found for this exchange.",
            new FileNotFoundException(
                "Ticker list file not found.",
                @"F:\TickerFiles\LSE\AimTickerList.json"
            )
        );
    }
    catch (Exception tlnfex)
    {
        Console.WriteLine($"{tlnfex.Message}\n{tlnfex.InnerException}");
    }
}
```

最后，`ThrowCustomExceptionC()`方法通过使用接受文本消息和内部异常的构造函数抛出`TickerListNotFoundException`。在我们的示例中，我们提供了一个有意义的消息，说明在该交易所找不到股票列表。内部的`FileNotFoundException`通过提供未找到的特定文件的名称来扩展这一点，这恰好是**伦敦证券交易所**（**LSE**）上的 Aim 公司的股票列表。

在这里，我们可以看到创建自定义异常的真正优势。但在大多数情况下，使用.NET Framework 中的内在异常应该就足够了。自定义异常的主要好处是它们是更有意义的异常，有助于调试和解决问题。

以下是 C#异常处理最佳实践的简要列表：

+   使用 try/catch/finally 块来从错误中恢复或释放资源。

+   处理常见条件而不抛出异常。

+   设计类以避免异常。

+   抛出异常而不是返回错误代码。

+   使用预定义的.NET 异常类型。

+   异常类的名称以单词**Exception**结尾。

+   在自定义异常类中包含三个构造函数。

+   确保在代码远程执行时可用异常数据。

+   使用语法正确的错误消息。

+   在每个异常中包含本地化的字符串消息。

+   在自定义异常中根据需要提供额外的属性。

+   放置 throw 语句，以便堆栈跟踪将有所帮助。

+   使用异常生成器方法。

+   当方法由于异常而无法完成时，恢复状态。

现在，是时候总结我们在异常处理方面学到的内容了。

# 总结

在本章中，您了解了已检查异常和未检查异常。已检查异常可以防止算术溢出条件进入任何生产代码，因为它们在编译时被捕获。未检查异常在编译时不被检查，通常会进入生产代码。这可能导致一些*难以跟踪*的错误在您的代码中通过意外数据值并最终导致抛出异常，导致程序崩溃。

然后，您了解了常见的`NullPointerException`以及如何使用自定义`Attribute`和`Validator`类来验证传入的参数，这些类放置在方法的顶部。这使您能够在验证失败时提供有意义的反馈。从长远来看，这将导致更健壮的程序。

然后，我们讨论了使用**BREs**来控制程序流程。您将学习如何通过期望异常输出来控制程序流程。然后，您将看到如何通过使用条件检查而不是使用异常来更好地控制计算机代码的流程。

讨论随后转向提供有意义的异常消息的重要性以及如何实现这一点；也就是说，通过编写继承自`Exception`类并实现所需的三个参数的自定义异常。通过提供的示例，你学会了如何使用自定义异常以及它们如何帮助更好地调试和解决问题。

所以，现在是时候通过回答一些问题来检验你所学到的知识了。如果你希望扩展本章学到的知识，还有进一步的阅读材料。

在下一章中，我们将学习单元测试以及如何先编写测试使其失败。然后，我们将编写足够的代码使测试通过，并在继续进行下一个单元测试之前对工作代码进行重构。

# 问题

1.  什么是已检查异常？

1.  什么是未检查异常？

1.  算术溢出异常是什么？

1.  什么是`NullPointerException`？

1.  你如何验证空参数以改进你的整体代码？

1.  BRE 代表什么？

1.  BRE 是好还是坏的实践，你为什么这样认为？

1.  BRE 的替代方案是什么，它是好还是坏，你为什么这样认为？

1.  你如何提供有意义的异常消息？

1.  编写自定义异常的要求是什么？

# 进一步阅读

+   [`docs.microsoft.com/en-us/dotnet/standard/exceptions/`](https://docs.microsoft.com/en-us/dotnet/standard/exceptions/)：这是处理和抛出.NET 异常的官方文档。

+   [`reflectoring.io/business-exceptions/`](https://reflectoring.io/business-exceptions/)：本文作者提供了五个原本认为 BRE 是一个好主意后认为它们是一个坏主意的原因。本文中还有一些本章未涉及的额外信息。

+   [`docs.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions`](https://docs.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions)：微软关于 C#异常处理的最佳实践，包括代码示例和解释。
