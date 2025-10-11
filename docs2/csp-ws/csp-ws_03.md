# 第三章：3. 委托、事件和 Lambda

概述

在本章中，你将学习如何定义和调用委托，并探索它们在.NET 生态系统中的广泛使用。有了这些知识，你将转向内置的`Action`和`Func`委托，以了解它们的用法如何减少不必要的样板代码。然后，你将看到如何利用多播委托向多个参与者发送消息，以及如何将事件集成到事件驱动代码中。在这个过程中，你将发现一些常见的陷阱和最佳实践，以防止一个优秀应用程序变成不可靠的应用程序。

本章将揭开 lambda 语法风格的神秘面纱，并展示如何有效地使用它。到本章结束时，你将能够舒适地使用 lambda 语法来创建简洁、易于理解和维护的代码。

# 简介

在上一章中，你学习了面向对象编程（OOP）的一些关键方面。在本章中，你将通过查看 C#中用于使类相互交互的常见模式来在此基础上进行构建。

你是否发现自己正在处理必须监听某些信号并在其上采取行动的代码，但你无法确定这些行动应该在运行时是什么？也许你有一段需要重用或传递给其他方法的代码块，以便它们在准备好时调用。或者，你可能想过滤一组对象，但需要根据用户偏好的组合来决定如何进行。许多这样的功能可以通过使用接口来实现，但通常更有效的方法是创建可以以类型安全的方式传递给其他类的代码块。这些块被称为委托，是许多.NET 库的骨架，允许将方法或代码片段作为参数传递。

委托的自然扩展是事件，这使得在软件中提供一种可选行为成为可能。例如，你可能有一个广播实时新闻和股票价格的组件，但除非你提供一种方式来选择加入这些服务，否则你可能限制了该组件的可使用性。

用户界面（UI）应用程序通常提供各种用户动作的通知，例如按键、滑动屏幕或点击鼠标按钮；这些通知在 C#中遵循一个标准模式，将在本章中详细讨论。在这种情况下，检测此类动作的 UI 元素被称为发布者，而执行这些消息的代码被称为订阅者。当它们结合在一起时，形成了一种称为发布者-订阅者或 pub-sub 的事件驱动设计。你将看到如何在所有类型的 C#中使用它。记住，它的使用并不局限于 UI 应用程序。

最后，你将学习关于 lambda 语句和 lambda 表达式，统称为 lambda。这些具有不寻常的语法，一开始可能需要一段时间才能适应。与在类中分散许多方法和函数不同，lambda 允许更小的代码块，通常是自包含的，并且位于代码中使用它们的附近，从而提供了一种更容易跟踪和维护代码的方法。你将在本章的后半部分详细了解 lambda。首先，你将学习关于委托（delegates）的内容。

# 委托

.NET 委托与其他语言中找到的函数指针类似，例如 C++；换句话说，它是一个指向将在运行时调用的方法的指针。本质上，它是一个代码块的占位符，这可能是一个简单的语句，也可能是一个完整的、多行的代码块，包括复杂的执行分支，你要求其他代码在某个时间点执行。术语“委托”暗示了一种形式的“代表”，这正是这个占位符概念所关联的。

委托允许对象之间的最小耦合和更少的代码。不需要创建从特定类或接口派生的类。通过使用委托，你定义了一个兼容方法的外观，无论它是在类或结构体中，静态的还是基于实例的。参数和返回类型定义了这个调用兼容性。

此外，委托可以以回调的方式使用，这允许将多个方法连接到单个发布源。它们通常需要的代码更少，并且提供的功能比基于接口的设计更多。

以下示例显示了委托的有效性。假设你有一个通过姓氏搜索用户的类。它可能看起来像这样：

```cs
public User FindBySurname(string name)
{
    foreach(var user in _users)
       if (user.Surname == name)
          return user;
    return null;
}
```

然后你需要扩展这个功能，包括搜索用户的登录名：

```cs
public User FindByLoginName(string name)
{
    foreach(var user in _users)
       if (user.LoginName == name)
          return user;
    return null;
}
```

再次，你决定添加另一个搜索，这次是通过位置：

```cs
public User FindByLocation(string name)
{
    foreach(var user in _users)
       if (user.Location == name)
          return user;
    return null;
}
```

你开始搜索的代码如下：

```cs
public void DoSearch()
{
  var user1 = FindBySurname("Wright");
  var user2 = FindByLoginName("JamesR");
  var user3 = FindByLocation("Scotland"); 
}
```

你能看出每次发生的是什么模式吗？你正在重复相同的代码，该代码遍历用户列表，应用布尔条件（也称为谓词）以找到第一个匹配的用户。

唯一不同的是，谓词（predicate）决定是否找到了匹配项。这是委托在基本级别使用的一个常见案例。`predicate`可以用一个委托替换，作为占位符，在需要时进行评估。

将此代码转换为委托风格，你定义一个名为`FindUser`的委托（这一步可以跳过，因为.NET 包含一个可以重用的委托定义；你将在稍后了解这一点）。

你只需要一个辅助方法，`Find`，它接受一个`FindUser`委托实例。`Find`知道如何遍历用户，调用委托并传入用户，以返回匹配的布尔值：

```cs
private delegate bool FindUser(User user);
private User Find(FindUser predicate)
{
  foreach (var user in _users)
    if (predicate(user))
      return user;
  return null;
}
public void DoSearch()
{
  var user4 = Find(user => user.Surname == "Wright");
  var user5 = Find(user => user.LoginName == "JamesR");
  var user6 = Find(user => user.Location == "Scotland");
}
```

如你所见，代码现在更加紧凑。不需要剪切和粘贴遍历用户的代码，因为所有这些都在一个地方完成。对于每种类型的搜索，你只需定义一个 delegate 并将其传递给 `Find`。要添加新的搜索类型，你只需在单行语句中定义它，而不是复制至少八行重复的循环函数。

Lambda 语法是定义方法体的一种基本风格，但它的奇怪语法可能会在最初成为障碍。乍一看，Lambda 表达式可能看起来很奇怪，因为它们有 `=>` 风格，但它们确实提供了一种更简洁的方式来指定目标方法。定义 Lambda 的行为与定义方法类似；你基本上省略了方法名，并使用 `=>` 作为代码块的占位符。

现在，你将查看另一个示例，这次使用接口。假设你正在开发一个图形引擎，并且需要在用户每次旋转或缩放时计算屏幕上图像的位置。注意，此示例跳过了任何复杂的数学计算。

考虑到你需要使用 `ITransform` 接口和名为 `Move` 的单个方法来转换 `Point` 类，如下面的代码片段所示：

```cs
public class Point
{
  public double X { get; set; } 
  public double Y { get; set; }
}
public interface ITransform
{
  Point Move(double height, double width);
}
```

当用户旋转一个对象时，你需要使用 `RotateTransform`，而对于缩放操作，你将使用 `ZoomTransform`，如下所示。两者都基于 `ITransform` 接口：

```cs
public class RotateTransform : ITransform
{
    public Point Move(double height, double width)
    {
        // do stuff
        return new Point();
    }
}
public class ZoomTransform : ITransform
{
    public Point Move(double height, double width)
    {
        // do stuff
        return new Point();
    }
}
```

因此，给定这两个类，可以通过创建一个新的 `Transform` 实例并将其传递给名为 `Calculate` 的方法来转换一个点，如下面的代码所示。`Calculate` 调用相应的 `Move` 方法，并在点上进行一些额外的未指定工作，然后返回点给调用者：

```cs
public class Transformer
{
    public void Transform()
    {
        var rotatePoint = Calculate(new RotateTransform(), 100, 20);
        var zoomPoint = Calculate(new ZoomTransform(), 5, 5);
    }
    private Point Calculate(ITransform transformer, double height, double width)
    {
        var point = transformer.Move(height, width);
        //do stuff to point
        return point;
    }
}
```

这是一个基于类和接口的标准设计，但你也可以看到，你为了从 `Move` 方法中仅使用一个数值就创建了大量的新类。将计算分解成易于遵循的实现是一个值得考虑的想法。毕竟，如果在一个方法中实现多个 if-then 分支，可能会引起未来的维护问题。

通过重新实现基于 delegate 的设计，你仍然拥有可维护的代码，但需要照看的部分要少得多。你可以有一个 `TransformPoint` delegate 和一个可以传递 `TransformPoint` delegate 的新 `Calculate` 函数。

你可以通过在其名称周围添加括号并传递任何参数来调用 delegate。这与调用标准类级别的函数或方法的方式相似。你将在稍后详细介绍这种调用；现在，考虑以下代码片段：

```cs
    private delegate Point TransformPoint(double height, double width);
    private Point Calculate(TransformPoint transformer, double height, double width)
    {
        var point = transformer(height, width);
        //do stuff to point
        return point;
    }
```

你仍然需要实际的 `Rotate` 和 `Zoom` 方法，但不需要创建不必要的类来执行此操作。你可以添加以下代码：

```cs
    private Point Rotate(double height, double width)
    {
        return new Point();
    }
    private Point Zoom(double height, double width)
    {
        return new Point();
    }
```

现在，调用名为 delegate 的方法就像以下这样简单：

```cs
    public void Transform()
    {
         var rotatePoint1 = Calculate(Rotate, 100, 20);
         var zoomPoint1 = Calculate(Zoom, 5, 5);
    }
```

注意使用 delegate 的这种方式如何帮助消除大量不必要的代码。

注意

