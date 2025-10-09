

# 最小化 API 弹性最佳实践

就像任何软件系统一样，最小化 API 可以以多种方式构建。通过仔细选择和应用不同的模式和遵循一些既定实践，应用程序可以得到极大的增强。

在你的最小化 API 设计中构建模式有多个很好的理由，首先是可读性。由于我们大多数人是团队的一部分，可能需要将代码委托给其他开发者或移交，因此尽可能使其易于访问至关重要。通过确保你的端点是整洁的，代码尽可能具有自文档性，以及命名约定一致，其他开发者将能够相对容易地支持 API 项目的维护。

接下来是可扩展性。如果请求数量增加，那么优化应用程序的需求也会增加。一致性和良好的设计使得满足需求变得简单。无论是添加负载均衡器来管理流量，还是更改数据存储方法，设计 API 时必须确保对系统的修改——无论是添加还是删除组件——不会破坏应用程序的功能。

最后，安全性同样重要。通过遵循最佳安全实践，如静态和传输中的加密、密码散列和加盐，以及范围访问，可以安全地管理敏感数据，降低数据泄露风险以及随之而来的法律挑战。

最终，实现这些目标取决于应用以下实践：关注代码库的结构以增强可读性，处理意外和致命场景的错误处理方式，以及从网络安全角度考虑的考量。

通过探索一些可以提高代码质量的设计实践和编码约定，让我们在你的最小化 API 项目中实现一些这些好处。

在本章中，我们将涵盖以下主要主题：

+   代码组织和结构

+   错误处理

+   安全性考虑

让我们开始吧！

# 技术要求

