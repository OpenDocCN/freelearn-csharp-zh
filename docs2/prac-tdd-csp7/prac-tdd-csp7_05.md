# 第五章：Tabula Rasa – 以 TDD 的心态处理应用程序

开发一个完整的应用程序似乎是一项艰巨的任务，特别是使用**测试驱动开发**（**TDD**）。到目前为止，所有的例子都相对较小。函数和方法具有微小、有限的范围。TDD 在开发完整的应用程序时是如何转换的？实际上，相当不错。

本章讨论的主题包括：

+   Yak shaving

+   早期大设计

+   YAGNI

+   测试要小

+   恶魔的辩护者

# 从哪里开始

最好的开始方式就是从开始的地方开始。在开发者开始编码之前，他们必须知道程序的目标是什么。应用程序的目的是什么？如果没有对试图解决的问题有清晰的理解，那么开始可能会很困难。至少，在没有某种计划的情况下开始是不明智的。

你开始编码越早，程序就会越晚完成。

– 罗伊·卡尔森，威斯康星大学

你有没有在没有目标的情况下开始一个手工艺项目？你是如何知道你要做什么的？项目结果如何？如果结果不错，你很可能在某个时候选择了一个方向，并开始实现目标。你可能甚至不得不从头开始或途中进行调整，以便完成项目。

现在，想象一下在事先定义好期望结果的情况下开始同一个手工艺项目。也许你想要画一幅画。也许你制定了一套计划。直到你开始实际开始项目的物理行动之前，你都不会有清晰的理解。在这个例子中，成功的可能性要大得多。在过程中遇到障碍的机会最小化。

这是否意味着所有问题都需要提前提出？在开始之前是否应该获得所有答案？当然不是。有时，对问题只有肤浅的了解就足以开始。但是，目标越明确，开发正确解决方案的可能性就越大。

# Yak shaving

在前几章提供的示例中，你可能已经注意到有很多代码的移动，这些代码似乎没有立即的益处。在 TDD 中，尤其是在项目的初期，有些工作看起来似乎没有太多意义。编写的测试除了证明类或方法的存在之外，没有做更多的事情。代码重构的方式只是将硬编码的值推入另一个依赖项。这意味着会创建更多的文件，你可能会发现自己正在编写大量的辅助类。所有这些活动都被称为“剃羊毛”。

“剃羊毛”有两个与软件开发相关的含义。第一个也是需要避免的是，为了拖延而编写不需要的东西。第二个是指完成所有必须做的事情来准备代码的行为。两者之间的区别是细微的。你站在哪一边取决于你编写代码的意图。你是避免编写应该写的代码，还是使用 TDD 来为高效和有效的软件开发打下基础？

在我们的案例中，正如前面章节所讨论的，我们要么在为未来的测试打下基础，要么在测试中实施防止写作障碍的已知技术。有时，为测试准备应用程序的过程可能需要相当长的时间。

当在遗留应用程序中工作时，你可能需要花费一周的大部分时间来创建工厂、向现有类添加接口、编写测试替身或进行安全的重构技术。所有这些活动都有助于提高可测试性和确保测试体验的顺畅。然而，避免沉迷于这些活动是很重要的。我们只想通过这些活动来推动下一个测试的进行。

# 大规模设计

以前，拥有一个漫长且昂贵的**软件开发生命周期**（**SDLC**）是一种常见的做法。组建了大型团队。会议被安排，讨论无休止地进行。收集需求并创建文档，这些文档消耗了大量的纸张，足以填满每个团队成员的档案柜。系统的设计通常是通过民主方式组装的，并为系统制定了一个计划。

一旦管理层和/或执行团队满意，开发就可以开始了。这个漫长而繁琐的过程通常意味着预算已经因为规划阶段的每个人时间成本而大幅减少。如果在开发周期中发现设计中的缺陷，通常会发出变更订单和一系列会议。

