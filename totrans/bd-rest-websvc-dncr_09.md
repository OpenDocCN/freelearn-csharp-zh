# 扩展RESTful服务（Web服务的性能）

在网络世界中，每个人要么在编写，要么在寻找一个网络应用程序。随着需求的增加，每个网络应用程序都需要能够服务更多的请求——有时每天成千上万的请求。因此，应用程序应该编写成能够处理这些大量请求。

例如，假设你是一个开发和支持团队的一员，负责开发公司的旗舰产品FlixOne Store。这个产品很受欢迎，并且获得了关注，导致你的电子商务网站（FlixOne）被消费者流量淹没。你系统中的支付服务很慢，这几乎使整个系统崩溃，导致你失去了客户。虽然这是一个虚构的场景，但在现实生活中确实可能发生，并可能导致业务损失。为了避免这种情况，你应该考虑FlixOne应用程序的可扩展性。

可扩展性是关键系统最重要的非功能性需求之一。为几百个用户提供数百笔交易与为几百万用户提供几百万笔交易的服务并不相同。

在本章中，我们将讨论一般性的可扩展性。我们还将讨论如何扩展RESTful服务，设计它们时需要考虑的因素，以及如何使用不同的模式来避免级联故障，包括技术、库和工具，这些也可以帮助我们的常规应用程序。

到本章结束时，你将了解以下内容：

+   集群

+   负载均衡

+   可扩展性简介

# 集群

集群是在多个服务器上提供相同服务的一种方式。随着更多服务器的添加，你可以避免不受控制的情况，例如故障转移、系统崩溃等。在数据库的上下文中，集群指的是几个服务器实例能够连接到单个服务器的能力。容错性和负载均衡是集群的两个主要优点。

# 负载均衡

在集群中，负载均衡器是一个有用的工具。你可以定义**负载均衡**为一个帮助在集群服务器内部和之间分配网络或应用程序流量的设备，并提高应用程序的响应性。

在实现中，负载均衡器被放置在客户端和服务器之间。它有助于在多个服务器之间平衡多个应用程序请求。换句话说，负载均衡器减少了单个服务器的时间并防止应用程序服务器故障。

# 它是如何工作的？

负载均衡器的工作是确保应用程序的服务器可用。如果一个应用程序的服务器不可用，负载均衡器将所有新的请求重定向到可用的服务器，如下面的图示所示：

![集群图](img/867aa115-47b5-4579-8b1b-e94c3e9f7bc8.png)

在前面的图中，您可以看到负载均衡器在其典型环境中，系统通过互联网从不同来源接受多个请求，然后由负载均衡器从多个服务器进行管理。

