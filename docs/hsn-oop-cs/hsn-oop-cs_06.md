# 第六章：事件和委托

事件和委托可能看起来像复杂的编程主题，但实际上并不是。在本章中，我们将首先通过分析它们各自名称的含义来学习这些概念。然后我们将把这些词的一般含义与编程联系起来。在本章中，我们将看到很多示例代码，这将帮助我们轻松理解这些概念。在我们深入讨论之前，让我们先看一下本章将涵盖的主题：

+   如何创建和使用委托

+   方法组转换

+   多播

+   协变和逆变

+   事件和多播事件

+   .NET 事件指南

# 什么是委托？

**委托**是一个代理，一个替代者，或者某人的代表。例如，我们可能在报纸上看到另一个国家的代表来到我们国家会见高级官员。这个人是一个代表，因为他们来到我们国家代表他们自己的国家。他们可能是总统、总理或者那个国家的任何其他高级官员的代表。让我们想象一下，这个代表是总统的代表。也许总统因某种原因无法亲自出席这次会议，这就是为什么派遣了一个代表代表他们。这个代表将会做总统应该在这次旅行中做的工作，并代表总统做出决定。代表不是一个固定的个人；可以是总统选择的任何合格的人。

委托的概念在软件开发中是类似的。我们可以有一个功能，其中一个方法不执行它被要求执行的实际工作，而是调用另一个方法来执行那项工作。此外，在编程中，那个不执行实际工作而是将其传递给另一个方法的方法被称为**委托**。因此，委托实际上将持有一个方法的引用。当调用委托时，引用的方法将被调用和执行。

现在，你可能会问，*"如果委托要调用另一个方法，为什么我不直接调用这个方法呢？"* 好吧，我们这样做是因为如果你直接调用方法，你会失去灵活性，使你的代码耦合在一起。你在代码中硬编码了方法名，所以每当那行代码运行时，该方法就会被执行。然而，使用委托，你可以在运行时决定调用哪个方法，而不是在编译时。

# 如何创建和使用委托

要创建一个委托，我们需要使用`delegate`关键字。让我向你展示如何以一般形式声明一个委托：

```cs
delegate returnType delegateName(parameters)
```

现在让我给你展示一些真实的示例代码：

```cs
using System;

namespace Delegate1
{
  delegate int MathFunc(int a, int b);

  class Program
  {
    static void Main(string[] args)
    {
      MathFunc mf = new MathFunc(add);

      Console.WriteLine("add");
      Console.WriteLine(mf(4, 5));

      mf = new MathFunc(sub);

      Console.WriteLine("sub");
      Console.WriteLine(mf(4, 5));

      Console.ReadKey();
    }

    public static int add(int a, int b)
    {
      return a + b;
    }

    public static int sub(int a, int b)
    {
      return (a > b) ? (a - b) : (b - a);
    }
  }
}
```

上述代码的输出将如下所示：

![](img/ec78bb93-f19d-4cd4-9c4b-3234205bb9c7.png)

现在让我们讨论上述代码。在命名空间内的顶部，我们可以看到委托的声明，如下所示：

```cs
delegate int MathFunc(int a, int b);
```

我们使用了`delegate`关键字来告诉编译器我们在声明一个`delegate`。然后我们将返回类型设置为`int`，并命名了委托为`MathFunc`。我们还在这个委托中传递了两个`int`类型的参数。

之后，`program`类开始运行，在该类中，除了主方法外，我们还有两个方法。一个是`add`，另一个是`sub`。如果你仔细观察这些方法，你会发现它们与委托具有相同的签名。这是故意这样做的，因为当方法具有与委托相同的签名时，方法可以使用`delegate`。

现在，如果我们看一下主方法，我们会发现以下有趣的代码：

```cs
MathFunc mf = new MathFunc(add);
```

