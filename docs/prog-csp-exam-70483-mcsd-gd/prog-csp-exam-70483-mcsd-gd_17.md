# 第十七章：模拟测试 1

1.  我们有一个名为 `LogException` 的类。该类使用以下代码段实现 `CaptureException` 方法：`public static void CaptureException(Exception ex)`。从以下语法中选择一个，以确保捕获类中的所有异常并重新抛出原始异常，包括堆栈：

    1.  `catch (Exception ex)`

        `{`

        `LogException.CaptureException(ex);`

        `throw;`

        `}`

    1.  `catch (Exception ex)`

        `{`

        `LogException.CaptureException(ex);`

        `throw ex;`

        `}`

    1.  `catch`

        `{`

        `LogException(new Exception());`

        `}`

    1.  `catch`

        `{`

        `var ex = new Exception();`

        `throw ex;`

        `}`

1.  你正在创建一个名为 `Store` 的类，该类应有一个满足以下要求的 `Store Type` 成员：

    +   该成员必须是公开可访问的。

    +   该成员必须仅获取一组受限的值。

    +   在设置值时，该成员必须确保它验证成员中设置的输入。

在实现分数成员时应采用哪种形式？

+   1.  `public string storeType;`

    1.  `protected String StoreType` `{` `get{}` `set{}` `}`

    1.  `private enum StoreType { Department, Store, Warehouse}`

        `public StoreType StoreTypeProperty`

        `{`

        `get{}`

        `set{}`

        `}`

    1.  `private enum StoreType { Department, Store, Warehouse}`

        `private StoreType StoreTypeProperty`

        `{`

        `get{}`

        `set{}`

        `` `}` ``

1.  为字符串编写一个扩展方法；它应该有一个 `IsEmail` 方法。该方法应检查字符串是否为有效的电子邮件。选择语法并将其映射到应放置的位置：

```cs
----------------------/*Line which needs to be filled*/
{
    ------------------/*Line which needs to be filled*/
    {
         Regex regex = new Regex(@"^([\w\.\-]+)@([\w\-]+)((\.(\w){2,3})+)$");
         return regex.IsMatch(str);
    }
}
```

1.  1.  `protected static class StringExtensions`

    1.  `public static class StringExtensions`

    1.  `public static bool IsEmail(this String str)`

    1.  `public static bool IsEmail(String str)`

    1.  `public class StringExtensions`

1.  你需要编写一个应用程序，确保垃圾回收器在进程完成之前不释放对象的资源。以下哪种语法你会使用？

    1.  `WaitForFullGCComplete()`

    1.  `RemoveMemoryPressure()`

    1.  `SuppressFinalize()`

    1.  `collect()`

1.  对于列表集合，有人编写了以下代码：

```cs
static void Main(string[] args)
{
    List<string> states = new List<string>()
    {
        "Delhi", "Haryana", "Assam", "Punjab", "Madhya Pradesh"
    };
}

private bool GetMatchingStates(List<string> states, string stateName)
{
    var findState = states.Exists(delegate(
    string stateNameToSearch)
    {
        return states.Equals(stateNameToSearch);
    });
    return findState;
}
```

以下哪个代码段是相应 Lambda 表达式的正确表示？

+   1.  `var findState = states.First(x => x == stateName);`

    1.  `var findState = states.Where(x => x == stateName);`

    1.  `var findState = states.Exists(x => x.Equals(stateName));`

    1.  `` `var findState = states.Where(x => x.Equals(stateName));` ``

1.  以下哪个集合对象能满足以下要求？

    +   它必须内部存储每个项目的键值对。

    +   它必须允许我们按键的顺序遍历集合。

    +   它允许我们使用键访问对象。

收集对象如下：

+   1.  字典

    1.  栈

    1.  列表

    1.  排序列表

1.  你正在创建一个包含 `Student` 类的应用程序。该应用程序必须有一个 `Save` 方法，该方法应满足以下要求：

    +   它必须是强类型的。

    +   该方法必须仅接受从 `Animal` 类继承且使用不接受任何参数的构造函数的类型。

选项如下：

+   1.  `public static void Save(Student target)`

        `{`

        `}`

    1.  `public static void Save<T>(T target) where T : Student , new()`

        `{`

        `}`

    1.  `public static void Save<T>(T target) where T : new(), Student`

        `{`

        `}`

    1.  `public static void Save<T>(T target) where T : Student`

        `{`

        `` `}` ``

1.  我们正在编写一个应用程序，它从另一个应用程序接收以下格式的 JSON 输入：

```cs
{
  "StudentFirstName" : "James",
  "StudentLastName" : "Donohoe",
  "StudentScores" : [45, 80, 68]
}
```

