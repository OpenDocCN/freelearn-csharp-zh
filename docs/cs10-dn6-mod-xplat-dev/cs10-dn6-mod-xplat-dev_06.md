# 第六章：实现接口和继承类

本章是关于使用**面向对象编程**（**OOP**）从现有类型派生新类型的。你将学习定义运算符和局部函数以执行简单操作，以及委托和事件以在类型之间交换消息。你将实现接口以实现通用功能。你将了解泛型以及引用类型和值类型之间的区别。你将创建一个派生类以从基类继承功能，覆盖继承的类型成员，并使用多态性。最后，你将学习如何创建扩展方法以及如何在继承层次结构中的类之间进行类型转换。

本章涵盖以下主题：

+   设置类库和控制台应用程序

+   更多关于方法的内容

+   引发和处理事件

+   使用泛型安全地重用类型

+   实现接口

+   使用引用和值类型管理内存

+   处理空值

+   从类继承

+   在继承层次结构中进行类型转换

+   继承和扩展.NET 类型

+   使用分析器编写更好的代码

# 设置类库和控制台应用程序

我们将首先定义一个包含两个项目的工作区/解决方案，类似于在*第五章*，*使用面向对象编程构建自己的类型*中创建的那个。即使你完成了该章的所有练习，也要按照下面的说明操作，因为我们将在类库中使用 C# 10 特性，因此它需要面向.NET 6.0 而不是.NET Standard 2.0：

1.  使用你喜欢的编码工具创建一个名为`Chapter06`的新工作区/解决方案。

1.  添加一个类库项目，如下列表定义：

    1.  项目模板：**类库** / `classlib`

    1.  工作区/解决方案文件和文件夹：`Chapter06`

    1.  项目文件和文件夹：`PacktLibrary`

1.  添加一个控制台应用程序项目，如下列表定义：

    1.  项目模板：**控制台应用程序** / `console`

    1.  工作区/解决方案文件和文件夹：`Chapter06`

    1.  项目文件和文件夹：`PeopleApp`

1.  在`PacktLibrary`项目中，将名为`Class1.cs`的文件重命名为`Person.cs`。

1.  修改`Person.cs`文件内容，如下所示：

    ```cs
    using static System.Console;
    namespace Packt.Shared;
    public class Person : object
    {
      // fields
      public string? Name;    // ? allows null
      public DateTime DateOfBirth;
      public List<Person> Children = new(); // C# 9 or later
      // methods
      public void WriteToConsole() 
      {
        WriteLine($"{Name} was born on a {DateOfBirth:dddd}.");
      }
    } 
    ```

1.  在`PeopleApp`项目中，添加对`PacktLibrary`的项目引用，如以下标记中突出显示的那样：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
      </PropertyGroup>
     **<ItemGroup>**
     **<ProjectReference**
     **Include=****"..\PacktLibrary\PacktLibrary.csproj"** **/>**
     **</ItemGroup>**
    </Project> 
    ```

1.  构建`PeopleApp`项目并注意输出，表明两个项目都已成功构建。

# 更多关于方法的内容

我们可能希望两个`Person`实例能够繁殖。我们可以通过编写方法来实现这一点。实例方法是对象对自己执行的操作；静态方法是类型执行的操作。

选择哪种方式取决于哪种对行动最有意义。

**最佳实践**：同时拥有静态方法和实例方法来执行类似操作通常是有意义的。例如，`string`类型既有`Compare`静态方法，也有`CompareTo`实例方法。这使得使用你的类型的程序员能够选择如何使用这些功能，为他们提供了更多的灵活性。

## 通过方法实现功能

让我们先通过使用静态和实例方法来实现一些功能：

1.  向`Person`类添加一个实例方法和一个静态方法，这将允许两个`Person`对象繁衍后代，如下面的代码所示：

    ```cs
    // static method to "multiply"
    public static Person Procreate(Person p1, Person p2)
    {
      Person baby = new()
      {
        Name = $"Baby of {p1.Name} and {p2.Name}"
      };
      p1.Children.Add(baby);
      p2.Children.Add(baby);
      return baby;
    }
    // instance method to "multiply"
    public Person ProcreateWith(Person partner)
    {
      return Procreate(this, partner);
    } 
    ```

    注意以下内容：

    +   在名为`Procreate`的`static`方法中，要繁衍后代的`Person`对象作为参数`p1`和`p2`传递。

    +   一个新的`Person`类名为`baby`，其名字由繁衍后代的两个人的名字组合而成。这可以通过设置返回的`baby`变量的`Name`属性来稍后更改。

    +   `baby`对象被添加到两个父母的`Children`集合中，然后返回。类是引用类型，意味着在内存中存储的`baby`对象的引用被添加，而不是`baby`对象的克隆。你将在本章后面学习引用类型和值类型之间的区别。

    +   在名为`ProcreateWith`的实例方法中，要与之繁衍后代的`Person`对象作为参数`partner`传递，它与`this`一起被传递给静态`Procreate`方法以重用方法实现。`this`是一个关键字，它引用当前类的实例。

    **最佳实践**：创建新对象或修改现有对象的方法应返回对该对象的引用，以便调用者可以访问结果。

1.  在`PeopleApp`项目中，在`Program.cs`文件的顶部，删除注释并导入我们的`Person`类和静态导入`Console`类型，如下面的代码所示：

    ```cs
    using Packt.Shared;
    using static System.Console; 
    ```

1.  在`Program.cs`中，创建三个人并让他们相互繁衍后代，注意要在`string`中添加双引号字符，你必须在其前面加上反斜杠字符，如下所示，`\"`，如下面的代码所示：

    ```cs
    Person harry = new() { Name = "Harry" }; 
    Person mary = new() { Name = "Mary" }; 
    Person jill = new() { Name = "Jill" };
    // call instance method
    Person baby1 = mary.ProcreateWith(harry); 
    baby1.Name = "Gary";
    // call static method
    Person baby2 = Person.Procreate(harry, jill);
    WriteLine($"{harry.Name} has {harry.Children.Count} children."); 
    WriteLine($"{mary.Name} has {mary.Children.Count} children."); 
    WriteLine($"{jill.Name} has {jill.Children.Count} children."); 
    WriteLine(
      format: "{0}'s first child is named \"{1}\".",
      arg0: harry.Name,
      arg1: harry.Children[0].Name); 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Harry has 2 children. 
    Mary has 1 children. 
    Jill has 1 children.
    Harry's first child is named "Gary". 
    ```

## 通过运算符实现功能

`System.String`类有一个名为`Concat`的`static`方法，它将两个字符串值连接起来并返回结果，如下面的代码所示：

```cs
string s1 = "Hello "; 
string s2 = "World!";
string s3 = string.Concat(s1, s2); 
WriteLine(s3); // Hello World! 
```

调用像`Concat`这样的方法是可以的，但对程序员来说，使用`+`符号运算符将两个`string`值“相加”可能更自然，如下面的代码所示：

```cs
string s3 = s1 + s2; 
```

一句广为人知的圣经格言是*去繁衍后代*，意指生育。让我们编写代码，使得`*`（乘法）符号能让两个`Person`对象繁衍后代。

我们通过为`*`符号定义一个`static`运算符来实现这一点。语法类似于方法，因为实际上，运算符*就是*一个方法，但使用符号代替方法名，使得语法更为简洁。

1.  在`Person.cs`中，创建一个`static`运算符用于`*`符号，如下所示：

    ```cs
    // operator to "multiply"
    public static Person operator *(Person p1, Person p2)
    {
      return Person.Procreate(p1, p2);
    } 
    ```

    **良好实践**：与方法不同，运算符不会出现在类型的 IntelliSense 列表中。对于您定义的每个运算符，都应同时创建一个方法，因为程序员可能不清楚该运算符可用。运算符的实现可以调用该方法，重用您编写的代码。提供方法的第二个原因是并非所有语言编译器都支持运算符；例如，尽管 Visual Basic 和 F#支持诸如*之类的算术运算符，但没有要求其他语言支持 C#支持的所有运算符。

1.  在`Program.cs`中，在调用`Procreate`方法和向控制台写入语句之前，使用`*`运算符再制造一个婴儿，如下所示：

    ```cs
    // call static method
    Person baby2 = Person.Procreate(harry, jill);
    **// call an operator**
    **Person baby3 = harry * mary;** 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Harry has 3 children. 
    Mary has 2 children. 
    Jill has 1 children.
    Harry's first child is named "Gary". 
    ```

## 使用局部函数实现功能

C# 7.0 引入的一个语言特性是能够定义**局部函数**。

局部函数相当于方法中的局部变量。换句话说，它们是仅在其定义的包含方法内部可访问的方法。在其他语言中，它们有时被称为**嵌套**或**内部函数**。

局部函数可以在方法内的任何位置定义：顶部、底部，甚至中间的某个位置！

我们将使用局部函数来实现阶乘计算：

1.  在`Person.cs`中，添加语句以定义一个`Factorial`函数，该函数在其内部使用局部函数来计算结果，如下所示：

    ```cs
    // method with a local function
    public static int Factorial(int number)
    {
      if (number < 0)
      {
        throw new ArgumentException(
          $"{nameof(number)} cannot be less than zero.");
      }
      return localFactorial(number);
      int localFactorial(int localNumber) // local function
      {
        if (localNumber < 1) return 1;
        return localNumber * localFactorial(localNumber - 1);
      }
    } 
    ```

1.  在`Program.cs`中，添加一条语句以调用`Factorial`函数并将返回值写入控制台，如下所示：

    ```cs
    WriteLine($"5! is {Person.Factorial(5)}"); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    5! is 120 
    ```

# 引发和处理事件

方法通常被描述为*对象可以执行的动作，无论是对自己还是对相关对象*。例如，`List<T>`可以向自身添加项目或清除自身，而`File`可以在文件系统中创建或删除文件。

事件通常被描述为*发生在对象上的动作*。例如，在用户界面中，`Button`有一个`Click`事件，点击是发生在按钮上的事情，而`FileSystemWatcher`监听文件系统的更改通知并引发`Created`和`Deleted`等事件，这些事件在目录或文件更改时触发。

