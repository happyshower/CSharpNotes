### AutoMapper

- AutoMapper 是一个对象-对象映射器。对象-对象映射的工作原理是将一种类型的输入对象转换为不同类型的输出对象。AutoMapper 提供了一些约定，以消除弄清楚如何将类型 A 映射到类型 B 的繁重工作。只要类型 B 遵循 AutoMapper 既定的约定，映射两种类型几乎需要零配置。

#### DTO

- DTO (Data Transfer Object) 数据传输对象，用于表现层和应用层之间的数据交互 。DTO 是在 PO (Persistant Object，持久化对象，与数据库结构映射的实体，数据库中的一条数据即为一个 PO 对象) 的基础上选择需要的字段的数据传输到表现层。大多数情况下，DTO 内部的数据结构来自多个表，通过 DTO 的方式，可以便捷高效从不同的实体获取并组装成新的对象。

#### 全局注册

- 使用自动扫描添加配置文件到主映射器配置中并注册服务
- 创建配置文件
- 通过构造函数注入服务
- 使用服务进行数据的映射

```csharp
** Program.cs **

AutoMapper.IConfigurationProvider config = new MapperConfiguration(cfg =>
{
    // 通过自动扫描配置文件，将配置文件添加到主映射器配置中
    cfg.AddMaps(AppDomain.CurrentDomain.GetAssemblies());
});

builder.Services.AddSingleton(config);

builder.Services.AddScoped<IMapper, Mapper>();

```

```csharp
** UserProfile.cs **

// 新建 UserProfile 继承 AutoMapper 的 Profile 类
public class UserProfile : Profile
{
    public UserProfile()
    {
        // 将源数据映射到 DTO
        CreateMap<User, UserDto>();
        // 将 DTO 映射到源数据
        CreateMap<UserDto, User>();
    }
}
```

```csharp
** HomeController.cs **

private readonly IMapper _mapper;

// 通过构造函数注入 AutoMapper
public HomeController(IMapper mapper)
{
    _mapper = mapper;
}

[Route("user")]
public UserDto GetUserInfo()
{
    var user = new User() { Name = "jojo", Age = 18, Location = "China", Tel = "233255" };
    var userDto = _mapper.Map<UserDto>(user);
    return userDto;
}
```

#### 局部注册

方式 1. 初始化 MapperConfiguration 实例时配置 DTO 映射

```csharp
** HomeController.cs **

[Route("user")]
public UserDto GetUserInfo()
{
    //创建一个 MapperConfiguration 实例并通过构造函数初始化配置
    var config = new MapperConfiguration(cfg => cfg.CreateMap<User, UserDto>());
    var mapper = config.CreateMapper();

    var user = new User() { Name = "jojo", Age = 18, Location = "China", Tel = "233255" };
    var userDto = mapper.Map<UserDto>(user);
    return userDto;
}
```

方式 2. 使用配置文件，配置 DTO 映射

```csharp
** UserProfile.cs **

// 新建 UserProfile 继承 AutoMapper 的 Profile 类
public class UserProfile : Profile
{
    public UserProfile()
    {
        // 将源数据映射到 DTO
        CreateMap<User, UserDto>();
        // 将 DTO 映射到源数据
        CreateMap<UserDto, User>();
    }
}
```

```csharp
** HomeController.cs **

[Route("user")]
public UserDto GetUserInfo()
{
    //创建一个 MapperConfiguration 实例并通过构造函数初始化配置
    var config = new MapperConfiguration(cfg => cfg.AddProfile<UserProfile>());
    var mapper = config.CreateMapper();

    var user = new User() { Name = "jojo", Age = 18, Location = "China", Tel = "233255" };
    var userDto = mapper.Map<UserDto>(user);
    return userDto;
}
```

#### 命名约定（Naming Conventions）

- SourceMemberNamingConvention 表示源类型命名规则
- DestinationMemberNamingConvention 表示目标类型命名规则

