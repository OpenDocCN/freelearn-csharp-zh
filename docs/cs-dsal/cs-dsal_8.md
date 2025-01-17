# 第八章：数据结构的分类

正如你在阅读本书时所看到的，有许多数据结构及其许多配置变体。因此，选择合适的数据结构并不是一件容易的事情，这可能会对开发解决方案的性能产生重大影响。即使本书中提到的主题形成了相当长的数据结构描述列表。因此，最好以某种方式对它们进行分类。

在本章中，描述的数据结构被分为线性和非线性类别。线性数据结构中的每个元素可以在逻辑上与其后一个或前一个元素相邻。在非线性数据结构的情况下，单个元素可以在逻辑上与许多其他元素相邻，不一定只有一个或两个。它们可以自由分布在内存中。

让我们来看看下面的图表，根据所提到的标准对数据结构进行分类：

![](img/eaaed9fa-1216-4071-a635-d462e5e24db1.png)

正如你所看到的，线性数据结构组包括数组、列表、栈和队列。当然，你还应该关注所提到的数据结构的各种子类型，比如链表的三种变体，它是列表的一个子类型。

在非线性数据结构的情况下，图表起着最重要的作用，因为它还包括树的子类型。此外，树包括二叉树和堆，而二叉搜索树是二叉树的一个子类型。同样，你可以描述本书中介绍和解释的其他数据结构的关系。

# 应用的多样性

你还记得书中展示的所有数据结构吗？由于描述的主题数量众多，最好再次查看以下数据结构，以及它们相关的算法，只是以简要摘要的形式，包括一些真实世界应用的信息。

# 数组

让我们从数组开始，这是第一章的两个主要主题之一。你可以使用这种数据结构来存储许多相同类型的变量，比如`int`、`string`或用户定义的类。重要的假设是数组中的元素数量在初始化后不能改变。此外，数组属于随机访问数据结构，这意味着你可以使用索引来访问数组的第一个、中间、第 n 个或最后一个元素。你可以从几种数组变体中受益，即单维、多维和不规则数组，也称为数组的数组。

所有这些变体都显示在下图中：

![](img/39074937-888a-45ff-96cc-6176c90d0b0a.png)

数组有许多应用，作为开发人员，你可能多次使用了这种数据结构。在本书中，你已经看到了如何使用它来存储各种数据，比如月份的名称、乘法表，甚至是游戏地图。在最后一种情况下，你创建了一个与地图大小相同的二维数组，其中每个元素指定了某种地形，例如草地或墙壁。

有许多算法可以对数组执行各种操作。然而，其中最常见的任务之一是对数组进行排序，以便将其元素按正确的顺序排列，无论是升序还是降序。本书重点介绍了四种算法，即选择排序、插入排序、冒泡排序以及快速排序。

# 列表

第一章描述的下一组数据结构与列表相关。它们类似于数组，但可以在必要时动态增加集合的大小。在下图中，你可以看到列表的几种变体，即单链表、双链表和循环链表：

![](img/498191b3-418b-4cb8-bd06-edc301235c9e.png)

值得一提的是，数组列表（`ArrayList`）以及其泛型（`List`）和排序（`SortedList`）变体都有内置的实现。后者可以被理解为一组键值对，始终按键排序。

短评对于单链表、双链表和循环链表可能是有益的。第一个数据结构使得可以通过`Next`属性轻松地从一个元素导航到下一个元素。然而，通过添加`Previous`属性可以进一步扩展，允许在前后方向导航，形成双链表。在循环链表中，第一个节点的`Previous`属性导航到最后一个节点，而`Next`属性将最后一个节点链接到第一个节点。值得注意的是，双链表有一个内置的实现（`LinkedList`），你可以很容易地扩展双链表以使其行为像循环链表。

列表有很多应用，可以解决各种类型应用程序中的不同问题。在本书中，你已经看到如何利用列表来存储一些浮点值并计算平均值，如何使用这种数据结构来创建一个简单的人员数据库，以及如何开发一个自动排序的地址簿。此外，你还准备了一个简单的应用程序，允许用户通过改变页面来阅读书籍，以及一个游戏，用户可以旋转具有随机动力的轮子。轮子旋转得越来越慢，直到停止。然后，用户可以再次旋转它，从上一次停止的位置开始，这说明了循环链表。

# 栈

