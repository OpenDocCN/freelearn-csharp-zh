# 第十四章。前方的路

在我们的书中，我们经历了一场关于各种主题的风暴之旅。如果你已经到达这个阶段，你可能已经遇到了一些新概念，或者可能对你已经知道的一些事情有了新的看法。总的来说，本书的主题可以分为以下四个部分：

+   将模式置于正确的视角（第一章和第二章）

+   GoF 实践（第三章至第七章）

+   面向对象/函数式编程（第八章至第十章）

+   （功能）响应式编程（第十一章、第十二章和第十三章）

模式是一个有趣的话题，它们通过提供经过验证和经得起时间考验的解决方案，帮助软件开发者解决复杂的业务问题。它也改善了开发者和他们的利益相关者之间的沟通。通过学习模式，作为一名开发者，你可以获得那些编目这些模式的资深程序员的知识和经验。但在你作为开发者或架构师的旅程中，还有一些其他主题是你应该知道的。本书的作者认为以下四个主题对专业人士来说非常有兴趣（作为本书所涵盖主题的延续）：

+   多语言编程和设计（多范式编程）

+   领域特定语言

+   本体论

+   反模式

# 多语言编程和设计

现代应用程序的开发可能很复杂，因为它们可能具有以下特点：

+   带有数据库持久化（SQL 或 NoSQL）的服务层

+   UI 代码必须对不同的设备形态做出响应并进行校准

+   前端代码主要基于某种**单页应用架构**（SPA）架构

+   在某些情况下，可能会有桌面和移动原生前端

被雇佣参与此类项目的开发者应该具备以下技能：

+   Java、C#、PHP、Ruby 或 Grails 用于编写服务层（掌握一种或多种技能）

+   对于编写 UI 代码，他们应该熟悉 CSS 库和 JavaScript（例如 JQuery）

+   对于基于 Web 的响应式前端，使用 TypeScript/JavaScript 结合 Angular、ReactJs 或其他基于 JavaScript 的框架

+   对于编写桌面应用程序，选择有 C#、C++、Objective C/C++、Python 或 Java

+   对于移动应用程序开发（原生），选择有 C#、Java 或 Objective C/Swift

这里底线是，一个人应该至少熟悉超过半打编程语言，才能在现代项目中工作。如果你还掌握了 PowerShell、Bash shell、JSP/HTML 模板引擎、XML、JSON 等技能，我们确实生活在一个一个人必须尽可能精通多种语言的世界。

让我们快速回顾 PC 平台下的开发历史，以追踪这一状况的演变。在过去十年中，编程环境发生了很大的变化。当 GoF 模式手册被编写时，像 Java 和 C# 这样的编程语言还没有出现。当时占主导地位的对象导向编程语言，从所有实际目的来看是 C++。即使是 C++，ANSI 标准（直到 1998 年才得到批准）。除了 C++，像 Delphi（Object Pascal）、Visual Basic、PowerBuilder 以及各种 xBase 方言（如 Clipper、FoxPro 等）在那个时代的程序员中被广泛使用。像 Microsoft Windows 这样的平台正在发展。各种版本的 UNIX 和 MS-DOS 是主要平台（甚至当时 GNU Linux 还没有达到临界质量！）

即使在编程的早期阶段，人们就已经意识到不能仅使用一种编程语言来构建解决方案。我们需要根据上下文混合使用适合的编程语言。在某个时期，大多数数值库都是用 Fortran 编写的，而这些库通过与 C 语言接口来编写系统，这些系统对数值计算有很高的要求。在编程的早期，人们通常使用高级语言编写代码，并将代码与用 C 和汇编语言编写的例程接口。因此，程序员需要了解高级语言、C 编程语言以及程序开发和部署环境中汇编语言的语法。使用不同的编程语言编写不同的子系统，并将它们组合起来创建更大系统的开发过程被称为多语言编程。设计这样的系统，其中存在多种编程语言和技术堆栈，被称为多语言设计。为了设计这样的系统，开发者应该能够以语言和平台无关的方式进行思考。

