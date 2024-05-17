# 第五章：05

# 使用面向对象编程构建自己的类型

本章是关于使用**面向对象编程**（**OOP**）创建自己的类型。您将了解类型可以具有的所有不同类别的成员，包括用于存储数据的字段和执行操作的方法。您将使用面向对象编程的概念，如聚合和封装。您还将了解语言功能，如元组语法支持、输出变量、推断的元组名称和默认文字。

本章将涵盖以下主题：

+   谈论面向对象编程

+   构建类库

+   使用字段存储数据

+   编写和调用方法

+   使用属性和索引器控制访问

+   对象的模式匹配

+   使用记录

# 谈论面向对象编程

现实世界中的对象是一种事物，比如汽车或人，而编程中的对象通常代表现实世界中的某些东西，比如产品或银行账户，但这也可以是更抽象的东西。

在 C#中，我们使用`class`（大多数情况）或`struct`（有时）关键字来定义对象的类型。您将在*第六章*中了解类和结构之间的区别，*实现接口和继承类*。您可以将类型视为对象的蓝图或模板。

这里简要描述了面向对象编程的概念：

+   **封装**是与对象相关的数据和操作的组合。例如，`BankAccount`类型可能具有数据，如`Balance`和`AccountName`，以及操作，如`Deposit`和`Withdraw`。在封装时，您通常希望控制可以访问这些操作和数据的内容，例如，限制外部如何访问或修改对象的内部状态。

+   **组合**是关于对象由什么组成。例如，`Car`由不同的部分组成，如四个`Wheel`对象，几个`Seat`对象和一个`Engine`。

+   **聚合**是关于可以与对象组合的内容。例如，`Person`不是`Car`对象的一部分，但他们可以坐在驾驶座上，然后成为汽车的`Driver`——两个单独的对象聚合在一起形成一个新的组件。

+   **继承**是通过让**子类**从**基类**或**超类**派生代码来重用代码。基类中的所有功能都被派生类继承并在派生类中可用。例如，基类或超`Exception`类具有一些成员，这些成员在所有异常中具有相同的实现，而子类或派生`SqlException`类继承这些成员，并具有仅在发生 SQL 数据库异常时相关的额外成员，比如数据库连接的属性。

+   **抽象**是捕捉对象的核心思想并忽略细节或具体内容。C#有`abstract`关键字来正式化这个概念。如果一个类没有明确的**abstract**，那么它可以被描述为**具体**。基类或超类通常是抽象的，例如，超类`Stream`是抽象的，它的子类，如`FileStream`和`MemoryStream`，是具体的。只有具体类才能用来创建对象；抽象类只能用作其他类的基础，因为它们缺少一些实现。抽象是一个棘手的平衡。如果你使一个类更抽象，更多的类将能够从它继承，但与此同时，将会有更少的功能可共享。

+   多态性是允许派生类覆盖继承的操作以提供自定义行为。

# 构建类库

类库程序集将类型组合成易于部署的单元（DLL 文件）。除了学习单元测试时，您只创建了控制台应用程序或.NET 交互式笔记本来包含您的代码。为了使您编写的代码可以在多个项目中重用，您应该将其放入类库程序集中，就像微软一样。

## 创建一个类库

第一个任务是创建一个可重用的.NET 类库：

1.  使用您喜欢的编码工具创建一个新的类库，如下列表所示：

1.  项目模板：**类库** / `classlib`

1.  工作区/解决方案文件和文件夹：`Chapter05`

1.  项目文件和文件夹：`PacktLibrary`

1.  打开 `PacktLibrary.csproj` 文件，并注意默认情况下类库的目标是.NET 6，因此只能与其他.NET 6 兼容的程序集一起使用，如下标记的部分所示：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<ImplicitUsings>enable</ImplicitUsings>

</PropertyGroup>

</Project>

```

1.  修改框架以目标为.NET Standard 2.0，并删除启用可空和隐式使用的条目，如下标记的部分所示：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

**<TargetFramework>netstandard**

**2.0**

**</TargetFramework>**

</PropertyGroup>

</Project>

```

1.  保存并关闭文件。

1.  删除名为 `Class1.cs` 的文件。

1.  编译项目，以便其他项目以后可以引用它：

1.  在 Visual Studio Code 中，输入以下命令：`dotnet build`。

1.  在 Visual Studio 中，导航到**生成** | **生成 PacktLibrary**。

**良好实践**：为了使用最新的 C#语言和.NET 平台功能，将类型放入.NET 6 类库中。为了支持像.NET Core、.NET Framework 和 Xamarin 这样的旧版.NET 平台，将可能重用的类型放入.NET Standard 2.0 类库中。

## 在命名空间中定义类

下一个任务是定义一个代表人的类：

1.  添加一个名为 `Person.cs` 的新类文件。

1.  静态导入 `System.Console`。

1.  将命名空间设置为 `Packt.Shared`。

**良好实践**：我们这样做是因为将类放入一个逻辑命名的命名空间中很重要。一个更好的命名空间名称应该是特定于领域的，例如，`System.Numerics` 用于与高级数字相关的类型。在这种情况下，我们将创建的类型是 `Person`，`BankAccount` 和 `WondersOfTheWorld`，它们没有典型的领域，所以我们将使用更通用的 `Packt.Shared`。

您的类文件现在应该如下面的代码所示：

```cs

使用

System;

using

static

System.Console;

namespace

Packt.Shared

{

public

class

Person

{

}

}

```

请注意，在类之前应用了 C#关键字 `public`。这个关键字是一个**访问修饰符**，它允许任何其他代码访问这个类。

如果您没有明确应用 `public` 关键字，那么它只能在定义它的程序集内部访问。这是因为类的隐式访问修饰符是 `internal`。我们需要这个类在程序集外部可访问，所以我们必须确保它是 `public`。

### 简化命名空间声明

如果您的目标是.NET 6.0，并且因此使用 C# 10 或更高版本，您可以在命名空间声明的末尾加上分号并删除大括号，如下面的代码所示，以简化您的代码：

```cs

使用

System;

namespace

Packt.Shared

; // 此文件中的类在此命名空间中

public

class

Person

{

}

```

这被称为文件范围的命名空间声明。您只能在一个文件中有一个文件范围的命名空间。我们将在本章后面的一个目标为.NET 6.0 的类库中使用这个。

**良好实践**：将创建的每种类型放在自己的文件中，以便您可以使用文件范围的命名空间声明。

## 理解成员

这种类型目前还没有任何封装在其中的成员。我们将在接下来的页面上创建一些成员。成员可以是字段、方法或两者的专门版本。您可以在这里找到它们的描述：

+   **字段** 用于存储数据。还有三种专门的字段类别，如下面的项目所示：

+   **常量**：数据永远不会改变。编译器会将数据直接复制到读取它的任何代码中。

+   **只读**：在类实例化后数据不能更改，但数据可以在实例化时计算或从外部来源加载。

+   **事件**：数据引用一个或多个你希望在发生某些事情时执行的方法，比如点击按钮或响应来自其他代码的请求。事件将在*第六章*，*实现接口和继承类*中介绍。

+   **方法**用于执行语句。在*第四章*，*编写、调试和测试函数*中学习函数时，你看到了一些示例。方法还有四个专门的类别：

+   **构造函数**：当使用`new`关键字分配内存来实例化一个类时执行语句。

+   **属性**：当获取或设置数据时执行语句。数据通常存储在字段中，但也可以存储在外部或在运行时计算。属性是封装字段的首选方式，除非需要公开字段的内存地址。

+   **索引器**：当使用“数组”语法`[]`获取或设置数据时执行语句。

+   **操作符**：当在你的类型的操作数上使用操作符如`+`和`/`时执行语句。

## 实例化一个类

在本节中，我们将创建`Person`类的一个实例。

### 引用一个程序集

在我们实例化一个类之前，我们需要从另一个项目引用包含它的程序集。我们将在控制台应用程序中使用这个类：

1.  使用你喜欢的编码工具向`Chapter05`工作区/解决方案添加一个新的控制台应用程序，命名为`PeopleApp`。

1.  如果你正在使用 Visual Studio Code：

1.  选择`PeopleApp`作为活动的 OmniSharp 项目。当看到弹出的警告消息说需要的资源丢失时，点击**是**来添加它们。

