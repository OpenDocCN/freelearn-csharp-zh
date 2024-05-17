# 第六章：探索图

在上一章中，您了解了树。但是，您知道这样的数据结构也属于图吗？但图是什么，以及您如何在应用程序中使用它？您可以在本章中找到这些问题的答案以及许多其他问题的答案！

在开始时，将介绍有关图的基本信息，包括节点和边的解释。此外，您将看到有向和无向边之间的区别，以及加权和非加权边之间的区别。由于图是常用的数据结构，您还将看到一些应用，例如在社交媒体中存储朋友的数据或在城市中寻找道路。然后，将介绍图的表示主题，即使用邻接表和矩阵。

在这个简短的介绍之后，您将学习如何在 C#语言中实现图。这项任务涉及到声明一些类，例如节点和边。整个必要的代码将在本章中详细描述。

此外，您还将有机会阅读两种图遍历模式的描述，即深度优先和广度优先搜索。对于两者，将显示 C#代码和详细描述。

下一部分将介绍最小生成树的主题，以及用于创建它们的两种算法，即 Kruskal 和 Prim。这些算法将以文本描述、基于 C#的代码片段以及易于理解的插图的形式呈现。此外，还将提供一个实际的示例应用程序。

另一个有趣的与图相关的问题是节点的着色，这将在本章的后面部分中考虑。最后，将使用 Dijkstra 算法分析在图中找到最短路径的主题。当然，还将展示一个实际的示例应用程序，以及基于 C#的实现。

正如您所看到的，图的主题涉及许多有趣的问题，本书只提到了其中一些。但是，所选择的主题适合在基于 C#的实现的背景下呈现各种与图相关的方面。您准备好深入研究图的主题了吗？如果是这样，请开始阅读本章！

在本章中，将涵盖以下主题：

+   图的概念

+   应用

+   表示

+   实施

+   遍历

+   最小生成树

+   着色

+   最短路径

# 图的概念

让我们从问题“什么是图？”开始。广义上说，图是由节点（也称为顶点）和边组成的数据结构。每条边连接两个节点。图数据结构不需要关于节点之间连接的任何特定规则，如下图所示：

![](img/a2f956e4-ed60-447e-961e-b32be6c7719d.png)

前面提到的概念似乎非常简单，不是吗？让我们尝试分析前面的图以消除任何疑虑。它包含九个节点，值在 1 和 9 之间。这些节点由 11 条边连接，例如节点 2 和 4 之间。此外，图可以包含循环，例如由节点 2、3 和 4 表示的循环，以及未连接在一起的单独节点组。但是，关于父节点和子节点的主题呢？这是您从树的学习中了解的。由于图中没有关于连接的特定规则，因此在这种情况下不使用这些概念。

图还可以包含自环。每个自环都是连接给定节点与自身的边。但是，这样的主题超出了本书的范围，并且在本章中显示的示例中没有考虑。

对于图中的边，还需要一些额外的说明。在前面的图中，你可以看到一个所有节点都通过**无向边**连接的图，也就是**双向边**。它们表示可以在两个方向之间旅行，例如，从节点**2**到**3**，从节点**3**到**2**。这些边在图形上呈现为直线。当一个图包含无向边时，它是一个**无向图**。

然而，当你需要表明节点之间的旅行只能单向进行时怎么办？在这种情况下，你可以使用**有向边**，也就是**单向边**，在图形上呈现为带箭头的直线，箭头表示边的方向。如果一个图包含有向边，它可以被称为**有向图**。

一个有向图的例子在右侧的下图中展示，而一个无向图在左侧展示：

![](img/78f53bad-97f1-4052-bb70-124a43cdfbf3.png)

简单解释一下，在前面图中右侧的有向图包含了 8 个节点，通过 15 条单向边连接。例如，它们表示可以在节点**1**和**2**之间双向旅行，但是只能从节点**1**到**3**单向旅行，所以无法直接从**3**到**1**。

无向和有向边之间的区分并不是唯一的。你还可以为特定的边指定**权重**（也称为**成本**），以表示节点之间旅行的成本。当然，这样的权重可以分配给无向和有向边。如果提供了权重，边被称为**加权边**，整个图被称为**加权图**。同样，如果没有提供权重，图中使用**无权重边**，可以称为**无权重图**。

下图展示了带有无向（左侧）和有向（右侧）边的例子加权图：

![](img/4a125b36-2efb-407e-8d60-b3cc03b89af7.png)

加权边的图形表示只是在线旁边添加了边的权重。例如，在前面图中左侧的无向图中，从节点**1**到**2**的旅行成本，以及从节点**2**到**1**的成本都等于**3**。在有向图的情况下（右侧），情况稍微复杂一些。在这里，你可以从节点**1**到**2**旅行的成本等于**9**，而在相反方向（从节点**2**到**1**）旅行的成本要便宜得多，只有**3**。

# 应用

简短介绍之后，你已经了解了一些关于图的基本信息，特别是关于节点和各种边的信息。但是，为什么图的主题如此重要，为什么它在这本书中占据了一个完整的章节？你的应用程序可以使用这种数据结构吗？答案是显而易见的：可以！在解决各种算法问题和有许多现实世界应用中，图通常被使用。下面的图表中展示了两个例子。

首先，让我们考虑社交媒体中可用的朋友结构。每个用户都有许多联系人，但他们也有许多朋友，等等。你应该选择什么数据结构来存储这样的数据？图是最简单的答案之一。在这种情况下，节点代表联系人，而边表示人与人之间的关系。例如，让我们看一个无向且无权重图的下图：

![](img/64331293-09c6-49c6-b1f7-979bf9cd68c4.png)

正如你所看到的，**Jimmy Stewart**有五个联系人，分别是**John Smith**，**Andy Wood**，**Eric Green**，**Ashley Lopez**和**Paula Scott**。与此同时，**Paula Scott**还有另外两个朋友：**Marcin Jamro**和**Tommy Butler**。通过使用图作为数据结构，你可以轻松地检查两个人是否是朋友，或者他们是否有共同的联系人。

图的另一个常见应用涉及搜索最短路径的问题。想象一下一个程序，它应该找到城市中两点之间的路径，考虑到行驶特定道路所需的时间。在这种情况下，你可以使用图来表示城市的地图，其中节点表示交叉路口，边表示道路。当然，你应该为边分配权重，以指示行驶给定道路所需的时间。搜索最短路径的主题可以理解为找到从源节点到目标节点的边的列表，其总成本最小。基于图的城市地图的图表如下所示：

![](img/65f4d4d1-604d-461d-89d8-745660ef781e.png)

正如你所看到的，选择了有向加权图。有向边的应用使得支持双向和单向道路成为可能，而加权边允许指定在两个交叉路口之间行驶所需的时间。

# 表示

现在你知道了图是什么，以及它何时可以使用，但是你如何在计算机的内存中表示它呢？解决这个问题有两种流行的方法，即使用**邻接表**和**邻接矩阵**。这两种方法将在接下来的部分中详细描述。

# 邻接表

第一种方法要求你通过指定其邻居的列表来扩展一个节点的数据。因此，你可以通过迭代给定节点的邻接表来轻松获取给定节点的所有邻居。这样的解决方案是空间高效的，因为你只存储相邻边的数据。让我们看一下下面的图表：

![](img/8b5077b4-5ac8-4d23-b617-5de2c40cdb61.png)

示例图包含 8 个节点和 10 条边。对于每个节点，创建了一个相邻节点（即邻居）的列表，如图表右侧所示。例如，节点**1**有两个邻居，即节点**2**和**3**，而节点**5**有四个邻居，即节点**4**，**6**，**7**和**8**。正如你所看到的，基于邻接表的无向无权图的表示非常直接，易于使用、理解和实现。

然而，在有向图的情况下，邻接表是如何工作的呢？答案是显而易见的，因为分配给每个节点的列表只显示可以从给定节点到达的相邻节点。示例图如下所示：

![](img/a9587b8b-ee85-43e5-a6e1-c72e45ee7320.png)

让我们看一下节点**3**。在这里，邻接表只包含一个元素，即节点**4**。节点**1**没有包括在内，因为它不能直接从节点**3**到达。

在加权图的情况下可能需要更多的解释。在这种情况下，还需要存储特定边的权重。你可以通过扩展邻接表中存储的数据来实现这个目标，如下图所示：

![](img/3c37de49-bcb7-4f69-9509-2832cb82e959.png)

节点**7**的邻接表包含两个元素，即指向节点**5**的边（权重为**4**）和指向节点**8**的边（权重为**6**）。

# 邻接矩阵

图的另一种表示方法涉及邻接矩阵，它使用二维数组来显示哪些节点通过边相连。矩阵包含相同数量的行和列，即节点的数量。主要思想是在矩阵的给定行和列中存储关于特定边的信息。行和列的索引取决于与边相连的节点。例如，如果你想获取索引为 1 和 5 的节点之间边的信息，你应该检查矩阵中索引为 1 的行和索引为 5 的列的元素。

