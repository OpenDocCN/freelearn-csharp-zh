# 第四章：对象协作

正如我们在前几章中看到的，面向对象编程的重点是对象。当我们使用这种方法设计软件时，我们会牢记面向对象编程的概念。我们还会尝试将软件组件分解为更小的对象，并创建对象之间的适当关系，以便它们可以共同工作，为我们提供所需的输出。对象之间的这种关系称为**对象协作**。

在本章中，我们将涵盖以下主题：

+   什么是对象协作？

+   不同类型的协作

+   什么是依赖协作？

+   什么是关联？

+   什么是继承？

# 对象协作的示例

对象协作是面向对象编程中最重要的主题之一。如果对象在程序中不相互协作，就无法实现任何目标。例如，如果我们考虑一个简单的 Web 应用程序，我们可以看到不同对象之间的关系在构建应用程序中起着重要作用。例如，Twitter 有许多对象彼此相关，以使应用程序正常运行。`User`对象包括 Twitter 用户的用户名、密码、名字、姓氏、图片和其他用户相关信息。可能还有另一个名为`Tweet`的对象，其中包括消息、日期和时间、发布推文的用户的用户名以及其他一些属性。还可能有另一个名为`Message`的对象，其中包含消息的内容、消息的发送者和接收者、日期和时间。这是对 Twitter 这样一个大型应用程序的最简单的分解；它几乎肯定包含许多其他对象。但现在，让我们只考虑这三个对象，并尝试找到它们之间的关系。

首先，我们将看一下`User`对象。这是 Twitter 中最重要的对象之一，因为它保存了用户信息。在 Twitter 中，一切都是由用户制作或为用户执行的，因此我们可以假设应该有一些其他对象需要与`User`对象有关系。现在让我们尝试看看`Tweet`对象是否与`User`对象有关系。推文是一条消息，如果`Tweet`对象是公开的，所有用户都应该能看到它。如果是私密的，只有该用户的关注者才能看到。正如我们所看到的，`Tweet`对象与`User`对象有着非常紧密的关系。因此，根据面向对象编程的方法，我们可以说`User`对象在 Twitter 应用程序中与`Tweet`对象协作。

如果我们也尝试分析`User`和`Message`对象之间的关系，我们会发现`Message`对象也与`User`对象有着非常强的关系。消息是由一个用户发送给另一个用户的；因此，没有用户，`Message`对象就没有合适的实现。

但`Tweet`和`Message`对象之间有关系吗？从已经说过的内容来看，我们可以说这两个对象之间没有关系。并不是每个对象都必须与所有其他对象相关联，但一个对象通常至少与另一个对象有关系。现在让我们看看 C#中有哪些不同类型的对象协作。

# C#中不同类型的对象协作

在编程中，对象可以以许多种方式与其他对象协作。然而，在本章中，我们只会讨论三个最重要的协作规则。

我们将首先尝试解释每种类型，看一些示例来帮助我们理解它们。如果你无法将这些概念与你的工作联系起来，你可能很难理解对象协作的重要性，但相信我，这些概念对你成为一名优秀的软件开发人员非常重要。

当你与其他人讨论软件设计时，或者当你设计自己的软件时，所有这些概念和术语都会派上用场。因此，我的建议是专注于理解这些概念，并将它们与你的工作联系起来，以便从这些信息中获益。

现在，让我们看看我们将在本章中讨论的三种协作类型，如下列表所示：

+   依赖

+   联想

+   继承

让我们想象一个应用程序，并尝试将这些协作概念与该应用程序的对象联系起来。当你能将概念与现实世界联系起来时，学习会更容易、更有趣，因此这是我们在接下来的章节中将采取的方法。

# 案例研究

由于本章的主要目标是学习对象协作涉及的概念，而不是设计一个完全成熟的、超级棒的应用程序，我们将以简单和最小的方式设计我们的对象。

对于我们的示例，我们将开发一些餐厅管理软件。这可以是豪华餐厅，也可以是人们来喝咖啡放松的小咖啡馆。在我们的情况下，我们考虑的是价格中等的餐厅。要开始构建这个应用程序，让我们考虑我们需要哪些类和对象。我们将需要一个`Food`类，一个`Chef`类，一个`Waiter`类，也许还需要一个`Beverage`类。

当你读完本章后，不要直接跳到下一章。相反，花一些时间思考一些在本章中没有提到的对象，并尝试分析你所想到的对象之间的关系。这将帮助你发展对对象协作概念的了解。记住：软件开发不是一份打字的工作，它需要大量的脑力工作。因此，你越多地思考这些概念，你在软件开发方面就会变得更加优秀。

