# 第三章：在 C#中实现面向对象编程

在前一章中，我们看了类、对象和面向对象编程的四个原则。在本章中，我们将学习一些使 C#语言成为面向对象编程语言的语言特性。如果不了解这些概念，使用 C#编程写面向对象的代码可能会很困难，或者会阻止你充分发挥其潜力。在第二章，*Hello OOP - Classes and Objects*中，我们学到了抽象、继承、封装和多态是面向对象编程的四个基本原则，但我们还没有学习 C#语言如何实现这些原则。我们将在本章讨论这个话题。

在本章中，我们将涵盖以下主题：

+   接口

+   抽象类

+   部分类

+   封闭类

+   元组

+   属性

+   类的访问修饰符

# 接口

类是一个蓝图，这意味着它包含了实例化对象将具有的成员和方法。**接口**也可以被归类为蓝图，但与类不同，接口没有任何方法实现。接口更像是实现接口的类的指南。

C#中接口的主要特点如下：

+   接口不能有方法体；它们只能有方法签名。

+   接口可以有方法、属性、事件和索引。

+   接口不能被实例化，因此不能创建接口的对象。

+   一个类可以扩展多个接口。

接口的一个主要用途是依赖注入。通过使用接口，可以减少系统中的依赖关系。让我们看一个接口的例子：

```cs
interface IBankAccount {
    void Debit(double amount);
    void Credit(double amount);
}
class BankAccount : IBankAccount {
    public void Debit(double amount){
        Console.WriteLine($"${amount} has been debited from your account!");
    } 
    public void Credit(double amount){
        Console.WriteLine($"${amount} has been credited to your account!");
    }
}
```

在前面的例子中，我们可以看到我们有一个接口，名为`IBankAccount`，它有两个成员：`Debit`和`Credit`。这两个方法在接口中没有实现。在接口中，方法签名更像是实现这个接口的类的指南或要求。如果任何类实现了这个接口，那么这个类必须实现方法体。这是面向对象编程概念继承的一个很好的用法。类将不得不给出在接口中提到的方法的实现。如果类没有实现接口的任何方法，编译器将抛出一个错误，表示类没有实现接口的所有方法。按照语言设计，如果一个类实现了一个接口，那么这个类的所有成员都必须在类中得到处理。因此，在前面的代码中，`BankAccount`类实现了`IBankAccount`接口，这就是为什么`Debit`和`Credit`这两个方法必须被实现的原因。

# 抽象类

**抽象类**是 C#编程语言中的一种特殊类。这个类与接口有类似的功能。例如，抽象类可以有带有和不带有实现的方法。因此，当一个类实现一个抽象类时，这个类必须重写抽象类的**抽象方法**。抽象类的一个主要特征是它不能被实例化。抽象类只能用于继承。它可能有也可能没有抽象方法和访问器。封闭和抽象修饰符不能放在同一个类中，因为它们有完全不同的含义。

让我们看一个抽象类的例子：

```cs
abstract class Animal {
    public string name;
    public int ageInMonths;
    public abstract void Move();
    public void Eat(){
        Console.WriteLine("Eating");
    }
}
class Dog : Animal {
    public override void Move() {
        Console.WriteLine("Moving");
    }
} 
```

在前面的例子中，我们看到`Dog`类实现了`Animal`类，而`Animal`类有一个名为`Move()`的抽象方法，`Dog`类必须重写它。

如果我们尝试实例化抽象类，编译器将抛出一个错误，如下所示：

```cs
using System;
namespace AnimalProject {
    abstract class Animal {
        public string name;
        public int ageInMonths;
        public abstract void Move();
        public void Eat(){
            Console.WriteLine("Eating");
        }
    }
    static void Main(){
        Animal animal = new Animal(); // Not possible as the Animal class is abstract class
```

```cs
    }
}
```

# 部分类