另一种思考事件的方式是，它们提供了一种在两个对象之间交换消息的方法。

事件基于**委托**构建，因此让我们先了解一下委托是什么以及它们如何工作。

## 使用委托调用方法

你已经看到了调用或执行方法的最常见方式：使用 `.` 运算符通过其名称访问该方法。例如，`Console.WriteLine` 告诉 `Console` 类型访问其 `WriteLine` 方法。

调用或执行方法的另一种方式是使用委托。如果你使用过支持**函数指针**的语言，那么可以将委托视为**类型安全的方法指针**。

换句话说，委托包含与委托具有相同签名的方法的内存地址，以便可以安全地使用正确的参数类型调用它。

例如，假设 `Person` 类中有一个方法，它必须接受一个 `string` 类型的唯一参数，并返回一个 `int` 类型，如下所示：

```cs
public int MethodIWantToCall(string input)
{
  return input.Length; // it doesn't matter what the method does
} 
```

我可以在名为 `p1` 的 `Person` 实例上调用此方法，如下所示：

```cs
int answer = p1.MethodIWantToCall("Frog"); 
```

或者，我可以定义一个与签名匹配的委托来间接调用该方法。请注意，参数的名称不必匹配。只有参数类型和返回值必须匹配，如下所示：

```cs
delegate int DelegateWithMatchingSignature(string s); 
```

现在，我可以创建一个委托实例，将其指向该方法，最后，调用该委托（即调用该方法），如下所示：

```cs
// create a delegate instance that points to the method
DelegateWithMatchingSignature d = new(p1.MethodIWantToCall);
// call the delegate, which calls the method
int answer2 = d("Frog"); 
```

你可能会想，“这有什么意义？”嗯，它提供了灵活性。

例如，我们可以使用委托来创建一个方法队列，这些方法需要按顺序调用。在服务中排队执行操作以提供更好的可扩展性是很常见的。

另一个例子是允许多个操作并行执行。委托内置支持异步操作，这些操作在不同的线程上运行，并且可以提供更好的响应性。你将在*第十二章*，*使用多任务提高性能和可扩展性*中学习如何做到这一点。

最重要的例子是，委托允许我们实现事件，以便在不需要相互了解的不同对象之间发送消息。事件是组件之间松散耦合的一个例子，因为组件不需要了解彼此，它们只需要知道事件签名。

委托和事件是 C# 中最令人困惑的两个特性，可能需要几次尝试才能理解，所以如果你感到迷茫，不要担心！

## 定义和处理委托

Microsoft 为事件提供了两个预定义的委托，其签名简单而灵活，如下所示：

```cs
public delegate void EventHandler(
  object? sender, EventArgs e);
public delegate void EventHandler<TEventArgs>(
  object? sender, TEventArgs e); 
```

**最佳实践**：当你想在自己的类型中定义一个事件时，你应该使用这两个预定义委托之一。

让我们来探索委托和事件：

1.  向 `Person` 类添加语句，并注意以下几点，如下所示：

    +   它定义了一个名为 `Shout` 的 `EventHandler` 委托字段。

    +   它定义了一个 `int` 字段来存储 `AngerLevel`。

    +   它定义了一个名为 `Poke` 的方法。

    +   每次有人被戳时，他们的`AngerLevel`都会增加。一旦他们的`AngerLevel`达到三，他们就会引发`Shout`事件，但前提是至少有一个事件委托指向代码中其他地方定义的方法；也就是说，它不是`null`：

    ```cs
    // delegate field
    public EventHandler? Shout;
    // data field
    public int AngerLevel;
    // method
    public void Poke()
    {
      AngerLevel++;
      if (AngerLevel >= 3)
      {
        // if something is listening...
        if (Shout != null)
        {
          // ...then call the delegate
          Shout(this, EventArgs.Empty);
        }
      }
    } 
    ```

    在调用其方法之前检查对象是否不为`null`是非常常见的。C# 6.0 及更高版本允许使用`?`符号在`.`运算符之前简化内联的`null`检查，如以下代码所示：

    ```cs
    Shout?.Invoke(this, EventArgs.Empty); 
    ```

1.  在`Program.cs`底部，添加一个具有匹配签名的方法，该方法从`sender`参数获取`Person`对象的引用，并输出有关他们的信息，如以下代码所示：

    ```cs
    static void Harry_Shout(object? sender, EventArgs e)
    {
      if (sender is null) return;
      Person p = (Person)sender;
      WriteLine($"{p.Name} is this angry: {p.AngerLevel}.");
    } 
    ```

    微软对于处理事件的方法命名的约定是`对象名 _ 事件名`。

1.  在`Program.cs`中，添加一条语句，将方法分配给委托字段，如以下代码所示：

    ```cs
    harry.Shout = Harry_Shout; 
    ```

1.  在将方法分配给`Shout`事件后，添加语句调用`Poke`方法四次，如以下突出显示的代码所示：

    ```cs
    harry.Shout = Harry_Shout;
    **harry.Poke();**
    **harry.Poke();**
    **harry.Poke();**
    **harry.Poke();** 
    ```

1.  运行代码并查看结果，注意哈利在前两次被戳时什么也没说，只有在被戳至少三次后才足够生气以至于大喊，如以下输出所示：

    ```cs
    Harry is this angry: 3\. 
    Harry is this angry: 4. 
    ```

## 定义和处理事件

你现在看到了委托如何实现事件最重要的功能：定义一个方法签名，该签名可以由完全不同的代码块实现，然后调用该方法以及连接到委托字段的其他任何方法。

那么事件呢？它们可能比你想象的要简单。

在将方法分配给委托字段时，不应使用我们在前述示例中使用的简单赋值运算符。

委托是多播的，这意味着你可以将多个委托分配给单个委托字段。我们本可以使用`+=`运算符而不是`=`赋值，这样我们就可以向同一个委托字段添加更多方法。当委托被调用时，所有分配的方法都会被调用，尽管你无法控制它们被调用的顺序。

如果`Shout`委托字段已经引用了一个或多个方法，通过分配一个方法，它将替换所有其他方法。对于用于事件的委托，我们通常希望确保程序员仅使用`+=`运算符或`-=`运算符来分配和移除方法：

1.  为了强制执行这一点，在`Person.cs`中，将`event`关键字添加到委托字段声明中，如以下突出显示的代码所示：

    ```cs
    public **event** EventHandler? Shout; 
    ```

1.  构建`PeopleApp`项目，并注意编译器错误消息，如以下输出所示：

    ```cs
    Program.cs(41,13): error CS0079: The event 'Person.Shout' can only appear on the left hand side of += or -= 
    ```

    这就是`event`关键字所做的（几乎）所有事情！如果你永远不会将一个以上的方法分配给委托字段，那么从技术上讲，你不需要“事件”，但仍然是一种良好的实践，表明你的意图，并期望委托字段被用作事件。

1.  将方法赋值修改为使用`+=`，如下列代码所示：

    ```cs
    harry.Shout += Harry_Shout; 
    ```

1.  运行代码并注意它具有与之前相同的行为。

# 通过泛型安全地重用类型

2005 年，随着 C# 2.0 和.NET Framework 2.0 的推出，微软引入了一项名为**泛型**的功能，它使你的类型能更安全地重用且更高效。它通过允许程序员传递类型作为参数来实现这一点，类似于你可以传递对象作为参数的方式。

## 使用非泛型类型

首先，让我们看一个使用非泛型类型的例子，以便你能理解泛型旨在解决的问题，例如弱类型参数和值，以及使用`System.Object`导致性能问题。

`System.Collections.Hashtable`可用于存储多个值，每个值都有一个唯一键，稍后可用于快速查找其值。键和值都可以是任何对象，因为它们被声明为`System.Object`。虽然这为存储整数等值类型提供了灵活性，但它速度慢，且更容易引入错误，因为添加项时不会进行类型检查。

让我们写一些代码：

1.  在`Program.cs`中，创建一个非泛型集合`System.Collections.Hashtable`的实例，然后添加四个项，如下列代码所示：

    ```cs
    // non-generic lookup collection
    System.Collections.Hashtable lookupObject = new();
    lookupObject.Add(key: 1, value: "Alpha");
    lookupObject.Add(key: 2, value: "Beta");
    lookupObject.Add(key: 3, value: "Gamma");
    lookupObject.Add(key: harry, value: "Delta"); 
    ```

1.  添加语句定义一个值为`2`的`key`，并使用它在哈希表中查找其值，如下列代码所示：

    ```cs
    int key = 2; // lookup the value that has 2 as its key
    WriteLine(format: "Key {0} has value: {1}",
      arg0: key,
      arg1: lookupObject[key]); 
    ```

1.  添加语句使用`harry`对象查找其值，如下列代码所示：

    ```cs
    // lookup the value that has harry as its key
    WriteLine(format: "Key {0} has value: {1}",
      arg0: harry,
      arg1: lookupObject[harry]); 
    ```

1.  运行代码并注意它按预期工作，如下列输出所示：

    ```cs
    Key 2 has value: Beta
    Key Packt.Shared.Person has value: Delta 
    ```

尽管代码能运行，但存在出错的可能性，因为实际上任何类型都可以用作键或值。如果其他开发人员使用了你的查找对象，并期望所有项都是特定类型，他们可能会将其强制转换为该类型，并因某些值可能为不同类型而引发异常。包含大量项的查找对象也会导致性能不佳。

**良好实践**：避免使用`System.Collections`命名空间中的类型。

## 使用泛型类型

`System.Collections.Generic.Dictionary<TKey, TValue>`可用于存储多个值，每个值都有一个唯一键，稍后可用于快速查找其值。键和值可以是任何对象，但你必须在首次实例化集合时告诉编译器键和值的类型。你通过在尖括号`<>`中指定**泛型参数**的类型来实现这一点，即`TKey`和`TValue`。

**良好实践**：当泛型类型有一个可定义的类型时，应将其命名为`T`，例如`List<T>`，其中`T`是列表中存储的类型。当泛型类型有多个可定义的类型时，应使用`T`作为名称前缀，并取一个合理的名称，例如`Dictionary<TKey, TValue>`。

