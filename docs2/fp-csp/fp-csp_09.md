# 9

# 柯里化和部分应用

恭喜！你已经覆盖了这本书的超过 90%！你太棒了，我要给你一个虚拟的击掌！在本章中，我们将讨论柯里化和部分应用。我知道有一个特殊的关键字，`partial`，它允许我们将类、结构体或接口拆分成多个部分；然而，在函数式编程中，部分应用有着不同的含义。柯里化将具有多个参数的函数转换成一系列函数，每个函数只接受一个参数。这种转换允许参数的增量应用，其中每一步都会返回一个新的函数，等待下一个输入。另一方面，部分应用涉及将一些参数固定到函数中，从而产生一个参数更少的函数。这两种技术都在函数的参数在执行过程中不是同时可用的情况下非常有用，因此提供了在参数可用时应用这些参数的灵活性。

为了学习这些新技术，我们将通过以下部分进行学习：

+   理解柯里化

+   柯里化实现的步骤

+   部分应用

+   部分应用的应用领域

+   挑战和限制

在继续之前，我不能不给你留下定期的自我检查任务来衡量你在阅读本章前后的知识水平。

# 任务 1 – 柯里化塔攻击函数

使用柯里化重构`AttackEnemy`函数，允许在游戏过程中预设`towerType`以供多次使用，同时接受动态输入的`enemyId`和`damage`：

```cs
public void AttackEnemy(TowerTypes towerType, int enemyId, int damage)
{
   Console.WriteLine($"Tower {towerType} attacks enemy {enemyId} for {damage} damage.");
}
```

# 任务 2 – 游戏设置的柯里化部分应用

应用部分应用来创建一个函数，用于快速设置标准游戏设置，其中`map`是预定义的，但允许动态设置`difficultyLevel`和`isMultiplayer`：

```cs
public void SetGameSettings(string map, int difficultyLevel, bool isMultiplayer)
{
   Console.WriteLine($"Setting game on map {map} with difficulty {difficultyLevel}, multiplayer: {isMultiplayer}");
}
```

# 任务 3 – 为游戏功能进行柯里化权限检查

柯里化此函数，使其首先接受一个`userRole`，然后返回另一个函数，该函数接受一个`feature`，确定指定角色是否有权访问它：

```cs
public bool CheckGameFeatureAccess(UserRoles userRole, GameFeatures feature)
{
    return _gameFeatureManager.HasAccess(userRole, feature);
}
```

你可能认为这些方法已经是最优的，但它们的简单性是有意为之的。这些专注的任务让你在不受干扰的情况下练习函数式编程。像往常一样，这些任务只是供你自我评估，你并不需要轻易解决它们。所以，如果你对它们有任何困难，就继续阅读本章。到本章结束时，你将像专业人士一样应对这些挑战。记住这一点，希望你能有一个愉快的编码体验！

# 理解柯里化

**柯里化**是一种将具有多个参数的函数转换为接受单个参数的函数序列的技术。以数学家哈斯克尔·柯里命名，柯里化允许参数的部分应用，其中每个提供的参数都返回一个准备接受下一个输入的新函数。这种方法在想要在不同场景中重用函数的部分或在多步函数的步骤之间进行一些计算时很有帮助。

在深入概念之前，让我们回顾一下史蒂夫和朱莉娅的对话。

朱莉娅：*嗨，史蒂夫，我看到你在函数式编程方面取得了很大的进步。今天，我们将探讨柯里化和部分应用这两种强大的技术，它们可以帮助你编写更可重用和* *模块化的代码。*

史蒂夫：*嗨，朱莉娅！这听起来很有趣。我听说过这些概念，但从未真正理解如何在 C#中应用它们。你能为我解释一下吗？*

朱莉娅：*绝对是这样！让我们从柯里化开始。柯里化将具有多个参数的函数转换为一串函数，每个函数只接受一个参数。这种转换允许参数的增量应用，其中每一步都返回一个等待下一个输入的新函数。*

史蒂夫：*所以，它就像创建一个函数链，每个函数都接受* *一个参数吗？*

