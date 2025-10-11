# 抽象问题

现在，很容易在网上找到资源集成到你的应用程序中。许多提供的功能非常适合任何数量的应用程序。毕竟，为什么要在别人已经做了大部分工作的基础上浪费时间呢？

在本章中，我们将了解：

+   抽象 Gravatar 服务

+   扩展存储库模式

+   使用通用存储库和 Entity Framework

# 抽象问题

现在有很多工具和库可以帮助创建一个功能齐全的应用程序。在应用程序中集成这些第三方系统可能相当容易。然而，有时你可能需要用另一个第三方库替换一个。或者，你可能会发现自己依赖于第三方系统提供的实现，结果发现该实现随着后续更新而发生了变化。你该如何避免这些潜在的问题？

依赖外部控制之外的代码可能会在未来给你带来问题。如果依赖的库中引入了变更，可能会破坏你的系统。或者，如果你的需求发生变化，系统不再满足你的特定需求，你可能需要重写应用程序的大量部分。

不要直接依赖任何第三方系统。抽象出细节，使你的应用程序只依赖于你定义的接口。如果你定义了接口并且只暴露你需要的功能，那么在需要时进行更改将变得非常简单。更改可能包括小的更新或替换整个库。你希望这些更改对应用程序的其他部分影响最小。

不要依赖第三方实现；专注于驱动测试你的代码。

在考虑测试驱动开发的应用程序开发过程中，往往会诱使人们测试第三方软件。虽然确保任何第三方库或工具集成到你的系统中表现良好很重要，但最好专注于你系统中的行为。确保你的系统能够良好地处理你希望公开的功能。

这意味着你应该处理*正常路径*以及可能抛出的任何*异常*。优雅地恢复错误将允许你的应用程序在第三方服务未按预期运行时继续运行。

# Gravatar

Speaker Meet 应用程序使用 Gravatar 来显示演讲者、社区和会议头像图像。Gravatar 是一个在线服务，将电子邮件地址与图像关联起来。用户可以创建一个账户并添加他们希望任何请求其图像的服务显示的图像。通过创建用户的电子邮件地址的 MD5 哈希并使用哈希值从 Gravatar 请求图像来检索图像。通过依赖哈希值，用户的电子邮件地址不会被暴露。

Gravatar 服务允许消费者向 HTTP 调用提供可选参数，以请求特定大小、评级或默认图像（如果未找到任何图像）。这些选项包括：

+   **s**：请求的图像大小；默认情况下，这是 80 x 80 像素

+   **d**：如果未找到任何图像，则返回的默认图像；选项包括 404、**mm**（**神秘人**）、identicon 等

+   **f**：强制默认；即使找到图像也总是返回默认图标

+   **r**：评级；用户可以将他们的图像标记为 G、PG、R 和 X

通过提供这些值，你可以控制你希望在应用程序中显示的图像的大小和类型。Speaker Meet 应用程序依赖于 Gravatar 的默认提供项。

# 从接口开始

看起来 Gravatar 网站上有许多可用的选项。为了保护应用程序的其他部分，Gravatar 的功能将通过 Speaker Meet 应用程序中的一个类来公开。这个功能首先由一个接口定义。

所需的接口可能看起来像这样：

```cs
  public interface IGravatarService
  {
    string GetGravatar(string emailAddress);
    string GetGravatar(string emailAddress, int size);
    string GetGravatar(string emailAddress, int size, string rating);
    string GetGravatar(string emailAddress, int size, string rating, 
    string imageType);
  }
```

要开始，你必须首先编写一些测试。记住，你不应该在没有失败的单元测试的情况下编写任何生产代码。

# 实现接口的测试版本

为了创建一个名为 `IGravatarService` 的接口，首先必须在应用程序中有一个需求。在 `SpeakerServiceTests` 的 `Get` 类中创建一个名为 `ItTakesGravatarService` 的测试：

```cs
[Fact]
public void ItTakesGravatarService()
{
  // Arrange
  var fakeGravatarService = new FakeGravatarService();
  var service = new SpeakerService(_fakeRepository, fakeGravatarService);           
}
```

这将导致编译错误。创建一个 `IGravatarService` 并修改 `SpeakerService` 的构造函数，使其成为一个参数。

接口：

```cs
public interface IGravatarService
{
}
```

`SpeakerService` 方法：

```cs

public SpeakerService(IRepository repository, IGravatarService gravatarService)
{
  _repository = repository;
}
```

为了让测试编译，创建一个 `FakeGravatarService`，它可以提供给正在测试的 `SpeakerService`。记住，你并不是在测试 `FakeGravatarService`，而是在测试 `SpeakerService` 是否接受一个 `IGravatarService` 实例。

现在，确保当请求一个单独的 *Speaker* 时调用 `FakeGravatarServiceGetGravatar` 方法。

```cs
[Fact]
public void ItCallsGravatarService()
{
  // Arrange
  var expectedSpeaker = SpeakerFactory.Create(_fakeRepository);
  var service = new SpeakerService(_fakeRepository, _fakeGravatarService);

  // Act
  service.Get(expectedSpeaker.Id);

  // Assert
  Assert.True(_fakeGravatarService.GetGravatarCalled);
}
```

修改接口以添加一个 `GetGravatar` 方法：

```cs
public interface IGravatarService
{
  void GetGravatar();
}
```

在 `FakeGravatarService` 中实现此方法。这与 第七章，*测试驱动 C# 应用程序*中的 `FakeRepository` 的 `GetCalled` 检查类似：

```cs
public class FakeGravatarService : IGravatarService
{
  public bool GetGravatarCalled { get; set; }

  public void GetGravatar()
  {
    GetGravatarCalled = true;
  } 
}
```

接下来，确保当调用 `SpeakerService` 的 `Get(id)` 时执行 `GetGravatar` 函数：

```cs
private readonly IRepository _repository;
private readonly IGravatarService _gravatarService;

public SpeakerService(IRepository repository, IGravatarService gravatarService)
{
  _repository = repository;
  _gravatarService = gravatarService;
}

public Models.SpeakerDetail Get(int id)
{
  var speaker = _repository.Get(id);

  if (speaker == null || speaker.IsDeleted)
  {
    throw new SpeakerNotFoundException();
  }

  var gravatar = _gravatarService.GetGravatar();

  return new Models.SpeakerDetail
  {
    Id = speaker.Id,
    Name = speaker.Name,
    Location = speaker.Location
  };
}
```

测试现在应该通过。然而，`FakeGravatarService` 目前并没有提供任何实际价值。`GetGravatar` 方法应该使用提供的电子邮件地址执行：

```cs
[Fact]
public void ItCallsGravatarServiceWithEmail()
{
  // Arrange
  var expectedSpeaker = SpeakerFactory.Create(_fakeRepository, emailAddress: "example@test.com");
  var service = new SpeakerService(_fakeRepository, _fakeGravatarService);

  // Act
  service.Get(expectedSpeaker.Id);

  // Assert
  Assert.True(_fakeGravatarService.WithEmailCalled);
  Assert.Equal(expectedSpeaker.EmailAddress, _fakeGravatarService.CalledWith);
}
```

你需要修改 `SpeakerFactory` 以接受电子邮件地址，并将 `Speaker` 模型类修改为包含电子邮件地址属性。

修改 `FakeGravatarService` 和 `IGravatarService` 接口中的 `GetGravatar` 方法以接受一个字符串 `emailAddress`。确保在执行 `GetGravatar` 时设置 `CalledWith` 属性：

```cs
public string CalledWith { get; set; }

public void GetGravatar(string emailAddress)
{
  GetGravatarCalled = true;
  CalledWith = emailAddress;
}
```

确保使用演讲者的电子邮件地址调用 `GetGravatar` 方法：

```cs
public Models.SpeakerDetail Get(int id)
{
  var speaker = _repository.Get(id);

  if (speaker == null || speaker.IsDeleted)
  {
    throw new SpeakerNotFoundException();
  }

  var gravatar = _gravatarService.GetGravatar(speaker.EmailAddress);

  return new Models.SpeakerDetail
  {
    Id = speaker.Id,
    Name = speaker.Name,
    Location = speaker.Location,
  };
}
```

最后，将 `GetGravatar` 方法的返回值设置到 `SpeakerDetail` 对象的新属性 `Gravatar` 上：

```cs
[Fact]
public void GivenGravatarServiceThenItSetsGravatar()
{
  // Arrange
  var expectedSpeaker = SpeakerFactory.Create(_fakeRepository);
  var service = new SpeakerService(_fakeRepository, 
   _fakeGravatarService);

  // Act
  var actualSpeaker = service.Get(expectedSpeaker.Id);
  var expectedGravatar = 
   _fakeGravatarService.GetGravatar(expectedSpeaker.EmailAddress);

  // Assert
  Assert.True(_fakeGravatarService.WithEmailCalled);
  Assert.Equal(expectedSpeaker.Id, actualSpeaker.Id);
  Assert.Equal(expectedSpeaker.Name, actualSpeaker.Name);
  Assert.Equal(expectedGravatar, actualSpeaker.Gravatar);
}
```

你需要修改 `FakeGravatarService` 及其接口，以及 `SpeakerService` 的 `Get` 方法以返回一个字符串，并在 `SpeakerDetail` 类中添加一个 `Gravatar` 属性：

