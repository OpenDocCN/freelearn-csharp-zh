# 第二章：面向对象编程 - 类和对象

**面向对象编程**（**OOP**）是特殊的。如果你在互联网上搜索关于 OOP 的书籍，你会发现数百本关于这个主题的书。但是这个主题永远不会变得陈旧，因为它是行业中最有效、最常用的编程方法。随着对软件开发人员的需求增加，对良好学习内容的需求也在增加。我们在这本书中的方法是以最简单的方式描述 OOP 的概念。理解 OOP 的基础对于想要使用 C#工作的开发人员来说是必须的，因为 C#是一种完全面向对象的语言。在本章中，我们将尝试理解 OOP 到底是什么，以及 OOP 的最基本概念，这些概念对我们开始编程之旅至关重要。在任何其他事情之前，让我们首先分析一下术语**面向对象编程**的含义。

第一个词是**对象**。根据词典的定义，对象是可以看到、感觉到或触摸到的东西；在现实世界中具有物理存在的东西。如果一个物品是虚拟的，这意味着它没有任何物理存在，不被视为对象。第二个词是**面向**，表示方向或目标。例如，当我们说我们面向建筑物时，我们的意思是我们正面朝它。第三个词是**编程**。我相信我不必解释什么是编程，但以防你完全不知道编程是什么并且正在阅读这本书来学习，让我简要解释一下编程是什么。编程只是给计算机指令。由于计算机不会说我们的语言，我们人类必须用计算机能理解的语言给它指令。我们人类称这些指令为**计算机程序**，因为我们正在引导或指导计算机做一件特定的事情。

现在我们知道了这三个关键词的定义，如果我们把所有这些词放在一起，我们就能理解短语“面向对象编程”的含义。OOP 意味着我们编写计算机程序时将对象置于思考的中心。OOP 既不是工具也不是编程语言，它只是一个概念。一些编程语言是设计遵循这个概念的。C#是最流行的面向对象语言之一。还有其他面向对象的语言，比如 Java、C++等等。

在 OOP 中，我们试图将软件组件看作小对象，并在它们之间建立关系来解决问题。你可能在编程世界中遇到过其他编程概念，比如过程式编程、函数式编程和其他类型的编程。有史以来最流行的计算机编程语言之一——C 编程语言是一种过程式编程语言。F#是函数式编程语言的一个例子。

在本章中，我们将涵盖 OOP 的以下主题：

+   OOP 中的类

+   类的一般形式

+   什么是对象？

+   类中的方法

+   OOP 的特点

# OOP 中的类

在 OOP 中，你从类中派生对象。在本节中，我们将更仔细地看看类到底是什么。

类是 OOP 中最重要的概念之一。你可以说它们是 OOP 的构建模块。类可以被描述为对象的蓝图。

类似于一个模板或蓝图，告诉我们这个类的实例将具有什么属性和行为。在大多数情况下，一个类本身实际上不能做任何事情——它只是用来创建对象的。让我们看一个例子来说明我所说的。假设我们有一个`Human`类。在这里，当我们说`Human`时，我们并不是指任何特定的人，而是指一般的人类。一个有两只手、两条腿和一个嘴巴的人，还可以走路、说话、吃饭和思考。这些属性及其行为适用于大多数人类。我知道这对于残疾人来说并非如此，但现在，我们将假设我们的一般人类是健全的，以保持我们的例子简单。因此，当我们在一个对象中看到上述属性和行为时，我们可以很容易地将该对象归类为人类对象或人。这种分类在面向对象编程中称为类。

让我们更仔细地看看`Human`类的属性和行为。人类可以列举数百种属性，但为了简单起见，我们可以说以下是人类的属性：

+   高度

+   体重

+   年龄

我们也可以对行为属性做同样的事情。一个人可以执行数百种特定的行为，但在这里我们只考虑以下行为：

+   走

+   谈

+   吃

# 类的一般形式

要在 C#中创建一个类，必须遵循特定的语法。其一般形式如下：

```cs
class class-name {
    // this is class body
}
```

`class`短语是 C#中的**保留关键字**，用于告诉编译器我们想要创建一个类。要创建一个类，需要在一个空格后放置`class`关键字，然后是类的名称。类的名称可以是以字符或下划线开头的任何内容。类名中也可以包括数字，但不能是类名的第一个字符。在选择的类名之后，必须放置一个开放的大括号，表示类体的开始。您可以在类中添加内容，例如属性和方法，然后用一个闭合的大括号结束类，如下所示：

