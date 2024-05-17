# 第三章：03

# 控制流，类型转换和异常处理

本章主要讲述编写对变量执行简单操作的代码，做出决策，执行模式匹配，重复语句或块，将变量或表达式值从一种类型转换为另一种类型，处理异常，并检查数字变量的溢出。

本章涵盖以下主题：

+   对变量进行操作

+   理解选择语句

+   理解迭代语句

+   转换和类型转换

+   处理异常

+   检查溢出

# 对变量进行操作

**运算符**对**操作数**（如变量和文字值）应用简单的操作，如加法和乘法。它们通常返回一个新值，这个值是操作的结果，可以分配给一个变量。

大多数运算符是二元的，意味着它们作用于两个操作数，如下伪代码所示：

```cs

var

resultOfOperation = firstOperand operator

secondOperand;

```

二元运算符的示例包括加法和乘法，如下代码所示：

```cs

int

x = 5

;

int

y = 3

;

int

resultOfAdding = x + y;

int

resultOfMultiplying = x * y;

```

一些运算符是一元的，意味着它们作用于单个操作数，并且可以在操作数之前或之后应用，如下伪代码所示：

```cs

var

resultOfOperation = onlyOperand operator

;

var

resultOfOperation2 = operator

onlyOperand;

```

一元运算符的示例包括递增器和检索类型或其大小（以字节为单位），如下代码所示：

```cs

int

x = 5

;

int

后缀递增= x++;

int

前缀递增= ++x;

整数的类型 theTypeOfAnInteger = typeof

(int

);

int

howManyBytesInAnInteger = sizeof

(int

);

```

三元运算符作用于三个操作数，如下伪代码所示：

```cs

var

resultOfOperation = firstOperand firstOperator

secondOperand secondOperator thirdOperand;

```

## 探索一元运算符

两个常见的一元运算符用于递增，`++`，和递减，`--`，一个数字。让我们编写一些示例代码来展示它们是如何工作的：

1.  如果您已经完成了前几章，那么您已经有了一个`Code`文件夹。如果没有，那么您需要创建它。

1.  使用您喜欢的编码工具创建一个新的控制台应用程序，如下列表所示：

1.  项目模板：**控制台应用程序** / `控制台`

1.  工作区/解决方案文件和文件夹：`第三章`

1.  项目文件和文件夹：`运算符`

1.  在`Program.cs`的顶部，静态导入`System.Console`。

1.  在`Program.cs`中，声明两个名为`a`和`b`的整数变量，将`a`设置为`3`，在将结果分配给`b`的同时递增`a`，然后输出它们的值，如下代码所示：

```cs

int

a = 3

;

int

b = a++;

WriteLine($"a 是

{a}

，b 是

{b}

"

);

```

1.  在运行控制台应用程序之前，问问自己一个问题：当输出时，你认为`b`的值会是多少？一旦你考虑过这个问题，运行代码，并将你的预测与实际结果进行比较，如下输出所示：

```cs

a 是 4，b 是 3

```

变量`b`的值为`3`，因为`++`运算符在分配之后执行；这被称为**后缀运算符**。如果您需要在分配之前递增，那么使用**前缀运算符**。

1.  复制并粘贴语句，然后修改它们以重命名变量并使用前缀运算符，如下代码所示：

```cs

int

c = 3

;

int

d = ++c; // 在分配之前递增 c

WriteLine($"c 是

{c}

，d 是

{d}

"

);

```

1.  重新运行代码并注意结果，如下输出所示：

```cs

a 是 4，b 是 3

c 是 4，d 是 4

```

**良好的实践**：由于 Swift 编程语言设计者在版本 3 中决定放弃对这个运算符的支持，因为在与赋值运算符`=`结合使用时，增量和减量运算符的前缀和后缀之间的混淆，我建议在 C#中永远不要将`++`和`--`运算符与赋值运算符`=`结合使用。将操作作为单独的语句执行。

## 探索二进制算术运算符

增量和减量是一元算术运算符。其他算术运算符通常是二元的，允许您对两个数字执行算术运算，如下所示：

1.  添加语句来声明并为名为`e`和`f`的两个整数变量分配值，然后应用这两个数字的五个常见二进制算术运算符，如下面的代码所示：

```cs

int

e = 11

;

int

f = 3

;

WriteLine($"e is

{e}

, f is

{f}

"

);

WriteLine($"e + f =

{e + f}

"

);

WriteLine($"e - f =

{e - f}

"

;

WriteLine($"e * f =

{e * f}

"

);

WriteLine($"e / f =

{e / f}

"

);

WriteLine($"e % f =

{e % f}

"

);

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

e 是 11，f 是 3

e + f = 14

e - f = 8

e * f = 33

e / f = 3

e % f = 2

```

要理解应用于整数的除法`/`和模数`%`运算符，您需要回想起小学时的情景。想象一下你有十一颗糖果和三个朋友。

你如何在朋友之间分糖果？你可以给每个朋友三颗糖果，然后还剩两颗。这两颗糖果是**模数**，也称为除法后的**余数**。如果你有十二颗糖果，那么每个朋友得到四颗，没有剩下的，所以余数是 0。

1.  添加语句来声明并为名为`g`的`double`变量分配一个值，以显示整数和实数除法之间的区别，如下面的代码所示：

```cs

double

g = 11.0

;

WriteLine($"g is

{g:N1}

, f is

{f}

"

);

WriteLine($"g / f =

{g / f}

"

);

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

g 是 11.0，f 是 3

g / f = 3.6666666666666665

```

如果第一个操作数是浮点数，例如值为`11.0`的`g`，那么除法运算符返回浮点数值，例如`3.6666666666665`，而不是整数。

## 赋值运算符

您已经在使用最常见的赋值运算符`=`。

为了使您的代码更加简洁，您可以将赋值运算符与算术运算符等其他运算符结合起来，如下面的代码所示：

```cs

int

p = 6

;

p += 3

; // 等同于 p = p + 3;

p -= 3

; // 等同于 p = p - 3;

p *= 3

; // 等同于 p = p * 3;

p /= 3

; // 等同于 p = p / 3;

```

## 探索逻辑运算符

逻辑运算符操作布尔值，因此它们返回`true`或`false`。让我们探索作用于两个布尔值的二进制逻辑运算符：

1.  使用您喜欢的编码工具将一个新的控制台应用添加到名为`BooleanOperators`的`Chapter03`工作区/解决方案中。

1.  在 Visual Studio Code 中，选择`BooleanOperators`作为活动的 OmniSharp 项目。当您看到弹出的警告消息说缺少所需的资产时，点击**是**以添加它们。

1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择。

**良好的实践**：记住静态导入`System.Console`类型以简化语句。

1.  在`Program.cs`中，添加语句来声明两个布尔变量，值为`true`和`false`，然后输出应用 AND、OR 和 XOR（异或）逻辑运算符的真值表，如下面的代码所示：

