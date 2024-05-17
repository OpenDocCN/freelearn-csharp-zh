# 第三章：控制流程、类型转换和异常处理

本章是关于编写对变量执行简单操作、做出决策、执行模式匹配、重复语句或代码块、将变量或表达式的值从一种类型转换为另一种类型、处理异常以及检查数字变量溢出的代码。

本章涵盖以下主题：

+   操作变量

+   理解选择语句

+   理解迭代语句

+   类型之间的转换和强制转换

+   处理异常

+   检查溢出

# 操作变量

**运算符**对**操作数**（如变量和字面值）执行简单的操作，如加法和乘法。它们通常返回一个新值，即操作的结果，该结果可以分配给一个变量。

大多数运算符是二元的，意味着它们作用于两个操作数，如下面的伪代码所示：

```cs
var resultOfOperation = firstOperand operator secondOperand; 
```

二元运算符的例子包括加法和乘法，如下面的代码所示：

```cs
int x = 5;
int y = 3;
int resultOfAdding = x + y;
int resultOfMultiplying = x * y; 
```

一些运算符是单目的，意味着它们作用于单个操作数，并且可以应用于操作数之前或之后，如下面的伪代码所示：

```cs
var resultOfOperation = onlyOperand operator; 
var resultOfOperation2 = operator onlyOperand; 
```

单目运算符的例子包括增量器和获取类型或其大小（以字节为单位），如下面的代码所示：

```cs
int x = 5;
int postfixIncrement = x++;
int prefixIncrement = ++x;
Type theTypeOfAnInteger = typeof(int); 
int howManyBytesInAnInteger = sizeof(int); 
```

三元运算符作用于三个操作数，如下面的伪代码所示：

```cs
var resultOfOperation = firstOperand firstOperator 
  secondOperand secondOperator thirdOperand; 
```

## 探索单目运算符

两个常用的单目运算符用于增加（`++`）和减少（`--`）一个数字。让我们写一些示例代码来展示它们是如何工作的：

1.  如果你已经完成了前面的章节，那么你将已经有一个`Code`文件夹。如果没有，那么你需要创建它。

1.  使用你喜欢的编程工具创建一个新的控制台应用程序，如下表所定义：

    1.  项目模板：**控制台应用程序** / `console`

    1.  工作区/解决方案文件和文件夹：`Chapter03`

    1.  项目文件和文件夹：`Operators`

1.  在`Program.cs`顶部，静态导入`System.Console`。

1.  在`Program.cs`中，声明两个整型变量`a`和`b`，将`a`设置为`3`，在将结果赋给`b`的同时增加`a`，然后输出它们的值，如下面的代码所示：

    ```cs
    int a = 3; 
    int b = a++;
    WriteLine($"a is {a}, b is {b}"); 
    ```

1.  在运行控制台应用程序之前，问自己一个问题：你认为输出时`b`的值会是多少？一旦你考虑了这一点，运行代码，并将你的预测与实际结果进行比较，如下面的输出所示：

    ```cs
    a is 4, b is 3 
    ```

    变量`b`的值为`3`，因为`++`运算符在赋值*之后*执行；这被称为**后缀运算符**。如果你需要在赋值*之前*增加，那么使用**前缀运算符**。

1.  复制并粘贴这些语句，然后修改它们以重命名变量并使用前缀运算符，如下面的代码所示：

    ```cs
    int c = 3;
    int d = ++c; // increment c before assigning it
    WriteLine($"c is {c}, d is {d}"); 
    ```

1.  重新运行代码并注意结果，如下面的输出所示：

    ```cs
    a is 4, b is 3
    c is 4, d is 4 
    ```

    **最佳实践**：由于增量和减量运算符的前缀和后缀结合赋值时的混淆，Swift 编程语言设计者在版本 3 中决定不再支持此运算符。我建议在 C#中使用时，切勿将`++`和`--`运算符与赋值运算符`=`结合使用。将这些操作作为单独的语句执行。

## 探索二元算术运算符

增量和减量是一元算术运算符。其他算术运算符通常是二元的，允许你对两个数字执行算术运算，如下所示：

1.  添加语句以声明并赋值给两个名为`e`和`f`的整型变量，然后对这两个数字应用五个常见的二元算术运算符，如下面的代码所示：

    ```cs
    int e = 11; 
    int f = 3;
    WriteLine($"e is {e}, f is {f}"); 
    WriteLine($"e + f = {e + f}"); 
    WriteLine($"e - f = {e - f}"); 
    WriteLine($"e * f = {e * f}"); 
    WriteLine($"e / f = {e / f}"); 
    WriteLine($"e % f = {e % f}"); 
    ```

1.  运行代码并注意结果，如下面的输出所示：

    ```cs
    e is 11, f is 3 
    e + f = 14
    e - f = 8 
    e * f = 33 
    e / f = 3 
    e % f = 2 
    ```

    要理解应用于整数的除法`/`和模`%`运算符，你需要回想小学时期。想象你有十一颗糖果和三个朋友。

    你如何将糖果分给你的朋友们？你可以给每位朋友三颗糖果，剩下两颗。那两颗糖果就是**模数**，也称为除法后的**余数**。如果你有十二颗糖果，那么每位朋友得到四颗，没有剩余，所以余数为 0。

1.  添加语句以声明并赋值给一个名为`g`的`double`变量，以展示整数和实数除法的区别，如下面的代码所示：

    ```cs
    double g = 11.0;
    WriteLine($"g is {g:N1}, f is {f}"); 
    WriteLine($"g / f = {g / f}"); 
    ```

1.  运行代码并注意结果，如下面的输出所示：

    ```cs
    g is 11.0, f is 3
    g / f = 3.6666666666666665 
    ```

如果第一个操作数是浮点数，例如值为`11.0`的`g`，那么除法运算符返回一个浮点值，例如`3.6666666666665`，而不是整数。

## 赋值运算符

你已经一直在使用最常见的赋值运算符，`=`。

为了使你的代码更简洁，你可以将赋值运算符与其他运算符如算术运算符结合使用，如下面的代码所示：

```cs
int p = 6;
p += 3; // equivalent to p = p + 3;
p -= 3; // equivalent to p = p - 3;
p *= 3; // equivalent to p = p * 3;
p /= 3; // equivalent to p = p / 3; 
```

## 探索逻辑运算符

逻辑运算符操作于布尔值，因此它们返回`true`或`false`。让我们探索操作于两个布尔值的二元逻辑运算符：

1.  使用你偏好的编程工具，在`Chapter03`工作区/解决方案中添加一个名为`BooleanOperators`的新控制台应用。

    1.  在 Visual Studio Code 中，选择`BooleanOperators`作为活动的 OmniSharp 项目。当你看到弹出警告消息说缺少必需资产时，点击**是**以添加它们。

    1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择的项目。

        **最佳实践**：记得静态导入`System.Console`类型以简化语句。

