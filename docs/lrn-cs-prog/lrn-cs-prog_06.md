# 第六章：泛型

在上一章中，我们学习了 C#中的面向对象编程。在本章中，我们将探讨泛型的概念。泛型允许我们以一种类型安全的环境中使用不同的数据类型创建类、结构、接口、方法和委托。泛型是作为 C# 2.0 版本的一部分添加的。它促进了代码的可重用性和可扩展性，是 C#最强大的特性之一。

在本章中，我们将学习以下概念：

+   泛型类和泛型继承

+   泛型接口和变体泛型接口

+   泛型结构

+   泛型方法

+   类型约束

通过本章结束时，您将具备编写泛型类型、方法和变体泛型接口以及使用类型约束所需的技能。

# 理解泛型

简而言之，泛型是用其他类型参数化的类型。正如我们之前提到的，我们可以创建一个类、结构、接口、方法或委托，它们接受一个或多个数据类型作为参数。这些参数被称为**类型参数**，充当编译时传递的实际数据类型的*占位符*。

例如，我们可以创建一个模拟列表的类，它是相同类型元素的可变长度序列。我们可以创建一个泛型类，它具有指定其元素实际类型的类型参数。然后，当我们实例化类时，我们将在编译时指定实际类型。

使用泛型的优点包括以下内容：

+   **泛型提供了可重用性**：我们可以创建代码的单个版本，并将其用于不同的数据类型。

+   **泛型提倡类型安全**：在使用泛型时，我们不需要执行显式类型转换。类型转换由编译器处理。

+   将`object`类型转换为引用类型是耗时的。因此，通过避免这些操作，它们有助于提高执行时间。

泛型类型和方法可以受限，以便只有满足要求的类型可以用作类型参数。关于实际类型的信息用于实例化可以在运行时使用反射获得的泛型类型。

泛型最常见的用途是创建集合或包装类。集合将是下一章的主题。

# 泛型类型

引用类型和值类型都可以是泛型的。我们已经在本书的早期看到了泛型类型的例子，比如`Nullable<T>`和`List<T>`。

在本节中，我们将学习如何创建泛型类、结构和接口。

## 泛型类

创建泛型类与创建非泛型类没有区别。唯一不同的是类型参数列表及其在类中作为实际类型的占位符的使用。让我们看一个泛型类的例子：

```cs
public class GenericDemo<T>
{
    public T Value { get; private set; }
    public GenericDemo(T value)
    {
        Value = value;
    }
    public override string ToString() => $"{typeof(T)} : {Value}";
}
```

在这里，我们定义了一个泛型类`GenericDemo`，它接受一个类型参数`T`。我们定义了一个名为`Value`的`T`类型属性，并在类构造函数中对其进行了初始化。构造函数接受`T`类型的参数。重写的方法`ToString()`将返回一个包含属性类型和值的字符串。

要实例化这个泛型类的对象，我们将按以下步骤进行：

```cs
var obj1 = new GenericDemo<int>(10);
var obj2 = new GenericDemo<string>("Hello World");
```

在这个例子中，我们在创建泛型类`GenericDemo<T>`的对象时为类型参数指定了数据类型。`obj1`和`obj2`都是相同泛型类型的实例，但它们的类型参数不同：一个是`int`，另一个是`string`。因此，它们彼此不兼容。这意味着如果我们尝试将一个对象分配给另一个对象，将导致编译时错误。

我们可以使用反射来获取关于这些对象类型和它们的通用类型参数的信息（我们将在第十一章“反射和动态编程”中进行讨论），如下面的示例所示：

```cs
var t1 = obj1.GetType();
Console.WriteLine(t1.Name);
Console.WriteLine(t1.GetGenericArguments()
                    .FirstOrDefault().Name);
var t2 = obj2.GetType();
Console.WriteLine(t2.Name);
Console.WriteLine(t2.GetGenericArguments()
                    .FirstOrDefault().Name);
Console.WriteLine(obj1);
Console.WriteLine(obj2);
```

执行后，我们将看到以下输出：

![图 6.1 - 显示类型反射内容的控制台截图](img/Figure_6.1_B12346.jpg)

图 6.1 - 显示类型反射内容的控制台截图

我们可以为泛型类型声明多个类型参数。在这种情况下，我们需要将所有类型参数指定为角括号内的逗号分隔值。以下是一个示例：

