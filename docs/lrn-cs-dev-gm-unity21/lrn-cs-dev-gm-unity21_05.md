# 第五章：使用类、结构和 OOP

出于明显的原因，本书的目标不是让您因信息过载而头痛欲裂。然而，接下来的主题将把您从初学者的小隔间带到**面向对象编程**（**OOP**）的开放空间。到目前为止，我们一直在依赖 C#语言中预定义的变量类型：底层的字符串、列表和字典都是类，这就是为什么我们可以创建它们并通过点表示法使用它们的属性。然而，依赖内置类型有一个明显的弱点——无法偏离 C#已经设定的蓝图。

创建您自己的类使您能够定义和配置设计的蓝图，捕获信息并驱动特定于您的游戏或应用程序的操作。实质上，自定义类和 OOP 是编程王国的关键；没有它们，独特的程序将寥寥无几。

在本章中，您将亲身体验从头开始创建类，并讨论类变量、构造函数和方法的内部工作原理。您还将了解引用类型和值类型对象之间的区别，以及这些概念如何在 Unity 中应用。随着您的学习，以下主题将会更详细地讨论：

+   引入 OOP

+   定义类

+   声明结构

+   理解引用类型和值类型

+   整合面向对象的思维方式

+   在 Unity 中应用 OOP

# 引入 OOP

在 C#编程时，OOP 是您将使用的主要编程范式。如果类和结构实例是我们程序的蓝图，那么 OOP 就是将所有东西都组合在一起的架构。当我们将 OOP 称为编程范式时，我们是说它对整个程序的工作和通信有特定的原则。

实质上，OOP 关注的是对象而不是纯粹的顺序逻辑——它们所持有的数据，它们如何驱动行动，以及最重要的是它们如何相互通信。

# 定义类

回到*第二章*，*编程的基本组成部分*，我们简要讨论了类是对象的蓝图，并提到它们可以被视为自定义变量类型。我们还了解到`LearningCurve`脚本是一个类，但是 Unity 可以将其附加到场景中的对象。关于类的主要事情要记住的是它们是*引用类型*——也就是说，当它们被分配或传递给另一个变量时，引用的是原始对象，而不是一个新的副本。在我们讨论结构之后，我们将深入讨论这一点。然而，在此之前，我们需要了解创建类的基础知识。

现在，我们将暂时搁置 Unity 中类和脚本的工作方式，专注于它们在 C#中是如何创建和使用的。类是使用`class`关键字创建的，如下所示：

```cs
accessModifier class UniqueName
{
    Variables 
    Constructors
    Methods
} 
```

在一个类内声明的任何变量或方法都属于该类，并通过其独特的类名访问。

为了使本章中的示例尽可能连贯，我们将创建和修改一个典型游戏可能拥有的简单`Character`类。我们还将摆脱代码截图，让您习惯于阅读和解释代码，就像在实际环境中看到的那样。然而，我们首先需要自己的自定义类，所以让我们创建一个。

在我们理解它们的内部工作原理之前，我们需要一个类来进行实践，所以让我们创建一个新的 C#脚本，从头开始。

1.  右键单击您在*第一章*中创建的`Scripts`文件夹，然后选择**创建** | **C#脚本**。

1.  将脚本命名为`Character`，在 Visual Studio 中打开它，并删除所有生成的代码。

