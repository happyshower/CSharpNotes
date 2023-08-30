### ASP .NET Core MVC 框架

- ASP .NET Core MVC 是使用“模型-视图-控制器（Model-View-Controller）”设计模式构建 Web 应用和 API 的丰富框架，该框架轻量、开源、高度可测试，并针对 ASP .NET Core 进行了优化。
- ASP .NET Core MVC 提供一种基于模式的方式，用于生成可彻底分开管理事务的动态网站。 它提供对标记的完全控制，支持 TDD 友好开发并使用最新的 Web 标准。
  ![ASP .NET Core MVC](https://2.bp.blogspot.com/-jQsIcxk0GuY/WVmlWe-fV2I/AAAAAAACVBc/vhSJb4lGVjAhjF9GUK7hlGD1JcR2tyVWQCLcBGAs/s1600/mvc_life_cycles_3.jpg)

#### 模型 Model

- 模型代表了应用的状态和业务逻辑或其可以展现的一些操作。业务逻辑应该封装在模型，连同应用持久化状态实现逻辑。
  ```csharp
  public class UserModel
  {
      public string Name { get; set; } = "jojo";
      public int Age { get; set; } = 18;
  }
  ```

#### 控制器 Controller

- 控制器用于对一组操作进行定义和分组。 操作（或操作方法）是控制器上一种用来处理请求的方法。 控制器按逻辑将类似的操作集合到一起。 通过这种操作的聚合，可以共同应用路由、缓存和授权等通用规则集。 请求通过路由映射到操作，控制器按请求激活和释放。
- 控制器通常继承自 Microsoft.AspNetCore.Mvc.Controller 类，且其命名通常以 "Controller" 结尾。

  ```csharp
  public class HomeController : Controller
  {
      public IActionResult Index()
      {
          var user = new UserModel();
          // 返回默认视图并传递数据 user
          return View(model: user);
      }

      public IActionResult Privacy()
      {   
          ViewData["Privacy"] = $"This is Customize Privacy Policy!";
          return View();
      }
  }
  ```

#### 视图 View

- 视图负责在用户界面呈现内容。它们使用 Razor 视图引擎在 HTML 标记中嵌入 .NET 代码。视图中应仅包含少量的逻辑，而这些逻辑应该是与呈现内容相关的。

  ```csharp
  @model WebApplication1.Models.UserModel;
  @{
      ViewData["Title"] = "Home Page";
  }

  <div class="text-center">
      <h1 class="display-4">Welcome</h1>
      <h2>UserInfo: @Model.Name, @Model.Age</h2>
  </div>
  ```
