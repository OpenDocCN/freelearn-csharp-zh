# 第五章：异常处理

让我们从两个词开始：异常和处理。在英语中，**exception**一词指的是不经常发生的异常情况。在编程中，异常一词有类似的含义，但与软件代码有关。根据它们的性质，计算机程序应该只执行我们指示它们执行的操作，当计算机不能或无法遵循我们的指示时，这被认为是异常。如果计算机程序无法遵循我们的指示，它在软件世界中被归类为异常。

**错误**是编程中经常使用的另一个词。重要的是我们要明白错误和异常不是同一回事。错误指的是软件甚至无法运行的情况。更具体地说，错误意味着编写的代码包含错误，这就是为什么编译器无法编译/构建代码。另一方面，异常是发生在运行时的事情。区分这两个概念的最简单方法是：如果代码无法编译/构建，那么你的代码中有错误。如果代码编译/构建了，但当你运行它时出现了一些异常行为，那么这就是一个异常。

**异常处理**意味着在运行程序时处理/控制/监督发生的异常。本章我们将探讨以下主题：

+   为什么我们需要在编程中处理异常

+   C#编程中的异常处理

+   异常处理的基础知识

+   `try`和`catch`

+   如果不处理异常会发生什么

+   多个`catch`块

+   `throw`关键字的用途

+   `finally`块的作用

+   异常类

+   一些常见的异常类

+   异常处理最佳实践

# 为什么我们需要在编程中处理异常

想象一下你已经写了一些代码。代码应该按照你的指示执行，对吧？但由于某种原因，软件无法执行你给出的命令。也许软件面临一些问题，使得它无法运行。

例如，假设您已经指示软件读取文件，收集数据并将其存储在数据库中。然而，软件无法在文件应该存在的位置找到文件。文件找不到的原因可能有很多：文件可能已被某人删除，或者可能已被移动到另一个位置。现在，你的软件会怎么做？它不够聪明以自动处理这种情况。如果软件对自己的工作不清楚，它会抛出异常。作为软件开发人员，我们有责任告诉软件在这种情况下该怎么做。

软件会通过传递消息告诉我们它被卡住了，无法解决这种情况。但它应该对我们说什么？“救命！救命！”不是一个合适的消息，这种消息不会让开发人员的生活变得更容易。我们需要更多关于情况的信息，以便我们可以指导计算机相应地工作。因此，.NET 框架创建了一些在编程中经常发生的非常常见的异常。如果软件面临的问题有预定义的异常，它会抛出该异常。例如，假设有一个程序试图将一个数字除以零。从数学上讲，这是不可能的，但计算机必须这样做，因为你已经指示它这样做。现在计算机陷入了大麻烦；它感到困惑和无助。它试图按照你的指示将数字除以零，但编译器会阻止它并说“向程序先生求助！”，这意味着“向你的主人抛出一个`DivideByZeroException`来寻求帮助”。程序将抛出一个`DivideByZeroException`，并期望程序员编写的一些代码来处理它。这就是我们实际上会知道我们需要在程序中处理哪些异常。这就是为什么我们在编程中需要异常。

# C#编程中的异常处理

.NET 框架和 C#编程语言已经开发了一些强大的方法来处理异常。`System.Exceptions`是.NET 中的一个类，在系统命名空间下具有一些功能，可以帮助您管理运行时发生的异常，并防止程序崩溃。如果您在代码中没有正确处理异常，您的软件将崩溃。这就是为什么异常处理在软件开发中非常重要。

现在，您可能想知道如何在代码中处理异常。异常是意外的事情。您如何知道在您的代码中会发生哪种异常并导致程序崩溃？这是一个很好的问题，我相信在设计语言时也会提出这个问题。这就是为什么他们为.NET 提出了一个解决方案，它创建了一个非常美妙的机制来处理异常。

# 异常处理的基础知识

C#中的异常处理主要通过四个关键字实现：`try`、`catch`、`throw`和`finally`。稍后，我们将详细讨论这些关键字。但是，为了让您对这些关键字的含义有一个基本的了解，让我们简要讨论一下：

+   `try`：当您不确定代码的预期行为或存在异常可能性时，应将该代码放入`try`块中。如果该块内部发生异常，`try`块将抛出异常。如果没有异常发生，`try`块将像普通代码块一样。`try`块实际上是设计用来抛出异常的，这是它的主要任务。

