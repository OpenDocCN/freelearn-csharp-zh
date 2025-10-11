# 对象生命周期

对象生命周期是指对象从创建到销毁的时间段。在函数式编程语言中，存储在常量变量中的数据有一个定义的范围，其本质上是不可变的。这意味着只要应用程序没有停止，它们的生命周期就具有功能范围（没有销毁）。另一方面，面向对象编程中的对象是可变的，具有不同类型的范围，这导致了不同的生活方式。

内存在应用程序生命周期中起着至关重要的作用。所有对象和变量都使用内存空间。因此，了解在应用程序执行期间对象流动的概念非常重要。除非我们知道如何通过适当的代码或模式释放空间，否则会导致**内存泄漏**。

如果计算机程序暴露了错误并错误地管理内存分配，那么资源将变得不可用。这种情况被称为内存泄漏。

为了避免内存泄漏，我们在设计类时应该采取适当的注意，以确保在需要时资源可用。这只会发生在程序在对象超出作用域时立即释放与对象关联的资源的情况下。因此，应用程序可以无缝运行，因为未使用的空间会定期清理。然而，这个过程并不是在所有情况下都是自动的。我们要探讨这个主题的原因是了解 DI 技术在不同场景下如何管理对象的生命周期不同，这反过来又帮助我们设计类时做出适当的决策。

当相关类被实例化时，对象就诞生了。新生的对象会存在一段时间，只要应用程序保持对该对象的引用并继续使用它。如果你的应用程序关闭或对象的引用在代码中超出作用域，那么.NET Framework 将标记该对象从内存中删除。

在这个特定的章节中，我们将学习.NET Core 如何管理对象。此外，我们还将探讨确定对象何时可回收以及如何与之交互的技术。

本章我们将涵盖以下主题：

+   管理和非管理资源。

+   对象的创建和销毁

+   `IDisposal`接口

+   .NET Core 中的对象生命周期管理

# 管理对象生命周期

内存中有两个基本的地方——栈和堆。

# 栈与堆的比较

让我们简要了解一下这些内存空间类型。

| **栈** | **堆** |
| --- | --- |
| 静态内存：为应用程序分配的固定内存空间。 | 动态内存：没有固定空间。 |
| 当一个方法被调用时，会预留一块栈空间来存储方法信息，如方法名、局部变量、返回类型等。 | 可以存储任何东西。开发者有灵活性来管理这个空间。 |
| 内存分配遵循**后进先出**（**LIFO**）的顺序。所以，当 `Main` 函数调用方法 `A()`，然后 `A()` 调用 `B()` 时，`B()` 将被存储在顶部并首先执行。参见图表： | 没有这样的模式来存储数据。 |

以下图表显示了数据在一个程序中如何存储在栈和堆中：

![](img/dff1fbbb-ae31-446a-87ba-1c011ddfec81.png)

`Main()` 调用 `A()`，然后 `A()` 调用 `B()` 方法。根据栈的特性，它首先移除最后一个，即 `B()`。所以， `B()` 首先执行。然后 `B()` 从栈中移除，然后处理并从内存中移除 `A()`。之后， `Main()` 方法执行并移除。在 `Main()` 中名为 `foo` 的引用类型变量存储在栈上，但实际的对象是在堆上分配内存的。

考虑以下示例：

```cs
    static void Main(string[] args)
    {
      string name = "Dependency Injection";
      SomeClass sc = new SomeClass()
    }
```

变量 `name` 是一个值类型，它直接存储在栈上。但是，当我们编写 `SomeClass sc = new SomeClass()` 时，实际上是在告诉框架将对象存储在堆上。除此之外，它还在栈上为变量 `sc` 分配了一块内存空间，该空间持有对这个对象的引用。

现在，当 `Main` 方法执行完成后，变量 `name` 和 `sc` 将被释放，内存空间变为空闲。但是这里有一个问题。变量 `sc`（引用类型）从栈上释放，但实际的对象仍然在堆上。只是引用被移除了。由于引用从栈上移除，所以实际上无法知道堆上是否存在与之相关的对象。我们现在遇到了一个管理问题。

为了解决这个问题（在 C++ 中），我们可能已经做了如下操作 `delete sc;`。然而，在 C# 这种托管语言中，有一个名为**垃圾回收器**（**GC**）的服务，它会通过分析所有那些标记为**超出作用域**的对象来自动清理未使用的内存。

**托管语言**是一种高级语言，它依赖于运行时环境提供的服务来执行，例如垃圾回收服务、安全服务、异常处理等等。它使用公共语言运行时（CLR）在 .Net 语言中执行或在 Java 中使用**Java 虚拟机**（**JVM**）。

# 托管和非托管资源

纯 .NET 代码被称为**托管资源**，因为它们可以直接由运行时环境管理。另一方面，非托管资源是指那些不在运行时直接控制之下的资源，例如文件句柄、COM 对象、数据库连接等等。例如，如果你打开到数据库服务器的连接，这将使用服务器上的资源（用于维护连接）以及可能的其他非 .NET 资源。

管理资源直接由 CLR（公共语言运行时）针对，因此垃圾收集器会清理它们，这是一个自动的过程。作为开发者，你通常不需要显式调用 GC。然而，有一个问题，当我们考虑非托管资源，如数据库连接时。我们必须自己处理它们，因为 CLR 无法处理。我们必须使用`Finalize`方法手动释放它们。

# 代数

堆被分为三代，以便它可以处理长期和短期对象。垃圾收集基本上回收了通常只占用堆一小部分空间的短暂对象。

堆上有以下三代对象：

+   **第 0 代**：当一个对象被初始化时，它的代数开始。它首先落入第 0 代。这个代的对象通常是短暂的。这些对象更容易被 GC 销毁。GC 收集这些短暂的对象，以便它们可以被释放出来，从而释放内存空间。如果对象在 GC 收集后仍然存活，这意味着它们将停留更长的时间，因此被提升到下一代。

+   **第 1 代**：这个代的对象比第 0 代对象存在的时间更长。GC 确实会从这个代收集对象，但不像第 0 代那样频繁，因为它们的生存期被应用程序扩展以进行更多操作。这个代幸存下来的对象将进入第 2 代。

+   **第 2 代**：这些是在应用程序中存在时间最长的对象。成功通过前两代的对象被认为是第 2 代。GC 在释放这些对象时很少介入。

# 对象创建

构造函数负责创建特定类的对象。如果我们创建任何没有构造函数的类，编译器将自动为该类创建一个默认构造函数。每个类至少有一个构造函数。

