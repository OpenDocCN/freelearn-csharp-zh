

# 第七章：函子和单子

从高阶函数和委托过渡到函子，我们进入了函数式编程的世界，函子是其中的关键角色。它们允许我们以结构化的方式处理包装值，如列表或计算结果。本章探讨了以下内容：

+   函子

+   函子法则

+   应用函子和法则

+   单子与单子法则

如同往常，以下三个自我检查任务可以帮助你理解函子和单子的现有知识。

# 任务 1 – 函子使用

给定一个表示塔列表的 `Result<List<Tower>, string>` 类型，其中 `Tower` 是一个包含 `Id`、`Name` 和 `Damage` 等属性的类，任务是用函子概念将一个函数应用到每个塔上，在名称末尾添加“(升级)”以表示塔已被升级：

```cs
public class Tower
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Damage { get; set; }
}
public Result<List<Tower>, string> UpgradeTowers(List<Tower> towers)
{
    // Write your code here
}
```

# 任务 2 – 应用函子

想象你有两个被 `Result` 类型包裹的函数：`Result<Func<Tower, bool>, string> ValidateDamage` 检查塔的伤害是否在可接受范围内，而 `Result<Func<Tower, bool>, string> ValidateName` 检查塔的名称是否符合某些标准。给定一个表示单个塔的 `Result<Tower, string>`，使用应用函子将这两个验证函数应用到塔上，确保两个验证都通过：

```cs
public Result<Func<Tower, bool>, string> ValidateDamage = new Result<Func<Tower, bool>, string>(tower => tower.Damage < 100);
public Result<Func<Tower, bool>, string> ValidateName = new Result<Func<Tower, bool>, string>(tower => tower.Name.Length > 5 && !tower.Name.Contains("BannedWord"));
```

# 任务 3 – 单子使用

给定一系列用于升级塔的操作——`FetchTower`、`UpgradeTower` 和 `DeployTower`——每个操作都可能失败并返回 `Result<Tower, string>`，使用单子概念将这些操作串联起来，针对给定的塔 ID。确保如果任何步骤失败，整个操作将短路并返回错误：

```cs
public Result<Tower, string> FetchTower(int towerId) { /* Fetches tower based on ID */ }
public Result<Tower, string> UpgradeTower(Tower tower) { /* Upgrades the tower and can fail */ }
public Result<Tower, string> DeployTower(Tower tower) { /* Attempts to deploy the tower */ }
```

如果你成功完成了所有三个任务，你真是太棒了！如果你现在遇到困难或者不知道如何解决这些任务，不要担心，完成这一章并解决这些问题后，你也会很棒。

# 什么是函子？

朱莉娅决定从史蒂夫的游戏中找一个类比来解释函子的概念。

朱莉娅：*想象函子是你的塔防游戏中的一个特殊升级站。这个站可以提升任何塔，但总是输出一个塔——只是一个* *改进版本*。

史蒂夫：*这很有道理。所以它就像是一种一致的方式来转换事物，而不改变它们的* *核心本质*？

术语**函子**起源于范畴论，这是一个处理复杂结构和映射的数学领域。在编程的世界里，我们采用这个概念的简化版本，使其适用于数据处理。简单来说，函子是特殊的容器，可以持有数据，并且能够将函数应用到它们持有的每一份数据上，同时保持整体结构不变。想象它们就像魔法盒子，可以改变里面的东西，而不会改变盒子本身。

然而，并非每个可以对其元素应用函数的数据容器都是函子。一个容器要被视为函子，需要遵守两个定律：

将`identity`函数映射到函子应该得到相同的函子。换句话说，如果你将`identity`函数映射到一个函子上，该函子应该保持不变。

**组合定律**：将两个函数组合起来，然后将结果函数映射到函子上，应该与首先将一个函数映射到函子上，然后映射另一个函数相同。这意味着函子映射应该是可组合的，不依赖于它们应用的顺序。

我知道这听起来可能有点复杂，所以让我们详细讨论这些定律。

## 恒等定律

恒等定律指出，将`identity`函数应用于我们的容器返回相同的容器。“恒等函数”在这里是一个总是返回其输入的函数。在代码中，我们可以用`Func<T, T>`来表示它：

