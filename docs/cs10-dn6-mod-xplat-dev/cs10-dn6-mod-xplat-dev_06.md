# 第六章：实现接口和继承类

本章介绍了使用**面向对象编程**（**OOP**）从现有类型派生新类型。您将学习定义运算符和执行简单操作的本地函数，以及在类型之间交换消息的委托和事件。您将为常见功能实现接口。您将学习泛型和引用类型与值类型之间的区别。您将创建一个派生类，以从基类继承功能，重写继承的类型成员，并使用多态。最后，您将学习如何创建扩展方法以及如何在继承层次结构中进行类之间的转换。

本章涵盖以下主题：

+   设置类库和控制台应用程序

+   更多关于方法

+   引发和处理事件

+   使用泛型使类型安全地可重用

+   实现接口

+   使用引用和值类型管理内存

+   处理空值

+   从类继承

+   在继承层次结构中进行转换

+   继承和扩展.NET 类型

+   使用分析器编写更好的代码

# 设置类库和控制台应用程序

我们将首先定义一个包含两个项目的工作空间/解决方案，就像在*第五章*，*使用面向对象编程构建自己的类型*中创建的那样。即使您在该章节中完成了所有的练习，也请按照以下说明操作，因为我们将在类库中使用 C# 10 功能，因此它需要针对.NET 6.0 而不是.NET Standard 2.0 进行目标设置：

1.  使用您喜欢的编码工具创建一个名为`Chapter06`的新工作空间/解决方案。

1.  添加一个类库项目，如下列表所定义的那样：

1.  项目模板：**类库** / `classlib`

1.  工作空间/解决方案文件和文件夹：`Chapter06`

1.  项目文件和文件夹：`PacktLibrary`

1.  添加一个控制台应用程序项目，如下列表所定义的那样：

1.  项目模板：**控制台应用程序** / `console`

1.  工作空间/解决方案文件和文件夹：`Chapter06`

1.  项目文件和文件夹：`PeopleApp`

1.  在`PacktLibrary`项目中，将名为`Class1.cs`的文件重命名为`Person.cs`。

1.  修改`Person.cs`文件内容，如下所示的代码：

```cs

使用

静态

System.Console;

命名空间

Packt.Shared

;

公共

类

Person

：对象

{

// 字段

公共

字符串

？ 名称；    // ？允许为空

公共

DateTime DateOfBirth;

公共

List<Person> Children = new

(); // C# 9 或更高版本

// 方法

公共

void

WriteToConsole

()

{

WriteLine($"

{Name}

出生于

{DateOfBirth:dddd}

。"

）;

}

}

```

1.  在`PeopleApp`项目中，添加对`PacktLibrary`的项目引用，如下标记中所示：

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

**<ProjectReference**

**Include=**

**"..\PacktLibrary\PacktLibrary.csproj"**

**/>**

**</ItemGroup>**

</Project>

```

1.  构建`PeopleApp`项目，并注意输出指示两个项目都已成功构建。

# 更多关于方法

我们可能希望两个`Person`的实例能够繁殖。我们可以通过编写方法来实现这一点。实例方法是对象对自身执行的操作；静态方法是类型执行的操作。

您的选择取决于对操作最合理的选择。

**良好实践**：同时具有静态方法和实例方法来执行类似的操作通常是有意义的。例如，`string`既有`Compare`静态方法，也有`CompareTo`实例方法。这样做将功能的使用方式放在了使用您的类型的程序员手中，为他们提供了更多的灵活性。

## 使用方法实现功能

让我们首先通过使用静态和实例方法来实现一些功能：

1.  向`Person`类添加一个实例方法和一个静态方法，允许两个`Person`对象繁殖，如下面的代码所示：

```cs

// "乘法"的静态方法

公共

静态

人

繁殖

(

Person p1, Person p2

)

{

Person baby = new

()

{

Name = $"Baby of

{p1.Name}

和

{p2.Name}

"

};

p1.Children.Add(baby);

p2.Children.Add(baby);

返回

baby;

}

// 实例方法来“乘法”

公共

人

与 arg1 繁殖

(

Person partner

)

{

返回

Procreate(this

，partner);

}

```

注意以下内容：

+   在名为`Procreate`的`static`方法中，传递了要繁殖的`Person`对象作为名为`p1`和`p2`的参数。

+   创建一个名为`baby`的新`Person`类，其名称由已经繁殖的两个人的组合组成。稍后可以通过设置返回的`baby`变量的`Name`属性来更改这一点。

+   `baby`对象被添加到两个父母的`Children`集合中，然后返回。类是引用类型，这意味着在内存中添加的是对`baby`对象的引用，而不是`baby`对象的克隆。您将在本章后面学习引用类型和值类型之间的区别。

+   在名为`ProcreateWith`的实例方法中，传递了要繁殖的`Person`对象作为名为`partner`的参数，它和`this`一起传递给静态的`Procreate`方法以重用方法的实现。`this`是一个关键字，用于引用类的当前实例。

**良好实践**：创建新对象或修改现有对象的方法应返回对该对象的引用，以便调用者可以访问结果。

1.  在`PeopleApp`项目中，在`Program.cs`文件的顶部，删除注释并导入我们的`Person`类的命名空间，并静态导入`Console`类型，如下面的代码所示：

```cs

使用

Packt.Shared;

使用

静态

System.Console;

```

1.  在`Program.cs`中，创建三个人，并让他们相互繁殖，注意，要将双引号字符添加到`string`中，必须使用反斜杠字符进行前缀处理，如下面的代码所示： 

```cs

Person harry = new

() { Name = "Harry"

};

Person mary = new

() { Name = "Mary"

};

Person jill = new

() { Name = "Jill"

};

// 调用实例方法

Person baby1 = mary.ProcreateWith(harry);

baby1.Name = "Gary"

;

// 调用静态方法

Person baby2 = Person.Procreate(harry, jill);

WriteLine($"

{harry.Name}

有

{harry.Children.Count}

children."

);

WriteLine($"

{mary.Name}

有

{mary.Children.Count}

孩子们。

);

WriteLine($"

{jill.Name}

有

{jill.Children.Count}

children."

);

WriteLine(

格式："{0}的第一个孩子名为“{1}”"

，

arg0: harry.Name,

harry.Children[0

].Name);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

Harry 有 2 个孩子。

Mary 有 1 个孩子。

Jill 有 1 个孩子。

Harry 的第一个孩子名为“Gary”。

```

## 使用运算符实现功能

`System.String`类有一个名为`Concat`的`static`方法，它连接两个字符串值并返回结果，如下面的代码所示：

```cs

字符串

s1 = "Hello "

;

字符串

s2 = "World!"

;

字符串

s3 = string

.Concat(s1, s2);

WriteLine(s3); // Hello World!

```

像`Concat`这样的方法是有效的，但程序员可能更自然地使用`+`符号运算符将两个`string`值“相加”在一起，如下面的代码所示：

```cs

字符串

s3 = s1 + s2;

```

一个众所周知的圣经短语是*去生育繁衍*，意思是生育。让我们编写代码，使`*`（乘）符号允许两个`Person`对象繁殖。

我们通过为`*`符号定义一个`static`运算符来实现这一点。语法有点像方法，因为实际上，运算符*就是一个方法，但是使用符号而不是方法名，这使得语法更加简洁。

1.  在`Person.cs`中，为`*`符号创建一个`static`运算符，如下面的代码所示：

```cs

// 乘法运算符

公共

静态

Person operator

*（Person p1，Person p2）

{

返回

Person.Procreate（p1，p2）;

}

```

**良好的实践**：与方法不同，运算符不会出现在类型的 IntelliSense 列表中。对于您定义的每个运算符，都要创建一个方法，因为程序员可能不明显地知道该运算符是可用的。然后，运算符的实现可以调用该方法，重用您编写的代码。提供方法的第二个原因是，并非每种语言编译器都支持运算符；例如，虽然 Visual Basic 和 F#支持算术运算符，如*，但并没有要求其他语言支持 C#支持的所有运算符。

1.  在`Program.cs`中，在调用`Procreate`方法之后并在写入控制台的语句之前，使用`*`运算符再生一个宝宝，如下面代码中所示：

```cs

//调用静态方法

Person baby2 = Person.Procreate（harry，jill）;

**//调用一个运算符**

**Person baby3 = harry * mary;**

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

哈利有 3 个孩子。

玛丽有 2 个孩子。

吉尔有 1 个孩子。

哈利的第一个孩子叫“加里”。

```

## 使用本地函数实现功能

C# 7.0 引入的一种语言特性是能够定义**本地函数**。

本地函数是本地变量的方法等价物。换句话说，它们是仅从定义它们的包含方法中访问的方法。在其他语言中，它们有时被称为**嵌套**或**内部函数**。

本地函数可以在方法的任何地方定义：顶部，底部，甚至中间的某个地方！

我们将使用本地函数来实现阶乘计算：

1.  在`Person.cs`中，添加语句来定义一个使用本地函数内部计算结果的`Factorial`函数，如下面的代码所示：

```cs

//带有本地函数的方法

公共

静态的

int

阶乘