朱莉娅：*没错！让我通过一个 YouTube 视频管理系统中的实际示例向你展示柯里化的应用。我们经常需要检查用户是否有执行不同操作（如查看、评论和* *上传视频）的适当权限。*

## 标准方法

通常，我们可能会使用这样的函数来检查权限：

```cs
public static bool CheckPermission(Actions action, UserRoles userRole)
{
     // Assume a method that checks a database or cache for permissions
     return _permissionsManager.HasPermission(action, userRole);
}
```

此函数有效，但每次检查同一角色的不同操作时都需要两个参数。

## 柯里化方法

通过柯里化此函数，我们创建了一个更适应的权限检查器，在会话期间处理同一用户角色的多个操作或在类似环境中非常有用：

```cs
public static Func<Actions, Func<UserRoles, bool>> CurryCheckPermission()
{
   return action => userRole =>
   {
      return _permissionsManager.HasPermission(action, userRole);
   };
}
```

按照以下方式使用柯里化方法：

```cs
var curriedPermissionChecker = CurryCheckPermission();
var checkViewerPermissions = curriedPermissionChecker(Actions.View);
var checkCommentPermissions = curriedPermissionChecker(Actions.Comment);
var checkUploadPermissions = curriedPermissionChecker(Actions.Upload);
bool canView = checkViewerPermissions(UserRoles.Admin);
bool canComment = checkCommentPermissions(UserRoles.Admin);
bool canUpload = checkUploadPermissions(UserRoles.Admin);
```

正如你在下面的示例中可以看到的，柯里化有几个优点：

**可复用性**：柯里化函数允许你预先定义操作并创建用于检查每个角色的特定函数。这在会话期间需要重复检查同一操作的多重角色时特别有益。

**减少冗余代码**：柯里化将操作与角色评估分离，在执行同一操作下的多个不同角色的多次检查时减少了重复代码。这提高了代码的可读性和可维护性。

**部分应用便利性**：当你知道所有后续检查都将针对该操作时，你可以部分应用该操作，这在会话或特定接口段专注于单一类型操作的场景中很常见。

# 柯里化的逐步实现

当朱莉娅解释完柯里化示例后，史蒂夫若有所思地点了点头。

Steve：*我觉得我开始明白了。但我们如何在代码中实际实现柯里化呢？*

Julia：*这是一个很好的问题！让我们一步一步地分解它...*

实现柯里化的过程实际上相当简单：

1.  识别函数：

    选择一个接受多个参数的函数。如果您经常发现自己一次只使用一些参数，或者参数自然地按阶段分组，则该函数是柯里化的候选者。

1.  定义柯里化函数：

    将多参数函数转换为一系列嵌套的单参数函数。每个函数返回另一个函数，该函数期望序列中的下一个参数。

1.  使用 Func 委托实现：

    我们可以利用 `Func` 委托来实现柯里化函数。每个 `Func` 返回另一个 `Func`，直到所有参数都被考虑在内，最终返回最终值。

1.  简化调用：

    尽管柯里化给函数调用增加了一层间接性，但它通过将其分解为可管理的步骤来简化过程，每个步骤都可以根据需要单独处理。

## 用例

虽然柯里化可能看起来有些繁琐，但它有许多有益的情况。让我们探讨一些常见的用例，看看柯里化如何应用于提高代码模块化和可重用性。

## 配置设置

当设置涉及多个参数的配置时，柯里化允许在整个应用程序中增量地指定这些设置。考虑一个例子，我们需要为每个用户组设置一个通知服务，限制每分钟发送的最大通知数：

```cs
public static Func<NotificationType, Func<int, Action<string>>> CurryNotificationConfig()
{
     return notificationType => maxNotificationsPerMinute => recipientEmail =>
     {
         Console.WriteLine($"Configuring {notificationType} notification for {recipientEmail} with max {maxNotificationsPerMinute} notifications per minute");
     };
}
// Usage
var configureNotifications = CurryNotificationConfig();
var configureEmailNotifications = configureNotifications(NotificationType.Email);
// Configure users to receive a maximum of 10 notifications per minute
var configureUserNotifications = configureEmailNotifications(10);
configureUserNotifications("alice@csharp-interview-preparation.com");
configureUserNotifications("bob@csharp-interview-preparation.com");
// Configure moderators to receive more notifications
var configureModeratorNotifications = configureEmailNotifications(50);
configureModeratorNotifications("moderator1@csharp-interview-preparation.com");
configureModeratorNotifications("moderator2@csharp-interview-preparation.com");
configureModeratorNotifications("moderator3@csharp-interview-preparation.com");
// Configure admins to receive even more notifications
var configureAdminNotifications = configureEmailNotifications(100);
configureAdminNotifications("admin1@csharp-interview-preparation.com");
configureAdminNotifications("admin2@csharp-interview-preparation.com");
```

