# 接下来是什么？

在这本书中，我们发现了 MVVM 架构模式，并探讨了利用该模式的优势开发 WPF 应用程序的过程，同时遵循其**关注点分离**的原则。我们调查了各种在不同应用程序层之间进行通信和构建我们的代码库的方法。

重要的是，我们考虑了多种调试我们的 WPF 应用程序和追踪编码问题的方法。特别是，我们揭示了一些技巧和窍门，帮助我们识别数据绑定错误的根源。此外，我们还学习了如何通过查看跟踪信息来帮助我们检测问题，即使是在我们的应用程序部署之后。

我们继续研究利用应用程序框架的好处，并开始设计和开发我们自己的框架。我们以不将我们的框架绑定到任何特定功能或技术的方式构建它，并尝试了多种封装所需功能的方法。

我们专门用了一章来探讨数据绑定这一基本艺术，详细研究了依赖属性和附加属性的创建。我们研究了设置依赖属性元数据，并介绍了关键的依赖属性设置优先级列表。然后我们涵盖了标准和层次数据模板，并研究了几个有趣的数据绑定示例。

调查内置 WPF 控件丰富的继承层次结构使我们能够看到它们的功能是如何从层次结构中的每个后续基类构建起来的。这反过来又使我们能够看到在某些情况下某些控件比其他控件更好用。我们还了解了如何自定义内置控件，并考虑了如何最好地创建我们自己的控件。

虽然在 WPF 应用程序中动画的可能性几乎是无限的，但我们调查了更实用的选项，主要关注 XAML 中使用的语法。然后我们直接在我们的应用程序框架中添加了动画功能，这样开发者就可以轻松使用。

在将注意力转向我们应用程序的外观之后，我们调查了多种技术，例如无边框窗口以及为更高级的方法添加阴影和发光效果，以使我们的应用程序在众多应用程序中脱颖而出。我们还把动画融入我们的日常控件中，以便为我们的应用程序带来一种独特感。

我们彻底调查了.NET Framework 为我们提供的数据验证选项，主要集中在前两个可用的验证接口，并探索了多种实现方式。我们探讨了高级技术，如多级验证和使用数据注释属性，然后在我们的应用程序框架中添加了一个完整的验证系统。

我们进一步扩展了我们的应用程序框架，加入了一个异步数据操作系统，该系统结合了一个完整的用户反馈组件，包括一个动画反馈显示机制。我们继续研究如何提供应用程序内的帮助和用户偏好，并实现耗时功能以节省用户的时间和精力。

我们还探索了多种可以用来提高我们的 WPF 应用程序性能的选项，从更有效地声明资源到使用更轻量级的控件和更高效的渲染绘图、图像和文本的方法。我们看到了更高效的数据绑定方法，并发现了解除事件处理程序的重要性。

最后，我们研究了任何专业应用程序开发中的最后一个任务，即部署。我们考虑了多种替代方法，但主要关注最流行的 ClickOnce 技术。我们研究了 ClickOnce 部署是如何进行的，以及我们如何在隔离存储中安全地存储和访问数据。我们以多种方式结束，这些方式可以让我们访问.NET 中可用的各种应用程序版本。

总体来说，我们涵盖了大量的信息，这些信息共同将使我们能够创建高效、视觉上吸引人、高度可用和高度生产力的 WPF 应用程序。更重要的是，我们现在有了自己的应用程序框架，我们可以为每个新创建的应用程序重复使用它。*那么，接下来是什么？*

# 将注意力转向未来的项目

你可以将这本书中的概念和想法应用到其他领域，并继续在这些新领域中实验和探索它们的效果。例如，我们学习了关于`Adorner`对象的知识，因此你可以利用这一新发现的知识在主窗口的 adorner 层中实现一些视觉反馈，用于常见的拖放功能。

然后，你可以进一步扩展这个想法，使用你关于附加属性所发现的知识，完全封装这个拖放功能，使利用你的应用程序框架的开发者能够以属性方式使用这个功能。

例如，你可以创建一个`DragDropProperties`类，声明附加属性，如`IsDragSource`、`IsDragTarget`、`DragEffects`、`DragDropType`和`DropCommand`，并且它可以由你的相关附加属性类扩展，例如`ListBoxProperties`类。

然后，你可以在`DragDropProperties`类中声明一个`BaseDragDropManager`类，用于将一切连接起来，通过附加和移除适当的事件处理程序，启动拖放过程，通过拖放效果更新光标，并在光标在屏幕上移动时执行分配给`DropCommand`属性的`ICommand`对象。

这导致了一个可以进一步扩展的领域。我们不仅可以在附加属性中处理 UI 事件，还可以将它们组合起来执行更复杂的功能。例如，假设我们有一个类型为`string`的附加属性，名为`Label`。

