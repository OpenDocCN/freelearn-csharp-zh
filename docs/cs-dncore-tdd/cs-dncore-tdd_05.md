# 第五章：数据驱动单元测试

在上一章中，我们讨论了良好单元测试的属性，以及 xUnit.net 支持的两种测试类型**Fact**和**Theory**。此外，我们还通过 xUnit.net 单元测试框架中可用的丰富测试断言集合创建了单元测试。

为软件项目编写的单元测试应该从开发阶段开始反复运行，在部署期间，维护期间，以及在项目的整个生命周期中都应该有效地运行。通常情况下，这些测试应该在不同的数据输入上运行相同的执行步骤，而测试和被测试的代码都应该在不同的数据输入下表现出一致的行为。

通过使用不同的数据集运行测试可以通过创建或复制具有相似步骤的现有测试来实现。这种方法的问题在于维护，因为必须在各种复制的测试中影响测试逻辑的更改。xUnit.net 通过其数据驱动单元测试功能解决了这一挑战，称为**theories**，它允许在不同的测试数据集上运行测试。

数据驱动单元测试，也可以称为 xUnit.net 中的数据驱动测试自动化，是用`Theory`属性装饰的测试，并将数据作为参数传递给这些测试。传递给数据驱动单元测试的数据可以来自各种来源，可以通过使用`InlineData`属性进行内联。数据也可以来自特定的数据源，例如从平面文件、Web 服务或数据库中获取数据。

在第四章中解释的示例数据驱动单元测试使用了内联方法。还有其他属性可以用于向测试提供数据，如`MemberData`和`ClassData`。

在本章中，我们将通过使用 xUnit.net 框架创建数据驱动单元测试，并涵盖以下主题：

+   数据驱动单元测试的好处

+   用于创建数据驱动测试的 xUnit.net `Theory`属性

+   内联数据驱动单元测试

+   属性数据驱动单元测试

+   整合来自其他来源的数据

# 数据驱动单元测试的好处

**数据驱动单元测试**是一个概念，因为它能够使用不同的数据集执行测试，所以它能够对代码行为提供深入的见解。通过数据驱动单元测试获得的见解可以帮助我们对应用程序开发方法做出明智的决策，并且可以识别出需要改进的潜在领域。可以从数据单元测试的报告和代码覆盖率中制定策略，这些策略可以后来用于重构具有潜在性能问题和应用程序逻辑中的错误的代码。

数据驱动单元测试的一些好处在以下部分进行了解释。

# 测试简洁性

通过数据驱动测试，可以更容易地减少冗余，同时保持全面的测试覆盖。这是因为可以避免测试代码的重复。传统上需要为不同数据集重复测试的测试现在可以用于不同的数据集。当存在具有相似结构但具有不同数据的测试时，这表明可以将这些测试重构为数据驱动测试。

让我们在以下片段中回顾`CarLoanCalculator`类和相应的`LoanCalculatorTest`测试类。与传统的编写测试方法相比，这将为我们提供宝贵的见解，说明为什么数据驱动测试可以简化测试，同时在编写代码时提供简洁性。

`CarLoanCalculator`扩展了`LoanCalculator`类，覆盖了`CalculateLoan`方法，执行与汽车贷款相关的计算，并返回一个`Loan`对象，该对象将使用 xUnit.net 断言进行验证：

```cs
public class CarLoanCalculator : LoanCalculator
{    
    public CarLoanCalculator(RateParser rateParser)
    {
        base.rateParser=rateParser;
    }

    public override Loan CalculateLoan(LoanDTO loanDTO)
    {
        Loan loan = new Loan();
        loan.LoanType=loanDTO.LoanType;
        loan.InterestRate=rateParser.GetRateByLoanType(loanDTO.LoanType, loanDTO.LocationType, loanDTO.JobType);
        // do other processing
        return loan    
    }   
}
```

