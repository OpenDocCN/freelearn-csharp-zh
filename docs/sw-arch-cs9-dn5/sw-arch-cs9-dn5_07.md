# 第七章：7

# Azure Kubernetes Service

本章致力于描述 Kubernetes 微服务编排器，特别是在 Azure 中的实现，名为 Azure Kubernetes Service。该章节解释了基本的 Kubernetes 概念，然后重点介绍了如何与 Kubernetes 集群进行交互，以及如何部署 Azure Kubernetes 应用程序。所有概念都通过简单的示例进行了实践。我们建议在阅读本章之前先阅读*第五章*的*将微服务架构应用于企业应用程序*和*第六章*的*Azure Service Fabric*，因为它依赖于这些先前章节中解释的概念。

更具体地说，在本章中，您将学习以下主题：

+   Kubernetes 基础

+   与 Azure Kubernetes 集群交互

+   高级 Kubernetes 概念

通过本章结束时，您将学会如何实现和部署基于 Azure Kubernetes 的完整解决方案。

# 技术要求

+   Visual Studio 2019 免费的 Community Edition 或更高版本，安装了所有数据库工具，或者任何其他`.yaml`文件编辑器，如 Visual Studio Code。

+   免费的 Azure 账户。*第一章*的*创建 Azure 账户*部分解释了如何创建一个。