构造函数也可以重载，这提供了一种方便的方式来使用不同的属性构建对象，这意味着它可以通过接受某些参数（就像一个普通方法一样）并在其体内将其属性赋值（也称为参数化构造函数）来实例化对象。构造函数应该与类名具有相同的名称。

# 复制构造函数

另一种类型的构造函数称为**复制构造函数**。正如其名所示，它可以复制一个对象到新的对象（同一类的对象）中，该对象将被实例化。换句话说，它是一个包含相同类型参数的参数化构造函数。复制构造函数的主要目的是将新实例初始化为现有实例的值。我们将在稍后的示例中看到如何实现这一点。

# 对象销毁

我们有不同的方式来销毁一个对象。让我们逐一探索。

# 最终化

终结器用于销毁对象。我们可以使用带有类名波浪号 (`~`) 符号的析构方法来设计终结器。我们很快就会看到它的实际应用：

```cs
    ~Order()
    {
      // Destructor or Finalizer
    }
```

垃圾回收器完全控制着终结过程，因为它在对象超出作用域时内部调用此方法。然而，我们可以在析构函数中编写代码以自定义我们的需求，但我们不能只是告诉某人调用析构函数。即使你非常确定对象不再需要并决定释放它，你也不能显式执行析构函数以释放空间。你必须等待垃圾回收器收集对象以进行销毁。

终结过程有两个收集周期。在第一个周期中，短生命周期的对象被标记为需要终结。在下一个周期中，它调用终结器以完全从内存空间中释放它们。

# `IDisposable` 接口

如我们之前讨论的，未托管资源不在框架的直接控制之下。我们可以在终结器中轻松回收这些资源，正如我们之前讨论的那样。这意味着，当对象被垃圾回收器销毁时，它们将被释放。然而，垃圾回收器仅在 CLR 需要更多空闲内存时才会销毁对象。因此，即使对象超出作用域很长时间，资源仍然可能存在。

因此，我们需要在完成使用资源后立即释放资源。如果你的类实现了 `IDisposable` 接口，它们可以提供一种机制来主动管理系统资源。此接口公开了一个方法，`Dispose()`，当客户端完成对一个对象的使用时应该调用它。你可以使用 `Dispose` 方法立即释放资源以执行诸如关闭文件和数据库连接等任务。与 `Finalize` 析构函数不同，`Dispose` 方法不是自动调用的。对象必须显式调用 `Dispose` 以释放资源。

此方法是 `IDisposable` 接口中的唯一方法，可以用来手动释放未托管资源。

```cs
    public interface IDisposable
    {
      void Dispose();
    }
```

现在只需调用 `Dispose()` 方法即可。但等等。你只能从实现了此接口并定义了 `Dispose()` 方法的类对象中调用此方法。

例如，`SqlConnection` 类实现了此接口，并为我们提供了 `Dispose()` 方法，可以使用如下方式使用它。一旦你完成对连接对象的操作，就调用 `Dispose`：

```cs
    var connection = new SqlConnection("ConnectionString");
    // Do Database related stuff here.

    // After we are done, let's call Dispose.
    connection.Dispose();
```

在 .NET 中处理对象销毁的另一种优雅方法是，而不是直接调用 `Dispose`，我们可以使用 `using` 块。相同的语句可以用以下方式用 `using` 块装饰：

```cs
     using (var connection = new SqlConnection("ConnectionString"))
    {
      // Use the connection object and do database operation.
    }
```

当我们这样做时，它将代码转换为 `try...finally` 中间代码。在 `finally` 块中销毁连接对象，这是我们创建的。除非你这样做，否则连接对象将保留在内存中。随着时间的推移，当我们获得大量连接时，内存开始泄漏。

如果您正在使用终结器（析构函数）方法，请确保在其中调用 `Dispose()` 以释放您想要释放的资源。这样，您将确保即使有人在使用您的类时忘记释放它们，资源也会被垃圾回收器清理。

出于好奇，您可能想知道，如果在调用 `Dispose()` 之前发生了一些异常，会发生什么？那个对象会被释放吗？这个问题的解决方案是将它包裹在一个 `try...finally` 块中，这样无论程序发生什么情况，`finally` 都会被调用，您可以在其中释放对象。为了简单起见，框架有一个叫做 `using` 块的漂亮功能，可以如下使用：

`Dispose()` 与 `Close()`: 对于 `SqlConnection` 对象，您是否感到困惑应该调用哪一个？它们是两种不同的方法，用于解决不同的问题。`Close()` 仅关闭连接。您可以使用相同的对象重新打开连接。然而，`Dispose()` 关闭连接（在底层调用 `Close()`）然后从内存中释放对象。您不能再使用该对象了。

您可以在 [`docs.microsoft.com/en-us/dotnet/standard/garbage-collection/`](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/) 上了解更多关于垃圾回收器的信息。

# 考虑一个例子

一个简单的 `Order` 类可以有一个默认的、参数化的、复制构造函数以及一个析构函数（用于销毁对象）：

```cs
    class Order
    {
        public string ProductName { get; set; }
        public int Quantity { get; set; }

        public Order()
        {
           // Default Constructor
        }

        public Order(string productName, int quantity)
        {
           // Constructor with two arguments
           ProductName = productName;
           Quantity = quantity;
        }

        public Order(Order someOrder)
        {
           // Copy constructor
           ProductName = someOrder.ProductName;
           Quantity = someOrder.Quantity;
        }

        ~Order()
        {
           // Destructor or Finalizer
        }
    }
```

您可以看到构造函数是如何使用和不使用参数形成的。注意复制构造函数，它接受一个与同一类相同的对象作为参数，并在体内将它的属性赋值给正在创建的对象。

终结器隐式地调用对象的基类的 `Finalize` 方法。因此，当垃圾回收器调用终结器时，可能会调用如下所示的方法：

```cs
    protected override void Finalize()
    {  
      try  
      {  
        // Cleanup statements...  
      }  
      finally  
      {  
         base.Finalize();  
      }  
    }
```

让我们通过在 .NET Core 2.0 的控制台应用程序中使用代码片段来验证这种行为：

