# 第二十七章：创建和使用 XML 文件

在本章中，我们将介绍 XML（可扩展标记语言）的基础知识。基本上，这是一种在互联网上结构化信息的方式。XML 的一个有用的方面是它是可扩展的，这意味着您可以创建自己的标签。

# 在 HTML 中添加按钮

启动一个项目。在`<html>`中唯一要放置的是一个`Button`控件。要做到这一点，转到工具箱，在搜索字段中输入`but`，然后将`Button`控件拖放到以`<form id=...`开头的行下面。将按钮上的文本更改为`Read XML`。

# 编写 XML

现在您需要一个可以阅读的文件。为此，转到解决方案资源管理器，右键单击网站的名称。在下拉菜单中选择`添加`，然后选择`添加新项...`。在搜索字段中输入`xml`，并确保选择 Visual C#中标有 XML 文件的 XML 文件。您的`XMLFile.xml`的起始屏幕应该如*图 27.1.1*所示：

![](img/79a637f2-e10c-4041-aa15-b4ffca6e7e8d.png)

图 27.1.1：XMLFile.xml 的起始屏幕

现在让我们逐行创建代码，这样您就可以看到到底发生了什么。基本上，就像在 HTML 中一样，XML 中有元素、元素的嵌套和属性。

首先，想象一下您有一家书店。在 XML 中，您可以创建自己的标签。因此，接下来输入以下内容：

```cs
<bookstore>
```

注意它自动创建了开放和关闭标签：`<bookstore>` `</bookstore>`。在这些标签之间插入几行空白。

当然，您的书店里有书，所以在第一个`<bookstore>`标签下面输入以下内容：

```cs
<book type="eBook">
```

一本书可能是传统的教科书，也可能是电子书。因此，我们将指定一个类型属性，并将其设置为我们第一本书的`eBook`。

现在让我们谈谈存储在`<book type="eBook">`下的一些元素。显然，一个基本的项目是书名，所以输入以下内容：

```cs
<booktitle>The Great Way</booktitle>
```

我们将这本书称为`The Great Way`。

在下一个阶段，自然地，您要输入作者，所以输入以下内容：

```cs
<author>Bob Jones</author>
```

因此，我们的书是由`Bob Jones`写的。

最后一项当然是价格，我们将说这个案例中是`$10.00`，所以输入以下内容：

```cs
<price>10.00</price>
```

这些信息提供了第一个书籍元素，正如您所看到的，它由称为`<booktitle>`、`<author>`和`<price>`的子元素组成。

现在让我们再做一本书，只是为了多样性，如下所示：

```cs
<book type="traditional">
    <booktitle>Happy People</booktitle>
    <author>Mary Jenkins</author>
    <price>11.00</price>
</book>
```

我们的简单 XML 文件如下代码块所示：

```cs
<?xml version="1.0" encoding="utf-8" ?>
<bookstore>
    <book type="eBook">
        <booktitle>The Great Way</booktitle>
        <author>Bob Jones</author>
        <price>10.00</price>
    </book>
    <book type="traditional">
        <booktitle>Happy People</booktitle>
        <author>Mary Jenkins</author>
        <price>11.00</price>
    </book>
</bookstore>
```

再次记住，XML 是*可扩展*的，因为您可以创建自己的标签，*标记*因为它具有类似 HTML 的结构，当然，它是一种*语言*。

现在，右键单击标有`XMLFile.xml`的选项卡，并从下拉菜单中选择`复制完整路径`。我们将很快使用这个路径。（如果您将鼠标悬停在`XMLFile.xml`选项卡上，可以看到完整路径，但它很长且难以记住，因此最好右键单击并选择`复制完整路径`。）

现在点击 HTML 中的`Default.aspx`选项卡，切换到设计视图，然后双击读取 XML 按钮。这会打开`Default.aspx.cs`中的事件处理代码。删除`Page_Load`存根。该项目的起始代码的相关部分应该如*图 27.1.2*所示：

![](img/75158be0-b1d7-4f5c-85f5-de95b55e1bf1.png)

图 27.1.2：该项目的起始代码

# 添加一个命名空间

让我们首先添加一个命名空间。您需要一个新的，所以在文件顶部附近的`using System`之后输入以下内容：

```cs
using System.Xml.Linq;
```

您将在编码中使用此命名空间。（您可以折叠`public partial class...`上面的所有代码。）

# 将 XML 文件加载到您的程序中

在下一个阶段，在以`protected void Button1_Click...`开头的行下面的一对大括号中输入以下内容：

