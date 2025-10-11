# 第十三章：解开混乱

并非所有应用程序都是考虑到测试而编写的。其中很少是最初使用 TDD 开发的。通常，原始的开发者已经离开，文档不正确、不完整或完全缺失。

在本章中，我们将了解：

+   处理继承的代码

+   特征测试

+   带着测试进行重构

# 继承代码

本章是关于需要（应该是）微小更改的遗留代码的案例研究。我们将很快发现，这个更改并不那么微小。首先，让我们看看遗留应用程序做了什么。

这里是这段代码运行的一些示例输出：

```cs
Take a guess: AAAA
---+
Take a guess: BBBA
-+-+
Take a guess: CBCA
++
Take a guess: DBDA
++-+
```

```cs
Take a guess: DBEA
++++
Congratulations you guessed the password in 5 tries.
Press any key to quit.
```

看起来，这个程序并不那么糟糕。在与业务分析师交谈中，应用程序被解释为是一款游戏。

# 游戏

这款特定的游戏被称为 *Mastermind*，是一种解码谜题。根据业务分析师的说法，密码由字母 A 到 F 组成，包含随机选择的四个字母。玩家的目标是确定通行码。

玩家在游戏中会收到提示。对于放置正确的字母，玩家会收到一个加号符号。对于位置错误但正确的字母，玩家会收到一个减号符号。如果字母不正确，玩家则不会收到任何符号。

# 请求进行更改

在游戏测试过程中，发现玩家发现通行码的速度太快。因此，游戏并没有像预期的那样有趣。建议的解决方案是通过允许使用超过六个字母来使密码更复杂。我们的任务是扩展字符范围到 A 到 Z。

我们可以先查看现有代码，以确定我们可能需要做出更改的地方。那就是我们发现这个地方的地方！

在文件 `Program.cs` 中：

```cs
class Program
{
  static void Main(string[] args)
  {
    char[] g;
    char[] p = new[] { 'A', 'A', 'A', 'A' };
    int i = 0;
    int j = 0;
    int x = 0;
    int c = 0;
    Random rand = new Random(DateTime.Now.Millisecond);
    if (args.Length > 0 && args[0] != null) p = args[0].ToCharArray();
    else goto randomize_password;
    guess: Console.Write("Take a guess: ");
    g = Console.ReadLine().ToArray();
    i = i + 1;
    if (g.Length != 4) goto wrong_size;
    if (g == p) goto success;
    x = 0;
    c = 0;
    check_loop:
    if (g[x] > 65 + 26) g[x] = (char)(g[x] - 32);
    if (g[x] == p[x]) Console.Write("+", c = c + 1);
    else if (p.Contains(g[x])) Console.Write("-");
    x = x + 1;
    if (x < 4) goto check_loop;
    Console.WriteLine();
    if (c == 4) goto success;
    goto guess;
    success: Console.WriteLine("Congratulations you guessed the   
    password in " + i + " tries.");
    goto end;
    wrong_size: Console.WriteLine("Password length is 4.");
    goto guess;
    randomize_password: j = 0;
    password_loop: p[j] = (char)(rand.Next(6) + 65);
    j = j + 1;
    if (j < 4) goto password_loop;
    goto guess;
    end: Console.WriteLine("Press any key to quit.");
    Console.ReadKey();
  }
}
```

现在我们有几个问题。首先，并不完全清楚字母是从哪里来的。其次，这段代码显然没有经过测试。最后，即使更改是直接的，确保我们没有破坏任何东西也不是那么简单。我们必须进行完整的手动回归测试来验证这些是否正常工作，而尝试验证所有字母，从 A 到 Z，可能需要非常长的时间。

# 生活有时会给你柠檬

虽然我希望你永远不会收到这么糟糕的代码，但我们将一步步讲解如何将即使是这样的代码也转变为可读、可维护和完全测试的代码。最好的部分，你可能会不相信的部分，是转换这段代码实际上是安全且相对容易的。

# 开始

在任何这样的代码情况下，我们首先必须做的是将问题代码从我们没有控制的环境中移除。在这种情况下，如果代码位于 `Program.main` 中，我们就无法测试它。所以，让我们把整个代码块抓取出来，放入一个名为 `Mastermind` 的类中。我们将有一个名为 `Play` 的单个函数来运行游戏。这被认为是一种安全的重构，因为我们没有改变任何现有的代码，只是将其移动到其他地方。

在文件 `Program.cs` 中：

```cs
class Program
{
  static void Main(string[] args)
  {
    var game = new Mastermind();
    game.Play(args);
  }
}
```

在文件 `Mastermind.cs` 中：

```cs
class Mastermind
{
  public void Play(string[] args)
  {
    char[] g;
    char[] p = new[] { 'A', 'A', 'A', 'A' };
    int i = 0;
    int j = 0;
    int x = 0;
    int c = 0;
    Random rand = new Random(DateTime.Now.Millisecond);
    if (args.Length > 0 && args[0] != null) p = args[0].ToCharArray();
    else goto randomize_password;
    guess: Console.Write("Take a guess: ");
    g = Console.ReadLine().ToArray();
    i = i + 1;
    if (g.Length != 4) goto wrong_size;
    if (g == p) goto success;
    x = 0;
    c = 0;
    check_loop:
    if (g[x] > 65 + 26) g[x] = (char)(g[x] - 32);
    if (g[x] == p[x]) Console.Write("+", c = c + 1);
    else if (p.Contains(g[x])) Console.Write("-");
    x = x + 1;
    if (x < 4) goto check_loop;
    Console.WriteLine();
    if (c == 4) goto success;
    goto guess;
    success: Console.WriteLine("Congratulations you guessed the 
    password in " + i + " tries.");
    goto end;
    wrong_size: Console.WriteLine("Password length is 4.");
    goto guess;
    randomize_password: j = 0;
    password_loop: p[j] = (char)(rand.Next(6) + 65);
    j = j + 1;
    if (j < 4) goto password_loop;
    goto guess;
    end: Console.WriteLine("Press any key to quit.");
    Console.ReadKey();
  }
}
```

在这个点上再次运行代码显示，一切仍然正常工作。下一步是一个外观上的改进；让我们将 `Play` 方法扩展到几个部分。这应该有助于我们确定大型公共方法内部存在哪些私有方法。