```cs
class Pair<T, U>
{
    public T Item1 { get; private set; }
    public U Item2 { get; private set; }
    public Pair(T item1, U item2)
    {
        Item1 = item1;
        Item2 = item2;
    }
}
var p1 = new Pair<int, int>(1, 2);
var p2 = new Pair<int, double>(1, 42.99);
var p3 = new Pair<string, bool>("true", true);
```

在这里，`Pair<T, U>`是一个需要两个类型参数的类。我们使用不同类型的组合来实例化对象`p1`、`p2`和`p3`。

这个类实际上与.NET 类`KeyValueType<TKey`,`TValue>`非常相似，它来自`System.Collections.Generic`命名空间。实际上，框架提供了许多泛型类。您应该在可能的情况下使用现有类型，而不是定义自己的类型。

## 泛型类的继承

泛型类可以作为*基类*或*派生类*。当从泛型类派生时，子类必须指定基类所需的类型参数。这些类型参数可以是实际类型，也可以是派生类的类型参数，即泛型类。

让我们通过这里展示的示例来理解泛型类的继承是如何工作的：

```cs
public abstract class Shape<T>
{
    public abstract T Area { get; }
}
```

我们定义了一个泛型抽象类`Shape`，其中包含一个表示形状面积的单个抽象属性`Area`。该属性的类型也是`T`。考虑这里的类定义：

```cs
public class Square : Shape<int>
{
    public int Length { get; set; }
    public Square(int length)
    {
        Length = length;
    }
    public override int Area => Length * Length;
}
```

在这里，我们定义了一个名为`Square`的类，它继承自泛型抽象类`Shape`。我们使用`int`类型作为类型参数。我们为`Square`类定义了一个名为`Length`的属性，并在构造函数中对其进行了初始化。我们重写了`Area`属性以计算正方形的面积。现在，考虑下面的另一个类定义：

```cs
public class Circle : Shape<double>
{
    public double Radius { get; set; }
    public Circle(double radius)
    {
        Radius = radius;
    }
    public override double Area => Math.PI * Radius * Radius;
}
```

`Circle`类也继承自泛型抽象类`Shape<T>`。父类`Shape`的类型参数现在指定为`double`。定义了`Radius`属性来存储圆的半径。我们再次重写了`Area`属性以计算圆的面积。我们可以如下使用这些派生类：

```cs
Square objSquare = new Square(10);
Console.WriteLine($"The area of square is {objSquare.Area}");
Circle objCircle = new Circle(7.5);
Console.WriteLine($"The area of circle is {objCircle.Area}");
```

我们创建`Square`和`Circle`的实例，并将每个形状的面积打印到控制台上。执行后，我们将看到以下输出：

![图 6.2 - 正方形和圆的面积显示在控制台上](img/Figure_6.2_B12346.jpg)

图 6.2 - 正方形和圆的面积显示在控制台上

重要的是要注意，尽管`Square`和`Circle`都是从`Shape<T>`派生出来的，但这些类型不能被多态地对待。一个是`Shape<int>`，另一个是`Shape<double>`。因此，`Square`和`Circle`的实例不能放在同质容器中。唯一可能的解决方案是使用`object`类型来保存对这些实例的引用，然后执行类型转换。

在这个例子中，`Shape<T>`是一个泛型类型。`Shape<int>`是从`Shape<T>`构造出来的类型，通过用`int`替换类型参数`T`。这样的类型被称为**构造类型**。这也是一个*封闭构造类型*，因为所有类型参数都已被替换。非泛型类型都是*封闭类型*。泛型类型是*开放类型*。构造泛型类型可以是开放的或封闭的。开放构造类型是具有未被替换的类型参数的类型。封闭构造类型是任何不是开放的类型。

创建通用类型时另一个重要的事情是，一些运算符，比如算术运算符，不能与类型参数的对象一起使用。让我们看下面的代码来举例说明这种情况：

```cs
public class Square<T> : Shape<T>
{
    public T Length { get; set; }
    public Square(T length)
    {
        Length = length;
    }
    /* ERROR: Operator '*' cannot be applied to operands 
    of type 'T' and 'T' */
    public override T Area => Length * Length;
}
```

`Square`类型现在是一个通用类型。类型参数`T`用于基类的类型参数以及`Length`属性。然而，在计算面积时，使用`*`运算符会产生编译错误。这是因为编译器不知道`T`将使用什么具体类型，以及它们是否已重载`*`运算符。为了确保在任何情况下都不会发生无效实例化，编译器会生成错误。

