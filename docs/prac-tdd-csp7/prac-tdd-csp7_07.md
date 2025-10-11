# 第七章：测试驱动 C#应用程序

对于 Speaker Meet 应用来说，最重要的两个功能被确定为演讲者列表和查看单个演讲者详情的能力。演讲者列表和演讲者详情将为我们的最小可行产品带来最大的价值。

会议组织者、用户组管理员以及公众最可能关心的是找到关于演讲者的信息。考虑到这一点，演讲者史诗是演讲者见面应用开发的起点。

在本章中，我们涵盖了以下内容：

+   Speaker Meet 需求

+   API、服务和存储库测试

+   演讲者详情和演讲者列表 API

# 审查需求

为了开始，通过定义初始需求集，为 Speaker Meet 应用中的演讲者部分打下基础。这些需求将有助于消除歧义，并发展对需求的共同理解，以及定义在整个项目中使用的共同词汇。

摘要是展示项目目的和价值的地方。任何项目在获得批准进行工作之前，都必须证明它能为公司提供的价值。无论你是为财富 500 强公司工作还是为只有两个人的初创公司工作，这都是正确的。

数据字典很重要，因为它为项目提供了一个通用、无处不在的语言。这个术语“无处不在的语言”来自领域驱动设计，表示一种**共享或通用的语言**。这个想法是，业务和开发团队的共享术语被固定在一个法典中，可以被所有人查看和使用。

最后，但同样重要的是，需求必须以商定的格式呈现。具体的格式不如格式协议重要。无论格式如何，良好的需求都提供了交互的背景、正在进行的交互以及根据背景和特定操作预期的结果。

# 演讲者列表

Speaker Meet 网站上的演讲者部分包含系统中所有演讲者的列表。演讲者列表将为多个群体带来价值，包括会议和用户组组织者以及会议和用户组参与者。从用户交互的角度来看，演讲者列表允许进入演讲者详情。演讲者详情是真正提供价值的部分，以可用性、即将到来的活动和特定演讲者的联系信息的形式呈现。

初始时，演讲者列表将帮助组织者快速访问演讲者发现。组织者将能够找到他们所知道的演讲者，并发现他们不知道的演讲者。一旦找到或发现，组织者将能够查看特定演讲者的详细信息，最终，组织者将能够使用可用的联系信息联系演讲者。

参与者将从演讲者列表中受益，类似于组织者。然而，参与者有一个重要区别：他们正在寻找演讲者已经作为演讲者附加的事件。类似地，这种信息，类似于联系信息，将在演讲者详情中可用。

# API

API 是进入演讲者 Meet 应用程序核心系统的主要入口。演讲者列表 API 应返回演讲者摘要 ViewModel 的列表。这些 ViewModel 只包含应用程序这部分所需的信息。ViewModel 代表演讲者，但不一定是直接复制到数据库中的演讲者对象。

`SpeakerSummary` ViewModel 将根据系统的需求进行定义。这个 ViewModel 将逐渐增长，只包含其有限用途所需的属性。

要开始，需要向 API 添加一个新方法。对于要添加的第一个新功能，需要在 `SpeakerController` 中创建一个新的 `GetAll` 方法。但首先，必须创建一个测试。

# API 测试

回顾一下，`SpeakerController` 中的代码在没有失败的单元测试的情况下可能不会被编写。首先，应该创建一个名为 `GetAll` 的新测试文件。这里将包含与 `SpeakerController` 的 `GetAll` 方法相关联的所有测试。

在设置 `SpeakerController` 测试时存在重复。尝试想出可以最小化这种重复的方法。

这样的第一个测试应该是标准的 `ItExists` 测试。基于前几章的示例，`SpeakerController` 在构造函数中接受一个 `ISpeakerService`。同样可以应用提供 `Moq` 对象的方法。

```cs
[Fact]
public void ItExists()
{
  // Arrange
  var speakerServiceMock = new Mock<ISpeakerService>();
  var controller = new SpeakerController(speakerServiceMock.Object);

  // Act
  controller.GetAll();
}
```

将这次测试与为 `SpeakerController` 中的 `Search` 方法编写的第一次测试进行比较，你可能已经注意到已经发生了一些重复。记住，应该避免重复。不要忘记这个缩写，**DRY**（**不要重复自己**）。

为了使这次测试通过，需要在 `SpeakerController` 中添加一个空的 `GetAll` 方法。这将允许应用程序编译，从而通过这次测试。记住，编译失败即测试失败。

```cs
public void GetAll()
{           
}
```

接下来，通过创建一个新的测试来确保 `SpeakerController` 的 `GetAll` 方法返回一个 `OkObjectResult`。不要担心结果本身的类型。这将在下一个测试中解决。

```cs
[Fact]
public void ItReturnsOkObjectResult()
{
  // Arrange
  var speakerServiceMock = new Mock<ISpeakerService>();
  var controller = new SpeakerController(speakerServiceMock.Object);

  // Act
  var result = controller.GetAll();

  // Assert
  Assert.NotNull(result);
  Assert.IsType<OkObjectResult>(result);
}
```

为了使这个测试通过，方法应该返回 `IActionResult` 而不是 `void`。方法还应更改为返回 `Ok()` 以使测试通过。为了使测试按编写的方式通过，方法不需要返回任何其他内容。不要编写比使测试通过所需的代码更多的代码。

```cs
public IActionResult GetAll()
{           
  return Ok();
}
```

