# 第八章：使用常见的 .NET 类型

本章介绍了一些随 .NET 一起提供的常见类型。这些类型包括用于操作数字、文本、集合、网络访问、反射和属性的类型；改进与跨度、索引和范围的工作；处理图像；以及国际化。

本章涵盖以下主题：

+   处理数字

+   处理文本

+   处理日期和时间

+   使用正则表达式进行模式匹配

+   在集合中存储多个对象

+   处理跨度、索引和范围

+   处理网络资源

+   使用反射和属性

+   处理图像

+   国际化你的代码

# 处理数字

最常见的数据类型之一是数字。.NET 中处理数字的最常见类型如下表所示：

| 命名空间 | 示例类型 | 描述 |
| --- | --- | --- |
| `System` | `SByte`, `Int16`, `Int32`, `Int64` | 整数；即零和正负整数 |
| `System` | `Byte`, `UInt16`, `UInt32`, `UInt64` | 基数；即零和正整数 |
| `System` | `Half`, `Single`, `Double` | 实数；即浮点数 |
| `System` | `Decimal` | 精确实数；即用于科学、工程或金融场景 |
| `System.Numerics` | `BigInteger`, `Complex`, `Quaternion` | 任意大整数、复数和四元数 |

.NET 自 .NET Framework 1.0 起就拥有 32 位浮点数和 64 位双精度类型。IEEE 754 标准还定义了一个 16 位浮点标准。机器学习和其他算法将从这种更小、精度更低的数字类型中受益，因此微软在 .NET 5 及更高版本中引入了 `System.Half` 类型。

目前，C# 语言未定义 `half` 别名，因此必须使用 .NET 类型 `System.Half`。未来可能会发生变化。

## 处理大整数

.NET 类型中能用 C# 别名表示的最大整数大约是十八万五千亿，存储在无符号 `long` 整数中。但如果需要存储更大的数字呢？

让我们探索数字：

1.  使用您喜欢的代码编辑器创建一个名为 `Chapter08` 的新解决方案/工作区。

1.  添加一个控制台应用程序项目，如下表所示：

    1.  项目模板：**控制台应用程序** / `console`

    1.  工作区/解决方案文件和文件夹：`Chapter08`

    1.  项目文件和文件夹：`WorkingWithNumbers`

1.  在`Program.cs`中，删除现有语句并添加一条语句以导入`System.Numerics`，如下所示：

    ```cs
    using System.Numerics; 
    ```

1.  添加语句以输出 `ulong` 类型的最大值，以及使用 `BigInteger` 表示的具有 30 位数字的数，如下所示：

    ```cs
    WriteLine("Working with large integers:");
    WriteLine("-----------------------------------");
    ulong big = ulong.MaxValue;
    WriteLine($"{big,40:N0}");
    BigInteger bigger =
      BigInteger.Parse("123456789012345678901234567890");
    WriteLine($"{bigger,40:N0}"); 
    ```

    格式代码中的 `40` 表示右对齐 40 个字符，因此两个数字都排列在右侧边缘。`N0` 表示使用千位分隔符且小数点后为零。

1.  运行代码并查看结果，如下所示：

    ```cs
    Working with large integers:
    ----------------------------------------
                  18,446,744,073,709,551,615
     123,456,789,012,345,678,901,234,567,890 
    ```

## 处理复数

复数可以表示为*a + bi*，其中*a*和*b*是实数，*i*是虚数单位，其中*i*² *= −1*。如果实部*a*为零，则它是纯虚数。如果虚部*b*为零，则它是实数。

复数在许多**STEM**（**科学、技术、工程和数学**）研究领域具有实际应用。此外，它们是通过分别添加被加数的实部和虚部来相加的；考虑这一点：

```cs
(a + bi) + (c + di) = (a + c) + (b + d)i 
```

让我们探索复数：

1.  在`Program.cs`中，添加语句以添加两个复数，如下列代码所示：

    ```cs
    WriteLine("Working with complex numbers:");
    Complex c1 = new(real: 4, imaginary: 2);
    Complex c2 = new(real: 3, imaginary: 7);
    Complex c3 = c1 + c2;
    // output using default ToString implementation
    WriteLine($"{c1} added to {c2} is {c3}");
    // output using custom format
    WriteLine("{0} + {1}i added to {2} + {3}i is {4} + {5}i",
      c1.Real, c1.Imaginary, 
      c2.Real, c2.Imaginary,
      c3.Real, c3.Imaginary); 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    Working with complex numbers:
    (4, 2) added to (3, 7) is (7, 9)
    4 + 2i added to 3 + 7i is 7 + 9i 
    ```

## 理解四元数

四元数是一种扩展复数系统的数字系统。它们构成了一个四维的关联范数除法代数，覆盖实数，因此也是一个域。

嗯？是的，我知道。我也不明白。别担心，我们不会用它们来编写任何代码！可以说，它们擅长描述空间旋转，因此视频游戏引擎使用它们，许多计算机模拟和飞行控制系统也是如此。

# 处理文本

变量的另一种最常见类型是文本。.NET 中最常见的处理文本的类型如下表所示：

| 命名空间 | 类型 | 描述 |
| --- | --- | --- |
| `System` | `Char` | 存储单个文本字符 |
| `System` | `String` | 存储多个文本字符 |
| `System.Text` | `StringBuilder` | 高效地操作字符串 |
| `System.Text.RegularExpressions` | `Regex` | 高效地匹配字符串模式 |

## 获取字符串长度

让我们探讨一下处理文本时的一些常见任务；例如，有时您需要找出存储在`string`变量中的文本片段的长度：

1.  使用您偏好的代码编辑器，在`Chapter08`解决方案/工作区中添加一个名为`WorkingWithText`的新控制台应用：

    1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择。

    1.  在 Visual Studio Code 中，选择`WorkingWithText`作为活动的 OmniSharp 项目。

1.  在`WorkingWithText`项目中，在`Program.cs`文件里，添加语句定义一个变量来存储城市伦敦的名称，然后将其名称和长度写入控制台，如下列代码所示：

    ```cs
    string city = "London";
    WriteLine($"{city} is {city.Length} characters long."); 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    London is 6 characters long. 
    ```

## 获取字符串的字符

`string`类内部使用`char`数组来存储文本。它还有一个索引器，这意味着我们可以使用数组语法来读取其字符。数组索引从零开始，因此第三个字符将在索引 2 处。

让我们看看这如何实际操作：

1.  添加一条语句，以写出`string`变量中第一和第三位置的字符，如下列代码所示：

    ```cs
    WriteLine($"First char is {city[0]} and third is {city[2]}."); 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    First char is L and third is n. 
    ```

## 分割字符串

有时，您需要根据某个字符（如逗号）分割文本：

1.  添加语句以定义一个包含逗号分隔的城市名称的单个`字符串`变量，然后使用`Split`方法并指定你希望将逗号作为分隔符，接着枚举返回的`字符串`值数组，如下所示：

    ```cs
    string cities = "Paris,Tehran,Chennai,Sydney,New York,Medellín"; 
    string[] citiesArray = cities.Split(',');
    WriteLine($"There are {citiesArray.Length} items in the array.");
    foreach (string item in citiesArray)
    {
      WriteLine(item);
    } 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    There are 6 items in the array.
    Paris 
    Tehran 
    Chennai
    Sydney
    New York
    Medellín 
    ```

本章稍后，你将学习如何处理更复杂的场景。

## 获取字符串的一部分

有时，你需要获取文本的一部分。`IndexOf`方法有九个重载，它们返回指定`字符`或`字符串`在`字符串`中的索引位置。`Substring`方法有两个重载，如下所示：

+   `Substring(startIndex, length)`：返回从`startIndex`开始并包含接下来`length`个字符的子字符串。

+   `Substring(startIndex)`：返回从`startIndex`开始并包含所有字符直到字符串末尾的子字符串。

让我们来看一个简单的例子：

1.  添加语句以在`字符串`变量中存储一个人的全名，其中名字和姓氏之间有一个空格字符，找到空格的位置，然后提取名字和姓氏作为两个部分，以便它们可以以不同的顺序重新组合，如下所示：

    ```cs
    string fullName = "Alan Jones";
    int indexOfTheSpace = fullName.IndexOf(' ');
    string firstName = fullName.Substring(
      startIndex: 0, length: indexOfTheSpace);
    string lastName = fullName.Substring(
      startIndex: indexOfTheSpace + 1);
    WriteLine($"Original: {fullName}");
    WriteLine($"Swapped: {lastName}, {firstName}"); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Original: Alan Jones
    Swapped: Jones, Alan 
    ```

如果初始全名的格式不同，例如`"姓氏, 名字"`，那么代码将需要有所不同。作为可选练习，尝试编写一些语句，将输入`"Jones, Alan"`转换为`"Alan Jones"`。

## 检查字符串内容

有时，你需要检查一段文本是否以某些字符开始或结束，或者是否包含某些字符。你可以使用名为`StartsWith`、`EndsWith`和`Contains`的方法来实现这一点：

1.  添加语句以存储一个`字符串`值，然后检查它是否以或包含几个不同的`字符串`值，如下所示：

    ```cs
    string company = "Microsoft";
    bool startsWithM = company.StartsWith("M"); 
    bool containsN = company.Contains("N");
    WriteLine($"Text: {company}");
    WriteLine($"Starts with M: {startsWithM}, contains an N: {containsN}"); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Text: Microsoft
    Starts with M: True, contains an N: False 
    ```

## 连接、格式化及其他字符串成员

还有许多其他的`字符串`成员，如下表所示：

| 成员 | 描述 |
| --- | --- |
| `修剪`，`TrimStart`，`TrimEnd` | 这些方法从开头和/或结尾修剪空格、制表符和回车等空白字符。 |
| `ToUpper`，`ToLower` | 这些方法将所有字符转换为大写或小写。 |
| `插入`，`移除` | 这些方法用于插入或移除某些文本。 |
| `替换` | 这会将某些文本替换为其他文本。 |
| `string.Empty` | 这可以用来代替每次使用空的双引号(`""`)字面量`字符串`值时分配内存。 |
| `string.Concat` | 这会将两个`字符串`变量连接起来。当在`字符串`操作数之间使用时，+ 运算符执行等效操作。 |
| `string.Join` | 这会将一个或多个`字符串`变量与每个变量之间的字符连接起来。 |
| `string.IsNullOrEmpty` | 这检查`字符串`变量是否为`null`或空。 |
| `string.IsNullOrWhitespace` | 这检查`字符串`变量是否为`null`或空白；即，任意数量的水平和垂直空白字符的混合，例如，制表符、空格、回车、换行等。 |
| `string.Format` | 输出格式化`字符串`值的另一种方法，使用定位参数而不是命名参数。 |

