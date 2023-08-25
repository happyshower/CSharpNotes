### C# 委托 Delegate

- 委托是一种引用方法的类型，它允许将方法作为参数传递给其他方法、存储在变量中，并且可以像普通方法一样调用它
- 委托可以用于实现回调机制、事件处理、异步编程等场景

#### 委托的使用

- 委托声明 Delegate

  ```csharp
  [访问修饰符] delegate [返回值类型] [委托名称]([参数...])

  例如：
  public delegate void TestDelegate();
  ```

- 委托实例化
  ```csharp
  ** 方式一 **
  var test = new TestDelegate(Method);  // 创建委托实例，并将该引用指向方法 Method
  ** 方式二 **
  TestDelegate test2 = Method;
  ```
- 调用委托 Delegate
  ```csharp
  TestDelegate test = Method;
  test();
  test.invoke();
  ```
- 多播委托
  ```csharp
  test += MethodA;  //使用 += 实现多播委托
  ```

#### 委托示例

```csharp
namespace PractiseDelegate
{
    //定义一个委托类型，可以引用接受一个字符串参数的方法
    public delegate void GreetDelegate(string name);

    public class Progrma
    {
        public static void Main(string[] args)
        {
            //创建委托实例，不能使用无参委托构造函数
            var greet = new GreetDelegate(ChineseGreet.GreetInChinese);

            //调用委托并传递参数
            greet("jojo"); //早上好，jojo！
            greet.Invoke("jojo"); //早上好，jojo！ 

            //将委托作为参数传递
            GreetToPeople("jojo", greet); //早上好，jojo！

        }

        static void GreetToPeople(string name, GreetDelegate GreetLanguage)
        {
            GreetLanguage(name);
        }
    }

    class ChineseGreet
    {
        public static void GreetInChinese(string name)
        {
            Console.WriteLine("早上好，{0}！", name);
        }
    }
}
```

#### 多播委托

- 多播委托是一种特殊的委托，可以持有对多个方法的引用，并且可以按照添加顺序依次调用这些方法。

```csharp
namespace PractiseDelegate
{
    public delegate void GreetDelegate(string name);

    public class Progrma
    {
        public static void Main(string[] args)
        {
            //创建委托实例，不能使用无参委托构造函数
            var greet = new GreetDelegate(ChineseGreet.GreetInChinese);
            //多播委托
            greet += EnglishGreet.GreetInEnglish;
            greet += FrenchGreet.GreetInFrench;

            greet("jojo"); //早上好，jojo！ Good morning，jojo!  Bonjour，jojo!
            greet.Invoke("jojo"); //早上好，jojo！ Good morning，jojo!  Bonjour，jojo!

            //取消委托
            greet -= EnglishGreet.GreetInEnglish;
            greet("jojo");  //早上好，jojo！ Bonjour，jojo!


            // 将委托作为参数传递
            GreetToPeople("jojo", greet); //早上好，jojo！ Bonjour，jojo!

        }

        static void GreetToPeople(string name, GreetDelegate GreetLanguage)
        {
            GreetLanguage(name);
        }
    }

    class ChineseGreet
    {
        public static void GreetInChinese(string name)
        {
            Console.WriteLine("早上好，{0}！", name);
        }
    }

    class EnglishGreet
    {
        public static void GreetInEnglish(string name)
        {
            Console.WriteLine("Good morning，{0}!", name);
        }
    }

    class FrenchGreet
    {
        public static void GreetInFrench(string name)
        {
            Console.WriteLine("Bonjour，{0}!", name);
        }
    }
}
```

### C# 事件 Event

- 事件是一种特殊类型的委托，它允许一个对象在特定条件下通知其他对象发生了某个事件。
- 引发事件的类称为发布者，接收通知的类称为订阅者。 一个事件可以有多个订阅者。
- 发布者会在发生某些操作时引发事件，有兴趣的订阅者应该注册一个事件并处理它。

#### 声明事件

- 声明一个委托

