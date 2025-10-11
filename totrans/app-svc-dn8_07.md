# 7

# 处理日期、时间和国际化

本章介绍了.NET 中包含的一些常见类型。这些包括用于操作日期和时间以及实现国际化的类型，包括全球化和本地化。

当编写处理时间的代码时，特别重要的是要考虑时区。错误通常是由于没有考虑到这一点，在不同时区比较两个时间而引入的。理解**协调世界时**（**UTC**）的概念并将时间值转换为 UTC 在进行时间操作之前非常重要。

您还应该注意可能需要的任何**夏令时**（**DST**）调整。

本章涵盖了以下主题：

+   处理日期和时间

+   处理时区

+   处理文化

+   处理 Noda Time

# 处理日期和时间

在数字和文本之后，接下来最常用的数据类型是日期和时间。主要有以下两种类型：

+   `DateTime`：表示固定时间点的日期和时间组合值。

+   `TimeSpan`：表示时间的持续时间。

这两种类型通常一起使用。例如，如果您从一个`DateTime`值减去另一个，结果是`TimeSpan`。如果您将`TimeSpan`添加到`DateTime`，则结果是`DateTime`值。

## 指定日期和时间值

创建日期和时间值的一种常见方式是指定日期和时间组件的单独值，如天和小时，如*表 7.1*中所述：

| **日期/时间参数** | **值范围** |
| --- | --- |
| `year` | 1 到 9,999 |
| `month` | 1 到 12 |
| `day` | 该月的天数到 1 |
| `hour` | 0 到 23 |
| `minute` | 0 到 59 |
| `second` | 0 到 59 |
| `millisecond` | 0 到 999 |
| `microsecond` | 0 到 999 |

表 7.1：格式化日期和时间值的参数

例如，为了实例化一个表示.NET 9 可能发布为**通用可用性**时的`DateTime`，如下面的代码所示：

```cs
DateTime dotnet9GA = new(year: 2024, month: 11, day: 12,
  hour: 11, minute: 0, second: 0); 
```

**良好实践**：前面的代码示例可能会让您想，“这个值代表的是哪个时区？”这是`DateTime`的大问题，也是为什么避免使用它而选择包含时区的`DateTimeOffset`是一个好习惯。我们将在本章后面更详细地探讨这个问题。

另一种方法是提供要解析的`string`值，但这可能取决于线程的默认文化而误解。例如，在英国，日期指定为 day/month/year，而在美国，日期指定为 month/day/year。

让我们看看您可能想要如何处理日期和时间：

1.  使用您首选的代码编辑器创建一个新项目，如下列所示：

    +   项目模板：**控制台应用程序** / `console`

    +   项目文件和文件夹：`WorkingWithTime`

    +   解决方案文件和文件夹：`Chapter07`

    +   **不要使用顶级语句**：已清除。

    +   **启用原生 AOT 发布**：已清除。

1.  在项目文件中，将警告视为错误，并添加一个元素以静态和全局导入 `System.Console` 类。

1.  添加一个名为 `Program.Helpers.cs` 的新类文件，并替换其内容，如下面的代码所示：

    ```cs
    using System.Globalization; // To use CultureInfo.
    partial class Program
    {
      private static void ConfigureConsole(string culture = "en-US",
        bool overrideComputerCulture = true)
      {
        // To enable special characters like Euro currency symbol.
        OutputEncoding = System.Text.Encoding.UTF8;
        Thread t = Thread.CurrentThread;
        if (overrideComputerCulture)
        {
          t.CurrentCulture = CultureInfo.GetCultureInfo(culture);
          t.CurrentUICulture = t.CurrentCulture;
        }
        CultureInfo ci = t.CurrentCulture;
        WriteLine($"Current culture: {ci.DisplayName}");
        WriteLine($"Short date pattern: {
          ci.DateTimeFormat.ShortDatePattern}");
        WriteLine($"Long date pattern: {
          ci.DateTimeFormat.LongDatePattern}");
        WriteLine();
      }
      private static void SectionTitle(string title)
      {
        ConsoleColor previousColor = ForegroundColor;
        ForegroundColor = ConsoleColor.DarkYellow;
        WriteLine($"*** {title}");
        ForegroundColor = previousColor;
      }
    } 
    ```

1.  在 `Program.cs` 中，删除现有的语句，然后添加语句以初始化一些特殊的日期/时间值，如下面的代码所示：

    ```cs
    ConfigureConsole(); // Defaults to en-US culture.
    SectionTitle("Specifying date and time values");
    WriteLine($"DateTime.MinValue:  {DateTime.MinValue}");
    WriteLine($"DateTime.MaxValue:  {DateTime.MaxValue}");
    WriteLine($"DateTime.UnixEpoch: {DateTime.UnixEpoch}");
    WriteLine($"DateTime.Now:       {DateTime.Now}");
    WriteLine($"DateTime.Today:     {DateTime.Today}");
    WriteLine($"DateTime.Today:     {DateTime.Today:d}");
    WriteLine($"DateTime.Today:     {DateTime.Today:D}"); 
    ```

1.  运行代码，并注意结果，如下面的输出所示：

    ```cs
    Current culture: English (United States)
    Short date pattern: M/d/yyyy
    Long date pattern: dddd, MMMM d, yyyy
    *** Specifying date and time values
    DateTime.MinValue:  1/1/0001 12:00:00 AM
    DateTime.MaxValue:  12/31/9999 11:59:59 PM
    DateTime.UnixEpoch: 1/1/1970 12:00:00 AM
    DateTime.Now:       5/30/2023 9:18:05 AM
    DateTime.Today:     5/30/2023 12:00:00 AM
    DateTime.Today:     5/30/2023
    DateTime.Today:     Tuesday, May 30, 2023 
    ```

    输出的日期和时间格式由控制台应用程序的文化设置决定。我们调用了 `ConfigureConsole` 方法以确保我们都能看到相同的默认输出（美国英语）。

1.  在 `Program.cs` 中，在调用 `ConfigureConsole` 的语句顶部设置参数，以便不覆盖您的本地计算机文化，如下面的代码所示：

    ```cs
    ConfigureConsole(overrideComputerCulture: false); 
    ```

1.  运行代码，并注意输出已本地化为您的计算机文化。

