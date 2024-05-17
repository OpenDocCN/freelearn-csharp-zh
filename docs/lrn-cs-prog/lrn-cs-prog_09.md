# 第九章：资源管理

在之前的章节中，我们讨论并使用了值类型和引用类型，并且也看到了它们的不同之处。我们也简要讨论了运行时如何管理分配的内存。

在本章中，我们将更详细地讨论这个主题，并查看管理内存和资源的语言特性和最佳实践。

本章将讨论以下主题：

+   垃圾回收

+   终结器

+   IDisposable 接口

+   `using`语句

+   平台调用

+   不安全的代码

在本章结束时，您将学会如何实现可处理的类型以及在不再需要时如何处理对象。您还将学会如何调用本机 API 并编写不安全的代码。

# 垃圾回收

**公共语言运行时**（**CLR**）负责管理对象的生命周期，并在不再使用时释放内存，以便在进程内分配新对象。它通过一个名为**垃圾收集器**（**GC**）的组件来实现这一点，该组件以高效的方式在托管堆上分配对象，并通过回收不再使用的对象来清除内存。垃圾收集器使得开发应用程序更容易，因为您不必担心手动释放内存。这就是使为.NET 编写的应用程序被称为*托管*的原因。

在我们讨论所有这些是如何发生之前，你需要理解**栈**和**堆**之间的区别，以及**类型**、**对象**和**引用**之间的区别。

类型（无论是在 C#中使用`class`还是`struct`关键字引入的）是构造对象的蓝图。它在源代码中使用语言特性描述。对象是类型的实例化，并存在于内存中。引用是一种句柄（基本上是一个存储位置），指向一个对象。

现在，让我们讨论内存。栈是编译器分配的一个相对较小的内存段，用于跟踪运行应用程序所需的内存。栈具有**后进先出（LIFO）**语义，并且随着程序执行调用函数或从函数返回而增长和缩小。另一方面，堆是程序可能在运行时分配内存的一个大内存段，在.NET 中由 CLR 管理。

值类型的对象可以存储在多个位置。它们通常存储在栈上，但也可以存储在 CPU 寄存器上。作为引用类型的值类型存储在堆上作为*封闭对象*的一部分。引用类型的对象总是存储在堆上，但对象的引用存储在栈或 CPU 寄存器上。

为了更好地理解这一点，让我们考虑下面的短程序，其中`Point2D`是一个值类型，`Engine`是一个引用类型：

```cs
class Program
{
    static void Main(string[] args)
    {
        var i = 42;
        var pt = new Point2D(1, 2); // value type
        var engine = new Engine();  // reference type
    }
}
```

在概念上（因为这是一个非常简单的表示），栈和堆将包含以下值：

![图 9.1 - 在上述程序执行期间栈和堆内容的概念表示](img/Figure_9.1_B12346.jpg)

图 9.1 - 在上述程序执行期间栈和堆内容的概念表示

栈由编译器管理，本章的其余部分我们将讨论堆以及运行时如何管理它。.NET 运行时将对象分为两组：

+   **大型**：这些对象是大于 85 KB 的对象；多维对象也包括在此类别中。

+   **小型**：这些对象是所有其他对象。

堆由称为**代**的几个内存段组成。内存有三代 - **0**，**1**和**2**：

+   第 0 代包含*小*，通常是*短寿命的对象*，比如局部变量或在函数调用的生命周期内实例化的对象。

+   第一代包含*小对象*，它们在对第 0 代内存进行垃圾收集后幸存下来。

+   第 2 代包含*长寿命的小对象*，它们在对第 1 代内存进行垃圾收集后幸存下来，以及大对象（总是分配在这个段上）。

当运行时需要在托管堆上分配对象而内存不足时，它会触发垃圾收集。垃圾收集有三个阶段：

+   首先，垃圾收集器构建了所有活动对象的图形，以便弄清楚什么仍在使用，什么可以被删除。

+   第二，更新将被压缩的对象的引用。

+   第三，死对象被移除，幸存对象被压缩。通常，包含大对象的大对象堆不会被压缩，因为移动大块数据会产生性能成本。