前面提到的一些方法是静态方法。这意味着该方法只能从类型调用，而不能从变量实例调用。在前面的表格中，我通过在它们前面加上`string.`来指示静态方法，例如`string.Format`。

让我们探索一些这些方法：

1.  添加语句以使用`Join`方法将字符串值数组重新组合成带有分隔符的单个字符串变量，如下所示：

    ```cs
    string recombined = string.Join(" => ", citiesArray); 
    WriteLine(recombined); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Paris => Tehran => Chennai => Sydney => New York => Medellín 
    ```

1.  添加语句以使用定位参数和插值字符串格式化语法来输出相同的三个变量两次，如下所示：

    ```cs
    string fruit = "Apples"; 
    decimal price =  0.39M; 
    DateTime when = DateTime.Today;
    WriteLine($"Interpolated:  {fruit} cost {price:C} on {when:dddd}."); 
    WriteLine(string.Format("string.Format: {0} cost {1:C} on {2:dddd}.",
      arg0: fruit, arg1: price, arg2: when)); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Interpolated:  Apples cost £0.39 on Thursday. 
    string.Format: Apples cost £0.39 on Thursday. 
    ```

请注意，我们可以简化第二条语句，因为`WriteLine`支持与`string.Format`相同的格式代码，如下所示：

```cs
WriteLine("WriteLine: {0} cost {1:C} on {2:dddd}.",
  arg0: fruit, arg1: price, arg2: when); 
```

## 高效构建字符串

您可以使用`String.Concat`方法或简单的`+`运算符将两个字符串连接起来以创建新的`字符串`。但这两种选择都是不良实践，因为.NET 必须在内存中创建一个全新的`字符串`。

如果您只是添加两个`字符串`值，这可能不明显，但如果您在循环中进行连接，并且迭代次数很多，它可能会对性能和内存使用产生显著的负面影响。在*第十二章*，*使用多任务提高性能和可扩展性*中，您将学习如何使用`StringBuilder`类型高效地连接`字符串`变量。

# 处理日期和时间

在数字和文本之后，接下来最常处理的数据类型是日期和时间。这两种主要类型如下：

+   `DateTime`：表示一个固定时间点的日期和时间值。

+   `TimeSpan`：表示一段时间。

这两种类型通常一起使用。例如，如果您从一个`DateTime`值中减去另一个，结果是一个`TimeSpan`。如果您将一个`TimeSpan`添加到`DateTime`，则结果是一个`DateTime`值。

## 指定日期和时间值

创建日期和时间值的常见方法是分别为日期和时间组件（如日和小时）指定单独的值，如下表所述：

| 日期/时间参数 | 值范围 |
| --- | --- |
| `年` | 1 到 9999 |
| `月` | 1 到 12 |
| `日` | 1 到该月的天数 |
| `小时` | 0 到 23 |
| `分钟` | 0 到 59 |
| `秒` | 0 到 59 |

另一种方法是提供一个`string`值进行解析，但这可能会根据线程的默认文化被误解。例如，在英国，日期指定为日/月/年，而在美国，日期指定为月/日/年。

让我们看看你可能想要如何处理日期和时间：

1.  使用你偏好的代码编辑器，在`Chapter08`解决方案/工作区中添加一个名为`WorkingWithTime`的新控制台应用。

1.  在 Visual Studio Code 中，选择`WorkingWithTime`作为活动 OmniSharp 项目。

1.  在`Program.cs`中，删除现有语句，然后添加语句以初始化一些特殊的日期/时间值，如以下代码所示：

    ```cs
    WriteLine("Earliest date/time value is: {0}",
      arg0: DateTime.MinValue);
    WriteLine("UNIX epoch date/time value is: {0}",
      arg0: DateTime.UnixEpoch);
    WriteLine("Date/time value Now is: {0}",
      arg0: DateTime.Now);
    WriteLine("Date/time value Today is: {0}",
      arg0: DateTime.Today); 
    ```

1.  运行代码并记录结果，如以下输出所示：

    ```cs
    Earliest date/time value is: 01/01/0001 00:00:00
    UNIX epoch date/time value is: 01/01/1970 00:00:00
    Date/time value Now is: 23/04/2021 14:14:54
    Date/time value Today is: 23/04/2021 00:00:00 
    ```

1.  添加语句以定义 2021 年的圣诞节（如果这已过去，则使用未来的一年），并以多种方式展示，如以下代码所示：

    ```cs
    DateTime christmas = new(year: 2021, month: 12, day: 25);
    WriteLine("Christmas: {0}",
      arg0: christmas); // default format
    WriteLine("Christmas: {0:dddd, dd MMMM yyyy}",
      arg0: christmas); // custom format
    WriteLine("Christmas is in month {0} of the year.",
      arg0: christmas.Month);
    WriteLine("Christmas is day {0} of the year.",
      arg0: christmas.DayOfYear);
    WriteLine("Christmas {0} is on a {1}.",
      arg0: christmas.Year,
      arg1: christmas.DayOfWeek); 
    ```

1.  运行代码并记录结果，如以下输出所示：

    ```cs
    Christmas: 25/12/2021 00:00:00
    Christmas: Saturday, 25 December 2021
    Christmas is in month 12 of the year.
    Christmas is day 359 of the year.
    Christmas 2021 is on a Saturday. 
    ```

1.  添加语句以执行与圣诞节相关的加法和减法，如以下代码所示：

    ```cs
    DateTime beforeXmas = christmas.Subtract(TimeSpan.FromDays(12));
    DateTime afterXmas = christmas.AddDays(12);
    WriteLine("12 days before Christmas is: {0}",
      arg0: beforeXmas);
    WriteLine("12 days after Christmas is: {0}",
      arg0: afterXmas);
    TimeSpan untilChristmas = christmas - DateTime.Now;
    WriteLine("There are {0} days and {1} hours until Christmas.",
      arg0: untilChristmas.Days,
      arg1: untilChristmas.Hours);
    WriteLine("There are {0:N0} hours until Christmas.",
      arg0: untilChristmas.TotalHours); 
    ```

1.  运行代码并记录结果，如以下输出所示：

    ```cs
    12 days before Christmas is: 13/12/2021 00:00:00
    12 days after Christmas is: 06/01/2022 00:00:00
    There are 245 days and 9 hours until Christmas.
    There are 5,890 hours until Christmas. 
    ```

1.  添加语句以定义圣诞节那天你的孩子们可能醒来打开礼物的时刻，并以多种方式展示，如以下代码所示：

    ```cs
    DateTime kidsWakeUp = new(
      year: 2021, month: 12, day: 25, 
      hour: 6, minute: 30, second: 0);
    WriteLine("Kids wake up on Christmas: {0}",
      arg0: kidsWakeUp);
    WriteLine("The kids woke me up at {0}",
      arg0: kidsWakeUp.ToShortTimeString()); 
    ```

1.  运行代码并记录结果，如以下输出所示：

    ```cs
    Kids wake up on Christmas: 25/12/2021 06:30:00
    The kids woke me up at 06:30 
    ```

## 全球化与日期和时间

当前文化控制日期和时间的解析方式：

1.  在`Program.cs`顶部，导入`System.Globalization`命名空间。

1.  添加语句以显示用于显示日期和时间值的当前文化，然后解析美国独立日并以多种方式展示，如以下代码所示：

    ```cs
    WriteLine("Current culture is: {0}",
      arg0: CultureInfo.CurrentCulture.Name);
    string textDate = "4 July 2021";
    DateTime independenceDay = DateTime.Parse(textDate);
    WriteLine("Text: {0}, DateTime: {1:d MMMM}",
      arg0: textDate,
      arg1: independenceDay);
    textDate = "7/4/2021";
    independenceDay = DateTime.Parse(textDate);
    WriteLine("Text: {0}, DateTime: {1:d MMMM}",
      arg0: textDate,
      arg1: independenceDay);
    independenceDay = DateTime.Parse(textDate,
      provider: CultureInfo.GetCultureInfo("en-US"));
    WriteLine("Text: {0}, DateTime: {1:d MMMM}",
      arg0: textDate,
      arg1: independenceDay); 
    ```

1.  运行代码并记录结果，如以下输出所示：

    ```cs
    Current culture is: en-GB
    Text: 4 July 2021, DateTime: 4 July
    Text: 7/4/2021, DateTime: 7 April
    Text: 7/4/2021, DateTime: 4 July 
    ```

    在我的电脑上，当前文化是英式英语。如果给定日期为 2021 年 7 月 4 日，则无论当前文化是英式还是美式，都能正确解析。但如果日期给定为 7/4/2021，则会被错误解析为 4 月 7 日。你可以通过在解析时指定正确的文化作为提供者来覆盖当前文化，如上文第三个示例所示。

1.  添加语句以循环从 2020 年到 2025 年，显示该年是否为闰年以及二月有多少天，然后展示圣诞节和独立日是否在夏令时期间，如以下代码所示：

    ```cs
    for (int year = 2020; year < 2026; year++)
    {
      Write($"{year} is a leap year: {DateTime.IsLeapYear(year)}. ");
      WriteLine("There are {0} days in February {1}.",
        arg0: DateTime.DaysInMonth(year: year, month: 2), arg1: year);
    }
    WriteLine("Is Christmas daylight saving time? {0}",
      arg0: christmas.IsDaylightSavingTime());
    WriteLine("Is July 4th daylight saving time? {0}",
      arg0: independenceDay.IsDaylightSavingTime()); 
    ```

1.  运行代码并记录结果，如以下输出所示：

    ```cs
    2020 is a leap year: True. There are 29 days in February 2020.
    2021 is a leap year: False. There are 28 days in February 2021.
    2022 is a leap year: False. There are 28 days in February 2022.
    2023 is a leap year: False. There are 28 days in February 2023.
    2024 is a leap year: True. There are 29 days in February 2024.
    2025 is a leap year: False. There are 28 days in February 2025.
    Is Christmas daylight saving time? False
    Is July 4th daylight saving time? True 
    ```

## 仅处理日期或时间

.NET 6 引入了一些新类型，用于仅处理日期值或时间值，分别名为 `DateOnly` 和 `TimeOnly`。这些类型比使用时间部分为零的 `DateTime` 值来存储仅日期值更好，因为它们类型安全且避免了误用。`DateOnly` 也更适合映射到数据库列类型，例如 SQL Server 中的 `date` 列。`TimeOnly` 适合设置闹钟和安排定期会议或活动，并映射到 SQL Server 中的 `time` 列。

