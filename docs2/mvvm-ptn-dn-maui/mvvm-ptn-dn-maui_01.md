

# 什么是 MVVM 设计模式？

**MVVM**或**模型-视图-视图模型**模式是.NET 生态系统中非常常用的设计模式，其中它已被证明非常适合使用**XAML**构建图形用户界面的前端框架。这一点并不难理解。

本书将为您提供对 MVVM 设计模式及其在**.NET MAUI**项目中有效应用的良好理解。需要注意的是，虽然我们将专注于在.NET MAUI 的上下文中应用 MVVM，但 MVVM 模式本身并不仅限于.NET 生态系统。它是一个广泛使用的设计模式，在包括 WPF、WinUI 等在内的各种软件开发生态系统中获得了流行。我们将深入研究.NET MAUI 的各个方面，以了解它如何支持并启用 MVVM 的使用。在整个书中，我们将构建“*食谱!*”应用作为实际示例，展示在.NET MAUI 中应用 MVVM 模式的各个方面。

在本章中，我们将学习 MVVM 设计模式及其核心组件。我们将探讨该模式在关注点分离方面带来的附加价值以及为什么这如此重要。一个示例应用将展示 MVVM 可以为您的代码带来的附加价值。最后，我们将讨论一些关于 MVVM 的常见误解。

到本章结束时，您将了解 MVVM 是什么，其主要组件是什么，以及每个组件的作用。您还将看到 MVVM 在提高代码的可测试性和可维护性方面带来的价值。

在本章中，我们将涵盖以下主要内容：

+   MVVM 的核心组件

+   关注点的分离很重要

+   MVVM 的实际应用

+   关于 MVVM 的常见误解

在深入探讨.NET MAUI 中 MVVM 的细节之前，熟悉 MVVM 模式的核心组件至关重要。在接下来的部分中，我们将定义关键术语，以确保我们有一个坚实的基础和共同的了解，以便继续前进。

# 技术要求

