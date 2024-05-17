# 第五章：C#中的面向对象编程

在上一章中，我们介绍了用户定义类型，并学习了类、结构和枚举。在本章中，我们将学习**面向对象编程**（简称**OOP**）。对 OOP 概念的深入理解对使用 C#编写更好的程序至关重要。面向对象编程可以减少代码复杂性，增加代码可重用性，并使软件易于维护和扩展。

我们将详细介绍以下概念：

+   理解面向对象编程

+   抽象

+   封装

+   继承

+   多态

+   SOLID 原则

通过本章的学习，您将学会如何使用面向对象编程创建类和方法。让我们从面向对象编程的概述开始。

# 理解面向对象编程

面向对象编程是一种允许我们围绕对象编写程序的范式。正如前一章中讨论的，对象包含数据和用于操作该数据的方法。每个对象都有自己的一组数据和方法。如果一个对象想要访问另一个对象的数据，它必须通过该对象中定义的方法进行访问。一个对象可以使用**继承**的概念继承另一个对象的属性。因此，我们可以说面向对象编程是围绕数据和允许对数据进行的操作组织起来的。

C#是一种通用多范式编程语言。面向对象编程只是其中的一种范式。其他支持的范式，如泛型和函数式编程，将在后续章节中讨论。

在讨论面向对象编程时，了解类和对象之间的区别是很重要的。如前所述，在上一章中，类是定义数据及其在内存中的表示以及操作这些数据的功能的蓝图。另一方面，对象是根据蓝图构建和运行的类的实例。它在内存中有一个物理表示，而类只存在于源代码中。

在进行面向对象编程时，您首先要确定需要操作的实体，它们之间的关系以及它们如何相互作用。这个过程称为**数据建模**。其结果是一组概括了确定实体的类。这些实体可以是从物理实体（人、物体、机器等）到抽象实体（订单、待办事项列表、连接字符串等）的各种形式。

抽象、封装、多态和继承是面向对象编程的核心原则。我们将在本章中详细探讨它们。

# 抽象

抽象是通过去除非必要的特征来以简单的方式描述实体和过程的过程。一个物理或抽象实体可能有许多特征，但是对于某些应用程序或领域的目的，并不是所有特征都是重要的。通过将实体抽象为简单模型（对应应用程序领域有意义的模型），我们可以构建更简单、更高效的程序。

让我们以员工为例。员工是一个人。一个人有姓名、生日、身体特征（如身高、体重、头发颜色、眼睛颜色）、亲戚和朋友、喜欢的爱好（如食物、书籍、电影、运动）、地址、财产（如房屋或公寓、汽车或自行车）、一个或多个电话号码和电子邮件地址，以及我们可以列出的许多其他事物。

根据我们构建的应用程序的类型，其中一些特征是相关的，而另一些则不是。例如，如果我们构建一个工资系统，我们对员工的姓名、生日、地址、电话和电子邮件感兴趣，以及入职日期、部门、职位、工资等。如果我们构建一个社交媒体应用程序，我们对用户的姓名、生日、地址、亲戚、朋友、兴趣、活动等感兴趣。

有时需要不同级别的抽象 - 一些更一般的，另一些更具体的。例如，如果我们构建一个可以绘制形状的图形系统，我们可能需要用少量功能模拟一个通用形状，比如能够自己绘制或变换（平移和旋转）的能力。然后我们可以有二维形状和三维形状，每个都有更具体的属性和功能，基于这些形状的特征。

我们可以将线条、椭圆和多边形构建为二维形状。一条线有诸如起点和终点之类的属性，但一个椭圆有两个焦点，以及一个面积和周长。三维对象，如立方体，可以投下阴影。虽然我们仍在抽象概念，但我们已经从更一般的抽象转向更具体的抽象。当这些抽象是基于彼此的时，实现它们的典型方式是通过继承。

# 封装

封装被定义为将数据和操作数据的代码绑定在一个单元中。数据被私下绑定在一个类中，外部无法直接访问。所有需要读取或修改对象数据的对象都应该通过类提供的公共方法来进行。这种特性被称为**数据隐藏**，通过定义有限数量的对象数据入口点，使代码更不容易出错。

让我们来看看这里的`Employee`类：

```cs
public class Employee
{
    private string name;
    private double salary;
    public string Name
    {
        get { return name; }
        set { name = value; }
    }
    public double Salary
    {
        get { return salary; }
    }
    public Employee(string name, double salary)
    {
        this.name = name;
        this.salary = salary;
    }
    public void GiveRaise(double percent)
    {
        salary *= (1.0 + percent / 100.0);
    }
}
```

这里模拟了员工有两个属性：`name`和`salary`。这些被实现为`private`类字段，这使它们只能从`Employee`类内部访问。这两个值在构造函数中设置。名字通过名为`Name`的属性暴露出来以供读取和写入。然而，`salary`变量只暴露出来供读取，使用名为`Salary`的只读属性。要更改工资，我们必须调用`GiveRaise()`方法。当然，这只是一种可能的实现。我们可以使用自动实现的属性而不是字段，或者可能使用不同的其他方法来修改工资。这个类可以如下使用：

