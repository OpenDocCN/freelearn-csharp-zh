# 第十章：构建 Web 客户端（消费 Web 服务）

到目前为止，在这本书中，我们已经创建了 RESTful 服务，以便我们可以在项目内部或外部调用或消费这些服务。在本章中，我们将讨论这些服务的某些用例，以及消费 RESTful Web 服务的技巧和方法。

在本章中，我们将涵盖以下主题：

+   消费 RESTful Web 服务

+   构建 REST Web 客户端

# 消费 RESTful Web 服务

到目前为止，我们已经创建了 RESTful 服务，并使用代码示例讨论了服务器端代码。我们使用外部第三方工具，如 Postman 和 Advanced RESTClient，来消费这些服务。我们还使用模拟对象和单元测试期间消费了这些服务。虽然这些消费示例很有帮助，但它们并没有真正展示 RESTful 服务的优势，因为它们要么测试了其功能，要么验证了其输出。

可能存在这样的情况，你需要在另一个类似于控制器或甚至你自己的应用程序中消费或使用这些服务。这些应用程序可以是以下任何一种：

+   基于控制台

+   基于 Web

+   基于移动或其他设备

让我们看看我们之前讨论的一个应用程序：假设你需要某种机制来实现或消费外部 API（在这种情况下，是 PayPal），同时集成在线支付系统。在这种情况下，我们之前已经介绍的外部工具，如 Postman 和 Advanced RESTClient，无法帮助；为了满足你的需求，你需要一个 REST 客户端。

以下图示说明了如何使用 HTTP 客户端通过 REST 客户端消费服务。在以下图中，REST 客户端正在与 ASP.NET Core 中开发的外部和服务进行交互（请求、响应），这些服务位于同一服务器或不同服务器上。

![](img/a80a5a5f-efbd-4d42-a85e-16b910703271.png)

**Web**、**控制台**、**移动**等，都是通过 REST 客户端帮助消费这些服务的客户端。

我们现在将讨论如何构建一个 REST 客户端，我们可以使用它来消费我们应用程序中的其他 RESTful Web 服务（即 API）。

# 构建 REST Web 客户端

RESTful 服务可能是也可能不是 Web 应用程序的一部分。Web 应用程序可以调用或消费来自同一应用程序的外部 API 或服务。使服务与应用程序消费这些服务的交互或通信（请求、响应）成为可能的程序称为**客户端**。

客户端帮助应用程序通过 API 进行（请求、响应）通信。

在本节中，我们将创建一个 Web 客户端。Web 客户端是用 ASP.NET Core 编写的应用程序或程序。

在我们构建测试 Web 客户端之前，我们需要讨论我们需要调用什么。

继续我们的 FlixOne BookStore 示例，以下表格列出了我们将调用和消费的生产者和服务：

| **API 资源** | **描述** |
| --- | --- |
| `GET /api/product` | 获取产品列表。 |
| `GET /api/product{id}` | 获取一个产品。 |
| `PUT /api/product{id}` | 更新现有产品。 |
| `DELETE /api/product{id}` | 删除现有产品。 |
| `POST /api/product` | 添加新产品。 |

我们的 FlixOne 产品服务是为以下任务设计的：

+   添加新产品

+   更新现有产品

+   删除现有产品

+   获取产品

我们已经确保了我们的产品 API 支持 Swagger（请参阅前面的章节以获取更多信息），让我们开始吧。要开始此项目，请按照以下步骤操作：

1.  首先，运行 Visual Studio 2017

1.  选择“文件 | 打开”

1.  选择项目 FlixOne.BookStore.ProductService

1.  按下 *F5* 键或直接从菜单中点击来运行项目

1.  输入以下 URL：`http://localhost:10065/swagger/`

你现在应该能看到你的产品 API 的 Swagger 文档，如下截图所示：

![图片](img/4202f70f-415a-491c-affe-7cf9831b0210.png)

产品 API 文档

# 构建 Web 客户端

我们已经讨论了哪些 API 需要消费以及哪些资源返回什么，现在是时候构建我们的 Web 客户端，以便我们可以消费和调用我们的产品 API。为此，请按照以下步骤操作：

1.  要为我们的新项目创建一个全新的解决方案，请转到“文件 | 新建 | 项目”（或按 *Ctrl* + *Shift* + *N*），如下截图所示。

