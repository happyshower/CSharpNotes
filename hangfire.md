### Hangfire 框架

- Hangfire 是一个开源的 .NET 任务调度框架，提供了内置集成化的控制台，可以直观明了的查看作业调度情况，并且 Hangfire 不需要依赖于单独的应用程序执行，并且支持持久性存储。
- Hangfire 控制面板不仅提供监控，也可以手动的触发执行定时任务且不影响下次定时任务的执行。

#### 基本构成

- 客户端：负责创建后台作业并将其保存到存储中，后台作业是应该在当前执行上下文之外执行的工作单元。
- 作业存储：存储是 Hangfire 保存与后台作业处理相关的所有信息的地方。所有细节（如类型、方法名称、参数等）都被序列化并放入存储中，没有数据保存在进程的内存中。
- 服务器：通过 Hangfire Server（一组非线程池的后台线程）来侦听存储中的新后台作业，从作业存储中取出任务并通过反序列化类型、方法和参数来执行新作业。

  ![Alt text](https://static.bookstack.cn/projects/Hangfire-zh-official/153a9d5156afd2e9.png)

#### 作业类型

- **Fire-and-forget Jobs**：自助式作业，创建后执行（在执行一系列的序列化和存储之后），仅执行一次。使用 BackgroundJob 类的静态方法 Enqueue 创建作业。

```csharp
BackgroundJob.Enqueue(() => Console.WriteLine("This is a Fire-and-forget Job!"));
```

- **Delay Jobs**：延迟作业，在指定的时间后执行，仅执行一次。使用 BackgroundJob 类的静态方法 Schedule 创建作业，并传入延迟配置。

```csharp
BackgroundJob.Schedule<HangfireMessage>(h => h.ShowDelayMessage(), TimeSpan.FromSeconds(10));
// TimeSpan.FromSeconds(10) 10秒后执行
// TimeSpan.FromMinutes(10) 10分钟后执行
// TimeSpan.FromHours(10) 10小时后执行
// TimeSpan.FromDays(1) 1天后执行
```

- **Recurring Jobs**：周期作业，执行多次。使用 RecurringJob 类的静态方法 AddOrUpdate 创建作业，并传入指定的周期执行表达式 Crontab Expression 。

```csharp
global::Hangfire.RecurringJob.AddOrUpdate<HangfireMessage>(h => h.ShowRecurringMessage(), "0/10 * * * * ?",
    new RecurringJobOptions()
    {
        TimeZone = TimeZoneInfo.FindSystemTimeZoneById("China Standard Time")
    });
```

#### 时间表达式 Crontab Expression

- '\*'：代表任意的数值都可以满足。\* 指的是任意，如果放入 seconds 这个字段，则代表每一秒都符合。
- '?'：只存在于 Day of month、Day of week 域中，它出现所代表的意思就是本字段没有指定值。Week 和 Day 存在互斥关系，不可以同时设定规则。
- ’-‘：这个符号则是表达一个范围，其中范围的开始和结束是从左往右读取的。

![Alt text](https://img2018.cnblogs.com/blog/1626261/202001/1626261-20200120163902333-1184627888.png)

#### Hangfire Dashboard 授权

- Hangfire 可以显示一个 Dashboard 页面来查看所有后台工作的实时状态。默认该页面对所有用户都是可用的，没有权限限制。使用 HangFire 包中的 IDashboardAuthorizationFilter 接口实现自定义 CustomDashboardAuthorizationFilter 类，可以集成它到权限认证系统。

```csharp
app.UseAuthentication();
app.UseAuthorization();
// 设置仪表盘的访问路径和授权配置，且应该在授权中间件之后被调用，否则身份认证将会失败
app.UseHangfireDashboard("/hangfire", new Hangfire.DashboardOptions
{
    Authorization = new[] { new CustomDashboardAuthorizationFilter() }
});

// 自定义面板授权
public class CustomDashboardAuthorizationFilter : IDashboardAuthorizationFilter
{
    public bool Authorize([NotNull] DashboardContext context)
    {
    }
}
```

#### 使用流程

1. 安装需要的包

   - Hangfire
   - Hangfire.MySqlStorage

2. 创建数据库或使用现有数据库并配置数据库连接（ MySql ）

```csharp
** appsetting.json **

"ConnectionStrings": {
  "Default": "server=localhost;database=users_db;user=root;password=12345678",
  "HangfireConnection": "server=localhost;database=hangfire_db;user=root;password=12345678;allow user variables=true"
},
```

3. 注册 HangFire 服务，使用 Hangfire 中间件（如仪表板 UI）并开启作业。

```csharp
** Program.cs **
// 注册 hangfire 服务
builder.Services.AddHangFireService(builder.Configuration);
// 中间件
app.UseHangfireServer();  // 启动 hangfire 服务
app.UseHangfireDashboard("/hangfire");  // 启动 hangfire 仪表盘
// 开始作业
HangfireTestMethod.BgJobEnqueue();
HangfireTestMethod.BgJobDelay();
HangfireTestMethod.RecurringJob();
```

4. 自定义 Hangfire 类，配置 Hangfire 存储介质（数据库）以及存储介质的配置等

```csharp
public static class HangFireExtension
{
    public static void AddHangFireService(this IServiceCollection services, IConfiguration configuration)
    {
        services.AddHangfire(config => config
            .SetDataCompatibilityLevel(CompatibilityLevel.Version_170)
            .UseSimpleAssemblyNameTypeSerializer()
            .UseRecommendedSerializerSettings()
            .UseStorage(
                new MySqlStorage(
                    configuration.GetConnectionString("HangfireConnection"),
                    new MySqlStorageOptions
                    {
                        QueuePollInterval = TimeSpan.FromSeconds(15), // 作业队列轮询间隔。默认值为15秒
                        JobExpirationCheckInterval = TimeSpan.FromHours(1), // 作业到期检查间隔（管理过期记录）。默认值为1小时
                        CountersAggregateInterval = TimeSpan.FromMinutes(5),  // 聚合计数器的间隔。默认为5分钟
                        PrepareSchemaIfNecessary = true, // 如果设置为true，则创建数据库表。默认是true
                        DashboardJobListLimit = 50000,  // 仪表板作业列表限制。默认值为50000。
                        TransactionTimeout = TimeSpan.FromMinutes(1), // 交易超时。默认为1分钟
                        TablesPrefix = "Hangfire",  // 数据库中表的前缀。默认为none
                    }
                )
            ));

        services.AddHangfireServer(options => options.WorkerCount = 1);
    }
}
```

5. 定义作业的类型

```csharp
public static class HangfireTestMethod
{
    public static void BgJobEnqueue()
    {
        BackgroundJob.Enqueue(() => Console.WriteLine("This is a Fire-and-forget Job!"));
    }

    public static void BgJobDelay()
    {
        BackgroundJob.Schedule<HangfireMessage>(h => h.ShowDelayMessage(), TimeSpan.FromSeconds(10));
    }

    public static void RecurringJob()
    {
        global::Hangfire.RecurringJob.AddOrUpdate<HangfireMessage>(h => h.ShowRecurringMessage(), "0/10 * * * * ?",
            new RecurringJobOptions()
            {
                TimeZone = TimeZoneInfo.FindSystemTimeZoneById("China Standard Time")
            });
    }
}

public class HangfireMessage
{
    public void ShowDelayMessage()
    {
        Console.WriteLine($"This is HangfireMessage class and this method is a Delay Job,now is {DateTime.Now}");
    }

    public void ShowRecurringMessage()
    {
        Console.WriteLine($"This is HangfireMessage class and this method is a Recurring Job,now is {DateTime.Now}");
    }
}
```
