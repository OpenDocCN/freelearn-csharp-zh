# 第十三章：13

# 探索通用、委托和更多

你在编程中花费的时间越多，你就会开始思考系统。构建类和对象如何相互交互、通信和交换数据，这些都是我们迄今为止所使用的系统的例子；现在的问题是如何使它们更安全、更高效。

由于这将是本书的最后一个实用章节，我们将介绍通用编程概念、委托、事件创建和错误处理的示例。每个主题都是一个独立的大领域，所以在你的项目中学到的东西，可以进一步扩展。在完成我们的实际编码后，我们将简要概述设计模式以及它们在你未来编程之旅中的作用。

在本章中，我们将涵盖以下主题：

+   通用编程

+   使用委托

+   创建事件和订阅

+   抛出和处理错误

+   理解设计模式

# 介绍通用

到目前为止，我们的所有代码在定义和使用类型方面都非常具体。然而，会有一些情况，你需要一个类或方法以相同的方式处理其实体，而不管其类型，同时仍然是类型安全的。通用编程允许我们使用占位符而不是具体类型来创建可重用的类、方法和变量。

当在编译时创建通用类实例或使用方法时，将分配一个具体类型，但代码本身将其视为通用类型。能够编写通用代码是一个巨大的好处，当你需要以相同的方式处理不同的对象类型时，例如需要能够对元素执行相同操作的自定义集合类型，或者需要相同底层功能的类。虽然你可能会问为什么我们不只是子类化或使用接口，但在我们的例子中，你会看到通用类以不同的方式帮助我们。

我们已经在`List`类型中看到了这一点，它是一种通用类型。无论它存储整数、字符串还是单个字符，我们都可以访问它的所有添加、删除和修改函数。

## 通用对象

创建通用类的方式与创建非通用类的方式相同，但有一个重要的区别：它的通用类型参数。让我们看一个我们可能想要创建的通用集合类的例子，以更清晰地了解它是如何工作的：

```cs
public class SomeGenericCollection**<****T****>** {} 
```

我们声明了一个名为`SomeGenericCollection`的通用集合类，并指定其类型参数将被命名为`T`。现在，`T`将代表通用列表将存储的元素类型，并且可以在通用类内部像任何其他类型一样使用。

每当我们创建一个`SomeGenericCollection`的实例时，我们需要指定它可以存储的值的类型：

```cs
SomeGenericCollection**<****int****>** highScores = new SomeGenericCollection<int>(); 
```

在这种情况下，`highScores`存储整数值，`T`代表`int`类型，但`SomeGenericCollection`类将以相同的方式处理任何元素类型。

你完全可以控制通用类型参数的命名，但在许多编程语言中，行业标准是使用大写的`T`。如果你要为你的类型参数命名不同的名称，考虑以大写的`T`开头以保持一致性和可读性。

让我们接下来创建一个更加游戏化的例子，使用通用的`Shop`类来存储一些虚构的库存物品，具体步骤如下：

1.  在`Scripts`文件夹中创建一个新的 C#脚本，命名为`Shop`，并将其代码更新为以下内容：

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// 1
public class Shop<T>
{
    // 2
    public List<T> inventory = new List<T>();
} 
```

1.  在`GameBehavior`中创建一个`Shop`的新实例：

```cs
public class GameBehavior : MonoBehaviour, IManager
{
    // ... No other changes needed ...

    public void Initialize()
    {
        // 3
        var itemShop = new Shop<string>();
        // 4
        Debug.Log("There are " + itemShop.inventory.Count + " items for sale.");
    }
} 
```

让我们来分解一下代码：

1.  声明一个名为`IShop`的新通用类，带有`T`类型参数

1.  添加一个类型为`T`的库存`List<T>`，用于存储我们用通用类初始化的任何物品类型

1.  在`GameBehavior`中创建一个`Shop<string>`的新实例，并指定字符串值作为通用类型

1.  打印出一个带有库存计数的调试消息：![](img/B17573_13_01.png)

图 13.1：来自泛型类的控制台输出

在功能方面还没有发生任何新的事情，但是 Visual Studio 因为其泛型类型参数`T`而将`Shop`识别为泛型类。这使我们能够包括其他泛型操作，如添加库存项目或查找每种项目的数量。

值得注意的是，Unity Serializer 默认不支持泛型。如果要序列化泛型类，就像我们在上一章中对自定义类所做的那样，您需要在类的顶部添加`Serializable`属性，就像我们在`Weapon`类中所做的那样。您可以在[`docs.unity3d.com/ScriptReference/SerializeReference.html`](https://docs.unity3d.com/ScriptReference/SerializeReference.html)找到更多信息。

## 泛型方法

一个独立的泛型方法可以有一个占位符类型参数，就像一个泛型类一样，这使它可以根据需要包含在泛型或非泛型类中：

```cs
public void GenericMethod**<****T****>**(**T** genericParameter) {} 
```

`T`类型可以在方法体内使用，并在调用方法时定义：

```cs
GenericMethod**<****string****>(****"Hello World!"****)**; 
```

如果要在泛型类中声明泛型方法，则不需要指定新的`T`类型：

```cs
public class SomeGenericCollection<T> 
{
    public void NonGenericMethod(**T** genericParameter) {}
} 
```

当调用使用泛型类型参数的非泛型方法时，没有问题，因为泛型类已经处理了分配具体类型的问题：

```cs
SomeGenericCollection**<****int****>** highScores = new SomeGenericCollection
<int> ();
highScores.NonGenericMethod(**35**); 
```

泛型方法可以被重载并标记为静态，就像非泛型方法一样。如果您想要这些情况的具体语法，请查看[`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods)。