```cs
class class-name {
 // property 1
 // property 2
 // ...

 // method 1
 // method 2
 // ...
}
```

还有其他关键字可以与类一起使用，以添加更多功能，例如访问修饰符、虚方法、部分方法等。不要担心这些关键字或它们的用途，因为我们将在本书的后面讨论这些内容。

# 编写一个简单的类

现在让我们创建我们的第一个类。假设我们正在为一家银行开发一些软件。我们的应用程序应该跟踪银行的客户及其银行账户，并对这些银行账户执行一些基本操作。由于我们将使用 C#设计我们的应用程序，因此我们必须以面向对象的方式思考我们的应用程序。我们将需要这个应用程序的一些对象，比如客户对象、银行账户对象和其他对象。因此，为了制作这些对象的蓝图，我们必须创建一个`Customer`类和一个`BankAccount`类，以及我们将需要的其他类。让我们首先使用以下代码创建`Customer`类：

```cs
class Customer
{
    public string firstName;
    public string lastName;
    public string phoneNumber;
    public string emailAddress;

    public string GetFullName()
    {
        return firstName + " " + lastName;
    }
}
```

我们从`class`关键字开始，然后是`Customer`类的名称。之后，我们在大括号`{}`内添加了类体。该类拥有的变量是`firstName`、`lastName`、`phoneNumber`和`emailAddress`。该类还有一个名为`GetFullName()`的方法，该方法使用`firstName`和`lastName`字段来准备全名并返回它。

现在让我们使用以下代码创建一个`BankAccount`类：

```cs
class BankAccount {
    public string bankAccountNumber;
    public string bankAccountOwnerName;
    public double amount;
    public datetime openningDate;

    public string Credit(){
        // Amount credited
    }

    public string Debit(){
        // Amount debited
    }
}
```

在这里，我们可以看到我们已经遵循了创建类的类似方法。我们使用了`class`关键字，然后是`BankAccount`类的名称。在名称之后，我们用一个开放的大括号开始了类体，并输入了字段，如`bankAccountNumber`、`bankAccountOwnerName`、`amount`和`openningDate`，然后是两个方法，`Credit`和`Debit`。通过放置一个闭合的大括号，我们结束了类体。

现在，不要担心诸如**public**之类的关键字；当我们讨论访问修饰符时，我们将在本书的后面学习这些关键字。

# 面向对象编程中的对象

我们现在知道了**类**是什么。现在让我们来看看面向对象编程中**对象**是指什么。

对象是类的一个实例。换句话说，对象是类的一个实现。例如，在我们的银行应用程序中，我们有一个`Customer`类，但这并不意味着我们实际上在我们的应用程序中有一个客户。要创建一个客户，我们必须创建`Customer`类的对象。假设我们有一个名为琼斯先生的客户。对于这个客户，我们必须创建`Customer`类的对象，其中人的名字是杰克琼斯。

由于琼斯先生是我们的客户，这意味着他也在我们的银行有一个账户。要为琼斯先生创建一个银行账户，我们必须创建一个`BankAccount`类的对象。

# 如何创建对象

在 C#中，要创建一个类的对象，您必须使用`new`关键字。让我们看一个对象的例子：

```cs
Customer customer1 = new Customer();
```

在这里，我们首先写了`Customer`，这是类的名称。这代表了对象的类型。之后，我们给出了对象的名称，在这种情况下是`customer1`。您可以给该对象任何名称。例如，如果客户是琼斯先生，我们可以将对象命名为`jackJones`。在对象名称之后，我们插入了一个等号（`=`），这意味着我们正在给`customer1`对象赋值。之后，我们输入了一个称为`new`的关键字，这是一个特殊的关键字，告诉编译器创建给定类的新对象。在这里，我们再次给出了`Customer`，并在其旁边加上了`()`。当我们放置`Customer()`时，我们实际上正在调用该类的构造函数。我们将在后续章节中讨论构造函数。

我们可以使用以下代码创建`jackJones`：

```cs
Customer jackJones = new Customer();
```

# C#中的变量

在前面的代码中，您可能已经注意到我们创建了一些变量。**变量**是一种变化的东西，这意味着它不是常数。在编程中，当我们创建一个变量时，计算机实际上会为其分配内存空间，以便可以将变量的值存储在那里。

让我们为我们在上一节中创建的对象的变量分配一些值。我们将首先处理`customer1`对象，如下所示的代码：

