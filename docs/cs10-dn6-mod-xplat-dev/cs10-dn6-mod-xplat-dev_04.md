# 第四章：编写、调试和测试函数

本章是关于编写函数以重用代码，开发期间调试逻辑错误，运行时记录异常，对代码进行单元测试以消除错误，并确保稳定性和可靠性。

本章涵盖以下主题：

+   编写函数

+   开发期间调试

+   运行时日志记录

+   单元测试

+   在函数中抛出和捕获异常

# 编写函数

编程的一个基本原则是**不要重复自己**（**DRY**）。

在编程时，如果发现自己一遍又一遍地编写相同的语句，那么将这些语句转换为函数。函数就像完成一个小任务的小程序。例如，您可以编写一个计算销售税的函数，然后在财务应用程序的许多地方重用该函数。

与程序一样，函数通常具有输入和输出。有时被描述为黑匣子，您在一端输入一些原材料，而在另一端出现了制造出的物品。一旦创建，您就不需要考虑它们是如何工作的。

## 乘法表示例

假设您想帮助孩子学习乘法表，因此希望轻松生成一个数字的乘法表，例如 12 乘法表：

```cs

1

x 12

= 12

2

x 12

= 24

...

12

x 12

= 144

```

您之前在本书的早些时候学习了`for`语句，因此您知道它可以用于生成重复的输出行，例如 12 乘法表，如下面的代码所示：

```cs

为

（int

行= 1

; 行 <= 12

; 行++）

{

Console.WriteLine（$“

{行}

x 12 =

{行 *

12

}

"

）;

}

```

但是，我们不想输出 12 乘法表，我们希望使其更加灵活，因此可以输出任何数字的乘法表。我们可以通过创建一个函数来实现这一点。

### 编写乘法表函数

通过创建一个函数来探索函数，以输出 1 到 12 的任何数字的乘法表：

1.  使用您喜欢的编码工具创建一个新的控制台应用程序，如下列表中定义的那样：

1.  项目模板：**控制台应用程序** / `console`

1.  工作区/解决方案文件和文件夹：`Chapter04`

1.  项目文件和文件夹：`WritingFunctions`

1.  静态导入`System.Console`。

1.  在`Program.cs`中，编写语句来定义一个名为`TimesTable`的函数，如下面的代码所示：

```cs

静态

空

TimesTable

（

字节

数字

）

{

WriteLine（$“这是

{数字}

乘法表："

）;

为

（int

行= 1

; 行 <= 12

; 行++)

{

WriteLine（$“

{行}

x

{数字}

=

{行 * 数字}

"

）;

}

写入行（）;

}

```

在上述代码中，请注意以下内容：

+   `TimesTable`必须有一个名为`number`的参数传递给它的`byte`值。

+   `TimesTable`是一个`static`方法，因为它将被`static`方法`Main`调用。

+   `TimesTable`不会向调用者返回值，因此在其名称之前使用`void`关键字声明。

+   `TimesTable`使用`for`语句输出传递给它的数字的乘法表。

1.  在静态导入`Console`类的语句之后，在`TimesTable`函数之前，调用该函数并传递一个`byte`值作为`number`参数，例如`6`，如下面的代码中所示：

```cs

使用

静态

System.Console;

**TimesTable（**

**6**

**);**

```

**良好实践**：如果一个函数有一个或多个参数，仅传递值可能不提供足够的含义，那么您可以选择指定参数的名称以及其值，如下面的代码所示：`TimesTable(number: 6)`。

1.  运行代码，然后查看结果，如下面的输出所示：

```cs

这是 6 乘法表：”

1 x 6 = 6

2 x 6 = 12

3 x 6 = 18

4 x 6 = 24

5 x 6 = 30

6 x 6 = 36

7 x 6 = 42

8 x 6 = 48

9 x 6 = 54

10 x 6 = 60

11 x 6 = 66

12 x 6 = 72

```

1.  将传递给`TimesTable`函数的数字更改为`0`到`255`之间的其他`byte`值，并确认输出的乘法表是正确的。

1.  请注意，如果您尝试传递非`byte`数字，例如`int`或`double`或`string`，将返回错误，如下面的输出所示：

```cs

错误：（1,12）：错误 CS1503：参数 1：无法从'int'转换为'byte'

```

## 编写一个返回值的函数

之前的函数执行了一些操作（循环和写入控制台），但没有返回值。假设您需要计算销售税或增值税（VAT）。在欧洲，增值税率可以从瑞士的 8%到匈牙利的 27%不等。在美国，州销售税的范围可以从俄勒冈州的 0%到加利福尼亚州的 8.25%。

税率随时变化，根据许多因素而变化。您无需与我联系告诉我弗吉尼亚的税率是 6%。谢谢。

让我们实现一个在世界各地计算税收的函数：

1.  添加一个名为`CalculateTax`的函数，如下面的代码所示：

```cs

static

十进制

CalculateTax

（

十进制

amount，

string

twoLetterRegionCode

）

{

十进制

rate = 0.0

M;

switch

（twoLetterRegionCode）

{

case

"CH"

: // 瑞士

rate = 0.08

M;

break

;

case

"DK"

: // 丹麦

case

"NO"

: // 挪威

rate = 0.25

M;

break

;

case

"GB"

: // 英国

case

"FR"

: // 法国

rate = 0.2

M;

break

;

case

"HU"

: // 匈牙利

rate = 0.27

M;

break

;

case

"OR"

: // 俄勒冈州

case

"AK"

: // 阿拉斯加

case

"MT"

: // 蒙大拿州

rate = 0.0

M;

break

;

case

"ND"

: // 北达科他州

case

"WI"

: // 威斯康星

case

"ME"

: // 缅因州

case

"VA"

: // 弗吉尼亚

rate = 0.05

M;

break

;

case

"CA"

: // 加利福尼亚

rate = 0.0825

M;

break

;

default

: // 大多数美国州

rate = 0.06

M;

break

;

}

return

amount * rate;

}

```

在上述代码中，请注意以下内容：

+   `CalculateTax`有两个输入：一个名为`amount`的参数，它将是花费的金额，以及一个名为`twoLetterRegionCode`的参数，它将是花费金额的地区。

+   `CalculateTax`将使用`switch`语句执行计算，然后返回金额上所欠销售税或增值税的`decimal`值；因此，在函数名称之前，我们已经声明了返回值的数据类型为`decimal`。

1.  注释掉`TimesTable`方法调用，并调用`CalculateTax`方法，传递金额的值，例如`149`和有效的区域代码，例如`FR`，如下面的代码所示：

```cs

// TimesTable(6);

十进制

taxToPay = CalculateTax(amount: 149

，twoLetterRegionCode: "FR"

);

WriteLine（$“您必须支付

{taxToPay}

in tax."

);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

您必须支付 29.8 的税款。

```

我们可以通过使用`{taxToPay:C}`将`taxToPay`输出格式化为货币，但它将使用您的本地文化来决定如何格式化货币符号和小数位。例如，对于我来说，在英国，我会看到`£29.80`。

您能想到`CalculateTax`函数的任何问题吗？如果用户输入诸如`fr`或`UK`之类的代码会发生什么？您如何重写函数以改进它？使用`switch` *表达式*而不是`switch` *语句*是否更清晰？

## 从基数转换为序数的数字

用于计数的数字称为**基数**数字，例如 1、2 和 3，而用于排序的数字是**序数**数字，例如 1st、2nd 和 3rd。让我们创建一个将基数转换为序数的函数：

1.  编写一个名为`CardinalToOrdinal`的函数，将基数`int`值转换为序数`string`值；例如，它将 1 转换为 1st，2 转换为 2nd，如下面的代码所示：

```cs

static

string

CardinalToOrdinal

(

int

number

)

{

switch

（数字）

{

case

11

: // 11th 到 13th 的特殊情况

case

12

:

case

13

：

return

$"

{number}

th"

;

default

：

int

lastDigit = number % 10

;

string

suffix = lastDigit switch

{

1

=> "st"

，

2

=> "nd"

，

3

=> "rd"

,

_ => "th"

};

return

$"

{number}{suffix}

"

;

}

}

```

从上述代码中，请注意以下内容：

+   `CardinalToOrdinal` 有一个输入：一个名为 `number` 的 `int` 类型参数，一个输出：一个 `string` 类型的返回值。

+   使用 `switch` *语句* 处理 11、12 和 13 的特殊情况。

+   然后，`switch` *表达式* 处理所有其他情况：如果最后一位是 1，则使用 `st` 作为后缀；如果最后一位是 2，则使用 `nd` 作为后缀；如果最后一位是 3，则使用 `rd` 作为后缀；如果最后一位是其他任何数字，则使用 `th` 作为后缀。