现在，确定该方法返回一个 `SpeakerSummary` 集合。

```cs
[Fact]
public void ItReturnsCollectionOfSpeakerSummary()
{
  // Arrange
  var speakerServiceMock = new Mock<ISpeakerService>();
  var controller = new SpeakerController(speakerServiceMock.Object);

  // Act
  var result = controller.GetAll() as OkObjectResult;

  // Assert
  Assert.NotNull(result);
  Assert.NotNull(result.Value);
  Assert.IsAssignableFrom<IEnumerable<SpeakerSummary>>(result.Value);
}
```

创建一个`SpeakerSummary`类来满足这个测试定义的要求。考虑一下新的`SpeakerSummary`应该放在哪里。这是一个 ViewModel，它将需要被测试访问，但不应该对应用程序的其他层可用。关于适当分离的更多内容将在未来的章节中介绍。

修改`SpeakerController`的`GetAll`方法，使其返回一组`SpeakerSummary`对象作为返回值。

```cs
public IActionResult GetAll()
{
  return Ok(new List<SpeakerSummary>());
}
```

# Moq

在前面的章节中，Moq 被用来为被测试的项目提供一组替代功能。对于模拟实例提供的结果是必需的，但实现对于测试的内容并不重要。

如前几章中的示例，`GetAll`的逻辑不应该在控制器本身中找到。相反，逻辑将包含在业务层中，特别是`ISpeakerService`的`SpeakerService`实现。当在`SpeakerController`中调用`GetAll`方法时，预期将调用`SpeakerService`的`GetAll`方法。

`SpeakerService`中不存在`GetAll`方法，因此下面的测试应该会失败。

```cs
[Fact]
public void ItCallsGetAllServiceOnce()
{
  // Arrange
  var speakerServiceMock = new Mock<ISpeakerService>();
  var controller = new SpeakerController(_speakerServiceMock.Object);

  // Act
  controller.GetAll();

  // Assert
  speakerServiceMock.Verify(mock => mock.GetAll(), Times.Once());
}
```

创建之前的测试强制在`ISpeakerService`接口中创建一个新的方法签名。以下方法签名应添加到`ISpeakerService`接口中。

```cs
IEnumerable<SpeakerSummary> GetAll();
```

为了使应用程序能够编译，`GetAll`也需要添加到`SpeakerService`类中。目前，这应该抛出一个异常。

```cs
public IEnumerable<SpeakerSummary> GetAll()
{
  throw new NotImplementedException();
}
```

为了使`ItCallsGetAllServiceOnce`测试通过，确保调用`SpeakerService`的`GetAll`方法。对于测试通过，调用方法本身就足够了，不需要返回值。

```cs
public IActionResult GetAll()
{
  _speakerService.GetAll();

  return Ok(new List<SpeakerSummary>());
}
```

注意，这将使测试通过，但这还不是完全正确的解决方案。需要一个新的测试来强制代码对服务返回值进行操作。继续前进，现在是时候对`SpeakerService.GetAll`调用的结果进行处理了。

```cs
[Fact]
public void GivenSpeakerServiceThenResultsReturned()
{
  // Arrange
  var speakers = new List<SpeakerSummary> { new SpeakerSummary
  {
    Name = "Speaker"
  } };

  var speakerServiceMock = new Mock<ISpeakerService>();
  speakerServiceMock.Setup(x => x.GetAll()).Returns(() => _speakers);

  var controller = new SpeakerController(speakerServiceMock.Object);

  // Act
  var result = controller.GetAll() as OkObjectResult;
  var speakers = ((IEnumerable<SpeakerSummary>)result.Value).ToList();

  // Assert
  Assert.Equal(_speakers, speakers);
}
```

不要忘记重构测试和代码。为了提高可读性，`Arrange`方法已被包含在之前的示例中。很可能，这些方法将被提取并定义为*字段*，并在构造函数中分配。

```cs
private readonly SpeakerController _controller;
private static Mock<ISpeakerService> _speakerServiceMock;
private readonly List<SpeakerSummary> _speakers;

public GetAll()
{
  _speakers = new List<SpeakerSummary> { new SpeakerSummary
  {
    Name = "test"
  } };

  _speakerServiceMock = new Mock<ISpeakerService>();
  _speakerServiceMock.Setup(x => x.GetAll()).Returns(() => _speakers);

  _controller = new SpeakerController(_speakerServiceMock.Object);
}
```

# 测试异常情况

如果请求的演讲者不存在，最好向 API 的消费者返回一个友好的错误信息。

```cs
[Fact]
public void GivenSpeakerNotFoundExceptionThenNotFoundObjectResult()
{
  // Arrange
  // Act
  var result = _controller.Get(-1);

  // Assert
  Assert.IsAssignableFrom<NotFoundObjectResult>(result);
}
```

创建一个新的异常类，命名为`SpeakerNotFoundException`。这将是由下面的`Moq`调用返回的特定异常。像之前的`SpeakerSummary`类文件一样，考虑一下`SpeakerNotFoundException`类文件应该保存的位置。

```cs
public class SpeakerNotFoundException : Exception
{
}
```

当提供一个特定 ID 时，在`Moq`中“抛出”一个新的异常需要一点设置。这类似于之前由`x.Get(It.IsAny<int>)`定义的。

```cs
_speakerServiceMock.Setup(x => x.Get(-1)).Returns(() => throw new SpeakerNotFoundException());
```

