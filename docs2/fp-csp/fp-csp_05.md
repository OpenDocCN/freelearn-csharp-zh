

# 第五章：错误处理

欢迎来到*第五章*！您做得很好！在本章中，我们将讨论函数式编程提供的新方法来处理错误。我们将通过以下部分来实现这一点：

+   C#中的传统错误处理

+   结果类型

+   面向铁路编程（ROP）

+   设计自己的错误处理机制

+   功能性错误处理的实用技巧

+   传统与函数式错误处理比较

+   功能性错误处理中的模式和反模式

史蒂夫今天真的很沮丧，因为他花了过去三天的时间修复错误，以至于没有时间写哪怕一行新的代码。此外，基准测试显示，带有 try-catch 块的代码比没有它们的代码运行速度慢得多，而且他的代码中有很多这样的块。因此，他决定问茱莉亚是否有更好的方法来使用函数式方法处理错误。她给史蒂夫发了一篇关于使用函数式编程进行错误处理的详细文章。

如您所见，本章深入探讨了函数式技术，这将帮助您不仅处理错误，而且以干净、高效和可维护的方式进行错误处理。在我们深入探讨之前，让我们看看这三个自我评估任务。

# 任务 1 – 自定义错误类型和结果使用

这里是史蒂夫的塔防游戏中一个升级塔并返回布尔值的函数。将其重构为返回`Result`类型，当升级失败时返回自定义错误：

```cs
public bool UpgradeTower(Tower tower)
{
     // Tower upgrading logic...
     if (/* upgrade fails */)
     {
                  return false;
     }
     return true;
}
```

# 任务 2 – 利用 ROP 进行验证和处理

史蒂夫有一个涉及解析、验证和处理敌人生成的流程。使用**面向铁路编程**（**ROP**）重构它以改进错误处理流程：

```cs
public void ProcessEnemySpawn(string enemyData)
{
     var parsedData = ParseEnemyData(enemyData);
     if (parsedData.IsValid)
     {
                  var validation = ValidateEnemySpawn(parsedData);
                  if (validation.IsValid)
                  {
                      SpawnEnemy(validation.Enemy);
                  }
     }
}
```

# 任务 3 – 使用函数式技术实现重试机制

编写一个函数，实现一个用于故障塔射击操作的重试机制，并返回`Result`类型。该函数应在返回错误之前重试操作指定次数：

```cs
public bool TowerFire(Tower tower, Enemy enemy)
{
     // Sometimes works and returns true
     // sometimes doesn't and returns false
}
```

如果这些任务对您来说很容易，您现在可以自由地跳过这一章，稍后再回来，当您阅读了所有其他章节或对错误处理有任何疑问时。

# C#中的传统错误处理

每个 C#开发者，无论是新手还是专家，都遇到过 try-catch 块。它一直是防止意外行为和系统故障的主要保护措施。在我们了解函数式范式提供的内容之前，让我们回顾一下这种传统机制。

## try-catch 块

try-catch 块尝试执行操作，如果失败，控制权将转移到 catch 块，确保应用程序不会崩溃。例如，假设我们正在进行一个简单的文件读取操作：

```cs
string content;
try
{
    content = File.ReadAllText("file.txt");
}
catch (FileNotFoundException ex)
{
    content = string.Empty;
    LogException(ex, "File not found. Check the file location.");
}
catch (IOException ex)
{
    content = string.Empty;
    LogException(ex, "An IO error occurred. Try again.");
}
```

在这里，我们根据抛出的异常类型记录不同的消息。

## 异常

C#提供了两种主要的异常类型类别：

+   `NullReferenceException`、`IndexOutOfRangeException`，或者我们刚刚遇到的：`FileNotFoundException`和`IOException`。

+   **应用程序异常**：这些是为特定应用程序需求创建的自定义异常。比如说，你正在开发一个电子商务平台，你需要一个库存不足的异常。下面是如何设计它的示例：

    ```cs
    public class OutOfStockException : Exception
    {
        public OutOfStockException(string itemName) : base($"{itemName} is out of stock.") { }
    }
    ```

    之后，检查项目的库存：

    ```cs
    if(item.Stock <= 0)
    {
        throw new OutOfStockException(item.Name);
    }
    ```

自定义异常使开发者能够传达特定的错误场景，确保错误处理具有信息性。

