# 第十章：使用反射在运行时查找、执行和创建类型

.NET 框架不仅包含代码，还包含元数据。元数据是关于程序中使用的程序集、类型、方法、属性等的描述性数据。这些程序集、属性、类型和方法是在 C# 编程语言内部定义的类。这些类、类型和方法在运行时被检索以解析开发者的应用程序逻辑以执行。属性允许我们在这些程序以及运行时可以使用的程序逻辑中添加额外的信息。

.NET 框架还允许开发者在开发期间定义此元数据信息。它可以在运行时通过反射读取。反射使我们能够创建检索到的类型的实例，并调用其方法和属性。

在本章中，我们将了解 .NET 框架如何允许我们读取和创建元数据，我们还将学习如何使用反射在运行时读取元数据并处理它。在 *属性* 部分，我们将专注于使用属性、创建自定义属性以及学习如何在运行时检索属性信息。*反射* 部分提供了如何使用反射创建类型、访问属性和调用方法的概述。反射还允许我们检索属性信息；例如，这可能是我们在执行应用程序逻辑时提供给 .NET 运行时的额外信息。

在本章中，我们将探讨以下主题：

+   属性

+   反射

# 技术要求

本章的练习可以使用 Visual Studio 2012 及更高版本以及 .NET Framework 2.0 及更高版本执行。然而，任何从 C# 7.0 及更高版本的新特性都需要您拥有 Visual Studio 2017。