当垃圾收集开始时，所有托管线程都被暂停，除了启动收集的线程。当垃圾收集结束时，线程会恢复。垃圾收集的第一阶段从所谓的**应用根**开始，这些是包含对堆上对象引用的存储位置。应用根包括对全局对象、静态对象、字段、局部对象、作为函数参数传递的对象、等待终结的对象以及包含对堆上对象引用的 CPU 寄存器的引用。

CLR 构建了可达堆对象的图形；所有不可达的对象将被删除。如果所有第 0 代对象都已经被评估，但释放的内存还不够，垃圾收集将继续评估第 1 代。如果之后需要更多内存，垃圾收集将继续评估第 2 代。

幸存下来的第 0 代垃圾收集的对象被分配到第 1 代，幸存下来的第 1 代对象被分配到第 2 代。然而，幸存下来的第 2 代垃圾收集的对象仍然留在第 2 代。如果垃圾收集过程结束后，在大对象堆上没有足够的内存（总是属于第 2 代）来分配所请求的内存，CLR 会抛出`OutOfMemoryException`类型的异常。这并不一定意味着没有更多的内存，而是这个段上未压缩的内存不包含足够大的块来存放新对象。

基类库包含一个名为`System.GC`的类，它使我们能够与垃圾收集器交互。然而，除了在本章后面将看到的*IDisposable 接口*部分中实现的可释放模式之外，这很少发生。这个类有几个成员：

![](img/Chapter_09_Table_1_01.png)

以下程序使用`System.GC`类来显示`Engine`对象的当前代数，以及调用时托管堆的估计大小：

```cs
class Program
{
    static void Main(string[] args)
    {
        var engine = new Engine("M270 Turbo", 1600, 75.0);
        Console.WriteLine(
          $"Generation of engine: 
        {GC.GetGeneration(engine)}"); 
        Console.WriteLine(
          $"Estimated heap size: {GC.
        GetTotalMemory(false)}"); 
    }
}
```

程序的输出如下：

![图 9.2 – 一个控制台截图显示了前面程序的输出](img/Figure_9.2_B12346.jpg)

图 9.2 – 一个控制台截图显示了前面程序的输出

我们将在下一节学习终结器。

# 终结器

垃圾收集器提供了托管资源的自动释放。然而，有些情况下你必须处理非托管资源，比如原始文件句柄、窗口或其他通过**平台调用服务**（**P/Invoke**）调用检索的操作系统资源，以及一些高级场景中的 COM 对象引用。这些资源在对象被垃圾收集器销毁之前必须显式释放，否则会发生资源泄漏。

每个对象都有一个特殊的方法，称为`System.Object`类有一个虚拟的受保护成员叫做`Finalize()`，带有一个空的实现。下面的代码展示了这一点：

```cs
class Object
{
    protected virtual void Finalize() {}
}
```

尽管这是一个虚方法，但你实际上不能直接重写它。相反，C#语言提供了一个与 C++中析构函数相同的语法来创建一个终结器并重写`System.Object`方法。然而，这只对引用类型实现是可能的；值类型不能有终结器，因为它们不会被垃圾收集。以下代码展示了这一点：

```cs
class ResourceWrapper
{
    // constructor
    ResourceWrapper() 
    {
        // construct the object
    }
    // finalizer
    ~ResourceWrapper()
    {
        // release unmanaged resources
    }
}
```

你不能显式重写`Finalize()`方法的原因是，C#编译器会添加额外的代码来确保在终结时实际上调用基类的实现（这意味着在继承链中的所有实例上都调用`Finalize()`方法）。因此，编译器用以下代码替换了之前显示的终结器：

```cs
class ResourceWrapper
{
    protected override void Finalize()
    {
        try
        {
            // release unmanaged resources
        }
        finally
        {
            base.Finalize();
        }
    }
}
```

尽管一个类可能有多个构造函数，但它只能有*一个终结器*。因此，终结器不能被重载或具有修饰符和参数；它们也不能被继承。终结器不会被直接调用，而是由垃圾收集器调用。

