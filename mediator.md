### Mediator .Net 框架

- Mediator .Net 是一个基于中介者模式的开源框架，用于发送命令、发布事件和请求响应，并支持管道，通过中介者对象实现了 controller 层与 service 层的解藕。

#### 中介者模式（Mediator Pattern）

- 中介者模式是一种行为型模式，用一个中介对象来封装一系列对象的交互，从而把一批原来可能是交互关系复杂的对象转换成一组松散耦合的中间对象，以有利于维护和修改。
- 中介者模式分离了类和类之间的耦合，简化了对象协议，中央控制对象交互，从而使个体对象变得更容易且更简单，它不需要传递数据给其他个体对象，仅需要传给中介者即可。个体对象不需要具有处理内部交流的逻辑，则更加突出它的面向对象特性。

#### 管道

- 全局接收管道：每当消息在到达下一个管道和处理程序之前发送、发布或请求时，都会触发此管道

- 命令接收管道：该管道将​在到达其命令处理程序之后和之前触发 GlobalReceivePipeline，该管道仅用于 ICommand

- 事件接收管道：该管道将​​在到达其事件处理程序之后和之前触发 GlobalReceivePipeline，该管道仅用于 IEvent

- 请求接收管道：该管道将在到达其请求处理程序之后和之前触发 GlobalReceivePipeline，该管道仅用于 IRequest

- 发布管道：当在处理程序中发布 an 时，将触发此管道 IEvent，此管道仅用于 IEvent 且通常用作传出拦截器
  ![mediator管道](https://cloud.githubusercontent.com/assets/3387099/21959127/9a065420-db09-11e6-8dbc-ca0069894e1c.png)

  注：管道最强大的功能是可以添加任意数量的中间件。

#### Mediator 的使用

- 注册 Mediator 服务。Autofac 的 ContainerBuilder 扩展方法 RegisterMediator 用于注册构建器。（安装的包是 Mediator.Net.Autofac ）
- Controller 发送信息，底层使用 SendMessage 来判断传来的 IMessage 是属于 command/request/event 中的哪一种类型
- 根据发送类型执行对应的 Handle，中介者 mediator 会将 ReceiveContext 传递给处理程序，并且它有一个属性 Message ，该属性是发送的原始消息。
- 根据发布的的事件 event 执行对应的事件处理程序 EventHandler

1. 注册 Mediator 服务

```csharp
builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory());
builder.Host.ConfigureContainer<ContainerBuilder>(build =>
{

    var mediaBuilder = new MediatorBuilder();
    // 注册 Program 所在程序集的所有 Handler
    mediaBuilder.RegisterHandlers(typeof(Program).Assembly).Build();
    // 注册 Mediator 服务
    build.RegisterMediator(mediaBuilder);

    build.RegisterType<UserRepository>().As<IUserRepository>().InstancePerLifetimeScope();
});
```

2. 在 Controller 中 注入 Mediator 依赖，并发对应的信息（message contract）

```csharp
// ICommand 和 IResponse 都继承自 Mediator.Net.Contracts.IMessage
public class CreateUserCommand : ICommand
{
    public User Student { get; set; }
}

public class CreateUserResponse : IResponse
{
    public string Result { get; set; }
}

[ApiController]
[Route("api/[controller]/[action]")]
public class MediatorController : ControllerBase
{
    private readonly IMediator _mediator;

    public MediatorController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost]
    [Route("create")]
    public async Task<IActionResult> CreateAsync([FromBody] CreateUserCommand command)
    {
        var response = await _mediator.SendAsync<CreateUserCommand, CreateUserResponse>(command)
            .ConfigureAwait(false);

        return Ok(response);
    }
}
```

3. 根据发送请求的类型，选择对应的管道，并执行相对应的 Handle 。如发送带有响应的请求 SendAsync<CreateUserCommand, CreateUserResponse>，会执行 ICommandHandler<CreateUserCommand,CreateUserResponse> 类型的 Handle。

```csharp
public class CreateUserCommandHandler : ICommandHandler<CreateUserCommand,CreateUserResponse>
{
    private readonly IUserRepository _userRepository;

    public CreateUserCommandHandler(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }

    public async Task<CreateUserResponse> Handle(IReceiveContext<CreateUserCommand> context, CancellationToken cancellationToken)
    {
        // 返回一个事件
        var @event = await _userRepository.AddUserAsync(context.Message, cancellationToken).ConfigureAwait(false);

        // 将返回的事件发布出去，触发对应的 EventHandler 处理程序
        await context.PublishAsync(@event, cancellationToken).ConfigureAwait(false);

        return new CreateUserResponse
        {
            Result = @event.Result
        };
    }
}
```

4. 根据发布的事件触发对应的事件处理程序 EventHandler，如 UserCreatedEvent 事件 触发 IEventHandler< UserCreatedEvent > 类型的处理程序。

```csharp
public class UserCreatedEvent : IEvent
{
    public string Result { get; set; }
}

public class UserCreatedEventHandler : IEventHandler<UserCreatedEvent>
{
    public async Task Handle(IReceiveContext<UserCreatedEvent> context, CancellationToken cancellationToken)
    {
        // Task.CompletedTask 返回一个已经完成的 Task 对象
        // await Task.CompletedTask 不会再去线程池启动一个新的线程来执行 await 关键字之后的代码，之前和之后的代码是在同一个线程上同步执行的
        await Task.CompletedTask;
    }
}
```

5. 通过 MyDbContext 操作数据库表，并返回一个 UserCreatedEvent 对象。

```csharp
public class UserRepository : IUserRepository
{
    private readonly MyDbContext _context;

    public UserRepository(MyDbContext context)
    {
        _context = context;
    }

    public async Task<UserCreatedEvent> AddUserAsync(CreateUserCommand command, CancellationToken cancellationToken)
    {
        return new UserCreatedEvent
        {
            Result = await CreateAsync(command.Student, cancellationToken).ConfigureAwait(false) > 0
                ? "数据写入成功"
                : "数据写入失败"
        };
    }

    public async Task<int> CreateAsync(User user, CancellationToken cancellationToken)
    {
        await _context.AddAsync(user, cancellationToken).ConfigureAwait(false);
        return await _context.SaveChangesAsync(cancellationToken).ConfigureAwait(false);
    }
}
```
