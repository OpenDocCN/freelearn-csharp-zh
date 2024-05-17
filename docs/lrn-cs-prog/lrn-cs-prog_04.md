# 第四章：*第四章：*理解各种用户定义类型

在上一章中，我们学习了 C#中的控制语句和异常。在本章中，我们将探讨 C#中的用户定义类型。我们将学习如何使用类、结构和枚举来创建自定义用户类型。我们将探讨类中的字段、属性、方法、索引器和构造函数。我们将研究 C#中的访问修饰符，并学习如何使用它们来定义类型和成员的可见性。我们还将学习 C#中的两个重要关键字`this`和`static`，并了解方法的`ref`、`in`和`out`参数修饰符。

我们将详细探讨以下主题：

+   类和对象

+   结构

+   枚举

+   命名空间

对这些概念的良好了解对于理解我们将在下一章中涵盖的**面向对象编程**（**OOP**）概念是必要的。

# 类和对象

在我们继续之前，重要的是你理解这两个关键概念。类是指定对象形式的模板或蓝图。它包含操作该数据的数据和代码。对象是类的一个实例。类是使用`class`关键字和一个类的类型是引用类型来定义的。引用类型的变量的默认值是`null`。您可以将其分配为类型实例的引用。实例 - 也就是对象 - 是使用`new`运算符创建的。

信息框

术语*类*和*对象*在不同的技术文档中经常可以互换使用。它们并不相同，这样使用是不正确的。类是指定对象的内存布局并定义与该内存操作的功能的蓝图。对象是根据蓝图创建和操作的实际实体。

看一下以下代码片段，以了解如何定义类。在这个例子中，我们创建了一个`Employee`类，其中包含三个字段，用于表示员工的 ID、名字和姓氏：

```cs
class Employee
{
    public int    EmployeeId;
    public string FirstName;
    public string LastName;
}
```

我们将使用`new`关键字来创建类的实例。`new`运算符在运行时为对象分配内存并返回对其的引用。然后，将该引用存储在指定对象名称的变量中。对象存储在堆上，对象的引用存储在与命名变量对应的堆栈存储位置上。

要创建`Employee`类的对象，我们将使用以下语句：

```cs
Employee obj = new Employee();
```

使用对象访问类的成员（字段、属性、方法），我们使用点（`.`）运算符。因此，要为对象的字段赋值（`obj`），我们将使用以下语句：

```cs
obj.EmployeeId = 1;
obj.FirstName = "John";
obj.LastName = "Doe"
```

以下图表概念上显示了这里发生的情况：

![图 4.1 - 先前雇员对象的概念内存布局](img/Figure_4.1._B12346.jpg)

图 4.1 - 先前雇员对象的概念内存布局

**Employee**类型的**obj**变量被分配在堆栈上。但是，堆栈不包含实际的**Employee**对象，而只包含对它的引用。对象分配在堆上，并且对象的地址存储在堆栈上，因此通过**obj**变量我们可以访问位于堆上的对象。

类的两个不同实例是两个不同的对象。对象的引用可以分配给多个变量。在这种情况下，通过一个变量对对象的修改将通过另一个变量可见。这在以下示例中显示：

```cs
Employee obj1 = new Employee();
obj1.EmployeeId = 1;
Employee obj2 = obj1; 
obj2.FirstName = "John";    // obj1.FirstName == "John"
obj2.LastName = "Doe";      // obj1.LastName == "Doe"
```

在这里，我们创建了`Employee`类的第一个实例，并且只为`EmployeeId`赋了一个值。然后，我们创建了第二个实例，并为名字和姓氏赋值，跳过了标识符。这是两个不同的对象，驻留在内存中的不同位置。

员工的属性存储在类的成员字段中。接下来将讨论这些。

## 字段

这些是直接在类内声明的变量，因此是类的成员。字段用于存储对象的状态，这是必须在类方法执行期间存活并且应该从多个方法中访问的数据。不在单个方法范围之外使用的变量应该被定义为局部变量而不是类字段。

在前面的部分中，`EmployeeId`、`FirstName`和`LastName`是提到的字段。这些被称为**实例字段**，因为它们属于类的实例，这意味着每个对象都有自己的这些字段的实例。另一方面，*静态字段*属于类，并且被所有类的实例共享。静态成员将在本章的后面部分讨论。

这些字段已被声明为`public`，这意味着任何人都可以访问它们。然而，这是一个不好的做法。字段通常应该声明为`private`（只能被类成员访问）或者`protected`（也可以被派生类访问）。这确保了更好的封装，这将在下一章进一步讨论。字段可以通过方法、属性和索引器进行读取和写入。我们将在下面的部分讨论这些。

用`const`修饰符声明的字段称为**常量**。只有内置类型可以是常量。常量始终使用字面值初始化，并且是在编译时已知的值，不能在运行时更改：

```cs
class Employee
{
    public const int StartId = 100;
}
```

常量字段在中间语言代码中被其字面值替换，这意味着不能通过引用传递常量字段。但这还有另一个更微妙的含义：如果常量值在类型定义的程序集之外的程序集中被引用，并且常量的字面值在将来的版本中被更改，那么引用常量的程序集将继续具有旧版本，直到它们被重新编译。

例如，如果在 A 程序集中定义了一个整数常量并且初始值为 42，然后在 B 程序集中引用了它，那么值 42 将被存储在 B 程序集中。将常量的值更改为其他值（比如 100）将不会反映在 B 程序集中，它将继续存储旧值，直到使用新版本的 A 程序集重新编译。

字段也可以用`readonly`修饰符声明。这些字段只能在构造函数中初始化，它们的值以后不能被改变。它们可以被看作是**运行时常量**。

在下面的例子中，`EmployeeId`字段是一个在构造函数中初始化的`readonly`字段。类的实例只能改变姓和名字段：

```cs
class Employee
{
   public readonly int EmployeeId;
   public string       FirstName;
   public string       LastName;
   public Employee(int id)
   {
      EmployeeId = id;
   }
}
Employee obj = new Employee(1);
obj.FirstName = "John";
obj.LastName = "Doe";
```

现在我们已经看到如何使用字段，让我们学习一下方法。

## 方法

方法是在调用方法时执行的一个或多个语句的系列。实例方法需要对象才能被调用。静态方法属于类，不使用对象调用。

方法有一个所谓的*签名*，由几个部分组成：

+   默认为`private`。

+   `virtual`、`abstract`、`sealed`或`static`：这些都是可选的，将在后面的部分讨论。

+   如果方法不返回任何值，则为`void`。

+   **名字**：这必须是一个有效的标识符。

+   `ref`、`in`或`out`修饰符。

在下面的例子中，我们将在`Employee`类中添加一个方法：

```cs
class Employee
{
    public int    EmployeeId;
    public string FirstName;
    public string LastName;
    public string GetEmployeeName()
    {
        return $"{FirstName} {LastName}";
    }
}
```

在这里，我们添加了一个名为`GetEmployeeName()`的方法。访问修饰符是`public`，这允许从代码的任何部分调用这个方法。返回类型是`string`，因为该方法通过连接`FirstName`和`LastName`字段并用空格分隔返回员工的名字。

