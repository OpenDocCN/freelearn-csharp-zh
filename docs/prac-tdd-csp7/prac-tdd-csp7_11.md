# 第十一章：需求变更

在任何应用程序的进展过程中，可能会添加新的和不同的需求。有时这些需求增强了应用程序的现有功能。在其他时候，这些新需求可能与现有功能冲突。当需求冲突时，重要的是要解决这些问题，以便构建适当的功能。

那么，你可能会看到哪些需求的变化？变化通常包括对业务规则的修改、新功能或增强，或者对系统中发现的错误或缺陷所需的修改。

随着时间的推移，通常需要修改现有的业务规则。这可能是对用户反馈的反应、来自业务的澄清，或者通过系统使用过程中发现的需求。当发现需要变化时，现有的应用程序将需要做出改变。一个全面的测试套件将确保在实施新更改后，系统的其余部分仍然按预期运行。首先，通过修改和/或创建新的测试来覆盖系统的新期望功能。

在软件开发中有一个常见的说法，*软件永远不会完成；它只是被放弃*。也就是说，如果一个应用程序要继续有用，它将通过新的开发继续增长和演变。如果没有添加新功能，那么很可能项目已经被放弃。如果一个应用程序要继续有用，那么你可以预期需要实现新功能。再次强调，从测试开始，添加新的测试来帮助指导任何和所有新功能的实现。

当发现错误并确定根本原因时，需要做出更改以解决问题。为了防止这个错误在未来再次出现，应该编写一个新的测试或一系列测试来覆盖任何可能导致错误行为的潜在场景。

在本章中，我们将了解：

+   需求变更

+   新功能

+   处理缺陷

+   Speaker Meet 的变更

+   过早优化

# Hello World

回顾我们最早的例子之一，看看这个 *Hello World* 示例应用程序。记住，根据一天中的时间，用户会看到不同的消息。在中午之前，用户会被问候为早上好，中午之后，用户会收到下午好的问候。

# 需求变更

根据一天中的时间，用户会被问候为早上好或下午好。为了扩展功能并引入新功能，如果一天中的时间是晚上 6 点到午夜，让我们用晚上好来称呼用户。

# 晚上好

为了引入这个新功能，从测试开始。需要修改现有的测试，以及添加一个或多个新的测试来覆盖需求的变化。

修改提供给`GivenAfternoon_ThenAfternoonMessage`的`Theory`数据，以便只包括中午到下午 6 点之间的测试。现在，创建一个新的测试方法，`GivenEvening_ThenEveningMessage`：

```cs
[Theory]
[InlineData(19)]
[InlineData(20)]
[InlineData(21)]
[InlineData(22)]
[InlineData(23)]
public void GivenEvening_ThenEveningMessage(int hour)
{
  // Arrange  
  var eveningTime = new TestTimeManager();
  eveningTime.SetDateTime(new DateTime(2017, 7, 13, hour, 0, 0));
  var messageUtility = new MessageUtility(eveningTime);

  // Act
  var message = messageUtility.GetMessage();

  // Assert
  Assert.Equal("Good evening", message);
}
```

现在通过修改现有代码来使`Theory`通过：

```cs
public string GetMessage()
{
  if (_timeManager.Now.Hour < 12)
    return "Good morning";
  if (_timeManager.Now.Hour <= 18)
    return "Good afternoon";
  return "Good evening";
}
```

这确实是一个相当简单的例子。实现开始增长一个你可能满意或不满意的设计。你可以自由地尝试替代实现。你现在应该有足够的测试，让你感到安全地重构到一个你更喜欢的模式。如果你破坏了实现或发现了你引入的错误，为这个场景添加一个测试。

# FizzBuzz

接下来，从第二章的 FizzBuzz 示例，*设置.NET 测试环境*，扩展这个代码 kata 的经典行为，并引入一些新行为。

# 新特性

