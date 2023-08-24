### C# 访问权限修饰符

- public：同一程序集中的任何其他代码或引用该程序集的其他程序集都可以访问该类型或成员。 某一类型的公共成员的可访问性水平由该类型本身的可访问性级别控制。
- private：只有同一 class 或 struct 中的代码可以访问该类型或成员。
- protected：只有同一 class 或者从该 class 派生的 class 中的代码可以访问该类型或成员。
- internal：同一程序集中的任何代码都可以访问该类型或成员，但其他程序集中的代码不可以。
- protected internal：该类型或成员可由对其进行声明的程序集或另一程序集中的派生 class 中的任何代码访问。
- private protected：该类型或成员可以通过从 class 派生的类型访问，这些类型在其包含程序集中进行声明。

### C# struct 结构体

- 结构体是一个具有很多相同的或者类似特征的数据结构，使用关键字 struct 声明
- 结构体在声明字段时不可以赋值
- 结构体是一个是值类型，在赋值时进行复制，即产生一个新的副本，修改原数据不会影响副本的数据
- 结构的默认构造函数是无参数的，并且不能被修改，但可以自定义有参构造函数，有参构造函数中必须给所有字段赋值
- 结构对象的创建可以使用 new 来实例化，也可以不使用 new ，但需要手动完成对象的初始化
- 结构不能继承其他结构或类，也不可以作为其他结构的基结构，但结构可以实现一个或多个接口

  ```csharp
  namespace Practise
  {
      internal class PractiseDemo
      {
          public static void Main(string[] args)
          {
              //使用结构体的默认构造函数创建实例
              Person p1 = new Person();
              p1.name = "张三";
              p1.age = 18;
              p1.ShowInfo();  //张三 18

              //结构体是值类型
              Person person = p1;  //赋值操作
              p1.age = 33;
              Console.WriteLine(person.age); //18

              //使用结构体的有参构造函数创建实例
              Person p2 = new Person("李四", 24);
              p2.ShowInfo();
              Console.WriteLine(p2.GetPersonInfo()); //李四 24

              //不使用 new 实例化获取结构对象
              Person p3;
              p3.name = "王五";
              p3.age = 22;
              p3.ShowInfo(); //王五 22
              //同名方法的重载
              p3.ShowInfo("zhongShan");  //王五 22 zhongShan
          }
      }

      //定义一个接口
      interface IPerson
      {
          //接口成员
          string GetPersonInfo();
      }

      //使用 struct 声明一个结构体，并实现 IPersonMethods 接口（可以实现多个接口）
      struct Person:IPerson
      {
          public string name;
          public int age;

          //自定义有参构造函数
          public Person(string name, int age)
          {
              //所有的字段都需要赋值
              this.name = name;
              this.age = age;
          }

          public void ShowInfo()
          {
              Console.WriteLine(name + age);
          }

          //同名方法的重载
          public void ShowInfo(string location)
          {
              Console.WriteLine(name + age + location);
          }

          //实现接口的方法
          public string GetPersonInfo()
          {
              return name + age;
          }
      }
  }
  ```

### C# class 类

- 类是一种数据结构，可在一个单元中就将状态（字段）和操作（方法和其他函数成员）结合起来。类为类实例（亦称为“对象”）提供了定义 。类支持继承和多形性，即派生类可以扩展和专门针对基类的机制
- 类的默认构造函数是无参的，可以自定义有参构造函数，当该类为派生类时，可以通过 base 关键字调用父类的构造函数
- 实例对象可以使用 new 关键字来创建
- static 修饰成员变量和成员函数时，该变量被类的实例对象共享，即不会给每个实例对象创建相应的副本；当 static 修饰静态变量时构造函数时（构造函数必须是无参的），该构造函数被称为静态构造函数，该函数在类的实例化之前执行，且只执行一次，不能使用访问修饰符
- 当派生类中的方法和基类的方法同名时，默认隐藏基类的方法，使用派生类的方法，可以通过 new 方法显式隐藏基类方法，也可以通过 base 方法调用隐藏的基类方法
- 类可以继承一个基类实现多个接口

  ```csharp
  namespace Practise1
  {
      internal class PractiseDemo
      {
          public static void Main(string[] args)
          {
              //用 new 关键字实例化一个 Square 实例对象
              Square square = new Square(10, 10, "pink");
              square.Color = "yellow";
              Square.Graphics = "sharp";
              Console.WriteLine("图形标识：" + Square.Graphics);  //sharp
              Console.WriteLine("矩形面积：" + square.GetArea()); //10000
              Console.WriteLine("矩形周长：" + square.GetPerimeter()); //40
              square.ShowSquare(); //yellow

              Square square2 = new Square(5, 15, "skyblue");
              square.Color = "blue";
              Console.WriteLine("图形标识：" + Square.Graphics);  //sharp ,共享静态成员
          }
      }

      public class Rectangle
      {
          private int _width;
          private int _height;

          public Rectangle(int width, int height)
          {
              _height = height;
              _width = width;
          }

          public int GetArea()
          {
              return _width * _height;
          }

          public int GetPerimeter()
          {
              return 2 * _width + 2 * _height;
          }
      }

      // 通过 : 符号实现父类的继承和接口的实现（只能继承一个父类，但可以实现多个接口）
      public class Square : Rectangle
      {
          // color 被定义为私有字段，只能在类内部使用
          // private string color;
          // //使用常规属性，定义一个私有的字段（color）和一个公共的属性(Color),再使用get、set访问器
          // public string Color  //定义属性
          // {
          //     get      //使用get访问器获取 color 字段
          //     {
          //         return color;
          //     }
          //     set      //使用set访问器修改 color 字段
          //     {
          //         // 通过上下文关键字 value 来修改字段的值
          //         color = value;
          //     }
          // }

          //用 static 定义一个静态成员变量
          public static string Graphics = "square";
          //使用自动属性
          public string Color { get; set; }

          //在类实例化之前执行，只执行一次
          static Square()
          {

          }

          //自定义有参构造函数，通过 base 调用父类的构造函数
          public Square(int width, int height, string color) : base(width, height)
          {
              Color = color;
          }

          public void ShowSquare()
          {
              Console.WriteLine("矩形颜色：" + Color + "-" + Graphics);
          }

          //当子类的方法和父类的方法重名时，会隐式的隐藏父类的方法
          //使用new关键字明确表示隐藏父类的同名方法
          public new int GetArea()
          {
              Console.WriteLine("父类的方法" + base.GetArea());  //100
              return base.GetArea() * 100;
          }

          //用 ~ 符号标识为析构函数，且只能有一个析构函数
          ~Square()
          {
              Console.WriteLine("析构函数被执行！");
          }
      }
  }
  ```

### struct 与 class 的不同点

- 值类型和引用类型： struct 是值类型，而 class 是引用类型。 struct 的实例在内存中直接存储其值，而 class 的实例在内存中存储的是对数据的引用。
- 拷贝和引用： 当将一个 struct 实例传递给函数或赋值给另一个变量时，会发生值的拷贝。而传递一个 class 实例时，实际上是传递了引用，多个变量可以引用同一个对象。
- struct 不能定义默认构造函数或析构函数，而类可以定义构造函数和析构函数。
- struct 自定义有参构造函数时，需要给所有的字段都进行赋值操作，而类不用全部赋值。
- struct 无法在声明的字段时赋予初始值，而类可以在声明时赋予默认值。
