

# 处理 HTTP 方法与路由

在 *第二章* 中，我们讨论了如何在最小化 API 中定义端点和使用路由的方法。那是一个高层次的观点。

然而，在本章中，我们将更详细地讨论如何配置路由和端点以处理传入的请求。我们将更详细地介绍如何使用路由参数来更具体地说明每个端点接收到的所需参数，我们还将探讨请求验证的示例，确保请求正确形成，并在必要时发出相关响应。

最后，如果一个 API 的端点不能充分地从接收无效数据中恢复，那么它就不能被认为是可靠的，因此我们还将探讨如何优雅地处理验证错误。

为了更好地理解这些主题，我们将使用一个用于管理任务的示例应用程序。该应用程序是生产力套件的一部分，它有一个用于管理待办事项列表和项目的 API。通过构建这个 API 的元素，你将更深入地了解最小化 API 如何接收请求以及如何处理它们。

总结来说，本章将涵盖以下主要主题：

+   处理请求

+   在 Todo API 中定义端点

+   管理路由参数

+   请求验证和错误处理

# 技术要求

本章的代码可在 GitHub 仓库中找到：[`github.com/PacktPublishing/Minimal-APIs-in-ASP.NET-9`](https://github.com/PacktPublishing/Minimal-APIs-in-ASP.NET-9)。

如果你已经安装了带有 .NET 9 的 Visual Studio 2022 / Visual Studio Code，你当然可以边读章节边编写代码。

# 处理请求

要处理传入的请求，我们首先需要一个最小化的 API 端点集合，以便将这些请求发送到。让我们回顾一下在 *第二章* 中我们探索的内容，即创建具有不同 HTTP 方法的最小化 API 端点（**GET**、**POST**、**PUT**、**DELETE** 和 **PATCH**）。我们可以通过创建一些静态的 *模拟* 数据来刷新我们的记忆，这些数据将代表我们的 API 处理的任务实体。然后，我们可以定义一些简单的端点来操作或查询这些数据。

让我们先创建模拟数据。我们将通过创建一个简单的 **TodoItem** 类和一个静态列表来实现，该列表将包含这个类的实例：

```cs
public class TodoItem
{
    public int Id { get; set; }
    public DateTime StartDate { get; set; }
    public DateTime DueDate { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public string Assignee { get; set; }
    public int Priority { get; set; }
    public bool IsComplete { get; set; }
}
```

目前，**TodoItem** 类可以保持相当简单。随着我们对需求的进一步了解，它可以扩展到具有更具体属性的类。同样的方法也可以用于下一段代码，目前它将是一个 **TodoItem** 的列表，简单地称为 **ToDoItems**。在这个列表中，我们存储实例化的 **TodoItem**，以便在请求期间由端点处理。让我们将这个列表放在 **Program.cs** 中：

```cs
List<TodoItem> ToDoItems = new List<TodoItem>();
```

现在我们有了以列表形式存在的临时数据存储解决方案，我们可以专注于创建处理请求和管理 todo 项的端点。

# 在 Todo API 中定义端点

在创建最小 API 时，尽可能简单是明智的。毕竟，*最小 API* 这个名字本身就意味着简单。但这并不仅仅是为了简单。目前，我们的 API 只需要覆盖一个领域：todo 项。当然，范围可能会在未来进一步扩大，最小 API 仍然可以以可扩展的方式构建，因此具有一定的未来性，但在更多需求变得明显之前（例如，将待办事项分配给用户，将待办事项添加到特定项目等），目标是保持最小化。考虑到这一点，我们现在将保持我们的端点在 **Program.cs** 中。

我们现在应该问自己一个简单的问题：*在这个 API 中，我想做什么？*

我的意思是 API 需要促进的动作。例如，获取 todo 项，更新 todo 项，删除 todo 项等。

在理解构成您 API 行动的一部分的 **动词** 时，我们可以确定所需的 HTTP 方法。考虑对 todo 项的基本操作。我们当然会想要 *检索* todo 项。这将需要一个 HTTP **GET** 方法。此外，我们还将想要 *创建* 一个 **TodoItem** 对象。这将需要一个 HTTP **POST** 方法。让我们从第一个例子开始，检索一些 todo 项，然后在此基础上构建。

## 获取 todo 项

如果您已经阅读了 *第二章*，您将已经看到如何在最小 API 中创建 HTTP **GET** 方法作为端点的示例。对于这个项目，我们首先创建一个端点，它简单地检索我们之前创建的 **List<TodoItem>** 的内容。

为了实现这一点，我们需要将 HTTP **GET** 方法映射到我们的最小 API 所运行的 **WebApplication** 实例上。在 **WebApplication** 中有几个函数可以实现这一点。每个函数都以单词 **Map** 开头，后面跟着相关的方法动词。在这个例子中，我们将使用 **MapGet()** :

```cs
app.MapGet("/todoitems", () =>
{
});
```

在此代码中，一个 HTTP **GET** 方法已被映射到 **/todoitems** 路由，这意味着如果用户请求 API 的基本 URL，然后是 **/todoitems**，则会到达此端点。

例如，如果我们的 URL 域名是 [`example.org/reallysimpletodoapi`](https://example.org/reallysimpletodoapi)，访问 [`example.org/reallysimpletodoapi/todoitems`](https://example.org/reallysimpletodoapi/todoitems) 将会到达这个端点。

现在我们可以进入请求的处理，这发生在函数体中。注意，我们创建的端点在路由定义之后有一个 lambda 表达式。当前表达式体是空的。我们将在该表达式体中通过检索所需数据并响应用户来处理请求。

在这种情况下，因为我们只是返回 **ToDoItems** 列表的内容，数据已经准备好了，但我们如何将数据返回给客户端呢？ASP.NET 提供了一个名为 **IResult** 的辅助对象，其 **Results** 对象公开了各种用于响应请求的方法。这处理了返回响应的基本方面，如状态码、响应体等。

对于这个简单的 HTTP **GET** 方法，我们只需通过返回 **Results.OK(ToDoItems)** 的结果，就可以返回一个 HTTP **200 OK** 的响应，并附带所需的数据。这个函数生成相关的状态码，并接受一个类型为 **object** 的参数，代表应返回给客户端的数据。一旦添加，端点应该看起来像这样：

```cs
app.MapGet("/todoitems", () =>
{
    return Results.Ok(ToDoItems);
});
```

到目前为止，我们的重点一直是将请求路由到 API 以检索数据。我们还需要在系统中创建新数据；因此，让我们将注意力转向待办事项的创建。

## 创建待办事项

让我们看看另一个对 API 至关重要的操作：创建实体。要创建一个新的 **TodoItem**，我们会使用 HTTP **POST** 方法。

HTTP **POST** 方法的映射与我们在映射 **GET** 方法时编写的代码类似。再次使用以 **Map** 为前缀的方法。这个方法是 **MapPost()**。然而，与我们的 **GET** 方法相比，语法上有一点不同，因为我们现在需要接收一个数据结构。在创建 **TodoItem** 的情况下，我们将需要客户端发送一个类型为 **TodoItem** 的对象，客户端以 JSON 格式表示。然后 ASP.NET 将负责将 JSON 解析为 **TodoItem** 的强类型实例，我们可以在处理请求时使用它。

为了让方法能够接收作为传入请求一部分的对象，我们可以在端点体中的 lambda 表达式开头的括号中添加该对象。例如，注意我们之前创建的用于检索 **TodoItem** 的端点，它有这些空括号：

```cs
app.MapGet("/todoitems", () =>
```

我们对最小 API 端点的处理是通过 lambda 表达式表示的。Lambda 表达式以参数的形式开始，这些参数通过这些空括号传入，如下面的代码所示。这意味着对于我们的 HTTP **POST** 端点，我们可以在添加的 **MapPost()** 方法中添加一个类型为 **TodoItem** 的参数，如下所示：

```cs
app.MapPost("/todoitems", (TodoItem item) =>
{
});
```

现在，我们有一个 HTTP **POST** 端点，位于 **/todoitems** 路径上，就像我们的 HTTP **GET** 端点一样。不同之处在于，它不仅响应不同的 HTTP 动词，而且还需要客户端发送一个与 **TodoItem** 结构相对应的 JSON 负载。

在端点内的 lambda 表达式中我们还没有任何内容，这意味着当客户端发送请求时，不会发生任何操作。让我们最终通过将传入的 **TodoItem** 添加到列表中，然后返回相关响应来处理请求：

```cs
app.MapPost("/todoitems", (TodoItem item) =>
{
    ToDoItems.Add(item);
    return Results.Created();
});
```

就像在 **GET** 示例中一样，我们使用 **Results** 来构建响应并将其发送回客户端。然而，在这种情况下，我们选择了稍微不同的响应。因为请求的目的是创建一个实体，所以在成功创建后返回一个 **HTTP 201 CREATED** 状态码是合适的，因此使用了 **Results.Created();** 。

## 更新现有的 Todo 项目

当涉及到更新一个 **Todoitem** 时，我们有几个 HTTP 方法可供选择。让我们从 HTTP **PUT** 方法开始。

HTTP **PUT** 要求客户端发送要更新的实体的完整副本。然后它会用副本替换实体。这是一个 *完整更新*。因此，我们需要创建一个端点，它接收请求中的 **TodoItem** 作为对象参数，在找到我们列表中的相关项目并将其替换为传入的 **TodoItem** 之前：

```cs
app.MapPut("/todoitems", (TodoItem item) =>
{
});
```

接下来，请求应该通过找到我们打算更新的 **TodoItem** 来处理。我们可以使用一个 **Language Integrated Query** ( **LINQ** ) 查询通过其唯一的 ID 来找到该项目，使用 **FindIndex();** 。

LINQ 查询

在 C# 中，使用 lambda 表达式的 LINQ 查询可以让你轻松地在集合（如列表）中搜索和操作数据。你首先定义你的数据源，然后使用如 **Where** 的方法来过滤数据，使用 **Select** 来选择你想要的数据。每个方法都接受一个 lambda 表达式，这是一个简单的函数，它定义了你的标准。在我们的例子中，我们使用 LINQ 查询来找到列表中与给定项目具有相同 ID 的项目的索引。

一旦找到，**TodoItem** 可以被替换为传入的项目：

```cs
app.MapPut("/todoitems", (TodoItem item) =>
{
    var index = ToDoItems.FindIndex(x => x.Id == item.Id);
    if (index == -1)
    {
        return Results.NotFound();
    }
    ToDoItems[index] = item;
    return Results.NoContent();
});
```

注意，在这个例子中我们没有再次返回 **Results.OK**。因为我们只是更新一个资源，客户端不期望返回内容；所以我们通过返回一个 **HTTP 204 NO CONTENT** 状态码来表示成功，使用 **Results.NoContent();** 。

通过 HTTP **PATCH**更新**待办事项**的处理方式略有不同。与**PUT**不同，我们通过再次找到相关项目来处理请求，但这次，我们只更改项目的一些特定属性，这些属性由请求参数指定。这通常用于你希望在路由上创建一个特定更新端点的场景。因此，在这种情况下，我们不再在**/todoitems**路由上创建端点。相反，我们将通过将**PATCH**方法映射到**/updateTodoItemDueDate**路由来具体说明我们想要更改的内容。在这个例子中，我们创建了一个旨在单一目的的端点——更改目标**待办事项**的截止日期：

```cs
app.MapPatch("/updateTodoItemDueDate/{id}",
    (int id, DateTime newDueDate) =>
{
    var index = ToDoItems.FindIndex(x => x.Id == id);
    if (index == -1)
    {
        return Results.NotFound();
    }
    ToDoItems[index].DueDate = newDueDate;
    return Results.NoContent();
});
```

代码看起来像我们创建的**PUT**方法，但你可以看到参数有所不同。我们不是要求客户端发送完整的**ToDoItems**对象，而是要求两个参数，一个**int**参数（通过 ID 查找目标项目）和一个包含新截止日期的**DateTime**参数。然后可以使用另一个 LINQ 查询找到目标项目，然后只更新其**DueDate**属性。

到目前为止，我们已经处理了需要获取所有项目、创建项目以及更新项目的场景。接下来，我们将探讨我们打算获取单个项目和删除项目的场景。然而，为了做到这一点，我们首先需要了解路由参数的概念。

# 管理路由参数

**路由参数**赋予我们从 API 端点的 URL 中捕获值的能力。这在许多需要针对特定实体进行操作的场景中非常有用，例如在通过 ID 请求**待办事项**时。

添加路由参数非常简单，并且使用花括号来定义从 URL 中捕获的参数。

让我们以一个**GET**请求为例。在这个请求中，客户端请求具有以下 ID 的**待办事项**：

```cs
app.MapGet("/todoitems/{id}", (int id) =>
{
    var index = ToDoItems.FindIndex(x => x.Id == id);
    if (index == -1)
    {
        return Results.NotFound();
    }
    return Results.Ok(ToDoItems[index]);
});
```

就像我们创建的用于获取所有待办事项的通用**GET**请求一样，这个端点位于**/todoitems**路由上。然而，在 URL 中附加了该路由的一个额外部分。客户端预期也会添加一个 ID 值作为额外的 URL 部分。这通过路由中存在的**{id}**来表示。

这种在花括号中使用参数的方法是 ASP.NET 处理路由 URL 中动态内容的方式。通过模板化的一种形式，它可以替换我们添加**{id}**的 URL 部分，并用客户端指定的值替换。

这种情况的另一个例子可以在 HTTP **DELETE**端点中看到。再次强调，当删除**待办事项**时，我们想要删除特定的项目，因此我们还需要传递一个 ID 来指定要删除的目标。让我们编写这段代码，我们将映射一个新的 HTTP **DELETE**方法到**/todoitems**路由。在路由上，我们将添加一个路由参数来传递要删除的**待办事项**的 ID：

```cs
app.MapDelete("/todoitems/{id}", (int id) =>
{
    var index = ToDoItems.FindIndex(x => x.Id == id);
    if (index == -1)
    {
        return Results.NotFound();
    }
    ToDoItems.RemoveAt(index);
    return Results.NoContent();
});
```

在收到对 **/todoitems** 路由的 **DELETE** 请求时，如果 URL 上附加了一个 **int**，它将被剥离并在请求中作为参数使用。参数数据类型的问题非常重要。在 **DELETE** 示例中，我们传递了一个 **int** 作为 ID 参数，因为这是在 **TodoItem** 类（我们的模型）的 ID 属性上使用的数据类型。

如果有人发送了不同的数据类型作为参数，比如一个字符串呢？我们当然需要处理这种情况，但不需要在代码中确保传入的 ID 是一个 **int**。确保请求只有在发送的参数是正确数据类型时才击中路由的更好方法是：**路由参数约束**。

在路由参数上添加约束会使传入的参数必须以特定方式形成，以便找到路由并接收请求。在我们的 **DELETE** 端点中，我们可以使用参数约束来规定参数必须是一个整数。

向参数添加约束非常简单。我们只需在参数后追加一个 **:** 字符，然后跟约束。让我们向我们的 **DELETE** 端点添加一个约束，以确保只有当 **id** 参数是 **int** 类型时才使用该路由：

```cs
app.MapDelete("/todoitems/{id:int}", (int id) =>
{
    var index = ToDoItems.FindIndex(x => x.Id == id);
    if (index == -1)
    {
        return Results.NotFound();
    }
    ToDoItems.RemoveAt(index);
    return Results.NoContent();
});
```

现在我们已经设置了约束，如果收到一个无法作为 **int** 处理的请求，API 将返回 **404 NOT FOUND** 响应。它这样做是因为约束阻止了 ASP.NET 尝试将参数用作 ID，因为它已经通过约束评估了数据类型。结果是找不到其他合适的路由。（除非在 **/todoitems** URL 上有一个可以接收字符串并且也是 HTTP **DELETE** 方法的路由。） 

参数约束不仅限于数据类型。参数可以通过字符串长度、数值范围、正则表达式模式等进行约束；列表还在继续。

让我们通过添加范围约束进一步约束 **DELETE** 范围。我们将使其只能删除前 100 个 ID。我们可以像这样将多个约束添加到单个路由中：

```cs
app.MapDelete("/todoitems/{id:int:range(1,100)}",
    (int id) =>
{
    var index = ToDoItems.FindIndex(x => x.Id == id);
    if (index == -1)
    {
        return Results.NotFound();
    }
    ToDoItems.RemoveAt(index);
    return Results.NoContent();
});
```

通过将另一个约束链接到现有的约束上，我们现在强制执行了 **Id** 参数必须是一个 **int**，并且其值必须在 **1** 到 **100** 之间。

*表 4.1* 展示了一些其他约束示例：

| **约束类型** | **路由示例** | **约束详情** |
| --- | --- | --- |
| 长度 | **/** **users/{username:length(3,20)}** | 用户名字符串长度必须在三个到二十个字符之间 |
| 长度 | **/** **users/{username:length(8)}** | 用户名字符串必须正好有八个字符长 |
| 最小长度 | **/** **users/{username:minlength(5)}** | 用户名字符串必须至少有五个字符长 |
| 最大长度 | **/** **users/{username:maxlength(30)}** | 用户名字符串长度不得超过三十个字符 |
| 正则表达式 | **/** **addNewCreditCard/{cardNumber:regex(³[47][0-9]{{13}}$)}** | 信用卡号码必须符合美国运通卡号的模式 |

表 4.1：最小 API 中参数约束的示例

现在我们更关注参数如何传递到我们的请求中，我们可以将注意力集中在请求体上，在其中我们主要处理请求。处理任何请求的主要部分是验证。最小 API，就像任何其他 API 一样，将在请求中接收数据，这些数据必须满足处理请求所需的条件。让我们看看我们可以用来管理请求流和处理可能因违反验证规则而出现的任何错误的验证技术。

# 请求验证和错误处理

我们有几种不同的验证方法可供选择。在本节中，我们将探讨其中的两种：**手动验证**和**数据注解**以及**模型绑定验证**。

## 手动验证

这种验证是最简单的，因为您是在路由处理程序（端点内的 lambda 表达式体）内部编写代码来验证请求并决定适当的响应。

我们已经在待办事项 API 的某些部分应用了手动验证。例如，我们创建的用于更新项目截止日期的 **PATCH** 方法首先检查具有目标 ID 的 **Todo** 项。它**可以**假设 **TodoItem** 已存在于列表中，但相反，我们首先检查它是否存在，如果存在，则返回 **404 NOT FOUND** 状态码：

```cs
app.MapPatch("/updateTodoItemDueDate/{id}",
    (int id, DateTime newDueDate) =>
{
    var index = ToDoItems.FindIndex(x => x.Id == id);
    if (index == -1)
    {
        return Results.NotFound();
    }
    ToDoItems[index].DueDate = newDueDate;
    return Results.NoContent();
});
```

通过添加这个手动检查，我们正在积极验证和处理异常情况。在 API 端点中进行验证不仅是一种最佳实践，而且是至关重要的。手动验证是验证的最基本形式。问题是它依赖于编写代码的开发者的直觉。这是一个有争议的话题，因为许多验证方法都有漏洞，但仅依赖手动验证可能会导致脆弱的 API。

为了减轻这种情况，我们还可以采用一个更标准化的验证框架，由 ASP.NET 提供的：使用数据注解的验证。

## 使用数据注解和模型绑定进行验证

如本章中我们构建的简单 API 示例所示，模型是最小 API 的重要方面。它们允许我们表示传入请求检索、移动和转换的实体。在待办事项 API 中，我们创建了一个 **TodoItem** 类作为模型，然后将实体存储在 **List<TodoItem>** 中。

可以通过请求绑定到特定模型的方式验证请求的数据。例如，在 **TodoItem** 模型中，当创建 **TodoItem** 时，合理地期望 **Title** 字段被填充。

我们可以通过注解字段来指定字段的要求。属性是有用的元数据片段，允许我们向代码应用约束。在这种情况下，属性的最常见用途之一是 **[** **Required]** 属性。

我们需要的属性是 **System.ComponentModel.DataAnnotations** 命名空间的一部分。

验证的必需命名空间

与 **Todo** 类一样，**System.ComponentModel.DataAnnotations** 也必须添加到 **Program.cs** 中以执行验证。

将此命名空间添加到 **TodoItem** 类的顶部，然后在 **Title** 字段上方添加 **[Required]** 属性：

```cs
using System.ComponentModel.DataAnnotations;
namespace TodoApi
{
    public class TodoItem
    {
        public int Id { get; set; }
        public DateTime StartDate { get; set; }
        public DateTime DueDate { get; set; }
        [Required]
        public string Title { get; set; }
        public string Description { get; set; }
        public string Assignee { get; set; }
        public int Priority { get; set; }
        public bool IsComplete { get; set; }
    }
}
```

仅添加一个 **[Required]** 属性本身不足以触发验证。我们仍然需要在我们的请求中调用验证。但是，我们可以在请求中调用一次，然后所有需要验证的项目都将根据我们应用的属性进行验证。以下是如何从我们之前创建的 **POST** 请求中调用我们的模型验证的方法：

```cs
app.MapPost("/todoitems", (TodoItem item) =>
{
    var validationResults = new List<ValidationResult>();
    var validationContext = new ValidationContext(item);
    bool isValid = Validator.TryValidateObject(
        item, validationContext, validationResults, true);
    if (!isValid)
    {
        return Results.BadRequest(validationResults);
    }
    ToDoItems.Add(item);
    return Results.Created();
});
```

在这个例子中，我们初始化了一个新的集合，一个 **ValidationResult** 的列表。这将包含有关验证成功或失败的信息。如果验证失败，我们将返回此集合给客户端。

我们还创建了一个新的 **ValidationContext**，传入要验证的项目。因为我们想验证作为有效负载发送的 **TodoItem** 实例，所以我们将其传递给 **ValidationContext**。

然后，我们可以通过调用 **Validator.TryValidateObject()** 来执行验证，传入项目作为验证目标，我们将要验证的上下文，以及将保存结果的集合，最后是一个布尔值 **true**，表示应验证所有属性。

现在，当发送一个请求，从有效负载中省略了 **Title** 字段时，以下错误 JSON 将自动生成并发送回客户端：

```cs
[
    {
        "memberNames": [
            "Title"
        ],
        "errorMessage": "The Title field is required."
    }
]
```

由于使用了属性和内置验证器，所有这些验证和错误处理都是自动发生的。

显示的错误信息是自动生成的，因为使用了 **[Required]** 属性。这可以通过向属性添加参数来覆盖。

这里是更新后的 **TodoItem** 模型代码，包含自定义的错误信息：

```cs
    public class TodoItem
    {
        public int Id { get; set; }
        public DateTime StartDate { get; set; }
        public DateTime DueDate { get; set; }
        [Required(ErrorMessage =
            "You need to add a title my dude!")]
        public string Title { get; set; }
        public string Description { get; set; }
        public string Assignee { get; set; }
        public int Priority { get; set; }
        public bool IsComplete { get; set; }
    }
```

现在，我们可以在生成的错误响应 JSON 中看到自定义的错误信息：

```cs
[
    {
        "memberNames": [
            "Title"
        ],
        "errorMessage": "You need to add a title my dude!"
    }
]
```

**[Required]** 只是数据注解提供的许多验证属性之一。您还可以添加许多其他约束。以下是一些这些约束的示例：

+   **[EmailAddress]** : 这确保值符合电子邮件地址的格式。

+   **[AllowedValues]** : 这强制使用特定的值。

+   **[DeniedValues]** : 这与 **[AllowedValues]** 相反，拒绝使用特定的值。

+   **[StringLength(x)]** : 这要求字符串值具有特定的长度。

+   **[CreditCard]** : 注释的值必须是有效的信用卡格式。

这些只是可以用来验证传入响应的许多属性中的几个，根据需要返回适当的错误。

管理 HTTP 方法和处理请求是最小 API 的关键方面，就像在任何 API 实现中一样。

### 使用过滤器进行验证

你还可以使用过滤器应用更具体的验证规则。**IEndpointFilter**是一个接口，可以用来在请求到达端点内的逻辑之前对传入的请求信息进行验证。

有一个方便的扩展方法，**AddEndPointFilter<T>**，它允许你将实现**IEndpointFilter**接口的类附加到端点上。

让我们通过我们的 todo API 上的**POST**端点来探索这个例子。我们将创建一个规则，其中 todo 项不能分配给任何名为 Joe Bloggs 的人：

1.  创建一个实现**IEndpointFilter**接口的类。这个类需要定义一个名为**Invoke**的函数，返回**ValueTask<object?>**。该函数接受**EndPointFilterInvocationContext**和**EndpointFilterDelegate**作为参数，以便执行验证逻辑：

    ```cs
    public class CreateTodoFilter : IEndpointFilter
    {
        public async ValueTask<object?> InvokeAsync(
            EndpointFilterInvocationContext context,
            EndpointFilterDelegate next)
        {
        }
    }
    ```

1.  **EndPointFilterInvocationContext**将包含传入的**TodoItem**，因为它代表了我们要验证的端点的上下文。在**InvokeAsync**内部，定义逻辑以从端点的上下文中访问传入的**TodoItem**，然后验证我们不是试图将其分配给 Joe Bloggs。如果是这样，返回适当的响应，使验证失败：

    ```cs
      var todoItem = context.GetArgument<TodoItem>(0);
      if(todoItem.Assignee == "Joe Bloggs")
      {
          return Results.Problem(
              "Joe Bloggs cannot be assigned a todoitem");
      }
    ```

1.  最后，对于验证通过的情况，我们希望将执行流程返回到原始端点（或附加到其上的任何其他链式逻辑，就像我们对中间件管道所做的那样）。通过返回对**EndpointFilterDelegate**的调用，我们作为参数接收，并传入端点上下文来完成此操作：

    ```cs
    return await next(context);
    ```

1.  最后，按照以下方式将过滤器验证添加到端点：

    ```cs
          app.MapPost("/todoitems", (TodoItem item) =>
          {
              ToDoItems.Add(item);
              return Results.Created();
          }).AddEndpointFilter<CreateTodoFilter>();
    ```

1.  或者，如果你想内联定义端点过滤器，可以在**AddEndpointFilter**之后传递一个匿名函数而不是类型：

    ```cs
        app.MapPost("/todoitems", (TodoItem item) =>
        {
            ToDoItems.Add(item);
            return Results.Created();
        }).AddEndpointFilter(async (context, next) =>
        {
            var toDoItem =
                context.GetArgument<TodoItem>(0);
            if (toDoItem.Assignee == "Joe Bloggs")
            {
                return Results.Problem(
                    "Joe Bloggs cannot be assigned todo
                    items");
            }
            return await next(context);
        });
    ```

现在你已经了解了我们可以用来为不同的 API 端点实现验证的各种方法，让我们回顾一下本章学到的内容。

# 摘要

在本章中，我们以高级视角概述了 HTTP 方法及其处理方式。我们进一步探讨了请求的路由方式，允许通过路由参数从路由定义中提取基本参数。

我们还深入探讨了验证的基础知识，首先是通过在我们的 API 路由上放置约束来确保接收到的数据格式正确。随后，我们学习了如何以不同的方式处理验证，包括手动验证，以及在模型中使用数据注释在端点体内部集中调用验证。

在本章中，我们看到了如何通过验证技术捕捉错误，以及如何将具有信息量的错误响应反馈给客户端，以指导他们的调试策略。

到现在为止，你应该能够构建一个基本但功能齐全的最小 API 项目。在下一章中，我们将学习如何以中间件的形式向我们的应用程序管道中引入自定义功能。
