### ASP .Net Core 中间件

- 中间件是一种装配到应用管道以处理请求和响应的软件。 每个组件选择是否将请求传递到管道中的下一个组件，可在管道中的下一个组件前后执行工作。
- 请求管道中的每个中间件组件负责调用管道中的下一个组件，或使管道短路。 当中间件短路时，它被称为“终端中间件”，因为它阻止中间件进一步处理请求。

#### 管道

- 针对于客户端发起 Http 请求来到服务器以后，服务器中经过每个环节的处理，把处理的结果写入到 Response 中，最后响应给客户端。 多个处理的环节就好似一根管子一样。按照每个环节去处理，这个处理的环节称为管道。

#### 使用 WebApplication 创建中间件管道

![中间件管道图]([image.png](https://learn.microsoft.com/zh-cn/aspnet/core/fundamentals/middleware/index/_static/request-delegate-pipeline.png?view=aspnetcore-7.0))

- ASP .NET Core 请求管道包含一系列请求委托，依次调用。每一个中间件都会被执行两次，在下一个中间件执行之前和之后各执行一次，分别是在处理请求和处理响应，当执行到终端中间件时，不会再执行后面的中间件，所以执行到它管道就会回转。

#### 中间件注册（扩展）

- Use 方法

  - Use 可以将多个中间件连接在一起，实现在下一个中间件的前后执行操作，Use 方法也可以使管道短路，即不调用 next 请求委托,此时和 Run 的作用一样。

  ```csharp
  var builder = WebApplication.CreateBuilder(args);
  var app = builder.Build();

  app.Use(async (context, next) =>
  {
      await context.Response.WriteAsync("First Middleware Start !");
      //调用下一个中间件
      await next.Invoke();
      await context.Response.WriteAsync("First Middleware End !");
  });

  app.Use(async (context, next) =>
  {
      await context.Response.WriteAsync("Second Middleware Start !");
      //调用下一个中间件
      await next.Invoke();
      await context.Response.WriteAsync("Second Middleware End !");
  });

  //终端中间件
  app.Run(async context =>
  {
      await context.Response.WriteAsync("Hello Terminal Middleware !");
  });

  app.MapGet("/", () => "Hello World!");

  app.Run();

  // 输出顺序：
  // First Middleware Start !
  // Second Middleware Start !
  // Hello Terminal Middleware !
  // Second Middleware End !
  // First Middleware End !
  ```

- Run 方法

  - Run 该方法为管道添加一个中间件，并标识该中间件为管道终点，称为终端中间件。即该中间件就是管道的末尾，在该中间件之后注册的中间件将永远都不会被执行。

  ```csharp
  var builder = WebApplication.CreateBuilder(args);
  var app = builder.Build();

  //通过 Run 方法向管道添加中间件
  app.Run(async context =>
  {
      await context.Response.WriteAsync("Hello Terminal Middleware !");
  });

  //由于上面已经执行了终端中间件，该中间件不再执行
  app.Run(async context =>
  {
      await context.Response.WriteAsync("After Terminal Middleware   !");
  });

  app.MapGet("/", () => "Hello World!");

  app.Run();
  ```

- Map 方法

  - 一旦进入了管道分支，则不会再回到主管道。
  - 用该方法时，会将匹配的路径从 HttpRequest.Path 中删除，并将其追加到 HttpRequest.PathBase 中
  - 路径必须以“/”开头，且不能只有一个'/'字符
  - 支持同时匹配多个段，支持内部嵌套 Use、Run 方法

  ```csharp
  var builder = WebApplication.CreateBuilder(args);
  var app = builder.Build();

  // 创建管道分支，访问 /get 或 /get/xxx 时会进入该管道分支
  app.Map("/get", app =>
  {
      // 为该管道分支注册中间件
      app.Use(async (context, next) =>
      {
          Console.WriteLine("Map Get: Use Start !");
          Console.WriteLine($"Request Path: {context.Request.Path}");
          Console.WriteLine($"Request PathBase: {context.Request.PathBase}");

          await next.Invoke();
      });
      // 为该管道分支注册终端中间件
      app.Run(async context =>
      {
          Console.WriteLine("Map Get: Run");

          await context.Response.WriteAsync("Map Get Message");
      });

  });

  app.MapGet("/", () => "Hello World!");
  app.Run();

  // 输出：
  // Map Get: Use Start !
  // Request Path:
  // Request PathBase: /get
  // Map Get: Run

  ```

#### 中间件执行顺序

- 中间件的执行顺序是根据代码逻辑执行的顺序来决定了中间件顺序处理请求和逆序处理响应。 此顺序对于安全性、性能和功能至关重要。
  - ExceptionHandler： 异常捕捉
  - HSTS (HTTP Strict Transport Security) ：HTTP 严格传输安全协议
  - HttpsRedirection：HTTPS 重定向
  - Static Files：静态资源
  - Authentication：身份验证
  - Authorization：授权