您的下一个任务是创建一个方法，将新的泛型项目添加到库存，并在`GameBehavior`脚本中使用它。

由于我们已经有了一个具有定义类型参数的泛型类，让我们添加一个非泛型方法来看它们如何一起工作：

1.  打开`Shop`并按以下方式更新代码：

```cs
public class Shop<T>
{
    public List<T> inventory = new List<T>();
    **// 1**
    **public****void****AddItem****(****T newItem****)**
    **{**

        **inventory.Add(newItem);**
    **}**
} 
```

1.  进入`GameBehavior`并向`itemShop`添加一个项目：

```cs
public class GameBehavior : MonoBehaviour, IManager
{
    // ... No other changes needed ...

     public void Initialize()
    {
        var itemShop = new Shop<string>();
        **// 2**
        itemShop**.AddItem(****"Potion"****);**
        itemShop**.AddItem(****"Antidote"****);**
       Debug.Log("There are " + itemShop.inventory.Count + " items for sale.");
    }
} 
```

让我们来分解代码：

1.  声明一个添加`newItems`的类型`T`到库存的方法

1.  使用`AddItem()`向`itemShop`添加两个字符串项目，并打印出调试日志：![](img/B17573_13_02.png)

图 13.2：向泛型类添加项目后的控制台输出

我们编写了`AddItem()`以接受与我们的泛型`Shop`实例相同类型的参数。由于`itemShop`被创建为保存字符串值，我们可以毫无问题地添加`"Potion"`和`"Antidote"`字符串值。

然而，如果尝试添加一个整数，例如，您将收到一个错误，指出`itemShop`的泛型类型不匹配：

![](img/B17573_13_03.png)

图 13.3：泛型类中的转换错误

现在，您已经编写了一个泛型方法，需要知道如何在单个类中使用多个泛型类型。例如，如果我们想要向`Shop`类添加一个方法，找出库存中有多少个给定项目？我们不能再次使用类型`T`，因为它已经在类定义中定义了。那么我们该怎么办呢？

将以下方法添加到`Shop`类的底部：

```cs
// 1
public int GetStockCount<U>()
{
    // 2
    var stock = 0;
    // 3
    foreach (var item in inventory)
    {
        if (item is U)
        {
            stock++;
        }
    }
    // 4
    return stock;
} 
```

让我们来分解我们的新方法：

1.  声明一个方法，返回我们在库存中找到的类型`U`的匹配项目的 int 值

+   泛型类型参数的命名完全取决于您，就像命名变量一样。按照惯例，它们从`T`开始，然后按字母顺序继续。

1.  创建一个变量来保存我们找到的匹配库存项目的数量，并最终从库存中返回

1.  使用`foreach`循环遍历库存列表，并在找到匹配时增加库存值

1.  返回匹配库存项目的数量

问题在于我们在商店中存储字符串值，因此如果我们尝试查找我们有多少字符串项目，我们将得到完整的库存：

```cs
Debug.Log("There are " + itemShop.GetStockCount<string>() + " items for sale."); 
```

这将在控制台上打印出类似以下内容：

![](img/B17573_13_04.png)

图 13.4：使用多个泛型字符串类型的控制台输出

另一方面，如果我们试图在我们的库存中查找整数类型，我们将得不到结果，因为我们只存储字符串：

```cs
Debug.Log("There are " + itemShop.GetStockCount<int>() + " items for sale."); 
```

这将在控制台上打印类似以下内容：