```cs

bool

a = true

;

bool

b = false

;

WriteLine($"AND  | a     | b    "

);

WriteLine($"a    |

{a & a,

-5

}

|

{a & b,

-5

}

"

);

WriteLine($"b    |

{b & a,

-5

}

|

{b & b,

-5

}

"

);

WriteLine();

WriteLine($"OR   | a     | b    "

);

WriteLine($"a    |

{a | a,

-5

}

|

{a | b,

-5

}

"

);

WriteLine($"b    |

{b | a，

-5

}

|

{b | b，

-5

}

"

）;

WriteLine（）;

WriteLine（$“XOR  | a     | b    "

）;

WriteLine（$“a    |

{a ^ a，

-5

}

|

{a ^ b，

-5

}

"

）;

WriteLine（$“b    |

{b ^ a，

-5

}

|

{b ^ b，

-5

}

"

）;

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

AND  | a     | b

a    | 真  | 假

b    | 假 | 假

OR   | a     | b

a    | 真  | 真

b    | 真  | 假

XOR  | a     | b

a    | 假 | 真

b    | 真  | 假

```

对于 AND `&`逻辑运算符，两个操作数必须都为`true`才能得到`true`的结果。对于 OR `|`逻辑运算符，任一操作数都可以为`true`才能得到`true`的结果。对于 XOR `^`逻辑运算符，任一操作数都可以为`true`（但不能都为真！）才能得到`true`的结果。

## 探索条件逻辑运算符

条件逻辑运算符类似于逻辑运算符，但您使用两个符号而不是一个，例如，`&&`而不是`&`，或`||`而不是`|`。

在*第四章*，*编写、调试和测试函数*中，您将更详细地了解函数，但我现在需要介绍函数来解释条件逻辑运算符，也称为短路布尔运算符。

函数执行语句，然后返回一个值。该值可以是布尔值，如`true`，用于布尔运算。让我们利用条件逻辑运算符：

1.  在`Program.cs`的底部，编写语句来声明一个向控制台写入消息并返回`true`的函数，如下面的代码所示：

```cs

静态

布尔

DoStuff

（）

{

WriteLine（“我正在做一些事情。”

）;

返回

true

;

}

```

**良好实践**：如果您正在使用.NET 交互式笔记本，请在单独的代码单元中编写`DoStuff`函数，然后执行它，以使其上下文可用于其他代码单元。

1.  在之前的`WriteLine`语句之后，对`a`和`b`变量以及调用函数的结果执行 AND `&`操作，如下面的代码所示：

```cs

WriteLine（）;

WriteLine（$“a & DoStuff（）=

{a & DoStuff（）}

"

）;

WriteLine（$“b & DoStuff（）=

{b & DoStuff（）}

"

）;

```

1.  运行代码，查看结果，并注意函数被调用了两次，一次是为 a，一次是为 b，如下面的输出所示：

```cs

我正在做一些事情。

a & DoStuff（）= 真

我正在做一些事情。

b & DoStuff（）= 假

```

1.  将`&`运算符更改为`&&`运算符，如下面的代码所示：

```cs

WriteLine（$“a && DoStuff（）=

{a && DoStuff（）}

"

）;

WriteLine（$“b && DoStuff（）=

{b && DoStuff（）}

"

）;

```

1.  运行代码，查看结果，并注意当与变量`a`结合时，函数确实运行。当与变量`b`结合时，它不会运行，因为变量`b`是`false`，所以结果将是`false`，因此不需要执行函数，如下面的输出所示：

```cs

我正在做一些事情。

a && DoStuff（）= 真

b && DoStuff（）= 假 // DoStuff 函数未被执行！

```

**良好实践**：现在你可以看到为什么条件逻辑运算符被描述为短路。它们可以使您的应用程序更有效，但也可能在您假定函数总是被调用的情况下引入微妙的错误。在与引起副作用的函数结合使用时，最安全的做法是避免它们。

## 探索位运算和二进制移位运算符

位运算符影响数字中的位。二进制移位运算符可以比传统运算符更快地执行一些常见的算术计算，例如，任何乘以 2 的因子。

让我们探索位运算和二进制移位运算符：

1.  使用您喜欢的编码工具将一个新的**控制台应用程序**添加到名为`BitwiseAndShiftOperators`的`Chapter03`工作区/解决方案中。

1.  在 Visual Studio Code 中，将`BitwiseAndShiftOperators`选择为活动的 OmniSharp 项目。当看到弹出的警告消息说缺少所需的资产时，点击**是**添加它们。

1.  在 `Program.cs` 中，输入语句声明两个整数变量，值分别为 10 和 6，然后输出应用 AND、OR 和 XOR 位运算符的结果，如下面的代码所示：

```cs

int

a = 10

; // 00001010

int

b = 6

;  // 00000110

WriteLine($"a =

{a}

"

);

WriteLine($"b =

{b}

"

);

WriteLine($"a & b =

{a & b}

"

); // 2-bit column only

WriteLine($"a | b =

{a | b}

"

); // 8, 4, and 2-bit columns

WriteLine($"a ^ b =

{a ^ b}

"

); // 8 and 4-bit columns

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

a = 10

b = 6

a & b = 2

a | b = 14

a ^ b = 12

```

1.  在 `Program.cs` 中，添加语句输出将变量 `a` 的位移三列，将 `a` 乘以 8，以及将变量 `b` 的位移一列的结果，如下面的代码所示：

```cs

// 01010000 将 a 左移三位

WriteLine($"a << 3 =

{a <<

3

}

"

);

// 将 a 乘以 8

WriteLine($"a * 8 =

{a *

8

}

"

);

// 00000011 将 b 向右移动一位列

WriteLine($"b >> 1 =

{b >>

1

}

"

);

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

a << 3 = 80

a * 8 = 80

b >> 1 = 3

```

`80` 的结果是因为其中的位向左移了三列，所以 1 位移入了 64 位和 16 位列，64 + 16 = 80。这相当于乘以 8，但 CPU 可以更快地执行位移。`3` 的结果是因为 `b` 中的 1 位向右移了一列，进入了 2 位和 1 位列。

**良好实践**：记住，在操作整数值时，`&` 和 `|` 符号是位运算符，在操作像 `true` 和 `false` 这样的布尔值时，`&` 和 `|` 符号是逻辑运算符。

我们可以通过将整数值转换为由零和一组成的二进制字符串来说明这些操作：

1.  在 `Program.cs` 底部，添加一个函数，将整数值转换为最多八个零和一的二进制（Base2）`string`，如下面的代码所示：

```cs

static

string

ToBinaryString

(

int

value

)

{

return

Convert.ToString(value

, toBase: 2

).PadLeft(8

, '0'

);

}

```

1.  在函数上方，添加语句输出 `a` 、 `b` 和各种位运算符的结果，如下面的代码所示：

```cs

WriteLine();

WriteLine("将整数输出为二进制："

);

WriteLine($"a =

{ToBinaryString(a)}

"

);

WriteLine($"b =

{ToBinaryString(b)}

"

);

WriteLine($"a & b =

{ToBinaryString(a & b)}

"

);

WriteLine($"a | b =

{ToBinaryString(a | b)}

"

);

WriteLine($"a ^ b =

{ToBinaryString(a ^ b)}

"

);

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

将整数输出为二进制：

a =     00001010

b =     00000110

a & b = 00000010

a | b = 00001110

a ^ b = 00001100

```

## 其他运算符

`nameof` 和 `sizeof` 在处理类型时是方便的运算符：

+   `nameof` 在处理类型时是一个方便的运算符，它返回变量、类型或成员的短名称（不包括命名空间）作为 `string` 值，这在输出异常消息时很有用。

