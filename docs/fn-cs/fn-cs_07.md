# 第七章。学习递归

在函数式编程的首次公告中，许多函数式语言没有循环功能来迭代序列。我们所要做的就是构建递归过程来迭代序列。尽管 C#具有诸如`for`和`while`之类的迭代功能，但最好还是在函数式方法中讨论递归。递归也将简化我们的代码。因此，在本章中，我们将讨论以下主题：

+   理解递归例程的工作方式

+   将迭代重构为递归

+   区分尾递归和累加器传递风格与续传风格

+   理解间接递归和直接递归

+   使用 Aggregate LINQ 运算符在函数式方法中应用递归

# 探索递归

递归函数是调用自身的函数。与迭代循环（例如`while`和`for`循环）一样，它用于逐步解决复杂的任务并组合结果。但是，`for`循环和`while`循环之间存在区别。迭代将持续重复直到任务完成，而递归将将任务分解成较小的部分以解决更大的问题，然后组合结果。在函数式方法中，递归更接近数学方法，因为它通常比迭代更短，尽管在设计和测试上可能更难一些。

在第一章中，*在 C#中品尝函数式风格*，我们在讨论函数式编程的概念时熟悉了递归函数。在那里，我们分析了命名为`GetFactorial()`的阶乘函数在命令式和函数式方法中的实现。为了提醒我们，以下是`GetFactorial()`函数的实现，我们可以在`SimpleRecursion.csproj`项目中找到：

```cs
public partial class Program 
{ 
  private static int GetFactorial(int intNumber) 
  { 
    if (intNumber == 0) 
    { 
      return 1; 
    } 
    return intNumber * GetFactorial(intNumber - 1); 
  } 
} 

```

在我们在第一章的讨论中，*在 C#中品尝函数式风格*，我们知道非负整数`N`的阶乘是小于或等于`N`的所有正整数的乘积。因此，假设我们有以下函数来计算五的阶乘：

```cs
private static void GetFactorialOfFive() 
{ 
  int i = GetFactorial(5); 
  Console.WriteLine("5! is {0}",i); 
} 

```

正如我们可以预测的那样，如果我们调用前面的`GetFactorialOfFive()`方法，我们将在控制台上得到以下输出：

![探索递归](img/Image00083.jpg)

回到`GetFactorial()`方法，我们可以看到在该方法的实现中有结束递归的代码，如下面的代码片段所示：

```cs
if (intNumber == 0) 
{ 
  return 1; 
} 

```

我们可以看到前面的代码是递归的基本情况，递归通常有基本情况。这个基本情况将定义递归链的结束，因为在这种情况下，每次运行递归时方法都会改变`intNumber`的状态，并且如果`intNumber`为零，链条将停止。

## 递归例程的工作方式

为了理解递归例程的工作方式，让我们检查一下程序的流程，看看如果我们找到五的阶乘时`intNumber`的状态是怎样的：

```cs
int i = GetFactorial(5) 
  (intNumber = 5) != 0 
  return (5 * GetFactorial(4)) 
    (intNumber = 4) != 0 
    return (4 * GetFactorial(3)) 
      (intNumber = 3) != 0 
      return (3 * GetFactorial(2)) 
        (intNumber = 2) != 0 
        return (2 * GetFactorial(1)) 
          (intNumber = 1) != 0 
          return (1 * GetFactorial(0)) 
            (intNumber = 0) == 0 
            return 1 
          return (1 * 1 = 1) 
        return (2 * 1 = 2) 
      return (3 * 2 = 6) 
    return (4 * 6 = 24) 
  return (5 * 24 = 120) 
i = 120 

```

使用前述流程，递归的工作方式变得更清晰。我们定义的基本情况定义了递归链的结束。编程语言编译器将特定情况的递归转换为迭代，因为基于循环的实现通过消除对函数调用的需求而变得更有效率。

### 提示