![](img/B17573_13_05.png)

图 13.5：使用多个不匹配的泛型类型的控制台输出

这两种情况都不理想，因为我们无法确保我们的商店库存既存储又可以搜索相同的物品类型。但这就是泛型真正发挥作用的地方——我们可以为我们的泛型类和方法添加规则，以强制执行我们想要的行为，我们将在下一节中介绍。

## 约束类型参数

泛型的一大优点是它们的类型参数可以受限制。这可能与我们迄今为止学到的有所矛盾，但只是因为一个类*可以*包含任何类型，并不意味着应该允许它这样做。

为了约束泛型类型参数，我们需要一个新关键字和一个我们以前没有见过的语法：

```cs
public class SomeGenericCollection<T> where T: ConstraintType {} 
```

`where`关键字定义了`T`必须通过的规则，然后才能用作泛型类型参数。它基本上说`SomeGenericClass`可以接受任何`T`类型，只要它符合约束类型。约束规则并不神秘或可怕；它们是我们已经涵盖的概念：

+   添加`class`关键字将限制`T`为类类型

+   添加`struct`关键字将限制`T`为结构类型

+   添加接口，如`IManager`，作为类型将限制`T`为采用该接口的类型

+   添加自定义类，如`Character`，将限制`T`仅为该类类型