多语言编程的潮流始于 IBM PC 平台的早期编程阶段。多语言编程主要涉及将用 C 编程语言编写的例程与用于编写系统的所选高级语言进行接口。

## 使用 C/C++ 进行多语言编程

当人们使用除 C/C++以外的各种语言进行编程时，有选项可以用于与 C/C++接口，代码在 Windows 中以 DLL 打包，在 Unix/Linux 中以`.so`打包。在 MS-DOS 中，所有 xBase 语言都提供了一个与用 C/C++和汇编器编写的本地代码交互的接口机制。在 Microsoft Windows 平台上，Visual Basic 提供了一个调用用 C/C++编写的例程的机制。使用这个机制，人们消耗了大量的由 Microsoft 提供的 Windows API 函数。人们开始用 Visual Basic 和 Visual C++编写 Windows 程序。像 Delphi 和 PowerBuilder 这样的可视化开发环境也提供了类似的机制来消耗用 C/C++编写的代码。这可以被认为是使用多种语言编程系统的开始，现在被称为多语言编程。

随着 Java 编程语言的问世，平台无关编程的承诺在业界成为了一个热门词汇。**一次编写，到处运行**（**WORE**）的口号浮出水面，这并没有阻止 Java 的实现者提供一个与用 C/C++编写的本地代码交互的机制。这个接口被称为**Java 本地接口**（**JNI**）。通过这个机制，人们围绕 Direct3D 和 OpenGL 等技术编写了包装器，创建了如 Java 3D 这样的库。流行的计算机视觉系统 OpenCV 就是通过 JNI 与 Java 软件进行接口的。

.NET 平台在 2001 年公布，它提供了两个与本地代码接口的接口。第一个是 PInvoke，这是一个与从 DLL 中导出的 Win2/Win64 调用进行接口的机制。Microsoft 还提供了一个调用基于 C++的 COM 组件的机制，称为 COM 互操作性。他们将其命名为 COM 调用包装器。为了演示 P/Invoke 的使用，这里有一个小的 C/C++代码片段（编译成 DLL）与 C#代码接口：

```cs
    #include <stdio.h> 
    #include <cstring> 
    #include <Windows.h> 

    using namespace std; 
    //A C/C++ Windows DLL which exports a Function 
    //ListOfSquares to be consumed from C# programming 
    //language. The DLL is compiled using Minimalist 
    //GNU for Windows (MINGW),which is freely downloadable 
    // 
    //Compile and Link 
    //---------------- 
    //  gcc -c -o add_basic.o cpp_code.cpp 
    //  gcc -o add_basic.dll -s -shared add_basic.o -Wl, 
    //  --subsystem,windows 
    // 

    extern "C" __declspec(dllexport) void __stdcall 
    ListOfSquares( 
      long (__stdcall  *square_callback) (int rs) ) 

    { 
      for (int i = 0; i<10; ++i) {  
        //---- Call the C# routine which handles the  
        //---- integer and returns square.  
        //---- This is a blocking call! 

        double ret = (*square_callback)(i); 
        printf("Printing from C/C++ ... %g\n", ret); 
      } 
    } 

```

我们可以使用 Visual C++编译器或 MinGW 编译器（Windows 下的 GCC）来编译前面的代码，创建一个 DLL。我们使用了 MinGW 32 位编译器来生成 Win32 DLL。该 DLL 可以与使用 Visual Studio 或 Mono 编译器在 Windows 下编写的 C#代码接口。如果你知道如何在 Mac OS X 或 GNU Linux 下构建共享库（`.so`），我们也可以在这些平台上运行代码。C#代码可以用来调用之前编写的 DLL：

