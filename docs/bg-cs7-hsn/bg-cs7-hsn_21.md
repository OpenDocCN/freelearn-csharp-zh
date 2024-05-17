# 第二十一章：使用 LINQ 操作来自 SQL Server 的表

在本章中，您将学习如何将 LINQ 与 SQL 和 SQL Server 一起使用。

# 更改 ExperimentValues 表中的数据

我们将使用在上一章中创建的数据库表`ExperimentValues`，如*图 21.6.1*所示：

![](img/af7483e3-85ce-4d88-9e4b-b5f558bc4805.png)

图 21.6.1：第二十章的 ExperimentValues 表

请记住，表中有一个`Id`字段（PK，主键整数，非空），然后是`XValues（十进制，（18, 3）`，表示宽 18 个单位，小数点后 3 位，然后左边 15 个单位，总共 18 个单位。如果你愿意，你可以将其设置为`null`。同样，`YValues（十进制，（18, 3）`；所以，小数点后 3 位，然后左边 15 个单位，总共 18 个单位。

现在确保你在里面有数据。所以，右键单击`dbo.ExperimentValues`并选择查看数据。你应该看到我们在上一章中输入的数据。当然，你可以随时更改它。为了使事情变得更容易，让我们将值更改为*图 21.6.2*中显示的值：

![](img/f0ecdcfe-09f3-4fd1-a485-cf7f7dd35f1f.png)

图 21.6.2：ExperimentValues 表的新数据

如果你愿意，你可以重新加载它以查看它是否已保存。这就是我们简单的数据库表。

# 总结字段

现在我们将进入并总结字段。您将使用 LINQ 找到`X`值的总和和`Y`值的总和。首先，进入<html>，并在以`<form id= ....`开头的行下方放置一个按钮。转到工具箱（*Ctrl* + *Alt* + *X*），获取一个`Button`控件，并将其拖放到那里。更改按钮上的文本以显示 Sum Fields。当然，还可以执行其他几个操作。这只是一个操作：求和。

关闭工具箱并切换到设计视图。双击 Sum Fields 按钮。这将带我们进入`Default.aspx.cs`。删除`Page_Load`块。该项目的起始代码的相关部分应如*图 21.6.3*所示：

![](img/6540bf85-5f28-45fb-9087-4753daf880e7.png)

图 21.6.3：该项目的起始代码部分

# 添加命名空间

首先，在文件顶部的`using System`下，输入以下所有必要的行：

```cs
using System.Data.SqlClient;
using System.Linq;
using System.Data;
```

# 构建连接字符串

下一阶段将是建立连接字符串，所以在以下行下的一对大括号中，首先输入`string connString =`，然后跟上`@`符号使其成为逐字字符串，然后放入`""`符号。现在要获取连接字符串，执行以下操作：

1.  单击菜单栏中的“查看”，然后选择“SQL Server 对象资源管理器”。

1.  右键单击`People`数据库，然后选择属性。

1.  在属性窗格中，双击连接字符串以选择带有其长描述的内容。

1.  然后右键单击长描述并复制它。

1.  将描述粘贴在一对`""`符号之间。

然后连接字符串行应如下所示：

```cs
string connString = @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=People;Integrated Security=True;Connect Timeout=30;Encrypt=False;TrustServerCertificate=True;ApplicationIntent=ReadWrite;MultiSubnetFailover=False";
```

现在可以关闭 SQL Server 对象资源管理器和属性窗格。

# 建立 SQL 连接

在下一阶段，我们将像往常一样进行。所以，输入以下行：

```cs
using (SqlConnection conn = new SqlConnection(connString))
```

注意，当你输入这个时，你会看到文件顶部的`using System.Data.SqlClient;`行中的`SqlClient`变为活动状态。它变成了黑色。这意味着 SQL 连接存储在那里，如果你将鼠标悬停在上面，它还会告诉你这一点：class System.Data.SqlClient.SqlConnection

