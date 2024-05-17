# 第十二章：执行聚合函数的查询

在本章中，我们将进一步探讨 LINQ。具体来说，我们将看看 LINQ 执行聚合函数的能力，比如平均值、求和、计数等。此外，我们还将讨论列表的列表，这是一个非常实用的东西。

# 在 HTML 中添加显示按钮

打开项目，并且为了保持简洁，我们将在以`<form id=....`开头的行下面放一个按钮。要做到这一点，去工具箱，拖动一个`Button`控件，并把它拖到那里。将按钮上的文本更改为`Show`。

现在，切换到设计视图，并双击显示按钮。这将带我们进入`Default.aspx.cs`。删除`Page_Load`块。我们不需要那个。这个项目的起始代码的相关部分应该看起来像*图 12.7.1*：

![](img/578ad64b-616f-4e92-9494-7532354fad7b.png)

图 12.7.1：这个项目的起始代码部分

在下一阶段，转到文件顶部，在`using System`下面输入以下内容：

```cs
using System.Collections.Generic;
using System.Linq;
```

# 创建一个数组

在本章中有很多要输入的代码，但这是机械的。首先，我们将创建一个数组，所以在以`protected void Button1_Click...`开头的行的大括号之间输入以下内容：

```cs
IEnumerable<int> scores = new int[] { 45, 98, 99, 78, 89, 87, 77, 67, 71, 81 };
```

在这里，`IEnumerable`是数据类型，`scores`是数组的名称。你放入数组的值并不重要。

# 对列表中的值求平均值

现在，首先我们将找到这个列表的平均值。所以，输入以下内容：

```cs
var goodStudentAverage = (from score in scores where score >= 90 select score).Average();
```

我们将选择得分为`90`或以上的学生。想象一下，这些是几个学生的学期成绩。因此，在前一行中，我们说得分是`>=90`，选择那个分数。这是一个你可以在一行中写的查询。在这个上下文中，`score`是范围变量，`scores`是数组，选择的条件是`where score=>90`。然后，你输入`.`（点）`Average()`来平均整个东西。换句话说，这样写的方式是，括号中的查询将运行，然后平均数组中的值。如果你将鼠标悬停在这一行的`var`上，你会看到它说`double`，因为，如果你将鼠标悬停在`Average`上，你也会看到它返回一个`double`。因此，这个`Average()`函数作用于`IEnumerable`类型的列表，但它返回一个`double`数据类型给我们。

# 显示结果

现在，你当然可以显示结果，因为记住它只是一个单一的数值，一个聚合数量。你现在可以在这一行下面说以下内容：

```cs
sampLabel.Text = $"<br>The average for great students is {goodStudentAverage}";
```

# 使用 Count 函数

现在，如果你愿意，你也可以使用`Count`函数，所以你可以说下面的内容：

```cs
var averageStudentCount = scores.Where(grade =>70 <= grade && grade <80).Count();
```

在以`var`开头的前一行中，我们在单行中使用了查询语法，或者内联查询语法，因为我们使用了`from`和`where`。现在，我们可以使用方法链接和其中的 Lambda 表达式来表达相同的事情。所以，这里我们说`scores.Where`，然后我们说`grade`是这样的，即`70 <=grade`，但`grade <80`。因此，我们正在定义那些得分在`70`和`80`之间的人，不包括`80`的分数，并且我们将它们标记为平均学生。然后我们将`Count`它们。这将告诉我们有多少这样的人，然后我们可以显示那个数字。例如，你可以输入以下内容：

```cs
sampLabel.Text += $"<br>There are {averageStudentCount} average students.";
```

记住，`averageStudentCount`产生一个数字，所以，例如，结果可能是，*有 25 个平均学生*。

# 使用列表的列表

现在，这个概念的一个非常现实的应用可能是有一个列表的列表。首先输入以下内容：

```cs
List<int> firstStudent = new List<int> { 90, 89, 92 };
```

想象一下，您有一个学生`firstStudent`。然后，他或她有一些成绩，所以您创建了整数的`new List`，然后在一对大括号中初始化了这个列表。因此，按照所示的方式添加一些值。（请注意，我输入的值在`90` +/-范围内。）这是您可以以前未见过的方式初始化列表。

现在，让我们再为另一个学生做一个整数的列表。为此，输入以下内容以为`secondStudent`创建`new List`。同样，使用另一组值初始化此列表。（请注意，在这一行中，我将输入的值在`80` +/-范围内。）现在，当您有一个完整的班级时，您将有这样的列表，对吧？这是因为您有多个学生在一个班级中：

```cs
List<int> secondStudent = new List<int> { 78, 81, 79};
```