```cs
    class BaseClass
    {
      ~BaseClass()
      {
        System.Diagnostics.Trace.WriteLine("BaseClass's destructor is called.");
      }
    }
    class DeriveClass1 : BaseClass
    {
      ~DeriveClass1()
      {
        System.Diagnostics.Trace.WriteLine("DeriveClass1's destructor
            is called.");
      }
    }

   class DeriveClass2 : DeriveClass1
   {
      public DeriveClass2()
      {
        System.Diagnostics.Trace.WriteLine("DeriveClass2's constructor is called.");
      }

      ~DeriveClass2()
      {        
         System.Diagnostics.Trace.WriteLine("DeriveClass2's destructor 
          is called.");
      }
   }

   class Program
  {
    static void Main(string[] args)
    {
       DeriveClass2 t = new DeriveClass2();

       // Unlike .NET Framework, .NET Core 2.0, as of now, 
       // does not call GC on application termination 
       // to finalise the objects.
       // So, we are trying to manually call GC
       // to see the output.
       System.GC.Collect();
    }
  }
```

首先创建 `DeriveClass2` 对象，它记录构造函数消息。然后执行 `DeriveClass2` 的析构函数。因此，当 `Main` 函数执行完成后，它首先被销毁。此外，父类也有析构函数。因为子类 (`DeriveClass2`) 对象已经被销毁，它也会运行父类的析构函数。以下截图来自 Visual Studio 的输出窗口。请确保以发布模式运行应用程序：

![](img/ffc917d6-732d-47c0-b688-4056666c2f2e.png)

# 实现 `IDisposable` 接口

学习如何实现 `IDisposable` 接口很重要，因为您可能在项目中使用用户定义的类来处理非托管资源。系统定义使用非托管资源的类实现 `IDisposable` 并公开 `Dispose`，这样我们就可以轻松地调用该方法来释放对象，就像我们在 `SqlConnection` 类的代码片段中看到的那样。

有一个名为 **Dispose 模式** 的模式，开发者在实现 `IDisposable` 时必须遵循。让我们来探索一下。我会一步一步地讲解。

# Step1 - 类的基本结构

我们将有一个 `ExampleIDisposable` 类，它实现了 `IDisposable` 接口。我不会演示非托管资源的用法，因为我们的目的是学习这个模式。我只是在构造函数中有一个控制台行，说明我们正在获取非托管资源。

```cs
    class ExampleIDisposable : IDisposable
    {
        public Dictionary<int, string> Chapters{ get; set; }
        public ExampleIDisposable(Dictionary<int, string> chapters)
        {
           // Managed resources
           Console.WriteLine("Managed Resources acquired");
           Chapters = chapters;

           // Some Unmanaged resources
           Console.WriteLine("Unmanaged Resources acquired");
        }

        public void Dispose()
        {
           Console.WriteLine("Someone called Dispose");

           // Dispose managed resources
           if (Chapters != null)
           {
              Chapters = null;
           }

          // Dispose unmanaged resources
        }
     }
```

你可以看到这个类包含一个托管属性，它在构造函数中被初始化。我们将为它打印一行。同样，我们可能有一些在类中声明并由构造函数赋予生命的非托管资源属性。由于我们实现了 `IDisposable`，我们被迫定义唯一的方法 `Dispose()`。目前，我们只是在其中有一个控制台行。

让我们试一试：

```cs
    static void Main(string[] args)
   {
      ExampleIDisposable disposable = new ExampleIDisposable(new Dictionary<int,
        string> {{ 5, "Object Composition" },
                { 6, "Object Lifetime" }
        });

      disposable.Dispose();
      Console.ReadLine();
    }
```

这会产生以下输出：

![](img/af3d32ed-b02a-4594-b70c-2984ed113c4d.png)

在继续前进之前，我们需要理解两个重要的观点。

**当 GC 调用 Finalizer 时会发生什么？**当一个对象超出作用域时，它将被添加到 Finalizer 队列中，垃圾回收器将对其采取行动以从内存中释放它。我们不知道这会发生什么时间。如果你在 `Dispose()` 中杀死了托管资源，那么我们需要限制进入 Finalizer 队列的对象，从而通知 GC 不要对它们采取行动，因为它们已经不存在了。此外，这对 GC 来说也是一个开销。

**如果开发者忘记调用 Dispose 会怎样？**假设，使用我们类的开发者没有销毁它。我们仍然需要处理这种情况。我们可以通过在 Finalizer 中调用 `Dispose()` 来轻松地做到这一点，但是等等！我们需要要求 `Dispose()` 方法只杀死非托管资源，而不是托管资源，因为 GC 会自动处理它们。

这就是另一个方法出现的地方。

# Step2 - 定义一个 Dispose 重载方法

我们在类内部定义的 `Dispose()` 方法将在我们直接通过类的对象调用它时帮助我们。然而，我们还需要在类内部定义另一个 `Dispose()` 重载，这将回答我们之前讨论过的问题。让我们将这个方法引入到我们的类中。

```cs
    class ExampleIDisposable : IDisposable
    {
      public Dictionary<int, string> Chapters { get; set; }
      public ExampleIDisposable(Dictionary<int, string> chapters)
      {
        // Managed resources
        System.Diagnostics.Trace.WriteLine("Managed Resources acquired");
        Chapters = chapters;

        // Some Unmanaged resources
         System.Diagnostics.Trace.WriteLine("Unmanaged Resources acquired");
      }
      public void Dispose()
       {
         System.Diagnostics.Trace.WriteLine("Someone called Dispose");
         Dispose(true);
         GC.SuppressFinalize(this);
       }
       public void Dispose(bool disposeManagedResources)
        {
          if (disposeManagedResources)
          {
            if (Chapters != null)
            {
              Chapters = null;
            }

            System.Diagnostics.Trace.WriteLine("Managed Resources disposed");
          }
          System.Diagnostics.Trace.WriteLine("Unmanaged Resources disposed");
        }

        ~ExampleIDisposable()
        {
          System.Diagnostics.Trace.WriteLine("Finalizer called: Managed
              resources will be cleaned");
          Dispose(false);
        }
      }
```

对 `Dispose` 方法所做的修改如下所述：

+   `public void Dispose()`: 现在，我们要求 `Dispose(bool)` 方法释放所有类型的资源。`public void Dispose(). GC.SuppressFinalize(this)`; 抑制了 GC Finalizer 的调用，因为我们已经在 `Dispose(bool)` 中处理了所有内容。

+   `public void Dispose(bool)`: 这个方法是这个模式的重要部分。通过 `bool` 参数，它决定是否需要杀死托管资源。

我已经将控制台行替换为跟踪行，这样`main`方法就会结束，我们可以在输出屏幕上看到这些行。如果您只是从`Main`方法中删除`Console.ReadLine();`并再次运行应用程序，生成的输出将如下所示：

![图片](img/fd4c2965-f338-4a83-96b1-6dca53511e7c.png)

