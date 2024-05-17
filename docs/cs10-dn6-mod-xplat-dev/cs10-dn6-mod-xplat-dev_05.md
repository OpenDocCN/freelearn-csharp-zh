# 第五章：使用面向对象编程构建自己的类型

本章是关于使用**面向对象编程**（**OOP**）创建自己的类型。你将了解类型可以拥有的所有不同类别的成员，包括存储数据的字段和执行操作的方法。你将使用 OOP 概念，如聚合和封装。你还将了解语言特性，如元组语法支持、输出变量、推断元组名称和默认文字。

本章将涵盖以下主题：

+   谈论 OOP

+   构建类库

+   使用字段存储数据

+   编写和调用方法

+   使用属性和索引器控制访问

+   使用对象进行模式匹配

+   使用记录

# 谈论 OOP

现实世界中的对象是某个事物，比如汽车或人，而在编程中，对象通常代表现实世界中的某个事物，比如产品或银行账户，但也可能是更抽象的东西。

在 C#中，我们使用`class`（大多数情况下）或`struct`（有时）C#关键字来定义对象类型。你将在*第六章*，*实现接口和继承类*中了解类和结构之间的区别。你可以将类型视为对象的蓝图或模板。

OOP 的概念简要描述如下：

+   **封装**是与对象相关的数据和操作的组合。例如，`BankAccount`类型可能具有数据，如`Balance`和`AccountName`，以及操作，如`Deposit`和`Withdraw`。在封装时，你通常希望控制可以访问这些操作和数据的内容，例如，限制从外部访问或修改对象的内部状态。

+   **组合**是关于对象由什么构成的。例如，`Car`由不同的部分组成，例如四个`Wheel`对象，几个`Seat`对象和一个`Engine`。

+   **聚合**是关于可以与对象结合的内容。例如，`Person`不是`Car`对象的一部分，但他们可以坐在驾驶员的`Seat`上，然后成为汽车的`Driver`——两个独立的对象聚合在一起形成一个新的组件。

+   **继承**是通过让**子类**从**基类**或**超类**派生来重用代码。基类中的所有功能都被继承并可在**派生**类中使用。例如，基类或超类`Exception`具有一些成员，这些成员在所有异常中具有相同的实现，而子类或派生类`SqlException`继承了这些成员，并且具有仅与 SQL 数据库异常发生时相关的额外成员，例如数据库连接的属性。

+   **抽象**是捕捉对象核心思想并忽略细节或具体内容的概念。C#有`abstract`关键字正式化这一概念。如果一个类没有明确地**抽象**，那么它可以被描述为**具体**的。基类或超类通常是抽象的，例如，超类`Stream`是抽象的，而它的子类，如`FileStream`和`MemoryStream`，是具体的。只有具体类可以用来创建对象；抽象类只能用作其他类的基类，因为它们缺少一些实现。抽象是一个棘手的平衡。如果你使一个类更抽象，更多的类将能够继承自它，但同时，可共享的功能将更少。

+   **多态性**是指允许派生类覆盖继承的操作以提供自定义行为。

# 构建类库

类库程序集将类型组合成易于部署的单元（DLL 文件）。除了学习单元测试时，你只创建了控制台应用程序或.NET Interactive 笔记本以包含你的代码。为了使你编写的代码可跨多个项目重用，你应该将其放入类库程序集中，就像 Microsoft 所做的那样。

## 创建类库

第一个任务是创建一个可重用的.NET 类库：

1.  使用你喜欢的编码工具创建一个新的类库，如下列表所定义：

    1.  项目模板：**类库** / `classlib`

    1.  工作区/解决方案文件和文件夹：`Chapter05`

    1.  项目文件和文件夹：`PacktLibrary`

1.  打开`PacktLibrary.csproj`文件，并注意默认情况下类库面向.NET 6，因此只能与其他.NET 6 兼容的程序集一起工作，如下面的标记所示：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <TargetFramework>net6.0</TargetFramework>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
      </PropertyGroup>
    </Project> 
    ```

1.  将框架修改为目标.NET Standard 2.0，并删除启用可空引用类型和隐式 using 的条目，如下面的标记中突出显示的那样：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
     **<TargetFramework>netstandard****2.0****</TargetFramework>**
      </PropertyGroup>
    </Project> 
    ```

1.  保存并关闭文件。

1.  删除名为`Class1.cs`的文件。

1.  编译项目以便其他项目稍后可以引用它：

    1.  在 Visual Studio Code 中，输入以下命令：`dotnet build`。

    1.  在 Visual Studio 中，导航到**生成** | **生成 PacktLibrary**。

**最佳实践**：为了使用最新的 C#语言和.NET 平台特性，将类型放入.NET 6 类库中。为了支持如.NET Core、.NET Framework 和 Xamarin 等遗留.NET 平台，将可能重用的类型放入.NET Standard 2.0 类库中。

## 在命名空间中定义一个类

接下来的任务是定义一个代表人的类：

1.  添加一个名为`Person.cs`的新类文件。

1.  静态导入`System.Console`。

1.  将命名空间设置为`Packt.Shared`。

**良好实践**：我们这样做是因为将你的类放在逻辑命名的命名空间中很重要。更好的命名空间名称应该是特定领域的，例如，`System.Numerics`用于与高级数字相关的类型。在这种情况下，我们将创建的类型是`Person`，`BankAccount`和`WondersOfTheWorld`，它们没有典型的领域，因此我们将使用更通用的`Packt.Shared`。

你的类文件现在应该看起来像以下代码：

```cs
using System;
using static System.Console;
namespace Packt.Shared
{
  public class Person
  {
  }
} 
```

注意，C#关键字`public`在类之前应用。这个关键字是**访问修饰符**，它允许任何其他代码访问这个类。

如果你没有明确应用`public`关键字，那么它将只能在定义它的程序集中访问。这是因为类的默认访问修饰符是`internal`。我们需要这个类在程序集外部可访问，因此必须确保它是`public`。

### 简化命名空间声明

如果你针对的是.NET 6.0，因此使用 C# 10 或更高版本，你可以用分号结束命名空间声明并删除大括号，如下所示：

```cs
using System; 
namespace Packt.Shared; // the class in this file is in this namespace
public class Person
{
} 
```

这被称为文件范围的命名空间声明。每个文件只能有一个文件范围的命名空间。我们将在本章后面针对.NET 6.0 的类库中使用这个。

**良好实践**：将你创建的每个类型放在其自己的文件中，以便你可以使用文件范围的命名空间声明。

## 理解成员

这种类型还没有任何成员封装在其中。我们将在接下来的页面上创建一些。成员可以是字段、方法或两者的特殊版本。你将在这里找到它们的描述：

+   **字段**用于存储数据。还有三种特殊类别的字段，如下所示：

    +   **常量**：数据永不改变。编译器会将数据直接复制到任何读取它的代码中。

    +   **只读**：类实例化后数据不能改变，但数据可以在实例化时计算或从外部源加载。

    +   **事件**：数据引用一个或多个你希望在某些事情发生时执行的方法，例如点击按钮或响应来自其他代码的请求。事件将在*第六章*，*实现接口和继承类*中介绍。

+   **方法**用于执行语句。你在学习*第四章*，*编写、调试和测试函数*中的函数时看到了一些例子。还有四种特殊类别的方法：

    +   **构造函数**：当你使用`new`关键字分配内存以实例化类时执行语句。

    +   **属性**：当你获取或设置数据时执行语句。数据通常存储在字段中，但也可能存储在外部或在运行时计算。属性是封装字段的首选方式，除非需要暴露字段的内存地址。

    +   **索引器**：当你使用"数组"语法`[]`获取或设置数据时，执行这些语句。

    +   **运算符**：当你在你的类型的操作数上使用运算符如`+`和`/`时，执行这些语句。

## 实例化一个类

在本节中，我们将创建一个`Person`类的实例。

### 引用程序集

在我们能够实例化一个类之前，我们需要从另一个项目引用包含该类的程序集。我们将在控制台应用程序中使用该类：

1.  使用你偏好的编码工具，在`Chapter05`工作区/解决方案中添加一个名为`PeopleApp`的新控制台应用程序。