1.  在 `Program.cs` 中，添加语句以声明两个布尔变量，其值分别为 `true` 和 `false`，然后输出应用 AND、OR 和 XOR（异或）逻辑运算符的结果的真值表，如下面的代码所示：

    ```cs
    bool a = true;
    bool b = false;
    WriteLine($"AND  | a     | b    ");
    WriteLine($"a    | {a & a,-5} | {a & b,-5} ");
    WriteLine($"b    | {b & a,-5} | {b & b,-5} ");
    WriteLine();
    WriteLine($"OR   | a     | b    ");
    WriteLine($"a    | {a | a,-5} | {a | b,-5} ");
    WriteLine($"b    | {b | a,-5} | {b | b,-5} ");
    WriteLine();
    WriteLine($"XOR  | a     | b    ");
    WriteLine($"a    | {a ^ a,-5} | {a ^ b,-5} ");
    WriteLine($"b    | {b ^ a,-5} | {b ^ b,-5} "); 
    ```

1.  运行代码并注意结果，如下面的输出所示：

    ```cs
    AND  | a     | b    
    a    | True  | False 
    b    | False | False 
    OR   | a     | b    
    a    | True  | True  
    b    | True  | False 
    XOR  | a     | b    
    a    | False | True  
    b    | True  | False 
    ```

对于 AND `&` 逻辑运算符，两个操作数都必须为 `true` 才能使结果为 `true`。对于 OR `|` 逻辑运算符，任一操作数为 `true` 即可使结果为 `true`。对于 XOR `^` 逻辑运算符，任一操作数可以为 `true`（但不能同时为 `true`！）以使结果为 `true`。

## 探索条件逻辑运算符

条件逻辑运算符类似于逻辑运算符，但你需要使用两个符号而不是一个，例如，`&&` 代替 `&`，或者 `||` 代替 `|`。

在 *第四章*，*编写、调试和测试函数* 中，你将更详细地了解函数，但我现在需要介绍函数以解释条件逻辑运算符，也称为短路布尔运算符。

函数执行语句然后返回一个值。该值可以是用于布尔运算的布尔值，例如 `true`。让我们利用条件逻辑运算符：

1.  在 `Program.cs` 底部，编写语句以声明一个向控制台写入消息并返回 `true` 的函数，如下面的代码所示：

    ```cs
    static bool DoStuff()
    {
      WriteLine("I am doing some stuff.");
      return true;
    } 
    ```

    **最佳实践**：如果你使用的是 .NET 交互式笔记本，请在单独的代码单元格中编写 `DoStuff` 函数，然后执行它，以便其上下文可供其他代码单元格使用。

1.  在前面的 `WriteLine` 语句之后，对 `a` 和 `b` 变量以及调用函数的结果执行 AND `&` 操作，如下面的代码所示：

    ```cs
    WriteLine();
    WriteLine($"a & DoStuff() = {a & DoStuff()}"); 
    WriteLine($"b & DoStuff() = {b & DoStuff()}"); 
    ```

1.  运行代码，查看结果，并注意该函数被调用了两次，一次是为 a，一次是为 b，如以下输出所示：

    ```cs
    I am doing some stuff. 
    a & DoStuff() = True
    I am doing some stuff. 
    b & DoStuff() = False 
    ```

1.  将 `&` 运算符更改为 `&&` 运算符，如下面的代码所示：

    ```cs
    WriteLine($"a && DoStuff() = {a && DoStuff()}"); 
    WriteLine($"b && DoStuff() = {b && DoStuff()}"); 
    ```

1.  运行代码，查看结果，并注意当与 `a` 变量结合时，函数确实运行了。当与 `b` 变量结合时，它不会运行，因为 `b` 变量是 `false`，所以结果无论如何都将是 `false`，因此不需要执行该函数，如下面的输出所示：

    ```cs
    I am doing some stuff. 
    a && DoStuff() = True
    b && DoStuff() = False // DoStuff function was not executed! 
    ```

    **最佳实践**：现在你可以明白为什么条件逻辑运算符被描述为短路运算符。它们可以使你的应用程序更高效，但它们也可能在假设函数总是被调用的情况下引入微妙的错误。当与产生副作用的函数结合使用时，最安全的是避免使用它们。

## 探索位运算和二进制移位运算符

位运算符影响数字中的位。二进制移位运算符可以比传统运算符更快地执行一些常见的算术计算，例如，任何乘以 2 的倍数。

让我们探索位运算和二进制移位运算符：

1.  使用您喜欢的编码工具，在`Chapter03`工作区/解决方案中添加一个新的**控制台应用程序**，命名为`BitwiseAndShiftOperators`。

1.  在 Visual Studio Code 中，选择`BitwiseAndShiftOperators`作为活动 OmniSharp 项目。当看到弹出警告消息提示缺少必需资产时，点击**是**以添加它们。

1.  在`Program.cs`中，键入语句以声明两个整数变量，值分别为 10 和 6，然后输出应用 AND、OR 和 XOR 位运算符的结果，如下面的代码所示：

    ```cs
    int a = 10; // 00001010
    int b = 6;  // 00000110
    WriteLine($"a = {a}");
    WriteLine($"b = {b}");
    WriteLine($"a & b = {a & b}"); // 2-bit column only 
    WriteLine($"a | b = {a | b}"); // 8, 4, and 2-bit columns 
    WriteLine($"a ^ b = {a ^ b}"); // 8 and 4-bit columns 
    ```

1.  运行代码并注意结果，如下面的输出所示：

    ```cs
    a = 10
    b = 6
    a & b = 2 
    a | b = 14
    a ^ b = 12 
    ```

1.  在`Program.cs`中，添加语句以输出应用左移运算符将变量`a`的位移动三列、将`a`乘以 8 以及右移变量`b`的位一列的结果，如下面的代码所示：

    ```cs
    // 01010000 left-shift a by three bit columns
    WriteLine($"a << 3 = {a << 3}");
    // multiply a by 8
    WriteLine($"a * 8 = {a * 8}");
    // 00000011 right-shift b by one bit column
    WriteLine($"b >> 1 = {b >> 1}"); 
    ```

1.  运行代码并注意结果，如下面的输出所示：

    ```cs
    a << 3 = 80
    a * 8 = 80
    b >> 1 = 3 
    ```

结果`80`是因为其中的位向左移动了三列，因此 1 位移动到了 64 位和 16 位列，64 + 16 = 80。这相当于乘以 8，但 CPU 可以更快地执行位移。结果 3 是因为`b`中的 1 位向右移动了一列，进入了 2 位和 1 位列。

**良好实践**：记住，当操作整数值时，`&`和`|`符号是位运算符，而当操作布尔值如`true`和`false`时，`&`和`|`符号是逻辑运算符。

我们可以通过将整数值转换为零和一的二进制字符串来演示这些操作：

1.  在`Program.cs`底部，添加一个函数，将整数值转换为最多包含八个零和一的二进制（Base2）`字符串`，如下面的代码所示：

    ```cs
    static string ToBinaryString(int value)
    {
      return Convert.ToString(value, toBase: 2).PadLeft(8, '0');
    } 
    ```

1.  在函数上方，添加语句以输出`a`、`b`以及各种位运算符的结果，如下面的代码所示：

    ```cs
    WriteLine();
    WriteLine("Outputting integers as binary:");
    WriteLine($"a =     {ToBinaryString(a)}");
    WriteLine($"b =     {ToBinaryString(b)}");
    WriteLine($"a & b = {ToBinaryString(a & b)}");
    WriteLine($"a | b = {ToBinaryString(a | b)}");
    WriteLine($"a ^ b = {ToBinaryString(a ^ b)}"); 
    ```

