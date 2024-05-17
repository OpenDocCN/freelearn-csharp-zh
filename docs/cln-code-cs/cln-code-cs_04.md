编写干净的函数

干净的函数是小方法（它们有两个或更少的参数）并且避免重复。理想的方法没有参数，也不修改程序的状态。小方法不太容易出现异常，因此你将编写更加健壮的代码，从长远来看，你将有更少的错误需要修复。

函数式编程是一种将计算视为数学计算的软件编码方法。本章将教你将计算视为数学函数的评估的好处，以避免改变对象的状态。

大方法（也称为函数）阅读起来笨拙且容易出错，因此编写小方法有其优势。因此，我们将看看如何将大方法分解为小方法。在本章中，我们将介绍 C#中的函数式编程以及如何编写小而干净的方法。

构造函数和具有多个参数的方法可能会变得非常麻烦，因此我们需要寻找解决方法来处理和传递多个参数，以及如何避免使用超过两个参数。减少参数数量的主要原因是它们可能变得难以阅读，会让其他程序员感到烦恼，并且如果参数足够多的话会造成视觉压力。它们也可能表明该方法试图做太多的事情，或者你需要考虑重构你的代码。

在本章中，我们将涵盖以下主题：

+   理解函数式编程

+   保持方法小

+   避免重复

+   避免多个参数

通过本章的学习，你将具备以下技能：

+   描述函数式编程是什么

+   在 C#编程语言中提供现有的函数式编程示例

+   编写函数式的 C#代码

+   避免编写超过两个参数的方法

+   编写不可变的数据对象和结构

+   保持你的方法小

+   编写符合单一职责原则（SRP）的代码

让我们开始吧！

# 第五章：理解函数式编程

函数式编程与其他编程方法的唯一区别在于函数不修改数据或状态。在深度学习、机器学习和人工智能等场景中，当需要对相同的数据集执行不同的操作时，你将使用函数式编程。

.NET Framework 中的 LINQ 语法是函数式编程的一个例子。因此，如果你想知道函数式编程是什么样子，如果你以前使用过 LINQ，那么你已经接触过函数式编程，并且应该知道它是什么样子的。

由于函数式编程是一个深入的主题，关于这个主题存在许多书籍、课程和视频，所以我们在本章中只会简要涉及这个主题，通过查看纯函数和不可变数据。

纯函数只能对传入的数据进行操作。因此，该方法是可预测的，避免产生副作用。这对程序员有好处，因为这样的方法更容易推理和测试。

一旦初始化了一个不可变的数据对象或数据结构，其中包含的数据值将不会被修改。因为数据只是被设置而不是修改，你可以很容易地推断出数据是什么，它是如何设置的，以及任何操作的结果会是什么，给定了输入。不可变数据也更容易测试，因为你知道你的输入是什么，以及期望的输出是什么。这使得编写测试用例变得更容易，因为你不需要考虑那么多事情，比如对象状态。不可变对象和结构的好处在于它们是线程安全的。线程安全的对象和结构可以作为良好的数据传输对象（DTOs）在线程之间传递。

但是如果结构包含引用类型，它们仍然可以是可变的。解决这个问题的一种方法是使引用类型成为不可变的。C# 7.2 增加了对`readonly struct`和`ImmutableStruct`的支持。因此，即使我们的结构包含引用类型，我们现在也可以使用这些新的 C# 7.2 构造来使具有引用类型的结构成为不可变的。

现在，让我们来看一个纯函数的例子。对象属性的唯一设置方式是通过构造函数在构造时进行。这个类是一个`Player`类，其唯一工作是保存玩家的姓名和他们的最高分。提供了一个方法来更新玩家的最高分：

```cs
public class Player
{
    public string PlayerName { get; }
    public long HighScore { get; }

    public Player(string playerName, long highScore)
    {
        PlayerName = playerName;
        HighScore = highScore;
    }

    Public Player UpdateHighScore(long highScore)
    {
        return new Player(PlayerName, highScore);
    }

}
```