在文件 `Mastermind.cs` 中：

```cs
class Mastermind
{
  public void Play(string[] args)
  {
    // Variable Declarations - Global??
    char[] g;
    char[] p = new[] { 'A', 'A', 'A', 'A' };
    int i = 0;
    int j = 0;
    int x = 0;
    int c = 0;
    // Initialize randomness
    Random rand = new Random(DateTime.Now.Millisecond);
    // Determine if a password was passed in?
    if (args.Length > 0 && args[0] != null) p = args[0].ToCharArray();
    else goto randomize_password; // Create a password if one was not 
    provided
    // Player move - guess the password
    guess: Console.Write("Take a guess: ");
    g = Console.ReadLine().ToArray();
    i = i + 1;
    if (g.Length != 4) goto wrong_size;
    if (g == p) goto success;
    x = 0;
    c = 0;
    // Check if the password provided by the player is correct
    check_loop:
    if (g[x] > 65 + 26) g[x] = (char)(g[x] - 32);
    if (g[x] == p[x]) Console.Write("+", c = c + 1);
    else if (p.Contains(g[x])) Console.Write("-");
    x = x + 1;
    if (x < 4) goto check_loop; // Still checking??
    Console.WriteLine();
    if (c == 4) goto success; // Password must have been correct
    goto guess; // No correct, try again
    // Game over you win
    success: Console.WriteLine("Congratulations you guessed the 
    password in " + i + " tries.");
    goto end;
    // Password guess was wrong size - Error Message
    wrong_size: Console.WriteLine("Password length is 4.");
    goto guess;
    // Create a random password
    randomize_password: j = 0;
    password_loop: p[j] = (char)(rand.Next(6) + 65);
    j = j + 1;
    if (j < 4) goto password_loop;
    goto guess; // Start the game
    // Game is complete - exit
    end: Console.WriteLine("Press any key to quit.");
    Console.ReadKey();
  }
}
```

我们现在已经使用空白将程序分成几个部分，并添加了注释来解释我们认为每个部分在做什么。到目前为止，我们几乎准备好开始测试了。我们只是有几件事情需要处理，其中最糟糕的是 `Console` 类。

# 抽象第三方类

如果我们现在尝试测试，应用程序会调用第一个 `ReadLine` 调用，并且测试会超时。控制台具有重定向输入和输出的能力，但我们不会使用这个特性，因为它特定于控制台，我们想要展示一个更通用的解决方案，你可以在任何地方应用。

我们需要的是一个提供与控制台类似接口的类。然后我们可以为测试注入我们的类，并为生产代码提供一个薄薄的包装器。现在让我们测试这个接口。

在文件 `InputOutputTests.cs` 中：

```cs
public class InputOutputTests
{
  [Fact]
  public void ItExists()
  {
    var inout = new MockInputOutput();
  }
}
```

在文件 `ReadLineTests.cs` 中：

```cs
public class ReadLineTests
{
  [Fact]
  public void ItCanBeReadFrom()
  {
    var inout = new MockInputOutput();
    inout.InFeed.Enqueue("Test");

    // Act
    var input = inout.ReadLine();
  }

  [Fact]
  public void ProvidedInputCanBeRetrieved()
  {
    // Arrange
    var inout = new MockInputOutput();
    inout.InFeed.Enqueue("Test");

    // Act
    var input = inout.ReadLine();

    // Assert
    Assert.Equal("Test", input);
  }

  [Fact]
  public void ProvidedInputCanBeRetrievedInSuccession()
  {
    // Arrange
    var inout = new MockInputOutput();
    inout.InFeed.Enqueue("Test 1");
    inout.InFeed.Enqueue("Test 2");

    // Act
    var input1 = inout.ReadLine();
    var input2 = inout.ReadLine();

    // Assert
    Assert.Equal("Test 1", input1);
    Assert.Equal("Test 2", input2);
  }
}
```

在文件 `ReadTests.cs` 中：

```cs
public class ReadTests
{
  [Fact]
  public void ItCanBeReadFrom() 
  {
    var inout = new MockInputOutput();
    inout.InFeed.Enqueue("T");

    // Act
    var input = inout.Read();
  }

  [Fact]
  public void ProvidedInputCanBeRetrieved()
  {
    // Arrange
    var inout = new MockInputOutput();
    inout.InFeed.Enqueue("T");

    // Act
    var input = inout.Read();

    // Assert
    Assert.Equal('T', input);
  }

  [Fact]
  public void ProvidedInputCanBeRetrievedInSuccession()
  {
    // Arrange
    var inout = new MockInputOutput();
    inout.InFeed.Enqueue("T");
    inout.InFeed.Enqueue("E");

    // Act
    var input1 = inout.Read();
    var input2 = inout.Read();

    // Assert
    Assert.Equal('T', input1);
    Assert.Equal('E', input2);
  }
}
```

在文件 `WriteTests.cs` 中：

```cs
public class WriteTests
{
  [Fact]
  public void ItCanBeWrittenTo()
  {
    var inout = new MockInputOutput();

    // Act
    inout.Write("Text");
  }

  [Fact]
  public void WrittenTextCanBeRetrieved()
  {
    // Arrange
    var inout = new MockInputOutput();
    inout.Write("Text");

    // Act    
    var writtenText = inout.OutFeed;

    // Assert
    Assert.Single(writtenText);
    Assert.Equal("Text", writtenText.First());
  }
}
```

在文件 `WriteLineTests.cs` 中：

```cs
public class WriteLineTests
{
  [Fact]
  public void ItCanBeWrittenTo()
  {
    var inout = new MockInputOutput();

    // Act
    inout.WriteLine("Text");
  }

  [Fact]
  public void WrittenTextCanBeRetrieved()
  {
    // Arrange
    var inout = new MockInputOutput();
    inout.WriteLine("Text");

    // Act
    var writtenText = inout.OutFeed;

    // Assert
    Assert.Single(writtenText);
    Assert.Equal("Text" + Environment.NewLine, writtenText.First());
  }
}
```

在文件 `IInputOutput.cs` 中：

```cs
public interface IInputOutput
{
  void Write(string text);
  void WriteLine(string text);
  char Read();
  string ReadLine();
}
```

在文件 `MockInputOutput.cs` 中：