```cs
    using System; 
    using System.Collections.Generic; 
    using System.Linq; 
    using System.Runtime.InteropServices; 
    using System.Text; 
    using System.Threading.Tasks; 

    namespace entrypoint 
    { 
      class Program 
      { 
        //--- Now you know, a C/C++ Function Pointer 
        //--- is indeed a delegate! 
        public delegate long CallBack(int i); 
        //--- Declare the Function Prototype of the Function 
        //--- We are planning to call from C/C++ 
        [DllImport("D:\\DLL_FOR_DEMO\\add_basic.dll")] 
        private static extern long ListOfSquares(CallBack a); 

        public static long SpitConsole(int i) 
        { 
          Console.WriteLine("Printing from C# {0}", i * i); 
          return i * i; 
        } 

        static void Main(string[] args) 
        { 
          ///////////////////////////// 
          // CallBack Demo 
          // Will print Square from C# and C++ 
          ListOfSquares(SpitConsole); 
        } 
      } 
    } 

```

## 多语言网络编程

**万维网**（**WWW**）在 1995 年左右达到了临界质量，编程景观突然发生了变化。那些学习使用 Windows API 和 X/Windows API 编写桌面系统程序的人发现自己处于极大的困难之中，被迫处理**通用网关接口**（**CGI**）、NSAPI 和 ISAPI 编程模型的复杂性，以生成动态网页。选择的语言是用于 CGI 的 Perl 和 TCL，以及用于 NSAPI/ISAPI 的 C/C++。

这种情况持续了一段时间，直到微软推出了用于创建动态网页的 **Active Server Pages** (**ASP**)，一切突然发生了变化。不久之后，Sun Microsystems 推出了 Java Servlet API。Servlet API 是一个类似于基于 C++ 的 ISAPI 的低级 API，Sun 很快又推出了 **JavaServer Pages** (**JSP**)，这是一种将 Java 代码嵌入到标记中的技术。JSP 在执行机器上可用的 Java 编译器系统的动态转换成 Servlet 代码。还有像 ColdFusion 这样的技术，它利用自定义标记来编写可扩展的应用程序，并且能够与用 Java 编写的代码进行交互。网络编程模型要求人们学习 C++、Java、Visual Basic 和 ASP 来编写动态网页。在微软平台上，人们开始编写 VBScript 代码，并在其中穿插调用用 C++ 和 Visual Basic 编写的 COM/ActiveX 对象。实际上，基于网络的 Web 应用程序交付迫使程序员成为真正的多语言者。

### JavaScript 的发展历程

JavaScript 的出现又迫使开发者学习另一种语言，以便在网页中添加一些交互性。在当时，人们大多对静态类型语言感到舒适，并认为 JavaScript 是一种奇怪的语言。JavaScript 在一段时间内相对默默无闻，直到有人发现了一种巧妙的方法将 JavaScript 与 IE ActiveX 插件混合。在 21 世纪初的早期，人们开始通过在桌面容器中嵌入浏览器控件来编写用户界面，并使用 JavaScript 作为粘合语言与嵌入在这些页面中的 ActiveX 控件进行交互。随着 Google 的 V8 引擎和 Node.js 平台的推出，JavaScript 开始获得动力，并在那时人们开始将其视为一等编程语言。另一个推动力是支持使用 JavaScript 语言进行函数式编程风格，以及诸如 JQuery、Angular、ReactJS 等库的出现。在过去的十年中，对 JavaScript 编程语言的知识对所有程序员来说都变得至关重要，无论他们编写的是服务器端代码还是客户端代码。JavaScript 是程序员被迫学习的另一种语言。

### 动态/脚本语言

Perl、Python、PHP、Ruby、TCL 和 Groovy 等语言的兴起，用于编写命令行、GUI 和网络应用程序，迫使开发者必须在日常工作中掌握其中的一些。PERL 仍然被广泛用于编写 CGI 应用程序、自动化脚本，甚至全球范围内的 GUI 应用程序（Perl/TK）。由于 Python 编程语言中提供了丰富的库，因此它成为了编写（或至少学习）机器学习程序的默认编程语言，并且这些库被广泛用于使用一些 MVC 框架编写网络应用程序。