确保在之前的设置之后添加此操作，因为`Moq`将首先处理最后一个值。通过理解`Moq`将如何评估其上下文中设置的内容，避免出现假阳性。

接下来，修改控制器中的`Get`方法以捕获异常并返回适当的响应代码。

```cs
public IActionResult Get(int id)
{
  try
  {
    var speaker = _speakerService.Get(id);
    return Ok(speaker);
  }
  catch (SpeakerNotFoundException)
  {
    return NotFound();
  }
}
```

初始要求指出，应向客户端返回一个友好的错误消息。创建一个测试以确保在找不到具有提供的 ID 的演讲者时，向消费者返回一个友好的消息。

```cs
[Fact]
public void GivenSpeakerNotFoundExceptionThenMessageReturned()
{
  // Arrange
  // Act
  var result = _controller.Get(-1) as NotFoundObjectResult;

  // Assert
  Assert.NotNull(result);
  Assert.Equal("Speaker Not Found", result.Value);
}
```

为了使这个测试通过，必须修改`SpeakerNotFoundException`类以返回一个友好的错误信息。

```cs
public class SpeakerNotFoundException : Exception
{
  public SpeakerNotFoundException() : base("Speaker Not Found")
  {
  }
}
```

最后，修改控制器中的`Get`方法以返回消息。

```cs
public IActionResult Get(int id)
{
  try
  {
    var speaker = _speakerService.Get(id);
    return Ok(speaker);
  }
  catch (SpeakerNotFoundException ex)
  {
    return NotFound(ex.Message);
  }
}
```

# 服务

`GetAll`方法的业务逻辑应放在`SpeakerService`中。和之前一样，为了编写一行代码，必须首先编写一个测试。

# 服务测试

在上一个示例的基础上，从`ItExists`测试开始。

```cs
[Fact]
public void ItHasGetAllMethod()
{
  var speakerService = new SpeakerService();
  speakerService.GetAll();
}
```

由于此方法之前已添加到`SpeakerService`中，尽管使用了`NotImplementedException`，但最好看到这个测试因为正确的原因而失败。从`SpeakerService`中删除`GetAll`方法，以便应用程序无法编译。现在，将方法添加回来，以查看应用程序再次编译，因此这个测试通过。这次，让该方法返回`null`而不是抛出一个新的`NotImplementedException`。

```cs
public IEnumerable<SpeakerSummary> GetAll()
{
  return null;
}
```

现在，确保`GetAll`方法通过创建一个新的测试来返回一个`SpeakerSummary`对象的集合。

```cs
[Fact]
public void ItReturnsCollectionOfSpeakerSummary()
{
  // Arrange
  // Act
  var speakers = _speakerService.GetAll();

  // Assert
  Assert.NotNull(speakers);
  Assert.IsAssignableFrom<IEnumerable<SpeakerSummary>>(speakers);
}
```

修改`SpeakerService`中的`GetAll`方法以使这个测试通过。使这个测试通过所需的最小代码量是返回一个`SpeakerSummary`对象的新`List`。不要添加超过使这个测试通过所需的代码。

```cs
public IEnumerable<SpeakerSummary> GetAll()
{
  return new List<SpeakerSummary>();
}
```

在前一章的示例的基础上，使用之前硬编码的数据。将`hardCodedSpeakers`提取到一个字段中，以便在`Search`方法和`GetAll`方法中使用这些数据：

```cs
public readonly List<Speaker> HardCodedSpeakers = new List<Speaker>
{
  new Speaker {Name = "Josh"},
  new Speaker {Name = "Joshua"},
  new Speaker {Name = "Joseph"},
  new Speaker {Name = "Bill"}
};
```

注意，该字段已被公开。这将允许测试使用这些数据来比较*断言*。不用担心，这个字段以及其中包含的硬编码数据将不会长期存在。一旦这些不再需要，它们可以安全地被删除。

现在，创建一个测试以确保`SpeakerService`中的`GetAll`方法返回了`HardCodedSpeakers`中包含的所有数据。首先，验证方法返回的硬编码数据中的演讲者数量是否相同。

```cs
[Fact]
public void ItReturnsAllSpeakers()
{
  // Arrange
  // Act
  var speakers = _speakerService.GetAll();

  // Assert
  Assert.NotNull(speakers);
  Assert.IsAssignableFrom<IEnumerable<SpeakerSummary>>(speakers);
  Assert.Equal(_speakerService.HardCodedSpeakers.Count, speakers.Count());
}
```

为了使这个测试通过，只需遍历硬编码的值，并为每个条目返回一个新的`SpeakerSummary`。由于测试尚未检查返回的演讲者的值，所以只需要返回正确的`SpeakerSummary`对象数量。

```cs
public IEnumerable<SpeakerSummary> GetAll()
{
  return HardCodedSpeakers.Select(speaker => new SpeakerSummary());
}
```

现在，确保演讲者被正确地转换为`SpeakerSummary`对象。首先，检查`Name`属性是否相同。

```cs
[Fact]
public void ItReturnsAllSpeakersWithName()
{
  // Arrange
  // Act
  var speakers = _speakerService.GetAll().ToList();

  // Assert
  Assert.NotNull(speakers);
  Assert.IsAssignableFrom<IEnumerable<SpeakerSummary>>(speakers);

  for (var i = 0; i < speakers.Count; i++)
  {
    Assert.NotNull(_speakerService.HardCodedSpeakers[i].Name);
    Assert.Equal(_speakerService.HardCodedSpeakers[i].Name, speakers[i].Name);
  }
}
```