1.  如果你使用的是 Visual Studio Code：

    1.  选择`PeopleApp`作为活动 OmniSharp 项目。当你看到弹出警告消息说缺少必需资产时，点击**是**以添加它们。

    1.  编辑`PeopleApp.csproj`以添加对`PacktLibrary`的项目引用，如下所示突出显示：

        ```cs
        <Project Sdk="Microsoft.NET.Sdk">
          <PropertyGroup>
            <OutputType>Exe</OutputType>
            <TargetFramework>net6.0</TargetFramework>
            <Nullable>enable</Nullable>
            <ImplicitUsings>enable</ImplicitUsings>
          </PropertyGroup>
         **<ItemGroup>**
         **<ProjectReference Include=****"../PacktLibrary/PacktLibrary.csproj"** **/>**
         **</ItemGroup>**
        </Project> 
        ```

    1.  在终端中，输入命令编译`PeopleApp`项目及其依赖项`PacktLibrary`项目，如下所示：

        ```cs
        dotnet build 
        ```

1.  如果你使用的是 Visual Studio：

    1.  将解决方案的启动项目设置为当前选择。

    1.  **解决方案资源管理器**中，选择`PeopleApp`项目，导航至**项目** | **添加项目引用…**，勾选复选框选择`PacktLibrary`项目，然后点击**确定**。

    1.  导航至**生成** | **生成 PeopleApp**。

## 导入命名空间以使用类型

现在，我们准备好编写与`Person`类交互的语句了：

1.  在`PeopleApp`项目/文件夹中，打开`Program.cs`。

1.  在`Program.cs`文件顶部，删除注释，并添加语句以导入我们`Person`类的命名空间并静态导入`Console`类，如下所示：

    ```cs
    using Packt.Shared;
    using static System.Console; 
    ```

1.  在`Program.cs`中，添加以下语句：

    +   创建`Person`类型的实例。

    +   使用自身的文本描述输出实例。

    `new`关键字为对象分配内存并初始化任何内部数据。我们可以使用`var`代替`Person`类名，但随后我们需要在`new`关键字后指定`Person`，如下所示：

    ```cs
    // var bob = new Person(); // C# 1.0 or later
    Person bob = new(); // C# 9.0 or later
    WriteLine(bob.ToString()); 
    ```

    你可能会疑惑，“为什么`bob`变量有一个名为`ToString`的方法？`Person`类是空的！”别担心，我们即将揭晓！

1.  运行代码并查看结果，如下所示：

    ```cs
    Packt.Shared.Person 
    ```

## 理解对象

尽管我们的`Person`类没有明确选择继承自某个类型，但所有类型最终都直接或间接继承自一个名为`System.Object`的特殊类型。

`System.Object`类型中`ToString`方法的实现仅输出完整的命名空间和类型名称。

回到原始的`Person`类，我们本可以明确告诉编译器`Person`继承自`System.Object`类型，如下所示：

```cs
public class Person : System.Object 
```

当类 B 继承自类 A 时，我们称 A 为基类或父类，B 为派生类或子类。在这种情况下，`System.Object`是基类或父类，`Person`是派生类或子类。

你也可以使用 C#别名关键字`object`，如下列代码所示：

```cs
public class Person : object 
```

### 继承自 System.Object

让我们使我们的类显式继承自`object`，然后回顾所有对象拥有的成员：

1.  修改你的`Person`类，使其显式继承自`object`。

1.  点击`object`关键字内部，按 F12，或者右键点击`object`关键字并选择**转到定义**。

你将看到微软定义的`System.Object`类型及其成员。这方面的细节你目前无需了解，但请注意它有一个名为`ToString`的方法，如*图 5.1*所示：

![图形用户界面，文本，应用程序，电子邮件 描述自动生成](img/B17442_05_01.png)

**图 5.1**：System.Object 类定义

**最佳实践**：假设其他程序员知道，如果未指定继承，则类将继承自`System.Object`。

# 在字段中存储数据

在本节中，我们将定义类中的一系列字段，用于存储有关个人的信息。

## 定义字段

假设我们已决定一个人由姓名和出生日期组成。我们将把这两个值封装在一个人内部，并且这些值对外可见。

在`Person`类内部，编写语句以声明两个公共字段，用于存储一个人的姓名和出生日期，如下列代码所示：

```cs
public class Person : object
{
  // fields
  public string Name;
  public DateTime DateOfBirth;
} 
```

字段可以使用任何类型，包括数组和集合，如列表和字典。如果你需要在单个命名字段中存储多个值，这些类型就会派上用场。在本例中，一个人只有一个名字和一个出生日期。

## 理解访问修饰符

封装的一部分是选择成员的可见性。

请注意，正如我们对类所做的那样，我们明确地对这些字段应用了`public`关键字。如果我们没有这样做，那么它们将默认为`private`，这意味着它们只能在类内部访问。

有四个访问修饰符关键字，以及两种访问修饰符关键字的组合，你可以将其应用于类成员，如字段或方法，如下表所示：

| 访问修饰符 | 描述 |
| --- | --- |
| `private` | 成员仅在类型内部可访问。这是默认设置。 |
| `internal` | 成员在类型内部及同一程序集中的任何类型均可访问。 |
| `protected` | 成员在类型内部及其任何派生类型中均可访问。 |
| `public` | 成员在任何地方均可访问。 |
| `internal``protected` | 成员在类型内部、同一程序集中的任何类型以及任何派生类型中均可访问。相当于一个虚构的访问修饰符，名为`internal_or_protected`。 |
| `private``protected` | 成员在类型内部、任何派生类型以及同一程序集中均可访问。相当于一个虚构的访问修饰符，名为`internal_and_protected`。这种组合仅在 C# 7.2 或更高版本中可用。 |

**良好实践**：明确地对所有类型成员应用一个访问修饰符，即使你想要使用成员的隐式访问修饰符，即`private`。此外，字段通常应该是`private`或`protected`，然后你应该创建`public`属性来获取或设置字段值。这是因为它控制访问。你将在本章后面这样做。

## 设置和输出字段值

现在我们将在你的代码中使用这些字段：

1.  在`Program.cs`顶部，确保导入了`System`命名空间。我们需要这样做才能使用`DateTime`类型。

1.  实例化`bob`后，添加语句以设置他的姓名和出生日期，然后以美观的格式输出这些字段，如下所示：

    ```cs
    bob.Name = "Bob Smith";
    bob.DateOfBirth = new DateTime(1965, 12, 22); // C# 1.0 or later
    WriteLine(format: "{0} was born on {1:dddd, d MMMM yyyy}", 
      arg0: bob.Name,
      arg1: bob.DateOfBirth); 
    ```

    我们本可以使用字符串插值，但对于长字符串，它会在多行上换行，这在印刷书籍中可能更难以阅读。在本书的代码示例中，请记住`{0}`是`arg0`的占位符，依此类推。

1.  运行代码并查看结果，如下所示：

    ```cs
    Bob Smith was born on Wednesday, 22 December 1965 
    ```

    根据你的地区设置（即语言和文化），你的输出可能看起来不同。

    `arg1`的格式代码由几个部分组成。`dddd`表示星期几的名称。`d`表示月份中的日期号。`MMMM`表示月份的名称。小写的`m`用于时间值中的分钟。`yyyy`表示年份的完整数字。`yy`表示两位数的年份。

    你还可以使用花括号的简写**对象初始化器**语法初始化字段。让我们看看如何操作。

1.  在现有代码下方添加语句以创建另一个名为 Alice 的新人。注意在向控制台输出她的出生日期时使用的不同格式代码，如下所示：

    ```cs
    Person alice = new()
    {
      Name = "Alice Jones",
      DateOfBirth = new(1998, 3, 7) // C# 9.0 or later
    };
    WriteLine(format: "{0} was born on {1:dd MMM yy}",
      arg0: alice.Name,
      arg1: alice.DateOfBirth); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Alice Jones was born on 07 Mar 98 
    ```

## 使用枚举类型存储值

有时，一个值需要是有限选项集中的一个。例如，世界上有七大古代奇迹，一个人可能有一个最喜欢的。在其他时候，一个值需要是有限选项集的组合。例如，一个人可能有一个他们想要访问的古代世界奇迹的遗愿清单。我们能够通过定义一个枚举类型来存储这些数据。