```cs
public string GetGravatar(string emailAddress)
{
  WithEmailCalled = true;
  CalledWith = emailAddress;

  return System.Reflection.MethodBase.GetCurrentMethod().Name;
}
```

`GetGravatar` 方法的返回值不重要，只要它是一个已知的值。记住，你并不是在测试 `FakeGravatarService` 是否返回一个有效的 Gravatar 图像 URL，只是测试该方法返回了某个值，并且返回值被设置到 `SpeakerDetail` 对象的 `Gravatar` 属性上。

# 实现接口的生产版本

到目前为止，已经创建了一个包含一个方法 `GetGravatar` 的 `IGravatarService` 接口。有几种选项可以用来与 Gravatar 交互。你可以选择编写自己的方法来直接与其公共 API 通信。Speaker Meet 应用程序使用可用的 `NuGet` 包之一，`GravatarHelper.NetStandard`。

通过 `NuGet` 安装 `GravatarHelper.NetStandard` 的最新版本以便跟随操作。

在审查 Gravatar 网站时，看起来他们提供了各种可选参数。为了扩展 `IGravatarService` 接口及其实现，创建一个新的测试类 `GetGravatar`：

```cs
public class GetGravatar
{
}
```

现在测试 `GravatarService` 是否存在：

```cs
[Fact]
public void ItExists()
{
  var gravatarService = new GravatarService();
}
```

通过在 `SpeakerService` 相同的位置创建一个 `GravatarService` 类来使这个测试通过：

```cs
namespace SpeakerMeet.Api.Services
{
  public class GravatarService
  {}
}
```

现在，确保 `GravatarService` 实现了 `IGravatarInterface`：

```cs
[Fact]
public void ItImplementsIGravatarInterface()
{
  // Arrange
  // Act
  var gravatarService = new GravatarService();

  // Assert
  Assert.IsAssignableFrom<IGravatarService>(gravatarService);
}
```

从之前的一组测试中，已经在接口中定义了一个 `GetGravatar` 方法。通过实现接口来使测试通过：

```cs
public class GravatarService : IGravatarService
{
  public string GetGravatar(string emailAddress)
  {
    throw new System.NotImplementedException();
  }
}
```

通过一个新的测试来验证 `GetGravatar` 方法是否存在：

```cs
[Fact]
public void ItHasGetGravatarMethod()
{
  // Arrange
  IGravatarService gravatarService = new GravatarService();

  // Act
  gravatarService.GetGravatar("example@test.com");
}
```

通过返回一个空字符串来允许这个测试通过：

```cs
public string GetGravatar(string emailAddress)
{
  return string.Empty;
}
```

以下测试被分类为 *集成测试*，因为它们正在测试 Speaker Meet 应用程序如何与第三方系统交互。将类装饰为这样的形式：

```cs
[Trait("Category", "Integration")]
public class GetGravatar
```

许多测试运行器允许你根据特征类别有条件地运行或排除这些测试。一旦定义了集成测试并且已知它们可以成功运行，你可以在更改时忽略或禁用它们，或者只在提交前运行它们。

现在，测试当提供电子邮件地址时，Gravatar 服务返回一个已知值。如果你有一个 Gravatar 账户，请随意提供你自己的电子邮件地址并测试你的 Gravatar URL：

```cs
[Fact]
public void GivenEmailAddressThenGravatarReturned()
{
  // Arrange
  IGravatarService gravatarService = new GravatarService();

  // Act
  var actual = gravatarService.GetGravatar("example@test.com");

  // Assert
  Assert.Equal("http://www.gravatar.com/avatar/29e3f53ee49fae541ee0f48fb712c231", actual);
}
```

现在，通过调用`GravatarHelper`提供的静态方法来使这个测试通过：

```cs
public string GetGravatar(string emailAddress)
{
  return Gravatar.GetGravatarImageUrl(emailAddress);
}
```

测试现在应该通过了。你可以看到实现是如何从应用程序的其他部分隐藏起来的。界面是通过在`SpeakerService`中进行一系列测试而出于必要性设计的。

那么，为什么不直接从`SpeakerService`和其他地方调用`GravatarHelper`方法呢？记住，你不应该依赖于第三方实现。如果`GravatarHelper`被更改或被完全替换，那么任何直接调用它的类可能都需要更改。通过使用接口和外观，唯一可能需要更改的类是`GravatarService`。

# 未来的规划

未来的规划可能不好。如果你现在编写代码是为了预期未来的问题，你可能会浪费努力。不要编写你不需要的代码。这可能会增加复杂性，从而减慢开发速度。

记住术语**YAGNI**（**你不需要它**），这适用于任何没有立即需求的代码。之前添加的`GravatarService`方法可以用作那个例子的说明。到目前为止提供的示例中，没有一个需要刚刚创建的额外方法。如果由于某种原因`GravatarHelper`的实现发生变化，那么已经编写的代码可能需要更改。如果它目前没有被使用，这将是徒劳的努力。

那么，未来的规划从哪里开始，好的抽象在哪里结束？抽象第三方系统。仅暴露立即需要的方法和功能。通过屏蔽应用程序的其他部分不受任何第三方系统细节的影响来最小化更改的痛苦。这包括.NET 框架和诸如 Entity Framework 之类的 ORM。

# 抽象数据层

数据层抽象已经通过实现存储库模式开始了。在本节中，我们将努力创建一个有效的抽象，以便连接到 Entity Framework。在我们能够与 Entity Framework 通信之后，我们将专注于使存储库更加通用，能够与多个数据模型一起工作。

# 扩展存储库模式

创建有效数据层抽象的第一步是确保 CRUD（**创建**、**读取**、**更新**和**删除**）已经被处理。**CRUD**是可以在任何数据集上执行的基本操作。`IRepository`尚未提供对所有这些功能的访问，因此我们将从扩展它开始。

首先创建一个文件夹来包含`SpeakerRepository`的测试。这个文件夹应该与包含`SpeakerService`测试和`SpeakerController`测试的文件夹命名一致。像往常一样，我们从失败的测试开始。在这种情况下，测试失败是因为无法编译：

```cs
[Trait("Category", "SpeakerRepository")]
public class Class
{
  [Fact]
  public void ItExists()
  {
    var repo = new SpeakerRepository();
  }
}
```

创建`SpeakerRepository`，测试应该通过：

```cs
public class SpeakerRepository
{
}
```

`SpeakerRepository`需要继承自`IRepository`，因此要么将存在性测试转换为类型测试，要么创建一个新的测试：

```cs
[Fact]
public void ItIsARepository()
{
  // Arrange / Act
  var repo = new SpeakerRepository();

  // Assert
  Assert.IsAssignableFrom<IRepository<Speaker>>(repo);
}
```

现在，通过继承自`IRepository`使测试通过。目前我们没有针对功能性的测试，所以将仓库方法留为未实现：

```cs
public class SpeakerRepository : IRepository<Speaker>
{
  public Speaker Get(int id)
  {
    throw new System.NotImplementedException();
  }

  public IQueryable<Speaker> GetAll()
  {
    throw new System.NotImplementedException();
  }
}
```

# 获取方法

现在`SpeakerRepository`正确地继承自`IRepository`，目前由`IRepository`定义的两个方法需要实现。就像我们对`SpeakerService`所做的那样，我们需要为`Get`方法创建一个新的测试类。再次强调，虽然在这个阶段可能看起来有些过度，但为每个方法创建一个文件将有助于随着测试套件的扩展进行组织：

```cs
[Trait("Category", "SpeakerRepository")]
public class Get
{
}
```

现在类已经存在，可以编写的第一个测试方法是简单的存在方法：

```cs
[Fact]
public void ItHasGetMethod()
{
  // Arrange
  var repo = new SpeakerRepository();

  // Act
  var result = repo.Get(0);
}
```

初始时，这个测试将失败，因为 Visual Studio 提供的占位符实现只是抛出`NotImplementedException`。为了修复这个问题，我们必须返回一些东西；那么应该返回什么呢？没有测试来解释期望的结果，所以我们必须选择一个可以编译但几乎肯定是不正确的值。在这种情况下，正确的、不正确的返回值可能是`null`：

```cs
public Speaker Get(int id)
{
  return null;
}
```

# 获取所有方法

测试现在通过了。让我们在这里暂停一下，并围绕接口强制执行的`GetAll`方法进行相同数量的测试。像之前一样，为`GetAll`创建一个新的类：

```cs
[Trait("Category", "SpeakerRepository")]
public class GetAll
{
}
```

创建一个存在方法，它将确保当被调用时该方法不会抛出异常：

```cs
[Fact]
public void ItHasGetAllMethod()
{
  // Arrange  
  var repo = new SpeakerRepository();

  // Act
  var result = repo.GetAll();
}
```

# 创建方法

如果假设所有仓库方法都存在，测试仓库模式可能会更容易。不幸的是，仓库呈现了一个鸡生蛋的问题。如果没有在仓库中创建条目的方法，我们如何测试`Get`或`GetAll`？同时，如果没有从仓库检索条目的方法，我们如何测试`Create`或`Delete`？

接下来创建一个新的类用于`Create`：

```cs
[Trait("Category", "SpeakerRepository")]
public class Create
{
}
```

如前所述，编写一个检查存在性的方法。在这个测试中，我们需要确保测试的是仓库，而不是`Create`类的实现。这将迫使我们添加到接口中：