1.  运行代码并注意结果，如下面的输出所示：

    ```cs
    Outputting integers as binary:
    a =     00001010
    b =     00000110
    a & b = 00000010
    a | b = 00001110
    a ^ b = 00001100 
    ```

## 杂项运算符

`nameof`和`sizeof`是在处理类型时方便的运算符：

+   `nameof`返回变量、类型或成员的简短名称（不包含命名空间）作为`字符串`值，这在输出异常消息时非常有用。

+   `sizeof`返回简单类型的大小（以字节为单位），这对于确定数据存储的效率非常有用。

还有许多其他运算符；例如，变量与其成员之间的点称为**成员访问运算符**，函数或方法名称末尾的圆括号称为**调用运算符**，如下面的代码所示：

```cs
int age = 47;
// How many operators in the following statement?
char firstDigit = age.ToString()[0];
// There are four operators:
// = is the assignment operator
// . is the member access operator
// () is the invocation operator
// [] is the indexer access operator 
```

# 理解选择语句

每个应用程序都需要能够从选项中选择并沿不同的代码路径分支。C#中的两种选择语句是`if`和`switch`。你可以使用`if`来编写所有代码，但`switch`可以在某些常见场景中简化你的代码，例如当存在一个可以有多个值的单一变量，每个值都需要不同的处理时。

## if 语句的分支

`if`语句通过评估一个布尔表达式来决定执行哪个分支。如果表达式为`true`，则执行该代码块。`else`块是可选的，如果`if`表达式为`false`，则执行`else`块。`if`语句可以嵌套。

`if`语句可以与其他`if`语句结合，形成`else if`分支，如下列代码所示：

```cs
if (expression1)
{
  // runs if expression1 is true
}
else if (expression2)
{
  // runs if expression1 is false and expression2 if true
}
else if (expression3)
{
  // runs if expression1 and expression2 are false
  // and expression3 is true
}
else
{
  // runs if all expressions are false
} 
```

每个`if`语句的布尔表达式都是独立的，与`switch`语句不同，它不需要引用单一值。

让我们编写一些代码来探索像`if`这样的选择语句：

1.  使用你喜欢的编程工具，在`Chapter03`工作区/解决方案中添加一个名为`SelectionStatements`的新**控制台应用程序**。

1.  在 Visual Studio Code 中，选择`SelectionStatements`作为活动 OmniSharp 项目。

1.  在`Program.cs`中，输入语句以检查密码是否至少有八个字符，如下列代码所示：

    ```cs
    string password = "ninja";
    if (password.Length < 8)
    {
      WriteLine("Your password is too short. Use at least 8 characters.");
    }
    else
    {
      WriteLine("Your password is strong.");
    } 
    ```

1.  运行代码并注意结果，如下列输出所示：

    ```cs
     Your password is too short. Use at least 8 characters. 
    ```

### 为什么你应该始终在 if 语句中使用大括号

由于每个代码块内只有一条语句，前面的代码可以不使用花括号编写，如下列代码所示：

```cs
if (password.Length < 8)
  WriteLine("Your password is too short. Use at least 8 characters."); 
else
  WriteLine("Your password is strong."); 
```

应避免这种风格的`if`语句，因为它可能引入严重的错误，例如苹果 iPhone iOS 操作系统中臭名昭著的#gotofail 错误。

在苹果发布 iOS 6 后的 18 个月内，即 2012 年 9 月，其**安全套接字层**（**SSL**）加密代码中存在一个错误，这意味着任何使用 Safari（设备上的网页浏览器）尝试连接到安全网站（如银行）的用户都没有得到适当的保护，因为一个重要的检查被意外跳过了。

仅仅因为你可以在没有花括号的情况下编写代码，并不意味着你应该这样做。没有它们的代码并不会“更高效”；相反，它更难以维护，且可能更危险。

## if 语句的模式匹配

引入 C# 7.0 及更高版本的一个特性是模式匹配。`if`语句可以通过结合使用`is`关键字和声明一个局部变量来使代码更安全：

1.  添加语句，以便如果存储在名为`o`的变量中的值是`int`，则该值被赋给局部变量`i`，然后可以在`if`语句中使用。这比使用名为`o`的变量更安全，因为我们确信`i`是一个`int`变量，而不是其他东西，如下列代码所示：

    ```cs
    // add and remove the "" to change the behavior
    object o = "3"; 
    int j = 4;
    if (o is int i)
    {
      WriteLine($"{i} x {j} = {i * j}");
    }
    else
    {
      WriteLine("o is not an int so it cannot multiply!");
    } 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    o is not an int so it cannot multiply! 
    ```

1.  删除围绕`"3"`值的双引号字符，以便存储在名为`o`的变量中的值是`int`类型而不是`string`类型。

1.  重新运行代码以查看结果，如下面的输出所示：

    ```cs
    3 x 4 = 12 
    ```

## 使用 switch 语句进行分支

`switch`语句与`if`语句不同，因为`switch`将单个表达式与多个可能的`case`语句列表进行比较。每个`case`语句都与单个表达式相关。每个`case`部分必须以：

+   使用`break`关键字（如下面的代码中的 case 1）

+   或者使用`goto` `case`关键字（如下面的代码中的 case 2）

+   或者它们可以没有任何语句（如下面的代码中的 case 3）

+   或者`goto`关键字，它引用一个命名标签（如下面的代码中的 case 5）

+   或者使用`return`关键字离开当前函数（未在代码中显示）

让我们编写一些代码来探索`switch`语句：

1.  为`switch`语句编写语句。您应该注意到倒数第二条语句是一个可以跳转到的标签，而第一条语句生成一个介于 1 和 6 之间的随机数（代码中的数字 7 是上限）。`switch`语句分支基于这个随机数的值，如下面的代码所示：

    ```cs
    int number = (new Random()).Next(1, 7); 
    WriteLine($"My random number is {number}");
    switch (number)
    {
      case 1: 
        WriteLine("One");
        break; // jumps to end of switch statement
      case 2:
        WriteLine("Two");
        goto case 1;
      case 3: // multiple case section
      case 4:
        WriteLine("Three or four");
        goto case 1;
      case 5:
        goto A_label;
      default:
        WriteLine("Default");
        break;
    } // end of switch statement
    WriteLine("After end of switch");
    A_label:
    WriteLine($"After A_label"); 
    ```

    **最佳实践**：您可以使用`goto`关键字跳转到另一个 case 或标签。大多数程序员对`goto`关键字持反对态度，但在某些情况下，它可以是解决代码逻辑的好方法。但是，您应该谨慎使用它。

1.  多次运行代码以查看随机数在各种情况下的结果，如下面的示例输出所示：

    ```cs
    // first random run
    My random number is 4 
    Three or four
    One
    After end of switch
    After A_label
    // second random run
    My random number is 2 
    Two
    One
    After end of switch
    After A_label
    // third random run
    My random number is 6
    Default
    After end of switch
    After A_label
    // fourth random run
    My random number is 1 
    One
    After end of switch
    After A_label
    // fifth random run
    My random number is 5
    After A_label 
    ```

## 使用 switch 语句进行模式匹配

与`if`语句一样，`switch`语句在 C# 7.0 及更高版本中支持模式匹配。`case`值不再需要是文字值；它们可以是模式。