这样的解决方案可以快速检查两个特定节点是否通过边相连。然而，它可能需要存储的数据比邻接表要多得多，特别是如果图中节点之间的边不多的话。

首先，让我们分析无向无权图的基本情况。在这种情况下，邻接矩阵可能只存储布尔值。在第`i`行和第`j`列的元素中放置的`true`值表示索引为`i`的节点和索引为`j`的节点之间存在连接。如果听起来很复杂，看看下面的例子：

![](img/94179e3f-bede-42b6-b542-7b579cf89c08.png)

在这里，邻接矩阵包含 64 个元素（八行八列），因为图中有八个节点。数组中许多元素的值设置为`false`，表示缺失指示。其余的用叉号表示`true`值。例如，在第四行和第三列的元素中的这样一个值表示图中节点**4**和**3**之间有一条边，如前面图中所示。

由于所呈现的图是无向的，邻接矩阵是对称的。如果节点`i`和`j`之间有一条边，那么节点`j`和`i`之间也有一条边。

下一个例子涉及有向无权图。在这种情况下，可以使用相同的规则，但邻接矩阵不需要对称。让我们看一下下图所示的图和邻接矩阵：

![](img/f1c3511f-6e80-4aa4-8199-fbb7c4ed7a53.png)

在所示的邻接矩阵中，你可以找到 15 条边的数据，用 15 个带有`true`值的元素表示，矩阵中用叉号表示。例如，从节点**5**到**4**的单向边在矩阵的第五行和第四列处用叉号表示。

在前面的两个例子中，你已经学会了如何使用邻接矩阵来表示无权图。然而，你如何存储加权图的数据，无论是无向还是有向的？答案很简单——你只需要将邻接矩阵中特定元素存储的数据类型从布尔值改为数字。因此，你可以指定边的权重，如下图所示：

![](img/5ac401e9-75be-451b-b89b-1df9a325af0c.png)

前面的图和邻接矩阵都是不言自明的。然而，为了消除任何疑惑，让我们看一下节点**5**和**6**之间权重为**2**的边。这样的边由矩阵中第五行和第六列的元素表示。元素的值等于这些节点之间的旅行成本。

# 实现

你已经了解了一些关于图的基本信息，包括节点、边以及邻接表和邻接矩阵两种表示方法。然而，你如何在应用程序中使用这样的数据结构呢？在本节中，你将学习如何使用 C#语言实现图。为了让你更容易理解所呈现的内容，提供了两个例子。

# 节点

首先，让我们来看一下表示图中单个节点的泛型类的代码。这样的类名为`Node`，其代码如下所示：

```cs
public class Node<T> 
{ 
    public int Index { get; set; } 
    public T Data { get; set; } 
    public List<Node<T>> Neighbors { get; set; }  
        = new List<Node<T>>(); 
    public List<int> Weights { get; set; } = new List<int>(); 

    public override string ToString() 
    { 
        return $"Node with index {Index}: {Data}, 
            neighbors: {Neighbors.Count}"; 
    } 
} 
```

该类包含四个属性。由于所有这些元素在本章中显示的代码片段中都扮演着重要角色，让我们详细分析它们：

+   第一个属性（`Index`）存储了图中特定节点在节点集合中的索引，以简化访问特定元素的过程。因此，可以通过使用索引轻松获取表示特定节点的`Node`类的实例。

+   下一个属性名为`Data`，只是在节点中存储一些数据。值得一提的是，此类数据的类型与创建泛型类实例时指定的类型一致。

+   `Neighbors`属性表示特定节点的邻接表。因此，它包含对表示相邻节点的`Node`实例的引用。

+   最后一个属性名为`Weights`，用于存储分配给相邻边的权重。在加权图的情况下，`Weights`列表中的元素数量与相邻节点（`Neighbors`）的数量相同。如果图是无权的，则`Weights`列表为空。

除了属性之外，该类还包含重写的`ToString`方法，该方法返回对象的文本表示。在这里，以格式`"Node with index [index]: [data], neighbors: [count]"`返回字符串。

# 边

如在关于图的简短介绍中提到的，图由节点和边组成。由于节点由`Node`类的实例表示，因此`Edge`泛型类可用于表示边。代码的适当部分如下所示：

```cs
public class Edge<T> 
{ 
    public Node<T> From { get; set; } 
    public Node<T> To { get; set; } 
    public int Weight { get; set; } 

    public override string ToString() 
    { 
        return $"Edge: {From.Data} -> {To.Data},  
            weight: {Weight}"; 
    } 
} 
```

该类包含三个属性，分别表示与边相邻的节点（`From`和`To`），以及边的权重（`Weight`）。此外，`ToString`方法被重写以呈现有关边的一些基本信息。

# 图

下一个类名为`Graph`，表示整个图，具有有向或无向边，以及加权或无权边。实现包括各种字段和方法，如下所述。

让我们来看一下`Graph`类的基本版本：

```cs
public class Graph<T> 
{ 
    private bool _isDirected = false; 
    private bool _isWeighted = false; 
    public List<Node<T>> Nodes { get; set; }  
        = new List<Node<T>>(); 
} 
```

该类包含两个字段，指示边是否有向（`_isDirected`）和加权（`_isWeighted`）。此外，声明了`Nodes`属性，该属性存储图中存在的节点列表。

该类还包含以下构造函数：

```cs
public Graph(bool isDirected, bool isWeighted) 
{ 
    _isDirected = isDirected; 
    _isWeighted = isWeighted; 
} 
```

在这里，根据传递给构造函数的参数的值，只设置了`_isDirected`和`_isWeighted`私有字段的值。

`Graph`类的下一个有趣成员是索引器，它接受两个索引，即两个节点的索引，以返回表示这些节点之间边的`Edge`泛型类的实例。实现如下代码片段所示：

```cs
public Edge<T> this[int from, int to] 
{ 
    get 
    { 
        Node<T> nodeFrom = Nodes[from]; 
        Node<T> nodeTo = Nodes[to]; 
        int i = nodeFrom.Neighbors.IndexOf(nodeTo); 
        if (i >= 0) 
        { 
            Edge<T> edge = new Edge<T>() 
            { 
                From = nodeFrom, 
                To = nodeTo, 
                Weight = i < nodeFrom.Weights.Count  
                    ? nodeFrom.Weights[i] : 0 
            }; 
            return edge; 
        } 

        return null; 
    } 
} 
```

在索引器中，根据索引获取表示两个节点（`nodeFrom`和`nodeTo`）的`Node`类的实例。如果要找到从第一个节点（`nodeFrom`）到第二个节点（`nodeTo`）的边，需要尝试在第一个节点的相邻节点集合中找到第二个节点，使用`IndexOf`方法。如果这样的连接不存在，`IndexOf`方法将返回负值，并且索引器将返回`null`。否则，创建`Edge`类的新实例，并设置其属性的值，包括`From`和`To`。如果提供了关于特定边权重的数据，则还设置`Edge`类的`Weight`属性的值。

现在您知道如何存储图中节点的数据，但是如何添加新节点呢？为此，实现了`AddNode`方法，如下所示：

```cs
public Node<T> AddNode(T value) 
{ 
    Node<T> node = new Node<T>() { Data = value }; 
    Nodes.Add(node); 
    UpdateIndices(); 
    return node; 
} 
```

在此方法中，您创建`Node`类的新实例，并根据参数的值设置`Data`属性的值。 然后，新创建的实例被添加到`Nodes`集合中，并调用`UpdateIndices`方法（稍后描述）来更新集合中存储的所有节点的索引。 最后，返回表示新添加节点的`Node`实例。

您也可以移除现有节点。 这个操作是由`RemoveNode`方法执行的，如下面的代码片段所示：

```cs
public void RemoveNode(Node<T> nodeToRemove) 
{ 
    Nodes.Remove(nodeToRemove); 
    UpdateIndices(); 
    foreach (Node<T> node in Nodes) 
    { 
        RemoveEdge(node, nodeToRemove); 
    } 
} 
```

该方法接受一个参数，即应该被移除的节点的实例。 首先，您将其从节点集合中移除。 然后，您更新剩余节点的索引。 最后，您遍历图中的所有节点，以删除与已删除节点相连的所有边。

正如您已经知道的，图由节点和边组成。 因此，`Graph`类的实现应该为开发人员提供添加新边的方法。 当然，它应该支持各种边的变体，无论是有向的，无向的，加权的，还是无权重的。 提议的实现如下所示：

```cs
public void AddEdge(Node<T> from, Node<T> to, int weight = 0) 
{ 
    from.Neighbors.Add(to); 
    if (_isWeighted) 
    { 
        from.Weights.Add(weight); 
    } 

    if (!_isDirected) 
    { 
        to.Neighbors.Add(from); 
        if (_isWeighted) 
        { 
            to.Weights.Add(weight); 
        } 
    } 
} 
```

`AddEdge`方法接受三个参数，即表示由边连接的两个`Node`类实例（`from`和`to`），以及连接的权重（`weight`），默认设置为`0`。