从`Main`方法中移除`Dispose()`调用，即`disposable.Dispose();`，将导致以下结果。注意在`Main`方法结束时调用`GC.Collect();`，就像我们在第一步中做的那样：

![图片](img/3ab0882c-c4b8-412a-a457-cc9b83b35a01.png)

这意味着无论开发者是否忘记丢弃，最终都会调用最终化器，我们在其中调用了`Dispose(false);`，最终释放了非托管资源。当然，最终化器会自动删除托管资源。您可以看到，在最后一种情况下，缺少了“有人调用了 Dispose”和“已丢弃托管资源”这两行。

# 第 3 步 - 修改派生类的 Dispose(bool)

由于我们有`Dispose(bool)`重载，它将直接在对象上可用以调用。我们没有必要将`Dispose(bool)`暴露给对象以进行直接调用，因为我们从`Dispose()`和最终化器内部调用它。

用户不应该传递布尔值并决定丢弃什么以及如何丢弃。他们唯一要做的事情是调用`Dispose()`，这将释放所有类型的资源。因此，我们将通过将访问修饰符从`public`更改为`protected`来限制对`Dispose(bool)`的调用。

`Dispose(bool)`是正在实现`IDisposable`的类的逻辑块。任何将要派生实现`IDisposable`的基类的类都可能有自己的自定义丢弃逻辑。因此，他们不需要添加另一个丢弃方法，只需覆盖基类的`Dispose(bool)`即可。为了实现这一点，我们需要在方法名前添加一个`virtual`关键字。

前面的段落要求修改我们非常知名的方法`Dispose(bool)`：

```cs
    protected virtual void Dispose(bool disposeManagedResources)
    {
      if (disposeManagedResources)
      {
        if (Chapters != null)
        {
           Chapters = null;
        }
        System.Diagnostics.Trace.WriteLine("Managed Resources disposed");
      }
      System.Diagnostics.Trace.WriteLine("Unmanaged Resources disposed");
    }
```

# 第 4 步 - 处理重复的 Dispose 调用

我们应该管理用户可能多次调用`Dispose()`的情况。如果我们不处理这种情况，后续的调用将只是对运行时来说不必要的执行，因为运行时将尝试释放已经被丢弃的对象。

我们可以在类内部轻松地放置一个标志，该标志将指示对象是否已被丢弃。

```cs
    bool disposed = false;
    protected virtual void Dispose(bool disposeManagedResources)
    {
      if (disposed)
      {
        System.Diagnostics.Trace.WriteLine("Dispose(bool) already called");
         return;
      }
      if (disposeManagedResources)
        {
          if (Chapters != null)
          {
             Chapters = null;
          }
          System.Diagnostics.Trace.WriteLine("Managed Resources disposed");
       }
       System.Diagnostics.Trace.WriteLine("Unmanaged Resources disposed");
       disposed = true;
     }
```

注意`disposed`变量，它在`Dispose(bool)`内部使用。我们在方法内部检查它是否为真。如果是真的，那么我们就从方法中返回/退出，否则执行丢弃代码。最后，我们将它设置为真。因此，第一次`Dispose(bool)`将完全执行，之后，它将仅在调用一次后返回。这样，我们防止了多次丢弃同一个对象，这是一个开销。

让我们修改代码以多次调用`Dispose()`：

```cs
    disposable.Dispose();
    disposable.Dispose();
    disposable.Dispose();
```

这将给出以下输出：

![图片](img/ac96fa07-23c1-4416-9e2f-fb32c5d50b49.png)

你可以看到，对于第一次调用，一切如预期进行。对于同一对象的下一个两个后续的 Dispose()调用，结果只是方法的一个简单返回。这就是为什么我们看到两组“Someone called Dispose”和“Dispose(bool) already called”的消息。

好的，我想在所有这些步骤之后展示最终的代码：

```cs
    class ExampleIDisposable : IDisposable
    {
        public Dictionary<int, string> Chapters { get; set; }
        bool disposed = false;

        public ExampleIDisposable(Dictionary<int, string> chapters)
        {
            // Managed resources
            System.Diagnostics.Trace.WriteLine("Managed Resources acquired");
            Chapters = chapters;

            // Some Unmanaged resources
            System.Diagnostics.Trace.WriteLine("Unmanaged Resources acquired");
        }

        public void Dispose()
        {
           System.Diagnostics.Trace.WriteLine("Someone called Dispose");

           Dispose(true);
           GC.SuppressFinalize(this);
        }

        protected virtual void Dispose(bool disposeManagedResources)
        {
           if (disposed)
           {
               System.Diagnostics.Trace.WriteLine("Dispose(bool) already called");
               return;
           }

           if (disposeManagedResources)
           {
              if (Chapters != null)
              {
                 Chapters = null;
              }
              System.Diagnostics.Trace.WriteLine("Managed Resources
                  disposed");
           }

           System.Diagnostics.Trace.WriteLine("Unmanaged Resources disposed");
           disposed = true;
        }

        ~ExampleIDisposable()
        {
          System.Diagnostics.Trace.WriteLine("Finalizer called: Managed 
                resources will be cleaned");
          Dispose(false);
        }
      }
```

不要忘记，你可以使用`using`语句与任何实现了`IDisposable`接口的类一起使用。例如，让我们为`ExampleIDisposable`编写代码：

```cs
    using (ExampleIDisposable disposable = 
        new ExampleIDisposable(new Dictionary<int, string> {
        { 5, "Object Composition" },
        { 6, "Object Lifetime" }
        }))
    {
      // Do something with the "disposable" object.
    }
```

如果你运行这个，它会产生与*步骤 2：定义 Dispose 重载方法*部分下的第一个截图相同的结果。

# .NET Core 中的对象生命周期管理

在前面的章节中，我们已经探讨了依赖注入是如何集成到.NET Core 中的。现在我们已经了解了对象是如何由.NET Framework 管理的，让我们来看看它们在.NET Core 中的生命周期。

只用一行代码，我可以说，在启动时，.NET Core 会取一个类，给它标记一个生命周期，然后实例化并存储在容器或服务集合中。考虑以下截图：

![](img/663079d7-0bb5-478d-a787-557867ec8ef7.png)

我们将探讨在.NET Core 中如何处理以下内容：

+   对象创建。

+   对象的生命周期。

+   完成所有工作后对象的处置。

# 对象创建

通常在 ASP.NET Core 2.0 中，注入的类型被称为**服务**。例如，注入的接口被称为`IServiceCollection`，我们可以通过使用这里的`AddSingleton`方法添加所需的服务。我们很快就会了解更多关于它的内容：