```cs
[Fact]
public void ItHasCreateMethod()
{
  // Arrange
  IRepository<Speaker> repo = new SpeakerRepository();

  // Act
  var result = repo.Create(new Speaker());
}
```

你可能已经注意到，在这个测试中，我们从`Create`方法中得到了一个结果。这可能不是很明显，因为我们从`Get`和`GetAll`方法中得到了值，但我们选择打破**CQRS**（**命令查询责任分离**）以支持更 RESTful 的方法。在**REST**（**表征状态转移**）中，因为它必须保持无状态并且不能提供关于已经完成的操作的信息，所以服务通常返回创建的对象或将来检索该对象的方式。

在这种情况下，你可能会认为我们现在提供了一个有漏洞的抽象的 Web。它可以这样解释。我更喜欢将这个选择视为打开选项而不是限制它们。在隐藏 CQRS 实现背后比反过来工作要容易得多。

现在，为了通过当前失败的测试，需要在`IRepository`接口中添加一个方法定义，在`SpeakerRepository`中添加一个方法实现，并且需要修改`SpeakerRepository`的实现以避免抛出异常：

```cs
public interface IRepository<T>
{
  T Get(int id);
  IQueryable<T> GetAll();
  T Create(T item);
}
```

`Speaker Save`方法：

```cs
public Speaker Save(Speaker speaker)
{
 return null;
}
```

还需要在定义在第七章，*测试驱动 C#应用程序*中的`FakeRepository`中添加一个存根实现。

# 删除方法

接下来，我们将添加`Delete`方法。就像之前一样，创建一个新的测试类：

```cs
[Trait("Category","SpeakerRepository")]
public class Delete
{
}
```

就像对于`Create`方法一样，我们需要将`SpeakerRepository`视为`IRepository`。创建一个`Delete`存在的方法：

```cs
[Fact]
public void ItHasDeleteMethod()
{
  // Arrange
  IRepository<Speaker> repo = new SpeakerRepository();
  var speaker = new Speaker();

  // Act
  repo.Delete(speaker);
}
```

实现这个方法将会稍微容易一些，因为它是一个无返回值的方法，我们并不期望得到结果。关于当方法传递给一个不存在的演讲者时应该如何表现，我们将在稍后做出一些决定。现在，我们可以假设什么也不发生，方法执行成功。

就像之前一样，修改`IRepository`以包含一个`Delete`方法：

```cs
public interface IRepository<T>
{
  T Get(int id);
  IQueryable<T> GetAll();
  T Create(T item);
  void Delete(T item);
}
```

现在在`SpeakerRepository`和`FakeRepository`中创建一个存根方法：

```cs
public void Delete(Speaker speaker)
{
}
```

# 更新方法

对于一个有效的仓储模式和任何有用的系统来说，还需要一个最后的方法。我们需要能够更新我们正在工作的模型。与最后两个方法一样，这个方法在仓储中还不存在，所以让我们添加它。

就像之前一样，首先为它创建一个测试类：

```cs
[Trait("Category", "SpeakerRepository")]
public class Update
{
}
```

就像其他方法一样，创建一个引用`IRepository`的`ItExists`测试：

```cs
[Fact]
public void ItHasUpdateMethod()
{
  // Arrange
  IRepository<Speaker> repo = new SpeakerRepository();
  var speaker = new Speaker();

  // Act
  var result = repo.Update(speaker);
}
```

就像之前一样，我们在这里只是存根功能，所以我们还没有实际的演讲者去更新。这个测试，以及其他测试，几乎肯定会在我们开始测试实际功能时发生变化。现在，它们足以确保接口和类有适当的方法。

就像之前一样，将方法添加到接口中，然后添加到`SpeakerRepository`和`FakeRepository`中：

```cs
public interface IRepository
{
  TGet(int id);
  IQueryable<T> GetAll();
  T Create(T item);
  T Update(T item);
  void Delete(T item);
}
```

`Speaker Update`方法：

```cs
public Speaker Update(Speaker speaker)
{
  return null;
}
```

# 确保功能

现在所有的方法都已经定义好了，我们可以开始编写功能测试了。我们将从`Create`开始，然后逐步进行到`Delete`。

# 创建一个演讲者

之前提到的“鸡生蛋”场景让我们陷入了困境。如果没有创建演讲者，我们就无法从存储库中读取演讲者。除非我们能从存储库中检索到一个演讲者，否则我们也无法验证一个演讲者实际上已经被创建。

解决这个问题的方法之一是使用一种特殊的测试双胞胎，它暴露了类的内部功能，以便对相关信息进行断言。对于`Create`，我们将使用这种方法。在`Create.cs`文件中，让我们添加一个假设可测试类已经存在的测试：

```cs
[Fact]
public void ItAddsASpeakerToTheRepository()
{
  // Arrange
  var repo = new TestableSpeakerRepository();

  // Act
  var result = repo.Create(new Speaker());

  // Assert
  Assert.Equal(1, repo.SpeakersCollection.Count);
}
```

为了解决编译错误，必须创建可测试的类。现在，为了更有效地与之一起工作，在同一文件中创建这个类：

```cs
public class TestableSpeakerRepository : SpeakerRepository
{
}
```

现在类已经创建，初始的编译错误已经解决，但一个新的错误出现了：

```cs
// Assert
Assert.Equal(1, repo.SpeakersCollection.Count);
```

这个错误稍微有点难以解决。实际上，我们希望在真实存储库中存在一个演讲者集合。然而，我们没有理由暴露这个集合。实际上并没有创建任何集合。现在，在这个测试中，我们正在询问是否存在一个集合。测试要求`TestableSpeakerRepository`存在一个集合，但我们知道我们还需要一个用于真实的`SpeakerRepository`。让我们扮演魔鬼的代言人，实际上做我们知道并不完全正确的事情：

```cs
public class TestableSpeakerRepository : SpeakerRepository
{
  public IQueryable<Speaker> SpeakersCollection { get; set; }
}
```

这个更改并没有完全使测试通过；在编写测试时，我们匆忙访问了`Count`属性。`Count`属性只在列表上，但为了限制接口的暴露，直到我们实际上可以用测试来要求它，我们实际上应该使用一个`IQueryable`。我们可以快速更新测试以反映这个选择：

```cs
// Assert
Assert.Equal(1, repo.SpeakersCollection.Count);
```

现在，执行测试，它最终以实际消息失败。解决这个失败的方法是在调用`Create`时向演讲者集合添加一个条目。问题是`Speakers`类与`Create`类不同。因此，我们必须在`SpeakerRepository`中也有一个集合：

```cs
protected readonly IList<Speaker> Speakers = new List<Speaker>();

public Speaker Create(Speaker speaker)
{
  Speakers.Add(speaker);

  return speaker;
}
```

需要注意的是`_speakers`的范围和类型。它是一个`IList`，因为我们需要向其中添加一个项目；一个`IQueryable`是不够的。它也是受保护的；`_speakers`必须对外部世界隐藏，但同时也必须可以从可测试的类中访问。提供这种功能的范围操作符是受保护的。

我们还必须在可测试的类中进行更改，以便使这个测试通过：

```cs
internal class TestableSpeakerRepository : SpeakerRepository
{
  public IList<Speaker> SpeakersCollection => Speakers;
}
```

继续使用`Create`，我们现在需要验证，当创建一个新的演讲者时，它将收到一个唯一的 ID：

```cs
[Fact]
public void ItAssignsUniqueIdsToEachSpeaker()
{
  // Arrange
  var repo = new TestableSpeakerRepository();

  // Act
  var speaker1 = repo.Create(new Speaker());
  var speaker2 = repo.Create(new Speaker());

  // Assert
  Assert.NotEqual(speaker1.Id, speaker2.Id);
}
```

为了使这个测试通过，我们必须想出某种 ID 生成系统。有很多选择，但其中最简单的一个是在每次调用`Create`时创建一个私有字段并递增其值：

```cs
private int _currentId = 0;

public Speaker Create(Speaker speaker)
{
  speaker.Id = ++_currentId;

  Speakers.Add(speaker);

  return speaker;
}
```

那个测试通过后，我们现在可以将注意力转向抽象中的漏洞。我们只是简单地将传入的对象放入数据集中。这可能会在需要修复的应用程序中引起问题。

仓库应该通过传递和存储克隆对象来将其对象与应用程序的其他部分隔离，而不是直接访问和提供对象：

```cs
[Fact]
public void ItReturnsANewSpeaker()
{
  // Arrange
  var repo = new TestableSpeakerRepository();
  var speaker = new Speaker { Id = 0 };

  // Act
  var result = repo.Create(speaker);

  // Assert
  Assert.Equal(0, speaker.Id);
}
```

要使这个测试通过，我们需要一些克隆机制。为了尽快使这个测试通过，我们可以简单地使用一个新的对象和对象初始化器：

```cs
public Speaker Create(Speaker speaker)
{
  var newSpeaker = new Speaker
  {
    Id = ++_currentId,
    Name = speaker.Name,
    Location = speaker.Location,
    IsDeleted = speaker.IsDeleted
  };

  Speakers.Add(newSpeaker);

  return newSpeaker;
}
```

现在，我们必须处理引用传递的另一个方向。存储在演讲者集合中的值不应该直接从 `Create` 方法返回给我们：