您可以将一个类、结构体或接口分割成可以放在不同代码文件中的较小部分。如果要这样做，必须使用关键字**partial**。即使代码在单独的代码文件中，编译时它们将被视为一个整体类。部分类有许多好处。一个好处是不同的开发人员可以同时在不同的代码文件上工作。另一个好处是，如果您正在使用自动生成的代码，并且想要扩展该自动生成的代码的某些功能，可以在单独的文件中使用部分类。因此，您不是直接触及自动生成的代码，而是在类中添加新功能。

部分类有一些要求，其中之一是所有类必须在其签名中有关键字`partial`。所有部分类还必须具有相同的名称，但文件名可以不同。部分类还必须具有相同的可访问性，如 public、private 等。

以下是部分类的示例：

```cs
// File name: Animal.cs
using System;
namespace AnimalProject {
    public partial class Animal {
        public string name;
        public int ageInMonths;

        public void Eat(){
            Console.WriteLine("Eating");
        }
     }
}
// File name: AnimalMoving.cs
using System;
namespace AnimalProject {
    public partial class Animal {

        public void Move(){
            Console.WriteLine("Moving");
        }
    }
}
```

如前面的代码所示，您可以创建一个类的许多部分类。这将增加代码的可读性，使代码组织更加结构化。

# 密封类

面向对象编程的原则之一是继承，但有时您可能需要限制代码中的继承，以符合应用程序的架构。C#提供了一个名为`sealed`的关键字。如果在类的签名之前放置这个关键字，该类被视为**密封类**。如果一个类是密封的，那个特定的类就不能被其他类继承。如果任何类尝试继承一个密封类，编译器将抛出一个错误。结构体也可以是密封的，在这种情况下，没有类可以继承该结构体。

让我们看一个密封类的示例：

```cs
sealed class Animal {
    public string name;
    public int ageInMonths;
    public void Move(){
        Console.WriteLine("Moving");
    }
    public void Eat(){
        Console.WriteLine("Eating");
    }
}
public static void Main(){
    Animal dog = new Animal();
    dog.name = "Doggy";
    dog.ageInMonths = 1;

    dog.Move();
    dog.Eat();
}
```

在前面的示例中，我们可以看到如何创建一个密封类。只需在`class`关键字之前使用`sealed`关键字即可使类成为密封类。在前面的示例中，我们创建了一个`Animal`密封类，在`main`方法中，我们实例化了该类并使用了它。现在一切都运行正常。然而，如果我们尝试创建一个将继承`Animal`类的`Dog`类，如下面的代码所示，那么编译器将抛出一个错误，说密封的`Animal`类不能被继承：

```cs
class Dog : Animal {
    public char gender;
}
```

这是编译器将显示的屏幕截图：

![](img/6f52ff92-f316-4553-b8e9-d224d8e367c9.png)

# 元组

**元组**是一种保存一组数据的数据结构。当您想要对数据进行分组和使用时，元组通常很有帮助。通常，C#方法只能返回一个值。通过使用元组，可以从方法中返回多个值。`Tuple`类位于`System.Tuple`命名空间下。可以使用`Tuple<>`构造函数或`Tuple`类附带的名为`Create`的抽象方法来创建元组。

您可以固定元组中的任何数据类型，并使用`Item1`、`Item2`等进行访问。让我们看一个例子，以更好地理解这一点：

```cs
var person = new Tuple<string, int, string>("Martin Dew", 42, "Software Developer"); // name, age, occupation
or 
var person = new Tuple.Create("Martin Dew", 42, "Software Developer");
```

让我们看看如何通过以下代码从方法中返回一个元组：

```cs
public static Tuple<string, int, string> GetPerson() {
    var person = new Tuple<string, int, string>("Martin Dew", 42, "Software Developer");
    return person;
}
static void Main() {
    var developer = GetPerson();
    Console.WriteLine("The person is {0}. He is {1} years old. He is a {2}", developer.Item1, developer.Item2, developer.Item3 );
}
```

