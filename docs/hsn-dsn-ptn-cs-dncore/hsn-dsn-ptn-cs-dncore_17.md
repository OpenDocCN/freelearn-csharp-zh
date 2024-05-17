# 第十四章：评估

# 第一章 - .NET Core 和 C#中的面向对象编程概述

1.  晚期和早期绑定这两个术语是指什么？

早期绑定是在源代码编译时建立的，而晚期绑定是在组件运行时建立的。

1.  C#支持多重继承吗？

不支持。原因是多重继承会导致源代码更复杂。

1.  在 C#中，可以使用什么封装级别防止类被库外访问？

`internal`访问修饰符可用于将类的可见性限制为仅在库内部。

1.  聚合和组合之间有什么区别？

两者都是关联的类型，最容易区分的方法是涉及的类是否可以在没有关联的情况下存在。在组合关联中，涉及的类具有紧密的生命周期依赖性。这意味着当一个类被删除时，相关的类也被删除。

1.  接口可以包含属性吗？（这是一个有点棘手的问题）

接口可以定义属性，但是接口没有主体...

1.  狗吃鱼吗？

狗很可爱，但它们吃它们能够放进嘴里的大多数东西。

# 第二章 - 现代软件设计模式和原则

1.  在 SOLID 中，S 代表什么？责任是什么意思？

单一责任原则。责任可以被视为变更的原因。

1.  围绕循环构建的 SDLC 方法是瀑布还是敏捷？

敏捷是围绕开发过程在一系列周期中进行的概念构建的。

1.  装饰者模式是创建模式还是结构模式？

装饰者模式是一种结构模式，允许功能在类之间分配，并且特别适用于在运行时增强类。

1.  pub-sub 集成代表什么？

发布-订阅是一种有用的模式，其中进程发布消息，其他进程订阅以接收消息。

# 第三章 - 实现设计模式 - 基础部分 1

1.  在为组织开发软件时，有时很难确定需求的原因是什么？

为组织开发软件存在许多挑战。例如，组织行业的变化可能导致当前需求需要进行修改。

1.  瀑布式软件开发与敏捷软件开发相比的两个优点和缺点是什么？

瀑布式软件开发相对于敏捷软件开发提供了优势，因为它更容易理解和实现。在某些情况下，当项目的复杂性和规模较小时，瀑布式软件开发可能是比敏捷式软件开发更好的选择。然而，瀑布式软件开发不擅长处理变化，并且由于范围更大，项目完成之前需求变化的可能性更大。

1.  在编写单元测试时，依赖注入如何帮助？

通过将依赖项注入类，类变得更容易测试，因为依赖项是明确已知且更容易访问的。

1.  为什么以下陈述是错误的？使用 TDD，您不再需要人员测试新软件部署。

测试驱动开发通过将清晰的测试策略纳入软件开发生命周期中来提高解决方案的质量。然而，定义的测试可能不完整，因此仍然需要额外的资源来验证交付的软件。

# 第四章 - 实现设计模式 - 基础部分 2

1.  提供一个示例，说明为什么使用单例不是限制对共享资源访问的好机制？

单例有意在应用程序中创建瓶颈。它也是开发人员学习使用的第一个模式之一，因此通常在不需要限制对共享资源访问的情况下使用。

1.  以下陈述是否正确？为什么？`ConcurrentDictionary` 防止集合中的项目被多个线程同时更新。

对于许多 C# 开发人员来说，意识到 `ConcurrentDictionary` 不能防止集合中的项目被多个线程同时更新是一个痛苦的教训。`ConcurrentDictionary` 保护共享字典免受同时访问和修改。

1.  什么是竞争条件，为什么应该避免？

竞争条件是指多个线程的处理顺序可能导致不同的结果。

1.  工厂模式如何帮助简化代码？

工厂模式是在应用程序内部创建对象的有效方式。

1.  .NET Core 应用程序需要第三方 IoC 容器吗？

.NET Core 具有强大的控制反转内置到框架中。在需要时可以通过其他 IoC 容器进行增强，但不是必需的。

# 第五章 - 实现设计模式 - .NET Core

1.  如果不确定要使用哪种类型的服务生命周期，最好将类注册为哪种类型？为什么？

瞬态生命周期服务在每次请求时创建。大多数类应该是轻量级、无状态的服务，因此这是最好的服务生命周期。

1.  在 .NET Core ASP .NET 解决方案中，范围是定义为每个 Web 请求还是每个会话？

一个范围是每个 Web 请求（连接）。

1.  在 .NET Core DI 框架中将类注册为单例会使其线程安全吗？

不，框架将为后续请求提供相同的实例，但不会使类线程安全。