请注意，`UpdateHighScore`方法不会更新`HighScore`属性。相反，它通过传入已在类中设置的`PlayerName`变量和方法参数`highScore`来实例化并返回一个新的`Player`类。您现在已经看到了一个非常简单的示例，说明如何在不改变其状态的情况下编写软件。

函数式编程是一个非常庞大的主题，对于过程式和面向对象的程序员来说，它需要进行思维转变，这可能非常困难。由于这超出了本书的范围（深入探讨函数式编程的主题），我们鼓励您自行查阅 PacktPub 提供的函数式编程资源。

Packt 有一些非常好的书籍和视频，专门教授功能编程的顶级知识。您将在本章末尾的*进一步阅读*部分找到一些 Packt 功能编程资源的链接。

在我们继续之前，我们将看一些 LINQ 示例，因为 LINQ 是 C#中函数式编程的一个例子。有一个例子数据集会很有帮助。以下代码构建了一个供应商和产品列表。我们将首先编写`Product`结构：

```cs
public struct Product
{
    public string Vendor { get; }
    public string ProductName { get; }
    public Product(string vendor, string productName)
    {
        Vendor = vendor;
        ProductName = productName;
    }
}
```

现在我们有了结构体，我们将在`GetProducts()`方法中添加一些示例数据：

```cs
public static List<Product> GetProducts()
{
    return new List<Products>
    {
        new Product("Microsoft", "Microsoft Office"),
        new Product("Oracle", "Oracle Database"),
        new Product("IBM", "IBM DB2 Express"),
        new Product("IBM", "IBM DB2 Express"),
        new Product("Microsoft", "SQL Server 2017 Express"),
        new Product("Microsoft", "Visual Studio 2019 Community Edition"),
        new Product("Oracle", "Oracle JDeveloper"),
        new Product("Microsoft", "Azure"),
        new Product("Microsoft", "Azure"),
        new Product("Microsoft", "Azure Stack"),
        new Product("Google", "Google Cloud Platform"),
        new Product("Amazon", "Amazon Web Services")
    };
}
```

最后，我们可以开始在我们的列表上使用 LINQ。在前面的示例中，我们将获得一个按供应商名称排序的产品的不同列表，并打印出结果：

```cs
class Program
{
    static void Main(string[] args)
    {
        var vendors = (from p in GetProducts()
                        select p.Vendor)
                        .Distinct()
                        .OrderBy(x => x);
        foreach(var vendor in vendors)
            Console.WriteLine(vendor);
        Console.ReadKey();
    }
}
```

在这里，我们通过调用`GetProducts()`获取供应商列表，并仅选择`Vendor`列。然后，我们过滤列表，使其只包括一个供应商，通过调用`Distinct()`方法。然后，通过调用`OrderBy(x => x)`按字母顺序对供应商列表进行排序，其中`x`是供应商的名称。在获得排序后的不同供应商列表后，我们遍历列表并打印供应商的名称。最后，我们等待用户按任意键退出程序。

函数式编程的一个好处是，您的方法比其他类型的编程方法要小得多。接下来，我们将看一下为什么保持方法小巧是有益的，以及我们可以使用的技术，包括函数式编程。

# 保持方法的小巧

在编写干净和可读的代码时，保持方法小巧是很重要的。在 C#世界中，最好将方法保持在*10 行以下*。最佳长度不超过*4 行*。保持方法小巧的一个好方法是考虑是否应该捕获错误或将其传递到调用堆栈的更高层。通过防御性编程，您可能会变得过于防御，这可能会增加您发现自己编写的代码量。此外，捕获错误的方法将比不捕获错误的方法更长。

让我们考虑以下可能会抛出`ArgumentNullException`的代码：

```cs
        public UpdateView(MyEntities context, DataItem dataItem)
        {
            InitializeComponent();
            try
            {
                DataContext = this;
                _dataItem = dataItem;
                _context = context;
                nameTextBox.Text = _dataItem.Name;
                DescriptionTextBox.Text = _dataItem.Description;
            }
            catch (Exception ex)
            {
                Debug.WriteLine(ex);
                throw;
            }
        }
```