1.  声明一个名为`Character`的公共类，后面跟着一对花括号，然后保存文件。您的类代码应该与以下代码完全匹配：

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Character
{ 
} 
```

1.  我们删除了生成的代码，因为我们不需要将这个脚本附加到 Unity 游戏对象上。

`Character`现在被注册为一个公共类蓝图。这意味着项目中的任何类都可以使用它来创建角色。然而，这些只是指示——创建角色需要额外的步骤。这个创建步骤被称为*实例化*，并且是下一节的主题。

## 实例化类对象

实例化是根据特定一组指令创建对象的行为，这个对象被称为实例。如果类是蓝图，实例就是根据它们的指令建造的房屋；每个新的`Character`实例都是它的对象，就像根据相同指令建造的两个房屋仍然是两个不同的物理结构一样。一个的变化对另一个没有任何影响。

在*第四章*，*控制流和集合类型*中，我们创建了列表和字典，这些是 C#默认的类，使用它们的类型和`new`关键字。我们可以对自定义类（比如`Character`）做同样的事情，这就是你接下来要做的。

我们将`Character`类声明为公共的，这意味着在任何其他类中都可以创建`Character`实例。由于我们已经在`LearningCurve`中工作了，让我们在`Start()`方法中声明一个新的角色。

在`Start()`方法中打开`LearningCurve`并声明一个名为`hero`的新的`Character`类型变量：

```cs
Character hero = new Character(); 
```

让我们一步一步来分解这个问题：

1.  变量类型被指定为`Character`，这意味着变量是该类的一个实例。

1.  变量名为`hero`，它是使用`new`关键字创建的，后面跟着`Character`类名和两个括号。这是实例在程序内存中创建的地方，即使类现在是空的。

1.  我们可以像处理到目前为止的任何其他对象一样使用`hero`变量。当`Character`类有了自己的变量和方法时，我们可以使用点符号从`hero`中访问它们。

在创建`hero`变量时，你也可以使用推断声明，就像这样：

```cs
var hero = new Character(); 
```

现在我们的角色类没有任何类字段的话就不能做太多事情。在接下来的几节中，你将添加类字段，以及更多内容。

## 添加类字段

向自定义类添加变量或字段与我们在`LearningCurve`中已经做过的事情没有什么不同。相同的概念适用，包括访问修饰符、变量作用域和值分配。然而，属于类的任何变量都是与类实例一起创建的，这意味着如果没有分配值，它们将默认为零或空。一般来说，选择设置初始值取决于它们将存储的信息：

+   如果一个变量在创建类实例时需要具有相同的起始值，设置初始值是一个很好的主意。这对于像经验点或起始分数之类的东西会很有用。

+   如果一个变量需要在每个类实例中进行自定义，比如`CharacterName`，就将其值保持未分配，并使用类构造函数（这是我们将在*使用构造函数*部分讨论的一个主题）。

每个角色类都需要一些基本字段；在接下来的部分中，你需要添加它们。

让我们加入两个变量来保存角色的名称和起始经验点数：

1.  在`Character`类的大括号内添加两个`public`变量——一个用于名称的`string`变量，一个用于经验点的`integer`变量。

1.  将`name`值留空，但将经验点设置为`0`，这样每个角色都从零开始：

```cs
public class Character
{
    public string name;
    public int exp = 0; 
} 
```

1.  在`Character`实例初始化后，在`LearningCurve`中添加一个调试日志。使用它来打印出新角色的`name`和`exp`变量，使用点符号表示法：

```cs
Character hero = new Character(); 
Debug.LogFormat("Hero: {0} - {1} EXP", hero.name, hero.exp); 
```

1.  当`hero`被初始化时，`name`被分配一个空值，在调试日志中显示为空格，而`exp`打印出`0`。请注意，我们不需要将`Character`脚本附加到场景中的任何游戏对象上；我们只是在`LearningCurve`中引用它们，Unity 会完成其余的工作。控制台现在将调试输出我们的角色信息，如下所示：![](img/B17573_05_01.png)

图 5.1：控制台中打印的自定义类属性的屏幕截图

到目前为止，我们的类可以工作，但是使用这些空值并不是很实用。您需要使用所谓的类构造函数来解决这个问题。

## 使用构造函数

类构造函数是特殊的方法，当创建类实例时会自动触发，这类似于`LearningCurve`中`Start`方法的运行方式。构造函数根据其蓝图构建类：

+   如果没有指定构造函数，C#会生成一个默认构造函数。默认构造函数将任何变量设置为它们的默认类型值——数值变量设置为零，布尔变量设置为 false，引用类型（类）设置为 null。

+   自定义构造函数可以带有参数，就像任何其他方法一样，并且用于在初始化时设置类变量的值。

+   一个类可以有多个构造函数。

构造函数的编写方式与常规方法相似，但有一些区别；例如，它们需要是公共的，没有返回类型，方法名始终是类名。例如，让我们向`Character`类添加一个没有参数的基本构造函数，并将名称字段设置为非 null 值。

将这段新代码直接添加到类变量下面，如下所示：

```cs
public string name;
public int exp = 0;
**public****Character****()**
**{**
 **name =** **"Not assigned"****;**
**}** 
```

在 Unity 中运行项目，您将看到`hero`实例使用这个新的构造函数。调试日志将显示英雄的名称为**未分配**，而不是空值：

![](img/B17573_05_02.png)

图 5.2：控制台中打印的未分配的自定义类变量的屏幕截图

这是一个很好的进展，但是我们需要使类构造函数更加灵活。这意味着我们需要能够传入值，以便它们可以作为起始值使用，接下来您将要做的就是这个。

现在，`Character`类开始表现得更像一个真正的对象，但我们可以通过添加第二个构造函数来使其更好，以便在初始化时接受一个名称并将其设置为`name`字段：

1.  在`Character`中添加另一个接受`string`参数的构造函数，称为`name`。

1.  使用`this`关键字将参数分配给类的`name`变量。这被称为*构造函数重载*：

```cs
public Character(string name)
{
    this.name = name;
} 
```

为了方便起见，构造函数通常会具有与类变量同名的参数。在这些情况下，使用`this`关键字指定变量属于类。在这个例子中，`this.name`指的是类的名称变量，而`name`是参数；如果没有`this`关键字，编译器会发出警告，因为它无法区分它们。

1.  在`LearningCurve`中创建一个新的`Character`实例，称为`heroine`。在初始化时使用自定义构造函数传入一个名称，并在控制台中打印出详细信息：

```cs
Character heroine = new Character("Agatha");
Debug.LogFormat("Hero: {0} - {1} EXP", heroine.name,
        heroine.exp); 