我们在我们的应用程序中编写了以下代码来处理输入。在 `ConvertFromJSON` 方法中，为了确保将输入转换为等效的学生格式，正确的语法是什么？

```cs
public class Student
{ 
   public string StudentFirstName {get; set;}
   public string StudentLastName {get; set;}
   public int[] StudentScores {get; set;}
}

public static Student ConvertFromJSON(string json)
{
  var ser = new JavaScriptSerializer();
  ----------------/*Insert a line here*/
}
```

选项如下：

+   1.  `Return ser.Desenalize (json, typeof(Student));`

    1.  `Return ser.ConvertToType<Student>(json);`

    1.  `Return ser.Deserialize<Student>(json);`

    1.  `Return ser.ConvertToType (json, typeof (Student));`

1.  您有一个包含 `studentId` 值的整数数组。您将使用哪种代码逻辑来完成以下操作？

    +   仅选择唯一的 `studentID`

    +   从数组中删除特定的 `studentID`

    +   将结果按降序排序到另一个数组中

您的选项如下：

+   1.  `int[] filteredStudentIDs = studentIDs.Distinct().Where(value => value != studentIDToRemove).OrderByDescending(x => x).ToArray();`

    1.  `int[] filteredStudentIDs = studentIDs.Where(value => value != studentIDToRemove).OrderBy(x => x).ToArray();`

    1.  `int[] filteredStudentIDs = studentIDs.Where(value => value != studentIDToRemove).OrderByDescending(x => x).ToArray();`

    1.  `int[] filteredStudentIDs = studentIDs.Distinct().OrderByDescending(x => x).ToArray();`

1.  在以下代码行中识别缺失的行：

```cs
private static IEnumerable<Country> ReadCountriesFromDB(string sqlConnectionString)
{
    List countries = new List<Country>();
    SqlConnection conn = new SqlConnection();
    using (sqlConnectionString)
    {
        SqlCommand sqlCmd = new SqlCommand("Select name, continent 
        from Counties", sqlConnectionString);
        conn.Open();
        using (SqlDataReader reader = sqlCmd.ExecuteReader())
        {
            // Insert the Line Here
            {
                Country con = new Country();
                con.CountryName = (String)reader["name"];
                con.ContinentName = (String)reader["continent"];
                counties.Add(con);
            }
        }
    }
    return countries;
}
```

+   1.  `while (reader.Read())`

    1.  `while (reader.NextResult())`

    1.  `while (reader.Being())`

    1.  `` `while (reader.Exists())` ``

1.  以便使用 `foreach` 循环处理集合中的每个对象，请按以下方式编写 `StudentCollection` 类：

```cs
public class StudentCollection //Insert Code Here
{
    private Student[] students;
    public StudentCollection(Student[] student)
    {
        students = new Student[student.Length];

        for (int i=0; i< student.Length; i++)
        {
            students[i] = student[i];
        }
    }

    //Insert Code Here
    {
        //Insert Code Here
    }
}
```

+   1.  `: IComparable`

    1.  `: IEnumerable`

    1.  `: IDisposable`

    1.  `public void Dispose()`

    1.  `return students.GetEnumerator();`

    1.  `return obj == null ? 1: students.Length;`

    1.  `` `public IEnumerator GetEnumerator()` ``

1.  如果您正在编写代码以按照以下条件打开文件，您会使用以下哪一行代码？

    +   不应更改文件。

    +   如果文件不存在，应用程序应抛出错误。

    +   在操作进行时，不允许其他进程更新此文件。

    1.  `var fs = File.Open(Filename, FileMode.OpenOrCreate, FileAccess.Read, FileShare.ReadWrite);`

    1.  `var fs = File.Open(Filename, FileMode.Open, FileAccess.Read, FileShare.ReadWrite);`

    1.  `var fs = File.Open(Filename, FileMode.Open, FileAccess.Read, FileShare.Read);`

    1.  `var fs = File.Open(Filename, FileMode.Open, FileAccess.ReadWrite, FileShare.Read);`

1.  在将浮点数转换为整数时，您会使用以下哪一行代码？您需要确保转换不会抛出 `Float floatPercentage;` 异常：

    1.  `int roundPercentage = (int)floatPercentage;`

    1.  `int roundPercentage = (int)(double)floatPercentage;`

    1.  `int roundPercentage = floatPercentage;`

    1.  `int roundPercentage = (int)(float)floatPercentage;`

1.  以下哪行代码用于将浮点数转换为整数？你需要确保转换不会抛出`Float floatPercentage;`异常：

    1.  `int roundPercentage = (int)floatPercentage;`

    1.  `int roundPercentage = (int)(double)floatPercentage;`

    1.  `int roundPercentage = floatPercentage;`

    1.  `int roundPercentage = (int)(float)floatPercentage;`