1.  编辑`PeopleApp.csproj`以添加对`PacktLibrary`的项目引用，如下标记中所示：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

<OutputType>Exe</OutputType>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<ImplicitUsings>enable</ImplicitUsings>

</PropertyGroup>

**<ItemGroup>**

**<ProjectReference Include=**

**"../PacktLibrary/PacktLibrary.csproj"**

**/>**

**</ItemGroup>**

</Project>

```

1.  在终端中，输入一个命令来编译`PeopleApp`项目及其依赖的`PacktLibrary`项目，如下命令所示：

```cs

dotnet build

```

1.  如果你正在使用 Visual Studio：

1.  将解决方案的启动项目设置为当前选择。

1.  在**解决方案资源管理器**中，选择`PeopleApp`项目，导航到**项目**|**添加项目引用…**，选中`PacktLibrary`项目的复选框，然后点击**确定**。

1.  导航到**生成**|**生成 PeopleApp**。

## 导入一个命名空间以使用类型

现在，我们准备编写语句来处理`Person`类：

1.  在`PeopleApp`项目/文件夹中，打开`Program.cs`。

1.  在`Program.cs`文件的顶部，删除注释，并添加语句来导入我们`Person`类的命名空间并静态导入`Console`类，如下代码所示：

```cs

使用

Packt.Shared;

使用

static

System.Console;

```

1.  在`Program.cs`中，添加语句：

+   创建`Person`类型的一个实例。

+   输出实例并使用其自身的文本描述。

`new`关键字为对象分配内存并初始化任何内部数据。我们可以在`new`关键字的位置使用`var`代替`Person`类名，但然后我们需要在`new`关键字之后指定`Person`，如下代码所示：

```cs

// var bob = new Person(); // C# 1.0 or later

Person bob = new

(); // C# 9.0 or later

WriteLine(bob.ToString());

```

你可能会想，“为什么`bob`变量有一个名为`ToString`的方法？`Person`类是空的！”别担心，我们马上就会找出答案！

1.  运行代码并查看结果，如下输出所示：

```cs

Packt.Shared.Person

```

## 理解对象

尽管我们的`Person`类没有明确选择从某个类型继承，但所有类型最终都直接或间接地继承自一个名为`System.Object`的特殊类型。

`System.Object`类型中`ToString`方法的实现只是输出完整的命名空间和类型名称。

回到原始的`Person`类中，我们本可以明确告诉编译器`Person`继承自`System.Object`类型，如下面的代码所示：

```cs

public

class

Person

：System.Object

```

当 B 类继承自 A 类时，我们说 A 是基类或超类，B 是派生类或子类。在这种情况下，`System.Object`是基类或超类，`Person`是派生类或子类。

您还可以使用 C#别名关键字`object`，如下面的代码所示：

```cs

public

类

Person

：对象

```

### 从 System.Object 继承

让我们明确地让我们的类继承自`object`，然后回顾一下所有对象都具有哪些成员：

1.  修改您的`Person`类，明确地从`object`继承。

1.  点击`object`关键字内部，然后按 F12，或右键单击`object`关键字，选择**转到定义**。

您将看到 Microsoft 定义的`System.Object`类型及其成员。这是您暂时不需要了解细节的内容，但请注意它有一个名为`ToString`的方法，如*图 5.1*所示：

![图形用户界面，文本，应用程序，电子邮件描述自动生成](img/Image00064.jpg)

图 5.1：System.Object 类定义

**良好实践**：假设其他程序员知道，如果没有指定继承，该类将继承自`System.Object`。

# 在字段内存储数据

在本节中，我们将定义类中的一些字段，以存储有关一个人的信息。

## 定义字段

假设我们已经决定一个人由姓名和出生日期组成。我们将把这两个值封装在一个人内部，并且这些值将对外部可见。

在`Person`类内部，编写语句来声明两个公共字段，用于存储一个人的姓名和出生日期，如下面的代码所示：

```cs

public

类

Person

：对象

{

//字段

public

字符串

Name;

public

DateTime DateOfBirth;

}

```

您可以使用任何类型的字段，包括数组和集合，如列表和字典。如果您需要在一个命名字段中存储多个值，那么可以使用这些。在这个例子中，一个人只有一个姓名和一个出生日期。

## 理解访问修饰符

封装的一部分是选择成员的可见性。

请注意，与类一样，我们明确地将`public`关键字应用于这些字段。如果我们没有这样做，那么它们将隐式地对类私有，这意味着它们只能在类内部访问。

有四个访问修饰符关键字，以及两种访问修饰符关键字的组合，你可以应用到类的成员，比如字段或方法，如下表所示：

| 访问修饰符 | 描述 |
| --- | --- |
| `private` | 成员只能在类型内部访问。这是默认值。 |
| `internal` | 成员在类型内部和同一程序集中的任何类型中都可以访问。 |
| `protected` | 成员在类型内部和任何继承自该类型的类型中都可以访问。 |
| `public` | 成员可以在任何地方访问。 |
| `internal``protected` | 成员在类型内部、同一程序集中的任何类型以及继承自该类型的任何类型中都可以访问。相当于一个名为`internal_or_protected`的虚构访问修饰符。 |
| `private``protected` | 成员在类型内部和任何继承自该类型并且在同一程序集中的类型中都可以访问。相当于一个名为`internal_and_protected`的虚构访问修饰符。此组合仅适用于 C# 7.2 或更高版本。 |

**良好的实践**：明确地将访问修饰符应用于所有类型成员，即使您想要使用成员的隐式访问修饰符，即`private`。此外，字段通常应该是`private`或`protected`，然后您应该创建`public`属性来获取或设置字段值。这是因为它控制访问。您将在本章后面做到这一点。

## 设置和输出字段值

现在我们将在您的代码中使用这些字段：

1.  在`Program.cs`的顶部，确保导入`System`命名空间。我们需要这样做才能使用`DateTime`类型。

1.  实例化`bob`后，添加语句设置他的姓名和出生日期，然后输出这些字段的格式化内容，如下面的代码所示：

```cs

bob.Name = "Bob Smith"

;

bob.DateOfBirth = new

DateTime（1965

，12

，22

）// C# 1.0 或更高版本

WriteLine(format: "{0} was born on {1:dddd, d MMMM yyyy}"

，

参数 0：bob.Name，

参数 1：bob.DateOfBirth);

```

我们也可以使用字符串插值，但对于长字符串，它会跨多行，这在打印的书中可能更难阅读。在本书的代码示例中，请记住`{0}`是`arg0`的占位符，依此类推。

1.  运行代码并查看结果，如下面的输出所示：

```cs

Bob Smith was born on Wednesday, 22 December 1965

```

根据您的区域设置（即语言和文化），您的输出可能会有所不同。

`arg1`的格式代码由几部分组成。`dddd`表示星期几的名称。`d`表示月份的日期。`MMMM`表示月份的名称。小写的`m`用于时间值中的分钟。`yyyy`表示完整的年份。`yy`表示两位数的年份。

您还可以使用简写的**对象初始化程序**语法使用大括号初始化字段。让我们看看。

1.  在现有代码下面添加语句以创建另一个名为 Alice 的新人。请注意，在将她写入控制台时，日期格式代码不同，如下面的代码所示：

```cs

Person alice = new

()

{

Name = "Alice Jones"

，

DateOfBirth = new

（1998

，3

，7

）// C# 9.0 或更高版本

};

WriteLine(format: "{0} was born on {1:dd MMM yy}"

，

参数 0：alice.Name，

参数 1：alice.DateOfBirth);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

Alice Jones was born on 07 Mar 98

```

## 使用枚举类型存储值

有时，一个值需要是有限选项集中的一个。例如，世界上有七个古代奇迹，一个人可能有一个最喜欢的。其他时候，一个值需要是有限选项集的组合。例如，一个人可能有一个想要参观的古代世界奇迹清单。我们可以通过定义`enum`类型来存储这些数据。

`enum`类型是存储一个或多个选择的非常有效的方式，因为在内部，它使用整数值与`string`描述的查找表相结合：

1.  在`PacktLibrary`项目中添加一个名为`WondersOfTheAncientWorld.cs`的新文件。

1.  修改`WondersOfTheAncientWorld.cs`文件，如下面的代码所示：

```cs