史蒂夫坐在椅子上，脸上写满了挫败感。他花了过去三天的时间在与他的塔防游戏中的虫子作斗争，代码库正变成一个 try-catch 块的迷宫。就在这时，他的手机响了。是朱莉娅发来的消息。

朱莉娅：*游戏进展得怎么样了？*

史蒂夫：*不太好。我在错误处理中感到窒息。你有什么功能编程的智慧可以分享吗？*

朱莉娅：*事实上，我确实如此。让我告诉你一种更优雅的错误处理方式...*

# 结果类型

与在异常发生后处理异常相比，如果我们设计代码来预测并优雅地传达错误会怎样？请欢迎`Result`类型，这是功能错误处理的基石。

在其核心，`Result`类型封装了成功值或错误。这听起来可能和异常相似，但有一个关键的区别：错误成为了一等公民，直接影响应用程序的流程。

与只能区分现有值和非现有值的`Option`类型相比，`Result`类型描述了发生的错误，更重要的是，它可以用于链式方法应用。我们将在本章后面讨论这项技术。

例如，传统上，一个方法可能会返回一个值或抛出一个异常：

```cs
public Product GetProduct(int id)
{
    var product = _productRepository.Get(id);
    if(product is null)
    {
        throw new ProductNotFoundException($"Product with ID {id} was not found.");
    }
    return product;
}
```

相比之下，使用`Result`类型，方法更明确地传达了其意图和可能的失败：

```cs
public Result<Product, string> GetProduct(int id)
{
    var product = _productRepository.Get(id);
    if(product is null)
    {
        return Result<Product, string>.Fail($"Product with ID {id} not found.");
    }
    return Result<Product, string>.Success(product);
}
```

这段代码更明确。没有隐藏的异常。没有意外的行为。只有清晰、坦诚的沟通。

## 实现结果类型

让我们进一步深入，看看`Result`类型的通用实现是什么样的：

```cs
public class Result<T, E>
{
    private T _value;
    private E _error;
    public bool IsSuccess { get; private set; }
    private Result(T value, E error, bool isSuccess)
    {
        _value = value;
        _error = error;
        IsSuccess = isSuccess;
    }
    public T Value
    {
        get
        {
            if (!IsSuccess) throw new InvalidOperationException("Cannot fetch Value from a failed result.");
            return _value;
        }
    }
    public E Error
    {
        get
        {
            if (IsSuccess) throw new InvalidOperationException("Cannot fetch Error from a successful result.");
            return _error;
        }
    }
    public static Result<T, E> Success(T value) => new Result<T, E>(value, default, true);
    public static Result<T, E> Fail(E error) => new Result<T, E>(default, error, false);
}
```

## 使用`Result`类型

使用`Result`类型导致了一种更系统化的错误处理方法：

```cs
var productResult = GetProduct(42);
if (productResult.IsSuccess)
{
    DisplayProduct(productResult.Value);
}
else
{
    ShowError(productResult.Error);
}
```

没有更多的分散的 try-catch 块。现在错误只是代码可以采取的另一种路径，导致更可预测和可维护的系统。

`Result`类型从根本上改变了我们看待错误的方式：不是作为突然的中断，而是作为预期的结果。随着我们进一步探讨，你将看到这个功能工具如何与其他高级技术集成，从而创造一种新的错误管理方法。

# 面向铁路编程（ROP）

史蒂夫对`Result`类型很感兴趣，但他仍然有疑问。

史蒂夫：*这对单个操作来说很棒，但当我有一系列步骤都需要* *成功时怎么办呢？*

朱莉娅：*我很高兴你问了。让我向你介绍* *面向铁路编程（Railway-Oriented Programming）*。

函数式编程的核心是追求可预测性和清晰性。尽管传统的错误处理技术很强大，但它们通常会将错误处理逻辑分散在代码库中。受铁路道岔切换比喻的启发，ROP 提供了一种连贯、结构化的错误处理方法，使代码既具有表现力又简洁。

ROP 提供了一种管理一系列操作中错误的方法。将其视为管理两条并行轨道：成功轨道（快乐路径）和错误轨道。操作在快乐路径上顺序运行。然而，一旦遇到错误，流程就会转移到错误轨道，跳过后续操作。

## Bind 的本质

