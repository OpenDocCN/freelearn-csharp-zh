# 第七章：C#中的泛型

泛型是 C#编程语言中非常重要的一个主题。据我所知，很难找到任何不使用泛型的 C#编写的现代软件。

本章中我们将涵盖的主题如下：

+   什么是泛型？

+   我们为什么需要泛型？

+   泛型的不同约束

+   泛型方法

+   泛型中的协变和逆变

# 什么是泛型？

在 C#中，泛型用于创建不特定但通用的类、方法、结构和其他组件。这使我们能够为不同的原因使用通用组件。例如，如果您有一种通用的肥皂，您可以用它来进行任何类型的清洗。您可以用它来洗手，洗衣服，甚至洗脏碗。但是，如果您有一种特定类别的肥皂，比如洗衣粉，它只能用来洗衣服，而不能用来做其他事情。因此，泛型为我们的代码提供了一些额外的可重用性，这对于应用程序是有益的，因为会有更少的代码来执行类似的工作。泛型并不是新开发的；它们自 C# 2 以来就已经可用。因此，经过这么多年的使用，泛型已经成为程序员常用的工具。

让我们来看一个`Generic`类的例子：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Chapter7
{
  class Price<T>
  {
    T ob;

    public Price(T o)
    {
      ob = o;
    }

    public void PrintType()
    {
      Console.WriteLine("The type is " + typeof(T));
    }

    public T GetPrice()
    {
      return ob;
    }
  }

  class Code_7_1
  {
    static void Main(string[] args)
    {
      Price<int> price = new Price<int>(55);

      price.PrintType();

      int a = price.GetPrice();

      Console.WriteLine("The price is " + a);

      Console.ReadKey();
    }
  }
}
```

前面代码的输出如下：

![](img/140f13d4-7a65-4ca8-aec5-3d936a425256.png)

如果您对泛型的语法完全不熟悉，您可能会对在`Price`类旁边看到的尖括号`<>`感到惊讶。您可能还想知道`<>`中的`T`是什么。这是 C#中泛型的语法。通过将`<>`放在类名旁边，我们告诉编译器这是一个泛型类。此外，`<>`中的`T`是一个类型参数。是的，我知道您在问：“'什么是类型参数？'”**类型参数**就像 C#编程中的任何其他参数一样，只是它传递的是类型而不是值或引用。现在，让我们分析前面的代码。

我们创建了一个泛型`Price`类。为了使它成为泛型，我们在类名旁边放置了`<T>`。这里，`T`是一个类型参数，但它并不是固定的，您可以使用任何东西来表示类型参数，而不一定非要使用`T`。但是，传统上使用`T`来表示类型参数。如果有更多的类型参数，会使用`V`和`E`。在使用两个或更多参数时，还有另一种常用的约定，即将参数命名为`TValue`和`TKey`，而不仅仅是`V`和`E`，这样做可以提高可读性。但是，正如您所看到的，我们在`Value`和`Key`之前加了`T`前缀，这是为了区分类型参数和一般参数。

在`Price<T>`类中，我们首先创建了一个名为`ob`的变量，它是`T`类型的：

```cs
T ob;
```

当我们运行前面的代码时，我们在类中传递的类型将是这个对象的类型。因此，我们可以说`T`是一个占位符，在运行时将被一些其他具体的 C#类型（`int`、`double`、`string`或任何其他复杂类型）替换。

在接下来的几行中，我们创建了一个构造函数：

```cs
public Price(T o)
{
    ob = o;
}
```

在构造函数中，我们传递了一个`T`类型的参数，然后将传递的参数`o`的值分配给局部变量`ob`。我们可以这样做是因为在构造函数中传递的参数也是`T`类型。

然后，我们创建了第二个方法：

```cs
public void PrintType()
{
    Console.WriteLine("The type is " + typeof(T));
}