命名空间

Packt.Shared

{

公共

枚举

WondersOfTheAncientWorld

{

吉萨的金字塔，

巴比伦空中花园，

奥林匹亚宙斯神像，

以弗所的阿尔忒弥斯神庙，

Halicarnassus 陵墓，

罗得岛的巨像，

亚历山大灯塔

}

}

```

**良好的实践**：如果您在.NET 交互式笔记本中编写代码，则包含`enum`的代码单元格必须位于定义`Person`类的代码单元格之上。

1.  在`Person`类中，将以下语句添加到字段列表中：

```cs

公共

WondersOfTheAncientWorld FavoriteAncientWonder;

```

1.  在`Program.cs`中，添加以下语句：

```cs

bob.FavoriteAncientWonder = WondersOfTheAncientWorld.StatueOfZeusAtOlympia;

WriteLine(

格式："{0}的最喜欢的奇迹是{1}。它的整数是{2}。"

，

参数 0：bob.Name，

参数 1：bob.FavoriteAncientWonder，

参数 2：（整数

）bob.FavoriteAncientWonder);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

鲍勃·史密斯最喜欢的奇迹是奥林匹亚宙斯神像。它的整数值为 2。

```

`enum` 值在内部存储为 `int` 以提高效率。`int` 值会自动从 `0` 开始分配，因此我们 `enum` 中的第三个世界奇迹的值为 `2`。您可以分配未在 `enum` 中列出的 `int` 值。它们将输出为 `int` 值而不是名称，因为找不到匹配项。

## 使用枚举类型存储多个值

对于桶列表，我们可以创建一个 `enum` 实例的数组或集合，集合将在本章后面进行解释，但有一个更好的方法。我们可以使用 `enum` **flags** 将多个选择组合成单个值：

1.  通过使用 `[System.Flags]` 属性修饰 `enum`，并为每个奇迹设置一个代表不同位列的 `byte` 值，如下面的代码中所示的高亮部分所示，修改 `enum`：

```cs

namespace

Packt.Shared

{

**[**

**System.Flags**

**]**

public

enum

古代世界奇迹

**:**

**byte**

{

**None                     =**

**0b**

**_0000_0000,**

**// 即 0**

吉萨大金字塔

**=**

**0b**

**_0000_0001,**

**// 即 1**

巴比伦空中花园

**=**

**0b**

**_0000_0010,**

**// 即 2**

奥林匹亚宙斯神像

**=**

**0b**

**_0000_0100,**

**// 即 4**

以弗所的阿尔忒弥斯神庙

**=**

**0b**

**_0000_1000,**

**// 即 8**

哈里卡纳索斯陵墓

**=**

**0b**

**_0001_0000,**

**// 即 16**

罗得峡谷巨像

**=**

**0b**

**_0010_0000,**

**// 即 32**

亚历山大灯塔

**=**

**0b**

**_0100_0000**

**// 即 64**

}

}

```

我们为每个选择分配了明确的值，这些值在存储在内存中的位时不会重叠。我们还应该使用 `System.Flags` 属性修饰 `enum` 类型，以便当返回值时，它可以自动与多个值匹配，作为逗号分隔的 `string` 返回，而不是返回一个 `int` 值。

通常，`enum` 类型在内部使用 `int` 变量，但由于我们不需要那么大的值，我们可以通过使用 `byte` 变量来减少 75% 的内存需求，即每个值使用 1 个字节而不是 4 个字节。

如果我们想要指示我们的桶列表包括 *巴比伦空中花园* 和 *哈里卡纳索斯陵墓* 这两个古代世界奇迹，那么我们希望 `16` 和 `2` 位设置为 `1`。换句话说，我们将存储值 `18`：

| 64 | 32 | 16 | 8 | 4 | 2 | 1 |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | 0 | 1 | 0 | 0 | 1 | 0 |

1.  在 `Person` 类中，将以下语句添加到字段列表中，如下面的代码所示：

```cs

public

古代世界奇迹 BucketList;

```

1.  在 `Program.cs` 中，添加语句使用 `|` 运算符（按位逻辑或）来组合枚举值来设置桶列表。我们也可以使用将数字 18 转换为枚举类型来设置值，如注释中所示，但我们不应该这样做，因为这会使代码难以理解，如下面的代码所示：

```cs

bob.BucketList =

古代世界奇迹.巴比伦空中花园

| 古代世界奇迹.哈里卡纳索斯陵墓;

// bob.BucketList = (WondersOfTheAncientWorld)18;

WriteLine($"

{bob.Name}

的桶列表是

{bob.BucketList}

"

);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

鲍勃·史密斯的桶列表是巴比伦空中花园，哈里卡纳索斯陵墓

```

**最佳实践**：使用 `enum` 值存储离散选项的组合。如果有多达八个选项，从 `byte` 派生一个 `enum` 类型，如果有多达 16 个选项，从 `ushort` 派生一个 `enum` 类型，如果有多达 32 个选项，从 `uint` 派生一个 `enum` 类型，如果有多达 64 个选项，从 `ulong` 派生一个 `enum` 类型。

# 使用集合存储多个值

现在让我们添加一个字段来存储一个人的孩子。这是聚合的一个例子，因为孩子是与当前人相关的类的实例，但不是人本身的一部分。我们将使用泛型`List<T>`集合类型，它可以存储任何类型的有序集合。您将在*第八章* *使用常见的.NET 类型*中了解更多关于集合的知识。现在，只需跟着做：

1.  在`Person.cs`中，导入`System.Collections.Generic`命名空间，如下面的代码所示：

```cs

使用

System.Collections.Generic; // List<T>

```

1.  在`Person`类中声明一个新字段，如下面的代码所示：

```cs

public

List

<

Person

> Children

= new

List<Person>();

```

`List<Person>`读作"Person 列表"，例如，"名为`Children`的属性的类型是`Person`实例的列表"。我们明确地将类库更改为目标.NET Standard 2.0（使用 C# 7 编译器），因此我们不能使用目标类型的新来初始化`Children`字段。如果我们将其保留为.NET 6.0 目标，那么我们可以使用目标类型的新，如下面的代码所示：

```cs

public

List<Person> Children = new

();

```

我们必须确保集合在我们添加项目之前被初始化为`Person`列表的新实例，否则，该字段将为`null`，当我们尝试使用其成员（如`Add`）时，它将抛出运行时异常。

## 理解泛型集合

`List<T>`类型中的尖括号是 C#中称为**泛型**的功能，它是在 2005 年的 C# 2.0 中引入的。这是一个用于使集合**强类型**的花哨术语，也就是说，编译器明确知道可以存储在集合中的对象类型。泛型提高了代码的性能和正确性。

**强类型**与**静态类型**有不同的含义。旧的`System.Collection`类型是静态类型，包含弱类型的`System.Object`项。新的`System.Collection.Generic`类型是静态类型，包含强类型的`<T>`实例。

具有讽刺意味的是，术语*泛型*意味着我们可以使用更具体的静态类型！

1.  在`Program.cs`中，添加语句为`Bob`添加两个孩子，然后显示他有多少孩子以及他们的名字，如下面的代码所示：

```cs

bob.Children.Add(new

Person { Name = "Alfred"

}); // C# 3.0 及更高版本

bob.Children.Add(new

() { Name = "Zoe"

}); // C# 9.0 及更高版本

WriteLine(

$"

{bob.Name}

has

{bob.Children.Count}

children:"

);

for

(int

childIndex = 0

; childIndex < bob.Children.Count; childIndex++)

{

WriteLine($"

{bob.Children[childIndex].Name}

"

);

}

```

我们还可以使用`foreach`语句来枚举集合。作为额外的挑战，将`for`语句更改为使用`foreach`输出相同的信息。

1.  运行代码并查看结果，如下面的输出所示：

```cs

Bob Smith 有 2 个孩子：

Alfred

Zoe