在上面的代码中，我们可以清楚地看到有两个位置可能会引发`ArgumentNullException`。可能引发`ArgumentNullException`的第一行代码是`nameTextBox.Text = _dataItem.Name;`；可能引发相同异常的第二行代码是`DescriptionTextBox.Text = _dataItem.Description;`。我们可以看到异常处理程序在发生异常时捕获异常，将其写入控制台，然后简单地将其抛回堆栈。

请注意，从人类阅读的角度来看，有*8 行*代码形成了`try/catch`块。

你可以通过编写自己的参数验证器，用一行文本完全替换`try/catch`异常处理。为了解释这一点，我们将提供一个例子。

让我们首先看一下`ArgumentValidator`类。这个类的目的是抛出一个带有包含空参数的方法名称的`ArgumentNullException`：

```cs
using System;
namespace CH04.Validators
{
    internal static class ArgumentValidator
    {
        public static void NotNull(
            string name, 
            [ValidatedNotNull] object value
        )
        {
            if (value == null)
                throw new ArgumentNullException(name);
        }
    }

    [AttributeUsage(
        AttributeTargets.All, 
        Inherited = false, 
        AllowMultiple = true)
    ]
    internal sealed class ValidatedNotNullAttribute : Attribute
    {
    }
}
```

现在我们有了我们的空验证类，我们可以对我们的方法中的空值参数执行新的验证方式。所以，让我们看一个简单的例子：

```cs
public ItemsUpdateView(
    Entities context, 
    ItemsView itemView
)
{
    InitializeComponent();
    ArgumentValidator.NotNull("ItemsUpdateView", itemView);
    // ### implementation omitted ###
}
```

正如你可以清楚地看到的，我们用一个一行代码替换了整个`try catch`块。当这个验证检测到空参数时，会抛出一个`ArgumentNullException`，阻止代码继续执行。这使得代码更容易阅读，也有助于调试。

现在，我们将看一下如何使用缩进格式化函数，使其易于阅读。

## 缩进代码

一个非常长的方法在任何时候都很难阅读和跟踪，特别是当你不得不多次滚动方法才能到达底部时。但是，如果方法没有正确格式化并且缩进级别不正确，那么这将是一个真正的噩梦。

如果你遇到任何格式不良的方法代码，那么作为专业程序员，在你做任何其他事情之前，要把代码整理好是你自己的责任。大括号之间的任何代码被称为**代码块**。代码块内的代码应该缩进一级。代码块内的代码块也应该缩进一级，如下面的例子所示：

```cs
public Student Find(List<Student> list, int id) 
{          
Student r = null;foreach (var i in list)          
{             
if (i.Id == id)                   
    r = i;          }          return r;     
}
```

上面的例子展示了糟糕的缩进和糟糕的循环编程。在这里，你可以看到正在搜索学生列表，以便找到并返回具有指定 ID 的学生，该 ID 作为参数传递。一些程序员感到恼火并降低了应用程序的性能，因为在上面的代码中，即使找到了学生，循环仍在继续。我们可以改进上面的代码的缩进和性能如下：

```cs
public Student Find(List<Student> list, int id) 
{          
    Student r = null;
    foreach (var i in list)          
    {             
        if (i.Id == id)                  
        {
            r = i; 
            break;         
        }      
    }
    return r;         
}
```

在上面的代码中，我们改进了格式，并确保代码正确缩进。我们在`for`循环中添加了`break`，以便在找到匹配项时终止`foreach`循环。

现在不仅代码更易读，而且性能也更好。想象一下，代码正在针对一个校园有 73,000 名学生的大学以及远程学习进行运行。考虑一下，如果学生与 ID 匹配是列表中的第一个，那么如果没有`break`语句，代码将不得不运行 72,999 次不必要的计算。你可以看到`break`语句对上面的代码性能有多大的影响。

