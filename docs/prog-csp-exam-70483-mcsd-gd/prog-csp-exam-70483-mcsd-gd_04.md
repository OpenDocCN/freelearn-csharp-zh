# 第四章：实现程序流程

本章重点介绍如何在 C# 中管理程序流程。换句话说，本章将帮助您了解程序如何通过 C# 中可用的语句控制、验证输入和输出参数并做出决策。我们将涵盖各种布尔表达式，例如 If/Else 和 Switch，它们根据特定条件控制代码的流程。我们还将评估各种运算符，例如条件运算符和相等运算符（`<`, `>`, `==`），它们都控制代码的流程。我们将关注如何通过集合（使用 `for` 循环、`while` 循环等）和显式跳转语句进行迭代。

本章将涵盖以下主题。

+   理解运算符

+   理解条件/选择语句

+   迭代语句

# 技术要求

本章的练习可以使用 Visual Studio 2012 或更高版本以及 .NET Framework 2.0 或更高版本进行练习。然而，任何从版本 7.0 及以上版本的新 C# 功能都需要您拥有 Visual Studio 2017。

如果您没有这些产品的许可证，您可以下载 Visual Studio 2017 的社区版本：[`visualstudio.microsoft.com/downloads/`](https://visualstudio.microsoft.com/downloads/)。

本章的示例代码可以在本书的 GitHub 仓库中找到：[`github.com/PacktPublishing/Programming-in-C-sharp-Exam-70-483-MCSD-Guide/tree/master/Chapter04`](https://github.com/PacktPublishing/Programming-in-C-sharp-Exam-70-483-MCSD-Guide/tree/master/Chapter04)。

# 理解运算符

在我们深入这个主题之前，让我们了解运算符和操作数是什么。这两个是我们将在本书的这一部分使用的重要术语：

+   运算符是应用于表达式或语句中的一个或多个操作数的编程元素。

+   操作数是可以被操作的对象。

C# 提供不同类型的运算符，例如一元运算符（`[增量运算符] ++`, `new`），它接受一个操作数，二元算术类型的运算符（`+`, `-`, `*`, `/`），关系类型（`>` ,`<`, `<=`, `>=`），相等类型（`=`, `!=`），以及位移类型（`>>`, `<<`），所有这些都在两个操作数之间使用。C# 还提供了一种三元运算符，它接受三个操作数（`?:`）。

# 一元运算符

只需要一个操作数的运算符称为**一元运算符**。它们可以执行增量、减量、取反等操作。它们也可以应用于操作数之前（前缀）或之后（后缀）。

下表列出了几个一元运算符。`x` 在左侧列中是应用运算符的操作数：

| **表达式** | **描述** |
| --- | --- |
| `+x` | **恒等性**：此运算符可以用作一元或二元运算符。如果用于数值，则返回该值。如果应用于两个数值操作数，则返回操作数的总和。在字符串上，它连接两个操作数。 |
| `-x` | **取反**: 这个运算符可以用作一元或二元运算符。对数值类型应用此运算符会导致操作数的数值取反。 |
| `!x` | **逻辑取反**: 这个运算符取反操作数。它应用于布尔操作数，并在操作数为假时返回。 |
| `~x` | **位取反**: 通过反转每个位来产生操作数的补码。 |
| `++x` | **前置递增**: 这是一个递增运算符，可以出现在操作数之前或之后。当作为前缀时，结果放在递增之后。如果作为后缀，结果放在递增之前。 |
| `--x` | **前置递减**: 这是一个递减运算符，可以出现在操作数之前或之后。当作为前缀时，结果放在递减之后。如果作为后缀，结果放在递减之前。 |

在以下代码中，我们将声明一些变量并使用它们来展示前面运算符的示例：

```cs
int firstvalue = 5; 
int secondvalue = 6;
string firststring = "Hello ";
string secondstring = "World";
```

`+` 和 `-` 可以与单个操作数或多个操作数一起使用。当与多个整数类型的操作数一起使用时，它们要么对操作数求和，要么得到差值。`+` 运算符也可以与字符串类型的操作数一起使用。在这种情况下，它将连接两个字符串。字符串和运算符始终是二进制运算符：

```cs
//'+' operator
Console.WriteLine(+firstvalue); // output: 5
Console.WriteLine(firstvalue + secondvalue); // output: 11
Console.WriteLine(firststring + secondstring); // output: Hello World
//'-' operator
Console.WriteLine(-firstvalue); // output: -5
Console.WriteLine(firstvalue - secondvalue); // output = -1
```

`!` 运算符与布尔操作数配合良好，它产生逻辑取反；也就是说，真变为假，而 `~` 运算符与位运算符配合使用。在以下示例中，显示了一个二进制位表示及其位取反。我们取一个整数值，将其转换为二进制值，然后使用 `~` 运算符取反，并以二进制格式显示：

```cs
//'!' operator
Console.WriteLine(!true); 

//output : false

//'~' operator
Console.WriteLine("'~' operator");
int digit = 60;
Console.WriteLine("Number is : {0} and binary form is {1}:", digit, IntToBinaryString(digit));
int digit1 = ~digit;
Console.WriteLine("Number is : {0} and binary form is {1}:", digit1, Convert.ToString(digit1, 2));

//Output: 
Number is : 60 and binary form is 111100:
Number is : -61 and binary form is 11111111111111111111111111000011
```

`++` 和 `--` 运算符，当应用于整数操作数时，分别对操作数执行递增或递减操作。这些可以应用于操作数的前面或后面。以下示例显示了后置和前置递增和递减运算符。前置在显示之前产生结果，后置在显示之后产生结果：

```cs
// '++' Operator
Console.WriteLine(++firstvalue); // output: 6
// '--' Operator
Console.WriteLine(--firstvalue); // output: 5
// '++' Operator
Console.WriteLine(firstvalue++); // output: 5
Console.WriteLine(firstvalue--); // output: 6
```

# 关系运算符

如其名所示，关系运算符测试或定义两个操作数之间的关系，例如，如果第一个操作数小于第二个操作数，或者大于等于它。这些运算符应用于数值操作数。

下表列出了几个二进制运算符：

| **表达式** | **描述** |
| --- | --- |
| `<` | 定义为小于运算符。用作 `X < Y`。如果第一个操作数小于第二个操作数，则返回 true。 |
| `>` | 定义为大于运算符。用作 `X > Y`。如果第一个操作数大于第二个操作数，则返回 true。 |
| `<=` | 小于或等于运算符。用作 `X <= Y`。 |
| `>=` | 大于或等于运算符。用作 `X >= Y`。 |

我们将使用前面示例中定义的相同变量来理解这些关系运算符。在这里，我们正在尝试找出`firstvalue`是否小于`secondvalue`或`firstvalue`是否大于`secondvalue`：

```cs
// '<' Operator
Console.WriteLine(firstvalue < secondvalue); 
// output = true

// '>' Operator
Console.WriteLine(firstvalue > secondvalue); 
// output = false

// '>=' Operator
Console.WriteLine(secondvalue >= firstvalue); 
// output = true

// '<=' Operator
Console.WriteLine(firstvalue <= secondvalue); 
// output = true
```

# 相等运算符

相等运算符是一种二元运算符，需要两个操作数。因为它们检查操作数的相等性，所以这些也可以称为关系运算符。

以下表格列出了可用的相等运算符：

| **表达式** | **描述** |
| --- | --- |
| `==` | 这适用于预定义值类型。定义为相等运算符。用作`X == Y`。如果第一个操作数等于第二个操作数，则返回`true`。 |
| `!=` | 定义为不等运算符。用作`X! = Y`。如果操作数不相等，则返回`true`。 |

我们将使用前面示例中创建的相同变量来尝试理解相等运算符。在这里，我们正在尝试检查`firstvalue`是否等于`secondvalue`或不相等：

```cs
//using variables created earlier.
// '==' Operator
Console.WriteLine(secondvalue == firstvalue); // output = false
// '!=' Operator
Console.WriteLine(firstvalue != secondvalue); // output = true
```

# 位移运算符

位移运算符是另一种类型的二元运算符。它们接受两个整数操作数，并根据指定的位数将位左移或右移。

以下表格列出了可用的位移运算符：

| **表达式** | **描述** |
| --- | --- |
| `<<` | 这是一个二元运算符的示例，允许你将第一个操作数左移由第二个操作数指定的位数。第二个操作数必须是整数类型。 |
| `>>` | 这是一个二元运算符的示例，允许你将第一个操作数右移由第二个操作数指定的位数。第二个操作数必须是整数类型。 |

在以下示例中，程序接受一个整数操作数，并将其左移或右移 1 位。位移操作适用于二元运算符，因此，为了我们的理解，我编写了一个方法，将整数转换为二进制格式并显示。当我们向程序传递整数`9`，即`i`，并使用`>>`运算符时，其二进制字符串右移 1 位，并显示结果。当`1001`右移时，它变为`100`：

```cs
public static string IntToBinaryString(int number)
{
    const int mask = 1;
    var binary = string.Empty;
    while (number > 0)
    {
        // Logical AND the number and prepend it to the result string
        binary = (number & mask) + binary;
        number = number >> 1;
    }
    return binary;
}

// '>>' Operator
Console.WriteLine("'>>' operator");
int number = 9;
Console.WriteLine("Number is : {0} and binary form is {1}:", number, IntToBinaryString(number));
number = number >> 1;
Console.WriteLine("Number is : {0} and binary form is {1}:", number, IntToBinaryString(number));

//Output: 
//Number is : 9 and binary form is 1001
//Number is : 4 and binary form is 100

// '<<' Operator
Console.WriteLine("'<<' operator");
Console.WriteLine("Number is : {0} and binary form is {1}:", number, IntToBinaryString(number));
number = number << 1;
Console.WriteLine("Number is : {0} and binary form is {1}:", number, IntToBinaryString(number));

//Output: 
//Number is : 4 and binary form is 100
//Number is : 8 and binary form is 1000
```

# 逻辑、条件和 null 运算符

C# 允许你使用`OR`（`||`）、`AND`（`&&`）或`XOR`（`^`）将上述运算符组合起来。这些应用于表达式中的两个操作数。

以下表格列出了逻辑、条件和`null`运算符：

| **表达式** | **描述和示例** |
| --- | --- |
| 逻辑或 (`&#124;`) | 此运算符计算两个操作数，如果两个操作数都为`false`，则返回`false`。 |
| 逻辑与 (`&`) | 此运算符有两种形式：一元地址运算符或二元逻辑运算符。当用作一元地址运算符时，它返回操作数的地址。如果用作二元运算符，它评估两个操作数，如果两个操作数都为`true`，则返回`true`；否则，它将返回`false`。 |
| 条件 AND (`&&`) | 当需要评估两个布尔操作数时使用此条件运算符。当应用时，计算两个操作数，如果两个操作数都为 `true`，则返回 `true`。如果第一个操作数返回 `false`，则条件运算符不会评估另一个操作数。这也被称为*短路*逻辑 `AND` 运算符。 |
| 条件 OR (`&#124;&#124;`) | 这也称为*短路*逻辑 `OR` 运算符。条件 `OR` 运算符评估两个布尔操作数，如果任一操作数为 `true`，则返回 `true`。如果第一个操作数返回 `true`，则不会评估第二个操作数。 |
| 逻辑异或 (`^`) | 此运算符对于整型类型是按位异或，对于布尔类型是逻辑异或和。当应用时，它计算两个操作数，如果任一操作数为 `true`，则返回 `true`；否则返回 `false`。 |
| 空合并 (`??`) | 空合并运算符计算两个操作数并返回非 `null` 的操作数。其用法如下：`int y = x ?? 1;`。在这种情况下，如果 `x` 为 `null`，则 `y` 被分配值为 `1`；否则，`y` 被分配值为 `x`。 |
| 三元运算符 (`?:`) | 条件运算符也称为三元运算符，用于评估布尔表达式。条件是 `? true value : false value`。如果条件为 `true`，则运算符返回 `true value`，但如果条件为 `false`，则运算符返回 `false value`。三元运算符支持嵌套表达式或运算符，这也称为*右结合性*。 |

以下代码将使我们能够详细了解每个语句。最初，我们将定义所需的变量和方法，然后对每个语句进行处理。以下代码可在 GitHub 上找到。链接在*技术要求*部分提供：

```cs
int firstvalue = 5; 
int secondvalue = 6;
int? nullvalue = null;
private bool SecondOperand(bool result)
{
    Console.WriteLine("SecondOperand computed");
    return result;
}
private bool FirstOperand(bool result)
{
    Console.WriteLine("FirstOperand computed");
    return result;
}
```

在以下代码块中，逻辑 `OR` (`|`) 展示了 `|` 运算符的用法。以下代码块中有两个布尔表达式在运行时评估并返回 `true` 或 `false`。此运算符始终返回 `true`，除非两个操作数都返回 `false`：

```cs
//LOGICAL OR (|)
Console.WriteLine((firstvalue > secondvalue) | (firstvalue < secondvalue)); 
// output : true
Console.WriteLine((firstvalue < secondvalue) | (firstvalue < secondvalue)); 
// output : true
Console.WriteLine((firstvalue < secondvalue) | (firstvalue > secondvalue)); 
// output : true
Console.WriteLine((firstvalue > secondvalue) | (firstvalue > secondvalue)); 
// output : false
```

在以下代码块中，逻辑 `AND` 展示了 `&` 运算符的用法。逻辑 `AND` 评估两个操作数，如果两个操作数都评估为 `true`，则返回 `true`；否则返回 `false`：

```cs
//LOGICAL AND (&)
Console.WriteLine(FirstOperand(true) & SecondOperand(true)); 
// output : FirstOperand computed, SecondOperand computed, true
Console.WriteLine(FirstOperand(false) & SecondOperand(true)); 
// output : FirstOperand computed, SecondOperand computed, false
Console.WriteLine(FirstOperand(true) & SecondOperand(false)); 
// output : FirstOperand computed, SecondOperand computed, false
Console.WriteLine(FirstOperand(false) & SecondOperand(false)); 
// output : FirstOperand computed, SecondOperand computed, false
```

在以下代码块中，条件 `AND` (`&&`) 说明了 `&&` 运算符。此运算符评估第一个操作数，如果它是 `true`，则评估第二个操作数。否则，它返回 `false`：

```cs
//CONDITIONAL AND (&&)
Console.WriteLine(FirstOperand(true) && SecondOperand(true)); 
// output = FirstOperand computed, SecondOperand computed, true
Console.WriteLine(FirstOperand(false) && SecondOperand(true)); 
// output = FirstOperand computed, false
Console.WriteLine(FirstOperand(true) && SecondOperand(false)); 
// output = FirstOperand computed, false
Console.WriteLine(FirstOperand(false) && SecondOperand(false)); 
// output = FirstOperand computed, false
```

在以下代码块中，条件 `OR` (`||`) 说明了 `||` 运算符。此运算符如果任一操作数为 `true`，则返回 `true`；否则返回 `false`：

```cs
//CONDITIONAL OR (||)
Console.WriteLine(FirstOperand(true) || SecondOperand(true)); 
// output = FirstOperand computed, true
Console.WriteLine(FirstOperand(false) || SecondOperand(true)); 
// output = FirstOperand computed, SecondOperand computed, true
Console.WriteLine(FirstOperand(true) || SecondOperand(false)); 
// output = FirstOperand computed, true
Console.WriteLine(FirstOperand(false) || SecondOperand(false)); 
// output = FirstOperand computed, SecondOperand computed, false 
```

在以下代码中，逻辑 `XOR` (`^`) 解释了 `^` 运算符在 `bool` 操作数上的用法。如果其中一个操作数是 `true`，则返回 `true`。这与逻辑 `OR` 运算符类似：

```cs
//LOGICAL XOR (^)
Console.WriteLine(FirstOperand(true) ^ SecondOperand(true)); 
// output = FirstOperand computed, SecondOperand computed, false
Console.WriteLine(FirstOperand(false) ^ SecondOperand(true)); 
// output = FirstOperand computed, SecondOperand computed,true
Console.WriteLine(FirstOperand(true) ^ SecondOperand(false)); 
// output = FirstOperand computed, SecondOperand computed,true
Console.WriteLine(FirstOperand(false) ^ SecondOperand(false)); 
// output = FirstOperand computed, SecondOperand computed,false
```

在这里，我们将探讨空合并和三元运算符。空合并运算符 `??` 用于在返回其值之前检查操作数是否为 `null`。如果第一个操作数不是 `null`，则返回其值；否则，返回第二个操作数。这也可以以嵌套形式使用。

三元运算符 (`?:`) 用于评估表达式。如果它是 `true`，则返回 `true-value`；否则，返回 `false-value`：

```cs
//Null Coalescing (??)
Console.WriteLine(nullvalue ?? firstvalue);// output : 5

//Ternary Operator (? :)
Console.WriteLine((firstvalue > secondvalue) ? firstvalue : secondvalue);// output : 6
Console.WriteLine((firstvalue < secondvalue) ? firstvalue : secondvalue);// output : 5
```

# 理解条件/选择语句

C# 提供了多个条件/选择语句，帮助我们在整个编程过程中做出决策。我们可以使用我们在上一节中学到的所有运算符与这些语句一起使用。这些语句帮助程序根据表达式评估为 `true` 或 `false` 来采取特定的流程。这些语句是 C# 中最广泛使用的语句。

以下表格列出了可用的条件/选择语句：

| **Expression** | **描述** |
| --- | --- |
| `If..else` | 如果语句评估提供的表达式。如果它是 `true`，则执行这些语句。如果它是 `false`，则执行 `else` 语句。 |
| `Switch..case..default` | Switch 语句评估特定表达式，如果模式与匹配表达式匹配，则执行 switch 部分。 |
| `break` | `break` 允许我们终止控制流并继续执行下一个语句。 |
| `goto` | `goto` 用于在表达式评估为 `true` 时将控制转移到特定标签。 |

在以下小节中，我们将详细介绍这些语句。

# if...else

在用户希望满足条件时执行特定代码块的场景中，使用 `if` 语句简单且容易。C# 提供了广泛使用的 `if` 语句，使我们能够实现所需的功能。

`If` (`true`) then-语句 `Else` (`false`) else-语句。以下为 `If / Else` 语句的一般语法：

```cs
If(Boolean expression)
{
    Then statements;  //these are executed when Boolean expression is true
}
Else
{
    Else statements; //these are executed when Boolean expression is false
}
```

当布尔表达式评估为 `true` 时，执行 `then-statements`；当布尔表达式评估为 `false` 时，执行 `else-statements`。当布尔表达式评估为 `true` 或 `false` 时，程序允许你执行单个或多个语句。然而，多个语句需要用花括号 `{}` 括起来。这将确保所有语句在一个上下文中按顺序执行。这也被称为 **代码块**。对于单个语句，这些括号是可选的，但从代码可读性的角度来看，它们是推荐的。此外，我们需要理解变量的作用域仅限于它们被定义的代码块内。

else 语句是可选的。如果没有提供，程序将评估布尔表达式并执行`then-statement`。在任何给定的时间，`if-else`语句的`then-statements`或`else-statements`中只有一个会被执行。

让我们看看几个例子。在下面的代码块中，我们已经将条件变量设置为`true`，所以当 if 语句中的布尔表达式被评估时，它返回`true`，并执行代码块（`then-statement`）。`Else-statement`被忽略：

```cs
bool condition = true;
if (condition) 
{
    Console.WriteLine("Then-Statement executed");
}
else
{
    Console.WriteLine("Else-Statement executed");
}
//output: Then-Statement executed
```

在以下场景中，如果语句不包括 else 部分，当布尔表达式评估为`true`时，默认情况下将执行`then-statements`：

```cs
if (condition) 
{
    Console.WriteLine("Then-Statement without an Else executed");
}
//output: Then-Statement without an Else executed
```

C#还允许嵌套的`if`和嵌套的`else`语句。在下面的代码中，我们将看到如何在程序中使用嵌套的 if 语句。

当条件`1`评估为`true`时，默认情况下，将执行条件`1`的`then-statements`。同样，当条件`2`评估为`true`时，将执行条件`2`的`then-statements`：

```cs
int variable1 = 15;
int variable2 = 10;

if (variable1 > 10)//Condition 1
{
    Console.WriteLine("Then-Statement of condition 1 executed");
     if (variable2 < 15) //Condition 2
     {
          Console.WriteLine("Then-Statement of condition 2 executed");
     }
     else
     {
          Console.WriteLine("Else-Statement of condition 2 executed");
     }
}
else
{
    Console.WriteLine("Then-Statement condition 1 executed");
}
//Output: 
Then-Statement of condition 1 executed
Then-Statement of condition 2 executed
```

我们也可以在`Else`语句中定义嵌套的 if。例如，用户想要找出输入的字符是否是元音，如果是，则打印它。下面的代码块说明了如何使用多个 if 语句。程序检查输入的字符是否是元音，并打印结果：

```cs
Console.Write("Enter a character: ");
char ch = (char)Console.Read();
if (ch.Equals('a'))
{
    Console.WriteLine("The character entered is a vowel and it is 'a'.");
}
else if (ch.Equals('e'))
{
    Console.WriteLine("The character entered is a vowel and it is 'e'.");
}
else if (ch.Equals('i'))
{
    Console.WriteLine("The character entered is a vowel and it is 'i'.");
}
else if (ch.Equals('o'))
{
    Console.WriteLine("The character entered is a vowel and it is 'o'.");
}
else if (ch.Equals('u'))
{
    Console.WriteLine("The character entered is a vowel and it is 'u'.");
}
else
{
    Console.WriteLine("The character entered is not vowel. It is:" + ch );
}
```

# switch...case...default

switch 语句根据条件或多个条件评估一个表达式，并执行一个带标签的代码块。这些带标签的代码块被称为 switch 标签。每个 switch 标签后面跟着一个 break 语句，这有助于程序跳出循环并继续执行下一个语句。在先前的例子中，我们使用`if...else`语句检查元音，我们为每个元音使用`if...else`，并为任何其他字符提供一个默认值。这可以通过使用`switch...case...default`语句进一步简化。

我们所希望的是，一个条件表达式检查字符。如果它与任何匹配的表达式匹配，即元音，则打印元音；否则，打印它不是元音：

```cs
Console.Write("Enter a character: ");
char ch1 = (char)Console.Read();
switch (ch1)
{
    case 'a' :
    case 'e':
    case 'i' :
    case 'o' :
    case 'u':
     Console.WriteLine("The character entered is a vowel and it is: " + ch1);
        break;
    default:
        Console.WriteLine("The character entered is not vowel and it is: " + ch1);
        break;
}
```

# break

在 C#中，`break;`语句允许我们在包含它的循环或语句块中跳出。例如，在递归函数中，你可能需要在 n 次迭代后跳出。或者，在一个例子中，你想要在 10 次迭代的循环中打印前 5 个数字，你将想要使用 break 语句：

```cs
for (int i = 1; i <= 10; i++)
{
    if (i == 5)
    {
        break;
    }
    Console.WriteLine(i);
}

//output:
1
2
3
4
```

# goto

`Goto`语句允许程序将控制权转移到特定的部分或代码块。这也被称为带标签的语句。经典的例子是我们在上一节讨论的`Switch..case`语句。当一个表达式匹配一个 case 时，该代码块中的带标签的准则语句将被执行：

```cs
for (int i = 1; i <= 10; i++)
{
    if (i == 5)
    {
        goto  number5;
    }
    Console.WriteLine(i);
}
number5:
    Console.WriteLine("You are here because of Goto Label");
//Output
1
2
3
4
You are here because of Goto Label

```

# continue

`continue;`语句允许程序跳过该代码块中语句的执行，直到代码块结束，然后继续下一个迭代。例如，在一个`1..10`的`for`循环中，如果`continue`语句放在一个表达式内，即`i <= 5`，它会查看所有 10 个数字，但只有 6、7、8、9 和 10 会执行操作：

```cs
for (int i = 1; i <= 10; i++)
{
    if (i <= 5)
    {
        continue;
    }
    Console.WriteLine(i);
}
//output
6
7
8
9
10

```

# 迭代语句

迭代语句帮助执行循环特定次数或直到满足条件表达式。当循环开始时，代码块中的所有语句都按顺序执行。如果程序遇到`跳转语句`或`继续语句`，执行流程将改变。在`go-to`的情况下，控制移动到标记的代码块，而在`继续语句`的情况下，循环忽略`继续`之后的全部语句。

以下是在 C#中需要迭代或循环时使用的关键字：

+   `do`

+   `for`

+   `foreach...in`

+   `while`

# do...while

总是与`while`语句一起使用`do`语句。`do`语句执行代码块并评估`while`表达式。如果`while`语句评估为`true`，则代码块再次执行。只要`while`评估为`true`，这个过程就会继续。因为条件表达式是在代码块执行之后评估的，所以`do...while`总是至少执行代码块一次。

`break;`、`continue;`、`return`或`throw`可以在执行过程中任何时候退出这个循环：

```cs
int intvariable = 0;
 do
 {
     Console.WriteLine("Number is :" + intvariable);
     intvariable++;

 } while (intvariable < 5);

//Output
Number is :0
Number is :1
Number is :2
Number is :3
Number is :4

```

# for

与`do..while`不同，`for`首先评估条件表达式，如果为`true`，则执行代码块。一旦条件为真，代码块才会执行。与`do..while`类似，我们可以使用`return`、`throw`、`goto`或`continue`语句退出循环。

看一下以下`for`语句的结构：

```cs
for (initializer; condition; iterator) 
{
    body
}
```

初始化器、条件、迭代器都是可选的。主体可以是一行语句或整个代码块：

```cs
for (int i = 0; i <= 5; i++)
{
    Console.WriteLine("Number is :" + i);
}

//output
Number is :0
Number is :1
Number is :2
Number is :3
Number is :4
Number is :5
```

# 初始化部分

这个部分只执行一次。当程序控制遇到`for`循环时，初始化部分被触发。`#`允许在`for`循环的初始化部分使用一个或多个以下语句，语句之间用逗号分隔：

+   局部循环变量的声明。这个变量在循环外部不可用。

+   一个赋值语句。

+   方法调用。

+   前置/后置递增或递减。

+   新对象创建。

+   等待表达式。我们将在接下来的章节中更详细地探讨这个问题。

# 条件部分

如我们之前提到的，这是一个可选部分。如果没有提供，默认情况下，它被评估为`true`。如果提供了，则在每次迭代执行之前评估条件表达式。如果条件评估为`false`，则循环终止。

# 迭代部分

迭代部分定义了循环体的行为。正如在 *初始化器部分* 中详细说明的那样，它可以包含上述一个或多个语句。

# 语句的罕见用法示例

这里有一个参考示例：

```cs
int k;
int j = 10;
for (k = 0, Console.WriteLine("Start: j={j}"); k < j; k++, j--, Console.WriteLine("Step: k={k}, j={j}"))
{
    // Body of the loop.
}
for (; ; )
{
    // Body of the loop.
}
```

# foreach...in

`Foreach` 可用于 `IEnumerable` 类型或泛型集合的实例。这与 `for` 循环类似工作。`Foreach` 不仅限于这些类型；它还可以应用于任何实现无参数 `GetEnumerator` 方法并返回类、结构或接口类型的实例。`Foreach` 还可以应用于由 `GetEnumerator` 的 `Current` 属性和参数较少的 `MoveNext` 方法返回的类型，这些方法返回一个布尔值。

从 C# 7.3 开始，`Current` 属性返回对返回值的引用（`ref T`），其中 `T` 是集合元素类型。

在以下示例中，我们声明了一个字符串列表，并希望遍历列表，在屏幕上显示每个项目：

```cs
List<string> stringlist = new List<string>() { "One", "Two", "Three" };
foreach (string str in stringlist)
{
    Console.WriteLine("Element #"+ str);
}

//Output:
Element #One
Element #Two
Element #Three
```

`IEnumerator` 有一个名为 `Current` 的属性和一个名为 `MoveNext` 的方法。由于 `foreach` 循环旨在迭代实现这两个方法的集合，它跟踪集合中当前正在评估和处理的项目。这确保了控制不会传递到集合的末尾。此外，`foreach` 循环不允许用户更改初始化的循环变量，但允许它们修改变量引用的对象中的值。

# while

与 `for` 循环类似，在执行代码块之前会评估条件。这意味着代码块要么执行多次，要么根本不执行。就像任何其他循环一样，您可以使用 `break`、`continue`、`return` 或 `throw` 语句退出循环：

```cs
int n = 0;
while (n < 5)
{
    Console.WriteLine(n);
    n++;
}
//output
0
1
2
3
4
```

# 摘要

在本章中，我们探讨了算术运算符、关系运算符、移位运算符以及相等、条件和逻辑运算符，这些运算符可以与一个或两个操作数一起使用，并使用逻辑和条件运算符作为布尔表达式进行评估。

我们探讨了条件语句和选择性语句，这些语句帮助我们做出决策。这些语句的例子包括 if 条件、then 语句和 else 语句。`switch...case...default` 帮助匹配多个表达式并执行多个 switch 标签。

我们还探讨了迭代语句，它允许用户遍历一个集合。当与 `goto`、`continue` 等跳转语句一起使用时，它们可以退出循环。

在下一章中，我们将详细探讨委托和事件。委托和事件在 C# 编程中扮演着重要角色。能够为事件调用回委托使我们能够解耦我们的程序。我们还将了解 Lambda 表达式，它可以用来创建委托。这些也被称为 **匿名** 方法。

# 问题

1.  你有一个评估很多条件的情况。在某个特定场景中，你想要评估两个操作数，如果为真，则执行代码块。以下哪个语句你会使用？

    1.  `&&`

    1.  `||`

    1.  `&`

    1.  `^`

1.  你在代码中使用`for`循环，并且想要在满足条件时执行特定的代码块。以下哪个语句你会使用？

    1.  `break;`

    1.  `continue;`

    1.  `throw;`

    1.  `goto;`

1.  在你的程序中，有一个你想要至少执行一次并且直到条件评估为真才停止执行的代码块。以下哪个语句你会使用？

    1.  `While;`

    1.  `Do...while;`

    1.  `For;`

    1.  `foreach;`

# 答案

1.  **c**

1.  **d**

1.  **b**

# 进一步阅读

更多关于语句、表达式和操作符的信息可以在[`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/)找到。

Packt Publishing 网站上有一个有帮助的视频。它被称为*Programming in C# .NET* ([`search.packtpub.com/?query=70-483&refinementList%5Breleased%5D%5B0%5D=Available`](https://search.packtpub.com/?query=70-483&refinementList%5Breleased%5D%5B0%5D=Available))。