垃圾收集器调用终结器的方式如下。当创建一个具有终结器的对象时，垃圾收集器将其引用添加到一个名为*终结队列*的内部结构中。在收集对象时，垃圾收集器调用终结队列中所有对象的终结器，除非它们已经通过调用`GC.SupressFinalize()`免除了终结。这也是在应用程序域被卸载时进行的操作，但仅适用于.NET Framework；对于.NET Core 来说，情况并非如此。终结器的调用仍然是不确定的。调用的确切时刻以及调用发生的线程都是未定义的。此外，即使两个对象的终结器相互引用，也不能保证以任何特定顺序发生。

信息框

由于终结器会导致性能损失，请确保不要创建空的终结器。只有在对象必须处理未托管资源时才实现终结器。

以下代码中显示的`HandleWrapper`类是一个本机句柄的包装器。实际的实现可能更复杂；这只是为教学目的而显示的。原始句柄可能是在本机代码中创建并传递给托管应用程序。这个类拥有句柄的所有权，因此在对象不再需要时需要释放它。这是通过使用*P/Invoke*调用`CloseHandle()`系统 API 来完成的。该类定义了一个终结器来实现这一点。让我们看一下以下代码：

```cs
public class HandleWrapper
{
    [DllImport("kernel32.dll", SetLastError=true)]
    static extern bool CloseHandle(IntPtr hHandle);
    public IntPtr Handle { get; private set; }

    public HandleWrapper(IntPtr ptr)
    {
        Handle = ptr;
    }

    ~HandleWrapper()
    {
        if(Handle != default)
            CloseHandle(Handle);
    } 
}
```

很少有情况下你实际上需要创建一个终结器。对于前面提到的情景，有系统包装器可用于处理未托管资源。你应该使用以下安全句柄之一：

+   `SafeFileHandle`：文件句柄的包装器

+   `SafeMemoryMappedFileHandle`，内存映射文件句柄的包装器

+   `SafeMemoryMappedViewHandle`，一个对未托管内存块的指针的包装器

+   `SafeNCryptKeyHandle`，`SafeNCryptProviderHandle`和`SafeNCryptSecretHandle`，加密句柄的包装器

+   `SafePipeHandle`，管道句柄的包装器

+   `SafeRegistryHandle`，对注册表键句柄的包装器

+   `SafeWaitHandle`，等待句柄的包装器

如前所述，终结器仍然是不确定的。为了确保资源的确定性释放，无论是托管的还是未托管的，一个类型应该提供一个`Close()`方法或实现`IDisposable`接口。在这种情况下，终结器只能用于在未调用`Dispose()`方法时释放未托管资源。

我们将在下一节学习`IDisposable`接口。

# `IDisposable`接口

资源的确定性处理可以通过实现`System.IDisposable`接口来完成。这个接口有一个叫做`Dispose()`的方法，当一个对象不再被使用并且它的资源可以被处理时，用户可以显式调用这个方法。然而，你只应该在以下情况下实现这个接口：

+   这个类拥有*非托管资源*

+   这个类拥有*托管资源*，它们本身是可处理的

这个接口应该如何实现取决于这个类是否拥有非托管资源。当你既有托管资源又有非托管资源时，通常的模式如下：

```cs
public class MyResource : IDisposable
{
    private bool disposed = false;
    protected virtual void Dispose(bool disposing)
    {
        if (!disposed)
        {
            if (disposing)
            {
                // dispose managed objects
            }
            // free unmanaged resources
            // set large fields to null.
            disposed = true;
        }
    }
    ~MyResource()
    {
        Dispose(false);
    }
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
}
```

从`IDisposable`接口的`Dispose()`方法中，我们调用一个受保护的虚拟方法，方法名相同（尽管可以有任何名字），并且有一个参数指定对象正在被销毁。为了确保资源的处理只发生一次，使用了一个布尔字段（这里叫做`disposed`）。重载的`Dispose()`方法的布尔参数指示这个方法是由用户以确定性方式调用的，还是由垃圾收集器在对象终结时以非确定性方式调用的。

在前一种情况下，托管和非托管资源都应该被处理，并且对象的终结应该被抑制，通过调用`GC.SupressFinalize()`。在后一种情况下，只有非托管资源必须被处理，因为处理不是由用户调用的，而是由垃圾收集器调用的。这个函数是虚拟的和受保护的原因是，派生类应该能够重写它，但不应该能够直接从类外部调用它。

