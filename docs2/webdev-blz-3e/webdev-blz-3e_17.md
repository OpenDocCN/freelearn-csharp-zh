

# 检查源生成器

在本章中，我们将探讨编写生成代码的代码。尽管本章与 Blazor 开发没有直接关系，但我们将发现它仍然与 Blazor 有关。

源生成器是一个单独的主题，但我想介绍一下，因为它们被 Blazor 使用，而且坦白说，这是我最喜欢的功能之一。

我就是这样的人，如果我知道我需要反复重复这 10 分钟，我会花一整天的时间编写源代码来节省这 10 分钟。重复性任务从来不是我的最爱。

在本章中，我们将涵盖以下内容：

+   源生成器是什么

+   如何开始使用源生成器

+   社区项目

本章的想法是让你将其作为参考，以便你可以自己实现一个新项目。

# 技术要求

本章是一个参考章节，并且与本书的其他章节没有任何关联。

你可以在 [`github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter17`](https://github.com/PacktPublishing/Web-Development-with-Blazor-Third-Edition/tree/main/Chapter17) 找到本章结果的源代码。

# 源生成器是什么

在许多情况下，我们会发现自己反复编写相同类型的代码。在过去，我使用 T4 模板生成代码，甚至编写了 **存储过程** 和可以帮助我生成代码的应用程序。**源生成器** 是 .NET 编译器平台（Roslyn）SDK 的一部分。

生成器为我们提供了访问表示当前正在编译的所有用户代码的编译对象的权限。从那里，我们可以检查对象，并基于此编写额外的代码。

好吧，这听起来很复杂，如果我说编写源生成器很容易，那我就撒谎了，但它可以立即为我们节省大量时间。所以，让我们稍微分解一下。

当我们编译代码时，编译器会执行以下步骤：

1.  编译过程开始运行。

1.  源生成器分析代码。

1.  源生成器生成新的代码。

1.  编译过程继续。

*步骤 2* 和 *步骤 3* 是源生成器所做的事情。

在 Blazor 中，源生成器一直被使用；这是一个将 `.razor` 文件转换为 C# 代码的源生成器。

我们可以通过向我们的 `.csproj` 文件中添加以下内容来查看 Blazor 生成的代码：

```cs
<EmitCompilerGeneratedFiles>true</EmitCompilerGeneratedFiles> 
```

添加此代码将在 `obj` 文件夹中为 `razor` 组件生成输出文件。

我们可以在以下位置找到它们：`\obj\Debug\net8.0\generated\Microsoft.NET.Sdk.Razor.SourceGenerators\Microsoft.NET.Sdk.Razor.SourceGenerators.RazorSourceGenerator`。

我们可以通过使用以下方式选择文件的输出位置：

```cs
<CompilerGeneratedFilesOutputPath>THEPATH</CompilerGeneratedFilesOutputPath> 
```

你可以将 `THEPATH` 替换为你希望文件输出的路径。

在那个文件夹中，我们可以找到一个名为 `Pages_Counter_razor.g.cs` 的文件，它是计数器组件的 C# 表示形式。

`Microsoft.NET.Sdk.Razor.SourceGenerators-generator` 当然是一个非常高级的源生成器。

让我们考虑一个场景：在工作中，我们为服务创建服务和接口。这些接口的唯一用途是用于测试目的，就像我们在整本书中构建存储库的方式一样。

在这种情况下，向服务添加方法意味着我们需要将方法添加到类和接口中。我们试图通过将接口和类放在同一个文件中来简化这个过程。然而，我们仍然忘记了接口，提交了代码，直到构建完成并生成了 NuGet 包才注意到错误。

我们发现了一个名为`InterfaceGenerator`的源生成器；向我们的类添加一个属性将为我们生成接口。

让我们看看这个例子：

```cs
public class SampleService
{
    public double Multiply(double x, double y)
    {
        return x * y;
    }
    public int NiceNumber => 42;
} 
```

这是一个简单的服务类（来自`InterfaceGenerator`的 GitHub 页面）。向代码添加一个属性将自动生成一个接口，我们可以添加对该接口的引用：

```cs
[GenerateAutoInterface]
public class SampleService: ISampleService
… 
```

生成的接口将始终是最新的。这个示例是一个极好的例子，说明了源代码生成器如何节省时间和消除痛点。

源生成器很强大；我们可以访问一个语法树，我们可以查询它。我们可以遍历所有类，找到具有特定属性或实现接口的类，例如，然后基于这些信息生成代码。

存在一些限制。我们无法知道源生成器将按什么顺序运行，因此我们无法根据生成的代码生成代码。我们只能添加代码，而不能修改代码。

下一个部分将探讨我们如何构建我们的源生成器。

# 如何开始使用源生成器

是时候看看我们如何构建我们的源代码生成器了。`Chapter17`文件夹是我们讨论的完成示例。说明将不会是逐步指南。

要创建一个源代码生成器，我们需要一个针对*.NET Standard 2.0*的类库。我们还需要在该库中添加对 NuGet 包`Microsoft.CodeAnalysis.CSharp`和`Microsoft.CodeAnalysis.Analyzers`的引用。我们还需要确保我们的`.csproj`文件有`<LangVersion>latest</LangVersion>`。

要创建一个源代码生成器，我们需要创建一个具有两个功能的类：

+   它需要具有`[Generator]`属性。

+   它需要实现`ISourceGenerator`。

模板代码应该看起来像这样：

```cs
using Microsoft.CodeAnalysis;
namespace SourceGenerator;
[Generator]
public class HelloSourceGenerator : ISourceGenerator
{
    public void Execute(GeneratorExecutionContext context)
    {
        // Code generation goes here
    }
    public void Initialize(GeneratorInitializationContext context)
    {
        // No initialization required for this one
    }
} 
```

在`Initialize`方法中，我们添加可能需要的任何初始化；而在`Execute`方法中，我们编写生成的代码。

我们现在正在构建的生成器当然是一个愚蠢的例子，但它也展示了源生成器的一些强大功能。

在`Execute`方法中，我们添加以下代码：

```cs
 // Build up the source code
        string source = """
namespace BlazorWebAssemblyApp;
public class GeneratedService
    {
        public string GetHello()
        {
            return "Hello from generated code";
        }
    }
""";
        // Add the source code to the compilation
        context.AddSource($"GeneratedService.g.cs", source); 
```

它将源变量中的代码保存为 `GeneratedService.g.cs`。我们还在此文件中使用原始字符串字面量 – 这是 .NET7 中我最兴奋的功能。通过添加三个双引号，我们不需要转义字符串；我们可以在字符串内部自由添加更多双引号。如果您想转义超过三个双引号，您可以在开头和结尾添加更多。

要将源生成器添加到我们的项目中，我们可以像这样添加项目：

```cs
 <ItemGroup>
    <ProjectReference 
        Include="..\SourceGenerator\SourceGenerator.csproj" 
        OutputItemType="Analyzer"
        ReferenceOutputAssembly="false"/>
  </ItemGroup> 
```

当我们编译我们的项目时，`GeneratedService` 将被生成，我们可以使用这段代码。

现在，我们可以注入服务并在我们的组件中使用它：

```cs
@page "/"
@inject GeneratedService service
<h1>@service.GetHello()</h1> 
```

不要忘记将其添加到 `Program.cs` 中：

```cs
builder.Services.AddScoped<GeneratedService>(); 
```

上述示例并不是在现实场景中实际使用的方式，但我想要展示的是，开始使用它并不复杂。

有时 Visual Studio 编辑器不会识别这些生成的文件，我们会在代码编辑器中看到一些红色波浪线。这是因为源生成器的顺序（没有保证的顺序）会导致这些问题，尤其是在将源生成器与其他也生成的类（如 .`razor` 文件）结合使用时。

在下一节中，我们将探讨我们可以在项目中使用的源生成器。

# 社区项目

源生成器自 .NET5/6 以来一直存在，我们可以使用许多社区/开源项目在我们的项目中。让我们在接下来的章节中探索它们。

## InterfaceGenerator

我们已经讨论了 `InterfaceGenerator`。无需重复编写相同的内容即可生成接口，这将节省时间并帮助您避免问题，尤其是如果您只使用接口进行测试。

我们可以在这里找到它：

[`github.com/daver32/InterfaceGenerator`](https://github.com/daver32/InterfaceGenerator)

## Blazorators

David Pine，以及许多贡献者，构建了 Blazorators，它可以将 TypeScript 定义文件转换为可用于任何 Blazor 项目的 JavaScript 互操作代码。Blazorators 在编写 JavaScript 互操作时消除了许多痛点。

在这里查看他的项目：

[`github.com/IEvangelist/blazorators`](https://github.com/IEvangelist/blazorators)

## C# 源生成器

Amadeusz Sadowski，以及许多贡献者，制作了一个令人印象深刻的列表，列出了更多关于源生成器的信息以及一些杰出的项目。您可以在以下位置找到这个出色的资源：

[`github.com/amis92/csharp-source-generators`](https://github.com/amis92/csharp-source-generators)

## Roslyn SDK 示例

微软已经向他们的 Roslyn SDK 仓库添加了一些示例。这是一个深入了解源生成器的良好开端。您可以在以下位置找到这些示例：

[`github.com/dotnet/roslyn-sdk/tree/main/samples/CSharp/SourceGenerators`](https://github.com/dotnet/roslyn-sdk/tree/main/samples/CSharp/SourceGenerators)

## Microsoft Learn

Microsoft Learn 是学习任何与 C# 相关内容的绝佳资源，源生成器也不例外。

如果你像我一样认为源生成器是自切片面包以来最好的东西，我建议你深入研究 Microsoft Learn 上的文档：

[`learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview`](https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview)

# 摘要

在本章中，我们探讨了编写代码以节省时间和减少重复性任务的代码。

Blazor 使用源生成器将 razor 代码转换为 C# 代码，因此，间接地，我们一直在使用它们。

在下一章中，我们将通过访问 .NET MAUI 来了解 Blazor 混合模式。