你可以在[`packt.link/AcwZA`](https://packt.link/AcwZA)找到用于此示例的代码。

除了调用单个占位符方法外，委托还包含额外的管道，允许它以**多播**方式使用，即一种将多个目标方法链接在一起的方式，每个方法依次被调用。这通常被称为调用列表或委托链，由充当发布源代码启动。

如何将这种多播概念应用于 UI 的一个简单例子可以在 UI 中看到。想象一下，你有一个显示一个国家地图的应用程序。当用户将鼠标移到地图上时，你可能想执行以下各种操作：

+   当鼠标悬停在建筑物上时，将鼠标指针更改为不同的形状。

+   显示一个计算真实世界经纬度坐标的工具提示。

+   在状态栏中显示计算鼠标悬停区域人口的消息。

要实现这一点，你需要一种方法来检测用户何时将鼠标移到屏幕上。这通常被称为发布者。在这个例子中，它的唯一目的是检测鼠标移动并将它们发布给任何正在监听的人。

要执行所需的三个 UI 操作，你需要创建一个类，该类具有一个对象列表，当鼠标位置改变时通知这些对象，允许每个对象独立于其他对象执行所需的任何活动。这些对象中的每一个都被称为订阅者。

当你的发布者检测到鼠标已移动时，你将遵循以下伪代码：

```cs
MouseEventArgs args = new MouseEventArgs(100,200)
foreach(subscription in subscriptionList)
{
   subscription.OnMouseMoved(args)
} 
```

这假设`subscriptionList`是一个对象列表，可能基于具有`OnMouseMoved`方法的接口。添加代码以使感兴趣方能够订阅和取消订阅`OnMouseMoved`通知取决于你。如果之前已订阅的代码没有取消订阅的方式，并且在不再需要调用时反复被调用，这将是一个不幸的设计。

在前面的代码中，发布者和订阅者之间存在相当多的耦合，你将回到使用接口进行类型安全实现。如果你还需要监听按键，包括按键按下和按键释放，很快就会感到非常沮丧，需要反复复制如此相似的代码。

幸运的是，委托类型包含所有这些内置行为。你可以单目标或多目标方法互换使用；你所需要做的就是调用一个委托，委托将为你处理其余部分。

你将很快深入了解多播委托，但首先，你将探索单目标方法场景。

## 定义自定义委托

委托的定义方式与标准方法类似。编译器不关心目标方法体中的代码，只关心它可以在某个时间点安全地被调用。

使用以下格式定义 `delegate` 关键字：

```cs
public delegate void MessageReceivedHandler(string message, int size);
```

以下列表描述了此语法的每个组成部分：

+   范围：一个访问修饰符，例如 `public`、`private` 或 `protected`，用于定义委托的作用域。如果你不包括修饰符，编译器将默认将其标记为 `private`，但总是最好明确地显示你代码的意图。

+   `delegate` 关键字。

+   返回类型：如果没有返回类型，则使用 `void`。

+   委托名称：这可以是任何你喜欢的名称，但名称必须在命名空间内是唯一的。许多命名约定（包括微软的）建议将 `Handler` 或 `EventHandler` 添加到你的委托名称中。

+   如果需要，则提供参数。

    注意

    委托可以嵌套在类或命名空间内；它们也可以在全局命名空间中定义，尽管这种做法是不被推荐的。在 C# 中定义类时，常见的做法是在父命名空间中定义它们，通常基于以公司名称开始，然后是产品名称，最后是功能的分层约定。这有助于为类型提供更独特的标识。

    通过在没有任何命名空间的情况下定义委托，如果它也在没有命名空间保护的库中定义了具有相同名称的另一个委托，那么它很可能与另一个委托发生冲突。这可能导致编译器混淆，不知道你指的是哪个委托。

在 .NET 的早期版本中，定义自定义委托是一种常见的做法。此类代码后来被各种内置的 .NET 委托所取代，你将在稍后查看。现在，你将简要介绍定义自定义委托的基础知识。如果你维护任何遗留的 C# 代码，了解这一点是很有价值的。

在下一个练习中，你将创建一个自定义委托，它接受一个 `DateTime` 参数并返回一个布尔值以指示有效性。

## 练习 3.01：定义和调用自定义委托

假设你有一个允许用户订购产品的应用程序。在填写订单详情时，客户可以指定订单日期和交货日期，这两个日期在接受订单之前都必须经过验证。你需要一种灵活的方式来验证这些日期。对于某些客户，你可能允许周末的交货日期，而对于其他客户，交货日期至少必须提前七天。你还可以允许某些客户的订单日期倒退。

你知道委托提供了一种在运行时改变实现的方式，因此这是最佳的做法。你不想使用多个接口，或者更糟糕的是，一个复杂的 `if-then` 语句的混乱，来实现这一点。

根据客户的配置文件，你可以创建一个名为 `Order` 的类，该类可以传递不同的日期验证规则。这些规则可以通过一个 `Validate` 方法进行验证：

执行以下步骤来完成此操作：

1.  创建一个名为 `Chapter03` 的新文件夹。

1.  切换到`Chapter03`文件夹，并使用 CLI `dotnet`命令创建一个新的控制台应用程序，名为`Exercise01`，如下所示：

    ```cs
    source\Chapter03>dotnet new console -o Exercise01
    ```

你将看到以下输出：

```cs
The template "Console Application" was created successfully.
Processing post-creation actions...
Running 'dotnet restore' on Exercise01\Exercise01.csproj...
  Determining projects to restore...
  Restored source\Chapter03\Exercise01\Exercise01.csproj (in 191 ms).
Restore succeeded.
```

1.  打开`Chapter03\Exercise01.csproj`并将内容替换为以下设置：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
      </PropertyGroup>
    </Project>
    ```

1.  打开`Exercise01\Program.cs`并清除内容。

1.  使用命名空间来防止与其他库中的对象冲突的偏好已经在前面提到过，因此为了保持独立，使用`Chapter03.Exercise01`作为命名空间。

要实现你的日期验证规则，你将定义一个接受单个`DateTime`参数并返回布尔值的代理。你将把它命名为`DateValidationHandler`：

```cs
using System;
namespace Chapter03.Exercise01 
{
    public delegate bool DateValidationHandler(DateTime dateTime);
}
```

1.  接下来，你将创建一个名为`Order`的类，它包含订单的详细信息，并且可以被传递给两个日期验证代理：

    ```cs
       public class Order
        {
            private readonly DateValidationHandler _orderDateValidator;
            private readonly DateValidationHandler _deliveryDateValidator;
    ```

注意你如何声明了两个只读的类级别实例`DateValidationHandler`，一个用于验证订单日期，另一个用于验证交付日期。这种设计假设对于这个`Order`实例，日期验证规则不会改变。

1.  现在是构造函数，你传递这两个代理：

    ```cs
           public Order(DateValidationHandler orderDateValidator,
                DateValidationHandler deliveryDateValidator)
            {
                _orderDateValidator = orderDateValidator;
                _deliveryDateValidator = deliveryDateValidator;
            }  
    ```

在这个设计中，通常有一个不同的类负责根据所选客户的配置文件决定使用哪个代理。

1.  你需要添加两个待验证的日期属性。这些日期可以使用一个监听按键并直接将用户编辑应用到这个类的 UI 来设置：

    ```cs
            public DateTime OrderDate { get; set; }
            public DateTime DeliveryDate { get; set; }
    ```

1.  现在添加一个`IsValid`方法，将`OrderDate`传递给`orderDateValidator`代理，将`DeliveryDate`传递给`deliveryDateValidator`代理：

    ```cs
            public bool IsValid() => 
                _orderDateValidator(OrderDate) &&
                _deliveryDateValidator(DeliveryDate);
        }
    ```

如果两者都有效，那么这个调用将返回`true`。关键在于`Order`不需要知道单个客户的日期验证规则的确切实现，因此你可以在程序的其他地方轻松重用`Order`。要调用代理，你只需将任何参数用括号括起来，在这种情况下，将正确的日期属性传递给每个代理实例：

1.  要创建一个控制台应用程序来测试这个，添加一个名为`Program`的`static`类：

    ```cs
        public static class Program
        {
    ```

1.  你想要创建两个函数来验证传递给它们的日期是否有效。这些函数将构成你的代理目标方法的基础：

    ```cs
            private static bool IsWeekendDate(DateTime date)
            {
                Console.WriteLine("Called IsWeekendDate");
                return date.DayOfWeek == DayOfWeek.Saturday ||
                       date.DayOfWeek == DayOfWeek.Sunday;
            }
            private static bool IsPastDate(DateTime date)
            {
                Console.WriteLine("Called IsPastDate");
                return date < DateTime.Today;
            }
    ```

注意它们都有`DateValidationHandler`代理期望的确切签名。它们两个都不知道它们正在验证的日期的性质，因为这不是它们的关注点。由于它们在这个类的任何地方都不与任何变量或属性交互，所以它们都被标记为`static`。

1.  现在是`Main`入口点。在这里，你创建了两个`DateValidationHandler`代理实例，将`IsPastDate`传递给一个，将`IsWeekendDate`传递给另一个。这些是当每个代理被调用时将被调用的目标方法：

    ```cs
            public static void Main()
            {
               var orderValidator = new DateValidationHandler(IsPastDate);
               var deliverValidator = new DateValidationHandler(IsWeekendDate);
    ```

1.  现在你可以创建一个`Order`实例，传递代理并设置订单和交付日期：

    ```cs
              var order = new Order(orderValidator, deliverValidator)
                {
                    OrderDate = DateTime.Today.AddDays(-10), 
                    DeliveryDate = new DateTime(2020, 12, 31)
                };
    ```

创建委托有多种方式。在这里，您首先将它们分配给变量，以使代码更清晰（您将在后面了解不同的风格）。

1.  现在只需在控制台中显示日期并调用`IsValid`即可，这将依次调用您的每个委托方法。请注意，使用了自定义日期格式，以便使日期更易于阅读：

    ```cs
              Console.WriteLine($"Ordered: {order.OrderDate:dd-MMM-yy}");
              Console.WriteLine($"Delivered: {order.DeliveryDate:dd-MMM-yy }");
              Console.WriteLine($"IsValid: {order.IsValid()}");
            }
        }
    }
    ```

1.  运行控制台应用程序的输出如下：

    ```cs
    Ordered: 07-May-22
    Delivered: 31-Dec-20
    Called IsPastDate
    Called IsWeekendDate
    IsValid: False
    ```

此顺序**不**有效，因为交付日期是星期四，而不是您所需的周末：

您已经学会了如何定义自定义委托，并创建了两个实例，这些实例使用小的辅助函数来验证日期。这使您了解委托是多么灵活。

注意

您可以在[`packt.link/cmL0s`](https://packt.link/cmL0s)找到用于此练习的代码。

## 内置的 Action 和 Func 委托

当您定义一个委托时，您正在描述其签名，即返回类型和输入参数列表。话虽如此，考虑这两个委托：

```cs
public delegate string DoStuff(string name, int age);
public delegate string DoMoreStuff(string name, int age);
```

它们具有相同的签名，但仅名称不同，这就是为什么您可以在调用时声明每个实例，并使它们**都**指向**相同**的目标方法：

```cs
public static void Main()
{
    DoStuff stuff = new DoStuff(MyMethod);
    DoMoreStuff moreStuff = new DoMoreStuff(MyMethod);
    Console.WriteLine($"Stuff: {stuff("Louis", 2)}");
    Console.WriteLine($"MoreStuff: {moreStuff("Louis", 2)}");
}
private static string MyMethod(string name, int age)
{
    return $"{name}@{age}";
}
```

运行控制台应用程序在两次调用中都产生相同的结果：

```cs
Stuff: Louis@2
MoreStuff: Louis@2
```

注意

您可以在[`packt.link/r6B8n`](https://packt.link/r6B8n)找到用于此示例的代码。

如果您能省去定义`DoStuff`和`DoMoreStuff`委托，并使用具有完全相同签名的更通用委托，那就太好了。毕竟，在先前的代码片段中，您创建`DoStuff`或`DoMoreStuff`委托都没有关系，因为它们都调用相同的目标方法。

.NET 实际上提供了各种内置委托，您可以直接使用它们，从而节省了您自己定义这些委托的努力。这些是`Action`和`Func`委托。

`Action`和`Func`委托的组合有很多种可能，每种组合都允许使用更多参数。您可以从 0 到 16 种不同的参数类型中进行选择。由于组合众多，您几乎不需要定义自己的委托类型。

值得注意的是，`Action`和`Func`委托是在.NET 的后续版本中添加的，因此自定义委托的使用通常在较老的遗留代码中找到。您不需要自己创建新的委托。

在以下代码片段中，`MyMethod`使用三个参数的`Func`变体调用；您很快就会了解这个看起来很奇怪的`<string, int, string>`语法：

```cs
Func<string, int, string> funcStuff = MyMethod;
Console.WriteLine($"FuncStuff: {funcStuff("Louis", 2)}");
```

这产生的返回值与前面的两次调用相同：

```cs
FuncStuff: Louis@2
```

在你继续探索 `Action` 和 `Func` 委托之前，探索一下 `Action<string, int, string>` 语法可能会有所帮助。这种语法允许使用类型参数来定义类和方法。这些被称为泛型，并作为特定类型的占位符。在 *第四章*，*数据结构和 LINQ* 中，你会更详细地了解泛型，但在这里用 `Action` 和 `Func` 委托总结它们的用法是值得的。

`Action` 委托的非泛型版本在 .NET 中如下预定义：

```cs
public delegate void Action()
```

如你所知，从你对委托的早期了解来看，这是一个不接受任何参数且没有返回类型的委托；这是可用的最简单类型的委托。

与.NET 中预定义的泛型 `Action` 委托之一进行对比：

```cs
public delegate void Action<T>(T obj)
```

你可以看到这包括一个 `<T>` 和 `T` 参数部分，这意味着它接受一个受限于字符串的 `Action`，它接受单个字符串参数并返回无值，如下所示：

```cs
Action<string> actionA;
```

那么一个受限于 `int` 的版本呢？这也没有返回类型，并且接受单个 `int` 参数：

```cs
Action<int> actionB;
```

你能在这里看到模式吗？本质上，你指定的类型可以在编译时用来声明一个类型。如果你想用两个、三个、四个……或者 16 个参数呢？很简单。有 `Action` 和 `Func` 泛型类型可以接受多达 **16** 种不同的参数类型。你不太可能编写需要超过 16 个参数的代码。

这个接受两个参数的 `Action` 以 `int` 和 `string` 作为参数：

```cs
Action<int, string> actionC;
```

你可以把它转过来。这里有一个接受两个参数的 `Action`，但这个 `Action` 接受一个 `string` 参数和一个 `int` 参数：

```cs
Action<string, int> actionD;
```

这些涵盖了大多数参数组合，所以你可以看到，创建自己的委托类型是非常罕见的。

同样的规则适用于返回值的委托；这就是 `Func` 类型被使用的地方。泛型 `Func` 类型以单个值类型参数开始：

```cs
public delegate T Func<T>()
```

在下面的示例中，`funcE` 是一个返回布尔值且不接受任何参数的委托：

```cs
Func<bool> funcE;
```

你能猜出这个相对较长的 `Func` 声明返回的类型是什么吗？

```cs
Func<bool, int, int, DateTime, string> funcF;
```

这给出了一个返回 `string` 的委托。换句话说，`Func` 中的最后一个参数类型定义了返回类型。注意 `funcF` 接受四个参数：`bool`、`int`、`int` 和 `DateTime`。

总结来说，泛型是定义类型的好方法。通过允许类型参数作为占位符，它们可以节省大量的重复代码。

## 分配委托

你在 *练习 3.01* 中介绍了创建自定义委托和简要介绍了如何分配和调用委托。然后你查看使用首选的 `Action` 和 `Func` 等效项，但你有其他方法来分配构成委托的方法（或方法）吗？还有其他调用委托的方法吗？

委托可以被分配给一个变量，其方式与分配一个类实例类似。你还可以传递新的实例或静态实例，而无需使用变量。一旦分配，你可以调用委托或将引用传递给其他类，以便它们可以调用它，这通常在框架 API 中完成。

现在将查看一个`Func`委托，它接受一个`DateTime`参数并返回一个`bool`值以指示有效性。你将使用包含两个辅助方法的`static`类，这些方法形成实际的目标：

```cs
public static class DateValidators
{
    public static bool IsWeekend(DateTime dateTime)
        => dateTime.DayOfWeek == DayOfWeek.Saturday ||
           dateTime.DayOfWeek == DayOfWeek.Sunday;
    public static bool IsFuture(DateTime dateTime) 
      => dateTime.Date > DateTime.Today;
}
```

注意

你可以在[`packt.link/mwmxh`](https://packt.link/mwmxh)找到此示例使用的代码。

注意，`DateValidators`类被标记为`static`。你可能听说过“静态是不高效的”这句话。换句话说，创建一个包含许多静态类的应用程序是一种弱实践。静态类在第一次通过运行代码访问时被实例化，并保持内存中直到应用程序关闭。这使得控制它们的生存期变得困难。如果确实是无状态的，将小型实用类定义为静态就不再是问题。无状态意味着它们不会设置任何局部变量。设置局部状态的静态类非常难以进行单元测试；你永远无法确定设置的变量来自一个测试还是另一个测试。

在前面的代码片段中，`IsFuture`如果`DateTime`参数的`Date`属性晚于当前日期，则返回`true`。你正在使用静态的`DateTime.Today`属性来检索当前系统日期。"IsWeekend"使用表达式主体语法定义，如果`DateTime`参数的星期几是星期六或星期日，则返回`true`。

你可以像分配常规变量一样分配委托（记住你分配了`futureValidator`和`weekendValidator`。每个构造函数都传递实际的目标方法，分别是`IsFuture`或`IsWeekend`实例）：

```cs
var futureValidator = new Func<DateTime, bool>(DateValidators.IsFuture);
var weekendValidator = new Func<DateTime, bool>(DateValidators.IsWeekend);
```

注意，使用`var`关键字在不包含`Func`前缀的情况下分配委托是不合法的：

```cs
var futureValidator = DateValidation.IsFuture;
```

这会导致以下编译器错误：

```cs
Cannot assign method group to an implicitly - typed variable
```

在了解了委托的知识之后，继续了解如何调用委托。

## 调用委托

调用一个委托有多种方式。例如，考虑以下定义：

```cs
var futureValidator = new Func<DateTime, bool>(DateValidators.IsFuture);
```

要调用`futureValidator`，你必须传递一个`DateTime`值，它将使用以下任何一种样式返回一个`bool`值：

+   使用空合并运算符调用：

    ```cs
    var isFuture1 = futureValidator?.Invoke(new DateTime(2000, 12, 31));
    ```

这是首选且最安全的方法；在调用`Invoke`之前，你应该始终检查是否为 null。如果有可能委托不指向内存中的对象，那么在访问方法和属性之前必须执行空引用检查。不这样做会导致抛出`NullReferenceException`。这是运行时警告你对象没有指向任何东西的方式。

通过使用空合并运算符，编译器会为你添加空值检查。在代码中，你明确声明了`futureValidator`，所以在这里它不能为空。但如果你是从另一个方法传递`futureValidator`，你如何确保调用者正确地分配了引用？

委托有额外的规则，使得它们在调用时可以抛出`NullReferenceException`。在前面的例子中，`futureValidator`有一个单一的目标，但正如你将看到的，`NullReferenceException`。

+   直接调用

这与前面的方法相同，但没有空值检查的安全性。同样，这也不推荐，因为委托可能会抛出`NullReferenceException`：

```cs
var isFuture1 = futureValidator.Invoke(new DateTime(2000, 12, 31));
```

+   没有使用`Invoke`前缀

这样看起来更简洁，因为你只需调用委托而不需要`Invoke`前缀。然而，这并不推荐，因为可能存在空引用：

```cs
var isFuture2 = futureValidator(new DateTime(2050, 1, 20));
```

通过将它们放在一起进行练习，尝试安全地分配和调用一个委托。

## 练习 3.02：分配和调用委托

在这个练习中，你将编写一个控制台应用程序，展示如何使用`Func`委托提取数值。你将创建一个具有`Distance`和`JourneyTime`属性的`Car`类。你将提示用户输入昨天和今天的行程距离，并将这些信息传递给一个`Comparison`类，该类被告知如何提取值并计算它们的差异。

执行以下步骤来完成此操作：

1.  切换到`Chapter03`文件夹，并使用 CLI `dotnet`命令创建一个新的控制台应用程序，命名为`Exercise02`：

    ```cs
    source\Chapter03>dotnet new console -o Exercise02
    ```

1.  打开`Chapter03\Exercise02.csproj`并将整个文件替换为以下设置：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
      </PropertyGroup>
    </Project>
    ```

