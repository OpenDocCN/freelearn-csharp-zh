# 第六章：委托和 Lambda 表达式之间的连接

在本章中，我们将看一下委托和 Lambda 表达式之间的连接。

# 在 HTML 中添加一个显示结果的按钮

打开一个项目，在<html>中放入一个显示结果的按钮。要做到这一点，去工具箱中找到`Button`控件。将其拖放到以`<form id=...`开头的行下面。你可以删除`<div>`行，因为你不需要它们。确保在按钮行的末尾插入一个`<br>`标签：

```cs
<asp:Button ID="Button1" runat="server" Text="Show Results" /><br />
```

我将做一些大杂烩的事情，只是为了向你展示不同的概念。

转到设计视图，并双击显示结果按钮。这将带我们进入`Default.aspx.cs`。删除`Page_Load`块。该项目的初始代码屏幕应该如*图 6.1.1*所示：

![](img/e3e4308a-8be4-4a24-bef4-e473f2ba9749.png)

图 6.1.1：该项目的初始 Default.aspx.cs 代码

# 添加委托

在第一阶段，你需要添加委托。虽然你可以把它们放到一个单独的文件中，但为了我们的目的，让我们把它们放在这里。因此，在以`public partial class...`开头的行上面输入以下内容：

```cs
public delegate bool Compare(double x, double y);
```

记住，委托实际上是函数或方法的包装器。然后，在这一行的正下方，输入以下内容：

```cs
public delegate double Multiply(double x, double y);
```

你可以看到我们有两个委托。一个返回`Boolean`数据类型，另一个返回`double`数据类型。

# 设置变量

接下来，在`Button1_Click`的事件处理程序中，我们将创建两个变量：`x`（设置为`10`）和`y`（等于`25`）。因此，在大括号之间输入以下内容：

```cs
double x = 10, y = 25;
```

# 创建委托类型的对象

现在，我们要在上一行下面输入以下内容：

```cs
Compare comp = (a, b) => (a == b);
```

当你开始输入`Compare`时，注意到从弹出窗口，一旦你有了一个委托（`Compare`），实质上，你可以创建那种对象；然后，输入`comp`。

# 定义 Lambda 表达式

现在，要定义一个 Lambda 表达式，你需要输入`= (a,b)`，如所示。然后这将被映射到后面的操作；所以你可以把`=>`看作映射符号或映射操作符。它将被映射到`(a==b)`操作。因此，换句话说，`comp`将允许我们检查这两个值是否相同，这发生在`a`和`b`被比较的阶段。基本上，`(a, b)`是参数，被评估的表达式是`a`是否等于`b`。

接下来，输入以下内容：

```cs
sampLabel.Text = $"{x} and {y} are equal is {comp(x, y).ToString().ToLower()}";
```

要调用这个，注意你要输入`comp`，然后传入`x`和`y`的值。然后，为了显示你可以进一步操作它，一旦你得到了结果，你可以将其转换为字符串版本，然后全部转换为小写，就像前面的代码行所示。

记住，这是*函数链接*，所以它从左到右执行。换句话说，首先运行`comp`，然后是`ToString`，最后是`ToLower`。

另外，请注意，在运行时，当你传入`x`和`y`值时，调用`comp(x, y)`，基本上就是`(a==b)`会被触发；比较将会被执行，并且值将被返回。

接下来，我们还可以做`Multiply`委托，所以在这一行下面输入以下内容：

```cs
Multiply mult = (a, b) => (a * b);
```

注意到`(a,b)`可以被使用和重复使用等等。记住，这里的`(a,b)`是参数，你可以使用它们并重复使用它们。它们在出现的每一行中都是局部的。因此，你可以在另一行中使用它。然后，你再次说`(a,b)`映射到`(a*b)`的操作。以分号结束。

现在，要调用这个乘法委托（它代表的 Lambda 表达式），从上面复制（*Ctrl* + *C*）`sampLabel.Text`行，然后粘贴（*Ctrl* + *V*）到下面，如下所示：

```cs
sampLabel.Text += $"<br>{x}*{y} is {mult(x, y).toString()}";
```

在这里，我们说`{x}*{y}`，然后`+=`追加，并删除`are equal`，并用我们对象的名称`mult`替换`comp`。你不需要`toString`让它工作，而且因为它会返回一个数字，你也不需要`ToLower`。

# 操作数组

现在，在下一个阶段，你可以做的另一件事是操作一个数组。例如，你可以创建一个双精度数组。我们将其称为`dubsArray`，这将是一个新的双精度数组。要做到这一点，在下一行输入以下内容：

```cs
double[] dubsArray = new double[] { 1, 2, 3, 4, 5 };
```