现在，让我们看看当我考虑了应该包括在我们想象的餐厅应用程序中的对象时，我想到了哪些对象：

+   食品

+   牛肉汉堡

+   意面

+   饮料

+   可乐

+   咖啡

+   订单

+   订单项目

+   员工

+   厨师

+   服务员

+   食品存储库

+   饮料存储库

+   员工存储库

现在，有些对象可能对你来说并没有太多意义。例如，`FoodRepository`、`BeverageRepository`和`StaffRepository`对象实际上并不是业务对象，而是帮助不同模块在应用程序中相互交互的辅助对象。例如，`FoodRepository`对象将用于从数据库和 UI 保存和检索`Food`对象。同样，`BeverageRepository`对象将处理饮料。我们还有一个名为`Food`的类，它是一种通用类型的类，以及更具体的食品对象，如`Beef Burger`和`Pasta`。这些对象是`Food`对象的子类别。作为软件开发人员，我们已经确定了开发此软件所需的对象。现在，是时候以解决软件将被用于的问题的方式使用这些对象了；然而，在我们开始编写代码之前，我们需要了解并弄清楚对象之间如何关联，以便应用程序能够达到最佳状态。让我们从依赖关系开始。

# 依赖

当一个对象使用另一个无关的对象来执行任务时，它们之间的关系被称为**依赖**。在软件世界中，我们也将这种关系称为**使用关系**。现在，让我们看看我们为餐厅应用程序所考虑的对象之间是否存在任何依赖关系。

如果我们分析一下`FoodRepository`对象，它将从数据库中保存和检索`Food`对象并将其传递给 UI，我们可以说`FoodRepository`对象必须使用`Food`对象。这意味着`Food`和`FoodRepository`对象之间的关系是一种依赖关系。如果我们考虑在前端创建新的`Food`对象时的流程，该对象将被传递给`FoodRepository`。然后，`FoodRepository`将把`Food`对象序列化为数据库数据以便将其保存在数据库中。如果`FoodRepository`不使用`Food`对象，那它怎么知道要序列化和存储在数据库中的内容呢？在这里，`FoodRepository`必须与`Food`对象存在依赖关系。让我们看看这段代码：

```cs
public class Food {
 public int? FoodId {get;set;}
 public string Name {get;set;}
 public decimal Price {get;set;}
}

public class FoodRepository {
 public int SaveFood(Food food){
 int result = SaveFoodInDatabase(food);
 return result;
 }

 public Food GetFood(int foodId){
 Food result = new Food();
 result = GetFoodFromDatabaseById(foodId);
 return result;
 }
}
```

在上面的代码中，我们可以看到`FoodRepository`类有两个方法。一个方法是`SaveFood`，另一个是`GetFood`。

`SaveFood`方法涉及获取一个`Food`对象并将其保存在数据库中。在将食品项目保存在数据库后，它将新创建的`foodId`返回给`FoodRepository`。然后，`FoodRepository`将新创建的`FoodId`传递给 UI，通知用户食品项目创建成功。另一方面，另一个`GetFood`方法从 UI 获取一个 ID 作为参数，并检查该 ID 是否是有效输入。如果是，`FoodRepository`将`FoodId`传递给`databasehandler`对象，该对象在数据库中搜索食品并将其映射回作为`Food`对象。之后，将`Food`对象返回给视图。

在这里，我们可以看到`FoodRepository`对象需要使用`Food`对象来完成其工作。这种关系称为**依赖关系**。我们还可以使用*uses a*短语来识别这种关系。`FoodRepository`使用`Food`对象来保存食品在数据库中。

与`FoodRepository`一样，`BeverageRepository`对`Beverage`对象做了同样的事情：它在数据库和 UI 中保存和检索饮料对象。现在让我们看看`BeverageRepository`的代码是什么样的：

```cs
public class Beverage {
    public int? BeverageId {get;set;}
    public string Name { get;set;}
    public decimal Price {get;set;}
}

public class BeverageRepository {
    public int SaveBeverage(Beverage beverage){
        int result = SaveBeverageInDatabase(beverage);
        return result;
    }

public Beverage GetBeverage(int beverageId) {
        Beverage result = new Beverage();
        result = GetBeverageFromDatabaseById(beverageId);
        return result;
    }
}
```

