# 第二十八章：使用 C#创建 XML 文件

在本章中，您将学习如何将 XML 写入文件，然后在记事本和 Internet Explorer 中查看结果。

# 向 HTML 添加按钮

启动一个项目，并在 HTML 页面中放置一个按钮。要做到这一点，转到视图|工具箱（*Ctrl* + *Alt*-*X*），在搜索字段中输入`but`，并将`Button`控件拖放到以`<form id=...`开头的行下面。更改按钮上的文本为`保存文件`。

接下来，转到设计视图。双击保存文件按钮。这会打开`Default.aspx.cs`中的事件处理程序。删除`Page_Load`存根。折叠`using System;`上下的所有注释—你不需要它们。该项目起始代码的相关部分应该看起来像*图 28.2.1*中的那样：

![](img/e5d29865-ef41-4272-8a72-ba42846e7d38.png)

图 28.2.1：该项目的起始代码

# 添加命名空间

首先，让我们添加一些命名空间。在文件顶部附近的`using System`后面输入以下内容：

```cs
using System.Xml;
using System.Diagnostics;
```

您需要`using System.Diagnostics;`，这样您就可以在创建文件后立即在 Internet Explorer 和记事本中查看文件。

# 编码`XmlWriter`设置

接下来，您将设置`XmlWriter`设置。因此，在以`protected void Button1_Click...`开头的行下面的大括号之间输入以下内容：

```cs
XmlWriterSettings settings = new XmlWriterSettings();
```

在这一行中，您创建了该类的设置对象，然后设置了功能。接下来输入以下内容：

```cs
settings.Indent = true;
```

在此行下面输入以下内容：

```cs
settings.IndentChars = "\t";
```

在这里，`"\t"`是一个制表符。

# 写入硬盘

现在，因为`XmlWriter`类使用硬盘等，您需要将其包含在`using`语句中。因此，接下来输入以下内容：

```cs
using (XmlWriter writer = XmlWriter.Create(@"c:\data\sampfile2.xml", settings))
```

您将在硬盘上创建一个文件，`c:\data\sampfile2.xml`，然后将设置传递给要使用的设置。设置对象作为参数传递给`XmlWriter`内定义的`Create`函数。

在下一阶段，我们将实际写入，因此在大括号之间输入以下内容：

```cs
writer.WriteStartElement("bookstore");
writer.WriteEndElement();
```

在第二行，您立即关闭`WriteStartElement`方法。我们在这里添加一个结构。

现在，您将在这两行之间添加几行代码。首先编写一个属性字符串，如下所示：

```cs
writer.WriteAttributeString("name", "Tom's Book Store");
```

接下来，您将创建另一个元素。在这里，如果您缩进代码，将有所帮助，这表明`book`元素位于`bookstore`元素下面。为此，输入以下内容：

```cs
writer.WriteStartElement("book");
```

要写入的元素是`book`。接下来输入以下内容：

```cs
writer.WriteStartElement("bookauthor");
```

现在让我们做以下操作来关闭这个：

```cs
writer.WriteEndElement();
```

您这样做是为了保持结束和开始成对。

现在，在此处（在`WriteEndElement`行上方），您可以写入另一个元素。在这一行中，您将包括特定的书籍作者。同样，您将写入一个字符串，作者的名字将是值。输入以下内容：

```cs
writer.WriteString("John Smith");
```

在这里，要注意`WriteAttribute`与`WriteString`是不同的。`WriteString`在标签之间，而`WriteAttribute`给出属性，因此是不同的。这对我们的目的已经足够了。

# 格式化结果

现在，您希望确保结果看起来不错。因此，在最后一个`WriteEndElement`行下面的闭合大括号外面，输入以下内容：

```cs
Process.Start("notepad.exe", @"c:\data\sampfile2.xml");
```

您将在记事本中查看结果，然后需要文件的路径，因此从前面的`using`行中复制，`c:\data\sampfile2.xml`，并粘贴到此行中。

现在让我们再做一个。基本上，只需重复此行，并将其中的`notepad.exe`更改为`iexplore.exe`，如下所示，以指示接下来应该使用 Internet Explorer：

```cs
Process.Start("iexplore.exe", @"c:\data\sampfile2.xml");
```

# 运行程序

现在让我们在浏览器中打开并查看结果。单击保存文件按钮，您将看到在 Internet Explorer 中的样子：

![](img/19885c8c-225f-4153-a138-9a8bac1776bd.png)

图 28.2.2：在 Internet Explorer 中运行程序的结果

您可以看到它有结构，结果甚至是可折叠的，如 XML 标签之前的-符号所示，当然也是可展开的。书店的名称是汤姆的书店，这是属性，然后是约翰·史密斯，这是作为字符串写在书的作者标签或元素之间。

同样，在记事本中，它看起来像*图 28.2.3*中显示的屏幕，格式正确的 XML：

![](img/d78866d8-e93f-474f-9bc2-a3f26a347b5a.png)

图 28.2.3：在记事本中运行程序的结果

所以，这些就是进行这些操作的基础知识。

# 章节回顾

本章的`Default.aspx.cs`文件的完整版本，包括注释，如下所示：

```cs
//using is a directive
//System is a name space
//name space is a collection of features that our needs to run
using System;
using System.Xml;
using System.Diagnostics;
//public means accessible anywhere
//partial means this class is split over multiple files
//class is a keyword and think of it as the outermost level of grouping
//:System.Web.UI.Page means our page inherits the features of a Page
public partial class _Default : System.Web.UI.Page
{
    protected void Button1_Click(object sender, EventArgs e)
    {
        //make a setting object
        XmlWriterSettings settings = new XmlWriterSettings();
        //set indent to true
        settings.Indent = true;
        //use tabs for indenting
        settings.IndentChars = "\t";
        //create file to write to
        using (XmlWriter writer = 
        XmlWriter.Create(@"c:\data\sampfile2.xml", settings))
        {
            //outermost element
            writer.WriteStartElement("bookstore");
            //attribute of book store
            writer.WriteAttributeString("name", "Tom's Book Store");
                //new element called book
                writer.WriteStartElement("book");
                    //new element called author
                    writer.WriteStartElement("bookauthor");
                    //this goes between the author tags
                    writer.WriteString("John Smith");
                writer.WriteEndElement();
            writer.WriteEndElement();
        }
        //priview the files in notepad and internet explorer
        Process.Start("notepad.exe", @"c:\data\sampfile2.xml");
        Process.Start("iexplore.exe", @"c:\data\sampfile2.xml");
    }
}
```

# 总结

在本章中，您学会了如何将 XML 写入文件，然后在记事本和 Internet Explorer 中查看结果。您编写了`XmlWriter`设置，并编写了将结果写入硬盘并格式化结果的代码。

在下一章中，您将学习如何将 LINQ 和 XML 结合起来，使其更加实用。