如果您需要更灵活的方法来考虑具有子类的类，您可以使用`where T：U`，它指定泛型`T`类型必须是`U`类型或派生自`U`类型。这对我们的需求来说有点高级，但您可以在[`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters)找到更多详细信息。

只是为了好玩，让我们将`Shop`限制为只接受一个名为`Collectable`的新类型：

1.  在`Scripts`文件夹中创建一个新脚本，命名为`Collectable`，并添加以下代码：

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Collectable
{
    public string name;
}

public class Potion : Collectable
{
    public Potion()
    {
        this.name = "Potion";
    }
}

public class Antidote : Collectable
{
    public Antidote()
    {
        this.name = "Antidote";
    }
} 
```

我们在这里所做的只是声明一个名为`Collectable`的新类，具有一个名称属性，并为药水和解毒剂创建了子类。有了这个结构，我们可以强制我们的`Shop`只接受`Collectable`类型，并且我们的库存查找方法也只接受`Collectable`类型，这样我们就可以比较它们并找到匹配项。

1.  打开`Shop`并更新类声明：

```cs
public class Shop<T> **where****T** **:** **Collectable** 
```

1.  更新`GetStockCount()`方法以将`U`约束为与初始泛型`T`类型相等：

```cs
public int GetStockCount<U>() **where** **U : T**
{
    var stock = 0;
    foreach (var item in inventory)
    {
        if (item is U)
        {
            stock++;
        }
    }
    return stock;
} 
```

1.  在`GameBehavior`中，将`itemShop`实例更新为以下代码：

```cs
var itemShop = new Shop<**Collectable**>();
itemShop.AddItem(**new** **Potion()**);
itemShop.AddItem(**new** **Antidote()**);
Debug.Log("There are " + itemShop.GetStockCount<**Potion**>() + " items for sale."); 
```

这将导致类似以下的输出：

![](img/B17573_13_12.png)

图 13.6：更新后的 GameBehavior 脚本输出

在我们的示例中，我们可以确保只有可收集类型被允许在我们的商店中。如果我们在代码中意外地尝试添加不可收集类型，Visual Studio 将警告我们尝试违反我们自己的规则！

## 向 Unity 对象添加泛型

泛型也适用于 Unity 脚本和游戏对象。例如，我们可以轻松地创建一个通用的可销毁类，用于删除场景中的任何`MonoBehaviour`或对象`Component`。如果这听起来很熟悉，那就是我们的`BulletBehavior`为我们做的事情，但它不适用于除该脚本之外的任何东西。为了使其更具可扩展性，让我们使任何从`MonoBehaviour`继承的脚本都可销毁。

1.  在`Scripts`文件夹中创建一个新脚本，命名为`Destroyable`，并添加以下代码：

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Destroyable<T> : MonoBehaviour where T : MonoBehaviour
{
    public int OnscreenDelay;

    void Start()
    {
        Destroy(this.gameObject, OnscreenDelay);
    }
} 
```

1.  删除`BulletBehavior`中的所有代码，并继承自新的通用类：

```cs
public class BulletBehavior : **Destroyable****<****BulletBehavior****>**
{
} 
```

现在，我们已经将我们的`BulletBehavior`脚本转换为通用的可销毁对象。在 Bullet Prefab 中没有任何更改，但我们可以通过从通用的`Destroyable`类继承来使任何其他对象可销毁。在我们的示例中，如果我们创建了多个抛射物 Prefab 并希望它们都在不同的时间被销毁，那么这将提高代码效率和可重用性。

通用编程是我们工具箱中的一个强大工具，但是在掌握了基础知识之后，是时候谈谈编程旅程中同样重要的一个主题——委托了！

# 委托操作

有时您需要将一个文件中的方法执行委托给另一个文件。在 C#中，可以通过委托类型来实现这一点，它存储对方法的引用，并且可以像任何其他变量一样对待。唯一的限制是委托本身和任何分配的方法都需要具有相同的签名——就像整数变量只能保存整数和字符串只能保存文本一样。

创建委托是编写函数和声明变量的混合： 

```cs
public **delegate** returnType DelegateName(int param1, string param2); 
```

您首先使用访问修饰符，然后是`delegate`关键字，这将其标识为`delegate`类型。`delegate`类型可以像常规函数一样具有返回类型和名称，如果需要还可以有参数。但是，这种语法只是声明了`delegate`类型本身；要使用它，您需要像使用类一样创建一个实例：

```cs
public **DelegateName** someDelegate; 
```

声明了一个`delegate`类型变量后，很容易分配一个与委托签名匹配的方法：

```cs
public DelegateName someDelegate = **MatchingMethod**;
public void **MatchingMethod****(****int** **param1,** **string** **param2****)** 
{
    // ... Executing code here ...
} 
```

请注意，在将`MatchingMethod`分配给`someDelegate`变量时，不要包括括号，因为此时并不是在调用该方法。它所做的是将`MatchingMethod`的调用责任委托给`someDelegate`，这意味着我们可以如下调用该函数：

```cs
someDelegate(); 
```

在您的 C#技能发展到这一点时，这可能看起来很麻烦，但我向您保证，能够将方法存储和执行为变量将在未来派上用场。

## 创建一个调试委托

让我们创建一个简单的委托类型来定义一个接受字符串并最终使用分配的方法打印它的方法。打开`GameBehavior`并添加以下代码：

```cs
public class GameBehavior : MonoBehaviour, IManager
{
    // ... No other changes needed ...

    **// 1**
    **public****delegate****void****DebugDelegate****(****string** **newText****)****;**

    **// 2**
    **public** **DebugDelegate debug = Print;**

    public void Initialize() 
    {
        _state = "Game Manager initialized..";
        _state.FancyDebug();
        **// 3**
        **debug(_state);**
   // ... No changes needed ...
    }
    **// 4**
    **public****static****void****Print****(****string** **newText****)**
    **{**
        **Debug.Log(newText);**
    **}**
} 
```

让我们分解一下代码：

1.  声明一个名为`DebugDelegate`的`public delegate`类型，用于保存一个接受`string`参数并返回`void`的方法

1.  创建一个名为`debug`的新`DebugDelegate`实例，并为其分配一个具有匹配签名的方法`Print()`

1.  用`debug`委托实例替换`Initialize()`中的`Debug.Log(_state)`代码

1.  声明`Print()`为一个接受`string`参数并将其记录到控制台的`static`方法

图 13.7：委托操作的控制台输出

控制台中没有任何变化，但是现在`Initialize()`中不再直接调用`Debug.Log()`，而是将该操作委托给了`debug`委托实例。虽然这是一个简单的例子，但是当您需要存储、传递和执行方法作为它们的类型时，委托是一个强大的工具。

在 Unity 中，我们已经通过使用`OnCollisionEnter()`和`OnCollisionExit()`方法来处理委托的示例，这些方法是通过委托调用的。在现实世界中，自定义委托在与事件配对时最有用，我们将在本章的后面部分看到。

## 委托作为参数类型

既然我们已经看到如何创建委托类型来存储方法，那么委托类型本身也可以作为方法参数使用就是合情合理的。这与我们已经做过的并没有太大的不同，但是涵盖基础知识是个好主意。

让我们看看委托类型如何作为方法参数使用。使用以下代码更新`GameBehavior`：

```cs
public class GameBehavior : MonoBehaviour, IManager
{
    // ... No changes needed ...
    public void Initialize() 
    {
        _state = "Game Manager initialized..";
        _state.FancyDebug();
        debug(_state);
        **// 1**
        **LogWithDelegate(debug);**
    }
    **// 2**
    **public****void****LogWithDelegate****(****DebugDelegate del****)**
    **{**
        **// 3**
        **del(****"Delegating the debug task..."****);**
    **}**
} 
```

让我们分解一下代码：

1.  调用`LogWithDelegate()`并将我们的`debug`变量作为其类型参数传递

1.  声明一个新的方法，接受`DebugDelegate`类型的参数

1.  调用委托参数的函数，并传入一个字符串文字以打印出来：![](img/B17573_13_07.png)

图 13.8：委托作为参数类型的控制台输出

我们创建了一个接受`DebugDelegate`类型参数的方法，这意味着传入的实际参数将表示一个方法，并且可以被视为一个方法。将这个例子视为一个委托链，其中`LogWithDelegate()`距离实际进行调试的方法`Print()`有两个步骤。创建这样的委托链并不总是在游戏或应用程序场景中常见的解决方案，但是当您需要控制委托级别时，了解涉及的语法是很重要的。在涉及到委托链跨多个脚本或类的情况下，这一点尤为重要。

如果您错过了重要的心理联系，很容易在委托中迷失，所以回到本节开头的代码并查看文档：[`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)。