可以确保只有符合预定义约束的类型在编译时用于实例化通用类型或调用通用方法。这些被称为*类型约束*，将在本章的*类型参数约束*部分中讨论。

现在我们已经看到如何创建和使用通用类，让我们看看如何使用通用接口。

## 通用接口

在前面的例子中，通用类`Shape<T>`除了一个抽象属性之外什么也没有。这不是一个好的类候选，它应该是一个接口。通用接口与非通用接口的区别与通用类与非通用类的区别相同。以下是一个通用接口的例子：

```cs
public interface IShape<T>
{
    public T Area { get; }
}
```

类型参数的指定方式与类或结构相同。这个接口可以这样实现：

```cs
public class Square : IShape<int>
{
    public int Length { get; set; }
    public Square(int length)
    {
        Length = length;
    }
    public int Area => Length * Length;
}
public class Circle : IShape<double>
{
    public double Radius { get; set; }
    public Circle(double radius)
    {
        Radius = radius;
    }
    public double Area => Math.PI * Radius * Radius;
}
```

`Square`和`Circle`类的实现与前一节中所见的略有不同。

具体类，比如这里的`Square`和`Circle`，可以实现封闭构造的接口，比如`IShape<int>`或`IShape<double>`。如果类参数列表提供了接口所需的所有类型参数，通用类也可以实现通用或封闭构造的接口。另一方面，通用接口可以继承非通用接口；然而，通用类必须是逆变的。

通用接口的变异将在下一节中讨论。

## 变异通用接口

可以将通用接口中的类型参数声明为*协变*或*逆变*：

+   *协变*类型参数用`out`关键字声明，允许接口方法具有比指定类型参数更多派生的返回类型。

+   *逆变*类型参数用`in`关键字声明，允许接口方法具有比指定类型参数更少派生的参数。

具有协变或逆变类型参数的通用接口称为**变异通用接口**。变异只支持引用类型。

为了理解协变是如何工作的，让我们看看`System.IEnumerable<T>`通用接口。这是一个变异接口，因为它的类型参数声明为协变。接口定义如下：

```cs
public interface IEnumerable
{
    IEnumerator GetEnumerator();
}
public interface IEnumerable<out T> : IEnumerable
{
    IEnumerator<T> GetEnumerator();
}
```

实现`IEnumerable<T>`（和其他接口）的类是`List<T>`。因为`T`是协变的，我们可以编写以下代码：

```cs
IEnumerable<string> names = 
   new List<string> { "Marius", "Ankit", "Raffaele" };
IEnumerable<object> objects = names;
```

在这个例子中，`names`是`IEnumerable<string>`，`objects`是`IEnumerable<object>`。前者不派生自后者，但`string`派生自`object`，并且因为`T`是协变的，我们可以将`names`赋值给`objects`。然而，这只有在使用变异接口时才可能。

实现变异接口的类本身不是变异的，而是不变的。这意味着下面的例子，我们用`List<T>`替换`IEnumerable<T>`，将产生编译错误，因为`List<string>`不能赋值给`List<object>`：

```cs
IEnumerable<string> names = 
   new List<string> { "Marius", "Ankit", "Raffaele" };
List<object> objects = names; // error
```

如前所述，值类型不支持变异。`IEnumerable<int>`不能赋值给`IEnumerable<object>`：

```cs
IEnumerable<int> numbers = new List<int> { 1, 1, 2, 3, 5, 8 };
IEnumerable<object> objects = numbers; // error
```

总之，接口中的协变类型参数必须：

+   必须以`out`关键字为前缀

+   只能用作方法的返回类型，而不能用作方法参数的类型

+   不能用作接口方法的泛型约束

逆变是处理传递给接口方法的参数的另一种变体形式。为了理解它是如何工作的，让我们考虑一个情况，我们想要比较各种形状的大小，定义如下：

```cs
public interface IShape
{
    public double Area { get; }
}
public class Square : IShape
{
    public double Length { get; set; }
    public Square(int length)
    {
        Length = length;
    }
    public double Area => Length * Length;
}
public class Circle : IShape
{
    public double Radius { get; set; }
    public Circle(double radius)
    {
        Radius = radius;
    }
    public double Area => Math.PI * Radius * Radius;
}
```

