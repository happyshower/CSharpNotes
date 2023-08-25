### C# 泛型 Generic
- 泛型（Generic）允许延迟编写类或方法中的编程元素的数据类型的规范，即泛型允许编写一个可以与任何数据类型一起工作的类或方法
- C# 允许使用类型参数而不使用特定数据类型来定义泛型类、接口、抽象类、字段、方法、静态方法、属性、事件、委托和运算符
- 类型参数是创建泛型类型实例时指定的特定类型的占位符
  
#### 泛型优势
- 泛型增加了代码的可重用性。 您不需要编写代码来处理不同的数据类型
- 泛型是类型安全的。使用与定义中指定的数据类型不同的数据类型，则会出现编译时错误。
- 泛型具有性能优势，因为它消除了装箱和拆箱的可能性。
#### 泛型类
- 泛型类是使用类名后 <> 中的类型参数定义的。
```csharp
namespace PractiseGeneric
{
    internal class Program
    {
        public static void Main(string[] args)
        {
            //指定 泛型类 的类型参数为 string 
            var cards = new BankCardGeneric<string>(5);
            cards.AddCard(0,"45465448456");
            cards.AddCard(1,"12312323123");
            
            Console.WriteLine(cards.GetCard(0));  //45465448456
            Console.WriteLine(cards.GetCard(1));  //12312323123
            
            GetData(250); //泛型参数类型：System.Int32
            GetData("Test"); //泛型参数类型：System.String
        }

        //非泛型类 通过在带有方法名称的 <> 中指定类型参数来包含泛型方法
        static T GetData<T>(T num)
        {
            Console.WriteLine("泛型参数类型：{0}", typeof(T));
            return num;
        }
    }

    //定义泛型类
    class BankCardGeneric<T> 
    {
        //使用泛型定义私有字段
        private readonly T[] _cards;

        public BankCardGeneric(int count)
        {
            _cards = new T[count];
        }

        //泛型方法，value 参数的实际数据类型在 实例化类 时指定
        public void AddCard(int index,T value)
        {
            _cards[index] = value;
        }
        //泛型方法，返回值的实际数据类型在 实例化类 时指定
        public T GetCard(int index)
        {
            return _cards[index];
        }
    }
}
```
#### 泛型方法
- 一个方法里面的参数有类型参数，或者返回类型是类型参数的话，即为为泛型方法
- 泛型类泛型方法 通过在类内部定义方法并使用泛型作为返回值类型或作为类型参数
- 非泛型类方法 通过在带有方法名称的 <T> 中指定类型参数来包含泛型方法
```csharp
namespace PractiseGeneric
{
    internal class Program
    {
        public static void Main(string[] args)
        {            
            //根据传入的参数指定类型参数
            GetData(250); //泛型参数类型：System.Int32
            GetData("Test"); //泛型参数类型：System.String
        }

        //非泛型类 通过在带有方法名称的 <T> 中指定类型参数来包含泛型方法
        static T GetData<T>(T num)
        {
            Console.WriteLine("泛型参数类型：{0}", typeof(T));
            return num;
        }
    }
}
```