1.  打开`Exercise02\Program.cs`并清除其内容。

1.  首先添加一个名为`Car`的记录。包含`System.Globalization`命名空间以进行字符串解析。使用`Chapter03.Exercise02`命名空间以保持代码与其他练习的分离。

1.  添加两个属性，`Distance`和`JourneyTime`。它们将只有`init`属性，所以你将使用`init`关键字：

    ```cs
    using System;
    using System.Globalization;
    namespace Chapter03.Exercise02
    {
        public record Car
        {
            public double Distance { get; init; }
            public double JourneyTime { get; init; }
        }
    ```

1.  接下来，创建一个名为`Comparison`的类，该类传递一个`Func`委托以进行操作。`Comparison`类将使用委托提取`Distance`或`JourneyTime`属性，并计算两个`Car`实例之间的差异。通过使用委托的灵活性，`Comparison`将不知道它是在提取`Distance`还是`JourneyTime`，只知道它正在使用一个双精度值来计算差异。这表明，如果你需要将来计算其他`Car`属性，你可以重用这个类：

    ```cs
        public class Comparison
        {
            private readonly Func<Car, double> _valueSelector;
            public Comparison(Func<Car, double> valueSelector)
            {
                _valueSelector = valueSelector;
            } 
    ```

1.  添加三个属性，如下所示：

    ```cs
            public double Yesterday { get; private set; }
            public double Today { get; private set; }
            public double Difference { get; private set; }
    ```

1.  现在进行计算，传递两个`Car`实例，一个用于昨天的汽车行程`yesterdayCar`，另一个用于今天的行程`todayCar`：

    ```cs
            public void Compare(Car yesterdayCar, Car todayCar)
            {
    ```

1.  要计算 `Yesterday` 的值，调用 `valueSelector` `Func` 委托，传入 `yesterdayCar` 实例。再次记住，`Comparison` 类不知道它是在提取 `Distance` 还是 `JourneyTime`；它只需要知道当 `delegate` 被一个 `Car` 参数调用时，它会返回一个双精度浮点数：

    ```cs
                Yesterday = _valueSelector(yesterdayCar);
    ```

1.  使用相同的 `Func` 委托，通过传递 `todayCar` 实例来提取 `Today` 的值，但使用相同的方式：

    ```cs
                Today = _valueSelector(todayCar);
    ```

1.  现在只是计算两个提取的数字之间的差异；你不需要使用 `Func` 委托来做这件事：

    ```cs
                Difference = Yesterday - Today;
            }
         }
    ```

1.  因此，你有一个知道如何调用 `Func` 委托来提取特定 `Car` 属性的类。现在，你需要一个类来封装 `Comparison` 实例。为此，添加一个名为 `JourneyComparer` 的类：

    ```cs
        public class JourneyComparer
        {
            public JourneyComparer()
            {
    ```

1.  对于汽车行程，你需要计算 `Yesterday` 和 `Today` 的 `Distance` 属性之间的差异。为此，创建一个 `Comparison` 类，告诉它如何从一个 `Car` 实例中提取值。你可以使用与提取汽车 `Distance` 相同的名称来命名这个 `Comparison` 类。记住，`Comparison` 构造函数需要一个 `Func` 委托，它接收一个 `Car` 实例并返回一个双精度值。你很快就会添加 `GetCarDistance()`；这最终将通过传递昨天和今天的行程 `Car` 实例来调用：

    ```cs
              Distance = new Comparison(GetCarDistance);
    ```

1.  按照前面步骤描述的过程，为 `JourneyTime` `Comparison` 重复此过程；这个应该被告知使用 `GetCarJourneyTime()`，如下所示：

    ```cs
              JourneyTime = new Comparison(GetCarJourneyTime);
    ```

1.  最后，添加另一个名为 `AverageSpeed` 的 `Comparison` 属性，如下所示。你很快就会看到 `GetCarAverageSpeed()` 是另一个函数：

    ```cs
               AverageSpeed = new Comparison(GetCarAverageSpeed);
    ```

1.  对于 `GetCarDistance` 和 `GetCarJourneyTime` 这两个局部函数，它们接收一个 `Car` 实例，并相应地返回 `Distance` 或 `JourneyTime`：

    ```cs
               static double GetCarDistance(Car car) => car.Distance; 
               static double GetCarJourneyTime(Car car) => car.JourneyTime;
    ```

1.  如其名所示，`GetCarAverageSpeed` 返回平均速度。在这里，你展示了 `Func` 委托只需要一个兼容的函数；只要返回 `double` 类型，它返回什么并不重要。`Comparison` 类不需要知道它在调用 `Func` 委托时返回的是这样的计算值：

    ```cs
              static double GetCarAverageSpeed(Car car)             => car.Distance / car.JourneyTime;
           }
    ```

1.  三个 `Comparison` 属性应该这样定义：

    ```cs
            public Comparison Distance { get; }
            public Comparison JourneyTime { get; }
            public Comparison AverageSpeed { get; }
    ```

1.  现在是主 `Compare` 方法。这个方法将传入两个 `Car` 实例，一个用于 `yesterday`，一个用于 `today`，并且它只是简单地调用三个 `Comparison` 项目上的 `Compare`，传入两个 `Car` 实例：

    ```cs
            public void Compare(Car yesterday, Car today)
            {
                Distance.Compare(yesterday, today);
                JourneyTime.Compare(yesterday, today);
                AverageSpeed.Compare(yesterday, today);
            }
        }
    ```

1.  你需要一个控制台应用程序来输入每天行驶的英里数，因此添加一个名为 `Program` 的类，并包含一个静态的 `Main` 入口点：

    ```cs
        public class Program
        {
            public static void Main()
            {
    ```

