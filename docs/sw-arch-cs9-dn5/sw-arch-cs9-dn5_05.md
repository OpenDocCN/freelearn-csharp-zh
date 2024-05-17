# 第五章：将微服务架构应用于企业应用程序

本章专门描述基于称为微服务的小模块的高度可扩展架构。微服务架构允许进行细粒度的扩展操作，每个模块都可以根据需要进行扩展，而不会影响系统的其他部分。此外，它们还允许更好的持续集成/持续部署（CI/CD），因为每个系统子部分都可以独立演进和部署，而不受其他部分的影响。

在本章中，我们将涵盖以下主题：

+   什么是微服务？

+   什么时候使用微服务有帮助？

+   .NET 如何处理微服务？

+   管理微服务所需的工具有哪些？

通过本章的学习，您将学会如何在.NET 中实现单个微服务。第六章“Azure Service Fabric”和第七章“Azure Kubernetes Service”还介绍了如何部署、调试和管理基于微服务的整个应用程序。

# 技术要求

在本章中，您将需要以下内容：

+   安装了所有数据库工具的 Visual Studio 2019 免费社区版或更高版本。

+   一个免费的 Azure 账户。第一章“理解软件架构的重要性”中的“创建 Azure 账户”部分解释了如何创建账户。

+   如果您想在 Visual Studio 中调试 Docker 容器化的微服务，需要 Windows 版 Docker Desktop（[`www.docker.com/products/docker-desktop`](https://www.docker.com/products/docker-desktop)）。

# 什么是微服务？

微服务架构允许将解决方案的每个模块独立于其他模块进行扩展，以实现最大吞吐量和最小成本。事实上，对整个系统进行扩展而不是当前瓶颈部分必然会导致资源的明显浪费，因此对子系统扩展的细粒度控制对系统的整体成本有着重要影响。

然而，微服务不仅仅是可扩展的组件-它们是可以独立开发、维护和部署的软件构建块。将开发和维护分割成可以独立开发、维护和部署的模块，可以改善整个系统的 CI/CD 周期（CI/CD 概念在第三章的“使用 Azure DevOps 组织工作”部分和“使用 Azure DevOps 记录需求”部分中有更详细的解释）。

由于微服务的“独立性”，CI/CD 的改进是可能的，因为它实现了以下功能：

+   在不同类型的硬件上进行微服务的扩展和分布。

+   由于每个微服务都是独立部署的，因此不存在二进制兼容性或数据库结构兼容性约束。因此，不需要对组成系统的不同微服务的版本进行对齐。这意味着每个微服务可以根据需要进行演进，而不受其他微服务的限制。

+   将开发任务分配给完全独立的小团队，从而简化工作组织并减少处理大型团队时不可避免的协调低效问题。

+   使用更合适的技术和更合适的环境来实现每个微服务，因为每个微服务都是一个独立的部署单元。这意味着选择最适合您需求的工具和最大程度减少开发工作和/或最大程度提高性能的环境。

+   由于每个微服务可以使用不同的技术、编程语言、工具和操作系统来实现，企业可以通过将环境与开发人员的能力匹配来利用所有可用的人力资源。例如，如果使用 Java 实现微服务并具有相同的所需行为，那么未充分利用的 Java 开发人员也可以参与.NET 项目。

+   遗留子系统可以嵌入独立的微服务中，从而使它们能够与新的子系统合作。这样，公司可以减少新系统版本的上市时间。此外，这样，遗留系统可以逐渐向更现代的系统演进，对成本和组织的影响可接受。

下一小节解释了微服务的概念是如何构思的。然后，我们将继续通过探索基本的微服务设计原则并分析为什么微服务通常被设计为 Docker 容器来继续介绍本章节。

## 微服务和模块概念的演变

为了更好地理解微服务的优势以及它们的设计技术，我们必须牢记软件模块化和软件模块的双重性质：

+   代码模块化是指使我们能够修改一块代码而不影响应用程序其余部分的代码组织。通常，它是通过面向对象设计来实现的，其中模块可以用类来标识。

+   **部署模块化**取决于部署单元是什么以及它们具有哪些属性。最简单的部署单元是可执行文件和库。因此，例如，动态链接库（DLL）肯定比静态库更模块化，因为它们在部署之前不需要与主要可执行文件链接。

虽然代码模块化的基本概念已经达到了稳定状态，但部署模块化的概念仍在不断发展，微服务目前是这一演变路径上的最新技术。

作为对导致微服务发展的主要里程碑的简要回顾，我们可以说，首先，将单体可执行文件拆分为静态库。随后，动态链接库（DLL）取代了静态库。

当.NET（以及其他类似的框架，如 Java）改进了可执行文件和库的模块化时，发生了巨大的变化。实际上，使用.NET，它们可以部署在不同的硬件和不同的操作系统上，因为它们部署在第一次执行库时编译的中间语言中。此外，它们克服了以前 DLL 的一些版本问题，因为任何可执行文件都会带有一个与安装在操作系统中的相同 DLL 版本不同的版本的 DLL。

然而，.NET 不能接受两个引用的 DLL - 假设为*A*和*B* - 使用共同依赖项的两个不同版本 - 假设为*C*。例如，假设有一个新版本的*A*，具有许多我们想要使用的新功能，反过来依赖于*B*不支持的*C*的新版本。在这种情况下，由于*C*与*B*的不兼容性，我们应该放弃*A*的新版本。这个困难导致了两个重要的变化：

+   开发世界从 DLL 和/或单个文件转向了包管理系统，如 NuGet 和 npm，这些系统可以通过语义化版本控制自动检查版本兼容性。

+   **面向服务的架构**（SOA）。部署单元开始被实现为 SOAP，然后是 REST Web 服务。这解决了版本兼容性问题，因为每个 Web 服务在不同的进程中运行，并且可以使用最合适的每个库的版本，而不会导致与其他 Web 服务不兼容的风险。此外，每个 Web 服务公开的接口是平台无关的，也就是说，Web 服务可以与使用任何框架的应用程序连接并在任何操作系统上运行，因为 Web 服务协议基于普遍接受的标准。SOA 和协议将在*第十四章*《使用.NET Core 应用面向服务的架构》中详细讨论。

微服务是 SOA 的演变，并增加了更多功能和约束，以改善服务的可伸缩性和模块化，以改善整体的 CI/CD 周期。有时人们说*微服务是 SOA 做得好*。

## 微服务设计原则

总之，微服务架构是最大程度地实现独立性和细粒度扩展的 SOA。现在我们已经澄清了微服务独立性和细粒度扩展的所有优势，以及独立性的本质，我们可以看看微服务设计原则。

让我们从独立性约束产生的原则开始。我们将在单独的小节中讨论它们。

### 设计选择的独立性

每个微服务的设计不能依赖于在其他微服务实现中所做的设计选择。这个原则使得每个微服务的 CI/CD 周期完全独立，并让我们在如何实现每个微服务上有更多的技术选择。这样，我们可以选择最好的可用技术来实现每个微服务。

这个原则的另一个结果是，不同的微服务不能连接到相同的共享存储（数据库或文件系统），因为共享相同的存储也意味着共享决定存储子系统结构的所有设计选择（数据库表设计，数据库引擎等）。因此，要么一个微服务有自己的数据存储，要么根本没有存储，并与负责处理存储的其他微服务进行通信。

在这里，拥有专用的数据存储并不意味着物理数据库分布在微服务本身的进程边界内，而是微服务具有对由外部数据库引擎处理的数据库或一组数据库表的独占访问权限。事实上，出于性能原因，数据库引擎必须在专用硬件上运行，并具有针对其存储功能进行优化的操作系统和硬件功能。

通常，*设计选择的独立性*以更轻的形式解释，通过区分逻辑和物理微服务。更具体地说，逻辑微服务是由使用相同数据存储但独立负载平衡的多个物理微服务实现的。也就是说，逻辑微服务被设计为一个逻辑单元，然后分割成更多的物理微服务以实现更好的负载平衡。

### 独立于部署环境

微服务在不同的硬件节点上进行扩展，并且不同的微服务可以托管在同一节点上。因此，微服务越少依赖操作系统提供的服务和其他安装的软件，它就可以部署在更多的硬件节点上。还可以进行更多的节点优化。

这就是为什么微服务通常是容器化并使用 Docker 的原因。容器将在本章的*容器和 Docker*小节中更详细地讨论，但基本上，容器化是一种技术，允许每个微服务携带其依赖项，以便它可以在任何地方运行。

### 松散耦合

每个微服务必须与所有其他微服务松散耦合。这个原则具有双重性质。一方面，这意味着，根据面向对象编程原则，每个微服务公开的接口不能太具体，而应尽可能通用。然而，这也意味着微服务之间的通信必须最小化，以减少通信成本，因为微服务不共享相同的地址空间，运行在不同的硬件节点上。

### 不要有链接的请求/响应

当请求到达微服务时，它不能引起对其他微服务的递归链式请求/响应，因为类似的链式请求/响应会导致无法接受的响应时间。如果所有微服务的私有数据模型在每次更改时都与推送通知同步，就可以避免链式请求/响应。换句话说，一旦由微服务处理的数据发生变化，这些变化就会发送到可能需要这些数据来处理其请求的所有微服务。这样，每个微服务都在其私有数据存储中拥有处理所有传入请求所需的所有数据，无需向其他微服务请求缺少的数据。

总之，每个微服务必须包含其所需的所有数据，以提供传入请求并确保快速响应。为了使其数据模型保持最新并准备好处理传入请求，微服务必须在数据发生变化时立即通知其它微服务。这些数据变化应通过异步消息进行通信，因为同步嵌套消息会导致不可接受的性能问题，因为它们会阻塞所有涉及调用树的线程，直到返回结果。

值得指出的是，“设计选择的独立性”原则实际上是领域驱动设计中的有界上下文原则，我们将在《第十二章：理解软件解决方案中的不同领域》中详细讨论。在本章中，我们将看到，通常情况下，完整的领域驱动设计方法对于每个微服务的“更新”子系统非常有用。

总的来说，按照有界上下文原则开发的所有系统通常都更适合使用微服务架构来实现。实际上，一旦将系统分解为几个完全独立且松耦合的部分，由于不同的流量和不同的资源需求，这些不同的部分很可能需要独立扩展。

除了上述约束，我们还必须添加一些构建可重用 SOA 的最佳实践。关于这些最佳实践的更多细节将在《第十四章：使用.NET Core 应用面向服务的架构》中给出，但是现在，大多数 SOA 最佳实践都是由用于实现 Web 服务的工具和框架自动强制执行的。

细粒度的扩展要求微服务足够小，以便隔离明确定义的功能，但这也需要一个复杂的基础架构来自动实例化微服务，将实例分配到各种硬件计算资源上，通常称为“节点”，并根据需要进行扩展。这些结构将在本章的“需要哪些工具来管理微服务？”部分中介绍，并在《第六章：Azure Service Fabric》和《第七章：Azure Kubernetes Service》中详细讨论。

此外，通过异步通信进行通信的细粒度扩展的分布式微服务要求每个微服务具有弹性。实际上，由于硬件故障或在负载平衡操作期间目标实例被终止或移动到另一个节点的简单原因，针对特定微服务实例的通信可能会失败。

临时故障可以通过指数级重试来克服。这意味着在每次失败后，我们会延迟指数级地重试相同的操作，直到达到最大尝试次数。例如，首先，我们会在 10 毫秒后重试，如果这次重试操作失败，那么在 20 毫秒后进行新的尝试，然后是 40 毫秒，依此类推。

另一方面，长期故障通常会导致重试操作的激增，可能会使所有系统资源饱和，类似于拒绝服务攻击。因此，通常会将指数级重试与“断路器策略”一起使用：在一定数量的失败之后，假定存在长期故障，并且通过返回立即失败而不尝试通信操作来阻止对资源的访问一段时间。

同样重要的是，某些子系统的拥塞，无论是由于故障还是请求高峰，都不会传播到其他系统部分，以防止整体系统拥塞。**隔离舱壁**通过以下方式避免拥塞传播：

+   只允许一定数量的类似的同时出站请求；比如说，10。这类似于对线程创建设置上限。

+   超过先前限制的请求将被排队。

+   如果达到最大队列长度，任何进一步的请求都会导致抛出异常以中止它们。

重试策略可能导致同一消息被接收和处理多次，因为发送方未收到消息已被接收的确认，或者因为操作超时，而接收方实际上已接收了消息。这个问题的唯一可能解决方案是设计所有消息都是幂等的，也就是说，设计消息的处理多次与处理一次具有相同的效果。

例如，将数据库表字段更新为一个值是幂等操作，因为重复一次或两次会产生完全相同的效果。然而，递增十进制字段不是幂等操作。微服务设计者应该努力设计整个应用程序，尽可能多地使用幂等消息。剩下的非幂等消息必须以以下方式或其他类似技术转换为幂等消息：

+   附上时间和一些唯一标识符，以唯一标识每条消息。

+   将所有已接收的消息存储在一个字典中，该字典已由附加到前一点提到的消息的唯一标识符进行索引。

+   拒绝旧消息。

+   当接收到可能是重复的消息时，请验证它是否包含在字典中。如果是，则已经被处理，因此拒绝它。

+   由于旧消息被拒绝，它们可以定期从字典中删除，以避免指数级增长。

我们将在*第六章*的示例中使用这种技术，*Azure Service Fabric*。

值得指出的是，一些消息代理（如 Azure Service Bus）提供了实施先前描述的技术的设施。Azure Service Bus 在“.NET 通信设施”子部分中进行了讨论。

在下一小节中，我们将讨论基于 Docker 的微服务容器化。

## 容器和 Docker

我们已经讨论了具有不依赖于运行环境的微服务的优势：更好的硬件使用、能够将旧软件与新模块混合使用、能够混合使用多个开发堆栈以使用最佳堆栈来实现每个模块等。通过在私有虚拟机上部署每个微服务及其所有依赖项，可以轻松实现与托管环境的独立性。

然而，启动具有其操作系统私有副本的虚拟机需要很长时间，而微服务必须快速启动和停止，以减少负载平衡和故障恢复成本。事实上，新的微服务可能会启动以替换故障的微服务，或者因为它们从一个硬件节点移动到另一个硬件节点以执行负载平衡。此外，为每个微服务实例添加整个操作系统副本将是一个过度的开销。

幸运的是，微服务可以依赖一种更轻量级的技术：容器。容器是一种轻量级虚拟机。它们不会虚拟化整个机器-它们只是虚拟化位于操作系统内核之上的操作系统文件系统级别。它们使用托管机器的操作系统（内核、DLL 和驱动程序），并依赖于操作系统的本机功能来隔离进程和资源，以确保运行的图像的隔离环境。

因此，容器与特定的操作系统绑定，但它们不会遭受在每个容器实例中复制和启动整个操作系统的开销。

在每台主机机器上，容器由运行时处理，该运行时负责从*图像*创建容器，并为每个容器创建一个隔离的环境。最著名的容器运行时是 Docker，它是容器化的*事实上的*标准。

图像是指定放入每个容器的内容以及要在容器外部公开的容器资源（如通信端口）的文件。图像不需要显式指定其完整内容，但可以分层。这样，通过在现有图像之上添加新的软件和配置信息来构建图像。

例如，如果您想将.NET 应用程序部署为 Docker 镜像，只需将软件和文件添加到 Docker 镜像中，然后引用已经存在的.NET Docker 镜像即可。

为了方便图像引用，图像被分组到可能是公共或私有的注册表中。它们类似于 NuGet 或 npm 注册表。Docker 提供了一个公共注册表（[`hub.docker.com/_/registry`](https://hub.docker.com/_/registry)），您可以在其中找到大多数您可能需要在自己的图像中引用的公共图像。然而，每个公司都可以定义私有注册表。例如，Azure 提供了 Microsoft 容器注册表，您可以在其中定义您的私有容器注册表服务：[`azure.microsoft.com/en-us/services/container-registry/`](https://azure.microsoft.com/en-us/services/container-registry/)。在那里，您还可以找到大多数与.NET 相关的图像，您可能需要在您的代码中引用它们。

在实例化每个容器之前，Docker 运行时必须解决所有递归引用。这个繁琐的工作不是每次创建新容器时都执行的，因为 Docker 运行时有一个缓存，它存储与每个输入图像对应的完全组装的图像。

由于每个应用程序通常由几个模块组成，这些模块在不同的容器中运行，Docker 还允许使用称为`.yml`文件的组合文件，指定以下信息：

+   部署哪些图像。

+   如何将每个图像公开的内部资源映射到主机机器的物理资源。例如，如何将 Docker 图像公开的通信端口映射到物理机器的端口。

我们将在本章的* .NET 如何处理微服务？*部分中分析 Docker 图像和`.yml`文件。

Docker 运行时处理单个机器上的图像和容器，但通常，容器化的微服务是部署和负载均衡在由多台机器组成的集群上的。集群由称为**编排器**的软件组成。编排器将在本章的*需要哪些工具来管理微服务？*部分中介绍，并在*第六章*，*Azure 服务织物*和*第七章*，*Azure Kubernetes 服务*中详细描述。

现在我们已经了解了微服务是什么，它们可以解决什么问题以及它们的基本设计原则，我们准备分析何时以及如何在我们的系统架构中使用它们。下一节将分析我们应该何时使用它们。

# 微服务何时有帮助？

回答这个问题需要我们理解微服务在现代软件架构中的作用。我们将在以下两个小节中进行讨论：

+   分层架构和微服务

+   什么时候考虑微服务架构是值得的？

让我们详细了解分层架构和微服务。

## 分层架构和微服务

企业系统通常以逻辑独立的层组织。第一层是与用户交互的层，称为表示层，而最后一层负责存储/检索数据，称为数据层。请求起源于表示层，并通过所有层传递，直到达到数据层，然后返回，反向穿过所有层，直到达到表示层，表示层负责向用户/客户端呈现结果。层不能“跳过”。

每个层从前一层获取数据，处理数据，并将其传递给下一层。然后，它从下一层接收结果并将其发送回前一层。此外，抛出的异常不能跨越层 - 每个层必须负责拦截所有异常并解决它们，或将它们转换为以其前一层语言表达的其他异常。层架构确保每个层的功能与所有其他层的功能完全独立。

例如，我们可以更改数据库引擎而不影响数据层以上的所有层。同样，我们可以完全更改用户界面，即表示层，而不影响系统的其余部分。

此外，每个层实现了不同类型的系统规范。数据层负责系统“必须记住”的内容，表示层负责系统用户交互协议，而中间的所有层实现了领域规则，指定数据如何处理（例如，如何计算员工工资）。通常，数据层和表示层之间只有一个领域规则层，称为业务或应用层。

每个层都“说”不同的语言：数据层“说”所选择的存储引擎的语言，业务层“说”领域专家的语言，表示层“说”用户的语言。因此，当数据和异常从一层传递到另一层时，它们必须被转换为目标层的语言。

关于如何构建分层架构的详细示例将在《第十二章》《理解软件解决方案中的不同领域》的《用例 - 理解用例的领域》部分中给出，该部分专门讨论领域驱动设计。

话虽如此，微服务如何适应分层架构？它们是否适用于所有层的功能还是只适用于某些层？单个微服务是否可以跨越多个层？

最后一个问题最容易回答：是的！实际上，我们已经说过微服务应该在其逻辑边界内存储所需的数据。因此，有些微服务跨越业务和数据层。其他一些微服务负责封装共享数据并保持在数据层中。因此，我们可能有业务层微服务、数据层微服务以及跨越两个层的微服务。那么，表示层呢？

### 表示层

如果在服务器端实现，表示层也可以适应微服务架构。单页应用程序和移动应用程序在客户端机器上运行表示层，因此它们要么直接连接到业务微服务层，要么更常见地连接到公共接口并负责将请求路由到正确的微服务的 API 网关。

在微服务架构中，如果表示层是一个网站，可以使用一组微服务来实现。然而，如果它需要重型的 Web 服务器和/或重型的框架，将它们容器化可能不方便。这个决定还必须考虑到容器化 Web 服务器和系统其余部分之间可能需要硬件防火墙的性能损失。

ASP.NET 是一个轻量级的框架，运行在轻量级的 Kestrel Web 服务器上，因此可以高效地容器化，并用于内部网络应用的微服务。然而，公共高流量的网站需要专用的硬件/软件组件，阻止它们与其他微服务一起部署。实际上，虽然 Kestrel 对于内部网络网站是一个可接受的解决方案，但公共网站需要更完整的 Web 服务器，如 IIS、Apache 或 NGINX。在这种情况下，安全性和负载均衡要求更加紧迫，需要专用的硬件/软件节点和组件。因此，基于微服务的架构通常提供专门的组件来处理与外部世界的接口。例如，在第七章《Azure Kubernetes 服务》中，我们将看到在 Kubernetes 集群中，这个角色由所谓的“入口”扮演。

单体网站可以轻松地分解为负载均衡的较小子网站，而无需使用微服务特定的技术，但是微服务架构可以将所有微服务的优势带入单个 HTML 页面的构建中。更具体地说，不同的微服务可以负责每个 HTML 页面的不同区域。不幸的是，在撰写本文时，使用现有的.NET 技术很难实现类似的场景。

可以在这里找到一个使用基于 ASP.NET 的微服务实现每个 HTML 页面构建的网站的概念验证：[`github.com/Particular/Workshop/tree/master/demos/asp-net-core`](https://github.com/Particular/Workshop/tree/master/demos/asp-net-core)。这种方法的主要限制是微服务仅合作生成生成 HTML 页面所需的数据，而不是生成实际的 HTML 页面。相反，这由一个单体网关处理。实际上，在撰写本文时，诸如 ASP.NET MVC 之类的框架并不提供任何用于分发 HTML 生成的功能。我们将在第十五章《展示 ASP.NET Core MVC》中回到这个例子。

现在我们已经澄清了系统的哪些部分可以从采用微服务中受益，我们准备好陈述在决定如何采用微服务时的规则了。

## 什么时候值得考虑微服务架构？

微服务可以改进业务层和数据层的实现，但是它们的采用也有一些成本：

+   为节点分配实例并对其进行扩展会产生云费用或内部基础设施和许可证的成本。

+   将一个独特的进程分解为更小的通信进程会增加通信成本和硬件需求，特别是如果微服务被容器化。

+   为微服务设计和测试软件需要更多的时间，并增加了工程成本，无论是时间还是复杂性。特别是，使微服务具有弹性并确保它们充分处理所有可能的故障，以及使用集成测试验证这些功能，可能会将开发时间增加一个数量级以上。

那么，什么时候微服务的成本值得使用？有哪些功能必须实现为微服务？

对于第二个问题的粗略答案是：是的，当应用程序在流量和/或软件复杂性方面足够大时。实际上，随着应用程序的复杂性增加和流量增加，我们建议支付与其扩展相关的成本，因为这样可以进行更多的扩展优化，并在开发团队方面更好地处理。我们为此支付的成本很快就会超过采用微服务的成本。

因此，如果细粒度的扩展对我们的应用程序有意义，并且我们能够估计细粒度扩展和开发带来的节省，我们可以轻松计算出一个整体应用程序吞吐量限制，从而使采用微服务变得方便。

微服务的成本也可以通过增加我们产品/服务的市场价值来证明。由于微服务架构允许我们使用针对其使用进行优化的技术来实现每个微服务，因此增加到我们的软件中的质量可能会证明所有或部分微服务的成本。

然而，扩展和技术优化并不是唯一需要考虑的参数。有时候，我们被迫采用微服务架构，无法进行详细的成本分析。

如果负责整个系统的 CI/CD 的团队规模增长过大，这个大团队的组织和协调会导致困难和低效。在这种情况下，最好将整个 CI/CD 周期分解为可以由较小团队负责的独立部分的架构。

此外，由于这些开发成本只能通过大量请求来证明，我们可能有高流量由不同团队开发的独立模块正在处理。因此，扩展优化和减少开发团队之间的交互的需求使得采用微服务架构非常方便。

从这个可以得出结论，如果系统和开发团队增长过快，就需要将开发团队分成较小的团队，每个团队负责一个高效的有界上下文子系统。在类似的情况下，微服务架构很可能是唯一可行的选择。

另一种迫使采用微服务架构的情况是将新的子部分与基于不同技术的遗留子系统集成，因为容器化的微服务是实现遗留系统与新的子部分之间高效交互的唯一方式，以逐步用新的子部分替换遗留子部分。同样，如果我们的团队由具有不同开发堆栈经验的开发人员组成，基于容器化的微服务架构可能成为必需。

在下一节中，我们将分析可用的构建块和工具，以便我们可以实现基于.NET 的微服务。

# .NET 如何处理微服务？

.NET 被设计为一个多平台框架，足够轻量级和快速，以实现高效的微服务。特别是，ASP.NET 是实现文本 REST 和二进制 gRPC API 与微服务通信的理想工具，因为它可以在轻量级 Web 服务器（如 Kestrel）上高效运行，并且本身也是轻量级和模块化的。

整个.NET 框架在设计时就考虑了微服务作为战略部署平台，并提供了用于构建高效轻量级 HTTP 和 gRPC 通信的工具和包，以确保服务的弹性和处理长时间运行的任务。下面的小节描述了一些可以用来实现基于.NET 的微服务架构的不同工具或解决方案。

## .NET 通信设施

微服务需要两种类型的通信渠道。

+   第一种是用于接收外部请求的通信渠道，可以直接接收或通过 API 网关接收。由于可用的 Web 服务标准和工具，HTTP 是外部通信的常用协议。.NET 的主要 HTTP/gRPC 通信工具是 ASP.NET，因为它是一个轻量级的 HTTP/gRPC 框架，非常适合在小型微服务中实现 Web API。我们将在*第十四章*的*使用.NET Core 应用服务导向架构*中详细介绍 ASP.NET 应用程序，该章节专门介绍 HTTP 和 gRPC 服务。.NET 还提供了一种高效且模块化的 HTTP 客户端解决方案，能够池化和重用重型连接对象。此外，`HttpClient`类将在*第十四章*的*使用.NET Core 应用服务导向架构*中详细介绍。

+   第二种是一种不同类型的通信渠道，用于向其他微服务推送更新。实际上，我们已经提到过，由于对其他微服务的阻塞调用形成了复杂的阻塞调用树，因此无法通过正在进行的请求触发微服务之间的通信，这将增加请求的延迟时间，达到不可接受的水平。因此，在使用更新之前不应立即请求更新，并且应在状态发生变化时推送更新。理想情况下，这种通信应该是异步的，以实现可接受的性能。实际上，同步调用会在等待结果时阻塞发送者，从而增加每个微服务的空闲时间。然而，如果通信足够快（低通信延迟和高带宽），那么只将请求放入处理队列然后返回成功通信的确认而不是最终结果的同步通信是可以接受的。发布者/订阅者通信将是首选，因为在这种情况下，发送者和接收者不需要彼此了解，从而增加了微服务的独立性。实际上，对某种类型的通信感兴趣的所有接收者只需要注册以接收特定的*事件*，而发送者只需要发布这些事件。所有的连接工作由一个负责排队事件并将其分发给所有订阅者的服务执行。发布者/订阅者模式将在*第十一章*的*设计模式和.NET 5 实现*中详细描述，以及其他有用的模式。

虽然.NET 没有直接提供可帮助实现异步通信或实现发布者/订阅者通信的客户端/服务器工具，但 Azure 提供了一个类似的服务，即*Azure Service Bus*。Azure Service Bus 通过 Azure Service Bus *队列*处理队列异步通信和通过 Azure Service Bus *主题*处理发布者/订阅者通信。

一旦在 Azure 门户上配置了 Azure Service Bus，您就可以通过`Microsoft.Azure.ServiceBus` NuGet 包中的客户端连接到它，以便发送消息/事件和接收消息/事件。

Azure Service Bus 有两种类型的通信：基于队列和基于主题。在基于队列的通信中，发送者放入队列的每个消息都会被第一个从队列中拉取的接收者从队列中删除。另一方面，基于主题的通信是发布者/订阅者模式的一种实现。每个主题都有多个订阅，可以从每个主题订阅中拉取发送到主题的每个消息的不同副本。

设计流程如下：

1.  定义 Azure Service Bus 的私有命名空间。

1.  获取由 Azure 门户创建的根连接字符串和/或定义具有较少权限的新连接字符串。

1.  定义队列和/或主题，发送者将以二进制格式发送其消息。

1.  为每个主题定义所需订阅的名称。

1.  在基于队列的通信中，发送者将消息发送到一个队列，接收者从同一个队列中拉取消息。每个消息被传递给一个接收者。也就是说，一旦接收者获得对队列的访问权，它就会读取并删除一个或多个消息。

1.  在基于主题的通信中，每个发送者将消息发送到一个主题，而每个接收者从与该主题关联的私有订阅中拉取消息。

Azure Service Bus 还有其他商业替代品，如 NServiceBus、MassTransit、Brighter 和 ActiveMQ。还有一个免费的开源选项：RabbitMQ。RabbitMQ 可以在本地、虚拟机或 Docker 容器中安装。然后，您可以通过`RabbitMQ.Client` NuGet 包中的客户端与其连接。

RabbitMQ 的功能与 Azure Service Bus 提供的功能类似，但您必须处理所有实现细节、执行操作的确认等，而 Azure Service Bus 会处理所有低级操作并为您提供一个更简单的接口。Azure Service Bus 和 RabbitMQ 将在第十一章“设计模式和.NET 5 实现”中与基于发布者/订阅者的通信一起进行描述。

如果微服务发布到 Azure Service Fabric 中，将在下一章（第六章“Azure Service Fabric”）中描述，我们可以使用内置的可靠二进制通信。

通信是弹性的，因为通信原语自动使用重试策略。这种通信是同步的，但这不是一个大的限制，因为 Azure Service Fabric 中的微服务具有内置队列；因此，一旦接收者接收到消息，他们可以将其放入队列中并立即返回，而不会阻塞发送者。

然后，队列中的消息由一个单独的线程处理。这种内置通信的主要限制是它不基于发布者/订阅者模式；发送者和接收者必须相互了解。当这种情况不可接受时，应该使用 Azure Service Bus。我们将在第六章“Azure Service Fabric”中学习如何使用 Service Fabric 的内置通信。

## 弹性任务执行

弹性通信和一般情况下的弹性任务执行可以通过一个名为 Polly 的.NET 库轻松实现，该项目是.NET 基金会的成员之一。Polly 可以通过 Polly NuGet 包获得。

在 Polly 中，您定义策略，然后在这些策略的上下文中执行任务，如下所示：

```cs
var myPolicy = Policy
  .Handle<HttpRequestException>()
  .Or<OperationCanceledException>()
  .Retry(3);
....
....
myPolicy.Execute(()=>{
    //your code here
}); 
```

每个策略的第一部分指定了必须处理的异常。然后，您指定在捕获其中一个异常时要执行的操作。在上述代码中，如果由`HttpRequestException`异常或`OperationCanceledException`异常报告了失败，则`Execute`方法将重试最多三次。

以下是指数重试策略的实现：

```cs
var erPolicy= Policy
    ...
    //Exceptions to handle here
    .WaitAndRetry(6, 
        retryAttempt => TimeSpan.FromSeconds(Math.Pow(2,
            retryAttempt))); 
```

`WaitAndRetry`的第一个参数指定在失败的情况下最多执行六次重试。作为第二个参数传递的 lambda 函数指定下一次尝试之前等待的时间。在这个具体的例子中，这个时间随着尝试次数的增加呈指数增长（第一次重试 2 秒，第二次重试 4 秒，依此类推）。

以下是一个简单的断路器策略：

```cs
var cbPolicy=Policy
    .Handle<SomeExceptionType>()
    .CircuitBreaker(6, TimeSpan.FromMinutes(1)); 
```

在六次失败之后，由于返回了异常，任务将在 1 分钟内无法执行。

以下是 Bulkhead 隔离策略的实现（有关更多信息，请参见“微服务设计原则”部分）：

```cs
Policy
  .Bulkhead(10, 15) 
```

`Execute`方法允许最多 10 个并行执行。进一步的任务被插入到执行队列中。这个队列有一个 15 个任务的限制。如果超过队列限制，将抛出异常。

为了使 Bulkhead 隔离策略正常工作，以及为了使每个策略正常工作，必须通过相同的策略实例触发任务执行；否则，Polly 无法计算特定任务的活动执行次数。

策略可以与`Wrap`方法结合使用：

```cs
var combinedPolicy = Policy
  .Wrap(erPolicy, cbPolicy); 
```

Polly 提供了更多选项，例如用于返回特定类型的任务的通用方法、超时策略、任务结果缓存、定义自定义策略等等。还可以将 Polly 配置为任何 ASP.NET 和.NET 应用程序的依赖注入部分的`HttPClient`定义的一部分。这样，定义弹性客户端就非常简单。

官方 Polly 文档的链接在*进一步阅读*部分中。

## 使用通用主机

每个微服务可能需要运行多个独立的线程，每个线程对接收到的请求执行不同的操作。这些线程需要多个资源，例如数据库连接、通信通道、执行复杂操作的专用模块等等。此外，当微服务由于负载平衡或错误而停止时，必须适当地初始化所有处理线程，并在停止时优雅地停止。

所有这些需求促使.NET 团队构思和实现*托管服务*和*主机*。主机为运行多个任务（称为**托管服务**）提供了适当的环境，并为它们提供资源、公共设置和优雅的启动/停止。

Web 主机的概念主要是为了实现 ASP.NET Core Web 框架，但是从.NET Core 2.1 开始，主机概念扩展到了所有.NET 应用程序。

在撰写本书时，在任何 ASP.NET Core 或 Blazor 项目中，都会自动为您创建一个`Host`，因此您只需要在其他项目类型中手动添加它。

与`Host`概念相关的所有功能都包含在`Microsoft.Extensions.Hosting` NuGet 包中。

首先，您需要使用流畅的接口配置主机，从一个`HostBuilder`实例开始。此配置的最后一步是调用`Build`方法，该方法使用我们提供的所有配置信息组装实际的主机：

```cs
var myHost=new HostBuilder()
    //Several chained calls
    //defining Host configuration
    .Build(); 
```

主机配置包括定义公共资源、定义文件的默认文件夹、从多个来源加载配置参数（JSON 文件、环境变量和传递给应用程序的任何参数）以及声明所有托管服务。

值得指出的是，ASP.NET Core 和 Blazor 项目使用执行`Host`的预配置方法，其中包括前面列出的几个任务。

然后，可以启动主机，这将导致所有托管服务启动：

```cs
host.Start(); 
```

程序在前面的指令上保持阻塞，直到主机关闭。主机可以通过其中一个托管服务或通过调用`awaithost.StopAsync(timeout)`来关闭。这里，`timeout`是一个时间段，定义了等待托管服务正常停止的最长时间。在此时间之后，如果托管服务尚未终止，所有托管服务都将被中止。

通常，微服务关闭的事实是通过在协调器启动微服务时传递的`cancellationToken`来表示的。当微服务托管在 Azure Service Fabric 中时，就会发生这种情况。

因此，在大多数情况下，我们可以使用`RunAsync`或`Run`方法，而不是使用`host.Start()`，可能会传递一个从协调器或操作系统中获取的`cancellationToken`：

```cs
await host.RunAsync(cancellationToken) 
```

这种关闭方式在`cancellationToken`进入取消状态时立即触发。默认情况下，主机在关闭时有 5 秒的超时时间，即一旦请求关闭，它会等待 5 秒钟然后退出。这个时间可以在`ConfigureServices`方法中进行更改，该方法用于声明*托管服务*和其他资源：

```cs
var myHost = new HostBuilder()
    .ConfigureServices((hostContext, services) =>
    {
        services.Configure<HostOptions>(option =>
        {
            option.ShutdownTimeout = System.TimeSpan.FromSeconds(10);
        });
        ....
        ....
        //further configuration
    })
    .Build(); 
```

然而，增加主机超时时间不会增加编排器超时时间，因此如果主机等待时间过长，编排器将终止整个微服务。

如果在`Run`或`RunAsync`中没有显式传递取消令牌，则会自动生成一个取消令牌，并在操作系统通知应用程序即将终止时自动发出信号。这个取消令牌将传递给所有托管服务，以便它们有机会优雅地停止。

托管服务是`IHostedService`接口的实现，其唯一的方法是`StartAsync(cancellationToken)`和`StopAsync(cancellationToken)`。

这两个方法都传递了一个`cancellationToken`。`StartAsync`方法中的`cancellationToken`表示请求了关闭。`StartAsync`方法在执行启动主机所需的所有操作时定期检查这个`cancellationToken`，如果它被触发，主机启动过程将被中止。另一方面，`StopAsync`方法中的`cancellationToken`表示关闭超时已过期。

托管服务可以在用于定义主机选项的同一个`ConfigureServices`方法中声明，如下所示：

```cs
services.AddHostedService<MyHostedService>(); 
```

然而，一些项目模板（如 ASP.NET Core 项目模板）在不同的类中定义了一个`ConfigureServices`方法。如果这个方法接收与`HostBuilder.ConfigureServices`方法中可用的`services`参数相同的参数，那么这将正常工作。

`ConfigureServices`内的大多数声明需要添加以下命名空间：

```cs
using Microsoft.Extensions.DependencyInjection; 
```

通常情况下，不直接实现`IHostedService`接口，而是可以从`BackgroundService`抽象类继承，该抽象类公开了更容易实现的`ExecuteAsync(CancellationToken)`方法，我们可以在其中放置整个服务的逻辑。通过将`cancellationToken`作为参数传递，可以更容易地处理关闭。我们将在*第六章*的示例中查看`IHostedService`的实现，*Azure Service Fabric*。

为了允许托管服务关闭主机，我们需要将`IApplicationLifetime`接口声明为其构造函数参数：

```cs
public class MyHostedService: BackgroundService 
{
    private readonly IHostApplicationLifetime applicationLifetime;
    public MyHostedService(IHostApplicationLifetime applicationLifetime)
    {
        this.applicationLifetime=applicationLifetime;
    }
    protected Task ExecuteAsync(CancellationToken token) 
    {
        ...
        applicationLifetime.StopApplication();
        ...
    }
} 
```

当创建托管服务时，它会自动传递一个`IHostApplicationLifetime`的实现，其中的`StopApplication`方法将触发主机关闭。这个实现是自动处理的，但我们也可以声明自定义资源，其实例将自动传递给所有声明它们为参数的主机服务构造函数。因此，假设我们定义了如下构造函数：

```cs
Public MyClass(MyResource x, IResourceInterface1 y)
{
    ...
} 
```

有几种方法可以定义上述构造函数所需的资源：

```cs
services.AddTransient<MyResource>();
services.AddTransient<IResourceInterface1, MyResource1>();
services.AddSingleton<MyResource>();
services.AddSingleton<IResourceInterface1, MyResource1>(); 
```

当我们使用`AddTransient`时，会创建一个不同的实例，并将其传递给所有需要该类型实例的构造函数。另一方面，使用`AddSingleton`时，会创建一个唯一的实例，并将其传递给所有需要声明类型的构造函数。带有两个泛型类型的重载允许您传递一个接口和实现该接口的类型。这样，构造函数需要接口，并与该接口的具体实现解耦。

如果资源的构造函数包含参数，则这些参数将以递归方式使用在`ConfigureServices`中声明的类型进行自动实例化。这种与资源的交互模式称为**依赖注入**（**DI**），将在*第十一章*的*设计模式和.NET 5 实现*中详细讨论。

`HostBuilder`还有一个方法，我们可以用来定义默认文件夹，也就是用来解析所有.NET 方法中提到的所有相对路径的文件夹：

```cs
.UseContentRoot("c:\\<deault path>") 
```

它还有一些方法，我们可以用来添加日志记录目标：

```cs
.ConfigureLogging((hostContext, configLogging) =>
    {
        configLogging.AddConsole();
        configLogging.AddDebug();
    }) 
```

前面的示例显示了一个基于控制台的日志记录源，但我们也可以使用适当的提供程序记录到 Azure 目标。*进一步阅读*部分包含了一些可以与部署在 Azure Service Fabric 中的微服务一起使用的 Azure 日志记录提供程序的链接。一旦您配置了日志记录，您可以通过在它们的构造函数中添加`ILoggerFactory`或`ILogger<T>`参数来启用托管服务并记录自定义消息。

最后，`HostBuilder`有一些方法，我们可以用来从各种来源读取配置参数：

```cs
.ConfigureHostConfiguration(configHost =>
    {
        configHost.AddJsonFile("settings.json", optional: true);
        configHost.AddEnvironmentVariables(prefix: "PREFIX_");
        configHost.AddCommandLine(args);
    }) 
```

应用程序内部如何使用参数将在*第十五章* *介绍 ASP.NET Core MVC*中更详细地解释，该章节专门讨论 ASP.NET。

## Visual Studio 对 Docker 的支持

Visual Studio 支持创建、调试和部署 Docker 图像。Docker 部署要求我们在开发机器上安装*Windows Docker 桌面*，以便我们可以运行 Docker 图像。下载链接可以在本章开头的*技术要求*部分找到。在开始任何开发活动之前，我们必须确保它已安装并运行（当 Docker 运行时运行时，您应该在窗口通知栏中看到一个 Docker 图标）。

Docker 支持将以一个简单的 ASP.NET MVC 项目来描述。让我们创建一个。要做到这一点，请按照以下步骤：

1.  将项目命名为`MvcDockerTest`。

1.  为简单起见，如果尚未禁用身份验证，请禁用身份验证。

1.  在创建项目时，您可以选择添加 Docker 支持，但请不要勾选 Docker 支持复选框。您可以测试如何在创建项目后添加 Docker 支持。

一旦您的 ASP.NET MVC 应用程序脚手架和运行，右键单击**解决方案资源管理器**中的项目图标，然后选择**添加**，然后选择**容器编排器支持** | **Docker Compose**。

您将会看到一个对话框，询问您选择容器应该使用的操作系统；选择与安装*Windows Docker 桌面*时选择的相同的操作系统。这将不仅启用 Docker 图像的创建，还将创建一个 Docker Compose 项目，帮助您配置 Docker Compose 文件，以便它们同时运行和部署多个 Docker 图像。实际上，如果您向解决方案添加另一个 MVC 项目并为其启用容器编排器支持，新的 Docker 图像将被添加到相同的 Docker Compose 文件中。

启用 Docker Compose 而不仅仅是`Docker`的优势在于，您可以手动`配置`图像在开发机器上的运行方式，以及通过编辑添加到解决方案中的 Docker Compose 文件来映射 Docker 图像端口到外部端口。

如果您的 Docker 运行时已经正确安装并运行，您应该能够从 Visual Studio 运行 Docker 图像。

### 分析 Docker 文件

让我们分析一下由 Visual Studio 创建的 Docker 文件。这是一系列的图像创建步骤。每个步骤都是通过`From`指令来丰富现有的图像，这是一个对已经存在的图像的引用。以下是第一步：

```cs
FROM mcr.microsoft.com/dotnet/aspnet:x.x AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443 
```

第一步使用了由 Microsoft 在 Docker 公共存储库中发布的`mcr.microsoft.com/dotnet/aspnet:x.x` ASP.NET（Core）运行时（其中`x.x`是您项目中选择的 ASP.NET（Core）版本）。

`WORKDIR`命令在将要创建的图像中创建了随后的目录。如果目录尚不存在，则在图像中创建它。两个`EXPOSE`命令声明了图像端口将被暴露到图像外部并映射到实际托管机器的端口。映射的端口在部署阶段通过 Docker 命令的命令行参数或 Docker Compose 文件中决定。在我们的例子中，有两个端口：一个用于 HTTP（80），另一个用于 HTTPS（443）。

这个中间图像由 Docker 缓存，它不需要重新计算，因为它不依赖于我们编写的代码，而只依赖于所选的 ASP.NET（Core）运行时版本。

第二步生成一个不同的图像，不用于部署，而是用于创建将被部署的特定应用程序文件：

```cs
FROM mcr.microsoft.com/dotnet/core/sdk:x  AS build
WORKDIR /src
COPY ["MvcDockerTest/MvcDockerTest.csproj", "MvcDockerTest/"]
RUN dotnet restore MvcDockerTest/MvcDockerTest.csproj
COPY . .
WORKDIR /src/MvcDockerTest
RUN dotnet build MvcDockerTest.csproj -c Release -o /app/build
FROM build AS publish
RUN dotnet publish MvcDockerTest.csproj -c Release -o /app/publish 
```

此步骤从包含我们不需要添加到部署的 ASP.NET SDK 图像开始；这些是用于处理项目代码的。在“构建”图像中创建了新的`src`目录，并使其成为当前图像目录。然后，将项目文件复制到`/src/MvcDockerTest`中。

`RUN`命令在图像上执行操作系统命令。在这种情况下，它调用`dotnet`运行时，要求其恢复先前复制的项目文件引用的 NuGet 包。

然后，`COPY..`命令将整个项目文件树复制到`src`图像目录中。最后，将项目目录设置为当前目录，并要求`dotnet`运行时以发布模式构建项目并将所有输出文件复制到新的`/app/build`目录中。最后，在名为`publish`的新图像中执行`dotnet publish`任务，将发布的二进制文件输出到`/app/publish`中。

最后一步从我们在第一步中创建的图像开始，其中包含 ASP.NET（Core）运行时，并添加在上一步中发布的所有文件：

```cs
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MvcDockerTest.dll"] 
```

`ENTRYPOINT`命令指定执行图像所需的操作系统命令。它接受一个字符串数组。在我们的例子中，它接受`dotnet`命令及其第一个命令行参数，即我们需要执行的 DLL。

### 发布项目

如果我们右键单击项目并单击“发布”，将显示几个选项：

+   将图像发布到现有或新的 Web 应用程序（由 Visual Studio 自动创建）

+   发布到多个 Docker 注册表之一，包括私有 Azure 容器注册表，如果尚不存在，可以从 Visual Studio 内部创建

Docker Compose 支持允许您运行和发布多容器应用程序，并添加其他图像，例如可在任何地方使用的容器化数据库。

以下 Docker Compose 文件将两个 ASP.NET 应用程序添加到同一个 Docker 图像中：

```cs
version: '3.4'
services:
  mvcdockertest:
    image: ${DOCKER_REGISTRY-}mvcdockertest
    build:
      context: .
      dockerfile: MvcDockerTest/Dockerfile
  mvcdockertest1:
    image: ${DOCKER_REGISTRY-}mvcdockertest1
    build:
      context: .
      dockerfile: MvcDockerTest1/Dockerfile 
```

上述代码引用了现有的 Docker 文件。任何与环境相关的信息都放在`docker-compose.override.yml`文件中，当从 Visual Studio 启动应用程序时，该文件与`docker-compose.yml`文件合并：

```cs
version: '3.4'
services:
  mvcdockertest:
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:8 
    ports:
      - "3150:80"
      - "44355:443"
    volumes:
      - ${APPDATA}/Asp.NET/Https:/root/.aspnet/https:ro
  mvcdockertest1:
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_HTTPS_PORT=44317
    ports:
      - "3172:80"
      - "44317:443"
    volumes:
      - ${APPDATA}/Asp.NET/Https:/root/.aspnet/https:ro 
```

对于每个图像，该文件定义了一些环境变量，当应用程序启动时，这些变量将在图像中定义，还定义了端口映射和一些主机文件。

主机中的文件直接映射到图像中。每个声明包含主机中的路径，路径在图像中的映射方式以及所需的访问权限。在我们的例子中，使用`volumes`来映射 Visual Studio 使用的自签名 HTTPS 证书。

现在，假设我们想要添加一个容器化的 SQL Server 实例。我们需要像下面这样的指令，分别在`docker-compose.yml`和`docker-compose.override.yml`之间进行拆分：

```cs
sql.data:
  image: mssql-server-linux:latest
  environment:
  - SA_PASSWORD=Pass@word
  - ACCEPT_EULA=Y
  ports:
  - "5433:1433" 
```

在这里，前面的代码指定了 SQL Server 容器的属性，以及 SQL Server 的配置和安装参数。更具体地说，前面的代码包含以下信息：

+   `sql.data`是给容器命名的名称。

+   `image`指定从哪里获取图像。在我们的例子中，图像包含在公共 Docker 注册表中。

+   `environment`指定 SQL Server 所需的环境变量，即管理员密码和接受 SQL Server 许可证。

+   如往常一样，`ports`指定了端口映射。

+   `docker-compose.override.yml`用于在 Visual Studio 中运行图像。

如果您需要为生产环境或测试环境指定参数，可以添加更多的`docker-compose-xxx.override.yml`文件，例如`docker-compose-staging.override.yml`和`docker-compose-production.override.yml`，然后在目标环境中手动启动它们，类似以下代码：

```cs
docker-compose -f docker-compose.yml -f docker-compose-staging.override.yml 
```

然后，您可以使用以下代码销毁所有容器：

```cs
docker-compose -f docker-compose.yml -f docker-compose.test.staging.yml down 
```

虽然`docker-compose`在处理节点集群时的能力有限，但主要用于测试和开发环境。对于生产环境，需要更复杂的工具，我们将在本章后面的*需要哪些工具来管理微服务？*部分中看到。

## Azure 和 Visual Studio 对微服务编排的支持

Visual Studio 具有基于 Service Fabric 平台的微服务应用程序的特定项目模板，您可以在其中定义各种微服务，配置它们，并将它们部署到 Azure Service Fabric，这是一个微服务编排器。Azure Service Fabric 将在*第六章*，*Azure Service Fabric*中详细介绍。

Visual Studio 还具有特定的项目模板，用于定义要部署在 Azure Kubernetes 中的微服务，并且具有用于调试单个微服务的扩展，同时与部署在 Azure Kubernetes 中的其他微服务进行通信。

还提供了用于在开发机器上测试和调试多个通信微服务的工具，无需安装任何 Kubernetes 软件，并且可以使用最少的配置信息自动部署到 Azure Kubernetes 上。

所有用于 Azure Kubernetes 的 Visual Studio 工具将在*第七章*，*Azure Kubernetes Service*中进行描述。

# 需要哪些工具来管理微服务？

在 CI/CD 周期中有效地处理微服务需要一个私有的 Docker 镜像注册表和一个先进的微服务编排器，该编排器能够执行以下操作：

+   在可用的硬件节点上分配和负载均衡微服务

+   监视服务的健康状态，并在发生硬件/软件故障时替换故障服务

+   记录和展示分析数据

+   允许设计师动态更改要分配给集群的硬件节点、服务实例数量等要求

下面的小节描述了我们可以使用的 Azure 设施来存储 Docker 镜像。Azure 中可用的微服务编排器在各自的章节中进行了描述，即*第六章*，*Azure Service Fabric*和*第七章*，*Azure Kubernetes Service*。

## 在 Azure 中定义您的私有 Docker 注册表

在 Azure 中定义您的私有 Docker 注册表很容易。只需在 Azure 搜索栏中键入`Container registries`，然后选择**Container registries**。在出现的页面上，点击**Add**按钮。

将出现以下表单：

![](img/B16756_05_01.png)

图 5.1：创建 Azure 私有 Docker 注册表

您选择的名称用于组成整体注册表 URI：<name>.azurecr.io。与往常一样，您可以指定订阅、资源组和位置。**SKU**下拉菜单可让您选择不同级别的服务，这些服务在性能、可用内存和一些其他辅助功能方面有所不同。

无论何时在 Docker 命令或 Visual Studio 发布表单中提到图像名称，都必须在注册表 URI 前加上前缀：<name>.azurecr.io/<my imagename>。

如果使用 Visual Studio 创建了图像，则可以按照发布项目后出现的说明进行发布。否则，您必须使用`docker`命令将它们推送到您的注册表中。

使用与 Azure 注册表交互的 Docker 命令的最简单方法是在计算机上安装 Azure CLI。从[`aka.ms/installazurecliwindows`](https://aka.ms/installazurecliwindows)下载安装程序并执行它。安装了 Azure CLI 后，您可以从 Windows 命令提示符或 PowerShell 使用`az`命令。为了连接到您的 Azure 帐户，您必须执行以下登录命令：

```cs
az login 
```

此命令应启动您的默认浏览器，并引导您完成手动登录过程。

登录到 Azure 帐户后，您可以通过输入以下命令登录到私有注册表：

```cs
az acr login --name {registryname} 
```

现在，假设您在另一个注册表中有一个 Docker 镜像。作为第一步，让我们在本地计算机上拉取镜像：

```cs
docker pull other.registry.io/samples/myimage 
```

如果有几个版本的前面的图像，则将拉取最新版本，因为没有指定版本。可以按如下方式指定图像的版本：

```cs
docker pull other.registry.io/samples/myimage:version1.0 
```

使用以下命令，您应该在本地图像列表中看到`myimage`：

```cs
docker images 
```

然后，使用您想要在 Azure 注册表中分配的路径为图像打上标签：

```cs
docker tag myimage myregistry.azurecr.io/testpath/myimage 
```

名称和目标标签都可以有版本（`:<version name>`）。

最后，使用以下命令将其推送到您的注册表中：

```cs
docker push myregistry.azurecr.io/testpath/myimage 
```

在这种情况下，您可以指定一个版本；否则，将推送最新版本。

通过执行以下命令，您可以使用以下命令从本地计算机中删除图像：

```cs
docker rmi myregistry.azurecr.io/testpath/myimage 
```

# 摘要

在本章中，我们描述了什么是微服务以及它们是如何从模块的概念演变而来的。然后，我们讨论了微服务的优势以及何时值得使用它们，以及它们的设计的一般标准。我们还解释了 Docker 容器是什么，并分析了容器与微服务架构之间的紧密联系。

然后，我们通过描述在.NET 中可用的所有工具来进行更实际的实现，以便我们可以实现基于微服务的架构。我们还描述了微服务所需的基础设施以及 Azure 集群如何提供 Azure Kubernetes 服务和 Azure Service Fabric。

下一章详细讨论了 Azure Service Fabric 编排器。

# 问题

1.  模块概念的双重性质是什么？

1.  缩放优化是微服务的唯一优势吗？如果不是，请列出一些其他优势。

1.  Polly 是什么？

1.  Visual Studio 提供了哪些对 Docker 的支持？

1.  什么是编排器，Azure 上有哪些编排器可用？

1.  为什么基于发布者/订阅者的通信在微服务中如此重要？

1.  什么是 RabbitMQ？

1.  为什么幂等消息如此重要？

# 进一步阅读

以下是 Azure Service Bus 和 RabbitMQ 两种事件总线技术的官方文档链接：

+   **Azure Service Bus**：[`docs.microsoft.com/en-us/azure/service-bus-messaging/`](https://docs.microsoft.com/en-us/azure/service-bus-messaging/)

+   **RabbitMQ**：[`www.rabbitmq.com/getstarted.html`](https://www.rabbitmq.com/getstarted.html)

+   Polly 是一种可靠通信/任务工具，其文档可以在这里找到：[`github.com/App-vNext/Polly`](https://github.com/App-vNext/Polly)。

+   有关 Docker 的更多信息可以在 Docker 的官方网站上找到：[`docs.docker.com/`](https://docs.docker.com/)。

+   Kubernetes 和`.yaml`文件的官方文档可以在这里找到：[`kubernetes.io/docs/home/`](https://kubernetes.io/docs/home/)。

+   Azure Kubernetes 的官方文档可以在这里找到：[`docs.microsoft.com/en-US/azure/aks/`](https://docs.microsoft.com/en-US/azure/aks/)。

+   Azure Service Fabric 的官方文档可以在此处找到：[`docs.microsoft.com/zh-cn/azure/service-fabric/`](https://docs.microsoft.com/zh-cn/azure/service-fabric/)。

+   Azure Service Fabric 可靠服务的官方文档可以在此处找到：[`docs.microsoft.com/zh-cn/azure/service-fabric/service-fabric-reliable-services-introduction`](https://docs.microsoft.com/zh-cn/azure/service-fabric/service-fabric-reliable-services-introduction)。