让我们看看如何为不同的情况实现这个。首先，我们将考虑这样一个情况，即类只有可处理的托管资源。在下面的例子中，`Engine`类实现了`IDisposable`。它具体做什么，管理什么资源，以及如何处理它们并不重要。然而，`Car`类拥有对`Engine`对象的拥有引用，这个引用应该在`Car`对象被销毁时立即销毁。此外，这应该以确定性的方式进行，当`Car`不再需要时。在这种情况下，`Car`类必须按照以下方式实现`IDisposable`接口：

```cs
public class Engine : IDisposable {}
public class Car : IDisposable
{
    private Engine engine;
    public Car(Engine e)
    {
        engine = e;
    }
    #region IDisposable Support
    private bool disposed = false;
    protected virtual void Dispose(bool disposing)
    {
        if (!disposed)
        {
            if (disposing)
            {
                engine?.Dispose();
            }
            disposed = true;
        }
    }
    public void Dispose()
    {
        Dispose(true);
    }
    #endregion
}
```

由于这个类没有终结器，重载的`Dispose()`方法在这里用处不大，代码可以进一步简化。然而，派生类可以重写它并处理更多的资源。

在前一节中，我们实现了一个叫做`HandleWrapper`的类，它有一个终结器来关闭它拥有的系统句柄。在下面的清单中，你可以看到这个类的修改版本，它实现了`IDisposable`接口：

```cs
public class HandleWrapper : IDisposable
{
    [DllImport("kernel32.dll", SetLastError = true)]
    static extern bool CloseHandle(IntPtr hHandle);
    public IntPtr Handle { get; private set; }
    public HandleWrapper(IntPtr ptr)
    {
        Handle = ptr;
    }
    private bool disposed = false; // To detect redundant calls
    protected virtual void Dispose(bool disposing)
    {
        if (!disposed)
        {
            if (disposing)
            {
                // nothing to dispose
            }
            if (Handle != default)
                CloseHandle(Handle);
            disposed = true;
        }
    }
    ~HandleWrapper()
    {
        Dispose(false);
    }
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
}
```

这个类既有一个`Dispose()`方法（可以被用户调用），又有一个终结器（在用户没有调用`Dispose()`方法的情况下，由垃圾收集器调用）。在这个例子中没有托管资源需要释放，所以重载的`Dispose()`方法的布尔参数基本上是没有用的。

语言为我们提供了一种自动处理实现`IDisposable`接口的对象的方式，当它们不再需要时。我们将在下一节中了解这个。

# using 语句

在我们介绍`using`语句之前，让我们看看如何以正确的方式进行显式资源管理。这将帮助你更好地理解`using`语句的需要和工作原理。

我们在前一节中看到的`Car`类可以这样使用：

```cs
Car car = null;
try
{
    car = new Car(new Engine());
    // use the car here
}
finally
{
    car?.Dispose();
}
```

应该使用`try-catch-finally`块（尽管这里没有明确显示`catch`）来确保在不再需要对象时正确处理对象。然而，C#语言提供了一个方便的语法来确保使用`using`语句正确处理对象的释放。它的形式如下：

```cs
using (ResourceType resource = expression) statement
```

编译器将其转换为以下代码：

```cs
{
    ResourceType resource = expression;
    try {
        statement;
    }
    finally {
        resource.Dispose();
    }
}
```

`using`语句引入了一个变量的作用域，并确保在退出作用域之前正确处理对象。实际的处理取决于资源是值类型、可空值类型、引用类型还是动态类型。之前对`resource.Dispose()`的调用实际上是以下之一：

```cs
// value types
((IDisposable)resource).Dispose();
// nullable value types or reference types
if (resource != null) 
    ((IDisposable)resource).Dispose();
// dynamic
if (((IDisposable)resource) != null) 
    ((IDisposable)resource).Dispose();
```

对于汽车示例，我们可以如下使用它：

```cs
using (Car car = new Car(new Engine()))
{
    // use the car here
}
```

多个对象可以实例化到同一个`using`语句中，如下例所示：

```cs
using (Car car1 = new Car(new Engine()),
           car2 = new Car(new Engine()))
{
    // use car1 and car2 here
}
```

另一方面，多个`using`语句可以链接在一起，如下所示，这等效于前面的代码：