简单地评估表达式并可能返回评估结果的方法可以使用另一种语法`member => expression;`形式编写，并且支持所有类成员，不仅仅是方法，还包括字段、属性、索引器、构造函数和终结器。表达式评估的结果值类型必须与方法的返回类型匹配。

以下代码显示了使用表达式体定义的`GetEmployeeName()`方法的实现：

```cs
public string GetEmployeeName() => $"{FirstName} {LastName}";
```

**重载方法**是具有相同名称但不同签名的多个方法。这样的方法是可以存在的。在方法重载的上下文中，这些方法的返回类型不是签名的一部分。这意味着你不能有两个具有相同参数列表但不同返回值的方法。

在以下示例中，`GetEmployeeName(bool)`是前一个`GetEmployeeName()`方法的重载方法：

```cs
public string GetEmployeeName(bool lastNameFirst) => lastNameFirst ? $"{LastName} {FirstName}" : 
                $"{FirstName} {LastName}";
```

这个方法与前一个方法同名，但参数列表不同。它接受一个布尔值，指示是否应该先放姓氏，否则返回名字和姓氏，就像前一个方法一样。

## 构造函数

构造函数是在类中定义的特殊方法，当我们实例化一个类的对象时会调用它。构造函数用于在对象创建时初始化类的成员。构造函数不能有返回类型，并且与类同名。可以存在具有不同参数的多个构造函数。

没有任何参数的构造函数称为*默认构造函数*。编译器为所有类提供了这样的构造函数。默认构造函数在编译时创建并将成员变量初始化为它们的默认值。对于数值数据类型，默认值为 0，对于`bool`类型为`false`，对于引用类型为`null`。如果我们定义自己的构造函数，编译器将不再提供默认构造函数。

构造函数可以有访问修饰符。构造函数的默认访问修饰符是`private`。然而，这个修饰符使得在类外部无法实例化类本身。在大多数情况下，构造函数的访问修饰符被定义为`public`，因为构造函数通常是从类的外部调用的。

私有构造函数在某些情况下很有用。一个例子是在实现单例模式时。

让我们尝试通过以下示例来理解到目前为止涵盖的所有概念：

```cs
class Employee
{
    public int EmployeeId;
    public string FirstName;
    public string LastName;
    public Employee(int employeeId, 
                    string firstName, string lastName)
    {
        EmployeeId = employeeId;
        FirstName = firstName;
        LastName = lastName;
    }
    public string GetEmployeeName() => 
           $"{FirstName} {LastName}";   
}
```

我们扩展了`Employee`类并在其中包含了一个构造函数。这个构造函数将接受三个参数来初始化所有三个字段的值：`EmployeeId`、`FirstName`和`LastName`。在创建类的实例时，必须为类的构造函数指定适当的参数：

```cs
Employee obj = new Employee(1, "John", "Doe");
Console.WriteLine("Employee ID is: {0}", obj.EmployeeID);
Console.WriteLine("The full name of employee is: {0}",
                   obj.GetEmployeeName());
```

执行后，该程序将给出以下截图中显示的输出：

![图 4.2 - 显示前面片段输出的控制台截图](img/Figure_4.2._B12346.jpg)

图 4.2 - 显示前面片段输出的控制台截图

对象可以使用所谓的*对象初始化器*以声明方式进行初始化。您调用一个构造函数，并且除了为构造函数提供必要的参数之外，还为可访问成员（如字段、属性或索引器）提供一个初始化语句列表，放在花括号内。

考虑`Employee`类没有用户定义的构造函数，由编译器提供的默认（无参数）构造函数，我们可以编写以下代码来初始化类的实例：

```cs
Employee obj = new Employee()
{
    EmployeeId = 1,
    FirstName = "John",
    LastName = "Doe"
};
```

到目前为止，在本章中，我们已经使用字段来存储对象的状态。C#语言提供了一种替代字段的方法：属性，这是下一节的主题。

## 属性

属性是字段和访问该字段的方法的组合。它们看起来像字段，但实际上是称为**访问器**的方法。属性使得以简单的方式读取或写入类状态成为可能，并隐藏实现细节，包括验证代码。

属性定义的两个访问器称为`get`（用于从属性返回值）和`set`（用于分配新值）。在`set`访问器的上下文中，`value`关键字定义正在访问的值（即从用户代码分配的值）。

在下面的例子中，本章前面显示的`Employee`类被重写，以便员工 ID、名和姓是私有字段，通过属性对类客户端可用：

```cs
class Employee
{
   private int employeeId;
   private string firstName;
   private string lastName;
   public int EmployeeId
   {
      get { return employeeId; }
      set { employeeId = value; }
   }
   public string FirstName
   {
      get { return firstName; }
      set { firstName = value; }
   }
   public string LastName
   {
      get { return lastName; }
      set { lastName = value; }
   }
}
```

实际上，使用属性的`get`和`set`访问器是透明的。您不会显式调用它们，而是像字段一样使用属性。以下示例显示了如何访问`Employee`类的三个属性进行读写：

```cs
Employee obj = new Employee();
obj.EmployeeId = 1;
obj.FirstName = "John";
obj.LastName = "Doe";
Console.WriteLine($"{obj.EmployeeId} - {obj.LastName}, {obj.FirstName}");
```

在前面的代码中显示的属性的实现是直接的——它只返回或设置私有字段的值。但是，访问器就像任何其他方法一样，因此您可以编写任何代码，例如参数验证，如下例所示：

```cs
public int EmployeeId
{
    get { return employeeId; }
    set {
        if (value < 0)
            throw new ArgumentException(
              "ID must be greater than zero.");
        employeeId = value;
    }
}
```

另一方面，属性不需要引用相应的字段。属性可以返回不从一个字段中读取的值，或者可以从评估不同字段计算出的值。以下示例显示了一个`Name`属性，它连接了`FirstName`和`LastName`属性的值：

```cs
public string Name
{
    get { return $"{FirstName} {LastName}"; }
}
```

请注意，在这个属性的情况下，缺少`set`访问器。`get`和`set`访问器都是可选的。但是，至少必须实现一个。另一方面，只写属性没有太多价值，您可能希望将这些功能实现为常规方法。此外，`get`和`set`访问器可以具有不同的访问修饰符。

以这种方式实现属性是很麻烦的，因为您需要明确定义除了属性之外其他地方不使用的私有字段。此外，每个属性都必须明确实现`get`和`set`访问器，基本上是一遍又一遍地重复相同的代码。可以使用*自动实现的属性*以更短的语法实现相同的结果。这些属性是编译器将提供私有字段和`get`和`set`访问器的实现，就像我们之前做的那样。

`Employee`类使用自动实现的属性进行了重写，如下所示。这非常类似于我们第一次实现时使用公共字段的情况：

```cs
class Employee
{
    public int EmployeeId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```

如果您不想设置这些属性的值，可以只声明`get`访问器为`public`。在这种情况下，`set`访问器可能是`private`，并且您将通过构造函数提供值。这里显示了一个示例：

