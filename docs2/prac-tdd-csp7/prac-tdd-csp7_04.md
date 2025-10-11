# 开始之前需要了解的内容

你已经取得了相当不错的进展。到现在为止，你应该开始对测试驱动开发背后的基本概念感到舒适。你知道 TDD 的基本前提以及如何在 C#和 JavaScript 中编写单元测试。

在本章中：

+   我们将介绍更多 TDD 背后的实践

+   将提供具体的建议，以避免在过程中遇到陷阱

+   我们将解释定义和测试边界的重要性，以及抽象第三方代码（包括.NET 框架）

+   我们将开始介绍更高级的概念，例如间谍、模拟和伪造

首先，让我们了解一下在尝试测试现有应用程序时可能会遇到的一些问题。希望这能帮助你避免在下一个绿色田野应用程序中遇到问题。

# 无法测试的代码

有许多明显的迹象表明一个应用程序、类或方法将难以测试，甚至可能根本无法测试。当然，有一些方法可以绕过以下一些例子，但通常最好是避免这些解决方案和程序性杂技。简单通常是最好的，你的未来自己和/或未来的维护者会感谢你保持事物的简单性。

# 依赖注入

如果你是在构造函数或方法内部创建外部资源实例，而不是将它们传递进来，将非常难以编写测试来覆盖这些类和方法。通常，在当今的现代应用程序中，依赖注入框架用于创建和提供外部依赖项给一个类。许多人选择定义一个*接口*作为依赖项的契约，提供一种更灵活的测试方法，并减少对外部资源的耦合。

# 静态

你可能需要访问静态第三方类或方法。而不是直接访问静态资源，最好通过*接口*来访问这些资源。以 C#中的`DateTime`为例，`Now`是一个静态属性，这阻止了你控制被测试的类或方法使用的`DateTime`值。这使得验证测试用例和确保程序逻辑根据特定日期或时间正确行为变得更加困难。

# 单例

*单例*是共享状态的本质。为了确保你的测试在隔离环境中运行，最好避免使用它们。如果需要*单例*（例如，`Logging`、`Data Context`等），大多数依赖注入框架允许用非单例类替换单例实例，从而提供单例的功能和灵活性。对于生产代码，这允许你控制单例实例的作用域。

# 全局状态

很早就已经明白，应用程序中的全局状态会对系统造成破坏，并导致难以追踪的意外行为。在一个地方更改代码可能会对系统的其余部分产生深远的影响。为了可测试性，这通常意味着在设置上需要更多的努力，并且测试执行会变慢。

# 抽象第三方软件

随着你的应用程序增长，你可能会引入外部依赖。当然，这些系统、应用程序和库的开发者已经彻底测试了他们的产品。你应该专注于测试你的应用程序，而不是测试第三方代码。你的应用程序应该足够健壮，能够处理边缘情况，并且你需要考虑到预期和非预期的行为。你希望抽象掉第三方代码的细节，并测试预期的（以及意外的）结果。

那么，什么是**第三方代码**？任何你未编写的代码。这包括.NET 框架本身。实现第三方代码抽象的一种方法是通过使用测试替身。

# 测试替身

测试替身是函数和类，通过允许你验证功能或绕过难以测试的依赖项来帮助测试过程。测试替身被用于所有级别以隔离正在测试的代码。很多时候，对测试替身的需求推动了代码的架构。

C#中的`DateTime`对象就是一个这样的例子。`System.DateTime`是.NET 框架的一部分，通常你不会认为你需要在代码中抽象这部分。大多数开发者的本能是在一个*using 语句*中简单地引用它，然后在他们的代码中直接访问`DateTime.Now`。

一个无法重复的测试是一个糟糕的测试。

这通常很难测试。如果我们尝试使用`DateTime.Now`测试一个方法，我们将无法阻止`DateTime.Now`的默认功能。`DateTime.Now`返回存储在`DateTime`对象中的当前日期和时间。无法操纵此对象的返回值会导致我们的测试变得不可预测和不可重复。一个无法重复的测试是一个糟糕的测试。

许多开发者已经理解了可预测性的必要性。你可能听说过这样的话，“如果无法重现，就不是 bug”或类似的观点。这是因为，为了验证我们已修复了一个 bug，我们必须能够可预测地重复该错误。这样，一旦重现错误的步骤不再产生 bug，我们就可以自信地说 bug 已经被修复。至少对于那一系列步骤是这样的。

测试与修复 bug 没有不同；它遵循所有相同的步骤。我们只是确切知道是什么导致了 bug；代码尚未编写，或者我们刚刚尝试的重构失败了。

创建测试替身有时可能会变得有些复杂。因此，为了支持这些测试替身创建，几乎为每种拥有测试框架的语言都创建了框架。这些框架通常被称为模拟框架或库。在 C#中，目前最广泛使用的框架是*Moq*，发音为*mock*。同样，在 JavaScript 中，最常引用的模拟库似乎是*Sinon*，发音为*sign on*。

# 模拟框架