在编写程序逻辑时应谨慎应用递归。如果您错过了基本情况或给出了错误的值，可能会陷入无限递归。例如，在前面的`GetFactorial()`方法中，如果我们传递`intNumber < 0`，那么我们的程序将永远不会结束。

## 将迭代重构为递归

递归使我们的程序更易读，并且在函数式编程方法中是必不可少的。在这里，我们将把 for 循环迭代重构为递归方法。让我们来看看以下代码，我们可以在`RefactoringIterationToRecursion.csproj`项目中找到：

```cs
public partial class Program 
{ 
  public static int FindMaxIteration( 
     int[] intArray) 
  { 
    int iMax = 0; 
    for (int i = 0; i < intArray.Length; i++) 
    { 
      if (intArray[i] > iMax) 
      { 
        iMax = intArray[i]; 
      } 
    } 
    return iMax; 
  } 
} 

```

上述的`FindMaxIteration()`方法用于选择数组中的最大数。考虑到我们有以下代码来运行`FindMaxIteration()`方法：

```cs
public partial class Program 
{ 
  static void Main(string[] args) 
  { 
    int[] intDataArray =  
       {8, 10, 24, -1, 98, 47, -101, 39 }; 
    int iMaxNumber = FindMaxIteration(intDataArray); 
    Console.WriteLine( 
       "Max Number (using FindMaxRecursive) = " + 
         iMaxNumber); 
  } 
} 

```

正如我们所期望的，我们将在控制台窗口中得到以下输出：

![将迭代重构为递归](img/Image00084.jpg)

现在，让我们将`FindMaxIteration()`方法重构为递归函数。以下是`FindMaxRecursive()`方法的实现，它是`FindMaxIteration()`方法的递归版本：

```cs
public partial class Program 
{ 
  public static int FindMaxRecursive( 
     int[] intArray,  
      int iStartIndex = 0) 
  { 
    if (iStartIndex == intArray.Length - 1) 
    { 
      return intArray[iStartIndex]; 
    } 
    else 
    { 
      return Math.Max(intArray[iStartIndex],
        FindMaxRecursive(intArray,iStartIndex + 1)); 
    } 
  } 
} 

```

我们可以使用与`FindMaxIteration()`方法相同的代码来调用上述的`FindMaxRecursive()`方法，如下所示：

```cs
public partial class Program 
{ 
  static void Main(string[] args) 
  { 
    int[] intDataArray = {8, 10, 24, -1, 98, 47, -101, 39 }; 
    int iMaxNumber = FindMaxRecursive(intDataArray); 
    Console.WriteLine"Max Number(using FindMaxRecursive) = " +
        iMaxNumber); 
  } 
} 

```

正如我们在上面的方法中所看到的，我们有以下基本情况来定义递归链的结束：

```cs
if (iStartIndex == intArray.Length - 1) 
{ 
  return intArray[iStartIndex]; 
} 

```

如果我们运行上述代码，我们将得到与之前方法中得到的相同结果，如下面的控制台截图所示：

![将迭代重构为递归](img/Image00085.jpg)

现在，让我们来看一下以下流程，了解我们如何在使用递归函数时得到这个结果：

```cs
Array = { 8, 10, 24, -1, 98, 47, -101, 39 }; 
Array.Length - 1 = 7 
int iMaxNumber = FindMaxRecursive(Array, 0) 
  (iStartIndex = 0) != 7 
  return Max(8, FindMaxRecursive(Array, 1)) 
    (iStartIndex = 1) != 7 
    return Max(10, FindMaxRecursive(Array, 2)) 
      (iStartIndex = 2) != 7 
      return Max(24, FindMaxRecursive(Array, 3)) 
        (iStartIndex = 3) != 7 
        return Max(-1, FindMaxRecursive(Array, 4)) 
          (iStartIndex = 4) != 7 
           return Max(98, FindMaxRecursive(Array, 5)) 
            (iStartIndex = 5) != 7 
            return Max(47, FindMaxRecursive(Array, 6)) 
              (iStartIndex = 6) != 7 
              return Max(-101, FindMaxRecursive(Array, 7)) 
                (iStartIndex = 7) == 7 
                return 39 
              return Max(-101, 39) = 39 
            return Max(47, 39) = 47 
          return Max(98, 47) = 98 
        return Max(-1, 98) = 98 
      return Max(24, 98) = 98 
    return Max(10, 98) = 98 
  return Max(8, 98) = 98 
iMaxNumber = 98 

```

