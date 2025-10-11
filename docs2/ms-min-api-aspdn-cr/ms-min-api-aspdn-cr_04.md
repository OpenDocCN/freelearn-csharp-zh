# 4

# 在最小 API 项目中实现依赖注入

在本书的这一章中，我们将讨论.NET 6.0 中最小 API 的一些基本主题。我们将学习它们如何与我们之前在.NET 的旧版本中使用的基于控制器的 Web API 不同。我们还将尝试强调这种新的 API 编写方法的优势和劣势。

在本章中，我们将涵盖以下主题：

+   什么是依赖注入？

+   在最小 API 项目中实现依赖注入

# 技术要求

为了跟随本章的解释，你需要创建一个 ASP.NET Core 6.0 Web API 应用程序。你可以参考*第二章**，探索最小 API 及其优势*的技术要求部分，了解如何操作。

本章中所有的代码示例都可以在本书的 GitHub 仓库中找到，网址为[`github.com/PacktPublishing/Minimal-APIs-in-ASP.NET-Core-6/tree/main/Chapter04`](https://github.com/PacktPublishing/Minimal-APIs-in-ASP.NET-Core-6/tree/main/Chapter04)。

# 什么是依赖注入？

一段时间以来，.NET 已经原生支持**依赖注入**（通常称为**DI**）软件设计模式。

依赖注入是.NET 中实现服务类及其依赖项之间**控制反转**（**IoC**）模式的一种方式。顺便说一句，在.NET 中，许多基本服务都是使用依赖注入构建的，例如日志记录、配置和其他服务。

让我们通过一个实际例子来了解它是如何工作的。

一般而言，依赖是一个依赖于另一个对象的对象。在以下示例中，我们有一个名为`LogWriter`的类，其中只有一个方法，称为`Log`：

```cs
public class LogWriter
{
    public void Log(string message)
    {
        Console.WriteLine($"LogWriter.Write
          (message: \"{message}\")");
    }
}
```

项目中的其他类或另一个项目中的类可以创建`LogWriter`类的实例并使用`Log`方法。

看看以下示例：

```cs
public class Worker
{
    private readonly LogWriter _logWriter = new LogWriter();
    protected async Task ExecuteAsync(CancellationToken 
                                      stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            _logWriter.Log($"Worker running at: 
             {DateTimeOffset.Now}");
             await Task.Delay(1000, stoppingToken);
        }
    }
}
```

这个类直接依赖于`LogWriter`类，并且在你的项目的每个类中都是硬编码的。

这意味着如果你想要更改`Log`方法；例如，你将不得不替换解决方案中每个类的实现。

如果你想在你的解决方案中实现单元测试，前面的实现有一些问题。创建`LogWriter`类的模拟并不容易。

通过对我们代码的一些更改，依赖注入可以解决这些问题：

1.  使用接口来抽象依赖。

1.  在.NET 中注册依赖注入到内置的服务连接。

1.  将服务注入到类的构造函数中。

前面的内容可能看起来需要你对代码进行大的改动，但实际上它们很容易实现。

让我们看看我们如何可以通过之前的例子实现这个目标：

1.  首先，我们将创建一个具有我们日志抽象的`ILogWriter`接口：

    ```cs
    public interface ILogWriter
    {
        void Log(string message);
    }
    ```

1.  接下来，在一个名为`ConsoleLogWriter`的实类中实现这个`ILogWriter`接口：

    ```cs
    public class ConsoleLogWriter : ILogWriter
    {
        public void Log(string message)
        {
            Console.WriteLine($"ConsoleLogWriter.
            Write(message: \"{message}\")");
        }
    }
    ```

1.  现在，修改`Worker`类，将显式的`LogWriter`类替换为新的`ILogWriter`接口：

    ```cs
    public class Worker
    {
        private readonly ILogWriter _logWriter;
        public Worker(ILogWriter logWriter)
        {
            _logWriter = logWriter;
        }
        protected async Task ExecuteAsync
          (CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                _logWriter.Log($"Worker running at: 
                                 {DateTimeOffset.Now}");
                 await Task.Delay(1000, stoppingToken);
            }
        }
    }
    ```

如你所见，以这种方式工作非常简单，优势很大。以下是依赖注入的一些优点：

+   可维护性

+   可测试性

+   可重用性

现在我们需要执行最后一步，即在应用程序启动时注册依赖项。

1.  在`Program.cs`文件的顶部添加以下代码行：

    ```cs
    builder.Services.AddScoped<ILogWriter, ConsoleLogWriter>();
    ```

在下一节中，我们将讨论依赖注入生命周期之间的差异，这是在使用最小 API 项目进行依赖注入之前你需要理解的概念。

## 理解依赖注入的生命周期

在上一节中，我们学习了在项目中使用依赖注入的好处以及如何将我们的代码转换为使用它。

在最后一段中，我们将我们的类作为服务添加到.NET 的`ServiceCollection`中。

在本节中，我们将尝试理解每种依赖注入的生命周期差异。

服务生命周期定义了对象在容器创建之后将存活多长时间。

当它们被注册时，依赖项需要生命周期定义。这定义了何时创建新的服务实例。

在以下列表中，你可以找到.NET 中定义的生命周期：

+   **瞬态**：每次请求时都会创建该类的新实例。

+   **作用域**：每个作用域创建该类的一个新实例，例如，对于同一个 HTTP 请求。

+   **单例**：仅在第一次请求时创建该类的新实例。下一次请求将使用相同类的相同实例。

在网络应用程序中，你通常只找到前两种生命周期，即瞬态和作用域。

如果你有一个需要单例的特殊用例，这并不被禁止，但为了最佳实践，建议在 Web 应用程序中避免使用它们。

在前两种情况下，即瞬态和作用域内，服务在请求结束时被销毁。

在下一节中，我们将看到如何通过一个简短的演示来实现我们在上一节中提到的所有概念（依赖注入的定义及其生命周期），你可以将其作为你下一个项目的起点。

# 在最小 API 项目中实现依赖注入

在理解了如何在 ASP.NET Core 项目中使用依赖注入之后，让我们尝试理解如何在我们的最小 API 项目中使用依赖注入，从使用`WeatherForecast`端点的默认项目开始。

这是`WeatherForecast` GET 端点的实际代码：

```cs
app.MapGet("/weatherforecast", () =>
{
    var forecast = Enumerable.Range(1, 5).Select(index =>
    new WeatherForecast
    (
        DateTime.Now.AddDays(index),
        Random.Shared.Next(-20, 55),
        summaries[Random.Shared.
        Next(summaries.Length)]
    ))
    .ToArray();
    return forecast;
});
```

正如我们之前提到的，这段代码可以工作，但它不容易测试，尤其是天气的新值的创建。

最佳选择是使用一个服务来创建假值，并使用依赖注入来使用它。

让我们看看我们如何更好地实现我们的代码：

1.  首先，在`Program.cs`文件中，添加一个名为`IWeatherForecastService`的新接口，并定义一个返回`WeatherForecast`实体数组的函数：

    ```cs
    public interface IWeatherForecastService
    {
               WeatherForecast[] GetForecast();
    }
    ```

1.  下一步是创建从接口继承的类的实际实现。

代码应该看起来像这样：

```cs
public class WeatherForecastService : IWeatherForecastService
{
}
```

1.  现在将项目模板中的代码复制粘贴到我们新实现的服务中。最终的代码如下所示：

    ```cs
    public class WeatherForecastService : IWeatherForecastService
    {
        public WeatherForecast[] GetForecast()
        {
            var summaries = new[]
            {
                "Freezing", "Bracing", "Chilly", "Cool", 
                "Mild", "Warm", "Balmy", "Hot", "Sweltering", 
                "Scorching"
            };
            var forecast = Enumerable.Range(1, 5).
            Select(index =>
            new WeatherForecast
            (
                DateTime.Now.AddDays(index),
                Random.Shared.Next(-20, 55),
                summaries[Random.Shared.Next
                (summaries.Length)]
            ))
            .ToArray();
            return forecast;
        }
    }
    ```

1.  现在我们已经准备好将`WeatherForecastService`的实现添加到我们的项目中作为依赖注入。为此，在`Program.cs`文件的第一行代码下方插入以下行：

    ```cs
    builder.Services.AddScoped<IWeatherForecastService, WeatherForecastService>();
    ```

当应用程序启动时，将我们的服务插入到服务集合中。我们的工作还没有完成。

我们需要在`WeatherForecast`端点的默认`MapGet`实现中使用我们的服务。

最小 API 有自己的参数绑定实现，并且非常容易使用。

首先，为了使用依赖注入实现我们的服务，我们需要从端点中移除所有旧代码。

在移除代码后，端点的代码看起来像这样：

```cs
app.MapGet("/weatherforecast", () =>
{
});
```

我们可以通过简单地用新代码替换旧代码来轻松改进我们的代码并使用依赖注入：

```cs
app.MapGet("/weatherforecast", (IWeatherForecastService weatherForecastService) =>
{
    return weatherForecastService.GetForecast();
});
```

在最小 API 项目中，服务集合中服务的实际实现被作为参数传递给函数，并且可以直接使用它们。

有时，在启动阶段，你可能需要在主函数中直接使用依赖注入中的服务。在这种情况下，你必须直接从服务集合中检索实现实例，如下面的代码片段所示：

```cs
using (var scope = app.Services.CreateScope())
{
    var service = scope.ServiceProvider.GetRequiredService
                  <IWeatherForecastService>();
    service.GetForecast();
}
```

在本节中，我们从默认模板开始，在最小 API 项目中实现了依赖注入。

我们重用了现有的代码，但用更符合未来维护和测试的架构逻辑来实现它。

# 摘要

依赖注入是实现现代应用的重要方法之一。在本章中，我们学习了依赖注入是什么，并讨论了其基本原理。然后，我们看到了如何在最小 API 项目中使用依赖注入。

在下一章中，我们将关注现代应用的另一个重要层，并讨论如何在最小 API 项目中实现日志策略。