(

int

数字

）

{

如果

（数字<0

）

{

抛出

新的

参数异常（

$"

{

nameof

（数字）}

不能小于零。"

）;

}

返回

localFactorial（数字）;

int

localFactorial

(

int

localNumber

）

//本地函数

{

如果

本地号码小于 1

）返回

1

;

返回

localNumber * localFactorial(localNumber - 1

）;

}

}

```

1.  在`Program.cs`中，添加一个语句来调用`Factorial`函数并将返回值写入控制台，如下面的代码所示：

```cs

WriteLine（$“5！是

{Person.Factorial（

5

)}

"

）;

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

5！是 120

```

# 引发和处理事件

方法经常被描述为*对象可以执行的动作，可以是在自身上执行，也可以是在相关对象上执行*。例如，`List<T>`可以向自身添加项目或清除自身，而`File`可以在文件系统中创建或删除文件。

事件经常被描述为*发生在对象上的动作*。例如，在用户界面中，`Button`有一个`Click`事件，点击是发生在按钮上的事情，而`FileSystemWatcher`监听文件系统的更改通知，并触发`Created`和`Deleted`等事件，当目录或文件发生更改时会触发这些事件。

另一种思考事件的方式是它们提供了两个对象之间交换消息的方式。

事件建立在**委托**之上，所以让我们先看看委托是什么以及它们是如何工作的。

## 使用委托调用方法

您已经看到了调用或执行方法的最常见方式：使用`.`运算符访问其名称。例如，`Console.WriteLine`告诉`Console`类型访问其`WriteLine`方法。

调用或执行方法的另一种方式是使用委托。如果您使用过支持**函数指针**的语言，那么将委托视为**类型安全的方法指针**。

换句话说，委托包含与委托相同签名的方法的内存地址，以便可以安全地使用正确的参数类型调用它。

例如，想象一下`Person`类中有一个方法，它必须传递一个`string`类型作为它唯一的参数，并返回一个`int`类型，如下面的代码所示：

```cs

public

int

我想要调用的方法

(

string

输入

)

{

return

input.Length; // 方法的具体操作并不重要

}

```

我可以这样在名为`p1`的`Person`实例上调用这个方法：

```cs

int

answer = p1.MethodIWantToCall("Frog"

);

```

或者，我可以定义一个具有匹配签名的委托来间接调用方法。请注意，参数的名称不必匹配。只有参数和返回值的类型必须匹配，如下面的代码所示：

```cs

委托

int

具有匹配签名的委托

(

string

s

)

;

```

现在，我可以创建一个委托的实例，将其指向方法，最后调用委托（调用方法），如下面的代码所示：

```cs

// 创建一个委托实例，指向方法

具有匹配签名的委托 d = new

(p1.MethodIWantToCall);

// 调用委托，调用方法

int

答案 2 = d("Frog"

);

```

你可能会想，“这有什么意义呢？”嗯，它提供了灵活性。

例如，我们可以使用委托来创建一个需要按顺序调用的方法队列。在服务中排队执行需要执行的操作是很常见的，以提供更好的可伸缩性。

另一个例子是允许多个操作并行执行。委托内置支持在不同线程上运行的异步操作，并且可以提供改进的响应性。您将在*第十二章*中学习如何做到这一点，*使用多任务改进性能和可伸缩性*。

最重要的例子是，委托允许我们实现事件，以在不需要相互了解的不同对象之间发送消息。事件是组件之间松散耦合的一个例子，因为组件不需要相互了解，它们只需要知道事件签名。

委托和事件是 C#中最令人困惑的特性之一，可能需要几次尝试才能理解，所以如果你感到迷茫，不要担心！

## 定义和处理委托

微软有两个预定义的委托，用作事件。它们的签名很简单，但灵活，如下面的代码所示：

```cs

public

委托

void

EventHandler

(

object

? sender, EventArgs e

)

;

public

委托

void

EventHandler

<

TEventArgs

>(

object

? sender, TEventArgs e

)

;

```

**良好实践**：当您想在自己的类型中定义事件时，应该使用这两个预定义的委托之一。

让我们探讨委托和事件：

1.  在`Person`类中添加语句，并注意以下几点，如下面的代码所示：

+   它定义了一个名为`Shout`的`EventHandler`委托字段。

+   它定义了一个`int`字段来存储`AngerLevel`。

+   它定义了一个名为`Poke`的方法。

+   每次戳一个人，他们的`AngerLevel`都会增加。一旦他们的`AngerLevel`达到三，他们就会触发`Shout`事件，但前提是代码中的某个地方定义了至少一个事件委托指向另一个方法；也就是说，它不是`null`：

```cs

// 委托字段

public

EventHandler? Shout;

// 数据字段

public

int

AngerLevel;

// 方法

public

void

戳

()

{

AngerLevel++;

如果

(AngerLevel >= 3

)

{

// 如果有监听...

如果

(Shout != null

)

{

// ...然后调用委托

Shout(this

, EventArgs.Empty);

}

}

}

```

在调用对象的方法之前检查对象是否不为`null`是非常常见的。C# 6.0 及更高版本允许使用`?`符号在`.`运算符之前内联简化`null`检查，如下面的代码所示：

```cs

Shout?.Invoke(this

, EventArgs.Empty);

```

1.  在`Program.cs`的底部，添加一个具有匹配签名的方法，该方法从`sender`参数获取对`Person`对象的引用，并输出有关它们的一些信息，如下面的代码中所示：

```cs

静态的

void

Harry_Shout

(

对象

? 发送者，事件参数

)

{

如果

（发送者是

null

) return

;

Person p = (Person)sender;

WriteLine($"

{p.Name}

这是生气的吗：

{p.AngerLevel}

."

);

}

```

处理事件的方法名称的微软约定是`ObjectName_EventName`。

1.  在`Program.cs`中，添加一个语句将该方法分配给委托字段，如下面的代码中所示：

```cs

harry.Shout = Harry_Shout;

```

1.  添加语句以调用`Poke`方法四次，然后将该方法分配给`Shout`事件，如下面的代码中所示：

```cs

harry.Shout = Harry_Shout;

**harry.Poke();**

**harry.Poke();**

**harry.Poke();**

**harry.Poke();**

```

1.  运行代码并查看结果，注意 Harry 在被戳两次时不说话，只有在被戳至少三次后才生气到大喊大叫，如下面的输出所示：

```cs

Harry is this angry: 3\.

Harry is this angry: 4.

```

## 定义和处理事件

您现在已经看到委托如何实现事件的最重要功能：定义一个方法的签名，该方法可以由完全不同的代码实现，然后调用该方法和任何其他连接到委托字段的方法。

但是事件呢？它们比您想象的要少。

在将方法分配给委托字段时，不应使用简单的赋值运算符，就像我们在前面的示例中所做的那样。

委托是多播的，这意味着您可以将多个委托分配给单个委托字段。我们可以使用`+=`运算符而不是`=`赋值，以便将更多方法添加到相同的委托字段。当调用委托时，将调用所有分配的方法，尽管您无法控制调用它们的顺序。

如果`Shout`委托字段已经引用了一个或多个方法，通过分配一个方法，它将替换所有其他方法。对于用于事件的委托，我们通常希望确保程序员只使用`+=`运算符或`-=`运算符来分配和删除方法：

1.  为了强制执行这一点，在`Person.cs`中，添加`event`关键字到委托字段声明，如下面的代码中所示：

```cs

公共

**事件**

EventHandler? Shout;

```

1.  构建`PeopleApp`项目并注意编译器错误消息，如下面的输出所示：

```cs

Program.cs(41,13): error CS0079: The event 'Person.Shout' can only appear on the left hand side of += or -=

```

这就是`event`关键字的（几乎）全部作用！如果您永远不会将多个方法分配给委托字段，那么从技术上讲，您不需要“事件”，但表明您的意思并且期望委托字段被用作事件仍然是一个好习惯。

1.  修改方法分配以使用`+=`，如下面的代码中所示：

```cs

harry.Shout += Harry_Shout;

```

1.  运行代码并注意它具有与以前相同的行为。

# 使用泛型使类型可以安全重用

2005 年，随着 C# 2.0 和.NET Framework 2.0，微软引入了一个名为**泛型**的功能，它使您的类型可以更安全地重用并且更有效。它通过允许程序员将类型作为参数传递来实现这一点，类似于您可以将对象作为参数传递。

## 使用非泛型类型

首先，让我们看一个使用非泛型类型的示例，以便您可以了解泛型旨在解决的问题，例如弱类型参数和值，以及使用`System.Object`引起的性能问题。

`System.Collections.Hashtable`可用于存储多个值，每个值都有一个唯一的键，以便以后可以快速查找其值。键和值都可以是任何对象，因为它们声明为`System.Object`。尽管这在存储整数等值类型时提供了灵活性，但它很慢，并且在添加项目时很容易引入错误，因为不进行类型检查。

让我们写一些代码：

1.  在`Program.cs`中，创建一个非泛型集合`System.Collections.Hashtable`的实例，然后向其中添加四个项目，如下面的代码所示：

```cs

// 非泛型查找集合

System.Collections.Hashtable lookupObject = new

();

lookupObject.Add(key: 1

，值

: "Alpha"

);

lookupObject.Add(key: 2

，值

: "Beta"

);

lookupObject.Add(key: 3

, value

: "Gamma"

);

lookupObject.Add(key: harry, value

: "Delta"

);

```

1.  添加语句以定义一个`key`，其值为`2`，并使用它在哈希表中查找其值，如下面的代码所示：

```cs

int

key = 2

; // 查找具有 2 作为其键的值

WriteLine(format: "Key {0} has value: {1}"

,

arg0: key,

arg1: lookupObject[key]);

```

1.  添加语句以使用`harry`对象查找其值，如下面的代码所示：

```cs

// 查找具有 harry 作为其键的值

WriteLine(format: "Key {0} has value: {1}"

，

arg0: harry,

arg1: lookupObject[harry]);

```

1.  运行代码并注意它的工作原理，如下面的输出所示：

```cs

键 2 的值为：Beta

Key Packt.Shared.Person has value: Delta

```

尽管代码可以工作，但是由于键或值可以使用任何类型，因此存在出错的可能性。如果另一个开发人员使用您的查找对象，并且期望所有项目都是某种类型，他们可能会将它们转换为该类型并因为某些值可能是不同类型而导致异常。具有大量项目的查找对象也会导致性能不佳。

**良好实践**：避免在`System.Collections`命名空间中使用类型。

## 使用泛型类型

`System.Collections.Generic.Dictionary<TKey, TValue>`可用于存储多个值，每个值都有一个唯一的键，以便以后可以快速查找其值。键和值都可以是任何对象，但是在首次实例化集合时，必须告诉编译器键和值的类型是什么。您可以通过在尖括号`<>`，`TKey`和`TValue`中指定**泛型参数**的类型来实现这一点。

**良好实践**：当泛型类型有一个可定义的类型时，应将其命名为`T`，例如`List<T>`，其中`T`是列表中存储的类型。当泛型类型有多个可定义的类型时，它们应该使用`T`作为名称前缀，并具有合理的名称，例如`Dictionary<TKey, TValue>`。

这提供了灵活性，速度更快，并且更容易避免错误，因为在添加项目时进行了类型检查。

让我们编写一些代码来使用泛型解决问题：

1.  在`Program.cs`中，创建一个泛型查找集合`Dictionary<TKey, TValue>`的实例，然后向其中添加四个项目，如下面的代码所示：

```cs

// 泛型查找集合

Dictionary<int

, string

> lookupIntString = new

();

lookupIntString.Add(key: 1

, value

: "Alpha"

);

lookupIntString.Add(key: 2

, value

: "Beta"

);

lookupIntString.Add(key: 3

, value

: "Gamma"

);

lookupIntString.Add(key: harry, value

: "Delta"

);

```

1.  注意在使用`harry`作为键时的编译错误，如下面的输出所示：

```cs

/Users/markjprice/Code/Chapter06/PeopleApp/Program.cs(98,32): error CS1503: Argument 1: cannot convert from 'Packt.Shared.Person' to 'int' [/Users/markjprice/Code/Chapter06/PeopleApp/PeopleApp.csproj]

```

1.  用`4`替换`harry`。

1.  添加语句以将`key`设置为`3`，并使用它在字典中查找其值，如下面的代码所示：

```cs

key = 3

;

WriteLine(format: "Key {0} has value: {1}"

,

arg0: key,

arg1: lookupIntString[key]);

```

1.  运行代码并注意它可以工作，如下面的输出所示：

```cs

键 3 的值为：Gamma

```

# 实现接口

接口是连接不同类型以创建新事物的一种方式。将它们视为 LEGO™积木顶部的凸起，使它们可以“粘”在一起，或者是插头和插座的电气标准。

如果类型实现了接口，那么它就承诺.NET 的其余部分支持特定功能。这就是为什么有时它们被描述为合同。

## 常见接口

以下是一些您的类型可能需要实现的常见接口：

| 接口 | 方法 | 描述 |
| --- | --- | --- |
| `IComparable` | `CompareTo(other)` | 这定义了一个类型实现的比较方法，用于对其实例进行排序。 |
| `IComparer` | `Compare(first, second)` | 这定义了一个次要类型实现的比较方法，用于对主类型的实例进行排序。 |
| `IDisposable` | `Dispose()` | 这定义了一种释放非托管资源的处理方法，比等待终结器更有效（有关更多详细信息，请参见本章后面的*释放非托管资源*部分。 |
| `IFormattable` | `ToString(format, culture)` | 这定义了一个具有文化意识的方法，将对象的值格式化为字符串表示形式。 |
| `IFormatter` | `Serialize(stream, object)``Deserialize(stream)` | 这定义了将对象转换为字节流并从中转换的方法，以便进行存储或传输。 |
| `IFormatProvider` | `GetFormat(type)` | 这定义了一种根据语言和地区格式化输入的方法。 |

## 在排序时比较对象

您可能想要实现的最常见的接口之一是`IComparable`。它有一个名为`CompareTo`的方法。它有两种变体，一种适用于可空的`object`类型，一种适用于可空的泛型类型`T`，如下面的代码所示：

```cs

namespace

System

{

public

interface

IComparable

{

int

CompareTo

(

object

? obj

)

;

}

public

interface

IComparable

<in

T

{

int

CompareTo

(

T? other

)

;

}

}

```

例如，`string`类型通过返回`-1`来实现`IComparable`，如果`string`小于要比较的`string`，则返回`1`。`int`类型通过返回`-1`来实现`IComparable`，如果`int`小于要比较的`int`，则返回`1`。

如果类型实现了`IComparable`接口之一，则数组和集合可以对其进行排序。

在我们为`Person`类实现`IComparable`接口及其`CompareTo`方法之前，让我们看看当我们尝试对`Person`实例数组进行排序时会发生什么：

1.  在`Program.cs`中，添加语句创建`Person`实例数组并将项目写入控制台，然后尝试对数组进行排序并再次将项目写入控制台，如下面的代码所示：

```cs

Person[] people =

{

new

() { Name = "Simon"

},

new

() { Name = "Jenny"

},

new

() { Name = "Adam"

},

new

() { Name = "Richard"

}

};

WriteLine("人员的初始列表："

);

foreach

(Person p in

people)

{

WriteLine($"

{p.Name}

"

);

}

WriteLine("使用 Person 的 IComparable 实现进行排序："

);

Array.Sort(people);

foreach

(Person p in

people)

{

WriteLine($"

{p.Name}

"

);

}

```

1.  运行代码会抛出异常。正如消息所解释的那样，为了解决问题，我们的类型必须实现`IComparable`，如下面的输出所示：

```cs

未处理的异常：System.InvalidOperationException：无法比较数组中的两个元素。--->System.ArgumentException：至少一个对象必须实现 IComparable。

```

1.  在`Person.cs`中，从`object`继承后，添加逗号并输入`IComparable<Person>`，如下面的代码所示：

```cs

public

class

Person

: object

, IComparable

<Person

```

您的代码编辑器将在新代码下面绘制红色波浪线，以警告您尚未实现您承诺的方法。如果单击灯泡并选择**实现接口**选项，则代码编辑器可以为您编写骨架实现。

1.  向下滚动到`Person`类的底部，找到为您编写的方法，并删除抛出`NotImplementedException`错误的语句，如下面代码中所示：

```cs

public

int

CompareTo

(

Person? 其他

)

{

**抛出**

**新**

**NotImplementedException();**

}

```

1.  添加一个语句来调用`Name`字段的`CompareTo`方法，该方法使用`string`类型的`CompareTo`实现并返回结果，如下面代码中所示：

```cs

public

int

CompareTo

(

Person? 其他

)

{

如果

(Name is

null

) 返回

0

;

**返回**

**Name.CompareTo(other?.Name);**

}

```

我们选择通过比较它们的`Name`字段来比较两个`Person`实例。因此，`Person`实例将按其名称的字母顺序排序。为简单起见，我在这些示例中没有添加`null`检查。

1.  运行代码并注意，这次它按预期工作，如下面的输出所示：

```cs

人员的初始列表：

西蒙

珍妮

亚当

理查德

使用 Person 的 IComparable 实现进行排序：

亚当

珍妮

理查德

西蒙

```

**良好实践**：如果有人想要对您的类型的实例数组或集合进行排序，则实现`IComparable`接口。

## 使用单独类比较对象

有时，您可能无法访问类型的源代码，并且它可能没有实现`IComparable`接口。幸运的是，还有另一种方法可以对类型的实例进行排序。您可以创建一个实现略有不同接口的单独类型，名为`IComparer`：

1.  在`PacktLibrary`项目中，添加一个名为`PersonComparer.cs`的新类文件，其中包含一个实现`IComparer`接口的类，该接口将比较两个人，即两个`Person`实例。通过比较它们的`Name`字段的长度来实现它，或者如果名称长度相同，则按字母顺序比较名称，如下面的代码所示：

```cs

命名空间

Packt.Shared

;

public

类

PersonComparer

: IComparer

<Person

{

public

int

比较

(

Person? x, Person? y

)

{

如果

(x is

null

|| y is

null

)

{

返回

0

;

}

// 比较名称长度...

int

result = x.Name.Length.CompareTo(y.Name.Length);

// ...如果它们相等...

如果

(result == 0

)

{

// ...然后按名称比较...

返回

x.Name.CompareTo(y.Name);

}

否则

// 结果将是-1 或 1

{

// ...否则按长度比较。

返回

result;

}

}

}

```

1.  在`Program.cs`中，添加语句以使用此替代实现对数组进行排序，如下面的代码所示：

```cs

WriteLine("使用 PersonComparer 的 IComparer 实现进行排序："

);

Array.Sort(people, new

PersonComparer());

foreach

(Person p in

人)

{

WriteLine($"

{p.Name}

"

);

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

使用 PersonComparer 的 IComparer 实现进行排序：

亚当

珍妮

西蒙

理查德

```

这次，当我们对`people`数组进行排序时，我们明确要求排序算法使用`PersonComparer`类型，以便首先对人员进行排序，例如亚当，最后对最长的名称进行排序，例如理查德；当两个或更多名称的长度相等时，按字母顺序对它们进行排序，例如珍妮和西蒙。

## 隐式和显式接口实现

接口可以被隐式和显式地实现。隐式实现更简单，更常见。只有在类型必须具有多个具有相同名称和签名的方法时，才需要显式实现。

例如，`IGamePlayer`和`IKeyHolder`都可能有一个名为`Lose`的方法，因为游戏和钥匙都可以丢失。在必须实现两个接口的类型中，`Lose`的实现只能是隐式方法。如果两个接口可以共享相同的实现，那么可以，但如果不行，那么另一个`Lose`方法将必须以不同的方式实现并显式调用，如下面的代码所示：

```cs

公共的

接口

IGamePlayer

{

void

失去

()

;

}

公共的

接口

IKeyHolder

{

void

失去

()

;

}

公共的

类

人

: IGamePlayer

, IKeyHolder

{

公共的

void

失去

()

// 隐式实现

{

// 实现失去钥匙

}

void

IGamePlayer.Lose() // 显式实现

{

// 实现失去游戏

}

}

// 调用 Lose 的隐式和显式实现

人 p = new

();

p.Lose(); // 调用失去钥匙的隐式实现

((IGamePlayer)p).Lose(); // 调用失去游戏的显式实现

IGamePlayer player = p as

IGamePlayer;

player.Lose(); // 调用失去游戏的显式实现

```

## 定义具有默认实现的接口

C# 8.0 引入的一种语言特性是接口的**默认实现**。让我们看看它的作用：

1.  在`PacktLibrary`项目中，添加一个名为`IPlayable.cs`的新文件。

1.  修改语句以定义一个公共的`IPlayable`接口，其中包含两个`Play`和`Pause`方法，如下面的代码所示：

```cs

namespace

Packt.Shared

;

公共的

接口

IPlayable

{

void

播放

()

;

void

暂停

()

;

}

```

1.  在`PacktLibrary`项目中，添加一个名为`DvdPlayer.cs`的新类文件。

1.  修改文件中的语句以实现`IPlayable`接口，如下面的代码所示：

```cs

使用

静态的

System.Console;

namespace

Packt.Shared

;

公共的

类

DvdPlayer

: IPlayable

{

公共的

void

暂停

()

{

写行("DVD 播放器正在暂停。"

);

}

公共的

void

播放

()

{

写行("DVD 播放器正在播放。"

);

}

}

```

这很有用，但如果我们决定添加一个名为`Stop`的第三个方法怎么办？在 C# 8.0 之前，一旦至少有一个类型实现了原始接口，这将是不可能的。接口的主要目的之一是它是一个固定的合同。

C# 8.0 允许接口在发布后添加新成员，只要它们有默认实现。C#纯粹主义者不喜欢这个想法，但出于实际原因，比如避免破坏性更改或不得不定义一个全新的接口，这是有用的，其他语言如 Java 和 Swift 也支持类似的技术。

默认接口实现的支持需要对底层平台进行一些基本更改，因此只有在目标框架为.NET 5.0 或更高版本、.NET Core 3.0 或更高版本、或.NET Standard 2.1 时才支持。因此，它们不受.NET Framework 的支持。

1.  修改`IPlayable`接口以添加一个具有默认实现的`Stop`方法，如下面代码中的高亮部分所示：

```cs

**使用**

**静态的**

**System.Console;**

namespace

Packt.Shared

;

公共的

接口

IPlayable

{

void

播放

()

;

void

暂停

()

;

**void**

**停止**

**()**

**// 默认接口实现**

**{**

**写行(**

**"停止的默认实现。"**

**);**

**}**

}

```

1.  构建`PeopleApp`项目，并注意尽管`DvdPlayer`类没有实现`Stop`，但项目仍然成功编译。将来，我们可以通过在`DvdPlayer`类中实现它来覆盖`Stop`的默认实现。

# 使用引用和值类型管理内存

我已经多次提到引用类型。让我们更详细地看看它们。

内存有两种类别：**栈**内存和**堆**内存。在现代操作系统中，栈和堆可以位于物理或虚拟内存中的任何位置。

堆栈内存的工作速度更快（因为它由 CPU 直接管理，并且因为它使用后进先出机制，更有可能在其 L1 或 L2 缓存中有数据），但大小有限，而堆内存速度较慢，但更加丰富。

例如，在 macOS 终端中，我可以输入命令`ulimit -a`来发现堆栈大小限制为 8192 KB，其他内存为“无限”。这有限的堆栈内存量是为什么很容易填满并出现“堆栈溢出”的原因。

## 定义引用类型和值类型

有三个 C#关键字可以用来定义对象类型：`class`，`record`和`struct`。所有这些都可以有相同的成员，比如字段和方法。它们之间的一个区别是内存分配的方式。

当你使用`record`或`class`定义类型时，你正在定义一个**引用类型**。这意味着对象本身的内存分配在堆上，只有对象的内存地址（和一些开销）存储在堆栈上。

当你使用`record struct`或`struct`定义类型时，你正在定义一个**值类型**。这意味着对象本身的内存分配在堆栈上。

如果一个`struct`使用的字段类型不是`struct`类型，那么这些字段将存储在堆上，这意味着该对象的数据同时存储在堆和堆栈中！

这些是最常见的结构类型：

+   **数字**`System`**类型**：`byte`，`sbyte`，`short`，`ushort`，`int`，`uint`，`long`，`ulong`，`float`，`double`，和`decimal`

+   **其他**`System`**类型**：`char`，`DateTime`，和`bool`

+   `System.Drawing`**类型**：`Color`，`Point`，和`Rectangle`

几乎所有其他类型都是`class`类型，包括`string`。

除了在内存中存储类型数据的位置不同之外，另一个主要区别是你不能从`struct`继承。

## 引用类型和值类型在内存中的存储方式

假设你有一个控制台应用程序，声明了一些变量，如下面的代码所示：

```cs

int

number1 = 49

；

long

number2 = 12

；

System.Drawing.Point location = new

（x：4

，y：5

）;

Person kevin = new

() { Name = "Kevin"

，

DateOfBirth = new

（年份：1988

，月：9

，日：23

) };

Person sally;

```

让我们回顾一下当这些语句执行时，在堆栈和堆上分配了哪些内存，如*图 6.1*所示，并在以下列表中描述：

+   `number1`变量是一个值类型（也称为`struct`），因此它在堆栈上分配，并且使用 4 个字节的内存，因为它是一个 32 位整数。它的值 49 直接存储在变量中。

+   `number2`变量也是一个值类型，因此也在堆栈上分配，并且使用 8 个字节，因为它是一个 64 位整数。

+   `location`变量也是一个值类型，因此它在堆栈上分配，并且使用 8 个字节，因为它由两个 32 位整数`x`和`y`组成。

+   `kevin`变量是一个引用类型（也称为`class`），因此在堆栈上分配了 8 个字节用于 64 位内存地址（假设 64 位操作系统），并在堆上分配了足够的字节来存储`Person`的实例。

+   `sally`变量是一个引用类型，因此在堆栈上分配了 8 个字节用于 64 位内存地址。它目前是`null`，意味着尚未为它在堆上分配内存。

![](img/Image00069.jpg)

图 6.1：值类型和引用类型在堆栈和堆中的分配方式

引用类型的所有分配的内存都存储在堆上。如果引用类型如`Person`的字段使用值类型如`DateTime`，那么`DateTime`的值将存储在堆上。

如果值类型有一个字段是引用类型，那么值类型的那部分存储在堆上。`Point`是一个值类型，由两个字段组成，它们都是值类型，因此整个对象可以分配在堆栈上。如果`Point`值类型有一个字段是引用类型，比如`string`，那么`string`的字节将存储在堆上。

## 类型的相等性

比较两个变量使用`==`和`!=`运算符是很常见的。这两个运算符的行为对于引用类型和值类型是不同的。

当您检查两个值类型变量的相等性时，.NET 会在堆栈上直接比较这两个变量的值，并在它们相等时返回`true`，如下面的代码所示：

```cs

int

a = 3

;

int

b = 3

;

WriteLine($"a == b:

{(a == b)}

"

); // true

```

当您检查两个引用类型变量的相等性时，.NET 比较这两个变量的内存地址，并在它们相等时返回`true`，如下面的代码所示：

```cs

Person a = new

() { Name = "Kevin"

};

Person b = new

() { Name = "Kevin"

};

WriteLine($"a == b:

{(a == b)}

"

); // false

```

这是因为它们不是相同的对象。如果两个变量指向堆上的同一个对象，那么它们将相等，如下面的代码所示：

```cs

Person a = new

() { Name = "Kevin"

};

Person b = a;

WriteLine($"a == b:

{(a == b)}

"

); // true

```

这种行为的一个例外是`string`类型。它是一个引用类型，但相等运算符已被重写，使它们的行为就像值类型一样，如下面的代码所示：

```cs

字符串

a = "Kevin"

;

字符串

b = "Kevin"

;

WriteLine($"a == b:

{(a == b)}

"

); // true

```

您可以对您的类做类似的事情，使相等运算符返回`true`，即使它们不是相同的对象（堆上的相同内存地址），而是如果它们的字段具有相同的值，但这超出了本书的范围。或者，使用`record class`，因为它们的一个好处是它们为您实现了这种行为。

## 定义结构类型

让我们探索定义自己的值类型：

1.  在`PacktLibrary`项目中，添加一个名为`DisplacementVector.cs`的文件。

1.  修改文件，如下面的代码所示，并注意以下内容：

+   类型是使用`struct`而不是`class`声明的。

+   它有两个`int`字段，名为`X`和`Y`。

+   它有一个用于设置`X`和`Y`的初始值的构造函数。

+   它有一个用于将两个实例相加并返回类型的新实例的操作符，其中`X`添加到`X`，`Y`添加到`Y`。

```cs

命名空间

Packt.Shared

;

公共

结构

DisplacementVector

{

公共

int

X;

公共

int

Y;

公共

位移向量

(

int

initialX，

int

initialY

)

{

X = initialX;

Y = initialY;

}

公共

静态

DisplacementVector operator

+(

DisplacementVector vector1，

DisplacementVector vector2)

{

返回

新

(

vector1.X + vector2.X，

vector1.Y + vector2.Y);

}

}

```

1.  在`Program.cs`中，添加语句创建两个新的`DisplacementVector`实例，将它们相加，并输出结果，如下面的代码所示：

```cs

DisplacementVector dv1 = new

(3

，5

);

DisplacementVector dv2 = new

(-2

，7

);

DisplacementVector dv3 = dv1 + dv2;

WriteLine($"(

{dv1.X}

，

{dv1.Y}

) + (

{dv2.X}

，

{dv2.Y}

) = (

{dv3.X}

，

{dv3.Y}

)"

);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