public T GetPrice()
{
    return ob;
}
```

这里，第一个方法打印`T`的类型。这将有助于在运行程序时识别类型。另一个方法是返回局部变量`ob`。在这里，我们注意到我们从`GetPrice`方法中返回了`T`。

现在，如果我们专注于我们的主方法，我们会看到在第一行中我们正在用`int`作为类型参数实例化我们的泛型类`Price`，并将整数值`55`传递给构造函数：

```cs
Price<int> price = new Price<int>(55);
```

当我们这样做时，编译器将`Price`类中的每个`T`视为`int`。因此，局部参数`ob`将是`int`类型。当我们运行`PrintType`方法时，应该在屏幕上打印 System.Int32，当我们运行`GetPrice`方法时，应该返回一个`Int`类型的值。

现在，由于`Price`方法是泛型的，我们也可以将此`Price`方法用于字符串类型。为此，我们必须将类型参数设置为`string`。让我们在前面的例子中添加一些代码，这将创建一个处理字符串的`Price`对象：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Chapter7
{
  class Price<T>
  {
    T ob;

    public Price(T o)
    {
      ob = o;
    }

    public void PrintType()
    {
      Console.WriteLine("The type is " + typeof(T));
    }

    public T GetPrice()
    {
      return ob;
    }
  }

  class Code_7_2
  {
    static void Main(string[] args)
    {
      Price<int> price = new Price<int>(55);

      price.PrintType();

      int a = price.GetPrice();

      Console.WriteLine("the price is " + a);

      Price<string> priceStr = new Price<string>("Hello People");

      priceStr.PrintType();

      string b = priceStr.GetPrice();

      Console.WriteLine("the string is " + b);

      Console.ReadKey();
    }
  }
}
```

上述代码的输出如下：

![](img/6dc6301a-d676-4bdc-8179-1c371c2e0321.png)

# 我们为什么需要泛型？

看到前面的例子后，您可能会想知道为什么我们需要泛型，当我们可以使用`object`类型时。`object`类型可以用于 C#中的任何类型，并且可以通过使用`object`类型实现前面的例子。是的，可以通过使用对象类型实现前面的例子，但不会有类型安全。相反，泛型确保了在代码执行时存在类型安全。

如果你和我一样，肯定想知道什么是类型安全。**类型安全**实际上是指在程序执行任何任务时保持类型安全或不可更改。这有助于减少运行时错误。

现在，让我们使用对象类型而不是泛型来编写前面的程序，看看泛型如何处理类型安全，而对象类型无法处理：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Chapter7
{
  class Price
  {
    object ob;

    public Price(object o)
    {
      ob = o;
    }

    public void PrintType()
    {
      Console.WriteLine("The type is " + ob.GetType());
    }

    public object GetPrice()
    {
      return ob;
    }
  }

  class Code_7_3
  {
    static void Main(string[] args)
    {
      Price price = new Price(55);

      price.PrintType();

      int a = (int)price.GetPrice();

      Console.WriteLine("the price is " + a);

      Console.ReadKey();
    }
  }
}
```

上述代码的输出如下：

![](img/1a223402-1e51-43ed-97e4-ea3c0aea78f4.png)

# 泛型的不同约束

在 C#泛型中有不同类型的约束：

+   基类约束

+   接口约束

+   引用类型和值类型约束

+   多个约束

最常见和流行的类型是基类约束和接口约束，因此我们将在以下部分重点关注它们。

# 基类约束

这种约束的想法是只有扩展基类的类才能用作泛型类型。例如，如果您有一个名为`Person`的类，并且将此`Person`类用作`Generic`约束的基类，那么只有`Person`类或继承`Person`类的任何其他类才能用作该泛型类的类型参数。让我们看一个例子：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Chapter7
{
  public class Person
  {
    public void PrintName()
    {
      Console.WriteLine("My name is Raihan");
    }
  }

  public class Boy : Person
  {

  }

  public class Toy
  {

  }

  public class Human<T> where T : Person
  {
    T obj;

    public Human(T o)
    {
      obj = o;
    }

    public void MustPrint()
    {
      obj.PrintName();
    }
  }

  class Code_7_3
  {
    static void Main(string[] args)
    {
      Person person = new Person();
      Boy boy = new Boy();
      Toy toy = new Toy();

      Human<Person> personTypeHuman = new Human<Person>(person);
      personTypeHuman.MustPrint();

      Human<Boy> boyTypeHuman = new Human<Boy>(boy);
      boyTypeHuman.MustPrint();

      /* Not allowed
      Human<Toy> toyTypeHuman = new Human<Toy>(toy);
      toyTypeHuman.MustPrint();
      */

      Console.ReadKey();
    }
  }
}
```