```cs
public class MockInputOutput : IInputOutput
{
  public List<string> OutFeed { get; set; }
  public Queue<string> InFeed { get; set; }

  public MockInputOutput()
  {
    OutFeed = new List<string>();
    InFeed = new Queue<string>();
  }

  public void Write(string text)
  {
    OutFeed.Add(text);
  }

  public void WriteLine(string text)
  {
    OutFeed.Add(text + Environment.NewLine);
  }

  public char Read()
  {
    return InFeed.Dequeue().ToCharArray().First();
  }

  public string ReadLine()
  {
    return InFeed.Dequeue();
  }
}
```

这处理了我们的模拟输入和输出，但我们需要创建控制台的生产包装器类，并需要使用 `Program.cs` 将该类注入到 `Mastermind` 类中。

# 非预期的输入

在将控制台的调用替换为对我们注入的类的调用时，我们发现了一些我们没有计划到的用例。第一个用例相当复杂，有几个参数我们需要处理：

```cs
Console.Write("+", c = c + 1);
```

第二个用例更简单，不需要任何参数：

```cs
Console.WriteLine();
```

第二个用例处理起来最简单，所以现在让我们为它编写一个快速测试。

在文件 `WriteLineTests.cs` 中：

```cs
[Fact]
public void ItCanWriteABlankLine()
{
  // Arrange
  var inout = new MockInputOutput();

  // Act
  inout.WriteLine();

  // Assert
  Assert.Single(inout.OutFeed);
  Assert.Equal(Environment.NewLine, inout.OutFeed.First());
}
```

在文件 `IInputOutput.cs` 中：

```cs
void WriteLine(string text = null);
```

在文件 `MockInputOutput.cs` 中：

```cs
public void WriteLine(string text = null)
{
  OutFeed.Add((text ?? "") + Environment.NewLine);
}
```

下一个问题稍微复杂一些。如果我们想准确处理它，我们需要进行相当多的正则表达式和字符串操作。然而，我们不需要它“正确”；我们只需要它按照应用程序的预期工作。在唯一使用这个功能的情况下，应该放入正在写入的字符串中的值并没有放入。原始的开发者滥用了 `Console.Write` 的功能来减少 if 语句中的行数，从而避免使用括号。因此，为了让代码继续工作，我们只需要允许输入即可。一个简单的接口扩展应该能为我们提供这个功能。

在文件 `IInputOutput.cs` 中：

```cs
void Write(string text, params object[] args);
```

在文件 `MockInputOutput.cs` 中：

```cs
public void Write(string text, params object[] args)
{
  OutFeed.Add(text);
}
```

在应用程序代码中，我们可以完成我们的更改。以下是更新后的应用程序。

在文件 `ConsoleInputOutput.cs` 中：

```cs
public class ConsoleInputOutput : IInputOutput
{
  public void Write(string text, params object[] args)
  {
    Console.Write(text, args);
  }

  public void WriteLine(string text)
  {
    Console.WriteLine(text);
  }

  public char Read()
  {
    return Console.ReadKey().KeyChar;
  }

  public string ReadLine()
  {
    return Console.ReadLine();
  }
}
```

在文件 `Program.cs` 中：

```cs
class Program
{
  static void Main(string[] args)
  {
    var inout = new ConsoleInputOutput();
    var game = new Mastermind(inout);
    game.Play(args);
  }
}
```

在文件 `Mastermind.cs` 中：

```cs
public class Mastermind
{
  private readonly IInputOutput _inout;

  public Mastermind(IInputOutput inout)
  {
    _inout = inout;
  }

  public void Play(string[] args)
  {
    // Variable Declarations - Global??
    char[] g;
    char[] p = new[] { 'A', 'A', 'A', 'A' };
    int i = 0;
    int j = 0;
    int x = 0;
    int c = 0;

    // Initialize randomness
    Random rand = new Random(DateTime.Now.Millisecond);

    // Determine if a password was passed in?
    if (args.Length > 0 && args[0] != null) p = args[0].ToCharArray();
    else goto randomize_password; // Create a password if one was not 
    provided
    // Player move - guess the password
    guess: _inout.Write("Take a guess: ");
    g = _inout.ReadLine().ToArray();
    i = i + 1;
    if (g.Length != 4) goto wrong_size;
    if (g == p) goto success;
    x = 0;
    c = 0;

    // Check if the password provided by the player is correct
    check_loop:
    if (g[x] > 65 + 26) g[x] = (char)(g[x] - 32);
    if (g[x] == p[x]) _inout.Write("+", c = c + 1);
    else if (p.Contains(g[x])) _inout.Write("-");
    x = x + 1;
    if (x < 4) goto check_loop; // Still checking??
    _inout.WriteLine();
    if (c == 4) goto success; // Password must have been correct
    goto guess; // No correct, try again

    // Game over you win
    success: _inout.WriteLine("Congratulations you guessed the password 
    in " + i + " tries.");
    goto end;
    // Password guess was wrong size - Error Message
    wrong_size: _inout.WriteLine("Password length is 4.");
    goto guess;

    // Create a random password
    randomize_password: j = 0;
    password_loop: p[j] = (char)(rand.Next(6) + 65);
    j = j + 1;
    if (j < 4) goto password_loop;
    goto guess; // Start the game

    // Game is complete - exit
    end: _inout.WriteLine("Press any key to quit.");
    _inout.Read();
  }
}
```

快速运行测试确认应用程序正在正确工作：

```cs
Take a guess: AAAA
Take a guess: BBBB
Take a guess: CCCC
Take a guess: DDDD
+-++
Take a guess: DEDD
+++
Take a guess: DFDD
++++
Congratulations you guessed the password in 6 tries.

Press any key to quit.
```

现在，我们可以编写一个黄金标准或特性测试，以验证代码的所有部分是否正常工作。这个测试不会覆盖的唯一代码部分是随机密码生成：

