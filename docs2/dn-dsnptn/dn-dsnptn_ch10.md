# 第十章. 使用对象/函数式编程实现模式

大多数现代编程语言（部分或全部）现在都支持**函数式编程**（**FP**）结构。正如前几章所述，多核计算的兴起是这种渐进式演化的一个因素。在某些情况下，我们可以使用面向对象编程（OOP）来编码解决方案，同时也可以有解决方案的函数式版本。最实用的 FP 结构使用方法是通过巧妙地将它们与 OOP 代码混合。这也被称为对象/函数式编程，并且正在成为 F#、Scala、Ruby 等语言中的主流范式。C#编程语言也不例外。有些程序员滥用 FP 结构来让自己看起来更现代，这往往会导致代码难以阅读。编程作为一种社会活动（除了其智力吸引力外），代码的可读性与其优雅性和性能一样重要。在本章中，我们将通过在 GoF 模式的应用中应用它来更深入地探讨这种流行的范式。

本章将涵盖以下主题：

+   使用 FP/OOP 实现的策略模式

+   使用 FP/OOP 重新审视迭代器模式

+   MapReduce 编程惯用用法

+   使用 FP/OOP 对模板方法模式的新颖应用

# 使用 FP/OOP 实现的策略模式

为了专注于编程模型方面，让我们编写一个冒泡排序例程来对`int`、`double`或`float`类型的数组进行排序。在排序例程中，我们需要比较相邻元素以决定是否应该交换元素的位置。在计算机科学文献中，这被称为**比较器**。由于我们正在使用泛型编程技术，我们可以编写一个泛型比较器接口来模拟在排序过程中需要执行的比较操作，并且我们将应用策略模式来提供基于类型的比较器。

```cs
    interface IComparatorStrategy<T> 
    { int Execute(T a, T b); } 

```

尽管我们可以使用单个泛型实现来比较元素，但在现实生活中，我们可能需要具体类，这些类针对特定类型。我们将实现两个具体类来比较整数和双精度浮点数。

```cs
    class IntComparator : IComparatorStrategy<int> { 
      public int Execute(int a, int b) { 
        return a > b ? 1 : (b > a ) ? -1 : 0; 
      } 
    } 

    class DoubleComparator : IComparatorStrategy<double> { 
      public int Execute(double a, double b){ 
        return a > b ? 1 : (b > a) ? -1 : 0; 
      } 
    } 

```

借助比较类，我们可以通过参数化比较类来编写一个泛型排序例程，如下所示：

```cs
    private static void BSort<T>(this T[] arr,  
    IComparatorStrategy<T> test) where T : struct 
    { 
      int n = arr.Length; 
      for(int i = 0; i<n; ++i) 
      for (int j = 0; j < n-i-1; j++) 
      if (test.Execute(arr[j],arr[j + 1]) > 0){ 
        T temp = arr[j]; arr[j] = arr[j + 1]; 
        arr[j + 1] = temp; 
      } 
    } 

```

以下代码片段展示了如何调用冒泡排序例程来对整数进行排序：

```cs
    int [] s= {  -19,20,41, 23, -6}; 
    s.BSort(new IntComparator()); 
    foreach( var n in s )  
    Console.WriteLine(n); 

```

接下来给出排序双精度浮点数的等效版本：

```cs
    double[] s2 = { -19.3, 20.5, 41.0, 23.6, -6.0 }; 
    s2.BSort(new DoubleComparator()); 
    foreach (var n in s2) 
    Console.WriteLine(n); 

```

OOP 版本的代码简单直观。但我们需要求助于接口声明及其实现来编写我们的代码。

### 注意

在大多数情况下，比较逻辑可以通过 lambda 在调用位置给出。

## 使用 FP 结构使代码更优

Lambda 函数可以用来以更简洁、更直观的方式编写比较策略例程。我们可以利用 C#函数类型`Func<TIn,TResult>`构造来做到这一点。让我们重写排序例程以利用这个习惯用法。我们还可以通过扩展方法将这种排序能力赋予数组类型。

