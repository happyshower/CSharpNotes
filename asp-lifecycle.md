### ASP .NET Core 概述

- ASP .NET Core 是一个可以用来构建现代、可伸缩和高性能的跨平台软件应用程序的通用开发框架,可用于为 Windows、Linux 和 MacOS 构建软件应用程序。
- ASP .NET Core 并不局限于单一的编程语言，它支持 C#、VB.NET、F#、XAML 和 TypeScript。
- ASP .NET Core 提供了最先进、最成熟和最广泛的类库、公共 API、多语言支持和工具。

### ASP .NET Core 6/7

- 自 ASP .NET Core 6 开始，项目的创建会默认使用最小托管模型(minimal hosting model)统一 Startup.cs 到 Program.cs ，只有单个 Program.cs 文件作为程序的入口点。其特点如下：
  - 默认使用 顶级语句 ，意味着可以没有 Program.Main() 样板
  - 隐式 using 指令 意味着不需要 using 语句。

#### 生命周期

1. 创建应用程序构建器：

   - 通过 WebApplication.CreateBuilder(args) 创建应用程序构建器。

   ```csharp
   var builder = WebApplication.CreateBuilder(args);  //创建构建器
   ```

2. 添加服务：

   - 使用构建器 WebApplicationBuilder 的 Services 属性添加所需的服务到容器中。

   ```csharp
   builder.Services.AddControllers();  //添加服务
   ```

3. 构建应用程序实例

   - 通过构建器 WebApplicationBuilder 实例的 Build() 方法构建应用程序 WebApplication 实例。

   ```csharp
   var app = builder.Build();  //创建 WebApplication 实例
   ```

4. 构建中间件管道

   - 使用 app.UseExceptionHandler() 配置全局异常处理中间件
   - 使用 app.UseStaticFiles() 启用静态文件中间件，允许提供静态资源。
   - 使用 app.UseRouting() 启用路由中间件，允许定义路由和端点，为请求提供目标处理位置。
   - 使用 app.UseAuthorization() 启用授权中间件，验证用户的身份和权限，确保安全访问。

   ```csharp
   app.UseExceptionHandler();
   app.UseStaticFiles();
   app.UseRouting();
   app.UseAuthorization();
   ```

5. 启动应用程序：

   - 使用 WebApplication 实例的 Run() 启动应用程序，监听并处理传入的 HTTP 请求。

   ```csharp
   app.Run();  //启动启动应用程序
   ```
- 完整代码示例
  ```csharp
  var builder = WebApplication.CreateBuilder(args);

  builder.Services.AddControllers();

  var app = builder.Build();

  app.UseHttpsRedirection();

  app.UseStaticFiles();

  app.UseAuthorization();

  //终结点路由中间件，默认会包装整个中间件管道
  app.MapGet("/", () => "Hello World!");

  app.Run();
  ```

### ASP .NET Core 服务生命周期

- ASP .NET Core 的依赖注入（DI）容器提供了三种生命周期：瞬时（Transient）、作用域（Scoped）和单例（Singleton）。 这些生命周期影响着服务实例的创建和销毁方式，进而影响着应用程序的性能和可靠性

  - Singleton 单例模式：创建服务类的单个实例，将其存储在内存中，并在整个应用程序中重复使用。

  ```csharp
  builder.Services.AddSingleton<IPersonService,PersonService>();
  ```

  - Scoped 作用域模式：每个请求会创建一次服务实例。参与处理单个请求的所有中间件、MVC 控制器等等，都将获得相同的实例。

  ```csharp
  builder.Services.AddScoped<IPersonService, PersonService>();
  ```

  - Transient 瞬时模式：每次请求 Transient 生命周期服务时都会创建它们。此生命周期最适合轻量级、无状态的服务。

  ```csharp
  builder.Services.AddTransient<IPersonService, PersonService>();
  ```

### Startup 与最小托管模型的结合使用

- 将 Startup 与最小托管模型结合使用具有以下优点：
  - 没有隐藏反射用于调用 Startup 类
  - 可以编写异步代码，方便开发人员控制对 Startup 的调用
  - 可以编写交错 ConfigureServices 和 Configure 的代码

```csharp
** Program.cs **

var builder = WebApplication.CreateBuilder(args);

var startup = new Startup(builder.Configuration);

startup.ConfigureServices(builder.Services);

var app = builder.Build();

startup.Configure(app, app.Environment);

app.Run();
```

```csharp
** Startup.cs **

public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddRazorPages();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        else
        {
            app.UseExceptionHandler("/Error");
            app.UseHsts();
        }

        app.UseHttpsRedirection();
        app.UseStaticFiles();
        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapRazorPages();
        });
    }
}
```