现在，通过在 `GetAll` 方法中分配 `Name` 来使这个测试通过。

```cs
public IEnumerable<SpeakerSummary> GetAll()
{
  return HardCodedSpeakers.Select(speaker => new SpeakerSummary
  {
    Name = speaker.Name     
  });
}
```

继续构建具有所需属性的 `SpeakerSummary` 对象。已添加 `Name` 属性。现在，添加一个 ID 并确保它被正确分配和返回。

```cs
[Fact]
public void ItReturnsAllSpeakersWithId()
{
  // Arrange
  // Act
  var speakers = _speakerService.GetAll().ToList();

  // Assert
  Assert.NotNull(speakers);
  Assert.IsAssignableFrom<IEnumerable<SpeakerSummary>>(speakers);

  for (var i = 0; i < speakers.Count; i++)
  {
    Assert.NotNull(_speakerService.HardCodedSpeakers[i].Id);
    Assert.Equal(_speakerService.HardCodedSpeakers[i].Id, speakers[i].Id);
  }
}
```

为了使这个通过，需要在 `SpeakerService` 的 `GetAll` 方法中映射一个 ID，并添加一个 ID 属性到 `Speaker` 和 `SpeakerSummary` 对象中。

```cs
public IEnumerable<SpeakerSummary> GetAll()
{
  return HardCodedSpeakers.Select(speaker => new SpeakerSummary
  {
    Id = speaker.Id,
    Name = speaker.Name     
  });
}
```

接下来，添加一个 `Location` 供 `GetAll` 方法返回。这也需要修改 `Speaker` 和 `SpeakerSummary` 对象。在 `HardCodedSpeakers` 集合中为新位置属性提供独特的值，以确保值被正确返回。

```cs
[Fact]
public void ItReturnsAllSpeakersWithLocation()
{
  // Arrange
  // Act
  var speakers = _speakerService.GetAll().ToList();

  // Assert
  Assert.NotNull(speakers);
  Assert.IsAssignableFrom<IEnumerable<SpeakerSummary>>(speakers);

  for (var i = 0; i < speakers.Count; i++)
  {
    Assert.NotNull(_speakerService.HardCodedSpeakers[i].Location);
    Assert.Equal(_speakerService.HardCodedSpeakers[i].Location, speakers[i].Location);
  }
}
```

将一些位置添加到硬编码的数据中。

```cs
public readonly List<Speaker> HardCodedSpeakers = new List<Speaker>
{
  new Speaker {Id = 1, Name = "Josh", Location = “Tampa, FL”},
  new Speaker {Id = 2, Name = "Joshua", Location = “Louisville, KY”},
  new Speaker {Id = 3, Name = "Joseph", Location = “Las Vegas, NV”},
  new Speaker {Id = 4, Name = "Bill", Location = “New York, NY”},
};
```

最后，将位置映射到 `SpeakerSummary` 视图模型。

```cs
public IEnumerable<SpeakerSummary> GetAll()
{
  return HardCodedSpeakers.Select(speaker => new SpeakerSummary
  {
    Id = speaker.Id,
    Name = speaker.Name,  
    Location = speaker.Location,
  });
}
```

如前所述，测试应该有一个单一的操作。这并不排除它们可以有多个断言。为了最小化重复，应该合并属性测试。

# 清洁测试

测试套件应该得到良好的维护。这是应用程序的第一个消费者，并为系统的功能提供了最全面的文档。为了清理刚刚创建的测试，是时候进行一些重构了。

将 `SpeakerSummary` 属性合并为单个动作，包含多个断言。这将有助于使测试套件更小、更易于阅读和维护，并且可能会更快地执行。一个执行快速的测试套件更有可能被开发者频繁运行。

将 `ItReturnsAllSpeakersWithName` 重命名为 `ItReturnsAllSpeakersWithProperties` 并将 `ID` 和 `Location` 测试合并到这个测试中。

```cs
[Fact]
public void ItReturnsAllSpeakersWithProperties()
{
  // Arrange
  // Act
  var speakers = _speakerService.GetAll().ToList();

  // Assert
  Assert.NotNull(speakers);
  Assert.IsAssignableFrom<IEnumerable<SpeakerSummary>>(speakers);

  for (var i = 0; i < speakers.Count; i++)
  {
    Assert.NotNull(_speakerService.HardCodedSpeakers[i].Name);
    Assert.Equal(_speakerService.HardCodedSpeakers[i].Name, speakers[i].Name);
    Assert.NotNull(_speakerService.HardCodedSpeakers[i].Id);
    Assert.Equal(_speakerService.HardCodedSpeakers[i].Id, speakers[i].Id);
    Assert.NotNull(_speakerService.HardCodedSpeakers[i].Location);
    Assert.Equal(_speakerService.HardCodedSpeakers[i].Location, speakers[i].Location);
  }
}
```

# 存储

在前一章中，数据是在 `SpeakerController` 类中硬编码的。数据随后被移动到 `SpeakerService` 中的硬编码集合中。最终，数据将持久化到数据库中。目前，将数据从 `SpeakerService` 中移除就足够了。

将使用存储库层来将数据访问层与其他应用程序部分分离。为了实现这一点，必须引入一个存储库。为了创建存储库，必须建立需求。通过要求 `SpeakerService` 接受一个 `IRepository` 来缓慢开始。