如果你看一下前面的代码，你会发现`BeverageRepository`有两个方法：`SaveBeverage`和`GetBeverage`。这两个方法都使用`Beverage`对象。这意味着`BeverageRepository`与`Beverage`对象存在依赖关系。

现在让我们来看一下我们迄今为止创建的两个类，如下所示的代码：

```cs
public class FoodRepository {
    public int SaveFood(Food food){
        int result = SaveFoodInDatabase(food);
        return result;
    }

    public Food GetFood(int foodId){
        Food result = new Food();
        result = GetFoodFromDatabaseById(foodId);
        return result;
    }
}

public class BeverageRepository {
    public int SaveBeverage(Beverage beverage){
        int result = SaveBeverageInDatabase(beverage);
        return result;
    }

public Beverage GetBeverage(int beverageId){
        Beverage result = new Beverage();
        result = GetBeverageFromDatabaseById(beverageId);
        return result;
    }
}
```

一个对象可以使用依赖关系与多个对象相关联。在面向对象编程中，这种关系非常常见。

让我们来看另一个依赖关系的例子。`程序员`和`计算机`之间的关系可能是一种依赖关系。怎么样？我们知道`程序员`很可能是一个人，而`计算机`是一台机器。`程序员`使用`计算机`来编写计算机程序，但`计算机`不是`程序员`的属性。`程序员`*使用*计算机，并且这不一定是一个特定的计算机——可以是任何计算机。那么我们可以说`程序员`和`计算机`之间的关系是一种依赖关系吗？是的，我们当然可以。让我们看看如何在代码中表示这一点：

```cs
public class Programmer {
    public string Name { get; set; }
    public string Age { get; set; }
    public List<ProgrammingLanguages> ProgrammingLanguages { get; set; }
    public ProgrammerType Type { get; set; } // Backend/Frontend/Full Stack/Web/Mobbile etc

    public bool WorkOnAProject(Project project, Computer computer){
        // use the provided computer to do the project
        // here we can see that the programmer is using a computer
    }
}

public class Computer {
    public int Id { get; set; }
    public string ModelNumber { get; set; }
    public Company Manufacturer { get; set; }
    public Ram Ram { get; set; }
    public MotherBoard MotherBoard { get; set; }
    public CPU CPU { get; set; }
}
```

在上面的例子中，我们可以清楚地看到`程序员`和`计算机`之间有一种依赖关系，但这并不总是如此：这取决于你如何设计你的对象。如果你设计了`程序员`类，使得每个程序员都必须有一台专用的计算机，你可以在`程序员`类中使用`计算机`作为属性，那么程序员和计算机之间的关系将会改变。因此，关系取决于对象的设计方式。

我在本节的主要目标是澄清依赖关系。我希望依赖关系的本质现在对你来说是清楚的。

现在让我们看看依赖关系在**统一建模语言**（**UML**）图表中是如何绘制的，如下图所示：

![](img/4ad50b5f-04ab-4fb3-829c-3e23eef738c8.png)

用实线表示依赖关系。

# 关联

另一种关系类型是关联关系。这种关系类型不同于依赖关系。在这种关系类型中，一个对象知道另一个对象并与之相关联。这种关系是通过将一个对象作为另一个对象的属性来实现的。在软件社区中，这种关系类型也被称为*拥有*关系。例如，汽车拥有引擎。如果你能想到任何可以用*拥有*短语来关联的对象，那么这种关系就是关联关系。在我们的汽车例子中，引擎是汽车的一部分。没有引擎，汽车无法执行任何功能。虽然引擎本身是一个独立的对象，但它是汽车的一部分，因此汽车和引擎之间存在关联。

这种关联关系可以分为以下两类：

+   聚合

+   组合

让我们看看这两种关系是什么，它们之间有什么不同。

# 聚合

当一个对象在其属性中有另一个独立的对象时，这被称为**聚合关系**。让我们看看前一节中的例子，试着看看这是否是一个聚合关系。

前面的例子是关于汽车和引擎之间的关系。我们都知道汽车必须有引擎，这就是为什么引擎是汽车的属性，如下代码所示：

```cs
public class Car {
    public Engine Engine { get; set; }
    // Other properties and methods
}
```

现在的问题是，这种关系是什么类型？决定因素是引擎是一个独立的对象，可以独立于汽车运行。制造商在制造汽车的其他零件时并不制造引擎：他们可以单独制造它。即使没有汽车，引擎也可以进行测试，甚至用于其他目的。因此，我们可以说汽车与引擎之间的关系是一种*聚合关系*。