```cs
[Fact]
public void ItProtectsAgainstObjectChangesAfterCreation()
{
  // Arrange
  var repo = new TestableSpeakerRepository();
  var speaker = repo.Create(new Speaker());

  // Act
  speaker.Name = "test name";

  // Audit
  var result = repo.SpeakersCollection.First();

  // Assert
  Assert.NotEqual("test name", result.Name);
}
```

注意额外的审计步骤。有时，你可能需要采取一个行动，然后断言一个深层嵌套的值或一个遥远的值。在这种情况下，你可以通过添加审计步骤来保持一个干净的单一步骤行动。

要使这个测试通过，我们必须采取与我们在 `Create` 方法顶部已经做过的类似行动：

```cs
public Speaker Create(Speaker speaker)
{
  var newSpeaker = new Speaker
  {
    Id = ++_currentId,
    Name = speaker.Name,
    Location = speaker.Location,
    IsDeleted = speaker.IsDeleted
  };

  Speakers.Add(newSpeaker);

  var returnableSpeaker = new Speaker
  {
    Id = newSpeaker.Id,
    Name = newSpeaker.Name,
    Location = newSpeaker.Location,
    IsDeleted = newSpeaker.IsDeleted
  };

  return returnableSpeaker;
}
```

这就完成了 `Create` 方法所需的功能。现在我们真的应该做一些长期未完成的重构。首先，让我们专注于测试，并减少创建新仓库的重复调用。

创建一个构造函数和一个私有的 `repo` 字段：

```cs
private readonly TestableSpeakerRepository _repo;

public Create()
{
  _repo = new TestableSpeakerRepository();
}
```

然后在测试中将所有仓库引用替换为 `_repo`。在减少测试中创建的仓库数量后，测试看起来相当不错。现在我们可以专注于 `SpeakerRepository` 类。

在 `SpeakerRepository` 中，一个立即引人注目的是我们用来克隆演讲者的代码。相同的代码在同一个方法中基本上被输入了两次。让我们现在将其抽象为仓库内部的私有函数。我们可能最终会做出更复杂的解决方案，但就目前而言，这应该足够了。

在类的底部，在所有公共方法之后，我们可以创建一个 `CloneSpeaker` 方法：

```cs
private Speaker CloneSpeaker(Speaker speaker)
{
  return new Speaker
  {
    Id = speaker.Id,
    Name = speaker.Name,
    Location = speaker.Location,
    IsDeleted = speaker.IsDeleted
  };
}
```

然后我们在 `Create` 中使用 `CloneSpeaker` 方法：

```cs
public Speaker Create(Speaker speaker)
{
  var newSpeaker = CloneSpeaker(speaker);

  newSpeaker.Id = ++_currentId;

  Speakers.Add(newSpeaker);

  return CloneSpeaker(newSpeaker);
}
```

# 获取单个演讲者

随着 `Create` 的存在，我们现在可以非常容易地断言检索现有或不存在演讲者。根据鲍勃叔叔的转换优先级前提，测试单个项目比测试复数项目更容易、更简单，所以虽然这并不完全符合前提的意图，但我们将测试检索单个演讲者。

我们已经有了存在性测试，那么下一个测试会是什么？最简单的测试是检索单个演讲者，但如果我们试图避免黄金标准，最合适的测试将是检查演讲者不存在时会发生什么。

对于不存在的演讲者，我们有一些立即显而易见的选择。我们可以抛出一个错误，声明请求的演讲者不在系统中。另一个选项是返回一个 `null` 对象。最后一个选项是简单地返回 `null`。

抛出错误可能是最直接的选择，所以让我们首先考察这个选项：

```cs
[Fact]
public void ItThrowsWhenSpeakerIsNotFound()
{
  // Arrange
  var repo = new SpeakerRepository();

  // Act
  var result = Record.Exception(() => repo.Get(-1));

  // Assert
  Assert.IsType<SpeakerNotFoundException>(result.GetBaseException());
}
```

要使这个测试通过，首先我们必须让它编译。在这个情况下，`SpeakerNotFoundException`与我们正在使用的`SpeakerService`中的不同。因此，需要创建它：

```cs
public class SpeakerNotFoundException : Exception
{
  public SpeakerNotFoundException()
  {
  }
}
```

现在测试已经正确编译并失败，我们可以向仓库添加适当的代码以使测试通过：

```cs
public Speaker Get(int id)
{
  if (id == -1)
  {
    throw new SpeakerNotFoundException();
  }

  return null;
}
```

另一个测试的指导原则，帮助你了解你是否走在正确的道路上，而不是挖坑，再次来自 Uncle Bob，*A*随着测试变得更加具体，代码变得更加通用。如果我们看看我们刚刚编写的代码，它似乎更具体而不是更通用。让我们重构它，以保持向通用性的趋势：

```cs
public Speaker Get(int id)
{
  if (id > -1)
  {
    return null;
  }

  throw new SpeakerNotFoundException();
}
```

这里的变化很微妙，但很重要。考虑这个方法在生产环境中的操作，默认情况确实是抛出异常。在包含所有整数的集合中，我们实际上有一个演讲者的情况只是一个很小的子集，所以通用情况是抛出异常。具体情况实际上是找到一个演讲者。

当找不到演讲者时抛出错误的问题在于，它给应用程序流程带来了可能意外和突然的结束。应用程序到达这个方法的整个逻辑路径现在被破坏，并且必须处理异常。即使我们处理了异常，C#在第一次异常错误处理过程中也会使用额外的 CPU 周期。有时抛出确实是正确的决定；然而，异常应该保留给真正异常的事件。如前所述，`Get`被一个无效的 ID 调用比被一个有效的 ID 调用的可能性要大得多。所以，在这种情况下，请求一个无效的演讲者并不一定是一个适当的异常事件。

让我们探索空对象模式的替代方案。首先，我们需要将我们的代码回滚到我们开始工作在`Get`方法时的状态。我们使用源代码控制，因此我们可以非常简单地回滚我们的代码。如果你没有使用源代码控制，我建议你开始使用。以下是我们在再次开始之前`Get`方法应该处于的状态：

```cs
public Speaker Get(int id)
{
  return null;
}
```

我们可以直接删除断言抛出异常的测试。

空对象模式是一个简单的模式，但其实现稍微复杂一些。基本上，你创建一个从所需类继承的对象，但以完全正确的方式什么都不做。

从我们最终网站使用的角度来看，一个空对象演讲者可能有一个像“Mr. Unknown”这样的名字，并在像“Mid-Nowhere Tech Fest”这样的会议上发表演讲。我们可以创建一个有趣的个人资料图片，并填写无害的用户信息，让用户知道他们请求了一个不存在的演讲者。所有这些信息都可以在之后确定并填写。空对象的重要部分是它代表一个完全有能力的对象，但以正确的方式什么都不做，不会对你的应用程序造成伤害：

```cs
[Fact]
public void ItReturnsANullSpeakerWhenNotFound()
{
  // Arrange
  var repo = new SpeakerRepository();

  // Act
  var result = repo.Get(-1);

  // Assert
  Assert.IsType<NullSpeaker>(result);
}
```

为了修复编译错误，创建一个`NullSpeaker`类，并让它从`Speaker`类继承：

```cs
public class NullSpeaker : Speaker
{
}
```

使测试通过相当简单。在这种情况下，我们不必担心现有测试可能会因为返回空对象而中断：

```cs
public Speaker Get(int id)
{
  return new NullSpeaker();
}
```

表面上看，空对象模式似乎是一个相当好的解决方案。实际上，这并不总是如此。在系统中正确地什么都不做是非常困难的。在获取扬声器的情况下，空对象模式可能会工作得很好。但我们还有一个选项要探索。

最后一个选项是简单地返回`null`。好吧，简单可能不是最好的词。`Null`可能会在系统中引起很多麻烦。如果你没有处理接收到的`null`，而系统的遥远部分试图将其作为非`null`使用，它将造成混乱。早些时候，当我们设计服务时，我们决定期望从调用存储库返回`null`作为可能的结果；因此，在这种情况下，影响范围应该是有限的，`null`不应该在系统中引起重大的负面影响。让我们再次回退代码，并探索简单地返回`null`：

```cs
[Fact]
public void ItReturnsNullWhenNotFound()
{
  // Arrange
  var repo = new SpeakerRepository();

  // Act
  var result = repo.Get(-1);

  // Assert
  Assert.Null(result);
}
```

在`SpeakerRepository`中没有事情要做，因为我们已经返回了`null`。通常，这个测试不会在这里编写。我们会等到有一个返回有效扬声器的测试时再写这个测试。我们想要推迟这个测试的原因是我们不能让它失败。你不会看到这个测试变红，这通常是一个问题。要么这个测试是不必要的，因为它是多余的，在其他地方已经覆盖了，要么这个测试是有缺陷的，因为它不能失败。

在这个测试之前，我们应该写一个应该检索有效值的测试。目前，忽略这个测试，我们将在下一个测试之后回来观察它失败：

```cs
[Fact(Skip = "Can't fail")]
public void ItReturnsNullWhenNotFound()
{
  // Arrange
  var repo = new SpeakerRepository();

  // Act
  var result = repo.Get(-1);

  // Assert
  Assert.Null(result);
}
```

那么，如果我们即将测试一个现有扬声器的检索，之前我们称之为“黄金标准”，为什么我们最初要避免它呢？我们现在要首先测试黄金标准的原因纯粹是因为实现选择。如果我们打算将其用作异常或空对象，我们早就已经那样做了。现在的问题是我们的代码已经返回了`null`，所以测试空对象对我们没有任何好处。