```cs
using (var car1 = new Car(new Engine()))
using (var car2 = new Car(new Engine()))
{
    // use car1 and car2 here
}
```

在 C# 8 中，`using`语句可以写成如下形式：

```cs
using Car car = new Car(new Engine());
// use the car here
```

有关更多信息，请参阅*第十五章*，*C# 8 的新功能*。

# 平台调用

在本章的早些时候，我们实现了一个句柄包装类，该类使用 Windows API 函数`CloseHandle()`在对象被处理时删除系统句柄。C#程序可以调用 Windows API，也可以调用从本机**动态链接库（DLL）**导出的任何函数，都是通过**平台调用服务**，也称为**平台调用**或**P/Invoke**。

P/Invoke 定位并调用导出的函数，并在托管和非托管边界之间进行参数传递。为了能够使用 P/Invoke 调用函数，您必须知道函数的名称和签名，以及它所在的 DLL 的名称。然后，您必须创建非托管函数的托管定义。为了理解这是如何工作的，我们将看一个`user32.dll`中可用的`MessageBox()`函数的示例。函数签名如下：

```cs
int MessageBox(HWND hWnd, LPCTSTR lpText,
               LPCTSTR lpCaption, UINT uType);
```

我们可以为函数创建以下托管定义：

```cs
static class WindowsAPI
{
    [DllImport("user32.dll")]
    public static extern int MessageBox(IntPtr hWnd, 
                                        string lpText, 
                                        string lpCaption, 
                                        uint uType);
}
```

这里有几件事情需要注意：

+   托管定义的签名必须与本机定义匹配，使用等效的托管类型作为参数。

+   函数必须定义为`static`和`extern`。

+   函数必须用`DllImportAttribute`修饰。此属性为运行时调用本机函数定义了必要的信息。

`DllImportAttribute`至少需要指定从中导出本机函数的 DLL 的名称。您可以省略 DLL 中入口点的名称，此时将使用托管函数的名称来标识它。但是，您也可以使用属性的`EntryPoint`显式指定它。您可以指定的其他属性如下：

+   `BestFitMapping`：一个布尔标志，指示是否启用最佳匹配映射。在从 Unicode 到 ANSI 字符的转换时使用。最佳匹配映射使得互操作编组器在不存在精确匹配时使用最接近的字符（例如，版权字符被替换为*c*）。

+   `CallingConvention`：入口点的调用约定。默认值为`Winapi`，默认为`StdCall`。

+   `CharSet`：指定字符串参数的编组行为。它还用于指定要调用的入口点名称。例如，对于消息框示例，Windows 实际上有两个函数—`MessageBoxA()`和`MessageBoxW()`。`CharSet`参数的值使运行时能够在其中选择一个；更准确地说，以`CharSet.Ansi`结尾的名称用于`CharSet.Ansi`（这是 C#的默认值），以`CharSet.Unicode`结尾的名称用于`CharSet.Unicode`。

+   `EntryPoint`：入口点名称或序数。

+   `ExactSpelling`：指示`CharSet`字段是否确定 CLR 搜索非托管 DLL 以查找除已指定的之外的入口点名称。

+   `PreserveSig`：一个布尔标志，指示`HRESULT`或`retval`值是直接翻译（如果为`true`）还是自动转换为异常（如果为`false）。默认值为`true`。

+   `SetLastError`：如果为`true`，则表示被调用者在返回之前调用`SetLastError()`。在这种情况下，CLR 调用`GetLastError()`并缓存该值，以防止被其他 Windows API 调用覆盖和丢失。要检索该值，可以调用`Marshal.GetLastWin32Error()`。

+   `ThrowOnUnmappableChar`：指示（当为`true`时）编组器在将 Unicode 字符转换为 ANSI '`?`'时是否应抛出错误。默认值为`false`。

以下表格显示了 Windows API 和 C 风格函数中的数据类型，以及它们对应的 C#或.NET Framework 类型：

![](img/Chapter_09_Table_2_01.jpg)

重要提示

[1] 在`string`参数上使用`CharSet.Ansi`修饰或使用`[MarshalAs(UnmanagedType.LPStr)]`属性。

[2] 在`string`参数上使用`CharSet.Unicode`修饰或使用`[MarshalAs(UnmanagedType.LPWStr)]`属性。

为了能够正确调用我们之前定义的`MessageBox()`函数，我们还应该为可能的参数和返回值定义常量。下面是一个片段：

```cs
static class WindowsAPI
{
    public static class MessageButtons
    {
        public const int MB_OK = 0;
        public const int MB_OKCANCEL = 1;
        public const int MB_YESNOCANCEL = 3;
        public const int MB_YESNO = 4; 
    }