Ruby on Rails 和 Groovy 与 Grails 都引领了快速开发周期时代，使它们在创业生态系统中迅速渗透。PHP 是世界上最受欢迎的 Web 开发系统，特别适合编写前端应用程序。该平台提供的众多内容管理系统数量也是一个吸引力。在一些人圈子中，使用 PHP 构建前端页面、Java 构建服务逻辑、Silverlight 构建前端（用于交互部分）变得流行！REST 范式的流行使得服务器端逻辑的编写可以任选任何一种语言。只要 URI 保持一致，路由系统就会确保它解析并执行正确的处理逻辑。

### 函数式编程的兴起

如第一章所述，Herb Sutter 的标志性文章《The Free Lunch Is Over》重新点燃了对函数式编程的兴趣，这种编程主要被限制在学术圈。FP 模型非常适合利用多核处理世界。FP 的状态无状态计算模型有助于将应用程序从单核扩展到数十核，而无需任何额外的编程工作。Scala、F#、Clojure、Haskell（在某些细分领域）等语言的兴起，以及它们在 JVM 和 CLR 世界中的可用性，使它们成为编写某些应用程序的可行选择。在微软的世界里，人们开始用 C#和 F#编写代码，将代码打包在同一系统中。学习一种（或多种）函数式语言对于现代程序员来说变得绝对必要。成为多语言者的另一个借口！

### 移动革命

对于世界上的每一位软件开发者来说，具备移动应用程序开发的能力是必要的。苹果的 iOS、谷歌的 Android 和微软的 Windows Phone 是这一领域的领先平台。不幸的是，这些平台的原生应用程序开发语言分别是 Objective C/Swift（iOS）、Java（Android）和 C#（Windows Phone）。.NET 开发者必须学习 Java、Objective C 或 Swift 才能为这些平台编写原生应用程序。如果你想避免这种情况，你可能不得不使用 JavaScript 作为编程语言的混合应用程序开发。无论如何，你需要成为一个多语言者！

## 关于多语言持久性的简要说明

很长一段时间以来，**关系数据库管理系统**（**RDBMSs**）成为了每个已开发业务应用程序的骨架。Oracle Server、Microsoft SQL Server、MYSQL 和 Postgres 系统是数据持久化的默认选择。NoSQL 运动的兴起为我们提供了适合分布式程序的不同类型的数据库，这些数据库应该具有良好的可扩展性。它们如下列出：

+   列存储数据库

+   键值数据库

+   文档数据库

+   图数据库

NoSQL 是一个庞大且多样化的主题，读者应搜索 Google 来了解它，因为对其公平的对待需要许多书籍。因此，一个现代的基于 Web 的应用程序可能会根据应用程序上下文存储在可用的各种持久化技术中的数据。一个典型的用例可能是以下任何一个：

+   用于存储事务数据的 RDBMS

+   用于存储主数据（以实现更快查找）的键/值数据库

+   用于存储实体之间关系的图数据库

+   用于存储分析查询数据的列式数据库

通过选择适当的后端技术，应用程序可以确保大规模应用程序开发的吞吐量和可伸缩性。在应用程序中使用不同持久化技术的过程称为**多语言持久性**。开发者应该真正熟悉这种范式以及多语言编程。

## 如何成为一名多语言程序员？

从编程范式角度来看，世界上只有三种类型的编程语言，如下所示：

+   函数式语言（基于λ演算）

+   逻辑语言（基于谓词逻辑）

+   命令式语言（基于图灵机）