```cs
Employee employee = new Employee("John Doe", 2500);
Console.WriteLine($"{employee.Name} earns {employee.Salary}");
employee.GiveRaise(5.5);
Console.WriteLine($"{employee.Name} earns {employee.Salary}");
```

我们已经创建了`Employee`类的对象，并使用构造函数将值设置为私有字段。`Employee`类不允许直接访问其字段。要读取它们的值，我们使用公共属性的`get`访问器。要更改工资，我们使用`GiveRaise()`方法。该程序的输出如下所示：

![图 5.1 - 从前面片段的执行中得到的控制台输出](img/Figure_5.1_B12346.jpg)

图 5.1 - 从前面片段的执行中得到的控制台输出

封装允许我们将类内部的数据隐藏起来，这就是为什么它也被称为数据隐藏。

封装很重要，因为它通过为不同组件定义最小的公共接口来减少它们之间的依赖关系。它还增加了代码的可重用性和安全性，并使代码更容易进行单元测试。

# 继承

继承是一种机制，通过它一个类可以继承另一个类的属性和功能。如果我们有一组多个类共享的常见功能和数据，我们可以将它们放在一个称为**父**或**基**类的类中。其他类可以继承父类的这些功能和数据，同时扩展或修改它们并添加额外的功能和属性。从另一个类继承的类称为**子**或**派生**类。因此，继承有助于*代码重用*。

在 C#中，继承仅支持引用类型。只有定义为类的类型才能从其他类型派生。定义为结构的类型是值类型，不能从其他类型派生。但是，在 C#中，所有类型都是值类型或引用类型，并且间接从`System.Object`类型派生。这种关系是隐式的，不需要开发人员做任何特殊处理。

C#支持三种类型的继承：

+   单一继承：当一个类从一个父类继承时。子类不应该作为任何其他类的父类。参考下图，类 B 从类 A 继承：

![图 5.2 - 类图显示类 B 从类 A 继承](img/Figure_5.2_B12346.jpg)

图 5.2 - 类图显示类 B 从类 A 继承

+   多级继承：这实际上是对前一种情况的扩展，因为子类又是另一个类的父类。在下图中，类 B 既是类 A 的子类，也是类 C 的父类：

![图 5.3 - 类图显示类 A 作为类 B 的基类，而类 B 又是类 C 的基类](img/Figure_5.3_B12346.jpg)

图 5.3 - 类图显示类 A 作为类 B 的基类，而类 B 又是类 C 的基类

+   分层继承：一个类作为多个类的父类。参考下图。这里，类 B 和类 C 都继承自同一个父类 A：

![图 5.4 - 类图显示类 B 和类 C 从基类 A 继承](img/Figure_5.4_B12346.jpg)

图 5.4 - 类图显示类 B 和类 C 从基类 A 继承

与其他编程语言（如 C++）不同，C#不支持多重继承。这意味着一个类不能从多个类派生。

要理解继承，让我们考虑以下示例：我们正在构建一个必须表示地形、障碍、人物、机械等对象的游戏。这些是具有不同属性的各种类型的对象。例如，人和机器可以移动和战斗，障碍可以被摧毁，地形可以是可穿越的或不可穿越的，等等。然而，所有这些游戏对象都有一些共同的属性：它们都在游戏中有一个位置，并且它们都可以在表面上绘制（可以是屏幕、内存等）。我们可以表示一个提供这些功能的基类如下：

```cs
class GameUnit
{
    public Position Position { get; protected set; }
    public GameUnit(Position position)
    {
        Position = position;
    }
    public void Draw(Surface surface)
    {
        surface.DrawAt(GetImage(), Position);
    }
    protected virtual char GetImage() { return ' '; }
}
```

`GameUnit`是一个具有名为`Position`的属性的类；访问器`get`是公共的，但访问器`set`是受保护的，这意味着它只能从这个类或它的派生类中访问。`Draw()`公共方法在当前单位位置在表面上绘制单位。`GetImage()`是一个虚方法，返回单位的表示（在我们的例子中是一个单一字符）。在基类中，这只返回一个空格，但在派生类中，这将被实现为返回一个实际字符。

这里看到的`Position`和`Surface`类的实现如下：

```cs
struct Position
{
    public int X { get; private set; }
    public int Y { get; private set; }
    public Position(int x = 0, int y = 0)
    {
        X = x;
        Y = y;
    }
}
class Surface
{
    private int left;
    private int top;
    public void BeginDraw()
    {
        Console.Clear();
        left = Console.CursorLeft;
        top = Console.CursorTop;
    }
    public void DrawAt(char c, Position position)
    {
        try
        {
            Console.SetCursorPosition(left + position.X, 
                                      top + position.Y);
            Console.Write(c);
        }
        catch (ArgumentOutOfRangeException e)
        {
            Console.Clear();
            Console.WriteLine(e.Message);
        }
    }
}
```