+   `catch`：当捕获到异常时，将执行`catch`块。`try`块抛出的异常将由接下来的`catch`块处理。对于`try`块可以有多个`catch`块。每个`catch`块可以专门处理特定的异常。因此，我们应该为不同类型的异常编写不同的`catch`块。

+   `throw`：当您希望手动抛出异常时使用。可能存在您希望控制特定情况的情况。

+   `finally`：这是一段代码，将被强制执行。不管`try`块是否抛出异常，`finally`块都将被执行。这主要用于编写一些在任何情况下都必须处理的任务。

# 尝试和捕获

`try`和`catch`关键字是 C#异常处理中最重要的两个关键字。如果您编写一个没有`catch`块的`try`块，那么它就没有任何意义，因为如果`try`块抛出异常而没有`catch`块来处理它，那么有什么好处呢？异常仍然未处理。`catch`块实际上依赖于`try`块。如果没有与之关联的`try`块，`catch`块就不能存在。让我们看一下如何编写`try`-`catch`块：

```cs
try 
{
  int a = 5 / 0; 
} 
catch(DivideByZeroException ex)
{
  Console.WriteLine(“You have divided by zero”);
}
```

我们也可以为`try`块有更多的`catch`块。让我们看一个例子：

```cs
try 
{
  int a = 5 / 0; 
} 
catch(DivideByZeroException ex)
{ 
  Console.WriteLine(“You have divided by zero”); 
} 
catch(Exception ex) 
{ 
  Console.WriteLine(“Normal exception”); 
}
```

# 如果不处理异常会发生什么？

异常真的很重要吗？在逻辑中存在大量复杂性时，处理它们是否值得花费时间？是的，它们非常重要。让我们探讨一下如果不处理异常会发生什么。当触发异常时，如果没有代码处理它，异常将传递到系统运行时。

此外，当系统运行时遇到异常时，它会终止程序。所以，现在您明白为什么您应该处理异常了。如果您未能这样做，您的应用程序可能会在运行中间崩溃。我相信您个人不喜欢在使用它们时程序崩溃，所以我们必须小心编写无异常的软件。让我们看一个例子，看看如果未处理异常会发生什么：

```cs
Using system;

class LearnException {
    public static void Main()
    {
        int[] a = {1,2,3,4};
        for (int i=0; i<10; i++)
        {
            Console.WriteLine(a[i]);
        }
    }
}
```

如果我们运行这段代码，那么前四次运行时，它将完美执行并打印出从一到四的一些数字。但之后，它将抛出`IndexOutOfRangeException`的异常，并且系统运行时将终止程序。

# 多个 catch 块

在一个`try`块中获得不同类型的异常是正常的。但是你该如何处理它们呢？您不应该使用通用异常来做这个。如果您抛出通用异常而不是抛出特定异常，您可能会错过有关异常的一些重要信息。因此，C#语言为`try`块引入了多个`catch`块。您可以指定一个`catch`块，它将被一个类型的异常调用，并且您可以创建其他`catch`块，每个后面都有不同的异常类型。当抛出特定异常时，只有那个特定的`catch`块将被执行，如果它有一个专门的`catch`块来处理这种类型的异常。让我们看一个例子：

```cs
using System;

class ManyCatchBlocks 
{     
    public static void Main()
    {
        try
        {
            var a = 5;
            var b = 0;
            Console.WriteLine("Here we will divide 5 by 0");
            var c = a/b;
        }
        catch(IndexOutOfRangeException ex)
        {
            Console.WriteLine("Index is out of range " + ex);
        }
        catch(DivideByZeroException ex)
        {
            Console.WriteLine("You have divided by zero, which is not correct!");
        }
    }
}
```

如果运行上述代码，您将看到只有第二个`catch`块被执行。如果您打开控制台窗口，您将看到以下行已被打印出来：

```cs
You have divided by zero, which is not correct!
```

因此，我们可以看到，如果有多个`catch`块，只有与抛出的异常类型匹配的特定`catch`块将被执行。

现在你可能会想，“你说我们不应该使用通用异常处理程序。但为什么呢？是的，我们可能会错过一些信息，但我的系统没有崩溃！这样做不是更好吗？”实际上，这个问题的答案并不直接。这可能因系统而异，但让我告诉你为什么有时候你希望系统崩溃。假设你有一个处理非常复杂和敏感数据的系统。当这样的系统发生异常时，允许客户继续使用软件可能非常危险。客户可能会对数据造成严重破坏，因为异常没有得到适当处理。但是，如果你认为即使出现未知异常，如果允许用户继续使用系统也不会有问题，你可以使用通用的`catch`块。现在让我告诉你如何做到这一点。如果你希望`catch`块捕获任何类型的异常，无论异常类型如何，那么你的`catch`块应该接受`Exception`类作为参数，如下面的代码所示：