这些与之前使用的类型略有不同，因为`IShape`不再是泛型，以保持示例简单。我们想要的是能够比较形状。为此，提供了一系列类，如下所示：

```cs
public class ShapeComparer : IComparer<IShape>
{
    public int Compare(IShape x, IShape y)
    {
        if (x is null) return y is null ? 0 : -1;
        if (y is null) return 1;
        return x.Area.CompareTo(y.Area);
    }
}
public class SquareComparer : IComparer<Square>
{
    public int Compare(Square x, Square y)
    {
        if (x is null) return y is null ? 0 : -1;
        if (y is null) return 1;
        return x.Length.CompareTo(y.Length);
    }
}
public class CircleComparer : IComparer<Circle>
{
    public int Compare(Circle x, Circle y)
    {
        if (x is null) return y is null ? 0 : -1;
        if (y is null) return 1;
        return x.Radius.CompareTo(y.Radius);
    }
}
```

在这里，`ShapeComparer`通过它们的面积比较`IShape`对象，`SquareComparer`通过它们的长度比较正方形，`CircleComparer`通过它们的半径比较圆。所有这些类都实现了`System.Collections.Generic`命名空间中的`IComparer<T>`接口。该接口定义如下：

```cs
public interface IComparer<in T>
{
    int Compare(T x, T y);
}
```

这个接口有一个名为`Compare()`的方法，它接受两个`T`类型的对象并返回以下之一：

+   如果第一个小于第二个，则为负数

+   如果它们相等，则为 0

+   如果第一个大于第二个，则为正数

然而，其定义的关键是使用类型参数的`in`关键字，使其逆变。因此，可以在期望`Square`或`Circle`的地方传递`IShape`引用。这意味着我们可以安全地传递`IComparer<IShape>`到需要`IComparer<Square>`的地方。让我们看一个具体的例子。

以下类包含一个检查`Square`对象是否比另一个大的方法。`IsBigger()`方法还接受一个实现`IComparer<Square>`的对象的引用：

```cs
public class SquareComparison
{
    public static bool IsBigger(Square a, Square b,
                                IComparer<Square> comparer)
    {
        return comparer.Compare(a, b) >= 0;
    }
}
```

我们可以调用这个方法传递`SquareComparer`或`ShapeComparer`，结果将是相同的：

```cs
Square sqr1 = new Square(4);
Square sqr2 = new Square(5);
SquareComparison.IsBigger(sqr1, sqr2, new SquareComparer());
SquareComparison.IsBigger(sqr1, sqr2, new ShapeComparer());
```

如果`IComparer<T>`接口是不变的，传递`ShapeComparer`将导致编译错误。如果我们尝试传递`CircleComparer`，也会发出编译错误，因为`Circle`不是`Square`的派生类，它实际上是继承层次结构中的同级。

总之，接口中的逆变类型参数：

+   必须以`in`关键字为前缀

+   只能用于方法参数，而不能作为返回类型

+   可以用作接口方法的泛型约束

可以定义一个既是*协变又是逆变*的接口，如下所示：

```cs
interface IMultiVariant<out T, in U>
{
    T Make();
    void Take(U arg);
}
```

在前面的片段中显示的`IMultiVariant<T, U>`接口对`T`是协变的，对`U`是逆变的。

## 泛型结构

与泛型类类似，我们也可以创建泛型结构。泛型结构的语法与泛型类相同。在前面的示例中使用的`Circle`和`Square`类型很小，可以定义为结构而不是类：

```cs
public struct Square : IShape<int>
{
    public int Length { get; set; }
    public Square(int length)
    {
        Length = length;
    }
    public int Area => Length * Length;
}
public struct Circle : IShape<double>
{
    public double Radius { get; set; }
    public Circle(double radius)
    {
        Radius = radius;
    }
    public double Area => Math.PI * Radius * Radius;
}
```

所有适用于泛型类的规则也适用于泛型结构。因为值类型不支持继承，结构不能从其他泛型类型派生，但可以实现任意数量的泛型或非泛型接口。

# 泛型方法

C#允许我们创建接受一个或多个泛型类型参数的泛型方法。我们可以在泛型类内部创建泛型方法，也可以在非泛型类内部创建泛型方法。静态方法和非静态方法都可以是泛型的。类型推断的规则对所有类型都是相同的。类型参数必须在方法名之后、参数列表之前的尖括号内声明，就像我们对类型所做的那样。

让我们通过以下示例来了解如何使用泛型方法：