因此，现在您可以创建构造函数。接下来输入以下内容：

```cs
List<List<int>> classList = new List<List<int>>();
```

# 向 classList 添加学生

在这里，我们有一个整数的列表列表——您可以在其他列表中嵌入列表。然后，我们会说，例如，`classList`列表等于一个新的列表列表。要初始化此列表，您可以使用`Add`。在下一行，输入以下内容：

```cs
classList.Add(firstStudent);
classList.Add(secondStudent);
```

这是如何将第一个学生、第二个学生等添加到班级列表中的方法。

# 总结 classList 中的信息

在下一阶段，您希望能够获得一些有用的信息。例如，想象一下您有这样一个列表的列表，您希望进行总结。因此，接下来输入以下内容：

```cs
var avgPerStudent = classList.Select(student => student.Average());
```

现在，以`avgPerStudent`为例，表示平均学生分数。现在，在输入`classList.Select()`之后，要选择的数量是代表每个学生的列表，由`(student => student.Average())`捕获。现在，请确保您理解`student`参数是什么。在这里，您选择一个学生并平均他们的成绩。将鼠标悬停在`student`上，您会看到该数量代表与第一个学生对应的整数列表。然后，`student.Average`表示对该学生进行平均，然后对下一个学生重复此过程。如果将鼠标悬停在`var`上，您会看到在这种情况下返回的是`IEnumerable`类型。您可以迭代这些值。要做到这一点，您将输入以下内容：

```cs
foreach(var studentAvg in avgPerStudent)
```

现在，在此行下方，输入以下内容以在一对大括号中显示结果：

```cs
sampLabel.Text += $"<br>Average grade={studentAvg}";
```

# 运行程序

现在，构建此程序并在浏览器中运行它。单击“显示”按钮：

![](img/69bcb0bf-e199-4312-af4e-1d98e351469b.png)

图 12.7.2：运行我们程序的结果

现在这些是一些专业的结果。优秀学生的平均分是 98.5。有三个平均学生。两个列表的扩展平均成绩显示在最后。

因此，您学到了更多关于 LINQ 的知识——`Average`函数和`Count`函数，还学会了如何制作一个列表的列表。您可以使用`Select`等语句对这些列表进行操作，然后可以嵌入 Lambda 表达式以单独处理列表中的每个列表。

# 章节回顾

回顾一下，包括注释在内的本章的`Default.aspx.cs`文件的完整版本如下所示：

```cs
//using is a directive
//System is a name space
//name space is a collection of features that our needs to run
using System;
using System.Collections.Generic;
using System.Linq;
//public means accessible anywhere
//partial means this class is split over multiple files
//class is a keyword and think of it as the outermost level of grouping
//:System.Web.UI.Page means our page inherits the features of a Page
public partial class _Default : System.Web.UI.Page
{
    protected void Button1_Click(object sender, EventArgs e)
    {
        IEnumerable<int> scores = 
        new int[] { 45, 98, 99, 78, 89, 87, 77, 67, 71, 81 }; 
        //array of integers
        //line 17 below selects all scores 90 or above, and averages them,
        //giving back a double value
        var goodStudentAverage = (from score in scores where score >= 90 select score).Average();
        //line 19 below displays the average
        sampLabel.Text = $"<br>The average for great students is {goodStudentAverage}";
        //line 21 below selects all students below 70 and 80,
        //and counts them
        var averageStudentCount = scores.Where(grade => 70 <= grade && grade < 80).Count();
        //line 23 below displays the student count
        sampLabel.Text += $"<br>There are {averageStudentCount} average students.";
        //lines 25 and 26 create two new lists with initializer lists
        List<int> firstStudent = new List<int> {90,89,92};
        List<int> secondStudent = new List<int> { 78, 81, 79 };
        //line 28 creates a list of lists
        List<List<int>> classList = new List<List<int>>();
        classList.Add(firstStudent);
        classList.Add(secondStudent);
        //line 32 below find the average for each list, and 
        //stores the averages
        //so avgPerStudent is of type IEnumerable
        var avgPerStudent = classList.Select(student => student.Average());
        //lines 35-38 display the averages
        foreach(var studentAvg in avgPerStudent)
        {
            sampLabel.Text += $"<br>Average grade={studentAvg}";
        }
    }
}
```

# 摘要

在本章中，我们进一步探讨了 LINQ。具体来说，我们研究了 LINQ 执行聚合函数（如平均、求和和计数）的能力。此外，我们还讨论了列表的列表。您对列表中的值进行了平均，使用了`Count`函数，处理了列表的列表，向`classList`添加了学生，并总结了`classList`中的信息。

在下一章中，您将学习关于元组的知识，它们基本上是包含多个值的集合。