```csharp
// 通过创建实例时初始化的方式
var config = new MapperConfiguration(cfg => {
  //设置源数据和目标数据映射的命名约定，property_name -> PropertyName
  cfg.SourceMemberNamingConvention = LowerUnderscoreNamingConvention.Instance;
  cfg.DestinationMemberNamingConvention = PascalCaseNamingConvention.Instance;
});

// 通过配置文件的方式
public class UserProfile : Profile
{
    public UserProfile()
    {
        //设置源数据和目标数据映射的命名约定，property_name -> PropertyName
        SourceMemberNamingConvention = LowerUnderscoreNamingConvention.Instance;
        DestinationMemberNamingConvention = PascalCaseNamingConvention.Instance;
    }
}
```

#### 替换字符（Replacing characters）

- 在成员名称匹配期间替换源成员中的单个字符或整个单词

```csharp
public class AnimalDto
{
    public string Name { get; set; }
    public string Location { get; set; }
    public string Kind { get; set; }
    public string ZookeeperName { get; set; }
    public int Level { get; set; }
    public string Address { get; set; }
}

[Route("animal")]
public AnimalDto GetAnimalMessage()
{
    var config = new MapperConfiguration(cfg =>
    {
        cfg.CreateMap<Animal, AnimalDto>();
        // 在匹配时将源数据的属性 Location 转换为 Address
        cfg.ReplaceMemberName("Location", "Address");
    });

    var mapper = config.CreateMapper();

    var animal = new Animal() { Name = "jojo", Location = "China", Kind = "哺乳类", Zookeeper = new User() { Name = "李四" } };

    var animalDto = mapper.Map<AnimalDto>(animal);
    // animalDto：{"name":"jojo","location":"China","kind":"哺乳类","zookeeperName":"李四","level":1,"address":"China"}

    return animalDto;
}
```

#### 识别前/后缀（Recognizing pre/postfixes）

- 源/目标属性将具有常见的前/后缀，导致必须执行一堆自定义成员映射，因为名称不匹配。为了解决这个问题，可以识别前置/后缀。

```csharp
public class Animal
{
    public string SourceName { get; set; }
    public string SourceLocation { get; set; }
}

public class AnimalDto
{
    public string Name { get; set; }
    public string Location { get; set; }
}

public AnimalDto GetAnimalMessage()
{
    var config = new MapperConfiguration(cfg =>
    {
        cfg.RecognizePrefixes("Source");
        cfg.CreateMap<Animal, AnimalDto>();
    });
    var mapper = config.CreateMapper();

    var animal = new Animal() { SourceName = "jojo", SourceLocation = "China" };

    var animalDto = mapper.Map<AnimalDto>(animal);
    // animalDto：{"name":"jojo","location":"China"}

    return animalDto;
}
```

#### 配置可见性（Configuring visibility）

- 默认情况下，AutoMapper 仅识别公共成员。它可以映射到私有设置器，但如果整个属性是私有/内部的，则将跳过内部/私有方法和属性。要指示 AutoMapper 识别具有其他可见性的成员，请覆盖默认过滤器 ShouldMapField 和/或 ShouldMapProperty 。

```csharp
public class Animal
{
    private string Name { get; set; } = "jojo";
}
public class AnimalDto
{
    public string Name { get; set; }
}

public AnimalDto GetAnimalMessage()
{
    var config = new MapperConfiguration(cfg =>
    {
        cfg.CreateMap<Animal, AnimalDto>();
        // 当属性为公有、私有时都可以映射
        cfg.ShouldMapProperty = p => p.GetMethod.IsPublic || p.GetMethod.IsPrivate || p.GetMethod.IsAssembly;
    });
    var mapper = config.CreateMapper();

    var animal = new Animal();
    var animalDto = mapper.Map<AnimalDto>(animal);
    // animalDto：{"name":"jojo"}
    return animalDto;
}
```

#### 常用映射方式