```cs
    private static void BSort2<T>(this T[] arr,  
    Func<T,T,int> test) where T : struct 
    { 
      int n = arr.Length; 
      for (int i = 0; i < n; ++i) 
      for (int j = 0; j < n - i - 1; j++) 
      if (test(arr[j], arr[j + 1]) > 0) { 
        T temp = arr[j]; arr[j] = arr[j + 1]; 
        arr[j + 1] = temp; 
      } 
    } 

```

我们可以通过显式定义一个`Func<T1,T2,TReturn>`方法来执行比较来调用该例程。以下代码片段演示了此方法：

```cs
    int[] s3 = { -19, 20, 41, 23, -6 }; 
    Func<int ,int ,int> fn = (int a, int b ) => { 
      return (a > b) ? 1 : -1; 
    }; 
    s3.BSort2(fn); 
    foreach (var n in s3) 
    Console.WriteLine(n); 

```

我们也可以通过编写如下所示的调用位置 lambda 来调用该例程：

```cs
    s3.BSort2( (int a, int b ) =>  (a > b ) ? 1 : -1); 

```

如果你想对一个双精度数组进行排序，以下代码片段将完成这项任务：

```cs
    s4.BSort2( (double  a, double  b ) =>  (a > b ) ? 1 : -1); 

```

实现为 lambda 表达式的策略例程使代码更加简洁和可读。

# 使用 FP/OOP 重新审视迭代器模式

C#编程语言内置了对迭代器模式的支持，使用`foreach`循环构造。任何值得尊敬的程序员都可能编写过遍历集合并应用某种转换的代码。为了使事情更有条理，让我们全局编写一些代码来计算一组数字的算术平均数（平均值）。

```cs
    double[] arr = { 2, 4, 4, 4 ,5, 5, 7, 9}; 
    List<double> arrlist = arr.ToList(); 
    double nsum = 0; 
    foreach (var n in arr) { 
      nsum += n;  
    } 
    double avg = nsum / arrlist.Count; 
    Console.WriteLine(avg); 

```

之前的过程式代码确实完成了这项工作，并且相当可读。让我们探索是否可以进一步改进情况。如果你仔细观察，代码会遍历一个集合，并应用一些依赖于上下文的逻辑。如果我们想计算数字的乘积，我们需要通过稍微改变操作来编写相同的代码。在计算几何平均数时，我们计算数字的乘积。为了参数化计算，我们将使用 lambda 表达式或函数。

让我们编写一个聚合函数，该函数将遍历一个数字数组并执行在调用位置指定的操作。我们将使用该函数通过传递转换函数作为参数来计算算术和（称为**sigma**）或乘积和（称为**PI**）：

```cs
    private static double Aggregate( double [] p ,  
    double init,Func<double,double,double> fn)  
    { 
      double temp = init; 
      foreach (var n in p) 
      temp =  fn(n,temp); 
      return temp; 
    } 

```

让我们看看如何使用我们的聚合函数来编写算术平均函数。以下代码使用 lambda 表达式来指定计算：

```cs
    private static double AMEAN(double[] p) 
    { 
      return Aggregate(p, 0, (double a, double b) =>  
      { return b += a; }) / p.Length; 
    } 

```

几何平均数可以通过以下方式计算（为了简洁，我们忽略了诸如存在零、空或 null 数组等鲁棒性问题）：

```cs
    private static double GMEAN(double[] p) 
    { 
      double pi = Aggregate(p, 1,  
      (double a, double accum) => { return accum *= a; }); 
      return Math.Exp(Math.Log(pi)*(1 / p.Length)); 
    } 

```

为了将所有内容整合在一起，我们将编写一个标准差（`STD`）方法来计算数组中元素的标准差：

```cs
    private static double STD(double[] p) 
    { 
      double avg = Aggregate(p, 0,(double a, double b) =>  
      { return b += a; }) / p.Length; 
      double var = Aggregate(p, 0, (double a, double b) =>  
      { return b += ((a - avg)*(a-avg)); }) / p.Length; 
      return Math.Sqrt(var); 
    } 

```