- 使用 event 关键字声明委托的变量

  ```csharp
  [访问修饰符] event [委托名称] [事件名称]

  例如：
  public delegate void TestDelegate();  //声明委托
  public event TestDelegate myEvent;   //定义事件
  ```

#### 事件示例

```csharp
namespace PractiseEvent
{
    //定义事件委托
    public delegate void EventDelegate();

    internal class Program
    {
        public static void Main(string[] args)
        {
            //实例化事件触发类
            var publisher = new Publisher();
            //事件订阅
            publisher.myEvent += new EventDelegate(Subscriber.ExecuteWithEnglish);
            publisher.myEvent += new EventDelegate(Subscriber.ExecuteWithChinese);
            //激活事件，并执行订阅事件
            publisher.OnActivateEvent();
        }
    }

    //事件发布类
    class Publisher
    {
        //定义事件
        public event EventDelegate myEvent;

        //执行订阅的方法
        public void OnActivateEvent()
        {
            myEvent?.Invoke();
        }
    }

    //事件订阅类
    class Subscriber
    {
        public static void ExecuteWithEnglish()
        {
            Console.WriteLine("Executing Subscriber method...");
        }

        public static void ExecuteWithChinese()
        {
            Console.WriteLine("正在执行订阅者方法......");
        }
    }
}
```

#### 带参事件（EventArgs 和 自定义 EventArgs）

- 使用自定义 EventArgs 传递希望传递的参数给事件订阅者

```csharp
namespace PractiseEvent
{
    //定义内置带参委托
    public delegate void EventArgsDelegate(Object sender, EventArgs s);

    //定义自定义带参委托
    public delegate void MyEventArgsDelegate(Object sender, MyEventArgs s);

    internal class Program
    {
        public static void Main(string[] args)
        {
            //实例化事件触发类
            var publisher = new Publisher();
            //带参事件订阅，使用内置 EventArgs 类型
            publisher.myEventOfEventArgs += Subscriber.ExecuteWithChineseByEventArgs;
            //激活事件，并执行订阅事件
            publisher.OnActivateEventArgsEvent(null);
            //带参事件订阅，使用自定义 MyEventArgs 类型，该类型继承 EventArgs 类
            publisher.myEventOfMyEventArgs += Subscriber.ExecuteWithChineseByMyEventArgs;
            //激活事件，并执行订阅事件
            publisher.OnActivateMyEventArgsEvent(new MyEventArgs() { IsSuccessful = true, Description = "自定义EventArgs" });
        }
    }

    //事件发布类
    class Publisher
    {
        // 定义事件
        public event EventArgsDelegate myEventOfEventArgs;
        public event MyEventArgsDelegate myEventOfMyEventArgs;

        //执行订阅的方法
        public void OnActivateEventArgsEvent(EventArgs e)
        {
            myEventOfEventArgs?.Invoke(this, e);
        }

        public void OnActivateMyEventArgsEvent(MyEventArgs myEventArgs)
        {
            myEventOfMyEventArgs?.Invoke(this,myEventArgs);
        }
    }

    //事件订阅类
    class Subscriber
    {
        // 使用基类 EventArgs 传递参数
        public static void ExecuteWithChineseByEventArgs(object sender, EventArgs s)
        {
            Console.WriteLine("正在执行订阅者有参方法......");
            Console.WriteLine(sender);
            Console.WriteLine(s);
        }
        //使用自定义 EventArgs 传递自定义参数
        public static void ExecuteWithChineseByMyEventArgs(object sender, MyEventArgs myEventArgs)
        {
            Console.WriteLine("正在执行订阅者自定义有参方法......");
            Console.WriteLine(sender);
            Console.WriteLine(myEventArgs.IsSuccessful);
            Console.WriteLine(myEventArgs.Description);
        }
    }

    // 继承 EventArgs 参数类实现自定义 EventArgs 传递数据，并用作参数类型
    public class MyEventArgs : EventArgs
    {
        public bool IsSuccessful { get; set; }
        public string Description { get; set; }
    }
}
```