```cs
Func<T, T> identity = x => x;
```

这个函数的使用可以展示如下：

```cs
int number = 29;
int result = identity(number);
Console.WriteLine(result);
  // Output: 29
string text = "Hello Functional programming in C# readers!";
string resultText = identity(text);
Console.WriteLine(resultText);
  // Output: Hello Functional programming in C# readers!
```

初看起来，`identity`函数似乎没有用，但在函数式编程中的数学证明中它起着重要作用。回到函子，恒等定律意味着如果我们将恒等函数映射到我们的容器上，我们应该得到相同的结果。让我们继续讨论第二个定律。

## 组合定律

这个定律指出，要么我们将两个函数组合起来，然后将结果映射到我们的容器上，要么依次映射这些函数；结果将是相同的。为了理解它如何应用于我们的代码，让我们首先创建两个函数：

```cs
Book AddPages(Book book, int pages) => new Book { Title = book.Title, Pages = book.Pages + pages };
Book AppendSubtitle(Book book, string subtitle) => new Book { Title = $"{book.Title}: {subtitle}", Pages = book.Pages };
```

一个函数向书中添加页面并返回它；另一个向书的标题添加副标题并返回结果。相当简单，对吧？现在，借助这些函数，我们可以用代码表达我们的组合定律：

```cs
List<Book> books = new()
{
    new Book { Title = "C# Basics", Pages = 100 },
    new Book { Title = "Advanced C#", Pages = 200 }
};
// Apply AddPages to each book and then apply AppendSubtitle
var sequentialApplicationResult = books.Select(book => AddPages(book, 50)).Select(book => AppendSubtitle(book, "Updated Edition"));
// Apply AddPages then AppendSubtitle to each book
var combinedApplicationResult = books.Select(book => AppendSubtitle(AddPages(book, 50), "Updated Edition"));
// Print the results
Console.WriteLine("books.Select(AddPages).Select(AppendSubtitle): " + string.Join(", ", sequentialApplicationResult.Select(b => b.Title)));
Console.WriteLine("books.Select(book => AppendSubtitle(AddPages(book, 50))): " + string.Join(", ", combinedApplicationResult.Select(b => b.Title)));
// Output:
// books.Select(AddPages).Select(AppendSubtitle): C# Basics: Updated Edition, Advanced C#: Updated Edition
// books.Select(book => AppendSubtitle(AddPages(book, 50))): C# Basics: Updated Edition, Advanced C#: Updated Edition
```

如我们从输出中可以理解的，生成的集合是相等的，因此我们的`List<Book>`容器遵守组合定律。

## 创建我们自己的函子

注意，一个容器不需要包含一组元素就可以被视为函子。让我们回忆一下在*第五章*中使用的`Result`类型，并通过添加`Map`方法来增强它，以便能够应用于内部值：

```cs
public class Result<TValue, TError>
{
    private TValue _value;
    private TError _error;
    public bool IsSuccess { get; private set; }
    private Result(TValue value, TError error, bool isSuccess)
    {
        _value = value;
        _error = error;
        IsSuccess = isSuccess;
    }
    public TValue Value
    {
        get
        {
            if (!IsSuccess) throw new InvalidOperationException("Cannot fetch Value from a failed result.");
            return _value;
        }
    }
    public TError Error
    {
        get
        {
            if (IsSuccess) throw new InvalidOperationException("Cannot fetch Error from a successful result.");
            return _error;
        }
    }
    public static Result<TValue, TError> Success(TValue value) => new Result<TValue, TError>(value, default, true);
    public static Result<TValue, TError> Failure(TError error) => new Result<TValue, TError>(default, error, false);
    public Result<TResult, TError> Map<TResult>(Func<TValue, TResult> mapper)
    {
        return IsSuccess
            ? Result<TResult, TError>.Success(mapper(_value!))
            : Result<TResult, TError>.Failure(_error!);
    }
}
```

`Map`方法在容器包含值时将传入的函数应用于该值；否则，不调用任何函数，并返回错误结果。由于`Result`类型现在可以对底层值应用函数，它开始遵守两个函子定律。让我们通过以下示例来查看：