如果由于市场变化或其他外部条件的变化而导致需求发生变化，这可能会使整个项目偏离轨道。如果软件开发生命周期（SDLC）不允许快速适应变化和快速调整方向，那么这通常会给整个项目带来灾难。更糟糕的是，如果变化足够大，它可能会使应用程序的需求变得过时。

不幸的是，以这种方式开发软件成本很高，而且失败的可能性比成功要大。变化的成本太高，由此产生的干扰往往对过程有害。如今，软件项目更有可能以某种敏捷的方式进行开发。

# 一张干净的画布

那么，我们如何使用 TDD 开始一个新的应用？带着 TDD 的想法开始实际上与开始任何软件开发项目没有太大区别。开发者必须对应用的目标有一个大致的了解。基本需求应该被理解。就像我们通过测试来扩展应用一样，需求应该随着时间的推移而增长。

# 一口一口吃

你怎么吃大象？一口一口吃。

尝试一次性定义和开发一个单体应用是一项庞大的任务。如果你被分配去创建 Facebook，你可能不知道从哪里开始。但是，如果你将应用分解为逻辑部分，例如*登录*、*用户仪表板*和*新闻源*，它就会变得更容易管理。

# 最小可行产品

每个工作定义都应该分解成小的可交付成果。**最小可行产品**的概念可以应用到我们代码的各个方面。随着单体应用的需求被分解成可管理的块，可能开始编码。如果一个编程任务只需要几个小时就能完成，那么很难交付一个完全偏离目标的东西。然而，如果需要变更，应该提供反馈，并且可以快速进行调整。

# 不同的思维方式

当应用以 TDD 为导向进行开发时，你应该对小的可交付成果采取同样的方法。写一点测试，写足够的代码使其通过，然后重构。如果你一直在运行测试套件，或者更好的是，你正在使用像 NCrunch 这样的持续测试运行器，你的反馈周期确实应该非常快。

永远不要留下未关注的测试，或者一次不要忽略超过一个测试。

如果在开发周期中测试开始失败，应该很容易恢复。刚刚编写的代码就是问题所在。暂停当前的工作并评估。这个变更是否必要？失败的测试是否需要更改？如果需要，可以*跳过*（xUnit）或*忽略*（MSTest）当前的测试。修复代码，然后通过取消忽略测试来继续。永远不要留下未关注的测试，或者一次不要忽略超过一个测试。这样做只会增加测试（或者更糟，多个测试）永远不会完成、修复或恢复的风险。一个被忽略的测试没有任何价值。如果测试在以后被你或其他人取消忽略，并且现在（或仍然）失败，可能很难确定测试是否有效并指示真正的失败，或者无效并可能让你走上歧途。确保你的测试是有效、准确并提供价值的。

# YAGNI - 你不需要它

有时，你可能会被迫编写一些代码，因为你认为你需要它。这只是一个简单的方法。如果你有一张满载数据的数据表，你可能需要一个`GetAll`方法和一个`GetById`方法。在这里提醒一下：在没有真正需求之前，不要编写任何代码。编写的代码越多，需要维护的代码就越多。如果你编写了你认为可能需要但从未实际使用的代码，你就浪费了努力。更糟糕的是，你引入了必须维护直到或除非它被删除的代码。

不要为了未来的需求而编写代码。这是浪费的，并且通常开发和维护成本高昂。

# 测试小规模

在进行 TDD 时，要考虑的最重要的事情之一是测试的大小和范围。TDD 是一项完全理解你试图解决的问题的练习，并且能够尽可能地将解决方案分解成尽可能多的微小部分。

作为例子，让我们考虑一些简单的事情：一个管理需要完成的项目列表的应用程序。我们如何将这个应用程序的使用案例分解？

首先，使用我们讨论的剃羊毛方法，我们可以验证应用程序是否存在。

```cs
public class ToDoApplicationTests
{
  [Fact] 
  public void TodoListExists()
  {
    var todo = new TodoList();

    Assert.NotNull(todo);
  }
}

internal class TodoList
{
  public TodoList()
  {
  }
}
```

接下来，验证你是否能够检索待办事项的列表。