1.  .NET Core DI 框架只能被其他由微软提供的 DI 框架替换是真的吗？

是的，有许多 DI 框架可以用来替代原生 DI 框架。

# 第六章 - 实现 Web 应用程序的设计模式 - 第一部分

1.  什么是 Web 应用程序？

这是一个使用 Web 浏览器的程序，如果在公共网络上可用，可以从任何地方访问。它基于客户端/服务器架构，通过接收 HTTP 请求并提供 HTTP 响应来为客户端提供服务。

1.  制作您选择的 Web 应用程序，并描述 Web 应用程序的工作的图像。

参考 FlixOne 应用程序。

1.  什么是控制反转？

控制反转（IoC）是一个容器，用于反转或委托控制。它基于 DI 框架。.NET Core 具有内置的 IoC 容器。

1.  什么是 UI/架构模式？您想使用哪种模式，为什么？

UI 架构模式旨在创建强大的用户界面，以使用户更好地体验应用程序。从开发人员的角度来看，MVC、MVP 和 MVVM 是流行的模式。

# 第七章 - 实现 Web 应用程序的设计模式 - 第二部分

1.  认证和授权是什么？

认证是一个系统通过凭据（通常是用户 ID 和密码）验证或识别传入请求的过程。如果系统发现提供的凭据错误，那么它会通知用户（通常通过 GUI 屏幕上的消息）并终止授权过程。

授权始终在认证之后。这是一个过程，允许经过验证的用户在验证他们对特定资源或数据的访问权限后访问资源或数据。

1.  在请求的第一级使用认证，然后允许进入受限区域的传入请求安全吗？

这并不总是安全的。作为开发人员，我们应该采取一切必要的步骤，使我们的应用程序更安全。在第一级请求认证之后，系统还应检查资源级别的权限。

1.  您将如何证明授权始终在认证之后？

在 Web 应用程序的简单场景中，它首先通过请求登录凭据来验证用户，然后根据角色授权用户访问特定资源。

1.  什么是测试驱动开发，为什么开发人员关心它？

测试驱动开发是一种确保代码经过测试的方法；这就像通过编写代码来测试代码。TDD 也被称为红/蓝/绿概念。开发人员应该遵循它，使他们的代码/程序在没有任何错误的情况下工作。

1.  定义 TDD Katas。它如何帮助我们改进我们的 TDD 方法？

