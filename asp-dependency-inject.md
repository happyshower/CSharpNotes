#### ASP .NET Core 依赖注入

- ASP .NET Core 应用在启动过程中会依赖各种组件提供服务，而这些组件会以接口的形式标准化，这些组件就是服务，ASP .NET Core 框架建立在一个底层的依赖注入框架之上，它使用容器提供所需的服务。
- ASP .NET Core 支持依赖关系注入 (DI) 软件设计模式，这是一种在类及其依赖关系之间实现控制反转 (IoC) 的技术。
- 依赖注入是实现依赖倒置原则的方式之一，其核心概念是高层模块不应该依赖于底层模块。二者都应该依赖于抽象；抽象不应该依赖于细节，细节应该依赖于抽象。

#### 依赖注入的优点

- **降低组件之间的耦合度**。各个组件之间经常需要直接引用其他组件来完成各自的任务。这样会导致代码之间的耦合度很高，一旦其中一个组件需要改动，可能会对其他组件产生影响，这样的代码难以维护。使用依赖注入可以解决这个问题，通过在组件之间插入一个中间层，把各个组件之间的依赖关系解耦，使得代码更加灵活、可维护。
- **提高代码的可测试性**。依赖注入可以方便地为每个组件提供单独的测试环境，使得在测试时可以更容易地模拟各种不同的情况，减少测试所需要的时间和成本。
- **提高代码的可维护性**。依赖注入可以更好地遵循面向对象编程的设计原则，使得代码更加模块化和可复用，减少代码的冗余和重复，提高代码的可维护性。
- **支持更好的代码重用和模块化**。通过依赖注入，可以更好地实现组件的重用和模块化，使得代码更加可复用和可维护。

#### 依赖注入步骤：

- 声明接口
- 实现接口
- 依赖注册
- 注入依赖

```csharp
** 定义实体类 **
public class Student
{
    public string Name { get; set; }
    public int Age { get; set; }
}

** 声明接口 **
public interface IMyDependencyService
{
    List<Student> GetStudentInfo();
}

** 实现接口 **
public class MyDependencyService: IMyDependencyService
{
    public List<Student> GetStudentInfo()
    {
        Console.WriteLine("Get Student Information !");
        var list = new List<Student>()
        {
            new Student(){Name = "张三",Age = 45},
            new Student(){Name = "李四",Age = 67},
            new Student(){Name = "王五",Age = 36},
            new Student(){Name = "黄六",Age = 55}
        };
        return list;
    }
}

** 在 Program.cs 中通过 WebApplicationBuilder 的 Services 属性添加服务( 依赖注入 ) **
builder.Services.AddTransient<IMyDependencyService,MyDependencyService>();
//使用 AddTransient ，每次请求都会创建新的服务实例

** Controller.cs  **
public class HomeController : Controller
{
    // 使用传统的方式通过 new 创建 MyDependencyService 实例对象
    // 实例对象的创建依赖于 MyDependencyService，一旦 MyDependencyService 发生改变，所有依赖于 MyDependencyService 的都需要进行调整
    // 这种方式高度耦合，不利于代码的修改和维护
    // private readonly MyDependencyService _dependency = new MyDependencyService();

    // 创建一个指向 IMyDependencyService 的变量 _dependency ；
    private readonly IMyDependencyService _dependency ;

    // 通过构造函数注入
    public HomeController(IMyDependencyService dependency)
    {
        _dependency = dependency;
    }

    public IActionResult Index()
    {
        var list = _dependency.GetStudentInfo();
        for (int i = 0; i < list.Count; i++)
        {
            Console.WriteLine($"Student {i} Information: Name: {list[i].Name}, Age: {list[i].Age}");
        }
        return View();
    }
}

// 输出：
// Get Student Information !
// Student 0 Information: Name: 张三, Age: 45
// Student 1 Information: Name: 李四, Age: 67
// Student 2 Information: Name: 王五, Age: 36
// Student 3 Information: Name: 黄六, Age: 55

```

#### 注入依赖方式拓展

- 构造函数注入

  - 常用的注入方式，依赖项通过类的构造函数传入

  ```csharp
  public class HomeController : Controller
  {
      private readonly IMyDependencyService _dependency ;

      public HomeController(IMyDependencyService dependency)
      {
          _dependency = dependency;
      }

      public IActionResult Index()
      {
          var list = _dependency.GetStudentInfo();
          for (int i = 0; i < list.Count; i++)
          {
              Console.WriteLine($"Student {i} Information: Name: {list[i].Name}, Age: {list[i].Age}");
          }
          return View();
      }
  }
  ```

- FromServices 属性注入

  - 依赖项通过属性设置，用于某些特定场景。

  ```csharp
  public class HomeController : Controller
  {
      [FromServices]
      public IMyDependencyService MyDependencyService { get; set; }

      public IActionResult Index()
      {
          var list = MyDependencyService.GetStudentInfo();
          for (int i = 0; i < list.Count; i++)
          {
              Console.WriteLine($"Student {i} Information: Name: {list[i].Name}, Age: {list[i].Age}");
          }
          return View();
      }
  }
  ```

- FromServices 动作方法注入
  - 当需要在多个方法中使用注入的实例时，应该使用构造函数注入。若只需要在特定的动作方法中使用实例，最好在动作方法中注入实例，而不是使用构造函数注入。
  ```csharp
  public class HomeController : Controller
  {
      public IActionResult Index([FromServices]IMyDependencyService dependency)
      {
          var list = dependency.GetService<IMyDependencyService>().GetStudentInfo();
          for (int i = 0; i < list.Count; i++)
          {
              Console.WriteLine($"Student {i} Information: Name: {list[i].Name}, Age: {list[i].Age}");
          }
          return View();
      }
  }
  ```
- IServiceProvider 服务器定位注入

  - 当需要在控制器中注入许多不同的服务时，若使用构造函数注入，则必须在构造函数中指定多个参数。所以，这种场景下，有一个更好的解决方案，就是使用 IServiceProvider。

  ```csharp
  public class HomeController : Controller
  {
      private readonly IServiceProvider _provider;

      public HomeController(IServiceProvider provider)
      {
          _provider = provider;
      }

      public IActionResult Index()
      {
          var list = _provider.GetService<IMyDependencyService>().GetStudentInfo();
          for (int i = 0; i < list.Count; i++)
          {
              Console.WriteLine($"Student {i} Information: Name: {list[i].Name}, Age: {list[i].Age}");
          }
          return View();
      }
  }
  ```