    public static class MessageIcons
    {
        public const int MB_ICONERROR = 0x10;
        public const int MB_ICONQUESTION = 0x20;
        public const int MB_ICONWARNING = 0x30;
        public const int MB_ICONINFORMATION = 0x40;
    }

    public static class MessageResult
    {
        public const int IDOK = 1;
        public const int IDYES = 6;
        public const int IDNO = 7;
    }
}
```

设置好这一切后，我们可以调用`MessageBox()`函数，如下所示：

```cs
class Program
{
    static void Main(string[] args)
    {
        var result = WindowsAPI.MessageBox(
            IntPtr.Zero, 
            "Is this book helpful?",
            "Question",
            WindowsAPI.MessageButtons.MB_YESNO | 
            WindowsAPI.MessageIcons.MB_ICONQUESTION);

        if(result == WindowsAPI.MessageResult.IDYES)
        {
            // time to learn more
        }
    }
}
```

许多 Windows API 需要使用缓冲区来返回数据。例如，`advapi32.dll`中的`GetUserName()`函数返回与当前执行线程关联的用户的名称。函数签名如下：

```cs
BOOL GetUserName(LPSTR lpBuffer, LPDWORD pcbBuffer);
```

第一个参数是一个指向字符数组的指针，用于接收用户的名称，而第二个参数是一个指向无符号整数的指针，用于指定缓冲区的大小。缓冲区需要足够大以接收用户名。否则，函数将返回`false`，在`pcbBuffer`参数中设置所需的大小，并将最后的错误设置为`ERROR_INSUFFICIENT_BUFFER`。

虽然您可以分配一个足够大的缓冲区来容纳结果（一些函数对返回值的大小施加限制），但您并不总是能确定。因此，通常，您会调用这样的函数两次：

+   首先，使用一个空缓冲区来获取实际所需的缓冲区大小

+   然后，分配必要的内存后，再次调用，使用足够大的缓冲区来接收结果

为了看到这是如何工作的，我们将 P/Invoke`GetUserName()`函数，其托管定义如下：

```cs
[DllImport("advapi32.dll", SetLastError = true,
           CharSet = CharSet.Unicode)]
public static extern bool GetUserName(StringBuilder lpBuffer,
                                      ref uint nSize);