```cs
using System;

namespace Chapter2
{
    public class Code_2_2
    {
        static void Main(string[] args)
        {
            Customer customer1 = new Customer();
            customer1.firstName = "Molly";
            customer1.lastName = "Dolly";
            customer1.phoneNumber = "98745632";
            customer1.emailAddress = "mollydolly@email.com";

            Console.WriteLine("First name is " + customer1.firstName);
            Console.ReadKey();
        }
    }

    public class Customer
    {
        public string firstName;
        public string lastName;
        public string phoneNumber;
        public string emailAddress;

        public string GetFullName()
        {
            return firstName + " " + lastName;
        }
    }
}
```

在这里，我们正在给`customer1`对象赋值。该代码指示计算机在内存中创建一个空间并将值存储在其中。稍后，每当您访问变量时，计算机将转到内存位置并找出变量的值。现在，如果我们编写一个语句，将打印`firstName`变量的值以及其前面的附加字符串，它将如下所示：

```cs
Console.WriteLine("First name is " + customer1.firstName);
```

这段代码的输出将如下所示：

```cs
First name is Molly
```

# 类中的方法

让我们谈谈另一个重要的话题——方法。**方法**是在代码文件中编写的可以重复使用的代码片段。一个方法可以包含许多行代码，在调用时将被执行。让我们来看一下方法的一般形式：

```cs
access-modifier return-type method-name(parameter-list) {
    // method body
}
```

我们可以看到方法声明中的第一件事是`access-modifier`。这将设置方法的访问权限。然后，我们有方法的`return-type`，它将保存方法将返回的类型，例如`string`，`int`，`double`或其他类型。之后，我们有`method-name`，然后是括号`()`，表示这是一个方法。在括号中，我们有`parameter-list`。这可以是空的，也可以包含一个或多个参数。最后，我们有花括号`{}`，其中包含方法体。方法将执行的代码放在这里。

按照这种结构的任何代码将被 C#编译器视为方法。

# 创建一个方法

既然我们知道了方法是什么，让我们来看一个例子，如下所示的代码：

```cs
public string GetFullName(string firstName, string lastName){
    return firstName + lastName;
}
```

这段代码将创建一个名为`GetFullName`的方法。这个方法接受两个参数，`firstName`和`lastName`，放在括号里。我们还可以看到，我们必须指定这些参数的类型。在这个特定的例子中，这两个参数的类型都是`string`。

现在，让我们看一下方法体，即大括号之间的部分`{}`。我们可以看到，代码返回`firstName + lastName`，这意味着它正在连接这两个参数`firstName`和`lastName`，并返回`string`。因为我们打算从这个方法返回一个`string`，所以我们将方法的返回类型设置为`string`。另一个需要注意的是，这个方法的访问类型设置为`public`，这意味着任何其他类都可以访问它。

# 类的构造函数

在每个类中，都有一种特殊类型的方法，称为**构造函数**。你可以在一个类中创建一个构造函数并对其进行编程。如果你自己不创建一个，编译器将创建一个非常简单的构造函数并使用它。让我们来看看构造函数是什么，它的作用是什么。

构造函数是在创建类的对象时触发的方法。构造函数主要用于设置类的先决条件。例如，如果你正在创建`Human`类的对象，那个人的对象必须有一个`出生日期`。没有出生日期，就不会有人存在。你可以在构造函数中设置这个要求。你还可以配置构造函数，如果没有提供出生日期，则将出生日期设置为今天。这取决于你的应用程序的需求。另一个例子可能是`bank account`对象，你必须提供银行账户持有人。没有所有者，就不可能存在银行账户，所以你可以在构造函数中设置这个要求。

让我们来看一下构造函数的一般形式，如下面的代码所示：

```cs
access-modifier class-name(parameter-list) {
    // constructor body
}
```

在这里，我们可以看到构造函数和普通方法之间有一个区别，即构造函数没有返回类型。这是因为构造函数不能返回任何东西；它是用于初始化，而不是用于任何其他类型的操作。通常，构造函数的访问类型是`public`，因为否则无法实例化对象。如果你特别想阻止类的对象被实例化，你可以将构造函数设置为`private`。让我们看一个构造函数的例子，如下面的代码所示：

```cs
class BankAccount {
    public string owner;

    public BankAccount(){
        owner = "Some person";
    }
}
```

在这个例子中，我们可以看到有一个名为`BankAccount`的类，它有一个名为`owner`的变量。正如我们所知，没有所有者的银行账户是不存在的，所以我们需要在创建对象时为`owner`赋值。为了创建一个`构造函数`，我们只需将构造函数的访问类型设置为`public`，因为我们希望对象被实例化。我们还可以在构造函数中将银行账户所有者的姓名作为参数，并将其用于赋值给变量，如下面的代码所示：

