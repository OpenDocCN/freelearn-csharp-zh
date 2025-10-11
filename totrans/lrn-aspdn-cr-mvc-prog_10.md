# 第十章。使用 ASP.NET Web API 构建基于 HTTP 的 Web 服务

到目前为止，我们已经学习了如何使用 ASP.NET Core 创建 Web 应用程序。但有时仅仅创建一个 Web 应用程序是不够的。让我们假设你正在使用 ASP.NET Core 创建一个提供全球所有城市天气信息的 Web 应用程序。人们访问你的 Web 应用程序以获取天气信息，并且对服务感到满意。但这个天气信息可能被许多其他网站或 Web 应用程序需要，例如旅游网站、新闻网站以及许多其他移动应用程序。

而不是为他们的网站重新编写代码，你可以创建和发布 Web 服务，网站可以在需要时消费所需的 Web 服务。

在本章中，你将学习以下主题：

+   基于 HTTP 的服务是什么以及它如何有用

+   Fiddler 是什么

+   如何使用 Fiddler 组合 HTTP 请求并触发以获取 HTTP 响应

+   如何使用 Web API 设计和实现 HTTP 服务

微软为程序员提供了 ASP.NET Web API 来构建基于 HTTP 的服务。但 HTTP 不仅仅用于服务网页。你可以将 HTTP 作为一个平台。这带来了几个优势：

+   由于使用 ASP.NET Web API 构建的 Web 服务使用 HTTP 进行通信，因此这些 Web 服务可以从各种应用程序中消费，从控制台应用程序到 Web 应用程序，以及从 WCF 服务到移动应用程序

+   无论何时，只要 Web 服务的逻辑/代码有任何变化，客户端（使用这些服务的网站）都不需要做任何改变。他们可以像之前一样消费 Web 服务

# HTTP 基础知识

HTTP 是构建服务的强大平台。你可以使用现有的 HTTP 动词来构建服务。例如，你可以使用现有的 HTTP 动词 GET 来获取产品列表或 POST 来更新产品信息。让我们快速看一下 HTTP 在构建服务方面的作用。

在 ASP.NET MVC 中服务 HTML 页面和在 HTTP 服务上下文中服务数据的基本机制之间没有区别。两者都遵循请求-响应模式以及相同的路由机制。

一个 HTTP 请求可以从任何客户端（桌面、笔记本电脑、平板电脑、手机等等）发送到服务器，服务器将回送一个 HTTP 响应。HTTP 响应可以以任何格式发送到客户端，例如 JSON 或 XML。这将在以下图中展示：

![HTTP 基础知识](img/Image00194.jpg)

在前面的图中，请求是从桌面计算机发送的（它同样可以从手机或平板电脑发送；没有区别）并且服务器为请求发送 HTTP 响应。由于 HTTP 在大多数设备上得到支持，因此它是无处不在的。

## HTTP 动词

HTTP 动词描述了请求必须如何发送。这些是在 HTTP 中定义的方法，规定了 HTTP 请求从客户端发送到服务器的方式

## GET 方法

当我们使用 HTTP GET 请求时，信息通过 URL 本身传递：

```cs
GET api/employees/{id} 

```

这个 `GET` 请求根据传递的 ID 获取员工信息。使用 `GET` 请求的优势在于它轻量级，所有必要的信息都会通过 URL 或头信息本身传递，如下面的图所示：

![GET 方法](img/Image00195.jpg)

## PUT 方法

`PUT` 方法用于创建资源或更新它。`PUT` 是一个幂等操作，这意味着即使多次执行，预期的行为也不会改变：

![PUT 方法](img/Image00196.jpg)

## POST 方法

您可以使用 POST 来创建或更新资源。通常，POST 用于创建资源而不是更新它。根据 HTTP 标准，每次创建新资源时，都应该返回一个 **201 HTTP** 状态码：

![POST 方法](img/Image00197.jpg)

## DELETE 方法

DELETE 方法用于删除资源。通常，当你删除资源时，你会传递一个 ID 作为参数，并且你不会在请求体中传递任何内容：

![DELETE 方法](img/Image00198.jpg)

通常，HTTP 服务会被其他应用程序和服务消费。消费服务的应用程序被称为客户端。测试 HTTP 服务的一个选项是构建客户端。但这会耗费时间，一旦测试了 HTTP 服务，我们可能会丢弃客户端代码。

另一个广泛使用的选择是使用能够让我们发送 HTTP 请求并监控响应的应用程序。有许多应用程序可供选择，Fiddler 就是其中广泛使用的一个。

