# 2

# 创建你的第一个最小化 API

在上一章中，你采取的步骤充分体现了.NET 生态系统的便利性。.NET 中的最小 API 不仅具有最小逻辑和依赖，而且具有最小设置要求，真正做到了名副其实。

只需创建你的第一个项目，你实际上已经创建了你的第一个最小化 API。它开箱即用，包含一个**GET**端点，返回响应。

显然，最小 API 的内容远不止我们在**Hello World**示例中看到的那样。我们需要考虑不同的 HTTP 请求方法、不同的端点以及更高级的响应生成，这些都是简单 API 的一部分。

在本章中，我们将涵盖以下主要内容：

+   项目结构和组织

+   定义端点和路由

+   构建员工管理 API

+   处理 HTTP 请求

到本章结束时，你将获得定义端点以处理 HTTP 请求的经验。你还将能够实现基本的 HTTP 请求处理和响应生成。

# 技术要求

要遵循本章的指示，你需要在你的 Windows、macOS 或 Linux 机器上安装以下内容：

+   .NET 9.0 **软件开发** **工具包** ( **SDK** )

+   Visual Studio 或 Visual Studio Code

+   Visual Studio Code 的 C#扩展（如果你使用 Visual Studio Code）

如果你遵循了*第一章*中的设置指南，那么你就可以按照本章的指示进行操作。然而，如果你仍然需要配置前面的工具，那么请在*安装所需工具和依赖项*部分下遵循*第一章*的设置说明。本章的代码可在 GitHub 仓库中找到：[`github.com/PacktPublishing/Minimal-APIs-in-ASP.NET -9`](https://github.com/PacktPublishing/Minimal-APIs-in-ASP.NET-9) 。

# 项目结构和组织

关于项目结构和组织的规则并不完全严格，但确保你以一致和可访问的方式组织项目是很重要的。在一般情况下，我们追求的是简单性。在最小 API 中，项目结构和组织也是如此。因此，以下示例中的最小 API 项目结构可能不会让你感到惊讶，它将非常基础。

在以下三个小节中，我们将探讨简化项目结构所需元素。

## 端点

**端点**是 API 的**开放门户**。每个端点位于你的域（例如，员工或库存）的一个区域，并负责该域内的特定操作，如添加、更新或删除库存项。

在基于 ASP.NET **Controller** 的项目中，您通常会根据域的每个区域将端点分配到逻辑组中，称为 **Controllers**。每个 **Controller** 是一个包含与该 **Controller** 的域区域相关的端点的类。

然而，在最小化 API 中，端点通常简单地定义在应用程序的入口点，在 `Program.cs` 中。

## 模型

就像在 **Model-View-Controller**（**MVC**）项目中一样，在最小 API 中，模型可以用来封装域对象。模型通常创建为其自己的类。在即将到来的示例中，我将创建用于管理员工的 API 的逻辑。因此，我将使用 **Employee** 类作为模型。

使用一个类来表示域对象会带来许多好处，包括以下内容：

+   **关注点分离**：模型封装了应用程序的数据，将其与业务逻辑分离，在我们的情况下，业务逻辑将在 API 层找到。

+   **可重用性**：模型对象可以在应用程序的业务逻辑中被重用。

+   **松散耦合**：由于数据和业务逻辑之间的关注点分离，API 的更改不需要无意中影响数据的结构。这在常见情况下尤为重要，其中模型通过 ORM（如 Entity Framework 或 Dapper）镜像数据库表的结构（见 *第八章*）。

保持组织

虽然最小 API 项目结构可以保持非常简单，但仍然重要的是要保持它们有组织。良好的做法是将所有模型放在专门的 **Models** 文件夹中，所有端点放在它们自己的 **Endpoints** 文件夹中。

对于项目结构拼图中的最后一部分，我们有路由。

## 路由

如果端点是进入 API 的敞开大门，那么 **路由** 就是每扇门的位置。通过创建路由，您正在定义您的 API 将响应的 URL，以及将执行哪个逻辑片段。

路由可以是独立的或包含将被传递到结果 API 逻辑中的参数。

在以下页面上的端点中，您将看到以字符串形式定义的路由，指示应附加到应用程序的基本 URL 上的文本，以访问特定端点。

例如，如果我们的应用程序托管在 `getInventoryItem` 上，该端点的完整 URL 将成为 [`adventureworks.com/getInventoryItem`](https://adventureworks.com/getInventoryItem)。

我们可以，然而，使我们的路由更加通用。`getInventoryItem` 确实很清晰，但更好的做法是，在可能的情况下，向通用路由发送不同类型的请求，根据使用的 HTTP 方法触发不同的逻辑。

例如，我们不必将路由命名为**getinventoryitem**，而可以将通用名称**inventoryitems**应用于与 HTTP 方法相关的任何路由。这意味着对这个路由的**GET**请求（我们将在下一节中了解这些方法）将获取一个库存项目，但对该路由的**POST**请求将创建一个。

这被认为是一种命名路由的最佳实践，原因有几个：

+   **一致性**：由于具有通用端点已成为 API 约定，它允许您的 API 符合一个已同意的标准。

+   **直观性**：大多数消费 API 的开发人员都会遵循相同的标准。这意味着他们将能够更快地了解您的 API。

+   **RESTful 原则**：允许 HTTP 方法识别端点的功能而不是路由，使我们能够符合 RESTful 原则，这些原则鼓励良好的**CREATE、READ、UPDATE、DELETE**（**CRUD**）操作、幂等性（在多个相同的请求中，服务器状态不会改变，并且每次对相同请求返回相同的结果），以及需要全面覆盖标准 HTTP 方法（**GET**、**POST**、**PUT**、**PATCH**、**DELETE**）。

与列表相反，如果您有支持 HTTP 方法的 API 端点，这些方法在非常特定的用例中，您仍然可以创建更具体的路由。然而，常规做法是使用我们概述的通用路由命名约定。

理解路由对于最小 API 开发至关重要，除了大多数 API 框架之外。让我们转向定义路由及其端点的方式。

# 定义端点和路由

与任何其他 RESTful API 框架一样，使用不同的 HTTP 方法访问最少的 API 端点。根据用于联系 API 端点的 HTTP 方法，会产生不同的结果或执行不同的操作。

在接下来的几节中，我们将查看一些端点与不同 HTTP 方法映射的示例。

## GET 方法

HTTP **GET** 方法是一个请求信息。在成功检索后，API 端点返回一个 **200 OK** 状态码，以及请求的数据。

示例显示一个**GET**端点映射到**"/employees"**路由。它获取 URL 中包含的员工 ID，并使用这个 ID 来查找相关的员工数据，然后再返回。

例如，如果这个 API 托管在 constoso.com，对[contoso.com/employees/24](https://contoso.com/employees/24)的**GET**调用将检索 ID 为**24**的员工：

```cs
app.MapGet("/employees/{id}", (int id) =>
{
    var employee = EmployeeManager.Get(id);
    return Results.Ok(employee);
});
```

让我们来看看**POST**方法。

## POST 方法

HTTP **POST** 方法是创建某物的请求。在成功执行后，API 端点通常返回一个**201 Created**状态码（这是最佳实践，尽管一些 API 返回标准的**200 OK**代码），以及任何相关的数据。

以下示例展示了将 **POST** 方法映射到 **"/employees"** 端点。它期望接收一个 JSON 格式的员工负载。端点然后将负载转换为 **Employee** 类型的对象，在调用一些其他后端代码以使用此对象作为参数创建 **Employee** 之前。

如果一切按预期工作，端点返回 **201 Created** 状态码，以及 **Employee** **Created** 消息。

例如，如果这个 API 在 **constoso.com** 上托管，一个 **POST** 调用将执行以下代码：

```cs
app.MapPost("/employees", (Employee employee) =>
{
    EmployeeManager.Create(employee);
    return Results.Created();
});
```

接下来，我们将查看 **PUT** 方法。

## **PUT** 方法

HTTP **PUT** 方法是一个用于更新内容的请求。重要的是要记住，与具有自己更新方式的 **PATCH** 方法相比，**PUT** 方法以特定的方式更新资源。

**PUT** 方法需要一个表示正在更新的整个资源的负载。因此，在我们的 **Employee** API 的例子中，如果您要使用 **PUT** 端点更新现有员工，API 端点将期望在请求中发送一个完整的 **Employee** 对象。然后它会找到现有的员工，并用请求中发送的一个替换它。

在成功执行后，API 端点返回标准的 **200 OK** 状态码。

示例展示了将 **PUT** 方法映射到 **"/employees"** 路由。它期望接收一个 JSON 格式的员工负载。与前面的 **POST** 示例类似，端点将此 JSON 负载转换为 **Employee** 类型的对象，然后在找到原始员工并调用一个方法用更新后的对象替换它之前。

例如，如果这个 API 在 constoso.com 上托管，一个 **PUT** 调用将执行以下代码：

```cs
app.MapPut("/employees", (Employee employee) =>
{
    EmployeeManager.Update(employee);
    return Results.Ok();
});
```

## **PATCH** 方法

与 **PUT** 方法类似，HTTP **PATCH** 方法也是一个用于更新内容的请求。然而，它执行更新的方式不同。**PATCH** 方法不需要包含整个对象表示的负载，只需将需要更改的个别值作为请求的一部分发送即可。然后 API 可以负责更新现有对象的相关属性。

在成功执行后，API 端点通常返回标准的 **200 OK** 状态码。

示例展示了将 **PATCH** 方法映射到 **"/updateEmployeeName"** 路由。它期望接收一个类似于 **POST** 和 **PUT** 的 **Employee** 对象。然而，它只对 **Name** 和 **Id** 属性感兴趣。这意味着只要发送一个包含这些属性的 JSON 负载，它就会工作。使用这些属性，代码根据给定的 Id 获取正确的 **Employee** 类型对象。然后只更新检索到的员工的名称属性，而不覆盖整个对象。

例如，如果这个 API 在 constoso.com 上托管，一个 **P** **ATCH** 调用将执行以下代码：

```cs
app.MapPatch("/updateEmployeeName", (Employee employee) =>
{
    EmployeeManager.ChangeName(employee.Id, employee.Name);
    return Results.Ok();
});
```

我们将要查看的最后一个方法是 **DELETE** 。

## DELETE 方法

**DELETE** 方法在描述其功能方面是自解释的。将资源的 ID 作为参数发送，就像我们在**GET**端点所做的那样，API 可以定位到该特定资源并将其删除。

在大多数情况下，成功时，**DELETE**方法通常会返回标准的**200 OK**状态码，但也可以返回**204 No Content**，这也是可以的。

以下示例显示了一个映射到**"/employees"**路由的**DELETE**方法。端点将使用**Id**参数来找到要删除的**Employee**类型的对应对象。删除**Employee**对象后，它向客户端返回**204 No Content**状态码。

如果这个 API 托管在 constoso.com 上，对[`contoso.com/employees`](https://contoso.com/employees)的**DELETE**调用将执行以下代码：

```cs
app.MapDelete("/deleteEmployee/{id}", (int id) =>
{
    EmployeeManager.Delete(id);
    return Results.Ok();
});
```

一旦你觉得你已经理解了我们 API 提供的不同 HTTP 方法，就转到下一节，开始基于一个简单、真实世界的用例构建一个基本的 API。

# 构建员工管理 API

现在我们已经概述了路由和端点，以及它们如何支持各种 HTTP 方法，让我们开始构建一个新的最小 API 项目，以员工管理为例。

本节的目标是创建一个员工库，我们的 API 可以在其上操作。然后，API 将能够获取、创建、更新和删除员工。

## 创建 API

按照以下步骤创建员工管理 API。

如果你还没有按照*第一章*中的步骤在 Visual Studio 或 Visual Studio Code 中创建你的 ASP.NET 项目，请首先遵循这些步骤，然后继续下一步：

1.  你可能已经在项目中有了这个，如果没有，确保它在**Program.cs**的顶部。这将构建托管最小 API 的**WebApplication**实例：

    ```cs
    var builder = WebApplication.CreateBuilder(args);
    var app = builder.Build();
    ```

1.  上一步中的两行足以构建**WebApplication**实例，但仍然需要启动。将此行添加到类的底部以启动实例：

    ```cs
    app.Run();
    ```

现在你有一个可运行的程序，但没有端点。在我们定义这些端点之前，我们需要一些数据来工作。我们将在稍后回到**Program.cs**来定义端点。但在那之前，让我们创建**Employee**类型的模型：

1.  在项目的文件夹结构中，创建一个名为**Models**的新文件夹。

1.  在新文件夹内，创建一个名为**Employee**的新类。

1.  在**Employee**类中创建这些属性：

    ```cs
        public class Employee
        {
            public int Id { get; set; }
            public string Name { get; set; }
            public decimal Salary { get; set; }
            public string Address { get; set; }
            public string City { get; set; }
            public string Region { get; set; }
            public string PostalCode { get; set; }
            public string Country { get; set; }
            public string Phone { get; set; }
        }
    ```

    此模型可以用来表示**Employee**资源，API 可以在其上执行各种 CRUD 操作。通常，我们会将此类数据保存在数据库中，例如 SQL，但为了简单起见，我们现在将将其保存在一个集合中。为此，该集合需要位于可以被我们即将创建的端点访问的地方。

1.  在项目顶层（与**Program.cs**相同的级别），创建一个名为**EmployeeManager**的新类。

1.  我们希望这个类在任何时候都可以使用，而不需要实例化它，所以将其设为静态类。

1.  在类顶部添加一个私有的**List**类型的**Employee**。你的类现在应该看起来像这样：

    ```cs
    public static class EmployeeManager
    {
        private static List<Employee> _employees =
            new List<Employee>();
    }
    ```

现在我们有一个易于访问的类，可以存储员工。由于集合是私有的，我们现在可以添加一组可以公开暴露给端点以执行操作的方法。

我们即将为对集合中每个**Employee**执行的 CRUD 操作创建逻辑。作为这部分，将需要查找列表中的每个员工对象。让我们添加一个私有的函数，它将为我们找到这些信息，以便在每次 CRUD 操作中重复使用。添加了私有函数后，类现在看起来像这样：

```cs
public static class EmployeeManager
{
    private static List<Employee> _employees =
        new List<Employee>();
    private static int getEmployeeIndex(int id)
    {
        var employeeIndex =
            _employees.FindIndex(x => x.Id == id);
        if (employeeIndex == -1)
        {
            throw new ArgumentException(
                $"Employee with Id {id} does not exist");
        }
        return employeeIndex;
    }
}
```

最后，更新类，使其包含此代码片段中显示的 CRUD 方法和函数：

```cs
public static void Create(Employee employee)
{
    _employees.Add(employee);
}
public static void Update(Employee employee)
{
    _employees[_getEmployeeIndex(employee.Id)] =
        employee;
}
public static void ChangeName(int id, string name)
{
    _employees[_getEmployeeIndex(id)].Name = name;
}
public static void Delete(int id)
{
    _employees.RemoveAt(_getEmployeeIndex(id));
}
public static Employee Get(int id)
{
    var employee =
        _employees.FirstOrDefault(x => x.Id == id);
    if (employee == null)
    {
        throw new ArgumentException("Employee Id invalid");
    }
    return employee;
}
```

**EmployeeManager**类将使我们的 API 端点能够对集合中的员工执行特定的 CRUD 操作。

## 创建第一个端点

要添加的第一个端点将是用于通过其 ID 检索特定**Employee**对象的**GET**端点。

在**Program.cs**文件中，在之前章节中添加的最终**app.Run()**行上方创建以下**GET**端点：

```cs
app.MapGet("/employees/{id}", (int id) =>
{
    var employee = EmployeeManager.Get(id);
    return Results.Ok(employee);
});
```

此代码首先引用了名为**app**的**WebApplication**实例，调用**MapGet**方法。在**WebApplication**中，有基于 HTTP 方法类型的等效映射方法。例如，包括**MapPut**、**MapPost**等等。

映射方法预期的第一个参数将是您希望监听的路径。在这种情况下，我们正在映射到**"/employees/{id}"**路由，它使用一个路由参数（更多内容请参阅*第四章*）来捕获员工 ID。

这随后是一个以 lambda 表达式形式提供的第二个参数，它使用传入的 ID 来执行预期的逻辑。在此阶段，代码在最终将结果返回给客户端之前，调用了**EmployeeManager**类中定义的**Create()**函数。

在我们刚刚创建的**GET**端点下方和**app.Run()**方法上方，将剩余的端点添加到**Program.cs**中。现在**Program.cs**应该看起来像这样：

```cs
using System.Text.Json;
using WebApplication2;
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();
app.MapGet("/employees/{id}", (int id) =>
{
    var employee = EmployeeManager.Get(id);
    return Results.Ok(employee);
});
app.MapPost("/employees", (Employee employee) =>
{
    EmployeeManager.Create(employee);
    return Results.Created();
});
app.MapPut("/employees", (Employee employee) =>
{
    EmployeeManager.Update(employee);
    return Results.Ok();
});
app.MapPatch("/updateEmployeeName", (Employee employee) =>
{
    EmployeeManager.ChangeName(employee.Id, employee.Name);
    return Results.Ok();
});
app.MapDelete("/employees/{id}", (int id) =>
{
    EmployeeManager.Delete(id);
    return Results.Ok();
});
app.Run();
```

你会注意到，您映射的所有后续端点都遵循与您添加的第一个**GET**端点相似的模式。每个端点都指定了正在使用的 HTTP 方法类型，然后是路由，然后是任何参数。然后是一个在返回结果之前执行相关逻辑的正文。

到目前为止，代码应该可以编译。（如果不行，请检查是否所有内容都已正确输入。）因此，运行应用程序（在 Visual Studio 中点击播放按钮或在 Visual Studio Code 中使用 **dotnet run** 终端命令）并对每个创建的端点进行一些测试请求。**POST**、**PUT** 和 **PATCH** 端点期望一个 **Employee** 对象作为参数，因此请确保您发送的 JSON 与 **Employee** 模型的结构相匹配。看看这个例子：

```cs
{
  "Id": 3,
  "Name": "Happy McHappyson",
  "Salary": 100000.00,
  "Address": "1 Sunny Lane",
  "City": "Happyville",
  "Region": "The Joyful Mountains",
  "PostalCode": "1234565",
  "Country": "Laughland",
  "Phone": "876542-2345-3242-234"
}
```

创建端点后，现在是时候测试它了。

### 使用 OpenAPI 测试您的端点

.NET 9 引入了 OpenAPI 集成，这意味着您只需安装一个包，并在 **Program.cs** 中更改 API 的配置，就可以生成 API 及其端点的 JSON 表示。这很有用，因为您可以将它导入 API 工具（如 Postman），然后您可以从那里轻松测试您的 API。

如果您想以这种方式进行测试，请按照以下步骤操作：

1.  通过 NuGet 安装 **Microsoft.AspNetCore.OpenApi** 包。

1.  更新 **Program.cs** 以使用 OpenApi：

    ```cs
    builder.Services.AddOpenApi();
    var app = builder.Build();
    app.MapOpenApi();
    ```

运行 API 并导航到 **openAPI/v1.json** 路由。这将为您提供 API 架构的表示，可以将其导入 API 客户端（如 Postman）进行测试。

到目前为止，在本章的这个阶段，您已经映射了具有不同 HTTP 方法的端点，为它们提供了路由和参数，创建了模型，并添加了要执行的逻辑。端点的目标是向客户端返回响应。我们现在需要通过返回响应来处理请求。

# 处理 HTTP 请求

ASP.NET 提供了一个方便的辅助对象，用于将响应发送回客户端，称为 **IResult**。

**IResult** 包含可以用于表示许多不同场景的标准 HTTP 响应的属性。例如，我们可以使用 IResult 返回特定的状态码，返回 JSON 数据，甚至触发 **ASP.NET Identity** 提供程序的功能，如挑战和登录/注销。

我们可以使用 ASP.NET 的 **Results** 工厂类轻松创建一个新的 IResult。在前面的示例中，您已经看到了对这个工厂类的引用，其中 API 通过调用 **Results.OK()** 和 **Results.Created()** 等方式返回状态码。

这些简单的 HTTP 状态码方法中的一些有可选参数，允许您以 JSON 格式返回强类型对象。例如，虽然您可以通过在 **Results.OK()** 中省略任何参数简单地返回一个 **200** 结果，但您也可以传递一个对象参数，并将其发送回客户端。我们在 Employee API 端点的 **GET** 端点中就是这样做的：

```cs
app.MapGet("/employees/{id}", (int id) =>
{
    var employee = EmployeeManager.Get(id);
    //RETURN 200 OK RESULT WITH THE EMPLOYEE OBJECT
    return Results.Ok(employee);
});
```

在辅助方法中将强类型 .NET 对象（如 **Employee**）传递回客户端的能力是最小 API 最强大的特性之一。

在*第四章*中将有更多详细示例来处理 HTTP 请求。现在，这里有一些示例说明如何使用**Results**来返回常见的 HTTP 响应：

```cs
    //200 OK
    return Results.Ok();
    //201 CREATED
    return Results.Created();
    //202 ACCEPTED
    return Results.Accepted();
    //204 NO CONTENT
    return Results.NoContent();
    //400 BAD REQUEST
    return Results.BadRequest();
    //401 UNAUTHORIZED
    return Results.Unauthorized();
    //403 FORBIDDEN
    return Results.Forbid();
    //404 NOT FOUND
    return Results.NotFound();
    //409 CONFLICT
    return Results.Conflict();
```

在探索了**Results**之后，让我们看看它的一个替代方案。

### **Typed Results**

使用**Results**返回特定 HTTP 状态码的替代方案是**TypedResults**，它与**Results**类似，但**Results**的示例响应每次返回一个**IResult**，而**TypedResults**返回一个表示状态码的强类型对象。

**TypedResults**实现了一个工厂，返回适当的强类型对象，该对象实现了**IResult**，用于指定的状态（例如，对于**200** **OK**结果返回**OK<string>**）。

你可以使用**TypedResults**几乎与使用**Results**相同的方式。以下是一个示例：

```cs
    //200 OK
    return TypedResults.Ok();
    //201 CREATED
    return TypedResults.Created();
    //202 ACCEPTED
    return TypedResults.Accepted();
    //204 NO CONTENT
    return TypedResults.NoContent();
```

.NET 9 还引入了对**500 内部服务器错误**响应的支持，在**TypedResults**中：

```cs
    //204 NO CONTENT
    return TypedResults.InternalServerError();
```

到目前为止，你应该已经对如何处理 HTTP 请求有了相当的了解，包括返回相关的 HTTP 状态码和强类型对象给客户端。让我们总结一下本章所涵盖的所有内容。

# **总结**

在本章中，我们涵盖了创建简单最小 API 的大部分入门内容，并且你创建了你的第一个最小 API 项目。

你学习了如何定义端点，以及它们如何作为进入 API 的门，每个端点都位于它们各自的路径上。以**Employee** API 为例，你了解了如何构建你的项目结构，将端点和数据分离以实现松耦合和可重用性的好处。

你探讨了模型作为数据结构的概念，这些数据结构描述了 API 使用的域对象，并且你构建了一个简单的 CRUD 系统来处理请求中的数据。

最后，你了解了如何在最小 API 中处理 HTTP 请求，使用 ASP.NET 辅助逻辑来组合并返回响应给客户端。

在掌握了基础知识之后，转向下一章，我们将探讨最小 API 的结构。我们将更深入、更科学地探讨构成最小 API 的关键组件，并介绍一些可以采用以充分发挥其潜力的各种架构和设计模式。