在下一阶段，在此行下的一对大括号之间输入以下内容：

```cs
SqlCommand command = new SqlCommand("select * from dbo.ExperimentValues", conn);
```

在`SqlCommand()`后面的括号之间，将定义命令的文本直接放入构造函数作为参数。记住，你已经有了`ExperimentValues`。`*`符号表示选择所有列。所以，你需要命令文本和连接。

# 制作适配器

接下来，你将创建一个适配器。所以，输入以下内容：

```cs
SqlDataAdapter adapter = new SqlDataAdapter(command);
```

在这里，`SqlDataAdapter`是存在于实际数据库和我们之间的东西。这是一种将信息从这里适应到那里的方式。要初始化它，可以传入特定的 SQL 命令。因此，在我们的情况下，我们将传入`(command)`。您可以在此行后面添加注释`//make adapter`。

# 制作数据表

接下来，您将制作一个数据表，如下所示：

```cs
DataTable dt = new DataTable();
```

再次注意，一旦输入`DataTable`，文件顶部的`using System.Data;`命名空间就会激活。因此，如果将鼠标悬停在`DataTable`上，它会说 System.Data.DataTable 类。这就是它存储的地方。因此，它存储在`Data`命名空间中。

# 用数据填充表

现在我们需要用一些信息填充这个表。因此，接下来输入以下内容：

```cs
adapter.Fill(dt);
```

在这里，您输入适配器的名称，然后填充数据集。因此，通过这三行，首先制作一个适配器并使用 SQL 命令获取信息，然后制作一个数据表。然后使用适配器填充该表。现在我们可以如下使用它：

```cs
var summedXValues = dt.AsEnumerable().Sum(row => row.Field<decimal>(1));
```

在这里，我们可以将数据表变成可枚举的，以便我们可以浏览它。请注意，我们在其中使用`=>`添加了一个 Lambda 表达式；`<decimal>`是数据类型，然后，如果将鼠标悬停在`<decimal>()`括号后面，工具提示会说(`DataColumn column`): 提供对指定行中每个列值的强类型访问。因此，在括号之间插入 1。

接下来，为`summedYValues`变量输入以下内容，并注意我们在括号之间放了一个 2：

```cs
var summedYValues = dt.AsEnumerable().Sum(row => row.Field<decimal>(2));
```

一旦您输入了所有这些，然后您可以显示`x`和`y`值的总和，因此接下来输入以下行：

```cs
sampLabel.Text = $"Sum of y values={summedYValues}";
sampLabel.Text += $"<br>Sum of x values={summedXValues}";
```

# 显示总和值

在前面的行中，请注意第一行不需要`<br>`标签，但下一行需要。此外，第一行只需要`=`, 而下一行需要`+=`来追加。

# 运行程序

请记住，目的是对字段求和，因此打开浏览器，然后单击 Sum Fields 按钮：

![](img/3259b843-4776-4386-a4ca-53cb1b6f2276.png)

图 21.6.4：运行我们程序的初始结果

您可以看到 Y 值的总和为 50.000，X 值的总和为 10.000。您可以通过打开 SQL Server 对象资源管理器窗格，右键单击`ExperimentValues`表，并将值相加来确认这是否按预期工作，如*图 21.6.5*所示：

![](img/2a699f26-0334-4448-ba0e-7081be08fe93.png)

图 21.6.5：添加 X 和 Y 列中的值

XValues 列加起来是 10.000，YValues 列加起来是 50.000。这两个总和与程序运行的结果一致。

关闭`ExperimentValues`表窗口和 SQL 对象资源管理器窗格。这次又按预期工作了。

# 添加注释

现在在连接字符串行上面添加此注释：

```cs
//make connection string
```

每当处理低级资源时，应用`using`块。在以`using (SqlConnection conn...`开头的行上面添加以下评论：

```cs
//make connection object
```