使用上述流程，我们可以区分每次调用`FindMaxRecursive()`方法时得到的最大数的每个状态变化。然后，我们可以证明给定数组中的最大数是`98`。

# 使用尾递归

在我们之前讨论的`GetFactorial()`方法中，使用传统递归来计算阶乘数。这种递归模型首先执行递归调用并返回值，然后计算结果。使用这种递归模型，我们在递归调用完成之前不会得到结果。

除了传统的递归模型，我们还有另一种称为尾递归的递归。尾调用成为函数中的最后一件事，并且在递归之后不执行任何操作。让我们来看看以下代码，我们可以在`TailRecursion.csproj`项目中找到：

```cs
public partial class Program 
{ 
  public static void TailCall(int iTotalRecursion) 
  { 
    Console.WriteLine("Value: " + iTotalRecursion); 
    if (iTotalRecursion == 0) 
    { 
      Console.WriteLine("The tail is executed"); 
      return; 
    } 
    TailCall(iTotalRecursion - 1); 
  } 
} 

```

从上面的代码中，当`iTotalRecursion`达到`0`时，尾部被执行，如下面的代码片段所示：

```cs
if (iTotalRecursion == 0) 
{ 
  Console.WriteLine("The tail is executed"); 
  return; 
} 

```

如果我们运行上述的`TailCall()`方法，并为`iTotalRecursion`参数传递`5`，我们将在控制台上得到以下输出：

![使用尾递归](img/Image00086.jpg)

现在，让我们来看看在这段代码中每次递归调用时状态的变化：

```cs
TailCall(5) 
  (iTotalRecursion = 5) != 0 
  TailCall(4) 
    (iTotalRecursion = 4) != 0 
    TailCall(3) 
      iTotalRecursion = 3) != 0 
      TailCall(2) 
        iTotalRecursion = 2) != 0 
        TailCall(1) 
          iTotalRecursion = 1) != 0 
          TailCall(0) 
            iTotalRecursion = 0) == 0 
            Execute the process in tail 
        TailCall(1) => nothing happens 
      TailCall(2) => nothing happens 
    TailCall(3) => nothing happens 
  TailCall(4) => nothing happens 
TailCall(5) => nothing happens 

```

从递归的流程中，该过程仅在最后的递归调用中运行。之后，其他递归调用不会发生任何事情。换句话说，我们可以得出以下流程：

```cs
TailCall(5) 
   (iTotalRecursion = 5) != 0 
  TailCall(4) 
    (iTotalRecursion = 4) != 0 
    TailCall(3) 
      iTotalRecursion = 3) != 0 
      TailCall(2) 
        iTotalRecursion = 2) != 0 
        TailCall(1) 
          iTotalRecursion = 1) != 0 
          TailCall(0) 
            iTotalRecursion = 0) == 0 
            Execute the process in tail 
Finish! 

```

现在，我们的尾递归流程显而易见。尾递归的思想是尽量减少堆栈的使用，因为堆栈有时是我们拥有的昂贵资源。使用尾递归，代码不需要记住上次返回时必须返回的状态，因为在这种情况下，它在累加器参数中有临时结果。接下来的主题是遵循尾递归的两种风格；它们是**累加器传递风格**（**APS**）和**续传风格**（**CPS**）。

## 累加器传递风格

在**累加器传递风格**（**APS**）中，递归首先执行计算，执行递归调用，然后将当前步骤的结果传递给下一个递归步骤。让我们来看看我们从`GetFactorial()`方法重构的尾递归代码的累加器传递风格，我们可以在`AccumulatorPassingStyle.csproj`项目中找到：