已经向经典的 FizzBuzz kata 添加了一个新要求。新的要求是，当一个数字不能被 3 或 5 整除，且大于 1 时，应返回“未找到数字”的消息。这应该足够简单。再次从测试开始，并做出必要的修改。

# 未找到数字

要开始，需要一个新的测试方法来验证是否返回了“未找到数字”的消息：

```cs
[Fact]
public void GivenNonDivisibleGreaterThan1ThenNumberNotFound()
{
  // Arrange
  // Act
  var result = FizzBuzz(2);
  // Assert
  Assert.Equal("Number not found", result);
}
```

现在，通过修改现有代码来使测试通过：

```cs
private object FizzBuzz(int value)
{
  if (value % 15 == 0)
    return "FizzBuzz";
  if (value % 5 == 0)
    return "Buzz";
  if (value % 3 == 0)
    return "Fizz";
  if (value == 2)
    return "Number not found";
}
```

这涵盖了第一个实例。然而，这满足新的要求吗？创建一个`Theory`集来强制正确的解决方案：

```cs
[Theory]
[InlineData(2)]
[InlineData(4)]
[InlineData(7)]
[InlineData(8)]
public void GivenNonDivisibleGreaterThan1ThenNumberNotFound(int number)
{
  // Arrange
  // Act
  var result = FizzBuzz(number);
  // Assert
  Assert.Equal("Number not found", result);
}
```

正确地使测试通过。修改现有代码，以便返回所需的结果：

```cs
private object FizzBuzz(int value)
{
  if (value % 15 == 0)
    return "FizzBuzz";
  if (value % 5 == 0)
    return "Buzz";
  if (value % 3 == 0)
    return "Fizz";
  return value == 1 ? (object)value : "Number not found";
}
```

注意，在整个练习过程中，所有现有的测试都应该继续通过。如果你发现了一个错误，写一个新的测试来验证这个场景，并相应地纠正代码。

# TODO 应用

*TODO*应用是我们早期的 TDD 示例之一。这个应用远未完成，并且我们已经从业务那里收到了新的要求，要求向应用中添加一个功能。

现在业务希望有完成 TODO 列表中任务的能力。这个特性是“当前迭代计划”的，是我们接下来要工作的下一个故事。

# 标记完成

对于“标记完成”的故事，我们被要求允许用户完成 TODO 列表中的任何任务。添加这个功能应该与这本书中的任何其他 TDD 练习类似。在阅读我们对这个问题的解决方案之前，试着独立完成这个任务。在你通过测试后，再回来查看这本书中的解决方案。

# 添加测试

在`ToDoApplicationTests`文件中，我们添加了一个“剃羊毛”测试来强迫我们创建完成方法。这个测试也有助于定义方法的 API：

```cs
[Fact(Skip = "Yak shaving - no longer needed")]
public void CompleteTodoExists()
{
  // Arrange
  var todo = new TodoList();
  var item = new Todo();

  todo.AddTodo(item);

  // Act
  todo.Complete(item);
}
```

这导致我们在`TodoList`类中创建了一个方法存根。为了使这个测试通过，我们必须从生成的方法中移除未实现异常。创建方法后，我们添加了一个跳过到这个测试，类似于同一文件中之前的“剃羊毛”测试。

接下来，我们需要创建一个`TodoListCompleteTests`文件来存放完成方法的函数测试：

```cs
public class TodoListCompleteTests
{
  [Fact]
  public void ItRemovesAnItemFromTheList()
  {
    // Arrange
    var todo = new TodoList();
    var item = new Todo();

    todo.AddTodo(item);
    // Act
    todo.Complete(item);
    // Assert
    Assert.Equal(0, todo.Items.Count());
  }
}
```

在编写第一个测试并实现代码使其通过之后，我们很难再编写另一个失败的测试。因此，我们假设现在我们已经完成了。

# 生产代码

完成任务的测试代码相当简单，只需要一个单行方法：

```cs
public void Complete(Todo item)
{
  _items.Remove(item);
}
```

这就是我们需要的所有内容。我们现在已经准备好进行冲刺演示。