```cs
[Fact]
public void ItAcceptsIRepository()
{
  // Arrange
  IRepository fakeRepository = new FakeRepository();

  // Act
  var service = new SpeakerService(fakeRepository);

  // Assert
  Assert.NotNull(service);
}
```

当然，这将导致应用程序无法编译。创建一个 `IRepository` 接口，一个 `FakeRepository` 类，并修改 `SpeakerService` 以接受一个 `IRepository`。

```cs
public SpeakerService(IRepository repository)
{
}
```

# `IRepository` 接口

`IRepository` 接口将定义与数据访问层交互的方法签名。这个接口将缓慢增长，由测试指导。在 第八章 的 *抽象问题* 中，将提供更多细节并引入更多概念。目前，这个接口将仅仅是用于 `SpeakerService` 测试的 `FakeRepository` 的一个合同。

# FakeRepository

现在，`FakeRepository`已经创建，可以将`HardCodedSpeakers`移动到`FakeRepository`中。首先，需要创建几个迭代测试。

与你自己创建的`FakeRepository`交互允许你替换值并为测试目的创建额外的功能。

```cs
[Fact]
public void ItCallsRepository()
{
  // Arrange
  FakeRepository fakeRepository = new FakeRepository();
  var service = new SpeakerService(fakeRepository);

  // Act
  var speakers = service.GetAll();

  // Assert
  Assert.True(fakeRepository.GetAllCalled);
}
```

通过引入一个公共字段，可以在`FakeRepository`中应用与`Moq`相同的功能。

```cs
public bool GetAllCalled { get; private set; }

public void GetAll()
{
  GetAllCalled = true;
}
```

现在，通过修改现有的`ItReturnsAllSpeakers`和`ItReturnsAllSpeakersWithProperties`测试，确保当调用`GetAll`时`FakeRepository`返回`HardCodedSpeakers`。

```cs
[Fact]
public void ItReturnsAllSpeakers()
{
  // Arrange
  // Act
  var speakers = _speakerService.GetAll();

  // Assert
  Assert.NotNull(speakers);
  Assert.IsAssignableFrom<IEnumerable<SpeakerSummary>>(speakers);
  Assert.Equal(_fakeRepository.HardCodedSpeakers.Count, speakers.Count());
}
```

```cs
[Fact]
public void ItReturnsAllSpeakersWithProperties()
{
  // Arrange
  // Act
  var speakers = _speakerService.GetAll().ToList();

  // Assert
  Assert.NotNull(speakers);
  Assert.IsAssignableFrom<IEnumerable<SpeakerSummary>>(speakers);

  for (var i = 0; i < speakers.Count; i++)
  {
    Assert.NotNull(_fakeRepository.HardCodedSpeakers[i].Name);
    Assert.Equal(_fakeRepository.HardCodedSpeakers[i].Name, speakers[i].Name);
    Assert.NotNull(_fakeRepository.HardCodedSpeakers[i].Id);
    Assert.Equal(_fakeRepository.HardCodedSpeakers[i].Id, speakers[i].Id);
    Assert.NotNull(_fakeRepository.HardCodedSpeakers[i].Location);
    Assert.Equal(_fakeRepository.HardCodedSpeakers[i].Location, speakers[i].Location);
  }
}    
```

可能看起来已经付出了很多努力，只是为了把问题推到一边。所有这些都是为了成功地向一个真正功能性和可维护的应用程序迈进所必需的努力。然而，还有更多的工作要做。

# 使用工厂与`FakeRepository`交互

到目前为止，这已经是一个相对直接的练习。`Speaker`类代表了将被持久化到数据库的对象的形状。`HardCodedSpeakers`集合代表了数据库中所有的*演讲者*。

无论是在测试文件中还是在其他地方，拥有或维护一组硬编码的数据都不是完全理想的。为测试编写者提供一种定义测试数据的方式将更加灵活。

使用工厂创建演讲者并将其添加到`FakeRepository`提供了一种更干净、更易于维护的方式来管理需要特定数据场景的测试的状态。

```cs
public static class SpeakerFactory
{
  public static Speaker Create(FakeRepository fakeRepository, int id = 1, string name = "Joshua", string location = "Springfield, IL")
  {
    var speaker = new Speaker
    {
      Id = id,
      Name = name,
      Location = location
    };

    fakeRepository.Speakers.Add(speaker);

    return speaker;
  }
}
```

注意，已为 id、name 和 location 定义了默认值。这允许用户在需要时提供特定值，或者在没有提供它们的情况下继续操作。

`FakeRepository`还必须进行修改，以删除`HardCodedSpeakers`并公开一个演讲者集合。

```cs
public class FakeRepository : IRepository
{
  public List<Speaker> Speakers = new List<Speaker>();
  public bool GetAllCalled { get; private set; }

  public IEnumerable<Speaker> GetAll()
  {
    GetAllCalled = true;

    return Speakers;
  }
}
```

现在，对于每个测试，可以提供一组特定的数据来进行测试。所需的所有操作只是调用工厂来创建一个或多个演讲者并将其添加到`FakeRepository`中。

```cs
public GetAll()
{
  _fakeRepository = new FakeRepository();
  SpeakerFactory.Create(_fakeRepository);
  _speakerService = new SpeakerService(_fakeRepository);
}
```

如果你一直在跟随前几章中的相同解决方案，你可能需要修改搜索测试。