# 使用操作

现在，我们将讨论操作，所以在下一行输入以下内容：

```cs
Action<double> showDouble = (a) => sampLabel.Text += "<br>" + (a * a);
```

注意，“操作”是一个代理。所以，如果你右键单击“操作”并选择“转到定义”，你会看到`public delegate void Action()`。如果你展开它，它说，封装了一个没有参数并且不返回值的方法。这是.NET 中操作的基本定义。

但是，你可以扩展一个“操作”代理。它们可以是通用的。例如，如果你输入`Action<double>`，然后右键单击它并再次选择“转到定义”，这个特定形式确实需要一个参数。如果你展开它，参数部分说，这个代理封装的方法的参数。此外，“摘要”部分说，封装了一个具有单个参数并且不返回值的方法。所以，再次强调，没有必要猜测。右键单击并选择“转到定义”或将鼠标悬停在上面。它会告诉你需要知道的内容。在我们的情况下，实际上将是在前一行中看到的`showDouble`。现在，另一个 lambda 可以用来定义这个；所以你在那里插入`(a)`作为单个参数，然后输入映射符号`=>`，然后是`sampLabel.text`。你想要将这个附加到现有的文本上，所以输入`+=`，然后说，`<br>`，然后显示`a`的平方，输入`+ (a * a)`并以分号结束。

现在记住从“操作”的定义中，它们不返回值，对吧？事实上，如果我们输入“Action<double>”，并查看弹出提示，如果你浏览整个列表直到 T16，它说，封装了一个具有 16 个参数并且不返回值的方法，如*图 6.1.2*所示：

![](img/290730d1-917b-43a2-a32e-80470a439338.png)

图 6.1.2。在输入 Action<double>后，没有一个操作返回值，

所以，它们都不返回值。这是“操作”作为它们在这里定义的一个基本特性，但请记住，最终它只是一个代理。

然后，例如，要利用这些“操作”，你可以做的一件事是输入以下内容：

```cs
foreach(var d in dubsArray)
```

在下一个阶段，在这行下面的花括号之间输入以下内容来调用这些操作：

```cs
showDouble(d);
```

这些是使用代理和 Lambda 表达式的基础。文件顶部的两个代理是我们程序的核心，然后是`Compare`和`Multiply`，它们是下面使用的代理类型，然后是使用这些代理定义的 Lambda 表达式，如`(a, b) => (a == b)`，`(a, b) => (a * b)`，和`(a) => sampLabel.Text += "<br>" + (a * a)`。

现在，用浏览器查看一下。点击“显示结果”按钮。它说，10 和 25 相等是假的，10*25 是 250，然后打印出了平方。这些是基本的结果，一切看起来都是应该的：

![](img/9925cc6d-8d67-430e-b075-7fa7c6f5fcd5.png)

图 6.1.3。运行我们的程序的结果

# 章节回顾

出于复习目的，包括注释的本章的`Default.aspx.cs`文件的完整版本如下所示：

```cs
//using is a directive
//System is a name space
//name space is a collection of features that our needs to run
using System;
//public means accessible anywhere
//partial means this class is split over multiple files
//class is a keyword and think of it as the outermost level of grouping
//:System.Web.UI.Page means our page inherits the features of a Page
public delegate bool Compare(double x, double y);
public delegate double Multiply(double x, double y);
public partial class _Default : System.Web.UI.Page
{
    protected void Button1_Click(object sender, EventArgs e)
    {
        double x = 10, y = 25; //declare two variables
        //the two variables are accessible inside the lambda expressions
        Compare comp = (a, b) => (a == b);//define comparison lambda
        //invoke the lambda in the line below
        sampLabel.Text =
         $"{x} and {y} are equal is {comp(x, y).ToString().ToLower()}";
        //line define a lambda for multiplication
        Multiply mult = (a, b) => (a * b);
        //invoke the multiplication lambda
        sampLabel.Text += $"<br>{x}*{y} is {mult(x, y)}";
        //make array of doubles
        double[] dubsArray = new double[] { 1, 2, 3, 4, 5 };
        //actions encapsulate functions that do not return a value
        //but actions can accept arguments to operate on
        Action<double> showDouble = 
        (a) => sampLabel.Text += "&lt;br>" + (a * a);
        //it's now possible to perform the action on each d repeatedly
        foreach (var d in dubsArray)
        {
            showDouble(d);
        }
    }
}
```

# 总结

在本章中，你学习了代理和 lambda 表达式之间的关系。你添加了代理，设置了项目变量，创建了代理类型的对象，操作了一个数组，并使用了“操作”。

在下一章中，你将学习关于表达式主体成员以及由代码块定义的 lambda 表达式。
