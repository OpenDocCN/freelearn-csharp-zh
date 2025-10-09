

# 文件系统编年史

*文件系统* 和 IO

计算机是令人难以置信的机器，但它们有一个缺点。如果电源关闭，它们会忘记一切。如果我们不希望丢失我们的工作，我们必须将其存储在其他地方。我们可以打印数据，将其放在网络上，或者存储在永久存储中。这是最常见的选项。当然，我们需要一种方法将数据输入 CPU。我们可以从文件或网络中读取数据。我们甚至可以使用键盘输入数据。这是我们（程序员）和我（作者）都非常熟悉的事情。

当我们编写软件时，我们提到 **流** 的概念。流表示随时间提供的一系列数据元素。这个序列可以存储在磁盘上，可以是通过网络线缆流动的数据，也可以是内存芯片的状态。无论我们使用什么物理介质，数据都必须来回流动。这一章处理这个主题，涵盖流、文件以及其他 **输入和输出**（**IO**）的方式。

在本章中，我们将不会深入探讨的主题是网络。网络是一个如此不同的概念，以至于将有一个单独的章节来处理这个主题。您可以在 *第八章* 中找到所有低级网络细节。然而，通过网络处理数据的概念对于文件和其他媒体是相同的。因此，这里阐述的原则仍然适用。

在本章中，我们将涵盖以下主题：

+   如何使用 .NET 处理文件

+   如何使用 Win32 API 与文件系统交互

+   如何处理目录和路径

+   为什么以及如何使用异步 IO

+   如何使用加密和压缩

我们有很多内容要覆盖，所以让我们深入探讨吧！

# 技术要求