ROP 的核心是 Bind 函数。它接受一个操作和一个后续操作，如果第一个操作成功，则执行后续操作。然而，如果发生错误，它将跳过第二个操作，并立即传播错误：

```cs
public static Result<Tout, E> Bind<Tin, Tout, E>(this Result<Tin, E> input, Func<Tin, Result<Tout, E>> bindFunc)
{
    return input.IsSuccess ? bindFunc(input.Value) : Result<Tout, E>.Fail(input.Error);
}
```

## 使用 Bind 连接操作

想象一系列步骤，我们做以下操作：

1.  解析输入

1.  验证解析后的数据

1.  转换验证后的数据

1.  存储转换后的数据

ROP 允许我们将这些步骤表达为一个连贯的链：

```cs
public Result<bool, string> HandleData(string input)
{
    return ParseInput(input)
           .Bind(parsedData => ValidateData(parsedData))
           .Bind(validData => TransformData(validData))
           .Bind(transformedData => StoreData(transformedData));
}
```

## 使用 ROP 组合错误处理

ROP 的一个优势是它促进了可组合的错误处理。你的应用程序的每个组件都可以定义自己的错误场景，当这些组件连接在一起时，组合操作可以处理更广泛的错误范围，而不会失去粒度。

考虑为用户输入、业务逻辑和数据库操作分别创建独立的组件。每个组件可以有自己的错误定义。当这些组件的操作连接在一起时，系统可以无缝地处理来自任何组件的错误，从而创建一个统一的错误处理策略。

```cs
public Result<DBResponse, CompositeError> ProcessUserRequest(string userInput)
{
    return GetUserInput(userInput)
           .Bind(inputData => ApplyBusinessLogic(inputData))
           .Bind(businessData => UpdateDatabase(businessData));
}
```

在这里，`CompositeError` 可能会封装来自输入验证、业务逻辑违规和数据库故障的错误。

## 处理多种错误类型

直接的 ROP 实现的一个挑战是它假设整个链中有一个统一的错误类型。然而，现实场景往往涉及多种错误类型。为了管理这一点，你可以引入一种机制来转换或映射错误类型：

```cs
public static Result<TOut, EOut> Bind<TIn, TOut, EIn, EOut>(
    this Result<TIn, EIn> input,
    Func<TIn, Result<TOut, EOut>> bindFunc,
    Func<EIn, EOut> errorMap)
{
    return input.IsSuccess ? bindFunc(input.Value) : Result<TOut, EOut>.Fail(errorMap(input.Error));
}
```

此增强的 `Bind` 函数将一种错误类型映射到另一种类型，从而实现更复杂和多样化的错误处理场景。

## 隔离的优势

ROP 隔离了错误处理，确保你的主要业务逻辑保持整洁。在阅读核心操作时，可以专注于主要逻辑，而不会被错误处理的复杂性所分散。

对于开发者来说，这种隔离简化了认知负荷。他们可以信任系统处理错误，并专注于构建主要逻辑。在调试时，ROP 的结构化特性使得问题可能偏离轨道的地方一目了然，从而简化了故障排除过程。

## 扩展 ROP 以支持异步操作

在现代应用程序中，许多方法都是异步的。可以使用 `BindAsync` 等技术将 ROP 适应异步操作：

```cs
public static async Task<Result<TOut, E>> BindAsync<TIn, TOut, E>(
    this Result<TIn, E> input,
    Func<TIn, Task<Result<TOut, E>>> bindFuncAsync)
{
    return input.IsSuccess ? await bindFuncAsync(input.Value) : Result<TOut, E>.Fail(input.Error);
}
```

使用 `BindAsync`，你现在可以像同步操作一样轻松地链式调用异步操作，使 ROP 在同步和异步环境中都变得灵活。

深入研究 ROP 后，我们见证了我们在感知和处理错误方面的范式转变。我们不再将错误视为异常事件，而是将它们整合到我们应用程序的逻辑中，从而产生更健壮、可读性和可维护的代码。

# 设计你自己的错误处理机制

当 Steve 开始重构他的代码时，他意识到预构建的解决方案并不完全适合他游戏的所有独特场景。

Steve: *Julia，我认为我需要为我的游戏创建一些自定义的错误类型。这样可以吗？*

Julia: *非常好。事实上，让我们谈谈你如何设计适合你* *游戏需求* *的自定义错误处理机制。*