1.  编写一个名为 `RunCardinalToOrdinal` 的函数，该函数使用 `for` 语句循环从 1 到 40，为每个数字调用 `CardinalToOrdinal` 函数，并将返回的 `string` 写入控制台，用空格字符分隔，如下面的代码所示：

```cs

静态

void

RunCardinalToOrdinal

()

{

for

(int

number = 1

; number <= 40

; number++)

{

Write($"

{CardinalToOrdinal(number)}

"

);

}

WriteLine();

}

```

1.  注释掉 `CalculateTax` 语句，并调用 `RunCardinalToOrdinal` 方法，如下面的代码所示：

```cs

// TimesTable(6);

// decimal taxToPay = CalculateTax(amount: 149, twoLetterRegionCode: "FR");

// WriteLine($"You must pay {taxToPay} in tax.");

RunCardinalToOrdinal();

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

1st 2nd 3rd 4th 5th 6th 7th 8th 9th 10th 11th 12th 13th 14th 15th 16th 17th 18th 19th 20th 21st 22nd 23rd 24th 25th 26th 27th 28th 29th 30th 31st 32nd 33rd 34th 35th 36th 37th 38th 39th 40th

```

## 使用递归计算阶乘

5 的阶乘是 120，因为阶乘是通过将起始数字乘以比它小 1 的数字，然后再乘以比它小的数字，依此类推，直到数字减少到 1 来计算的。一个例子可以在这里看到：5 x 4 x 3 x 2 x 1 = 120。

阶乘写成这样：5!，其中感叹号读作 bang，所以 5! = 120，也就是说，*五 bang 等于一百二十*。Bang 是阶乘的一个好名字，因为它们的大小增长非常迅速，就像爆炸一样。

我们将编写一个名为 `Factorial` 的函数；这将为传递给它的 `int` 计算阶乘。我们将使用一种称为**递归**的巧妙技术，这意味着一个函数在其实现中直接或间接地调用自身：

1.  添加一个名为 `Factorial` 的函数，并调用它，如下面的代码所示：

```cs

静态

int

阶乘

(

int

number

)

{

if

(number < 1

)

{

返回

0

;

}

else

if

(number == 1

)

{

返回

1

;

}

else

{

返回

number * Factorial(number - 1

);

}

}

```

与以前一样，前面的代码中有几个值得注意的元素，包括以下内容：

+   如果输入参数 `number` 是零或负数，`Factorial` 返回 `0`。

+   如果输入参数 `number` 是 `1`，`Factorial` 返回 `1`，因此停止调用自身。

+   如果输入参数 `number` 大于 1，那么在所有其他情况下，`Factorial` 将数字乘以调用自身并传递比 `number` 小 1 的结果。这使得函数递归。

**更多信息**：递归很聪明，但它可能会导致问题，比如由于太多函数调用而导致堆栈溢出，因为内存用于在每次函数调用时存储数据，最终使用过多。在诸如 C#之类的语言中，迭代是一个更实际但不够简洁的解决方案。您可以在以下链接中阅读更多信息：[`en.wikipedia.org/wiki/Recursion_(computer_science)#Recursion_versus_iteration`](https://en.wikipedia.org/wiki/Recursion_(computer_science)#Recursion_versus_iteration)。

1.  添加一个名为 `RunFactorial` 的函数，该函数使用 `for` 语句输出从 1 到 14 的数字的阶乘，调用其循环中的 `Factorial` 函数，然后输出结果，使用代码 `N0` 进行格式化，这意味着数字格式使用千位分隔符，小数位数为零，如下面的代码所示：

```cs

静态

void

RunFactorial

()

{

for

(int

i = 1

; i < 15

; i++)

{

WriteLine($"

{i}

! =

{Factorial(i):N0}

"

);

}

}

```

1.  Comment out the `RunCardinalToOrdinal` method call and call the `RunFactorial` method.

1.  Run the code and view the results, as shown in the following output:

```cs

1! = 1

2! = 2

3! = 6

4! = 24

5! = 120

6! = 720

7! = 5,040

8! = 40,320

9! = 362,880

10! = 3,628,800

11! = 39,916,800

12! = 479,001,600

13! = 1,932,053,504

14! = 1,278,945,280

```

It is not immediately obvious in the previous output, but factorials of 13 and higher overflow the `int` type because they are so big. 12! is 479,001,600, which is about half a billion. The maximum positive value that can be stored in an `int` variable is about two billion. 13! is 6,227,020,800, which is about six billion and when stored in a 32-bit integer it overflows silently without showing any problems.

Do you remember what we can do to be notified of a numeric overflow?

What should you do to get notified when an overflow happens? Of course, we could solve the problem for 13! and 14! by using a `long` (64-bit integer) instead of an `int` (32-bit integer), but we will quickly hit the overflow limit again.

The point of this section is to understand that numbers can overflow and how to show that rather than ignore it, not specifically how to calculate factorials higher than 12!.

1.  Modify the `Factorial` function to check for overflows, as shown highlighted in the following code:

```cs

**checked**

**// for overflow**

**{**

return

number * Factorial(number - 1

);

**}**

```

1.  Modify the `RunFactorial` function to handle overflow exceptions when calling the `Factorial` function, as shown highlighted in the following code:

```cs

**try**

**{**

WriteLine($"

{i}

! =

{Factorial(i):N0}

"

);

**}**

**catch (System.OverflowException)**

**{**

**WriteLine(**

**$"**

**{i}**

**! is too big for a 32-bit integer."**

**);**

**}**

```

1.  Run the code and view the results, as shown in the following output:

```cs

1! = 1

2! = 2

3! = 6

4! = 24

5! = 120

6! = 720

7! = 5,040

8! = 40,320

9! = 362,880

10! = 3,628,800

11! = 39,916,800

12! = 479,001,600

13! is too big for a 32-bit integer.

14! is too big for a 32-bit integer.

```

## Documenting functions with XML comments

By default, when calling a function such as `CardinalToOrdinal` , code editors will show a tooltip with basic information, as shown in *Figure 4.1* :

![Graphical user interface, text, application Description automatically generated](img/Image00041.jpg)

Figure 4.1: A tooltip showing the default simple method signature

Let's improve the tooltip by adding extra information:

1.  If you are using Visual Studio Code with the **C#** extension, you should navigate to **View** | **Command Palette** | **Preferences: Open Settings (UI)** , and then search for `formatOnType` and make sure that is enabled. C# XML documentation comments are a built-in feature of Visual Studio 2022.

1.  On the line above the `CardinalToOrdinal` function, type three forward slashes `///` , and note that they are expanded into an XML comment that recognizes that the function has a single parameter named `number` .

1.  Enter suitable information for the XML documentation comment for a summary and to describe the input parameter and the return value for the `CardinalToOrdinal` function, as shown in the following code:

```cs

///

<summary>

///

Pass a 32-bit integer and it will be converted into its ordinal equivalent.

///

</summary>

///

<param name="number">

Number is a cardinal value e.g. 1, 2, 3, and so on.

</param>

///

<returns>

Number as an ordinal value e.g. 1st, 2nd, 3rd, and so on.

</returns>

```

1.  Now, when calling the function, you will see more details, as shown in *Figure 4.2* :![Graphical user interface, text, application, chat or text message, email Description automatically generated](img/Image00042.jpg)

Figure 4.2: A tooltip showing the more detailed method signature

At the time of writing the sixth edition, C# XML documentation comments do not work in .NET Interactive notebooks.

**良好实践**：为所有函数添加 XML 文档注释。

## 在函数实现中使用 lambda

F#是微软的强类型函数优先编程语言，与 C#一样，编译为 IL 以供.NET 执行。函数语言起源于λ演算；一个仅基于函数的计算系统。代码看起来更像数学函数，而不是食谱中的步骤。

函数语言的一些重要属性在以下列表中定义：

+   **模块化**：在 C#中定义函数的同样好处适用于函数语言。将大型复杂的代码库分解为较小的部分。

+   **不变性**：在 C#中，变量不存在。函数内部的任何数据值都不能更改。相反，可以从现有数据值创建新的数据值。这减少了错误。

+   **可维护性**：代码更清晰（适用于数学倾向的程序员！）。

自 C# 6 以来，微软一直致力于为语言添加功能，以支持更加函数式的方法。例如，在 C# 7 中添加了**元组**和**模式匹配**，在 C# 8 中添加了**非空引用类型**，并改进了模式匹配并添加了记录，即 C# 9 中的**不可变对象**。

在 C# 6 中，微软添加了对**表达式主体函数成员**的支持。我们现在将看一个例子。