这提供了灵活性，速度更快，且更容易避免错误，因为添加项时会进行类型检查。

让我们编写一些代码，使用泛型来解决问题：

1.  在`Program.cs`中，创建泛型查找集合`Dictionary<TKey, TValue>`的实例，然后添加四个项目，如下面的代码所示：

    ```cs
    // generic lookup collection
    Dictionary<int, string> lookupIntString = new();
    lookupIntString.Add(key: 1, value: "Alpha");
    lookupIntString.Add(key: 2, value: "Beta");
    lookupIntString.Add(key: 3, value: "Gamma");
    lookupIntString.Add(key: harry, value: "Delta"); 
    ```

1.  注意使用`harry`作为键时出现的编译错误，如下面的输出所示：

    ```cs
    /Users/markjprice/Code/Chapter06/PeopleApp/Program.cs(98,32): error CS1503: Argument 1: cannot convert from 'Packt.Shared.Person' to 'int' [/Users/markjprice/Code/Chapter06/PeopleApp/PeopleApp.csproj] 
    ```

1.  将`harry`替换为`4`。

1.  添加语句将`key`设置为`3`，并使用它在字典中查找其值，如下面的代码所示：

    ```cs
    key = 3;
    WriteLine(format: "Key {0} has value: {1}",
      arg0: key,
      arg1: lookupIntString[key]); 
    ```

1.  运行代码并注意它按预期工作，如下面的输出所示：

    ```cs
    Key 3 has value: Gamma 
    ```

# 实现接口

接口是一种将不同类型连接起来以创建新事物的方式。将它们想象成乐高™积木顶部的凸起，使它们能够“粘合”在一起，或者是插头和插座的电气标准。

如果类型实现了接口，那么它就是在向.NET 的其余部分承诺它支持特定的功能。这就是为什么它们有时被描述为合同。

## 常见接口

以下是您的类型可能需要实现的一些常见接口：

| 接口 | 方法 | 描述 |
| --- | --- | --- |
| `IComparable` | `CompareTo(other)` | 这定义了一个比较方法，类型通过该方法实现对其实例的排序。 |
| `IComparer` | `Compare(first, second)` | 这定义了一个比较方法，辅助类型通过该方法实现对主类型实例的排序。 |
| `IDisposable` | `Dispose()` | 这定义了一个处置方法，以更有效地释放非托管资源，而不是等待终结器（有关详细信息，请参阅本章后面的*释放非托管资源*部分）。 |
| `IFormattable` | `ToString(format, culture)` | 这定义了一个文化感知的方法，将对象的值格式化为字符串表示。 |
| `IFormatter` | `Serialize(stream, object)``Deserialize(stream)` | 这定义了将对象转换为字节流以及从字节流转换回对象的方法，用于存储或传输。 |
| `IFormatProvider` | `GetFormat(type)` | 这定义了一个根据语言和区域格式化输入的方法。 |

## 排序时比较对象

您最常想要实现的接口之一是`IComparable`。它有一个名为`CompareTo`的方法。它有两种变体，一种适用于可空`object`类型，另一种适用于可空泛型类型`T`，如下面的代码所示：

```cs
namespace System
{
  public interface IComparable
  {
    int CompareTo(object? obj);
  }
  public interface IComparable<in T>
  {
    int CompareTo(T? other);
  }
} 
```

例如，`string`类型通过返回`-1`（如果`string`小于被比较的`string`）或`1`（如果它更大）来实现`IComparable`。`int`类型通过返回`-1`（如果`int`小于被比较的`int`）或`1`（如果它更大）来实现`IComparable`。

如果类型实现了`IComparable`接口之一，那么数组和集合就可以对其进行排序。

在我们为`Person`类实现`IComparable`接口及其`CompareTo`方法之前，让我们看看当我们尝试对`Person`实例数组进行排序时会发生什么：

1.  在`Program.cs`中，添加语句以创建`Person`实例的数组，并将项目写入控制台，然后尝试对数组进行排序，并将项目再次写入控制台，如下面的代码所示：

    ```cs
    Person[] people =
    {
      new() { Name = "Simon" },
      new() { Name = "Jenny" },
      new() { Name = "Adam" },
      new() { Name = "Richard" }
    };
    WriteLine("Initial list of people:"); 
    foreach (Person p in people)
    {
      WriteLine($"  {p.Name}");
    }
    WriteLine("Use Person's IComparable implementation to sort:");
    Array.Sort(people);
    foreach (Person p in people)
    {
      WriteLine($"  {p.Name}");
    } 
    ```

1.  运行代码，将会抛出异常。正如消息所述，要解决问题，我们的类型必须实现`IComparable`，如下面的输出所示：

    ```cs
    Unhandled Exception: System.InvalidOperationException: Failed to compare two elements in the array. ---> System.ArgumentException: At least one object must implement IComparable. 
    ```

1.  在`Person.cs`中，在继承自`object`之后，添加一个逗号并输入`IComparable<Person>`，如下面的代码所示：

    ```cs
    public class Person : object, IComparable<Person> 
    ```

    你的代码编辑器会在新代码下方画一条红色波浪线，警告你尚未实现承诺的方法。点击灯泡并选择**实现接口**选项，你的代码编辑器可以为你编写骨架实现。

1.  向下滚动至`Person`类的底部，找到为你编写的方法，并删除抛出`NotImplementedException`错误的语句，如以下代码中突出显示的部分所示：

    ```cs
    public int CompareTo(Person? other)
    {
    **throw****new** **NotImplementedException();**
    } 
    ```

1.  添加一条语句以调用`Name`字段的`CompareTo`方法，该方法使用`string`类型的`CompareTo`实现并返回结果，如下面的代码中突出显示的部分所示：

    ```cs
    public int CompareTo(Person? other)
    {
      if (Name is null) return 0;
    **return** **Name.CompareTo(other?.Name);** 
    } 
    ```

    我们选择通过比较`Person`实例的`Name`字段来比较两个`Person`实例。因此，`Person`实例将按其名称的字母顺序排序。为简单起见，我没有在这些示例中添加`null`检查。

1.  运行代码，并注意这次它按预期工作，如下面的输出所示：

    ```cs
    Initial list of people:
      Simon
      Jenny
      Adam
      Richard
    Use Person's IComparable implementation to sort:
      Adam
      Jenny
      Richard
      Simon 
    ```

**最佳实践**：如果有人想要对类型的数组或集合进行排序，那么请实现`IComparable`接口。

## 使用单独的类比较对象

有时，你可能无法访问类型的源代码，并且它可能未实现`IComparable`接口。幸运的是，还有另一种方法可以对类型的实例进行排序。你可以创建一个单独的类型，该类型实现一个略有不同的接口，名为`IComparer`：

1.  在`PacktLibrary`项目中，添加一个名为`PersonComparer.cs`的新类文件，其中包含一个实现`IComparer`接口的类，该接口将比较两个人，即两个`Person`实例。通过比较他们的`Name`字段的长度来实现它，如果名称长度相同，则按字母顺序比较名称，如下面的代码所示：

    ```cs
    namespace Packt.Shared;
    public class PersonComparer : IComparer<Person>
    {
      public int Compare(Person? x, Person? y)
      {
        if (x is null || y is null)
        {
          return 0;
        }
        // Compare the Name lengths...
        int result = x.Name.Length.CompareTo(y.Name.Length);
        // ...if they are equal...
        if (result == 0)
        {
          // ...then compare by the Names...
          return x.Name.CompareTo(y.Name);
        }
        else // result will be -1 or 1
        {
          // ...otherwise compare by the lengths.
          return result; 
        }
      }
    } 
    ```

1.  在`Program.cs`中，添加语句以使用此替代实现对数组进行排序，如下面的代码所示：

    ```cs
    WriteLine("Use PersonComparer's IComparer implementation to sort:"); 
    Array.Sort(people, new PersonComparer());
    foreach (Person p in people)
    {
      WriteLine($"  {p.Name}");
    } 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Use PersonComparer's IComparer implementation to sort:
      Adam
      Jenny
      Simon
      Richard 
    ```

这次，当我们对`people`数组进行排序时，我们明确要求排序算法使用`PersonComparer`类型，以便人们按名字最短的先排序，如 Adam，名字最长的后排序，如 Richard；当两个或多个名字长度相等时，按字母顺序排序，如 Jenny 和 Simon。

## 隐式与显式接口实现

接口可以隐式和显式实现。隐式实现更简单、更常见。只有当类型必须具有具有相同名称和签名的多个方法时，才需要显式实现。

例如，`IGamePlayer`和`IKeyHolder`可能都有一个名为`Lose`的方法，参数相同，因为游戏和钥匙都可能丢失。在必须实现这两个接口的类型中，只能有一个`Lose`方法作为隐式方法。如果两个接口可以共享相同的实现，那很好，但如果不能，则另一个`Lose`方法必须以不同的方式实现并显式调用，如下所示：

```cs
public interface IGamePlayer
{
  void Lose();
}
public interface IKeyHolder
{
  void Lose();
}
public class Person : IGamePlayer, IKeyHolder
{
  public void Lose() // implicit implementation
  {
    // implement losing a key
  }
  void IGamePlayer.Lose() // explicit implementation
  {
    // implement losing a game
  }
}
// calling implicit and explicit implementations of Lose
Person p = new();
p.Lose(); // calls implicit implementation of losing a key
((IGamePlayer)p).Lose(); // calls explicit implementation of losing a game
IGamePlayer player = p as IGamePlayer;
player.Lose(); // calls explicit implementation of losing a game 
```

## 定义具有默认实现的接口

C# 8.0 引入的一项语言特性是接口的**默认实现**。让我们看看它的实际应用：

1.  在`PacktLibrary`项目中，添加一个名为`IPlayable.cs`的新文件。

1.  修改语句以定义一个具有两个方法`Play`和`Pause`的公共`IPlayable`接口，如下所示：

    ```cs
    namespace Packt.Shared;
    public interface IPlayable
    {
      void Play();
      void Pause();
    } 
    ```