```cs
public partial class Program 
{ 
  public static int GetFactorialAPS(int intNumber, 
    int accumulator = 1) 
  { 
    if (intNumber == 0) 
    { 
      return accumulator; 
    } 
    return GetFactorialAPS(intNumber - 1, 
       intNumber * accumulator); 
  } 
} 

```

与`GetFactorial()`方法相比，`GetFactorialAPS()`方法现在有一个名为 accumulator 的第二个参数。由于阶乘`0`的结果是`1`，我们将默认值 1 赋给 accumulator 参数。现在它不仅返回一个值，而且每次调用递归函数时都返回阶乘的计算结果。为了证明这一点，考虑我们有以下代码来调用`GetFactorialAPS()`方法：

```cs
public partial class Program 
{ 
  private static void GetFactorialOfFiveUsingAPS() 
  { 
    int i = GetFactorialAPS(5); 
    Console.WriteLine( 
       "5! (using GetFactorialAPS) is {0}",i); 
  } 
} 

```

如果我们运行上述方法，我们将在控制台上得到以下输出：

![累加器传递风格](img/Image00087.jpg)

现在，让我们检查`GetFactorialAPS()`方法的每个调用，以查看程序的以下流程中方法内部的状态变化：

```cs
int i = GetFactorialAPS(5, 1) 
  accumulator = 1 
  (intNumber = 5) != 0 
  return GetFactorialAPS(4, 5 * 1) 
    accumulator = 5 * 1 = 5 
    (intNumber = 4) != 0 
    return GetFactorialAPS(3, 4 * 5) 
      accumulator = 4 * 5 = 20 
      (intNumber = 3) != 0 
      return GetFactorialAPS(2, 3 * 20) 
        accumulator = 3 * 20 = 60 
        (intNumber = 2) != 0 
        return GetFactorialAPS(1, 2 * 60) 
          accumulator = 2 * 60 = 120 
          (intNumber = 1) != 0 
          return GetFactorialAPS(0, 1 * 120) 
            accumulator = 1 * 120 = 120 
            (intNumber = 0) == 0 
            return accumulator 
          return 120 
        return 120 
      return 120 
    return 120 
  return 120 
i = 120 

```

从上述流程中可以看出，由于每次调用时都执行计算，我们现在在函数的最后一次调用中得到了计算的结果，当`intNumber`参数达到`0`时，如下面的代码片段所示：

```cs
return GetFactorialTailRecursion(0, 1 * 120) 
  accumulator = 1 * 120 = 120 
  (intNumber = 0) == 0 
  return accumulator 
return 120 

```

我们还可以将上述的`GetFactorialAPS()`方法重构为`GetFactorialAPS2()`方法，以便不返回任何值，这样尾递归的 APS 将变得更明显。代码将如下所示：

```cs
public partial class Program 
{ 
  public static void GetFactorialAPS2( 
      int intNumber,int accumulator = 1) 
  { 
    if (intNumber == 0) 
    { 
      Console.WriteLine("The result is " + accumulator); 
      return; 
    } 
    GetFactorialAPS2(intNumber - 1, intNumber * accumulator); 
  } 
} 

```

假设我们有以下`GetFactorialOfFiveUsingAPS2()`方法来调用`GetFactorialAPS2()`方法：

```cs
public partial class Program 
{ 
  private static void GetFactorialOfFiveUsingAPS2() 
  { 
    Console.WriteLine("5! (using GetFactorialAPS2)"); 
    GetFactorialAPS2(5); 
  } 
} 

```

因此，如果我们调用上述的`GetFactorialOfFiveUsingAPS2()`方法，我们将在控制台上得到以下输出：

![累加器传递风格](img/Image00088.jpg)

现在，`GetFactorialAPS2()`方法的流程变得更清晰，如下面的程序流程所示：