在本例中，`CurryNotificationConfig` 函数接受通知类型、每分钟最大通知数和收件人电子邮件作为柯里化参数。每个用户组都有自己的设置，这使得为每个特定用户的配置更具可重用性。

## 事件处理

在事件驱动编程中，柯里化可以用于处理具有特定预填充参数的事件，简化事件处理程序逻辑。让我们考虑一个处理按钮点击事件的例子：

```cs
public static Func<string, Func<EventArgs, void>> CurryButtonClickHandler()
{
     return buttonName => eventArgs =>
     {
         Console.WriteLine($"Button {buttonName} clicked!");
         // Handle the button click event
     };
}
// Usage
var handleButtonClick = CurryButtonClickHandler();
var handleSaveClick = handleButtonClick("Save");
var handleCancelClick = handleButtonClick("Cancel");
// Attach event handlers
saveButton.Click += (sender, e) => handleSaveClick(e);
cancelButton.Click += (sender, e) => handleCancelClick(e);
```

通过对按钮点击处理程序进行柯里化，您可以创建针对每个按钮（`handleSaveClick` 和 `handleCancelClick`）的专用函数，并预先填充按钮名称。这简化了事件处理程序附加过程，并使代码更易于阅读和维护。

## 部分应用

柯里化促进了部分应用，其中可以通过在事先固定一些参数值来将具有许多参数的函数转换为具有较少参数的函数。我们将在下一节中详细讨论它，但现在让我们看看一个日志函数的例子：

```cs
public static Func<string, Func<string, void>> CurryLogMessage()
{
     return logLevel => message =>
     {
         Console.WriteLine($"{logLevel}: {message}");
         // Log the message with the specified log level
     };
}
// Usage
var logMessage = CurryLogMessage();
var logError = logMessage("ERROR");
var logWarning = logMessage("WARNING");
logError("An error occurred.");
logWarning("This is a warning message.");
```

在这种情况下，柯里化允许您部分应用 `logLevel` 参数，创建 `logError` 和 `logWarning` 函数以适应不同的日志级别。这减少了重复传递日志级别的需求，并使日志调用更加简洁和表达性强。

## 异步编程

柯里化可以通过将参数的准备与异步操作的执行分离来简化异步代码。以下是一个制作 HTTP 请求的示例：

```cs
public static Func<string, Func<Dictionary<string, string>, Func<CancellationToken, Task<string>>>> CurryHttpGetRequest()
{
     return url => headers => async cancellationToken =>
     {
         using (var client = new HttpClient())
         {
             foreach (var header in headers)
             {
                 client.DefaultRequestHeaders.Add(header.Key, header.Value);
             }
             return await client.GetStringAsync(url, cancellationToken);
         }
     };
}
// Usage
var getRequest = CurryHttpGetRequest();
var getWithUrl = getRequest("https://api.example.com/data");
var getWithHeaders = getWithUrl(new Dictionary<string, string>
{
     { "Authorization", "Bearer token123" },
     { "Content-Type", "application/json" }
});
string response = await getWithHeaders(CancellationToken.None);
```

通过柯里化 HTTP `GET` 请求函数，我们可以将 URL 的配置、头部信息和取消令牌分开。这允许我们编写更灵活和模块化的异步代码，其中每个请求方面都可以独立准备并在需要时组合。

通常，柯里化的目标是通过减少冗余并简化函数在应用程序不同部分的使用来简化代码。

# 部分应用

