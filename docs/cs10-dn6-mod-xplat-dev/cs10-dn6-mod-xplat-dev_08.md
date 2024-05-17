# 第八章：使用常见的.NET 类型

本章介绍了.NET 中包含的一些常见类型。这些类型包括用于操作数字、文本、集合、网络访问、反射和属性的类型；改进了使用跨度、索引和范围进行工作；操作图像；以及国际化。

本章涵盖以下主题：

+   处理数字

+   处理文本

+   处理日期和时间

+   使用正则表达式进行模式匹配

+   在集合中存储多个对象

+   使用跨度、索引和范围进行工作

+   处理网络资源

+   使用反射和属性进行工作

+   处理图像

+   国际化您的代码

# 处理数字

数据的最常见类型之一是数字。 .NET 中用于处理数字的最常见类型如下表所示：

| 命名空间 | 示例类型 | 描述 |
| --- | --- | --- |
| `System` | `SByte` , `Int16` , `Int32` , `Int64` | 整数；即零和正整数和负整数 |
| `System` | `Byte` , `UInt16` , `UInt32` , `UInt64` | 无符号整数；即零和正整数 |
| `System` | `Half` , `Single` , `Double` | 实数；即浮点数 |
| `System` | `Decimal` | 精确的实数；即用于科学、工程或金融场景 |
| `System.Numerics` | `BigInteger` , `Complex` , `Quaternion` | 任意大的整数、复数和四元数 |

.NET 自 .NET Framework 1.0 以来就有 32 位浮点和 64 位双精度类型。IEEE 754 规范还定义了 16 位浮点标准。机器学习和其他算法将受益于这种更小、低精度的数字类型，因此微软在 .NET 5 及以后引入了 `System.Half` 类型。

目前，C#语言没有定义 `half` 别名，因此必须使用.NET 类型 `System.Half` 。这在将来可能会改变。

## 处理大整数

可以存储在具有 C#别名的.NET 类型中的最大整数约为十八万五千亿，存储在无符号的 `long` 整数中。但如果需要存储比这更大的数字怎么办？

让我们探索数字：

1.  使用您喜欢的代码编辑器创建一个名为 `Chapter08` 的新解决方案/工作区。

1.  添加控制台应用程序项目，如下列表所示：

1.  项目模板：**控制台应用程序** / `console`

1.  工作区/解决方案文件和文件夹：`Chapter08`

1.  项目文件和文件夹：`WorkingWithNumbers`

1.  在 `Program.cs` 中，删除现有语句并添加一个语句以导入 `System.Numerics` ，如下面的代码所示：

```cs

使用

System.Numerics;

```

1.  添加语句以输出 `ulong` 类型的最大值，以及使用 `BigInteger` 输出 30 位数字的数字，如下面的代码所示：

```cs

WriteLine("处理大整数："

);

WriteLine("-----------------------------------"

);

ulong

big = ulong

.MaxValue;

WriteLine($"

{big,

40

:N0}

"

);

BigInteger bigger =

BigInteger.Parse("123456789012345678901234567890"

);

WriteLine($"

{bigger,

40

:N0}

"

);

```

格式代码中的 `40` 表示右对齐 40 个字符，因此两个数字都对齐到右边缘。 `N0` 表示使用千位分隔符和零位小数。

1.  运行代码并查看结果，如下面的输出所示：

```cs

处理大整数：

----------------------------------------

18,446,744,073,709,551,615

123,456,789,012,345,678,901,234,567,890

```

## 处理复数

复数可以表示为 *a + bi* ，其中 *a* 和 *b* 是实数，*i* 是一个虚数单位，其中 *i* ² *= −1* 。如果实部 *a* 为零，则为纯虚数。如果虚部 *b* 为零，则为实数。

复数在许多**STEM**（**科学、技术、工程和数学**）研究领域有实际应用。此外，它们是通过分别添加和实部和虚部的和数部分而添加的；考虑这个：

```cs

(a + bi) + (c + di) = (a + c) + (b + d)i

```

让我们探索复数：

1.  在`Program.cs`中，添加语句以添加两个复数，如下面的代码所示：

```cs

WriteLine("使用复数："

);

Complex c1 = new

（实数：4

，虚数：2

）;

Complex c2 = new

（实数：3

，虚数：7

）;

Complex c3 = c1 + c2;

// 使用默认的 ToString 实现输出

WriteLine($"

{c1}

加上

{c2}

是

{c3}

"

）;

// 使用自定义格式输出

WriteLine("{0} + {1}i 加上 {2} + {3}i 是 {4} + {5}i"

，

c1.Real, c1.Imaginary,

c2.Real, c2.Imaginary,

c3.Real, c3.Imaginary);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

使用复数：

（4, 2）加上（3, 7）是（7, 9）

4 + 2i 加上 3 + 7i 是 7 + 9i

```

## 理解四元数

四元数是一种扩展复数的数字系统。它们形成了一个四维的、关联的、规范的、除法代数，覆盖了实数，因此也是一个域。

嗯？是的，我知道。我也不明白。别担心；我们不打算使用它们编写任何代码！可以说，它们擅长描述空间旋转，因此视频游戏引擎使用它们，许多计算机模拟和飞行控制系统也使用它们。

# 处理文本

变量的另一种最常见的数据类型是文本。在.NET 中用于处理文本的最常见类型如下表所示：

| 命名空间 | 类型 | 描述 |
| --- | --- | --- |
| `系统` | `Char` | 存储单个文本字符 |
| `系统` | `String` | 存储多个文本字符 |
| `System.Text` | `StringBuilder` | 高效操作字符串 |
| `System.Text.RegularExpressions` | `Regex` | 高效地匹配字符串 |

## 获取字符串的长度

让我们探索一些处理文本时常见的任务；例如，有时您需要找出存储在`string`变量中的文本的长度：

1.  使用您喜欢的代码编辑器将一个名为`WorkingWithText`的新控制台应用添加到`Chapter08`解决方案/工作空间中：

1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择。

1.  在 Visual Studio Code 中，选择`WorkingWithText`作为活动的 OmniSharp 项目。

1.  在`WorkingWithText`项目中，在`Program.cs`中，添加语句来定义一个变量来存储城市伦敦的名称，然后将其名称和长度写入控制台，如下面的代码所示：

```cs

字符串

city = "伦敦"

;

WriteLine($"

{城市}

是

{city.Length}

字符长。

);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

伦敦有 6 个字符长。

```

## 获取字符串的字符

`string`类在内部使用`char`数组来存储文本。它还有一个索引器，这意味着我们可以使用数组语法来读取它的字符。数组索引从零开始，因此第三个字符将在索引 2 处。

让我们看看它的实际效果：

1.  添加一个语句，以在`string`变量中的第一个和第三个位置写入字符，如下面的代码所示：

```cs

WriteLine($"第一个字符是

{城市[

0

]}

和第三个是

{城市[

2

]}

。"

）;

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

第一个字符是 L，第三个是 n。

```

## 分割字符串

有时，您需要在有字符的地方分割一些文本，比如逗号：

1.  添加语句来定义一个包含逗号分隔的城市名称的单个`string`变量，然后使用`Split`方法并指定你想要将逗号作为分隔符，然后枚举返回的`string`值数组，如下面的代码所示：

```cs

字符串

cities = "巴黎，德黑兰，金奈，悉尼，纽约，麦德林"

;

字符串

[] citiesArray = cities.Split(','

);

WriteLine($"数组中有

{citiesArray.Length}

数组中的项目。

);

foreach

（字符串

项目在

citiesArray)

{

WriteLine(item);

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

数组中有 6 个项目。

巴黎

德黑兰

金奈

悉尼

纽约

麦德林

```

在本章的后面，您将学习如何处理更复杂的情况。

## 获取字符串的一部分

有时，您需要获取一些文本的一部分。`IndexOf`方法有九个重载，返回指定`char`或`string`在`string`中的索引位置。`Substring`方法有两个重载，如下列表所示：

+   `Substring(startIndex, length)`：返回从`startIndex`开始并包含接下来的`length`个字符的子字符串。

+   `Substring(startIndex)`：返回从`startIndex`开始并包含直到字符串结束的所有字符的子字符串。

让我们来探讨一个简单的例子：

1.  添加语句将一个人的全名存储在一个带有名字和姓氏之间的空格字符的`string`变量中，找到空格的位置，然后提取名字和姓氏作为两部分，以便它们可以以不同的顺序重新组合，如下代码所示：

```cs

string

fullName = "Alan Jones"

;

int

indexOfTheSpace = fullName.IndexOf(' '

);

string

firstName = fullName.Substring(

startIndex: 0

, length: indexOfTheSpace);

string

lastName = fullName.Substring(

startIndex: indexOfTheSpace + 1

);

WriteLine($"原始：

{fullName}

"

);

WriteLine($"交换：

{lastName}

,

{firstName}

"

);

```

1.  运行代码并查看结果，如下输出所示：

```cs

原始：Alan Jones

交换：Jones, Alan

```

如果初始全名的格式不同，例如`"LastName, FirstName"`，那么代码将需要不同。作为一个可选的练习，尝试编写一些语句，将输入`"Jones, Alan"`更改为`"Alan Jones"`。

## 检查字符串的内容

有时，您需要检查一段文本是否以某些字符开头或结尾，或者是否包含某些字符。您可以使用名为`StartsWith`、`EndsWith`和`Contains`的方法来实现这一点：

1.  添加语句来存储一个`string`值，然后检查它是否以或包含一对不同的`string`值，如下代码所示：

```cs

string

company = "Microsoft"

;

bool

startsWithM = company.StartsWith("M"

);

bool

containsN = company.Contains("N"

);

WriteLine($"文本：

{company}

"

);

WriteLine($"以 M 开头：

{startsWithM}

，包含 N：

{containsN}

"

);

```

1.  运行代码并查看结果，如下输出所示：

```cs

文本：Microsoft

以 M 开头：True，包含 N：False

```

## 连接、格式化和其他字符串成员

还有许多其他`string`成员，如下表所示：

| 成员 | 描述 |
| --- | --- |
| `Trim` , `TrimStart` , `TrimEnd` | 这些方法从开头和/或结尾修剪空格字符，例如空格、制表符和回车。 |
| `ToUpper` , `ToLower` | 这些将所有字符转换为大写或小写。 |
| `Insert` , `Remove` | 这些方法插入或删除一些文本。 |
| `Replace` | 这将一些文本替换为其他文本。 |
| `string.Empty` | 这可以用来代替每次使用一个空的双引号(`""`)分配内存来使用一个文字`string`值。 |
| `string.Concat` | 这将两个`string`变量连接起来。当在`string`操作数之间使用+运算符时，它会执行相同的操作。 |
| `string.Join` | 这将一个或多个`string`变量与一个字符连接起来。 |
| `string.IsNullOrEmpty` | 这检查一个`string`变量是否为`null`或空。 |
| `string.IsNullOrWhitespace` | 这检查一个`string`变量是否为`null`或空格；也就是说，任意数量的水平和垂直间距字符的混合，例如，制表符、空格、回车、换行等等。 |
| `string.Format` | 一种用于输出格式化`string`值的替代方法，它使用位置参数而不是命名参数。 |