现在，我们将从基类派生出几个其他类。为了简单起见，我们将暂时专注于地形对象：

```cs
class Terrain : GameUnit
{
    public Terrain(Position position) : base(position) { }
}
class Water : Terrain
{
    public Water(Position position) : base(position) { }
    protected override char GetImage() { return '░'; }
}
class Hill : Terrain
{
    public Hill(Position position) : base(position) { }
    protected override char GetImage() { return '≡'; }
}
```

我们在这里定义了一个从`GameUnit`派生的`Terrain`类，它本身是所有类型地形的基类。在这个类中我们没有太多东西，但在一个真实的应用程序中，会有各种功能。`Water`和`Hill`是从`Terrain`派生的两个类，它们覆盖了`GetImage()`类，返回一个不同的字符来表示地形。我们可以使用这些来构建一个游戏：

```cs
var objects = new List<v1.GameUnit>()
{
    new v1.Water(new Position(3, 2)),
    new v1.Water(new Position(4, 2)),
    new v1.Water(new Position(5, 2)),
    new v1.Hill(new Position(3, 1)),
    new v1.Hill(new Position(5, 3)),
};
var surface = new v1.Surface();
surface.BeginDraw();
foreach (var unit in objects)
    unit.Draw(surface);
```

该程序的输出如下截图所示：

![图 5.5 - 前一个程序执行的控制台输出](img/Figure_5.5_B12346.jpg)

图 5.5 - 前一个程序执行的控制台输出

## 虚成员

在前面的例子中，我们已经看到了一个虚方法。这是一个在基类中有实现但可以在派生类中被重写的方法，这对于改变实现细节很有帮助。方法默认情况下是非虚的。基类中的虚方法使用`virtual`关键字声明。派生类中虚方法的重写实现使用`override`关键字定义，而不是`virtual`关键字。虚方法和重写方法的方法签名必须匹配。

方法不是唯一可以是虚的类成员。`virtual`关键字可以应用于属性、索引器和事件。但是`virtual`修饰符不能与`static`、`abstract`、`private`或`override`修饰符一起使用。

在派生类中重写的虚成员可以在派生类的派生类中进一步重写。这种虚继承链会无限继续，除非使用`sealed`关键字明确停止，如后续章节所述。

之前显示的类可以修改为在以下代码中使用虚属性`Image`，而不是虚方法`GetImage()`。在这种情况下，它们将如下所示：

```cs
class GameUnit
{
    public Position Position { get; protected set; }
    public GameUnit(Position position)
    {
        Position = position;
    }
    public void Draw(Surface surface)
    {
        surface.DrawAt(Image, Position);
    }
    protected virtual char Image => ' ';
}
class Terrain : GameUnit
{
    public Terrain(Position position) : base(position) { }
}
class Water : Terrain
{
    public Water(Position position) : base(position) { }
    protected override char Image => '░';
}
class Hill : Terrain
{
    public Hill(Position position) : base(position) { }
    protected override char Image => '≡';
}
```

有时候你希望一个方法在派生类中被重写，但在基类中不提供实现。这样的虚方法称为*抽象*，将在下一节中讨论。

## 抽象类和成员

到目前为止我们看到的例子有一个不便之处，因为`GameUnit`和`Terrain`类只是一些在游戏中没有实际表示的基类，我们仍然可以实例化它们。这是不幸的，因为我们只希望能够创建`Water`和`Hill`的对象。此外，`GetImage()`虚方法或`Image`虚属性必须在基类中有一个实现，这并没有太多意义。实际上，我们只希望在表示物理对象的类中有一个实现。这可以通过使用抽象类和成员来实现。

使用`abstract`关键字声明抽象类。抽象类不能被实例化，这意味着我们不能创建抽象类的对象。如果我们尝试创建抽象类的实例，将导致编译时错误。抽象类应该是其他类的基类，这些类将实现类定义的抽象。

抽象类必须包含至少一个抽象成员，可以是方法、属性、索引器或事件。抽象成员也使用`abstract`关键字声明。从抽象类派生的非抽象类必须实现所有继承的抽象成员和属性访问器。

我们可以使用抽象类和成员重写游戏单位示例。如下所示：

```cs
abstract class GameUnit
{
    public Position Position { get; protected set; }
    public GameUnit(Position position)
    {
        Position = position;
    }
    public void Draw(Surface surface)
    {
        surface.DrawAt(Image, Position);
    }
    protected abstract char Image { get; }
}
abstract class Terrain : GameUnit
{
    public Terrain(Position position) : base(position) { }
}
class Water : Terrain
{
    public Water(Position position) : base(position) { }
    protected override char Image => '░';
}
class Hill : Terrain
{
    public Hill(Position position) : base(position) { }
    protected override char Image => '≡';
}
```