```cs
class Employee
{
    public int EmployeeId { get; private set; }
    public string FirstName { get; private set; }
    public string LastName { get; private set; }
    public Employee(int id, string firstName, string lastName)
    {
        EmployeeId = id;
        FirstName = firstName;
        LastName = lastName;
    }
}
```

可以使用表达式主体定义来实现属性。前面显示的`Name`属性可以实现如下：

```cs
public string Name => $"{FirstName} {LastName}";
```

这是一个只有`get`访问器的只读属性。但是，您可以显式实现`get`和`set`访问器作为表达式主体成员。这在以下清单中显示：

```cs
class Employee
{
    private int employeeId;
    public int EmployeeId
    {
        get => employeeId;
        set => employeeId = value > 0 ? value : 
                 throw new ArgumentException(
                      "ID must be greater than zero.");
    }
}
```

自动实现的属性也可以使用以下示例中显示的语法进行初始化：

```cs
class Employee
{
   public int EmployeeId { get; set; } = 1;
   public string FirstName { get; set; }
   public string LastName { get; set; }
}
```

`EmployeeId`属性的值被初始化为`1`。除非另行明确设置，`Employee`类的所有实例都将`EmployeeId`设置为`1`。

如果使用表达式主体定义实现只读属性，则不需要指定`get`访问器。在这种情况下，语法如下：

```cs
class Employee
{
    public int EmployeeId => 1;
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```

然而，这看起来与以下内容非常相似：

```cs
class Employee
{
    public int EmployeeId = 1;
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```

这些语法之间存在很大的区别：

+   在前面的例子中，使用`=>`时，`EmployeeId`是一个具有*表达式体定义*的*只读公共属性*。

+   在后面的例子中，使用`=`时，`EmployeeId`是一个*公共字段*，具有*初始化程序*。

有一种特殊形式的属性可以接受参数，并允许使用`[]`运算符访问类实例。这些被称为**索引器**，将在下一节中讨论。

## 索引器

索引器允许像数组一样对对象进行索引。索引器定义了`get`和`set`访问器，类似于属性。索引器没有显式名称。它是通过使用`this`关键字创建的。索引器有一个或多个参数，可以是任何类型。与属性一样，`get`和`set`访问器通常很简单，由一个返回或设置值的单个语句组成。

在下面的例子中，`ProjectRoles`类包含项目 ID 和员工在每个项目中担任的角色的映射。这个映射是私有的，但可以通过索引器访问：

```cs
class ProjectRoles
{
    readonly Dictionary<int, string> roles = 
        new Dictionary<int, string>();
    public string this[int projectId]
    {
        get
        {
            if (!roles.TryGetValue(projectId, out string role))
                throw new Exception("Project ID not found!");
            return role;
        }
        set
        {
            roles[projectId] = value;
        }
    }
}
```

索引器使用`public string this[int projectId]`语法进行定义，其中包含以下内容：

+   访问修饰符

+   索引器的类型，即`string`

+   `this`关键字和方括号`[]`中的参数列表

`get`和`set`访问器的实现方式与常规属性相同。`ProjectRoles`类可以在`Employee`类中如下使用：

```cs
class Employee
{
    public int          EmployeeId { get; set; }
    public string       FirstName { get; set; }
    public string       LastName { get; set; }
    public ProjectRoles Roles { get; private set; }
    public Employee() => Roles = new ProjectRoles();
}
```

我们可以使用`Roles[i]`语法访问员工角色，就好像`Roles`是一个数组一样。在这个例子中，参数不是数组中的索引，而是一个项目标识符，实际上是项目和角色字典的键。参数可以是任何类型，不仅仅是数值类型：

```cs
Employee obj = new Employee()
{
    EmployeeId = 1,
    FirstName = "John",
    LastName = "Doe"
};
obj.Roles[1] = "Dev";
obj.Roles[3] = "SA";

for(int i = 1; i <= 3; ++i)
{
    try
    {
        Console.WriteLine($"Project {i}: role is {obj.Roles[i]}");
    }
    catch(Exception ex)
    {
        Console.WriteLine(ex.Message);
    }
}
```

执行此示例代码的输出显示在以下截图中：

![图 4.3 - 执行前面代码片段的控制台输出](img/Figure_4.3._B12346.jpg)

图 4.3 - 执行前面代码片段的控制台输出

索引器，包括只读索引器，可以使用表达式体定义。但是，没有自动实现的索引器；它们必须显式实现。

如前所述，索引器是使用`this`关键字定义的。但是，这个关键字在索引器范围之外还有其他意义。这个主题将在下一节中讨论。

## this 关键字

`this`关键字用于表示类的当前实例。当调用方法时，使用`this`将调用对象的引用传递给它。这不是显式地完成的，而是由编译器在后台完成的。

`this`关键字有两个重要的目的：

+   当参数或局部变量具有相同名称时，限定类成员

+   将对当前实例的引用作为参数传递给另一个方法

让我们看看`Employee`类的以下实现：

```cs
class Employee
{
    public int EmployeeID;
    public string FirstName;
    public string LastName;
    public Employee(int EmployeeID, 
                    string FirstName, string LastName)
    {
       this.EmployeeID = EmployeeID;
       this.FirstName = FirstName;
       this.LastName = LastName;
    }
}
```

在这个例子中，构造函数的参数与类的字段具有相同的名称。C#允许我们对参数和实例变量使用相同的名称。由于参数名称是方法局部的，局部名称优先于实例变量。为了缓解这种情况，我们使用`this`关键字来引用当前方法调用的实例变量。

到目前为止，我们已经看到`this`关键字用于引用类的当前实例和声明索引。但是，它还用于另一个目的，即声明扩展方法。这将在*第八章* *高级主题*中讨论。现在，让我们看看另一个重要的关键字：`static`。

## 静态关键字

`static`关键字可用于声明类或类成员。这与我们目前所见的不同，因为您不创建静态类的实例，也不需要对象来访问静态成员。我们将在以下小节中详细探讨这些内容。

### 静态成员

字段、属性、方法和构造函数可以声明为`static`。索引器和终结器不能声明为`static`。静态成员不属于对象（如实例成员的情况），而是属于类型本身。因此，您不能通过对象访问静态成员，而是通过类型名称。

在下面的示例中，我们有一个`Employee`类的实现，其中有一个名为`id`的静态字段和一个名为`Create()`的静态方法。静态字段存储下一个员工 ID 的值，静态方法用于创建类的新实例，因为构造函数是`private`，因此只能从类内部调用：

```cs
class Employee
{
    private static int id = 1;
    public int EmployeeId { get; private set; }
    public string FirstName { get; private set; }
    public string LastName { get; private set; }
    private Employee(int id, string firstName, string lastName)
    {
        EmployeeId = id;
        FirstName = firstName;
        LastName = lastName;
    }
    public static Employee Create(string firstName, 
                                  string lastName)
    {
        return new Employee(id++, firstName, lastName);
    }
}
```

我们可以按以下方式调用`Create()`方法来实例化这个类的新对象：

