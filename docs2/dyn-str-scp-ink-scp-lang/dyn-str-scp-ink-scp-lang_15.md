# 第十二章：*第十二章*：使用 ink 进行程序化叙事

在本章中，我们将使用墨水和 Unity 回顾**程序化叙事**。Inkle 公司创建了并维护了叙事脚本语言 ink，并发布了多个结合 ink 和 Unity 的游戏。这些游戏使用程序化叙事，根据随机性和玩家的选择，为每个会话提供不同的体验。本章将介绍实现相同一般结果的方法：在 ink 中加载值和在 Unity 中编码集合。

在第一个主题中，我们将更广泛地回顾术语*程序化叙事*。基于最初在*第三章*中介绍的概念，“序列、循环和文本洗牌”，我们将学习如何使用墨水中的洗牌功能，根据简单的规则创建动态内容。

深入 ink，第二个主题将演示如何在 ink 中加载值。这个过程侧重于使用 ink 根据简单规则为玩家生成内容。

在最后一个主题中，我们将重点从 ink 转移到 Unity。我们将使用 Unity 来加载值并在 ink 中调用函数来处理值。这种方法在 Unity 中使用更复杂的代码，但在 ink 中使用更简单的代码。

在本章中，我们将涵盖以下主题：

+   在墨水（ink）中引入程序化叙事

+   将值加载到 ink 中

+   在 Unity 中编码集合

# 技术要求