```

## 使字段静态化

到目前为止，我们创建的字段都是**实例**成员，这意味着对于创建的类的每个实例，每个字段都存在不同的值。`alice`和`bob`变量具有不同的`Name`值。

有时，您希望定义一个字段，该字段只有一个值，该值在所有实例之间共享。

这些被称为**静态** *成员*，因为字段不是唯一可以是静态的成员。让我们看看可以使用`static`字段实现什么：

1.  在`PacktLibrary`项目中，添加一个名为`BankAccount.cs`的新类文件。

1.  修改类以给它三个字段，两个实例字段和一个静态字段，如下面的代码所示：

```cs

命名空间

Packt.Shared

{

public

类

BankAccount

{

public

string

AccountName; // 实例成员

public

decimal

Balance; // 实例成员

public

static

decimal

InterestRate; // 共享成员

}

}

```

`BankAccount`的每个实例都将有自己的`AccountName`和`Balance`值，但所有实例将共享一个`InterestRate`值。

1.  ```cs

string

```cs

```cs

```

```

{bob.Name}

is a

BankAccount gerrierAccount = new

```

"

arg0: gerrierAccount.AccountName,

,

使用构造函数初始化字段

WriteLine($"

在 `Program.cs` 中，添加一个语句将 Bob 的姓名和家乡星球写入控制台，如下面的代码所示：

```cs

**最佳实践**：出于两个重要原因，使用只读字段而不是常量字段是更好的选择：该值可以在运行时计算或加载，并且可以使用任何可执行语句来表示。因此，只读字段可以使用构造函数或字段赋值来设置。对字段的每个引用都是一个活动引用，因此任何将来的更改都将由调用代码正确反映出来。

要获取常量字段的值，必须写类的名称，而不是类的实例的名称。在 `Program.cs` 中，添加一个语句将 Bob 的姓名和物种写入控制台，如下面的代码所示：

BankAccount jonesAccount = new

jonesAccount.Balance = 2400

您还可以声明 `static` `readonly` 字段，其值将在类型的所有实例之间共享。

;

WriteLine(format: "{0} earned {1:C} interest."

Bob Smith is a Homo Sapien

readonly

1.  ;

gerrierAccount.Balance = 98

Microsoft 类型中 `const` 字段的示例包括 `System.Int32.MaxValue` 和 `System.Math.PI`，因为这两个值都永远不会改变，如 *图 5.2* 中所示：

运行代码并查看额外的输出：

Ms. Gerrier earned £1.18 interest.

);

## ```

Species = "Homo Sapien"

1.  使字段只读

,

arg0: jonesAccount.AccountName,

Mrs. Jones earned £28.80 interest.

Bob Smith was born on Earth

arg1: jonesAccount.Balance * BankAccount.InterestRate);

在 `Program.cs` 中，添加语句来设置共享的利率，然后创建两个 `BankAccount` 类型的实例，如下面的代码所示：

**最佳实践**：常量并不总是两个重要原因的最佳选择：该值必须在编译时知道，并且必须能够表示为文字 `string`、`Boolean` 或数字值。对 `const` 字段的每个引用都将在编译时替换为文字值，因此，如果值在将来的版本中更改，而您没有重新编译任何引用它的程序集以获取新值，则不会反映出这种更改。

HomePlanet = "Earth"

1.  WriteLine(format: "{0} earned {1:C} interest."

();

```

使字段成为常量

public

Figure 5.2: Examples of constants

```

```cs

WriteLine($"

1.  BankAccount.InterestRate = 0.012

string

arg1: gerrierAccount.Balance * BankAccount.InterestRate);

```cs

`:C` 是一个格式代码，告诉 .NET 使用货币格式来显示数字。在 *第八章* *使用常见的 .NET 类型* 中，您将学习如何控制决定货币符号的文化。现在，它将使用您操作系统安装的默认值。我住在英国伦敦，因此我的输出显示英镑 (£)。

字段不是唯一可以是静态的成员。构造函数、方法、属性和其他成员也可以是静态的。

M; // store a shared value

public

## ;

```

1.  (); // C# 9.0 and later

;

;

通常，对于不应更改的字段，更好的选择是将它们标记为只读：

```cs

![图形用户界面、文本、应用程序、电子邮件描述自动生成](img/Image00065.jpg)

{bob.Name}

);

如果字段的值永远不会改变，您可以使用 `const` 关键字并在编译时分配一个文字值：

1.  {Person.Species}

```

// 只读字段

在 `Person.cs` 中，添加一个语句来声明一个实例只读字段，用于存储一个人的家乡星球，如下面的代码所示：

出生在

运行代码并查看结果，如下面的输出所示：

jonesAccount.AccountName = "Mrs. Jones"

```

运行代码并查看结果，如下面的输出所示：

1.  gerrierAccount.AccountName = "Ms. Gerrier"

;

// 常量

在 `Person.cs` 中，添加以下代码：

const

```cs

## {bob.HomePlanet}

字段通常需要在运行时初始化。您可以在构造函数中执行此操作，当您使用 `new` 关键字创建类的实例时，将调用构造函数。构造函数在任何字段由使用类型的代码设置之前执行。

1.  在 `Person.cs` 中，在现有的只读 `HomePlanet` 字段之后添加语句，以定义第二个只读字段，然后在构造函数中设置 `Name` 和 `Instantiated` 字段，如下所示：

```cs

// 只读字段

public

readonly

string

HomePlanet = "Earth"

;

**public**

**只读**

**DateTime Instantiated;**

**// 构造函数**

**public**

**Person**

**()**

**{**

**// 为字段设置默认值**

**// 包括只读字段**

**Name =**

**"Unknown"**

**;**

**Instantiated = DateTime.Now;**

**}**

```

1.  在 `Program.cs` 中，添加语句来实例化一个新的人，然后输出其初始字段值，如下所示：

```cs

Person blankPerson = new

();

WriteLine(format:

"{0} of {1} was created at {2:hh:mm:ss} on a {2:dddd}."

，

arg0: blankPerson.Name,

arg1: blankPerson.HomePlanet,

arg2: blankPerson.Instantiated);

```

1.  运行代码并查看结果，如下所示：

```cs

Unknown of Earth was created at 11:58:12 on a Sunday

```

### 定义多个构造函数

类型中可以有多个构造函数。这对鼓励开发人员为字段设置初始值特别有用：

1.  在 `Person.cs` 中，添加语句来定义允许开发人员为人的名称和家乡设置初始值的第二个构造函数，如下所示：

```cs

public

Person

(

string

initialName,

string

homePlanet

)

{

Name = initialName;

HomePlanet = homePlanet;

Instantiated = DateTime.Now;

}

```

1.  在 `Program.cs` 中，添加语句来使用带有两个参数的构造函数创建另一个人，如下所示：

```cs

Person gunny = new

(initialName: "Gunny"

, homePlanet: "Mars"

);

WriteLine(format:

"{0} of {1} was created at {2:hh:mm:ss} on a {2:dddd}."

,

arg0: gunny.Name,

arg1: gunny.HomePlanet,

arg2: gunny.Instantiated);

```

1.  运行代码并查看结果：

```cs

Gunny of Mars was created at 11:59:25 on a Sunday

```

构造函数是方法的一种特殊类别。让我们更详细地看一下方法。

# 编写和调用方法

**方法** 是执行一系列语句的类型的成员。它们是属于类型的函数。

## 从方法返回值

方法可以返回单个值，也可以不返回任何内容：

+   执行一些操作但不返回值的方法在方法名称之前用 `void` 类型表示。

+   执行一些操作并返回值的方法在方法名称之前用返回值的类型表示。

例如，在下一个任务中，您将创建两个方法：

+   `WriteToConsole`：这将执行一个操作（向控制台写入一些文本），但不会从方法中返回任何内容，由 `void` 关键字表示。

+   `GetOrigin`：这将返回一个文本值，由 `string` 关键字表示。

让我们编写代码：

1.  在 `Person.cs` 中，添加语句来定义我之前描述的两个方法，如下所示：

```cs

// 方法

public

void

WriteToConsole

()

{

WriteLine($"

{Name}

was born on a

{DateOfBirth:dddd}

."

);

}

public

string

GetOrigin

()

{

return

$"

{Name}

was born on

{HomePlanet}

。"

;

}

```

1.  在 `Program.cs` 中，添加语句来调用两个方法，如下所示：

```cs

bob.WriteToConsole();

WriteLine(bob.GetOrigin());

```

1.  运行代码并查看结果，如下所示：

```cs