```cs
Employee obj1 = Employee.Create("John", "Doe");
Employee obj2 = Employee.Create("Jane", "Doe");
Console.WriteLine($"{obj1.EmployeeId} {obj1.FirstName}");
Console.WriteLine($"{obj2.EmployeeId} {obj2.FirstName}");
```

像这样创建的第一个对象将`EmployeeID`设置为`1`，第二个对象将`EmployeeID`设置为`2`，依此类推。请注意，我们使用了`Employee.Create()`语法来调用静态方法。

### 静态类

`static`类也使用`static`关键字声明。`static`类不能被实例化。由于我们无法创建`static`类的实例，因此我们使用类名本身来访问类成员。静态类的所有成员本身必须是静态的。静态类基本上与非静态类相同，具有`private`构造函数和所有成员声明为`static`。

静态类通常用于定义仅在其参数（如果有）上操作并且不依赖于类字段的方法。这通常是实用类的情况。

下面的示例显示了一个名为`MassConverters`的静态类，其中包含用于在千克和磅之间转换的静态方法：

```cs
static class MassConverters
{
    public static double PoundToKg(double pounds)
    {
        return pounds * 0.45359237;
    }
    public static double KgToPound(double kgs)
    {
        return kgs * 2.20462262185;
    }
}
var lbs = MassConverters.KgToPound(42.5);
var kgs = MassConverters.PoundToKg(180);
```

因为静态类不能被实例化，所以`this`关键字在这样的类的上下文中没有意义。尝试使用它将导致编译器错误。

### 静态构造函数

一个类可以有静态构造函数，无论类本身是否是静态的。静态构造函数没有参数或访问修饰符，用户无法调用它。CLR 在以下情况下自动调用静态构造函数：

+   在静态类中，当第一次访问类的第一个静态成员时

+   在非静态类中，当类首次实例化时

静态构造函数对于初始化静态字段非常有用。例如，`static readonly`字段只能在声明期间或在静态构造函数中初始化。当值来自配置文件时，用于将条目写入日志文件，或者用于编写非托管代码的包装器时，静态构造函数可以调用`LoadLibrary()`API，这是非常有用的。

在下面的示例中，修改了`Employee`类的先前实现，提供了一个静态构造函数来初始化静态`id`字段的值。这个构造函数从应用程序文件中读取下一个员工的 ID，如果找不到文件，则将其初始化为`1`。每次创建类的新实例时，下一个员工 ID 的值都将写入此文件：

```cs
class Employee
{
    private static int id;
    public int EmployeeId { get; private set; }
    public string FirstName { get; private set; }
    public string LastName { get; private set; }
    static Employee()
    {
        string text = "1";
        try
        {
            text = File.ReadAllText("app.data");
        }
        catch { }
        int.TryParse(text, out id);
    }
    private Employee(int id, string firstName, string lastName)
    {
        EmployeeId = id;
        FirstName = firstName;
        LastName = lastName;
    }
    public static Employee Create(string firstName, 
                                  string lastName)
    {
        var employee = new Employee(id++, firstName, lastName);
        File.WriteAllText("app.data", id.ToString());
        return employee;
    }
}
```

如果您多次运行以下代码，第一次两个员工的 ID 将是`1`和`2`，然后是`3`和`4`，依此类推：

```cs
Employee obj1 = Employee.Create("John", "Doe");
Employee obj2 = Employee.Create("Jane", "Doe");
Console.WriteLine($"{obj1.EmployeeId} {obj1.FirstName}");
Console.WriteLine($"{obj2.EmployeeId} {obj2.FirstName}");
```

到目前为止，我们已经看到了如何创建方法和构造函数。在下一节中，我们将学习有关将参数传递给它们的不同方法。

## 引用、输入和输出参数

当我们将参数传递给方法时，它是按值传递的。这意味着会创建一个副本。如果类型是值类型，则参数的值将被复制到方法参数中。如果类型是引用类型，则引用将被复制到方法参数中。当您更改参数值时，它会更改本地副本。这意味着值类型的参数更改不会传播到调用者。至于引用类型的参数，您可以更改堆上的引用对象，但不能更改引用本身。使用`ref`、`in`和`out`关键字可以改变这种行为。

### ref 关键字

`ref`关键字允许我们创建*按引用调用*机制，而不是按值调用机制。在声明和调用方法时指定了`ref`关键字。使用`ref`关键字改变参数，使其成为参数的别名，必须是一个变量。这意味着您不能将属性或索引器（实际上是一个方法）作为`ref`参数的参数传递。`ref`参数必须在方法调用之前初始化。

让我们看一下以下代码示例：

```cs
class Program
{
    static void Swap(ref int a, ref int b)
    {
        int temp = a;
        a = b;
        b = temp;
    }
    static void Main(string[] args)
    {
        int num1 = 10;
        int num2 = 20;
        Console.WriteLine($"Before swapping: num1={num1}, num2={num2}");
        Swap(ref num1, ref num2);
        Console.WriteLine($"After swapping:  num1={num1}, num2={num2}");
    }
}
```

在此程序中，我们定义了一个`Swap`方法来交换两个整数值。我们使用`ref`关键字来声明方法参数。我们将此方法定义为`static`，以便我们可以在没有对象引用的情况下调用它。在`Main`方法中，我们初始化了两个整数变量。

在调用`Swap`方法时，我们还使用了`ref`关键字和参数名称。这些`ref`参数作为引用传递，并且`num1`和`num2`变量的实际值将被交换。这种更改会反映在`Main`方法中的变量中。该程序的输出如下截图所示：

![图 4.4 - 控制台显示交换前后 num1 和 num2 的值](img/Figure_4.4._B12346.jpg)

图 4.4 - 控制台显示交换前后 num1 和 num2 的值

`ref`关键字可用于指定引用返回值。在这种情况下，它必须出现在以下位置：

+   在方法签名中，在返回类型之前。

+   在返回语句中，在`return`关键字和返回的值之间。这样的值称为*ref 返回值*。

+   在将返回的引用分配给本地变量的声明中，在变量的类型之前。这样的变量称为*ref 本地变量*。

+   在调用带有`ref`返回的方法之前。

在以下示例中，`Project`类具有`Employee`类型的成员字段。在构造函数中设置了对`Employee`实例的引用。`GetOwner()`方法返回对成员字段的引用：

```cs
class Project
{
    Employee owner;
    public string Name { get; private set; }
    public Project(string name, Employee owner)
    {
        Name = name;
        this.owner = owner;
    }
    public ref Employee GetOwner()
    {
        return ref owner;
    }
    public override string ToString() => 
      $"{Name} (Owner={owner.FirstName} {owner.LastName})";
}
```

可以按以下方式用于检索和更改项目的所有者。在以下代码中，请注意在本地变量声明和调用`GetOwner()`方法中使用`ref`关键字：

```cs
Employee e1 = new Employee(1, "John", "Doe");
Project proj = new Project("Demo", e1);
Console.WriteLine(proj);
ref Employee owner = ref proj.GetOwner();
owner = new Employee(2, "Jane", "Doe");
Console.WriteLine(proj);
```

该程序的输出如下截图所示：