**斐波那契数列**总是以 0 和 1 开始。然后，使用将前两个数字相加的规则生成序列的其余部分，如下面的数字序列所示：

```cs

0

1

1

2

3

5

8

13

21

34

55

...

```

序列中的下一个项将是 34 + 55，即 89。

我们将使用斐波那契数列来说明命令式和声明式函数实现之间的区别：

1.  添加一个名为`FibImperative`的函数，该函数将以命令式风格编写，如下面的代码所示：

```cs

静态

int

FibImperative

（

int

术语

）

{

if

（term == 1

）

{

返回

0

;

}

否则

如果

（term == 2

）

{

返回

1

;

}

否则

{

返回

FibImperative（term-1

）+ FibImperative（term-2

）;

}

}

```

1.  添加一个名为`RunFibImperative`的函数，该函数在`for`语句中调用`FibImperative`，该语句循环从 1 到 30，如下面的代码所示：

```cs

静态

void

RunFibImperative

（）

{

对于

（int

i = 1

; i <= 30

; i ++）

{

WriteLine（“斐波那契数列的第{0}项是{1：N0}。”

，

arg0：CardinalToOrdinal（i），

arg1：FibImperative（term：i））;

}

}

```

1.  注释掉其他方法调用，并调用`RunFibImperative`方法。

1.  运行代码并查看结果，如下面的输出所示：

```cs

斐波那契数列的第 1 项是 0。

斐波那契数列的第 2 项是 1。

斐波那契数列的第 3 项是 1。

斐波那契数列的第 4 项是 2。

斐波那契数列的第 5 项是 3。

斐波那契数列的第 6 项是 5。

斐波那契数列的第 7 项是 8。

斐波那契数列的第 8 项是 13。

斐波那契数列的第 9 项是 21。

斐波那契数列的第 10 项是 34。

斐波那契数列的第 11 项是 55。

斐波那契数列的第 12 项是 89。

斐波那契数列的第 13 项是 144。

斐波那契数列的第 14 项是 233。

斐波那契数列的第 15 项是 377。

斐波那契数列的第 16 项是 610。

斐波那契数列的第 17 项是 987。

斐波那契数列的第 18 项是 1,597。

斐波那契数列的第 19 项是 2,584。

斐波那契数列的第 20 项是 4,181。

斐波那契数列的第 21 项是 6,765。

斐波那契数列的第 22 项是 10,946。

斐波那契数列的第 23 项是 17,711。

斐波那契数列的第 24 项是 28,657。

斐波那契数列的第 25 项是 46,368。

斐波那契数列的第 26 项是 75,025。

斐波那契数列的第 27 项是 121,393。

斐波那契数列的第 28 项是 196,418。

斐波那契数列的第 29 项是 317,811。

斐波那契数列的第 30 项是 514,229。

```

1.  添加一个名为`FibFunctional`的函数，以声明式风格编写，如下面的代码所示：

```cs

静态

int

FibFunctional

(

int

项

)

=>

术语切换

{

1

=> 0

，

2

=> 1

，

_ => FibFunctional(term - 1

）+ FibFunctional(term - 2

）

};

```

1.  添加一个函数来调用它，该函数在一个`for`语句中循环从 1 到 30，如下面的代码所示：

```cs

静态

空

RunFibFunctional

()

{

对于

（int

i = 1

; i <= 30

; i++)

{

WriteLine("斐波那契数列的第{0}项是{1:N0}。"

，

arg0：CardinalToOrdinal(i),

arg1：FibFunctional(term: i));

}

}

```

1.  注释掉`RunFibImperative`方法调用，并调用`RunFibFunctional`方法。

1.  运行代码并查看结果（与以前相同）。

# 开发期间调试

在本节中，您将学习如何在开发时调试问题。您必须使用具有调试工具的代码编辑器，例如 Visual Studio 或 Visual Studio Code。在撰写本文时，您无法使用.NET 交互式笔记本来调试代码，但预计将来会添加。