简而言之，部分应用涉及取一个接受多个参数的函数，提供其中一些参数，并返回一个只需要剩余参数的新函数。这与柯里化所做的非常相似。

柯里化将具有多个参数的函数转换为一串函数，每个函数只接受一个参数。这使得部分应用变得自然，因为柯里化函数返回的每个函数都可以被视为部分应用函数。

柯里化和部分应用之间的关键区别在于它们的实现和使用：

**柯里化**是关于转换函数结构本身，将多参数函数转换为一串单参数函数

另一方面，**部分应用**并不一定改变函数结构，但通过预先填充一些参数来减少它需要的参数数量

部分应用在涉及配置、重复性任务和预定义条件的情况下尤其有用。它简化了函数最终用户的接口，并通过在部分应用函数中封装常见参数，可以使代码库更容易维护。

史蒂夫挠了挠头，看起来有些困惑。

史蒂夫：*朱莉娅，我有点难以区分柯里化和部分应用。它们看起来* *非常相似*。

朱莉娅：*史蒂夫，你* *说得对，它们是相关的。关键区别在于它们的实现和使用方式。让我* *进一步解释...*

# 部分应用的应用领域

尽管部分应用在相当类似的场景中使用，但拥有多个参数的可能性使其更适用。以下是一些应用部分应用的好地方：

+   **配置管理**：在配置在不同环境或应用程序的部分之间略有差异的系统，部分应用可以通过预先设置常见参数来简化配置管理。

+   **用户界面事件**：在 GUI 编程中，事件处理器通常需要特定的参数，一旦设置就不会改变。部分应用允许开发者使用预定义的参数准备这些处理器，使代码更整洁且易于管理。

+   **API 集成**：在与外部 API 交互时，某些参数（如 API 密钥和用户令牌）在请求之间保持不变。将这些参数部分应用到 API 请求函数中可以简化函数调用，并通过隔离敏感数据来提高安全性。

+   **日志记录和监控**：在应用程序中，日志记录通常涉及重复的信息，如日志级别和类别。部分应用可以创建更易于使用且减少错误发生可能性的专用日志记录函数。

+   **数据处理管道**：在涉及数据转换和处理的场景中，函数通常需要特定的设置或操作以保持一致性。部分应用可以预先配置这些函数所需的设置，使管道更加模块化和可重用。

## 部分应用函数的配置设置示例

考虑到在内容管理系统中的场景，不同类型的内容可能需要特定的渲染设置。处理渲染的函数可能需要多个参数，但许多这些参数在内容类型之间是通用的。

## 标准渲染函数

假设我们有一个渲染内容的函数，它接受多个参数，并且只有在拥有所有这些参数的情况下才能使用：

```cs
public string RenderContent(string content, string format, int width, int height, string theme)
{
     // Render content based on the provided settings
     return $"Rendering {content} as {format} in {theme} theme with dimensions {width}x{height}";
}
```

## 部分应用函数

为了简化在整个应用程序中使用此功能，尤其是在大多数内容使用标准格式和主题的情况下，您可以部分应用这些常用参数：

```cs
public Func<string, int, int, string> RenderStandardContent()
{
     string defaultFormat = "HTML";
     string defaultTheme = "Light";
     return (content, width, height) => RenderContent(content, defaultFormat, width, height, defaultTheme);
}
var renderStandard = RenderStandardContent();
string renderedOutput = renderStandard("Hello, world!", 800, 600);
Console.WriteLine(renderedOutput);
```

这种方法使我们能够在不重复指定格式和主题的情况下使用`renderStandard`函数。你现在可能认为我们在这里可以使用默认参数，并消除所有这些部分应用。但如果我们不仅有“标准”的渲染方式，还有多种方式：桌面、手机和平板？这就是我们在“主要”的`RenderContent`函数中创建针对每种情况的具体函数，并在其中共享代码的时候。这看起来像是函数的某种继承。

在研究了柯里化和部分应用之后，史蒂夫不禁感到有些不知所措。他联系了朱莉娅，表达了他的担忧。

史蒂夫：*朱莉娅，虽然我看到了这些技术的潜力，但我担心复杂性增加和潜在的性能问题。你是如何在你的项目中管理这些挑战的？*