```

当一个类有多个构造函数或一个方法有多个变体时，Visual Studio 会在自动完成弹出窗口中显示一组箭头，可以使用箭头键滚动浏览：

![](img/B17573_05_03.png)

图 5.3：Visual Studio 中多个方法构造函数的屏幕截图

1.  现在，我们可以在初始化新的`Character`类时选择基本构造函数或自定义构造函数。`Character`类本身在配置不同情况下的不同实例时现在更加灵活了：![](img/B17573_05_04.png)

图 5.4：控制台中打印的多个自定义类实例的屏幕截图

现在真正的工作开始了；除了作为变量存储设施之外，我们的类还需要方法才能做任何有用的事情。您的下一个任务是将这个付诸实践。

## 声明类方法

将自定义类添加方法与将它们添加到`LearningCurve`没有任何区别。然而，这是一个很好的机会来谈谈良好编程的基本原则——**不要重复自己**（**DRY**）。DRY 是所有良好编写代码的基准。基本上，如果你发现自己一遍又一遍地写同样的代码行，那么是时候重新思考和重新组织了。这通常以创建一个新方法来保存重复的代码形式出现，这样可以更容易地修改和调用该功能，无论是在当前脚本中还是在其他脚本中。

在编程术语中，你会看到这被称为**抽象**出一个方法或特性。

我们已经有了相当多的重复代码，所以让我们看看在哪里可以增加脚本的可读性和效率。

我们重复的调试日志是一个很好的机会，可以将一些代码直接抽象到`Character`类中：

1.  向`Character`类添加一个名为`PrintStatsInfo`的新`public`方法，返回类型为`void`。

1.  将`LearningCurve`中的调试日志复制粘贴到方法体中。

1.  将变量更改为`name`和`exp`，因为现在可以直接从类引用它们。

```cs
public void PrintStatsInfo()
{
      Debug.LogFormat("Hero: {0} - {1} EXP", name, exp);
} 
```

1.  用对`PrintStatsInfo`的方法调用替换我们之前添加到`LearningCurve`中的角色调试日志，然后点击播放：

```cs
 Character hero = new Character();
 **hero.PrintStatsInfo();**
 Character heroine = new Character("Agatha");
 **heroine.PrintStatsInfo();** 
```

1.  现在`Character`类有了一个方法，任何实例都可以使用点表示法自由访问它。由于`hero`和`heroine`都是独立的对象，`PrintStatsInfo`会将它们各自的`name`和`exp`值调试到控制台。

这种行为比直接在`LearningCurve`中拥有调试日志要好。将功能组合到一个类中并通过方法驱动操作总是一个好主意。这样可以使代码更易读——因为我们的`Character`对象在打印调试日志时发出了命令，而不是重复代码。

整个`Character`类应该如下所示的代码：

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Character
{
    public string name;
    public int exp = 0;

    public Character()
    {
        name = "Not assigned";
    }

    public Character(string name)
    {
        this.name = name;
    }

    public void PrintStatsInfo()
    {
        Debug.LogFormat("Hero: {0} - {1} EXP", name, exp);
    }
} 
```

通过对类的讲解，你已经可以写出可读性强、轻量级且可重用的模块化代码了。现在是时候来解决类的表亲对象——结构体了！

# 声明结构体

**结构体**与类似，它们也是你想在程序中创建的对象的蓝图。主要区别在于它们是*值类型*，这意味着它们是按值传递而不是按引用传递的，就像类一样。当结构体被分配或传递给另一个变量时，会创建结构体的一个新副本，因此原始结构体根本没有被引用。我们将在下一节中更详细地讨论这一点。首先，我们需要了解结构体的工作原理以及创建它们时适用的具体规则。

结构体的声明方式与类相同，可以包含字段、方法和构造函数。

```cs
accessModifier struct UniqueName 
{
    Variables
    Constructors
    Methods
} 
```

与类一样，任何变量和方法都属于结构体，并且通过其唯一名称访问。

然而，结构体有一些限制：

+   除非标记为`static`或`const`修饰符，否则不能在结构体声明中使用值初始化变量——你可以在*第十章*，*重新审视类型、方法和类*中了解更多信息。

+   不允许没有参数的构造函数。

+   结构体带有一个默认构造函数，它会自动将所有变量设置为它们的默认值，根据它们的类型。

每个角色都需要一把好武器，这些武器是结构体对象的完美选择。我们将在本章的*理解引用和值类型*部分讨论为什么这样做。然而，首先，你要创建一个结构体来玩耍。

我们的角色需要好的武器来完成任务，这对于一个简单的结构体来说是一个很好的选择：

1.  右键单击`Scripts`文件夹，选择**创建**，然后选择**C#脚本**。

1.  将其命名为`Weapon`，在 Visual Studio 中打开它，然后删除`using UnityEngine`后面生成的所有代码。

1.  声明一个名为`Weapon`的公共结构体，然后保存文件。

1.  添加一个`string`类型的`name`字段和一个`int`类型的`damage`字段：

你可以在彼此嵌套的类和结构中，但这通常是不被赞同的，因为它会使代码变得混乱。

```cs
public struct Weapon
{
    public string name;
    public int damage;
} 
```

1.  使用`name`和`damage`参数声明一个构造函数，并使用`this`关键字设置结构字段：

```cs
public Weapon(string name, int damage)
{
    this.name = name;
    this.damage = damage;
} 
```

1.  在构造函数下面添加一个调试方法来打印武器信息：

```cs
public void PrintWeaponStats()
{
    Debug.LogFormat("Weapon: {0} - {1} DMG", name, damage);
} 
```

1.  在`LearningCurve`中，使用自定义构造函数和`new`关键字创建一个新的`Weapon`结构：

```cs
Weapon huntingBow = new Weapon("Hunting Bow", 105); 
```

