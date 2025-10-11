# 第二章. 消除对 Xamarin.iOS 的神秘感

现在我们对 Mono 和 Xamarin 有了基本的了解，让我们深入探讨一下 Xamarin.iOS 的工作原理。本章涵盖了以下主题：

+   Xamarin.iOS 和 AOT 编译

+   Mono 组件

+   Xamarin.iOS 绑定

+   Xamarin.iOS 应用的内存管理

+   XIB 和故事板代码生成

+   Xamarin.iOS 设计器

# Xamarin.iOS 和即时编译

与大多数 Mono 或 .NET 应用不同，Xamarin.iOS 应用是静态编译的，编译是通过 Mono 的 **即时编译**（**AOT**）功能完成的。AOT 用于满足苹果的要求，例如，iOS 应用用于编译，避免使用即时编译功能或在虚拟机上运行。

使用 AOT 编译在 C# 语言方面带来了一些限制。在讨论了将 iOS 绑定到 C# 和 .NET 的方法之后，这些限制将更容易讨论。这就是为什么我们将这个主题推到了本章后部分的 *使用 AOT 编译的限制* 部分。

### 注意

有关 Mono AOT 编译的更多信息，请参阅以下链接：

[`www.mono-project.com/AOT#Full_AOT`](http://www.mono-project.com/AOT#Full_AOT)

# 理解 Mono 组件

Xamarin.iOS 随带一个扩展的 Silverlight 和桌面 .NET 组件集。这些库为开发者提供了 .NET 运行时库支持，包括 `System.IO` 和 `System.Threading` 等命名空间。

Xamarin.iOS 与为不同配置编译的组件不兼容，这意味着您的代码必须重新编译以生成针对 Xamarin.iOS 配置的组件。如果您针对其他配置，如 Silverlight 或 .NET 4.5，也需要做同样的事情。

### 提示

有关随 Xamarin.iOS 一起提供的组件集的完整列表，请参阅以下链接：

[`docs.xamarin.com/guides/ios/under_the_hood/assemblies`](http://docs.xamarin.com/guides/ios/under_the_hood/assemblies)

# Xamarin.iOS 绑定

在本节中，您将发现支撑 Xamarin.iOS 的主要力量之一。它附带了一套绑定库，为 iOS 开发提供支持。接下来，我们将详细介绍这些绑定的一些细节。

## 设计原则

一系列目标或设计原则指导了绑定库的开发。这些原则对于使 C# 开发者在 iOS 开发中保持高效至关重要。以下是对设计原则的总结：

+   允许开发者以与其他 .NET 类相同的方式子类化 Objective-C 类

+   提供调用任意 Objective-C 库的方法

+   将常见的 Objective-C 任务转化为更简单的事情，同时使困难的 Objective-C 任务变得可完成

+   将 Objective-C 属性暴露为 C# 属性，同时暴露强类型 API

+   当可能时，使用原生 C# 类型代替 Objective-C 类型

+   支持 C# 事件和 Objective-C 委托，并将 C# 委托暴露给 Objective-C API

    ### 提示

    本节为您提供了需要牢记的原则的一般概念。如果您想找到完整的讨论，可以参考以下链接提供的官方文档：

    [`docs.xamarin.com/guides/ios/under_the_hood/api_design/`](http://docs.xamarin.com/guides/ios/under_the_hood/api_design/)

## C# 类型与类型安全性

Xamarin.iOS 绑定旨在使用 C# 开发者熟悉的类型，并在可能的情况下提高类型安全性。

例如，API 使用 C# 字符串而不是 `NSString`，这意味着 `UILabel` 中的文本属性在 iOS SDK 中定义如下：

```cs
@property(nonatomic, copy) NSString *text
```

此外，在 Xamarin.iOS 中是这样暴露的：

```cs
public string Text { get; set; }
```

在幕后，框架负责将 C# 类型映射到 iOS SDK 所期望的适当类型。

另一个例子是 `NSArray` 的处理。Xamarin.iOS 不是暴露弱类型数组，而是向 `UIView` 的以下 Objective-C 属性暴露强类型数组：

```cs
@property(nonatomic, readonly, copy) NSArray *subviews
```

这以下列方式暴露为 C# 属性：

```cs
UIView[] Subviews { get; }
```

## 继承的使用

Xamarin.iOS 允许你以扩展任何 C# 类型的方式扩展任何 Objective-C 类型，并且像从重写方法中调用 "base" 这样的功能按预期工作。

## 映射 Objective-C 委托

在 Objective-C 中，委托设计模式被广泛用于将责任分配给各种对象。Xamarin 在将 iOS 委托映射到 C# 和 .NET 时面临了一些固有的挑战。

在 Objective-C 中，委托作为响应一组方法的对象来实现。这组方法通常定义为协议，尽管它与 C# 接口相似，但实际上 C# 接口和 Objective-C 协议之间存在显著差异：

+   在 C# 中，实现接口的对象必须实现接口上定义的所有方法

+   另一方面，在 Objective-C 中，采用协议的对象不需要在给定情况下实现协议的方法

另一个挑战是，在许多方面，传统的 .NET 框架更多地依赖于事件来实现类似的功能，并且事件模型对 .NET 开发者来说更为熟悉。

考虑到这些差异，并希望尽可能使 Xamarin.iOS 对 C# 开发者来说直观，同时不妥协 iOS 架构，Xamarin.iOS 提供了四种不同的方式来实现委托功能：

+   通过 .NET 事件

+   通过 .NET 属性

+   通过强类型委托

+   通过弱类型委托

### 通过 .NET 事件

对于许多类型，Xamarin.iOS 自动创建适当的委托并将委托调用转发到相应的 .NET 事件。这使得开发体验对 C# 和 .NET 开发者来说非常自然。

`UIWebView`是这一点的良好示例。iOS 定义了`UIWebViewDelegate`，其中包含了一系列方法，如果分配了包含以下内容的委托，`UIWebView`将转发这些方法：

+   `webViewDidStartLoad`

+   `webViewDidFinishLoad`

+   `webView:didFailLoadWithError`

在 Xamarin.iOS 类`MonoTouch.UIKit.UIWebView`中，我们发现有三个事件对应以下方法：

+   `LoadStarted`

+   `LoadFinished`

+   `LoadError`

### 通过.NET 属性

尽管事件具有拥有多个订阅者的优势，但它们也带来自己的限制。具体来说，这可能就是事件不能有返回类型的地方。在需要返回值的委托方法的情况下，使用委托属性。以下示例展示了如何使用委托属性`UITextField`。在这个例子中，一个匿名方法被分配给`UITextField`上的委托属性`ShouldReturn`：

```cs
firstNameTextField.ShouldReturn = delegate (textfield)
{
      textfield.ResignFirstResponder ();
      return true;
}
```

### 通过强类型委托

如果没有提供事件或委托属性，或者你更愿意使用委托，你将很高兴地听到 Xamarin.iOS 提供了一套.NET 类，对应于每个 iOS 委托。这些类包含对应协议上定义的每个方法的定义。需要实现的方法被定义为抽象的，而可选的方法被定义为虚拟的。要使用这些委托之一，开发者只需创建一个新的类，继承自所需的委托，并重写需要实现的方法。

为了举例说明如何使用强类型委托，我们将转向`UITableViewDataSource`。这是 iOS 定义的用于填充`UITableView`实例的协议。以下示例演示了一个可以用于用电话号码填充`UITableView`的数据源：

```cs
public class PhoneDataSource : UITableViewDataSource
{
    List<string>_phoneList;
    public PhoneDataSource (List<string> phoneList) {
    _phoneList = phoneList;
}
public override int RowsInSection(UITableView
tableView, int section)
{
  return _phoneList.Count;
}
public override UITableViewCell GetCell(UITableView
  tableView, NSIndexPath indexPath) {
      ... // create and populate UITableViewCell
      return cell;
}
}
```

现在我们已经创建了委托，我们需要将其分配给一个`UITableView`实例。`UITableViewDataSource`委托的属性名为`Source`，以下代码展示了如何进行分配：

```cs
phoneTableView.Source = new PhoneDataSource(phoneList);
```

### 通过弱类型委托

最后，Xamarin.iOS 为你提供了一种使用弱类型委托的方法。不幸的是，这种方法需要开发者做更多的工作。

在 Xamarin.iOS 中，可以使用继承自`NSObject`的任何类创建弱委托。在创建弱委托时，你将负责使用`Export`属性正确装饰你的类，这实际上教会 iOS 如何映射方法。以下示例展示了具有适当属性规范的弱委托：

```cs
public class WeakPhoneDataSource : NSObject
{
...
[Export ("tableView:numberOfRowsInSection:")]
public override int RowsInSection(UITableView
      tableView, int section)
  {
    ...
  }
[Export ("tableView:cellForRowAtIndexPath:")]
  public override UITableViewCell GetCell(UITableView
      tableView, NSIndexPath indexPath) {
...
  }
}
```

最后几个步骤将弱委托分配给一个`UITableView`实例。按照 Xamarin.iOS 的约定，弱委托属性名称总是以`Weak`开头。以下示例展示了如何分配弱数据源委托：

```cs
phoneTableView.WeakSource =
    new WeakPhoneDataSource(...);
```

一旦弱委托被分配，任何已分配的强委托将不再接收调用。

# 创建绑定库

可能会有这样的情况，你需要为 Xamarin.iOS 未提供且在 Xamarin 组件商店找不到的 Objective-C 库创建自己的绑定库。Xamarin 提供了大量的指导来创建绑定，以及一个自动化的工具来帮助处理一些繁琐的工作。以下链接提供了创建 Objective-C 库自定义绑定的指导：

| 信息类型 | 访问它的 URL |
| --- | --- |
| 一般绑定信息 | [`docs.xamarin.com/guides/ios/advanced_topics/binding_objective-c/`](http://docs.xamarin.com/guides/ios/advanced_topics/binding_objective-c/) |
| Objective Sharpie 自动化工具的使用 | [`docs.xamarin.com/guides/ios/advanced_topics/binding_objective-c/objective_sharpie/`](http://docs.xamarin.com/guides/ios/advanced_topics/binding_objective-c/objective_sharpie/) |
| 绑定类型参考 | [`docs.xamarin.com/guides/ios/advanced_topics/binding_objective-c/binding_types_reference_guide/`](http://docs.xamarin.com/guides/ios/advanced_topics/binding_objective-c/binding_types_reference_guide/) |

# 内存管理

当涉及到释放资源时，Xamarin.iOS 通过**垃圾回收器**（**GC**）为你处理这些，它代表你完成这些工作。除此之外，所有从`NSObject`派生的对象都利用`System.IDisposable`接口，这样当内存释放时，开发者可以对其有所控制。

`NSObject`不仅实现了`IDisposable`接口，还遵循.NET 销毁模式。`IDisposable`接口只需要实现一个方法，即`Dispose()`。销毁模式需要实现一个额外的`Dispose(bool disposing)`方法。销毁参数指示该方法是否从`Dispose()`方法调用，在这种情况下值为`true`，或者从`Finalize`方法调用，在这种情况下值为`false`。

销毁参数旨在用于确定是否应该释放托管对象。如果值为`true`，则应释放托管对象。无论值如何，都应释放非托管对象。以下代码演示了`Dispose()`方法中应该包含的内容：

```cs
public void Dispose ()
{
  this.Dispose (true);
  GC.SuppressFinalize (this);
}
```

注意`Dispose(bool disposing)`的调用，其值为`true`。方便的是，框架为你实现了`Dispose()`方法，作为`NSObject`上的虚拟方法。以下代码演示了`Dispose(bool disposing)`方法的实现：

```cs
class MyViewController : UIViewController {

  UIImagemyImage;

  . . .   

  public override void Dispose (bool disposing)
  {
    if (disposing){
      if (myImage!= null) {
        myImage.Dispose ();
        myImage = null;
      }
    }

    // Free unmanaged objects regardless of value.

    base.Dispose (disposing)

  }
}
```

### 注意

再次注意对`base.Dispose(disposing)`的调用。这个调用非常重要，因为它处理框架内部管理的资源。

为什么会有这么大的麻烦？为什么不在 `Dispose()` 中清理一切？答案在于垃圾回收器。垃圾回收器销毁对象的顺序未定义，因此不可预测且可能发生变化。.NET 释放模式有助于防止终结器在已释放的对象上调用 `Dispose()`。

### 小贴士

您可以在以下链接中了解更多关于 .NET 释放模式的信息：

[`msdn.microsoft.com/en-us/library/fs2xkftw.aspx`](http://msdn.microsoft.com/en-us/library/fs2xkftw.aspx)

释放托管对象使其变得无用。即使该对象的引用可能仍然存在，您也需要以假设已释放的对象不再有效的方式来构建您的软件。在某些情况下，当访问已释放对象的成员方法时，将抛出 `ObjectDisposedException`。

## 对象的释放

任何时候您有一个持有大量资源且不再需要的对象，请调用 `Dispose()` 方法。GC 很方便且相当复杂，但可能无法完全了解特定对象分配的资源量，尤其是如果这些资源与非托管对象相关。

## 保持对象

要防止对象被销毁，您只需确保至少有一个对象引用被维护。一旦对象的引用计数达到 `0`，GC 就会高兴地调用其上的 `Dispose()` 方法，并且对象将不再可用。

# 使用 AOT 编译的限制

正如我们在本章前面提到的，使用 AOT 编译有一些限制。以下各节概述了由于使用 AOT 编译而由 Xamarin.iOS 施加的限制：

+   不允许 `NSObject` 的泛型子类。以下将不允许，因为 `UIViewController` 是 `NSObject` 的子类：

    ```cs
    class MainViewController<T> : UIViewController {
    ...
    }
    ```

+   P/Invoke 在泛型类中不受支持，因此以下在 Xamarin.iOS 中不受支持：

    ```cs
    class MyGenericType<T> {
        [DllImport ("System")]
        public static extern int getpid ();
    }
    ```

+   在 `Nullable<T> 类型` 上不支持 `Property.SetInfo()`。使用反射的 `Property.SetInfo()` 在 `Nullable<T>` 上设置值目前不受支持。

+   不允许动态代码生成。iOS 内核阻止应用程序动态生成代码，因此 Xamarin.iOS 强制实施以下限制：

    +   无论是 `System.Reflection.Emit` 还是 `System.Runtime.Remoting` 都不可用

    +   不支持动态创建类型

    +   反向回调必须在编译时在运行时注册

+   对于反向回调，还有进一步的限制。在 Mono 中，您可以将 C# 委托传递给非托管代码，而不是传递函数指针。AOT 的使用对此施加了一些限制：

    +   回调方法必须使用 Mono 属性 `MonoPInvokeCallbackAttribute` 标记

    +   回调方法必须是静态的；不支持实例方法

## 禁用的运行时功能

在 Xamarin.iOS 中禁用了以下功能：

+   分析器

+   `Reflection.Emit` 功能

+   `Reflection.Emit.Save`功能

+   COM 绑定

+   JIT 引擎

+   元数据验证器（由于没有 JIT）

# 为 XIB 和 Storyboard 文件生成代码

Apple Interface Builder 是 Xcode 中内置的设计器，允许进行用户界面的可视化设计。使用 Interface Builder 是可选的；用户界面可以完全使用 iOS API 构建。Interface Builder 创建的定义保存在 XIB 或 Storyboard 文件中，区别在于 XIB 文件通常包含单个视图。另一方面，Storyboard 包含一组视图以及视图之间的过渡或 segues。

Xamarin Studio 与 Xcode 协同工作以支持 UI 设计。当在 Xamarin Studio 中双击 Storyboard 或 XIB 文件时，将启动 Xcode 以方便设计 UI。一旦在 Xcode 中保存更改并切换回 Xamarin Studio，就会生成 C#代码以支持在 Xcode 中捕获的 UI 设计。以下各节将更详细地描述此过程。

## 生成的类

Xamarin Studio 为 XIB 文件或 Storyboard 文件中找到的每个自定义类生成两个文件，一个设计器文件和一个非设计器文件。例如，名为`LoginViewController`的视图控制器将导致以下文件生成：

+   `LoginViewController.cs`

+   `LoginViewController.designer.cs`

这些文件是在 Xcode 中保存更改后生成的，并且 Xamarin Studio 获得焦点。

### 设计器文件

设计器文件包含在 XIB 或 Storyboard 文件中找到的自定义类的部分类定义，为输出创建属性，并为找到的操作创建部分方法。以下示例是一个具有两个`UITextField`控件和一个单个`UIButton`控件的视图控制器：

```cs
[Register ("LoginViewController")]
partial class LoginViewController
{
  [Outlet]
  MonoTouch.UIKit.UITextField textPassword { get; set; }
  [Outlet]
  MonoTouch.UIKit.UITextField textUserId { get; set; }
  [Action ("btnLoginClicked:")]
  partial void btnLoginClicked
         (MonoTouch.Foundation.NSObject sender);
void ReleaseDesignerOutlets ()
{
if (textUserId != null) {
      textUserId.Dispose ();
textUserId = null;
}
if (textPassword != null) {
textPassword.Dispose ();
textPassword = null;
}
}
```

### 注意

一旦修改了 XIB 或 Storyboard 文件，设计器文件将自动更新。因此，不应手动修改它们，因为任何更改在 Xamarin Studio 更新它们时都将丢失。

### 非设计器文件

设计器文件单独使用，但与一个非设计器文件结合使用。非设计器文件包含部分类规范，它完成了其对应设计器文件中定义的类。非设计器文件标识基类，定义了实例化类所需的构造函数，并提供了一个位置来实现功能，无论是通过为部分方法提供实现，还是通过在基类上重写虚拟方法。以下示例显示了一个具有重写和部分方法实现的非设计器文件：

```cs
public partial class LoginViewController : UIViewController
{
  public LoginViewController (IntPtr handle) :
    base (handle)
{
}
  public override void ViewDidLoad ()
  {
    base.ViewDidLoad ();
    // logic to perform when view has loaded...
  }
partial void btnLoginClicked (NSObject sender)
{
// logic for login goes here...
}
}
```

### 小贴士

注意，此文件中部分方法的实现是为了响应在相应的 XIB 或 Storyboard 文件中定义的操作而生成的设计器文件中的方法。

对非设计器文件所做的更改不会丢失，因为这些文件仅在 Xamarin Studio 首次遇到新自定义类时创建，并且随后不会更新。

### 输出属性

设计器类包含私有属性，这些属性对应于在自定义类上定义的输出，然后可以从非设计器文件中找到的 `CodeBehind` 类中使用。如果你需要将这些属性设置为公共的，你只需要将访问器属性添加到非设计器文件中，就像为任何给定的私有字段做的那样。

### 动作属性

设计器文件具有包含与自定义类上定义的所有动作相关联的部分方法的属性。你应该注意，这些方法不包含实现，它们具有双重作用：

+   它的第一个目的是，当你将部分代码插入非设计器文件的类体中时，Xamarin Studio 将会提供自动完成所有未实现的部分方法签名，这允许开发者实现动作的逻辑。

+   它的另一个目的是它们的签名上应用了一个属性，使它们暴露给 Objective-C 世界。因此，一旦在 iOS 中触发相应的动作，它们就可以被调用。

# Xamarin.iOS 设计器

Xamarin 提供了 Apple 的 Interface Builder 的替代方案。Xamarin.iOS 设计器是用于 Xamarin Studio 环境的一个插件，它为 iOS storyboards 添加了全功能的拖放用户界面设计，所有操作都可以在 Xamarin Studio 内完成。Xamarin.iOS 设计器提供了以下关键特性：

+   **兼容的 storyboard 格式**：正如你所期望的，Xamarin.iOS 设计器创建的 storyboards 使用的是与 Xcode 和 iOS SDK 相同的格式，因此在某些时候切换回 Xcode 是允许的

+   **消除与 Xcode 的同步**：使用 Xamarin.iOS 设计器消除了在开发过程中使用 Xcode 的需要，以及可能发生在 Xamarin Studio 和 Xcode 之间的同步问题

+   **简单属性**：Xamarin.iOS 设计器在控件被拖放到视图中时自动创建引用控件的属性

+   **简单的事件处理程序**：Xamarin.iOS 设计器提供了一个更直观的方式来创建事件处理程序，它们的工作方式与 Visual Studio 在其他 UI 框架（如 Silverlight 和 WPF）上的工作方式非常相似

+   **自定义控件**：用户可以在工具箱面板中创建自己的自定义 UI 控件

Xamarin.iOS 设计器只能用于创建 storyboards。如果你更喜欢或需要处理 XIB 文件，你将需要继续使用 Xcode。

# 概述

在本章中，我们介绍了 Xamarin.iOS 架构的要点，并试图揭示 Xamarin.iOS 允许开发者使用 C# 和 Mono 为 iOS 创建优秀原生应用的方式。虽然我们显然没有涵盖整个 iOS SDK，但我们已经描述了构建 Xamarin.iOS 所使用的方法和原则。有了这些知识，你应该能够顺利地继续进行 Xamarin.iOS 开发，并在出现问题时解决它们。

在下一章中，我们将尝试为 Xamarin.Android 实现相同的目标。
