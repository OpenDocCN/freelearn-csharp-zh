# 第九章：测试和打包应用程序

在第八章中，*创建持续集成构建流程*，我们介绍了 Cake 自动化构建工具的安装和设置过程。此外，我们广泛演示了使用 Cake 编写构建脚本的过程，以及其丰富的 C#领域特定语言。我们还介绍了在 Visual Studio 中安装 Cake 扩展，并使用*任务资源管理器*窗口运行 Cake 脚本。

CI 流程为软件开发带来的好处不言而喻；它通过早期和快速检测，促进了项目代码库中错误的轻松修复。使用 CI，可以自动化运行和报告单元测试项目的测试覆盖率，以及项目构建和部署。

为了有效地利用 CI 流程的功能，代码库中的单元测试项目应该运行，并且应该由 CI 工具生成测试覆盖报告。在本章中，我们将修改 Cake 构建脚本，以运行我们的一系列 xUnit.net 测试。

在本章后面，我们将探讨.NET Core 版本控制以及它对应用程序开发的影响。最后，我们将为在.NET Core 支持的各种平台上分发的`LoanApplication`项目进行打包。之后，我们将探讨如何将.NET Core 应用程序打包以在 NuGet 上共享。

本章将涵盖以下主题：

+   使用 Cake 执行 xUnit.net 测试

+   .NET Core 版本控制

+   .NET Core 包和元包

+   用于 NuGet 分发的打包

# 使用 Cake 执行 xUnit.net 测试

在第八章中，*创建持续集成构建流程*，在*LoanApplication 构建脚本*部分，我们介绍了使用 Cake 自动化构建脚本创建和运行构建步骤的过程。使用 xUnit 控制台运行程序和 xUnit 适配器，可以更轻松地从 Visual Studio IDE、Visual Studio Code 或任何其他适合构建.NET 和.NET Core 应用程序的 IDE 中获取单元测试的测试结果和覆盖率。然而，为了使 CI 流程和构建流程完整和有效，单元测试项目应该作为构建步骤的一部分进行编译和执行。

# 在.NET 项目中执行 xUnit.net 测试

Cake 对运行 xUnit.net 测试有很好的支持。Cake 有两个别名，用于运行不同版本的 xUnit.net 测试——xUnit 用于运行早期版本的 xUnit.net，xUnit2 用于 xUnit.net 的版本 2。要使用别名的命令，必须在`XUnit2Settings`类中指定到 xUnit.net 的**ToolPath**，或者在`build.cake`文件中包含工具指令，以指示 Cake 从 NuGet 获取运行 xUnit.net 测试所需的二进制文件。

以下是包含 xUnit.net 工具指令的语法：

```cs
#tool "nuget:?package=xunit.runner.console"
```

Cake 的`XUnit2Alias`有不同形式的重载，用于运行指定程序集中的 xUnit.net 版本测试。该别名位于 Cake 的`Cake.Common.Tools.XUnit`命名空间中。第一种形式是`XUnit2(ICakeContext, IEnumerable<FilePath>)`，用于在`IEnumerable`参数中运行指定程序集中的所有 xUnit.net 测试。以下脚本显示了如何使用`GetFiles`方法将要执行的测试程序集获取到`IEnumerable`对象，并将其传递给`XUnit2`方法：

```cs
#tool "nuget:?package=xunit.runner.console"

Task("Execute-Test")  
    .Does(() =>
    {
        var assemblies = GetFiles("./LoanApplication.Tests.Unit/bin/Release/LoanApplication.Tests.Unit.dll");
        XUnit2(assemblies);    
    });
```

`XUnit2(ICakeContext, IEnumerable<FilePath>, XUnit2Settings)`别名类似于第一种形式，还增加了`XUnit2Settings`类，用于指定 Cake 应该如何执行 xUnit.net 测试的选项。以下代码片段描述了用法：

```cs
#tool "nuget:?package=xunit.runner.console"

Task("Execute-Test")  
    .Does(() =>
    {
        var assemblies = GetFiles("./LoanApplication.Tests.Unit/bin/Release/LoanApplication.Tests.Unit.dll");
        XUnit2(assemblies,
         new XUnit2Settings {
            Parallelism = ParallelismOption.All,
            HtmlReport = true,
            NoAppDomain = true,
            OutputDirectory = "./build"
        });
    });
```