要成为一名当代开发者，需要掌握每个家族中的几种语言。要精通函数式编程（FP），可用的语言选项有 F#、Clojure 和 Scala。逻辑编程语言有 Prolog 和 Datalog。学习它们将有助于提高设计技能，尤其是在构建层次数据结构方面。F#、Scala 和 C#中可用的类型推断算法使用统一算法，这是 Prolog 语言的基础。因此，理解 Prolog 机器模型有助于你欣赏并利用现代编程语言中丰富的类型系统。大多数流行的语言在本质上都是命令式的，掌握几种面向对象/函数式语言（如 C#、Java 8 和 Scala）确实很有帮助。一旦你学习了上述每个家族中的一个代表性语言，你就已经实现了对今后可能创建的或即将创建的任何编程语言的认知飞跃！

# 领域特定语言

一种用于表达特定领域（如金融、工资单、电子电路设计、解析器生成器、输入验证等）问题的解决方案的语言被称为**领域特定语言**（**DSL**）。两个大多数程序员都熟悉的常见例子是**结构化查询语言**（**SQL**）和**正则表达式**（**RE**）。如果我们编写 imperative 代码从数据库中检索数据，这将是一项困难的任务，并且容易出错。SQL 为你提供了一个声明性语言来实现相同的目标，并且它有坚实的数学基础。在搜索字符串时，RE 帮助我们给出复杂的模式以匹配字符串。它有助于避免编写繁琐的逻辑来搜索复杂的字符串匹配。

作为.NET 和 Java 世界中相当流行的 DSL 的具体例子，我们在此粘贴了一个提供给**ANTLR**工具的规范，用于编写一个非常简单的算术评估器。该工具自动从这个规范生成词法分析器和解析器。请查阅 ANTLR 文档以了解以下脚本的语义：

```cs
    grammar Evaluator; 

    options { 
      language=CSharp3; 
    } 

    @lexer::namespace{AntlrExample} 
    @parser::namespace{AntlrExample} 

    public addop 
    : mulop (( '+' | '-' ) mulop)*; 

    mulop 
    : INTEGER (( '*' | '/' ) INTEGER)*; 
    /* 
      * Lexical Analysis Rules 
    */ 

    INTEGER : '0'..'9'+; 
    WS :  (' '|'\t'|'\r'|'\n')+ {Skip();} ; 

```

ANTLR 工具将生成一个词法分析器、一个解析模块，甚至一个树遍历器来处理表达式。我们可以使用这个工具为 C#、Java，甚至 C/C++编写解析器。在 Unix 和 Windows 原生编程世界中，用于相同目的的工具是 Lex（GNU Flex）和 Yacc（GNU Bison）。

作为另一个案例，作者在一个领域特定语言（DSL）项目中工作，该项目评估了一个电子表格的一致性。以下是该项目为图灵完备的 DSL 生成的代码片段：

```cs
    Boolean range_flag; 
    NUMERIC sum_value; 

    BEGIN  

    //--- See whether B3 cell in the CONTROL worksheet 
    //--- present in the Relational database 
    VALIDATE  LOOKUP($(CONTROL.B3), 
    @(EI_SEGMENT_MAPPING.EI_SEGMENT_CODE))==TRUE; 

    //---- The Sum of Range should be 100 or Zero 
    range_flag = SUM_RANGE($(HOURLY_DISTRIBUTION.C2), 
    $(HOURLY_DISTRIBUTION.C25))  == 100.0; 
    range_flag =  range_flag || 
    SUM_RANGE($(HOURLY_DISTRIBUTION.C2), 
    $(HOURLY_DISTRIBUTION.C25)) == 0.0; 

    //--- If false throw exception 
    VALIDATE range_flag; 

    EXCEPTION 
    //----- on Exception execute this 
    ErrorString = "FAILED WHILE VALIDATING CONTROL Sheet Or" 
    ErrorString = ErroString +  
    " WorkSheet Error in the range C2-C25"; 
    WRITELOG  ErrorString; 
    bResult = false; 

    END 

```

上述代码片段使用手动编写的递归下降解析器进行解析，并生成了一个**抽象语法树**（**AST**）。该树以深度优先的方式遍历，以生成.NET IL 代码，进而生成.NET 程序集（DLL）。生成的程序集就像是用 C#编写的逻辑一样被消费！