要测试一个现有的扬声器，首先必须存在一个扬声器。在我们的测试安排部分，我们需要确保我们创建了一个扬声器：

```cs
[Fact]
public void ItReturnsASpeakerWhenFound()
{
  // Arrange
  var repo = new SpeakerRepository();
  var speaker = repo.Create(new Speaker {Name = "Test Speaker"});

  // Act
  var result = repo.Get(speaker.Id);

  // Assert
  Assert.NotNull(result);
}
```

第一步是简单地断言结果不是`null`。当然，这失败了，因为我们现在所做的只是返回`null`。让我们来修复它：

```cs
public Speaker Get(int id)
{
  return new Speaker();
}
```

尽管我们现在被迫走黄金标准路线，但我们仍然可以尽可能地避免它。一种方法是通过扮演魔鬼的代言人，我们可以通过返回一个扬声器，但不是正确的扬声器来实现这一点。

我们现在必须修改测试，以确保我们关心的值是正确的：

```cs
// Assert
Assert.NotNull(result);
Assert.Equal("Test Speaker", result.Name);
```

现在来修改代码。我们可以继续扮演魔鬼的代言人，但在这个阶段，那只会导致测试中更多不必要的劳动：

```cs
public Speaker Get(int id)
{
  return Speakers.SingleOrDefault(s => s.Id == id);
}
```

到目前为止，我们面临着一个难题。我们不能调用`Single`或`First`，因为如果找不到请求的演讲者，它们会抛出异常。尽管如此，我们确实有一个系统规则，当找不到演讲者时返回`null`。系统默认行为是正确的。我们可以简单地重新启用我们的测试，并接受空测试无法正确验证。另一个选择是有意编写代码，在演讲者集合缺少请求的演讲者时返回一个新的演讲者。选择第二个选项将允许我们看到空测试失败，但我们只是移除它以使测试通过。

在这个情况下，第二个选项可能是最合适的。你们中很多人可能认为没有必要添加强制非空的代码；还有很多人可能认为不需要测试，因为它似乎没有提供价值。这两组人同时是正确的和错误的。他们是正确的：只是为了在两秒后删除而放入代码是愚蠢的。添加一个无法真正失败的测试也是愚蠢的。

然而，我们需要这个测试，因为在未来的某个时刻，可能会出现一个要求，导致空值意外地变得不可能，我们需要记录业务需求，说明该值应该是空的。我们还需要看到每个测试都失败。在这个特定的情况下，我们可能可以避开这部分，但如果我们养成了跳过测试失败的习惯，很快就会让你付出代价。

因此，让我们做出适当的更改，强制空测试失败，然后修复我们的“错误：”

```cs
public Speaker Get(int id)
{
  return Speakers.SingleOrDefault(s => s.Id == id) ?? new Speaker();
}
```

现在，我们可以移除空测试上的跳过属性，然后回到这里移除空合并运算符。

我们还需要一个针对`Get`的测试。我们必须确保返回的演讲者指针与存储库中的演讲者不同：

```cs
[Fact]
public void ItProtectsAgainstObjectChanges()
{
  // Arrange
  var repo = new SpeakerRepository();
  var speaker = repo.Create(new Speaker { Name = "Test Speaker" });
  var retrievedSpeaker = repo.Get(speaker.Id);
  retrievedSpeaker.Name = "New Speaker Name";

  // Act
  var result = repo.Get(speaker.Id);

  // Assert
  Assert.NotEqual(retrievedSpeaker.Name, result.Name);
}
```

这个测试很难理解，但却是必要的。我们首先必须创建一个演讲者。然后检索我们创建的演讲者，更新检索到的演讲者的名字，然后再检索演讲者。最后，我们验证修改后的演讲者与检索到的演讲者不同。它们不应该相同，因为我们没有保存数据，所以不应该发生任何更新。

这个问题的修复方案已经创建好了，只需要实施：

```cs
public Speaker Get(int id)
{
  var speaker = Speakers.SingleOrDefault(s => s.Id == id);

  if (speaker != null)
  {
    speaker = CloneSpeaker(speaker);
  }

  return speaker;
}
```

最后，我们应该重构。遵循你为创建和提取存储库创建所采取的相同步骤。这个测试类的简单性意味着它不需要任何进一步的重构。

# 获取多个演讲者

获取多个说话者比获取单个说话者容易得多。我们不必担心找不到说话者。我们也不必处理任何错误条件。我们只需测试的是检索正确的说话者数量，并确保数据安全性与我们迄今为止对其他方法所做的一样。

让我们从在没有说话者时检索所有说话者开始：

```cs
[Fact]
public void ItReturnsNoSpeakersWhenThereAreNoSpeakers()
{
  // Arrange
  var repo = new SpeakerRepository();

  // Act
  var result = repo.GetAll();

  // Assert
  Assert.NotNull(result);
}
```

这在仓库中是一个简单的修复：

```cs
public IQueryable<Speaker> GetAll()
{
  return new List<Speaker>().AsQueryable();
}
```

记住，我们是在扮演魔鬼的辩护者；这是其目的的正确回应。

现在，我们需要断言方法响应的类型和大小：

```cs
// Assert
Assert.NotNull(result);
Assert.IsAssignableFrom<IQueryable<Speaker>>(result);
Assert.Equal(0, result.Count());
```

我知道我们已经提到了单条断言规则以及它并不像听起来那样意味着什么。这个测试仍然遵循规则，因为我们正在断言`GetAll`返回一个非空空集合的说话者。

接下来，我们测试仓库中存在单个说话者的情况：

```cs
[Fact]
public void ItReturnsASingleSpeakerWhenOnlyOneSpeakerExists()
{
  // Arrange
  var repo = new SpeakerRepository();
  repo.Create(new Speaker { Name = "Test Speaker"});

  // Act
  var result = repo.GetAll();

  // Assert
  Assert.Equal(1, result.Count());
}
```

再次，在仓库中进行适当的调整相当简单：

```cs
public IQueryable<Speaker> GetAll()
{
  return Speakers.AsQueryable();
}
```

接下来，我们想要通过确保返回的说话者是我们创建的说话者来结束这一系列的测试。我们预计这会立即通过，因为我们已经知道这是正确的。这个断言对于确保仓库未来的完整性非常重要。

```cs
// Act
var result = repo.GetAll().ToList();

// Assert
Assert.Single(result);
Assert.Equal("Test Speaker", result.First().Name);
```

注意动作的变化。执行计数和检索集合中的第一个元素都会导致枚举。枚举等于 CPU 周期。为了减少测试时间和生产执行时间，我们想要减少枚举的数量。将`IQueryable`转换为列表将强制进行单个枚举。

让我们进行最后一次测试，以确保返回值正确，并检查多个说话者是否能够正确返回：

```cs
[Fact]
public void ItReturnsManySpeakersWhenManySpeakersExists()
{
  // Arrange
  var repo = new SpeakerRepository();
  repo.Create(new Speaker());
  repo.Create(new Speaker());
  repo.Create(new Speaker());

  // Act
  var result = repo.GetAll().ToList();

  // Assert
  Assert.Equal(3, result.Count);           
}
```

这个测试只是一个理智的检查，会立即通过。我们的下一个测试将检查数据完整性：

```cs
[Fact]
public void ItProtectsAgainstObjectChanges()
{
  // Arrange
  var repo = new SpeakerRepository();
  repo.Create(new Speaker {Name = "Test Name"});
  var speakers = repo.GetAll().ToList();
  speakers.First().Name = "New Name";

  // Act
  var result = repo.GetAll();

  // Assert
  Assert.NotEqual(speakers.First().Name, result.First().Name);
}
```

仓库中的更新与我们为其他方法所做的是相似的，只有一个例外。我们正在操作整个集合，而不仅仅是单个项目：

```cs
public IQueryable<Speaker> GetAll()
{
  return Speakers.Select(CloneSpeaker).AsQueryable();
}
```

一些同学可能之前见过这种语法。我们所做的是将`CloneSpeaker`方法的函数指针作为 linq 中`Select`方法所需的 Lambda 表达式传递。前面的行在功能上与下面的行完全相同：

```cs
public IQueryable<Speaker> GetAll()
{
  return Speakers.Select(CloneSpeaker).AsQueryable();
}
```

是时候清理了。我们与之前相同的清理步骤。唯一真正需要重构的是提取仓库创建。

# 更新说话者

我们现在可以创建和检索说话者。我们还确保了不能意外地更新它们。所以，让我们确保我们可以有意地更新它们：

```cs
[Fact]
public void ItUpdatesASpeaker()
{
  // Arrange
  var repo = new SpeakerRepository();
  var speaker = repo.Create(new Speaker {Name = "Test Name"});
  speaker.Name = "New Name";

  // Act
  var result = repo.Update(speaker);

  // Assert
  Assert.Equal(speaker.Name, result.Name);
}
```

再次扮演魔鬼的辩护者，这是对仓库的一个简单更新：

```cs
public Speaker Update(Speaker speaker)
{
  return speaker;
}
```

这显然是错误的解决方案；让我们写另一个更具体的测试：