现在您知道如何处理基本委托了，是时候谈谈事件如何用于在多个脚本之间高效地传递信息了。老实说，委托的最佳用例是与事件配对使用，接下来我们将深入探讨。

# 触发事件

C#事件允许您基本上创建一个基于游戏或应用程序中的操作的订阅系统。例如，如果您想在收集物品时发送事件，或者当玩家按下空格键时，您可以这样做。然而，当事件触发时，并不会自动有一个订阅者或接收者来处理任何需要在事件动作之后执行的代码。

任何类都可以通过调用事件被触发的类来订阅或取消订阅事件；就像在手机上注册接收 Facebook 上分享新帖子通知一样，事件形成了一种分布式信息高速公路，用于在应用程序中共享操作和数据。

声明事件类似于声明委托，因为事件具有特定的方法签名。我们将使用委托来指定我们希望事件具有的方法签名，然后使用`delegate`类型和`event`关键字创建事件：

```cs
public delegate void EventDelegate(int param1, string param2);
public **event** EventDelegate eventInstance; 
```

这个设置允许我们将`eventInstance`视为一个方法，因为它是一个委托类型，这意味着我们可以随时调用它来发送它：

```cs
eventInstance(35, "John Doe"); 
```

你的下一个任务是在`PlayerBehavior`内部创建一个自己的事件并在适当的位置触发它。

## 创建和调用事件

让我们创建一个事件，以便在玩家跳跃时触发。打开`PlayerBehavior`并添加以下更改：

```cs
public class PlayerBehavior : MonoBehaviour 
{
    // ... No other variable changes needed ...
    **// 1**
    **public****delegate****void****JumpingEvent****()****;**
    **// 2**
    **public****event** **JumpingEvent playerJump;**
    void Start()
    {
        // ... No changes needed ...
    }
    void Update() 
    {
        // ... No changes needed ...
;
    }
    void FixedUpdate()
    {
        if(IsGrounded() &&  _isJumping)
        {
            _rb.AddForce(Vector3.up * jumpVelocity,
               ForceMode.Impulse);
            **// 3**
            **playerJump();**
        }
    }
    // ... No changes needed in IsGrounded or OnCollisionEnter
} 
```

让我们来分解一下代码：

1.  声明一个返回`void`并且不带任何参数的新`delegate`类型

1.  创建一个`JumpingEvent`类型的事件，名为`playerJump`，可以被视为一个方法，与前面的委托的`void`返回和无参数签名相匹配

1.  在`Update()`中施加力后调用`playerJump`

我们已成功创建了一个简单的委托类型，它不带任何参数并且不返回任何内容，以及一个该类型的事件，以便在玩家跳跃时执行。每次玩家跳跃时，`playerJump`事件都会发送给所有订阅者，通知它们该操作。

事件触发后，由订阅者来处理它并执行任何额外的操作，我们将在*处理事件订阅*部分中看到。

## 处理事件订阅

现在，我们的`playerJump`事件没有订阅者，但更改很简单，非常类似于我们在上一节中将方法引用分配给委托类型的方式：

```cs
someClass.eventInstance += EventHandler; 
```

由于事件是属于声明它们的类的变量，而订阅者将是其他类，因此需要引用包含事件的类来进行订阅。`+=`运算符用于分配一个方法，当事件执行时将触发该方法，就像设置一个外出邮件一样。与分配委托一样，事件处理程序方法的方法签名必须与事件的类型匹配。在我们之前的语法示例中，这意味着`EventHandler`需要是以下内容：

