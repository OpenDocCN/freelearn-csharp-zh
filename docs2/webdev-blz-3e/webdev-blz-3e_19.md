

# 从哪里开始

这本书即将结束，我想让你记住我们在 Blazor 预览阶段以来在生产环境中遇到的一些事情。我们还将讨论从这里开始的方向。

在本章中，我们将涵盖以下主题：

+   运行 Blazor 的生产经验总结

+   下一步

# 运行 Blazor 的生产经验总结

由于 Blazor 处于预览阶段，我们一直在生产环境中运行 Blazor 服务器。在大多数情况下，一切运行良好，没有问题。偶尔，我们遇到了一些问题，我将在本节中与您分享我们的经验总结。

我们将查看以下内容：

+   解决内存问题

+   解决并发问题

+   解决错误

+   旧浏览器

这些是我们遇到的一些问题，我们已经以对我们有效的方式解决了它们。

## 解决内存问题

我们最新的升级增加了许多用户，随之而来的是服务器负载的增加。服务器管理内存相当好，但在这个版本中，后端系统有点慢，所以用户会按 *F5* 重新加载页面。然后，电路会断开，并创建一个新的电路。旧的电路会等待用户再次连接到服务器，默认时间为 3 分钟。

然后，用户将有一个新的电路，并且永远不会再次连接到旧的电路，但在这三分钟内，用户的会话状态仍然会占用内存。这可能对大多数应用程序来说不是问题，但我们正在将大量数据加载到内存中——数据、渲染树以及与之相关的所有内容都将保留在内存中。

那么，我们能从中学到什么呢？Blazor 是一个单页应用程序。重新加载页面就像重启应用程序一样，这意味着我们应该始终确保添加从页面内部更新数据的能力（如果这对应用程序有意义）。我们也可以像在 *第十一章*，*管理状态 – 第二部分* 中所做的那样，在数据变化时更新数据。

在我们这个案例中，我们给服务器增加了更多内存，并确保在用户界面中有重新加载按钮，这些按钮可以刷新数据而不重新加载整个页面。最终目标是添加实时更新，当数据发生变化时，持续更新用户界面。

如果增加服务器的内存不是一个选择，我们可以尝试将垃圾回收从服务器改为桌面。.NET 垃圾回收有两个模式：

+   **工作站**模式针对在通常没有很多内存的工作站上运行进行了优化。它每秒运行垃圾回收多次。

+   **服务器**模式针对通常有很多内存的服务器进行了优化，优先考虑速度，这意味着它每 2 秒只会运行垃圾回收器一次。

垃圾回收器的模式可以在项目文件或 `runtimeconfig.json` 文件中通过更改 `ServerGarbageCollection` 节点来设置：

```cs
<PropertyGroup>
<ServerGarbageCollection>true</ServerGarbageCollection>
</PropertyGroup> 
```

虽然增加内存可能是一个更好的主意。

我们也注意到了处理我们的数据库上下文的重要性。请确保使用 `IDbContextFactory` 创建数据上下文的实例，并在我们完成时，通过使用 `Using` 关键字来释放它。

然后，数据上下文将只短暂可用，然后被释放，快速释放内存。

## 解决并发问题

我们经常遇到数据上下文已经被使用，无法从两个不同的线程访问数据库的问题。

这可以通过使用 `IDbContextFactory` 并在我们完成使用数据上下文后释放它来解决。

在非 Blazor 网站中，同时加载多个组件永远不会是问题（因为网页一次只做一件事），所以 Blazor 能够同时做很多事情，这是我们设计架构时需要考虑的。

## 解决错误

Blazor 通常会给出易于理解的错误，但在一些罕见的情况下，我们确实会遇到难以解决的问题。我们可以通过在 `Startup.cs` 中添加以下选项来向我们的电路（对于 Blazor Server）添加详细的错误：

```cs
services.AddServerSideBlazor().AddCircuitOptions(options => { options.DetailedErrors = true; }); 
```

通过这样做，我们将获得更详细的错误。然而，我不建议在生产环境中使用详细的错误。话虽如此，我们在生产中的内部应用程序中打开了此设置，因为内部用户已经接受了培训并了解如何处理它。这使得我们更容易帮助用户，并且错误消息仅在网页浏览器的开发者工具中可见，而不是在用户界面上可见。

## 旧浏览器

一些客户在旧系统上运行旧浏览器，尽管 Blazor 支持所有主流浏览器，但这种支持不包括真正旧的浏览器。我们最终帮助这些客户升级到 Edge 或 Chrome，仅仅是因为我们认为他们不应该使用不再接收安全补丁的浏览器浏览网页。