```cs
GetFactorialAPS2(5, 1) 
  accumulator = 1 
  (intNumber = 5) != 0 
  GetFactorialAPS2(4, 5 * 1) 
    accumulator = 5 * 1 = 5 
    (intNumber = 4) != 0 
    GetFactorialAPS2(3, 4 * 5) 
      accumulator = 4 * 5 = 20 
      (intNumber = 3) != 0 
      GetFactorialAPS2(2, 3 * 20) 
        accumulator = 3 * 20 = 60 
        (intNumber = 2) != 0 
        GetFactorialAPS2(1, 2 * 60) 
          accumulator = 2 * 60 = 120 
          (intNumber = 1) != 0 
          GetFactorialAPS2(0, 1 * 120) 
            accumulator = 1 * 120 = 120 
            (intNumber = 0) == 0 
            Show the accumulator value 
Finish! 

```

从上述流程中，我们可以看到每次调用`GetFactorialAPS2()`方法时都会计算 accumulator。这种递归类型的结果是，我们不再需要使用堆栈，因为函数在调用自身时不需要记住其起始位置。

## 继续传递风格

**继续传递风格**（**CPS**）与 APS 具有相同的目的，即使用尾调用实现递归函数，但在处理操作时具有显式的继续。CPS 函数的返回值将传递给继续函数。

现在，让我们将`GetFactorial()`方法重构为以下`GetFactorialCPS()`方法，我们可以在`ContinuationPassingStyle.csproj`项目中找到它：

```cs
public partial class Program 
{ 
  public static void GetFactorialCPS(int intNumber, Action<int> 
         actCont) 
  { 
    if (intNumber == 0) 
      actCont(1); 
    else 
      GetFactorialCPS(intNumber - 1,x => actCont(intNumber * x)); 
  } 
} 

```

正如我们所看到的，与`GetFactorialAPS()`方法中使用 accumulator 不同，我们现在使用`Action<T>`来委托一个匿名方法，这个方法作为继续使用。假设我们有以下代码来调用`GetFactorialCPS()`方法：

```cs
public partial class Program 
{ 
  private static void GetFactorialOfFiveUsingCPS() 
  { 
    Console.Write("5! (using GetFactorialCPS) is "); 
    GetFactorialCPS(5,  x => Console.WriteLine(x)); 
  } 
} 

```

如果我们运行上述的`GetFactorialOfFiveUsingCPS()`方法，我们将在控制台上得到以下输出：

![继续传递风格](img/Image00089.jpg)

实际上，与`GetFactorial()`方法或`GetFactorialAPS2()`方法相比，我们得到了相同的结果。然而，递归的流程现在变得有点不同，如下面的解释所示：

```cs
GetFactorialCPS(5, Console.WriteLine(x)) 
  (intNumber = 5) != 0 
  GetFactorialCPS(4, (5 * x)) 
    (intNumber = 4) != 0 
    GetFactorialCPS(3, (4 * x)) 
      (intNumber = 3) != 0 
      GetFactorialCPS(2, (3 * x)) 
        (intNumber = 2) != 0 
        GetFactorialCPS(1, (2 * x)) 
          (intNumber = 1) != 0 
          GetFactorialCPS(0, (1 * x)) 
            (intNumber = 0) != 0 
            GetFactorialCPS(0, (1 * 1)) 
          (1 * 1 = 1) 
        (2 * 1 = 2) 
      (3 * 2 = 6) 
    (4 * 6 = 24) 
  (5 * 24 = 120) 
Console.WriteLine(120) 

```

现在，每次递归的返回值都传递给继续过程，即`Console.WriteLine()`函数。

## 间接递归比直接递归

我们之前讨论过递归方法。实际上，在我们之前的讨论中，我们应用了直接递归，因为我们只处理了一个单一的方法，并且一遍又一遍地调用它，直到基本情况被执行。然而，还有另一种递归类型，称为间接递归。间接递归涉及至少两个函数，例如函数 A 和函数 B。在间接递归的应用中，函数 A 调用函数 B，然后函数 B 再次调用函数 A。这被认为是递归，因为当方法 B 调用方法 A 时，函数 A 实际上是活动的，当它再次调用函数 B 时。换句话说，当函数 B 再次调用函数 A 时，函数 A 的调用尚未完成。让我们来看看下面的代码，它演示了我们可以在`IndirectRecursion.csproj`项目中找到的间接递归：