1.  在`PacktLibrary`项目中，添加一个名为`DvdPlayer.cs`的新类文件。

1.  修改文件中的语句以实现`IPlayable`接口，如下所示：

    ```cs
    using static System.Console;
    namespace Packt.Shared;
    public class DvdPlayer : IPlayable
    {
      public void Pause()
      {
        WriteLine("DVD player is pausing.");
      }
      public void Play()
      {
        WriteLine("DVD player is playing.");
      }
    } 
    ```

    这很有用，但如果我们决定添加一个名为`Stop`的第三个方法呢？在 C# 8.0 之前，一旦至少有一个类型实现了原始接口，这是不可能的。接口的主要特点之一是它是一个固定的契约。

    C# 8.0 允许接口在发布后添加新成员，只要它们具有默认实现。C#纯粹主义者可能不喜欢这个想法，但由于实用原因，例如避免破坏性更改或不得不定义一个全新的接口，它是有用的，其他语言如 Java 和 Swift 也启用了类似的技术。

    默认接口实现的支持需要对底层平台进行一些根本性的改变，因此只有在目标框架是.NET 5.0 或更高版本、.NET Core 3.0 或更高版本或.NET Standard 2.1 时，它们才受 C#支持。因此，它们不受.NET Framework 的支持。

1.  修改`IPlayable`接口以添加具有默认实现的`Stop`方法，如下所示突出显示：

    ```cs
    **using****static** **System.Console;**
    namespace Packt.Shared;
    public interface IPlayable
    {
      void Play();
      void Pause();
    **void****Stop****()** **// default interface implementation**
     **{**
     **WriteLine(****"Default implementation of Stop."****);**
     **}**
    } 
    ```

1.  构建`PeopleApp`项目并注意，尽管`DvdPlayer`类没有实现`Stop`，但项目仍能成功编译。将来，我们可以通过在`DvdPlayer`类中实现它来覆盖`Stop`的默认实现。

# 使用引用类型和值类型管理内存

我已经多次提到引用类型。让我们更详细地了解一下它们。

内存分为两类：**栈**内存和**堆**内存。在现代操作系统中，栈和堆可以在物理或虚拟内存的任何位置。

栈内存处理速度更快（因为它直接由 CPU 管理，并且采用后进先出机制，更有可能将数据保存在其 L1 或 L2 缓存中），但大小有限；而堆内存较慢，但资源丰富得多。

例如，在 macOS 终端中，我可以输入命令`ulimit -a`来发现栈大小被限制为 8192 KB，而其他内存则是“无限制”的。这种有限的栈内存量使得很容易填满它并导致“栈溢出”。

## 定义引用类型和值类型

定义对象类型时，可以使用三个 C#关键字：`class`、`record`和`struct`。它们都可以拥有相同的成员，如字段和方法。它们之间的一个区别在于内存分配方式。

当你使用`record`或`class`定义类型时，你定义的是**引用类型**。这意味着对象本身的内存是在堆上分配的，而只有对象的内存地址（以及少量开销）存储在栈上。

当你使用`record struct`或`struct`定义类型时，你定义的是**值类型**。这意味着对象本身的内存是在栈上分配的。

如果`struct`使用的字段类型不是`struct`类型，那么这些字段将存储在堆上，这意味着该对象的数据同时存储在栈和堆上！

以下是最常见的结构体类型：

+   **数字** `System` **类型**：`byte`、`sbyte`、`short`、`ushort`、`int`、`uint`、`long`、`ulong`、`float`、`double`和`decimal`

+   **其他** `System` **类型**：`char`、`DateTime`和`bool`

+   `System.Drawing` **类型**：`Color`、`Point`和`Rectangle`

几乎所有其他类型都是`class`类型，包括`string`。

除了类型数据在内存中存储位置的差异外，另一个主要区别是`struct`不支持继承。

## 引用类型和值类型在内存中的存储方式

想象一下，你有一个控制台应用程序，它声明了一些变量，如下面的代码所示：

```cs
int number1 = 49;
long number2 = 12;
System.Drawing.Point location = new(x: 4, y: 5);
Person kevin = new() { Name = "Kevin", 
  DateOfBirth = new(year: 1988, month: 9, day: 23) };
Person sally; 
```

让我们回顾一下执行这些语句时栈和堆上分配的内存，如*图 6.1*所示，并按以下列表描述：

+   `number1`变量是值类型（也称为`struct`），因此它在栈上分配，由于它是 32 位整数，所以占用 4 字节内存。其值 49 直接存储在变量中。

+   `number2`变量也是值类型，因此它也在栈上分配，由于它是 64 位整数，所以占用 8 字节。

+   `location`变量也是值类型，因此它在栈上分配，由于它由两个 32 位整数`x`和`y`组成，所以占用 8 字节。

+   `kevin`变量是引用类型（也称为`class`），因此在栈上分配了 64 位内存地址所需的 8 字节（假设是 64 位操作系统），并在堆上分配了足够字节来存储`Person`实例。

+   `sally`变量是引用类型，因此在 64 位内存地址的栈上分配了 8 字节。目前它为`null`，意味着堆上尚未为其分配内存。

![](img/B17442_06_01.png)

图 6.1：值类型和引用类型在栈和堆上的分配方式

引用类型的所有已分配内存都存储在堆上。如果值类型如`DateTime`被用作引用类型如`Person`的字段，那么`DateTime`值将存储在堆上。

如果值类型有一个引用类型的字段，那么该部分值类型将存储在堆上。`Point`是一个值类型，由两个字段组成，这两个字段本身也是值类型，因此整个对象可以在栈上分配。如果`Point`值类型有一个引用类型的字段，如`string`，那么`string`字节将存储在堆上。

## 类型相等性

通常使用`==`和`!=`运算符比较两个变量。这两个运算符对于引用类型和值类型的行为是不同的。

当你检查两个值类型变量的相等性时，.NET 会直接比较这两个变量在栈上的值，如果它们相等，则返回`true`，如下列代码所示：

```cs
int a = 3;
int b = 3;
WriteLine($"a == b: {(a == b)}"); // true 
```

当你检查两个引用类型变量的相等性时，.NET 会比较这两个变量的内存地址，如果它们相等，则返回`true`，如下列代码所示：

```cs
Person a = new() { Name = "Kevin" };
Person b = new() { Name = "Kevin" };
WriteLine($"a == b: {(a == b)}"); // false 
```

这是因为它们并非同一对象。如果两个变量确实指向堆上的同一对象，那么它们将被视为相等，如下列代码所示：

```cs
Person a = new() { Name = "Kevin" };
Person b = a;
WriteLine($"a == b: {(a == b)}"); // true 
```

此行为的一个例外是`string`类型。它虽是引用类型，但其相等运算符已被重载，使其表现得如同值类型一般，如下列代码所示：

```cs
string a = "Kevin";
string b = "Kevin";
WriteLine($"a == b: {(a == b)}"); // true 
```

你可以对你的类进行类似操作，使相等运算符即使在它们不是同一对象（即堆上同一内存地址）时也返回`true`，只要它们的字段具有相同值即可，但这超出了本书的范围。或者，使用`record class`，因为它们的一个好处是为你实现了这种行为。

## 定义结构类型

让我们来探讨如何定义自己的值类型：

1.  在`PacktLibrary`项目中，添加一个名为`DisplacementVector.cs`的文件。

1.  按照下列代码所示修改文件，并注意以下事项：

    +   该类型使用`struct`声明而非`class`。

    +   它有两个名为`X`和`Y`的`int`字段。

    +   它有一个构造函数，用于设置`X`和`Y`的初始值。

    +   它有一个运算符，用于将两个实例相加，返回一个新实例，其中`X`与`X`相加，`Y`与`Y`相加。

    ```cs
    namespace Packt.Shared;
    public struct DisplacementVector
    {
      public int X;
      public int Y;
      public DisplacementVector(int initialX, int initialY)
      {
        X = initialX;
        Y = initialY;
      }
      public static DisplacementVector operator +(
        DisplacementVector vector1,
        DisplacementVector vector2)
      {
        return new(
          vector1.X + vector2.X,
          vector1.Y + vector2.Y);
      }
    } 
    ```

1.  在`Program.cs`文件中，添加语句以创建两个新的`DisplacementVector`实例，将它们相加，并输出结果，如下列代码所示：

    ```cs
    DisplacementVector dv1 = new(3, 5); 
    DisplacementVector dv2 = new(-2, 7); 
    DisplacementVector dv3 = dv1 + dv2;
    WriteLine($"({dv1.X}, {dv1.Y}) + ({dv2.X}, {dv2.Y}) = ({dv3.X}, {dv3.Y})"); 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    (3, 5) + (-2, 7) = (1, 12) 
    ```

**最佳实践**：如果类型中所有字段占用的总字节数不超过 16 字节，且仅使用值类型作为字段，并且你永远不希望从该类型派生，那么微软建议使用`struct`。如果你的类型使用的堆栈内存超过 16 字节，使用引用类型作为字段，或者可能希望继承它，那么应使用`class`。

## 处理记录结构类型

C# 10 引入了使用`record`关键字与`struct`类型以及`class`类型一起使用的能力。

我们可以定义`DisplacementVector`类型，如下列代码所示：

```cs
public record struct DisplacementVector(int X, int Y); 
```

即使`class`关键字可选，微软仍建议在定义`record class`时明确指定`class`，如下列代码所示：

```cs
public record class ImmutableAnimal(string Name); 
```

## 释放非托管资源

在前一章中，我们了解到构造器可用于初始化字段，且一个类型可以有多个构造器。设想一个构造器分配了一个非托管资源，即不由.NET 控制的任何资源，如操作系统控制下的文件或互斥体。由于.NET 无法使用其自动垃圾回收功能为我们释放这些资源，我们必须手动释放非托管资源。

垃圾回收是一个高级话题，因此对于这个话题，我将展示一些代码示例，但你无需亲自编写代码。

每种类型都可以有一个单一的**终结器**，当资源需要被释放时，.NET 运行时会调用它。终结器的名称与构造器相同，即类型名称，但前面加了一个波浪线`~`。