现在让我们来看一下我们的餐厅管理软件的例子。如果我们分析`Food`和`Chef`对象之间的关系，很明显没有厨师就没有食物。必须有人来烹饪、烘焙和准备食物，食物本身无法做到这一点。因此，我们可以说食物有厨师。这意味着`Food`对象应该有一个名为`Chef`的属性，用来保存该`Food`的`Chef`对象。让我们来看一下这种关系的代码：

```cs
public class Food {
    public int? FoodId {get;set;}
    public string Name { get; set; }
    public string Price { get; set; }
    public Chef Chef { get; set; }
}
```

如果我们考虑`Beverage`对象，每种饮料都必须有一个公司或制造商。例如，商业饮料是由百事公司、可口可乐公司等公司生产的。这些公司生产的饮料是它们的合法财产。饮料也可以在本地制造，这种情况下公司名称将是当地商店的名称。然而，这里的主要观点是饮料必须有一个制造商公司。让我们看看`Beverage`类在代码中是什么样子的：

```cs
public class Beverage {
    public int? BeverageId {get;set;}
    public string Name { get; set; }
    public string Price { get; set; }
    public Manufacturer Manufacturer { get; set; }
}
```

在这两个例子中，`Chef`和`Manufacturer`对象都是`Food`和`Beverage`的属性。我们也知道`Chef`或`Manufacturer`公司是独立的。因此，`Food`和`Chef`之间的关系是一种聚合关系。`Beverage`和`Manufacturer`也是如此。

为了让事情更清晰，让我们看另一个聚合的例子。我们用于编程或执行任何其他任务的计算机由不同的组件组成。我们有主板、RAM、CPU、显卡、屏幕、键盘、鼠标和许多其他东西。一些组件与计算机具有聚合关系。例如，主板、RAM 和 CPU 是构建计算机所需的内部组件。所有这些组件都可以独立存在于计算机之外，因此所有这些组件都与计算机具有聚合关系。让我们看看`Computer`类如何与`MotherBoard`类相关联的以下代码：

```cs
public class Computer {
    public int Id { get; set; }
    public string ModelNumber { get; set; }
    public Company Manufacturer { get; set; }
    public Ram Ram { get; set; }
    public MotherBoard MotherBoard { get; set; }
    public CPU CPU { get; set; }
}

public class Ram {
    // Ram properties and methods
}

public class CPU {
    // CPU properties and methods
}

public class MotherBoard {
    // MotherBoard properties and methods
}
```

现在，让我们看看在 UML 图中如何绘制聚合关系。如果我们尝试用 RAM、CPU 和主板显示前面的计算机类聚合关系，那么它看起来会像下面这样：

![](img/e0467b14-4fd0-4131-a993-b34defc2694b.jpg)

实线和菱形用于表示聚合关系。菱形放在持有属性的类的一侧，如下图所示：

![](img/a1c0dbc5-417b-4e68-86b3-c2d464f80da0.jpg)

# 组合

组合关系是一种关联关系。这意味着一个对象将另一个对象作为其属性，但与聚合的不同之处在于，在组合中，作为属性的对象不能独立存在；它必须借助另一个对象才能发挥作用。如果我们考虑`Chef`和`Manufacturer`类，这些类的存在并不完全依赖于`Food`和`Beverage`类。相反，这些类可以独立存在，因此具有聚合关系。

然而，如果我们考虑`Order`和`OrderItem`对象之间的关系，我们会发现`OrderItem`对象没有没有`Order`就没有意义。让我们看一下`Order`类的以下代码：

```cs
public class Order {
    public int OrderId { get; set; }
    public List<OrderItem> OrderItems { get; set; }
    public DateTime OrderTime { get; set; }
    public Customer Customer { get; set; }
}
```

在这里，我们可以看到`Order`对象中有一个`OrderItems`列表。这些`OrderItems`是顾客订购的`Food`项目。顾客可以订购一个菜或多个菜，这就是为什么`OrderItems`是一个列表类型。现在是时候证明我们的想法了。`OrderItem`是否真的与`Order`有组合关系？我们有没有犯任何错误？我们是否把聚合关系当作组合关系了？

