### C# 特性 Attribute

- 特性（Attribute）是用于在运行时传递程序中各种元素（比如类、方法、结构、枚举、组件等）的行为信息的声明性标签。您可以通过使用特性向程序添加声明性信息。一个声明性标签是通过放置在它所应用的元素前面的方括号（[ ]）来描述的。
- 特性（Attribute）用于添加元数据，如编译器指令和注释、描述、方法、类等其他信息。.Net 框架提供了两种类型的特性：预定义特性和自定义特性。

  #### 预定义特性

  - AttributeUsage：描述了如何使用一个自定义特性类，规定了特性可应用到的项目的类型。
    ```csharp
    ** 特性语法 **
    [AttributeUsage(validon,AllowMultiple=allowmultiple,Inherited=inherited)]
    ```
    - 参数 validon 规定特性可被放置的语言元素。它是枚举器 AttributeTargets 的值的组合。默认值是 AttributeTargets.All。
    - 参数 allowmultiple（可选的）为该特性的 AllowMultiple 属性（property）提供一个布尔值。如果为 true，则该特性是多用的。默认值是 false（单用的）。
    - 参数 inherited（可选的）为该特性的 Inherited 属性（property）提供一个布尔值。如果为 true，则该特性可被派生类继承。默认值是 false（不被继承）
    ```csharp
    ** 示例 **
    [AttributeUsage(AttributeTargets.Class | AttributeTargets.Constructor |
    AttributeTargets.Field | AttributeTargets.Method | AttributeTargets.Property,
    AllowMultiple = true)]
    ```
  - Conditional：标记了一个条件方法，其执行依赖于指定的预处理标识符，它会引起方法调用的条件编译，取决于指定的值，比如 Debug 或 Trace。
    ```csharp
    ** 特性语法 **
    [Conditional(conditionalSymbol)]
    ```
  - Obsolete：这个预定义特性标记了不应被使用的程序实体。它可以让您通知编译器丢弃某个特定的目标元素。
    ```csharp
    ** 特性语法 **
    [Obsolete(message)]
    [Obsolete(message,iserror)]
    ```

  #### 自定义特性

  - 创建并使用自定义特性步骤：

    - 声明自定义特性
    - 构建自定义特性
    - 在目标程序元素上应用自定义特性
    - 通过反射访问特性

    ```csharp
    // 声明自定义特性
    [AttributeUsage(AttributeTargets.Property | AttributeTargets.Method)]

    // 构建自定义特性
    class PractiseAttribute : Attribute  //继承 System.Attribute 特性类
    {
        public string Description { get; }

        public PractiseAttribute(string description)
        {
            Description = description;
        }
    }

    // 应用自定义特性
    class PractiseDemoClass
    {
        [Practise("this is a property")]   //使用特性
        public string Message { get; }

        [Practise("this is a method")]   //使用特性
        public void GetMessage()
        {
            Console.WriteLine("Executing PractiseDemoClass" + Message);
        }
    }
    ```

### C# 反射 Reflection

- 反射允许在运行时分析、检查和操作程序集、类型、对象等信息。
- 使用反射可以用访问到 程序集 模块 类型 与及类型上面的一些元信息 Attribute, 这些统称为元数据（metadata）。

  ##### 使用反射获取类型

  ```csharp
  public static void Main(string[] args)
  {
      //通过 typeof 获取某个值的类型
      Type practiseType = typeof(PractiseDemoClass);
      Console.WriteLine(practiseType);  //Practise4.PractiseDemoClass
      //获取数据类型名
      Console.WriteLine(practiseType.Name); //PractiseDemoClass
      //获取数据类型的完全限定名（包括命名空间）
      Console.WriteLine(practiseType.FullName); //Practise4.PractiseDemoClass
      //获取数据类型的命名空间名
      Console.WriteLine(practiseType.Namespace); //Practise4
      //获取数据类型的父类类型
      Console.WriteLine(practiseType.BaseType);  //System.Object

      //通过一个实例对象获取该对象所对应的类的类型
      PractiseDemoClass demo = new PractiseDemoClass();
      Type demoType = demo.GetType();
      Console.WriteLine(demoType); //Practise4.PractiseDemoClass

      //通过名称字符串获取类型：”命名空间.类名“
      Type strType = Type.GetType("Practise4.PractiseDemoClass");
      Console.WriteLine(strType); //Practise4.PractiseDemoClass
  }
  ```

  ##### Type 中其他常用的方法

  - GetMember、GetMembers：返回 MemberInfo 类型，用于取得该类的所有成员的信息
  - GetMethod、GetMethods：返回 MethodInfo 类型，用于取得该类的方法的信息
  - GetProperty、GetProperties：返回 PropertyInfo 类型，用于取得该类的属性的信息
  - GetField()、GetFields()：返回 FieldInfo 类型，用于取得该类的字段（成员变量）的信息

  ##### 使用反射获取自定义特性

  ```csharp
  using System.Reflection;

  namespace Practise4
  {
      internal class PractiseDemo4
      {
          public static void Main(string[] args)
          {

              // 使用反射获取属性上的特性
              PropertyInfo messageProperty = typeof(PractiseDemoClass).GetProperty("Message");  //使用反射获取 PractiseDemoClass 类的 Message属性信息
              object[] messageAttributes = messageProperty.GetCustomAttributes(typeof(PractiseAttribute), false); //获取属性上的特性
              if (messageAttributes.Length > 0)
              {
                  PractiseAttribute methodAttribute = (PractiseAttribute)messageAttributes[0];
                  Console.WriteLine("Property Description: " + methodAttribute.Description);
              }

              // 使用反射获取方法上的特性
              MethodInfo methodInfo = typeof(PractiseDemoClass).GetMethod("GetMessage");  //使用反射获取 PractiseDemoClass 类的 GetMessage 方法信息
              object[]  methodAttributes = methodInfo.GetCustomAttributes(typeof(PractiseAttribute), false); //获取方法上的特性
              if (methodAttributes.Length > 0)
              {
                  PractiseAttribute methodAttribute = (PractiseAttribute)methodAttributes[0];
                  Console.WriteLine("Method Description: " + methodAttribute.Description);
              }
          }
      }

      [AttributeUsage(AttributeTargets.Property | AttributeTargets.Method)]  //指定应用特性的类型为 Property 和 Method
      // 继承 System.Attribute 实现自定义特性
      class PractiseAttribute : Attribute
      {
          public string Description { get; }

          public PractiseAttribute(string description)
          {
              Description = description;
          }
      }

      class PractiseDemoClass
      {
          [Practise("this is a property")]  //应用特性
          public string Message { get; }

          public PractiseDemoClass(string message)
          {
              Message = message;
          }

          [Practise("this is a method")]  //应用特性
          public void GetMessage()
          {
              Console.WriteLine("Executing PractiseDemoClass" + Message);
          }
      }
  }
  ```