当此属性被设置时，它可以将资源中的特定`ControlTemplate`元素应用到当前`TextBox`对象的`Template`属性。此模板可以显示此属性中的文本在一个次要文本元素中，因此充当内部标签。当`TextBox`对象有值时，标签文本元素可以通过扩展我们的`BaseVisibilityConverter`类的`IValueConverter`实现来隐藏：

```cs
<TextBlock Text="{Binding (Attached:TextBoxProperties.Label),
  RelativeSource={RelativeSource AncestorType=TextBox}, FallbackValue=''}"
  Foreground="{Binding (Attached:TextBoxProperties.LabelColor), 
  RelativeSource={RelativeSource AncestorType=TextBox}, 
  FallbackValue=#FF000000}" Visibility="{Binding Text, 
  RelativeSource={RelativeSource AncestorType=TextBox},  
  Converter={StaticResource StringToVisibilityConverter},  
  FallbackValue=Collapsed}" ... />  
```

如前例所示，我们还可以声明另一个附加属性，名为`LabelColor`，类型为`Brush`，它指定了当设置`Label`附加属性时要使用的颜色。请注意，如果未设置`LabelColor`属性，则它将使用默认值，如果设置了，或者使用`FallbackValue`属性中指定的值。

# 改进我们的应用程序框架

你还可以继续在自定义应用程序框架方面进行工作，并使其适应你的个别需求。考虑到这一点，你可以在外部资源文件中构建一个完整的自定义控件集合，具有特定的外观和感觉，以便在所有应用程序中使用。

本书还提供了许多其他示例，这些示例可以轻松扩展。例如，你可以更新我们的`DependencyManager`类，以使每个接口可以注册多个具体类。

而不是使用`Dictionary<Type, Type>`对象来存储我们的注册信息，你可以定义新的自定义对象。你可以声明一个具有`Type`属性和`object`数组以存储可能需要的任何构造函数输入参数的`ConcreteImplementation`结构体：

```cs
public ConcreteImplementation(Type type, 
  params object[] constructorParameters) 
{ 
  Type = type; 
  ConstructorParameters = constructorParameters; 
} 
```

你可以声明一个`DependencyRegistration`类，用于将接口类型与具体实现集合配对：

```cs
public DependencyRegistration(Type interfaceType,  
  IEnumerable<ConcreteImplementation> concreteImplementations) 
{ 
  if (!concreteImplementations.All(c =>
    interfaceType.IsAssignableFrom(c.Type)))
    throw new ArgumentException("The System.Type object specified by the 
    ConcreteImplementation.Type property must implement the interface type 
    specified by the interfaceType input parameter.", 
    nameof(interfaceType));
  ConcreteImplementations = concreteImplementations; 
  InterfaceType = interfaceType; 
} 
```

在我们的`DependencyManager`类中，你可以将`registeredDependencies`字段的类型更改为`DependencyRegistration`类型集合。然后，当前的`Register`和`Resolve`方法也可以更新为使用此新集合类型。

或者，你可以包含其他常见的功能，这些功能包含在流行的依赖注入和控制反转容器中，例如在程序集级别自动注册具体类到接口。为此，你可以使用一些基本的反射：

```cs
using System.Reflection;

...

public void RegisterAllInterfacesInAssemblyOf<T>() where T : class 
{ 
  Assembly assembly = typeof(T).Assembly; 
  IEnumerable<Type> interfaces = 
    assembly.GetTypes().Where(p => p.IsInterface); 
  foreach (Type interfaceType in interfaces) 
  { 
    IEnumerable<Type> implementingTypes = assembly.GetTypes().
      Where(p => interfaceType.IsAssignableFrom(p) && !p.IsInterface); 
    ConcreteImplementation[] concreteImplementations = implementingTypes.
      Select(t => new ConcreteImplementation(t, null)).ToArray();       
    if (concreteImplementations != null && concreteImplementations.Any()) 
      registeredDependencies.Add(interfaceType, concreteImplementations); 
  } 
} 
```

此方法首先访问包含泛型类型参数的组件，然后获取该组件中的接口集合。然后它遍历接口集合，找到实现每个接口的类集合，并为每个匹配项实例化一个`ConcreteImplementation`元素。每个匹配项都被添加到`registeredDependencies`集合中，并与其相关的接口类型相关联。

以这种方式，你可以将我们的`Models`、`Managers`和`ViewModels`项目中的任何接口类型传递过去，以自动注册它们组件中找到的所有接口和具体类。在更大的应用程序中这样做有明显的优势，因为它意味着你不必手动注册每个类型：

```cs
private void RegisterDependencies() 
{ 
  DependencyManager.Instance.ClearRegistrations(); 
  DependencyManagerAdvanced.Instance. 
    RegisterAllInterfacesInAssemblyOf<IDataProvider>(); 
  DependencyManagerAdvanced.Instance. 
    RegisterAllInterfacesInAssemblyOf<IUiThreadManager>(); 
  DependencyManagerAdvanced.Instance. 
    RegisterAllInterfacesInAssemblyOf<IUserViewModel>(); 
} 
```

