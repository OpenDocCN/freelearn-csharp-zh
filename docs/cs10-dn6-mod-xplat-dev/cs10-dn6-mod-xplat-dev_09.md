# 第九章：处理文件、流和序列化

本章涉及文件和流的读写、文本编码和序列化。

我们将涵盖以下主题：

+   管理文件系统

+   使用流进行读写

+   编码和解码文本

+   序列化对象图

+   控制 JSON 处理

# 管理文件系统

您的应用程序经常需要在不同的环境中执行文件和目录的输入和输出操作。`System`和`System.IO`命名空间包含了用于此目的的类。

## 处理跨平台环境和文件系统

让我们探索如何处理跨平台环境，比如 Windows 和 Linux 或 macOS 之间的差异。Windows、macOS 和 Linux 的路径是不同的，所以我们将从探索.NET 如何处理这一点开始：

1.  使用您喜欢的代码编辑器创建一个名为`Chapter09`的新解决方案/工作空间。

1.  添加一个控制台应用程序项目，如下列表所示：

1.  项目模板:**控制台应用程序**/`console`

1.  工作空间/解决方案文件和文件夹：`Chapter09`

1.  项目文件和文件夹：`WorkingWithFileSystems`

1.  在`Program.cs`中，添加语句来静态导入`System.Console`、`System.IO.Directory`、`System.Environment`和`System.IO.Path`类型，如下面的代码所示：

```cs

使用

静态

System.Console;

使用

静态

System.IO.Directory;

使用

静态

System.IO.Path;

使用

静态

System.Environment;

```

1.  在`Program.cs`中，创建一个静态的`OutputFileSystemInfo`方法，并在其中编写语句来执行以下操作：

+   输出路径和目录分隔符。

+   输出当前目录的路径。

+   输出一些系统文件、临时文件和文档的特殊路径。

```cs

静态

void

OutputFileSystemInfo

()

{

WriteLine("{0,-33} {1}"

，arg0："Path.PathSeparator"

，

arg1:PathSeparator);

WriteLine("{0,-33} {1}"

，arg0:"Path.DirectorySeparatorChar"

，

arg1:DirectorySeparatorChar);

WriteLine("{0,-33} {1}"

，arg0："Directory.GetCurrentDirectory()"

，

arg1:GetCurrentDirectory());

WriteLine("{0,-33} {1}"

，arg0:"Environment.CurrentDirectory"

，

arg1:CurrentDirectory);

WriteLine("{0,-33} {1}"

，arg0:"Environment.SystemDirectory"

，

arg1:SystemDirectory);

WriteLine("{0,-33} {1}"

，arg0:"Path.GetTempPath()"

，

arg1:GetTempPath());

WriteLine("GetFolderPath(SpecialFolder"

);

WriteLine("{0,-33} {1}"

，arg0:".System)"

，

arg1:GetFolderPath(SpecialFolder.System));

WriteLine("{0,-33} {1}"

，arg0:".ApplicationData)"

，

arg1:GetFolderPath(SpecialFolder.ApplicationData));

WriteLine("{0,-33} {1}"

，arg0:".MyDocuments)"

，

arg1:GetFolderPath(SpecialFolder.MyDocuments));

WriteLine("{0,-33} {1}"

，arg0:".Personal)"

，

arg1:GetFolderPath(SpecialFolder.Personal));

}

```

`Environment`类型还有许多其他有用的成员，我们在这段代码中没有使用，包括`GetEnvironmentVariables`方法和`OSVersion`和`ProcessorCount`属性。

1.  在`Program.cs`中，在函数上方，调用`OutputFileSystemInfo`方法，如下面的代码所示：

```cs

OutputFileSystemInfo();

```

1.  运行代码并查看结果，如*图 9.1*所示：![自动生成的文本描述](img/Image00086.jpg)

图 9.1：运行应用程序以在 Windows 上显示文件系统信息

在使用 Visual Studio Code 的`dotnet run`运行控制台应用程序时，`CurrentDirectory`将是项目文件夹，而不是`bin`文件夹中的文件夹。

**良好实践**：Windows 使用反斜杠`\`作为目录分隔符。macOS 和 Linux 使用正斜杠`/`作为目录分隔符。在组合路径时，不要假设在您的代码中使用哪个字符。

## 管理驱动器

要管理驱动器，请使用`DriveInfo`类型，该类型具有一个静态方法，返回有关连接到计算机的所有驱动器的信息。每个驱动器都有一个驱动器类型。

让我们探索驱动器：

1.  创建一个`WorkWithDrives`方法，并编写语句来获取所有驱动器并输出它们的名称、类型、大小、可用空间和格式，但仅当驱动器准备就绪时，如下面的代码所示：

```

静态

);

运行代码并查看结果，如*图 9.2*所示：![](img/Image00087.jpg)

()

// 为新文件夹定义一个目录路径

关闭文件以释放系统资源和文件锁（这通常在`try-finally`语句块内完成，以确保即使在写入时发生异常，文件也会关闭）。

,

在`Program.cs`中，注释掉先前的方法调用，并添加对`WorkWithDirectories`的调用。

// 删除目录

, "格式"

GetFolderPath(SpecialFolder.Personal),

在处理文件时，您可以像处理目录类型一样静态导入文件类型，但是，对于下一个示例，我们不会这样做，因为它具有与目录类型相同的一些方法，并且它们会发生冲突。文件类型的名称足够短，在这种情况下并不重要。步骤如下：

);

创建文本文件。

DriveInfo.GetDrives()中

DriveInfo 驱动器）

{

运行代码并查看结果，并使用您喜欢的文件管理工具确认目录已创建后，再按 Enter 键将其删除，如下面的输出所示：

WriteLine("删除它..."

{

WriteLine(

// 从用户文件夹开始

}

, drive.Name, drive.DriveType);

管理目录

```cs

else

WriteLine("{0,-30} | {1,-10} | {2,-7} | {3,18} | {4,18}"

要管理目录，请使用`Directory`，`Path`和`Environment`静态类。这些类型包括许多成员，用于处理文件系统。

);

"代码"

}

（驱动器已准备就绪）

```

, "大小（字节）"

1.  在`Program.cs`中，注释掉先前的方法调用，并添加对`WorkWithDrives`的调用，如下面的代码中所突出显示的那样：

{newFolder}

**// OutputFileSystemInfo();**

void

}

1.  );

,

## {

在构建自定义路径时，必须小心编写代码，以便不对平台（例如，用于目录分隔符字符）做任何假设：

ReadLine();

1.  ```

+   通过创建字符串数组来定义用户主目录下的自定义路径名称，然后使用`Path`类型的`Combine`方法正确组合它们。

+   使用`Directory`类的`Exists`方法检查自定义目录路径是否存在。

+   创建然后删除目录，包括其中的文件和子目录，使用`Directory`类的`CreateDirectory`和`Delete`方法：

```cs

静态

, "可用空间"

{

foreach

drive.TotalSize, drive.AvailableFreeSpace);

