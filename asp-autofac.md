### AutoFac 框架

- AutoFac 是一个用于 .NET 平台的开源依赖注入（Dependency Injection）容器框架。依赖注入是一种设计模式，旨在降低模块之间的耦合，通过在应用程序中动态地注入依赖项，实现对象之间的解耦。
- AutoFac 提供了一种简单而灵活的方法来管理应用程序中的对象依赖关系。它通过使用反射和配置来实例化并注入对象。AutoFac 允许定义和配置对象之间的依赖关系，并在需要的时候自动解析和创建它们。
- AutoFac 支持各种不同的注入方式，包括构造函数注入、属性注入和方法注入。它还提供了可以根据不同条件选择不同实现的功能，称为条件注册。此外，AutoFac 还支持生命周期管理，可以控制对象的创建和销毁时机。

#### 依赖注入（构造函数）

1. 在 Program.cs 中，通过 RegisterType 或 RegisterModule 注册服务。

```csharp
builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory());
builder.Host.ConfigureContainer<ContainerBuilder>(build =>
{
    // 1. 注册服务，默认使用瞬时（Transient）模式
    build.RegisterType<StudentService>().As<IPersonService>();
    // 单例模式
    build.RegisterType<StudentService>().As<IPersonService>().SingleInstance();
    // 作用域模式
    build.RegisterType<StudentService>().As<IPersonService>().InstancePerLifetimeScope();

    // 2. 通过自定义模块组装实现注册
    build.RegisterModule(new CustomRegisterModule());
});
```

2. 继承 AutoFac.Module 类，重写 Load 方法，实现自定义模块组装类 CustomRegisterModule。

```csharp
public class CustomRegisterModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        builder.RegisterType<StudentService>().As<IPersonService>();
    }
}
```

3. 在构造函数中注入依赖

```csharp
public class InjectionTestController : ControllerBase
{
    private readonly IPersonService _person;

    public InjectionTestController(IPersonService person)
    {
        _person = person;
    }

    [HttpGet("~/test")]
    public string GetMessage()
    {
        return _person.GetIdentity();
    }
}
```

#### 批量注入

1. 继承 AutoFac.Module 类，重写 Load 方法，实现自定义模块组装类 CustomRegisterModule。
2. 获取当前执行的程序集并注册程序集中的所有 具体类 。

```csharp
public class CustomRegisterModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        // 获取当前正在执行的代码所在的程序集
        var dataAccess = Assembly.GetExecutingAssembly();

        // 默认情况下，程序集中的所有 具体类 都将被注册
        builder.RegisterAssemblyTypes(dataAccess)
            // 根据名称约定（服务层的接口和实现均以Service结尾），实现服务接口和服务实现的依赖
            .Where(t => t.Name.EndsWith("Service"))
            // 将该类型注册为为所有公共接口提供的服务
            .AsImplementedInterfaces();
    }
}
```

#### 属性注入

1. 替换从 IServiceProvider 中解析控制器实例的所有者，这是因为控制器本身的实例（以及它的处理）是由框架创建和拥有的，而不是由容器所有。
2. 在 ContainerBuilder 中通过注册控制器，并使用属性注入功能实现.
3. 注册组件方法，并使用属性注入 PropertiesAutowired() 标注。
4. 在控制器中使用属性来接收, 其中注入属性必须标注为 public。

```csharp
** Program.cs **

var builder = WebApplication.CreateBuilder(args);

// 替换控制器的所有者（ 必须在 AddControllers 之前 ）
builder.Services.Replace(ServiceDescriptor.Transient<IControllerActivator, ServiceBasedControllerActivator>());

builder.Services.AddControllers();

builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory());

builder.Host.ConfigureContainer<ContainerBuilder>(build =>
{
    // 找到所有的 controller 进行注册，并使用属性注入功能
    var controllerTypesInAssembly = typeof(Program).Assembly.GetExportedTypes()
        .Where(type => typeof(ControllerBase).IsAssignableFrom(type)).ToArray();

    build.RegisterTypes(controllerTypesInAssembly).PropertiesAutowired();

    // 注册组件并开启属性注入
    build.RegisterType<StudentService>().As<IPersonService>().InstancePerLifetimeScope().PropertiesAutowired();
});

var app = builder.Build();

app.MapControllers();

app.Run();

```

```csharp
** Controller.cs **
public class InjectionTestController : ControllerBase
{
    // 注入的属性必须是公有的
    public IPersonService PersonService { get; set; }

    [HttpGet("~/test")]
    public string GetMessage()
    {
        return PersonService.GetIdentity();
    }
}
```