+   `sizeof` 返回简单类型的字节大小，这对于确定数据存储的效率很有用。

还有许多其他运算符；例如，变量和其成员之间的点被称为**成员访问运算符**，函数或方法名称末尾的圆括号被称为**调用运算符**，如下面的代码所示：

```cs

int

age = 47

;

// 以下语句中有多少个运算符？

char

firstDigit = age.ToString()[0

];

// 有四个运算符：

// = 是赋值运算符

// . 是成员访问运算符

// () 是调用运算符

// [] 是索引器访问运算符

```

# 理解选择语句

每个应用程序都需要能够从选择中进行选择，并沿着不同的代码路径进行分支。C#中的两个选择语句是`if`和`switch`。您可以在所有代码中使用`if`，但在一些常见情况下，例如存在一个可以具有多个值并且每个值都需要不同处理的单个变量时，`switch`可以简化您的代码。

## 使用 if 语句进行分支

`if`语句通过评估布尔表达式来确定要遵循哪个分支。如果表达式为`true`，则执行该块。`else`块是可选的，如果`if`表达式为`false`，则执行。`if`语句可以嵌套。

`if`语句可以与其他`if`语句组合为`else if`分支，如下面的代码所示：

```cs

如果

（表达式 1）

{

//当 expression1 为 true 时运行

}

否则

如果

（表达式 2）

{

//当 expression1 为 false 且 expression2 为 true 时运行

}

否则

如果

（表达式 3）

{

//当 expression1 和 expression2 为 false 时运行

//并且 expression3 为 true

}

否则

{

//当所有表达式都为 false 时运行

}

```

每个`if`语句的布尔表达式是独立的，不像`switch`语句那样，不需要引用单个值。

让我们编写一些代码来探索`if`等选择语句：

1.  使用您喜欢的编码工具向`Chapter03`工作区/解决方案中添加一个新的**控制台应用程序**，命名为`SelectionStatements`。

1.  在 Visual Studio Code 中，选择`SelectionStatements`作为活动的 OmniSharp 项目。

1.  在`Program.cs`中，键入语句以检查密码是否至少有八个字符，如下面的代码所示：

```cs

字符串

密码="ninja"

;

如果

（密码长度<8

）

{

WriteLine("您的密码太短。至少使用 8 个字符。"

);

}

否则

{

WriteLine("您的密码很强壮。"

);

}

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

您的密码太短。至少使用 8 个字符。

```

### 为什么您应该始终在 if 语句中使用大括号

由于每个块内只有一个语句，因此前面的代码可以不使用大括号编写，如下面的代码所示：

```cs

如果

（密码长度<8

）

WriteLine("您的密码太短。至少使用 8 个字符。"

);

否则

WriteLine("您的密码很强壮。"

);

```

应该避免使用这种风格的`if`语句，因为它可能会引入严重的错误，例如苹果 iPhone iOS 操作系统中臭名昭著的#gotofail 错误。

苹果 iOS 6 发布后的 18 个月内，即 2012 年 9 月，其**安全套接字层**（**SSL**）加密代码中存在一个错误，这意味着运行 Safari 的任何用户，设备的网络浏览器，尝试连接到安全网站（例如他们的银行）时，由于一个重要的检查被意外跳过，因此并不安全。

仅仅因为您可以省略大括号并不意味着您应该这样做。没有它们，您的代码并不会“更高效”；相反，它会更难维护，可能更危险。

## 使用 if 语句进行模式匹配

C# 7.0 及更高版本引入的一个功能是模式匹配。`if`语句可以使用`is`关键字与声明一个局部变量结合使用，使您的代码更安全：

1.  添加语句，以便如果存储在名为`o`的变量中的值是`int`，则将该值分配给名为`i`的局部变量，然后可以在`if`语句中使用。这比使用名为`o`的变量更安全，因为我们确切知道`i`是一个`int`变量，而不是其他东西，如下面的代码所示：

```cs

//添加和删除""以更改行为

对象

o="3"

;

int

j=4

;

如果

（o 是

int

i）

{

WriteLine($"```

{i}

x

{j}

=

{i*j}

"

);

}

否则