`STD`方法首先计算数字列表的平均值，然后使用该值（`avg`）来计算方差。在计算方差时，将平均值从每个数字中减去并平方，以使结果值变为正数，然后将其累积到一个运行总和（`b`）变量中。最后，调用平方根函数来计算标准差。生成的代码可读性良好，可以用作其他列表转换的基础。

# MapReduce 编程习惯

在函数式编程（FP）的世界里，**MapReduce**被视为一种编程惯用用法。

### 注意

映射的过程可以描述为将函数或计算应用于序列中的每个元素以产生一个新的序列。归约将计算出的元素聚集起来，以产生过程、算法或函数变换的结果。

在 2003 年，两位谷歌工程师（Sanjay Ghemawat 和 Jeff Dean）发表了一篇关于公司如何使用 MapReduce 编程模型简化其分布式编程任务的论文。这篇题为《MapReduce：简化大型集群上的数据处理》的论文可在公共领域找到。这篇特定的论文非常有影响力，Hadoop 分布式编程模型就是基于论文中概述的思想。您可以在互联网上搜索论文的详细信息和 Hadoop 数据操作系统的起源。

为了降低复杂性，我们将实现一个 MapReduce 函数，用于对双精度浮点数数组进行计算。下面的`Map`函数对数组中的每个元素应用一个 lambda 表达式，并返回另一个数组；这实际上转换了原始数组，但没有对其进行修改。对于大型数组，为了加快操作速度，我们可以利用代码中的`Parallel.For`构造。

```cs
    public static double []  Map( double []  src,  
    Func<double,double> fnapply) 
    { 
      double[] ret = new double[src.Length]; 
      Parallel.For(0,ret.Length, 
      new ParallelOptions {  
        MaxDegreeOfParallelism =Environment.ProcessorCount }, 
        (i) => { ret[i] = fnapply(src[i]); 
      }); 
      return ret; 
    } 

```

MapReduce 模型也可以被视为计算中的散列/聚集模型。在上一个例子中，我们利用并行编程技术将计算散列到不同的任务中，这些任务将在不同的线程中调度。这样，大型数组的计算可以更快。在分布式版本场景中，我们将计算散列到不同的机器上。在归约场景中，我们累积或汇总在映射阶段发生的计算结果。以下代码展示了在汇总值到一个累加器中使用串行操作：

```cs
    public static double Reduce( double[] arr, 
    Func<double,double,double> reducer,double init) 
    { 
      double accum = init; 
      for(int i=0; i< arr.Length; ++i) 
      accum = reducer(arr[i], accum); 
      return accum; 
    } 

```

前面的两个函数可以用来计算标准差，如下所示：

```cs
    double[] arr = { 2, 4, 4, 4 ,5,5,7,9}; 
    double avg = Reduce( Map(arr,(double a) => {return a;}),  
    (double a, double b) =>  
    {return b +=a;},0)/arr.Length; 
    double var = Reduce(Map(arr, (double a) =>  
    {return (a-avg)*(a-avg); }), 
    (double a, double b) =>  
    {return b += a; }, 1) / arr.Length; 
    Console.WriteLine(Math.Sqrt(var)); 

```

目前，我们将讨论的重点放在 MapReduce 编程模型上。这是一个非常广泛的话题，至少在分布式计算环境的背景下。微软 Azure 平台将其服务的一部分作为 MapReduce 实现提供。要成为一名现代开发者，应该了解 MapReduce 编程模型及其作为各种产品的一部分提供的流处理对应物。

# 使用 FP/OOP 对模板方法模式进行新的尝试

在本节中，我们将展示如何使用命令式编程技术实现模板方法模式，并将对代码进行重构以编写一个函数式编程（FP）版本。作为一个具体的例子，我们计划使用**内部收益率**（**IRR**）的计算作为运行示例。