```

请注意，我们在缓冲区参数中使用`StringBuilder`。虽然这可以增长到任何容量，但我们需要知道要指定的大小。而不是指定一个随机的大尺寸，我们调用函数两次，如下所示：

```cs
uint size = 0;
var result = WindowsAPI.GetUserName(null, ref size);
if(!result &&
   Marshal.GetLastWin32Error() ==
       WindowsAPI.ErrorCodes.ERROR_INSUFFICIENT_BUFFER)
{
    Console.WriteLine($"Requires buffer size: {size}");
    StringBuilder buffer = new StringBuilder((int)size);
    result = WindowsAPI.GetUserName(buffer, ref size);
    if(result)
    {
        Console.WriteLine($"User name: {buffer.ToString()}");
    }
}
```

在这个例子中，`StringBuffer`对象是用初始容量创建的，尽管这并不是真正必要的。您不必指定其容量；它将增长到所需的容量并接收正确的结果。

让我们总结一下平台调用服务，使用以下几点：

+   允许调用从本地 DLL 导出的函数。

+   您必须为函数创建一个托管定义，具有相同的签名和本机类型的等效托管类型。

+   在定义托管函数时，您必须至少指定函数入口点和导出 DLL 的名称。

使用 P/Invoke 时存在一些缺点，因此您应该牢记以下几点：

+   如果您使用 P/Invoke 调用 Windows API 中的函数，则您的应用程序将仅在 Windows 上运行。如果您不打算使其跨平台，这不是问题。否则，您必须完全避免这种情况。

+   如果您需要调用 C++库中的函数，您必须在导入声明中指定装饰名称，这可能会很麻烦。如果您还要编写 C++库，可以导出具有`extern "C"`链接的函数，以防止链接器对名称进行装饰。

+   在托管类型和非托管类型之间进行编组会有一些轻微的开销。

+   有时这可能不太直观；例如，指针和句柄使用什么类型。

在本章的最后一节中，我们将讨论不安全的代码和指针类型，这是 C#中的第三类类型。

# 不安全的代码

当我们讨论.NET Framework 和 C#语言支持的类型时，我们指的是值类型（结构）和引用类型（类）。然而，还有一种类型得到了支持，那就是**指针类型**。如果你不熟悉 C 或 C++编程语言，特别是指针，那么你应该知道指针就像*引用*——它们是包含对象地址的存储位置。引用基本上是由 CLR 管理的*安全指针*。

要使用指针类型，你必须建立所谓的*不安全上下文*。在 CLR 术语中，这被称为*不可验证的代码*，因为 CLR 无法验证其安全性。不安全的代码不一定是危险的，但你完全有责任确保你不会引入指针错误或安全风险。

事实上，在 C#中，有很少的情况下你实际上需要在不安全的上下文中使用指针。有两种常见的情况可能会出现这种情况：

+   调用从本机 DLL 或 COM 服务器导出的需要指针类型作为参数的函数。然而，在大多数情况下，你仍然可以使用`System.IntPtr`和`System.Runtime.InteropServices.Marshal`类型的成员来使用安全代码。

+   优化特定算法，性能至关重要。

你可以使用`unsafe`关键字定义不安全的上下文。这可以应用于以下情况：

+   类型（类、结构、接口、委托），在这种情况下，整个类型的文本上下文被视为不安全：

```cs
unsafe struct Node
{
    public int value;
    public Node* left;
    public Node* right;
}
```

+   方法、字段、属性、事件、索引器、运算符、实例和静态构造函数以及析构函数，在这种情况下，成员的整个文本上下文被视为不安全：

```cs
struct Node
{
    public int Value;
    public unsafe Node* Left;
    public unsafe Node* Right;
}
unsafe void Increment(int* value)
{
    *value += 1;
}
```

+   一个语句（块），在这种情况下，整个块的文本上下文被视为不安全：

```cs
static void Main(string[] args)
{
    int value = 42;
    unsafe
    {
        int* p = &value;
        *p += 1;
    }
    Console.WriteLine(value); // prints 43
 }
```

然而，为了能够编译使用不安全上下文的代码，你必须显式地使用`/unsafe`编译器开关。在 Visual Studio 中，你可以在**项目属性** | **构建**下的**常规**部分中勾选**允许不安全代码**选项，如下截图所示：

![图 9.3 – Visual Studio 的项目属性页面，允许启用不安全代码选项](img/Figure_9.3_B12346.jpg)

图 9.3 – Visual Studio 的项目属性页面，允许启用不安全代码选项

不安全的代码只能从另一个不安全的上下文中执行。例如，如果你有一个声明为`unsafe`的方法，你只能从不安全的上下文中调用它。这在下面的例子中得到了展示，其中不安全的`Increment()`方法（之前介绍过）从一个`unsafe`上下文中被调用。在安全的上下文中尝试这样做会导致编译错误：

```cs
static void Main(string[] args)
{
    int value = 42;
    Increment(&value);     // error
    unsafe
    {
        Increment(&value); // OK
    }
 }