```cs
[Fact]
public void CanGetTodos()
{
  // Arrange
  var todo = new TodoList();

  // Act
  var result = todo.Items;

  // Assert
  Assert.NotNull(result);
}
```

# 魔鬼的代言人

我们将继续展示小规模的测试，但已经触及了下一个例子。在许多情况下，充当魔鬼的代言人是一种有用的技巧。在 TDD 中，我们通过想象使测试通过的最简单、可能也是最错误的方法来充当魔鬼的代言人。我们希望迫使测试使代码正确，而不是编写我们认为正确的代码。例如，在这种情况下，我们的目标是使刚刚编写的测试通过添加一个*Items*列表。但在这个阶段，测试并不需要这一点。它只要求 Items 作为类的一个属性存在。测试中没有指定类型。因此，为了充当魔鬼的代言人，使用*Object*作为类型，并将`Items`对象设置为简单的非空值。

```cs
internal class TodoList
{
  public object Items { get; } = new object();

  public TodoList()
  {
  }
}
```

好的，现在所有测试都通过了，但这显然不是一个合适的解决方案。从小处着手，我们可以迫使实现有一个计数，这肯定需要它是一个*Todos*列表的集合。在最后一个测试中添加以下内容：

```cs
Assert.Empty();
```

为了使它通过，`Items`必须改变：

```cs
public IEnumerable<Object> Items { get; } = new List<Object>();
```

记住我们在第四章“开始之前要知道什么”中讨论的 SOLID 原则。我们希望使用接口隔离，并限制自己只使用所需的接口。我们不需要完整的`IList`接口功能，因此不需要使用它。所需的是能够遍历项目集合的能力。为此，最简单的接口是`IEnumerable`。

然而，我们仍然有一个问题：我们正在使用`Object`作为我们的可枚举类型。我们只想使用一个特定的类。现在让我们解决这个问题。修改最后一个测试，包括一个类型断言。

```cs
[Fact]
public void CanGetTodos()
{
  // Arrange
  var todo = new TodoList();

  // Act
  var result = todo.Items;

  // Assert
  Assert.NotNull(result);
  Assert.IsAssignableFrom<IEnumerable<Todo>>(result);
  Assert.Empty();
}
```

现在，更新类，如下所示：

```cs
internal class TodoList
{
  public IEnumerable<Todo> Items { get; } = new List<Todo>();

  public TodoList()
  {
  }
}

public class Todo 
{
}
```

正如你所见，我们添加了一个看似相当小的测试，最终创建了一个属性，分配了一个默认值，并创建了一个类。你能想到任何方法可以使这个更小吗？

我们接下来的测试可能将验证待办事项项开始时是空的，但如果我们回顾 TDD（测试驱动开发）的法则，第一条法则是编写一个失败的测试。目前，如果我们编写一个验证`Items`为空的测试，我们会期望这个测试通过。那么，我们应该编写什么样的测试呢？

我们决定编写下一个测试，以验证添加待办事项项的方法。

```cs
[Fact]
public void AddTodoExists()
{
  // Arrange
  var todo = new TodoList();
  var item = new Todo();

  // Act
  todo.AddTodo(item);
}

internal class TodoList
{
  public IEnumerable<Todo> Items { get; } = new List<Todo>();

  public TodoList()
  {
  }

  internal void AddTodo(Todo item)
  {
  }
}
```

到目前为止，我们一直在采取一些可能与你正常开发中采取的步骤相似的步骤，将大量功能切割到代码中。这是第一个在我们真正实现有价值功能之前就停止的测试。这是采取那些小步骤的一部分。我们目前可以部署这个应用。它不会很有用，但我们确实有这个选项。如果我们已经到达了冲刺的终点，产品负责人可能会要求为了尽快部署，我们硬编码一些待办事项，以便在 UI 中提供一些内容。

我们接下来的测试看起来相当直接。我们将验证我们是否真的可以使用我们的新方法添加一个待办事项。但是有一个问题，因为这个测试是在测试功能而不是通用类结构。我们建议为这个方法创建一个专门的测试类。