**更多信息**：有些人发现为 Visual Studio Code 设置 OmniSharp 调试器有些棘手。我已经包含了最常见问题的说明，但如果您仍然遇到问题，请尝试阅读以下链接中的信息：[`github.com/OmniSharp/omnisharp-vscode/blob/master/debugger.md`](https://github.com/OmniSharp/omnisharp-vscode/blob/master/debugger.md)

## 创建有意的错误的代码

让我们通过创建一个有意的错误的控制台应用程序来探索调试，然后使用您的代码编辑器中的调试工具来追踪并修复错误：

1.  使用您喜欢的编码工具向`Chapter04`工作区/解决方案添加一个新的**控制台应用程序**，命名为`Debugging`。

1.  在 Visual Studio Code 中，选择`Debugging`作为活动的 OmniSharp 项目。当看到弹出的警告消息说缺少所需的资产时，点击**是**以添加它们。

1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择。

1.  在`Program.cs`中，添加一个有意的错误的函数，如下面的代码所示：

```cs

静态

双

添加

(

双

a，

双

b

)

{

返回

a * b; // 有意的错误！

}

```

1.  在`Add`函数下面，编写语句来声明和设置一些变量，然后使用有错误的函数将它们相加，如下面的代码所示：

```cs

双

a = 4.5

;

双

b = 2.5

;

双

answer = Add(a, b);

WriteLine($"

{a}

+

{b}

=

{answer}

"

);

WriteLine("按 ENTER 键结束应用程序。"

);

ReadLine(); // 等待用户按 ENTER 键

```

1.  运行控制台应用程序并查看结果，如下面的部分输出所示：

```cs

4.5 + 2.5 = 11.25

```

但是，请等一下，有一个错误！4.5 加上 2.5 应该是 7，而不是 11.25！

我们将使用调试工具来寻找并消灭错误。

## 设置断点并开始调试

断点允许我们标记要暂停以检查程序状态并查找错误的代码行。

### 使用 Visual Studio 2022

让我们设置一个断点，然后使用 Visual Studio 2022 开始调试：

1.  点击声明名为`a`的变量的语句。

1.  转到**调试** | **切换断点**或按 F9。然后，左侧边栏上会出现一个红色圆圈，并且该语句将以红色突出显示，以指示已设置断点，如*图 4.3*所示：![](img/Image00043.jpg)

图 4.3：使用 Visual Studio 2022 切换断点

断点可以使用相同的操作切换关闭。您还可以在边栏中单击左键以切换断点的打开和关闭，或者右键单击断点以查看更多选项，例如删除、禁用或编辑现有断点的条件或操作。

1.  导航到**调试** | **开始调试**或按 F5。Visual Studio 启动控制台应用程序，然后在命中断点时暂停。这称为断点模式。额外的窗口标题为**Locals**（显示本地变量的当前值）、**Watch 1**（显示您定义的任何监视表达式）、**Call Stack**、**Exception Settings**和**Immediate Window**出现。**调试**工具栏出现。将要执行的下一行将以黄色高亮显示，并且黄色箭头指向边栏的该行，如*图 4.4*所示：![](img/Image00044.jpg)

图 4.4：Visual Studio 2022 中的断点模式

如果您不想看如何使用 Visual Studio Code 开始调试，可以跳过下一节，继续阅读标题为*使用调试工具栏导航*的部分。

### 使用 Visual Studio Code

让我们设置一个断点，然后使用 Visual Studio Code 开始调试：

1.  单击声明名为`a`的变量的语句。

1.  导航到**运行** | **切换断点**或按 F9。左侧边栏中会出现一个红色圆圈，表示已设置断点，如*图 4.5*所示：![](img/Image00045.jpg)

图 4.5：使用 Visual Studio Code 切换断点

可以使用相同的操作关闭断点。您还可以在边栏中左键单击以切换断点的开启和关闭，或者右键单击以查看更多选项，例如删除、编辑或禁用现有断点；或者在断点尚不存在时添加断点、条件断点或日志点。

日志点，也称为跟踪点，表示您想要记录一些信息，而无需在该点停止执行代码。

1.  导航到**视图** | **运行**，或者在左侧导航栏中单击**运行和调试**图标（三角形“播放”按钮和“bug”），如*图 4.5*所示。

1.  在**DEBUG**窗口顶部，单击**开始调试**按钮（绿色三角形“播放”按钮）右侧的下拉菜单，并选择**.NET Core Launch (console) (Debugging)**，如*图 4.6*所示：![](img/Image00046.jpg)

图 4.6：使用 Visual Studio Code 选择要调试的项目

**良好实践**：如果在**调试**项目的下拉列表中看不到选项，那是因为该项目没有进行调试所需的资源。这些资源存储在`.vscode`文件夹中。要为项目创建`.vscode`文件夹，请导航到**视图** | **命令面板**，选择**OmniSharp: 选择项目**，然后选择**调试**项目。几秒钟后，当提示**“从'Debugging'中缺少构建和调试所需的资源。是否添加？”**时，点击**是**以添加缺失的资源。

1.  在**DEBUG**窗口顶部，单击**开始调试**按钮（绿色三角形“播放”按钮），或导航到**运行** | **开始调试**，或按 F5。Visual Studio Code 启动控制台应用程序，然后在命中断点时暂停。这称为断点模式。将要执行的下一行将以黄色高亮显示，并且黄色块指向边栏的该行，如*图 4.7*所示：![](img/Image00047.jpg)

图 4.7：Visual Studio Code 中的断点模式

## 使用调试工具栏导航

Visual Studio Code 显示一个浮动工具栏，其中包含按钮，便于访问调试功能。Visual Studio 2022 在其**标准**工具栏中有一个按钮，用于启动或继续调试，以及一个单独的**调试**工具栏用于其他工具。

两者都显示在*图 4.8*中，并如下列表所述：

![图形用户界面描述自动生成，中等置信度](img/Image00048.jpg)

图 4.8：Visual Studio 2022 和 Visual Studio Code 中的调试工具栏

+   **继续**/F5：此按钮将从当前位置继续运行程序，直到结束或命中另一个断点。

+   **Step Over** /F10，**Step Into** /F11 和**Step Out** /Shift + F11（蓝色箭头覆盖点）：这些按钮以各种方式逐步执行代码语句，稍后您将看到。

+   **Restart** /Ctrl 或 Cmd + Shift + F5（圆形箭头）：此按钮将停止然后立即重新启动带有调试器的程序。

+   **Stop** /Shift + F5（红色方块）：此按钮将停止调试会话。

## 调试窗口

在调试时，Visual Studio Code 和 Visual Studio 都会显示额外的窗口，允许您监视有用的信息，例如变量，同时逐步执行代码。

以下是最有用的窗口：

+   **VARIABLES**，包括**Locals**，自动显示任何本地变量的名称、值和类型。在逐步执行代码时，密切关注此窗口。

+   **WATCH**或**Watch 1**，显示您手动输入的变量和表达式的值。

+   **CALL STACK**显示函数调用堆栈。

+   **BREAKPOINTS**显示所有断点并允许对其进行更精细的控制。

在断点模式下，编辑区域底部还有一个有用的窗口：

+   **DEBUG CONSOLE**或**Immediate Window**可以实现与代码的实时交互。您可以查询程序状态，例如输入变量的名称。例如，您可以通过输入`1+2`并按 Enter 来询问问题，如*图 4.9*所示：![](img/Image00049.jpg)

图 4.9：查询程序状态

## 逐步执行代码

让我们探索一些使用 Visual Studio 或 Visual Studio Code 逐步执行代码的方法：

1.  导航到**Run/Debug** | **Step Into**，或单击工具栏中的**Step Into**按钮，或按 F11。黄色高亮显示将向前移动一行。

1.  导航到**Run/Debug** | **Step Over**，或单击工具栏中的**Step Over**按钮，或按 F10。黄色高亮显示将向前移动一行。目前，您可以看到使用**Step Into**或**Step Over**之间没有区别。

1.  现在您应该在调用`Add`方法的行上，如*图 4.10*所示：![](img/Image00050.jpg)

图 4.10：进入和跳过代码

**Step Into**和**Step Over**之间的区别在于您即将执行方法调用时：

+   如果单击**Step Into**，调试器将*进入*该方法，以便您可以逐行执行该方法中的每一行。

+   如果单击**Step Over**，整个方法将一次执行完毕；它不会跳过方法而不执行它。

1.  单击**Step Into**以进入方法内部。

1.  将鼠标指针悬停在代码编辑窗口中的`a`或`b`参数上，注意到会显示一个工具提示，显示它们的当前值。

1.  选择表达式`a * b`，右键单击表达式，然后选择**Add to Watch**或**Add Watch**。该表达式将添加到**WATCH**窗口，显示该运算符正在将`a`乘以`b`以得到结果`11.25`。

1.  在**WATCH**或**Watch 1**窗口中，右键单击表达式，然后选择**Remove Expression**或**Delete Watch**。

1.  在`Add`函数中将`*`更改为`+`以修复 bug。

1.  停止调试，重新编译，并通过单击圆形箭头**Restart**按钮或按 Ctrl 或 Cmd + Shift + F5 重新启动调试。

1.  跳过该函数，花一分钟时间注意它现在如何正确计算，并单击**Continue**按钮或按**F5**。

1.  使用 Visual Studio Code 时，请注意在调试期间向控制台写入时，输出将显示在**DEBUG CONSOLE**窗口中，而不是**TERMINAL**窗口中，如*图 4.11*所示：![](img/Image00051.jpg)

图 4.11：在调试期间写入 DEBUG CONSOLE

## 自定义断点

很容易创建更复杂的断点：

1.  如果您仍在调试中，请单击调试工具栏中的**Stop**按钮，或导航到**Run/Debug** | **Stop Debugging**，或按 Shift + F5。

1.  导航到**运行** | **删除所有断点**或**调试** | **删除所有断点**。

1.  单击输出答案的`WriteLine`语句。

1.  通过按 F9 或导航到**运行/调试** | **切换断点**来设置断点。

1.  在 Visual Studio Code 中，右键单击断点，选择**编辑断点...**，然后输入一个表达式，比如`answer`变量必须大于 9，并注意表达式必须为真才能激活断点，如*图 4.12*所示：![](img/Image00052.jpg)

图 4.12：使用 Visual Studio Code 自定义断点表达式

1.  在 Visual Studio 中，右键单击断点，选择**条件...**，然后输入一个表达式，比如`answer`变量必须大于 9，并注意表达式必须为真才能激活断点。

1.  开始调试，并注意断点未命中。

1.  停止调试。

1.  编辑断点或其条件，并将其表达式更改为小于 9。

1.  开始调试，并注意断点被命中。

1.  停止调试。

1.  编辑断点或其条件（在 Visual Studio 中单击**添加条件**），然后选择**命中计数**，然后输入一个数字，比如`3`，这意味着您必须命中断点三次才能激活它，如*图 4.13*所示：![图形用户界面，文本，应用程序描述自动生成](img/Image00053.jpg)

图 4.13：使用 Visual Studio 2022 自定义断点表达式和命中计数

1.  将鼠标悬停在断点的红色圆圈上以查看摘要，如*图 4.14*所示：![图形用户界面，文本，应用程序描述自动生成](img/Image00054.jpg)

图 4.14：Visual Studio Code 中自定义断点的摘要

您现在已经使用一些调试工具修复了一个错误，并看到了一些设置断点的高级可能性。

# 开发和运行时日志记录

一旦您相信代码中的所有错误都已经被修复，那么您将编译一个发布版本并部署应用程序，以便人们可以使用它。但是没有代码是完全没有错误的，在运行时可能会发生意外错误。

最终用户极其不擅长记住、承认并准确描述发生错误时他们正在做什么，因此您不应该依赖于他们提供有用的信息来重现问题以理解问题的原因并解决问题。相反，您可以**仪表化您的代码**，这意味着记录感兴趣的事件。

**良好实践**：在整个应用程序中添加代码来记录发生的情况，特别是在发生异常时，以便您可以查看日志并使用它们来跟踪问题并解决问题。虽然我们将在*第十章*，*使用 Entity Framework Core 处理数据*和*第十五章*，*使用模型-视图-控制器模式构建网站*中再次看到日志记录，但日志记录是一个庞大的主题，所以我们只能在本书中涵盖基础知识。

## 了解日志记录选项

.NET 包括一些内置的方法来通过添加日志记录功能来仪表化您的代码。我们将在本书中介绍基础知识。但是日志记录是一个领域，第三方已经创建了丰富的强大解决方案生态系统，扩展了微软提供的功能。我无法提出具体的建议，因为最佳的日志记录框架取决于您的需求。但我在以下列表中包含了一些常见的日志记录框架：

+   Apache log4net

+   NLog

+   Serilog

## 使用 Debug 和 Trace 进行仪表化

有两种类型可用于向您的代码添加简单的日志记录：`Debug`和`Trace`。

在我们更详细地深入研究它们之前，让我们快速概述一下每个选项：

+   `Debug`类用于添加仅在开发期间写入的日志记录。

+   `Trace`类用于添加在开发和运行时都会写入的日志记录。

您已经看到了`Console`类型及其`WriteLine`方法写入到控制台窗口。还有一对名为`Debug`和`Trace`的类型，它们在写入时具有更大的灵活性。

`Debug`和`Trace`类写入到任何跟踪侦听器。跟踪侦听器是一种类型，当调用`WriteLine`方法时，可以配置为在任何地方写入输出。.NET 提供了几个跟踪侦听器，包括一个输出到控制台的侦听器，甚至可以通过继承`TraceListener`类型来创建自己的侦听器。

### 写入到默认跟踪侦听器

一个跟踪侦听器，`DefaultTraceListener`类，会自动配置并写入到 Visual Studio Code 的**DEBUG CONSOLE**窗口或 Visual Studio 的**Debug**窗口。您可以使用代码配置其他跟踪侦听器。

让我们看看跟踪侦听器的作用：

1.  使用您喜欢的编码工具向`Chapter04`工作区/解决方案中添加一个新的**控制台应用程序**，命名为`Instrumenting`。

1.  在 Visual Studio Code 中，选择`Instrumenting`作为活动的 OmniSharp 项目。当您看到弹出的警告消息说缺少所需的资产时，点击**Yes**添加它们。

1.  在`Program.cs`中，导入`System.Diagnostics`命名空间。

1.  从`Debug`和`Trace`类中写入一条消息，如下面的代码所示：

```cs

Debug.WriteLine("调试显示，我在观察！"

);

Trace.WriteLine("跟踪显示，我在观察！"

);

```

1.  在 Visual Studio 中，导航到**View** | **Output**，确保选择**Show output from:** **Debug**。

1.  开始调试`Instrumenting`控制台应用程序，并注意在 Visual Studio Code 的**DEBUG CONSOLE**或 Visual Studio 2022 的**Output**窗口中显示两条消息，混合其他调试信息，如*图 4.15*和*图 4.16*所示：![图形用户界面，文本，网站描述自动生成](img/Image00055.jpg)

图 4.15：Visual Studio Code DEBUG CONSOLE 显示两条消息为蓝色

![](img/Image00056.jpg)

图 4.16：Visual Studio 2022 输出窗口显示调试输出，包括两条消息

## 配置跟踪侦听器

现在，我们将配置另一个跟踪侦听器，它将写入到一个文本文件：

1.  在对`Debug`和`Trace`的`WriteLine`调用之前，添加一个语句在桌面上创建一个新的文本文件，并将其传递给一个知道如何写入文本文件的新的跟踪侦听器，并为其缓冲区启用自动刷新，如下面的代码中所突出显示的那样：

```cs

**// 写入到项目文件夹中的文本文件**

**Trace.Listeners.Add(**

**new**

**TextWriterTraceListener(**

**File.CreateText(Path.Combine(Environment.GetFolderPath(**

**Environment.SpecialFolder.DesktopDirectory),**

**"log.txt"**

**))));**

**// 文本编写器是缓冲的，因此此选项在写入后调用**

**// 在写入后刷新所有侦听器**

**Trace.AutoFlush =**

**true**

**;**

Debug.WriteLine("调试显示，我在观察！"

);

Trace.WriteLine("跟踪显示，我在观察！"

);

```

**良好实践**：任何表示文件的类型通常都会实现缓冲以提高性能。数据不会立即写入文件，而是写入内存缓冲区，只有当缓冲区满时才会一次性写入文件。这种行为在调试时可能会令人困惑，因为我们不会立即看到结果！启用`AutoFlush`意味着在每次写入后自动调用`Flush`方法。

1.  在 Visual Studio Code 中，通过在`Instrumenting`项目的**TERMINAL**窗口中输入以下命令来运行控制台应用程序的发布配置，并注意似乎没有发生任何事情：

```cs

dotnet run --configuration Release

```

1.  在 Visual Studio 2022 中，在标准工具栏中，选择**Solution Configurations**下拉列表中的**Release**，如*图 4.17*所示：![](img/Image00057.jpg)

图 4.17：在 Visual Studio 中选择 Release 配置

1.  在 Visual Studio 2022 中，通过导航到**调试** | **开始时不调试**来运行控制台应用程序的发布配置。

1.  在桌面上，打开名为`log.txt`的文件，并注意它包含消息`Trace says, I am watching!`。

1.  在 Visual Studio Code 中，通过在`Instrumenting`项目的**TERMINAL**窗口中输入以下命令来运行控制台应用程序的调试配置：

```cs

dotnet run --configuration Debug

```

1.  在 Visual Studio 中，从标准工具栏中，在**解决方案配置**下拉列表中选择**Debug**，然后通过导航到**调试** | **开始调试**来运行控制台应用程序。

1.  在桌面上，打开名为`log.txt`的文件，并注意它包含消息`Debug says, I am watching!`和`Trace says, I am watching!`。

**良好实践**：当使用`Debug`配置运行时，`Debug`和`Trace`都是活动的，并且将写入任何跨度侦听器。当使用`Release`配置运行时，只有`Trace`将写入任何跨度侦听器。因此，您可以在代码中自由使用`Debug.WriteLine`调用，知道它们在构建应用程序的发布版本时会自动剥离，并且不会影响性能。

## 切换跟踪级别

`Trace.WriteLine`调用即使在发布后也会留在您的代码中。因此，最好能够在输出它们时有精细控制。这是我们可以使用**跟踪开关**做的事情。

跟踪开关的值可以使用数字或单词进行设置。例如，数字`3`可以替换为单词`Info`，如下表所示：

| 数字 | 单词 | 描述 |
| --- | --- | --- |
| 0 | 关闭 | 这将不输出任何内容。 |
| 1 | 错误 | 这将只输出错误。 |
| 2 | 警告 | 这将输出错误和警告。 |
| 3 | 信息 | 这将输出错误、警告和信息。 |
| 4 | 冗长 | 这将输出所有级别。 |

让我们探索使用跟踪开关。首先，我们将向项目添加一些 NuGet 包，以启用从 JSON`appsettings`文件加载配置设置。

### 在 Visual Studio Code 中向项目添加包

Visual Studio Code 没有添加 NuGet 包到项目的机制，因此我们将使用命令行工具：

1.  导航到`Instrumenting`项目的**TERMINAL**窗口。

1.  输入以下命令：

```cs

dotnet add package Microsoft.Extensions.Configuration

```

1.  输入以下命令：

```cs

dotnet add package Microsoft.Extensions.Configuration.Binder

```

1.  输入以下命令：

```cs

dotnet add package Microsoft.Extensions.Configuration.Json

```

1.  输入以下命令：

```cs

dotnet add package Microsoft.Extensions.Configuration.FileExtensions

```

`dotnet add package`将向您的项目文件添加对 NuGet 包的引用。它将在构建过程中下载。`dotnet add reference`将向您的项目文件添加项目对项目的引用。如果需要，引用的项目将在构建过程中进行编译。

### 在 Visual Studio 2022 中向项目添加包

Visual Studio 有一个用于添加包的图形用户界面。

1.  在**解决方案资源管理器**中，右键单击**Instrumenting**项目，然后选择**管理 NuGet 包**。

1.  选择**浏览**选项卡。

1.  在搜索框中，输入`Microsoft.Extensions.Configuration`。

1.  选择这些 NuGet 包中的每一个，并单击**安装**按钮，如*图 4.18*所示：

1.  `Microsoft.Extensions.Configuration`

1.  `Microsoft.Extensions.Configuration.Binder`

1.  `Microsoft.Extensions.Configuration.Json`

1.  `Microsoft.Extensions.Configuration.FileExtensions`![图形用户界面，文本，应用程序描述自动生成](img/Image00058.jpg)

图 4.18：使用 Visual Studio 2022 安装 NuGet 包

**良好实践**：还有用于从 XML 文件、INI 文件、环境变量和命令行加载配置的包。在项目中使用最合适的技术来设置配置。

### 审查项目包

添加 NuGet 包后，我们可以在项目文件中看到引用：

1.  打开`Instrumenting.csproj`（在 Visual Studio 的**Solution Explorer**中双击**Instrumenting**项目），注意带有添加的 NuGet 包的`<ItemGroup>`部分，如下标记的部分所示：

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

**<PackageReference**

**Include=**

**"Microsoft.Extensions.Configuration"**

**Version=**

**"6.0.0"**

**/>**

**<PackageReference**

**Include=**

**"Microsoft.Extensions.Configuration.Binder"**

**Version=**

**"6.0.0"**

**/>**

**<PackageReference**

**Include=**

**"Microsoft.Extensions.Configuration.FileExtensions"**

**Version=**

**"6.0.0"**

**/>**

**<PackageReference**

**Include=**

**"Microsoft.Extensions.Configuration.Json"**

**Version=**

**"6.0.0"**

**/>**

**</ItemGroup>**

</Project>

```

1.  在`Instrumenting`项目文件夹中添加名为`appsettings.json`的文件。

1.  修改`appsettings.json`以定义名为`PacktSwitch`的设置，并设置`Level`值，如下所示的代码：

```cs

{

"PacktSwitch"

: {

"Level"

: "Info"

}

}

```

1.  在 Visual Studio 2022 中，在**Solution Explorer**中，右键单击`appsettings.json`，选择**Properties**，然后在**Properties**窗口中，将**Copy to Output Directory**更改为**Copy if newer**。这是必要的，因为与在项目文件夹中运行控制台应用程序的 Visual Studio Code 不同，Visual Studio 在`Instrumenting\bin\Debug\net6.0`或`Instrumenting\bin\Release\net6.0`中运行控制台应用程序。

1.  在`Program.cs`的顶部，导入`Microsoft.Extensions.Configuration`命名空间。

1.  在`Program.cs`的末尾添加一些语句，创建一个配置生成器，查找名为`appsettings.json`的当前文件夹中的文件，构建配置，创建一个跟踪开关，通过绑定到配置设置其级别，然后输出四个跟踪开关级别，如下所示的代码：

```cs

ConfigurationBuilder builder = new

();

builder.SetBasePath(Directory.GetCurrentDirectory())

.AddJsonFile("appsettings.json"

,

optional: true

, reloadOnChange: true

);

IConfigurationRoot configuration = builder.Build();

TraceSwitch ts = new

(

displayName: "PacktSwitch"

,

description: "This switch is set via a JSON config."

);

configuration.GetSection("PacktSwitch"

).Bind(ts);

Trace.WriteLineIf(ts.TraceError, "Trace error"

);

Trace.WriteLineIf(ts.TraceWarning, "Trace warning"

);

Trace.WriteLineIf(ts.TraceInfo, "Trace information"

);

Trace.WriteLineIf(ts.TraceVerbose, "Trace verbose"

);

```

1.  在`Bind`语句上设置断点。

1.  开始调试`Instrumenting`控制台应用程序。在**VARIABLES**或**Locals**窗口中，展开`ts`变量表达式，并注意其`Level`为`Off`，`TraceError`，`TraceWarning`等都为`false`，如*图 4.19*所示：![图形用户界面，文本，应用程序描述自动生成](img/Image00059.jpg)

图 4.19：在 Visual Studio 2022 中查看跟踪开关变量属性

1.  通过单击**Step Into**或**Step Over**按钮或按 F11 或 F10 进入`Bind`方法的调用，并注意`ts`变量监视表达式更新为`Info`级别。

1.  步入或跳过对`Trace.WriteLineIf`的四次调用，并注意所有级别直到`Info`都写入**DEBUG CONSOLE**或**Output - Debug**窗口，但`Verbose`不会，如*图 4.20*所示：![图形用户界面，文本，应用程序描述自动生成](img/Image00060.jpg)

图 4.20：在 Visual Studio Code 的 DEBUG CONSOLE 中显示不同的跟踪级别

1.  停止调试。

1.  修改`appsettings.json`以设置`2`级别，表示警告，如下所示的 JSON 文件：

```cs

{

"PacktSwitch"

: {

"Level"

: "2"

}

}

```

1.  保存更改。

1.  在 Visual Studio Code 中，通过在**TERMINAL**窗口中输入以下命令来运行控制台应用程序，为`Instrumenting`项目：

```cs

dotnet run --configuration Release

```

1.  在 Visual Studio 中，在标准工具栏中，选择**解决方案配置**下拉列表中的**发布**，然后通过导航到**调试** | **开始调试**来运行控制台应用程序。

1.  打开名为`log.txt`的文件，并注意这次只有跟踪错误和警告级别是四个潜在跟踪级别的输出，如下文本文件所示：

```cs

跟踪说，我在看着呢！

跟踪错误

跟踪警告

```

如果没有传递参数，则默认的跟踪开关级别为“关闭”（0），因此不会输出任何开关级别。

# 单元测试

修复代码中的错误是昂贵的。在开发过程中发现错误越早，修复的成本就越低。

单元测试是在开发过程中早期发现错误的好方法。一些开发人员甚至遵循程序员应该在编写代码之前创建单元测试的原则，这被称为**测试驱动开发**（**TDD**）。

微软有一个专有的单元测试框架称为**MS Test**。还有一个名为**NUnit**的框架。然而，我们将使用免费的开源第三方框架**xUnit.net**。xUnit 是由构建 NUnit 的同一团队创建的，但他们纠正了他们之前认为犯的错误。xUnit 更具可扩展性，并且有更好的社区支持。

## 了解测试类型

单元测试只是许多测试类型之一，如下表所述：

| 测试类型 | 描述 |
| --- | --- |
| 单元 | 测试代码的最小单元，通常是一个方法或函数。单元测试是在与其依赖项隔离的代码单元上执行的，如果需要，可以通过模拟来隔离它们。每个单元应该有多个测试：一些具有典型输入和预期输出，一些具有极端输入值以测试边界，一些具有故意错误的输入以测试异常处理。 |
| 集成 | 测试较小的单元和较大的组件是否作为软件的单个部分一起工作。有时涉及与您没有源代码的外部组件集成。 |
| 系统 | 测试软件运行环境的整个系统。 |
| 性能 | 测试软件的性能；例如，您的代码必须在不到 20 毫秒的时间内向访问者返回一个充满数据的网页。 |
| 负载 | 测试您的软件在保持所需性能的同时可以处理多少个请求，例如，网站上有 1 万个并发访问者。 |
| 用户验收 | 测试用户是否可以愉快地使用您的软件完成工作。 |

## 创建需要测试的类库

首先，我们将创建一个需要测试的函数。我们将在一个类库项目中创建它。类库是一组可以被其他.NET 应用程序分发和引用的代码包：

1.  使用您喜欢的编码工具将新的**类库**添加到名为`CalculatorLib`的`Chapter04`工作区/解决方案中。`dotnet new`模板名为`classlib`。

1.  将名为`Class1.cs`的文件重命名为`Calculator.cs`。

1.  修改文件以定义一个带有故意错误的`Calculator`类，如下面的代码所示：

```cs

命名空间

Packt

{

公共

类

计算器

{

公共

双

添加

(

双

a，

双

b

）

{

返回

a * b;

}

}

}

```

1.  编译您的类库项目：

1.  在 Visual Studio 2022 中，导航到**生成** | **构建 CalculatorLib**。

1.  在 Visual Studio Code 中，在**TERMINAL**中，输入命令`dotnet build`。

1.  使用您喜欢的编码工具将新的**xUnit 测试项目[C#]**添加到名为`CalculatorLibUnitTests`的`Chapter04`工作区/解决方案中。`dotnet new`模板名为`xunit`。

1.  如果您使用 Visual Studio，在**Solution Explorer**中，选择`CalculatorLibUnitTests`项目，导航到**项目** | **添加项目引用…**，选中`CalculatorLib`项目的复选框，然后单击**确定**。

1.  如果您使用 Visual Studio Code，请使用`dotnet add reference`命令或单击名为`CalculatorLibUnitTests.csproj`的文件，并修改配置以添加一个项目引用到`CalculatorLib`项目的项目组，如下标记所示：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<IsPackable>false

</IsPackable>

</PropertyGroup>

<ItemGroup>

<PackageReference Include="Microsoft.NET.Test.Sdk"

版本="16.10.0"

/>

<PackageReference Include="xunit"

版本="2.4.1"

/>

<PackageReference Include="xunit.runner.visualstudio"

版本="2.4.3"

<IncludeAssets>runtime；build；native；contentfiles；

分析器；buildtransitive</IncludeAssets>

<PrivateAssets>all</PrivateAssets>

</PackageReference>

<PackageReference Include="coverlet.collector"

版本="3.0.2"

<IncludeAssets>runtime；build；native；contentfiles；

分析器；buildtransitive</IncludeAssets>

<PrivateAssets>all</PrivateAssets>

</PackageReference>

</ItemGroup>

**<ItemGroup>**

**<ProjectReference**

**包括=**

**"..\CalculatorLib\CalculatorLib.csproj"**

**/>**

**</ItemGroup>**

</Project>

```

1.  构建`CalculatorLibUnitTests`项目。

## 编写单元测试

一个良好编写的单元测试将包括三个部分：

+   **安排**：这部分将声明和实例化输入和输出变量。

+   **行动**：这部分将执行您正在测试的单元。在我们的案例中，这意味着调用我们想要测试的方法。

+   **断言**：这部分将对输出进行一个或多个断言。断言是一种信念，如果不成立，就表示测试失败。例如，当添加 2 和 2 时，我们期望结果为 4。

现在，我们将为`Calculator`类编写一些单元测试：

1.  将文件`UnitTest1.cs`重命名为`CalculatorUnitTests.cs`，然后打开它。

1.  在 Visual Studio Code 中，将类重命名为`CalculatorUnitTests`。（当您重命名文件时，Visual Studio 会提示您重命名类。）

1.  导入`Packt`命名空间。

1.  修改`CalculatorUnitTests`类，为添加 2 和 2 以及添加 2 和 3 编写两个测试方法，如下所示：

```cs

使用

Packt；

使用

Xunit；

命名空间

CalculatorLibUnitTests

{

public

类

CalculatorUnitTests

{

[事实

]

public

空

TestAdding2And2

()

{

//安排

双重

a=2

;

双重

b=2

;

双重

预期=4

;

Calculator calc = new

();

//行动

双重

实际=calc.Add(a，b);

//断言

Assert.Equal(expected，actual);

}

[事实

]

public

空

TestAdding2And3

()

{

//安排

双重

a=2

;

双重

b=3

;

双重

预期=5

;

Calculator calc = new

();

//行动

双重

实际=calc.Add(a，b);

//断言

Assert.Equal(expected，actual);

}

}

}

```

### 使用 Visual Studio Code 运行单元测试

现在我们准备运行单元测试并查看结果：

1.  在`CalculatorLibUnitTest`项目的**TERMINAL**窗口中，运行测试，如下命令所示：

```cs

dotnet test

```

1.  请注意，结果表明运行了两个测试，一个测试通过了，一个测试失败了，如*图 4.21*所示：![](img/Image00061.jpg)

图 4.21：Visual Studio Code 的 TERMINAL 中的单元测试结果

### 使用 Visual Studio 运行单元测试

现在我们准备运行单元测试并查看结果：

1.  导航到**测试** | **运行所有测试**。

1.  在**测试资源管理器**中，注意结果表明运行了两个测试，一个测试通过了，一个测试失败了，如*图 4.22*所示：![图形用户界面，文本，应用程序，电子邮件描述自动生成](img/Image00062.jpg)

图 4.22：Visual Studio 2022 的测试资源管理器中的单元测试结果

### 修复错误

现在您可以修复错误：

1.  修复`Add`方法中的错误。

1.  再次运行单元测试，以查看错误现在已经被修复，并且两个测试都通过了。

# 在函数中抛出和捕获异常

在*第三章*，*控制流，转换类型和处理异常*中，您已经了解了异常以及如何使用`try-catch`语句来处理它们。但是，只有在有足够的信息来减轻问题时，您才应该捕获和处理异常。如果没有，那么您应该允许异常通过调用堆栈传递到更高级别。

## 理解使用错误和执行错误

**使用错误**是指程序员错误使用函数，通常是通过将无效值作为参数传递。他们可以通过程序员更改其代码以传递有效值来避免。当一些程序员首次学习 C#和.NET 时，他们有时会认为异常总是可以避免，因为他们假设所有错误都是使用错误。在生产运行时之前，应该修复所有使用错误。

**执行错误**是指运行时发生无法通过编写“更好”的代码修复的事情。执行错误可以分为**程序错误**和**系统错误**。如果尝试访问网络资源但网络已关闭，则需要能够通过记录异常来处理该系统错误，并可能暂时后退一段时间并重试。但是，某些系统错误，例如内存耗尽，根本无法处理。如果尝试打开一个不存在的文件，则可能能够捕获该错误并通过编程方式处理它，例如创建一个新文件。程序错误可以通过编写智能代码进行程序化修复。系统错误通常无法通过编程方式修复。

## 函数中常见的抛出异常

很少情况下应该定义新类型的异常来指示使用错误。.NET 已经定义了许多您应该使用的异常。

在定义具有参数的自己的函数时，您的代码应该检查参数值，并在它们具有阻止函数正常运行的值时抛出异常。

例如，如果参数不应为`null`，则抛出`ArgumentNullException`。对于其他问题，请抛出`ArgumentException`，`NotSupportedException`或`InvalidOperationException`。对于任何异常，都包括描述问题的消息（通常是类库和函数的开发人员受众，或者如果它位于 GUI 应用程序的最高级别，则是最终用户），如下面的代码所示：

```cs

静态

void

撤回

（

字符串

accountName，

十进制

量

）

{

如果

（accountName 是

null

）

{

扔

新

ArgumentNullException（paramName：nameof

（accountName）;

}

如果

（金额<0

）

{

扔

新

ArgumentException（

消息：$"

{

nameof

（量）}

不能小于零。"

）;

}

//处理参数

}

```

**良好实践**：如果函数无法成功执行其操作，您应该将其视为函数失败，并通过抛出异常来报告。

您永远不应该需要编写`try-catch`语句来捕获这些使用类型错误。您希望应用程序终止。这些异常应该导致调用函数的程序员修复其代码以防止问题。它们应该在生产部署之前修复。这并不意味着您的代码不需要抛出使用错误类型的异常。您应该这样做-强制其他程序员正确调用您的函数！

## 理解调用堆栈

.NET 控制台应用程序的入口点是`Program`类的`Main`方法，无论您是否明确定义了这个类和方法，还是它是由顶级程序功能为您创建的。

`Main`方法将调用其他方法，然后调用其他方法，依此类推，这些方法可以在当前项目中或在引用的项目和 NuGet 包中，如*图 4.23*所示：

！[](img/Image00063.jpg)

图 4.23：创建调用堆栈的方法调用链

让我们创建一个类似的方法链，以探索我们可以在哪里捕获和处理异常：

1.  使用您喜欢的编码工具向`Chapter04`工作区/解决方案中添加一个名为`CallStackExceptionHandlingLib`的新**类库**。

1.  将`Class1.cs`文件重命名为`Calculator.cs`。

1.  打开`Calculator.cs`并修改其内容，如下所示：

```cs

using

static

System.Console;

namespace

Packt

;

public

class

Calculator

{

public

static

void

Gamma

()

// public so it can be called from outside

{

WriteLine("In Gamma"

);

Delta();

}

private

static

void

Delta

()

// private so it can only be called internally

{

WriteLine("In Delta"

);

File.OpenText("bad file path"

);

}

}

```

1.  使用您喜欢的编码工具向`Chapter04`工作区/解决方案中添加一个名为`CallStackExceptionHandling`的新**控制台应用程序**。

1.  在 Visual Studio Code 中，选择`CallStackExceptionHandling`作为活动的 OmniSharp 项目。当您看到弹出的警告消息说缺少所需的资产时，点击**是**以添加它们。

1.  在`CallStackExceptionHandling`项目中，添加对`CallStackExceptionHandlingLib`项目的引用。

1.  在`Program.cs`中，添加语句来定义两个方法并链接调用它们，以及类库中的方法，如下所示：

```cs

使用

Packt;

使用

static

System.Console;

WriteLine("In Main"

);

Alpha();

static

void

Alpha

()

{

WriteLine("In Alpha"

);

Beta();

}

static

void

Beta

()

{

WriteLine("In Beta"

);

Calculator.Gamma();

}

```

1.  运行控制台应用程序，并注意结果，如下部分输出所示：

```cs

在 Main

在 Alpha

在 Beta

在 Gamma

在 Delta

未处理的异常。System.IO.FileNotFoundException：找不到文件'C:\Code\Chapter04\CallStackExceptionHandling\bin\Debug\net6.0\bad file path'。

在 Microsoft.Win32.SafeHandles.SafeFileHandle.CreateFile(...

在 Microsoft.Win32.SafeHandles.SafeFileHandle.Open(...

在 System.IO.Strategies.OSFileStreamStrategy..ctor(...

在 System.IO.Strategies.FileStreamHelpers.ChooseStrategyCore(...

在 System.IO.Strategies.FileStreamHelpers.ChooseStrategy(...

在 System.IO.StreamReader.ValidateArgsAndOpenPath(...

在 System.IO.File.OpenText(String path)中...

在 C:\Code\Chapter04\CallStackExceptionHandlingLib\Calculator.cs 的 Packt.Calculator.Delta()中：第 16 行

在 C:\Code\Chapter04\CallStackExceptionHandlingLib\Calculator.cs 的 Packt.Calculator.Gamma()中：第 10 行

在 C:\Code\Chapter04\CallStackExceptionHandling\Program.cs 的<Program>$.<<Main>$>g__Beta|0_1()中：第 16 行

在 C:\Code\Chapter04\CallStackExceptionHandling\Program.cs 的<Program>$.<<Main>$>g__Alpha|0_0()中：第 10 行

在 C:\Code\Chapter04\CallStackExceptionHandling\Program.cs 的<Program>$.<Main>$(String[] args)中：第 5 行

```

注意以下内容：

+   调用堆栈是颠倒的。从底部开始，您会看到：

+   第一次调用是对自动生成的`Program`类中的`Main`入口点函数的调用。这是将参数作为`string`数组传递的地方。

+   第二次调用是对`Alpha`函数的调用。

+   第三次调用是对`Beta`函数的调用。

+   第四次调用是对`Gamma`函数的调用。

+   第五次调用是对`Delta`函数的调用。该函数尝试通过传递错误的文件路径来打开文件。这会导致抛出异常。任何带有`try-catch`语句的函数都可以捕获此异常。如果它们没有这样做，它会自动传递到调用堆栈的顶部，直到达到顶部，.NET 会输出异常（以及此调用堆栈的详细信息）。

## 在哪里捕获异常

程序员可以决定是否要在失败点附近捕获异常，或者在调用堆栈的更高位置集中处理异常。这使得您的代码可以简化和标准化。您可能知道调用异常可能会引发一个或多个类型的异常，但您不需要在调用堆栈的当前点处理任何异常。

## 重新抛出异常

有时您希望捕获异常，记录它，然后重新抛出它。有三种在`catch`块内重新抛出异常的方法，如下列表所示：

1.  将捕获的异常与其原始调用堆栈一起抛出，调用`throw`。

1.  要像在调用堆栈的当前级别抛出捕获的异常一样，使用捕获的异常调用`throw`，例如`throw ex`。这通常是不好的做法，因为您可能会丢失一些用于调试的有用信息。

1.  要在另一个异常中包装捕获的异常，该异常可以在消息中包含更多信息，这可能有助于调用者理解问题，请抛出一个新异常，并将捕获的异常作为`innerException`参数传递。

如果在调用`Gamma`函数时可能发生错误，那么我们可以捕获异常，然后执行重新抛出异常的三种技术之一，如下面的代码所示：

```cs

尝试

{

Gamma();

}

catch (IOException ex)

{

LogException(ex);

//像在这里发生一样抛出捕获的异常

//这将丢失原始调用堆栈

抛出

ex;

//重新抛出捕获的异常并保留其原始调用堆栈

抛出

;

//抛出一个包含捕获的异常的新异常

抛出

新

InvalidOperationException(

消息：“计算具有无效值。请参阅内部异常原因。”

，

innerException: ex);

}

```

让我们通过调用堆栈示例来看看这个过程：

1.  在`CallStackExceptionHandling`项目的`Program.cs`中，在`Beta`函数中，添加一个`try-catch`语句来调用`Gamma`函数，如下面的代码所示：

```cs

静态

void

Beta

()

{

WriteLine("在 Beta"``

）;

**尝试**

**{**

**Calculator.Gamma();**

**}**

**catch (Exception ex)**

**{**

**WriteLine(**

**$"捕获到：**

**{ex.Message}**

**“**

**);**

**抛出**

**ex;**

**}**

}

```

1.  注意`ex`下的绿色波浪线，警告您将丢失调用堆栈信息。

1.  运行控制台应用程序并注意输出不包括调用堆栈的某些细节，如下输出所示：

```cs

捕获到：找不到文件'C:\Code\Chapter04\CallStackExceptionHandling\bin\Debug\net6.0\bad file path'。

未处理的异常。System.IO.FileNotFoundException：找不到文件'C:\Code\Chapter04\CallStackExceptionHandling\bin\Debug\net6.0\bad file path'。

文件名：'C:\Code\Chapter04\CallStackExceptionHandling\bin\Debug\net6.0\bad file path'

在 C:\Code\Chapter04\CallStackExceptionHandling\Program.cs:line 25 处的<Program>$.<<Main>$>g__Beta|0_1()

在 C:\Code\Chapter04\CallStackExceptionHandling\Program.cs:line 11 处的<Program>$.<<Main>$>g__Alpha|0_0()

在 C:\Code\Chapter04\CallStackExceptionHandling\Program.cs:line 6 处的<Program>$.<Main>$(String[] args)

```

1.  重新抛出时删除`ex`。

1.  运行控制台应用程序并注意输出包括调用堆栈的所有细节。

## 实现测试者-执行者模式

**测试者-执行者模式**可以避免一些抛出的异常（但不能完全消除）。该模式使用一对函数：一个用于执行测试，另一个用于执行如果测试未通过则会失败的操作。

.NET 本身实现了这种模式。例如，在通过调用`Add`方法向集合添加项目之前，您可以测试是否为只读，这将导致`Add`失败并因此引发异常。

例如，在从银行账户中提取资金之前，您可能会测试账户是否透支，如下面的代码所示：

```cs

如果

（！bankAccount.IsOverdrawn（））

{

bankAccount.Withdraw(amount);

}

```

### 测试者-执行者模式的问题

测试者-执行者模式可能会增加性能开销，因此您还可以实现**尝试模式**，实际上将测试和执行部分合并为单个函数，就像我们在`TryParse`中看到的那样。

测试人员-执行者模式的另一个问题是当您使用多个线程时。在这种情况下，一个线程可以调用测试函数并返回正常。但然后另一个线程执行了改变状态的操作。然后原始线程继续执行，假设一切都很好，但实际上并不好。这被称为竞争条件。我们将看到如何在*第十二章*中处理它，*使用多任务改进性能和可伸缩性*。

如果您实现自己的 try 模式函数并且失败了，请记住将`out`参数设置为其类型的默认值，然后返回`false`，如下面的代码所示：

```cs

静态的

布尔

尝试解析

（

字符串

？输入，

输出

人

值

）

{

如果

（某些失败）

{

值

= 默认

（人）;

返回

假

;

}

//成功将字符串解析为 Person

值

= 新的

Person() { ... };

返回

真

;

}

```

# 练习和探索

通过回答一些问题来测试您的知识和理解，进行一些动手实践，并深入研究本章涵盖的主题。

## 练习 4.1-测试您的知识

回答以下问题。如果遇到困难，请尝试谷歌答案，必要时记住，如果完全困住，答案在附录中（可在[`github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf`](https://github.com/markjprice/cs10dotnet6/blob/main/9781801077361_Bonus_Content.pdf)找到）：

1.  C#关键字`void`的含义是什么？

1.  命令式和函数式编程风格之间有哪些区别？

1.  在 Visual Studio Code 或 Visual Studio 中，按 F5、Ctrl 或 Cmd + F5、Shift + F5 和 Ctrl 或 Cmd + Shift + F5 之间有什么区别？

1.  `Trace.WriteLine`方法的输出写入到哪里？

1.  五个跟踪级别是什么？

1.  `Debug`和`Trace`类之间有什么区别？

1.  在编写单元测试时，三个“A”是什么？

1.  在使用 xUnit 编写单元测试时，必须用什么属性装饰测试方法？

1.  什么`dotnet`命令执行 xUnit 测试？

1.  在不丢失堆栈跟踪的情况下，应该使用什么语句来重新抛出一个名为`ex`的捕获异常？

## 练习 4.2-练习编写带有调试和单元测试的函数

质因数是最小质数的组合，当它们相乘时将产生原始数字。考虑以下例子：

+   4 的质因数是：2 x 2

+   7 的质因数是：7

+   30 的质因数是：5 x 3 x 2

+   40 的质因数是：5 x 2 x 2 x 2

+   50 的质因数是：5 x 5 x 2

创建一个名为`PrimeFactors`的工作空间/解决方案，其中包含三个项目：一个类库，其中有一个名为`PrimeFactors`的方法，当作为参数传递一个`int`变量时，返回显示其质因数的`string`；一个单元测试项目；和一个控制台应用程序来使用它。

为了简单起见，您可以假设输入的最大数字为 1,000。

使用调试工具并编写单元测试，以确保您的函数能够正确处理多个输入并返回正确的输出。

## 练习 4.3-探索主题

使用以下页面上的链接，了解本章涵盖的主题的更多细节：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-4---writing-debugging-and-testing-functions`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-4---writing-debugging-and-testing-functions)

# 摘要

在本章中，您学会了如何以命令式和函数式风格编写具有输入参数和返回值的可重用函数，以及如何使用 Visual Studio 和 Visual Studio Code 调试和诊断功能来修复其中的任何错误。最后，您学会了如何在函数中抛出和捕获异常，并了解调用堆栈。

在下一章中，您将学习如何使用面向对象的编程技术构建自己的类型。