![图 4.5 - 上一段代码的输出截图](img/Figure_4.5._B12346.jpg)

图 4.5 - 上一段代码的输出截图

在使用`ref`返回值时，必须注意以下事项：

+   不可能返回对局部变量的引用。

+   不可能返回对`this`的引用。

+   可以返回对类字段的引用，也可以返回对没有`set`访问器的属性的引用。

+   可以返回对`ref`/`in`/`out`参数的引用。

+   通过引用返回会破坏封装，因为调用者可以完全访问对象的状态或部分状态。

让我们现在来看一下`in`关键字。

### in 关键字

`in`关键字与`ref`关键字非常相似。它导致参数通过引用传递。然而，关键区别在于`in`参数不能被调用方法修改。`in`参数基本上是一个`readonly ref`参数。如果被调用的方法尝试修改值，编译器将发出错误。作为`in`参数传递的变量在传递给调用方法的参数之前必须初始化。

以下示例显示了一个接受两个`in`参数的方法。任何试图更改它们的值都会导致编译器错误：

```cs
static void DontTouch(in int value, in string text)
{
    value = 42;   // error
    ++value;      // error
    text = null;  // error
}
int i = 0;
string s = "hello";
DontTouch(i, s);
```

在传递参数给方法时，指定`in`关键字是可选的。在上面的例子中，这是被省略的。

`in`说明符主要用于在热路径上传递值类型对象的引用，即重复调用的函数。当将值类型对象传递给函数时，在堆栈上会创建一个值的副本。通常，这不会引起任何性能问题，但是当这种情况一再发生时，性能问题就会出现。通过使用`in`说明符，可以传递对象的只读引用，从而避免复制。

`in`说明符的另一个好处是清晰地传达参数不应该被方法修改的设计意图。

### out 关键字

`out`关键字类似于`ref`关键字。不同之处在于，作为`out`参数传递的变量在调用方法之前不必初始化，但是在返回之前，接受`out`参数的方法必须为其分配一个值。在方法定义和方法调用之前，`out`关键字必须都存在。

返回输出值在方法需要返回多个值或者需要返回一个值但也需要返回执行是否成功的信息时非常有用。一个例子是`int.TryParse()`，它返回一个布尔值，指示解析是否成功，并将实际解析的值作为`out`参数返回。

为了看看它是如何工作的，让我们看下面的例子：

```cs
static void Square(int input, out int output)
{
    output = input * input;
}
```

我们已经定义了一个`static`方法来返回一个整数的平方。`Square`方法将接受两个参数。`int`参数将是一个整数值，并且它将通过`out`参数输出返回输入数字的平方。可以如下使用：

```cs
int num = 10;
int SquareNum;
Square(num, out SquareNum);
```

执行后，此程序的输出将是`100`。

作为`out`参数使用的变量可以在方法调用中内联声明。这会产生更简单、更紧凑的代码。内联变量的作用域是调用方法的作用域。

上述代码可以简化如下：

```cs
int num = 10;
Square(num, out int SquareNum);
```

在使用这些参数说明符时有一些限制，将在下一节中进行解释。

了解它们的限制

在使用`ref`、`in`和`out`参数时，必须注意几个限制。这些关键字不能与以下内容一起使用：

+   使用`async`修饰符定义的异步方法。

+   迭代器方法，其中包括`yield return`或`yield break`。

另一方面，在重载解析的上下文中，`ref`、`in`和`out`关键字不被视为方法签名的一部分。这意味着你不能有两个相同方法的重载：一个接受`ref`参数，另一个接受相同参数作为`out`参数。但是，如果一个方法有一个值参数，另一个方法有一个`ref`、`in`或`out`参数，那么可以有重载的方法：

```cs
class foo
{
  public void DoSomething(ref int i);
  public void DoSomething(out int i); // error: cannot overload
}
class bar
{
  public void DoSomethingElse(int i);
  public void DoSomethingElse(ref int i);  // OK
}
```

到目前为止，我们在本书中看到的所有方法都有固定数量的参数。然而，语言允许我们定义可以接受可变数量参数的方法。下面将讨论这个主题。

## 具有可变数量参数的方法

到目前为止，我们只看到了接受零个或固定数量参数的方法。然而，也可以定义接受相同类型任意数量参数的方法。为此，必须有一个由`params`关键字引导的单维数组参数。这个参数不一定要是方法的唯一参数，但在它之后不允许有更多的参数。

在以下示例中，我们有两个方法—`Any()`和`All()`—它们接受可变数量的布尔值，并返回一个布尔值，指示它们中是否有任何一个为`true`，以及分别是否它们全部为`true`：

```cs
static bool Any(params bool [] values)
{
    foreach (bool v in values)
        if (v) return true;
    return false;
}
static bool All(params bool[] values)
{
    if (values.Length == 0) return false;
    foreach (bool v in values)
        if (!v) return false;
    return true;
}
```

这两种方法都可以用零个、一个或任意数量的参数来调用，如下所示：

```cs
var a = Any(42 > 15, "text".Length == 3);  // a=true
var b = All(true, false, true);            // b=false
var c = All();                             // c=false
```

方法调用时提供参数的方式是灵活的。接下来我们将看看现有的可能性。

## 命名和可选参数

到目前为止，我们所见过的所有例子中，方法调用的参数都是按照方法签名中参数声明的顺序提供的。这些被称为*位置参数*，因为它们是基于给定位置进行评估的。此外，所有参数都是必需的，这意味着除非为参数列表中的每个参数提供了参数，否则不能发生调用。

然而，C#支持另外两种类型的参数：*可选参数*和*命名参数*。这些经常一起使用，使我们能够为可选参数列表中的参数提供部分参数。这些可以用于方法、索引器、构造函数和委托。

### 可选参数

在声明方法、构造函数、索引器或委托时，我们可以为参数指定默认值。当存在这样的参数时，在成员调用时提供参数是可选的。如果没有提供参数，编译器将使用默认值。参数的默认值必须是以下之一：

+   常量表达式

+   `new T()`形式的表达式，其中`T`是值类型

+   `default(T)`形式的表达式，其中`T`也是值类型

方法可以有必需和可选参数。如果存在可选参数，则它们必须跟在所有非可选参数后面。非可选参数不能跟在可选参数后面。

让我们考虑`Point`结构的以下实现：

```cs
struct Point
{
    public int X { get; }
    public int Y { get; }
    public Point(int x = 0, int y = 0)
    {
        X = x;
        Y = y;
    }
}
```

构造函数接受两个参数，它们都具有默认值`0`。这意味着它们都是可选的。我们可以以以下任何形式调用构造函数：

```cs
Point p1 = new Point();     // x = 0, y = 0
Point p2 = new Point(1);    // x = 1, y = 0
Point p3 = new Point(1, 2); // x = 1, y = 2
```

在第一个例子中，没有提供`Point`构造函数的参数，因此编译器将使用`0`作为`x`和`y`的值。在第二个例子中，提供了一个参数，它将用于绑定到第一个构造函数参数。因此，`x`将是`1`，`y`将是`0`。在第三个和最后一个例子中，提供了两个参数，它们按照这个顺序绑定到`x`和`y`。因此，`x`是 1，`y`是`2`。

