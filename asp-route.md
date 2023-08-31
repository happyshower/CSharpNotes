### ASP .NET Core 路由

- 路由负责匹配传入的 HTTP 请求，然后将这些请求发送到应用的可执行终结点。 终结点是应用的可执行请求处理代码单元。 终结点在应用中进行定义，并在应用启动时进行配置。 终结点匹配过程可以从请求的 URL 中提取值，并为请求处理提供这些值。 通过使用应用中的终结点信息，路由还能生成映射到终结点的 URL。

#### 路由基础

- 终结点

  - 通过 MapGet、MapPost 和 MapGet 等定义终结点
  - 通过匹配 URL 和 HTTP 方法来选择终结点
  - 执行请求委托并连接到路由系统。

  ```csharp
  var builder = WebApplication.CreateBuilder(args);
  var app = builder.Build();
  // 定义终结点，当匹配到该终结点时,执行操作响应并返回数据
  app.MapGet("/", () => "Hello World!");
  app.Run();
  ```

- 路由使用一对由 UseRouting 和 UseEndpoints 注册的中间件：

  - UseRouting 向中间件管道添加路由匹配。 此中间件会查看应用中定义的终结点集，并根据请求选择最佳匹配。
  - UseEndpoints 向中间件管道添加终结点执行。 它会运行与所选终结点关联的委托。
  - 应用通常不需要调用 UseRouting 或 UseEndpoints。 WebApplicationBuilder 配置中间件管道，该管道使用 UseRouting 和 UseEndpoints 包装在 Program.cs 中添加的中间件。

  ```csharp
  ** 显式调用 UseRouting 和 UseEndpoints **

  var builder = WebApplication.CreateBuilder(args);

  builder.Services.AddControllers();

  var app = builder.Build();

  app.Use(async (context, next) =>
  {
      await context.Response.WriteAsync("First Middleware Start!");
      await next.Invoke();
      await context.Response.WriteAsync("First Middleware End!");
  });
  
  app.UseRouting();

  app.UseEndpoints(endpoints =>
  {
      endpoints.MapGet("/", async context => await context.Response.WriteAsync("get"));
      endpoints.MapGet("/test", async context => await context.Response.WriteAsync("test"));
      endpoints.MapControllers();
  });

  app.Run();
  ```

#### 传统路由

- MapControllerRoute 用于创建单个路由，且该路由命名为 default 路由。调用 MapControllerRoute 可以映射传统路由控制器和属性路由控制器。
- 路由模版

  ```csharp
  app.MapControllerRoute(
      name: "default",
      pattern: "{controller=Home}/{action=Index}/{id?}");

  // MapControllerRoute 的简化，默认配置的 name 和 pattern 与上面一致
  app.MapDefaultControllerRoute()
  ```

  - 第一个路径段 {controller=Home} 映射到控制器名称。
  - 第二段 {action=Index} 映射到操作名称。
  - 第三段 {id?} 用于可选 id。 {id?} 中的 ? 使其成为可选。 id 用于映射到模型实体。

- 路由映射仅基于控制器和操作名称，不基于命名空间、源文件位置或方法参数。

#### 属性路由

- 调用 MapControllers 可以映射属性路由控制器。
  ```csharp
  var builder = WebApplication.CreateBuilder(args);
  builder.Services.AddControllers();
  var app = builder.Build();
  app.MapControllers();
  app.Run();
  ```
- 使用属性路由，通过使用 Route()属性来定义路由。
- 路由属性可应用于控制器或控制器中的操作方法上。
- 使用属性路由时，路由属性需要设置在实际使用它们的操作方法上方。
- 属性路由和传统路由相比, 提供了更多的灵活性。

##### 属性路由的使用

- 使用 Route、HTTPGet 等定义路由

```csharp
public class StudentController : Controller
{
    [HttpGet]   // 使用 HttpGet 标记该请求方法为 Get 类型
    [Route("~/")]   // 使用 ~ 重写路由前缀
    [Route("student")]
    [Route("student/{num:int?}")]  // 使用路由传参并指定为 int 类型（路由约束）
    public List<Student> GetStudentInfo(int? num)
    {
        var list = new List<Student>()
        {
            new Student() { Name = "张三", Age = 45 + num ?? 5 },
            new Student() { Name = "李四", Age = 67 },
            new Student() { Name = "王五", Age = 36 },
            new Student() { Name = "黄六", Age = 55 }
        };
        return list;
    }

    [HttpGet("details")]   // 标记请求方法，并定义路由，/details
    [Route("student/details")]
    public Student GetStudentDetails()
    {
        var student = new Student()
        {
            Name = "张三", Age = 45
        };
        return student;
    }
}
```

- 使用 action 和 controller 标记

```csharp
[Route("[controller]")]   // 使用 controller 特性标记，表示该路由的前缀是以控制器的名称开头 ，如 student
// [Route("[controller]/[action]")]  
public class StudentController : Controller
{
    [Route("[action]")]  // 使用 action 特性标记，表示已该操作的方法名进行匹配，如 /student/GetStudentName
    public string GetStudentName()
    {
        return "张三";
    }
}
```