在这个例子中，`GameUnit`类被声明为`abstract`。它有一个抽象属性`Image`，不再有实现。`Terrain`是从`GameUnit`派生的，但因为它没有重写抽象属性，它本身是一个抽象类，必须使用`abstract`修饰符声明。`Water`和`Hill`类都重写了`Image`属性，并使用`override`关键字进行了重写。

抽象类的一些特点如下：

+   抽象类可以有抽象和非抽象成员。

+   如果一个类包含抽象成员，则该类必须标记为`abstract`。

+   抽象成员不能是私有的。

+   抽象成员不能有实现。

+   抽象类必须为其实现的所有接口的所有成员提供实现（如果有的话）。

同样，抽象方法或属性具有以下特点：

+   抽象方法隐式地是虚方法。

+   声明为抽象的成员不能是`static`或`virtual`。

+   派生类中的实现必须在成员的声明中指定`override`关键字。

到目前为止，我们已经看到了如何派生和重写类和成员。然而，可以阻止这种情况发生。我们将在下一节中学习如何做到这一点。

## 封闭类和成员

如果我们想要限制一个类不被另一个类继承，那么我们将该类声明为`sealed`。如果我们尝试继承一个被封闭的类，将导致编译时错误。我们使用`sealed`关键字来创建一个封闭的类。参考以下示例：

```cs
sealed class Water : Terrain
{
   public Water(Position position) : base(position) { }
   protected override char Image => '░';
}
class Lake : Water  // ERROR: cannot derived from sealed type
{
   public Lake(Position position) : base(position) { }
}
```

这里的`Water`类被声明为`sealed`。尝试将其用作另一个类的基类将导致编译时错误。

不仅可以将类声明为`sealed`，还可以将重写的成员声明为`sealed`。类可以通过在`override`前面使用`sealed`关键字来阻止成员的虚继承。在进一步派生类中再次尝试重写它将导致编译器错误。

在以下示例中，`Water`类没有被封闭，但其`Image`属性被封闭。尝试在`Lake`派生类中重写它将产生编译器错误：

```cs
class Water : Terrain
{
    public Water(Position position) : base(position) { }
    protected sealed override char Image => '░';
}

class Lake : Water
{
    public Lake(Position position) : base(position) { }
    protected sealed override char Image => '░';  // ERROR
}
```

现在我们已经看到了如何使用封闭类和成员，让我们看看如何隐藏基类成员。

## 隐藏基类成员

在某些情况下，您可能希望在派生类中使用`new`关键字在成员的返回类型前面隐藏基类的现有成员，而不是*虚调用*（即在类层次结构中调用虚方法）。可以通过在派生类中成员的返回类型前面使用`new`关键字来实现这一点，如以下示例所示：

```cs
class Base
{
    public int Get() { return 42; }
}
class Derived : Base
{
    public new int Get() { return 10; }
}
```

以这种方式定义的新成员在通过对派生类型的引用调用时将被调用。然而，如果通过对基类型的引用调用成员，则将调用隐藏的基成员，如下所示：

```cs
Derived d = new Derived();
Console.WriteLine(d.Get()); // prints 10
Base b = d;
Console.WriteLine(b.Get()); // prints 42
```

与虚方法不同，虚方法是根据用于调用它们的对象的运行时类型在运行时调用的，隐藏方法是根据用于调用它们的对象的编译时类型在编译时解析的。

隐藏成员的一个可能用途在以下示例中显示，我们有一个需要支持克隆方法的类层次结构。然而，每个类应该返回自己的新副本，而不是基类的引用：

```cs
class Pet
{
     public string Name { get; private set; }
     public Pet(string name)
     { Name = name; }
     public Pet Clone() { return new Pet(Name); }
}
class Dog : Pet
{
     public string Color { get; private set; }
     public Dog(string name, string color):base(name)
     { Color = color; }
     public new Dog Clone() { return new Dog(Name, Color); }
}
```

有了这些定义，我们可以编写以下代码：

```cs
Pet pet = new Pet("Lola");
Dog dog = new Dog("Rex", "black");
Pet cpet = pet.Clone();
Dog ddog = dog.Clone();
```

请注意，这仅在我们从该类的对象调用`Clone()`方法时才起作用，而不是通过对基类的引用。因为调用在编译时解析，如果你有一个对`Pet`的引用，即使对象的运行时类型是`Dog`，也只会克隆`Pet`。这在以下示例中得到了说明：

```cs
Pet another = new Dog("Dark", "white");
Dog copy = another.Clone(); // ERROR this method returns a Pet
```

一般来说，成员隐藏被认为是代码异味（即设计和代码库中存在更深层次问题的指示）并且应该避免。通过成员隐藏实现的目标通常可以通过更好的方式实现。例如，这里显示的克隆示例可以通过使用创建型设计模式，通常是**原型**模式，但可能还有其他模式，如**工厂方法**来实现。

