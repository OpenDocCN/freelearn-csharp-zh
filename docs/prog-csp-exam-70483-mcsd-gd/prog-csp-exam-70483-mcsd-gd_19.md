# 第十九章：模拟测试 3

1.  在您的应用程序中，您实现了`LogException(string message)`方法来记录异常。当您的应用程序抛出异常时，您想记录并重新抛出原始异常。您如何实现这一点？

    1.  `catch(Exception ex){LogException(ex.Message); throw;}`

    1.  `catch(Exception ex){LogException(ex.Message); throw ex;}`

    1.  `catch{LogException(ex.Message); throw new Exception();}`

    1.  `catch{LogException(ex.Message); rethrow;}`

1.  您创建了一个应用程序，在其中实现了自定义异常类型，并实现了多个日志方法，如下所示：

```cs
public class CustomException1 : System.Exception{}
public class CustomException2 : CustomException1 {}
public class CustomException3 : CustomException1{}

void Log(Exception ex){}
void Log(CustomException2 ex) {}
void Log(CustomException3 ex) {}
```

您有一个可能会抛出上述任一异常的方法。您需要确保，当捕获到异常时，通过日志方法记录异常信息；当捕获到`CustomException2`时，日志方法接受`CustomException2`；对于`CustomException3`也是如此。您希望如何实现这一点？请指定`catch`语句的顺序：

+   1.  `catch(CustomException1 ex){...}`

        `catch(CustomExceotion2 ex){...}`

        `catch(CustomException3 ex){...}`

    1.  `catch(Exception ex){...}`

        `catch(CustomExceotion2 ex){...}`

        `catch(CustomException3 ex){...}`

    1.  `catch(Exception ex){...}`

        `catch(CustomExceotion1 ex){...}`

    1.  `catch(CustomException3 ex){...}`

        `catch(CustomExceotion2 ex){...}`

        `catch(Exception ex){...}`

1.  您的应用程序正在使用任务工厂运行多个任务。然而，一位客户要求您在父任务抛出异常时运行特定任务。您如何实现这一点？

    1.  `task.when()`

    1.  `task.whenany()`

    1.  `task.continuewhenany()`

    1.  以上都不是

1.  您的应用程序正在运行多个工作线程。您如何确保应用程序等待所有线程完成执行？

    1.  `Thread.Sleep()`

    1.  `Thread.WaitAll()`

    1.  `Thread.Join()`

    1.  以上都不是

1.  秘密密钥加密也称为非对称加密。

    1.  `True`

    1.  `` `False` ``

1.  在公钥加密中，任何拥有公钥的人都可以处理消息。

    1.  `True`

    1.  `False`

1.  在您的示例应用程序中使用`RSACryptoServiceProvider`时，您如何获取您的公钥和私钥？

    1.  `RSACryptoServiceProvider rsa = new RSACryptoServiceProvider();`

        `string publicKey = rsa.ToXmlString(false);`

        `string pricateKey = rsa.ToXmlString(true);`

    1.  `RSACryptoServiceProvider rsa = new RSACryptoServiceProvider();`

        `string publicKey = rsa.ToXmlString(true);`

        `string pricateKey = rsa.ToXmlString(false);`

    1.  `RSACryptoServiceProvider rsa = new RSACryptoServiceProvider();`

        `string publicKey = rsa.ToXmlString(public);`

        `string pricateKey = rsa.ToXmlString(private);`

    1.  `RSACryptoServiceProvider rsa = new RSACryptoServiceProvider();`

        `string publicKey = rsa.ToXmlString("public");`

        `string pricateKey = rsa.ToXmlString("private");`

1.  最佳的验证发送者的方式是什么？

    1.  加密您的消息。

    1.  签署您的消息。

    1.  使用数字签名。

    1.  以上所有。

1.  当您对一个字符串应用哈希算法时，输出会是什么？

    1.  字符串被加密。

    1.  每个字符都被哈希到一个不同的二进制字符串。

    1.  字符串被整体哈希。

    1.  以上都不是。

1.  您正在为您的客户向现有应用程序添加新功能。当您部署它们时，您得到一个程序集清单不匹配错误。解决此问题的最佳可能解决方案是什么？

    1.  更新当前和依赖程序集的所有主要和次要版本，然后重新构建和部署。

    1.  检查当前和依赖程序集的所有程序集版本，并确保配置或策略已更新以反映程序集版本的更改，然后重新构建和部署。

    1.  更新`machine.config`文件以忽略此类错误。

    1.  更新`web.config`并将自定义错误模式设置为`off`。

1.  您创建了一个发布包并将应用程序部署到生产环境。当用户开始使用应用程序时，他们收到一个错误。您无法在任何较低级别的环境中重现它，因此您决定在生产环境中调试您的应用程序。然而，应用程序从未在断点处停止。这是为什么？

    1.  您在系统上没有本地管理员权限。

    1.  Visual Studio 的调试模块未加载。

    1.  发布版本不允许我们调试。

    1.  以上所有。

1.  您创建了一个应用程序，您想在它执行时监控它。因此，您决定实现跟踪。您如何跟踪您的应用程序，以便您可以在输出窗口中看到您的跟踪消息？

    1.  使用`Console.WriteLine()`。

    1.  使用`tracelistener`添加输出窗口并使用`trace.write`。

    1.  `Debug.WriteLine()`。

    1.  `Output.WriteLine()`。