```cs
    public void ConfigureServices(IServiceCollection services)
    {
      services.AddMvc();
      services.AddSingleton<IExampleService, ExampleService>();
    }
```

当我们执行前面的代码时，ASP.NET Core 内置的 DI 框架执行两个重要的步骤：

+   **实例化**：提供的服务的对象（例如：`ExampleService`）被实例化，以便在调用控制器时可以使用。对象通过构造函数注入或属性注入获得。`IExampleService`将是控制器的参数。这个接口的实现者`ExampleService`可以被实例化和注入。我们稍后会看到构造函数。

+   **生命周期管理**：然后它决定注入到控制器中的对象的生命周期（创建和处置）。框架提供了不同类型的生活周期，我们将在下一节学习。

让我们探索*ASP.NET Core*默认提供的三个生命周期模式。以下是一个关于这些生活周期的快速参考表：

| **生命周期** | **描述** | **处置** |
| --- | --- | --- |
| 原型（临时或短期） | 每次请求服务时都会创建一个新的实例。 | 从不 |
| 作用域（范围或范围） | 为定义的作用域创建一个实例。当达到作用域结束时，如果需要，将创建另一个实例。一个简单的范围可以表述为一个`Web 请求`。任何请求特定`Web 请求`实例的资源都将从容器中提供相同的实例。 | 作用域结束时被处置 |
| 单例（单一或个体） | 容器创建一个实例，该实例将在整个应用程序的每个请求中用于/共享。 | 当容器被释放时被销毁 |

我们现在将详细查看每个概念。不用担心，如果你在阅读后感到困惑。在精心设计的例子之后，你肯定会理解这个概念的。

首先，让我们设计我们的 ASP.NET MVC Core 应用程序以使用所有这些类型的生命周期。这个例子与官方文档中给出的例子相似，可以在[`docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection`](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)找到。

此处的主要目标是理解实例在传入请求期间是如何保持的。我们将从两个地方调查这些实例：一个是从控制器，另一个是从一个类。这两个家伙都将使用相同的接口作为构造函数中注入的依赖项。让我们一步一步地来了解这些内容。

# 设计接口

首先编写一个接口`IExampleService`，它有一个`Guid`类型的属性`ExampleId`。然后编写四个实现此接口的接口。我们根据不同的生命周期类型给它们命名，这样我们就可以在之后轻松地识别它们：

```cs
    public interface IExampleService
    {
      Guid ExampleId { get; }
    }

    public interface IExampleTransient : IExampleService
    {
    }
    public interface IExampleScoped : IExampleService
    {
    }
    public interface IExampleSingleton : IExampleService
    {
    }
    public interface IExampleSingletonInstance : IExampleService
    {
    }
```

# 具体类

抽象意味着可以概念化的东西，而具体则意味着存在于现实中的东西，而不是抽象的。假设我们考虑三个形状，比如圆形、矩形和正方形。形状看起来像是一个概念，不是吗？另一方面，圆形、矩形和正方形实际上具有形状的行为来表示一个具体的概念。抽象类在本质上是不完整的，当一个类（或继承）其行为时，它就变得完整，这被表示为**具体**。

因此，抽象（或不完整）的概念意味着通过继承它的其他类来完成，形成*具体类*。抽象类是不完整的，只是一个概念，所以实例化它是没有意义的。例如，形状不是用来实例化的，而是用来被实际的形状类继承的。以下图表显示了这种关系：

![](img/36b8337a-1d81-4ff2-9ace-c1d5a66d9050.png)

一个单独的`Example`类实现了所有这些接口，并定义了`ExampleId` guid。有两个构造函数：一个接受`Guid`作为参数，另一个是默认的，它初始化一个新的 guid。所有这些接口都将在这个类的*Startup*中解析。我们很快就会接近那段代码：

```cs
    using LifetimesExample.Interfaces;
    using System;

    namespace LifetimesExample.Models
   {
      public class Example : IExampleScoped, IExampleSingleton, 
         IExampleTransient, IExampleSingletonInstance
      {
        public Guid ExampleId { get; set; }
        public Example()
        {
          ExampleId = Guid.NewGuid();
        }
        public Example(Guid exampleId)
        {
          ExampleId = exampleId;
        }
      }
    }
```

# 服务类

正如我们之前所说的，使用这些接口创建一个简单的类，这个类被称作`ExampleService`。这里有一个构造函数，它等待接口被注入并分配给局部接口类型变量。

```cs
    using LifetimesExample.Interfaces;
    namespace LifetimesExample.Services
    {
    public class ExampleService
    {
        public IExampleTransient TransientExample { get; }
        public IExampleScoped ScopedExample { get; }
        public IExampleSingleton SingletonExample { get; }
        public IExampleSingletonInstance SingletonInstanceExample { get; }

        public ExampleService(IExampleTransient transientExample,
            IExampleScoped scopedExample,
            IExampleSingleton singletonExample,
            IExampleSingletonInstance instanceExample)
        {
            TransientExample = transientExample;
            ScopedExample = scopedExample;
            SingletonExample = singletonExample;
            SingletonInstanceExample = instanceExample;
        }
      }
    }
```

# 控制器

控制器几乎与 `Service` 类相同，只是多了一个 `ExampleService` 的引用。它有一个构造函数来初始化它们。`ExampleService` 的属性将在视图中打印出来，这就是为什么我们引用那个类：

```cs
    using Microsoft.AspNetCore.Mvc;
    using LifetimesExample.Services;
    using LifetimesExample.Interfaces;

    namespace LifetimesExample.Controllers
    {
      public class ExampleController : Controller
      {
        private readonly ExampleService _exampleService;
        private readonly IExampleTransient _transientExample;
        private readonly IExampleScoped _scopedExample;
        private readonly IExampleSingleton _singletonExample;
        private readonly IExampleSingletonInstance _singletonInstanceExample;

        public ExampleController(ExampleService ExampleService,
            IExampleTransient transientExample,
            IExampleScoped scopedExample,
            IExampleSingleton singletonExample,
            IExampleSingletonInstance singletonInstanceExample)
        {
            _exampleService = ExampleService;
            _transientExample = transientExample;
            _scopedExample = scopedExample;
            _singletonExample = singletonExample;
            _singletonInstanceExample = singletonInstanceExample;
        }

        public IActionResult Index()
        {
            // viewbag contains controller-requested services
            ViewBag.Transient = _transientExample;
            ViewBag.Scoped = _scopedExample;
            ViewBag.Singleton = _singletonExample;
            ViewBag.SingletonInstance = _singletonInstanceExample;

            // Example service has its own requested services
            ViewBag.Service = _exampleService;

            return View();
        }
      }
    }
```