```cs
public class GoldStandardTests
{
  [Fact]
  public void StandardTestRun()
  {
    // Arrange
    var inout = new MockInputOutput();
    var game = new Mastermind(inout);

    // Arrange - Inputs
    inout.InFeed.Enqueue("AAA");
    inout.InFeed.Enqueue("AAAA");
    inout.InFeed.Enqueue("ABBB");
    inout.InFeed.Enqueue("ABCC");
    inout.InFeed.Enqueue("ABCD");
    inout.InFeed.Enqueue("ABCF");
    inout.InFeed.Enqueue(" ");

    // Arrange - Outputs
    var expectedOutputs = new Queue<string>();
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("Password length is 4." + 
     Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("+");
    expectedOutputs.Enqueue("-");
    expectedOutputs.Enqueue("-");
    expectedOutputs.Enqueue("-");
    expectedOutputs.Enqueue(Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("+");
    expectedOutputs.Enqueue("+");
    expectedOutputs.Enqueue("-");
    expectedOutputs.Enqueue("-");
    expectedOutputs.Enqueue(Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("+");
    expectedOutputs.Enqueue("+");
    expectedOutputs.Enqueue("+");
    expectedOutputs.Enqueue("-");
    expectedOutputs.Enqueue(Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("+");
    expectedOutputs.Enqueue("+");
    expectedOutputs.Enqueue("+");
    expectedOutputs.Enqueue(Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("+");
    expectedOutputs.Enqueue("+");
    expectedOutputs.Enqueue("+");
    expectedOutputs.Enqueue("+");
    expectedOutputs.Enqueue(Environment.NewLine);
    expectedOutputs.Enqueue("Congratulations you guessed the password 
     in 6 tries." + Environment.NewLine);
    expectedOutputs.Enqueue("Press any key to quit." + 
     Environment.NewLine);
    // Act
    game.Play(new[] { "ABCF" });

    // Assert
    inout.OutFeed.ForEach((text) =>
    {    
      Assert.Equal(expectedOutputs.Dequeue(), text);
    });
  }
}
```

这是一个极其长的测试，它具有非常规的结构，但这个单独的测试几乎涵盖了应用中的所有逻辑。你可能无法总是用一个单独的测试来完成这项工作，但在开始任何重大重构之前，这些测试必须存在。

# 弄清楚混乱

现在我们已经编写了黄金标准测试，我们可以开始安全地重构代码。任何试图做出的更改，如果破坏了黄金标准测试，都必须撤销，并采取新的方法。

看一下 `Mastermind` 类，`Play` 方法顶部所有这些变量都可以移动到类级别字段中。这将使它们对类内的所有代码都可用，并有助于弄清楚它们的作用以及它们在应用中使用的频率：

```cs
private char[] g;
private char[] p = new[] { 'A', 'A', 'A', 'A' };
private int i = 0;
private int j = 0;
private int x = 0;
private int c = 0;
```

接下来，我们将逐步向下工作到 `Play` 方法，将所有可以提取的内容提取到微小的私有方法中。在我们需要切换到修复这个应用程序中一些过时的逻辑之前，我们只能进行一些微小的重构：

```cs
public class Mastermind
{
  private readonly IInputOutput _inout;
  private char[] g;
  private char[] p = new[] { 'A', 'A', 'A', 'A' };
  private int i = 0;
  private int j = 0;
  private int x = 0;
  private int c = 0;

  public Mastermind(IInputOutput inout)
  {
    _inout = inout;
  }

  public void Play(string[] args)
  {
    // Determine if a password was passed in?
    if (args.Length > 0 && args[0] != null) p = args[0].ToCharArray();
    else CreateRandomPassword(); // Create a password if one was not 
    provided
    // Player move - guess the password
    guess:
    _inout.Write("Take a guess: ");
    g = _inout.ReadLine().ToArray();
    i = i + 1;
    if (g.Length != 4) goto wrong_size;
    if (g == p) goto success;
    x = 0;
    c = 0;

    // Check if the password provided by the player is correct
    check_loop:
    if (g[x] > 65 + 26) g[x] = (char)(g[x] - 32);
    if (g[x] == p[x]) _inout.Write("+", c = c + 1);
    else if (p.Contains(g[x])) _inout.Write("-");
    x = x + 1;
    if (x < 4) goto check_loop; // Still checking??
    _inout.WriteLine();
    if (c == 4) goto success; // Password must have been correct
    goto guess; // No correct, try again

    // Password guess was wrong size - Error Message
    wrong_size: _inout.WriteLine("Password length is 4.");
    goto guess;

    // Game over you win
    success: _inout.WriteLine("Congratulations you guessed the password 
     in " + i + " tries.");
    _inout.WriteLine("Press any key to quit.");
    _inout.Read();
  }
  private void CreateRandomPassword()
  {
    // Initialize randomness
    Random rand = new Random(DateTime.Now.Millisecond);

    j = 0;

    password_loop:
    p[j] = (char)(rand.Next(6) + 65);
    j = j + 1;   
    if (j < 4) goto password_loop;
  }
}
```

我们能够提取出一个密码生成方法。我们还能够简化成功代码的结构。然而，如果不解决所选循环结构的复杂性，我们无法继续前进。编写这个代码的开发者没有使用通用的循环结构，如 while 和 for 循环。我们需要修复这个问题，以便更好地理解和处理这段代码：

```cs
public void Play(string[] args)
{
  // Determine if a password was passed in?
  if (args.Length > 0 && args[0] != null) p = args[0].ToCharArray();
  else CreateRandomPassword(); // Create a password if one was not 
   provided
  // Player move - guess the password           
  while (c != 4)
  {
    _inout.Write("Take a guess: ");
    g = _inout.ReadLine().ToArray();

    i = i + 1;

    if (g.Length != 4)
    {
      // Password guess was wrong size - Error Message
      _inout.WriteLine("Password length is 4.");
    }
    else
    {
      // Check if the password provided by the player is correct
      for (x = 0, c = 0; g.Length == 4 && x < 4; x++)
      {
        if (g[x] > 65 + 26) g[x] = (char)(g[x] - 32);
        if (g[x] == p[x]) _inout.Write("+", c = c + 1);
        else if (p.Contains(g[x])) _inout.Write("-");
      }

      _inout.WriteLine();
    }
  }           

  // Game over you win
  _inout.WriteLine("Congratulations you guessed the password in " + i +   
  " tries.");
  _inout.WriteLine("Press any key to quit.");
  _inout.Read();
}
```

我们现在有一个可以开始工作的结构。让我们先弄清楚这些变量名：