朱莉娅：*史蒂夫，这些观点是有效的。确实，柯里化和部分应用有其挑战和局限性。让我们探讨一些这些挑战，并讨论克服它们的策略...*

# 挑战和局限性

在研究了柯里化和部分应用之后，史蒂夫不禁感到有些不知所措。他联系了朱莉娅，表达了他的担忧。

史蒂夫：*朱莉娅，虽然我看到了这些技术的潜力，但我担心复杂性增加和潜在的性能问题。你是如何在你的项目中管理这些挑战的？*

朱莉娅：*这些观点很有道理，史蒂夫。确实，柯里化和部分应用有其挑战和局限性。以下是一些例子：*

+   **增加的复杂性**：对于不熟悉函数式编程的开发者来说，柯里化和部分应用可能会使代码看起来更复杂，更难以理解，从而导致学习曲线更陡峭。

+   **性能开销**：.NET 中的每个函数调用都涉及一定的开销，而柯里化通过将单个多参数函数转换为多个单参数函数来增加函数调用的数量。这可能会对性能产生潜在影响，尤其是在性能关键的应用中。

+   **调试难度**：调试柯里化函数可能更具挑战性，因为数据流和执行流程分布在多个函数调用中，而不是集中在单个函数体内。

*听起来有点吓人，但不必担心，因为我们可以使用以下策略来克服这些挑战：*

+   **教育和培训**：提供关于函数式编程概念的培训课程和资源，可以帮助团队成员理解和有效使用柯里化和部分应用。

+   **选择性使用**：有选择地应用柯里化和部分应用，专注于它们能提供明确好处的地方，例如在配置管理和 API 交互中，而不是普遍应用。

+   **性能监控**：始终监控柯里化在特定环境中的性能影响。使用分析工具来识别任何瓶颈，并在必要时重构代码以优化性能。

+   **增强的调试技术**：利用高级调试工具和技术，如条件断点和调用栈分析，以更好地管理柯里化函数的调试。

+   **集成策略**：在面向对象框架内工作时，逐步集成函数式编程技术，并确保它们补充而不是复杂化架构。

虽然柯里化和部分应用可能会引入一些复杂性和挑战，但有了正确的策略和工具，这些挑战可以有效地得到管理。

当他们讨论完柯里化和部分应用后，史蒂夫看起来很深思。

史蒂夫：*这真的很启发人心，朱莉娅。我可以看出这些技术如何使我们的塔防游戏代码更加灵活* *和易于维护。*

朱莉娅：*没错，史蒂夫！记住，就像编程中的任何工具一样，关键在于知道何时以及如何有效地应用这些* *概念。*

史蒂夫：*谢谢，朱莉娅。我期待在下次* *编码会议* *中尝试这些！*

朱莉娅微笑着，对史蒂夫在函数式编程概念上的热情和进步感到满意。

# 练习

在本章中，我们探讨了柯里化和部分应用的功能编程概念。现在，让我们将这些技术应用到移动塔防游戏的实际场景中。

## 练习 1

使用柯里化重构 `AttackEnemy` 函数，允许在游戏过程中预设 `towerType` 以供多次使用，同时接受 `enemyId` 和 `damage` 的动态输入：

```cs
public void AttackEnemy(TowerTypes towerType, int enemyId, int damage)
{
   Console.WriteLine($"Tower {towerType} attacks enemy {enemyId} for {damage} damage.");
}
```

## 练习 2

应用部分应用创建一个用于快速设置标准游戏设置的函数，其中 `map` 是预定义的，但允许动态设置 `difficultyLevel` 和 `isMultiplayer`：

```cs
public void SetGameSettings(string map, int difficultyLevel, bool isMultiplayer)
{
   Console.WriteLine($"Setting game on map {map} with difficulty {difficultyLevel}, multiplayer: {isMultiplayer}");
}
```

## 练习 3

将此函数柯里化，使其首先接受一个 `userRole`，然后返回另一个函数，该函数接受一个 `feature`，以确定指定角色是否有权访问它：

```cs
public bool CheckGameFeatureAccess(UserRoles userRole, GameFeatures feature)
{
    return _gameFeatureManager.HasAccess(userRole, feature);
}
```