1.  我们的新`huntingBow`对象使用了自定义构造函数，并在初始化时为两个字段提供了值。

将脚本限制为单个类是一个好主意，但看到仅由一个类专用的结构体包含在文件中是相当常见的。

现在我们已经有了引用（类）和值（结构）对象的例子，是时候熟悉它们各自的细节了。更具体地说，你需要了解这些对象是如何在内存中传递和存储的。

# 理解引用类型和值类型

除了关键字和初始字段值之外，到目前为止我们还没有看到类和结构之间有太大的区别。类最适合将复杂的操作和数据组合在一起，并且这些数据在程序运行过程中会发生变化；而结构更适合简单的对象和数据，这些数据在大部分时间内都保持不变。除了它们的用途之外，在一个关键领域它们有根本的不同——那就是它们是如何在变量之间传递或赋值的。类是*引用类型*，意味着它们是通过引用传递的；结构是*值类型*，意味着它们是通过值传递的。

## 引用类型

当我们的`Character`类的实例被初始化时，`hero`和`heroine`变量并不持有它们的类信息——相反，它们持有对象在程序内存中的位置的引用。如果我们将`hero`或`heroine`分配给同一类中的另一个变量，那么内存引用就会被分配，而不是角色数据。这有几个影响，其中最重要的是，如果我们有多个变量存储相同的内存引用，对其中一个的更改会影响它们全部。

这样的话题最好是通过演示而不是解释来展示；接下来就由你来在实际例子中尝试一下。

现在是时候测试`Character`类是引用类型了：

1.  在`LearningCurve`中，声明一个名为`hero2`的新`Character`变量。将`hero2`分配给`hero`变量，并使用`PrintStatsInfo`方法打印出两组信息。

1.  点击播放并查看在控制台中显示的两个调试日志：

```cs
Character hero = new Character();
**Character hero2 = hero;**

hero.PrintStatsInfo();
**hero2.PrintStatsInfo();** 
```

1.  两个调试日志将是相同的，因为在创建`hero2`时它被赋值给了`hero`。此时，`hero2`和`hero`都指向内存中`hero`所在的位置！[](img/B17573_05_05.png)

图 5.5：控制台打印的结构体统计信息的屏幕截图

1.  现在，将`hero2`的名字改成有趣的东西，然后再次点击播放：

```cs
Character hero2 = hero;
**hero2.name =** **"Sir Krane the Brave"****;** 
```

1.  你会看到现在`hero`和`hero2`都有相同的名字，即使只有一个角色的数据被改变了！[](img/B17573_05_06.png)

图 5.6：控制台打印的类实例属性的屏幕截图

这里的教训是，引用类型需要小心处理，不要在分配给新变量时进行复制。对一个引用的任何更改都会影响所有持有相同引用的其他变量。

如果你想复制一个类，要么创建一个新的独立实例，要么重新考虑是否结构体可能是你对象蓝图的更好选择。在接下来的部分中，你将更好地了解值类型。

## 值类型

当创建一个结构对象时，它的所有数据都存储在相应的变量中，没有引用或连接到它的内存位置。这使得结构对于创建需要快速高效地复制的对象非常有用，同时仍保留它们各自的身份。在接下来的练习中，尝试使用我们的`Weapon`结构来实现这一点。

让我们通过将`huntingBow`复制到一个新变量中并更新其数据来创建一个新的武器对象，以查看更改是否影响两个结构体：

1.  在`LearningCurve`中声明一个新的`Weapon`结构，并将`huntingBow`分配为其初始值：

```cs
Weapon huntingBow = new Weapon("Hunting Bow", 105);
**Weapon warBow = huntingBow;** 
```

1.  使用调试方法打印出每个武器的数据：

```cs
**huntingBow.PrintWeaponStats();**
**warBow.PrintWeaponStats();** 
```

1.  现在它们的设置方式是，`huntingBow`和`warBow`将有相同的调试日志，就像我们在改变任何数据之前的两个角色一样！

图 5.7：控制台中打印的结构体实例的屏幕截图

1.  将`warBow.name`和`warBow.damage`字段更改为你选择的值，然后再次点击播放：

```cs
 Weapon warBow = huntingBow;
 **warBow.name =** **"War Bow"****;**
 **warBow.damage =** **155****;** 
```

1.  控制台将显示只有与`warBow`相关的数据被更改，而`huntingBow`保留其原始数据。

图 5.8：打印到控制台的更新后的结构体属性的屏幕截图

从这个例子中可以得出的结论是，结构体很容易被复制和修改为它们各自的对象，而类则保留对原始对象的引用。现在我们对结构体和类在底层是如何工作有了一些了解，并确认了引用和值类型在它们的自然环境中的行为，我们可以开始谈论编程中最重要的一个主题，OOP，以及它如何适应编程领域。

# 整合面向对象的思维方式

物理世界中的事物在 OOP 的类似级别上运行；当你想要买一罐软饮料时，你拿的是一罐苏打水，而不是液体本身。这个罐子是一个对象，将相关信息和动作组合在一个自包含的包中。然而，在处理对象时有一些规则，无论是在编程中还是在杂货店——例如，谁可以访问它们。不同的变化和通用的动作都影响着我们周围所有对象的性质。