让我们看一个使用文件夹路径的`switch`语句进行模式匹配的示例。如果您使用的是 macOS，则交换设置路径变量的注释语句，并将我的用户名替换为您的用户文件夹名称：

1.  添加语句以声明一个`string`路径到文件，将其打开为只读或可写流，然后根据流的类型和功能显示一条消息，如下面的代码所示：

    ```cs
    // string path = "/Users/markjprice/Code/Chapter03";
    string path = @"C:\Code\Chapter03";
    Write("Press R for read-only or W for writeable: "); 
    ConsoleKeyInfo key = ReadKey();
    WriteLine();
    Stream? s;
    if (key.Key == ConsoleKey.R)
    {
      s =  File.Open(
        Path.Combine(path, "file.txt"), 
        FileMode.OpenOrCreate, 
        FileAccess.Read);
    }
    else
    {
      s =  File.Open( 
        Path.Combine(path, "file.txt"), 
        FileMode.OpenOrCreate, 
        FileAccess.Write);
    }
    string message; 
    switch (s)
    {
      case FileStream writeableFile when s.CanWrite:
        message = "The stream is a file that I can write to.";
        break;
      case FileStream readOnlyFile:
        message = "The stream is a read-only file.";
        break;
      case MemoryStream ms:
        message = "The stream is a memory address.";
        break;
      default: // always evaluated last despite its current position
        message = "The stream is some other type.";
        break;
      case null:
        message = "The stream is null.";
        break;
    }
    WriteLine(message); 
    ```

1.  运行代码并注意名为`s`的变量被声明为`Stream`类型，因此它可以是流的任何子类型，例如内存流或文件流。在此代码中，流是通过`File.Open`方法创建的，该方法返回一个文件流，并根据您的按键，它将是可写的或只读的，因此结果将是一条描述情况的 message，如下面的输出所示：

    ```cs
    The stream is a file that I can write to. 
    ```

在.NET 中，`Stream`有多个子类型，包括`FileStream`和`MemoryStream`。在 C# 7.0 及更高版本中，你的代码可以根据流子类型更简洁地分支，并声明和赋值一个局部变量以安全使用。你将在*第九章*，*文件、流和序列化操作*中了解更多关于`System.IO`命名空间和`Stream`类型的信息。

此外，`case`语句可以包含`when`关键字以执行更具体的模式匹配。在前述代码的第一个 case 语句中，`s`仅在流为`FileStream`且其`CanWrite`属性为`true`时匹配。

## 使用 switch 表达式简化 switch 语句

C# 8.0 及以上版本中，你可以使用**switch 表达式**简化`switch`语句。

大多数`switch`语句虽然简单，但需要大量输入。`switch`表达式旨在简化所需代码，同时在所有情况都返回值以设置单个变量的情况下，仍表达相同意图。`switch`表达式使用 lambda `=>`表示返回值。

让我们将之前使用`switch`语句的代码实现为`switch`表达式，以便比较两种风格：

1.  使用`switch`表达式，根据流的类型和功能输入语句设置消息，如下列代码所示：

    ```cs
    message = s switch
    {
      FileStream writeableFile when s.CanWrite
        => "The stream is a file that I can write to.", 
      FileStream readOnlyFile
        => "The stream is a read-only file.", 
      MemoryStream ms
        => "The stream is a memory address.", 
      null
        => "The stream is null.",
      _
        => "The stream is some other type."
    };
    WriteLine(message); 
    ```

    主要区别在于移除了`case`和`break`关键字。下划线字符`_`用于表示默认返回值。

1.  运行代码，并注意结果与之前相同。

# 理解迭代语句

迭代语句重复一个语句块，要么在条件为真时，要么对集合中的每个项。选择使用哪种语句基于解决问题逻辑的易理解性和个人偏好。

## while 语句循环

`while`语句评估布尔表达式，并在其为真时继续循环。让我们探索迭代语句：

1.  使用你偏好的编程工具，在`Chapter03`工作区/解决方案中添加一个名为`IterationStatements`的**控制台应用程序**。

1.  在 Visual Studio Code 中，选择`IterationStatements`作为活动 OmniSharp 项目。

1.  在`Program.cs`中，输入语句定义一个`while`语句，当整型变量值小于 10 时循环，如下列代码所示：

    ```cs
    int x = 0;
    while (x < 10)
    {
      WriteLine(x);
      x++;
    } 
    ```

1.  运行代码并查看结果，应显示数字 0 至 9，如下列输出所示：

    ```cs
    0
    1
    2
    3
    4
    5
    6
    7
    8
    9 
    ```

## do 语句循环

与`while`类似，`do`语句在代码块底部而非顶部检查布尔表达式，这意味着代码块至少会执行一次，如下所示：

1.  输入语句定义一个`do`循环，如下列代码所示：

    ```cs
    string? password;
    do
    {
      Write("Enter your password: "); 
      password = ReadLine();
    }
    while (password != "Pa$$w0rd");
    WriteLine("Correct!"); 
    ```

1.  运行代码，并注意你需要反复输入密码，直到输入正确，如下列输出所示：

    ```cs
    Enter your password: password 
    Enter your password: 12345678 
    Enter your password: ninja
    Enter your password: correct horse battery staple 
    Enter your password: Pa$$w0rd
    Correct! 
    ```

1.  作为可选挑战，添加语句，以便用户只能在出现错误消息之前尝试十次。

## 使用 for 语句循环

`for`语句类似于`while`，只不过它更为简洁。它结合了：

+   一个**初始化表达式**，它在循环开始时执行一次。

+   一个**条件表达式**，它在每次迭代开始时执行，以检查循环是否应继续。

+   一个**迭代器表达式**，它在每次循环的底部执行。

`for`语句通常与整数计数器一起使用。让我们探讨一些代码：

1.  键入一个`for`语句以输出数字 1 到 10，如下所示：

    ```cs
    for (int y = 1; y <= 10; y++)
    {
      WriteLine(y);
    } 
    ```

1.  运行代码以查看结果，结果应为数字 1 到 10。

## 使用 foreach 语句循环

`foreach`语句与之前的三种迭代语句略有不同。

它用于对序列（例如数组或集合）中的每个项执行一组语句。通常，每个项都是只读的，如果在迭代期间修改序列结构（例如，通过添加或删除项），则会抛出异常。

尝试以下示例：

1.  键入语句以创建一个字符串变量数组，然后输出每个变量的长度，如下所示：

    ```cs
    string[] names = { "Adam", "Barry", "Charlie" };
    foreach (string name in names)
    {
      WriteLine($"{name} has {name.Length} characters.");
    } 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Adam has 4 characters. 
    Barry has 5 characters. 
    Charlie has 7 characters. 
    ```

### 理解 foreach 内部工作原理

任何表示多个项（如数组或集合）的类型的创建者应确保程序员可以使用`foreach`语句枚举该类型的项。

从技术上讲，`foreach`语句将适用于遵循以下规则的任何类型：

1.  该类型必须具有一个名为`GetEnumerator`的方法，该方法返回一个对象。

1.  返回的对象必须具有名为`Current`的属性和名为`MoveNext`的方法。

1.  `MoveNext`方法必须更改`Current`的值，并在还有更多项要枚举时返回`true`，或在无更多项时返回`false`。

存在名为`IEnumerable`和`IEnumerable<T>`的接口，它们正式定义了这些规则，但编译器实际上并不要求类型实现这些接口。

