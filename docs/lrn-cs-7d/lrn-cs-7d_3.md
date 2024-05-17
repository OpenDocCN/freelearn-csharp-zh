# 第三章：第三天 - C#中的新功能

今天，我们将学习 C#语言的当前版本中非常新的和新发布的功能，即 C# 7.0（这是本书审阅中最新的适应版本）。其中一些元素是全新的，而其他一些在过去的版本中已经存在，并在当前版本中得到了升级。C# 7.0 将带来许多新功能。其中一些元素，比如元组，是已经存在的概念的扩展，而其他一些是全新的。以下是我们将在第三天学习的基本元素：

+   元组和解构

+   模式匹配

+   本地函数

+   文字改进

+   异步主函数

+   默认表达式

+   推断元组名称

# 元组和解构

元组并不是在当前版本中新引入的，而是在.NET 4.0 发布时引入的。在当前版本中，它们得到了改进。

# 元组

每当特定情况需要从方法返回多个值时，就会使用元组。例如，假设我们需要从给定数字系列中找到奇数和偶数。

元组是一个包含相关数据的不可变数据值。元组用于聚合相关数据，例如一个人的姓名、年龄、性别以及任何你想要的数据作为输入。

为了完成这个问题，我们的方法应该返回或提供一个数字，并说明这是一个奇数还是偶数。对于将返回这些多个值的方法，我们可以使用自定义数据类型、动态返回类型或输出参数，这有时会让开发人员感到困惑。

要使用元组，您需要添加 NuGet 包：