让我们用它们来为英国女王策划一场派对：

1.  添加语句以定义女王的生日及派对开始时间，然后将这两个值合并以创建日历条目，以免错过她的派对，如下列代码所示：

    ```cs
    DateOnly queensBirthday = new(year: 2022, month: 4, day: 21);
    WriteLine($"The Queen's next birthday is on {queensBirthday}.");
    TimeOnly partyStarts = new(hour: 20, minute: 30);
    WriteLine($"The Queen's party starts at {partyStarts}.");
    DateTime calendarEntry = queensBirthday.ToDateTime(partyStarts);
    WriteLine($"Add to your calendar: {calendarEntry}."); 
    ```

1.  运行代码并注意结果，如下列输出所示：

    ```cs
    The Queen's next birthday is on 21/04/2022.
    The Queen's party starts at 20:30.
    Add to your calendar: 21/04/2022 20:30:00. 
    ```

# 正则表达式模式匹配

正则表达式对于验证用户输入非常有用。它们功能强大且可能非常复杂。几乎所有编程语言都支持正则表达式，并使用一组通用的特殊字符来定义它们。

让我们尝试一些正则表达式的示例：

1.  使用您偏好的代码编辑器，在 `Chapter08` 解决方案/工作区中添加一个名为 `WorkingWithRegularExpressions` 的新控制台应用。

1.  在 Visual Studio Code 中，选择 `WorkingWithRegularExpressions` 作为活动 OmniSharp 项目。

1.  在 `Program.cs` 中，导入以下命名空间：

    ```cs
    using System.Text.RegularExpressions; 
    ```

## 检查作为文本输入的数字

我们将从实现验证数字输入的常见示例开始：

1.  添加语句提示用户输入年龄，然后使用正则表达式检查其有效性，该正则表达式查找数字字符，如下列代码所示：

    ```cs
    Write("Enter your age: "); 
    string? input = ReadLine();
    Regex ageChecker = new(@"\d"); 
    if (ageChecker.IsMatch(input))
    {
      WriteLine("Thank you!");
    }
    else
    {
      WriteLine($"This is not a valid age: {input}");
    } 
    ```

    注意以下关于代码的内容：

    +   `@` 字符关闭了在字符串中使用转义字符的能力。转义字符以前缀反斜杠表示。例如，`\t` 表示制表符，`\n` 表示新行。在编写正则表达式时，我们需要禁用此功能。借用电视剧《白宫风云》中的一句话，“让反斜杠就是反斜杠。”

    +   一旦使用 `@` 禁用了转义字符，它们就可以被正则表达式解释。例如，`\d` 表示数字。在本主题后面，您将学习更多以反斜杠为前缀的正则表达式。

1.  运行代码，输入一个整数如 `34` 作为年龄，并查看结果，如下列输出所示：

    ```cs
    Enter your age: 34 
    Thank you! 
    ```

1.  再次运行代码，输入 `carrots`，并查看结果，如下列输出所示：

    ```cs
    Enter your age: carrots
    This is not a valid age: carrots 
    ```

1.  再次运行代码，输入 `bob30smith`，并查看结果，如下列输出所示：

    ```cs
    Enter your age: bob30smith 
    Thank you! 
    ```

    我们使用的正则表达式是 `\d`，表示*一个数字*。然而，它并未指定在该数字之前和之后可以输入什么。这个正则表达式可以用英语描述为“输入任何你想要的字符，只要你至少输入一个数字字符。”

    在正则表达式中，您使用插入符号`^`符号表示某些输入的开始，使用美元`$`符号表示某些输入的结束。让我们使用这些符号来表示我们期望在输入的开始和结束之间除了数字外没有任何其他内容。

1.  将正则表达式更改为`^\d$`，如下面的代码中突出显示：

    ```cs
    Regex ageChecker = new(@"^**\d$"**); 
    ```

1.  再次运行代码并注意它拒绝除单个数字外的任何输入。我们希望允许一个或多个数字。为此，我们在`\d`表达式后添加一个`+`，以修改其含义为一个或多个。

1.  更改正则表达式，如下面的代码中突出显示：

    ```cs
    Regex ageChecker = new(@"^**\d+$"**); 
    ```

1.  再次运行代码并注意正则表达式仅允许长度为零或正整数的任何长度的数字。

## 正则表达式性能改进

.NET 中用于处理正则表达式的类型被广泛应用于.NET 平台及其构建的许多应用程序中。因此，它们对性能有重大影响，但直到现在，它们还没有得到微软太多的优化关注。

在.NET 5 及更高版本中，`System.Text.RegularExpressions`命名空间已重写内部以挤出最大性能。使用`IsMatch`等方法的常见正则表达式基准测试现在快了五倍。最好的事情是，您无需更改代码即可获得这些好处！

## 理解正则表达式的语法

以下是一些您可以在正则表达式中使用的常见正则表达式符号：

| 符号 | 含义 | 符号 | 含义 |
| --- | --- | --- | --- |
| `^` | 输入开始 | `$` | 输入结束 |
| `\d` | 单个数字 | `\D` | 单个非数字 |
| `\s` | 空白 | `\S` | 非空白 |
| `\w` | 单词字符 | `\W` | 非单词字符 |
| `[A-Za-z0-9]` | 字符范围 | `\^` | ^（插入符号）字符 |
| `[aeiou]` | 字符集 | `[^aeiou]` | 不在字符集中 |
| `.` | 任何单个字符 | `\.` | .（点）字符 |

此外，以下是一些影响正则表达式中前述符号的正则表达式量词：

| 符号 | 含义 | 符号 | 含义 |
| --- | --- | --- | --- |
| `+` | 一个或多个 | `?` | 一个或无 |
| `{3}` | 恰好三个 | `{3,5}` | 三个到五个 |
| `{3,}` | 至少三个 | `{,3}` | 最多三个 |

## 正则表达式示例

以下是一些带有其含义描述的正则表达式示例：

| 表达式 | 含义 |
| --- | --- |
| `\d` | 输入中某处的单个数字 |
| `a` | 输入中某处的字符*a* |
| `Bob` | 输入中某处的单词*Bob* |
| `^Bob` | 输入开头的单词*Bob* |
| `Bob$` | 输入末尾的单词*Bob* |
| `^\d{2}$` | 恰好两个数字 |
| `^[0-9]{2}$` | 恰好两个数字 |
| `^[A-Z]{4,}$` | ASCII 字符集中仅包含至少四个大写英文字母 |
| `^[A-Za-z]{4,}$` | ASCII 字符集中仅包含至少四个大写或小写英文字母 |
| `^[A-Z]{2}\d{3}$` | ASCII 字符集中仅包含两个大写英文字母和三个数字 |
| `^[A-Za-z\u00c0-\u017e]+$` | 至少一个 ASCII 字符集中的大写或小写英文字母，或 Unicode 字符集中的欧洲字母，如下表所示：ÀÁÂÃÄÅÆÇÈÉÊËÌÍÎÏÐÑÒÓÔÕÖ×ØÙÚÛÜÝÞßàáâãäåæçèéêëìíîïðñòóôõö÷øùúûüýþÿıŒœŠšŸ Žž |
| `^d.g$` | 字母*d*，然后是任何字符，然后是字母*g*，因此它会匹配*dig*和*dog*或*d*和*g*之间的任何单个字符 |
| `^d\.g$` | 字母*d*，然后是一个点（.），然后是字母*g*，因此它只会匹配*d.g* |

**良好实践**：使用正则表达式验证用户输入。相同的正则表达式可以在 JavaScript 和 Python 等其他语言中重复使用。

## 分割复杂的逗号分隔字符串

本章前面，你学习了如何分割一个简单的逗号分隔的字符串变量。但电影标题的以下示例呢？

```cs
"Monsters, Inc.","I, Tonya","Lock, Stock and Two Smoking Barrels" 
```

字符串值使用双引号围绕每个电影标题。我们可以利用这些来判断是否需要在逗号处分割（或不分割）。`Split`方法不够强大，因此我们可以使用正则表达式代替。