这些练习旨在帮助你在移动塔防游戏的背景下应用柯里化和部分应用。通过练习这些技术，你将增强游戏代码库的功能性和可扩展性，使其更容易管理和扩展。记住，你练习这些函数式编程技术越多，你在构建复杂游戏逻辑方面的熟练程度就会越高。

# 解决方案

这里是关于移动塔防游戏中柯里化和部分应用练习的解决方案。这些解决方案展示了如何实现所讨论的概念，并提供了在游戏开发中实际使用这些概念的示例。

## 解决方案 1

要使用柯里化重构 `AttackEnemy` 函数，我们首先定义原始函数，然后将其转换：

```cs
public Func<int, int, void> CurriedAttack(TowerTypes towerType)
{
   return (enemyId, damage) =>
   {
      Console.WriteLine($"Tower {towerType} attacks enemy {enemyId} for {damage} damage.");
   };
}
var attackWithCannon = CurriedAttack(TowerTypes.Cannon);
attackWithCannon(1, 50); // Attack enemy 1 with 50 damage
attackWithCannon(2, 75); // Attack enemy 2 with 75 damage
```

此柯里化函数允许一次设置 `towerType` 并在多次攻击中重复使用，从而提高代码的可重用性并减少冗余。

## 解决方案 2

使用部分应用简化游戏设置配置：

```cs
public Func<int, bool, void> ConfigureWithMap(Maps map)
{
   return (difficultyLevel, isMultiplayer) =>
   {
      Console.WriteLine($"Setting game on map {map} with difficulty {difficultyLevel}, multiplayer: {isMultiplayer}");
   };
}
var configureForMapDesert = ConfigureWithMap(Maps.Desert);
// Configure for Desert map with difficulty 5 and multiplayer enabled
configureForMapDesert(5, true);
// Configure for Desert map with difficulty 3 and multiplayer disabled
configureForMapDesert(3, false);
```

此函数部分应用了映射设置，允许动态配置其他设置，例如 `difficultyLevel` 和 `isMultiplayer`。

## 解决方案 3

实现一个基于用户角色和动作的权限管理的柯里化函数：

```cs
public Func<GameFeatures, bool> CurriedCheckPermission(UserRoles userRole)
{
   return (feature) =>
   {
      return _gameFeatureManager.HasAccess(userRole, feature);
   };
}
var checkAdminPermissions = CurriedCheckPermission(UserRoles.Admin);
bool canEdit = checkAdminPermissions(GameFeatures.EditLevel);
bool canPlay = checkAdminPermissions(GameFeatures.PlayGame);
Console.WriteLine($"Admin permissions - Edit: {canEdit}, Play: {canPlay}");
```

此柯里化函数通过一次设置 `userRole` 并允许对各种功能进行动态检查，简化了权限检查过程，从而简化了权限管理流程。

这些解决方案展示了柯里化和部分应用在移动塔防游戏中的实际应用，提高了代码的结构和可维护性。通过实施这些技术，开发者可以增强游戏逻辑的灵活性和可读性。

# 摘要

在本章中，我们探讨了柯里化和部分应用。柯里化将多参数函数转换为一系列单参数函数，每个函数接受一个参数并返回另一个函数，该函数准备好接受下一个参数。这种技术特别有利于创建可配置和高度模块化的代码。

部分应用涉及固定函数的一些参数，并创建一个需要较少参数的新函数。这种方法在特定参数反复使用相同值的情况下非常有价值，因为它简化了函数调用并减少了冗余。

如果某些内容仍然不清楚，请使用本章提供的示例——修改它们，实验它们，并将它们整合到你的项目中。换句话说，看看它在实际中的应用。无论是简化配置管理、使事件处理更加直接，还是减少 API 交互的复杂性，柯里化和部分应用都能帮助你减少函数调用的复杂性，增强代码的模块化和可读性，并且总体上，将你的程序在函数式编程方面提升到新的水平。

在下一章中，我们将总结前几章中关于管道和组合所学的所有内容，增加更多现实世界的场景，并讨论更多高级主题，如错误处理、测试和性能考虑。