前面的一些方法是静态方法。这意味着该方法只能从类型调用，而不能从变量实例调用。在前面的表中，我通过在其前面加上`string.`来表示静态方法，如`string.Format`。

让我们探索其中一些方法：

1.  添加语句以获取字符串值数组并使用`Join`方法将它们重新组合成单个字符串变量，如下面的代码所示：

```cs

字符串

recombined = 字符串

.Join(" =>

，citiesArray);

WriteLine(recombined);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

巴黎=>德黑兰=>金奈=>悉尼=>纽约=>麦德林

```

1.  添加语句以使用定位参数和插值字符串格式化语法两次输出相同的三个变量，如下面的代码所示：

```cs

字符串

fruit = "苹果"

;

十进制

价格 = 0.39

M;

DateTime when

= DateTime.Today;

WriteLine($"插值：

{水果}

成本

{price:C}

上

{

何时

:dddd}

。"

);

WriteLine(string

.Format("string.Format：{0}在{2:dddd}花费{1:C}。"

，

arg0: 水果，arg1: 价格，arg2: 何时

））;

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

插值：苹果星期四花费£0.39。

string.Format：苹果星期四花费£0.39。

```

请注意，我们可以简化第二个语句，因为`WriteLine`支持与`string.Format`相同的格式代码，如下面的代码所示：

```cs

WriteLine("WriteLine：{0}在{2:dddd}花费{1:C}。"

，

arg0: 水果，arg1: 价格，arg2: 何时

);

```

## 高效构建字符串

您可以使用`String.Concat`方法将两个字符串连接起来，以创建一个新的`string`，或者只是使用`+`运算符。但这两种选择都是不好的做法，因为.NET 必须在内存中创建一个全新的`string`。

如果只添加两个`string`值，则可能不会注意到这一点，但如果在具有许多迭代的循环中进行连接，它可能会对性能和内存使用产生重大负面影响。在*第十二章*，*使用多任务改进性能和可伸缩性*，您将学习如何使用`StringBuilder`类型高效地连接`string`变量。

# 处理日期和时间

在数字和文本之后，要处理的下一个最受欢迎的数据类型是日期和时间。两种主要类型如下：

+   `DateTime`：表示固定时间点的组合日期和时间值。

+   `TimeSpan`：表示时间的持续时间。

这两种类型经常一起使用。例如，如果你从另一个`DateTime`值中减去一个`DateTime`值，结果就是一个`TimeSpan`。如果你将一个`TimeSpan`添加到一个`DateTime`中，结果就是一个`DateTime`值。

## 指定日期和时间值

创建日期和时间值的常见方法是指定日期和时间组件的单独值，如下表所述：

| 日期/时间参数 | 值范围 |
| --- | --- |
| `year` | 1 到 9999 |
| `month` | 1 到 12 |
| `day` | 1 到该月的天数 |
| `hour` | 0 到 23 |
| `minute` | 0 到 59 |
| `second` | 0 到 59 |

另一种方法是将值提供为要解析的`string`，但这取决于线程的默认文化，可能会被误解。例如，在英国，日期是以日/月/年的方式指定的，而在美国，日期是以月/日/年的方式指定的。

让我们看看您可能想要处理日期和时间的内容：

1.  使用您喜欢的代码编辑器将一个名为`WorkingWithTime`的新控制台应用添加到`Chapter08`解决方案/工作区中。

1.  在 Visual Studio Code 中，将`WorkingWithTime`选择为活动的 OmniSharp 项目。

1.  在`Program.cs`中，删除现有语句，然后添加语句以初始化一些特殊的日期/时间值，如下面的代码所示：

```cs

WriteLine("最早的日期/时间值为：{0}"

，

arg0: DateTime.MinValue);

WriteLine("UNIX 纪元日期/时间值为：{0}"

,

arg0: DateTime.UnixEpoch);

WriteLine("日期/时间值现在是：{0}"

，

arg0：DateTime.Now);

WriteLine("今天的日期/时间值是：{0}"

，

arg0：DateTime.Today);

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

最早的日期/时间值是：01/01/0001 00:00:00

UNIX 纪元日期/时间值是：01/01/1970 00:00:00

现在的日期/时间值是：2021 年 4 月 23 日 14:14:54

今天的日期/时间值是：23/04/2021 00:00:00

```

1.  添加语句来定义 2021 年的圣诞节（如果这是过去的日期，则使用未来的年份），并以各种方式显示它，如下面的代码所示：

```cs

DateTime 圣诞节=新的

（年：2021

，月：12

，日：25

）;

WriteLine("圣诞节：{0}"

，

arg0：圣诞节）；//默认格式

WriteLine("圣诞节：{0:dddd, dd MMMM yyyy}"

，

arg0：圣诞节）；//自定义格式

WriteLine("圣诞节是一年中的第{0}个月。"

，

arg0：圣诞节月）;

WriteLine("圣诞节是一年中的第{0}天。"

，

arg0：圣诞节一年中的第一天）;

WriteLine("圣诞节{0}是星期{1}。"

，

arg0：圣诞节年，

arg1：圣诞节星期几）;

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

圣诞节：2021 年 12 月 25 日 00:00:00

圣诞节：星期六，2021 年 12 月 25 日

圣诞节是一年中的第 12 个月。

圣诞节是一年中的第 359 天。

2021 年的圣诞节是在星期六。

```

1.  添加语句以执行圣诞节的加法和减法，如下面的代码所示：

```cs

DateTime beforeXmas = christmas.Subtract(TimeSpan.FromDays(12

）;

DateTime afterXmas = christmas.AddDays(12

）;

WriteLine("圣诞节前 12 天是：{0}"

，

arg0：beforeXmas);

WriteLine("圣诞节后 12 天是：{0}"

，

arg0：afterXmas);

直到圣诞节的时间跨度=圣诞节- DateTime.Now;

WriteLine("距离圣诞节还有{0}天{1}小时。"

，

arg0：untilChristmas.Days，

arg1：untilChristmas.Hours);

WriteLine("距离圣诞节还有{0:N0}小时。"

，

arg0：untilChristmas.TotalHours);

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

圣诞节前 12 天是：2021 年 12 月 13 日 00:00:00

圣诞节后 12 天是：2022 年 1 月 6 日 00:00:00

距离圣诞节还有 245 天 9 小时。

距离圣诞节还有 5,890 小时。

```

1.  添加语句以定义您的孩子可能在圣诞节醒来开礼物的时间，并以各种方式显示它，如下面的代码所示：

```cs

DateTime kidsWakeUp =新的

(

年：2021

，月：12

，日：25

，

小时：6

，分钟：30

，秒：0

）;

WriteLine("孩子们在圣诞节醒来：{0}"

，

arg0：kidsWakeUp);

WriteLine("孩子们在{0}叫醒我"

，

arg0：kidsWakeUp.ToShortTimeString());

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

孩子们在圣诞节醒来：2021 年 12 月 25 日 06:30:00

孩子们在 06:30 叫醒我

```

## 全球化与日期和时间

当前文化控制日期和时间的解析方式：

1.  在`Program.cs`的顶部，导入`System.Globalization`命名空间。

1.  添加语句以显示用于显示日期和时间值的当前文化，然后解析美国独立日并以各种方式显示它，如下面的代码所示：

```cs

当前文化是：en-GB

，

arg0：CultureInfo.CurrentCulture.Name);

字符串

textDate = "2021 年 7 月 4 日"

;

DateTime independenceDay = DateTime.Parse(textDate);

WriteLine("文本：{0}，日期时间：{1:d MMMM}"

，

arg0：textDate，

arg1：独立日）;

textDate = "7/4/2021"

;

独立日 = DateTime.Parse(textDate);

WriteLine("文本：{0}，日期时间：{1:d MMMM}"

，

arg0：textDate，

arg1：独立日）;

独立日= DateTime.Parse(textDate，

提供程序：CultureInfo.GetCultureInfo("en-US"

）;

WriteLine("文本：{0}，日期时间：{1:d MMMM}"

，

arg0：textDate，

arg1：独立日）;

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

当前文化是：en-GB

文本：2021 年 7 月 4 日，日期时间：7 月 4 日

文本：7/4/2021，日期时间：7 月 4 日

文本：7/4/2021，日期时间：7 月 4 日

```

在我的计算机上，当前的文化是英式英语。如果日期是 2021 年 7 月 4 日，那么无论当前文化是英式还是美式，它都会被正确解析。但是，如果日期是 7/4/2021，那么它会被错误解析为 4 月 7 日。您可以通过在解析时指定正确的文化作为提供程序来覆盖当前的文化，如上面的第三个示例所示。

1.  添加语句循环从 2020 年到 2025 年，显示该年是否为闰年以及 2 月有多少天，然后显示圣诞节和独立日是否在夏令时期间，如下面的代码所示：

```cs

对于

（int

年份=2020

；年份<2026

；年份++)

{

Write($"

{年份}

是闰年：

{DateTime.IsLeapYear(year)}

。"

）；

WriteLine("2 月有{0}天。"

，

arg0：DateTime.DaysInMonth(year: year, month: 2

），arg1：年份）;

}

WriteLine("圣诞节是夏令时吗？{0}"

，

arg0：christmas.IsDaylightSavingTime());

WriteLine("7 月 4 日是夏令时吗？{0}"

，

arg0：independenceDay.IsDaylightSavingTime());

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

2020 是闰年：真。2020 年 2 月有 29 天。

2021 不是闰年：假。2021 年 2 月有 28 天。

2022 不是闰年：假。2022 年 2 月有 28 天。

2023 不是闰年：假。2023 年 2 月有 28 天。

2024 是闰年：真。2024 年 2 月有 29 天。

2025 不是闰年：假。2025 年 2 月有 28 天。

圣诞节是夏令时吗？假

7 月 4 日是夏令时吗？真

```

## 仅使用日期或时间

.NET 6 引入了一些新类型，用于处理仅日期值或仅时间值，命名为`DateOnly`和`TimeOnly`。这比使用带有零时间的`DateTime`值来存储仅日期值更好，因为它是类型安全的，避免了误用。`DateOnly`还更好地映射到数据库列类型，例如 SQL Server 中的`date`列。`TimeOnly`适用于设置闹钟和安排定期会议或活动，并映射到 SQL Server 中的`time`列。