```cs
class CompareObjects
{
    public bool Compare<T>(T input1, T input2)
    {
        return input1.Equals(input2);
    }
}
```

非泛型类`CompareObjects`包含一个泛型方法`Compare`，用于比较两个对象。该方法接受两个参数——`input1`和`input2`。我们使用`System.Object`基类的`Equals()`方法来比较输入参数。该方法将根据输入是否相等返回一个布尔值。考虑下面的代码：

```cs
CompareObjects comps = new CompareObjects();
Console.WriteLine(comp.Compare<int>(10, 10));
Console.WriteLine(comp.Compare<double>(10.5, 10.8));
Console.WriteLine(comp.Compare<string>("a", "a"));
Console.WriteLine(comp.Compare<string>("a", "b"));
```

我们正在创建`CompareObjects`类的对象，并为各种数据类型调用`Compare()`方法。在这个例子中，类型参数是显式指定的。然而，编译器能够从参数中推断出来，因此可以省略，如下所示：

```cs
CompareObjects comp = new CompareObjects();
Console.WriteLine(comp.Compare(10, 10));
Console.WriteLine(comp.Compare(10.5, 10.8));
Console.WriteLine(comp.Compare("a", "a"));
Console.WriteLine(comp.Compare("a", "b"));
```

如果泛型方法具有与定义它的类、结构或接口的类型参数相同的类型参数，编译器会发出警告，因为方法类型参数隐藏了外部类型的类型参数，如下面的代码所示：

```cs
class ConflictingGenerics<T>
{
    public void DoSomething<T>(T arg) // warning
    { 
    }
}
```

泛型方法和泛型类型都支持类型参数约束来对类型施加限制。这个主题将在本章的下一节中讨论。

# 类型参数约束

泛型类型或方法中的类型参数可以被任何有效类型替换。然而，在某些情况下，我们希望限制可以用作类型参数的类型。例如，我们之前看到的泛型`Shape<T>`类或`IShape<T>`接口。

类型参数`T`被用于`Area`属性的类型。我们期望它要么是整数类型，要么是浮点类型。但是没有限制，有人可以使用`bool`，`string`或任何其他类型。当然，根据类型参数的使用方式，这可能导致各种编译错误。然而，能够限制用于实例化泛型类型或调用泛型方法的类型是有用的。

为此，我们可以对类型参数应用约束。约束用于告诉编译器类型参数必须具有什么样的能力。如果我们不指定约束，那么类型参数可以被任何类型替换。应用约束将限制可以用作类型参数的类型。

约束使用关键字`where`来指定。C#定义了以下八种泛型约束类型：

![](img/Chapter_6Table_1_01.jpg)

约束应该在类型参数之后指定。我们可以通过逗号分隔它们来使用多个约束。对于使用这些约束有一些规则：

+   `struct`约束意味着`new()`约束，因此所有值类型必须有一个公共的无参数构造函数。这两个约束，`struct`和`new()`，不能一起使用。

+   `unmanaged`约束意味着`struct`约束；因此，这两个不能一起使用。它也不能与`new()`约束一起使用。

+   在使用多个约束时，`new()`约束必须在约束列表中最后提及。

+   `notnull`约束从 C# 8 开始可用，必须在可空上下文中使用，否则编译器会生成警告。当约束被违反时，编译器不会生成错误，而是生成警告。

+   从 C# 7.3 开始，`System.Enum`，`System.Delegate`和`System.MulticastDelegate`可以用作基类约束。

没有约束的类型参数称为*无界*。无界类型参数有几条规则：

+   你不能使用`!=`和`==`运算符来处理这些类型，因为不可能知道具体类型是否重载了它们。

+   它们可以与`null`进行比较。对于值类型，这种比较总是返回`false`。

+   它们可以转换为和从`System.Object`。

+   它们可以转换为任何接口类型。

为了理解约束的工作原理，让我们从以下泛型结构的示例开始：

```cs
struct Point<T>
{
    public T X { get; }
    public T Y { get; }
    public Point(T x, T y)
    {
        X = x;
        Y = y;
    }
}
```

`Point<T>`是表示二维空间中的点的结构。这个类是泛型的，因为我们可能希望使用整数值作为点坐标或实数值（浮点值）。但是，我们可以使用任何类型来实例化该类，例如`bool`，`string`或`Circle`，如下例所示：