```cs
Book AddPages(Book book, int pages) => new Book { Title = book.Title, Pages = book.Pages + pages };
Book AppendSubtitle(Book book, string subtitle) => new Book { Title = $"{book.Title}: {subtitle}", Pages = book.Pages };
Func<Book, Book> identity = book => book;
var success = Result<Book, string>.Success(new Book { Title = "C# Basics", Pages = 100 });
var error = Result<Book, string>.Failure("Error message");
// Identity law
var successAfterIdentity = success.Map(identity);
// successAfterIdentity should have value "C# Basics", 100 pages
var errorAfterIdentity = error.Map(identity);
// errorAfterIdentity should have the "Error message" error
// Composition law
Func<Book, Book> composedFunction = book => AppendSubtitle(AddPages(book, 50), "Updated Edition");
var success = Result<Book, string>.Success(new Book { Title = "C# Basics", Pages = 100 });
// Applying composed function directly
var directComposition = success.Map(composedFunction);
// directComposition should hold value "C# Basics: Updated Edition", 150 pages
// Applying functions one after the other
var stepwiseComposition = success.Map(book => AddPages(book, 50)).Map(book => AppendSubtitle(book, "Updated Edition"));
// stepwiseComposition should also hold value "C# Basics: Updated Edition", 150 pages
```

虽然我们现在不能直接检索内部值，但可以使用调试器或`LinqPad`中的`Dump()`扩展方法来查看。正如我们所看到的，我们的`Result`类型变成了一个函子。

## 函子优势

将我们的`Result<TValue, TError>`类型转换为函子提供了几个简洁的优点，增强了函数式编程中的错误处理和操作结果：

+   **简化的错误处理**：将成功和错误结果整合到一个结构中，简化错误管理

+   **可组合的操作**：促进在成功结果上链式操作，具有自动错误传播，提高代码的可重用性

+   `Map`函数的意图很明确——在成功时转换值，或在出错时跳过，使代码更易于理解

+   **类型安全和清晰**：类型签名中的显式成功和错误状态增强了可预测性和安全性，确保全面处理结果

听起来很吸引人，对吧？我们可以通过将其制作成一个应用函子来使我们的类变得更好。

# 应用函子

**应用函子**是一种允许将封装在函子内的函数应用于也封装在函子内的值的函子类型。这个概念可以转化为实现能够优雅地处理多层计算上下文的操作，例如错误处理或异步操作。

当函子允许我们将函数应用于封装的值时，应用函子通过允许将函数本身封装在上下文中来扩展这种能力。这种区别对于函数应用本身可能导致计算上下文（如失败、延迟或不确定性）的操作至关重要。让我们通过图书出版系统示例来看看这种区别。

考虑一个基于书籍销售计算版税的函数，以及另一个根据市场条件调整这些版税的函数。这两个函数可能由于各种原因而失败，它们的输出可能被封装在`Result`类型中以表示成功或失败。应用函子允许我们将这些可能失败的函数应用于可能失败的输入，协调复杂操作，优雅地处理多层潜在失败。

```cs
Result<Func<int, decimal>, string> CalculateRoyaltiesFunc = new Result<Func<int, decimal>, string>(sales => sales * 0.1m);
Result<Func<decimal, decimal>, string> AdjustRoyaltiesFunc = new Result<Func<decimal, decimal>, string>(royalties => royalties * 1.05m);
```

在这个例子中，`CalculateRoyaltiesFunc`是一个函数，它接受销售数量并计算版税为销售额的 10%。`AdjustRoyaltiesFunc`是一个函数，它接受一个初始版税金额，并通过乘以`1.05`的因子来调整以反映市场条件。

现在，让我们假设我们有一个`Result<int, string>`表示书籍销售数量，它也可能失败：

```cs
Result<int, string> salesResult = new Result<int, string>(150);
```

为了计算调整后的版税，我们首先将`CalculateRoyaltiesFunc`应用于`salesResult`，然后将`AdjustRoyaltiesFunc`应用于结果：

```cs
var royaltiesResult = salesResult
    .Apply(CalculateRoyaltiesFunc)
    .Apply(AdjustRoyaltiesFunc);
// the royaltiesResult holds the value 15.75
```