让我们用它们来为英国女王计划一个派对：

1.  添加语句来定义女王的生日，以及她的派对开始时间，然后将这两个值组合起来，以便我们不会错过她的派对，如下面的代码所示：

```cs

DateOnly queensBirthday = new

（年份：2022

，月份：4

，日：21

）；

WriteLine($"女王的下一个生日是在

{queensBirthday}

。"

）；

TimeOnly partyStarts = new

（小时：20

，分钟：30

）；

WriteLine($"女王的派对从

{partyStarts}

。"

）；

DateTime calendarEntry = queensBirthday.ToDateTime(partyStarts);

WriteLine($"添加到您的日历：

{calendarEntry}

。"

）；

```

1.  运行代码并注意结果，如下面的输出所示：

```cs

女王的下一个生日是在 2022 年 4 月 21 日。

女王的派对从 20:30 开始。

添加到您的日历：2022 年 4 月 21 日 20:30:00。

```

# 使用正则表达式进行模式匹配

正则表达式对于验证用户输入非常有用。它们非常强大，可以变得非常复杂。几乎所有的编程语言都支持正则表达式，并使用一组常见的特殊字符来定义它们。

让我们尝试一些示例正则表达式：

1.  使用您喜欢的代码编辑器将一个名为`WorkingWithRegularExpressions`的新控制台应用添加到`Chapter08`解决方案/工作区中。

1.  在 Visual Studio Code 中，将`WorkingWithRegularExpressions`选择为活动的 OmniSharp 项目。

1.  在`Program.cs`中，导入以下命名空间：

```cs

使用

System.Text.RegularExpressions;

```

## 检查以文本形式输入的数字

我们将首先实现验证数字输入的常见示例：

1.  添加语句提示用户输入他们的年龄，然后使用查找数字字符的正则表达式来检查其是否有效，如下面的代码所示：

```cs

Write("输入您的年龄："

）；

字符串

？输入= ReadLine();

Regex ageChecker = new

(@"\d"

);

if

（ageChecker.IsMatch(input)）

{

WriteLine("谢谢！"

);

}

否则

{

WriteLine($"这不是一个有效的年龄：

{input}

"

);

}

```

注意代码中的以下内容：

+   `@`字符关闭了在字符串中使用转义字符的能力。转义字符以反斜杠为前缀。例如，`\t`表示制表符，`\n`表示换行。在编写正则表达式时，我们需要禁用此功能。用《白宫风云》电视节目的话来说，“让反斜杠成为反斜杠。”

+   一旦使用`@`禁用了转义字符，它们就可以被正则表达式解释。例如，`\d`表示数字。您将在本主题后面学习更多以反斜杠为前缀的正则表达式。

1.  运行代码，输入一个整数，如年龄`34`，并查看结果，如下面的输出所示：

```cs

输入您的年龄：34

谢谢！

```

1.  再次运行代码，输入`carrots`，并查看结果，如下面的输出所示：

```cs

输入您的年龄：carrots

这不是一个有效的年龄：carrots

```

1.  再次运行代码，输入`bob30smith`，并查看结果，如下面的输出所示：

```cs

输入您的年龄：bob30smith

谢谢！

```

我们使用的正则表达式是`\d`，意思是*一个数字*。但它没有指定在那一个数字之前和之后可以输入什么。这个正则表达式可以用英语描述为“输入任何字符，只要至少输入一个数字字符。”

在正则表达式中，您用插入符`^`符号表示输入的开始，用美元`$`符号表示输入的结束。让我们使用这些符号来表示我们期望在输入的开始和结束之间除了一个数字之外什么都不输入。

1.  将正则表达式更改为`^\d$`，如下面的代码中所示高亮显示：

```cs

Regex ageChecker = new

(@"^

**\d$"**

);

```

1.  再次运行代码，并注意它拒绝除一个数字之外的任何输入。我们希望允许一个或多个数字。为此，我们在`\d`表达式后添加`+`以修改其含义为一个或多个。

1.  将正则表达式更改，如下面的代码中所示高亮显示：

```cs

Regex ageChecker = new

(@"^

**\d+$"**

);

```

1.  再次运行代码，并注意正则表达式只允许零或正整数的任意长度。

## 正则表达式性能改进

在整个.NET 平台和许多使用.NET 构建的应用程序中，使用正则表达式的.NET 类型。因此，它们对性能有重大影响，但直到现在，它们并没有得到微软的太多优化关注。

在.NET 5 及以后，`System.Text.RegularExpressions`命名空间已重写内部以获得最大性能。使用`IsMatch`等方法的常见正则表达式基准现在快了五倍。最好的是，您无需更改代码即可获得这些好处！

## 理解正则表达式的语法

以下是一些常见的正则表达式符号，您可以在正则表达式中使用：

| 符号 | 意义 | 符号 | 意义 |
| --- | --- | --- | --- |
| `^` | 输入的开头 | `$` | 输入的结尾 |
| `\d` | 一个数字 | `\D` | 一个非数字 |
| `\s` | 空白字符 | `\S` | 非空白字符 |
| `\w` | 单词字符 | `\W` | 非单词字符 |
| `[A-Za-z0-9]` | 字符范围 | `\^` | ^（插入符）字符 |
| `[aeiou]` | 字符集 | `[^aeiou]` | 非字符集中的字符 |
| `.` | 任意单个字符 | `\.` | .（点）字符 |

此外，这里有一些影响正则表达式中先前符号的正则表达式量词：

| 符号 | 意义 | 符号 | 意义 |
| --- | --- | --- | --- |
| `+` | 一个或多个 | `?` | 一个或零个 |
| `{3}` | 恰好三个 | `{3,5}` | 三到五个 |
| `{3,}` | 至少三个 | `{,3}` | 最多三个 |

## 正则表达式示例

以下是一些带有其含义描述的正则表达式示例：

| 表达式 | 意义 |
| --- | --- |
| `\d` | 输入中的单个数字 |
| `a` | 输入中的字符*a* |
| `Bob` | 输入中的*Bob* |
| `^Bob` | 输入的开头是*Bob* |
| `Bob$` | 输入的结尾是*Bob* |
| `^\d{2}$` | 正好两位数字 |
| `^[0-9]{2}$` | 正好两位数字 |
| `^[A-Z]{4,}$` | 至少四个大写英文字母，仅在 ASCII 字符集中 |
| `^[A-Za-z]{4,}$` | ASCII 字符集中至少四个大写或小写英文字母 |
| `^[A-Z]{2}\d{3}$` | ASCII 字符集中的两个大写英文字母和三个数字 |
| `^[A-Za-z\u00c0-\u017e]+$` | ASCII 字符集中至少一个大写或小写英文字母，或 Unicode 字符集中的欧洲字母，如下列表所示：ÀÁÂÃÄÅÆÇÈÉÊËÌÍÎÏÐÑÒÓÔÕÖ×ØÙÚÛÜÝÞßàáâãäåæçèéêëìíîïðñòóôõö÷øùúûüýþÿıŒœŠšŸ Žž |
| `^d.g$` | 字母*d*，然后是任何字符，然后是字母*g*，因此它将匹配*dig*和*dog*或*d*和*g*之间的任何单个字符 |
| `^d\.g$` | 字母*d*，然后是一个点(.)，然后是字母*g*，因此它只匹配*d.g* |

**良好实践**：使用正则表达式验证用户输入。相同的正则表达式可以在 JavaScript 和 Python 等其他语言中重复使用。

## 拆分复杂的逗号分隔字符串

在本章的前面，您学习了如何拆分一个简单的逗号分隔的字符串变量。但是对于以下电影标题的示例呢？

```cs

"怪物公司"

，“我，托尼娅”

，"Lock, Stock and Two Smoking Barrels"

```

`string`值在每个电影标题周围使用双引号。我们可以使用这些来确定我们是否需要在逗号上拆分（或不拆分）。`Split`方法不够强大，因此我们可以使用正则表达式代替。