在 `Index()` 动作中，我们正在将所有这些值返回到 `ViewBag`。

# 视图

最后，但同样重要的是，我们将简单地设计一个视图，以显示所有这些值，这样我们就可以开始观察。这将是 `Views/Example` 中的 `Index.cshtml`。

```cs
    @using LifetimesExample.Interfaces
    @using LifetimesExample.Services

    @{
      ViewData["Title"] = "Index";
     }

    @{
      IExampleTransient transient = (IExampleTransient)ViewData["Transient"];
      IExampleTransient scoped = (IExampleTransient)ViewData["Scoped"];
      IExampleTransient singleton = (IExampleTransient)ViewData["Singleton"];
      IExampleTransient singletonInstance = (IExampleTransient)ViewData["SingletonInstance"];
      ExampleService service = (ExampleService)ViewBag.Service;
    }

    <h2>Lifetimes</h2>

    <h3>ExampleController Dependencies</h3>
    <table>
     <tr>
        <th>Lifestyle</th>
        <th>Guid Value</th>
     </tr>
    <tr>
        <td>Transient</td>
        <td>@transient.ExampleId</td>
    </tr>
    <tr>
        <td>Scoped</td>
        <td>@scoped.ExampleId</td>
    </tr>
    <tr>
        <td>Singleton</td>
        <td>@singleton.ExampleId</td>
    </tr>
    <tr>
        <td>Instance</td>
        <td>@singletonInstance.ExampleId</td>
    </tr>
   </table>

   <h3>ExampleService Dependencies</h3>
   <table>
    <tr>
        <th>Lifestyle</th>
        <th>Guid Value</th>
    </tr>
    <tr>
        <td>Transient</td>
        <td>@service.TransientExample.ExampleId</td>
    </tr>
    <tr>
        <td>Scoped</td>
        <td>@service.ScopedExample.ExampleId</td>
    </tr>
    <tr>
        <td>Singleton</td>
        <td>@service.SingletonExample.ExampleId</td>
    </tr>
    <tr>
        <td>Instance</td>
        <td>@service.SingletonInstanceExample.ExampleId</td>
    </tr>
  </table>
```

最后，我们完成了代码。你确定吗？我们忘记了主要入口点。决定哪个类将针对哪个接口进行解析的那个。

# Startup ConfigureServices

现在，每种类型的生命周期都被添加到使用 `Example` 类解析的容器中。`ExampleService` 类为自己解析。这意味着，无论何时在应用程序的任何地方需要 `ExampleService` 进行注入，它都会自动分配一个具有所有属性的该类对象：

```cs
    public void ConfigureServices(IServiceCollection services)
    {
        // Add framework services.
        services.AddMvc();

        services.AddTransient<IExampleTransient, Example>();
        services.AddScoped<IExampleScoped, Example>();
        services.AddSingleton<IExampleSingleton, Example>();
        services.AddSingleton<IExampleSingletonInstance, Example>();
        services.AddSingleton(new Example(Guid.Empty));
        services.AddTransient<ExampleService, ExampleService>();
    }
```

`Add***` 方法（具有不同的生命周期）确保对象根据预期的行为被创建。一旦这些对象被初始化，它们就可以在任何需要的地方被注入。

如果我们现在运行应用程序，对于第一个请求，我们将得到以下输出：

![](img/9a77645f-5f93-4b86-9e98-17f9d68cf2aa.png)

我在另一个标签页中打开了页面（或者你也可以简单地刷新标签页）。我看到以下输出：

![](img/eed82ac7-6517-4824-90c6-44afd3b3ed53.png)

# 对象生命周期

让我们逐一检查所有生命周期，根据我们在这些截图中看到的价值。这些值帮助我们识别对象。

# 单例

观察截图和图表，首先明显看到的是 `Singleton`。无论在运行应用程序后进行多少次请求，对象都是相同的。它不依赖于控制器或 `Service` 类。

# 作用域

对于特定的请求，对象在整个作用域内是相同的。注意在第二个请求中对象的 guid 值是如何不同的。然而，在两个请求中，控制器和 `Service` 类的值是相同的。这里的范围是 Web 请求。当服务另一个请求时，对象将被重新创建。

虽然在代码中创建作用域是可能的。有一个名为 `IServiceScopeFactory` 的接口，它公开了 `CreateScope` 方法。`CreateScope` 是 `IServiceScope` 类型，它实现了 `IDisposable`。在这里，`using` 块帮助我们处理作用域实例的释放：

```cs
    var serviceProvider = services.BuildServiceProvider();
    var serviceScopeFactory = serviceProvider.GetRequiredService<
          IServiceScopeFactory>();

    IExampleScoped scopedOne;
    IExampleScoped scopedTwo;

    using (var scope = serviceScopeFactory.CreateScope())
    {
      scopedOne = scope.ServiceProvider.GetService<IExampleScoped>();
    }
    using (var scope = serviceScopeFactory.CreateScope())
    {
     scopedTwo = scope.ServiceProvider.GetService<IExampleScoped>();
    }
```

我们在前面代码中使用 `CreateScope` 创建了两个作用域实例。这两个实例相互独立，并且不像普通的作用域实例那样在请求中共享。这是因为我们手动指定了作用域而不是默认的 Web 请求作用域。

# 临时

简单！每次从容器请求时都会创建一个新的对象。在截图中，你可以看到即使对于单个请求，控制器和 `Service` 类的 guid 值也是不同的。

# 实例

最后一个是 Singleton 的特殊情况，用户创建对象并将其提供给`AddSingleton`方法。因此，我们明确地创建了`Example`类的对象（`services.AddSingleton(new Example(Guid.Empty))`），并要求 DI 框架将其注册为 Singleton。在这种情况下，我们发送了`Guid.Empty`。因此，一个空的 GUID 被分配，对所有请求保持不变。

# 对象销毁

当我们直接使用`Add***`方法注册服务时，容器负责创建对象和管理其生命周期。本质上，它会为实现了`IDisposable`接口的对象调用`Dispose`方法，根据生命周期进行管理。

考虑以下示例，其中`ServiceDisposable`实现了`IDisposable`，我们告诉服务将其生命周期管理为`Scoped`。因此，它将创建实例，然后在整个应用程序的资源中为单个请求提供它。最后，当请求结束时，它将销毁它：

```cs
    public class ServiceDisposable : IDisposable {}
    public void ConfigureServices(IServiceCollection services)
    {
      services.AddScoped(ServiceDisposable);
    }
```