### 命名参数

命名参数使我们能够通过它们的名称而不是在参数列表中的位置来调用方法。参数可以以任何顺序指定，并且与默认参数结合使用，我们可以为方法调用指定部分参数。通过指定参数名称后跟冒号（`:`）和值来提供命名参数。

让我们考虑以下例子：

```cs
Point p1 = new Point(x: 1, y: 2); // x = 1, y = 2
Point p2 = new Point(1, y: 2);    // x = 1, y = 2
Point p3 = new Point(x: 1, 2);    // x = 1, y = 2
Point p4 = new Point(y: 2);       // x = 0, y = 2
Point p5 = new Point(x: 1);       // x = 1, y = 0
```

前三个构造函数调用是等效的；`p1`、`p2`和`p3`代表同一个点。构造函数的调用使用了一个或多个命名参数，但效果是相同的。另一方面，构造`p4`时，只指定了`y`的值。因此，`x`将是`0`，`y`将是`2`。最后，通过仅指定`x`的命名参数来创建`p5`。因此，`x`将是`1`，`y`将是`0`。

## 访问修饰符

访问修饰符用于定义 C#中类型或成员的可见性。它指定程序集中的其他代码部分或其他程序集可以访问类型或类型成员的内容。C#定义了六种类型的访问修饰符，如下所示：

+   `public`：公共字段可以被同一程序集中的任何代码部分或另一个程序集中的代码访问。

+   `protected`：受保护类型或成员只能在当前类和派生类中访问。

+   `internal`：内部类型或成员只能在当前程序集中访问。

+   `protected internal`：这是`protected`和`internal`访问级别的组合。受保护的内部类型或成员可以在当前程序集中或派生类中访问。

+   `private`：私有类型或成员只能在类或结构内部访问。这是 C#中定义的最不可访问级别。

+   `private protected`：这是`private`和`protected`访问级别的组合。私有受保护类型或类型成员可以被同一类中的代码或派生类中的代码访问，但只能在同一程序集中。

尝试访问超出其访问级别的类型或类型成员将导致编译时错误。

适用于类型和类型成员的可访问性有不同种类的规则：

+   `public`或`internal`（默认）。另一方面，派生类不能比其基类型具有更大的可访问性。这意味着如果有一个`internal`类`B`，则不能从中派生一个`public`类`D`。

+   `public`，`internal`或`private`。这些规则适用于嵌套的结构和类。类和结构成员的默认访问级别是`private`。私有的嵌套类型只能从包含类型中访问。成员的可访问性不能大于包含它的类型。

此外，字段、属性或事件的类型必须至少与字段本身一样可访问。类似地，方法、索引器或委托的返回类型以及其参数的类型不能比成员本身更不可访问。

+   `public`和`static`。终结器不能有访问修饰符。直接在命名空间中定义的接口可以是`public`或`internal`（默认）。访问修饰符不能应用于任何接口成员，它们隐式为`public`。类似地，枚举成员隐式为`public`，并且不能有访问修饰符。委托类似于类和结构-当直接在命名空间中定义时，默认访问级别为`internal`，在另一个类型中嵌套时为`private`。

以下代码显示了类型和类型成员的访问修饰符的各种用法：

```cs
public interface IEngine
{
   double Power { get; }
   void Start();
}
public class DieselEngine : IEngine
{
   private double _power;
   public double Power { get { return _power; } }
   public void Start() { }
}
```

在本章中，我们学习了如何定义自定义类。到目前为止的所有示例中，整个类都是在一个地方定义的。然而，可以将一个类分割成几个不同的定义，可以在同一个文件或不同的文件中进行，这是我们将在下一节中讨论的内容。

## 部分类

部分类允许我们将类分成多个类定义，当一个类变得非常庞大或者我们想要逻辑上将一个类分成多个部分时，这是非常有用的。这使得诸如 WPF 之类的技术能够更好地工作，因为用户代码和 IDE 设计者编写的代码被分隔到不同的源文件中。

每个部分可以使用`partial`关键字在不同的源文件中定义。此关键字必须立即出现在`class`关键字之前。这些部分必须在编译时可用。在编译过程中，这些部分被合并成一个单一类型。

`partial`关键字不仅可以应用于类，还可以应用于结构、接口和方法。所有这些都适用相同的规则。

这里展示了`partial`类的一个示例：

```cs
partial class Employee
{
    partial void Promote();
}
partial class Employee
{
    public int EmployeeId { get; set; }
}
partial class Employee
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    partial void Promote()
    {
        Console.WriteLine("Employee promoted!");
    }
}
```

在这里，我们将类定义分成了两个`partial`类。两个`partial`类都包含一些属性。我们可以实例化`partial`类并像普通类一样使用它的属性。参考以下代码片段：

```cs
Employee obj = new Employee()
{
    EmployeeId = 1,
    FirstName = "John",
    LastName = "Doe"
};
obj.Promote();
```

以下列表包含了部分类型的属性，以及它们的规则：

+   所有部分必须具有相同的可访问性。

+   不同的部分可以指定不同的基接口。最终类型将实现所有列出的接口。

+   如果多个部分指定了一个基类，那么它必须是相同的基类，因为 C#不支持多重继承。基类只能在一个部分上指定。其他部分是可选的。

+   所有部分的属性在编译时合并在一起。最终类型将具有所有部分声明中使用的属性。

+   嵌套类也可以是 partial 的。

方法也可以是 partial 的。这使我们能够在`partial`类或结构的一个部分中提供签名，而在另一个部分中提供实现。这在 IDE 中很有用，可以提供开发人员可能或可能不实现的方法挂钩。如果一个 partial 方法没有实现，它将在编译时从类定义中移除。partial 方法不能有访问修饰符，它们是隐式私有的。此外，partial 方法不能返回值；partial 方法的返回类型必须是`void`。

# 结构

到目前为止，本章的内容都集中在类上。作为类定义的类型是引用类型。然而，在.NET 和 C#中，还有另一类类型：**值类型**。值类型具有值语义，这意味着在赋值时复制的是对象的值，而不是对象的引用。

值类型使用`struct`关键字来定义，而不是`class`。在大多数方面，结构与类是相同的，本章介绍的特性也适用于结构。然而，它们也有一些关键的区别：

+   结构不支持继承。虽然一个结构可以实现任意数量的接口，但它不能从另一个结构派生。因此，结构成员不能有`protected`访问修饰符。此外，结构的方法或属性不能是`abstract`或`virtual`。

+   结构不能声明默认（无参数）构造函数。

+   结构可以在不使用`new`运算符的情况下实例化。

+   在结构声明中，除非声明为`const`或`static`，否则字段不能被初始化。

让我们考虑以下示例，在这个示例中，我们定义了一个名为`Point`的结构，它有两个整数字段：

```cs
struct Point
{
    public int x;
    public int y;
}
```