**良好实践**：你可以在 Stack Overflow 文章中找到更详细的解释，该文章启发了此任务，链接如下：[`stackoverflow.com/questions/18144431/regex-to-split-a-csv`](https://stackoverflow.com/questions/18144431/regex-to-split-a-csv)

要在`string`值中包含双引号，我们可以在它们前面加上反斜杠：

1.  添加语句以存储一个复杂的逗号分隔的`string`变量，然后使用`Split`方法以一种笨拙的方式分割它，如下面的代码所示：

    ```cs
    string films = "\"Monsters, Inc.\",\"I, Tonya\",\"Lock, Stock and Two Smoking Barrels\"";
    WriteLine($"Films to split: {films}");
    string[] filmsDumb = films.Split(',');
    WriteLine("Splitting with string.Split method:"); 
    foreach (string film in filmsDumb)
    {
      WriteLine(film);
    } 
    ```

1.  添加语句以定义一个正则表达式，用于智能地分割并写出电影标题，如下面的代码所示：

    ```cs
    WriteLine();
    Regex csv = new(
      "(?:^|,)(?=[^\"]|(\")?)\"?((?(1)[^\"]*|[^,\"]*))\"?(?=,|$)");
    MatchCollection filmsSmart = csv.Matches(films);
    WriteLine("Splitting with regular expression:"); 
    foreach (Match film in filmsSmart)
    {
      WriteLine(film.Groups[2].Value);
    } 
    ```

1.  运行代码并查看结果，如下面的输出所示：

    ```cs
    Splitting with string.Split method: 
    "Monsters
     Inc." 
    "I
     Tonya" 
    "Lock
     Stock and Two Smoking Barrels" 
    Splitting with regular expression: 
    Monsters, Inc.
    I, Tonya
    Lock, Stock and Two Smoking Barrels 
    ```

# 在集合中存储多个对象

另一种最常见的数据类型是集合。如果你需要在变量中存储多个值，那么你可以使用集合。

集合是一种内存中的数据结构，可以以不同方式管理多个项目，尽管所有集合都具有一些共享功能。

.NET 中用于处理集合的最常见类型如下表所示：

| 命名空间 | 示例类型 | 描述 |
| --- | --- | --- |
| `System .Collections` | `IEnumerable`, `IEnumerable<T>` | 集合使用的接口和基类。 |
| `System .Collections .Generic` | `List<T>`, `Dictionary<T>`, `Queue<T>`, `Stack<T>` | 在 C# 2.0 和.NET Framework 2.0 中引入，这些集合允许你使用泛型类型参数指定要存储的类型（更安全、更快、更高效）。 |
| `System .Collections .Concurrent` | `BlockingCollection`, `ConcurrentDictionary`, `ConcurrentQueue` | 这些集合在多线程场景中使用是安全的。 |
| `System.Collections.Immutable` | `ImmutableArray`、`ImmutableDictionary`、`ImmutableList`、`ImmutableQueue` | 设计用于原始集合内容永远不会改变的场景，尽管它们可以创建作为新实例的修改后的集合。 |

## 所有集合的共同特点

所有集合都实现了`ICollection`接口；这意味着它们必须有一个`Count`属性来告诉你其中有多少对象，如下面的代码所示：

```cs
namespace System.Collections
{
  public interface ICollection : IEnumerable
  {
    int Count { get; }
    bool IsSynchronized { get; }
    object SyncRoot { get; }
    void CopyTo(Array array, int index);
  }
} 
```

例如，如果我们有一个名为`passengers`的集合，我们可以这样做：

```cs
int howMany = passengers.Count; 
```

所有集合都实现了`IEnumerable`接口，这意味着它们可以使用`foreach`语句进行迭代。它们必须有一个`GetEnumerator`方法，该方法返回一个实现了`IEnumerator`的对象；这意味着返回的`对象`必须具有`MoveNext`和`Reset`方法来遍历集合，以及一个包含集合中当前项的`Current`属性，如下面的代码所示：

```cs
namespace System.Collections
{
  public interface IEnumerable
  {
    IEnumerator GetEnumerator();
  }
}
namespace System.Collections
{
  public interface IEnumerator
  {
    object Current { get; }
    bool MoveNext();
    void Reset();
  }
} 
```

例如，要对`passengers`集合中的每个对象执行一个操作，我们可以编写以下代码：

```cs
foreach (Passenger p in passengers)
{
  // perform an action on each passenger
} 
```

除了基于`object`的集合接口外，还有泛型接口和类，其中泛型类型定义了集合中存储的类型，如下面的代码所示：

```cs
namespace System.Collections.Generic
{
  public interface ICollection<T> : IEnumerable<T>, IEnumerable
  {
    int Count { get; }
    bool IsReadOnly { get; }
    void Add(T item);
    void Clear();
    bool Contains(T item);
    void CopyTo(T[] array, int index);
    bool Remove(T item);
  }
} 
```

## 通过确保集合的容量来提高性能

自.NET 1.1 以来，像`StringBuilder`这样的类型就有一个名为`EnsureCapacity`的方法，可以预先设置其内部存储数组到预期的最终大小。这提高了性能，因为它不需要在添加更多字符时反复增加数组的大小。

自.NET Core 2.1 以来，像`Dictionary<T>`和`HashSet<T>`这样的类型也有了`EnsureCapacity`。

在.NET 6 及更高版本中，像`List<T>`、`Queue<T>`和`Stack<T>`这样的集合现在也有了一个`EnsureCapacity`方法，如下面的代码所示：

```cs
List<string> names = new();
names.EnsureCapacity(10_000);
// load ten thousand names into the list 
```

## 理解集合选择

有几种不同的集合选择，你可以根据不同的目的使用：列表、字典、栈、队列、集合，以及许多其他更专业的集合。

### 列表

列表，即实现`IList<T>`的类型，是**有序集合**，如下面的代码所示：

```cs
namespace System.Collections.Generic
{
  [DefaultMember("Item")] // aka this indexer
  public interface IList<T> : ICollection<T>, IEnumerable<T>, IEnumerable
  {
    T this[int index] { get; set; }
    int IndexOf(T item);
    void Insert(int index, T item);
    void RemoveAt(int index);
  }
} 
```

`IList<T>`继承自`ICollection<T>`，因此它具有一个`Count`属性，以及一个`Add`方法，用于在集合末尾添加一个项，以及一个`Insert`方法，用于在列表中指定位置插入一个项，以及`RemoveAt`方法，用于在指定位置删除一个项。

当你想要手动控制集合中项目的顺序时，列表是一个好的选择。列表中的每个项目都有一个自动分配的唯一索引（或位置）。项目可以是`T`定义的任何类型，并且项目可以重复。索引是`int`类型，从`0`开始，因此列表中的第一个项目位于索引`0`处，如下表所示：

| 索引 | 项 |
| --- | --- |
| 0 | 伦敦 |
| 1 | 巴黎 |
| 2 | 伦敦 |
| 3 | 悉尼 |

如果一个新项（例如，圣地亚哥）被插入到伦敦和悉尼之间，那么悉尼的索引会自动增加。因此，你必须意识到，在插入或删除项后，项的索引可能会改变，如下表所示：

| 索引 | 项 |
| --- | --- |
| 0 | 伦敦 |
| 1 | 巴黎 |
| 2 | 伦敦 |
| 3 | 圣地亚哥 |
| 4 | 悉尼 |

### 字典

当每个**值**（或对象）有一个唯一的子值（或自定义值）可以用作**键**，以便稍后在集合中快速找到一个值时，字典是一个好选择。键必须是唯一的。例如，如果你正在存储一个人员列表，你可以选择使用政府颁发的身份证号码作为键。

将键想象成现实世界词典中的索引条目。它允许你快速找到一个词的定义，因为词（例如，键）是按顺序排列的，如果我们知道要查找*海牛*的定义，我们会跳到词典中间开始查找，因为字母*M*位于字母表的中间。

编程中的字典在查找内容时同样智能。它们必须实现接口`IDictionary<TKey, TValue>`，如下面的代码所示：

```cs
namespace System.Collections.Generic
{
  [DefaultMember("Item")] // aka this indexer
  public interface IDictionary<TKey, TValue>
    : ICollection<KeyValuePair<TKey, TValue>>,
      IEnumerable<KeyValuePair<TKey, TValue>>, IEnumerable
  {
    TValue this[TKey key] { get; set; }
    ICollection<TKey> Keys { get; }
    ICollection<TValue> Values { get; }
    void Add(TKey key, TValue value);
    bool ContainsKey(TKey key);
    bool Remove(TKey key);
    bool TryGetValue(TKey key, [MaybeNullWhen(false)] out TValue value);
  }
} 
```

字典中的项是`struct`的实例，也就是值类型`KeyValuePair<TKey, TValue>`，其中`TKey`是键的类型，`TValue`是值的类型，如下面的代码所示：

```cs
namespace System.Collections.Generic
{
  public readonly struct KeyValuePair<TKey, TValue>
  {
    public KeyValuePair(TKey key, TValue value);
    public TKey Key { get; }
    public TValue Value { get; }
    [EditorBrowsable(EditorBrowsableState.Never)]
    public void Deconstruct(out TKey key, out TValue value);
    public override string ToString();
  }
} 
```

一个示例`Dictionary<string, Person>`使用`string`作为键，`Person`实例作为值。`Dictionary<string, string>`对两者都使用`string`值，如下表所示：

| 键 | 值 |
| --- | --- |
| BSA | 鲍勃·史密斯 |
| MW | 马克斯·威廉姆斯 |
| BSB | 鲍勃·史密斯 |
| AM | 阿米尔·穆罕默德 |

### 栈

当你想要实现**后进先出**（**LIFO**）行为时，栈是一个好选择。使用栈，你只能直接访问或移除栈顶的项，尽管你可以枚举来读取整个栈的项。例如，你不能直接访问栈中的第二个项。

例如，文字处理器使用栈来记住你最近执行的操作顺序，然后当你按下 Ctrl + Z 时，它会撤销栈中的最后一个操作，然后是倒数第二个操作，依此类推。

### 队列

当你想要实现**先进先出**（**FIFO**）行为时，队列是一个好选择。使用队列，你只能直接访问或移除队列前端的项，尽管你可以枚举来读取整个队列的项。例如，你不能直接访问队列中的第二个项。

例如，后台进程使用队列按到达顺序处理工作项，就像人们在邮局排队一样。

.NET 6 引入了`PriorityQueue`，其中队列中的每个项都有一个优先级值以及它们在队列中的位置。

### 集合

当你想要在两个集合之间执行集合操作时，集合是一个好的选择。例如，你可能有两个城市名称的集合，并且你想要知道哪些名称同时出现在两个集合中（这被称为集合之间的*交集*）。集合中的项必须是唯一的。

### 集合方法总结

每种集合都有一套不同的添加和移除项的方法，如下表所示：

| 集合 | 添加方法 | 移除方法 | 描述 |
| --- | --- | --- | --- |
| 列表 | `添加`，`插入` | `移除`，`移除位置` | 列表是有序的，因此项具有整数索引位置。`添加`将在列表末尾添加一个新项。`插入`将在指定的索引位置添加一个新项。 |
| 字典 | `添加` | `移除` | 字典是无序的，因此项没有整数索引位置。你可以通过调用`ContainsKey`方法来检查一个键是否已被使用。 |
| 栈 | `压栈` | `弹栈` | 栈总是使用`压栈`方法在栈顶添加一个新项。第一个项位于栈底。总是使用`弹栈`方法从栈顶移除项。调用`Peek`方法可以查看此值而不移除它。 |
| 队列 | `入队` | `出队` | 队列总是使用`入队`方法在队列末尾添加一个新项。第一个项位于队列前端。总是使用`出队`方法从队列前端移除项。调用`Peek`方法可以查看此值而不移除它。 |

## 使用列表

让我们探索列表：

1.  使用你偏好的代码编辑器，在`Chapter08`解决方案/工作区中添加一个名为`WorkingWithCollections`的新控制台应用。

1.  在 Visual Studio Code 中，选择`WorkingWithCollections`作为活动的 OmniSharp 项目。

1.  在`Program.cs`中，删除现有语句，然后定义一个函数，输出带有标题的`string`值集合，如下所示：

    ```cs
    static void Output(string title, IEnumerable<string> collection)
    {
      WriteLine(title);
      foreach (string item in collection)
      {
        WriteLine($"  {item}");
      }
    } 
    ```

1.  定义一个名为`WorkingWithLists`的静态方法，以展示一些定义和使用列表的常见方式，如下所示：

    ```cs
    static void WorkingWithLists()
    {
      // Simple syntax for creating a list and adding three items
      List<string> cities = new(); 
      cities.Add("London"); 
      cities.Add("Paris"); 
      cities.Add("Milan");
      /* Alternative syntax that is converted by the compiler into
         the three Add method calls above
      List<string> cities = new()
        { "London", "Paris", "Milan" };
      */
      /* Alternative syntax that passes an 
         array of string values to AddRange method
      List<string> cities = new(); 
      cities.AddRange(new[] { "London", "Paris", "Milan" });
      */
      Output("Initial list", cities);
      WriteLine($"The first city is {cities[0]}."); 
      WriteLine($"The last city is {cities[cities.Count - 1]}.");
      cities.Insert(0, "Sydney");
      Output("After inserting Sydney at index 0", cities); 
      cities.RemoveAt(1); 
      cities.Remove("Milan");
      Output("After removing two cities", cities);
    } 
    ```

1.  在`Program.cs`顶部，在命名空间导入之后，调用`WorkingWithLists`方法，如下所示：

    ```cs
    WorkingWithLists(); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Initial list
      London
      Paris
      Milan
    The first city is London. 
    The last city is Milan.
    After inserting Sydney at index 0
      Sydney
      London
      Paris
      Milan
    After removing two cities
      Sydney
      Paris 
    ```

## 使用字典

让我们探索字典：

1.  在`Program.cs`中，定义一个名为`WorkingWithDictionaries`的静态方法，以展示一些使用字典的常见方式，例如，查找单词定义，如下所示：

    ```cs
    static void WorkingWithDictionaries()
    {
      Dictionary<string, string> keywords = new();
      // add using named parameters
      keywords.Add(key: "int", value: "32-bit integer data type");
      // add using positional parameters
      keywords.Add("long", "64-bit integer data type"); 
      keywords.Add("float", "Single precision floating point number");
      /* Alternative syntax; compiler converts this to calls to Add method
      Dictionary<string, string> keywords = new()
      {
        { "int", "32-bit integer data type" },
        { "long", "64-bit integer data type" },
        { "float", "Single precision floating point number" },
      }; */
      /* Alternative syntax; compiler converts this to calls to Add method
      Dictionary<string, string> keywords = new()
      {
        ["int"] = "32-bit integer data type",
        ["long"] = "64-bit integer data type",
        ["float"] = "Single precision floating point number", // last comma is optional
      }; */
      Output("Dictionary keys:", keywords.Keys);
      Output("Dictionary values:", keywords.Values);
      WriteLine("Keywords and their definitions");
      foreach (KeyValuePair<string, string> item in keywords)
      {
        WriteLine($"  {item.Key}: {item.Value}");
      }
      // lookup a value using a key
      string key = "long";
      WriteLine($"The definition of {key} is {keywords[key]}");
    } 
    ```

1.  在`Program.cs`顶部，注释掉之前的方法调用，然后调用`WorkingWithDictionaries`方法，如下所示：

    ```cs
    // WorkingWithLists();
    WorkingWithDictionaries(); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    Dictionary keys:
      int
      long
      float
    Dictionary values:
      32-bit integer data type
      64-bit integer data type
      Single precision floating point number
    Keywords and their definitions
      int: 32-bit integer data type
      long: 64-bit integer data type
      float: Single precision floating point number
    The definition of long is 64-bit integer data type 
    ```

## 使用队列

让我们探索队列：

1.  在`Program.cs`中，定义一个名为`WorkingWithQueues`的静态方法，以展示一些使用队列的常见方式，例如，处理排队购买咖啡的顾客，如下所示：

    ```cs
    static void WorkingWithQueues()
    {
      Queue<string> coffee = new();
      coffee.Enqueue("Damir"); // front of queue
      coffee.Enqueue("Andrea");
      coffee.Enqueue("Ronald");
      coffee.Enqueue("Amin");
      coffee.Enqueue("Irina"); // back of queue
      Output("Initial queue from front to back", coffee);
      // server handles next person in queue
      string served = coffee.Dequeue();
      WriteLine($"Served: {served}.");
      // server handles next person in queue
      served = coffee.Dequeue();
      WriteLine($"Served: {served}.");
      Output("Current queue from front to back", coffee);
      WriteLine($"{coffee.Peek()} is next in line.");
      Output("Current queue from front to back", coffee);
    } 
    ```

1.  在`Program.cs`顶部，注释掉之前的方法调用，并调用`WorkingWithQueues`方法。

1.  运行代码并查看结果，如下所示：

    ```cs
    Initial queue from front to back
      Damir
      Andrea
      Ronald
      Amin
      Irina
    Served: Damir.
    Served: Andrea.
    Current queue from front to back
      Ronald
      Amin
      Irina
    Ronald is next in line.
    Current queue from front to back
      Ronald
      Amin
      Irina 
    ```

1.  定义一个名为`OutputPQ`的静态方法，如下所示：

    ```cs
    static void OutputPQ<TElement, TPriority>(string title,
      IEnumerable<(TElement Element, TPriority Priority)> collection)
    {
      WriteLine(title);
      foreach ((TElement, TPriority) item in collection)
      {
        WriteLine($"  {item.Item1}: {item.Item2}");
      }
    } 
    ```

    请注意，`OutputPQ`方法是泛型的。你可以指定作为`collection`传递的元组中使用的两个类型。

1.  定义一个名为`WorkingWithPriorityQueues`的静态方法，如下所示：

    ```cs
    static void WorkingWithPriorityQueues()
    {
      PriorityQueue<string, int> vaccine = new();
      // add some people
      // 1 = high priority people in their 70s or poor health
      // 2 = medium priority e.g. middle aged
      // 3 = low priority e.g. teens and twenties
      vaccine.Enqueue("Pamela", 1);  // my mum (70s)
      vaccine.Enqueue("Rebecca", 3); // my niece (teens)
      vaccine.Enqueue("Juliet", 2);  // my sister (40s)
      vaccine.Enqueue("Ian", 1);     // my dad (70s)
      OutputPQ("Current queue for vaccination:", vaccine.UnorderedItems);
      WriteLine($"{vaccine.Dequeue()} has been vaccinated.");
      WriteLine($"{vaccine.Dequeue()} has been vaccinated.");
      OutputPQ("Current queue for vaccination:", vaccine.UnorderedItems);
      WriteLine($"{vaccine.Dequeue()} has been vaccinated.");
      vaccine.Enqueue("Mark", 2); // me (40s)
      WriteLine($"{vaccine.Peek()} will be next to be vaccinated.");
      OutputPQ("Current queue for vaccination:", vaccine.UnorderedItems);
    } 
    ```

1.  在`Program.cs`顶部，注释掉之前的方法调用，并调用`WorkingWithPriorityQueues`方法。

1.  运行代码并查看结果，如下所示：

    ```cs
    Current queue for vaccination:
      Pamela: 1
      Rebecca: 3
      Juliet: 2
      Ian: 1
    Pamela has been vaccinated.
    Ian has been vaccinated.
    Current queue for vaccination:
      Juliet: 2
      Rebecca: 3
    Juliet has been vaccinated.
    Mark will be next to be vaccinated.
    Current queue for vaccination:
      Mark: 2
      Rebecca: 3 
    ```

## 排序集合

`List<T>`类可以通过手动调用其`Sort`方法进行排序（但请记住，每个项的索引会改变）。手动对`string`值或其他内置类型的列表进行排序无需额外努力，但如果你创建了自己的类型的集合，则该类型必须实现名为`IComparable`的接口。你在《第六章：实现接口和继承类》中学过如何做到这一点。

`Stack<T>`或`Queue<T>`集合无法排序，因为你通常不需要这种功能；例如，你可能永远不会对入住酒店的客人队列进行排序。但有时，你可能想要对字典或集合进行排序。

有时拥有一个自动排序的集合会很有用，即在添加和删除项时保持项的排序顺序。

有多种自动排序集合可供选择。这些排序集合之间的差异通常很微妙，但可能会影响应用程序的内存需求和性能，因此值得努力选择最适合你需求的选项。

一些常见的自动排序集合如下表所示：

| 集合 | 描述 |
| --- | --- |
| `SortedDictionary<TKey, TValue>` | 这表示一个按键排序的键/值对集合。 |
| `SortedList<TKey, TValue>` | 这表示一个按键排序的键/值对集合。 |
| `SortedSet<T>` | 这表示一个唯一的对象集合，这些对象按排序顺序维护。 |

## 更专业的集合

还有其他一些用于特殊情况的集合。

### 使用紧凑的位值数组

`System.Collections.BitArray`集合管理一个紧凑的位值数组，这些位值表示为布尔值，其中`true`表示位已打开（值为 1），`false`表示位已关闭（值为 0）。

### 高效地使用列表

`System.Collections.Generics.LinkedList<T>`集合表示一个双向链表，其中每个项都有对其前一个和下一个项的引用。与`List<T>`相比，在频繁从列表中间插入和删除项的场景中，它们提供了更好的性能。在`LinkedList<T>`中，项无需在内存中重新排列。

## 使用不可变集合

有时你需要使集合不可变，这意味着其成员不可更改；即，你不能添加或删除它们。

如果你导入了`System.Collections.Immutable`命名空间，那么任何实现`IEnumerable<T>`的集合都会获得六个扩展方法，用于将其转换为不可变列表、字典、哈希集等。

让我们看一个简单的例子：

1.  在`WorkingWithCollections`项目中，在`Program.cs`中，导入`System.Collections.Immutable`命名空间。

1.  在`WorkingWithLists`方法中，在方法末尾添加语句，将`cities`列表转换为不可变列表，然后向其添加一个新城市，如下代码所示：

    ```cs
    ImmutableList<string> immutableCities = cities.ToImmutableList();
    ImmutableList<string> newList = immutableCities.Add("Rio");
    Output("Immutable list of cities:", immutableCities); 
    Output("New list of cities:", newList); 
    ```

1.  在`Program.cs`顶部，注释掉之前的方法调用，并取消对`WorkingWithLists`方法调用的注释。

1.  运行代码，查看结果，并注意当对不可变城市列表调用`Add`方法时，该列表并未被修改；相反，它返回了一个包含新添加城市的新列表，如下输出所示：

    ```cs
    Immutable list of cities:
      Sydney
      Paris
    New list of cities:
      Sydney
      Paris
      Rio 
    ```

**良好实践**：为了提高性能，许多应用程序在中央缓存中存储了常用对象的共享副本。为了安全地允许多个线程使用这些对象，同时确保它们不会被更改，你应该使它们不可变，或者使用并发集合类型，你可以在以下链接中了解相关信息：[`docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent`](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent)

## 集合的良好实践

假设你需要创建一个处理集合的方法。为了最大程度地灵活，你可以声明输入参数为`IEnumerable<T>`，并使方法泛型化，如下代码所示：

```cs
void ProcessCollection<T>(IEnumerable<T> collection)
{
  // process the items in the collection,
  // perhaps using a foreach statement
} 
```

我可以将数组、列表、队列、栈或任何其他实现`IEnumerable<T>`的集合传递给此方法，它将处理这些项。然而，将任何集合传递给此方法的灵活性是以性能为代价的。

`IEnumerable<T>`的一个性能问题同时也是其优点之一：延迟执行，亦称为懒加载。实现此接口的类型并非必须实现延迟执行，但许多类型确实如此。

但`IEnumerable<T>`最糟糕的性能问题是迭代时必须在堆上分配一个对象。为了避免这种内存分配，你应该使用具体类型定义你的方法，如下代码中突出显示的部分所示：

```cs
void ProcessCollection<T>(**List<T>** collection)
{
  // process the items in the collection,
  // perhaps using a foreach statement
} 
```

这将使用 `List<T>.Enumerator GetEnumerator()` 方法，该方法返回一个 `struct`，而不是返回引用类型的 `IEnumerator<T> GetEnumerator()` 方法。您的代码将快两到三倍，并且需要更少的内存。与所有与性能相关的建议一样，您应该通过在产品环境中运行实际代码的性能测试来确认好处。您将在*第十二章*，*使用多任务提高性能和可扩展性*中学习如何做到这一点。

# 处理跨度、索引和范围

Microsoft 在 .NET Core 2.1 中的目标之一是提高性能和资源使用率。实现这一目标的关键 .NET 特性是 `Span<T>` 类型。

## 使用跨度高效利用内存

在操作数组时，您通常会创建现有子集的新副本，以便仅处理该子集。这样做效率不高，因为必须在内存中创建重复对象。

如果您需要处理数组的子集，请使用**跨度**，因为它就像原始数组的窗口。这在内存使用方面更有效，并提高了性能。跨度仅适用于数组，不适用于集合，因为内存必须是连续的。

在我们更详细地了解跨度之前，我们需要了解一些相关对象：索引和范围。

## 使用 Index 类型识别位置

C# 8.0 引入了两个特性，用于识别数组中项的索引以及使用两个索引的范围。

您在上一主题中学到，可以通过将整数传递给其索引器来访问列表中的对象，如下所示：

```cs
int index = 3;
Person p = people[index]; // fourth person in array
char letter = name[index]; // fourth letter in name 
```

`Index` 值类型是一种更正式的识别位置的方式，并支持从末尾计数，如下所示：

```cs
// two ways to define the same index, 3 in from the start 
Index i1 = new(value: 3); // counts from the start 
Index i2 = 3; // using implicit int conversion operator
// two ways to define the same index, 5 in from the end
Index i3 = new(value: 5, fromEnd: true); 
Index i4 = ⁵; // using the caret operator 
```

## 使用 Range 类型识别范围

`Range` 值类型使用 `Index` 值来指示其范围的起始和结束，使用其构造函数、C# 语法或其静态方法，如下所示：

```cs
Range r1 = new(start: new Index(3), end: new Index(7));
Range r2 = new(start: 3, end: 7); // using implicit int conversion
Range r3 = 3..7; // using C# 8.0 or later syntax
Range r4 = Range.StartAt(3); // from index 3 to last index
Range r5 = 3..; // from index 3 to last index
Range r6 = Range.EndAt(3); // from index 0 to index 3
Range r7 = ..3; // from index 0 to index 3 
```

已向 `string` 值（内部使用 `char` 数组）、`int` 数组和跨度添加了扩展方法，以使范围更易于使用。这些扩展方法接受一个范围作为参数并返回一个 `Span<T>`。这使得它们非常节省内存。

## 使用索引、范围和跨度

让我们探索使用索引和范围来返回跨度：

1.  使用您喜欢的代码编辑器将名为 `WorkingWithRanges` 的新控制台应用程序添加到 `Chapter08` 解决方案/工作区。

1.  在 Visual Studio Code 中，选择 `WorkingWithRanges` 作为活动 OmniSharp 项目。

1.  在 `Program.cs` 中，键入语句以使用 `string` 类型的 `Substring` 方法使用范围来提取某人姓名的部分，如下所示：

    ```cs
    string name = "Samantha Jones";
    // Using Substring
    int lengthOfFirst = name.IndexOf(' ');
    int lengthOfLast = name.Length - lengthOfFirst - 1;
    string firstName = name.Substring(
      startIndex: 0,
      length: lengthOfFirst);
    string lastName = name.Substring(
      startIndex: name.Length - lengthOfLast,
      length: lengthOfLast);
    WriteLine($"First name: {firstName}, Last name: {lastName}");
    // Using spans
    ReadOnlySpan<char> nameAsSpan = name.AsSpan();
    ReadOnlySpan<char> firstNameSpan = nameAsSpan[0..lengthOfFirst]; 
    ReadOnlySpan<char> lastNameSpan = nameAsSpan[^lengthOfLast..⁰];
    WriteLine("First name: {0}, Last name: {1}", 
      arg0: firstNameSpan.ToString(),
      arg1: lastNameSpan.ToString()); 
    ```

1.  运行代码并查看结果，如下所示：

    ```cs
    First name: Samantha, Last name: Jones 
    First name: Samantha, Last name: Jones 
    ```

# 处理网络资源

有时您需要处理网络资源。.NET 中用于处理网络资源的最常见类型如下表所示：

| 命名空间 | 示例类型 | 描述 |
| --- | --- | --- |
| `System.Net` | `Dns`, `Uri`, `Cookie`, `WebClient`, `IPAddress` | 这些用于处理 DNS 服务器、URI、IP 地址等。 |
| `System.Net` | `FtpStatusCode`, `FtpWebRequest`, `FtpWebResponse` | 这些用于与 FTP 服务器进行交互。 |
| `System.Net` | `HttpStatusCode`, `HttpWebRequest`, `HttpWebResponse` | 这些用于与 HTTP 服务器进行交互；即网站和服务。来自`System.Net.Http`的类型更容易使用。 |
| `System.Net.Http` | `HttpClient`, `HttpMethod`, `HttpRequestMessage`, `HttpResponseMessage` | 这些用于与 HTTP 服务器（即网站和服务）进行交互。你将在*第十六章*，*构建和消费 Web 服务*中学习如何使用这些。 |
| `System.Net.Mail` | `Attachment`, `MailAddress`, `MailMessage`, `SmtpClient` | 这些用于处理 SMTP 服务器；即发送电子邮件。 |
| `System.Net.NetworkInformation` | `IPStatus`, `NetworkChange`, `Ping`, `TcpStatistics` | 这些用于处理低级网络协议。 |

## 处理 URI、DNS 和 IP 地址

让我们探索一些用于处理网络资源的常见类型：

1.  使用你偏好的代码编辑器，在`Chapter08`解决方案/工作区中添加一个名为`WorkingWithNetworkResources`的新控制台应用。

1.  在 Visual Studio Code 中，选择`WorkingWithNetworkResources`作为活动 OmniSharp 项目。

1.  在`Program.cs`顶部，导入用于处理网络的命名空间，如下所示：

    ```cs
    using System.Net; // IPHostEntry, Dns, IPAddress 
    ```

1.  输入语句以提示用户输入网站地址，然后使用`Uri`类型将其分解为其组成部分，包括方案（HTTP、FTP 等）、端口号和主机，如下所示：

    ```cs
    Write("Enter a valid web address: "); 
    string? url = ReadLine();
    if (string.IsNullOrWhiteSpace(url))
    {
      url = "https://stackoverflow.com/search?q=securestring";
    }
    Uri uri = new(url);
    WriteLine($"URL: {url}"); 
    WriteLine($"Scheme: {uri.Scheme}"); 
    WriteLine($"Port: {uri.Port}"); 
    WriteLine($"Host: {uri.Host}"); 
    WriteLine($"Path: {uri.AbsolutePath}"); 
    WriteLine($"Query: {uri.Query}"); 
    ```

    为了方便，代码还允许用户按下 ENTER 键使用示例 URL。

1.  运行代码，输入有效的网站地址或按下 ENTER 键，查看结果，如下所示：

    ```cs
    Enter a valid web address:
    URL: https://stackoverflow.com/search?q=securestring 
    Scheme: https
    Port: 443
    Host: stackoverflow.com 
    Path: /search
    Query: ?q=securestring 
    ```

1.  添加语句以获取输入网站的 IP 地址，如下所示：

    ```cs
    IPHostEntry entry = Dns.GetHostEntry(uri.Host); 
    WriteLine($"{entry.HostName} has the following IP addresses:"); 
    foreach (IPAddress address in entry.AddressList)
    {
      WriteLine($"  {address} ({address.AddressFamily})");
    } 
    ```

1.  运行代码，输入有效的网站地址或按下 ENTER 键，查看结果，如下所示：

    ```cs
    stackoverflow.com has the following IP addresses: 
      151.101.193.69 (InterNetwork)
      151.101.129.69 (InterNetwork)
      151.101.1.69 (InterNetwork)
      151.101.65.69 (InterNetwork) 
    ```

## ping 服务器

现在你将添加代码以 ping 一个 Web 服务器以检查其健康状况：

1.  导入命名空间以获取更多网络信息，如下所示：

    ```cs
    using System.Net.NetworkInformation; // Ping, PingReply, IPStatus 
    ```

1.  添加语句以 ping 输入的网站，如下所示：

    ```cs
    try
    {
      Ping ping = new();
      WriteLine("Pinging server. Please wait...");
      PingReply reply = ping.Send(uri.Host);
      WriteLine($"{uri.Host} was pinged and replied: {reply.Status}.");
      if (reply.Status == IPStatus.Success)
      {
        WriteLine("Reply from {0} took {1:N0}ms", 
          arg0: reply.Address,
          arg1: reply.RoundtripTime);
      }
    }
    catch (Exception ex)
    {
      WriteLine($"{ex.GetType().ToString()} says {ex.Message}");
    } 
    ```

1.  运行代码，按下 ENTER 键，查看结果，如下所示在 macOS 上的输出：

    ```cs
    Pinging server. Please wait...
    stackoverflow.com was pinged and replied: Success.
    Reply from 151.101.193.69 took 18ms took 136ms 
    ```

1.  再次运行代码，但这次输入[`google.com`](http://google.com)，如下所示：

    ```cs
    Enter a valid web address: http://google.com
    URL: http://google.com
    Scheme: http
    Port: 80
    Host: google.com
    Path: /
    Query: 
    google.com has the following IP addresses:
      2a00:1450:4009:807::200e (InterNetworkV6)
      216.58.204.238 (InterNetwork)
    Pinging server. Please wait...
    google.com was pinged and replied: Success.
    Reply from 2a00:1450:4009:807::200e took 24ms 
    ```

# 处理反射和属性

**反射**是一种编程特性，允许代码理解和操作自身。一个程序集由最多四个部分组成：

+   **程序集元数据和清单**：名称、程序集和文件版本、引用的程序集等。

+   **类型元数据**：关于类型、其成员等的信息。

+   **IL 代码**：方法、属性、构造函数等的实现。

+   **嵌入资源**（可选）：图像、字符串、JavaScript 等。

元数据包含有关您的代码的信息项。元数据自动从您的代码生成（例如，关于类型和成员的信息）或使用属性应用于您的代码。

属性可以应用于多个级别：程序集、类型及其成员，如下列代码所示：

```cs
// an assembly-level attribute
[assembly: AssemblyTitle("Working with Reflection")]
// a type-level attribute
[Serializable] 
public class Person
{
  // a member-level attribute 
  [Obsolete("Deprecated: use Run instead.")] 
  public void Walk()
  {
... 
```

基于属性的编程在 ASP.NET Core 等应用程序模型中大量使用，以启用路由、安全性、缓存等功能。

## 程序集版本控制

.NET 中的版本号是三个数字的组合，带有两个可选的附加项。如果遵循语义版本规则，这三个数字表示以下内容：

+   **主要**：破坏性更改。

+   **次要**：非破坏性更改，包括新功能，通常还包括错误修复。

+   **补丁**：非破坏性错误修复。

**良好实践**：在更新您已在项目中使用的 NuGet 包时，为了安全起见，您应该指定一个可选标志，以确保您仅升级到最高次要版本以避免破坏性更改，或者如果您特别谨慎并且只想接收错误修复，则升级到最高补丁，如下列命令所示：`Update-Package Newtonsoft.Json -ToHighestMinor` 或 `Update-Package Newtonsoft.Json -ToHighestPatch`。

可选地，版本可以包括这些：

+   **预发布**：不支持的预览版本。

+   **构建编号**：每日构建。

**良好实践**：遵循语义版本规则，详情请参见以下链接：[`semver.org`](http://semver.org)

## 读取程序集元数据

让我们探索属性操作：

1.  使用您喜欢的代码编辑器，在`Chapter08`解决方案/工作区中添加一个名为`WorkingWithReflection`的新控制台应用程序。

1.  在 Visual Studio Code 中，选择`WorkingWithReflection`作为活动 OmniSharp 项目。

1.  在`Program.cs`顶部，导入反射命名空间，如下列代码所示：

    ```cs
    using System.Reflection; // Assembly 
    ```

1.  添加语句以获取控制台应用程序的程序集，输出其名称和位置，并获取所有程序集级属性并输出它们的类型，如下列代码所示：

    ```cs
    WriteLine("Assembly metadata:");
    Assembly? assembly = Assembly.GetEntryAssembly();
    if (assembly is null)
    {
      WriteLine("Failed to get entry assembly.");
      return;
    }
    WriteLine($"  Full name: {assembly.FullName}"); 
    WriteLine($"  Location: {assembly.Location}");
    IEnumerable<Attribute> attributes = assembly.GetCustomAttributes(); 
    WriteLine($"  Assembly-level attributes:");
    foreach (Attribute a in attributes)
    {
      WriteLine($"   {a.GetType()}");
    } 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
    Assembly metadata:
      Full name: WorkingWithReflection, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
      Location: /Users/markjprice/Code/Chapter08/WorkingWithReflection/bin/Debug/net6.0/WorkingWithReflection.dll
      Assembly-level attributes:
        System.Runtime.CompilerServices.CompilationRelaxationsAttribute
        System.Runtime.CompilerServices.RuntimeCompatibilityAttribute
        System.Diagnostics.DebuggableAttribute
        System.Runtime.Versioning.TargetFrameworkAttribute
        System.Reflection.AssemblyCompanyAttribute
        System.Reflection.AssemblyConfigurationAttribute
        System.Reflection.AssemblyFileVersionAttribute
        System.Reflection.AssemblyInformationalVersionAttribute
        System.Reflection.AssemblyProductAttribute
        System.Reflection.AssemblyTitleAttribute 
    ```

    请注意，因为程序集的全名必须唯一标识程序集，所以它是以下内容的组合：

    +   **名称**，例如，`WorkingWithReflection`

    +   **版本**，例如，`1.0.0.0`

    +   **文化**，例如，`neutral`

    +   **公钥标记**，尽管这可以是`null`

    既然我们已经了解了一些装饰程序集的属性，我们可以专门请求它们。

1.  添加语句以获取`AssemblyInformationalVersionAttribute`和`AssemblyCompanyAttribute`类，然后输出它们的值，如下列代码所示：

    ```cs
    AssemblyInformationalVersionAttribute? version = assembly
      .GetCustomAttribute<AssemblyInformationalVersionAttribute>(); 
    WriteLine($"  Version: {version?.InformationalVersion}");
    AssemblyCompanyAttribute? company = assembly
      .GetCustomAttribute<AssemblyCompanyAttribute>();
    WriteLine($"  Company: {company?.Company}"); 
    ```

1.  运行代码并查看结果，如下列输出所示：

    ```cs
     Version: 1.0.0
      Company: WorkingWithReflection 
    ```

    嗯，除非设置版本，否则默认值为 1.0.0，除非设置公司，否则默认值为程序集名称。让我们明确设置这些信息。在旧版.NET Framework 中设置这些值的方法是在 C#源代码文件中添加属性，如下所示：

    ```cs
    [assembly: AssemblyCompany("Packt Publishing")] 
    [assembly: AssemblyInformationalVersion("1.3.0")] 
    ```

    .NET 使用的 Roslyn 编译器会自动设置这些属性，因此我们不能采用旧方法。相反，必须在项目文件中设置它们。

1.  编辑`WorkingWithReflection.csproj`项目文件，添加版本和公司元素，如下所示高亮显示：

    ```cs
    <Project Sdk="Microsoft.NET.Sdk">
      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
     **<Version>****6.3.12****</Version>**
     **<Company>Packt Publishing</Company>**
      </PropertyGroup>
    </Project> 
    ```

1.  运行代码并查看结果，如下所示输出：

    ```cs
     Version: 6.3.12
      Company: Packt Publishing 
    ```

## 创建自定义属性

你可以通过继承`Attribute`类来定义自己的属性：

1.  向项目中添加一个名为`CoderAttribute.cs`的类文件。

1.  定义一个属性类，该类可以装饰类或方法，并存储程序员姓名和上次修改代码的日期这两个属性，如下所示：

    ```cs
    namespace Packt.Shared;
    [AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, 
      AllowMultiple = true)]
    public class CoderAttribute : Attribute
    {
      public string Coder { get; set; }
      public DateTime LastModified { get; set; }
      public CoderAttribute(string coder, string lastModified)
      {
        Coder = coder;
        LastModified = DateTime.Parse(lastModified);
      }
    } 
    ```

1.  在`Program.cs`中，导入一些命名空间，如下所示：

    ```cs
    using System.Runtime.CompilerServices; // CompilerGeneratedAttribute
    using Packt.Shared; // CoderAttribute 
    ```

1.  在`Program.cs`底部，添加一个带有方法的类，并用包含两位程序员信息的`Coder`属性装饰该方法，如下所示：

    ```cs
    class Animal
    {
      [Coder("Mark Price", "22 August 2021")]
      [Coder("Johnni Rasmussen", "13 September 2021")] 
      public void Speak()
      {
        WriteLine("Woof...");
      }
    } 
    ```

1.  在`Program.cs`中，在`Animal`类上方，添加代码以获取类型，枚举其成员，读取这些成员上的任何`Coder`属性，并输出信息，如下所示：

    ```cs
    WriteLine(); 
    WriteLine($"* Types:");
    Type[] types = assembly.GetTypes();
    foreach (Type type in types)
    {
      WriteLine();
      WriteLine($"Type: {type.FullName}"); 
      MemberInfo[] members = type.GetMembers();
      foreach (MemberInfo member in members)
      {
        WriteLine("{0}: {1} ({2})",
          arg0: member.MemberType,
          arg1: member.Name,
          arg2: member.DeclaringType?.Name);
        IOrderedEnumerable<CoderAttribute> coders = 
          member.GetCustomAttributes<CoderAttribute>()
          .OrderByDescending(c => c.LastModified);
        foreach (CoderAttribute coder in coders)
        {
          WriteLine("-> Modified by {0} on {1}",
            coder.Coder, coder.LastModified.ToShortDateString());
        }
      }
    } 
    ```

1.  运行代码并查看结果，如下所示部分输出：

    ```cs
    * Types:
    ...
    Type: Animal
    Method: Speak (Animal)
    -> Modified by Johnni Rasmussen on 13/09/2021
    -> Modified by Mark Price on 22/08/2021
    Method: GetType (Object)
    Method: ToString (Object)
    Method: Equals (Object)
    Method: GetHashCode (Object)
    Constructor: .ctor (Program)
    ...
    Type: <Program>$+<>c
    Method: GetType (Object)
    Method: ToString (Object)
    Method: Equals (Object)
    Method: GetHashCode (Object)
    Constructor: .ctor (<>c)
    Field: <>9 (<>c)
    Field: <>9__0_0 (<>c) 
    ```

`<Program>$+<>c`类型是什么？

这是一个编译器生成的**显示类**。`<>`表示编译器生成，`c`表示显示类。它们是编译器的未记录实现细节，可能会随时更改。你可以忽略它们，因此作为一个可选挑战，向你的控制台应用程序添加语句，通过跳过带有`CompilerGeneratedAttribute`装饰的类型来过滤编译器生成的类型。

## 利用反射实现更多功能

这只是反射所能实现功能的一个尝鲜。我们仅使用反射从代码中读取元数据。反射还能执行以下操作：

+   **动态加载当前未引用的程序集**：[`docs.microsoft.com/en-us/dotnet/standard/assembly/unloadability`](https://docs.microsoft.com/en-us/dotnet/standard/assembly/unloadability)

+   **动态执行代码**：[`docs.microsoft.com/en-us/dotnet/api/system.reflection.methodbase.invoke`](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.methodbase.invoke)

+   **动态生成新代码和程序集**：[`docs.microsoft.com/en-us/dotnet/api/system.reflection.emit.assemblybuilder`](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.emit.assemblybuilder)

# 处理图像

ImageSharp 是一个第三方跨平台 2D 图形库。当.NET Core 1.0 正在开发时，社区对缺少用于处理 2D 图像的`System.Drawing`命名空间有负面反馈。

**ImageSharp**项目正是为了填补现代.NET 应用中的这一空白而启动的。

微软在其官方文档中关于`System.Drawing`的部分指出：“由于不支持在 Windows 或 ASP.NET 服务中使用，且不支持跨平台，`System.Drawing`命名空间不建议用于新开发。推荐使用 ImageSharp 和 SkiaSharp 作为替代。”

让我们看看 ImageSharp 能实现什么：

1.  使用您偏好的代码编辑器，向`Chapter08`解决方案/工作区添加一个名为`WorkingWithImages`的新控制台应用。

1.  在 Visual Studio Code 中，选择`WorkingWithImages`作为活动 OmniSharp 项目。

1.  创建一个`images`目录，并从以下链接下载九张图片：[`github.com/markjprice/cs10dotnet6/tree/master/Assets/Categories`](https://github.com/markjprice/cs10dotnet6/tree/master/Assets/Categories)

1.  添加对`SixLabors.ImageSharp`的包引用，如下所示：

    ```cs
    <ItemGroup>
      <PackageReference Include="SixLabors.ImageSharp" Version="1.0.3" />
    </ItemGroup> 
    ```

1.  构建`WorkingWithImages`项目。

1.  在`Program.cs`顶部，导入一些用于处理图像的命名空间，如下所示：

    ```cs
    using SixLabors.ImageSharp;
    using SixLabors.ImageSharp.Processing; 
    ```

1.  在`Program.cs`中，输入语句将`images`文件夹中的所有文件转换为灰度缩略图，大小为原图的十分之一，如下所示：

    ```cs
    string imagesFolder = Path.Combine(
      Environment.CurrentDirectory, "images");
    IEnumerable<string> images =
      Directory.EnumerateFiles(imagesFolder);
    foreach (string imagePath in images)
    {
      string thumbnailPath = Path.Combine(
        Environment.CurrentDirectory, "images",   
        Path.GetFileNameWithoutExtension(imagePath)
        + "-thumbnail" + Path.GetExtension(imagePath));
      using (Image image = Image.Load(imagePath))
      {
        image.Mutate(x => x.Resize(image.Width / 10, image.Height / 10));   
        image.Mutate(x => x.Grayscale());
        image.Save(thumbnailPath);
      }
    }
    WriteLine("Image processing complete. View the images folder."); 
    ```

1.  运行代码。

1.  在文件系统中，打开`images`文件夹，注意字节数显著减少的灰度缩略图，如图*8.1*所示：![应用程序图片 自动生成的描述](img/B17442_08_01.png)

    图 8.1：处理后的图像

ImageSharp 还提供了用于程序化绘制图像和处理网络图像的 NuGet 包，如下表所示：

+   `SixLabors.ImageSharp.Drawing`

+   `SixLabors.ImageSharp.Web`

# 国际化您的代码

国际化是使代码在全球范围内正确运行的过程。它包括两个部分：**全球化**和**本地化**。

全球化意味着编写代码时要考虑多种语言和地区组合。语言与地区的组合被称为文化。代码需要了解语言和地区，因为例如魁北克和巴黎虽然都使用法语，但日期和货币格式却不同。

所有文化组合都有**国际标准化组织**（**ISO**）代码。例如，代码`da-DK`中，`da`代表丹麦语，`DK`代表丹麦地区；而在代码`fr-CA`中，`fr`代表法语，`CA`代表加拿大地区。

ISO 并非缩写。ISO 是对希腊语单词*isos*（意为相等）的引用。

本地化是关于定制用户界面以支持一种语言，例如，将按钮的标签更改为关闭（`en`）或 Fermer（`fr`）。由于本地化更多地涉及语言，因此它并不总是需要了解区域，尽管具有讽刺意味的是，标准化（`en-US`）和标准化（`en-GB`）暗示了相反的情况。

## 检测和更改当前文化

国际化是一个庞大的主题，已有数千页的书籍专门论述。在本节中，你将通过`System.Globalization`命名空间中的`CultureInfo`类型简要了解基础知识。

让我们写一些代码：

1.  使用你偏好的代码编辑器，在`Chapter08`解决方案/工作区中添加一个名为`Internationalization`的新控制台应用。

1.  在 Visual Studio Code 中，选择`Internationalization`作为活动的 OmniSharp 项目。

1.  在`Program.cs`的顶部，导入用于使用全球化类型的命名空间，如下面的代码所示：

    ```cs
    using System.Globalization; // CultureInfo 
    ```

1.  添加语句以获取当前的全球化文化和本地化文化，并输出有关它们的一些信息，然后提示用户输入新的文化代码，并展示这如何影响常见值（如日期和货币）的格式化，如下面的代码所示：

    ```cs
    CultureInfo globalization = CultureInfo.CurrentCulture; 
    CultureInfo localization = CultureInfo.CurrentUICulture;
    WriteLine("The current globalization culture is {0}: {1}",
      globalization.Name, globalization.DisplayName);
    WriteLine("The current localization culture is {0}: {1}",
      localization.Name, localization.DisplayName);
    WriteLine();
    WriteLine("en-US: English (United States)"); 
    WriteLine("da-DK: Danish (Denmark)"); 
    WriteLine("fr-CA: French (Canada)"); 
    Write("Enter an ISO culture code: ");  
    string? newCulture = ReadLine();
    if (!string.IsNullOrEmpty(newCulture))
    {
      CultureInfo ci = new(newCulture); 
      // change the current cultures
      CultureInfo.CurrentCulture = ci;
      CultureInfo.CurrentUICulture = ci;
    }
    WriteLine();
    Write("Enter your name: "); 
    string? name = ReadLine();
    Write("Enter your date of birth: "); 
    string? dob = ReadLine();
    Write("Enter your salary: "); 
    string? salary = ReadLine();
    DateTime date = DateTime.Parse(dob);
    int minutes = (int)DateTime.Today.Subtract(date).TotalMinutes; 
    decimal earns = decimal.Parse(salary);
    WriteLine(
      "{0} was born on a {1:dddd}, is {2:N0} minutes old, and earns {3:C}",
      name, date, minutes, earns); 
    ```

    当你运行一个应用程序时，它会自动将其线程设置为使用操作系统的文化。我在英国伦敦运行我的代码，因此线程被设置为英语（英国）。

    代码提示用户输入替代的 ISO 代码。这允许你的应用程序在运行时替换默认文化。

    应用程序然后使用标准格式代码输出星期几，使用格式代码`dddd`；使用千位分隔符的分钟数，使用格式代码`N0`；以及带有货币符号的薪水。这些会根据线程的文化自动调整。

1.  运行代码并输入`en-GB`作为 ISO 代码，然后输入一些样本数据，包括英国英语中有效的日期格式，如下面的输出所示：

    ```cs
    Enter an ISO culture code: en-GB 
    Enter your name: Alice
    Enter your date of birth: 30/3/1967 
    Enter your salary: 23500
    Alice was born on a Thursday, is 25,469,280 minutes old, and earns
    £23,500.00 
    ```

    如果你输入`en-US`而不是`en-GB`，则必须使用月/日/年的格式输入日期。

1.  重新运行代码并尝试不同的文化，例如丹麦的丹麦语，如下面的输出所示：

    ```cs
    Enter an ISO culture code: da-DK 
    Enter your name: Mikkel
    Enter your date of birth: 12/3/1980 
    Enter your salary: 340000
    Mikkel was born on a onsdag, is 18.656.640 minutes old, and earns 340.000,00 kr. 
    ```

在此示例中，只有日期和薪水被全球化为丹麦语。其余文本硬编码为英语。本书目前不包括如何将文本从一种语言翻译成另一种语言。如果你希望我在下一版中包含这一点，请告诉我。

**良好实践**：考虑你的应用程序是否需要国际化，并在开始编码之前为此做好计划！写下用户界面中需要本地化的所有文本片段。考虑所有需要全球化的数据（日期格式、数字格式和排序文本行为）。

# 实践和探索

通过回答一些问题来测试您的知识和理解，进行一些实践练习，并深入研究本章的主题。

## 练习 8.1 – 测试您的知识

使用网络回答以下问题：

1.  一个`string`变量中最多可以存储多少个字符？

1.  何时以及为何应使用`SecureString`类型？

1.  何时适合使用`StringBuilder`类？

1.  何时应使用`LinkedList<T>`类？

1.  何时应使用`SortedDictionary<T>`类而非`SortedList<T>`类？

1.  威尔士的 ISO 文化代码是什么？

1.  本地化、全球化与国际化之间有何区别？

1.  在正则表达式中，`$`是什么意思？

1.  在正则表达式中，如何表示数字？

1.  为何不应使用电子邮件地址的官方标准来创建正则表达式以验证用户的电子邮件地址？

## 练习 8.2 – 练习正则表达式

在`Chapter08`解决方案/工作区中，创建一个名为`Exercise02`的控制台应用程序，提示用户输入正则表达式，然后提示用户输入一些输入，并比较两者是否匹配，直到用户按下*Esc*，如下所示：

```cs
The default regular expression checks for at least one digit.
Enter a regular expression (or press ENTER to use the default): ^[a-z]+$ 
Enter some input: apples
apples matches ^[a-z]+$? True
Press ESC to end or any key to try again.
Enter a regular expression (or press ENTER to use the default): ^[a-z]+$ 
Enter some input: abc123xyz
abc123xyz matches ^[a-z]+$? False
Press ESC to end or any key to try again. 
```

## 练习 8.3 – 练习编写扩展方法

在`Chapter08`解决方案/工作区中，创建一个名为`Exercise03`的类库，该库定义了扩展数字类型（如`BigInteger`和`int`）的扩展方法，该方法名为`ToWords`，返回一个描述数字的`string`；例如，`18,000,000`将是“一千八百万”，而`18,456,002,032,011,000,007`将是“一千八百五十六万万亿，二万亿，三十二亿，一千一百万，七”。

您可以在以下链接中阅读更多关于大数名称的信息：[`en.wikipedia.org/wiki/Names_of_large_numbers`](https://en.wikipedia.org/wiki/Names_of_large_numbers)

## 练习 8.4 – 探索主题

请使用以下页面上的链接，以了解更多关于本章所涵盖主题的详细信息：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-8---working-with-common-net-types`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-8---working-with-common-net-types)

# 总结

在本章中，您探索了用于存储和操作数字、日期和时间以及文本（包括正则表达式）的类型选择，以及用于存储多个项目的集合；处理了索引、范围和跨度；使用了某些网络资源；反思了代码和属性；使用微软推荐的第三方库操作图像；并学习了如何国际化您的代码。

下一章，我们将管理文件和流，编码和解码文本，并执行序列化。