在方法内的第一行，您将表示第二个节点的`Node`实例添加到第一个节点的邻居节点列表中。 如果考虑加权图，上述边的权重也将被添加。

代码的以下部分仅在图是无向的情况下考虑。 在这种情况下，您需要自动在相反方向上添加一条边。 为此，您将表示第一个节点的`Node`实例添加到第二个节点的邻居节点列表中。 如果边是加权的，那么上述边的权重也将添加到`Weights`列表中。

从图中删除边的过程由`RemoveEdge`方法支持。 代码如下：

```cs
public void RemoveEdge(Node<T> from, Node<T> to) 
{ 
    int index = from.Neighbors.FindIndex(n => n == to); 
    if (index >= 0) 
    { 
        from.Neighbors.RemoveAt(index);
        if (_isWeighted)
        { 
            from.Weights.RemoveAt(index); 
        }
    } 
} 
```

该方法接受两个参数，即两个节点（`from`和`to`），它们之间有一条应该被移除的边。 首先，您尝试在第一个节点的邻居节点列表中找到第二个节点。 如果找到了，您将其移除。 当然，如果考虑加权图，您还应该移除权重数据。

最后一个公共方法名为`GetEdges`，它可以获取图中所有可用边的集合。 提议的实现如下：

```cs
public List<Edge<T>> GetEdges() 
{ 
    List<Edge<T>> edges = new List<Edge<T>>(); 
    foreach (Node<T> from in Nodes) 
    { 
        for (int i = 0; i < from.Neighbors.Count; i++) 
        { 
            Edge<T> edge = new Edge<T>() 
            { 
                From = from, 
                To = from.Neighbors[i], 
                Weight = i < from.Weights.Count  
                    ? from.Weights[i] : 0 
            }; 
            edges.Add(edge); 
        } 
    } 
    return edges; 
} 
```

首先，初始化一个新的边列表。 然后，您使用`foreach`循环遍历图中的所有节点。 在其中，您使用`for`循环创建`Edge`类的实例。 实例的数量应该等于当前节点（`foreach`循环中的`from`变量）的邻居节点的数量。 在`for`循环中，通过设置其属性的值来配置`Edge`类的新创建实例，即第一个节点（`from`变量，即`foreach`循环中的当前节点），第二个节点（当前分析的邻居节点），以及权重。 然后，新创建的实例被添加到边的集合中，由`edges`变量表示。 最后，返回结果。

在各种方法中，您使用`UpdateIndices`方法。 代码如下：

```cs
private void UpdateIndices() 
{ 
    int i = 0; 
    Nodes.ForEach(n => n.Index = i++); 
} 
```

该方法只是遍历图中的所有节点，并更新`Index`属性的值为连续的数字，从`0`开始。 值得注意的是，迭代是使用`ForEach`方法执行的，而不是`foreach`或`for`循环。

现在您知道如何创建图的基本实现。 下一步是将其应用于表示一些示例图，如下面的两个部分所示。

# 示例-无向且无权重的边

让我们尝试使用先前的实现来创建无向和无权图，如下图所示：

![](img/74eaa194-4d25-4862-bf52-87ab7b6bf942.png)

如您所见，图包含 8 个节点和 10 条边。您可以在`Program`类的`Main`方法中配置示例图。实现始于以下代码行，该行初始化了一个新的无向图（第一个参数的值为`false`）和一个无权图（第二个参数的值为`false`）：

```cs
Graph<int> graph = new Graph<int>(false, false); 
```

然后，您添加必要的节点，并将它们存储为`Node<int>`类型的新变量，如下所示：

```cs
Node<int> n1 = graph.AddNode(1); 
Node<int> n2 = graph.AddNode(2); 
Node<int> n3 = graph.AddNode(3); 
Node<int> n4 = graph.AddNode(4); 
Node<int> n5 = graph.AddNode(5); 
Node<int> n6 = graph.AddNode(6); 
Node<int> n7 = graph.AddNode(7); 
Node<int> n8 = graph.AddNode(8); 
```

最后，您只需要根据图的先前图表在节点之间添加边。必要的代码如下所示：

```cs
graph.AddEdge(n1, n2); 
graph.AddEdge(n1, n3); 
graph.AddEdge(n2, n4); 
graph.AddEdge(n3, n4); 
graph.AddEdge(n4, n5); 
graph.AddEdge(n5, n6); 
graph.AddEdge(n5, n7); 
graph.AddEdge(n5, n8); 
graph.AddEdge(n6, n7); 
graph.AddEdge(n7, n8); 
```

就是这样！如您所见，使用这种数据结构的建议实现非常容易配置图。现在，让我们继续进行一个稍微复杂一点的有向和加权边的场景。

# 示例 - 有向和加权边

下一个示例涉及有向加权图，如下所示：

![](img/37e3110d-10bd-43cf-8a81-fa6144a9e3de.png)

该实现与前一节中描述的实现非常相似。但是，需要进行一些修改。首先，构造函数的参数值不同，即使用`true`而不是`false`来指示正在考虑有向和加权的边。适当的代码行如下：

```cs
Graph<int> graph = new Graph<int>(true, true); 
```

关于添加节点的部分与前面的例子完全相同：

```cs
Node<int> n1 = graph.AddNode(1); 
Node<int> n2 = graph.AddNode(2); 
Node<int> n3 = graph.AddNode(3); 
Node<int> n4 = graph.AddNode(4); 
Node<int> n5 = graph.AddNode(5); 
Node<int> n6 = graph.AddNode(6); 
Node<int> n7 = graph.AddNode(7); 
Node<int> n8 = graph.AddNode(8); 
```

在关于添加边的代码行中，一些更改很容易看到。在这里，您指定了带有它们权重的有向边，如下所示：

```cs
graph.AddEdge(n1, n2, 9); 
graph.AddEdge(n1, n3, 5); 
graph.AddEdge(n2, n1, 3); 
graph.AddEdge(n2, n4, 18); 
graph.AddEdge(n3, n4, 12); 
graph.AddEdge(n4, n2, 2); 
graph.AddEdge(n4, n8, 8); 
graph.AddEdge(n5, n4, 9); 
graph.AddEdge(n5, n6, 2); 
graph.AddEdge(n5, n7, 5); 
graph.AddEdge(n5, n8, 3); 
graph.AddEdge(n6, n7, 1); 
graph.AddEdge(n7, n5, 4); 
graph.AddEdge(n7, n8, 6); 
graph.AddEdge(n8, n5, 3); 
```

您刚刚完成了图的基本实现，分别在两个示例中展示。因此，让我们继续另一个主题，即遍历图。

# 遍历

图上执行的一个有用操作是其**遍历**，即以某种特定顺序访问所有节点。当然，前面提到的问题可以以各种方式解决，例如使用**深度优先搜索**（**DFS**）或**广度优先搜索**（**BFS**）方法。值得一提的是，遍历主题与在图中搜索给定节点的任务密切相关。

# 深度优先搜索

本章描述的第一个图遍历算法称为 DFS。在示例图的上下文中，其步骤如下：

![](img/a1e3f5de-294d-47ce-b2fe-0bdd4a626522.png)

当然，仅仅通过查看前面的图表就很难理解 DFS 算法是如何操作的。因此，让我们尝试分析其各个阶段。

在第一步中，您可以看到具有八个节点的图。节点**1**用灰色背景标记（表示该节点已被访问），并用红色边框标记（表示当前正在访问的节点）。此外，算法中的邻居节点（以虚线边框显示为圆圈）起着重要作用。当您了解特定指示器的作用时，很明显，在第一步中访问了节点**1**。它有两个邻居（节点**2**和**3**）。

然后，首个邻居（节点**2**）被考虑，并执行相同的操作，即访问节点并分析邻居（节点**1**和**4**）。由于节点**1**已经被访问过，所以它被跳过。在下一步（标为**步骤#3**）中，节点**2**的第一个合适的邻居被考虑——节点**4**。它有两个邻居，即节点**2**（已经被访问）和**8**。接下来，节点**8**被访问（**步骤#4**），根据相同的规则，访问节点**5**（**步骤#5**）。它有四个邻居，即节点**4**（已经被访问）、**6**、**7**和**8**（已经被访问）。因此，在下一步中，节点**6**被考虑（**步骤#6**）。由于它只有一个邻居（节点**7**），所以下一个被访问的是它（**步骤#7**）。

然后，您检查节点**7**的邻居，即节点**5**和**8**。两者都已经被访问过，所以您返回到具有未访问邻居的节点。在这个例子中，节点**1**有一个未访问的节点，即节点**3**。当它被访问（**步骤#8**）时，所有节点都被遍历，不需要进行进一步的操作。

给定这个例子，让我们尝试在 C#语言中创建实现。首先，`DFS`方法的代码（在`Graph`类中）如下所示：

```cs
public List<Node<T>> DFS() 
{ 
    bool[] isVisited = new bool[Nodes.Count]; 
    List<Node<T>> result = new List<Node<T>>(); 
    DFS(isVisited, Nodes[0], result); 
    return result; 
} 
```