```cs
class BankAccount {
    public string owner;

    public BankAccount(string theOwner){
        owner = theOwner;
    }
}
```

如果在构造函数中放入参数，那么在初始化对象时，需要传递参数，如下面的代码所示：

```cs
BankAccount account = new BankAccount("Some Person");
```

另一个有趣的事情是，你可以在一个类中有多个构造函数。你可能有一个构造函数带有一个参数，另一个不带任何参数。根据初始化对象的方式，将调用相应的构造函数。让我们看下面的例子：

```cs
class BankAccount {
    public string owner;

    public BankAccount(){
        owner = "Some person";
    }

    public BankAccount(string theOwner){
        owner = theOwner;
    }
}
```

在上面的例子中，我们可以看到`BankAccount`类有两个构造函数。如果在创建`BankAccount`对象时传递参数，它将调用第二个构造函数，这将设置值并创建对象。如果在创建对象时不传递参数，将调用第一个构造函数。如果这两个构造函数都没有，那么这种对象创建方法将不可用。

如果您不创建一个类，那么编译器会为该类创建一个空的构造函数，如下所示：

```cs
class BankAccount {
    public string owner;

    public BankAccount()
    {
    }
}
```

# 面向对象编程的特点

面向对象编程是当今最重要的编程方法之一。整个概念依赖于四个主要思想，被称为**面向对象编程的支柱**。这四个支柱如下：

+   继承

+   封装

+   多态

+   抽象

# 继承

**继承**一词意味着从其他地方接收或衍生出某物。在现实生活中，我们可能会谈论一个孩子从父母那里继承房子。在这种情况下，孩子对房子拥有与父母相同的权力。这种继承的概念是面向对象编程的支柱之一。在编程中，当一个类从另一个类派生时，这被称为继承。这意味着派生类将具有与父类相同的属性。在编程术语中，从另一个类派生的类被称为**父类**，而继承自这些类的类被称为**子类**。

让我们看一个例子：

```cs
public class Fruit {
    public string Name { get; set; }
    public string Color { get; set; }
}

public class Apple : Fruit {
    public int NumberOfSeeds { get; set; }
}
```

在上面的例子中，我们使用了继承。我们有一个名为`Fruit`的父类。这个类包含每种水果都有的共同属性：`Name`和`Color`。我们可以为所有水果使用这个`Fruit`类。

如果我们创建一个名为`Apple`的新类，这个类可以继承`Fruit`类，因为我们知道苹果是一种水果。`Fruit`类的属性也是`Apple`类的属性。如果`Apple`继承`Fruit`类，我们就不需要为`Apple`类编写相同的属性，因为它从`Fruit`类继承了这些属性。

# 封装

**封装**意味着隐藏或覆盖。在 C#中，封装是通过**访问修饰符**实现的。在 C#中可用的访问修饰符如下：

+   公共

+   私有

+   保护

+   内部

+   内部保护

封装是当您想要控制其他类对某个类的访问时使用的。比如说您有一个`BankAccount`类。出于安全原因，让这个类对所有类都可访问并不是一个好主意。最好将其设为`私有`或使用其他类型的访问修饰符。

您还可以限制对类的属性和变量的访问。例如，您可能需要保持`BankAccount`类对某些原因是`public`的，但将`AccountBalance`属性设为`private`，这样除了`BankAccount`类之外，其他类都无法访问这个属性。您可以这样做：

```cs
public class BankAccount {
    private double AccountBalance { get; set; }
}
```

像变量和属性一样，您还可以为方法使用访问修饰符。您可以编写不需要其他类使用的`private`方法，或者您不希望向其他类公开的方法。让我们看下面的例子：

```cs
public class BankAccount{
    private double AccountBalance { get; set; }
    private double TaxRate { get; set; }

    public double GetAccountBalance() {
        double balanceAfterTax = GetBalanceAfterTax();
        return balanceAfterTax;
    }

    private double GetBalanceAfterTax(){
        return AccountBalance * TaxRate;
    }
}
```

在上面的例子中，`GetBalanceAfterTax`方法是一个其他类不需要的方法。我们只想提供税后的`AccountBalance`，所以我们可以将这个方法设为私有。

封装是面向对象编程的一个非常重要的部分，因为它让我们对代码有控制权。

# 抽象