**可扩展样式表语言**（**XSL**）是另一种领域特定语言，对大多数程序员来说非常熟悉。为了完整性，这里提供了一个小的 XSL 片段，并期望读者能够掌握这门语言以生成具有不同布局的网站：

```cs
    <?xml version="1.0"?> 
    <xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform" 
    version="1.0"> 
      <xsl:output method="xml"/> 
      <xsl:template match="*"> 
        <xsl:element name="{name()}"> 
          <xsl:for-each select="@*"> 
            <xsl:element name="{name()}"> 
              <xsl:value-of select="."/> 
            </xsl:element> 
          </xsl:for-each> 
          <xsl:apply-templates select="*|text()"/> 
        </xsl:element> 
        </xsl:template>  
    </xsl:stylesheet> 

```

其他流行的 DSL 包括 VHDL 和 Verilog（数字硬件描述语言），CSS，以及不同技术堆栈中可用的模板语言，JBoss Drools 等等。

编写和设计自己的 DSL 是一个非常广泛的话题，无法在短短的一章中涵盖。以下是一些需要注意的事项：

+   设计一个模仿领域关注点的对象模型

+   设计一个语言抽象及其相关关键词，以到达语言元素和组成复合元素的规则

+   将语言元素映射到创建的对象模型

+   决定 DSL 应该是内部还是外部？

+   决定 DSL 是否应该是图灵完备的

这个主题在 Debasish Ghosh 和 Martin Fowler/Rebecca Parsons 的精彩书籍中得到了很好的覆盖。请参考以下书籍以深入了解相同的内容：

+   *《在行动中的领域特定语言》（DSLs in Action*）由 Debashish Ghosh（Manning）著

+   *《领域特定语言》（Domain-Specific Languages*）由 Martin Fowler 和 Rebecca Parsons（Addison Wesley Martin Fowler 签名系列）著

# 本体

在软件工程的术语中，本体是描述特定领域话语中存在的实体（类型）、属性（属性）和关系的艺术、工艺和科学。它也可以被视为描述实体、属性、它们的关系甚至标准行为的模型。本体真正有助于提出一个普遍的语言（如在**领域驱动设计**（**DDD**）中），所有利益相关者都同意，这有助于沟通。它避免了在开发多领域软件系统时的混淆。本体定义并代表了软件工程环境中存在的基本术语和关系。

从信息处理的角度来看，以下几点需要注意：

+   命题逻辑和谓词逻辑提供了推理规则和形式结构来编码事实

+   本体以明确的方式定义并帮助表示实体、它们的属性以及实体之间的关系

+   计算有助于在计算机上实现本体——它帮助我们超越哲学本体（前两个步骤）

要编写非平凡的推理系统，软件工程师应该掌握定义和解释本体的技能。他还应该能够将本体及其相关规则编码到计算机上，以创建所讨论本体的有效计算模型。在软件工程中，一个明确定义的本体有助于以下方面：

+   使用通用术语在利益相关者之间共享关于问题/解决方案领域的知识

+   通过过滤本体来创建用于应用开发的模型和元模型（本体的投影）

+   使用本体中定义的元素进行计算

我们可以通过不同的方式来编码本体。所有这些方式都共享以下共同元素：

+   类、类型或概念

+   关系、角色或属性

+   形式规则或公理

+   实例

W3C 语义网创建了一种基于 XML 的语言，称为**Web 本体语言**（**OWL**），用于以结构化的方式描述本体。OWL 嵌入**资源描述格式**（**RDF**）标签来定义标准本体。来自 IBM 网站的一个运输本体如下所示。这些 OWL 标记指定有三个类：