# 接口约束

与基类约束类似，当您的泛型类约束设置为接口时，我们看到接口约束。只有实现该接口的类才能在泛型方法中使用。

# 引用类型和值类型约束

当您想要区分泛型类和引用类型和值类型时，您需要使用此约束。当您使用引用类型约束时，泛型类将只接受引用类型对象。为了实现这一点，您必须使用`class`关键字扩展您的泛型类：

```cs
... where T : class
```

此外，当您想要使用值类型时，您需要编写以下代码：

```cs
... where T : struct
```

正如我们所知，`class`是引用类型，`struct`是值类型。因此，当您设置值类型约束时，这意味着泛型只能用于值类型，如`int`或`double`。不会有任何引用类型，如字符串或任何其他自定义类。

# 多个约束

在 C#中，可以在泛型类中使用多个约束。当这样做时，需要注意顺序。实际上，您可以包含多少约束都没有限制；您可以使用您需要的多少个。

# 泛型方法

像`Generic`类一样，可以有泛型方法，泛型方法不一定要在泛型类中。泛型方法也可以在非泛型类中。要创建泛型方法，必须在方法名之后和括号之前放置类型参数。一般形式如下：

```cs
access-modifier return-type method-name<type-parameter>(params){ method-body }
```

现在，让我们看一个泛型方法的例子：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Chapter7
{
  class Hello
  {
    public static T Larger<T>(T a, T b) where T : IComparable<T>
    {
      return a.CompareTo(b) > 0 ? a : b;
    }
  }

  class Code_7_4
  {
    static void Main(string[] args)
    {
      int result = Hello.Larger<int>(3, 4);

      double doubleResult = Hello.Larger<double>(4.3, 5.6);

      Console.WriteLine("The Large value is " + result);
      Console.WriteLine("The Double Large value is " + doubleResult);

      Console.ReadKey();
    }
  }
}
```

上述代码的输出如下：

![](img/e51e658f-c3b2-4124-afc4-89e61c35953d.png)

在这里，我们可以看到我们的`Hello`类不是一个泛型类。然而，`Larger`方法是一个泛型方法。这个方法接受两个参数并比较它们，返回较大的值。这个方法还实现了一个约束，即`IComparable<T>`。在主方法中，我们多次调用了这个泛型方法，一次使用`int`值，一次使用`double`值。在输出中，我们可以看到该方法成功地比较并返回了较大的值。

在这个例子中，我们只使用了一种类型的参数，但是在泛型方法中可以有多个参数。在这个示例代码中，我们还创建了一个`static`方法，但是泛型方法也可以是非静态的。静态/非静态与是否为泛型方法无关。

# 类型推断

编译器变得更加智能。一个例子就是泛型方法中的类型推断。**类型推断**意味着调用泛型方法而不指定类型参数，并让编译器确定使用哪种类型。这意味着在前面的例子中，当调用方法时，我们无法指定类型参数。

让我们看一些类型推断的示例代码：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Chapter7
{
  class Hello
  {
    public static T Larger<T>(T a, T b) where T : IComparable<T>
    {
      return a.CompareTo(b) > 0 ? a : b;
    }
  }

  class Code_7_5
  {
    static void Main(string[] args)
    {
      int result = Hello.Larger(3, 4);

      double doubleResult = Hello.Larger(4.3, 5.6);

      Console.WriteLine("The Large value is " + result);
      Console.WriteLine("The Double Large value is " + doubleResult);

      Console.ReadKey();
    }
  }
}
```

上述代码的输出如下：

![](img/465f1489-acd9-43d6-aaa3-d0ad0c2c2728.png)

在这段代码中，我们可以看到在泛型方法中没有指定类型参数。然而，代码仍然编译并显示正确的输出。这是因为编译器使用类型推断来确定传递给方法的参数类型，并执行方法，就好像参数类型已经给编译器了。因此，当使用类型推断时，不允许在泛型方法中提供不同类型的参数。如果需要传递不同类型的参数，应该明确指定。也可以对可以应用于类的方法应用约束。

# 泛型中的协变和逆变

如果你学过委托，我相信你一定听说过协变和逆变。这些主要是为非泛型委托引入的。然而，从 C# 4 开始，这些也适用于泛型接口和委托。泛型中的协变和逆变概念几乎与委托中的相同。让我们通过示例来看一下。