```cs
C ~= Correct Letter Guesses
G ~= Current Guess
P ~= Password
I ~= Tries
X ~= Loop Index / Pointer to Guess Character being checked
J ~= Loop Index / Pointer to Password Character being generated
```

我们将想要更新 `Play` 方法，使其反映我们对变量含义的确定。以下我们将单个字母变量名替换为更恰当地表示变量用途的名称：

```cs
public void Play(string[] args)
{
  // Determine if a password was passed in?
  if (args.Length > 0 && args[0] != null) password =   
   args[0].ToCharArray();
  else CreateRandomPassword(); // Create a password if one was not 
   provided
  // Player move - guess the password           
  while (correctPositions != 4)
  {
    _inout.Write("Take a guess: ");
    guess = _inout.ReadLine().ToArray();

    tries = tries + 1;

    if (guess.Length != 4)
    {
      // Password guess was wrong size - Error Message
      _inout.WriteLine("Password length is 4.");
    }
    else
    {
      // Check if the password provided by the player is correct
      for (x = 0, correctPositions = 0; x < 4; x++)
      {
        if (guess[x] > 65 + 26) guess[x] = (char)(guess[x] - 32);
        if (guess[x] == password[x]) _inout.Write("+", correctPositions 
          = correctPositions + 1);
        else if (password.Contains(guess[x])) _inout.Write("-");
      }
      _inout.WriteLine();
    }
  }           
  // Game over you win
  _inout.WriteLine("Congratulations you guessed the password in " + 
   tries + " tries.");
  _inout.WriteLine("Press any key to quit.");
  _inout.Read();
}
```

接下来，如果我们现在能更新接口，那会很好，因为我们对应用程序有了更好的理解。我们想要改变的两件事是输入和游戏的最后阶段。如果输入是一个简单的字符串而不是字符数组，那就更好了。`Play` 方法可以接受一个字符串，程序可以确定如何从参数中获取密码字符串。

沿着同样的思路，我们可以减少总的写入次数，并将连续的加号和减号 `Write` 命令转换成一个单独的 `WriteLine` 命令。这将破坏我们的黄金标准测试，但实际上不会改变代码的功能。它仍然会在一行上打印加号和减号。

要将猜测从字符数组转换为字符串，我们首先必须理解这一行正在发生什么：

```cs
if (guess[x] > 65 + 26) guess[x] = (char)(guess[x] - 32);
```

分析这一行，我们看到数字 `65`、`26` 和 `32`。如果你熟悉 ASCII 码，那么这些行可能对你来说是有意义的。数字 `65` 是字母字符在 ASCII 表中的起始点。英语字母表中共有 26 个字母。并且，在 "a" 和 "A" 之间有 32 个值。因此，可以假设这段代码是在对指定索引处的字符进行大写或小写转换。我们可以使用 C# 中的 `String.ToUpper()` 方法来近似实现这一点。

当我们在进行一些小的黄金标准更改时，我们还应该将 `Play` 方法的最后两行移到 `Program.cs` 中，因为它们与控制台应用程序更相关。

在文件 `Program.cs` 中：

```cs
class Program
{
  static void Main(string[] args)
  {
    var inout = new ConsoleInputOutput();
    var game = new Mastermind(inout);

    var password = args.Length > 0 ? args[0] : null;
    game.Play(password);

    inout.WriteLine("Press any key to quit.");
    inout.Read();
  }
}
```

在文件 `Mastermind.cs` 中：

```cs
public class Mastermind
{
  private readonly IInputOutput _inout;
  private string guess;
  private int tries;
  private int correctPositions;

  public Mastermind(IInputOutput inout)
  {
    _inout = inout;
  }

  public void Play(string password = null)
  {
    // Determine if a password was passed in?
    password = password ?? CreateRandomPassword();           

    // Player move - guess the password           
    while (correctPositions != 4)
    {
      _inout.Write("Take a guess: ");
      guess = _inout.ReadLine();
      tries = tries + 1;

      if (guess.Length != 4)
      {
        // Password guess was wrong size - Error Message
        _inout.WriteLine("Password length is 4.");
      }
      else
      {
        // Check if the password provided by the player is correct
        guess = guess.ToUpper();
        var guessResult = "";

        for (var x = 0; x < 4; x++)
        {
          if (guess[x] == password[x])
          {                            
            guessResult += "+";
          }
          else if (password.Contains(guess[x]))
          {
            guessResult += "-";
          }
        }

        correctPositions = guessResult.Count(c => c == '+');
        _inout.WriteLine(guessResult);
      }
    }

    // Game over you win
    _inout.WriteLine("Congratulations you guessed the password in " + 
     tries + " tries.");           
  }

  private string CreateRandomPassword()
  {
    // Initialize randomness
    Random rand = new Random(DateTime.Now.Millisecond);

    var password = new [] {'A', 'A', 'A', 'A'};

    var j = 0;

    password_loop:
    password[j] = (char)(rand.Next(6) + 65);
    j = j + 1;

    if (j < 4) goto password_loop;
    return password.ToString();
  }
}
```

在文件 `GoldStandardTests.cs` 中：

```cs
public class GoldStandardTests
{
  [Fact]
  public void StandardTestRun()
  {
    // Arrange
    var inout = new MockInputOutput();
    var game = new Mastermind(inout);

    // Arrange - Inputs
    inout.InFeed.Enqueue("AAA");
    inout.InFeed.Enqueue("AAAA");
    inout.InFeed.Enqueue("ABBB");
    inout.InFeed.Enqueue("ABCC");
    inout.InFeed.Enqueue("ABCD");
    inout.InFeed.Enqueue("ABCF");
    inout.InFeed.Enqueue(" ");

    // Arrange - Outputs
    var expectedOutputs = new Queue<string>();
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("Password length is 4." + 
    Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("+---" + Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("++--" + Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("+++-" + Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("+++" + Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("++++" + Environment.NewLine);
    expectedOutputs.Enqueue("Congratulations you guessed the password 
    in 6 tries." + Environment.NewLine);

    // Act
    game.Play("ABCF");

    // Assert
    inout.OutFeed.ForEach(text =>
    {
      Assert.Equal(expectedOutputs.Dequeue(), text);
    });
  }
}
```

# 最终美化

现在所有其他工作都已完成，代码也运行正确，是时候在我们开始增强之前进行最后的重构了。我们希望方法尽可能小。在这种情况下，这意味着 `Play` 函数应该几乎没有任何逻辑在主游戏循环之外。