1.  在 `Program.cs` 中，设置参数以指定替代语言，如加拿大法语 (`fr-CA`) 或英国英语 (`en-GB`)，如下面的代码所示：

    ```cs
    ConfigureConsole("fr-CA"); 
    ```

    **更多信息**：以下链接提供了一个常见文化代码表：[`en.wikipedia.org/wiki/Language_localisation#Language_tags_and_codes`](https://en.wikipedia.org/wiki/Language_localisation#Language_tags_and_codes)

1.  运行代码，并注意输出已本地化为指定的文化。

1.  将控制台配置重置为默认设置，以便使用美国英语文化，如下面的代码所示：

    ```cs
    ConfigureConsole(); // Defaults to en-US culture. 
    ```

## 格式化日期和时间值

您刚刚看到日期和时间有基于当前文化的默认格式。

您可以使用自定义格式代码完全控制日期和时间格式，如下表 7.2 所示：

| **格式代码** | **描述** |
| --- | --- |
| `/` | 日期部分分隔符。根据文化不同而变化；例如，`en-US` 使用 `/`, 但 `fr-FR` 使用 `-` (破折号)。 |
| `\` | 转义字符。如果您想将特殊格式代码作为字面字符使用，则很有用；例如，`h \h m \m` 将格式化为上午 9:30 的时间为 `9 h 30 m`。 |
| `:` | 时间部分分隔符。根据文化不同而变化；例如，`en-US` 使用 `:`, 但 `fr-FR` 使用 `.` (点)。 |
| `d`, `dd` | 月份中的日期，从 `1` 到 `31`，或带前导零从 `01` 到 `31`。 |
| `ddd, dddd` | 周几的缩写或全称，例如，`Mon` 或 `Monday`，根据当前文化本地化。 |
| `f`, `ff`, `fff` | 十分之一秒、百分之一秒或毫秒。 |
| `g` | 时期或纪元，例如，`A.D.` |
| `h`, `hh` | 小时，使用 12 小时制从 `1` 到 `12`，或从 `01` 到 `12`。 |
| `H`, `HH` | 小时，使用 24 小时制从 `0` 到 `23`，或从 `01` 到 `23`。 |
| `K` | 时区信息。对于未指定的时区为 `null`，对于 UTC 为 `Z`，对于从 UTC 调整的本地时间为类似 `-8:00` 的值。 |
| `m`, `mm` | 分钟，从 `0` 到 `59`，或带前导零从 `00` 到 `59`。 |
| `M`, `MM` | 月份，从 `1` 到 `12`，或带前导零从 `01` 到 `12`。 |
| `MMM`, `MMMM` | 月份的缩写或全称，例如，`Jan` 或 `January`，针对当前文化本地化。 |
| `s`, `ss` | 秒，从 `0` 到 `59`，或带前导零从 `00` 到 `59`。 |
| `t`, `tt` | AM/PM 标识符的第一个或前两个字符。 |
| `y`, `yy` | 当前世纪的年份，从 `0` 到 `99`，或带前导零从 `00` 到 `99`。 |
| `yyy` | 至少三位数的年份，最多所需位数。例如，公元 1 年是 `001`。罗马城第一次被攻陷是在 `410` 年。这本书出版的那一年是 `2023` 年。 |
| `yyyy`, `yyyyy` | 四位或五位数的年份。 |
| `z`, `zz` | 从 UTC 偏移的小时，不带前导零，或带前导零。 |
| `zzz` | 从 UTC 偏移的小时和分钟，带前导零，例如，`+04:30`。 |

表 7.2：日期和时间值的自定义格式代码

**更多信息**：有关自定义格式代码的完整列表，请参阅以下链接：[`learn.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings`](https://learn.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings)

您可以使用简单的格式代码应用标准日期和时间格式，就像我们在代码示例中使用的 `d` 和 `D` 一样，如 *表 7.3* 所示：

| **格式代码** | **描述** |
| --- | --- |
| `d` | 短日期模式。因文化而异；例如，`en-US` 使用 `M/d/yyyy`，而 `fr-FR` 使用 `dd/MM/yyyy`。 |
| `D` | 长日期模式。因文化而异；例如，`en-US` 使用 `mmmm, MMMM d, yyyy`，而 `fr-FR` 使用 `mmmm, dd MMMM yyyy`。 |
| `f` | 完整日期/时间模式（短时间 - 小时和分钟）。因文化而异。 |
| `F` | 完整日期/时间模式（长时间 – 小时、分钟、秒和 AM/PM）。因文化而异。 |
| `o, O` | 标准化模式，适用于序列化日期/时间值进行往返，例如，`2023-05-30T13:45:30.0000000-08:00`。 |
| `r`, `R` | RFC1123 模式。 |
| `t` | 短时间模式。因文化而异；例如，`en-US` 使用 `h:mm tt`，而 `fr-FR` 使用 `HH:mm`。 |
| `T` | 长时间模式。因文化而异；例如，`en-US` 使用 `h:mm:ss tt`，而 `fr-FR` 使用 `HH:mm:ss`。 |
| `u` | 通用可排序日期/时间模式，例如，`2009-06-15 13:45:30Z`。 |
| `U` | 通用完整日期/时间模式。因文化而异；例如，`en-US` 可能是 `Monday, June 15, 2009 8:45:30 PM`。 |

表 7.3：日期和时间值的标准格式代码

**更多信息**：有关格式代码的完整列表，请参阅以下链接：[`learn.microsoft.com/en-us/dotnet/standard/base-types/standard-date-and-time-format-strings`](https://learn.microsoft.com/en-us/dotnet/standard/base-types/standard-date-and-time-format-strings).

让我们运行一些示例：

1.  在`Program.cs`中添加语句来定义 2024 年的圣诞节并以各种方式显示，如下所示代码：

    ```cs
    DateTime xmas = new(year: 2024, month: 12, day: 25);
    WriteLine($"Christmas (default format): {xmas}");
    WriteLine($"Christmas (custom short format): {xmas:ddd d/M/yy}");
    WriteLine($"Christmas (custom long format): {
      xmas:dddd, dd MMMM yyyy}");
    WriteLine($"Christmas (standard long format): {xmas:D}");
    WriteLine($"Christmas (sortable): {xmas:u}");
    WriteLine($"Christmas is in month {xmas.Month} of the year.");
    WriteLine($"Christmas is day {xmas.DayOfYear} of {xmas.Year}.");
    WriteLine($"Christmas {xmas.Year} is on a {xmas.DayOfWeek}."); 
    ```

1.  运行代码，并注意结果，如下所示输出：

    ```cs
    Christmas (default format): 12/25/2024 12:00:00 AM
    Christmas (custom short format): Wed, 25/12/24
    Christmas (custom long format): Wednesday, 25 December 2024
    Christmas (standard long format): Wednesday, December 25, 2024
    Christmas (sortable): 2024-12-25 00:00:00Z
    Christmas is in month 12 of the year.
    Christmas is day 360 of 2024.
    Christmas 2024 is on a Wednesday. 
    ```

1.  禁用覆盖您的计算机文化或传递特定的文化代码，例如法国的法国文化，如下所示代码：

    ```cs
    ConfigureConsole("fr-FR"); // Defaults to en-US culture. 
    ```

1.  运行代码，并注意结果应该本地化为该文化。

1.  将控制台配置重置为默认的 US English。

## 日期和时间计算

现在，让我们尝试对日期和时间值进行简单计算：

1.  在`Program.cs`中添加语句来对 2024 年的圣诞节进行加法和减法运算，如下所示代码：

    ```cs
    SectionTitle("Date and time calculations");
    DateTime beforeXmas = xmas.Subtract(TimeSpan.FromDays(12));
    DateTime afterXmas = xmas.AddDays(12);
    WriteLine($"12 days before Christmas: {beforeXmas:d}");
    WriteLine($"12 days after Christmas: {afterXmas:d}");
    TimeSpan untilXmas = xmas - DateTime.Now;
    WriteLine($"Now: {DateTime.Now}");
    WriteLine($"There are {untilXmas.Days} days and {untilXmas.Hours
      } hours until Christmas {xmas.Year.");
    WriteLine("There are {untilXmas.TotalHours:N0} hours " +
      $"until Christmas {xmas.Year}."); 
    ```

1.  运行代码，并注意结果，如下所示输出：

    ```cs
    *** Date and time calculations
    12 days before Christmas: 12/13/2024
    12 days after Christmas: 1/6/2025
    Now: 5/30/2023 1:57:01 PM
    There are 574 days and 10 hours until Christmas 2024.
    There are 13,786 hours until Christmas 2024. 
    ```

1.  添加语句来定义孩子们（或狗、猫、鬣蜥？）可能会醒来打开礼物的圣诞节时间，并以各种方式显示，如下所示代码：

    ```cs
    DateTime kidsWakeUp = new(
      year: 2024, month: 12, day: 25, 
      hour: 6, minute: 30, second: 0);
    WriteLine($"Kids wake up: {kidsWakeUp}");
    WriteLine($"The kids woke me up at {
      kidsWakeUp.ToShortTimeString()}"); 
    ```

1.  运行代码，并注意结果，如下所示输出：

    ```cs
    Kids wake up: 25/12/2024 06:30:00 AM
    The kids woke me up at 06:30 AM 
    ```

## 微秒和纳秒

在.NET 的早期版本中，时间测量的最小单位是刻度。一个刻度是 100 纳秒，因此开发者以前必须自己进行纳秒的计算。.NET 7 为构造函数引入了毫秒和微秒参数，并将微秒和纳秒属性添加到`DateTime`、`DateTimeOffset`、`TimeSpan`和`TimeOnly`类型中。

让我们看看一些示例：

1.  在`Program.cs`中添加语句来构造一个比以前更精确的日期和时间值，并显示其值，如下所示代码：

    ```cs
    SectionTitle("Milli-, micro-, and nanoseconds");
    DateTime preciseTime = new(
      year: 2022, month: 11, day: 8,
      hour: 12, minute: 0, second: 0,
      millisecond: 6, microsecond: 999);
    WriteLine($"Millisecond: {preciseTime.Millisecond}, Microsecond: {
      preciseTime.Microsecond}, Nanosecond: {preciseTime.Nanosecond}");
    preciseTime = DateTime.UtcNow;
    // Nanosecond value will be 0 to 900 in 100 nanosecond increments.
    WriteLine($"Millisecond: {preciseTime.Millisecond}, Microsecond: {
      preciseTime.Microsecond}, Nanosecond: {preciseTime.Nanosecond}"); 
    ```

1.  运行代码，并注意结果，如下所示输出：

    ```cs
    *** Milli-, micro-, and nanoseconds
    Millisecond: 6, Microsecond: 999, Nanosecond: 0
    Millisecond: 243, Microsecond: 958, Nanosecond: 400 
    ```

## 日期和时间的全球化

当前文化控制日期和时间的格式化和解析方式：

1.  在`Program.cs`顶部，导入用于全球化操作的命名空间，如下所示代码：

    ```cs
    using System.Globalization; // To use CultureInfo. 
    ```

1.  添加语句来显示用于显示日期和时间值的当前文化，然后解析美国的独立日并以各种方式显示，如下所示代码：

    ```cs
    SectionTitle("Globalization with dates and times");
    // Same as Thread.CurrentThread.CurrentCulture.
    WriteLine($"Current culture: {CultureInfo.CurrentCulture.Name}");
    string textDate = "4 July 2024";
    DateTime independenceDay = DateTime.Parse(textDate);
    WriteLine($"Text: {textDate}, DateTime: {independenceDay:d MMMM}");
    textDate = "7/4/2024";
    independenceDay = DateTime.Parse(textDate);
    WriteLine($"Text: {textDate}, DateTime: {independenceDay:d MMMM}");
    // Explicitly override the current culture by setting a provider.
    independenceDay = DateTime.Parse(textDate,
      provider: CultureInfo.GetCultureInfo("en-US"));
    WriteLine($"Text: {textDate}, DateTime: {independenceDay:d MMMM}"); 
    ```

    **良好实践**：虽然您可以使用构造函数创建`CultureInfo`实例，除非您需要对其进行更改，否则您应该通过调用`GetCultureInfo`方法来获取只读共享实例。

1.  在`Program.cs`顶部，将文化设置为英国英语，如下所示代码：

    ```cs
    ConfigureConsole("en-GB"); 
    ```

1.  运行代码，并注意结果，如下所示输出：

    ```cs
    *** Globalization with dates and times
    Current culture is: en-GB
    Text: 4 July 2024, DateTime: 4 July
    Text: 7/4/2024, DateTime: 7 April
    Text: 7/4/2024, DateTime: 4 July 
    ```

    当当前文化设置为*英语（英国）*时，如果给定日期为 2024 年 7 月 4 日，则无论当前文化是英国还是美国，都能正确解析。但如果日期给定为`7/4/2024`，则解析为 4 月 7 日。在解析时，可以通过指定正确的文化作为提供者来覆盖当前文化，如上面第三个示例所示。

1.  添加语句从 2023 年循环到 2028 年，显示该年是否是闰年以及二月有多少天，然后显示圣诞节和独立日是否在夏令时期间，如下所示代码：

    ```cs
    for (int year = 2023; year <= 2028; year++)
    {
      Write($"{year} is a leap year: {DateTime.IsLeapYear(year)}. ");
      WriteLine($"There are {DateTime.DaysInMonth(year: year, month: 2)
        } days in February {year}.");
    }
    WriteLine($"Is Christmas daylight saving time? {
      xmas.IsDaylightSavingTime()}");
    WriteLine($"Is July 4th daylight saving time? {
      independenceDay.IsDaylightSavingTime()}"); 
    ```

1.  运行代码，并注意结果，如下所示输出：

    ```cs
    2023 is a leap year: False. There are 28 days in February 2023.
    2024 is a leap year: True. There are 29 days in February 2024.
    2025 is a leap year: False. There are 28 days in February 2025.
    2026 is a leap year: False. There are 28 days in February 2026.
    2027 is a leap year: False. There are 28 days in February 2027.
    2028 is a leap year: True. There are 29 days in February 2028.
    Is Christmas daylight saving time? False
    Is July 4th daylight saving time? True 
    ```

## 夏令时（DST）的复杂性

夏令时不是所有国家都使用；它也由半球决定，政治也起着作用。例如，美国目前正在辩论是否应该使夏令时永久化。他们可能会决定将决定权留给各州。在接下来的几年里，这可能会让美国人感到更加困惑。

每个国家都有自己的规则来决定夏令时（DST）在什么日子和什么时间开始。这些规则被.NET 编码，以便在需要时自动调整。

在美国的春季，时钟在凌晨 2 点“跳”前一小时。在秋季，它们在凌晨 2 点“退”后一小时。维基百科在以下链接中解释了这一点：[`en.wikipedia.org/wiki/Daylight_saving_time_in_the_United_States`](https://en.wikipedia.org/wiki/Daylight_saving_time_in_the_United_States)

在英国的春季，时钟在凌晨 1 点“跳”前一小时。在秋季，它们在凌晨 2 点“退”后一小时。英国政府在以下链接中解释了这一点：[`www.gov.uk/when-do-the-clocks-change`](https://www.gov.uk/when-do-the-clocks-change)

假设你需要设置一个闹钟在凌晨 1:30 AM 醒来，以便从英国的希思罗机场赶飞机。不幸的是，你的航班恰好是在夏令时生效的那天出发。

在英国的春季，时钟显示为凌晨 12:59，然后下一分钟它们会跳到凌晨 2:00。1:30 AM 永远不会发生，你的闹钟不会响，你可能会错过航班！在.NET 中，1:30 AM 是一个无效的时间，如果你尝试将这个值存储在变量中，它将抛出一个异常。

在英国的秋季，时钟显示为 1:59，然后下一分钟，它们会退回到 1:00 并重复那个小时。在这种情况下，1:30 AM 会发生两次。

## 本地化 DayOfWeek 枚举

`DayOfWeek`是一个`enum`，所以它不能像你预期或希望的那样本地化。它的`string`值是硬编码在英语中的，如下所示代码：

```cs
namespace System
{
  public enum DayOfWeek
  {
    Sunday = 0,
    Monday = 1,
    Tuesday = 2,
    Wednesday = 3,
    Thursday = 4,
    Friday = 5,
    Saturday = 6
  }
} 
```

解决这个问题的有两个方案。首先，你可以将`dddd`日期格式代码应用到整个日期值上。例如，

```cs
WriteLine($"The day of the week is {DateTime.Now:dddd}."); 
```

其次，你可以使用`DateTimeFormatInfo`类的辅助方法将`DayOfWeek`值转换为本地化的`string`，以便作为文本输出。

让我们看看一个问题和解决方案的例子：

1.  在`Program.cs`中添加语句，显式地将当前文化设置为丹麦语，然后输出该文化中的当前星期几，如下所示代码：

    ```cs
    SectionTitle("Localizing the DayOfWeek enum");
    CultureInfo previousCulture = Thread.CurrentThread.CurrentCulture;
    // Explicitly set culture to Danish (Denmark).
    Thread.CurrentThread.CurrentCulture = 
      CultureInfo.GetCultureInfo("da-DK");
    // DayOfWeek is not localized to Danish.
    WriteLine("Culture: {Thread.CurrentThread.CurrentCulture
      .NativeName}, DayOfWeek: {DateTime.Now.DayOfWeek}";
    // Use dddd format code to get day of the week localized.
    WriteLine($"Culture: {Thread.CurrentThread.CurrentCulture
      .NativeName}, DayOfWeek: {DateTime.Now:dddd}");
    // Use GetDayName method to get day of the week localized.
    WriteLine("Culture: {Thread.CurrentThread.CurrentCulture
      .NativeName}, DayOfWeek: {DateTimeFormatInfo.CurrentInfo
      .GetDayName(DateTime.Now.DayOfWeek)}");
    Thread.CurrentThread.CurrentCulture = previousCulture; 
    ```

1.  运行代码，并注意结果，如下所示输出：

    ```cs
    *** Localizing the DayOfWeek enum
    Culture: dansk (Danmark), DayOfWeek: Thursday
    Culture: dansk (Danmark), DayOfWeek: torsdag
    Culture: dansk (Danmark), DayOfWeek: torsdag 
    ```

## 仅处理日期或时间

.NET 6 引入了一些新类型，用于仅处理日期值或仅处理时间值，分别命名为`DateOnly`和`TimeOnly`。

这些比使用具有零时间的 `DateTime` 值来存储仅日期的值要好，因为它类型安全且避免了误用。`DateOnly` 也更好地映射到数据库列类型，例如 SQL Server 中的 `date` 列。`TimeOnly` 适用于设置闹钟和安排定期会议或组织的营业时间，它映射到 SQL Server 中的 `time` 列。

让我们使用它们来计划 .NET 9 的发布派对，可能是在 2024 年 11 月 12 日星期二，美国总统选举后一周：

1.  在 `Program.cs` 中，添加语句以定义 .NET 9 发布派对及其开始时间，然后将这两个值组合成一个日历条目，以免错过，如下面的代码所示：

    ```cs
    SectionTitle("Working with only a date or a time");
    DateOnly party = new(year: 2024, month: 11, day: 12);
    WriteLine($"The .NET 9 release party is on {party.ToLongDateString()}.");
    TimeOnly starts = new(hour: 11, minute: 30);
    WriteLine($"The party starts at {starts}.");
    DateTime calendarEntry = party.ToDateTime(starts);
    WriteLine($"Add to your calendar: {calendarEntry}."); 
    ```

1.  运行代码并注意结果，如下面的输出所示：

    ```cs
    *** Working with only a date or a time
    The .NET 9 release party is on Tuesday, November 12, 2024.
    The party starts at 11:30 AM.
    Add to your calendar: 11/12/2024 11:30:00 AM. 
    ```

## 获取日期/时间格式化信息

每个文化都有自己的日期/时间格式化规则。这些规则定义在 `CultureInfo` 实例的 `DateTimeFormat` 属性中。

让我们输出一些常用信息：

1.  在 `Program.cs` 中，添加语句以获取当前文化的日期/时间格式化信息并输出其中一些最有用的属性，如下面的代码所示：

    ```cs
    SectionTitle("Working with date/time formats");
    DateTimeFormatInfo dtfi = DateTimeFormatInfo.CurrentInfo;
    // Or use Thread.CurrentThread.CurrentCulture.DateTimeFormat.
    WriteLine($"Date separator: {dtfi.DateSeparator}");
    WriteLine($"Time separator: {dtfi.TimeSeparator}");
    WriteLine($"Long date pattern: {dtfi.LongDatePattern}");
    WriteLine($"Short date pattern: {dtfi.ShortDatePattern}");
    WriteLine($"Long time pattern: {dtfi.LongTimePattern}");
    WriteLine($"Short time pattern: {dtfi.ShortTimePattern}");
    Write("Day names:");
    for (int i = 0; i < dtfi.DayNames.Length - 1; i++)
    {
      Write($"  {dtfi.GetDayName((DayOfWeek)i)}");
    }
    WriteLine();
    Write("Month names:");
    for (int i = 1; i < dtfi.MonthNames.Length; i++)
    {
      Write($"  {dtfi.GetMonthName(i)}");
    }
    WriteLine(); 
    ```

1.  运行代码，并注意结果，如下面的输出所示：

    ```cs
    *** Working with date/time formats
    Date separator: /
    Time separator: :
    Long date pattern: dddd, MMMM d, yyyy
    Short date pattern: M/d/yyyy
    Long time pattern: h:mm:ss tt
    Short time pattern: h:mm tt
    Day names:  Sunday  Monday  Tuesday  Wednesday  Thursday  Friday
    Month names:  January  February  March  April  May  June  July  August  September  October  November  December 
    ```

1.  将文化更改为其他，运行代码并注意结果。

## 使用时间提供程序进行单元测试

为需要当前时间的组件编写单元测试很棘手，因为时间是不断变化的！

假设你希望电子商务网站的访客在周末下单时获得 20% 的折扣。在工作日，他们需要支付全价。我们如何测试这个功能？

为了控制单元测试中使用的时间，.NET 8 引入了 `TimeProvider` 类。

让我们定义一个执行此计算的功能：

1.  在 `Chapter07` 解决方案中，添加一个名为 `TimeFunctionsLib` 的新 **类库**/`classlib` 项目。

1.  在 `TimeFunctionsLib` 项目中，将 `Class1.cs` 重命名为 `DiscountService.cs`。

1.  在 `DiscountService.cs` 中，定义一个执行计算的功能，如下面的代码所示：

    ```cs
    namespace Northwind.Services;
    public class DiscountService
    {
      public decimal GetDiscount()
      {
        // This has a dependency on the current time provided by the system.
        var now = DateTime.UtcNow;
        return now.DayOfWeek switch
        {
          DayOfWeek.Saturday or DayOfWeek.Sunday => 0.2M,
          _ => 0M
        };
      }
    } 
    ```

1.  在 `Chapter07` 解决方案中，添加一个名为 `TestingWithTimeProvider` 的新 **xUnit 测试项目**/`xunit` 项目。

1.  在 `TestingWithTimeProvider` 项目中，添加对 `TimeFunctionsLib` 项目的引用，如下面的标记所示：

    ```cs
    <ItemGroup>
      <ProjectReference Include=
        "..\TimeFunctionsLib\TimeFunctionsLib.csproj" />
    </ItemGroup> 
    ```

1.  构建名为 `TestingWithTimeProvider` 的项目。

1.  在 `TestingWithTimeProvider` 项目中，将 `Test1.cs` 重命名为 `TimeTests.cs`。

1.  在 `TimeTests.cs` 文件中，修改语句以导入折扣服务的命名空间，然后定义两个测试，一个用于工作日，一个用于周末，如下面的代码所示：

    ```cs
    using Northwind.Services; // To use DiscountService.
    namespace TestingWithTimeProvider;
    public class TimeTests
    {
      [Fact]
      public void TestDiscountDuringWorkdays()
      {
        // Arrange
        DiscountService service = new();
        // Act
        decimal discount = service.GetDiscount();
        // Assert
        Assert.Equal(0M, discount);
      }
      [Fact]
      public void TestDiscountDuringWeekends()
      {
        DiscountService service = new();
        decimal discount = service.GetDiscount();
        Assert.Equal(0.2M, discount);
      }
    } 
    ```

1.  运行两个测试，并注意在任何时候只能成功运行一个测试。如果你在工作日运行测试，周末测试将失败。如果你在周末运行测试，工作日测试将失败！

既然你已经看到了问题，我们该如何解决它？

微软解决这个问题的方法是，每个创建.NET 库的团队定义自己的内部`ISystemClock`接口，至少包含一个名为`UtcNow`的属性，有时还有其他成员，以及通常使用内置系统时钟但略有不同的实现。以下是一个典型的示例代码：

```cs
using System;
namespace Microsoft.Extensions.Internal
{
  public interface ISystemClock
  {
    DateTimeOffset UtcNow { get; }
  }
  public class SystemClock
  {
    public DateTimeOffset UtcNow 
    {
      return DateTimeOffset.UtcNow;
    }
  }
} 
```

最后，随着.NET 8 的推出，.NET 核心团队引入了前面代码的适当等效实现，该实现使用系统时钟。不幸的是，他们没有定义一个接口。相反，他们定义了一个名为`TimeProvider`的抽象类。

让我们使用它：

1.  在`TimeFunctionsLib`项目中，在`DiscountService.cs`文件中，注释掉使用`UtcNow`属性的部分，并添加一个语句来添加一个构造函数注入的服务，如下面的代码所示：

    ```cs
    namespace Northwind.Services;
    public class DiscountService
    {
    **private** **TimeProvider _timeProvider;**
    **public****DiscountService****(****TimeProvider timeProvider****)**
     **{**
     **_timeProvider = timeProvider;**
     **}**
      public decimal GetDiscount()
      {
        // This has a dependency on the current time provided by the system.
        **//** var now = DateTime.UtcNow;
    **var** **now = _timeProvider.GetUtcNow();**
        // This has a dependency on the current time provided by the system.
        return now.DayOfWeek switch
        {
          DayOfWeek.Saturday or DayOfWeek.Sunday => 0.2M,
          _ => 0M
        };
      }
    } 
    ```

1.  在`TestingWithTimeProvider`项目中，在`TimeTests.cs`文件中，为两个测试添加语句以展示如何使用新的`TimeProvider`及其`System`属性（这仍然依赖于系统时钟！），如下面的代码所示：

    ```cs
    // This would use the .NET 8 or later dependency service,
    // but its implementation is still the system clock.
    DiscountService service = new(TimeProvider.System); 
    ```

1.  在`TestingWithTimeProvider`项目中，添加对`Moq`的引用，这是一个用于模拟依赖项的包，如下面的标记所示：

    ```cs
    <!-- The newest version before the controversy. -->
    <PackageReference Include="Moq" Version="4.18.4" /> 
    ```

    Moq 4.18.4 是在开发者添加在构建期间执行的混淆代码后引发争议之前的最后一个版本。你可以在以下链接中了解更多信息：[`github.com/devlooped/moq/issues/1370`](https://github.com/devlooped/moq/issues/1370)。我计划在未来几个月内密切关注这种情况，然后决定是否应该切换到替代方案。

1.  在`TimeTests.cs`文件中，导入命名空间以使用`Mock.Of<T>`扩展方法，如下面的代码所示：

    ```cs
    using Moq; // To use Mock.Of<T> method. 
    ```

1.  在`TestDiscountDuringWorkdays`方法中，注释掉使用`System`提供者的语句，并用以下代码中的语句替换，以模拟一个在工作日总是返回固定日期和时间的时钟提供者：

    ```cs
    TimeProvider timeProvider = Mock.Of<TimeProvider>();
    // Mock the time provider so it always returns the date of
    // 2023-11-07 09:30:00 UTC which is a Tuesday.
    Mock.Get(timeProvider).Setup(s => s.GetUtcNow()).Returns(
      new DateTimeOffset(year: 2023, month: 11, day: 7, 
      hour: 9, minute: 30, second: 0, offset: TimeSpan.Zero));
    DiscountService service = new(timeProvider); 
    ```

1.  在`TestDiscountDuringWeekends`方法中，注释掉使用`System`提供者的语句，并用以下代码中的语句替换，以模拟一个在周末总是返回固定日期和时间的时钟提供者：

    ```cs
    TimeProvider timeProvider = Mock.Of<TimeProvider>();
    // Mock the time provider so it always returns the date of
    // 2023-11-04 09:30:00 UTC which is a Saturday.
    Mock.Get(timeProvider).Setup(s => s.GetUtcNow()).Returns(
      new DateTimeOffset(year: 2023, month: 11, day: 4, 
      hour: 9, minute: 30, second: 0, offset: TimeSpan.Zero));
    DiscountService service = new(timeProvider); 
    ```

1.  运行单元测试，并注意它们都成功了。

# 与时区一起工作

在关于.NET 发布派对的代码示例中，使用`TimeOnly`实际上并不是一个好主意，因为`TimeOnly`值没有包含时区信息。只有在你处于正确的时区时才有用。因此，`TimeOnly`对于事件来说是一个较差的选择。对于事件，我们需要理解和处理时区。

## 理解`DateTime`和`TimeZoneInfo`

`DateTime`类有许多与时区相关的重要成员，如下表 7.4 所示：

| **成员** | **描述** |
| --- | --- |
| `Now`属性 | 表示本地时区的当前日期和时间的`DateTime`值。 |
| `UtcNow` 属性 | 表示 UTC 时区的当前日期和时间的 `DateTime` 值。 |
| `Kind` 属性 | 表示 `DateTime` 值是 `Unspecified`、`Utc` 还是 `Local` 的 `DateTimeKind` 值。 |
| `IsDaylightSavingTime` 方法 | 一个 `bool`，指示 `DateTime` 值是否在夏令时（DST）。 |
| `ToLocalTime` 方法 | 将 UTC `DateTime` 值转换为等效的本地时间。 |
| `ToUniversalTime` 方法 | 将本地 `DateTime` 值转换为等效的 UTC 时间。 |

表 7.4：与时区相关的 DateTime 成员

`TimeZoneInfo` 类具有许多有用的成员，如 *表 7.5* 所示：

| **成员** | **描述** |
| --- | --- |
| `Id` 属性 | 一个 `string`，唯一标识时区。 |
| `Local` 属性 | 表示当前本地时区的 `TimeZoneInfo` 值。取决于代码执行的地点。 |
| `Utc` 属性 | 表示 UTC 时区的 `TimeZoneInfo` 值。 |
| `StandardName` 属性 | 当夏令时不激活时，表示时区名称的 `string`。 |
| `DaylightName` 属性 | 当夏令时激活时，表示时区名称的 `string`。 |
| `DisplayName` 属性 | 表示时区的通用名称的 `string`。 |
| `BaseUtcOffset` 属性 | 表示此时区与 UTC 时区之间的差异的 `TimeSpan`，忽略任何潜在的夏令时调整。 |
| `SupportsDaylightSavingTime` 属性 | 一个 `bool` 值，指示此时区是否有夏令时调整。 |
| `ConvertTime` 方法 | 将 `DateTime` 值转换为另一个时区的 `DateTime` 值。您可以指定源时区和目标时区。 |
| `ConvertTimeFromUtc` 方法 | 将 UTC 时区的 `DateTime` 值转换为指定时区的 `DateTime` 值。 |
| `ConvertTimeToUtc` 方法 | 将指定时区的 `DateTime` 值转换为 UTC 时区的 `DateTime` 值。 |
| `IsDaylightSavingTime` 方法 | 返回一个 `bool`，指示 `DateTime` 值是否在夏令时。 |
| `GetSystemTimeZones` 方法 | 返回操作系统注册的时区集合。 |

表 7.5：TimeZoneInfo 有用成员

一些 EF Core 数据库提供者仅允许您存储使用 `Kind` 属性确定是否为 UTC 的 `DateTime` 值，因此如果您需要处理这些值，可能需要将它们转换为 `DateTimeOffset`。

## 探索 DateTime 和 TimeZoneInfo

使用 `TimeZoneInfo` 类处理时区：

1.  使用您首选的代码编辑器将名为 `WorkingWithTimeZones` 的新 **控制台应用程序**/ `console` 项目添加到 `Chapter07` 解决方案中：

    1.  在 Visual Studio 2022 中，将 **启动项目** 设置为 **当前选择**。

    1.  将警告视为错误，并静态和全局导入 `System.Console` 类。

1.  添加一个名为 `Program.Helpers.cs` 的新类文件。

1.  修改其内容以定义一些辅助方法，以视觉上不同的方式输出一个部分标题，输出当前系统中的所有时区列表，以及输出关于`DateTime`或`TimeZoneInfo`对象的详细信息，如下面的代码所示：

    ```cs
    using System.Collections.ObjectModel; // To use ReadOnlyCollection<T>
    partial class Program
    {
      private static void SectionTitle(string title)
      {
        ConsoleColor previousColor = ForegroundColor;
        ForegroundColor = ConsoleColor.DarkYellow;
        WriteLine($"*** {title}");
        ForegroundColor = previousColor;
      }
      private static void OutputTimeZones()
      {
        // get the time zones registered with the OS
        ReadOnlyCollection<TimeZoneInfo> zones = 
          TimeZoneInfo.GetSystemTimeZones();
        WriteLine($"*** {zones.Count} time zones:");
        // order the time zones by Id instead of DisplayName
        foreach (TimeZoneInfo zone in zones.OrderBy(z => z.Id))
        {
          WriteLine($"{zone.Id}");
        }
      }
      private static void OutputDateTime(DateTime dateTime, string title)
      {
        SectionTitle(title);
        WriteLine($"Value: {dateTime}");
        WriteLine($"Kind: {dateTime.Kind}");
        WriteLine($"IsDaylightSavingTime: {dateTime.IsDaylightSavingTime()}");
        WriteLine($"ToLocalTime(): {dateTime.ToLocalTime()}");
        WriteLine($"ToUniversalTime(): {dateTime.ToUniversalTime()}");
      }
      private static void OutputTimeZone(TimeZoneInfo zone, string title)
      {
        SectionTitle(title);
        WriteLine($"Id: {zone.Id}");
        WriteLine($"IsDaylightSavingTime(DateTime.Now): {
          zone.IsDaylightSavingTime(DateTime.Now)}");
        WriteLine($"StandardName: {zone.StandardName}");
        WriteLine($"DaylightName: {zone.DaylightName}");
        WriteLine($"BaseUtcOffset: {zone.BaseUtcOffset}");
      }
      private static string GetCurrentZoneName(TimeZoneInfo zone, DateTime when)
      {
        // time zone names change if Daylight Saving time is active
        // e.g. GMT Standard Time becomes GMT Summer Time
        return zone.IsDaylightSavingTime(when) ?
          zone.DaylightName : zone.StandardName;
      }
    } 
    ```

1.  在`Program.cs`中，删除现有的语句。添加语句以输出本地和 UTC 时区的当前日期和时间，然后输出本地和 UTC 时区的详细信息，如下面的代码所示：

    ```cs
    OutputTimeZones();
    OutputDateTime(DateTime.Now, "DateTime.Now");
    OutputDateTime(DateTime.UtcNow, "DateTime.UtcNow");
    OutputTimeZone(TimeZoneInfo.Local, "TimeZoneInfo.Local");
    OutputTimeZone(TimeZoneInfo.Utc, "TimeZoneInfo.Utc"); 
    ```

1.  运行控制台应用程序并注意结果，包括在您的操作系统上注册的时区（在我的 Windows 11 笔记本电脑上有 141 个），以及目前是 2022 年 5 月 31 日下午 4:17 在英格兰，这意味着我处于 GMT 标准时区。然而，由于夏令时正在活跃，它目前被称为 GMT 夏令时，比 UTC 快一个小时，如下面的输出所示：

    ```cs
    *** 141 time zones:
    Afghanistan Standard Time
    Alaskan Standard Time
    ...
    West Pacific Standard Time
    Yakutsk Standard Time
    Yukon Standard Time
    *** DateTime.Now
    Value: 31/05/2022 16:17:03
    Kind: Local
    IsDaylightSavingTime: True
    ToLocalTime(): 31/05/2022 16:17:03
    ToUniversalTime(): 31/05/2022 15:17:03
    *** DateTime.UtcNow
    Value: 31/05/2022 15:17:03
    Kind: Utc
    IsDaylightSavingTime: False
    ToLocalTime(): 31/05/2022 16:17:03
    ToUniversalTime(): 31/05/2022 15:17:03
    *** TimeZoneInfo.Local
    Id: GMT Standard Time
    IsDaylightSavingTime(DateTime.Now): True
    StandardName: GMT Standard Time
    DaylightName: GMT Summer Time
    BaseUtcOffset: 00:00:00
    *** TimeZoneInfo.Utc
    Id: UTC
    IsDaylightSavingTime(DateTime.Now): False
    StandardName: Coordinated Universal Time
    DaylightName: Coordinated Universal Time
    BaseUtcOffset: 00:00:00 
    ```

    **GMT 标准时间**时区的`BaseUtcOffset`为零，因为通常夏令时不活跃。这就是为什么它前面有`Base`前缀。

1.  在`Program.cs`中，添加语句提示用户输入一个时区（使用东部标准时间作为默认值），获取该时区，输出其详细信息，然后比较用户输入的时间与另一个时区的等效时间，并捕获潜在的异常，如下面的代码所示：

    ```cs
    Write("Enter a time zone or press Enter for US East Coast: ");
    string zoneId = ReadLine()!;
    if (string.IsNullOrEmpty(zoneId))
    {
      zoneId = "Eastern Standard Time";
    }
    try
    {
      TimeZoneInfo otherZone = TimeZoneInfo.FindSystemTimeZoneById(zoneId);
      OutputTimeZone(otherZone,
        $"TimeZoneInfo.FindSystemTimeZoneById(\"{zoneId}\")");
      SectionTitle($"What's the time in {zoneId}?");
      Write("Enter a local time or press Enter for now: ");
      string? timeText = ReadLine();
      DateTime localTime;
      if (string.IsNullOrEmpty(timeText) || 
        !DateTime.TryParse(timeText, out localTime))
      {
        localTime = DateTime.Now;
      }
      DateTime otherZoneTime = TimeZoneInfo.ConvertTime(
        dateTime: localTime, sourceTimeZone: TimeZoneInfo.Local,
        destinationTimeZone: otherZone);
      WriteLine($"{localTime} {GetCurrentZoneName(TimeZoneInfo.Local,
        localTime)} is {otherZoneTime} {GetCurrentZoneName(otherZone,
        otherZoneTime)}.");
    }
    catch (TimeZoneNotFoundException)
    {
      WriteLine($"The {zoneId} zone cannot be found on the local system.");
    }
    catch (InvalidTimeZoneException)
    {
      WriteLine($"The {zoneId} zone contains invalid or missing data.");
    }
    catch (System.Security.SecurityException)
    {
      WriteLine("The application does not have permission to read time zone information.");
    }
    catch (OutOfMemoryException)
    {
      WriteLine($"Not enough memory is available to load information on the {zoneId} zone.");
    } 
    ```

1.  运行控制台应用程序，按*Enter*键选择美国东部时间，然后输入`12:30pm`作为当地时间，并注意以下输出结果：

    ```cs
    Enter a time zone or press Enter for US East Coast:
    *** TimeZoneInfo.FindSystemTimeZoneById("Eastern Standard Time")
    Id: Eastern Standard Time
    IsDaylightSavingTime(DateTime.Now): True
    StandardName: Eastern Standard Time
    DaylightName: Eastern Summer Time
    BaseUtcOffset: -05:00:00
    *** What's the time in Eastern Standard Time?
    Enter a local time or press Enter for now: 12:30pm
    31/05/2023 12:30:00 GMT Summer Time is 31/05/2023 07:30:00 Eastern Summer Time. 
    ```

    我的本地时区是 GMT 标准时间，因此目前我与美国东部时间之间有五个小时的时差。您的本地时区可能会有所不同。

1.  运行控制台应用程序，将一个时区复制到剪贴板，在提示符中粘贴它，然后按*Enter*键输入当地时间。注意以下输出结果：

    ```cs
    Enter a time zone or press Enter for US East Coast: AUS Eastern Standard Time
    *** TimeZoneInfo.FindSystemTimeZoneById("AUS Eastern Standard Time")
    Id: AUS Eastern Standard Time
    IsDaylightSavingTime(DateTime.Now): False
    StandardName: AUS Eastern Standard Time
    DaylightName: AUS Eastern Summer Time
    BaseUtcOffset: 10:00:00
    *** What's the time in AUS Eastern Standard Time?
    Enter a local time or press Enter for now:
    31/05/2023 17:00:04 GMT Summer Time is 01/06/2023 02:00:04 AUS Eastern Standard Time. 
    ```

澳大利亚的悉尼目前比我们快九个小时，所以对我来说大约是下午 5 点，对他们来说大约是第二天凌晨 2 点。

这需要学习很多关于日期、时间和时区的内容。但我们还没有完成。现在，我们需要关注更广泛的文化主题，这些文化是语言和区域的组合，并不仅仅影响日期和时间格式。

# 与文化打交道

国际化是使您的代码能够在全球范围内正确运行的过程。它有两个部分，**全球化**和**本地化**，它们都与处理文化有关。

全球化涉及编写代码以适应多种语言和区域组合。语言和区域的组合称为文化。对于您的代码来说，了解语言和区域都很重要，因为例如，尽管魁北克和巴黎都使用法语，但它们的日期和货币格式是不同的。

对于所有文化组合，都有国际标准化组织（**ISO**）代码。例如，在代码 `da-DK` 中，`da` 表示丹麦语言，`DK` 表示丹麦地区，而在代码 `fr-CA` 中，`fr` 表示法语，`CA` 表示加拿大地区。

ISO 不仅仅是一个缩写。ISO 是指希腊单词 *isos*（意为 *平等*）。您可以在以下链接中查看 ISO 文化代码列表：[`lonewolfonline.net/list-net-culture-country-codes/`](https://lonewolfonline.net/list-net-culture-country-codes/)。

本地化是关于定制用户界面以支持一种语言，例如，将按钮标签更改为 **关闭**（`en`）或 **Fermer**（`fr`）。由于本地化更多地涉及语言，因此它通常不需要了解地区，尽管讽刺的是，单词 *standardization*（`en-US`）和 *standardisation*（`en-GB`）似乎暗示了相反的情况。

**良好实践**：我不是专业软件用户界面翻译员，因此请将本章中的所有示例视为一般性指导。我对法语用户界面标签常见实践的研究使我找到了以下链接，但如果您不是母语人士，最好还是聘请专业人士：[`french.stackexchange.com/questions/12969/translation-of-it-terms-like-close-next-search-etc`](https://french.stackexchange.com/questions/12969/translation-of-it-terms-like-close-next-search-etc) 和 [`www.linguee.com/english-french/translation/close+button.html`](https://www.linguee.com/english-french/translation/close+button.html)。

## 检测和更改当前文化

国际化是一个巨大的主题，有成千上万页的书籍被撰写。在本节中，您将简要了解基础知识，使用 `System.Globalization` 命名空间中的 `CultureInfo` 和 `RegionInfo` 类型。

让我们编写一些代码：

1.  使用您首选的代码编辑器，将一个名为 `WorkingWithCultures` 的新 **控制台应用程序**/ `console` 项目添加到 `Chapter07` 解决方案中。

    +   在项目文件中，将警告视为错误，然后全局导入 `System.Console` 类，并全局导入 `System.Globalization` 命名空间，以便我们可以使用 `CultureInfo` 类，如下面的标记所示：

    ```cs
    <ItemGroup>
      <Using Include="System.Console" Static="true" />
      <Using Include="System.Globalization" />
    </ItemGroup> 
    ```

1.  添加一个名为 `Program.Helpers.cs` 的新类文件，并修改其内容以向部分 `Program` 类添加一个方法，该方法将输出有关用于全球化和本地化的文化信息，如下面的代码所示：

    ```cs
    partial class Program
    {
      private static void OutputCultures(string title)
      {
        ConsoleColor previousColor = ForegroundColor;
        ForegroundColor = ConsoleColor.DarkYellow;
        WriteLine($"*** {title}");
        // Get the cultures from the current thread.
        CultureInfo globalization = CultureInfo.CurrentCulture;
        CultureInfo localization = CultureInfo.CurrentUICulture;
        WriteLine($"The current globalization culture is {
          globalization.Name}: {globalization.DisplayName}");
        WriteLine($"The current localization culture is {
          localization.Name}: {localization.DisplayName}");
        WriteLine($"Days of the week: {string.Join(", ",
          globalization.DateTimeFormat.DayNames)}");
        WriteLine($"Months of the year: {string.Join(", ",
          globalization.DateTimeFormat.MonthNames
          // Some have 13 months; most 12, and the last is empty.
          .TakeWhile(month => !string.IsNullOrEmpty(month)))}");
        WriteLine($"1st day of this year: {new DateTime(
          year: DateTime.Today.Year, month: 1, day: 1)
          .ToString("D", globalization)}");
        WriteLine($"Number group separator: {globalization
          .NumberFormat.NumberGroupSeparator}");
        WriteLine($"Number decimal separator: {globalization
          .NumberFormat.NumberDecimalSeparator}");
        RegionInfo region = new(globalization.LCID);
        WriteLine($"Currency symbol: {region.CurrencySymbol}");
        WriteLine($"Currency name: {region.CurrencyNativeName} ({
          region.CurrencyEnglishName})");
        WriteLine($"IsMetric: {region.IsMetric}");
        WriteLine();
        ForegroundColor = previousColor;
      }
    } 
    ```

1.  在 `Program.cs` 文件中，删除现有的语句，并添加语句以设置控制台输出编码以支持 Unicode。然后，输出有关全球化本地化文化的信息。最后，提示用户输入一个新的文化代码，并展示它如何影响常见值（如日期和货币）的格式化，如下面的代码所示：

    ```cs
    // To enable special characters like €.
    OutputEncoding = System.Text.Encoding.UTF8;
    OutputCultures("Current culture");
    WriteLine("Example ISO culture codes:");
    string[] cultureCodes = { 
      "da-DK", "en-GB", "en-US", "fa-IR", 
      "fr-CA", "fr-FR", "he-IL", "pl-PL", "sl-SI" };
    foreach (string code in cultureCodes)
    {
      CultureInfo culture = CultureInfo.GetCultureInfo(code);
      WriteLine($"  {culture.Name}: {culture.EnglishName} / {
        culture.NativeName}");
    }

    WriteLine();
    Write("Enter an ISO culture code: ");
    string? cultureCode = ReadLine();
    if (string.IsNullOrWhiteSpace(cultureCode))
    {
      cultureCode = "en-US";
    }  
    CultureInfo ci;
    try
    {
      ci = CultureInfo.GetCultureInfo(cultureCode);
    }
    catch (CultureNotFoundException)
    {
      WriteLine($"Culture code not found: {cultureCode}");
      WriteLine("Exiting the app.");
      return;
    }
    // change the current cultures on the thread
    CultureInfo.CurrentCulture = ci;
    CultureInfo.CurrentUICulture = ci;
    OutputCultures("After changing the current culture");
    Write("Enter your name: ");
    string? name = ReadLine();
    if (string.IsNullOrWhiteSpace(name))
    {
      name = "Bob";
    }
    Write("Enter your date of birth: ");
    string? dobText = ReadLine();
    if (string.IsNullOrWhiteSpace(dobText))
    {
      // If they do not enter a DOB then use
      // sensible defaults for their culture.
      dobText = ci.Name switch
        {
          "en-US" or "fr-CA" => "1/27/1990",
          "da-DK" or "fr-FR" or "pl-PL" => "27/1/1990",
          "fa-IR" => "1990/1/27",
          _ => "1/27/1990"
        };
    }
    Write("Enter your salary: ");
    string? salaryText = ReadLine();
    if (string.IsNullOrWhiteSpace(salaryText))
    {
      salaryText = "34500";
    }
    DateTime dob = DateTime.Parse(dobText);
    int minutes = (int)DateTime.Today.Subtract(dob).TotalMinutes;
    decimal salary = decimal.Parse(salaryText);
    WriteLine($"{name} was born on a {dob:dddd}. {name} is {
      minutes:N0} minutes old. {name} earns {salary:C}."); 
    ```

    当你运行一个应用程序时，它会自动将线程设置为使用操作系统的文化。我在英国伦敦运行我的代码，所以线程被设置为英语（大不列颠）。

    代码提示用户输入一个替代的 ISO 代码。这允许你的应用程序在运行时替换默认的文化。

    应用程序随后使用标准的格式代码，使用格式代码`dddd`输出星期几，使用格式代码`N0`输出带千位分隔符的分钟数，以及使用货币符号的薪水。这些会根据线程的文化自动调整。

1.  运行代码，输入 ISO 代码`en-US`（或按*Enter*键），然后输入一些示例数据，包括一个适用于美国英语的日期格式，如下面的输出所示：

    ```cs
    *** Current culture
    The current globalization culture is en-GB: English (United Kingdom)
    The current localization culture is en-GB: English (United Kingdom)
    Days of the week: Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday
    Months of the year: January, February, March, April, May, June, July, August, September, October, November, December
    1st day of this year: 01 January 2023
    Number group separator: ,
    Number decimal separator: .
    Currency symbol: £
    Currency name: British Pound (British Pound)
    IsMetric: True
    Example ISO culture codes:
      da-DK: Danish (Denmark) / dansk (Danmark)
      en-GB: English (United Kingdom) / English (United Kingdom)
      en-US: English (United States) / English (United States)
      fa-IR: Persian (Iran) / فارسی (ایران)
      fr-CA: French (Canada) / français (Canada)
      fr-FR: French (France) / français (France)
      he-IL: Hebrew (Israel) / עברית (ישראל)
      pl-PL: Polish (Poland) / polski (Polska)
      sl-SI: Slovenian (Slovenia) / slovenščina (Slovenija)
    Enter an ISO culture code: en-US
    *** After changing the current culture
    The current globalization culture is en-US: English (United States)
    The current localization culture is en-US: English (United States)
    Days of the week: Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday
    Months of the year: January, February, March, April, May, June, July, August, September, October, November, December
    1st day of this year: Sunday, January 1, 2023
    Number group separator: ,
    Number decimal separator: .
    Currency symbol: $
    Currency name: US Dollar (US Dollar)
    IsMetric: False
    Enter your name: Alice
    Enter your date of birth: 3/30/1967
    Enter your salary: 34500
    Alice was born on a Thursday. Alice is 29,541,600 minutes old. Alice earns $34,500.00 
    ```

1.  再次运行代码，并尝试在丹麦使用丹麦语（`da-DK`），如下面的输出所示：

    ```cs
    Enter an ISO culture code: da-DK
    *** After changing the current culture
    The current globalization culture is da-DK: dansk (Danmark)
    The current localization culture is da-DK: dansk (Danmark)
    Days of the week: søndag, mandag, tirsdag, onsdag, torsdag, fredag, lørdag
    Months of the year: januar, februar, marts, april, maj, juni, juli, august, september, oktober, november, december
    1st day of this year: søndag den 1\. januar 2023
    Number group separator: .
    Number decimal separator: ,
    Currency symbol: kr.
    Currency name: dansk krone (Danish Krone)
    IsMetric: True
    Enter your name: Mikkel
    Enter your date of birth: 16/3/1980
    Enter your salary: 65000
    Mikkel was born on a søndag. Mikkel is 22.723.200 minutes old. Mikkel earns 65.000,00 kr.. 
    ```

    在此示例中，只有日期和薪水被全球化为丹麦语。其余的文本硬编码为英语。稍后，我们将翻译这些英语文本为其他语言。现在，让我们看看文化之间的其他差异。

1.  再次运行代码，并尝试在波兰使用波兰语（`pl-PL`）。请注意，波兰的语法规则使得月份名称的日期是所有格，所以月份`styczeń`变为`stycznia`，如下面的输出所示：

    ```cs
    The current globalization culture is pl-PL: polski (Polska)
    ...
    Months of the year: styczeń, luty, marzec, kwiecień, maj, czerwiec, lipiec, sierpień, wrzesień, październik, listopad, grudzień
    1st day of this year: niedziela, 1 stycznia 2023
    ...
    Enter your name: Bob
    Enter your date of birth: 1972/4/16
    Enter your salary: 50000
    Bob was born on a niedziela. Bob is 26 886 240 minutes old. Bob earns 50 000,00 zł. 
    ```

1.  再次运行代码，并尝试在伊朗使用波斯语（`fa-IR`）。请注意，伊朗的日期必须指定为年/月/日，并且今年（2023 年）在波斯历中是 1401 年，如下面的输出所示：

    ```cs
    The current globalization culture is fa-IR: فارسی (ایران)
    The current localization culture is fa-IR: فارسی (ایران)
    Days of the week: یکشنبه, دوشنبه, سهشنبه, چهارشنبه, پنجشنبه, جمعه, شنبه
    Months of the year: فروردین, اردیبهشت, خرداد, تیر, مرداد, شهریور, مهر, آبان, آذر, دی, بهمن, اسفند
    1st day of this year: 1401 دی 11, شنبه
    Number group separator: ٬
    Number decimal separator: ٫
    Currency symbol: ریال
    Currency name: ریال ایران (Iranian Rial)
    IsMetric: True
    Enter your name: Cyrus
    Enter your date of birth: 1372/4/16
    Enter your salary: 50000
    Cyrus was born on a چهارشنبه. Cyrus is 15٬723٬360 minutes old. Cyrus earns ریال50٬000. 
    ```

尽管我试图与一位波斯语读者确认此示例是否正确，但由于在控制台应用程序中处理从右到左的语言很棘手，以及从控制台窗口复制粘贴到文字处理器的因素，如果此示例全部混乱，我提前向我的波斯语读者道歉！

## 暂时使用不变的文化

有时，你可能需要暂时使用不同的文化，而无需切换当前线程到该文化。例如，当自动生成包含数据值的文档、查询和命令时，你可能需要忽略当前的文化并使用更标准化的文化。为此，你可以使用基于美国英语的不变文化。

例如，你可能需要生成一个带有小数数值的 JSON 文档，并使用两位小数格式化数字，如下面的代码所示：

```cs
decimal price = 54321.99M;
string document = $$"""
  {
    "price": "{{price:N2}}"
  }
  """; 