```cs
public partial class Program 
{ 
  private static bool IsOdd(int targetNumber) 
  { 
    if (targetNumber == 0) 
    { 
      return false; 
    } 
    else 
    { 
      return IsEven(targetNumber - 1); 
    } 
  } 
  private static bool IsEven(int targetNumber) 
  { 
    if (targetNumber == 0) 
    { 
      return true; 
    } 
    else 
    { 
      return IsOdd(targetNumber - 1); 
    } 
  } 
} 

```

在上面的代码中，我们有两个函数：`IsOdd()`和`IsEven()`。每个函数在比较结果为`false`时都会调用另一个函数。当`targetNumber`不为零时，`IsOdd()`函数将调用`IsEven()`，`IsEven()`函数也是如此。每个函数的逻辑都很简单。例如，`IsOdd()`方法通过调查前一个数字`targetNumber - 1`是否为偶数来决定`targetNumber`是否为奇数。同样，`IsEven()`方法通过调查前一个数字是否为奇数来决定`targetNumber`是否为偶数。它们都将`targetNumber`减一，直到它变为零，由于零是一个偶数，现在很容易确定`targetNumber`是奇数还是偶数。现在，我们添加以下代码来检查数字`5`是偶数还是奇数：

```cs
public partial class Program 
{ 
  private static void CheckNumberFive() 
  { 
    Console.WriteLine("Is 5 even number? {0}", IsEven(5)); 
  } 
} 

```

如果我们运行上述的`CheckNumberFive()`方法，将在控制台上得到以下输出：

![间接递归与直接递归](img/Image00090.jpg)

现在，为了更清楚地理解，让我们来看看涉及`IsOdd()`和`IsEven()`方法的以下间接递归流程：

```cs
IsEven(5) 
  (targetNumber = 5) != 0 
  IsOdd(4) 
    (targetNumber = 4) != 0 
    IsEven(3) 
      (targetNumber = 3) != 0 
      IsOdd(2) 
        (targetNumber = 2) != 0 
        IsEven(1) 
          (targetNumber = 1) != 0 
            IsOdd(0) 
            (targetNumber = 0) == 0 
              Result = False 

```

从上面的流程中，我们可以看到，当我们检查数字 5 是偶数还是奇数时，我们向下移动到数字 4 并检查它是否为奇数。然后我们检查数字 3，依此类推，直到我们达到 0。通过达到 0，我们可以很容易地确定它是奇数还是偶数。

# 使用 LINQ Aggregate 进行函数式递归

当我们处理阶乘公式时，我们可以使用 LINQ Aggregate 将我们的递归函数重构为函数式方法。LINQ Aggregate 将累积给定的序列，然后我们将从累加器中得到递归的结果。在第一章中，我们已经进行了这种重构。让我们借用该章节的代码来分析`Aggregate`方法的使用。下面的代码将使用`Aggregate`方法，我们可以在`RecursionUsingAggregate.csproj`项目中找到：

```cs
public partial class Program 
{ 
  private static void GetFactorialAggregate(int intNumber) 
  { 
    IEnumerable<int> ints =  
       Enumerable.Range(1, intNumber); 
    int factorialNumber =  
       ints.Aggregate((f, s) => f * s); 
    Console.WriteLine("{0}! (using Aggregate) is {1}",
       intNumber, factorialNumber); 
  } 
} 

```

如果我们运行上述的`GetFactorialAggregate()`方法，并将`5`作为参数传递，将在控制台上得到以下输出：

![使用 LINQ Aggregate 进行函数式递归](img/Image00091.jpg)

正如我们在上面的控制台截图中所看到的，与非累积递归相比，我们得到了完全相同的结果。

## 深入研究 Aggregate 方法