另外，`XUnit2`别名允许传递字符串的`IEnumerable`，该字符串应包含要执行的 xUnit.net 版本 2 测试项目的程序集路径。形式为`XUnit2(ICakeContext, IEnumerable<string>)`，以下代码片段描述了用法：

```cs
#tool "nuget:?package=xunit.runner.console"

Task("Execute-Test")  
    .Does(() =>
    {
        XUnit2(new []{
        "./LoanApplication.Tests.Unit/bin/Release/LoanApplication.Tests.Unit.dll",
        "./LoanApplication.Tests/bin/Release/LoanApplication.Tests.dll"
    });
    });
```

# 在.NET Core 项目中执行 xUnit.net 测试

为了成功完成构建过程，重要的是在解决方案中运行测试项目，以验证代码是否正常工作。通过使用`DotNetCoreTest`别名，相对容易地在.NET Core 项目中运行 xUnit.net 测试，使用`dotnet test`命令。为了访问**dotnet-xunit**工具的其他功能，最好使用`DotNetCoreTool`运行测试。

在.NET Core 项目中，通过运行`dotnet test`命令来执行单元测试。该命令支持编写.NET Core 测试的所有主要单元测试框架，前提是该框架具有测试适配器，`dotnet test`命令可以集成以公开可用的单元测试功能。

使用 dotnet-xunit 框架工具运行.NET Core 测试可以访问 xUnit.net 中的功能和设置，并且是执行.NET Core 测试的首选方式。要开始，应该通过编辑`.csproj`文件并在`ItemGroup`部分包含`DotNetCliToolReference`条目，将 dotnet-xunit 工具安装到.NET Core 测试项目中。还应该添加`xunit.runner.visualstudio`和`Microsoft.NET.Test.Sdk`包，以便能够使用`dotnet test`或`dotnet xunit`命令执行测试：

```cs
<ItemGroup>
  <DotNetCliToolReference Include="dotnet-xunit" Version="2.3.1" />
  <PackageReference Include="xunit" Version="2.3.1" />
  PackageReference Include="xunit.runner.visualstudio" Version="2.3.1" />
</ItemGroup>
```

此外，还有其他参数可用于在使用`dotnet xunit`命令执行.NET Core 单元测试时自定义 xUnit.net 框架的行为。可以通过在终端上运行`dotnet xunit --help`命令来显示这些参数及其用法。

Cake 具有别名，可用于调用 dotnet SDK 命令来执行 xUnit.net 测试。`DotNetCoreRestore`别名使用`dotnet restore`命令还原解决方案中使用的 NuGet 包。此外，`DotNetCoreBuild`别名负责使用`dotnet build`命令构建.NET Core 解决方案。使用`DotNetCoreTest`别名执行测试项目中的单元测试，该别名使用`dotnet test`命令。请参见以下 Cake 片段，了解别名的用法。

```cs
var configuration = Argument("Configuration", "Release"); 

Task("Execute-Restore")  
    .Does(() =>
    {
        DotNetCoreRestore();
    });

Task("Execute-Build")  
    .IsDependentOn("Execute-Restore")
    .Does(() =>
    {
        DotNetCoreBuild("./LoanApplication.sln"
           new DotNetCoreBuildSettings()
                {
                    Configuration = configuration
                }
        );
    });

Task("Execute-Test")  
    .IsDependentOn("Execute-Build")
    .Does(() =>
    {
        var testProjects = GetFiles("./LoanApplication.Tests.Unit/*.csproj");
        foreach(var project in testProjects)
        {
            DotNetCoreTest(
                project.FullPath,
                new DotNetCoreTestSettings()
                {
                    Configuration = configuration,
                    NoBuild = true
                }
            );
        }
 });

```

另外，可以使用`DotNetCoreTool`别名来执行.NET Core 项目的 xUnit.net 测试。`DotNetCoreTool`是 Cake 中的通用别名，可用于执行任何 dotnet 工具。这是通过提供工具名称和必要的参数（如果有）来完成的。`DotNetCoreTool`公开了`dotnet xunit`命令中可用的其他功能，从而灵活地调整单元测试的执行方式。使用`DotNetCoreTool`别名时，需要手动将命令行参数传递给别名。请参见以下片段中别名的用法：