```

如果你在斯洛文尼亚的计算机上执行此操作，你会得到以下输出：

```cs
{
  "price": " 54.321,99"
} 
```

如果你尝试将此 JSON 文档插入云数据库，它将失败，因为它不会理解使用逗号表示小数点，点表示组的数字格式。

因此，你可以在输出数字作为`string`值时覆盖当前文化，并指定不变的文化，如下面的代码所示：

```cs
decimal price = 54321.99M;
string document = $$"""
  {
    "price": "{{price.ToString("N2", CultureInfo.InvariantCulture)}}"
  }
  """; 
```

如果你在斯洛文尼亚（或任何其他文化）的计算机上执行此操作，你现在将得到以下输出，这将成功被云数据库识别而不会抛出异常：

```cs
{
  "price": " 54,321.99"
} 
```

现在，让我们看看如何将文本从一种语言翻译成另一种语言，以便标签提示在当前文化中正确显示。

## 本地化用户界面

本地化应用程序分为两部分：

+   包含对所有区域都相同的代码和在其他资源文件未找到时使用的资源的程序集。

+   包含不同区域用户界面资源的一个或多个程序集，这些被称为**卫星程序集**。

此模型允许初始应用程序部署时使用默认的不变资源，并且随着时间的推移，可以部署额外的卫星程序集，因为资源被翻译。在编码任务中，你将创建一个具有嵌入式不变文化的控制台应用程序，以及丹麦语、法语、法语-加拿大、波兰语和伊朗语（波斯语）的卫星程序集。要将来添加更多文化，只需遵循相同的步骤。

用户界面资源包括任何消息、日志、对话框、按钮、标签或甚至图像、视频等文件名的文本。资源文件是具有 `.resx` 扩展名的 XML 文件。文件名包括文化代码，例如，`PacktResources.en-GB.resx` 或 `PacktResources.da-DK.resx`。

如果资源文件或单个条目缺失，资源自动文化回退搜索路径从特定文化（语言和区域）到中性文化（仅语言）再到不变文化（理论上独立，但实际上是美式英语）。如果当前线程文化是 `en-AU`（澳大利亚英语），那么它将按以下顺序搜索资源文件：

1.  澳大利亚英语：`PacktResources.en-AU.resx`

1.  中性英语：`PacktResources.en.resx`

1.  不变项：`PacktResources.resx`

## 定义和加载资源

要从这些卫星程序集中加载资源，我们使用一些标准的 .NET 类型，名为 `IStringLocalizer<T>` 和 `IStringLocalizerFactory`。这些实现的加载来自 .NET 通用宿主作为依赖服务：

1.  在 `WorkingWithCultures` 项目中，添加对 Microsoft 扩展的包引用以使用通用托管和本地化，如下所示：

    ```cs
    <ItemGroup>
      <PackageReference Include="Microsoft.Extensions.Hosting" 
                        Version="8.0.0" />
      <PackageReference Include="Microsoft.Extensions.Localization" 
                        Version="8.0.0" />
    </ItemGroup> 
    ```

1.  构建具有 `WorkingWithCultures` 项目的项目以恢复包。

1.  在项目文件夹中，创建一个名为 `Resources` 的新文件夹。

1.  在 `Resources` 文件夹中，添加一个名为 `PacktResources.resx` 的新 XML 文件，并修改其内容以包含默认的不变语言资源（通常相当于美式英语），如下所示：

    ```cs
    <?xml version="1.0" encoding="utf-8"?>
    <root>
      <data name="EnterYourDob" xml:space="preserve">
        <value>Enter your date of birth: </value>
      </data>
      <data name="EnterYourName" xml:space="preserve">
        <value>Enter your name: </value>
      </data>
      <data name="EnterYourSalary" xml:space="preserve">
        <value>Enter your salary: </value>
      </data>
      <data name="PersonDetails" xml:space="preserve">
        <value>{0} was born on a {1:dddd}. {0} is {2:N0} minutes old. {0} earns {3:C}.</value>
      </data>
    </root> 
    ```

1.  在 `WorkingWithCultures` 项目文件夹中，添加一个名为 `PacktResources.cs` 的新类文件，该文件将加载用户界面的文本资源，如下所示：

    ```cs
    using Microsoft.Extensions.Localization; // To use IStringLocalizer and so on.
    public class PacktResources
    {
      private readonly IStringLocalizer<PacktResources> localizer = null!;
      public PacktResources(IStringLocalizer<PacktResources> localizer)
      {
        this.localizer = localizer;
      }
      public string? GetEnterYourNamePrompt()
      {
        string resourceStringName = "EnterYourName";
        // 1\. Get the LocalizedString object.
        LocalizedString localizedString = localizer[resourceStringName];
        // 2\. Check if the resource string was found.
        if (localizedString.ResourceNotFound)
        {
          ConsoleColor previousColor = ForegroundColor;
          ForegroundColor = ConsoleColor.Red;
          WriteLine($"Error: resource string \"{resourceStringName}\" not found."
            + Environment.NewLine
            + $"Search path: {localizedString.SearchedLocation}");
          ForegroundColor = previousColor;
          return $"{localizedString}: ";
        }
        // 3\. Return the found resource string.
        return localizedString;
      }
      public string? GetEnterYourDobPrompt()
      {
        // LocalizedString has an implicit cast to string
        // that falls back to the key if the resource 
        // string is not found.
        return localizer["EnterYourDob"];
      }
      public string? GetEnterYourSalaryPrompt()
      {
        return localizer["EnterYourSalary"];
      }
      public string? GetPersonDetails(
        string name, DateTime dob, int minutes, decimal salary)
      {
        return localizer["PersonDetails", name, dob, minutes, salary];
      }
    } 
    ```

    对于`GetEnterYourNamePrompt`方法，我将实现分解为步骤以获取有用的信息，例如检查资源字符串是否找到，如果没有找到则显示搜索路径。其他方法实现如果资源未找到，则使用简化回退到资源字符串的键名。

1.  在`Program.cs`文件顶部，导入用于处理托管和依赖注入的命名空间，然后配置一个启用本地化和`PacktResources`服务的宿主，如下所示：

    ```cs
    using Microsoft.Extensions.Hosting; // To use IHost, Host.
    // To use AddLocalization, AddTransient<T>.
    using Microsoft.Extensions.DependencyInjection;
    using IHost host = Host.CreateDefaultBuilder(args)
      .ConfigureServices(services =>
      {
        services.AddLocalization(options =>
        {
          options.ResourcesPath = "Resources";
        });
        services.AddTransient<PacktResources>();
      })
      .Build(); 
    ```

    **良好实践**：默认情况下，`ResourcesPath`是一个空字符串，这意味着它将在当前目录中查找`.resx`文件。我们将通过将资源放入子文件夹来使项目更整洁。

1.  在更改当前文化后，添加一个获取`PacktResources`服务的语句，并使用它来输出用户输入姓名、出生日期和薪水的本地化提示。然后，输出他们的详细信息，如下所示：

    ```cs
    OutputCultures("After changing the current culture");
    **PacktResources resources =**
     **host.Services.GetRequiredService<PacktResources>();**
    Write(**resources.GetEnterYourNamePrompt()**);
    string? name = ReadLine();
    if (string.IsNullOrWhiteSpace(name))
    {
      name = "Bob";
    }
    Write(**resources.GetEnterYourDobPrompt()**);
    string? dobText = ReadLine();
    if (string.IsNullOrWhiteSpace(dobText))
    {
      // If they do not enter a DOB then use
      // sensible defaults for their culture.
      dobText = ci.Name switch
        {
          "en-US" or "fr-CA" => "1/27/1990",
          "da-DK" or "fr-FR" or "pl-PL" => "27/1/1990",
          "fa-IR" => "1990/1/27",
          _ => "1/27/1990"
        };
    }
    Write(**resources.GetEnterYourSalaryPrompt()**);
    string? salaryText = ReadLine();
    if (string.IsNullOrWhiteSpace(salaryText))
    {
      salaryText = "34500";
    }
    DateTime dob = DateTime.Parse(dobText);
    int minutes = (int)DateTime.Today.Subtract(dob).TotalMinutes;
    decimal salary = decimal.Parse(salaryText);
    WriteLine(**resources.GetPersonDetails(name, dob, minutes, salary)**); 
    ```

## 测试全球化与本地化

现在，我们可以运行控制台应用程序并查看资源正在被加载：

1.  运行控制台应用程序，并输入`da-DK`作为 ISO 代码。请注意，提示信息使用的是美式英语，因为我们目前只有不变的文化资源。

    为了节省时间并确保你有正确的结构，你可以复制、粘贴并重命名`.resx`文件，而不是创建空的新文件。或者，你也可以从本书的 GitHub 仓库中复制这些文件。

1.  在`Resources`文件夹中，添加一个名为`PacktResources.da.resx`的新 XML 文件，并修改其内容以包含非区域特定的丹麦语资源，如下所示：

    ```cs
    <?xml version="1.0" encoding="utf-8"?>
    <root>
      <data name="EnterYourDob" xml:space="preserve">
        <value>Indtast din fødselsdato: </value>
      </data>
      <data name="EnterYourName" xml:space="preserve">
        <value>Indtast dit navn: </value>
      </data>
      <data name="EnterYourSalary" xml:space="preserve">
        <value>Indtast din løn: </value>
      </data>
      <data name="PersonDetails" xml:space="preserve">
        <value>{0} blev født på en {1:dddd}. {0} er {2:N0} minutter gammel. {0} tjener {3:C}.</value>
      </data>
    </root> 
    ```

1.  在`Resources`文件夹中，添加一个名为`PacktResources.fr.resx`的新 XML 文件，并修改其内容以包含非区域特定的法语资源，如下所示：

    ```cs
    <?xml version="1.0" encoding="utf-8"?>
    <root>
      <data name="EnterYourDob" xml:space="preserve">
        <value>Entrez votre date de naissance: </value>
      </data>
      <data name="EnterYourName" xml:space="preserve">
        <value>Entrez votre nom: </value>
      </data>
      <data name="EnterYourSalary" xml:space="preserve">
        <value>Entrez votre salaire: </value>
      </data>
      <data name="PersonDetails" xml:space="preserve">
        <value>{0} est né un {1:dddd}. {0} a {2:N0} minutes. {0} gagne {3:C}.</value>
      </data>
    </root> 
    ```

1.  在`Resources`文件夹中，添加一个名为`PacktResources.fr-CA.resx`的新 XML 文件，并修改其内容以包含加拿大地区的法语资源，如下所示：

    ```cs
    <?xml version="1.0" encoding="utf-8"?>
    <root>
      <data name="EnterYourDob" xml:space="preserve">
        <value>Entrez votre date de naissance / Enter your date of birth: </value>
      </data>
      <data name="EnterYourName" xml:space="preserve">
        <value>Entrez votre nom / Enter your name: </value>
      </data>
      <data name="EnterYourSalary" xml:space="preserve">
        <value>Entrez votre salaire / Enter your salary: </value>
      </data>
      <data name="PersonDetails" xml:space="preserve">
        <value>{0} est né un {1:dddd}. {0} a {2:N0} minutes. {0} gagne {3:C}.</value>
      </data>
    </root> 
    ```

1.  在`Resources`文件夹中，添加一个名为`PacktResources.pl-PL.resx`的新 XML 文件，并修改其内容以包含波兰地区的波兰语资源，如下所示：

    ```cs
    <?xml version="1.0" encoding="utf-8"?>
    <root>
      <data name="EnterYourDob" xml:space="preserve">
        <value>Wpisz swoją datę urodzenia: </value>
      </data>
      <data name="EnterYourName" xml:space="preserve">
        <value>Wpisz swoje imię i nazwisko: </value>
      </data>
      <data name="EnterYourSalary" xml:space="preserve">
        <value>Wpisz swoje wynagrodzenie: </value>
      </data>
      <data name="PersonDetails" xml:space="preserve">
        <value>{0} urodził się na {1:dddd}. {0} ma {2:N0} minut. {0} zarabia {3:C}.</value>
      </data>
    </root> 
    ```

1.  在`Resources`文件夹中，添加一个名为`PacktResources.fa-IR.resx`的新 XML 文件，并修改其内容以包含伊朗地区的波斯语资源，如下所示：

    ```cs
    <?xml version="1.0" encoding="utf-8"?>
    <root>
      <data name="EnterYourDob" xml:space="preserve">
        <value>تاریخ تولد خود را وارد کنید / Enter your date of birth: </value>
      </data>
      <data name="EnterYourName" xml:space="preserve">
        <value>اسمت را وارد کن / Enter your name: </value>
      </data>
      <data name="EnterYourSalary" xml:space="preserve">
        <value>حقوق خود را وارد کنید / Enter your salary: </value>
      </data>
      <data name="PersonDetails" xml:space="preserve">
        <value>{0} در {1:dddd}به دنیا آمد. {0} {2:N0} دقیقه است. {0} {3:C}.</value>
      </data>
    </root> 
    ```

1.  运行代码，并输入`da-DK`作为 ISO 代码。请注意，提示信息使用的是丹麦语，如下所示：

    ```cs
    The current localization culture is da-DK: dansk (Danmark)
    ...
    Indtast dit navn: Bob
    Indtast din fødselsdato: 3/4/1987
    Indtast din løn: 45449
    Bob blev født på en fredag. Bob er 19.016.640 minutter gammel. Bob tjener 45.449,00 kr. 
    ```

1.  运行代码，并输入`fr-FR`作为 ISO 代码。请注意，提示信息只使用法语，如下所示：

    ```cs
    The current localization culture is fr-FR: français (France)
    ...
    Entrez votre nom: Monique
    Entrez votre date de naissance: 2/12/1990
    Entrez votre salaire: 45000
    Monique est né un dimanche. Monique a 17 088 480 minutes. Monique gagne 45 000,00 €. 
    ```

1.  运行代码，并输入`fr-CA`作为 ISO 代码。请注意，提示信息使用的是法语和英语，因为加拿大可能需要支持这两种官方语言，如下所示：

    ```cs
    The current localization culture is fr-CA: français (Canada)
    ...
    Entrez votre nom / Enter your name: Sophie
    Entrez votre date de naissance / Enter your date of birth: 4/5/2001
    Entrez votre salaire / Enter your salary: 65000
    Sophie est né un jeudi. Sophie a 11 649 600 minutes. Sophie gagne 65 000,00 $ CA. 
    ```

1.  运行代码，并输入`fa-IR`作为 ISO 代码。注意提示为波斯/波斯语和英语，并且存在从右到左的语言的额外复杂性，如下所示：

    ```cs
    The current localization culture is fa-IR: فارسی (ایران)
    ...
    اسمت را وارد کن / Enter your name: Hoshyar
    تاریخ تولد خود را وارد کنید / Enter your date of birth: 1370/3/6
    حقوق خود را وارد کنید / Enter your salary: 90000
    Hoshyar در چهارشنبهبه دنیا آمد. Hoshyar 11٬190٬240 دقیقه است. Hoshyar ریال90٬000. 
    ```

    如果你需要处理波斯日期，那么有一些 NuGet 包和开源 GitHub 存储库可供尝试，尽管我不能保证它们的正确性，如[`github.com/VahidN/DNTPersianUtils.Core`](https://github.com/VahidN/DNTPersianUtils.Core)和[`github.com/imanabidi/PersianDate.NET`](https://github.com/imanabidi/PersianDate.NET)。

1.  在`Resources`文件夹中，在`PacktResources.da.resx`中，修改内容以故意更改提示输入姓名的键，通过附加`Wrong`，如以下标记所示：

    ```cs
    <?xml version="1.0" encoding="utf-8"?>
    <root>
      <data name="EnterYourDob" xml:space="preserve">
        <value>Indtast din fødselsdato: </value>
      </data>
      <data name="EnterYourName**Wrong**" xml:space="preserve">
        <value>Indtast dit navn: </value>
      </data>
      <data name="EnterYourSalary" xml:space="preserve">
        <value>Indtast din løn: </value>
      </data>
      <data name="PersonDetails" xml:space="preserve">
        <value>{0} blev født på en {1:dddd}. {0} er {2:N0} minutter gammel. {0} tjener {3:C}.</value>
      </data>
    </root> 
    ```

1.  运行代码，并输入`da-DK`作为 ISO 代码。注意提示为丹麦语，除了`Enter your name`提示，它显示错误并使用键名作为最后的-退路回退，如下所示：

    ```cs
    The current localization culture is da-DK: dansk (Danmark)
    ...
    **Enter your name**: Bob
    Indtast din fødselsdato: 3/4/1987
    Indtast din løn: 45449
    Bob blev født på en fredag. Bob er 18.413.280 minutter gammel. Bob tjener 45.449,00 kr. 
    ```

1.  在`Resources`文件夹中，在`PacktResources.resx`中，修改内容以故意更改提示输入姓名的键，通过附加`Wrong`。

1.  运行代码，并输入`da-DK`作为 ISO 代码。注意提示为丹麦语，除了`Enter your name`提示，它显示错误并使用键名作为最后的-退路回退，如下所示：

    ```cs
    The current localization culture is da-DK: dansk (Danmark)
    ...
    **Error: resource string "EnterYourName" not found.**
    **Search path: WorkingWithCultures.Resources.PacktResources**
    **EnterYourName:** Bob
    Indtast din fødselsdato: 3/4/1987
    Indtast din løn: 45449
    Bob blev født på en fredag. Bob er 18.413.280 minutter gammel. Bob tjener 45.449,00 kr. 
    ```

1.  从两个资源文件中移除`Wrong`后缀。

1.  在**解决方案资源管理器**中，切换**显示所有文件**，并展开`bin/Debug/net8.0/da`文件夹，如图 7.1 所示：

![](img/B19587_07_01.png)

图 7.1：文化资源卫星组件文件夹

1.  注意名为`WorkingWithCultures.resources.dll`的卫星组件，用于中性的丹麦资源。

其他文化资源组件的命名相同，但存储在匹配适当文化代码的文件夹中。你可以使用像**ResX 资源管理器**（可在以下链接找到：[`dotnetfoundation.org/projects/resx-resource-manager`](https://dotnetfoundation.org/projects/resx-resource-manager)）这样的工具来创建更多的`.resx`文件，将它们编译成卫星组件，然后部署给用户，而无需重新编译原始控制台应用程序。

**良好实践**：考虑你的应用程序是否需要国际化，并在开始编码之前为此做好准备！思考所有需要全球化的数据（日期格式、数字格式和排序文本行为）。写下用户界面中所有需要本地化的文本片段。

微软有一个在线工具（可在以下链接找到：[`www.microsoft.com/en-us/Language/`](https://www.microsoft.com/en-us/Language/))，可以帮助你翻译用户界面中的文本，如图 7.2 所示：

![](img/B19587_07_02.png)

图 7.2：微软用户界面在线文本翻译工具

我们已经看到了 .NET BCL 提供的许多日期和时间功能。它是否提供了我们处理国际化所需的一切？不幸的是，并没有。这就是为什么你可能会想使用 Noda Time。

# 使用 Noda Time

**Noda Time** 是为那些觉得内置的日期/时间处理库不够好的开发者设计的。Noda Time 类似于 Joda Time，是 Java 的替代日期/时间处理库。

Noda Time 3.0 或更高版本支持 .NET Standard 2.0 和 .NET 6 或更高版本。这意味着你可以使用它与旧平台，如 .NET Framework 和 Xamarin，以及现代 .NET。

要了解内置 .NET 日期/时间类型的核心缺陷之一，可以想象，如果 .NET 团队没有为数字定义单独的类型，如 `int` (`System.Int32`)、`double` (`System.Double`) 和 `decimal` (`System.Decimal`)，而是只定义了一个名为 `System.Number` 的类型，并有一个名为 `Kind` 的属性来指示它是哪种类型的数字，它在内存中的存储方式，如何处理它等等。

那就是团队对 `System.DateTime` 类型所做的事情。该类型有一个 `Kind` 属性，指示它是否是本地时间、UTC 时间或未指定。它的行为会根据你如何处理它而有所不同。这使得在 .NET 中实现的日期/时间值在操作和理解上非常复杂。

**更多信息**: Noda Time 的创建者 Jon Skeet 在一篇 2011 年的博客文章中详细讨论了 .NET 和 `DateTime` 对日期/时间支持的局限性，具体内容可以在以下链接找到：[`blog.nodatime.org/2011/08/what-wrong-with-datetime-anyway.html`](https://blog.nodatime.org/2011/08/what-wrong-with-datetime-anyway.html)

## Noda Time 中的重要概念和默认值

内置的 `DateTime` 类型存储全局和本地值，或者未指定的值。除非非常小心地处理，否则这会打开细微的错误和误解的大门。

与 Noda Time 的第一个重大区别在于，它强制你在类型级别做出选择。因此，Noda Time 有更多类型，并且一开始可能会显得更复杂。Noda Time 中的类型是全球性的或本地的。世界上任何地方的每个人在同一个瞬间都会共享相同的全局值，而每个人会有不同的本地值，这取决于他们的时区等因素。

.NET 内置的日期/时间类型仅精确到 tick，大约是 100 纳秒。Noda Time 精确到 1 纳秒，比它精确 100 倍。

Noda Time 的“零”基线是 1970 年 1 月 1 日午夜在 UTC 时区开始的时间。Noda Time 的 `Instant` 是从那时起（如果是一个负值）或到那时（如果是一个正值）的纳秒数，它代表全球时间线上的一个时间点。

Noda Time 的默认日历系统是 ISO-8601 日历，因为它是一个标准，你可以在以下链接中了解更多信息：[`en.wikipedia.org/wiki/ISO_8601`](https://en.wikipedia.org/wiki/ISO_8601)。支持自动转换为其他日历系统，如儒略、科普特和佛教日历。

Noda Time 有一些类似于某些 .NET 日期/时间类型的常见类型，总结如下 *表 7.6*：

| **Noda Time 类型** | **描述** |
| --- | --- |
| `Instant` 结构体 | 表示全球时间线上的一个瞬间，具有纳秒级分辨率。 |
| `Interval` 结构体 | 两个 `Instant` 值，一个包含的起始点和一个排除的终点。 |
| `LocalDateTime` 结构体 | 在特定日历系统中的日期/时间值，但它不表示全球时间线上的一个固定点。例如，2023 年除夕的午夜对不同时区的人来说是不同的。如果你不知道用户的时区，你可能会不得不使用此类型。 |
| `LocalDate` 和 `LocalTime` 结构体 | 与 `LocalDateTime` 结构体类似，但只有日期或时间部分。当提示用户输入日期和时间值时，你通常分别使用它们，然后将它们组合成一个单一的 `LocalDateTime` 结构体。 |
| `DateTimeZone` 类 | 表示时区。可以使用 `BclDateTimeZone` 从 .NET 的 `TimeZoneInfo` 转换，使用 `DateTimeZoneProviders.Tzdb` 获取时区，基于以下链接中列出的标准名称：[`en.wikipedia.org/wiki/List_of_tz_database_time_zones`](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) |
| `ZonedDateTime` 结构体 | 在特定日历系统和特定时区中的日期/时间值，因此它确实代表全球时间线上的一个固定点。 |
| `Offset` 结构体 | 表示偏移量。如果本地时间早于 UTC，则为正值；如果本地时间晚于 UTC，则为负值。 |
| `OffsetDateTime` 结构体 | 你可能知道从 UTC 的偏移量，但这并不总是干净地映射到一个单一时区。在这种情况下应使用此类型而不是 `ZonedDateTime`。 |
| `Duration` 结构体 | 表示固定数量的纳秒。具有转换为常用时间单位（如 `Days`、`Hours`、`Minutes` 和 `Seconds`）的属性，由于它们返回一个 `int`，所以会向下或向上舍入到零，以及非舍入属性如 `TotalDays`、`TotalMinutes` 等，因为它们返回一个 `double`。用于对 `Instant` 和 `ZonedDateTime` 值进行计算。使用此类型代替 .NET 的 `TimeSpan` 类型。 |
| `Period` 类 | 可变持续时间，因为 2024 年 1 月和 2 月表示的“两个月”与 2024 年 6 月和 7 月的“两个月”或甚至非闰年（2024 年是闰年，因此 2024 年 2 月有 29 天）表示的“两个月”长度不同。 |

表 7.6：常见的 Noda Time 类型

**良好实践**：使用 `Instant` 记录某事发生的时间点。这对于时间戳很有用。然后可以将其表示为用户所在时区的本地时间。用于记录用户输入的日期/时间值的常见类型如下：`ZonedDateTime`、`OffsetDateTime`、`LocalDateTime`、`LocalDate` 和 `LocalTime`。

## 在 Noda Time 日期/时间类型之间进行转换

为了总结在 Noda Time 类型之间转换的常见方法，请查看以下非详尽图示：

![图片](img/B19587_07_03.png)

图 7.3：在 Noda Time 日期/时间类型之间转换的常见方法

## 探索控制台应用程序中的节点时间

让我们编写一些代码：

1.  使用你喜欢的代码编辑器将一个新的 **控制台应用程序**/`console` 项目命名为 `WorkingWithNodaTime` 添加到 `Chapter07` 解决方案中。

    +   在项目文件中，将警告视为错误，然后静态和全局导入 `System.Console` 类，并添加对 `NodaTime` 的包引用，如下面的标记所示：

        ```cs
        <ItemGroup>
          <PackageReference Include="NodaTime" Version="3.1.9" />
        </ItemGroup> 
        ```

1.  添加一个名为 `Program.Helpers.cs` 的新类文件，并替换其内容，如下面的代码所示：

    ```cs
    partial class Program
    {
      private static void SectionTitle(string title)
      {
        ConsoleColor previousColor = ForegroundColor;
        ForegroundColor = ConsoleColor.DarkYellow;
        WriteLine($"*** {title}");
        ForegroundColor = previousColor;
      }
    } 
    ```

1.  在 `Program.cs` 文件中，删除现有的语句，添加语句以获取当前时间点，并将其转换为各种 Noda Time 类型，包括 UTC、几个时区和本地时间，如下面的代码所示：

    ```cs
    using NodaTime; // To use SystemClock, Instant and so on.
    SectionTitle("Converting Noda Time types");
    // Get the current instant in time.
    Instant now = SystemClock.Instance.GetCurrentInstant();
    WriteLine($"Now (Instant): {now}");
    WriteLine();
    ZonedDateTime nowInUtc = now.InUtc();
    WriteLine($"Now (DateTimeZone): {nowInUtc.Zone}");
    WriteLine($"Now (ZonedDateTime): {nowInUtc}");
    WriteLine($"Now (DST): {nowInUtc.IsDaylightSavingTime()}");
    WriteLine();
    // Use the Tzdb provider to get the time zone for US Pacific.
    // To use .NET compatible time zones, use the Bcl provider.
    DateTimeZone zonePT = DateTimeZoneProviders.Tzdb["US/Pacific"];
    ZonedDateTime nowInPT = now.InZone(zonePT);
    WriteLine($"Now (DateTimeZone): {nowInPT.Zone}");
    WriteLine($"Now (ZonedDateTime): {nowInPT}");
    WriteLine($"Now (DST): {nowInPT.IsDaylightSavingTime()}");
    WriteLine();
    DateTimeZone zoneUK = DateTimeZoneProviders.Tzdb["Europe/London"];
    ZonedDateTime nowInUK = now.InZone(zoneUK);
    WriteLine($"Now (DateTimeZone): {nowInUK.Zone}");
    WriteLine($"Now (ZonedDateTime): {nowInUK}");
    WriteLine($"Now (DST): {nowInUK.IsDaylightSavingTime()}");
    WriteLine();
    LocalDateTime nowInLocal = nowInUtc.LocalDateTime;
    WriteLine($"Now (LocalDateTime): {nowInLocal}");
    WriteLine($"Now (LocalDate): {nowInLocal.Date}");
    WriteLine($"Now (LocalTime): {nowInLocal.TimeOfDay}");
    WriteLine(); 
    ```

1.  运行控制台应用程序，并注意结果，包括“本地”时间不考虑任何夏令时偏移；例如，在我的情况下，居住在英国，我必须使用伦敦时区来获取夏令时（上午 10:21），而不是“本地”时间（上午 9:21），如下面的输出所示：

    ```cs
    *** Converting Noda Time types
    Now (Instant): 2023-06-01T09:21:05Z
    Now (DateTimeZone): UTC
    Now (ZonedDateTime): 2023-06-01T09:21:05 UTC (+00)
    Now (DST): False
    Now (DateTimeZone): US/Pacific
    Now (ZonedDateTime): 2023-06-01T02:21:05 US/Pacific (-07)
    Now (DST): True
    Now (DateTimeZone): Europe/London
    Now (ZonedDateTime): 2023-06-01T10:21:05 Europe/London (+01)
    Now (DST): True
    Now (LocalDateTime): 01/06/2023 09:21:05
    Now (LocalDate): 01 June 2023
    Now (LocalTime): 09:21:05 
    ```

1.  在 `Program.cs` 文件中，添加语句以探索如何使用时间段，如下面的代码所示：

    ```cs
    SectionTitle("Working with periods");
    // The modern .NET era began with the release of .NET Core 1.0
    // on June 27, 2016 at 10am Pacific Time, or 5pm UTC.
    LocalDateTime start = new(year: 2016, month: 6, day: 27, 
      hour: 17, minute: 0, second: 0);
    LocalDateTime end = LocalDateTime.FromDateTime(DateTime.UtcNow);
    WriteLine("Modern .NET era");
    WriteLine($"Start: {start}");
    WriteLine($"End: {end}");
    WriteLine();
    Period period = Period.Between(start, end);
    WriteLine($"Period: {period}");
    WriteLine($"Years: {period.Years}");
    WriteLine($"Months: {period.Months}");
    WriteLine($"Weeks: {period.Weeks}");
    WriteLine($"Days: {period.Days}");
    WriteLine($"Hours: {period.Hours}");
    WriteLine();
    Period p1 = Period.FromWeeks(2);
    Period p2 = Period.FromDays(14);
    WriteLine($"p1 (period of two weeks): {p1}");
    WriteLine($"p2 (period of 14 days): {p2}");
    WriteLine($"p1 == p2: {p1 == p2}");
    WriteLine($"p1.Normalize() == p2: {p1.Normalize() == p2}"); 
    ```

1.  运行控制台应用程序并注意结果，包括在 2023 年 6 月 1 日运行控制台应用程序时，现代 .NET 时代已经持续了 6 年 11 个月和 4 天，`Period` 类型的序列化格式，以及如何比较两个时间段以及比较之前应该进行归一化，如下面的输出所示：

    ```cs
    *** Working with periods
    Modern .NET era
    Start: 27/06/2016 17:00:00
    End: 01/06/2023 09:21:05
    Period: P6Y11M4DT16H21M5S889s9240t
    Years: 6
    Months: 11
    Weeks: 0
    Days: 4
    Hours: 16
    p1 (period of two weeks): P2W
    p2 (period of 14 days): P14D
    p1 == p2: False
    p1.Normalize() == p2: True 
    ```

**更多信息**：使用 `Normalize` 方法归一化 `Period` 意味着将任何周数乘以 7 并加到天数上，然后将 `Weeks` 设置为零，以及其他计算。更多信息请参阅以下链接：[`nodatime.org/3.1.x/api/NodaTime.Period.html#NodaTime_Period_Normalize`](https://nodatime.org/3.1.x/api/NodaTime.Period.html#NodaTime_Period_Normalize)