```cs
using System;

namespace ExceptionCode
{
  class Program
  {
    static void Main(string[] args)
    {
      try
      {
        var a = 0;
        var b = 5;
        var c = b / a;
      }
      catch (IndexOutOfRangeException ex)
      {
        Console.WriteLine("Index out of range " + ex);
      }
      catch (Exception ex)
      {
        Console.WriteLine("I will catch you exception! You can't hide from me!" + ex);
      }

      Console.WriteLine("Hello");
      Console.ReadKey();
     }
   }
}
```

或者，您还可以向`catch`块传递一个`no`参数。这也将捕获每种类型的异常并执行主体中的代码。以下代码给出了一个示例：

```cs
using System;

namespace ExceptionCode
{
  class Program
  {
    static void Main(string[] args)
    {
      try
      {
        var a = 0;
        var b = 5;
        var c = b / a;
      }
      catch (IndexOutOfRangeException ex)
      {
        Console.WriteLine("Index out of range " + ex);
      }
      catch
      {
        Console.WriteLine("I will catch you exception! You can't hide from me!");
      }

      Console.WriteLine("Hello");
      Console.ReadKey();
     }
   }
}
```

但是，请记住，这必须是最后一个`catch`块，否则将会出现运行时错误。

# 使用 throw 关键字

有时，在您自己的程序中，您必须自己创建异常。不，不是为了报复用户，而是为了您的应用程序。有时，有些情况下，您需要抛出异常来绕过困难，记录一些东西，或者只是重定向软件的流程。不用担心：通过这样做，您不会成为坏人；实际上，您是在拯救程序免受麻烦的英雄。但是，您如何创建异常呢？为此，C#有一个名为`throw`的关键字。这个关键字将帮助您创建异常类型的实例并抛出它。让我给你一个`throw`关键字的例子：

```cs
using System;

namespace ExceptionCode
{
 class Program
 {
 public static void Main(string[] args)
 {
 try
 {
 Console.WriteLine("You are the boss!");
 throw new DivideByZeroException();
 }
 catch (IndexOutOfRangeException ex)
 {
 Console.WriteLine("Index out of range " + ex);
 }
 catch (DivideByZeroException ex)
 {
 Console.WriteLine("Divide by zero " + ex);
 }
 catch
 {
 Console.WriteLine("I will catch you exception! You can't hide from me!");
 }

 Console.WriteLine("See, i told you!");
 Console.ReadKey();
 }
 }
}
```

输出如下：

![](img/19a3ed57-7485-4785-872d-de991b0338f1.png)

您可以看到，如果运行上述代码，将执行`DivideByZeroException` `catch`块。

因此，如果你想抛出异常（因为你希望上层的`catch`块来处理它，例如），你只需抛出一个新的异常实例。这可以是任何类型的异常，包括系统异常或自定义异常。只需记住有一个`catch`块将处理它。

# finally 块是做什么的？

当我们说“最后”，我们指的是我们一直在等待的或者将要结束进程的东西。在异常处理中也是差不多的。`finally` 块是一段无论 `try` 或 `catch` 块中发生了什么都会执行的代码。无论抛出了什么类型的异常，或者是否被处理，`finally` 块都会执行。现在你可能会问，"*为什么我们需要* `finally` *块呢？如果程序中有任何异常，我们会用* `catch` *块来处理它！我们不能把代码写在* `catch` *块里而不是* `finally` *块里吗？*"

是的，你可以，但是如果抛出了异常而 `catch` 块没有被触发会发生什么？这意味着 `catch` 块内的代码将不会被执行。因此，`finally` 块很重要。无论是否有异常，`finally` 块都会运行。让我给你展示一个 `finally` 块的例子：

```cs
using System;

namespace ExceptionCode
{
 class Program
 {
 static void Main(string[] args)
 {
 try
 {
 int a = 0;
 int b = 5;
 int c = b / a;
 }
 catch (IndexOutOfRangeException ex)
 {
 Console.WriteLine("Index out of range " + ex);
 }
 catch (DivideByZeroException ex)
 {
 Console.WriteLine("Divide by zero " + ex);
 }
 catch
 {
 Console.WriteLine("I will catch you exception! You can't hide from me!");
 }
 finally
 {
 Console.WriteLine("I am the finally block i will run by hook or by crook!");
 }
 Console.ReadLine();
 }
 }
}
```