{

WriteLine("o 不是 int，因此无法相乘！"

);

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

o 不是 int，因此无法相乘！

```

1.  删除围绕“`3`”值的双引号字符，以便存储在名为`o`的变量中的值是`int`类型，而不是`string`类型。

1.  重新运行代码以查看结果，如下面的输出所示：

```cs

3 x 4 = 12

```

## 使用 switch 语句进行分支

`switch`语句与`if`语句不同，因为`switch`将单个表达式与多个可能的`case`语句列表进行比较。每个`case`语句都与单个表达式相关。每个`case`部分必须以：

+   `break`关键字（就像下面代码中的 case 1 一样）

+   或者`goto` `case`关键字（就像下面代码中的 case 2 一样）

+   或者它们不应该有语句（就像下面代码中的 case 3 一样）

+   或者引用命名标签的`goto`关键字（就像下面代码中的 case 5 一样）

+   或者`return`关键字离开当前函数（代码中未显示）

让我们写一些代码来探索`switch`语句：

1.  `switch`语句的类型语句。您应该注意，倒数第二个语句是可以跳转到的标签，第一个语句生成 1 到 6 之间的随机数（代码中的数字 7 是一个排他性的上限）。`switch`语句的分支基于这个随机数的值，如下面的代码所示：

```cs

int

number = (new

Random()).Next(1

，7

）;

WriteLine($"我的随机数是

{number}

"

);

开关

（number）

{

案例

1

：

WriteLine("一个"

);

休息

; // 跳转到 switch 语句的结尾

案例

2

：

WriteLine("二"

);

转到

案例

1

;

案例

3

： // 多个 case 部分

案例

4

：

WriteLine("三或四"

);

转到

案例

1

;

案例

5

：

转到

A_label;

默认

：

WriteLine("默认"

）;

休息

;

} // 结束 switch 语句

WriteLine("开关结束后"

）;

A_label:

WriteLine($"A_label 之后"

);

```

**良好实践**：您可以使用`goto`关键字跳转到另一个 case 或标签。`goto`关键字受到大多数程序员的反对，但在某些情况下可以成为代码逻辑的良好解决方案。但是，您应该谨慎使用它。

1.  多次运行代码以查看在各种随机数情况下发生的情况，如下面的示例输出所示：

```cs

// 第一次随机运行

我的随机数是 4

三或四

一个

开关结束后

A_label 之后

// 第二次随机运行

我的随机数是 2

二

一个

开关结束后

A_label 之后

// 第三次随机运行

我的随机数是 6

默认

开关结束后

A_label 之后

// 第四次随机运行

我的随机数是 1

一个

开关结束后

A_label 之后

// 第五次随机运行

我的随机数是 5

A_label 之后

```

## 使用 switch 语句进行模式匹配

与`if`语句一样，`switch`语句支持 C# 7.0 及更高版本中的模式匹配。`case`值不再需要是文字值；它们可以是模式。

让我们看一个使用文件夹路径进行模式匹配的`switch`语句的示例。如果您使用的是 macOS，则交换注释语句，设置路径变量并用您的用户文件夹名称替换我的用户名：

1.  添加语句声明`string`路径到文件，将其打开为只读或可写流，然后根据流的类型和功能显示消息，如下面的代码所示：

```cs

// string path = "/Users/markjprice/Code/Chapter03";

字符串

path = @"C:\Code\Chapter03"

;

Write("按 R 进行只读或按 W 进行可写："

）;

ConsoleKeyInfo key = ReadKey();

WriteLine();

流？s;

如果

（key.Key == ConsoleKey.R）

{

s =  File.Open(

Path.Combine(path, "file.txt"

），

FileMode.OpenOrCreate，

FileAccess.Read);

}

否则

{

s =  File.Open(

Path.Combine(path, "file.txt"

），

FileMode.OpenOrCreate，

FileAccess.Write);

}

字符串

消息;

开关

（s）

{

案例

FileStream writeableFile when

s.CanWrite:

message = "该流是我可以写入的文件。"

;

休息

;

案例

FileStream readOnlyFile:

message = "该流是只读文件。"

;

休息

;

案例

MemoryStream ms:

message = "该流是内存地址。"

;

休息

;

默认

：//始终在当前位置之后评估

消息=“流是其他类型。”

;

中断

;

案例

null

：

消息=“流为空。”

;

中断

;

}

WriteLine(message);

```

1.  运行代码并注意名为`s`的变量被声明为`Stream`类型，因此它可以是流的任何子类型，例如内存流或文件流。在此代码中，流是使用`File.Open`方法创建的，该方法返回文件流，并且根据您的按键，它将是可写或只读的，因此结果将是描述情况的消息，如下面的输出所示：

```cs

流是我可以写入的文件。

```

在.NET 中，有多个`Stream`的子类型，包括`FileStream`和`MemoryStream`。在 C# 7.0 及更高版本中，您的代码可以更简洁地分支，基于流的子类型，并声明和分配一个本地变量以安全地使用它。您将在*第九章*中了解有关`System.IO`命名空间和`Stream`类型的更多信息，*使用文件、流和序列化*。

此外，`case`语句可以包括`when`关键字以执行更具体的模式匹配。在前面代码中的第一个`case`语句中，只有当流是`FileStream`并且其`CanWrite`属性为`true`时，`s`才是匹配项。

## 使用 switch 表达式简化 switch 语句

在 C# 8.0 或更高版本中，您可以使用**switch 表达式**简化`switch`语句。

大多数`switch`语句非常简单，但是需要大量输入。`switch`表达式旨在简化您需要键入的代码，同时在所有情况下返回一个值以设置单个变量的情况下仍表达相同的意图。`switch`表达式使用 lambda，`=>`，表示返回值。

让我们使用`switch`表达式实现先前使用`switch`语句的代码，以便您可以比较这两种风格：

1.  编写语句以根据流的类型和功能设置消息，使用`switch`表达式，如下面的代码所示：

```cs

消息= s 开关

{

FileStream writeableFile when

s.CanWrite

= > “流是我可以写入的文件。”

，

FileStream readOnlyFile

= > “流是只读文件。”

，

MemoryStream ms

= > “流是内存地址。”

，

空

= > “流为空。”

，

_

= > “流是其他类型。”

};

WriteLine(message);

```

主要区别在于删除了`case`和`break`关键字。下划线字符`_`用于表示默认返回值。

1.  运行代码，注意结果与以前相同。

# 理解迭代语句

迭代语句重复一组语句，要么在条件为真时，要么对集合中的每个项目。选择使用哪个语句是基于解决逻辑问题的易理解性和个人偏好的组合。

## 使用 while 语句循环

`while`语句评估布尔表达式，并在其为真时继续循环。让我们探索迭代语句：

1.  使用您喜欢的编码工具将新的**控制台应用程序**添加到名为`IterationStatements`的`Chapter03`工作区/解决方案中。

1.  在 Visual Studio Code 中，选择`IterationStatements`作为活动的 OmniSharp 项目。

1.  在`Program.cs`中，编写语句以定义`while`语句，当整数变量的值小于 10 时循环，如下面的代码所示：

```cs

int

x = 0

;

而

(x < 10

）

{

WriteLine(x);

x ++;

}

```

1.  运行代码并查看结果，应该是 0 到 9 的数字，如下面的输出所示：

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

## 使用 do 语句循环

`do`语句类似于`while`，只是在块的底部检查布尔表达式，而不是在顶部检查，这意味着块始终至少执行一次，如下所示：

1.  编写语句以定义`do`循环，如下面的代码所示：

```cs

string

？密码;

做

{

Write("输入您的密码： "```

);

密码= ReadLine（）;

}

而

（密码！= "Pa$$w0rd"

）;

WriteLine（"正确！"

）;

```

1.  运行代码，并注意您将被提示重复输入密码，直到正确输入为止，如下面的输出所示：

```cs

输入密码：密码

输入密码：12345678

输入密码：忍者

输入密码：正确的马，电池，订书钉

输入密码：Pa$$w0rd

正确！

```

1.  作为可选挑战，添加语句，以便用户在显示错误消息之前只能尝试十次。

## 使用`for`语句循环

`for`语句类似于`while`，只是更简洁。它结合了：

+   一个**初始化表达式**，它在循环开始时执行一次。

+   一个**条件表达式**，它在循环开始时的每次迭代中执行，以检查循环是否应该继续。

+   一个**迭代器表达式**，它在语句底部的每次循环中执行。

`for`语句通常与整数计数器一起使用。让我们来探索一些代码：

1.  键入`for`语句以输出 1 到 10 的数字，如下面的代码所示：

```cs

为

（int

y = 1

; y <= 10

; y ++）

{

WriteLine（y）;

}

```

1.  运行代码以查看结果，应该是 1 到 10 的数字。

## 使用`foreach`语句循环

`foreach`语句与前三个迭代语句有些不同。

它用于对序列中的每个项目执行一组语句，例如数组或集合。每个项目通常是只读的，如果在迭代期间修改了序列结构，例如通过添加或删除项目，则会抛出异常。

尝试以下示例：

1.  键入语句以创建字符串变量的数组，然后输出每个字符串的长度，如下面的代码所示：

```cs

字符串

[] names = { "亚当"

，“巴里”

，“查理”

};

foreach

（字符串

名字在

名字）

{

WriteLine($"

{name}

有

{name.Length}

字符。"

）;

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

亚当有 4 个字符。

巴里有 5 个字符。

查理有 7 个字符。

```

### 了解`foreach`的内部工作原理

任何代表多个项目的类型的创建者，例如数组或集合，都应确保程序员可以使用`foreach`语句枚举类型的项目。

技术上，`foreach`语句将适用于遵循以下规则的任何类型：

1.  类型必须有一个名为`GetEnumerator`的方法，返回一个对象。

1.  返回的对象必须有一个名为`Current`的属性和一个名为`MoveNext`的方法。

1.  `MoveNext`方法必须更改`Current`的值并返回`true`，如果有更多项目要枚举，或者如果没有更多项目，则返回`false`。

有名为`IEnumerable`和`IEnumerable<T>`的接口，正式定义了这些规则，但从技术上讲，编译器不要求类型实现这些接口。

编译器将前面示例中的`foreach`语句转换为以下伪代码：

```cs

IEnumerator e = names.GetEnumerator（）;

而

（e.MoveNext（））

{

字符串

name =（string

）e.Current; // Current 是只读的！

WriteLine（$"

{name}

有

{name.Length}

字符。"

）;

}

```

由于使用了迭代器，`foreach`语句中声明的变量不能用于修改当前项目的值。

# 强制转换和类型转换

通常需要在不同类型的变量值之间进行转换。例如，数据输入通常以文本形式输入到控制台，因此最初存储在`string`类型的变量中，但随后需要将其转换为日期/时间，或数字，或其他数据类型，具体取决于应该如何存储和处理。

有时，您需要在数字类型之间进行转换，例如在整数和浮点数之间进行转换，然后执行计算。

转换也被称为**转换**，有两种类型：**隐式**和**显式**。隐式转换会自动发生，它是安全的，意味着您不会丢失任何信息。

由于可能会丢失信息，因此必须手动执行显式转换，例如数字的精度。通过显式转换，您告诉 C#编译器您理解并接受风险。

## 隐式和显式地转换数字

将`int`变量隐式转换为`double`变量是安全的，因为不会丢失任何信息，如下所示：

1.  使用您喜欢的编码工具将一个新的**控制台应用程序**添加到名为`Chapter03`的工作区/解决方案中，命名为`CastingConverting`。

1.  在 Visual Studio Code 中，选择`CastingConverting`作为活动的 OmniSharp 项目。

1.  在`Program.cs`中，输入语句来声明和分配一个`int`变量和一个`double`变量，然后在将整数值隐式转换为`double`变量时，如下面的代码所示：

```cs

int

a = 10

;

double

b = a; // int 可以安全地转换为 double

WriteLine(b);

```

1.  输入语句来声明和分配一个`double`变量和一个`int`变量，然后在将`double`值隐式转换为`int`变量时，如下面的代码所示：

```cs

double

c = 9.8

;

int

d = c; // 编译器会为这一行报错

WriteLine(d);

```

1.  运行代码并注意错误消息，如下面的输出所示：

```cs

错误：(6,9): error CS0266: Cannot implicitly convert type 'double' to 'int'. An explicit conversion exists (are you missing a cast?)

```

这个错误消息也会出现在 Visual Studio 错误列表或 Visual Studio Code 问题窗口中。

您不能将`double`变量隐式转换为`int`变量，因为这可能是不安全的，并且可能会丢失数据，比如小数点后的值。您必须使用一对圆括号将`double`变量显式转换为`int`变量。一对圆括号是**转换运算符**。即使如此，您也必须注意，小数点后的部分将被截断而没有警告，因为您选择执行显式转换，因此理解后果。

1.  修改`d`变量的赋值语句，如下面的代码所示：

```cs

int

d = (int

)c;

WriteLine(d); // d 是 9，丢失了.8 部分

```

1.  运行代码以查看结果，如下面的输出所示：

```cs

10

9

```

在将较大的整数值和较小的整数值之间转换值时，我们必须执行类似的操作。同样，要注意，您可能会丢失信息，因为任何太大的值都将被复制其位，然后以您可能意想不到的方式进行解释！

1.  输入语句来声明并将一个 64 位长整型变量分配给一个 32 位整型变量，两者都使用一个可以工作的小值和一个太大的值，如下面的代码所示：

```cs

long

e = 10

;

int

f = (int

)e;

WriteLine($"e is

{e:N0}

和 f 是

{f:N0}

"

);

e = long

.MaxValue;

f = (int

)e;

WriteLine($"e is

{e:N0}

和 f 是

{f:N0}

"

);

```

1.  运行代码以查看结果，如下面的输出所示：

```cs

e 是 10，f 是 10

e 是 9,223,372,036,854,775,807，f 是-1

```

1.  修改`e`的值为 50 亿，如下面的代码所示：

```cs

e = 5

_000_000_000;

```

1.  运行代码以查看结果，如下面的输出所示：

```cs

e 是 5,000,000,000，f 是 705,032,704

```

## 使用 System.Convert 类型进行转换

使用`System.Convert`类型的替代方法是使用`System.Convert`类型。`System.Convert`类型可以在 C#的所有数字类型之间进行转换，还可以在布尔值、字符串和日期时间值之间进行转换。

让我们写一些代码来看看它的运行情况：

1.  在`Program.cs`的顶部，静态导入`System.Convert`类，如下面的代码所示：

```cs

using

static

System.Convert;

```

1.  在`Program.cs`的底部，输入语句以声明并分配一个`double`变量的值，将其转换为整数，然后将两个值都写入控制台，如下面的代码所示：

```cs

双精度

g = 9.8

;

int

h = ToInt32(g); // System.Convert 的一个方法

WriteLine($"g 是

{g}

和 h 是

{h}

"

);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

g 是 9.8 和 h 是 10

```

强制转换和转换之间的一个区别是，转换将`double`值`9.8`四舍五入为`10`，而不是截取小数点后的部分。

## 四舍五入数字

您现在已经看到强制转换运算符截取实数的小数部分，而`System.Convert`方法四舍五入。然而，四舍五入的规则是什么呢？

### 了解默认的四舍五入规则

在英国的小学中，5 到 11 岁的学生被教导如果小数部分为.5 或更高，则四舍五入为*向上*，如果小数部分小于.5，则四舍五入为*向下*。

让我们探讨一下 C#是否遵循相同的小学规则：

1.  类型语句以声明并分配一个`double`值数组，将每个值转换为整数，然后将结果写入控制台，如下面的代码所示：

```cs

double

[] 双精度 = new

[]

{ 9.49

, 9.5

, 9.51

, 10.49

, 10.5

, 10.51

};

对于每个

（双精度

n 在

双精度)

{

WriteLine($"ToInt32(

{n}

）是

{ToInt32(n)}

"

);

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

ToInt32(9.49) 是 9

ToInt32(9.5) 是 10

ToInt32(9.51) 是 10

ToInt32(10.49) 是 10

ToInt32(10.5) 是 10

ToInt32(10.51) 是 11

```

我们已经证明了 C#中的四舍五入规则与小学规则略有不同：

+   如果小数部分小于中点.5，它总是四舍五入*向下*。

+   如果小数部分大于中点.5，它总是四舍五入*向上*。

+   如果小数部分是中点.5，非小数部分是*奇数*，它将四舍五入*向上*，但如果非小数部分是*偶数*，它将四舍五入*向下*。

这个规则被称为**银行家舍入**，它更受欢迎，因为它通过交替四舍五入来减少偏差。不幸的是，其他语言如 JavaScript 使用小学规则。

## 控制四舍五入规则

您可以通过使用`Math`类的`Round`方法来控制四舍五入规则：

1.  类型语句以使用“远离零”四舍五入规则（也称为“向上”四舍五入）对每个`double`值进行四舍五入，然后将结果写入控制台，如下面的代码所示：

```cs

对于每个

（双精度

n 在

双精度)

{

WriteLine(format:

"Math.Round({0}, 0, MidpointRounding.AwayFromZero) 是 {1}"

,

arg0: n,

arg1: Math.Round(value

: n, digits: 0

,

模式：MidpointRounding.AwayFromZero));

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

Math.Round(9.49, 0, MidpointRounding.AwayFromZero) 是 9

Math.Round(9.5, 0, MidpointRounding.AwayFromZero) 是 10

Math.Round(9.51, 0, MidpointRounding.AwayFromZero) 是 10

Math.Round(10.49, 0, MidpointRounding.AwayFromZero) 是 10

Math.Round(10.5, 0, MidpointRounding.AwayFromZero) 是 11

Math.Round(10.51, 0, MidpointRounding.AwayFromZero) 是 11

```

**良好实践**：对于您使用的每种编程语言，请检查其四舍五入规则。它们可能不会按照您的期望工作！

## 从任何类型转换为字符串

最常见的转换是从任何类型转换为`string`变量以输出为人类可读的文本，因此所有类型都有一个名为`ToString`的方法，它们从`System.Object`类继承而来。

`ToString`方法将当前变量的值转换为文本表示。一些类型无法合理地表示为文本，因此它们返回它们的命名空间和类型名称。

让我们将一些类型转换为`string`：

1.  类型语句以声明一些变量，将它们转换为它们的`string`表示，并将它们写入控制台，如下面的代码所示：

```cs

int

number = 12

;

WriteLine(number.ToString());

布尔值

布尔值 = true

;

];

."

{encoded}

```

// allocate array of 128 bytes

```cs

I was born 27 years ago.

```cs

1.  Write($"

byte

26 EE

The opposite of `ToString` is `Parse` . Only a few types have a `Parse` method, including all the number types and `DateTime` .

);

Run the code and view the result, as shown in the following output:

int

## Binary Object as bytes:

(int

{birthday:D}

True

1.  index = 0

```

[] binaryObject = new

[128

My birthday is 04 July 1980.

```

(new

Parsing from strings to numbers or dates and times

WriteLine("Binary Object as bytes:"

**Good Practice** : Use the standard date and time format specifiers, as shown at the following link: [`docs.microsoft.com/en-us/dotnet/standard/base-types/standard-date-and-time-format-strings#table-of-format-specifiers`](https://docs.microsoft.com/en-us/dotnet/standard/base-types/standard-date-and-time-format-strings#table-of-format-specifiers)

Errors using Parse

{birthday}

When you have a binary object like an image or video that you want to either store or transmit, you sometimes do not want to send the raw bits because you do not know how those bits could be misinterpreted, for example, by the network protocol transmitting them or another operating system that is reading the store binary object.

{age}

years ago."

encoded = ToBase64String(binaryObject);

"

byte

Binary Object as Base64: s01V3i0Ou8++TeZTw8KbZwNF +eUgYX5PeoHsSfBJHY7U99tU r6CBBbi+zvg2kHrUNkIEdYEbq1HOBWOsInLedC9Xf8vnR7diw/QtYZOFGOoGFxKuRKgNuEyJha k81eJG4FnJ3xCv7e+KobGN7kq+SO x5pQpfLzCHSsd/XcENJu4=

10 AF ED EF 8AA1 B1 8D EE 4A BE 48 EC 79 A5 A 5F 2F 30 87 4A C7 7F 5D C1 D

02/28/2021 17:33:54

61 93 85 18 EA 6 17 12 AE 44 A8 D B8 4C 89 85 A9 3C D5 E2 46 E0 59 C9 DF

WriteLine();

WriteLine($"My birthday is

();

```cs

System.Object

);

);

DateTime now = DateTime.Now;

Run the code and view the result, as shown in the following output:

);

age = int

1.  WriteLine(boolean.ToString());

```cs

Type statements to parse an integer and a date and time value from strings and then write the result to the console, as shown in the following code:

WriteLine($"I was born

"

One problem with the `Parse` method is that it gives errors if the `string` cannot be converted.

By default, a date and time value outputs with the short date and time format. You can use format codes such as `D` to output only the date part using the long date format.

```

);

DateTime birthday = DateTime.Parse("4 July 1980"

## object

The safest thing to do is to convert the binary object into a `string` of safe characters. Programmers call this **Base64** encoding.

Converting from a binary object to a string

The `Convert` type has a pair of methods, `ToBase64String` and `FromBase64String` , that perform this conversion for you. Let's see them in action:

1.  ```cs

.Parse("27"

WriteLine(now.ToString());

{binaryObject[index]:X}

4 75 81 1B AB 51 CE 5 63 AC 22 72 DE 74 2F 57 7F CB E7 47 B7 62 C3 F4 2D

string

The second most common conversion is from strings to numbers or date and time values.

// convert to Base64 string and output as text

for

}

WriteLine($"Binary Object as Base64:

By default, an `int` value would output assuming decimal notation, that is, base10\. You can use format codes such as `:X` to format the value using hexadecimal notation.

."

Type statements to create an array of bytes randomly populated with byte values, write each byte nicely formatted to the console, and then write the same bytes converted to Base64 to the console, as shown in the following code:

);

// populate array with random bytes

```

Let's see `Parse` in action:

Run the code and view the result, as shown in the following output:

WriteLine($"My birthday is

; index < binaryObject.Length; index++)

1.  ```

);

me = new

WriteLine(me.ToString());

Random()).NextBytes(binaryObject);

);

My birthday is 04/07/1980 00:00:00\.

B3 4D 55 DE 2D E BB CF BE 4D E6 53 C3 C2 9B 67 3 45 F9 E5 20 61 7E 4F 7A 81 EC 49 F0 49 1D 8E D4 F7 DB 54 AF A0 81 5 B8 BE CE F8 36 90 7A D4 36 42

### 12

{

1.  Type a statement to attempt to parse a string containing letters into an integer variable, as shown in the following code:

```cs

int

count = int

.Parse("abc"

);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

未处理的异常：System.FormatException：输入字符串的格式不正确。

```

除了前面的异常消息，您还将看到堆栈跟踪。我没有在本书中包含堆栈跟踪，因为它们占用太多空间。

### 使用 TryParse 方法避免异常

为了避免错误，您可以使用`TryParse`方法。`TryParse`尝试转换输入的`string`，如果可以转换，则返回`true`，如果无法转换，则返回`false`。

`out`关键字是必需的，以允许`TryParse`方法在转换成功时设置计数变量。

让我们看看`TryParse`的运行情况：

1.  将`int` `count`声明替换为使用`TryParse`方法的语句，并要求用户输入鸡蛋数量的计数，如下面的代码所示：

```cs

Write("有多少鸡蛋？"

);

string

? input = ReadLine(); // or use "12" in notebook

if

(int

.TryParse(input, out

int

count))

{

WriteLine($"There are

{count}

鸡蛋。"

);

}

else

{

WriteLine("I could not parse the input."

);

}

```

1.  运行代码，输入`12`，并查看结果，如下面的输出所示：

```cs

有多少鸡蛋？12

有 12 个鸡蛋。

```

1.  运行代码，输入`twelve`（或在笔记本中将`string`值更改为`"twelve"`），并查看结果，如下面的输出所示：

```cs

有多少鸡蛋？twelve

我无法解析输入。

```

您还可以使用`System.Convert`类型的方法将`string`值转换为其他类型；但是，就像`Parse`方法一样，如果无法转换，它会报错。

# 处理异常

您已经看到了几种在转换类型时发生错误的情况。有些语言在出现问题时返回错误代码。.NET 使用异常，它们比具有多种用途的返回值更丰富，并且仅设计用于报告失败。当这种情况发生时，我们说*发生了运行时异常*。

当抛出异常时，线程被挂起，如果调用代码定义了`try-catch`语句，则有机会处理异常。如果当前方法没有处理它，那么它的调用方法将有机会处理，依此类推。

正如您所见，控制台应用程序或.NET 交互式笔记本的默认行为是输出有关异常的消息，包括堆栈跟踪，然后停止运行代码。应用程序被终止。这比允许代码继续在可能处于损坏状态的情况下执行要好。您的代码应该只捕获和处理它理解并且可以正确修复的异常。

**良好实践**：尽量避免编写可能引发异常的代码，也许可以通过执行`if`语句检查来实现。有时候你不能，有时候最好让异常被调用你的代码的更高级组件捕获。您将在*第四章*，*编写、调试和测试函数*中学习如何做到这一点。

## 将易出错的代码包装在 try 块中

当您知道某个语句可能会导致错误时，应该将该语句包装在`try`块中。例如，从文本解析为数字可能会导致错误。`catch`块中的任何语句只有在`try`块中的语句抛出异常时才会执行。

我们不必在`catch`块中做任何事情。让我们看看它的运行情况：

1.  使用您喜欢的编码工具将新的**控制台应用程序**添加到名为`HandlingExceptions`的`Chapter03`工作区/解决方案中。

1.  在 Visual Studio Code 中，将`HandlingExceptions`选择为活动的 OmniSharp 项目。

1.  输入语句提示用户输入他们的年龄，然后将他们的年龄写入控制台，如下面的代码所示：

```cs

WriteLine("Before parsing"

);

Write("你多大了？ "

);

string

? input = ReadLine(); // or use "49" in a notebook

尝试

{

int

年龄 = int

.Parse(input);

WriteLine($"您今年

{年龄}

岁了。"

);

}

catch

{

}

WriteLine("解析之后"

);

```

您将看到以下编译器消息：`警告 CS8604 可能的空引用参数's'在'int int.Parse(string s)'中。`在新的.NET 6 项目中，默认情况下，Microsoft 已启用了可空引用类型，因此您将看到更多类似的编译器警告。在生产代码中，您应该添加代码来检查 `null` 并适当处理可能性。在本书中，我不会包括这些 `null` 检查，因为代码示例并非设计为生产质量，而且到处都是 `null` 检查会使代码混乱，并占用宝贵的页面。在这种情况下，`input` 不可能为 `null`，因为用户必须按 Enter 键才能使 `ReadLine` 返回，并且这将返回一个空的 `string`。在本书的代码示例中，您将看到数百个潜在的 `null` 变量示例。这些警告在书中的代码示例中是可以忽略的。只有在编写自己的生产代码时才需要类似的警告。在 *第六章*，*实现接口和继承类* 中，您将了解更多关于空值处理的内容。

这段代码包括两条消息，指示*解析之前*和*解析之后*，以便更清楚地了解代码的流程。随着示例代码变得更加复杂，这将特别有用。

1.  运行代码，输入 `49`，并查看结果，如下面的输出所示：

```cs

解析之前

你多大了？ 49

你今年 49 岁。

解析之后

```

1.  运行代码，输入 `Kermit`，并查看结果，如下面的输出所示：

```cs

解析之前

你多大了？ Kermit

解析之后

```

当代码被执行时，错误异常被捕获，但默认消息和堆栈跟踪没有输出，控制台应用程序继续运行。这比默认行为要好，但可能有必要看到发生的错误类型。

**良好实践**：在生产代码中，您不应该使用空的 `catch` 语句，因为它会“吞没”异常并隐藏潜在的问题。如果您无法或不想正确处理异常，至少应该记录异常，或者重新抛出异常，以便更高级别的代码可以决定。您将在 *第四章*，*编写、调试和测试函数* 中了解有关日志记录的内容。

### 捕获所有异常

要获取可能发生的任何类型的异常信息，可以在 `catch` 块中声明一个类型为 `System.Exception` 的变量：

1.  在 `catch` 块中添加异常变量声明，并使用它将异常信息写入控制台，如下面的代码所示：

```cs

catch (Exception ex)

{

WriteLine($"

{ex.GetType()}

说

{ex.Message}

"

);

}

```

1.  运行代码，再次输入 `Kermit`，并查看结果，如下面的输出所示：

```cs

解析之前

你多大了？ Kermit

System.FormatException 表示输入字符串格式不正确。

解析之后

```

### 捕获特定异常

现在我们知道发生了哪种特定类型的异常，我们可以通过仅捕获该类型的异常并自定义向用户显示的消息来改进我们的代码：

1.  保留现有的 `catch` 块，并在其上方添加一个新的格式异常类型的 `catch` 块，如下面突出显示的代码所示：

```cs

**catch (FormatException)**

**{**

**WriteLine(**

**"您输入的年龄不是有效的数字格式。"**

**);**

**}**

catch (Exception ex)

{

WriteLine($"

{ex.GetType()}

说

{ex.Message}

"

);

}

```

1.  运行代码，再次输入 `Kermit`，并查看结果，如下面的输出所示：

```cs

解析之前

你多大了？ Kermit

您输入的年龄不是有效的数字格式。

解析之后

```

我们希望保留下面更一般的 catch 的原因是可能会发生其他类型的异常。

1.  运行代码，输入`9876543210`，并查看结果，如下面的输出所示：

```cs

解析之前

你多大了？9876543210

System.OverflowException 说 Value was either too large or too small for an Int32.

解析之后

```

让我们为这种类型的异常添加另一个`catch`块。

1.  保留现有的`catch`块，并为溢出异常类型添加一个新的`catch`块，如下面突出显示的代码所示：

```cs

**catch（OverflowException）**

**{**

**WriteLine（**

**“您的年龄是有效的数字格式，但要么太大要么太小。”**

**);**

**}**

catch（FormatException）

{

WriteLine("您输入的年龄不是有效的数字格式。"

）;

}

```

1.  运行代码，输入`9876543210`，并查看结果，如下面的输出所示：

```cs

解析之前

你多大了？9876543210

您的年龄是有效的数字格式，但要么太大要么太小。

解析之后

```

捕获异常的顺序很重要。正确的顺序与异常类型的继承层次结构有关。您将在*第五章*，*使用面向对象编程构建自己的类型*中了解继承。但是，不要太担心这一点——如果您以错误的顺序获得异常，编译器将给您构建错误。

**良好实践**：避免过度捕获异常。通常应允许它们传播到调用堆栈的更高级别，以便在更多信息已知的级别处理它们的逻辑可能发生变化。您将在*第四章*，*编写、调试和测试函数*中了解这一点。

### 使用过滤器捕获

您还可以使用`when`关键字向 catch 语句添加过滤器，如下面的代码所示：

```cs

Write("输入金额："

）;

字符串

？金额= ReadLine（）;

尝试

{

十进制

amountValue = decimal

.Parse（amount）;

}

catch（FormatException）when

（amount.Contains（“$”

））

{

WriteLine("金额不能使用美元符号！"

）;

}

catch（FormatException）

{

WriteLine("金额只能包含数字！"

）;

}

```

# 检查溢出

之前，我们看到在数字类型之间进行转换时，可能会丢失信息，例如，从`long`变量转换为`int`变量时。如果类型中存储的值太大，它将溢出。

## 使用 checked 语句抛出溢出异常

`checked`语句告诉.NET 在发生溢出时抛出异常，而不是默认情况下允许它悄悄发生，这是出于性能原因而做的。

我们将把`int`变量的初始值设置为其最大值减去 1。然后，我们将递增它多次，每次输出其值。一旦它超过其最大值，它就会溢出到其最小值，并继续从那里递增。让我们看看这个过程：

1.  使用您喜欢的编码工具向`Chapter03`工作区/解决方案添加一个新的**控制台应用程序**，名称为`CheckingForOverflow`。

1.  在 Visual Studio Code 中，将`CheckingForOverflow`选择为活动的 OmniSharp 项目。

1.  在`Program.cs`中，键入语句以声明并将整数分配给其最大可能值的一个小于的值，然后递增它并将其值写入控制台三次，如下面的代码所示：

```cs

int

x = int

.MaxValue - 1

;

WriteLine（$“初始值：

{x}

"

）;

x++;

WriteLine（$“增量后：

{x}

"

）;

x++;

WriteLine（$“增量后：

{x}

"

）;

x++;

WriteLine（$“增量后：

{x}

"

）;

```

1.  运行代码并查看结果，显示值溢出并悄悄包装到大的负值，如下面的输出所示：

```cs

初始值：2147483646

增量后：2147483647

增量后：-2147483648

增量后：-2147483647

```

1.  现在，让我们通过使用`checked`语句块来包装语句，让编译器警告我们溢出，如下面的代码中所示：

```cs

**checked**

**{**

int

x = int

.MaxValue - 1

;

WriteLine($"初始值：

{x}

"

);

x++;

WriteLine($"递增后：

{x}

"

);

x++;

WriteLine($"递增后：

{x}

"

);

x++;

WriteLine($"递增后：

{x}

"

);

**}**

```

1.  运行代码并查看结果，显示检查溢出并导致抛出异常，如下面的输出所示：

```cs

初始值：2147483646

递增后：2147483647

未处理的异常：System.OverflowException：算术运算导致溢出。

```

1.  就像任何其他异常一样，我们应该将这些语句包装在`try`语句块中，并为用户显示一个更好的错误消息，如下面的代码所示：

```cs

try

{

// 前面的代码放在这里

}

catch (OverflowException)

{

WriteLine("代码溢出，但我捕获了异常。"

);

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

初始值：2147483646

递增后：2147483647

代码溢出，但我捕获了异常。

```

## 使用未经检查的语句禁用编译器溢出检查

前一节是关于*运行时*的默认溢出行为以及如何使用`checked`语句来更改该行为。本节是关于*编译时*溢出行为以及如何使用`unchecked`语句来更改该行为。

一个相关的关键字是`unchecked`。这个关键字关闭编译器在一段代码块中执行的溢出检查。让我们看看如何做到这一点：

1.  在前面的语句末尾键入以下语句。编译器不会编译此语句，因为它知道会溢出：

```cs

int

y = int

.MaxValue + 1

;

```

1.  将鼠标指针悬停在错误上，并注意显示编译时检查的错误消息，如*图 3.1*所示：![图形用户界面，文本，应用程序，电子邮件自动生成的描述](img/Image00039.jpg)

图 3.1：在 PROBLEMS 窗口中的编译时检查

1.  要禁用编译时检查，将语句包装在`unchecked`块中，将`y`的值写入控制台，递减它，并重复，如下面的代码所示：

```cs

未经检查的

{

int

y = int

.MaxValue + 1

;

WriteLine($"初始值：

{y}

"

);

y--;

WriteLine($"递减后：

{y}

"

);

y--;

WriteLine($"递减后：

{y}

"

);

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

初始值：-2147483648

递减后：2147483647

递减后：2147483646

```

当然，你很少会想要明确关闭这样的检查，因为它允许溢出发生。但也许你可以想到一个情景，你可能希望出现这种行为。

# 练习和探索

通过回答一些问题来测试你的知识和理解，进行一些实践，并深入研究本章主题。

## 练习 3.1 - 测试你的知识

回答以下问题：

1.  当你将`int`变量除以`0`时会发生什么？

1.  当你将`double`变量除以`0`时会发生什么？

1.  当你溢出一个`int`变量时会发生什么？

1.  `x = y++;`和`x = ++y;`之间有什么区别？

1.  在循环语句中使用`break`，`continue`和`return`时有什么区别？

1.  `for`语句的三个部分是什么，它们中的哪些是必需的？

1.  `=`和`==`运算符之间有什么区别？

1.  以下语句是否编译？

```cs

for

( ; true

; ) ;

```

1.  `switch`表达式中的下划线`_`代表什么？

1.  对象必须实现哪个接口才能通过`foreach`语句进行枚举？

## 练习 3.2 - 探索循环和溢出

如果这段代码执行会发生什么？

```cs

int

max = 500

;

for

(byte

i = 0

; i < max; i++)

{

WriteLine(i);

}

```

在`Chapter03`中创建一个名为`Exercise02`的控制台应用程序，并输入上述代码。运行控制台应用程序并查看输出。会发生什么？

您可以添加什么代码（不要更改任何先前的代码）来警告我们有关问题？

## 练习 3.3–练习循环和运算符

**FizzBuzz**是一个儿童的团体词游戏，教他们关于除法。玩家轮流递增计数，用单词*fizz*替换任何可以被三整除的数字，用单词*buzz*替换任何可以被五整除的数字，用*fizzbuzz*替换任何可以同时被三和五整除的数字。

在`Chapter03`中创建一个名为`Exercise03`的控制台应用程序，输出一个模拟的 FizzBuzz 游戏，计数到 100。输出应该类似于*图 3.2*：

![自动生成的文本描述](img/Image00040.jpg)

图 3.2：模拟的 FizzBuzz 游戏输出

## 练习 3.4–练习异常处理

在`Chapter03`中创建一个名为`Exercise04`的控制台应用程序，询问用户两个范围在 0-255 之间的数字，然后将第一个数字除以第二个数字：

```cs

输入一个介于 0 和 255 之间的数字：100

输入另一个介于 0 和 255 之间的数字：8

100 除以 8 等于 12

```

编写异常处理程序以捕获任何抛出的错误，如下面的输出所示：

```cs

输入一个介于 0 和 255 之间的数字：苹果

输入另一个介于 0 和 255 之间的数字：香蕉

格式异常：输入字符串格式不正确。

```

## 练习 3.5–测试您对运算符的了解

在以下语句执行后，`x`和`y`的值是多少？

1.  递增和加法运算符：

```cs

x = 3

;

y = 2

+ ++x;

```

1.  二进制移位运算符：

```cs

x = 3

<< 2

;

y = 10

>> 1

;

```

1.  位运算符：

```cs

x = 10

& 8

;

y = 10

| 7

;

```

## 练习 3.6–探索主题

使用以下页面上的链接详细了解本章涵盖的主题：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-3---controlling-flow-and-converting-types`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-3---controlling-flow-and-converting-types)

# 摘要

在本章中，您尝试了一些运算符，学习了如何分支和循环，如何在类型之间转换，以及如何捕获异常。

您现在已经准备好学习如何通过定义函数重用代码块，如何将值传递给它们并获取返回值，以及如何跟踪代码中的错误并消除它们！