```cs
public class TodoListAddTests
{
  [Fact]
  public void ItAddsATodoItemToTheTodoList()
  {
    // Arrange
    var todo = new TodoList();
    var item = new Todo();

    // Act
    todo.AddTodo(item);

    // Assert
    Assert.Single(todo.Items);
  }
}

internal class TodoList
{
  private List<Todo> _items = new List<Todo>();

  public IEnumerable<Todo> Items
  {
    get
    {
      return _items;
    }
  }

  public TodoList()
  {
  }

  public void AddTodo(Todo item)
  {
    _items.Add(item);
  }
}
```

现在，这真是一个从悬崖上飞扑下来的跳跃。那个测试几乎改变了我们应用的所有代码。我们完全改变了`Items`的实现，并在`AddTodo`方法中添加了代码。我们能否将这些分成两个或更多步骤？我们还有许多事情要做，我们将会覆盖其中的一些。但在我们继续之前，写下你认为你会编写的下一个几个测试。尽量别跳过这个练习，因为将功能分解成这样的小块是大多数开发者学习 TDD 时遇到的最大难题之一。

我们将暂时暂停这个示例应用的进度，因为我们已经陷入了一个困境。为了防止受阻，我们应该首先测试负案例。

# 首先测试负案例

首先要测试负案例意味着什么？在许多电脑游戏中，尤其是在角色扮演游戏中，游戏设计师通常会设计得非常困难，如果你直接去挑战 Boss，就很难赢得游戏。相反，你必须完成支线任务，走错路，在故事中迷失方向，才能与 Boss 战斗。测试也是如此。在问题得到解决之前，我们必须首先处理不良输入，防止异常，并解决业务需求中的冲突。

在 Todo 应用程序中，我们错误地快速通过了添加一个项目到 Todo 列表，而没有验证该项目是否有效。现在，冲刺已经结束，我们的用户界面开发者对我们很生气，因为他们不知道如何处理一个没有任何详细信息的 Todo 项目。我们应该做的首先是处理我们收到的不良数据的情况。让我们回放并暂时跳过我们刚刚制作的测试。

```cs
[Fact(Skip="Forgot to test negative cases first")]
public void ItAddsATodoItemToTheTodoList()
```

我们现在需要编写的测试应该放在刚刚忽略的测试之上，但仍在同一个文件中。记住我们需要有小的测试增量，我们可以编写一个测试来防止最简单的坏数据，即`null`。

```cs
[Fact]
public void OnNullAnArgumentNullErrorOccurs()
{
  // Arrange
  var todo = new TodoList();
  Todo item = null;

  // Act
  var exception = Record.Exception(() => todo.AddTodo(item));

  // Assert
  Assert.NotNull(exception);
  Assert.IsType<ArgumentNullException>(exception);
}

public void AddTodo(Todo item)
{
  throw new ArgumentNullException();
}
```

注意，我们已经移除了`AddTodo`中现有的代码。我们本可以保留这些代码，但在这个阶段，它们只是杂乱无章，而且目前没有测试强制要求这些代码存在。有时，当你忽略一个测试时，删除该测试所验证的功能性代码比绕过它更容易。有时，杂乱无章可能会限制你的重构努力，并导致更糟糕的代码。不要害怕删除被跳过的测试代码，也不要害怕删除进入源代码控制中的被跳过的测试。

在进行这个更改时，我们遇到的一个其他问题是，之前在`TodoApplicationTests`类中定义的`AddTodoExists`方法现在失败了。这个测试最初是一个剃须测试，并没有给测试套件增加任何真正的价值，所以直接移除它。

既然我们已经用我们的方法覆盖了空值情况，接下来可能出错的是什么？思考一下，Todo 是否需要必填字段？在我们将其添加到列表之前，我们可能需要确保 Todo 至少有一个标题或描述。