```cs
public void EventHandler(int param1, string param2) {} 
```

在需要取消订阅事件的情况下，您只需使用`-=`运算符执行分配的相反操作：

```cs
someClass.eventInstance -= EventHandler; 
```

事件订阅通常在类初始化或销毁时处理，这样可以轻松管理多个事件，而不会出现混乱的代码实现。

现在您已经知道了订阅和取消订阅事件的语法，现在轮到您在`GameBehavior`脚本中将其付诸实践了。

现在，我们的事件每次玩家跳跃时都会触发，我们需要一种捕获该动作的方法：

1.  返回到`GameBehavior`并更新以下代码：

```cs
public class GameBehavior : MonoBehaviour, IManager
{
    // 1
    public PlayerBehavior playerBehavior;

    // 2
    void OnEnable()
    {
        // 3
        GameObject player = GameObject.Find("Player");
        // 4
        playerBehavior = player.GetComponent<PlayerBehavior>();
        // 5
        playerBehavior.playerJump += HandlePlayerJump;
        debug("Jump event subscribed...");
    }

    // 6
    public void HandlePlayerJump()
    {
         debug("Player has jumped...");
    **}**
    // ... No other changes ...
} 
```

让我们来分解一下代码：

1.  创建一个`PlayerBehavior`类型的公共变量

1.  声明`OnEnable()`方法，每当附加了脚本的对象在场景中变为活动状态时都会调用该方法

`OnEnable`是`MonoBehaviour`类中的一个方法，因此所有 Unity 脚本都可以访问它。这是一个很好的地方来放置事件订阅，而不是在`Awake`中执行，因为它只在对象活动时执行，而不仅仅是在加载过程中执行。

1.  在场景中查找`Player`对象并将其`GameObject`存储在一个局部变量中

1.  使用`GetComponent()`检索附加到`Player`的`PlayerBehavior`类的引用，并将其存储在`playerBehavior`变量中

1.  使用`+=`运算符订阅了在`PlayerBehavior`中声明的`playerJump`事件，并使用名为`HandlePlayerJump`的方法

1.  声明`HandlePlayerJump()`方法，其签名与事件的类型匹配，并使用调试委托每次接收到事件时记录成功消息！[](img/B17573_13_08.png)

图 13.9：委托事件订阅的控制台输出

为了正确订阅和接收`GameBehavior`中的事件，我们必须获取到玩家附加的`PlayerBehavior`类的引用。我们本可以一行代码完成所有操作，但将其拆分开来更加可读。然后，我们分配了一个方法给`playerJump`事件，每当接收到事件时都会执行该方法，并完成订阅过程。

现在每次跳跃时，您都会看到带有事件消息的调试消息：

![](img/B17573_13_09.png)

图 13.10：委托事件触发的控制台输出

由于事件订阅是在脚本中配置的，并且脚本附加到 Unity 对象上，我们的工作还没有完成。当对象被销毁或从场景中移除时，我们仍然需要处理如何清理订阅，这将在下一节中介绍。

## 清理事件订阅

即使在我们的原型中，玩家永远不会被销毁，但在游戏中失去玩家是一个常见的特性。清理事件订阅非常重要，因为它们占用了分配的资源，正如我们在*第十二章*“保存、加载和序列化数据”中讨论的流一样。

我们不希望在订阅对象被销毁后仍然保留任何订阅，因此让我们清理一下我们的跳跃事件。在`OnEnable`方法之后，将以下代码添加到`GameBehavior`中：

```cs
// 1
private void OnDisable()
{
    // 2
    playerBehavior.playerJump -= HandlePlayerJump;
    debug("Jump event unsubscribed...");
} 
```

让我们来分解我们的新代码添加：

1.  声明`OnDisable()`方法，它属于`MonoBehavior`类，并且是我们之前使用的`OnEnable()`方法的伴侣

+   您需要编写的任何清理代码通常应该放在这个方法中，因为它在附加了脚本的对象处于非活动状态时执行

1.  使用`-=`运算符取消`HandlePlayerJump`中的`playerJump`事件的订阅，并打印出控制台消息

现在我们的脚本在游戏对象启用和禁用时正确订阅和取消订阅事件，不会在我们的游戏场景中留下未使用的资源。

这就结束了我们对事件的讨论。现在你可以从一个脚本广播它们到游戏的每个角落，并对玩家失去生命、收集物品或更新 UI 等情况做出反应。然而，我们仍然需要讨论一个非常重要的话题，没有它，没有程序能成功，那就是错误处理。

# 处理异常