为了验证`CarLoanCalculator`类的一致行为，将使用以下测试场景验证`CalculateLoan`方法返回的`Loan`对象，当方法参数`LoanDTO`具有不同的`LoanType`、`LocationType`和`JobType`组合时。`CarLoanCalculatorTest`类中的`Test_CalculateLoan_ShouldReturnLoan`测试方法验证了描述的每个场景：

```cs
public class CarLoanCalculatorTest
{    
    private CarLoanCalculator carLoanCalculator;

    public CarLoanCalculatorTest()
    {
        RateParser rateParser= new RateParser();
        this.carLoanCalculator=new CarLoanCalculator(rateParser);
    }

    [Fact]
    public void Test_CalculateLoan_ShouldReturnLoan()
    {
        // first scenario
        LoanDTO loanDTO1 = new LoanDTO();
        loanDTO1.LoanType=LoanType.CarLoan;
        loanDTO1.LocationType=LocationType.Location1;
        loanDTO1.JobType=JobType.Professional
        Loan loan1=carLoanCalculator.CalculateLoan(loanDTO1);

        Assert.NotNull(loan1);
        Assert.Equal(8,loan1.InterestRate);        

        // second scenario
        LoanDTO loanDTO2 = new LoanDTO();
        loanDTO2.LoanType=LoanType.CarLoan;
        loanDTO2.LocationType=LocationType.Location2;
        loanDTO2.JobType=JobType.Professional;
        Loan loan2=carLoanCalculator.CalculateLoan(loanDTO2);

        Assert.NotNull(loan2);
        Assert.Equal(10,loan2.InterestRate);
    }   
}
```

在上述片段中的`Test_CalculateLoan_ShouldReturnLoan`方法包含了用于测试`CalculateLoan`方法两次的代码行。这个测试明显包含了重复的代码，测试与测试数据紧密耦合。此外，测试代码不够清晰，因为当添加更多的测试场景时，测试方法将不得不通过添加更多的代码行来进行修改，从而使测试变得庞大而笨拙。通过数据驱动测试，可以避免这种情况，并且可以消除测试中的重复代码。

# 包容性测试

当业务人员和质量保证测试人员参与自动化测试过程时，可以改善软件应用程序的质量。他们可以使用数据文件作为数据源，无需太多的技术知识，就可以向数据源中填充执行测试所需的数据。可以使用不同的数据集多次运行测试，以彻底测试代码，以确保其健壮性。

使用数据驱动测试，您可以清晰地分离测试和数据。原本可能会与数据混在一起的测试现在将使用适当的逻辑进行分离。这确保了数据源可以在不更改使用它们的测试的情况下进行修改。

通过数据驱动单元测试，应用程序的整体质量得到改善，因为您可以使用各种数据集获得良好的覆盖率，并具有用于微调和优化正在开发的应用程序以获得改进性能的指标。

# xUnit.net 理论属性用于创建数据驱动测试

在 xUnit.net 中，数据驱动测试被称为理论。它们是使用`Theory`属性装饰的测试。当测试方法使用`Theory`属性装饰时，必须另外使用数据属性装饰，测试运行器将使用该属性确定要在执行测试时使用的数据源：

```cs
[Theory]
public void Test_CalculateRates_ShouldReturnRate()
{
   // test not implemented yet
}
```

当测试标记为数据理论时，从数据源中提供的数据直接映射到测试方法的参数。与使用`Fact`属性装饰的常规测试不同，数据理论的执行次数基于从数据源获取的可用数据行数。

至少需要传递一个数据属性作为测试方法参数，以便 xUnit.net 将测试视为数据驱动并成功执行。要传递给测试的数据属性可以是`InlineData`、`MemberData`和`ClassData`中的任何一个。这些数据属性源自`Xunit.sdk.DataAttribute`。

# 内联数据驱动单元测试

**内联数据驱动测试**是使用*xUnit.net 框架*编写数据驱动测试的最基本或最简单的方式。内联数据驱动测试使用`InlineData`属性编写，该属性用于装饰测试方法，除了`Theory`属性之外：

```cs
[Theory, InlineData("arguments")]
```