在编程术语中，这些规则是 OOP 的主要原则：*封装*、*继承*和*多态*。

## 封装

OOP 最好的一点是它支持封装——定义对象的变量和方法对外部代码的可访问性（有时被称为*调用代码*）。以我们的苏打罐为例——在自动售货机中，可能的互动是有限的。由于机器被锁住，不是每个人都可以过来拿一罐；如果你碰巧有合适的零钱，你将被允许有条件地访问它，但数量是有限制的。如果机器本身被锁在一个房间里，只有拿着门钥匙的人才会知道苏打罐的存在。

你现在要问自己的问题是，我们如何设置这些限制？简单的答案是，我们一直在使用封装，通过为我们的对象变量和方法指定访问修饰符。

如果你需要复习，请回到*第三章*，*深入变量、类型和方法*中的*访问修饰符*部分。

让我们尝试一个简单的封装示例，以了解这在实践中是如何工作的。我们的`Character`类是公共的，它的字段和方法也是公共的。但是，如果我们想要一个方法来将角色的数据重置为其初始值，会怎样呢？这可能会很方便，但如果意外调用了它，可能会造成灾难，这就是一个私有对象成员的完美候选者：

1.  在`Character`类中创建一个名为`Reset`的`private`方法，没有返回值。将`name`和`exp`变量分别设置为`"Not assigned"`和`0`：

```cs
private void Reset()
{
    this.name = "Not assigned";
    this.exp = 0;
} 
```

1.  尝试在打印出`hero2`数据后从`LearningCurve`中调用`Reset()`！

图 5.9：Character 类中一个无法访问的方法的屏幕截图

如果你想知道 Visual Studio 是否出了问题，它没有。将方法或变量标记为私有将使其在这个类或结构体内部使用点表示法时无法访问；如果你手动输入并悬停在`Reset()`上，你会看到有关该方法受保护的错误消息。

要实际调用这个私有方法，我们可以在类构造函数中添加一个重置命令：

```cs
public Character()
{
    Reset();
} 
```

封装确实允许对象具有更复杂的可访问性设置；然而，现在我们将坚持使用 `public` 和 `private` 成员。在下一章中，当我们开始完善游戏原型时，我们将根据需要添加不同的修饰符。

现在，让我们谈谈继承，在创建未来游戏中的类层次结构时，它将成为您的好朋友。

## 继承

C# 类可以按照另一个类的形象创建，共享其成员变量和方法，但能够定义其独特的数据。在面向对象编程中，我们将这称为*继承*，这是一种创建相关类的强大方式，而无需重复代码。再次以汽水为例，市场上有通用汽水，它们具有相同的基本属性，还有特殊汽水。特殊汽水共享相同的基本属性，但具有不同的品牌或包装，使它们与众不同。当您将两者并排看时，很明显它们都是汽水罐，但它们显然不是同一种。

原始类通常称为基类或父类，而继承类称为派生类或子类。任何使用 `public`、`protected` 或 `internal` 访问修饰符标记的基类成员都自动成为派生类的一部分，除了构造函数。类构造函数始终属于其包含类，但可以从派生类中使用，以将重复的代码最小化。现在不要太担心不同的基类情况。相反，让我们尝试一个简单的游戏示例。

大多数游戏都有多种类型的角色，因此让我们创建一个名为 `Paladin` 的新类，该类继承自 `Character` 类。您可以将这个新类添加到 `Character` 脚本中，也可以创建一个新的脚本。如果要将新类添加到 `Character` 脚本中，请确保它在 `Character` 类的花括号之外：

```cs
public class Paladin: Character
{
} 
```

就像 `LearningCurve` 从 `MonoBehavior` 继承一样，我们只需要添加一个冒号和我们想要继承的基类，C# 就会完成剩下的工作。现在，任何 `Paladin` 实例都可以访问 `name` 属性和 `exp` 属性，以及 `PrintStatsInfo` 方法。

通常最好为不同的类创建新的脚本，而不是将它们添加到现有的脚本中。这样可以分离脚本，并避免在任何单个文件中有太多行代码（称为臃肿文件）。

这很好，但继承类如何处理它们的构造？您可以在下一节中找到答案。

### 基础构造函数

当一个类从另一个类继承时，它们形成一种金字塔结构，成员变量从父类流向任何派生子类。父类不知道任何子类，但所有子类都知道它们的父类。然而，父类构造函数可以直接从子类构造函数中调用，只需进行简单的语法修改：

```cs
public class ChildClass: ParentClass
{
    public ChildClass(): **base****()**
    {
    }
} 
```

`base` 关键字代表父构造函数，这种情况下是默认构造函数。然而，由于 `base` 代表一个构造函数，构造函数是一个方法，子类可以将参数传递给其父类构造函数。

由于我们希望所有 `Paladin` 对象都有一个名称，而 `Character` 已经有一个处理这个问题的构造函数，我们可以直接从 `Paladin` 类调用 `base` 构造函数，而不必重写构造函数：

1.  为 `Paladin` 类添加一个构造函数，该构造函数接受一个 `string` 参数，称为 `name`。使用 `colon` 和 `base` 关键字调用父构造函数，传入 `name`：

```cs
public class Paladin: Character
{
**public****Paladin****(****string** **name****):** **base****(****name****)**
 **{**

 **}**
} 
```