虽然本章提供了 MVVM 的理论概述，但稍后会有一些代码展示 MVVM 的实际应用，以便您开始看到这种模式带来的价值。要自己实现此示例，您需要以下条件：*Visual Studio 2022（17.3 或更高版本），或任何允许您创建.NET MAUI 应用的 IDE*。在*第二章**，什么是.NET MAUI？*的末尾，我们将探讨如何为开发.NET MAUI 应用准备您的机器。示例代码也可以在 GitHub 上找到：[`github.com/PacktPublishing/MVVM-pattern-.NET-MAUI`](https://github.com/PacktPublishing/MVVM-pattern-.NET-MAUI)。

# 查看 MVVM 的核心组件

MVVM 提供了一种非常清晰的方式来分离 UI 和业务逻辑，促进代码的可重用性、可维护性和可测试性，同时允许灵活的 UI 设计和更改。

随着应用程序规模和复杂性的增长，将业务逻辑放在代码背后（code-behind）很快就会变得具有挑战性。代码背后指的是将业务逻辑放置在与用户界面元素相同的文件中的做法，这通常会导致大量代码通过事件处理器被调用。这通常会导致 UI 元素和底层业务逻辑之间的紧密耦合，因为 UI 元素在代码中被直接引用和操作。因此，调整 UI 和执行单元测试可能会变得更加困难。在本章的“MVVM 实战”部分，我们将看到在代码背后放置业务逻辑意味着什么，以及它如何复杂化维护性、测试等。

初看起来，MVVM 可能有点令人不知所措或困惑。为了完全理解这个模式是什么以及为什么它如此受 XAML 开发者的欢迎，我们首先需要剖析 MVVM 模式并查看其核心组件。

模式的名称已经揭示了三个基本的核心组件：**模型**、**视图**和**视图模型**。还有两个对于高效使用 MVVM 至关重要的辅助元素：**命令**和**数据绑定**。这些元素中的每一个都有自己的独特角色和责任。

以下图表显示了 MVVM 的核心组件以及它们之间是如何相互作用的：

![图 1.1 – MVVM 的核心组件](img/B20941_01_01.jpg)

图 1.1 – MVVM 的核心组件

如果你想要有效地使用 MVVM 模式，重要的是你不仅要了解每个组件的责任，还要了解它们之间是如何相互作用的。在较高层次上，视图（View）了解视图模型（ViewModel），而视图模型（ViewModel）了解模型（Model），但模型（Model）对视图模型（ViewModel）并不知情。这种分离防止了 UI 和业务逻辑之间的紧密耦合。

为了有效地使用 MVVM 模式，重要的是要了解如何将应用程序代码组织成合适的类，以及这些类是如何相互作用的。现在，让我们来看看核心组件。

## 模型

模型负责表示应用程序的数据和业务逻辑。它封装了数据并提供了一种操作它的方法。模型可以与应用程序内的其他组件通信，例如数据库或 Web 服务。

它通常使用代表应用程序领域对象的类来实现，例如客户、订单或产品。这些类通常包含表示对象属性的属性和定义对象行为的方法。

模型被设计成与应用程序中使用的 UI 框架独立。因此，如果需要，它可以在其他应用程序中重用。它可以独立于应用程序的 UI 进行维护和测试。

注意

对于刚开始接触 MVVM 的开发者来说，往往不清楚模型可以是或应该是哪种类型的对象：**数据传输对象**（**DTOs**）、**普通 CLR 对象**（**POCOs**）、领域实体、代理对象、服务等等。所有这些类型的对象都可以被视为模型。

此外，在大多数情况下，模型（Model）是不同类型对象的组合。不要将模型视为单一的事物。实际上，它包括视图和 ViewModel 以外的所有“外部”内容——它是应用程序的领域和业务逻辑。最终，模型类型的选取将取决于应用程序的具体需求、用例和架构。一些开发者可能更喜欢使用 DTOs（数据传输对象）以实现简单易用，而其他人可能更喜欢使用领域实体以实现更好的封装和与领域模型的保持一致。

然而，无论使用的是哪种具体的模型类型，都重要的是将业务逻辑保持在模型“层”内，并在 ViewModel 中避免它。这有助于保持关注点的分离，并使 ViewModel 专注于展示逻辑，而模型则专注于业务逻辑。

## 视图（View）

视图负责向用户展示数据。它由（UI）元素组成，例如按钮、标签、集合和输入。视图接收来自用户的输入或操作，并与 ViewModel 通信，以便它可以对这些交互做出反应。

在 .NET MAUI 中，视图通常使用 XAML 实现，这是一种用于定义 UI 元素和布局的标记语言。XAML 文件定义了 UI 的结构及其与 ViewModel 的绑定。需要注意的是，尽管 MVVM 模式通常与基于 XAML 的 UI 相关联，但其原则并不局限于特定的 UI 框架。MVVM 模式也可以应用于其他 UI 框架。在 .NET MAUI 中，大多数应用程序使用 XAML 定义 UI，但也可以完全使用 C# 创建 UI。即使采用这种方法，开发者仍然可以有效地应用 MVVM 模式来分离视图、ViewModel 和模型的关注点。

视图最重要的方面是它应该尽可能简单，没有任何业务规则或逻辑。其主要目的是向用户展示数据并接收他们的输入。视图的焦点应放在展示逻辑上，包括格式化和布局，并且不应包含任何业务逻辑或应用程序逻辑，以在 MVVM 中保持适当的关注点分离。

小贴士

通过保持视图简单并专注于展示逻辑，代码可以更容易维护和测试。它还使得在不影响底层业务逻辑的情况下更改 UI 成为可能，从而在适应和随时间演变应用程序方面提供了更大的灵活性。

视图（The View），无论它是使用 XAML 还是 C# 实现的，都是应用程序的用户界面层。本质上，它是构成用户界面的 UI 组件集合，例如页面和控制元素。

## ViewModel

ViewModel 位于 Model 和 View 之间。它是 UI 和业务逻辑之间的“粘合剂”。其职责是向 View 暴露数据以供显示，并处理任何用户输入或 UI 事件。它为 View 提供了一个与包含应用程序实际数据和业务逻辑的 Model 交互的接口。让我们逐步深入了解 ViewModel 的功能：

1.  **暴露数据**：ViewModel 从 Model 中检索数据并通过公共属性将其暴露给 View。这些属性通常绑定到 View 中的元素，例如文本框或标签。这样，Model 中的数据就会显示在屏幕上。

1.  **反映更改**：现在，你可能想知道当用户在屏幕上更改某些内容时会发生什么。这就是数据绑定发挥作用的地方。数据绑定就像 View 和 ViewModel 之间的一条实时通信通道。当用户在 View 中修改数据时，ViewModel 会立即得到通知并相应地更新其属性。同样，如果 ViewModel 中发生更改，View 也会得到更新。

1.  **处理用户交互**：ViewModel 不仅被动地提供数据；它还负责处理用户交互，例如按钮点击和文本输入。例如，如果 View 中有一个 **保存** 按钮，ViewModel 需要知道当用户点击它时应该做什么。

1.  **更新 Model**：当用户与 UI 交互并做出更改时，ViewModel 在更新 Model 中发挥着至关重要的作用。ViewModel 接收用户的更改并将它们转换为 Model 可以理解和处理的行为。

ViewModel 充当中间人，处理 View 和 Model 之间数据和行为的流动，确保用户界面准确反映底层数据并对用户交互做出正确响应。这种模块化方法，即它将特定责任与 View 和 Model 分开处理，有助于编写更干净、更易于维护的代码。最后，重要的是要理解 ViewModel 不应与任何特定的 UI 框架绑定。独立于 UI 框架允许它在不同的应用程序之间共享，并便于从应用程序的其他部分独立进行测试。

## 命令

命令是用于表示可以由 UI 触发的行为的重要概念。它是 View 将用户操作传达给 ViewModel 的方式。命令是一个实现 `ICommand` 接口的对象，该接口定义了两个方法：`Execute` 和 `CanExecute`。当命令被触发时调用 `Execute` 方法，而 `CanExecute` 方法用于确定命令在其当前状态下是否可以执行。

命令可以看作是传统事件驱动编程中事件的等价物。命令和事件都作为处理应用程序中用户操作或其他触发器的机制。例如，当用户在视图中点击按钮时，ViewModel 中的命令被触发，ViewModel 采取适当的行动，例如保存数据或获取新信息。命令非常灵活，不仅限于按钮。它们可以与各种 UI 元素相关联，如菜单项、工具栏按钮，甚至是手势控制。让我们举一个实际例子：假设你有一个文本编辑应用程序。你可以有一个与`Execute`方法关联的命令，当调用该方法时执行必要的操作。

在 MVVM 模式中使用命令的一个关键好处是，它们允许视图和 ViewModel 之间更好地分离关注点。通过使用命令，视图可以设计而无需了解与特定 UI 元素相关的底层功能。这意味着 ViewModel 可以处理与命令相关的所有逻辑，而无需与 UI 紧密耦合。

为了将命令绑定到视图中的 UI 元素，命令需要由 ViewModel 作为公共属性公开。然后视图可以使用数据绑定绑定到这个属性。当命令绑定到视图中的 UI 元素时，该 UI 元素会监听事件，例如按钮点击。当该事件被触发时，包含响应事件的代码的命令的`Execute`方法被调用。

总体而言，命令是 MVVM 模式中一个强大且灵活的概念，它使得视图和 ViewModel 之间的关注点分离更加有效。

关于命令的更多信息

我们将在*第三章*中深入了解命令及其在实际中的应用，即**.NET MAUI**中的数据绑定构建块。

## 数据绑定

数据绑定是 MVVM 的核心特性，它允许 ViewModel 以松耦合的方式与视图通信，以及视图与 ViewModel 之间的通信。数据绑定允许您将 ViewModel 中的数据属性绑定到视图中的 UI 元素，例如输入字段、标签和列表视图。它用于同步视图和 ViewModel 之间的数据。当 ViewModel 中的数据发生变化时，数据绑定引擎会更新视图，反之亦然，具体取决于绑定的配置方式。这允许 UI 反映应用程序的当前状态。

绑定过程涉及三个组件：一个**源**对象（ViewModel），一个**目标**对象（UI 元素），以及一个**绑定表达式**，该表达式指定了两个对象应该如何连接。

数据可以以不同的方向流动：从 ViewModel 到视图，反之亦然，或者两者都流动。以下是数据流动的方式：

+   **单向**: 从视图模型到视图，这允许视图模型上属性的值在视图中显示。这种类型的绑定通常用于当视图模型的更新应自动更新视图上的值时。

+   **单向到源**: 与单向相反，数据只从视图流向视图模型。在视图中输入的值将自动反映在视图模型上。

+   **一次性**: 与单向类似，数据从视图模型流向视图，但数据绑定引擎将不会监听绑定属性上发生的任何更改。一旦初始值在 UI 上显示，该属性的任何后续更改都不会反映在视图中。这在进行大量数据绑定时可以产生显著的正面性能影响。

+   **双向**: 数据以两种方式流动；从视图到视图模型，以及从视图模型到视图。对 UI 中数据的更改会自动传播回视图模型。使用双向数据绑定的常见场景是在视图中显示用户可以修改的属性。在这种情况下，该属性通常使用双向数据绑定绑定到一个输入字段。这允许属性的初始值在输入字段中显示，并且用户所做的任何更改都会自动反映到视图模型，无需额外的事件处理或手动更新。

数据绑定是一个非常强大的概念，是 MVVM 模式中的一个基本组件。它允许视图模型以简单的属性形式向视图公开数据和行为，这种方式与 UI 框架无关。通过数据绑定，这些属性可以以松耦合的方式绑定到 UI。通过使用数据绑定，MVVM 中的视图和视图模型可以无缝同步，无需任何手动代码。绑定模式决定了数据如何在视图和视图模型之间传播。此外，数据绑定还允许视图通过命令将用户输入回传到视图模型，然后视图模型可以更新模型。

更多关于数据绑定的信息

在*第三章**数据绑定构建块在.NET MAUI 中*和*第四章**.NET MAUI 中的数据绑定*中，详细介绍了关于数据绑定以及如何在.NET MAUI 中有效使用它的所有必要信息。

现在我们已经对 MVVM 的核心组件有了很好的理解，并对每个组件的责任有了深入了解，让我们更详细地讨论一下为什么 MVVM 很重要以及它增加了什么价值。

# 关注点分离很重要

**关注点分离**是软件开发中的一个重要原则，旨在将软件设计和实现划分为具有特定和明确责任的不同部分。这个原则通过降低每个组件的复杂性，允许更模块化和可重用的代码，帮助开发者创建更可维护和灵活的应用程序。

在实际应用中，关注点分离意味着将系统的不同方面分离并独立处理，不重叠关注点。这是通过创建具有各自明确责任和接口的独立层或模块，并最小化它们之间的依赖关系来实现的。

让我们以一家餐厅的管理系统为例。在这样的系统中，可能会有几个关注点，如餐桌预订、点餐、厨房操作和结账。根据关注点分离的原则，这些领域中的每一个都应该由一个独立的模块来处理。

每个模块都有自己的责任集，并且会通过定义良好的接口与其他模块进行通信。例如，当顾客点菜时，点餐模块会与厨房操作模块通信，以便准备菜品。

这种分离意味着，如果您需要更改餐桌预订的工作方式，您可以在不影响厨房操作或结账的情况下进行更改。这种分离使代码库更加有序，更容易维护，并允许对单个模块进行集中的测试。

让我们探讨一下关注点分离如何提高可维护性和可测试性。这两个方面对于任何软件应用的长期成功至关重要。

## 可维护性

通过应用 MVVM，我们正在将 UI 与业务逻辑分离，并将 ViewModel 与视图松散耦合。这极大地提高了可维护性，因为每个组件都有其独特的角色和关注点。

例如，视图可以很容易地更改或更新。UI 元素可以移动或替换为更现代的元素，而无需更改 ViewModel，只要不需要显示额外的数据。这是因为视图和 ViewModel 之间唯一的接口是通过数据绑定。因此，更新视图不应影响 ViewModel，除非需要显示额外的或更改后的数据。

此外，当视图需要显示额外的数据或 UI 元素更新需要进一步的数据时，ViewModel 可能也需要更新。然而，如果所需数据已经在 Model 中可用，ViewModel 可以在 Model 和视图之间进行转换，而不会影响 Model 本身。ViewModel 负责管理 Model 和视图之间的数据流，确保应用程序不同层之间有清晰的关注点分离。

此外，Model 业务逻辑的任何更改都不应对 ViewModel 或 View 产生重大影响。由于 ViewModel 充当 Model 和 View 之间的中介，它负责将 Model 中的数据转换为 View 使用。因此，如果任何底层业务逻辑应该发生变化，ViewModel 可以处理转换而不影响 View，确保对 Model 所做的任何更改对用户都是透明的。

即使需要更新 Model 或 ViewModel，由于 MVVM 模式的特点，这些更改可以轻松地相互独立测试，最重要的是，独立于 UI。这种分离使得测试既高效又专注。

## 测试性

测试性是软件开发中一个非常重要的因素。不仅测试确保测试的代码执行其预期的功能，而且还保证它继续按照最初的设计运行。当运行一组成功的测试时，如果在代码更新时不小心破坏了功能，则会立即提供反馈。这对于维护代码库的质量和稳定性无疑是至关重要的。本质上，全面的测试在保持代码的可维护性方面发挥着关键作用，允许随着时间的推移进行高效的修改和改进。

注意

通过在设计应用程序时考虑测试性，开发者可以创建更可靠、可维护和可扩展的代码。

关注点的分离对于实现测试性至关重要，因为它允许开发者隔离和测试应用程序的各个独立组件，而无需担心对系统其他部分的依赖。

从 MVVM 设计模式的视角来看，关注点的分离和测试性密切相关。由于 View、ViewModel 和 Model 被视为不同的组件，每个组件都能够独立于其他组件编写和测试。因为 ViewModel 与 View 解耦，并且对任何特定的 UI 框架都是不可知的，因此很容易为其编写自动单元测试。这些自动测试比手动或自动 UI 测试更快、更可靠，这允许开发者尽早在开发过程中识别错误和回归，在它们变得难以修复和昂贵之前。同样，Model 逻辑也可以在独立于 ViewModel 和 View 的情况下进行有效测试。

这就是理论。现在，让我们看看一个 MVVM 示例！

# MVVM 实践

让我们看看一个非常简单的应用程序，它会在屏幕上显示用户的每日名言。当用户点击屏幕上的按钮时，应用程序将从 API 获取每日名言并显示在屏幕上。一旦按钮被点击，它应该被隐藏。代码-behind 方法将向您展示如何在不使用 MVVM 模式的情况下编写此应用程序，而第二个示例则展示了使用 MVVM 实现相同功能，并考虑了可测试性。

## 代码-behind 方法

在下面的代码片段中，没有关注点的分离；所有的代码都在 XAML 页面的代码-behind 中处理。虽然这段代码看起来似乎在执行预期的操作，但没有简单、快速或健壮的方法来测试它是否工作。这是因为所有的逻辑都在按钮的点击事件处理器中处理。

在`MainPage.xaml`中，我们定义了一个`Button`和一个`Label`：

```cs
<Grid>
    <Button
        x:Name="GetQuoteButton"
        Clicked="Button_Clicked"
        Text="Get Quote of the day" />
    <Label
        x:Name="QuoteLabel"
        HorizontalOptions="Center"
        IsVisible="false"
        Style="{DynamicResource TitleStyle}"
        VerticalOptions="Center" />
</Grid>
```

两个按钮和标签控件都有一个`Name`，这样就可以从代码-behind 中引用它们。标签默认情况下是不可见的。

在代码-behind（`MainPage.xaml.cs`）中，我们按照以下方式处理按钮点击事件：

```cs
private async void Button_Clicked(
    object sender, EventArgs e)
{
    GetQuoteButton.IsVisible = false;
    try
    {
        var client = new HttpClient();
        var quote = await
            client.GetStringAsync(
            "https://my-quotes-api.com/quote-of-the-day");
        QuoteLabel.Text = quote;
        QuoteLabel.IsVisible = true;
    }
    catch (Exception)
    {
    }
}
```

在按钮点击时，`GetQuoteButton`被隐藏，并调用获取名言。`QuoteLabel`的`Text`属性被分配了检索到的名言的值。

注意，这段代码中有一个微小的错误：如果 API 调用失败，抛出的异常会被静默捕获，但`GetQuoteButton`的可见性不会被恢复为可见，这会导致应用程序无法使用。但由于没有简单的方法来测试这段代码，这种情况很可能直到应用程序进入 QA 阶段才被发现，希望手动测试能够发现这个问题。

## 使用 MVVM

现在，让我们看看如何使用 MVVM 模式来转换这个应用程序，同时考虑到关注点的分离和可测试性。可能这个例子中的一些内容现在还不清楚，这是完全可以接受的。在阅读这本书的过程中，所有这些方面都应该会很快变得清晰。

让我们首先看看这个应用程序中的模型是什么。

### 模型

在我们的示例应用程序中，我们正在处理的主要数据是从 API 获取的名言。与这个 API 通信以获取数据的逻辑可以被视为模型。我们不再像之前那样直接在代码-behind 中发起 HTTP 请求，而是可以将这个逻辑封装到一个单独的类中。这个类将负责获取名言，并充当我们的模型。下面是这个类的可能样子：

```cs
public class QuoteService : IQuoteService
{
    private readonly HttpClient httpClient;
    public QuoteService()
    {
        httpClient = new HttpClient();
    }
    public async Task<string> GetQuote()
    {
        var response = await
        httpClient.GetAsync("https://my-quotes-api.com/
        quote-of-the-day");
        if (response.IsSuccessStatusCode)
        {
            return await
            response.Content.ReadAsStringAsync();
        }
        throw new Exception("Failed to retrieve quote.");
    }
}
public interface IQuoteService
{
    Task<string> GetQuote();
}
```

如您所见，我们创建了一个`QuoteService`类，其中包含从 API 获取名言的逻辑。这个类实现了一个名为`IQuoteService`的接口。通过定义一个接口，我们使得替换实现或为测试目的模拟此服务变得更加容易。

通过将之前在代码后部中存在的逻辑封装在一个专门的类中，这已经为我们提供了一个更清晰的关注点分离。

这涵盖了模型-视图-ViewModel 模式中的模型部分。让我们看看 ViewModel 可能的样子。

### ViewModel

ViewModel 充当视图和模型之间的中介。它持有视图将绑定到的数据和命令。对于我们的简单应用程序，我们需要在 ViewModel 中包含以下内容：

+   一个属性用于在检索到引言后持有每日引言

+   两个属性用于控制按钮和标签的可见性

+   一个在按钮被点击时将被触发的命令；这将负责检索引言

我们还需要有一个接受类型为`IQuoteService`的参数的构造函数。这就是依赖注入发挥作用的地方。

更多关于依赖注入的信息

依赖注入是使类可测试和模块化的关键，并且在 MVVM 中非常常用。*第七章**，依赖注入、服务和消息传递*，深入探讨了这一概念。

让我们看看这可能在代码中是什么样子：

```cs
public class MainPageViewModel : INotifyPropertyChanged
{
    private readonly IQuoteService quoteService;
    public MainPageViewModel(IQuoteService quoteService)
    {
        this.quoteService = quoteService;
    }
    public event PropertyChangedEventHandler
    PropertyChanged;
}
```

构造函数接受一个`IQuoteService`实例作为参数。这个实例被分配到类中的`quoteService`字段。这样，ViewModel 就可以访问引言检索服务，允许它在需要时检索引言。

还要注意，这个类实现了`INotifyPropertyChanged`接口，因此需要实现`PropertyChanged`事件。其主要目的是确保当 ViewModel 中的数据发生变化时，UI 会得到通知，这样视图和 ViewModel 就可以保持同步。*第三章**，.NET MAUI 中的数据绑定构建块*，对此进行了更深入的探讨！

`QuoteOfTheDay`是持有检索到的引言的属性。这只是一个简单的属性，持有字符串值。它“特殊”的地方在于它会触发`PropertyChanged`事件，以通知数据绑定引擎关于更新值的更新：

```cs
private string quoteOfTheDay;
public string QuoteOfTheDay
{
    get => quoteOfTheDay;
    set
    {
        quoteOfTheDay = value;
        PropertyChanged?.Invoke(this,
            new PropertyChangedEventArgs(
            nameof(QuoteOfTheDay)));
    }
}
```

控制按钮和标签可见性的两个属性分别是`IsButtonVisible`和`IsLabelVisible`。同样，这些也是简单的公共属性，会触发`PropertyChanged`事件：

```cs
private bool isButtonVisible = true;
public bool IsButtonVisible
{
    get => isButtonVisible;
    set
    {
        isButtonVisible = value;
        PropertyChanged?.Invoke(this,
            new PropertyChangedEventArgs(
            nameof(IsButtonVisible)));
    }
}
bool isLabelVisible;
public bool IsLabelVisible
{
    get => isLabelVisible;
    set
    {
        isLabelVisible = value;
        PropertyChanged?.Invoke(this,
            new PropertyChangedEventArgs(
            nameof(IsLabelVisible)));
    }
}
```

在我们实现当用户点击按钮时应调用的命令之前，让我们首先实现该命令需要执行的逻辑：

```cs
private async Task GetQuote()
{
    IsButtonVisible = false;
    try
    {
        var quote = await quoteService.GetQuote();
        QuoteOfTheDay = quote;
        IsLabelVisible = true;
    }
    catch (Exception)
    {
    }
}
```

`GetQuote`方法是非同步的，这意味着它允许非阻塞执行，这在从网络源获取数据时尤为重要。它首先将`IsButtonVisible`属性设置为`false`，这应该会隐藏屏幕上的按钮。接下来，我们调用`quoteService`字段的`GetQuote`方法，这将出去获取一个引言。这个服务实际上是如何获取引言的并不重要，因为这不是 ViewModel 的关注点。一旦我们从`quoteService`的`GetQuote`方法收到引言，我们就将这个值分配给`QuoteOfTheDay`属性。将`IsLabelVisible`属性设置为`true`，以便显示引言的标签变得可见。

最后，我们可以创建一个触发此方法的命令：

```cs
public ICommand GetQuoteCommand => new Command(async _ => await GetQuote());
```

通过`GetQuoteCommand`，我们现在可以调用 ViewModel 上定义的`GetQuote`方法。

在 Model 和 ViewModel 已经就绪的情况下，我们终于可以看看 View 了。

### View

实际上，对 UI 的更改并不多：我们仍然需要一个按钮来触发`QuoteOfTheDay`的检索，以及一个标签来显示它。由于我们不再从代码背后访问标签，因此不需要设置`x:Name`属性：

```cs
<Grid>
    <Button
        Text="Get Quote of the day" />
    <Label
        HorizontalOptions="Center"
        IsVisible="false"
        Style="{DynamicResource TitleStyle}"
        VerticalOptions="Center" />
</Grid>
```

此外，我们之前在代码背后使用的`Button_Clicked`事件处理器可以从代码中移除。而且当我们身处其中时，我们还可以将页面的`BindingContext`分配给`MainPageViewModel`的一个实例：

```cs
public partial class MainPage_MVVM : ContentPage
{
    public MainPage_MVVM()
    {
        InitializeComponent();
        BindingContext = new MainPageViewModel(
            new QuoteService());
    }
}
```

`BindingContext`基本上是我们将要绑定的源。有了这个，我们对 XAML 进行一些最后的调整，包括数据绑定语句：

```cs
<Grid>
    <Button
        Command="{Binding GetQuoteCommand}"
        IsVisible="{Binding IsButtonVisible}"
        Text="Get Quote of the day" />
    <Label
        HorizontalOptions="Center"
        IsVisible="{Binding IsLabelVisible}"
        Style="{DynamicResource TitleStyle}"
        Text="{Binding QuoteOfTheDay}"
        VerticalOptions="Center" />
</Grid>
```

这些绑定语句“链接”了我们 ViewModel 公开暴露的属性到 UI 元素的属性上。当用户点击按钮时，`MainPageViewModel`上的`GetQuoteCommand`将被调用，这反过来将执行`GetQuote`方法。在执行`GetQuote`方法的同时，`IsButtonVisible`和`IsLabelVisible`属性正在被更新，从`QuoteService`检索到的引言将被设置为`QuoteOfTheDay`属性的值。通过数据绑定，这些更改将立即自动反映在 View 上。

就这样！这基本上是我们之前拥有的相同的应用程序。然而，这次它是使用 MVVM 模式编写的，同时考虑到关注点的分离和可测试性。

立即引人注目的是，MVVM 示例中的代码更多。这主要是因为 ViewModel。幸运的是，ViewModel 应该相当简单。它们不应该包含任何业务逻辑。在这个例子中，业务逻辑位于`QuoteService`类中，ViewModel 会调用它来获取`QuoteOfTheDay`值。ViewModel 上的属性用于表示 View 的状态，例如控制按钮和标签的可见性，以及保存`QuoteService`将返回的`Quote`。

到现在为止，应该很清楚，这个例子中的每个部分都有自己的职责：

+   视图层由 `MainPage_MVVM` 负责。它包含视觉元素，如 `Label` 和 `Button`，并将它们布局。

+   在此场景中，`QuoteService` 的单一职责是获取一个 `Quote`。

+   `MainPageViewModel` 将所有这些粘合在一起。它提供视图所需的属性和值，以及任何用于处理交互的命令。

每个组件只有一个改变的理由，这使得代码比将所有内容放在代码后部更容易维护。与之前的示例相比，不仅可维护性提高了，而且可测试性也得到了很大提升。不要只听我的话；让我们看看我们如何测试我们应用程序的功能。

### 测试 ViewModel

最后，让我们快速看看这对可测试性意味着什么。以下代码示例展示了 `MainPageViewModel` 的一些单元测试。再次强调，这里可能不是所有内容都清楚，但本书将彻底涵盖所有内容。此外，*第十三章**，单元测试*，完全致力于编写 ViewModels 的单元测试。在这些测试中，我们使用 `IQuoteService` 接口。模拟是单元测试中用来创建一个模仿真实对象行为的假或模拟对象的技术。这对于隔离正在测试的代码和消除对外部元素（如数据库或 API）的依赖特别有用。在测试类构造函数中，它在每个测试之前运行，我们创建一个新的模拟实例，该实例的 `GetQuote` 方法返回一个空字符串：

```cs
private Mock<IQuoteService> quoteServiceMock;
public MainPageViewModelTests()
{
    quoteServiceMock = new Mock<IQuoteService>();
    quoteServiceMock.Setup(m => m.GetQuote())
      .ReturnsAsync(string.Empty);
}
```

这个模拟实例可以在创建 `MainPageViewModel` 类的新实例时作为参数传递。这允许我们在没有任何外部依赖的情况下测试 ViewModel。

第一个片段显示了两个测试，测试 `IsButtonVisible` 属性的值：

```cs
[Fact]
public void ButtonShouldBeVisible()
{
    var sut = new
        MainPageViewModel(quoteServiceMock.Object);
    Assert.True(sut.IsButtonVisible);
}
[Fact]
public void GetQuoteCommand_ShouldSetButtonInvisible()
{
    var sut = new
        MainPageViewModel(quoteServiceMock.Object);
    sut.GetQuoteCommand.Execute(null);
    Assert.False(sut.IsButtonVisible);
}
```

前面的第一个测试检查 `IsButtonVisible` 属性的初始值是否为 `true`。我们创建一个新的 `MainPageViewModel` 实例，传入模拟的 `IQuoteService` 实例。现在我们可以使用我们的 `sut` 变量（系统正在测试）来进行断言，看看一切是否按预期工作。第二个测试检查一旦调用 `GetQuoteCommand`，`IsButtonVisible` 属性就变为 `false`。

下一个测试检查由注入的 `IQuoteService` 的 `GetQuote` 方法返回的 `Quote` 值是否被设置为 `QuoteOfTheDay` 属性的值：

```cs
[Fact]
public void GetQuoteCommand_GotQuote_ShowQuote()
{
    var quote = "My quote of the day";
    quoteServiceMock.Setup(m =>
        m.GetQuote()).ReturnsAsync(quote);
    var sut = new
        MainPageViewModel(quoteServiceMock.Object);
    sut.GetQuoteCommand.Execute(null);
    Assert.Equal(quote, sut.QuoteOfTheDay);
}
```

在前面的测试中，我们定义了模拟的 `IQuoteSerivce` 的 `GetQuote` 方法应该返回特定的值。执行 `GetQuoteCommand` 后，`QuoteOfTheDay` 属性应该具有相同的值。

最后，我们有一个不测试应用程序的“快乐路径”的测试。相反，它测试在`quoteService`未能检索引言后，`IsButtonVisible`属性是否设置为`true`，使用户可以再次尝试：

```cs
[Fact]
public void GetQuoteCommand_ServiceThrows_ShouldShowButton()
{
    quoteServiceMock.Setup(m =>
        m.GetQuote()).ThrowsAsync(new Exception());
    var sut = new
        MainPageViewModel(quoteServiceMock.Object);
    sut.GetQuoteCommand.Execute(null);
    Assert.True(sut.IsButtonVisible);
}
```

这个测试失败了，揭示了实现中的一个问题：ViewModel 中的`GetQuote`方法静默地处理来自`IQuoteService`的任何异常，但未能重新启用按钮，使应用程序处于无用的状态。即使没有运行应用程序一次，不需要部署任何其他组件，也不需要依赖于每日引言 API 的可用性，应用程序的行为实际上正在被测试，一个简单的错误已经在开发过程的早期就被识别出来了。这些测试确保应用程序按预期运行，但同时也确保它在未来保持这种运行方式。如果代码的更改会引入不同的（意外的）行为，自动化测试将失败，通知开发者他们已经破坏了某些内容，在发布应用程序之前需要修复它。像这样的单元测试非常有价值，并且非常容易编写，只要存在清晰的关注点分离，并且应用程序是以可测试性为前提编写的。MVVM 模式对此非常完美！就像这个例子一样，ViewModel 可以在完全隔离的情况下进行测试，因为它没有绑定到视图或任何特定的 UI 框架。这个 ViewModel 可以在任何类型的.NET 应用程序中工作！

与具有代码背后实现的先前的例子相比，测试变得具有挑战性：必须部署应用程序，并使用 UI 测试框架来启动应用程序，与 UI 控件交互，并验证 UI 是否显示预期的内容。自动化 UI 测试的编写、运行和维护可能很耗时。然而，您是否更愿意完全依赖手动测试和 QA，而不是利用自动化测试的优势来确保应用程序行为的质量和可靠性？

当使用 MVVM 模式时，编写单元测试来测试应用程序的不同区域变得非常容易，因为它促进了关注点的分离，并且应该是 UI 框架无关的。当所有内容都在代码背后时，通过自动化 UI 测试来测试业务逻辑会变得非常复杂，难以维护，并且很快就会出错。UI 测试有其目的，因为它们可以测试应用程序的用户界面是否按预期运行。这两种类型的测试都很重要，在确保应用程序质量方面发挥着不同的作用。但是（自动化的）UI 测试应该只做这件事：测试 UI。您的 ViewModel 和业务逻辑应该已经通过其他自动化测试进行了测试。

# 关于 MVVM 的常见误解

关于 MVVM 存在一些常见的误解，这些误解可能导致对其原则和最佳实践的误解。让我们消除其中的一些误解，并对该模式提供清晰的解释。

## 在代码背后不应该有代码

虽然 MVVM 的主要目的是将表示逻辑与应用逻辑分离，但这并不意味着代码隐藏中不应该有任何代码。代码隐藏仍然可以用来处理简单的 UI 相关事件或与视图紧密耦合的任何逻辑。

实际上，在某些情况下，将一些代码放在代码隐藏中可能比尝试将所有内容移动到视图模型中更高效、更易于维护。例如，处理 UI 动画、滚动、控制焦点或复杂的视觉行为可能比通过数据绑定实现更容易。

为了确保在 MVVM 中正确分离关注点，必须避免在视图的代码隐藏中包含业务逻辑。代码隐藏应保持最小化，并尽可能简单，以保持视图和视图模型之间的分离。

## 模型应该是 DTO、领域实体或 POCO

虽然在 MVVM 中使用模型的对象类型一直是一个有争议的话题，但在我看来，这并不是一个关键因素。MVVM 的主要原则是将业务逻辑从视图中移除，并在视图模型中只包含简单的验证逻辑。因此，模型可以是任何对象类型，通常是不同类型对象的组合。模型不是单一类型的事物；它是包含应用程序实体、业务逻辑、存储库等所有“在视图和视图模型之外”的内容。重要的是要记住，视图和视图模型不应包含任何业务或持久化逻辑。

## 视图（View）和视图模型（ViewModel）之间不应相互了解

虽然视图模型不应该了解视图以保持关注点的分离，但视图可以引用视图模型。需要注意的是，只要视图模型不依赖于视图，这并不违反 MVVM 的原则。在.NET MAUI 等平台上使用“编译绑定”可以提供显著的性能提升，但为了使它们能够工作，视图必须了解它所绑定的类型。

更多关于编译绑定的信息

想了解更多关于编译绑定的信息？*第四章**，.NET MAUI 中的数据绑定*，为您提供了全面的介绍。

此外，可能存在一些情况下，视图需要从代码隐藏中直接调用视图模型上的方法。在 UI 非常复杂且命令无法轻松绑定的情况下，这可能是有必要的。

## MVVM 过于复杂，仅适用于大型应用程序

MVVM 本身并不一定复杂，但要熟练掌握它可能需要一些学习和实践。对于不习惯使用这种设计模式的开发者来说，理解关注点分离的概念并在 MVVM 中实现它可能有点挑战性。此外，正确设置绑定可能需要一些努力，尤其是在处理大型和复杂的视图时。然而，一旦您理解了它，您会发现开发过程变得更加简单，代码也变得更加易于维护和测试。尽管可能存在学习曲线，但在应用程序开发中采用 MVVM 仍然值得，即使是小型和简单的应用程序。这类应用程序需要随着时间的推移进行维护和更新。随着时间的推移，它们的业务逻辑也将从单元测试中受益，对吧？

话虽如此，对于具有非常简单 UI 的应用程序，例如只有几个 UI 元素并且几乎没有业务逻辑反映在 UI 上的应用程序，MVVM 可能是过度设计。然而，再次审视之前仅包含两个 UI 控件的示例，我们发现它通过应用 MVVM 设计模式受益于可测试性。

# 摘要

总结来说，MVVM 模式将数据、UI 和逻辑的关注点分开，这使得应用程序更容易进行测试、修改和扩展。通过使用模型来表示数据和业务逻辑，视图来向用户展示数据，以及视图模型在模型和视图之间进行调解，MVVM 模式促进了职责的清晰分离，这使得开发和维护复杂应用程序变得更加容易。此外，使用命令和数据绑定提供了一种强大的方式来处理用户输入并保持 UI 与应用程序状态同步。理解 MVVM 的组件对于构建可维护、可扩展且易于测试的 .NET MAUI 应用程序至关重要。

在*第二章**，什么是 .NET MAUI？*中，我们将深入了解 .NET MAUI，以便您对该框架有一个良好的理解。如果您已经对 .NET MAUI 有深入的了解，您可以跳过这一章。如果您对其基础知识有所了解，这将是一个很好的复习。