```

如果你熟悉 C 或 C++，你会知道指针符号（`*`）可以放在类型旁边、变量旁边或者中间。在 C/C++中，以下都是等价的：

```cs
int* a;
int * a;
int *a;
int* a, *b; // define two variables of type pointer to int
```

然而，在 C#中，你总是在类型旁边放上`*`，就像下面的例子一样：

```cs
int* a, b; // define two variables of type pointer to int
```

变量可以是两种类型——**固定的**和**可移动的**。可移动的变量驻留在由垃圾收集器控制的存储位置中，因此可以移动或收集。固定的变量驻留在不受垃圾收集器操作影响的存储位置中。

在不安全的代码中，你可以使用`&`运算符无限制地获取固定变量的地址。然而，你只能使用固定语句来处理可移动变量。固定语句是用`fixed`关键字引入的，在许多方面类似于`using`语句。

以下是使用固定语句的一个例子：

```cs
class Color
{
    public byte Alpha;
    public byte Red;
    public byte Green;
    public byte Blue;
    public Color(byte a, byte r, byte g, byte b)
    {
        Alpha = a;
        Red = r;
        Green = g;
        Blue = b;
    }
}
static void SetTransparency(Color color, double value)
{
    unsafe
    {
        fixed (byte* alpha = &color.Alpha)
        {
            *alpha = (byte)(value * 255);
        }
    }
}
```

`SetTransparency()`函数使用指向`Alpha`字段的指针来更改`Color`对象的 alpha 值。尽管这是值类型的`byte`类型，但它位于托管堆上，因为它是引用类型的一部分。垃圾回收器可能会在访问`Alpha`字段之前移动或收集`Color`对象。因此，检索其地址的唯一可能方法是使用`fixed`语句。这基本上固定了托管对象，以便垃圾回收器不会移动或收集它。

除了`usafe`和`fixed`，还有两个关键字可以在不安全的上下文中使用：

+   `stackalloc`用于声明在调用堆栈上分配内存的变量（类似于 C 中的`_alloca()`）：

```cs
static unsafe void AllocArrayExample(int size)
{
    int* arr = stackalloc int[size];
    for (int i = 1; i <= size; ++i)
    arr[i] = i;
}
```

+   `sizeof`用于获取值类型的字节大小。对于原始类型和枚举类型，`sizeof`运算符实际上也可以在安全的上下文中调用：

```cs
static void SizeOfExample()
{
    unsafe
    {
        Console.WriteLine(
          $"Pointer size: {sizeof(int*)}");
    }
}
```

让我们通过查看以下关键点来总结不安全代码：

+   它只能在不安全的上下文中执行，使用`unsafe`关键字在使用`/unsafe`开关编译时引入。

+   类型、成员和代码块可以是不安全的上下文。

+   它引入了安全性和稳定性风险，你需要对此负责。

+   只有极少数情况下需要使用它。

# 总结

本章重点介绍了运行时（通过垃圾回收器）如何管理对象和资源的生命周期。我们学习了垃圾回收器的工作原理，以及如何编写终结器来处理本机资源。我们已经看到了如何正确实现`IDisposable`接口和`using`语句的模式，以确定性地释放对象。我们还研究了平台调用服务，它使我们能够从托管代码中进行本机调用，以及编写不安全的代码——这是 CLR 无法验证安全性的代码。

在本书的下一章中，我们将研究不同的编程范式，函数式编程，并了解它在 C#中的关键概念以及它们能够让我们做什么。

# 测试你学到的东西

1.  栈和堆是什么？每个上面分配了什么？

1.  堆的内存段是什么，每个上面分配了什么？

1.  垃圾回收是如何工作的？

1.  终结器是什么？处理和终结之间有什么区别？

1.  `GC.SupressFinalize()`方法是做什么的？

1.  `IDisposable`是什么，何时应该使用它？

1.  `using`语句是什么？

1.  你如何在 C#中从本机 DLL 调用函数？

1.  不安全代码是什么，它通常在哪些场景中使用？

1.  你可以声明哪些程序元素为不安全？

# 进一步阅读

+   *垃圾回收：Microsoft .NET Framework 中的自动内存管理*，Jeffrey Richter – MSDN Magazine: [`docs.microsoft.com/en-us/archive/msdn-magazine/2000/november/garbage-collection-automatic-memory-management-in-the-microsoft-net-framework`](https://docs.microsoft.com/en-us/archive/msdn-magazine/2000/november/garbage-collection-automatic-memory-management-in-the-microsoft-net-framework)

+   *垃圾回收：第二部分：Microsoft .NET Framework 中的自动内存管理*，Jeffrey Richter – MSDN Magazine: [`docs.microsoft.com/en-us/archive/msdn-magazine/2000/december/garbage-collection-part-2-automatic-memory-management-in-the-microsoft-net-framework`](https://docs.microsoft.com/en-us/archive/msdn-magazine/2000/december/garbage-collection-part-2-automatic-memory-management-in-the-microsoft-net-framework)