要确定它是哪种类型的关联关系，我们必须问自己一些问题。`OrderItem`可以在没有`Order`的情况下存在吗？如果不能，那为什么？它是一个独立的对象！然而，如果你再深入思考一下，你会意识到没有`Order`，没有`OrderItem`可以存在，因为顾客必须订购商品，没有`Order`对象，`OrderItem`对象就无法跟踪。`OrderItem`无法提供给任何顾客，因为没有关于`OrderItem`是为哪个顾客的数据。因此，我们可以说`OrderItem`与`Order`对象有组合关系。

让我们看另一个组合的例子。在我们的学校系统中，我们有学生、老师、科目和成绩，对吧？现在，我会说`Subject`对象和`Grade`对象之间的关系是组合关系。让我证明我的答案。看看这两个类的以下代码：

```cs
public class Subject {
    public int Id { get; set; }
    public string Name { get; set; }
    public Grade Grade { get; set; }
}

public class Grade {
    public int Id { get; set; }
    public double Mark { get; set; }
    public char GradeSymbol { get; set; } // A, B, C, D, F etc
}
```

在这里，我们可以看到`Grade`对象保存了学生在特定科目的考试成绩。它还保存了`GradeSymbol`，比如`A`，`B`或`F`，取决于学校的评分规则。我们可以在`Subject`类中看到有一个叫做`Grade`的属性。这个属性保存了特定`Subject`对象的成绩。如果我们只是单独考虑`Grade`而不是与`Subject`类关联，我们会有点困惑，想知道成绩是为哪个科目的。

因此，`Grade`和`Subject`之间的关系是组合关系。

让我们看看如何在 UML 图中展示组合关系，使用`Subject`和`Grade`的前面的例子：

![](img/21c4e007-c706-470e-9cb2-be5c87ffceae.jpg)

使用实线和黑色菱形表示组合关系。菱形放置在持有属性的类的一侧：

![](img/44e0de3f-109f-439e-a5a6-c1b9c0288418.png)

# 继承

这是面向对象编程的四大支柱之一。**继承**是一个对象继承或重用另一个对象的属性或方法。被继承的类称为**基类**，继承基类的类通常称为**派生类**。继承关系可以被视为一个*是一个*的关系。例如，意大利面是一种`Food`。`Pasta`对象在数据库中有一个唯一的 ID，还有其他属性，比如名称、价格和厨师。因此，由于`Pasta`满足`Food`类的所有属性，它可以继承`Food`类并使用`Food`类的属性。让我们看一下代码：

```cs
public class Pasta : Food {
    public string Type { get; set; }
    public Sauce Sauce { get; set; }
    public string[] Spices { get; set; }
}
```

对于饮料也是一样的。例如，`Coffee`是一种饮料，具有`Beverage`对象具有的所有属性。咖啡有名称和价格，可能有糖、牛奶和咖啡豆。让我们编写`Coffee`类，看看它是什么样子的：

```cs
public class Coffee : Beverage {
    public int Sugar { get; set; }
    public int Milk { get; set; }
    public string LocationOfCoffeeBean { get; set; }
}
```

因此，我们可以说`Coffee`正在继承`Beverage`类。在这里，`Coffee`是派生类，`Beverage`是基类。

在之前的例子中，我们使用了`Programmer`对象。在这种情况下，你认为`Programmer`类实际上可以继承`Human`类吗？是的，当然可以。在这个例子中，程序员无非就是一个人。如果我们看一下`Programmer`的属性和`Human`的属性，我们会发现有一些共同的属性，比如姓名、年龄等。因此，我们可以修改`Programmer`类的代码如下：

```cs
public class Programmer : Human {
 // Name, Age properties can be inherited from Human
 public List<ProgrammingLanguages> ProgrammingLanguages { get; set; }
 public ProgrammerType Type { get; set; } // Backend/Frontend/Full Stack/Web/Mobbile etc

 public bool WorkOnAProject(Project project, Computer computer){
 // use the provided computer to do the project
 // here we can see that the programmer is using a computer
 }
}
```

现在，让我们看看如何为我们的`Programmer`类绘制 UML 图：

![](img/3db18b8b-3f8e-460b-bf50-c4bd42c833d5.png)

继承由一条实线和一个三角形符号表示。这个三角形指向超类的方向：

![](img/f549a7f4-2baa-4c24-865b-5fcfdc37d65d.png)

# 总结

我们在本章中看到的对象协作类型是 C#中最常用的类型。在设计应用程序或架构软件时，对象协作非常重要。它将定义软件的灵活性，可以添加多少新功能，以及维护代码的难易程度。对象协作非常重要。

在下一章中，我们将讨论异常处理。这也是编程中非常重要的一部分。
