# *第四章*：变量、列表和函数

本章基于在*第二章**、*结、转向和循环模式*中引入的多个概念。在第一个主题中，我们将检查关键字 `VAR` 在墨水中与单个值的结合方式，以及它如何与 `LIST` 关键字结合使用。

在墨水中，我们可以将变量组合成一个称为列表的概念。我们还将检查如何创建和更改列表中的值。然后，我们将回顾它们在项目中最佳的使用情况以及可能更适合多个单独变量的情况。这次讨论将引导我们进入下一个主题，我们将探讨如何使用函数。

列表中的值可以通过称为**函数**的其他概念进行更改。在第三个主题中，我们将调用一些内置函数来处理不同的列表值。这将允许我们在列表上执行操作，例如确定条目数量或随机选择一个。使用函数将帮助我们为下一步做准备，即创建函数。

在最后一个主题中，我们将探讨如何在墨水中创建我们的函数。正如我们将看到的，函数允许我们定义小任务或一系列动作，我们可以通过调用创建的函数多次使用。我们将了解到，函数是墨水中**结**的特殊形式。这意味着我们可以向函数以及结发送数据。然而，只有函数可以返回数据。

在本章中，我们将涵盖以下主要主题：

+   使用 `VAR` 存储值

+   使用 `LIST`

+   调用函数

+   创建新函数和调用结

# 技术要求