在主方法的第一行，我们创建了一个代理对象。在这样做时，我们将`add`方法传递给构造函数。这是必需的，因为你需要传递一个你想要使用代理的方法。然后我们可以看到，当我们调用代理`mf(4,5)`时，它返回`9`。这意味着它实际上调用了`add`方法。之后，我们将`sub`分配给`delegate`。在调用`mf(4,5)`时，这次我们得到了`1`。这意味着调用了`sub`方法。通过这种方式，一个`delegate`可以用于具有相同签名的许多方法。

# 方法组转换

在上一个例子中，我们看到了如何创建一个代理对象并在构造函数中传递方法名。现在我们将看另一种实现相同目的的方法，但更简单。这被称为**方法组转换**。在这里，你不需要初始化`delegate`对象，而是可以直接将方法分配给它。让我给你举个例子：

```cs
using System;

namespace Delegate1
{
 delegate int MathFunc(int a, int b);

 class Program
 {
 static void Main(string[] args)
 {
 MathFunc mf = add;

 Console.WriteLine("add");
 Console.WriteLine(mf(4, 5));

 mf = sub;

 Console.WriteLine("sub");
 Console.WriteLine(mf(4, 5));
 Console.ReadKey();
 }

 public static int add(int a, int b)
 {
 return a + b;
 }

 public static int sub(int a, int b)
 {
 return (a > b) ? (a - b) : (b - a);
 }
 }
}
```

在这里，我们可以看到，我们直接将方法分配给它，而不是在构造函数中传递方法名。这是在 C#中分配代理的一种快速方法。

# 使用静态和实例方法作为代理

在之前的例子中，我们在代理中使用了静态方法。然而，你也可以在代理中使用实例方法。让我们看一个例子：

```cs
using System;

namespace Delegate1
{
  delegate int MathFunc(int a, int b);

  class Program
  {
    static void Main(string[] args)
    {
      MyMath mc = new MyMath();

      MathFunc mf = mc.add;

      Console.WriteLine("add");
      Console.WriteLine(mf(4, 5));

      mf = mc.sub;

      Console.WriteLine("sub");
      Console.WriteLine(mf(4, 5));

      Console.ReadKey();
    }
  }
  class MyMath
  {
    public int add(int a, int b)
    {
      return a + b;
    }

    public int sub(int a, int b)
    {
      return (a > b) ? (a - b) : (b - a);
    }
  }
}
```

在上面的例子中，我们可以看到我们在`MyMath`类下有实例方法。要在代理中使用这些方法，我们首先必须创建该类的对象，然后简单地使用对象实例将方法分配给代理。

# 多播

**多播**是代理的一个很好的特性。通过多播，你可以将多个方法分配给一个代理。当执行该代理时，它依次运行所有被分配的方法。使用`+`或`+=`运算符，你可以向代理添加方法。还有一种方法可以从代理中删除添加的方法。要做到这一点，你必须使用`-`或`-=`运算符。让我们看一个例子来清楚地理解多播是什么：

```cs
using System;

namespace MyDelegate
{
  delegate void MathFunc(ref int a);

  class Program
  {
    static void Main(string[] args)
    {
      MathFunc mf;
      int number = 10;
      MathFunc myAdd = MyMath.add5;
      MathFunc mySub = MyMath.sub3;

      mf = myAdd;
      mf += mySub;

      mf(ref number);

      Console.WriteLine($"Final number: {number}");

      Console.ReadKey();
    }
  }

  class MyMath
  {
    public static void add5(ref int a)
    {
      a = a + 5;
      Console.WriteLine($"After adding 5 the answer is {a}");
    }

    public static void sub3(ref int a)
    {
      a = a - 3;
      Console.WriteLine($"After subtracting 3 the answer is {a}");
    }
  }
}
```

上面的代码将给出以下输出：

![](img/9e13be43-8af9-4cc0-84fb-356853c3a9e3.png)