```cs
public Search()
{
  var fakeRepository = new FakeRepository();
  SpeakerFactory.Create(fakeRepository);
  SpeakerFactory.Create(fakeRepository, name:"Josh");
  SpeakerFactory.Create(fakeRepository, name:"Joseph");
  SpeakerFactory.Create(fakeRepository, name:"Bill");
  _speakerService = new SpeakerService(fakeRepository);
}
```

# 软删除

决定能够从系统中“软删除”一个演讲者将是有用的。一个“软删除”允许记录被标记为已删除，而不需要物理删除记录。这将有助于维护引用完整性，同时实现预期的结果。

首先，向`SpeakerFactory`添加一个扩展方法`IsDeleted`，该方法将设置演讲者以供删除。

```cs
public static Speaker IsDeleted(this Speaker speaker)
{
  speaker.IsDeleted = true;
  return speaker;
}
```

现在，创建一个测试以确保当调用`GetAll`时不会返回这个演讲者。

```cs
[Fact]
public void GivenSpeakerIsDeletedSpeakerIsNotReturned()
{
  // Arrange
  var fakeRepository = new FakeRepository();
  SpeakerFactory.Create(fakeRepository).IsDeleted();
  var speakerService = new SpeakerService(fakeRepository);

  // Act
  var speakers = speakerService.GetAll().ToList();

  // Assert
  Assert.NotNull(speakers);
  Assert.IsAssignableFrom<IEnumerable<SpeakerSummary>>(speakers);
  Assert.Equal(0, speakers.Count);           
}
```

最后，修改代码以确保不会返回“已删除”的演讲者。

```cs
public IEnumerable<SpeakerSummary> GetAll()
{
  return _repository.GetAll()
    .Where(x => !x.IsDeleted)
    .Select(speaker => new SpeakerSummary
      {
        Id = speaker.Id,
        Name = speaker.Name,
        Location = speaker.Location
      });
}
```

# 演讲者详细信息

接下来，我们将讨论演讲者详细信息。我们选择在后端应用程序中继续，因为我们将在接下来的章节中将整个程序结合起来。

如前所述，这是第一组需求真正价值所在的地方。用户组和会议组织者将能够使用详细信息视图中的信息联系演讲者。

# API

为了返回单个演讲者的详细信息，需要一个新端点。需要一个名为`Get`的新方法，它将接受一个整数 ID 并返回一个`SpeakerDetail` ViewModel。

# API 测试

为了开始，添加一个名为`Get`的新测试类。现在，添加一个测试来检查`Get`方法是否存在。

```cs
[Fact]
public void ItExists()
{
  // Arrange
  var speakerServiceMock = new Mock<ISpeakerService>();
  var controller = new SpeakerController(speakerServiceMock.Object);

  // Act
  var result = controller.Get();
}
```

通过向`SpeakerController`添加`Get`方法来使这个测试通过。注意，在以下示例中，`Arrange`测试设置已经被移动到测试类的构造函数中。

```cs
[Fact]
public void ItExists()
{
  // Arrange
  // Act
  _controller.Get();
}
```

接下来，确保`Get`方法接受一个整数。

```cs
[Fact]
public void ItAcceptsInteger()
{
  // Arrange
  // Act
  _controller.Get(1);
}
```

为了使这个测试通过，需要在`Get`方法中添加一个整数参数。此时，可以安全地删除`ItExists`方法。这个测试需要修改以适应这个变化，并且其存在性将通过新的测试来验证。

```cs
public void Get(int id)
{
} 
```

现在测试已经确认`Get`方法接受一个整数，现在确认它返回一个`Ok`结果。

```cs
[Fact]
public void ItReturnsOkObjectResult()
{
  // Arrange
  // Act
  var result = _controller.Get(1);

  // Assert
  Assert.IsType<OkObjectResult>(result);
}
```

现在，确保结果是`SpeakerDetail`类型。

```cs
[Fact]       
public void ItReturnsSpeakerDetail()
{
  // Arrange
  // Act
  var result = _controller.Get(1) as OkObjectResult;

  // Assert
  Assert.NotNull(result);
  Assert.NotNull(result.Value);
  Assert.IsType<SpeakerDetail>(result.Value);
}
```

为了使这个测试通过，需要一个`SpeakerDetail`对象。创建一个没有任何属性的空对象，因为测试目前还没有要求任何属性。

```cs
public IActionResult Get(int id)
{
  return Ok(new SpeakerDetail());
} 
```

就像`GetAll`方法一样，这个操作的逻辑应该位于*Service*中。创建一个测试来检查`SpeakerService`中的`Get`方法是否使用`Moq`被调用。

```cs
[Fact]
public void ItCallsGetServiceOnce()
{
  // Arrange
  // Act
  _controller.Get(1);

  // Assert
  _speakerServiceMock.Verify(mock => mock.Get(), Times.Once());
}
```

为了使应用程序能够编译，需要在`IService`接口中添加一个`Get`方法的签名。

```cs
void Get();
```

为了使应用程序能够编译，需要修改`SpeakerService`。

```cs
public void Get()
{
  throw new NotImplementedException();
}
```

为了使这个测试通过，只需调用`SpeakerService`的`Get`方法。

```cs
public IActionResult Get(int id)
{
  _speakerService.Get();

  return Ok(new SpeakerDetail());
}
```