`isVisited`数组发挥了重要作用。它的元素数量与节点数量相同，并存储指示给定节点是否已经被访问的值。如果是，就存储`true`值，否则存储`false`。遍历节点的列表以`result`变量的形式表示。此外，这里调用了`DFS`方法的另一个变体，传递了三个参数，即对`isVisited`数组的引用、要分析的第一个节点，以及用于存储结果的列表。

上述`DFS`方法的代码如下所示：

```cs
private void DFS(bool[] isVisited, Node<T> node,  
    List<Node<T>> result) 
{ 
    result.Add(node); 
    isVisited[node.Index] = true; 

    foreach (Node<T> neighbor in node.Neighbors) 
    { 
        if (!isVisited[neighbor.Index]) 
        { 
            DFS(isVisited, neighbor, result); 
        } 
    } 
} 
```

所示的实现非常简单。在开始时，当前节点被添加到遍历节点的集合中，并更新`isVisited`数组中的元素。然后，您使用`foreach`循环来遍历当前节点的所有邻居。对于每一个邻居，如果它还没有被访问过，就会递归调用`DFS`方法。

您可以在[`en.wikipedia.org/wiki/Depth-first_search`](https://en.wikipedia.org/wiki/Depth-first_search)找到有关 DFS 的更多信息。

最后，让我们看一下可以放在`Program`类的`Main`方法中的代码。其主要部分如下代码片段所示：

```cs
Graph<int> graph = new Graph<int>(true, true); 
Node<int> n1 = graph.AddNode(1); (...) 
Node<int> n8 = graph.AddNode(8); 
graph.AddEdge(n1, n2, 9); (...) 
graph.AddEdge(n8, n5, 3); 
List<Node<int>> dfsNodes = graph.DFS(); 
dfsNodes.ForEach(n => Console.WriteLine(n)); 
```

在这里，您初始化了一个有向加权图。要开始遍历图，您只需要调用`DFS`方法，它会返回一个`Node`实例的列表。然后，您可以轻松地遍历列表中的元素，打印每个节点的一些基本信息。结果如下所示：

```cs
    Node with index 0: 1, neighbors: 2
    Node with index 1: 2, neighbors: 2
    Node with index 3: 4, neighbors: 2
    Node with index 7: 8, neighbors: 1
    Node with index 4: 5, neighbors: 4
    Node with index 5: 6, neighbors: 1
    Node with index 6: 7, neighbors: 2
    Node with index 2: 3, neighbors: 1

```

就是这样！正如您所看到的，该算法试图尽可能深入，然后返回以找到下一个可以遍历的未访问邻居。然而，所呈现的算法并不是解决图遍历问题的唯一方法。在下一部分，您将看到另一种方法，以及一个基本示例和其实现。

# 广度优先搜索

在前面的部分，您学习了 DFS 方法。现在您将看到另一种解决方案，即 BFS。它的主要目的是首先访问当前节点的所有邻居，然后继续到下一级节点。

如果前面的描述听起来有点复杂，请看一下这个图表，它描述了 BFS 算法的步骤：

![](img/c3e3596a-13bc-4a71-907f-4ca85b345502.png)

该算法从访问节点**1**（**步骤＃1**）开始。它有两个邻居，即节点**2**和**3**，接下来访问它们（**步骤＃2**和**步骤＃3**）。由于节点**1**没有更多的邻居，因此考虑其第一个邻居（节点**2**）的邻居。由于它只有一个邻居（节点**4**），它在下一步被访问。按照相同的方法，剩下的节点按照这个顺序被访问：**8**，**5**，**6**，**7**。

听起来很简单，不是吗？让我们看一下实现：

```cs
public List<Node<T>> BFS() 
{ 
    return BFS(Nodes[0]); 
} 
```

`BFS`公共方法被添加到`Graph`类中，仅用于启动图的遍历。它调用私有的`BFS`方法，将第一个节点作为参数传递。其代码如下所示：

```cs
private List<Node<T>> BFS(Node<T> node) 
{ 
    bool[] isVisited = new bool[Nodes.Count]; 
    isVisited[node.Index] = true; 

    List<Node<T>> result = new List<Node<T>>(); 
    Queue<Node<T>> queue = new Queue<Node<T>>(); 
    queue.Enqueue(node); 
    while (queue.Count > 0) 
    { 
        Node<T> next = queue.Dequeue(); 
        result.Add(next); 

        foreach (Node<T> neighbor in next.Neighbors) 
        { 
            if (!isVisited[neighbor.Index]) 
            { 
                isVisited[neighbor.Index] = true; 
                queue.Enqueue(neighbor); 
            } 
        } 
    } 

    return result; 
} 
```

代码中的重要角色由`isVisited`数组发挥，它存储布尔值，指示特定节点是否已经被访问。这样的数组在`BFS`方法开始时被初始化，与当前节点相关的元素的值被设置为`true`，表示该节点已被访问。

然后，创建用于存储遍历节点的列表（`result`）和用于存储应在以下迭代中访问的节点的队列（`queue`）。就在初始化队列之后，当前节点被添加到队列中。

直到队列为空为止，执行以下操作：从队列中获取第一个节点（`next`变量），将其添加到已访问节点的集合中，并迭代当前节点的邻居。对于每个邻居，检查它是否已经被访问。如果没有，通过在`isVisited`数组中设置适当的值来标记为已访问，并将邻居添加到队列中，以便在`while`循环的下一次迭代中进行分析。

你可以在[`www.geeksforgeeks.org/breadth-first-traversal-for-a-graph/`](https://www.geeksforgeeks.org/breadth-first-traversal-for-a-graph/)找到有关 BFS 算法及其实现的更多信息。

最后，返回已访问节点的列表。如果要测试所描述的算法，可以将以下代码放入`Program`类中的`Main`方法中：

```cs
Graph<int> graph = new Graph<int>(true, true); 
Node<int> n1 = graph.AddNode(1); (...) 
Node<int> n8 = graph.AddNode(8); 
graph.AddEdge(n1, n2, 9); (...) 
graph.AddEdge(n8, n5, 3); 
List<Node<int>> bfsNodes = graph.BFS(); 
bfsNodes.ForEach(n => Console.WriteLine(n)); 
```

代码初始化图形，添加适当的节点和边，并调用`BFS`公共方法来根据 BFS 算法遍历图形。最后一行负责迭代结果以在控制台中呈现节点的数据：

```cs
 Node with index 0: 1, neighbors: 2
 Node with index 1: 2, neighbors: 2
 Node with index 2: 3, neighbors: 1
 Node with index 3: 4, neighbors: 2
 Node with index 7: 8, neighbors: 1
 Node with index 4: 5, neighbors: 4
 Node with index 5: 6, neighbors: 1
 Node with index 6: 7, neighbors: 2
```

你刚刚学会了两种遍历图的算法，即 DFS 和 BFS。为了让你更容易理解这些主题，本章包含了详细的描述、图表和示例。现在，让我们继续到下一节，了解另一个重要主题，即最小生成树，它在现实世界中有许多应用。

# 最小生成树

在谈论图形时，介绍**生成树**的主题是有益的。什么是生成树？生成树是连接图中所有节点而没有循环的边的子集。当然，在同一个图中可能有许多生成树。例如，让我们看一下以下图表：

![](img/84d01292-80ee-4542-9afb-10e292ca0a0e.png)

左侧是一个由以下边组成的生成树：(**1**, **2**), (**1**, **3**), (**3**, **4**), (**4**, **5**), (**5**, **6**), (**6**, **7**), 和 (**5**, **8**)。总权重等于 40。右侧显示了另一个生成树。这里考虑了以下边：(**1**, **2**), (**1**, **3**), (**2**, **4**), (**4**, **8**), (**5**, **8**), (**5**, **6**), 和 (**6**, **7**)。总权重等于 31。

然而，前述的生成树都不是该图的最小生成树（MST）。生成树是*最小*的是什么意思？答案非常简单：它是图中所有生成树中成本最低的生成树。您可以通过用（5，7）替换（6，7）边来获得最小生成树。然后，成本等于 30。还值得一提的是，生成树中的边数等于节点数减一。

为什么最小生成树的主题如此重要？让我们想象一个场景，当您需要将许多建筑物连接到电信电缆时。当然，有各种可能的连接，例如从一个建筑物到另一个建筑物，或者使用中心。而且，环境条件可能会严重影响投资成本，因为需要穿越道路甚至河流。您的任务是以尽可能低的成本成功将所有建筑物连接到电信电缆。您应该如何设计连接？要回答这个问题，您只需要创建一个图，其中节点表示连接器，边表示可能的连接。然后，找到最小生成树，就这样！

上述连接许多建筑物到电信电缆的问题在最小生成树的相关部分结束时的示例中进行了介绍。

下一个问题是如何找到最小生成树？有各种方法来解决这个问题，包括应用 Kruskal 或 Prim 算法，这些算法在下面的部分中进行了介绍和解释。

# Kruskal 算法

找到最小生成树的算法之一是由 Kruskal 发现的。它的操作非常简单。该算法从剩余的边中取出权重最小的边，并将其添加到最小生成树中，只有在添加它不会创建循环时才这样做。当所有节点连接时，算法停止。

让我们来看一下使用 Kruskal 算法找到最小生成树的步骤的图表：

![](img/f5ee36e3-7ca4-425f-8907-e1830736b7bc.png)

在第一步中，选择边（5，8），因为它的权重最小，即 1。然后选择边（1，2），（2，4），（5，6），（1，3），（5，7）和（4，8）。值得注意的是，在选择（4，8）边之前，考虑了（6，7）边，因为它的权重更低。然而，将其添加到最小生成树中将会引入由（5，6），（6，7）和（5，7）边组成的循环。因此，忽略这样的边，算法选择了边（4，8）。最后，最小生成树中的边数为 7。节点数等于 8，这意味着算法可以停止运行并找到最小生成树。

让我们来看一下实现。它涉及到`MinimumSpanningTreeKruskal`方法，应该添加到`Graph`类中。建议的代码如下：

```cs
public List<Edge<T>> MinimumSpanningTreeKruskal() 
{ 
    List<Edge<T>> edges = GetEdges(); 
    edges.Sort((a, b) => a.Weight.CompareTo(b.Weight)); 
    Queue<Edge<T>> queue = new Queue<Edge<T>>(edges); 

    Subset<T>[] subsets = new Subset<T>[Nodes.Count]; 
    for (int i = 0; i < Nodes.Count; i++) 
    { 
        subsets[i] = new Subset<T>() { Parent = Nodes[i] }; 
    } 

    List<Edge<T>> result = new List<Edge<T>>(); 
    while (result.Count < Nodes.Count - 1) 
    { 
        Edge<T> edge = queue.Dequeue(); 
        Node<T> from = GetRoot(subsets, edge.From); 
        Node<T> to = GetRoot(subsets, edge.To); 
        if (from != to) 
        { 
            result.Add(edge); 
            Union(subsets, from, to); 
        } 
    } 

    return result; 
} 
```

该方法不接受任何参数。首先，通过调用`GetEdges`方法获得边的列表。然后，按权重升序对边进行排序。这一步非常关键，因为您需要在算法的后续迭代中获得具有最小成本的边。在下一行，创建一个新的队列，并使用`Queue`类的构造函数将`Edge`实例入队。

在下一段代码中，创建了一个包含子集数据的数组。默认情况下，每个节点都被添加到一个单独的子集中。这就是为什么`subsets`数组中的元素数量等于节点数的原因。这些子集用于检查将边添加到最小生成树是否会导致创建循环。

然后，创建用于存储来自 MST 的边的列表（`result`）。代码中最有趣的部分是`while`循环，它迭代直到在 MST 中找到正确数量的边。在循环内，通过在`Queue`实例上调用`Dequeue`方法来获取具有最小权重的边。然后，检查添加找到的边是否引入了循环。在这种情况下，将边添加到目标列表，并调用`Union`方法来合并两个子集。

在分析前面的方法时，提到了`GetRoot`方法。它的目的是更新子集的父节点，并返回子集的根节点。

```cs
private Node<T> GetRoot(Subset<T>[] subsets, Node<T> node) 
{ 
    if (subsets[node.Index].Parent != node) 
    { 
        subsets[node.Index].Parent = GetRoot( 
            subsets, 
            subsets[node.Index].Parent); 
    } 

    return subsets[node.Index].Parent; 
} 
```

最后一个私有方法名为`Union`，执行两个集合的联合操作（按秩）。它接受三个参数，即`Subset`实例的数组和两个`Node`实例，表示应在其上执行联合操作的子集的根节点。代码的适当部分如下：

```cs
private void Union(Subset<T>[] subsets, Node<T> a, Node<T> b) 
{ 
    if (subsets[a.Index].Rank > subsets[b.Index].Rank) 
    { 
        subsets[b.Index].Parent = a; 
    } 
    else if (subsets[a.Index].Rank < subsets[b.Index].Rank) 
    { 
        subsets[a.Index].Parent = b; 
    } 
    else 
    { 
        subsets[b.Index].Parent = a; 
        subsets[a.Index].Rank++; 
    } 
} 
```

在前面的代码片段中，您可以看到`Subset`类，但它是什么样子的呢？让我们看一下它的声明：

```cs
public class Subset<T> 
{ 
    public Node<T> Parent { get; set; } 
    public int Rank { get; set; } 

    public override string ToString() 
    { 
        return $"Subset with rank {Rank}, parent: {Parent.Data}  
            (index: {Parent.Index})"; 
    } 
} 
```

该类包含代表父节点（`Parent`）和子集秩（`Rank`）的属性。该类还重写了`ToString`方法，以文本形式呈现有关子集的一些基本信息。

所呈现的代码基于[`www.geeksforgeeks.org/greedy-algorithms-set-2-kruskals-minimum-spanning-tree-mst/`](https://www.geeksforgeeks.org/greedy-algorithms-set-2-kruskals-minimum-spanning-tree-mst/)中显示的实现。您还可以在那里找到有关 Kruskal 算法的更多信息。

让我们看一下`MinimumSpanningTreeKruskal`方法的用法：

```cs
Graph<int> graph = new Graph<int>(false, true); 
Node<int> n1 = graph.AddNode(1); (...) 
Node<int> n8 = graph.AddNode(8); 
graph.AddEdge(n1, n2, 3); (...) 
graph.AddEdge(n7, n8, 20); 
List<Edge<int>> mstKruskal = graph.MinimumSpanningTreeKruskal(); 
mstKruskal.ForEach(e => Console.WriteLine(e)); 
```

首先，初始化一个无向加权图，并添加节点和边。然后，调用`MinimumSpanningTreeKruskal`方法，使用 Kruskal 算法找到 MST。最后，使用`ForEach`方法将 MST 中每条边的数据写入控制台。

# Prim 算法

解决查找 MST 问题的另一种方法是**Prim 算法**。它使用两组不相交的节点，即位于 MST 中的节点和尚未放置在那里的节点。在接下来的迭代中，算法找到连接第一组节点和第二组节点的具有最小权重的边。尚未在 MST 中的边的节点将被添加到此集合中。

前面的描述听起来相当简单，不是吗？让我们通过分析图表，看看使用 Prim 算法找到 MST 的步骤：

![](img/ee23c538-4e24-4625-b17d-b734be8dabac.png)

让我们看一看图中节点旁边添加的额外指示器。它们表示从任何邻居到达该节点所需的最小权重。默认情况下，起始节点的此值设置为**0**，而其他所有节点均设置为无穷大。

在**步骤#2**中，将起始节点添加到形成 MST 的节点子集中，并更新到其邻居的距离，即到达节点**3**的距离为**5**，到达节点**2**的距离为**3**。

在下一步（即**步骤#3**）中，选择具有最小成本的节点。在这种情况下，选择节点**2**，因为成本等于**3**。它的竞争对手（即节点**3**）的成本等于**5**。接下来，您需要更新到达当前节点的邻居的成本，即将节点**4**的成本设置为**4**。

显然，下一个选择的节点是节点**4**，因为它不存在于 MST 集合中，并且具有最低的到达成本（**步骤#4**）。以相同的方式，按以下顺序选择下一个边：（**1**，**3**），（**4**，**8**），（**8**，**5**），（**5**，**6**）和（**5**，**7**）。现在，所有节点都包含在 MST 中，算法可以停止其操作。

鉴于对算法步骤的详细描述，让我们继续进行基于 C#的实现。大部分操作都在`MinimumSpanningTreePrim`方法中执行，应将其添加到`Graph`类中：

```cs
public List<Edge<T>> MinimumSpanningTreePrim() 
{ 
    int[] previous = new int[Nodes.Count]; 
    previous[0] = -1; 

    int[] minWeight = new int[Nodes.Count]; 
    Fill(minWeight, int.MaxValue); 
    minWeight[0] = 0; 

    bool[] isInMST = new bool[Nodes.Count]; 
    Fill(isInMST, false); 

    for (int i = 0; i < Nodes.Count - 1; i++) 
    { 
        int minWeightIndex = GetMinimumWeightIndex( 
            minWeight, isInMST); 
        isInMST[minWeightIndex] = true; 

        for (int j = 0; j < Nodes.Count; j++) 
        { 
            Edge<T> edge = this[minWeightIndex, j]; 
            int weight = edge != null ? edge.Weight : -1; 
            if (edge != null 
                && !isInMST[j] 
                && weight < minWeight[j]) 
            { 
                previous[j] = minWeightIndex; 
                minWeight[j] = weight; 
            } 
        } 
    } 

    List<Edge<T>> result = new List<Edge<T>>(); 
    for (int i = 1; i < Nodes.Count; i++) 
    { 
        Edge<T> edge = this[previous[i], i]; 
        result.Add(edge); 
    } 
    return result; 
} 
```

`MinimumSpanningTreePrim`方法不接受任何参数。它使用三个辅助的与节点相关的数组，为图的节点分配附加数据。首先，即`previous`存储先前节点的索引，可以从该节点到达给定节点。默认情况下，所有元素的值都相等为`0`，除了第一个元素，它设置为`-1`。`minWeight`数组存储访问给定节点的边的最小权重。默认情况下，所有元素都设置为`int`类型的最大值，而第一个元素的值设置为`0`。`isInMST`数组指示给定节点是否已经在 MST 中。首先，所有元素的值都应设置为`false`。

代码中最有趣的部分位于`for`循环中。在其中，找到未位于 MST 中的节点集合中可以以最小成本到达的节点的索引。`GetMinimumWeightIndex`方法执行此任务。然后，使用另一个`for`循环。在其中，获取连接具有索引`minWeightIndex`和`j`的节点的边。检查节点是否尚未位于 MST 中，以及到达节点的成本是否小于先前的最小成本。如果是，则更新`previous`和`minWeight`数组中与节点相关的元素的值。

代码的其余部分只是准备最终结果。在这里，创建一个新的带有形成 MST 的边数据的列表的实例。使用`for`循环获取以下边的数据，并将它们添加到`result`列表中。

在分析代码时，提到了`GetMinimumWeightIndex`私有方法。其代码如下：

```cs
private int GetMinimumWeightIndex(int[] weights, bool[] isInMST) 
{ 
    int minValue = int.MaxValue; 
    int minIndex = 0; 

    for (int i = 0; i < Nodes.Count; i++) 
    { 
        if (!isInMST[i] && weights[i] < minValue) 
        { 
            minValue = weights[i]; 
            minIndex = i; 
        } 
    } 

    return minIndex; 
} 
```

`GetMinimumWeightIndex`方法只是找到一个索引，该索引是未位于 MST 中且可以以最小成本到达的节点的索引。为此，使用`for`循环遍历所有节点。对于每个节点，检查当前节点是否未位于 MST 中，以及到达它的成本是否小于已存储的最小值。如果是，则更新`minValue`和`minIndex`变量的值。最后，返回索引。

所示的代码基于[`www.geeksforgeeks.org/greedy-algorithms-set-5-prims-minimum-spanning-tree-mst-2/`](https://www.geeksforgeeks.org/greedy-algorithms-set-5-prims-minimum-spanning-tree-mst-2/)中显示的实现。您还可以在那里找到有关 Prim 算法的更多信息。

此外，还使用了辅助的`Fill`方法。它只是将数组中所有元素的值设置为作为第二个参数传递的值。该方法的代码如下：

```cs
private void Fill<Q>(Q[] array, Q value) 
{ 
    for (int i = 0; i < array.Length; i++) 
    { 
        array[i] = value; 
    } 
} 
```

让我们来看一下`MinimumSpanningTreePrim`方法的用法：

```cs
Graph<int> graph = new Graph<int>(false, true); 
Node<int> n1 = graph.AddNode(1); (...) 
Node<int> n8 = graph.AddNode(8); 
graph.AddEdge(n1, n2, 3); (...) 
graph.AddEdge(n7, n8, 20); 
List<Edge<int>> mstPrim = graph.MinimumSpanningTreePrim(); 
mstPrim.ForEach(e => Console.WriteLine(e)); 
```

首先，初始化一个无向加权图，并添加节点和边。然后，调用`MinimumSpanningTreePrim`方法使用 Prim 算法找到 MST。最后，使用`ForEach`方法将 MST 中每条边的数据写入控制台。

# 示例-通信电缆

正如在 MST 主题的介绍中提到的，这个问题有一些重要的现实应用，比如为建筑物之间的连接创建一个供给所有建筑物的通信电缆的计划，成本最小。当然，有各种可能的连接，比如从一个建筑物到另一个建筑物，或者使用一个中心。此外，环境条件可能会对投资成本产生严重影响，因为需要穿越道路甚至河流。例如，让我们创建一个解决这个问题的程序，该程序在以下图表中显示了建筑物集合的上下文：

![](img/d0d94024-3527-415b-b829-0f7ab8e73b81.png)

正如您所看到的，房地产社区由 12 栋建筑组成，包括位于河边的公寓楼和小亭。建筑物位于一条小河的两侧，只有一座桥。此外，还有两条道路。当然，连接各个点之间的成本各不相同，这取决于距离和环境条件。例如，两栋建筑物（**B1**和**B2**）之间的直接连接成本为**2**，而使用桥（**R1**和**R5**之间）的成本为**75**。如果您需要在没有桥的情况下穿过河流（**R3**和**R6**之间），成本甚至更高，为**100**。

您的任务是找到 MST。在此示例中，您将应用 Kruskal 和 Prim 算法来解决此问题。首先，让我们初始化无向加权图，并添加节点和边，如下所示：

```cs
Graph<string> graph = new Graph<string>(false, true); 
Node<string> nodeB1 = graph.AddNode("B1"); (...) 
Node<string> nodeR6 = graph.AddNode("R6"); 
graph.AddEdge(nodeB1, nodeB2, 2); (...) 
graph.AddEdge(nodeR6, nodeB6, 10); 
```

然后，您只需要调用`MinimumSpanningTreeKruskal`方法来使用 Kruskal 算法找到 MST。当结果获得时，您可以轻松地在控制台中呈现它们，同时呈现总成本。代码的适当部分如下所示：

```cs
Console.WriteLine("Minimum Spanning Tree - Kruskal's Algorithm:"); 
List<Edge<string>> mstKruskal =  
    graph.MinimumSpanningTreeKruskal(); 
mstKruskal.ForEach(e => Console.WriteLine(e)); 
Console.WriteLine("Total cost: " + mstKruskal.Sum(e => e.Weight)); 
```

在控制台中呈现的结果如下所示：

```cs
    Minimum Spanning Tree - Kruskal's Algorithm:
    Edge: R4 -> R3, weight: 1
    Edge: R3 -> R2, weight: 1
    Edge: R2 -> R1, weight: 1
    Edge: B1 -> B2, weight: 2
    Edge: B3 -> B4, weight: 2
    Edge: R6 -> R5, weight: 3
    Edge: R6 -> B5, weight: 3
    Edge: B5 -> B6, weight: 6
    Edge: B1 -> B3, weight: 20
    Edge: B2 -> R2, weight: 25
    Edge: R1 -> R5, weight: 75
    Total cost: 139

```

如果您在地图上可视化这样的结果，将找到以下 MST：

![](img/83e8b4f4-5b14-4e99-a011-47a7f5e4e48d.png)

类似地，您可以应用 Prim 算法：

```cs
Console.WriteLine("nMinimum Spanning Tree - Prim's Algorithm:"); 
List<Edge<string>> mstPrim = graph.MinimumSpanningTreePrim(); 
mstPrim.ForEach(e => Console.WriteLine(e)); 
Console.WriteLine("Total cost: " + mstPrim.Sum(e => e.Weight)); 
```

获得的结果如下：

```cs
    Minimum Spanning Tree - Prim's Algorithm:
    Edge: B1 -> B2, weight: 2
    Edge: B1 -> B3, weight: 20
    Edge: B3 -> B4, weight: 2
    Edge: R6 -> B5, weight: 3
    Edge: B5 -> B6, weight: 6
    Edge: R2 -> R1, weight: 1
    Edge: B2 -> R2, weight: 25
    Edge: R2 -> R3, weight: 1
    Edge: R3 -> R4, weight: 1
    Edge: R1 -> R5, weight: 75
    Edge: R5 -> R6, weight: 3
    Total cost: 139

```

就是这样！您刚刚完成了与 MST 的实际应用相关的示例。您准备好继续进行另一个与图相关的主题了吗？它被称为着色。

# 着色

寻找 MST 并不是唯一与图相关的问题。其中包括**节点着色**。其目的是为所有节点分配颜色（数字），以符合不能存在相同颜色的两个节点之间的边的规则。当然，颜色的数量应尽可能少。这样的问题在现实世界中有一些应用，例如着色地图，这是稍后显示的示例的主题。

您知道每个平面图的节点最多可以用四种颜色着色吗？如果您对此话题感兴趣，请查看**四色定理**（[`mathworld.wolfram.com/Four-ColorTheorem.html`](http://mathworld.wolfram.com/Four-ColorTheorem.html)）。本章中所示的着色算法的实现简单，并且在某些情况下可能使用比实际需要更多的颜色。

让我们看一下以下图表：

![](img/63167778-a260-403b-9e82-ff572d8b399f.png)

第一个图表（左侧显示）呈现了使用四种颜色着色的图：红色（索引等于**0**），绿色（**1**），蓝色（**2**）和紫罗兰色（**3**）。如您所见，没有使用相同颜色的节点通过边连接。右侧显示的图表描绘了具有两条额外边的图，即（**2**，**6**）和（**2**，**5**）。在这种情况下，着色已更改，但颜色数量保持不变。

问题是，您如何找到节点的颜色以符合上述规则？幸运的是，算法非常简单，其实现在此处呈现。应添加到`Graph`类的`Color`方法的代码如下：

```cs
public int[] Color() 
{ 
    int[] colors = new int[Nodes.Count]; 
    Fill(colors, -1); 
    colors[0] = 0; 

    bool[] availability = new bool[Nodes.Count]; 
    for (int i = 1; i < Nodes.Count; i++) 
    { 
        Fill(availability, true); 

        int colorIndex = 0; 
        foreach (Node<T> neighbor in Nodes[i].Neighbors) 
        { 
            colorIndex = colors[neighbor.Index]; 
            if (colorIndex >= 0) 
            { 
                availability[colorIndex] = false; 
            } 
        } 

        colorIndex = 0; 
        for (int j = 0; j < availability.Length; j++) 
        { 
            if (availability[j]) 
            { 
                colorIndex = j; 
                break; 
            } 
        } 

        colors[i] = colorIndex; 
    } 

    return colors; 
} 
```

`Color`方法使用两个辅助节点相关数组。第一个名为`colors`，存储为特定节点选择的颜色的索引。默认情况下，所有元素的值都设置为`-1`，除了第一个元素，它设置为`0`。这意味着第一个节点的颜色自动设置为第一个颜色（例如红色）。另一个辅助数组（`availability`）存储有关特定颜色的可用性的信息。

代码的最关键部分是`for`循环。在其中，通过将`true`设置为`availability`数组中所有元素的值，重置颜色的可用性。然后，你遍历当前节点的相邻节点，读取它们的颜色，并通过将`false`设置为`availability`数组中特定元素的值，标记这些颜色为不可用。最后的内部`for`循环只是遍历`availability`数组，并找到当前节点的第一个可用颜色。

所呈现的代码基于[`www.geeksforgeeks.org/graph-coloring-set-2-greedy-algorithm/`](https://www.geeksforgeeks.org/graph-coloring-set-2-greedy-algorithm/)中展示的实现。此外，你可以在那里找到有关着色问题的更多信息。

此外，辅助的`Fill`方法与之前的示例中解释的完全相同的代码一起使用。它只是将数组中所有元素的值设置为作为第二个参数传递的值。方法的代码如下：

```cs
private void Fill<Q>(Q[] array, Q value) 
{ 
    for (int i = 0; i < array.Length; i++) 
    { 
        array[i] = value; 
    } 
} 
```

让我们看一下`Color`方法的用法：

```cs
Graph<int> graph = new Graph<int>(false, false); 
Node<int> n1 = graph.AddNode(1); (...) 
Node<int> n8 = graph.AddNode(8); 
graph.AddEdge(n1, n2); (...) 
graph.AddEdge(n7, n8); 

int[] colors = graph.Color(); 
for (int i = 0; i < colors.Length; i++) 
{ 
    Console.WriteLine($"Node {graph.Nodes[i].Data}: {colors[i]}"); 
} 
```

在这里，你创建了一个新的无向无权图，添加了节点和边，并调用`Color`方法来执行节点着色。结果，你收到了一个包含特定节点颜色索引的数组。然后，你在控制台中呈现结果：

```cs
    Node 1: 0
    Node 2: 1
    Node 3: 1
    Node 4: 0
    Node 5: 1
    Node 6: 0
    Node 7: 2
    Node 8: 3
```

在这个简短的介绍之后，你已经准备好继续进行真实世界的应用，即对省份地图进行着色，接下来将会呈现。

# 示例 - 省份地图

让我们创建一个代表波兰省份地图的程序，将其表示为一个图，并对这些区域进行着色，以便具有共同边界的两个省份不具有相同的颜色。当然，你应该限制颜色的数量。

首先，让我们考虑图的表示。在这里，节点代表特定的省份，而边代表省份之间的共同边界。

已经着色的波兰地图如下图所示：

![](img/876d451d-6a0b-47e8-b31c-d49027b05301.png)

你的任务就是使用已经描述的算法对图中的节点进行着色。为此，你创建无向无权图，添加代表省份的节点，并添加边来表示共同的边界。代码如下：

```cs
Graph<string> graph = new Graph<string>(false, false); 
Node<string> nodePK = graph.AddNode("PK"); (...) 
Node<string> nodeOP = graph.AddNode("OP"); 
graph.AddEdge(nodePK, nodeLU); (...) 
graph.AddEdge(nodeDS, nodeOP);
```

然后，在`Graph`实例上调用`Color`方法，并返回特定节点的颜色索引。最后，你只需在控制台中呈现结果。代码的适当部分如下所示：

```cs
int[] colors = graph.Color(); 
for (int i = 0; i < colors.Length; i++) 
{ 
    Console.WriteLine($"{graph.Nodes[i].Data}: {colors[i]}"); 
} 
```

部分结果如下所示：

```cs
    PK: 0
    LU: 1 (...)
    OP: 2

```

你刚刚学会了如何给图中的节点着色！然而，这并不是本书中介绍的关于图的有趣主题的结束。现在，让我们继续搜索图中的最短路径。

# 最短路径

图是一个用于存储各种地图数据的优秀数据结构，例如城市和它们之间的距离。因此，图的一个明显的真实世界应用之一是搜索两个位置之间的**最短路径**，考虑到特定的成本，例如距离、所需时间，甚至所需燃料的数量。

在图中搜索最短路径有几种方法。然而，其中一个常见的解决方案是**Dijkstra 算法**，它可以计算从起始节点到图中所有节点的距离。然后，你不仅可以轻松地获得两个节点之间的连接成本，还可以找到位于起始节点和结束节点之间的节点。

Dijkstra 算法使用两个辅助节点相关数组，分别用于存储前一个节点的标识符（可以通过最小总成本到达当前节点的节点），以及访问当前节点所需的最小距离（成本）。此外，它使用队列来存储应该被检查的节点。在连续的迭代中，算法更新图中特定节点的最小距离。最后，辅助数组包含了从选择的起始节点到达所有节点的最小距离（成本），以及如何使用最短路径到达每个节点的信息。

在继续示例之前，让我们看一下以下图表，展示了使用 Dijkstra 算法找到的两条不同的最短路径。左侧显示了从节点**8**到**1**的路径，而右侧显示了从节点**1**到**7**的路径：

![](img/8d62b464-51f2-48f8-a815-63207ac35c59.png)

现在是时候看一些 C#代码了，这些代码可以用来实现 Dijkstra 算法。主要作用由`GetShortestPathDijkstra`方法执行，该方法应添加到`Graph`类中。代码如下：

```cs
public List<Edge<T>> GetShortestPathDijkstra( 
    Node<T> source, Node<T> target) 
{ 
    int[] previous = new int[Nodes.Count]; 
    Fill(previous, -1); 

    int[] distances = new int[Nodes.Count]; 
    Fill(distances, int.MaxValue); 
    distances[source.Index] = 0; 

    SimplePriorityQueue<Node<T>> nodes =  
        new SimplePriorityQueue<Node<T>>(); 
    for (int i = 0; i < Nodes.Count; i++) 
    { 
        nodes.Enqueue(Nodes[i], distances[i]); 
    } 

    while (nodes.Count != 0) 
    { 
        Node<T> node = nodes.Dequeue(); 
        for (int i = 0; i < node.Neighbors.Count; i++) 
        { 
            Node<T> neighbor = node.Neighbors[i]; 
            int weight = i < node.Weights.Count  
                ? node.Weights[i] : 0; 
            int weightTotal = distances[node.Index] + weight; 

            if (distances[neighbor.Index] > weightTotal) 
            { 
                distances[neighbor.Index] = weightTotal; 
                previous[neighbor.Index] = node.Index; 
                nodes.UpdatePriority(neighbor,  
                    distances[neighbor.Index]); 
            } 
        } 
    } 

    List<int> indices = new List<int>(); 
    int index = target.Index; 
    while (index >= 0) 
    { 
        indices.Add(index); 
        index = previous[index]; 
    } 

    indices.Reverse(); 
    List<Edge<T>> result = new List<Edge<T>>(); 
    for (int i = 0; i < indices.Count - 1; i++) 
    { 
        Edge<T> edge = this[indices[i], indices[i + 1]]; 
        result.Add(edge); 
    } 
    return result; 
} 
```

`GetShortestPathDijkstra`方法接受两个参数，即`source`和`target`节点。首先，它创建了两个与节点相关的辅助数组，用于存储前一个节点的索引，从中可以以最小总成本到达给定节点（`previous`），以及用于存储到给定节点的当前最小距离（`distances`）。默认情况下，`previous`数组中所有元素的值都设置为`-1`，而在`distances`数组中它们设置为`int`类型的最大值。当然，到源节点的距离设置为`0`。然后，创建一个新的优先队列，并将所有节点的数据入队。每个元素的优先级等于到达该节点的当前距离。

值得注意的是，示例使用了 NuGet 中的`OptimizedPriorityQueue`包。有关此包的更多信息，请访问[`www.nuget.org/packages/OptimizedPriorityQueue`](https://www.nuget.org/packages/OptimizedPriorityQueue)，以及第三章中的*优先队列*部分。

代码中最有趣的部分是`while`循环，该循环执行直到队列为空。在`while`循环中，您从队列中获取第一个节点，并使用`for`循环迭代其所有邻居。在这样的循环内部，通过将当前节点的距离和边的权重相加来计算到邻居的距离。如果计算出的距离小于当前存储的值，则更新关于给定邻居的最小距离的值，以及可以到达邻居的前一个节点的索引。值得注意的是，队列中元素的优先级也应该更新。

其余操作用于使用`previous`数组中存储的值解析路径。为此，您将下一个节点的索引（相反方向）保存在`indices`列表中。然后，您将其反转以获得从源节点到目标节点的顺序。最后，您只需创建边的列表，以便以适合从方法返回的形式呈现结果。

所呈现和描述的实现是基于[`en.wikipedia.org/wiki/Dijkstra%27s_algorithm`](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm)上显示的伪代码。您可以在那里找到有关 Dijkstra 算法的一些额外信息。

让我们看一下`GetShortestPathDijkstra`方法的用法：

```cs
Graph<int> graph = new Graph<int>(true, true); 
Node<int> n1 = graph.AddNode(1); (...) 
Node<int> n8 = graph.AddNode(8); 
graph.AddEdge(n1, n2, 9); (...) 
graph.AddEdge(n8, n5, 3); 
List<Edge<int>> path = graph.GetShortestPathDijkstra(n1, n5); 
path.ForEach(e => Console.WriteLine(e)); 
```

在这里，您创建了一个新的有向加权图，添加了节点和边缘，并调用了`GetShortestPathDijkstra`方法来搜索两个节点之间的最短路径，即节点`1`和`5`之间的路径。 结果，您将收到形成最短路径的边缘列表。 然后，您只需遍历所有边缘并在控制台中呈现结果：

```cs
    Edge: 1 -> 3, weight: 5
    Edge: 3 -> 4, weight: 12
    Edge: 4 -> 8, weight: 8
    Edge: 8 -> 5, weight: 3
```

在这个简短的介绍之后，再加上简单的示例，让我们继续进行更高级和有趣的与游戏开发相关的应用。 让我们开始吧！

# 示例-游戏地图

本章中显示的最后一个示例涉及应用 Dijkstra 算法来查找游戏地图中的最短路径。 假设您有一个带有各种障碍物的棋盘。 因此，玩家只能使用棋盘的一部分移动。 您的任务是找到棋盘上两个位置之间的最短路径。

首先，让我们将棋盘表示为一个二维数组，其中棋盘上的给定位置可以用于移动或不可用。 适当的代码部分应添加到`Program`类中的`Main`方法中，如下所示：

```cs
string[] lines = new string[] 
{ 
    "0011100000111110000011111", 
    "0011100000111110000011111", 
    "0011100000111110000011111", 
    "0000000000011100000011111", 
    "0000001110000000000011111", 
    "0001001110011100000011111", 
    "1111111111111110111111100", 
    "1111111111111110111111101", 
    "1111111111111110111111100", 
    "0000000000000000111111110", 
    "0000000000000000111111100", 
    "0001111111001100000001101", 
    "0001111111001100000001100", 
    "0001100000000000111111110", 
    "1111100000000000111111100", 
    "1111100011001100100010001", 
    "1111100011001100001000100" 
}; 
bool[][] map = new bool[lines.Length][]; 
for (int i = 0; i < lines.Length; i++) 
{ 
    map[i] = lines[i] 
        .Select(c => int.Parse(c.ToString()) == 0) 
        .ToArray(); 
} 
```

为了提高代码的可读性，地图被表示为一个`string`值的数组。 每一行都以文本形式呈现，字符数等于列数。 每个字符的值表示点的可用性。 如果等于`0`，则该位置可用。 否则，不可用。 然后，基于`string`的地图表示应转换为布尔型二维数组。 此任务由几行代码执行，如前面的代码片段所示。

下一步是创建图表，以及添加必要的节点和边缘。 适当的代码部分如下所示：

```cs
Graph<string> graph = new Graph<string>(false, true); 
for (int i = 0; i < map.Length; i++) 
{ 
    for (int j = 0; j < map[i].Length; j++) 
    { 
        if (map[i][j]) 
        { 
            Node<string> from = graph.AddNode($"{i}-{j}"); 

            if (i > 0 && map[i - 1][j]) 
            { 
                Node<string> to = graph.Nodes.Find( 
                    n => n.Data == $"{i - 1}-{j}"); 
                graph.AddEdge(from, to, 1); 
            } 

            if (j > 0 && map[i][j - 1]) 
            { 
                Node<string> to = graph.Nodes.Find( 
                    n => n.Data == $"{i}-{j - 1}"); 
                graph.AddEdge(from, to, 1); 
            } 
        } 
    } 
} 
```

首先，您初始化了一个新的无向加权图。 然后，您使用两个`for`循环来迭代棋盘上的所有位置。 在这些循环内，您检查给定位置是否可用。 如果是，您创建一个新节点（`from`）。 然后，您检查当前节点上方是否也可用。 如果是，将添加权重为`1`的适当边缘。 以类似的方式检查当前节点左侧是否可用，并在必要时添加边缘。

现在，您只需要获取表示源节点和目标节点的`Node`实例。 您可以通过使用`Find`方法并提供节点的文本表示（例如`0-0`或`16-24`）来实现。 然后，只需调用`GetShortestPathDijkstra`方法。 在这种情况下，算法将尝试找到第一行和列中的节点与最后一行和列中的节点之间的最短路径。 代码如下：

```cs
Node<string> source = graph.Nodes.Find(n => n.Data == "0-0"); 
Node<string> target = graph.Nodes.Find(n => n.Data == "16-24"); 
List<Edge<string>> path = graph.GetShortestPathDijkstra( 
   source, target); 
```

代码的最后部分与在控制台中呈现地图有关：

```cs
Console.OutputEncoding = Encoding.UTF8; 
for (int row = 0; row < map.Length; row++) 
{ 
    for (int column = 0; column < map[row].Length; column++) 
    { 
        ConsoleColor color = map[row][column]  
            ? ConsoleColor.Green : ConsoleColor.Red; 
        if (path.Any(e => e.From.Data == $"{row}-{column}"  
            || e.To.Data == $"{row}-{column}")) 
        { 
            color = ConsoleColor.White; 
        } 

        Console.ForegroundColor = color; 
        Console.Write("\u25cf "); 
    } 
    Console.WriteLine(); 
} 
Console.ForegroundColor = ConsoleColor.Gray; 
```

首先，您需要在控制台中设置适当的编码，以便能够呈现 Unicode 字符。 然后，您使用两个`for`循环来迭代棋盘上的所有位置。 在这些循环内，您选择应用于在控制台中表示点的颜色，可以是绿色（点可用）或红色（不可用）。 如果当前分析的点是最短路径的一部分，则颜色将更改为白色。 最后，您只需设置适当的颜色并写入表示子弹的 Unicode 字符。 当程序执行退出两个循环时，将设置默认的控制台颜色。

运行应用程序时，您将看到以下结果：

![](img/249c8656-1962-4bf7-abdf-4742c051434e.png)

干得好！ 现在，让我们进行简短的总结，以总结您在阅读本章时学到的内容。

# 总结

您刚刚完成了与开发应用程序时最重要的数据结构之一，即图相关的章节。正如您所学到的，图是由节点和边组成的数据结构。每条边连接两个节点。此外，图中有各种变体的边，如无向和有向，以及无权重和有权重。所有这些都已经被详细描述和解释，还有图表和代码示例。图的两种表示方法，即使用邻接表和邻接矩阵，也已经被解释。当然，您还学会了如何使用 C#语言实现图。

在谈论图时，也很重要介绍一些现实世界的应用，特别是由于这种数据结构的常见使用。例如，本章包含了社交媒体中可用的朋友结构的描述，或者在城市中搜索最短路径的问题。

在本章的主题中，您已经了解了如何遍历图，即以某种特定顺序访问所有节点。介绍了两种方法，即深度优先搜索和广度优先搜索。值得一提的是，遍历主题也可以应用于在图中搜索给定节点。

在另一节中，介绍了生成树和最小生成树的主题。提醒一下，生成树是连接图中所有节点而没有循环的边的子集，而最小生成树是具有图中所有可用生成树中最小成本的生成树。有几种方法可以找到最小生成树，包括 Kruskal 或 Prim 算法的应用。

然后，您学习了下面两个流行的与图相关的问题的解决方案。第一个是节点的着色，您需要为所有节点分配颜色（数字），以符合不能存在相同颜色的两个节点之间的边的规则。当然，颜色的数量应该尽可能少。

另一个问题是在两个节点之间搜索最短路径，考虑了特定的成本，比如距离、所需时间，甚至所需燃料的数量。在图中搜索最短路径有几种方法。然而，其中一个常见的解决方案是 Dijkstra 算法，它可以计算从起始节点到图中所有节点的距离。这个主题已经在本章中进行了介绍和解释。

现在，是时候进行总结，看看到目前为止在书中介绍的所有数据结构和算法。让我们翻开书页，继续到最后一章！