1.  您正在创建一个应用程序，其中有一个`if`语句和一个`else`语句。在`if`语句中，您有两个条件。您想在执行代码块之前验证这两个条件。您如何实现这一点？

    1.  使用`&&`运算符。

    1.  使用`&`运算符。

    1.  使用`|`运算符。

    1.  使用`||`运算符。

1.  当您的表达式返回 null 值时，您如何将默认值返回到变量中？

    1.  使用`ternary`运算符。

    1.  使用`binary`运算符。

    1.  使用条件`OR`。

    1.  使用`null`合并运算符。

1.  您的代码中有多个相同方法的版本。您的客户要求您确保所有依赖的应用程序都使用该方法的特定版本。您如何确保没有人调用任何可能引起其他异常的其他方法？

    1.  更改所有其他方法的访问修饰符。

    1.  从这些方法中抛出异常。

    1.  使用`Obsolete`属性让用户知道正确的使用方法。

    1.  以上所有。

1.  您正在创建一个 C#应用程序，您需要输出多行，每行之间有一个换行符。您如何实现这一点？

    1.  `var sb = new StringBuilder();sb.AppendLine(Line1); sb.AppendLine(Environment.NewLine);`

    1.  `var sb = new StringBuilder();foreach(string line in strList){sb.AppendLine(Line1); sb.AppendLine(Environment.NewLine); }`

    1.  `var sb = new StringBuilder();sb.Append(Line1); sb.Append('\t');`

    1.  以上皆是

1.  你如何确保父类方法在继承类中不可访问？你将使用哪种访问修饰符？

    1.  私有的

    1.  内部

    1.  受保护的

    1.  摘要

1.  指定在运行时加载程序集的代码：

    1.  `Assembly.Load()`

    1.  `Assembly.Create("A1.dll");Assembly.Load();`

    1.  `Assembly.Load("a.dll");`

    1.  `Assembly.GetType().Load();`

1.  当你创建一个 C#控制台应用程序时，你在解决方案资源管理器中看到哪些文件？

    1.  项目、`App.Config`、`Program.cs`、解决方案、属性和引用

    1.  `App.Config`、`Program.cs`

    1.  项目、`App.Config`、`Program.cs`、解决方案、属性

    1.  `App.Config`、`Program.cs`、属性

1.  当你创建一个控制台应用程序并将`static Main(string[] args)`更改为`static main(string[] args)`时，会发生什么？

    1.  会引发编译时错误。

    1.  会引发运行时错误。

    1.  **a**和**b**都正确。

    1.  以上皆非。

1.  声明为内部（internal）的类或类成员只能被同一程序集中的类访问，而不能被外部程序集访问。

    1.  正确

    1.  错误

1.  考虑以下陈述。**陈述 1**：值类型保持变量的地址。**陈述 2**：指向地址 1 的两个引用类型变量反映了更新的值。

    1.  两个都是正确的。

    1.  陈述 1 是正确的，陈述 2 是错误的。

    1.  陈述 1 是错误的，陈述 2 是正确的。

    1.  两个都是错误的。

1.  在定义接口时，为方法使用访问修饰符是一种良好的做法。

    1.  正确

    1.  错误

1.  如何定义可选参数？

    1.  `void AddNumbers(int a=1, int b)`

    1.  `void AddNumbers(int a, int b optional)`

    1.  `void AddNumbers(int a, int b=4)`

    1.  `void Add numbers(int a, optional int b)`

1.  在使用指针声明的程序函数中，你使用的关键字是什么？

    1.  密封的

    1.  安全的

    1.  内部

    1.  不安全的

1.  我们使用什么语法向文件追加文本？

    1.  `File.CreateText`

    1.  `FileInfo.Create`

    1.  `File.Create`

    1.  `File.AppendText`

1.  哪种集合类型可以用来创建一个强类型、基于零的索引，以 FIFO 方式处理对象？

    1.  `Queue<T>`

    1.  `List<T>`

    1.  `Array`

    1.  `Dictionary`

1.  你正在创建一个管理信息的应用程序。你在类中定义了一个保存方法，并希望确保只有这个类及其继承类可以调用该方法。你希望将保存方法定义为强类型方法。你如何实现这一点？

    1.  `public static void Save<T>(T target) where T : new(), ParentClass {}`

    1.  `public static void Save<T>(T target) where T : ParentClass,new() {}`

    1.  `public static void Save<T>(T target) where T : ParentClass {}`

    1.  `public static void Save(ParentClass target) {}`

1.  你正在开发一个将被多个应用程序使用的程序集。你需要将其安装到 GAC 中。你将执行哪些操作来实现这一点？

    1.  对程序集进行签名并使用 Gacutil 工具将程序集安装到 GAC。

    1.  版本化程序集并使用 Regsvr32 工具将程序集安装到 GAC。

    1.  拖放到 Windows 程序集文件夹。

    1.  以上皆是。

1.  当两个实体需要使用非对称算法进行通信时，他们需要共享哪个密钥？

    1.  私钥

    1.  公钥

    1.  两者

    1.  无