```cs
var configuration = Argument("Configuration", "Release");  

Task("Execute-Test")  
    .Does(() =>
    {
        var testProjects = GetFiles("./LoanApplication.Tests.Unit/*.csproj");
        foreach(var testProject in testProjects)
        {
            DotNetCoreTool(
                projectPath: testProject.FullPath, 
                command: "xunit", 
                arguments: $"-configuration {configuration} -diagnostics -stoponfail"
            );
        }
    });
```

# .NET Core 版本

对.NET Core SDK 和运行时进行版本控制使得平台易于理解，并且具有更好的灵活性。.NET Core 平台本质上是作为一个单元分发的，其中包括不同发行版的框架、工具、安装程序和 NuGet 包。此外，对.NET Core 平台进行版本控制可以在不同的.NET Core 平台上实现并行应用程序开发，具有很大的灵活性。

从.NET Core 2.0 开始，使用了易于理解的顶级版本号来对.NET Core 进行版本控制。一些.NET Core 版本组件一起进行版本控制，而另一些则不是。然而，从 2.0 版本开始，对.NET Core 发行版和组件采用了一致的版本控制策略，其中包括网页、安装程序和 NuGet 包。

.NET Core 使用的版本模型基于框架的运行时组件`[major].[minor]`版本号。与运行时版本号类似，SDK 版本使用带有额外独立`[patch]`的`[major].[minor]`版本号，该版本号结合了 SDK 的功能和补丁语义。

# 版本原则

截至.NET Core 2.0 版本，采用了以下原则：

+   将所有.NET Core 发行版版本化为*x.0.0*，例如第一个版本为 2.0.0，然后一起向前发展

+   文件和软件包名称应清楚地表示组件或集合及其版本，将版本分歧调和留给次要和主要版本边界

+   高阶版本和链接多个组件的安装程序之间应存在清晰的沟通。

此外，从.NET Core 2.0 开始，共享框架和相关运行时、.NET Core SDK 和相关.NET Core CLI 以及`Microsoft.NETCore.App`元包的版本号被统一了。使用单个版本号可以更容易地确定在开发机器上安装的 SDK 版本以及在将应用程序移动到生产环境时应该使用的共享框架版本。

# 安装程序

每日构建和发布的下载符合新的命名方案。从.NET Core 2.0 开始，下载中提供的安装程序 UI 也已修改，以显示正在安装的组件的名称和版本。命名方案格式如下：

```cs
[product]-[component]-[major].[minor].[patch]-[previewN]-[optional build #]-[rid].[file ext]
```

此外，格式详细显示了正在下载的内容，其版本，可以在哪种操作系统上使用，以及它是否可读。请参见下面显示的格式示例：

```cs
dotnet-runtime-2.0.7-osx-x64.pkg                    # macOS runtime installer
dotnet-runtime-2.0.7-win-x64.exe                    # Windows SDK installer
```

安装程序中包含的网站和 UI 字符串的描述保持简单、准确和一致。有时，SDK 版本可能包含多个运行时版本。在这种情况下，当安装过程完成时，安装程序 UX 仅在摘要页面上显示 SDK 版本和已安装的运行时版本。这适用于 Windows 和 macOS 的安装程序。

此外，可能需要更新.NET Core 工具，而不一定需要更新运行时。在这种情况下，SDK 版本会增加，例如到 2.1.2。下次更新时，运行时版本将增加，例如，下次更新时，运行时和 SDK 都将作为 2.1.3 进行发布。

# 软件包管理器

.NET Core 平台的灵活性使得分发不仅仅由微软完成；其他实体也可以分发该平台。该平台的灵活性使得为 Linux 发行版所有者分发安装程序和软件包变得容易。同时，也使得软件包维护者可以轻松地将.NET Core 软件包添加到其软件包管理器中。

最小软件包集的详细信息包括`dotnet-runtime-[major].[minor]`，这是具有特定 major+minor 版本组合的.NET 运行时，并且在软件包管理器中可用。`dotnet-sdk`包括前向 major、minor、patch 版本以及更新卷。软件包集中还包括`dotnet-sdk-[major].[minor]`，这是具有最高指定版本的共享框架和最新主机的 SDK，即`dotnet-host`。

# Docker

与安装程序和软件包管理器类似，docker 标签采用命名约定，其中版本号放在组件名称之前。可用的 docker 标签包括以下运行时版本：

+   `1.0.8-runtime`

+   `1.0.8-sdk`