在创建自己的功能性错误处理时，`Result` 类型是一个很好的起点。让它足够通用，以适应不同的场景：

```cs
public class Result<TSuccess, TFailure>
{
    public TSuccess SuccessValue { get; }
    public TFailure FailureValue { get; }
    public bool IsSuccess { get; }
    //... Constructors and other methods ...
}
```

## 使用工厂方法进行创建

工厂方法提供清晰性和易用性：

```cs
public static class Result
{
    public static Result<T, string> Success<T>(T value) => new Result<T, string>(value, default, true);
    public static Result<T, string> Fail<T>(string error) => new Result<T, string>(default, error, false);
}
```

使用方法如下：

```cs
var successResult = Result.Success("Processed!");
var errorResult = Result.Fail("Oops! Something went wrong.");
```

## 使用 Bind 扩展

使用 `Bind` 方法来增加流畅性：

```cs
public Result<TOut, TFailure> Bind<TOut>(Func<TSuccess, Result<TOut, TFailure>> func)
{
    return IsSuccess ? func(SuccessValue) : new Result<TOut, TFailure>(default, FailureValue, false);
}
```

## 自定义错误类型

而不是仅仅使用字符串，创建特定的错误类型来传达详细的信息：

```cs
public class ValidationError
{
    public string FieldName { get; }
    public string ErrorDescription { get; }
    //… Constructor and methods ...
}
```

然后，按如下方式使用它们：

```cs
public Result<User, ValidationError> ValidateUser(User user)
{
    if (string.IsNullOrEmpty(user.Name))
    {
        return Result.Fail<User, ValidationError>(new ValidationError("Name", "Name cannot be empty."));
    }
    //... Other validations ...
    return Result.Success<User, ValidationError>(user);
}
```

## 利用扩展方法

扩展方法可以提供增强的可读性：

```cs
public static class ResultExtensions
{
    public static bool IsFailure<TSuccess, TFailure>(this Result<TSuccess, TFailure> result)
    {
        return !result.IsSuccess;
    }
}
```

按如下方式使用它们：

```cs
if (result.IsFailure())
{
    // Handle the failure scenario
}
```

## 与现有代码集成

通过封装旧方法，我们可以无缝地与非功能性代码集成：

```cs
public static Result<T, Exception> TryExecute<T>(Func<T> action)
{
    try
    {
        return Result.Success(action());
    }
    catch (Exception ex)
    {
        return Result.Fail<T, Exception>(ex);
    }
}
```

## 总是迭代和改进

自定义错误机制是活生生的实体。随着你的应用程序增长，根据反馈和新要求不断迭代和改进。

设计你自己的错误处理机制不仅赋予你量身定制的解决方案，而且加深了你对于函数式范式的理解。深入其中，动手实践，看看你的应用程序如何成为健壮性和清晰性的典范。

# 功能性错误处理的实用技巧

改进了一周后，Steve 取得了一些进展，但他感到所有这些新概念让他感到不知所措。

Steve: *我不确定我是否* *做对了…*

Julia: *别担心，在学习新的范式时感到这样很正常。让我分享一些实用的技巧，帮助你在这个* *新领域* *中导航。*

## 使用选项避免 null

努力不要返回 null。听起来很简单，然而这是一个等待绊倒粗心大意的用户的陷阱。为什么？因为 null 很容易获得，然而，正确处理它们要困难得多，如果你做得不好，可能会导致级联失败：

```cs
public User FindUser(string login)
{
    // This can return null!
    return users.FirstOrDefault(u => u.Login.Equals(login));
}
```

转换如下：

```cs
public Option<User> FindUser(string login)
{
    var user = users.FirstOrDefault(u => u.Login.Equals(login));
    return user is not null ? Option.Some(user) : Option.None<User>();
}
```

## 记录错误

记录是至关重要的，但尽量避免破坏功能性方法的副作用。一个好的想法是将记录行为委托出去，保持函数的纯洁性：

```cs
public Result<Order, Error> ProcessOrder(int id, Action<string> logError)
{
    if (invalid(id))
    {
        logError($"Invalid order id: {id}");
        return Result.Fail<Order, Error>(new Error("Invalid ID"));
    }
    // ... process further ...
}
```

## 两种替换异常的策略

我知道在某些情况下，你的第一反应可能是使用 try-catch。抵制。使用这些策略来坚持函数式范式。