请记住，目的是正确地制作、使用和处理它，以便不会留下任何内存泄漏。每当处理硬盘访问时，例如，都要这样做。

在以`SqlCommand command =...`开头的行上面添加以下评论：

```cs
//make SQL command
```

然后，在以`sqlDataAdapter adapter...`开头的行上面添加以下评论以强调适配器的目的：

```cs
//make adapter object and pass in the command
```

同时，在该行的末尾添加此评论：

```cs
//make adapter
```

接下来，对于`DataTable dt...`，添加此注释：

```cs
//make table
```

适配器是允许我们填充表的机制，因此在`adapter.Fill(dt);`行的末尾添加以下评论：

```cs
//fill table with adapter
```

接下来，在第 30 行上面添加以下评论：

```cs
//lines 30 - 31 use LINQ to sum each column
```

最后，在第 33 行上面添加以下评论：

```cs
//lines 33-34 display the results in the web page
```

在下一行中，请注意这里的字段是`decimal`，因为这是我们在 SQL Server 中创建的，1 只是表示第一个字段，索引为 1。然而，请记住，这实际上意味着第二列，因为有三列：

```cs
var summedXValues = dt.AsEnumerable().Sum(row => row.Field<decimal>(1));
```

如*图 21.6.6*所示，Id 实际上是索引 0，XValues 是索引 1，YValues 是索引 2。这就是为什么我们在这里使用 1 和 2，因为有三列，其中第二列位于索引 1：

![](img/a74c7f3f-19ba-46a5-a6e0-6e4b887e35c0.png)

图 21.6.6：Id 是索引 0，XValues 是索引 1，YValues 是索引 2

# 章节回顾

本章的`Default.aspx.cs`文件的完整版本，包括注释，如下面的代码块所示。

```cs
//using is a directive
//System is a name space
//name space is a collection of features that our needs to run
using System;
using System.Data.SqlClient;
using System.Linq;
using System.Data;
//public means accessible anywhere
//partial means this class is split over multiple files
//class is a keyword and think of it as the outermost level of grouping
//:System.Web.UI.Page means our page inherits the features of a Page
public partial class _Default : System.Web.UI.Page
{
    protected void Button1_Click(object sender, EventArgs e)
    {
        //make connection string
        string connString = @"Data Source=DESKTOP-4L6NSGO\SQLEXPRESS;Initial Catalog=People;Integrated Security=True;Connect Timeout=15;Encrypt=False;TrustServerCertificate=False;ApplicationIntent=ReadWrite;MultiSubnetFailover=False";
        //make connection object
        using (SqlConnection conn = new SqlConnection(connString))
        {
            //make sql command
            SqlCommand command = new SqlCommand("select * from dbo.ExperimentValues", conn);
            //make adapter object and pass in the command
            //make adapter
            SqlDataAdapter adapter = new SqlDataAdapter(command);
            //make table
            DataTable dt = new DataTable();
            adapter.Fill(dt); //fill table with adapter
            //lines 30 - 31 use linq to sum each column
            var summedXValues = dt.AsEnumerable().Sum(row => row.Field< 
            decimal>(1));
            var summedYValues = dt.AsEnumerable().Sum(row => row.Field< 
            decimal>(2));
            //lines 33-34 display the results in the web page
            sampLabel.Text = $"Sum of y values={summedYValues}";
            sampLabel.Text += $"<br>Sum of x values={summedXValues}";
        }
    }
} 
```

# 总结

在本章中，您学习了如何将 LINQ 与 SQL 和 SQL Server 一起使用。您更改了`ExperimentValues`表中的数据，编写了使用 LINQ 对字段进行汇总的代码，添加了命名空间，构建了连接字符串，建立了 SQL 连接，创建了适配器，创建了数据表，填充了数据表，显示了汇总值，运行了程序，并最终添加了注释。

在下一章中，您将学习如何制作一个页面，将页面上的内容保存到硬盘上，然后再读取它。