+   `2.0.4-runtime`

+   `2.0.4-sdk`

+   `2.1.1-runtime`

+   `2.1.1-sdk`

当包含在 SDK 中的.NET Core CLI 工具被修复并重新发布时，SDK 版本会增加，例如，当版本从 2.1.1 增加到版本 2.1.2。此外，重要的是要注意，SDK 标签已更新以表示 SDK 版本而不是运行时。基于此，运行时将在下次发布时赶上 SDK 版本编号，例如，下次发布时，SDK 和运行时将都采用版本号 2.1.3。

# 语义版本控制

.NET Core 使用语义版本控制来描述.NET Core 版本中发生的更改的类型和程度。**语义版本控制**（**SemVer**）使用`MAJOR.MINOR.PATCH`版本模式：

```cs
MAJOR.MINOR.PATCH[-PRERELEASE-BUILDNUMBER]
```

SemVer 的`PRERELEASE`和`BUILDNUMBER`部分是可选的，不是受支持的版本的一部分。它们专门用于夜间构建、从源目标进行本地构建和不受支持的预览版本。

当旧版本不再受支持时，采用现有依赖项的较新`MAJOR`版本，或者切换兼容性怪癖的设置时，将递增版本的`MAJOR`部分。每当现有依赖项有较新的`MINOR`版本，或者有新的依赖项、公共 API 表面积或新行为添加时，将递增`MINOR`。每当现有依赖项有较新的`PATCH`版本、对较新平台的支持或有错误修复时，将递增`PATCH`。

当`MAJOR`被递增时，`MINOR`和`PATCH`被重置为零。同样，当`MINOR`被递增时，`PATCH`被重置为零，而`MAJOR`不受影响。这意味着每当有多个更改时，受影响的最高元素会被递增，而其他部分会被重置为零。

通常，预览版本的版本会附加`-preview[number]-([build]|"final")`，例如，2.1.1-preview1-final。开发人员可以根据.NET Core 的两种可用发布类型**长期支持**（**LTS**）和**当前**，选择所需的功能和稳定级别。

LTS 版本是一个相对更稳定的平台，支持时间更长，而新功能添加得更少。当前版本更频繁地添加新功能和 API，但允许安装更新的时间较短，提供更频繁的更新，并且支持时间比 LTS 更短。

# .NET Core 软件包和 metapackages

.NET Core 平台是作为一组通常称为 metapackages 的软件包进行发布的。该平台基本上由 NuGet 软件包组成，这有助于使其轻量级且易于分发。.NET Core 中的软件包提供了平台上可用的原语和更高级别的数据类型和常用实用程序。此外，每个软件包直接映射到一个具有相同名称的程序集；`System.IO.FileSystem.dll`程序集是`System.IO.FileSystem`软件包。

.NET Core 中的软件包被定义为细粒度。这带来了巨大的好处，因为在该平台上开发的应用程序的结果是印刷小，只包含在项目中引用和使用的软件包。未引用的软件包不会作为应用程序分发的一部分进行发布。此外，细粒度软件包可以提供不同的操作系统和 CPU 支持，以及仅适用于一个库的特定依赖关系。.NET Core 软件包通常与平台支持一起发布。这允许修复作为轻量级软件包更新进行分发和安装。

以下是.NET Core 可用的一些 NuGet 软件包：

+   `System.Runtime`：这是.NET Core 软件包，包括`Object`、`String`、`Array`、`Action`和`IList<T>`。

+   `System.Reflection`：此软件包包含用于加载、检查和激活类型的类型，包括`Assembly`、`TypeInfo`和`MethodInfo`。

+   `System.Linq`：用于查询对象的一组类型，包括`Enumerable`和`ILookup<TKey,TElement>`。

+   `System.Collections`：用于通用集合的类型，包括`List<T>`和`Dictionary<TKey,TValue>`。

+   `System.Net.Http`：用于 HTTP 网络通信的类型，包括`HttpClient`和`HttpResponseMessage`。

+   `System.IO.FileSystem`：用于读取和写入本地或网络磁盘存储的类型，包括**文件**和**目录**。

在您的.Net Core 项目中引用软件包相对容易。例如，如果您在项目中包含`System.Reflection`，则可以在项目中引用它，如下所示：