Bob Smith was born on a Wednesday.

Bob Smith was born on Earth.

```

## 使用元组组合多个返回值

每个方法只能返回一个具有单一类型的值。该类型可以是简单类型，例如上一个示例中的 `string`，也可以是复杂类型，例如 `Person`，或者是集合类型，例如 `List<Person>`。

假设我们想要定义一个名为`GetTheData`的方法，该方法需要返回一个`string`值和一个`int`值。我们可以定义一个名为`TextAndNumber`的新类，其中包含一个`string`字段和一个`int`字段，并返回该复杂类型的实例，如下面的代码所示：

```cs

公共

类

文本和数字

{

公共

字符串

文本;

公共

int

数字;

}

公共

类

生活宇宙和一切

{

公共

TextAndNumber

GetTheData

（）

{

返回

新

TextAndNumber

{

Text=“生活的意义是什么？”

，

数字= 42

};

}

}

```

但是，仅仅为了将两个值组合在一起而定义一个类是不必要的，因为在现代版本的 C#中，我们可以使用**元组**。元组是将两个或更多值组合成单个单元的有效方法。我把它们发音为 tuh-ples，但我听说其他开发人员把它们发音为 too-ples。我猜是西红柿，土豆，土豆，土豆，我猜。

元组自从第一个版本以来一直是一些语言的一部分，比如 F#，但是.NET 直到 2010 年才使用`System.Tuple`类型添加了对它们的支持。

### 元组的语言支持

直到 2017 年的 C# 7.0，C#才使用括号字符`（）`添加了元组的语言语法支持，同时.NET 添加了一个新的`System.ValueTuple`类型，该类型在某些常见情况下比旧的.NET 4.0`System.Tuple`类型更有效。 C#元组语法使用更有效的语法。

让我们探索元组：

1.  在`Person.cs`中，添加语句来定义一个返回组合`string`和`int`的元组的方法，如下面的代码所示：

```cs

公共

（字符串

，int

）获取水果（）

{

返回

（“苹果”

，5

）;

}

```

1.  在`Program.cs`中，添加语句调用`GetFruit`方法，然后自动输出元组的字段，名称为`Item1`和`Item2`，如下面的代码所示：

```cs

（字符串

，int

）水果=鲍勃。获取水果（）;

WriteLine（$“

{fruit.Item1}

，

{fruit.Item2}

有。

）;

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

苹果，5 个。

```

### 给元组的字段命名

要访问元组的字段，默认名称为`Item1`，`Item2`等。

您还可以明确指定字段名称：

1.  在`Person.cs`中，添加语句来定义一个返回具有命名字段的元组的方法，如下面的代码所示：

```cs

公共

（字符串

名称，int

数字）GetNamedFruit（）