编译器将前面的示例中的`foreach`语句转换为类似以下伪代码的内容：

```cs
IEnumerator e = names.GetEnumerator();
while (e.MoveNext())
{
  string name = (string)e.Current; // Current is read-only!
  WriteLine($"{name} has {name.Length} characters.");
} 
```

由于使用了迭代器，`foreach`语句中声明的变量不能用于修改当前项的值。

# 类型之间的转换和转换

你经常需要在不同类型的变量之间转换值。例如，数据输入通常作为控制台上的文本输入，因此最初存储在`string`类型的变量中，但随后需要将其转换为日期/时间，或数字，或其他数据类型，具体取决于应如何存储和处理。

有时在进行计算之前，你需要在整数和浮点数等数字类型之间进行转换。

转换也称为**类型转换**，它有两种形式：**隐式**和**显式**。隐式转换会自动发生，是安全的，意味着不会丢失任何信息。

显式转换必须手动执行，因为它可能会丢失信息，例如数字的精度。通过显式转换，你告诉 C#编译器你理解并接受这种风险。

## 隐式和显式转换数字

将`int`变量隐式转换为`double`变量是安全的，因为不会丢失任何信息，如下所示：

1.  使用你喜欢的编码工具在`Chapter03`工作区/解决方案中添加一个新的**控制台应用程序**，命名为`CastingConverting`。

1.  在 Visual Studio Code 中，选择`CastingConverting`作为活动 OmniSharp 项目。

1.  在`Program.cs`中，输入语句以声明和赋值一个`int`变量和一个`double`变量，然后当将整数值赋给`double`变量时隐式转换整数值，如下所示：

    ```cs
    int a = 10;
    double b = a; // an int can be safely cast into a double
    WriteLine(b); 
    ```

1.  输入语句以声明和赋值一个`double`变量和一个`int`变量，然后当将`double`值赋给`int`变量时隐式转换`double`值，如下所示：

    ```cs
    double c = 9.8;
    int d = c; // compiler gives an error for this line
    WriteLine(d); 
    ```

1.  运行代码并注意错误消息，如下所示：

    ```cs
    Error: (6,9): error CS0266: Cannot implicitly convert type 'double' to 'int'. An explicit conversion exists (are you missing a cast?) 
    ```

    此错误消息也会出现在 Visual Studio 错误列表或 Visual Studio Code 问题窗口中。

    你不能隐式地将`double`变量转换为`int`变量，因为这可能不安全且可能丢失数据，例如小数点后的值。你必须使用一对圆括号将要转换的`double`类型括起来，显式地将`double`变量转换为`int`变量。这对圆括号是**类型转换运算符**。即便如此，你也必须注意，小数点后的部分将被截断而不会警告，因为你选择执行显式转换，因此理解其后果。

1.  修改`d`变量的赋值语句，如下所示：

    ```cs
    int d = (int)c;
    WriteLine(d); // d is 9 losing the .8 part 
    ```

1.  运行代码以查看结果，如下所示：

    ```cs
    10
    9 
    ```

    当在较大整数和较小整数之间转换值时，我们必须执行类似的操作。再次提醒，你可能会丢失信息，因为任何过大的值都会复制其位，然后以你可能意想不到的方式进行解释！

1.  输入语句以声明和赋值一个长 64 位变量到一个 int 32 位变量，两者都使用一个小值和一个过大的值，如下所示：

    ```cs
    long e = 10; 
    int f = (int)e;
    WriteLine($"e is {e:N0} and f is {f:N0}"); 
    e = long.MaxValue;
    f = (int)e;
    WriteLine($"e is {e:N0} and f is {f:N0}"); 
    ```

1.  运行代码以查看结果，如下所示：

    ```cs
    e is 10 and f is 10
    e is 9,223,372,036,854,775,807 and f is -1 
    ```

1.  将`e`的值修改为 50 亿，如下所示：

    ```cs
    e = 5_000_000_000; 
    ```

1.  运行代码以查看结果，如下所示：

    ```cs
    e is 5,000,000,000 and f is 705,032,704 
    ```

## 使用 System.Convert 类型进行转换

使用类型转换操作符的替代方法是使用`System.Convert`类型。`System.Convert`类型可以转换为和从所有 C#数字类型，以及布尔值、字符串和日期时间值。

让我们写一些代码来实际看看：

1.  在`Program.cs`顶部，静态导入`System.Convert`类，如下所示：

    ```cs
    using static System.Convert; 
    ```

1.  在`Program.cs`底部，输入语句声明并赋值给一个`double`变量，将其转换为整数，然后将两个值写入控制台，如下所示：

    ```cs
    double g = 9.8;
    int h = ToInt32(g); // a method of System.Convert
    WriteLine($"g is {g} and h is {h}"); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    g is 9.8 and h is 10 
    ```

类型转换和转换之间的一个区别是，转换将`double`值`9.8`向上取整为`10`，而不是截去小数点后的部分。

## 取整数字

你现在已经看到，类型转换操作符会截去实数的小数部分，而`System.Convert`方法则会上下取整。但是，取整的规则是什么呢？

### 理解默认的取整规则

在英国 5 至 11 岁儿童的初等学校中，学生被教导如果小数部分为.5 或更高，则向上取整；如果小数部分小于.5，则向下取整。

让我们探究一下 C#是否遵循相同的初等学校规则：

1.  输入语句声明并赋值给一个`double`数组，将每个值转换为整数，然后将结果写入控制台，如下所示：

    ```cs
    double[] doubles = new[]
      { 9.49, 9.5, 9.51, 10.49, 10.5, 10.51 };
    foreach (double n in doubles)
    {
      WriteLine($"ToInt32({n}) is {ToInt32(n)}");
    } 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    ToInt32(9.49) is 9
    ToInt32(9.5) is 10
    ToInt32(9.51) is 10
    ToInt32(10.49) is 10
    ToInt32(10.5) is 10
    ToInt32(10.51) is 11 
    ```

我们已经表明，C#中的取整规则与初等学校的规则略有不同：

+   如果小数部分小于中点.5，它总是向下取整。

+   如果小数部分大于中点.5，它总是向上取整。

+   如果小数部分是中点.5，且非小数部分为*奇数*，则向上取整；但如果非小数部分为*偶数*，则向下取整。

这一规则被称为**银行家取整**，因其通过交替上下取整来减少偏差，所以被优先采用。遗憾的是，其他语言如 JavaScript 使用的是初等学校的规则。

## 掌握取整规则

你可以通过使用`Math`类的`Round`方法来控制取整规则：

1.  输入语句，使用“远离零”取整规则（也称为“向上”取整）对每个`double`值进行取整，然后将结果写入控制台，如下所示：

    ```cs
    foreach (double n in doubles)
    {
      WriteLine(format:
        "Math.Round({0}, 0, MidpointRounding.AwayFromZero) is {1}",
        arg0: n,
        arg1: Math.Round(value: n, digits: 0,
                mode: MidpointRounding.AwayFromZero));
    } 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Math.Round(9.49, 0, MidpointRounding.AwayFromZero) is 9
    Math.Round(9.5, 0, MidpointRounding.AwayFromZero) is 10
    Math.Round(9.51, 0, MidpointRounding.AwayFromZero) is 10
    Math.Round(10.49, 0, MidpointRounding.AwayFromZero) is 10
    Math.Round(10.5, 0, MidpointRounding.AwayFromZero) is 11
    Math.Round(10.51, 0, MidpointRounding.AwayFromZero) is 11 
    ```

    **最佳实践**：对于你使用的每种编程语言，检查其取整规则。它们可能不会按照你预期的方式工作！