(3, 5) + (-2, 7) = (1, 12)

```

**良好实践**：如果类型中所有字段使用的总字节数不超过 16 字节，类型只使用值类型作为其字段，并且永远不想从类型派生，那么 Microsoft 建议使用`struct`。如果类型使用的堆栈内存超过 16 字节，如果它使用引用类型作为其字段，或者可能想要从中继承，那么使用`class`。

## 使用记录结构类型

C# 10 引入了使用`record`关键字与`struct`类型以及类类型一起使用的能力。

我们可以定义`DisplacementVector`类型，如下面的代码所示：

```cs

公共

记录

结构

位移矢量

（

int

X，

int

Y

）

;

```

通过这个改变，微软建议如果要定义`record class`，则明确指定`class`，即使`class`关键字是可选的，如下面的代码所示：

```cs

公共

记录

类

ImmutableAnimal

（

字符串

名称

）

;

```

## 释放非托管资源

在前一章中，我们看到构造函数可以用来初始化字段，并且一个类型可以有多个构造函数。想象一下，一个构造函数分配了一个非托管资源；也就是说，任何不受.NET 控制的东西，比如操作系统控制下的文件或互斥体。非托管资源必须手动释放，因为.NET 无法使用其自动垃圾收集功能为我们释放它。

垃圾收集是一个高级主题，所以在这个主题中，我将展示一些代码示例，但您不需要自己编写代码。

每种类型都可以有一个**终结器**，当需要释放资源时，.NET 运行时将调用该终结器。终结器与构造函数具有相同的名称；也就是说，类型名称，但前缀是波浪号`~`。

不要将终结器（也称为**析构函数**）与`Deconstruct`方法混淆。析构函数释放资源；也就是说，它在内存中销毁对象。`Deconstruct`方法返回一个分解为其组成部分的对象，并使用 C#分解语法，例如，在使用元组时：

```cs

