# 第十八章：模拟测试 2

1.  您需要编写一个应用程序，在该应用程序中创建一个类，该类与 SQL Server 建立连接并读取某个表中的记录。我们需要确保类中的以下内容：

    +   类应在操作完成后自动释放所有连接。

    +   类应支持迭代。

在类中，您会实现以下哪个接口？

+   1.  `IEnumerator`

    1.  `IEquatable`

    1.  `IComparable`

    1.  `IDisposable`

1.  如果您需要编写一个可以接受可变数量参数的函数，您会使用什么？

+   1.  接口

    1.  方法重写

    1.  方法重载

    1.  Lambda 表达式

1.  您正在编写一个需要反转字符串的应用程序。您会使用以下哪个代码片段？

    1.  `char[] characters = str.ToCharArray();`

        `for (int start = 0, end = str.Length - 1; start < end; start++, end--)`

        `{`

        `characters[end] = str[start];`

        `characters[start] = str[end];`

        `}`

        `string reversedstring = new string(characters);`

        `Console.WriteLine(reversedstring);`

    1.  `char[] characters = str.ToCharArray();`

        `for (int start = 0, end = str.Length - 1; start < end; start++, end--)`

        `{`

        `characters[start] = str[end];`

        `characters[end] = str[start];`

        `}`

        `string reversedstring = new string(characters);`

        `Console.WriteLine(reversedstring);`

    1.  `char[] characters = str.ToCharArray();`

        `for (int start = 0, end = str.Length; start < end; start++, end--)`

        `{`

        `characters[start] = str[end];`

        `characters[end] = str[start];`

        `}`

        `string reversedstring = new string(characters);`

        `Console.WriteLine(reversedstring);`

    1.  `char[] characters = str.ToCharArray();`

        `for (int start = 0, end = str.Length; start < end; ++start, end--)`

        `{`

        `characters[start] = str[end];`

        `characters[end] = str[start];`

        `}`

        `string reversedstring = new string(characters);`

        `` `Console.WriteLine(reversedstring);` ``

1.  如果您需要比较应用程序不同构建版本的内存使用情况，您会使用 Visual Studio 的以下哪个功能？

    1.  智能感应

    1.  使用性能分析器的 CPU 使用情况

    1.  使用性能分析器的内存使用情况

    1.  使用性能分析器的 UI 分析

1.  您会使用以下哪个正则表达式来确保正在验证的输入是非负十进制数？

    1.  `^(?!\D+$)\+?\d*?(?:\.\d*)?$`

    1.  `^(?:[1-9]\d*|0)?(?:\.\d+)?$`

    1.  `^\d+(\.\d\d)?$`

    1.  `^(-)?\d+(\.\d\d)?$`

1.  我们正在开发一个应用程序，我们正在使用一个名为 X 的程序集。如果我们需要调试程序集中的代码，我们应该做什么？

    1.  对于应用程序，在项目构建属性中，设置允许不安全代码属性。

    1.  对于应用程序，在调试窗格中，在调试选项中，设置启用本地代码和继续。

    1.  对于应用程序，在调试窗格中，在调试选项中，取消选中启用仅我的代码。

    1.  对于应用程序，在项目调试属性中，选择启动外部程序单选按钮并选择程序集 X。

1.  我们正在创建一个包含`Student`类的应用程序。在应用程序中，我们还声明了一个名为`students`的变量。以下哪个语句用于检查`students`变量是否为`Student`类型对象的列表？

    1.  `if(students.GetType() is List<Student>[])`

    1.  `if(students.GetType() is List<Student>)`

    1.  `if(students is List<Student>[])`

    1.  `` `if(students is List<Student>)` ``

1.  以下哪个代码段不会导致数据丢失？

    1.  `public void AddDeposit(float deposit)`

        `{`

        `AddToActBalance(Convert.ToDouble(deposit));`

        `}`

        `public void AddToActBalance(Double deposit)`

        `{`

        `}`

    1.  `public void AddDeposit(float deposit)`

        `{`

        `AddToActBalance(Convert.ToDecimal(deposit));`

        `}`

        `public void AddToActBalance(Decimal deposit)`

        `{`

        `}`

    1.  `public void AddDeposit(float deposit)`

        `{`

        `AddToActBalance(Convert.ToInt32(deposit));`

        `}`

        `public void AddToActBalance(int deposit)`

        `{`

        `}`

    1.  `public void AddDeposit(float deposit)`

        `{`

        `AddToActBalance((Decimal)(deposit));`

        `}`

        `public void AddToActBalance(Decimal deposit)`

        `{`

        `}`

1.  我们正在编写一个应用程序，需要将一些文本写入文件。此代码的语法如下：

```cs
public async void PerformFileWriteOperation()
{
    string path = @"InputFile.txt";
    string text = "Text to read\r\n"
    await PerformFileUpdateAsync(path, text);
}
private async Task PerformFileUpdateAsync(string path, string textToUpdate)
{
    byte[] encodedBits = Encoding.Unicode.GetBytes(textToUpdate);
    using(FileStream stream = new FileStream(
    path, FileMode.Append, FileAccess.Write, FileShare.None, bufferSize: 4096, useAsync: true))
    {
        /// Insert the code here
    }
}
```

以下哪一行代码应该插入到前面的代码中，以确保在文件操作进行之前不会停止执行？