通常，如果一个方法中包含任何类型的块（例如，if、while、for 等），我们希望该块是方法中唯一的内容。通常，还有检查输入的守卫语句，但应该仅此而已。让我们重构以遵循该约定，并看看重构后的代码是什么样子。

在文件 `Mastermind.cs` 中：

```cs
public class Mastermind
{
  private readonly IInputOutput _inout;
  private int _tries;

  public Mastermind(IInputOutput inout)
  {
    _inout = inout;
  }

  public void Play(string password = null)
  {
    password = password ?? CreateRandomPassword();
    var correctPositions = 0;

    while (correctPositions != 4)
    {
      correctPositions = GuessPasswordAndCheck(password);
    }

    _inout.WriteLine("Congratulations you guessed the password in " + 
    _tries + " tries.");
  }

  private int GuessPasswordAndCheck(string password)
  {
    var guess = Guess();                        
    return Check(guess, password);
  }

  private int Check(string guess, string password)
  {
    var checkResult = "";

    for (var x = 0; x < 4; x++)
    {
      if (guess[x] == password[x])
      {
        checkResult += "+";
      }
      else if (password.Contains(guess[x]))
      {
        checkResult += "-";
      }
    }

    _inout.WriteLine(checkResult);
    return checkResult.Count(c => c == '+');
  }

  private string Guess()
  {
    _tries = _tries + 1;

    _inout.Write("Take a guess: ");
    var guess = _inout.ReadLine();

    if (guess.Length == 4)
    {
      return guess.ToUpper();
    }

    // Password guess was wrong size - Error Message
    _inout.WriteLine("Password length is 4.");
    return Guess();
  }

  private string CreateRandomPassword()
  {
    // Initialize randomness
    Random rand = new Random(DateTime.Now.Millisecond);

    var password = new[] { 'A', 'A', 'A', 'A' };

    var j = 0;

    password_loop:
    password[j] = (char)(rand.Next(6) + 65);
    j = j + 1;

    if (j < 4) goto password_loop;

    return password.ToString();
  }
}
```

这段代码可以有多种重构方式；这只是其中一种。现在代码已经重构，我们准备继续前进并开始进行增强。

# 准备进行增强

我们现在到达了一个点，代码已经足够清晰，我们可以开始处理我们的变更请求了。我们已经将随机密码生成部分的代码拆分成了它自己的方法，因此现在我们可以独立地工作在这个方法上。

我们需要做的第一件事是停止使用 `Random`。`Random` 本质上是不可预测的，并且不受我们的控制。我们需要一种方法来提供数字生成，以验证当 `Random` 提供特定输入时，我们能否得到预期的输出。

我们将提取一个接口和模拟类，类似于我们为 Console 所做的。以下是创建的第一轮测试、模拟类和接口。

在文件 `RandomNumberTests.cs` 中：

```cs
public class RandomNumberTests
{
  private readonly MockRandomGenerator _rand;

  public RandomNumberTests()
  {
    _rand = new MockRandomGenerator();
  }

  [Fact]
  public void ItExists()
  {
    _rand.Number();
  }

  [Fact]
  public void ItReturnsDefaultValue()
  {
    // Act
    var result = _rand.Number();

    // Assert
    Assert.Equal(0, result);
  }

  [Fact]
  public void ItCanReturnPredeterminedNumbers()
  {
    // Arrange
    _rand.SetNumbers(1, 2, 3, 4, 5);

    // Act
    var a = _rand.Number();
    var b = _rand.Number();
    var c = _rand.Number();
    var d = _rand.Number();
    var e = _rand.Number();

    // Arrange
    Assert.Equal(1, a);
    Assert.Equal(2, b);
    Assert.Equal(3, c);
    Assert.Equal(4, d);
    Assert.Equal(5, e);
  }

  [Fact]
  public void ItCanHaveAMaxRange()
  {
    // Arrange
    const int maxRange = 3;
    _rand.SetNumbers(1, 2, 3, 4, 5);

    // Act
    var a = _rand.Number(maxRange);
    var b = _rand.Number(maxRange);
    var c = _rand.Number(maxRange);
    var d = _rand.Number(maxRange);
    var e = _rand.Number(maxRange);

    // Arrange
    Assert.Equal(1, a);
    Assert.Equal(2, b);
    Assert.Equal(3, c);
    Assert.Equal(3, d);
    Assert.Equal(3, e);
  }

  [Fact]
  public void ItCanHaveAMinMaxRange()
  {
    // Arrange
    const int minRange = 2;
    const int maxRange = 3;
    _rand.SetNumbers(1, 2, 3, 4, 5);

    // Act
    var a = _rand.Number(minRange, maxRange);
    var b = _rand.Number(minRange, maxRange);
    var c = _rand.Number(minRange, maxRange);
    var d = _rand.Number(minRange, maxRange);
    var e = _rand.Number(minRange, maxRange);

    // Arrange
    Assert.Equal(2, a);
    Assert.Equal(2, b);
    Assert.Equal(3, c);
    Assert.Equal(3, d);
    Assert.Equal(3, e);
  }
}
```

在文件 `IRandomGenerator.cs` 中：

```cs
public interface IRandomGenerator
{
  int Number(int max = 100);
  int Number(int min, int max);
}
```

在文件 `MockRandomGenerator.cs` 中：

```cs
public class MockRandomGenerator : IRandomGenerator
{
  private readonly List<int> _numbers;
  private List<int>.Enumerator _numbersEnumerator;

  public MockRandomGenerator(List<int> numbers = null)
  {
    _numbers = numbers ?? new List<int>();
    _numbersEnumerator = _numbers.GetEnumerator();
  }

  public int Number(int min, int max)
  {
    var result = Number(max);

    return result < min ? min : result;
  }

  public int Number(int max = 100)
  {
    _numbersEnumerator.MoveNext();
    var result = _numbersEnumerator.Current;

    return result > max ? max : result;
  }

  public void SetNumbers(params int[] args)
  {
    _numbers.AddRange(args);
    _numbersEnumerator = _numbers.GetEnumerator();
  }
}
```

现在，创建生产 `RandomGenerator` 类并将其注入到我们的应用程序中。

