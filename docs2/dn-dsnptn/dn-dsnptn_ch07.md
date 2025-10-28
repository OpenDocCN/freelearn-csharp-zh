# 第七章. .NET 基础类库中的模式

在本章中，我们将关注.NET Framework 库的创建者如何利用 GoF 设计模式来创建**应用程序编程接口**（**APIs**）。这将使你对设计模式作为暴露定义良好的软件接口的机制有巨大的洞察力。大多数开发者消费基于设计模式的软件接口，而不太了解其基础。本章将揭示基于模式的 API 设计。作为读者，你将在.NET Framework 库中找到以下 GoF 模式的实际应用：

+   适配器模式

+   策略模式

+   构建者模式

+   装饰者模式

+   责任链模式

+   桥接模式

+   工厂模式

+   观察者模式

+   组合模式

+   外观模式

+   迭代器模式

# .NET BCL 中的适配器模式

适配器模式（也称为包装器）将一个类的接口转换为客户端期望的兼容接口。这允许对象协同工作，而这些对象通常因为接口不兼容而无法协同工作。

### 注意

这通过向客户端提供其协作接口，同时使用原始接口调用对象提供的核心功能来实现。

实现这一点的代码量通常很小。适配器还负责将数据转换为库期望的适当形式，该库实现了实际逻辑。适配器可以是类适配器或对象适配器。.NET Framework 中的`SQLDataAdapter`类代表一组数据命令（`select`、`update`和`delete`）和数据库连接，用于填充`DataSet`（在`select`的情况下），或更新数据源：

```cs
    private static RetrieveRows( 
      string connectionString,string queryString)  
      { 
        using (SqlConnection connection =  
        new SqlConnection(connectionString)) 
        { 
          //--- Adapter class here creates a DataSet  
          //--- without bothering the developer about 
          //--- opening the connections, fetching data 
          //--- from the Data Reader and Filling the Data 
          //--- to a DataSet  
          SqlDataAdapter adapter = new SqlDataAdapter(); 
          adapter.SelectCommand = new SqlCommand( 
          queryString, connection); 
          adapter.Fill(dataset); 
          return dataset; 
        } 
      } 

```

# .NET BCL 中的策略模式

策略模式（也称为策略模式）是一种设计模式，其中我们可以根据上下文选择算法。该模式旨在提供一种定义算法家族的方法，这些算法封装为对象以使它们可互换。策略模式允许算法独立于使用它们的客户端而变化。

`Array`和`ArrayList`类通过`Sort`方法提供了对它们包含的对象进行排序的能力。可以通过利用基于策略设计模式的 API 来使用不同的策略进行排序。.NET Framework 的设计者为我们提供了`IComparer<T>`接口来提供排序策略。`Array`和`ArrayList`通过`Sort`方法提供了对集合中包含的对象进行排序的能力。策略设计模式与`Array`和`ArrayList`一起使用，以启用使用不同策略的排序，而无需更改任何客户端代码，通过`IComparable`接口：

```cs
    class Employee  
    { 
      public String name {get;set;} 
      public int age {get;set;} 
      public double salary { get; set; }  
    } 

```

要对 `Employee` 列表进行排序，我们需要创建一个实现 `IComparer<T>` 接口的类。这个类的实例需要传递给 `List<T>` 实例的 `Sort` 例程。`IComparer<T>` 的 `Compare` 方法应该实现一个 `SIGNUM` 函数，该函数返回 `1`、`0` 或 `-1`：

```cs

class SortByAge : IComparer<Employee> 
    { 
      public int Compare(Employee a, Employee b) 
      { 
        return a.age > b.age ? 1 : -1; 
      } 
    } 

```

以下代码片段展示了我们如何根据上下文更改排序标准：

```cs
    List<Employee> ls = new List<Employee>(); 
    ls.Add(
       new Employee { name = "A", age = 40, salary =  10000 });
    ls.Add(
       new Employee { name = "C", age = 20, salary = 6000 });
    ls.Add(
       new Employee { name = "B", age = 30, salary = 4000  }); 
    ls.Sort(new SortByAge()); 
    foreach(Employee e in ls) 
    { 
      Console.WriteLine(e.name + " " +  
      e.age + " " + e.salary); 
    } 

```

要按名称排序，我们需要创建另一个实现相同接口的对象：

```cs
    class SortByName : IComparer<Employee> 
    { 
      public int Compare(Employee a, Employee b) 
      { 
        return a.name.CompareTo(b.name); 
      } 
    } 

```