到目前为止，在本章中，我们已经看到了如何创建类和类的层次结构。面向对象编程中的另一个重要概念是接口，这是我们接下来要讨论的主题。

## 接口

接口包含一组必须由实现接口的任何类或结构实现的成员。接口定义了一个由实现接口的所有类型支持的合同。这也意味着使用接口的客户端不需要了解任何关于实际实现细节的信息，这有助于松耦合，有助于维护和可测试性。

因为语言不支持多重类继承或结构的继承，接口提供了一种模拟它们的方法。无论是引用类型还是值类型，类型都可以实现任意数量的接口。

通常，接口*只包含成员的声明*，而*不包含实现*。从 C# 8 开始，接口可以包含默认方法；这是一个将在*第十五章*中详细介绍的主题，*C# 8 的新特性*。在 C#中，接口使用`interface`关键字声明。

以下列表包含在使用接口时需要考虑的重要要点：

+   接口只能包含方法、属性、索引器和事件。它们不能包含字段。

+   如果一个类型实现了一个接口，那么它必须为接口的所有成员提供实现。接口的方法签名和返回类型不能被实现接口的类型改变。

+   当一个接口定义属性或索引器时，实现可以为它们提供额外的访问器。例如，如果接口中的属性只有`get`访问器，实现也可以提供`set`访问器。

+   接口不能有构造函数或运算符。

+   接口不能有静态成员。

+   接口成员隐式定义为`public`。如果尝试在接口的成员上使用访问修饰符，将导致编译时错误。

+   一个接口可以被多种类型实现。一个类型可以实现多个接口。

+   如果一个类从另一个类继承并同时实现一个接口，那么基类名称必须在接口名称之前，用逗号分隔。

+   通常，接口名称以字母`I`开头，比如`IEnumerable`、`IList<T>`等等。

为了理解接口的工作原理，我们将考虑游戏单位的示例。

在以前的实现中，我们有一个名为`Surface`的类，负责绘制游戏对象。我们的实现是打印到控制台，但这可以是任何东西——游戏窗口、内存、位图等等。为了能够轻松地在这些之间进行切换，并且不将`GameUnit`类与特定的表面实现绑定，我们可以定义一个接口，指定任何实现必须提供的功能。然后游戏单位将使用这个接口进行渲染。这样的接口可以定义如下：

```cs
interface ISurface
{
    void BeginDraw();
    void EndDraw();
    void DrawAt(char c, Position position);
}
```

它包含三个成员函数，都是隐式的`public`。然后`Surface`类将实现这个接口：

```cs
class Surface : ISurface
{
    private int left;
    private int top;
    public void BeginDraw()
    {
        Console.Clear();
        left = Console.CursorLeft;
        top = Console.CursorTop;
    }
    public void EndDraw()
    {
        Console.WriteLine();
    }
    public void DrawAt(char c, Position position)
    {
        try
        {
            Console.SetCursorPosition(left + position.X, 
                                      top + position.Y);
            Console.Write(c);
        }
        catch (ArgumentOutOfRangeException e)
        {
            Console.Clear();
            Console.WriteLine(e.Message);
        }
    }
}
```

该类必须实现接口的所有成员。但是也可以跳过。在这种情况下，类必须是抽象的，并且必须声明抽象成员以匹配它没有实现的接口成员。

在前面的例子中，`Surface`类实现了`ISurface`接口的所有三个方法。这些方法被明确声明为`public`。使用其他访问修饰符会导致编译错误，因为接口中的成员在类中是隐式公共的，类不能降低它们的可见性。`GameUnit`类将发生变化，使得`Draw()`方法将有一个`ISurface`参数：

```cs
abstract class GameUnit
{
    public Position Position { get; protected set; }
    public GameUnit(Position position)
    {
        Position = position;
    }
    public void Draw(ISurface surface)
    {
        surface?.DrawAt(Image, Position);
    }
    protected abstract char Image { get; }
}
```

让我们进一步扩展这个例子，考虑另一个名为`IMoveable`的接口，它定义了一个`MoveTo()`方法，将游戏对象移动到另一个位置：

```cs
interface IMoveable
{
    void MoveTo(Position pos);
}
```

这个接口将被所有可以移动的游戏对象实现，比如人、机器等等。一个名为`ActionUnit`的类作为所有这些对象的基类，并实现了`IMoveable`：

```cs
abstract class ActionUnit : GameUnit, IMoveable
{
    public ActionUnit(Position position) : base(position) { }
    public void MoveTo(Position pos)
    {
        Position = pos;
    }
}
```

`ActionUnit`也是从`GameUnit`派生的，因此基类出现在接口列表之前。然而，由于这个类只作为其他类的基类，它不实现`Image`属性，因此必须是抽象的。下面的代码显示了一个从`ActionUnit`派生的`Meeple`类：

```cs
class Meeple : ActionUnit
{
    public Meeple(Position position) : base(position) { }
    protected override char Image => 'M';
}
```