当测试方法需要简单的参数并且不接受类实例化作为`InlineData`参数时，可以使用内联数据驱动测试。使用内联数据驱动测试的主要缺点是缺乏灵活性。不能将内联数据与另一个测试重复使用。

当在数据理论中使用`InlineData`属性时，数据行是硬编码的，并内联传递到测试方法中。要用于测试的所需数据可以是任何数据类型，并作为参数传递到`InlineData`属性中：

```cs
public class TheoryTest
{
    [Theory,
    InlineData("name")]
    public void TestCheckWordLength_ShouldReturnBoolean(string word)
    {
        Assert.Equal(4, word.Length);
    }
}
```

内联数据驱动测试可以有多个`InlineData`属性，指定测试方法的参数。多个`InlineData`数据理论的语法在以下代码中指定：

```cs
[Theory, InlineData("argument1"), InlineData("argument2"), InlineData("argumentn")]
```

`TestCheckWordLength_ShouldReturnBoolean`方法可以更改为具有三个内联数据行，并且可以根据需要添加更多数据行到测试中。为了保持测试的清晰，建议每个测试不要超过必要或所需的内联数据：

```cs
public class TheoryTest
{
    [Theory,
    InlineData("name"),
    InlineData("word"),
    InlineData("city")
    ]
    public void TestCheckWordLength_ShouldReturnBoolean(string word)
    {
        Assert.Equal(4, word.Length);
    }
}
```

在编写内联数据驱动单元测试时，必须确保测试方法中的参数数量与传递给`InlineData`属性的数据行中的参数数量匹配；否则，xUnit 测试运行器将抛出`System.InvalidOperationException`。以下代码片段中`TestCheckWordLength_ShouldReturnBoolean`方法中的`InlineData`属性已被修改为接受两个参数：

```cs
public class TheoryTest
{
    [Theory,
    InlineData("word","name")]
    public void TestCheckWordLength_ShouldReturnBoolean(string word)
    {
        Assert.Equal(4, word.Length);
    }
}
```

当您在前面的代码片段中运行数据理论测试时，xUnit 测试运行器会因为传递了两个参数`"word"`和`"name"`给 InlineData 属性，而不是预期的一个参数，导致测试失败并显示`InvalidOperationException`，如下面的屏幕截图所示：

![](img/23ebb8d6-1117-4d33-ab01-019ba6d8ba69.png)

当运行内联数据驱动测试时，xUnit.net 将根据添加到测试方法的`InlineData`属性或数据行的数量创建测试的数量。在以下代码片段中，xUnit.net 将创建两个测试，一个用于`InlineData`属性的参数`"name"`，另一个用于参数`"city"`：

```cs
 [Theory,
    InlineData("name"),
    InlineData("city")]
    public void TestCheckWordLength_ShouldReturnBoolean(string word)
    {
        Assert.Equal(4, word.Length);
    }
```

如果您在 Visual Studio 中使用测试运行器运行`TestCheckWordLength_ShouldReturnBoolean`测试方法，测试应该成功运行并通过。基于属性创建的两个测试可以通过从`InlineData`属性传递给它们的参数来区分：

![](img/134d525d-7134-4729-85a3-735ad3552326.png)

现在，让我们修改*数据驱动单元测试的好处*部分中的`Test_CalculateLoan_ShouldReturnCorrectRate`测试方法，使用`InlineData`来加载测试数据，而不是直接在测试方法的代码中硬编码测试数据：

```cs
[Theory,InlineData(new LoanDTO{ LoanType =LoanType.CarLoan, JobType =JobType.Professional, LocationType=LocationType.Location1 })]
 public void Test_CalculateLoan_ShouldReturnCorrectRate(LoanDTO loanDTO)
 {
     Loan loan = carLoanCalculator.CalculateLoan(loanDTO);
     Assert.NotNull(loan);
     Assert.Equal(8, loan.InterestRate);
 }
```

在 Visual Studio 中，上述代码片段将导致语法错误，IntelliSense 上下文菜单显示错误——属性参数必须是常量表达式、表达式类型或属性参数类型的数组创建表达式：

