# 第十五章：使用内连接加入数据集

在本章中，您将学习如何使用 LINQ 编写查询，以连接不同的结果集或不同的数据集。本章的代码并不是很复杂，只有一点点。

# 在 HTML 中添加一个连接类的按钮

打开一个项目。在 HTML 页面中放置一个按钮，上面写着连接类，放在以`<form id=....`开头的行下面。因此，我们将有两个不同的类，然后将它们连接在一起，产生一些结果，然后显示它们。这是目标：

```cs
<asp:Button ID="Button1" runat="server" Text="Join Classes" />
```

接下来，切换到设计视图，并双击连接类按钮。这将带我们进入`Default.aspx.cs`。删除`Page_Load`块。该项目的起始代码的相关部分应如*图 15.10.1*所示：

![](img/eb1eda2d-e891-4908-8db1-912e2d9669ba.png)

图 15.10.1：该项目的起始代码部分

# 添加命名空间

现在，我们将编写以下代码。我们需要 LINQ 和泛型集合命名空间；因此，在文件顶部的`using System`下面输入以下内容：

```cs
using System.Linq;
using System.Collections.Generic;
```

# 创建人员和汽车类

我们将创建两个类。一个是`person`，另一个是`car`类。为此，请在以`public partial class _Default...`开头的行的正上方直接输入以下内容：

```cs
public class Person
```

现在，我们只需要一个名字；因此，在此行下面的一对大括号之间输入以下内容：

```cs
public string Name { get; set; }
```

然后，我们还需要创建一个名为`Car`的类。因此，在前一行下面的封闭大括号下面输入以下内容：

```cs
public class Car
```

接下来，在此行下面的一对大括号之间输入以下内容：

```cs
public Person Owner { get; set; }
```

如您现在所见，`public Person` 被定义为类内字段的数据类型。例如，一辆车有一个所有者。

现在，在前一行下面添加另一个数据类型，如下所示：

```cs
public string Maker { get; set; }
```

显然，您可以看到`Car`类内有`Person`字段。这些类之间存在连接。我们很快就会用到这个。现在，让我们来构建。

# 创建人员对象

首先，我们必须创建一些`Person`对象，否则我们将没有任何东西可以连接。因此，在以`protected void Button1_Click...`开头的行下面的一对大括号之间输入以下内容：

```cs
Person per1 = new Person() { Name = "Mark Owens" };
```

现在，复制此行并将其直接粘贴到下面。编辑该行，将其更改为`Person per2`，并将`Name`变量的值更改为`Jenny Smith`：

```cs
Person per2 = new Person() { Name = "Jenny Smith" };
```

最后，复制前面的行并将其粘贴到下面。编辑该行，将其更改为`Person per3`，并将`Name`变量的值更改为`John Jenkins`：

```cs
Person per3 = new Person() { Name = "John Jenkins" };
```

所以，现在我们有一些人将成为汽车的所有者。

# 创建汽车对象

现在，让我们创建一些`car`对象。跳过一行，然后输入以下内容：

```cs
Car car1 = new Car() { Owner = per1, Maker = "Honda" };
```

要初始化`car1`，您可以从`Owner = per1`开始。这建立了两个类之间的连接；也就是说，`car1`的所有者是`per1`，即`Mark Owens`。然后，您添加制造商，我们将说`car1`的制造商是`Honda`。

再次复制此行，并将其直接粘贴到前一行下面。编辑该行，将其更改为`Car car2`，所有者更改为`per2`，但制造商保持为`Honda`：

```cs
Car car2 = new Car() { Owner = per2, Maker = "Honda" };
```

有时，不幸的是，为了阐明一个概念，我必须写相当多的代码，否则很难阐明这个概念。

再次复制前面的行并将其粘贴到下面。编辑该行，将其更改为`Car car3`，`Owner`更改为`per1`，但这次将`Maker`更改为`Toyota`：

```cs
Car car3 = new Car() { Owner = per1, Maker = "Toyota" };
```

最后，复制前面的行并将其粘贴到下面。编辑该行，将其更改为`Car car4`，`Owner`更改为`per4`，并将`Maker`更改为`Tesla`：

```cs
Car car4 = new Car() { Owner = per2, Maker = "Tesla" }; 
```

当然，要注意的是，`per3`变量并未被用作汽车的所有者，对吧？因此，当我们进行连接时，连接这两个数据集的查询将返回共享的记录。这意味着，例如，没有一辆车是由`per3`拥有的。

# 创建所有者及其汽车的列表

接下来，跳过一行，输入以下内容：

```cs
List<Person> people = new List<Person> { per1, per2, per3 };
```