+   1.  `async stream.Write(encodedBits, 0, encodedBits.Length);`

    1.  `await stream.Write(encodedBits,0, encodedBits.Length);`

    1.  `async stream.WriteAsync(encodedBits,0, encodedBits.Length);`

    1.  `await stream.WriteAsync(encodedBits,0, encodedBits.Length);`

1.  我们正在编写一个应用程序，其中已经编写了以下代码：

```cs
public class Car
{
     public Car()
     {
         Console.WriteLine("Inside Car");
     }
     public void Accelerate()
     {
         Console.WriteLine("Inside Acceleration of Car");
     }
 }
 public class Ferrari : Car
 {
     public Ferrari()
     {
         Console.WriteLine("Inside Ferrari");
     }
     public void Accelerate()
     {
         Console.WriteLine("Inside Acceleration of Ferrari");
     }
 }

class Program
{
     static void Main(string[] args)
     {
         Car b = new Ferrari();
         b.Accelerate();
     }
}
```

程序的输出会是什么？

+   1.  编译时错误

    1.  运行时错误

    1.  `Inside Acceleration of Ferrari`

    1.  `Inside Acceleration of Car`

1.  我们有一个应用程序，在其中我们使用`while`循环编写了以下逻辑：

```cs
int i = 1;
while(i < 10)
{ 
    Console.WriteLine(i);
    ++i;
}

```

你会如何将其转换为等效的`for`循环？

+   1.  `for (int i = 0; i < 10 ; i++)`

        `{`

        `Console.WriteLine(i);`

        `}`

    1.  `for (int i = 1; i < 10 ; i++)`

        `{`

        `Console.WriteLine(i);`

        `}`

    1.  `for (int i = 1; i < 10 ; ++i)`

        `{`

        `Console.WriteLine(i);`

        `}`

    1.  `for (int i = 1; i <= 10 ; i++)`

        `{`

        `Console.WriteLine(i);`

        `` `}` ``

1.  以下程序的输出会是什么？

```cs
try
{
    int[] input = new int[5] { 0, 1, 2, 3, 4 };
    for (int i = 1; i <= 5; i++)
    {
         Console.Write(input[i]);
    }
}
catch (System.IndexOutOfRangeException e)
{
    System.Console.WriteLine("An error has occured in collection operation");
    throw;
}
catch (System.NullReferenceException e)
{
    System.Console.WriteLine("An error has occured in null reference operation");
    throw;
}
catch (Exception e)
{
    System.Console.WriteLine("Error logged for the application");
}
```

+   1.  `01234`

    1.  `1234`

        `集合操作中发生错误`

    1.  `1234` `集合操作中发生错误` `应用程序已记录错误`

    1.  `1234` `集合操作中发生错误` `空引用操作中发生错误` `应用程序已记录错误`

1.  委托可以通过以下方式实例化：

    1.  匿名方法

    1.  Lambda 表达式

    1.  命名方法

    1.  所有上述选项

1.  在挂起状态下，将线程移动到运行状态需要执行哪些操作？

    1.  恢复

    1.  中断

    1.  中断

    1.  挂起

1.  以下程序的输出是什么？

```cs
public class DisposeImplementation : IDisposable
{
     private bool isDisposed = false;
     public DisposeImplementation()
     {
         Console.WriteLine("Creating object of DisposeImplementation");
     }
     ~DisposeImplementation()
     {
         if(!isDisposed)
         {
             Console.WriteLine("Inside the finalizer of class DisposeImplementation");
             this.Dispose();
         }
     }
     public void Dispose()
     {
         isDisposed = true;
         Console.WriteLine("Inside the dispose of class DisposeImplementation");

     }
 }

DisposeImplementation d = new DisposeImplementation();
d.Dispose();
d = null;
GC.Collect();
Console.ReadLine();
```

+   1.  `Creating object of DisposeImplementation`

        `Inside the dispose of class DisposeImplementation`

        `Inside the finalizer of class DisposeImplementation`

    1.  `创建 DisposeImplementation 对象`

        `在 DisposeImplementation 类的处置内部`

    1.  应用程序中的运行时错误

    1.  `创建 DisposeImplementation 对象`

        `在 DisposeImplementation 类的终结器内部`

        `在 DisposeImplementation 类的处置内部`

1.  看看以下程序：

```cs
static int CalculateResult(int parameterA, int parameterB, int parameterC, int parameterD = 0)
{
    int result = ((parameterA + parameterB) / (parameterC - parameterD));
    return result;
}
```

当使用以下语法调用时，输出会是什么？

```cs
CalculateResult(parameterA: 20, 15, 5)
```

+   1.  `-7`

    1.  `1`

    1.  `7`

    1.  运行时错误

1.  哪些可以用来验证用户输入？

    1.  对称算法

    1.  非对称算法

    1.  哈希值

    1.  数字签名

1.  我们正在处理一组大量的学生对象。您需要使用一种数据结构，它允许以任何顺序访问元素，并且允许不需要将它们分组在特定键下时重复值。您会选择什么？

    1.  列表

    1.  栈

    1.  字典

    1.  队列

1.  您的应用程序正在运行多个工作线程。您如何确保应用程序等待所有线程完成它们的执行？

    1.  `Thread.Sleep()`

    1.  `Thread.WaitAll()`

    1.  `Thread.Join()`

    1.  无

1.  在一个应用程序中，我们在一个类中编写一个方法，该方法应该可以被同一类中的类以及继承自该类的同一程序集中的类访问。你需要什么？

+   1.  私有受保护

    1.  受保护内部

    1.  受保护

    1.  内部