![](img/cbdb20bd-06a9-498b-be91-9f33f13b6bbc.png)

在`InlineData`属性中使用属性或自定义类型作为参数类型是不允许的，这表明`LoanDTO`类的新实例不能作为`InlineData`属性的参数。这是`InlineData`属性的限制，因为它不能用于从属性、类、方法或自定义类型加载数据。

# 属性数据驱动单元测试

在编写内联数据驱动测试时遇到的灵活性不足可以通过使用属性数据驱动测试来克服。属性数据驱动单元测试是通过使用`MemberData`和`ClassData`属性在 xUnit.net 中编写的。使用这两个属性，可以创建从不同数据源（如文件或数据库）加载数据的数据理论。

# MemberData 属性

当要创建并加载来自以下数据源的数据行的数据理论时，使用`MemberData`属性：

+   静态属性

+   静态字段

+   静态方法

在使用`MemberData`时，数据源必须返回与`IEnumerable<object[]>`兼容的独立对象集。这是因为在执行测试方法之前，`return`属性会被`.ToList()`方法枚举。

`Test_CalculateLoan_ShouldReturnCorrectRate`测试方法在*数据驱动单元测试的好处*部分中，可以重构以使用`MemberData`属性来加载测试的数据。创建一个静态的`IEnumerable`方法`GetLoanDTOs`，使用`yield`语句返回一个`LoanDTO`对象给测试方法：

```cs
public static IEnumerable<object[]> GetLoanDTOs()
{
       yield return new object[]
       {
           new LoanDTO
            {
                LoanType = LoanType.CarLoan,
                JobType = JobType.Professional,
                 LocationType = LocationType.Location1
             }
        };

       yield return new object[]
       {
            new LoanDTO
            {
                LoanType = LoanType.CarLoan,
                JobType = JobType.Professional,
                LocationType = LocationType.Location2
            }
        };
  }
```

`MemberData`属性要求将数据源的名称作为参数传递给它，以便在后续调用中加载测试执行所需的数据行。静态方法、属性或字段的名称可以作为字符串传递到`MemberData`属性中，形式为`MemberData("methodName")`：

```cs
 [Theory, MemberData("GetLoanDTOs")]
 public void Test_CalculateLoan_ShouldReturnCorrectRate(LoanDTO loanDTO)
 {
     Loan loan = carLoanCalculator.CalculateLoan(loanDTO);
     Assert.NotNull(loan);
     Assert.InRange(loan.InterestRate, 8, 12);
 }
```

另外，数据源名称可以通过`nameof`表达式传递给`MemeberData`属性，`nameof`是 C#关键字，用于获取变量、类型或成员的字符串名称。语法是`MemberData(nameof(methodName))`：

```cs
 [Theory, MemberData(nameof(GetLoanDTOs))]
 public void Test_CalculateLoan_ShouldReturnCorrectRate(LoanDTO loanDTO)
 {
     Loan loan = carLoanCalculator.CalculateLoan(loanDTO);
     Assert.NotNull(loan);
     Assert.InRange(loan.InterestRate, 8, 12);
 }
```

与`MemberData`属性一起使用静态方法类似，静态字段和属性可以用于提供数据理论的数据集。

`Test_CalculateLoan_ShouldReturnCorrectRate`可以重构以使用静态属性代替方法：

```cs
[Theory, MemberData("LoanDTOs")]
        public void Test_CalculateLoan_ShouldReturnCorrectRate(LoanDTO loanDTO)
        {
            Loan loan = carLoanCalculator.CalculateLoan(loanDTO);
            Assert.NotNull(loan);
            Assert.InRange(loan.InterestRate, 8, 12);
        }
```

创建一个静态属性`LoanDTOs`，返回`IEnumerable<object[]>`，这是作为`MemberData`属性参数的资格要求。`LoanDTOs`随后用作属性的参数：