# 协变

这意味着具有`T`类型参数的通用接口可以返回`T`或任何派生自`T`的类。为了实现这一点，参数应该与`out`关键字一起使用。让我们看看通用形式：

```cs
access-modifier interface-name<out T>{}
```

# 逆变

逆变是泛型中实现的另一个特性。"逆变"这个词听起来可能有点复杂，但其背后的概念非常简单。通常，在创建泛型方法时，我们传递给它的参数与`T`的类型相同。如果尝试传递另一种类型的参数，将会得到编译时错误。然而，使用逆变时，可以传递类型参数实现的基类。此外，要使用逆变，我们必须遵循一种特殊的语法。让我们看看泛型语法：

```cs
access-modifier interface interface-name<in T>{}
```

如果分析上述语句，会发现在`T`之前使用了一个关键字，即`in`。这个关键字告诉编译器这是逆变。如果不包括`in`关键字，逆变将不适用。

现在，让我们看一些示例代码，以便更清楚地理解我们的理解：

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Chapter7
{
  public interface IFood<in T>
  {
    void PrintMyName(T obj);
  }

  class HealthyFood<T> : IFood<T>
  {
    public void PrintMyName(T obj)
    {
      Console.WriteLine("This is " + obj);
    }
  }

  class Vegetable
  {
    public override string ToString()
    {
      return "Vegetable";
    }
  }

  class Potato : Vegetable
  {
    public override string ToString()
    {
      return "Potato";
    }
  }

  class Code_7_6
  {
    static void Main(string[] args)
    {
      IFood<Potato> mySelf = new HealthyFood<Potato>();
      IFood<Potato> mySelf2 = new HealthyFood<Vegetable>();

      mySelf2.PrintMyName(new Potato());

      Console.ReadKey();
    }
  }
}
```

上述代码的输出如下：

![](img/74ef526e-0505-4898-9e7e-e1df617488ad.png)

如果现在分析这段代码，会发现我们创建了一个名为`IFood`的接口，它使用了逆变。这意味着如果这个接口在一个泛型类中实现，该类将允许提供的类型参数的**基类**。

`IFood`接口有一个方法签名：

```cs
void PrintMyName(T obj);
```

这里，`T`被用作方法的参数。

现在，一个名为`HealthyFood`的类实现了接口，而类中实现的方法只打印一个字符串：

```cs
class HealthyFood<T> : IFood<T>
{
  public void PrintMyName(T obj)
  {
    Console.WriteLine("This is " + obj);
  }
}
```

然后，我们创建了两个类：`Vegetable`和`Potato`。`Potato`扩展`Vegetable`。两个类都重写了`ToString()`方法，并且如果类是`Potato`，则返回`Potato`，如果类是`Vegetable`，则返回`Vegetable`。

在主方法中，我们创建了一个`Potato`类的对象和一个`Vegetable`类的对象。这两个对象都保存在`IFood<Potato>`变量中：

```cs
IFood<Potato> mySelf = new HealthyFood<Potato>();
IFood<Potato> mySelf2 = new HealthyFood<Vegetable>();
```

有趣的部分在于`mySelf2`变量是`IFood<Potato>`类型，但它持有`HealthyFood<Vegetable>`类型的对象。这只有因为逆变性才可能。

请查看以下语句：

```cs
mySelf2.PrintMyName(new Potato());
```

当我们执行它时，可以看到输出如下：

```cs
This is Potato
```

如果删除`in`关键字并尝试再次运行程序，您将失败，并且编译器将抛出错误，表示这是不可能的。之所以能够运行代码，仅仅是因为逆变性。

# 摘要

C#中的泛型是一个非常强大的功能，它减少了代码重复，使程序更加结构化，并提供了可扩展性。一些重要的数据结构是基于泛型概念创建的；例如，List（集合）是 C#中的一种泛型类型。这是现代开发中最常用的数据结构之一。

在下一章中，我们将学习如何使用图表来设计和建模我们的软件，以便更好地进行沟通。在开发软件时，如果软件设计没有清晰地传达给开发人员，那么软件很可能无法达到其建立的目的。因此，理解重要的模型和图表非常重要。