我们可以使用`Meeple`类的实例来扩展我们在之前示例中构建的游戏：

```cs
var objects = new List<GameUnit>()
{
    new Water(new Position(3, 2)),
    new Water(new Position(4, 2)),
    new Water(new Position(5, 2)),
    new Hill(new Position(3, 1)),
    new Hill(new Position(5, 3)),
    new Meeple(new Position(0, 0)),
    new Meeple(new Position(4, 3)),
};
ISurface surface = new Surface();
surface.BeginDraw();
foreach (var unit in objects)
   unit.Draw(surface);
surface.EndDraw();
```

该程序的输出如下：

![图 5.6 - 修改后的游戏执行的控制台输出](img/Figure_5.6_B12346.jpg)

图 5.6 - 修改后的游戏执行的控制台输出

现在我们已经了解了继承，是时候看看面向对象编程的最后一个支柱，即多态性了。

# 多态性

面向对象编程的最后一个核心支柱是多态性。多态性是一个希腊词，代表着*多种形式*。这是使用一个实体的多种形式的能力。有两种类型的多态性：

+   *编译时多态性*：当我们有相同名称但参数数量或类型不同的方法时，这被称为方法重载。

+   *运行时多态性*：这有两个不同的方面：

一方面，派生类的对象可以无缝地用作数组或其他类型的集合、方法参数和其他位置中的基类对象。

另一方面，类可以定义虚拟方法，可以在派生类中重写。在运行时，**公共语言运行时**（**CLR**）将调用与对象的运行时类型相对应的虚拟成员的实现。当派生类的对象被用于替代基类的对象时，对象的声明类型和运行时类型不同时。

多态性促进了代码重用，这可以使代码更容易阅读、测试和维护。它还促进了关注点的分离，这是面向对象编程中的一个重要原则。另一个好处是它有助于隐藏实现细节，因为它允许通过一个公共接口与不同的类进行交互。

在前面的章节中，我们已经看到了这两个方面的例子。我们已经看到了如何声明虚拟成员以及如何重写它们，以及如何使用`sealed`关键字停止虚拟继承。我们还看到了派生类的对象在基类数组中的使用的例子。这里再次是这样一个例子：

```cs
var objects = new List<GameUnit>()
{
    new Water(new Position(3, 2)),
    new Hill(new Position(3, 1)),
    new Meeple(new Position(0, 0)),
};
```

编译时多态性由*方法和运算符重载*表示。我们将在接下来的章节中探讨这些。

## 方法重载

方法重载允许我们在同一个类中声明两个或多个具有相同名称但不同参数的方法。这可以是不同数量的参数或不同类型的参数。返回类型不考虑重载解析。如果两个方法只在返回类型上有所不同，那么编译器将发出错误。此外，`ref`、`in`和`out`参数修饰符不参与重载解析。这意味着两个方法不能仅在参数修饰符上有所不同，比如一个方法有一个`ref`参数，另一个方法有相同的参数指定为`in`或`out`修饰符。另一方面，一个没有修饰符的参数的方法可以被一个具有相同参数指定为`ref`、`in`或`out`的方法重载。

让我们看下面的例子来理解方法重载。考虑之前显示的`IMoveable`接口，我们可以修改它，使其包含两个名为`MoveTo()`的方法，参数不同：

```cs
interface IMoveable
{
    void MoveTo(Position pos);
    void MoveTo(int x, int y);
}
abstract class ActionUnit : GameUnit, IMoveable
{
    public ActionUnit(Position position) : base(position) { }
    public void MoveTo(Position pos)
    {
        Position = pos;
    }
    public void MoveTo(int x, int y)
    {
        Position = new Position(x, y);
    }
}
```

`ActionUnit`类提供了这两种重载的实现。当调用重载的方法时，编译器会根据提供的参数的类型和数量找到最佳匹配，并调用适当的重载。示例如下：

```cs
Meeple m = new Meeple(new Position(3, 4));
m.MoveTo(new Position(1, 1));
m.MoveTo(2, 5);
```

识别方法调用的最佳匹配的过程称为*重载解析*。有许多规则定义了如何找到最佳匹配，列出它们都超出了本书的范围。简单来说，重载解析的执行如下：

1.  创建一个具有指定名称的成员集合。

1.  消除从调用范围不可访问的所有成员。

1.  消除所有不适用的成员。适用的成员是指每个参数都有一个参数，并且参数可以隐式转换为参数的类型。

1.  如果一个成员有一个带有可变数量参数的形式，那么评估它们并消除不适用的形式。

1.  对于剩下的集合，应用找到最佳匹配的规则。更具体的参数比不太具体的更好。这意味着，例如，更具体的派生类比基类更好。此外，非泛型参数比泛型参数更具体。

与方法重载类似，但语法和语义略有不同的是运算符重载，我们将在下面看到。

## 运算符重载