在这里，我们可以看到我们的代理依次执行了两种方法。我们必须记住它的工作原理就像一个队列，所以你添加的第一个方法将是第一个执行的方法。现在让我们看看如何从代理中删除一个方法：

```cs
using System;

namespace MyDelegate
{
  delegate void MathFunc(ref int a);

  class Program
  {
    static void Main(string[] args)
    {
      MathFunc mf;
      MathFunc myAdd = MyMath.add5;
      MathFunc mySub = MyMath.sub3;
      MathFunc myMul = MyMath.mul10;

      mf = myAdd;
      mf += mySub;
      int number = 10;

      mf(ref number);

      mf -= mySub;
      mf += myMul;
      number = 10;

      mf(ref number);

      Console.WriteLine($"Final number: {number}");

      Console.ReadKey();
    }
  }

  class MyMath
  {
    public static void add5(ref int a)
    {
      a = a + 5;
      Console.WriteLine($"After adding 5 the answer is {a}");
    }

    public static void sub3(ref int a)
    {
      a = a - 3;
      Console.WriteLine($"After subtracting 3 the answer is {a}");
    }

    public static void mul10(ref int a)
    {
      a = a * 10;
      Console.WriteLine($"After multiplying 10 the answer is {a}");
    }
  }
}
```

上面的代码将给我们以下输出：

![](img/375ab662-d4d2-4e43-98e7-77e000cf9870.png)

在这里，我们首先向代理添加了两种方法。然后，我们删除了`sub3`方法并添加了`mul10`方法。在进行了所有这些更改后，当我们执行了代理时，我们看到`5`被加到了数字上，然后`10`被乘以数字。没有发生减法。

# 协变和逆变

有两个重要的代理特性。到目前为止，我们学到的是通常情况下，要向代理注册一个方法，该方法必须与代理的签名匹配。这意味着方法和代理的返回类型和参数必须相同。然而，通过协变和逆变的概念，你实际上可以向代理注册不具有相同返回类型或参数的方法。然后在调用时，代理将能够执行它们。

**协变**是指当你将一个返回类型是委托返回类型的派生类型的方法分配给委托时。例如，如果类`B`是从类`A`派生出来的，并且如果委托返回类`A`，那么可以向委托注册返回类`B`的方法。让我们看看以下代码中的例子：

```cs
using System;

namespace EventsAndDelegates
{
 public delegate A DoSomething();

 public class A
 {
 public int value { get; set; }
 }

 public class B : A {}

 public class Program
 {
 public static A WorkA()
 {
 A a = new A();
 a.value = 1;
 return a;
 }

 public static B WorkB()
 {
 B b = new B();
 b.value = 2;
 return b;
 }

 public static void Main(string[] args)
 {
 A someA = new A();

 DoSomething something = WorkB;

 someA = something();

 Console.WriteLine("The value is " + someA.value);

 Console.ReadLine();
 }
 }
}
```

上面代码的输出将如下所示：

![](img/a25f3142-a1f1-4a9a-b26d-b33aa0902724.png)

另一方面，**逆变**是指当一个方法传递给委托时，该方法的参数与委托的参数不匹配。在这里，我们必须记住，方法的参数类型至少必须派生自委托的参数类型。让我们看一个逆变的例子：

```cs
using System;

namespace EventsAndDelegates
{
 public delegate int DoSomething(B b);

 public class A
 {
 public int value = 5;
 }

 public class B : A {}

 public class Program
 {
 public static int WorkA(A a)
 {
 Console.WriteLine("Method WorkA called: ");
 return a.value * 5;
 }

 public static int WorkB(B b)
 {
 Console.WriteLine("Method WorkB called: ");
 return b.value * 10;
 }

 public static void Main(string[] args)
 {
 B someB = new B();

 DoSomething something = WorkA;

 int result = something(someB);

 Console.WriteLine("The value is " + result);

 Console.ReadLine();
 }
 }
}
```