我们将利用 `SortByName` 类根据字典顺序对对象进行排序。我们还将使用 `StringBuilder` 类来创建一个要在控制台上打印的字符串对象。`StringBuilder` 类是 .NET 框架设计者利用构建器模式的一个实例：

```cs
    ls.Sort(new SortByName()); 
    foreach(Employee e in ls) 
    { 
      StringBuilder strbuilder = new StringBuilder(); 
      strbuilder.Append(e.name); 
      strbuilder.Append(" "); 
      strbuilder.Append(e.age); 
      strbuilder.Append(" "); 
      strbuilder.Append(e.salary); 
      Console.WriteLine(strbuilder.ToString()); 
    } 

```

# .NET BCL 中的构建器模式

构建器模式是一种创建型模式，它将复杂对象的构建与其表示分离。通常，它解析复杂的表示以创建一个或多个目标对象。大多数情况下，构建器创建复合对象。在 `System.Data.SqlClient` 命名空间中，`SqlConnectionStringBuilder` 帮助构建连接字符串，用于连接到 RDBMS 引擎：

```cs
    SqlConnectionStringBuilder builder = new   
    SqlConnectionStringBuilder(); 
    builder["Data Source"] = "(local)"; 
    builder["integrated Security"] = true; 
    builder["Initial Catalog"] = "AdventureWorks;NewValue=Bad"; 
    Console.WriteLine(builder.ConnectionString); 

```

.NET BCL 也包含一个类，可以帮助我们通过组装其组成部分来创建 URI。以下代码片段创建了一个安全的 HTTP (`https`) URL，该 URL 将数据发送到端口 `3333`：

```cs
    var builder = new UriBuilder(url); 
    builder.Port = 3333 
    builder.Scheme = "https"; 
    var result = builder.Uri; 

```

# .NET BCL 中的装饰器模式

装饰器模式动态地为对象附加额外的职责。继承并不总是可行的，因为它总是静态的，并且适用于整个类。装饰器为扩展功能提供了对子类化的灵活替代方案。该模式有助于在运行时向单个对象添加行为或状态。.NET 框架在流处理类的情况下使用了装饰器。流处理类的层次结构如下：

+   `System.IO.Stream`

    +   `System.IO.BufferedStream`

    +   `System.IO.FileStream`

    +   `System.IO.MemoryStream`

    +   `System.Net.Sockets.NetworkStream`

    +   `System.Security.Cryptography.CryptoStream`

以下代码片段展示了如何使用 `FileStream` 从操作系统磁盘文件中读取内容：

```cs
    using (FileStream fs = new FileStream(path, FileMode.Open))  
    { 
      using (StreamReader sr = new StreamReader(fs))  
      { 
        while (sr.Peek() >= 0)  
        { 
          Console.WriteLine(sr.ReadLine()); 
        } 
      } 
    } 

```

`StreamReader` 是一个装饰器对象，它使用额外的缓冲功能来避免磁盘访问，从而加快操作速度。

# ASP.net 中的责任链模式

责任链模式是一种设计模式，它由一系列处理对象组成，我们通过这些对象传递数据流进行过滤或修改。最终，当数据流通过链尾的最后处理对象时，处理过程终止。ASP.NET 管道是一个很好的例子，其中利用了责任链模式来提供一个可扩展的编程模型。ASP.NET 基础设施通过 HTTP 模块和处理器实现了 WebForms API、ASMX Web 服务、WCF、ASP.NET Web API 和 ASP.NET MVC。管道中的每个请求在到达目标处理器（实现 `IHttpHandler` 的类）之前都会通过一系列模块（实现 `IHttpModule` 的类）。一旦管道中的模块完成了它的任务，它将请求处理的责任传递给链中的下一个模块。最后，它到达处理器。以下代码片段展示了如何编写一个利用责任链模式创建过滤传入请求的模块的对象。这些过滤器配置为链，并由 ASP.net 运行时将请求内容传递给链中的下一个过滤器：

```cs
    public class SimpleHttpModule : IHttpModule 
    { 
      public SimpleHttpModule(){} 
      public String ModuleName 
      { 
        get { return "SimpleHttpModule"; } 
      } 
      public void Init(HttpApplication application) 
      { 
        application.BeginRequest +=  
        (new EventHandler(this.Application_BeginRequest)); 
        application.EndRequest +=  
        (new EventHandler(this.Application_EndRequest)); 
      } 
      private void Application_BeginRequest(Object source,  
      EventArgs e) 
      { 
        HttpApplication application = (HttpApplication)source; 
        HttpContext context = application.Context; 
        context.Response.Write(SomeHtmlString); 
      } 
      private void Application_EndRequest(Object source, EventArgs e) 
      { 
        HttpApplication application =      (HttpApplication)source; 
        HttpContext context = application.Context; 
        context.Response.Write(SomeHtmlString); 
      } 
      public void Dispose(){} 
    } 

```