WriteLine("创建它..."

, "类型"

图 9.2：在 Windows 上显示驱动器信息

newFolder = Combine(

drive.Name, drive.DriveType, drive.DriveFormat,

```cs

// 创建目录

, "NewFolder"

);

);

将文件复制到备份。

WriteLine("{0,-30} | {1,-10}"

);

创建一个`WorkWithFiles`方法，并编写语句来执行以下操作：

WriteLine($"它存在吗？

**WorkWithDrives();**

"

);

"

// 检查它是否存在

检查文件是否存在。

);

{

{Exists(newFolder)}

向文件写入一行文本。

string

创建一个`WorkWithDirectories`方法，并编写语句来执行以下操作：

确认目录存在，然后按 ENTER 键：

void

, "第九章"

WorkWithDirectories

**良好实践**：在读取`TotalSize`等属性之前，请检查驱动器是否准备就绪，否则将看到异常抛出可移动驱动器。

```cs

Delete(newFolder, recursive: true

"

{Exists(newFolder)}

WriteLine($"它存在吗？

它存在吗？ False

}

```

1.  {Exists(newFolder)}

1.  }

Write("确认目录存在，然后按 ENTER 键："

);

创建它...

它存在吗？ True

如果

删除它...

()

CreateDirectory(newFolder);

## 管理文件

WorkWithDrives

1.  正在处理：/Users/markjprice/Code/Chapter09/NewFolder 它存在吗？ False

1.  "名称"

1.  WriteLine($"正在处理：

1.  "{0,-30} | {1,-10} | {2,-7} | {3,18:N0} | {4,18:N0}"

1.  WriteLine($"它存在吗？

1.  （在

1.  删除原始文件。

1.  读取备份文件的内容，然后关闭它：

```cs

static

void

WorkWithFiles

()

{

// 定义一个输出文件的目录路径

// 从用户文件夹开始

string

dir = Combine(

GetFolderPath(SpecialFolder.Personal),

"代码"

，"Chapter09"

，"OutputFiles"

);

CreateDirectory(dir);

// 定义文件路径

string

textFile = Combine(dir, "Dummy.txt"

);

string

backupFile = Combine(dir，"Dummy.bak"

);

WriteLine($"正在处理：

{textFile}

"

);

// 检查文件是否存在

WriteLine($"它存在吗？

{File.Exists(textFile)}

"

);

// 创建一个新的文本文件并向其写入一行

StreamWriter textWriter = File.CreateText(textFile);

textWriter.WriteLine("Hello, C#!"

);

textWriter.Close(); //关闭文件并释放资源

WriteLine($"它存在吗？

{File.Exists(textFile)}

"

);

// 复制文件，并覆盖如果已存在

File.Copy(sourceFileName: textFile,

destFileName: backupFile, overwrite: true

);

WriteLine(

$"是否存在

{backupFile}

存在吗？

{File.Exists(backupFile)}

"

);

Write("确认文件存在，然后按 ENTER 键："

);

ReadLine();

// 删除文件

File.Delete(textFile);

WriteLine($"它存在吗？

{File.Exists(textFile)}

"

);

// 从文本文件备份中读取

WriteLine($"读取

{backupFile}

:"

);

StreamReader textReader = File.OpenText(backupFile);

WriteLine(textReader.ReadToEnd());

textReader.Close();

}

```

1.  在`Program.cs`中，注释掉先前的方法调用，并添加对`WorkWithFiles`的调用。

1.  运行代码并查看结果，如下输出所示：

```cs

正在处理：/Users/markjprice/Code/Chapter09/OutputFiles/Dummy.txt

它存在吗？ False

它存在吗？ True

/Users/markjprice/Code/Chapter09/OutputFiles/Dummy.bak 存在吗？ True

确认文件存在，然后按 ENTER 键：

它存在吗？ False

读取/Users/markjprice/Code/Chapter09/OutputFiles/Dummy.bak 的内容：

Hello, C#!

```

## 管理路径

有时，您需要处理路径的部分；例如，您可能只想提取文件夹名称、文件名或扩展名。有时，您需要生成临时文件夹和文件名。您可以使用`Path`类的静态方法来实现这一点：

1.  在`WorkWithFiles`方法的末尾添加以下语句：

```cs

// 管理路径

WriteLine($"文件夹名称：

{GetDirectoryName(textFile)}

"

);

WriteLine($"文件名：

{GetFileName(textFile)}

"

);

WriteLine("没有扩展名的文件名：{0}"

，

GetFileNameWithoutExtension(textFile));

WriteLine($"文件扩展名：

{GetExtension(textFile)}

"

);

WriteLine($"随机文件名：

{GetRandomFileName()}

"

);

WriteLine($"临时文件名：

{GetTempFileName()}

"

);

```

1.  运行代码并查看结果，如下输出所示：

```cs

文件夹名称：/Users/markjprice/Code/Chapter09/OutputFiles

文件名：Dummy.txt

没有扩展名的文件名：Dummy

文件扩展名：.txt

随机文件名：u45w1zki.co3

临时文件名：

/var/folders/tz/xx0y_wld5sx0nv0fjtq4tnpc0000gn/T/tmpyqrepP.tmp

```

`GetTempFileName`创建一个零字节的文件并返回其名称，准备供您使用。`GetRandomFileName`只返回一个文件名；它不会创建文件。

## 获取文件信息

要获取有关文件或目录的更多信息，例如其大小或上次访问时间，可以创建`FileInfo`或`DirectoryInfo`类的实例。

`FileInfo`和`DirectoryInfo`都继承自`FileSystemInfo`，因此它们都有诸如`LastAccessTime`和`Delete`之类的成员，以及特定于自身的额外成员，如下表所示：

| 类 | 成员 |
| --- | --- |
| `FileSystemInfo` | Fields: `FullPath`，`OriginalPath`Properties: `Attributes`，`CreationTime`，`CreationTimeUtc`，`Exists`，`Extension`，`FullName`，`LastAccessTime`，`LastAccessTimeUtc`，`LastWriteTime`，`LastWriteTimeUtc`，`Name`Methods: `Delete`，`GetObjectData`，`Refresh` |
| `DirectoryInfo` | 属性：`Parent`，`Root`方法：`Create`，`CreateSubdirectory`，`EnumerateDirectories`，`EnumerateFiles`，`EnumerateFileSystemInfos`，`GetAccessControl`，`GetDirectories`，`GetFiles`，`GetFileSystemInfos`，`MoveTo`，`SetAccessControl` |
| `FileInfo` | 属性：`Directory`，`DirectoryName`，`IsReadOnly`，`Length`方法：`AppendText`，`CopyTo`，`Create`，`CreateText`，`Decrypt`，`Encrypt`，`GetAccessControl`，`MoveTo`，`Open`，`OpenRead`，`OpenText`，`OpenWrite`，`Replace`，`SetAccessControl` |

让我们编写一些使用`FileInfo`实例的代码，以便有效地对文件执行多个操作：

1.  在`WorkWithFiles`方法的末尾添加语句，为备份文件创建一个`FileInfo`实例，并将有关它的信息写入控制台，如下所示的代码：

```cs

FileInfo info = new

(backupFile);

WriteLine($"

{backupFile}

:"

);

WriteLine($"包含

{info.Length}

字节"

);

WriteLine($"上次访问

{info.LastAccessTime}

"

);

WriteLine($"具有只读设置为

{info.IsReadOnly}

"

);

```

1.  运行代码并查看结果，如下所示：

```cs

/Users/markjprice/Code/Chapter09/OutputFiles/Dummy.bak：

包含 11 个字节

上次访问 26/10/2021 09:08:26

具有只读设置为 False

```

字节数可能因您的操作系统而异，因为操作系统可以使用不同的换行符。

## 控制如何处理文件

在处理文件时，通常需要控制它们的打开方式。`File.Open`方法有多个重载，可以使用`enum`值指定附加选项。

`enum`类型如下：

+   `FileMode`：这控制您要对文件执行的操作，比如`CreateNew`，`OpenOrCreate`或`Truncate`。

+   `FileAccess`：这控制您需要的访问级别，比如`ReadWrite`。

+   `FileShare`：这控制文件上的锁定，以允许其他进程指定级别的访问，比如`Read`。

您可能想要打开一个文件并从中读取，并允许其他进程也读取它，如下所示的代码：

```cs

FileStream file = File.Open(pathToFile，

FileMode.Open, FileAccess.Read, FileShare.Read);

```

还有一个文件属性的`enum`如下：

+   `FileAttributes`：这是为了检查`FileSystemInfo` -派生类型的`Attributes`属性的值，如`Archive`和`Encrypted`。

您可以检查文件或目录的属性，如下所示的代码：

```cs

FileInfo info = new

(backupFile);

WriteLine("备份文件是否压缩？{0}"

，

info.Attributes.HasFlag(FileAttributes.Compressed));

```

# 使用流进行读写

**流**是可以从中读取和写入的字节序列。尽管文件可以像数组一样进行处理，通过知道文件中的字节位置提供随机访问，但将文件作为可以按顺序访问的流进行处理可能是有用的。

流还可以用于处理终端输入和输出以及不提供随机访问并且不能寻找（即移动）到位置的网络资源，如套接字和端口。您可以编写代码来处理一些任意字节，而不知道或关心它来自何处。您的代码只需读取或写入流，另一段代码处理字节实际存储的位置。

## 理解抽象和具体流

有一个名为`Stream`的`abstract`类，代表任何类型的流。请记住，`abstract`类不能使用`new`实例化；它们只能被继承。

有许多具体的类从这个基类继承，包括`FileStream`，`MemoryStream`，`BufferedStream`，`GZipStream`和`SslStream`，所以它们都以相同的方式工作。所有流都实现了`IDisposable`，因此它们有一个`Dispose`方法来释放非托管资源。

`Stream`类的一些常见成员描述在下表中：

| 成员 | 描述 |
| --- | --- |
| `CanRead`，`CanWrite` | 这些属性确定是否可以从流中读取和写入。 |
| `Length`，`Position` | 这些属性确定流中的总字节数和当前位置。这些属性对某些类型的流可能会抛出异常。 |
| `Dispose` | 这个方法关闭流并释放其资源。 |
| `Flush` | 如果流有缓冲区，则此方法将缓冲区中的字节写入流，并清除缓冲区。 |
| `CanSeek` | 此属性确定是否可以使用`Seek`方法。 |
| `Seek` | 这个方法将当前位置移动到其参数中指定的位置。 |
| `Read`，`ReadAsync` | 这些方法从流中读取指定数量的字节到一个字节数组中，并推进位置。 |
| `ReadByte` | 这个方法读取流中的下一个字节并推进位置。 |
| `Write`，`WriteAsync` | 这些方法将字节数组的内容写入流中。 |
| `WriteByte` | 这个方法向流中写入一个字节。 |

### 理解存储流

一些表示将存储字节的位置的存储流在下表中描述：

| 命名空间 | 类 | 描述 |
| --- | --- | --- |
| `System.IO` | `FileStream` | 在文件系统中存储的字节。 |
| `System.IO` | `MemoryStream` | 在当前进程中存储的字节。 |
| `System.Net.Sockets` | `NetworkStream` | 存储在网络位置的字节。 |

`FileStream`在.NET 6 中已经重写，以在 Windows 上具有更高的性能和可靠性。

### 理解功能流

一些功能流不能单独存在，但只能“插入”到其他流中以添加功能，这些流在下表中描述：

| 命名空间 | 类 | 描述 |
| --- | --- | --- |
| `System.Security.Cryptography` | `CryptoStream` | 这个加密和解密流。 |
| `System.IO.Compression` | `GZipStream`，`DeflateStream` | 这些压缩和解压缩流。 |
| `System.Net.Security` | `AuthenticatedStream` | 这在流中发送凭据。 |

### 理解流助手

尽管有时您需要以低级别处理流，但大多数情况下，您可以将辅助类插入到链中以使事情变得更容易。所有流的辅助类型都实现了`IDisposable`，因此它们具有`Dispose`方法来释放非托管资源。

一些处理常见情况的辅助类在下表中描述：

| 命名空间 | 类 | 描述 |
| --- | --- | --- |
| `System.IO` | `StreamReader` | 这以纯文本形式从基础流中读取。 |
| `System.IO` | `StreamWriter` | 这以纯文本形式写入基础流。 |
| `System.IO` | `BinaryReader` | 这以.NET 类型从流中读取。例如，`ReadDecimal`方法将作为`decimal`值的下一个 16 个字节从基础流中读取，`ReadInt32`方法将作为`int`值的下一个 4 个字节读取。 |
| `System.IO` | `BinaryWriter` | 这将以.NET 类型写入流。例如，带有`decimal`参数的`Write`方法将 16 个字节写入基础流，带有`int`参数的`Write`方法将写入 4 个字节。 |
| `System.Xml` | `XmlReader` | 这以 XML 格式从基础流中读取。 |
| `System.Xml` | `XmlWriter` | 这以 XML 格式写入基础流。 |

## 写入文本流

让我们输入一些代码来向流中写入文本：

1.  使用您喜欢的代码编辑器将名为`WorkingWithStreams`的新控制台应用添加到`Chapter09`解决方案/工作空间中：

1.  在 Visual Studio 中，将解决方案的启动项目设置为当前选择。

1.  在 Visual Studio Code 中，将`WorkingWithStreams`选择为活动的 OmniSharp 项目。

1.  在`WorkingWithStreams`项目的`Program.cs`中，导入`System.Xml`命名空间，并静态导入`System.Console`，`System.Environment`和`System.IO.Path`类型。

1.  在`Program.cs`的底部，定义一个名为`Viper`的静态类，其中包含名为`Callsigns`的静态`string`值数组，如下面的代码所示：

```cs

静态

类

Viper

{

//定义 Viper 飞行员呼号数组

public

静态

string

[] Callsigns = new

[]

{

“Husker”

，“Starbuck”

，“Apollo”

，“Boomer”

，

“牛头犬”

，“Athena”

，“Helo”

，“Racetrack”

};

}

```

1.  在`Viper`类的上方，定义一个`WorkWithText`方法，该方法枚举 Viper 呼号，将每个呼号单独写入单个文本文件的一行，如下面的代码所示：

```cs

静态

void

WorkWithText

()

{

//定义要写入的文件

string

textFile = Combine(CurrentDirectory, "streams.txt"

);

//创建一个文本文件并返回一个辅助写入器

StreamWriter text = File.CreateText(textFile);

//枚举字符串，写入每个字符串

//在单独的行上写入流

foreach

(string

item in

Viper.Callsigns)

{

text.WriteLine(item);

}

text.Close(); //释放资源

//输出文件的内容

WriteLine("{0} 包含 {1:N0} 字节。"

，

arg0: textFile,

arg1: new

FileInfo(textFile).Length);

WriteLine(File.ReadAllText(textFile));

}

```

1.  在命名空间导入下方，调用`WorkWithText`方法。

1.  运行代码并查看结果，如下面的输出所示：

```cs

/Users/markjprice/Code/Chapter09/WorkingWithStreams/streams.txt 包含

60 字节。

Husker

Starbuck

Apollo

Boomer

Bulldog

Athena

Helo

Racetrack

```

1.  打开已创建的文件并检查其中是否包含呼号列表。

## 写入 XML 流

有两种方法可以写入 XML 元素，如下所示：

+   `WriteStartElement`和`WriteEndElement`：当元素可能有子元素时使用此对。

+   `WriteElementString`：当元素没有子元素时使用。

现在，让我们尝试将`string`值的 Viper 飞行员呼号数组存储在 XML 文件中：

1.  创建一个`WorkWithXml`方法，该方法枚举呼号，并将每个呼号作为单个 XML 文件中的元素写入，如下面的代码所示：

```cs

静态

void

WorkWithXml

()

{

//定义要写入的文件

string

xmlFile = Combine(CurrentDirectory, "streams.xml"

);

//创建文件流

FileStream xmlFileStream = File.Create(xmlFile);

//将文件流包装在 XML 写入器助手中

//并自动缩进嵌套元素

XmlWriter xml = XmlWriter.Create(xmlFileStream,

new

XmlWriterSettings { Indent = true

});

//写入 XML 声明

xml.WriteStartDocument();

//写入根元素

xml.WriteStartElement("callsigns"

);

//枚举字符串，将每个字符串写入流

foreach

(string

item in

Viper.Callsigns)

{

xml.WriteElementString("callsign"

，item);

}

//写入关闭根元素

xml.WriteEndElement();

//关闭辅助和流

xml.Close();

xmlFileStream.Close();

//输出文件的所有内容

WriteLine("{0} 包含 {1:N0} 字节。"

，

arg0: xmlFile,

arg1: new

FileInfo(xmlFile).Length);

WriteLine(File.ReadAllText(xmlFile));

}

```

1.  在`Program.cs`中，注释掉先前的方法调用，并添加对`WorkWithXml`方法的调用。

1.  运行代码并查看结果，如下面的输出所示：

```cs

/Users/markjprice/Code/Chapter09/WorkingWithStreams/streams.xml 包含

310 字节。

<?xml version="1.0" encoding="utf-8"?>

<callsigns>

<callsign> Husker </callsign>

<callsign> Starbuck </callsign>

<callsign> Apollo </callsign>

<callsign> Boomer </callsign>

<callsign> Bulldog </callsign>

<callsign> Athena </callsign>

<callsign> Helo </callsign>

<callsign> Racetrack </callsign>

</callsigns>

```

## 处理文件资源

当您打开文件以读取或写入文件时，您正在使用.NET 之外的资源。这些被称为**非托管资源**，在完成工作后必须将其处理掉。为了确定性地控制它们何时被处理掉，我们可以在`finally`块内调用`Dispose`方法。

让我们改进之前处理 XML 的代码，以正确处理其非托管资源：

1.  修改`WorkWithXml`方法，如下面的代码所示：

```cs

静态

void

WorkWithXml

()

{

**FileStream? xmlFileStream =**

**null**

**;**

**XmlWriter? xml =**

**null**

**;**

**try**

**{**

// 定义要写入的文件

string

xmlFile = Combine(CurrentDirectory, "streams.xml"

);

// 创建文件流

**xmlFileStream = File.Create(xmlFile);**

// 将文件流包装在 XML 写入器助手中

// 并自动缩进嵌套元素

**xml = XmlWriter.Create(xmlFileStream,**

**new**

**XmlWriterSettings { Indent =**

**true**

**});**

// 写入 XML 声明

xml.WriteStartDocument();

// 写入根元素

xml.WriteStartElement("callsigns"

);

// 枚举字符串，将每个字符串写入流

foreach

(string

项目

Viper.Callsigns)

{

xml.WriteElementString("callsign"

, item);

}

// 写入关闭根元素

xml.WriteEndElement();

// 关闭辅助程序和流

xml.Close();

xmlFileStream.Close();

// 输出文件的所有内容

WriteLine($"

{

0

}

包含

{

1

:N0}

字节。"

,

arg0: xmlFile,

arg1: new

FileInfo(xmlFile).Length);

WriteLine(File.ReadAllText(xmlFile));

**}**

**catch (Exception ex)**

**{**

**// 如果路径不存在，异常将被捕获**

**WriteLine(**

**$"**

**{ex.GetType()}**

**说**

**{ex.Message}**

**"**

**);**

**}**

**finally**

**{**

**如果**

**(xml !=**

**null**

**)**

**{**

**xml.Dispose();**

**WriteLine(**

**"XML 写入器的非托管资源已被处理。"**

**);**

**if**

**(xmlFileStream !=**

**null**

**)**

**{**

**xmlFileStream.Dispose();**

**WriteLine(**

**"文件流的非托管资源已被处理。"**

**);**

**}**

**}**

**}**

}

```

您还可以返回并修改先前创建的其他方法，但我将把它作为一个可选练习留给您。

1.  运行代码并查看结果，如下所示：

```cs

XML 写入器的非托管资源已被处理。

文件流的非托管资源已被处理。

```

**良好的实践**：在调用`Dispose`方法之前，检查对象是否为空。

### 通过使用`using`语句简化处理

您可以通过使用`using`语句简化需要检查`null`对象然后调用其`Dispose`方法的代码。通常，我建议使用`using`而不是手动调用`Dispose`，除非您需要更高级别的控制。

令人困惑的是，`using`关键字有两种用法：导入命名空间和生成调用实现`IDisposable`的对象的`Dispose`的`finally`语句。

编译器将`using`语句块更改为`try`-`finally`语句，而不带`catch`语句。您可以使用嵌套的`try`语句；因此，如果确实想要捕获任何异常，可以像下面的代码示例中所示的那样捕获异常：

```cs

使用

(FileStream file2 = File.OpenWrite(

Path.Combine(path, "file2.txt"

)))

{

using

(StreamWriter writer2 = new

StreamWriter(file2))

{

try

{

writer2.WriteLine("欢迎，.NET!"

);

}

catch(Exception ex)

{

WriteLine($"

{ex.GetType()}

说

{ex.Message}

"

);

}

} // 如果对象不为空，则自动调用 Dispose

} // 如果对象不为空，则自动调用 Dispose

```

甚至可以通过不明确指定`using`语句的大括号和缩进来进一步简化代码，如下面的代码所示：

```cs

using

FileStream file2 = File.OpenWrite(

Path.Combine(path, "file2.txt"

));

using

StreamWriter writer2 = new

(file2);

try

{

writer2.WriteLine("欢迎，.NET!"

);

}

catch(Exception ex)

{

WriteLine($"

{ex.GetType()}

说

{ex.Message}

"

);

}

```

## 压缩流

XML 相对冗长，因此占用的字节数比纯文本多。让我们看看如何使用一种称为 GZIP 的常见压缩算法来压缩 XML：

1.  在`Program.cs`的顶部，导入用于处理压缩的命名空间，如下面的代码所示：

```cs

using

System.IO.Compression; // BrotliStream, GZipStream, CompressionMode

```

1.  添加一个`WorkWithCompression`方法，该方法使用`GZipStream`的实例创建一个包含与以前相同的 XML 元素的压缩文件，然后在读取并输出到控制台时对其进行解压缩，如下面的代码所示：

```cs

static

void

WorkWithCompression

()

{

字符串

fileExt = "gzip"

;

// 压缩 XML 输出

字符串

filePath = Combine(

CurrentDirectory, $"streams.

**{fileExt}**

"

）;

FileStream file = File.Create(filePath);

Stream compressor = new

GZipStream(file, CompressionMode.Compress);

使用

（compressor）

{

使用

（XmlWriter xml = XmlWriter.Create(compressor))

{

xml.WriteStartDocument();

xml.WriteStartElement("callsigns"

);

foreach

（字符串

item in

Viper.Callsigns)

{

xml.WriteElementString("callsign"

， item);

}

// 正常调用 WriteEndElement 不是必要的

// 因为当 XmlWriter 释放时，它将

// 自动结束任何深度的元素

}

} // 也关闭底层流

// 输出压缩文件的所有内容

WriteLine("{0} contains {1:N0} bytes."

，

filePath, new

FileInfo(filePath).Length);

WriteLine($"The compressed contents:"

）；

WriteLine(File.ReadAllText(filePath));

// 读取压缩文件

WriteLine("Reading the compressed XML file:"

）；

file = File.Open(filePath, FileMode.Open);

Stream decompressor = new

GZipStream(file,

CompressionMode.Decompress);

using

（decompressor）

{

使用

(XmlReader reader = XmlReader.Create(decompressor))

{

while

（reader.Read()） // 读取下一个 XML 节点

{

// 检查我们是否在名为 callsign 的元素节点上

if

((reader.NodeType == XmlNodeType.Element)

&& (reader.Name == "callsign"

))

{

reader.Read(); // 移动到元素内部的文本

WriteLine($"

{reader.Value}

"

）； // 读取其值

}

}

}

}

}

```

1.  在`Program.cs`中，保留对`WorkWithXml`的调用，并添加对`WorkWithCompression`的调用，如下面代码中突出显示的那样：

```cs

// WorkWithText();

**WorkWithXml();**

**WorkWithCompression();**

```

1.  运行代码并比较 XML 文件和压缩的 XML 文件的大小。它比没有压缩的相同 XML 文件的大小少一半，如下编辑后的输出所示：

```cs

/Users/markjprice/Code/Chapter09/WorkingWithStreams/streams.xml 包含 310 字节。

/Users/markjprice/Code/Chapter09/WorkingWithStreams/streams.gzip 包含 150 字节。

```

## 使用 Brotli 算法压缩

在.NET Core 2.1 中，微软引入了 Brotli 压缩算法的实现。在性能上，Brotli 类似于 DEFLATE 和 GZIP 中使用的算法，但输出约密度高 20%。步骤如下：

1.  修改`WorkWithCompression`方法，使其具有一个可选参数，指示是否应使用 Brotli，并默认使用 Brotli，如下面代码中突出显示的那样：

```cs

static

void

WorkWithCompression

（

**bool**

**useBrotli =**

**true**

）

{

字符串

fileExt =

**useBrotli ?**

**"brotli"**

**：**

**"gzip"**

**;**

// 压缩 XML 输出

字符串

filePath = Combine(

CurrentDirectory, $"streams.

{fileExt}

"

）；

FileStream file = File.Create(filePath);

**Stream compressor;**

**if**

**（useBrotli）**

**{**

**compressor =**

**new**

**BrotliStream(file, CompressionMode.Compress);**

**}**

**else**

**{**

**compressor =**

**new**

**GZipStream(file, CompressionMode.Compress);**

**}**

使用

（compressor）

{

使用

(XmlWriter xml = XmlWriter.Create(compressor))

{

xml.WriteStartDocument();

xml.WriteStartElement("callsigns"

）；

foreach

（字符串

item in

Viper.Callsigns)

{

xml.WriteElementString("callsign"

， item);

}

}

} // 也关闭底层流

// 输出压缩文件的所有内容

WriteLine("{0} contains {1:N0} bytes."

，

filePath, new

FileInfo(filePath).Length);

WriteLine($"The compressed contents:"

）；

WriteLine(File.ReadAllText(filePath));

// 读取压缩文件

WriteLine("Reading the compressed XML file:"

）；

file = File.Open(filePath, FileMode.Open);

**Stream decompressor;**

**if**

**（useBrotli）**

**{**

**decompressor =**

**new**

**BrotliStream(**

**file, CompressionMode.Decompress);**

**}**

**else**

**{**

解压器 =

**new**

**GZipStream(**

**file, CompressionMode.Decompress);**

**}**

using

(decompressor)

{

using

(XmlReader reader = XmlReader.Create(decompressor))

{

while

(reader.Read())

{

// 检查是否在名为 callsign 的元素节点上

如果

((reader.NodeType == XmlNodeType.Element)

&& (reader.Name == "callsign"

))

{

reader.Read(); // 移动到元素内部的文本

WriteLine($"

{reader.Value}

"

); // 读取其值

}

}

}

}

}

```

1.  在`Program.cs`的顶部附近，调用`WorkWithCompression`两次，一次使用默认的 Brotli，一次使用 GZIP，如下面的代码所示：

```cs

WorkWithCompression();

WorkWithCompression(useBrotli: false

);

```

1.  运行代码并比较两个压缩的 XML 文件的大小。如下编辑后的输出所示，Brotli 比 GZIP 更密集，超过 21%：

```cs

/Users/markjprice/Code/Chapter09/WorkingWithStreams/streams.brotli 包含 118 个字节。

/Users/markjprice/Code/Chapter09/WorkingWithStreams/streams.gzip 包含 150 个字节。

```

# 编码和解码文本

文本字符可以用不同的方式表示。例如，字母表可以使用莫尔斯电码编码成一系列点和破折号，以在电报线上传输。

类似地，计算机内部的文本以位（1 和 0）的形式存储，表示代码空间内的代码点。大多数代码点代表一个字符，但它们也可以有其他含义，比如格式化。

例如，ASCII 具有 128 个代码点的代码空间。.NET 使用一种称为**Unicode**的标准来内部编码文本。Unicode 有一百多万个代码点。

有时，您需要将文本移出.NET 以供不使用 Unicode 或使用 Unicode 变体的系统使用，因此学习如何在编码之间转换是很重要的。

以下表格列出了计算机常用的一些替代文本编码：

| 编码 | 描述 |
| --- | --- |
| ASCII | 这使用字节的低 7 位来编码有限范围的字符。 |
| UTF-8 | 这将每个 Unicode 代码点表示为一个到四个字节的序列。 |
| UTF-7 | 这是为了在 7 位通道上比 UTF-8 更有效而设计的，但它存在安全性和健壮性问题，因此建议使用 UTF-8 而不是 UTF-7。 |
| UTF-16 | 这将每个 Unicode 代码点表示为一个或两个 16 位整数的序列。 |
| UTF-32 | 这将每个 Unicode 代码点表示为一个 32 位整数，因此是一种固定长度编码，不同于其他 Unicode 编码，它们都是可变长度编码。 |
| ANSI/ISO 编码 | 这提供了对用于支持特定语言或语言组的各种代码页的支持。 |

**良好实践**：在大多数情况下，UTF-8 是一个很好的默认选择，这就是为什么它实际上是默认编码，即`Encoding.Default`。

## 将字符串编码为字节数组

让我们探索文本编码：

1.  使用您喜欢的代码编辑器将一个名为`WorkingWithEncodings`的新控制台应用添加到`Chapter09`解决方案/工作区中。

1.  在 Visual Studio Code 中，将`WorkingWithEncodings`选择为活动的 OmniSharp 项目。

1.  在`Program.cs`中，导入`System.Text`命名空间并静态导入`Console`类。

1.  添加语句，使用用户选择的编码对`string`进行编码，循环遍历每个字节，然后将其解码回`string`并输出，如下面的代码所示：

```cs

WriteLine("编码"

);

WriteLine("[1] ASCII"

);

WriteLine("[2] UTF-7"

);

WriteLine("[3] UTF-8"

);

WriteLine("[4] UTF-16 (Unicode)"

);

WriteLine("[5] UTF-32"

);

WriteLine("[任意其他键] 默认"

);

// 选择一个编码

Write("按数字选择编码："

);

ConsoleKey number = ReadKey(intercept: false

).Key;

WriteLine();

WriteLine();

编码编码器 = 数字开关

{

ConsoleKey.D1 => Encoding.ASCII,

ConsoleKey.D2 => Encoding.UTF7,

ConsoleKey.D3 => Encoding.UTF8,

ConsoleKey.D4 => Encoding.Unicode,

ConsoleKey.D5 => Encoding.UTF32,

_             => Encoding.Default

};

定义一个要编码的字符串

字符串

message = "Café cost: £4.39"

;

// 将字符串编码为字节数组

字节

[] encoded = encoder.GetBytes(message);

// 检查编码需要多少字节

WriteLine("{0} 使用 {1:N0} 字节。"

，

encoder.GetType().Name, encoded.Length);

WriteLine();

// 枚举每个字节

WriteLine($"BYTE HEX CHAR"

);

foreach

(byte

b in

encoded)

{

WriteLine($"

{b,

4

}

{b.ToString(

"X"

),

4

}

{(

char

)b,

5

}

"

);

}

// 将字节数组解码回字符串并显示它

字符串

decoded = encoder.GetString(encoded);

WriteLine(decoded);

```

1.  运行代码并注意警告，避免使用`Encoding.UTF7`，因为它是不安全的。当然，如果您需要使用该编码生成文本以与另一个系统兼容，则需要将其保留为.NET 中的一个选项。

1.  按 1 选择 ASCII 并注意，当输出字节时，英镑符号（£）和重音 e（é）无法用 ASCII 表示，因此它使用问号代替。

```cs

BYTE  HEX  CHAR

67   43     C

97   61     a

102   66     f

63   3F     ?

32   20

111   6F     o

115   73     s

116   74     t

58   3A     :

32   20

63   3F     ?

52   34     4

46   2E     .

51   33     3

57   39     9

Caf? cost: ?4.39

```

1.  重新运行代码并按 3 选择 UTF-8，并注意 UTF-8 需要两个额外的字节来存储每个需要 2 个字节的字符（总共 18 个字节而不是 16 个字节），但它可以编码和解码é和£字符。

```cs

UTF8EncodingSealed 使用 18 个字节。

BYTE  HEX  CHAR

67   43     C

97   61     a

102   66     f

195   C3     Ã

169   A9     ©

32   20

111   6F     o

115   73     s

116   74     t

58   3A     :

32   20

194   C2     Â

163   A3     £

52   34     4

46   2E     .

51   33     3

57   39     9

Café cost: £4.39

```

1.  重新运行代码并按 4 选择 Unicode（UTF-16），注意 UTF-16 对每个字符需要两个字节，因此总共需要 32 个字节，并且它可以编码和解码é和£字符。这种编码在.NET 内部用于存储`char`和`string`值。

## 在文件中编码和解码文本

在使用流辅助类（如`StreamReader`和`StreamWriter`）时，可以指定要使用的编码。当您写入辅助程序时，文本将自动编码，当您从辅助程序读取时，字节将自动解码。

要指定编码，请将编码作为辅助类型的构造函数的第二个参数传递，如下面的代码所示：

```cs

StreamReader reader = new

(stream, Encoding.UTF8);

StreamWriter writer = new

(stream, Encoding.UTF8);

```

**良好实践**：通常，您无法选择要使用的编码，因为您将生成供另一个系统使用的文件。但是，如果您可以选择，请选择使用最少数量的字节但可以存储您需要的每个字符的编码。

# 序列化对象图

**序列化** 是将活动对象转换为使用指定格式的字节序列的过程。**反序列化** 是相反的过程。您可以这样做来保存活动对象的当前状态，以便将来可以重新创建它。例如，保存游戏的当前状态，以便您可以在明天继续。序列化的对象通常存储在文件或数据库中。

您可以指定数十种格式，但最常见的两种格式是**可扩展标记语言**（**XML**）和**JavaScript 对象表示**（**JSON**）。

**良好实践**：JSON 更紧凑，最适合 Web 和移动应用程序。XML 更冗长，但在更多的旧系统中得到更好的支持。使用 JSON 来最小化序列化对象图的大小。JSON 也是将对象图发送到 Web 应用程序和移动应用程序的良好选择，因为 JSON 是 JavaScript 和移动应用程序的本机序列化格式，这些应用程序通常通过有限的带宽进行调用，因此字节数量很重要。

.NET 有多个类可以将 XML 和 JSON 序列化和反序列化。我们将首先看看`XmlSerializer`和`JsonSerializer`。

## 将 XML 序列化

让我们首先看看 XML，可能是世界上最常用的序列化格式（目前）。为了展示一个典型的例子，我们将定义一个自定义类来存储有关一个人的信息，然后使用`Person`实例的列表创建一个对象图并进行嵌套：

1.  使用您喜欢的代码编辑器将一个名为`WorkingWithSerialization`的新控制台应用添加到`Chapter09`解决方案/工作区中。

1.  在 Visual Studio Code 中，选择`WorkingWithSerialization`作为活动的 OmniSharp 项目。

1.  添加一个名为`Person`的类，其中包含一个`Salary`属性，该属性是`protected`，意味着它只能被自身和派生类访问。为了填充工资，该类有一个带有单个参数的构造函数来设置初始工资，如下面的代码所示：

```cs

namespace

Packt.Shared

;

public

class

Person

{

public

Person

(

decimal

initialSalary

)

{

Salary = initialSalary;

}

public

string

? FirstName { get

; set

; }

public

string

? LastName { get

; set

; }

public

DateTime DateOfBirth { get

; set

; }

public

HashSet<Person>? Children { get

; set

; }

protected

decimal

Salary { get

; set

; }

}

```

1.  在`Program.cs`中，导入用于处理 XML 序列化的命名空间，并静态导入`Console`，`Environment`和`Path`类，如下面的代码所示：

```cs

using

System.Xml.Serialization; // XmlSerializer

using

Packt.Shared; // Person

using

static

System.Console;

using

static

System.Environment;

using

static

System.IO.Path;

```

1.  添加语句来创建一个`Person`实例的对象图，如下面的代码所示：

```cs

// 创建一个对象图

List<Person> people = new

()

{

new

(30000

M)

{

FirstName = "Alice"

，

LastName = "Smith"

，

DateOfBirth = new

(1974

, 3

, 14

)

},

new

(40000

M)

{

FirstName = "Bob"

，

LastName = "Jones"

，

DateOfBirth = new

(1969

, 11

, 23

)

},

new

(20000

M)

{

FirstName = "Charlie"

，

LastName = "Cox"

，

DateOfBirth = new

(1984

, 5

, 4

),

Children = new

()

{

new

(0

M)

{

FirstName = "Sally"

，

LastName = "Cox"

，

DateOfBirth = new

(2000

, 7

, 12

)

}

}

}

};

// 创建一个将 List of Persons 格式化为 XML 的对象

XmlSerializer xs = new

(people.GetType());

// 创建一个要写入的文件

string

path = Combine(CurrentDirectory, "people.xml"

);

using

(FileStream stream = File.Create(path))

{

// 将对象图序列化到流中

xs.Serialize(stream, people);

}

WriteLine("已将{0:N0}字节的 XML 写入到{1}"

，

arg0: new

FileInfo(path).Length,

arg1: path);

WriteLine();

// 显示序列化的对象图

WriteLine(File.ReadAllText(path));

```

1.  运行代码，查看结果，注意会抛出异常，如下面的输出所示：

```cs

未处理的异常: System.InvalidOperationException: Packt.Shared.Person 无法序列化，因为它没有无参数的构造函数。

```

1.  在`Person`中，添加一个语句来定义一个无参数的构造函数，如下面的代码所示：

```cs

public

Person

()

{ }

```

构造函数不需要做任何事情，但必须存在，以便`XmlSerializer`在反序列化过程中调用它来实例化新的`Person`实例。

1.  重新运行代码并查看结果，注意对象图被序列化为 XML 元素，如`<FirstName>Bob</FirstName>`，并且`Salary`属性未包括在内，因为它不是一个`public`属性，如下面的输出所示：

```cs

已将 752 字节的 XML 写入

/Users/markjprice/Code/Chapter09/WorkingWithSerialization/people.xml

<?xml version="1.0"?>

<ArrayOfPerson  >

<Person>

<FirstName>Alice</FirstName>

<LastName>Smith</LastName>

<DateOfBirth>1974-03-14T00:00:00</DateOfBirth>

</Person>

<Person>

<FirstName>Bob</FirstName>

<LastName>Jones</LastName>

<DateOfBirth>1969-11-23T00:00:00</DateOfBirth>

</Person>

<Person>

<FirstName>Charlie</FirstName>

<LastName>Cox</LastName>

<DateOfBirth>1984-05-04T00:00:00</DateOfBirth>

<Children>

<Person>

<FirstName>Sally</FirstName>

<LastName>Cox</LastName>

<DateOfBirth>2000-07-12T00:00:00</DateOfBirth>

</Person>

</Children>

</Person>

</ArrayOfPerson>

```

## 生成紧凑的 XML

我们可以使用属性而不是元素使 XML 更紧凑：

1.  在`Person`中，导入`System.Xml.Serialization`命名空间，以便可以使用`[XmlAttribute]`属性装饰一些属性。

1.  装饰名、姓和出生日期属性与`[XmlAttribute]`属性，并为每个属性设置一个简短的名称，如下面的代码中所示：

```cs

**[**

**XmlAttribute(**

**"fname"**

**)**

**]**

public

string

FirstName { get

; set

; }

**[**

**XmlAttribute(**

**"lname"**

**)**

**]**

public

string

LastName { get

; set

; }

**[**

**XmlAttribute(**

**"dob"**

**)**

**]**

public

DateTime DateOfBirth { get

; set

; }

```

1.  运行代码并注意文件的大小已从 752 减少到 462 字节，通过将属性值输出为 XML 属性，节省了超过三分之一的空间，如下面的输出所示：

```cs

已将 462 字节的 XML 写入到/Users/markjprice/Code/Chapter09/ WorkingWithSerialization/people.xml

<?xml version="1.0"?>

<ArrayOfPerson  >

<Person fname="Alice" lname="Smith" dob="1974-03-14T00:00:00" />

<Person fname="Bob" lname="Jones" dob="1969-11-23T00:00:00" />

<Person fname="Charlie" lname="Cox" dob="1984-05-04T00:00:00">

<Children>

<Person fname="Sally" lname="Cox" dob="2000-07-12T00:00:00" />

</Children>

</Person>

</ArrayOfPerson>

```

## 反序列化 XML 文件

现在让我们尝试将 XML 文件反序列化为内存中的实时对象：

1.  添加语句来打开 XML 文件，然后对其进行反序列化，如下面的代码所示：

```cs

using

(FileStream xmlLoad = File.Open(path, FileMode.Open))

{

// 反序列化并将对象图转换为 Person 列表

List<Person>? loadedPeople =

xs.Deserialize(xmlLoad) as

List<Person>;

if

(loadedPeople is

not

null

)

{

foreach

(Person p in

loadedPeople)

{

WriteLine("{0}有{1}个孩子。"

,

p.LastName, p.Children?.Count ?? 0

);

}

}

}

```

1.  运行代码并注意人们已成功从 XML 文件加载，然后被枚举，如下面的输出所示：

```cs

Smith has 0 children.

Jones has 0 children.

Cox has 1 children.

```

还有许多其他属性可以用来控制生成的 XML。

如果您不使用任何注释，`XmlSerializer`在反序列化时使用属性名称进行不区分大小写的匹配。

**良好实践**：使用`XmlSerializer`时，请记住只有公共字段和属性会被包括在内，并且类型必须具有无参数的构造函数。您可以使用属性自定义输出。

## 使用 JSON 进行序列化

.NET 中最受欢迎的用于处理 JSON 序列化格式的库之一是 Newtonsoft.Json，也称为 Json.NET。它是成熟且功能强大的。让我们看看它的作用：

1.  在`WorkingWithSerialization`项目中，添加对`Newtonsoft.Json`最新版本的包引用，如下标记所示：

```cs

<ItemGroup>

<PackageReference Include="Newtonsoft.Json"

Version="13.0.1"

/>

</ItemGroup>

```

1.  构建`WorkingWithSerialization`项目以还原包。

1.  在`Program.cs`中，添加语句创建一个文本文件，然后将人们序列化为 JSON 格式写入文件，如下面的代码所示：

```cs

// 创建一个要写入的文件

string

jsonPath = Combine(CurrentDirectory, "people.json"

);

using

(StreamWriter jsonStream = File.CreateText(jsonPath))

{

// 创建一个将格式化为 JSON 的对象

Newtonsoft.Json.JsonSerializer jss = new

();

// 将对象图序列化为字符串

jss.Serialize(jsonStream, people);

}

WriteLine();

WriteLine("已将 {0:N0} 字节的 JSON 写入到：{1}"

,

arg0: new

FileInfo(jsonPath).Length,

arg1: jsonPath);

// 显示序列化的对象图

WriteLine(File.ReadAllText(jsonPath));

```

1.  运行代码并注意，与使用元素的 XML 相比，JSON 所需的字节数少了一半。它甚至比使用属性的 XML 文件还要小，如下面的输出所示：

```cs

已将 366 字节的 JSON 写入：/Users/markjprice/Code/Chapter09/ WorkingWithSerialization/people.json[{"FirstName":"Alice"，"LastName":"Smith"，"DateOfBirth":"1974-03-

14T00:00:00"，"Children":null}，{"FirstName":"Bob"，"LastName":"Jones"，"Date

OfBirth"："1969-11-23T00:00:00"，"Children"：null}，{"FirstName"："Charlie"，"L astName"："Cox"，"DateOfBirth"："1984-05-04T00:00:00"，"Children"：[{"FirstNam e"："Sally"，"LastName"："Cox"，"DateOfBirth"："2000-07-12T00:00:00"，"Children "：null}]}]

```

## 高性能 JSON 处理

.NET Core 3.0 引入了一个新的用于处理 JSON 的命名空间`System.Text.Json`，通过利用`Span<T>`等 API 进行了性能优化。

此外，像 Json.NET 这样的旧库是通过读取 UTF-16 来实现的。使用 UTF-8 来读写 JSON 文档会更高效，因为大多数网络协议，包括 HTTP，都使用 UTF-8，你可以避免将 UTF-8 转码为 Json.NET 的 Unicode `string`值。

使用新 API，微软在不同的场景中实现了 1.3 倍到 5 倍的性能提升。

Json.NET 的原始作者 James Newton-King 加入了微软，并一直与他们合作开发他们的新 JSON 类型。正如他在讨论新的 JSON API 时所说的那样，“Json.NET 不会消失”，如*图 9.3*所示：

![图形用户界面，文本，应用程序，电子邮件描述自动生成](img/Image00088.jpg)

图 9.3：Json.NET 的原始作者的评论

让我们看看如何使用新的 JSON API 来反序列化 JSON 文件：

1.  在`WorkingWithSerialization`项目的`Program.cs`中，导入新的 JSON 类以使用别名执行序列化，以避免与之前使用的 Json.NET 发生冲突，如下面的代码所示：

```cs

使用

NewJson = System.Text.Json.JsonSerializer;

```

1.  添加语句以打开 JSON 文件，对其进行反序列化，并输出人员的姓名和孩子的数量，如下面的代码所示：

```cs

使用

(FileStream jsonLoad = File.Open(jsonPath，FileMode.Open))

{

// 将对象图反序列化为 Person 列表

List<Person>? loadedPeople =

等待

NewJson.DeserializeAsync(utf8Json：jsonLoad，

returnType: typeof

(List<Person>)) as

List<Person>;

如果

(loadedPeople 是

不

空

）

{

foreach

(Person p in

loadedPeople)

{

WriteLine("{0}有{1}个孩子。"

，

p.LastName，p.Children?.Count ?? 0

）；

}

}

}

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

Smith 有 0 个孩子。

Jones 有 0 个孩子。

Cox 有 1 个孩子。

```

**良好实践**：选择 Json.NET 以提高开发人员的生产力和丰富的功能集，或者选择`System.Text.Json`以获得更好的性能。

# 控制 JSON 处理

有许多选项可以控制 JSON 的处理方式，如下面的列表所示：

+   包括和排除字段。

+   设置大小写策略。

+   选择大小写敏感策略。

+   选择紧凑和美化的空格。

让我们看看一些实际操作：

1.  使用您喜欢的代码编辑器向`Chapter09`解决方案/工作区添加一个名为`WorkingWithJson`的新控制台应用程序。

1.  在 Visual Studio Code 中，将`WorkingWithJson`选择为活动的 OmniSharp 项目。

1.  在`WorkingWithJson`项目的`Program.cs`中，删除现有的代码，导入处理 JSON 的两个主要命名空间，然后静态导入`System.Console`，`System.Environment`和`System.IO.Path`类型，如下面的代码所示：

```cs

使用

System.Text.Json; // JsonSerializer

使用

System.Text.Json.Serialization; // [JsonInclude]

使用

静态

System.Console;

使用

静态

System.Environment;

使用

静态

System.IO.Path;

```

1.  在`Program.cs`的底部，定义一个名为`Book`的类，如下面的代码所示：

```cs

公共

类

书

{

// 构造函数设置非空属性

公共

书

(

字符串

标题

）

{

Title = title;

}

// 属性

公共

字符串

Title { get

; 设置

; }

公共

字符串

？作者{获取

; set

; }

// 字段

[JsonInclude

] // 包括此字段

公共

DateOnly PublishDate;

[JsonInclude

] // 包括此字段

公共

DateTimeOffset Created;

公共

ushort

Pages;

}

```

1.  在`Book`类上方，添加语句以创建`Book`类的实例并将其序列化为 JSON，如下面的代码所示：

```cs

Book csharp10 = new

（标题：

"C# 10 and .NET 6 - Modern Cross-platform Development"

)

{

Author = "Mark J Price"

,

PublishDate = new

（年：2021

，月：11

，日：9

），

Pages = 823

,

Created = DateTimeOffset.UtcNow,

};

JsonSerializerOptions options = new

()

{

IncludeFields = true

, // 包括所有字段

PropertyNameCaseInsensitive = true

，

WriteIndented = true

,

PropertyNamingPolicy = JsonNamingPolicy.CamelCase,

};

字符串

filePath = Combine(CurrentDirectory, "book.json"

);

使用

（使用文件流= File.Create(filePath))

{

JsonSerializer.Serialize<Book>(

utf8Json：fileStream，value

：csharp10，选项）;

}

WriteLine("Written {0:N0} bytes of JSON to {1}"

,

arg0: new

FileInfo(filePath).Length,

arg1: filePath);

WriteLine();

// 显示序列化的对象图

WriteLine(File.ReadAllText(filePath));

```

1.  运行代码并查看结果，如下面的输出所示：

```cs

将 315 字节的 JSON 写入 C:\Code\Chapter09\WorkingWithJson\bin\Debug\net6.0\book.json

{

"title": "C# 10 and .NET 6 - Modern Cross-platform Development",

"author": "Mark J Price",

"publishDate": {

"year": 2021,

"month": 11,

"day": 9,

"dayOfWeek": 2,

"dayOfYear": 313,

"dayNumber": 738102

},

"created": "2021-08-20T08:07:02.3191648+00:00",

"pages": 823

}

```

请注意以下内容：

+   JSON 文件为 315 字节。

+   成员名称使用驼峰命名，例如`publishDate`。这对于在浏览器中使用 JavaScript 进行后续处理最佳。

+   由于设置的选项，所有字段都包括在内，包括`pages`。

+   JSON 经过美化以便于人类阅读。

+   `DateTimeOffset`值以单个标准字符串格式存储。

+   `DateOnly`值存储为具有日期部分的子属性的对象，例如`year`和`month`。

1.  在`Program.cs`中，设置`JsonSerializerOptions`时，注释掉设置大小写策略，写入缩进，并包括字段。

1.  运行代码并查看结果，如下面的输出所示：

```cs

将 230 字节的 JSON 写入 C:\Code\Chapter09\WorkingWithJson\bin\Debug\net6.0\book.json

{"Title":"C# 10 and .NET 6 - Modern Cross-platform Development","Author":"Mark J Price","PublishDate":{"Year":2021,"Month":11,"Day":9,"DayOfWeek":2,"DayOfYear":313,"DayNumber":738102},"Created":"2021-08-20T08:12:31.6852484+00:00"}

```

请注意以下内容：

+   JSON 文件为 230 字节，减少了 25%以上。

+   成员名称使用正常大小写，例如`PublishDate`。

+   `Pages`字段缺失。其他字段由于`PublishDate`和`Created`字段上的`[JsonInclude]`属性而包含在内。

+   JSON 紧凑，最小化空白以节省传输或存储的带宽。

## 用于处理 HTTP 响应的新 JSON 扩展方法

在.NET 5 中，微软对`System.Text.Json`命名空间中的类型进行了改进，例如为`HttpResponse`添加了扩展方法，您将在*第十六章*，*构建和使用 Web 服务*中看到。

## 从 Newtonsoft 迁移到新 JSON

如果您有现有代码使用 Newtonsoft Json.NET 库，并且想要迁移到新的`System.Text.Json`命名空间，那么微软有专门的文档，您可以在以下链接找到：

[`docs.microsoft.com/en-us/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to`](https://docs.microsoft.com/en-us/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to)

# 练习和探索

通过回答一些问题来测试您的知识和理解，进行一些实践，并通过更深入的研究来探索本章的主题。

## 练习 9.1-测试您的知识

回答以下问题：

1.  使用`File`类和`FileInfo`类有什么区别？

1.  流的`ReadByte`方法和`Read`方法之间有什么区别？

1.  何时使用`StringReader`，`TextReader`和`StreamReader`类？

1.  `DeflateStream`类型是做什么的？

1.  UTF-8 编码每个字符使用多少字节？

1.  什么是对象图？

1.  选择最佳序列化格式以减少空间要求是什么？

1.  选择最佳序列化格式以实现跨平台兼容性是什么？

1.  为什么使用`string`值如`"\Code\Chapter01"`来表示路径是不好的，你应该做什么？

1.  您在哪里可以找到有关 NuGet 软件包及其依赖关系的信息？

## 练习 9.2-练习将 XML 序列化

在`Chapter09`解决方案/工作区中，创建一个名为`Exercise02`的控制台应用程序，该应用程序创建一个形状列表，使用 XML 进行序列化并将其保存到文件系统，然后再次进行反序列化：

```cs

//创建一个要序列化的形状列表

List<Shape> listOfShapes = new

()

{

新

圆形{颜色="红色"

，半径=2.5

},

新

矩形{颜色="蓝色"

，高度=20.0

，宽度=10.0

},

新

圆形{颜色="绿色"

，半径=8.0

},

新

圆形{颜色="紫色"

，半径=12.3

},

新

矩形{颜色="蓝色"

，高度=45.0

，宽度=18.0

}

};

```

形状应该有一个名为`Area`的只读属性，这样当你反序列化时，你可以输出形状的列表，包括它们的面积，如下所示：

```cs

List<Shape> loadedShapesXml =

serializerXml.Deserialize(fileXml) as

List<Shape>;

foreach

形状项目中

loadedShapesXml)

{

WriteLine("{0}是{1}，面积为{2:N2}"

，

item.GetType().Name，item.Colour，item.Area);

}

```

当您运行控制台应用程序时，您的输出应该如下所示：

```cs

从 XML 加载形状：

圆形是红色的，面积为 19.63

矩形是蓝色的，面积为 200.00

圆形是绿色的，面积为 201.06

圆形是紫色的，面积为 475.29

矩形是蓝色的，面积为 810.00

```

## 练习 9.3-探索主题

使用以下页面上的链接，了解本章涵盖的主题的更多细节：

[`github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-9---working-with-files-streams-and-serialization`](https://github.com/markjprice/cs10dotnet6/blob/main/book-links.md#chapter-9---working-with-files-streams-and-serialization)

# 摘要

在本章中，您学会了如何从文本文件和 XML 文件中读取和写入，如何压缩和解压文件，如何编码和解码文本，以及如何将对象序列化为 JSON 和 XML（以及再次反序列化）。

在下一章中，您将学习如何使用 Entity Framework Core 处理数据库。