```cs
Point<int> p1 = new Point<int>(3, 4);
Point<double> p2 = new Point<double>(3.12, 4.55);
Point<bool> p3 = new Point<bool>(true, false);
Point<string> p4 = new Point<string>("alpha", "beta");
```

为了将`Point<T>`的实例化限制为数字类型（即整数和浮点类型），我们可以为类型参数`T`编写约束，如下所示：

```cs
struct Point<T>
    where T : struct, 
              IComparable, IComparable<T>,
              IConvertible,
              IEquatable<T>,
              IFormattable
{
    public T X { get; }
    public T Y { get; }
    public Point(T x, T y)
    {
        X = x;
        Y = y;
    }
}
```

我们使用了两种类型的约束：`struct`约束和接口约束，并且它们用逗号分隔列出。不幸的是，没有约束可以将类型定义为数字，但这些约束是表示数字类型的最佳组合，因为所有数字类型都是值类型，并且它们都实现了这里列出的五个接口。`bool`类型实现了前四个，但没有实现`IFormattable`。因此，使用`bool`或`string`实例化`Point<T>`现在将产生编译错误。

类型或方法可以有多个类型参数，每个类型参数都可以有自己的约束。我们可以在下面的示例中看到这一点：

```cs
class RestrictedDictionary<TKey, TValue> : Dictionary<TKey, List<TValue>>
    where TKey : System.Enum
    where TValue : class, new()
{
    public T Make<T>(TKey key) where T : TValue, new()
    {
        var value = new T();
        if (!TryGetValue(key, out List<TValue> list))
            Add(key, new List<TValue>() { value });
        else
            list.Add(value);
        return value;
    }
}
```

`RestrictedDictionary<TKey, TValue>`类是一个特殊的字典，它只允许枚举类型作为键类型。为此，它使用了基类约束`System.Enum`。值的类型必须是具有公共默认构造函数的引用类型。为此，它使用了`class`和`new()`约束。这个类有一个名为`Make<T>()`的公共泛型方法。

类型参数`T`必须是`TValue`或从`TValue`派生的类型，并且还必须具有公共默认构造函数。此方法创建类型`T`的新实例，将其添加到与指定键关联的字典中的列表中，并返回对新创建对象的引用。

让我们也考虑以下形状类的层次结构。请注意，为简单起见，这些被保持在最低限度：

```cs
enum ShapeType { Sharp, Rounded };
class Shape { }
class Ellipsis  : Shape { }
class Circle    : Shape { }
class Rectangle : Shape { }
class Square    : Shape { }
```

我们可以像这样使用`RestrictedDictionary`类：

```cs
var dictionary = new RestrictedDictionary<ShapeType, Shape>();
var c = dictionary.Make<Circle>(ShapeType.Rounded);
var e = dictionary.Make<Ellipsis>(ShapeType.Rounded);
var r = dictionary.Make<Rectangle>(ShapeType.Sharp);
var s = dictionary.Make<Square>(ShapeType.Sharp);
```

在这个例子中，我们将几种形状（圆形、椭圆形、矩形和正方形）添加到受限制的字典中。键类型是`ShapeType`，值类型是`Shape`。`Make()`方法接受`ShapeType`类型的参数，并返回对形状对象的引用。每种类型都必须派生自`Shape`并具有公共默认构造函数。否则，代码将产生错误。

# 总结

在本章中，我们学习了 C#中的泛型。泛型允许我们在 C#中创建参数化类型。泛型增强了代码的可重用性并确保类型安全。我们探讨了如何创建泛型类和泛型结构。我们还在泛型类中实现了继承。

我们学习了如何在泛型类型或方法的类型参数上实现约束。约束允许我们限制可以用作类型参数的数据类型。我们还学习了如何创建泛型方法和泛型接口。

您可以主要用于创建集合和包装的泛型。在下一章中，我们将探讨.NET 中最重要的集合。

# 测试你所学到的

1.  泛型是什么，它们提供了什么好处？

1.  什么是类型参数？

1.  如何定义泛型类？泛型方法呢？

1.  一个类可以从泛型类型派生吗？结构呢？

1.  什么是构造类型？

1.  泛型接口的协变类型参数是什么？

1.  泛型接口的逆变类型参数是什么？

1.  什么是类型参数约束，以及如何指定它们？

1.  `new()`类型参数约束是做什么的？

1.  C# 8 中引入了什么类型参数约束，它是做什么的？
