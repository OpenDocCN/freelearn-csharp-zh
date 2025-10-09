# 4

# 诚实函数、`null` 和 `Option`

在本章中，我们讨论诚实函数的艺术和科学、`null` 的复杂性以及针对它们定制的 C# 工具。但在我们深入探讨之前，让我们制定路线图：

+   理解诚实函数

+   隐藏的 `null` 的问题

+   拥抱可空引用类型的诚实

+   除了 `null`：`Option`

+   现实场景

在你浏览这一章的过程中，要寻找实用的建议和有证据支持的指南。带着好奇心来，到结束时，你将掌握编写更透明和健壮的 C# 代码的知识。

就像我们在前面的章节中所做的那样，让我们评估你的位置。这里有三个任务供你完成。如果你不确定如何应对它们，立即进入这一章。但是，如果你对自己的技能有信心，也许你可以快速浏览并跳到最具挑战性的部分。准备好了吗？让我们开始吧！

# 任务 1 – 重新设计诚实返回类型

这里有一个史蒂夫的塔防游戏中的函数，根据其 ID 获取一个塔。将其重构为返回一个表示潜在空值的诚实类型：

```cs
public Tower GetTower(int towerId)
{
     var tower = _gameState.GetTowerById(towerId);
     return tower;
}
```

# 任务 2 – 防止空输入

将以下函数重构为使用预条件来防止空输入，并在预条件未满足时抛出适当的异常：

```cs
public void UpgradeTower(TowerUpgradeInfo upgradeInfo)
{
     _gameState.UpgradeTower(upgradeInfo);
}
```

# 任务 3 – 使用可空类型进行模式匹配

给定以下类，编写一个方法，接收一个敌人并使用模式匹配返回其描述字符串：

```cs
public abstract class Enemy {}
public class GroundEnemy : Enemy
{
     public int Speed { get; set; }
     public int Armor { get; set; }
}
public class FlyingEnemy : Enemy
{
     public int Altitude { get; set; }
     public int DodgeChance { get; set; }
}
public class BossEnemy : Enemy
{
     public int Health { get; set; }
     public string SpecialAbility { get; set; }
}
public string GetEnemyDetails(Enemy? enemy)
{
     // Your code here
}
```

再次，如果你确信你知道所有三个任务的正确答案，你可以跳过这一章。当然，如果你有任何问题，随时可以回来。现在让我们来讨论诚实函数及其好处。

# 诚实函数 – 定义和理解

在与朱莉娅关于纯函数和副作用交谈几天后，史蒂夫渴望了解更多。他打电话给朱莉娅询问他接下来应该学习什么。

朱莉娅：*让我们来谈谈诚实函数、`null` 和 `Option` 类型，*她建议。“这些概念对于编写清晰、可预测的代码至关重要。”

史蒂夫：*听起来很棒！但* *诚实函数* *究竟是什么？*

朱莉娅：*一个诚实函数在函数与其调用者之间提供了一个清晰、明确的契约，从而使得代码更加健壮，更不容易出错，并且更容易* *推理* *。”

那么一个诚实函数究竟是什么呢？

简单来说，一个诚实函数是指其类型签名完全且准确地描述了其行为。如果一个函数声称它将接受一个整数并返回另一个整数，那么它就会做到这一点。没有任何隐藏的陷阱，没有突然抛出的异常，也没有未在函数签名中反映的全局或静态状态的变化。

考虑以下 C# 函数：

```cs
public static int Divide(int numerator, int denominator)
{
    return numerator / denominator;
}
```

初看，这个函数似乎履行了其承诺。它接受两个整数并返回它们的商。然而，当我们传递零作为分母时会发生什么？会抛出`DivideByZeroException`。这个函数并没有完全对我们诚实；其签名没有警告我们这个潜在的风险。这个函数的诚实版本会在其签名中明确指出失败的可能性。

那么，这种诚实概念如何与函数式编程和软件开发行业的更广泛背景相结合呢？在函数式编程中，函数是代码的构建块。这些函数描述其行为越清晰、越诚实，构建更大、更复杂的程序就越容易。每个函数都作为一个可靠的组件存在，其行为正如其签名所描述的那样，允许开发者自信地组合和重用这些函数。

当朱莉娅解释诚实函数时，史蒂夫的眼睛因理解而亮了起来。

史蒂夫：*所以，就像我告诉我的队友我会在周五交付一个功能一样，我应该* *真正地做到这一点吗*？

朱莉娅：*没错！在编程中，我们的函数应该* *同样可靠*。

现在，让我们深入探讨使用诚实函数的好处：

+   **提高可读性**：诚实函数使代码更易于阅读和理解。无需深入研究实现细节即可了解函数的功能。函数的签名是一个合同，准确描述了其行为。

+   **增强可预测性**：使用诚实函数，意外情况大大减少。函数的行为正是签名中描述的那样，导致运行时出现更少的意外错误和异常。

+   **提高可靠性**：通过最小化意外情况并明确处理潜在的错误场景，诚实函数导致代码库更加健壮，能够承受现实世界使用的严酷考验。

考虑一个每个函数都是诚实的代码库。任何开发者，无论是否熟悉代码，都可以查看函数签名并立即了解其功能、所需条件和可能返回的内容，包括任何潜在的错误条件。这就像拥有一个文档齐全的代码库，而不需要冗长的文档。这就是诚实函数的力量和承诺。

在接下来的章节中，我们将探讨如何在 C#中实现诚实函数以及如何处理潜在的欺骗行为，例如空值或异常。系好安全带，因为我们对诚实与不诚实函数世界的探索才刚刚开始！

# 隐藏空值的弊端

史蒂夫回忆起他最近在塔防游戏中的一个错误。