我们可以使用`new`运算符来实例化它，这将调用默认构造函数，将所有成员字段初始化为它们的默认值，或者直接在没有`new`运算符的情况下实例化它。在这种情况下，成员字段将保持未初始化状态。这可能对性能有用，但在所有字段正确初始化之前，这样的对象不能被使用：

```cs
Point p = new Point()
{
    x = 2,
    y = 3
};
```

前面的代码使用`new`运算符来创建类型的实例。另一方面，在下面的例子中，对象是在没有`new`运算符的情况下创建的：

```cs
Point p;
p.x = 2;
p.y = 3;
```

虽然结构和类有许多共同之处，但它们在一些关键方面也有所不同。重要的是要理解何时应该使用类，何时应该使用结构。在以下情况下应该使用结构：

+   当它表示单个值（例如一个点，一个 GUID 等）

+   当它很小（通常不超过 16 个字节）

+   当它是不可变的时候

+   当它是短暂的时候

+   当它在装箱和拆箱操作中不经常使用（这会影响性能）

在所有其他情况下，类型应该被定义为类。

值类型的变量不能被赋予`null`值。然而，对于值类型来说，当没有值是有效的值时，可以使用可空值类型（使用简写语法表示为`T?`）。可空类型在*第二章* *数据类型和运算符*中已经讨论过。

以下是一个将可空的`Point`变量赋值为`null`的示例：

```cs
Point? p = null;
```

文献中经常提到值类型的实例存储在堆栈上。这种说法只是部分正确的。堆栈是一个实现细节；它不是值类型的特征之一。值类型的局部变量或临时变量确实存储在堆栈上（除非它们没有封闭在 lambda 或匿名方法的外部变量中），并且不是迭代器块的一部分。

否则，它们通常存储在堆上。然而，这完全是一个实现和编译器的细节，事实上，值类型可以存储在许多地方：在堆栈中，在 CPU 寄存器中，在 FPU 帧上，在垃圾收集器管理的堆上，在 AppDomain 的加载器堆中，或者在线程本地存储中（如果变量具有`ThreadStorage`属性）。

当将值类型对象（存储位置直接包含值）赋给引用类型对象（存储位置包含实际值的引用）时，会发生装箱的过程。反之，这个过程称为拆箱。我们在本书中之前已经讨论过这两个过程，在*第二章* *数据类型和运算符*中。

看一下下面的例子：

```cs
struct Point
{
   public int X { get; }
   public int Y { get; }
   public Point(int x = 0, int y = 0)
   {
      X = x;
      Y = y;
   }
}
Point p1 = new Point(2, 3);
Point p2 = new Point(0, 3);
if (p1.Equals(p2)) { /* do something */ }
```

在这里，我们有两个`Point`值类型的变量，我们想要检查它们是否相等。为此，我们调用了在`System.Object`基类中定义的`Equals()`方法。当我们这样做时，会发生装箱，因为`Equals`的参数是一个对象，即一个引用类型。如果装箱频繁进行，可能会成为性能问题。有两种方法可以避免对值类型进行装箱。

第一种解决方案是实现包含单个`Equals(T)`方法的`IEquatable<T>`接口。这个方法允许值类型和引用类型都实现一种确定两个对象是否相等的方式。这个接口被泛型集合用于在各种方法中测试相等性。因此，出于性能原因，所有可能存储在泛型集合中的类型都应该实现这个接口。

实现了实现`IEquatable<T>`的`Point`结构如下：

```cs
struct Point : IEquatable<Point>
{
    public int X { get; }
    public int Y { get; }
    public Point(int x = 0, int y = 0)
    {
        X = x;
        Y = y;
    }
    public bool Equals(Point other)
    {
        return X == other.X && Y == other.Y;
    }
    public override bool Equals(object obj)
    {
        if (obj is Point other)
        {
            return this.Equals(other);
        }
        return false;
    }
    public override int GetHashCode()
    {
        return X.GetHashCode() * 17 + Y.GetHashCode();
    }
}
```

在这个例子中，你应该注意到`IEquatable`的泛型类型参数是类型本身，即`Point`。这是一种称为*奇异递归模板模式*的技术。该类实现了`Equals(Point)`，检查类型的属性。然而，它还重写了`System.Object`虚拟方法，`Equals()`和`GetHashCode()`，确保这两个实现是一致的。

在实现`IEquatable<T>`接口时，应牢记以下几点：

+   `Equals(T)`和`Equals(object)`必须返回一致的结果。

+   如果值是可比较的，那么它也应该实现`IComparable<T>`。

+   如果类型实现了`IComparable<T>`，那么它也应该实现`IEquatable<T>`。

第二种解决方案是重载`==`和`!=`运算符。可以这样做：

```cs
struct Point
{
    public int X { get; }
    public int Y { get; }
    public Point(int x = 0, int y = 0)
    {
        X = x;
        Y = y;
    }
    public override bool Equals(object obj)
    {
        if (obj is Point other)
        {
            return this.Equals(other);
        }
        return false;
    }
    public override int GetHashCode()
    {
        return X.GetHashCode() * 17 + Y.GetHashCode();
    }
    public static bool operator !=(Point p1, Point p2)
    {
        return p1.X != p2.X || p1.Y != p2.Y;
    }
    public static bool operator ==(Point p1, Point p2)
    {
        return p1.X == p2.X && p1.Y == p2.Y;
    }
}
```

在这种情况下，我们将不再使用`Equals()`来比较值，而是使用两个运算符`==`和`!=`：

```cs
Point p1 = new Point(2, 3);
Point p2 = new Point(0, 3);
if (p1 == p2) { /* do something */ }
```

然而，如果你想要能够双向检查相等性，也可以同时实现`IEquatable<T>`和重载比较运算符。我们将在*第五章* *C#面向对象编程*中更详细地讨论运算符重载。

# 枚举

枚举是一组命名的整数常量。我们使用`enum`关键字声明枚举。枚举是值类型。当我们想要为某个特定目的使用有限数量的整数值时，枚举非常有用。定义和使用枚举有几个优点：

+   我们使用命名常量而不是文字值。这样做使代码更易读和更易于维护。

+   当使用诸如 Visual Studio 之类的 IDE 时，可以看到可以分配给变量的可能值列表。

+   它强制使用数字常量进行类型安全。

下面的例子显示了一个名为`Priority`的枚举，有四个可能的值：

```cs
enum Priority
{
    Low,
    Normal,
    Important,
    Urgent
}
```

枚举的每个元素代表一个整数值。默认情况下，第一个标识符被分配为零（`0`）。每个后续标识符的值将递增一。还可以为每个元素指定显式值。以下规则适用：

+   这些值必须在基础类型的范围内。

+   这些值不必是连续的或有序的。

+   可以定义具有相同数值的多个标识符。

如定义的枚举，语义上等同于以下情况，其中值是显式指定的：

```cs
enum Priority
{
    Low = 0,
    Normal = 1,
    Important = 2,
    Urgent = 3
}
```

如前所述，枚举的每个元素都可以具有任何数值。下面的例子显示了`Priority`枚举的定义。其中一些元素具有显式值，其他元素是基于它们计算的：

```cs
enum Priority
{
    Low = 10,
    Normal,
    Important = 20,
    Urgent
}
```

