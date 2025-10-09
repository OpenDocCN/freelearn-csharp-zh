

# 第七章：创建 API

当使用 WebAssembly 运行 Blazor（InteractiveWebAssembly 或 InteractiveAuto）时，我们需要能够检索数据并更改我们的数据。为了实现这一点，我们需要一个 API 来访问数据。在本章中，我们将使用 **Minimal API** 创建一个 Web API。

当使用 Blazor Server 时，API 将由页面（如果我们添加 **Authorize** 属性）进行保护，因此我们免费获得这个功能。但是，与 WebAssembly 一起使用时，所有操作都将执行在浏览器中，因此我们需要 WebAssembly 可以与之通信以更新服务器上的数据。

要做到这一点，我们需要涵盖以下主题：

+   创建服务

+   创建客户端

# 技术要求

确保您已经阅读了前面的章节或使用 `Chapter06` 文件夹作为起点。

您可以在此处找到本章最终结果的源代码：[`github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter07`](https://github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter07)。

# 创建服务

创建服务有多种方式，例如通过 REST。

对于那些之前没有使用过 REST 的人来说，**REST** 代表 **representational state transfer**。简单来说，它是一种机器使用 HTTP 与其他设备通信的方式。

在 REST 中，我们使用不同的 HTTP 动词来执行不同的操作。它们可能看起来像这样：

| **URI** | **Verb** | **Action** |
| --- | --- | --- |
| `/BlogPosts` | Get | 获取博客文章列表 |
| `/BlogPosts` | Post | 创建新的博客文章 |
| `/BlogPosts/{id}` | Get | 获取具有特定 ID 的博客文章 |
| `/BlogPosts/{id}` | Put | 替换博客文章 |
| `/BlogPosts/{id}` | Patch | 更新博客文章 |
| `/BlogPosts/{id}` | Delete | 删除博客文章 |

表 7.1：REST 调用

我们将为 **标签**、**类别**和 **博客文章**实现 API。

由于 API 负责处理 `Post` 是否应该被创建，我们将作弊并仅实现 `Put`（替换），因为我们不知道我们是创建还是更新数据。

我们将在 **BlazorWebApp** 项目中实现 API。

## 了解 Minimal API

在我们深入实现 Minimal API 之前，让我们花点时间了解一下它。回到 2019 年 11 月，**分布式应用运行时**（**Dapr**）团队的一名成员编写了几个教程，介绍了如何使用不同的语言构建分布式计算器。

他们提供了使用 Go、Python、Node.js 和 .NET Core 的示例。代码展示了在 C# 中编写分布式计算器比在其他语言中要困难得多。

微软询问了各种非 .NET 开发者他们对 C# 的看法。他们的回应并不理想。然后，微软让他们使用 Minimal APIs 的早期版本完成一个教程。

在教程之后，他们被询问他们的看法，他们的回应发生了变化，现在更加积极；感觉就像在家一样。

最小 API 的目标是减少复杂性和仪式，拥抱简约。我以为“最小”意味着我无法做所有事情，但深入代码后，我很快意识到并非如此。

从我的角度来看，最小 API 是编写 API 的更好方式。其理念是，如果我们需要，我们可以扩展我们的 API，一旦我们觉得合适，我们可以将代码移动到控制器中以获得更多结构。在我的工作场所，我们转向了最小 API，因为我们认为语法更简洁。

添加最小 API 的一个非常简单的示例就是在 `Program.cs` 中添加这一行：

```cs
app.MapGet("/api/helloworld", () => "Hello world!"); 
```

如果我们导航到一个没有指定任何路由的 URL，只是 `"/"`，我们将返回一个包含 `"Hello World"` 的字符串。

当然，这是一个可能的简单示例，但也可以实现更复杂的事情，正如我们将在下一节中看到的那样。

## 添加 API 控制器

我们有三个数据模型：博客文章、标签和分类。

让我们创建三个不同的文件，每个数据模型一个，以展示使用最小 API 添加更复杂 API 的友好方式。对于一个小项目，可能更合理的是在 `Program.cs` 中添加所有内容。

### 添加处理博客文章的 API

让我们先添加处理博客文章的 API 方法。

执行以下步骤以创建 API：

1.  在 `BlazorWebApp` 项目中，添加一个名为 `Endpoints` 的新文件夹。

1.  在 `Endpoints` 文件夹中，创建一个名为 `BlogPostEndpoints.cs` 的类。目的是创建一个扩展方法，我们可以在 `Program.cs` 中稍后使用。

    在文件顶部添加这些 `using` 语句：

    ```cs
    using Data.Models;
    using Data.Models.Interfaces;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc; 
    ```

1.  将类替换为以下代码：

    ```cs
    public static class BlogPostEndpoints
    {
        public static void MapBlogPostApi(this WebApplication app)
        {
            app.MapGet("/api/BlogPosts",
            async (IBlogApi api, [FromQuery] int numberofposts, [FromQuery] int startindex) =>
            {
                return Results.Ok(await api.GetBlogPostsAsync(numberofposts, startindex));
            });
           }
    } 
    ```

    由于我们正在创建一个扩展方法，我们必须确保类是静态的。`MapBlogPostApi` 方法使用 `this` 关键字，这使得该方法在任何 `WebApplication` 类上都可用。

    我们通过使用 `MapGet` 和一个路径来设置最小 API，这意味着如果使用 `Get` 动词通过正确的参数访问该路径，该方法将运行。

    该方法接受几个参数。第一个是 `IBlogApi` 类型，它将使用依赖注入来获取我们需要的类的实例，在这种情况下，是 `BlogApiJsonDirectAccess`，它将访问我们存储的 JSON 文件。

    其他参数将使用查询字符串（因为我们使用的是 `query` 属性）；在大多数情况下，最小 API 会自动处理这些事情，但引导它走向正确的方向从不会错。

    我们创建了一个直接从数据库返回数据的方法（与 Blazor 服务器项目使用的相同 API）。

    我们还需要确保从 `Program.cs` 中调用它。

1.  在 `Program.cs` 中添加以下命名空间：

    ```cs
    using BlazorWebApp.Endpoints; 
    ```

1.  还在 `app.Run();` 之上添加以下代码：

    ```cs
    app.MapBlogPostApi(); 
    ```

1.  是时候测试 API 了；确保启动 `BlazorWebApp` 项目。在 .NET 6 中，端口号是随机的，所以将 `{REPLACEWITHYOURPORTNUMBER}` 替换为你的项目端口号。

    请访问以下网址：`https://localhost:{REPLACEWITHYOURPORTNUMBER}/Api/BlogPosts?numberofposts=10&startindex=0`（端口号可能不同）。我们将获取一些 JSON 返回，其中包含我们的博客文章列表。

    我们已经取得了良好的开端！现在，我们需要实现 API 的其余部分。

1.  在`Endpoints/BlogPostEndpoint.cs`文件中，在`MapBlogPostApi`方法中，让我们添加获取博客文章计数的代码：

    ```cs
    app.MapGet("/api/BlogPostCount",
    async (IBlogApi api) =>
    {
        return Results.Ok(await api.GetBlogPostCountAsync());
    }); 
    ```

    我们使用`Get`动词，但使用另一个路由。

1.  我们还需要能够获取一篇博客文章。添加以下代码：

    ```cs
    app.MapGet("/api/BlogPosts/{*id}",
    async (IBlogApi api, string id) =>
    {
        return Results.Ok(await api.GetBlogPostAsync(id));
    }); 
    ```

    在这种情况下，我们使用`Get`动词，但使用另一个包含我们想要获取的`Post` ID 的 URL。

    我们使用一个字符串作为 ID，例如，像 RavenDB 这样的数据库使用类似这样的 ID：`CollectionName/IdOfThePost`；我们还确保在参数中添加`*`。这样，它将使用任何跟在后面的内容作为 ID，否则它将把斜杠解释为路由的一部分，而找不到端点。

    接下来，我们需要一个受保护的 API，通常是更新或删除内容的 API。

1.  让我们添加一个保存博客文章的 API。在刚刚添加的代码下方添加以下代码：

    ```cs
    app.MapPut("/api/BlogPosts",
    async (IBlogApi api, [FromBody] BlogPost item) =>
    {
        return Results.Ok(await api.SaveBlogPostAsync(item));
    }).RequireAuthorization(); 
    ```

    如我在本章前面提到的，我们只会添加一个用于创建和更新博客文章的 API，并且我们将使用`Put`动词（替换）来完成这个操作。我们在最后添加了`RequireAuthorization`方法，这将确保用户需要经过身份验证才能调用该方法。

1.  接下来，我们添加删除博客文章的代码。为此，添加以下代码：

    ```cs
    app.MapDelete("/api/BlogPosts/{*id}",
    async (IBlogApi api, string id) =>
    {
        await api.DeleteBlogPostAsync(id);
        return Results.Ok();
    }).RequireAuthorization(); 
    ```

在这种情况下，我们使用`Delete`动词，就像保存一样，我们在最后添加了`RequireAuthorization`方法。

接下来，我们还需要对`Categories`和`Tags`进行同样的操作。

### 添加处理类别的 API

让我们从`Categories`开始。按照以下步骤操作：

1.  在`Endpoints`文件夹中，添加一个名为`CategoryEndpoints.cs`的新类。用以下代码替换代码：

    ```cs
    using Data.Models;
    using Data.Models.Interfaces;
    using Microsoft.AspNetCore.Mvc;
    namespace BlazorWebApp.Endpoints;
    public static class CategoryEndpoints
    {
        public static void MapCategoryApi(this WebApplication app)
        {
            app.MapGet("/api/Categories",
            async (IBlogApi api) =>
            {
                return Results.Ok(await api.GetCategoriesAsync());
            });
            app.MapGet("/api/Categories/{*id}",
            async (IBlogApi api, string id) =>
            {
                return Results.Ok(await api.GetCategoryAsync(id));
            });
            app.MapPut("/api/Categories",
            async (IBlogApi api, [FromBody] Category item) =>
            {
                return Results.Ok(await api.SaveCategoryAsync(item));
            }).RequireAuthorization();
            app.MapDelete("/api/Categories/{*id}",
            async (IBlogApi api, string id) =>
            {
                await api.DeleteCategoryAsync(id);
                return Results.Ok();
            }).RequireAuthorization();
        }
    } 
    ```

1.  在`Program.cs`中，在`app.Run()`上方添加以下代码：

    ```cs
    app.MapCategoryApi(); 
    ```

这些都是处理`Categories`所需的所有方法。

接下来，让我们用`Tags`做同样的事情。

### 添加处理标签的 API

让我们按照以下步骤为标签做同样的事情：

1.  在`Endpoints`文件夹中，添加一个名为`TagEndpoints.cs`的新类。添加以下代码：

    ```cs
    using Data.Models;
    using Data.Models.Interfaces;
    using Microsoft.AspNetCore.Mvc;
    namespace BlazorWebApp.Endpoints;
    public static class TagEndpoints
    {
        public static void MapTagApi(this WebApplication app)
        {
            app.MapGet("/api/Tags",
            async (IBlogApi api) =>
            {
                return Results.Ok(await api.GetTagsAsync());
            });
            app.MapGet("/api/Tags/{*id}",
            async (IBlogApi api, string id) =>
            {
                return Results.Ok(await api.GetTagAsync(id));
            });
            app.MapPut("/api/Tags",
            async (IBlogApi api, [FromBody] Tag item) =>
            {
                return Results.Ok(await api.SaveTagAsync(item));
            }).RequireAuthorization();          app.MapDelete("/api/Tags/{*id}",
            async (IBlogApi api, string id) =>
            {
                await api.DeleteTagAsync(id);
                return Results.Ok();
            }).RequireAuthorization();
        }
    } 
    ```

1.  在`Program.cs`中，在`app.Run()`上方添加以下代码：

    ```cs
    app.MapTagApi(); 
    ```

但是等等！关于评论怎么办？我们实现评论的方式意味着该组件永远不会作为 WebAssembly 运行，所以我们实际上不需要在 API 中实现它。但我们不会让评论悬而未决——让我们也实现这些评论！

### 添加处理评论的 API

让我们按照以下步骤为评论做同样的事情：

1.  在`Endpoints`文件夹中，添加一个名为`CommentEndpoints.cs`的新类。添加以下代码：

    ```cs
    using Data.Models;
    using Data.Models.Interfaces;
    using Microsoft.AspNetCore.Mvc;
    namespace BlazorWebApp.Endpoints;
    public static class CommentEndpoints
    {
        public static void MapCommentApi(this WebApplication app)
        {
            app.MapGet("/api/Comments/{*blogPostid}",
            async (IBlogApi api, string blogPostid) =>
            {
                return Results.Ok(await api.GetCommentsAsync(blogPostid));
            });
            }).RequireAuthorization();
            app.MapPut("/api/Comments",
            async (IBlogApi api, [FromBody] Comment item) =>
            {
                return Results.Ok(await api.SaveCommentAsync(item));
            }).RequireAuthorization();
            app.MapDelete("/api/Comments/{*id}",
            async (IBlogApi api, string id) =>
            {
                await api.DeleteCommentAsync(id);
                return Results.Ok();
        }
    } 
    ```

1.  在`Program.cs`中，在`app.Run()`上方添加以下代码：

    ```cs
    app.MapCommentApi(); 
    ```

太好了！我们有了 API！现在，是时候创建一个客户端来访问这个 API 了。

# 创建客户端

为了访问 API，我们需要创建一个客户端。有多种方法可以做到这一点，但我们将以最简单的方式通过编写代码来实现。

客户端将实现相同的`IBlogApi`接口。这样，无论我们使用哪种实现，代码都是相同的，并且可以直接通过`BlogApiJsonDirectAccess`或`BlogApiWebClient`（我们接下来将要创建的）进行 JSON 访问：

1.  在`BlazorWebApp.Client`下的**Dependencies**节点上右键点击，并选择**Manage NuGet Packages**。

1.  搜索`Microsoft.AspNetCore.Components.WebAssembly.Authentication`并点击**Install**。

1.  此外，搜索`Microsoft.Extensions.Http`并点击**Install**。

1.  在`BlazorWebApp.Client`项目中，在项目根目录下添加一个新的类，并将其命名为`BlogApiWebClient.cs`。

1.  打开新创建的文件并添加以下命名空间：

    ```cs
    using Data.Models;
    using Data.Models.Interfaces;
    using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
    using System.Net.Http.Json;
    using System.Text.Json; 
    ```

1.  将`IBlogApi`添加到类中，并使其公开，如下所示：

    ```cs
    namespace BlazorWebApp.Client;
    public class BlogApiWebClient : IBlogApi
    {
    } 
    ```

1.  一些 API 调用将是公开的（不需要认证），但`HttpClient`将被配置为需要令牌。

    令牌的处理由 Blazor 自动处理，所以我们只需要一个客户端，在这个案例中，我们将其称为`Api`。

    为了能够调用 API，我们需要注入`HttpClient`。将以下代码添加到类中：

    ```cs
     private readonly IHttpClientFactory _factory;
        public BlogApiWebClient(IHttpClientFactory factory)
        {
            _factory = factory;
        } 
    ```

1.  现在，是时候实现对 API 的调用了。让我们从博客文章的**Get**调用开始。添加以下代码：

    ```cs
    public async Task<BlogPost?> GetBlogPostAsync(string id)
        {
            var httpclient = _factory.CreateClient("Api");
            return await httpclient.GetFromJsonAsync<BlogPost>($"api/BlogPosts/{id}");
        }
        public async Task<int> GetBlogPostCountAsync()
        {
            var httpclient = _factory.CreateClient("Api");
            return await httpclient.GetFromJsonAsync<int>("/api/BlogPostCount");
        }
        public async Task<List<BlogPost>?> GetBlogPostsAsync(int numberofposts, int startindex)
        {
            var httpclient = _factory.CreateClient("Api");
            return await httpclient.GetFromJsonAsync<List<BlogPost>>($"/api/BlogPosts?numberofposts={numberofposts}&startindex={startindex}");
        } 
    ```

    我们使用注入的`HttpClient`，然后调用`GetFromJsonAsync`，这将自动下载 JSON 并将其转换为提供给泛型方法的类。

    现在，事情变得有点复杂：我们需要处理认证。幸运的是，这已经内置在`HttpClient`中，所以我们只需要处理`AccessTokenNotAvailableException`。如果令牌缺失，它将自动尝试更新它，但如果出现问题（例如，用户未登录），我们可以重定向到登录页面。

    我们将在**第八章**，**认证和授权**中回到令牌和认证的工作方式。

1.  接下来，我们添加需要认证的 API 调用，例如保存或删除博客文章。

    在我们刚刚添加的代码下方添加以下代码：

    ```cs
    public async Task<BlogPost?> SaveBlogPostAsync(BlogPost item)
    {
        try
        {
            var httpclient = _factory.CreateClient("Api");
            var response = await httpclient.PutAsJsonAsync<BlogPost>
               ("api/BlogPosts", item);
            var json = await response.Content.ReadAsStringAsync();
            return JsonSerializer.Deserialize<BlogPost>(json);
        }
        catch (AccessTokenNotAvailableException exception)
        {
            exception.Redirect();
        }
        return null;
    }
    public async Task DeleteBlogPostAsync(string id)
    {
        try
        {
            var httpclient = _factory.CreateClient("Api");
            await httpclient.DeleteAsync($"api/BlogPosts/{id}");
        }
        catch (AccessTokenNotAvailableException exception)
        {
            exception.Redirect();
        }
    } 
    ```

    如果调用抛出`AccessTokenNotAvailableException`，这意味着`HttpClient`无法自动获取或更新令牌，用户需要登录。

    这种状态可能永远不会发生，因为我们将会确保当用户导航到那个页面时，他们需要登录，但防患于未然总是好的。

1.  现在，我们需要为`Categories`做同样的事情。将以下代码添加到`BlogApiWebClient`类中：

    ```cs
    public async Task<List<Category>?> GetCategoriesAsync()
    {
        var httpclient = _factory.CreateClient("Api");
        return await httpclient.GetFromJsonAsync<List<Category>>($"api/Categories");
    }
    public async Task<Category?> GetCategoryAsync(string id)
    {
        var httpclient = _factory.CreateClient("Api");
        return await httpclient.GetFromJsonAsync<Category>($"api/Categories/{id}");
    }
    public async Task DeleteCategoryAsync(string id)
    {
        try
        {
            var httpclient = _factory.CreateClient("Api");
            await httpclient.DeleteAsync($"api/Categories/{id}");
        }
        catch (AccessTokenNotAvailableException exception)
        {
            exception.Redirect();
        }
    }
    public async Task<Category?> SaveCategoryAsync(Category item)
    {
        try
        {
            var httpclient = _factory.CreateClient("Api");
            var response = await httpclient.PutAsJsonAsync<Category>("api/Categories", item);
            var json = await response.Content.ReadAsStringAsync();
            return JsonSerializer.Deserialize<Category>(json);
        }
        catch (AccessTokenNotAvailableException exception)
        {
            exception.Redirect();
        }
        return null;
    } 
    ```

1.  接下来，我们将为`Tags`做同样的事情。将以下代码添加到我们刚刚添加的代码下方：

    ```cs
    public async Task<Tag?> GetTagAsync(string id)
    {
        var httpclient = _factory.CreateClient("Api");
        return await httpclient.GetFromJsonAsync<Tag>($"api/Tags/{id}");
    }
    public async Task<List<Tag>?> GetTagsAsync()
    {
        var httpclient = _factory.CreateClient("Api");
        return await httpclient.GetFromJsonAsync<List<Tag>>($"api/Tags");
    }
    public async Task DeleteTagAsync(string id)
    {
        try
        {
            var httpclient = _factory.CreateClient("Api");
            await httpclient.DeleteAsync($"api/Tags/{id}");
        }
        catch (AccessTokenNotAvailableException exception)
        {
            exception.Redirect();
        }
    }
    public async Task<Tag?> SaveTagAsync(Tag item)
    {
        try
        {
            var httpclient = _factory.CreateClient("Api");
            var response = await httpclient.PutAsJsonAsync<Tag>("api/Tags", item);
            var json = await response.Content.ReadAsStringAsync();
            return JsonSerializer.Deserialize<Tag>(json);
        }
        catch (AccessTokenNotAvailableException exception)
        {
            exception.Redirect();
        }
        return null;
    } 
    ```

1.  让我们不要忘记我们的评论！将以下代码添加到我们刚刚添加的代码下方：

    ```cs
    public async Task<List<Comment>> GetCommentsAsync(string blogpostid)
        {
            var httpclient = _factory.CreateClient("Api");
            return await httpclient.GetFromJsonAsync<List<Comment>>($"api/Comments/{blogpostid}");
        }

        public async Task DeleteCommentAsync(string id)
        {
            try
            {
                var httpclient = _factory.CreateClient("Api");
                await httpclient.DeleteAsync($"api/Comments/{id}");
            }
            catch (AccessTokenNotAvailableException exception)
            {
                exception.Redirect();
            }
        }
        public async Task<Comment?> SaveCommentAsync(Comment item)
        {
            try
            {
                var httpclient = _factory.CreateClient("Api");
                var response = await httpclient.PutAsJsonAsync<Comment>("api/Comments", item);
                var json = await response.Content.ReadAsStringAsync();
                return JsonSerializer.Deserialize<Comment>(json);
            }
            catch (AccessTokenNotAvailableException exception)
            {
                exception.Redirect();
            }
            return null;
        } 
    ```

干得好！我们的 API 客户端现在已经完成了！

# 摘要

在本章中，我们学习了如何使用 Minimal APIs 和 API 客户端创建一个 API，这是大多数应用程序的重要组成部分。这样，我们就可以从数据库中获取博客文章，并在运行在 WebAssembly 时展示它们。值得一提的是，我们始终可以使用 Web API 来运行我们的应用程序；这只是为了展示我们可以根据当前使用的托管模型使用不同的方式来访问我们的数据。

在下一章中，我们将向我们的网站添加登录功能，并首次调用我们的 API。