公共

类

动物

{

公共

动物

（）

//构造函数

{

//分配任何非托管资源

}

~动物（）//终结器也称为析构函数

{

// 释放任何非托管资源

}

}

}

在处理非托管资源时，上述代码示例是您应该做的最少工作。但是，仅提供终结器的问题在于，.NET 垃圾收集器需要两次垃圾收集才能完全释放为该类型分配的资源。

虽然可选，但建议还提供一个方法，允许使用您的类型的开发人员显式释放资源，以便垃圾收集器可以立即和确定地释放非托管资源的托管部分，比如文件，然后在单次垃圾收集中释放对象的托管内存部分，而不是两轮垃圾收集。

有一个实现`IDisposable`接口的标准机制，如下例所示：

```cs

公共

类

动物

：IDisposable

{

公共

动物

（）

{

//分配非托管资源

}

~动物（）//终结器

{

Dispose(false

）;

}

布尔

已处理=假

; //资源是否已被释放？

公共

无效

处理

（）

{

Dispose(true

）;

//告诉垃圾收集器不需要调用终结器

GC.SuppressFinalize(this

）;

}

protected

虚拟

无效

处理

（

布尔

处理

）

{

如果

（已处理）返回

;

//释放*非托管*资源

// ...

如果

（处理）

{

// 释放任何其他*托管*资源

// ...

}

已处理=真

;

}

}

```