史蒂夫：*我想我遇到过这个问题。当我尝试升级一个* *不存在的塔* *时，我的游戏崩溃了*。

朱莉娅：*这是一个典型的例子，让我们看看我们今天所学的内容如何防止这种情况发生*。

你是否曾经感到困惑，试图弄清楚为什么你的程序停止工作？大多数时候，问题来自众所周知的 `NullReferenceException`。本节探讨了 C# 和空值之间复杂的关系，指出了许多开发者面临的问题。

## 快速回顾一下——C# 和空值

为了理解我们当前的问题，让我们回顾一下。Tony Hoare，一位重要的计算机专家，称空引用为“一个巨大的错误”。这个想法是为开发者提供一个工具来显示值缺失的情况。起初，这似乎是一个好计划，但最终它导致了许多问题和错误，包括 C# 语言。

当 C# 被引入时，它采用了来自较老编程方法的一个想法，允许开发者使用空值来表示某物缺失。但随着时间的推移，这个简单的选择导致了大量的错误和混淆。

## 常见错误和令人烦恼的 NullReferenceException

所有 C# 开发者，无论是新手还是经验丰富的，都遇到过 `NullReferenceException`。当你尝试使用不存在的东西时，这个错误会发生。

想象一个获取用户信息的程序。你可能会期望总能找到用户：

```cs
var user = FindUserById(userId);
var fullName = $"{user.FirstName} {user.LastName}";
```

哎呀！如果 `FindUserById` 无法找到用户，第二行将抛出 `NullReferenceException`，如果不捕获它，将会终止整个线程的执行。这个错误发生是因为我们以为用户总是会存在的。这显示了隐藏的空值如何在我们的代码中引起意外的问题。

许多程序比这个例子大得多，这使得这些错误难以找到和修复。这些隐藏的问题可能长时间隐藏，在你最不期望的时候引发错误。

隐藏的空值可以被视为看不见的陷阱。它们可以捕捉到新手和经验丰富的程序员。此外，它们与函数式编程的主要理念相悖，函数式编程重视清晰和预期的结果。

## 并非全是坏事——空值的价值

容易把问题归咎于空值。但真正的问题是它的使用方式。如果使用清晰的方式来表示某物缺失，空值是有帮助的。问题在于其使用不明确，导致许多潜在的错误。

C# 与空值的旅程有起有落。但，正如我们将在下一节中学习的，C# 现在有多种处理空值的方法，使我们的代码更清晰、更直接。其中一种方法就是使用可空引用类型。

# 用可空引用类型拥抱诚实

在 C# 中处理空值一直是一个很大的挑战。许多软件开发者（包括我）都主张在代码审查清单中检查 `NullReferenceException` 作为一项强制性任务。在大多数情况下，仅通过查看拉取请求就可以轻松检查可能的空值，即使没有 IDE。最近，当微软引入了可空引用类型时，我们得到了帮助。因此，现在，编译器将加入我们寻找由空值引起的可能灾难的行列。

## 什么是可空引用类型？

简而言之，**可空引用类型**（或简称**NRTs**）是 C#中的一个特性，允许开发者明确地指出一个引用类型是否可以为空。有了这个特性，C#为我们提供了一个工具，从代码一开始就清楚地表达我们的意图。把它想象成一个路标，引导其他开发者（甚至是我们未来的自己）了解我们的代码应该期待什么。

没有 NRTs，C#中的每个引用类型都可能被赋值为 `null`。这会变成一个猜谜游戏。这个变量会具有值还是会被赋值为 `null`？现在，有了 NRTs，我们不再需要猜测。代码本身就在讲述这个故事。

让我们通过一个基本示例来理解这个概念：

```cs
string notNullable = "Hello, World!";
string? nullable = null;
```

在前面的代码片段中，`notNullable` 变量是一个普通的字符串，不能被赋值为 `null`（如果你尝试这样做，编译器会发出警告）。另一方面，自从 C# 8.0 开始，可空类型会明确地用 `?` 标记，表示它可以被赋值为 `null`。

在某些情况下，你可能想将 `null` 赋值给未标记为可空的变量。在这种情况下，为了抑制警告，你可以使用 `!` 符号来让编译器知道你已经意识到自己在做什么，并且一切都在计划中进行：

```cs
string notNullable = "Hello, World!";
notNullable = null!;
```

NRTs（可空引用类型）最大的优点之一是，C# 编译器会警告你如果在使用可空值时可能正在执行有风险的操作。这就像有一个友好的向导始终在关注你的肩膀，确保你不会陷入常见的空值误用的陷阱。

例如，如果你尝试在未检查 `null` 的情况下访问可空引用类型的属性或方法，编译器会提前警告你。

## 转向 NRTs

对于那些有现有 C#项目的开发者来说，你可能想知道：*如果我启用 NRTs，我的项目会被警告信息充斥吗？* 答案是否定的。默认情况下，NRTs 是关闭的。你可以选择按文件启用此功能，以便平稳过渡。

NRTs 是解决长期存在的空引用挑战的好方法。通过在我们的代码中明确表示空值的存在，我们迈出了向清晰、安全和功能诚实的大步。最终，拥抱 NRTs 不仅使我们的代码更加健壮，而且确保了我们的意图，作为开发者，是透明的。

## 启用可空引用类型

要启用 NRTs，我们需要告诉 C#编译器我们已经准备好接受它的指导。这是通过一个简单的指令完成的：`#nullable enable`。

在你的 `.cs` 文件开头放置以下内容：

```cs
#nullable enable
```

从文件的这个点开始，编译器创建了一个特定的可空上下文，并假设所有引用类型默认为不可空。如果你想使一个类型为可空，你必须明确地用 `?` 标记它。

