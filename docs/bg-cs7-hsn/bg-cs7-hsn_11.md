# 第十一章：使用查询语法构造查询

在本章中，您将学习如何使用查询语法编写查询，例如方法链接，就像我们以前做过的那样。

# 向 HTML 添加显示按钮

打开一个项目，唯一放入<html>中的是一个按钮，没有其他内容。为此，请转到工具箱，获取一个`Button`控件，并将其拖放到以`<form id=...`开头的行下方。将按钮上的文本替换为显示：

```cs
<asp:Button ID="Button1" runat="server" Text="Show" />
```

现在，切换到设计视图，并双击显示按钮。这将带我们进入`Default.aspx.cs`。删除事件处理存根。此项目的起始代码的相关部分应如*图 11.6.1*所示：

![](img/c34a42e9-1784-4147-9aed-cd71dc1f298c.png)

图 11.6.1：此项目的起始代码部分

接下来，转到文件顶部，在`using System`下输入以下内容：

```cs
using System.Collections.Generic;
using System.Linq;
```

为了利用这一点，我们将做如下操作。这是例行代码；这是机械的。首先，当有人单击显示按钮时，您希望创建一个标签，以便始终有一个累积输出。为此，请在以`protected void Button1_Click...`开头的行下的大括号之间输入以下内容：

```cs
sampLabel.Text = "";
```

# 创建一个 decimal 工资数组

接下来，在上一行下面，您将创建一个名为`salaries`的`decimal`数组，自然而然地。因此，输入以下内容：

```cs
decimal[] salaries = new decimal[] { 56789, 78888, 35555, 34533, 75000 };
```

这就是您可以查询`decimal`数组的方法。这是一个特定的`decimal`数组，但基本上可以是任何数组。我们加入了一些值，就这样。

# 使用范围变量

接下来，在此行下面输入以下内容：

```cs
IEnumerable<string> salResults = from salary in salaries
```

请注意，返回或结果集将是`string`类型，而不是`decimal`类型。在`salResults =`之后，您希望定义 LINQ 查询的主体，因此您说`from salary in salaries`。如果您将鼠标悬停在此处的`salary`上，您会看到它被称为*范围变量*，如*图 11.6.2*所示。因此，您要求它查看`salaries`。作为范围变量，它是逐个遍历所有条目的数量。

![](img/b52e9e05-d447-46e1-8bfd-173af7d283a7.png)

图 11.6.2：范围变量

# 选择工资范围并按降序排列

现在，您将指定某种逻辑条件。例如，以某种方式过滤结果。因此，在此行下缩进输入以下内容：

```cs
where 35000 <= salary && salary <= 75000
select $"<br>{salary:C}";
```

接下来，您可以对结果集进行`orderby`，以按降序列出工资，例如；因此，请直接在此行下面输入以下内容：

```cs
orderby salary descending
```

默认情况下是按升序排列的，从小到大，您希望将其反转。当您加入`descending`关键字时，它就会从大到小排列。

接下来，记住目标是获得一个填充有字符串的`IEnumerable`构造。因此，最后，请在此代码块的一部分中输入以下内容：

```cs
select $"<br>{salary:C}";
```

您还可以在原地格式化结果，就像我在这一行中所做的那样，例如，以货币格式。

# 显示结果

有了这个代码块，当然，下一阶段是迭代并显示结果。为此，您可以进行转换为列表并打印，或者您可以接下来做以下操作：

```cs
foreach(string formattedSalary in salResults)
```

我们怎么知道在这一行中应该说`string`？记住，前面的`IEnumerable`行填充有字符串，对吗？如果您将鼠标悬停在`IEnumerable`上，它会说`IEnumerable<out T>`，而`T`是字符串。

现在，为了显示结果，请在上一行下面的一对大括号之间输入以下内容：

```cs
sampLabel.Text += formattedSalary;
```

在这里，`formattedSalary`是要显示的内容。

现在，在浏览器中打开此内容。单击显示按钮。结果将显示如*图 11.6.3*所示：

![](img/6e88179b-abdd-4c82-9165-450db2f5d507.png)

图 11.6.3：运行程序的初始结果

这里的工资按降序排列，从高到低，并且在 75000 美元到 35000 美元的范围内。所以，这正如预期的那样工作。

# 观察延迟执行

现在，你应该知道的一件事是所谓的*延迟执行*的概念。所以，为了了解这意味着什么，看看接下来的内容。

想象一下，我在`foreach(string formattedSalary in salResults)`行右边放了一个断点。然后，从调试菜单中选择单步执行，并点击显示按钮。注意每一行是如何连续运行的（每行后面都会显示毫秒数）。你应该看到它是如何进入的；它是如何运行的。

你应该注意的一件事是 LINQ 的延迟执行的概念，这意味着`salResults`实际上是运行的，正如你所看到的，所以它本质上是一个查询变量。它在你使用`foreach`循环迭代时运行，就像下面的代码块中所示的那样，而不是在你在前面的`IEnumerable`块中编写时运行。它不会在那时运行。它会在你迭代它时运行。否则，你的程序可能会在这些查询结果中携带大量信息。所以，这就是延迟执行的内容：

```cs
IEnumerable<string> salResults = from salary in salaries
                                 where 35000 <= salary && salary <= 75000
                                 orderby salary descending
                                 select $"<br>{salary:C}";
        foreach(string formattedSalary in salResults)
        {
            //display formatted salaries one at a time
            sampLabel.Text += formattedSalary;
        }
```

在下一阶段，我们将看一个可能能做的另一个实际例子。我们还想显示水平线，所以在`sampLabel.Text += formattedSalary`行下面的闭合大括号之后，输入以下内容：

```cs
sampLabel.Text += "<br><hr/>";
```