```cs
<Project Sdk="Microsoft.NET.Sdk">
<PropertyGroup>
<TargetFramework>netstandard2.0</TargetFramework>
</PropertyGroup>
<ItemGroup>
<PackageReference Include="System.Reflection" Version="4.3.0" />
</ItemGroup>
</Project> 
```

# Metapackage

**元包**是除了项目中已引用的目标框架之外，添加到.NET Core 项目中的引用或依赖关系。例如，您可以将`Microsoft.NETCore.App`或`NetStandard.Library`添加到.NET Core 项目中。

有时，需要在项目中使用一组包。这是通过使用元包来完成的。元包是经常一起使用的一组包。此外，元包是描述一组或一套包的 NuGet 包。当指定框架时，元包可以为这些包创建一个框架。

当您引用一个元包时，实质上是引用了元包中包含的所有包。实质上，这使得这些包中的库在使用 Visual Studio 进行项目开发时可以进行智能感知。此外，这些库在项目发布时也将可用。

在.NET Core 项目中，元包是由项目中的目标框架引用的，这意味着元包与特定框架强烈关联或绑定在一起。元包可以访问已经确认和测试过可以一起工作的一组包。

.NET Standard 元包是`NETStandard.Library`，它构成了.NET 标准中的一组库。这适用于.NET 平台的不同变体：.NET Core、.NET Framework 和 Mono 框架。

`Microsoft.NETCore.App`和`Microsoft.NETCore.Portable.Compatibility`是主要的.NET Core 元包。`Microsoft.NETCore.App`描述了构成.NET Core 分发的库集，并依赖于`NETStandard.Library`。

`Microsoft.NETCore.Portable.Compatibility`描述了一组 facade，使得基于 mscorlib 的**可移植类库**（**PCLs**）可以在.NET Core 上工作。

# Microsoft.AspNetCore.All 元包

`Microsoft.AspNetCore.All`是 ASP.NET Core 的元包。该元包包括由 ASP.NET Core 团队支持和维护的包，Entity Framework Core 支持的包，以及 ASP.NET Core 和 Entity Framework Core 都使用的内部和第三方依赖项。

针对 ASP.NET Core 2.0 的可用默认项目模板使用`Microsoft.AspNetCore.All`包。ASP.NET Core 版本号和 Entity Framework Core 版本号与`Microsoft.AspNetCore.All`元包的版本号相似。ASP.NET Core 2.x 和 Entity Framework Core 2.x 中的所有可用功能都包含在`Microsoft.AspNetCore.All`包中。

当您创建一个引用`Microsoft.AspNetCore.All`元包的 ASP.NET Core 应用程序时，.NET Core Runtime Store 将可供您使用。.NET Core Runtime Store 公开了运行 ASP.NET Core 2.x 应用程序所需的运行时资源。

在部署过程中，引用的 ASP.NET Core NuGet 包中的资源不会与应用程序一起部署，这些资源位于.NET Core Runtime Store 中。这些资源经过预编译以提高性能，加快应用程序启动时间。此外，排除未使用的包是可取的。这是通过使用包修剪过程来完成的。

要使用`Microsoft.AspNetCore.All`包，应将其添加为.NET Core 的`.csproj`项目文件的引用，就像以下 XML 配置中所示：

```cs
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp2.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.0" />
  </ItemGroup>

</Project>
```

# NuGet 分发的打包

.NET Core 的灵活性不仅限于应用程序的开发，还延伸到部署过程。部署.NET Core 应用程序可以采用两种形式——**基于框架的部署**（**FDD**）和**独立部署**（**SCD**）。

使用 FDD 方法需要在开发应用程序的计算机上安装系统范围的.NET Core。安装的.NET Core 运行时将被应用程序和在该计算机上部署的其他应用程序共享。

这使得应用程序可以在不同版本或安装的 .NET Core 框架之间轻松移植。此外，使用此方法时，部署将是轻量级的，只包含应用程序的代码和使用的第三方库。使用此方法时，为应用程序创建了 `.dll` 文件，以便可以从命令行启动。

SCD 允许您将应用程序与运行所需的 .NET Core 库和 .NET Core 运行时一起打包。实质上，您的应用程序不依赖于部署计算机上已安装的 .NET Core 的存在。

使用此方法时，可执行文件（本质上是平台特定的 .NET Core 主机的重命名版本）将作为应用程序的一部分打包。在 Windows 上，此可执行文件为 `app.exe`，在 Linux 和 macOS 上为 `app`。与使用 *依赖于框架的方法* 部署应用程序时一样，为应用程序创建了 `.dll` 文件，以便启动应用程序。