我们可以在 `Web.config` 文件中配置前面的 HTTP 模块如下：

```cs
    <configuration> 
      <system.web> 
        <httpModules> 
          <add name=" SimpleHttpModule " type=" SimpleHttpModule "/> 
        </httpModules> 
      </system.web> 
    </configuration> 

```

在 ASP.NET 管道中，一个请求在到达处理器之前会通过一系列 HTTP 模块。以下是一个简单的 HTTP 处理器例程：

```cs
    public class SimpleHttpHandler: IHttpHandler 
    { 
      public void ProcessRequest(System.Web.HttpContext context){ 
        context.Response.Write("The page request ->" +          
        context.Request.RawUrl.ToString()); 
      } 
      public bool IsReusable 
      { 
        get{ return true; } 
      } 
    } 

```

我们可以按以下方式配置处理器。每次我们创建一个具有 `.smp` 扩展名的 ASP.NET 资源时，处理器将是 `SimpleHttpHandler`：

```cs
    <system.web> 
      <httpHandlers> 
        <add verb="*" path="*.smp" type="SimpleHttpHandler"/> 
      </httpHandlers> 
    </system.web> 

```

之前利用责任链模式的技术在其他 Web 技术中也是可用的，例如 Java Servlets（称为 Servlet 过滤器），同样在 IIS 中也作为 ISAPI 过滤器可用。

# .NET RCW 中的桥接模式

将作为库打包的 **组件对象模型**（**COM**）技术解决方案可以通过 .NET 平台中的 **运行时调用包装器**（**RCW**）来使用。通过允许托管类和 COM 组件交互，尽管它们的接口不同，RCWs 是桥接模式的一个例子（实现为一个适配器！）。请查阅有关 **Com Callable Wrapper**（**CCW**）和 RCW 的文档，以了解桥接模式是如何实现与其他语言（主要是 C++/ATL）编写的组件进行互操作的。从技术上讲，ADO.NET API 也利用了桥接模式，以与数据库供应商实现的 ODBC 和其他本地驱动程序进行交互。

# .NET BCL 中的工厂模式

工厂设计模式已被 `System.Data.Common` 用于创建提供程序、连接、命令或适配器对象实例，以便从关系型数据库中获取数据。以下代码片段展示了这一概念：

```cs
    DbProviderFactory factory = 
    DbProviderFactories.GetFactory(providerName); 
    DbConnection connection = factory.CreateConnection(); 
    connection.ConnectionString = connectionString; 
    string queryString = "SELECT CategoryName FROM Categories"; 
    DbCommand command = factory.CreateCommand(); 
    command.CommandText = queryString; 
    command.Connection = connection; 
    DbDataAdapter adapter = factory.CreateDataAdapter(); 
    adapter.SelectCommand = command; 
    DataTable table = new DataTable(); 
    adapter.Fill(table); 

```

# WPF 中的观察者模式

`ObservableCollection` 可以被视为一种数据结构，它利用观察者模式在项目被添加或删除，或者整个列表被刷新时提供通知：

```cs
    class ObservableDataSource  
    { 
      public ObservableCollection<string> data  
      { get; set; } 

      public ObservableDataSource() 
      { 
        data = new ObservableCollection<string>(); 
        data.CollectionChanged += OnCollectionChanged; 
      } 
      void OnCollectionChanged(object sender,  
      NotifyCollectionChangedEventArgs args)  
      { 
        Console.WriteLine("The Data got changed ->" +  
        args.NewItems[0]); 
      } 
      public void TearDown() 
      { 
        data.CollectionChanged -= OnCollectionChanged; 
      } 
    } 

```

以下代码片段创建了一个基于`Observableconnection`的`ObservableDataSource`类的实例。当我们向该类添加项目时，我们会在`ObservableDataSource`类的`OnCollectionDataChanged`方法中获得通知：

```cs
    ObservableDataSource obs = new ObservableDataSource(); 
    obs.data.Add("Hello"); 
    obs.data.Add("World"); 
    obs.data.Add("Save"); 

```

# .NET Framework 中的组合模式