上面的代码将产生以下输出：

![](img/8c0d2d79-d593-41b5-acac-a4a38caeab8e.png)

在这里，我们可以看到委托以类型`B`作为参数。然而，当`WorkA`方法被注册为委托中的一个方法时，它并没有给出任何错误或警告，尽管`WorkA`方法的参数类型是`A`类型。它能够工作的原因是因为`B`类型是从`A`类型派生出来的。

# 事件

你可以将**事件**看作是在某些情况下执行的一种方法，并通知处理程序或委托有关该事件的发生。例如，当你订阅电子邮件时，你会收到来自网站的关于最新文章、博客帖子或新闻的电子邮件。这些电子邮件可以是每天、每周、每月、每年，或者根据你选择的其他指定时间段。这些电子邮件不是由人手动发送的，而是由自动系统/软件发送的。可以使用事件来开发这种自动电子邮件发送器。现在，你可能会想，为什么我需要一个事件来做这个，我不能通过普通方法发送电子邮件给订阅者吗？是的，你可以。但是，假设在不久的将来，你还想引入一个功能，即在移动应用程序上收到通知。你将不得不更改代码并添加该功能。几天后，如果你想进一步扩展你的系统并向特定订阅者发送短信，你又必须再次更改代码。不仅如此，如果你使用普通方法编写代码，那么你编写的代码将非常紧密耦合。你可以使用`event`来解决这类问题。你还可以创建不同的事件处理程序，并将这些事件处理程序分配给一个事件，这样，每当该事件被触发时，它将通知所有注册的处理程序来执行它们的工作。现在让我们看一个例子来使这一点更清晰：

```cs
using System;

namespace EventsAndDelegates
{
  public delegate void GetResult();

  public class ResultPublishEvent
  {
    public event GetResult PublishResult;

    public void PublishResultNow()
    {
      if (PublishResult != null)
      {
        Console.WriteLine("We are publishing the results now!");
        Console.WriteLine("");
        PublishResult();
      }
    }
  }

  public class EmailEventHandler
  {
    public void SendEmail()
    {
      Console.WriteLine("Results have been emailed successfully!");
    }
  }

  public class Program
  {
    public static void Main(string[] args)
    {
      ResultPublishEvent e = new ResultPublishEvent();

      EmailEventHandler email = new EmailEventHandler();

      e.PublishResult += email.SendEmail;
      e.PublishResultNow();

      Console.ReadLine();
    }
  }
}
```

上面代码的输出如下：

![](img/4c7feacb-1170-499c-b38e-72151467b6f4.png)

在上面的代码中，我们可以看到，当调用`PublishResultNow()`方法时，它基本上触发了`PublishResult`事件。此外，订阅了该事件的`SendMail()`方法被执行，并在控制台上打印出`Results have been emailed successfully!`。

# 多播事件

在事件中，你可以像在委托中一样进行多播。这意味着你可以注册多个事件处理程序（订阅事件的方法）到一个事件中，当事件被触发时，所有这些处理程序都会依次执行。要进行多播，你必须使用`+=`符号来注册事件处理程序到事件中。你也可以使用`-=`运算符从事件中移除事件处理程序。当应用多播时，首先注册的事件处理程序将首先执行，然后是第二个，依此类推。通过多播，你可以在应用程序中轻松扩展或减少事件处理程序而不需要做太多工作。让我们看一个多播的例子：