输出如下：

![](img/32173a85-0984-4cbd-acbb-cb9eb559c8e1.png)

`finally` 块的一个重要用例可能是在 `try` 块中打开数据库连接！你必须关闭它，否则该连接将一直保持打开状态，会占用大量资源。此外，数据库可以建立的连接数量是有限的，所以如果你打开了一个连接却没有关闭它，那么这个连接字符串就浪费了。最佳实践是在完成与连接的工作后立即关闭连接。

`finally` 块在这里发挥了最好的作用。不管在 `try` 块中发生了什么，`finally` 块都会关闭连接，如下面的代码所示：

```cs
using System;

namespace ExceptionCode
{
  class Program
  {
    static void Main(string[] args)
    {
      try
      {
        // Step 1: Established database connection

        // Step 2: Do some activity in database
      }
      catch (IndexOutOfRangeException ex)
      {
        // Handle IndexOutOfRangeExceptions here
      }
      catch (DivideByZeroException ex)
      {
        // Handle DivideByZeroException here
      }
      catch
      {
        // Handle All other exception here
      }
      finally
      {
        // Close the database connection
      }
    }
  }
}
```

在这里，我们在 `try` 块中执行了两个主要任务。首先，我们打开了数据库连接，其次，我们在数据库中执行了一些活动。现在，如果在执行任何这些任务时发生了异常，那么异常将被 `catch` 块处理。最后，`finally` 块将关闭数据库连接。

`finally` 块不是处理异常必须要有的东西，但如果需要的话，应该使用它。

# 异常类

`exception` 简单来说就是 C# 中的一个类。它有一些属性和方法。最常用的四个属性如下：

| **属性** | **描述** |
| --- | --- |
| `Message` | 这包含了异常的内容。 |
| `StackTrace` | 这包含了方法调用堆栈信息。 |
| `TargetSite` | 这提供了一个包含发生异常的方法的对象。 |
| `InnerException` | 这提供了引起异常的异常实例。 |

异常类的属性和方法

这个类中最受欢迎的方法之一是 `ToString()`。这个方法返回一个包含异常信息的字符串。当以字符串格式表示时，异常更容易阅读和理解。

让我们看一个使用这些属性和方法的例子：

```cs
using System;

namespace ExceptionCode
{
 class Program
 {
 static void Main(string[] args)
 {
 try
 {
 var a = 0;
 var b = 5;
 var c = b / a;
 }
 catch (DivideByZeroException ex)
 {
 Console.WriteLine("Message:");
 Console.WriteLine(ex.Message);
 Console.WriteLine("Stack Trace:");
 Console.WriteLine(ex.StackTrace);
 Console.WriteLine("String:");
 Console.WriteLine(ex.ToString());
 }

 Console.ReadKey();
 }
 }
}
```

输出如下：

![](img/9295d985-106f-47ac-9530-b2818991d2b0.png)

在这里，我们可以看到异常的 `message` 属性包含了信息 `Attempted to divide by zero`。此外，`ToString()` 方法提供了大量关于异常的信息。这些属性和方法在处理程序中处理异常时会帮助你很多。

# 一些常见的异常类

.NET Framework 中有许多异常类可用。.NET Framework 团队创建了这些类来简化开发人员的生活。.NET Framework 提供了关于异常的具体信息。以下是一些常见的异常类：

| **异常类** | **描述** |
| --- | --- |
| `DivideByZeroException` | 当任何数字被零除时，会抛出此异常。 |
| `IndexOutOfRangeException` | 当应用程序尝试使用不存在的数组索引时，会抛出此异常。 |
| `InvalidCastException` | 当尝试执行无效转换时，会引发此异常。 |
| `NullReferenceException` | 当尝试使用或访问空引用类型时，会引发此异常。 |

.NET 框架的不同异常类

让我们看一个示例，其中使用了这些异常类中的一个。在这个例子中，我们使用了`IndexOutOfRange`异常类：

```cs
using System;

namespace ExceptionCode
{
 class Program
 {
 static void Main(string[] args)
 {
 int[] a = new int[] {1,2,3};

 try
 {
 Console.WriteLine(a[5]);
 }
 catch (IndexOutOfRangeException ex)
 {
 Console.WriteLine("Message:");
 Console.WriteLine(ex.Message);
 Console.WriteLine("Stack Trace:");
 Console.WriteLine(ex.StackTrace);
 Console.WriteLine("String:");
 Console.WriteLine(ex.ToString());
 }

 Console.ReadKey();
 }
 }
}
```