在文件 `RandomGenerator.cs` 中：

```cs
public class RandomGenerator : IRandomGenerator
{
  private readonly Random _rand;

  public RandomGenerator()
  {
    _rand = new Random();
  }

  public int Number(int max = 100)
  {
    return _rand.Next(0, max);
  }

  public int Number(int min, int max)
  {
    return _rand.Next(min, max);
  }
}
```

在文件 `Program.cs` 中：

```cs
class Program
{
  static void Main(string[] args)
  {
    var rand = new RandomGenerator();
    var inout = new ConsoleInputOutput();
    var game = new Mastermind(inout, rand);

    var password = args.Length > 0 ? args[0] : null;
    game.Play(password);

    inout.WriteLine("Press any key to quit.");
    inout.Read();
  }
}
```

在文件 `Mastermind.cs` 中：

```cs
public class Mastermind
{
  private readonly IInputOutput _inout;
  private readonly IRandomGenerator _random;

  private int _tries;

  public Mastermind(IInputOutput inout, IRandomGenerator random)
  {
    _inout = inout;
    _random = random;
  }

  public void Play(string password = null)
  {
    password = password ?? CreateRandomPassword();
    var correctPositions = 0;

    while (correctPositions != 4)
    {
      correctPositions = GuessPasswordAndCheck(password);
    }

    _inout.WriteLine("Congratulations you guessed the password in " + 
    _tries + " tries.");
  }

  private int GuessPasswordAndCheck(string password)
  {
    var guess = Guess();
    return Check(guess, password);
  }

  private int Check(string guess, string password)
  {
    var checkResult = "";

    for (var x = 0; x < 4; x++)
    {
      if (guess[x] == password[x])
      {
        checkResult += "+";
      }
      else if (password.Contains(guess[x]))
      {
        checkResult += "-";
      }
    }

    _inout.WriteLine(checkResult);
    return checkResult.Count(c => c == '+');
  }

  private string Guess()
  {
    _tries = _tries + 1;

    _inout.Write("Take a guess: ");
    var guess = _inout.ReadLine();

    if (guess.Length == 4)
    {
      return guess.ToUpper();
    }

    // Password guess was wrong size - Error Message
    _inout.WriteLine("Password length is 4.");
    return Guess();
  }

  private string CreateRandomPassword()
  {
    var password = new[] { 'A', 'A', 'A', 'A' };

    var j = 0;

    password_loop:
    password[j] = (char)(_random.Number(6) + 65);
    j = j + 1;

    if (j < 4) goto password_loop;

    return new string(password);
  }
}
```

最后，让我们修改黄金标准测试以使用随机密码生成。

在文件 `GoldStandardTests.cs` 中：

```cs
public class GoldStandardTests
{
  [Fact]
  public void StandardTestRun()
  {
    // Arrange
    var inout = new MockInputOutput();
    var rand = new MockRandomGenerator();
    var game = new Mastermind(inout, rand);

    // Arrange - Inputs
    rand.SetNumbers(0, 1, 2, 5);
    inout.InFeed.Enqueue("AAA");
    inout.InFeed.Enqueue("AAAA");
    inout.InFeed.Enqueue("ABBB");
    inout.InFeed.Enqueue("ABCC");
    inout.InFeed.Enqueue("ABCD");
    inout.InFeed.Enqueue("ABCF");
    inout.InFeed.Enqueue(" ");

    // Arrange - Outputs
    var expectedOutputs = new Queue<string>();
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("Password length is 4." + 
    Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("+---" + Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("++--" + Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("+++-" + Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("+++" + Environment.NewLine);
    expectedOutputs.Enqueue("Take a guess: ");
    expectedOutputs.Enqueue("++++" + Environment.NewLine);
    expectedOutputs.Enqueue("Congratulations you guessed the password 
    in 6 tries." + Environment.NewLine);

    // Act
    game.Play();

    // Assert
    inout.OutFeed.ForEach(text =>
    {
      Assert.Equal(expectedOutputs.Dequeue(), text);
    });
  }
}
```

现在我们准备重构密码生成方法，并扩展它以提供所需的变化。首先，有一个不是语言核心的循环结构。让我们专注于 `CreateRandomPassword` 方法并修复循环结构：

```cs
private string CreateRandomPassword()
{
  var password = new[] { 'A', 'A', 'A', 'A' };

  for(var j = 0; j < 4; j++)
  {
    password[j] = (char)(_random.Number(6) + 65);
  }

  return new string(password);
}
```

接下来，为了好玩，让我们看看我们能否泛化和压缩这个循环，因为我们 `Check` 方法中有一个非常相似的循环。虽然这不是必要的，但这是一个减少代码重复的好例子。以下是重构的样子：

```cs
private int Check(string guess, string password)
{
  var checkResult = "";

  Times(4, x => {
    if (guess[x] == password[x])
    {
      checkResult += "+";
    }
    else if (password.Contains(guess[x]))
    {
      checkResult += "-";
    }
  });

  _inout.WriteLine(checkResult);
  return checkResult.Count(c => c == '+');
}

private string CreateRandomPassword()
{
  var password = new[] { 'A', 'A', 'A', 'A' };

  Times(4, x => password[x] = (char)(_random.Number(6) + 65));

  return new string(password);
}

private static void Times(int count, Action<int> act)
{
  for (var index = 0; index < count; index++)
  {
    act(index);
  }
}
```

现在我们扩展应用程序之前，让我们再进行一次重构。观察字符的生成方式，并不明显。相反，我们希望代码尽可能简单直接。没有理由随机生成器类不能直接返回字母，所以让我们添加这个功能。

在文件 `RandomLetterTests.cs` 中：