# 属性

出于安全原因，类的所有字段不应该暴露给外部世界。因此，在 C#中通过属性来暴露私有字段，这些属性是该类的成员。属性下面是称为**访问器**的特殊方法。属性包含两个访问器：`get`和`set`。`get`访问器从字段获取值，而`set`访问器将值设置到字段。属性有一个特殊的关键字，名为`value`。这代表了字段的值。

通过使用访问修饰符，属性可以具有不同的访问级别。属性可以是 `public`、`private`、`read only`、`open for read and write` 和 `write only`。如果只实现了 `set` 访问器，这意味着只有写入权限。如果同时实现了 `set` 和 `get` 访问器，这意味着该属性对读和写都是开放的。

C# 提供了一种聪明的方式来编写 `setter` 和 `getter` 方法。如果你在 C# 中创建一个属性，你不需要为特定字段手动编写 `setter` 和 `getter` 方法。因此，在 C# 中的常见做法是在类中创建属性，而不是为这些字段创建字段和 `setter` 和 `getter` 方法。

让我们看看如何在 C# 中创建属性，如下面的代码所示：

```cs
class Animal {
    public string Name {set; get;}
    public int Age {set; get;}
}
```

`Animal` 类有两个属性：`Name` 和 `Age`。这两个属性都有 `Public` 访问修饰符以及 `setter` 和 `getter` 方法。这意味着这两个属性都对读和写操作是开放的。约定是属性应该使用驼峰命名法。

如果你想修改你的 `set` 和 `get` 方法，你可以这样做：

```cs
class Animal {
    public string Name {
        set {
            name = value;
        }
        get {
            return name;
        }
    }
    public int Age {set; get;}
}
```

在上面的例子中，我们没有使用为 `Name` 属性创建 `setter` 和 `getter` 的快捷方式。我们广泛地写了 `set` 和 `get` 方法应该做什么。如果你仔细看，你会看到 `name` 字段是小写的。这意味着当你使用驼峰命名法创建属性时，一个同名的字段会在内部创建，但是是以帕斯卡命名法。`value` 是一个特殊关键字，实际上代表了该属性的值。

属性在后台工作，这使得代码更加清晰和易于使用。强烈建议您使用属性而不是本地字段。

# 类的访问修饰符

**访问修饰符**，或者**访问修饰符**，是一些保留关键字，用于确定类、方法、属性或其他实体的可访问性。在 C# 中，使用这些访问修饰符实现了面向对象的封装原则。总共有五个访问修饰符。让我们看看这些是什么，它们之间的区别是什么。

# 公共

**公共**访问修饰符意味着对正在修改的实体没有限制。如果将类或成员设置为 `public`，则可以被同一程序集中的其他类或程序、其他程序集甚至安装在运行该程序的操作系统中的其他程序访问。通常，应用程序的起点或主方法被设置为 `public`，这意味着它可以被其他人访问。要使类为 `public`，只需在关键字 class 前面放置一个 `public` 关键字，如下面的代码所示：

```cs
public class Animal {
}
```

上述的 `Animal` 类可以被任何其他类访问，而且由于成员 `Name` 也是公共的，它也可以从任何位置访问。

# 私有

**私有**修饰符是 C# 编程语言中最安全的访问修饰符。通过将类或类的成员设置为 `private`，你确定该类或成员将不允许其他类访问。`private` 成员的范围在类内。例如，如果你创建一个 `private` 字段，那个字段就不能在类外部被访问。那个 `private` 字段只能在该类内部使用。

让我们看一个带有 `private` 字段的类的例子：

```cs
public class Animal {
    private string name;
    public string GetName() {
        return name;
    }
}
```

在这里，由于 `GetName()` 方法和 `private` 字段 `name` 在同一个类中，该方法可以访问该字段。但是，如果 `Animal` 类之外的另一个方法尝试访问 `name` 字段，它将无法访问。