枚举类型是一种非常高效的方式来存储一个或多个选择，因为它内部使用整数值与`string`描述的查找表相结合：

1.  向`PacktLibrary`项目添加一个名为`WondersOfTheAncientWorld.cs`的新文件。

1.  修改`WondersOfTheAncientWorld.cs`文件，如下所示：

    ```cs
    namespace Packt.Shared
    {
      public enum WondersOfTheAncientWorld
      {
        GreatPyramidOfGiza,
        HangingGardensOfBabylon,
        StatueOfZeusAtOlympia,
        TempleOfArtemisAtEphesus,
        MausoleumAtHalicarnassus,
        ColossusOfRhodes,
        LighthouseOfAlexandria
      }
    } 
    ```

    **良好实践**：如果你在.NET Interactive 笔记本中编写代码，那么包含`enum`的代码单元格必须位于定义`Person`类的代码单元格之上。

1.  在`Person`类中，向字段列表添加以下语句：

    ```cs
    public WondersOfTheAncientWorld FavoriteAncientWonder; 
    ```

1.  在`Program.cs`中，添加以下语句：

    ```cs
    bob.FavoriteAncientWonder = WondersOfTheAncientWorld.StatueOfZeusAtOlympia;
    WriteLine(
      format: "{0}'s favorite wonder is {1}. Its integer is {2}.",
      arg0: bob.Name,
      arg1: bob.FavoriteAncientWonder,
      arg2: (int)bob.FavoriteAncientWonder); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Bob Smith's favorite wonder is StatueOfZeusAtOlympia. Its integer is 2. 
    ```

`enum`值内部作为`int`存储以提高效率。`int`值从`0`开始自动分配，因此我们的`enum`中的第三个世界奇迹的值为`2`。你可以分配`enum`中未列出的`int`值。如果找不到匹配项，它们将输出为`int`值而不是名称。

## 使用枚举类型存储多个值

对于愿望清单，我们可以创建一个`enum`实例的数组或集合，本章后面将解释集合，但有一个更好的方法。我们可以使用`enum`**标志**将多个选择合并为一个值：

1.  通过为`enum`添加`[System.Flags]`属性进行修改，并为每个代表不同位列的奇迹显式设置一个`byte`值，如下列代码中突出显示的那样：

    ```cs
    namespace Packt.Shared
    {
     **[****System.Flags****]**
      public enum WondersOfTheAncientWorld **:** **byte**
      {
        **None                     =** **0b****_0000_0000,** **// i.e. 0**
        GreatPyramidOfGiza       **=** **0b****_0000_0001,** **// i.e. 1**
        HangingGardensOfBabylon  **=** **0b****_0000_0010,** **// i.e. 2**
        StatueOfZeusAtOlympia    **=** **0b****_0000_0100,** **// i.e. 4**
        TempleOfArtemisAtEphesus **=** **0b****_0000_1000,** **// i.e. 8**
        MausoleumAtHalicarnassus **=** **0b****_0001_0000,** **// i.e. 16**
        ColossusOfRhodes         **=** **0b****_0010_0000,** **// i.e. 32**
        LighthouseOfAlexandria   **=** **0b****_0100_0000** **// i.e. 64**
      }
    } 
    ```

    我们正在为每个选择分配明确的值，这些值在查看内存中存储的位时不会重叠。我们还应该用`System.Flags`属性装饰`enum`类型，以便当值返回时，它可以自动与多个值匹配，作为逗号分隔的`string`而不是返回`int`值。

    通常，`enum`类型内部使用`int`变量，但由于我们不需要那么大的值，我们可以通过告诉它使用`byte`变量来减少 75%的内存需求，即每个值 1 字节而不是 4 字节。

    如果我们想表明我们的愿望清单包括*巴比伦空中花园*和*哈利卡纳苏斯的摩索拉斯陵墓*这两大古代世界奇迹，那么我们希望将`16`和`2`位设置为`1`。换句话说，我们将存储值`18`：

    | 64 | 32 | 16 | 8 | 4 | 2 | 1 |
    | --- | --- | --- | --- | --- | --- | --- |
    | 0 | 0 | 1 | 0 | 0 | 1 | 0 |

1.  在`Person`类中，添加以下语句到你的字段列表中，如下列代码所示：

    ```cs
    public WondersOfTheAncientWorld BucketList; 
    ```

1.  在`Program.cs`中，添加语句使用`|`运算符（按位逻辑或）来组合`enum`值以设置愿望清单。我们也可以使用数字 18 强制转换为`enum`类型来设置值，如注释所示，但我们不应该这样做，因为这会使代码更难以理解，如下列代码所示：

    ```cs
    bob.BucketList = 
      WondersOfTheAncientWorld.HangingGardensOfBabylon
      | WondersOfTheAncientWorld.MausoleumAtHalicarnassus;
    // bob.BucketList = (WondersOfTheAncientWorld)18;
    WriteLine($"{bob.Name}'s bucket list is {bob.BucketList}"); 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    Bob Smith's bucket list is HangingGardensOfBabylon, MausoleumAtHalicarnassus 
    ```

**最佳实践**：使用`enum`值来存储离散选项的组合。如果有最多 8 个选项，则从`byte`派生`enum`类型；如果有最多 16 个选项，则从`ushort`派生；如果有最多 32 个选项，则从`uint`派生；如果有最多 64 个选项，则从`ulong`派生。

# 使用集合存储多个值

现在，让我们添加一个字段来存储一个人的子女。这是一个聚合的例子，因为子女是与当前人物相关联的类的实例，但并不属于该人物本身。我们将使用泛型`List<T>`集合类型，它可以存储任何类型的有序集合。你将在*第八章*，*使用常见的.NET 类型*中了解更多关于集合的内容。现在，只需跟随操作：

1.  在`Person.cs`中，导入`System.Collections.Generic`命名空间，如下面的代码所示：

    ```cs
    using System.Collections.Generic; // List<T> 
    ```

1.  在`Person`类中声明一个新字段，如下面的代码所示：

    ```cs
    public List<Person> Children = new List<Person>(); 
    ```

`List<Person>`读作“Person 列表”，例如，“名为`Children`的属性的类型是`Person`实例的列表。”我们明确地将类库的目标更改为.NET Standard 2.0（使用 C# 7 编译器），因此我们不能使用目标类型的新来初始化`Children`字段。如果我们保持目标为.NET 6.0，那么我们可以使用目标类型的新，如下面的代码所示：

```cs
public List<Person> Children = new(); 
```

我们必须确保在向集合添加项之前，集合已初始化为一个新的`Person`列表实例，否则字段将为`null`，当我们尝试使用其任何成员（如`Add`）时，将抛出运行时异常。

## 理解泛型集合

`List<T>`类型中的尖括号是 C#的一个特性，称为**泛型**，于 2005 年随 C# 2.0 引入。这是一个用于创建**强类型**集合的术语，即编译器明确知道集合中可以存储哪种类型的对象。泛型提高了代码的性能和正确性。

**强类型**与**静态类型**有不同的含义。旧的`System.Collection`类型静态地包含弱类型的`System.Object`项。新的`System.Collection.Generic`类型静态地包含强类型的`<T>`实例。

讽刺的是，*泛型*这一术语意味着我们可以使用更具体的静态类型！