## Fiddler 工具

Fiddler 是一个代理服务器应用程序，用于监控 HTTP 和 HTTPS 流量。您可以监控从客户端发送到服务器的请求，发送到客户端的响应，以及从服务器接收到的响应。这就像看到服务器和客户端之间管道中的流量。您甚至可以编写请求，发送它，并分析收到的响应，而无需为服务编写客户端。

您可以在 [`www.telerik.com/fiddler`](http://www.telerik.com/fiddler) 下载 Fiddler。您将看到以下窗口：

![Fiddler 工具](img/Image00199.jpg)

理论已经足够。让我们使用 ASP.NET Web API 创建一个简单的 Web 服务。

启动 Visual Studio 2015：

![Fiddler 工具](img/Image00200.jpg)

当您点击 **确定** 时，将创建一个 Web API 解决方案。就像 ASP.NET Core 应用程序控制器从 Controller 类继承一样。

![Fiddler 工具](img/Image00201.jpg)

Web API 类也将继承相同的 Controller 类。这是 ASP.NET Core 与早期版本的 ASP.NET MVC 之间的区别。在早期版本中，所有 Web API 控制器类都继承自`ApiController`类。在 ASP.NET 5 中，它已经统一，相同的基类 Controller 被用于构建 Web 应用程序和服务。

以下是在创建项目时选择**Web API**模板选项时默认创建的`ValuesController`类：

![Fiddler 工具](img/Image00202.jpg)

在我们创建自己的自定义 Controller 之前，让我们分析默认的 API Controller。在`ValuesController`文件中，已经定义了几个 API 方法。

有两个重载的`GET`方法——一个带有参数，另一个不带参数。不带参数的`GET`方法返回该类型的所有资源。在这种情况下，我们只返回几个字符串。在现实世界中，我们会返回资源的元数据。例如，如果我们对电影 API 控制器发起`GET`请求，它会返回关于所有电影的信息。带有`id`参数的`GET`方法返回与传递的 ID 匹配的资源。例如，如果你传递一个电影 ID，它会返回关于该电影的信息。其他方法（如`PUT`、`POST`和`DELETE`）的正文在这个控制器中为空，我们将在稍后讨论这些方法。

当你运行应用程序时，你会得到以下输出：

![Fiddler 工具](img/Image00203.jpg)

默认情况下，它会向`api/values`发起请求，并在浏览器中显示这些值。

让我们学习如何在 Fiddler 应用程序中发起 HTTP 请求。打开 Fiddler 应用程序。在左下角，选择红色方框中的**Web 浏览器**选项。选择此选项将使我们能够查看来自**Web 浏览器**的流量：

![Fiddler 工具](img/Image00204.jpg)

选择**作曲家**选项卡，输入 URL `http://localhost:49933/api/values`，如以下截图所示，然后在右上角点击**执行**按钮：

![Fiddler 工具](img/Image00205.jpg)

一旦你点击**执行**按钮，就会创建一个 HTTP 会话，在左侧面板中可见（如蓝色方框所示）。点击会话，然后在右上角面板中选择**检查器**选项卡。在右下角面板中选择 JSON 选项卡（如下面的截图所示，由紫色边框突出显示）。

你可以看到 HTTP 请求返回的 JSON 数据——以下截图中的**value1**和**value2**：

![Fiddler 工具](img/Image00206.jpg)

现在轮到我们编写自定义 API 了。

在这个自定义 API 中，我们将提供创建员工对象、列出所有员工对象以及删除员工对象的 API 方法。

首先，让我们为员工创建一个模型。我们需要创建一个文件夹来存放这些模型。右键单击项目，选择 **添加** | **新建文件夹**，并将文件夹命名为 `Models`：

![Fiddler 工具](img/Image00207.jpg)

右键单击 `Models` 文件夹，选择 **添加** | **新建项…** 来创建一个员工模型类。这个员工模型类只是一个 POCO 类。请看以下代码：

```cs
public class Employee
{
  public int Id {get; set;}
  public string FirstName {get; set;}
  public string LastName {get; set;}
  public string Department {get; set;}
}
```

然后，我们定义存储库接口来处理模型：

```cs
public interface IEmployeeRepository
{
  void AddEmployee(Employee e);
  IEnumerable<Employee> GetAllEmployees();
  Employee GetEmployee(int id);
  Employee RemoveEmployee(int id);
  void UpdateEmployee(Employee employee);
}
```

然后，我们为这个模型实现接口：

```cs
public class EmployeeRepository : IEmployeeRepository
{
  private static List<Employee> employees = new List<Employee>();

  public EmployeeRepository()
  {

    Employee employee1 = new Employee
    {
      FirstName = "Mugil",
      LastName = "Ragu",
      Department = "Finance",
      Id = 1
    };

   Employee employee2 = new Employee
   {
      FirstName = "John",
      LastName = "Skeet",
      Department = "IT",
      Id = 2
   };

  employees.Add(employee1);
  employees.Add(employee2);
  }

  public IEnumerable<Employee> GetAllEmployees()
  {
    return employees;
  }

  public void AddEmployee(Employee e)
  {
    e.Id = GetNextRandomId();
    employees.Add(e);
  }

  public Employee GetEmployee(int id)
  {
    return employees.Where(emp => emp.Id == id).FirstOrDefault();
  }

  public Employee RemoveEmployee(int id)
  {
    Employee employee = employees.Where(emp => emp.Id ==   id).FirstOrDefault();
    if (employee !=null )
    {
      employees.Remove(employee);
    }
    return employee;
  }

  public void UpdateEmployee(Employee emp)
  {
    Employee employee = employees.Where(e => e.Id ==  emp.Id).FirstOrDefault();
    if(employee != null)
    {
    employee.Department = emp.Department;
    employee.FirstName = emp.FirstName;
    employee.LastName = emp.LastName;
    }
  }

  private int GetNextRandomId()
  {
    int id = -1;
    bool isIdExists;
    Random random = new Random();
    do
    {
      id = random.Next();
      isIdExists = employees.Any(emp => emp.Id == id);
    } while (isIdExists);
    return id;
  }
}
```

在实现类中需要注意以下几点：

+   我们决定不使用数据库，因为我们的目标是使用 Web API 创建一个 HTTP 服务，而不是编写数据访问代码。

+   我们正在使用内存中的列表来存储数据。所有操作都将在这个列表上执行。实际上，数据可以是任何形式，从关系型数据库到简单的内存列表。

+   在构造函数方法中，我们正在将一个对象添加到列表中。这个列表将作为我们 HTTP 服务的数据库。

+   `GetAllEmployees` API 方法将返回所有员工作为 `IEnumerable` 接口。

+   `AddEmployee` 方法会将传入的员工（作为参数）添加到列表中。

+   `GetEmployee` 方法将返回与参数 ID 匹配的员工。

+   `RemoveEmployee` 方法将从列表中删除员工。

+   `UpdateEmployee` 方法将更新员工信息。

+   `GetNextRandomId` 方法将返回下一个可用的随机整数。这个整数值被用来生成员工 ID。

# 依赖注入

在大多数实际项目中，我们不会在任何一个控制器中使用 `new` 实例化任何对象，原因是我们不希望依赖组件（控制器和存储库之间）之间有紧密耦合。相反，我们向控制器传递一个接口，当需要为控制器创建对象时，依赖注入容器（如 **Unity**）会为我们创建对象。这种设计模式通常被称为 **控制反转**。

假设有一个名为 *ClassA* 的类使用了另一个名为 *ClassB* 的类。在这种情况下，*ClassA* 只需要知道 *ClassB* 的行为、方法和属性，而不需要知道 *ClassB* 的内部实现细节。因此，我们可以抽象 *ClassB* 并将其转换为接口，然后使用这个接口作为参数，而不是具体的类。这种方法的优点是，只要它实现了一个共同约定的合同（接口），我们就可以在运行时传递任何类。

在 ASP.NET 5（包括 ASP.NET Core 和 Web API）中，我们内置了对依赖注入的支持。在`ConfigureServices`方法中，我们添加了（以粗体突出显示）执行依赖注入的行。我们指示内置的依赖注入容器在引用`IEmployeeRepository`接口的地方创建`EmployeeRepository`类，并且我们还指示它为单例；这意味着相同的对象（由依赖注入容器创建）将在整个应用程序的生命周期中共享：

```cs
public void ConfigureServices(IServiceCollection services)
{
  // Add framework services.
  services.AddApplicationInsightsTelemetry(Configuration);
  services.AddMvc();

 services.AddSingleton<IEmployeeRepository, EmployeeRepository>(); 

}

```

在前面的代码中，我们使用了单例模式进行依赖注入，它只在第一次请求时创建服务。还有其他类型的生命周期服务，如**Transient**和**Scoped**。瞬态生命周期服务每次请求时都会创建，而作用域生命周期服务每次请求都会创建一次。以下是在使用此类生命周期时创建的代码片段：

```cs

services.AddTransient

 <IEmployeeRepository, EmployeeRepository>(); 

services.AddScoped  <

IEmployeeRepository, EmployeeRepository>();

```

现在是时候进入创建 API 控制器的核心操作了。在**Controllers**文件夹上右键单击，然后选择**Add | New Item**。然后从列表中选择**Web API Controller Class**，如下所示截图。命名你的控制器，然后点击**Add**按钮：

![依赖注入](img/Image00208.jpg)

从控制器中删除生成的代码，并添加以下构造函数：

```cs
public EmployeeController(IEmployeeRepository employeesRepo)
{
  employeeRepository = employeesRepo;
}
private IEmployeeRepository employeeRepository {get; set;}
```

在前面的构造函数中，我们正在注入依赖。在调用此构造函数时，将创建`EmployeeRepository`对象。

让我们实现几个`GET`方法——第一个将返回所有员工的详细信息，第二个`GET`方法将根据传递的员工 ID 返回员工：

```cs
public IEnumerable<Employee> GetAll()
{
  return employeeRepository.GetAllEmployees();
}

[HttpGet("{id}",Name ="GetEmployee")]
public IActionResult GetById(int id)
{
  var employee = employeeRepository.GetEmployee(id);
  if(employee == null)
  {
  return NotFound();
  }
  return new ObjectResult(employee);
}
```

让我们从 Fiddler 中调用这些 HTTP 方法。

运行解决方案，打开 Fiddler 应用程序，并点击**Composer**标签。

选择 HTTP 方法（我们选择了`GET`方法，因为我们有一个 GET API 方法）并输入 URL `http://localhost:49933/api/employee`。

请注意，当我的应用程序运行时，它运行在端口`49933`上；你的端口号码可能会有所不同，因此请相应地构造你的 URL。

一旦你输入了 URL 并选择了方法，点击以下截图所示的**Execute**按钮：

![依赖注入](img/Image00209.jpg)

一旦你点击**Execute**按钮，将创建一个 HTTP 会话，并发送请求。

点击左侧面板上的会话（如下所示截图），然后在右侧面板中选择**Inspectors**标签。你可以在底部右侧面板的**JSON**标签中查看结果：

![依赖注入](img/Image00210.jpg)

让我们再发送一个 HTTP 请求来获取特定员工的信息，比如说 ID 为 2 的员工。我们将通过附加 ID 来构造 URL，如下所示：`http://localhost:49933/api/employee/2`。

![依赖注入](img/Image00211.jpg)

选择最近创建的 HTTP 会话并点击它：

你可以在右侧面板中看到以 JSON 格式的结果。

现在，我们将向我们的服务添加 `Create`、`Update` 和 `Delete` 操作。首先，我们将提供创建功能以将员工添加到我们的服务中：

```cs
[HttpPost]
public IActionResult Add([FromBody] Employee emp)
{
  if (emp == null)
  {
    return BadRequest();
  }
  employeeRepository.AddEmployee(emp);
  return CreatedAtRoute("GetEmployee", new { id = emp.Id }, emp);
}
```

在遵循前面的 `Add` 方法时，应考虑以下要点：

1.  我们将 `Employee` 对象作为参数传递。我们指示 `Add` 方法通过指定 `[FromBody]` 属性从请求体中获取该对象：

    +   如果没有传递员工对象，我们将向调用客户端返回错误请求。

    +   如果它不为空，我们将调用存储库方法将员工添加到我们的列表中（在现实世界中，我们会将其添加到数据库中）。

1.  一旦我们添加了员工，当创建新资源时，我们将返回 *201 状态码*（根据 HTTP 标准）。

打开 Fiddler 应用程序并按照以下步骤添加员工：

1.  选择 HTTP 方法为 `POST` 并输入 URL `http://localhost:54504/api/employee/`。

1.  你需要在请求头中指定内容类型为 `application/json`。请参阅以下截图，其中我们在请求头中添加了 `Content-Type: application/json`。

1.  如代码所述，我们必须以 JSON 格式在请求体中传递员工对象。在以下请求中，我们创建了一个包含 `Employee` 对象属性的 JSON，括号内为值 { "FirstName" : "James", "LastName" : "Anderson","Department" : "IT"}：![依赖注入](img/Image00212.jpg)

一旦你已编写完请求，你可以点击 **执行** 按钮来发送请求。这将返回 *201 HTTP 状态码*，这是创建新资源的标准 HTTP 响应：

![依赖注入](img/Image00213.jpg)

一旦我们在服务器上创建了资源，我们将重定向响应以获取新创建的资源。这发生在我们调用 `CreatedAtRoute` 方法并将新创建的员工 ID 作为参数传递时。

点击左侧的会话并选择右侧面板中的 **检查器** 选项卡。现在你可以看到请求的响应。响应包含在服务器上新创建的 `Employee` 对象。我们必须注意，`Employee` 对象的 ID 是在服务器上生成的，并在以下响应中可用。在这种情况下，为员工生成的 ID 是 `1771082655`：

![依赖注入](img/Image00214.jpg)

在上一个 Fiddler 窗口的右下角面板中，我们可以看到新创建资源的完整 JSON 响应。

现在我们将添加一个 Web API 方法来更新资源。更新资源的方法与创建资源的方法非常相似，只有一些细微差别。当我们创建资源时，我们使用了 `HTTP POST` 方法，而当我们更新资源时，我们使用了 `HTTP PUT` 方法。

如果传递的员工 ID 在存储库中找不到，我们将返回 *404 错误* 响应，这是未找到资源的 HTTP 标准错误响应。

以下是为更新资源而编写的 Web API 控制器方法代码：

```cs
[HttpPut]
public IActionResult Update([FromBody] Employee emp)
{
  if( emp == null)
  {
    return BadRequest();
  }
  Employee employee = employeeRepository.GetEmployee(emp.Id);
  if(employee == null)
  {
    return NotFound();
  }
  employeeRepository.UpdateEmployee(emp);
  return new NoContentResult();
}
```

以下是为更新员工信息的存储库层代码：

```cs
public void UpdateEmployee(Employee emp)
{
  Employee employee = employees.Where(e => e.Id == emp.Id).FirstOrDefault();
  if (employee != null)
  {
    employee.Department = emp.Department;
    employee.FirstName = emp.FirstName;
    employee.LastName = emp.LastName;
  }
}
```

打开 Fiddler 应用程序，并编写一个 `HTTP PUT` 请求。由于我们将在请求体中传递 `Employee` 对象，因此需要指定内容类型为 `application/json`。在请求体中，我们需要以 JSON 格式提供 `Employee` 对象，如下截图所示：

![依赖注入](img/Image00215.jpg)

当你点击 **执行** 按钮，将触发 `HTTP PUT` 请求，并且我们的 Web API 方法将被调用。一旦成功，将返回 *HTTP 204* 响应：

![依赖注入](img/Image00216.jpg)

## 删除方法

当删除资源时，应使用 `HTTP DELETE` 方法。请求体中无需传递任何内容。

### 删除资源的 Web API 方法

`Delete` Web API 方法有一个 `void` 返回类型，它将返回 *HTTP 200* 响应：

```cs
[HttpDelete("{id}")]
public void Delete(int id)
{
  employeeRepository.RemoveEmployee(id);
}
```

### 删除员工数据的 Web 存储库层代码

在以下存储库层方法中，我们正在从员工的内部列表中删除（其 ID 与传递的参数匹配）员工。但在现实生活中，我们会与数据库交互以删除该特定员工。考虑以下代码：

```cs
public Employee RemoveEmployee(int id)
{
  Employee employee = employees.Where(emp => emp.Id ==   id).FirstOrDefault();
  if(employee != null)
  {
    employees.Remove(employee);
  }
  return employee;
}
```

打开 Fiddler 应用程序，选择 `DELETE` HTTP 方法，传递带有参数的 URL，然后点击 **执行** 按钮。请注意，我们不在请求头中传递内容类型，因为我们没有在请求体中传递任何员工对象：

![删除员工数据的 Web 存储库层代码](img/Image00217.jpg)

由于我们返回的是空值，Web API 的 `DELETE` 方法返回 *HTTP 200* 状态，如 Fiddler 应用程序的左侧窗格所示：

![删除员工数据的 Web 存储库层代码](img/Image00218.jpg)

# 摘要

在本章中，你学习了 HTTP 服务及其用途。我们讨论了如何使用 Web API 设计和实现 HTTP 服务。我们使用 Fiddler 工具构建 HTTP 请求并获取响应。我们还学习了如何编写 Web API 方法以执行 CRUD 操作，从编写 Web API 方法到发送请求并获取响应。

# 读累了记得休息一会哦~

**公众号：古德猫宁李**

+   电子书搜索下载

+   书单分享

+   书友学习交流

**网站：**[沉金书屋 https://www.chenjin5.com](https://www.chenjin5.com)

+   电子书搜索下载

+   电子书打包资源分享

+   学习资源分享