# 但不要从列表中删除！

在冲刺演示期间，我们的产品负责人询问任务完成后发生了什么。我们解释说，它被从这份列表中移除了。这并不好。产品负责人希望我们能提供关于未来任务的指标。她希望我们跟踪任务的完成情况，而不是删除它。

在与其他开发者讨论之后，我们决定任务将获得一个完成属性，并从列表中隐藏。为了实现这一点，我们不得不进行一些重构并添加新的测试。再次提醒，尝试自己完成这个练习，然后再看看我们的解决方案进行比较。

# 添加测试

这个更改需要相当多的新测试。然而，在我们能够创建新测试之前，我们必须首先将现有的完成测试重命名，以表示正确的功能。在`TodoListCompleteTests`文件中添加两个额外的测试，我们验证了项目被标记为完成，并且它没有被从 TODO 列表中删除：

```cs
public class TodoListCompleteTests
{
  [Fact]
  public void ItHidesAnItemFromTheList()
  {
    // Arrange
    var todo = new TodoList();
    var item = new Todo { Description = "Test Todo" };

    todo.AddTodo(item);

    // Act
    todo.Complete(item);

    // Assert
    Assert.Equal(0, todo.Items.Count());
  }

  [Fact]
  public void ItMarksAnItemComplete()
  {
    // Arrange
    var todo = new TodoList();
    var item = new Todo { Description = "Test Todo" };

    todo.AddTodo(item);

    // Act
    todo.Complete(item);

    // Assert
    Assert.True(item.IsComplete);
  }

  [Fact]
  public void ItShowsCompletedItems()
  {
    // Arrange
    var todo = new TodoList();
    var item = new Todo { Description = "Test Todo" };

    todo.ShowCompleted = true;
    todo.AddTodo(item);

    // Act
    todo.Complete(item);

    // Assert
    Assert.Equal(1, todo.Items.Count());
  }
}
```

为了添加`ShowComplete`，我们在`ToDoApplicationTests`文件中创建了一个用于完整性的“剃毛”测试：

```cs
[Fact(Skip = "Yak shaving - no longer needed")]
public void ShowCompletedExists()
{
  // Arrange
  var todo = new TodoList();

  // Act
  todo.ShowCompleted = true;
}
```

我们还必须在`TodoModelTests`文件中添加一个类似的测试：

```cs
[Fact]
public void ItHasIsComplete()
{
  // Arrange
  var todo = new Todo();

  // Act
  todo.IsComplete = true;
}
```

# 生产代码

对于如此小的代码库，新测试所需的变化相当显著。首先，我们在`Todo`模型中添加了一个`IsComplete`属性：

```cs
internal class Todo
{
  public bool IsComplete { get; set; }
  public string Description { get; set; }

  internal void Validate()
  {
    Description = Description ?? throw new DescriptionRequiredException();
  }
}
```

其余的更改影响`TodoList`类。添加了一个布尔属性来切换已完成项的可见性，修改了`Complete`方法，使其仅标记项目为完成，并在从列表检索的项目中添加了一个`where`子句：

```cs
internal class TodoList
{
  private readonly List<Todo> _items = new List<Todo>();

  public IEnumerable<Todo> Items => _items.Where(t => !t.IsComplete || ShowCompleted);

  public bool ShowCompleted { get; set; }

  public void AddTodo(Todo item)
  {
    item = item ?? throw new ArgumentNullException();
    item.Validate();
    _items.Add(item);
  }

  public void Complete(Todo item)
  {
    item.IsComplete = true;            
  }        
}
```

# Speaker Meet 的更改

在任何应用程序中，变化都是不可避免的。由于新的业务规则、功能增强、缺陷的发现和修复等原因，需求会发生变化。当进行测试驱动开发时，变化尤其确定。幸运的是，通过 TDD 的过程，你的应用程序应该可以轻松且安全地进行修改。

