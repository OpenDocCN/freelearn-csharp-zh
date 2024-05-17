# 第七章：第 7 天 - 用 C#理解面向对象编程

今天是我们七天学习系列的第七天。昨天（第六天），我们学习了一些高级主题，讨论了属性、泛型和 LINQ。今天，我们将开始学习使用 C#的面向对象编程（OOP）。

这将是一个实际的面向对象编程方法，同时涵盖所有方面。即使没有任何面向对象编程的基础知识，您也将受益，并且可以自信地在工作场所轻松地进行实践。

我们将涵盖以下主题：

+   面向对象编程介绍

+   讨论对象关系

+   封装

+   抽象

+   继承

+   多态

# 面向对象编程介绍

OOP 是纯粹基于对象的编程范式之一。这些对象包含数据（请参考第七天了解更多细节）。

当我们对编程语言进行分类时，称之为编程范式。有关更多信息，请参阅[`en.wikipedia.org/wiki/Programming_paradigm`](https://en.wikipedia.org/wiki/Programming_paradigm)。

OOP 已经被考虑来克服早期编程方法的局限性（考虑过程语言方法）。

一般来说，我将 OOP 定义如下：

*一种现代编程语言，我们使用对象作为构建应用程序的基本组件。*

我们周围有很多对象的例子，在现实世界中，我们有各种方面是对象的代表。让我们回到我们的编程世界，思考一个定义如下的程序：

*程序是一系列指令，指示语言编译器要做什么。*

为了更深入地理解 OOP，我们应该了解早期的编程方法，主要是过程式编程、结构化编程等。

+   结构化编程：这是由艾兹格·W·迪科斯特拉在 1966 年创造的一个术语。结构化编程是一种编程范式，用于解决处理 1000 行代码并将其分成小部分的问题。这些小部分通常被称为子例程、块结构、`for`和`while`循环等。使用结构化编程技术的已知语言包括 ALGOL、Pascal、PL/I 等。

+   过程式编程：这是从结构化编程派生出来的一种范式，简单地基于我们如何进行调用（称为过程调用）。使用过程式编程技术的已知语言包括 COBOL、Pascal、C。Go 编程语言的一个最新例子于 2009 年发布。

这两种方法的主要问题是，一旦程序增长，程序就变得难以管理。具有更复杂和庞大代码库的程序使这两种方法变得紧张。简而言之，使用这两种方法会使代码的可维护性变得繁琐。为了克服这些问题，现在我们有了 OOP，它具有以下特点：

+   继承

+   封装

+   多态

+   抽象

# 讨论对象关系

在我们开始讨论 OOP 之前，首先我们应该了解关系。在现实世界中，对象之间有关系，也有层次结构。面向对象编程中有以下类型的关系：

+   关联：关联表示对象之间的关系，所有对象都有自己的生命周期。在关联中，这些对象没有所有者。例如，会议中的人。在这里，人和会议是独立的；它们没有父级。一个人可以参加多个会议，一个会议可以组合多个人。会议和人都是独立初始化和销毁的。

聚合和组合都是关联的类型。

+   **聚合**：聚合是一种特殊形式的关联。与关联类似，对象在聚合中有自己的生命周期，但它涉及所有权，这意味着子对象不能属于另一个父对象。聚合是一种单向关系，对象的生命周期彼此独立。例如，子对象和父对象的关系是一种聚合，因为每个子对象都有父对象，但并不是每个父对象都有子对象。

+   **组合**：组合是一种*死亡*关系，表示两个对象之间的关系，一个对象（子对象）依赖于另一个对象（父对象）。如果父对象被删除，所有的子对象都会自动被删除。例如，一个房子和一个房间。一个房子有多个房间。但一个房间不能属于多个房子。如果我们拆毁了房子，房间会自动删除。

在接下来的部分，我们将详细讨论面向对象编程的所有特性。此外，我们将了解如何使用 C#实现这些特性。

# 继承

继承是面向对象编程中最重要的特性/概念之一。它在名称上是不言自明的；继承从一个类继承特性。简而言之，继承是一个在编译时执行的活动，通过语法的帮助进行指示。继承另一个类的类称为子类或派生类，被继承的类称为基类或父类。在这里，派生类继承基类的所有特性，无论是实现还是重写。

在接下来的部分，我们将使用 C#的代码示例详细讨论继承。

# 理解继承

继承作为面向对象编程的一个特性，帮助您定义一个子类。这个子类继承了父类或基类的行为。

继承一个类意味着重用这个类。在 C#中，继承是用冒号（*:*）符号来象征性地定义的。

修饰符（参考第二章，*第 02 天-开始使用 C#*）告诉我们基类对派生类的重用范围。例如，考虑类*B*继承类*A*。在这里，类*B*包含类*A 的所有特性，包括它自己的特性。请参考以下图表：

![](img/00102.gif)

在上图中，派生类（即*B*）通过忽略修饰符继承了所有特性。特性的继承无论是公共的还是私有的。这些修饰符在这些特性要被实现时才会考虑。在实现时只有公共特性才会被考虑。所以，在这里，公共特性，即*A*、*B*和*C*将被实现，但私有特性，即*B*将不会被实现。

# 继承的类型

直到这一点，我们已经了解了继承的概念。现在，是时候讨论继承的类型了；继承有以下几种类型：

+   **单一继承：**

这是一种广泛使用的继承类型。单一继承是指一个类继承另一个类。继承另一个类的类称为子类，被继承的类称为父类或基类。在子类中，类只从一个父类继承特性。

C#只支持单一继承。

在下一节中，您可以按层次继承类（正如我们将在下一节中看到的），但这是派生类的自然单一继承。请参考以下图表：

![](img/00103.gif)

前面的图表是单一继承的表示，显示*Class B*（继承类）继承*Class A*（基类）。*Class B*可以重用所有特性，即*A*、*B*和*C*，包括它自己的特性，即 D。继承中成员的可见性或可重用性取决于保护级别（这将在接下来的部分“继承中的成员可见性”中讨论）。

+   **多重继承：**

多重继承发生在派生类继承多个基类时。诸如 C++之类的语言支持多重继承。C#不支持多重继承，但我们可以借助接口实现多重继承。如果您想知道为什么 C#不支持多重继承，请参考官方链接[`blogs.msdn.microsoft.com/csharpfaq/2004/03/07/why-doesnt-c-support-multiple-inheritance/`](https://blogs.msdn.microsoft.com/csharpfaq/2004/03/07/why-doesnt-c-support-multiple-inheritance/)。参考以下图表：

![](img/00104.jpeg)

前面的图表是多重继承的表示（在没有接口帮助的情况下在 C#中不可能），显示了*Class C*（派生类）从两个基类（*A*和*B*）继承。在多重继承中，派生的*Class C*将拥有*Class A*和*Class B*的所有特性。

+   **层次继承：**

当超过一个类从一个类继承时，层次继承发生。参考以下图表：

![](img/00105.jpeg)

在前面的图表中，*Class B*（派生类）和*Class C*（派生类）从*Class A*（基类）继承。借助层次继承，*Class B*可以使用*Class A*的所有特性。同样，*Class C*也可以使用*Class A*的所有特性。

+   **多级继承：**

当一个类从已经是派生类的类派生时，称为多级继承。

在多级继承中，最近派生的类拥有所有先前派生类的特性。

在这种情况下，派生类可以有其父类和父类的父类。参考以下图表：

![](img/00106.gif)

前面的图表表示多级继承，并显示*Class C*（最近派生的类）可以重用*Class B*和*Class A*的所有特性。

+   **混合继承：**

混合继承是多重继承的组合。

C#不支持混合继承。

多重和多级继承的组合是层次继承，其中一个父类是一个派生类，最近派生的类继承多个父类。还可以有更多的组合。参考以下图表：

![](img/00107.gif)

前面的图像代表混合继承，显示了层次和多重继承的组合。您可以看到*Class A*是一个父类，所有其他类都直接或间接地从*Class A*派生而来。我们的派生*Class E*可以重用*Class A*、*B*、*C*和*D*类的所有特性。

+   **隐式继承：**

.NET 中的所有类型都隐式继承自`system.object`或其派生类。有关隐式继承的更多信息，请参考[`docs.microsoft.com/en-us/dotnet/csharp/tutorials/inheritance#implicit-inheritance`](https://docs.microsoft.com/en-us/dotnet/csharp/tutorials/inheritance#implicit-inheritance)。

# 继承中的成员可见性

正如我们之前讨论的，在继承中，派生类可以重用父类的功能并使用或修改其父类的成员。但是这些成员可以根据其访问修饰符或可见性进行重用或修改（有关更多详细信息，请参阅第四章，*第 04 天 - 讨论 C#类成员*）。

在本节中，我们将简要讨论继承中的成员可见性。在 C#语言中可能的任何类型的继承中，以下成员不能被基类继承：

+   静态构造函数：静态构造函数是初始化静态数据的构造函数（参考第四章，*第 4 天：讨论 C#类成员*中的*修饰符*部分）。静态构造函数的重要性在于，在创建类的第一个实例或在某些操作中调用或引用其他静态成员之前调用这些构造函数。作为静态数据初始化程序，静态构造函数不能被派生类继承。

+   **实例构造函数**：这不是静态构造函数；每当创建类的新实例时，都会调用构造函数，即实例类。一个类可以有多个构造函数。由于实例构造函数用于创建类的实例，因此不能被派生类继承。有关构造函数的更多信息，请参阅[`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constructors`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constructors)。

+   **Finalizers**：这些只是类的析构函数。这些在运行时由垃圾收集器调用或使用来销毁类的实例。由于析构函数只调用一次且每个类只有一个，因此不能被派生类继承。有关析构函数的更多信息，请参阅[`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/destructors`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/destructors)。

派生类可以重用或继承基类的所有成员，但其使用或可见性取决于其访问修饰符（参考第四章，*第 4 天 - 讨论 C#类成员*）。这些成员的不同可见性取决于以下可访问性修饰符：

+   **私有**：如果成员是`private`，则`private`成员的可见性仅限于其派生类；如果派生类嵌套在其基类中，则派生类中将可用`private`成员。

考虑以下代码片段：

```cs
public class BaseClass
{
   private const string AuthorName = "Gaurav Aroraa";
   public class DeriveClass: BaseClass
   {
      public void Display()
      {
         Write("This is from inherited Private member:");
         WriteLine($"{nameof(AuthorName)}'{AuthorName}'");
         ReadLine();
      }
   }
}
BaseClass is to have one private member, AuthorName, and this will be available in DeriveClass, as DeriveClass is a nested class of BaseClass. You can also see this in compile time while moving the cursor over to the usage of the private AuthorName member. See the following screenshot:
```

![](img/00108.jpeg)

前面的图像显示了派生类中私有方法的可见性。如果类嵌套在其基类中，则派生类中的私有方法是可见的。

如果类没有嵌套在其父类/基类中，则可以看到以下编译时异常：

![](img/00109.jpeg)

在前面的屏幕截图中，我们有`ChildClass`，它继承自`BaseClass`。在这里，我们不能使用`BaseClass`的私有成员，因为`ChildClass`没有嵌套在`BaseClass`中。

+   **受保护**：如果成员是受保护修饰符，则仅对派生类可见。在使用基类的实例时，这些成员将不可用或不可见，因为它们被定义为受保护。

以下屏幕截图显示了如何使用基类访问/可见受保护的成员：

![](img/00110.jpeg)

在前面的屏幕截图中，受保护的成员`EditorName`在`ChildClass`中可见，因为它继承了`BaseClass`。

以下屏幕截图显示了使用`BaseClass`实例中不可访问受保护成员的情况。如果尝试这样做，将会收到编译时错误：

![](img/00111.jpeg)

+   **内部**：具有内部修饰符的成员仅在与基类相同程序集的派生类中可用。这些成员对于属于其他程序集的派生类将不可用或不可见。

考虑以下代码片段：

```cs
namespace Day07
{
   public class MemberVisibility
   {
      private void InternalMemberExample()
      {
         var childClass = new Lib.ChildClass();
         WriteLine("Calling from derived class that
         belongs to same assembly of BaseClass");
         childClass.Display();
      }
   }
}
```

前面的代码显示了内部成员的可见性。在这里，`ChildClass`属于`Lib`程序集，而`BaseClass`就在其中。

另一方面，如果`BaseClass`存在于`Lib`之外的程序集中，则内部成员将无法访问；请参阅以下屏幕截图：

![](img/00112.jpeg)

上面的截图显示了一个编译时错误，告诉我们内部成员是不可访问的，因为它们在同一个程序集中不可用。

+   **Public**：公共成员在派生类中可用或可见，并且可以进一步使用。

考虑以下代码片段：

```cs
public class ChilClassYounger : ChildClass
{
   private string _copyEditor = "Diwakar Shukla";
   public new void Display()
   {
      WriteLine($"This is from ChildClassYounger: copy
      editor is '{_copyEditor}'");
      WriteLine("This is from ChildClass:");
      base.Display();
   }
}
ChildClassYoung has a Display() method that displays the console output. ChildClass also has a public Display() method that also displays the console output. In our derived class, we can reuse the Display() method of ChildClass because it is declared as public. After running the previous code, it will give the following output:
```

![](img/00113.jpeg)

在前面的代码中，您应该注意到我们在`ChildClassYounger`类的`Display()`方法中添加了一个`new`关键字。这是因为我们在父类（即`ChildClass`）中有一个同名的方法。如果我们不添加`new`关键字，我们将看到一个编译时警告，如下面的截图所示：

![](img/00114.jpeg)

通过应用`new`关键字，您隐藏了从`ChildClass`继承的`ChildClass.Display()`成员。在 C#中，这个概念被称为方法隐藏。

# 实现继承

在前一节中，您详细了解了继承，并了解了继承成员的可见性。在本节中，我们将实现继承。

继承是**IS-A**关系的表示，这表明`Author`**IS-A**`Person`，`Person`**IS-A**`Human`，所以`Author`**IS-A**`Human`。让我们在一个代码示例中理解这一点：

```cs
public class Person
{
   public string FirstName { get; set; } = "Gaurav";
   public string LastName { get; set; } = "Aroraa";
   public int Age { get; set; } = 43;
   public string Name => $"{FirstName} {LastName}";
   public virtual void Detail()
   {
      WriteLine("Person's Detail:");
      WriteLine($"Name: {Name}");
      WriteLine($"Age: {Age}");
      ReadLine();
   }
}
public class Author:Person
{
   public override void Detail()
   {
      WriteLine("Author's detail:");
      WriteLine($"Name: {Name}");
      WriteLine($"Age: {Age}");
      ReadLine();
   }
}
public class Editor : Person
{
   public override void Detail()
   {
    WriteLine("Editor's detail:");
    WriteLine($"Name: {Name}");
    WriteLine($"Age: {Age}");
    ReadLine();
   }
}
public class Reviewer : Person
{
   public override void Detail()
   {
      WriteLine("Reviewer's detail:");
      WriteLine($"Name: {Name}");
      WriteLine($"Age: {Age}");
      ReadLine();
   }
}
```

在上面的代码中，我们有一个基类`Person`和三个派生类，分别是`Author`、`Editor`和`Reviewer`。这显示了单一继承。以下是先前代码的实现：

```cs
private static void InheritanceImplementationExample()
{
   WriteLine("Inheritance implementation");
   WriteLine();
   var person = new Person();
   WriteLine("Parent class Person:");
   person.Detail();
   var author = new Author();
   WriteLine("Derive class Author:");
   Write("First Name:");
   author.FirstName = ReadLine();
   Write("Last Name:");
   author.LastName = ReadLine();
   Write("Age:");
   author.Age = Convert.ToInt32(ReadLine());
   author.Detail();
   //code removed
}
```

在上面的代码中，我们实例化了一个单一类并调用了 details；每个类都继承了`Person`类，因此，它的所有成员。这产生了以下输出：

![](img/00115.gif)

# 在 C#中实现多重继承

我们已经在前一节中讨论过 C#不支持多重继承。但是我们可以借助接口实现多重继承（请参阅第二章，*第 02 天-开始使用 C#*）。在本节中，我们将使用 C#实现多重继承。

让我们考虑前一节的代码片段，该片段实现了单一继承。让我们通过实现接口来重写代码。

接口代表**具有**/**能够做**的关系，这表明`Publisher`**具有**`Author`和`Author`**具有**`Book`。在 C#中，您可以将类的实例分配给任何类型为接口或基类的变量。从面向对象的角度来看，这个概念被称为多态性（有关更多细节，请参阅*多态性*部分）。

首先，让我们创建一个接口：

```cs
public interface IBook
{
   string Title { get; set; }
   string Isbn { get; set; }
   bool Ispublished { get; set; }
   void Detail();
}
IBook interface, which is related to book details. This interface is intended to collect book details, such as Title, ISBN, and whether the book is published. It has a method that provides the complete book details.
```

现在，让我们实现`IBook`接口来派生`Author`类，该类继承了`Person`类：

```cs
public class Author:Person, IBook
{
   public string Title { get; set; }
   public string Isbn { get; set; }
   public bool Ispublished { get; set; }
   public override void Detail()
   {
      WriteLine("Author's detail:");
      WriteLine($"Name: {Name}");
      WriteLine($"Age: {Age}");
      ReadLine();
   }
   void IBook.Detail()
   {
      WriteLine("Book details:");
      WriteLine($"Author Name: {Name}");
      WriteLine($"Author Age: {Age}");
      WriteLine($"Title: {Title}");
      WriteLine($"Isbn: {Isbn}");
      WriteLine($"Published: {(Ispublished ? "Yes" :
      "No")}");
      ReadLine(); 
   } 
} 
IBook interface. Our derived class Author inherits the Person base class and implements the IBook interface. In the preceding code, a notable point is that both the class and interface have the Detail() method. Now, it depends on which method we want to modify or which method we want to reuse. If we try to modify the Detail() method of the Person class, then we need to override or hide it (using the new keyword). On the other hand, if we want to use the interface's method, we need to explicitly call the IBook.Detail() method. When you call interface methods explicitly, modifiers are not required; hence, there is no need to put a public modifier here. This method implicitly has public visibility:
```

```cs
//multiple Inheritance
WriteLine("Book details:");
Write("Title:");
author.Title = ReadLine();
Write("Isbn:");
author.Isbn = ReadLine();
Write("Published (Y/N):");
author.Ispublished = ReadLine() == "Y";((IBook)author).Detail(); //
we need to cast as both Person class and IBook has same named methods
Author class with IBook:
```

![](img/00116.gif)

上面的图像显示了使用接口实现的代码的输出。接口的所有成员对子类都是可访问的；在实例化子类时，不需要特殊实现。子类的实例能够访问所有可见成员。在上述实现中的重要一点是在`((IBook)author).Detail();`语句中，我们显式地将子类的实例转换为接口，以获得接口成员的实现。默认情况下，它提供类成员的实现，因此我们需要明确告诉编译器我们需要一个接口方法。

# 抽象

抽象是通过隐藏不相关或不必要的信息来显示相关数据的过程。例如，如果您购买了一部手机，您可能对您的消息是如何传递的或者您的呼叫是如何连接到另一个号码不感兴趣，但您可能会对知道每当您在手机上按下呼叫按钮时，它应该连接您的呼叫感兴趣。在这个例子中，我们隐藏了那些用户不感兴趣的功能，并提供了用户感兴趣的功能。这个过程叫做抽象。

# 实现抽象

在 C#中，抽象类可以通过以下方式实现：

# 抽象类

抽象类是半定义的，这意味着它提供了一种方法来覆盖其子类的成员。我们应该在需要将相同成员提供给所有子类并具有自己实现或想要覆盖的项目中使用基类。例如，如果我们有一个抽象类 Car，其中有一个抽象方法 color，并且有子类 HondCar、FordCar、MarutiCar 等。在这种情况下，所有子类都将具有 color 成员，但具有不同的实现，因为 color 方法将在子类中被覆盖并具有自己的实现。这里需要注意的一点是 - 抽象类代表 IS-A 关系。

您还可以在 Day04 部分“抽象”中重新讨论我们对抽象类的讨论，并查看代码示例以了解实现。

# 抽象类的特点

在前一节中，我们了解了抽象类的一些特点：

+   抽象类不能被初始化，这意味着您不能创建抽象类的对象。

+   抽象类旨在充当基类，因此其他类可以继承它。

+   如果声明了抽象类，则按设计必须由其他类继承。

+   抽象类可以同时拥有具体方法和抽象方法。抽象方法应该在继承抽象类的子类中实现。

# 接口

接口不包含功能或具体成员。您可以称之为类或结构将实现以定义功能签名的合同。通过使用接口，您可以确保每当类或结构实现它时，该类或结构将使用接口的合同。例如，如果 ICalculator 接口具有 Add()方法，这意味着每当类或结构实现此接口时，它都提供了一个特定的合同功能，即加法。

有关接口的更多信息，请参阅：[`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/index`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/index)

接口只能拥有以下成员：

+   方法

+   属性

+   索引器

+   事件

# 接口的特点

以下是接口的主要特点

+   接口默认为 internal

+   接口的所有成员默认为 public，并且不需要显式应用 public 修饰符到成员上。

+   类似地，接口也不能被实例化。它们只能实现，并且实现它的类或结构应该实现所有成员。

+   接口不能包含任何具体方法

+   接口可以被另一个接口、类或结构实现。

+   类或结构可以实现多个接口。

类可以继承抽象类或普通类并实现接口。

在本节中，我们将使用抽象类来实现抽象。让我们考虑以下代码片段：

```cs
public class AbstractionImplementation
{
public void Display()
{
BookAuthor author = new BookAuthor();
author.GetDetail();
BookEditor editor = new BookEditor();
editor.GetDetail();
BookReviewer reviewer = new BookReviewer();
reviewer.GetDetail();
}
}
```

上面的代码片段只包含一个负责显示操作的公共方法。Display()方法是获取书籍作者、编辑和评论员详细信息的方法。乍一看，我们可以说上面的代码是不同实现的不同类。但实际上，我们正在使用抽象类来抽象我们的代码，然后派生类提供所需的详细信息。

考虑以下代码：

```cs
public abstract class Team
{
public abstract void GetDetail();
}
```

我们有一个抽象类 Team，其中有一个抽象方法 GetDetail()，这个方法负责获取团队的详细信息。现在，想一下这个团队包括什么，这个团队由作者、编辑和评论员组成。因此，我们有以下代码片段：

```cs
public class BookAuthor : Team
{
public override void GetDetail() => Display();
private void Display()
{
WriteLine("Author detail");
Write("Enter Author Name:");
var name = ReadLine();
WriteLine($"Book author is: {name}");
}
}
```

BookAuthor 类继承 Team 并重写 GetDetail()方法。该方法进一步调用一个私有方法 Display()，这是用户不会意识到的。用户只会调用 GetDetail()方法。

同样，我们还有 BookEditor 和 BookReviewer 类：

```cs
public class BookEditor : Team
{
public override void GetDetail() => Display();
private void Display()
{
WriteLine("Editor detail");
Write("Enter Editor Name:");
var name = ReadLine();
WriteLine($"Book editor is: {name}");
}
}
public class BookReviewer : Team
{
public override void GetDetail() => Display();
private void Display()
{
WriteLine("Reviewer detail");
Write("Enter Reviewer Name:");
var name = ReadLine();
WriteLine($"Book reviewer is: {name}");
}
}
```

在上述代码中，类将只公开一个方法，即`GetDetail()`，以提供所需的详细信息。

当客户端调用此代码时，将会得到以下输出：

![](img/00117.jpeg)

# 封装

封装是一种数据不直接对用户可访问的过程。当你想要限制或隐藏客户端或用户对数据的直接访问时，这种活动或过程就被称为封装。

当我们说信息隐藏时，意味着隐藏用户不需要或对信息不感兴趣的信息，例如-当你买一辆自行车时，你可能不会对引擎如何工作，内部如何供应燃料等感兴趣，但你可能对自行车的里程等感兴趣。

信息隐藏不是数据隐藏，而是在 C#中的实现隐藏，欲了解更多信息，请参考：[`blog.ploeh.dk/2012/11/27/Encapsulationofproperties/`](http://blog.ploeh.dk/2012/11/27/Encapsulationofproperties/)。

在 C#中，当函数和数据组合在一个单元中（称为类），并且你不能直接访问数据时，称为封装。在 C#类中，应用访问修饰符到成员、属性，以避免其他情况或用户直接访问数据。

在本节中，我们将详细讨论封装。

# C#中的访问修饰符是什么？

如前一节所讨论的，封装是隐藏信息不让外部世界知道的概念。在 C#中，我们有访问修饰符或访问限定符，可以帮助我们隐藏信息。这些访问修饰符帮助你定义类成员的范围和可见性。

以下是访问修饰符：

+   公共的

+   私有的

+   受保护的

+   内部的

+   受保护的内部

我们在第四天已经讨论了所有上述的访问修饰符。请参考*访问修饰符*部分以及它们的可访问性，以了解这些修饰符如何工作并帮助我们定义可见性。

# 实现封装

在本节中，我们将在 C# 7.0 中实现封装。想象一个场景，我们需要提供一个`Author`的信息，包括最近出版的书。考虑以下代码片段：

```cs
internal class Writer
{
   private string _title;
   private string _isbn;
   private string _name;
   public void SetName(string fname, string lName)
   {
      if (string.IsNullOrEmpty(fname) ||
      string.IsNullOrWhiteSpace(lName))
      throw new ArgumentException("Name can not be
      blank.");
      _name = $"{fname} {lName}";
   }
   public void SetTitle(string title)
   {
      if (string.IsNullOrWhiteSpace(title))
      throw new ArgumentException("Book title can not be
      blank.");
      _title = title;
   }
   public void SetIsbn(string isbn)
   {
      if (!string.IsNullOrEmpty(isbn))
      {
         if (isbn.Length == 10 | isbn.Length == 13)
         {
            if (!ulong.TryParse(isbn, out _))
            throw new ArgumentException("The ISBN can
            consist of numeric characters only.");
         }
         else
      throw new ArgumentException("ISBN should be 10 or 13
      characters numeric string only.");
      }
    _isbn = isbn;
   }
   public override string ToString() => $"Author '{_name}'
   has authored a book '{_title}' with ISBN '{_isbn}'";
  }
```

在上述显示封装实现的代码片段中，我们隐藏了用户不想知道的字段。因为主要目的是展示最近的出版物。

以下是客户端需要的信息的代码：

```cs
public class EncapsulationImplementation
{
   public void Display()
   {
      WriteLine("Encapsulation example");
      Writer writer = new Writer();
      Write("Enter First Name:");
      var fName = ReadLine();
      Write("Enter Last Name:");
      var lName = ReadLine();
      writer.SetName(fName,lName);
      Write("Book title:");
      writer.SetTitle(ReadLine());
      Write("Enter ISBN:");
      writer.SetIsbn(ReadLine());
      WriteLine("Complete details of book:");
      WriteLine(writer.ToString());
   }
}
```

上述代码片段只是为了获取所需的信息。用户不会知道信息是如何从类中获取/检索的。

![](img/00118.jpeg)

上述图像显示了在执行前面的代码后，您将看到的确切输出。

# 多态性

简单来说，多态意味着有多种形式。在 C#中，我们可以将一个接口表达为多个函数，这就是多态。多态来自希腊词，意思是“多种形式”。

在 C#中，所有类型（包括用户定义的类型）都继承自对象，因此 C#中的每种类型都是多态的。

正如我们讨论的，多态意味着多种形式。这些形式可以是函数的形式，在派生类中以不同形式实现相同名称和相同参数的函数。此外，多态性具有提供相同名称的方法的不同实现的能力。

在接下来的部分中，我们将讨论各种类型的多态性，包括它们在 C# 7.0 中的实现。

# 多态性的类型

在 C#中，我们有两种多态性，这些类型是：

+   **编译时多态**

编译时多态也被称为早期绑定或重载或静态绑定。它在编译时确定，并用于具有不同参数的相同函数名称。编译时或早期绑定进一步分为两种类型，这些类型是：

+   +   **函数重载**

函数重载，如其名所示，函数被重载。当您声明具有相同名称但不同参数的函数时，它被称为函数重载。您可以声明尽可能多的重载函数。

考虑以下代码片段：

```cs
public class Math
{
    public int Add(int num1, int num2) =>   num1 + num2;
    public double Add(double num1, double num2) => num1 + num2;
}
```

上述代码是重载的一种表示，`Math`类有一个带有双重载参数的`Add()`方法。这些方法用于分离行为。考虑以下代码：

```cs
public class CompileTimePolymorphismImplementation
{
   public void Run()
   {
      Write("Enter first number:");
      var num1 = ReadLine();
      Write("Enter second number:");
      var num2 = ReadLine();
      Math math = new Math();
      var sum1 = math.Add(FloatToInt(num1),
      FloatToInt(num1));
      var sum2 = math.Add(ToFloat(num1), ToFloat(num2));
      WriteLine("Using Addd(int num1, int num2)");
      WriteLine($"{FloatToInt(num1)} + {FloatToInt(num2)}
      = {sum1}");
      WriteLine("Using Add(double num1, double num2)");
      WriteLine($"{ToFloat(num1)} + {ToFloat(num2)} =
      {sum2}");
   }
   private int FloatToInt(string num) =>
   (int)System.Math.Round(ToFloat(num), 0);
   private float ToFloat(string num) = 
   float.Parse(num);
}
```

上述代码片段同时使用了这两种方法。以下是上述实现的输出：

![](img/00119.jpeg)

如果您分析之前的结果，您会发现接受双参数的重载方法提供了准确的结果，即 99，因为我们提供了小数值并且它添加了小数。另一方面，带有整数类型参数的`Add`方法，将双精度数四舍五入并将其转换为整数，因此显示了错误的结果。然而，之前的例子与正确的计算无关，但它告诉了关于使用函数重载进行编译时多态性的情况。

+   +   **运算符重载**

运算符重载是重新定义特定运算符的实际功能的一种方式。

在处理用户定义的复杂类型时，这一点非常重要，直接使用内置运算符是不可能的。

我们已经在第二章中详细讨论了运算符重载，*第 02 天 - 开始使用 C#*部分 - *运算符重载* - 如果您想复习运算符重载，请参考本节。

+   运行时多态性

运行时多态性也被称为晚期绑定、覆盖或动态绑定。我们可以通过在 C#中覆盖方法来实现运行时多态性。虚拟或抽象方法可以在派生类中被覆盖。

在 C#中，抽象类提供了一种在派生类中覆盖抽象方法的方法。`virtual`关键字也是在派生类中覆盖方法的一种方式。我们在第二章中讨论了`virtual`关键字（如果您想复习，请参考）。

考虑以下示例：

```cs
internal abstract class Team
{
   public abstract string Detail();
}
internal class Author : Team
{
   private readonly string _name;
   public Author(string name) => _name = name;
   public override string Detail()
   {
      WriteLine("Author Team:");
      return $"Member name: {_name}";
   }
}
Team is having an abstract method Detail() that is overridden.
```

```cs
public class RunTimePolymorphismImplementation
{
   public void Run()
   {
      Write("Enter name:");
      var name = ReadLine();
      Author author = new Author(name);
      WriteLine(author.Detail());
   }
}
Author class and produces the following output:
```

![](img/00120.jpeg)

上图显示了一个实现抽象类的程序示例的输出。

我们还可以使用抽象类和虚拟方法实现运行时多态性，考虑以下代码片段：

```cs
internal class Team
{
   protected string Name;
   public Team(string name)
   {
      Name = name;
   }
   public virtual string Detail() => Name;
}
internal class Author : Team
{
   public Author(string name) : base(name)
   {}
   public override string Detail() => Name;
}
internal class Editor : Team
{
   public Editor(string name) : base(name)
   {}
   public override string Detail() => Name;
}
internal class Client
{
   public void ShowDetail(Team team) =>
   WriteLine($"Member: {team.Detail()}");
}
Team and perform the operation by knowing the type of a class at runtime.
```

![](img/00121.jpeg)

我们的`ShowDetail()`方法显示了特定类型的成员名称。

# 实现多态性

让我们在一个完整的程序中实现多态性，考虑以下代码片段：

```cs
public class PolymorphismImplementation
{
   public void Build()
   {
      List<Team> teams = new List<Team> {new Author(), new
      Editor(), new Reviewer()};
      foreach (Team team in teams)
      team.BuildTeam();
   }
}
public class Team
{
   public string Name { get; private set; }
   public string Title { get; private set; }
   public virtual void BuildTeam()
   {
      Write("Name:");
      Name = ReadLine();
      Write("Title:");
      Title = ReadLine();
      WriteLine();
      WriteLine($"Name:{Name}\nTitle:{Title}");
      WriteLine();
   }
}
internal class Author : Team
{
   public override void BuildTeam()
   {
      WriteLine("Building Author Team");
      base.BuildTeam();
   }
}
internal class Editor : Team
{
   public override void BuildTeam()
   {
      WriteLine("Building Editor Team");
      base.BuildTeam();
   }
}
internal class Reviewer : Team
{
   public override void BuildTeam()
   {
      WriteLine("Building Reviewer Team");
      base.BuildTeam();
   }
}
```

上述代码片段是多态性的一种表示，即构建不同的团队。它产生以下输出：

![](img/00122.jpeg)

上图显示了一个程序的结果，该程序表示了多态性的实现。

# 动手练习

以下是今天学习中未解决的问题：

1.  什么是面向对象编程？

1.  为什么我们应该使用面向对象的语言而不是过程式语言？

1.  定义继承？

1.  通常有多少种继承类型？

1.  为什么我们不能在 C#中实现多重继承？

1.  我们如何在 C#中实现多重继承。

1.  用一个简短的程序定义继承成员的可见性。

1.  定义隐藏并用一个简短的程序详细说明。

1.  什么是覆盖？

1.  何时使用隐藏，何时使用覆盖，用一个简短的程序详细说明（提示：参考 - [`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/knowing-when-to-use-override-and-new-keywords`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/knowing-when-to-use-override-and-new-keywords)）

1.  什么是隐式继承？

1.  抽象类和接口之间有什么区别？

1.  什么是封装，用一个简短的程序来详细说明。

1.  定义有助于封装的访问修饰符或访问说明符。

1.  什么是抽象？用一个现实世界的例子详细说明。

1.  用一个现实世界的例子说明封装和抽象的区别。(提示：[`stackoverflow.com/questions/16014290/simple-way-to-understand-encapsulation-and-abstraction`](https://stackoverflow.com/questions/16014290/simple-way-to-understand-encapsulation-and-abstraction))

1.  何时使用抽象类和接口，用简短的程序详细说明。(提示：[`dzone.com/articles/when-to-use-abstract-class-and-intreface`](https://dzone.com/articles/when-to-use-abstract-class-and-intreface))

1.  抽象函数和虚函数有什么区别？(提示：[`stackoverflow.com/questions/391483/what-is-the-difference-between-an-abstract-function-and-a-virtual-function`](https://stackoverflow.com/questions/391483/what-is-the-difference-between-an-abstract-function-and-a-virtual-function))

1.  在 C#中定义多态？

1.  有多少种多态性，使用 C# 7.0 编写一个简短的程序来实现？

1.  用实际例子定义晚期绑定和早期绑定。

1.  用程序证明 - 在 C#中，每种类型都是多态的。

1.  重载和重写有什么区别？

# 重温第 7 天

最后，我们到达了我们 7 天学习系列的最后一天，也就是第七天。今天，我们已经学习了面向对象编程范式的概念，从对象关系开始，概述了关联、聚合和组合，然后讨论了结构化和过程化语言。我们讨论了面向对象编程的封装、抽象、继承和多态这四个特性。我们还使用 C# 7.0 实现了面向对象编程的概念。

明天，第八天，我们将开始一个真实世界的应用程序，这将帮助我们复习到目前为止学到的所有概念。如果你现在想复习，请继续查看之前几天的学习。

# 接下来做什么？

今天我们结束了我们的 7 天学习系列。在这段旅程中，我们从非常基础的开始，然后逐渐适应了高级术语，但这只是一个开始，还有更多要学习。我尽量将几乎所有的东西都结合在这里，为下一步，我建议你应该学习这些：

1.  多线程

1.  构造函数链

1.  索引器

1.  扩展方法

1.  高级正则表达式

1.  高级不安全代码实现

1.  垃圾回收的高级概念

有关更高级的主题，请参考以下内容：

1.  C# 7.0 和 .NET Core Cookbook ([`www.packtpub.com/application-development/c-7-and-net-core-cookbook`](https://www.packtpub.com/application-development/c-7-and-net-core-cookbook))

1.  [`questpond.over-blog.com/`](http://questpond.over-blog.com/)

1.  Functional C# ([`www.packtpub.com/application-development/functional-c`](https://www.packtpub.com/application-development/functional-c))

1.  使用 C# 7.0 进行多线程的 Cookbook - 第二版 ([`www.packtpub.com/application-development/multithreading-c-cookbook-second-edition`](https://www.packtpub.com/application-development/multithreading-c-cookbook-second-edition))