### 安全执行

创建一个以无异常方式执行任何代码的方法：

```cs
public static Result<T, E> SafelyExecute<T, E>(Func<T> function, E error)
{
    try
    {
        return Result.Success(function());
    }
    catch
    {
        return Result.Fail<T, E>(error);
    }
}
```

使用方法如下：

```cs
var orderResult = SafelyExecute(() => GetOrder(orderId), new DatabaseError("Failed getting order"));
```

### 回退

对于某些操作，你可以提供一个回退结果：

```cs
public Result<Order, string> GetOrderWithFallback(int orderId)
{
    var orderResult = GetOrder(orderId);
    return orderResult.IsSuccess ? orderResult : Result.Success(new DefaultOrder());
}
```

## 预测错误 - 使其可预测

而不是等待错误，预测它们。在使用之前验证你的数据。你可以手动完成或借助守卫子句：

```cs
public Result<Order, string> ProcessOrder(int id)
{
    if (id < 0)
    {
        return Result.Fail<Order, string>("ID cannot be negative.");
    }
    // ... further processing ...
}
```

## 拥抱组合

使用函数组合进行更清晰的错误处理：

```cs
var result = GetData()
             .Bind(Validate)
             .Bind(Process)
             .Bind(Save);
```

## 教育你的团队

最后，确保每个人都同意。一致的错误处理方法确保了清晰性和可靠性。

# 传统与函数式错误处理比较

每个开发者迟早都会遇到：编码时遇到错误。但随着编码世界的改变，我们处理这些问题的方法也在改变。让我们看看 C#中处理错误的老旧和新旧方法之间的明显差异，并了解为什么这种新方法越来越受欢迎。

## 传统方式

在传统的面向对象编程中，异常是首选机制：

+   **抛出异常**：当事情出错时，我们依赖于系统或自定义异常：

    ```cs
    public User GetUser(int id)
    {
        if (id < 0)
            throw new ArgumentOutOfRangeException(nameof(id));
        // ... fetch the user ...
    }
    ```

+   **捕获异常**：使用 try-catch 块来处理和可能恢复错误：

    ```cs
    try
    {
        var user = GetUser(-5);
    }
    catch (ArgumentOutOfRangeException ex)
    {
        Console.WriteLine(ex.Message);
    }
    ```

传统方式的优点如下：

+   大多数开发者习惯于使用异常，使其成为一个被广泛理解的方法

+   可以使用不同的异常类型进行细粒度错误处理

然而，也有一些缺点：

+   异常可能会打断代码执行的正常流程

+   异常处理引入了性能开销

+   这可能很难推理，可能导致“异常地狱”

## 函数式方式

函数式编程更喜欢一种更优雅的错误处理形式：

+   **使用如下结构**：**结果类型**

    ```cs
    public Result<User, string> GetUser(int id)
    {
        if (id < 0)
        {
            return Result.Fail<User, string>("ID cannot be negative.");
        }
        // ... get and return the user ...
    }
    ```

    消费前面的函数变得简单：

    ```cs
    var userResult = GetUser(-5);
    if (userResult.IsFailure)
    {
        Console.WriteLine(userResult.Error);
    }
    ```

+   **ROP**：

    ```cs
    var result = GetData()
                 .Bind(ValidateData)
                 .Bind(ProcessData);
    ```

函数式方式的优点如下：

+   更清晰的意图和流程

+   避免异常性能开销

+   更容易的操作链

同样也有一些缺点：

+   开发者可能需要学习新的概念

+   与传统异常相比，粒度更少

## 比较分析

让我们更深入地看看在性能、可读性和可维护性方面，传统异常处理与函数式编程之间的差异：

+   **性能**：由于创建异常对象和回滚堆栈的开销，传统的异常处理可能会较慢。函数式编程提供了一个更可预测的性能配置文件。

+   **可读性**：try-catch 块可能会使代码杂乱无章，使其可读性降低。通常这些块还包含无法抛出异常的代码。函数式编程将错误封装在数据中，使代码流程明显。

+   **可维护性**：传统方法分散错误处理，使维护变得复杂。函数式编程鼓励隔离的、纯函数，这简化了调试和测试。

## 转变

从函数式错误处理开始可能看起来很奇怪，尤其是如果你一直生活在传统范式下。但一旦你做出改变，好处是深远的。记住：