`ISpeakerService`中`Get`方法的签名需要修改为返回`SpeakerDetail`而不是`void`。

```cs
SpeakerDetail Get();
```

现在确保传递给`SpeakerController`中的`Get`方法的 ID 与传递给`SpeakerService`中的`Get`方法的 ID 相同。

```cs
[Fact]
public void ItCallsGetServiceWithProvidedId()
{
  // Arrange
  const int id = 1;

  // Act
  _controller.Get(id);

  // Assert
  _speakerServiceMock.Verify(mock => mock.Get(id),Times.Once());
}
```

这将需要对`ISpeakerService`接口以及`SpeakerService`类进行修改。

```cs
SpeakerDetail Get(int id);

...

public SpeakerDetail Get(int id)
{
  throw new NotImplementedException();
}
```

现在返回`SpeakerService`的`Get`方法的返回结果。

```cs
[Fact]
public void GivenSpeakerServiceThenResultIsReturned()
{
  // Arrange
  // Act
  var result = _controller.Get(1) as OkObjectResult;

  // Assert
  Assert.NotNull(result);
  var speaker = ((SpeakerDetail)result.Value);
  Assert.Equal(_speaker, speaker);
}
```

为了使这个测试通过，只需返回`Get`方法的返回结果。

```cs
public IActionResult Get(int id)
{
  var speaker = _speakerService.Get();

  return Ok(speaker);
}
```

这是`SpeakerController`当前最终结果的样子：

```cs
using Microsoft.AspNetCore.Mvc;
using SpeakerMeet.Api.Services;

namespace SpeakerMeet.Api.Controllers
{
  [Route("api/[controller]")]
  public class SpeakerController : Controller
  {
    private readonly ISpeakerService _speakerService;

    public SpeakerController(ISpeakerService speakerService)
    {
      _speakerService = speakerService;
    }

    [Route("search")]
    public IActionResult Search(string searchString)
    {
      var speakers = _speakerService.Search(searchString);

      return Ok(speakers);
    }

    public IActionResult GetAll()
    {
      var speakers = _speakerService.GetAll();

      return Ok(speakers);
    }

    public IActionResult Get(int id)
    {
      var speaker = _speakerService.Get(id);

      return Ok(speaker);
    }
  }
}
```

# 服务

现在控制器正在调用`Moq`服务的`Get`方法，现在是时候在`SpeakerService`中实现这个方法了。

# 服务测试

`Get`方法是由于之前的测试声明的。创建一个新的`ItExists`测试并删除实现以查看它失败。

```cs
[Fact]
public void ItHasGetMethod()
{
  // Act
  // Arrange
  _speakerService.Get();
}
```

通过实现`Get`方法来使这个测试通过。

```cs
public void Get()
{
}
```

现在确保`Get`方法接受一个整数。

```cs
[Fact]
public void ItAcceptsAnInteger()
{
  // Act
  // Arrange
  _speakerService.Get(1);
}
```

修改`Get`方法以接受一个整数。

```cs
public SpeakerDetail Get(int id)
{
}
```

测试`Get`方法返回一个`SpeakerDetail`对象。

```cs
[Fact]
public void ItReturnsSpeakerDetail()
{
  // Arrange
  // Act
  var speaker = _speakerService.Get(1);

  // Assert
  Assert.NotNull(speaker);
  Assert.IsType<SpeakerDetail>(speaker);
}
```

为了使这个测试通过，只需返回一个新的`SpeakerDetail`对象。

```cs
public SpeakerDetail Get(int id)
{
  return new SpeakerDetail();
}
```

验证返回的`SpeakerDetail`包含一个 ID。

```cs
[Fact]
public void GivenSpeakerReturnsId()
{
  // Arrange
  // Act
  var speaker = _speakerService.Get(1);

  // Assert
  Assert.Equal(1, speaker.Id);
}
```

现在使测试通过。

```cs
public SpeakerDetail Get(int id)
{
  return new SpeakerDetail
  {
    Id = 1,
    Name = "Joshua"
  };
}
```

确认`SpeakerDetail`包含一个名字。

```cs
[Fact]
public void GivenSpeakerReturnsName()
{
  // Arrange
  // Act
  var speaker = _speakerService.Get(1);

  // Assert
  Assert.Equal("Joshua", speaker.Name);
}
```

并使测试通过。

```cs
public SpeakerDetail Get(int id)
{
  return new SpeakerDetail
  {
    Id = 1,
    Name = "Joshua"
  };
}
```

最后，确保返回`Location`。

```cs
[Fact]
public void GivenSpeakerReturnsLocation()
{
  // Arrange
  // Act
  var speaker = _speakerService.Get(1);

  // Assert
  Assert.Equal("Tampa, FL", speaker.Location);
}
```

通过返回位置来使测试通过。

```cs
public SpeakerDetail Get(int id)
{
  return new SpeakerDetail
  {
    Id = 1,
    Name = "Joshua",
    Location = "Tampa, FL"
  };
}
```

# 清理测试

不要忘记清理和重构测试。折叠属性测试。

```cs
[Fact]
public void GivenSpeakerReturnsSpeakerWithProperties()
{
  // Arrange
  // Act
  var speaker = _speakerService.Get(1);

  // Assert
  Assert.Equal(1, speaker.Id);
  Assert.Equal("Joshua", speaker.Name);
}
```

# 更多来自存储库的信息

现在，验证存储库是否被调用。