启用 NRTs 后，C#编译器成为你的安全网，指出代码中潜在的空值问题。无论何时你尝试将 `null` 赋值给没有 `?` 标记的引用类型，或者尝试访问未检查的可空变量，编译器都会发出警告。

这里有一个例子：

```cs
string name = null; // This will trigger a warning
string? maybeName = null; // This is okay
```

## 禁用可空引用类型

在将项目过渡到使用 NRTs 的过程中，你的代码中可能会有一些部分你希望延迟过渡。你可以使用`#nullable disable`指令关闭这些特定部分的 NRTs：

```cs
#nullable disable
```

这告诉编译器恢复到旧的行为，将所有引用类型视为可能为 null。

你可能会想知道为什么 C#选择使用指令来实现这个特性。原因是灵活性。通过使用指令，开发者可以逐步将 NRTs（非空引用类型）引入到他们的项目中，一次一个文件，甚至一次一个代码段。这种分阶段的方法使得适应现有项目变得更加容易。

## 警告和注释选项

说到分阶段的方法，还有两个选项可以设置我们的可空上下文：`warnings`和`annotations`。你可以通过编写以下内容来使用它们：

```cs
#nullable enable warnings
```

或者，你可以这样写：

```cs
#nullable enable annotations
```

这些选项的主要目的是为了简化将现有代码从完全禁用的 null 上下文迁移到完全启用上下文的过程。简而言之，我们希望通过开启`warnings`选项来获取解引用警告。当所有警告都修复后，我们可以切换到`annotations`。这个选项不会给我们带来任何警告，但它将开始将我们的变量视为不可为 null，除非用`?`标记声明。