{```

返回

（名称：“苹果”

，数字：5

）;

}

```

1.  在`Program.cs`中，添加语句来调用该方法并输出元组的命名字段，如下面的代码所示：

```cs

变量

fruitNamed= bob.GetNamedFruit（）;

WriteLine（$“有

{fruitNamed.Number}

{fruitNamed.Name}

。”

）;

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

有 5 个苹果。

```

### 推断元组名称

如果您正在从另一个对象构造元组，则可以使用 C# 7.1 中引入的名为**元组名称推断**的功能。

在`Program.cs`中，创建两个元组，每个元组由一个`string`和`int`值组成，如下面的代码所示：

```cs

变量

thing1=（“内维尔”

，4

）;

WriteLine（$“

{thing1.Item1}

有

{thing1.Item2}

孩子们。”

）;

变量

thing2=（bob.Name，bob.Children.Count）;

WriteLine（$“

{thing2.Name}

有

{thing2.Count}

孩子们。”

）;

```

在 C# 7.0 中，两者都将使用`Item1`和`Item2`命名方案。在 C# 7.1 及更高版本中，`thing2`可以推断出名称`Name`和`Count`。

### 解构元组

您还可以将元组解构为单独的变量。解构声明具有与命名字段元组相同的语法，但没有元组的命名变量，如下面的代码所示：

```cs

//将返回值存储在具有两个字段的元组变量中

（字符串

TheName，int

TheNumber）tupleWithNamedFields= bob.GetNamedFruit（）;

// tupleWithNamedFields.TheName

// tupleWithNamedFields.TheNumber

//解构返回值为两个单独的变量

（字符串

名称，int

数字）= GetNamedFruit（）;

//名称

//数字

```

This has the effect of splitting the tuple into its parts and assigning those parts to new variables.

1.  In `Program.cs` , add statements to deconstruct the tuple returned from the `GetFruit` method, as shown in the following code:

```cs

(string

fruitName, int

fruitNumber) = bob.GetFruit();

WriteLine($"Deconstructed:

{fruitName}

,

{fruitNumber}

"

);

```

1.  Run the code and view the result, as shown in the following output:

```cs

Deconstructed: Apples, 5

```

### Deconstructing types

Tuples are not the only type that can be deconstructed. Any type can have special methods named `Deconstruct` that break down the object into parts. Let's implement some for the `Person` class:

1.  In `Person.cs` , add two `Deconstruct` methods with `out` parameters defined for the parts we want to deconstruct into, as shown in the following code:

```cs

// deconstructors

public

void

Deconstruct

(

out

string

name,

out

DateTime dob

)

{

name = Name;

dob = DateOfBirth;

}

public

void

Deconstruct

(

out

string

name,

out

DateTime dob,

out

WondersOfTheAncientWorld fav

)

{

name = Name;

dob = DateOfBirth;

fav = FavoriteAncientWonder;

}

```

1.  In `Program.cs` , add statements to deconstruct `bob` , as shown in the following code:

```cs

// Deconstructing a Person

var

(name1, dob1) = bob;

WriteLine($"Deconstructed:

{name1}

,

{dob1}

"

);

var

(name2, dob2, fav2) = bob;

WriteLine($"Deconstructed:

{name2}

,

{dob2}

,

{fav2}

"

);

```

1.  Run the code and view the result, as shown in the following output:

```cs

Deconstructed: Bob Smith, 22/12/1965 00:00:00

Deconstructed: Bob Smith, 22/12/1965 00:00:00, StatueOfZeusAtOlympia

B

```

## Defining and passing parameters to methods

Methods can have parameters passed to them to change their behavior. Parameters are defined a bit like variable declarations but inside the parentheses of the method, as you saw earlier in this chapter with constructors. Let's see more examples:

1.  In `Person.cs` , add statements to define two methods, the first without parameters and the second with one parameter, as shown in the following code:

```cs

public

string

SayHello

()

{

return

$"

{Name}

says 'Hello!'"

;

}

public

string

SayHelloTo

(

string

name

)

{

return

$"

{Name}

says 'Hello

{name}

!'"

;

}

```

1.  In `Program.cs` , add statements to call the two methods and write the return value to the console, as shown in the following code:

```cs

WriteLine(bob.SayHello());

WriteLine(bob.SayHelloTo("Emily"

));

```

1.  Run the code and view the result:

```cs

Bob Smith says 'Hello!'

Bob Smith says 'Hello Emily!'

```

When typing a statement that calls a method, IntelliSense shows a tooltip with the name and type of any parameters, and the return type of the method, as shown in *Figure 5.3* :

![Graphical user interface, text, website Description automatically generated](img/Image00066.jpg)

Figure 5.3: An IntelliSense tooltip for a method with no overloads

## Overloading methods

Instead of having two different method names, we could give both methods the same name. This is allowed because the methods each have a different signature.

A **method signature** is a list of parameter types that can be passed when calling the method. Overloaded methods cannot differ only in the return type.

1.  In `Person.cs` , change the name of the `SayHelloTo` method to `SayHello` .

1.  In `Program.cs` , change the method call to use the `SayHello` method, and note that the quick info for the method tells you that it has one additional overload, 1/2, as well as 2/2, as shown in *Figure 5.4* :![Graphical user interface Description automatically generated with medium confidence](img/Image00067.jpg)

Figure 5.4: An IntelliSense tooltip for an overloaded method

**Good Practice** : Use overloaded methods to simplify your class by making it appear to have fewer methods.

## Passing optional and named parameters

Another way to simplify methods is to make parameters optional. You make a parameter optional by assigning a default value inside the method parameter list. Optional parameters must always come last in the list of parameters.

我们现在将创建一个具有三个可选参数的方法：

1.  在 `Person.cs` 中，添加语句来定义方法，如下面的代码所示：

```cs

公共

串

OptionalParameters

(

串

命令  =

"运行！"

，

双

数字 =

0.0

，

布尔

活动 =

真

)

{

返回

串

.Format(

格式："命令是{0}，数字是{1}，活动是{2}"

，

arg0：命令，

arg1：数字，

arg2：活动）;

}

```

1.  在 `Program.cs` 中，添加一条语句来调用该方法并将其返回值写入控制台，如下面的代码所示：

```cs

WriteLine(bob.OptionalParameters());

```

1.  当您键入代码时，IntelliSense 会出现。您将看到一个工具提示，显示三个可选参数及其默认值，如*图 5.5*所示：![图形用户界面，文本，应用程序，聊天或文本消息自动生成的描述](img/Image00068.jpg)

图 5.5：IntelliSense 在键入代码时显示可选参数

1.  运行代码并查看结果，如下面的输出所示：

```cs

命令是运行！，数字是 0，活动是真的

```

1.  在 `Program.cs` 中，添加一条语句，传递 `command` 参数的 `string` 值和 `number` 参数的 `double` 值，如下面的代码所示：

```cs

WriteLine(bob.OptionalParameters("跳！"

， 98.5

));

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

命令是跳！，数字是 98.5，活动是真的

```

`command` 和 `number` 参数的默认值已被替换，但 `active` 的默认值仍然是 `true`。

### 调用方法时命名参数值

在调用方法时，可选参数通常与命名参数结合使用，因为命名参数允许以与声明顺序不同的顺序传递值。

1.  在 `Program.cs` 中，添加一条语句，传递 `command` 参数的 `string` 值和 `number` 参数的 `double` 值，但使用命名参数，以便可以交换它们通过的顺序，如下面的代码所示：

```cs

WriteLine(bob.OptionalParameters(

数字：52.7

，命令："隐藏！"

));

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

命令是隐藏！，数字是 52.7，活动是真的

```

甚至可以使用命名参数跳过可选参数。

1.  在 `Program.cs` 中，添加一条语句，使用位置顺序传递 `command` 参数的 `string` 值，跳过 `number` 参数，并使用命名的 `active` 参数，如下面的代码所示：

```cs

WriteLine(bob.OptionalParameters("戳！"

，活动：假

));

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

命令是戳！，数字是 0，活动是假的

```

## 控制参数传递方式

当参数传递到方法中时，可以以三种方式之一传递：

+   按**值**（这是默认值）：把这些看作是*仅输入*。

+   按**引用**作为 `ref` 参数：把这些看作是*输入和输出*。

+   作为 `out` 参数：把这些看作是*仅输出*。

让我们看一些传递参数的示例：

1.  在 `Person.cs` 中，添加语句来定义一个具有三个参数的方法，一个 `in` 参数，一个 `ref` 参数和一个 `out` 参数，如下面的方法所示：

```cs

公共

void

PassingParameters

(

int

x，

ref

int

y，

out

int

z

)

{

// out 参数不能有默认值

// 必须在方法内初始化

z = 99

;

// 递增每个参数

x++;

y++;

z++;

}

```

1.  在 `Program.cs` 中，添加语句来声明一些 `int` 变量并将它们传递到方法中，如下面的代码所示：

```cs

int

a = 10

;

int

b = 20

;

int

c = 30

;

WriteLine($"之前：a =

{a}

， b =

{b}

， c =

{c}

"

);

bob.PassingParameters(a, ref

b，out

c）;

WriteLine($"之后：a =

{a}

, b =

{b}

， c =

{c}

"

）;

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

之前：a = 10，b = 20，c = 30

之后：a = 10，b = 21，c = 100

```

+   默认情况下，通过参数传递变量时，传递的是其当前值，而不是变量本身。因此，`x`具有`a`变量的值的副本。`a`变量保留其原始值`10`。

+   当将变量作为`ref`参数传递时，会传递对变量的引用到方法中。因此，`y`是对`b`的引用。当`y`参数增加时，`b`变量也会增加。

+   将变量作为`out`参数传递时，会传递对变量的引用到方法中。因此，`z`是对`c`的引用。`c`变量的值将被方法内部执行的任何代码替换。我们可以简化`Main`方法中的代码，不给`c`变量赋值`30`，因为它总是会被替换。

### 简化的 out 参数

在 C# 7.0 及更高版本中，我们可以简化使用 out 变量的代码。

在`Program.cs`中，添加语句来声明一些更多的变量，包括内联声明的`out`参数`f`，如下面的代码所示：

```cs

int

d = 10

;

int

e = 20

；

WriteLine($"之前：d =

{d}

，e =

{e}

，f 尚不存在！"

);

//简化的 C# 7.0 或更高版本的 out 参数语法

bob.PassingParameters(d, ref

e, out

int

f);

WriteLine($"之后：d =

{d}

，e =

{e}

, f =

{f}

"

);

```

## 理解 ref 返回

在 C# 7.0 或更高版本中，`ref`关键字不仅用于将参数传递给方法；它还可以应用于`return`值。这允许外部变量引用内部变量，并在方法调用后修改其值。这在高级场景中可能很有用，例如将占位符传递到大型数据结构中，但这超出了本书的范围。

## 使用 partial 拆分类

在处理具有多个团队成员的大型项目时，或者在处理特别大型和复杂的类实现时，能够将类的定义分割到多个文件中是很有用的。您可以使用`partial`关键字来实现这一点。

假设我们想要向`Person`类添加语句，这些语句是由像对象关系映射器这样的工具自动生成的，该工具从数据库中读取模式信息。如果类被定义为`partial`，那么我们可以将类分成自动生成的代码文件和手动编辑的代码文件。

让我们编写一些模拟这个例子的代码：

1.  在`Person.cs`中，添加`partial`关键字，如下面的代码中所示：

```cs

命名空间

Packt.Shared

{

公共

**partial**

类

人

{

```

1.  在`PacktLibrary`项目/文件夹中，添加一个名为`PersonAutoGen.cs`的新类文件。

1.  添加语句到新文件，如下面的代码所示：

```cs

命名空间

Packt.Shared

{

公共

部分

类

人

{

}

}

```

我们为本章编写的其余代码将在`PersonAutoGen.cs`文件中编写。

# 使用属性和索引器控制访问

之前，您创建了一个名为`GetOrigin`的方法，该方法返回一个包含人的姓名和出生地的`string`。像 Java 这样的语言经常这样做。C#有更好的方法：属性。

属性只是一个方法（或一对方法），当您想要获取或设置一个值时，它的行为和外观就像一个字段，从而简化了语法。

## 定义只读属性

`readonly`属性只有一个`get`实现。

1.  在`PersonAutoGen.cs`中，在`Person`类中，添加语句来定义三个属性：

1.  第一个属性将使用适用于所有 C#版本的属性语法执行与`GetOrigin`方法相同的角色（尽管它使用了 C# 6 及更高版本的字符串插值语法）。

1.  第二个属性将使用 C# 6 及更高版本的 lambda 表达式体`=>`语法返回问候消息。

1.  第三个属性将计算人的年龄。

这是代码：

```cs

//使用 C# 1-5 语法定义的属性

公共

字符串

出身地

{

get

{

返回

$"

{Name}

出生于

{HomePlanet}

"

;

}

}

// 使用 C# 6+ lambda 表达式体语法定义了两个属性

public

string

Greeting => $"

{Name}

says 'Hello!'"

;

public

int

Age => System.DateTime.Today.Year - DateOfBirth.Year;

```

**Good Practice**：这不是计算某人年龄的最佳方法，但我们并不是在学习如何从出生日期计算年龄。如果您需要正确地这样做，请阅读以下链接的讨论：[`stackoverflow.com/questions/9/how-do-i-calculate-someones-age-in-c`](https://stackoverflow.com/questions/9/how-do-i-calculate-someones-age-in-c)

1.  在`Program.cs`中，添加语句来获取属性，如下面的代码所示：

```cs

Person sam = new

()

{

Name = "Sam"

,

DateOfBirth = new

(1972

, 1

, 27

)

};

WriteLine(sam.Origin);

WriteLine(sam.Greeting);

WriteLine(sam.Age);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

Sam was born on Earth

Sam says 'Hello!'

49

```

输出显示 49，因为我在 2021 年 8 月 15 日运行了控制台应用程序，当时 Sam 已经 49 岁了。

## 定义可设置的属性

要创建一个可设置的属性，您必须使用较旧的语法，并提供一对方法——不仅仅是`get`部分，还有一个`set`部分：

1.  在`PersonAutoGen.cs`中，添加语句来定义一个具有`get`和`set`方法（也称为 getter 和 setter）的`string`属性，如下面的代码所示：

```cs

public

string

FavoriteIceCream { get

; set

; } // auto-syntax

```

虽然您没有手动创建一个字段来存储人的最喜欢的冰淇淋，但它是自动由编译器为您创建的。

有时，当属性被设置时，您需要更多的控制。在这种情况下，您必须使用更详细的语法，并手动创建一个`private`字段来存储属性的值。

1.  在`PersonAutoGen.cs`中，添加语句来定义一个具有`get`和`set`的`string`字段和`string`属性，如下面的代码所示：

```cs

private

string

favoritePrimaryColor;

public

string

FavoritePrimaryColor

{

get

{

return

favoritePrimaryColor;

}

set

{

switch

(value

.ToLower())

{

case

"red"

:

case

"green"

:

case

"blue"

:

favoritePrimaryColor = value

;

break

;

default

:

throw

new

System.ArgumentException(```

$"

{

value

}

is not a primary color. "

+

"Choose from: red, green, blue."

);

}

}

}