1.  在 `LearningCurve` 中，创建一个名为 `knight` 的新 `Paladin` 实例。使用基础构造函数来分配一个值。从 `knight` 调用 `PrintStatsInfo`，并查看控制台：

```cs
Paladin knight = new Paladin("Sir Arthur");
knight.PrintStatsInfo(); 
```

1.  调试日志将与我们的其他`Character`实例相同，但名称将与我们分配给`Paladin`构造函数的名称相同：![](img/B17573_05_10.png)

图 5.10：基本角色构造函数属性的屏幕截图

当`Paladin`构造函数触发时，它将`name`参数传递给`Character`构造函数，从而设置`name`值。基本上，我们使用`Character`构造函数来为`Paladin`类做初始化工作，使`Paladin`构造函数只负责初始化其独特的属性，而在这一点上它还没有。

除了继承，有时你会想要根据其他现有对象的组合来创建新对象。想想乐高积木；你不是从零开始建造——你已经有了不同颜色和结构的积木块可以使用。在编程术语中，这被称为“组合”，我们将在下一节讨论。

## 构成

除了继承，类还可以由其他类组成。以我们的`Weapon`结构为例，`Paladin`可以在自身内部轻松包含一个`Weapon`变量，并且可以访问其所有属性和方法。让我们通过更新`Paladin`来接受一个起始武器并在构造函数中分配其值：

```cs
public class Paladin: Character
{
   **public** **Weapon weapon;**

    public Paladin(string name, **Weapon weapon**): base(name)
    {
        **this****.weapon = weapon;**
    }
} 
```

由于`weapon`是`Paladin`独有的，而不是`Character`的，我们需要在构造函数中设置其初始值。我们还需要更新`knight`实例以包含一个`Weapon`变量。所以，让我们使用`huntingBow`：

```cs
Paladin knight = new Paladin("Sir Arthur", **huntingBow**); 
```

如果现在运行游戏，你不会看到任何不同，因为我们使用的是`Character`类的`PrintStatsInfo`方法，它不知道`Paladin`类的`weapon`属性。为了解决这个问题，我们需要谈谈多态性。

## 多态性

多态是希腊词“多形”的意思，在面向对象编程中有两种不同的应用方式：

+   派生类对象被视为与父类对象相同。例如，一个`Character`对象数组也可以存储`Paladin`对象，因为它们是从`Character`派生而来的。

+   父类可以将方法标记为`virtual`，这意味着它们的指令可以被派生类使用`override`关键字修改。在`Character`和`Paladin`的情况下，如果我们可以为每个类从`PrintStatsInfo`中调试不同的消息，那将是有用的。

多态性允许派生类保留其父类的结构，同时也可以自由地调整动作以适应其特定需求。你标记为`virtual`的任何方法都将给你对象多态性的自由。让我们利用这个新知识并将其应用到我们的角色调试方法中。

让我们修改`Character`和`Paladin`以使用`PrintStatsInfo`打印不同的调试日志：

1.  在`Character`类中通过在`public`和`void`之间添加`virtual`关键字来更改`PrintStatsInfo`：

```cs
public **virtual** void PrintStatsInfo()
{
    Debug.LogFormat("Hero: {0} - {1} EXP", name, exp);
} 
```

1.  使用`override`关键字在`Paladin`类中声明`PrintStatsInfo`方法。添加一个调试日志，以你喜欢的方式打印`Paladin`属性：

```cs
public override void PrintStatsInfo()
{
    Debug.LogFormat("Hail {0} - take up your {1}!", name, 
             weapon.name);
} 
```

这可能看起来像重复的代码，我们已经说过这是不好的形式，但这是一个特殊情况。通过在`Character`类中将`PrintStatsInfo`标记为`virtual`，我们告诉编译器这个方法可以根据调用类的不同而有不同的形式。

1.  当我们在`Paladin`中声明了`PrintStatsInfo`的重写版本时，我们添加了仅适用于该类的自定义行为。多亏了多态性，我们不必选择从`Character`或`Paladin`对象调用哪个版本的`PrintStatsInfo`——编译器已经知道了：![](img/B17573_05_11.png)

图 5.11：多态角色属性的屏幕截图

我知道这是很多内容，所以让我们在接近终点时回顾一些面向对象编程的主要要点：

+   面向对象编程是将相关数据和操作分组到对象中——这些对象可以相互通信并独立行动。

+   可以使用访问修饰符来设置对类成员的访问，就像变量一样。

+   类可以继承自其他类，创建父/子关系的层级结构。

+   类可以拥有其他类或结构类型的成员。

+   类可以覆盖任何标记为`virtual`的父方法，允许它们执行自定义操作同时保留相同的蓝图。

OOP 并不是 C#唯一可用的编程范式，你可以在这里找到其他主要方法的实际解释：[`cs.lmu.edu/~ray/notes/paradigms`](http://cs.lmu.edu/~ray/notes/paradigms)。

在本章学到的所有 OOP 都直接适用于 C#世界。然而，我们仍需要将其与 Unity 放在适当的位置，这将是你在本章剩余时间里专注的内容。

# 在 Unity 中应用 OOP