如果某物是抽象的，意味着它在现实中没有实例，但作为一个想法或概念存在。在编程中，我们使用这种技术来组织我们的思想。这是面向对象编程的支柱之一。在 C#中，我们有`abstract`类，它实现了抽象的概念。**抽象类**是没有任何实例的类，实现`abstract`类的类将实现该`abstract`类的属性和方法。让我们看一个`abstract`类的例子，如下面的代码所示：

```cs
public abstract class Vehicle {
    public abstract int GetNumberOfTyres();
}

public class Bicycle : Vehicle {
    public string Company { get; set; }
    public string Model { get; set; }
    public int NumberOfTyres { get; set; }

    public override int GetNumberOfTyres() {
        return NumberOfTyres;
    }
}

public class Car : Vehicle {
    public string Company { get; set; }
    public string Model { get; set; }
    public int FrontTyres { get; set; }
    public int BackTyres { get; set; }

    public override int GetNumberOfTyres() {
        return FrontTyres + BackTyres;
    }
}
```

在前面的例子中，我们有一个名为`Vehicle`的抽象类。它有一个名为`GetNumberOfTyres()`的抽象方法。由于它是一个抽象方法，这个方法必须被实现抽象类的类所覆盖。我们的`Bicycle`和`Car`类实现了`Vehicle`抽象类，因此它们也覆盖了抽象方法`GetNumberOfTyres()`。如果你看一下这两个类中这些方法的实现，你会发现实现是不同的，这是由于抽象性。

# 多态性

多态一词意味着许多形式。要正确理解多态的概念，让我们举个例子。让我们想想一个人，比如比尔·盖茨。我们都知道比尔·盖茨是一位伟大的软件开发者、商人、慈善家，也是一位伟大的人。他是一个人，但他有不同的角色和执行不同的任务。这就是多态性。当比尔·盖茨正在开发软件时，他扮演着软件开发者的角色。他在思考他正在编写的代码。后来，当他成为微软的首席执行官时，他开始管理人员并思考如何发展业务。他是同一个人，但担任不同的角色和不同的责任。

在 C#中，有两种多态性：静态多态性和动态多态性。静态多态性是一种多态性，其中方法的角色在编译时确定，而在动态多态性中，方法的角色在运行时确定。静态多态性的例子包括方法重载和运算符重载。让我们看一个方法重载的例子：

```cs
public class Calculator {
    public int AddNumbers(int firstNum, int secondNum){
        return firstNum + secondNum;
    }

    public double AddNumbers(double firstNum, double secondNum){
        return firstNum + secondNum;
    }
}
```

在这里，我们可以看到我们有两个同名的方法`AddNumbers`。通常情况下，我们不能有两个同名的方法；然而，由于这些方法的参数是不同的，编译器允许方法具有相同的名称。编写一个与另一个方法同名但参数不同的方法称为方法重载。这是一种多态性。

像方法重载一样，运算符重载也是一种静态多态性。让我们看一个运算符重载的例子来证明这一点：

```cs
public class MyCalc
{
    public int a;
    public int b;

    public MyCalc(int a, int b)
    {
        this.a = a;
        this.b = b;
    }

    public static MyCalc operator +(MyCalc a, MyCalc b)
    {
        return new MyCalc(a.a * 3 ,b.b * 3);
    }
}
```

在前面的例子中，我们可以看到加号（+）被重载为另一种计算。因此，如果你对两个`MyCalc`对象求和，你将得到一个重载的结果，而不是正常的和，这种重载发生在编译时，因此它是静态多态性。

**动态多态性**指的是使用抽象类。当你编写一个抽象类时，不能从该抽象类创建实例。当任何其他类使用或实现该抽象类时，该类也必须实现该抽象类的抽象方法。由于不同的类可以实现抽象类并且可以有不同的抽象方法实现，因此实现了多态行为。在这种情况下，我们有相同名称但不同实现的方法。

# 总结

这一章涵盖了类和对象，这是面向对象编程中最重要的构建模块。这些是我们在跳入面向对象编程的任何其他主题之前应该学习的两件事。在继续其他想法之前，确保我们的思想中清楚了这些概念是很重要的。在这一章中，我们了解了类是什么，以及为什么在面向对象编程中需要它。我们还看了如何在 C#中创建一个类以及如何定义一个对象。之后，我们看了类和对象之间的关系以及如何实例化一个类并使用它。我们还讨论了类中的变量和方法。最后，我们涵盖了面向对象编程的四大支柱。在下一章中，我们将学习更多关于继承和类层次结构的知识。