高效地将错误和异常纳入代码中，是你编程之旅中的专业和个人标杆。在你开始大喊“我花了这么多时间避免错误，为什么要添加错误？！”之前，你应该知道我并不是指添加错误来破坏你现有的代码。相反，包括错误或异常，并在功能部分被错误使用时适当处理它们，会使你的代码库更加强大，更不容易崩溃，而不是更弱。

## 抛出异常

当我们谈论添加错误时，我们将这个过程称为*异常抛出*，这是一个恰当的视觉类比。抛出异常是防御性编程的一部分，这基本上意味着你在代码中积极有意识地防范不当或非计划的操作。为了标记这些情况，你从一个方法中抛出一个异常，然后由调用代码处理。

举个例子：假设我们有一个`if`语句，检查玩家的电子邮件地址是否有效，然后才允许他们注册。如果输入的电子邮件无效，我们希望我们的代码抛出异常：

```cs
public void ValidateEmail(string email)
{
    if(!email.Contains("@"))
    {
        **throw****new** **System.ArgumentException(****"Email is invalid"****);**
    }
} 
```

我们使用`throw`关键字来抛出异常，异常是使用`new`关键字后跟我们指定的异常创建的。`System.ArgumentException()`默认会记录关于异常在何时何地执行的信息，但也可以接受自定义字符串，如果你想更具体。

`ArgumentException`是`Exception`类的子类，并且通过之前显示的`System`类访问。C#带有许多内置的异常类型，包括用于检查空值、超出范围的集合值和无效操作的子类。异常是使用正确的工具来做正确的工作的一个典型例子。我们的例子只需要基本的`ArgumentException`，但你可以在[`docs.microsoft.com/en-us/dotnet/api/system.exception#Standard`](https://docs.microsoft.com/en-us/dotnet/api/system.exception#Standard)找到完整的描述列表。

在我们第一次尝试异常处理时，让事情保持简单，并确保我们只有在提供正的场景索引号时才重新开始关卡：

1.  打开`Utilities`并将以下代码添加到重载版本的`RestartLevel(int)`中：

```cs
public static class Utilities 
{
    // ... No changes needed ...
    public static bool RestartLevel(int sceneIndex) 
    {
        **// 1**
        **if****(sceneIndex <** **0****)**
        **{**
            **// 2**
            **throw****new** **System.ArgumentException(****"Scene index cannot be negative"****);**
         **}**

        Debug.Log("Player deaths: " + PlayerDeaths);
        string message = UpdateDeathCount(ref PlayerDeaths);
        Debug.Log("Player deaths: " + PlayerDeaths);
        Debug.Log(message);

        SceneManager.LoadScene(sceneIndex);
        Time.timeScale = 1.0f;

        return true;
    }
} 
```

1.  在`GameBehavior`中将`RestartLevel()`更改为接受负场景索引并且输掉游戏：

```cs
// 3
public void RestartScene()
{
    Utilities.RestartLevel(**-1**);
} 
```

让我们来分解一下代码：

1.  声明一个`if`语句来检查`sceneIndex`是否不小于 0 或负数

1.  如果传入一个负的场景索引作为参数，抛出一个带有自定义消息的`ArgumentException`

1.  使用场景索引为`-1`调用`RestartLevel()`：![](img/B17573_13_10.png)

图 13.11：抛出异常时的控制台输出

现在当我们输掉游戏时，会调用`RestartLevel()`，但由于我们使用`-1`作为场景索引参数，我们的异常会在任何场景管理逻辑执行之前被触发。我们目前游戏中没有配置其他场景，但这个防御性代码作为保障，不让我们执行可能导致游戏崩溃的操作（Unity 在加载场景时不支持负索引）。

现在你成功地抛出了一个错误，你需要知道如何处理错误的后果，这将引导我们进入下一节和`try-catch`语句。

## 使用 try-catch

现在我们已经抛出了一个错误，我们的工作是安全地处理调用`RestartLevel()`可能产生的可能结果，因为在这一点上，这没有得到适当的处理。要做到这一点，需要使用一种新的语句，称为`try-catch`：

```cs
try
{
    // Call a method that might throw an exception
}
catch (ExceptionType localVariable)
{
    // Catch all exception cases individually
} 
```

`try-catch`语句由连续的代码块组成，这些代码块在不同的条件下执行；它就像一个专门的`if`/`else`语句。我们在`try`块中调用可能引发异常的任何方法——如果没有引发异常，代码将继续执行而不中断。如果引发异常，代码将跳转到与抛出异常匹配的`catch`语句，就像`switch`语句与其 case 一样。`catch`语句需要定义它们要处理的异常，并指定一个本地变量名，该变量将在`catch`块内表示它。

