

# 第三章：管理状态 – 第一部分

在本章中，我们将开始探讨状态管理。本章的续篇是*第十一章*，*管理状态 – 第二部分*。

管理状态或持久化数据的方法有很多。一旦我们离开组件，状态就会消失。如果我们从示例页面点击计数器按钮，看到计数器计数增加然后导航离开，我们就不知道需要点击计数器按钮多少次，并不得不从头开始。您无法想象我多年来点击那个计数器按钮的次数。这是一个简单而强大的 Blazor 演示，也是 Steve 在 2017 年原始演示的一部分。

为了快速入门，我已经将本章分为两部分。在本章中，我们将专注于数据访问，而在第二部分中，我们将回到更多状态管理。由于本书专注于 Blazor，因此我们不会探索如何连接到数据库，而是创建简单的 JSON 存储。

在第一版中，我们使用 Entity Framework 连接到数据库，但有些人不习惯使用 Entity Framework，他们很快就陷入了困境。使用 Entity Framework 本身就是一本书，所以我选择不将其包含在本书中，以消除任何额外的复杂性。

在 GitHub 上的 repo 中，您可以找到更多将数据存储在数据库中的示例，例如`RavenDB`或`MSSQL`。

我们将使用一个称为**仓储模式**的通用模式。

我们还将创建一个 API 来访问 JSON 存储库中的数据。

到本章结束时，您将学会如何创建 JSON 存储库和 API。

我们将涵盖以下主要主题：

+   创建数据项目

+   将 API 添加到 Blazor

# 技术要求

确保您已经遵循了前面的章节，或者使用 GitHub 上的`Chapter02`文件夹作为起点。