```cs
public static IEnumerable<object[]> LoanDTOs
{
            get
            {
                yield return new object[]
                {
                    new LoanDTO
                    {
                        LoanType = LoanType.CarLoan,
                        JobType = JobType.Professional,
                        LocationType = LocationType.Location1
                    }
                };

                yield return new object[]
                {
                    new LoanDTO
                    {
                        LoanType = LoanType.CarLoan,
                        JobType = JobType.Professional,
                        LocationType = LocationType.Location2
                    }
                };
 }
```

每当运行`Test_CalculateLoan_ShouldReturnCorrectRate`时，将创建两个测试，对应于作为数据源返回的两个数据集。

遵循上述方法要求静态方法、字段或属性用于加载测试数据的位置与数据理论相同。为了使测试组织良好，有时需要将测试方法与用于加载数据的静态方法或属性分开放在不同的类中：

```cs
public class DataClass
{
    public static IEnumerable<object[]> LoanDTOs
    {
            get
             {
                    yield return new object[]
                    {
                        new LoanDTO
                        {
                            LoanType = LoanType.CarLoan,
                            JobType = JobType.Professional,
                            LocationType = LocationType.Location1
                        }
                    };

                yield return new object[]
                 {
                        new LoanDTO
                        {
                            LoanType = LoanType.CarLoan,
                            JobType = JobType.Professional,
                            LocationType = LocationType.Location2
                        }
                };
           }
     }
}
```

当测试方法写在与静态方法不同的单独类中时，必须在`MemberData`属性中指定包含方法的类，使用`MemberType`，并分配包含类，使用类名，如下面的代码片段所示：

```cs
[Theory, MemberData(nameof(LoanDTOs), MemberType = typeof(DataClass))]
public void Test_CalculateLoan_ShouldReturnCorrectRate(LoanDTO loanDTO)
{
       Loan loan = carLoanCalculator.CalculateLoan(loanDTO);
       Assert.NotNull(loan);
       Assert.InRange(loan.InterestRate, 8, 12);
}        
```

在使用静态方法时，该方法也可以有一个参数，当处理数据时可能需要使用该参数。例如，可以将整数值传递给方法，以指定要返回的记录数。该参数可以直接从`MemberData`属性传递给静态方法：

```cs
[Theory, MemberData(nameof(GetLoanDTOs),  parameters: 1, MemberType = typeof(DataClass))]
public void Test_CalculateLoan_ShouldReturnCorrectRate3(LoanDTO loanDTO)
{
     Loan loan = carLoanCalculator.CalculateLoan(loanDTO);
     Assert.NotNull(loan);
     Assert.InRange(loan.InterestRate, 8, 12);
}        
```

`DataClass`中的`GetLoanDTOs`方法可以重构为接受一个整数参数，用于限制要返回的记录数，以填充执行`Test_CalculateLoan_ShouldReturnCorrectRate`所需的数据行：

```cs
public class DataClass
{
    public static IEnumerable<object[]> GetLoanDTOs(int records)
    {
        var loanDTOs = new List<object[]>
        {
               new object[]
               {
                   new LoanDTO
                   {
                       LoanType = LoanType.CarLoan,
                       JobType = JobType.Professional,
                         LocationType = LocationType.Location1
                    }
               },
               new object[]
               {
                    new LoanDTO
                    {
                        LoanType = LoanType.CarLoan,
                        JobType = JobType.Professional,
                        LocationType = LocationType.Location2
                     }
                 }
         };
        return loanDTOs.TakeLast(records);
    }
}
```

# ClassData 属性

`ClassData`是另一个属性，可以使用它来通过来自类的数据创建数据驱动测试。`ClassData`属性接受一个可以实例化以获取将用于执行数据理论的数据的类。具有数据的类必须实现`IEnumerable<object[]>`，每个数据项都作为`object`数组返回。还必须实现`GetEnumerator`方法。

让我们创建一个`LoanDTOData`类，用于提供数据以测试`Test_CalculateLoan_ShouldReturnCorrectRate`方法。`LoanDTOData`将返回`LoanDTO`的`IEnumerable`对象：