为了更好地理解，让我们假设我们的第一个函数返回一个错误：

```cs
Result<Func<int, decimal>, string> CalculateRoyaltiesFunc = Result<Func<int, decimal>, string>.Failure("Can't calculate royalties");
```

如果我们再次尝试计算 `royaltiesResult`，`IsSuccess` 将为 `false`，并且 `Error` 属性将包含字符串 `"Can't calculate royalties"`。如果 `AdjustRoyaltiesFunc` 调用导致错误，也会发生相同的情况。两种方法都可能失败；然而，多亏了 `Apply` 方法，我们可以以安全的方式调用它们。听起来很棒，但这个 `Apply` 方法看起来是什么样子呢？

## 应用方法实现

为了实现应用函子模式，我们引入了 `Apply` 方法。该方法接受包含函数的 `Result`，并在当前 `Result` 实例中的值成功的情况下将其应用于该值。如果函数或值被包装在失败的 `Result` 中，`Apply` 方法会传播错误：

```cs
public Result<TResult, TError> Apply<TResult>(Result<Func<TValue, TResult>, TError> resultFunc)
{
    if (resultFunc.IsSuccess && this.IsSuccess)
    {
        return Result<TResult, TError>.Success(resultFunc.Value(this.Value));
    }
    else
    {
        var error = resultFunc.IsSuccess ? this._error! : resultFunc.Error;
        return Result<TResult, TError>.Failure(error);
    }
}
```

如您所见，这里没有什么特别之处。首先，我们确保当前容器状态和传入的 `Result<Func<TValue, TResult>, TError>` 状态都是成功的。然后，我们返回一个新的 `Result<TResult, TError>`。如果任一 `IsSuccess` 属性为 `false`，则返回相应的错误。然而，一个类要被认为是应用函子，不仅仅需要一个 `Apply` 方法；它必须遵守应用函子定律。

当 Steve 正在掌握函子的概念时，Julia 提出了一个新挑战。

Julia：*现在，如果你想要一次性应用多个升级到塔上，但有些升级可能会失败，这时应用函子就派上用场了。*

Steve：*一次性应用多个升级？这真的可以简化我的升级系统！*

## 应用函子定律

有四个应用函子定律：恒等、同态、交换和组合。让我们逐一介绍它们。

### 恒等定律

恒等定律指出，将恒等函数应用于 `Result` 包装的值应该产生没有任何变化的原始 `Result`：

```cs
// Identity function
Func<int, int> identity = x => x;
// Result-wrapped value, representing, for example, a count of books
var bookCount = Result<int, string>.Success(10);
// Applying the identity function to the bookCount
var identityApplied = bookCount.Map(identity);
// The identity operation should not alter the original Result
Console.WriteLine(identityApplied.IsSuccess && identityApplied.Value == 10);  // Output: True
```

### 同态定律

这个定律表明，将一个函数应用于一个值然后包装它，与先包装值然后在该 `Result` 中应用函数是等价的：

```cs
Func<int, double> calculateRoyalties = sales => sales * 0.15;
int bookSales = 100;
// Applying function then wrapping
var directApplication = Result<double, string>.Success(calculateRoyalties(bookSales));
// Wrapping then applying function
var wrappedApplication = Result<int, string>.Success(bookSales).Map(calculateRoyalties);
// Both operations should yield the same result
Console.WriteLine(directApplication.IsSuccess && wrappedApplication.IsSuccess && directApplication.Value == wrappedApplication.Value);  // Output: True
```

### 交换定律

这个定律表明，将包装的函数应用于包装的值应该等同于应用一个将它的参数应用于包装值的函数。

对于这个定律，我们需要扩展我们的 `Result` 类型以支持将一个 `Result` 包装的函数应用于一个 `Result` 包装的值，这并不是由提供的 `Result` 类型结构直接支持的。然而，在一个支持这种功能的系统中，概念上的应用可能看起来像这样：