在这个实现中，`Low`是 10，`Normal`是 11，`Important`是 20，`Urgent`是 21。

枚举的默认基础类型是`int`，但可以指定任何整数类型作为基础类型。`char`类型不能作为枚举的基础类型。在下面的例子中，`byte`是`Priority`的基础类型：

```cs
enum Priority : byte
{
    Low = 10,
    Normal,
    Important = 20,
    Urgent
}
```

要使用枚举的元素，需要指定枚举名称，后跟一个点（`.`）和元素名称，例如`Priority.Normal`：

```cs
Priority p = Priority.Normal;
Console.WriteLine(Priority.Urgent);
```

可以将基础类型的任何值分配给枚举变量，即使不存在具有相应数值的元素。这只能通过强制转换来实现。但是，文字`0`可以隐式转换为任何枚举类型，无需强制转换：

```cs
Priority p1 = (Priority)42;   // p1 is 42
Priority p2 = 0;              // p2 is 0
Priority p3 = (int)10;        // p3 is Low
```

另一方面，枚举和整数类型之间没有隐式转换。要获得枚举标识符的整数值，必须使用显式转换，如下所示：

```cs
int i = (int)Priority.Normal;
```

因为枚举的所有元素的引用在编译时都被替换为它们的文字值，所以改变枚举元素的值会影响引用的程序集。当枚举类型在其他程序集中使用时，数值将存储在这些程序集中。除非重新编译，否则对枚举的更改不会反映在依赖的程序集中。

如果需要从字符串解析枚举值，可以使用通用的`Enum.TryParse()`方法，如下例所示：

```cs
Enum.TryParse("Normal", out Priority p); // p is Normal
```

然而，如果要从字符串中解析并忽略大小写，则需要使用相同方法的非泛型重载，如下所示：

```cs
Enum.TryParse(typeof(Priority), "normal", true, out object o);
Priority p = (Priority)o;   // p is Normal
```

在这个例子中，字符串`"normal"`被解析，忽略大小写以识别`Priority`枚举的可能值。输出参数中返回的值是`Priority.Normal`。

# 命名空间

在本书中我们已经多次提到了命名空间，但没有解释它们到底是什么。命名空间用于将代码组织成逻辑单元。命名空间定义了一个包含类型的声明空间。这个声明空间有一个名称，是类型名称的一部分。例如，.NET 类型`String`在`System`命名空间中声明。类型的完整名称是`System.String`。这被称为类型的完全限定名称。通常，我们只使用类型的未限定名称（在这种情况下是`String`），因为我们使用`using`指令从特定命名空间将声明引入当前范围。

命名空间主要用于两个目的：

+   为了帮助组织代码。通常，属于一起的类型在同一个命名空间中声明。

+   为了避免类型可能的名称冲突。程序可能依赖于不同的库，很可能在这些库中存在同名的类型。通过使用具有高度唯一性的命名空间，可以大大减少名称冲突的机会。

+   命名空间是用`namespace`关键字引入的。它们是隐式公共的，当声明它们时不能使用访问修饰符。命名空间可以包含任意数量的类型（类、结构、枚举或委托）。

以下示例显示了如何定义一个名为`chapter_04`的命名空间：

```cs
namespace chapter_04
{
   class foo { }
}
```

命名空间可以嵌套；一个命名空间可以包含其他命名空间。下面的代码片段中展示了一个例子，其中`chapter_04`命名空间包含一个名为`demo`的嵌套命名空间：

```cs
namespace chapter_04
{
   namespace demo
   {
      class foo { }
   }
}
```

在这个例子中，`foo`类型的完全限定名称是`chapter_04.demo.foo`。

为了简洁起见，嵌套命名空间可以用简写语法声明：只需要一个命名空间声明，而不是多个。命名空间的名称是所有命名空间名称的连接，用点分隔。前面的声明等同于以下内容：

```cs
namespace chapter_04.demo
{
   class foo { }
}
```

要使用`foo`类型的实例，您必须使用它的完全限定名称，如下所示：

```cs
namespace chapter_04
{
    class Program
    {
        static void Main(string[] args)
        {
            var f = new chapter_04.demo.foo();
        }
    }
}
```

为了避免这种情况，您可以使用`using`指令，指定命名空间名称如下：

```cs
using chapter_04.demo;
namespace chapter_04
{
    class Program
    {
        static void Main(string[] args)
        {
            var f = new foo();
        }
    }
}
```

`using`指令只能存在于命名空间级别（而不是局部于方法或类型）。通常，您将它们放在源文件的开头，在这种情况下，它的类型在整个源代码中都可用。或者，您可以将它们指定在特定的命名空间中，在这种情况下，它的类型只对该命名空间可用。

命名空间被称为是开放式的。这意味着您可以有多个具有相同名称的命名空间声明，无论是在同一个源文件中还是在不同的源文件中。在这种情况下，所有这些声明都代表同一个命名空间，并且贡献到同一个声明空间。下面的代码片段演示了这种情况的一个例子：

```cs
namespace chapter_04.demo
{
   class foo { }
}
namespace chapter_04.demo
{
   class bar { }
}
```

有一个隐式命名空间，它是所有命名空间的根（包含所有未在命名空间中声明的命名空间和类型）。这个命名空间叫做`global`。如果您需要包含它以指定完全限定名称，那么您必须用`::`而不是点来分隔它，就像`global::System.String`一样。这在命名空间名称冲突的情况下可能是必要的。这里有一个例子：

```cs
namespace chapter_04.System
{
    class Console { }

    class Program
    {
        static void Main(string[] args)
        {
            global::System.Console.WriteLine("Hello, world!");
        }
    }
}
```

在这个例子中，如果没有`global::`别名，用户定义的`chapter_04.System.Console`类型将在`Main()`函数中使用，而不是预期的`System.Console`类型。

# 总结

在本章中，我们已经学习了 C#中的用户定义类型。我们学习了类和结构，这些帮助我们在 C#中创建自定义用户类型。我们还学习了如何在类中创建和使用字段、属性、方法、索引器和构造函数，以及我们学习了`this`和`static`关键字。

我们探讨了访问修饰符的概念，并了解了如何为类型和成员定义不同级别的访问。我们还学习了`ref`、`in`和`out`参数修饰符，以及具有可变数量参数的方法。最后但同样重要的是，我们学习了命名空间以及如何使用它们来组织代码并避免名称冲突。

在下一章中，我们将学习**面向对象编程**（**OOP**）的概念。我们将探讨 OOP 的构建模块——封装、继承、多态和抽象。我们还将学习抽象类和接口。

# 测试你所学到的知识

1.  什么是类，什么是对象？

1.  类和结构之间有什么区别？

1.  什么是只读字段？

1.  什么是表达式主体定义？

1.  默认构造函数是什么，静态构造函数又是什么？

1.  什么是自动实现属性？

1.  索引器是什么，它们如何定义？

1.  静态类是什么，它可以包含什么？

1.  参数修饰符是什么，它们有什么不同？

1.  什么是枚举，它们在什么时候有用？