```cs
[Fact]
public void ItUpdatesASpeakerInTheRepository()
{
  // Arrange
  var repo = new SpeakerRepository();
  var speaker = repo.Create(new Speaker {Name = "Test Name"});
  speaker.Name = "New Name";

  // Act
  var updatedSpeaker = repo.Update(speaker);

  // Audit
  var result = repo.Get(speaker.Id);

  // Assert
  Assert.Equal("New Name", result.Name);
}
```

使这个测试通过有点棘手，但不算太坏：

```cs
public Speaker Update(Speaker speaker)
{
  var oldSpeaker = Speakers.FirstOrDefault(s => s.Id == speaker.Id);
  var index = Speakers.IndexOf(oldSpeaker);
  Speakers[index] = speaker;           

  return speaker;
}
```

然而，这个更改导致存在测试失败。查看那个测试，它失败是有好理由的，并且根据我们对`Update`当前理解的工作方式，实际上不应该通过。让我们对那个测试进行一个小但重要的更新：

```cs
[Fact]
public void ItHasUpdateMethod()
{
  // Arrange
  IRepository<Speaker> repo = new SpeakerRepository();
  var speaker = repo.Create(new Speaker());

  // Act
  var result = repo.Update(speaker);
}
```

在我们的测试时间表上接下来的是处理由存在测试失败所突出显示的错误。如果有人请求更新一个不存在的演讲者，存储库会崩溃。这可能是一种所需的行为，但它应该通过一个信息性异常崩溃，而不是索引越界异常：

```cs
[Fact]
public void ItThrowsNotFoundExceptionWhenSpeakerDoesNotExist()
{
  // Arrange
  var repo = new SpeakerRepository();
  var speaker = new Speaker {Id = 5, Name = "Test Name"};

  // Act
  var result = Record.Exception(() => repo.Update(speaker));

  // Assert
  Assert.IsAssignableFrom<SpeakerNotFoundException>(result.GetBaseException());
}
```

系统中已经存在一个`SpeakerNotFoundException`，但它来自不同的层，因此我们需要创建一个新的异常：

```cs
public class SpeakerNotFoundException : Exception
{
  public SpeakerNotFoundException(int id) : base($"Speaker {id} not found.")
  {
  }
}
```

作为一项练习，看看你是否可以用测试来覆盖这个异常。能够在后续章节中讨论的实际情况之后反向工作并编写测试，是一项非常有价值的技能。

现在，让我们继续并使这个测试通过：

```cs
public Speaker Update(Speaker speaker)
{
  var oldSpeaker = Speakers.FirstOrDefault(s => s.Id == speaker.Id);
  var index = Speakers.IndexOf(oldSpeaker);

  if (index == -1)
  {
    throw new SpeakerNotFoundException(speaker.Id);
  }

  Speakers[index] = speaker;

  return speaker;
}
```

就像其他测试一样，我们现在必须确保数据完整性：

```cs
[Fact]
public void ItProtectsAgainstObjectChanges()
{
  // Arrange
  var repo = new SpeakerRepository();
  var speaker = repo.Create(new Speaker {Name = "Test Name"});
  speaker.Name = "New Name";
  var updatedSpeaker = repo.Update(speaker);

  // Act
  updatedSpeaker.Name = "Updated Name";

  // Audit
  var result = repo.Get(updatedSpeaker.Id);

  // Assert
  Assert.NotEqual("Updated Name", result.Name);
}
```

我们正接近测试复杂度过高的边缘。如果这些测试变得更加复杂，我们将想要考虑重新思考我们的方法。这个测试证实了我们的想法：返回的演讲者没有受到更改的保护。让我们修复这个问题：

```cs
public Speaker Update(Speaker speaker)
{
  var oldSpeaker = Speakers.FirstOrDefault(s => s.Id == speaker.Id);
  var index = Speakers.IndexOf(oldSpeaker);

  if (index == -1)
  {
    throw new SpeakerNotFoundException(speaker.Id);
  }

  Speakers[index] = speaker;

  return CloneSpeaker(speaker);
}
```

一个相当简单的解决方案：只需将返回值用`CloneSpeaker`调用包裹起来。你是否看到了另一个潜在的数据完整性问题？我们正在做些什么来保护传递给演讲者的演讲者免受进一步更改？让我们编写一个测试来确保对传递给演讲者的演讲者所做的任何事后更改都不会影响存储库：

```cs
[Fact]
public void ItProtectsAgainstOriginalObjectChanges()
{
  // Arrange
  var repo = new SpeakerRepository();
  var speaker = repo.Create(new Speaker { Name = "Test Name" });
  speaker.Name = "New Name";
  var updatedSpeaker = repo.Update(speaker);

  // Act
  speaker.Name = "Updated Name";

  // Audit
  var result = repo.Get(updatedSpeaker.Id);

  // Assert
  Assert.NotEqual("Updated Name", result.Name);
}
```

这个测试看起来几乎与上一个测试相同。唯一的重大变化是操作。我们不是在`updatedSpeaker`上更改值，而是在原始演讲者上更改值。为了使这个测试通过，对存储库的更改与之前的修复类似：

```cs
public Speaker Update(Speaker speaker)
{
  var oldSpeaker = Speakers.FirstOrDefault(s => s.Id == speaker.Id);
  var index = Speakers.IndexOf(oldSpeaker);

  if (index == -1)
  {
    throw new SpeakerNotFoundException(speaker.Id);
  }

  Speakers[index] = CloneSpeaker(speaker);

  return CloneSpeaker(speaker);
}
```

剩下要重构和清理的只有测试和存储库。

# 删除演讲者

我们终于到达了这个存储库中的最后一个方法。对于演讲者会议，我们并不真的想删除演讲者；我们可能会将演讲者标记为已删除，但我们并不真的想从数据集中删除他们。因此，我们的`Delete`方法将更像是一个更新。

对于`Delete`，我们没有任何奇怪的行为或约束，因此我们应该从失败情况开始。如果我们请求删除一个不存在的用户，会发生什么？我们应该抛出异常吗？在这种情况下，我们实际上可以假设工作已经完成。如果没有具有给定 ID 的演讲者，那么该演讲者的删除可以被认为是成功的。

好吧，现在我们已经考虑了失败情况，我们看到它确实有特殊的行为，这会导致失败情况直接通过。让我们看看成功情况，看看它是否会导致测试失败：

```cs
[Fact]
public void ItMarksTheGivenSpeakerAsDeleted()
{
  // Arrange
  var repo = new SpeakerRepository();
  var speaker = repo.Create(new Speaker {Name = "Test Name"});

  // Act
  repo.Delete(speaker);

  // Audit
  var result = repo.Get(speaker.Id);

  // Assert
  Assert.True(result.IsDeleted);
}
```

使这个测试通过应该相当简单：我们只需在将`isDeleted`标志更改为`true`后调用本地更新方法：

```cs
public void Delete(Speaker speaker)
{
  speaker.IsDeleted = true;

  Update(speaker);
}
```

不要忘记修复由于添加此代码而失败的任何其他测试，在确认失败是有效原因之后。如果我们编写更多测试时测试失败，我们需要检查需求并确保它们之间没有冲突。我们绝不想无意中破坏现有的需求。

现在，我们必须处理如果演讲者不存在于上下文中的情况。如之前决定的那样，我们对此请求的失败可以忽略，因为删除一个不存在的演讲者会导致与成功删除一个现有演讲者相同的情况：

```cs
[Fact]
public void ItDoesNothingWhenDeletingANonexistingSpeaker()
{
  // Arrange
  var repo = new SpeakerRepository();
  var speaker = new Speaker();

  // Act
  var result = Record.Exception(() => repo.Delete(speaker));

  // Assert
  Assert.Null(result);
}
```

为了通过这个测试，我们必须做一些通常不推荐或不首选的事情。我们必须吞下异常。在采取这样的步骤并忽略异常之前，确保你只忽略特定的异常，并确保你已经彻底思考过：

```cs
public void Delete(Speaker speaker)
{
  speaker.IsDeleted = true;

  try
  {
    Update(speaker);
  }
  catch (SpeakerNotFoundException ex)
  {
    // We can assume non-existing speakers are deleted
  }
}
```

对于这个方法的最后一个测试，我们必须确保我们没有意外地污染传入的对象：

```cs
[Fact]
public void ItProtectsAgainstPassedObjectChanges()
{
  // Arrange
  var repo = new SpeakerRepository();
  var speaker = repo.Create(new Speaker {Name = "Test Name"});

  // Act
  repo.Delete(speaker);

  // Assert
  Assert.False(speaker.IsDeleted);
}
```

我们可以使用我们一直在使用的相同方法来确保数据完整性：

```cs
speaker = CloneSpeaker(speaker);
speaker.IsDeleted = true;
```

# 泛型化仓库

这个仓库很棒，但我们真的希望为每个需要从数据源检索的数据模型重复这个逻辑和所有这些测试吗？答案是：不；如果你发现自己作为开发者反复做同样的事情，你就是在做错事。

那么，我们如何保护自己免受这种繁琐的困扰？

一种方法是用泛型。让我们重构 `SpeakerRepository` 以使用泛型，这还将涉及重构许多测试，使它们适用于 `GenericRepository` 而不是具体的 `SpeakerRepository`。

# 第一步 – 抽象接口

在 `IRepository` 中，我们每处使用 `Speaker` 都需要用 C# 泛型来替换：

```cs
public interface IRepository<T>
{
  T Create(T item);
  T Get(int id);
  IQueryable<T> GetAll();
  T Update(T item);
  void Delete(T item);
}
```