此外，你可以声明另一个方法，用于注册由泛型类型参数`T`指定的类型的组件中找到的所有类型，其中找到实现接口的匹配项。这可以在测试期间使用，这样你就可以在测试期间只需传递模拟项目中的任何类型，再次节省时间和精力：

```cs
DependencyManager.Instance.
  RegisterAllConcreteImplementationsInAssemblyOf<MockUiThreadManager>(); 
```

就像所有严肃的开发项目一样，我们需要测试构成代码库的代码。这样做显然有助于减少应用程序中的错误数量，但也会在添加新代码的同时提醒我们现有功能是否被破坏。它们还为重构提供了一个安全网，使我们能够持续改进我们的设计，同时确保现有功能不会被破坏。

因此，你可以在应用程序中改进的一个领域是实施一个完整的测试套件。这本书已经解释了在测试期间替换代码的多种方法，并且这种模式可以很容易地扩展。如果一个管理类使用某种在测试期间不能使用的资源，那么你可以为它创建一个接口，添加一个模拟类，并在运行时和测试期间使用`DependencyManager`类来实例化相关的具体实现。

书中可以扩展的另一个领域与我们的`AnimatedStackPanel`类有关。你可以从这个类中提取可重用的属性和动画代码到一个`AnimatedPanel`基类，以便它可以服务于多种不同类型的动画面板。

如第七章《精通实用动画》中建议的，你可以进一步扩展基类，通过公开额外的动画属性，使用户能够对提供的动画有更多的控制。例如，你可以添加对齐、方向、持续时间以及/或动画类型属性，以便框架的用户能够使用广泛的动画选项。

这些属性可以在进入和退出动画之间分配，以实现对其的独立控制。通过在基类中提供广泛的这些附加属性，你可以极大地简化添加新动画面板的过程。

例如，你可以通过简单地扩展基类来添加一个新的`AnimatedWrapPanel`，或者可能是一个`AnimatedColumnPanel`，只需要在新面板中实现两个`MeasureOverride`和`ArrangeOverride`方法。

# 记录错误

在本书的代码示例中，你可能在多个地方看到了`Log error`注释。一般来说，记录错误不仅是良好的实践，而且还可以帮助你追踪错误并提高你应用程序用户的整体用户体验。

记录错误最简单的地方是`Errors`数据库，你想要存储的最少有用信息字段应包括当前用户的详细信息、错误发生的时间、异常消息、堆栈跟踪以及它发生的组件或区域。这个字段可以在异常的`TargetSite`属性的`Module`属性中找到：

```cs
public Error(Exception exception, User createdBy)  
{ 
  Id = Guid.NewGuid(); 
  Message = FlattenInnerExceptions(exception); 
  StackTrace = exception.StackTrace; 
  Area = exception.TargetSite.Module.ToString(); 
  CreatedOn = DateTime.Now; 
  CreatedBy = createdBy; 
} 
```

注意使用自定义的`FlattenInnerExceptions`方法，该方法还会输出抛出异常可能包含的任何内部异常的消息。构建自己的`FlattenInnerExceptions`方法的另一个选择是简单地保存异常的`ToString`输出，这将包含任何内部异常的详细信息，尽管它也会包含堆栈跟踪和其他信息。

# 使用在线资源

作为最后的注意事项，如果你还不熟悉**Microsoft Docs**网站，你真的应该熟悉一下。它是为 Microsoft 开发者社区维护的，包括从各种语言的详细 API、教程演练和代码示例，到软件下载等所有内容。

它可以在[`docs.microsoft.com`](https://docs.microsoft.com)上找到，并且当出现关于.NET 中各种类成员的问题时，应该是你首先查看的地方。如果你在它们的 API 中没有找到所需的信息，你可以在它们的论坛中提问，并迅速从社区和 Microsoft 员工那里获得答案。

另一个伟大的开发者资源是**Stack Overflow**开发专业人士问答网站，我在有时间的时候仍然会回答问题。它可以在[`stackoverflow.com/`](http://stackoverflow.com/)上找到，社区成员通常在几秒钟内提供答案，这确实很难超越，并且是周围最好的开发论坛之一。

对于更多教程，请访问[`www.wpftutorial.net/`](https://www.wpftutorial.net/)网站，在那里你可以找到从基础到复杂的丰富教程。对于有趣和创新的可下载自定义控件和额外教程，请尝试访问 Code Project 网站的 WPF 部分[`www.codeproject.com/kb/wpf/`](https://www.codeproject.com/kb/wpf/)。

现在剩下的只是祝愿你在未来的应用程序开发以及你蒸蒸日上的职业生涯中一切顺利。