如果你在 OOP 语言中待得足够长，你最终会听到像*一切都是对象*这样的短语在开发者之间像秘密祈祷一样被低声诉说。遵循 OOP 原则，程序中的一切都应该是一个对象，但 Unity 中的 GameObject 可以代表你的类和结构。然而，并不是说 Unity 中的所有对象都必须在物理场景中，所以我们仍然可以在幕后使用我们新发现的编程类。

## 对象是一个优秀的类

回到*第二章*，*编程的基本组成部分*，我们讨论了当脚本添加到 Unity 中的 GameObject 时，脚本会被转换为组件。从 OOP 的组合原则来看，GameObject 是父容器，它们可以由多个组件组成。这可能与每个脚本一个 C#类的想法相矛盾，但事实上，这更多是为了更好的可读性而不是实际要求。类可以嵌套在彼此内部——只是会变得很混乱。然而，将多个脚本组件附加到单个 GameObject 上可能非常有用，特别是在处理管理类或行为时。

总是试图将对象简化为它们最基本的元素，然后使用组合来构建更大、更复杂的对象。这样做比修改由大型笨重组成的 GameObject 更容易，因为 GameObject 由小型、可互换的组件组成。

让我们来看看**Main Camera**，看看它是如何运作的：

![](img/B17573_05_12.png)

图 5.12：检视器中主摄像机对象的屏幕截图

在前面的屏幕截图中，每个组件（**Transform**、**Camera**、**Audio Listener**和**Learning Curve**脚本）最初都是 Unity 中的一个类。就像`Character`或`Weapon`的实例一样，当我们点击播放时，这些组件在计算机内存中变成对象，包括它们的成员变量和方法。

如果我们将`LearningCurve`（或任何脚本或组件）附加到 1,000 个 GameObject 上并点击播放，将创建并存储 1,000 个单独的`LearningCurve`实例。

我们甚至可以使用组件名称作为数据类型来创建这些组件的实例。与类一样，Unity 组件类是引用类型，可以像其他变量一样创建。然而，查找和分配这些 Unity 组件与你迄今为止所见到的略有不同。为此，你需要在下一节更多地了解 GameObject 的工作原理。

## 访问组件

现在我们知道了组件如何作用于 GameObject，那么我们如何访问它们的特定实例呢？幸运的是，Unity 中的所有 GameObject 都继承自`GameObject`类，这意味着我们可以使用它们的成员方法来在场景中找到我们需要的任何东西。有两种方法可以分配或检索当前场景中活动的 GameObject：

1.  通过`GameObject`类中的`GetComponent()`或`Find()`方法，这些方法可以使用公共和私有变量。

1.  通过将游戏对象直接从**Project**面板拖放到**Inspector**选项卡中的变量槽中。这个选项只适用于 C#中的公共变量，因为它们是唯一会出现在检查器中的变量。如果您决定需要在检查器中显示一个私有变量，可以使用`SerializeField`属性进行标记。

