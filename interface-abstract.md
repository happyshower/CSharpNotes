### C# 关键字 abstract 和 virtual

- virtual 关键字用于在基类中声明一个虚方法，表示这个方法可以在派生类中进行重写（覆盖）。派生类可以使用 override 关键字重写基类中的虚方法，从而在派生类中提供不同的实现。
- abstract 关键字用于在基类中声明一个抽象方法，表示这个方法没有实际的实现，只有方法的签名，且该方法必须声明在抽象类或接口中。派生类必须实现基类中的抽象方法，否则派生类也必须声明为抽象类

### C# interface 接口

- 接口中的成员不允许使用 public、private、protected、internal 访问修饰符,所有的接口成员都必须是公共的，但接口声明可以使用任何的访问修饰符，public、protected、internal、private，默认是 public。

- 接口中的成员不允许使用 static、virtual、abstract、sealed 修饰符。

- 在接口中不能定义字段，定义的方法不能包含方法的具体实现

- 如果两个接口中有相同的方法名，那么同时实现这两个接口的类，C#提供了显式接口实现技术，就是在方法名前加接口名称，用接口名称来限定成员，用“接口名.方法名()”来区分实现的是哪一个接口。显式实现接口时，在方法名前不能加任何访问修饰符。

- 隐式实现时对象声明为接口和类都可以访问到其行为，显式实现只有声明为接口可以访问。

- 当子类继承类和接口，类中方法和接口方法同名时，若子类没有重写该方法，则默认会优先调用父类中的方法，不会报错；若子类重写了该方法，通过显式实现可以调用接口中的方法，否则调用的仍然是该类中固有的方法。

```csharp
namespace Practise3
{
    class PractiseDemo3
    {
        static void Main(string[] args)
        {
            Sheep sheep = new Sheep();
            sheep.GetAnimalLocation();
            sheep.GetMammalEnvironment();
            sheep.GetBehavior();

            //创建了一个 IAnimal 接口类型的引用，然后将它指向了 Sheep 类的实例
            //通过接口引用，只能访问 IAnimal 接口中定义的方法，而无法直接访问 Sheep 类独有的方法
            IAnimal animal = new Sheep();
            animal.GetAnimalLocation();

            //创建了一个 IMammal 接口类型的引用，然后将它指向了 Sheep 类的实例
            //通过接口引用，只能访问 IMammal 接口中定义的方法，而无法直接访问 Sheep 类独有的方法
            IMammal mammal = new Sheep();
            mammal.GetAnimalLocation();

            //创建 Sheep 类的实例，并将它通过强制类型转换赋值给一个 IAnimal 接口类型的变量
            Sheep sheep2 = new Sheep();
            IAnimal animal2 = (IAnimal)sheep2;
            animal2.GetAnimalLocation();

        }
    }

    interface IAnimal
    {
        string Color { get; set; }
        void GetAnimalLocation();
        void GetBehavior();
    }

    interface IMammal
    {
        string Level { get; set; }
        void GetAnimalLocation();
        void GetMammalEnvironment();
    }

    class Beast
    {
        public void GetBehavior()
        {
            Console.WriteLine("Beast");
        }
    }

    //继承一个类 Beast 和两个接口 IAnimal, IMammal
    class Sheep : Beast,IAnimal, IMammal
    {
        //实现接口的属性
        public string Color { get; set; }
        public string Level { get; set; }

        //当继承自基类的方法与继承的接口的方法同名时，若不实现接口的方法，不会报错，会调用基类的同名方法
        //若在派生类中实现继承自接口的方法，则会默认隐藏父类的同名方法
        public void GetBehavior()
        {
            Console.WriteLine("Sheep");
        }

        //当继承多个接口，不同接口存在相同方法时,只需实现一次该方法即可（隐式实现）
        public void GetAnimalLocation()
        {
            Console.WriteLine("Sheep location");
        }

        //显式实现 IAnimal 接口的 GetAnimalLocation 方法，不能带有访问权限修饰符
         void IAnimal.GetAnimalLocation()
         {
             Console.WriteLine("Animal location");
         }

        //显式实现 IMammal 接口的 GetAnimalLocation 方法，不能带有访问权限修饰符
         void IMammal.GetAnimalLocation()
         {
             Console.WriteLine("Mammal location");
         }

        //实现 IMammal 接口的方法
        public void GetMammalEnvironment()
        {
            Console.WriteLine("Environment");
        }
    }

}
```