本章的代码可在[`github.com/PacktPublishing/Software-Architecture-with-C-9-and-.NET-5`](https://github.com/PacktPublishing/Software-Architecture-with-C-9-and-.NET-5)找到。

# Kubernetes 基础

Kubernetes 是一个先进的开源编排器，您可以在私人机器集群上本地安装。在撰写本文时，它是最广泛使用的编排器，因此微软也将其作为 Azure Service Fabric 的更好替代品，因为它目前是*事实上*的标准，并且可以依赖于广泛的工具和应用程序生态系统。本节介绍了基本的 Kubernetes 概念和实体。

Kubernetes 集群是运行 Kubernetes 编排器的虚拟机集群。与 Azure Service Fabric 一样，组成集群的虚拟机被称为节点。我们可以在 Kubernetes 上部署的最小软件单元不是单个应用程序，而是一组容器化的应用程序，称为 pod。虽然 Kubernetes 支持各种类型的容器，但最常用的容器类型是 Docker，我们在*第五章*的*将微服务架构应用于企业应用程序*中进行了分析，因此我们将把讨论限制在 Docker 上。

`pod`很重要，因为属于同一 pod 的应用程序确保在同一节点上运行。这意味着它们可以通过本地主机端口轻松通信。然而，不同 pod 之间的通信更复杂，因为 pod 的 IP 地址是临时资源，因为 pod 没有固定的节点在其上运行，而是由编排器从一个节点移动到另一个节点。此外，为了提高性能，pod 可能会被复制，因此，通常情况下，将消息发送到特定 pod 是没有意义的，而只需发送到同一 pod 的任何相同副本之一即可。

在 Azure Service Fabric 中，基础设施会自动为相同副本组分配虚拟网络地址，而在 Kubernetes 中，我们需要定义显式资源，称为服务，这些服务由 Kubernetes 基础设施分配虚拟地址，并将其通信转发到相同 pod 的集合。简而言之，服务是 Kubernetes 分配常量虚拟地址给 pod 副本集的方式。

所有 Kubernetes 实体都可以分配名称值对，称为标签，用于通过模式匹配机制引用它们。更具体地说，选择器通过列出它们必须具有的标签来选择 Kubernetes 实体。

因此，例如，所有从同一服务接收流量的 pod 都是通过在服务定义中指定的标签来选择的。

服务将其流量路由到所有连接的 Pod 的方式取决于 Pod 的组织方式。无状态的 Pod 被组织在所谓的`ReplicaSets`中，它们类似于 Azure Service Fabric 服务的无状态副本。与 Azure Service Fabric 无状态服务一样，`ReplicaSets`分配给整个组的唯一虚拟地址，并且流量在组中的所有 Pod 之间均匀分配。

有状态的 Kubernetes Pod 副本被组织成所谓的`StatefulSets`。与 Azure Service Fabric 有状态服务类似，`StatefulSets`使用分片将流量分配给它们的所有 Pod。因此，Kubernetes 服务为它们连接的`StatefulSet`的每个 Pod 分配一个不同的名称。这些名称看起来像这样：`basename-0.<base URL>`，`basename-1.<base URL>`，...，`basename-n.<base URL>`。这样，消息分片可以轻松地完成如下：

1.  每次需要将消息发送到由*N*个副本组成的`StatefulSet`时，计算 0 到*N*-1 之间的哈希值，例如`x`。

1.  将后缀`x`添加到基本名称以获取集群地址，例如`basename-x.<base URL>`。

1.  将消息发送到`basename-x.<base URL>`集群地址。

Kubernetes 没有预定义的存储设施，也不能使用节点磁盘存储，因为 Pod 会在可用节点之间移动，因此必须使用分片的云数据库或其他类型的云存储来提供长期存储。虽然`StatefulSet`的每个 Pod 可以使用常规的连接字符串技术访问分片的云数据库，但 Kubernetes 提供了一种技术来抽象外部 Kubernetes 集群环境提供的类似磁盘的云存储。我们将在*高级 Kubernetes 概念*部分中描述这些内容。

在这个简短的介绍中提到的所有 Kubernetes 实体都可以在`.yaml`文件中定义，一旦部署到 Kubernetes 集群中，就会创建文件中定义的所有实体。接下来的子节描述了`.yaml`文件，而随后的其他子节详细描述了到目前为止提到的所有基本 Kubernetes 对象，并解释了如何在`.yaml`文件中定义它们。在整个章节中将描述更多的 Kubernetes 对象。

## .yaml 文件

`.yaml`文件与 JSON 文件一样，是一种以人类可读的方式描述嵌套对象和集合的方法，但它们使用不同的语法。你有对象和列表，但对象属性不用`{}`括起来，列表也不用`[]`括起来。相反，嵌套对象通过简单地缩进其内容来声明。可以自由选择缩进的空格数，但一旦选择了，就必须一致使用。

列表项可以通过在前面加上连字符（`-`）来与对象属性区分开。

以下是涉及嵌套对象和集合的示例：

```cs
Name: Jhon
Surname: Smith
Spouse: 
  Name: Mary
  Surname: Smith
Addresses:
- Type: home
  Country: England
  Town: London
  Street: My home street
- Type: office
  Country: England
  Town: London
  Street: My home street 
```

前面的`Person`对象有一个嵌套的`Spouse`对象和一个嵌套的地址集合。

`.yaml`文件可以包含多个部分，每个部分定义一个不同的实体，它们由包含`---`字符串的行分隔。注释以`#`符号开头，在每行注释前必须重复该符号。

每个部分都以声明 Kubernetes API 组和版本开始。实际上，并不是所有对象都属于同一个 API 组。对于属于`core` API 组的对象，我们可以只指定 API 版本，如下面的示例所示：

```cs
apiVersion: v1 
```

虽然属于不同 API 组的对象也必须指定 API 名称，如下面的示例所示：

```cs
apiVersion: apps/v1 
```

在下一个子节中，我们将详细分析构建在其上的`ReplicaSets`和`Deployments`。

## ReplicaSets 和 Deployments

Kubernetes 应用程序的最重要的构建块是`ReplicaSet`，即一个被复制*n*次的 Pod。然而，通常情况下，您会采用一个更复杂的对象，该对象建立在`ReplicaSet`之上 - `Deployment`。`Deployments`不仅创建`ReplicaSet`，还监视它们以确保副本的数量保持恒定，独立于硬件故障和可能涉及`ReplicaSets`的其他事件。换句话说，它们是一种声明性的定义`ReplicaSets`和 Pod 的方式。

每个`Deployment`都有一个名称（`metadata->name`），一个指定所需副本数量的属性（`spec->replicas`），一个键值对（`spec->selector->matchLabels`）用于选择要监视的 Pod，以及一个模板（`spec->template`），用于指定如何构建 Pod 副本：

```cs
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: my-deployment-name
  namespace: my-namespace #this is optional
spec: 
   replicas: 3
   selector: 
     matchLabels: 
       my-pod-label-name: my-pod-label-value
         ...
   template:
      ... 
```

`namespace`是可选的，如果未提供，则假定为名为`default`的命名空间。命名空间是保持 Kubernetes 集群中对象分开的一种方式。例如，一个集群可以托管两个完全独立的应用程序的对象，每个应用程序都放在一个单独的`namespace`中。

缩进在模板内部是要复制的 Pod 的定义。复杂的对象（如`Deployments`）还可以包含其他类型的模板，例如外部环境所需的类似磁盘的内存的模板。我们将在“高级 Kubernetes 概念”部分进一步讨论这个问题。

反过来，Pod 模板包含一个`metadata`部分，其中包含用于选择 Pod 的标签，以及一个`spec`部分，其中包含所有容器的列表：

```cs
metadata: 
  labels: 
    my-pod-label-name: my-pod-label-value
      ...
spec: 
  containers:
   ...
  - name: my-container-name
    image: <Docker imagename>
    resources: 
      requests: 
        cpu: 100m 
        memory: 128Mi 
      limits: 
        cpu: 250m 
        memory: 256Mi 
    ports: 
    - containerPort: 6379
    env: 
    - name: env-name
      value: env-value
       ... 
```

每个容器都有一个名称，并且必须指定用于创建容器的 Docker 镜像的名称。如果 Docker 镜像不包含在公共 Docker 注册表中，则名称必须是包含存储库位置的 URI。

然后，容器必须指定它们需要创建在`resources->requests`对象中的内存和 CPU 资源。只有在当前可用这些资源的情况下才会创建 Pod 副本。相反，`resources->limits`对象指定容器副本实际可以使用的最大资源。如果在容器执行过程中超过了这些限制，将采取措施限制它们。具体来说，如果超过了 CPU 限制，容器将被限制（其执行将停止以恢复其 CPU 消耗），而如果超过了内存限制，容器将被重新启动。`containerPort`必须是容器暴露的端口。在这里，我们还可以指定其他信息，例如使用的协议。

CPU 时间以毫核表示，其中 1,000 毫核表示 100％的 CPU 时间，而内存以 Mebibytes（*1Mi = 1024*1024 字节*）或其他单位表示。`env`列出了要传递给容器的所有环境变量及其值。

容器和 Pod 模板都可以包含进一步的字段，例如定义虚拟文件的属性和定义返回容器就绪状态和健康状态的命令的属性。我们将在“高级 Kubernetes 概念”部分中分析这些内容。

下面的子部分描述了用于存储状态信息的 Pod 集。

## 有状态集

`StatefulSets`与`ReplicaSet`非常相似，但是`ReplicaSet`的 Pod 是不可区分的处理器，通过负载均衡策略并行地为相同的工作贡献，而`StatefulSet`中的 Pod 具有唯一的标识，并且只能通过分片方式共享相同的工作负载。这是因为`StatefulSets`被设计用于存储信息，而信息无法并行存储，只能通过分片的方式在多个存储之间分割。

出于同样的原因，每个 Pod 实例始终与其所需的任何虚拟磁盘空间绑定在一起（参见“高级 Kubernetes 概念”部分），因此每个 Pod 实例负责向特定存储写入。

此外，`StatefulSets`的 pod 实例附带有序号。它们按照这些序号顺序启动，并按相反的顺序停止。如果`StatefulSet`包含*N*个副本，这些序号从零到*N*-1。此外，通过将模板中指定的 pod 名称与实例序号链接起来，可以获得每个实例的唯一名称，方式如下 - `<pod 名称>-<实例序号>`。因此，实例名称将类似于`mypodname-0`，`mypodname-1`等。正如我们将在*服务*子部分中看到的那样，实例名称用于为所有实例构建唯一的集群网络 URI，以便其他 pod 可以与`StatefulSets`的特定实例通信。

以下是典型的`StatefulSet`定义：

```cs
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-stateful-set-name
spec:
  selector:
    matchLabels:
      my-pod-label-name: my-pod-label-value
...
  serviceName: "my-service-name"
  replicas: 3 
  template:
    ... 
```

模板部分与`Deployments`相同。与`Deployments`的主要概念上的区别是`serviceName`字段。它指定必须与`StatefulSets`连接以为所有 pod 实例提供唯一网络地址的服务的名称。我们将在*服务*子部分中详细讨论这个主题。此外，通常，`StatefulSets`使用某种形式的存储。我们将在*高级 Kubernetes 概念*部分详细讨论这个问题。

值得指出的是，`StatefulSets`的默认有序创建和停止策略可以通过为`spec->podManagementPolicy`属性指定显式的`Parallel`值来更改（默认值为`OrderedReady`）。

下面的子部分描述了如何为`ReplicaSets`和`StatefulSets`提供稳定的网络地址。

## 服务

由于 pod 实例可以在节点之间移动，因此它们没有与之关联的稳定 IP 地址。服务负责为整个`ReplicaSet`分配一个唯一且稳定的虚拟地址，并将流量负载均衡到所有与之连接的实例。服务不是在集群中创建的软件对象，只是为实施其功能所需的各种设置和活动的抽象。

服务在协议栈的第 4 级工作，因此它们理解诸如 TCP 之类的协议，但它们无法执行例如 HTTP 特定的操作/转换，例如确保安全的 HTTPS 连接。因此，如果您需要在 Kubernetes 集群上安装 HTTPS 证书，您需要一个能够在协议栈的第 7 级进行交互的更复杂的对象。`Ingress`对象就是为此而设计的。我们将在下一个子部分中讨论这个问题。

服务还负责为`StatefulSet`的每个实例分配一个唯一的虚拟地址。实际上，有各种类型的服务；一些是为`ReplicaSet`设计的，另一些是为`StatefulSet`设计的。

`ClusterIP`服务类型被分配一个唯一的集群内部 IP 地址。它通过标签模式匹配指定与之连接的`ReplicaSets`或`Deployments`。它使用由 Kubernetes 基础设施维护的表来将接收到的流量在所有与之连接的 pod 实例之间进行负载均衡。

因此，其他 pod 可以通过与分配了稳定网络名称`<service 名称>.<service 命名空间>.svc.cluster.local`的服务进行交互，与连接到该服务的 pod 进行通信。由于它们只分配了本地 IP 地址，因此无法从 Kubernetes 集群外部访问`ClusterIP`服务。以下是典型`ClusterIP`服务的定义：

```cs
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: my-namespace
spec:
  selector:
    my-selector-label: my-selector-value
    ...
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377 
```

每个服务可以在多个端口上工作，并且可以将任何端口（`port`）路由到容器公开的端口（`targetPort`）。但是，很常见的情况是`port = targetPort`。端口可以有名称，但这些名称是可选的。此外，协议的规范是可选的，如果不指定，则允许所有支持的第 4 级协议。`spec->selector`属性指定选择服务要将其接收到的通信路由到的所有名称/值对。

由于无法从 Kubernetes 集群外部访问`ClusterIP`服务，我们需要其他类型的服务来将 Kubernetes 应用程序暴露在公共 IP 地址上。

`NodePort`类型的服务是将 pod 暴露给外部世界的最简单方式。为了实现`NodePort`服务，在 Kubernetes 集群的所有节点上都打开相同的端口`x`，并且每个节点将其接收到的流量路由到一个新创建的`ClusterIP`服务。

反过来，`ClusterIP`服务将其流量路由到服务选择的所有 pod：

![](img/B16756_07_01.png)

图 7.1：NodePort 服务

因此，只需通过任何集群节点的公共 IP 与端口`x`通信，就可以访问与`NodePort`服务连接的 pod。当然，整个过程对开发人员来说是完全自动和隐藏的，他们唯一需要关注的是获取端口号`x`以确定外部流量的转发位置。

`NodePort`服务的定义与`ClusterIP`服务的定义相同，唯一的区别是它们将`spec->type`属性的值设置为`NodePort`。

```cs
...
spec:
  type: NodePort
  selector:
  ... 
```

默认情况下，每个`Service`指定的`targetPort`都会自动选择 30000-327673 范围内的节点端口`x`。对于`NodePortServices`来说，与每个`targetPort`关联的端口属性是无意义的，因为所有流量都通过所选的节点端口`x`传递，并且按照惯例，设置为与`targetPort`相同的值。开发人员还可以通过`nodePort`属性直接设置节点端口`x`：

```cs
...
ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443
      nodePort: 30020
... 
```

当 Kubernetes 集群托管在云中时，将一些 pod 暴露给外部世界的更方便的方式是通过`LoadBalancer`服务，此时 Kubernetes 集群通过所选云提供商的第四层负载均衡器暴露给外部世界。

`LoadBalancer`服务的定义与`ClusterIp`服务相同，唯一的区别是`spec->type`属性必须设置为`LoadBalancer`：

```cs
...
spec:
  type: LoadBalancer
  selector:
  ... 
```

如果没有添加进一步的规范，动态公共 IP 将被随机分配。然而，如果需要特定的公共 IP 地址给云提供商，可以通过在`spec->loadBalancerIP`属性中指定它来用作集群负载均衡器的公共 IP 地址：

```cs
...
spec:
  type: LoadBalancer
  loadBalancerIP: <your public ip>
  selector:
  ... 
```

在 Azure Kubernetes 中，您还必须在注释中指定分配 IP 地址的资源组：

```cs
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-resource-group: <IP resource group name>
  name: my-service name
... 
```

在 Azure Kubernetes 中，您可以保留动态 IP 地址，但可以获得类型为`<my-service-label>.<location>.cloudapp.azure.com`的公共静态域名，其中`<location>`是您为资源选择的地理标签。`<my-service-label>`是一个您验证过使前面的域名唯一的标签。所选标签必须在您的服务的注释中声明，如下所示：

```cs
apiVersion: v1
kind: Service
metadata:
  annotations:
service.beta.kubernetes.io/azure-dns-label-name: <my-service-label>
  name: my-service-name
... 
```

`StatefulSets`不需要任何负载均衡，因为每个 pod 实例都有自己的标识，只需要为每个 pod 实例提供一个唯一的 URL 地址。这个唯一的 URL 由所谓的无头服务提供。无头服务的定义与`ClusterIP`服务相同，唯一的区别是它们的`spec->clusterIP`属性设置为`none`：

```cs
...
spec:
clusterIP: none
  selector:
... 
```

所有由无头服务处理的`StatefulSets`必须将服务名称放置在其`spec->serviceName`属性中，如*StatefulSets*子部分中所述。

无头服务为其处理的所有`StatefulSets` pod 实例提供的唯一名称是`<unique pod name>.<service name>.<namespace>.svc.cluster.local`。

服务只能理解低级协议，如 TCP/IP，但大多数 Web 应用程序位于更复杂的 HTTP 协议上。这就是为什么 Kubernetes 提供了基于服务的更高级实体`Ingresses`。下一小节描述了这些内容，并解释了如何通过级别 7 协议负载均衡器将一组`pods`公开，该负载均衡器可以为您提供典型的 HTTP 服务，而不是通过`LoadBalancer`服务。

## Ingresses

`Ingresses`主要用于使用 HTTP(S)。它们提供以下服务：

+   HTTPS 终止。它们接受 HTTPS 连接并将其路由到云中的任何服务的 HTTP 格式。

+   基于名称的虚拟主机。它们将多个域名与同一个 IP 地址关联，并将每个域或`<domain>/<path prefix>`路由到不同的集群服务。

+   负载均衡。

`Ingresses`依赖于 Web 服务器来提供上述服务。实际上，只有在安装了`Ingress Controller`之后才能使用`Ingresses`。`Ingress Controllers`是必须安装在集群中的自定义 Kubernetes 对象。它们处理 Kubernetes 与 Web 服务器之间的接口，可以是外部 Web 服务器或作为`Ingress Controller`安装的 Web 服务器的一部分。

我们将在*高级 Kubernetes 概念*部分中描述基于 NGINX Web 服务器的`Ingress Controller`的安装，作为使用 Helm 的示例。 *进一步阅读*部分包含有关如何安装与外部 Azure 应用程序网关进行接口的`Ingress Controller`的信息。

HTTPS 终止和基于名称的虚拟主机可以在`Ingress`定义中进行配置，与所选择的`Ingress Controller`无关，而负载均衡的实现方式取决于所选择的特定`Ingress Controller`及其配置。一些`Ingress Controller`配置数据可以通过`Ingress`定义的`metadata->annotations`字段传递。

基于名称的虚拟主机在 Ingress 定义的`spec>rules`部分中定义：

```cs
...
spec:
...
  rules:
  - host: *.mydomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service-name
            port:
              number: 80
  - host: my-subdomain.anotherdomain.com
... 
```

每个规则都指定了一个可选的主机名，可以包含`*`通配符。如果没有提供主机名，则规则匹配所有主机名。对于每个规则，我们可以指定多个路径，每个路径重定向到不同的服务/端口对，其中服务通过其名称引用。与每个`path`的匹配方式取决于`pathType`的值；如果该值为`Prefix`，则指定的`path`必须是任何匹配路径的前缀。否则，如果该值为`Exact`，则匹配必须完全相同。匹配区分大小写。

通过将特定主机名与在 Kubernetes 密钥中编码的证书关联，可以指定特定主机名上的 HTTPS 终止：

```cs
...
spec:
...
  tls:
  - hosts:
      - www.mydomain.com
      secretName: my-certificate1
      - my-subdomain.anotherdomain.com
      secretName: my-certificate2
... 
```

可以免费获取 HTTPS 证书，网址为[`letsencrypt.org/`](https://letsencrypt.org/)。该过程在网站上有详细说明，但基本上，与所有证书颁发机构一样，您提供一个密钥，他们根据该密钥返回证书。还可以安装一个**证书管理器**，它负责自动安装和更新证书。在 Kubernetes 密钥/证书对如何编码为 Kubernetes 密钥的字符串中，详细说明在*高级 Kubernetes 概念*部分中。

整个`Ingress`定义如下所示：

```cs
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-example-ingress
  namespace: my-namespace
spec:
  tls:
  ...
  rules:
... 
```

在这里，`namespace`是可选的，如果未指定，则假定为`default`。

在下一节中，我们将通过定义 Azure Kubernetes 集群并部署一个简单应用程序来实践这里解释的一些概念。

# 与 Azure Kubernetes 集群交互

要创建一个**Azure Kubernetes 服务**（**AKS**）集群，请在 Azure 搜索框中键入`AKS`，选择**Kubernetes 服务**，然后单击**添加**按钮。将显示以下表单：

![](img/B16756_07_02.png)

图 7.2：创建 Kubernetes 集群

值得一提的是，您可以通过将鼠标悬停在任何带有圆圈的“i”上来获取帮助，如上述屏幕截图所示。

与往常一样，您需要指定订阅、资源组和区域。然后，您可以选择一个唯一的名称（Kubernetes 集群名称），以及您想要使用的 Kubernetes 版本。对于计算能力，您需要为每个节点选择一个机器模板（节点大小）和节点数量。初始屏幕显示默认的三个节点。由于三个节点对于 Azure 免费信用来说太多了，我们将其减少为两个。此外，默认虚拟机也应该被更便宜的虚拟机替换，因此单击“更改大小”并选择“DS1 v2”。

“可用区”设置允许您将节点分布在多个地理区域以实现更好的容错性。默认值为三个区域。由于我们只有两个节点，请将其更改为两个区域。

在进行了上述更改后，您应该会看到以下设置：

![](img/B16756_07_03.png)

图 7.3：选择的设置

现在，您可以通过单击“查看+创建”按钮来创建您的集群。应该会出现一个审查页面，请确认并创建集群。

如果您单击“下一步”而不是“查看+创建”，您还可以定义其他节点类型，然后可以提供安全信息，即“服务主体”，并指定是否希望启用基于角色的访问控制。在 Azure 中，服务主体是与您可能用于定义资源访问策略的服务相关联的帐户。您还可以更改默认网络设置和其他设置。

部署可能需要一些时间（10-20 分钟）。之后，您将拥有您的第一个 Kubernetes 集群！在本章结束时，当不再需要该集群时，请不要忘记删除它，以避免浪费您的 Azure 免费信用。

在下一小节中，您将学习如何通过 Kubernetes 的官方客户端 Kubectl 与集群进行交互。

## 使用 Kubectl

创建完集群后，您可以使用 Azure Cloud Shell 与其进行交互。单击 Azure 门户页面右上角的控制台图标。以下屏幕截图显示了 Azure Shell 图标：

![](img/B16756_07_04.png)

图 7.4：Azure Shell 图标

在提示时，选择“Bash Shell”。然后，您将被提示创建一个存储帐户，确认并创建它。

我们将使用此 Shell 与我们的集群进行交互。在 Shell 的顶部有一个文件图标，我们将使用它来上传我们的`.yaml`文件：

![](img/B16756_07_05.png)

图 7.5：如何在 Azure Cloud Shell 中上传文件

还可以下载一个名为 Azure CLI 的客户端，并在本地机器上安装它（参见[`docs.microsoft.com/en-US/cli/azure/install-azure-cli`](https://docs.microsoft.com/en-US/cli/azure/install-azure-cli)），但在这种情况下，您还需要安装与 Kubernetes 集群交互所需的所有工具（Kubectl 和 Helm），这些工具已预先安装在 Azure Cloud Shell 中。

创建 Kubernetes 集群后，您可以通过`kubectl`命令行工具与其进行交互。`kubectl`已集成在 Azure Shell 中，因此您只需激活集群凭据即可使用它。您可以使用以下 Cloud Shell 命令来完成此操作：

```cs
az aks get-credentials --resource-group <resource group> --name <cluster name> 
```

上述命令将凭据存储在`/.kube/config`配置文件中，该凭据是自动创建的，以便您与集群进行交互。从现在开始，您可以无需进一步身份验证即可发出`kubectl`命令。

如果您发出`kubectl get nodes`命令，您将获得所有 Kubernetes 节点的列表。通常，`kubectl get <对象类型>`列出给定类型的所有对象。您可以将其与`nodes`、`pods`、`statefulset`等一起使用。`kubectl get all`显示在您的集群中创建的所有对象的列表。如果您还添加了特定对象的名称，您将只获取该特定对象的信息，如下所示：

```cs
kubectl get <object type><object name> 
```

如果添加`--watch`选项，对象列表将持续更新，因此您可以看到所有选定对象的状态随时间变化。您可以通过按下 Ctrl + c 来退出此观察状态。

以下命令显示了有关特定对象的详细报告：

```cs
kubectl describe <object name> 
```

可以使用以下命令创建在`.yaml`文件中描述的所有对象，例如`myClusterConfiguration.yaml`：

```cs
kubectl create -f myClusterConfiguration.yaml 
```

然后，如果您修改了`.yaml`文件，可以使用`apply`命令在集群上反映所有修改，如下所示：

```cs
kubectl apply -f myClusterConfiguration.yaml 
```

`apply`执行与`create`相同的工作，但如果资源已经存在，`apply`会覆盖它，而`create`则会显示错误消息。

您可以通过将相同的文件传递给`delete`命令来销毁使用`.yaml`文件创建的所有对象，如下所示：

```cs
kubectl delete -f myClusterConfiguration.yaml 
```

`delete`命令还可以传递对象类型和要销毁的该类型对象的名称列表，如下例所示：

```cs
kubectl delete deployment deployment1 deployment2... 
```

上述`kubectl`命令应足以满足大部分实际需求。有关更多详细信息，请参阅*Further reading*部分中的官方文档链接。

在下一小节中，我们将使用`kubectl create`安装一个简单的演示应用程序。

## 部署演示 Guestbook 应用程序

Guestbook 应用程序是官方 Kubernetes 文档示例中使用的演示应用程序。我们将使用它作为 Kubernetes 应用程序的示例，因为它的 Docker 镜像已经在公共 Docker 存储库中可用，所以我们不需要编写软件。

Guestbook 应用程序存储了访问酒店或餐厅的客户的意见。它由一个使用`Deployment`实现的 UI 层和一个使用基于 Redis 的内存存储实现的数据库层组成。而 Redis 存储则是由一个用于写入/更新的唯一主存储和几个只读副本组成，这些副本始终基于 Redis，并实现了读取并行性。写入/更新并行性可以通过多个分片的 Redis 主节点来实现，但由于应用程序的特性，写入操作不应占主导地位，因此在实际情况下，单个主数据库应该足够满足单个餐厅/酒店的需求。整个应用程序由三个`.yaml`文件组成，您可以在与本书相关的 GitHub 存储库中找到。

以下是包含在`redis-master.yaml`文件中的基于 Redis 的主存储的代码：

```cs
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: redis-master
  labels:
    app: redis
spec:
  selector:
    matchLabels:
      app: redis
      role: master
      tier: backend
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
        role: master
        tier: backend
    spec:
      containers:
      - name: master
        image: k8s.gcr.io/redis:e2e
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
    tier: backend
spec:
  ports:
  - port: 6379
    targetPort: 6379
  selector:
    app: redis
    role: master
    tier: backend 
```

该文件由两个对象定义组成，由一个只包含`---`的行分隔，即`.yaml`文件的对象定义分隔符。第一个对象是一个具有单个副本的`Deployment`，第二个对象是一个`ClusterIPService`，它在内部`redis-master.default.svc.cluster.local`网络地址上的`6379`端口上公开`Deployment`。`Deployment pod template`定义了三个`app`、`role`和`tier`标签及其值，这些值在 Service 的`selector`定义中用于将 Service 与在`Deployment`中定义的唯一 pod 连接起来。

让我们将`redis-master.yaml`文件上传到 Cloud Shell，然后使用以下命令在集群中部署它：

```cs
kubectl create -f redis-master.yaml 
```

操作完成后，您可以使用`kubectl get all`命令检查集群的内容。

从`redis-slave.yaml`文件中定义了从存储，它与主存储完全类似，唯一的区别是这次有两个副本和不同的 Docker 镜像。

让我们也上传此文件，并使用以下命令部署它：

```cs
kubectl create -f redis-slave.yaml 
```

UI 层的代码包含在`frontend.yaml`文件中。`Deployment`有三个副本和不同的服务类型。让我们使用以下命令上传并部署此文件：

```cs
kubectl create -f frontend.yaml 
```

值得分析的是`frontend.yaml`文件中的服务代码：

```cs
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: guestbook
    tier: frontend 
```

这种类型的服务属于`LoadBalancer`类型，因为它必须在公共 IP 地址上公开应用程序。为了获取分配给服务和应用程序的公共 IP 地址，请使用以下命令：

```cs
kubectl get service 
```

前面的命令应该显示所有已安装服务的信息。您应该在列表的`EXTERNAL-IP`列下找到公共 IP。如果您只看到`<none>`的值，请重复该命令，直到公共 IP 地址分配给负载均衡器。

一旦获得 IP 地址，使用浏览器导航到该地址。应用程序的主页现在应该显示出来了！

在完成对应用程序的实验后，使用以下命令从集群中删除应用程序，以避免浪费您的 Azure 免费信用额度（公共 IP 地址需要付费）：

```cs
kubectl delete deployment frontend redis-master redis-slave 
kubectl delete service frontend redis-master redis-slave 
```

在下一节中，我们将分析其他重要的 Kubernetes 功能。

# 高级 Kubernetes 概念

在本节中，我们将讨论其他重要的 Kubernetes 功能，包括如何为`StatefulSets`分配永久存储，如何存储密码、连接字符串或证书等秘密信息，容器如何通知 Kubernetes 其健康状态，以及如何使用 Helm 处理复杂的 Kubernetes 包。所有主题都按照专门的子部分进行组织。我们将从永久存储的问题开始。

## 需要永久存储

由于 Pod 会在节点之间移动，它们不能依赖于当前运行它们的节点提供的永久存储。这给我们留下了两个选择：

1.  **使用外部数据库**：借助数据库，`ReplicaSets`也可以存储信息。然而，如果我们需要更好的写入/更新操作性能，我们应该使用基于非 SQL 引擎（如 Cosmos DB 或 MongoDB）的分布式分片数据库（参见*第九章*，*如何在云中选择数据存储*）。在这种情况下，为了充分利用表分片，我们需要使用`StatefulSets`，其中每个`pod`实例负责不同的表分片。

1.  **使用云存储**：由于不与物理集群节点绑定，云存储可以永久关联到`StatefulSets`的特定 Pod 实例。

由于访问外部数据库不需要任何特定于 Kubernetes 的技术，而是可以使用通常的连接字符串来完成，我们将集中讨论云存储。

Kubernetes 提供了一个名为**PersistentVolumeClaim**（PVC）的存储抽象，它独立于底层存储提供商。更具体地说，PVC 是分配请求，可以与预定义资源匹配或动态分配。当 Kubernetes 集群在云中时，通常使用由云提供商安装的动态提供商进行动态分配。

Azure 等云提供商提供具有不同性能和不同成本的不同存储类。此外，PVC 还可以指定`accessMode`，可以是：

+   `ReadWriteOnce` - 卷可以被单个 Pod 挂载为读写。

+   `ReadOnlyMany` - 卷可以被多个 Pod 挂载为只读。

+   `ReadWriteMany` - 卷可以被多个 Pod 挂载为读写。

卷声明可以添加到`StatefulSets`的特定`spec->volumeClaimTemplates`对象中：

```cs
volumeClaimTemplates:
-  metadata:
   name: my-claim-template-name
spec:
  resources:
    request:
      storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  storageClassName: my-optional-storage-class 
```

`storage`属性包含存储需求。将`volumeMode`设置为`Filesystem`是一种标准设置，表示存储将作为文件路径可用。另一个可能的值是`Block`，它将内存分配为`未格式化`。`storageClassName`必须设置为云提供商提供的现有存储类。如果省略，则将假定默认存储类。

可以使用以下命令列出所有可用的存储类：

```cs
kubectl get storageclass 
```

一旦`volumeClaimTemplates`定义了如何创建永久存储，那么每个容器必须指定将永久存储附加到的文件路径，位于`spec->containers->volumeMounts`属性中。

```cs
...
volumeMounts
- name: my-claim-template-name
  mountPath: /my/requested/storage
  readOnly: false
... 
```

在这里，`name`必须与 PVC 的名称相对应。

以下子节显示了如何使用 Kubernetes secrets。

## Kubernetes secrets

秘密是一组键值对，它们被加密以保护它们。可以通过将每个值放入文件中，然后调用以下`kubectl`命令来创建它们：

```cs
kubectl create secret generic my-secret-name \
  --from-file=./secret1.bin \
  --from-file=./secret2.bin 
```

在这种情况下，文件名成为键，文件内容成为值。

当值为字符串时，可以直接在`kubectl`命令中指定，如下所示：

```cs
kubectl create secret generic dev-db-secret \
  --from-literal=username=devuser \
  --from-literal=password=sdsd_weew1' 
```

在这种情况下，键和值按顺序列出，由`=`字符分隔。

定义后，可以在 pod（`Deployment`或`StatefulSettemplate`）的`spec->volume`属性中引用 secrets，如下所示：

```cs
...
volumes:
  - name: my-volume-with-secrets
    secret:
      secretName: my-secret-name
... 
```

之后，每个容器可以在`spec->containers->volumeMounts`属性中指定要将它们挂载到的路径：

```cs
...
volumeMounts:
    - name: my-volume-with-secrets
      mountPath: "/my/secrets"
      readOnly: true
... 
```

在上面的示例中，每个键被视为具有与键相同名称的文件。文件的内容是秘密值，经过 base64 编码。因此，读取每个文件的代码必须解码其内容（在.NET 中，`Convert.FromBase64`可以完成这项工作）。

当 secrets 包含字符串时，它们也可以作为环境变量传递给`spec->containers->env object`：

```cs
env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: dev-db-secret
          key: username
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
          name: dev-db-secret
          key: password 
```

在这里，`name`属性必须与 secret 的`name`匹配。当容器托管 ASP.NET Core 应用程序时，将 secrets 作为环境变量传递非常方便，因为在这种情况下，环境变量立即在配置对象中可用（请参阅*第十五章*的*加载配置数据并与选项框架一起使用*部分，*介绍 ASP.NET Core MVC*）。

Secrets 还可以使用以下`kubectl`命令对 HTTPS 证书的密钥/证书对进行编码：

```cs
kubectl create secret tls test-tls --key="tls.key" --cert="tls.crt" 
```

以这种方式定义的 secrets 可用于在`Ingresses`中启用 HTTPS 终止。只需将 secret 名称放置在`spec->tls->hosts->secretName`属性中即可。

## 活跃性和就绪性检查

Kubernetes 会自动监视所有容器，以确保它们仍然存活，并且将资源消耗保持在`spec->containers->resources->limits`对象中声明的限制范围内。当某些条件被违反时，容器要么被限制，要么被重新启动，要么整个 pod 实例在不同的节点上重新启动。Kubernetes 如何知道容器处于健康状态？虽然它可以使用操作系统来检查节点的健康状态，但它没有适用于所有容器的通用检查。

因此，容器本身必须通知 Kubernetes 它们的健康状态，否则 Kubernetes 必须放弃验证它们。容器可以通过两种方式通知 Kubernetes 它们的健康状态，一种是声明返回健康状态的控制台命令，另一种是声明提供相同信息的端点。

这两个声明都在`spec->containers->livenessProb`对象中提供。控制台命令检查声明如下所示：

```cs
...
  livenessProbe:
    exec:
      command:
      - cat
      - /tmp/healthy
    initialDelaySeconds: 10
    periodSeconds: 5
 ... 
```

如果`command`返回`0`，则容器被视为健康。在上面的示例中，我们假设在容器中运行的软件将其健康状态记录在`/tmp/healthy`文件中，因此`cat/tmp/healthy`命令返回它。`PeriodSeconds`是检查之间的时间，而`initialDelaySeconds`是执行第一次检查之前的初始延迟。始终需要初始延迟，以便给容器启动的时间。

端点检查非常类似：

```cs
...
  livenessProbe:
    exec:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
          - name: Custom-Health-Header
          value: container-is-ok
    initialDelaySeconds: 10
    periodSeconds: 5
 ... 
```

如果 HTTP 响应包含声明的标头和声明的值，则测试成功。您还可以使用纯 TCP 检查，如下所示：

```cs
...
  livenessProbe:
    exec:
      tcpSocket:
        port: 8080
    initialDelaySeconds: 10
    periodSeconds: 5
 ... 
```

在这种情况下，如果 Kubernetes 能够在声明的端口上打开与容器的 TCP 套接字，则检查成功。

类似地，一旦安装了容器，就会使用就绪性检查来监视容器的就绪性。就绪性检查的定义方式与活跃性检查完全相同，唯一的区别是将`livenessProbe`替换为`readinessProbe`。

以下小节解释了如何自动缩放`Deployments`。

## 自动缩放

与手动修改`Deployment`中的副本数以适应负载的减少或增加不同，我们可以让 Kubernetes 自行决定副本的数量，试图保持声明的资源消耗恒定。因此，例如，如果我们声明目标为 10%的 CPU 消耗，当每个副本的平均资源消耗超过此限制时，将创建一个新副本，而如果平均 CPU 低于此限制，则销毁一个副本。用于监视副本的典型资源是 CPU 消耗，但我们也可以使用内存消耗。

通过定义`HorizontalPodAutoscaler`对象来实现自动缩放。以下是`HorizontalPodAutoscaler`定义的示例：

```cs
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: my-autoscaler
spec:
  scaleTargetRef:
    apiVersion: extensions/v1beta1
    kind: Deployment
    name: my-deployment-name
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 25 
```

`spec-> scaleTargetRef->name`指定要自动缩放的`Deployment`的名称，而`targetAverageUtilization`指定目标资源（在我们的情况下是 CPU）的使用百分比（在我们的情况下是 25%）。

以下小节简要介绍了 Helm 软件包管理器和 Helm 图表，并解释了如何在 Kubernetes 集群上安装 Helm 图表。给出了安装`Ingress Controller`的示例。

## Helm - 安装 Ingress Controller

Helm 图表是组织安装包含多个`.yaml`文件的复杂 Kubernetes 应用程序的一种方式。Helm 图表是一组`.yaml`文件，组织成文件夹和子文件夹。以下是从官方文档中获取的 Helm 图表的典型文件夹结构：

![](img/B16756_07_06.png)

图 7.6：Helm 图表的文件夹结构

特定于应用程序的`.yaml`文件放置在顶级`templates`目录中，而`charts`目录可能包含其他用作辅助库的 Helm 图表。顶级`Chart.yaml`文件包含有关软件包（名称和描述）的一般信息，以及应用程序版本和 Helm 图表版本。以下是典型示例：

```cs
apiVersion: v2
name: myhelmdemo
description: My Helm chart
type: application
version: 1.3.0
appVersion: 1.2.0 
```

在这里，`type`可以是`application`或`library`。只能部署`application`图表，而`library`图表是用于开发其他图表的实用程序。`library`图表放置在其他 Helm 图表的`charts`文件夹中。

为了配置每个特定应用程序的安装，Helm 图表`.yaml`文件包含在安装 Helm 图表时指定的变量。此外，Helm 图表还提供了一个简单的模板语言，允许仅在满足取决于输入变量的条件时包含一些声明。顶级`values.yaml`文件声明了输入变量的默认值，这意味着开发人员只需要指定几个需要与默认值不同的变量。我们不会描述 Helm 图表模板语言，但您可以在*进一步阅读*部分中找到官方 Helm 文档。

Helm 图表通常以与 Docker 镜像类似的方式组织在公共或私有存储库中。有一个 Helm 客户端，您可以使用它从远程存储库下载软件包，并在 Kubernetes 集群中安装图表。Helm 客户端立即在 Azure Cloud Shell 中可用，因此您可以开始在 Azure Kubernetes 集群中使用 Helm，而无需安装它。

在使用其软件包之前，必须添加远程存储库，如下例所示：

```cs
helm repo add <my-repo-local-name> https://kubernetes-charts.storage.googleapis.com/ 
```

上述命令使远程存储库的软件包可用，并为其指定本地名称。之后，可以使用以下命令安装远程存储库的任何软件包：

```cs
helm install <instance name><my-repo-local-name>/<package name> -n <namespace> 
```

在这里，`<namespace>`是要安装应用程序的命名空间。通常情况下，如果未提供，则假定为`default`命名空间。`<instance name>`是您为安装的应用程序指定的名称。您需要此名称才能使用以下命令获取有关已安装应用程序的信息：

```cs
helm status <instance name> 
```

您还可以使用以下命令获取使用 Helm 安装的所有应用程序的信息：

```cs
helm ls 
```

还需要应用程序名称来通过以下命令从集群中删除应用程序：

```cs
helm delete <instance name> 
```

当我们安装应用程序时，我们还可以提供一个包含要覆盖的所有变量值的`.yaml`文件。我们还可以指定 Helm 图表的特定版本，否则将假定为最新版本。以下是一个同时覆盖版本和值的示例：

```cs
helm install <instance name><my-repo-local-name>/<package name> -f  values.yaml –version <version> 
```

最后，值覆盖也可以通过`--set`选项提供，如下所示：

```cs
...--set <variable1>=<value1>,<variable2>=<value2>... 
```

我们还可以使用`upgrade`命令升级现有的安装，如下所示：

```cs
helm upgrade <instance name><my-repo-local-name>/<package name>... 
```

`upgrade`命令可以使用`-f`选项或`--set`选项指定新的值覆盖，并且可以使用`--version`指定新版本。

让我们使用 Helm 为 guestbook 演示应用程序提供一个`Ingress`。更具体地说，我们将使用 Helm 安装一个基于 Nginx 的`Ingress-Controller`。要遵循的详细过程如下：

1.  添加远程存储库：

```cs
helm repo add gcharts https://kubernetes-charts.storage.googleapis.com/ 
```

1.  安装`Ingress-Controller`：

```cs
helm install ingress gcharts/nginx-ingress 
```

1.  安装完成后，如果键入`kubectl get service`，您应该在已安装的服务中看到已安装的`Ingress-Controller`的条目。该条目应包含一个公共 IP。请记下此 IP，因为它将是应用程序的公共 IP。

1.  打开`frontend.yaml`文件并删除`type: LoadBalancer`行。保存并上传到 Azure Cloud Shell。我们将前端应用程序的服务类型从`LoadBalancer`更改为`ClusterIP`（默认）。此服务将连接到您将要定义的新 Ingress。

1.  使用`kubectl`部署`redis-master.yaml`，`redis-slave.yaml`和`frontend.yaml`，如*部署演示 Guestbook 应用程序*子部分所述。创建一个`frontend-ingress.yaml`文件，并将以下代码放入其中：

```cs
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: simple-frontend-ingress
spec:
  rules:
  - http:
      paths:
      - path:/
        backend:
          serviceName: frontend
          servicePort: 80 
```

1.  将`frontend-ingress.yaml`上传到 Cloud Shell，并使用以下命令部署它：

```cs
kubectl apply -f frontend-ingress.yaml 
```

1.  打开浏览器并导航到您在*步骤 3*中注释的公共 IP。在那里，您应该看到应用程序正在运行。

由于分配给`Ingress-Controller`的公共 IP 在 Azure 的*公共 IP 地址*部分中可用（使用 Azure 搜索框找到它），您可以在那里检索它，并为其分配一个类型为`<chosen name>.<your Azure region>.cloudeapp.com`的主机名。

鼓励您为应用程序公共 IP 分配一个主机名，然后使用此主机名从[`letsencrypt.org/`](https://letsencrypt.org/)获取免费的 HTTPS 证书。获得证书后，可以使用以下命令从中生成一个密钥：

```cs
kubectl create secret tls guestbook-tls --key="tls.key" --cert="tls.crt" 
```

然后，您可以通过将前面的密钥添加到`frontend-ingress.yamlIngress`中，将以下`spec->tls`部分添加到其中：

```cs
...
spec:
...
  tls:
  - hosts:
      - <chosen name>.<your Azure region>.cloudeapp.com
secretName: guestbook-tls 
```

进行更正后，将文件上传到 Azure Cloud Shell，并使用以下内容更新先前的`Ingress`定义：

```cs
kubectl apply frontend-ingress.yaml 
```

此时，您应该能够通过 HTTPS 访问 Guestbook 应用程序。

当您完成实验时，请不要忘记从集群中删除所有内容，以避免浪费您的免费 Azure 信用额度。您可以通过以下命令来完成：

```cs
kubectl delete frontend-ingress.yaml
kubectl delete frontend.yaml
kubectl delete redis-slave.yaml
kubectl delete redis-master.yaml
helm delete ingress 
```

# 摘要

在本章中，我们介绍了 Kubernetes 的基本概念和对象，然后解释了如何创建 Azure Kubernetes 集群。我们还展示了如何部署应用程序，以及如何使用简单的演示应用程序监视和检查集群的状态。

本章还介绍了更高级的 Kubernetes 功能，这些功能在实际应用程序中起着基本作用，包括如何为在 Kubernetes 上运行的容器提供持久存储，如何通知 Kubernetes 容器的健康状态，以及如何提供高级 HTTP 服务，如 HTTPS 和基于名称的虚拟主机。

最后，我们回顾了如何使用 Helm 安装复杂的应用程序，并对 Helm 和 Helm 命令进行了简短的描述。

在下一章中，您将学习如何使用 Entity Framework 将.NET 应用程序与数据库连接。

# 问题

1.  为什么需要服务（Services）？

1.  为什么需要`Ingress`？

1.  为什么需要 Helm？

1.  是否可以在同一个`.yaml`文件中定义多个 Kubernetes 对象？如果可以，如何操作？

1.  Kubernetes 如何检测容器故障？

1.  为什么需要持久卷索赔（Persistent Volume Claims）？

1.  `ReplicaSet`和`StatefulSet`之间有什么区别？

# 进一步阅读

+   一个很好的书籍，可以扩展在本章中获得的知识，是以下这本：[`www.packtpub.com/product/hands-on-kubernetes-on-azure-second-edition/9781800209671`](https://www.packtpub.com/product/hands-on-kubernetes-on-azure-second-edition/9781800209671)。

+   Kubernetes 和`.yaml`文件的官方文档可以在这里找到：[`kubernetes.io/docs/home/`](https://kubernetes.io/docs/home/)。

+   有关 Helm 和 Helm charts 的更多信息可以在官方文档中找到。这些文档写得非常好，包含一些很好的教程：[`helm.sh/`](https://helm.sh/)。

+   Azure Kubernetes 的官方文档可以在这里找到：[`docs.microsoft.com/en-US/azure/aks/`](https://docs.microsoft.com/en-US/azure/aks/)。

+   基于 Azure 应用程序网关的`Ingress Controller`的官方文档可以在这里找到：[`github.com/Azure/application-gateway-kubernetes-ingress`](https://github.com/Azure/application-gateway-kubernetes-ingress)。

+   `Ingress`证书的发布和更新可以按照这里的说明进行自动化：[`docs.microsoft.com/en-us/azure/application-gateway/ingress-controller-letsencrypt-certificate-application-gateway`](https://docs.microsoft.com/en-us/azure/application-gateway/ingress-controller-letsencrypt-certificat)。虽然该过程指定了基于 Azure 应用程序网关的入口控制器，但适用于任何`Ingress Controller`。