我们将返回值保留在原始位置，因为编译器可能会抱怨并非所有代码路径都返回一个值。这也是我们添加`break`语句的原因。很明显，正确的缩进提高了代码的可读性，从而帮助程序员理解代码。这使程序员能够进行任何他们认为必要的更改。

# 避免重复

代码可以是**DRY**或**WET**。WET 代码代表**每次写**，是 DRY 的相反，DRY 代表**不要重复自己**。WET 代码的问题在于它是*bug*的完美候选者。假设您的测试团队或客户发现了一个 bug 并向您报告。您修复了 bug 并传递了它，但它会在您的计算机程序中遇到该代码的次数一样多次回来咬您。

现在，我们通过消除重复来 DRY 我们的 WET 代码。我们可以通过提取代码并将其放入方法中，然后以一种可访问所有需要它的计算机程序区域的方式将方法集中起来。

举个例子。假设您有一个费用项目集合，其中包含`Name`和`Amount`属性。现在，考虑通过`Name`获取费用项目的十进制`Amount`。

假设您需要这样做 100 次。为此，您可以编写以下代码：

```cs
var amount = ViewModel
    .ExpenseLines
    .Where(e => e.Name.Equals("Life Insurance"))
    .FirstOrDefault()
    .Amount;
```

没有理由您不能写相同的代码 100 次。但有一种方法可以只写一次，从而减少代码库的大小并提高您的生产力。让我们看看我们可以如何做到这一点：

```cs
public decimal GetValueByName(string name)
{
    return ViewModel
        .ExpenseLines
        .Where(e => e.Name.Equals(name))
        .FirstOrDefault()
        .Amount;
}
```

要从`ViewModel`中的`ExpenseLines`集合中提取所需的值，您只需将所需值的名称传递给`GetValueName(string name)`方法，如下面的代码所示：

```cs
var amount = GetValueByName("Life Insurance");
```

那一行代码非常易读，获取值的代码行包含在一个方法中。因此，如果出于任何原因（例如修复 bug）需要更改方法，您只需在一个地方修改代码。

编写良好的函数的下一个逻辑步骤是尽可能少地使用参数。在下一节中，我们将看看为什么我们不应该超过两个参数，以及如何处理参数，即使我们需要更多。

# 避免多参数

Niladic 方法是 C#中理想的方法类型。这种方法没有参数（也称为*参数*）。Monadic 方法只有一个参数。Dyadic 方法有两个参数。Triadic 方法有三个参数。具有三个以上参数的方法称为多参数方法。您应该尽量保持参数数量最少（最好少于三个）。

在 C#编程的理想世界中，您应尽力避免三参数和多参数方法。这不是因为它是糟糕的编程，而是因为它使您的代码更易于阅读和理解。具有大量参数的方法可能会给程序员带来视觉压力，并且也可能成为烦恼的根源。随着添加更多参数，IntelliSense 也可能变得难以阅读和理解。

让我们看一个更新用户帐户信息的多参数方法的不良示例：

```cs
public void UpdateUserInfo(int id, string username, string firstName, string lastName, string addressLine1, string addressLine2, string addressLine3, string addressLine3, string addressLine4, string city, string postcode, string region, string country, string homePhone, string workPhone, string mobilePhone, string personalEmail, string workEmail, string notes) 
{
    // ### implementation omitted ###
}
```

如`UpdateUserInfo`方法所示，代码难以阅读。我们如何修改该方法，使其从多参数方法转变为单参数方法？答案很简单 - 我们传入一个`UserInfo`对象。首先，在修改方法之前，让我们看一下我们的`UserInfo`类：

```cs
public class UserInfo
{
    public int Id { get;set; }
    public string Username { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string AddressLine1 { get; set; }
    public string AddressLine2 { get; set; }
    public string AddressLine3 { get; set; }
    public string AddressLine4 { get; set; }
    public string City { get; set; }
    public string Region { get; set; }
    public string Country { get; set; }
    public string HomePhone { get; set; }
    public string WorkPhone { get; set; }
    public string MobilePhone { get; set; }
    public string PersonalEmail { get; set; }
    public string WorkEmail { get; set; }
    public string Notes { get; set; }
}
```