# dotnet publish 命令

`dotnet publish` 命令用于编译应用程序，并在将应用程序和依赖项复制到准备部署的文件夹之前检查应用程序的依赖项。执行该命令是准备 .NET Core 应用程序进行部署的唯一官方支持的方式。概要在此处：

```cs
dotnet publish [<PROJECT>] [-c|--configuration] [-f|--framework] [--force] [--manifest] [--no-dependencies] [--no-restore] [-o|--output] [-r|--runtime] [--self-contained] [-v|--verbosity] [--version-suffix]

dotnet publish [-h|--help]
```

运行命令时，输出将包含 `.dll` 程序集中包含的**中间语言**（**IL**）代码，包含项目依赖项的 `.deps.json` 文件，指定预期共享运行时的 `.runtime.config.json` 文件，以及从 NuGet 缓存中复制到输出文件夹中的应用程序依赖项。

命令的参数和选项在此处解释：

+   `PROJECT`：用于指定要编译和发布的项目，默认为当前文件夹。

+   -c|--configuration：用于指定构建配置的选项，可取 `Debug` 和 `Release` 值，默认值为 `Debug`。

+   -f|--framework <FRAMEWORK>：目标框架选项，与命令一起指定时，将为目标框架发布应用程序。

+   --force：用于强制解析依赖项，类似于删除 `project.assets.json` 文件。

+   -h|--help：显示命令的帮助信息。

+   --manifest <PATH_TO_MANIFEST_FILE>：用于指定要在修剪应用程序发布的软件包时使用的一个或多个目标清单。

+   --no-dependencies：此选项用于忽略项目对项目的引用，但会还原根项目。

+   --no-restore：指示命令不执行隐式还原。

+   -o|--output <OUTPUT_DIRECTORY>：用于指定输出目录的路径。如果未指定该选项，则默认为 FDD 的 `./bin/[configuration]/[framework]/` 或 SCD 的 `./bin/[configuration]/[framework]/[runtime]`。

+   -r|--runtime <RUNTIME_IDENTIFIER>：用于为特定运行时发布应用程序，仅在创建 SCD 时使用。

+   --self-contained：用于指定 SCD。当指定运行时标识符时，默认值为 true。

+   -v|--verbosity <LEVEL>：用于指定 `dotnet publish` 命令的详细程度。允许的值为 `q[uiet]`、`n[ormal]`、`m[inimal]`、`diag[nostic]` 和 `d[etailed]`。

+   --version-suffix <VERSION_SUFFIX>：用于指定在项目文件的版本字段中替换星号 (`*`) 时要使用的版本后缀。

命令的使用示例是在命令行上运行 `dotnet publish`。这将发布当前文件夹中的项目。要发布本书中使用的 `LoanApplication` 项目，可以运行 `dotnet publish` 命令。这将使用项目中指定的框架发布应用程序。ASP.NET Core 应用程序依赖的解决方案中的项目将与之一起构建。请参阅以下屏幕截图：

![](img/6e3b3ee4-f49d-48dd-aefc-4d7c53f9f986.png)

在`netcoreapp2.0`文件夹中创建了一个`publish`文件夹，其中将复制所有编译文件和依赖项：

![](img/71dacc7f-c199-41f2-a5bb-a3985c070d53.png)

# 创建一个 NuGet 软件包

**NuGet**是.NET 的软件包管理器，它是一个开源的软件包管理器，为构建在.NET Framework 和.NET Core 平台上的应用程序提供了更简单的版本控制和分发库的方式。NuGet 库是.NET 的中央软件包存储库，用于托管包作者和消费者使用的所有软件包。

使用.NET Core 的`dotnet pack`命令可以轻松创建 NuGet 软件包。运行此命令时，它会构建.NET Core 项目，并从中创建一个 NuGet 软件包。打包的.NET Core 项目的 NuGet 依赖项将被添加到`.nuspec`文件中，以确保在安装软件包时它们被解析。显示以下命令概要：

```cs
dotnet pack [<PROJECT>] [-c|--configuration] [--force] [--include-source] [--include-symbols] [--no-build] [--no-dependencies]
 [--no-restore] [-o|--output] [--runtime] [-s|--serviceable] [-v|--verbosity] [--version-suffix]

dotnet pack [-h|--help]
```