```cs
Result<Func<int, double>, string> wrappedCalculateRoyalties = new Result<Func<int, double>, string>(calculateRoyalties);
Result<int, string> salesResult = Result<int, string>.Success(bookSales);
// Wrapped function applied to wrapped value
var applied = salesResult.Apply(wrappedCalculateRoyalties);
// Equivalent to applying a function that takes a function and applies it to the value
Func<Func<int, double>, Result<double, string>> applyFuncToValue = func => Result<double, string>.Success(func(bookSales));
var interchangeResult = wrappedCalculateRoyalties.Map(applyFuncToValue);
// The results of applied and interchangeResult should be equivalent
```

### 组合定律

组合定律确保当我们组合两个或更多函数并将它们应用于一个函子时，函数应用的顺序不会影响结果。这个称为结合性的属性是组合定律的核心。它确保如果我们有函数 f、g 和 h，将它们组合并应用于函子 F 的结果，无论在组合过程中如何分组函数，都是相同的。

例如，考虑两个函数，f(x) = x + 1 和 g(x) = x * 3。根据组合定律，将 f 和 g 组合并应用于函子内部的值应该产生与将 f 应用到函子然后 g 相同的结果。用数学表达式来说，F.map(g(f(x))) 等价于 F.map(f).map(g)。

这种结合性质允许我们自信地推理组合函数，知道函数应用分组不会影响最终结果，当应用于函子时。这一原则增强了函数式编程的可预测性和可靠性，允许开发者简洁且安全地组合复杂的转换。

对于这个定律，类似于交换定律，我们需要一个机制在 `Result` 上下文中组合函数，而我们的当前 `Result` 类型定义并不直接支持。从概念上讲，它看起来像这样：

```cs
Func<int, double> calculateRoyalties = sales => sales * 0.15;
Func<double, double> adjustForMarket = royalties => royalties * 1.05;
// Composition of functions outside the Result context
Func<int, double> composed = sales => adjustForMarket(calculateRoyalties(sales));
// Applying composed function to a Result-wrapped value
var composedApplication = Result<int, string>.Success(bookSales).Map(composed);
// The result of composedApplication should be equivalent to applying each function within the Result context in sequence
```

正如你所见，我们的 `Result` 类型遵循所有这些定律，可以被视为一个应用函子。从这里，我们需要再迈出一步，使其成为一个单子。

# 单子

史蒂夫对函子很兴奋，但仍然难以看到它们在他游戏逻辑中的全部潜力。朱莉娅知道是时候引入单子了。

Julia：*让我们将你的游戏升级系统再向前推进一步。想象一系列操作：检查玩家的金币，扣除费用，并应用升级。每一步都依赖于前一步的成功。这正是单子的优势所在。*

史蒂夫：*这听起来正是我升级系统所需要的。单子是如何处理这个问题的？*

**单子**代表了我们对函子和应用函子概念探索的演变。虽然函子允许我们将函数映射到包装的值上，而应用函子使得我们可以将包装的函数应用于包装的值，但单子引入了以处理这些操作上下文的能力——无论是错误、列表、选项或其他计算上下文。因此，我们可以说单子是一个遵循某些额外定律的应用函子。

单子的本质在于其能够展开由返回包装值的函数应用所引起的包装层。这在执行多个操作时避免深层嵌套结构至关重要。让我们用一个来自我们出版系统的例子来分解这个概念。

想象一个场景，我们需要获取一份手稿，编辑它，然后格式化它以供出版。这些操作中的每一个都可能失败并返回 `Result<TValue, TError>`，导致嵌套的 `Result<Result<...>>` 类型。单子允许我们以更干净的方式按顺序执行这些操作。

## `Bind` 方法

单子的关键在于 `Bind` 方法（在不同的语言和框架中通常被称为 `flatMap` 或 `SelectMany`）。此方法将一个函数应用于包装的值，返回相同类型的包装器，然后展开结果：

```cs
public Result<TResult, TError> Bind<TResult>(Func<TValue, Result<TResult, TError>> func)
    {
        return IsSuccess ? func(_value!) : Result<TResult, TError>.Failure(_error!);
    }
```

使用 `Bind`，我们可以链式操作而不需要嵌套。让我们将其应用于我们的出版系统：