您可以在`try`块之后链接多个`catch`语句，以处理从单个方法抛出的多个异常，只要它们捕获不同的异常。例如：

```cs
try
{
    // Call a method that might throw an exception
}
catch (ArgumentException argException)
{
    // Catch argument exceptions here
}
catch (FileNotFoundException fileException)
{
    // Catch exceptions for files not found here
} 
```

还有一个可选的`finally`块，可以在任何`catch`语句之后声明，无论是否抛出异常，它都将在`try-catch`语句的最后执行：

```cs
finally
{
    // Executes at the end of the try-catch no matter what
} 
```

您的下一个任务是使用`try-catch`语句处理重新启动关卡时抛出的任何错误。现在我们有一个在游戏失败时抛出的异常，让我们安全地处理它。使用以下代码更新`GameBehavior`，然后再次失败游戏：

```cs
public class GameBehavior : MonoBehaviour, IManager
{
    // ... No variable changes needed ...
    public void RestartScene()
    {
        // 1 
        try
        {
            Utilities.RestartLevel(-1);
            debug("Level successfully restarted...");
        }
        // 2
        catch (System.ArgumentException exception)
        {
            // 3
            Utilities.RestartLevel(0);
            debug("Reverting to scene 0: " + exception.ToString());
        }
        // 4
        finally
        {
            debug("Level restart has completed...");
        }
    }
} 
```

让我们分解一下代码：

1.  声明`try`块，并将调用`RestartLevel()`移至其中，并使用`debug`命令打印出重新启动是否完成而没有任何异常。

1.  声明`catch`块，并将`System.ArgumentException`定义为它将处理的异常类型，`exception`作为局部变量名。

1.  如果抛出异常，则在默认场景索引处重新启动游戏：

+   使用`debug`委托打印出自定义消息，以及可以从`exception`访问并使用`ToString()`方法将其转换为字符串的异常信息

由于`exception`是`ArgumentException`类型，因此与`Exception`类关联的有几个属性和方法，您可以访问这些属性和方法。当您需要关于特定异常的详细信息时，这些通常很有用。

1.  添加一个带有调试消息的`finally`块，以表示异常处理代码的结束！[](img/B17573_13_11.png)

图 13.12：完整的 try-catch 语句的控制台输出

当现在调用`RestartLevel()`时，我们的`try`块安全地允许其执行，如果出现错误，则在`catch`块内捕获。`catch`块在默认场景索引处重新启动关卡，代码继续执行到`finally`块，该块只是为我们记录一条消息。

了解如何处理异常很重要，但不应该养成在代码中随处放置异常的习惯。这将导致臃肿的类，并可能影响游戏的处理时间。相反，您应该在最需要的地方使用异常——无效或数据处理，而不是游戏机制。

C#允许您自由创建自己的异常类型，以满足代码可能具有的任何特定需求，但这超出了本书的范围。这只是一个未来要记住的好事情：[`docs.microsoft.com/en-us/dotnet/standard/exceptions/how-to-create-user-defined-exceptions`](https://docs.microsoft.com/en-us/dotnet/standard/exceptions/how-to-create-user-defined-exceptions)。

# 摘要

虽然本章将我们带到了 C#和 Unity 2020 的实际冒险的尽头，但我希望您的游戏编程和软件开发之旅刚刚开始。您已经学会了从创建变量、方法和类对象到编写游戏机制、敌人行为等方方面面的知识。

本章涵盖的主题已经超出了我们在大部分书中处理的水平，这是有充分理由的。你已经知道你的编程大脑是需要锻炼的肌肉，才能进入下一个阶段。泛型、事件和设计模式都只是编程阶梯上的下一个台阶。

在下一章中，我将为你提供资源、进一步阅读以及有关 Unity 社区和软件开发行业的大量其他有用（我敢说，很酷）的机会和信息。

编程愉快！

# 弹出测验-中级 C#

1.  泛型和非泛型类之间有什么区别？

1.  在为委托类型分配值时需要匹配什么？

1.  你如何取消订阅事件？

1.  在你的代码中，你会使用哪个 C#关键字来发送异常？

# 加入我们的 Discord！

与其他用户、Unity/C#专家和 Harrison Ferrone 一起阅读本书。提出问题，为其他读者提供解决方案，通过*问我任何事*与作者交流，以及更多。

立即加入！

[`packt.link/csharpunity2021`](https://packt.link/csharpunity2021)

![](img/QR_Code_9781801813945.png)