有两个`Dispose`方法，一个`public`，一个`protected`：

+   `public void Dispose`方法将由使用您的类型的开发人员调用。调用时，需要释放非托管和托管资源。

+   具有`bool`参数的`protected virtual void Dispose`方法在内部用于实现资源的释放。它需要检查`disposing`参数和`disposed`字段，因为如果终结器线程已经运行并调用了`~Animal`方法，那么只需要释放非托管资源。

调用`GC.SuppressFinalize(this)`是通知垃圾收集器不再需要运行终结器，并且消除了需要进行第二次垃圾收集。

## 确保调用 Dispose

当有人使用实现了`IDisposable`的类型时，他们可以确保使用`using`语句调用公共的`Dispose`方法，如下面的代码所示：

```cs

使用

（Animal a = new

（）

{

// 使用 Animal 实例的代码

}

```

编译器将您的代码转换为以下内容，这样即使发生异常，`Dispose`方法仍将被调用：

```cs

Animal a = new

();

try

{

// 使用 Animal 实例的代码

}

最终

{

if

(a != null

）a.Dispose();

}

```

您将看到使用`IDisposable`、`using`语句和`try`...`finally`块释放非托管资源的实际示例，*第九章*，*使用文件、流和序列化*中有详细介绍。

# 使用 null 值

您已经看到如何将诸如数字之类的原始值存储在`struct`变量中。但是如果一个变量还没有值怎么办？我们如何表示？C#有一个`null`值的概念，可以用来表示一个变量还没有被设置。

## 使值类型可空

默认情况下，像`int`和`DateTime`这样的值类型必须始终有一个值，因此它们的名称。有时，例如在读取存储在允许空、缺失或空值的数据库中的值时，允许值类型为`null`是方便的。我们称之为**可空值类型**。

您可以通过在声明变量时在类型后面添加问号来启用此功能。

让我们看一个例子：

1.  使用您喜欢的编码工具将一个新的**控制台应用程序**添加到名为`NullHandling`的`Chapter06`工作区/解决方案中。本节需要一个带有项目文件的完整应用程序，因此您将无法使用.NET 交互式笔记本。

1.  在 Visual Studio Code 中，选择`NullHandling`作为活动的 OmniSharp 项目。在 Visual Studio 中，将`NullHandling`设置为启动项目。

1.  在`Program.cs`中，输入语句来声明和赋值，包括`null`，给`int`变量，如下面的代码所示：

```cs

int

thisCannotBeNull  = 4

;

thisCannotBeNull = null

; // 编译错误！

int

? thisCouldBeNull = null

;

WriteLine(thisCouldBeNull);

WriteLine(thisCouldBeNull.GetValueOrDefault());

thisCouldBeNull = 7

;

WriteLine(thisCouldBeNull);

WriteLine(thisCouldBeNull.GetValueOrDefault());

```

1.  注释掉导致编译错误的语句。

1.  运行代码并查看结果，如下面的输出所示：

```cs

0

7

7

```

第一行是空的，因为它输出了`null`值！

## 理解可空引用类型

`null`值的使用是如此普遍，在许多语言中都是如此，以至于许多有经验的程序员从未质疑其存在的必要性。但有许多情况下，如果一个变量不允许有`null`值，我们可以编写更好、更简单的代码。

C# 8 中对语言最重大的改变是引入了可空和非可空引用类型。"但等等!"，你可能会想，"引用类型已经是可空的了!"

你是对的，但在 C# 8 及以后的版本中，可以通过设置文件或项目级选项来配置引用类型不再允许`null`值，从而启用这一有用的新功能。由于这对 C#来说是一个重大的改变，微软决定将这一功能设置为选择性的。

由于成千上万的现有库包和应用程序将期望旧的行为，这种新的 C#语言特性需要多年的时间才能产生影响。即使是微软也没有时间在.NET 6 之前在所有主要的.NET 包中完全实现这一新功能。

在过渡期间，您可以为自己的项目选择几种方法：

+   **默认**：不需要更改。不支持非可空引用类型。

+   **选择性项目，选择性文件**：在项目级别启用该功能，并对需要保持与旧行为兼容的任何文件进行选择性退出。这是微软在更新自己的包以使用这一新功能时内部使用的方法。

+   **选择性文件**：仅为单个文件启用该功能。

## 启用可空和非空引用类型

要在项目级别启用该功能，请将以下内容添加到项目文件中：

```cs

<PropertyGroup>

...

<Nullable>enable</Nullable>

</PropertyGroup>

```

这现在是项目模板的默认设置，目标为.NET 6.0。

要在文件级别禁用该功能，请在代码文件顶部添加以下内容：

```cs

#nullable disable

```

要在文件级别启用该功能，请在代码文件顶部添加以下内容：

```cs

#nullable enable

```

## 声明非空变量和参数

如果您启用了可空引用类型，并且希望将引用类型分配为`null`值，则必须使用与使值类型可空相同的语法，即在类型声明后添加`?`符号。

那么，可空引用类型是如何工作的呢？让我们看一个例子。当存储有关地址的信息时，您可能希望强制对街道、城市和地区进行赋值，但建筑可以留空，即`null`：

1.  在`NullHandling.csproj`文件的`Program.cs`文件中，向文件底部添加语句来声明一个具有四个字段的`Address`类，如下面的代码所示：

```cs

类

地址

{

公共

字符串

？建筑;

公共

字符串

街道;

公共

字符串

城市;

公共

字符串

地区;

}

```

1.  几秒钟后，注意警告，比如`Street`未初始化，如*图 6.2*所示：![图形用户界面，文本，应用程序，聊天或短信描述自动生成](img/Image00070.jpg)

图 6.2：PROBLEMS 窗口中关于非空字段的警告消息

1.  将空的`string`值分配给三个非空字段，如下面的代码所示：

```cs

公共

字符串

街道=字符串

.Empty;

公共

字符串

城市=字符串

.Empty;

公共

字符串

地区=字符串

.Empty;

```

1.  在`Program.cs`文件的顶部，静态导入`Console`，然后添加语句来实例化`Address`并设置其属性，如下面的代码所示：

```cs

Address address = new

();

address.Building=null

;

address.Street = null

;

address.City = "伦敦"

;

address.Region=null

;

```

1.  请注意警告，如*图 6.3*所示：![图形用户界面，文本，应用程序，聊天或短信，电子邮件描述自动生成](img/Image00071.jpg)

图 6.3：关于将 null 赋给非空字段的警告消息

因此，这就是为什么新的语言特性被称为可空引用类型。从 C# 8.0 开始，未装饰的引用类型可以变为非空，而使引用类型可空的语法与使值类型可空的语法相同。

## 检查 null

检查可空引用类型或可空值类型变量当前是否包含`null`很重要，因为如果不这样做，可能会抛出`NullReferenceException`，导致错误。在使用可空变量之前，应检查是否为`null`值，如下面的代码所示：

```cs

//在使用之前检查变量是否为 null

如果

（thisCouldBeNull != null

)

{

//访问 thisCouldBeNull 的成员

int

长度= thisCouldBeNull.Length; //可能会抛出异常

...

}

```

C# 7 引入了`is`与`!`（非）运算符结合，作为`!=`的替代，如下面的代码所示：

```cs

如果

(!(thisCouldBeNull is

null

))

{

```

C# 9 引入了`is not`作为更清晰的替代方案，如下面的代码所示：

```cs

如果

（这可能为空

不是

null

)

{

```

如果您尝试使用可能为`null`的变量的成员，请使用空值条件运算符`?.`进行检查，如下面的代码所示：

```cs

字符串

authorName=null

;

//以下会引发 NullReferenceException

int

x = authorName.Length;

//而不是抛出异常，将 null 分配给 y

int

？y=authorName?.Length;

```

有时，您可能希望将变量分配给结果，或者在变量为`null`时使用替代值，例如`3`。您可以使用空值合并运算符`??`来实现这一点，如下面的代码所示：

```cs

// 如果 authorName?.Length 为 null，则结果将为 3

int

result = authorName?.Length ?? 3

;

Console.WriteLine(result);

```

**良好实践**：即使启用了可空引用类型，您仍应检查非可空参数是否为`null`并抛出`ArgumentNullException`。

### 检查方法参数中的 null

在定义带参数的方法时，最好检查`null`值。

在 C#的早期版本中，您将不得不编写`if`语句来检查`null`参数值，然后为任何`null`参数抛出`ArgumentNullException`，如下面的代码所示：

```cs

公共

void

雇佣

（

人经理，人员工

）

{

如果

（经理== null

）

{

抛出

新

ArgumentNullException(nameof

（经理）;

}

如果

（员工== null

）

{

抛出

新

ArgumentNullException(nameof

（员工）;

}

...

}

```

C# 11 可能会引入一个新的`!!`后缀，为您执行此操作，如下面的代码所示：

```cs

公共

void

雇佣

(

人经理！！，人员工！！

)

{

...

}

```

`if`语句和抛出异常已经为您完成。

# 从类继承

