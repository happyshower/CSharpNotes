### Entity Framework Core 框架

- Entity Framework (EF) Core 是轻量化、可扩展、开源和跨平台版的常用 Entity Framework 数据访问技术。用于程序中的 class 类和数据库中的表互相之间建立映射关系。
- EF Core 是一个对象关系映射器 ( object-relational mapper ORM)。 对象关系映射是一种技术，通过执行在应用程序编程语言中定义的对象和存储在关系数据源中的数据之间进行映射所需的工作，开发人员能够以面向对象的方式处理数据。

#### EF 工作模式

- DataBase First： 传统的表驱动方式创建 EDM（Entity Data Model），然后通过 EDM 生成模型和数据层代码。

- Code First： 手动创建实体类 Entity Class，数据层 DbContext 及映射关系去生成数据库脚本。

#### 模型映射（Model Mapping）

- 对于 EF Core，使用模型执行数据访问。 模型由实体类和表示数据库会话的上下文对象构成。
- 在上下文 DbContext 中包含操作数据库的 DbSet 属性，DbSet 是某个实体类的集合，每一个实体类的的属性都映射到表的列，每一个 DbSet 属性都对应一个表，它是对实体进行数据库操作的网关。
- EF Core 可以从/向数据库中读取和写入实体实例，如果使用的是关系数据库，EF Core 可以通过迁移为实体创建表。

  ![模型映射](https://img2023.cnblogs.com/blog/1469373/202301/1469373-20230120102437885-1933436152.png)

#### DbContext

- DbContext 是实体类和数据库之间的桥梁，DbContext 主要负责与数据交互，主要作用：
  - DBContext 包含所有实体映射到数据库的实体集 DbSet< TEntity >
  - DBContext 将 LINQ TO Entities 查询转换成 DBServer 认识的 SQL 语句
  - DBContext 跟踪实体从数据库中查询出来后发生的修改变化
  - DBContext 支持持久化数据库

初始化 DbContext 代码示例：

```csharp
** appsetting.json **
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  // 连接数据库的字符串
  "ConnectionStrings": {
    "Default": "server=localhost;database=users_db;user=root;password=12345678"
  },
  "AllowedHosts": "*"
}

** Program.cs **
// 读取json文件参数
var connection = builder.Configuration.GetValue<string>("ConnectionStrings:Default");
// 将 MyDbContext 上下文注册为服务，并为上下文配置数据库提供程序（如SQL Server、MySQL等）并连接
builder.Services.AddDbContext<MyDbContext>(options => options.UseMySQL(connection));
builder.Services.AddScoped<IUserRepository, UserRepository>();

** User.cs **
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
    public string Location { get; set; }
    public string Tel { get; set; }
}

** MyDbContext.cs **
public class MyDbContext : DbContext
{
    // 通过构造函数对 DbContextOptions 进行实例化，将应用程序的配置信息传递给 DbContext
    public MyDbContext(DbContextOptions<MyDbContext> option) : base(option)
    {
    }

    // 配置模型
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<User>().ToTable("Users");
    }

    // DbSet<User> 属性表示 User 对象的集合，并默认映射到名为 Users 的表
    public DbSet<User> Users { get; set; }
}
```

---

#### 配置模型

1. 数据注释（Data Annotations） 配置模型，数据注释会替代约定，但会被 Fluent API 配置替代。

```csharp
[Table("users")]  // 指定映射的数据库表为 Users
public class User
{
    [Key] // 主键
    public int Id { get; set; }

    [Required] // 非空声明
    [StringLength(250)]   // 字符串类型 string 的长度
    public string Name { get; set; }

    // 最小长度和最大长度
    [MinLength(6),MaxLength(12)]
    public string Email { get; set; }

    // 指定该属性映射数据库表的字段为 address
    [Column("address")]
    public string Location { get; set; }

    // 指定该属性映射数据库表的字段为 phone，且该字段类型为 varchar，长度为 50
    [Column(name:"phone",TypeName = "varchar(50)" )]
    public string Tel { get; set; }

}

[Table("animals")]
public class Animal
{
    // 组合键（将多个属性配置为实体的键）
    [Key]
    // 通过 name 设置表的字段映射， Order 指定属性作为键的先后顺序
    [Column(name:"animal_id",Order = 100)]
    // EF Code First对于int类型的主键，会自动的设置其为自动增长列
    // DatabaseGeneratedOption.None：取消自增
    // DatabaseGeneratedOption.Identity：开启自增
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int AnimalId { get; set; }
    [Key]
    [Column(Order = 200)]
    public string Name { get; set; }

    public string Location { get; set; }

    [NotMapped]  // 该属性不进行表字段的映射
    public string Kind { get; set; }

    public List<User> Zookeeper { get; set; }
}
```

2. Fluent API 配置模型

- 在派生上下文中重写 OnModelCreating 方法，并使用 Fluent API 来配置模型。 此配置方法最为有效，并可在不修改实体类的情况下指定配置。 Fluent API 配置具有最高优先级，并将替代约定和数据注释。 配置按调用方法的顺序应用，如果存在任何冲突，最新调用将替代以前指定的配置。

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // 配置非空声明
    modelBuilder.Entity<User>()
        .ToTable("Users")
        .Property(u => u.Name)
        .IsRequired();

    // 将多个属性配置为实体的键（组合键）
    modelBuilder.Entity<User>()
        .HasKey(u => new { u.Id, u.Name });

    // 将 Location 属性和表的 列 address 进行映射
    modelBuilder.Entity<User>()
      .Property(u => u.Location)
      .HasColumnName("address");

    // 该属性不进行表字段的映射
    modelBuilder.Entity<User>().Ignore(u => u.Tel);

    // 设置默认值
    modelBuilder.Entity<User>().Property(u => u.Age).HasDefaultValue(18);

    // 备用键字段的值可以唯一标识一条数据，它对应数据库的唯一约束。
    // 数据标识方式只能配置主键，使用Key特性，备用键只能通过FluentAPI进行配置
    modelBuilder.Entity<User>()
    .HasKey(u=>u.Id)    //主键
    .HasAlternateKey(u => u.IDNumber);  //备用键
}

```

---

#### 仓储 (Repository)

- 仓储模式作为领域驱动设计（Domain-Driven Design，DDD）的一部分，在系统设计中的使用非常广泛。它主要用于解除业务逻辑层与数据访问层之间的耦合，使业务逻辑层在存储、访问数据库时无须关心数据的来源及存储方式，仓储模式带来的好处是一套代码可以适用于多个类，提高代码复用。

```csharp
** 仓储基本使用示例 **

public interface IUserRepository
{
    User GetUseById(int id);
    IEnumerable<User> GetAllUser(int id);
    User Add(User user);
    User Update(User user);
    User Delete(int id);
}

public class UserRepository : IUserRepository
{
    private readonly MyDbContext _context;
    // 通过构造函数注入依赖
    public UserRepository(MyDbContext context)
    {
        _context = context;
    }

    public User GetUseById(int id)
    {
        return _context.Users.Find(id);
    }

    public IEnumerable<User> GetAllUser(int id)
    {
        return _context.Users;
    }

    public User Add(User user)
    {
        _context.Users.Add(user);
        _context.SaveChanges();
        return user;
    }

    public User Update(User user)
    {
        var newUser = _context.Users.Attach(user);
        newUser.State = EntityState.Modified;
        _context.SaveChanges();
        return user;
    }

    public User Delete(int id)
    {
        var user = _context.Users.Find(id);
        if (user != null)
        {
            _context.Users.Remove(user);
            _context.SaveChanges();
        }

        return user;
    }
}
```