```cs
public class RandomLetterTests
{
  private readonly MockRandomGenerator _rand;

  public RandomLetterTests()
  {
    _rand = new MockRandomGenerator();
  }

  [Fact]
  public void ItExists()
  {
    _rand.Letter();
  }

  [Fact]
  public void ItReturnsDefaultValue()
  {
    // Act
    var result = _rand.Letter();

    // Assert
    Assert.Equal('A', result);
  }

  [Fact]
  public void ItCanReturnPredeterminedLetters()
  {
    // Arrange
    _rand.SetLetters('A', 'B', 'C', 'D', 'E');

    // Act
    var a = _rand.Letter();
    var b = _rand.Letter();
    var c = _rand.Letter();
    var d = _rand.Letter();
    var e = _rand.Letter();

    // Assert
    Assert.Equal('A', a);
    Assert.Equal('B', b);
    Assert.Equal('C', c);
    Assert.Equal('D', d);
    Assert.Equal('E', e);
  }

  [Fact]
  public void ItCanHaveAMaxRange()
  {
    // Arrange
    const char maxRange = 'C';
    _rand.SetLetters('A', 'B', 'C', 'D', 'E');

    // Act
    var a = _rand.Letter(maxRange);
    var b = _rand.Letter(maxRange);
    var c = _rand.Letter(maxRange);
    var d = _rand.Letter(maxRange);
    var e = _rand.Letter(maxRange);

    // Arrange
    Assert.Equal('A', a);
    Assert.Equal('B', b);
    Assert.Equal('C', c);
    Assert.Equal('C', d);
    Assert.Equal('C', e);
  }

  [Fact]
  public void ItCanHaveAMinMaxRange()
  {
    // Arrange
    const char minRange = 'B';
    const char maxRange = 'C';
    _rand.SetLetters('A', 'B', 'C', 'D', 'E');

    // Act
    var a = _rand.Letter(minRange, maxRange);
    var b = _rand.Letter(minRange, maxRange);
    var c = _rand.Letter(minRange, maxRange);
    var d = _rand.Letter(minRange, maxRange);
    var e = _rand.Letter(minRange, maxRange);

    // Arrange
    Assert.Equal('B', a);
    Assert.Equal('B', b);
    Assert.Equal('C', c);
    Assert.Equal('C', d);
    Assert.Equal('C', e);
  }
}
```

在文件 `MockRandomGenerator.cs` 中：

```cs
public class MockRandomGenerator : IRandomGenerator
{
  private readonly List<int> _numbers;
  private List<int>.Enumerator _numbersEnumerator;

  private readonly List<char> _letters;
  private List<char>.Enumerator _lettersEnumerator;

  private const char NullChar = '\0';

  public MockRandomGenerator(List<int> numbers = null, List<char> 
  letters = null)
  {
    _numbers = numbers ?? new List<int>();
    _numbersEnumerator = _numbers.GetEnumerator();

    _letters = letters ?? new List<char>();
    _lettersEnumerator = _letters.GetEnumerator();
  }

  public int Number(int min, int max)
  {
    var result = Number(max);

    return result < min ? min : result;
  }

  public int Number(int max = 100)
  {
    _numbersEnumerator.MoveNext();
    var result = _numbersEnumerator.Current;

    return result > max ? max : result;
  }

  public void SetNumbers(params int[] args)
  {
    _numbers.AddRange(args);
    _numbersEnumerator = _numbers.GetEnumerator();
  }

  public int Letter(char min, char max)
  {
    var result = Letter(max);

    return result < min ? min : result;
  }

  public char Letter(char max = 'Z')
  {
    _lettersEnumerator.MoveNext();           
    var result = _lettersEnumerator.Current;
    result = result == NullChar ? 'A' : result;

    return result > max ? max : result;
  }

  public void SetLetters(params char[] args)
  {
    _letters.AddRange(args);
    _lettersEnumerator = _letters.GetEnumerator();
  }
}
```

在文件 `IRandomGenerator.cs` 中：

```cs
public interface IRandomGenerator
{
  int Number(int max = 100);
  int Number(int min, int max);
  char Letter(char max = 'Z');
  char Letter(char min, char max);
}
```

在文件 `RandomGenerator.cs` 中：

```cs
public class RandomGenerator : IRandomGenerator
{
  private readonly Random _rand;

  public RandomGenerator()
  {
    _rand = new Random();
  }

  public int Number(int max = 100)
  {
    return Number(0, max);
  }

  public int Number(int min, int max)
  {
    return _rand.Next(min, max);
  }

  public char Letter(char max = 'Z')
  {
    return Letter('A', max);
  }

  public char Letter(char min, char max)
  {
    return (char) _rand.Next(min, max);
  }
}
```

在文件 `Mastermind.cs` 中：

```cs
private string CreateRandomPassword()
{
  var password = new[] { 'A', 'A', 'A', 'A' };

  Times(4, x => password[x] = _random.Letter('F'));

  return new string(password);
}
```

在文件 `GoldStandardTests.cs` 中：

```cs
// Arrange - Inputs
rand.SetLetters('A', 'B', 'C', 'F');
```

那就是这个练习的最终重构。我们只有一件事要做，那就是扩展应用程序以使用整个英语字母表的范围生成密码。由于我们在测试和重构上付出的努力，这现在是一件微不足道的事情，实际上只需要在 `Mastermind` 类中删除三个字符。

在文件 `Mastermind.cs` 中：

```cs
private string CreateRandomPassword()
{
  var password = new[] { 'A', 'A', 'A', 'A' };

  Times(4, x => password[x] = _random.Letter());

  return new string(password);
}
```

现在创建一个更复杂的密码，包含整个字母表的范围。这导致了一个更难的密码，并且一个输出类似于以下的游戏：

```cs
Take a guess: AAAA
Take a guess: BBBB
Take a guess: CCCC
---+
Take a guess: DDDC
+
Take a guess: EEEC
+
Take a guess: FFFC
+
Take a guess: GGGC
+
Take a guess: HHHC
+
Take a guess: IIIC
+
Take a guess: JJJC
+
Take a guess: KKKC
+
Take a guess: LLLC
+
Take a guess: mmmc
+
Take a guess: nnnc
+
Take a guess: oooc
+--+
Take a guess: oppc
++-+
Take a guess: opqc
+++
Take a guess: oprc
+++
Take a guess: opsc
+++
Take a guess: optc
+++
Take a guess: opuc
+++
Take a guess: opvc
+++
Take a guess: opwc
++++
Congratulations you guessed the password in 23 tries.

Press any key to quit.
```

# 摘要

你现在有一个编写良好的示例，由测试覆盖。所涉及的努力可能令人畏惧，但对于任何非平凡的应用程序来说，这可能非常值得。

在 第十四章，《更好的起点》，我们将总结我们所学的知识，并给你一些如何作为 TDD 专家重新融入世界的建议。