我们之前创建的`Person`类型派生（继承）自`object`，即`System.Object`的别名。现在，我们将创建一个从`Person`继承的子类：

1.  在`PacktLibrary`项目中，添加一个名为`Employee.cs`的新类文件。

1.  修改其内容以定义一个名为`Employee`的从`Person`派生的类，如下面的代码所示：

```cs

使用

System;

命名空间

Packt.Shared

;

公共

类

员工

：人

{

}

```

1.  在`Program.cs`中，添加语句来创建`Employee`类的一个实例，如下面的代码所示：

```cs

Employee john = new

()

{

Name = "John Jones"

，

DateOfBirth = new

（年份：1990

，月：7

，日：28

)

};

john.WriteToConsole();

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

John Jones 出生于一个星期六。

```

请注意，`Employee`类继承了`Person`的所有成员。

## 扩展类以添加功能

现在，我们将添加一些特定于员工的成员以扩展该类。

1.  在`Employee.cs`中，添加语句来定义员工代码和他们被雇佣的日期两个属性，如下面的代码所示：

```cs

公共

字符串

? EmployeeCode { get

; set

; }

公共

DateTime HireDate { get

; set

; }

```

1.  在`Program.cs`中，添加语句来设置 John 的员工代码和雇佣日期，如下面的代码所示：

```cs

john.EmployeeCode = "JJ001"

;

john.HireDate = new

（年份：2014

，月：11

，日：23

）;

WriteLine($"

{john.Name}

被雇佣于

{john.HireDate:dd/MM/yy}

"

）;

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

John Jones 于 23/11/14 被雇佣

```

## 隐藏成员

到目前为止，`WriteToConsole`方法是从`Person`继承的，它只输出员工的姓名和出生日期。我们可能希望更改此方法对于员工的操作：

1.  在`Employee.cs`中，添加语句来重新定义`WriteToConsole`方法，如下面的代码中所突出显示的那样：

```cs

**使用**

**静态**

**System.Console;**

命名空间

Packt.Shared

;

公共

类

员工

：人

{

公共

字符串

? EmployeeCode { get

; set

; }

公共

DateTime HireDate { get

; set

; }

**公共**

**void**

**WriteToConsole**

**()**

**{**

**WriteLine（格式：**

**"{0}出生于{1:dd/MM/yy}，并于{2:dd/MM/yy}被雇佣"**

**，**

**arg0：姓名，**

参数 1：出生日期，

**arg2：HireDate）;**

**}**

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

John Jones 出生于 28/07/90，于 01/01/01 被雇佣

John Jones 于 23/11/14 被雇佣

```

您的编码工具警告您，您的方法现在通过在方法名称下绘制一个波浪线来隐藏`Person`的方法，**PROBLEMS**/**Error List**窗口包含更多详细信息，当您构建和运行控制台应用程序时，编译器将输出警告，如*图 6.4*所示：

![图形用户界面，文本，应用程序，电子邮件描述自动生成](img/Image00072.jpg)

图 6.4：隐藏方法警告

如警告所述，您可以通过对方法应用`new`关键字来隐藏此消息，以表明您有意替换旧方法，如下面的代码中所突出显示的那样：

```cs

public

**new**

void

WriteToConsole

()

```

## 覆盖成员

与其隐藏一个方法，通常最好是**覆盖**它。只有在基类选择允许覆盖时，才能覆盖，方法是通过对应用`virtual`关键字到应允许覆盖的任何方法。

让我们看一个例子：

1.  在`Program.cs`中，添加一条语句，使用其`string`表示形式将`john`变量的值写入控制台，如下面的代码所示：

```cs

WriteLine(john.ToString());

```

1.  运行代码并注意`ToString`方法是从`System.Object`继承的，因此实现返回命名空间和类型名称，如下面的输出所示：

```cs

Packt.Shared.Employee

```

1.  在`Person.cs`中，通过添加一个`ToString`方法来覆盖这种行为，以输出人的姓名以及类型名称，如下面的代码所示：

```cs

// 覆盖的方法

public

覆盖

string

ToString

()

{

return

$"

{Name}

是一个

{

基类

.ToString()}

"

;

}

```

`base`关键字允许子类访问其超类的成员；即，它继承或派生自的**基类**。

1.  运行代码并查看结果。现在，当调用`ToString`方法时，它会输出人的姓名，并返回基类的`ToString`实现，如下面的输出所示：

```cs

John Jones 是 Packt.Shared.Employee

```

**良好实践**：许多现实世界的 API，例如 Microsoft 的 Entity Framework Core，Castle 的 DynamicProxy 和 Episerver 的内容模型，要求您在类中定义的属性标记为`virtual`，以便它们可以被覆盖。仔细决定哪些方法和属性成员应标记为`virtual`。

## 从抽象类继承

在本章的前面，您了解了可以定义一组成员的接口，以满足基本功能的要求。这些非常有用，但它们的主要限制是，在 C# 8 之前，它们不能提供任何自己的实现。

这是一个特殊的问题，如果您仍然需要创建将与.NET Framework 和其他不支持.NET Standard 2.1 的平台一起使用的类库。

在早期的平台上，您可以使用抽象类作为纯接口和完全实现的类之间的一种中间状态。

当一个类被标记为`abstract`时，这意味着它不能被实例化，因为您表明该类不完整。在可以实例化之前，它需要更多的实现。

例如，`System.IO.Stream`类是抽象的，因为它实现了所有流都需要的常见功能，但它并不完整，因此您不能使用`new Stream()`来实例化它。

让我们比较两种类型的接口和两种类型的类，如下面的代码所示：

```cs

public

接口

INoImplementation

// C# 1.0 及更高版本

{

void

Alpha

()

; // 必须由派生类型实现

}

public

接口

ISomeImplementation

// C# 8.0 及更高版本

{

void

Alpha

()

; // 必须由派生类型实现

void

Beta

()

{

// 默认实现；可以被覆盖

}

}

public

抽象

类

部分实现

// C# 1.0 及更高版本

{

public

抽象

void

Gamma

()

; // 必须由派生类型实现

public

虚拟

void

Delta

()

// 可以被覆盖

{

// 实现

}

}

公共

类

FullyImplemented

: 部分实现

, ISomeImplementation

{

公共

void

阿尔法

()

{

// 实现

}

公共

覆盖

void

伽玛

()

{

// 实现

}

}

// 您只能实例化完全实现的类

FullyImplemented a = 新的

();

// 所有其他类型都会导致编译错误

部分实现 b = 新的

(); // 编译错误！

ISomeImplementation c = 新的

(); // 编译错误！

INoImplementation d = 新的

(); // 编译错误！

```

## 防止继承和覆盖

您可以通过在其定义中应用 `sealed` 关键字来防止另一个开发人员从您的类继承。如下面的代码所示，没有人可以从 Scrooge McDuck 继承：

```cs

公共

封闭

类

ScroogeMcDuck

{

}

```

.NET 中`sealed`的一个例子是`string`类。微软在`string`类内部实现了一些极端的优化，可能会受到您的继承的负面影响，因此微软阻止了这种情况。

您可以通过在方法中应用 `sealed` 关键字防止其他人进一步覆盖 `virtual` 方法。没有人可以改变 Lady Gaga 唱歌的方式，如下面的代码所示：

```cs

使用

静态

System.Console;

命名空间

Packt.Shared

;

公共

类

歌手

{

// 虚拟允许此方法被覆盖

公共

虚拟

void

唱歌

()

{

WriteLine("唱歌..."

);

}

}

公共

类

LadyGaga

: 歌手

{

// 封闭防止在子类中覆盖该方法

公共

封闭

覆盖

void

唱歌

()

{

WriteLine("以风格唱歌..."

);

}

}

```

您只能封闭一个被覆盖的方法。

## 理解多态性

您现在已经看到了两种改变继承方法行为的方式。我们可以使用 `new` 关键字*隐藏*它（称为**非多态继承**），或者我们可以*覆盖*它（称为**多态继承**）。

两种方式都可以使用 `base` 关键字访问基类或超类的成员，那么有什么区别呢？

这完全取决于持有对象引用的变量的类型。例如，`Person` 类型的变量可以持有对 `Person` 类的引用，或者从 `Person` 派生的任何类型的引用。

让我们看看这如何影响您的代码：

1.  在 `Employee.cs` 中，添加语句覆盖 `ToString` 方法，以便将员工的姓名和代码写入控制台，如下面的代码所示：

```cs

公共

覆盖

字符串

ToString

()

{

返回

$"

{Name}

的代码是

{EmployeeCode}

"

;

}

```

1.  在 `Program.cs` 中，编写语句创建一个名为 Alice 的新员工，将其存储在 `Person` 类型的变量中，并调用两个变量的 `WriteToConsole` 和 `ToString` 方法，如下面的代码所示：

```cs

雇员 aliceInEmployee = 新的

()

{ 名字 = "爱丽丝"

, EmployeeCode = "AA123"

};

Person aliceInPerson = aliceInEmployee;

aliceInEmployee.WriteToConsole();

aliceInPerson.WriteToConsole();

WriteLine(aliceInEmployee.ToString());

WriteLine(aliceInPerson.ToString());

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

爱丽丝是在 01/01/01 出生的，雇佣于 01/01/01

爱丽丝是在星期一出生的

Alice 的代码是 AA123

Alice 的代码是 AA123

```

当一个方法被 `new` 隐藏时，编译器不够聪明，不知道对象是 `Employee`，所以调用 `Person` 中的 `WriteToConsole` 方法。

当一个方法被 `virtual` 和 `override` 覆盖时，编译器足够聪明，知道虽然变量声明为 `Person` 类，但对象本身是 `Employee` 类，因此调用 `Employee` 的 `ToString` 实现。

成员修饰符及其影响总结在下表中：

| 变量类型 | 成员修饰符 | 执行的方法 | 在类中 |
| --- | --- | --- | --- |
| `人` |  | `写入控制台` | `人` |
| `雇员` | `新` | `WriteToConsole` | `雇员` |
| `人` | `虚拟` | `ToString` | `雇员` |
| `雇员` | `覆盖` | `ToString` | `雇员` |

在我看来，多态对大多数程序员来说是学术性的。如果你理解这个概念，那很好；但是，如果不理解，我建议你不要担心。有些人喜欢通过说理解多态对所有 C#程序员来说都很重要来让其他人感到自卑，但在我看来并不是这样。

您可以在 C#中有一个成功的职业，而不需要能够解释多态性，就像赛车手不需要能够解释燃油喷射背后的工程一样。

**良好的实践**：您应该尽可能使用`virtual`和`override`而不是`new`来更改继承方法的实现。

# 在继承层次结构内进行转换

类型之间的转换与类型之间的转换略有不同。转换是在相似类型之间进行的，比如在 16 位整数和 32 位整数之间，或者在超类和其子类之间。转换是在不同类型之间进行的，比如在文本和数字之间。

## 隐式转换

在前面的示例中，您看到了如何将派生类型的实例存储在其基类型的变量中（或其基类型的基类型，依此类推）。当我们这样做时，它被称为**隐式转换**。

## 显式转换

另一种方式是显式转换，您必须使用括号将要转换为的类型作为前缀来执行：

1.  在`Program.cs`中，添加一个语句将`aliceInPerson`变量赋值给一个新的`Employee`变量，如下面的代码所示：

```cs

Employee explicitAlice = aliceInPerson;

```

1.  您的编码工具显示了一个红色的波浪线和一个编译错误，如*图 6.5*所示：![图形用户界面，文本，应用程序，电子邮件，网站描述自动生成](img/Image00073.jpg)

图 6.5：缺少显式转换编译错误

1.  更改语句，将分配的变量名前缀为`Employee`类型的转换，如下面的代码所示：

```cs

Employee explicitAlice = (Employee)aliceInPerson;

```

## 避免转换异常

现在编译器很高兴了；但是，因为`aliceInPerson`可能是一个不同的派生类型，比如`学生`而不是`员工`，我们需要小心。在一个更复杂的代码的真实应用中，这个变量的当前值可能已经被设置为`学生`实例，然后这个语句会抛出`InvalidCastException`错误。

我们可以通过编写`try`语句来处理这个问题，但有一个更好的方法。我们可以使用`is`关键字来检查对象的类型：

1.  将显式转换语句包装在`if`语句中，如下面的代码中所突出显示的那样：

```cs

**if**

**(aliceInPerson**

**是**

**员工)**

**{**

**WriteLine(**

**$"**

**{**

**nameof**

**(aliceInPerson)}**

**是员工"**

**);**

Employee explicitAlice = (Employee)aliceInPerson;

**// 安全地对 explicitAlice 进行一些操作**