不要将终结器（也称为**析构器**）与`Deconstruct`方法混淆。析构器释放资源，即它在内存中销毁一个对象。`Deconstruct`方法将对象分解为其组成部分，并使用 C#解构语法，例如在处理元组时：

```cs
public class Animal
{
  public Animal() // constructor
  {
    // allocate any unmanaged resources
  }
  ~Animal() // Finalizer aka destructor
  {
    // deallocate any unmanaged resources
  }
} 
```

前面的代码示例是在处理非托管资源时你应做的最低限度。但仅提供终结器的问题在于，.NET 垃圾回收器需要两次垃圾回收才能完全释放该类型分配的资源。

虽然可选，但建议提供一个方法，让使用你类型的开发者能明确释放资源，以便垃圾回收器可以立即且确定性地释放非托管资源（如文件）的托管部分，并在一次垃圾回收中释放对象的托管内存部分，而不是经过两次垃圾回收。

通过实现`IDisposable`接口，有一个标准机制可以做到这一点，如下例所示：

```cs
public class Animal : IDisposable
{
  public Animal()
  {
    // allocate unmanaged resource
  }
  ~Animal() // Finalizer
  {
    Dispose(false);
  }
  bool disposed = false; // have resources been released?
  public void Dispose()
  {
    Dispose(true);
    // tell garbage collector it does not need to call the finalizer
    GC.SuppressFinalize(this); 
  }
  protected virtual void Dispose(bool disposing)
  {
    if (disposed) return;
    // deallocate the *unmanaged* resource
    // ...
    if (disposing)
    {
      // deallocate any other *managed* resources
      // ...
    }
    disposed = true;
  }
} 
```

存在两个`Dispose`方法，一个`public`，一个`protected`：

+   `public void Dispose`方法将由使用你类型的开发者调用。当被调用时，无论是非托管资源还是托管资源都需要被释放。

+   `protected virtual void Dispose`方法带有一个`bool`参数，内部用于实现资源的释放。它需要检查`disposing`参数和`disposed`字段，因为如果终结器线程已经运行并调用了`~Animal`方法，那么只需要释放非托管资源。

调用`GC.SuppressFinalize(this)`是为了通知垃圾收集器不再需要运行终结器，从而消除了进行第二次垃圾收集的需求。

## 确保 Dispose 方法被调用

当有人使用实现了`IDisposable`的类型时，他们可以使用`using`语句确保调用公共`Dispose`方法，如下列代码所示：

```cs
using (Animal a = new())
{
  // code that uses the Animal instance
} 
```

编译器将你的代码转换成类似下面的形式，这保证了即使发生异常，`Dispose`方法仍然会被调用：

```cs
Animal a = new(); 
try
{
  // code that uses the Animal instance
}
finally
{
  if (a != null) a.Dispose();
} 
```

你将在*第九章*，*文件、流和序列化操作*中看到使用`IDisposable`、`using`语句以及`try`...`finally`块释放非托管资源的实际示例。

# 处理 null 值

你已经知道如何在`struct`变量中存储像数字这样的基本值。但如果一个变量还没有值呢？我们该如何表示这种情况？C#中有一个`null`值的概念，可以用来表示变量尚未被赋值。

## 使值类型可空

默认情况下，像`int`和`DateTime`这样的值类型必须始终有值，因此得名。有时，例如在读取数据库中允许空、缺失或`null`值存储的值时，允许值类型为`null`会很方便。我们称这种类型为**可空值类型**。

你可以通过在声明变量时在类型后添加问号后缀来启用此功能。

让我们来看一个例子：

1.  使用你偏好的编程工具，在`Chapter06`工作区/解决方案中添加一个名为`NullHandling`的**控制台应用程序**。本节需要一个完整的应用程序，包含项目文件，因此你无法使用.NET Interactive 笔记本。

1.  在 Visual Studio Code 中，选择`NullHandling`作为活动的 OmniSharp 项目。在 Visual Studio 中，将`NullHandling`设置为启动项目。

1.  在`Program.cs`中，输入声明并赋值的语句，包括`null`，给`int`变量，如下列代码所示：

    ```cs
    int thisCannotBeNull  = 4; 
    thisCannotBeNull = null; // compile error!
    int? thisCouldBeNull = null; 
    WriteLine(thisCouldBeNull); 
    WriteLine(thisCouldBeNull.GetValueOrDefault());
    thisCouldBeNull = 7; 
    WriteLine(thisCouldBeNull); 
    WriteLine(thisCouldBeNull.GetValueOrDefault()); 
    ```

1.  注释掉导致编译错误的语句。

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    0
    7
    7 
    ```

第一行是空白的，因为它输出了`null`值！

## 理解可空引用类型

在众多语言中，`null`值的使用非常普遍，以至于许多经验丰富的程序员从未质疑过其存在的必要性。但在许多情况下，如果我们不允许变量具有`null`值，就能编写出更优、更简洁的代码。

C# 8 中最显著的语言变化是引入了可空和不可空的引用类型。“但是等等！”你可能会想，“引用类型不是已经可空了吗！”

您说得没错，但在 C# 8 及更高版本中，引用类型可以通过设置文件级或项目级选项来配置，不再允许`null`值，从而启用这一有用的新特性。由于这对 C#来说是一个重大变化，微软决定让该功能为可选。

由于成千上万的现有库包和应用程序期望旧的行为，这项新的 C#语言特性需要多年时间才能产生影响。即使是微软，也直到.NET 6 才在所有主要的.NET 包中完全实现这一新特性。

在过渡期间，您可以为您的项目选择几种方法之一：

+   **默认**：无需更改。不支持不可空的引用类型。

+   **项目级选择加入，文件级选择退出**：在项目级别启用该功能，并为需要与旧行为保持兼容的任何文件选择退出。这是微软在更新其自己的包以使用此新功能时内部采用的方法。

+   **文件级选择加入**：仅对个别文件启用该功能。

## 启用可空和不可空的引用类型

要在项目级别启用该功能，请在项目文件中添加以下内容：

```cs
<PropertyGroup>
  ...
  <Nullable>enable</Nullable>