这里解释了命令的参数和选项：

+   `PROJECT`用于指定要打包的项目，可以是目录的路径或`.csproj`文件。默认为当前文件夹。

+   `c|--configuration`：此选项用于定义构建配置。它接受`Debug`和`Release`值。默认值为`Debug`。

+   `--force`：用于强制解析依赖项，类似于删除`project.assets.json`文件。

+   `-h|--help`：显示命令的帮助信息。

+   `-include-source`：用于指定源文件包含在 NuGet 软件包的`src`文件夹中。

+   `--include-symbols`：生成`nupkg`符号。

+   `--no-build`：这是为了指示命令在打包之前不要构建项目。

+   `--no-dependencies`：此选项用于忽略项目对项目的引用，但恢复根项目。

+   `--no-restore`：这是为了指示命令不执行隐式还原。

+   `-o|--output <OUTPUT_DIRECTORY>`：用于指定输出目录的路径，以放置构建的软件包。

+   `-r|--runtime <RUNTIME_IDENTIFIER>`：此选项用于指定要为其还原软件包的目标运行时。

+   `-s|--serviceable`：用于在软件包中设置可服务标志。

+   `-v|--verbosity <LEVEL>`：用于指定命令的详细程度。允许的值为`q[uiet]`、`m[inimal]`、`n[ormal]`、`d[etailed]`和`diag[nostic]`。

+   `--version-suffix <VERSION_SUFFIX>`：用于指定在项目文件的版本字段中替换星号(`*`)时要使用的版本后缀。

运行`dotnet pack`命令将打包当前目录中的项目。要打包`LoanApplication.Core`项目，可以运行以下命令：

```cs
dotnet pack C:\LoanApplication\LoanApplication.Core\LoanApplication.Core.csproj --output nupkgs
```

运行该命令时，`LoanApplication.Core`项目将被构建并打包到项目文件夹中的`nupkgs`文件中。将创建`LoanApplication.Core.1.0.0.nupkg`文件，其中包含打包项目的库：

![](img/15b74cee-dd94-4076-b5dc-faa7f6040573.png)

应用程序打包后，可以使用`dotnet nuget push`命令将其发布到 NuGet 库。为了能够将软件包推送到 NuGet，您需要注册 NuGet API 密钥。在上传软件包到 NuGet 时，这些密钥需要作为`dotnet nuget push`命令的选项进行指定。

运行`dotnet nuget push LoanApplication.Core.1.0.0.nupkg -k <api-key> -s https://www.nuget.org/`命令将创建的 NuGet 软件包推送到库中，从而使其他开发人员可以使用。运行该命令时，将建立到 NuGet 服务器的连接，以在您的帐户下推送软件包：

![](img/601521bb-89f1-45a2-bf67-f262cb128c78.png)

将软件包推送到 NuGet 库后，登录您的帐户，您可以在已发布软件包的列表中找到推送的软件包：

![](img/680b6ae6-3f27-4c89-af9b-224ae44623d1.png)

当您将软件包上传到 NuGet 库时，其他程序员可以直接从 Visual Studio 使用 NuGet 软件包管理器搜索您的软件包，并在其项目中添加对库的引用。

# 总结

在本章中，我们首先使用 Cake 执行了 xUnit.net 测试。此外，我们广泛讨论了.NET Core 的版本控制、概念以及它对.NET Core 平台应用开发的影响。之后，我们为 NuGet 分发打包了本书中使用的`LoanApplication`项目。

在本书中，您已经经历了一次激动人心的 TDD 之旅。使用 xUnit.net 单元测试框架，TDD 的概念被介绍并进行了广泛讨论。还涵盖了数据驱动的单元测试，这使您能够使用不同数据源的数据来测试您的代码。

Moq 框架被用来介绍和解释如何对具有依赖关系的代码进行单元测试。TeamCity CI 服务器被用来解释 CI 的概念。Cake，一个跨平台构建系统被探讨并用于创建在 TeamCity 中执行的构建步骤。此外，另一个 CI 工具 Microsoft VSTS 被用来执行 Cake 脚本。

最后，有效地使用 TDD 在代码质量和最终应用方面是非常有益的。通过持续的实践，本书中解释的所有概念都可以成为您日常编程例行的一部分。