## 使用 Noda Time 进行单元测试和 JSON 序列化

Noda Time 有两个可选包用于编写单元测试（`NodaTime.Testing`）和与 JSON.NET 一起工作（`NodaTime.Serialization.JsonNet`）。

Noda Time 的文档可以在以下链接找到：[`nodatime.org/`](https://nodatime.org/)

# 练习和探索

通过回答一些问题、进行一些动手实践和更深入地研究本章主题来测试你的知识和理解。

## 练习 7.1 – 测试你的知识

使用网络回答以下问题：

1.  本地化、全球化和国际化之间的区别是什么？

1.  .NET 中可用的最小时间度量是什么？

1.  在 .NET 中，“滴答”的时间是多长？

1.  在什么场景下你可能使用 `DateOnly` 值而不是 `DateTime` 值？

1.  对于一个时区，它的 `BaseUtcOffset` 属性告诉你什么？

1.  你如何获取代码执行所在本地时区的信息？

1.  对于一个 `DateTime` 值，它的 `Kind` 属性告诉你什么？

1.  你如何控制执行代码的当前文化环境？

1.  威尔士的 ISO 文化代码是什么？

1.  本地化资源文件回退是如何工作的？

## 练习 7.2 – 探索主题

使用以下页面上的链接了解本章涵盖主题的更多详细信息：

[`github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-7---handling-dates-times-and-internationalization`](https://github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-7---handling-dates-times-and-internationalization)

## 练习 7.3 – 向专家 Jon Skeet 学习

Jon Skeet 是国际化的世界知名专家。观看他在以下链接中展示的 *Working with Time is Easy*：[`www.youtube.com/watch?v=saeKBuPewcU`](https://www.youtube.com/watch?v=saeKBuPewcU)

# 摘要

在本章中，你：

+   探索了日期和时间，包括 .NET 8 的 `TimeProvider` 以改进单元测试。

+   学会了如何处理时区。

+   学习了如何使用全球化和本地化来国际化你的代码。

+   探索了 Noda Time 的一些功能。

在下一章中，你将学习如何使用 ASP.NET Core Minimal API 构建网络服务，以及如何确保和保护它们。