```cs
    <owl:Class rdf:ID="Winery"/>  
    <owl:Class rdf:ID="Region"/>  
    <owl:Class rdf:ID="ConsumableThing"/> 

    <owl:Class rdf:ID="PotableLiquid"> 
      <rdfs:subClassOf rdf:resource="#ConsumableThing" /> 
    </owl:Class> 

    <owl:Class rdf:ID="Wine">  
      <rdfs:subClassOf rdf:resource="&food;PotableLiquid"/>  
      <rdfs:label xml:lang="en">wine</rdfs:label>  
      <rdfs:label xml:lang="fr">vin</rdfs:label>  
    </owl:Class>  

    <owl:Thing rdf:ID="CentralCoastRegion" />  
    <owl:Thing rdf:about="#CentralCoastRegion"> 
      <rdf:type rdf:resource="#Region"/>  
    </owl:Thing> 

    <owl:Class rdf:ID="WineDescriptor" /> 
    <owl:Class rdf:ID="WineColor"> 
      <rdfs:subClassOf rdf:resource="#WineDescriptor" /> 
    </owl:Class> 

    <owl:ObjectProperty rdf:ID="hasWineDescriptor"> 
      <rdfs:domain rdf:resource="#Wine" /> 
      <rdfs:range  rdf:resource="#WineDescriptor" /> 
    </owl:ObjectProperty> 

    <owl:ObjectProperty rdf:ID="hasColor"> 
      <rdfs:subPropertyOf rdf:resource="#hasWineDescriptor" /> 
      <rdfs:range rdf:resource="#WineColor" /> 
    </owl:ObjectProperty> 

```

本节旨在激发本书读者的本体论兴趣。领域驱动设计（DDD）与本体论密切相关，编写面向对象系统的人可以很快地理解它们。为了进入基于本体的软件开发方法，我们推荐一本由 John Sowa 撰写的精彩书籍，书名为《知识表示：逻辑、哲学和计算基础》。该书的第二章讨论了本体在软件工程中的应用。还可以通过访问他们的网站了解 W3C 的语义网倡议。

# 反模式

反模式是一种看似最初能带来收益的解决方案，但最终却证明是反效果的。由于模式是以解决方案命名的，它可能不适合某些场景，最终变成反模式。我们应用模式时的上下文非常重要。反模式出现在软件开发生命周期的各种场景中。它们被广泛地分为以下三个类别：

+   软件开发反模式

+   架构反模式

+   管理反模式

《反模式：危机中的软件、架构和项目的重构》由 William J. Brown、Raphael C. Malveau、Hays W. McCormick III 和 Thomas J. Mowbray 所著，是关于反模式的开创性作品。

为了启动讨论，我们想介绍一些普遍存在的反模式，这些反模式可能是本书读者在其环境中能够相关联的：

+   **粘液反模式**：这种情况通常发生在来自过程式编程语言背景的人设计面向对象系统时。他们创建一个包含大部分逻辑的大类，而相关的类只是数据容器。这种反模式导致一个类承担大部分责任，而其他类执行的是微不足道的逻辑。解决方案是对该类进行重构并分配责任。

+   **拉夫流程反模式**：这也被称为死代码，通常出现在最初作为 POC（原型）系统开始，逐渐演变成生产系统的系统中。在这个转变过程中，大量代码未被使用或从未被触及。

+   **功能分解反模式**：这种反模式出现在系统的主要设计者来自过程式语言（如 C 或 Fortran）的场景中，他们试图将经过时间考验的技术适应面向对象范式。即使面向对象在二十年后成为主流，这种模式仍然反复出现。

+   **鬼魂（又称吉普赛人）**：在面向对象编程的上下文中，有时开发者会创建生命周期短的类，以启动核心类（生命周期长）的处理。这可以通过重构代码，将这些鬼魂类的逻辑包含到为解决问题而创建的类层次结构中来实现。