1.  你可以随机分配行程时间以节省一些输入，因此添加一个新的 `Random` 实例和 `do-while` 循环的开始，如下所示：

    ```cs
                var random = new Random();
                string input;
                do
                {
    ```

1.  按照以下方式读取昨天的距离：

    ```cs
                    Console.Write("Yesterday's distance: ");
                    input = Console.ReadLine();
                    double.TryParse(input, NumberStyles.Any,                    CultureInfo.CurrentCulture, out var distanceYesterday);
    ```

1.  你可以使用距离来创建昨天的 `Car`，并随机分配 `JourneyTime`，如下所示：

    ```cs
                    var carYesterday = new Car
                    {
                        Distance = distanceYesterday,
                        JourneyTime = random.NextDouble() * 10D
                    };
    ```

1.  对于今天的距离也做同样的处理：

    ```cs
                    Console.Write("    Today's distance: ");
                    input = Console.ReadLine();
                    double.TryParse(input, NumberStyles.Any,                    CultureInfo.CurrentCulture, out var distanceToday);
                    var carToday = new Car
                    {
                        Distance = distanceToday,
                        JourneyTime = random.NextDouble() * 10D
                    };
    ```

1.  现在您有两个填充了昨天和今天值的`Car`实例，您可以创建`JourneyComparer`实例并调用`Compare`。这将调用您的三个`Comparison`实例的`Compare`方法：

    ```cs
                    var comparer = new JourneyComparer();
                    comparer.Compare(carYesterday, carToday);
    ```

1.  现在，将结果写入控制台：

    ```cs
                    Console.WriteLine();
                    Console.WriteLine("Journey Details   Distance\tTime\tAvg Speed");
                    Console.WriteLine("-------------------------------------------------");
    ```

1.  输出昨天的成果：

    ```cs
                    Console.Write($"Yesterday         {comparer.Distance.Yesterday:N0}   \t");
                    Console.WriteLine($"{comparer.JourneyTime.Yesterday:N0}\t {comparer.AverageSpeed.Yesterday:N0}");
    ```

1.  输出今天的成果：

    ```cs
                    Console.Write($"Today             {comparer.Distance.Today:N0}     \t");                 Console.WriteLine($"{comparer.JourneyTime.Today:N0}\t {comparer.AverageSpeed.Today:N0}");
    ```

1.  最后，使用`Difference`属性写入摘要值：

    ```cs
                    Console.WriteLine("=================================================");
                    Console.Write($"Difference             {comparer.Distance.Difference:N0}     \t");                Console.WriteLine($"{comparer.JourneyTime.Difference:N0} \t{comparer.AverageSpeed.Difference:N0}");
                   Console.WriteLine("=================================================");
    ```

1.  完成以下`do-while`循环，如果用户输入空字符串则退出：

    ```cs
                } 
                while (!string.IsNullOrEmpty(input));
            }
        }
    }
    ```

运行控制台并输入距离`1000`和`900`产生以下结果：

```cs
Yesterday's distance: 1000
    Today's distance: 900
Journey Details   Distance      Time    Avg Speed
-------------------------------------------------
Yesterday         1,000         8       132
Today             900           4       242
=================================================
Difference        100           4       -109
```

程序将在循环中运行，直到您输入空白值。您会注意到输出不同，因为`JourneyTime`是使用`Random`类实例返回的随机值设置的。

注意