本章中的示例可以在 GitHub 上找到：[`github.com/PacktPublishing/Dynamic-Story-Scripting-with-the-ink-Scripting-Language/tree/main/Chapter12`](https://github.com/PacktPublishing/Dynamic-Story-Scripting-with-the-ink-Scripting-Language/tree/main/Chapter12)。

# 在墨水（ink）中引入程序化叙事

术语*程序化叙事*的名字来源于另一个术语，*程序化生成*。在“生成”之前，“程序化”一词意味着内容是根据一系列*程序*（即规则）创建的。也就是说，规则。当提到生成资产，如 3D 模型或在某些游戏世界中的非玩家角色时，术语*程序化生成*适用。当讨论与交互项目中玩家的故事或体验叙事相关的规划、生成或动态排序内容时，更好的术语是**程序化叙事**。

当一个项目使用规则来定义玩家如何与其故事内容的部分交互或遭遇时，就会发生程序化叙事。例如，如果一个项目有一套规则来为其角色创建动态名称，那么在科幻设定中的玩家可能会与生成的角色名称*Neldronor*互动，而另一个玩家可能会看到同一实体的名称*Vynear*。程序化生成故事内容也可以扩展到替换名称之外，决定玩家可以访问哪些任务，他们在遇到某些角色时，甚至在他们游戏会话中可能发生的事件。

在这个主题中，我们将介绍三个基于随机性的简单模式，以了解墨水中的程序生成。第一个，*随机遭遇*，将解释如何结合使用洗牌和墨水中的线程来创建玩家的可能遭遇列表。第二个模式，*加权随机性*，使用与第一个模式相同的概念，但定义了玩家可能看到某些内容的加权概率。这两种模式都是向现有项目添加简单程序生成而不破坏其现有结构的简单方法。最后一部分和模式，*条件内容*，将涵盖使用之前的行为和玩家的输入来影响玩家看到的遭遇。它是通过使用前两种模式的概念来做到这一点的。

## 随机遭遇

许多桌面角色扮演游戏使用一个称为**随机表格**的概念。在游戏的材料或书中，有一个表格或列表，列出了玩家可能在某个地点或场景中遇到的可能事物。游戏主持人会掷一些骰子，查阅表格以找到与掷出的数字匹配的行，然后告诉玩家他们遇到了什么。这个系统创造了**随机**遭遇的可能性。基于骰子的随机元素，玩家每次使用相同的表格生成游戏内容时，都会看到或与不同的事物互动。

在数字环境中，桌面游戏的随机表格可以成为一组可能的遭遇。在墨水中，我们可以使用洗牌和线程来创建这个：

```cs
{shuffle:
    - <- encounter.animal
    - <- encounter.machine
    - <- encounter.person
}
```

因为洗牌总是会随机选择其条目之一，所以每个线程发生的概率相同。然而，可能并不那么明显，**每个可能的遭遇都是额外内容**。与没有程序化叙事功能的、由作者大量创作的体验不同，即使是添加一个可能的遭遇表格这样的简单操作，也意味着为每个可能的遭遇创建新内容：

示例 1：

```cs
{shuffle:
    - <- encounter.animal
    - <- encounter.machine
    - <- encounter.person
}
== encounter
= animal
You hear a soft thud and then see a face peering at you. The sound starts as if it is a meow and then turns into language the longer you listen. "Meee-Hello. Sorry. I'm not used to talking to people.
-> DONE
= machine
The small machine buzzes to life in front of you. "Hi, there! I'm Ge8at10, but you can call me 'Great!'"
-> DONE
= person
You look around and see a man standing awkwardly against a tree. He waves and then looks away before speaking. "Uh. Yeah. Over here. Hi."
-> DONE
```

*示例 1* 展示了使用墨水中的洗牌，包含三个不同的线程，作为一个简单的随机遭遇系统。每个线程都引用一个名为 `encounter` 的结点，其针线以遭遇的类型命名，例如 `animal`。基于随机选择，玩家将看到三种可能遭遇中的一种。

使用随机遭遇，如定义在洗牌中，是向项目中添加简单程序化叙事的一种简单方法。这确实需要为洗牌中的每个条目添加额外内容，但这种模式对现有项目的影响最小。也有可能拥有额外的集合，并创建更动态的结果，玩家可能在多个设置、情境或级别中遇到任何数量的事物，每个使用都基于洗牌和墨水中的线程。

在下一节中，我们将检查加权概率作为控制一系列遭遇随机性的一个部分。与使用洗牌时所有条目都有相同发生概率的情况不同，可能存在某些遭遇应该更频繁地发生的情况。

## 加权随机性

洗牌是 ink 中强大的功能。它允许我们创建一组可能的条目，然后随机选择一个。正如在*随机遭遇*部分所看到的，可以创建一个简单的可能遭遇集；然后，可以使用洗牌按需选择一个。

然而，可能存在不需要等加权概率的情况。例如，开发者可能只想让玩家在游戏中穿越森林区域时 30%的时间遇到一个角色。对于这些情况，我们希望对遭遇的随机性进行*加权*。

在 ink 中，`RANDOM()`函数允许我们定义产生的随机整数的范围。如果我们想要`1`到`10`之间的数字，我们可以使用`RANDOM(1,10)`。根据`RANDOM()`函数返回的数字，可以测试值，并且只有在结果在某个范围内时才采取行动：

示例 2：

```cs
VAR percentage = 0
~ percentage = RANDOM(1,10)
{
    - percentage <= 3: 
        <- encounter.brown_wizard
    - else:
        <- encounter.travel
}
== encounter
= brown_wizard
As you move through the forest, you encounter a strange man on a sled driven by large rabbits. You talk for a moment before the man moves away from you and deeper into the forest.
-> DONE
= travel
They travel through the forest.
-> DONE
```

在*示例 2*中，只有在使用`RANDOM(1, 10)`的结果小于或等于`3`时，才会遇到`brown_wizard`针法。这为玩家遇到这个角色创造了 30%的机会。这是一个*加权概率*的例子。与两个遭遇之间的概率相等不同，其中一个比另一个更受重视。`travel`针法比其他针法`brown_wizard`更有可能被玩家遇到。

在上一节*随机遭遇*中，我们学习了如何在针法中创建不同的内容，并使用洗牌以相等的概率选择它们。在本节中，我们通过在 ink 中使用`RANDOM()`函数的加权概率来控制这种随机性。

在下一节中，我们将结合这种和之前的*条件性内容*模式。根据玩家之前选择的选项，我们可以通过随机性和比较其他值来影响玩家遇到的遭遇。

## 条件性内容

在不使用随机性的项目中，使用条件块或选择来响应玩家选择以及他们如何通过故事进展是很常见的。正如我们在*随机遭遇*和*加权随机性*部分所看到的，我们也可以在 ink 中使用洗牌和`RANDOM()`函数来塑造故事。在本节中，我们将查看一个使用这两个概念结合来创建更复杂的程序以生成玩家之间内容连接的示例。

在上一节，*加权随机性*中，我们看到了如何在块内创建一组不同的条件语句来控制玩家接下来会遇到什么。在 *示例 2* 中，这是 `brown_wizard` 或 `travel` 线迹的加权结果，其中 `travel` 线迹更有可能被玩家看到。然而，玩家很少只想在游戏中阅读文本。他们希望对故事发生的事情有所输入。

通过使用墨水中的选择标签，我们可以测试玩家是否选择了特定的选项，然后影响玩家的加权结果：

示例 3：

```cs
VAR percentage = 0
A vast forest stretches out before you and alongside the forest is a winding river.
* (travel_forest) [Enter the forest]
* (travel_river) [Travel by river]
-
~ percentage = RANDOM(1,10)
{
    - percentage <= 3 && travel_forest == 1: 
        <- encounter.brown_wizard
    - percentage > 3 && travel_forest == 1:
        <- encounter.travel
    - travel_river == 1:
        <- encounter.river
}
== encounter
= brown_wizard
As you move through the forest, you encounter a strange man on a sled driven by large rabbits. You talk for a moment before the man moves away from you and deeper into the forest.
-> DONE
= travel
They travel through the forest.
-> DONE
= river
You travel down the river safely.
-> DONE
```

*示例 3* 是 *示例 2* 的更新版本。然而，它不仅仅展示文本，玩家在编织中会看到两个选项。根据他们选择的哪一个，故事随后会沿着两条可能的路径分支。在第一种情况下，如果玩家选择在森林中旅行，他们有 30%的几率会遇到一个角色。在第二种情况下，如果玩家选择沿着河流旅行，他们将不会遇到这个角色。

虽然一些项目可能会使用洗牌或加权选项，但更多的项目会结合玩家的选择和过去的选项与随机性。这不仅让玩家对他们的体验有更多的控制权，还允许作者构建一个故事，以及某些可预测的结果。在只使用洗牌时，不是尝试解释多个结果，而是使用编织及其有限的选择数量来塑造未来遭遇的可能路径。因为波浪中只有两个选项，所以只有两个可能的主要分支，加权随机性只影响其中一个，不影响另一个。

在这个主题中，我们探讨了在 ink 项目中引入或调整简单的程序化叙事规则的三种不同模式。在第一部分，*随机遭遇*，我们学习了如何使用线程的洗牌来创建一组等权重的条目，以引入不同的故事内容。在第二部分，*加权随机性*，我们探讨了如何通过加权结果来控制随机性，其中一种结果比另一种更有可能。

在最后一节，*条件内容*中，我们将随机性与玩家选择选项的结果相结合，研究了如何创建看似更高级的规则，其中编织内的选择数量对故事形状的影响比任何分支中包含的随机性更强。

在下一个主题中，我们将探讨更复杂的模式。对于许多项目来说，ink 将是内容生成和项目如何使用程序化叙事的驱动力。我们将探讨如何将值加载到 ink 中，作为检查如何编写故事语法的部分，并计划玩家如何以动态方式遇到其不同部分。

# 将值加载到墨水中

程序化叙事的*程序化*方面可以主要存在于 ink 或 Unity 中。在这个主题中，我们将检查将值加载到 ink 中的过程。我们将集中设计，让 ink 在游戏会话期间对用户可能看到或与之交互的内容做出程序化决策。

在第一部分，*替换语法*中，我们将考虑如何使用我们在上一部分*ink 中介绍程序化叙事*中学到的知识来构建一套可能的玩家事件。这将引导我们进入下一部分*故事规划*，在那里我们将规则应用于语法本身。这将允许我们控制不同遭遇集如何受先前遭遇的影响，为复杂的故事创建简单的公式。

## 替换语法

在语言学中，**语法**描述了定义语言工作方式的规则。例如，在英语中，句子中有主语、谓语和宾语的具体顺序。在编程环境中，我们可以定义所谓的**替换语法**，其中一组规则描述了单词或短语如何被替换。这通常用于定义特定的顺序，例如在英语句子中使用主语和谓语。在编程环境中，可以产生动态构造，其中动态或随机值被*替换*在定义模式中的特定位置。

在 ink 中，我们可以创建函数来根据洗牌返回值。通过编写语法——即规定条目出现的顺序的规则——我们可以创建一个简单的替换模式，其中从特定集合中随机使用条目来创建动态文本交互：

示例 4：

```cs
First, we saw the {getLocation()}. Next, we visited the 
  {getMarker()}.
== function getLocation() ==
~ return "{~tower|ruin|temple}"
== function getMarker() ==
~ return "{~grave|farmstead|ancient tree}"
```

在*示例 4*中，`getLocation()`和`getMarker()` ink 函数提供了句子语法中的替换。通过在函数内放置洗牌操作并用引号包围它们，可以返回文本的结果；也就是说，在函数被调用时。因为所有函数都是全局的，这也意味着它们可以在代码中多次使用。

警告

根据*示例 4*，可能会诱使我们假设函数也可以用来使用洗牌生成可能的分叉目标。这不是事实。虽然变量可以持有分叉目标，但在 ink 中不允许函数进行分叉，并且语言阻止了调用函数和使用返回值来线程或分叉到故事中的另一个部分的组合。

ink 中的函数可以用于生成和返回文本。然而，由于其设计，ink 不允许函数控制故事流程。在每种洗牌操作可能还想使用分叉或线程的情况下，我们可以创建一个扩展的隧道，其中隧道的每一部分都作为语法的一部分：

示例 5：

```cs
 location -> marker -> DONE
== location
First, we saw the <>
{shuffle:
    - tower
    - ruin
    - temple
}<>.
->->
== marker
Next, we saw the <>
{shuffle:
    - grave
    - farmstead
    - ancient tree
}<>.
->->
```

*示例 5* 是使用扩展隧道对 *示例 4* 进行重写的一个版本。对于简单的文本替换，*示例 4* 中展示的模式，即使用洗牌和函数，可以非常实用。然而，*示例 5* 中的模式，即使用结和多行洗牌，允许洗牌中的每个条目可能改变或使用线程本身。这通常是创建替换语法的首选模式，其中语法的每个部分都可以根据需要扩展。

在本节中，我们学习了如何使用替换语法进行文本替换，然后是更高级的语法来整合隧道。在下一节中，我们将将替换语法作为故事规划过程的一部分来应用。循环和其他条件方面将被引入以创建更高级的替换语法。

## 故事规划

在上一节中，我们看到了替换语法如何为我们提供特定的事件顺序。通过使用洗牌，我们可以为每个部分选择随机的条目，为玩家创造动态体验。在上一节中展示的示例中，每个语法部分也只有一个条目。有一个用于 `location`，一个用于 `marker`，然后隧道结束。这很有用，但许多游戏将希望根据先前条目创建动态模式。换句话说，也有可能在语法内部基于先前的条目来设定未来条目的范围。

当我们创建一个公式，其中先前条目可以作为高级语法的部分影响未来条目时，我们正在使用一个称为 **故事规划** 的概念。在程序化叙事中，当故事根据生成比简单替换更复杂模式的规则进行规划时，就会发生故事规划。

如在 *第三章* 中所述，*序列、循环和洗牌文本*，替代方案可以嵌套在彼此内部。这意味着我们可以在多行条件块中使用替代方案，以创建基于先前值的上下文，从而选择随机的条目：

示例 6：

```cs
VAR location_pick = 0
-> location -> marker -> DONE
== location
First, we saw the <>
{shuffle:
    - tower
        ~ location_pick = "tower"
    - ruin
        ~ location_pick = "ruin"
    - temple
        ~ location_pick = "temple"
}<>.
->->
== marker
Next, we saw the <>
{
    - location_pick == "tower":
       {shuffle:
            - grave
            - memorial stone
        }
    - else:
        {shuffle:
            - farmstead
            - ancient tree
        }
}<>.
->->
```

在 *示例 6* 中，根据 *示例 5* 中的代码引入了一个新变量。在这个新版本中，`marker` 结的值基于 `location_pick` 变量。在从 `location` 结到 `marker` 结的扩展隧道中，`location_pick` 变量会改变。根据其值进入 `marker` 结，可以产生不同的结果。如果 `location` 的随机条目是 `"tower"`，则前两个值 `grave` 和 `memorial stone` 被启用。否则，`farmstead` 和 `ancient tree` 的值被启用。

在本主题中，我们专注于在 ink 中加载和生成值。在第一部分，*替换语法*中，我们学习了如何创建简单的模式。在本节，*叙事规划*中，我们回顾了一个使用单个变量在隧道第二部分中进行分支的简单叙事规划示例。根据所需的规划，作者可以使用不同的变量创建非常复杂的语法，其中先前值可以分支出未来的计算和洗牌或其他替代方案中的条目范围。

在下一个主题中，我们将从墨水（ink）转向 Unity。当涉及到脚本化叙事体验时，ink 是一种令人难以置信的语言。然而，ink 在处理更复杂的值操作时并不适用。在 Unity 中，使用 C#，我们可以执行更复杂的程序化叙事方法，其中我们可以决定加载哪个 ink 故事以及如何传递其值以进行内部决策。

# 在 Unity 中编码集合

在上一个主题中，我们探讨了 ink 为玩家创建和规划内容的方法。在本节中，我们回到 Unity。在大型项目中，无论是故事还是其他内容，叙事内容通常会是游戏中的多个复杂互锁机制之一。在这些情况下，程序化叙事将是多个系统之一，Unity 作为推动项目的游戏引擎，将被编程为在更复杂的操作和规划中使用一个故事而不是另一个。在这些情况下，叙事内容存储在 C#称为*集合*的地方。这些可以是像数组这样简单的东西，也可以是更复杂的数据结构，能够根据模式或其内部元素的值对内部元素进行排序。

在第一部分，*使用多个故事*中，我们将查看一个示例，将项目的程序化叙事方面从 ink 移动到 Unity。我们不会在 ink 中使用洗牌（shuffles），而是在 Unity 中使用随机性来选择集合中不同的可能故事，然后从未来的选择中移除它们。这将使我们能够专注于 ink 中的故事内容，在单独的文件中创建对话或玩家选择，然后使用 Unity 来选择要向玩家展示的内容。

在最后一节，*条件性地选择故事*中，我们将使用 Unity 中的 ink 应用简单的叙事规划概念，正如*叙事规划*子节中所示。这与 ink 中的操作非常相似，这将使我们能够开始定义一个替换语法，以确定我们希望故事内容如何呈现给玩家，但由 Unity 中的编码集合执行选择部分的工作，而不是 ink。

## 使用多个故事

如我们首先在 *第十一章* 中探讨的，在 *Quest Tracking and Branching Narratives*（任务跟踪和分支叙事）主题中，在 *Tracking progress across multiple quests*（跟踪多个任务进度）这一主题中，可以使用多个墨迹文件作为项目中 `Story` 类的独立实例。在那个主题中，每个文件都是一个独立的故事。然而，也可以将每个文件用作更大故事中的一个场景。在这些情况下，每个墨迹文件将代表玩家的一种独立的叙事体验。一旦被 Unity 选择，它就可以作为会话的一部分或更长的故事，以随机顺序向玩家展示。

注意

本节完成的项目的示例可以在 GitHub 上的 *第十二章* 找到，名称为 `Chapter12-MultipleStories`。只会展示与正在考察的概念相关的代码部分。

当你使用多个墨迹文件将故事分解成不同的场景，每个场景可以独立访问时，一次性加载它们是最简单的方法。以下项目使用名为 `GetFiles()` 的方法处理编译后的 JSON 文件并创建 `Story` 类实例。每当创建一个新的对象时，它就会被添加到一个名为 `Stories` 的 `List<Story>` 集合中：

```cs
void GetFiles()
{
     string inkPath = Application.dataPath + "/Ink/";
     foreach (string file in Directory.GetFiles(inkPath,
       "*.json"))
     {
           string contents = File.ReadAllText(file);
           Stories.Add(new Story(contents));
     }
}
```

在 *Introducing procedural storytelling in ink*（介绍墨迹中的程序化叙事）主题的 *Random encounter*（随机遭遇）部分，使用了洗牌来在不同的线程之间进行选择。在 C# 中，`Random` 类的工作方式类似。它基于某个范围提供随机数据。使用其 `Next()` 方法以及集合的 `Count` 属性，它提供了一个索引来在 `List<Story>` 集合中选择条目，该集合由 `GetFiles()` 方法填充：

```cs
void PickRandomStory()
{
      if (Stories.Count > 0)
     {
           System.Random rand = new System.Random();
           int index = rand.Next(Stories.Count);
           Story entry = Stories[index];
           Stories.RemoveAt(index);
           UpdateContent(entry);
     }
     else
     {
           SceneDescription.text = "(There are no more
             stories.)";
     }
}
```

为了防止相同的故事再次出现，`RemoveAt()` 方法会随机地从 `List<Story>` 集合中移除条目。这防止了相同的故事被展示两次。

总的来说，`Start()` 方法用于调用多个其他方法来解析文件并随机选择一个故事。基于随机选择的 `Story` 中的编织内容，名为 `UpdateContent()` 的方法（由 `PickRandomStory()` 调用）向玩家提供两个选项，作为 `Button` 游戏对象。点击任何一个都会改变故事中变量的值。然后，这些更新作为两个变量的显示呈现给玩家，这两个变量在 Unity 中被跟踪：`violence`（暴力）和 `peace`（和平）：

```cs
void Start()
{
     Stories = new List<Story>();
     UpdateStatistics();
     GetFiles();
     PickRandomStory();
}
```

虽然相对简单，但本节中展示的项目说明了在 ink 和 Unity 作为程序化叙事的独立系统之间平衡的重要方面。墨水故事的复杂性不会反映在选择它从集合中或显示其内容所需的 C#代码中。在 Unity 中，可以使用简单的代码随机选择一个墨水故事，该故事本身在其设计中使用随机性、替换语法或其故事规划。在 Unity 中，可以使用 C#的`Random`类，而不需要了解墨水故事正在做什么。

在下一节中，我们将遵循与我们在“在 ink 中引入程序化叙事”主题中所做的类似动作。在本节中，我们专注于使用多个 ink 故事与 C#的`Random`类，并在它们之间进行平等选择。然而，大多数项目只想根据先决条件选择墨水故事。在下一节中，我们将探讨在墨水故事之间进行条件选择。

## 条件选择故事

在上一节中，我们看到了 C#的`Random`类如何允许我们根据`Story`类作为集合`List<Story>`的一部分来选择对象。这对大多数项目来说作用有限。相反，大多数开发者更希望控制何时选择一个墨水故事以及它成为可用的条件。在本节中，我们将探讨一个简单系统的实现，该系统在加载任何内容之前会检查故事的先决条件。值将在集合中的故事之间跟踪，如果满足故事的先决条件，它将被视为可用。如果不满足，则会被忽略。

注意

本节完成的项目的示例可以在 GitHub 上的*第十二章*示例中找到，名称为`Chapter12-ConditionalStories`。将仅展示与正在检查的概念相关的代码部分。

要检查墨水故事的先决条件，需要一个单独的类，`ConditionalStory`，它包含墨水故事以及原本出现在*第十一章*中“显示和奖励玩家进度”部分的“跟踪任务值”子部分的`ObserveVariables()`和`UpdateVariable()`方法：

```cs
public void ObserveVariables(Story.VariableObserver
  callback)
{
      InkStory.ObserveVariables(new List<string>() {
        "violence", "peace" }, callback);
}
public void UpdateVariable(string name, object value)
{
      if(InkStory.variablesState.
        GlobalVariableExistsWithName(name))
      {
          if (!InkStory.variablesState[name].Equals(value))
          {
                 InkStory.variablesState[name] = value;
          }
      }
}
```

`ConditionalStory`类有一个名为`Available()`的方法。内部，它使用`Story`类的`EvaluateFunction()`方法来调用一个名为`check()`的 ink 函数。假设墨水故事包含该函数，它将被调用，并将结果转换为布尔值：

```cs
public bool Available()
{
     bool result = false;
     if(InkStory.HasFunction("check"))
     {
           result = (bool)
              InkStory.EvaluateFunction("check");
     }
     return result;
}
```

每个故事文件都有一个条件检查，该检查被反馈到`ConditionalStory`类的`Available()`方法中。如果`check()` ink 函数返回`true`，则故事可用于使用。

对上一节中显示的代码进行了各种修改，*使用多个故事*。第一个是使用`ConditionalStory`作为一个包含基于`Story`类的对象的类。第二个是`SelectStories()`方法。与随机选择条目不同，它使用`List<ConditionalStory>`的`FindAll()`方法搜索其条目。如果`Available()`方法（每次调用时都会调用 ink 中的`check()`函数）报告`true`，则认为故事是可用的：

```cs
List<ConditionalStory> selection = Stories.FindAll(e => 
  e.Available());
if (selection.Count > 0)
{
      System.Random rand = new System.Random();
      int index = rand.Next(selection.Count);
      ConditionalStory entry = selection[index];
      Stories.Remove(entry);
      UpdateContent(entry);
}
```

如果每个墨迹故事都定义了它是否以及如何可用于更大的项目，这允许墨迹和 Unity 中的 C#代码分别开发。为了使其可用以便选择，ink 中的`check()`函数必须向 C#中的`ConditionalStory`类报告`true`。这创建了一个简单但易于重复的模式，用于在 Unity 中创建基于对如何使用`FindAll()`方法（如本节所述）和`Random`类（如前节所述）的集合工作原理的理解的基于条件的故事。

# 摘要

本章的目标不是解决程序化叙事的所有问题或涵盖所有可能的算法。第一个主题，*在 ink 中介绍程序化叙事*，回顾了重要概念，例如随机性如何在 ink 中选择内容中发挥作用。第二个部分，*将值加载到 ink 中*，探讨了如何使用 ink 结合更高级的概念，如语法和故事规划。最后，在*在 Unity 中编码集合*这一主题中，我们看到了 Unity 如何用于在集合中随机选择 ink 故事（第一部分），以及如何通过 ink 故事和 Unity 中的 C#类之间的通信结合一些简单的条件测试。

我们现在已经完成了这本书的最后一章，希望你能带着不同的概念去探索，以及一些简单的模式来用于更高级的项目。程序化叙事是一个多样且深奥的主题。许多研究人员和开发者已经创建并继续探索构建替换语法的可能方法、规划故事以及使用 ink 和 Unity（单独或一起）为玩家制作复杂故事和体验的简单规则。

# 问题

1.  什么是程序化叙事？

1.  什么是随机表？

1.  什么是加权随机性？

1.  什么是替换语法？

1.  什么是故事规划？