```cs
public class LoanDTOData : IEnumerable<object[]>
{
     private IEnumerable<object[]> data => new[]
     {
                new object[]
                {
                    new LoanDTO
                    {
                        LoanType = LoanType.CarLoan,
                        JobType = JobType.Professional,
                        LocationType = LocationType.Location1
                    }
                },
                new object[]
                {
                    new LoanDTO
                    {
                        LoanType = LoanType.CarLoan,
                        JobType = JobType.Professional,
                        LocationType = LocationType.Location2
                    }
                }
      };

      IEnumerator IEnumerable.GetEnumerator()
      {
            return GetEnumerator();
      }

      public IEnumerator<object[]> GetEnumerator()
      {
            return data.GetEnumerator();
      }
}

```

实现了`LoanDTOData`类之后，可以使用`ClassData`属性装饰`Test_CalculateLoan_ShouldReturnCorrectRate`，并将`LoanDTOData`作为属性参数传递，以指定`LoanDTOData`将被实例化以返回测试方法执行所需的数据：

```cs
[Theory, ClassData(typeof(LoanDTOData))]
public void Test_CalculateLoan_ShouldReturnCorrectRate(LoanDTO loanDTO)
{
    Loan loan = carLoanCalculator.CalculateLoan(loanDTO);
    Assert.NotNull(loan);
    Assert.InRange(loan.InterestRate, 8, 12);
}
```

使用任何合适的方法，都可以灵活地实现枚举器，无论是使用类属性还是方法。在运行测试之前，xUnit.net 框架将在类上调用`.ToList()`。在使用`ClassData`属性将数据传递给您的测试时，您总是需要创建一个专用类来包含您的数据。

# 整合来自其他来源的数据

虽然您可以使用前面讨论过的 xUnit.net 理论属性编写基本的数据驱动测试，但有时您可能希望做更多的事情，比如连接到 SQL Server 数据库表，以获取用于执行测试的数据。xUnit.net 的早期版本具有来自`xUnit.net.extensions`的其他属性，允许您轻松地从不同来源获取数据，以用于您的测试。`xUnit.net.extensions`包在**xUnit.net v2**中不再可用。