输出如下：

![](img/2ae2fd64-2e62-4365-aa7c-23b2e7104ae7.png)

# 用户定义的异常

有时，您可能会遇到一种情况，认为预定义的异常不满足您的条件。在这种情况下，您可能希望有一种方法来创建自己的异常类并使用它们。值得庆幸的是，在 C#中，实际上有一种机制可以创建自定义异常，并且可以编写适用于该类型异常的任何消息。让我们看一个创建和使用自定义异常的示例：

```cs
using System;

namespace ExceptionCode
{

 class HelloException : Exception
 {
 public HelloException() { }
 public HelloException(string message) : base(message) { }
 public HelloException(string message, Exception inner) : base(message, inner) { }
 }

 class Program
 {
 static void Main(string[] args)
 {
 try
 {
 throw new HelloException("Hello is an exception!");
 }
 catch (HelloException ex)
 {
 Console.WriteLine("Exception Message:");
 Console.WriteLine(ex.Message);
 }

 Console.ReadKey();
 }
 }
}
```

输出如下：

![](img/b38a2822-1170-4403-8bfe-b6e93aea6a89.png)

因此，我们可以从上面的示例中看到，您只需创建一个将扩展`Exception`类的类。该类应该有三个构造函数：一个不应该带任何参数，一个应该带一个字符串并将其传递给基类，一个应该带一个字符串和一个异常并将其传递给基类。

使用自定义异常就像使用.NET Framework 提供的任何其他内置异常一样。

# 异常筛选器

在撰写本文时，异常筛选器功能并不是很古老——它是在 C# 6 中引入的。其主要好处是可以在一个块中捕获更具体的异常。让我们看一个例子：

```cs
using System;

namespace ExceptionCode
{
 class Program
 {
 static void Main(string[] args)
 {

 int[] a = new int[] {1,2,3};

 try
 {
 Console.WriteLine(a[5]);
 }
 catch (IndexOutOfRangeException ex) when (ex.Message == "Test Message")
 {
 Console.WriteLine("Message:");
 Console.WriteLine("Test Message");
 }
 catch (IndexOutOfRangeException ex) when (ex.Message == "Index was outside the bounds of the array.")
 {
 Console.WriteLine("Message:");
 Console.WriteLine(ex.Message);
 Console.WriteLine("Stack Trace:");
 Console.WriteLine(ex.StackTrace);
 Console.WriteLine("String:");
 Console.WriteLine(ex.ToString());
 }

 Console.ReadKey();
 }
 }
}
```

输出如下：

![](img/0742e721-1391-4da1-89eb-707e2c898206.png)

要筛选异常，必须在`catch`声明行的旁边使用`when`关键字。因此，当抛出任何异常时，它将检查异常的类型，然后检查`when`关键字之后提供的条件。在我们的示例中，异常类型是`IndexOutOfRangeException`，条件是`ex.Message == "Index was outside the bounds of the array."`。我们可以看到，当代码运行时，只有满足所有条件的特定`catch`块被执行。

# 异常处理最佳实践

正如您所看到的，处理异常有不同的方式：有时可以抛出异常，有时可以使用`finally`块，有时可以使用多个`catch`块。因此，如果您对异常处理没有足够的经验，可能会在开始时感到困惑。但幸运的是，C#社区为异常处理提供了一些最佳实践。让我们看看其中一些：

+   使用`finally`块关闭/清理可能会在将来引起问题的依赖资源。

+   捕获特定异常并正确处理。如果需要，可以使用多个`catch`块。

+   如有需要，创建并使用自定义异常。

+   尽快处理异常。

+   如果可以使用特定处理程序处理异常，则不要使用通用异常处理程序。

+   异常消息应该非常清晰。

# 总结

我们都梦想着一个没有错误或意外情况的完美世界，但现实中这是不可能的。软件开发也不免于错误和异常。软件开发人员不希望他们的软件崩溃，但意外异常时有发生。因此，处理这些异常对于开发出色的软件是必要的。在本章中，我们熟悉了软件开发中异常的概念。我们还学习了如何处理异常，为什么需要处理异常，如何创建自定义异常以及许多其他重要主题。在应用程序中实施异常处理时，请尽量遵循最佳实践，以确保应用程序运行顺畅。