推荐使用 Visual Studio 2022 或最新版本的 Visual Studio code 来运行本章的代码。本章的代码示例可在 GitHub 存储库中找到：[`github.com/PacktPublishing/Minimal-APIs-in-ASP.NET-9`](https://github.com/PacktPublishing/Minimal-APIs-in-ASP.NET-9)。

其中一个示例使用了第九章和第十二章的代码，这两章都依赖于 Entity Framework Core。建议你在阅读本章之前完成这些章节。

# 代码组织和结构

关于在任何系统中组织和结构化代码，可能最重要的事情是要理解没有一种正确的方法。虽然有一些广泛接受的结构模式，但这可能是一个非常个人化的话题，因为结构必须服务于维护者。然而，正如我们之前所确认的，在商业或开源环境中，大多数最小 API 系统都会有多个维护者，因此一个一致的结构将使开发者尽可能容易地在代码库上协作。

我们将探讨两种项目组织方式，它们都共享一个关键主题——模块化。

**模块化**是将你的代码组织成更小、自包含和可复用的单元或模块的实践。让我们分析一下这种实践的一些好处：

+   **关注点分离**：通过将具有相似功能的代码分组在一起，我们在代码库中创建了反映它们所服务的业务域的上下文。例如，仅基于管理用户上下文的代码与仅基于管理产品的代码是分开的。在这些上下文之间建立清晰的边界可以确保依赖性最小化。

+   **可复用性**：采用模块化设计允许你创建诸如我们在本书中探讨的组件——例如，服务和中间件。在一个以关注点分离为目标的系统中，拥有可复用组件可以帮助在必要时以减少依赖创建的方式连接上下文。

+   **易于维护**：模块可以独立于彼此开发和测试，这使得多个开发者之间的并行开发更容易。模块化还支持开放-封闭原则，该原则指出，“*软件实体（类、模块、函数等）应该对扩展开放，但对修改封闭*。”

这意味着在一个理想的世界里，每当我们想要通过新功能扩展我们的最小 API 时，我们*不需要*改变现有的代码库来启用这个变化。

代码的有效组织通常受架构设计模式的影响。虽然这确实很重要，但仅仅重新组织项目的文件夹结构就可以在很大程度上使代码更易于阅读和维护。

让我们探索一些示例文件夹结构。

# 探索文件夹结构

大多数时候，简单地考虑项目文件夹在项目中的排列方式可以显著提高最小 API 的可读性和可维护性。我们正在寻找一个一致的系统来布局类和接口。让我们看看一些具体的文件夹结构，我们可以将这些结构应用到我们的项目中以实现这一点。

## 基于功能的模块化结构

在这种结构中，最小 API 项目按功能组织，每个功能都有自己的文件夹，包含与该功能相关的所有内容，无论使用哪种组件。以下是一个此类结构的示例：

```cs
/src
  /MyMinimalApiProject
    /Modules
      /Users
        UserEndpoints.cs
        UserService.cs
        UserRepository.cs
        User.cs
        UserDto.cs
        UserValidator.cs
      /Products
        ProductEndpoints.cs
        ProductService.cs
        ProductRepository.cs
        Product.cs
        ProductDto.cs
        ProductValidator.cs
    /Middleware
      ErrorHandlingMiddleware.cs
      AuthenticationMiddleware.cs
    /Configuration
      SwaggerConfig.cs
      DependencyInjectionConfig.cs
    /Utils
      DateTimeHelper.cs
      LoggingHelper.cs
    Program.cs
    appsettings.json
```

在这种结构中，期望开发者采用基于功能的心态。例如，如果你想添加一个与用户管理相关的端点，你会去一个基于用户的文件夹，而不是一个专门用于端点的文件夹。

如本章前面所述，文件夹结构可能是一个个人化和有争议的话题。有些人可能不喜欢在功能集的旗帜下混合组件类型，而其他人则喜欢这种基于领域的结构特性，并且不太关心每个领域内哪种组件在起作用。

## 分层模块结构

这种结构是我个人偏好的，因为我倾向于在考虑功能或业务领域之前先考虑组件类型。在分层模块结构中，项目首先按组件（例如，端点和服务）分组，然后进一步细分为业务模块/功能。

如果你像我一样，在考虑它所在的领域之前，倾向于先考虑我要创建或编辑的类或文件类型，这种文件夹结构将更适合你。然而，重要的是要注意，虽然这种结构在创建文件夹时优先考虑组件类型，但仍有一个专门的**领域**文件夹，用于存放实体模型和**数据传输对象**（**DTO**），这些对象描述了业务领域。以下是一个最小 API 项目中的分层模块文件结构示例：

```cs
/src
  /MyMinimalApiProject
    /Endpoints
      /Users
        UserEndpoints.cs
        UserValidator.cs
      /Products
        ProductEndpoints.cs
        ProductValidator.cs
    /Services
      /Users
        UserService.cs
      /Products
        ProductService.cs
    /Repositories
      /Users
        UserRepository.cs
      /Products
        ProductRepository.cs
    /Domain
      /Entities
        User.cs
        Product.cs
      /DTOs
        UserDto.cs
        ProductDto.cs
    /Middleware
      ErrorHandlingMiddleware.cs
      AuthenticationMiddleware.cs
    /Configuration
      SwaggerConfig.cs
      DependencyInjectionConfig.cs
    /Utils
      DateTimeHelper.cs
      LoggingHelper.cs
    Program.cs
    appsettings.json
```

现在我们已经探讨了如何在最小 API 项目中组织文件夹的一些简单示例，让我们看看在组织项目代码时可以采用的重复模式。这些模式被称为*设计模式*，就像文件夹结构一样，关于哪些模式构成最佳实践存在争议。

# 设计模式

这本书不是要告诉你哪些模式是最好的；相反，它旨在为你提供一些指导，告诉你如何一致地组织你的代码，以创建一个一致的 API 系统。以下是一些示例模式。

## 工厂模式

**工厂模式**旨在在不指定将要创建的确切对象类的情况下创建对象。之前我提到了开闭原则，工厂模式通过关闭代码以供修改，同时使其易于扩展，帮助最小 API 遵守这一原则。

让我们考虑一个示例用例，在这个用例中，你想要在不同的位置创建日志。一个位置是通过数据库，另一个是在文本文件中。

在未来，你可能希望添加更多的日志源，例如 Webhook 或第三方 API。工厂可以帮助你检索适用于你的用例的正确记录器，同时使添加新记录器变得简单，而无需更改旧记录器。

让我们看看如何通过实现工厂模式来改进日志记录的一个例子：

1.  首先，创建一个名为**ILogger**的接口，它将由所有记录器实现，无论它们在将日志保存到各自源时执行的具体日志是什么。**ILogger**是一个接口，它将代表一个实现将日志写入不同源逻辑的对象：

    ```cs
    public interface ILogger
    {
        void Log(string message);
    }
    ```

1.  接下来，创建两个实现**ILogger**的类。其中一个类，**FileLogger**，将用于将日志记录到文件，另一个类，**DatabaseLogger**，将记录到数据库：

    ```cs
    public class FileLogger : ILogger
    {
        public void Log(string message)
        {
            // Logic to log to a file
        }
    }
    public class DatabaseLogger : ILogger
    {
        public void Log(string message)
        {
            // Logic to log to a database
        }
    }
    ```

    这些类可能有不同的名称，但它们都是**ILogger**对象，这意味着它们必须实现**Log()**方法。

1.  此外，我们可以创建一个函数，它返回**ILogger**，如下所示：

    ```cs
    public static class LoggerFactory
    {
        public static ILogger CreateLogger(
            string loggerType
        )
        {
            return loggerType switch
            {
                "File" => new FileLogger(),
                "Database" => new DatabaseLogger(),
                _ => throw new ArgumentException(
                "Invalid logger type")
            };
        }
    }
    ```

在这个例子中，我们创建了一个**LoggerFactory**类，其中包含一个根据调用者输入的字符串内容返回相关日志类函数。如果**loggerType**参数无效，则会抛出异常，以便处理错误。

这样做的最大好处是，要添加另一个记录器，我们只需在**CreateLogger()**中的**switch**语句中添加新条目前创建一个新的实现**ILogger**的类。我们无需引入任何破坏性更改来扩展 API 中支持的记录器类型。

## 存储库模式

此模式为数据访问逻辑创建了一个抽象层，为访问数据库数据提供了更通用的 API，以最小的 API 实现。

在本书早期，我们探讨了 Entity Framework Core 以从数据库中访问数据。只需使用 Entity Framework Core，你的代码就已经使用了存储库模式，因为它提供了一个利用内置**DBContext**的存储库模式实现。

然而，仍然值得实现一个自定义存储库模式来处理数据，这样你可以进一步泛化解决方案，这意味着 Entity Framework Core 可以从最小 API 应用程序中替换出来，而不会影响整体的数据访问逻辑。

要在 Entity Framework Core 之上创建存储库模式，我们可以简单地为数据库中的每个实体创建一个类。每个存储库类通过依赖注入接收 Entity Framework 上下文，然后可以添加针对此实体的通用**创建、读取、更新、删除**（**CRUD**）操作。随后，每个存储库也可以注册为依赖注入，以便在应用程序的其他地方使用。

在以下示例中，**EmployeeRepository**反映了员工实体可用的数据操作。如描述，Entity Framework 上下文被注入为在仓库中使用的数据访问层：

```cs
public class EmployeeRepository : IEmployeeRepository
{
    private readonly MyCompanyContext _context;
    public EmployeeRepository(MyCompanyContext context)
    {
        _context = context;
    }
    public async Task<Employee> GetByIdAsync(int id)
    {
        return await _context.Employees.FindAsync(id);
    }
    public async Task<IEnumerable<Employee>> GetAllAsync()
    {
        return await _context.Employees.ToListAsync();
    }
    public async Task AddAsync(Employee employee)
    {
        await _context.Employees.AddAsync(employee);
        await _context.SaveChangesAsync();
    }
    public async Task UpdateAsync(Employee employee)
    {
        _context.Employees.Update(employee);
        await _context.SaveChangesAsync();
    }
    public async Task DeleteAsync(int id)
    {
        var employee = await
            _context.Employees.FindAsync(id);
        if (employee != null)
        {
            _context.Employees.Remove(employee);
            await _context.SaveChangesAsync();
        }
    }
}
```

从现在开始，如果决定替换 Entity Framework Core，唯一需要改变的是仓库。仓库的消费者将不会受到影响，因为他们调用仓库中的方法和函数将保持其原始签名，尽管其底层逻辑发生了变化。

## 策略模式

**策略模式**允许我们定义一组算法，每个算法由一个类表示。在存在多种执行操作方式的情况下，这种模式非常强大，因为我们可以无缝地在它们之间动态切换。

让我们看看一个涉及最小 API 端点的示例，该端点计算员工有多少年假。在这个例子中，根据不同的因素（如员工所在的国家、他们是否处于试用期以及他们在公司服务了多少年）有不同的计算休假的方法。

这里是一个计算休假的逻辑示例大纲（与任何国家的特定劳动法无关！）：

+   如果员工处于试用期，以下适用：

    +   奖励的最少休假天数是 10 天

    +   如果员工在英国，他们将额外获得三天

+   如果员工不在试用期，以下适用：

    +   奖励的最少休假天数是 16 天

    +   如果员工在英国，他们将额外获得三天

    +   对于每一年服务，将额外奖励一天

这里有很多因素在起作用，但首先，我们可以根据员工是否处于试用期来缩小计算休假的所需操作。这意味着我们有两种计算休假的策略，我们可以自动切换。我们如何实现这一点？

1.  首先，我们创建一个接口来表示一个策略。它规定我们需要一个**CalculateLeaveAllowance()**函数，该函数接受一个类型为**Employee**的参数并返回一个整数：

    ```cs
    public interface IAnnualLeaveStrategy
    {
        int CalculateLeaveAllowance(
            Models.Employee employee
        );
    }
    ```

1.  然后，我们将创建**ProbationaryAnnualLeaveStrategy**，该类实现了接口。在这个类中，**CalculateLeaveAllowance**将封装计算试用期员工可用的总休假的逻辑：

    ```cs
    public class ProbationaryAnnualLeaveStrategy
        : IAnnualLeaveStrategy
    {
        public int CalculateLeaveAllowance(
            Models.Employee employee
        )
        {
            var leaveTotal = 10;
            if(employee.Country == "United Kingdom")
            {
                leaveTotal += 3;
            }
            return leaveTotal;
        }
    }
    ```

1.  然后，同样可以为不在试用期的员工做同样的事情：

    ```cs
    public class PostProbationaryAnnualLeaveStrategy
        : IAnnualLeaveStrategy
    {
        public int CalculateLeaveAllowance(
            Models.Employee employee
        )
        {
            var leaveTotal = 16;
            if(employee.Country == "United Kingdom")
            {
                leaveTotal += 3;
            }
        leaveTotal += employee.YearsOfService;
            return leaveTotal;
        }
    }
    ```

1.  这两种策略应在**Program.cs**中进行注册以供依赖注入：

    ```cs
                builder.Services
                  .AddScoped<
                    ProbationaryAnnualLeaveStrategy
                  >();
                builder.Services
                  .AddScoped<
                    PostProbationaryAnnualLeaveStrategy
                  >();
    ```

1.  最后，可以创建一个端点来计算给定**Employee** ID 的年假。在端点内部，我们根据员工是否处于试用期来创建策略。然后我们使用客户端发送的 ID 检索**Employee**，并调用**CalculateLeaveAllowance()**函数来获取结果。这样，根据客户端在请求中发送的数据，自动使用适当的策略来执行正确的逻辑：

    ```cs
    app.MapGet(
        "/calculate-employee-leave-allowance/
            {employeeId}",
        async (int employeeId,
            bool employeeOnProbation,
            [FromServices]
            EmployeeService employeeService) =>
    {
        IAnnualLeaveStrategy annualLeaveStrategy =
            employeeOnProbation
              ? new ProbationaryAnnualLeaveStrategy()
              : new PostProbationaryAnnualLeaveStrategy();
        var employee = await
            employeeService.GetEmployeeById(employeeId);
        return annualLeaveStrategy
            .CalculateLeaveAllowance(employee);
    });
    ```

通过将逻辑分离成单独的策略，策略模式允许最小化 API 符合开闭原则。它是通过允许我们通过添加新功能来扩展代码库，而不是修改现有代码来实现的。

它还提供了自包含性，意味着执行类似任务的方式不会相互污染，这减少了出现错误的可能性。

单独的设计模式并不能构建一个健壮的系统。健壮性是通过良好地理解错误发生的位置和方式来实现的。

需要服务年数属性

如果你在*第九章*中使用的原始**Employee**对象上添加一个名为**YearsOfService**的**int**属性，本节中的示例将能够工作。我们假设你在尝试遵循这个策略模式示例之前已经完成了这项工作。

考虑到这一点，让我们继续探讨在最小化 API 中关于错误处理的最佳实践。

# 错误处理

当谈到错误处理和健壮性的话题时，第一个是方法，第二个是结果。通过实施有效的错误处理，我们实现了健壮性。

因此，虽然在整个代码库中使用**try/catch**很重要，但在顶层以标准化的方式处理错误仍然至关重要。对于最小化 API，中间件是处理顶层错误的有效方式。让我们通过一个例子来探讨。

注意

本节假设你已经阅读了*第五章*或者你已经对如何在 ASP.NET 中编写中间件有深入的了解。

实现中间件确保我们在最小化 API 中有一个全局的错误处理解决方案。把它想象成一个巨大的**try/catch**，它围绕了所有最小化 API 的端点。

由于我们在*第五章*中广泛探讨了中间件，我们不需要详细说明中间件是如何构建的，因此我们将直接进入一个错误处理中间件类的例子，如即将到来的代码块所示。

首先，我们为中间件创建一个类，包括一个构造函数和一个可以用来启动中间件逻辑的**InvokeAsync**方法：

```cs
public class ErrorHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ErrorHandlingMiddleware>
        _logger;
    public ErrorHandlingMiddleware(
        RequestDelegate next,
        ILogger<ErrorHandlingMiddleware> logger
    )
    {
        _next = next;
        _logger = logger;
    }
    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(
                ex, "An unhandled exception occurred."
            );
            await HandleExceptionAsync(context, ex);
        }
    }
```

在此之后，我们可以添加一个接受当前**HttpContext**的方法来处理任何检测到的错误：

```cs
    private static Task HandleExceptionAsync(
        HttpContext context, Exception exception)
    {
    If (context.Response.HasStarted) return;
    context.Response.ContentType = "application/json";
    context.Response.StatusCode =
        (int)HttpStatusCode.InternalServerError;
        var response = new
        {
            message =
                "An unexpected error occurred. Please try
                again later.",
            details = exception.Message
        };
        return context.Response.WriteAsJsonAsync(response);
    }
}
```

值得注意的是，这个用于错误处理的中间件示例可以很容易地替换*第五章*中的类似错误处理示例。

此中间件将捕获请求管道中更高层抛出的异常，确保错误通过**HttpContext**返回给请求客户端。这确保了无论调用的是哪个端点，客户端都能收到一致的错误响应。

如您从本书前面的内容中了解到的，中间件，例如依赖注入服务，必须在**Program.cs**中进行注册。将此中间件类注册为第一个要注册的中间件，然后创建一个示例端点，该端点会抛出异常：

```cs
app.UseMiddleware<ErrorHandlingMiddleware>();
app.MapGet(
    "/error",
    () => {
        throw new InvalidOperationException(
            "This is a test exception");
    });
```

您应该会发现，不仅异常的消息被返回，而且还有一个一致性的通用消息，就像这里显示的那样：

```cs
{
    "message": "An unexpected error occurred. Please try
        again later.",
    "details": "This is a test exception"
}
```

在开发任何 API 时，还应保持一致的实践是安全开发。让我们探讨一些您可以在授权请求到您的最小化 API 时应用的良好安全实践。

# 安全性考虑

在最小化 API 中，有两个关键的安全领域——身份验证和授权。无论它们之间的差异如何，对其实施的态度应该大致相同——**不要自行实现**。

这个格言作为一个警告，表明一个经过验证的安全框架通常比你自己设计的更安全。

让我们首先看看身份验证和授权之间的区别，以及您如何使用知名技术**JSON Web Tokens**（**JWTs**）实现良好的安全级别。

## 身份验证

**身份验证**验证访问您的 API 的用户或系统的身份。它允许您只允许合法请求进入系统。

JWT 因其最小化 API 中的无状态身份验证功能而被广泛使用。用户进行一次身份验证并接收一个令牌，该令牌包含在随后的请求中以访问受保护资源。

## 授权

**授权**是一种检查已验证用户是否在其特定权限内访问资源的手段。在 JWT 中，这些权限被称为**声明**。

一个声明可以是资源的名称或用户类型，也可以是一个角色。无论如何，JWT 都内置了在最小化 API 中针对特定端点定义和验证声明的功能：

1.  要开始使用此授权框架，我们首先需要添加**Microsoft.AspNetCore.Authentication.JwtBearer** NuGet 包。通过**工具** | **NuGet 包管理器** | **包管理器控制台**进行此操作，该控制台可通过**工具**菜单访问：

    ```cs
    dotnet add package Microsoft
        .AspNetCore.Authentication.JwtBearer
    ```

1.  然后，我们需要确保**Program.cs**有相关的命名空间：

    ```cs
    using Microsoft.AspNetCore.Authentication.JwtBearer;
    using Microsoft.IdentityModel.Tokens;
    using System.Text;
    using System.IdentityModel.Tokens.Jwt;
    using System.Security.Claims;
    using Microsoft.AspNetCore.Authorization;
    ```

1.  JWT 可以作为中间件实现，因此首先应在**Program.cs**中设置：

    ```cs
    builder.AddAuthorization();
    builder.Services.AddAuthentication(
        JwtBearerDefaults.AuthenticationScheme)
        .AddJwtBearer(options =>
        {
            options.TokenValidationParameters =
                new TokenValidationParameters
            {
                ValidateIssuer = true,
                ValidateAudience = true,
                ValidateLifetime = true,
                ValidateIssuerSigningKey = true,
                ValidIssuer = "https://yourdomain.com",
                ValidAudience = "https://yourdomain.com",
                IssuerSigningKey =
                    new SymmetricSecurityKey(
                        Encoding.UTF8.GetBytes(
                            "A_Not_Very_Secret_Key_1234567
                                890"
                        )
                    )
            };
        });
    var app = builder.Build();
    app.UseAuthentication();
    app.UseAuthorization();
    ```

1.  此注册指定了如何验证您的令牌。接下来，我们需要提供一种生成令牌的方法。我们可以通过创建一个专门的 API 端点来实现：

    ```cs
        app.MapGet("/generate-token", () =>
        {
            var tokenHandler =
                new JwtSecurityTokenHandler();
            var key = Encoding.UTF8.GetBytes(
                " A_Not_Very_Secret_Key_1234567890"
            );
            var tokenDescriptor =
                new SecurityTokenDescriptor
            {
                Subject = new ClaimsIdentity(new[]
                {
        new Claim(ClaimTypes.Name, "TestUser"),
        new Claim(ClaimTypes.Role, "Admin")
    }),
                Expires = DateTime.UtcNow.AddHours(1),
                Issuer = "https://yourdomain.com",
                Audience = "https://yourdomain.com",
                SigningCredentials =
                    new SigningCredentials(
                        new SymmetricSecurityKey(key),
                            SecurityAlgorithms
                                .HmacSha256Signature
                    )
            };
            var token =
                tokenHandler.CreateToken(tokenDescriptor);
            var tokenString =
                tokenHandler.WriteToken(token);
            return Results.Ok(tokenString);
        });
    ```

1.  最后，既然我们已经有了创建 JWT 令牌的方法，我们可以创建一个需要它的端点。让我们再次创建一个简单的**GET**端点，但这次，我们添加一个**[Authorize]**属性并将**RequireAuthorization()**链接到它：

    ```cs
    app.MapGet(
        "/secure",
        [Authorize] () => "This is a secure endpoint")
        .RequireAuthorization();
    ```

1.  为了测试这个，我们可以对这个端点发起一个**GET**请求，并将返回的 JWT 令牌作为承载令牌添加。使用一个示例 JWT 令牌，头部看起来可能如下所示：

    ```cs
    "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1bmlxdWVfbmFtZSI6IlRlc3RVc2VyIiwicm9sZSI6IkFkbWluIiwibmJmIjoxNjQyNzE2MzY0LCJleHAiOjE2NDI3MTk5NjQsImlhdCI6MTY0MjcxNjM2NCwiaXNzIjoiaHR0cHM6Ly95b3VyZG9tYWluLmNvbSIsImF1ZCI6Imh0dHBzOi8veW91cmRvbWFpbi5jb20ifQ.-Ym30PjdvWl5eYdltZd0yA5XQ1ikf5D4KrDlmHMIj0s"
    ```

    ASP.NET core 将根据你在注册 JWT 中件间时指定的验证参数对请求进行身份验证。

1.  进一步来说，如果我们想创建一个更受限制的端点，只有具有角色声明**Admin**的用户才能访问，我们可以在**[Authorize]**属性中添加一个参数：

    ```cs
    app.MapGet(
        "/admin",
        [Authorize(Roles = "Admin")] () =>
    {
        return Results.Ok("Welcome, Admin!");
    });
    ```

使用 JWT 令牌，我们的最小 API 提供了对未经授权活动的保护。然而，仅此并不能保证授权使用不会被滥用。

实际上，这是无法保证的，但你可以采取的一个额外步骤来确保最小 API 不会被滥用，就是在指定时间段内限制可以发出的请求数量。这种做法被称为**速率限制**。

## 速率限制

通过控制客户端在特定时间段内对最小 API 发出的请求数量，速率限制可以帮助防止系统因请求过多而超负荷，无论请求是否合法。

让我们探索一个 ASP.NET core 最小 API 中速率限制的简单示例。

首先，通过 NuGet 包管理器控制台添加**AspNetCoreRateLimit**包。您可以通过在 Visual Studio 中点击**工具** | **管理 NuGet 包** | **包管理器控制台**来打开它。这将打开控制台，您可以在其中输入以下内容：

```cs
dotnet add package AspNetCoreRateLimit
```

然后，通过**Program.cs**添加速率限制（以及内存缓存）：

```cs
using AspNetCoreRateLimit;
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddMemoryCache();
builder.Services.Configure<IpRateLimitOptions>(options =>
{
    options.GeneralRules = new List<RateLimitRule>
    {
        new RateLimitRule
        {
            Endpoint = "*",
            Limit = 10,
            Period = "1m"
        }
    };
});
builder.Services
    .AddSingleton<IRateLimitConfiguration,
        RateLimitConfiguration>();
builder.Services.AddInMemoryRateLimiting();
var app = builder.Build();
app.UseIpRateLimiting();
app.MapGet("/", () => "Hello, World!");
app.Run();
```

此配置将限制所有端点每分钟每个 IP 地址最多 100 个请求。如果超过限制，客户端将收到一个**429 Too Many Requests**的响应。

要尝试这个示例，将每分钟的请求数量最大值更改为**2**，看看结果如何。

**AspNetCoreRateLimit**与**Microsoft.AspNetCore.RateLimiting**

除了**AspNetCoreRateLimit**之外，还有另一种速率限制的选项，如前例所示。**Microsoft.AspNetCore.RateLimiting**也可以用来管理允许请求的速率。**AspNetCoreRateLimit**是一个第三方库，而**Microsoft.AspNetCore.RateLimiting**是一个内置的中件间。对于配置，**AspNetCoreRateLimit**使用 JSON 或程序性配置，而**Microsoft.AspNetCore.RateLimiting**使用带有**RateLimiterOptions**的程序性配置。

**AspNetCoreRateLimit**提供了不同的策略和分布式速率限制的更多灵活性，而**Microsoft.AspNetCore.RateLimiting**则专注于内置算法和端点特定策略。

我们只是触及了可以实施以提高最小 API 弹性的一些实践，但我们所探讨的例子是改进它们设计的一个良好起点。让我们回顾一下本章中我们看到的实践。

# 摘要

我们首先观察了实施有效设计实践的合理性——即，从维护的可扩展性和安全性角度来看，增加弹性。

然后，我们讨论了文件夹结构，概述了几个文件夹结构示例，用于在新的项目中使用。

我们随后以工厂、仓库和策略三种设计模式的形式探讨了三个设计模式，为以可扩展的方式安排最小 API 代码提供了坚实的基础。

然后，我们简要回顾了一个示例，说明中间件如何充当“全局捕获”，标准化向客户端返回错误响应的方式，最后探索了一些简单的请求认证和限制它们处理速率的方法。

我们正接近本书的结尾，这意味着我们需要学习如何管理一个在用户群体中部署和活跃的最小 API。考虑到这一点，下一章将涵盖在我们将最小 API 提供给更广泛的世界时需要考虑的最重要的事情——测试、部署和文档。