您可以在 Unity 文档中了解有关属性和`SerializeField`的更多信息：[`docs.unity3d.com/ScriptReference/SerializeField.html`](https://docs.unity3d.com/ScriptReference/SerializeField.html)。

让我们来看看第一个选项的语法。

### 在代码中访问组件

使用`GetComponent`相当简单，但它的方法签名与我们迄今为止看到的其他方法略有不同：

```cs
GameObject.GetComponent<ComponentType>(); 
```

我们只需要寻找的组件类型，`GameObject`类将返回该组件（如果存在）和`null`（如果不存在）。`GetComponent`方法还有其他变体，但这是最简单的，因为我们不需要了解我们要查找的`GameObject`类的具体信息。这被称为`通用`方法，我们将在*第十三章*“探索泛型、委托和更多内容”中进一步讨论。然而，现在让我们只使用摄像机的变换。

由于`LearningCurve`已经附加到**Main Camera**对象上，让我们获取摄像机的`Transform`组件并将其存储在一个公共变量中。`Transform`组件控制 Unity 中对象的位置、旋转和缩放，因此这是一个很好的例子：

1.  在`LearningCurve`中添加一个新的公共`Transform`类型变量，称为`CamTransform`：

```cs
public Transform CamTransform; 
```

1.  在`Start`中使用`GetComponent`方法从`GameObject`类初始化`CamTransform`。使用`this`关键字，因为`LearningCurve`附加到与`Transform`组件相同的`GameObject`组件上。

1.  使用点表示法访问和调试`CamTransform`的`localPosition`属性：

```cs
void Start()
{
    CamTransform = this.GetComponent<Transform>();
    Debug.Log(CamTransform.localPosition); 
} 
```

1.  我们在`LearningCurve`的顶部添加了一个未初始化的`public Transform`变量，并在`Start`中使用`GetComponent`方法进行了初始化。`GetComponent`找到了附加到此`GameObject`组件的`Transform`组件，并将其返回给`CamTransform`。现在`CamTransform`存储了一个`Transform`对象，我们可以访问它的所有类属性和方法，包括以下屏幕截图中的`localPosition`！[](img/B17573_05_13.png)

图 5.13：控制台中打印的 Transform 位置的屏幕截图

`GetComponent`方法非常适用于快速检索组件，但它只能访问调用脚本所附加到的游戏对象上的组件。例如，如果我们从附加到**Main Camera**的`LearningCurve`脚本中使用`GetComponent`，我们只能访问**Transform**、**Camera**和**Audio Listener**组件。

如果我们想引用另一个游戏对象上的组件，比如**Directional Light**，我们需要先使用`Find`方法获取对该对象的引用。只需要游戏对象的名称，Unity 就会返回适当的游戏对象供我们存储或操作。

要参考每个游戏对象的名称，可以在所选对象的**Inspector**选项卡顶部找到：

![](img/B17573_05_14.png)

图 5.14：检查器中的 Directional Light 对象的屏幕截图

在 Unity 中找到游戏场景中的对象是至关重要的，因此您需要进行练习。让我们拿到手头的对象并练习查找和分配它们的组件。

让我们尝试一下`Find`方法，并从`LearningCurve`中检索**Directional Light**对象：

1.  在`CamTransform`下面的`LearningCurve`中添加两个变量，一个是`GameObject`类型，一个是`Transform`类型：

```cs
public GameObject DirectionLight;
public Transform LightTransform; 
```

1.  通过名称找到`Directional Light`组件，并在`Start()`方法中用它初始化`DirectionLight`：

```cs
void Start()
{
    DirectionLight = GameObject.Find("Directional Light"); 
} 
```

1.  将`LightTransform`的值设置为附加到`DirectionLight`的`Transform`组件，并调试其`localPosition`。由于`DirectionLight`现在是它的`GameObject`，`GetComponent`完美地工作：

```cs
LightTransform = DirectionLight.GetComponent<Transform>();
Debug.Log(LightTransform.localPosition); 
```

1.  在运行游戏之前，重要的是要理解方法调用可以链接在一起，以减少代码步骤。例如，我们可以通过组合`Find`和`GetComponent`来在一行中初始化`LightTransform`，而不必经过`DirectionLight`：

```cs
GameObject.Find("Directional Light").GetComponent<Transform>(); 
```

警告：长串链接的代码会导致可读性差和混乱，特别是在处理复杂应用程序时。避免超过这个示例的长行是个好的经验法则。

虽然在代码中查找对象总是有效的，但你也可以简单地将对象本身拖放到**Inspector**选项卡中。让我们在下一节中演示如何做到这一点。

### 拖放

既然我们已经介绍了代码密集的做事方式，让我们快速看一下 Unity 的拖放功能。虽然拖放比在代码中使用`GameObject`类要快得多，但 Unity 有时会在保存或导出项目时，或者在 Unity 更新时，丢失通过这种方式建立的对象和变量之间的连接。

当你需要快速分配几个变量时，尽管利用这个功能。但大多数情况下，我建议坚持编码。

让我们改变`LearningCurve`来展示如何使用拖放来分配一个`GameObject`组件：

1.  注释掉下面的代码行，我们使用`GameObject.Find()`来检索并将`Directional Light`对象分配给`DirectionLight`变量：

```cs
//DirectionLight = GameObject.Find("Directional Light"); 
```

1.  选择**Main Camera** GameObject，将**Directional Light**拖到**Learning Curve**组件的`Direction Light`字段中，然后点击播放：![](img/B17573_05_15.png)

图 5.15：将 Directional Light 拖到脚本属性的截图

1.  **Directional Light** GameObject 现在分配给了`DirectionLight`变量。没有涉及任何代码，因为 Unity 在内部分配了变量，而`LearningCurve`类没有发生变化。

在决定是使用拖放还是`GameObject.Find()`来分配变量时，理解一些重要的事情是很重要的。首先，`Find()`方法速度较慢，如果在多个脚本中多次调用该方法，可能会导致性能问题。其次，你需要确保你的 GameObject 在场景层次结构中都有唯一的名称；如果没有，可能会在有多个相同名称的对象或更改对象名称本身的情况下导致一些严重的错误。

# 总结

我们对类、结构和面向对象编程的探索标志着 C#基础知识的第一部分的结束。你已经学会了如何声明你的类和结构，这是你将来制作的每个应用程序或游戏的支架。你还确定了这两种对象在如何传递和访问上的差异，以及它们与面向对象编程的关系。最后，你通过继承、组合和多态来实现面向对象编程的原则。

识别相关的数据和操作，创建蓝图来赋予它们形状，并使用实例来构建交互，这是处理任何程序或游戏的坚实基础。再加上访问组件的能力，你就成为了一个 Unity 开发者的基础。

下一章将过渡到游戏开发的基础知识，并直接在 Unity 中编写对象行为脚本。我们将从详细说明简单的开放世界冒险游戏的要求开始，在场景中使用 GameObject，并最终完成一个为我们的角色准备好的白盒环境。

# 小测验 - 面向对象编程的一切

1.  哪个方法处理类内的初始化逻辑？

1.  作为值类型，结构是如何传递的？

1.  面向对象编程的主要原则是什么？

1.  你会使用哪个`GameObject`类方法来在调用类的同一对象上找到一个组件？

# 加入我们的 Discord！

与其他用户一起阅读本书，与 Unity/C#专家和 Harrison Ferrone 一起阅读。提出问题，为其他读者提供解决方案，通过*问我任何事*与作者交流，等等。

立即加入！

[`packt.link/csharpunity2021`](https://packt.link/csharpunity2021)

![](img/QR_Code_9781801813945.png)