首先，在我们能够验证字段已被填充之前，我们需要验证该字段存在于模型上。编写模型测试可能看起来有点过度，但我们发现这些测试有助于更好地定义应用程序，供其他人进入。它们还提供了一个良好的附加点，用于稍后当业务决定 Todo 的描述字段最大长度为 255 个字符时的字段验证测试。让我们为 Todo 模型测试创建一个新的类。

```cs
public class TodoModelTests
{
  [Fact] 
  public void ItHasDescription()
  {
    // Arrange
    var todo = new Todo();

    // Act
    todo.Description = "Test Description";
  }
}

public class Todo
{
  public string Description { get; set; }
}
```

正如你所见，这类测试没有真正的断言。只需验证我们能否设置描述值而不抛出错误就足够了。

现在我们有了描述字段，我们可以验证它是必需的。

```cs
[Fact]
public void OnNullADescriptionRequiredValidationErrorOccurs()
{
  // Arrange
  var todo = new TodoList();
  var item = new Todo()
  {
    Description = null
  };

  // Act
  var exception = Record.Exception(() => todo.AddTodo(item));

  // Assert
  Assert.NotNull(exception);
  Assert.IsType(typeof(DescriptionRequiredException), exception); 
}

internal class TodoList
{
  ...

  public void AddTodo(Todo item)
  {
    item = item ?? throw new ArgumentNullException();

    item.Description = item.Description ?? throw new                   
                                           DescriptionRequiredException();
  }
}
```

我们已经很久没有重构了，这是一个暂停我们的测试工作并进行重构的好地方。我们希望将模型验证移动到模型中。让我们为待办模型上的验证方法创建一个快速测试，然后将该逻辑移动到`Todo`类中。

```cs
public class TodoModelValidateTests
{
  [Fact]
  public void ItExists()
  {
    // Arrange
    var todo = new Todo();

    // Act
    todo.Validate();
  }
}

public class Todo
{
  public string Description { get; set; }

  internal void Validate()
  {
  }
}
```

现在，至少目前，我们想把我们的验证逻辑从待办事项列表移动到模型中。在创建验证测试和移动逻辑的过程中，我们导致我们的剃羊毛测试失败。测试失败是因为，尽管存在必需的方法，但它抛出了异常，因为我们没有填充我们的待办事项的描述。我们将不得不移除这个测试，因为它不再增加价值。

```cs
public class TodoModelValidateTests
{
  [Fact]
  public void OnNullADescriptionRequiredValidationErrorOccurs()
  {
    // Arrange
    var item = new Todo()
    {
      Description = null
    };

    // Act
    var exception = Record.Exception(() => item.Validate());

    // Assert
    Assert.NotNull(exception);
    Assert.IsType(typeof(DescriptionRequiredException), exception); 
  }
}

public class Todo
{
  public string Description { get; set; }

  internal void Validate()
  {
    Description = Description ?? throw new DescriptionRequiredException();
  }
}
```

最后，我们在进行我们想要的重构更改之前需要编写的测试已经完成。现在我们可以简单地用对模型上的`Validate`的调用替换`TodoList`类中处理模型验证的异常逻辑。

```cs
public void AddTodo(Todo item)
{
  item = item ?? throw new ArgumentNullException();

  item.Validate();
}
```

这个更改对我们的测试或我们的结果逻辑不应有任何影响。我们只是在重新定位验证代码。可能还有更多的验证可以发生。你能想到一些可能很有价值的验证吗？

现在是时候添加回我们跳过的测试，并进行一些小的修改以通过验证。

```cs
[Fact]
public void ItAddsATodoItemToTheTodoList()
{
  // Arrange
  var todo = new TodoList();
  var item = new Todo
  {
    Description = "Test Description"
  };

  // Act
  todo.AddTodo(item);

  // Assert
  Assert.Single(todo.Items);
}

public void AddTodo(Todo item)
{
  item = item ?? throw new ArgumentNullException();

  item.Validate();

  _items.Add(item);
}
```

# 当测试变得痛苦时