## 将任何类型转换为字符串

最常见的转换是将任何类型转换为`string`变量，以便输出为人类可读的文本，因此所有类型都从`System.Object`类继承了一个名为`ToString`的方法。

`ToString`方法将任何变量的当前值转换为文本表示。某些类型无法合理地表示为文本，因此它们返回其命名空间和类型名称。

让我们将一些类型转换为`字符串`：

1.  键入语句以声明一些变量，将它们转换为其`字符串`表示，并将它们写入控制台，如下所示：

    ```cs
    int number = 12; 
    WriteLine(number.ToString());
    bool boolean = true; 
    WriteLine(boolean.ToString());
    DateTime now = DateTime.Now; 
    WriteLine(now.ToString());
    object me = new(); 
    WriteLine(me.ToString()); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    12
    True
    02/28/2021 17:33:54
    System.Object 
    ```

## 从二进制对象转换为字符串

当您有一个二进制对象（如图像或视频）想要存储或传输时，有时您不希望发送原始位，因为您不知道这些位可能会如何被误解，例如，通过传输它们的网络协议或其他正在读取存储二进制对象的操作系统。

最安全的方法是将二进制对象转换为安全的字符`字符串`。程序员称这种编码为**Base64**。

`Convert`类型有一对方法，`ToBase64String`和`FromBase64String`，它们为您执行此转换。让我们看看它们的实际应用：

1.  键入语句以创建一个随机填充字节值的字节数组，将每个字节格式化良好地写入控制台，然后将相同的字节转换为 Base64 写入控制台，如下所示：

    ```cs
    // allocate array of 128 bytes
    byte[] binaryObject = new byte[128];
    // populate array with random bytes
    (new Random()).NextBytes(binaryObject); 
    WriteLine("Binary Object as bytes:");
    for(int index = 0; index < binaryObject.Length; index++)
    {
      Write($"{binaryObject[index]:X} ");
    }
    WriteLine();
    // convert to Base64 string and output as text
    string encoded = ToBase64String(binaryObject);
    WriteLine($"Binary Object as Base64: {encoded}"); 
    ```

    默认情况下，`int`值会假设为十进制表示法输出，即基数 10。您可以使用`：X`等格式代码以十六进制表示法格式化该值。

1.  运行代码并查看结果，如下所示：

    ```cs
    Binary Object as bytes:
    B3 4D 55 DE 2D E BB CF BE 4D E6 53 C3 C2 9B 67 3 45 F9 E5 20 61 7E 4F 7A 81 EC 49 F0 49 1D 8E D4 F7 DB 54 AF A0 81 5 B8 BE CE F8 36 90 7A D4 36 42
    4 75 81 1B AB 51 CE 5 63 AC 22 72 DE 74 2F 57 7F CB E7 47 B7 62 C3 F4 2D
    61 93 85 18 EA 6 17 12 AE 44 A8 D B8 4C 89 85 A9 3C D5 E2 46 E0 59 C9 DF
    10 AF ED EF 8AA1 B1 8D EE 4A BE 48 EC 79 A5 A 5F 2F 30 87 4A C7 7F 5D C1 D
    26 EE
    Binary Object as Base64: s01V3i0Ou8++TeZTw8KbZwNF +eUgYX5PeoHsSfBJHY7U99tU r6CBBbi+zvg2kHrUNkIEdYEbq1HOBWOsInLedC9Xf8vnR7diw/QtYZOFGOoGFxKuRKgNuEyJha k81eJG4FnJ3xCv7e+KobGN7kq+SO x5pQpfLzCHSsd/XcENJu4= 
    ```

## 从字符串解析到数字或日期和时间

第二常见的转换是从字符串到数字或日期和时间值。

`ToString`的相反操作是`Parse`。只有少数类型具有`Parse`方法，包括所有数字类型和`DateTime`。

让我们看看`Parse`的实际应用：

