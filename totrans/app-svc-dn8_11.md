# 11

# 使用 SignalR 进行实时通信广播

在本章中，您将了解 SignalR，这是一种技术，它使开发者能够创建一个可以拥有多个客户端，并能实时向所有客户端或其中一部分客户端广播消息的服务。典型的例子是群聊应用。其他例子包括需要即时更新信息的系统，如股票价格。

本章将涵盖以下主题：

+   理解 SignalR

+   使用 SignalR 构建实时通信服务

+   使用 SignalR JavaScript 库构建 Web 客户端

+   构建一个.NET 控制台应用程序客户端

+   使用 SignalR 进行数据流

# 理解 SignalR

要理解 SignalR 解决的问题，我们需要了解没有它时 Web 开发是什么样的。Web 的基础是 HTTP，超过 30 年来，它对于构建通用网站和服务来说一直很好。然而，Web 并没有为需要网页即时更新新信息的特定场景而设计。

## 网络上实时通信的历史

要了解 SignalR 的好处，了解 HTTP 的历史以及组织如何努力使其更适合客户端和服务器之间的实时通信是有帮助的。

在 20 世纪 90 年代 Web 的早期，浏览器必须向 Web 服务器发送完整的 HTTP `GET`请求，以获取新鲜信息向访客展示。

在 1999 年底，微软发布了带有名为**XMLHttpRequest**组件的 Internet Explorer 5，该组件可以在后台进行异步 HTTP 调用。这，连同**动态 HTML**（**DHTML**），使得网页的部分可以平滑地用新鲜数据更新。

这种技术的优势很明显，很快，所有浏览器都添加了相同的组件。

## AJAX

Google 充分利用这一功能构建了像 Google Maps 和 Gmail 这样的聪明 Web 应用。几年后，这项技术被普遍称为**异步 JavaScript 和 XML**（**AJAX**）。

虽然 AJAX 仍然使用 HTTP 进行通信，但它有一些局限性：

+   首先，HTTP 是一个请求-响应通信协议，这意味着服务器不能向客户端推送数据。它必须等待客户端发起请求。

+   其次，HTTP 请求和响应消息具有包含大量可能不必要的开销的头部。

## WebSocket

**WebSocket**是全双工的，这意味着客户端或服务器都可以发起新的通信数据。WebSocket 在整个连接生命周期中使用相同的 TCP 连接。它发送的消息大小也更有效率，因为它们以 2 个字节的最小帧格式发送。

WebSocket 通过 HTTP 端口`80`和`443`工作，因此它与 HTTP 协议兼容，WebSocket 握手使用 HTTP 的**Upgrade**头从 HTTP 协议切换到 WebSocket 协议。

现代网络应用程序预期提供最新的信息。实时聊天是典型的例子，但还有许多潜在的应用，从股价到游戏。