```cs
[Fact]
public void ItCallsRepository()
{
  // Arrange
  var fakeRepository = new FakeRepository();
  var service = new SpeakerService(fakeRepository);

  // Act
  service.Get(-1);

  // Assert
  Assert.True(fakeRepository.GetCalled);
}
```

现在，通过实现必要的修改来确保测试通过。

```cs
public SpeakerDetail Get(int id)
{
  _repository.Get();

  return new SpeakerDetail
  {
    Id = 1,
    Name = "Joshua"
  };
}
```

# 额外的工厂工作

如前所述，如果值不是硬编码的，那就很理想了。使用工厂创建一个说话者，并让存储库返回指定的说话者。

```cs
[Fact]
public void ItReturnsSpeakerFromRepository()
{
  // Arrange
  var fakeRepository = new FakeRepository();
  var expectedSpeaker = SpeakerFactory.Create(fakeRepository, 2, "Bill");
  var service = new SpeakerService(fakeRepository);

  // Act
  var actualSpeaker = service.Get(expectedSpeaker.Id);

  // Assert
  Assert.True(fakeRepository.GetCalled);
  Assert.Equal(expectedSpeaker.Id, actualSpeaker.Id);
  Assert.Equal(expectedSpeaker.Name, actualSpeaker.Name);
}
```

要使这个测试通过，需要对`IRepository`、`FakeRepository`和`Service`进行修改。

`IRepository`:

```cs
        Speaker Get(int id);
```

`FakeRepository`:

```cs
public Speaker Get(int id)
{
  GetCalled = true;

  return Speakers.Find(x => x.Id == id);
}
```

`Service`:

```cs
public SpeakerDetail Get(int id)
{
  var speaker = _repository.Get(id);

  return new SpeakerDetail
  {
    Id = speaker.Id,
    Name = speaker.Name
  };
}
```

所有之前的`ItReturnsSpeakerFromRepository`测试现在都可以删除。这些都是为了达到这个目的而进行的繁琐工作。

现在，为了确保这可以与多个值一起工作，将最后一个测试转换为理论集。

```cs
[Theory]
[InlineData(1, "Joshua")]
[InlineData(2, "Bill")]
[InlineData(3, "Suzie")]
public void ItReturnsSpeakerFromRepository(int id, string name)
{
  // Arrange
  var expectedSpeaker = SpeakerFactory.Create(_fakeRepository, id, name);
  var service = new SpeakerService(_fakeRepository);

  // Act
  var actualSpeaker = service.Get(expectedSpeaker.Id);

  // Assert
  Assert.True(_fakeRepository.GetCalled);
  Assert.Equal(expectedSpeaker.Id, actualSpeaker.Id);
  Assert.Equal(expectedSpeaker.Name, actualSpeaker.Name);
}
```

所有测试都应该通过。如果由于某种原因遇到失败的测试，不要继续，直到失败的测试得到解决。

# 测试异常情况

测试异常情况是一个非常重要的步骤。在这种情况下，业务已经定义了一个情况，即如果说话者不存在，我们将返回 SPEAKER NOT FOUND 错误。对于开发者来说，考虑业务可能遗漏的任何重大边缘情况也很重要。如果可能的话，与业务讨论它们，并将它们添加到规范中。

现在测试说话者必须存在。

```cs
[Fact]
public void GivenSpeakerNotFoundThenSpeakerNotFoundException()
{
  // Arrange
  var service = new SpeakerService(_fakeRepository);

  // Act
  var exception = Record.Exception(() => service.Get(-1));

  // Assert
  Assert.IsAssignableFrom<SpeakerNotFoundException>(exception);
}
```

并使其通过。

```cs
public SpeakerDetail Get(int id)
{
  var speaker = _repository.Get(id);

  if (speaker == null)
  {
    throw new SpeakerNotFoundException();
  }

  return new SpeakerDetail
  {
    Id = speaker.Id,
    Name = speaker.Name
  };
}
```

现在，验证说话者没有被删除。如果被删除，则抛出相同的`SpeakerNotFoundException`*.*

```cs
[Fact]
public void GivenSpeakerIsDeletedThenSpeakerNotException()
{
  // Arrange
  var expectedSpeaker = SpeakerFactory.Create(_fakeRepository).IsDeleted();
  var service = new SpeakerService(_fakeRepository);

  // Act
  var exception = Record.Exception(() => service.Get(expectedSpeaker.Id));

  // Assert
  Assert.IsAssignableFrom<SpeakerNotFoundException>(exception);
}
```

使这个测试通过的最简单、最有效的方法是在找到的说话者已被删除时抛出异常。对`Get`方法进行必要的更改。

```cs
public SpeakerDetail Get(int id)
{
  var speaker = _repository.Get(id);

  if (speaker == null || speaker.IsDeleted)
  {
    throw new SpeakerNotFoundException();
  }

  return new SpeakerDetail
  {
    Id = speaker.Id,
    Name = speaker.Name
   };
}
```

# 摘要

现在，你应该对围绕“说话者见面”应用的要求感到相当舒适，并且已经对后端应用说话者部分的 API、Service 和 Repository 层有了良好的介绍。Mocks 和 Fakes 在程序的测试驱动中继续发挥作用。

在第八章“抽象问题”中，将讨论更多关于抽象的内容。`SpeakerSummary`和`SpeakerDetail`的模型将扩展以包含更多属性。将提供更多关于如何最佳地增加应用程序的功能和复杂性的详细信息。