现在我们有一个包含所有需要传递给`UpdateUserInfo`方法的信息的类。`UpdateUserInfo`方法现在可以从多参数方法转变为单参数方法，如下所示：

```cs
public void UpdateUserInfo(UserInfo userInfo)
{
    // ### implementation omitted ###
}
```

前面的代码看起来好多了吗？它更小，更易读。经验法则应该是少于三个参数，理想情况下是零。如果您的类遵守 SRP，则考虑实现*参数对象模式*，就像我们在这里所做的那样。

## 实施 SRP

您编写的所有对象和方法应该最多只有一个职责，而不再有其他。对象可以有多个方法，但这些方法在组合时应该都朝着它们所属的对象的单一目的工作。方法可以调用多个方法，每个方法都做不同的事情。但方法本身应该只做一件事。

一个了解和做得太多的方法被称为**上帝方法**。同样，一个了解和做得太多的对象被称为**上帝对象**。上帝对象和方法很难阅读、维护和调试。这样的对象和方法通常会多次重复相同的错误。擅长编程技艺的人会避免上帝对象和上帝方法。让我们看一个做了不止一件事的方法：

```cs
public void SrpBrokenMethod(string folder, string filename, string text, emailFrom, password, emailTo, subject, message, mediaType)
{
    var file = $"{folder}{filename}";
    File.WriteAllText(file, text);
    MailMessage message = new MailMessage();  
    SmtpClient smtp = new SmtpClient();  
    message.From = new MailAddress(emailFrom);  
    message.To.Add(new MailAddress(emailTo));  
    message.Subject = subject;  
    message.IsBodyHtml = true;  
    message.Body = message;  
    Attachment emailAttachment = new Attachment(file); 
    emailAttachment.ContentDisposition.Inline = false; 
    emailAttachment.ContentDisposition.DispositionType =        
        DispositionTypeNames.Attachment; 
    emailAttachment.ContentType.MediaType = mediaType;  
    emailAttachment.ContentType.Name = Path.GetFileName(filename); 
    message.Attachments.Add(emailAttachment);
    smtp.Port = 587;  
    smtp.Host = "smtp.gmail.com";
    smtp.EnableSsl = true;  
    smtp.UseDefaultCredentials = false;  
    smtp.Credentials = new NetworkCredential(emailFrom, password);  
    smtp.DeliveryMethod = SmtpDeliveryMethod.Network;  
    smtp.Send(message);
}
```

`SrpBrokenMethod`显然做了不止一件事，因此它违反了 SRP。我们现在将这个方法分解为多个只做一件事的较小方法。我们还将解决该方法的多参数性质的问题。

在我们开始将方法分解为只做一件事的较小方法之前，我们需要查看方法执行的所有操作。该方法首先将文本写入文件。然后创建电子邮件消息，分配附件，最后发送电子邮件。因此，我们需要以下方法：

+   将文本写入文件

+   创建电子邮件消息

+   添加电子邮件附件

+   发送电子邮件

查看当前方法，我们有四个参数传递给它来写入文本到文件：一个用于文件夹，一个用于文件名，一个用于文本，一个用于媒体类型。文件夹和文件名可以合并为一个名为`filename`的单个参数。如果`filename`和`folder`是在调用代码中分开使用的两个变量，则可以将它们作为单个插值字符串传递到方法中，例如`$"{folder}{filename}"`。

至于媒体类型，这可以在构造时私下设置在一个结构体内。我们可以使用该结构体来设置我们需要的属性，以便我们可以将该结构体作为单个参数传递进去。让我们看一下实现这一点的代码：