```cs
XElement fromFile = XElement.Load(@"C:\Users\towsi\Documents\Visual Studio 2015\WebSites\CSharpTemplateUpdated76143\XMLFile.xml");
```

你想要加载`XElement fromFile`，所以你说`XElement.Load()`。然后，在括号内，你放置`@`符号使其成为原始字符串，然后是双引号。现在你需要利用从`XMLFile.xml`中复制的路径，这样你就可以从文件中加载 XML。所以，将路径粘贴在一对`""`符号之间。这将允许你加载可扩展标记文件。现在将鼠标悬停在`XElement`上。它说，类 System.Xml.Linq.XElement，表示 XML 元素。

# 遍历 XML 文件的内容

现在，输入以下内容：

```cs
foreach(XElement childElement in fromFile.Elements())
```

当你将鼠标悬停在这一行末尾的`Elements`上时，你会发现它是一个函数，返回的是 IEnumerable，所以你可以遍历它的内容，其中每个成员都是一个元素。

# 显示结果

现在你可以显示它们，所以在一对大括号之间输入以下内容：

首先，你需要书的类型。要获取它，在你输入`sampLabel.Text += $"<br>Book Type:`之后，你说`{childElement.Attribute("type")`，然后获取值，你输入`.Value}";`：

```cs
sampLabel.Text += $"<br>Book Type:{childElement.Attribute("type").Value}";
```

现在，要获取作者，你使用`{childElement.Element("author")}";`，如下所示：

```cs
sampLabel.Text += $"<br>{childElement.Element("author")}";
```

这就是你可以将所有元素取出来的方法。在这个阶段，你可以直接复制并粘贴这行代码，因为对于书名和书价来说基本上是一样的。

对于书名，你可以这样说：`{childElement.Element("booktitle")}";`，如下所示：

```cs
sampLabel.Text += $"<br>{childElement.Element("booktitle")}";
```

对于价格，你可以这样说：`{childElement.Element("price")}";`，如下所示：

```cs
sampLabel.Text += $"<br>{childElement.Element("price")}";
```

最后，为了分隔开，你可以使用`"<br><hr/>";`，如下所示：

```cs
sampLabel.Text += $"<br><hr/>";
```

# 运行程序

现在让我们在这里试一下，在浏览器中打开它。记住，你实际上是在将 XML 读入网页。这是我们的目标。点击“读取 XML”按钮。结果显示在图 27*.1.3*中：

![](img/8865cc92-d6fa-4ed2-896f-a27a8cda8e80.png)

图 27.1.3：运行程序的结果

信息被报告的方式与你输入的方式完全一样，这是你所期望的。请记住，水平线存在是因为你在 HTML 页面中输入了`"<br><hr/>"`，这添加了一个换行和一个水平规则或线。

这就是你可以将从 XML 文件中读取的内容与 C#结合起来，然后产生结果的方法。

# 章节回顾

本章的`Default.aspx.cs`文件的完整版本，包括注释，如下所示：

```cs
//using is a directive
//System is a name space
//name space is a collection of features that our needs to run
using System;
using System.Xml.Linq;//needed for XElement
//public means accessible anywhere
//partial means this class is split over multiple files
//class is a keyword and think of it as the outermost level of grouping
//:System.Web.UI.Page means our page inherits the features of a Page
public partial class _Default : System.Web.UI.Page
{
    protected void Button1_Click(object sender, EventArgs e)
    {
        //load XML file into "fromFile" variable
        XElement fromFile = XElement.Load(@"C:\Users\towsi\Documents\Visual Studio 2015\WebSites\CSharpTemplateUpdated76143\XMLFile.xml" );

        foreach(XElement childElement in fromFile.Elements())
        {
            //display value
            sampLabel.Text += $"<br>Book Type:{childElement.Attribute("type").Value}";
            //display author
            sampLabel.Text += $"<br>{childElement.Element("author")}";
            //display book title
            sampLabel.Text += $"<br>{childElement.Element("booktitle")}";
            //display price
            sampLabel.Text += $"<br>{childElement.Element("price")}";
            //adds horizontal rule across the page
            sampLabel.Text += $"<br><hr/>";
        }
    }
}
```

# 总结

在本章中，你学习了 XML 的基础知识。你编写了 XML 代码，将生成的 XML 文件加载到程序中，遍历了 XML 文件的内容，并编写了显示结果的代码。

在下一章中，你将学习如何将 XML 写入文件，然后在记事本和 Internet Explorer 中查看结果。因此，你将遇到许多有用的小技巧。