`<hr/>`标签将在输出中添加一条水平线。

# 创建一个字典

接下来，我们将创建一个`Dictionary`；为此，请在下面输入以下行：

```cs
Dictionary<string, decimal> nameSalaries = new Dictionary<string, decimal>();
```

在这里，`<string,decimal>`代表键值对。

# 处理键值对

现在，让我们添加一些键值对。所以，从输入以下内容开始：

```cs
nameSalaries.Add("John Jones", 45355);
```

在这一行中，`John Jones`是键，值是他的薪水，或者$45,355。

然后，你可以重复这个几次，所以直接在它下面再粘贴这一行三次。比如，约翰·史密斯，76900；约翰·詹金斯，89000；史蒂夫·乔布斯，98000：

```cs
nameSalaries.Add("John Smith", 76900);
nameSalaries.Add("John Jenkins", 89000);
nameSalaries.Add("Steve Jobs", 98000);
```

请注意，我在这里重复了名字*约翰*几次，因为我想简要说明一个概念。最后一个列出的是*史蒂夫·乔布斯*，当然他的薪水远远超过了 98000！

# 查询键值对中的数据

现在，我们将再次查询这个。这是我们在键值对中拥有的数据，我们将对其进行查询。所以，在这些行下面输入以下内容：

```cs
var dictResults = from nameSalary in nameSalaries
```

在这里，`nameSalary`是一个范围变量，它指的是约翰·琼斯、约翰·史密斯、约翰·詹金斯、史蒂夫·乔布斯等等，而`nameSalaries`是字典本身。`nameSalary`是键和值的特定组合。

然后，在这一行下面，缩进以下代码：

```cs
where nameSalary.Key.Contains("John") && nameSalary.Value >= 65000
```

在这里，我们说键包含名字约翰，薪水大于或等于$65,000。如果你愿意，你可以添加`OrderBy`等等，但对于我们的目的，直接在这一行下面输入以下内容：

```cs
select $"<br>{nameSalary.Key} earns {nameSalary.Value:C} per year.";
```

在这一行中，我们将选择那些符合这两个条件的记录：姓名是约翰，薪水超过$65,000。所以，在我们的情况下，肯定是约翰·史密斯和约翰·詹金斯。为了使输出看起来好看，我们说`nameSalary.Value:C`以货币格式化，然后添加`per year`。

现在，将鼠标悬停在`dictResults`上。你看到弹出提示中说`IEnumerable`吗？现在，`var`被称为*隐式类型*。我们之前见过`var`。有时很难从像我们创建的这样的查询中判断输出会是什么，因为它足够复杂。所以，如果你使用隐式类型，它会告诉你输出应该是什么，所以是`string`类型的`IEnumerable`。现在，如果你愿意，你可以将这一行改为：

```cs
IEnumerable<string> dictResults = from nameSalary in nameSalaries
```

现在我们也知道这一点，因为在这个查询的末尾，你看到了字符串，对吧？这些是包含格式化信息的字符串，当然，你可以像往常一样迭代它们；所以，接下来输入以下内容：

```cs
foreach(string nameSal in dictResults)
```

然后，在这一行下面的一对大括号之间，以以下内容结束：

```cs
sampLabel.Text += nameSal;
```

# 运行程序

在浏览器中运行这个，确保它按预期工作。点击显示按钮。结果显示在*图 11.6.4*中：

![](img/3a14c5ac-533f-4199-a81f-eb700c65164b.png)

图 11.6.4：运行我们程序的结果

现在你已经得到了我之前描述的第一组结果，第二组结果显示在它们下面，显示两个名字都包含 John，金额为$65,000 或更多。

# 章节回顾

作为回顾，包括注释的本章`Default.aspx.cs`文件的完整版本如下所示：

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
        //clear label on button click
        sampLabel.Text = "";
        //make array of salaries
        decimal[] salaries = 
        new decimal[] { 56789, 78888, 35555, 34533, 75000 };
        //construct Linq query, which produces a collection of 
        //formatted strings
        IEnumerable<string> salResults = from salary in salaries
                              where 35000 <= salary && salary <= 75000
                              orderby salary descending
                              select $"<br>{salary:C}";
        foreach(string formattedSalary in salResults)
        {
            //display formatted salaries one at a time
            sampLabel.Text += formattedSalary;
        }
        //show horizontal rule on screen
        sampLabel.Text += "<br><hr/>";
        //make dictionary to hold names and salaries as key/value pairs
        Dictionary<string, decimal> nameSalaries = 
        new Dictionary<string, decimal>();
        nameSalaries.Add("John Jones", 45355);
        nameSalaries.Add("John Smith", 76900);
        nameSalaries.Add("John Jenkins", 89000);
        nameSalaries.Add("Steve Jobs", 98000);
        //query below represents all people named John who make 65000 
        //and more
        //this query gives back a formatted string for each key/value 
        //pair that 
        //satisfies the condition
        IEnumerable<string> dictResults = from nameSalary in nameSalaries
                            where nameSalary.Key.Contains("John") && 
                            nameSalary.Value >= 65000
                            select $"<br>{nameSalary.Key} earns 
                            {nameSalary.Value:C} per year.";
        foreach(string nameSal in dictResults)
        {
            sampLabel.Text += nameSal;//display named and salaries
        }
    }
}
```

# 总结

在本章中，你学会了如何使用查询语法编写查询。你创建了一个十进制薪水数组，使用了范围变量，观察了延迟执行，创建了一个字典，使用了键值对，查询了键值对中的数据，并学习了隐式类型。

在下一章中，我们将进一步探讨 LINQ。具体来说，我们将研究 LINQ 的一些强大功能，如平均、求和和计数等聚合函数。此外，我们还将讨论列表的列表，这是非常实用的东西。