要查看本章中的所有代码，您可以访问以下链接：[`github.com/PacktPublishing/Systems-Programming-with-C-Sharp-and-.NET/tree/main/SystemsProgrammingWithCSharpAndNet/Chapter05`](https://github.com/PacktPublishing/Systems-Programming-with-C-Sharp-and-.NET/tree/main/SystemsProgrammingWithCSharpAndNet/Chapter05)。

# 文件写入基础

没有什么比写入文件更直接的了，对吧？这就是为什么我认为这是一个好的起点。以下是实现这一点的代码：

```cs
var path = System.IO.Path.GetTempPath();
var fileName = "WriteLines.txt";
var fullPath = Path.Combine(path, fileName);
File.WriteAllText(fullPath, "Hello, System Programmers");
```

第一行获取系统 `temp` 路径。然后我们指定文件名，将其添加到 `temp` 路径中，并向该文件写入一行文本。

这个例子足够简单，但它已经展示了某些有用的内容。首先，我们可以快速访问 `temp` 文件夹；我们不需要在我们的代码中指定它的位置。其次，我们可以组合文件名和路径，而不用担心路径分隔符。在 Windows 上，路径的部分由反斜杠分隔，而在 Linux 上，这是一个正斜杠。CLR 会确定应该使用什么，并使用正确的一个。

`File.WriteAllText` 然后使用这些数据创建一个文件，打开它，写入字符串，然后关闭文件。如果文件已经存在，系统将覆盖它。

如果我们想要使用临时文件名而不是 `WriteLines.Text`，代码可以更加简单：

```cs
var path = System.IO.Path.GetTempFileName();
File.WriteAllText(path, "Hello, System Programmers");
```

系统会查找 `temp` 文件的路径，生成一个具有唯一文件名的新的文件，并使用该文件写入字符串。缺点是现在我们不知道它是哪个文件。我们必须将其记录在某个地方；否则，我们的 `temp` 文件夹会很快填满未使用的文件（尽管大多数操作系统都会清理 `temp` 文件夹，所以这里没有真正的担忧）。

您当然可以使用任何您想要的文件夹。然而，如果您想使用一些特殊文件夹，例如 Windows 上的 `Documents` 文件夹，系统也可以帮助您访问这些文件夹。看看以下代码片段：

```cs
var path = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
var fileName = "WriteLines.txt";
var fullPath = Path.Combine(path, fileName);
File.WriteAllText(fullPath, "Hello, System Programmers");
```

这段代码查找我的机器上 `My Documents` 的位置，并返回该位置，以便我可以将文件写入该位置。您可以从一个很长的特殊位置列表中进行选择，所有这些位置都是 `SpecialFolder` 枚举的一部分。我不会列出所有这些位置；您可以在以下链接中找到它们：[`learn.microsoft.com/en-us/dotnet/api/system.environment.specialfolder?view=net-8.0`](https://learn.microsoft.com/en-us/dotnet/api/system.environment.specialfolder?view=net-8.0)。

这种写文件的方式毫不费力。然而，正如我们之前多次看到的，方便往往伴随着控制力的减少。作为系统程序员，我们希望获得尽可能多的控制权。让我们夺回一些控制权。

## FileStream

静态的 `File` 类易于使用，如果您想快速向文件写入或从文件读取内容，它非常方便。然而，它并不是最快的方式。至少，如果我们谈论执行时间，它不是最快的。作为系统程序员，我们非常关注速度，即使这意味着放弃编码的便利性。

以下示例比之前的示例快约 20%，但它执行的是相同的事情。它只需要几行额外的代码：

```cs
var fileName = Path.GetTempFileName();
var info = new UTF8Encoding(true).GetBytes("Hello, System Developers!");
using FileStream? fs = File.Create(fileName, info.Length);
try
{
    fs.Write(info, 0, info.Length);
}
finally
{
    fs.Close();
}
```

这个示例使用了 `File.Create()` 返回的 `FileStream`。我们当然可以自己创建一个。将创建 `FileStream` 的行替换为以下内容：

```cs
using var fs = new FileStream(
    path: fileName,
    mode: FileMode.Create,
    access: FileAccess.Write,
    share: FileShare.None,
    bufferSize:0x1000,
    options: FileOptions.Asynchronous);
```

我在这里使用了最全面的重载来向您展示您可以使用的部分选项。大多数选项都是不言自明的，但我想要强调两个参数：**share** 和 **options**。

`Share` 是一个标志，用于告诉操作系统在使用文件时如何共享文件。它有以下选项：

| **标志** | **值** | **描述** |
| --- | --- | --- |
| `None` | 0 | 不允许共享。任何尝试访问文件的进程都将失败。 |
| `Read` | 1 | 当我们仍在使用文件时，其他进程可以读取文件。 |
| `Write` | 2 | 其他进程可能同时向文件写入。 |
| `ReadWrite` | 3 | 这结合了 `Read` 和 `Write` 标志。 |
| `Delete` | 4 | 这允许我们在使用文件的同时请求删除文件。 |
| `Inheritable` | 16 | 文件句柄可以被子进程继承。然而，这在 Win32 应用程序中不起作用。 |

表 5.1：文件共享选项

虽然指定这个列表中的标志可能表明其他进程在我们使用文件时可以对我们文件进行操作，但并不能保证这些其他进程实际上可以这样做。通常，它们还需要其他权限。

`Delete`是一个很好的标志。它允许我们在使用文件的同时进行删除。这可能会导致奇怪的情况。如果我们创建一个文件并指定我们允许删除，我们可能在另一个进程已经删除文件的情况下向文件写入。系统不会抱怨并继续运行。然而，你最终会失去那个文件，这意味着永远失去了你的数据。让我给你展示一下我的意思：

```cs
using System.Text;
var fileName = Path.GetTempFileName();
var info = new UTF8Encoding(true).GetBytes("Hello fellow System Developers!");
using (var fs = new FileStream(
    path: fileName,
    mode: FileMode.Create,
    access: FileAccess.Write,
    share: FileShare.Delete, // We allow other processes to delete the                               //file.
    bufferSize: 0x1000,
    options: FileOptions.Asynchronous))
{
    try
    {
        fs.Write(info, 0, info.Length);
        Console.WriteLine($"Wrote to the file. Now try to delete it.             You can find it here:\n{fileName}");
        Console.ReadKey();
        fs.Write(info);
        Console.WriteLine("Done with all the writing");
        Console.ReadKey();
    }
    finally
    {
        fs.Close();
    }
}
Console.WriteLine("Done.");
Console.ReadKey();
```

这个例子很简单。我们首先获取一个临时文件名。然后，我们获取构成有效载荷的字节。之后，我们将创建一个`FileStream`实例，在创建过程中设置一些属性。

其中之一是`Share`选项。我们将其设置为`FileShare.Delete`。

我们将向文件写入一些数据，然后暂停程序。如果你运行它，这就是你获取输出的时候，它会告诉你文件的名称和位置，然后将其删除。你应该注意到你可以这样做。然后继续程序。正如你所看到的，下一行再次将相同的数据写入我们刚刚删除的文件。什么也没有发生。真的，什么也没有发生。没有错误，也没有任何数据被写入任何地方。

在大多数情况下，这是你想要避免的行为。然而，也许你的用例需要的就是这种行为。在这种情况下，现在你知道如何做到这一点。

## 更快的是 – Win32

存在一种更快的方式来写入文件。如果我们移除了 CLR 的开销，我们可以将文件写入速度提高大约 20%。速度提高 20%可能意味着一个运行缓慢的应用程序和一个看起来闪电般快速的应用程序之间的区别。通常，这会带来一定的代价。CLR 为我们提供的一切现在都掌握在我们自己的手中。我们必须做更多的工作。然而，如果你在寻找将数据写入文件的最快方式，Win32 方法再次是做这件事的最佳方式。

我们将首先声明一些常量：

```cs
private const uint GENERIC_WRITE = 0x40000000;
private const uint CREATE_ALWAYS = 0x00000002;
private const uint FILE_APPEND_DATA = 0x00000004;
```

`GENERIC_WRITE`告诉系统我们想要写入文件。`CREATE_ALWAYS`指定每次调用此方法时都想要创建一个新文件。`FILE_APPEND_DATA`意味着我们想要向当前文件添加内容（这没有太多意义，因为我们刚刚创建了文件）。

是时候导入 Win32 API 了：

```cs
[DllImport("kernel32.dll", SetLastError = true)]
private static extern SafeFileHandle CreateFile(
    string lpFileName,
    uint dwDesiredAccess,
    uint dwShareMode,
    IntPtr lpSecurityAttributes,
    uint dwCreationDisposition,
    uint dwFlagsAndAttributes,
    IntPtr hTemplateFile);
[DllImport("kernel32.dll", SetLastError = true)]
[return: MarshalAs(UnmanagedType.Bool)]
private static extern bool WriteFile(
    SafeFileHandle hFile,
    byte[] lpBuffer,
    uint nNumberOfBytesToWrite,
    out uint lpNumberOfBytesWritten,
    IntPtr lpOverlapped);
[DllImport("kernel32.dll", SetLastError = true)]
[return: MarshalAs(UnmanagedType.Bool)]
private static extern bool CloseHandle(SafeFileHandle hObject);
```

我们将从`kernel32.dll`导入三个方法。`CreateFile`创建一个文件，`WriteFile`向该文件写入，而`CloseHandle`关闭句柄，在我们的例子中，是文件的句柄。

这就是我们需要的所有内容。让我给你展示它是如何工作的：

```cs
public void WriteToFile(string fileName, string textToWrite)
{
    var fileHandle = CreateFile(
        fileName,
        GENERIC_WRITE,
        0,
        IntPtr.Zero,
        CREATE_ALWAYS,
        FILE_APPEND_DATA,
        IntPtr.Zero);
    if (!fileHandle.IsInvalid)
        try
        {
            var bytes = Encoding.ASCII.GetBytes(textToWrite);
            var writeResult = WriteFile(
                fileHandle,
                bytes,
                (uint)bytes.Length,
                out var bytesWritten,
                IntPtr.Zero);
        }
        finally
        {
            // Always close the handle once you are done
            CloseHandle(fileHandle);
        }
    else
        Console.WriteLine("Failed to open file.");
}
```

根据你现在的知识，你应该能够跟上。我们首先将创建一个具有正确参数的文件。如果这成功了，我们将得到我们想要写入的字节，然后使用`WriteFile`进行实际的写入。之后，我们将关闭句柄。我们在`finally`块中这样做；句柄很昂贵，并且锁定对文件的访问。我们希望关闭它，以便其他进程可以访问文件。

如果你认为这看起来还不错，你部分是正确的。这非常简单。然而，我省略了很多东西，比如错误检查。你还记得我之前关于性能的讨论吗？我在前面的章节中说，与正常的 CPU 操作相比，文件 I/O 要慢得多。因此，我们必须尽可能多地使用异步方法。你可以用 Win32 做到这一点，但这相当复杂。在这里，我不会向你展示如何做，但如果你在 Win32 API、`CreateFile`和`FILE_FLAG_OVERLAPPED`上快速搜索，你可以了解它是如何工作的。简而言之，你将不得不自己检查一切。我的建议是坚持使用 CLR 函数。我们将在本章后面讨论异步 I/O。

我们已经学习了如何写入文件以及与之相关的所有内容。然而，这仅仅是故事的一部分。让我们转向等式的另一半：读取文件。

# 文件读取基础

太棒了。我们已经写了一个文件。现在我们应该能够读取它，对吧？好的，让我们深入探讨一下。我们将从一个简单的例子开始：一个包含一些文本行的文件，我们希望将其读取到字符串中：

```cs
public string ReadFromFile(string fileName)
{
    var text = File.ReadAllText(fileName);
    return text;
}
```

我无法使它比这更简单了。我们有一个静态的`ReadAllText`方法，它接受一个文件名并将所有文本读取到字符串中。然后我们返回它。请记住，并非所有文件都包含文本。我甚至敢说*大多数*文件都不包含文本。它们是二进制的。现在，从技术上讲，一个`text`文件也是一个`binary`文件。所以，让我们再次读取文件，但现在是通过读取实际的字节。这次我使用`FileStream`，这样我们就可以对发生的事情有更多的控制：

```cs
public string ReadWithStream(string fileName)
{
    byte[] fileContent;
    using (FileStream fs = File.OpenRead(fileName))
    {
        fileContent = new byte[fs.Length];
        fs.Read(fileContent, 0, (int)fs.Length);
        fs.Close();
    }
    return Encoding.ASCII.GetString(fileContent);
}
```

`FileStream`的好处是它知道流的长度。这意味着我们可以为我们的数组分配足够的空间来包含所有数据。

我们将通过一次调用`fs.Read()`来读取所有数据，给它一个字节数组，起始位置`0`，以及要读取的总字节数。再次强调，当我们完成时，我们将关闭流。

最后，我们将假设内容是 ASCII 字符，将文件转换为字符串。

如果文件相对较小，这种方式读取是可行的。在这种情况下，你可以一次性读取所有内容。然而，如果文件太大，你必须分块读取。

为了做到这一点，`Read()`方法通过告诉你它读取了多少数据来帮助你。你可以创建一个循环并遍历整个文件。

我们可以像这样重写读取文件的部分：

```cs
fileContent = new byte[fs.Length];
int i = 0;
int bytesRead=0;
do
{
    var myBuffer = new byte[1];
    bytesRead = fs.Read(myBuffer, 0, 1);
    if(bytesRead > 0)
        fileContent[i++] = myBuffer[0];
}while(bytesRead > 0);
fs.Close();
```

这是一个愚蠢的方法，但它说明了我的观点。我们将继续读取文件，直到我们有了所有数据，在这种情况下 `fs.Read()` 返回 `0`。

## 读取二进制数据

如果你有一个你知道结构的 `binary` 文件，你可以使用 `BinaryReader` 来帮助。

二进制数据通常比文本数据更节省内存。由于我们作为系统程序员总是在寻找更高效的代码，这值得一看。

假设我有一个以下这样的类。这并没有什么特殊的意义；它只是一个数据集合：

```cs
class MyData
{
    public int Id { get; set; }
    public double SomeMagicNumber { get; set; }
    public bool IsThisAGoodDataSet { get; set; }
    public MyFlags SomeFlags { get; set; }
    public string? SomeText { get; set; }
}
[Flags]
public enum MyFlags
{
    FlagOne,
    FlagTwo,
    FlagThree
}
```

假设我已经创建了一个具有属性 `42`、`3.1415`、`True`、`MyFlags.One | MyFlags.Three` 和 `Hello, Systems Programmers` 的该类实例。我可以使用 JSON 序列化将其写入文件。这会产生一个 114 字节的文件。如果使用二进制格式，我可以将其缩小到 44 字节。这是一个相当大的节省，尤其是在将数据放在网络上时。

使用 `BinaryReader` 类读取该文件是直接的。让我给你展示一下：

```cs
public MyData Read(string fileName)
{
    var myData = new MyData();
    using var fs = File.OpenRead(fileName);
    try
    {
        using BinaryReader br = new(fs);
        myData.Id = br.ReadInt32();
        myData.IsThisAGoodDataSet = br.ReadBoolean();
        myData.SomeMagicNumber = br.ReadDouble();
        myData.SomeFlags = (MyFlags)br.ReadInt32();
        myData.SomeText = br.ReadString();
    }
    finally
    {
        fs.Close();
    }
    return myData;
}
```

这样做意味着你必须非常小心。你必须精确地知道文件的结构。你必须负责以正确的顺序获取所有数据，并确切地知道每个字段的类型。然而，这样做可以确保效率，并且可以节省你许多 CPU 周期。

我们现在已经了解了如何读取和写入文件。然而，文件系统中的不仅仅是文件。我们需要一种方法来组织所有这些文件。这把我们带到了 IO 的下一个主题：目录！

# 目录操作

想象一下有一个只有一个根文件夹的文件系统。你的驱动器上的所有文件都存储在那里。你将很难找到所有文件。幸运的是，操作系统都支持文件夹或目录的概念。CLR 通过给我们提供两个类来帮助我们处理路径、文件夹和目录：**Path** 和 **Directory**。

## 路径类

`Path` 是一个具有处理路径的辅助方法的类。我指的是表示目录名称的字符串。当处理实际的目录和文件时，你应该使用 `Directory` 类。

我们已经在之前的示例中看到了 `Path` 类。我使用它来获取临时文件名和 `Documents` 文件夹的名称。我还用它来合并路径和文件名，以避免自己处理路径分隔符。

`Path` 类中有许多实用的方法和属性。你可以在以下表中看到一些最常用的。

| **方法** | **描述** |
| --- | --- |
| `Path.Combine` | 将两个或多个字符串合并为一个路径 |
| `Path.GetFileName` | 返回指定路径字符串的文件名和扩展名 |
| `Path.GetFileNameWithoutExtension` | 返回指定路径字符串的文件名，不带扩展名 |
| `Path.GetExtension` | 获取指定路径字符串的扩展名（包括点） |
| `Path.GetDirectoryName` | 获取指定路径字符串的目录信息 |
| `Path.GetFullPath` | 将相对路径转换为绝对路径 |
| `Path.GetTempPath` | 返回系统临时文件夹的路径 |
| `Path.GetRandomFileName` | 返回一个尚未被使用的随机文件名 |
| `Path.GetInvalidFileNameChars` | 返回当前平台上不允许在文件名中使用的字符数组 |
| `Path.GetInvalidPathChars` | 返回当前平台上不允许在路径字符串中使用的字符数组 |
| `Path.ChangeExtension` | 改变文件路径的扩展名 |
| `Path.HasExtension` | 确定路径是否包含文件名扩展名 |
| `Path.IsPathRooted` | 获取一个值，指示指定的路径字符串是否包含根 |
| `Path.DirectorySeparatorChar` | 用于路径字符串的平台特定分隔符 |

表 5.2：Path 类及其方法和属性

如您所见，`Path` 类有一组非常好用且方便的辅助工具。当我们调查其他平台时，我们还会遇到它们，但现在，请记住尽可能多地使用它们。

## 目录类

`Directory` 类处理您文件系统中的实际目录。这个类与 `Path` 类紧密合作。如果您需要指定目录的名称（以及其位置），您将使用 `Path` 类。

假设我们想在 Windows 机器上的 `Pictures` 文件夹中列出所有图像。您会这样做：

```cs
var imagesPath =
Environment.GetFolderPath(Environment.SpecialFolder.MyPictures);
string[] allFiles =
    Directory.GetFiles(
        path: imagesPath,
        searchPattern: "*.jPg",
        searchOption: SearchOption.AllDirectories);
foreach (string file in allFiles)
{
    Console.WriteLine(file);
}
```

我在这里使用 `Environment.SpecialFolder.MyPictures` 来标识包含所有图片的文件夹。实际的路径取决于您的操作系统、用户名以及您如何设置您的机器。这意味着有很多可能的变体，但我们不必过于担心这一点。让操作系统去处理这个问题，只要我们得到正确的文件夹即可。

我使用了 `Directory.GetFiles()` 方法来遍历该文件夹。我想获取所有子文件夹中收集的所有 JPEG 图像。注意我在 `searchPattern` 变量中如何拼写扩展名：`*.jPg`。在 Windows 上，文件名不区分大小写。在 Linux 上，它们是区分大小写的。因此，在基于 Linux 的机器上，这不会起作用。好吧，它将起作用，但它不会返回您可能期望得到的所有文件。不幸的是，`GetFiles()` 无法设置不区分大小写的过滤器。如果您想获取所有 JPG 图像，无论它们的扩展名看起来如何，您必须以另一种方式来做：

```cs
var regex = new Regex(@"\.jpe?g$", RegexOptions.IgnoreCase);
var allFiles =
    Directory.EnumerateFiles(imagesPath)
        .Where(file => regex.IsMatch(file));
```

我在这里创建了一个正则表达式，表示我想过滤以 `.jpg` 或 `jpeg` 结尾的字符串，并忽略大小写。然后我使用 `Directory.EnumerateFiles()` 并应用 `Where()` LINQ 操作符来应用 `regex` 过滤器。

此方法在所有平台上都工作得很好。您本可以通过以下代码避免使用 `regex` 过滤器，该代码更冗长，但我假设对许多人来说更易读：

```cs
var files = Directory.EnumerateFiles(imagesPath)
    .Where(file => file.EndsWith(".jpg",
StringComparison.OrdinalIgnoreCase) ||
                   file.EndsWith(".jpeg",
StringComparison.OrdinalIgnoreCase));
```

我在下面的表中为您收集了`Directory`类最常用的方法和属性：

| **方法** **或属性** | **描述** |
| --- | --- |
| `Directory.CreateDirectory` | 在指定的路径中创建所有目录和子目录，除非它们已经存在 |
| `Directory.Delete` | 删除指定的目录，以及可选地删除目录中的任何子目录和文件 |
| `Directory.Exists` | 确定给定的路径是否指向磁盘上的现有目录 |
| `Directory.GetCurrentDirectory` | 获取应用程序的当前工作目录 |
| `Directory.GetDirectories` | 获取指定目录中子目录的名称（包括它们的路径） |
| `Directory.GetFiles` | 返回指定目录中文件的名称（包括它们的路径） |
| `Directory.GetFileSystemEntries` | 返回指定目录中所有文件和子目录的名称 |
| `Directory.GetLastAccessTime` | 返回指定文件或目录最后访问的日期和时间 |
| `Directory.GetLastWriteTime` | 返回指定文件或目录最后写入的日期和时间 |
| `Directory.GetParent` | 获取指定路径的父目录，包括绝对路径和相对路径 |
| `Directory.Move` | 将文件或目录及其内容移动到新位置 |
| `Directory.SetCreationTime` | 设置指定文件或目录的创建日期和时间 |
| `Directory.SetCurrentDirectory` | 将应用程序的当前工作目录设置为指定的目录 |
| `Directory.SetLastAccessTime` | 设置指定文件或目录最后访问的日期和时间 |
| `Directory.SetLastWriteTime` | 设置指定文件或目录最后写入的日期和时间 |

表 5.3：目录类的属性和方法

`Directory`有一些不错的辅助方法和属性。你可以自己找出所有这些属性，但为什么麻烦自己去做，如果 CLR 足够友好地帮助你呢？当我们以后转移到其他平台时，这些属性也将是有益的。

## `DirectoryInfo`类

我还想讨论另一个类：`DirectoryInfo`类。`Directory`和`DirectoryInfo`之间的区别在于前者使用静态方法，而后者用作实例。`Directory`返回关于目录的字符串信息。`DirectoryInfo`返回包含更多信息的对象。让我给你举一个例子：

```cs
var imagesPath = Environment.GetFolderPath(
    Environment.SpecialFolder.MyPictures);
var directoryInfo = new DirectoryInfo(imagesPath);
Console.WriteLine(directoryInfo.FullName);
Console.WriteLine(directoryInfo.CreationTime);
Console.WriteLine(directoryInfo.Attributes);
```

我创建了一个`DirectoryInfo`类的实例，并给它提供了我们`images`文件夹的路径。这个实例有很多有价值的属性，例如完整名称、创建时间、属性等等。我在下表中列出了最常用的属性和方法。

| **方法** **或属性** | **描述** |
| --- | --- |
| `DirectoryInfo.Create` | 创建一个目录 |
| `DirectoryInfo.Delete` | 删除此 `DirectoryInfo` 实例，指定是否删除子目录和文件 |
| `DirectoryInfo.Exists` | 获取一个值，指示目录是否存在 |
| `DirectoryInfo.Extension` | 获取表示目录扩展部分的字符串 |
| `DirectoryInfo.FullName` | 获取目录或文件的完整路径 |
| `DirectoryInfo.Name` | 获取此 `DirectoryInfo` 实例的名称 |
| `DirectoryInfo.Parent` | 获取指定子目录的父目录 |
| `DirectoryInfo.Root` | 获取路径的根部分 |
| `DirectoryInfo.GetFiles` | 从当前目录返回文件列表 |
| `DirectoryInfo.GetDirectories` | 返回当前目录的子目录 |
| `DirectoryInfo.GetFileSystemInfos` | 获取表示当前目录中的文件和子目录的 `FileSystemInfo` 对象数组 |
| `DirectoryInfo.MoveTo` | 将 `DirectoryInfo` 实例及其内容移动到新路径 |
| `DirectoryInfo.Refresh` | 刷新对象的状态 |
| `DirectoryInfo.EnumerateFiles` | 返回当前目录中文件信息的可枚举集合 |
| `DirectoryInfo.EnumerateDirectories` | 返回当前目录中目录信息的可枚举集合 |
| `DirectoryInfo.Enumerate FileSystemInfos` | 返回当前目录中文件系统信息的可枚举集合 |

表 5.4：DirectoryInfo 属性和方法

如您所见，`Path`、`Directory` 和 `DirectoryInfo` 在处理文件时可以提供很大帮助。

# 文件系统监控

作为系统程序员，我们必须找到与我们的应用程序通信的方法。毕竟，没有用户界面让用户可以表明他们的期望操作。

那个类别的多数应用程序都监听网络端口或以其他方式让系统与之通信。其中一种方式是等待文件或目录的变化。

监视文件或文件夹是一个相当常见的场景。例如，我们可以构建一个系统来处理通过电子邮件系统获取的文件。一旦文件作为附件发送，邮件客户端就会将其放置在一个目录中，我们的系统就会去获取它。

这意味着我们需要有一种方法来监视那个文件夹。幸运的是，这并不太难实现。这确实需要一些解释，所以让我带你了解一下。

我们将从其他类与之交互的类开始：

```cs
internal class MyFolderWatcher : Idisposable
{
    protected virtual void Dispose(bool disposing)
    {
        if (disposing)
        {
            // Dispose managed state (managed objects).
        }
    }
     ~MyFolderWatcher()
    {
        Dispose(false);
    }
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
}
```

我们稍后需要清理一些资源，所以我在这里实现了 `IDisposable` 接口。我们需要清理的类是 `FileSystemWatcher` 类型的实例。这个类在实例化时监视一个文件夹，并且可选地监视文件名过滤器。如果那里发生了一些有趣的事情，`FileSystemWatcher` 会通知我们。定义“有趣的事情”是什么取决于我们。

让我们将它设置为我们类的一个私有成员：

```cs
private FileSystemWatcher? _watcher;
```

我们可以将我们的 `Dispose`(bool disposing) 方法改为清理这些内容，但我会暂时保留这个。我们需要做的不仅仅是销毁 `FileSystemWatcher`。

`FileSystemWatcher` 是资源密集型的。监视一个文件夹可能会导致大量的 CPU 压力。因此，我们必须确保只有在需要时才启用它。

然后，我们将添加一个启用监视器并设置一些设置的方法：

```cs
public void SetupWatcher(string pathToWatch)
{
    if(_watcher != null)
        throw new InvalidOperationException(
            "The watcher has already been set up");
    if(!Path.Exists(pathToWatch))
        throw new ArgumentOutOfRangeException(
            nameof(pathToWatch),
            "The path does not exist");
    // Set the folder to keep an eye on
    _watcher = new FileSystemWatcher(pathToWatch);
    // We only want notifications when a file is created or
    // when it has changed.
    _watcher.NotifyFilter =
        NotifyFilters.FileName |
        NotifyFilters.LastWrite;
    // Set the callbacks
    _watcher.Created += WatcherCallback;
    _watcher.Changed += WatcherCallback;
    // Start watching
    _watcher.EnableRaisingEvents = true;
}
```

我们将开始进行两项检查。首先，我们将查看监视器是否已经被创建。如果已经创建，我们将抛出一个错误。第二是检查提供的路径是否存在。

如果这两个检查都通过，我们将创建一个 `FileSystemWatcher` 类的实例，并给它我们想要监视的路径。

您可以指定您想要监视的内容。这由 `NotifyFilter` 属性控制。这个属性接受一个枚举或 `NotifyFilter` 枚举的组合。您可以在以下表中查看您的选项。

| **NotifyFilters 枚举** | **描述** |
| --- | --- |
| `Attributes` | 监视文件或文件夹属性的变化 |
| `CreationTime` | 监视文件和目录的创建时间的变化 |
| `DirectoryName` | 监视目录名称的变化 |
| `FileName` | 监视文件名称的变化 |
| `LastAccess` | 监视文件和目录的最后访问时间的变化 |
| `LastWrite` | 监视文件和目录的最后写入时间的变化 |
| `Security` | 监视文件和目录的安全设置的变化 |
| `Size` | 监视文件和目录的大小变化 |

表 5.5：NotifyFilters 选项

我只对文件夹中的新文件或更改的文件感兴趣。因此，我给它设置了 `NotifyFilters.FileName` | `NotifyFilters.LastWrite` 的值。当然，当您第一次创建文件时，文件的 `FileName` 会发生变化。我也可以选择 `CreationTime`，它几乎不会变化。我还会监视 `LastWrite`，它告诉我文件何时发生变化。

在此之后，我将给 `_watcher` 一个回调，当两个我关心的任意一个事件被触发时调用。由于所有事件共享相同的签名，我可以用一个方法来完成。这个方法就是我们接下来要看的。然而，在我们这样做之前，我们需要通过将 `_watcher.EnableRaisingEvents` 设置为 `True` 来启动监视器。下一段代码包含了 `eventhandler` 的主体：

```cs
private void WatcherCallback(object sender, FileSystemEventArgs e)
{
    switch (e.ChangeType)
    {
        case WatcherChangeTypes.Created:
            FileAdded?.Invoke(this, new FileCreatedEventArgs                 (e.FullPath));
            break;
        case WatcherChangeTypes.Changed:
            FileChanged?.Invoke(this, new
                FileChangedEventArgs(e.FullPath));
            break;
    }
}
```

当监视器调用这个回调时，我们得到一个 `FileSystemEventArgs` 类的实例。这个类包含一个名为 `ChangeType` 的字段，它指示触发了这个调用的变化类型。它还包含受影响的文件的完整路径和名称，在 `FullPath` 属性中。

我们将切换到 `ChangeType` 字段，并调用我们类中的一个事件处理器。我们类中的两个事件处理器如下所示：

```cs
public event EventHandler<FileCreatedEventArgs>? FileAdded;
public event EventHandler<FileChangedEventArgs>? FileChanged;
```

`FileCreatedEventArgs`和`FileChangedEventArgs`类型对于`EventHandler`来说也很直接。我本可以使用一个类型。然而，为了未来的使用，我决定为它们提供不同的类，我可能在某个时候通过更多信息扩展它们。它们看起来像这样：

```cs
public class FileCreatedEventArgs : EventArgs
{
    public FileCreatedEventArgs(string filePath)
    {
        FilePath = filePath;
    }
    public string FilePath { get; }
}
public class FileChangedEventArgs : EventArgs
{
    public FileChangedEventArgs(string filePath)
    {
        FilePath = filePath;
    }
    public string FilePath { get; }
}
```

`FileSystemWatcher`实现了`IDisposable`。因此，当我们不再使用它时，我们必须将其释放。我们需要重写自己的`Dispose(bool disposing)`方法，使其看起来像这样：

```cs
protected virtual void Dispose(bool disposing)
{
    if (!disposing) return;
    if (_watcher == null)
        return;
    // Stop raising events
    _watcher.EnableRaisingEvents = false;
    // Clean whoever has subscribed to us
    // to prevent memory leaks
    FileAdded = null;
    FileChanged = null;
    _watcher.Dispose();
    _watcher = null;
}
```

在进行了一些检查后，我们将停止系统接收任何事件。然后我们将清除事件。如果我们不这样做，其他对象可能会持有对我们类的引用，从而阻止此类从内存中释放。

当那件事完成时，我们将释放`_watcher`并将其设置为 null。

就这样。如果你从你的程序中运行它，给它一个文件夹，并附加一些`eventhandlers`，你将能够看到当你向该文件夹添加或更改文件时会发生什么。

几乎完美。几乎——但并不完全。

如果你添加一个文件，你会得到多个事件。如果你这么想，这是有道理的。毕竟，文件是在文件系统中创建的，然后立即被更改。如果你愿意，你可以更改我们的类来考虑这一点。这并不难做，所以我会把它留给你。

# 异步 I/O

我之前已经说过，但这一点非常重要，所以我必须在这里重复一遍：I/O 很慢。任何与 I/O 一起工作的代码都应该异步执行。幸运的是，`System.IO`命名空间中的大多数类都有我们可以使用的异步成员，我们可以与 async/await 一起使用。

如果微软决定将`System.IO`中所有非异步方法标记为过时，我会很高兴。

## 天真的方法

你在`System.IO`中知道的大部分方法都有一个异步版本。所以，只需在方法名后添加`async`后缀并等待它即可。很简单！

仔细想想，不。这并不简单。

让我给你举一个例子：

```cs
public async Task CreateBigFileNaively(string fileName)
{
    var stream = File.CreateText(fileName);
    for (int i = 0; i < Int32.MaxValue; i++)
    {
            var value = $"This is line {i}";
            Console.Writeline(value);
            await stream.WriteLineAsync(value);
                await Task.Delay(10);
    }
    Console.WriteLine("Closing the stream");
    stream.Close();
    await stream.DisposeAsync();
}
```

此方法创建一个文件，然后向其中写入一行字符串。完成后，它关闭文件并很好地释放它。它是异步执行的。所以这就是事情应该这样做的方式，对吧？

让我们使用这个方法：

```cs
var asyncSample = new AsyncSample();
await asyncSample.CreateBigFileNaively(@"c:\temp\bigFile.txt");
```

将这两行代码添加到你的主控制台应用程序中。运行它并让它运行几秒钟。注意屏幕上写入的行（它应该说的是类似于“这是第 n 行”，其中`n`是行的编号）。然后按*Ctrl* + *C*取消操作。程序将停止。现在，请打开文件并看看它走了多远。有很大可能性你会看到文件上最后写入的行不是你在屏幕上看到的数字。

你可能会想知道为什么？CLR 确保我们的代码的性能尽可能高。所以，所有写入文件系统的数据都会在发送到 SSD 或其他媒体之前缓冲到缓存中。毕竟，写入存储是慢的。然而，由于我们终止了进程，CLR 没有时间刷新缓存。

## 使用取消令牌

当然，在现实世界中这种情况不会经常发生。然而，你可能想要取消一个长时间运行的 IO 进程，然后你可能会遇到这种情况。

这有一个解决方案。还记得我们讨论线程的那一章吗？还记得我说过有一个叫做`CancellationToken`的东西吗？那就是我们需要的东西。

让我们重写写入文件的代码。让我们从方法名称中移除`naïve`；我们现在知道得更多了：

```cs
public async Task CreateBigFile(string fileName, CancellationToken cancellationToken)
{
    var stream = File.CreateText(fileName);
    for (int i = 0; i < Int32.MaxValue; i++)
    {
        if (cancellationToken.IsCancellationRequested)
        {
            Console.WriteLine("We are being cancelled");
            break;
        }
        else
        {
            var value = $"This is line {i}";
            Console.WriteLine(value);
            await stream.WriteLineAsync(value);
            try
            {
                await Task.Delay(10, cancellationToken);
            }
            catch (TaskCanceledException)
            {
                Console.WriteLine("We are being cancelled");
                break;
            }
        }
    }
    Console.WriteLine("Closing the stream");
    stream.Close();
    await stream.DisposeAsync();
}
```

我们在这里添加了很多代码。让我带你看看。首先，我们向方法添加了一个`CancellationToken`类型的参数。我们将在循环中不断检查是否请求了`Cancel`。如果是这样，我们将在屏幕上打印消息并优雅地退出循环。

在`Task.Delay()`中，我们也传入了`CancellationToken`。毕竟，当系统等待这个延迟时，也可以请求取消。然而，当这发生在`Task.Delay()`期间时，CLR 将抛出一个`TaskCanceledException`类型的异常。我们必须捕获它以防止我们的程序崩溃并停止。这就是为什么我们在这里有`try..catch`块。我们需要这个`try..catch`块来防止异常向上冒泡到调用堆栈。

我们必须模拟从外部中断循环。将调用此方法的代码更改为以下内容：

```cs
var cancellationTokenSource = new
CancellationTokenSource();
ThreadPool.QueueUserWorkItem((_) =>
{
    Thread.Sleep(10000);
    Console.WriteLine("About to cancel the operation");
    cancellationTokenSource.Cancel();
});
var asyncSample = new AsyncSample();
await asyncSample.CreateBigFile(
    @"c:\temp\bigFile.txt",
    cancellationTokenSource.Token);
```

首先，我们将创建一个新的`CancellationtokenSource`。然后，我们将从`ThreadPool`中拉取一个线程并给它一些事情做。等待 10 秒后，它将请求取消。

现在调用`CreateBigFile`时有一个`CancellationTokenSource`。

运行它并看到它在 10 秒后停止。注意它停止在哪一行，并检查实际的文件以查看是否是最后写入的一行。在我的机器上，这工作得很好。

记住：在处理异步文件处理时，无论你做什么，尽量使用`CancellationSourceToken`。同时，确保你处理任何副作用。确保在请求取消后进行清理，以便 CLR 可以正确刷新缓存并清理其资源。

## BufferedStream

CLR 在最大化 I/O 操作性能方面做得相当不错。正如我们所见，它可以在写入外部设备之前缓存数据。这种缓存加快了我们的代码，因为我们不再需要等待缓慢的写入操作完成。然而，CLR 对那些缓存做出了明智的猜测。有时它会出错。如果我们知道我们想要写入的数据的大小，我们可以利用这个知识从我们的应用程序中获得更高的性能。

假设我们有一个系统，它会将以下记录写入 I/O：

```cs
internal readonly record struct DataRecord
{
    public int Id { get; init; }
    public DateTime LogDate { get; init; }
    public double Price { get; init; }
}
```

这个块长 24 字节。我们可以通过将`int`、`DateTime`和`double`的大小相加来快速确定这一点。

如果我们将这些内容写入文件，CLR 会将其缓存，直到系统找到一个合适的时机将实际的数据写入存储。然而，我们可以改进这一点。我们可以使用`BufferedStream`类首先将数据写入缓冲区。然后，当 CLR 认为这是最佳时机时，可以将该缓冲区刷新到底层存储。这里的优势在于我们可以控制该缓冲区或缓存的大小。如果我们指定的大小恰到好处，我们就不会浪费内存。然而，我们也不会将其设置得太小，以免刷新过于频繁。这对我们来说刚刚好。

实现这一功能的代码看起来像这样：

```cs
public async Task WriteBufferedData(string fileName)
{
    var data = new DataRecord
    {
        Id = 42,
        LogDate = DateTime.UtcNow,
        Price = 12.34
    };
    await using FileStream stream = new(fileName, FileMode.CreateNew,     FileAccess.Write);
    await using BufferedStream bufferedStream = new(stream,
    Marshal.SizeOf<DataRecord>());
    await using BinaryWriter writer = new(bufferedStream);
    writer.Write(data.Id);
    writer.Write(data.LogDate.ToBinary());
    writer.Write(data.Price);
}
```

首先，我们将创建一个`FileStream`。这个`FileStream`是我们写入的实际文件的句柄。然后，我们将创建一个`BufferedStream`，并给它提供`FileStream`以及我们想要写入的记录的大小。之后，我们将创建一个`BinaryWriter`，以便尽可能高效地将我们的数据写入缓冲区。

一切都设置好之后，我们再进行写作。

一个警告

如果你不确定数据的大小，使用`BufferedStream`可能会对你不利。`BufferedStream`在执行大量已知大小的较小、频繁的数据写入时表现最佳。否则，缓存管理最好留给 CLR。

# 文件系统安全

文件是我们存储东西的地方。那些东西可能不是每个人都想看到的。有时，我们必须隐藏数据或确保只有我们信任的程序可以访问它。操作系统可以提供帮助。每个操作系统都有处理文件和目录访问的方式。你通常可以允许或拒绝对这些文件的读取或写入访问。

然而，当你想要分享文件时会发生什么呢？让我们假设你想要通过电线传输数据或者将其存储在另一个驱动器上，比如可移动的 USB 驱动器。在这种情况下，确保那样的安全级别是非常具有挑战性的。这意味着你可能需要加密数据以防止其被滥用。

安全性——一个独立的话题

我在这里只介绍安全性和加密的基础知识。这并不是这个复杂且广泛主题的完整指南。仅关于这个主题就有数百本书籍被撰写。我想让你知道你可以进行安全和加密。然而，如果你想要认真对待这个问题，我建议你出去寻找一些关于这些主题的优质资源，并从那里学习。

## 加密基础

基本上，我们有两种加密类型：**对称**和**非对称**算法。尽管它们之间有很多相似之处，但一个很大的区别在于它们处理密钥的方式。

让我们讨论一个基本的例子。假设你有一条信息，想要将它传输给其他人。由于信息内容敏感，你不想让其他人能够阅读它，所以你决定对其进行加密。这意味着你需要改变信息的内文，使得没有人能够理解它。接收者随后解密它，将你的文本转换成可理解的内容。我们称人们可以实际阅读并理解的内容为**明文**。相反，加密的、不可读的文本是我们所说的**密文**。人们阅读明文；密文需要解密。

这种保护信息的方式并不新颖。2000 多年前，凯撒就做过这样的事情。他使用了一种简单的替换算法。他所做的就是取一段他想发送给战场指挥官的文字，然后将所有字符向左或向右移动一定的位置。这里的数字就是我们所说的他的密钥。

因此，如果凯撒选择了`3`这个密钥，他信息中的所有 A 都会变成 D。字符 B 会变成 E，以此类推。

如果你知道了密钥，你就可以用他的密文进行逆向操作，恢复成明文。

这里的问题是传输实际要使用的数字。双方都需要知道密钥，否则事情永远不会成功。你需要一种安全的方式告诉对方使用哪个密钥，这样他们才能将你的密文解密成明文。

如果你认识对方，共享这个密钥并不难。你可以走上前去，将密钥写在一张密封的信封里递给他们，并告诉他们只有在收到加密信息时才能打开。然而，如今这样做要困难得多。计算机不知道它们想要与之通信的其他计算机。安全地交换密钥很困难。

解决这个问题的可能方法是使用非对称加密和解密。这个解决方案很复杂，但其基础是这样的：你有两个密钥。一个密钥用于加密数据，另一个用于解密数据。其中一个密钥是私有的，另一个是公开的。私钥只属于你一个人。你用它来加密文件。任何拥有公钥的人都可以解密它。当然，如果你想信息只被另一个特定方阅读，你可以反过来操作。你可以要求对方与你共享他们的公钥。然后，你用那个密钥加密信息。现在，只有对方才能用他们的私钥再次解密。

对称算法比非对称算法快得多。然而，它们面临密钥共享的问题。这就是为什么大多数算法结合两种方法的原因。它们使用非对称算法加密一个密钥，这个密钥可以用于对称加密。这个密钥相对较小，因此加密和解密可以相对快速地进行。然后，使用这个对称密钥来加密整个消息。这样，对称密钥就可以成为消息的一部分。它本身被加密，所以只有预期的接收者才能解密密钥，从而解密消息的其余部分。

如果这听起来很复杂，我有好消息：CLR 有许多类可以帮助我们完成这项工作。它们的使用也很简单。

## 对称加密和解密

让我们看看我们是否可以在 C#代码中加密和解密一个简单的消息：

```cs
public static void EncryptFileSymmetric(string inputFile, string outputFile, string key)
{
    using (FileStream inputFileStream = new
    FileStream(inputFile, FileMode.Open, FileAccess.Read))
    using (FileStream outputFileStream = new FileStream(outputFile,     FileMode.Create, FileAccess.Write))
    {
        byte[] keyBytes = Encoding.UTF8.GetBytes(key);
        using (Aes aesAlg = Aes.Create())
        {
            aesAlg.Key = keyBytes;
            aesAlg.GenerateIV();
            byte[] ivBytes = aesAlg.IV;
            outputFileStream.Write(ivBytes, 0, ivBytes.Length);
            using (CryptoStream csEncrypt = new
               CryptoStream(outputFileStream,                aesAlg.CreateEncryptor(),
                       CryptoStreamMode.Write))
            {
                byte[] buffer = new byte[4096];
                int bytesRead;
                while ((bytesRead =
                   inputFileStream.Read(buffer,                    0, buffer.Length)) > 0)
                {
                    csEncrypt.Write(buffer, 0, bytesRead);
                }
            }
        }
    }
}
```

这种方法接受输入文件名、输出文件名和一个密钥。然后，它打开输入文件，读取其内容，加密它，并将密文写入输出文件。

这种工作方式非常直接。

首先，我们将创建两个流。然后，我们将获取密钥并生成其字节数组。密钥必须是 128 位、192 位或 256 位数组。换句话说，它必须是 16、24 或 32 字节长。密钥越长，破解就越困难。然而，长密钥也会减慢加密和解密过程。选择权在你。

我们将创建一个`Aes`类的实例。**高级加密标准**（**AES**）被广泛认为是一个好且安全的加密算法。为了使事情更加安全，我们将使用**初始化向量**（**IV**）增强我们的密钥。你可以将其视为我们添加到密钥中以使其更难以阅读的东西。我们将把这个 IV 作为文件中的第一件事写入。

然后，我们将创建一个`CryptoStream`类的实例。这个类帮助我们写入加密数据，正如你将在接下来的代码块中看到的那样。我们将字节数组传递给`CryptoStream`类。由于我们使用 AES 类（更确切地说，是该类`CreateEncryptor`调用的结果）初始化了`CryptoStream`类，因此它使用我们的密钥来加密数据。

解密也很简单。它遵循相同的原理：从密钥获取文件，从文件中读取 IV，然后解密其余部分并将其存储在新文件中。这看起来是这样的：

```cs
public static void DecryptFileSymmetric(string inputFile, string outputFile, string key)
{
    using (FileStream inputFileStream = new FileStream(inputFile,     FileMode.Open, FileAccess.Read))
    using (FileStream outputFileStream = new FileStream(outputFile,     FileMode.Create, FileAccess.Write))
    {
        byte[] keyBytes = Encoding.UTF8.GetBytes(key);
        using (Aes aesAlg = Aes.Create())
        {
            byte[] ivBytes = new byte[aesAlg.BlockSize / 8];
            inputFileStream.Read(ivBytes, 0,
               ivBytes.Length);
            aesAlg.Key = keyBytes;
            aesAlg.IV = ivBytes;
            using (CryptoStream csDecrypt =
                   new CryptoStream(outputFileStream,
                   aesAlg.CreateDecryptor(), CryptoStreamMode.Write))
            {
                byte[] buffer = new byte[4096];
                int bytesRead;
                while ((bytesRead =
                inputFileStream.Read(buffer, 0, buffer.Length)) > 0)
                {
                    csDecrypt.Write(buffer, 0, bytesRead);
                }
            }
        }
    }
}
```

我们现在不是从`CryptoStream`获取`Encryptor`，而是获取`Decryptor`。其余的现在应该很容易理解了。

## 非对称加密和解密

在前面的例子中，我们生成了一个简单的 128 位、192 位或 256 位密钥。例如，你可以传递一个字符串，如`SystemSoftware42`，并获取字节。相同的密钥用于加密和解密。

对于非对称加密，获取密钥要困难一些。然而，有辅助类可以帮助完成这项工作，所以在实践中并不困难。以下是代码：

```cs
public static (string, string) GenerateKeyPair()
{
    using RSA rsa = RSA.Create();
    byte[] publicKeyBytes = rsa.ExportRSAPublicKey();
    byte[] privateKeyBytes = rsa.ExportRSAPrivateKey();
    string publicKeyBase64 = Convert.ToBase64String(publicKeyBytes);
    string privateKeyBase64 = Convert.ToBase64String(privateKeyBytes);
    return (publicKeyBase64, privateKeyBase64);
}
```

我使用了`RSA`类来生成密钥对。**Rivest, Shamir, and Adleman**（**RSA**）类是以发明这个算法的三位密码学家命名的。

我们将通过调用`Create()`来创建一个`RSA`实例。然后，我们将调用`ExportRSAPublicKey()`和`ExportRSAPrivateKey()`来从其中获取生成的密钥。

由于密钥是字节数组，我们将使用`ToBase64String()`来使它们更易于阅读。这使得共享密钥变得更加容易。

现在我们有了密钥对，我们可以用它来加密一条消息。当然，我们也可以再次解密它。这段代码看起来是这样的：

```cs
public static byte[] EncryptWithPublicKey(
    byte[] data,
    byte[] publicKeyBytes)
{
    using RSA rsa = RSA.Create();
    rsa.ImportRSAPublicKey(publicKeyBytes, out _);
    return rsa.Encrypt(data, RSAEncryptionPadding.OaepSHA256);
}
public static byte[] DecryptWithPrivateKey(
    byte[] encryptedData,
    byte[] privateKeyBytes)
{
    using RSA rsa = RSA.Create();
    rsa.ImportRSAPrivateKey(privateKeyBytes, out _);
    return rsa.Decrypt(encryptedData, RSAEncryptionPadding.    OaepSHA256);
}
```

这段代码足够简单。我只想指出`rsa.Encrypt()`和`rsa.Decrypt()`方法中的最后一个参数。在这里我们将使用填充来向结果中添加额外的数据（并且在解密时我们会将其移除）。这种填充使得攻击者尝试破解我们的消息变得更加困难。

你可以将这三个方法结合起来使用，如下所示：

```cs
(string, string) keyPair = Encryption.GenerateKeyPair();
keyPair.Item1.Dump();
keyPair.Item2.Dump();
var publicKey = Convert.FromBase64String(keyPair.Item1);
var privateKey = Convert.FromBase64String(keyPair.Item2);
string message = "This is the text that we, as System Programmers,     want to secure.";
byte[] messageBytes = Encoding.UTF8.GetBytes(message);
byte[] encryptedBytes = Encryption.EncryptWithPublicKey(messageBytes,     publicKey);
string encrypted = Encoding.UTF8.GetString(encryptedBytes);
encrypted.Dump(ConsoleColor.DarkYellow);
byte[] decryptedBytes = Encryption.    DecryptWithPrivateKey(encryptedBytes, privateKey);
string decrypted = Encoding.UTF8.GetString(decryptedBytes);
decrypted.Dump(ConsoleColor.DarkYellow);
```

首先，我们将创建一个密钥对。我们的方法将这个对作为字符串返回，这样我们就可以打印它们（我再次使用我们方便的`Dump()`扩展方法）。然而，密钥需要以二进制格式存在，所以我将它们转换成字节数组。

我将定义我想要加密的消息，获取该消息的字节，并对其进行加密。然后，我将打印加密后的消息。如果你这样做，我想你会同意这很难看到实际的消息。它是一堆字符的混乱。

然后，我们将通过调用`DecryptWithPrivateKey()`来反转它。这个方法返回我们的字符串。

如果我们将我们的公钥的`Base64`版本发送给某人，然后传输编码后的消息，他们可以用这个公钥来解码它。他们会确信我们发送了这条消息；除了我们之外，没有人能够生成可以被该公钥解密的消息。毕竟，私钥和公钥是一对。你需要一个来加密，以便第二个可以解密。

朱利叶斯·凯撒会为我们感到骄傲！

然而，我们还有另一件事要讨论。我们需要减轻负担。好吧，不是我们个人，而是我们文件中的有效载荷可能会从中受益。让我们谈谈文件压缩。

# 文件压缩

文件可能会变得相当大。正如我们已经讨论过的，文件 I/O 和网络 I/O 需要很长时间，尤其是与 CPU 的速度相比。我们能够做的任何减少从 I/O 读取或写入所需时间的操作都可能是有价值的。即使这意味着我们必须让 CPU 做更多的工作，这也同样是正确的。当然，你需要测量这一点并看看它是否也适用于你的情况，但有时，为了加快 I/O 速度而牺牲 CPU 时间可能会产生巨大的差异。

实现这一点的其中一种方法是通过限制写入文件或网络流中的数据量。这可以通过压缩来完成。

在 CLR 中，你有选择。你可以使用 `DeflateStream` 或 `GZipStream` 来做这件事。`GZipStream` 在内部使用 `DeflateStream`，所以 `DeflateStream` 显然更快。然而，`GZipStream` 生成的压缩文件可以被外部软件读取。GZip 是一个标准化的压缩算法。

## 压缩一些数据

让我们使用 `GZipStream` 压缩一个字符串：

```cs
public async Task<byte[]> CompressString(string input,
    CancellationToken cancellationToken)
{
    // Get the payload as bytes
    byte[] data =
    System.Text.Encoding.UTF8.GetBytes(input);
    // Compress to a MemoryStream
    await using var ms = new MemoryStream();
    await using var compressionStream = new GZipStream(ms,
    CompressionMode.Compress);
    await compressionStream.WriteAsync(data, 0,
    data.Length, cancellationToken);
    await compressionStream.FlushAsync(cancellationToken);
    // Get the compressed data.
    byte[] compressedData = ms.ToArray();
    return compressedData;
}
```

由于压缩和解压缩可能需要很长时间才能完成，我们在这里真的应该使用 `Async/Await` 模式。

我们将取一些想要压缩的字符串并将它们传递给输入变量。在这个例子中，我使用了 `MemoryStream`，但你也可以使用你喜欢的任何流。大多数现实世界的例子使用某种形式的 `FileStream`。

我将创建一个 `GZipStream` 类的实例，并给它一个 `MemoryStream` 实例。这个内存流是它写入数据的地方。我还会告诉这个类我想要压缩数据。

然后，我只需将数据写入其中，刷新缓冲区，并从中获取字节。

就这样！我已经压缩了一个字符串。

## 解压缩一些数据

解压缩同样简单。看看下面的代码示例：

```cs
public async Task<string> DecompressString(byte[] input,
    CancellationToken cancellationToken)
{
    // Write the data into a memory stream
    await using var ms = new MemoryStream();
    await ms.WriteAsync(input, cancellationToken);
    await ms.FlushAsync(cancellationToken);
    ms.Position = 0;
    // Decompress
    await using var decompressionStream = new GZipStream(ms,     CompressionMode.Decompress);
    await using var resultStream = new MemoryStream();
    await decompressionStream.CopyToAsync(resultStream,     cancellationToken);
    // Convert to readable text.
    byte[] decompressedData = resultStream.ToArray();
    string decompressedString =
    System.Text.Encoding.UTF8.GetString(decompressedData);
    return decompressedString;
}
```

在这里，我使用了两个 `MemoryStream` 类的实例。我使用一个作为数据的源，另一个作为未压缩数据的目的地。再次提醒，请使用你想要的任何流。

你可以使用以下方法：

```cs
var cts = new CancellationTokenSource();
var myText = "This is some text that I want to compress.";
var compression = new Compression();
var compressed = await compression.CompressString(myText, cts.Token);
var decompressed = await
    compression.DecompressString(compressed, cts.Token);
decompressed.Dump(ConsoleColor.DarkYellow);
```

那并不太难，对吧？

然而，我们还没有完成。我们想要存储或读取的数据需要以某种格式存在。如果你有一个包含数据的 C# 类，你不能简单地将其写入文件。我们需要以某种方式将其转换。这就是序列化的用武之地。

# 序列化 – JSON 和二进制

在本章的早期，我们看到了如何将二进制数据写入流。我们可以调用所有 `write` 方法将各种类型写入文件。然而，这可能会相当困难，并且也容易出错。你必须跟踪数据的格式。一个简单的错误会使你的文件无法读取。

更好的方法是将你的数据序列化成流可以理解的格式。有两种方法可以做到这一点：**JSON** 和 **二进制**。

JSON 很简单：大多数编程语言和平台都理解它。JSON 已经成为在文本中显示结构的既定标准。在大多数地方，JSON 已经取代了 XML。JSON 更小，更轻量。

然而，它还可以更轻量。你还可以将你的数据序列化为二进制流。这需要更多的编码，但通常会产生更小文件和数据流。再次提醒，这可能是我们作为系统程序员所寻找的。

## JSON 序列化

要将对象序列化为 JSON 格式，人们过去默认使用 `NewtonSoft.JSON`。`NewtonSoft.JSON` 是首选库。它易于使用（并且仍然如此）并提供了许多人们喜欢的功能，例如自定义转换器。然而，微软随后发布了 `System.Text.Json`，它执行相同的操作但效率更高。作为系统程序员，我们关心内存效率和速度，因此我将重点关注这一点。

在我们能够序列化某个对象之前，我们需要一个可以序列化的对象。`System.Text.Json` 的优势在于我无需更改带有属性的类。框架足够智能，能够找出所需的内容并完成它。

我将使用本章前面看到的相同数据类在这些示例中。然而，为了节省您翻页的时间，我再次在这里向您展示它：

```cs
class MyData
{
    public int Id { get; set; }
    public double SomeMagicNumber { get; set; }
    public bool IsThisAGoodDataSet { get; set; }
    public MyFlags SomeFlags { get; set; }
    public string? SomeText { get; set; }
}
[Flags]
public enum MyFlags
{
    FlagOne,
    FlagTwo,
    FlagThree
}
```

如果我们想将此序列化为 JSON 以存储为文本并在以后重新读取，我们将使用以下代码：

```cs
public string SerializeToJSon(MyData myData)
{
    var options = new JsonSerializerOptions
    {
        WriteIndented = true,
        PropertyNamingPolicy = JsonNamingPolicy.CamelCase
    };
    var result =
    System.Text.Json.JsonSerializer.Serialize(myData,options );
    return result;
}
```

这里展示的代码相当直接。我们将使用 `MyData` 类并将其传递给静态 `Serialize` 方法，`System.Text.Json.JsonSerializer`。此方法有几个重载。我将使用一个接受 `JsonSerializerOptions` 类实例的重载。这样，我可以格式化输出。我将 `WriteIdented` 属性设置为 `True`。如果没有这样做，我将会得到整个字符串在一行上。诚然，这会节省我几个换行符和制表符字符，但为了可读性，我更喜欢这种方式。

如果我们在类中运行一些值，我们将得到以下结果：

```cs
{
  "id": 42,
  "someMagicNumber": 3.1415,
  "isThisAGoodDataSet": true,
  "someFlags": 2,
  "someText": "This is some text that we want to serialize"
}
```

反序列化，即逆转该过程，同样简单：

```cs
public MyData DeserializeFromJSon(string json)
{
    var options = new JsonSerializerOptions
    {
        WriteIndented = true,
        PropertyNamingPolicy = JsonNamingPolicy.CamelCase
    };
    var result = System.Text.Json.JsonSerializer.        Deserialize<MyData>(json, options);
    return result!;
}
```

如您所见，这个过程足够简单。

## 二进制序列化

这并不是将对象编码以便存储在文件中的最有效方式。它相对较快，但实际数据也相当大。二进制格式化需要更多的工作，结果也不是人类可读的，但它确实导致了文件更小。这意味着将数据写入和读取到慢速存储介质所需的时间显著减少。当然，权衡是 CPU 会变得稍微忙碌一些，但这可能值得。一如既往，先衡量，然后决定这是否适用于您的情况。

在 .NET Framework 中，在 .NET Core 和 .NET 之前，我们有一个名为 `BinaryFormatter` 的类。然而，该类现在已被标记为过时。与该类相关的存在严重的安全担忧，因此微软决定将其淘汰。

您可以使用第三方包来实现相同的目标。然而，如果您不想使用那些，您始终可以自己完成。我们已经讨论了 `BinaryWriter` 类及其方法。使用该类并没有什么问题，但缺点是您必须编写所有代码，包括写入和读取每个字段或属性。`BinaryFormatter` 类就是这样做的。坦白说，这相当方便。

目前实现相同功能的最佳包是 `protobuf-net`。此包可在 NuGet 上找到，这使得在项目中安装变得容易。如果您想使用 `protobuf-net`，您必须在序列化之前对您的类进行注解。再次使用我们的 `MyData` 类，它看起来会是这样：

```cs
[ProtoContract]
public class MyData
{
    [ProtoMember(1)]
    public int Id { get; set; }
    [ProtoMember(2)]
    public double SomeMagicNumber { get; set; }
    [ProtoMember(3)]
    public bool IsThisAGoodDataSet { get; set; }
    [ProtoMember(4)]
    public MyFlags SomeFlags { get; set; }
    [ProtoMember(5)]
    public string? SomeText { get; set; }
}
```

我们用 `ProtoContract` 属性装饰了类。然后，我们用 `ProtoMember` 属性装饰了属性。这个属性可以关联数据，但第一个是必需的。这是标签，它定义了字段在文件中的存储位置。除了一个规则之外，没有关于编号或顺序的硬性规定：你不能从 0 开始。是的。确实如此。我听到你在那里倒吸一口凉气。这是我能想到的唯一一个在编程中从 0 开始是禁止的例子。如果你想从 42 开始，你可以这样做。然而，数字必须是一个正整数，而 0 不是一个正整数。

序列化和反序列化很简单。您必须确保数据可以在内存流中可用或可以写入内存流，但这只是稍微复杂的一件事。这是代码：

```cs
public async Task<byte[]> SerializeToBinary(MyData myData)
{
    await using var stream = new MemoryStream();
    ProtoBuf.Serializer.Serialize(stream, myData);
    return stream.ToArray();
}
public async Task<MyData> DeserializeFromBinary(byte[] payLoad)
{
    await using var stream = new MemoryStream(payLoad);
    var myData =
        ProtoBuf.Serializer.Deserialize<MyData>(stream);
    return myData;
}
```

就这样了。我承诺这会很简单，不是吗？

如果我将与 JSON 序列化相同的数据进行比较，我可以看到二进制版本的数据量要小得多。即使我使用不写入预期文件的选择，从而节省了换行符和制表符，JSON 版本也有 131 字节。相比之下，二进制版本只有 60 字节长。这是一个很大的差异！

# 下一步

I/O 对所有软件都是至关重要的。没有软件是孤立运行的，尤其是为系统编写的软件。毕竟，这些应用程序没有传统的用户界面；它们是供其他软件使用的。与该软件的唯一通信方式是通过以某种方式交换数据。

本章探讨了将数据序列化和反序列化到存储中的方法。我们了解到 JSON 简单且生成可读性强的数据。然而，数据可能相当庞大。相比之下，二进制版本的数据量要小得多，但这样的数据不再适合人类阅读。此外，它还需要第三方包。最佳解决方案是什么？这取决于您的使用场景！无论您使用文件还是网络连接，它们都是 I/O 的方法。在本章中，您看到了如何高效、快速且安全地实现这一点。

然而，对于系统软件来说，一种更高效的方式来通信是通过直接在 **进程间通信**（**IPC**）上进行通信。IPC 是系统软件建立其他软件可以与之交谈或监听的接口层的一个完美方式。这也是下一章的主题。