</PropertyGroup> 
```

这在面向.NET 6.0 的项目模板中现已默认完成。

要在文件级别禁用该功能，请在代码文件顶部添加以下内容：

```cs
#nullable disable 
```

要在文件级别启用该功能，请在代码文件顶部添加以下内容：

```cs
#nullable enable 
```

## 声明不可为空的变量和参数

如果您启用了可空引用类型，并且希望引用类型被赋予`null`值，那么您将不得不使用与使值类型可空相同的语法，即在类型声明后添加一个`?`符号。

那么，可空引用类型是如何工作的呢？让我们看一个例子。当存储地址信息时，您可能希望强制为街道、城市和地区提供值，但建筑可以留空，即`null`：

1.  在`NullHandling.csproj`中，在`Program.cs`文件底部，添加声明一个具有四个字段的`Address`类的语句，如下所示：

    ```cs
    class Address
    {
      public string? Building; 
      public string Street; 
      public string City; 
      public string Region;
    } 
    ```

1.  几秒钟后，注意关于不可为空的字段的警告，例如`Street`未初始化，如*图 6.2*所示：![Graphical user interface, text, application, chat or text message  Description automatically generated](img/B17442_06_02.png)

    图 6.2：PROBLEMS 窗口中关于不可为空的字段的警告信息

1.  将空`string`值分配给三个不可为空的字段中的每一个，如下所示：

    ```cs
    public string Street = string.Empty; 
    public string City = string.Empty; 
    public string Region = string.Empty; 
    ```

1.  在`Program.cs`中，在文件顶部，静态导入`Console`，然后添加语句来实例化一个`Address`并设置其属性，如下所示：

    ```cs
    Address address = new(); 
    address.Building = null; 
    address.Street = null; 
    address.City = "London"; 
    address.Region = null; 
    ```

1.  注意警告，如*图 6.3*所示：![图形用户界面，文本，应用程序，聊天或短信，电子邮件 自动生成描述](img/B17442_06_03.png)

    图 6.3：关于将 null 分配给不可空字段的警告消息

因此，这就是为什么新语言特性被命名为可空引用类型。从 C# 8.0 开始，未修饰的引用类型可以变为不可空，并且用于使引用类型可空的语法与用于值类型的语法相同。

## 检查是否为空

检查可空引用类型或可空值类型变量当前是否包含`null`很重要，因为如果不这样做，可能会抛出`NullReferenceException`，导致错误。在使用可空变量之前，应检查其是否为`null`，如下所示：

```cs
// check that the variable is not null before using it
if (thisCouldBeNull != null)
{
  // access a member of thisCouldBeNull
  int length = thisCouldBeNull.Length; // could throw exception
  ...
} 
```

C# 7 引入了`is`与`!`（非）运算符的组合作为`!=`的替代方案，如下所示：

```cs
if (!(thisCouldBeNull is null))
{ 
```

C# 9 引入了`is not`作为更清晰的替代方案，如下所示：

```cs
if (thisCouldBeNull is not null)
{ 
```

如果您尝试使用可能为`null`的变量的成员，请使用空条件运算符`?.`，如下所示：

```cs
string authorName = null;
// the following throws a NullReferenceException
int x = authorName.Length;
// instead of throwing an exception, null is assigned to y
int? y = authorName?.Length; 
```

有时您希望将变量分配给结果，或者如果变量为`null`，则使用备用值，例如`3`。您可以使用空合并运算符`??`执行此操作，如下所示：

```cs
// result will be 3 if authorName?.Length is null 
int result = authorName?.Length ?? 3; 
Console.WriteLine(result); 
```

**良好实践**：即使启用了可空引用类型，您仍应检查不可空参数是否为`null`并抛出`ArgumentNullException`。

### 在方法参数中检查是否为空

在定义带有参数的方法时，检查`null`值是良好的实践。

在早期版本的 C#中，您需要编写`if`语句来检查`null`参数值，并对任何为`null`的参数抛出`ArgumentNullException`，如下所示：

```cs
public void Hire(Person manager, Person employee)
{
  if (manager == null)
  {
    throw new ArgumentNullException(nameof(manager));
  }
  if (employee == null)
  {
    throw new ArgumentNullException(nameof(employee));
  }
  ...
} 
```

C# 11 可能会引入一个新的`!!`后缀，为您执行此操作，如下所示：

```cs
public void Hire(Person manager!!, Person employee!!)
{
  ...
} 
```

`if`语句和抛出异常的操作已为您完成。

# 继承自类

我们之前创建的`Person`类型派生（继承）自`object`，即`System.Object`的别名。现在，我们将创建一个从`Person`继承的子类：

1.  在`PacktLibrary`项目中，添加一个名为`Employee.cs`的新类文件。

1.  修改其内容以定义一个名为`Employee`的类，该类派生自`Person`，如下所示：

    ```cs
    using System;
    namespace Packt.Shared;
    public class Employee : Person
    {
    } 
    ```

1.  在`Program.cs`中，添加语句以创建`Employee`类的一个实例，如下所示：

    ```cs
    Employee john = new()
    {
      Name = "John Jones",
      DateOfBirth = new(year: 1990, month: 7, day: 28)
    };
    john.WriteToConsole(); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    John Jones was born on a Saturday. 
    ```

请注意，`Employee`类继承了`Person`类的所有成员。

## 扩展类以添加功能

现在，我们将添加一些特定于员工的成员以扩展该类。

1.  在`Employee.cs`中，添加语句以定义员工代码和雇佣日期这两个属性，如下所示：

    ```cs
    public string? EmployeeCode { get; set; } 
    public DateTime HireDate { get; set; } 
    ```

1.  在`Program.cs`中，添加语句以设置 John 的员工代码和雇佣日期，如下列代码所示：

    ```cs
    john.EmployeeCode = "JJ001";
    john.HireDate = new(year: 2014, month: 11, day: 23); 
    WriteLine($"{john.Name} was hired on {john.HireDate:dd/MM/yy}"); 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    John Jones was hired on 23/11/14 
    ```

## 隐藏成员

到目前为止，`WriteToConsole`方法是从`Person`继承的，它仅输出员工的姓名和出生日期。我们可能希望为员工改变此方法的功能：

1.  在`Employee.cs`中，添加语句以重新定义`WriteToConsole`方法，如下列高亮代码所示：

    ```cs
    **using****static** **System.Console;** 
    namespace Packt.Shared;
    public class Employee : Person
    {
      public string? EmployeeCode { get; set; }
      public DateTime HireDate { get; set; }
    **public****void****WriteToConsole****()**
     **{**
     **WriteLine(format:**
    **"{0} was born on {1:dd/MM/yy} and hired on {2:dd/MM/yy}"****,**
     **arg0: Name,**
     **arg1: DateOfBirth,**
     **arg2: HireDate);**
     **}**
    } 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    John Jones was born on 28/07/90 and hired on 01/01/01 
    John Jones was hired on 23/11/14 
    ```

你的编码工具会警告你，你的方法现在通过在方法名下划波浪线来隐藏来自`Person`的方法，**问题**/**错误列表**窗口包含更多细节，编译器会在你构建并运行控制台应用程序时输出警告，如*图 6.4*所示：

![图形用户界面，文本，应用程序，电子邮件 描述自动生成](img/B17442_06_04.png)

图 6.4：隐藏方法警告

正如警告所述，你可以通过将`new`关键字应用于该方法来隐藏此消息，以表明你是有意替换旧方法，如下列高亮代码所示：

```cs
public **new** void WriteToConsole() 
```

## 覆盖成员

与其隐藏一个方法，通常更好的做法是**覆盖**它。只有当基类选择允许覆盖时，你才能覆盖，这通过将`virtual`关键字应用于应允许覆盖的任何方法来实现。

来看一个例子：

1.  在`Program.cs`中，添加一条语句，使用其`string`表示形式将`john`变量的值写入控制台，如下列代码所示：

    ```cs
    WriteLine(john.ToString()); 
    ```

1.  运行代码并注意`ToString`方法是从`System.Object`继承的，因此实现返回命名空间和类型名称，如下列输出所示：

    ```cs
    Packt.Shared.Employee 
    ```

1.  在`Person.cs`中，通过添加一个`ToString`方法来覆盖此行为，该方法输出人的姓名以及类型名称，如下列代码所示：

    ```cs
    // overridden methods
    public override string ToString()
    {
      return $"{Name} is a {base.ToString()}";
    } 
    ```

    `base`关键字允许子类访问其超类的成员；即它继承或派生自的**基类**。

1.  运行代码并查看结果。现在，当调用`ToString`方法时，它输出人的姓名，并返回基类`ToString`的实现，如下列输出所示：

    ```cs
     John Jones is a Packt.Shared.Employee 
    ```

**最佳实践**：许多现实世界的 API，例如微软的 Entity Framework Core、Castle 的 DynamicProxy 和 Episerver 的内容模型，要求你在类中定义的属性标记为`virtual`，以便它们可以被覆盖。仔细决定你的哪些方法和属性成员应标记为`virtual`。

## 继承自抽象类

本章早些时候，你了解到接口可以定义一组成员，类型必须拥有这些成员才能达到基本的功能水平。这些接口非常有用，但主要局限在于，直到 C# 8 之前，它们无法提供任何自身的实现。

如果你仍然需要创建与.NET Framework 和其他不支持.NET Standard 2.1 的平台兼容的类库，这将是一个特定问题。

在那些早期平台中，你可以使用抽象类作为一种介于纯接口和完全实现类之间的半成品。

当一个类被标记为`abstract`时，这意味着它不能被实例化，因为你表明该类不完整。它需要更多的实现才能被实例化。

例如，`System.IO.Stream`类是抽象的，因为它实现了所有流都需要的一般功能，但并不完整，因此你不能使用`new Stream()`来实例化它。

让我们比较两种类型的接口和两种类型的类，如下代码所示：

```cs
public interface INoImplementation // C# 1.0 and later
{
  void Alpha(); // must be implemented by derived type
}
public interface ISomeImplementation // C# 8.0 and later
{
  void Alpha(); // must be implemented by derived type
  void Beta()
  {
    // default implementation; can be overridden
  }
}
public abstract class PartiallyImplemented // C# 1.0 and later
{
  public abstract void Gamma(); // must be implemented by derived type
  public virtual void Delta() // can be overridden
  {
    // implementation
  }
}
public class FullyImplemented : PartiallyImplemented, ISomeImplementation
{
  public void Alpha()
  {
    // implementation
  }
  public override void Gamma()
  {
    // implementation
  }
}
// you can only instantiate the fully implemented class
FullyImplemented a = new();
// all the other types give compile errors
PartiallyImplemented b = new(); // compile error!
ISomeImplementation c = new(); // compile error!
INoImplementation d = new(); // compile error! 
```

## 防止继承和覆盖

通过在其定义中应用`sealed`关键字，你可以防止其他开发者继承你的类。没有人能继承史高治·麦克达克，如下代码所示：

```cs
public sealed class ScroogeMcDuck
{
} 
```

.NET 中`sealed`的一个例子是`string`类。微软在`string`类内部实现了一些极端优化，这些优化可能会因你的继承而受到负面影响，因此微软阻止了这种情况。

你可以通过在方法上应用`sealed`关键字来防止某人进一步覆盖你类中的`virtual`方法。没有人能改变 Lady Gaga 的唱歌方式，如下代码所示：

```cs
using static System.Console;
namespace Packt.Shared;
public class Singer
{
  // virtual allows this method to be overridden
  public virtual void Sing()
  {
    WriteLine("Singing...");
  }
}
public class LadyGaga : Singer
{
  // sealed prevents overriding the method in subclasses
  public sealed override void Sing()
  {
    WriteLine("Singing with style...");
  }
} 
```

你只能密封一个被覆盖的方法。

## 理解多态性

你现在看到了两种改变继承方法行为的方式。我们可以使用`new`关键字*隐藏*它（称为**非多态继承**），或者我们可以*覆盖*它（称为**多态继承**）。

两种方式都可以使用`base`关键字访问基类或超类的成员，那么区别是什么呢？

这完全取决于持有对象引用的变量类型。例如，类型为`Person`的变量可以持有`Person`类或任何派生自`Person`的类型的引用。

让我们看看这如何影响你的代码：

1.  在`Employee.cs`中，添加语句以覆盖`ToString`方法，使其将员工的名字和代码写入控制台，如下代码所示：

    ```cs
    public override string ToString()
    {
      return $"{Name}'s code is {EmployeeCode}";
    } 
    ```

1.  在`Program.cs`中，编写语句以创建名为 Alice 的新员工，将其存储在类型为`Person`的变量中，并调用两个变量的`WriteToConsole`和`ToString`方法，如下代码所示：

    ```cs
    Employee aliceInEmployee = new()
      { Name = "Alice", EmployeeCode = "AA123" };
    Person aliceInPerson = aliceInEmployee; 
    aliceInEmployee.WriteToConsole(); 
    aliceInPerson.WriteToConsole(); 
    WriteLine(aliceInEmployee.ToString()); 
    WriteLine(aliceInPerson.ToString()); 
    ```

1.  运行代码并查看结果，如下输出所示：

    ```cs
    Alice was born on 01/01/01 and hired on 01/01/01 
    Alice was born on a Monday
    Alice's code is AA123 
    Alice's code is AA123 
    ```

当一个方法被`new`隐藏时，编译器不够智能，无法知道该对象是`Employee`，因此它调用`Person`中的`WriteToConsole`方法。

当一个方法被`virtual`和`override`覆盖时，编译器足够智能，知道尽管变量声明为`Person`类，但对象本身是`Employee`类，因此调用`Employee`的`ToString`实现。

成员修饰符及其效果总结在下表中：

| 变量类型 | 成员修饰符 | 执行的方法 | 所在类 |
| --- | --- | --- | --- |
| `Person` |  | `WriteToConsole` | `Person` |
| `Employee` | `new` | `WriteToConsole` | `Employee` |
| `Person` | `virtual` | `ToString` | `Employee` |
| `Employee` | `override` | `ToString` | `Employee` |

在我看来，多态性对大多数程序员来说是学术性的。如果你理解了这个概念，那很酷；但如果不理解，我建议你不必担心。有些人喜欢通过说理解多态性对所有 C#程序员学习很重要来让别人感到自卑，但在我看来并非如此。

你可以通过 C#拥有成功的职业生涯，而不必解释多态性，正如赛车手无需解释燃油喷射背后的工程原理一样。

**最佳实践**：应尽可能使用`virtual`和`override`而不是`new`来更改继承方法的实现。

# 继承层次结构内的强制转换

类型之间的强制转换与类型转换略有不同。强制转换是在相似类型之间进行的，例如 16 位整数和 32 位整数之间，或者超类及其子类之间。转换是在不同类型之间进行的，例如文本和数字之间。

## 隐式转换

在前面的示例中，你看到了如何将派生类型的实例存储在其基类型（或其基类型的基类型等）的变量中。当我们这样做时，称为**隐式转换**。

## 显式转换

反向操作是显式转换，你必须在要转换的类型周围使用括号作为前缀来执行此操作：

1.  在`Program.cs`中，添加一个语句，将`aliceInPerson`变量赋值给一个新的`Employee`变量，如下所示：

    ```cs
    Employee explicitAlice = aliceInPerson; 
    ```

1.  你的编码工具会显示红色波浪线和编译错误，如*图 6.5*所示：![图形用户界面，文本，应用程序，电子邮件，网站 自动生成描述](img/B17442_06_05.png)

    图 6.5：缺少显式转换的编译错误

1.  将语句更改为在赋值变量名前加上`Employee`类型的强制转换，如下所示：

    ```cs
    Employee explicitAlice = (Employee)aliceInPerson; 
    ```

## 避免强制转换异常

编译器现在满意了；但是，因为`aliceInPerson`可能是不同的派生类型，比如`Student`而不是`Employee`，我们需要小心。在更复杂的代码的实际应用程序中，此变量的当前值可能已被设置为`Student`实例，然后此语句将抛出`InvalidCastException`错误。

我们可以通过编写`try`语句来处理这种情况，但还有更好的方法。我们可以使用`is`关键字检查对象的类型：

1.  将显式转换语句包裹在`if`语句中，如下所示突出显示：

    ```cs
    **if** **(aliceInPerson** **is** **Employee)**
    **{**
     **WriteLine(****$"****{****nameof****(aliceInPerson)}** **IS an Employee"****);** 
      Employee explicitAlice = (Employee)aliceInPerson;
    **// safely do something with explicitAlice**
    **}** 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    aliceInPerson IS an Employee 
    ```

    你可以通过使用声明模式进一步简化代码，这将避免需要执行显式转换，如下所示：

    ```cs
    if (aliceInPerson is Employee explicitAlice)  
    {
      WriteLine($"{nameof(aliceInPerson)} IS an Employee"); 
      // safely do something with explicitAlice
    } 
    ```

    或者，你可以使用`as`关键字进行转换。如果无法进行类型转换，`as`关键字不会抛出异常，而是返回`null`。

1.  在`Main`中，添加语句，使用`as`关键字转换 Alice，然后检查返回值是否不为空，如下所示：

    ```cs
    Employee? aliceAsEmployee = aliceInPerson as Employee; // could be null
    if (aliceAsEmployee != null)
    {
      WriteLine($"{nameof(aliceInPerson)} AS an Employee");
      // safely do something with aliceAsEmployee
    } 
    ```

    由于访问`null`变量的成员会抛出`NullReferenceException`错误，因此在使用结果之前应始终检查`null`。

1.  运行代码并查看结果，如下所示：

    ```cs
    aliceInPerson AS an Employee 
    ```

如果你想在 Alice 不是员工时执行一组语句，该怎么办？

在过去，你可能会使用`!`（非）运算符，如下所示：

```cs
if (!(aliceInPerson is Employee)) 
```

使用 C# 9 及更高版本，你可以使用`not`关键字，如下所示：

```cs
if (aliceInPerson is not Employee) 
```

**最佳实践**：使用`is`和`as`关键字避免在派生类型之间转换时抛出异常。如果不这样做，你必须为`InvalidCastException`编写`try`-`catch`语句。

# 继承和扩展.NET 类型

.NET 拥有预建的类库，包含数十万个类型。与其完全创建全新的类型，不如从微软的类型中派生，继承其部分或全部行为，然后覆盖或扩展它，从而获得先机。

## 继承异常

作为继承的一个例子，我们将派生一种新的异常类型：

1.  在`PacktLibrary`项目中，添加一个名为`PersonException.cs`的新类文件。

1.  修改文件内容，定义一个名为`PersonException`的类，包含三个构造函数，如下所示：

    ```cs
    namespace Packt.Shared;
    public class PersonException : Exception
    {
      public PersonException() : base() { }
      public PersonException(string message) : base(message) { }
      public PersonException(string message, Exception innerException)
        : base(message, innerException) { }
    } 
    ```

    与普通方法不同，构造函数不会被继承，因此我们必须显式声明并在`System.Exception`中显式调用基类构造函数实现，以便让可能希望使用这些构造函数的程序员能够使用我们自定义的异常。

1.  在`Person.cs`中，添加语句以定义一个方法，如果日期/时间参数早于某人的出生日期，则抛出异常，如下所示：

    ```cs
    public void TimeTravel(DateTime when)
    {
      if (when <= DateOfBirth)
      {
        throw new PersonException("If you travel back in time to a date earlier than your own birth, then the universe will explode!");
      }
      else
      {
        WriteLine($"Welcome to {when:yyyy}!");
      }
    } 
    ```

1.  在`Program.cs`中，添加语句以测试当员工 John Jones 试图穿越回太久远的时间时会发生什么，如下所示：

    ```cs
    try
    {
      john.TimeTravel(when: new(1999, 12, 31));
      john.TimeTravel(when: new(1950, 12, 25));
    }
    catch (PersonException ex)
    {
      WriteLine(ex.Message);
    } 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Welcome to 1999!
    If you travel back in time to a date earlier than your own birth, then the universe will explode! 
    ```

**最佳实践**：在定义自己的异常时，应提供与内置异常相同的三个构造函数，并显式调用它们。

## 当你无法继承时扩展类型

之前，我们了解到`sealed`修饰符可用于防止继承。

微软已将`sealed`关键字应用于`System.String`类，以确保无人能继承并可能破坏字符串的行为。

我们还能给字符串添加新方法吗？可以，如果我们使用名为**扩展方法**的语言特性，该特性是在 C# 3.0 中引入的。

### 使用静态方法重用功能

自 C#的第一个版本以来，我们就能创建`static`方法来重用功能，例如验证`string`是否包含电子邮件地址的能力。其实现将使用正则表达式，你将在*第八章*，*使用常见的.NET 类型*中了解更多相关内容。

让我们来编写一些代码：

1.  在`PacktLibrary`项目中，添加一个名为`StringExtensions`的新类，如下列代码所示，并注意以下事项：

    +   该类导入了一个用于处理正则表达式的命名空间。

    +   `IsValidEmail`方法是`static`的，它使用`Regex`类型来检查与一个简单的电子邮件模式匹配，该模式寻找`@`符号前后有效的字符。

    ```cs
    using System.Text.RegularExpressions;
    namespace Packt.Shared;
    public class StringExtensions
    {
      public static bool IsValidEmail(string input)
      {
        // use simple regular expression to check
        // that the input string is a valid email
        return Regex.IsMatch(input,
          @"[a-zA-Z0-9\.-_]+@[a-zA-Z0-9\.-_]+");
      }
    } 
    ```

1.  在`Program.cs`中，添加语句以验证两个电子邮件地址示例，如下列代码所示：

    ```cs
    string email1 = "pamela@test.com"; 
    string email2 = "ian&test.com";
    WriteLine("{0} is a valid e-mail address: {1}", 
      arg0: email1,
      arg1: StringExtensions.IsValidEmail(email1));
    WriteLine("{0} is a valid e-mail address: {1}",
      arg0: email2,
      arg1: StringExtensions.IsValidEmail(email2)); 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    pamela@test.com is a valid e-mail address: True 
    ian&test.com is a valid e-mail address: False 
    ```

这可行，但扩展方法能减少我们必须输入的代码量并简化此功能的使用。

### 使用扩展方法重用功能

将`static`方法转换为扩展方法很容易：

1.  在`StringExtensions.cs`中，在类前添加`static`修饰符，并在`string`类型前添加`this`修饰符，如下列代码中突出显示：

    ```cs
    public **static** class StringExtensions
    {
      public static bool IsValidEmail(**this** string input)
      { 
    ```

    这两个改动告诉编译器，应将该方法视为扩展`string`类型的方法。

1.  在`Program.cs`中，添加语句以使用扩展方法检查需要验证的`string`值是否为有效电子邮件地址，如下列代码所示：

    ```cs
    WriteLine("{0} is a valid e-mail address: {1}",
      arg0: email1,
      arg1: email1.IsValidEmail());
    WriteLine("{0} is a valid e-mail address: {1}", 
      arg0: email2,
      arg1: email2.IsValidEmail()); 
    ```

    注意调用`IsValidEmail`方法的语法中微妙的简化。较旧、较长的语法仍然有效。

1.  `IsValidEmail`扩展方法现在看起来就像是`string`类型的所有实际实例方法一样，例如`IsNormalized`和`Insert`，如*图 6.6*所示：![图形用户界面，文本，应用程序 自动生成的描述](img/B17442_06_06.png)

    图 6.6：扩展方法在 IntelliSense 中与实例方法并列显示

1.  运行代码并查看结果，其将与之前相同。

**良好实践**：扩展方法不能替换或覆盖现有实例方法。例如，你不能重新定义`Insert`方法。扩展方法会在 IntelliSense 中显示为重载，但具有相同名称和签名的实例方法会被优先调用。

尽管扩展方法可能看似没有带来巨大好处，但在*第十一章*，*使用 LINQ 查询和操作数据*中，你将看到扩展方法的一些极其强大的用途。

# 使用分析器编写更优质的代码

.NET 分析器能发现潜在问题并提出修复建议。**StyleCop**是一个常用的分析器，帮助你编写更优质的 C#代码。

让我们看看实际操作，指导如何在面向.NET 5.0 的控制台应用项目模板中改进代码，以便控制台应用已具备一个包含`Main`方法的`Program`类：

1.  使用您喜欢的代码编辑器添加一个控制台应用程序项目，如下表所定义：

    1.  项目模板：**控制台应用程序** / `console -f net5.0`

    1.  工作区/解决方案文件和文件夹：`Chapter06`

    1.  项目文件和文件夹：`CodeAnalyzing`

    1.  目标框架：**.NET 5.0（当前）**

1.  在 `CodeAnalyzing` 项目中，添加对 `StyleCop.Analyzers` 包的引用。

1.  向您的项目添加一个名为 `stylecop.json` 的 JSON 文件，以控制 StyleCop 设置。

1.  修改其内容，如下面的标记所示：

    ```cs
    {
      "$schema": "https://raw.githubusercontent.com/DotNetAnalyzers/StyleCopAnalyzers/master/StyleCop.Analyzers/StyleCop.Analyzers/Settings/stylecop.schema.json",
      "settings": {
      }
    } 
    ```

    `$schema` 条目在代码编辑器中编辑 `stylecop.json` 文件时启用 IntelliSense。

1.  编辑项目文件，将目标框架更改为 `net6.0`，添加条目以配置名为 `stylecop.json` 的文件，使其不在发布的部署中包含，并在开发期间作为附加文件进行处理，如下面的标记中突出显示的那样：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
      </PropertyGroup>
     **<ItemGroup>**
     **<None Remove=****"stylecop.json"** **/>**
     **</ItemGroup>**
     **<ItemGroup>**
     **<AdditionalFiles Include=****"stylecop.json"** **/>**
     **</ItemGroup>**
      <ItemGroup>
        <PackageReference Include="StyleCop.Analyzers" Version="1.2.0-*">
          <PrivateAssets>all</PrivateAssets>
          <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
        </PackageReference>
      </ItemGroup>
    </Project> 
    ```

1.  构建您的项目。

1.  您将看到它认为有问题的所有内容的警告，如图 *6.7* 所示：![](img/B17442_06_07.png)

    图 6.7：StyleCop 代码分析器警告

1.  例如，它希望 `using` 指令放在命名空间声明内，如下面的输出所示：

    ```cs
    C:\Code\Chapter06\CodeAnalyzing\Program.cs(1,1): warning SA1200: Using directive should appear within a namespace declaration [C:\Code\Chapter06\CodeAnalyzing\CodeAnalyzing.csproj] 
    ```

## 抑制警告

要抑制警告，您有几种选择，包括添加代码和设置配置。

要抑制使用属性，如下面的代码所示：

```cs
[assembly:SuppressMessage("StyleCop.CSharp.OrderingRules", "SA1200:UsingDirectivesMustBePlacedWithinNamespace", Justification = "Reviewed.")] 
```

要抑制使用指令，如下面的代码所示：

```cs
#pragma warning disable SA1200 // UsingDirectivesMustBePlacedWithinNamespace
using System;
#pragma warning restore SA1200 // UsingDirectivesMustBePlacedWithinNamespace 
```

通过修改 `stylecop.json` 文件来抑制警告：

1.  在 `stylecop.json` 中，添加一个配置选项，将 `using` 语句设置为允许在命名空间外部使用，如下面的标记中突出显示的那样：

    ```cs
    {
      "$schema": "https://raw.githubusercontent.com/DotNetAnalyzers/StyleCopAnalyzers/master/StyleCop.Analyzers/StyleCop.Analyzers/Settings/stylecop.schema.json",
      "settings": {
        "orderingRules": {
          "usingDirectivesPlacement": "outsideNamespace"
        }
      }
    } 
    ```

1.  构建项目并注意警告 SA1200 已消失。

1.  在 `stylecop.json` 中，将 using 指令的位置设置为 `preserve`，允许 `using` 语句在命名空间内部和外部使用，如下面的标记所示：

    ```cs
    "orderingRules": {
      "usingDirectivesPlacement": "preserve"
    } 
    ```

### 修复代码

现在，让我们修复所有其他警告：

1.  在 `CodeAnalyzing.csproj` 中，添加一个元素以自动生成文档的 XML 文件，如下面的标记中突出显示的那样：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
     **<GenerateDocumentationFile>****true****</GenerateDocumentationFile>**
      </PropertyGroup> 
    ```

1.  在 `stylecop.json` 中，添加一个配置选项，为公司名称和版权文本的文档提供值，如下面的标记中突出显示的那样：

    ```cs
    {
      "$schema": "https://raw.githubusercontent.com/DotNetAnalyzers/StyleCopAnalyzers/master/StyleCop.Analyzers/StyleCop.Analyzers/Settings/stylecop.schema.json",
      "settings": {
        "orderingRules": {
          "usingDirectivesPlacement": "preserve"
        },
    **"documentationRules"****: {**
    **"companyName"****:** **"Packt"****,**
    **"copyrightText"****:** **"Copyright (c) Packt. All rights reserved."**
     **}**
      }
    } 
    ```

1.  在 `Program.cs` 中，为文件头添加公司和版权文本的注释，将 `using System;` 声明移至命名空间内部，并为类和方法设置显式访问修饰符和 XML 注释，如下面的代码所示：

    ```cs
    // <copyright file="Program.cs" company="Packt">
    // Copyright (c) Packt. All rights reserved.
    // </copyright>
    namespace CodeAnalyzing
    {
      using System;
      /// <summary>
      /// The main class for this console app.
      /// </summary>
      public class Program
      {
        /// <summary>
        /// The main entry point for this console app.
        /// </summary>
        /// <param name="args">A string array of arguments passed to the console app.</param>
        public static void Main(string[] args)
        {
          Console.WriteLine("Hello World!");
        }
      }
    } 
    ```

1.  构建项目。

1.  展开 `bin/Debug/net6.0` 文件夹并注意名为 `CodeAnalyzing.xml` 的自动生成的文件，如下面的标记所示：

    ```cs
    <?xml version="1.0"?>
    <doc>
        <assembly>
            <name>CodeAnalyzing</name>
        </assembly>
        <members>
            <member name="T:CodeAnalyzing.Program">
                <summary>
                The main class for this console app.
                </summary>
            </member>
            <member name="M:CodeAnalyzing.Program.Main(System.String[])">
                <summary>
                The main entry point for this console app.
                </summary>
                <param name="args">A string array of arguments passed to the console app.</param>
            </member>
        </members>
    </doc> 
    ```

### 理解常见的 StyleCop 建议

在代码文件内部，应按以下列表所示顺序排列内容：

1.  外部别名指令

1.  使用指令

1.  命名空间

1.  委托

1.  枚举

1.  接口

1.  结构体

1.  类

在类、记录、结构或接口内部，应按以下列表所示顺序排列内容：

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

**良好实践**：你可以在以下链接了解所有 StyleCop 规则：[`github.com/DotNetAnalyzers/StyleCopAnalyzers/blob/master/DOCUMENTATION.md`](https://github.com/DotNetAnalyzers/StyleCopAnalyzers/blob/master/DOCUMENTATION.md)。

# 实践与探索

通过回答一些问题来测试你的知识和理解。通过更深入的研究，获得一些实践经验并探索本章的主题。

## 练习 6.1 – 测试你的知识

回答以下问题：

1.  什么是委托？

1.  什么是事件？

1.  基类和派生类是如何关联的，派生类如何访问基类？

1.  `is`和`as`操作符之间有什么区别？

1.  哪个关键字用于防止一个类被派生或一个方法被进一步重写？

1.  哪个关键字用于防止一个类通过`new`关键字实例化？

1.  哪个关键字用于允许成员被重写？

1.  析构函数和解构方法之间有什么区别？

1.  所有异常应具有的构造函数的签名是什么？

1.  什么是扩展方法，如何定义一个？

## 练习 6.2 – 实践创建继承层次结构

通过以下步骤探索继承层次结构：

1.  向你的`Chapter06`解决方案/工作区中添加一个名为`Exercise02`的新控制台应用程序。

1.  创建一个名为`Shape`的类，其属性名为`Height`、`Width`和`Area`。

1.  添加三个从它派生的类——`Rectangle`、`Square`和`Circle`——根据你认为合适的任何额外成员，并正确地重写和实现`Area`属性。

1.  在`Main`中，添加语句以创建每种形状的一个实例，如下列代码所示：

    ```cs
    Rectangle r = new(height: 3, width: 4.5);
    WriteLine($"Rectangle H: {r.Height}, W: {r.Width}, Area: {r.Area}"); 
    Square s = new(5);
    WriteLine($"Square H: {s.Height}, W: {s.Width}, Area: {s.Area}"); 
    Circle c = new(radius: 2.5);
    WriteLine($"Circle H: {c.Height}, W: {c.Width}, Area: {c.Area}"); 
    ```

1.  运行控制台应用程序，并确保结果与以下输出相符：

    ```cs
    Rectangle H: 3, W: 4.5, Area: 13.5
    Square H: 5, W: 5, Area: 25
    Circle H: 5, W: 5, Area: 19.6349540849362 
    ```

## 练习 6.3 – 探索主题

使用以下页面上的链接来了解更多关于本章涵盖的主题：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-6---implementing-interfaces-and-inheriting-classes`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-6---implementing-interfaces-and-inheriting-classes)

# 总结

在本章中，你学习了局部函数和操作符、委托和事件、实现接口、泛型以及使用继承和 OOP 派生类型。你还学习了基类和派生类，以及如何重写类型成员、使用多态性以及在类型之间进行转换。

在下一章中，你将学习.NET 是如何打包和部署的，以及在后续章节中，它为你提供的实现常见功能（如文件处理、数据库访问、加密和多任务处理）的类型。