然而，当我们显式创建对象并将其提供给 DI 时，我们需要自己处理其销毁。在以下情况下，我们必须在某个地方手动调用`Dispose()`以销毁实例：

```cs
    public class ServiceDisposable : IDisposable {}
    public void ConfigureServices(IServiceCollection services)
    {
      services.AddScoped(new ServiceDisposable());
    }
```

# 何时选择什么？

我们如何处理应用程序中的不同对象，以使它们消耗有限的内存空间和资源，同时优化性能，这一点很重要。

+   需要占用更多空间并使用大量服务器资源的对象不应被重新创建，而应重用。例如，数据库对象应重用于所有遵循`Singleton`模式的后续请求。

+   在批量或循环中运行的操作可以在特定请求中重用，但对于另一个请求则应重新创建。这表明是 Scoped 生命周期。

+   每次我们尝试创建`Model`和`View Model`类时，都应该实例化它们。在进行 CRUD 操作时，我们不能重用`Model`类对象，否则可能会导致错误的值进入数据库。当然，这是`Transient`的，因为它存在的时间很短。

# 生命周期之间的关系和依赖

在本节中，我们将深入挖掘生命周期之间的关系。原因很清楚。当我们用不同类型的生活方式实例化我们的类时，我们可能会遇到需要另一个类依赖一个类的情况。但是，这里的难点是它们可能不遵循类似的生活方式。那么，在`Singleton`类内部引用的遵循 scoped 生命周期的实例会发生什么？它是否表现得像 Scoped？

让我们动手做一些代码修改，将依赖注入到彼此中。

# 根据 Scoped 和 Transient 选择 Singleton

首先，我们需要向现有的`IExampleSingleton`接口添加两个新属性：

```cs
    public interface IExampleSingleton : IExampleService
    {
        Guid ScopedExampleId { get; }
        Guid TransientExampleId { get; }
   }
```

接下来，我们想要为所有生命周期设计一个新类。正如我们计划的，让我们通过构造函数将 Transient 和 Scoped 依赖项注入到这个`Singleton`类中。为依赖的生活方式定义的属性将从参数中相应地分配值。

```cs
    using System;
    namespace LifetimesExample
    {
      public class ExampleSingleton : IExampleSingleton
      {
        public Guid ExampleId { get; set; }
        public Guid ScopedExampleId { get; set; }
        public Guid TransientExampleId { get; set; }

        public ExampleSingleton(IExampleTransient transient, IExampleScoped scoped)
        {
            ExampleId = Guid.NewGuid();
            ScopedExampleId = scoped.ExampleId;
            TransientExampleId = transient.ExampleId;
        }
      }
      public class ExampleScoped : IExampleScoped
      {
        public Guid ExampleId { get; set; }

        public ExampleScoped()
        {
            ExampleId = Guid.NewGuid();
        }
      }
      public class ExampleTransient : IExampleTransient
      {
        public Guid ExampleId { get; set; }

        public ExampleTransient()
        {
            ExampleId = Guid.NewGuid();
        }
       }
     }
```

我只是为了这本书的可读性，在同一个地方定义了所有类。理想情况下，你应该每次都把它们添加到不同的文件中。

控制器是我们需要添加操作的下一个地方，该操作将返回一个视图，我们将展示其中的值。

```cs
    using Microsoft.AspNetCore.Mvc;
    namespace LifetimesExample.Controllers
    {
      public class ExampleController : Controller
      {
        private readonly ExampleService _exampleService;
        private readonly IExampleTransient _transientExample;
        private readonly IExampleScoped _scopedExample;
        private readonly IExampleSingleton _singletonExample;

        public ExampleController(ExampleService ExampleService,
            IExampleTransient transientExample,
            IExampleScoped scopedExample,
            IExampleSingleton singletonExample)
        {
            _exampleService = ExampleService;
            _transientExample = transientExample;
            _scopedExample = scopedExample;
            _singletonExample = singletonExample;
        }

        public IActionResult SingletonDependencies()
        {
            ViewBag.Singleton = _singletonExample;

            ViewBag.Service = _exampleService;

            return View("Singleton");
        }
      }
    }
```

这与我们在`Index`操作中做的是相似的。区别在于我们移除了`SingletonInstance`引用，并返回了一个名为`Singleton`的视图。

视图看起来可能如下所示：

```cs
  @{
    ViewData["Title"] = "Index";
  }

  @{
    IExampleSingleton singleton = (IExampleSingleton)ViewData["Singleton"];
    ExampleService service = (ExampleService)ViewBag.Service;
  }
  <h2>Singleton Lifetime Dependencies</h2>

  <h3>ExampleController</h3>

  <h5><u>Singleton ExampleId: @singleton.ExampleId</u></h5>

  <table>
    <tr>
        <th>Dependencies</th>
        <th>Guid Value</th>
    </tr>

    <tr>
        <td>Scoped Dependency</td>
        <td>@singleton.ScopedExampleId</td>
    </tr>
    <tr>
        <td>Transient Dependency</td>
        <td>@singleton.TransientExampleId</td>
    </tr>
  </table>

  <h3>ExampleService</h3>

  <h5><u>Singleton ExampleId: @service.SingletonExample.ExampleId</u></h5>

  <table>
    <tr>
        <th>Dependencies</th>
        <th>Guid Value</th>
    </tr>

    <tr>
        <td>Scoped Dependency</td>
        <td>@service.SingletonExample.ScopedExampleId</td>
    </tr>
    <tr>
        <td>Transient Dependency</td>
        <td>@service.SingletonExample.TransientExampleId</td>
    </tr>
  </table>
```

因此，我正在尝试打印`Singleton`对象的`ExampleId`以及与依赖对象（`Transient`和`Scoped`）相关的属性。我已经省略了这段代码中的样式，只是为了使表格看起来更酷。

是时候告诉`Startup` `ConfigureService`注册具有适当生命周期的类了：

```cs
public void ConfigureServices(IServiceCollection services)
{
        // Add framework services.
        services.AddMvc();

        services.AddSingleton<IExampleSingleton, ExampleSingleton>();
        services.AddScoped<IExampleScoped, ExampleScoped>();
        services.AddTransient<IExampleTransient, ExampleTransient>();

        services.AddTransient<ExampleService, ExampleService>();
}
```

哇！我们完成了。让我们检查输出。我已经并排粘贴了两个请求的截图，这样我们可以轻松地标记发现的内容：