1.  我们正在创建一个具有`StudentType`属性的`Student`类。我们需要确保`StudentType`属性只能在`Student`类内部或由继承自`Student`类的类访问。以下哪种实现你会使用？

    1.  `public class Student`

        `{`

        `protected string StudentType`

        `{`

        `get;`

        `set;`

        `}`

        `}`

    1.  `public class Student`

        `{`

        `internal string StudentType`

        `{`

        `get;`

        `set;`

        `}`

        `}`

    1.  `public class Student`

        `{`

        `private string StudentType`

        `{`

        `get;`

        `set;`

        `}`

        `}`

    1.  `public class Student`

        `{`

        `public string StudentType`

        `{`

        `get;`

        `set;`

        `}`

        `}`

1.  我们正在编写一个应用程序，其中我们声明了一个具有两个属性`CarCategory`和`CarName`的`Car`类。在执行过程中，我们需要将类转换为它的 JSON 字符串表示。参考以下代码片段：

```cs
public enum CarCategory
{
     Luxury,
     Sports,
     Family,
     CountryDrive
}

[DataContract]
public class Car
{
     [DataMember]
     public string CarName { get; set; }
     [DataMember]
     public enum CarCategory { get; set; }
}

void ShareCareDetails()
{
     var car = new Car { CarName = "Mazda", CarCategory = CarCategory.Family };
     var serializedCar = /// Insert the code here 
}
```

以下哪行代码用于获取 JSON 表示的正确结构？

+   1.  `new DataContractSerializer(typeof(Car))`

    1.  `new XmlSerializer(typeof(Car))`

    1.  `new NetDataContractSerializer()`

    1.  `new DataContractJsonSerializer(typeof(Car))`

1.  我们正在为一家银行编写一个应用程序，其中我们使用以下代码来计算指定月份的利息金额以及银行初始存入的金额：

```cs
1   private static decimal CalculateBankAccountInterest(decimal initialAmount, int numberOfMonths)
2   {
3        decimal interestAmount;
4        decimal interest;
5        if(numberOfMonths > 0 && numberOfMonths < 6 && initialAmount < 5000)
6        {
7            interest = 0.05m;
8        }
9        else if(numberOfMonths > 6 && initialAmount > 5000)
10       {
11           interest = 0.065m;
12       } 
13       else
14       {
15           interest = 0.06m;
16       }
17    
18       interestAmount = interest * initialAmount * numberOfMonths / 12;
19       return interestAmount;
20  }
```

我们了解到，如果月份是 6，应用程序计算出的利息金额是不正确的。如果月份是 6，利率应该是 6.2%。以下哪行代码会进行更改？

+   1.  将第 7 行替换为`interest = 0.062m`

    1.  将第 11 行替换为`interest = 0.06m`

    1.  将第 4 行替换为`decimal interest = 0.062m`

    1.  将第 15 行替换为`interest = 0.062m`

1.  我们正在编写一个应用程序，其中我们正在对三个不同的服务进行异步调用，如下例所示：

```cs
public async Task ExecuteMultipleRequestsInParallel() 
{ 
    HttpClient client = new HttpClient(); 
    Task task1 = client.GetStringAsync("ServiceUrlA"); 
    Task task2 = client.GetStringAsync("ServiceUrlB"); 
    Task task3 = client.GetStringAsync("ServiceUrlC"); 
    // Insert the call here 
}
```

在控制权可以返回到调用函数之前，你需要等待从前面三个服务获取结果，以下哪一行代码你会插入？

+   1.  `await Task.Yield();`

    1.  `await Task.WhenAll(task1, task2, task3);`

    1.  `await Task.WaitForCompletion(task1, task2, task3);`

    1.  `await Task.WaitAll();`

1.  我们正在编写一个应用程序，其中我们正在执行多个操作，如字符串变量的赋值、修改和替换。以下哪个关键字会用来确保操作尽可能少地消耗内存？

    1.  `String.Concat`

    1.  `+ operator`

    1.  `StringBuilder`

    1.  `` `String.Add` ``

1.  我们正在编写一个应用程序，其中我们在列表中维护学生的分数，如下面的代码块所示。我们需要编写一个语句来过滤出大于 75 分的分数。您会使用哪个语句？

```cs
List<int> scores = new List<int>()
{
    90,
    55,
    80,
    65
};
```

1.  1.  `var filteredScores = scores.Skip(75);`

    1.  `var filteredScores = scores.Where(i => i > 75);`

    1.  `var filteredScores = scores.Take(75);`

    1.  `var filteredScores = from i in scores`

        `groupby i into tempList`

        `where tempList.Key > 75`

        `select i;`