```

**Good Practice**：避免向您的 getter 和 setter 添加太多代码。这可能表明您的设计存在问题。考虑添加私有方法，然后在 setter 和 getter 中调用它们，以简化您的实现。

1.  在`Program.cs`中，添加语句来设置 Sam 最喜欢的冰淇淋和颜色，然后将它们写出来，如下面的代码所示：

```cs

sam.FavoriteIceCream = "Chocolate Fudge"

;

WriteLine($"Sam's favorite ice-cream flavor is

{sam.FavoriteIceCream}

."

);

sam.FavoritePrimaryColor = "Red"

;

WriteLine($"Sam's favorite primary color is

{sam.FavoritePrimaryColor}

."

);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

Sam's favorite ice-cream flavor is Chocolate Fudge.

Sam's favorite primary color is Red.

```

如果您尝试将颜色设置为除红色、绿色或蓝色之外的任何值，那么代码将抛出异常。调用代码可以使用`try`语句来显示错误消息。

**Good Practice**：当您想要验证可以存储什么值时，请使用属性而不是字段，当您想要在 XAML 中进行数据绑定时，请使用属性，我们将在*第十九章*中介绍，*使用.NET MAUI 构建移动和桌面应用程序*（可在[`github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf`](https://github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf)找到），以及当您想要读取和写入字段而不使用`GetAge`和`SetAge`这样的方法对时。

## 要求在实例化期间设置属性

C# 10 引入了`required`修饰符。如果您在属性上使用它，编译器将确保在实例化时将属性设置为一个值，如下面的代码所示：

```cs

public

class

Book

