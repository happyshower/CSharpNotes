### C# 集合 Collections

- C# 包括存储一系列值或对象的专用类，称为集合
- C# 中有三种类型的集合：非泛型集合，泛型集合和并发集合，在大多数情况下建议使用泛型集合，因为它们比非泛型集合执行得更快，还可以得到强类型的好处

#### ArrayList（非泛型集合）

- ArrayList 是非泛型集合类，可以存储各种类型的对象。但由于不限制元素类型，所以在使用时需要进行类型转换，这涉及到了装箱（将值类型转换成引用类型）和拆箱 (将引用类型转换为原始值类型) 操作，影响性能。

  ```csharp
  namespace PractiseCollections
  {
    internal class Program
    {
        public static void Main(string[] args)
        {
            var arrayList = new ArrayList();
            arrayList.Add(666); //添加元素
            arrayList.Add("okkk"); //添加元素
            arrayList.Add(233);
            arrayList.Remove("okkk"); //移除元素
            arrayList.Insert(0, "insert"); //通过索引插入元素
            foreach (var o in arrayList)
            {
                 Console.WriteLine(o);
            }
        }
    }
  }
  ```

#### List（泛型集合）

- List<T> 是泛型集合类，提供了类型安全的集合操作,可以在创建时指定元素的类型，避免了类型转换的问题.

  ```csharp
  namespace PractiseCollections
  {
      internal class Program
      {
          public static void Main(string[] args)
          {
              //泛型集合，创建集合时，指定集合的元素为 int 类型
              var list = new List<int>() { 2, 8, 6, 5, 5, 1, 4 };
              var list2 = new List<int>() { 33, 34, 76, 89, 23 };
              list.Add(7); //添加元素
              list.AddRange(list2); //将数组或集合的所有元素添加到 list
              list.Remove(5); // 移除指定元素
              list.RemoveAt(0); // 通过索引移除元素
              list.Insert(0, 886); //通过索引值插入数据，即 list[0]=886

              // 判断是否存在某元素（简单数据类型）
              list.Contains(886);

              //初始化复杂数据类型
              var list3 = new List<ComplexTypeClass>()
              {
                  new ComplexTypeClass() { Name = "zs", Age = 18 },
                  new ComplexTypeClass() { Name = "ls", Age = 22 }
              };
              // 判断是否存在某元素（复杂数据类型）
              //使用 Lambda 表达式
              list3.Any(item => item.Name == "zs");
              foreach (var i in list)
              {
                  Console.WriteLine(i);
              }

              var dictionary = new Dictionary<string, int>(){};

          }
      }
      class ComplexTypeClass
      {
          public string Name;
          public int Age;
      }
  }
  ```

#### Dictionary（泛型集合）

- Dictionary<TKey, TValue> 是一个通用字典集合，它以没有特定顺序存储键值对
##### Dictionary 特征
- 实现 IDictionary<TKey, TValue> 接口
- 键必须是唯一的，不能为空，但值可以为空或重复
- 可以通过在索引器中传递关联的键来访问值
- 元素存储为键值对 KeyValuePair<TKey, TValue> 对象
  ```csharp
  namespace PractiseCollections
  {
      class Program
      {
          static void Main(string[] args)
          {
              //泛型集合，通过指定类型参数创建字典
              var dictionary = new Dictionary<string,int>();
              dictionary.Add("count", 99); //设置 键名 和 键值 添加元素
              dictionary.Add("num", 67);
              dictionary.Add("code",964393);

              dictionary.Remove("count");  //根据 键名 删除元素

              //通过 ContainsKey() 方法判断是否存在某元素
              //字典中不能同时存在相同的 key，如向字典中添加已存在的 key 会报错
              if (dictionary.ContainsKey("num"))
              {
                  dictionary.Add("num", 777);
                  Console.WriteLine("新增元素！");
              }
              else
              {
                  Console.WriteLine("该元素不存在！");
              }

              //通过索引器来更新值，如果这个key 不存在的话则会添加一个新的，存在则更新值
              dictionary["code"]= 356;

              dictionary.Clear();  //移除所有元素


              // 通过索引初始化器初始化数据
              var dictionary2 = new Dictionary<string, int>()
              {
                  ["age"] = 18,
                  ["code"] = 89080989
              };

              // 通过集合初始化器初始化数据
              var dictionary3 = new Dictionary<string, int>()
              {
                  { "age",18},
                  { "code",90889008}
              };

          }
      }
  }

  ```

  #### Array、ArrayList、List 和 Dictionary 总结
  - Array：在初始化时必须指定其大小和类型，因为其在内存中是连续存储的，所以数组的索引速度是非常快的。
  - ArrayList：非泛型集合，在初始化的时候不需要指定其大小和类型，可以存储不同的数据类型，但是在存取过程中会引起装箱和拆箱操作，影响性能。
  - List：泛型集合，在初始化的时候必须指定其类型，但是不需要指定大小，不存在存取过程中引起的装箱和拆箱操作影响性能的问题。
  - Dictionary：泛型集合，在初始化的时候必须指定其类型，初始化元素时指定一个Key，并必须保证这个Key是唯一的。因此，Dictionary的索引速度非常快。通常通过Key来查找元素的，且元素的顺序是不定的。