无论何时你需要服务器将更新推送到网页，你都需要一种兼容网页的、实时通信技术。WebSocket 可以使用，但并非所有客户端都支持它。您可以使用以下链接中的网页检查哪些客户端支持 WebSocket：[`caniuse.com/websockets`](https://caniuse.com/websockets)。

WebSocket 或 WebSockets？ “**WebSocket** 协议于 2011 年由 IETF 标准化为 RFC 6455。允许网络应用程序使用此协议的当前 API 规范被称为 *WebSockets*。” 有关更多信息，请参阅以下链接中的维基百科页面：[`en.wikipedia.org/wiki/WebSocket`](https://en.wikipedia.org/wiki/WebSocket)。

## 介绍 SignalR

**ASP.NET Core SignalR** 是一个开源库，通过在多个底层通信技术之上提供抽象，简化了向应用程序添加实时 Web 功能的过程，允许您使用 C# 代码添加实时通信功能。

开发者不需要理解或实现底层技术，SignalR 将根据访问者的 Web 浏览器支持的底层技术自动切换。例如，当 WebSocket 可用时，SignalR 将使用 WebSocket，当不可用时，它将优雅地回退到其他技术，如 AJAX 长轮询，而您的应用程序代码保持不变。

SignalR 是一个服务器到客户端的 **远程过程调用**（RPCs）API。RPCs 从服务器端的 .NET 代码调用客户端的 JavaScript 函数。SignalR 有中心点来定义管道，并自动使用两个内置中心点协议：JSON 和基于 MessagePack 的二进制协议来处理消息分发。

在服务器端，SignalR 在 ASP.NET Core 运行的任何地方都可以运行：Windows、macOS 或 Linux 服务器。SignalR 支持以下客户端平台：

+   包括 Chrome、Firefox、Safari 和 Edge 在内的当前浏览器的 JavaScript 客户端。

+   .NET 客户端，包括 Blazor、.NET MAUI 和用于 Android 和 iOS 移动应用的 Xamarin。

+   Java 8 及以后版本。

## Azure SignalR 服务

之前我提到，将 SignalR 服务托管项目与使用 JavaScript 库作为客户端的 Web 项目分开是一个好的实践。这是因为 SignalR 服务可能需要处理大量的并发客户端请求，并且需要快速响应对所有请求。

一旦您将 SignalR 托管分离出来，您就可以利用 **Azure SignalR 服务**。这提供了全球覆盖和世界级的数据中心和网络，并且可以扩展到数百万个连接，同时满足 SLA，如提供合规性和高安全性。

您可以在以下链接中了解更多关于 Azure SignalR 服务的相关信息：[`learn.microsoft.com/en-us/azure/azure-signalr/signalr-overview`](https://learn.microsoft.com/en-us/azure/azure-signalr/signalr-overview)。

## 设计方法签名

当为 SignalR 服务设计方法签名时，定义具有单个消息参数的方法而不是多个简单类型参数是良好实践。这种良好实践不是由 SignalR 技术强制执行的，因此您必须自律。

例如，而不是传递多个 `string`（或其他类型）值，定义一个具有多个属性的类型，用作单个 `Message` 参数，如下面的代码所示：

```cs
// Bad practice: RPC method with multiple parameters.
public void SendMessage(string to, string body)
// Good practice: single parameter using a complex type.
public class Message
{
  public string To { get; set; }
  public string Body { get; set; }
}
public void SendMessage(Message message) 
```

这种良好实践的原因是它允许未来的更改，例如为消息 `Title` 添加第三个属性。对于不良实践的例子，需要添加一个名为 `title` 的第三个 `string` 参数，并且现有的客户端会因为它们没有发送额外的 `string` 值而得到错误。但是，使用良好实践的例子不会破坏方法签名，因此现有的客户端可以像更改之前一样继续调用它。在服务器端，额外的 `Title` 属性将只有一个 `null` 值，可以进行检查，也许可以将其设置为默认值。

SignalR 方法参数被序列化为 JSON，因此如果需要，所有嵌套对象在 JavaScript 中都是可访问的。

现在我们已经探讨了 SignalR 的基础知识及其各个方面，如方法签名设计的好实践，让我们来看看如何使用 SignalR 构建实时通信服务。

# 使用 SignalR 构建实时通信服务

SignalR 的 **服务器** 库包含在 ASP.NET Core 中，但 JavaScript 的 **客户端** 库不是自动包含在项目中的。记住，SignalR 支持多种客户端类型，而使用 JavaScript 的网页只是其中之一。

我们将使用 **Library Manager CLI** 从 **unpkg** 获取客户端库，**unpkg** 是一个可以交付 Node.js 包管理器中找到的任何内容的 **内容分发网络**（**CDN**）。

让我们在 ASP.NET Core MVC 项目中添加一个 SignalR 服务器端中心和一个客户端 JavaScript，以实现一个允许访客向以下地址发送消息的聊天功能：

+   当前正在使用网站的每个人。

+   动态定义的组。

+   指定单个用户。

**良好实践**：在生产解决方案中，最好将 SignalR 中心托管在单独的 Web 项目中，以便它可以独立于网站的其他部分进行托管和扩展。实时通信往往会对网站造成过大的负载。

## 定义一些共享模型

首先，我们将定义两个可以在服务器端和客户端 .NET 项目中使用的共享模型，这些项目将与我们的聊天服务一起工作：

1.  使用您首选的代码编辑器创建一个新项目，如下列所示：

    +   项目模板：**类库** / `classlib`

    +   解决方案文件和文件夹：`Chapter11`

    +   项目文件和文件夹：`Northwind.Common`

1.  在`Northwind.Common`项目中，将`Class1.cs`文件重命名为`UserModel.cs`。

1.  修改其内容以定义一个用于注册用户姓名、唯一连接 ID 以及他们所属的组别的模型，如下面的代码所示：

    ```cs
    namespace Northwind.Chat.Models;
    public class UserModel
    {
      public string Name { get; set; } = null!;
      public string ConnectionId { get; set; } = null!;
      public string? Groups { get; set; } // comma-separated list
    } 
    ```

    **良好实践**：在实际应用中，您可能希望为`Groups`属性使用`string`值的集合，但这个编码任务并不是关于如何提供编辑多个`string`值的 Web 用户体验。我们将提供一个简单的文本框，并专注于学习 SignalR。

1.  在`Northwind.Common`项目中，添加一个名为`MessageModel.cs`的类文件。修改其内容以定义一个消息模型，包含消息接收者、消息发送者以及消息正文等属性，如下面的代码所示：

    ```cs
    namespace Northwind.Chat.Models;
    public class MessageModel
    {
      public string From { get; set; } = null!;
      public string To { get; set; } = null!;
      public string? Body { get; set; }
    } 
    ```

## 启用服务器端 SignalR 中心

接下来，我们将在 ASP.NET Core MVC 项目中在服务器端启用一个 SignalR 中心：

1.  使用您喜欢的代码编辑器添加一个新项目，如下面的列表所示：

    +   项目模板：**ASP.NET Core Web App (Model-View-Controller)** / `mvc`

    +   解决方案文件和文件夹：`Chapter11`

    +   项目文件和文件夹：`Northwind.SignalR.Service.Client.Mvc`

    +   认证类型：无。

    +   配置 HTTPS：已选择。

    +   启用 Docker：已清除。

    +   不要使用顶级语句：已清除。

1.  在`Northwind.SignalR.Service.Client.Mvc`项目中，将警告视为错误，并添加对`Northwind.Common`项目的项目引用，如下面的标记所示：

    ```cs
    <ItemGroup>
      <ProjectReference
        Include="..\Northwind.Common\Northwind.Common.csproj" />
    </ItemGroup> 
    ```

1.  在`Properties`文件夹中，在`launchSettings.json`中，在`https`配置文件中，修改`applicationUrl`以使用端口`5111`进行`https`连接和`5112`进行`http`连接，如下面高亮显示的配置所示：

    ```cs
    "**https**": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
    **"applicationUrl"****:****"https://localhost:5111;http://localhost:5112"****,**
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      } 
    ```

1.  在`Northwind.SignalR.Service.Client.Mvc`项目中，添加一个名为`ChatHub.cs`的类文件。

1.  在`ChatHub.cs`中，修改其内容以继承自`Hub`类并实现两个可以被客户端调用的方法，如下面的代码所示：

    ```cs
    using Microsoft.AspNetCore.SignalR; // To use Hub.
    using Northwind.Chat.Models; // To use UserModel, MessageModel.
    namespace Northwind.SignalR.Service.Hubs;
    public class ChatHub : Hub
    {
      // A new instance of ChatHub is created to process each method so we
      // must store user names, connection IDs, and groups in a static field.
      private static Dictionary<string, UserModel> Users = new();
      public async Task Register(UserModel newUser)
      {
        UserModel user;
        string action = "registered as a new user";
        // Try to get a stored user with a match on new user.
        if (Users.ContainsKey(newUser.Name))
        {
          user = Users[newUser.Name];
          // Remove any existing group registrations.
          if (user.Groups is not null)
          {
            foreach (string group in user.Groups.Split(','))
            {
              await Groups.RemoveFromGroupAsync(user.ConnectionId, group);
            }
          }
          user.Groups = newUser.Groups;
          // Connection ID might have changed if the browser 
          // refreshed so update it.
          user.ConnectionId = Context.ConnectionId;
          action = "updated your registered user";
        }
        else
        {
          if (string.IsNullOrEmpty(newUser.Name))
          {
            // Assign a GUID for name if they are anonymous.
            newUser.Name = Guid.NewGuid().ToString();
          }
          newUser.ConnectionId = Context.ConnectionId;
          Users.Add(key: newUser.Name, value: newUser);
          user = newUser;
        }
        if (user.Groups is not null)
        {
          // A user does not have to belong to any groups
          // but if they do, register them with the Hub.
          foreach (string group in user.Groups.Split(','))
          {
            await Groups.AddToGroupAsync(user.ConnectionId, group);
          }
        }
        // Send a message to the registering user informing of success.
        MessageModel message = new() 
        { 
          From = "SignalR Hub", To = user.Name, 
          Body = string.Format(
            "You have successfully {0} with connection ID {1}.",
            arg0: action, arg1: user.ConnectionId)
        };
        IClientProxy proxy = Clients.Client(user.ConnectionId);
        await proxy.SendAsync("ReceiveMessage", message);
      }
      public async Task SendMessage(MessageModel message)
      {
        IClientProxy proxy;
        if (string.IsNullOrEmpty(message.To))
        {
          message.To = "Everyone";
          proxy = Clients.All;
          await proxy.SendAsync("ReceiveMessage", message);
          return;
        }
        // Split To into a list of user and group names.
        string[] userAndGroupList = message.To.Split(',');
        // Each item could be a user or group name.
        foreach (string userOrGroup in userAndGroupList)
        {
          if (Users.ContainsKey(userOrGroup))
          {
            // If the item is in Users then send the message to that user
            // by looking up their connection ID in the dictionary.
            message.To = $"User: {Users[userOrGroup].Name}";
            proxy = Clients.Client(Users[userOrGroup].ConnectionId);
          }
          else // Assume the item is a group name to send the message to.
          {
            message.To = $"Group: {userOrGroup}";
            proxy = Clients.Group(userOrGroup);
          }
          await proxy.SendAsync("ReceiveMessage", message);
        }
      }
    } 
    ```

    注意以下内容：

    +   `ChatHub`有一个私有字段用于存储已注册用户列表。它是一个以他们的名字作为唯一键的字典。

    +   `ChatHub`有两个客户端可以调用的方法：`Register`和`SendMessage`。

    +   `Register`方法接受一个类型为`UserModel`的单个参数。用户的姓名、连接 ID 和组别被存储在静态字典中，以便以后可以使用用户姓名查找连接 ID 并直接向该用户发送消息。在注册新用户或更新现有用户的注册信息后，会向客户端发送一条消息，告知他们操作成功。

    +   `SendMessage` 有一个类型为 `MessageModel` 的单个参数。该方法根据 `To` 属性的值进行分支。如果 `To` 没有值，它调用 `All` 属性以获取一个将与每个客户端通信的代理。如果 `To` 有值，则使用逗号分隔符将 `string` 分割成一个数组。检查数组中的每个项是否与 `Users` 中的用户匹配。如果匹配，它调用 `Client` 方法以获取一个将仅与该客户端通信的代理。如果不匹配，该项可能是一个组，因此它调用 `Group` 方法以获取一个将仅与该组成员通信的代理。最后，它使用代理异步发送消息。

1.  在 `Program.cs` 中，导入你的 SignalR 中心的命名空间，如下面的代码所示：

    ```cs
    using Northwind.SignalR.Service.Hubs; // To use ChatHub. 
    ```

1.  在配置服务的部分，添加一个语句以向服务集合添加对 SignalR 的支持，如下面的代码所示：

    ```cs
    builder.Services.AddSignalR(); 
    ```

1.  在配置 HTTP 管道的部分，在调用映射控制器路由之前，添加一个语句将相对 URL 路径 `/chat` 映射到你的 SignalR 中心，如下面的代码所示：

    ```cs
    app.MapHub<ChatHub>("/chat"); 
    ```

# 使用 SignalR JavaScript 库构建 Web 客户端

接下来，我们将添加 SignalR 客户端 JavaScript 库，以便我们可以在网页上使用它：

1.  打开 `Northwind.SignalR.Service.Client.Mvc` 项目/文件夹的命令提示符或终端。

1.  按照以下命令安装库管理器 CLI 工具：

    ```cs
    dotnet tool install -g Microsoft.Web.LibraryManager.Cli 
    ```

    此工具可能已经全局安装。要更新到最新版本，重复命令，但将 `install` 替换为 `update`。

1.  输入命令将 `signalr.js` 和 `signalr.min.js` 库添加到项目，从 `unpkg` 源，如下面的命令所示：

    ```cs
    libman install @microsoft/signalr@latest -p unpkg -d wwwroot/js/signalr --files dist/browser/signalr.js --files dist/browser/signalr.min.js 
    ```

    从 PDF 中复制长命令并直接粘贴到命令提示符是不推荐的。始终在基本的文本编辑器中清理它们，以删除多余的换行符等，然后再重新复制。为了更容易输入长命令行，你可以从以下链接复制它们：[`github.com/markjprice/apps-services-net8/blob/main/docs/command-lines.md`](https://github.com/markjprice/apps-services-net8/blob/main/docs/command-lines.md)

1.  注意成功消息，如下面的输出所示：

    ```cs
    wwwroot/js/signalr/dist/browser/signalr.js written to disk
    wwwroot/js/signalr/dist/browser/signalr.min.js written to disk
    Installed library "@microsoft/signalr@latest" to "wwwroot/js/signalr" 
    ```

Visual Studio 2022 还有一个用于添加客户端 JavaScript 库的 GUI。要使用它，右键单击一个 Web 项目，然后导航到 **添加** | **客户端库**。

## 向 MVC 网站添加聊天页面

接下来，我们将向主页添加聊天功能：

1.  在 `Views/Home` 的 `Index.cshtml` 中，修改其内容，如下面的标记所示：

    ```cs
    @using Northwind.Chat.Models
    @{
      ViewData["Title"] = "SignalR Chat";
    }
    <div class="container">
      <h1>@ViewData["Title"]</h1>
      <hr />
      <div class="row">
        <div class="col">
          <h2>Register User</h2>
          <div class="mb-3">
            <label for="myName" class="form-label">My name</label>
            <input type="text" class="form-control" 
                   id="myName" value="Alice" required />
          </div>
          <div class="mb-3">
            <label for="myGroups" class="form-label">My groups</label>
            <input type="text" class="form-control" 
                   id="myGroups" value="Sales,IT" />
          </div>
          <div class="mb-3">
            <input type="button" class="form-control" 
                   id="registerButton" value="Register User" />
          </div>
        </div>
        <div class="col">
          <h2>Send Message</h2>
          <div class="mb-3">
            <label for="from" class="form-label">From</label>
            <input type="text" class="form-control" 
                   id="from" value="Alice" readonly />
          </div>
          <div class="mb-3">
            <label for="to" class="form-label">To</label>
            <input type="text" class="form-control" id="to" />
          </div>
          <div class="mb-3">
            <label for="body" class="form-label">Body</label>
            <input type="text" class="form-control" id="body" />
          </div>
          <div class="mb-3">
            <input type="button" class="form-control" 
                   id="sendButton" value="Send Message" />
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col">
          <hr />
          <h2>Messages received</h2>
          <ul id="messages"></ul>
        </div>
      </div>
    </div>
    <script src="img/signalr.js"></script>
    <script src="img/chat.js"></script> 
    ```

    注意以下内容：

    +   页面上有三个部分：**注册用户**、**发送消息**和**收到的消息**。

    +   **注册用户**部分有两个输入框用于访客的姓名，一个逗号分隔的列表，列出他们想要成为成员的组，以及一个点击以注册的按钮。

    +   **发送消息**部分有三个输入框，分别用于输入消息发送者的用户名、消息接收者的用户名和群组名，以及消息正文，还有一个点击按钮来发送消息。

    +   **接收到的消息**部分有一个项目符号列表元素，当收到消息时，会动态填充一个列表项。

    +   有两个脚本元素用于 SignalR JavaScript 客户端库和聊天客户端的 JavaScript 实现。

1.  在`wwwroot/js`中添加一个名为`chat.js`的新 JavaScript 文件，并修改其内容，如下面的代码所示：

    ```cs
    "use strict";
    var connection = new signalR.HubConnectionBuilder()
      .withUrl("/chat").build();
    document.getElementById("registerButton").disabled = true;
    document.getElementById("sendButton").disabled = true;
    document.getElementById("myName").addEventListener("input",
      function () {
        document.getElementById("from").value = 
          document.getElementById("myName").value;
      }
    );
    connection.start().then(function () {
      document.getElementById("registerButton").disabled = false;
      document.getElementById("sendButton").disabled = false;
    }).catch(function (err) {
      return console.error(err.toString());
    });
    connection.on("ReceiveMessage", function (received) {
      var li = document.createElement("li");
      document.getElementById("messages").appendChild(li);
      li.textContent =
        // This string must use backticks ` to enable an interpolated 
        // string. If you use single quotes ' then it will not work!
        `To ${received.to}, From ${received.from}: ${received.body}`;
    });
    document.getElementById("registerButton").addEventListener("click",
      function (event) {
        var registermodel = {
          name: document.getElementById("myName").value,
          groups: document.getElementById("myGroups").value
        };
        connection.invoke("Register", registermodel).catch(function (err) {
          return console.error(err.toString());
        });
        event.preventDefault();
      });
    document.getElementById("sendButton").addEventListener("click",
      function (event) {
        var messagemodel = {
          from: document.getElementById("from").value,
          to: document.getElementById("to").value,
          body: document.getElementById("body").value
        };
        connection.invoke("SendMessage", messagemodel).catch(function (err) {
          return console.error(err.toString());
        });
        event.preventDefault();
    }); 
    ```

    注意以下内容：

    +   脚本创建了一个 SignalR 中心连接构建器，指定服务器上聊天中心的相对 URL 路径`/chat`。

    +   脚本在成功连接到服务器端中心之前禁用**注册**和**发送**按钮。

    +   为**我的名字**文本框添加了一个`input`事件处理器，以保持它与**来自**文本框同步。

    +   当连接从服务器端中心接收到`ReceiveMessage`调用时，它会在`messages`项目符号列表中添加一个列表项元素。列表项的内容包含消息的详细信息，如`from`、`to`和`body`。对于我们在 C#中定义的两个模型，请注意，JavaScript 使用 camelCase，而 C#使用 PascalCase。

    消息使用 JavaScript 插值`string`格式化。此功能需要在`string`值的开始和结束处使用反引号`` ` ``，并使用花括号`${}`来表示动态占位符。

    +   为**注册用户**按钮添加了一个`click`事件处理器，该处理器创建一个包含用户名和其群组的注册模型，然后在服务器端调用`Register`方法。

    +   为**发送消息**按钮添加了一个`click`事件处理器，该处理器创建一个包含`from`、`to`和`body`的消息模型，然后在服务器端调用`SendMessage`方法。

## 测试聊天功能

现在我们已经准备好尝试在多个网站访客之间发送聊天消息：

1.  使用`https`配置文件启动`Northwind.SignalR.Service.Client.Mvc`项目网站：

    +   如果你使用 Visual Studio 2022，则在工具栏中选择**https**配置文件，然后在不进行调试的情况下启动`Northwind.SignalR.Service.Client.Mvc`项目。

    +   如果你使用 Visual Studio Code，则在命令提示符或终端中输入以下命令：

        ```cs
        dotnet run --launch-profile https 
        ```

    +   在 Windows 上，如果 Windows Defender 防火墙阻止访问，则点击**允许访问**。

1.  启动 Chrome 并导航到`https://localhost:5111/`。

1.  注意，`Alice`的名字已经输入，`Sales,IT`群组也已经输入。点击**注册用户**，注意**SignalR 聊天**返回的响应，如图*图 11.1*所示：

![](img/B19587_11_01.png)

图 11.1：在聊天中注册新用户

1.  打开一个新的 Chrome 窗口或启动另一个浏览器，如 Firefox 或 Edge。

1.  导航到`https://localhost:5111/`。

1.  输入`Bob`作为姓名，`Sales`作为他的组，然后点击**注册用户**。

1.  打开一个新的 Chrome 窗口或启动另一个浏览器，如 Firefox 或 Edge。

1.  导航到`https://localhost:5111/`。

1.  输入`Charlie`作为姓名，`IT`作为他的组，然后点击**注册用户**。

1.  调整浏览器窗口，以便可以同时看到所有三个窗口。

    PowerToys 及其 FancyZones 功能是管理窗口的绝佳工具。更多信息请访问以下链接：[`learn.microsoft.com/en-us/windows/powertoys/`](https://learn.microsoft.com/en-us/windows/powertoys/)。

1.  在 Alice 的浏览器中输入以下内容：

    +   **收件人**：`Sales`

    +   **正文**：`卖更多！`

1.  点击**发送消息**。

1.  注意，Alice 和 Bob 收到了消息，如图 11.2 所示：

![](img/B19587_11_02.png)

图 11.2：Alice 向 Sales 组发送消息

1.  在 Bob 的浏览器中输入以下内容：

    +   **收件人**：`IT`

    +   **正文**：`修复更多错误！`

1.  点击**发送消息**。

1.  注意，Alice 和 Charlie 收到了消息，如图 11.3 所示：

![](img/B19587_11_03.png)

图 11.3：Bob 向 IT 组发送消息

1.  在 Alice 的浏览器中输入以下内容：

    +   **收件人**：`Bob`

    +   **正文**：`你好，Bob！`

1.  点击**发送消息**。

1.  注意，只有 Bob 收到了消息。

1.  在 Charlie 的浏览器中输入以下内容：

    +   **收件人**：留空。

    +   **正文**：`现在大家都跳舞吧！`

1.  点击**发送消息**。

1.  注意，每个人都收到了消息，如图 11.4 所示：

![](img/B19587_11_04.png)

图 11.4：Charlie 向所有人发送消息

1.  在 Charlie 的浏览器中输入以下内容：

    +   **收件人**：`HR,Alice`

    +   **正文**：`有人正在听 HR 吗？`

1.  点击**发送消息**。

1.  注意，Alice 收到了直接发送给她的消息，但由于 HR 组不存在，没有收到发送给该组的消息，如图 11.5 所示：

![](img/B19587_11_05.png)

图 11.5：Charlie 向 Alice 和一个不存在的组发送消息

1.  关闭浏览器并关闭 Web 服务器。

# 构建.NET 控制台应用程序客户端

您刚刚看到了一个.NET 服务托管 SignalR 中心，以及一个 JavaScript 客户端通过该 SignalR 中心与其他客户端交换消息。现在，让我们创建一个.NET 客户端用于 SignalR。

## 创建 SignalR 的.NET 客户端

我们将使用控制台应用程序，尽管任何.NET 项目类型都需要相同的包引用和实现代码：

1.  使用您首选的代码编辑器添加一个新项目，如下所示：

    +   项目模板：**控制台应用程序** / `console`

    +   解决方案文件和文件夹：`Chapter11`

    +   项目文件和文件夹：`Northwind.SignalR.Client.Console`

1.  为 ASP.NET Core SignalR 客户端添加包引用，并为`Northwind.Common`添加项目引用，将警告视为错误，并在全局和静态导入`System.Console`类，如下所示：

    ```cs
    <ItemGroup>
      <PackageReference Include="Microsoft.AspNetCore.SignalR.Client" 
                        Version="8.0.0" />
    </ItemGroup>
    <ItemGroup>
      <ProjectReference 
        Include="..\Northwind.Common\Northwind.Common.csproj" />
    </ItemGroup> 
    ```

1.  构建项目以恢复包并构建引用的项目。

1.  在 `Program.cs` 中，删除现有的语句，导入用于作为客户端处理 SignalR 和聊天模型的命名空间，然后添加语句提示用户输入用户名和要注册的组，创建中心连接，并最终监听接收到的消息，如下面的代码所示：

    ```cs
    using Microsoft.AspNetCore.SignalR.Client; // To use HubConnection.
    using Northwind.Chat.Models; // To use UserModel, MessageModel.
    Write("Enter a username (required): ");
    string? username = ReadLine();
    if (string.IsNullOrEmpty(username))
    {
      WriteLine("You must enter a username to register with chat!");
      return;
    }
    Write("Enter your groups (optional): ");
    string? groups = ReadLine();
    HubConnection hubConnection = new HubConnectionBuilder()
      .WithUrl("https://localhost:5111/chat")
      .Build();
    hubConnection.On<MessageModel>("ReceiveMessage", message =>
    {
      WriteLine($"To {message.To}, From {message.From}: {message.Body}");
    });
    await hubConnection.StartAsync();
    WriteLine("Successfully started.");
    UserModel registration = new()
    {
      Name = username,
      Groups = groups
    };
    await hubConnection.InvokeAsync("Register", registration);
    WriteLine("Successfully registered.");
    WriteLine("Listening... (press ENTER to stop.)");
    ReadLine(); 
    ```

## 测试 .NET 控制台应用程序客户端

让我们启动 SignalR 服务，并在控制台应用程序中调用它：

1.  使用 `https` 配置文件（不进行调试）启动 `Northwind.SignalR.Service.Client.Mvc` 项目的网站。

1.  启动 Chrome 并导航到 `https://localhost:5111/`。

1.  点击 **注册用户**。

1.  启动 `Northwind.SignalR.Client.Console` 项目。

1.  输入你的名字和组：`Sales,Admins`。

1.  将浏览器和控制台应用程序窗口排列好，以便你可以同时看到它们。

1.  在 Alice 的浏览器中输入以下内容：

    +   **发送到**：`Sales`

    +   **正文**：`Go team!`

1.  **点击** **发送消息**，并注意 Alice 和你都能收到消息，如图 *11.6* 所示：

![](img/B19587_11_06.png)

图 11.6：Alice 在控制台应用程序中向销售团队发送消息，包括一个用户

1.  在控制台应用程序中，按 *Enter* 键停止监听。

1.  关闭 Chrome 并关闭 Web 服务器。

# 使用 SignalR 进行数据流

到目前为止，我们已经看到了 SignalR 如何向一个或多个客户端广播结构化消息。这对于相对较小且结构化的数据，并且完全存在于某个时间点的情况效果很好。但是，对于随着时间的推移分批到达的数据怎么办呢？

**流**可以用于这些场景。SignalR 支持服务到客户端（从流中下载数据）和客户端到服务（将数据上传到流）。

要启用下载流，中心方法必须返回 `IAsyncEnumerable<T>`（仅支持 C# 8 或更高版本）或 `ChannelReader<T>`。

要启用上传流，中心方法必须接受类型为 `IAsyncEnumerable<T>`（仅支持 C# 8 或更高版本）或 `ChannelReader<T>` 的参数。

## 定义用于流的中心

让我们添加一些流方法来看看它们在实际操作中的工作情况：

1.  在 `Northwind.Common` 项目中，添加一个名为 `StockPrice.cs` 的新文件，并修改其内容以定义股票价格数据的 `record`，如下面的代码所示：

    ```cs
    namespace Northwind.SignalR.Streams;
    public record StockPrice(string Stock, double Price); 
    ```

1.  构建 `Northwind.SignalR.Service.Client.Mvc` 项目以更新其引用的项目。

1.  在 `Northwind.SignalR.Service.Client.Mvc` 项目中，添加一个名为 `StockPriceHub.cs` 的新类，并修改其内容以定义一个具有两个流方法的中心，如下面的代码所示：

    ```cs
    using Microsoft.AspNetCore.SignalR; // To use Hub.
    using System.Runtime.CompilerServices; // To use [EnumeratorCancellation].
    using Northwind.SignalR.Streams; // To use StockPrice.
    namespace Northwind.SignalR.Service.Hubs;
    public class StockPriceHub : Hub
    {
      public async IAsyncEnumerable<StockPrice> GetStockPriceUpdates(
        string stock,
        [EnumeratorCancellation] CancellationToken cancellationToken)
      {
        double currentPrice = 267.10; // Simulated initial price.
        for (int i = 0; i < 10; i++)
        {
          // Check the cancellation token regularly so that the server will stop
          // producing items if the client disconnects.
          cancellationToken.ThrowIfCancellationRequested();
          // Increment or decrement the current price by a random amount.
          // The compiler does not need the extra parentheses but it
          // is clearer for humans if you put them in.
          currentPrice += (Random.Shared.NextDouble() * 10.0) - 5.0;
          StockPrice stockPrice = new(stock, currentPrice);
         Console.WriteLine("[{0}] {1} at {2:C}",
           DateTime.UtcNow, stockPrice.Stock, stockPrice.Price);
          yield return stockPrice;
          await Task.Delay(4000, cancellationToken); // milliseconds
        }
      }
      public async Task UploadStocks(IAsyncEnumerable<string> stocks)
      {
        await foreach (string stock in stocks)
        {
          Console.WriteLine($"Receiving {stock} from client...");
        }
      }
    } 
    ```

1.  在 `Northwind.SignalR.Service.Client.Mvc` 项目的 `Program.cs` 中，在注册聊天中心的语句之后注册股票价格中心，如下面的代码所示：

    ```cs
    `app.MapHub<StockPriceHub>("/stockprice");` 
    ```

## 创建用于流传输的 .NET 控制台应用程序客户端

现在，我们可以创建一个简单的客户端来从 SignalR 中心下载数据流并将其上传到 SignalR 中心：

1.  使用你喜欢的代码编辑器添加一个新项目，如下面的列表所示：

    +   项目模板：**控制台应用程序** / `console`

    +   解决方案文件和文件夹：`Chapter11`

    +   项目文件和文件夹：`Northwind.SignalR.Client.Console.Streams`

1.  在 `Northwind.SignalR.Client.Console.Streams` 项目文件中，将警告视为错误，添加 ASP.NET Core SignalR 客户端包引用，添加对 `Northwind.Common` 的项目引用，并全局和静态导入 `System.Console` 类，如下所示突出显示的标记：

    ```cs
    <ItemGroup>
      <PackageReference Include="Microsoft.AspNetCore.SignalR.Client" 
                        Version="8.0.0" />
    </ItemGroup>
    <ItemGroup>
      <ProjectReference 
        Include="..\Northwind.Common\Northwind.Common.csproj" />
    </ItemGroup> 
    ```

1.  在 `Northwind.SignalR.Client.Console.Streams` 项目中，添加一个名为 `Program.Methods.cs` 的新类文件，并修改其内容以在部分 `Program` 类中定义静态方法，异步生成十个四字母的随机股票代码，如下所示代码：

    ```cs
    // Defined in the empty default namespace to merge with the auto-
    // generated partial Program class.
    partial class Program
    {
      static async IAsyncEnumerable<string> GetStocksAsync()
      {
        for (int i = 0; i < 10; i++)
        {
          // Return a random four-letter stock code.
          yield return $"{AtoZ()}{AtoZ()}{AtoZ()}{AtoZ()}";
          await Task.Delay(TimeSpan.FromSeconds(3));
        }
      }
      static string AtoZ()
      {
        return char.ConvertFromUtf32(Random.Shared.Next(65, 91));
      }
    } 
    ```

1.  在 `Northwind.SignalR.Client.Console.Streams` 项目中，在 `Program.cs` 中删除现有语句。导入用于作为客户端处理 SignalR 的命名空间，然后添加提示用户输入股票、创建 Hub 连接、监听接收到的股票价格流，并将异步股票流发送到服务的语句，如下所示代码：

    ```cs
    using Microsoft.AspNetCore.SignalR.Client; // To use HubConnection.
    using Northwind.SignalR.Streams; // To use StockPrice.
    Write("Enter a stock (press Enter for MSFT): ");
    string? stock = ReadLine();
    if (string.IsNullOrEmpty(stock))
    {
      stock = "MSFT";
    }
    HubConnection hubConnection = new HubConnectionBuilder()
      .WithUrl("https://localhost:5111/stockprice")
      .Build();
    await hubConnection.StartAsync();
    try
    {
      CancellationTokenSource cts = new();
      IAsyncEnumerable<StockPrice> stockPrices = 
        hubConnection.StreamAsync<StockPrice>(
          "GetStockPriceUpdates", stock, cts.Token);
      await foreach (StockPrice sp in stockPrices)
      {
        WriteLine($"{sp.Stock} is now {sp.Price:C}.");
        Write("Do you want to cancel (y/n)? ");
        ConsoleKey key = ReadKey().Key;
        if (key == ConsoleKey.Y)
        {
          cts.Cancel();
        }
        WriteLine();
      }
    }
    catch (Exception ex)
    {
      WriteLine($"{ex.GetType()} says {ex.Message}");
    }
    WriteLine();
    WriteLine("Streaming download completed.");
    await hubConnection.SendAsync("UploadStocks", GetStocksAsync());
    WriteLine("Uploading stocks to service... (press ENTER to stop.)");
    ReadLine();
    WriteLine("Ending console app."); 
    ```

## 测试流服务客户端

最后，我们可以测试流数据功能：

1.  使用 `https` 配置不带调试启动 `Northwind.SignalR.Service.Client.Mvc` 项目网站。

1.  不带调试启动 `Northwind.SignalR.Client.Console.Streams` 控制台应用程序。

1.  将 ASP.NET Core MVC 网站和客户端控制台应用程序的控制台窗口排列在一起，以便你可以并排看到它们。

1.  在客户端控制台应用程序中，按 *Enter* 使用微软股票代码，如下所示输出：

    ```cs
    Enter a stock (press Enter for MSFT):
    MSFT is now £265.00.
    Do you want to cancel (y/n)? 
    ```

1.  在网站控制台窗口中等待大约十秒钟，并注意在服务中已生成但尚未发送给客户端的几个股票价格，如下所示输出：

    ```cs
    info: Microsoft.Hosting.Lifetime[14]
          Now listening on: https://localhost:5131
    info: Microsoft.Hosting.Lifetime[14]
          Now listening on: http://localhost:5132
    info: Microsoft.Hosting.Lifetime[0]
          Application started. Press Ctrl+C to shut down.
    info: Microsoft.Hosting.Lifetime[0]
          Hosting environment: Development
    info: Microsoft.Hosting.Lifetime[0]
          Content root path: C:\apps-services-net7\Chapter13\Northwind.SignalR.Service.Client.Mvc
    [12/09/2022 17:52:26] MSFT at £265.00
    [12/09/2022 17:52:30] MSFT at £260.78
    [12/09/2022 17:52:34] MSFT at £264.86
    [12/09/2022 17:52:38] MSFT at £262.10 
    ```

1.  在客户端控制台应用程序中，按 *n* 接收下一个更新的价格。继续按 *n* 直到服务发送价格并由客户端读取，然后按 *y*，注意 SignalR 服务收到一个取消令牌所以停止，客户端现在开始上传股票，如下所示输出：

    ```cs
    MSFT is now £260.78.
    Do you want to cancel (y/n)? n
    MSFT is now £264.86.
    Do you want to cancel (y/n)? n
    MSFT is now £262.10.
    Do you want to cancel (y/n)? y
    System.Threading.Tasks.TaskCanceledException says A task was canceled.
    Streaming download completed.
    Uploading stocks to service... (press ENTER to stop.) 
    ```

1.  在网站控制台窗口中，注意接收到了随机股票代码，如下所示输出：

    ```cs
    Receiving PJON from client...
    Receiving VWJD from client...
    Receiving HMOJ from client...
    Receiving QQMQ from client... 
    ```

1.  关闭两个控制台窗口。

# 练习和探索

通过回答一些问题、进行一些实际操作和深入探索本章主题来测试你的知识和理解。

## 练习 11.1 – 测试你的知识

回答以下问题：

1.  SignalR 使用哪些传输方式，默认的是哪种？

1.  RPC 方法签名设计的好习惯是什么？

1.  你可以使用什么工具下载 SignalR JavaScript 库？

1.  如果你向一个连接 ID 不存在的客户端发送 SignalR 消息会发生什么？

1.  将 SignalR 服务与其他 ASP.NET Core 组件分离有什么好处？

## 练习 11.2 – 探索主题

使用以下页面上的链接了解更多关于本章涵盖主题的详细信息：

[`github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-11---broadcasting-real-time-communication-using-signalr`](https://github.com/markjprice/apps-services-net8/blob/main/docs/book-links.md#chapter-11---broadcasting-real-time-communication-using-signalr)

# 摘要

在本章中，你学习了以下内容：

+   SignalR 之前的技术历史。

+   支撑 SignalR 的概念和技术。

+   使用 SignalR 实现聊天功能，包括在网站项目中构建一个 hub，以及使用 JavaScript 和 .NET 控制台应用程序的客户端。

+   使用 SignalR 下载和上传数据流。

在下一章中，你将了解 GraphQL，这是另一个允许客户端控制从服务返回的数据的标准。
