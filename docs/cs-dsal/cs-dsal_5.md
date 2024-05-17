# 第五章：树的变体

在前几章中，您已经了解了许多数据结构，从简单的数组开始。现在，是时候让您了解一组显著更复杂的数据结构，即**树**。

在本章的开头，将介绍基本树，以及在 C#语言中的实现和一些示例展示它的运行情况。然后，将介绍二叉树，详细描述其实现并举例说明其应用。二叉搜索树是另一种树的变体，是许多算法中使用的最流行的树类型之一。接下来的两节将涵盖自平衡树，即 AVL 和红黑树。

本章的其余部分将专门介绍堆作为基于树的数据结构。将介绍三种堆：二叉堆、二项式堆和斐波那契堆。这些类型将被简要介绍，并将展示这些数据结构的应用，使用外部包。

数组、列表、栈、队列、字典、集合，现在...树。您准备好提高难度并学习下一组数据结构了吗？如果是这样，让我们开始阅读！

在本章中，将涵盖以下主题：

+   基本树

+   二叉树

+   二叉搜索树

+   AVL 树

+   红黑树

+   二叉堆

+   二项式堆

+   斐波那契堆

# 基本树

让我们从介绍树开始。它们是什么？您对这样的数据结构应该是什么样子有任何想法吗？如果没有，让我们看一下以下图表，其中描述了一个带有关于其特定元素的标题的树：

![](img/7c15bf17-435a-40a2-a6ab-3b2768d96d92.png)

树由多个**节点**组成，包括一个**根**（图表中的**100**）。根不包含**父**节点，而所有其他节点都包含。例如，节点**1**的父元素是**100**，而节点**96**的父元素是**30**。此外，每个节点可以有任意数量的**子**节点，例如**根**的情况下有三个**子**节点（即**50**、**1**和**150**）。同一节点的子节点可以被称为**兄弟**，就像节点**70**和**61**的情况一样。没有子节点的节点称为**叶子**，例如图表中的**45**和**6**。看一下包含三个节点（即**30**、**96**和**9**）的矩形。树的这一部分可以称为**子树**。当然，您可以在树中找到许多子树。

让我们简要讨论节点的最小和最大子节点数。一般来说，这些数字是没有限制的，每个节点可以包含零、一个、两个、三个，甚至更多的子节点。然而，在实际应用中，子节点的数量通常限制为两个，正如您将在以下部分中看到的。

# 实现

基本树的 C#实现似乎是相当明显和不复杂的。为此，您可以声明两个类，表示单个节点和整个树，如下一节所述。

# 节点

第一个类名为`TreeNode`，声明为通用类，以便为开发人员提供指定存储在每个节点中的数据类型的能力。因此，您可以创建强类型化的解决方案，从而消除了将对象转换为目标类型的必要性。代码如下：

```cs
public class TreeNode<T> 
{ 
    public T Data { get; set; } 
    public TreeNode<T> Parent { get; set; } 
    public List<TreeNode<T>> Children { get; set; } 

    public int GetHeight() 
    { 
        int height = 1; 
        TreeNode<T> current = this; 
        while (current.Parent != null) 
        { 
            height++; 
            current = current.Parent; 
        } 
        return height; 
    } 
} 
```

该类包含三个属性：节点中存储的数据（`Data`）是在创建类的实例时指定的类型（`T`）的引用，指向父节点（`Parent`）的引用，以及指向子节点（`Children`）的引用的集合。

除了属性之外，`TreeNode`类还包含`GetHeight`方法，该方法返回节点的高度，即到根节点的距离。该方法的实现非常简单，因为它只是使用`while`循环从节点向上移动，直到没有父元素（达到根时）。

# 树

下一个必要的类名为`Tree`，它代表整个树。它的代码甚至比前一节中呈现的更简单，如下所示：

```cs
public class Tree<T> 
{ 
    public TreeNode<T> Root { get; set; } 
} 
```

该类只包含一个属性，`Root`。您可以使用此属性访问根节点，然后可以使用其`Children`属性获取树中其他节点的数据。

值得注意的是，`TreeNode`和`Tree`类都是泛型的，这些类使用相同的类型。例如，如果树节点应存储`string`值，则在`Tree`和`TreeNode`类的实例中应使用`string`类型。

# 示例 - 标识符的层次结构

您想看看如何在基于 C#的应用程序中使用树吗？让我们看看第一个示例。目标是构建具有几个节点的树，如下图所示。只有深色背景的节点组将在代码中呈现。但是，调整代码以自行扩展此树是一个好主意。

![](img/6afd0e1c-87c6-4057-a278-d08277502586.png)

正如您在示例中看到的那样，每个节点都存储一个整数值。因此，`int`将是`Tree`和`TreeNode`类都使用的类型。以下代码的一部分应放在`Program`类的`Main`方法中：

```cs
Tree<int> tree = new Tree<int>(); 
tree.Root = new TreeNode<int>() { Data = 100 }; 
tree.Root.Children = new List<TreeNode<int>> 
{ 
    new TreeNode<int>() { Data = 50, Parent = tree.Root }, 
    new TreeNode<int>() { Data = 1, Parent = tree.Root }, 
    new TreeNode<int>() { Data = 150, Parent = tree.Root } 
}; 
tree.Root.Children[2].Children = new List<TreeNode<int>>() 
{ 
    new TreeNode<int>()  
        { Data = 30, Parent = tree.Root.Children[2] } 
}; 
```

代码看起来相当简单，不是吗？

首先，创建`Tree`类的新实例。然后，通过创建`TreeNode`类的新实例，设置`Data`属性的值（为`100`），并将对`TreeNode`实例的引用分配给`Root`属性来配置根节点。

在接下来的几行中，指定了根节点的子节点，其值分别为`50`，`1`和`150`。对于每个节点，`Parent`属性的值都设置为对先前添加的根节点的引用。

代码的其余部分显示了如何为给定节点添加子节点，即根节点的第三个子节点，即值等于`150`的节点。在这里，只添加了一个节点，其值设置为`30`。当然，您还需要指定对父节点的引用。

就是这样！您已经创建了使用树的第一个程序。现在可以运行它，但您在控制台中看不到任何输出。如果要查看节点数据是如何组织的，可以调试程序并在调试时查看变量的值。

# 示例 - 公司结构

在前面的示例中，您看到如何将整数值用作树中每个节点的数据。但是，还可以将用户定义的类的实例存储在节点中。在此示例中，您将看到如何创建一个树，展示公司的结构，分为三个主要部门：开发、研究和销售。

在每个部门中都可以有另一个结构，例如开发团队的情况。在这里，**John Smith**是**开发部门主管**。他是**Chris Morris**的上司，后者是两名初级开发人员**Eric Green**和**Ashley Lopez**的经理。后者还是**Emily Young**的主管，后者是**开发实习生**。

以下是示例树的示意图：

![](img/d22420a3-eabe-4864-abeb-a7c8eeb5b16b.png)

正如您所看到的，每个节点应存储的信息不仅仅是一个整数值。应该有一个标识符、一个名称和一个角色。这些数据存储为`Person`类实例的属性值，如下面的代码片段所示：

```cs
public class Person 
{ 
    public int Id { get; set; } 
    public string Name { get; set; } 
    public string Role { get; set; } 

    public Person() { } 

    public Person(int id, string name, string role) 
    { 
        Id = id; 
        Name = name; 
        Role = role; 
    } 
} 
```

该类包含三个属性（`Id`，`Name`和`Role`），以及两个构造函数。第一个构造函数不带任何参数，而另一个带有三个参数，并设置特定属性的值。

除了创建一个新类之外，还需要在`Program`类的`Main`方法中添加一些代码。必要的行如下：

```cs
Tree<Person> company = new Tree<Person>(); 
company.Root = new TreeNode<Person>() 
{ 
    Data = new Person(100, "Marcin Jamro", "CEO"), 
    Parent = null 
}; 
company.Root.Children = new List<TreeNode<Person>>() 
{ 
    new TreeNode<Person>() 
    { 
        Data = new Person(1, "John Smith", "Head of Development"), 
        Parent = company.Root 
    }, 
    new TreeNode<Person>() 
    { 
        Data = new Person(50, "Mary Fox", "Head of Research"), 
        Parent = company.Root 
    }, 
    new TreeNode<Person>() 
    { 
        Data = new Person(150, "Lily Smith", "Head of Sales"), 
        Parent = company.Root 
    } 
}; 
company.Root.Children[2].Children = new List<TreeNode<Person>>() 
{ 
    new TreeNode<Person>() 
    {
        Data = new Person(30, "Anthony Black", "Sales Specialist"),
        Parent = company.Root.Children[2]
    } 
}; 
```

在第一行，创建了`Tree`类的一个新实例。值得一提的是，在创建`Tree`和`TreeNode`类的新实例时，使用了`Person`类作为指定类型。因此，你可以轻松地为每个节点存储多个简单数据。

代码的其余部分看起来与基本树的第一个示例相似。在这里，你还指定了根节点（`CEO`角色），然后配置了它的子元素（`John Smith`，`Mary Fox`和`Lily Smith`），并为现有节点之一设置了一个子节点，即`Head of Sales`的节点。

看起来简单明了吗？在下一节中，你将看到一种更受限制但非常重要和著名的树的变体：二叉树。

# 二叉树

一般来说，基本树中的每个节点可以包含任意数量的子节点。然而，在**二叉树**的情况下，一个节点不能包含超过两个子节点。这意味着它可以包含零个、一个或两个子节点。这一要求对二叉树的形状有重要影响，如下图所示展示了二叉树：

![](img/97d1ae70-d6ba-4cba-bddc-beef55c0cba2.png)

如前所述，二叉树中的节点最多可以包含两个子节点。因此，它们被称为**左子节点**和**右子节点**。在前面图中左侧显示的二叉树中，节点**21**有两个子节点，**68**为左子节点，**12**为右子节点，而节点**100**只有一个左子节点。