TDD Katas 是帮助通过实践学习编码的小场景或问题。您可以参考 Fizz Buzz Kata 的例子，开发人员应该应用编码来学习和练习 TDD。如果您想练习 TDD Katas，请参考此存储库：[`github.com/garora/TDD-Katas.`](https://github.com/garora/TDD-Katas)

# 第八章 – .NET Core 中的并发编程

1.  什么是并发编程？

每当事情/任务同时发生时，我们说任务是同时发生的。在我们的编程语言中，每当程序的任何部分同时运行时，它就是并发编程。

1.  真正的并行是如何发生的？

在单个 CPU 机器上不可能实现真正的并行，因为任务不可切换，它只在具有多个 CPU（多个核心）的机器上发生。

1.  什么是竞争条件？

更多线程可以访问相同的共享数据并以不可预测的结果进行更新的潜力可以称为竞争条件。

1.  为什么我们应该使用`ConcurrentDictionary`？

并发字典是一个线程安全的集合类，它存储键值对。这个类有锁语句的实现，并提供了一个线程安全的类。

# 第九章 – 函数式编程实践 – 一种方法

1.  什么是函数式编程？

函数式编程是一种符号计算的方法，就像我们解决数学问题一样。任何函数式编程都是基于数学函数的。任何函数式编程风格的语言都是通过两个术语来解决问题的：要解决什么和如何解决？

1.  函数式编程中的引用透明度是什么？

在函数式程序中，一旦我们定义了变量，它们在整个程序中不会改变其值。由于函数式程序没有赋值语句，如果我们需要存储值，就没有其他选择；相反，我们定义新变量。

1.  什么是`Pure`函数？

`Pure`函数是通过说它们是纯净的来加强函数式编程的函数。这些函数满足两个条件：

+   +   提供的参数的最终结果/输出将始终保持不变。

+   即使调用了一百次，这些都不会影响程序的行为或应用程序的执行路径。

# 第十章 – 响应式编程模式和技术

1.  什么是流？

一系列事件称为流。流可以发出三种东西：一个值，一个错误和一个完成信号。

1.  什么是响应式属性？

当事件触发时，响应式属性是会做出反应的绑定属性。

1.  什么是响应式系统？

根据响应式宣言，我们可以得出结论，响应式系统如下：

+   +   **响应式**：响应式系统是基于事件的设计系统，因此这种设计方法使得这些系统能够快速响应任何请求。

+   **可扩展**：响应式系统具有响应性。这些系统可以通过扩展或减少分配的资源来对可扩展性进行调整。

+   **弹性**：弹性系统是指即使出现任何故障/异常，也不会停止的系统。响应式系统是以这样的方式设计的，即使出现任何异常或故障，系统也不会死掉；它仍然在工作。

+   **基于消息**：任何项目的数据都代表一条消息，可以发送到特定的目的地。当消息或数据到达给定状态时，会发出一个信号事件来通知消息已经被接收。响应式系统依赖于这种消息传递。

1.  **合并两个响应流是什么意思？**

合并两个响应流实际上是将两个相似或不同的响应流的元素组合成一个新的响应流。例如，如果你有`stream1`和`stream2`，那么`stream3 = stream1.merge(stream2)`，但`stream3`的顺序不会按顺序排列。

1.  **什么是 MVVM 模式？**

**模型-视图-视图模型**（MVVM）是**模型-视图-控制器**（MVC）的变体之一，以满足现代 UI 开发方法，其中 UI 开发是设计师/UI 开发人员的核心责任，而不是应用程序开发人员。在这种开发方法中，一个更注重图形的设计师专注于使用户界面更具吸引力，可能并不关心应用程序的开发部分。通常，设计师（UI 人员）使用各种工具使用户界面更具吸引力。MVVM 的定义如下：

+   +   **模型**：也称为领域对象，它只保存数据；没有业务逻辑、验证等。

+   **视图**：这是为最终用户表示数据。

+   **视图模型**：这将视图和模型分开；它的主要责任是为最终用户提供更好的东西。

# 第十一章 - 高级数据库设计和应用技术

1.  **什么是分类账式数据库？**

这个数据库只用于插入操作；没有更新。然后，你创建一个视图，将插入聚合在一起。

1.  **什么是 CQRS？**

命令查询责任分离是一种将查询（插入）和命令（更新）之间的责任分离的模式。

1.  **何时使用 CQRS？**

CQRS 可以是一个适用于基于任务或事件驱动系统的良好模式，特别是当解决方案由多个应用程序组成而不是单个单片网站或应用程序时。它是**一种模式而不是一种架构**，因此应该在特定情况下应用，而不是在所有业务场景中应用。

# 第十二章 - 云编码

1.  **这是一个真实的陈述吗？大多数模式是最近开发的，只适用于基于云的应用。**

不，这不是真的。随着软件开发的变化，模式一直在不断发展，但许多核心模式已存在几十年。

1.  **ESB 代表什么？它可以用于哪种架构：EDA、SOA 还是单片？**

它代表企业服务总线。它可以有效地用于事件驱动架构和面向服务的架构。

1.  **基于队列的负载平衡主要用于 DevOps、可伸缩性还是可用性？**

可用性。基于队列的负载平衡主要用于处理负载的大幅波动，作为缓冲以减少应用程序变得不可用的机会。

1.  **CI/CD 的好处是什么？在全球分散团队的大量还是单个小型团队的共同开发人员中更有益？**

一般来说，CI/CD 有助于通过频繁执行合并和部署来及早识别开发生命周期中的问题。更大、更复杂的解决方案往往比更小、更简单的解决方案更有益。

1.  **在遵循静态内容托管的网站中，浏览器是直接通过 CDN 检索图像和静态内容，还是 Web 应用程序代表浏览器检索信息？**

内容交付网络可以通过在多个数据中心缓存静态资源来提高性能和可用性，从而使浏览器可以直接从最近的数据中心检索内容。

# 附录 A - 杂项最佳实践

1.  **什么是实践？从我们的日常生活中举几个例子。**

练习可能是一个或多个日常活动。要学会开车，我们应该练习驾驶。练习是一种不需要记忆的活动。我们日常生活中有很多练习的例子：一边看电视节目一边吃饭，等等。在你观看最喜欢的电视节目时吃东西并不会打乱你的节奏。

1.  **我们可以通过练习来掌握特定的编码技能。解释一下。**

是的，我们可以通过练习来掌握特定的编码技能。练习需要注意力和一贯性。例如，你想学习测试驱动开发。为了做到这一点，你需要先学会它。你可以通过练习 TDD-Katas 来学习它。

1.  **什么是测试驱动开发，它如何帮助开发者练习？**

测试驱动开发是一种确保代码经过测试的方法；就好像我们通过编写代码来测试代码一样。TDD 也被称为红/蓝/绿概念。开发者应该遵循它，使他们的代码/程序能够在没有任何错误的情况下运行。