例如，在以下代码中，`Main` 方法正在尝试设置 `private` 字段 name，这是不允许的：

```cs
using System;
namespace AnimalProject {
    static void Main(){
        Animal animal = new Animal();
        animal.name = "Dog"; // Not possible, as the name field is private
        animal.GetName(); // Possible, as the GetName method is public
    }
}
```

# 内部

如果将`internal`设置为访问限定符，这意味着该实体只能在同一程序集内访问。程序集中的所有类都可以访问该类或成员。在.NET 中构建项目时，它会创建一个程序集文件，可以是`dll`或`exe`。一个解决方案中可能有多个程序集，而内部成员只能被那些特定程序集中的类访问。

让我们看一个示例，如下所示的代码：

```cs
using System;
namespage AnimalProject {
    static void Main(){
        Dog dog = new Dog();
        dog.GetName();
    }

    internal class Dog {
        internal string GetName(){
            return "doggy";
        }
    }
}
```

# 受保护的

受保护的成员可以被类本身访问，以及继承该类的子类。除此之外，没有其他类可以访问受保护的成员。受保护的访问修饰符在继承发生时非常有用。

让我们通过以下代码来学习如何使用这个：

```cs
using System;
namespage AnimalProject {
    static void Main(){
        Animal animal = new Animal();
        Dog dog = new Dog();
        animal.GetName(); // Not possible as Main is not a child of Animal
        dog.GetDogName();
    }

    class Animal {
        protected string GetName(){
            return "doggy";
        }
    }
    class Dog : Animal {
        public string GetDogName() {
            return base.GetName();
        }
    }
}
```

# 受保护的内部

**受保护的内部**是受保护的访问修饰符和内部访问修饰符的组合。其访问修饰符为`protected internal`的成员可以被同一程序集中的所有类访问，以及任何继承它的类，无论程序集如何。例如，假设您在名为`Assembly1.dll`的程序集中有一个名为`Animal`的类。在`Animal`类中，有一个受保护的内部方法叫做`GetName`。`Assembly1.dll`中的任何其他类都可以访问`GetName`方法。现在，假设还有另一个名为`Assembly2.dll`的程序集。在`Assembly2.dll`中，有一个名为`Dog`的类，它扩展了`Animal`类。由于`GetName`是受保护的内部，即使`Dog`类在一个单独的程序集中，它仍然可以访问`GetName`方法。

让我们通过以下示例来更清楚地理解这一点：

```cs
//Assembly1.dll
using System;
namespace AnimalProject {
    public class Animal {
        protected internal string GetName(){
            return "Nice Animal";
        }
    }
}
//Assembly2.dll
using System;
namespace AnimalProject2 {
    public class Dog : Animal {
        public string GetDogName(){
            return base.GetName(); // This will work
        }
    }
    public class Cat {
        Animal animal = new Animal();

        public string GetCatName(){
            return animal.GetName(); // This is not possible, as GetName is protected internal
        }
    }
}
```

# 总结

在本章中，我们看了类层次结构和一些其他特性，使 C#编程语言成为面向对象的语言。了解这些概念对于 C#开发人员至关重要。通过了解类层次结构，您可以设计系统，使其解耦且灵活。您需要知道如何在应用程序中使用继承来充分发挥面向对象的优势。接口、抽象类、密封类和部分类将帮助您更好地控制应用程序。在团队中工作时，正确定义类层次结构将有助于您维护代码质量和安全性。

了解元组和属性将提高您的代码清晰度，并在开发应用程序时使您的生活更加轻松。访问限定符是封装的面向对象编程概念的实现。熟悉这些概念非常重要。您需要知道哪些代码片段应该是公开的，哪些应该是私有的，哪些应该是受保护的。如果滥用这些访问限定符，您可能会陷入应用程序存在安全漏洞和代码重复的境地。

在下一章中，我们将讨论对象协作的重要和有趣的主题。