**}**

```

1.  运行代码并查看结果，如下所示：

```cs

aliceInPerson 是员工

```

您可以进一步简化代码，使用声明模式，这将避免需要执行显式转换，如下面的代码所示：

```cs

if

(aliceInPerson is

Employee explicitAlice)

{

WriteLine($"

{**

nameof

(aliceInPerson)}

是员工"

);

// 安全地对 explicitAlice 进行一些操作

}

```

或者，您可以使用`as`关键字进行转换。与抛出异常不同，`as`关键字在无法转换类型时返回`null`。

1.  在`Main`中，添加使用`as`关键字对 Alice 进行转换然后检查返回值是否不为空的语句，如下面的代码所示：

```cs

Employee? aliceAsEmployee = aliceInPerson as

员工；//可能为空

if

(aliceAsEmployee != null

)

{

WriteLine($"

{

nameof

(aliceInPerson)}

作为员工"

);

// 安全地对 aliceAsEmployee 进行一些操作

}

```

由于访问`null`变量的成员将抛出`NullReferenceException`错误，因此在使用结果之前，您应该始终检查`null`。

1.  运行代码并查看结果，如下所示：

```cs

aliceInPerson 作为员工

```

如果你想在 Alice 不是员工时执行一系列语句怎么办？

在过去，您将不得不使用`！`（非）运算符，如下面的代码所示：

```cs

如果

（！（aliceInPerson 是

员工）

```

使用 C# 9 及更高版本，您可以使用`not`关键字，如下面的代码所示：

```cs

如果

（aliceInPerson 是

不

员工）

```

**良好实践**：使用`is`和`as`关键字在派生类型之间进行强制转换时避免抛出异常。如果不这样做，您必须为`InvalidCastException`编写`try`-`catch`语句。

# 继承和扩展.NET 类型

.NET 有预构建的类库，包含数十万种类型。您通常可以通过从 Microsoft 的类型中派生来继承其部分或全部行为，然后重写或扩展它，而不是创建自己完全新的类型。

## 继承异常

作为继承的一个例子，我们将派生一个新的异常类型：

1.  在`PacktLibrary`项目中，添加一个名为`PersonException.cs`的新类文件。

1.  修改文件的内容，定义一个名为`PersonException`的类，有三个构造函数，如下面的代码所示：

```cs

命名空间

Packt.Shared

;

公共

类

PersonException

：异常

{

公共

PersonException

（）：

基础

（）

{ }

公共

PersonException

（

字符串

消息

)：

基础

（

消息

）

{ }

公共

PersonException

(

字符串

消息，异常 innerException

）

：

基础

（

消息，innerException

）

{ }

}

```

与普通方法不同，构造函数不会被继承，因此我们必须显式声明并显式调用`System.Exception`中的基础构造函数实现，以使它们对可能想要使用我们自定义异常的程序员可用。

1.  在`Person.cs`中，添加语句来定义一个方法，如果日期/时间参数早于一个人的出生日期，则抛出异常，如下面的代码所示：

```cs

公共

void

时间旅行

（

日期时间

当

）

{

如果

（当

<= 出生日期)

{

抛出

新

PersonException("如果你回到比自己出生日期更早的日期，那么宇宙将爆炸！"

）;

}

其他

{

WriteLine（“欢迎来到

{

当

:yyyy}

！”

）;

}

}

```

1.  在`Program.cs`中，添加语句来测试当员工 John Jones 试图时间旅行太久时会发生什么，如下面的代码所示：

```cs

尝试