[`www.nuget.org/packages/System.ValueTuple/`](https://www.nuget.org/packages/System.ValueTuple/)

对于这个问题，我们有一个元组对象，在 C# 7.0 中我们有两种不同的东西，元组类型和元组文字，用于从方法返回多个值。

让我们使用一个代码示例详细讨论元组。考虑以下代码片段：

```cs
public static (int, string) FindOddEvenBySingleNumber(int number) 
{ 
   string oddOrEven = IsOddNumber(number) ? "Odd" :"Even"; 
   return (number, oddOrEven);//tuple literal 
} 
FindOddEvenBySingleNumber is returning multiple values, which tells us whether a number is odd or even. See the return statement return (number, oddOrEven) of the preceding code: here, we are simply returning two different variables. Now, how are these values accessible from the caller method? In this case, we are returning a tuple value and the caller method will receive a tuple with these values, which are nothing but elements or items of a tuple. In this case, the number will be available as Item1 and oddOrEven as Item2 for the caller method. The following is from the caller method:
```

```cs
var result = OddEven.FindOddEvenBySingleNumber(Convert.ToInt32(number); 
Console.WriteLine($"Number:{result.Item1} is {result.Item2}"); 
result.Item1 represents number and result.Item2 represents oddOrEven. This is fine when someone knows the representation of these tuple items/elements. But think of a scenario where we have numerous tuple elements and the developer who is writing the caller method is not aware of the representation of these items/elements. In that case, it is bit complex to consume these tuple items/elements. To overcome this problem, we can give a name to these tuple items. We call these named tuple items/elements. Let us modify our method FindOddEvenBySingleNumber to return named tuple items:
```

```cs
public static (int number, string oddOrEvent) FindOddEvenBySingleNumber (int number) 
{ 
   string result = IsOddNumber(number) ? "Odd" : "Even"; 
   return (number:number, oddOrEvent: result);//returning
   named tuple element in tuple literal 
} 
```

在上述代码片段中，我们为元组添加了更具描述性的名称。现在调用方法可以直接使用这些名称，如下面的代码片段所示：

```cs
var result = OddEven.FindOddEvenBySingle(Convert.ToInt32(number)); 
Console.WriteLine($"Number:{result.number} is {result.oddOrEvent}"); 
```

通过为元组添加一些描述性名称，我们可以在调用方法中轻松识别和使用元组的项目/元素。

# System.ValueTuple 结构

在 C# 7.0 中，元组需要 NuGet 包`System.ValueType`。这实际上是一个结构。它包含一些静态和公共方法来进行操作：

+   **CompareTo(ValueTuple)**：比较`ValueTuple`实例的公共方法。如果比较成功，则该方法返回 0，否则返回 1。

+   这里有两个方法展示了`CompareTo`方法的强大之处：

```cs
public static bool CompareToTuple(int number) 
{ 
   var oddEvenValueTuple =
   FindOddEvenBySingleNumber(number); 
   var differentTupleValue =
   FindOddEvenBySingleNumberNamedElement(number + 1); 
   var res =
   oddEvenValueTuple.CompareTo(differentTupleValue); 
   return res == 0; // 0 if other is a ValueTuple instance
   and 1 if other is null 
} 
public static bool CompareToTuple1(int number) 
{ 
    var oddEvenValueTuple =
    FindOddEvenBySingleNumber(number); 
    var sameTupleValue =
    FindOddEvenBySingleNumberNamedElement(number); 
    var res = oddEvenValueTuple.CompareTo(sameTupleValue); 
    return res == 0;// 0 if other is a ValueTuple instance
    and 1 if other is null 
} 
```

以下是调用代码片段，用于从上述代码中获取结果：

```cs
Console.Clear(); 
Console.Write("Enter number: "); 
var num = Console.ReadLine(); 
var resultNum = OddEven.FindOddEvenBySingleNumberNamedElement(Convert.ToInt32(num)); 
Console.WriteLine($"Number:{resultNum.number} is {resultNum.oddOrEven}."); 
Console.WriteLine(); 
var comp = OddEven.CompareToTuple(Convert.ToInt32(num)); 
Console.WriteLine($"Comparison of two Tuple objects having different value is:{comp}"); 
var comp1 = OddEven.CompareToTuple1(Convert.ToInt32(num)); 
Console.WriteLine($"Comparison of two Tuple objects having same value is:{comp1}"); 
```

当我们执行上述代码时，将会得到以下输出：

![](img/00044.jpeg)

+   **Equals(Object)**：返回 true/false 的公共方法，指示`TupleValue`实例是否等于提供的对象。如果成功则返回 true。

+   以下是实现：

```cs
public static bool EqualToTuple(int number) 
{ 
   var oddEvenValueTuple =
   FindOddEvenBySingleNumber(number); 
   var sameTupleValue =
   FindOddEvenBySingleNumberNamedElement(number); 
   var res = oddEvenValueTuple.Equals(sameTupleValue); 
   return res;//true if obj is a ValueTuple instance;
   otherwise, false. 
} 
```

以下是调用方法的代码片段：

```cs
var num1 = Console.ReadLine(); 
var namedElement = OddEven.FindOddEvenBySingleNumberNamedElement(Convert.ToInt32(num1)); 
Console.WriteLine($"Number:{namedElement.number} is {namedElement.oddOrEven}."); 
Console.WriteLine(); 
var equalToTuple = OddEven.EqualToTuple(Convert.ToInt32(num1)); 
Console.WriteLine($"Equality of two Tuple objects is:{equalToTuple}"); 
var equalToObject = OddEven.EqualToObject(Convert.ToInt32(num1)); 
Console.WriteLine($"Equality of one Tuple object with other non tuple object is:{equalToObject}"); 
```

最后，输出如下：

![](img/00045.gif)

+   **Equals(ValueTuple)**：始终返回 true 的公共方法，这是设计上的。它是这样设计的，因为`ValueTuple`是一个零元素元组，因此当两个 ValueTuples 执行相等操作时，由于没有元素，将始终返回零。

+   **GetHashCode()**：返回对象的哈希码的公共方法。

+   **GetType()**：提供当前实例的具体类型的公共方法。

+   **ToString()**：是`ValueTuple`实例的字符串表示形式的公共方法。然而，根据设计，它总是返回零。

+   **Create()**：创建一个新的`ValueTuple`（0 元组）的静态方法。我们可以按以下方式创建 0 元组：

```cs
public static ValueTuple CreateValueTuple() => ValueTuple.Create();
```

+   **Create<T1>(T1) ... Create<T1, T2, T3, T4, T5, T6, T7, T8>(T1, T2, T3, T4, T5, T6, T7, T8)**：所有这些都是创建具有 1 个组件（单例）到 8 个组件（八元组）的 Value Tuples 的静态方法。

+   请参见以下代码片段，显示了单例和八元组示例：

```cs
public static ValueTuple<int> CreateValueTupleSingleton(int number) => ValueTuple.Create(number); 
public static ValueTuple<int, int, int, int, int, int, int, ValueTuple<int,string>> OctupleUsingCreate() => ValueTuple.Create(1, 2, 3, 4, 5, 6, 7, ValueTuple.Create(8, IsOddNumber(8) ? "Odd" : "Even")); 
```

如果出现编译警告，您需要将 NuGet 包更新到 Microsoft.Net.Compilers 2.0 预览版。要这样做，只需选择预览并从 NuGet 包管理器中搜索 Microsoft.Net.Compilers 到 2.0[[`www.nuget.org/packages/Microsoft.Net.Compilers/`](https://www.nuget.org/packages/Microsoft.Net.Compilers/)]。

# 解构

在前面的部分中，我们看到了使用`ValueTuple`的多个返回值可以通过其项/元素进行访问。现在想象一种情况，我们想要直接将这些元素值分配给变量。在这里，解构帮助了我们。解构是一种我们可以拆开方法返回的元组的方式。

主要有两种方法可以解构元组：

+   显式声明类型：我们明确声明每个字段的类型。让我们看下面的代码示例：

```cs
public static string ExplicitlyTypedDeconstruction(int num) 
{ 
   (int number, string evenOdd) =
   FindOddEvenBySingleNumber(num); 
   return $"Entered number:{number} is {evenOdd}."; 
} 
```

+   隐式声明类型：我们隐式声明每个字段的类型。让我们看下面的代码示例：

```cs
public static string ImplicitlyTypedDeconstruction(int num) 
{ 
   var (number, evenOdd) =
   FindOddEvenBySingleNumber(num); 
   //Following deconstruct is also valid 
   //(int number, var evenOdd) =
   FindOddEvenBySingleNumber(num); 
   return $"Entered number:{number} is {evenOdd}."; 
} 
```

我们还可以通过使用 out 参数实现解构 UserDefined/Custom 类型；请参阅以下代码示例：

```cs
public static string UserDefinedTypeDeconstruction(int num) 
{ 
   var customModel = new UserDefinedModel(num,
   IsOddNumber(num) ? "Odd" : "Even"); 
   var (number,oddEven) = customModel; 
   return $"Entered number:{number} is {oddEven}."; 
} 
```

在上面的代码中，deconstruct 方法使得从`UserDefinedModel`到一个 int 和一个 string 的赋值成为可能，它们分别代表了属性`number`和`OddEven`。

# 元组 - 需要记住的重要要点

在前面的部分中，我们讨论了元组，并注意到它们如何在需要多个值和复杂数据值（除了自定义类型）的情况下帮助我们。以下是我们在使用元组时应该记住的重要要点：

+   使用元组，我们需要 NuGet 包`System.ValueTuple`。

+   `ValueTuple` (`System.ValueTuple`)是一个结构，而不是一个类。

+   `ValueTuple`实现了`IEquatable<ValueTuple>, IStructuralEquatable, IStructuralComparable, IComparable, IComparable<ValueTuple>`接口。

+   ValueTuples 是可变的。

+   ValueTuples 是灵活的数据容器，可以是未命名的，也可以是命名的：

+   **未命名**：当我们不为字段提供任何名称时，这些是未命名的元组，并且可以使用默认字段`Item1`，`Item2`等进行访问：

```cs
var oddNumber = (3, "Odd"); //Unnamed tuple 
```

+   +   **命名**：当我们为字段明确提供一些描述性名称时：

```cs
var oddNumber = (number: 3, oddOrEven: "Odd"); //Named Tuple 
```

+   赋值：当我们将一个元组分配给另一个元组时，只有值被分配，而字段名称不会：

```cs
Console.Write("Enter first number: "); 
var userInputFirst = Console.ReadLine(); 
Console.Write("Enter second number: "); 
var userInputSecond = Console.ReadLine(); 
var noNamed = OddEven.FindOddEvenBySingleNumber(Convert.ToInt32(userInputFirst)); 
var named = OddEven.FindOddEvenBySingleNumberNamedElement(Convert.ToInt32(userInputSecond)); 
Console.WriteLine($"First Number:{noNamed.Item1} is {noNamed.Item2} using noNamed tuple."); 
Console.WriteLine($"Second Number:{named.number} is {named.oddOrEven} using Named tuple."); 

Console.WriteLine("Assigning 'Named' to 'NoNamed'"); 
                        noNamed = named; 
Console.WriteLine($"Number:{noNamed.Item1} is {named.Item2} after assignment."); 
Console.Write("Enter third number: "); 
var userInputThird = Console.ReadLine(); 
var noNamed2 = OddEven.FindOddEvenBySingleNumber(Convert.ToInt32(userInputThird)); 
Console.WriteLine($"Third Number:{noNamed2.Item1} is {noNamed2.Item2} using second noNamed tuple."); 
Console.WriteLine("Assigning 'second NoNamed' to 'Named'"); 
named = noNamed2; 
Console.WriteLine($"Second Number:{named.number} is {named.oddOrEven} after assignment."); 
```

前面代码片段的输出将如下所示：

![](img/00046.jpeg)

在前面的代码片段中，我们可以看到分配的元组的输出与分配的元组相同。

# 模式匹配

一般来说，模式匹配是一种在表达式中比较预定义格式的内容的方法。格式实际上是不同匹配的组合。

在 C# 7.0 中，模式匹配是一个特性。通过使用这个特性，我们可以在对象的类型之外实现方法分派。

模式匹配支持各种表达式；让我们通过代码示例讨论这些。

模式可以是常量模式：类型模式或变量模式。

# is 表达式

`is`表达式使得可以检查对象及其属性，并确定它是否满足模式：

```cs
public static string MatchingPatterUsingIs(object character) 
{ 
   if (character is null) 
   return $"{nameof(character)} is null. "; 
   if (character is char) 
   { 
      var isVowel = IsVowel((char) character) ? "is a
      vowel" : "is a consonent"; 
      return $"{character} is char and {isVowel}. "; 
   } 
   if (character is string) 
   { 
      var chars = ((string) character).ToArray(); 
      var stringBuilder = new StringBuilder(); 
      foreach (var c in chars) 
      { 
         if (!char.IsWhiteSpace(c)) 
         { 
         var isVowel = IsVowel(c) ? "is a vowel" : "is a
         consonent"; 
         stringBuilder.AppendLine($"{c} is char of string
         '{character}' and {isVowel}."); 
         } 
       } 

       return stringBuilder.ToString(); 
     } 
     throw new ArgumentException(
     "character is not a recognized data type.", 
     nameof(character)); 
} 
```

前面的代码没有展示任何花哨的东西，只是告诉我们输入参数是否是特定类型和元音或辅音。在这里我们简单地使用`is`运算符，告诉对象是否是相同类型。

`is`运算符([`goo.gl/79sLW5`](https://goo.gl/79sLW5))检查对象，如果对象是相同类型，则返回 true；如果不是，则返回 false。

在前面的代码中，当我们检查对象是否为字符串时，我们需要显式地将对象转换为字符串，然后将其传递给我们的实用方法`IsVowel()`。在前面的代码中，我们正在做两件事：第一是检查传入参数的类型，如果类型相同，我们就将其转换为所需的类型，并根据我们的情况执行操作。有时，当我们需要使用表达式编写更复杂的逻辑时，这会造成混淆。

C# 7.0 以微妙的方式解决了这个问题，使我们的表达式更简单。现在我们可以在表达式中直接声明一个变量，同时检查类型；请参阅以下代码：

```cs
if (character is string str) 
{ 
    var chars = str.ToArray(); 
    var stringBuilder = new StringBuilder(); 
    foreach (var c in chars) 
    { 
        if (!char.IsWhiteSpace(c)) 
        { 
            var isVowel = IsVowel(c) ? "is a vowel" : "is
            a consonent"; 
            stringBuilder.AppendLine($"{c} is char of
            string '{character}' and {isVowel}."); 
        } 
    } 

    return stringBuilder.ToString(); 
} 
```

在前面的代码中，更新了`is`表达式，它既测试变量又将其分配给所需类型的新变量。有了这个改变，就不需要像在以前的代码中那样显式地转换类型`((string) character)`。

让我们在前面的代码中添加一个条件：

```cs
if (character is int number) 
return $"{nameof(character)} is int {number}."; 
```

在前面的代码中，我们正在检查*对象*是否为`int`，这是一个*结构*。前面的条件完全正常，并产生了预期的结果。

以下是我们完整的代码：

```cs
private static IEnumerable<char> Vowels => new[] {'a', 'e', 'i', 'o', 'u'}; 

public static string MatchingPatterUsingIs(object character) 
{ 
    if (character is null) 
    return $"{nameof(character)} is null. "; 
    if (character is char) 
    { 
        var isVowel = IsVowel((char) character) ? "is a 
        vowel" : "is a consonent"; 
        return $"{character} is char and {isVowel}. "; 
    } 
    if (character is string str) 
    { 
        var chars = str.ToArray(); 
        var stringBuilder = new StringBuilder(); 
        foreach (var c in chars) 
        { 
            if (!char.IsWhiteSpace(c)) 
            { 
                var isVowel = IsVowel(c) ? "is a vowel" :
                "is a consonent"; 
                stringBuilder.AppendLine($"{c} is char of
                string '{character}' and {isVo 
            } 
        } 

        return stringBuilder.ToString(); 
    } 

    if (character is int number) 
    return $"{nameof(character)} is int {number}."; 

    throw new ArgumentException( 
    "character is not a recognized data type.", 
    nameof(character)); 
} 

private static bool IsVowel(char character) => Vowels.Contains(char.ToLower(character));
```

`is`表达式可以很好地处理值类型和引用类型。

在前面的代码示例中，只有当相应的表达式匹配结果为`true`时，变量`str`和`number`才会被赋值。

# switch 语句

我们已经在第 2 天讨论了`switch`语句。`switch`模式非常有用，因为它可以使用任何数据类型进行匹配，此外`case`提供了一种方式，因此它匹配了条件。

`match`表达式是相同的，但在 C# 7.0 中，这个特性以三种不同的方式得到了增强。让我们使用代码示例来理解它们。

# 常量模式

在 C#的早期版本中，`switch`语句只支持*常量*模式，在这种模式中，我们在`switch`中评估一些变量，然后根据常量情况进行条件调用。请参阅以下代码示例，我们试图检查`inputChar`是否具有特定长度，这是在`switch`中计算的：

```cs
public static string ConstantPatternUsingSwitch(params char[] inputChar) 
{ 
    switch (inputChar.Length) 
    { 

        case 0: 
            return $"{nameof(inputChar)} contains no
            elements."; 
        case 1: 
            return $"'{inputChar[0]}' and
            {VowelOrConsonent(inputChar[0])}."; 
        case 2: 
            var sb = new
            StringBuilder().AppendLine($"'{inputChar[0]}'
            and {VowelOrConsonent(inputChar[0])}."); 
            sb.AppendLine($"'{inputChar[1]}' and
            {VowelOrConsonent(inputChar[1])}."); 
            return sb.ToString(); 
        case 3: 
            var sb1 = new
            StringBuilder().AppendLine($"'{inputChar[0]}'
            and {VowelOrConsonent(inputChar[0])}."); 
            sb1.AppendLine($"'{inputChar[1]}' and
            {VowelOrConsonent(inputChar[1])}."); 
            sb1.AppendLine($"'{inputChar[2]}' and
            {VowelOrConsonent(inputChar[2])}."); 
            return sb1.ToString(); 
        case 4: 
            var sb2 = new
            StringBuilder().AppendLine($"'{inputChar[0]}'
            and {VowelOrConsonent(inputChar[0])}."); 
            sb2.AppendLine($"'{inputChar[1]}' and
            {VowelOrConsonent(inputChar[1])}."); 
            sb2.AppendLine($"'{inputChar[2]}' and
            {VowelOrConsonent(inputChar[2])}."); 
            sb2.AppendLine($"'{inputChar[3]}' and
            {VowelOrConsonent(inputChar[3])}."); 
            return sb2.ToString(); 
        case 5: 
            var sb3 = new
            StringBuilder().AppendLine($"'{inputChar[0]}'
            and {VowelOrConsonent(inputChar[0])}."); 
            sb3.AppendLine($"'{inputChar[1]}' and
            {VowelOrConsonent(inputChar[1])}."); 
            sb3.AppendLine($"'{inputChar[2]}' and
            {VowelOrConsonent(inputChar[2])}."); 
            sb3.AppendLine($"'{inputChar[3]}' and
            {VowelOrConsonent(inputChar[3])}."); 
            sb3.AppendLine($"'{inputChar[4]}' and
            {VowelOrConsonent(inputChar[4])}."); 
            return sb3.ToString(); 
            default: 
            return $"{inputChar.Length} exceeds from
            maximum input length."; 
    } 
} 
```

在前面的代码中，我们的主要任务是检查`inputChar`是元音还是辅音，我们在这里要做的是首先评估`inputChar`的长度，然后根据需要执行操作，这会导致更复杂条件的更多工作/代码。

# 类型模式

通过引入*类型*模式，我们可以克服我们在*常量*模式（在前一节中）中遇到的问题。请考虑以下代码：

```cs
public static string TypePatternUsingSwitch(IEnumerable<object> inputObjects) 
{ 
    var message = new StringBuilder(); 
    foreach (var inputObject in inputObjects) 
    switch (inputObject) 
        { 
            case char c: 
                message.AppendLine($"{c} is char and
                {VowelOrConsonent(c)}."); 
                break; 
            case IEnumerable<object> listObjects: 
                foreach (var listObject in listObjects)

                message.AppendLine(MatchingPatterUsingIs(
                listObject)); 
                break; 
            case null: 
                break; 
        } 
    return message.ToString(); 
} 
```

在前面的代码中，现在根据类型模式执行操作变得更容易。

# 在`case`表达式中的`when`子句

通过在`case`表达式中引入`when`子句，您可以在表达式中执行特殊操作；请参阅以下代码：

```cs
public static string TypePatternWhenInCaseUsingSwitch(IEnumerable<object> inputObjects) 
{ 
    var message = new StringBuilder(); 
    foreach (var inputObject in inputObjects) 
    switch (inputObject) 
        { 
            case char c: 
                message.AppendLine($"{c} is char and
                {VowelOrConsonent(c)}."); 
                break; 
            case IEnumerable<object> listObjects when
                listObjects.Any(): 
                foreach (var listObject in listObjects) 
                message.AppendLine(MatchingPatterUsingIs
                (listObject)); 
                break; 
            case IEnumerable<object> listInlist: 
                break; 
            case null: 
                break; 
        } 
    return message.ToString(); 
} 
```

在前面的代码中，`case`与`when`确保只有在`listObjects`有一些值时才执行操作。

`case`语句要求每个`case`都以`break`、`return`或`goto`结束。

# 本地函数

在之前的版本中，可以使用匿名方法来实现函数和动作，但仍然存在一些限制：

+   泛型

+   `ref`和`out`参数

+   `params`

本地函数的特点是在块范围内声明。这些函数非常强大，并且具有与任何其他普通函数相同的功能，但不同之处在于它们在声明它们的块内部范围内。

考虑以下代码示例：

```cs
public static string FindOddEvenBySingleNumber(int number) => IsOddNumber(number) ? "Odd" : "Even";
```

在前面的代码中，方法`FindOddEvenBySingleNumber()`只是简单地返回大于 1 的数字为*奇数*或*偶数*。这使用了一个私有方法`IsOddNumber()`，如下所示：

```cs
private static bool IsOddNumber(int number) => number >= 1 && number % 2 != 0; 
```

方法`IsOddNumber()`是一个私有方法，只能在声明它的类中使用。因此，它的范围在类内部，而不是在代码块内部。

让我们看一个本地函数的代码示例：

```cs
public string FindOddEvenBySingleNumberUsingLocalFunction(int someInput) 
{ 
    //Local function, scoped within
    FindOddEvenBySingleNumberUsingLocalFunction 
    bool IsOddNumber(int number) 
    { 
        return number >= 1 && number % 2 != 0; 
    } 

    return IsOddNumber(someInput) ? "Odd" : "Even"; 
} 
```

在前面的代码中，本地函数`IsOddNumber()`执行的操作与上一节中的`private`方法相同。但是在这里，`IsOddNumber()`的范围在方法`FindOddEvenBySingleNumberUsingLocalFunction()`内。因此，它在此代码块之外将不可用。

# 文字改进

在使用文字时，我们可以考虑声明各种变量常量，有时这些变量对于方法的生命周期非常重要，因为这些变量对于方法或做出任何决定非常重要。并且会导致错误的决策，因为对数字常量的误读。为了克服这种混淆，C# 7.0 引入了两个新功能，二进制文字和数字分隔符。

# 二进制文字

二进制数字对于执行复杂操作非常重要。二进制数字的常量可以声明为*0b<binaryvalue>*，其中 0b 告诉我们这是一个二进制文字，而二进制值是您的十进制数字的值。以下是一些例子：

```cs
//Binary literals
public const int Nineteen = 0b00010011; 
public const int Ten = 0b00001010; 
public const int Four = 0b0100; 
public const int Eight = 0b1000; 
```

# 数字分隔符

引入了数字分隔符后，我们可以轻松阅读长数字、二进制数字。数字分隔符可以用于数字和二进制数字。对于二进制数字，数字分隔符，即下划线(`_`)，适用于位模式，对于数字，它可以出现在任何地方，但最好将 1,000 作为分隔符。看一下以下例子：

```cs
//Digit separator - Binary numbers 
public const int Hundred = 0b0110_0100; 
public const int Fifty = 0b0011_0010; 
public const int Twenty = 0b0001_0100; 
//Numeric separator 
public const long Billion = 100_000_0000; 

```

数字分隔符也可以用于十进制、浮点和双精度类型。

<p>以下是作为 C# 7.1 语言特性随 Visual Studio 2017 更新 3 一起发布的新功能，我们将根据：[`github.com/dotnet/roslyn/blob/master/docs/Language%20Feature%20Status.md`](https://github.com/dotnet/roslyn/blob/master/docs/Language%20Feature%20Status.md)讨论所有功能

有关 Visual Studio 2017 新版本的更多信息，请参考：[`www.visualstudio.com/en-us/news/releasenotes/vs2017-relnotes`](https://www.visualstudio.com/en-us/news/releasenotes/vs2017-relnotes)

如果您想了解如何设置您现有的项目或使用 C# 7.0 的新项目 - 那么您不用担心，Visual Studio 2017 更新 3 会帮助您。每当您开始使用 C# 7.1 的新功能时，您需要遵循以下步骤：

1.  Visual Studio 将警告现有版本的支持，并建议升级您的项目，如果您想使用 C# 7.1 的新功能。

1.  只需点击黄色灯泡，选择最适合您需求的选项，您就可以使用新的 C# 7.1 了。

以下图片告诉您准备好 C# 7.1 的两个步骤：

![](img/00047.jpeg)

让我们开始讨论 C# 7.1 语言的新功能：

# 异步主函数

C# 7.1 的一项新功能，使应用程序的入口点即`Main`成为可能。异步主函数使`main`方法可以等待，这意味着`Main`方法现在是异步的，可以获得`Task`或`Task<int>`。有了这个功能，以下是有效的入口点：

```cs
static Task Main()
{
    //stuff goes here
}
static Task<int> Main()
{
    //stuff goes here
}
static Task Main(string[] args)
{
    //stuff goes here
}
static Task<int> Main(string[] args)
{
    //stuff goes here
}
```

# 在使用新签名时有一些限制。

+   您可以使用这些新签名的入口点，如果没有先前签名的重载存在，那么这些标记为有效，这意味着如果您使用现有的入口点。

```cs
public static void Main()
{
    NewMain().GetAwaiter().GetResult();
}
private static async Task NewMain()
{
    //async stuff goes here
}
```

+   这并不是强制将入口点标记为异步的，这意味着您仍然可以使用现有的异步入口点：

```cs
private static void Main(string[] args)
{
    //stuff goes here
}
```

可能会有更多的入口点用法可以整合到应用程序中 - 请参考此功能的官方文档：[`github.com/dotnet/csharplang/blob/master/proposals/async-main.md`](https://github.com/dotnet/csharplang/blob/master/proposals/async-main.md)

# 默认表达式

C# 7.1 引入了一个新的表达式，即默认文字。引入这个新文字后，表达式可以隐式转换为任何类型，并产生类型的默认值。

新的默认文字与旧的`default(T)`不同。早期的`default`转换了`T`的目标类型，但新的可以转换任何类型。

```cs
default:
```

```cs
//Code removed
case 8:
    Clear();
    WriteLine("C# 7.1 feature: default expression");
    int thisIsANewDefault = default;
    var thisIsAnOlderDefault = default(int);
    WriteLine($"New default:{thisIsANewDefault}. Old
    default:{thisIsAnOlderDefault}");
    PressAnyKey();
    break;
//Code removed
```

在上述代码中，当我们写`int thisIsANewDefault = default;`时，这是 C# 7.1 中有效的表达式，它隐式地将表达式转换为 int 类型，并将默认值 0（零）赋给`thisIsANewDefault`。这里值得注意的是，默认文字隐式地检测`thisIsANewDefault`的类型并设置值。另一方面，我们需要明确告诉目标类型在表达式`var thisIsAnOlderDefault = default(int);`中设置默认值。

上述代码生成以下输出：

![](img/00048.gif)

有多种实现新的默认文字，因此，您可以在以下情况下使用相同的：

# 成员变量

新的`default`表达式可以用于为变量分配默认值，以下是各种方式：

```cs
int thisIsANewDefault = default;
int thisIsAnOlderDefault = default(int);
var thisIsAnOlderDefaultAndStillValid = default(int);
var thisIsNotValid = default; //Not valid, as we cannot assign default to implicit-typed variable
```

# 常量

与变量类似，使用 default 我们可以声明常量，以下是各种方式：

```cs
const int thisIsANewDefaultConst = default; //valid
const int thisIsAnOlderDefaultCont = default(int); //valid
const int? thisIsInvalid = default; //Invalid, as nullable cannot be declared const
```

还有更多的情景可以使用这个新的默认文字，比如方法中的可选参数，更多信息请参考：[`github.com/dotnet/csharplang/blob/master/meetings/2017/LDM-2017-03-07.md`](https://github.com/dotnet/csharplang/blob/master/meetings/2017/LDM-2017-03-07.md)

# 推断元组名称

随着这个新功能的引入，您不需要显式声明元组候选名称。我们在之前的*Tuples and Deconstructions*部分讨论了元组。推断元组名称功能是 C# 7.0 引入的元组值的扩展。

要使用这个新功能，您需要更新之前在*Tuple*部分安装的`ValueTuple`的 NuGet 包。要更新 NuGet 包，转到*NuGet 包管理器*，点击更新选项卡，然后点击更新最新版本。以下截图提供了完整信息：

![](img/00049.jpeg)

以下代码片段显示了声明元组的各种方式：

```cs
public static void InferTupleNames(int num1, int num2)
{
    (int, int) noNamed = (num1, num2);
    (int, int) IgnoredName = (A:num1, B:num2);
    (int a, int b) typeNamed = (num1, num2);
    var named = (num1, num2);
    var noNamedVariation = (num1, num1);
    var explicitNaming = (n: num1, num1);
    var partialnamed = (num1, 5);
}
```

上述代码是不言自明的，元组`noNamed`没有任何成员名称，可以使用`item1`和`item2`进行访问。同样，在元组`IgnoredName`中，所有定义的成员名称将被忽略，因为声明中没有定义成员名称。以下代码片段讲述了如何访问各种元组的完整故事：

```cs
public static void InferTupleNames(int num1, int num2)
{
    (int, int) noNamed = (num1, num2);
    Console.WriteLine($"NoNamed:{noNamed.Item1},
    {noNamed.Item2}");
    (int, int) ignoredName = (A:num1, B:num2);
    Console.WriteLine($"IgnoredName:{ignoredName.Item1}
    ,{ignoredName.Item2}");
    (int a, int b) typeNamed = (num1, num2);
    Console.WriteLine($"typeNamed using default member-
    names:{typeNamed.Item1}
    {typeNamed.Item2}");
    Console.WriteLine($"typeNamed:{typeNamed.a},
    {typeNamed.b}");
    var named = (num1, num2);
    Console.WriteLine($"named using default member-names
    :{named.Item1},{named.Item2}");
    Console.WriteLine($"named:{named.num1},{named.num2}");
    var noNamedVariation = (num1, num1);
    Console.WriteLine($"noNamedVariation:
    {noNamedVariation.Item1},{noNamedVariation.Item2}");
    var explicitNaming = (n: num1, num1);
    Console.WriteLine($"explicitNaming:{explicitNaming.n},
    {explicitNaming.num1}");
    var partialnamed = (num1, 5);
    Console.WriteLine($"partialnamed:{partialnamed.num1},
    {partialnamed.Item2}");
}
```

上述代码产生以下输出：

![](img/00050.gif)

还有更多变化可以使用这个新功能，更多信息，请参考：[`github.com/dotnet/roslyn/blob/master/docs/features/tuples.md`](https://github.com/dotnet/roslyn/blob/master/docs/features/tuples.md)

# 其他预计发布的功能

在最终发布的 C# 7.1 编程语言中将会有更多功能，除了之前的功能外，以下是遇到 bug 或部分实现的功能。

# 泛型模式匹配

该模式匹配与泛型提议在这里：[`github.com/dotnet/csharplang/blob/master/proposals/generics-pattern-match.md`](https://github.com/dotnet/csharplang/blob/master/proposals/generics-pattern-match.md) 作为 C# 7.1 的新功能，遇到了一个 bug，可以在这里看到：[`github.com/dotnet/roslyn/issues/16195`](https://github.com/dotnet/roslyn/issues/16195)

该功能的实现将基于`as`运算符，详细信息请参见：[`github.com/dotnet/csharplang/blob/master/spec/expressions.md#the-as-operator`](https://github.com/dotnet/csharplang/blob/master/spec/expressions.md#the-as-operator)

# 引用程序集

引用程序集功能尚未纳入 IDE 中，您可以参考：[`github.com/dotnet/roslyn/blob/master/docs/features/refout.md`](https://github.com/dotnet/roslyn/blob/master/docs/features/refout.md) 这里获取更多详细信息。

# 动手练习

回答以下问题，涵盖了今天学习的概念：

1.  什么是`ValueTuple`类型？

1.  ValueTuples 是可变的；通过示例证明。

1.  创建一个包含 10 个元素的`ValueTuple`。

1.  创建一个如下所示的用户定义类 employee，然后编写一个程序来解构用户定义的类型：

```cs
public class employee
{
public Guid EmplId { get; set; }
public String First { get; set; }
public string Last { get; set; }
public char Sex { get; set; }
public string DepartmentId { get; set; }
public string Designation { get; set; }
}
```

1.  创建一个使用数字分隔符的各种常量的类，并将这些常量实现到函数`ToDecimal()`和`ToBinary()`中。

1.  本地函数是什么？它们与私有函数有什么不同？

1.  使用通用本地函数重写`OddEven`程序。

1.  使用`switch`语句中的类型模式重写`OddEven`程序。

1.  编写一个程序，利用 C# 7.1 语言的推断元组名称特性来找出`OddEven`。

1.  默认表达式（C# 7.1）是什么，通过程序详细说明？

# 重温第 03 天

今天，我们讨论了 C# 7.0 中引入的所有新功能，并提供了代码示例。我们还了解了这些功能的重要点和用法。

我们讨论了 ValueTuples 如何帮助我们收集数据信息以及我们期望从方法中获得多个输出的情况。`ValueTuple`的一个优点是它是可变的和`ValueType`。`System.ValueTuple`提供了一些`public`和`static`方法，我们可以利用这些方法实现许多复杂的场景。

然后我们了解了模式匹配的优势和能力；这有助于编码人员执行各种复杂的条件场景，这在 C#语言的先前版本中是不可能的。类型模式和`case`语句中的`when`子句使这个功能非常出色。

本地函数是 C# 7.0 中引入的最重要的功能之一。它们在需要使我们的代码对称的场景中非常有帮助，这样你就可以完美地阅读代码，当我们不需要在方法外部使用方法，或者我们不需要重用在块范围内需要的操作时。

随着字面改进，现在我们可以将二进制数字声明为常量，并像使用其他变量一样使用它们。添加数字分隔符下划线（`_`）的能力使这个功能更加有用。

最后，我们已经了解了作为 Visual Studio 更新 3 的一部分发布的 C# 7.1 语言的新功能。

早些时候，计划中有更多的功能计划发布，但最终发布的版本却包含了之前的新功能。下一个版本已经计划中，还有更强大的功能尚未推出。您可以在这里查看计划和下一个版本的功能列表：[`github.com/dotnet/csharplang/tree/master/proposals`](https://github.com/dotnet/csharplang/tree/master/proposals)。