有可能你可能会遇到一些痛苦。也许你因为你的设计而把自己逼到了角落。也许你不确定下一个最有趣的测试会是什么。当然，你并不是故意的，但可能你在测试之间跳得太大了。无论情况如何，可能有一段时间，测试会变得痛苦。

# 一个尖峰

如果你发现自己陷入了困境，或者在你如何继续前进的选项之间犹豫不决，运行一个尖峰可能会有所帮助。尖峰是一种你可以用来调查想法的手段。给自己设定一个时间限制或其他限制性指标。一旦通过练习获得了足够的知识或洞察力，就丢弃结果。尖峰的目的不是带走可工作的代码。目标应该是获得理解，并为前进的道路提供一个更好的想法。

# 首先断言

有时，你可能知道你想要编写的下一个测试，但并不完全确定如何开始。如果发生这种情况，从*断言*开始，以确定预期的结果。一旦定义了期望，就着手使实际值与期望值相匹配。你可能更愿意经常采取这种方法，以确保你只编写足够的代码来使所需的测试通过。

# 保持组织

记住，测试是应用程序的第一个消费者。你可以提供的最好和最准确的文档是一套全面且维护良好的测试。在你的测试套件中，创建文件夹、嵌套类或利用测试框架的功能来使你的测试更易于阅读。记住，如果你在以后遇到测试失败，一个描述性的测试名称和适当的断言将有助于描述结果是如何产生的。

使用`Describe`来更好地组织你的 JavaScript 测试。通过在测试中使用多个`Describe`来嵌套多个层级。

# 解构 Speaker Meet

Speaker Meet 应用始于一个简单的目标：连接技术演讲者、社区和会议。这个想法很简单，但可能会演变成广泛的复杂性。在早期阶段，决定从小处着手，并在有需要时添加功能。新想法应该能够以最小的努力实现和测试。如果一个想法被证明是网站错误的方向，新的功能可以轻松删除并放弃。简单开始，快速发布小功能以获取反馈。

初始网站定义了三个主要部分：*演讲者*、*社区*和*会议*。每个部分都需要列出所有演讲者/社区/会议，提供查看选定项目详细信息的方式，并提供基于预定义标准搜索项目的方式。这将是初始发布的最小可行产品。

# 演讲者

在一开始，决定演讲者将是首要的关注点。演讲者将包含姓名、电子邮件地址、技术选择和位置。*Gravatar*将用于提供头像。未来不包括在最小可行产品中的增强功能包括演讲列表、旅行距离和评分。通过关注这一有限的功能，可以收集初始反馈，并将未来的努力适当引导。

# 社区

Speaker Meet 应用次要的关注点围绕在技术社区。Meetup 和用户组通常由专门的志愿者运营，他们总是在寻找新的和有趣的演讲者来参加他们的会议。网站社区部分的主要目标是定义成员社区的名字、位置、会议日期/时间以及技术选择。

# 会议

技术会议是 Speaker Meet 网站的第三和最后一个关注点。会议在要求上与社区相似，即它们需要名称、位置、日期和技术选择。它们主要在规模、范围和日期上有所不同。用户组通常每月举行一次会议，一位演讲者可能向一小群人发表演讲。会议通常每年举行一次，持续一到多天，有多个演讲者向更多的参与者发表演讲。

# 技术要求

这个项目的技术选择是在团队的知识和经验基础上早期决定的。前端网站将使用 JavaScript 和 ReactJS。后端将使用 C#和 WebAPI，以及.NET Core、Entity Framework 和 SQL Server。所有这些都将托管在 Azure 上。在编码开始之前就了解这些技术需求对于定义你的系统部分大有裨益。

# 摘要

现在，你应该对 Yak shaving（剃牛毛）及其如何帮助你开始有所了解。你已经被告诫要避免*前期大设计*和为了可能需要某个功能而提前创建可能不需要的东西（YAGNI）。确保测试小部分，扮演魔鬼的代言人，并测试负面情况。

在第六章“解决问题”中，将更详细地讨论演讲者见面网站的三个部分。将投入更多努力将这些初始声明分解成有意义的需求和可管理的作业单元。