```cs
using System;

namespace EventsAndDelegates
{
 public delegate void GetResult();

 public class ResultPublishEvent
 {
 public event GetResult PublishResult;

 public void PublishResultNow()
 {
 if (PublishResult != null)
 {
 Console.WriteLine("");
 Console.WriteLine("We are publishing the results now!");
 Console.WriteLine("");
 PublishResult();
 }
 }
 }

 public class EmailEventHandler
 {
 public void SendEmail()
 {
 Console.WriteLine("Results have been emailed successfully!");
 }
 }

 public class SmsEventHandler
 {
 public void SmsSender()
 {
 Console.WriteLine("Results have been messaged successfully!");
 }
 }

 public class Program
 {
 public static void Main(string[] args)
 {
 ResultPublishEvent e = new ResultPublishEvent();

 EmailEventHandler email = new EmailEventHandler();
 SmsEventHandler sms = new SmsEventHandler();

 e.PublishResult += email.SendEmail;
 e.PublishResult += sms.SmsSender;

 e.PublishResultNow();

 e.PublishResult -= sms.SmsSender;

 e.PublishResultNow();

 Console.ReadLine();
 }
 }
}
```

上面代码的输出如下：

![](img/fd85efc4-6818-457e-ab15-3c69366339d0.png)

现在，如果我们分析上面的代码，我们可以看到我们创建了另一个类`SmsEventHandler`，这个类有一个名为`SmsSender`的方法，它的签名与我们的委托`GetResult`相同，如下面的代码所示：

```cs
public class SmsEventHandler
{
  public void SmsSender()
  {
    Console.WriteLine("Results have been messaged successfully!");
  }
}
```

然后，在主方法中，我们创建了这个`SmsEventHandler`类的一个实例，并将`SmsSender`方法注册到事件中，如下面的代码所示：

```cs
e.PublishResult += sms.SmsSender;
```

触发事件一次后，我们使用`-=`运算符从事件中移除`SmsSender`事件处理程序，如下所示：

```cs
e.PublishResult -= sms.SmsSender;
```

当我们再次触发事件时，可以在输出中看到只有电子邮件事件处理程序被执行。

# .NET 中的事件准则

为了更好的稳定性，.NET Framework 提供了一些在 C# 中使用事件的准则。并不是说你一定要遵循这些准则，但遵循这些准则肯定会使你的程序更加高效。现在让我们看看需要遵循哪些准则。

事件应该有以下两个参数：

+   生成事件的对象的引用

+   `EventArgs` 的类型将保存事件处理程序所需的其他重要信息

代码的一般形式应该如下：

```cs
void eventHandler(object sender, EventArgs e)
{
}
```

让我们看一个遵循这些准则的例子：

```cs
using System;

namespace EventsAndDelegates
{
  class MyEventArgs : EventArgs
  {
    public int number;
  }

  delegate void MyEventHandler(object sender, MyEventArgs e);

  class MyEvent
  {
    public static int counter = 0;

    public event MyEventHandler SomeEvent;

    public void GetSomeEvent()
    {
      MyEventArgs a = new MyEventArgs();

      if (SomeEvent != null)
      {
        a.number = counter++;
        SomeEvent(this, a);
      }
    }

  }

  class X
  {
    public void Handler(object sender, MyEventArgs e)
    {
      Console.WriteLine("Event number: " + e.number);
      Console.WriteLine("Source Object: " + sender);
      Console.WriteLine();
    }
  }

  public class Program
  {
    public static void Main(string[] args)
    {
      X x = new X();

      MyEvent myEvent = new MyEvent();

      myEvent.SomeEvent += x.Handler;

      myEvent.GetSomeEvent();
      myEvent.GetSomeEvent();

      Console.ReadLine();
    }
  }
}
```

上述代码的输出如下：

![](img/0eb0b9af-e892-43c3-ab91-1f574877789d.png)

如果我们分析上述代码，我们会看到我们使用 `EventArgs` 参数传递了计数器的值，使用 `object` 参数传递了对象的引用。

# 摘要

在本章中，我们学习了委托和事件。这些主题在软件开发中非常重要，因为它们提供了在特定场合自动化代码的功能。这些概念在 Web 开发领域都被广泛使用。

在下一章中，我们将学习 C# 中的泛型和集合。这些是 C# 编程语言非常有趣的特性，你可以使用它们在程序中编写通用的委托。