1. 扁平化映射（Flattening）
   - 名称相同的属性进行映射，不区分大小写。
   - 带 Get 前缀的方法进行映射，映射器会把 Get 和 Property 进行分割，取属性名 Property，将属性进行匹配映射
   - 目标类型属性分割，例子中: 映射器会把 AnimalDto 中的 ZookeeperName 分割成 Zookeeper、Name。然后在 Animal 中去 User 类属性中查找 Name 的属性。

```csharp
public class Animal
{
    public string Name { get; set; }
    // 将 User 类型的属性 Zookeeper 结合 User 的属性 Name 与 AnimalDto 的 ZookeeperName 进行映射
    public User Zookeeper { get; set; }
    // 带 Get 前缀的方法与 AnimalDto 的 Level 进行映射
    public int GetLevel()
    {
        return 1;
    }
}

public class AnimalDto
{
    public string Name { get; set; }
    public string ZookeeperName { get; set; }
    public int Level { get; set; }
}

[Route("animal")]
public AnimalDto GetAnimalMessage()
{
    // 初始化时
    var config = new MapperConfiguration(cfg => cfg.CreateMap<Animal, AnimalDto>());

    var mapper = config.CreateMapper();

    var animal = new Animal(){ Name = "jojo", Zookeeper = new User() { Name = "李四" } };

    var animalDto = mapper.Map<AnimalDto>(animal);
    // animalDto：{"name":"jojo","zookeeperName":"李四","level":1}

    return animalDto;
}
```

2. 自定义映射(Projection)
   - ForMember 方法允许我们指定两个个 action 委托去配置每个成员的映射关系。

```csharp
public class User
{
 public string Name { get; set; }
 public string Tel { get; set; }
}

public class UserDto
{
    public string Name { get; set; }
    public string Phone { get; set; }
}

[Route("user")]
public UserDto GetUserInfo()
{
   // 初始化时配置自定义映射
   var config = new MapperConfiguration(cfg => cfg.CreateMap<User, UserDto>().
         ForMember(dest=>dest.phone,opt=>opt.MapFrom(src=>src.Tel)));

   // 通过配置文件的方式
   // var config = new MapperConfiguration(cfg => cfg.AddProfile<UserProfile>());

   var mapper = config.CreateMapper();

   var user = new User() { Name = "jojo", Tel = "233255" };

   var userDto = mapper.Map<UserDto>(user);
   // userDto：{"name":"jojo","phone":"233255"}

   return userDto;
}

// 通过配置文件配置自定义映射
public class UserProfile : Profile
{
   public UserProfile()
   {
       // 将源数据映射到 DTO
       CreateMap<User, UserDto>()
           .ForMember
           (
               // 将 源数据的属性转换为目标数据的属性
               dest => dest.Phone,  // 目标数据的 Phone
               opt =>opt.MapFrom(src => src.Tel)  //源数据的 Tel
           );

       //将 DTO 映射到源数据
       CreateMap<UserDto, User>();
   }
}
```

#### 配置验证

- 手工制作的映射代码虽然繁琐，但具有可测试的优点。 AutoMapper 不仅消除了自定义映射代码，也消除了手动测试的需要。由于从源到目标的映射是基于约定的，因此仍然需要测试配置。
- AssertConfigurationIsValid 执行验证期间，AutoMapper 会检查每个目标类型的属性，逐一去匹配源中是否存在合适相等的类型。

```csharp
public class Source
{
    public int SomeValue { get; set; }
}

public class Destination
{
    public int SomeValueDiff { get; set; }
}

var config = new MapperConfiguration(cfg =>
  cfg.CreateMap<Source, Destination>());
// 通过 AssertConfigurationIsValid 方法去测试配置项
config.AssertConfigurationIsValid();
```

- AssertConfigurationIsValid 执行验证期间，存在不相等的类型时，会抛出一个 AutoMapperConfigurationException 异常，此异常处理(Overriding configuration errors)方式有：

  - 自定义值解析器
  - 指定字段映射(Projection)
  - 使用忽略(Ignore())选项

  ```csharp
  cfg.CreateMap<Source, Destination>()
    .ForMember(dest => dest.SomeValuefff, opt => opt.Ignore());
  ```