要获取有关这些选项和生成文件中 null 上下文的信息，以及了解更多关于三种 nullability（不可知、可空和非空）的内容，我建议你阅读文章《可空引用类型》([`learn.microsoft.com/en-us/dotnet/csharp/nullable-references`](https://learn.microsoft.com/en-us/dotnet/csharp/nullable-references))。你可能还想阅读文章“使用可空引用类型更新代码库以改进 null 诊断警告”([`learn.microsoft.com/en-us/dotnet/csharp/nullable-migration-strategies`](https://learn.microsoft.com/en-us/dotnet/csharp/nullable-migration-strategies))。

## 更大的图景——项目级设置

虽然指令对于细粒度控制很好，但你也可以为整个项目启用 NRTs。在项目设置中，或直接在`.csproj`文件中，将`<Nullable>`元素设置为启用：

```cs
<PropertyGroup>
    <Nullable>enable</Nullable>
</PropertyGroup>
```

这个设置将项目中的每个文件都视为从`#nullable enable`指令开始。当你为整个项目开启可空上下文时，你可能希望保留一些代码部分不生成警告，但在之后使用项目级别的可空上下文。在这种情况下，你可以使用`restore`选项：

```cs
#nullable enable
// The section of the code where nullable reference types are enabled.
#nullable restore
```

启用可空引用类型就像在之前昏暗的房间里打开一盏灯。它揭示了潜在的陷阱，并确保我们编写更安全、更清晰的代码。有了 C#提供的工具，我们对此功能既有细粒度的控制，也有广泛的控制，使得过渡到更透明的编码风格既可行又有益。

# 有意返回

在函数式编程领域，函数的主要目标是透明。透明意味着函数不仅应该做其名称暗示的事情，而且其返回类型应该提供对预期内容的清晰契约。让我们深入探讨在 C#中构建诚实的返回类型。

经验丰富的开发者知道，一个函数的名称或签名本身可能无法描述整个故事。考虑以下：

```cs
UserProfile GetUserProfile(int userId);
```

表面上看，这个函数似乎承诺根据用户 ID 获取用户资料。然而，疑问仍然存在。如果用户不存在怎么办？如果检索资料时出现错误怎么办？

现在考虑一个替代方案：

```cs
UserProfile? GetUserProfile(int userId);
```

通过简单地在返回类型中引入`?`，函数变得更加透明地表达了其意图。它暗示着：“我将尝试为这个 ID 获取用户资料。但有可能你会得到*一个 null*。”

## UserProfile 和 UserProfile?之间的区别

虽然这种区别可能看起来微不足道，但其影响是深远的：

+   `UserProfile?`，你立刻知道有可能得不到用户资料。

+   **防御性编程**：知道返回值可能为 null，你会自然地编写更安全的代码来处理这些情况。

+   **错误处理**：而不是使用异常或错误代码来表示数据缺失，可空返回类型提供了一种清晰、类型安全的方式来表达缺失的可能性。

让我们看看实际应用：

```cs
var userProfile = GetUserProfile(userId);
if (userProfile is null)
{
    // Handle the scenario when the profile is not available
}
else
{
    // Proceed with the user profile data
}
```

正如你所见，借助 NRTs（非空引用类型），我们可以更清晰地理解代码并正确处理其结果。而我们为了实现这一点所做的，仅仅是稍微调整了方法签名。

## 尊重函数的契约

一个诚实的函数不仅仅表明可能返回 null，它还创造了一种“有意返回”的心态。当有意返回时，你不仅仅是返回数据；你是在传达一种状态。

考虑以下示例。

`null`检查并避免潜在的`NullReferenceException`：

```cs
List<Order> GetOrdersForUser(int userId)
{
    return ordersRepository.FindByUserId(userId) ?? new List<Order>();
}
```

**获取数据**：在检索可能不存在的数据时，而不是抛出异常，可空类型可以更清晰地描绘情况：

```cs
Product? FindProductById(int productId)
{
    return productsRepository.GetById(productId);
}
```

透明的返回类型导致更少的意外和更健壮的代码。通过清楚地传达函数可以返回的内容，你做了以下事情：

+   **减少错误**：因为开发者将处理他们可能忽视的其他场景

+   **提高清晰度**：开发者花费更少的时间挖掘函数实现，依靠返回类型来指导行为

+   **建立信任**：清晰的契约确保函数履行其承诺，在代码库中创造一种可靠性感

随着我们继续探索诚实的函数，请始终记住：你的函数既是执行者也是沟通者。让它们不仅完成其任务，而且透明地传达它们的意图和潜在结果。这样做，你构建了弹性系统，创建了清晰的契约，并鼓励一个更安全、更可预测的编码环境。

# 对函数输入提出诚实的要求

一个真正健壮的系统不仅关乎你返回什么，还关乎你接受什么。保护你的函数免受可能误导或有害的输入是创建可预测、容错应用程序的关键。

考虑这个常见的场景：你有一个期望特定类型输入的函数。然而，当 `null` 值悄悄进入时，你的函数会崩溃，导致臭名昭著的 `NullReferenceException`。为了减轻这种情况，C# 提供了一种使用可空引用类型要求函数输入诚实的方法。

假设你定义了一个函数如下：

```cs
public void UpdateUserProfile(UserProfile profile)
{
    // Some operations on profile
}
```

目的明确：函数期望 `UserProfile`。然而，什么阻止了开发者传递 `null`？

## 可空引用类型拯救了

如我们之前讨论的，在 C# 8.0 中，可空引用类型增加了另一层保护。通过使用 `#nullable enable` 启用可空引用类型，编译器变成了你的守护者：

```cs
#nullable enable
public void UpdateUserProfile(UserProfile profile)
{
    // Some operations on profile
}
```

现在，如果任何开发者尝试将可空的 `UserProfile` 传递给函数，编译器将发出警告。这促使开发者走向正确的方向，并防止潜在的运行时错误。

然而，警告并不能保证不会使用 `null`，所以让我们看看另一种方法。在这里，我们用最简单的保护方法——直接的空检查来捍卫这个方法：

```cs
public void UpdateUserProfile(UserProfile profile)
{
    if (profile is null)
    {
        throw new ArgumentNullException(nameof(profile), "Profile cannot be null!");
    }
    // Some operations on profile
}
```

这个检查确保如果函数被提供了 `null`，它将立即停止执行并提供一个明确的原因。

## 使用先决条件和契约

契约和先决条件将保护的概念提升到了一个新的水平。通过定义一组在函数可以继续之前必须成立的规则，你使得函数的期望变得明确。

考虑微软提供的 CodeContracts ([`github.com/Microsoft/CodeContracts`](https://github.com/Microsoft/CodeContracts)) 库。有了这个库，你可以使用更丰富的语法来确保函数的先决条件：

```cs
public void UpdateUserProfile(UserProfile profile)
{
    Contract.Requires<ArgumentNullException>(profile != null, "Profile cannot be null!");
    // Some operations on profile
}
```

这段代码更加简洁，但它以同样的方式保护我们的方法免受 `null` 值的影响。

## 使用内置检查

在 C# 6.0 中，我们得到了一种新的方法来防止空值 - `ArgumentNullException.ThrowIfNull`：

```cs
public void UpdateUserProfile(UserProfile profile)
{
    ArgumentNullException.ThrowIfNull(profile);
    // Some operations on profile
}
```

现在，这段代码看起来更加整洁，也更容易阅读。此外，C# 7.0 中出现了一种更完善的方法来处理字符串：`ArgumentException.ThrowIfNullOrEmpty`。它不仅检查是否为空，还确保字符串不为空：

```cs
public void UpdateUserEmail(long userId, string email)
{
    ArgumentException.ThrowIfNullOrEmpty(email);
    // Updating the email after finding the user by ID
}
```

## 显式非空输入的力量

通过要求非空参数，你做了以下几件事：

+   **提高可预测性**：函数的行为符合预期，因为恶意的 `null` 值不会使它们出轨

+   **提升开发者信心**：有了清晰的契约，开发者可以调用函数而无需犹豫

+   **减少调试时间**：在编译时捕捉潜在问题总是比运行时调试快

为了编写好的函数式代码，确保你接受的内容清晰，就像确保你返回的内容透明一样重要。通过要求函数输入诚实，您为编写既可靠又具有弹性的代码奠定了坚实的基础。

# 模式匹配和可空类型

模式匹配是 C#工具箱中的强大工具，它不仅使代码更具表现力，而且更清晰、更安全。当与可空类型结合使用时，模式匹配帮助我们保护代码免受潜在陷阱的影响。

模式匹配，自 C# 7.0 引入并在后续版本中增强，是一种允许您将值与模式进行比较的功能，当值符合模式时，提供从值中提取信息的方法。

考虑经典的`switch`语句，结合模式匹配进化而来：

```cs
object tower = GetRandomTower();
switch (tower)
{
    case ArcherTower a:
        Console.WriteLine($"It's an Archer Tower with a range of {a.Range}!");
        break;
    case CannonTower c:
        Console.WriteLine($"It's a Cannon Tower with an explosion radius of {c.ExplosionRadius}!");
        break;
    default:
        throw new Exception("Unknown tower type!");
        break;
}
```

在这里，`ArcherTower`和`CannonTower`是类型。如果`tower`是`ArcherTower`类型，它不仅进入相应的 case 块，还将它转换为`ArcherTower`类型，允许您直接访问其属性。

## 带有可空类型的模式匹配

模式匹配在处理可空类型时真正大放异彩。让我们考虑一个获取用户配置文件的情况：

```cs
UserProfile? profile = GetUserProfile(userId);
```

我们如何以类型安全和清晰的方式处理这种潜在的 null？请进入模式匹配。

### 使用“is”模式

理想情况下，我们的代码应该易于阅读，还有什么比使用普通英语更容易呢？“is”模式就是为了帮助实现这一点：

```cs
if (profile is null)
{
    Console.WriteLine("Profile not found.");
}
else
{
    Console.WriteLine($"Welcome, {profile.Name}!");
}
```

这立即使代码更易于阅读，清楚地区分了不同的情况。

## 带有属性模式的 switch 表达式

自 C# 8.0 引入的`switch`表达式与属性模式结合，提供了一种更简洁的方式来处理复杂条件：

```cs
public class Book
{
    public bool IsPublished { get; set; }
    public bool IsDraft { get; set; }
}
string bookStatus = book switch
{
    null => "No book found",
    { IsPublished: true, IsDraft: false } => "Published Book",
    { IsPublished: false, IsDraft: true } => "Draft Book",
    { IsPublished: false, IsDraft: false } => "Unpublished Book",
    _ => "Unknown book status"
};
```

在这里，我们不仅检查 null，还检查`Book`对象的特定属性，根据它们的值做出决策。

## 使用可空类型确保清晰性

模式匹配与可空类型的结合确保以下内容：

+   **安全性**：通过显式处理潜在的 null，您降低了运行时错误的风险

+   **表现力**：模式允许您将复杂的条件逻辑压缩成简洁、易读的结构

+   **可读性**：不同条件和结果之间的清晰区分使开发者能够轻松理解流程。

当与可空类型结合使用时，模式匹配是任何 C#开发者的强大工具。它不仅简化了代码，而且增强了代码，使您能够编写对意外情况更具弹性的应用程序。使用它，您的代码将既易于编写，又可靠。

# 空对象模式

空对象模式是一种设计模式，它提供了一个对象作为给定接口中缺少的对象的代理。本质上，它在没有有意义的数据或行为的情况下提供默认行为。这种模式在预期有对象但实际没有，且不希望不断检查`null`的场景中特别有用。

## `null`检查的问题

想象你正在开发一个系统，在这个系统中，你需要在`User`对象上执行一系列操作。现在，并不是每个用户都可能在系统中初始化，这通常会导致以下情况：

```cs
if (user != null)
{
    user.PerformOperation();
}
```

这对于单个检查可能看起来很无害。但当你代码库中充满了这样的`null`检查时，代码变得冗长且难以阅读。`null`检查的泛滥也可能掩盖主要业务逻辑，使得代码库更难以维护和理解。

## 空对象解决方案

空对象模式为这个问题提供了一个优雅的解决方案。而不是使用空引用来传达对象的缺失，你创建了一个实现预期接口但不做任何事情的对象——一个“空对象”。

这里有一个例子：

```cs
public interface IUser
{
    void PerformOperation();
}
public class User : IUser
{
    public void PerformOperation()
    {
        // Actual implementation here
    }
}
public class NullUser : IUser
{
    public void PerformOperation()
    {
        // Do nothing: this is a null object
    }
}
```

在用户不可用的情况下，你将返回`NullUser`的实例，而不是返回`null`。

现在，当你想要执行操作时，你可以自信地这样做，而无需检查`null`：

```cs
user.PerformOperation();
```

无论`user`是`RealUser`还是`NullUser`，代码都不会抛出`NullReferenceException`。

## 优势

实现空对象模式提供了几个关键优势：

+   **减少条件语句**: 你可以减少显式`null`检查的数量，从而得到更干净、更易读的代码。

+   **安全性**: `null`引用异常的风险大大降低。

+   **多态性**: 通过将空对象与其他对象同等对待，你可以利用多态的力量，这可以简化并澄清代码。

+   **意图的清晰性**: 空对象可以有有意义的名称，这使得在“什么都不做”或默认行为是故意的时非常清楚。

## 局限性和考虑因素

虽然空对象模式有其优点，但它并不总是最佳解决方案：

+   **开销**: 对于非常大的系统或深度嵌套的结构，在各个地方引入空对象可能会增加开销

+   **复杂性**: 如果接口或基类频繁更改，维护相应的空对象可能会变得繁琐

+   **隐藏的错误**: 如果你不够小心，使用空对象可能会潜在地掩盖系统中的问题，因为它们提供了默认行为，可能会隐藏其他情况下由空引用暴露的问题。

空对象模式是 C#开发者工具箱中的强大工具。它不是万能的解决方案，但当你明智地应用它时，它可以极大地提高代码的清晰度和健壮性。像所有设计模式一样，理解何时何地应用它是至关重要的。所以不要急于在所有地方使用它，让我们看看一种更函数式的方法——`Option`类型。

# 超越`null` – 使用`Option`

`Nullable<T>`，它是 C#中内置的，是一个处理`null`值的好工具，但有一个更简单直接的结构来处理`null`值。`Option`是一种类型，它提供了更多表达性的工具来传达值的缺失或存在，作为可空类型的更丰富替代品。

## `Option`的简要介绍

在其核心，`Option`类型可以被视为一个可能包含或不包含值的容器。通常，它表示为`Some`（包装一个值）或`None`（表示值的缺失）。

通常，`Option`的实现看起来像这样：

```cs
public struct Option<T>
{
     private readonly bool _isSome;
     private readonly T _value;
     public static Option<T> None => default;
     public static Option<T> Some(T value) => new Option<T>(value);
     Option(T value)
     {
          _value = value;
          _isSome = _value is not null;
     }
     public bool IsSome(out T value)
     {
          value = _value;
          return _isSome;
     }
}
```

虽然 C# 没有内置的`Option`类型，但你可以使用上面的类型或使用像 LanguageExt ([`github.com/louthy/language-ext`](https://github.com/louthy/language-ext))这样的库，它提供了这个功能。

现在，让我们看看我们如何利用`Option`类型：

```cs
public Option<UserProfile> GetUserProfile(int userId)
{
    var user = database.FindUserById(userId);
    return new Option<UserProfile>(user);
}
```

使用`Option`的另一种方法是编写以下内容：

```cs
public Option<UserProfile> GetUserProfile(int userId)
{
    var user = database.FindUserById(userId);
    return user is not null
   ? Option<UserProfile>.Some(user)
   : Option<UserProfile>.None;
}
```

虽然这段代码更直接，但之前的代码更简洁。

现在，当调用此函数时，我们可以以更表达的方式处理结果：

```cs
var profileOption = GetUserProfile(userId);
UserProfile profile;
if (!profileOption.IsSome(out profile))
{
     // Handle the scenario when the profile is not available
}
// Continue with profile operations
```

这种方法使处理潜在缺失的值变得明确和清晰。

当史蒂夫深入研究`Option`类型时，他想“这让我想起了我们在游戏中处理升级的方式，有时它们在那里，有时它们不在。”

## `Option`类型相对于可空类型的优势

当涉及到处理可能缺失的值时，选择`Option`类型在你的代码中具有明显的优势：

+   `Some`和`None`构造使得代码的意图一目了然。有值和无值之间有一个清晰的区分。

+   `Option`类型强制你处理`Some`和`None`两种情况，减少了代码中的潜在疏忽。

+   `Option`类型通常与其他函数式方法一起工作，实现强大且简洁的转换。

## `Option`和可空类型之间的相互作用

值得注意的是，虽然`Option`类型提供了一个强大的机制来管理可选值，但它并不完全取代可空类型。相反，将它们视为你的工具箱中的工具，每个都有其优势：

+   当使用原生 C#构造或与期望它们的库/框架交互时，请使用可空引用类型

+   在需要更丰富函数操作或构建具有函数式风格的库的场景中采用`Option`类型

`Option` 类型为 C#世界带来了纯函数式编程的滋味，提供了一套复杂的工具集来处理可选值。通过将这些结构集成到您的应用程序中，您可以提高代码的清晰度、稳健性和表现力。虽然学习曲线可能比可空类型更陡峭，但就代码质量和弹性而言，这些回报是值得努力的。

# 实际场景 – 有效处理空值

正如您可能已经看到的，处理空值不仅仅是一个理论上的问题；它是一个日常挑战。通过分析现实世界的情况，我们可以更好地理解在管理空值和确保稳健、可靠的应用程序中明确策略的必要性。

## 案例研究 – 管理 YouTube 视频

**场景**: 一个系统从数据库检索视频详细信息。并非每个查询的视频 ID 都会有一个相应的视频条目。

**传统方法**:

```cs
public Video GetVideoDetails(int videoId)
{
    var video = database.FindVideoById(videoId);
    if (video == null)
    {
        throw new VideoNotFoundException($"Video with ID {videoId} not found.");
    }
    return video;
}
```

**选项方法**:

```cs
public Option<Video> GetVideoDetails(int videoId)
{
    var video = database.GetVideoById(videoId);
    return new Option<Video>(video);
}
```

通过返回`Video?`，我们明确地表示视频可能不存在。调用函数可以使用模式匹配或直接空值检查来优雅地处理缺失情况。

## 案例研究 – 管理不同视频类型

**场景**: 随着 YouTube 开始支持各种视频格式和来源（例如，直播、360 度视频、标准上传），后端系统必须正确识别和处理每种视频类型。随着平台的日益复杂和新的视频格式被引入，使用传统的 if-else 语句来处理这些情况变得越来越繁琐且难以维护。模式匹配成为解决这一挑战的有效解决方案。

**传统方法**:

```cs
public string GetVideoDetails(Video video)
{
    if (video is LiveStream)
    {
        var liveStream = video as LiveStream;
        return $"Live Stream titled '{liveStream.Title}' is currently {liveStream.Status}.";
    }
    else if (video is Video360)
    {
        var video360 = video as Video360;
        return $"360-Degree Video titled '{video360.Title}' with a resolution of {video360.Resolution}.";
    }
    // ... and so on for other video types
    else
    {
        return "Unknown video type.";
    }
}
```

**模式匹配方法**:

```cs
public string GetVideoDetails(Video video)
{
    return video switch
    {
        LiveStream l => $"Live Stream titled '{l.Title}' is currently {l.Status}.",
        Video360 v => $"360-Degree Video titled '{v.Title}' with a resolution of {v.Resolution}.",
        StandardUpload s => $"Standard video titled '{s.Title}' uploaded on {s.UploadDate}.",
        _ => "Unknown video type."
    };
}
```

模式匹配提供了一种更优雅、简洁和可读的方法来处理不同的视频类型。随着 YouTube 引入新的视频格式或功能，它们可以无缝集成到`GetVideoDetails`函数中。这种现代方法减少了潜在的错误，简化了代码，并提高了可维护性。

## 案例研究 – 处理不存在的对象

**场景**: 随着 YouTube 在全球范围内的扩张，由于各种原因（如网络故障、数据迁移问题或区域内容限制），遇到不完整或缺失数据的情况越来越普遍。使用传统的空值检查变得越来越繁琐，导致代码库中逻辑分散。空对象模式提供了一种系统性的方法，通过提供一个默认对象而不是空引用来解决问题。

**传统方法**:

```cs
public Video GetVideo(int videoId)
{
    var video = database.FindVideoById(videoId);
    if (video == null)
    {
        throw new VideoNotFoundException($"Video with ID {videoId} not found.");
    }
    return video;
}
```

当显示视频时：

```cs
var video = GetVideo(videoId);
if(video != null)
{
    Display(video);
}
else
{
    ShowError("Video not found");
}
```

**空对象模式方法**:

首先，为视频创建一个默认对象：

```cs
public class NullVideo : Video
{
    public override string Title => "Video not available";
    public override string Description => "This video is currently not available.";
    // Other default properties or methods...
}
```

然后修改获取方法：

```cs
public Video GetVideo(int videoId)
{
    return database.FindVideoById(videoId) ?? new NullVideo();
}
```

然后显示视频：

```cs
var video = GetVideo(someId);
Display(video); // No special null handling here
```

通过采用空对象模式，YouTube 的视频管理系统成功地将与空或缺失数据相关的行为封装在默认的“空对象”中，从而实现了更统一和可预测的系统行为。它消除了代码库中散布的许多空值检查，减少了空引用异常的可能性，并增强了系统的健壮性和可维护性。

## 在现实世界场景中处理空值的影响

在现实生活中的情况下，我们如何处理代码中的空值可能会有很大的影响。让我们探讨一下处理空值如何产生差异：

+   `NullReferenceException`

+   **更清晰的意图**：使用诸如可空引用类型和模式匹配等工具，我们的代码更透明地传达潜在的结果

+   **开发者信心**：当系统的行为可预测时，开发者可以更有信心地进行集成和扩展

在现实世界的场景中，未知和不确定性很多。通过引入对潜在空值的清晰和透明处理，我们确保我们的应用程序既具有弹性又易于维护。无论您是在管理视频数据、处理表单还是与第三方服务集成，对空值管理的深思熟虑的方法都可以产生重大差异。

# C# 中诚实的现实——为什么永远不会真正有诚实的函数

C# 是一种多范式语言，提供了许多功能，每个功能都是考虑到各种因素而设计的。当我们探讨“诚实函数”这一主题时，我们也必须承认，尽管 C# 的设计非常强大，但也存在一些权衡。

# C# 语言设计的妥协

C# 的设计涉及某些权衡。让我们来检查它们并了解它们的影响：

+   **历史包袱**：C# 在这些年中不断发展，添加了新功能同时确保向后兼容。这意味着较老、不那么“诚实”的做法将始终是语言的一部分。

+   **性能与安全性的权衡**：提供低级性能导向功能和高级安全功能并不容易。有时，性能的要求可能会让开发者偏离纯粹的“诚实”结构。

+   **广泛的受众**：C# 是为广泛的开发者设计的，从编写系统级代码的人到高级企业应用程序的开发者。因此，语言不能过于偏袒任何单一范式，包括函数式编程。

+   `OutOfMemoryException`，例如。虽然，我们可能会认为如果我们是代码的创造者，它就会按照我们的意愿行事，但别忘了**公共语言运行时**（**CLR**）才是真正的掌控者，它可以干预我们的意图。

C#提供了丰富的功能，使开发者能够在不同的范式下构建解决方案。在我们追求诚实和清晰的过程中，我们应该认识到并尊重语言设计内在的权衡。此外，我们需要记住，我们的代码并不在一个理想的世界中运行，环境也会影响我们的程序的工作方式。

# 实用技巧和最佳实践

在我们探索 C#中函数式编程的细微差别及其处理 null 的方法时，一些实用的策略浮现出来。这些策略确保我们的应用程序保持稳健，同时受益于函数范式提供的增强清晰度和可预测性。

## 将现有代码库迁移以采用可空引用类型和 Option 的策略

让我们考虑将现有代码库迁移以采用可空引用类型和`Option`的策略：

+   `#nullable` `enable` 指令：

    ```cs
    #nullable enable
    // The section of the code where nullable reference types are enabled.
    #nullable restore
    ```

+   **使用分析工具**：如 Roslyn 分析器（[`docs.microsoft.com/en-us/visualstudio/code-quality/roslyn-analyzers-overview`](https://docs.microsoft.com/en-us/visualstudio/code-quality/roslyn-analyzers-overview)）等工具可以帮助识别代码中的潜在 nullability 问题。

+   `Option`类型，确保你理解调用代码的影响。函数可能返回不同的类型，需要调整调用逻辑。

## 常见陷阱及其规避方法

让我们更深入地看看常见的陷阱，并了解如何有效地规避它们：

+   **过早假设非 null 值**：即使启用了可空引用类型，也始终要验证输入，尤其是如果它们来自外部来源：

    ```cs
    public void ProcessData(string? data)
    {
        if (data is null)
        {
            throw new ArgumentNullException(nameof(data));
        }
        // Rest of the processing...
    }
    ```

+   `Option`类型功能强大，可能并不适合每个场景。对于 nullability 显而易见的简单情况，可空引用类型可能更为合适。

+   **忘记遗留代码**：代码库中的旧部分可能不符合新的范式。在整合新旧代码时，要小心潜在的期望不匹配。

## 测试策略：处理 null 和 Option

让我们讨论如何以函数式编程的方式测试处理 null 和`Option`的代码。确保代码正确工作非常重要，现在我们将探讨如何做到这一点：

+   **使用 null 值的单元测试**：确保单元测试覆盖将 null 值传递给函数的场景。这有助于在这些问题到达生产环境之前捕捉潜在的 null 相关问题：

    ```cs
    [Test]
    public void GetUser_NullInput_ThrowsException()
    {
        // arrange part of the test
        // act & assert
        Assert.Throws<ArgumentNullException>(() => sut.GetUser(null));
    }
    ```

+   `Option`类型，测试应涵盖`Some`和`None`场景，以确保所有代码路径都得到验证：

    ```cs
    [Test]
    public void GetUser_PresetUserId_ReturnsProfile()
    {
        // arrange part of the test
        // act
        var result = sut.GetUser(123);
        // assert
        User user;
        if (!result.IsSome(out user))
        {
            Assert.Fail("Expected a user profile.");
        }
        // The rest of the assertions
    }
    ```

+   **集成测试**：除了单元测试之外，还应该使用集成测试来验证不同组件之间的交互，尤其是在处理可能返回意外 null 值的数据库、API 或其他外部系统时。

在 C# 中学习函数式编程可能不是一件容易的事情，但你已经开始了这个旅程，并一直持续到现在的代码行。你做得很好，所以继续保持，让我们通过我们传统的三个练习来巩固你获得的知识。

# 练习

现在，史蒂夫已经了解了诚实函数、空值和 Option 类型，朱莉娅已经准备了一些挑战，帮助他将这些概念应用到他的塔防游戏中。让我们看看你是否能帮助史蒂夫解决这些问题！

## 练习 1

史蒂夫的游戏需要可靠地获取塔信息。重构这个函数以使用诚实的返回类型，明确指示塔可能找不到的情况：

```cs
public Tower GetTowerByPosition(Vector2 position)
{
     var tower = _gameMap.FindTowerAt(position);
     return tower;
}
```

## 练习 2

在游戏中，玩家可以将增强道具应用到塔上。重构这个函数以确保它能够优雅地处理空输入：

```cs
public void ApplyPowerUp(Tower tower, PowerUp powerUp)
{
     tower.ApplyPowerUp(powerUp);
     _gameState.UpdateTower(tower);
}
```

## 练习 3

史蒂夫希望向玩家提供他们面对的敌人的详细信息。使用他游戏中的敌人类，实现一个函数，为每种敌人类型生成描述性字符串：

```cs
public abstract class Enemy {}
public class Goblin : Enemy
{
     public int Strength { get; set; }
     public bool HasWeapon { get; set; }
}
public class Dragon : Enemy
{
     public int FireBreathDamage { get; set; }
     public int WingSpan { get; set; }
}
public class Wizard : Enemy
{
     public string[] Spells { get; set; }
     public int MagicPower { get; set; }
}
public string DescribeEnemy(Enemy? enemy)
{
     // Your implementation here
}
```

# 解决方案

## 练习 1

将 `Option` 返回类型纳入，以表示用户可能找到或不找到：

```cs
public Option<Tower> GetTowerByPosition(Vector2 position)
{
     var tower = _gameMap.FindTowerAt(position);
     return Option<Tower>.Some(tower);
}
```

## 练习 2

利用内置的空输入检查，如果用户配置文件为 `null`，则抛出异常：

```cs
public void ApplyPowerUp(Tower? tower, PowerUp? powerUp)
{
     ArgumentNullException.ThrowIfNull(tower, nameof(tower));
     ArgumentNullException.ThrowIfNull(powerUp, nameof(powerUp));
     tower.ApplyPowerUp(powerUp);
     _gameState.UpdateTower(tower);
}
```

## 练习 3

使用模式匹配优雅地处理潜在的空值：

```cs
public string DescribeEnemy(Enemy? enemy)
{
     return enemy switch
     {
                  Goblin g => $"A goblin with {g.Strength} strength, {(g.HasWeapon ? "armed" : "unarmed")}.",
                  Dragon d => $"A dragon with {d.FireBreathDamage} fire breath damage and a {d.WingSpan}m wingspan.",
                  Wizard w => $"A wizard with {w.MagicPower} magic power, knowing {w.Spells.Length} spells.",
                  null => "No enemy in sight.",
                  _ => "An unknown enemy approaches!"
     };
}
```

这些练习及其解决方案提供了对所涵盖概念的应用理解，指导你走向处理空值和诚实函数的实用且健壮的方法。继续实验，不断迭代，并始终遵循函数式编程的原则，以编写更清晰、更具弹性的 C# 代码。

# 摘要

随着我们通过诚实函数和 C# 中空值处理的复杂性的旅程即将结束，让我们反思我们的发现并展望未来。

我们已经探讨了 C# 中 `null` 的历史和影响。我们已经理解了它的细微差别、它的危险和它的力量。到现在，臭名昭著的 `NullReferenceException` 应该不再是你的敌人，而是一个你在房间另一边点头致意的老熟人，承认它的存在，但永远不会让它打扰你的日常生活。

诚实函数——或者明确声明其意图、输入和输出的函数——代表了向可预测性、清晰性和弹性的范式转变。在函数中拥抱诚实不仅仅是为了避免陷阱，更是为了拥抱一种透明度的哲学。通过这样做，我们创建的代码可以让其他开发者信任、理解和在此基础上构建。

我们已经深入研究了可空引用类型和模式匹配的领域，甚至触及了空对象模式和 `Option` 类型，所有这些都为我们提供了强大的工具，以精确和优雅地表达我们的意图。

然而，就像所有工具一样，我们必须记住，它们的强大之处在于它们的适当应用。C#的世界广阔无垠，虽然函数式编程原则提供了很多价值，但它们只是丰富多彩的画卷中的一个方面。选择何时应用这些概念，何时转向其他范式，取决于你，作为开发者。当我们把函数式范式添加到我们的习惯性编码方式中时，可能会引发不同的错误，我们最好准备好与他们一起工作。这就是为什么我邀请你阅读下一章，关于错误处理。

# 第二部分：高级函数式技术

在第一部分建立的基础之上，我们现在深入探讨更高级的函数式编程技术。我们将从探索错误处理的函数式方法开始，超越传统的 try-catch 块，到更优雅的解决方案。接下来，我们将介绍高阶函数和委托，解锁函数作为一等公民的力量。本节以对 functors 和 monads 的深入探讨结束，这些高级概念为管理代码中的复杂性提供了强大的工具。

本部分包含以下章节：

+   *第五章**，错误处理*

+   *第六章**，高阶函数和委托*

+   *第七章**，Functors 和 Monads*