您可以在[`github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter03`](https://github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter03)找到本章结果的源代码。

# 创建数据项目

存储数据的方式有很多：文档数据库、关系数据库和文件，仅举几例。为了避免本书的复杂性，我们将使用最简单的方式为我们的项目创建博客文章，将它们存储在文件夹中的 JSON 中。

要保存我们的博客文章，我们将使用存储在文件夹中的 JSON 文件，为此，我们需要创建一个新的项目。

## 创建新项目

我们可以从 Visual Studio 内部创建一个新的项目（说实话，我会这样做），但为了了解.NET CLI，让我们从命令行操作。

要创建一个新的项目，请按照以下步骤操作：

1.  打开 PowerShell 提示符。

1.  导航到`MyBlog`文件夹。

1.  通过输入以下命令创建一个类库（`classlib`）：

    ```cs
    dotnet new classlib -o Data 
    ```

    `dotnet`工具现在应该已经创建了一个名为`Data`的文件夹。

1.  我们还需要创建一个项目，以便我们可以放置我们的模型：

    ```cs
    dotnet new classlib -o Data.Models 
    ```

1.  通过运行以下命令将新项目添加到我们的解决方案中：

    ```cs
    dotnet sln add Data
    dotnet sln add Data.Models 
    ```

它将在当前文件夹中查找任何解决方案。

我们将项目命名为 `Data` 和 `Data.Models`，这样它们的用途将很容易理解，并且很容易找到。

默认项目包含一个 `class1.cs` 文件 - 随意删除该文件。

下一步是创建数据类来存储我们的信息。

## 创建数据类

现在我们需要为我们的博客文章创建一个类。为此，我们将回到 Visual Studio：

1.  在 Visual Studio 中打开 `MyBlog` 解决方案（如果尚未打开）。

    我们现在应该在解决方案中有一个名为 `Data` 的新项目。我们可能会收到一个弹出窗口询问我们是否想要重新加载解决方案；如果是，请点击 **重新加载**。

1.  右键单击 `Data.Models` 项目，然后选择 **添加** | **类**。将类命名为 `BlogPost.cs` 并点击 **添加**。

1.  右键单击 `Data.Models` 项目，然后选择 **添加** | **类**。将类命名为 `Category.cs` 并点击 **添加**。

1.  右键单击 `Data.Models` 项目，然后选择 **添加** | **类**。将类命名为 `Tag.cs` 并点击 **添加**。

1.  右键单击 `Data.Models` 项目，然后选择 **添加** | **类**。将类命名为 `Comment.cs` 并点击 **添加**。

1.  打开 `Category.cs` 并将内容替换为以下代码：

    ```cs
    namespace Data.Models;
    public class Category
    {
    	public string? Id { get; set; }
    	public string Name { get; set; } = string.Empty;
    } 
    ```

    `Category` 类包含 `Id` 和 `Name` 属性。`Id` 属性是字符串类型可能看起来有些奇怪，但这是因为我们将支持多种数据存储类型，包括 MSSQL、RavenDB 和 JSON。字符串是一个支持所有这些类型的优秀数据类型。`Id` 属性也可以为空，因此如果我们创建一个新的类别，我们将以空值作为 `Id` 发送。

1.  打开 `Tag.cs` 并将内容替换为以下代码：

    ```cs
    namespace Data.Models;
    public class Tag
    {
        public string? Id { get; set; }
        public string Name { get; set; } = string.Empty;
    } 
    ```

    `Tag` 类包含一个 `Id` 和 `Name`。

1.  打开 `Comment.cs` 并将内容替换为以下代码：

    ```cs
    namespace Data.Models;
    public class Comment
    {
        public string? Id { get; set; }
        public required string BlogPostId { get; set; }
        public DateTime Date { get; set; }
        public string Text { get; set; } = string.Empty;
        public string Name { get; set; } = string.Empty;
    } 
    ```

1.  `comment` 类可以是 `Blogpost` 类的一部分，但为了使用相同的类对不同数据库类型进行操作，我们将评论作为一个单独的实体引用博客文章。

1.  打开 `BlogPost.cs` 并将内容替换为以下代码：

    ```cs
    namespace Data.Models;
    public class BlogPost
    {
        public string? Id { get; set; }
        public string Title { get; set; } = string.Empty;
        public string Text { get; set; } = string.Empty;
        public DateTime PublishDate { get; set; }
        public Category? Category { get; set; }
        public List<Tag> Tags { get; set; } = new();
    } 
    ```

在这个类中，我们定义了博客文章的内容。我们需要一个 `Id` 来标识博客文章，一个标题，一些文本（文章），以及发布日期。我们还在类中有一个 `Category` 属性，它是 `Category` 类型。在这种情况下，一篇博客文章只能有一个类别，一篇博客文章可以包含零个或多个标签。我们使用 `List<Tag>` 定义 `Tag` 属性。

我们现在已经创建了一些我们将使用的类。我已将这些类的复杂性保持在最低，因为我们在这里是为了学习 Blazor。

接下来，我们将创建一种存储和检索博客文章信息的方法。

## 创建接口

在本节中，我们将创建一个 API。

我们将创建一个具有直接数据库访问的 API 和一个将通过 Web API 获取数据的 API。

在 *第七章*，*创建 API* 中，我们将回到创建 Web API。为什么我们要创建两个 API？

我们不是创建两个 API；我们正在创建一个具有直接数据库访问的服务和一个通过网络进行操作然后使用直接数据库访问的客户端。但我们将使用相同的接口来处理这两种情况，使得在服务器上使用一个，在客户端使用另一个成为可能。

在实际应用中，以统一的方式访问所有数据会更合理，而不是使用两种方式。但重点是展示混合匹配并选择适合你场景的正确方式。

我们将从具有直接数据库访问的 API 开始：

1.  右键点击`Data.Models`项目，选择**添加** | **新建文件夹**，并将其命名为`Interfaces`。

1.  右键点击`Interfaces`文件夹，选择**添加** | **类**。

1.  在不同的模板列表中，选择**接口**，并将其命名为`IBlogApi.cs`。

1.  打开`IBlogApi.cs`，并将其内容替换为以下内容：

    ```cs
    namespace Data.Models.Interfaces;
    public interface IBlogApi
    {
        Task<int> GetBlogPostCountAsync();
        Task<List<BlogPost>> GetBlogPostsAsync(int numberofposts, int startindex);
        Task<List<Category>> GetCategoriesAsync();
        Task<List<Tag>> GetTagsAsync();
        Task<List<Comment>> GetCommentsAsync(string blogPostId);
        Task<BlogPost?> GetBlogPostAsync(string id);
        Task<Category?> GetCategoryAsync(string id);
        Task<Tag?> GetTagAsync(string id);
        Task<BlogPost?> SaveBlogPostAsync(BlogPost item);
        Task<Category?> SaveCategoryAsync(Category item);
        Task<Tag?> SaveTagAsync(Tag item);
        Task<Comment?> SaveCommentAsync(Comment item);
        Task DeleteBlogPostAsync(string id);
        Task DeleteCategoryAsync(string id);
        Task DeleteTagAsync(string id);
        Task DeleteCommentAsync(string id);
    } 
    ```

1.  好吧，关于这个`IBlogApi`的事情是这样的。它基本上是我们处理所有博客内容的速查表，比如帖子、评论、标签和分类。需要获取一些帖子或者将其从存在中删除？这个接口就是你的首选。它主要是为了让我们在编写博客时生活更轻松，保持事情整洁和直接。

现在，我们有一个 API 接口，其中包含了我们需要列出博客帖子、标签和分类，以及保存（创建/更新）和删除它们的方法。接下来，让我们实现这个接口。

## 实现接口

想法是创建一个类，将我们的博客帖子、标签、评论和分类作为 JSON 文件存储在我们的文件系统中。我们将从实现直接访问实现开始。当我们直接从数据库访问信息而不是通过 Web API 时，我们可以使用这个直接访问实现。当我们在服务器上运行我们的组件并访问数据库时，我们的 Web API 也将使用它来访问数据库，但我们将回到第七章，创建 API。

要实现直接数据库访问实现的接口，请按照以下步骤操作：

1.  首先，为了能够访问我们的数据模型，我们需要在我们的`Data`模型中添加一个引用。展开`Data`项目，右键点击**依赖项**节点。选择**添加项目引用**，并勾选`Data.Models`项目。点击**确定**。

1.  再次右键点击**依赖项**节点，但选择**管理 NuGet 包**。在**浏览**选项卡中，搜索`Microsoft.Extensions.Options`并点击**安装**。

1.  我们需要一个类来保存我们的设置。

    在`Data`项目中，添加一个名为`BlogApiJsonDirectAccessSetting.cs`的新类，并将其内容替换为：

    ```cs
    namespace Data;
    public class BlogApiJsonDirectAccessSetting
    {
        public string BlogPostsFolder { get; set; } = string.Empty;
        public string CategoriesFolder { get; set; } = string.Empty;
        public string TagsFolder { get; set; } = string.Empty;
        public string CommentsFolder { get; set; } = string.Empty;
        public string DataPath { get; set; } = string.Empty;
    } 
    ```

1.  这是一个保存我们的设置以及我们将用于存储 JSON 文件的文件夹的类。`IOptions`在配置依赖项期间在`program`中配置，并注入到所有请求特定类型的类中。

1.  接下来，我们需要为我们的 API 创建一个类。右键单击 `Data` 项目，选择 **添加** | **类**，并将类命名为 `BlogApiJsonDirectAccess.cs`。

1.  打开 `BlogApiJsonDirectAccess.cs` 文件，并将代码替换为以下内容：

    ```cs
    using Data.Models.Interfaces;
    using Microsoft.Extensions.Options;
    using System.Text.Json;
    using Data.Models;
    namespace Data;
    public class BlogApiJsonDirectAccess: IBlogApi
    {
    } 
    ```

    这是我们的 JSON 直接访问类的开始。它引用了 `IBlogAPI`，我们将实现接口想要的所有方法。

    错误列表应该包含许多错误，因为我们还没有实现这些方法。我们正在继承自 `IBlogApi`，因此我们知道要公开哪些方法。

1.  为了能够读取设置，我们也添加了一种注入 `IOptions` 的方式。通过这种方式获取设置，我们不需要添加任何代码——它可以从数据库、设置文件，甚至可以硬编码。

    这是我最喜欢的获取设置的方式，因为代码的这一部分本身并不知道如何实现——相反，我们通过依赖注入添加所有配置。

    将以下代码添加到 `BlogApiJsonDirectAccess` 类中：

    ```cs
     BlogApiJsonDirectAccessSetting _settings;
        public BlogApiJsonDirectAccess( IOptions<BlogApiJsonDirectAccessSetting> option)
        {
            _settings = option.Value;
            ManageDataPaths();
        }
        private void ManageDataPaths()
        {
            CreateDirectoryIfNotExists(_settings.DataPath);
            CreateDirectoryIfNotExists($@"{_settings.DataPath}\{_settings.BlogPostsFolder}");
            CreateDirectoryIfNotExists($@"{_settings.DataPath}\{_settings.CategoriesFolder}");
            CreateDirectoryIfNotExists($@"{_settings.DataPath}\{_settings.TagsFolder}");
            CreateDirectoryIfNotExists($@"{_settings.DataPath}\{_settings.CommentsFolder}");
        }
        private static void CreateDirectoryIfNotExists(string path)
        {
            if (!Directory.Exists(path))
            {
                Directory.CreateDirectory(path);
            }
        } 
    ```

    我们获取注入的设置并确保我们有正确的数据文件夹结构。

1.  现在，是时候实现 API 了，但首先，我们需要一些辅助方法来从我们的文件系统中加载数据。为此，我们将以下代码添加到我们的类中：

    ```cs
    private async Task<List<T>> LoadAsync<T>(string folder)
    {
        var list = new List<T>();
        foreach (var f in Directory.GetFiles($@"{_settings.DataPath}\{folder}"))
        {
            var json = await File.ReadAllTextAsync(f);
            var blogPost = JsonSerializer.Deserialize<T>(json);
            if (blogPost is not null)
            {
                list.Add(blogPost);
            }
        }
        return list;
    } 
    ```

1.  `LoadAsync` 方法是一个泛型方法，允许我们使用相同的方法加载博客文章、标签、评论和类别。它将在我们请求时从文件系统中加载数据。这是一个放置一些缓存逻辑的好地方，但如果我们使用数据库（我们总是会询问数据库），实现将更接近这个样子。

1.  接下来，我们将添加几个帮助方法来操作数据，即 `SaveAsync` 和 `Delete`。添加以下方法：

    ```cs
    private async Task SaveAsync<T>(string folder, string filename, T item)
    {
        var filepath = $@"{_settings.DataPath}\{folder}\{filename}.json";
        await File.WriteAllTextAsync(filepath, JsonSerializer.Serialize<T>(item));
    }
    private Task DeleteAsync(string folder, string filename)
     {
         var filepath = $@"{_settings.DataPath}\{folder}\{filename}.json";
         if (File.Exists(filepath))
         {
             File.Delete(filepath);
         }
         return Task.CompletedTask;
     } 
    ```

    这些方法也是泛型的，以便尽可能多地共享代码并避免为每种类型的类（`BlogPost`、`Category`、`Comment` 和 `Tag`）重复代码。

1.  接下来，是时候通过添加获取博客文章的方法来实现 API 了。添加以下代码：

    ```cs
    public async Task<int> GetBlogPostCountAsync()
     {
         var list = await LoadAsync<BlogPost>(_settings.BlogPostsFolder);
         return list.Count;
     }
     public async Task<List<BlogPost>> GetBlogPostsAsync(int numberofposts, int startindex)
     {
         var list = await LoadAsync<BlogPost>(_settings.BlogPostsFolder);
         return list.Skip(startindex).Take(numberofposts).ToList();
     } 
    public async Task<BlogPost?> GetBlogPostAsync(string id)
        {
            var list = await LoadAsync<BlogPost>(_settings.BlogPostsFolder);
            return list.FirstOrDefault(bp => bp.Id == id);
        } 
    ```

    `GetBlogPostsAsync` 方法接受一些参数，我们将在稍后用于分页。它将从我们的 JSON 存储中获取博客文章，并返回我们请求的文章，跳过并取回正确的数量以供分页使用。

    我们还有一个返回当前博客文章计数的函数，我们将用它来进行分页。最后但同样重要的是，我们有 `GetBlogPostAsync` 用于从我们的 JSON 存储中获取单个博客文章。

1.  现在，我们需要为类别添加相同的方法。为此，添加以下代码：

    ```cs
     public async Task<List<Category>> GetCategoriesAsync()
        {
            return await LoadAsync<Category>(_settings.CategoriesFolder);
        }
        public async Task<Category?> GetCategoryAsync(string id)
        {
            var list = await LoadAsync<Category>(_settings.CategoriesFolder);
            return list.FirstOrDefault(c => c.Id == id);
        } 
    ```

1.  `Category` 方法没有分页支持。否则，它们应该看起来很熟悉，因为它们几乎与博客文章方法相同。

1.  现在，是时候为标签做同样的事情了。添加以下代码：

    ```cs
     public async Task<List<Tag>> GetTagsAsync()
        {
            return await LoadAsync<Tag>(_settings.TagsFolder);
        }
        public async Task<Tag?> GetTagAsync(string id)
        {
            var list = await LoadAsync<Tag>(_settings.TagsFolder);
            return list.FirstOrDefault(t => t.Id == id);
        } 
    ```

    如我们所见，`tag` 代码基本上是类别代码的副本。

    我们还需要一种方法来检索博客文章的评论。我们不会创建一个检索单个 `comment` 的方法；我们总是获取特定文章的所有评论。添加以下方法：

    ```cs
     public async Task<List<Comment>> GetCommentsAsync(string blogPostId)
        {
            var list = await LoadAsync<Comment>(_settings.
    CommentsFolder);
            return list.Where(t => t.BlogPostId == blogPostId).ToList();
        } 
    ```

    此方法将获取博客文章的所有评论。

1.  我们还需要几个用于保存数据的方法，所以接下来，我们将添加保存博客文章、分类、评论和标签的方法。

    添加以下代码：

    ```cs
    public async Task<BlogPost?> SaveBlogPostAsync(BlogPost item)
    {
        item.Id ??= Guid.NewGuid().ToString();
        await SaveAsync(_settings.BlogPostsFolder, item.Id, item);
        return item;
    }
    public async Task<Category?> SaveCategoryAsync(Category item)
    {
        item.Id ??= Guid.NewGuid().ToString();
        await SaveAsync(_settings.CategoriesFolder, item.Id, item);
        return item;
    }
    public async Task<Tag?> SaveTagAsync(Tag item)
    {
        item.Id ??= Guid.NewGuid().ToString();
        await SaveAsync(_settings.TagsFolder, item.Id, item);
        return item;
    }
    public async Task<Comment?> SaveCommentAsync(Comment item)
    {
        item.Id ??= Guid.NewGuid().ToString();
        await SaveAsync(_settings.CommentsFolder, item.Id, item);
        return item;
    } 
    ```

    我们首先要做的是检查项目的 `id` 是否为空。如果是，我们创建一个新的 `Guid`。这是新项目的 `id`。这将是存储在文件系统上的 JSON 文件的名称。

1.  现在我们有了保存和获取项目的方法。但有时事情并不按计划进行，我们需要一种方法来删除我们创建的项目。接下来，我们将添加一些删除方法。添加以下代码：

    ```cs
     public async Task DeleteBlogPostAsync(string id)
        {
            await DeleteAsync(_settings.BlogPostsFolder, id);

            var comments = await GetCommentsAsync(id);
            foreach (var comment in comments)
            {
                if (comment.Id != null)
                {
                    await DeleteAsync(_settings.CommentsFolder, comment.Id);
                }
            }
        }
        public async Task DeleteCategoryAsync(string id)
        {
            await DeleteAsync(_settings.CategoriesFolder, id);
        }
        public async Task DeleteTagAsync(string id)
        {
            await DeleteAsync(_settings.TagsFolder, id);
        }
        public async Task DeleteCommentAsync(string id)
        {
            await DeleteAsync(_settings.CommentsFolder, id);
        } 
    ```

我们刚刚添加的代码调用了 `DeleteAsync` 方法，该方法删除了博客文章、标签、分类等。

我们的 JSON 存储已完成！

最后，文件系统上将有四个文件夹，一个用于博客文章，一个用于分类，一个用于评论，一个用于标签。

下一步是将 Blazor 项目添加并配置为使用我们新的存储。

# 将 API 添加到 Blazor

现在我们有了一种访问存储在文件系统上的 JSON 文件的方法。在 GitHub 上的仓库中，你可以找到更多使用 RavenDB 或 SQL Server 存储我们数据的方法，但请注意保持重点（Blazor）。

现在是时候将 API 添加到我们的 Blazor 服务器项目中：

1.  在 `BlazorWebApp` 项目中，**添加对 `Data` 项目的项目引用**。打开 `Program.cs` 并添加以下命名空间：

    ```cs
    using Data;
    using Data.Models.Interfaces; 
    ```

1.  在 `.AddInteractiveWebAssemblyComponents();` 之后添加以下代码：

    ```cs
    builder.Services.AddOptions<BlogApiJsonDirectAccessSetting>().Configure(options =>
    {
        options.DataPath = @"..\..\..\Data\";
        options.BlogPostsFolder = "Blogposts";
        options.TagsFolder = "Tags";
        options.CategoriesFolder = "Categories";
        options.CommentsFolder = "Comments";
    });
    builder.Services.AddScoped<IBlogApi, BlogApiJsonDirectAccess>(); 
    ```

```cs
IOptions<BlogApiJsonDirectAccessSetting>, the dependency injection will return an object populated with the information we have supplied above. This is an excellent place to load configuration from our .NET configuration, a key vault, or a database.
```

我们还说明，当我们请求 `IBlogAPI` 时，我们将从我们的依赖注入中获取 `BlogApiJsonDirectAccess` 的实例。我们将在 *第四章*，*理解基本 Blazor 组件* 中返回依赖注入。

现在，我们可以使用我们的 API 来访问 Blazor 项目中的数据库。

# 摘要

本章教会了我们如何创建一个简单的 JSON 存储库来存储我们的数据。我们还了解到，如果您想查看其他选项，GitHub 仓库中还有其他替代方案。

我们还创建了一个接口来访问数据，我们将在本书的后面部分使用它。

在下一章中，我们将学习组件，特别是 Blazor 模板中的内置组件。我们还将使用本章中创建的 API 和存储库创建我们的第一个组件。