但是，`xUnit.net.extensions`中的类在示例项目中可用：[`github.com/xUnit.net/samples.xUnit.net.`](https://github.com/xUnit.net/samples.xUnit.net)如果您希望使用此属性，可以将示例项目中的代码复制到您的项目中。

# SqlServerData 属性

在项目的`SqlDataExample`文件夹中，有一些文件可以复制到您的项目中，以便为您提供直接连接到 SQL Server 数据库或可以使用*OLEDB*访问的任何数据源的功能。该文件夹中的四个类是`DataAdapterDataAttribute`，`DataAdapterDataAttributeDiscoverer`，`OleDbDataAttribute`和`SqlServerDataAttribute`。

需要注意的是，由于.NET Core 不支持 OLEDB，因此无法在.NET Core 项目中使用前面的扩展。这是因为 OLEDB 技术是基于 COM 的，依赖于仅在 Windows 上可用的组件。但是您可以在常规.NET 项目中使用此扩展。

GitHub 上的 xUnit.net 存储库中提供了`SqlServerData`属性的代码清单，该属性可用于装饰数据理论，以直接从 Microsoft SQL Server 数据库表中获取测试执行所需的数据。

为了测试`SqlServerData`属性，您应该在您的 SQL Server 实例中创建一个名为`TheoryDb`的数据库。创建一个名为`Palindrome`的表；它应该有一个名为`varchar`的列。用样本数据填充表，以便用于测试：

```cs
CREATE TABLE [dbo].Palindrome NOT NULL
) ;

INSERT INTO [dbo].[Palindrome] ([word]) VALUES ('civic')
GO
INSERT INTO [dbo].[Palindrome] ([word]) VALUES ('dad')
GO
INSERT INTO [dbo].[Palindrome] ([word]) VALUES ('omo')
GO
```

`PalindronmeChecker`类运行一个`IsWordPalindrome`方法来验证一个单词是否是回文，如下面的代码片段所示。回文是一个可以在两个方向上阅读的单词，例如`dad`或`civic`。在不使用算法实现的情况下，快速检查这一点的方法是反转单词并使用字符串`SequenceEqual`方法来检查这两个单词是否相等：

```cs
public class PalindromeChecker
{
    public bool IsWordPalindrome(string word)
    {
        return word.SequenceEqual(word.Reverse());
    }
}
```

为了测试`IsWordPalindrome`方法，将实现一个测试方法`Test_IsWordPalindrome_ShouldReturnTrue`，并用`SqlServerData`属性进行装饰。此属性需要三个参数——数据库服务器地址、数据库名称和用于从包含要加载到测试中的数据的表或视图中检索数据的选择语句：

```cs
public class PalindromeCheckerTest
    {
        [Theory, SqlServerData(@".\sqlexpress", "TheoryDb", "select word from Palindrome")]
        public void Test_IsWordPalindrome_ShouldReturnTrue(string word)
        {
            PalindromeChecker palindromeChecker = new PalindromeChecker();
            Assert.True(palindromeChecker.IsWordPalindrome(word));
        }
    }
```

当运行`Test_IsWordPalindrome_ShouldReturnTrue`时，将执行`SqlServerData`属性，以从数据库表中获取记录，用于执行测试方法。要创建的测试数量取决于表中可用的记录。在这种情况下，将创建并执行三个测试：

![](img/f7472981-0328-44c8-8843-367ee078defd.png)

# 自定义属性

与 xUnit.net GitHub 存储库中可用的`SqlServerData`属性类似，您可以创建一个自定义属性来从任何源加载数据。自定义属性类必须实现`DataAttribute`，这是一个表示理论要使用的数据源的抽象类。自定义属性类必须重写并实现`GetData`方法。该方法返回`IEnumerable<object[]>`，用于包装要返回的数据集的内容。

让我们创建一个`CsvData`自定义属性，可以用于从`.csv`文件中加载数据，用于数据驱动的单元测试。该类将具有一个构造函数，它接受两个参数。第一个是包含`.csv`文件的完整路径的字符串参数。第二个参数是一个布尔值，当为`true`时，指定是否应使用包含在`.csv`文件中的数据的第一行作为列标题，当为`false`时，指定忽略文件中的列标题，这意味着 CSV 数据从第一行开始。

自定义属性类是`CsvDataAttribute`，它实现了`DataAttribute`类。该类用`AttributeUsage`属性修饰，该属性具有以下参数—`AttributeTargets`用于指定应用属性的有效应用元素，`AllowMultiple`用于指定是否可以在单个应用元素上指定属性的多个实例，`Inherited`用于指定属性是否可以被派生类或覆盖成员继承：

```cs
[AttributeUsage(AttributeTargets.Method, AllowMultiple = false, Inherited = false)]
    public class CsvDataAttribute : DataAttribute
    {
        private readonly string filePath;
        private readonly bool hasHeaders;
        public CsvDataAttribute(string filePath, bool hasHeaders)
        {
            this.filePath = filePath;
            this.hasHeaders = hasHeaders;
        }       
        // To be followed by GetData implementation
    }
```

下一步是实现`GetData`方法，该方法将覆盖`DataAttribute`类中可用的实现。此方法使用`System.IO`命名空间中的`StreamReader`类逐行读取`.csv`文件的内容。实现了第二个实用方法`ConverCsv`，用于将 CSV 数据转换为整数值：

```cs
public override IEnumerable<object[]> GetData(MethodInfo methodInfo)
{
    var methodParameters = methodInfo.GetParameters();
    var parameterTypes = methodParameters.Select(x => x.ParameterType).ToArray();
    using (var streamReader = new StreamReader(filePath))
    {
        if(hasHeaders)
            streamReader.ReadLine();
        string csvLine=string.Empty;
        while ((csvLine = streamReader.ReadLine()) != null)
        {
            var csvRow = csvLine.Split(',');
            yield return ConvertCsv((object[])csvRow, parameterTypes);
        }
    }
}

 private static object[] ConvertCsv(IReadOnlyList<object> csvRow, IReadOnlyList<Type> parameterTypes)
 {
    var convertedObject = new object[parameterTypes.Count];
    //convert object if integer
    for (int i = 0; i < parameterTypes.Count; i++)
      convertedObject[i] = (parameterTypes[i] == typeof(int)) ? Convert.ToInt32(csvRow[i]) : csvRow[i]; 
    return convertedObject;
 }
```

创建的自定义属性现在可以与 xUnit.net 的`Theory`属性一起使用，以从`.csv`文件中提供数据给理论。

`Test_IsWordPalindrome_ShouldReturnTrue`测试方法将被修改以使用新创建的`CsvData`属性，以从`.csv`文件中获取测试执行的数据：

```cs
 public class PalindromeCheckerTest
 {
        [Theory, CsvData(@"C:\data.csv", false)]
        public void Test_IsWordPalindrome_ShouldReturnTrue(string word)
        {
            PalindromeChecker palindromeChecker = new PalindromeChecker();
            Assert.True(palindromeChecker.IsWordPalindrome(word));
        }
 }
```

当您在 Visual Studio 中运行前面片段中的`Test_IsWordPalindrome_ShouldReturnTrue`测试方法时，测试运行器将创建三个测试。这应该对应于从`.csv`文件中检索到的记录或数据行数。测试信息可以从测试资源管理器中查看：

![](img/7bae2c94-06b8-4b72-9505-3493b7cfb2c8.png)

`CsvData`自定义属性可以从任何`.csv`文件中检索数据，无论单行上存在多少列。记录将被提取并传递给测试方法中的`Theory`属性。

让我们创建一个具有两个整数参数`firstNumber`和`secondNumber`的方法。该方法将计算整数值`firstNumber`和`secondNumber`的最大公约数。这两个整数的最大公约数是能够整除这两个整数的最大值：

```cs

public int GetGcd(int firstNumber, int secondNumber)
{
    if (secondNumber == 0)
        return firstNumber;    
    else
        return GetGcd(secondNumber, firstNumber % secondNumber);    
}
```

现在，让我们编写一个测试方法来验证`GetGcd`方法。`Test_GetGcd_ShouldRetunTrue`将是一个数据理论，并具有三个整数参数—`firstNumber`、`secondNumber`和`gcdValue`。该方法将检查在调用时`gdcValue`参数中提供的值是否与调用时`GetGcd`方法返回的值匹配。测试的数据将从`.csv`文件中加载：

```cs
[Theory, CsvData(@"C:\gcd.csv", false)]
public void Test_GetGcd_ShouldRetunTrue(int firstNumber, int secondNumber, int gcd)
{
    int gcdValue=GetGcd(firstNumber,secondNumber);
    Assert.Equal(gcd,gcdValue);
}
```

根据`.csv`文件中提供的值，将创建测试。以下屏幕截图显示了运行时`Test_GetGcdShouldReturnTrue`的结果。创建了三个测试；一个通过，两个失败：

![](img/53b2079c-961e-4f2b-983f-04c88278c973.png)

# 摘要

数据驱动的单元测试是 TDD 的重要概念，它带来了许多好处，可以让您使用来自多个数据源的真实数据广泛测试代码库，为您提供调整和重构代码以获得更好性能和健壮性所需的洞察力。

在本章中，我们介绍了数据驱动测试的好处，以及如何使用 xUnit.net 的内联和属性属性编写有效的数据驱动测试。此外，我们还探讨了在 xUnit.net 中使用的`Theory`属性进行数据驱动的单元测试。这使您能够针对来自不同数据源的广泛输入对代码进行适当的验证和验证。

虽然 xUnit.net 提供的默认数据源属性非常有用，但您可以进一步扩展`DataAttribute`类，并创建一个自定义属性来从另一个源加载数据。我们演示了`CsvData`自定义属性的实现，以从`.csv`文件加载测试数据。

在下一章中，我们将深入探讨另一个重要且有用的 TDD 概念，即依赖项模拟。模拟允许您在不必直接构造或执行依赖项代码的情况下，有效地对方法和类进行单元测试。