本书的第三章专注于栈和队列。在本节中，让我们来看看栈，它是有限访问数据结构的代表。这个名字意味着你不能从结构中访问每个元素，获取元素的方式是严格指定的。在栈的情况下，你只能在顶部添加一个新元素（推入操作），并通过从顶部移除元素来获取元素（弹出操作）。因此，栈符合 LIFO 原则，即后进先出。

栈的图示如下所示：

![](img/474d2469-2ac1-47f0-9c22-7d3d5436f783.png)

当然，栈有许多现实世界的应用。其中一个例子是与许多盘子堆叠在一起的一堆盘子有关。你只能在堆叠的顶部添加一个新的盘子，只能从堆叠的顶部获取一个盘子。你不能移除第七个盘子而不先从顶部取出前六个盘子，也不能在堆叠的中间添加一个盘子。你还看到了如何使用栈来颠倒一个单词，以及如何应用它来解决数学游戏汉诺塔。

# 队列

本书第三章的另一个主题是队列。使用这种数据结构时，只能在队列的末尾添加新元素（入队操作），并且只能从队列的开头移除元素（出队操作）。因此，这种数据结构符合 FIFO 原则，即先进先出。

队列的图示如下所示：

![](img/6250fe7d-3cb5-4ecf-b654-d944151ec9d1.png)

还可以使用优先队列，它通过为每个元素设置优先级来扩展队列的概念。因此，`Dequeue`操作返回最早添加到队列中的优先级最高的元素。

以下是示例 BST 的图示：

# 第四章的主题与字典和集合有关。首先，让我们看一下字典，它允许将键映射到值并进行快速查找。字典使用哈希函数，可以理解为一组成对的集合，每对由键和值组成。字典有两个内置版本，即非泛型（`Hashtable`）和泛型（`Dictionary`）。还有排序的字典（`SortedDictionary`）可用。

字典

哈希表的机制如下图所示：

![](img/73a7e5c3-bc69-45cf-a947-d5684220a97e.png)

由于哈希表的出色性能，这种数据结构经常在许多现实世界的应用中使用，例如用于关联数组、数据库索引或缓存系统。在本书中，你已经看到如何创建电话簿来存储条目，其中一个人的名字是键，电话号码是值。在其他示例中，你已经开发了一个帮助商店员工找到产品放置位置的应用，并且应用了排序字典来创建简单的百科全书，用户可以添加条目并显示其全部内容。

# 集合

另一个数据结构是集合，它是一个不重复元素的集合，没有特定顺序。因此，你只能知道给定元素是否在集合中。集合与数学模型和操作严格相关，如并集、交集、差集和对称差。

以下是存储各种类型数据的示例集：

![](img/f426b497-d916-4491-9fb3-3a4d1c6fceeb.png)

在 C#语言中开发应用程序时，你可以从`HashSet`类提供的高性能集合相关操作中受益。例如，你已经看到如何创建一个处理一次性促销券的系统，并允许你检查扫描的促销券是否已经被使用。另一个例子是 SPA 中心系统的报告服务，有四个游泳池。通过使用集合，你已经计算了统计数据，例如游泳池的访客人数，最受欢迎的游泳池以及至少访问过一个游泳池的人数。

# 队列有许多现实世界的应用。例如，队列可以用来表示在商店结账处等待的人群。新人站在队伍的末尾，下一个人从队伍的开头被带到结账处。你不能选择队伍中间的人来服务。此外，你已经看到了呼叫中心解决方案的几个示例，其中有许多呼叫者（具有不同的客户标识符）和一个顾问，许多呼叫者和许多顾问，以及许多呼叫者（具有不同的计划，标准或优先支持）和只有一个顾问，回答等待的呼叫。

树

一般来说，树中的每个节点可以包含任意数量的子节点。但是，在二叉树的情况下，一个节点不能包含超过两个子节点，即它可以不包含子节点，或者只包含一个或两个，但是没有关于节点之间关系的规则。如果要使用二叉搜索树（BST），则引入下一个规则。它规定，对于任何节点，其左子树中所有节点的值必须小于其值，并且其右子树中所有节点的值必须大于其值。

下一个主题是关于树，它是由具有一个根节点的数据结构组成。根节点不包含父节点，而所有其他节点都包含。此外，每个节点可以有任意数量的子节点。同一节点的子节点可以称为兄弟节点，而没有子节点的节点称为叶子节点。

![](img/633e03bc-cdd7-42a5-b25c-ca8b34be1fa8.png)