你有没有想过如何遍历树中的所有节点？在树的遍历过程中，你如何指定节点的顺序？有三种常见的方法：前序遍历、中序遍历和后序遍历，如下图所示：

![](img/6203e8c5-d23c-41ca-b4ec-f7addeb54a82.png)

正如你在图中所看到的，这些方法之间存在明显的差异。然而，你有没有想过如何在二叉树中应用前序遍历、中序遍历或后序遍历？让我们详细解释所有这些方法。

如果你想使用**前序遍历**方法遍历二叉树，首先需要访问根节点。然后，访问左子节点。最后，访问右子节点。当然，这样的规则不仅适用于根节点，而且适用于树中的任何节点。因此，你可以理解前序遍历的顺序为首先访问当前节点，然后访问它的左子节点（使用前序遍历递归地遍历整个左子树），最后访问它的右子节点（以类似的方式遍历右子树）。

解释可能听起来有点复杂，所以让我们看一个简单的例子，关于前面图中左侧显示的树。首先，访问根节点（即**1**）。然后，分析它的左子节点。因此，下一个访问的节点是当前节点**9**。下一步是它的左子节点的前序遍历。因此，访问**5**。由于这个节点不包含任何子节点，你可以返回到遍历时**9**是当前节点的阶段。它已经被访问过，它的左子节点也是，所以现在是时候继续到它的右子节点。在这里，首先访问当前节点**6**，然后转到它的左子节点**3**。你可以应用相同的规则来继续遍历树。最终的顺序是**1**，**9**，**5**，**6**，**3**，**4**，**2**，**7**，**8**。

如果这听起来有点令人困惑，下图应该消除任何困惑：

![](img/7b456784-0ec6-474b-9b66-0f372d545dfb.png)

该图展示了前序遍历的以下步骤，并附有额外的指示：**C**表示**当前节点**，**L**表示**左子节点**，**R**表示**右子节点**。

第二个遍历模式称为**中序遍历**。它与前序遍历方法的区别在于节点访问的顺序：首先是左子节点，然后是当前节点，然后是右子节点。如果您看一下图表中显示的具有所有三种遍历模式的示例，您会发现第一个访问的节点是**5**。为什么？开始时，分析根节点，但不访问，因为中序遍历从左子节点开始。因此，它分析节点**9**，但它也有一个左子节点**5**，所以您继续到这个节点。由于此节点没有任何子节点，因此访问当前节点（**5**）。然后，返回到当前节点为**9**的步骤，并且 - 由于其左子节点已经被访问 - 您还访问当前节点。接下来，您转到右子节点，但它有一个左子节点**3**，应该先访问。根据相同的规则，您访问二叉树中的剩余节点。最终顺序是**5**，**9**，**3**，**6**，**1**，**4**，**7**，**8**，**2**。

最后的遍历模式称为**后序遍历**，支持以下节点遍历顺序：左子节点，右子节点，然后是当前节点。让我们分析图表右侧显示的后序遍历示例。开始时，分析根节点，但不访问，因为后序遍历从左子节点开始。因此 - 与中序遍历方法一样 - 继续到节点**9**，然后**5**。然后，需要分析节点**9**的右子节点。然而，节点**6**有左子节点（**3**），应该先访问。因此，在**5**之后，访问**3**，然后**6**，然后是**9**。有趣的是，二叉树的根节点在最后访问。最终顺序是**5**，**3**，**6**，**9**，**8**，**7**，**2**，**4**，**1**。