{

john.TimeTravel(when

：新

（1999

，12

，31

）;

john.TimeTravel(when

：新

（1950

，12

，25

）;

}

捕获（PersonException ex）

{

WriteLine（ex.Message）;

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

欢迎来到 1999 年！

如果你回到比自己出生日期更早的日期，那么宇宙将爆炸！

```

**良好实践**：在定义自己的异常时，给它们相同的三个构造函数，显式调用内置的构造函数。

## 在无法继承时扩展类型

之前，我们看到`sealed`修饰符可以用来防止继承。

微软已经对`System.String`类应用了`sealed`关键字，以防止任何人继承并潜在地破坏字符串的行为。

我们还能给字符串添加新的方法吗？是的，如果我们使用一种名为**扩展方法**的语言特性，这是在 C# 3.0 中引入的。

### 使用静态方法重用功能

自 C#的第一个版本以来，我们就能够创建`static`方法来重用功能，比如验证一个`string`是否包含电子邮件地址的能力。实现将使用一个正则表达式，您将在*第八章* *使用常见的.NET 类型*中了解更多。

让我们写一些代码：

1.  在`PacktLibrary`项目中，添加一个名为`StringExtensions`的新类，如下面的代码所示，并注意以下内容：

+   该类导入了一个处理正则表达式的命名空间。

+   `IsValidEmail`方法是`static`的，它使用`Regex`类型来检查与一个简单的电子邮件模式的匹配，该模式在`@`符号之前和之后寻找有效字符。

```cs

使用

System.Text.RegularExpressions;

命名空间

Packt.Shared

;

公共

类

StringExtensions

{

公共

静态

布尔

IsValidEmail

(

字符串

输入

)

{

// 使用简单的正则表达式进行检查

// 输入字符串是有效的电子邮件

返回

Regex.IsMatch(input，

@"[a-zA-Z0-9\.-_]+@[a-zA-Z0-9\.-_]+"

);

}

}

```

1.  在`Program.cs`中，添加语句以验证两个示例的电子邮件地址，如下面的代码所示：

```cs

字符串

email1 = "pamela@test.com"

;

字符串

email2 = "ian&test.com"

;

WriteLine("{0}是一个有效的电子邮件地址：{1}"

，

arg0：email1，

arg1：StringExtensions.IsValidEmail(email1));

WriteLine("{0}是一个有效的电子邮件地址：{1}"

，

arg0：email2，

arg1：StringExtensions.IsValidEmail(email2));

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

pamela@test.com 是一个有效的电子邮件地址：True

ian&test.com 是一个有效的电子邮件地址：False

```

这样做是有效的，但扩展方法可以减少我们必须键入的代码量，并简化此函数的使用。

### 使用扩展方法重用功能

将`static`方法转换为扩展方法很容易：

1.  在`StringExtensions.cs`中，在类之前添加`static`修饰符，并在`string`类型之前添加`this`修饰符，如下面的代码所示：

```cs

公共

**静态**

类

StringExtensions

{

公共

静态

布尔

IsValidEmail

(

**this**

字符串

输入

)

{

```

这两个更改告诉编译器应该将该方法视为扩展`string`类型的方法。

1.  在`Program.cs`中，添加语句以使用`string`值的扩展方法，这些值需要检查是否为有效的电子邮件地址，如下面的代码所示：

```cs

WriteLine("{0}是一个有效的电子邮件地址：{1}"

，

arg0：email1，

arg1：email1.IsValidEmail());

WriteLine("{0}是一个有效的电子邮件地址：{1}"

，

arg0：email2，

arg1：email2.IsValidEmail());

```

请注意调用`IsValidEmail`方法的语法上的微妙简化。旧的、更长的语法仍然有效。

1.  `IsValidEmail`扩展方法现在看起来就像`string`类型的所有实际实例方法一样，例如`IsNormalized`和`Insert`，如*图 6.6*所示：![图形用户界面，文本，应用程序描述自动生成](img/Image00074.jpg)

图 6.6：扩展方法出现在 IntelliSense 中，与实例方法并列

1.  运行代码并查看结果，结果将与以前相同。

**良好实践**：扩展方法不能替换或覆盖现有的实例方法。例如，您不能重新定义`Insert`方法。扩展方法将出现为 IntelliSense 中的重载，但实例方法将优先于具有相同名称和签名的扩展方法。

尽管扩展方法可能看起来并没有带来很大的好处，在*第十一章*，*使用 LINQ 查询和操作数据*中，您将看到一些极其强大的扩展方法的用法。

# 使用分析器编写更好的代码

.NET 分析器会发现潜在问题并为其提供修复建议。**StyleCop**是一个常用的分析器，可帮助您编写更好的 C#代码。

让我们看看它的作用，建议如何改进针对.NET 5.0 的控制台应用程序的项目模板，以便控制台应用程序已经具有`Program`类和`Main`方法：

1.  使用您喜欢的代码编辑器添加控制台应用程序项目，如下列表所示：

1.  项目模板：**控制台应用程序** / `console -f net5.0`

1.  工作区/解决方案文件和文件夹：`Chapter06`

1.  项目文件和文件夹：`CodeAnalyzing`

1.  目标框架：**.NET 5.0 (当前)**

1.  在`CodeAnalyzing`项目中，添加一个`StyleCop.Analyzers`的包引用。

1.  向项目添加一个名为`stylecop.json`的 JSON 文件，用于控制 StyleCop 设置。

1.  修改其内容，如下标记所示：

```cs

{

"$schema"

: "https://raw.githubusercontent.com/DotNetAnalyzers/StyleCopAnalyzers/master/StyleCop.Analyzers/StyleCop.Analyzers/Settings/stylecop.schema.json"

,

"settings"

: {

}

}

```

`$schema`条目使您在代码编辑器中编辑`stylecop.json`文件时启用智能感知。

1.  编辑项目文件，将目标框架更改为`net6.0`，添加条目来配置名为`stylecop.json`的文件不包含在发布部署中，并在开发过程中将其作为附加文件进行处理，如下标记所示：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

<OutputType>Exe</OutputType>

<TargetFramework>net6.0

</TargetFramework>

</PropertyGroup>

**<ItemGroup>**

**<None Remove=**

**"stylecop.json"**

**/>**

**</ItemGroup>**

**<ItemGroup>**

**<AdditionalFiles Include=**

**"stylecop.json"**

**/>**

**</ItemGroup>**

<ItemGroup>

<PackageReference Include="StyleCop.Analyzers"

Version="1.2.0-*"

<PrivateAssets>all</PrivateAssets>

<IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>

</PackageReference>

</ItemGroup>

</Project>

```

1.  构建您的项目。

1.  你会看到它认为有问题的所有警告，如*图 6.7*所示：![](img/Image00075.jpg)

图 6.7：StyleCop 代码分析器警告

1.  例如，它希望`using`指令放在命名空间声明内，如下面的输出所示：

```cs

C:\Code\Chapter06\CodeAnalyzing\Program.cs(1,1): warning SA1200: Using directive should appear within a namespace declaration [C:\Code\Chapter06\CodeAnalyzing\CodeAnalyzing.csproj]

```

## 抑制警告

要抑制警告，您有几个选项，包括添加代码和设置配置。

要抑制使用属性，如下面的代码所示：

```cs

[assembly:SuppressMessage(

"StyleCop.CSharp.OrderingRules"

,

"SA1200:UsingDirectivesMustBePlacedWithinNamespace"

, Justification =

"Reviewed."

)

]

```

要抑制使用指令，如下面的代码所示：

```cs

#

pragma

warning

disable SA1200 // UsingDirectivesMustBePlacedWithinNamespace

using

System;

#

pragma

warning

restore SA1200 // UsingDirectivesMustBePlacedWithinNamespace

```

让我们通过修改`stylecop.json`文件来抑制警告：

1.  在 `stylecop.json` 中，添加一个配置选项来设置`using`语句允许在命名空间外部，如下标记所示：

```cs

{

"$schema"

: "https://raw.githubusercontent.com/DotNetAnalyzers/StyleCopAnalyzers/master/StyleCop.Analyzers/StyleCop.Analyzers/Settings/stylecop.schema.json"

,

"settings"

: {

"orderingRules"

: {

"usingDirectivesPlacement"

: "outsideNamespace"

}

}

}

```

1.  构建项目并注意警告 SA1200 已经消失。

1.  在 `stylecop.json` 中，将使用指令放置设置为`preserve`，这允许`using`语句在命名空间内外都可以使用，如下标记所示：

```cs

"orderingRules"

: {

"usingDirectivesPlacement"

: "preserve"

}

```

### 修复代码

现在，让我们修复所有其他警告：

1.  在 `CodeAnalyzing.csproj` 中，添加一个元素来自动生成一个文档的 XML 文件，如下标记所示：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

<OutputType>Exe</OutputType>

<TargetFramework>net6.0

</TargetFramework>

**<GenerateDocumentationFile>**

**true**

**</GenerateDocumentationFile>**

</PropertyGroup>

```

1.  在 `stylecop.json` 中，添加一个配置选项来为公司名称和版权文本提供值，如下标记所示：

```cs

{

"$schema"

: "https://raw.githubusercontent.com/DotNetAnalyzers/StyleCopAnalyzers/master/StyleCop.Analyzers/StyleCop.Analyzers/Settings/stylecop.schema.json"

,

"settings"

: {

"orderingRules"

: {

"usingDirectivesPlacement"

: "preserve"

},

**"documentationRules"**

**: {**

**"companyName"**

**:**

**"Packt"**

**,**

**"copyrightText"**

**:**

**"Copyright (c) Packt. All rights reserved."**

**}**

}

}

```

1.  在`Program.cs`中，添加公司和版权文本的文件头注释，将`using System;`声明移动到命名空间内，并为类和方法设置显式访问修饰符和 XML 注释，如下面的代码所示：

```cs

// <版权文件="Program.cs"公司="Packt">

// 版权所有（c）Packt。保留所有权利。

// </版权>

命名空间

代码分析

{

使用

系统;

///

<摘要>

///

此控制台应用程序的主类。

///

</摘要>

公共

类

程序

{

///

<摘要>

///

此控制台应用程序的主入口点。

///

</摘要>

///

<param name="args">

传递给控制台应用程序的参数的字符串数组。

</参数>

公共

静态的

空

主要的

(

字符串

[] args

)

{

Console.WriteLine("Hello World!"

);

}

}

}

```

1.  构建项目。

1.  展开`bin/Debug/net6.0`文件夹，并注意自动生成的名为`CodeAnalyzing.xml`的文件，如下标记所示：

```cs

<?xml version="1.0"?>

<

文档

<

程序集

<

名称

代码分析</

名称

</

程序集

<

成员

<

成员

名称

=

"T:CodeAnalyzing.Program"

<

摘要

此控制台应用程序的主类。

</

摘要

</

成员

<

成员

名称

=

"M:CodeAnalyzing.Program.Main(System.String[])"

<

摘要

此控制台应用程序的主入口点。

</

摘要

<

参数

名称

=

"args"

传递给控制台应用程序的参数的字符串数组。</

参数

</

成员

</

成员

</

文档

```

### 理解常见的 StyleCop 建议

在代码文件中，应按照以下列表中显示的顺序排列内容：

1.  外部别名指令

1.  使用指令

1.  命名空间

1.  委托

1.  枚举

1.  接口

1.  结构体

1.  类

在类、记录、结构体或接口中，应按照以下列表中显示的顺序排列内容：

1.  字段

1.  构造函数

1.  析构函数（终结器）

1.  委托

1.  事件

1.  枚举

1.  接口

1.  属性

1.  索引器

1.  方法

1.  结构体

1.  嵌套类和记录

**良好实践**：您可以在以下链接了解所有 StyleCop 规则：[`github.com/DotNetAnalyzers/StyleCopAnalyzers/blob/master/DOCUMENTATION.md`](https://github.com/DotNetAnalyzers/StyleCopAnalyzers/blob/master/DOCUMENTATION.md) 。

# 练习和探索

通过回答一些问题来测试您的知识和理解。进行一些动手实践，并通过更深入的研究探索本章的主题。

## 练习 6.1-测试你的知识

回答以下问题：

1.  什么是委托？

1.  什么是事件？

1.  基类和派生类之间有什么关系，派生类如何访问基类？

1.  `is`和`as`运算符之间有什么区别？

1.  哪个关键字用于防止类被派生或方法被进一步覆盖？

1.  哪个关键字用于防止使用`new`关键字实例化类？

1.  哪个关键字用于允许成员被覆盖？

1.  析构函数和解构方法有什么区别？

1.  所有异常应该具有的构造函数签名是什么？

1.  什么是扩展方法，如何定义一个？

## 练习 6.2-练习创建继承层次结构

通过以下步骤探索继承层次结构：

1.  将一个名为`Exercise02`的新控制台应用程序添加到您的`Chapter06`解决方案/工作区中。

1.  创建一个名为`Shape`的类，其中包含名为`Height`，`Width`和`Area`的属性。

1.  添加三个从中派生的类-`矩形`，`正方形`和`圆`-以及您认为合适的任何其他成员，并正确覆盖和实现`Area`属性。

1.  在`Main`中，添加语句以创建每种形状的一个实例，如下面的代码所示：

```cs

矩形 r = 新的

（高度：3

，宽度：4.5

);

WriteLine($"矩形 H:

{r.Height}

，W:

{r.Width}

，面积：

{r.Area}

"

);

正方形 s = 新的

（5

);

WriteLine($"正方形 H:

{s.Height}

，W：

{s.Width}

，面积：

{s.Area}

"

);

圆形 c = 新的

（半径：2.5

);

WriteLine($"圆 H:

{c.Height}

，W：

{c.Width}

，面积：

{c.Area}

"

);

```

1.  运行控制台应用程序，并确保结果看起来像以下输出：

```cs

矩形 H: 3，W: 4.5，面积：13.5

正方形 H: 5，W: 5，面积：25

圆形 H: 5，W: 5，面积：19.6349540849362

```

## 练习 6.3 – 探索主题

使用以下页面上的链接了解本章涵盖的主题：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-6---implementing-interfaces-and-inheriting-classes`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-6---implementing-interfaces-and-inheriting-classes)

# 摘要

在本章中，您学习了关于本地函数和运算符、委托和事件、实现接口、泛型和使用继承和面向对象编程派生类型。您还学习了关于基类和派生类，以及如何重写类型成员，使用多态性和类型之间的转换。

在下一章中，您将学习.NET 如何打包和部署，并在随后的章节中，它为您提供的类型，以实现诸如文件处理、数据库访问、加密和多任务处理等常见功能。