另一组树称为自平衡树，它在添加和删除节点时始终保持树的平衡。它们的应用非常重要，因为它允许您形成正确排列的树，对性能有积极影响。自平衡树有各种变体，但 AVL 和红黑树（RBTs）是最受欢迎的。这两种树在本书中都有简要描述。

在谈论树时，也有必要介绍一些遍历树的方法。在本书中，您已经学习了先序、中序和后序遍历的变体。

树是一种非常适合表示各种数据的数据结构，例如公司的结构，分为几个部门，每个部门都有自己的结构。您还看到了一个树的例子，用于安排由几个问题和答案组成的简单测验，这些问题和答案根据先前的决定显示。

# 堆

堆是树的另一种变体，有两个版本，即最小堆和最大堆。对于每一个，必须满足额外的属性。对于最小堆，每个节点的值必须大于或等于其父节点的值。对于最大堆，每个节点的值必须小于或等于其父节点的值。所述规则起着关键作用，确保根节点始终包含最小值（在最小堆中）或最大值（在最大堆中）。因此，它是一个非常方便的数据结构，用于实现优先队列。

堆存在许多变体，包括二叉堆，它还必须遵守完全二叉树规则，即每个节点不能包含两个以上的子节点，并且树的所有级别必须完全填满，除了最后一个级别，它可以从左到右填满，右边留有一些空间。

示例堆如下所示：

![](img/8be416c2-bb1a-4ee9-b294-578a15d11647.png)

当然，二叉堆不是唯一可用的堆。除了二叉堆，还有二项堆和斐波那契堆。这三种变体都在本书中有所描述。

堆的一个有趣应用是排序算法，名为堆排序。

# 图

前一章与图相关，作为一种广泛应用于现实世界的非常流行的数据结构。提醒一下，图是一个由节点和边组成的数据结构。每条边连接两个节点。此外，图中有几种边的变体，如无向和有向，以及无权重和有权重。图可以表示为邻接表或邻接矩阵。

所有这些主题都在本书中有所描述，包括图的遍历问题、寻找最小生成树、节点着色以及在图中寻找最短路径的问题。

以下图显示了示例图：

![](img/ba397110-c2a0-493f-9cc2-be665e23e4da.png)

图数据结构通常用于各种应用程序，并且是表示各种数据的绝佳方式，例如社交媒体网站上可用的朋友结构。在这里，节点可以表示联系人，而边表示人与人之间的关系。因此，您可以轻松地检查两个联系人是否互相认识，或者需要多少人参与安排两个特定人之间的会面。

图的另一个常见应用涉及寻找路径的问题。例如，您可以使用图来找到城市中两点之间的路径，考虑到驾驶所需的距离或时间。您可以使用图来表示城市的地图，其中节点是交叉路口，边表示道路。当然，您应该为边分配权重，以指示驾驶给定道路所需的距离或时间。

图还有许多其他相关的应用。例如，最小生成树可以用来创建建筑物之间的连接计划，以最小的成本为它们提供电信电缆，就像前一章中所解释的那样。

节点着色问题已经被用来根据这样一个规则对波兰地图上的省进行着色，即有共同边界的两个省份不能有相同的颜色。当然，颜色的数量应该是有限的。

这本书中另一个例子涉及到 Dijkstra 算法，用于在游戏地图中寻找最短路径。任务是在棋盘上找到两个地方之间的最短路径，考虑到各种障碍。

# 最后的话

你刚刚到达了这本书的最后一章的结尾。首先，介绍了数据结构的分类，考虑了线性和非线性数据结构。在第一组中，你可以找到数组、列表、栈和队列，而第二组涉及到图、树、堆，以及它们的变体。在本章的后续部分中，考虑了各种数据结构的多样化应用。你已经看到了对每种描述的数据结构的简要总结，以及关于可以用特定数据结构解决的一些问题的信息，比如队列或图。为了使内容更容易理解，并提醒你之前章节中的各种主题，总结中配有数据结构的插图。

在这本书的介绍中，我邀请您开始您的数据结构和算法之旅。在阅读以下章节、编写数百行代码和调试的过程中，您有机会熟悉各种数据结构，从数组和列表开始，经过栈、队列、字典和哈希集，最后到树、堆和图。我希望这本书只是您与数据结构和算法长期、充满挑战和成功的冒险的第一步。

感谢您阅读这本书。如果您对所描述的内容有任何问题或困惑，请不要犹豫直接联系我，联系信息显示在[`jamro.biz`](http://jamro.biz)。我希望您在作为软件开发人员的职业生涯中一切顺利，并且希望您有许多成功的项目！
