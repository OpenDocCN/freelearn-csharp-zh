

# 应用开放封闭原则

开放封闭原则归功于伯特兰·梅耶（Bertrand Meyer），该原则在他的 1988 年著作《面向对象软件构造》（*Object-Oriented Software Construction*）中首次出现（[`en.wikipedia.org/wiki/Object-Oriented_Software_Construction`](https://en.wikipedia.org/wiki/Object-Oriented_Software_Construction)）。这本书描述了以下我们可以应用到我们的软件中的原则：

+   一个类型是开放的，如果它可以被扩展

+   当一个类型对其他类型可用时，它就是封闭的

假设我们有一个名为 **Shape** 的类，它有一个名为 **area** 的方法，该方法反过来计算形状的面积。我们希望能够在不修改 **Shape** 类的情况下将新形状添加到我们的程序中，因此我们使 **Shape** 类对扩展开放。

为了做到这一点，我们创建了一个名为 **Triangle** 的新类，它继承自 **Shape** 并重写了 **area** 方法来计算三角形的面积。我们还可以创建一个 **Rectangle** 类和任何其他我们想要的新的形状。

现在，每当我们需要计算形状的面积时，我们只需创建适当的形状类的新实例并调用其 **area** 方法。因为 **Shape** 类对修改是封闭的，所以我们不需要每次添加新形状到我们的程序时都修改它。

C# 中的类默认是开放的，可以扩展。我们可以从任何非密封的类继承，并为它们添加新的含义。但是，我们正在继承的基类应该是封闭的，这意味着新类型要工作，不需要对基类型进行任何更改。

这有助于我们设计可扩展的代码，并保持责任在正确的位置。

退一步，我们可以在系统级别应用一些内容。如果我们能够简单地扩展我们的系统，而不需要在类型的中心添加配置以使其了解添加的内容，那会怎么样？

这种思维方式是我们之前在 *第四章*，*使用反射推理类型* 中使用的，与 **ICommand** 接口及其实现有关。我们添加的两个命令不为系统的任何部分所知，但通过实现 **ICommand** 接口，我们可以通过系统的内省看到这两种类型。

在本章中，我们将涵盖以下主题：

+   开放封闭原则的目标

+   封装类型发现

+   封装实例的发现

+   与服务集合连接

+   一个实际的应用场景

到本章结束时，你将了解如何设置你的代码和项目以成功，使它们更加灵活和可扩展，以及如何创建欢迎变化和添加的代码，而无需每次都对代码进行大手术。

# 技术要求

本章的特定源代码可以在 GitHub 上找到（[`github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Chapter11`](https://github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Chapter11)），并且它基于**基础**代码，可以在[`github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Fundamentals`](https://github.com/PacktPublishing/Metaprogramming-in-C-Sharp/tree/main/Fundamentals)找到。

开放封闭原则的目标

在一个不断变化和增长的需求、新发现的商业机会，甚至是对你业务进行转型调整的世界中，如果你的软件需要进行三次旁路手术才能进行更改，那么这并不是很有帮助。敏捷思维的核心在于能够灵活并及时地应对变化。从商业角度来看，这种弹性是非常有用的。目标是当变化发生时能够成本效益。在我参与的项目或产品开发中，我经常注意到这一点转化为一种临时的心态，并且常常完全忽视以使代码能够持久并让开发者理解的方式进行编码。

开发初期，到达软件产品第一个版本令人兴奋的部分，是我们为代码库的未来代码设定基调的时候。然而，在大多数情况下，这仅代表了代码生命周期的一小部分。这就是为什么我相信我们应该更多地关注我们如何为代码的维护成功做好准备。

我经常遇到的一个非常常见的想法是，仅仅推出第一个版本是关键，之后我们可以开始修复我们最初不引以为豪的所有事情，比如我们因为认为没有时间做“正确”的事情而不得不采取的所有捷径。追逐**最小可行产品（MVP**）是一种非常流行的方法。这种追逐是由将产品推向市场并从中学习的需求驱动的。不幸的是，根据我的经验，我们往往根本无法创建一个可行的产品，而只是证明了我们想要构建的原型。它们可能表面上看起来像产品，但缺乏一个合适的立足点。从非技术角度来看，这正好是开发者想要的，他们想要你继续前进。而且谁又能责怪他们呢？开始使用这些产品的客户也可能觉得自己在使用一个产品，但他们根本不知道表面之下发生了什么。很快，在推出第一个版本之后，业务、客户和最终用户会回来提出他们希望软件能做的事情。我还没有看到任何企业在这个时候优先考虑修复基础。当这一点与软件中报告的不足或具体错误结合起来时，你最终会朝着新的目标冲刺。你推出第一个版本时的兴奋感消失了，工作变成了苦差事。

我们没有将基于设计的思考融入代码中，这对业务造成了巨大的损害，这本来可以为企业成功奠定基础。代码可能比原本的状态更糟糕，这使得进行所有这些更改变得困难。这通常会导致开发者满意度降低，他们最终可能会决定离开，希望他们下一个工作的地方能提供一个更好的工作环境来编写更好的代码。

我要带这些去哪里呢？我确实相信可以交付真正的最小可行产品（MVP），大幅度降低代码库中的技术债务，并使迭代 MVP 成为可能，以更快的速度、更准确的方式交付更多内容，并拥有一个强大的基础。我相当确信企业更愿意选择后者——一种不会一上市就慢慢消亡的东西。

在实现这一点的核心中，坐落着我最喜欢的一个原则：**开闭原则**。正如我之前提到的，我认为这是一种战略思维：不仅是一种编写类的战术方法，而是一种系统级的方法。我们如何确保我们可以在任何时候随意添加代码，扩展我们软件的功能，同时有信心它不会破坏现有的系统？如果我们能够避免修改现有代码来实现我们的新目标，我们就可以大幅度降低软件回归的风险。

我们可以利用这本书中迄今为止构建的构建块来实现这一点，并在此基础上进行改进，使它们更加友好。

# 封装类型发现

在*第四章*，“使用反射推理类型”，我们介绍了一个名为**Types**的类，它封装了用于查找类型的逻辑。这是一个非常强大的结构，它使软件能够扩展。我们可以使其更好一些，并在其之上构建一个结构，以简化其使用。

在**Types**类中，我们有一个名为**FindMultiple()**的方法，它允许我们找到实现了特定类型的类型。对这个方法的一个小改进是允许我们通过在描述此类型的特定类型的构造函数中引入依赖来表示我们想要实现的不同类型。

重要提示

您可以在*技术要求*部分提到的存储库的**基础**部分找到这个实现的实现。

这个概念基本上是将类型表示为一个接口，如下所示：

```cs
public interface IImplementationsOf<T> : IEnumerable<Type>
    where T : class
{
}
```

该接口接受一个泛型类型，该类型描述了您感兴趣的类型的实现。然后它说明这是一个**Type**的可枚举类型，这使得您可以直接遍历它。**class**的泛型约束存在是为了限制您可以工作的类型范围，因为允许原始类型（如**int**）是不继承的，这并不实用。

接口的实现非常直接，看起来如下所示：

```cs
public class ImplementationsOf<T> : IImplementationsOf<T>
    where T : class
{
    readonly IEnumerable<Type> _types;
    public ImplementationsOf(ITypes types)
    {
        _types = types.FindMultiple<T>();
    }
    public IEnumerator<Type> GetEnumerator()
    {
        return _types.GetEnumerator();
    }
    IEnumerator IEnumerable.GetEnumerator()
    {
        return _types.GetEnumerator();
    }
}
```

代码将**ITypes**作为依赖项，因为它将使用它来实际找到实现。

发现类型是一回事，这使得方法更优雅，代码更易读、更清晰。但这只提供了类型。更常见的方法是不仅获取类型，还要获取其实例。

# 封装实例的发现

我们可以执行的另一个封装，这可能更适合我们当前的场景，是找到实现并直接获取它们的实例。

重要提示

您可以在存储库的**基础**部分，在*技术要求*部分的*技术*要求中找到这个实现的实现。

这个概念基本上是将代码表示得与使用**IImplementationsOf<T>**作为接口的情况相似，如下所示：

```cs
public interface IInstancesOf<T> : IEnumerable<T>
    where T : class
{
}
```

代码中的接口使用泛型参数来确定它可以提供实例的类型。然后它实现**IEnumerable<T>**，这使得直接枚举其实例并获取该类型的实例成为可能。

接口的实现非常直接，如下所示：

```cs
public class InstancesOf<T> : IInstancesOf<T>
    where T : class
{
    readonly IEnumerable<Type> _types;
    readonly IServiceProvider _serviceProvider;
    public InstancesOf(ITypes types,
      IServiceProvider serviceProvider)
    {
        _types = types.FindMultiple<T>();
        _serviceProvider = serviceProvider;
    }
    public IEnumerator<T> GetEnumerator()
    {
        foreach (var type in _types) yield return
          (T)_serviceProvider.GetService(type)!;
    }
    IEnumerator IEnumerable.GetEnumerator()
    {
        foreach (var type in _types) yield return
          _serviceProvider.GetService(type);
    }
}
```

代码利用了**ITypes**，并且与**ImplementationsOf<T>**类似，它使用**FindMultiple()**方法来查找实际类型。由于它可能会发现它一无所知的类型，并且如果这些类型没有无参数的默认构造函数，它不知道如何实例化它们，因此它需要支持来完成这项工作。为此，它利用.NET 中找到的**IServiceProvider**实例。**IServiceProvider**是在使用.NET 依赖注入系统时设置的。然后，**GetEnumerator()**实现遍历类型，并通过在**IServiceProvider**上调用**GetService()**方法来提供实例。

注意，实现仅在枚举时请求**IServiceProvider**，而不是在构造函数中直接这样做。这样做的原因是，你不想承担某些东西依赖于生命周期比实现更长的**IInstancesOf<T>**的风险。例如，如果一个使用**IInstancesOf<T>**的类是**singleton**，那么使用它的类中的所有实例也都是单例。

唯一的缺点是，当为没有在**IServiceProvider**上注册具体类型的类型调用**GetService()**时，你会得到**null**。这并不很有用。

# 连接到服务集合

由于我们利用**ServiceProvider**来创建实例，并且由于它的默认行为是将所有内容显式注册到它那里，它可能没有为我们请求的具体类型注册。

在*第十章*，“约定优于配置”中，你实现了一个根据约定发现接口和实现之间关系的功能。我们可以扩展这种思考，并说应该能够根据它们的类型解决类本身。

要做到这一点，我们利用更多的反射元数据来过滤掉我们不感兴趣的不同类型。

在*第十章*，“约定优于配置”中，你创建了一个名为**Service** **CollectionExtensions**的东西。我们希望在这里也使用它，但我们还想添加一些额外的功能。将文件移动到共享的**Fundamentals**项目中。

打开**ServiceCollectionExtensions**类并添加以下内容：

```cs
public static IServiceCollection AddSelfBinding(this IServiceCollection services, ITypes types)
{
    const TypeAttributes staticType =
      TypeAttributes.Abstract | TypeAttributes.Sealed;
    types.All.Where(_ =>
        (_.Attributes & staticType) != staticType &&
        !_.IsInterface &&
        !_.IsAbstract &&
        services.Any(s => s.ServiceType !=
          _)).ToList().ForEach(_ =>
    {
        var __ = _.HasAttribute<SingletonAttribute>() ?
            services.AddSingleton(_, _) :
            services.AddTransient(_, _);
    });
    return services;
}
```

代码使用**ITypes**系统，通过忽略**静态**类来过滤类型，因为它们不能被实例化。此外，它还忽略接口和抽象类型，原因相同。然后，它忽略任何已在**services**集合中注册的类型。任何剩下的类型都会被注册。如果类型有**[Singleton]**属性，它将注册为单例；否则，它使用**transient**生命周期，这意味着每次你请求时都会得到一个新的实例。

这样一来，我们正在为更实际地应用我们所拥有的内容做准备。

# 实际用例

让我们构建一些利用新发现形式的东西，更重要的是，展示一个弹性增长系统的概念，该系统不需要在核心进行更改即可添加新功能。

让我们重新审视合规性的概念，这在软件中是一个非常常见的场景。我们在 *第四章*，*使用反射进行类型推理* 和 *第五章*，*利用属性* 中探讨了 GDPR，GDPR 只是合规性的一种类型，处理它非常重要。

目标是创建一个系统，其中核心“引擎”不知道可能存在的不同类型的合规性元数据，而是提供可扩展点，开发者可以在需要时添加新的合规性元数据类型。

我们希望引擎是通用的，并且对每个人都是可访问的。根据*技术要求*中提到的 GitHub 仓库，你会发现有一个**Fundamentals**项目。这里列出的所有代码都可以在那里找到。

让我们首先创建一个记录，它表示利用**Fundamentals**中找到的**ConceptAs<>**基记录所利用的元数据类型。添加一个名为**ComplianceMetadataType.cs**的文件，并使其看起来如下：

```cs
namespace Fundamentals.Compliance;
public record ComplianceMetadataType(Guid Value) : ConceptAs<Guid>(Value)
{
    public static readonly ComplianceMetadataType PII = new(Guid.Parse("cae5580e-83d6-44dc-9d7a-a72e8a2f17d7"));
    public static implicit operator
      ComplianceMetadataType(string value) =>
      new(Guid.Parse(value));
    public static implicit operator
      ComplianceMetadataType(Guid value) => new(value);
}
```

代码引入了一个继承自**ConceptAs<>**的**record**实例，并将其中的值设为**Guid**实例。然后它添加了一个用于**个人身份信息**的已知类型，或简称为**PII**。该类型还添加了从**Guid**实例的**string**表示形式以及直接从**Guid**实例转换的隐式运算符。

重要提示

**ConceptAs<>** 类型在 *第四章*，*使用反射进行类型推理* 中进行了解释。

接下来，我们想要一个表示实际元数据的类型。添加一个名为**ComplianceMetadata.cs**的文件，并使其看起来如下：

```cs
namespace Fundamentals.Compliance;
public record ComplianceMetadata(ComplianceMetadataType MetadataType, string Details);
```

代码将元数据表示为对元数据类型的引用，后跟一个**Details**字符串。

为了使我们能够发现元数据，我们需要某种可以提供这些元数据的东西。我们将其表示为一个接口，它可以独立地为我们提供发现。

添加一个名为**ICanProvideComplianceMetadataForType.cs**的文件，并使其看起来如下：

```cs
namespace Fundamentals.Compliance;
public interface ICanProvideComplianceMetadataForType
{
    bool CanProvide(Type type);
    ComplianceMetadata Provide(Type type);
}
```

代码表示一个提供者，它通过**CanProvide()**方法决定是否能够提供元数据，然后是一个实际提供元数据的方法。可扩展性的关键在于这种模式，它使我们能够插入任何可以针对任何类型调用的实现，而实现本身则决定它是否可以提供元数据。

由于**ICanProvideComplianceMetadataForType**仅关注为**Type**提供元数据，因此我们需要另一个提供者来处理类型的属性。

添加一个名为**ICanProvideComplianceMetadataForProperty.cs**的文件，并使其看起来如下：

```cs
namespace Fundamentals.Compliance;
public interface ICanProvideComplianceMetadataForProperty
{
    bool CanProvide(PropertyInfo property);
    ComplianceMetadata Provide(PropertyInfo property);
}
```

与**ICanProvideComplianceMetadataForType**一样，您会看到**ICanProvideComplianceMetadataForProperty**有**CanProvide()**方法和一个**Provide()**方法；唯一的区别是它专注于**PropertyInfo**。

在可发现接口就绪后，我们可以开始构建发现这些内容并将它们组合成可以作为一个系统利用的引擎。

让我们先通过添加一个接口来定义合规引擎的合同。添加一个名为**IComplianceMetadataResolver.cs**的文件，并使其看起来如下：

```cs
namespace Fundamentals.Compliance;
public interface IComplianceMetadataResolver
{
    bool HasMetadataFor(Type type);
    bool HasMetadataFor(PropertyInfo property);
    IEnumerable<ComplianceMetadata> GetMetadataFor(Type
      type);
    IEnumerable<ComplianceMetadata>
      GetMetadataFor(PropertyInfo property);
}
```

代码添加了询问是否**Type**或**PropertyInfo**与其关联元数据的方法，以及获取关联元数据的方法。

## 帮助开发者

明确调用代码是否跳过调用**Has*()**方法并直接调用**Get*()**方法，以及是否没有元数据是很重要的。如果没有元数据，**Get*()**方法实际上无法做任何事情，只能抛出异常。

添加一个名为**NoComplianceMetadataForType.cs**的文件，并使其看起来如下：

```cs
namespace Fundamentals.Compliance;
public class NoComplianceMetadataForType : Exception
{
    public NoComplianceMetadataForType(Type type)
        : base($"Types '{type.FullName}'  does not have any
        compliance metadata.")
    {
    }
}
```

代码使用清晰的名称和消息表示类型没有任何元数据来表示异常。

让我们对没有元数据的属性做同样的事情。添加一个名为**NoComplianceMetadataForProperty.cs**的文件，并使其看起来如下：

```cs
namespace Fundamentals.Compliance;
public class NoComplianceMetadataForProperty : Exception
{
    public NoComplianceMetadataForProperty(PropertyInfo
      property)
        : base($"Property '{property.Name}' on type
          '{property.DeclaringType?.FullName}' does not
          have any compliance metadata.")
    {
    }
}
```

与**NoComplianceMetadataForType**一样，**NoComplianceMetadataForProperty**异常很清晰，正如其异常名称和消息所暗示的，它说明了哪些属性没有与它们关联的元数据。

现在您可以创建实现**IComplianceMetadataResolver**接口的实现。

添加一个名为**ComplianceMetadataResolver.cs**的文件，并使其看起来如下：

```cs
namespace Fundamentals.Compliance;
public class ComplianceMetadataResolver : IComplianceMetadataResolver
{
    readonly IEnumerable<
      ICanProvideComplianceMetadataForType> _typeProviders;
    readonly IEnumerable<
      ICanProvideComplianceMetadataForProperty>
      _propertyProviders;
    public ComplianceMetadataResolver(
        IInstancesOf<ICanProvideComplianceMetadataForType>
          typeProviders,
        IInstancesOf<
          ICanProvideComplianceMetadataForProperty>
          propertyProviders)
    {
        _typeProviders = typeProviders.ToArray();
        _propertyProviders = propertyProviders.ToArray();
    }
}
```

代码使用**IInstancesOf<>**来处理**ICanProvideComplianceMetadataForType**提供者类型和**ICanProvideComplianceMetadataForProperty**提供者类型。它在构造函数中收集所有这些类型并创建它们的实例。

重要提示

可能不希望保留实例，就像这里展示的情况一样。这完全取决于实现。在这个用例中，这是可以的。

现在您需要实现接口中的方法。将以下代码添加到**ComplianceMetadataResolver**类的末尾：

```cs
public bool HasMetadataFor(Type type) => _typeProviders.Any(_ => _.CanProvide(type));
public IEnumerable<ComplianceMetadata> GetMetadataFor(Type type)
{
    ThrowIfNoComplianceMetadataForType(type);
    return _typeProviders
        .Where(_ => _.CanProvide(type))
        .Select(_ => _.Provide(type))
        .ToArray();
}
void ThrowIfNoComplianceMetadataForType(Type type)
{
    if (!HasMetadataFor(type))
    {
        throw new NoComplianceMetadataForType(type);
    }
}
```

代码使用在构造函数中发现的**_typeProviders**来确定它给出的类型是否是提供者可以提供元数据的类型。如果可以，它将返回 true；如果不可以，它将返回 false。**GetMetadataFor()**方法检查它是否可以提供；如果它不能，它将抛出**NoComplianceMetadataForType**异常。如果它可以，它将筛选出可以提供的提供者，然后要求它们提供。然后它将所有元数据组合成一个集合。

## 支持的属性

您现在想对属性做同样的事情。将以下代码添加到**ComplianceMetadataResolver**类的末尾：

```cs
public bool HasMetadataFor(PropertyInfo property) => _propertyProviders.Any(_ => _.CanProvide(property));
public IEnumerable<ComplianceMetadata> GetMetadataFor(PropertyInfo property)
{
    ThrowIfNoComplianceMetadataForProperty(property);
    return _propertyProviders
        .Where(_ => _.CanProvide(property))
        .Select(_ => _.Provide(property))
        .ToArray();
}
void ThrowIfNoComplianceMetadataForProperty(PropertyInfo property)
{
    if (!HasMetadataFor(property))
    {
        throw new
          NoComplianceMetadataForProperty(property);
    }
}
```

对于类型，代码询问**_propertyProviders**是否可以提供，然后使用它通过**GetMetadataFor()**方法过滤属性。如果没有元数据，当请求时将发生与类型抛出异常相同的操作。

在引擎就绪后，我们现在需要利用它并创建第一个实现。

在**基础**中，你应该已经有一个名为**个人可识别信息属性**的属性。现在你想要创建一个提供者，可以在属性级别提供此属性的元数据。

在**Fundamentals**文件夹中的**Compliance**文件夹的子文件夹中，创建一个名为**PersonalIdentifiableInformationMetadataProvider.cs**的文件，并使其看起来如下：

```cs
namespace Fundamentals.Compliance.GDPR;
public class PersonalIdentifiableInformationMetadataProvider :  ICanProvideComplianceMetadataForProperty
{
    public bool CanProvide(PropertyInfo property) =>
        property.GetCustomAttribute<PersonalIdentifiableInformationAttribute>() != default ||
        property.DeclaringType?.GetCustomAttribute<PersonalIdentifiableInformationAttribute>() != default;
    public ComplianceMetadata Provide(PropertyInfo
      property)
    {
        if (!CanProvide(property))
        {
            throw new
              NoComplianceMetadataForProperty(property);
        }
        var details = property.GetCustomAttribute<
          PersonalIdentifiableInformationAttribute>()!
          .ReasonForCollecting;
        return new
          ComplianceMetadata(ComplianceMetadataType.PII,
          details);
    }
}
```

代码在给定的属性中查找**个人可识别信息属性**；如果它在属性或声明类型中存在，它可以提供**合规元数据**。如果存在，此方法提供**合规元数据**并使用**收集原因**来提供详细信息。

## 使用 GDPR 基础设施

在你的第一个 GDPR 提供者就绪后，你可以开始创建利用它的系统。

现在让我们在存储库的根目录下创建一个名为**Chapter11**的文件夹。在命令行界面中切换到这个文件夹，并创建一个新的控制台项目，如下所示：

```cs
dotnet new console
```

你将利用 Microsoft 托管模型来获取.NET 默认服务提供者，而不需要启动 Web 应用程序。为此，你需要名为**Microsoft.Extensions.Hosting**的包。在终端中，你将按照以下操作添加引用：

```cs
dotnet add package Microsoft.Extensions.Hosting
```

下一步你需要做的是引用**基础**项目。在终端中，执行以下操作：

```cs
dotnet add reference ../Fundamentals/Fundamentals.csproj
```

现在你可以开始为患者系统建模一个简单的领域模型。首先添加一个名为**JournalEntry.cs**的文件，并使其看起来如下：

```cs
namespace Chapter11;
public class JournalEntry
{
    public string Title { get; set; } = string.Empty;
    public string Content { get; set; } = string.Empty;
}
```

代码添加了一个表示患者日志条目的类型，具有标题和内容属性。

添加一个名为**Patient.cs**的文件，并使其看起来如下：

```cs
using Fundamentals.Compliance.GDPR;
namespace Chapter11;
public class Patient
{
    [PersonalIdentifiableInformation("Employment records")]
    public string FirstName { get; set; } = string.Empty;
    [PersonalIdentifiableInformation("Employment records")]
    public string LastName { get; set; } = string.Empty;
    [PersonalIdentifiableInformation("Uniquely identifies
      the employee")]
    public string SocialSecurityNumber { get; set; } =
      string.Empty;
    public IEnumerable<JournalEntry> JournalEntries { get;
      set; } = Enumerable.Empty<JournalEntry>();
}
```

代码添加了一个定义，包含患者的名字、姓氏和社保号码，以及该患者的所有日志条目。对于个人信息，使用**[个人可识别信息]**作为元数据的形式。

打开**Program.cs**文件，并使其看起来如下：

```cs
using Chapter11;
using Fundamentals;
using Fundamentals.Compliance;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
var host = Host.CreateDefaultBuilder()
    .ConfigureServices((context, services) =>
    {
        var types = new Types();
        services.AddSingleton<ITypes>(types);
        services.AddBindingsByConvention(types);
        services.AddSelfBinding(types);
    })
    .Build();
```

代码设置了一个通用的宿主构建器，将**类型**注册为单例，然后利用**AddBindingsByConvention()**（你在*第十章*，*约定优于配置*中创建的）通过约定连接服务。然后调用**AddSelfBinding()**，这是你在本章早期引入的。最后，构建宿主实例。

使用宿主实例，我们获取**服务**属性，这是构建服务提供者。您使用此属性来获取**IComplianceMetadataResolver**的实例。

在 **Program.cs** 的末尾添加以下代码：

```cs
var complianceMetadataResolver = host.Services.
  GetRequiredService<IComplianceMetadataResolver>();
var typeToCheck = typeof(Patient);
Console.WriteLine($"Checking type for compliance rules: {typeToCheck.
  FullName}");
if (complianceMetadataResolver.HasMetadataFor(typeToCheck))
{
    var metadata =
      complianceMetadataResolver.GetMetadataFor(
      typeToCheck);
    foreach (var item in metadata)
    {
        Console.WriteLine($"Type level - {item.Details}");
    }
}
```

代码使用 **IComplianceMetadataResolver** 来获取类型的元数据。在你的情况下，目前它是硬编码为 **Patient**。

要从属性中获取元数据，请在 **Program.cs** 的末尾添加以下代码：

```cs
foreach (var property in typeToCheck.GetProperties())
{
    if (complianceMetadataResolver
      .HasMetadataFor(property))
    {
        var metadata = complianceMetadataResolver
          .GetMetadataFor(property);
        foreach (var item in metadata)
        {
            Console.WriteLine($"Property: {property.Name} –
              {item.Details}");
        }
    }
    else if (property.PropertyType.IsGenericType &&
      property.PropertyType.GetGenericTypeDefinition()
      .IsAssignableTo(typeof(IEnumerable<>)))
    {
        var type = property.PropertyType
          .GetGenericArguments().First();
        if (complianceMetadataResolver
          .HasMetadataFor(type))
        {
            Console.WriteLine($"\nProperty {property.Name}
              is a collection of type {type.FullName} with
              type level metadata");
            var metadata = complianceMetadataResolver.
               GetMetadataFor(type);
            foreach (var item in metadata)
            {
                Console.WriteLine($"{property.Name} –
                  {item.Details}");
            }
        }
    }
}
```

代码遍历 **typeToCheck** 的属性，然后打印出任何来自属性的元数据细节。它还寻找任何具有泛型参数并实现 **IEnumerable<>** 的属性，并打印出与项目类型关联的任何元数据。

运行你的程序应该会给你以下输出：

```cs
Checking type for compliance rules: Chapter11.Patient
Property: FirstName - Employment records
Property: LastName - Employment records
Property: SocialSecurityNumber - Uniquely identifies the employee
Property JournalEntries is a collection of type Chapter11.JournalEntry with type level metadata
```

## 添加更多提供者

在引擎就位后，你现在可以通过简单地添加新的提供者来开始向其中添加内容。让我们为 **JournalEntry** 添加一个。添加一个名为 **JournalEntryMetadataProvider.cs** 的文件，并使其看起来如下所示：

```cs
using Fundamentals.Compliance;
namespace Chapter11;
public class JournalEntryMetadataProvider : ICanProvideComplianceMetadataForType
{
    public bool CanProvide(Type type) => type == typeof(JournalEntry);
    public ComplianceMetadata Provide(Type type) => new("7242aed8-
          8d70-49df-8713-eea45e2764d4", "Journal entry");
}
```

代码使用 **JournalEntry** 类型来决定它是否可以提供。每个 **JournalEntry** 都应该被特别对待，因为它包含不应共享的关键信息。**Provide()** 方法会为 **JournalEntry** 类型创建一个新的条目，并带有唯一的标识符。

运行程序现在应该会给你更多信息；注意添加到你的输出中的日志条目：

```cs
Checking type for compliance rules: Chapter11.Patient
Property: FirstName - Employment records
Property: LastName - Employment records
Property: SocialSecurityNumber - Uniquely identifies the employee
Property JournalEntries is a collection of type Chapter11.JournalEntry
  with type level metadata
JournalEntries - Journal entry
```

为了继续向你的系统添加内容，让我们添加一些可以让你标记类型为 **confidential** 的东西。

添加一个名为 **ConfidentialAttribute.cs** 的文件，并使其看起来如下所示：

```cs
namespace Chapter11;
[AttributeUsage(AttributeTargets.Class, AllowMultiple = false,
  Inherited = true)]
public sealed class ConfidentialAttribute : Attribute
{
}
```

该属性针对类。

接下来，你需要一个提供者，可以为新属性提供元数据。添加一个名为 **ConfidentialMetadataProvider.cs** 的文件，并使其看起来如下所示：

```cs
using System.Reflection;
using Fundamentals.Compliance;
namespace Chapter11;
public class ConfidentialMetadataProvider :
  ICanProvideComplianceMetadataForType
{
    public bool CanProvide(Type type) =>
      type.GetCustomAttribute<ConfidentialAttribute>() !=
      null;
    public ComplianceMetadata Provide(Type type) =>
      new("8dd1709a-bbe1-4b98-84e1-9e7be2fd4912", "The data
      is confidential");
}
```

提供者寻找 **ConfidentialAttribute** 来决定它是否可以提供。如果可以，则 **Provide()** 方法会为它所代表的类型创建一个新的 **ComplianceMetadata** 实例，并带有唯一的标识符。

打开 **Patient.cs** 文件，并在 **Patient** 类前面添加 **[Confidential]** 属性，使类看起来如下所示：

```cs
using Fundamentals.Compliance.GDPR;
namespace Chapter11;
[Confidential]
public class Patient
{
    [PersonalIdentifiableInformation("Employment records")]
    public string FirstName { get; set; } = string.Empty;
    [PersonalIdentifiableInformation("Employment records")]
    public string LastName { get; set; } = string.Empty;
    [PersonalIdentifiableInformation("Uniquely identifies
      the employee")]
    public string SocialSecurityNumber { get; set; } =
      string.Empty;
    public IEnumerable<JournalEntry> JournalEntries { get;
      set; } = Enumerable.Empty<JournalEntry>();
}
```

现在运行程序应该会给你一个输出，其中在顶部包括类型级别的信息：

```cs
Checking type for compliance rules: Chapter11.Patient
Type level - The data is confidential
Property: FirstName - Employment records
Property: LastName - Employment records
Property: SocialSecurityNumber - Uniquely identifies the employee
Property JournalEntries is a collection of type Chapter11.JournalEntry with type level metadata
JournalEntries - Journal entry
```

你现在创建了一个灵活且可扩展的系统。你不必进入“引擎”来执行任何更改以引入新功能。这是一个强大的功能，也是健康系统的特征。

# 摘要

开发始终如一的软件很困难。能够经受住时间考验、不失去可维护性，并且能够在合理的时间内允许开发新功能的软件更加困难。将开发新业务价值所需的时间保持接近恒定是我们追求的目标。这使得企业能够更容易地确定新功能的影响。这也使得这种影响对开发者来说更加可预测。

为了达到这个目的，有一些技术、模式、实践和原则可以提供帮助。在我看来，以可扩展的方式思考并设计代码以避免变得僵化是关键。这样，大多数时候，我们可以专注于添加新功能，而不是不得不进行心脏手术般的修改才能添加新功能。

在下一章中，我们将探讨如何将本章所讨论的内容与你在*第十章*“约定优于配置”中开始实践的内容相结合。约定可以采取多种形式，并且它们可以帮助你超越 C#和.NET 的继承模型。