```cs
Result<Manuscript, string> FetchManuscript(int manuscriptId) { ... }
Result<EditedManuscript, string> EditManuscript(Manuscript manuscript) { ... }
Result<FormattedManuscript, string> FormatManuscript(EditedManuscript edited) { ... }
var manuscriptId = 101;
var publishingPipeline = FetchManuscript(manuscriptId)
    .Bind(EditManuscript)
    .Bind(FormatManuscript);
```

在这个管道中，只有前一个步骤成功时才会应用每个步骤，任何失败都会立即短路链。

## 单子法则

除了函子，单子也必须满足其法则以确保一致性和可预测性，它们有三个：左恒等、右恒等和结合律。

### 左恒等

包装一个值然后绑定到一个函数与直接应用该函数到值相同：

```cs
Func<int, Result<double, string>> calculateRoyalties = sales => new Result<double, string>(sales * 0.15);
int bookSales = 100;
var leftIdentity = Result<int, string>.Success(bookSales).Bind(calculateRoyalties);
var directApplication = calculateRoyalties(bookSales);
// leftIdentity should be equivalent to directApplication
```

### 右恒等

使用一个简单地重新包装值的函数绑定包装的值应该产生原始的包装值：

```cs
var manuscriptResult = Result<Manuscript, string>.Success(new Manuscript());
var rightIdentity = manuscriptResult.Bind(manuscript => Result<Manuscript, string>.Success(manuscript));
// rightIdentity should be equivalent to manuscriptResult
```

### 结合律

绑定操作的顺序不应影响：

```cs
var associativity1 = FetchManuscript(manuscriptId).Bind(EditManuscript).Bind(FormatManuscript);
var associativity2 = FetchManuscript(manuscriptId).Bind(manuscript => EditManuscript(manuscript).Bind(FormatManuscript));
// associativity1 should be equivalent to associativity2
```

## 利用单子

单子在涉及计算序列的操作中特别出色，其中每个步骤可能会失败或产生新的上下文。在我们的图书出版系统中，我们可以扩展这个模式来处理用户输入验证、数据库事务或网络调用，确保我们的代码保持清洁、可读和可维护：

```cs
Result<Publication, string> PublishManuscript(FormattedManuscript formatted) { ... }
var finalResult = FetchManuscript(manuscriptId)
    .Bind(EditManuscript)
    .Bind(FormatManuscript)
    .Bind(PublishManuscript);
```

这种方法不仅通过自动传播错误简化了错误处理，而且使快乐的路径代码保持清晰和直接，无需手动检查或嵌套条件。

单子就像智能容器，有助于管理一系列步骤，尤其是当你处理诸如错误或需要时间完成的任务等棘手情况时。通过掌握函子并提升到单子，你可以使你的代码不仅更具表现力，而且更容易维护和更可靠。想想我们如何在我们的图书出版示例中简化任务；单子和它们的伙伴们可以真正解开复杂的逻辑，使处理错误变得更加顺畅，使整个代码库更加友好，更容易工作。

# 关键要点

在我们总结关于函子和单子的这一章时，让我们花点时间总结一下我们学到了什么：

+   **函子的基本概念**：函子对于数据处理的功能性编程至关重要。它们充当“魔法盒子”，允许我们对它们所持有的数据进行函数应用，在保持原始结构的同时转换其内容。

+   **并非所有容器都是函子**：要使数据容器被视为函子，它必须遵守两个关键法则：恒等律和组合律。这些法则确保函子在其预期范式中操作时具有可预测性和一致性。

+   **恒等律**：恒等律强调，将恒等函数（返回其输入的函数）映射到函子上应该使函子保持不变。这条法则强调了函子转换的非侵入性。

+   **组合律**：组合律断言，函数组合和应用于函子的顺序不会影响最终结果。这条法则突出了函数在功能性编程中的可组合性和灵活性。

+   在`Result<TValue, TError>`类型的示例中，我们探讨了如何实际实现函子以增强错误处理和操作结果。`Map`方法展示了函数在函子封装的值中的应用，遵循函子定律。

+   **应用函子**：本章还介绍了应用函子的概念，它通过允许将包裹在函子中的函数应用到也包裹在函子中的值上，在基本函子之上构建。这种能力使得优雅地处理多层计算上下文成为可能。