如果你没有这些产品的任何许可证，你可以从 [`visualstudio.microsoft.com/downloads/`](https://visualstudio.microsoft.com/downloads/) 下载 Visual Studio 2017 的社区版。

本章的示例代码可以在 GitHub 上找到，地址为 [`github.com/PacktPublishing/Programming-in-C-sharp-Exam-70-483-MCSD-Guide/tree/master/Chapter10`](https://github.com/PacktPublishing/Programming-in-C-sharp-Exam-70-483-MCSD-Guide/tree/master/Chapter10)。

# 属性

可以使用属性将类型、方法和属性上的元数据或描述性信息关联起来。元数据指的是程序中定义的类型。例如，一个类是一个类型：每个类定义了某些属性和方法，每个属性属于一个类型，每个方法接受某些数据类型并返回某些数据类型。所有这些信息统称为元数据，可以在程序执行期间访问和检索。

就像任何其他方法一样，当你定义一个属性时，你也可以定义参数。你可以在汇编、类、方法或属性上定义一个或多个属性。根据程序要求，你可以定义应用程序需要的属性类型，并在程序中定义它们。一旦定义，你可以在程序执行时读取这些信息，然后进行处理。

在下一节中，我们将演示如何使用属性并按照我们的要求创建自定义属性。

# 使用属性

通过属性将信息关联到代码的一种声明性方法是。然而，只有少数属性可以用于每个类型。相反，它们用于特定类型。可以通过在要应用的类型上方使用方括号`[]`来指定任何类型的属性。

让我们看一下以下代码。通常，当我们想要将对象序列化为二进制或 XML 格式时，我们会看到`Serializable`属性。在实际应用中，当我们需要通过网络传输大型对象时，我们会将对象序列化为上述格式之一，然后发送。类上的序列化属性使运行时能够允许将对象转换为二进制或 XML 或程序所需的任何格式：

```cs
[Serializable]
public class AttributeTest
{
//object of this class can now be serialized
}
```

属性的另一种常见用途是在单元测试项目中。观察以下代码：

```cs
namespace Chapter10.Test
{
    [TestClass]
    public class UnitTest1
    {
        [TestMethod]
        public void TestMethod1()
        {
        }
    }
}
```

在前面的代码片段中，我们创建了一个新的测试项目，其中每个类和方法都添加了两个属性。通过这种方式添加它们，我们让框架知道这个类代表一个测试类，而这个方法是测试方法。

如前所述，属性的使用可以限制为特定类型。为了实现这一点，我们将使用属性目标。默认情况下，属性应用于前面的类型。但是，使用目标，我们可以设置属性是否应用于类、方法或汇编。

当目标设置为汇编时，意味着该属性应用于整个汇编。同样，目标可以设置为模块、字段、事件、方法、属性或类型。

例如，可以在字段上设置属性，让运行时知道可以接受哪种类型的输入。此外，它还可以设置在方法上，以指定它是一个普通方法还是一个 Web 方法。

框架定义的一些常见属性包括以下内容：

+   **全局**：在汇编或模块级别应用的属性通常是全局属性，例如`AssemblyVersionAttribute`。你可能已经在使用 Visual Studio 创建的每个.NET 项目中看到过它。

让我们看一下以下示例。在这里，你可以看到使用 Visual Studio 创建任何.NET 项目时创建的`assembly.cs`文件。每个汇编都包含以下代码，它告诉运行时正在执行当前汇编：

```cs
using System.Reflection;
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

[assembly: AssemblyTitle("Chapter10")]
[assembly: AssemblyDescription("")]
[assembly: AssemblyConfiguration("")]
[assembly: AssemblyCompany("")]
[assembly: AssemblyProduct("Chapter10")]
[assembly: AssemblyCopyright("Copyright © 2019")]
[assembly: AssemblyTrademark("")]
[assembly: AssemblyCulture("")]

[assembly: ComVisible(false)]

// The following GUID is for the ID of the typelib if this project is exposed to COM
[assembly: Guid("f8a2951a-4520-4d0f-ab30-7dd609db84d5")]

[assembly: AssemblyVersion("1.0.0.0")]
[assembly: AssemblyFileVersion("1.0.0.0")]
```

+   **过时**: 此属性允许我们标记不应使用的实体或类。因此，当应用时，它会产生一个在应用属性时提供的警告消息。此类定义了三个构造函数：第一个没有任何参数，第二个有一个参数，第三个有两个参数。从代码可读性的角度来看，建议我们使用带参数的构造函数，因为它们根据使用情况生成警告或错误消息。此外，在应用属性时将第二个参数设置为`true`将引发错误，而`false`将生成警告。在下面的代码中，我们将看到如何使用过时属性。

在下面的代码片段中，我们定义了一个名为`Firstclass`的类；后来，创建了一个名为`SecondClass`的新类。当我们希望新用户访问我们的库时使用第二个类而不是第一个类，那么我们可以使用带有消息的`Obsolete`属性，这样新用户就会看到并相应地采取行动：

```cs
[System.Obsolete(Firstclass is not used any more instead use SecondClass)]
class FirstClass
{
    //Do Firstthing
}

class SecondClass
{
    //Do Secondthing
}
```

+   **条件性**: 当应用条件属性时，前面代码的执行依赖于属性的评估。在实际项目场景中，当在一个实时环境中运行程序时，你不想记录信息和消息并填满你的存储空间。相反，你可以在你的日志方法上设置一个条件属性，这样你就可以在配置文件中的标志设置为`true`时进行写入。这样，你实际上可以实施选择日志。

在下面的代码中，我们有一个`LogMessage`方法；然而，类上方的属性会让运行时应用程序知道，当`LogErrorOn`属性设置为`yes`或`true`时，它应该执行此方法：

```cs
using System; 
using System.Diagnostics;
Public class Logging
{
    [Conditional(LogErrorON)]
    public static void LogMessage(string message)
    {
        Console.WriteLine(message)
    }
}
public class TestLogging
{     
    static void Main()     
    {         
        Trace.Msg("Main method executing...");         
        Console.WriteLine("This is the last statement.");     
    } 
} 

```

+   **调用者信息**: 调用者信息属性允许你检索调用方法的人。它们是`CallerfilePathAttribute`、`CallerLineNumberAttribute`、`CallerMemberNameAttribute`。每个都有自己的用途，正如它们的名称所暗示的那样。它们允许我们获取行号、方法名和文件路径。

# 创建自定义属性

C# 允许你定义自己的属性。这类似于正常的 C# 编程，其中你定义类和属性。要定义一个属性，你需要做的第一件事是从`System.Attribute`类继承它。你定义的类和属性用于在运行时存储和检索数据。

为了完成自定义属性的定义，你需要完成以下四个步骤：

+   属性使用

+   声明属性类

+   构造函数

+   属性

可以使用`System.AttributeUsageAttribute`定义属性使用。我们已提到，某些属性的使用有约束，这定义了它们可以在哪里使用——例如，在类、方法或属性中。`AttributeUsageAttribte`允许我们定义这样的约束。`AllowMultiple`指定此属性是否可以在特定类型上使用多次。定义子类的继承控件形成当前属性类。以下是使用`AttributeUsage`类的通用语法：

```cs
[AttributeUsage(AttributeTargets.All, Inherited = false, AllowMultiple = true)]
```

如您可能已观察到，您可以使用其构造函数在您想要定义的自定义属性上声明`AttributeUsage`属性，并使用三个参数。使用`AtributeTargetsAll`，您可以在任何类型为类、属性、方法等的元素上使用`CustomAttribute`。允许值的完整列表定义在[`docs.microsoft.com/en-us/dotnet/api/system.attributetargets?view=netframework-4.7.2#System_AttributeTargets_All`](https://docs.microsoft.com/en-us/dotnet/api/system.attributetargets?view=netframework-4.7.2#System_AttributeTargets_All)。

`Inherited`和`AllowMultiple`都是布尔属性，它们接受 true 或 false。

一旦我们定义了`AttributeUsage`，我们现在可以继续声明我们的自定义类。这应该是一个公共类，并且必须继承自`System.Attribute`类。

现在我们已经声明了我们的类，我们可以继续定义我们的构造函数和属性。框架允许我们定义一个或多个构造函数，覆盖所有可能的属性组合场景。让我们定义一个自定义属性。这些属性的构造函数接受三个参数——`AttributeTargets`、`AllowMultiple`和`Inherited`：

```cs
using System;

namespace Chapter10
{
    [System.AttributeUsage(System.AttributeTargets.Field | System.AttributeTargets.Property, Inherited =false,AllowMultiple = false)]
    public class CustomerAttribute : Attribute
    {
        public CustomerType Type { get; set; }

        public CustomerAttribute()
        {
            Type = CustomerType.Customer;
        }
    }

    public enum CustomerType
    {
        Customer,
        Supplier,
        Vendor
    }
}
```

上述代码定义了一个名为`CustomerAttribute`的自定义属性。我们还定义了一个`CustomerType`枚举，我们希望将其用作`Attribute`属性。通过在构造函数中不定义任何参数并将`Customer`类型分配给`Type`属性，我们告诉运行时，默认情况下，当其值是客户时。此外，此属性被设置为可以在字段或属性上使用，因此不能在类级别上使用。

现在，让我们看看我们如何在我们的类中使用这个属性：

```cs
namespace Chapter10
{

    internal class Account
    {
        public string CustomerName { get; set; }

        [Customer]
        public RatingType Rating { get; set; }
    }

    public enum RatingType
    {
        Gold =1,
        Silver =2,
        Bronze=3
    }
}
```

在这里，我们定义了一个`Account`类，在其中我们使用了我们的自定义属性。我们应用了一个不带任何参数的属性。这意味着，默认情况下，我们创建了一个客户类型的账户。在下一节中，我们将演示我们如何检索这些属性并在我们的应用程序逻辑中使用它们。

# 获取元数据

如您所知，获取属性信息与创建我们想要检索的属性的实例并调用`System.Attribute`类的`GetCustomAttribute`方法一样简单。

在以下示例中，我们定义了一个名为`ChapterInfo`的新属性，并定义了一个构造函数来标记其两个属性为必需参数：

```cs
[System.AttributeUsage(System.AttributeTargets.Class, Inherited =false,AllowMultiple = false)]
    public class ChapterInfoAttribute : Attribute
    {
        public string ChapterName{ get; set; }
        public string ChapterAuthor { get; set; }

        public ChapterInfoAttribute(string Name, string Author)
        {
            ChapterName = Name;
            ChapterAuthor = Author;
        }
    }
```

`ChapterName`和`ChapterAuthor`是开发者在使用此属性时必须定义的两个必需参数。

正如您所看到的，在下面的代码中，属性是在`Program`类上定义的，有两个值：`Name`和`Author`。在主方法中，调用了`GetCustomAttribute`来读取其属性，就像您对任何其他类类型变量所做的那样：

```cs
namespace Chapter10
{
    [ChapterInfo("SAMPLECHAPTER", "AUTHOR1")]
    class Program
    {
        static void Main(string[] args)
        {
            ChapterInfoAttribute _attribute = (ChapterInfoAttribute)Attribute.GetCustomAttribute(typeof(Program), typeof(ChapterInfoAttribute));
            Console.WriteLine($"Chapter Name is: {_attribute.ChapterName} and Chapter Author is: {_attribute.ChapterAuthor}");
            // Keep the console window open in debug mode.
            System.Console.WriteLine("Press any key to exit.");
            System.Console.ReadKey();
        }
    }
}
```

观察以下输出：

```cs
//Output
Chapter Name is: SAMPLECHAPTER and Chapter Author is: AUTHOR1
Press any key to exit.
```

正如您所看到的，在程序类属性定义中传递的（`[ChapterInfo("SAMPLECHAPTER", "AUTHOR1")]`）值被检索并显示出来。

# 反射

反射是从应用程序程序在运行时查询元数据的一种方式。反射提供了从加载到内存的程序集中获取的类型信息，您可以使用这些信息创建类的实例，并访问类的属性和方法。

例如，您的应用程序代码执行一个查询并返回一个数据集对象，但您的前端接受一个自定义类或模型，并且该模型在运行时定义。根据接收到的请求，可以使用反射在运行时创建所需的模型/类，通过遍历结果数据集来访问其属性或字段，并设置它们的值。

此外，在先前的章节中，我们学习了如何创建自定义属性。因此，在创建一个用于限制特定属性中数字的属性的属性的情况下，您可以使用反射来读取属性，获取前面的属性，实现应用逻辑以限制数字，或向用户显示消息。

我们还可以使用反射在运行时创建一个类型，并访问其方法和属性。反射与`System.Types`一起工作，以查询当前加载到内存并正在执行的程序集的信息。

**公共语言运行时**（**CLR**）管理具有相同作用域的对象的应用程序域。此过程包括将这些程序集加载到这些域中，并根据需要控制它们。

在.NET 世界中，程序集包含模块，模块包含类型，类型包含成员。程序集类用于加载程序集。模块用于标识程序集中类的信息以及全局和非全局方法。

`Reflection`类中提供了许多方法，例如`MethodInfo`、`PropertyInfo`、`Type`、`CustomAttribute`等。这些方法帮助开发者获取运行时信息。在先前的示例中，我们使用了`GetCustomAttribute`方法来检索属性信息并显示它。

# 调用方法和使用属性

在本节中，我们将探讨如何使用反射在运行时访问自定义类的属性和方法。

此示例旨在向您展示我们如何在运行时使用反射访问方法和属性。然而，根据您的需求，您可以动态地访问属性、它们的类型和方法以及它们的参数。

我们创建了一个新的自定义类，在其中我们定义了两个整数类型的属性：`Number1`和`Number2`。然后，我们定义了一个接受参数并返回要加或减的数字的公共方法：

```cs
internal class CustomClass1
    {
        public int Number1 { get; set; }
        public int Number2 { get; set; }

        public int Getresult(string action)
        {
            int result = 0;
            switch (action)
            {
                case "Add":
                    result = Number1 + Number2;
                    Console.WriteLine($"Sum of numbers {Number1} and {Number2} is : {result}");
                    break;

                case "Subtract":
                    result = Number1 - Number2;
                    Console.WriteLine($"Difference of numbers {Number1} and {Number2} is : {result}");
                    break;
            }
            return result;
        }
    }
```

然后，我们创建了一个简单的方法，通过这个方法我们可以访问我们之前创建的自定义类的属性和方法。在第一行，我们检索了自定义类的类型信息。使用这个类型，我们通过`Activator.CreateInstance`方法创建了一个类的实例。现在，使用我们检索到的类型的`GetProperties`方法，我们访问了所有的属性，并根据属性名称为每个属性设置了一个值。

在下一行，使用对象的`Type`信息，我们通过`GetMethod`方法检索`MethodInfo`。然后，我们通过两个不同的操作`Add`和`Subtract`调用了自定义类的`public`方法两次：

```cs
public static void GetResults()
        {
            Type objType = typeof(CustomClass1);
            object obj = Activator.CreateInstance(objType);
            foreach (PropertyInfo prop in objType.GetProperties())
            {
                if(prop.Name =="Number1")
                    prop.SetValue(obj, 100);
                if (prop.Name == "Number2")
                    prop.SetValue(obj, 50);
            }

            MethodInfo mInfo = objType.GetMethod("Getresult");
            mInfo.Invoke(obj, new string[] { "Add" });
            mInfo.Invoke(obj, new string[] { "Subtract" });
        }
```

如果你运行程序并逐行调试，你会看到每个属性都被检索了，并且已经设置了值。以下是程序的输出：

```cs
Sum of numbers 100 and 50 is : 150
Difference of numbers 100 and 50 is : 50
Press any key to exit.
```

这个示例很简单，因为我们创建了两个整数类型的属性。然而，在实际应用中，这样的简单场景可能并不存在。因此，在运行时，你需要使用`GetType`方法来理解获取到的属性类型。

此外，在示例中，我们能够获取到我们硬编码的`Custom`类的类型。使用泛型，我们甚至可以在运行时传递类并获取类型信息。

# 摘要

在这一章中，我们学习了如何使用系统属性，创建自定义属性，检索属性，然后在我们的应用程序逻辑中使用它们。通过反射检索属性信息，我们还探讨了如何创建类型，访问属性，以及调用方法。

在下一章中，我们将了解为什么验证应用程序输入很重要，流入我们应用程序的信息类型，以及我们如何处理它们。

# 问题

1.  在创建自定义属性时，是否可以设置一个目标来限制属性的用法？

    1.  True

    1.  False

1.  _______ 是用于检索属性信息的方法。

    1.  `GetAttributeValue`

    1.  `GetCustomAttribute`

    1.  `GetMetadata`

    1.  `GetAttributeMetadata`

1.  系统允许你从对象中检索属性信息吗？

    1.  True

    1.  False

# 答案

1.  1.  **True**

    1.  **GetAttributeValue**

    1.  **True**