![图片](img/7f232c75-ae75-41b4-84b9-5ff7b5735f92.png)

1.  从“新建项目”中选择 ASP.NET Core Web 应用程序。

1.  命名项目 `FlixOne.BookStore.WebClient` 并然后点击如下截图所示的“确定”：

![图片](img/4dd65c06-4f52-43fd-9c75-68a654d22aae.png)

1.  从如下截图所示的 ASP.NET Core 模板窗口中选择 Web Application 并点击“确定”：

![图片](img/b8014e2a-003d-4805-8349-e71cb4fc5e87.png)

1.  现在运行项目 Debug | 开始调试或按 *F5* 键。

1.  你现在应该看到一个默认的网站模板。

1.  我们现在将使用**RestSharp**创建一个 Web 客户端。我们需要添加对 RestSharp 的支持，以便能够对我们的 API 资源进行 HTTP 协议调用。

RestSharp 是一个轻量级的 HTTP 客户端库。你可以根据需要对其进行更改，因为它是一个开源库。你可以在[`github.com/restsharp/RestSharp`](https://github.com/restsharp/RestSharp)找到完整的仓库。

1.  使用如下截图所示的“打开包管理器”（在解决方案资源管理器中右键单击解决方案），添加一个 NuGet 包：

![图片](img/c88554eb-4b19-480f-8597-e0b8d2645b0c.png)

1.  搜索“RestSharp”，勾选包含预发布版本的复选框，然后点击“安装”，如下截图所示：

![图片](img/64ac9dd2-1ba4-409d-a422-2104d598a3f1.png)

选择 RestSharp NuGet 包

1.  现在将安装所需的包，如下截图所示：

![图片](img/65a1722d-161f-4ab7-a8d5-0e24d31e113d.png)

安装 RestSharp 包

在继续前进之前，让我们首先确保我们的产品 API 能够正确工作。运行产品 API 项目，打开 Swagger，然后按照以下方式点击`GET /api/product/productlist`资源：

![](img/d2bd41c1-e23c-40be-805d-0fefb43d419d.png)

执行前面的资源后，你应该会看到一个完整的产品列表，如下面的截图所示：

![](img/dbc16f11-5f14-4aa5-bbf3-a41d6d6c9cc6.png)

尝试所有可用的资源以确保你的产品 API 能够正确工作。

# 编写代码

到目前为止，我们已经为编写 REST Web 客户端的代码准备好了所需的东西；在本节中，我们将编写实际的代码：

1.  添加简单的代码来调用或消费你的产品 API。

如果你在同一解决方案中创建了一个新项目（参考*Cooking the web client*部分的*步骤 1*），请在开始你的 web 客户端项目之前确保 Product API 项目正在运行。

1.  在`Client`文件夹中添加一个新的类（*Ctrl* + *Shift* + *C*）并命名为`RestSharpWebClient`。

![](img/2981769f-110c-479f-a737-834a493a847b.png)

1.  现在打开`RestSharpWebClient`类并添加以下代码：

```cs
private readonly RestClient _client = new RestClient("http://localhost:10065/api/");
```

前面的代码初始化 RestSharp 的 RestClient 并接受一个字符串或 URI 作为基础 URL。

URI 代表统一资源标识符，它是一个用于标识资源的字符串表示。

你可能会遇到存在多个环境的情况；在这种情况下，你应该存储一个 URI，根据你的环境将其指向相应的位置。例如，你可以为你的开发环境使用 URI `http://devserver:10065/api/`，或者为你的 QA 环境使用 URI `http://testenv:10068/api/`。你应该将这些键存储在`config`文件或类似的位置，以便值易于访问。我们建议使用`new RestClient(somevariableforURI);`。

在我们的应用程序中，产品 API 在本地主机上运行，监听端口为`10065`。在你的情况下可能会有所不同。

让我们讨论以下代码片段，以调用或消费`GET /api/product /productlist`资源并填充完整的产品列表，如下所示：

```cs
public List<ProductViewModel> GetProducts()
{
  var request = new RestRequest("product/productlist", Method.GET);
  var response = _client.Execute<List<ProductViewModel>>(request);
  return response.Data ?? new List<ProductViewModel> {new
  ProductViewModel()};
}
```

在这里，我们使用`RestRequest`进行`GET`请求，其中我们传递了一个资源和方法。

要使用`productid`获取特定产品，请输入以下代码：

```cs
public ProductViewModel GetProductDetails(string productId)
{
  var request = new RestRequest("product/{productid}", Method.GET);
  request.AddParameter("productid", productId);
  var response = _client.Execute<ProductViewModel>(request);
  return response.Data ?? new ProductViewModel();
}
```

在前面的代码块中，`GetProductDetails`方法与`GetProducts`方法做类似的事情。区别在于它接受`productId`参数。

以下是我们 REST 客户端的完整代码：

```cs
public class RestSharpWebClient
{
  private readonly RestClient _client = new
  RestClient("http://localhost:10065/api/");
  public List<ProductViewModel> GetProducts()
  {
    var request = new RestRequest("product/productlist", 
    Method.GET);
    var response = _client.Execute<List<ProductViewModel>>
    (request);
    //To avoid any exception lets return an empty view model
    //On production environment return exact exception or your 
    custom code
    return response.Data ?? new List<ProductViewModel> {new
    ProductViewModel()};
  }
  public ProductViewModel GetProductDetails(string productId)
  {
    var request = new RestRequest("product/{productid}", 
    Method.GET);
    request.AddParameter("productid", productId);
    var response = _client.Execute<ProductViewModel>(request);
    //To avoid any exception lets return an empty view model
    //On production environment return exact exception or your 
    custom code
    return response.Data ?? new ProductViewModel();
  }
  public bool AddProduct(ProductViewModel product)
  {
    var request = new RestRequest("product/addproduct", 
    Method.POST);
    request.AddBody(product);
    var response = _client.Execute(request);
    return response.StatusCode == HttpStatusCode.OK;
  }
  public bool UpdateProduct(string id, ProductViewModel product)
  {
    var request = new RestRequest("updateproduct", Method.PUT);
    request.AddQueryParameter("productid", id);
    request.AddBody(product);
    var response = _client.Execute(request);
    return response.StatusCode == HttpStatusCode.NoContent;
  }
  public bool DeleteProduct(string id, ProductViewModel product)
  {
    var request = new RestRequest("deleteproduct", Method.DELETE);
    request.AddQueryParameter("productid", id);
    request.AddBody(product);
    var response = _client.Execute(request);
    return response.StatusCode == HttpStatusCode.NoContent;
  }
}
```

使用前面的代码片段，你现在已经添加了调用和消费你的产品 API 的功能。

# 实现 REST Web 客户端

RESTful 服务可能是也可能不是你 Web 应用的一部分，但我们需要了解如何实现它们。

因此，现在是时候做一些实际的工作了。将`ProductController`添加到项目中，以及以下操作：

```cs
public ActionResult Index()
{
  var webClient = new RestSharpWebClient();
  var products = webClient.GetProducts();
  return View("ProductList", products);
}

```

看一下前面的代码片段。我们已经调用了`RestSharpWebClient`的`GetProducts`方法，并用产品完整列表填充了`Index.cshtml`视图。

要添加另一个操作方法，输入以下完整的`ProductController`代码。以下代码片段包含`Index`操作方法，并给出了产品列表：

```cs
public class ProductController : Controller
{
  public ActionResult Index()
  {
    var webClient = new RestSharpWebClient();
    var products = webClient.GetProducts();
    return View("ProductList", products);
  }
  public ActionResult Details(string id)
  {
    var webClient = new RestSharpWebClient();
    var products = webClient.GetProductDetails(id);
    return View(products);
  }
```

现在我们来看两个`Create`操作方法：`HttpGet`和`HttpPost`。第一个提供了一个输入的入口屏幕，第二个使用`HttpPost`方法提交所有数据（输入值）。在服务器端，你可以通过`IFormCollection`参数接收所有数据，你也可以轻松地编写逻辑来获取`ProductViewModel`中的所有值。

```cs
  public IActionResult Create()
  {
    return View();
  }
  [HttpPost]
  [ValidateAntiForgeryToken]
  public IActionResult Create(IFormCollection collection)
  {
    try
    {
      var product = new ProductViewModel
      {
        ProductId = Guid.NewGuid(),
        ProductName = collection["ProductName"],
        ProductDescription = collection["ProductDescription"],
        ProductImage = collection["ProductImage"],
        ProductPrice = Convert.ToDecimal(collection["ProductPrice"]),
        CategoryId = new Guid("77DD5B53-8439-49D5-9CBC-DC5314D6F190"),
        CategoryName = collection["CategoryName"],
        CategoryDescription = collection["CategoryDescription"]
      };
      var webClient = new RestSharpWebClient();
      var producresponse = webClient.AddProduct(product);
      if (producresponse)
      return RedirectToAction(nameof(Index));
      throw new Exception();
    }
    catch
    {
      return View();
    }
  }
```

你也可以编写一个接受`ProductViewModel`类型参数的`HttpPost`方法。

以下代码片段显示了`Edit`操作方法的代码，它与`Create`操作方法类似，但不同之处在于它更新现有数据而不是插入新数据：

```cs

  public ActionResult Edit(string id)
  {
    var webClient = new RestSharpWebClient();
    var product = webClient.GetProductDetails(id);
    return View(product);
  }
  [HttpPost]
  [ValidateAntiForgeryToken]
  public ActionResult Edit(string id, IFormCollection collection)
  {
    try
    {
      var product = new ProductViewModel
      {
        ProductId = new Guid(collection["ProductId"]),
        ProductName = collection["ProductName"],
        ProductDescription = collection["ProductDescription"],
        ProductImage = collection["ProductImage"],
        ProductPrice = Convert.ToDecimal(collection["ProductPrice"]),
        CategoryId = new Guid(collection["CategoryId"]),
        CategoryName = collection["CategoryName"],
        CategoryDescription = collection["CategoryDescription"]
      };
      var webClient = new RestSharpWebClient();
      var producresponse = webClient.UpdateProduct(id, product);
      if (producresponse)
      return RedirectToAction(nameof(Index));
      throw new Exception();
    }
    catch
    {
      return View();
    }
  }
```

`Delete`操作方法旨在从数据库或集合中删除特定的记录或数据。`HttpGet`的`Delete`操作方法根据给定的 ID 获取记录并显示准备修改的数据。另一个`HttpPost`的`Delete`操作将修改后的数据发送到服务器进行进一步处理。这意味着系统可以删除数据和记录。

```cs

  public ActionResult Delete(string id)
  {
    var webClient = new RestSharpWebClient();
    var product = webClient.GetProductDetails(id);
    return View(product);
  }
  [HttpPost]
  [ValidateAntiForgeryToken]
  public ActionResult Delete(string id, IFormCollection collection)
  {
    try
    {
      var product = new ProductViewModel
      {
        ProductId = new Guid(collection["ProductId"]),
        ProductName = collection["ProductName"],
        ProductDescription = collection["ProductDescription"],
        ProductImage = collection["ProductImage"],
        ProductPrice = Convert.ToDecimal(collection["ProductPrice"]),
        CategoryId = new Guid(collection["CategoryId"]),
        CategoryName = collection["CategoryName"],
        CategoryDescription = collection["CategoryDescription"]
      };
      var webClient = new RestSharpWebClient();
      var producresponse = webClient.DeleteProduct(id, product);
      if (producresponse)
      return RedirectToAction(nameof(Index));
      throw new Exception();
    }
    catch
    {
      return View();
    }
  }
}
```

现在，让我们打开`Shared`文件夹中的`_Layout.cshtml`并添加以下行，以添加到我们新添加的`ProductController`的链接：

```cs
<li><a asp-area="" asp-controller="Product" asp-action="Index">Web Client</a></li>
```

当你运行项目时，你应该会看到一个名为 Web Client 的新菜单，如下面的截图所示：

![图片](img/0860e52c-5016-46d2-a6df-d0812d31377e.png)

我们现在准备好看到一些结果。点击 Web Client 菜单，你会看到以下屏幕：

![图片](img/1626a7e2-dcdc-4ad2-983c-4410f4e9784a.png)

从前面的屏幕中，你可以执行其他操作，以及调用和消费你的产品 API——即创建、编辑和删除。

# 摘要

创建 RESTful 服务对任何项目都很重要，但如果无法使用这些服务，这些服务就毫无用处。在本章中，我们探讨了如何将 RestSharp 支持添加到我们的 Web 项目中，并消费我们预先开发的产品 API。我们还创建了一个可以通过在 ASP.NET Core 网页上渲染输出来消费 Web 服务的 Web 客户端。

在下一章中，我们将讨论微服务这一热门话题，这是服务分离的下一级。我们将讨论微服务如何通信，它们的优点是什么，以及为什么我们需要它们。