+   **应用函子定律**：应用函子受额外的定律约束，包括恒等性、同构性、交换性和结合性。这些定律进一步确保了应用函子在复杂操作中的可靠和可预测的行为。

+   **单子**：讨论为函子和应用函子的概念演变到单子奠定了基础，单子通过允许操作链的连接来扩展函子和应用函子的概念，这些操作处理操作的上下文，如错误或异步计算。

最后，函子和单子不仅仅是理论结构；它们是实用的工具，可以将你的代码从普通转变为卓越。拥抱它们带来的机会，看看你的编程技能如何达到新的表达性和效率的高度。编码愉快！

# 练习

在解释了这些概念之后，Julia 挑战 Steve 应用他所学的知识。

Julia：*既然你已经了解了基础知识，为什么不尝试使用单子重构你的游戏升级系统呢？这应该会使你的代码更加健壮，并且更容易* *理解*。

Steve：*这是一个很好的想法！我已能看到这如何简化我的一些更复杂的* *游戏逻辑*。

## 练习 1

给定一个表示塔列表的`Result<List<Tower>, string>`类型，其中`Tower`是一个包含`Id`、`Name`和`Damage`等属性的类，任务是用函子概念将一个函数应用到每个塔上，在其名称末尾添加“(Upgraded)”以指示塔已被升级：

```cs
public class Tower
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Damage { get; set; }
}
public Result<List<Tower>, string> UpgradeTowers(List<Tower> towers)
{
    // Write your code here
}
```

## 练习 2

想象一下，你有两个被`Result`类型包裹的函数：`Result<Func<Tower, bool>, string> ValidateDamage`用于检查塔的伤害是否在可接受的范围内，而`Result<Func<Tower, bool>, string> ValidateName`用于检查塔的名称是否符合某些标准。给定一个表示单个塔的`Result<Tower, string>`，使用应用函子将这两个验证函数应用到塔上，确保两个验证都通过：

```cs
public Result<Func<Tower, bool>, string> ValidateDamage = new Result<Func<Tower, bool>, string>(tower => tower.Damage < 100);
public Result<Func<Tower, bool>, string> ValidateName = new Result<Func<Tower, bool>, string>(tower => tower.Name.Length > 5 && !tower.Name.Contains("BannedWord"));
```

## 练习 3

给定一系列用于升级塔的操作——`FetchTower`、`UpgradeTower`和`DeployTower`——每个操作都可能失败并返回`Result<Tower, string>`，使用单子概念将这些操作链在一起，针对给定的塔 ID。确保如果任何步骤失败，整个操作短路并返回错误：

```cs
public Result<Tower, string> FetchTower(int towerId) { /* Fetches tower based on ID */ }
public Result<Tower, string> UpgradeTower(Tower tower) { /* Upgrades the tower and can fail */ }
public Result<Tower, string> DeployTower(Tower tower) { /* Attempts to deploy the tower */ }
```

这些练习旨在帮助你们巩固本章学到的概念，并理解如何在不同的场景中应用它们。

# 解决方案

我希望你们都完成了所有的练习，只是想看看我的解决方案。如果没有，不要担心，一切都会随着经验而来。现在，让我们看看解决方案。

## 练习 1

使用 `Map` 方法将一个函数应用到列表中的每个塔上，该函数将其名字追加 “(Upgraded)”：

```cs
public Result<List<Tower>, string> UpgradeTowers(Result<List<Tower>, string> towersResult)
{
    return towersResult.Map(towers =>
        towers.Select(tower =>
        {
            tower.Name += " (Upgraded)";
            return tower;
        }).ToList());}
```

这个解决方案展示了函子如何封装数据和行为，允许在包含的数据上执行操作，同时保留整个操作（成功或失败）的上下文：

+   `towersResult.Map(...)`: 这一行使用了 `Map` 函数，这是函子模式的基本功能。如果 `Result` 成功，它将应用给定的函数到包含的值上，而不影响外部的 `Result` 结构。