本章中使用的示例，在 `*.ink` 文件中，可以在 GitHub 上找到：[`github.com/PacktPublishing/Dynamic-Story-Scripting-with-the-ink-Scripting-Language/tree/main/Chapter4`](https://github.com/PacktPublishing/Dynamic-Story-Scripting-with-the-ink-Scripting-Language/tree/main/Chapter4)。

# 使用 VAR 存储值

在*第二章*“结、转向和循环模式”中，变量作为在墨水循环结构编织中使用标记选项的一部分被引入。通过创建一个标签，一个选项可以记录它是否之前已经被展示过。这使得我们能够轻松地跟踪结内的循环数量。在墨水中，标记选项是存储和更改任何类型值的一般概念的一种形式。这种更一般的形式使用一个特殊的关键字：**VAR**。

`VAR` 关键字创建了一个能够存储不同类型数据的变量。使用 `VAR` 关键字创建的变量可以存储数字（包括小数）、`true` 或 `false` 值，甚至转向。使用 `VAR` 关键字创建的变量也是**全局的**：它们可以被任何属于整体项目的代码访问。

在 ink 中使用 `VAR` 关键字的变量名遵循结和针的相同规则：

+   它们可以包含数字。

+   它们可以包含大写和小写字母。

+   允许使用的唯一特殊符号是下划线。

类似于结和针的命名规范，变量名中的单词之间通常使用下划线分隔：

示例 1 (Example1.ink):

```cs
VAR reader_name = "Dan"
```

变量的值通过称为 `=` 的操作来赋予，将变量赋值为同一行上的符号之后的值。在变量名、等号（`=`）和分配给变量的值之间包含一个空格是常见的。

使用 `VAR` 关键字和为变量分配初始值时，必须始终遵循的一个明确规则是：无论值是什么，它都必须是静态的。在 ink 中，这意味着任何变量的第一次赋值不能是其他现有值的组合或执行数学运算的代码的结果。

然而，一旦创建，变量的值可以更改，但初始赋值必须始终存在，并且不能是任何计算的任何结果。

在这个主题中，我们将学习如何显示和更改变量值。因为替代方案（在第 *第三章*，*序列、循环和文本洗牌*）产生值，我们还将探讨如何保存它们产生的值，并将该值作为附加代码的一部分使用。

## 显示变量

ink 中使用开括号 `{` 和闭括号 `}` 花括号表示代码的使用。当与变量名结合使用时，ink 将将变量的值替换为周围文本的一部分。这允许作者将变量作为文本的一部分使用，并在 ink 的最终输出中将变量的名称与其值进行替换：

示例 2 (Example2.ink):

```cs
VAR reader_name = "Dan"
The name of the reader is {reader_name}.
```

当 *示例 2* 中的代码运行时，ink 创建一个名为 `reader_name` 的变量。接下来，它将变量设置为 `"Dan"` 的值。当它遇到文本时，ink 理解花括号是代码，并将变量的名称替换为其值：

![图 4.1 – 示例 2 的 ink 输出截图](img/Figure_4.1_B17597.jpg)

图 4.1 – 示例 2 的 ink 输出截图

使用花括号表示 ink 中使用任何代码。这意味着数学运算也可以在变量中的花括号内执行。ink 将将结果值作为最终输出的一部分替换，如下例所示：

示例 3 (Example3.ink):

```cs
VAR first_variable = 2
VAR second_variable = 2
Adding the values of the two variables together produces: 
  {first_variable + second_variable}.
```

在 *示例 3* 中有两个变量。每个变量都持有不同的数值。当 ink 遇到花括号的使用时，它将每个变量的值替换为其名称。接下来，因为花括号还包含加号，ink 将两个数字相加：

![图 4.2 – 示例 3 的 ink 输出截图](img/Figure_4.2_B17597.jpg)

图 4.2 – 示例 3 的 ink 输出截图

根据涉及的数据类型，数学运算可能不会总是根据在其他编程和脚本语言中的经验产生预期的输出。例如，加法（使用加号 `+`）、减法（使用连字符 `-`）、乘法（使用星号 `*`）和除法（使用正斜杠 `/`）都将作用于数值：

示例 4 (Example4.ink):

```cs
VAR number_two = 2
2 * 2 = {number_two * 2}
2 + 3 = {number_two + 3}
6 / 2 = {6 / number_two}
9 - 2 = {9 - number_two}
```

使用数值和字符串值进行数学运算会产生错误。使用数学符号与字符串值的唯一有效方式是使用加号 (`+`)。这执行的是 **连接** 操作：当一个数值或字符串值被 *添加* 到现有的字符串值时，它会产生一个新的字符串值，如下面的示例所示：

示例 5 (Example5.ink):

```cs
VAR example_string = "Hi"
Perform concatenation: {example_string + 3}
```

当运行 *示例 5* 中的代码时，ink 会创建一个具有字符串值的变量。然而，当它遇到文本和大括号时，它不会执行数学运算。相反，它 *连接* `example_string` 变量的值和数字 `3`。这在其输出中产生了这两个值的组合：

![图 4.3 – ink 为示例 5 生成的输出截图](img/Figure_4.3_B17597.jpg)

图 4.3 – ink 为示例 5 生成的输出截图

让我们现在转到下一部分，了解如何更新变量。

## 更新变量

变量的值可以改变。一旦在 ink 中使用 `VAR` 关键字创建了一个变量，就可以在任何代码点访问它并更改其值。然而，虽然 ink 理解变量的初始赋值必须与文本分开，但在更改变量值时，我们必须使用一个特殊符号：波浪号 (`~`)。例如，我们可以使用 `VAR` 关键字创建一个变量，然后在同一代码的稍后部分更改其值：

示例 6 (Example6.ink):

```cs
VAR reader_name = "Dan"
The reader's name is {reader_name}.
~ reader_name = "Jesse"
The reader's name is now {reader_name}.
```

如果一行代码以波浪号 (`~`) 开头，这会让 ink 知道该行将发生某种类型的代码。例如，当为变量分配初始值时，ink 理解波浪号 (`~`) 后将跟随与代码相关的某些内容。

我们可以使用 `VAR` 关键字创建变量，并使用以波浪号 (`~`) 开头的代码行来更新它们。然而，正如我们在 *第三章* *序列、循环和文本洗牌* 中所看到的，**替代方案**允许我们从一组值中生成一个值。正如我们将在下一节中看到的，我们可以将生成的值保存在变量中，并在同一代码的后续部分使用它。

## 存储替代方案当前值

**替代方案**在 *第三章* *序列、循环和文本洗牌* 中被引入。它们用于根据循环结构中它们被访问的次数生成 *替代* 文本内容。因为替代方案生成文本，它们的输出也可以保存在变量中：

示例 7 (Example7.ink):

```cs
VAR day_of_week = ""
~ day_of_week = "{~Monday|Tuesday|Wednesday|Thursday
    |Friday|Saturday|Sunday}"
-> calendar
== calendar
Today is {day_of_week}!
-> DONE
```

而不是需要重新运行替代方案来生成新的文本，**洗牌**可以将其输出保存以供将来使用。这允许将值纳入其他代码，而无需重新创建替代方案。

替代方案在运行时生成内容。这意味着它们不能用作 ink 中变量的初始赋值的一部分。原因在于 ink 在运行任何替代方案之前处理变量的创建。任何替代方案的生成值在变量最初创建时并不存在。因此，一个常见的模式是创建一个具有初始值的变量，然后在代码的后面用替代方案的生成输出覆盖这个值。

在*示例 7*中，引号包围了使用洗牌替代方案来创建字符串值的使用。当运行时，洗牌将生成一个值，然后根据其周围的引号成为字符串。作为一个字符串值，它将能够与带有波浪号(`~`)的赋值行一起使用。

将替代方案的输出保存到墨迹中通常需要至少两行代码。第一行用于使用`VAR`关键字创建一个变量并赋予其初始值，而第二行则在代码运行时将变量的值重新赋值为替代方案生成的结果。正如在*使用 VAR 存储值*主题的介绍中所解释的，使用`VAR`关键字的显式规则是变量创建时初始值必须存在。替代方案产生的输出被认为是动态的，使用`VAR`关键字创建的变量的初始值必须是静态的。

在这个主题中，我们处理了单一值。我们首先使用`VAR`关键字创建新变量，然后学习了如何使用以波浪号(`~`)开头的行来更新它们的值。我们还探讨了如何保存替代方案的输出，但变量必须被设置为初始值，然后更新为替代方案产生的动态输出。在下一个主题中，我们将基于变量的使用，并使用一个新的关键字`LIST`创建它们的集合。

# **处理 LIST**

每次使用`VAR`关键字都创建一个单一值。在许多项目中，一些单一值就足以在运行时跟踪所需的一切。然而，在某些情况下，可能需要一组值。对于这些情况，ink 有一个特殊的名为**LIST**的关键字，它可以创建一个*列表*的可能值。

列表中的值可以被视为其变量的可能*状态*。例如，对于名为`days_of_week`的`LIST`，可能值可能是每周的 7 天。这些可以用`LIST`本身定义，然后按需分配，而不是需要为每周的每一天使用字符串值。

在 ink 中，列表在项目上下文中定义了一个新的值集合。一旦创建，列表的值可以作为其他使用`VAR`关键字创建的变量的可能值。

然而，尽管在创建变量的新可能值方面功能强大，但创建的值有一些限制，并且通常需要额外的功能来执行 ink 中其他数据类型可用的某些常见操作。（`LIST` 函数将在本章后面的 *使用 LIST 函数* 部分介绍。）

在这个主题中，我们将从创建一个值列表开始。我们将探讨如何使用 `LIST` 关键字创建这个集合。然后，我们将通过遵循我们学习到的处理单个值的相同模式来更改集合中的值。

## 创建 LIST

可以使用 `LIST` 关键字创建一个新的列表。在以 `LIST` 关键字开头的行上，列表的名称后面跟着等号（`=`），其值由逗号分隔。列表的名称，就像 ink 中的其他变量一样，只能包含数字、字母和下划线字符。它不能包含其他特殊符号或空格。

与使用 `VAR` 关键字不同，当值分配给列表时，空格会被忽略，包括一个值和下一个值之间额外的空行。使用 `VAR` 关键字创建的变量必须在单行上定义。列表的值可以分布在多行上：

示例 8（Example8.ink）：

```cs
LIST days_of_week = 
Monday,
Tuesday,
Wednesday,
Thursday,
Friday,
Saturday,
Sunday
VAR day = Monday
Today is {day}.
```

在 *示例 8* 中，创建了一个包含七个可能值的列表。接下来，将其中的一个值，`Monday`，分配给了使用 `VAR` 关键字创建的变量。最后，代码的最后一行显示了 `day` 的值：

![图 4.4 – 示例 8 的 ink 输出截图](img/Figure_4.4_B17597.jpg)

图 4.4 – 示例 8 的 ink 输出截图

与使用 `VAR` 关键字创建的变量一样，一旦创建，列表的值也可以更新。这遵循我们在 *更新变量* 部分介绍的模式。要更新列表或其值，正如我们将在下一节中学习的，需要以波浪线（`~`）开头的行。

## 更新 LIST 值

虽然，如 *示例 8* 所示，值似乎可以显示，但创建的输出是值的名称，`Monday`，而不是字符串，`"Monday"`。关于项目的输出，这是使用 `VAR` 关键字和作为 `LIST` 关键字集合一部分的变量之间的小但重要的区别：只有 `LIST` 值可以添加到列表中。要向现有列表添加新值，它必须是由自身或其他列表创建的：

示例 9（Example9.ink）：

```cs
LIST all_pets = Cats, Dogs, Fish
LIST current_pets = Cats, Dogs
~ current_pets = current_pets + Fish
```

在*示例 9*中，`Fish`值可以被添加到`current_pets`中，因为它作为另一个列表`all_pets`的一部分被创建。这说明了使用列表中值的一个主要问题：虽然它们对于向项目中引入新的可能值非常有用，但它们必须在可以访问之前被定义。任何新的列表都依赖于之前定义或在其赋值内创建的值。然而，在 ink 中，可以将值更改为另一种数据类型：

示例 10 (Example10.ink):

```cs
LIST standing_with_family_members =
father = 0,
mother = 1,
sister = 2,
brother = 0
```

在*示例 10*中，`standing_with_family_members`列表的每个值也被分配了一个数字。在 ink 中允许这样做，并且可以是一种创建与项目中的数值相关联的特定列表值的有用方式。然而，访问这些数字需要理解 ink 的另一个概念：**函数**。

# 调用函数

**函数**是大多数编程语言的基础部分。在 ink 中，函数是一组代码，可以接受由逗号分隔的输入，可能产生输出，并且可以通过称为**调用**的操作来访问。

函数通过使用其名称，然后打开(`(`)和关闭(`)`)括号来调用。在 ink 中调用函数的操作会暂时将故事的流程移动到函数的代码中，并在代码完成后返回。

注意

函数只能在 ink 中的代码内调用。这意味着它们要么出现在开闭花括号内，要么出现在以波浪号(~)开头的行上，作为变量重新赋值的组成部分。

在这个主题中，我们将首先回顾 ink 中内置的一些函数以及它们如何帮助我们进行常见操作。接下来，我们将查看专门设计用于与使用`LIST`关键字创建的值一起工作的函数。这些函数在列表上执行常见操作，例如告诉我们列表中的条目数量或从其集合中随机选择一个条目。

## 常见数学函数

ink 中最常用的函数之一是`RANDOM()`。它接受一个最小值和一个最大整数值。然后它从这个指定的范围内选择一个随机数字。

对于许多角色扮演游戏，一个常见的需求是在一定范围内的数字，例如 1 到 4 或 1 到 20。`RANDOM()`函数允许我们设置一个范围，然后查看结果：

示例 11 (Example11.ink):

```cs
An example of a dice roll of 1-to-20 is {RANDOM(1,20)}.
```

当运行*示例 11*代码时，ink 将遇到一组花括号。然后它会看到具有最小值 1 和最大值 20 的`RANDOM()`函数。每次运行时，都会选择这个范围内的不同数字：

![图 4.5 – ink 输出示例 11 的截图](img/Figure_4.5_B17597.jpg)

图 4.5 – ink 输出示例 11 的截图

ink 还具有用于在不同类型数字之间进行转换的函数。`INT()` 函数将十进制数字转换为整数（整数）数字，而 `FLOAT()` 函数将整数转换为十进制（浮点）数字。每个函数都接受一个数字并产生不同类型的数字输出：

示例 12 (Example12.ink):

```cs
VAR example_decimal = 3.14
VAR example_integer = 5
Convert a decimal into an integer: {INT(example_decimal)}.
Convert an integer into a decimal: {FLOAT(example_integer +
  1.3)}.
```

函数产生的值可以保存在变量中。这允许，例如，使用 `RANDOM()` 函数及其值，该值已作为变量的一部分保存：

示例 13 (Example13.ink):

```cs
A common table-top role-playing game combination is 2d6 where two dice rolls of 1-to-6 are rolled, and their values combined.
VAR dice_one = 0
~ dice_one = RANDOM(1,6)
VAR dice_two = 0
~ dice_two = RANDOM(1,6)
The combined total of 2d6 is {dice_one + dice_two}.
```

*示例 13* 代码包含两个变量和两次对同一函数的使用。正如在 *使用 VAR 存储值* 主题开头所提到的，变量必须以静态值开头。在 *示例 13* 代码中，每个变量最初被赋予 `0` 的值。它们立即被赋予由 `RANDOM()` 函数生成的值。然而，作为使用 `VAR` 关键字创建的变量的显式规则的一部分，它们必须在重新分配由 `RANDOM()` 等函数生成的动态变量之前被设置为静态值。

当运行时，*示例 13* 代码创建了必要的变量，并从 `RANDOM()` 函数的调用中重新分配它们的值，最小值为 `1`，最大值为 `6`。当 ink 遇到文本和大括号的使用时，它会将这两个值相加：

![图 4.6 – 示例 13 的 ink 输出截图](img/Figure_4.6_B17597.jpg)

![图 4.6 – 示例 13 的 ink 输出截图](img/Figure_4.6_B17597.jpg)

图 4.6 – 示例 13 的 ink 输出截图

提示

与使用洗牌替代方案和 `VAR` 关键字一样，`RANDOM()` 函数的输出不能用作变量的初始值。它必须首先创建，然后才能重新分配 `RANDOM()` 生成的值。

正如我们所见，ink 中有多个内置函数用于处理单个值。这也适用于使用 `LIST` 关键字创建的值。在下一节中，我们将回顾一些专门为列表及其值设计的函数。

## 使用 LIST 函数

虽然有针对单个值设计的函数，但 ink 中的大多数内置函数都是与列表一起使用的。这些函数都以 `LIST_` 前缀开头，并且它们执行的操作或访问的内容作为第二个单词。例如，要计算列表中包含的值的数量，可以使用 `LIST_COUNT()` 函数：

示例 14 (Example14.ink):

```cs
LIST days_of_week = 
Monday,
Tuesday,
Wednesday,
Thursday,
Friday,
Saturday,
Sunday
The total days are {LIST_COUNT(days_of_week)}.
```

当运行时，*示例 14* 代码创建了一个包含七个值的列表。在大括号中是对 `LIST_COUNT()` 函数的调用。然后，该函数将 `days_of_week` 列表作为参数传递。根据列表赋值中使用的行，默认假设输出将是 `7`，基于列表中的天数。然而，情况并非如此。它的输出是 `0`：

![图 4.7 – 示例 14 的 ink 输出截图](img/Figure_4.7_B17597.jpg)

图 4.7 – ink 为示例 14 生成的截图

*示例 14*产生的输出显示了使用列表值工作的一个隐藏方面。技术上，用于创建列表的所有值都称为`true`或`false`，并且默认情况下，所有值都设置为`false`。

`LIST_COUNT()`函数*计数*列表中`true`值的数量。在*示例 14*中，没有。该函数产生的计数是正确的。要将值从其默认的`false`更改为`true`，需要将其括在开括号(`(`)和闭括号(`)`)内：

示例 15 (Example15.ink):

```cs
LIST days_of_week = 
(Monday),
(Tuesday),
(Wednesday),
(Thursday),
(Friday),
(Saturday),
(Sunday)
The total days are {LIST_COUNT(days_of_week)}.
```

在*示例 15*中，输出包括数字`7`。这是正确的。现在，*示例 14*列表中的每个值现在都包含在其自己的括号内，每个值从`false`变为`true`。

对于需要从列表中获取每个值的情况，无论其是`true`还是`false`，`LIST_ALL()`函数都会返回*所有*值：

示例 16 (Example16.ink):

```cs
LIST days_of_week = 
(Monday),
(Tuesday),
(Wednesday),
(Thursday),
(Friday),
(Saturday),
(Sunday)
The days of the week are: {LIST_ALL(days_of_week)}.
```

在*示例 16*中，使用`LIST_ALL()`函数返回当前`days_of_week`列表的所有值：

![Figure 4.8 – Screenshot of ink's output for Example 16](img/Figure_4.8_B17597.jpg)

![Figure 4.8 – Screenshot of ink's output for Example 16](img/Figure_4.8_B17597.jpg)

图 4.8 – ink 为示例 16 生成的截图

`LIST_RANDOM()`函数返回一个关于列表中`true`值总数的随机条目，如下例所示：

示例 17 (Example17.ink):

```cs
LIST days_of_week = 
(Monday),
(Tuesday),
(Wednesday),
Thursday,
Friday,
Saturday,
Sunday
A random day of the week is: {LIST_RANDOM(days_of_week)}.
```

在*示例 17*中，只有`Monday`、`Tuesday`和`Wednesday`值被设置为`true`。由于`days_of_week`的其他值默认设置为`false`，因此不能通过`LIST_RANDOM()`函数访问。

返回到*示例 10*的代码，`LIST_VALUE()`函数可以用来访问作为列表一部分分配给任何值的任何数据：

示例 18 (Example18.ink):

```cs
LIST standing_with_family_members =
father = 0,
mother = 1,
sister = 2,
brother = 0
The value of sister is {LIST_VALUE(sister)}.
```

在这个改进版本的*示例 10*，即*示例 18*中，可以使用`LIST_VALUE()`函数来访问分配给`sister`值的已分配数据。

虽然墨水有许多用于执行不同选项的功能，无论是数学上还是与列表值相关，但它还向作者提供了创建自定义功能的能力。在下一个主题中，我们将回顾如何创建和调用函数。

# 创建新函数和调用节点

*调用函数*主题介绍了接受输入、可能产生输出以及如何调用 ink 内置函数。

在 ink 中也可以使用`function`关键字创建新函数。在 ink 中创建的任何新函数都可以像其他函数一样使用，并且它们通常是创建可用于整个项目或多次使用而无需再次编写相同代码的独立代码行的有用方式。

在这个主题中，我们将探讨如何使用`function`关键字创建新的函数。我们将学习它们如何被调用，执行小任务，甚至可能返回数据。在*第二章**，*“结、转向和循环模式”，我们最初讨论了结。故事的不同部分由名称定义。在 ink 中，函数，正如我们将学习的，是特殊的结类型。这种关系意味着结也可以*被调用*并*传递*数据。

函数使用至少两个等号（`=`）、`function`关键字、函数名称，然后是括号（`(` 和 `)`），括号围绕其输入（如果有）。函数的名称遵循与变量和结相同的规则：它们可以包含数字、字母和下划线字符。它们不能包含其他特殊符号或空格。

与变量一样，函数在 ink 中也是*全局的*。一旦创建，它们就可以被项目中的任何其他代码访问。由于变量和函数都是全局的，因此常见的模式是设计一个改变单个变量的函数。这允许作者定义在调用函数时发生的行为，例如增加或减少其当前值：

示例 19（Example19.ink）：

```cs
VAR money = 30
VAR apples = 0
VAR oranges = 0
You approach the marketplace and consider what is on sale. 
-> market
== market
You have {money} gold.
You have purchased {apples} apples.
You have purchased {oranges} oranges.
+ {money > 10} [Buy Apple for 10 gold]
    ~ decreaseMoney(10)
    ~ increaseApples()
    -> market
+ {money > 15} [Buy Oranges for 15 gold]
    ~ decreaseMoney(15)
    ~ increaseOranges()
    -> market
* [Leave market]
    -> DONE
== function decreaseMoney(amount)
~ money = money - amount
== function increaseApples()
~ apples = apples + 1
== function increaseOranges()
~ oranges = oranges + 1
```

*示例 19*使用了三个不同的函数。第一个，`decreaseMoney()`，接受一个名为`amount`的值。这是一个**参数**的例子。在创建函数时，可以在其开括号和闭括号内定义不同的变量。这些被称为其*参数*，它们影响其执行计算或处理的方式。

当调用函数时，传递给它的数据被称为其`decreaseMoney()`，它有一个参数并接收一个单一的参数，以及两个不接受参数的函数，`increaseApples()`和`increaseOranges()`。

`increaseApples()`和`increaseOranges()`函数的位置也符合 ink 中的一种常见模式，即在代码底部找到用于调整变量值的函数。因为它们都是全局的，这意味着可以从项目的任何地方访问它们，所以函数可以在项目的任何地方定义。

然而，函数，就像它们的姐妹概念结（knots）一样，定义自己为从开始到下一个结或函数之间的所有线条。将它们放在文件底部可以防止代码可能被混淆或被认为是另一个结或函数的一部分的问题。

函数不是唯一能够定义参数和接受参数的概念。在 ink 中，**结**也可以像函数一样被调用。这是因为函数是特殊的结类型，可以返回数据。这也标志着它们之间的区别。结可以接受数据，但只有函数可以返回数据。然而，以这种方式使用结可以让我们轻松跟踪循环结构中的值：

示例 20（Example20.ink）：

```cs
-> time_machine(RANDOM(20,80))
== time_machine(loop)
The large machine looms over everything in the room. With flashing lights, odd wires running between parts, and a presence all its own, it seems to be almost a living, pulsating thing as the scientist runs between sections parts and adjusts various parts.
"I'm so close!" he shouts as he turns a knob and then pulls down a lever. "I just need more time to figure out how to control the loops!"
You regard him and the machine skeptically.
"If you could, just press that last button and everything should be all set for the first demonstration of my time machine! I'm so glad the newspaper sent you to cover this event," he says, adjusting more settings on the grand machine in front of you.
You pause to try to understand the blinking lights as he yells again. "Press the button for me! I just need to make some last-minute changes over here."
On the panel in front of you is a large, green button. You consider it and the scientist rushing around across the room.
+ [Press button]
    ~ loop = loop + 1
    There is a flash of light and the readings on the 
      machine show a message: "This is loop {loop}."
    -> time_machine(loop)
```

在**示例 20**中，存在一个变量，它是作为`time_machine`节点的参数创建的。在循环开始之前，使用`RANDOM()`函数从`20`到`80`的范围内选择一个值。这个值在第一次循环中传递给节点。每当玩家选择`loop`值增加一个，并将当前值传递给`time_machine`节点。在未来的任何循环中，`loop`变量增加一个，并将它的当前值发送到下一个循环。

**示例 20**中的代码还展示了如何在不使用`VAR`关键字的情况下使用变量。在节点内部，`loop`变量作为一个参数存在。这意味着它作为一个变量存在，但仅限于`time_machine`节点内部。以这种方式使用时，`loop`变量将不是全局的。作为`time_machine`节点的一部分，`loop`变量不能在其代码之外使用。

函数是墨水中的一个强大概念。然而，与使用节点相比，它们确实有两个主要的限制。第一个限制是函数不能使用任何类型的选项。函数不能分支故事，并且在完成时必须*返回*到它们被调用的位置。第二个限制是函数不能偏离到故事的另一个部分。就像第一个限制一样，函数应该只执行一个小任务或更改一个值。

将节点调用得像函数一样非常有用，对于许多项目来说都是如此。然而，与函数不同，节点不能返回值。正如**示例 20**所示，可以使用节点来完成一些与墨水中的函数相同的目的，但不是所有。作者必须考虑使用函数或节点来完成任务或展示信息哪种方式更好。如果目标是处理数据并返回值，函数是最好的选择。如果需要传递数据、提供选项或故事以某种方式偏离，节点是组织代码和数据更好的方式。

# 摘要

在本章中，我们更多地了解了变量是如何工作的，以及如何使用`VAR`关键字来创建它们。使用多种类型的数据，必须使用静态值来创建变量。然后可以通过使用以波浪号（`~`）开头的行进行赋值操作来更改它们。

在第二个主题中，对于需要多个值的情况，我们看到了可以使用`LIST`关键字。这个关键字允许我们创建其他变量可以使用的值，但也带来了限制，即只有使用`LIST`创建的值才能与列表一起使用。我们还研究了列表的所有值都是一个布尔集合的一部分，并且在创建时具有`true`或`false`值。

接下来，在第三个主题中，我们研究了函数在墨水中的工作方式。通过几个内置函数，我们可以创建随机数或在不同类型的数字之间进行转换。使用`LIST`值，我们通过检查如何将列表的值从创建时的`true`更改为`false`，比较了`LIST_COUNT()`和`LIST_ALL()`的结果。

最后，在最后一个主题中，我们使用`function`关键字编写了一些函数来执行简单的任务，例如调整变量的值。因为两个变量都是使用`VAR`关键字创建的，而函数是全局的，所以我们看到一种常见的模式是使用函数来改变变量的值。作为这个主题的一部分，我们还回顾了节点，并了解到函数是特殊的节点类型。这允许它们都通过参数接收数据，尽管只有函数可以返回数据。

正如我们将在接下来的章节中看到，结合 ink 和 Unity，理解值在 ink 中的存储和访问方式对于创建统一的项目至关重要。在我们能够在 Unity 中处理代码之前，我们必须了解 ink 如何与使用`VAR`关键字和`LIST`关键字创建的不同值一起工作。通过理解变量和函数之间的关系，我们可以开始编写 ink 代码，并且可以在 Unity 中与 C#代码一起运行，这要晚得多。

在下一章，*第五章**，* *隧道和线程*，我们将探讨 ink 中的最后两个主要概念：**隧道**和**线程**。利用上一章介绍的大多数概念，我们将使用隧道在 ink 中创建具有非常少代码的高级结构。使用线程，我们将把数字故事分成更多部分，并让 ink 为我们这个读者将一切组合起来，当我们从一个节点被引导到另一个节点时。这将创建一个复杂的叙事体验，基于理解和管理故事段落之间的故事流程。 

# 问题

1.  变量获得值的操作叫什么？

1.  当通过“添加”两个其他字符串或一个字符串和一个数字来创建字符串时，这个操作叫什么？

1.  在 ink 中，波浪号（`~`）是如何与变量和代码一起使用的？

1.  列表值的类型是什么？

1.  什么是作为函数或节点的一部分创建并在其括号内定义的变量的技术术语？