您可以在[`en.wikipedia.org/wiki/Binary_tree`](https://en.wikipedia.org/wiki/Binary_tree)找到有关二叉树的更多信息。

在这个简短的介绍之后，让我们继续进行基于 C#的实现。

# 实现

二叉树的实现真的很简单，特别是如果您使用了已经描述的基本树的代码。为了您的方便，整个必要的代码都放在了以下部分，但只有它的新部分被详细解释。

# 节点

二叉树中的节点由`BinaryTreeNode`的实例表示，它继承自`TreeNode`泛型类，具有以下代码：

```cs
public class TreeNode<T> 
{ 
    public T Data { get; set; } 
    public TreeNode<T> Parent { get; set; } 
    public List<TreeNode<T>> Children { get; set; } 

    public int GetHeight() 
    { 
        int height = 1; 
        TreeNode<T> current = this; 
        while (current.Parent != null) 
        { 
            height++; 
            current = current.Parent; 
        } 
        return height; 
    } 
} 
```

在`BinaryTreeNode`类中，需要声明两个属性`Left`和`Right`，它们分别表示节点的两个可能的子节点。代码的相关部分如下：

```cs
public class BinaryTreeNode<T> : TreeNode<T> 
{ 
    public BinaryTreeNode() => Children =  
        new List<TreeNode<T>>() { null, null }; 

    public BinaryTreeNode<T> Left 
    { 
        get { return (BinaryTreeNode<T>)Children[0]; } 
        set { Children[0] = value; } 
    } 

    public BinaryTreeNode<T> Right 
    { 
        get { return (BinaryTreeNode<T>)Children[1]; } 
        set { Children[1] = value; } 
    } 
} 
```

此外，您需要确保子节点的集合包含确切两个项目，最初设置为`null`。您可以通过在构造函数中为`Children`属性分配默认值来实现此目标，如前面的代码所示。因此，如果要添加子节点，应将对其的引用放置为列表（`Children`属性）的第一个或第二个元素。因此，这样的集合始终具有确切两个元素，并且可以访问第一个或第二个元素而不会出现任何异常。如果它设置为任何节点，则返回对其的引用，否则返回`null`。

# 树

下一个必要的类名为`BinaryTree`。它表示整个二叉树。通过使用泛型类，您可以轻松指定存储在每个节点中的数据类型。`BinaryTree`类的实现的第一部分如下：

```cs
public class BinaryTree<T> 
{ 
    public BinaryTreeNode<T> Root { get; set; } 
    public int Count { get; set; } 
} 
```

`BinaryTree`类包含两个属性：`Root`，表示根节点（作为`BinaryTreeNode`类的实例），以及`Count`，表示树中放置的节点的总数。当然，这些不是类的唯一成员，因为它还可以配备一组关于遍历树的方法。

本书中描述的第一个遍历方法是先序遍历。作为提醒，它首先访问当前节点，然后是其左子节点，最后是右子节点。`TraversePreOrder`方法的代码如下：

```cs
private void TraversePreOrder(BinaryTreeNode<T> node,  
    List<BinaryTreeNode<T>> result) 
{ 
    if (node != null) 
    { 
        result.Add(node); 
        TraversePreOrder(node.Left, result); 
        TraversePreOrder(node.Right, result); 
    } 
} 
```

该方法接受两个参数：当前节点（`node`）和已访问节点的列表（`result`）。递归实现非常简单。首先，通过确保参数不等于`null`来检查节点是否存在。然后，将当前节点添加到已访问节点的集合中，开始对左子节点执行相同的遍历方法，最后对右子节点执行相同的遍历方法。

类似的实现也适用于中序和后序遍历模式。让我们从`TraverseInOrder`方法的代码开始：

```cs
private void TraverseInOrder(BinaryTreeNode<T> node,  
    List<BinaryTreeNode<T>> result) 
{ 
    if (node != null) 
    { 
        TraverseInOrder(node.Left, result); 
        result.Add(node); 
        TraverseInOrder(node.Right, result); 
    } 
} 
```

在这里，您递归调用`TraverseInOrder`方法来处理左子节点，将当前节点添加到已访问节点的列表中，并开始对右子节点进行中序遍历。

下一个方法与后序遍历模式有关，如下所示：

```cs
private void TraversePostOrder(BinaryTreeNode<T> node,  
    List<BinaryTreeNode<T>> result) 
{ 
    if (node != null) 
    { 
        TraversePostOrder(node.Left, result); 
        TraversePostOrder(node.Right, result); 
        result.Add(node); 
    } 
} 
```

该代码与已描述的方法非常相似，但是应用了另一种访问节点的顺序。在这里，您首先访问左子节点，然后访问右子节点，最后访问当前节点。

最后，让我们添加用于以各种模式遍历树的公共方法，该方法调用先前介绍的私有方法。相关代码如下：

```cs
public List<BinaryTreeNode<T>> Traverse(TraversalEnum mode) 
{ 
    List<BinaryTreeNode<T>> nodes = new List<BinaryTreeNode<T>>(); 
    switch (mode) 
    { 
        case TraversalEnum.PREORDER: 
            TraversePreOrder(Root, nodes); 
            break; 
        case TraversalEnum.INORDER: 
            TraverseInOrder(Root, nodes); 
            break; 
        case TraversalEnum.POSTORDER: 
            TraversePostOrder(Root, nodes); 
            break; 
    } 
    return nodes; 
} 
```

该方法只接受一个参数，即`TraversalEnum`枚举的值，选择适当的先序、中序和后序模式。`Traverse`方法使用`switch`语句根据参数的值调用适当的私有方法。

为了使用`Traverse`方法，还需要声明`TraversalEnum`枚举，如下所示：

```cs
public enum TraversalEnum 
{ 
    PREORDER, 
    INORDER, 
    POSTORDER 
} 
```

本节中描述的最后一个方法是`GetHeight`。它返回树的高度，可以理解为从任何叶节点到根节点所需的最大步数。实现如下：

```cs
public int GetHeight() 
{ 
    int height = 0; 
    foreach (BinaryTreeNode<T> node  
        in Traverse(TraversalEnum.PREORDER)) 
    { 
        height = Math.Max(height, node.GetHeight()); 
    } 
    return height; 
} 
```

该代码只是使用先序遍历遍历树的所有节点，读取当前节点的高度（使用先前描述的`TreeNode`类的`GetHeight`方法），如果大于当前最大值，则将其保存为最大值。最后返回计算出的高度。

在介绍了二叉树的主题之后，让我们看一个示例，其中使用这种数据结构来存储简单测验中的问题和答案。

# 示例 - 简单的测验

作为二叉树的一个示例，将使用一个简单的测验应用程序。测验由几个问题和答案组成，根据先前做出的决定显示。应用程序呈现问题，等待用户按下*Y*（是）或*N*（否），然后继续下一个问题或显示答案。

测验的结构以二叉树的形式创建，如下所示：

![](img/2679026d-3aac-4d4d-8868-2b46f7dfbe2f.png)

首先，用户被问及是否有应用程序开发经验。如果是，程序会询问他或她是否已经作为开发人员工作了五年以上。在肯定答案的情况下，将呈现关于申请成为高级开发人员的结果。当然，在用户做出不同决定的情况下，还会显示其他答案和问题。

简单测验的实现需要`BinaryTree`和`BinaryTreeNode`类，这些类在先前已经介绍和解释过。除此之外，还应该声明`QuizItem`类来表示单个项目，例如问题或答案。每个项目只包含文本内容，存储为`Text`属性的值。适当的实现如下：

```cs
public class QuizItem 
{ 
    public string Text { get; set; } 
    public QuizItem(string text) => Text = text; 
} 
```

在`Program`类中需要进行一些修改。让我们来看一下修改后的`Main`方法：

```cs
static void Main(string[] args) 
{ 
    BinaryTree<QuizItem> tree = GetTree(); 
    BinaryTreeNode<QuizItem> node = tree.Root; 
    while (node != null) 
    { 
        if (node.Left != null || node.Right != null) 
        { 
            Console.Write(node.Data.Text); 
            switch (Console.ReadKey(true).Key) 
            { 
                case ConsoleKey.Y: 
                    WriteAnswer(" Yes"); 
                    node = node.Left; 
                    break; 
                case ConsoleKey.N: 
                    WriteAnswer(" No"); 
                    node = node.Right; 
                    break; 
            } 
        } 
        else 
        { 
            WriteAnswer(node.Data.Text); 
            node = null; 
        } 
    } 
} 
```

在方法中的第一行，调用`GetTree`方法（如下面的代码片段所示）来构建具有问题和答案的树。然后，将根节点作为当前节点，直到到达答案为止。

首先，检查左侧或右侧子节点是否存在，即是否为问题（而不是答案）。然后，在控制台中写入文本内容，并等待用户按键。如果等于*Y*，则显示有关选择*是*选项的信息，并使用当前节点的左子节点作为当前节点。在选择*否*的情况下执行类似的操作，但然后使用当前节点的右子节点。

当用户做出的决定导致答案显示时，它会在控制台中呈现，并将`null`赋给`node`变量。因此，您会跳出`while`循环。

如前所述，`GetTree`方法用于构建具有问题和答案的二叉树。其代码如下所示：

```cs
private static BinaryTree<QuizItem> GetTree() 
{ 
    BinaryTree<QuizItem> tree = new BinaryTree<QuizItem>(); 
    tree.Root = new BinaryTreeNode<QuizItem>() 
    { 
        Data = new QuizItem("Do you have experience in developing  
            applications?"), 
        Children = new List<TreeNode<QuizItem>>() 
        { 
            new BinaryTreeNode<QuizItem>() 
            { 
                Data = new QuizItem("Have you worked as a  
                    developer for more than 5 years?"), 
                Children = new List<TreeNode<QuizItem>>() 
                { 
                    new BinaryTreeNode<QuizItem>() 
                    { 
                        Data = new QuizItem("Apply as a senior  
                            developer!") 
                    }, 
                    new BinaryTreeNode<QuizItem>() 
                    { 
                        Data = new QuizItem("Apply as a middle  
                            developer!") 
                    } 
                } 
            }, 
            new BinaryTreeNode<QuizItem>() 
            { 
                Data = new QuizItem("Have you completed  
                    the university?"), 
                Children = new List<TreeNode<QuizItem>>() 
                { 
                    new BinaryTreeNode<QuizItem>() 
                    { 
                        Data = new QuizItem("Apply for a junior  
                            developer!") 
                    }, 
                    new BinaryTreeNode<QuizItem>() 
                    { 
                        Data = new QuizItem("Will you find some  
                            time during the semester?"), 
                        Children = new List<TreeNode<QuizItem>>() 
                        { 
                            new BinaryTreeNode<QuizItem>() 
                            { 
                                Data = new QuizItem("Apply for our  
                                   long-time internship program!") 
                            }, 
                            new BinaryTreeNode<QuizItem>() 
                            { 
                                Data = new QuizItem("Apply for  
                                   summer internship program!") 
                            } 
                        } 
                    } 
                } 
            } 
        } 
    }; 
    tree.Count = 9; 
    return tree; 
} 
```

首先，创建`BinaryTree`泛型类的新实例。还配置每个节点包含`QuizItem`类的实例的数据。然后，将`Root`属性分配给`BinaryTreeNode`的新实例。

有趣的是，即使在以编程方式创建问题和答案时，您也会创建某种类似树的结构，因为您使用`Children`属性并直接在这些结构中指定项目。因此，您无需为所有问题和答案创建许多本地变量。值得注意的是，与问题相关的节点是`BinaryTreeNode`类的实例，具有两个子节点（用于*是*和*否*决定），而与答案相关的节点不能包含任何子节点。

在所提供的解决方案中，`BinaryTreeNode`实例的`Parent`属性的值未设置。如果要使用它们或获取节点或树的高度，则应自行设置它们。

最后一个辅助方法是`WriteAnswer`，代码如下：

```cs
private static void WriteAnswer(string text) 
{ 
    Console.ForegroundColor = ConsoleColor.White; 
    Console.WriteLine(text); 
    Console.ForegroundColor = ConsoleColor.Gray; 
} 
```

该方法只是在控制台中以白色显示传递的文本参数。它用于显示用户做出的决定和答案的文本内容。

简单的测验应用程序已准备就绪！您可以构建项目，启动它，并回答一些问题以查看结果。然后，让我们关闭程序并继续到下一部分，介绍二叉树数据结构的变体。

# 二叉搜索树

二叉树是一种有趣的数据结构，允许创建元素的层次结构，每个节点最多可以包含两个子节点，但没有关于节点之间关系的任何规则。因此，如果要检查二叉树是否包含给定值，需要检查每个节点，使用三种可用模式之一遍历树：前序，中序或后序。这意味着查找时间是线性的，即*O(n)*。

如果树中存在一些关于节点关系的明确规则呢？假设有这样一种情况，左子树包含小于根值的节点，而右子树包含大于根值的节点。然后，您可以将搜索值与当前节点进行比较，并决定是否应继续在左侧或右侧子树中搜索。这种方法可以显著限制检查树是否包含给定值所需的操作数量。这似乎很有趣，不是吗？

这种方法应用于**二叉搜索树**数据结构，也称为**BST**。它是一种二叉树，引入了两个关于树中节点关系的严格规则。规则规定对于任何节点：

+   其左子树中所有节点的值必须小于其值

+   其右子树中所有节点的值必须大于其值

一般来说，二叉搜索树可以包含两个或更多具有相同值的元素。但是，在本书中给出了一个简化版本，不接受多个具有相同值的元素。

实际上是什么样子？让我们看一下以下二叉搜索树的图表：

![](img/3cf99bb5-d124-494d-a022-c15038af2e46.png)

左侧显示的树包含 12 个节点。让我们检查它是否符合二叉搜索树的规则。您可以通过分析树中除了叶节点以外的每个节点来进行检查。

让我们从根节点（值为**50**）开始，它在左子树中包含四个后代节点（**40**、**30**、**45**、**43**），都小于**50**。根节点在右子树中包含七个后代节点（**60**、**80**、**70**、**65**、**75**、**90**、**100**），都大于**50**。这意味着根节点满足了二叉搜索树的规则。如果您想检查节点**80**的二叉搜索树规则，您会发现左子树中所有后代节点的值（**70**、**65**、**75**）都小于**80**，而右子树中的值（**90**、**100**）都大于**80**。您应该对树中的所有节点执行相同的验证。同样，您可以确认图表右侧的二叉搜索树遵守了规则。

然而，这两个二叉搜索树在拓扑结构上有很大的不同。它们的高度相同，但节点的数量不同——12 和 7。左边的看起来很胖，而另一个则相对瘦。哪一个更好？为了回答这个问题，让我们考虑一下在树中搜索一个值的算法。例如，搜索值**43**的过程在下图中描述和展示：

![](img/32e6b37f-f619-4268-b1b6-0c81a8af6846.png)

开始时，您取根节点的值（即**50**）并检查给定的值（**43**）是较小还是较大。它较小，所以您继续在左子树中搜索。因此，您将**43**与**40**进行比较。这次选择右子树，因为**43**大于**40**。接下来，**43**与**45**进行比较，并选择左子树。在这里，您将**43**与**43**进行比较。因此，找到了给定的值。如果您看一下树，您会发现只需要四次比较，对性能的影响是显而易见的。

因此，很明显树的形状对查找性能有很大影响。当然，拥有高度有限的胖树要比高度更大的瘦树好得多。性能提升是由于在继续在左子树或右子树中搜索时做出决策，而无需分析所有节点的值。如果节点没有两个子树，对性能的积极影响将受到限制。在最坏的情况下，当每个节点只包含一个子节点时，搜索时间甚至是线性的。然而，在理想的二叉搜索树中，查找时间是*O(log n)*操作。

您可以在[`en.wikipedia.org/wiki/Binary_search_tree`](https://en.wikipedia.org/wiki/Binary_search_tree)找到更多关于二叉搜索树的信息。

在这个简短的介绍之后，让我们继续使用 C#语言进行实现。最后，您将看到一个示例，展示了如何在实践中使用这种数据结构。

# 实现

二叉搜索树的实现比先前描述的树的变体更困难。例如，它要求您准备树中节点的插入和删除操作，这些操作不会违反二叉搜索树中元素排列的规则。此外，您需要引入一个比较节点的机制。

# 节点

让我们从表示树中单个节点的类开始。幸运的是，您可以使用已经描述的二叉树类（`BinaryTreeNode`）的实现作为基础。修改后的代码如下：

```cs
public class BinaryTreeNode<T> : TreeNode<T> 
{ 
    public BinaryTreeNode() => Children =  
        new List<TreeNode<T>>() { null, null }; 

    public BinaryTreeNode<T> Parent { get; set; } 

    public BinaryTreeNode<T> Left 
    { 
        get { return (BinaryTreeNode<T>)Children[0]; } 
        set { Children[0] = value; } 
    } 

    public BinaryTreeNode<T> Right 
    { 
        get { return (BinaryTreeNode<T>)Children[1]; } 
        set { Children[1] = value; } 
    } 

    public int GetHeight() 
    { 
        int height = 1; 
        BinaryTreeNode<T> current = this; 
        while (current.Parent != null) 
        { 
            height++; 
            current = current.Parent; 
        } 
        return height; 
    } 
} 
```

由于 BST 是二叉树的一种变体，每个节点都有对其左右子节点（如果不存在则为`null`）以及父节点的引用。节点还存储给定类型的值。正如您在前面的代码中所看到的，`BinaryTreeNode`类添加了两个成员，即`Parent`属性（`BinaryTreeNode`类型）和`GetHeight`方法。它们是从`TreeNode`类的实现中移动和调整的。最终代码如下：

```cs
public class TreeNode<T> 
{ 
    public T Data { get; set; } 
    public List<TreeNode<T>> Children { get; set; } 
} 
```

修改的原因是为开发人员提供一种简单的方法，以便在不需要从`TreeNode`到`BinaryTreeNode`进行转换的情况下访问给定节点的父节点。

# 树

整个树由`BinarySearchTree`类的实例表示，该类继承自`BinaryTree`泛型类，如下面的代码片段所示：

```cs
public class BinarySearchTree<T> : BinaryTree<T>  
    where T : IComparable 
{ 
} 
```

值得一提的是，每个节点中存储的数据类型应该是可比较的。因此，它必须实现`IComparable`接口。这种要求是必要的，因为算法需要了解值之间的关系。

当然，这不是`BinarySearchTree`类实现的最终版本。在接下来的部分中，您将看到如何添加新功能，比如查找、插入和删除节点。

# 查找

让我们来看一下`Contains`方法，它检查树中是否包含具有给定值的节点。当然，此方法考虑了有关节点排列的 BST 规则，以限制比较的数量。代码如下：

```cs
public bool Contains(T data) 
{ 
    BinaryTreeNode<T> node = Root; 
    while (node != null) 
    { 
        int result = data.CompareTo(node.Data); 
        if (result == 0) 
        { 
            return true; 
        } 
        else if (result < 0) 
        { 
            node = node.Left; 
        } 
        else 
        { 
            node = node.Right; 
        } 
    } 
    return false; 
} 
```

该方法只接受一个参数，即应在树中找到的值。在方法内部，存在`while`循环。在其中，将搜索的值与当前节点的值进行比较。如果它们相等（比较返回`0`作为结果），则找到该值，并返回`true`布尔值以通知搜索成功完成。如果搜索的值小于当前节点的值，则算法继续在以当前节点的左子节点为根的子树中搜索。否则，使用右子树。

`CompareTo`方法由`System`命名空间中的`IComparable`接口的实现提供。这种方法使得比较值成为可能。如果它们相等，则返回`0`。如果调用该方法的对象大于参数，则返回大于`0`的值。否则，返回小于`0`的值。

循环执行直到找到节点或没有合适的子节点可以跟随。

# 插入

下一个必要的操作是将节点插入 BST。这项任务有点复杂，因为您需要找到一个不会违反 BST 规则的新元素添加位置。让我们来看一下`Add`方法的代码：

```cs
public void Add(T data) 
{ 
    BinaryTreeNode<T> parent = GetParentForNewNode(data); 
    BinaryTreeNode<T> node = new BinaryTreeNode<T>()  
        { Data = data, Parent = parent }; 

    if (parent == null) 
    { 
        Root = node; 
    } 
    else if (data.CompareTo(parent.Data) < 0) 
    { 
        parent.Left = node; 
    } 
    else 
    { 
        parent.Right = node; 
    } 

    Count++; 
} 
```

该方法接受一个参数，即应添加到树中的值。在方法内部，找到应将新节点添加为子节点的父元素（使用`GetParentForNewNode`辅助方法），然后创建`BinaryTreeNode`类的新实例，并设置其`Data`和`Parent`属性的值。

在方法的后续部分，您检查找到的父元素是否等于`null`。这意味着树中没有节点，新节点应该被添加为根节点，这在将节点的引用分配给`Root`属性的行中很明显。下一个比较检查要添加的值是否小于父节点的值。在这种情况下，新节点应该被添加为父节点的左子节点。否则，新节点将被放置为父节点的右子节点。最后，树中存储的元素数量增加。

让我们来看看用于查找新节点的父元素的辅助方法：

```cs
private BinaryTreeNode<T> GetParentForNewNode(T data) 
{ 
    BinaryTreeNode<T> current = Root; 
    BinaryTreeNode<T> parent = null; 
    while (current != null) 
    { 
        parent = current; 
        int result = data.CompareTo(current.Data); 
        if (result == 0) 
        { 
            throw new ArgumentException( 
                $"The node {data} already exists."); 
        } 
        else if (result < 0) 
        { 
            current = current.Left; 
        } 
        else 
        { 
            current = current.Right; 
        } 
    } 

    return parent; 
} 
```

该方法名为`GetParentForNewNode`，只需要一个参数，即新节点的值。在这个方法中，您声明了两个变量，表示当前分析的节点（`current`）和父节点（`parent`）。这些值在`while`循环中被修改，直到算法找到新节点的合适位置。

在循环中，您将当前节点的引用存储为潜在的父节点。然后，进行比较，就像在先前描述的代码片段中一样。首先，您检查要添加的值是否等于当前节点的值。如果是，将抛出异常，因为不允许向分析版本的 BST 中添加多个具有相同值的元素。如果要添加的值小于当前节点的值，则算法继续在左子树中搜索新节点的位置。否则，使用当前节点的右子树。最后，将`parent`变量的值返回以指示找到新节点的位置。

# 删除

现在你知道如何创建一个新的 BST，向其中添加一些节点，并检查树中是否已经存在给定的值。但是，你也能从树中删除一个项目吗？当然可以！您将在本节中学习如何实现这一目标。

从树中删除节点的主要方法名为`Remove`，只需要一个参数，即应该被删除的节点的值。`Remove`方法的实现如下：

```cs
public void Remove(T data) 
{ 
    Remove(Root, data); 
} 
```

正如您所看到的，该方法只是调用另一个名为`Remove`的方法。该方法的实现更加复杂，如下所示：

```cs
private void Remove(BinaryTreeNode<T> node, T data) 
{ 
    if (node == null)
    {
        throw new ArgumentException(
            $"The node {data} does not exist.");
    }
    else if (data.CompareTo(node.Data) < 0) 
    { 
        Remove(node.Left, data); 
    } 
    else if (data.CompareTo(node.Data) > 0) 
    { 
        Remove(node.Right, data); 
    } 
    else 
    { 
        if (node.Left == null && node.Right == null) 
        { 
            ReplaceInParent(node, null); 
            Count--; 
        } 
        else if (node.Right == null) 
        { 
            ReplaceInParent(node, node.Left); 
            Count--; 
        } 
        else if (node.Left == null) 
        { 
            ReplaceInParent(node, node.Right); 
            Count--; 
        } 
        else 
        { 
            BinaryTreeNode<T> successor =  
                FindMinimumInSubtree(node.Right); 
            node.Data = successor.Data; 
            Remove(successor, successor.Data); 
        } 
    } 
} 
```

在开始时，该方法检查当前节点（`node`参数）是否存在。如果不存在，则会抛出异常。然后，`Remove`方法尝试找到要删除的节点。通过将当前节点的值与要删除的值进行比较，并递归调用`Remove`方法，尝试在当前节点的左子树或右子树中找到要删除的节点。这些操作在条件语句中执行，条件为`data.CompareTo(node.Data) < 0`和`data.CompareTo(node.Data) > 0`。

最有趣的操作是在方法的以下部分执行的。在这里，您需要处理节点删除的四种情况，即：

+   删除叶节点

+   只有左子节点的节点

+   只有右子节点的节点

+   删除具有左右子节点的节点

在第一种情况中，您只需更新父元素中对被删除节点的引用。因此，父节点到被删除节点的引用将不存在，无法在遍历树时到达。

第二种情况也很简单，因为您只需要用被删除节点的左子节点替换父元素中对被删除节点的引用。这种情况在下图中显示，演示了如何删除只有左子节点的节点**80**：

![](img/a8d6e2e8-5b7d-4c51-a014-bcc3cc5a5750.png)

第三种情况与第二种情况非常相似。因此，您只需用被删除节点的右子节点替换对被删除节点（在父元素中）的引用。

所有这三种情况都通过调用辅助方法（`ReplaceInParent`）在代码中以类似的方式处理。它接受两个参数：要删除的节点和应该在父节点中替换它的节点。因此，如果要删除叶节点，只需将`null`作为第二个参数传递，因为您不希望用其他任何东西替换已删除的节点。在仅具有一个子节点的情况下，您将传递到左侧或右侧子节点的引用。当然，您还需要递减存储在树中的元素数量的计数器。

代码的相关部分如下（对于不同情况有所不同）：

```cs
ReplaceInParent(node, node.Left); 
Count--; 
```

当然，最复杂的情况是删除具有两个子节点的节点。在这种情况下，您会在要删除的节点的右子树中找到具有最小值的节点。然后，您交换要删除的节点的值与找到的节点的值。最后，您只需要对找到的节点递归调用`Remove`方法。代码的相关部分如下所示：

```cs
BinaryTreeNode<T> successor = FindMinimumInSubtree(node.Right); 
node.Data = successor.Data; 
Remove(successor, successor.Data); 
```

重要的角色由`ReplaceInParent`辅助方法执行，其代码如下：

```cs
private void ReplaceInParent(BinaryTreeNode<T> node,  
    BinaryTreeNode<T> newNode) 
{ 
    if (node.Parent != null) 
    { 
        if (node.Parent.Left == node) 
        { 
            node.Parent.Left = newNode; 
        } 
        else 
        { 
            node.Parent.Right = newNode; 
        } 
    } 
    else 
    { 
        Root = newNode; 
    } 

    if (newNode != null) 
    { 
        newNode.Parent = node.Parent; 
    } 
} 
```

该方法接受两个参数：要删除的节点（`node`）和应该在父节点中替换它的节点（`newNode`）。如果要删除的节点不是根，则检查它是否是父节点的左子节点。如果是，则更新适当的引用，也就是将新节点设置为要删除的节点的父节点的左子节点。以类似的方式，该方法处理了要删除的节点是父节点的右子节点的情况。如果要删除的节点是根，则将替换节点设置为根。

最后，您检查新节点是否不等于`null`，也就是说，您没有删除叶节点。在这种情况下，您将`Parent`属性的值设置为指示新节点应该与要删除的节点具有相同父节点。

最后的辅助方法名为`FindMinimumInSubtree`，代码如下：

```cs
private BinaryTreeNode<T> FindMinimumInSubtree( 
    BinaryTreeNode<T> node) 
{ 
    while (node.Left != null) 
    { 
        node = node.Left; 
    } 
    return node; 
} 
```

该方法只接受一个参数，即应找到最小值的子树的根。在方法内部，使用`while`循环来获取最左边的元素。当没有左子节点时，返回`node`变量的当前值。

所呈现的 BST 实现基于[`en.wikipedia.org/wiki/Binary_search_tree`](https://en.wikipedia.org/wiki/Binary_search_tree)上显示的代码。

代码看起来相当简单，不是吗？但是，在实践中它是如何工作的呢？让我们看一下图表，描述了删除具有两个子节点的节点的过程：

![](img/ee7545af-cf87-4982-bbd0-9c8dd810140b.png)

该图显示了如何删除值为**40**的节点。为此，您需要找到继承者，也就是要删除的节点右子树中具有最小值的节点。继承者是节点**42**，它替换了节点**40**。

# 示例-BST 可视化

在阅读有关 BST 的部分时，您已经了解了有关数据结构的很多知识。因此，现在是时候创建一个示例程序，以查看这种树的变体如何运作。该应用程序将展示如何创建 BST，手动添加一些节点（使用先前呈现的插入方法），删除节点，遍历树，并在控制台中可视化树。

让我们调整`Program`类的代码，如下所示：

```cs
class Program 
{ 
    private const int COLUMN_WIDTH = 5; 

    public static void Main(string[] args) 
    { 
        Console.OutputEncoding = Encoding.UTF8; 

        BinarySearchTree<int> tree = new BinarySearchTree<int>(); 
        tree.Root = new BinaryTreeNode<int>() { Data = 100 }; 
        tree.Root.Left = new BinaryTreeNode<int>()  
            { Data = 50, Parent = tree.Root }; 
        tree.Root.Right = new BinaryTreeNode<int>()  
            { Data = 150, Parent = tree.Root }; 
        tree.Count = 3; 
        VisualizeTree(tree, "The BST with three nodes  
            (50, 100, 150):"); 

        tree.Add(75); 
        tree.Add(125); 
        VisualizeTree(tree, "The BST after adding two nodes  
            (75, 125):"); (...) 

        tree.Remove(25); 
        VisualizeTree(tree,  
            "The BST after removing the node 25:"); (...) 

        Console.Write("Pre-order traversal:\t"); 
        Console.Write(string.Join(", ", tree.Traverse( 
            TraversalEnum.PREORDER).Select(n => n.Data))); 
        Console.Write("\nIn-order traversal:\t"); 
        Console.Write(string.Join(", ", tree.Traverse( 
            TraversalEnum.INORDER).Select(n => n.Data))); 
        Console.Write("\nPost-order traversal:\t"); 
        Console.Write(string.Join(", ", tree.Traverse( 
            TraversalEnum.POSTORDER).Select(n => n.Data))); 
    } 
```

一开始，通过创建`BinarySearchTree`类的新实例来准备一个新树（其中节点存储整数值）。通过手动配置，添加了三个节点，并指示了适当的子节点和父节点元素的引用。代码的相关部分如下：

```cs
BinarySearchTree<int> tree = new BinarySearchTree<int>(); 
tree.Root = new BinaryTreeNode<int>() { Data = 100 }; 
tree.Root.Left = new BinaryTreeNode<int>()  
    { Data = 50, Parent = tree.Root }; 
tree.Root.Right = new BinaryTreeNode<int>()  
    { Data = 150, Parent = tree.Root }; 
tree.Count = 3; 
```

然后，使用`Add`方法向树中添加一些节点，并使用`VisualizeTree`方法可视化树的当前状态，如下所示：

```cs
tree.Add(125); 
VisualizeTree(tree, "The BST after adding two nodes (75, 125):"); 
```

接下来的一系列操作与从树中删除各种节点以及可视化特定更改相关。代码如下：

```cs
tree.Remove(25); 
VisualizeTree(tree, "The BST after removing the node 25:"); 
```

最后，展示了所有三种遍历模式。与前序遍历相关的代码部分如下：

```cs
Console.WriteLine("Pre-order traversal:\t"); 
Console.Write(string.Join(", ",  
    tree.Traverse(TraversalEnum.PREORDER).Select(n => n.Data))); 
```

另一个有趣的任务是在控制台中开发树的可视化。这样的功能非常有用，因为它允许舒适快速地观察树，而无需在 IDE 中调试应用程序并展开工具提示中的当前变量值。然而，在控制台中呈现树并不是一项简单的任务。幸运的是，您不需要担心，因为您将在本节中学习如何实现这样的功能。

首先，让我们看一下`VisualizeTree`方法：

```cs
private static void VisualizeTree( 
    BinarySearchTree<int> tree, string caption) 
{ 
    char[][] console = InitializeVisualization( 
        tree, out int width); 
    VisualizeNode(tree.Root, 0, width / 2, console, width); 
    Console.WriteLine(caption); 
    foreach (char[] row in console) 
    { 
        Console.WriteLine(row); 
    } 
} 
```

该方法接受两个参数：代表整个树的`BinarySearchTree`类的实例，以及应该显示在可视化上方的标题。在方法内部，使用`InitializeVisualization`辅助方法初始化了不规则数组（其中包含应在控制台中显示的字符）。然后，调用`VisualizeNode`递归方法，将不同部分的不规则数组填充为有关树中特定节点的数据。最后，在控制台中写入标题和缓冲区（由不规则数组表示）中的所有行。

下一个有趣的方法是`InitializeVisualization`，它创建了前面提到的不规则数组，如下面的代码片段所示：

```cs
private static char[][] InitializeVisualization( 
    BinarySearchTree<int> tree, out int width) 
{ 
    int height = tree.GetHeight(); 
    width = (int)Math.Pow(2, height) - 1; 
    char[][] console = new char[height * 2][]; 
    for (int i = 0; i < height * 2; i++) 
    { 
        console[i] = new char[COLUMN_WIDTH * width]; 
    } 
    return console; 
}
```

不规则数组包含的行数等于树的高度乘以`2`，以便为连接节点与父节点的线留出空间。列数根据公式*宽度* * 2*^(高度)* - 1 计算，其中*宽度*是常量值`COLUMN_WIDTH`，*高度*是树的高度。如果您在控制台中查看结果，这些值可能更容易理解：

```cs
                                        100
                    ┌-------------------+-------------------┐
                    50                                      150
          ┌---------+---------┐                  ┌---------+---------┐
          25                  75                  125                 175
                               +----┐        ┌----+----┐
                                   90        110       135

```

在这里，不规则数组有 8 个元素。每个都是一个包含 75 个元素的数组。当然，您可以将其理解为具有 8 行和 75 列的屏幕缓冲区。

在`VisualizeTree`方法中，调用了`VisualizeNode`。您是否有兴趣了解它是如何工作的，以及如何呈现节点的值以及线条？如果是的话，让我们看一下它的代码，如下所示：

```cs
private static void VisualizeNode(BinaryTreeNode<int> node, 
    int row, int column, char[][] console, int width) 
{ 
    if (node != null) 
    { 
        char[] chars = node.Data.ToString().ToCharArray(); 
        int margin = (COLUMN_WIDTH - chars.Length) / 2; 
        for (int i = 0; i < chars.Length; i++) 
        { 
            console[row][COLUMN_WIDTH * column + i + margin]  
                = chars[i]; 
        } 

        int columnDelta = (width + 1) /  
            (int)Math.Pow(2, node.GetHeight() + 1); 
        VisualizeNode(node.Left, row + 2, column - columnDelta,  
            console, width); 
        VisualizeNode(node.Right, row + 2, column + columnDelta,  
            console, width); 
        DrawLineLeft(node, row, column, console, columnDelta); 
        DrawLineRight(node, row, column, console, columnDelta); 
    } 
} 
```

`VisualizeNode`方法接受五个参数：用于可视化的当前节点（`node`）、行的索引（`row`）、列的索引（`column`）、作为缓冲区的不规则数组（`console`）和宽度（`width`）。在方法内部，检查当前节点是否存在。如果存在，则获取节点的值作为`char`数组，计算边距，并将`char`数组（表示值的基于字符的表示）写入缓冲区（`console`变量）。

在接下来的代码中，为当前节点的左右子节点调用了`VisualizeNode`方法。当然，您需要调整行的索引（加`2`）和列的索引（加或减计算出的值）。

最后，通过调用`DrawLineLeft`和`DrawLineRight`方法来绘制线条。第一个方法在以下代码片段中呈现：

```cs
private static void DrawLineLeft(BinaryTreeNode<int> node,  
    int row, int column, char[][] console, int columnDelta) 
{ 
    if (node.Left != null) 
    { 
        int startColumnIndex =  
            COLUMN_WIDTH * (column - columnDelta) + 2; 
        int endColumnIndex = COLUMN_WIDTH * column + 2; 
        for (int x = startColumnIndex + 1;  
            x < endColumnIndex; x++) 
        { 
            console[row + 1][x] = '-'; 
        } 
        console[row + 1][startColumnIndex] = '\u250c'; 
        console[row + 1][endColumnIndex] = '+'; 
    } 
} 
```

该方法还接受五个参数：应该绘制线的当前节点（`node`）、行索引（`row`）、列索引（`column`）、作为缓冲区的嵌套数组（`console`）和在`VisualizeNode`方法中计算的增量值（`columnDelta`）。首先，你检查当前节点是否包含左子节点，因为只有在这种情况下才需要绘制线的左部分。如果是这样，你计算列的起始和结束索引，并用破折号填充嵌套数组的适当元素。最后，在绘制的线将与另一个元素的右线连接的地方，加入加号到嵌套数组中。此外，Unicode 字符┌（`\u250c`）也被添加到线的另一侧，以创建用户友好的可视化。

几乎以相同的方式，你可以为当前节点绘制右线。当然，你需要调整代码以计算列的起始和结束索引，并更改用于表示线方向变化的字符。`DrawLineRight`方法的最终代码版本如下：

```cs
private static void DrawLineRight(BinaryTreeNode<int> node, 
    int row, int column, char[][] console, int columnDelta) 
{ 
    if (node.Right != null) 
    { 
        int startColumnIndex = COLUMN_WIDTH * column + 2; 
        int endColumnIndex =  
            COLUMN_WIDTH * (column + columnDelta) + 2; 
        for (int x = startColumnIndex + 1;  
            x < endColumnIndex; x++) 
        { 
            console[row + 1][x] = '-'; 
        } 
        console[row + 1][startColumnIndex] = '+'; 
        console[row + 1][endColumnIndex] = '\u2510'; 
    } 
} 
```

就是这样！你已经编写了构建项目、启动程序并看到它运行所需的全部代码。启动后，你将看到第一个 BST，如下所示：

```cs
    The BST with three nodes (50, 100, 150):
          100
     ┌----+----┐
     50        150 
```

在添加了下一个两个节点`75`和`125`之后，BST 看起来有点不同：

```cs
    The BST after adding two nodes (75, 125):
                    100
          ┌---------+---------┐
          50                  150
           +----┐        ┌----+
               75        125
```

然后，你执行下一个五个元素的插入操作。这些操作对树形状有非常明显的影响，如在控制台中呈现的那样：

```cs
    The BST after adding five nodes (25, 175, 90, 110, 135):
                                        100
                    ┌-------------------+-------------------┐
                    50                                      150
          ┌---------+---------┐                  ┌---------+---------┐
          25                  75                  125                 175
                               +----┐        ┌----+----┐
                                   90        110       135  
```

在添加了 10 个元素后，程序展示了删除特定节点对树形状的影响。首先，让我们删除值为`25`的叶节点：

```cs
    The BST after removing the node 25:
                                        100
                    ┌-------------------+-------------------┐
                    50                                      150
                    +---------┐                   ┌---------+---------┐
                              75                  125                 175
                              +----┐         ┌----+----┐
                                   90        110       135 
```

然后，程序检查删除只有一个子节点的节点，即右侧节点。有趣的是右子节点也有一个右子节点。然而，在这种情况下，呈现的算法也能正常工作，你会得到以下结果：

```cs
    The BST after removing the node 50:
                                        100
                    ┌-------------------+-------------------┐
                    75                                      150
                    +----┐                        ┌---------+---------┐
                         90                       125                 175
                                             ┌----+----┐
                                             110       135  
```

最后的删除操作是最复杂的，因为它需要你删除具有两个子节点的节点，并且还扮演着根的角色。在这种情况下，找到根的右子树中最左边的元素，并替换要删除的节点，如树的最终视图所示：

```cs
    The BST after removing the node 100:
                                        110
                     ┌-------------------+-------------------┐
                    75                                      150
                    +---------┐                   ┌---------+---------┐
                              90                  125                 175
                                                  +----┐
                                                       135
```

还有一组操作剩下——以三种不同的方式遍历树：前序、中序和后序。应用程序呈现以下结果：

```cs
    Pre-order traversal:    110, 75, 90, 150, 125, 135, 175
    In-order traversal:     75, 90, 110, 125, 135, 150, 175
    Post-order traversal:   90, 75, 135, 125, 175, 150, 110
```

创建的应用程序看起来相当令人印象深刻，不是吗？你不仅从头开始创建了二叉搜索树的实现，还为在控制台中可视化它做好了准备。干得好！

让我们再来看看中序遍历方法的结果。正如你所看到的，它会给出二叉搜索树中按升序排序的节点。

然而，你能看到创建的解决方案存在潜在问题吗？如果你只从树的给定区域删除节点，或者插入已排序的值，会怎么样？这可能意味着，具有适当宽度深度比的胖树可能变成瘦树。在最坏的情况下，它甚至可能被描述为一个列表，其中所有节点只有一个子节点。你有没有想法如何解决不平衡树的问题，并始终保持它们平衡？如果没有，让我们继续到下一节，介绍两种自平衡树的变体。

# AVL 树

在这一节中，你将了解一种自平衡树的变体，它在添加和删除节点时始终保持树的平衡。然而，为什么这么重要呢？如前所述，查找时间的性能取决于树的形状。在节点的组织不当形成列表的情况下，查找给定值的过程可能是*O(n)*操作。通过正确排列树，性能可以显著提高到*O(log n)*。

您知道 BST 很容易变成**失衡树**吗？让我们对树添加以下九个数字进行简单测试，从 1 到 9。然后，您将得到左侧图表中显示的形状的树。然而，相同的值可以以另一种方式排列，作为**平衡树**，具有明显更好的宽度深度比，如右侧图表所示：

![](img/1c2951c6-4cf9-43dd-88e7-a0b612a15078.png)

您知道什么是失衡和平衡树，以及自平衡树的目的，但 AVL 树是什么？它是如何工作的？在使用这种数据结构时应该考虑哪些规则？

AVL 树是具有附加要求的二叉搜索树，对于每个节点，其左右子树的高度不能相差超过一。当然，在向树中添加和删除节点后，必须保持这个规则。**旋转**起着重要作用，用于修复节点的不正确排列。

在谈论 AVL 树时，还必须指出这种数据结构的性能。在这种情况下，插入、删除和查找的平均和最坏情况都是*O(log n)*，因此与二叉搜索树相比，在最坏情况下有显着的改进。

您可以在[`en.wikipedia.org/wiki/AVL_tree`](https://en.wikipedia.org/wiki/AVL_tree)找到有关 AVL 树的更多信息。

在这个简短的介绍之后，让我们继续实现。

# 实现

AVL 树的实现，包括保持树平衡状态所需的各种旋转，似乎相当复杂。幸运的是，您不需要从头开始创建其实现，因为您可以使用其中一个可用的 NuGet 包，例如**Adjunct**，它将用于创建我们的示例。

有关 Adjunct 库的更多信息可以在以下网址找到：

+   [`adjunct.codeplex.com/`](http://adjunct.codeplex.com/)

+   [`www.nuget.org/packages/adjunct-System.DataStructures.AvlTree/`](https://www.nuget.org/packages/adjunct-System.DataStructures.AvlTree/)。

该软件包为开发人员提供了一些类，可用于创建基于 C#的应用程序。让我们专注于`AvlTree`泛型类，它代表 AVL 树。该类非常易于使用，因此您无需了解 AVL 树的所有内部细节，就可以轻松地从中受益。

例如，`AvlTree`类配备有`Add`方法，该方法在树中的适当位置插入新节点。您可以使用`Remove`方法轻松删除节点。此外，您可以通过调用`Height`方法获取给定节点的高度。还可以使用`GetBalanceFactor`获取给定节点的平衡因子，该平衡因子是左右子树高度之差计算得出的。

另一个重要的类是`AvlTreeNode`。它实现了`IBinaryTreeNode`接口，并包含四个属性，表示节点的高度（`Height`），左右节点的引用（`Left`和`Right`），以及节点中存储的值（`Value`），在创建类的实例时指定了类型。

# 示例-保持树平衡

AVL 树的介绍中提到，有一个非常简单的测试可以导致 BST 树失衡。您只需添加有序数字即可创建一个又长又瘦的树。因此，让我们尝试创建一个使用`Adjunct`库实现的 AVL 树的示例，添加完全相同的数据集。

`Program`类中`Main`方法中的代码如下：

```cs
AvlTree<int> tree = new AvlTree<int>(); 
for (int i = 1; i < 10; i++) 
{ 
    tree.Add(i); 
} 

Console.WriteLine("In-order: "  
    + string.Join(", ", tree.GetInorderEnumerator())); 
Console.WriteLine("Post-order: "  
    + string.Join(", ", tree.GetPostorderEnumerator())); 
Console.WriteLine("Breadth-first: "  
    + string.Join(", ", tree.GetBreadthFirstEnumerator())); 

AvlTreeNode<int> node = tree.FindNode(8); 
Console.WriteLine($"Children of node {node.Value} (height =  
    {node.Height}): {node.Left.Value} and {node.Right.Value}."); 
```

首先，创建`AvlTree`类的新实例，并指示节点将存储整数值。然后，使用`for`循环将以下数字（从 1 到 9）添加到树中，使用`Add`方法。循环执行后，树应包含 9 个节点，按照 AVL 树的规则排列。

此外，您可以使用常规方法遍历树：中序（`GetInorderEnumerator`），后序（`GetPostorderEnumerator`）和广度优先（`GetBreadthFirstEnumerator`）方法。您已经了解了前两种方法，但是**广度优先遍历**是什么？它的目的是首先访问同一深度上的所有节点，然后继续到下一深度，直到达到最大深度。

当您运行应用程序时，您将收到以下遍历的结果：

```cs
    In-order: 1, 2, 3, 4, 5, 6, 7, 8, 9
    Post-order: 1, 3, 2, 5, 7, 9, 8, 6, 4
    Breadth-first: 4, 2, 6, 1, 3, 5, 8, 7, 9
```

代码的最后部分显示了 AVL 树的查找功能，使用`FindNode`方法。它用于获取表示具有给定值的节点的`AvlTreeNode`实例。然后，您可以轻松地获取有关节点的各种数据，例如其高度，以及`AvlTreeNode`类的属性的左右子节点的值。有关查找功能的控制台输出部分如下：

```cs
    Children of node 8 (height = 2): 7 and 9.
```

简单、方便，而且不需要太多的开发工作——这很准确地描述了应用其中一个可用包来支持 AVL 树的过程。通过使用它，您无需自己准备复杂的代码，可能出现的问题数量也可以得到显著减少。

# 红黑树

**红黑树**，也称为**RBT**，是自平衡二叉搜索树的下一个变体。作为 BST 的变体，这种数据结构要求维护标准的 BST 规则。此外，必须考虑以下规则：

+   每个节点必须被着为红色或黑色。因此，您需要为存储颜色的节点添加额外的数据。

+   所有具有值的节点不能是叶节点。因此，NIL 伪节点应该用作树中的叶子节点，而所有其他节点都是内部节点。此外，所有 NIL 伪节点必须是黑色的。

+   如果一个节点是红色，那么它的两个子节点必须是黑色。

+   对于任何节点，到后代叶子节点（即 NIL 伪节点）的路径上黑色节点的数量必须相同。

适当的 RBT 如下图所示：

![](img/897476a0-63da-4e1d-9ae4-d1330f1566d1.png)

树由九个节点组成，每个节点都着为红色或黑色。值得一提的是 NIL 伪节点，它们被添加为叶子节点。如果您再次查看前面列出的规则集，您可以确认在这种情况下所有这些规则都得到了遵守。

与 AVL 树类似，RBT 在添加或删除节点后也必须维护规则。在这种情况下，恢复 RBT 属性的过程更加复杂，因为它涉及**重新着色**和**旋转**。幸运的是，您无需了解和理解内部细节，这些细节相当复杂，才能从这种数据结构中受益并将其应用于您的项目中。

在谈论这种自平衡 BST 的变体时，还值得注意性能。在平均和最坏情况下，插入、删除和查找都是*O(log n)*操作，因此它们与 AVL 树的情况相同，并且在最坏情况下与 BST 相比要好得多。

您可以在[`en.wikipedia.org/wiki/Red-black_tree`](https://en.wikipedia.org/wiki/Red-black_tree)找到有关 RBT 的更多信息。

您已经学习了一些关于 RBT 的基本信息，所以让我们继续使用其中一个可用的库来实现。

# 实施

如果您想在应用程序中使用 RBT，您可以从头开始实现它，也可以使用其中一个可用的库，例如`TreeLib`，您可以使用 NuGet 软件包管理器轻松安装它。该库支持几种树，其中包括 RBT。

您可以在[`programmatom.github.io/TreeLib/`](http://programmatom.github.io/TreeLib/)和[`www.nuget.org/packages/TreeLib`](https://www.nuget.org/packages/TreeLib)找到有关该库的更多信息。

由于该库为开发人员提供了许多类，因此最好查看与 RBT 相关的类。第一个类名为`RedBlackTreeList`，表示 RBT。它是一个通用类，因此您可以轻松指定存储在每个节点中的数据类型。

该类包含一组方法，包括`Add`用于向树中插入新元素，`Remove`用于删除具有特定值的节点，`ContainsKey`用于检查树是否包含给定值，以及`Greatest`和`Least`用于返回树中存储的最大和最小值。此外，该类配备了几种遍历节点的变体，包括枚举器。

# 示例-RBT 相关功能

与 AVL 树一样，让我们使用外部库为 RBT 准备示例。简单的程序将展示如何创建新树，添加元素，删除特定节点，并从库的其他功能中受益。

让我们看一下以下代码片段，它应该添加到`Program`类中的`Main`方法中。第一部分如下：

```cs
RedBlackTreeList<int> tree = new RedBlackTreeList<int>(); 
for (int i = 1; i <= 10; i++) 
{ 
    tree.Add(i); 
} 
```

在这里，创建了`RedBlackTreeList`类的新实例。指定节点将存储整数值。然后，使用`for`循环将 10 个数字（从 1 到 10 排序）添加到树中，使用`Add`方法。执行后，具有 10 个元素的正确排列的 RBT 应该准备就绪。

在下一行中，使用`Remove`方法删除值等于 9 的节点：

```cs
tree.Remove(9); 
```

以下代码行检查树是否包含值等于`5`的节点。然后使用返回的布尔值在控制台中呈现消息：

```cs
bool contains = tree.ContainsKey(5); 
Console.WriteLine( 
    "Does value exist? " + (contains ? "yes" : "no")); 
```

代码的下一部分显示了如何使用`Count`属性以及`Greatest`和`Least`方法。这些功能允许计算树中元素的总数，以及存储在其中的最小和最大值。相关的代码行如下：

```cs
uint count = tree.Count; 
tree.Greatest(out int greatest); 
tree.Least(out int least); 
Console.WriteLine( 
    $"{count} elements in the range {least}-{greatest}"); 
```

在使用树数据结构时，您可能需要一种获取节点值的方法。您可以使用`GetEnumerable`方法来实现这个目标，如下所示：

```cs
Console.WriteLine( 
    "Values: " + string.Join(", ", tree.GetEnumerable())); 
```

在树中遍历节点的另一种方法涉及`foreach`循环，如以下代码片段所示：

```cs
Console.Write("Values: "); 
foreach (EntryList<int> node in tree) 
{ 
    Console.Write(node + " "); 
} 
```

正如您所看到的，使用`TreeLib`库非常简单，您可以在几分钟内将其添加到您的应用程序中。但是，在启动程序后控制台中显示的结果是什么？让我们看看：

```cs
    Does value exist? yes
    9 elements in the range 1-10
    Values: 1, 2, 3, 4, 5, 6, 7, 8, 10
    Values: 1 2 3 4 5 6 7 8 10

```

值得注意的是，`TreeLib`并不是唯一支持 RBT 的软件包，因此最好看看各种解决方案，并选择最适合您需求的软件包。

您已经到达关于自平衡二叉搜索树部分的章节的末尾。现在，让我们继续进行与堆相关的最后一部分。它们是什么，为什么它们位于树的章节中？您很快就会得到这些问题的答案以及许多其他问题的答案！

# 二叉堆

**堆**是树的另一种变体，存在两个版本：**最小堆**和**最大堆**。对于它们中的每一个，必须满足一个额外的属性：

+   对于最小堆：每个节点的值必须大于或等于其父节点的值

+   对于最大堆：每个节点的值必须小于或等于其父节点的值

这些规则起着非常重要的作用，因为它们规定了根节点始终包含最小值（在最小堆中）或最大值（在最大堆中）。因此，它是实现优先队列的便捷数据结构，详见第三章 *栈和队列*。

堆有许多变体，包括**二叉堆**，这是本节的主题。在这种情况下，堆必须符合先前提到的规则之一（取决于种类：最小堆或最大堆），并且必须遵守**完全二叉树**规则，该规则要求每个节点不能包含超过两个子节点，以及树的所有层都必须是完全填充的，除了最后一层，该层必须从左到右填充，并且右侧可能有一些空间。

让我们来看一下以下两个二叉堆：

![](img/57b79610-a0d6-41ec-beea-325932ddc8fa.png)

您可以轻松检查两个堆是否遵守所有规则。例如，让我们验证最小堆变体（左侧显示）中值等于**20**的节点的堆属性。该节点有两个子节点，值分别为**35**和**50**，均大于**20**。同样，您可以检查堆中的其余节点。二叉树规则也得到了遵守，因为每个节点最多包含两个子节点。最后一个要求是树的每一层都是完全填充的，除了最后一层不需要完全填充，但必须从左到右包含节点。在最小堆示例中，有三个层是完全填充的（分别有一个、两个和四个节点），而最后一层包含两个节点（**25**和**70**），位于最左边的两个位置。同样，您可以确认右侧显示的最大堆是否正确配置。

在这个关于堆的简短介绍，特别是关于二叉堆的介绍中，值得一提的是其广泛的应用范围。正如前面提到的，这种数据结构是实现优先队列的便捷方式，可以插入新值并移除最小值（在最小堆中）或最大值（在最大堆中）。此外，堆还用于堆排序算法，该算法将在接下来的示例中进行描述。该数据结构还有许多其他应用，例如在图算法中。

您可以在[`en.wikipedia.org/wiki/Binary_heap`](https://en.wikipedia.org/wiki/Binary_heap)找到有关二叉堆的更多信息。

您准备好看堆的实现了吗？如果是的话，让我们继续到下一节，介绍支持堆的可用库之一。

# 实现

二叉堆可以从头开始实现，也可以使用一些已有的实现。其中一个解决方案名为`Hippie`，可以通过 NuGet 软件包管理器安装到项目中。该库包含了堆的几个变体的实现，包括二叉堆、二项式堆和斐波那契堆，这些都在本书的本章中进行了介绍和描述。

您可以在[`github.com/pomma89/Hippie`](https://github.com/pomma89/Hippie)和[`www.nuget.org/packages/Hippie`](https://www.nuget.org/packages/Hippie)找到有关该库的更多信息。

该库包含了一些类，比如通用类`MultiHeap`，它适用于各种堆的变体，包括二叉堆。但是，如果同一个类用于二叉堆、二项式堆和斐波那契堆，那么您如何选择要使用哪种类型的堆呢？您可以使用`HeapFactory`类的静态方法来解决这个问题。例如，可以使用`NewBinaryHeap`方法创建二叉堆，如下所示：

```cs
MultiHeap<int> heap = HeapFactory.NewBinaryHeap<int>(); 
```

`MultiHeap`类配备了一些属性，例如用于获取堆中元素总数的`Count`和用于检索最小值的`Min`。此外，可用的方法允许添加新元素（`Add`），删除特定项（`Remove`），删除最小值（`RemoveMin`），删除所有元素（`Clear`），检查给定值是否存在于堆中（`Contains`）以及合并两个堆（`Merge`）。

# 示例-堆排序

作为使用`Hippie`库实现的二进制堆的示例，堆排序算法如下所示。应该将基于 C#的实现添加到`Program`类中的`Main`方法中，如下所示：

```cs
List<int> unsorted = new List<int>() { 50, 33, 78, -23, 90, 41 }; 
MultiHeap<int> heap = HeapFactory.NewBinaryHeap<int>(); 
unsorted.ForEach(i => heap.Add(i)); 
Console.WriteLine("Unsorted: " + string.Join(", ", unsorted)); 

List<int> sorted = new List<int>(heap.Count); 
while (heap.Count > 0) 
{ 
    sorted.Add(heap.RemoveMin()); 
} 
Console.WriteLine("Sorted: " + string.Join(", ", sorted)); 
```

正如您所看到的，实现非常简单和简短。首先，您创建一个包含未排序整数值的列表作为算法的输入。然后，准备一个新的二进制堆，并将每个输入值添加到堆中。在这个阶段，从输入列表中的元素写入控制台。

在代码的下一部分中，创建了一个新列表。它将包含排序后的值，因此它将包含算法的结果。然后，使用`while`循环在每次迭代中从堆中删除最小值。循环执行，直到堆中没有元素为止。最后，在控制台中显示排序后的列表。

堆排序算法的时间复杂度为*O(n * log(n))*。

当构建项目并运行应用程序时，您将看到以下结果：

```cs
    Unsorted: 50, 33, 78, -23, 90, 41
    Sorted: -23, 33, 41, 50, 78, 90

```

如前所述，二进制堆并不是堆的唯一变体。除其他外，二项堆是非常有趣的方法之一，这是下一节的主题。

# 二项堆

另一种堆是**二项堆**。这种数据结构由一组具有不同顺序的**二项树**组成。顺序为*0*的二项树只是一个单个节点。您可以使用两个顺序为*n-1*的二项树构造顺序为*n*的树。其中一个应该作为第一个树的父节点的最左子节点附加。听起来有点复杂，但以下图表应该消除任何困惑：

![](img/6d4e9b3e-b52d-44ad-a437-08ea02ad2aa7.png)

如前所述，顺序为**0**的二项树只是一个单个节点，如左侧所示。顺序为**1**的树由两个顺序为**0**的树（用虚线边框标记）连接在一起。在顺序为**2**的树的情况下，使用两个顺序为**1**的树。第二个作为第一个树的父节点的最左子节点附加。以同样的方式，您可以配置具有以下顺序的二项树。

然而，您如何知道二项堆中应该放置多少个二项树，以及它们应该包含多少个节点？答案可能有点令人惊讶，因为您需要准备节点数的二进制表示。例如，让我们创建一个包含**13**个元素的二项堆。数字**13**的二进制表示如下：**1101**，即*1*2⁰ + 0*2¹ + 1*2² + 1*2³*。

需要获取集合位的基于零的位置，即在这个例子中的**0**，**2**和**3**。这些位置表示应该配置的二项树的顺序：

![](img/9c7fc557-a3a4-4140-a694-2fff25163a83.png)

此外，在二项堆中不能有两个具有相同顺序（例如两个顺序为**2**的树）。还值得注意的是，每个二项树必须保持最小堆属性。

您可以在[`en.wikipedia.org/wiki/Binomial_heap`](https://en.wikipedia.org/wiki/Binomial_heap)找到有关二项堆的更多信息。

二项堆的实现比二进制堆复杂得多。因此，最好使用现有的实现之一，而不是从头开始编写自己的实现。正如在二进制堆的情况下所述，`Hippie`库是一个支持各种堆变体的解决方案，包括二项堆。

可能会让人惊讶，但与二进制堆的示例相比，代码中唯一的区别是在创建`MultiHeap`类的新实例的那一行进行了修改。为了支持二项堆，你需要使用`HeapFactory`类中的`NewBinomialHeap`方法，如下所示：

```cs
MultiHeap<int> heap = HeapFactory.NewBinomialHeap<int>(); 
```

不需要进行更多的更改！现在你可以执行剩下的操作，比如插入或删除元素，方式与二进制堆的情况完全相同。

你已经了解了两种堆，即二进制堆和二项堆。在接下来的部分中，将简要介绍斐波那契堆。

# 斐波那契堆

**斐波那契堆**是堆的一个有趣的变体，某些方面类似于二项堆。首先，它也由许多树组成，但对于每棵树的形状没有约束，因此比二项堆灵活得多。此外，堆中允许有多棵具有完全相同形状的树。

斐波那契堆的一个示例如下：

![](img/6ebf3b19-9827-4b21-b359-3ed027961a35.png)

其中一个重要的假设是每棵树都是最小堆。因此，整个斐波那契堆中的最小值肯定是其中一棵树的根节点。此外，所呈现的数据结构支持以“懒惰”的方式执行各种操作。这意味着除非真正必要，否则不执行额外的复杂操作。例如，它可以将一个新节点添加为只有一个节点的新树。

你可以在[`en.wikipedia.org/wiki/Fibonacci_heap`](https://en.wikipedia.org/wiki/Fibonacci_heap)找到更多关于斐波那契堆的信息。

与二项堆类似，斐波那契堆的实现也不是一项简单的任务，需要对这种数据结构的内部细节有很好的理解。因此，如果你需要在你的应用程序中使用斐波那契堆，最好使用现有的实现之一，而不是从头开始编写自己的实现。正如之前所述，`Hippie`库是一个支持许多堆变体的解决方案，包括斐波那契堆。

值得一提的是，与二进制和二项堆相比，代码中唯一的区别是在创建`MultiHeap`类的新实例的那一行进行了修改。为了支持斐波那契堆，你需要使用`HeapFactory`类中的`NewFibonacciHeap`方法，如下所示：

```cs
MultiHeap<int> heap = HeapFactory.NewFibonacciHeap<int>(); 
```

就是这样！你刚刚读了一个关于斐波那契堆的简要介绍，作为堆的另一种变体，因此也是树的另一种类型。这是本章的最后一个主题，所以是时候进行总结了。

# 总结

当前章节是本书迄今为止最长的章节。然而，它包含了许多关于树变体的信息。这些数据结构在许多算法中扮演着非常重要的角色，了解更多关于它们的知识以及如何在你的应用程序中使用它们是很有益的。因此，本章不仅包含简短的理论介绍，还包括图表、解释和代码示例。

一开始描述了树的概念。作为提醒，树由节点组成，包括一个根。根节点不包含父节点，而所有其他节点都包含。每个节点可以有任意数量的子节点。同一节点的子节点可以被称为兄弟节点，而没有子节点的节点被称为叶子节点。

树的各种变体都遵循这种结构。章节中描述的第一种是二叉树。在这种情况下，一个节点最多可以包含两个子节点。然而，BST 的规则更加严格。对于这种树中的任何节点，其左子树中所有节点的值必须小于节点的值，而右子树中所有节点的值必须大于节点的值。BST 具有非常广泛的应用，并且可以显著提高查找性能。不幸的是，很容易在向树中添加排序值时使树失衡。因此，性能的积极影响可能会受到限制。

为了解决这个问题，可以使用某种自平衡树，它在添加或删除节点时始终保持平衡。在本章中，介绍了两种自平衡树的变体：AVL 树和 RBT。第一种类型有额外的要求，即对于每个节点，其左右子树的高度不能相差超过一。RBT 稍微复杂一些，因为它引入了将节点着色为红色或黑色的概念，以及 NIL 伪节点。此外，要求如果一个节点是红色，那么它的两个子节点必须是黑色，并且对于任何节点，到后代叶子的路径上的黑色节点数量必须相同。正如您在分析这些数据结构时所看到的，它们的实现要困难得多。因此，本章还介绍了可通过 NuGet 软件包管理器下载的额外库。

本章剩下的部分与堆有关。作为提醒，堆是树的另一种变体，有两个版本，最小堆和最大堆。值得注意的是，每个节点的值必须大于或等于（对于最小堆）或小于或等于（对于最大堆）其父节点的值。堆存在许多变体，包括二叉堆、二项式堆和斐波那契堆。本章简要介绍了所有这些类型，以及关于使用来自 NuGet 软件包之一的实现的信息。

让我们继续讨论下一章的主题——图！