模拟框架是缓解大型项目中测试压力的绝佳工具。当尝试围绕遗留系统编写测试时，它们尤其有用。在这个例子中，遗留系统被定义为尚未围绕它编写测试的应用程序。这个定义来自 Michael Feather 的书籍《*与遗留代码有效工作*》。

在学习测试驱动开发和使用模拟框架时，请谨慎行事。模拟框架提供了一个非常有吸引力的替代方案，可以让你不必仔细考虑你的代码。你可能会编写一套完整的测试，最终却只真正测试了模拟框架。

许多模拟框架在这方面功能过于强大。在 C#中，存在一种模拟框架的分类，允许你替换外部代码。这些外部代码包括`DateTime.Now`和任何其他你无法控制的类。在 JavaScript 中，这被称为 monkey patching，每个框架都允许你这样做。

你问，这有什么害处吗？TDD（测试驱动开发）的一个好处是它鼓励做出明智的架构选择。当你有权力覆盖第三方代码的功能时，你就不再需要为了测试而进行抽象。

为什么那是个问题呢？如果我们想要保持代码的灵活性，并且想要遵循 SOLID 原则，那么对第三方代码的抽象是必要的。

# SOLID 原则

SOLID 原则是一组最初由 Robert C. Martin（即“Uncle Bob”）组合起来的概念。通常被宣传为*面向对象原则*，你应该把它们看作是纯粹的好的架构选择。SOLID 原则包括五个原则：单一职责原则、开闭原则、里氏替换原则、接口隔离原则和依赖倒置原则。

关于 SOLID 原则的原始文章可在[`butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod`](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)找到。

# 单一职责原则

在 Uncle Bob 的文章中，**单一职责原则（SRP**）被定义为：“一个类应该只有一个改变的理由。”

这意味着什么？这是困难的部分；有很多人理解其含义的方法。一种看法是，类应该只支持一个业务用户。另一种看法是，在应用程序内部，类应该只在有限的或特定的范围内使用。还有一种看法是，类应该具有有限的功能范围。这些都是正确的，但还不够充分。确保你遵守这一原则的一种方法是我们将要提到的“三到五规则”。

如果我们在讨论需求，例如，当一个需求有三个到五个验收标准时，那么它很可能在细节程度上是适当规模的。同样，如果我们正在讨论一个方法或函数，那么三到五行代码可能也是适当规模的。

三到五规则是一种通用的方式来了解你是否在遵守 SRP。该规则表述为：“少于三个是好的。三个到五个是不错的。超过五个则强烈考虑重构。”它并不像许多其他法律、原则和规则那样优雅，但三到五规则很容易遵循。这个规则只是一个指导方针，不应被视为最终命令。你应该尝试将这个规则应用到软件开发中的几乎所有事情上。你已经在本书中看到了它的应用。这个规则被用来确定第一章，“为什么 TDD 很重要”中需求的范围，以及到目前为止所包含的所有代码示例。

如果你使用三到五规则，几乎可以保证你正在遵循单一职责原则（SRP），并且它使你的代码、文件结构和需求保持小而易于维护。

# 开放/封闭原则

开放/封闭原则表述为：“软件实体（类、模块、函数等）应该对扩展开放，但对修改封闭。”SOLID 原则中的第二个听起来似乎并没有说很多，但它有着巨大的影响。

有许多方式来遵守这一原则。你可以或你的开发团队可以制定一个规则，只允许新的开发。也就是说，现有的功能不能被更新或改变，只能通过新的方法或类来替换。当我们到达创建代码分隔线的时候，你可以使用这些分隔线来创建一个进行功能交换的地方。

开放/封闭原则还使持续集成和部署成为可能。这是因为，如果你的应用程序从未违反与用户、自身或第三方之间的合同，那么它总是可以部署而不用担心引起生产问题。

# 李斯克夫替换原则