1.  在`Program.cs`中，添加语句为`Bob`添加两个孩子，然后展示他有多少孩子以及他们的名字，如下面的代码所示：

    ```cs
    bob.Children.Add(new Person { Name = "Alfred" }); // C# 3.0 and later
    bob.Children.Add(new() { Name = "Zoe" }); // C# 9.0 and later
    WriteLine(
      $"{bob.Name} has {bob.Children.Count} children:");
    for (int childIndex = 0; childIndex < bob.Children.Count; childIndex++)
    {
      WriteLine($"  {bob.Children[childIndex].Name}");
    } 
    ```

    我们也可以使用`foreach`语句来遍历集合。作为额外的挑战，将`for`语句改为使用`foreach`输出相同的信息。

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Bob Smith has 2 children:
      Alfred
      Zoe 
    ```

## 将字段设为静态

到目前为止我们创建的字段都是**实例**成员，意味着每个字段在创建的每个类实例中都有不同的值。`alice`和`bob`变量具有不同的`Name`值。

有时，您希望定义一个在所有实例中共享的单一值的字段。

这些被称为**静态** *成员*，因为字段不是唯一可以静态的成员。让我们看看使用`static`字段可以实现什么：

1.  在`PacktLibrary`项目中，添加一个名为`BankAccount.cs`的新类文件。

1.  修改类，使其具有三个字段，两个实例字段和一个静态字段，如下面的代码所示：

    ```cs
    namespace Packt.Shared
    {
      public class BankAccount
      {
        public string AccountName; // instance member
        public decimal Balance; // instance member
        public static decimal InterestRate; // shared member
      }
    } 
    ```

    每个`BankAccount`实例都将有自己的`AccountName`和`Balance`值，但所有实例将共享一个`InterestRate`值。

1.  在`Program.cs`中，添加语句以设置共享的利率，然后创建两个`BankAccount`类型的实例，如下面的代码所示：

    ```cs
    BankAccount.InterestRate = 0.012M; // store a shared value
    BankAccount jonesAccount = new(); // C# 9.0 and later
    jonesAccount.AccountName = "Mrs. Jones"; 
    jonesAccount.Balance = 2400;
    WriteLine(format: "{0} earned {1:C} interest.",
      arg0: jonesAccount.AccountName,
      arg1: jonesAccount.Balance * BankAccount.InterestRate);
    BankAccount gerrierAccount = new(); 
    gerrierAccount.AccountName = "Ms. Gerrier"; 
    gerrierAccount.Balance = 98;
    WriteLine(format: "{0} earned {1:C} interest.",
      arg0: gerrierAccount.AccountName,
      arg1: gerrierAccount.Balance * BankAccount.InterestRate); 
    ```

    `:C`是一个格式代码，告诉.NET 使用货币格式显示数字。在第八章《使用常见的.NET 类型》中，你将学习如何控制决定货币符号的文化。目前，它将使用你操作系统安装的默认设置。我住在英国伦敦，因此我的输出显示的是英镑（£）。

1.  运行代码并查看附加输出：

    ```cs
    Mrs. Jones earned £28.80 interest. 
    Ms. Gerrier earned £1.18 interest. 
    ```

字段并非唯一可声明为静态的成员。构造函数、方法、属性及其他成员也可以是静态的。

## 将字段设为常量

如果某个字段的值永远不会改变，你可以使用`const`关键字，并在编译时赋值一个字面量：

1.  在`Person.cs`中，添加以下代码：

    ```cs
     // constants
    public const string Species = "Homo Sapien"; 
    ```

1.  要获取常量字段的值，你必须写出类名，而不是类的实例名。在`Program.cs`中，添加一条语句，将 Bob 的名字和物种输出到控制台，如下所示：

    ```cs
    WriteLine($"{bob.Name} is a {Person.Species}"); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Bob Smith is a Homo Sapien 
    ```

    微软类型中的`const`字段示例包括`System.Int32.MaxValue`和`System.Math.PI`，因为这两个值永远不会改变，如图 5.2 所示：

    ![图形用户界面，文本，应用程序，电子邮件 描述自动生成](img/B17442_05_02.png)

图 5.2：常量示例

**最佳实践**：常量并不总是最佳选择，原因有二：其值必须在编译时已知，并且必须能表示为字面量`string`、`Boolean`或数值。对`const`字段的每次引用在编译时都会被替换为字面量值，因此，如果未来版本中该值发生变化，且你未重新编译引用它的任何程序集以获取新值，则不会反映这一变化。

## 将字段设为只读

对于不应更改的字段，通常更好的选择是将其标记为只读：

1.  在`Person.cs`中，添加一条语句，声明一个实例只读字段以存储人的母星，如下所示：

    ```cs
    // read-only fields
    public readonly string HomePlanet = "Earth"; 
    ```

1.  在`Program.cs`中，添加一条语句，将 Bob 的名字和母星输出到控制台，如下所示：

    ```cs
    WriteLine($"{bob.Name} was born on {bob.HomePlanet}"); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Bob Smith was born on Earth 
    ```

**最佳实践**：出于两个重要原因，建议使用只读字段而非常量字段：其值可以在运行时计算或加载，并且可以使用任何可执行语句来表达。因此，只读字段可以通过构造函数或字段赋值来设置。对字段的每次引用都是活跃的，因此任何未来的更改都将被调用代码正确反映。

你还可以声明`static` `readonly`字段，其值将在该类型的所有实例之间共享。

## 使用构造函数初始化字段

字段通常需要在运行时初始化。你可以在构造函数中执行此操作，该构造函数将在使用`new`关键字创建类的实例时被调用。构造函数在任何字段被使用该类型的代码设置之前执行。

1.  在`Person.cs`中，在现有的只读`HomePlanet`字段之后添加语句以定义第二个只读字段，然后在构造函数中设置`Name`和`Instantiated`字段，如下面的代码中突出显示的那样：

    ```cs
    // read-only fields
    public readonly string HomePlanet = "Earth";
    **public****readonly** **DateTime Instantiated;**
    **// constructors**
    **public****Person****()**
    **{**
    **// set default values for fields**
    **// including read-only fields**
     **Name =** **"Unknown"****;** 
     **Instantiated = DateTime.Now;**
    **}** 
    ```

1.  在`Program.cs`中，添加语句以实例化一个新的人，然后输出其初始字段值，如下面的代码所示：

    ```cs
    Person blankPerson = new();
    WriteLine(format:
      "{0} of {1} was created at {2:hh:mm:ss} on a {2:dddd}.",
      arg0: blankPerson.Name,
      arg1: blankPerson.HomePlanet,
      arg2: blankPerson.Instantiated); 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Unknown of Earth was created at 11:58:12 on a Sunday 
    ```

### 定义多个构造函数

一个类型中可以有多个构造函数。这对于鼓励开发者在字段上设置初始值特别有用：

1.  在`Person.cs`中，添加语句以定义第二个构造函数，允许开发者为人的姓名和家乡星球设置初始值，如下面的代码所示：

    ```cs
    public Person(string initialName, string homePlanet)
    {
      Name = initialName;
      HomePlanet = homePlanet;
      Instantiated = DateTime.Now;
    } 
    ```

1.  在`Program.cs`中，添加语句以使用带有两个参数的构造函数创建另一个人，如下面的代码所示：

    ```cs
    Person gunny = new(initialName: "Gunny", homePlanet: "Mars");
    WriteLine(format:
      "{0} of {1} was created at {2:hh:mm:ss} on a {2:dddd}.",
      arg0: gunny.Name,
      arg1: gunny.HomePlanet,
      arg2: gunny.Instantiated); 
    ```

1.  运行代码并查看结果：

    ```cs
    Gunny of Mars was created at 11:59:25 on a Sunday 
    ```

构造函数是一种特殊的方法类别。让我们更详细地看看方法。

# 编写和调用方法

**方法**是一种类型的成员，它执行一组语句。它们是属于某个类型的函数。

## 从方法中返回值

方法可以返回单个值或不返回任何值：

+   执行某些操作但不返回值的方法通过在方法名称前使用`void`类型来表示这一点。

+   执行某些操作并返回值的方法通过在方法名称前使用返回值的类型来表示这一点。

例如，在下一个任务中，你将创建两个方法：

+   `WriteToConsole`：这将执行一个动作（向控制台写入一些文本），但它不会从方法中返回任何内容，由`void`关键字表示。

+   `GetOrigin`：这将返回一个文本值，由`string`关键字表示。

让我们编写代码：

1.  在`Person.cs`中，添加语句以定义我之前描述的两种方法，如下面的代码所示：

    ```cs
    // methods
    public void WriteToConsole()
    {
      WriteLine($"{Name} was born on a {DateOfBirth:dddd}.");
    }
    public string GetOrigin()
    {
      return $"{Name} was born on {HomePlanet}.";
    } 
    ```

1.  在`Program.cs`中，添加语句以调用这两个方法，如下面的代码所示：

    ```cs
    bob.WriteToConsole(); 
    WriteLine(bob.GetOrigin()); 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Bob Smith was born on a Wednesday. 
    Bob Smith was born on Earth. 
    ```

## 使用元组组合多个返回值

每个方法只能返回一个具有单一类型的值。该类型可以是简单类型，如前例中的`string`，复杂类型，如`Person`，或集合类型，如`List<Person>`。

假设我们想要定义一个名为`GetTheData`的方法，该方法需要返回一个`string`值和一个`int`值。我们可以定义一个名为`TextAndNumber`的新类，其中包含一个`string`字段和一个`int`字段，并返回该复杂类型的实例，如下面的代码所示：

```cs
public class TextAndNumber
{
  public string Text;
  public int Number;
}
public class LifeTheUniverseAndEverything
{
  public TextAndNumber GetTheData()
  {
    return new TextAndNumber
    {
      Text = "What's the meaning of life?",
      Number = 42
    };
  }
} 
```

但仅仅为了组合两个值而定义一个类是不必要的，因为在现代版本的 C#中我们可以使用**元组**。元组是一种高效地将两个或更多值组合成单一单元的方式。我发音为 tuh-ples，但我听说其他开发者发音为 too-ples。番茄，西红柿，土豆，马铃薯，我想。

元组自 F#等语言的第一个版本以来就一直是其中的一部分，但.NET 直到 2010 年使用`System.Tuple`类型才在.NET 4.0 中添加了对它们的支持。

### 语言对元组的支持

直到 2017 年 C# 7.0，C#才通过使用圆括号字符`()`添加了对元组的语言语法支持，同时.NET 引入了一个新的`System.ValueTuple`类型，在某些常见场景下比旧的.NET 4.0 `System.Tuple`类型更高效。C#的元组语法使用了更高效的那个。

让我们来探索元组：

1.  在`Person.cs`中，添加语句以定义一个返回结合了`string`和`int`的元组的方法，如下列代码所示：

    ```cs
    public (string, int) GetFruit()
    {
      return ("Apples", 5);
    } 
    ```

1.  在`Program.cs`中，添加语句以调用`GetFruit`方法，然后自动输出名为`Item1`和`Item2`的元组字段，如下列代码所示：

    ```cs
    (string, int) fruit = bob.GetFruit();
    WriteLine($"{fruit.Item1}, {fruit.Item2} there are."); 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    Apples, 5 there are. 
    ```

### 命名元组的字段

要访问元组的字段，默认名称是`Item1`、`Item2`等。

你可以显式指定字段名称：

1.  在`Person.cs`中，添加语句以定义一个返回具有命名字段的元组的方法，如下列代码所示：

    ```cs
    public (string Name, int Number) GetNamedFruit()
    {
      return (Name: "Apples", Number: 5);
    } 
    ```

1.  在`Program.cs`中，添加语句以调用该方法并输出元组的命名字段，如下列代码所示：

    ```cs
    var fruitNamed = bob.GetNamedFruit();
    WriteLine($"There are {fruitNamed.Number} {fruitNamed.Name}."); 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    There are 5 Apples. 
    ```

### 推断元组名称

如果你是从另一个对象构建元组，你可以使用 C# 7.1 引入的特性，称为**元组名称推断**。

在`Program.cs`中，创建两个元组，每个元组由一个`string`和一个`int`值组成，如下列代码所示：

```cs
var thing1 = ("Neville", 4);
WriteLine($"{thing1.Item1} has {thing1.Item2} children.");
var thing2 = (bob.Name, bob.Children.Count); 
WriteLine($"{thing2.Name} has {thing2.Count} children."); 
```

在 C# 7.0 中，两者都会使用`Item1`和`Item2`命名方案。在 C# 7.1 及更高版本中，`thing2`可以推断出名称`Name`和`Count`。

### 解构元组

你也可以将元组解构成单独的变量。解构声明的语法与命名字段元组相同，但没有为元组指定名称的变量，如下列代码所示：

```cs
// store return value in a tuple variable with two fields
(string TheName, int TheNumber) tupleWithNamedFields = bob.GetNamedFruit();
// tupleWithNamedFields.TheName
// tupleWithNamedFields.TheNumber
// deconstruct return value into two separate variables
(string name, int number) = GetNamedFruit();
// name
// number 
```

这具有将元组分解为其各个部分并将这些部分分配给新变量的效果。

1.  在`Program.cs`中，添加语句以解构从`GetFruit`方法返回的元组，如下列代码所示：

    ```cs
    (string fruitName, int fruitNumber) = bob.GetFruit();
    WriteLine($"Deconstructed: {fruitName}, {fruitNumber}"); 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    Deconstructed: Apples, 5 
    ```

### 解构类型

元组并非唯一可被解构的类型。任何类型都可以有名为`Deconstruct`的特殊方法，这些方法能将对象分解为各个部分。让我们为`Person`类实现一些这样的方法：

1.  在`Person.cs`中，添加两个`Deconstruct`方法，为我们要分解的部分定义`out`参数，如下面的代码所示：

    ```cs
    // deconstructors
    public void Deconstruct(out string name, out DateTime dob)
    {
      name = Name;
      dob = DateOfBirth;
    }
    public void Deconstruct(out string name, 
      out DateTime dob, out WondersOfTheAncientWorld fav)
    {
      name = Name;
      dob = DateOfBirth;
      fav = FavoriteAncientWonder;
    } 
    ```

1.  在`Program.cs`中，添加语句以分解`bob`，如下面的代码所示：

    ```cs
    // Deconstructing a Person
    var (name1, dob1) = bob;
    WriteLine($"Deconstructed: {name1}, {dob1}");
    var (name2, dob2, fav2) = bob;
    WriteLine($"Deconstructed: {name2}, {dob2}, {fav2}"); 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Deconstructed: Bob Smith, 22/12/1965 00:00:00
    Deconstructed: Bob Smith, 22/12/1965 00:00:00, StatueOfZeusAtOlympia
    B 
    ```

## 定义和传递参数给方法

方法可以接收参数来改变其行为。参数的定义有点像变量声明，但位于方法的括号内，正如本章前面在构造函数中看到的那样。我们来看更多例子：

1.  在`Person.cs`中，添加语句以定义两种方法，第一种没有参数，第二种有一个参数，如下面的代码所示：

    ```cs
    public string SayHello()
    {
      return $"{Name} says 'Hello!'";
    }
    public string SayHelloTo(string name)
    {
      return $"{Name} says 'Hello {name}!'";
    } 
    ```

1.  在`Program.cs`中，添加语句以调用这两种方法，并将返回值写入控制台，如下面的代码所示：

    ```cs
    WriteLine(bob.SayHello()); 
    WriteLine(bob.SayHelloTo("Emily")); 
    ```

1.  运行代码并查看结果：

    ```cs
    Bob Smith says 'Hello!'
    Bob Smith says 'Hello Emily!' 
    ```

在输入调用方法的语句时，IntelliSense 会显示一个工具提示，其中包含任何参数的名称和类型，以及方法的返回类型，如*图 5.3*所示：

![图形用户界面，文本，网站 描述自动生成](img/B17442_05_03.png)

*图 5.3*：没有重载的方法的 IntelliSense 工具提示

## 方法重载

我们不必为两种不同的方法取不同的名字，可以给这两种方法取相同的名字。这是允许的，因为这两种方法的签名不同。

**方法签名**是一系列参数类型，可以在调用方法时传递。重载方法不能仅在返回类型上有所不同。

1.  在`Person.cs`中，将`SayHelloTo`方法的名称更改为`SayHello`。

1.  在`Program.cs`中，将方法调用更改为使用`SayHello`方法，并注意方法的快速信息告诉你它有一个额外的重载，1/2，以及 2/2，如*图 5.4*所示：![图形用户界面 描述自动生成，中等置信度](img/B17442_05_04.png)

*图 5.4*：有重载的方法的 IntelliSense 工具提示

**最佳实践**：使用重载方法简化类，使其看起来方法更少。

## 传递可选和命名参数

另一种简化方法的方式是使参数可选。通过在方法参数列表中赋予默认值，可以使参数成为可选参数。可选参数必须始终位于参数列表的最后。

我们现在将创建一个具有三个可选参数的方法：

1.  在`Person.cs`中，添加语句以定义该方法，如下面的代码所示：

    ```cs
    public string OptionalParameters(
      string command  = "Run!",
      double number = 0.0,
      bool active = true)
    {
      return string.Format(
        format: "command is {0}, number is {1}, active is {2}",
        arg0: command,
        arg1: number,
        arg2: active);
    } 
    ```

1.  在`Program.cs`中，添加一条语句以调用该方法，并将返回值写入控制台，如下面的代码所示：

    ```cs
    WriteLine(bob.OptionalParameters()); 
    ```

1.  随着你输入代码，观察 IntelliSense 的出现。你会看到一个工具提示，显示三个可选参数及其默认值，如*图 5.5*所示：![图形用户界面，文本，应用程序，聊天或短信 描述自动生成](img/B17442_05_05.png)

    图 5.5：IntelliSense 显示您键入代码时的可选参数

1.  运行代码并查看结果，如下所示：

    ```cs
    command is Run!, number is 0, active is True 
    ```

1.  在`Program.cs`中，添加一条语句，为`command`参数传递一个`string`值，为`number`参数传递一个`double`值，如下所示：

    ```cs
    WriteLine(bob.OptionalParameters("Jump!", 98.5)); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    command is Jump!, number is 98.5, active is True 
    ```

`command`和`number`参数的默认值已被替换，但`active`的默认值仍然是`true`。

### 调用方法时命名参数值

调用方法时，可选参数通常与命名参数结合使用，因为命名参数允许值以与声明不同的顺序传递。

1.  在`Program.cs`中，添加一条语句，为`command`参数传递一个`string`值，为`number`参数传递一个`double`值，但使用命名参数，以便它们传递的顺序可以互换，如下所示：

    ```cs
    WriteLine(bob.OptionalParameters(
      number: 52.7, command: "Hide!")); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    command is Hide!, number is 52.7, active is True 
    ```

    您甚至可以使用命名参数跳过可选参数。

1.  在`Program.cs`中，添加一条语句，按位置顺序为`command`参数传递一个`string`值，跳过`number`参数，并使用命名的`active`参数，如下所示：

    ```cs
    WriteLine(bob.OptionalParameters("Poke!", active: false)); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    command is Poke!, number is 0, active is False 
    ```

## 控制参数的传递方式

当参数传递给方法时，它可以以三种方式之一传递：

+   通过**值**（默认方式）：将其视为*仅输入*。

+   通过**引用**作为`ref`参数：将其视为*进出*。

+   作为`out`参数：将其视为*仅输出*。

让我们看一些参数传递的例子：

1.  在`Person.cs`中，添加语句以定义一个带有三个参数的方法，一个`in`参数，一个`ref`参数，以及一个`out`参数，如下所示：

    ```cs
    public void PassingParameters(int x, ref int y, out int z)
    {
      // out parameters cannot have a default
      // AND must be initialized inside the method
      z = 99;
      // increment each parameter
      x++; 
      y++; 
      z++;
    } 
    ```

1.  在`Program.cs`中，添加语句以声明一些`int`变量并将它们传递给方法，如下所示：

    ```cs
    int a = 10; 
    int b = 20; 
    int c = 30;
    WriteLine($"Before: a = {a}, b = {b}, c = {c}"); 
    bob.PassingParameters(a, ref b, out c); 
    WriteLine($"After: a = {a}, b = {b}, c = {c}"); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Before: a = 10, b = 20, c = 30 
    After: a = 10, b = 21, c = 100 
    ```

    +   当默认传递变量作为参数时，传递的是其当前值，而不是变量本身。因此，`x`是`a`变量值的副本。`a`变量保持其原始值`10`。

    +   当将变量作为`ref`参数传递时，变量的引用被传递到方法中。因此，`y`是对`b`的引用。当`y`参数递增时，`b`变量也随之递增。

    +   当将变量作为`out`参数传递时，变量的引用被传递到方法中。因此，`z`是对`c`的引用。`c`变量的值被方法内部执行的代码所替换。我们可以在`Main`方法中简化代码，不将值`30`赋给`c`变量，因为它总是会被替换。

### 简化`out`参数

在 C# 7.0 及更高版本中，我们可以简化使用 out 变量的代码。

在`Program.cs`中，添加语句以声明更多变量，包括一个名为`f`的内联声明的`out`参数，如下所示：

```cs
int d = 10; 
int e = 20;
WriteLine($"Before: d = {d}, e = {e}, f doesn't exist yet!");
// simplified C# 7.0 or later syntax for the out parameter 
bob.PassingParameters(d, ref e, out int f); 
WriteLine($"After: d = {d}, e = {e}, f = {f}"); 
```

## 理解 ref 返回

在 C# 7.0 或更高版本中，`ref`关键字不仅用于向方法传递参数；它还可以应用于`return`值。这使得外部变量可以引用内部变量并在方法调用后修改其值。这在高级场景中可能有用，例如，在大数据结构中传递占位符，但这超出了本书的范围。

## 使用 partial 拆分类

在处理大型项目或与多个团队成员合作时，或者在处理特别庞大且复杂的类实现时，能够将类的定义拆分到多个文件中非常有用。您可以通过使用`partial`关键字来实现这一点。

设想我们希望向`Person`类添加由类似对象关系映射器（ORM）的工具自动生成的语句，该工具从数据库读取架构信息。如果该类定义为`partial`，那么我们可以将类拆分为一个自动生成代码文件和一个手动编辑代码文件。

让我们编写一些代码来模拟此示例：

1.  在`Person.cs`中，添加`partial`关键字，如下所示突出显示：

    ```cs
    namespace Packt.Shared
    {
      public **partial** class Person
      { 
    ```

1.  在`PacktLibrary`项目/文件夹中，添加一个名为`PersonAutoGen.cs`的新类文件。

1.  向新文件添加语句，如下所示：

    ```cs
    namespace Packt.Shared
    {
      public partial class Person
      {
      }
    } 
    ```

本章剩余代码将在`PersonAutoGen.cs`文件中编写。

# 通过属性和索引器控制访问

之前，您创建了一个名为`GetOrigin`的方法，该方法返回一个包含人员姓名和来源的`string`。诸如 Java 之类的语言经常这样做。C#有更好的方法：属性。

属性本质上是一个方法（或一对方法），当您想要获取或设置值时，它表现得像字段一样，从而简化了语法。

## 定义只读属性

一个`readonly`属性仅具有`get`实现。

1.  在`PersonAutoGen.cs`中，在`Person`类中，添加语句以定义三个属性：

    1.  第一个属性将使用适用于所有 C#版本的属性语法执行与`GetOrigin`方法相同的角色（尽管它使用了 C# 6 及更高版本中的字符串插值语法）。

    1.  第二个属性将使用 C# 6 及更高版本中的 lambda 表达式体`=>`语法返回一条问候消息。

    1.  第三个属性将计算该人的年龄。

    以下是代码：

    ```cs
    // a property defined using C# 1 - 5 syntax
    public string Origin
    {
      get
      {
        return $"{Name} was born on {HomePlanet}";
      }
    }
    // two properties defined using C# 6+ lambda expression body syntax
    public string Greeting => $"{Name} says 'Hello!'";
    public int Age => System.DateTime.Today.Year - DateOfBirth.Year; 
    ```

    **良好实践**：这不是计算某人年龄的最佳方法，但我们并非学习如何从出生日期计算年龄。若需正确执行此操作，请阅读以下链接中的讨论：[`stackoverflow.com/questions/9/how-do-i-calculate-someones-age-in-c`](https://stackoverflow.com/questions/9/how-do-i-calculate-someones-age-in-c)

1.  在`Program.cs`中，添加获取属性的语句，如下列代码所示：

    ```cs
    Person sam = new()
    {
      Name = "Sam",
      DateOfBirth = new(1972, 1, 27)
    };
    WriteLine(sam.Origin); 
    WriteLine(sam.Greeting); 
    WriteLine(sam.Age); 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    Sam was born on Earth 
    Sam says 'Hello!'
    49 
    ```

输出显示 49，因为我在 2021 年 8 月 15 日运行了控制台应用程序，当时 Sam 49 岁。

## 定义可设置的属性

要创建一个可设置的属性，您必须使用较旧的语法并提供一对方法——不仅仅是`get`部分，还包括`set`部分：

1.  在`PersonAutoGen.cs`中，添加语句以定义一个具有`get`和`set`方法（也称为 getter 和 setter）的`string`属性，如下列代码所示：

    ```cs
    public string FavoriteIceCream { get; set; } // auto-syntax 
    ```

    尽管您没有手动创建一个字段来存储某人的最爱冰淇淋，但它确实存在，由编译器自动为您创建。

    有时，您需要更多控制权来决定属性设置时发生的情况。在这种情况下，您必须使用更详细的语法并手动创建一个`private`字段来存储该属性的值。

1.  在`PersonAutoGen.cs`中，添加语句以定义一个`string`字段和一个具有`get`和`set`的`string`属性，如下列代码所示：

    ```cs
    private string favoritePrimaryColor;
    public string FavoritePrimaryColor
    {
      get
      {
        return favoritePrimaryColor;
      }
      set
      {
        switch (value.ToLower())
        {
          case "red":
          case "green":
          case "blue":
            favoritePrimaryColor = value;
            break;
          default:
            throw new System.ArgumentException(
              $"{value} is not a primary color. " + 
              "Choose from: red, green, blue.");
        }
      }
    } 
    ```

    **最佳实践**：避免在您的 getter 和 setter 中添加过多代码。这可能表明您的设计存在问题。考虑添加私有方法，然后在 setter 和 getter 中调用这些方法，以简化您的实现。

1.  在`Program.cs`中，添加语句以设置 Sam 的最爱冰淇淋和颜色，然后将其写出，如下列代码所示：

    ```cs
    sam.FavoriteIceCream = "Chocolate Fudge";
    WriteLine($"Sam's favorite ice-cream flavor is {sam.FavoriteIceCream}."); 
    sam.FavoritePrimaryColor = "Red";
    WriteLine($"Sam's favorite primary color is {sam.FavoritePrimaryColor}."); 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    Sam's favorite ice-cream flavor is Chocolate Fudge. 
    Sam's favorite primary color is Red. 
    ```

    如果您尝试将颜色设置为除红色、绿色或蓝色之外的任何值，则代码将抛出异常。调用代码随后可以使用`try`语句来显示错误消息。

    **最佳实践**：当您希望验证可以存储的值时，或者在希望进行 XAML 数据绑定时（我们将在*第十九章*，*使用.NET MAUI 构建移动和桌面应用*中介绍），以及当您希望在不使用`GetAge`和`SetAge`这样的方法对的情况下读写字段时，请使用属性而不是字段。

## 要求在实例化时设置属性

C# 10 引入了`required`修饰符。如果您将其用于属性，编译器将确保在实例化时为该属性设置一个值，如下列代码所示：

```cs
public class Book
{
  public required string Isbn { get; set; }
  public string Title { get; set; }
} 
```

如果您尝试实例化一个`Book`而不设置`Isbn`属性，您将看到一个编译器错误，如下列代码所示：

```cs
Book novel = new(); 
```

`required`关键字可能不会出现在.NET 6 的最终发布版本中，因此请将本节视为理论性的。

## 定义索引器

索引器允许调用代码使用数组语法来访问属性。例如，`string`类型定义了一个**索引器**，以便调用代码可以访问`string`中的单个字符。

我们将定义一个索引器，以简化对某人子女的访问：

1.  在`PersonAutoGen.cs`中，添加语句定义一个索引器，以使用孩子的索引获取和设置孩子，如下所示：

    ```cs
    // indexers
    public Person this[int index]
    {
      get
      {
        return Children[index]; // pass on to the List<T> indexer
      }
      set
      {
        Children[index] = value;
      }
    } 
    ```

    您可以重载索引器，以便不同的类型可以用于其参数。例如，除了传递一个`int`值外，您还可以传递一个`string`值。

1.  在`Program.cs`中，添加语句向`Sam`添加两个孩子，然后使用较长的`Children`字段和较短的索引器语法访问第一个和第二个孩子，如下所示：

    ```cs
    sam.Children.Add(new() { Name = "Charlie" }); 
    sam.Children.Add(new() { Name = "Ella" });
    WriteLine($"Sam's first child is {sam.Children[0].Name}"); 
    WriteLine($"Sam's second child is {sam.Children[1].Name}");
    WriteLine($"Sam's first child is {sam[0].Name}"); 
    WriteLine($"Sam's second child is {sam[1].Name}"); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Sam's first child is Charlie 
    Sam's second child is Ella 
    Sam's first child is Charlie 
    Sam's second child is Ella 
    ```

# 对象的模式匹配

在*第三章*，*控制流程、转换类型和处理异常*中，您被介绍了基本的模式匹配。在本节中，我们将更详细地探讨模式匹配。

## 创建并引用.NET 6 类库

增强的模式匹配特性仅在支持 C# 9 或更高版本的现代.NET 类库中可用。

1.  使用您偏好的编码工具，在名为`Chapter05`的工作区/解决方案中添加一个名为`PacktLibraryModern`的新类库。

1.  在`PeopleApp`项目中，添加对`PacktLibraryModern`类库的引用，如下所示：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
      </PropertyGroup>
      <ItemGroup>
        <ProjectReference Include="../PacktLibrary/PacktLibrary.csproj" />
     **<ProjectReference** 
     **Include=****"../PacktLibraryModern/PacktLibraryModern.csproj"** **/>**
      </ItemGroup>
    </Project> 
    ```

1.  构建`PeopleApp`项目。

## 定义飞行乘客

在本例中，我们将定义一些代表飞行中各种类型乘客的类，然后我们将使用带有模式匹配的 switch 表达式来确定他们的飞行费用。

1.  在`PacktLibraryModern`项目/文件夹中，将文件`Class1.cs`重命名为`FlightPatterns.cs`。

1.  在`FlightPatterns.cs`中，添加语句定义三种具有不同属性的乘客类型，如下所示：

    ```cs
    namespace Packt.Shared; // C# 10 file-scoped namespace
    public class BusinessClassPassenger
    {
      public override string ToString()
      {
        return $"Business Class";
      }
    }
    public class FirstClassPassenger
    {
      public int AirMiles { get; set; }
      public override string ToString()
      {
        return $"First Class with {AirMiles:N0} air miles";
      }
    }
    public class CoachClassPassenger
    {
      public double CarryOnKG { get; set; }
      public override string ToString()
      {
        return $"Coach Class with {CarryOnKG:N2} KG carry on";
      }
    } 
    ```

1.  在`Program.cs`中，添加语句定义一个包含五种不同类型和属性值的乘客对象数组，然后枚举它们，输出他们的飞行费用，如下所示：

    ```cs
    object[] passengers = {
      new FirstClassPassenger { AirMiles = 1_419 },
      new FirstClassPassenger { AirMiles = 16_562 },
      new BusinessClassPassenger(),
      new CoachClassPassenger { CarryOnKG = 25.7 },
      new CoachClassPassenger { CarryOnKG = 0 },
    };
    foreach (object passenger in passengers)
    {
      decimal flightCost = passenger switch
      {
        FirstClassPassenger p when p.AirMiles > 35000 => 1500M, 
        FirstClassPassenger p when p.AirMiles > 15000 => 1750M, 
        FirstClassPassenger _                         => 2000M,
        BusinessClassPassenger _                      => 1000M,
        CoachClassPassenger p when p.CarryOnKG < 10.0 => 500M, 
        CoachClassPassenger _                         => 650M,
        _                                             => 800M
      };
      WriteLine($"Flight costs {flightCost:C} for {passenger}");
    } 
    ```

    在审查前面的代码时，请注意以下几点：

    +   要对对象的属性进行模式匹配，您必须命名一个局部变量，该变量随后可以在表达式中使用，如`p`。

    +   仅对类型进行模式匹配时，可以使用`_`来丢弃局部变量。

    +   switch 表达式也使用`_`来表示其默认分支。

1.  运行代码并查看结果，如下所示：

    ```cs
    Flight costs £2,000.00 for First Class with 1,419 air miles 
    Flight costs £1,750.00 for First Class with 16,562 air miles 
    Flight costs £1,000.00 for Business Class
    Flight costs £650.00 for Coach Class with 25.70 KG carry on 
    Flight costs £500.00 for Coach Class with 0.00 KG carry on 
    ```

## C# 9 或更高版本中模式匹配的增强

前面的示例使用的是 C# 8。现在我们将看看 C# 9 及更高版本的一些增强功能。首先，进行类型匹配时不再需要使用下划线来丢弃：

1.  在`Program.cs`中，注释掉 C# 8 语法，添加 C# 9 及更高版本的语法，修改头等舱乘客的分支，使用嵌套的 switch 表达式和新的条件支持，如`>`，如下所示：

    ```cs
    decimal flightCost = passenger switch
    {
      /* C# 8 syntax
      FirstClassPassenger p when p.AirMiles > 35000 => 1500M,
      FirstClassPassenger p when p.AirMiles > 15000 => 1750M,
      FirstClassPassenger                           => 2000M, */
      // C# 9 or later syntax
      FirstClassPassenger p => p.AirMiles switch
      {
        > 35000 => 1500M,
        > 15000 => 1750M,
        _       => 2000M
      },
      BusinessClassPassenger                        => 1000M,
      CoachClassPassenger p when p.CarryOnKG < 10.0 => 500M,
      CoachClassPassenger                           => 650M,
      _                                             => 800M
    }; 
    ```

1.  运行代码以查看结果，并注意它们与之前相同。

您还可以结合使用关系模式和属性模式来避免嵌套的 switch 表达式，如下面的代码所示：

```cs
FirstClassPassenger { AirMiles: > 35000 } => 1500,
FirstClassPassenger { AirMiles: > 15000 } => 1750M,
FirstClassPassenger => 2000M, 
```

# 处理记录

在我们深入了解 C# 9 及更高版本的新记录语言特性之前，让我们先看看一些其他相关的新特性。

## Init-only 属性

您在本章中使用了对象初始化语法来实例化对象并设置初始属性。那些属性也可以在实例化后更改。

有时，您希望将属性视为`只读`字段，以便它们可以在实例化期间设置，但不能在此之后设置。新的`init`关键字使这成为可能。它可以用来替代`set`关键字：

1.  在`PacktLibraryModern`项目/文件夹中，添加一个名为`Records.cs`的新文件。

1.  在`Records.cs`中，定义一个不可变人员类，如下面的代码所示：

    ```cs
    namespace Packt.Shared; // C# 10 file-scoped namespace
    public class ImmutablePerson
    {
      public string? FirstName { get; init; }
      public string? LastName { get; init; }
    } 
    ```

1.  在`Program.cs`中，添加语句以实例化一个新的不可变人员，然后尝试更改其一个属性，如下面的代码所示：

    ```cs
    ImmutablePerson jeff = new() 
    {
      FirstName = "Jeff",
      LastName = "Winger"
    };
    jeff.FirstName = "Geoff"; 
    ```

1.  编译控制台应用程序并注意编译错误，如下面的输出所示：

    ```cs
    Program.cs(254,7): error CS8852: Init-only property or indexer 'ImmutablePerson.FirstName' can only be assigned in an object initializer, or on 'this' or 'base' in an instance constructor or an 'init' accessor. [/Users/markjprice/Code/Chapter05/PeopleApp/PeopleApp.csproj] 
    ```

1.  注释掉尝试在实例化后设置`FirstName`属性的代码。

## 理解记录

Init-only 属性为 C#提供了一些不可变性。您可以通过使用**记录**将这一概念进一步推进。这些是通过使用`record`关键字而不是`class`关键字来定义的。这可以使整个对象不可变，并且在比较时它表现得像一个值。我们将在*第六章*，*实现接口和继承类*中更详细地讨论类、记录和值类型的相等性和比较。

记录不应具有在实例化后更改的任何状态（属性和字段）。相反，想法是您从现有记录创建新记录，其中包含任何更改的状态。这称为非破坏性突变。为此，C# 9 引入了`with`关键字：

1.  在`Records.cs`中，添加一个名为`ImmutableVehicle`的记录，如下面的代码所示：

    ```cs
    public record ImmutableVehicle
    {
      public int Wheels { get; init; }
      public string? Color { get; init; }
      public string? Brand { get; init; }
    } 
    ```

1.  在`Program.cs`中，添加语句以创建一辆`车`，然后创建其变异副本，如下面的代码所示：

    ```cs
    ImmutableVehicle car = new() 
    {
      Brand = "Mazda MX-5 RF",
      Color = "Soul Red Crystal Metallic",
      Wheels = 4
    };
    ImmutableVehicle repaintedCar = car 
      with { Color = "Polymetal Grey Metallic" }; 
    WriteLine($"Original car color was {car.Color}.");
    WriteLine($"New car color is {repaintedCar.Color}."); 
    ```

1.  运行代码以查看结果，并注意变异副本中汽车颜色的变化，如下面的输出所示：

    ```cs
    Original car color was Soul Red Crystal Metallic.
    New car color is Polymetal Grey Metallic. 
    ```

## 记录中的位置数据成员

定义记录的语法可以通过使用位置数据成员大大简化。

### 简化记录中的数据成员

与其使用花括号的对象初始化语法，有时您可能更愿意提供带有位置参数的构造函数，正如您在本章前面所见。您还可以将此与析构函数结合使用，以将对象分解为各个部分，如下面的代码所示：

```cs
public record ImmutableAnimal
{
  public string Name { get; init; } 
  public string Species { get; init; }
  public ImmutableAnimal(string name, string species)
  {
    Name = name;
    Species = species;
  }
  public void Deconstruct(out string name, out string species)
  {
    name = Name;
    species = Species;
  }
} 
```

属性、构造函数和析构函数可以为您自动生成：

1.  在`Records.cs`中，添加语句以使用称为位置记录的简化语法定义另一个记录，如下面的代码所示：

    ```cs
    // simpler way to define a record
    // auto-generates the properties, constructor, and deconstructor
    public record ImmutableAnimal(string Name, string Species); 
    ```

1.  在`Program.cs`中，添加语句以构造和析构不可变动物，如下列代码所示：

    ```cs
    ImmutableAnimal oscar = new("Oscar", "Labrador");
    var (who, what) = oscar; // calls Deconstruct method 
    WriteLine($"{who} is a {what}."); 
    ```

1.  运行应用程序并查看结果，如下列输出所示：

    ```cs
    Oscar is a Labrador. 
    ```

当我们查看 C# 10 支持创建`struct`记录时，你将在*第六章*，*实现接口和继承类*中再次看到记录。

# 实践与探索

通过回答一些问题来测试你的知识和理解，进行一些实践操作，并深入研究本章的主题。

## 练习 5.1 – 测试你的知识

回答以下问题：

1.  访问修饰符关键字的六种组合是什么，它们各自的作用是什么？

1.  当应用于类型成员时，`static`、`const`和`readonly`关键字之间有何区别？

1.  构造函数的作用是什么？

1.  当你想要存储组合值时，为什么应该对`enum`类型应用`[Flags]`属性？

1.  为什么`partial`关键字有用？

1.  什么是元组？

1.  `record`关键字的作用是什么？

1.  重载是什么意思？

1.  字段和属性之间有什么区别？

1.  如何使方法参数变为可选？

## 练习 5.2 – 探索主题

使用以下页面上的链接来了解更多关于本章所涵盖主题的详细信息：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-5---building-your-own-types-with-object-oriented-programming`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-5---building-your-own-types-with-object-oriented-programming)

# 总结

在本章中，你学习了使用面向对象编程（OOP）创建自己的类型。你了解了类型可以拥有的不同类别的成员，包括用于存储数据的字段和执行操作的方法，并运用了 OOP 概念，如聚合和封装。你看到了如何使用现代 C#特性，如关系和属性模式匹配增强、仅初始化属性以及记录的示例。

在下一章中，你将通过定义委托和事件、实现接口以及继承现有类来进一步应用这些概念。