+   从小开始。重构你的代码库的一部分，并观察差异。

+   拥抱纯函数。它们将简化你的错误处理故事。

+   教育你的团队。共同的理解至关重要。

总之，虽然传统的错误处理多年来一直为我们服务得很好，但函数式范式提供了一种更清新、更系统的方法。通过在我们的数据类型中将错误表示为一等公民，我们编写了更易于维护的代码。两者之间的选择通常取决于问题域、团队熟悉度和项目需求。但如果你想要清晰和表述性，函数式路径是正确的。

# 函数式错误处理中的模式和反模式

函数式编程重新定义了我们对错误处理的方法。通过将错误引入数据领域，我们确保了更安全、更可预测的代码。但与任何范式一样，有正确的方法和陷阱。让我们看看可以帮助你的模式以及反模式。

有几种模式可以帮助你以更函数式的方式处理错误。这些模式旨在提高代码的质量、可读性和可维护性。以下是一些需要考虑的关键模式：

+   **丰富的自定义** **错误类型**

    而不是使用通用的字符串或代码，使用详细的数据类型来描述错误：

    ```cs
    public Result<User, UserError> GetUser(int id)
    {
        if (id < 0)
            return Result.Fail<User, UserError>(new InvalidIdError(id));
        // ... other checks and logic ...
    }
    ```

+   **利用组合**

    无缝地链式调用多个函数以保持清晰的逻辑流程：

    ```cs
    GetData()
        .Bind(Validate)
        .Bind(Process)
        .Bind(Store);
    ```

+   **模式匹配** **与错误**

    这确保了每个错误场景都得到了处理：

    ```cs
    switch (GetUser(5))
    {
        case Success<User> user:
            // Handle user
            break;
        case Failure<UserError> error when error.Value is InvalidIdError:
            // Handle invalid ID error
            break;
        // ... other cases ...
    }
    ```

+   **隔离** **副作用**

    保持你的核心逻辑纯净，并单独处理副作用，如日志记录或 I/O：

    ```cs
    public Result<TSuccess, TError> ComputeValue<TSuccess, TError>(Data data)
    {
        if (data.IsValid())
        {
            TSuccess value = PerformComputation(data);
            return new Result<TSuccess, TError>(value);
        }
        else
        {
            TError errorDetails = GetErrorDetails(data);
            return new Result<TSuccess, TError>(errorDetails);
        }
    }
    ```

    使用方法如下：

    ```cs
    var result = ComputeValue<MySuccessType, MyErrorType>(data);
    if (result.IsSuccess)
    {
        HandleSuccess(result.Value);
    }
    else
    {
        HandleError(result.Error);
    }
    ```

同时也有一些反模式可能会使你的错误处理更加复杂和容易出错：

+   `Result` 类型可能会让其他软件开发者感到困惑：

    ```cs
    public Result<int, string> Compute()
    {
        if (condition)
        {
            throw new Exception("Oops!");
        }
        // ... return some result ...
    }
    ```

+   **不明确的错误**

    返回模糊的错误消除了函数式编程的表述性错误处理的价值：

    ```cs
    return Result.Fail<User, string>("Something went wrong.");
    ```

+   **忽略错误**

    只获取值而不处理潜在的错误破坏了函数式编程错误处理的概念。一个例子是当你有一个 Result 类型作为方法输出，但没有检查它：

    ```cs
    var result = GetData();
    ProcessData(result.Value);
    ```

+   **使用自定义类型** **过度复杂化**

    虽然详细的错误类型是有益的，但为每个微小的偏差创建一个可能会使事情过于复杂。请不要这样做错误：

    ```cs
    public class NameMissingFirstCharacterError : NameError { /* ... */ }
    public class NameMissingLastCharacterError : NameError { /* ... */ }
    ```

## 练习

史蒂夫渴望将他对函数式错误处理的新知识应用到他的塔防游戏中。朱莉娅对他的热情印象深刻，向他提出了三个挑战来测试他的理解并改进他的代码。

## 练习 1

这是升级塔并返回布尔值的函数。将其重构为返回一个`Result`类型，当支付失败时返回自定义错误：

```cs
public bool UpgradeTower(Tower tower)
{
     // Tower upgrading logic...
     if (/* upgrade fails */)
     {
                  return false;
     }
     return true;
}
```