Liskov 替换原则可能由于其复杂和数学定义而难以理解。从 Barbara Liskov 的 *数据抽象和层次结构* [[`pdfs.semanticscholar.org/36be/babeb72287ad9490e1ebab84e7225ad6a9e5.pdf`](https://pdfs.semanticscholar.org/36be/babeb72287ad9490e1ebab84e7225ad6a9e5.pdf)] 中，该原则被表述如下：

*这里所需要的是类似以下替换属性。如果对于类型 S 的每个对象 o1，都存在一个类型 T 的对象 o2，使得对于所有以 T 定义的程序 P，当用 o1 替换 o2 时，P 的行为保持不变，那么 S 是 T 的子类型。*

Uncle Bob 将这个定义简化为，*使用基类指针或引用的函数必须能够在不知道的情况下使用派生类的对象。* 看这个原则，它似乎只是继承。但是，它不仅仅是继承。这个原则意味着替换其他对象的那个对象不仅必须实现与原始对象相同的接口或合同，还必须遵守与原始对象相同的期望。

违反此原则的经典例子是在矩形类的地方使用正方形类。一个典型的矩形类需要具有长度和宽度属性。在数学上，正方形只是矩形的一种特殊类型。因此，许多人会认为创建具有长度和宽度的正方形类可以接受地替换矩形类。

这里的问题是，正方形要求长度和宽度具有相同的值。因此，当你更改正方形类中的任何一个时，该类将更新另一个以具有相同的值。这是一个问题，因为使用该对象的应用程序并不期望这种行为。因此，应用程序必须意识到长度或宽度可能在没有通知的情况下更改的可能性。

无法满足应用程序的期望被称为拒绝遗赠。拒绝遗赠可能导致应用程序行为不一致，并且至少需要更多的代码来补偿不匹配。

# 接口隔离原则

接口隔离原则是关于保持你的类小而简洁的交互合同。更具体地说，你的类所呈现的合同应该只有一个责任。

有时，拥有一个具有小而单一责任合同的类可能很困难或不是所希望的。在这些情况下，类应该实现多个合同，而不是创建一个组合合同。我们希望多个合同可以减少远距离依赖的数量。

每次修改基类或接口时，子类也必须进行修改。至少，子类现在必须重新编译。通过限制合同的范围，我们可以减少更改该合同的影响，并改善整体系统架构。

# 依赖倒置原则

依赖项倒置之所以重要，有几个原因，其中之一是倒置的依赖项增加了灵活性，减少了易碎性，并有助于代码的潜在重用。

依赖项倒置允许插件式架构。通过定义交互合同，一个模块可以确定它想要如何与依赖项交互。然后依赖项依赖于该合同。

因为顶层模块没有外部依赖，它可以独立部署。独立部署应用程序的一部分几乎从未发生过，但拥有一个可以独立部署的库在依赖项更改时不需要重新编译具有巨大的好处。

在正常开发中，依赖项的波动比高级模块要多得多。这种波动导致需要重新编译。当你的应用程序依赖项向下流动时，依赖项的重新编译也会触发依赖库的重新编译。因此，实际上，改变一个微小但常见的库中的实用辅助类将触发整个应用程序的重新编译。

然而，如果你正在倒置依赖项，这样的变化只会触发实用辅助库和应用程序库的重新编译。它不会触发介于两者之间的每个库的重新编译。

这就是 SOLID 原则的全部内容。如果你选择使用模拟框架，请记住它们。确保你不允许模拟框架诱使你构建一个僵化、易碎、不可移动的系统。

# 及时的问候

在扩展经典的*Hello World*示例时，如果你想要根据一天中的时间改变你的问候语怎么办？以下是一个示例：

```cs
As a visitor to the site 
 I want to receive a time-appropriate greeting 
 So that I may plan the submission of my talks 

Given it is before noon 
 When greeting is requested 
 Then morning message is returned

Given it is afternoon 
 When greeting is requested 
 Then afternoon message is returned
```

你可能会想，这很简单；我只需编写一个快速的方法来返回适当的消息。当然，你会是对的。这是一个相当容易的任务。你可能会想出像这样的一些东西：

```cs
public string GetGreeting()
{
  if (DateTime.Now.Hour < 12)
    return "Good morning";

  return "Good afternoon";
}
```

记住，在第一章，“为什么 TDD 很重要”中，我们讨论了 TDD 的三个定律。至关重要的第一个定律指出，你不允许在没有失败的测试的情况下编写任何生产代码。

# 易碎的测试

“但是，这是一个如此简单的方法，”你可能会说。如果你遇到了一个错误怎么办？如果你想在事后为这个方法编写一些测试怎么办？你是否需要在一个特定的时间运行你的测试套件以确保测试通过？你是否需要根据你运行测试的时间改变你的测试？

# 假阳性和假阴性

如果我们像在*消息*示例中那样直接留下代码，并编写一个测试来覆盖该方法，它可能看起来像这样：

```cs
[Fact]
public void GivenEvening_ThenAfternoonMessage()
{
  // Arrange
  // Act
  var message = GetGreeting();

  // Assert
  Assert.Equal("Good afternoon", message);
}
```

你能发现这个测试的问题吗？测试本身并没有固有的错误。问题是生产代码将根据一天中的时间返回不同的消息。这意味着如果你在下午运行测试，它会通过。如果你在早上运行测试，它会失败。

# 抽象 DateTime

`DateTime` 是 .NET Framework 的一部分，因此，它应该从我们的系统中抽象出来。通常，我们希望我们的系统依赖于接口，这样我们就可以在运行时替换实现。

以下是一个 `ITimeManager` 的示例：

```cs
public interface ITimeManager
{
  DateTime Now { get; }
}
```

为了测试目的，你可能会得到一个看起来像这样的 `ITimeManager` 实现：

```cs
public class TestTimeManager : ITimeManager
{
  public Func<DateTime> CurrentTime = () => DateTime.Now;

  public void SetDateTime(DateTime now)
  {
    CurrentTime = () => now;
  }

  public DateTime Now => CurrentTime();
}
```

这允许我们设置 `Now` 的值，以便我们可以向测试方法提供已知值。现在，让我们回顾我们的测试：

```cs
[Theory]
[InlineData(12)]
[InlineData(13)]
[InlineData(14)]
[InlineData(15)]
[InlineData(16)]
[InlineData(17)]
[InlineData(18)]
[InlineData(19)]
[InlineData(20)]
[InlineData(21)]
[InlineData(22)]
[InlineData(23)]
public void GivenAfternoon_ThenAfternoonMessage(int hour)
{
  // Arrange
  var afternoonTime = new TestTimeManager();
  afternoonTime.SetDateTime(new DateTime(2017, 7, 13, hour, 0, 0));
  var messageUtility = new MessageUtility(afternoonTime);

  // Act
  var message = messageUtility.GetGreeting();

  // Assert
  Assert.Equal("Good afternoon", message);
}

[Theory]
[InlineData(0)]
[InlineData(1)]
[InlineData(2)]
[InlineData(3)]
[InlineData(4)]
[InlineData(5)]
[InlineData(6)]
[InlineData(7)]
[InlineData(8)]
[InlineData(9)]
[InlineData(10)]
[InlineData(11)]
public void GivenMorning_ThenMorningMessage(int hour)
{
  // Arrange
  var morningTime = new TestTimeManager();
  morningTime.SetDateTime(new DateTime(2017, 7, 13, hour, 0, 0));
  var messageUtility = new MessageUtility(morningTime);

  // Act
  var message = messageUtility.GetGreeting();

  // Assert
  Assert.Equal("Good morning", message);
}
```

我们的生产代码最终可能看起来像这样：

```cs
public class MessageUtility
{
  private readonly ITimeManager _timeManager;

  public MessageUtility(ITimeManager timeManager)
  {
    _timeManager = timeManager;
  }

  public string GetMessage()
  {
    if (_timeManager.Now.Hour < 12)
      return "Good morning";

    return "Good afternoon";
  }
}
```

# Test double 类型

测试替身有很多种类。这些种类通常可以分为 dummies、stubs、spies、mocks 和 fakes。接下来，我们将讨论不同类型，并为每种类型提供 C# 和 JavaScript 的示例。

# Dummies

Dummies 是测试替身中最简单的一种形式。Dummies 没有可感知的功能。我们实际上并不期望在测试的类或方法的结果中使用 `dummy` 类或方法。

Dummies 通常在测试的类具有依赖关系，而你正在测试的方法或函数没有使用该依赖关系时使用。

你可以通过创建一个类或方法的新副本或实例来创建一个 dummies，然后在代码体中做任何事情。Void 方法将是空的，而期望返回值的函数或方法在调用时将抛出异常或返回该返回值的简单形式。

# Dummy logger

*Logging* 服务是一个完美的例子，可以用 dummies 替换。当你正在测试特定方法时，不太可能（也不推荐）同时测试日志功能。

# C# 示例

以下是一个 C# 中的 `DummyLogger` 示例。你会注意到当调用 `Log` 时，没有任何操作发生。

```cs
enum LogLevel
{
  None = 0,
  Error = 1,
  Warning = 2,
  Success = 3,
  Info = 4
}

interface ILogger
{
  void Log(LogLevel type, string message);
}

class DummyLogger: ILogger
{ 
  public void Log(LogLevel type, string message)
  {
    // Do Nothing
  }       
}
```

# JavaScript 示例

以下是一个 JavaScript 中的 `DummyLogger` 示例。你会注意到当调用 `info`、`warn`、`error` 和 `success` 时，没有任何操作发生。

```cs
export class DummyLogger {
   info(message) {
   }

   warn(message) {
   }

   error(message) {
   }

   success(message) {
   }
 }
```

# Stubs

Stubs 是比 Dummies 高一级的测试替身。Stub 测试替身将提供与传入的参数无关的相同响应。

当你想要测试代码中的不同执行路径时，会使用 Stubs。一个例子是在特定条件下必须抛出的错误。

Stubs 是通过创建需要返回 stub 值的类或方法的副本或覆盖，然后将其设置为返回所需值来创建的。记住，stubs 不评估参数，所以你只需要返回所需的值。

# C# 示例

以下是一个 C#中的`StubSpeakerContactServiceError`的示例。你会注意到，当调用`MessageSpeaker`时，会抛出一个新的`UnableToContactSpeakerException`错误。

```cs
class StubSpeakerContactServiceError : ISpeakerContactService
{
  public void MessageSpeaker(string message) 
  {
    throw new UnableToContactSpeakerException();
  } 
}
```

# JavaScript 示例

以下是一个 JavaScript 中`stubSpeakerReducer`的示例。你会注意到，无论传递什么操作，都会在状态中的错误数组中推送一个新的`*UNABLE_TO_RETRIEVE_SPEAKERS*`错误。

```cs
 import { SpeakerErrors } from './errors';
 import { SpeakerFilters } from './actions';

 const initialState = {
   speakerFilter: SpeakerActions.SHOW_ALL,
   speakers: [],
   errors: []
 };

 export function *stubSpeakerReducer*(state, action) {
   state = state || initialState;

   state.speakerFilter = action.filter || SpeakerFilters.SHOW_ALL;
   state.errors.push(SpeakerErrors.UNABLE_TO_RETRIEVE_SPEAKERS);

   return state;
 }
```

# 间谍（Spies）

间谍（Spies）是测试替身（test doubles）的下一进化阶段。间谍返回一个类似于存根（stub）的值，但它有一个极其重要且有用的区别。间谍可以报告与函数调用相关的信息。

当你想要验证一个函数是否以特定参数被调用时，间谍（Spies）通常被使用。这在你的应用程序的第三方边界处非常有用。例如，了解你的应用程序是否正确配置了数据库连接，使用了某些配置服务提供的凭据，这一点很重要。在某些情况下，测量被测试的方法或函数的副作用可能很困难。在这些情况下，你可以使用间谍来确保你首先调用了该方法或函数。

间谍（Spies）是通过从存根（stub）开始，并添加确定函数是否被调用、函数被调用的次数，或者报告传递给该函数的值的函数来创建的。

# C#示例

以下是一个 C#中的`SpySpeakerContactService`的示例。`SpySpeakerContactService`允许你确定服务是否被调用，以及它可能被调用多少次。

```cs
class SpySpeakerContactService : ISpeakerContactService
{
  public bool MessageSpeakerHasBeenCalled { get; private set; }

  public int MessageSpeakerCallCount { get; private set; }

  public void MessageSpeaker(string message)
  {
    MessageSpeakerHasBeenCalled = true;
    MessageSpeakerCallCount++;
  }
}
```

# JavaScript 示例

以下是一个 JavaScript 中`spySpeakerReducer`的示例。`spySpeakerReducer`允许你确定它可能被调用多少次。

```cs
import { speakerReducer as original_speakerReducer } from './reducers';

 export let callCounter = 0;

 export function *spySpeakerReducer*(state, action) {
   callCounter++;

   return original_speakerReducer(state,action);
 } 
```

# 模拟（Mocks）

模拟（Mocks）基本上是可编程的间谍。当你想要在多个测试中使用相同的测试替身时，模拟（Mocks）非常有用。模拟能够返回你设置的任何值。需要注意的是，模拟仍然没有执行任何逻辑。它们返回指定的值，并不检查传递给函数的参数。

模拟（Mocks）用于所有使用存根（dummies）、存根（stubs）和间谍（spies）的情况。模拟是测试替身的一个更重的实现，这也是为什么你可能不想总是使用它们的原因。与之前的测试替身相比，模拟的重用性较低，因为模拟的数据必须为每个测试设置，而存根、存根或间谍有一个固定的返回值，不需要配置。设置返回的测试数据通常比创建整个存根或间谍类更困难。

模拟（Mocks）是通过复制一个类或方法并创建一个可以设置为方法返回值的属性来创建的；然后，在正在模拟的方法中，返回该属性值。一旦创建，在每次测试之前，必须设置模拟的返回值。

# C#示例

以下是一个 C#中的`MockDateTimeService`示例。`MockDateTimeService`允许您设置服务返回的`DateTime`，以便可靠地测试系统其他部分基于特定`DateTime`可能的行为。

```cs
class MockDateTimeService
{
  public DateTime CurrentDateTime { get; set; } = new DateTime();

  public DateTime UTCNow()
  {
    return CurrentDateTime.ToUniversalTime();
  }
}
```

# JavaScript 示例

以下是一个 JavaScript 中的`MockDateTimeService`示例。与 C#中的`MockDateTimeService`类似，这允许您设置服务返回的`DateTime`，以便可靠地测试系统其他部分基于特定`DateTimes`可能的行为。

```cs
export class MockDateTimeService {
   constructor() {
     this.currentDateTime = new Date(2000, 0, 1);
   }

   now() {
     return this.currentDateTime;
   }
 }
```

# 模拟

模拟是最后也是最强大的测试双胞胎类型。模拟是一个试图表现得像它不是一个测试双胞胎的类。虽然模拟不会连接到数据库，但它会尝试表现得就像它正在连接到数据库一样。模拟不会使用系统时钟，但它会尝试拥有一个尽可能接近系统时钟的内部时钟。

模拟要么添加额外的测试功能，要么防止第三方库和系统的外部干扰。大多数应用程序都连接到某些数据源。可以创建一个使用其自己的内存数据源的模拟仓库，但除此之外，它的行为就像一个正常的数据连接。

模拟是通过生成一个全新的类或方法，然后编写足够的功能以使其与生产类或方法不可区分来创建的。对于模拟与生产类或方法之间的唯一重要区别是，模拟不进行外部连接，并且可能允许测试人员控制底层数据集。

# C#示例

以下是一个`FakeRepository`及其相关*接口*的示例。`FakeRepository`是一个泛型仓库的模拟实现。

```cs
public interface IRepository<T>
{
  T Get(Func<T, bool> predicate);
  IQueryable<T> GetAll();
  T Save(T item);
  IRepository<T> Include(Expression<Func<T, object>> path);
}

public interface IIdentity
{
  int Id {get;set;}
}

public class FakeRepository<T> : IRepository<T> where T : IIdentity
{
  private int _identityCounter = 0;
  public IList<T> DataSet { get; set; } = new List<T>();

  public T Get(Func<T, bool> predicate)
  {
    return GetAll().Where(predicate).FirstOrDefault();
  }

  public IQueryable<T> GetAll()
  {
    return DataSet.AsQueryable();
  }

  public T Save(T item)
  {
    return item.Id == default(int) ? Create(item) : Update(item);
  }           

  public IRepository<T> Include(Expression<Func<T, object>> path)
  {
    // Nothing to do here since this function is for EntityFramework
    // We are using Linq to Objects so there is not need for Include
    return this;
  }

  private T Create(T item)
  { 
    item.Id = ++_identityCounter;
    DataSet.Add(item);
    return item;
  }

  private T Update(T item)
  {
    var found = Get(x => x.Id == item.Id);

    if(found == null)
    {
      throw new KeyNotFoundException($"Item with Id {item.Id} not    
        found!");
    }

    DataSet.Remove(found);
    DataSet.Add(item);

    return item;
  }
}
```

# JavaScript 示例

以下是一个 JavaScript 中的`FakeDataContext`示例。

```cs
export class FakeDataContext {
   _identityCounter = 1;
   _dataSet = [];

   get DataSet() {
     return this._dataSet;
   }

   set DataSet(value) {
     this._dataSet = value;
   }

   get(predicate) {
     if (typeof(predicate) !== 'function') {
       throw new Error('Predicate must be a function');
     }

     const resultSet = this_dataSet.filter(predicate);

     return resultSet.length >= 1 ? {...resultSet[0]} : null;
   }

   getAll() {
     return this._dataSet.map((x) => {
       return {...x};
     });
   }

   save(item) {
     return item.id ? this.update(item) : this.create(item);
   }

   update(item) {
     if (!this._dataSet.some(x => x.id === item.id)) {
       this._dataSet.push({...item});
     } else {
       let itemIndex = this._dataSet.findIndex(x => x.id === item.id);
       this._dataSet[itemIndex] = {...item};
     }

     return {...item};
   }

   create(item) {
     let newItem = {...item};
     newItem.id = this._identityCounter++;
     this._dataSet.push({...newItem});

     return {...newItem};
   }
 } 
```

# N 层示例

现在，将您的注意力转回到第二章中的 API 控制器，“设置.NET 测试环境”。直接从控制器返回硬编码的数据并不能为构建应用程序提供一个坚实的基础。大多数现代的任何规模的.NET 应用程序都是用某种 N 层架构编写的。您会希望将您的业务逻辑与您的表示层分离，在这个例子中，就是 API 端点的表示层。

我们将引入一个演讲服务的*接口*，为使用依赖注入向控制器提供具体实现做准备，然后验证新服务中的正确方法是否被调用。您可能需要重新排列一些测试，以从控制器中移除业务逻辑。

# 表示层

要开始，添加一个新的测试来验证控制器是否接受`ISpeakerService`的*接口*：

```cs
[Fact]
public void ItAcceptsInterface()
{
  // Arrange 
  ISpeakerService testSpeakerService = new TestSpeakerService();

  // Act
  var controller = new SpeakerController(testSpeakerService);

  // Assert
  Assert.NotNull(controller);
}
```

现在，通过在`SpeakerController`中创建一个接受`ISpeakerService`接口的构造函数，引入一个字段变量和构造函数到你的`speaker controller`类中，来使测试通过：

```cs
public SpeakerController(ISpeakerService speakerService)
{
}
```

你的测试项目现在应该无法编译。这是因为在我们之前的例子中，从第二章，*设置.NET 测试环境*，我们在测试类的构造函数中定义了控制器实例。修改构造函数以创建一个实现`ISpeakerService`接口的`TestSpeakerService`实例，并将其传递给`SpeakerController`。你可以在测试类中自由创建`TestSpeakerService`：

```cs
public SpeakerControllerSearchTests()
{
  var testSpeakerService = new TestSpeakerService();

  _controller = new SpeakerController(testSpeakerService);
}
```

现在，你将想要验证`SpeakerService`的`Search`方法是否被控制器调用。但是，如何做到这一点呢？一种方法是通过一个名为*Moq*的模拟框架。

# Moq

要将*Moq*添加到你的单元测试项目中，右键单击你的测试项目，选择管理 NuGet 包。搜索*Moq*，并选择安装最新稳定版本。我们不会深入探讨*Moq*，但我们会展示模拟框架如何帮助促进对应用程序边界的测试。

添加一个测试来验证`SpeakerService`的`Search`方法是否从控制器的`Search`动作结果中调用一次：

```cs
[Fact]
public void ItCallsSearchServiceOnce()
{ 
  // Arrange
  // Act
  _controller.Search("jos");

  // Assert
  _speakerServiceMock.Verify(mock => mock.Search(It.IsAny<string>()), 
      Times.Once());
}
```

为了使测试通过，你还需要在`test`类的构造函数中进行一些额外的设置：

```cs
private readonly SpeakerController _controller;
private static Mock<ISpeakerService> _speakerServiceMock;

public SpeakerControllerSearchTests()
{
  var speaker = new Speaker
  {
    Name = "test"
  };

  // define the mock
  _speakerServiceMock = new Mock<ISpeakerService>();

  // when search is called, return list of speakers containing speaker
  _speakerServiceMock.Setup(x => x.Search(It.IsAny<string>()))
      .Returns(() => new List<Speaker> { speaker });

  // pass mock object as ISpeakerService
  _controller = new SpeakerController(_speakerServiceMock.Object);
}
```

确保修改*接口*，以便应用程序可以编译：

```cs
public interface ISpeakerService
{
  IEnumerable<Speaker> Search(string searchString);
}
```

现在，通过确保从控制器的`Search`动作结果中调用`SpeakerService`的`Search`方法来使测试通过。如果你还没有这样做，创建一个名为`_speakerService`的`field`变量，并在构造函数中通过`speakerService`参数进行赋值：

```cs
private readonly ISpeakerService _speakerService;

public SpeakerController(ISpeakerService speakerService)
{
  _speakerService = speakerService;
}

[Route("search")]
public IActionResult Search(string searchString)
{
  var hardCodedSpeakers = new List<Speaker>
  {
    new Speaker{Name = "Josh"},
    new Speaker{Name = "Joshua"},
    new Speaker{Name = "Joseph"},
    new Speaker{Name = "Bill"},
  };

  _speakerService.Search("foo");

  var speakers = hardCodedSpeakers.Where(x =>   
                       x.Name.StartsWith(searchString,  
                       StringComparison.OrdinalIgnoreCase)).ToList();

  return Ok(speakers);
}
```

接下来，添加一个测试来验证控制器`Search`动作结果提供的`searchString`与传递给`SpeakerService`的`Search`方法的`searchString`是否相同：

```cs
[Fact]
public void GivenSearchStringThenSpeakerServiceSearchCalledWithString(){
  // Arrange
  var searchString = "jos";

  // Act
  _controller.Search(searchString);

  // Assert
  _speakerServiceMock.Verify(mock => mock.Search(searchString),   
      Times.Once());
}
```

通过向`_speakerService`的`Search`方法提供`searchString`来使测试通过：

```cs
  _speakerService.Search(searchString);
```

现在，确保`SpeakerService`的`Search`方法的结果是动作结果返回的内容：

```cs
[Fact]
public void GivenSpeakerServiceThenResultsReturned()
{
  // Arrange
  var searchString = "jos";

  // Act 
  var result = _controller.Search(searchString) as OkObjectResult;

  // Assert
  Assert.NotNull(result);
  var speakers = ((IEnumerable<Speaker>)result.Value).ToList();
  Assert.Equal(_speakers, speakers);
}
```

记住，`SpeakerService`的`Search`方法返回的结果是由`Mock`定义的。你需要提取一个`field`来测试返回给*动作结果*的结果与为我们的`Mock`定义的结果是否相同：

```cs
private readonly SpeakerController _controller;
private static Mock<ISpeakerService> _speakerServiceMock;
private readonly List<Speaker> _speakers;

public SpeakerControllerSearchTests()
{
  _speakers = new List<Speaker> { new Speaker
  {
    Name = "test"
  } };

  _speakerServiceMock = new Mock<ISpeakerService>();
  _speakerServiceMock.Setup(x => x.Search(It.IsAny<string>()))
                .Returns(() => _speakers);

  _controller = new SpeakerController(_speakerServiceMock.Object);
}
```

仍然存在硬编码数据的问题。在使测试通过的同时，别忘了删除不必要的代码。记住*红、绿、重构*。这同样适用于你的生产代码以及测试代码。

在你删除硬编码数据后，可能会遇到一些失败的测试。目前，跳过这些测试，因为我们将会将这个逻辑移动到应用程序的另一个部分。现在，是时候创建一个`SpeakerService`了：

```cs
xUnit
 [Fact(Skip="Reason for skipping")]
MSTest
 [Skip]
```

# 业务层

你可能需要开始考虑如何有效地组织你的测试。随着你的应用程序的增长，测试文件的数量增加，你可能会发现导航你的解决方案变得越来越繁琐。一个可能的解决方案是为每个待测试的类创建单独的文件夹，并在类文件夹中为每个公共方法创建一个单独的文件。这可能会看起来像这样：

```cs
SpeakerService -> Search
```

你现在不一定需要处理这个问题，但为未来制定一个计划不会有害。应用程序往往会迅速增长，在你意识到之前，你可能会在你的解决方案中拥有十三个项目。你现在可以选择创建一个`Services`项目，并在此同时创建一个`ServicesTest`项目，以将业务层及其相关测试与表示层及其测试分开。这将被留给读者作为练习。

现在，为`SpeakerService`创建一个新的测试类。这里是你将在`SpeakerService`中创建所有`Search`测试方法的地方：

```cs
[Fact]
public void ItExists()
{
  var speakerService = new SpeakerService();
}
```

一旦这个测试通过，创建一些新的测试来确认`Search`方法存在并且返回一个演讲者集合：

```cs
[Fact]
public void ItHasSearchMethod()
{
  var speakerService = new SpeakerService();

  speakerService.Search("test");
}
```

接下来，测试`SpeakerService`是否实现了`ISpeakerService`接口：

```cs
[Fact]
public void ItImplementsISpeakerService()
{
  var speakerService = new SpeakerService();

  Assert.True(speakerService is ISpeakerService);
}
```

你的`SpeakerService`现在可能看起来像这样：

```cs
public class SpeakerService : ISpeakerService
{
  public IEnumerable<Speaker> Search(string searchString)
  {
    return new List<Speaker>();
  }
}
```

记住，要慢而有序地进行。你不允许在不编写失败的测试的情况下编写任何生产代码，而且你不应该编写比使测试通过所必需的更多的生产代码。

现在，开始将`s*kipped*`测试从`controller test`文件移动到`Speaker Service Search Test`文件。从`GivenExactMatchThenOneSpeakerInCollection`开始：

```cs
[Fact]
public void GivenExactMatchThenOneSpeakerInCollection()
{
  // Arrange
  // Act
  var result = _speakerService.Search("Joshua");

  // Assert
  var speakers = result.ToList();
  Assert.Equal(1, speakers.Count);
  Assert.Equal("Joshua", speakers[0].Name);
}
```

让这个测试通过，然后继续到`GivenCaseInsensitveMatchThenSpeakerInCollection`：

```cs
[Theory]
[InlineData("Joshua")]
[InlineData("joshua")]
[InlineData("JoShUa")]
public void GivenCaseInsensitveMatchThenSpeakerInCollection(string searchString)
{
  // Arrange
  // Act
  var result = _speakerService.Search(searchString);

  // Assert
  var speakers = result.ToList();
  Assert.Equal(1, speakers.Count);
  Assert.Equal("Joshua", speakers[0].Name);
}
```

最后，`GivenNoMatchThenEmptyCollection`和`Given3MatchThenCollectionWith3Speakers`：

```cs
[Fact]
public void GivenNoMatchThenEmptyCollection()
{
  // Arrange
  // Act
  var result = _speakerService.Search("ZZZ");

  // Assert
  var speakers = result.ToList();
  Assert.Equal(0, speakers.Count);
}

[Fact]
public void Given3MatchThenCollectionWith3Speakers()
{
  // Arrange
  // Act
  var result = _speakerService.Search("jos");

  // Assert
  var speakers = result.ToList();
  Assert.Equal(3, speakers.Count);
  Assert.True(speakers.Any(s => s.Name == "Josh"));
  Assert.True(speakers.Any(s => s.Name == "Joshua"));
  Assert.True(speakers.Any(s => s.Name == "Joseph"));
}
```

随着你对这个实践越来越熟悉，并且对 TDD 有更多的经验，你可能发现列出你想要实现的所有测试是有帮助的。这可以简单地是在一张纸上写下它们，或者在你的 IDE 中为跳过的或忽略的测试创建一些存根。

如果做得正确，你的代码应该看起来像这样：

```cs
public class SpeakerService : ISpeakerService
{
  public IEnumerable<Speaker> Search(string searchString)
  {
    var hardCodedSpeakers = new List<Speaker>
    {
      new Speaker{Name = "Josh"},
      new Speaker{Name = "Joshua"},
      new Speaker{Name = "Joseph"},
      new Speaker{Name = "Bill"},
    };

    var speakers = hardCodedSpeakers.Where(x => 
                        x.Name.StartsWith(searchString, 
                        StringComparison.OrdinalIgnoreCase)).ToList();

    return speakers;
  }
}
```

我们现在已经将硬编码的数据从我们的控制器移到了`SpeakerService`的业务层。你可能认为我们付出了很多努力，仅仅是为了将问题移到一个新的文件！虽然这在一定程度上是正确的，但这实际上使我们处于更好的位置，以便于未来的开发。所谓的“逻辑”已经被移动到一个可以被应用程序的其他部分以及潜在的新接口（例如原生和/或移动应用程序）重用的类中，而这些接口无法访问我们的原始控制器。

我们将在未来的章节中继续这个例子。我们最终将摆脱硬编码的数据，并使用 Entity 框架实现数据访问层。所有这些都可以通过测试驱动开发来完成。

# 摘要

在本章中，我们讨论了一些会阻碍 TDD 的陷阱，例如对第三方库的依赖、直接实例化类以及脆弱的测试。我们还讨论了避免或解决这些问题的方法。我们介绍了并讨论了每个 SOLID 原则。我们还讨论了不同类型的测试替身以及何时使用每种类型。最后，我们给出了一个 N 层应用的简短示例以及如何对其进行测试。

在第五章，“Tabula Rasa – Approaching an Application with TDD in Mind”，我们将探讨如何带着 TDD（测试驱动开发）的思路去接近一个应用程序，将理论转化为实践，以及如何通过测试更好地发展应用程序。