这个更改将导致 `SpeakerRepository` 中断，我们需要修复。目前，我们正在追逐编译器，依赖它告诉我们我们打破了什么。一旦测试再次通过，我们就知道我们又回到了正轨：

```cs
public class SpeakerRepository : IRepository<Speaker>
```

现在，我们必须在 SpeakerService 中做一些更改，因为它不再能够引用简单的 IRepository，而是需要一个 `IRepository<Speaker>`：

```cs
public class SpeakerService : ISpeakerService
{
  private readonly IRepository<Speaker> _repository;

  public SpeakerService(IRepository<Speaker> repository)
  {
    _repository = repository;
  }
 …
```

我们必须更新模拟仓库以使用正确的泛型仓库类型。

`public class FakeRepository : IRepository<Speaker>`

最后，我们有一些需要更新的测试。没有秘密代码；只需将 `IRepository` 更改为 `IRepository<Speaker>`。

# 第二步 – 抽象具体类

现在接口已经是泛型了，我们可以开始处理 `SpeakerRepository`。首先，让我们将其重命名为 `InMemorySpeakerRepository`。现在，我们想要开始使用泛型。创建一个新的类，`InMemoryRepository<T>`，并让演讲者仓库继承自它：

```cs
public abstract class InMemoryRepository<T> : IRepository<T>
{
  public abstract T Create(T speaker);
  public abstract T Get(int id);
  public abstract IQueryable<T> GetAll();
  public abstract T Update(T speaker);
  public abstract void Delete(T speaker);
}
```

为了缓慢移动并尽可能多地通过测试，我们使用抽象的，并将不得不让演讲者仓库中的每个方法覆盖基类方法。这使我们能够将每个方法单独移动到抽象中，而不是一次尝试解决整个问题。

从内存仓库继承，并为每个继承的方法添加覆盖：

```cs
public class InMemorySpeakerRepository : InMemoryRepository<Speaker>
public overrideSpeaker Create(Speaker speaker)
public override Speaker Get(int id)
public override IQueryable<Speaker> GetAll()
public override Speaker Update(Speaker speaker)
public override void Delete(Speaker speaker)
```

# 将`Create`转换为通用方法

我们将开始我们的通用方法之旅，从`Create`开始。第一步是将`Create`方法的内容从演讲者仓库复制到通用仓库。为了做到这一点，我们必须将抽象的`Create`方法更改为虚拟的`Create`方法：

```cs
public virtual T Create(T speaker)
{
  var newSpeaker = CloneSpeaker(speaker);
  newSpeaker.Id = ++CurrentId;
  Speakers.Add(newSpeaker);

  return CloneSpeaker(newSpeaker);
}
```

立刻，我们遇到了麻烦。我们没有通用的克隆方法。现在，我们先伪造它，直到我们找到真正的解决方案。

在通用仓库中添加以下受保护的抽象方法：

```cs
protected abstract T CloneEntity(T entity);
```

现在，我们可以在从演讲者仓库的`Create`方法复制的代码中做出类似的变化：

```cs
protected readonly IList<T> Entities = new List<T>();
protected int CurrentId;

public virtual T Create(T entity)
{
  var newSpeaker = CloneEntity(entity);
  newSpeaker.Id = ++CurrentId;
  Entities.Add(newSpeaker);

  return CloneEntity(newSpeaker);
}
```

现在看起来好多了，但我们仍然有一个问题。我们不能期望`Id`在任意对象上存在。编译器现在非常生气。有几个解决方案，但我们将创建一个简单的数据模型接口，并在`T`上放置一个约束，它必须继承该接口。接口中唯一的东西将是一个整型`Id`属性：

```cs
public interface IIdentity
{
  int Id { get; set; }
}

public abstract class InMemoryRepository<T> : IRepository<T> where T: IIdentity
```

这导致演讲者仓库中断。我们还必须让`Speaker`继承自`IIdentity`。现在，我们已经将`Create`方法中的所有逻辑移动到了通用仓库。删除演讲者仓库中的`Create`方法。

由于我们需要将演讲者仓库中的其他方法重新指向通用仓库中的支持对象，因此许多测试失败了。相反，遍历演讲者仓库并更新所有对`Speakers`的引用到`Entities`。

我们还需要调整`TestableSpeakerRepository`类：

```cs
internal class TestableSpeakerRepository : InMemorySpeakerRepository
{
  public IQueryable<Speaker> SpeakersCollection => Entities;
}
```

# 将`Get`转换为通用方法

接下来是列表中的`Get`方法。就像`Create`方法一样，将所有内容复制到通用仓库，并修复出现的任何错误：

```cs
public virtual T Get(int id)
{
  var entity = Entities.SingleOrDefault(e => e.Id == id);

  if (entity != null)
  {
    entity = CloneEntity(entity);
  }

  return entity;
}
```

这个方法证明是很容易复制的。现在，删除演讲者仓库中现有的方法。这次没有测试失败，所以我们可以继续到下一个方法。

# 将`GetAll`转换为通用方法

`GetAll`是转换起来最简单的方法。它甚至没有以文本形式引用演讲者：

```cs
public virtual IQueryable<T> GetAll()
{
  return Entities.Select(CloneEntity).AsQueryable();
}
```

删除演讲者仓库中现有的方法，继续到下一个方法，更新。

# 将`Update`转换为通用方法

更新过程与其他方法相同。复制现有代码的主体，将任何演讲者引用重命名为实体引用，然后删除现有方法：

```cs
public virtual T Update(T entity)
{
  var oldEntity = Entities.FirstOrDefault(s => s.Id == entity.Id);
  var index = Entities.IndexOf(oldEntity);

  if (index == -1)
  {
    throw new EntityNotFoundException(entity.Id);
  }

  Entities[index] = CloneEntity(entity);

  return CloneEntity(entity);
}
```

# 将`Delete`转换为通用方法

`Delete` 是一个不同的情况：每个对象在删除时可能都有不同的要求。一些实际上会被删除，而其他，如 Speaker，则仅仅会被标记为已删除。出于这个原因以及许多其他原因，我们选择将 `Delete` 的实现留给具体的仓储，而通用仓储将抛出一个未实现异常。

让我们为这个功能编写一个 `Delete` 测试类。它应该是简短且快速的：

```cs
public class Delete
{
  [Fact]
  public void ItThrowsNotImplementException()
  {
    // Arrange
    var repo = new InMemoryRepository<TestEntity>();

    // Act
    var result = Record.Exception(() => repo.Delete(new TestEntity()));

    // Assert
    Assert.IsAssignableFrom<NotImplementedException> 
    (result.GetBaseException());
     Assert.Equal("Delete is not avaliable for TestEntity", 
     result.Message);
  } 
}
```

```cs
public class TestEntity : IIdentity
{
  public int Id { get; set; }
}
```

编写这个测试也让我们将通用仓储从抽象类更改为普通类。更改类类型使我们不得不将 `CloneEntity` 方法从抽象更改为虚拟：

```cs
protected virtual T CloneEntity(T entity)
{
  return entity;
}
```

现在，我们可以写出通过测试的方法：

```cs
public virtual void Delete(T speaker)
{
  throw new NotImplementedException($"Delete is not avaliable for {typeof(T).Name}");
}
```

# 第三步 – 将测试重新定向以使用通用仓储

我们在处理 `Delete` 方法时开始了这个过程。但让我们继续处理其他方法。所有移动到通用仓储的方法都可以将大部分测试一起移动。

我们将放弃数据完整性测试，因为这些测试直接与演讲者仓储中的功能相关。出于相同的原因，我们将保留所有删除测试。

# InMemoryRepository 创建测试

为了实现 `InMemoryRepository` 的其余功能，我们将从 `Create` 方法的测试开始：

```cs
public class Create
{
  private readonly TestableEntityRepository _repo;

  public Create()
  {
    _repo = new TestableEntityRepository();
  }

  [Fact]
  public void ItExists()
  {
    // Act
    var result = _repo.Create(new TestEntity());
  }

  [Fact]
  public void ItAddsAEntityToTheRepository()
  {
    // Act
    var result = _repo.Create(new TestEntity());

    // Assert
    Assert.Equal(1, _repo.EntityCollection.Count());
  }

  [Fact]
  public void ItAssignsUniqueIdsToEachEntity()
  {
    // Act
    var entity1 = _repo.Create(new TestEntity());
    var entity2 = _repo.Create(new TestEntity());

    // Assert
    Assert.NotEqual(entity1.Id, entity2.Id);
  }
}
```

```cs
internal class TestableEntityRepository : InMemoryRepository<TestEntity>
{
  public IQueryable<TestEntity> EntityCollection => Entities;
}
```

# InMemoryRepository 获取测试

现在我们可以使用 `InMemoryRepository` 创建，我们应该能够准确测试 `Get` 方法：

```cs
public class Get
{
  private readonly InMemoryRepository<TestEntity> _repo;

  public Get()
  {
    _repo = new InMemoryRepository<TestEntity>();
  }

  [Fact]
  public void ItExists()
  {
    // Act
    var result = _repo.Get(0);
  }

  [Fact]
  public void ItReturnsAnEntityWhenFound()
  {
    // Arrange
    var entity = _repo.Create(new TestEntity() { Name = "Test Entity" });

    // Act
    var result = _repo.Get(entity.Id);

    // Assert
    Assert.NotNull(result);
    Assert.Equal("Test Entity", result.Name);
  }

  [Fact]
  public void ItReturnsNullWhenNotFound()
  {
    // Act
    var result = _repo.Get(-1);

    // Assert
    Assert.Null(result);
  }
}
```