## 练习 2

史蒂夫有一个涉及解析、验证和处理敌人生成的流程。使用面向铁路编程重构它，以改进错误处理流程：

```cs
public void ProcessEnemySpawn(string enemyData)
{
     var parsedData = ParseEnemyData(enemyData);
     if (parsedData.IsValid)
     {
              var validation = ValidateEnemySpawn(parsedData);
              if (validation.IsValid)
              {
                  SpawnEnemy(validation.Enemy);
              }
     }
}
```

## 练习 3

编写一个函数，实现一个用于故障操作的重试机制，并返回一个`Result`类型。该函数应在返回错误之前重试操作指定次数：

```cs
public bool TowerFire(Tower tower, Enemy enemy)
{
     // Sometimes works and returns true
     // sometimes doesn't and returns false
}
```

尝试自己完成这些练习，完成后，你可以用以下解决方案来检查你的工作。

# 解决方案

## 练习 1

将方法重构为使用带有自定义错误的`Result`类型，封装失败详情：

```cs
public enum TowerUpgradeError
{
     InsufficientResources,
     MaxLevelReached,
     TowerDestroyed
}
public Result<bool, TowerUpgradeError> UpgradeTower(Tower tower)
{
     // Tower upgrading logic...
     if (/* insufficient resources */)
     {
                  return Result.Fail<bool, TowerUpgradeError>(TowerUpgradeError.InsufficientResources);
     }
     else if (/* max level reached */)
     {
                  return Result.Fail<bool, TowerUpgradeError>(TowerUpgradeError.MaxLevelReached);
     }
     else if (/* tower is destroyed */)
     {
                  return Result.Fail<bool, TowerUpgradeError>(TowerUpgradeError.TowerDestroyed);
     }
     return Result.Ok<bool, TowerUpgradeError>(true);
}
```

## 练习 2

史蒂夫使用面向铁路编程重构了他的敌人生成系统，创建了一个处理敌人数据的清晰管道：

```cs
public Result<Enemy, EnemySpawnError> ProcessEnemySpawn(string enemyData)
{
     return ParseEnemyData(enemyData)
                  .Bind(ValidateEnemySpawn)
                  .Bind(SpawnEnemy);
}
// Assume these methods are implemented to return Result<T, EnemySpawnError>
public Result<ParsedEnemyData, EnemySpawnError> ParseEnemyData(string data) { /* ... */ }
public Result<ValidatedEnemy, EnemySpawnError> ValidateEnemySpawn(ParsedEnemyData data) { /* ... */ }
public Result<Enemy, EnemySpawnError> SpawnEnemy(ValidatedEnemy enemy) { /* ... */ }
```

## 练习 3

对于故障的塔发射机制，史蒂夫实现了一个重试函数，在放弃之前尝试多次操作：

```cs
public Result<bool, string> TryTowerFire(Tower tower, Enemy enemy, int maxRetries)
{
     for (int attempt = 0; attempt < maxRetries; attempt++)
     {
                  if (TowerFire(tower, enemy))
                  {
                      return Result.Ok<bool, string>(true);
                  }
     }
     return Result.Fail<bool, string>($"Tower firing failed after {maxRetries} attempts.");
}
```

这些练习将带你从理解到在实际编码场景中应用函数式原则。它们鼓励你以函数式的方式思考和编码，将错误处理视为编码过程的一个组成部分，而不是事后考虑：

# 摘要

在本章中，我们从传统的错误处理方法进步到函数式方法。我们确定了函数式编程的优势、挑战、模式和反模式。

函数式编程不仅提供了一种编码方式，而且是一种思维方式的转变。通过将错误视为数据，我们受益于类型安全、表达性和可预测性。

然而，我们的目标不是消除所有异常和空值，而是创建更易于阅读和健壮的软件。幸运的是，随着 C#的发展，函数式错误处理正变得更容易和更集成。

就像所有范式一样，函数式编程不是万能的。虽然将错误视为数据可能很强大，但你必须记住代码运行的实际情况。网络故障、数据库中断和硬件故障是现实。在函数式纯度和现实世界实用主义之间取得平衡是关键。

在本章中，我们几次使用了委托，为了更好地理解它们以及在函数式编程中的作用，在下一章中我们将深入探讨高阶函数和委托的概念。
