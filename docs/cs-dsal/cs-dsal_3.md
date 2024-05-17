# 第三章：堆栈和队列

到目前为止，您已经学到了很多关于数组和列表的知识。然而，这些结构并不是唯一可用的。除此之外，还有一组更专业的数据结构，它们被称为**有限访问数据结构**。

这意味着什么？为了解释这个名字，让我们暂时回到数组的话题，数组属于**随机访问数据结构**的一部分。它们之间的区别只有一个词，即有限或随机。正如您已经知道的那样，数组允许您存储数据并使用索引访问各种元素。因此，您可以轻松地从数组中获取第一个、中间、*n*^(th)或最后一个元素。因此，它可以被称为随机访问数据结构。

然而，*有限*是什么意思？答案非常简单——对于有限访问数据结构，您无法访问结构中的每个元素。因此，获取元素的方式是严格指定的。例如，您只能获取第一个或最后一个元素，但无法从数据结构中获取第*n*个元素。有限访问数据结构的常见代表是堆栈和队列。

在本章中，将涵盖以下主题：

+   堆栈

+   队列

+   优先队列

# 堆栈

首先，让我们谈谈**堆栈**。它是一种易于理解的数据结构，可以用许多盘子堆叠的例子来表示。您只能将新盘子添加到堆叠的顶部，并且只能从堆叠的顶部获取盘子。您无法在不从顶部取出前六个盘子的情况下移除第七个盘子，也无法在堆叠的中间添加盘子。

堆栈的操作方式与队列完全相同！它允许您在顶部添加新元素（**push**操作）并通过从顶部移除元素来获取元素（**pop**操作）。因此，堆栈符合**LIFO**原则，即**后进先出**。根据我们堆盘子的例子，最后添加的盘子（最后进）将首先从堆中移除（先出）。

堆栈的推送和弹出操作的图示如下：

![](img/a4fcda16-874b-42c5-afb0-e7012d152bcd.png)

看起来非常简单，不是吗？的确如此，您可以通过使用`System.Collections.Generic`命名空间中的内置通用`Stack`类来从堆栈的特性中受益。值得一提的是该类中的三种方法，即：

+   `Push`，在堆栈顶部插入元素

+   `Pop`，从堆栈顶部移除元素并返回

+   `Peek`，从堆栈顶部返回元素而不移除它

当然，您还可以使用其他方法，例如从堆栈中删除所有元素（`Clear`）或检查给定元素是否可用于堆栈（`Contains`）。您可以使用`Count`属性获取堆栈中的元素数量。

值得注意的是，如果容量不需要增加，`Push`方法是*O(1)*操作，否则是*O(n)*，其中*n*是堆栈中的元素数量。`Pop`和`Peek`都是*O(1)*操作。

