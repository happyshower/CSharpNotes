### DbUp 库

- DbUp 是一个开源 .NET 库，用于数据库版本管理和迁移，可以将更改部署到数据库。主要用于保持数据库的一致性，确保数据库的结构与应用程序的要求保持一致。
- DbUp 可以将数据库升级的过程自动化，避免手动执行 SQL 脚本的繁琐和出错风险。

#### 常用方法

- GetScriptsToExecute()：获取将要执行的脚本
- GetExecutedScripts()： 获取已执行的脚本
- IsUpgradeRequired()：检查是否需要升级
- MarkAsExecuted()：为任何新的迁移脚本创建版本记录而不执行它们
- TryConnect()：尝试连接数据库
- PerformUpgrade()：执行数据库升级
- LogScriptOutput()：日志脚本输出

#### DbUp 使用流程

1. 安装所需的包

   - dbup-core
   - dbup-mysql

2. 创建一个 **.sql** 文件，如 Script_initial_tables.sql

```csharp
create table if not exists users
(
    id int auto_increment
    primary key,
    name varchar(50) not null,
    age int not null,
    location varchar(50) not null,
    tel varchar(50) not null
)
charset=utf8mb4;

create table if not exists products
(
    id int auto_increment
    primary key,
    name varchar(50) not null,
    price double not null
)
charset=utf8mb4;
```

3. 在 **.csproj** 文件中配置嵌入资源

```csharp
// 写入脚本的路径,嵌入数据库脚本资源
<ItemGroup>
    <EmbeddedResource Include="Dbup\Scripts\Script_initial_tables.sql" />
</ItemGroup>
```

4. 使用 DbUp 连接数据库，在启动应用时扫描程序集的脚本资源并执行更新

```csharp
// 获取 appsettings.json 中的数据库连接字符串
var connection = builder.Configuration.GetValue<string>("ConnectionStrings:Default");
// 传入连接字符串，并开启 Dbup
new DbUpRunner(connection).Run();

public class DbUpRunner
{
    private readonly string _connectionString;

    public DbUpRunner(string connectionString)
    {
        _connectionString = connectionString;
    }

    public void Run()
    {
        // 检查目标数据库是否存在，如果不存在，它将创建数据库
        EnsureDatabase.For.MySqlDatabase(_connectionString);

        // DeployChanges.To 是入口点
        var upgradeEngine = DeployChanges.To.MySqlDatabase(_connectionString)
            .WithScriptsEmbeddedInAssembly(typeof(DbUpRunner).Assembly)  // 扫描当前程序集的脚本嵌入资源并执行脚本
            .WithTransaction()  // 开启事务
            .LogToAutodetectedLog()
            .LogToConsole()  // 输出日志到控制台
            .Build();

        // 更新数据库
        var result = upgradeEngine.PerformUpgrade();

        if (!result.Successful)
            throw result.Error;
        {
            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine("Success!");
            Console.ResetColor();
        }
    }
}
```

#### 补充

- DbUp 使用数据库中的表来记录已运行的迁移。
- DbUp 的默认表是 SchemaVersion，它由 Id、ScriptName 和 Applied 列组成。
- 重命名已在数据库上执行的脚本，DbUp 会将其视为不同的脚本并再次执行它。
- 当应用初次执行后，再修改改脚本的内容，如删除字段，增加表等操作，再次运行该应用程序，它不会再执行该脚本，因为数据库的 SchemaVersion 表中已有该脚本的迁移记录，需要删除该表的同名脚本记录后再重新启动应用程序即可生效。