```cs
    public struct TextFileData
    {
        public string FileName { get; private set; }
        public string Text { get; private set; }
        public MimeType MimeType { get; }        

        public TextFileData(string filename, string text)
        {
            Text = text;
            MimeType = MimeType.TextPlain;
            FileName = $"{filename}-{GetFileTimestamp()}";
        }

        public void SaveTextFile()
        {
            File.WriteAllText(FileName, Text);
        }

        private static string GetFileTimestamp()
        {
            var year = DateTime.Now.Year;
            var month = DateTime.Now.Month;
            var day = DateTime.Now.Day;
            var hour = DateTime.Now.Hour;
            var minutes = DateTime.Now.Minute;
            var seconds = DateTime.Now.Second;
            var milliseconds = DateTime.Now.Millisecond;
            return $"{year}{month}{day}@{hour}{minutes}{seconds}{milliseconds}";
        }
    }
```

`TextFileData`构造函数通过调用`GetFileTimestamp()`方法并将其附加到`FileName`的末尾来确保`FileName`的值是唯一的。要保存文本文件，我们调用`SaveTextFile()`方法。请注意，`MimeType`在内部设置为`MimeType.TextPlain`。我们本可以简单地将`MimeType`硬编码为`MimeType = "text/plain";`，但使用`enum`的优势在于代码是可重用的，而且您不必记住特定`MimeType`的文本或在互联网上查找它的好处。现在，我们将编写`enum`并为`enum`值添加描述：

```cs
[Flags]
public enum MimeType
{
    [Description("text/plain")]
    TextPlain
}
```

好吧，我们有了我们的`enum`，但现在我们需要一种方法来提取描述，以便可以轻松地分配给一个变量。因此，我们将创建一个扩展类，它将使我们能够获取`enum`的描述。这使我们能够设置`MimeType`，如下所示：

```cs
MimeType = MimeType.TextPlain;
```

没有扩展方法，`MimeType`的值将为`0`。但是通过扩展方法，`MimeType`的值为`"text/plain"`。现在您可以在其他项目中重用这个扩展，并根据需要构建它。

我们将编写的下一个类是`Smtp`类，其职责是通过`Smtp`协议发送电子邮件：

```cs
    public class Smtp
    {
        private readonly SmtpClient _smtp;

        public Smtp(Credential credential)
        {
            _smtp = new SmtpClient
            {
                Port = 587,
                Host = "smtp.gmail.com",
                EnableSsl = true,
                UseDefaultCredentials = false,
                Credentials = new NetworkCredential(
                 credential.EmailAddress, credential.Password),
                DeliveryMethod = SmtpDeliveryMethod.Network
            };
        }

        public void SendMessage(MailMessage mailMessage)
        {
            _smtp.Send(mailMessage);
        }
    }
```

`Smtp`类有一个构造函数，它接受一个`Credential`类型的参数。这个凭据用于登录到电子邮件服务器。服务器在构造函数中配置。当调用`SendMessage(MailMessage mailMessage)`方法时，消息被发送。

让我们编写一个`DemoWorker`类，将工作分成不同的方法：

```cs
    public class DemoWorker
    {
        TextFileData _textFileData;

        public void DoWork()        
        {
            SaveTextFile();
            SendEmail();
        }

        public void SendEmail()
        {
            Smtp smtp = new Smtp(new Credential("fakegmail@gmail.com", 
             "fakeP@55w0rd"));
            smtp.SendMessage(GetMailMessage());
        }

        private MailMessage GetMailMessage()
        {
            var msg = new MailMessage();
            msg.From = new MailAddress("fakegmail@gmail.com");
            msg.To.Add(new MailAddress("fakehotmail@hotmail.com"));
            msg.Subject = "Some subject";
            msg.IsBodyHtml = true;
            msg.Body = "Hello World!";
            msg.Attachments.Add(GetAttachment());
            return msg;
        }

        private Attachment GetAttachment()
        {
            var attachment = new Attachment(_textFileData.FileName);
            attachment.ContentDisposition.Inline = false;
            attachment.ContentDisposition.DispositionType = 
             DispositionTypeNames.Attachment;
            attachment.ContentType.MediaType = 
             MimeType.TextPlain.Description();
            attachment.ContentType.Name = 
             Path.GetFileName(_textFileData.FileName);
            return attachment;
        }

        private void SaveTextFile()
        {
            _textFileData = new TextFileData(
                $"{Environment.SpecialFolder.MyDocuments}attachment", 
                "Here is some demo text!"
            );
            _textFileData.SaveTextFile();
        }
    }
```