运算符重载允许我们针对特定类型提供用户定义的功能。当一个或两个操作数是该类型时，类型可以为可重载的运算符提供自定义实现。

在实现运算符重载时需要考虑的一些重要点如下：

+   `operator`关键字用于声明运算符。这样的方法必须是`public`和`static`。

+   赋值运算符不能被重载。

+   重载运算符方法的参数不应该使用`ref`、`in`或`out`修饰符。

+   我们不能通过运算符重载改变运算符的优先级。

+   我们不能改变运算符所需的操作数数量。但是，重载的运算符可以忽略一个操作数。

C#语言有一元、二元和三元运算符。然而，只有前两类运算符可以被重载。让我们从学习如何重载二元运算符开始。

### 重载二元运算符

二元运算符的至少一个参数必须是`T`或`T?`类型，其中`T`是定义运算符的类型。

让我们考虑一下我们想要重载运算符的类型：

```cs
struct Complex
{
    public double Real      { get; private set; }
    public double Imaginary { get; private set; }
    public Complex(double real = 0, double imaginary = 0)
    {
        Real = real;
        Imaginary = imaginary;
    }
}
```

这是一个非常简单的复数实现，只有实部和虚部两个属性。我们希望能够进行加法和减法等算术运算，如下所示：

```cs
var c1 = new Complex(2, 3);
var c2 = new Complex(4, 5);
var c3 = c1 + c2;
var c4 = c1 - c2;
```

要这样做，我们必须按照以下方式重载`+`和`-`二元运算符（前面显示的`Complex`结构的部分为简单起见而省略）：

```cs
public struct Complex
{
    // [...] omitted members
    public static Complex operator +(Complex number1, 
                                     Complex number2)
    {
        return new Complex()
        {
            Real = number1.Real + number2.Real,
            Imaginary = number2.Imaginary + number2.Imaginary
        };
    }
    public static Complex operator -(Complex number1, 
                                     Complex number2)
    {
        return new Complex()
        {
            Real = number1.Real - number2.Real,
            Imaginary = number2.Imaginary - number2.Imaginary
        };
    }
}
```

我们可能还想要能够进行对象比较。在这种情况下，我们需要重载`==`、`!=`、`<`、`>`、`<=`或`>=`运算符或它们的组合：

```cs
if (c3 == c2) { /* do something */}
if (c1 != c4) { /* do something else */}
```

在下面的清单中，您可以看到`Complex`类型的`==`和`!=`运算符的实现：

```cs
struct Complex
{
    // [...] omitted members
    public static bool operator ==(Complex number1, 
                                   Complex number2)
    {
        return number1.Real.Equals(number2.Real) &&
               number2.Imaginary.Equals(number2.Imaginary);
    }
    public static bool operator !=(Complex number1, 
      Complex number2)
    {
        return !number1.Real.Equals(number2.Real) ||
               !number2.Imaginary.Equals(number2.Imaginary);
    }
    public override bool Equals(object obj)
    {
        return Real.Equals(((Complex)obj).Real) &&
               Imaginary.Equals(((Complex)obj).Imaginary);
    }
    public override int GetHashCode()
    {
        return Real.GetHashCode() * 17 + 
          Imaginary.GetHashCode();
    }
}
```

在重载比较运算符时，你必须按照成对实现它们，如前所述：

+   如果你重载`==`或`!=`，你必须同时重载它们。

+   如果你重载`<`或`>`，你必须同时重载它们。

+   如果你重载`=<`或`>=`，你必须同时重载它们。

此外，当你重载`==`和`!=`时，你还需要重写`System.Object`的虚拟方法，`Equals()`和`GetHashCode()`。

### 重载一元运算符

一元运算符的单个参数必须是`T`或`T?`，其中`T`是定义运算符的类型。

我们将再次使用`Complex`类型和增量和减量运算符进行举例。可以实现如下：

```cs
struct Complex
{
    // [...] omitted members
    public static Complex operator ++(Complex number)
    {
        return new Complex(number.Real + 1, number.Imaginary);
    }
    public static Complex operator --(Complex number)
    {
        return new Complex(number.Real - 1, number.Imaginary);
    }
}
```

在这个实现中，增量(`++`)运算符和减量(`--`)运算符只改变复数的实部，并返回一个新的复数。然后我们可以编写以下代码来展示这些运算符如何被使用：

```cs
var c = new Complex(5, 7);
Console.WriteLine(c);  // 5i + 7
c++;
Console.WriteLine(c);  // 6i + 7
++c;
Console.WriteLine(c);  // 7i + 7
```

需要注意的是，当调用增量或减量运算符时，操作的对象被赋予一个新值。对于引用类型来说，这意味着被赋予一个新对象的引用。因此，增量和减量运算符不应该修改原始对象并返回对其的引用。通过将`Complex`类型实现为一个类来理解原因：

```cs
public class Complex
{
    // [...] omitted members
    public static Complex operator ++(Complex number)
    {
        // WRONG implementation
        number.Real++;
        return number;
    }
}
```