为了创建复杂的 UI 屏幕，.NET Framework 广泛地利用组合模式。WPF、ASP.NET Web Forms 和 Winforms 是这方面的关键示例。在 UI 场景中，可以有一个框架类，它充当所有子控件的容器。通常，开发者将面板放置以将物理屏幕划分为某种逻辑分组，并将子控件放置在这些面板中。列表、网格等控件可以嵌入其他控件。因此，这些都是组合模式的绝佳示例。

# .NET BCL 中的门面模式

GoF 门面模式在后台发生大量工作且这些类的接口通过简单的 API 公开的场景中使用。.NET BCL 中的`XMLSeralizer`类在其幕后做了大量工作，并通过一个非常简单的接口提供对这些例程的访问。以下代码片段创建了一个`DataSet`来存储数字`42`（记住道格拉斯·亚当斯！）的乘法表，并且`XMLSeralizer`类将表持久化到文本文件中：

```cs
    class Program 
    { 
      private static DataSet CreateMultTable() 
      { 
        DataSet ds = new DataSet("CustomDataSet"); 
        DataTable tbl = new DataTable("Multiplicationtable"); 
        DataColumn column_1 = new DataColumn("Multiplicand"); 
        DataColumn column_2 = new DataColumn("Multiplier"); 
        DataColumn column_3 = new DataColumn("REsult"); 
        tbl.Columns.Add(column_1); 
        tbl.Columns.Add(column_2); 
        tbl.Columns.Add(column_3); 

        ds.Tables.Add(tbl); 
        int Multiplicand = 42; 
        DataRow r; 
        for (int i = 0; i < 10; i++) 
        { 
          r = tbl.NewRow(); 
          r[0] = Multiplicand; 
          r[1] = i; 
          r[2] = Multiplicand * i; 
          tbl.Rows.Add(r); 
        } 
        return ds; 
      } 
      // The Entrypoint for execution 
      // using Serialize method, we can traverse the tree  
      // and persist the content to a XML file. Behind the scenes 
      // the code to traverse the DataSet structure is getting  
      // executed. Using FAÇADE pattern, the .NET framekwork 
      // designers have managed to hide the complexity  

      static void Main(string[] args) 
      { 
        XmlSerializer ser = new XmlSerializer(typeof(DataSet)); 
        DataSet ds = CreateMultTable(); 
        TextWriter writer = new StreamWriter("mult.xml"); 
        ser.Serialize(writer, ds); 
        writer.Close(); 
        Console.ReadKey(); 
      } 
    } 

```

# .NET BCL 中的迭代器模式

迭代器模式如此常见，以至于大多数平台和框架都提供了一种机制来支持它。.NET BCL 有`IEnumerable`及其泛型变体，即`IEnumerable<T>`来实现自定义迭代器。为了迭代，我们有 C#中的`foreach`循环结构。Java 中也有类似的构造。以下程序通过利用.NET 固定长度数组功能创建了一个自定义列表：

```cs
    public class CustomList<T> : IEnumerable<T> 
    { 
      //------ A Fixed Length array to  
      //------ Example very simple 
      T[] _Items = new T[100]; 
      int next_Index = 0; 

      public CustomList(){} 

      public void Add(T val) 
      { 
        // We are adding value without robust 
        // error checking 
        _Items[next_Index++] = val; 
      } 

      public IEnumerator<T> GetEnumerator() 
      { 
        foreach (T t in _Items) 
        { 
          //---only reference objects can be populated  
          //-- and checked for null 
          if (t == null) { break; } 
          yield return t; 
        } 
      } 

      System.Collections.IEnumerator  
      System.Collections.IEnumerable.GetEnumerator() 
      { 
        return this.GetEnumerator(); 
      }   
    } 
    // The Entry point function  
    // creates a Custom List instance and adds entry to the list 
    // and uses foreach loop to access the iterator implemented 
    class Program 
    { 
      static void Main(string[] args) 
      { 
        CustomList<string> lst = new CustomList<string>(); 
        lst.Add("first"); 
        lst.Add("second"); 
        lst.Add("third"); 
        lst.Add("fourth"); 

        foreach (string s in lst) 
        { 
          Console.WriteLine(s); 
        } 
        Console.ReadKey(); 
      } 
    } 

```

# 摘要

在本章中，我们学习了.NET BCL 的设计者如何利用设计模式来公开一个定义良好的编程模型和灵活的 API。你学习了.NET Framework 的设计者如何使用一些重要的模式。在下一章中，你将学习.NET 平台中的并发和并行编程。