`DemoWorker`类展示了发送电子邮件消息的更清晰版本。负责保存附件并通过电子邮件作为附件发送的主要方法称为`DoWork()`。这个方法只包含两行代码。第一行调用`SaveTextFile()`方法，而第二行调用`SendEmail()`方法。

`SaveTextFile()`方法创建一个新的`TextFileData`结构，并传入文件名和一些文本。然后调用`TextFileData`结构中的`SaveTextFile()`方法，负责将文本保存到指定的文件中。

`SendEmail()`方法创建一个新的`Smtp`类。`Smtp`类有一个`Credential`参数，而`Credential`类有两个字符串参数用于电子邮件地址和密码。电子邮件和密码用于登录 SMTP 服务器。一旦 SMTP 服务器被创建，就会调用`SendMessage(MailMessage mailMessage)`方法。

这个方法需要传入一个`MailMessage`对象。因此，我们有一个名为`GetMailMethod()`的方法，它构建一个`MailMessage`对象，然后将其传递给`SendMessage(MailMessage mailMessage)`方法。`GetMailMethod()`通过调用`GetAttachment()`方法向`MailMessage`添加附件。

从这些修改中可以看出，我们的代码现在更加简洁和易读。这是良好质量的代码的关键，它必须易于阅读和理解。这就是为什么你的方法应该尽可能小而干净，参数尽可能少的原因。

你的方法是否违反了 SRP？如果是，你应该考虑将方法分解为尽可能多的方法来承担责任。这就结束了关于编写清晰函数的章节。现在是时候总结你所学到的知识并测试你的知识了。

# 总结

在本章中，您已经看到函数式编程如何通过不修改状态来提高代码的安全性，这可能会导致错误，特别是在多线程应用程序中。通过保持方法小而有意义的名称，以及不超过两个参数，您已经看到您的代码有多么清晰和易于阅读。您还看到了我们如何消除代码中的重复部分以及这样做的好处。易于阅读的代码比难以阅读和解释的代码更容易维护和扩展！

我们现在将继续并看一下异常处理的主题。在下一章中，您将学习如何适当地使用异常处理，编写自己的自定义 C#异常以提供有意义的信息，并编写避免引发`NullPointerExceptions`的代码。

# 问题

1.  你如何称呼一个没有参数的方法？

1.  你如何称呼一个有一个参数的方法？

1.  你如何称呼一个有两个参数的方法？

1.  你如何称呼一个有三个参数的方法？

1.  你如何称呼一个有超过三个参数的方法？

1.  应该避免哪两种方法类型，为什么？

1.  用通俗的语言来说，什么是函数式编程？

1.  函数式编程有哪些优点？

1.  函数式编程的一个缺点是什么？

1.  什么是 WET 代码，为什么应该避免？

1.  什么是 DRY 代码，为什么应该使用它？

1.  你如何去除 WET 代码中的重复部分？

1.  为什么方法应该尽可能小？

1.  如何在不实现`try/catch`块的情况下实现验证？

# 进一步阅读

以下是一些额外资源，让您可以深入了解 C#函数式编程的领域：

+   *Functional C#* by Wisnu Anggoro: [`www.packtpub.com/application-development/functional-c`](https://www.packtpub.com/application-development/functional-c)。这本书致力于 C#函数式编程，如果您想了解更多，这是一个很好的起点。

+   《C#中的函数式编程》由 Jovan Poppavic（微软）编写：[`www.codeproject.com/Articles/375166/Functional-programming-in-Csharp`](https://www.codeproject.com/Articles/375166/Functional-programming-in-Csharp)。这是一篇关于函数式 C#编程的深度文章。它包含了图表，并且有 5 星的评分。