为了专注于本质，让我们快速编写一个计算 IRR 的引擎，给定一系列支付（`List<double>`）、利率和期限。作为第一步，我们需要使用折现技术计算一系列支付的现值。以下程序计算了一系列支付的现值：

```cs
    public static double CashFlowPVDiscreteAnnual( 
    List<double> arr, double rate, double period) 
    { 
      int len = arr.Count; 
      double PV = 0.0; 
      for (int i = 0; i < len; ++i) 
      PV += arr[i] / Math.Pow((1+rate/100), i); 
      return PV; 
    } 

```

### 备注

**IRR 的定义如下**：

IRR 是一种计算，通过找到使一系列支付折现到现值的利率，将一系列应积累的支付映射到现值。

以下程序是从书籍《C++金融数值算法》中移植过来的，作者是 Bernt Arne Odegard。有关详情，可以查阅互联网上该书的第 3.2.1 节（IRR 相关材料）。

```cs
    public static double CashFlow_IRR_Annual(List<double> arr, 
    double rate,double period)  
    { 
      const double ACCURACY = 1.0e-5; 
      const int MAX_ITERATIONS = 50; 
      const double ERROR = -1e30; 
      double x1 = 0.0; 
      double x2 = 20.0; 
      // create an initial bracket,  
      // with a root somewhere between bot,top 
      double f1 = CashFlowPVDiscreteAnnual(arr,x1,period); 
      double f2 = CashFlowPVDiscreteAnnual(arr,x2,period); 

      for (int j = 0; j < MAX_ITERATIONS; ++j) 
      { 
        if ( (f1*f2) < 0.0)  {break; } 

        if (Math.Abs(f1) < Math.Abs(f2))  
          f1 = CashFlowPVDiscreteAnnual(arr, 
          x1+= 1.06*(x1-x2),period ); 
        else  
          f2 = CashFlowPVDiscreteAnnual(arr, 
          x2+=1.06*(x2-x1),period); 
        if (f2*f1>0.0) 
          return ERROR; 
      } 
      double f = CashFlowPVDiscreteAnnual(arr,x1,period); 
      double rtb; 
      double dx=0; 
      if (f<0.0) { 
        rtb = x1;dx=x2-x1; 
      } 
      else { 
        rtb = x2; dx = x1-x2; 
      } 
      for (int i=0;i<MAX_ITERATIONS;i++) 
      { 
        dx *= 0.5; 
        double x_mid = rtb+dx; 
        double f_mid = CashFlowPVDiscreteAnnual(arr,x_mid,period); 
        if (f_mid<=0.0) { rtb = x_mid; } 
        if ( (Math.Abs(f_mid)<ACCURACY) ||  
        (Math.Abs(dx)<ACCURACY) ) 
        return x_mid; 
      } 
      return ERROR; // error. 
    } 

```

在掌握了这个程序后，我们可以创建一个 IRR 计算类，它将使用模板方法模式接收基于一系列未来支付的 IRR 计算输入。

让我们创建一个类，用于指定 IRR 计算子系统的输入：

```cs
    public class IRR_PARAMS 
    { 
      public List<double> revenue { get; set; } 
      public double rate { get; set; } 
      public double period { get; set; } 
    } 

```

在模板方法模式实现中，我们定义了一个`抽象`的基类，它执行大部分的计算，以及一个`抽象`的方法，具体类必须覆盖该方法以从中创建一个对象：

```cs
    public abstract class IRRComputationEngine 
    { 
      public abstract IRR_PARAMS Compute(); 
      public double Evaluate() 
      { 
        IRR_PARAMS par = Compute(); 
        if (par == null) return -1; 
        return CashFlow_IRR_Annual(par.revenue, 
        par.rate, par.period);  
      } 
      private static double CashFlow_IRR_Annual( 
      List<double> arr,double rate,double period) 
      { //----- Code Omitted } 
    } 
  } 

```

由于计算方法被标记为抽象，我们无法实例化一个对象。我们能做的事情主要是覆盖类并提供`抽象`方法的内容。