您可以在[`packt.link/EJTtS`](https://packt.link/EJTtS)找到用于此练习的代码。

在这个练习中，您已经看到了如何使用`Func<Car, double>`委托来创建通用代码，这些代码可以轻松重用，而无需创建额外的接口或类。

现在是时候看看删除的第二个重要方面以及它们将多个目标方法链接在一起的能力了。

## 多播委托

到目前为止，您已经调用了分配了单个方法的委托，通常是函数调用的形式。委托提供了使用`+=`运算符将一系列方法组合在一起执行的能力，可以添加任意数量的附加目标方法到目标列表中。每次调用委托时，每个目标方法都会被调用。但如果你决定要删除一个目标方法呢？这就是`-=`运算符的用武之地。

在以下代码片段中，您有一个名为`logger`的`Action<string>`委托。它以单个目标方法`LogToConsole`开始。如果您调用此委托并传入一个字符串，则将调用`LogToConsole`方法一次：

```cs
Action<string> logger = LogToConsole;
logger("1\. Calculating bill");  
```

如果您观察调用栈，您会看到这些调用：

```cs
logger("1\. Calculating bill")
--> LogToConsole("1\. Calculating bill")
```

要添加新的目标方法，您使用`+=`运算符。以下语句将`LogToFile`添加到`logger`委托的调用列表中：

```cs
logger += LogToFile;
```

现在，每次调用`logger`时，都会调用`LogToConsole`和`LogToFile`。现在再次调用`logger`：

```cs
logger("2\. Saving order"); 
```

调用栈看起来如下：

```cs
logger("2\. Saving order")
--> LogToConsole("2\. Saving order")
--> LogToFile("2\. Saving order")
```

再次假设您使用`+=`添加一个名为`LogToDataBase`的第三个目标方法，如下所示：

```cs
logger += LogToDataBase
```

现在再次调用它：

```cs
logger("3\. Closing order"); 
```

调用栈看起来如下：

```cs
logger("3\. Closing order")
--> LogToConsole("3\. Closing order")
--> LogToFile("3\. Closing order")
--> LogToDataBase("3\. Closing order")
```

然而，考虑您可能不再希望将`LogToFile`包含在目标方法列表中。在这种情况下，只需使用`-=`运算符将其删除，如下所示：

```cs
logger -= LogToFile
```

您可以再次按如下方式调用委托：

```cs
logger("4\. Closing customer"); 
```

现在，调用栈如下所示：

```cs
logger("4\. Closing customer")
--> LogToConsole("4\. Closing customer")
--> LogToDataBase("4\. Closing customer")
```

如所示，此代码只导致了`LogToConsole`和`LogToDataBase`的调用。

通过这种方式使用委托，您可以在运行时根据某些标准来决定调用哪些目标方法。这允许您将配置好的委托传递给其他方法，以便在需要时调用。

您已经看到 `Console.WriteLine` 可以用来将消息写入控制台窗口。要创建一个将日志记录到文件的方法（如前一个示例中的 `LogToFile` 所做的那样），您需要使用 `System.IO` 命名空间中的 `File` 类。`File` 有许多静态方法可以用来读取和写入文件。这里不会详细介绍 `File`，但值得提到的是 `File.AppendAllText` 方法，它可以用来创建或替换包含字符串值的文本文件，`File.Exists` 用于检查文件是否存在，以及 `File.Delete` 用于删除文件。

现在是时候通过练习来实践你所学到的知识了。

## 练习 3.03：调用多播委托

在这个练习中，您将使用多播委托创建一个当用户输入他们的 PIN 并要求查看他们的余额时记录详细信息的自动柜员机。为此，您将创建一个 `CashMachine` 类，该类调用配置好的 **记录** 委托，您可以使用它作为控制器类来决定消息是否发送到文件或控制台。

您将使用 `Action<string>` 委托，因为您不需要返回任何值。使用 `+=`，您可以在 `CashMachine` 调用您的委托时控制哪些目标方法被调用。

执行以下步骤来完成此操作：

1.  切换到 `Chapter03` 文件夹，并使用 CLI `dotnet` 命令创建一个新的控制台应用程序，名为 `Exercise03`：

    ```cs
    source\Chapter03>dotnet new console -o Exercise03
    ```

1.  打开 `Chapter03\Exercise03.csproj` 并用以下设置替换整个文件：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
      </PropertyGroup>
    </Project>
    ```

1.  打开 `Exercise03\Program.cs` 并清除其内容。

1.  添加一个名为 `CashMachine` 的新类。

1.  使用 `Chapter03.Exercise03` 命名空间：

    ```cs
    using System;
    using System.IO;
    namespace Chapter03.Exercise03
    {
        public class CashMachine
        {
            private readonly Action<string> _logger;
            public CashMachine(Action<string> logger)
            {
                _logger = logger;
            } 
    ```

`CashMachine` 构造函数传递了一个 `Action<string>` 委托，您可以将它分配给一个名为 `_logger` 的 `readonly` 类变量。

1.  添加一个 `Log` 辅助函数，在调用 `_logger` 委托之前检查它是否为 null：

    ```cs
            private void Log(string message)
                => _logger?.Invoke(message);
    ```

1.  当调用 `VerifyPin` 和 `ShowBalance` 方法时，应该记录一些详细信息。创建这些方法如下：

    ```cs
            public void VerifyPin(string pin) 
                => Log($"VerifyPin called: PIN={pin}");
            public void ShowBalance() 
                => Log("ShowBalance called: Balance=999");
        }
    ```

1.  现在，添加一个控制台应用程序，配置一个 `logger` 委托，您可以将其传递给 `CashMachine` 对象。请注意，这是一种常见的使用形式：一个负责决定其他类如何记录消息的类。使用一个常量 `OutputFile` 来指定用于文件记录的文件名，如下所示：

    ```cs
        public static class Program
        {
            private const string OutputFile = "activity.txt";
            public static void Main()
            {
    ```

1.  每次程序运行时，它应该以 `File.Delete` 开始，以删除输出文件：

    ```cs
                if (File.Exists(OutputFile))
                {
                    File.Delete(OutputFile);
                }
    ```

1.  创建一个委托实例，`logger`，它以单个目标方法 `LogToConsole` 开始：

    ```cs
                Action<string> logger = LogToConsole;
    ```

1.  使用 `+=` 操作符，将 `LogToFile` 作为第二个目标方法添加，以便在 `CashMachine` 调用委托时也会被调用：

    ```cs
                logger += LogToFile;
    ```

1.  你将很快实现两个目标日志方法；现在，创建一个`cashMachine`实例并准备调用其方法，如下所示：

    ```cs
                var cashMachine = new CashMachine(logger);
    ```

1.  提示输入`pin`并将其传递给`VerifyPin`方法：

    ```cs
                Console.Write("Enter your PIN:");
                var pin = Console.ReadLine();
                if (string.IsNullOrEmpty(pin))
                {
                    Console.WriteLine("No PIN entered");
                    return;
                }
                cashMachine.VerifyPin(pin);
                Console.WriteLine();
    ```

如果你输入一个空值，则会进行检查并显示警告。然后，程序将使用`return`语句关闭。

1.  在调用`ShowBalance`方法之前，等待`Enter`键被按下：

    ```cs
                Console.Write("Press Enter to show balance");
                Console.ReadLine();
                cashMachine.ShowBalance();
                Console.Write("Press Enter to quit");
                Console.ReadLine();
    ```

1.  现在来看一下日志方法。它们必须与你的`Action<string>`委托兼容。一个将消息写入控制台，另一个将其追加到文本文件。按照以下方式添加这两个静态方法：

    ```cs
                static void LogToConsole(string message)
                    => Console.WriteLine(message);
                static void LogToFile(string message)
                    => File.AppendAllText(OutputFile, message);
            }
         }
    }
    ```

1.  运行控制台应用程序，你会看到`VerifyPin`和`ShowBalance`调用被写入控制台：

    ```cs
    Enter your PIN:12345
    VerifyPin called: PIN=12345
    Press Enter to show balance
    ShowBalance called: Balance=999
    ```

1.  对于每个`logger`委托调用，`LogToFile`方法也会被调用，所以当你打开`activity.txt`时，你应该看到以下行：

    ```cs
    VerifyPin called: PIN=12345ShowBalance called: Balance=999
    ```

    注意

    你可以在[`packt.link/h9vic`](https://packt.link/h9vic)找到这个练习使用的代码。

重要的是要记住，委托是不可变的，所以每次你使用`+=`或`-=`运算符时，你都会创建一个新的委托实例。这意味着如果你在将委托传递给目标类之后修改了它，你将不会看到目标类内部调用该方法有任何变化。

你可以在下面的示例中看到这个操作：

```cs
MulticastDelegatesAddRemoveExample.cs
using System;
namespace Chapter03Examples
{
    class MulticastDelegatesAddRemoveExample
    {
        public static void Main()
        {
            Action<string> logger = LogToConsole;
            Console.WriteLine($"Logger1 #={logger.GetHashCode()}");
            logger += LogToConsole;
            Console.WriteLine($"Logger2 #={logger.GetHashCode()}");
            logger += LogToConsole;
            Console.WriteLine($"Logger3 #={logger.GetHashCode()}");
You can find the complete code here: https://packt.link/vqZMF.
```

C#中的所有对象都有一个`GetHashCode()`函数，它返回一个唯一的 ID。运行代码会产生以下输出：

```cs
Logger1 #=46104728
Logger2 #=1567560752
Logger3 #=236001992
```

你可以看到`+=`的调用。这表明对象引用每次都在改变。

现在看看另一个使用`Action<string>`委托的示例。在这里，你将使用`+=`运算符添加目标方法，然后使用`-=`来移除目标方法：

```cs
MulticastDelegatesExample.cs
using System;
namespace Chapter03Examples
{
    class MulticastDelegatesExample
    {
        public static void Main()
        {
            Action<string> logger = LogToConsole;
            logger += LogToConsole;
            logger("Console x 2");

            logger -= LogToConsole;
            logger("Console x 1");
            logger -= LogToConsole;
You can find the complete code here: https://packt.link/Xe0Ct.
```

你从一个目标方法`LogToConsole`开始，然后再次添加相同的目标方法。使用`logger("Console x 2")`调用日志委托会导致`LogToConsole`被调用两次。

然后使用`-=`移除`LogToConsole`**两次**，这样原本有两个目标，现在一个都没有了。运行代码会产生以下输出：

```cs
Console x 2
Console x 2
Console x 1
```

然而，由于`logger("logger is now null")`没有正确运行，你最终会得到一个未处理的异常，如下所示：

```cs
System.NullReferenceException
  HResult=0x80004003
  Message=Object reference not set to an instance of an object.
  Source=Examples
  StackTrace:
   at Chapter03Examples.MulticastDelegatesExample.Main() in Chapter03\MulticastDelegatesExample.cs:line 16
```

通过移除最后一个目标方法，`-=`运算符返回了一个空引用，然后你将其分配给了日志。正如你所看到的，在尝试调用委托之前始终检查委托是否为空是非常重要的。

### 使用 Func 委托进行多播

到目前为止，你已经在`Action`委托内部使用了`Action<string>`委托。

你已经看到当需要从调用的委托中获取返回值时使用`Func`委托。C#编译器在多播委托中使用`Func`委托也是完全合法的。

考虑以下示例，其中有一个`Func<string, string>`代理。此代理支持传递字符串并返回格式化字符串的函数。这可以在你需要通过删除`@`符号和点符号来格式化电子邮件地址时使用：

```cs
using System;
namespace Chapter03Examples
{
    class FuncExample
    {
        public static void Main()
        {
```

你首先将`RemoveDots`字符串函数分配给`emailFormatter`，并使用`Address`常量调用它：

```cs
            Func<string, string> emailFormatter = RemoveDots;
            const string Address = "admin@google.com";
            var first = emailFormatter(Address);
            Console.WriteLine($"First={first}");
```

然后你添加第二个目标，`RemoveAtSign`，并第二次调用`emailFormatter`：

```cs
            emailFormatter += RemoveAtSign;
            var second = emailFormatter(Address);
            Console.WriteLine($"Second={second}");
            Console.ReadLine();
            static string RemoveAtSign(string address)
                => address.Replace("@", "");
            static string RemoveDots(string address)
                => address.Replace(".", "");
        }
    }
} 
```

运行代码会产生以下输出：

```cs
First=admin@googlecom
Second=admingoogle.com
```

第一次调用返回`admin@googlecom`字符串。添加到目标列表中的`RemoveAtSign`返回一个只删除了`@`符号的值。

注意

你可以在[`packt.link/fshse`](https://packt.link/fshse)找到用于此示例的代码。

`Func1`和`Func2`都被调用，但只有`Func2`的值返回到`ResultA`和`ResultB`变量中，即使传递了正确的参数。当以这种方式使用多播的`Func<>`代理时，链中的所有目标`Func`实例都会被调用，但返回值将是链中最后一个`Func<>`的值。`Func<>`更适合单方法场景，尽管编译器仍然允许你将其用作多播代理而不会出现编译错误或警告。

### 当事情出错时会发生什么？

当调用代理时，调用列表中的所有方法都会被调用。对于单名称代理，这将是一个目标方法。如果其中的一个目标抛出异常，多播代理会发生什么？

考虑以下代码。当调用`logger`代理时，通过传入`try log this`，你可能期望方法按它们添加的顺序被调用：`LogToConsole`，`LogToError`，最后是`LogToDebug`：

```cs
MulticastWithErrorsExample.cs
using System;
using System.Diagnostics;
namespace Chapter03Examples
{
    class MulticastWithErrorsExample
    {
            public static void Main()
            {
                Action<string> logger = LogToConsole;
                logger += LogToError;
                logger += LogToDebug;
                try
                {
                    logger("try log this");
You can find the complete code here: https://packt.link/Ti3Nh.
```

如果任何目标方法抛出异常，例如你在`LogToError`中看到的，那么剩余的目标将**不会**被调用。

运行代码会产生以下输出：

```cs
Console: try log this
Caught oops!
All done
```

你会看到这个输出，因为`LogToDebug`方法根本没有被调用。考虑一个有多个目标监听鼠标按钮点击的 UI。第一个方法在按钮被按下时触发，并禁用按钮以防止双击，第二个方法更改按钮的图像以指示成功，第三个方法启用按钮。

如果第二个方法失败，则第三个方法将不会调用，按钮可能保持禁用状态，并分配了不正确的图像，从而使用户感到困惑。

为了确保所有目标方法都运行，你可以遍历调用列表并手动调用每个方法。看看.NET 的`MulticastDelegate`类型。你会发现有一个函数`GetInvocationList`，它返回一个包含代理对象的数组。这个数组包含已添加的目标方法：

```cs
public abstract class MulticastDelegate : Delegate {
  public sealed override Delegate[] GetInvocationList();
}
```

你现在可以遍历这些目标方法，并在`try`/`catch`块中执行每个方法。现在通过这个练习来练习你所学到的内容。

## 练习 3.04：确保在多播代理中调用所有目标方法

在本章中，你一直在使用`Action<string>`代理来执行各种日志操作。在这个练习中，你有一个日志代理的目标方法列表，并且你想要确保“所有”目标方法都被调用，即使早期的一些方法失败了。你可能有一个场景，日志记录到数据库或文件系统偶尔会失败，可能是因为网络问题。在这种情况下，你希望其他日志操作至少有机会执行它们的日志活动。

执行以下步骤来完成此操作：

1.  切换到`Chapter03`文件夹，并使用 CLI `dotnet`命令创建一个新的控制台应用程序，命名为`Exercise04`：

    ```cs
    source\Chapter03>dotnet new console -o Exercise04
    ```

1.  打开`Chapter03\Exercise04.csproj`并替换整个文件为以下设置：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
      </PropertyGroup>
    </Project>
    ```

1.  打开`Exercise04\Program.cs`并清除其内容。

1.  现在为你的控制台应用程序添加一个静态的`Program`类，包括`System`，以及额外的`System.IO`，因为你想要创建一个文件：

    ```cs
    using System;
    using System.IO;
    namespace Chapter03.Exercise04
    {
        public static class Program
        {
    ```

1.  使用一个`const`来命名日志文件。当程序执行时，此文件将被创建：

    ```cs
            private const string OutputFile = "Exercise04.txt";
    ```

1.  现在必须定义应用程序的`Main`入口点。在这里，如果你已经存在输出文件，则删除它。最好在这里从一个空文件开始，否则每次运行应用程序时，日志文件都会不断增长：

    ```cs
            public static void Main()
            {
                if (File.Exists(OutputFile))
                {
                    File.Delete(OutputFile);
                }
    ```

1.  你将开始于只有一个目标方法`logger`，即`LogToConsole`，你很快就会添加它：

    ```cs
                Action<string> logger = LogToConsole;
    ```

1.  你使用`InvokeAll`方法来调用代理，传入`"First call"`作为参数。这不会失败，因为`logger`有一个有效的方法，你很快也会添加`InvokeAll`：

    ```cs
                InvokeAll(logger, "First call"); 
    ```

1.  本练习的目的是拥有一个多播代理，因此添加一些额外的目标方法：

    ```cs
                logger += LogToConsole;
                logger += LogToDatabase;
                logger += LogToFile; 
    ```

1.  使用以下方式尝试第二次调用`InvokeAll`：

    ```cs
                InvokeAll(logger, "Second call"); 
                Console.ReadLine();
    ```

1.  现在对于添加到代理中的目标方法。为此添加以下代码：

    ```cs
                static void LogToConsole(string message)
                    => Console.WriteLine($"LogToConsole: {message}");
                static void LogToDatabase(string message)
                    => throw new ApplicationException("bad thing happened!");
                static void LogToFile(string message)
                    => File.AppendAllText(OutputFile, message);

    ```

1.  你现在可以实现`InvokeAll`方法：

    ```cs
                static void InvokeAll(Action<string> logger, string arg)
                {
                    if (logger == null)
                         return;
    ```

它传递了一个与`logger`代理类型匹配的`Action<string>`代理，以及一个用于调用每个目标方法的`arg`字符串。在此之前，检查`logger`是否已经为 null，以及你无法对 null 代理做任何事情是很重要的。

1.  使用代理的`GetInvocationList()`方法来获取所有目标方法的列表：

    ```cs
                    var delegateList = logger.GetInvocationList();
                    Console.WriteLine($"Found {delegateList.Length} items in {logger}"); 
    ```

1.  现在，按照以下方式遍历列表中的每个项目：

    ```cs
                    foreach (var del in delegateList)
                    {
    ```

1.  在将每个循环元素包裹在`try`/`catch`块中之后，将`del`强制转换为`Action<string>`：

    ```cs
                       try
                       {
                         var action = del as Action<string>; 
    ```

`GetInvocationList`返回每个项目作为基本代理类型，而不考虑它们的实际类型。

1.  如果它是正确的类型并且**不是**null，那么尝试调用它是安全的：

    ```cs
                          if (del is Action<string> action)
                          {
                              Console.WriteLine($"Invoking '{action.Method.Name}' with '{arg}'");
                              action(arg);
                          }
                          else
                          {
                              Console.WriteLine("Skipped null");
                          } 
    ```

你添加了一些额外的细节来显示将要使用代理的`Method.Name`属性调用的内容。

1.  最后，添加一个`catch`块来记录捕获到的错误消息： 

    ```cs
                      }
                      catch (Exception e)
                      {
                          Console.WriteLine($"Error: {e.Message}");
                      }
                    }
                }
            }
        }
    }
    ```

1.  运行代码，会创建一个名为`Exercise04.txt`的文件，其中包含以下结果：

    ```cs
    Found 1 items in System.Action`1[System.String]
    Invoking '<Main>g__LogToConsole|1_0' with 'First call'
    LogToConsole: First call
    Found 4 items in System.Action`1[System.String]
    Invoking '<Main>g__LogToConsole|1_0' with 'Second call'
    LogToConsole: Second call
    Invoking '<Main>g__LogToConsole|1_0' with 'Second call'
    LogToConsole: Second call
    Invoking '<Main>g__LogToDatabase|1_1' with 'Second call'
    Error: bad thing happened!
    Invoking '<Main>g__LogToFile|1_2' with 'Second call'
    ```

你会看到它捕获了`LogToDatabase`抛出的错误，同时仍然允许调用`LogToFile`。

注意

你可以在[`packt.link/Dp5H4`](https://packt.link/Dp5H4)找到用于此练习的代码。

现在重要的是要使用事件来扩展多播概念。

## 事件

在前面的章节中，你已经创建了委托，并在同一方法中直接调用它们，或者将它们传递给其他方法，以便在需要时调用。通过这种方式使用委托，你有一个简单的方法来让代码在发生感兴趣的事情时得到通知。到目前为止，这还没有成为一个大问题，但你可能已经注意到似乎没有方法可以防止一个可以访问委托的对象直接调用它。

考虑以下场景：你创建了一个应用程序，允许其他程序通过将它们的目标方法添加到你提供的委托中，来注册当收到新电子邮件时的通知。如果一个程序，无论是由于错误还是恶意原因，决定自己调用你的委托，会发生什么？这可能会轻易地压倒你的调用列表中的所有目标方法。这样的监听程序绝对不应该以这种方式调用委托——毕竟，它们应该是被动的监听者。

你可以添加额外的允许监听者将它们的目标方法添加或从调用列表中删除的方法，并保护委托免受直接访问，但如果你在一个应用程序中有数百个这样的委托，这将需要编写大量的代码。

`event`关键字指示 C#编译器添加额外的代码以确保委托只能由声明它的类或结构体调用。外部代码可以添加或删除目标方法，但被阻止调用委托。尝试这样做会导致编译器错误。

这种模式通常被称为发布-订阅模式。引发事件的对象被称为事件发送者或**发布者**；接收事件的对象被称为事件处理程序或**订阅者**。

## 定义一个事件

`event`关键字用于定义事件及其关联的委托。其定义看起来与委托的定义方式相似，但与委托不同，你不能使用全局命名空间来定义事件：

```cs
public event EventHandler MouseDoubleClicked
```

事件有四个元素：

+   范围：一个访问修饰符，如`public`、`private`或`protected`，用于定义作用域。

+   `event`关键字。

+   委托类型：关联的委托，例如本例中的`EventHandler`。

+   事件名称：可以是任何你喜欢的名称，例如`MouseDoubleClicked`。然而，名称必须在命名空间内是唯一的。

事件通常与内置的.NET 委托`EventHandler`或其泛型版本`EventHandler<>`相关联。为事件创建自定义委托很少见，但你可能会在`Action`和泛型`Action<T>`委托之前创建的旧版代码中找到它。

`EventHandler`委托在.NET 的早期版本中可用。它具有以下签名，接受一个发送者`object`和一个`EventArgs`参数：

```cs
public delegate void EventHandler(object sender, EventArgs e); 
```

更近期的基于泛型的`EventHandler<T>`委托看起来相似；它也接受一个发送者`object`和一个由类型`T`定义的参数：

```cs
public delegate void EventHandler<T>(object sender, T e); 
```

`sender`参数被定义为`object`，允许发送任何类型的对象到订阅者，以便他们可以识别事件的发送者。这在需要集中处理各种类型对象而不是特定实例的情况下非常有用。

例如，在一个 UI 应用程序中，你可能有一个订阅者监听 OK 按钮被点击，另一个订阅者监听**取消**按钮被点击——每个这些都可以由两个不同的方法处理。在多个复选框用于切换选项开或关的情况下，你可以使用一个单一的目标方法，只需要告诉它复选框是发送者，并相应地切换设置。这允许你重用相同的复选框处理程序，而不是为屏幕上的每个复选框创建一个方法。

在调用`EventHandler`委托时，包含发送者的详细信息不是强制的。通常，你可能不希望向外界透露你代码的内部工作原理；在这种情况下，将 null 引用传递给委托是一种常见的做法。

两个委托中的第二个参数可以用来提供关于事件的额外上下文信息（例如，是左键还是右键被按下？）。传统上，这些额外信息是通过从`EventArgs`派生的类包装的，但在较新的.NET 版本中，这种约定已经放宽。

你应该为你的事件定义使用两个标准的.NET 委托吗？

+   `EventHandler`: 当没有额外信息描述事件时可以使用。例如，复选框点击事件可能不需要任何额外信息，它只是被点击了。在这种情况下，传递 null 或`EventArgs.Empty`作为第二个参数是完全有效的。这个委托通常可以在使用从`EventArgs`派生的类进一步描述事件的旧版应用程序中找到。是鼠标的双击触发了这个事件吗？在这种情况下，可能已经向从`EventArgs`派生的类中添加了一个`Clicks`属性来提供这样的额外细节。

+   `EventHandler<T>`: 由于 C#中泛型的引入，这已成为更频繁使用的委托事件，仅仅因为使用泛型需要创建更少的类。

有趣的是，无论你给事件赋予什么范围（例如 `public`），C# 编译器都会在内部创建一个具有该名称的私有成员。这是事件的关键概念：只有定义事件的类才能**调用**它。消费者可以自由添加或删除他们的兴趣，但不能**自己**调用它。

当定义一个事件时，定义该事件的发布者类可以简单地按需调用它，就像调用委托一样。在早期示例中，强调了在调用之前始终检查委托是否为空。对于事件，也应采取相同的方法，因为你对订阅者如何或何时添加或删除目标方法几乎没有控制权。

当发布者类最初创建时，所有事件都有一个初始值为空。当任何订阅者添加目标方法时，这会变为非空。相反，一旦订阅者删除目标方法，如果调用列表中没有其他方法，事件将恢复为空。这是你之前在委托中看到的标准行为。

你可以通过在事件定义的末尾添加一个空委托来防止事件永远为空：

```cs
public event EventHandler<MouseEventArgs> MouseDoubleClicked = delegate {};
```

而不是使用默认的空值，你添加了自己的默认委托实例——一个什么也不做的实例。因此，在 `{}` 符号之间有一个空格。

在使用发布者类中的事件时，通常会遵循一个常见的模式，尤其是在可能进一步派生的类中。现在，通过一个简单的示例，你将看到这一点：

1.  定义一个名为 `MouseClickedEventArgs` 的类，该类包含关于事件的附加信息，在本例中，是检测到的鼠标点击次数：

    ```cs
    using System;
    namespace Chapter03Examples
    {
        public class MouseClickedEventArgs 
        {
            public MouseClickedEventArgs(int clicks)
            {
                Clicks = clicks;
            }
            public int Clicks { get; }
        }
    ```

观察一下 `MouseClickPublisher` 类，它使用泛型 `EventHandler<>` 委托定义了一个 `MouseClicked` 事件。

1.  现在添加 `delegate { };` 块以防止 `MouseClicked` 初始为空：

    ```cs
        public class MouseClickPublisher
        {
         public event EventHandler<MouseClickedEventArgs> MouseClicked = delegate { };
    ```

1.  添加一个 `OnMouseClicked` 虚拟方法，以便任何进一步派生的 `MouseClickPublisher` 类有机会抑制或更改事件通知，如下所示：

    ```cs
            protected virtual void OnMouseClicked( MouseClickedEventArgs e)
            {
                var evt = MouseClicked;
                evt?.Invoke(this, e);
            }
    ```

1.  现在你需要一个跟踪鼠标点击的方法。在这个例子中，你实际上不会展示如何检测鼠标点击，但你将调用 `OnMouseClicked`，传递 `2` 来指示双击。

1.  注意，你没有直接调用 `MouseClicked` 事件；你总是通过 `OnMouseClicked` 中间方法来调用。这为其他 `MouseClickPublisher` 的实现提供了覆盖事件通知的方式：

    ```cs
            private void TrackMouseClicks()
            {
                OnMouseClicked(new MouseClickedEventArgs(2));
            }
        } 
    ```

1.  现在添加一个基于 `MouseClickPublisher` 的新类型的发布者：

    ```cs
        public class MouseSingleClickPublisher : MouseClickPublisher
        {
            protected override void OnMouseClicked(MouseClickedEventArgs e)
            {
                if (e.Clicks == 1)
                {
                    OnMouseClicked(e);
                }
            }
        }
    } 
    ```

`MouseSingleClickPublisher` 覆盖了 `OnMouseClicked` 方法，并且只有在检测到单次点击时才调用基类的 `OnMouseClicked`。通过实现这种模式，你允许不同类型的发布者以定制的方式控制是否向订阅者触发事件。

注意

你可以在 [`packt.link/J1EiB`](https://packt.link/J1EiB) 找到用于此示例的代码。

你现在可以通过以下练习来练习你所学的知识。

## 练习 3.05：发布和订阅事件

在这个练习中，你将创建一个闹钟作为发布者的示例。闹钟将模拟 `Ticked` 事件。你还将添加一个 `WakeUp` 事件，当当前时间与闹钟时间匹配时发布。在 .NET 中，`DateTime` 用于表示时间点，因此你将使用它来表示当前时间和闹钟时间属性。你将使用 `DateTime.Subtract` 来获取当前时间和闹钟时间之间的差异，并在到期时发布 `WakeUp` 事件。

执行以下步骤来完成此操作：

1.  切换到 `Chapter03` 文件夹，并使用 CLI `dotnet` 命令创建一个新的控制台应用程序，命名为 `Exercise05`：

    ```cs
    dotnet new console -o Exercise05
    ```

1.  打开 `Chapter03\Exercise05.csproj` 并将整个文件替换为以下设置：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
      </PropertyGroup>
    </Project>
    ```

1.  打开 `Exercise05\Program.cs` 并清空其内容。

1.  添加一个名为 `AlarmClock` 的新类。在这里你需要使用 `DateTime` 类，因此包含 `System` 命名空间：

    ```cs
    using System;
    namespace Chapter03.Exercise05
    {
        public class AlarmClock
        {
    ```

你将为订阅者提供两个事件来监听——`WakeUp`，基于非泛型的 `EventHandler` 委托（因为你不会在此事件中传递任何额外信息），以及 `Ticked`，它使用泛型的 `EventHandler` 委托，并带有 `DateTime` 参数类型。

1.  你将使用这个方法来传递当前时间以在控制台显示。注意，两者都包含初始的 `delegate {};` 安全机制：

    ```cs
            public event EventHandler WakeUp = delegate {};
            public event EventHandler<DateTime> Ticked = delegate {};
    ```

1.  包含一个 `OnWakeUp` 重写作为示例，但不要对 `Ticked` 做同样的事情；这是为了展示不同的调用方法：

    ```cs
            protected void OnWakeUp()
            {
                WakeUp.Invoke(this, EventArgs.Empty);
            }
    ```

1.  现在添加两个 `DateTime` 属性，闹钟时间和时钟时间，如下所示：

    ```cs
            public DateTime AlarmTime { get; set; }
            public DateTime ClockTime { get; set; }
    ```

1.  使用 `Start` 方法来启动时钟。你通过一个简单的循环模拟时钟每分钟响一次，持续 `24 小时`，如下所示：

    ```cs
            public void Start()
            {
                // Run for 24 hours
                const int MinutesInADay = 60 * 24;
    ```

1.  对于每个模拟的分钟，使用 `DateTime.AddMinute` 增加时钟，并发布 `Ticked` 事件，传入 `this`（`AlarmClock` 发送实例）和时钟时间：

    ```cs
                for (var i = 0; i < MinutesInADay; i++)
                {
                    ClockTime = ClockTime.AddMinutes(1);
                    Ticked.Invoke(this, ClockTime);
    ```

使用 `ClockTime.Subtract` 来计算点击时间和闹钟时间之间的差异。

1.  你将 `timeRemaining` 值传递给本地函数 `IsTimeToWakeUp`，调用 `OnWakeUp` 方法，并在需要醒来时跳出循环：

    ```cs
                  var timeRemaining = ClockTime                 .Subtract(AlarmTime)                .TotalMinutes;
                   if (IsTimeToWakeUp(timeRemaining))
                    {
                        OnWakeUp();
                        break;
                    }
                }
    ```

1.  使用 `IsTimeToWakeUp`，一个关系模式，来查看是否剩余时间不足一分钟。为此添加以下代码：

    ```cs
                static bool IsTimeToWakeUp(double timeRemaining) 
                    => timeRemaining is (>= -1.0 and <= 1.0);
            }
        }   
    ```

1.  现在添加一个控制台应用程序，它从静态 `void Main` 入口点开始订阅闹钟及其两个事件：

    ```cs
             public static class Program
        {
            public static void Main()
            {
    ```

1.  创建 `AlarmClock` 实例，并使用 `+=` 运算符订阅 `Ticked` 事件和 `WakeUp` 事件。你将很快定义 `ClockTicked` 和 `ClockWakeUp`。现在，只需添加以下代码：

    ```cs
                var clock = new AlarmClock();
                clock.Ticked += ClockTicked;
                clock.WakeUp += ClockWakeUp; 
    ```

1.  设置时钟的当前时间，使用 `DateTime.AddMinutes` 向闹钟时间添加 `120` 分钟，然后启动时钟，如下所示：

    ```cs
                clock.ClockTime = DateTime.Now;
                clock.AlarmTime = DateTime.Now.AddMinutes(120);
                Console.WriteLine($"ClockTime={clock.ClockTime:t}");
                Console.WriteLine($"AlarmTime={clock.AlarmTime:t}");
                clock.Start(); 
    ```

1.  通过提示按 `Enter` 键来完成 `Main` 方法：

    ```cs
                Console.WriteLine("Press ENTER");
                Console.ReadLine();

    ```

1.  现在你可以添加事件订阅者的局部方法：

    ```cs
                static void ClockWakeUp(object sender, EventArgs e)
                {
                   Console.WriteLine();
                   Console.WriteLine("Wake up");
                }
    ```

`ClockWakeUp`传递发送者和`EventArgs`参数。你不需要这些参数，但它们对于`EventHandler`委托是必需的。当这个订阅者的方法被调用时，你将`"Wake up"`写入控制台。

1.  `ClockTicked`传递了`DateTime`参数，这是`EventHandler<DateTime>`委托所要求的。在这里，你传递当前时间，因此使用`:t`将时间以短格式写入控制台：

    ```cs
                 static void ClockTicked(object sender, DateTime e)
                    => Console.Write($"{e:t}...");
            }
        }
    } 
    ```

1.  运行应用程序会产生以下输出：

    ```cs
    ClockTime=14:59
    AlarmTime=16:59
    15:00...15:01...15:02...15:03...15:04...15:05...15:06...15:07...15:08...15:09...15:10...15:11...15:12...15:13...15:14...15:15...15:16...15:17...15:18...15:19...15:20...15:21...15:22...15:23...15:24...15:25...15:26...15:27...15:28...15:29...15:30...15:31...15:32...15:33...15:34...15:35...15:36...15:37...15:38...15:39...15:40...15:41...15:42...15:43...15:44...15:45...15:46...15:47...15:48...15:49...15:50...15:51...15:52...15:53...15:54...15:55...15:56...15:57...15:58...15:59...16:00...16:01...16:02...16:03...16:04...16:05...16:06...16:07...16:08...16:09...16:10...16:11...16:12...16:13...16:14...16:15...16:16...16:17...16:18...16:19...16:20...16:21...16:22...16:23...16:24...16:25...16:26...16:27...16:28...16:29...16:30...16:31...16:32...16:33...16:34...16:35...16:36...16:37...16:38...16:39...16:40...16:41...16:42...16:43...16:44...16:45...16:46...16:47...16:48...16:49...16:50...16:51...16:52...16:53...16:54...16:55...16:56...16:57...16:58...16:59...
    Wake up
    Press ENTER
    ```

在这个例子中，你可以看到闹钟每分钟模拟一次滴答声并发布一个`Ticked`事件。

注意

你可以在[`packt.link/GPkYQ`](https://packt.link/GPkYQ)找到用于此练习的代码。

现在是时候掌握事件和委托之间的区别了。

# 事件或委托？

表面上看，事件和委托看起来非常相似：

+   事件是委托的扩展形式。

+   两者都提供**后期绑定**的语义，因此，而不是在编译时精确知道的方法调用，你可以在运行时延迟一个目标方法列表。

+   两者都是`Invoke()`或更简单的`()`后缀快捷方式，理想情况下在使用之前进行空值检查。

关键考虑因素如下：

+   可选性：事件提供了一种可选的方法；调用者可以决定是否加入事件。如果你的组件可以在不需要任何订阅者方法的情况下完成任务，那么使用基于事件的方法更可取。

+   返回类型：你需要处理返回类型吗？与事件相关联的委托总是无返回值的。

+   生命周期：事件订阅者通常比发布者有更短的生存期，即使没有活跃的订阅者，发布者也会继续检测新消息。

## 静态事件可能导致内存泄漏

在你结束对事件的审视之前，使用事件时要格外小心，尤其是那些静态定义的事件。

每当你将订阅者的目标方法添加到发布者的事件中时，发布者类将存储对你的目标方法的引用。当你完成对订阅者实例的使用，并且它仍然附加到一个`static`发布者上时，你的订阅者使用的内存可能不会被清理。

这些通常被称为孤儿、幽灵或鬼魂事件。为了防止这种情况，始终尝试将每个`+=`调用与相应的`-=`操作符配对。

注意

反应式扩展（Rx）([`github.com/dotnet/reactive`](https://github.com/dotnet/reactive)) 是一个用于利用和驯服基于事件和异步编程的 LINQ 样式操作符的出色库。Rx 提供了一种时间转换的方法，例如，只需几行代码就可以将非常嘈杂的事件缓冲到可管理的流中。更重要的是，Rx 流非常容易进行单元测试，让你能够有效地控制时间。

现在来了解一下有趣的 lambda 表达式主题。

# Lambda 表达式

在前面的章节中，您主要使用类级方法作为委托和事件的目标，例如在*练习 3.05*中也使用的`ClockTicked`和`ClockWakeUp`方法：

```cs
var clock = new AlarmClock();
clock.Ticked += ClockTicked;
clock.WakeUp += ClockWakeUp;
static void ClockTicked(object sender, DateTime e)
  => Console.Write($"{e:t}...");

static void ClockWakeUp(object sender, EventArgs e)
{
    Console.WriteLine();
    Console.WriteLine("Wake up");
}
```

`ClockWakeUp`和`ClockTicked`方法易于理解和逐步执行。然而，通过将它们转换为 lambda 表达式语法，您可以拥有更简洁的语法，并且与它们在代码中的位置更接近。

现在将`Ticked`和`WakeUp`事件转换为使用两个不同的 lambda 表达式：

```cs
clock.Ticked += (sender, e) =>
{
    Console.Write($"{e:t}..."); 
};  
clock.WakeUp += (sender, e) =>
{
    Console.WriteLine();
    Console.WriteLine("Wake up");
}; 
```

您使用了相同的`+=`运算符，但您看到的是`(sender, e) =>`而不是方法名，以及与`ClockTicked`和`ClockWakeUp`中相同的代码块。

定义 lambda 表达式时，您可以在括号`()`内传递任何参数，然后是`=>`（这通常读作**goes to**），然后是您的表达式/语句块：

```cs
(parameters) => expression-or-block
```

代码块可以像您需要的那么复杂，如果它是一个基于`Func`的委托，则可以返回一个值。

编译器通常可以推断每个参数类型，因此您甚至不需要指定它们的类型。此外，如果只有一个参数并且编译器可以推断其类型，则可以省略括号。

无论何时需要将委托（记住`Action`、`Action<T>`和`Func<T>`是委托的内置示例）用作参数，而不是创建一个类或局部方法或函数，您都应该考虑使用 lambda 表达式。主要原因是这样通常会产生更少的代码，并且代码放置在它被使用的位置附近。

现在考虑另一个关于 Lambda 的例子。给定一个电影列表，您可以使用`List<string>`类来存储这些基于字符串的名称，如下面的代码片段所示：

```cs
using System;
using System.Collections.Generic;
namespace Chapter03Examples
{
    class LambdaExample
    {
        public static void Main()
        {
            var names = new List<string>
            {
                "The A-Team",
                "Blade Runner",
                "There's Something About Mary",
                "Batman Begins",
                "The Crow"
            };
```

您可以使用`List.Sort`方法按字母顺序对名称进行排序（最终输出将在本例的末尾显示）：

```cs
            names.Sort();
            Console.WriteLine("Sorted names:");
            foreach (var name in names)
            {
                Console.WriteLine(name);
            }
            Console.WriteLine();
```

如果您需要更多控制排序的方式，`List`类还有一个接受此形式委托的`Sort`方法：`delegate int Comparison<T>(T x, T y)`。此委托传递两个相同类型的参数（`x`和`y`）并返回一个`int`值。您可以使用`int`值来定义列表中项的排序顺序，而无需担心`Sort`方法的内部工作原理。

作为替代方案，您可以对名称进行排序，以排除电影标题开头的`"The"`。这通常用作列出名称的替代方法。您可以通过传递一个 lambda 表达式并使用`( )`语法来包装两个字符串`x, y`，当`Sort()`调用您的 lambda 时，这两个字符串将被传递，来实现这一点。

如果`x`或`y`以您的噪声词`"The"`开头，那么您将使用`string.Substring`函数跳过前四个字符。然后使用`String.Compare`返回一个数值，该数值比较结果字符串值，如下所示：

```cs
            const string Noise = "The ";
            names.Sort( (x, y) =>
            {
                if (x.StartsWith(Noise))
                {
                    x = x.Substring(Noise.Length);
                }
                if (y.StartsWith(Noise))
                {
                    y = x.Substring(Noise.Length);
                }
                return string.Compare(x , y);
            });
```

然后，您可以将排序后的结果输出到控制台：

```cs
            Console.WriteLine($"Sorted excluding leading '{Noise}':");
            foreach (var name in names)
            {
                Console.WriteLine(name);
            }
            Console.ReadLine();
         }
     }
} 
```

运行示例代码会产生以下输出：

```cs
Sorted names:
Batman Begins
Blade Runner
The A-Team
The Crow
There's Something About Mary
Sorted excluding leading 'The ':
The A-Team
Batman Begins
Blade Runner
The Crow
There's Something About Mary 
```

你可以看到，第二组名称按`"The"`被忽略的顺序排序。

注意

你可以在[`packt.link/B3NmQ`](http://packt.link/B3NmQ)找到用于此示例的代码。

为了看到这些 lambda 表达式在实际中的应用，尝试以下练习。

## 练习 3.06：使用语句 Lambda 反转句子中的单词

在这个练习中，你将创建一个实用类，该类可以将句子中的单词拆分并返回单词顺序颠倒的句子。

执行以下步骤来完成此操作：

1.  切换到`Chapter03`文件夹，并使用 CLI `dotnet`命令创建一个新的控制台应用程序，命名为`Exercise06`：

    ```cs
    source\Chapter03>dotnet new console -o Exercise06
    ```

1.  打开`Chapter03\Exercise06.csproj`，并将整个文件替换为以下设置：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
      </PropertyGroup>
    </Project>
    ```

1.  打开`Exercise02\Program.cs`并清除其内容。

1.  添加一个名为`WordUtilities`的新类，其中包含一个名为`ReverseWords`的字符串函数。你需要包含`System.Linq`命名空间以帮助进行字符串操作：

    ```cs
    using System;
    using System.Linq;
    namespace Chapter03.Exercise06
    {
        public static class WordUtilities
        {
            public static string ReverseWords(string sentence)
            {
    ```

1.  定义一个名为`swapWords`的`Func<string, string>`委托，它接受一个字符串输入并返回一个字符串值：

    ```cs
              Func<string, string> swapWords = 
    ```

1.  你将接受一个名为`phrase`的字符串输入参数：

    ```cs
                phrase =>
    ```

1.  现在来看 lambda 表达式体。使用`string.Split`函数将`phrase`字符串拆分为一个字符串数组，以空格作为分隔符：

    ```cs
                      {
                        const char Delimit = ' ';
                        var words = phrase
                            .Split(Delimit)
                            .Reverse();
                        return string.Join(Delimit, words);
                    };
    ```

`String.Reverse`在将反转的单词字符串数组使用`string.Join`连接成一个字符串之前，先反转数组中字符串的顺序。

1.  你已经定义了所需的`Func`，因此通过传递句子参数并返回结果来调用它：

    ```cs
                return swapWords(sentence);
             }
        }
    ```

1.  现在为一个控制台应用程序，它提示输入一个句子，该句子被传递给`WordUtilities.ReverseWords`，并将结果写入控制台：

    ```cs
        public static class Program
        {
            public static void Main()
            {
                do
                {
                    Console.Write("Enter a sentence:");
                    var input = Console.ReadLine();
                    if (string.IsNullOrEmpty(input))
                    {
                        break;
                    }
                    var result = WordUtilities.ReverseWords(input);
                    Console.WriteLine($"Reversed: {result}")
    ```

运行控制台应用程序会产生类似以下的结果输出：

```cs
Enter a sentence:welcome to c#
Reversed: c# to welcome
Enter a sentence:visual studio by microsoft
Reversed: microsoft by studio visual
```

注意

你可以在[`packt.link/z12sR`](https://packt.link/z12sR)找到用于此练习的代码。

你将通过查看一些不太明显的问题来结束对 lambda 表达式的探讨，这些问题你可能不会在运行和调试时预期看到。

## 捕获和闭包

Lambda 表达式可以**捕获**定义方法中任何变量或参数。捕获这个词用来描述 lambda 表达式如何捕获或向上进入父方法以访问任何变量或参数。

为了更好地理解这一点，考虑以下示例。在这里，你将创建一个名为`joiner`的`Func<int, string>`，它使用`Enumerable.Repeat`方法将单词连接起来。`word`变量（称为外部变量）被捕获在`joiner`表达式的主体中：

```cs
var word = "hello";
Func<int, string> joiner = 
    input =>
    {
        return string.Join(",", Enumerable.Repeat(word, input));
    };  
Console.WriteLine($"Outer Variables: {joiner(2)}"); 
```

运行前面的示例会产生以下输出：

```cs
Outer Variables: hello,hello
```

你通过传递`2`作为参数调用了`joiner`委托。在那个时刻，外部`word`变量的值为`"hello"`，它被重复两次。

这证实了从父方法捕获的变量在 `Func` 被调用时被评估。现在将 `word` 的值从 `hello` 更改为 `goodbye`，并再次调用 `joiner`，传递 `3` 作为参数：

```cs
word = "goodbye";
Console.WriteLine($"Outer Variables Part2: {joiner(3)}");
```

运行此示例会产生以下输出：

```cs
Outer Variables Part2: goodbye,goodbye,goodbye
```

值得记住的是，你定义 `joiner` 的位置在代码中并不重要。你可以在声明 `joiner` 之前或之后将 `word` 的值更改为任意数量的字符串。

将捕获进一步扩展，如果你在 lambda 中定义了一个与相同名称的变量，它将具有 `word` 的作用域，这将对该名称的外部变量没有任何影响：

```cs
Func<int, string> joinerLocal =
    input =>
    {
        var word = "local";
        return string.Join(",", Enumerable.Repeat(word, input));
    };
Console.WriteLine($"JoinerLocal: {joinerLocal(2)}");
Console.WriteLine($"JoinerLocal: word={word}");   
```

上述示例的结果如下。注意外部变量 `word` 从 `goodbye` 保持不变：

```cs
JoinerLocal: local,local
JoinerLocal: word=goodbye
```

最后，你将了解 C# 语言的一个微妙概念——闭包，它经常导致意外的结果。

在以下示例中，你有一个变量 `actions`，它包含一个 `Action` 委托的 `List`。你使用基本的 `for` 循环向列表中添加五个单独的 `Action` 实例。每个 `Action` 的 lambda 表达式只是将 `for` 循环中的 `i` 的值写入控制台。最后，代码只是遍历 `actions` 列表中的每个 `Action` 并调用它们：

```cs
var actions = new List<Action>();
for (var i = 0; i < 5; i++)
{
    actions.Add( () => Console.WriteLine($"MyAction: i={i}")) ;
}
foreach (var action in actions)
{
    action();
}
```

运行示例会产生以下输出：

```cs
MyAction: i=5
MyAction: i=5
MyAction: i=5
MyAction: i=5
MyAction: i=5
```

`MyAction: i` 没有从 `0` 开始的原因是，当从 `Action` 委托内部访问 `i` 的值时，它只会在 `Action` 被调用后评估。到每个委托被调用时，外部循环已经重复了五次。

注意

你可以在 [`packt.link/vfOPx`](https://packt.link/vfOPx) 找到用于此示例的代码。

这与你在捕获概念中观察到的类似，其中外部变量（在这种情况下为 `i`），只有在被调用时才会被评估。你在 `for` 循环中使用 `i` 将每个 `Action` 添加到列表中，但在调用每个动作时，`i` 已经有了它的最终值 `5`。

这通常会导致意外的行为，特别是如果你假设每个动作的循环变量中使用了 `i`。为了确保在 lambda 表达式内部使用 `i` 的递增值，你需要引入一个 `for` 循环，它包含迭代变量的副本。

在以下代码片段中，你添加了 `closurei` 变量。它看起来非常微妙，但现在你有一个更局部的作用域变量，你从 lambda 表达式内部而不是迭代器 `i` 访问它：

```cs
var actionsSafe = new List<Action>();
for (var i = 0; i < 5; i++)
{
    var closurei = i;
    actionsSafe.Add(() => Console.WriteLine($"MyAction: closurei={closurei}"));
}
foreach (var action in actionsSafe)
{
    action();
}
```

运行示例会产生以下输出。你可以看到，当每个 `Action` 被调用时，使用的是递增的值，而不是你之前看到的 `5` 的值：

```cs
MyAction: closurei=0
MyAction: closurei=1
MyAction: closurei=2
MyAction: closurei=3
MyAction: closurei=4
```

你已经涵盖了事件驱动应用程序中委托和事件的关键方面。你通过使用 lambda 提供的简洁编码风格扩展了这一点，以便在感兴趣的事件发生时得到通知。

现在，您将把这些想法整合到一个活动中，在这个活动中，您将使用一些内置的.NET 类及其自己的事件。您需要将这些事件适配到您自己的格式，并发布它们，以便控制台应用程序可以订阅。

现在，是时候通过以下活动来练习您所学到的一切。

## 活动 3.01：创建网页文件下载器

您计划调查美国风暴事件的模式。为此，您需要从在线来源下载风暴事件数据集以供后续分析。国家海洋和大气管理局是此类数据的一个来源，可以通过[`www1.ncdc.noaa.gov/pub/data/swdi/stormevents/csvfiles`](https://www1.ncdc.noaa.gov/pub/data/swdi/stormevents/csvfiles)访问。

您的任务是创建一个.NET Core 控制台应用程序，允许输入一个网址，并将该网址的内容下载到本地磁盘。为了尽可能友好，应用程序需要使用表示输入无效地址、下载进度和完成时的事件。

理想情况下，您应该尝试隐藏您用于下载文件的内部实现，更愿意将您使用的任何事件适配为调用者可以订阅的事件。这种适配形式通常用于通过隐藏内部细节来使代码更易于维护。

为了这个目的，C#中的`WebClient`类可以用于下载请求。与.NET 的许多部分一样，这个类返回实现`IDisposable`接口的对象。这是一个标准接口，它表示您正在使用的对象应该被`using`语句包裹，以确保在您完成使用对象后，任何资源或内存都会为您清理。`using`的格式如下：

```cs
using (IDisposable) { statement_block }
```

最后，`WebClient.DownloadFileAsync`方法在后台下载文件。理想情况下，您应该使用一种机制，允许代码的一部分使用`System.Threading.ManualResetEventSlim`类，该类具有`Set`和`Wait`方法，可以帮助进行此类信号。

对于这个活动，您需要执行以下步骤：

1.  添加一个进度更改的`EventArgs`类（一个示例名称可以是`DownloadProgressChangedEventArgs`），当发布进度事件时可以使用。这个类应该有`ProgressPercentage`和`BytesReceived`属性。

1.  应该使用`System.Net`中的`WebClient`类来下载请求的网页文件。您应该创建一个适配器类（建议的名称为`WebClientAdapter`），以隐藏您对`WebClient`的内部使用情况。

1.  您的适配器类应该提供三个事件——`DownloadCompleted`、`DownloadProgressChanged`和`InvalidUrlRequested`——调用者可以订阅这些事件。

1.  适配器类需要一个`DownloadFile`方法，该方法调用`WebClient`类的`DownloadFileAsync`方法来启动下载请求。这需要将基于字符串的网页地址转换为统一资源标识符（URI）类。`Uri.TryCreate()`方法可以从控制台输入的字符串创建一个绝对地址。如果`Uri.TryCreate`的调用失败，您应该发布`InvalidUrlRequested`事件来指示这个失败。

1.  `WebClient`有两个事件——`DownloadFileCompleted`和`DownloadProgressChanged`。您应该订阅这两个事件，并使用您自己的类似事件重新发布它们。

1.  创建一个控制台应用程序，使用`WebClientAdapter`的实例（如*步骤 2*中创建的）并订阅这三个事件。

1.  通过订阅`DownloadCompleted`事件，您应该在控制台指示成功。

1.  通过订阅`DownloadProgressChanged`，您应该在控制台报告进度消息，显示`ProgressPercentage`和`BytesReceived`值。

1.  通过订阅`InvalidUrlRequested`事件，您应该在控制台使用不同的控制台背景颜色显示警告。

1.  使用一个`do`循环允许用户重复输入一个网页地址。这个地址和一个临时的目标文件路径可以传递给`WebClientAdapter.DownloadFile()`，直到用户输入一个空地址来退出。

1.  一旦您运行了带有各种下载请求的控制台应用程序，您应该看到以下类似的输出：

    ```cs
    Enter a URL:
    https://www1.ncdc.noaa.gov/pub/data/swdi/stormevents/csvfiles/StormEvents_details-ftp_v1.0_d1950_c20170120.csv.gz
    Downloading https://www1.ncdc.noaa.gov/pub/data/swdi/stormevents/csvfiles/StormEvents_details-ftp_v1.0_d1950_c20170120.csv.gz...
    Downloading...73% complete (7,758 bytes)
    Downloading...77% complete (8,192 bytes)
    Downloading...100% complete (10,597 bytes)
    Downloaded to C:\Temp\StormEvents_details-ftp_v1.0_d1950_c20170120.csv.gz
    Enter a URL:
    https://www1.ncdc.noaa.gov/pub/data/swdi/stormevents/csvfiles/StormEvents_details-ftp_v1.0_d1954_c20160223.csv.gz
    Downloading https://www1.ncdc.noaa.gov/pub/data/swdi/stormevents/csvfiles/StormEvents_details-ftp_v1.0_d1954_c20160223.csv.gz...
    Downloading...29% complete (7,758 bytes)
    Downloading...31% complete (8,192 bytes)
    Downloading...54% complete (14,238 bytes)
    Downloading...62% complete (16,384 bytes)
    Downloading...84% complete (22,238 bytes)
    Downloading...93% complete (24,576 bytes)
    Downloading...100% complete (26,220 bytes)
    Downloaded to C:\Temp\StormEvents_details-ftp_v1.0_d1954_c20160223.csv.gz
    ```

通过完成这个活动，您已经看到了如何从现有的.NET 基于事件的发布者类（`WebClient`）订阅事件，在将它们重新发布到您的适配器类（`WebClientAdapter`）之前，根据您自己的规格进行适配，最终这些事件由控制台应用程序订阅。

注意

这个活动的解决方案可以在[`packt.link/qclbF`](https://packt.link/qclbF)找到。

# 摘要

在本章中，您深入了解了委托。您创建了自定义委托，并看到了它们如何被现代的内置`Action`和`Func`委托所替代。通过使用空引用检查，您发现了调用委托的安全方式，以及如何将多个方法链接在一起形成多播委托。您进一步扩展了委托，使用`event`关键字来限制调用，并遵循定义和调用事件时的首选模式。最后，您涵盖了简洁的 lambda 表达式风格，并看到了如何通过识别捕获和闭包的使用来避免错误。

在下一章中，您将了解 LINQ 和数据结构，这是 C#语言的基本组成部分。