即使是我们家里的电视也能运行 Blazor WebAssembly，所以旧浏览器可能不是大问题，但在考虑浏览器支持时，这值得思考。我们需要/想要支持哪些浏览器？

# 下一步

到这个时候，我们已经知道了 Blazor Server 和 Blazor WebAssembly 之间的区别，也知道何时选择什么，选择其中一个并不是真的那么重要。我们知道如何创建可重用组件，制作 API，管理状态，等等。但接下来我们该怎么办？下一步是什么？

## 社区

Blazor 社区不如其他框架那么大，但发展迅速。许多人通过博客或视频与社区分享内容。YouTube 和 PluralSight 有很多教程和课程。Twitch 上有越来越多的 Blazor 内容，但在庞大的内容目录中并不总是容易找到。

有许多值得提及的资源：

+   **吉米·恩斯特罗姆** – 如果我没有把我自己列入我的名单，我就不是一个大大的 Blazor 爱好者。我谈论 Blazor，时不时地来点双关语。当我们直播时，我们使用 CodingAfterWork（见下文）。我的博客有很多 Blazor 内容，还有更多即将到来：[`engstromjimmy.com/`](https://engstromjimmy.com/). X: `@EngstromJimmy`.

+   我们编写的**Blazm**组件库可以在[`blazm.net/`](http://blazm.net/)找到。虽然有许多更好的网格组件，但这也展示了网格组件既简单又复杂。

+   **工作后编码**在我们的播客和直播中有很多关于 Blazor 的内容；请在我们的社交媒体上关注我们：[`codingafterwork.com/FindUs`](http://codingafterwork.com/FindUs).

+   **丹尼尔·罗思**是 Blazor 的产品经理。他非常值得聆听，曾作为嘉宾出现在我们的播客中。在 YouTube 上搜索他。X: `@danroth27`.

+   **史蒂夫·桑德森**是 Blazor 的发明者；他绝对值得关注。他在演讲中继续做一些开创性的工作；在 YouTube 上搜索他。确保观看他在 NDC 奥斯陆的演讲，他在那里首次展示了 Blazor。X: `@stevensanderson`.

+   **Awesome-Blazor**有一个包含大量 Blazor 相关链接和资源的巨大列表，可以在[`github.com/AdrienTorris/awesome-blazor`](https://github.com/AdrienTorris/awesome-blazor)找到。

+   **杰夫·弗里茨**在 Twitch 上分享 Blazor 知识（以及其他内容）：[`www.twitch.tv/csharpfritz`](https://www.twitch.tv/csharpfritz). X: `@csharpfritz`.

+   **克里斯·圣蒂**是一位同行作者，他为 Blazor 制作了许多真正令人惊叹的包。他在博客上有很多内容：[`chrissainty.com/`](https://chrissainty.com/). X: `@chris_sainty`.

+   **卡尔·富兰克林**在[`BlazorTrain.com/`](https://BlazorTrain.com/)上制作了许多 Blazor 视频。X: `@carlfranklin`.

+   **约翰·希尔顿**有很多 Blazor 内容。你可以在[`jonhilton.net/`](https://jonhilton.net/)找到他。X: `@jonhilt`.

+   **帕特里克·戈德**在他的 YouTube 频道上有许多精彩内容：`www.youtube.com/@PatrickGod`. X: `@_PatrickGod`.

+   **大卫·派恩**是一位同行作者，也是 Blazorators 的创造者，可以在[`github.com/IEvangelist/blazorators`](https://github.com/IEvangelist/blazorators)找到他。X: `@davidpine7`.

+   **彼得·莫里斯**是 Fluxor 的创造者，是一位值得关注的伟大人物。X: `@MrPeterLMorris`

+   **迈克尔·华盛顿**是一位同行作者，我们可以在[`adefwebserver.com/`](https://adefwebserver.com/)找到他。X: `@ADefWebserver`.

+   **Ed Charbeneau** 总是能提供优质的内容。请务必关注他。[`edcharbeneau.com/`](https://edcharbeneau.com/) [`www.twitch.tv/edcharbeneau`](https://www.twitch.tv/edcharbeneau)，[`www.youtube.com/edwardcharbeneau`](https://www.youtube.com/edwardcharbeneau)，[`www.twitch.tv/codeitlive`](https://www.twitch.tv/codeitlive)，`www.youtube.com/@telerik`。X: `@EdCharbeneau`。

+   **Eric Johansson** 是 Twitch 上的常客，展示他的项目并将他的 .NET Framework 应用现代化到一个更现代的平台 [`www.twitch.tv/thindal`](https://www.twitch.tv/thindal) X: `@EricJohansson`。

+   **Egil Hansen** 是 bUnit 的创造者。我们可以在以下链接找到他：[`egilhansen.com/about/`](https://egilhansen.com/about/)。X: `@egilhansen`。

+   **Sam Basu** 是关注 .NET MAUI 内容时的一个优秀人选。[`www.twitch.tv/codeitlive`](https://www.twitch.tv/codeitlive) 和 `www.youtube.com/@telerik`。X: `@samidip`。

+   **Junichi Sakamoto** 制作了大量的出色 Blazor 库，从连接游戏手柄到翻译和预渲染，应有尽有。你可以在以下链接找到他的项目：[`github.com/jsakamoto`](https://github.com/jsakamoto)。X: `@jsakamoto`。

+   **Blazor University** 提供了大量的培训材料，是学习更多知识的绝佳资源：[`blazor-university.com/`](https://blazor-university.com/)。

+   **Gerald Versluis** 在他的 YouTube 频道上提供了大量与各种 .NET 相关的内容：[`youtube.com/GeraldVersluis`](https://youtube.com/GeraldVersluis)。X: `@jfversluis`。

+   **Maddy Montaquilla** 的视频非常值得观看；在 YouTube 上搜索她来观看她的视频。X: `@maddymontaquila`。

+   **James Montemagno** 拥有一个内容丰富的 YouTube 频道，其中包含大量的 .NET MAUI 内容：[`www.youtube.com/JamesMontemagno`](https://www.youtube.com/JamesMontemagno)。X: `@JamesMontemagno`。

+   **Daniel Hindrikes** 在这个 YouTube 频道上提供了一些关于 .NET MAUI 的优质内容：`www.youtube.com/@DanielHindrikes`。X: `@hindrikes`。

## 组件

大多数第三方组件供应商，如 Progress Telerik、DevExpress、Syncfusion、Radzen、ComponentOne 以及更多，都投资了 Blazor。有些需要付费，有些是免费的。还有很多开源组件库供我们使用。

这个问题经常出现：*我是 Blazor 的初学者。我应该使用哪个第三方供应商？* 我的建议是在投资库（无论是金钱还是时间）之前，先尝试弄清楚你需要什么。

许多供应商可以完成我们所需要的一切，但在某些情况下，使应用程序工作可能需要更多的努力。我们开始自己开发一个网格组件，过了一段时间后，我们决定将其开源。

这就是 Blazm 的诞生。我们有一些特殊的要求（并不复杂），但它们要求我们反复编写大量代码，以便在第三方供应商组件中使其工作。

通过编写我们的组件，我们学到了很多，这实际上是非常容易做到的。我的建议并不总是要编写自己的组件。专注于你试图解决的真正业务问题会更好。

对于我们来说，构建一个相当高级的网格组件教会了我们很多关于 Blazor 内部工作原理的知识。

想想你需要什么，尝试不同的供应商以查看哪个最适合你。也许一开始自己构建组件会更好，这样你可以更多地了解 Blazor。

但始终关注你的代码。如果你重复相同的代码，将其封装在组件中。始终思考：*这能成为一个可重用的组件吗？*

我们目前使用一个组件供应商，但我们把所有组件都封装在我们自己的一个组件中。这样，设置默认值和添加适合我们的逻辑就变得很容易，就像我们在整本书中学到的那样。

# 摘要

在本章中，我们检查了在生产环境中运行 Blazor 时遇到的一些挑战，并讨论了下一步该怎么做。

在整本书中，我们学习了 Blazor 的工作原理以及如何创建基本和高级组件。我们实现了使用认证和授权的安全功能。我们创建并使用了连接到“数据库”的 API。

我们进行了 JavaScript 调用和实时更新。我们调试了我们的应用程序并测试了我们的代码，最后但同样重要的是，我们探讨了如何部署到生产环境。

现在，我们已经准备好将所有这些知识应用到下一个冒险，另一个应用程序中。我希望你在阅读这本书的过程中和我写作这本书一样感到乐趣。成为 Blazor 社区的一员非常有趣，我们每天都在学习新事物。

感谢您阅读这本书。请保持联系。我很乐意了解您构建的东西！

*欢迎加入 Blazor 社区！*

# 加入我们的社区 Discord

加入我们的社区 Discord 空间，与作者和其他读者进行讨论：

[`packt.link/WebDevBlazor3e`](https://packt.link/WebDevBlazor3e)

![](img/QR_Code2668029180838459906.png)