+   `towers.Select(tower => ...)`: 在 `Map` 中，`Select` LINQ 方法遍历列表中的每个 `Tower` 对象，应用一个修改 `Name` 属性的 lambda 函数。

+   `tower.Name += " (Upgraded)"`：这个操作将 `"(Upgraded)"` 追加到每个塔的名字上，表示它已经被升级。

## 练习 2

应用函子模式通过允许函数本身被封装在上下文中（如 `Result`）来扩展函子的功能。这个解决方案使用这种能力将多个封装在 `Result` 类型中的验证函数应用到单个也封装在 `Result` 中的视频上：

```cs
public Result<bool, string> ValidateTower(Result<Tower, string> towerResult)
{
    var damageValidated = towerResult.Apply(ValidateDamage);
    var nameValidated = towerResult.Apply(ValidateName);
    return damageValidated.Bind(damageResult =>
        nameValidated.Map(nameResult => damageResult && nameResult));
}
```

在这里，我们以可组合的方式处理多个潜在失败点，展示了在函数式错误处理中应用函子的强大能力：

+   `towerResult.Apply(ValidateDamage)` 和 `towerResult.Apply` **(ValidateName)**：这些行展示了应用函子的 `Apply` 方法。它接受包含函数的 `Result` 并将其应用于另一个包含值的 `Result`，有效地展开两者并将函数应用于值。

+   `Bind` 和 `Map` 的链式调用确保了如果任何验证失败，整个验证过程将短路并返回失败。否则，它将两个验证的结果合并成一个布尔值。

## 练习 3

单子通过允许返回封装类型的操作链（如 `Result<Tower, string>`）来扩展应用函子的概念，实现操作的无缝和错误传播序列：

```cs
public Result<Tower, string> ProcessAndDeployTower(int towerId)
{
    return FetchTower(towerId)
        .Bind(UpgradeTower)
        .Bind(DeployTower);
}
```

这个解决方案展示了单子管理上下文中依赖操作序列的能力，使错误处理更加直观和线性，从而显著简化了复杂业务逻辑：

+   `FetchTower(towerId).Bind(UpgradeTower).Bind(DeployTower)`：这一行通过 `Bind` 方法展示了单子的本质。链中的每个函数（`FetchTower`、`UpgradeTower` 和 `DeployTower`）可能返回一个 `Result<Tower, string>`，而 `Bind` 确保只有前一个函数成功时，后续的函数才会执行。

+   这里的单调模式保证了如果在过程中任何一步失败，错误会立即通过链传播，绕过后续步骤并返回失败结果。

# 概述

在本章中，我们讨论了函子及其在函数式编程中的作用。我们从基础知识开始，解释说函子就像智能容器。它们可以持有数据，并且可以在保持其原始形状的同时对数据进行函数应用。

我们研究了函子的运作方式，展示了它们如何允许我们在不拆包和重新打包容器的情况下对它们持有的数据进行函数应用。我们还涵盖了函子遵循的两个主要规则：恒等性和组合律。

通过示例，我们看到了函子可以在不同情况下使用，例如处理列表或更平滑地处理错误。我们探讨了创建自己的函子，这为定制我们的代码以适应我们的需求提供了新的方法。我们从函子到应用函子和单子的完整旅程，了解了每个这些“容器”遵循的法则，并在一些实际场景中看到了它们的应用。

随着本章的结束，你应该已经对在项目中使用函子和单子有了坚实的基础。它们是简单而强大的工具，可以在你处理编码方式上产生重大影响。接下来，在第*第八章*中，*递归和尾调用*，我们将深入递归领域，了解函数如何调用自身，并探讨尾调用如何使递归更高效。

# 第三部分：实用函数式编程

在这部分中，我们从理论转向实践，探索如何将函数式编程概念应用于现实世界场景。我们将从递归和尾调用开始，学习如何编写高效的递归函数。然后，我们将探讨柯里化和部分应用，这些技术允许你创建更灵活和可重用的函数。最后，我们将探讨如何组合函数以创建强大的数据处理管道，将本书中学到的许多概念结合起来。

本部分包含以下章节：

+   *第八章**，递归和尾调用*

+   *第九章**，柯里化和部分应用*

+   *第十章**，管道和组合*