在.NET中，这种配置也被称为Web农场 ([https://www.codeproject.com/Articles/114910/What-is-the-difference-between-Web-Farm-and-Web-Ga](https://www.codeproject.com/Articles/114910/What-is-the-difference-between-Web-Farm-and-Web-Ga))。

负载均衡器使用各种算法，也称为负载均衡方法：最少连接方法、轮询方法、最少响应时间方法、最少带宽方法、最少数据包方法、自定义负载方法等。

负载均衡器在应用程序的可扩展性中扮演着重要角色，因为它确保应用程序的服务器能够处理服务器请求。请注意，您可能需要在代码不变的情况下安排您的硬件基础设施以适应负载均衡器（尽管，有些情况下可能需要代码更改）。市场上有很多负载均衡器，例如Incapsula ([https://www.incapsula.com/](https://www.incapsula.com/))、F5 ([https://www.f5.com/](https://www.f5.com/))、Citrix Netscaler ([https://www.citrix.com/](https://www.citrix.com/))、Dyn ([https://dyn.com/](https://dyn.com/))）、Amazon Elastic Load Balancing和Amazon ELB ([https://aws.amazon.com/](https://aws.amazon.com/))。

在接下来的部分中，我们将探讨您可以扩展系统的不同方法。

# 可扩展性简介

每个应用程序都有其自己的服务请求能力。应用程序的能力指的是其性能以及当负载增加时如何满足其目标。

许多Web应用程序将此称为在规定时间内的请求数量。

在设计您的Web应用程序时做出正确的设计决策非常重要；设计决策会影响您服务的可扩展性。确保找到正确的平衡点，以便您的方案不仅考虑服务，还要考虑其基础设施，以及任何扩展需求。

性能和可扩展性是系统的两个不同特性。性能涉及系统的吞吐量，而可扩展性涉及为更多用户或更多交易处理所需吞吐量。

# 向内扩展（垂直扩展）

**向内扩展**或**向上扩展**（也称为**垂直扩展**）是通过向同一机器添加更多资源（如内存或更快的处理器）来实现可扩展性的方法。这并不总是适用于所有应用程序，因为成本也是考虑垂直扩展时的一个因素。

您也可以升级资源或硬件，而不是向机器添加新资源。例如，如果您有8 GB的RAM，您可以将其升级到16 GB，同样的情况也适用于处理器和其他资源。不幸的是，随着硬件的升级，机器的可扩展性有一定的限制。这可能会导致仅仅是将瓶颈转移，而不是解决提高可扩展性的真正问题。

您也可以将应用程序迁移到完全不同的机器上，例如简单地迁移到更强大的MacOS，例如。

垂直扩展不涉及任何代码更改，因此这是一个简单的任务，但它涉及额外的成本，因为这是一种相当昂贵的技巧。Stack Overflow是那些罕见的基于.NET的系统之一，它进行了垂直扩展。

# 扩展（横向扩展）

扩展（向上扩展）、扩展（向外扩展）或横向扩展是通过添加更多服务器或节点来服务请求，而不是资源。如果您不想扩展应用程序，总有一种方法可以将其扩展。

当应用程序代码不依赖于其运行的服务器时，横向扩展是一种成功的策略。然而，如果需要在一个特定的服务器上执行请求，即如果应用程序代码具有服务器亲和性，那么横向扩展将会很困难。在无状态代码的情况下，在任何服务器上执行都更容易。因此，当无状态代码在横向扩展的机器或集群上运行时，可扩展性得到了提高。由于横向扩展的性质，这在整个行业中是一种常用的方法。有许多大型可扩展系统以这种方式管理，例如Google、Amazon和Microsoft。

# 线性可扩展性

线性可扩展性指的是应用阿姆达尔定律（[https://en.wikipedia.org/wiki/Amdahl%27s_law](https://en.wikipedia.org/wiki/Amdahl%27s_law)）来垂直扩展应用程序。在这里，您也可以考虑并行计算。

并行计算是一种计算架构，它表明通过执行多个处理器来实现同时处理。

线性可扩展性在您的应用程序中的好处包括：

+   不需要代码更改

+   可以轻松添加额外资源

+   存在物理可用性

# 分布式缓存

通过分布式缓存技术，我们可以提高我们RESTful Web服务的可扩展性（Web API）。分布式缓存可以存储在集群的多个节点上。分布式缓存提高了Web服务的吞吐量，因为缓存不再需要任何I/O跳转到任何外部资源。

这种方法有以下优点：

+   客户端获得相同的结果

+   分布式缓存由持久化存储支持，并作为一个不同的远程进程运行；即使应用程序服务器重新启动或出现任何问题，也不会影响缓存

+   源数据存储对其的请求较少

# 缓存持久化数据（数据层缓存）

与应用性能类似，你也应该考虑数据库性能。通过缓存持久化数据，在数据库中添加缓存层之后，你会获得更好的性能。这在应用中大量使用读取请求时也非常重要。现在，我们将以 EF Core 的缓存级别为例进行探讨。

# 一级缓存

这是一个由EF Core启用的内置会话缓存。从服务的第一个请求开始，从数据库中检索一个对象并将其存储在EF Core会话中。换句话说，EF Object Context和DbContext维护它们所管理的实体的状态信息。一旦上下文不再可用，其状态信息也随之消失。

# 二级缓存

对于主要采用分布式方式开发的应用或需要持久化数据的长时间运行请求，如Web应用，二级缓存非常重要。二级缓存存在于事务或应用范围之外，这些缓存对任何上下文或实例都可用。你可以使用应用提供的缓存机制，而不是编写自己的代码，例如Memcached。

# 应用缓存

应用缓存或应用层缓存有助于缓存应用中的任何对象。这进一步提高了应用的扩展性。在下一节中，我们将讨论可用的各种缓存机制。

# CacheCow

当你想在客户端和服务器上实现HTTP缓存时，CacheCow就派上用场。这是一个轻量级库，目前支持ASP.NET Web API。CacheCow是开源的，并附带MIT许可证，可在GitHub上找到（[https://github.com/aliostad/CacheCow](https://github.com/aliostad/CacheCow)）。

要开始使用CacheCow，你需要通过以下步骤为服务器和客户端做好准备：

1.  在你的ASP.NET Web API项目中安装`Install-Package CacheCow.Server` NuGet包；这将是你服务器。

1.  在你的客户端项目中安装`Install-Package CacheCow.Client` NuGet包；客户端应用将是WPF、Windows Form、控制台或任何其他Web应用。

1.  创建一个缓存存储。你需要创建一个

    存储

    在服务器端存储缓存元数据的缓存存储（[https://github.com/aliostad/CacheCow/wiki/Getting-started#cache-store](https://github.com/aliostad/CacheCow/wiki/Getting-started#cache-store)）。

# Memcached

Memcached是一个可定制的开源项目；你可以使用源代码，并根据你的需求对其进行添加和更新。Memcached由其官方页面（[https://memcached.org/](https://memcached.org/））定义为：

"一个用于存储来自数据库调用、API 调用或页面渲染结果的任意数据小块（字符串、对象）的内存中键值存储。"

参考以下链接获取完整教程[https://www.deanhume.com/memcached-for-c-a-walkthrough/](https://www.deanhume.com/memcached-for-c-a-walkthrough/)。

# Azure Redis Cache

Azure Redis Cache建立在名为Redis的开源存储之上([https://github.com/antirez/redis](https://github.com/antirez/redis))，这是一个内存数据库，数据持久化在磁盘上。根据Microsoft的描述([https://azure.microsoft.com/en-in/services/cache/](https://azure.microsoft.com/en-in/services/cache/))：

“Azure Redis Cache基于流行的开源Redis缓存。它为您提供了访问由Microsoft管理的安全、专用的Redis缓存，并且可以从Azure中的任何应用程序访问。”

如果您按照以下步骤操作，开始使用Azure Redis Cache非常简单：

1.  创建一个Web API项目。参考我们之前章节中的代码示例。

1.  实现Redis。参考点为[https://github.com/StackExchange/StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis)。同时，安装`Install-Package StackExchange.Redis` NuGet包。

1.  更新您的CacheConnection配置文件([https://docs.microsoft.com/en-us/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache#NuGet](https://docs.microsoft.com/en-us/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache#NuGet))。

1.  然后，在Azure上发布([https://docs.microsoft.com/en-us/azure/redis-cache/cache-web-app-howto#publish-and-run-in-azure](https://docs.microsoft.com/en-us/azure/redis-cache/cache-web-app-howto#publish-and-run-in-azure))。

# 通信（异步）

术语“通信”是自解释的；它是服务之间交互的行为。以下是一些例子：

+   在同一应用程序内部与另一个服务通信的服务

+   与应用程序外部（外部服务）的其他服务通信的服务

+   一个与组件（内部或外部）通信的服务

这种通信通过HTTP协议进行，消息或数据通过网络传输。

您应用程序的性能会影响服务之间如何通信。

异步通信是帮助扩展应用程序的方法之一。在ASP.NET Core中，我们可以通过使用异步HTTP调用（异步编程）来实现这一点：[https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/)

在处理异步通信时，你应该小心操作，例如，在添加带有图片的新产品时。系统被设计成可以创建不同尺寸的图片缩略图。这是一个耗时任务，如果处理不当可能会导致性能下降。从设计角度来看，异步操作在这种情况下是不可行的。在这里，你应该实现一个带有回调的任务，告诉系统何时完成工作。有时，你可能还需要中间件来处理请求。

实现异步通信的最佳方式是使用异步 RESTful API。

在创建可扩展的系统时，你必须始终考虑异步通信。

# 摘要

在本章中，我们讨论了可扩展性，包括帮助实现它的库、工具等等。然后我们讨论了如何扩展 RESTful 服务，设计它们时需要考虑什么，以及如何使用不同的模式避免级联故障。

在接下来的章节中，我们将讨论并构建一个网络客户端来调用和消费 RESTful 服务。