如果一个系统是松散耦合的，那么理论上，对系统某一部分的更改应该对系统的其余部分影响很小或没有影响。一套全面的单元测试应该减轻对做出更改的恐惧。

不幸的是，这些测试只对它们定义的场景有效。如果未能编写足够的测试来覆盖某些场景或边缘情况，那么确实有可能出现错误进入生产环境。如果不采用 TDD 方法，或者更糟糕的是，根本不编写测试，那么你可能会发现错误很容易通过你的代码审查过程和 CI/CD 构建管道的所有检查。

查看 Speaker Meet 应用的新要求。

# 后端更改

随着 Speaker Meet 应用的进展，引入了新的要求。演讲者必须在可见于系统部分之前被**批准**。这包括演讲者的完整列表、返回演讲者详细信息以及通过搜索结果。

在这种情况下，一位开发者来帮忙实施。这位开发者不熟悉 TDD，并且没有编写测试来验证他的工作。新要求得到了实施，并提交了代码审查：

```cs
public Models.SpeakerDetail Get(int id)
{
  var speaker = _repository.Get(id);

  if (speaker == null || speaker.IsDeleted || speaker.IsActive)
  {
    throw new SpeakerNotFoundException(id);
  }

  var gravatar = _gravatarService.GetGravatar(speaker.EmailAddress);

  return new Models.SpeakerDetail
  {
    Id = speaker.Id,
    Name = speaker.Name,
    Location = speaker.Location,
    Gravatar = gravatar
  };
}
```

并且添加了对类的更改：

```cs
public class Speaker
{
  public int Id { get; set; }

  [Required]
  [StringLength(50)]
  public string Name { get; set; }

  [Required]
  [StringLength(50)]
  public string Location { get; set; }

  [Required]
  [StringLength(255)]
  public string EmailAddress { get; set; }

  public bool IsDeleted { get; set; }

  public bool IsActive { get; set; }
}
```

你能发现问题吗？

代码被审查，并留下了评论。然而，评论被误解了（或者只是简单地被忽略了），代码被提交、合并并通过了部署流程。当然是一个故障，但这种情况有时会发生。

CI 服务器运行了测试套件。现有的测试通过了。由于没有现有的场景会捕捉到这个错误，所以错误没有被发现。由于没有创建新的测试，所以没有测试失败。CD 过程运行，代码进入了生产环境。

那么可以添加什么测试来确保正确代码的实现？在处理错误时，通常最好的做法是简单地编写验证错误行为的测试。在这种情况下，我们希望抛出一个错误。所以下面的测试应该断言抛出了正确的错误：

```cs
[Fact]
public void GivenSpeakerIsNotActiveThenSpeakerNotFoundException()
{
  // Arrange
  var expectedSpeaker = SpeakerFactory.Create(_fakeRepository);
  expectedSpeaker.IsActive = false;
  var service = new SpeakerService(_fakeRepository, _fakeGravatarService);
  // Act
  var exception = Record.Exception(() => service.Get(expectedSpeaker.Id));
  // Assert
  Assert.IsAssignableFrom<SpeakerNotFoundException>(exception);
}
```

通过修改服务来使这个新测试通过：

```cs
public Models.SpeakerDetail Get(int id)
{
  var speaker = _repository.Get(id);

  if (speaker == null || speaker.IsDeleted || !speaker.IsActive)
  {
    throw new SpeakerNotFoundException(id);
  }

  var gravatar = _gravatarService.GetGravatar(speaker.EmailAddress);

  return new Models.SpeakerDetail
  {
    Id = speaker.Id,
    Name = speaker.Name,
    Location = speaker.Location,
    Gravatar = gravatar
  };
}
```

然而，随着这个变化，现在将有一系列现有的测试会失败。这是因为`IsActive`属性的默认值是`false`。

为了快速让这些测试通过，你可以做些类似的事情：

```cs
public bool IsActive { get; set; } = true;
```

这可能会引入意外的结果，所以务必创建一些保护测试来验证正确性。