正如我们之前讨论的，`Aggregate`方法将累积给定的序列。让我们来看看下面的代码，我们可以在`AggregateExample.csproj`项目文件中找到，以演示`Aggregate`方法的工作原理：

```cs
public partial class Program 
{ 
  private static void AggregateInt() 
  { 
    List<int> listInt = new List<int>() { 1, 2, 3, 4, 5, 6 }; 
    int addition = listInt.Aggregate( 
       (sum, i) => sum + i); 
    Console.WriteLine("The sum of listInt is " + addition); 
  } 
} 

```

从上面的代码中，我们可以看到我们有一个`int`数据类型的列表，其中包含从 1 到 6 的数字。然后我们调用`Aggregate`方法来求和`listInt`的成员。以下是上述代码的流程：

```cs
(sum, i) => sum + i 
sum = 1 
sum = 1 + 2 
sum = 3 + 3 
sum = 6 + 4 
sum = 10 + 5 
sum = 15 + 6 
sum = 21 
addition = sum 

```

如果我们运行上述的`AggregateInt()`方法，将在控制台上得到以下输出：

![深入研究 Aggregate 方法](img/Image00092.jpg)

实际上，`Aggregate`方法不仅可以添加数字，还可以添加字符串。让我们来看下面的代码，演示了使用`Aggregate`方法来添加字符串序列：

```cs
public partial class Program 
{ 
  private static void AggregateString() 
  { 
    List<string> listString = new List<string>()
      {"The", "quick", "brown", "fox", "jumps", "over",
              "the", "lazy", "dog"};
    string stringAggregate = listString.Aggregate((strAll, str) => 
              strAll + " " + str); 
    Console.WriteLine(stringAggregate); 
  } 
} 

```

如果我们运行前面的`AggregateString()`方法，我们将在控制台上得到以下输出：

深入研究 Aggregate 方法

以下是我们可以在 MSDN 中找到的`Aggregate`方法的声明：

```cs
public static TSource Aggregate<TSource>( 
  this IEnumerable<TSource> source, 
  Func<TSource, TSource, TSource> func 
) 

```

以下是基于先前声明的`AggregateUsage()`方法的流程：

```cs
(strAll, str) => strAll + " " + str 
strAll = "The" 
strAll = strAll + " " + str 
strAll = "The" + " " + "quick" 
strAll = "The quick" + " " + "brown" 
strAll = "The quick brown" + " " + "fox" 
strAll = "The quick brown fox" + " " + "jumps" 
strAll = "The quick brown fox jumps" + " " + "over" 
strAll = "The quick brown fox jumps over" + " " + "the" 
strAll = "The quick brown fox jumps over the" + " " + "lazy" 
strAll = "The quick brown fox jumps over the lazy" + " " + "dog" 
strAll = "The quick brown fox jumps over the lazy dog" 
stringAggregate = str 

```

从前面的流程中，我们可以使用`Aggregate`方法连接`listString`中的所有字符串。这证明不仅可以处理`int`数据类型，还可以处理字符串数据类型。

# 摘要

虽然 C#有一个使用`for`或`while`循环迭代序列的功能，但最好我们使用递归来迭代序列来接触函数式编程。我们已经讨论了递归例程的工作原理，并将迭代重构为递归。我们知道在递归中，我们有一个将定义递归链结束的基本情况。

在传统的递归模型中，递归调用首先执行，然后返回值，然后计算结果。结果直到递归调用完成后才会显示。而尾递归在递归之后根本不做任何事情。尾递归有两种风格；它们是 APS 和 CPS。

除了直接递归，我们还讨论了间接递归。间接递归涉及至少两个函数。然后，我们将递归应用到使用 Aggregrate LINQ 运算符的函数方法中。我们还深入研究了 Aggregate 运算符以及它的工作原理。

在下一章中，我们将讨论优化技术，使我们的代码更加高效。我们将使用懒惰思维，这样代码将在完美的时间执行，还将使用缓存技术，这样代码不需要每次都执行。