您可以在[`msdn.microsoft.com/library/3278tedw.aspx`](https://msdn.microsoft.com/library/3278tedw.aspx)找到有关`Stack`通用类的更多信息。

现在是时候看一些例子了。让我们开始吧！

# 示例-反转单词

首先，让我们尝试使用堆栈来反转一个单词。您可以通过迭代形成字符串的字符，将每个字符添加到堆栈的顶部，然后从堆栈中移除所有元素来实现这一点。最后，您将得到反转的单词，如下图所示，它展示了反转**MARCIN**单词的过程：

![](img/a555d667-5a6c-4e2d-b410-bcede9eda7cc.png)

应添加到`Program`类中的`Main`方法的实现代码如下所示：

```cs
Stack<char> chars = new Stack<char>(); 
foreach (char c in "LET'S REVERSE!") 
{ 
    chars.Push(c); 
} 

while (chars.Count > 0) 
{ 
    Console.Write(chars.Pop()); 
} 
Console.WriteLine(); 
```

在第一行，创建了`Stack`类的一个新实例。值得一提的是，在这种情况下，堆栈只能包含`char`元素。然后，您使用`foreach`循环遍历所有字符，并通过在`Stack`实例上调用`Push`方法将每个字符插入堆栈顶部。代码的剩余部分包括`while`循环，该循环执行直到堆栈为空。使用`Count`属性来检查此条件。在每次迭代中，从堆栈中移除顶部元素（通过调用`Pop`）并在控制台中写入（使用`Console`类的`Write`静态方法）。

运行代码后，您将收到以下结果：

```cs
    !ESREVER S'TEL
```

# 示例 - 汉诺塔

下一个示例是堆栈的一个显着更复杂的应用。它与数学游戏**汉诺塔**有关。让我们从规则开始。游戏需要三根杆，您可以在上面放置圆盘。每个圆盘的大小都不同。开始时，所有圆盘都放在第一根杆上，形成堆栈，从最小的（顶部）到最大的（底部）排序如下：

![](img/48184ab8-6fd5-4a9e-bb5d-7a656bc8591d.png)

游戏的目标是将所有圆盘从第一个杆（**FROM**）移动到第二个杆（**TO**）。然而，在整个游戏过程中，您不能将较大的圆盘放在较小的圆盘上。此外，您一次只能移动一个圆盘，当然，您只能从任何杆的顶部取一个圆盘。您如何在杆之间移动圆盘以符合上述规则？问题可以分解为子问题。

让我们从只移动一个圆盘的示例开始。这种情况很简单，您只需要将一个圆盘从**FROM**杆移动到**TO**杆，而不使用**AUXILIARY**杆。

稍微复杂一点的情况是移动两个圆盘。在这种情况下，您应该将一个圆盘从**FROM**杆移动到**AUXILIARY**杆。然后，您将剩下的圆盘从**FROM**移动到**TO**。最后，您只需要将一个圆盘从**AUXILIARY**移动到**TO**。

如果要移动三个圆盘，您应该从**FROM**移动两个圆盘到**AUXILIARY**，使用前面描述的机制。操作将涉及**TO**杆作为辅助杆。然后，您将剩余的圆盘从**FROM**移动到**TO**，然后从**AUXILIARY**移动两个圆盘到**TO**，使用**FROM**作为辅助杆。

正如您所看到的，您可以通过将*n-1*个圆盘从**FROM**移动到**AUXILIARY**，使用**TO**作为辅助杆来解决移动*n*个圆盘的问题。然后，您应该将剩余的圆盘从**FROM**移动到**TO**。最后，您只需要将*n-1*个圆盘从**AUXILIARY**移动到**TO**，使用**FROM**作为辅助杆。

就是这样！现在您知道了基本规则，让我们继续进行代码。

首先，让我们专注于包含与游戏相关逻辑的`HanoiTower`类。代码的一部分如下所示：

```cs
public class HanoiTower 
{ 
    public int DiscsCount { get; private set; } 
    public int MovesCount { get; private set; } 
    public Stack<int> From { get; private set; } 
    public Stack<int> To { get; private set; } 
    public Stack<int> Auxiliary { get; private set; } 
    public event EventHandler<EventArgs> MoveCompleted; (...) 
} 
```

该类包含五个属性，存储总圆盘数（`DiscsCount`），执行的移动数（`MovesCount`）以及三个杆的表示（`From`，`To`，`Auxiliary`）。还声明了`MoveCompleted`事件。每次移动后都会触发它，以通知用户界面应该刷新。因此，您可以显示适当的内容，说明杆的当前状态。

除了属性和事件之外，该类还具有以下构造函数：

```cs
public HanoiTower(int discs) 
{ 
    DiscsCount = discs; 
    From = new Stack<int>(); 
    To = new Stack<int>(); 
    Auxiliary = new Stack<int>(); 
    for (int i = 1; i <= discs; i++) 
    { 
        int size = discs - i + 1; 
        From.Push(size); 
    } 
} 
```

构造函数只接受一个参数，即圆盘数量（`discs`），并将其设置为`DiscsCount`属性的值。然后，创建了`Stack`类的新实例，并将它们的引用存储在`From`、`To`和`Auxiliary`属性中。最后，使用`for`循环来创建必要数量的圆盘，并将元素添加到第一个堆栈（`From`）中。值得注意的是，`From`、`To`和`Auxiliary`堆栈只存储整数值（`Stack<int>`）。每个整数值表示特定圆盘的大小。由于移动圆盘的规则，这些数据是至关重要的。

通过调用`Start`方法来启动算法的操作，其代码如下所示：

```cs
public void Start() 
{ 
    Move(DiscsCount, From, To, Auxiliary); 
} 
```

该方法只是调用`Move`递归方法，将总圆盘数和三个堆栈的引用作为参数传递。但是，`Move`方法中发生了什么？让我们来看一下：

```cs
public void Move(int discs, Stack<int> from, Stack<int> to,  
    Stack<int> auxiliary) 
{ 
    if (discs > 0) 
    { 
        Move(discs - 1, from, auxiliary, to); 

        to.Push(from.Pop()); 
        MovesCount++; 
        MoveCompleted?.Invoke(this, EventArgs.Empty); 

        Move(discs - 1, auxiliary, to, from); 
    } 
} 
```

如您已经知道的，此方法是递归调用的。因此，有必要指定一些退出条件，以防止方法被无限调用。在这种情况下，当`discs`参数的值等于或小于零时，该方法将不会调用自身。如果该值大于零，则调用`Move`方法，但是堆栈的顺序会改变。然后，从由第二个参数（`from`）表示的堆栈中移除元素，并将其插入到由第三个参数（`to`）表示的堆栈的顶部。在接下来的几行中，移动次数（`MovesCount`）递增，并触发`MoveCompleted`事件。最后，再次调用`Move`方法，使用另一种杆顺序的配置。通过多次调用此方法，圆盘将从第一个（`From`）移动到第二个（`To`）杆。`Move`方法中执行的操作与在本示例的介绍中解释的在杆之间移动*n*个圆盘的问题的描述一致。

创建了关于汉诺塔游戏的逻辑的类之后，让我们看看如何创建用户界面，以便呈现算法的下一步移动。`Program`类中的必要更改如下：

```cs
private const int DISCS_COUNT = 10; 
private const int DELAY_MS = 250; 
private static int _columnSize = 30; 
```

首先，声明了两个常量，即整体圆盘数量（`DISCS_COUNT`，设置为`10`）和算法中两次移动之间的延迟（以毫秒为单位）（`DELAY_MS`，设置为`250`）。此外，声明了一个私有静态字段，表示用于表示单个杆的字符数（`_columnSize`，设置为`30`）。

`Program`类中的`Main`方法如下所示：

```cs
static void Main(string[] args) 
{ 
    _columnSize = Math.Max(6, GetDiscWidth(DISCS_COUNT) + 2); 
    HanoiTower algorithm = new HanoiTower(DISCS_COUNT); 
    algorithm.MoveCompleted += Algorithm_Visualize; 
    Algorithm_Visualize(algorithm, EventArgs.Empty); 
    algorithm.Start(); 
} 
```

首先，使用辅助的`GetDiscWidth`方法计算了单个列（表示杆）的宽度，其代码稍后将显示。然后，创建了`HanoiTower`类的新实例，并指示在触发`MoveCompleted`事件时将调用`Algorithm_Visualize`方法。接下来，调用了上述的`Algorithm_Visualize`方法来呈现游戏的初始状态。最后，调用`Start`方法来开始在杆之间移动圆盘。

`Algorithm_Visualize`方法的代码如下：

```cs
private static void Algorithm_Visualize( 
    object sender, EventArgs e) 
{ 
    Console.Clear(); 

    HanoiTowers algorithm = (HanoiTowers)sender; 
    if (algorithm.DiscsCount <= 0) 
    { 
        return; 
    } 

    char[][] visualization = InitializeVisualization(algorithm); 
    PrepareColumn(visualization, 1, algorithm.DiscsCount,  
        algorithm.From); 
    PrepareColumn(visualization, 2, algorithm.DiscsCount,  
        algorithm.To); 
    PrepareColumn(visualization, 3, algorithm.DiscsCount,  
        algorithm.Auxiliary); 

    Console.WriteLine(Center("FROM") + Center("TO") +  
        Center("AUXILIARY")); 
    DrawVisualization(visualization); 
    Console.WriteLine(); 
    Console.WriteLine($"Number of moves: {algorithm.MovesCount}"); 
    Console.WriteLine($"Number of discs: {algorithm.DiscsCount}"); 

    Thread.Sleep(DELAY_MS); 
} 
```

算法的可视化应该在控制台中呈现游戏的当前状态。因此，每当需要刷新时，`Algorithm_Visualize`方法清除控制台的当前内容（通过调用`Clear`方法）。然后，它调用`InitializeVisualization`方法来准备应该写入控制台的内容的交错数组。这样的内容包括三列，通过调用`PrepareColumn`方法准备。调用后，`visualization`数组包含应该只是呈现在控制台中的数据，没有任何额外的转换。为此，调用`DrawVisualization`方法。当然，标题和额外的解释使用`Console`类的`WriteLine`方法写入控制台。

重要的角色由代码的最后一行执行，其中调用了`System.Threading`命名空间中`Thread`类的`Sleep`方法。它暂停当前线程`DELAY_MS`毫秒。这样一行代码被添加以便以方便的方式呈现算法的以下步骤给用户。

让我们来看看`InitializeVisualization`方法的代码：

```cs
private static char[][] InitializeVisualization( 
    HanoiTowers algorithm) 
{ 
    char[][] visualization = new char[algorithm.DiscsCount][]; 

    for (int y = 0; y < visualization.Length; y++) 
    { 
        visualization[y] = new char[_columnSize * 3]; 
        for (int x = 0; x < _columnSize * 3; x++) 
        { 
            visualization[y][x] = ' '; 
        } 
    } 

    return visualization; 
} 
```

该方法声明了一个交错数组，行数等于总盘数（`DiscsCount`属性）。列数等于`_columnSize`字段的值乘以`3`（表示三根杆）。在方法内部，使用两个`for`循环来迭代遍历行（第一个`for`循环）和所有列（第二个`for`循环）。默认情况下，数组中的所有元素都被初始化为单个空格。最后，初始化的数组被返回。

要用当前杆的状态的插图填充上述的交错数组，需要调用`PrepareColumn`方法，其代码如下：

```cs
private static void PrepareColumn(char[][] visualization,  
    int column, int discsCount, Stack<int> stack) 
{ 
    int margin = _columnSize * (column - 1); 
    for (int y = 0; y < stack.Count; y++) 
    { 
        int size = stack.ElementAt(y); 
        int row = discsCount - (stack.Count - y); 
        int columnStart = margin + discsCount - size; 
        int columnEnd = columnStart + GetDiscWidth(size); 
        for (int x = columnStart; x <= columnEnd; x++) 
        { 
            visualization[row][x] = '='; 
        } 
    } 
} 
```

首先，计算左边距以在整体数组中的正确部分添加数据，即在正确的列范围内。然而，方法的主要部分是`for`循环，其中迭代次数等于给定堆栈中的盘数。在每次迭代中，使用`ElementAt`扩展方法（来自`System.Linq`命名空间）读取当前盘的大小。接下来，计算应该显示盘的行的索引，以及列的起始和结束索引。最后，使用`for`循环将等号（`=`）插入到作为`visualization`参数传递的交错数组的适当位置。

下一个与可视化相关的方法是`DrawVisualization`，其代码如下：

```cs
private static void DrawVisualization(char[][] visualization) 
{ 
    for (int y = 0; y < visualization.Length; y++) 
    { 
        Console.WriteLine(visualization[y]); 
    } 
} 
```

该方法只是遍历作为`visualization`参数传递的交错数组的所有元素，并为交错数组中的每个数组调用`WriteLine`方法。结果是，整个数组中的数据被写入控制台。

其中一个辅助方法是`Center`。它的目的是在参数中传递的文本之前和之后添加额外的空格，以使文本在列中居中。该方法的代码如下：

```cs
private static string Center(string text) 
{ 
    int margin = (_columnSize - text.Length) / 2; 
    return text.PadLeft(margin + text.Length) 
        .PadRight(_columnSize); 
} 
```

另一个方法是`GetDiscWidth`，它只返回以参数指定大小呈现的盘所需的字符数。其代码如下：

```cs
private static int GetDiscWidth(int size) 
{ 
    return 2 * size - 1; 
} 
```

您已经添加了运行应用程序所需的代码，该应用程序将呈现汉诺塔数学游戏的以下移动。让我们启动应用程序并看看它的运行情况！

在程序启动后，您将看到类似以下的结果，其中所有盘都位于第一根杆（`FROM`）中：

```cs
            FROM                  TO                AUXILIARY
             ==
            ====
           ======
          ========
         ==========
        ============
       ==============
      ================
     ==================
    ====================

```

在下一步中，最小的盘从第一根杆（`FROM`）的顶部移动到第三根杆（`AUXILIARY`）的顶部，如下图所示：

```cs
            FROM                  TO                AUXILIARY    

            ====
           ======
          ========
         ==========
        ============
       ==============
      ================
     ==================
    ====================                               ==

```

在进行许多其他移动时，您可以看到盘在三根杆之间移动。其中一个中间状态如下：

```cs
            FROM                  TO                AUXILIARY          

            ====
         ==========
        ============
       ==============
      ================
     ==================         ======
    ====================       ========                ==

```

当完成必要的移动后，所有圆盘都从第一个圆盘（`FROM`）移动到第二个圆盘（`TO`）。最终结果如下图所示：

```cs
            FROM                  TO                AUXILIARY
                                  ==
                                 ====
                                ======
                               ========
                              ==========
                             ============
                            ==============
                           ================
                          ==================
                         ====================

```

最后，值得一提的是完成汉诺塔游戏所需的移动次数。在 10 个圆盘的情况下，移动次数为 1,023。如果只使用三个圆盘，移动次数只有七次。一般来说，可以用公式*2^n-1*来计算移动次数，其中*n*是圆盘的数量。

就这些了！在本节中，您已经学习了第一个有限访问数据结构，即栈。现在，是时候更多地了解队列了。让我们开始吧！

# 队列

**队列**是一种数据结构，可以用在商店结账时等待的人排队的例子中。新人站在队伍的末尾，下一个人从队伍的开头被带到结账处。不允许您从中间选择一个人并按不同的顺序为他或她服务。

队列数据结构的操作方式完全相同。您只能在队列的末尾添加新元素（**enqueue**操作），并且只能从队列的开头删除一个元素（**dequeue**操作）。因此，这种数据结构符合**FIFO**原则，即**先进先出**。在商店结账时等待的人排队的例子中，先来的人（先进）将在后来的人之前（先出）被服务。

队列的操作如下图所示：

![](img/3f5f553b-30ad-467b-bc7e-54e3f368fd89.png)

值得一提的是，队列是一个**递归数据结构**，与栈类似。这意味着队列可以是空的，也可以由第一个元素和其余队列组成，后者也形成一个队列，如下图所示（队列的开始标记为灰色）：

![](img/64556717-48c1-4468-b84a-4b91eec6cce8.png)

队列数据结构似乎很容易理解，与栈类似，除了删除元素的方式。这是否意味着您也可以在程序中使用内置类来使用队列？幸运的是，可以！可用的通用类名为`Queue`，定义在`System.Collections.Generic`命名空间中。

`Queue`类包含一组方法，例如：

+   `Enqueue`，在队列末尾添加一个元素

+   `Dequeue`，从开头删除一个元素并返回它

+   `Peek`，从开头返回一个元素而不删除它

+   `Clear`，从队列中删除所有元素

+   `Contains`，检查队列是否包含给定元素

`Queue`类还包含`Count`属性，返回队列中的元素总数。它可以用于轻松检查队列是否为空。

值得一提的是，如果内部数组不需要重新分配，则`Enqueue`方法是*O(1)*操作，否则为*O(n)*，其中*n*是队列中的元素数量。`Dequeue`和`Peek`都是*O(1)*操作。

您可以在[`msdn.microsoft.com/library/7977ey2c.aspx`](https://msdn.microsoft.com/library/7977ey2c.aspx)找到有关`Queue`类的更多信息。

在想要从多个线程同时使用队列的情况下，需要额外的注释。在这种情况下，需要选择线程安全的队列变体，即`System.Collections.Concurrent`命名空间中的`ConcurrentQueue`通用类。该类包含一组内置方法，用于执行线程安全队列的各种操作，例如：

+   `Enqueue`，在队列末尾添加一个元素

+   `TryDequeue`，尝试从开头删除一个元素并返回它

+   `TryPeek`，尝试从开头返回一个元素而不删除它

值得一提的是，`TryDequeue`和`TryPeek`都有一个带有`out`关键字的参数。如果操作成功，这些方法将返回`true`，并将结果作为`out`参数的值返回。此外，`ConcurrentQueue`类还包含两个属性，即`Count`用于获取集合中存储的元素数量，以及`IsEmpty`用于返回一个值，指示队列是否为空。

您可以在[`msdn.microsoft.com/library/dd267265.aspx`](https://msdn.microsoft.com/library/dd267265.aspx)找到有关`ConcurrentQueue`类的更多信息。

在这个简短的介绍之后，您应该准备好继续进行两个示例，代表呼叫中心中的队列，有许多呼叫者和一个或多个顾问。

# 示例 - 仅有一个顾问的呼叫中心

这个第一个示例代表了呼叫中心解决方案的简单方法，其中有许多呼叫者（具有不同的客户标识符），以及只有一个顾问，他按照呼叫出现的顺序接听等待的电话。这种情况在下图中呈现：

![](img/ac923287-81b8-49c7-87d8-cff86e481094.png)

正如您在前面的图表中所看到的，呼叫者执行了四次呼叫。它们被添加到等待电话呼叫的队列中，即来自客户**#1234**，**#5678**，**#1468**和**#9641**。当顾问可用时，他或她会接听电话。通话结束后，顾问可以接听下一个等待的电话。根据这个规则，顾问将按照以下顺序与客户交谈：**#1234**，**#5678**，**#1468**和**#9641**。

让我们来看一下第一个类`IncomingCall`的代码，它代表了呼叫中心中由呼叫者执行的单个呼入呼叫。其代码如下：

```cs
public class IncomingCall 
{ 
    public int Id { get; set; } 
    public int ClientId { get; set; } 
    public DateTime CallTime { get; set; } 
    public DateTime StartTime { get; set; } 
    public DateTime EndTime { get; set; } 
    public string Consultant { get; set; } 
} 
```

该类包含六个属性，代表呼叫的标识符（`Id`），客户标识符（`ClientId`），呼叫开始的日期和时间（`CallTime`），呼叫被接听的日期和时间（`StartTime`），呼叫结束的日期和时间（`EndTime`），以及顾问的姓名（`Consultant`）。

这个实现中最重要的部分与`CallCenter`类相关，它代表了与呼叫相关的操作。其片段如下：

```cs
public class CallCenter 
{ 
    private int _counter = 0; 
    public Queue<IncomingCall> Calls { get; private set; } 

    public CallCenter() 
    { 
        Calls = new Queue<IncomingCall>(); 
    } 
} 
```

`CallCenter`类包含`_counter`字段，其中包含最后一次呼叫的标识符，以及`Calls`队列（带有`IncomingCall`实例），其中存储了等待呼叫的数据。在构造函数中，创建了`Queue`泛型类的新实例，并将其引用分配给`Calls`属性。

当然，该类还包含一些方法，比如`Call`，代码如下：

```cs
public void Call(int clientId) 
{ 
    IncomingCall call = new IncomingCall() 
    { 
        Id = ++_counter, 
        ClientId = clientId, 
        CallTime = DateTime.Now 
    }; 
    Calls.Enqueue(call); 
} 
```

在这里，您创建了`IncomingCall`类的新实例，并设置了其属性的值，即其标识符（连同预增量`_counter`字段）、客户标识符（使用`clientId`参数）和呼叫时间。通过调用`Enqueue`方法，将创建的实例添加到队列中。

下一个方法是`Answer`，它代表了回答呼叫的操作，来自队列中等待时间最长的人，也就是位于队列开头的人。`Answer`方法如下所示：

```cs
public IncomingCall Answer(string consultant) 
{ 
    if (Calls.Count > 0) 
    { 
        IncomingCall call = Calls.Dequeue(); 
        call.Consultant = consultant; 
        call.StartTime = DateTime.Now; 
        return call; 
    } 
    return null; 
} 
```

在这个方法中，您检查队列是否为空。如果是，该方法返回`null`，这意味着顾问没有可以接听的电话。否则，呼叫将从队列中移除（使用`Dequeue`方法），并通过设置顾问姓名（使用`consultant`参数）和开始时间（为当前日期和时间）来更新其属性。最后，返回呼叫的数据。

除了`Call`和`Answer`方法，您还应该实现`End`方法，每当顾问结束与特定客户的通话时都会调用该方法。在这种情况下，您只需设置结束时间，如下面的代码片段所示：

```cs
public void End(IncomingCall call) 
{ 
    call.EndTime = DateTime.Now; 
} 
```

`CallCenter`类中的最后一个方法名为`AreWaitingCalls`。它使用`Queue`类的`Count`属性返回一个值，指示队列中是否有任何等待的呼叫。其代码如下：

```cs
public bool AreWaitingCalls() 
{ 
    return Calls.Count > 0; 
} 
```

让我们继续到`Program`类和它的`Main`方法：

```cs
static void Main(string[] args) 
{ 
    Random random = new Random(); 

    CallCenter center = new CallCenter(); 
    center.Call(1234); 
    center.Call(5678); 
    center.Call(1468); 
    center.Call(9641); 

    while (center.AreWaitingCalls()) 
    { 
        IncomingCall call = center.Answer("Marcin"); 
        Log($"Call #{call.Id} from {call.ClientId}  
            is answered by {call.Consultant}."); 
        Thread.Sleep(random.Next(1000, 10000)); 
        center.End(call); 
        Log($"Call #{call.Id} from {call.ClientId}  
            is ended by {call.Consultant}."); 
    } 
} 
```

在这里，你创建了`Random`类的一个新实例（用于获取随机数），以及`CallCenter`类的一个实例。然后，你通过呼叫者模拟了一些呼叫，即使用以下客户标识符：`1234`，`5678`，`1468`和`9641`。代码中最有趣的部分位于`while`循环中，该循环执行直到队列中没有等待的呼叫为止。在循环内，顾问接听呼叫（使用`Answer`方法），并生成日志（使用`Log`辅助方法）。然后，线程暂停一段随机毫秒数（在 1,000 到 10,000 之间）以模拟呼叫的不同长度。当时间到达后，呼叫结束（通过调用`End`方法），并生成适当的日志。

这个示例中必要的最后一部分代码是`Log`方法：

```cs
private static void Log(string text) 
{ 
    Console.WriteLine($"[{DateTime.Now.ToString("HH:mm:ss")}]  
        {text}"); 
} 
```

当你运行这个示例时，你会收到类似以下的结果：

```cs
    [15:24:36] Call #1 from 1234 is answered by Marcin.
    [15:24:40] Call #1 from 1234 is ended by Marcin.
    [15:24:40] Call #2 from 5678 is answered by Marcin.
    [15:24:48] Call #2 from 5678 is ended by Marcin.
    [15:24:48] Call #3 from 1468 is answered by Marcin.
    [15:24:53] Call #3 from 1468 is ended by Marcin.
    [15:24:53] Call #4 from 9641 is answered by Marcin.
    [15:24:57] Call #4 from 9641 is ended by Marcin.

```

就是这样！你刚刚完成了关于队列数据结构的第一个示例。如果你想了解更多关于队列的线程安全版本，让我们继续到下一部分，看看下一个示例。

# 示例 - 带有多个顾问的呼叫中心

在前面的部分中显示的示例被故意简化，以使理解队列变得更简单。然而，现在是时候让它更相关于现实世界的问题了。在这一部分，你将看到如何扩展它以支持多个顾问，如下图所示：

![](img/40966381-d05b-420f-bf0e-78608de68883.png)

重要的是，呼叫者和顾问将同时工作。如果有更多的呼叫比可用的顾问多，新的呼叫将被添加到队列中，并等待直到有顾问可以接听呼叫。如果顾问过多而呼叫过少，顾问将等待呼叫。为了执行这个任务，你需要创建一些线程，它们将访问队列。因此，你需要使用`ConcurrentQueue`类的线程安全版本。

让我们看一下代码！首先，你需要声明`IncomingCall`类，其代码与前面的示例完全相同：

```cs
public class IncomingCall 
{ 
    public int Id { get; set; } 
    public int ClientId { get; set; } 
    public DateTime CallTime { get; set; } 
    public DateTime StartTime { get; set; } 
    public DateTime EndTime { get; set; } 
    public string Consultant { get; set; } 
} 
```

`CallCenter`类中需要进行各种修改，比如用`ConcurrentQueue`泛型类的实例替换`Queue`类的实例。适当的代码片段如下所示：

```cs
public class CallCenter 
{ 
    private int _counter = 0; 
    public ConcurrentQueue<IncomingCall> Calls  
        { get; private set; } 

    public CallCenter() 
    { 
        Calls = new ConcurrentQueue<IncomingCall>(); 
    } 
} 
```

由于`Enqueue`方法在`Queue`和`ConcurrentQueue`类中都可用，所以在`Call`方法的最重要部分不需要进行任何修改。然而，在将新呼叫添加到队列后，引入了一个小的修改来返回等待呼叫的数量。修改后的代码如下：

```cs
public int Call(int clientId) 
{ 
    IncomingCall call = new IncomingCall() 
    { 
        Id = ++_counter, 
        ClientId = clientId, 
        CallTime = DateTime.Now 
    }; 
    Calls.Enqueue(call); 
    return Calls.Count; 
} 
```

`ConcurrentQueue`类中不存在`Dequeue`方法。因此，你需要稍微修改`Answer`方法，使用`TryDequeue`方法，该方法返回一个值，指示元素是否已从队列中移除。移除的元素使用`out`参数返回。适当的代码部分如下：

```cs
public IncomingCall Answer(string consultant) 
{ 
    if (Calls.Count > 0  
        && Calls.TryDequeue(out IncomingCall call)) 
    { 
        call.Consultant = consultant; 
        call.StartTime = DateTime.Now; 
        return call; 
    } 
    return null; 
} 
```

在`CallCenter`类中声明的剩余方法`End`和`AreWaitingCalls`中不需要进行进一步的修改。它们的代码如下：

```cs
public void End(IncomingCall call) 
{ 
    call.EndTime = DateTime.Now; 
}

public bool AreWaitingCalls() 
{ 
    return Calls.Count > 0; 
}
```

在`Program`类中需要进行更多的修改。在这里，你需要启动四个线程。第一个代表呼叫者，而其他三个代表顾问。首先，让我们看一下`Main`方法的代码：

```cs
static void Main(string[] args) 
{ 
    CallCenter center = new CallCenter(); 
    Parallel.Invoke( 
        () => CallersAction(center), 
        () => ConsultantAction(center, "Marcin",  
                  ConsoleColor.Red), 
        () => ConsultantAction(center, "James",  
                  ConsoleColor.Yellow), 
        () => ConsultantAction(center, "Olivia",  
                  ConsoleColor.Green)); 
} 
```

在这里，在创建`CallCenter`实例后，您使用`System.Threading.Tasks`命名空间中`Parallel`类的`Invoke`静态方法开始执行四个操作，即代表呼叫者和三个咨询师，使用 lambda 表达式来指定将被调用的方法，即呼叫者相关操作的`CallersAction`和咨询师相关任务的`ConsultantAction`。您还可以指定其他参数，比如给定咨询师的名称和颜色。

`CallersAction` 方法代表了许多呼叫者循环执行的操作。其代码如下所示：

```cs
private static void CallersAction(CallCenter center) 
{ 
    Random random = new Random(); 
    while (true) 
    { 
        int clientId = random.Next(1, 10000); 
        int waitingCount = center.Call(clientId); 
        Log($"Incoming call from {clientId},  
            waiting in the queue: {waitingCount}"); 
        Thread.Sleep(random.Next(1000, 5000)); 
    } 
}
```

代码中最重要的部分是无限执行的`while`循环。在其中，您会得到一个随机数作为客户的标识符（`clientId`），并调用`Call`方法。等待呼叫的数量被记录下来，连同客户标识符。最后，呼叫者相关的线程将暂停一段随机毫秒数，范围在 1,000 毫秒到 5,000 毫秒之间，即 1 到 5 秒之间，以模拟呼叫者进行另一个呼叫之间的延迟。

下一个方法名为`ConsultantAction`，并在每个咨询师的单独线程上执行。该方法接受三个参数，即`CallCenter`类的一个实例，以及咨询师的名称和颜色。代码如下：

```cs
private static void ConsultantAction(CallCenter center,  
    string name, ConsoleColor color) 
{ 
    Random random = new Random(); 
    while (true) 
    { 
        IncomingCall call = center.Answer(name); 
        if (call != null) 
        { 
            Console.ForegroundColor = color; 
            Log($"Call #{call.Id} from {call.ClientId} is answered  
                by {call.Consultant}."); 
            Console.ForegroundColor = ConsoleColor.Gray; 

            Thread.Sleep(random.Next(1000, 10000)); 
            center.End(call); 

            Console.ForegroundColor = color; 
            Log($"Call #{call.Id} from {call.ClientId}  
                is ended by {call.Consultant}."); 
            Console.ForegroundColor = ConsoleColor.Gray; 

            Thread.Sleep(random.Next(500, 1000)); 
        } 
        else 
        { 
            Thread.Sleep(100); 
        } 
    } 
} 
```

与`CallersAction`方法类似，最重要和有趣的操作是在无限的`while`循环中执行的。在其中，咨询师尝试使用`Answer`方法回答第一个等待的呼叫。如果没有等待的呼叫，线程将暂停 100 毫秒。否则，根据当前咨询师的情况，以适当的颜色呈现日志。然后，线程将暂停 1 到 10 秒之间的随机时间。在此时间之后，咨询师结束呼叫，通过调用`End`方法来指示，并生成日志。最后，线程将暂停 500 毫秒到 1,000 毫秒之间的随机时间，这代表了呼叫结束和另一个呼叫开始之间的延迟。

最后一个辅助方法名为`Log`，与前一个示例中的方法完全相同。其代码如下：

```cs
private static void Log(string text) 
{ 
    Console.WriteLine($"[{DateTime.Now.ToString("HH:mm:ss")}]  
        {text}"); 
} 
```

当您运行程序并等待一段时间后，您将收到类似于以下截图所示的结果：

![](img/0a800625-5b63-44b0-a245-6919a7f84ff7.png)

恭喜！您刚刚完成了两个示例，代表了呼叫中心场景中队列的应用。

修改程序的各种参数是一个好主意，比如咨询师的数量，以及延迟时间，特别是呼叫者之间的延迟时间。然后，您将看到算法在呼叫者或咨询师过多的情况下是如何工作的。

然而，如何处理具有优先支持的客户呢？在当前解决方案中，他们将与标准支持计划的客户一起等待在同一个队列中。您需要创建两个队列并首先从优先队列中取客户吗？如果是这样，如果您引入另一个支持计划会发生什么？您需要添加另一个队列并在代码中引入这样的修改吗？幸运的是，不需要！您可以使用另一种数据结构，即优先队列，来支持这样的情景，如下一节中详细解释的那样。

# 优先队列

**优先级队列**使得可以通过为队列中的每个元素设置**优先级**来扩展队列的概念。值得一提的是，优先级可以简单地指定为整数值。然而，较小或较大的值是否表示更高的优先级取决于实现。在本章中，假设最高优先级等于 0，而较低的优先级由 1、2、3 等指定。因此，**出队**操作将返回具有最高优先级的元素，该元素首先添加到队列中，如下图所示：

![](img/fc121dc6-2124-4ec7-8ba1-033cd63f35e9.png)

让我们分析一下图表。首先，优先级队列包含两个具有相同优先级（等于**1**）的元素，即**Marcin**和**Lily**。然后，添加了具有更高优先级（**0**）的**Mary**元素，这意味着该元素位于队列的开头，即在**Marcin**之前。在下一步中，具有最低优先级（**2**）的**John**元素被添加到优先级队列的末尾。第三列显示了具有优先级等于**1**的**Emily**元素的添加，即与**Marcin**和**Lily**相同。因此，**Emily**元素在**Lily**之后添加。根据前述规则，您添加以下元素，即优先级设置为**0**的**Sarah**和优先级等于**1**的**Luke**。最终顺序显示在前述图表的右侧。

当然，可以自己实现优先级队列。但是，您可以通过使用其中一个可用的 NuGet 包，即`OptimizedPriorityQueue`来简化此任务。有关此包的更多信息，请访问[`www.nuget.org/packages/OptimizedPriorityQueue`](https://www.nuget.org/packages/OptimizedPriorityQueue)。

您知道如何将此包添加到您的项目中吗？如果不知道，您应该按照以下步骤进行：

1.  从解决方案资源管理器窗口中的项目节点的上下文菜单中选择管理 NuGet 包。

1.  选择打开窗口中的浏览选项卡。

1.  在搜索框中键入`OptimizedPriorityQueue`。

1.  单击 OptimizedPriorityQueue 项目。

1.  在右侧单击安装按钮。

1.  在预览更改窗口中单击确定。

1.  等待直到在输出窗口中显示完成消息。

`OptimizedPriorityQueue`库显着简化了在各种应用程序中应用优先级队列。其中，可用`SimplePriorityQueue`泛型类，其中包含一些有用的方法，例如：

+   `Enqueue`，向优先级队列中添加元素

+   `Dequeue`，从开头删除元素并返回它

+   `GetPriority`，返回元素的优先级

+   `UpdatePriority`，更新元素的优先级

+   `Contains`，检查优先级队列中是否存在元素

+   `Clear`，从优先级队列中删除所有元素

您可以使用`Count`属性获取队列中元素的数量。如果要从优先级队列的开头获取元素而不将其删除，可以使用`First`属性。此外，该类包含一组方法，这些方法在多线程场景中可能很有用，例如`TryDequeue`和`TryRemove`。值得一提的是，`Enqueue`和`Dequeue`方法都是*O(log n)*操作。

在对优先级队列的主题进行了简短介绍之后，让我们继续介绍具有优先级支持的呼叫中心的示例，该示例在以下部分中进行了描述。

# 示例 - 具有优先级支持的呼叫中心

作为优先级队列的示例，让我们介绍一种简单的方法，即呼叫中心示例，其中有许多呼叫者（具有不同的客户标识符），并且只有一个顾问，他首先从优先级队列中回答等待的呼叫，然后从具有标准支持计划的客户那里回答。

上述情景在以下图表中呈现。标有**-**的是标准优先级的呼叫，而标有*****的是优先级支持的呼叫，如下所示：

![](img/207dd920-227c-43c6-aa0d-851366641327.png)

让我们来看看优先级队列中元素的顺序。目前，它只包含三个元素，将按以下顺序提供服务：**#5678**（具有优先级支持），**#1234**和**#1468**。然而，来自标识符**#9641**的客户的呼叫导致顺序变为**#5678**，**#9641**（由于优先级支持），**#1234**和**#1468**。

是时候写一些代码了！首先，不要忘记将`OptimizedPriorityQueue`包添加到项目中，如前所述。当库配置正确时，您可以继续实现`IncomingCall`类：

```cs
public class IncomingCall 
{ 
    public int Id { get; set; } 
    public int ClientId { get; set; } 
    public DateTime CallTime { get; set; } 
    public DateTime StartTime { get; set; } 
    public DateTime EndTime { get; set; } 
    public string Consultant { get; set; } 
    public bool IsPriority { get; set; } 
} 
```

在这里，与之前呈现的简单呼叫中心应用程序的情景相比，只有一个变化，即添加了`IsPriority`属性。它指示当前呼叫是否具有优先级支持（`true`）或标准支持（`false`）。

`CallCenter`类中也需要进行一些修改，其片段如下代码片段所示：

```cs
public class CallCenter 
{ 
    private int _counter = 0; 
    public SimplePriorityQueue<IncomingCall> Calls  
        { get; private set; } 

    public CallCenter() 
    { 
        Calls = new SimplePriorityQueue<IncomingCall>(); 
    } 
} 
```

如您所见，`Calls`属性的类型已从`Queue`更改为`SimplePriorityQueue`泛型类。在`Call`方法中需要进行以下更改，代码如下所示：

```cs
public void Call(int clientId, bool isPriority = false) 
{ 
    IncomingCall call = new IncomingCall() 
    { 
        Id = ++_counter, 
        ClientId = clientId, 
        CallTime = DateTime.Now, 
        IsPriority = isPriority 
    }; 
    Calls.Enqueue(call, isPriority ? 0 : 1); 
} 
```

在这个方法中，使用参数设置了`IsPriority`属性（前面提到的）。此外，在调用`Enqueue`方法时，使用了两个参数，不仅是元素的值（`IncomingCall`类的实例），还有一个优先级的整数值，即在优先级支持的情况下为`0`，否则为`1`。

在`CallCenter`类的方法中不需要进行更多的修改，即`Answer`，`End`和`AreWaitingCalls`方法。相关代码如下：

```cs
public IncomingCall Answer(string consultant) 
{ 
    if (Calls.Count > 0) 
    { 
        IncomingCall call = Calls.Dequeue(); 
        call.Consultant = consultant; 
        call.StartTime = DateTime.Now; 
        return call; 
    } 
    return null; 
}

public void End(IncomingCall call) 
{ 
    call.EndTime = DateTime.Now; 
}

public bool AreWaitingCalls() 
{ 
    return Calls.Count > 0; 
} 
```

最后，让我们来看看`Program`类中`Main`和`Log`方法的代码：

```cs
static void Main(string[] args) 
{ 
    Random random = new Random(); 

    CallCenter center = new CallCenter(); 
    center.Call(1234); 
    center.Call(5678, true); 
    center.Call(1468); 
    center.Call(9641, true); 

    while (center.AreWaitingCalls()) 
    { 
        IncomingCall call = center.Answer("Marcin"); 
        Log($"Call #{call.Id} from {call.ClientId}  
            is answered by {call.Consultant} /  
            Mode: {(call.IsPriority ? "priority" : "normal")}."); 
        Thread.Sleep(random.Next(1000, 10000)); 
        center.End(call); 
        Log($"Call #{call.Id} from {call.ClientId}  
            is ended by {call.Consultant}."); 
    } 
} 
private static void Log(string text) 
{ 
    Console.WriteLine($"[{DateTime.Now.ToString("HH:mm:ss")}]  
        {text}"); 
} 
```

您可能会惊讶地发现，在代码的这一部分只需要进行两个更改！原因是使用的数据结构的逻辑被隐藏在`CallCenter`类中。在`Program`类中，您调用了一些方法并使用了`CallCenter`类公开的属性。您只需要修改向队列添加呼叫的方式，并调整呼叫被顾问接听时呈现的日志，以展示呼叫的优先级。就是这样！

运行应用程序时，您将收到类似以下的结果：

```cs
    [15:40:26] Call #2 from 5678 is answered by Marcin / Mode:    
 **priority**.
    [15:40:35] Call #2 from 5678 is ended by Marcin.
    [15:40:35] Call #4 from 9641 is answered by Marcin / Mode: 
 **priority**.
    [15:40:39] Call #4 from 9641 is ended by Marcin.
    [15:40:39] Call #1 from 1234 is answered by Marcin / Mode: **normal**.
    [15:40:48] Call #1 from 1234 is ended by Marcin.
    [15:40:48] Call #3 from 1468 is answered by Marcin / Mode: **normal**.
    [15:40:57] Call #3 from 1468 is ended by Marcin.

```

如您所见，呼叫按正确的顺序提供服务。这意味着具有优先级支持的客户的呼叫比具有标准支持计划的客户的呼叫更早得到服务，尽管这类呼叫需要等待更长时间才能得到答复。

# 总结

在本章中，您已经了解了三种有限访问数据结构，即栈、队列和优先级队列。值得记住的是，这些数据结构都有严格指定的访问元素的方式。它们都有各种各样的现实世界应用，本书中已经提到并描述了其中一些。

首先，您看到了栈如何按照 LIFO 原则运作。在这种情况下，您只能在栈的顶部添加元素（推送操作），并且只能从顶部移除元素（弹出操作）。栈已在两个示例中展示，即用于颠倒一个单词和解决汉诺塔数学游戏。

在本章的后续部分，您了解了队列作为一种数据结构，它根据 FIFO 原则运作。在这种情况下，介绍了入队和出队操作。队列已通过两个示例进行了解释，都涉及模拟呼叫中心的应用程序。此外，您还学会了如何运行几个线程，以及如何在 C#语言开发应用程序时使用线程安全的队列变体。

本章介绍的第三种数据结构称为优先队列，是队列的扩展，支持特定元素的优先级。为了更容易地使用这种数据结构，您已经学会了如何使用外部 NuGet 包。例如，呼叫中心场景已扩展为处理两种支持计划。

这只是本书的第三章，您已经学到了很多关于各种数据结构和算法的知识，这些知识在 C#应用程序开发中非常有用！您是否有兴趣通过学习字典和集合来增加您的知识？如果是的话，让我们继续下一章，了解更多关于这些数据结构的知识！