这个实现是错误的，因为它会影响对修改后对象的所有引用。考虑以下例子：

```cs
var c1 = new Complex(5, 7);
var c2 = c1;
Console.WriteLine(c1);  // 5i + 7
Console.WriteLine(c2);  // 5i + 7
c1++;
Console.WriteLine(c1);  // 6i + 7
Console.WriteLine(c2);  // 6i + 7
```

最初，`c1`和`c2`是相等的。然后我们增加了`c1`的值，由于`Complex`类中`++`运算符的实现，`c1`和`c2`将具有相同的值。正确的实现如下：

```cs
class Complex
{
    // [...] omitted members 
    public static Complex operator ++(Complex number)
    {
        return new Complex(number.Real + 1, number.Imaginary);
    }
}
```

虽然这对值类型不是问题，但你应该养成从一元运算符返回一个新对象的习惯。

# SOLID 原则

我们在本章讨论的原则——抽象、封装、继承和多态——是面向对象编程的支柱。然而，这些并不是开发人员在进行面向对象编程时所采用的唯一原则。还有许多其他原则，但在这一点上值得一提的是由缩写**SOLID**所知的五个原则。这些最初是由 Robert C. Martin 在 2000 年在一篇名为*设计原则和设计模式*的论文中首次提出的。后来，Michael Feathers 创造了 SOLID 这个术语：

+   **S**代表**单一职责原则**，它规定一个模块或类应该只有一个职责，其中职责被定义为变化的原因。当一个类提供的功能可能在不同时间和出于不同原因而发生变化时，这意味着这些功能不应该放在一起，应该分开成不同的类。

+   **O**代表**开闭原则**，它规定一个模块、类或函数应该对扩展开放，但对修改关闭。也就是说，当功能需要改变时，这些改变不应该影响现有的实现。继承是实现这一点的典型方式，因为派生类可以添加更多功能或专门化现有功能。扩展方法是 C#中的另一种可用技术。

+   **L**代表**里氏替换原则**，它规定如果 S 是 T 的子类型，那么 T 的对象可以被 S 的对象替换而不会破坏程序的功能。这个原则是以首次提出它的 Barbara Liskov 的名字命名的。为了理解这个原则，让我们考虑一个处理形状的系统。我们可能有一个椭圆类，其中有方法来改变它的两个焦点。当实现一个圆时，我们可能会倾向于专门化椭圆类，因为在数学上，圆是具有两个相等焦点的特殊椭圆。在这种情况下，圆必须在这两个方法中将两个焦点设置为相同的值。这是客户端不期望的，因此椭圆不能替换圆。为了避免违反这个原则，我们必须实现圆而不是从椭圆派生。为了确保遵循这个原则，你应该为所有方法定义前置条件和后置条件。前置条件在方法执行之前必须为真，后置条件在方法执行后必须为真。当专门化一个方法时，你只能用更弱的前置条件和更强的后置条件替换它的前置条件和后置条件。

+   **I**代表**接口隔离原则**，它规定更小、更具体的接口比更大、更一般的接口更可取。原因是客户端可能只需要实现它需要的功能，而不需要其他的。通过分离职责，这个原则促进了组合和解耦。

+   **D**代表**依赖反转原则**，是列表中的最后一个。该原则规定软件实体应依赖于抽象而不是实现。高级模块不应依赖低级模块；相反，它们都应该依赖于抽象。此外，抽象不应依赖具体实现，而是相反。对实现的依赖引入了紧耦合，使得难以替换组件。然而，对高级抽象的依赖解耦了模块，并促进了灵活性和可重用性。

这五个原则使我们能够编写更简单、更易理解的代码，这也使得它更容易维护。同时，它们使代码更具可重用性，也更容易测试。

# 摘要

在本章中，我们学习了面向对象编程的核心概念：抽象、封装、继承和多态。我们了解了使它们成为可能的语言功能，比如继承、虚成员、抽象类型和成员、密封类型和成员、接口，以及方法和运算符重载。在本章末尾，我们简要讨论了其他被称为 SOLID 的面向对象原则。

在下一章中，我们将学习 C#中的另一种编程范式——泛型编程。

# 测试你学到的东西

1.  什么是面向对象编程，其核心原则是什么？

1.  封装有哪些好处？

1.  什么是继承，C#支持哪些类型的继承？

1.  什么是虚方法？重写方法呢？

1.  如何防止派生类中的虚成员被重写？

1.  什么是抽象类，它们有哪些特点？

1.  什么是接口，它可以包含哪些成员？

1.  存在哪些多态类型？

1.  什么是重载方法？如何重载运算符？

1.  什么是 SOLID 原则？

# 进一步阅读

+   *由 Robert C. Martin 编写的《设计原则和设计模式》*：[`web.archive.org/web/20150906155800/http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf`](https://web.archive.org/web/20150906155800/http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf)