在这里，我们说一个人的列表，`people`，等于一个新的人的列表，然后，我们把这些个体——`per1`，`per2`和`per3`放进去。接下来，你要对汽车做同样的事情，所以输入以下内容：

```cs
List<Car> cars = new List<Car> { car1, car2, car3, car4 };
```

再次，要初始化汽车列表，你说`car1`，`car2`，`car3`和`car4`。

# 连接所有者和汽车列表

现在，你可以连接这些列表。要做到这一点，略过一行，然后输入以下内容：

```cs
var carsWithOwners = from person in people
```

对于有所有者的汽车，你可以编写查询：`from person in people`。接下来，继续输入以下内容：

```cs
join car in cars on person equals car.Owner
```

在这里，我们正在连接这两个列表。我们使用`person`来`join people`，并将其设置为`car.Owner`。然后，一旦它们连接起来，拥有汽车的人基本上，你可以接着说以下内容：

```cs
select new{ OwnerName = person.Name, CarMake = car.Maker };
```

在这一行中创建了一个匿名类型。因此，如果你将鼠标悬停在`var`上，它会说 T 是'a。这是一个匿名数据类型。因此，`carsWithOwners`基本上是一个匿名类型的列表，但因为它是一个列表，而且是`IEnumerable`，你可以使用`foreach`循环逐步进行遍历。

# 获取并显示结果

现在我们需要获取结果。所以，略过一行，说以下内容：

```cs
foreach(var ownedCar in carsWithOwners)
```

接下来，在这条线下面的花括号之间输入以下内容：

```cs
sampLabel.Text += $"<br>Owner={ownedCar.OwnerName} Car Make={ownedCar.CarMake}";
```

这将为我们显示结果。

# 运行程序

现在在浏览器中打开这个，然后点击连接类按钮。看一下结果，也显示在*图 15.10.2*中：

![](img/cfb73cca-c128-411c-9dd3-b75c8b2e1767.png)

图 15.10.2：这个项目的结果

所以，马克·欧文斯有两辆车。接下来，珍妮·史密斯有一辆本田和一辆特斯拉。对吗？

现在，因为约翰·詹金斯是`per3`，他不会出现在汽车列表中作为所有者。这意味着`per3`和`Car`列表之间没有连接。换句话说，在 LINQ 中进行连接时，会使用`per1`，因为它是按所有者`Car.Owner`进行的。因此，将使用`per1`和`per2`，但不会使用`per3`。然后，显示结果。

# 章节回顾

回顾一下，包括注释在内的本章的`Default.aspx.cs`文件的完整版本如下所示：

```cs
//using is a directive
//System is a name space
//name space is a collection of features that our needs to run
using System;
using System.Linq;
using System.Collections.Generic;
//public means accessible anywhere
//partial means this class is split over multiple files
//class is a keyword and think of it as the outermost level of grouping
//:System.Web.UI.Page means our page inherits the features of a Page
public class Person
{
    //define Person class
    public string Name { get; set; }
}
public class Car
{
    //define Car class, using field of type Person
    public Person Owner { get; set; }
    public string Maker { get; set; }
}
public partial class _Default : System.Web.UI.Page
{
    protected void Button1_Click(object sender, EventArgs e)
    {
        //make three new people
        Person per1 = new Person() { Name = "Mark Owens" };
        Person per2 = new Person() { Name = "Jenny Smith" };
        Person per3 = new Person() { Name = "John Jenkins" };
        //make four new cars
        Car car1 = new Car() { Owner = per1, Maker = "Honda" };
        Car car2 = new Car() { Owner = per2, Maker = "Honda" };
        Car car3 = new Car() { Owner = per1, Maker = "Toyota" };
        Car car4 = new Car() { Owner = per2, Maker = "Tesla" };
        //make lists of people and cars
        List<Person> people = new List<Person> { per1, per2, per3 };
        List<Car> cars = new List<Car> { car1, car2, car3, car4 };
        //use linq to write a query that joins the two lists by car Owner
        //here, the type of var is an enumerable list of anonymous 
        //data types
        var carsWithOwners = from person in people join car in cars on person equals car.Owner
        select new { OwnerName = person.Name, CarMake = car.Maker };
        //foreach loops iterates over carsWithOwners
        foreach(var ownedCar in carsWithOwners)
        {
            sampLabel.Text += $"<br>Owner={ownedCar.OwnerName} Car Make= {ownedCar.CarMake}";
        }
    }
}
```

# 总结

在本章中，你学会了如何使用 LINQ 编写查询，连接不同的结果集或不同的数据集。你创建了`Person`和`Car`类，制作了`Person`和`Car`对象，制作了所有者及其汽车的列表，并连接了所有者和汽车列表。

在下一章中，你将使用 SQL Server 2017 Express。