```cs
    public class BridgeIRR :IRRComputationEngine { 
      IRR_PARAMS ps = new IRR_PARAMS(); 
      public BridgeIRR(List<double> rev, double period, double rate){ 
        ps.period = period; ps.rate = rate; ps.revenue = rev; 
      } 
      public override IRR_PARAMS Compute() { return ps;} 
    } 

```

我们可以使用前面的类如下：

```cs
    double[] ns = { 10, 12, 13, 14, 20 }; 
    BridgeIRR test = new BridgeIRR(ns.ToList(),10,5); 
    double irr = test.Evaluate(); 
    Console.WriteLine(irr); 

```

## 使用 FP 实现模板方法模式

我们将使用 lambda 函数来简化模板方法模式的实现。而不是在抽象类子类化后覆盖方法，我们将使用匿名委托来实现相同的目标。以下给出的代码非常简单易懂：

```cs
    public class IRRComputationEngine2 
    { 
      public delegate IRR_PARAMS Compute(); 
      public Compute comp{get;set;}  

      public double Evaluate() 
      { 
        if (comp == null) return -1; 
        IRR_PARAMS par = comp(); 
        return CashFlow_IRR_Annual(par.revenue, 
        par.rate, par.period); 
      } 
      private static double CashFlow_IRR_Annual( 
        List<double> arr,double rate,double period) 
      { //--- Code Omitted } 
    } 

```

该类可以被利用如下：

```cs
    IRRComputationEngine2 n = new IRRComputationEngine2(); 
    double[] ns = { 10, 12, 13, 14, 20 }; 
    n.comp = () => { 
      IRR_PARAMS par = new IRR_PARAMS(); 
      par.revenue = ns.ToList(); 
      par.rate = 10; par.period = 5; 
      return par; 
    }; 
    double r = n.Evaluate(); 
    Console.WriteLine(r);
```

## 关于观察者模式的一个简要说明

函数式编程模型非常适合实现观察者模式。微软公司已经创建了一个名为 Rx 的库，以利用这种协同作用。这个库实现了一组属于组合事件流类别的功能。从下一章开始，我们将介绍观察者模式与函数式编程之间的关系。

## FP 是否取代 GoF 模式？

一些程序员有一种观点，认为函数式编程可以帮助我们远离 GoF 模式。就本书的作者而言，这有点言过其实。在本章中，我们已经看到如何使用函数式语言的习惯用法使 GoF 实现变得简单和更好。但本书中的示例大多使用 C#编写。如果我们使用 F#或 Haskell，这些语言的丰富类型系统中有一些技术可以用来消除一些 GoF 模式。

两个例子是：

+   部分函数应用使得对于 FP 编程语言来说构建器模式变得不再必要

+   函数模式匹配帮助我们消除访问者模式

如果你仔细观察事物的模式，GoF 模式是为了解决 C++ 静态和僵化的类型系统而提出的解决方案。之后，诸如 Java 和 C# 这样的语言在反思中脱颖而出，简化了 GoF 模式的实现。FP 可以被视为另一种增强。支持 FP 的动态语言可以帮助我们更好地实现 GoF 模式，有时甚至可以完全摒弃模式。但是，完全取代 GoF 模式有点像摸黑射击！

# 摘要

在本章中，你看到了如何将命令式代码（即 OOP 代码）与 FP 构造混合使用，以编写 GoF 设计模式的一些实现。一些函数式程序员认为，GoF 设计模式是为了克服 OOP（特别是 C++）的一般局限性而设计的机制。如果一个语言支持良好的 FP 构造，那么大多数模式实现都是不必要的。根据我们的理解，这种观点有些极端，FP 和 OOP 之间的中间道路似乎是一个更好的选择。在未来的日子里，FP 习惯用法将越来越受欢迎，进步将是一个平稳的过渡。在下一章中，我们将深入探讨函数式响应式编程技术的细微差别。这是一个非常重要的范式，其中 FP 和事件结合在一起，为我们提供了如 .NET **Reactive Extensions**（**Rx**）这样的框架。