{sam[

public

required string

Isbn { get

; set

; }

public

string

Title { get

; set

; }

}

```

如果您尝试实例化`Book`而没有设置`Isbn`属性，您将看到编译器错误，如下面的代码所示：

```cs

Book novel = new

();

```

`required`关键字可能不会出现在.NET 6 的最终版本中，因此将此部分视为理论性的。

## 定义索引器

索引器允许调用代码使用数组语法来访问属性。例如，`string`类型定义了一个**索引器**，以便调用代码可以访问`string`中的单个字符。

我们将定义一个索引器，以简化对人的子代的访问：

1.  在`PersonAutoGen.cs`中，添加语句来定义一个索引器，以使用子代的索引，如下面的代码所示：

```cs

// 索引器

public

Person this

[int

index]

{

get

{

return

Children[index]; // 传递给 List<T>索引器

}

set

{

Children[index] = value

;

}

}

```

您可以重载索引器，以便不同类型可以用于它们的参数。例如，除了传递`int`值外，您还可以传递`string`值。

1.  在`Program.cs`中，添加语句来向`Sam`添加两个子代，然后使用更长的`Children`字段和更短的索引器语法访问第一个和第二个子代，如下面的代码所示：

```cs

sam.Children.Add(new

() { Name = "Charlie"

});

sam.Children.Add(new

() { Name = "Ella"

});

WriteLine($"Sam's first child is

{sam.Children[

0

].Name}

"

);

WriteLine($"Sam's second child is

{sam.Children[

1

].Name}

"

);

WriteLine($"Sam's first child is

{sam[

0

].Name}

"

);

WriteLine($"Sam's second child is

{sam[

1

].Name}

"

);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

Sam's first child is Charlie

Sam's second child is Ella

Sam's first child is Charlie

Sam's second child is Ella

```

# Pattern matching with objects

在*第三章*，*控制流、类型转换和异常处理*中，您已经了解了基本的模式匹配。在本节中，我们将更详细地探讨模式匹配。

## 创建和引用.NET 6 类库

增强的模式匹配功能仅适用于支持 C# 9 或更高版本的现代.NET 类库。

1.  使用您喜欢的编码工具向工作区/解决方案中添加一个名为`Chapter05`的新类库`PacktLibraryModern`。

1.  在`PeopleApp`项目中，添加对`PacktLibraryModern`类库的引用，如下标记所示：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

<OutputType>Exe</OutputType>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<ImplicitUsings>enable</ImplicitUsings>

</PropertyGroup>

<ItemGroup>

<ProjectReference Include="../PacktLibrary/PacktLibrary.csproj"

/>

**<ProjectReference**

**Include=**

**"../PacktLibraryModern/PacktLibraryModern.csproj"**

**/>**

</ItemGroup>

</Project>

```

1.  构建`PeopleApp`项目。

## 定义航班乘客

在这个例子中，我们将定义一些代表飞行中各种类型乘客的类，然后我们将使用模式匹配的 switch 表达式来确定他们航班的费用。

1.  在`PacktLibraryModern`项目/文件夹中，将文件`Class1.cs`重命名为`FlightPatterns.cs`。

1.  在`FlightPatterns.cs`中，添加语句来定义三种不同属性的乘客类型，如下面的代码所示：

```cs

namespace

Packt.Shared

; // C# 10 文件范围的命名空间

public

class

BusinessClassPassenger

{

public

override

string

ToString

()

{

return

$"Business Class"

;

}

}

public

class

FirstClassPassenger

{

public

int

AirMiles { get

; set

; }

public

override

string

ToString

()

{

return

$"First Class with

{AirMiles:N0}

air miles"

;

}

}

public

类

CoachClassPassenger

{

public

double

CarryOnKG { get

; set

; }

public

override

string

ToString

()

{

return

$"教练舱，带有

{CarryOnKG:N2}

公斤随身携带"

;

}

}

```

1.  在`Program.cs`中，添加语句来定义一个包含五个不同类型和属性值的乘客的对象数组，然后枚举它们，输出他们的飞行费用，如下面的代码所示：

```cs

object

[] 乘客 = {

new

FirstClassPassenger { AirMiles = 1

_419 },

new

FirstClassPassenger { AirMiles = 16

_562 },

new

BusinessClassPassenger(),

new

CoachClassPassenger { CarryOnKG = 25.7

},

new

CoachClassPassenger { CarryOnKG = 0

},

};

foreach

(object

passenger in

passengers)

{

decimal

flightCost = passenger switch

{

FirstClassPassenger p when

p.AirMiles > 35000

=> 1500

M,

FirstClassPassenger p when

p.AirMiles > 15000

=> 1750

M,

FirstClassPassenger _                         => 2000

M,

BusinessClassPassenger _                      => 1000

M,

CoachClassPassenger p when

p.CarryOnKG < 10.0

=> 500

M,

CoachClassPassenger _                         => 650

M,

_                                             => 800

M

};

WriteLine($"飞行费用

{flightCost:C}

for

{passenger}

"

);

}

```

在审查前面的代码时，请注意以下内容：

+   要对对象的属性进行模式匹配，必须命名一个局部变量，然后可以在表达式中使用`p`。

+   要仅对类型进行模式匹配，可以使用`_`来丢弃局部变量。

+   switch 表达式还使用`_`来表示其默认分支。

1.  运行代码并查看结果，如下面的输出所示：

```cs

飞行费用为£2,000.00，头等舱带有 1,419 英里

飞行费用为£1,750.00，头等舱带有 16,562 英里

飞行费用为£1,000.00，商务舱

飞行费用为£650.00，教练舱带有 25.70 公斤的随身携带

飞行费用为£500.00，教练舱带有 0.00 公斤的随身携带

```

## C# 9 或更高版本中的模式匹配增强

之前的示例适用于 C# 8。现在我们将看一些 C# 9 及更高版本的增强功能。首先，当进行类型匹配时，您不再需要使用下划线来丢弃：

1.  在`Program.cs`中，注释掉 C# 8 的语法，并添加 C# 9 及更高版本的语法，以修改头等舱乘客的分支，使用嵌套的 switch 表达式和新的条件支持，如下面的代码所示：

```cs

decimal

flightCost = passenger switch

{

/* C# 8 语法

FirstClassPassenger p when p.AirMiles > 35000 => 1500M,

FirstClassPassenger p when p.AirMiles > 15000 => 1750M,

FirstClassPassenger                           => 2000M, */

// C# 9 或更高版本的语法

FirstClassPassenger p => p.AirMiles switch

{

> 35000

=> 1500

M,

> 15000

=> 1750

M,

_       => 2000

M

},

BusinessClassPassenger                        => 1000

M,

CoachClassPassenger p when

p.CarryOnKG < 10.0

=> 500

M，

CoachClassPassenger                           => 650

M,

_                                             => 800

M

};

```

1.  运行代码以查看结果，并注意它们与以前相同。

您还可以使用关系模式与属性模式结合，以避免嵌套的 switch 表达式，如下面的代码所示：

```cs

FirstClassPassenger { AirMiles: > 35000

} => 1500

,

FirstClassPassenger { AirMiles: > 15000

} => 1750

M,

FirstClassPassenger => 2000

M,

```

# 处理记录

在我们深入研究 C# 9 及更高版本的新记录语言功能之前，让我们看一些其他相关的新功能。

## 仅初始化属性

您在本章中始终使用对象初始化语法来实例化对象并设置初始属性。这些属性也可以在实例化后更改。

有时您希望将属性视为`readonly`字段，以便在实例化期间进行设置，但在之后不进行设置。新的`init`关键字使这成为可能。它可以用来代替`set`关键字：

1.  在`PacktLibraryModern`项目/文件夹中，添加一个名为`Records.cs`的新文件。

1.  In `Records.cs` , define an immutable person class, as shown in the following code:

```cs

namespace

Packt.Shared

; // C# 10 file-scoped namespace

public

class

ImmutablePerson

{

public

string

? FirstName { get

; init

; }

public

string

? LastName { get

; init

; }

}

```

1.  In `Program.cs` , add statements to instantiate a new immutable person and then try to change one of its properties, as shown in the following code:

```cs

ImmutablePerson jeff = new

()

{

FirstName = "Jeff"

,

LastName = "Winger"

};

jeff.FirstName = "Geoff"

;

```

1.  Compile the console app and note the compile error, as shown in the following output:

```cs

Program.cs(254,7): error CS8852: Init-only property or indexer 'ImmutablePerson.FirstName' can only be assigned in an object initializer, or on 'this' or 'base' in an instance constructor or an 'init' accessor. [/Users/markjprice/Code/Chapter05/PeopleApp/PeopleApp.csproj]

```

1.  Comment out the attempt to set the `FirstName` property after instantiation.

## Understanding records

Init-only properties provide some immutability to C#. You can take the concept further by using **records** . These are defined by using the `record` keyword instead of the `class` keyword. That can make the whole object immutable, and it acts like a value when compared. We will discuss equality and comparisons of classes, records, and value types in more detail in *Chapter 6* , *Implementing Interfaces and Inheriting Classes* .

Records should not have any state (properties and fields) that changes after instantiation. Instead, the idea is that you create new records from existing ones with any changed state. This is called non-destructive mutation. To do this, C# 9 introduced the `with` keyword:

1.  In `Records.cs` , add a record named `ImmutableVehicle` , as shown in the following code:

```cs

公共

record

ImmutableVehicle

{

public

int

Wheels { get

; init

; }

public

string

? Color { get

; init

; }

public

string

? Brand { get

; init

; }

}

```

1.  In `Program.cs` , add statements to create a `car` and then a mutated copy of it, as shown in the following code:

```cs

ImmutableVehicle car = new

()

{

Brand = "Mazda MX-5 RF"

,

Color = "Soul Red Crystal Metallic"

,

Wheels = 4

};

ImmutableVehicle repaintedCar = car

with

{ Color = "Polymetal Grey Metallic"

};

WriteLine($"Original car color was

{car.Color}

."

);

WriteLine($"New car color is

{repaintedCar.Color}

."

);

```

1.  Run the code to view the results, and note the change to the car color in the mutated copy, as shown in the following output:

```cs

Original car color was Soul Red Crystal Metallic.

New car color is Polymetal Grey Metallic.

```

## Positional data members in records

The syntax for defining a record can be greatly simplified using positional data members.

### Simplifying data members in records

Instead of using object initialization syntax with curly braces, sometimes you might prefer to provide a constructor with positional parameters as you saw earlier in this chapter. You can also combine this with a deconstructor for splitting the object into individual parts, as shown in the following code:

```cs

public

record

ImmutableAnimal

{

public

string

Name { get

; init

; }

public

string

Species { get

; init

; }

public

ImmutableAnimal

(

string

name,

string

species

)

{

Name = name;

Species = species;

}

public

void

Deconstruct

(

out

string

name,

out

string

species

)

{

name = Name;

species = Species;

}

}

```

The properties, constructor, and deconstructor can be generated for you:

1.  In `Records.cs` , add statements to define another record using simplified syntax known as positional records, as shown in the following code:

```cs

// simpler way to define a record

// auto-generates the properties, constructor, and deconstructor

public

record

ImmutableAnimal

(

string

Name,

string

Species

)

;

```

1.  In `Program.cs` , add statements to construct and deconstruct immutable animals, as shown in the following code:

```cs

ImmutableAnimal oscar = new

("Oscar"

, "Labrador"

);

var

(who, what) = oscar; // calls Deconstruct method

WriteLine($"

{谁}

是一个

{什么}

."

);

```

1.  运行应用程序并查看结果，如下面的输出所示：

```cs

奥斯卡是一只拉布拉多犬。

```

当我们在 *第六章* *实现接口和继承类* 中查看 C# 10 支持创建 `struct` 记录时，您将再次看到记录。

# 练习和探索

通过回答一些问题来测试您的知识和理解，进行一些实践，并深入研究本章的主题。

## 练习 5.1 – 测试你的知识

回答以下问题：

1.  有六种访问修饰符关键字的组合，它们分别是什么？

1.  `static` 、 `const` 和 `readonly` 关键字在应用于类型成员时有什么区别？

1.  构造函数是做什么的？

1.  当您想要存储组合值时，为什么应该将 `[Flags]` 属性应用于 `enum` 类型？

1.  为什么 `partial` 关键字有用？

1.  什么是元组？

1.  `record` 关键字是做什么的？

1.  重载是什么意思？

1.  字段和属性之间有什么区别？

1.  如何使方法参数可选？

## 练习 5.2 – 探索主题

使用以下页面上的链接，了解本章涵盖的主题的更多细节：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-5---building-your-own-types-with-object-oriented-programming`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-5---building-your-own-types-with-object-oriented-programming)

# 总结

在本章中，您学习了使用面向对象编程制作自己的类型。您了解了类型可以具有的不同成员类别，包括用于存储数据的字段和用于执行操作的方法，并且您使用了面向对象编程的概念，如聚合和封装。您看到了如何使用现代 C# 功能，如关系和属性模式匹配增强、仅初始化属性和记录。

在下一章中，您将进一步了解这些概念，包括定义委托和事件、实现接口以及继承现有类。