1.  键入语句以从字符串解析整数和日期时间值，然后将结果写入控制台，如下所示：

    ```cs
    int age = int.Parse("27");
    DateTime birthday = DateTime.Parse("4 July 1980");
    WriteLine($"I was born {age} years ago."); 
    WriteLine($"My birthday is {birthday}."); 
    WriteLine($"My birthday is {birthday:D}."); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    I was born 27 years ago.
    My birthday is 04/07/1980 00:00:00\. 
    My birthday is 04 July 1980. 
    ```

    默认情况下，日期和时间值以短日期和时间格式输出。您可以使用`D`等格式代码仅以长日期格式输出日期部分。

    **最佳实践**：使用标准日期和时间格式说明符，如下所示：[`docs.microsoft.com/en-us/dotnet/standard/base-types/standard-date-and-time-format-strings#table-of-format-specifiers`](https://docs.microsoft.com/en-us/dotnet/standard/base-types/standard-date-and-time-format-strings#table-of-format-specifiers)

### 使用 Parse 时的错误

`Parse`方法的一个问题是，如果`字符串`无法转换，它会给出错误。

1.  键入语句以尝试将包含字母的字符串解析为整数变量，如下所示：

    ```cs
    int count = int.Parse("abc"); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Unhandled Exception: System.FormatException: Input string was not in a correct format. 
    ```

除了上述异常消息外，你还会看到堆栈跟踪。由于堆栈跟踪占用太多空间，本书中未包含。

### 使用 TryParse 方法避免异常

为了避免错误，你可以使用`TryParse`方法。`TryParse`尝试转换输入的`string`，如果能够转换则返回`true`，否则返回`false`。

`out`关键字是必需的，以便`TryParse`方法在转换成功时设置计数变量。

让我们看看`TryParse`的实际应用：

1.  将`int` `count`声明替换为使用`TryParse`方法的语句，并要求用户输入鸡蛋的数量，如下所示：

    ```cs
    Write("How many eggs are there? "); 
    string? input = ReadLine(); // or use "12" in notebook
    if (int.TryParse(input, out int count))
    {
      WriteLine($"There are {count} eggs.");
    }
    else
    {
      WriteLine("I could not parse the input.");
    } 
    ```

1.  运行代码，输入`12`，查看结果，如下所示：

    ```cs
    How many eggs are there? 12
    There are 12 eggs. 
    ```

1.  运行代码，输入`twelve`（或在笔记本中将`string`值更改为`"twelve"`），查看结果，如下所示：

    ```cs
    How many eggs are there? twelve
    I could not parse the input. 
    ```

你还可以使用`System.Convert`类型的方法将`string`值转换为其他类型；然而，与`Parse`方法类似，如果无法转换，它会报错。

# 处理异常

你已经看到了几种类型转换时发生错误的场景。有些语言在出现问题时返回错误代码。.NET 使用异常，这些异常比返回值更丰富，专门用于失败报告，而返回值有多种用途。当这种情况发生时，我们说*运行时异常已被抛出*。

当抛出异常时，线程会被挂起，如果调用代码定义了`try-catch`语句，那么它就有机会处理这个异常。如果当前方法没有处理它，那么它的调用方法就有机会处理，以此类推，直到调用栈顶。

如你所见，控制台应用程序或.NET Interactive 笔记本的默认行为是输出有关异常的消息，包括堆栈跟踪，然后停止运行代码。应用程序终止。这比允许代码在可能损坏的状态下继续执行要好。你的代码应该只捕获和处理它理解并能正确修复的异常。

**良好实践**：尽量避免编写可能抛出异常的代码，或许可以通过执行`if`语句检查来实现。有时你做不到，有时最好让更高层次的组件来捕获调用你代码时抛出的异常。你将在*第四章*，*编写、调试和测试函数*中学习如何做到这一点。

## 将易出错的代码包裹在 try 块中

当你知道某个语句可能引发错误时，应该将该语句包裹在`try`块中。例如，从文本解析为数字可能引发错误。`catch`块中的任何语句只有在`try`块中的语句抛出异常时才会执行。

我们无需在`catch`块中做任何事情。让我们看看实际操作：

1.  使用您喜欢的编码工具，在`Chapter03`工作区/解决方案中添加一个新的**控制台应用程序**，命名为`HandlingExceptions`。

1.  在 Visual Studio Code 中，选择`HandlingExceptions`作为活动 OmniSharp 项目。

1.  输入语句以提示用户输入他们的年龄，然后将他们的年龄写入控制台，如下所示：

    ```cs
    WriteLine("Before parsing"); 
    Write("What is your age? "); 
    string? input = ReadLine(); // or use "49" in a notebook
    try
    {
      int age = int.Parse(input); 
      WriteLine($"You are {age} years old.");
    }
    catch
    {
    }
    WriteLine("After parsing"); 
    ```

    您将看到以下编译器消息：`Warning CS8604 Possible null reference argument for parameter 's' in 'int int.Parse(string s)'`。在新建的.NET 6 项目中，默认情况下，Microsoft 已启用可空引用类型，因此您会看到更多此类编译器警告。在生产代码中，您应添加代码以检查`null`并适当地处理这种可能性。在本书中，我不会包含这些`null`检查，因为代码示例并非设计为生产质量，并且到处都是`null`检查会使得代码杂乱无章，占用宝贵的页面。在这种情况下，`input`不可能为`null`，因为用户必须按 Enter 键，`ReadLine`才会返回，而那将返回一个空`string`。您将在本书的代码示例中看到数百个可能为`null`的变量。对于本书的代码示例，这些警告可以安全地忽略。只有在编写自己的生产代码时，您才需要类似的警告。您将在*第六章*，*实现接口和继承类*中了解更多关于空处理的内容。

    此代码包含两个消息，以指示*解析前*和*解析后*，使代码流程更清晰。随着示例代码变得更加复杂，这些将特别有用。

1.  运行代码，输入`49`，查看结果，如下所示：

    ```cs
    Before parsing
    What is your age? 49
    You are 49 years old. 
    After parsing 
    ```

1.  运行代码，输入`Kermit`，查看结果，如下所示：

    ```cs
    Before parsing
    What is your age? Kermit
    After parsing 
    ```

当代码执行时，错误异常被捕获，默认消息和堆栈跟踪未输出，控制台应用程序继续运行。这比默认行为更好，但查看发生的错误类型可能会有所帮助。

**良好实践**：在生产代码中，您绝不应使用像这样的空`catch`语句，因为它会“吞噬”异常并隐藏潜在问题。如果您无法或不想妥善处理异常，至少应记录该异常，或者重新抛出它，以便更高级别的代码可以决定如何处理。您将在*第四章*，*编写、调试和测试函数*中学习有关日志记录的内容。

### 捕获所有异常

要获取可能发生的任何类型异常的信息，您可以在`catch`块中声明一个类型为`System.Exception`的变量：

1.  在`catch`块中添加一个异常变量声明，并使用它将有关异常的信息写入控制台，如下所示：

    ```cs
    catch (Exception ex)
    {
      WriteLine($"{ex.GetType()} says {ex.Message}");
    } 
    ```

1.  再次运行代码，输入`Kermit`，查看结果，如下所示：

    ```cs
    Before parsing
    What is your age? Kermit
    System.FormatException says Input string was not in a correct format. 
    After parsing 
    ```

### 捕获特定异常

既然我们知道发生了哪种类型的特定异常，我们可以通过仅捕获该类型的异常并自定义向用户显示的消息来改进我们的代码：

1.  保留现有的`catch`块，并在其上方添加一个新的`catch`块，用于格式异常类型，如下面的突出显示代码所示：

    ```cs
    **catch (FormatException)**
    **{**
     **WriteLine(****"The age you entered is not a valid number format."****);**
    **}**
    catch (Exception ex)
    {
      WriteLine($"{ex.GetType()} says {ex.Message}");
    } 
    ```

1.  运行代码，再次输入`Kermit`，并查看结果，如下面的输出所示：

    ```cs
    Before parsing
    What is your age? Kermit
    The age you entered is not a valid number format. 
    After parsing 
    ```

    我们希望保留下面更一般的 catch 块的原因是，可能会有其他类型的异常发生。

1.  运行代码，输入`9876543210`，并查看结果，如下面的输出所示：

    ```cs
    Before parsing
    What is your age? 9876543210
    System.OverflowException says Value was either too large or too small for an Int32.
    After parsing 
    ```

    让我们为这种类型的异常再添加一个`catch`块。

1.  保留现有的`catch`块，并添加一个新的`catch`块，用于溢出异常类型，如下面的突出显示代码所示：

    ```cs
    **catch (OverflowException)**
    **{**
     **WriteLine(****"Your age is a valid number format but it is either too big or small."****);**
    **}**
    catch (FormatException)
    {
      WriteLine("The age you entered is not a valid number format.");
    } 
    ```

1.  运行代码，输入`9876543210`，并查看结果，如下面的输出所示：

    ```cs
    Before parsing
    What is your age? 9876543210
    Your age is a valid number format but it is either too big or small. 
    After parsing 
    ```

捕获异常的顺序很重要。正确的顺序与异常类型的继承层次结构有关。您将在*第五章*，*使用面向对象编程构建自己的类型*中学习继承。不过，不必过于担心这一点——如果异常顺序错误，编译器会给您构建错误。

**最佳实践**：避免过度捕获异常。通常应允许它们向上传播到调用堆栈，以便在更了解情况的层级处理，这可能会改变处理逻辑。您将在*第四章*，*编写、调试和测试函数*中学习到这一点。

### 使用过滤器捕获

您还可以使用`when`关键字在 catch 语句中添加过滤器，如下面的代码所示：

```cs
Write("Enter an amount: ");
string? amount = ReadLine();
try
{
  decimal amountValue = decimal.Parse(amount);
}
catch (FormatException) when (amount.Contains("$"))
{
  WriteLine("Amounts cannot use the dollar sign!");
}
catch (FormatException)
{
  WriteLine("Amounts must only contain digits!");
} 
```

# 检查溢出

之前，我们看到在数字类型之间转换时，可能会丢失信息，例如，将`long`变量转换为`int`变量时。如果存储在类型中的值太大，它将溢出。

## 使用 checked 语句抛出溢出异常

`checked`语句告诉.NET 在发生溢出时抛出异常，而不是默认允许它静默发生，这是出于性能原因。

我们将一个`int`变量的初始值设为其最大值减一。然后，我们将它递增几次，每次输出其值。一旦它超过其最大值，它就会溢出到最小值，并从那里继续递增。让我们看看这个过程：

1.  使用您喜欢的编程工具，在`Chapter03`工作区/解决方案中添加一个新的**控制台应用程序**，命名为`CheckingForOverflow`。

1.  在 Visual Studio Code 中，选择`CheckingForOverflow`作为活动的 OmniSharp 项目。

1.  在`Program.cs`中，键入语句以声明并赋值一个整数，其值为其最大可能值减一，然后递增它并在控制台上写入其值三次，如下面的代码所示：

    ```cs
    int x = int.MaxValue - 1; 
    WriteLine($"Initial value: {x}"); 
    x++;
    WriteLine($"After incrementing: {x}"); 
    x++;
    WriteLine($"After incrementing: {x}"); 
    x++;
    WriteLine($"After incrementing: {x}"); 
    ```

1.  运行代码并查看结果，显示值无声溢出并环绕至大负值，如下所示：

    ```cs
    Initial value: 2147483646
    After incrementing: 2147483647
    After incrementing: -2147483648
    After incrementing: -2147483647 
    ```

1.  现在，让我们通过使用`checked`语句块包裹这些语句，让编译器警告我们关于溢出的问题，如下所示：

    ```cs
    **checked**
    **{**
      int x = int.MaxValue - 1; 
      WriteLine($"Initial value: {x}"); 
      x++;
      WriteLine($"After incrementing: {x}"); 
      x++;
      WriteLine($"After incrementing: {x}"); 
      x++;
      WriteLine($"After incrementing: {x}");
    **}** 
    ```

1.  运行代码并查看结果，显示溢出检查导致异常抛出，如下所示：

    ```cs
    Initial value: 2147483646
    After incrementing: 2147483647
    Unhandled Exception: System.OverflowException: Arithmetic operation resulted in an overflow. 
    ```

1.  与其他异常一样，我们应该将这些语句包裹在`try`语句块中，并为用户显示更友好的错误消息，如下所示：

    ```cs
    try
    {
      // previous code goes here
    }
    catch (OverflowException)
    {
      WriteLine("The code overflowed but I caught the exception.");
    } 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Initial value: 2147483646
    After incrementing: 2147483647
    The code overflowed but I caught the exception. 
    ```

## 使用 unchecked 语句禁用编译器溢出检查

上一节讨论了*运行时*默认的溢出行为以及如何使用`checked`语句改变这种行为。本节将探讨*编译时*溢出行为以及如何使用`unchecked`语句改变这种行为。

相关关键字是`unchecked`。此关键字在代码块内关闭编译器执行的溢出检查。让我们看看如何操作：

1.  在之前的语句末尾输入以下语句。编译器不会编译此语句，因为它知道这将导致溢出：

    ```cs
    int y = int.MaxValue + 1; 
    ```

1.  将鼠标悬停在错误上，注意编译时检查显示为错误消息，如*图 3.1*所示：![图形用户界面，文本，应用程序，电子邮件 自动生成描述](img/B17442_03_01.png)

    图 3.1：PROBLEMS 窗口中的编译时检查

1.  要禁用编译时检查，将语句包裹在`unchecked`块中，将`y`的值写入控制台，递减它，并重复，如下所示：

    ```cs
    unchecked
    {
      int y = int.MaxValue + 1; 
      WriteLine($"Initial value: {y}"); 
      y--;
      WriteLine($"After decrementing: {y}"); 
      y--;
      WriteLine($"After decrementing: {y}");
    } 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Initial value: -2147483648
    After decrementing: 2147483647
    After decrementing: 2147483646 
    ```

当然，你很少会想要明确关闭这种检查，因为它允许溢出发生。但或许你能想到一个可能需要这种行为的场景。

# 实践与探索

通过回答一些问题来测试你的知识和理解，进行一些实践操作，并深入研究本章的主题。

## 练习 3.1 – 测试你的知识

回答以下问题：

1.  当`int`变量除以`0`时，会发生什么？

1.  当`double`变量除以`0`时，会发生什么？

1.  当`int`变量的值超出其范围（即溢出）时，会发生什么？

1.  `x = y++;`和`x = ++y;`之间有何区别？

1.  在循环语句中使用`break`、`continue`和`return`有何区别？

1.  `for`语句的三个部分是什么，哪些是必需的？

1.  `=`和`==`操作符之间有何区别？

1.  以下语句能否编译？

    ```cs
    for ( ; true; ) ; 
    ```

1.  在`switch`表达式中，下划线`_`代表什么？

1.  对象必须实现什么接口才能通过`foreach`语句进行枚举？

## 练习 3.2 – 探索循环和溢出

如果执行这段代码会发生什么？

```cs
int max = 500;
for (byte i = 0; i < max; i++)
{
  WriteLine(i);
} 
```

在`Chapter03`中创建一个名为`Exercise02`的控制台应用程序，并输入前面的代码。运行控制台应用程序并查看输出。发生了什么？

你可以添加什么代码（不要更改前面的任何代码）来警告我们这个问题？

## 练习 3.3 – 练习循环和运算符

**FizzBuzz** 是一种团体文字游戏，旨在教孩子们关于除法的知识。玩家轮流递增计数，用单词*fizz*替换任何可被三整除的数字，用单词*buzz*替换任何可被五整除的数字，用*fizzbuzz*替换任何可被两者整除的数字。

在`Chapter03`中创建一个名为`Exercise03`的控制台应用程序，输出一个模拟的 FizzBuzz 游戏，计数到 100。输出应类似于*图 3.2*：

![文本描述自动生成](img/B17442_03_02.png)

图 3.2：模拟的 FizzBuzz 游戏输出

## 练习 3.4 – 练习异常处理

在`Chapter03`中创建一个名为`Exercise04`的控制台应用程序，要求用户输入两个 0-255 范围内的数字，然后将第一个数字除以第二个数字：

```cs
Enter a number between 0 and 255: 100
Enter another number between 0 and 255: 8
100 divided by 8 is 12 
```

编写异常处理程序以捕获任何抛出的错误，如下面的输出所示：

```cs
Enter a number between 0 and 255: apples
Enter another number between 0 and 255: bananas 
FormatException: Input string was not in a correct format. 
```

## 练习 3.5 – 测试你对运算符的知识

执行以下语句后，`x`和`y`的值是什么？

1.  增量和加法运算符：

    ```cs
    x = 3;
    y = 2 + ++x; 
    ```

1.  二进制移位运算符：

    ```cs
    x = 3 << 2;
    y = 10 >> 1; 
    ```

1.  位运算符：

    ```cs
    x = 10 & 8;
    y = 10 | 7; 
    ```

## 练习 3.6 – 探索主题

使用下一页上的链接，详细了解本章涵盖的主题：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-3---controlling-flow-and-converting-types`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-3---controlling-flow-and-converting-types)

# 总结

本章中，你尝试了一些运算符，学习了如何分支和循环，如何进行类型转换，以及如何捕获异常。

你现在准备好学习如何通过定义函数重用代码块，如何向它们传递值并获取返回值，以及如何追踪代码中的错误并消除它们！