### C# abstract class 抽象类

- 抽象类是提供一个可供多个派生类共享的通用基类定义
- 抽象类可以包含抽象和非抽象方法，当一个类继承于抽象类，那么这个派生类必须实现所有的基类抽象方法，否则该类也必须声明为抽象类
- 抽象类不能使用 new 关键字创建对象，不能用​ ​sealed​​ 关键字来修饰
```csharp
namespace Practise2
{
    internal class PracticeDemo2
    {
        public static void Main(string[] args)
        {
            //创建一个Rectangle的实例对象
            Rectangle rectangle = new Rectangle();
            rectangle.GetAbstractShapeType();
            rectangle.GetAbstractDimension();
            rectangle.GetSimpleShapeType();
            rectangle.GetVirtualDimension();
        }
    }

    //使用 abstract 关键字装饰类时，该类为抽象类
    public abstract class Dimension
    {
        //使用 abstract 关键字装饰方法时，该方法为抽象方法，且无方法体
        public abstract void GetAbstractDimension();

        //使用 virtual 关键字装饰方法时，该方法为虚方法，有方法体
        public virtual void GetVirtualDimension()
        {
            Console.WriteLine("Dimension abstract class virtual method");
        }
    }

    //抽象类继承抽象了可以不重写父类的抽象方法
    public abstract class Shape : Dimension
    {
        public abstract void GetAbstractShapeType();

        public void GetSimpleShapeType()
        {
            Console.WriteLine("Shape abstract class inner shape");
        }
    }

    //类继承抽象类，抽象类的所有抽象方法必须被重写，且必须使用 overrider 关键字
    //类继承抽象类，普通方法在子类中可重新实现，也可直接使用继承自抽象类的普通方法
    //类继承抽象类，虚方法可使用关键字 override 进行重写，也可直接使用继承自抽象类的虚方法
    class Rectangle : Shape
    {
        //必须使用关键字 override 重写抽象类的方法
        public override void GetAbstractDimension()
        {
            Console.WriteLine("override abstract dimension");
        }
        //必须使用关键字 override 重写抽象类的方法
        public override void GetAbstractShapeType()
        {
            Console.WriteLine("override abstract shape");
        }
        //当不重新实现父类（抽象类）的普通方法时 ，默认使用父类的普通方法
        public void GetSimpleShapeType()
        {
            Console.WriteLine("Rectangle class inner shape");
        }
        //在父类（抽象类）中定义该方法为虚方法，子类可以继承该方法并直接使用，也可以通过 override 来重写虚方法
        public override void GetVirtualDimension()
        {
            //可以通过base方法调用抽象类的虚方法
            base.GetVirtualDimension();  //Dimension abstract class virtual method
            Console.WriteLine("override virtual method");  //override virtual method
        }
    }
}
```

### interface 与 abstract class 的比较

- 接口与抽象类非常相似，它定义了一些未实现的属性和方法。所有继承它的类都继承这些成员，在这个角度上，可以把接口理解为一个类的模板。接口最终的目的是起到统一的作用。

  1. 两者都不能使用关键字 new 直接实例化。

  2. 抽象成员可以是私有的，而接口的成员默认是公有的。

  3. 接口支持多继承，抽象类不能实现多继承。

  4. 接口只能定义抽象规则，抽象类既可以定义规则，还可能提供已实现的成员。

  5. 接口是一组行为规范，抽象类是一个不完全的类。

  6. 接口只包含方法、属性、索引器、事件的签名，但不能定义字段和包含实现的方法，抽象类可以定义字段、属性、包含有实现的方法。

  7. 接口可以作用于值类型和引用类型，抽象类只能作用于引用类型。例如，Struct 就可以继承接口，而不能继承类。