# InMemoryRepository 获取所有测试

在单个对象获取工作后，让我们测试获取所有对象：

```cs
public class GetAll
{
  private readonly InMemoryRepository<TestEntity> _repo;

  public GetAll()
  {
    _repo = new InMemoryRepository<TestEntity>();
  }

  [Fact]
  public void ItExists()
  {
    // Act
    var result = _repo.GetAll();
  }

  [Fact]
  public void ItReturnsNoEntitiesWhenThereAreNoEntities()
  {
    // Act
    var result = _repo.GetAll();

    // Assert
    Assert.NotNull(result);
    Assert.IsAssignableFrom<IQueryable<TestEntity>>(result);
    Assert.Equal(0, result.Count());
  }

  [Fact]
  public void ItReturnsASingleEntityWhenOnlyOneEntityExists()
  {
    // Arrange
    _repo.Create(new TestEntity { Name = "Test Entity" });

    // Act
    var result = _repo.GetAll().ToList();

    // Assert
    Assert.Equal(1, result.Count);
    Assert.Equal("Test Entity", result.First().Name);
  }

  [Fact]
  public void ItReturnsManyEntitiesWhenManyEntitiesExist()
  {
    // Arrange
    _repo.Create(new TestEntity());
    _repo.Create(new TestEntity());
    _repo.Create(new TestEntity());

    // Act
    var result = _repo.GetAll().ToList();

    // Assert
    Assert.Equal(3, result.Count);
  }
}
```

# InMemoryRepository 更新测试

最后但同样重要的是，我们将添加使用 `InMemoryRepository` 更新记录的能力：

```cs
public class Update
{
  private readonly InMemoryRepository<TestEntity> _repo;

  public Update()
  {
    _repo = new InMemoryRepository<TestEntity>();
  }

  [Fact]
  public void ItExists()
  {
    // Arrange
    var entity = _repo.Create(new TestEntity());
   // Act
    var result = _repo.Update(entity);
  }

  [Fact]
  public void ItUpdatesAnEntity()
  {
    // Arrange
    var entity = _repo.Create(new TestEntity(){ Name = "Test Name" });
    entity.Name = "New Name";
    // Act
    var result = _repo.Update(entity);
   // Assert
    Assert.Equal(entity.Name, result.Name);
  }

  [Fact]
  public void ItUpdatesAnEntityInTheRepository()
  {
    // Arrange
    var entity = _repo.Create(new TestEntity() { Name = "Test Name" });
    entity.Name = "New Name";
   // Act
    var updatedEntity = _repo.Update(entity);
   // Audit
    var result = _repo.Get(entity.Id);
   // Assert
    Assert.Equal("New Name", result.Name);
  }

  [Fact]
  public void ItThrowsNotFoundExceptionWhenEntityDoesNotExist()
  {
    // Arrange
    var entity = new TestEntity { Id = 5, Name = "Test Name" };
    // Act
    var result = Record.Exception(() => _repo.Update(entity));
    // Assert
    Assert.IsAssignableFrom<EntityNotFoundException>(result.GetBaseException());
  }
}
```

现在我们有一个可以与任何具有 ID 的数据模型一起使用的通用仓储。如果我们需要确保数据完整性，我们还可以通过继承并创建一个克隆所需保护的数据对象的方法来实现。

# 实体框架

**对象关系映射**（**ORM**）框架，如 Entity Framework，有助于提高生产力和优化代码重用及可维护性。然而，您不应将应用程序紧密耦合到 ORM。像对待任何第三方库或系统一样，抽象化 ORM，如 Entity Framework。

Speaker Meet 通过通用仓储使用 Entity Framework；为了开始，添加对 Entity Framework – Sql Server 的 NuGet 引用。

```cs
Microsoft.EntityFrameworkCore.SqlServer
```

接下来，将连接字符串添加到 `appsettings.json`：

```cs
 "ConnectionStrings": {"DefaultConnection":   
 "Server=.;Database=SpeakerMeetBook;Trusted_Connection=True;MultipleAct
  iveResultSets=true"}
```

# DbContext

最新版本的 Entity Framework，EF Core 2.0，已添加 DbContext 缓存，这有助于通过节省每次请求初始化新的 `DbContext` 实例的一些成本来提高性能。

修改 `Startup.cs` 中的 ConfigureServices 以引用连接字符串：

```cs
 var connectionString =   
  Configuration.GetConnectionString("DefaultConnection");
 services.AddDbContextPool<SpeakerMeetContext>(options =>   
  options.UseSqlServer(connectionString));
```

将一个 `DbContext` 添加到您的应用程序中。创建一个名为 `SpeakerMeetContext` 的新文件，并使用以下选项：

```cs
using Microsoft.EntityFrameworkCore;

namespace SpeakerMeet.Api.Entities
{
  public class SpeakerMeetContext : DbContext
  {
    public SpeakerMeetContext(DbContextOptions<SpeakerMeetContext> options) : base(options)
    { }

    public virtual DbSet<Speaker> Speakers { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
      modelBuilder.Entity<Speaker>().ToTable("Speaker");
    }
  }
}
```

# 模型

Entity Framework 模型可能包含您可能不希望暴露给系统其他部分或应用程序消费者的额外信息。创建一个名为`Speaker`的新模型。这将是由 Entity Framework 和通用仓库使用的模型。服务或业务层将负责将 Entity Framework 模型转换为应用程序其他部分使用的**数据传输对象**（**DTOs**）或 ViewModels：

```cs
using System.ComponentModel.DataAnnotations;

namespace SpeakerMeet.Api.Entities
{
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
  }
}
```

# 通用仓库

为了看到 Entity Framework 特定的通用仓库运行，您可能需要将以下类添加到您的应用程序中：

```cs
using System;
using System.Linq;
using Microsoft.EntityFrameworkCore;

namespace SpeakerMeet.Api.Repository
{
  public class Repository<T> : IRepository<T> where T : class
  {
    private readonly DbSet<T> _dbSet;
    protected readonly DbContext Context;

    public Repository(DbContext context)
    {
      Context = context;
      _dbSet = context.Set<T>();
    }

    public T Create(T entity)
    {
      throw new NotImplementedException();
    }

    public T Get(int id)
    {
      return _dbSet.Find(id);
    }

    public IQueryable<T> GetAll()
    {
      return _dbSet;
    }

    public T Update(T speaker)
    {
      throw new NotImplementedException();
    }

    public void Delete(T entity)
    {
      throw new NotImplementedException();
    }
  }
}
```

# 依赖注入

为了将所有组件连接起来，Speaker Meet 应用程序利用了内置的 **依赖注入**（**DI**）容器。依赖注入允许系统实现松耦合。有一些方法可以在不使用容器的情况下实现这一点（例如“穷人版 DI”等），这些方法将在后面的章节中介绍。测试本身并不依赖于 DI，而是选择按需实例化类。

# 连接所有组件

为了配置 DI 容器，请将以下内容添加到`Startup.cs`中的`ConfigureServices`：

```cs
services.AddSingleton(typeof(DbContext), typeof(SpeakerMeetContext));
services.AddScoped(typeof(IRepository<>), typeof(Repository<>));
services.AddTransient<ISpeakerService, SpeakerService>();
services.AddTransient<IGravatarService, GravatarService>();
```

这样可以避免在`SpeakerController`内部实例化`SpeakerService`的需要。DI 容器会为您处理这个问题。

如果您已经创建了数据库并填充了演讲者表，现在您应该能够运行应用程序并访问`SpeakerController`端点。试一试吧！

+   `http://localhost:41436/api/speaker/`

+   `http://localhost:41436/api/speaker/1`

+   `http://localhost:41436/api/speaker/search?searchString=test`

# Postman

有许多工具可用于手动测试 API。Postman 只是其中之一，并且在业界很受欢迎。Postman 提供了许多功能来帮助您进行 API 开发和测试。如果您对此感兴趣，值得一试。

要安装 Postman，只需访问网站（[`www.getpostman.com`](http://www.getpostman.com)）并按照说明操作。当 Speaker Meet 应用程序运行时，将 URL（例如：`http://localhost:41436/api/speaker/search`）输入框中，添加参数（例如：`[{"key":"searchString","value":"te","description":""}]`），然后点击发送。默认情况下，响应显示在消息体的 JSON 中。

这可以是一个非常强大的工具。随着 Speaker Meet API 的复杂性和功能集的增长，可以使用`POST`、`PUT`、`PATCH`和`DELETE`发送更复杂的消息。这些将在后面的章节中介绍。

# 摘要

本章主要讲述了抽象：何时以及如何使用它。你现在应该明白为什么在您自己编写的代码和第三方代码之间创建抽象如此重要。你也应该对如何实现这些抽象有一个很好的了解。

在本章中，我们抽象了一个 Gravatar 服务，扩展了仓库模式，并使用通用仓库为 Entity Framework。

在第九章，*测试 JavaScript 应用程序*，我们将专注于测试 JavaScript 应用程序。我们将逐步创建一个 React 应用程序，并讨论测试 JavaScript 应用程序的不同方法。