![](img/f98f9b19-cec3-4dbe-b7ec-7aa5d7e9b4a7.png)

**观察**：由于下划线的`ExampleId`值相同，`Singleton`对象在两个请求之间被共享。

等一下！这里有些奇怪。依赖对象的值在请求之间也是相同的。注意红色块中的值。尽管这些类被注册为`Scoped`和`Transient`，但它们的行为像`Singleton`。这意味着这些对象的正常生命周期被篡改了。

**推断**：不建议在`Singleton`类内部引用`Scoped`和`Transient`生命周期类，因为它们将失去其正常行为并变成 Singleton。

显然，一个`Singleton`类可以依赖于另一个`Singleton`类。同样，其他生活方式遵循相同的规则。因此，一个`Scoped`类可以引用另一个`Scoped`类，一个`Transient`可以引用另一个`Transient`。当执行时，它们都将按预期行为。

# Scoped 依赖于 Singleton 和 Transient

同样，我们可以在`Scoped`类内部测试依赖关系。我们将从向接口`IExampleScoped`添加两个属性开始：

```cs
public interface IExampleScoped : IExampleService
{
        Guid SingletonExampleId { get; }
        Guid TransientExampleId { get; }
}
```

`ExampleScoped`现在应该实现这两个属性。同时，与`Transient`和`Singleton`相关的接口需要注入到`constructor`中：

```cs
    public class ExampleScoped : IExampleScoped
    {
        public Guid ExampleId { get; set; }
        public Guid SingletonExampleId { get; set; }
        public Guid TransientExampleId { get; set; }

        public ExampleScoped(IExampleTransient transient, IExampleSingleton singleton)
        {
          ExampleId = Guid.NewGuid();
          SingletonExampleId = singleton.ExampleId;
          TransientExampleId = transient.ExampleId;
        }
    }
```

添加了一个新的操作，它将返回名为`Scoped`的视图：

```cs
    public IActionResult ScopedDependencies()
    {
        ViewBag.Scoped = _scopedExample;

        ViewBag.Service = _exampleService;

        return View("Scoped");
    }
```

看起来我们已经完成了。让我们运行应用程序：

![](img/ded12cfc-42d1-41fa-a62f-a1699b3d40d5.png)

哎呀！我们看到了一个异常屏幕，上面显示检测到一个循环依赖。

**循环依赖**，正如其名所示，是一个依赖于另一个类的类，而另一个类又反过来依赖于第一个类。我们设计了一切来测试 Scoped 生活方式中其他生活方式的依赖，但在做之前，我们忘记了一件事。之前，我们在`Singleton`类内部添加了`Scoped`类的依赖，现在，如果你看到前面的`ExampleScoped`构造函数，我们现在注入`IExampleSingleton`，它被解析为`Singleton`类`ExampleSingleton`。这就是它变成循环的原因。

因此，我们需要从`Singleton`类中移除依赖以进行测试。我们还可以通过为`Singleton`创建另一个接口和类来进行测试。所以，当你修复代码时，我们将得到以下输出。我这里不会写`View`的代码。它与我们在`Singleton`中做的几乎一样。我们只需要打印`ExampleId`、`SingletonExampleId`和`TransientExampleId`：

![](img/157c30df-1f4d-4d51-817f-978969316d74.png)

**观察：** 在红色框中，我们有瞬态对象值。这是因为它们不再是`Transient`了，它们的行为像`Scoped`一样，因为值在请求中是相同的，这并不是`Transient`的样子。它应该在每次请求时都不同。但在`Singleton`依赖的情况下，它在请求之间是相同的，这不仅满足了`Singleton`范式，而且在特定请求中表现得像`Scoped`。

**推断：** 这就是为什么建议在`Scoped`类内部使用`Scoped`和`Singleton`依赖，而不是`Transient`。

# Transient 依赖于 Singleton 和 Scoped

对于`Transient`，我们将按照相同的模式设计所需的接口和类。接口看起来如下：

```cs
public interface IExampleTransient : IExampleService
{
        Guid SingletonExampleId { get; }
        Guid ScopedExampleId { get; }
}
```

接下来是`Transient`类依赖于`Singleton`和`Scoped`。

```cs
public class ExampleTransient : IExampleTransient
{
        public Guid ExampleId { get; set; }
        public Guid SingletonExampleId { get; set; }
        public Guid ScopedExampleId { get; set; }

        public ExampleTransient(IExampleSingleton singleton, IExampleScoped scoped)
        {
                ExampleId = Guid.NewGuid();
                SingletonExampleId = singleton.ExampleId;
                ScopedExampleId = scoped.ExampleId;
        }
}
```

最后，但同样重要的是，一个新的动作来渲染视图`Transient`：

```cs
    public IActionResult TransientDependencies()
   {
        ViewBag.Transient = _transientExample;

        ViewBag.Service = _exampleService;

        return View("Transient");
   }
```

在设计视图之后运行，最终结果可能如下所示：

![](img/964da5d3-1efd-41bf-a166-cb3cd8aa5334.png)

你可以看到`Transient` `ExampleId`在请求内外是如何不同的。

**观察：** 你可能想知道为什么这张图片中没有红色框。那是因为一切看起来都很完美。`Singleton`到处都是相同的，`Scoped`在特定请求中是相同的，然而，在下一个请求中会有变化。

**推断：** 这意味着，这两种依赖在注入到*Transient*类内部时都保留了它们通常的特性。另外，如果注入到*Transient*类，另一个*Transient*依赖肯定也能工作。

说了这么多，我想说的是，这些都是将一种生活方式注入到另一种生活方式时应遵循的推荐模式。在设计类及其生活方式时，我们应该小心。要么我们最终会陷入循环依赖，要么会失去注入到生活方式中的适当行为，正如我们在前面的例子中所看到的。只要我们意识到后果，我们仍然可以混合使用。

只用一个简单的表格，我就能表达整个要点：

![](img/99131700-c1b9-4a79-a682-50d2c5038a10.png)

# 摘要

在本章中，我们学习了.NET Framework 如何创建和销毁对象。我们讨论了创建和销毁机制。*垃圾回收器*在通过终结器自动处理中扮演着重要角色，我们通过示例进行了分析。

最重要的是，我们看到了如何通过实现`IDisposal`接口来手动处理对象的步骤分解。

之后，我们探讨了.NET Core 中对象维护的不同生命周期。我们通过控制器和服务类示例了解了对象的创建和销毁过程。最重要的是，我们实验了不同生活方式之间的适应性。

依赖注入的另一个支柱——拦截，将在第七章“拦截”中介绍。