+   **黄金锤**：这种反模式出现在系统的主要设计者在解决问题时可能有一个偏爱的技术、技术或方法论的背景下。古老的谚语“对于有锤子的人来说，一切看起来都像是钉子”清楚地总结了这种心态。偏爱的工具在可能勉强适用的情况下被释放。作为轶事证据，作者认识一个人，他会试图通过 SNMP 解决每一个问题，而一个普通的 TCP/IP 程序就能完成这项工作。

+   **死胡同**：这种情况通常发生在团队可能利用 ISV 编写的自定义控件来加快开发过程的地方。在生产过程中，供应商可能会破产，或者收购供应商的公司可能不再支持该产品。到那时，代码库可能会与相关的自定义控件耦合。为这些接口编写隔离层可以避免这个问题。作者遇到过几十个这样的系统。

+   **简历驱动设计**：世界上大多数开发者，尤其是在软件服务行业中，将他们的简历放在他们试图解决的问题之前。某些热门技术的存在与否决定了他们的薪酬，他们会在不需要最新技术的情况下使用它们。这种情况很常见，很容易检测到，因为一些简单的场景将包含当前炒作周期中的所有技术。一位作者发现自己处于一个代码库恰好有一个 m 框架、NoSQL 数据库（一个文本文件就能解决问题）、基于 Microsoft **托管扩展性框架**（**MEF**）的插件架构、一个消息系统和一个用于系统的缓存管理器，而这个系统只需要三十行类似事务脚本的代码。将运行良好的生产项目转换为 Angular 2/TypeScript 的当前趋势是简历驱动设计的另一个案例。

+   **意大利面代码**：这是大多数系统中的一种涌现现象，即使在最初定义良好的系统中也是如此。曾经有一种错误的观念，认为意大利面代码只存在于过程式编程代码库和 COBOL 中。在面向对象系统中，随着系统的演变和初始开发者的离开，可能对整个系统不太了解的新开发者可能会以意大利面风格编写代码。最近，作者发现了一个使用 MVVM 模式的 Silverlight 应用程序，其意大利面风格代码比主机的 COBOL 程序还要多！

+   **剪切和粘贴编程**：这通常发生在大型系统中，新开发者由于时间紧迫，试图利用其他地方编写的代码来加快他的开发时间。开发者剪切并粘贴一大块实现类似功能的代码，并进行必要的修改。在这个过程中，一些残留代码在新内容中不必要地被执行。一位作者在与大型电子 CAD 软件合作时发现，由于开发者转向了**剪切和粘贴**模型开发，代码中出现了性能问题，导致一些嵌套循环不必要地运行。由于一段时间内积累的意大利面代码，这种情况在多个地方发生。

+   **烟囱系统**：在大多数企业中，由于应用开发的演进性，软件系统以临时方式集成。一些软件可能使用资源层集成（使用关系数据库），一些可能使用事务脚本（使用批量作业引擎），通过 REST/SOAP 服务进行点对点集成，以及基于 JMS 和 MSMQ 协议的多个消息系统上运行的多技术栈的应用程序。由于合并和收购，以及缺乏统一的企业战略，整个场景变成了维护噩梦。通过拥有企业集成战略和内部标准，我们可以减少集成的复杂性。

其他流行的反模式包括供应商锁定、委员会设计、暖体、隐含架构、智力暴力、分析瘫痪、烟雾和镜子等。

反模式是一个与模式一样重要的主题。虽然模式为你提供了时间验证的命名解决方案来解决常见问题，但反模式和它们的解决方案避免了你在软件工程生命周期中的陷阱。这个主题值得有一本自己的书。我们在这里只是给出了简要的描述。

# 摘要

本章为读者提供了一些指针，以进一步研究软件工程技术。涵盖的主题包括多语言编程、领域特定语言（DSL）、本体和反模式。我们只能简要提及这些主题，因为这些主题需要多本书才能公正地对待它们。确实，关于这些主题的 YouTube 视频、白皮书和书籍中有很多好的材料。还请尝试了解本书第一章中提到的各种模式目录，以构建未来系统架构的坚实基础。