这解释了为什么这个错误最初没有被捕捉到。`IsActive`属性被添加到数据库中，默认值为`true`。直到新演讲者被添加到数据库中，`IsActive`列的值为`false`时，错误才被发现。一旦发现了不正确的行为，缺陷就很容易被识别和修复。

# 前端更改

从概念或方法的角度来看，前端更改没有区别。你需要编写适当的测试来确保应用程序期望的行为，然后编写生产代码以使测试通过。

尽管如此，让我们以一个快速示例为例，为我们在前端代码中添加一个新功能。

# 按客户端评分排序

我们将要添加的功能是按评分对演讲者进行排序。在之前的章节中，评分并未讨论过，甚至没有强制执行，因此需要对迄今为止构建的模型进行修改，以包含评分。当然，如果你还没有按照 C#代码定义完成整个模型，那就需要这样做。

就像本章前面的例子一样，尝试自己添加这种行为，然后看看我们的后续解决方案。

在`speakerReducer.spec.js`文件中，我们为默认按排名排序演讲者添加了一个单一测试。该测试可以添加到演讲者还原器的 describe 块中：

```cs
it('sorts speakers by rank', () => {
   // Arrange
   const initialState = [];
   const speaker1 = { id: 'test-speaker-1', firstName: 'Test 1', lastName: 'Speaker', rank: 1};
   const speaker2 = { id: 'test-speaker-2', firstName: 'Test 2', lastName: 'Speaker', rank: 2};
   const action = actions.getSpeakersSuccess([speaker1, speaker2]);

   // Act
   const newState = speakersReducer(initialState, action);

   // Assert
   expect(newState).to.have.lengthOf(2);
   expect(newState[0]).to.deep.equal(speaker2);
 });
```

使这个测试通过的相关代码位于`speakerReducer.js`文件中：

```cs
export function speakersReducer(state = [], action) {
   switch(action.type) {
     case types.GET_SPEAKERS_SUCCESS:
       return action.speakers.sort((a, b) => {
         return b.rank > a.rank;
       });
     default:
       return state;
   }
 }
```

# 那么，接下来该做什么呢？

今后，实施任何必要的变更都应该很容易。这可能包括新功能、需求变更或发现的缺陷。这并不是说应用程序已经完成或没有错误，但你应该对应用程序以现有测试套件所考虑的方式运行有一定的信心。

# 过早优化

为了澄清，我们将优化定义为任何使代码变得模糊、使其更难以理解或限制了测试要求之外的可能性的事情。过早优化是指出于任何非要求原因进行的优化。

通常，优化工作是以性能为由进行的。在进行这些代码修改之前，应该存在一个明确说明需要变更的要求。

即使是通过测试驱动开发（Test-Driven Development）的实践，也有可能把自己逼入死角。在重构或设计下一个测试的过程中，有时可能会一次性解决太多问题，或者重构得过多。

总要记住，在 TDD 中，我们希望将问题分解成尽可能小的步骤。此外，如果解决方案超过一行或两行，不要在第一个测试中就追求解决方案。同时，即使是小解决方案，如果解决方案计算或算法密集，也应该分解，即使最终的解决方案只是一行生产代码。

警惕过早优化。

根据 Kent Beck 的说法，重构是去除重复的过程。记住，在重构测试时。仅通过去除重复，我们可以通过重构避免过早优化。有时重构解决方案并显著减少代码量，或者甚至使用一些花哨的新语言特性或 Linq 表达式来使测试通过，这是完全可能的，甚至在某些情况下是有吸引力的。但这些解决方案从长远来看是可行的，但在测试仍在构建的过程中，这些隐藏的优化可能会让你和你的测试迅速偏离轨道。

# 摘要

你现在可以看到，需求的变更、新的功能请求或缺陷可能需要应用程序进行更改。通过测试驱动开发（TDD）和一套全面的单元测试，这些更改可以安全且容易地进行。

在第十二章，“遗留问题”，我们将讨论如何处理可能没有考虑到测试的遗留应用程序。