**良好实践**：您可以在以下链接的 Stack Overflow 文章中阅读更详细的解释，该文章启发了此任务：[`stackoverflow.com/questions/18144431/regex-to-split-a-csv`](https://stackoverflow.com/questions/18144431/regex-to-split-a-csv)

为了在`string`值中包含双引号，我们使用反斜杠进行前缀处理：

1.  添加语句以存储一个复杂的逗号分隔的`string`变量，然后使用`Split`方法以愚蠢的方式拆分，如下面的代码所示：

```cs

字符串

电影=“\“怪物公司\”，\“我，托尼娅\”，\“锁，库存和两根烟\””

;

WriteLine($"要拆分的电影：

{电影}

"

);

字符串

[] filmsDumb = films.Split(','

);

WriteLine("使用 string.Split 方法拆分："

);

foreach

（字符串

电影在

filmsDumb)

{

WriteLine(film);

}

```

1.  添加语句以定义一个正则表达式，以智能方式拆分并写入电影标题，如下面的代码所示：

```cs

WriteLine();

Regex csv = 新

(

"(?:^|,)(?=[^\"]|(\")?)\"?((?(1)[^\"]*|[^,\"]*))\"?(?=,|$)"

);

MatchCollection filmsSmart = csv.Matches(films);

WriteLine("使用正则表达式拆分："

);

foreach

(匹配电影

filmsSmart)

{

WriteLine(film.Groups[2

].Value);

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

使用 string.Split 方法拆分：

"怪物

公司"

"我

托尼娅"

"锁

库存和两根烟"

使用正则表达式拆分：

怪物公司

我，托尼娅

Lock, Stock and Two Smoking Barrels

```

# 在集合中存储多个对象

另一种最常见的数据类型是集合。如果需要在一个变量中存储多个值，那么可以使用集合。

集合是内存中的数据结构，可以以不同的方式管理多个项目，尽管所有集合都具有一些共享功能。

.NET 中用于处理集合的最常见类型如下表所示：

| 命名空间 | 示例类型 | 描述 |
| --- | --- | --- |
| `System .Collections` | `IEnumerable`，`IEnumerable<T>` | 集合使用的接口和基类。 |
| `System .Collections .Generic` | `List<T>`，`Dictionary<T>`，`Queue<T>`，`Stack<T>` | 在 C# 2.0 和.NET Framework 2.0 中引入。这些集合允许您使用泛型类型参数指定要存储的类型（更安全，更快，更高效）。 |
| `System .Collections .Concurrent` | `BlockingCollection`，`ConcurrentDictionary`，`ConcurrentQueue` | 这些集合在多线程场景中使用是安全的。 |
| `System .Collections .Immutable` | `ImmutableArray`，`ImmutableDictionary`，`ImmutableList`，`ImmutableQueue` | 设计用于原始集合的内容永远不会更改的情况，尽管它们可以创建修改后的集合作为新实例。 |

## 所有集合的共同特征

所有集合都实现了`ICollection`接口；这意味着它们必须有一个`Count`属性来告诉您其中有多少个对象，如下面的代码所示：

```cs

命名空间

System.Collections

{

公共

接口

ICollection

: IEnumerable

{

int

计数{获取

; }

布尔

IsSynchronized {获取

; }

对象

SyncRoot {获取

; }

void

CopyTo

(

数组数组，

int

索引

)

;

}

}

```

例如，如果我们有一个名为`passengers`的集合，我们可以这样做：

```cs

int

howMany = passengers.Count;

```

所有集合都实现了`IEnumerable`接口，这意味着它们可以使用`foreach`语句进行迭代。它们必须有一个`GetEnumerator`方法，返回一个实现了`IEnumerator`的对象；这意味着返回的`object`必须有用于导航集合的`MoveNext`和`Reset`方法，以及包含集合中当前项的`Current`属性，如下面的代码所示：

```cs

命名空间

System.Collections

{

公共

接口

IEnumerable

{

IEnumerator

GetEnumerator

()

;

}

}

命名空间

System.Collections

{

公共

接口

IEnumerator

{

对象

当前{获取

; }

布尔

MoveNext

()

;

void

重置

()

;

}

}

```

例如，要对`passengers`集合中的每个对象执行操作，我们可以编写以下代码：

```cs

foreach

（乘客 p in

乘客）

{

//对每个乘客执行操作

}

```

除了基于`object`的集合接口外，还有泛型接口和类，其中泛型类型定义了集合中存储的类型，如下面的代码所示：

```cs

命名空间

System.Collections.Generic

{

公共

接口

ICollection

<T

> : IEnumerable

<T

>，IEnumerable

{

int

计数{获取

; }

布尔

IsReadOnly {获取

; }

void

添加

(

T 项

)

;

void

清除

()

;

布尔

包含

(

T 项

)

;

void

CopyTo

(

T[]数组，

int

索引

)

;

布尔

删除

(

T 项

)

;

}

}

```

## 通过确保集合的容量来提高性能

自.NET 1.1 以来，像`StringBuilder`这样的类型已经有了一个名为`EnsureCapacity`的方法，可以将其内部存储数组的预期最终大小预先设置为`string`的大小。这样做可以提高性能，因为它不必在附加更多字符时重复增加数组的大小。

自.NET Core 2.1 以来，像`Dictionary<T>`和`HashSet<T>`也有了`EnsureCapacity`。

在.NET 6 及更高版本中，像`List<T>`，`Queue<T>`和`Stack<T>`这样的集合现在也有`EnsureCapacity`方法，如下面的代码所示：

```cs

List<string

> names = new

();

names.EnsureCapacity(10

_000);

//将一万个名称加载到列表中

```

## 了解集合选择

有几种不同的集合选择可供您用于不同的目的：列表、字典、堆栈、队列、集合以及许多其他更专业的集合。

### 列表

列表，即实现`IList<T>`的类型，是**有序集合**，如下面的代码所示：

```cs

命名空间

System.Collections.Generic

{

默认成员（

"Item"

)

] //也称为此索引器

公共

接口

IList

<T

> : ICollection

<T

>，IEnumerable

<T

>，IEnumerable

{

T this

[int

索引] {获取

; 设置

; }

int

IndexOf

(

T 项

)

;

void

插入

(

int

索引，T 项

)

;

void

RemoveAt

(

int

索引

)

;

}

}

```

`IList<T>`派生自`ICollection<T>`，因此它具有`Count`属性，以及`Add`方法将项目放在集合的末尾，以及`Insert`方法将项目放在列表中的指定位置，以及`RemoveAt`以删除指定位置的项目。

当您想要手动控制集合中项目的顺序时，列表是一个不错的选择。列表中的每个项目都有一个唯一的索引（或位置），该索引（或位置）会自动分配。项目可以是由`T`定义的任何类型，并且项目可以重复。索引是`int`类型，并且从`0`开始，因此列表中的第一个项目位于索引`0`，如下表所示：

| 索引 | 项目 |
| --- | --- |
| 0 | 伦敦 |
| 1 | 巴黎 |
| 2 | 伦敦 |
| 3 | 悉尼 |

如果在伦敦和悉尼之间插入了一个新项目（例如，圣地亚哥），那么悉尼的索引会自动增加。因此，您必须意识到在插入或删除项目后，项目的索引可能会发生变化，如下表所示：

| 索引 | 项目 |
| --- | --- |
| 0 | 伦敦 |
| 1 | 巴黎 |
| 2 | 伦敦 |
| 3 | 圣地亚哥 |
| 4 | 悉尼 |

### 字典

当每个**值**（或对象）都有一个唯一的子值（或虚构的值）可以用作以后在集合中快速找到值的**键**时，字典是一个不错的选择。键必须是唯一的。例如，如果您正在存储人员列表，您可以选择使用政府颁发的身份证号作为键。

将键视为真实世界字典中的索引条目。它允许您快速找到单词的定义，因为单词（例如键）被保持排序，如果我们知道我们正在寻找*manatee*的定义，我们会跳到字典的中间开始查找，因为字母*M*在字母表的中间。

在编程中，字典在查找东西时同样聪明。它们必须实现接口`IDictionary<TKey, TValue>`，如下面的代码所示：

```cs

命名空间

System.Collections.Generic

{

[DefaultMember(

"项目"

）

] //也称为这个索引器

公共

接口

IDictionary

<TKey

，TValue

：ICollection

<KeyValuePair

<TKey

，TValue

>>,

IEnumerable

<KeyValuePair

<TKey

，TValue

>>, IEnumerable

{

TValue this

[TKey 键] {获取

; set

; }

ICollection<TKey>键 {获取

; }

ICollection<TValue>值 {获取

; }

空

添加

(

TKey 键，TValue

值

）

;

布尔值

ContainsKey

(

TKey 键

）

;

布尔值

删除

(

TKey 键

）

;

布尔值

TryGetValue

(

TKey 键，[MaybeNullWhen(

假

)]

out

TValue

值

）

;

}

}

```

字典中的项目是`struct`的实例，也称为值类型`KeyValuePair<TKey, TValue>`，其中`TKey`是键的类型，`TValue`是值的类型，如下面的代码所示：

```cs

命名空间

System.Collections.Generic

{

公共

只读

结构

KeyValuePair<TKey, TValue>

{

公开

KeyValuePair

(

TKey 键，TValue

值

）

;

公共

TKey 键 {获取

; }

公共

TValue 值 {获取

; }

[EditorBrowsable(EditorBrowsableState.Never)

]

公共

空

解构

(

out

TKey 键，

out

TValue

值

）

;

公共

覆盖

字符串

ToString

()

;

}

}

```

例如，`Dictionary<string, Person>`使用`string`作为键，`Person`实例作为值。`Dictionary<string, string>`使用`string`值，如下表所示：

| 键 | 值 |
| --- | --- |
| BSA | Bob Smith |
| MW | Max Williams |
| BSB | Bob Smith |
| AM | Amir Mohammed |

### 堆栈

当您想要实现**后进先出**（**LIFO**）行为时，堆栈是一个不错的选择。使用堆栈，您只能直接访问或删除堆栈顶部的一个项目，尽管您可以枚举以读取整个项目堆栈。例如，您不能直接访问堆栈中的第二个项目。

例如，文字处理器使用堆栈来记住您最近执行的操作序列，然后当您按下 Ctrl + Z 时，它将撤消堆栈中的最后一个操作，然后是倒数第二个操作，依此类推。

### 队列

当您想要实现**先进先出**（**FIFO**）行为时，队列是一个不错的选择。使用队列，您只能直接访问或移除队列前面的一个项目，尽管您可以枚举以读取整个项目队列。例如，您不能直接访问队列中的第二个项目。

例如，后台进程使用队列按照它们到达的顺序处理工作项，就像人们在邮局排队一样。

.NET 6 引入了`PriorityQueue`，队列中的每个项目都有一个分配的优先级值以及它们在队列中的位置。

### 集合

当您想要在两个集合之间执行集合操作时，集合是一个不错的选择。例如，您可能有两个城市名称的集合，并且想知道哪些名称出现在两个集合中（称为集合之间的*交集*）。集合中的项目必须是唯一的。

### 集合方法摘要

每个集合都有一组不同的方法来添加和删除项目，如下表所示：

| 集合 | 添加方法 | 删除方法 | 描述 |
| --- | --- | --- | --- |
| 列表 | `Add`，`Insert` | `Remove`，`RemoveAt` | 列表是有序的，因此项目具有整数索引位置。`Add`将在列表的末尾添加一个新项目。`Insert`将在指定的索引位置添加一个新项目。 |
| 字典 | `Add` | `Remove` | 字典没有顺序，因此项目没有整数索引位置。您可以通过调用`ContainsKey`方法来检查是否使用了键。 |
| 栈 | `Push` | `Pop` | 栈总是使用`Push`方法在栈的顶部添加新项目。第一个项目在底部。使用`Pop`方法总是从栈的顶部移除项目。调用`Peek`方法可以查看该值而不移除它。 |
| 队列 | `Enqueue` | `Dequeue` | 队列总是使用`Enqueue`方法在队列的末尾添加新项目。第一个项目在队列的前面。使用`Dequeue`方法总是从队列的前面移除项目。调用`Peek`方法可以查看该值而不移除它。 |

## 使用列表

让我们探索列表：

1.  使用您喜欢的代码编辑器向`Chapter08`解决方案/工作区添加一个名为`WorkingWithCollections`的新控制台应用程序。

1.  在 Visual Studio Code 中，将`WorkingWithCollections`选择为活动的 OmniSharp 项目。

1.  在`Program.cs`中，删除现有的语句，然后定义一个函数来输出一个带有标题的`string`值集合，如下面的代码所示：

```cs

static

void

Output

(

string

标题，IEnumerable<

string

> 集合

)

{

WriteLine(title);

foreach

(string

项目

集合)

{

WriteLine($"

{item}

"

);

}

}

```

1.  定义一个名为`WorkingWithLists`的静态方法，以演示定义和使用列表的一些常见方法，如下面的代码所示：

```cs

static

void

WorkingWithLists

()

{

// 创建列表并添加三个项目的简单语法

List<string

> cities = new

();

cities.Add("London"

);

cities.Add("Paris"

);

cities.Add("Milan"

);

/* 通过编译器转换的替代语法

上面的三个 Add 方法调用

List<string> cities = new()

{ "London", "Paris", "Milan" };

*/

/* 通过编译器转换的替代语法

将字符串值的数组添加到 AddRange 方法

List<string> cities = new();

cities.AddRange(new[] { "London", "Paris", "Milan" });

*/

输出("初始列表"

, cities);

WriteLine($"第一个城市是

{cities[

0

]}

。"

);

WriteLine($"最后一个城市是

{cities[cities.Count -

1

]}

。"

);

cities.Insert(0

, "悉尼"

);

输出("在索引 0 处插入悉尼后"

, cities);

cities.RemoveAt(1

);

cities.Remove("Milan"

);

Output("移除两个城市后"

, cities);

}

```

1.  在`Program.cs`的顶部，在命名空间导入之后，调用`WorkingWithLists`方法，如下面的代码所示：

```cs

WorkingWithLists();

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

初始列表

伦敦

巴黎

米兰

第一个城市是伦敦。

最后一个城市是米兰。

在索引 0 处插入悉尼后

悉尼

伦敦

巴黎

米兰

删除两个城市后

悉尼

巴黎

```

## 使用字典

让我们探索字典：

1.  在`Program.cs`中，定义一个名为`WorkingWithDictionaries`的静态方法，以说明使用字典的一些常见方法，例如查找单词定义，如下所示：

```cs

静态

void

WorkingWithDictionaries

()

{

Dictionary<string

, 字符串

> 关键字 = 新的

();

// 使用命名参数添加

关键字.Add(key: "int"

, value

: "32 位整数数据类型"

);

// 使用位置参数添加

关键字.Add("long"

, "64 位整数数据类型"

);

关键字.Add("float"

, "单精度浮点数"

);

/* 替代语法；编译器将其转换为对 Add 方法的调用

Dictionary<string, string> 关键字 = 新的()

{

{ "int", "32 位整数数据类型" },

{ "long", "64 位整数数据类型" },

{ "float", "单精度浮点数" },

}; */

/* 替代语法；编译器将其转换为对 Add 方法的调用

Dictionary<string, string> 关键字 = 新的()

{

["int"] = "32 位整数数据类型",

["long"] = "64 位整数数据类型",

["float"] = "单精度浮点数", // 最后一个逗号是可选的

}; */

输出("字典键："

, 关键字.Keys);

输出("字典值："

, 关键字.Values);

WriteLine("关键字及其定义"

);

foreach

(KeyValuePair<string

, 字符串

> 项目在

关键字)

{

WriteLine($"

{item.Key}

:

{item.Value}

"

);

}

// 使用键查找值

字符串

key = "long"

;

WriteLine($"int 的定义

{key}

是

{keywords[key]}

"

);

}

```

1.  在`Program.cs`的顶部，注释掉先前的方法调用，然后调用`WorkingWithDictionaries`方法，如下所示：

```cs

// WorkingWithLists();

WorkingWithDictionaries();

```

1.  运行代码并查看结果，如下所示：

```cs

字典键：

int

长

float

字典值：

32 位整数数据类型

64 位整数数据类型

单精度浮点数

关键字及其定义

int：32 位整数数据类型

long：64 位整数数据类型

float：单精度浮点数

long 的定义是 64 位整数数据类型

```

## 使用队列

让我们探索队列：

1.  在`Program.cs`中，定义一个名为`WorkingWithQueues`的静态方法，以说明使用队列的一些常见方法，例如，在咖啡队列中处理顾客，如下所示：

```cs

静态

void

WorkingWithQueues

()

{

Queue<string

> 咖啡 = 新的

();

coffee.Enqueue("达米尔"

); // 队列的前面

coffee.Enqueue("安德烈"

);

coffee.Enqueue("罗纳德"

);

coffee.Enqueue("阿敏"

);

coffee.Enqueue("伊琳娜"

); // 队列的后面

输出("从前到后的初始队列"

, 咖啡);

// 服务器处理队列中的下一个人

字符串

服务 = 咖啡.Dequeue();

WriteLine($"服务：

{服务}

."

);

// 服务器处理队列中的下一个人

服务 = 咖啡.Dequeue();

WriteLine($"服务：

{served}

."

);

输出("从前到后的当前队列"

, 咖啡);

WriteLine($"

{coffee.Peek()}

是下一个。"

);

输出("从前到后的当前队列"

, 咖啡);

}

```

1.  在`Program.cs`的顶部，注释掉先前的方法调用，并调用`WorkingWithQueues`方法。

1.  运行代码并查看结果，如下所示：

```cs

从前到后的初始队列

达米尔

安德烈

罗纳德

阿敏

伊琳娜

服务：达米尔。

服务：安德烈。

从前到后的当前队列

罗纳德

阿敏

伊琳娜

罗纳德是下一个。

从前到后的当前队列

罗纳德

阿敏

伊琳娜

```

1.  定义一个名为`OutputPQ`的静态方法，如下所示：

```cs

静态

void

OutputPQ

<

TElement

,

TPriority

>(

字符串

标题，

IEnumerable<(TElement Element, TPriority Priority

)> 集合)

{

WriteLine(title);

foreach

((TElement, TPriority) 项目在

集合)

{

WriteLine($"

{item.Item1}

：

{item.Item2}

"

);

}

}

```

请注意，`OutputPQ`方法是通用的。您可以指定作为`collection`传入的元组中使用的两种类型。

1.  定义一个名为`WorkingWithPriorityQueues`的静态方法，如下所示：

```cs

静态的

void

WorkingWithPriorityQueues

()

{

PriorityQueue<string

, int

> vaccine = new

();

// 添加一些人

// 1 = 高优先级的 70 多岁或健康状况不佳的人

// 2 = 中等优先级，例如中年人

// 3 = 低优先级，例如十几岁和二十几岁。

vaccine.Enqueue("Pamela"

, 1

);  // 我妈妈（70 多岁）

vaccine.Enqueue("Rebecca"

, 3

); // 我的侄女（十几岁）

vaccine.Enqueue("Juliet"

, 2

);  // 我妹妹（40 多岁）

vaccine.Enqueue("Ian"

, 1

);     // 我爸爸（70 多岁）

OutputPQ("当前接种疫苗的队列："

, vaccine.UnorderedItems);

WriteLine($"

{vaccine.Dequeue()}

已接种疫苗。"

);

WriteLine($"

{vaccine.Dequeue()}

已接种疫苗。"

);

OutputPQ("当前接种疫苗的队列："

, vaccine.UnorderedItems);

WriteLine($"

{vaccine.Dequeue()}

已接种疫苗。"

);

vaccine.Enqueue("Mark"

, 2

); // 我（40 多岁）

WriteLine($"

{vaccine.Peek()}

将是下一个接种疫苗的人。"

);

OutputPQ("当前接种疫苗的队列："

, vaccine.UnorderedItems);

}

```

1.  在`Program.cs`的顶部，注释掉以前的方法调用，并调用`WorkingWithPriorityQueues`方法。

1.  运行代码并查看结果，如下面的输出所示：

```cs

当前接种疫苗的队列：

帕梅拉：1

丽贝卡：3

朱丽叶：2

伊恩：1

帕梅拉已接种疫苗。

伊恩已接种疫苗。

当前接种疫苗的队列：

朱丽叶：2

丽贝卡：3

朱丽叶已接种疫苗。

马克将是下一个接种疫苗的人。

当前接种疫苗的队列：

马克：2

丽贝卡：3

```

## 排序集合

`List<T>`类可以通过手动调用其`Sort`方法进行排序（但请记住，每个项目的索引将发生变化）。手动对`string`值或其他内置类型的列表进行排序不需要额外的努力，但如果您创建了自己类型的集合，那么该类型必须实现一个名为`IComparable`的接口。您在*第六章*，*实现接口和继承类*中学习了如何做到这一点。

`Stack<T>`或`Queue<T>`集合不能排序，因为通常不需要这种功能；例如，您可能永远不会对入住酒店的客人队列进行排序。但有时，您可能需要对字典或集合进行排序。

有时，自动排序的集合会很有用，也就是说，当你添加和删除项目时，它会保持有序。

有多个自动排序的集合可供选择。这些排序集合之间的差异通常很微妙，但可能会影响应用程序的内存需求和性能，因此值得努力选择最合适的选项来满足您的需求。

一些常见的自动排序集合如下表所示：

| 集合 | 描述 |
| --- | --- |
| `SortedDictionary<TKey, TValue>` | 这代表了一个按键排序的键/值对集合。 |
| `SortedList<TKey, TValue>` | 这代表了一个按键排序的键/值对集合。 |
| `SortedSet<T>` | 这代表了一个维护在有序顺序中的唯一对象的集合。 |

## 更多专门的集合

还有一些其他用于特殊情况的集合。

### 使用紧凑的位值数组

`System.Collections.BitArray`集合管理一个紧凑的位值数组，这些位值被表示为布尔值，其中`true`表示位打开（值为 1），`false`表示位关闭（值为 0）。

### 使用高效的列表

`System.Collections.Generics.LinkedList<T>`集合表示一个双向链表，其中每个项目都有对其前一个和后一个项目的引用。它们在频繁在列表中间插入和删除项目的场景中提供了比`List<T>`更好的性能。在`LinkedList<T>`中，项目不必在内存中重新排列。

## 使用不可变集合

有时候你需要让一个集合成为不可变的，也就是说它的成员不能改变；也就是说，你不能添加或删除它们。

如果你导入了`System.Collections.Immutable`命名空间，那么任何实现`IEnumerable<T>`的集合都会得到六个扩展方法，用于将其转换为不可变列表、字典、哈希集等。

让我们看一个简单的例子：

1.  在`WorkingWithCollections`项目中的`Program.cs`中，导入`System.Collections.Immutable`命名空间。

1.  在`WorkingWithLists`方法中，添加语句到方法的末尾，将`cities`列表转换为不可变列表，然后向其中添加一个新的城市，如下面的代码所示：

```cs

ImmutableList<string

> immutableCities = cities.ToImmutableList();

ImmutableList<string

> newList = immutableCities.Add("里约"

);

Output("不可变城市列表："

, immutableCities);

Output("新的城市列表："

, newList);

```

1.  在`Program.cs`的顶部，注释掉之前的方法调用，并取消注释对`WorkingWithLists`方法的调用。

1.  运行代码，查看结果，注意当你在不可变列表上调用`Add`方法时，不可变城市列表不会被修改；相反，它会返回一个新的列表，其中包含了新添加的城市，如下面的输出所示：

```cs

不可变城市列表：

悉尼

巴黎

新的城市列表：

悉尼

巴黎

里约

```

**良好实践**：为了提高性能，许多应用程序在中央缓存中存储常用对象的共享副本。为了安全地允许多个线程使用这些对象，你应该让它们成为不可变的，或者使用一个并发集合类型，你可以在以下链接中了解更多信息：[`docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent`](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent)

## 集合的良好实践

假设你需要创建一个处理集合的方法。为了最大的灵活性，你可以声明输入参数为`IEnumerable<T>`，并且将方法声明为泛型，如下面的代码所示：

```cs

void

ProcessCollection

<

T

>(

IEnumerable<T> collection

)

{

// 处理集合中的项目，

// 可能使用 foreach 语句

}

```

我可以将数组、列表、队列、栈或任何其他实现`IEnumerable<T>`的东西传递给这个方法，它将处理这些项目。然而，将任何集合传递给这个方法的灵活性会带来性能成本。

`IEnumerable<T>`的性能问题之一也是它的好处之一：延迟执行，也称为惰性加载。实现这个接口的类型不必实现延迟执行，但很多类型都这样做。

但是`IEnumerable<T>`的最糟糕的性能问题是迭代必须在堆上分配一个对象。为了避免这种内存分配，你应该使用一个具体的类型来定义你的方法，如下面代码中的高亮部分所示：

```cs

void

ProcessCollection

<

T

>(

**List<T>**

collection

)

{

// 处理集合中的项目，

// 可能使用 foreach 语句

}

```

这将使用`List<T>.Enumerator GetEnumerator()`方法，该方法返回一个`struct`，而不是返回一个引用类型的`IEnumerator<T> GetEnumerator()`方法。你的代码将快两到三倍，并且需要更少的内存。与性能相关的所有建议一样，你应该在产品环境中运行性能测试来确认这种好处。你将在*第十二章*中学习如何做到这一点，*使用多任务改进性能和可伸缩性*。

# 使用范围、索引和范围

微软在.NET Core 2.1 中的一个目标是提高性能和资源使用率。实现这一目标的一个关键.NET 功能是`Span<T>`类型。

## 使用范围高效使用内存

在操作数组时，通常会创建现有子集的新副本，以便可以处理子集。这不是有效的，因为必须在内存中创建重复的对象。

如果需要处理数组的子集，请使用**span**，因为它就像是对原始数组的窗口。这在内存使用方面更有效，并提高了性能。Span 只能用于数组，而不是集合，因为内存必须是连续的。

在更详细地查看范围之前，我们需要了解一些相关的对象：索引和范围。

## 使用索引类型标识位置

C# 8.0 引入了两个功能，用于使用两个索引标识数组中项目的索引和项目范围。

您在上一个主题中学到，可以通过将整数传递到它们的索引器中来访问列表中的对象，如下面的代码所示：

```cs

int

index = 3

;

Person p = people[index]; //数组中的第四个人

char

letter = name[index]; //名字中的第四个字母

```

`Index`值类型是一种更正式的标识位置的方式，并支持从末尾计数，如下面的代码所示：

```cs

//定义相同索引的两种方式，从开头开始的 3

Index i1 = new

(value

3

); //从开头计数

Index i2 = 3

; //使用隐式 int 转换操作符

//定义相同索引的两种方式，从末尾开始的 5

Index i3 = new

(value

: 5

, fromEnd: true

);

Index i4 = ⁵

; //使用插入符操作符

```

## 使用范围类型标识范围

`Range`值类型使用`Index`值来指示其范围的开始和结束，使用其构造函数、C#语法或其静态方法，如下面的代码所示：

```cs

范围 r1 = new

(start: new

Index(3

), end: new

Index(7

));

范围 r2 = new

(start: 3

, end: 7

); //使用隐式 int 转换

范围 r3 = 3..7

; //使用 C# 8.0 或更高版本的语法

范围 r4 = Range.StartAt(3

); //从索引 3 到最后一个索引

范围 r5 = 3.

.; //从索引 3 到最后一个索引

范围 r6 = Range.EndAt(3

); //从索引 0 到索引 3

范围 r7 = ..3

; //从索引 0 到索引 3

```

已添加扩展方法到`string`值（内部使用`char`数组），`int`数组和范围，以使范围更容易使用。这些扩展方法接受范围作为参数并返回`Span<T>`。这使它们非常节省内存。

## 使用索引、范围和范围

让我们探索使用索引和范围返回范围：

1.  使用您喜欢的代码编辑器将新的控制台应用命名为`WorkingWithRanges`添加到`Chapter08`解决方案/工作区中。

1.  在 Visual Studio Code 中，选择`WorkingWithRanges`作为活动的 OmniSharp 项目。

1.  在`Program.cs`中，输入语句使用`string`类型的`Substring`方法进行比较，使用范围来提取某人姓名的部分，如下面的代码所示：

```cs

字符串

name = "Samantha Jones"

;

//使用 Substring

int

lengthOfFirst = name.IndexOf(' '

);

int

lengthOfLast = name.Length - lengthOfFirst - 1

;

字符串

firstName = name.Substring(

startIndex: 0

,

length: lengthOfFirst);

字符串

lastName = name.Substring(

startIndex: name.Length - lengthOfLast,

length: lengthOfLast);

WriteLine($"名: 

{firstName}

, 姓:

{lastName}

"

);

//使用范围

ReadOnlySpan<char

> nameAsSpan = name.AsSpan();

ReadOnlySpan<char

> firstNameSpan = nameAsSpan[0.

.lengthOfFirst];

ReadOnlySpan<char

> lastNameSpan = nameAsSpan[^lengthOfLast..⁰

];

WriteLine("名: {0}, 姓: {1}"

,

arg0: firstNameSpan.ToString(),

arg1: lastNameSpan.ToString());

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

名: Samantha, 姓: Jones

名: Samantha, 姓: Jones

```

# 使用网络资源

有时您需要处理网络资源。在.NET 中处理网络资源的最常见类型如下表所示：

| 命名空间 | 示例类型 | 描述 |
| --- | --- | --- |
| `System.Net` | `Dns`，`Uri`，`Cookie`，`WebClient`，`IPAddress` | 这些用于与 DNS 服务器、URI、IP 地址等进行交互。 |
| `System.Net` | `FtpStatusCode`，`FtpWebRequest`，`FtpWebResponse` | 这些用于与 FTP 服务器进行交互。 |
| `System.Net` | `HttpStatusCode`，`HttpWebRequest`，`HttpWebResponse` | 这些用于与 HTTP 服务器进行交互；也就是网站和服务。`System.Net.Http`中的类型更容易使用。 |
| `System.Net.Http` | `HttpClient`，`HttpMethod`，`HttpRequestMessage`，`HttpResponseMessage` | 这些用于与 HTTP 服务器进行交互；也就是网站和服务。您将在*第十六章*中学习如何使用这些内容，*构建和使用 Web 服务*。 |
| `System.Net.Mail` | `Attachment`，`MailAddress`，`MailMessage`，`SmtpClient` | 这些用于与 SMTP 服务器进行交互；也就是发送电子邮件消息。 |
| `System.Net .NetworkInformation` | `IPStatus`，`NetworkChange`，`Ping`，`TcpStatistics` | 这些用于处理低级网络协议。 |

## 处理 URI、DNS 和 IP 地址

让我们探索一些用于处理网络资源的常见类型：

1.  使用您喜欢的代码编辑器向`Chapter08`解决方案/工作区添加一个名为`WorkingWithNetworkResources`的新控制台应用程序。

1.  在 Visual Studio Code 中，选择`WorkingWithNetworkResources`作为活动的 OmniSharp 项目。

1.  在`Program.cs`的顶部，导入用于处理网络的命名空间，如下面的代码所示：

```cs

使用

System.Net; // IPHostEntry, Dns, IPAddress

```

1.  编写语句提示用户输入网站地址，然后使用`Uri`类型将其分解为其部分，包括方案（HTTP、FTP 等）、端口号和主机，如下面的代码所示：

```cs

Write("输入有效的网址："

);

string? url = ReadLine();

if

（字符串

.IsNullOrWhiteSpace(url))

{

url = "https://stackoverflow.com/search?q=securestring"

;

}

Uri uri = new

(url);

WriteLine($"URL：

{url}

"

);

WriteLine($"方案：

{uri.Scheme}

"

);

WriteLine($"端口：

{uri.Port}

"

);

WriteLine($"主机：

{uri.Host}

"

);

WriteLine($"路径：

{uri.AbsolutePath}

"

);

WriteLine($"查询：

{uri.Query}

"

);

```

为方便起见，该代码还允许用户按 ENTER 键使用示例 URL。

1.  运行代码，输入有效的网站地址或按 ENTER 键，并查看结果，如下面的输出所示：

```cs

输入有效的网址：

URL: https://stackoverflow.com/search?q=securestring

方案：https

端口：443

主机：stackoverflow.com

路径：/search

查询：?q=securestring

```

1.  添加语句以获取输入网站的 IP 地址，如下面的代码所示：

```cs

IPHostEntry entry = Dns.GetHostEntry(uri.Host);

WriteLine($"

{entry.HostName}

具有以下 IP 地址：

);

foreach

（IPAddress address in

entry.AddressList)

{

WriteLine($"

{address}

(

{address.AddressFamily}

)"

);

}

```

1.  运行代码，输入有效的网站地址或按 ENTER 键，并查看结果，如下面的输出所示：

```cs

stackoverflow.com 具有以下 IP 地址：

151.101.193.69 (InterNetwork)

151.101.129.69 (InterNetwork)

151.101.1.69 (InterNetwork)

151.101.65.69 (InterNetwork)

```

## Ping 服务器

现在，您将添加代码来 ping 一个 Web 服务器以检查其健康状况：

1.  导入命名空间以获取有关网络的更多信息，如下面的代码所示：

```cs

使用

System.Net.NetworkInformation; // Ping, PingReply, IPStatus

```

1.  添加语句以 ping 输入的网站，如下面的代码所示：

```cs

try

{

Ping ping = new

();

WriteLine("正在 ping 服务器。请稍候..."

);

PingReply reply = ping.Send(uri.Host);

WriteLine($"

{uri.Host}

被 ping 并回复：

{reply.Status}

."

);

if

(reply.Status == IPStatus.Success)

{

WriteLine("来自{0}的回复花费了{1:N0}毫秒"

,

arg0: reply.Address,

arg1：reply.RoundtripTime);

}

}

catch (Exception ex)

{

WriteLine($"

{ex.GetType().ToString()}

说

{ex.Message}

"

);

}

```

1.  运行代码，按 ENTER，并查看结果，如下面在 macOS 上的输出所示：

```cs

正在 ping 服务器。请稍候...

stackoverflow.com 被 ping 并回复：成功。

来自 151.101.193.69 的回复需要 18ms，需要 136ms

```

1.  再次运行代码，但这次输入[`google.com`](http://google.com)，如下面的输出所示：

```cs

输入有效的网址：http://google.com

URL：http://google.com

方案：http

端口：80

主机：google.com

路径：/

查询：

google.com 具有以下 IP 地址：

2a00:1450:4009:807::200e（InterNetworkV6）

216.58.204.238（InterNetwork）

正在 ping 服务器。请稍候...

google.com 被 ping 并回复：成功。

来自 2a00:1450:4009:807::200e 的回复需要 24ms

```

# 使用反射和属性

**反射**是一种编程特性，允许代码理解和操作自身。程序集由最多四部分组成：

+   **程序集元数据和清单**：名称、程序集和文件版本、引用的程序集等。

+   **类型元数据**：关于类型、它们的成员等的信息。

+   **IL 代码**：方法、属性、构造函数等的实现。

+   **嵌入式资源**（可选）：图像、字符串、JavaScript 等。

元数据包括有关您的代码的信息项。元数据是从您的代码自动生成的（例如，有关类型和成员的信息），或者使用属性应用于您的代码。

属性可以应用于多个级别：程序集，类型及其成员，如下面的代码所示：

```cs

// 一个程序集级属性

[assembly: AssemblyTitle(

"使用反射"

)

]

//一个类型级属性

[Serializable

]

公共

类

人

{

//一个成员级属性

过时的(

"已弃用：请改用 Run。"

)

]

公共

void

步行

()

{

...

```

基于属性的编程在应用程序模型中被广泛使用，如 ASP.NET Core，以启用路由、安全性和缓存等功能。

## 程序集的版本控制

.NET 中的版本号是三个数字的组合，还有两个可选的附加项。如果遵循语义版本控制的规则，这三个数字表示以下内容：

+   **主要**：破坏性更改。

+   **次要**：非破坏性更改，包括新功能，通常还包括错误修复。

+   **补丁**：非破坏性错误修复。

**良好实践**：当更新项目中已使用的 NuGet 包时，为了安全起见，您应该指定一个可选标志，以确保仅升级到最高的次要版本以避免破坏性更改，或者如果您特别谨慎并且只想接收错误修复，则升级到最高的补丁，如下面的命令所示：`Update-Package Newtonsoft.Json -ToHighestMinor`或`Update-Package Newtonsoft.Json -ToHighestPatch`。

可选地，版本可以包括这些内容：

+   **预发布**：不支持的预览版本。

+   **构建号**：每夜构建。

**良好实践**：遵循语义版本控制的规则，如下链接所述：[`semver.org`](http://semver.org)

## 读取程序集元数据

让我们探索使用属性：

1.  使用您喜欢的代码编辑器向`Chapter08`解决方案/工作区添加一个名为`WorkingWithReflection`的新控制台应用程序。

1.  在 Visual Studio Code 中，选择`WorkingWithReflection`作为活动的 OmniSharp 项目。

1.  在`Program.cs`的顶部，导入反射的命名空间，如下面的代码所示：

```cs

使用

System.Reflection; //程序集

```

1.  添加语句以获取控制台应用程序的程序集，输出其名称和位置，并获取所有程序集级属性并输出它们的类型，如下面的代码所示：

```cs

WriteLine("程序集元数据："

);

程序集？assembly = Assembly.GetEntryAssembly();

如果

（程序集是

空

)

{

WriteLine("无法获取入口程序集。"

);

返回

;

}

WriteLine($"  全名：

{assembly.FullName}

"

);

WriteLine($"  位置：

{assembly.Location}

"

);

IEnumerable<Attribute>属性= assembly.GetCustomAttributes();

WriteLine($"  程序集级别属性："

);

foreach

（在属性 a 中

属性）

{

WriteLine($"

{a.GetType()}

"

);

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

程序集元数据：

全名：WorkingWithReflection，版本=1.0.0.0，文化=中性，公钥令牌=null

位置：/Users/markjprice/Code/Chapter08/WorkingWithReflection/bin/Debug/net6.0/WorkingWithReflection.dll

程序集级别属性：

System.Runtime.CompilerServices.CompilationRelaxationsAttribute

系统。运行时。编译器。RuntimeCompatibilityAttribute

系统。诊断。DebuggableAttribute

系统。运行时。版本。TargetFrameworkAttribute

System.Reflection.AssemblyCompanyAttribute

系统。反射。AssemblyConfigurationAttribute

System.Reflection.AssemblyFileVersionAttribute

系统。反射。AssemblyInformationalVersionAttribute

系统。反射。AssemblyProductAttribute

系统。反射。AssemblyTitleAttribute

```

请注意，因为程序集的完整名称必须唯一标识程序集，它是以下内容的组合：

+   **名称**，例如，`WorkingWithReflection`

+   **版本**，例如，`1.0.0.0`

+   **文化**，例如，`中性`

+   **公钥令牌**，尽管这可以是`null`

现在我们知道一些装饰程序集的属性，我们可以专门要求它们。

1.  添加语句以获取`AssemblyInformationalVersionAttribute`和`AssemblyCompanyAttribute`类，然后输出它们的值，如下面的代码所示：

```cs

AssemblyInformationalVersionAttribute?版本= assembly

.GetCustomAttribute<AssemblyInformationalVersionAttribute>();

WriteLine($"  版本：

{version?.InformationalVersion}

"

);

AssemblyCompanyAttribute?公司= assembly

.GetCustomAttribute<AssemblyCompanyAttribute>();

WriteLine($"  公司：

{company?.Company}

"

);

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

版本：1.0.0

公司：WorkingWithReflection

```

嗯，除非您设置版本，否则默认为 1.0.0，除非您设置公司，否则默认为程序集的名称。让我们明确设置这些信息。设置这些值的传统.NET Framework 方式是在 C#源代码文件中添加属性，如下面的代码所示：

```cs

[程序集：公司(

"Packt Publishing"

)

]

[程序集：AssemblyInformationalVersion(

"1.3.0"

)

]

```

.NET 使用的 Roslyn 编译器会自动设置这些属性，因此我们无法使用旧的方式。相反，它们必须在项目文件中设置。

1.  编辑`WorkingWithReflection.csproj`项目文件，添加版本和公司元素，如下面标记中所示：

```cs

<Project Sdk="Microsoft.NET.Sdk"

<PropertyGroup>

<OutputType>Exe</OutputType>

<TargetFramework>net6.0

</TargetFramework>

<Nullable>enable</Nullable>

<ImplicitUsings>enable</ImplicitUsings>

**<版本>**

**6.3.12**

**</版本>**

**<Company>Packt Publishing</Company>**

</PropertyGroup>

</Project>

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

版本：6.3.12

公司：Packt Publishing

```

## 创建自定义属性

您可以通过从`Attribute`类继承来定义自己的属性：

1.  将一个名为`CoderAttribute.cs`的类文件添加到您的项目中。

1.  定义一个属性类，可以用两个属性装饰类或方法，用于存储编码者的名称和他们最后修改某些代码的日期，如下面的代码所示：

```cs

namespace

Packt.Shared

;

[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method,

AllowMultiple = true)

]

公共

类

CoderAttribute

：属性

{

公共

字符串

Coder {获取

; set

; }

公共

日期时间 LastModified {获取

; 设置

; }

公共

CoderAttribute

(

字符串

编码者，

字符串

lastModified

)

{

Coder = coder;

LastModified = DateTime.Parse(lastModified);

}

}

```

1.  在`Program.cs`中，导入一些命名空间，如下面的代码所示：

```cs

使用

System.Runtime.CompilerServices; // CompilerGeneratedAttribute

使用

Packt.Shared; // CoderAttribute

```

1.  在`Program.cs`的底部，添加一个带有方法的类，并使用`Coder`属性装饰该方法，其中包含有关两个编码器的数据，如下面的代码所示：

```cs

类

动物

{

[Coder（

“Mark Price”

，

“2021 年 8 月 22 日”

）

]

[Coder（

“Johnni Rasmussen”

，

“2021 年 9 月 13 日”

）

]

公共

空

说

（）

{

WriteLine（“Woof...”

）;

}

}

```

1.  在`Program.cs`中，在`Animal`类的上面，添加代码以获取类型，枚举其成员，读取这些成员上的任何`Coder`属性，并输出信息，如下面的代码所示：

```cs

WriteLine（）;

WriteLine（$“*类型：”

);

Type[] types = assembly.GetTypes（）;

foreach

（Type type in

types）

{

WriteLine（）;

WriteLine（$“类型：

{type.FullName}

“

）;

MemberInfo[] members = type.GetMembers（）;

foreach

（MemberInfo member in

成员）

{

WriteLine（“{0}：{1}（{2}）”）;

，

arg0：member.MemberType，

arg1：member.Name，

arg2：member.DeclaringType？.Name）;

IOrderedEnumerable<CoderAttribute> coders =

member.GetCustomAttributes<CoderAttribute>()

.OrderByDescending（c => c.LastModified）;

foreach

（CoderAttribute coder in

coders）

{

WriteLine（“->由{0}于{1}修改”

，

coder.Coder，coder.LastModified.ToShortDateString（））;

}

}

}

```

1.  运行代码并查看结果，如下面的部分输出所示：

```cs

*类型：

...

类型：Animal

方法：Speak（Animal）

->

由 Johnni Rasmussen 于 2021 年 9 月 13 日修改

->

由 Mark Price 于 2021 年 8 月 22 日修改

方法：GetType（Object）

方法：ToString（Object）

方法：Equals（Object）

方法：GetHashCode（Object）

构造函数：.ctor（Program）

...

类型：<Program>$+<>c

方法：GetType（Object）

方法：ToString（Object）

方法：Equals（Object）

方法：GetHashCode（Object）

构造函数：.ctor（<>c）

字段：<>9（<>c）

字段：<>9__0_0（<>c）

```

`<Program>$+<>c`类型是什么？

这是一个编译器生成的**显示类**。`<>`表示编译器生成，`c`表示显示类。它们是编译器的未记录的实现细节，随时可能更改。您可以忽略它们，因此作为一个可选的挑战，添加语句到您的控制台应用程序，通过跳过使用`CompilerGeneratedAttribute`装饰的类型来过滤编译器生成的类型。

## 使用反射做更多事情

这只是反射可以实现的一小部分。我们只使用反射从我们的代码中读取元数据。反射还可以做到以下事情：

+   **动态加载当前未引用的程序集**：[`docs.microsoft.com/en-us/dotnet/standard/assembly/unloadability`](https://docs.microsoft.com/en-us/dotnet/standard/assembly/unloadability)

+   **动态执行代码**：[`docs.microsoft.com/en-us/dotnet/api/system.reflection.methodbase.invoke`](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.methodbase.invoke)

+   **动态生成新代码和程序集**：[`docs.microsoft.com/en-us/dotnet/api/system.reflection.emit.assemblybuilder`](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.emit.assemblybuilder)

# 处理图像

ImageSharp 是一个第三方跨平台的 2D 图形库。当.NET Core 1.0 正在开发时，社区对缺少用于处理 2D 图像的`System.Drawing`命名空间发出了负面反馈。

**ImageSharp**项目的开始是为了填补现代.NET 应用程序的这一空白。

在他们对`System.Drawing`的官方文档中，微软表示：“不建议在 Windows 或 ASP.NET 服务中使用`System.Drawing`命名空间，因为它不受支持，也不跨平台。建议使用 ImageSharp 和 SkiaSharp 作为替代方案。”

让我们看看 ImageSharp 可以实现什么：

1.  使用您喜欢的代码编辑器将新的控制台应用程序命名为`WorkingWithImages`添加到`Chapter08`解决方案/工作区中。

1.  在 Visual Studio Code 中，选择`WorkingWithImages`作为活动的 OmniSharp 项目。

1.  创建一个`images`文件夹，并从以下链接下载九张图片：[`github.com/markjprice/cs10dotnet6/tree/master/Assets/Categories`](https://github.com/markjprice/cs10dotnet6/tree/master/Assets/Categories)

1.  为`SixLabors.ImageSharp`添加包引用，如下所示：

```cs

<ItemGroup>

<PackageReference Include="SixLabors.ImageSharp"

Version="1.0.3"

/>

</ ItemGroup>

```

1.  构建`WorkingWithImages`项目。

1.  在`Program.cs`的顶部，导入一些用于处理图像的命名空间，如下所示：

```cs

使用

SixLabors.ImageSharp;

使用

SixLabors.ImageSharp.Processing;

```

1.  在`Program.cs`中，输入语句将图像文件夹中的所有文件转换为十分之一大小的灰度缩略图，如下所示：

```cs

字符串

imagesFolder = Path.Combine（

Environment.CurrentDirectory，"images"

）;

IEnumerable<string

> 图像=

Directory.EnumerateFiles（imagesFolder）;

foreach

（字符串

imagePath 中

图像）

{

字符串

thumbnailPath = Path.Combine（

Environment.CurrentDirectory，"images"

，

Path.GetFileNameWithoutExtension（imagePath）

+“-thumbnail”

+ Path.GetExtension（imagePath））;

使用

（Image image = Image.Load（imagePath））

{

image.Mutate（x => x.Resize（image.Width / 10

，image.Height / 10

）;

image.Mutate（x => x.Grayscale（））;

image.Save（thumbnailPath）;

}

}

WriteLine（"图像处理完成。查看图像文件夹。"

）;

```

1.  运行代码。

1.  在文件系统中，打开`images`文件夹，并注意以字节为单位要小得多的灰度缩略图，如*图 8.1*所示：![包含应用程序描述的图片](img/Image00085.jpg)

图 8.1：处理后的图像

ImageSharp 还有用于以编程方式绘制图像和在 Web 上处理图像的 NuGet 包，如下列表所示：

+   `SixLabors.ImageSharp.Drawing`

+   `SixLabors.ImageSharp.Web`

# 国际化您的代码

国际化是使您的代码能够在全球范围内正确运行的过程。它有两个部分：**全球化**和**本地化**。

全球化是关于编写代码以适应多种语言和区域组合的。语言和区域的组合称为文化。对于您的代码来说，知道语言和区域都很重要，因为例如，尽管魁北克和巴黎都使用法语，但日期和货币格式却不同。

有关所有文化组合的**国际标准化组织**（**ISO**）代码。例如，在代码`da-DK`中，`da`表示丹麦语，`DK`表示丹麦地区，在代码`fr-CA`中，`fr`表示法语，`CA`表示加拿大地区。

ISO 不是一个首字母缩写词。 ISO 是希腊词*isos*（意思是相等）的引用。

本地化是关于定制用户界面以支持语言的，例如，将按钮的标签更改为 Close（`en`）或 Fermer（`fr`）。由于本地化更多关于语言，因此它并不总是需要了解区域，尽管具有讽刺意味的是，标准化（`en-US`）和标准化（`en-GB`）似乎表明了相反的情况。

## 检测和更改当前文化

国际化是一个庞大的主题，已经写了几千页的书。在本节中，您将简要介绍使用`System.Globalization`命名空间中的`CultureInfo`类型的基础知识。

让我们写一些代码：

1.  使用您喜欢的代码编辑器将新的控制台应用程序命名为`Internationalization`添加到`Chapter08`解决方案/工作区中。

1.  在 Visual Studio Code 中，将`Internationalization`选择为活动的 OmniSharp 项目。

1.  在`Program.cs`的顶部，导入使用全球化类型的命名空间，如下所示：

```cs

使用

System.Globalization; // CultureInfo

```

1.  添加语句以获取当前的全球化和本地化文化，并输出有关它们的一些信息，然后提示用户输入新的文化代码，并显示它如何影响常见值的格式，如日期和货币，如下面的代码所示：

```cs

CultureInfo globalization = CultureInfo.CurrentCulture;

CultureInfo localization = CultureInfo.CurrentUICulture;

WriteLine("当前的全球化文化是{0}：{1}"

，

globalization.Name, globalization.DisplayName);

WriteLine("当前的本地化文化是{0}：{1}"

，

localization.Name, localization.DisplayName);

WriteLine();

WriteLine("en-US：英语（美国）"

);

WriteLine("da-DK：丹麦语（丹麦）"

);

WriteLine("fr-CA：法语（加拿大）"

);

Write("输入 ISO 文化代码："

);

字符串

？newCulture= ReadLine();

如果

(!字符串

.IsNullOrEmpty(newCulture))

{

CultureInfo ci = new

（newCulture）;

//更改当前的文化

CultureInfo.CurrentCulture = ci;

CultureInfo.CurrentUICulture = ci;

}

WriteLine();

Write("输入您的名字："

);

字符串

？名字= ReadLine();

Write("输入您的出生日期："

);

字符串

？dob= ReadLine();

Write("输入您的工资："

);

字符串

？工资= ReadLine();

DateTime date = DateTime.Parse(dob);

int

分钟=（int

)DateTime.Today.Subtract(date).TotalMinutes;

十进制

earns=十进制

.Parse(工资);

WriteLine(

"{0}出生于{1:dddd}，已经{2:N0}分钟，工资为{3:C}"

，

名字，日期，分钟，赚取；

```

当您运行应用程序时，它会自动将其线程设置为使用操作系统的文化。我在英国伦敦运行我的代码，所以线程被设置为英语（英国）。

代码提示用户输入替代的 ISO 代码。这允许您的应用程序在运行时替换默认的文化。

然后应用标准格式代码来输出星期几，使用格式代码`dddd`；使用格式代码`N0`输出带千位分隔符的分钟数；和带有货币符号的工资。这些会根据线程的文化自动调整。

1.  运行代码并输入`en-GB`作为 ISO 代码，然后输入一些样本数据，包括一个符合英国英语格式的日期，如下面的输出所示：

```cs

输入 ISO 文化代码：en-GB

输入您的名字：爱丽丝

输入您的出生日期：30/3/1967

输入您的工资：23500

爱丽丝出生于星期四，已经 25,469,280 分钟，工资为

£23,500.00

```

如果您输入`en-US`而不是`en-GB`，那么您必须使用月/日/年输入日期。

1.  重新运行代码并尝试不同的文化，例如丹麦的丹麦语，如下面的输出所示：

```cs

输入 ISO 文化代码：da-DK

输入您的名字：米克尔

输入您的出生日期：12/3/1980

输入您的工资：340000

米克尔出生于星期三，已经 18,656,640 分钟，工资为 340,000.00 kr。

```

在这个例子中，只有日期和工资被全球化为丹麦语。其余的文本是硬编码为英语。本书目前不包括如何将文本从一种语言翻译成另一种语言。如果您希望我在下一版中包含这一点，请告诉我。

**良好的实践**：在开始编码之前考虑您的应用程序是否需要国际化，并为此做好计划！记录下用户界面中需要本地化的所有文本片段。考虑所有需要全球化的数据（日期格式、数字格式和排序文本行为）。

# 练习和探索

通过回答一些问题来测试您的知识和理解，进行一些实践，并深入研究本章主题。

## 练习 8.1-测试您的知识

使用网络回答以下问题：

1.  在`string`变量中最多可以存储多少个字符？

1.  何时以及为什么应该使用`SecureString`类型？

1.  何时适合使用`StringBuilder`类？

1.  何时应该使用`LinkedList<T>`类？

1.  何时应该使用`SortedDictionary<T>`类而不是`SortedList<T>`类？

1.  威尔士语的 ISO 文化代码是什么？

1.  本地化、全球化和国际化之间有什么区别？

1.  在正则表达式中，$代表什么？

1.  在正则表达式中，如何表示数字？

1.  为什么*不*应该使用官方标准的电子邮件地址来创建用于验证用户电子邮件地址的正则表达式？

## 练习 8.2 – 练习正则表达式

在`Chapter08`解决方案/工作空间中，创建一个名为`Exercise02`的控制台应用程序，提示用户输入一个正则表达式，然后提示用户输入一些内容，并比较两者是否匹配，直到用户按下*Esc*，如下所示的输出：

```cs

默认的正则表达式检查是否至少有一个数字。

输入一个正则表达式（或按 Enter 键使用默认值）：^[a-z]+$

输入一些内容：apples

apples 匹配 ^[a-z]+$? True

按 ESC 键结束或按任意键重试。

输入一个正则表达式（或按 Enter 键使用默认值）：^[a-z]+$

输入一些内容：abc123xyz

abc123xyz 匹配 ^[a-z]+$? False

按 ESC 键结束或按任意键重试。

```

## 练习 8.3 – 练习编写扩展方法

在`Chapter08`解决方案/工作空间中，创建一个名为`Exercise03`的类库，定义扩展方法，扩展`BigInteger`和`int`等数字类型，其中包含一个名为`ToWords`的方法，返回描述该数字的`string`；例如，`18,000,000`将是一千八百万，`18,456,002,032,011,000,007`将是一千八百四十六万亿，二千零三十二亿，十一万，七。

您可以在以下链接中了解更多关于大数字的名称：[`en.wikipedia.org/wiki/Names_of_large_numbers`](https://en.wikipedia.org/wiki/Names_of_large_numbers)

## 练习 8.4 – 探索主题

使用以下页面上的链接了解本章涵盖的主题的更多细节：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-8---working-with-common-net-types`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-8---working-with-common-net-types)

# 摘要

在本章中，您探讨了一些用于存储和操作数字、日期和时间以及文本的类型选择，包括正则表达式，以及用于存储多个项目的集合；使用索引、范围和跨度；使用了一些网络资源；反思了代码和属性；使用了一个微软推荐的第三方库来操作图像；并学会了如何国际化您的代码。

在下一章中，我们将管理文件和流，对文本进行编码和解码，并执行序列化。
