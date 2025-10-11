# 创建和实现事件与回调

本章重点介绍 C#中的事件和回调。了解它们很重要，因为它们使我们能够更好地控制程序。事件是对象属性更改或按钮被点击时发出的消息或通知。回调，也称为代理，持有函数的引用。C#自带 Lambda 表达式，可以用来创建代理。这些也被称为匿名方法。

我们还将花一些时间了解一个新的运算符，称为 Lambda 运算符。这些用于 Lambda 表达式。它们是在 C# 3.0 版本中引入的，以便开发人员可以实例化代理。Lambda 表达式取代了 C# 2.0 中引入的匿名方法，并且现在被广泛使用。

在本章中，我们将涵盖以下主题：

+   理解代理

+   处理和引发事件

到本章结束时，您将了解代理是什么以及如何在事件和回调中使用它们。

# 技术要求

本章中的练习可以使用 Visual Studio 2012 或更高版本以及.NET Framework 2.0 或更高版本进行练习。然而，从 7.0 版本开始的所有新的 C#功能都需要您安装 Visual Studio 2017。

如果您没有上述任何产品的许可证，您可以下载 Visual Studio 2017 的社区版，网址为：[`visualstudio.microsoft.com/downloads/`](https://visualstudio.microsoft.com/downloads/)。

本章的示例代码可以在 GitHub 上找到：[`github.com/PacktPublishing/Programming-in-C-Sharp-Exam-70-483-MCSD-Guide`](https://github.com/PacktPublishing/Programming-in-C-Sharp-Exam-70-483-MCSD-Guide)。

# 理解代理

**代理**实际上只是一个方法引用，包括一些参数和一个返回类型。当定义代理时，它可以与任何提供兼容签名和方法返回类型的实例相关联。换句话说，代理在 C 和 C++中可以定义为函数指针，但代理是类型安全、安全且面向对象的。

代理模型遵循观察者模式，允许订阅者注册并从提供者接收通知。为了更好地理解观察者模式，请参阅本章末尾的“进一步阅读”部分提供的参考资料。

代理的一个经典例子是 Windows 应用程序中的事件处理器，这些是代理调用的方法。在事件处理的上下文中，代理是事件源和处理事件代码之间的中介。

由于代理能够将方法作为参数传递，因此它们非常适合回调。代理是从`System.Delegate`类派生出来的。

`delegate`的一般语法如下：

```cs
delegate <return type> <delegate name> <parameter list>
```

代理声明的示例如下：

```cs
public delegate string delegateexample (string strVariable);
```

在前面的例子中，定义的委托可以被任何具有单个字符串参数并返回字符串变量的方法引用。

# 实例化一个委托

当我们使用 C# 2.0 之前的版本时，可以使用命名方法。2.0 版本引入了一种新的实例化委托的方法。我们将在接下来的章节中尝试理解这些方法。C# 3.0 版本用 Lambda 表达式替换了匿名方法，Lambda 表达式现在被广泛使用。

# 使用命名方法初始化委托

让我们看看`NamedMethod`的一个例子，以便我们了解如何初始化一个`delegate`。这是在 C# 2.0 之前使用的方法：

```cs
delegate void MathDelegate(int i, double j);
public class Chapter5Samples
{
  // Declare a delegate
  public void NamedMethod()
  {
    Chapter5Samples m = new Chapter5Samples();
    // Delegate instantiation using "Multiply"
    MathDelegate d = m.Multiply;
    // Invoke the delegate object.
    Console.WriteLine("Invoking the delegate using 'Multiply':");
    for (int i = 1; i <= 5; i++)
    {
      d(i, 5);
    }
    Console.WriteLine("");

  }
  // Declare the associated method.
  void Multiply(int m, double n)
  {
    System.Console.Write(m * n + " ");
  }
}
//Output:
Invoking the delegate using 'Multiply':
5 10 15 20 25
```

在前面的代码中，首先我们定义了一个名为`MathDelegate`的委托，它接受`2`个参数，`1`个整数和另一个双精度类型。然后，我们定义了一个类，我们想在其中使用一个名为`Multiply`的命名方法来调用`MathDelegate`。

`MathDelegate d = m.Multiply;`这一行是将一个命名方法赋值给委托。

命名方法委托可以封装任何可访问的类或结构，这些类或结构与委托类型匹配，从而允许开发者扩展这些方法。

在以下示例中，我们将看到如何将委托映射到静态和实例方法。将以下方法添加到我们之前创建的`Chapter5Samples`类中：

```cs
public void InvokeDelegate()
{
  HelperClass helper = new HelperClass();

  // Instance method mapped to delegate:
  SampleDelegate d = helper.InstanceMethod;
  d();

  // Map to the static method:
  d = HelperClass.StaticMethod;
  d();
}

//Create a new Helper class to hold two methods
// Delegate declaration
delegate void SampleDelegate();

internal class HelperClass
{
  public void InstanceMethod()
  {
    System.Console.WriteLine("Instance Method Invoked.");
  }

  static public void StaticMethod()
  {
    System.Console.WriteLine("Invoked function Static Method.");
  }
}

//Output: 
Invoked function Instance Method.
Invoked function Static Method.
```

在前面的代码中，我们定义了两个方法：第一个是一个普通方法，而第二个是一个静态方法。在调用使用命名方法的委托的情况下，我们可以使用第一个普通方法或第二个静态方法：

+   `SampleDelegate d = helper.InstanceMethod;`: 这是一个普通方法。

+   `d = HelperClass.StaticMethod;`: 这是一个静态方法。

# 使用匿名函数初始化委托

在创建新方法可能被视为开销的情况下，C# 允许我们初始化一个委托并指定一个代码块。当委托被调用时，它将处理这个代码块。这是 C# 2.0 中用于调用委托的方法。它们也被称为匿名方法。

一个内联定义的表达式或语句，而不是委托类型，被称为匿名函数。

有两种匿名函数：

+   Lambda 表达式

+   匿名方法

我们将在接下来的小节中查看这两种类型的函数。然而，在继续之前，我们还应该了解一个新运算符，称为**Lambda 运算符**。这个运算符用于表示 Lambda 表达式。

# Lambda 表达式

从 C# 3.0 开始，引入了 Lambda 表达式，并且在调用委托时被广泛使用。Lambda 表达式是通过 Lambda 运算符创建的。在运算符的左侧，我们指定输入参数，而在右侧，我们指定表达式或代码块。当 Lambda 运算符在表达式体中使用时，它将成员的名称与其实现分开。

Lambda 操作符表示为 `=>` 符号。这个操作符是右结合的，并且与赋值操作符具有相同的优先级。赋值操作符将右侧操作数的值赋给左侧操作数。

在下面的代码中，我们使用 Lambda 操作符来比较字符串数组中的一个特定单词并返回它。在这里，我们将 Lambda 表达式应用于 `words` 数组的每个元素：

```cs
words.Where(w => w.Equals("apple")).FirstOrDefault();
```

此示例还展示了我们如何使用 LINQ 查询来获得相同的结果。

我们尝试使用 LINQ 查询从一个单词数组中找到 "apple"。任何可枚举集合都允许我们使用 LINQ 进行查询，并返回所需的结果：

```cs
public void LambdaOperatorExample()
{
    string[] words = { "bottle", "jar", "drum" };
    // apply Lambda expression to each element in the array
    string searchedWord = words.Where(w => 
                            w.Equals("drum")).FirstOrDefault();
    Console.WriteLine(searchedWord);
    // Get the length of each word in the array.
    var query = from w in words
                where w.Equals("drum")
                select w;

    string search2 = query.FirstOrDefault();
    Console.WriteLine(search2);
}

//Output:
drum
drum
```

Lambda 表达式是 Lambda 操作符的右侧操作符，并且在表达式树中得到了广泛的应用。

更多关于表达式树的信息可以在 Microsoft 文档网站上找到。

这个 Lambda 表达式必须是一个有效的表达式。如果成员类型是 void，它会被归类为语句表达式。

从 C# 6 开始，这些表达式支持方法和属性获取语句，而从 C# 7 开始，这些表达式支持构造函数、析构函数、属性设置语句和索引器。

在下面的代码中，我们使用表达式来编写变量的第一个名字和最后一个名字，并且我们还使用了 `Trim()` 函数：

```cs
public override string ToString() => $"{fname} {lname}".Trim();
```

在对 Lambda 表达式和 Lambda 操作符有了基本理解之后，我们可以继续探讨如何使用 Lambda 表达式来调用委托。

回想一下，Lambda 表达式可以表示如下：

```cs
Input-Parameters => Expression
```

在下面的示例中，我们向现有方法中添加了两行代码来使用 Lambda 表达式调用委托。`X` 是输入参数，其中 `X` 的类型由编译器确定：

```cs
delegate void StringDelegate(string strVariable);
public void InvokeDelegatebyAnonymousFunction()
{
  //Named Method
  StringDelegate StringDel = HelperClass.StringMethod;
  StringDel("Chapter 5 - Named Method");

  //Anonymous method
  StringDelegate StringDelB = delegate (string s) { Console.WriteLine(s); };
  StringDelB("Chapter 5- Anonymous method invocation");

  //LambdaExpression
  StringDelegate StringDelC = (X)=> { Console.WriteLine(X); };
  StringDelB("Chapter 5- Lambda Expression invocation");

}

//Output:
Chapter 5 - Named Method
Chapter 5- Anonymous method invocation
Chapter 5- Lambda Expression invocation
```

# 匿名方法

C# 2.0 引入了匿名方法，而 C# 3.0 引入了 Lambda 表达式，后来 Lambda 表达式被匿名方法所取代。

在使用 Lambda 表达式时，匿名方法提供了一些使用 Lambda 表达式时无法实现的功能，例如，它们允许我们避免参数。这些允许匿名方法转换为具有不同签名的委托。

让我们看看如何使用匿名方法来初始化委托的示例：

```cs
public void InvokeDelegatebyAnonymousFunction()
{
  //Named Method
  StringDelegate StringDel = HelperClass.StringMethod;
  StringDel("Chapter 5");

  //Anonymous method
  StringDelegate StringDelB = delegate (string s) { Console.WriteLine(s); };
  StringDelB("Chapter 5- Anonymous method invocation");

}
internal class HelperClass
{
  public void InstanceMethod()
  {
    System.Console.WriteLine("Instance method Invoked.");
  }

  public static void StaticMethod()
  {
    System.Console.WriteLine("Invoked function Static Method.");
  }

  public static void StringMethod(string s)
  {
    Console.WriteLine(s);
  }
}

//Output:
Chapter 5
Chapter 5- Anonymous method invocation
```

在前面的代码中，我们定义了一个字符串委托并编写了一些内联代码来调用它。以下是我们定义内联委托（也称为匿名方法）的代码：

```cs
StringDelegate StringDelB = delegate (string s) { Console.WriteLine(s); };
```

以下代码展示了如何创建匿名方法：

```cs
// Creating a handler for a click event.
sampleButton.Click += delegate(System.Object o, System.EventArgs e)
                   { System.Windows.Forms.MessageBox.Show(
                     "Sample Button Clicked!"); };
```

在这里，我们创建了一个代码块并将其作为 `delegate` 参数传递。

如果在代码块内部遇到任何跳转语句（如 `goto`、`break` 或 `continue`），并且目标在代码块外部，匿名方法将抛出错误。此外，在跳转语句在代码块外部且目标在内部的情况下，使用 `int` 匿名方法将抛出异常。

任何在委托作用域之外创建并包含在匿名方法声明中的局部变量被称为匿名方法的*外部变量*。例如，在下面的代码段中，`n`是一个外部变量：

```cs
int n = 0;
Del d = delegate() { System.Console.WriteLine("Copy #:{0}", ++n); };
```

匿名方法不允许在 is 操作符的左侧。在匿名方法中不能访问或使用不安全代码，包括外部作用域的`in`、`ref`或`out`参数。

# 委托中的方差

C#支持具有匹配方法签名的委托类型中的协变。这个特性是在.NET Framework 3.5 中引入的。这意味着委托现在可以分配具有匹配签名的委托，同时方法也可以返回派生类型。

如果一个方法的返回类型是从在委托中定义的类型派生出来的，那么它在委托中定义为协变。同样，如果一个方法具有比在委托中定义的派生参数类型更少的类型，那么它定义为逆变。

让我们通过一个例子来了解协变。为了这个例子，我们将创建几个类。

在这里，我们将创建`ParentReturnClass`、`Child1ReturnClass`和`Child2Return`类。这些类中的每一个都有一个字符串类型属性。这两个子类都是从`ParentReturnClass`继承的：

```cs
internal class ParentReturnClass
{
  public string Message { get; set; }
}

internal class Child1ReturnClass : ParentReturnClass
{
  public string ChildMessage1 { get; set; }
}
internal class Child2ReturnClass : ParentReturnClass
{
  public string ChildMessage2 { get; set; }
}
```

现在，让我们向之前定义的辅助类添加两个新方法，每个方法返回我们之前定义的相应子类：

```cs
public Child1ReturnClass ChildMehod1() 
{ 
    return new Child1ReturnClass 
    { 
        ChildMessage1 = "ChildMessage1" 
    }; 
}
public Child2ReturnClass ChildMehod2() 
{ 
    return new Child2ReturnClass 
    { 
        ChildMessage2 = "ChildMessage2" 
    }; 
}
```

现在，我们将定义一个返回`ParentReturnClass`的委托。我们还将定义一个新的方法，将为每个子方法初始化这个委托。在下面的代码中，一个重要的观察点是，我们使用了显式类型转换将`ParentReturnClass`转换为`ChildReturnClass1`和`ChildReturnClass2`：

```cs
delegate ParentReturnClass covrianceDelegate();
public void CoVarianceSample()
{
  covrianceDelegate cdel;
  cdel = new HelperClass().ChildMehod1;
  Child1ReturnClass CR1 = (Child1ReturnClass)cdel();
  Console.WriteLine(CR1.ChildMessage1);
  cdel = new HelperClass().ChildMehod2;
 Child2ReturnClass CR2 = (Child2ReturnClass)cdel();
Console.WriteLine(CR2.ChildMessage2);
}

//Output:
ChildMessage1
ChildMessage2
```

在前面的例子中，委托返回`ParentReturnClass`。然而，`ChildMethod1`和`ChildMethod2`都返回从`ParentReturnClass`继承的子类。这意味着允许返回比在委托中定义的类型更派生的类型的方法。这被称为协变。

现在，让我们再看另一个例子来了解逆变。通过添加一个接受`ParentReturnClass`作为参数并返回 void 的新方法来扩展之前创建的辅助类：

```cs
public void Method1(ParentReturnClass parentVariable1) 
{ 
    Console.WriteLine(((Child1ReturnClass)parentVariable1).ChildMessage1); 
}
```

定义一个接受`Child1ReturnClass`作为参数的委托：

```cs
delegate void contravrianceDelegate(Child1ReturnClass variable1);
```

现在，创建一个初始化委托的方法：

```cs
public void ContraVarianceSample()
{
  Child1ReturnClass CR1 = new Child1ReturnClass() { ChildMessage1 = "ChildMessage1" };
  contravrianceDelegate cdel = new HelperClass().Method1;
  cdel(CR1);

}

//Output:
ChildMessage1
```

因为第一个方法与父类一起工作，所以它肯定可以与从父类继承的类一起工作。C#允许的派生类型参数比在委托中定义的少。

# 内置委托

到目前为止，我们已经看到了如何创建自定义委托并在我们的程序中使用它们。C#自带一些内置委托，开发者可以使用它们而不是必须创建自定义委托。它们如下所示：

+   `Func`

+   `Action`

`Func`接受零个或多个参数，并以一个`out`参数返回一个值，而`Action`接受零个或多个参数但不返回任何内容。

在使用 `Func` 或 `Action` 时，不需要显式声明委托：

```cs
public delegate TResult Func<out TResult>();
```

`Action` 可以定义为以下内容：

```cs
public delegate void Action();
```

正如我们之前提到的，它们都接受零个或多个参数。C# 支持 16 种不同的委托形式，所有这些都可以在我们的程序中使用。

`Func` 具有两个或更多参数的一般形式如下。它接受逗号分隔的输入和输出参数，其中最后一个参数始终是一个名为 `TResult` 的输出参数：

```cs
public delegate TResult Func<in T1,in T2,in T3,in T4,out TResult>(T1 arg1, T2 arg2, T3 arg3, T4 arg4);
```

与 `Func` 类似，`Action` 具有两个或更多参数的一般形式如下：

```cs
public delegate void Action<in T1,in T2,in T3,in T4>(T1 arg1, T2 arg2, T3 arg3, T4 arg4);
```

# 多播委托

通过委托调用多个方法称为多播。您可以使用 `+`、`-`、`+=` 或 `-+` 来向调用方法列表中添加或删除方法。这个列表称为调用列表。它在事件处理中使用。

以下示例展示了我们如何通过调用委托来调用多个方法。我们有两个方法，它们都接受一个字符串参数并在屏幕上显示它。在多播委托方法中，我们将两个方法与 `stringdelegate` 关联：

```cs
delegate void StringDelegate( string strVariable);
public void MulticastDelegate()
{
  StringDelegate StringDel = HelperClass.StringMethod;
  StringDel += HelperClass.StringMethod2;
  StringDel("Chapter 5 - Multicast delegate Method1");
}

//Helper Class Methods
public static void StringMethod(string s)
{
  Console.WriteLine(s);
}

public static void StringMethod2(string s)
{
  Console.WriteLine("Method2 :" + s);
}

/Output:
Chapter 5 - Multicast delegate Method1
Method2 :Chapter 5 - Multicast delegate Method1
```

# 处理和引发事件

正如我们在本章引言中提到的，事件是由用户执行的动作，例如按键、鼠标移动或 I/O 操作。有时，事件可以通过系统生成的操作引发，例如在表中创建/更新记录。

.NET 框架事件基于委托模型，该模型遵循观察者模式。观察者模式允许订阅者注册通知，并允许发布者注册推送通知。这就像延迟绑定，是对象广播发生某些事情的一种方式。

允许您订阅/取消订阅来自发布者的事件流的模式称为观察者模式。

例如，在上一章中，我们处理了一个代码片段，该程序用于查找用户输入的字符是否为元音。在这里，用户按下键盘上的键是发布者，它通知程序有关按下的键。现在，我们的程序，作为提供者的订阅者，通过检查输入的字符是否为元音并显示在屏幕上对此做出响应。

对象发送的消息以通知它已发生某些操作称为事件。引发此事件的对象称为事件发送者或发布者。接收并响应事件的对象称为订阅者。

发布者事件可以有多个订阅者，而订阅者可以处理发布事件。请记住，我们之前章节中讨论的多播委托在事件（发布-订阅模式）中得到了广泛的应用。

默认情况下，如果发布者有多个订阅者，它们都会同步调用。C# 支持异步调用这些事件方法。我们将在接下来的章节中详细了解这一点。

在我们深入示例之前，让我们尝试理解我们将要使用的一些术语：

| `event` | 这是一个关键字，用于在 C#中的`publisher`类中定义事件。 |
| --- | --- |
| `EventHandler` | 此方法用于处理事件。这可能包含或不包含事件数据。 |
| `EventArgs` | 它代表包含事件数据的类的基类。 |

事件处理器支持两种变体：一种没有事件数据，另一种有事件数据。以下代码表示一个处理没有事件数据的事件的函数：

```cs
public delegate void EventHandler(object sender, EventArgs e);

```

以下代码表示一个处理带有事件数据的事件的函数：

```cs
public delegate void EventHandler<TEventArgs>(object sender, TEventArgs e);
```

让我们来看一个示例，并尝试理解我们如何引发事件并处理它们。

在这个场景中，我们将有一个银行应用程序，客户可以进行创建新账户、查看他们的信用和借记金额以及请求他们的总余额等交易。每当进行此类交易时，我们将引发事件并通知客户。

我们将从`Account`类（`publisher`类）开始，以及所有支持的方法，如`credit()`、`debit()`、`showbalance()`和`initialdeposit()`。这些都是客户可以操作其账户的交易类型。因为客户需要在每次此类交易发生时得到通知，我们将定义一个事件和一个带有事件数据的事件处理器来处理该事件：

```cs
public delegate void BankTransHandler(object sender, 
BankTransEventArgs e); // Delegate Definition 
    class Account
    {
        // Event Definition
        public event BankTransHandler ProcessTransaction; 
        public int BALAmount;
        public void SetInitialDeposit(int amount)
        {
            this.BALAmount = amount;
            BankTransEventArgs e = new BankTransEventArgs(amount, 
                                   "InitialBalance");
            // InitialBalance transaction made
            OnProcessTransaction(e);
        }
        public void Debit(int debitAmount)
        {
            if (debitAmount < BALAmount)
            {
                BALAmount = BALAmount - debitAmount;
                BankTransEventArgs e = new BankTransEventArgs(
                                           debitAmount, "Debited");
                OnProcessTransaction(e); // Debit transaction made 
            }
        }
        public void Credit(int creditAmount)
        {
            BALAmount = BALAmount + creditAmount;
            BankTransEventArgs e = new BankTransEventArgs(
                                       creditAmount, "Credited");
            OnProcessTransaction(e); // Credit transaction made
        }
        public void ShowBalance()
        {
            BankTransEventArgs e = new BankTransEventArgs(
                                       BALAmount, "Total Balance");
            OnProcessTransaction(e); // Credit transaction made
        }
        protected virtual void OnProcessTransaction(
                                       BankTransEventArgs e)
        {
            ProcessTransaction?.Invoke(this, e);
        }
    }
```

你可能已经注意到了我们在上一个示例中使用的新类，即`TrasactionEventArgs`。这个类携带事件数据。我们现在将定义这个类，它继承自`EventArgs`基类。我们将定义两个变量，`amt`和`type`，以携带变量到事件处理器：

```cs
public class BankTransEventArgs : EventArgs
    {
        private int _transactionAmount;
        private string _transactionType;
        public BankTransEventArgs(int amt, string type)
        {
            this._transactionAmount = amt;
            this._transactionType = type;
        }
        public int TransactionAmount
        {
            get
            {
                return _transactionAmount;
            }
        }
        public string TranactionType
        {
            get
            {
                return _transactionType;
            }
        }
    }
```

现在，让我们定义一个订阅者类来测试我们的事件和事件处理器是如何工作的。在这里，我们将定义一个`AlertCustomer`方法，其签名与在`publisher`类中声明的代理相匹配。将此方法的引用传递给代理，以便它对事件做出反应：

```cs
public class EventSamples
{
 private void AlertCustomer(object sender, BankTransEventArgs e)
 {
  Console.WriteLine("Your Account is {0} for Rs.{1} ", 
                     e.TranactionType, e.TransactionAmount);
 }
 public void Run()
 {
  Account bankAccount = new Account();
  bankAccount.ProcessTransaction += new 
      BankTransHandler(AlertCustomer);
  bankAccount.SetInitialDeposit(5000);
  bankAccount.ShowBalance();
  bankAccount.Credit(500);
  bankAccount.ShowBalance();
  bankAccount.Debit(500);
  bankAccount.ShowBalance();
 }
}
```

当你执行前面的程序时，对于每次进行的交易，都会引发一个交易处理器事件，该事件调用通知客户方法并在屏幕上显示发生了什么类型的交易，如下所示：

```cs
//Output:
Your Account is InitialBalance for Rs.5000
Your Account is Total Balance for Rs.5000
Your Account is Credited for Rs.500
Your Account is Total Balance for Rs.5500
Your Account is Debited for Rs.500
Your Account is Total Balance for Rs.5000
```

# 摘要

在本章中，我们学习了代理以及我们如何在程序中定义、启动和使用它们。我们了解了代理的变体、内置代理和多播代理。最后，我们在理解事件、事件处理器和`EventArgs`之前，研究了代理如何成为事件的基础。

现在，我们可以说事件封装了代理，而代理封装了方法。

在下一章中，我们将学习 C#中的多线程和异步处理。我们将理解并使用程序中的线程，了解任务、并行类、async、await 以及更多内容。

# 问题

1.  代理非常适合 ___，因为它们能够将方法作为参数传递。

    1.  多播代理

    1.  内置委托

    1.  回调

    1.  事件

1.  有哪些不同的方式来初始化委托？选择所有适用的。

    1.  匿名方法

    1.  Lambda 表达式

    1.  命名方法

    1.  所有上述选项

1.  哪个方法可以有比在委托中定义的返回类型更派生的类型？

    1.  匿名方法

    1.  协变

    1.  匿名函数

    1.  Lambda 表达式

1.  哪个内置委托接受零个或多个参数并返回 void？

    1.  `Action`

    1.  `Func`

    1.  `event`

    1.  `delegate`

1.  在 C# 事件声明中，以下哪个被使用？

    1.  `event`

    1.  `delegate`

    1.  `EventHandler`

    1.  `class`

1.  订阅者可以通知发布者关于对象发生的更改。

    1.  True

    1.  False

# 答案

1.  **回调**

1.  **所有上述选项**

1.  **协变**

1.  **Action**

1.  **event**

1.  **False**

# 进一步阅读

为了更好地理解观察者模式，请查看[`docs.microsoft.com/en-us/dotnet/standard/events/observer-design-pattern`](https://docs.microsoft.com/en-us/dotnet/standard/events/observer-design-pattern)。

以下是一篇关于声明、初始化和使用委托的好文章。那里也可以找到示例：[`docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/how-to-declare-instantiate-and-use-a-delegate`](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/how-to-declare-instantiate-and-use-a-delegate)。
