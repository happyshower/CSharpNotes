### C# 进程 Process

- 进程是操作系统中的一个基本概念，它包含着一个运行程序所需要的资源。一个正在运行的应用程序在操作系统中被视为一个进程。
- 进程之间是相互独立的，一个进程无法访问另一个进程的数据，一个进程运行的失败也不会影响其他进程的运行。通过 Process 类可启动、停止本机或远程进程。
- 进程划分为若干个独立的执行流，每个流都称为一个线程，进程可以包括一个或多个线程。

### C# 线程 Thread

- 在 C# 中，System.Threading.Thread 类用于线程的工作。它允许创建并访问多线程应用程序中的单个线程。进程中第一个被执行的线程称为主线程。
- 当 C# 程序开始执行时，主线程自动创建。使用 Thread 类创建的线程被主线程的子线程调用。可以使用 Thread 类的 CurrentThread 属性访问线程。
- 通过 Thread 类创建的线程默认为前台线程，主程序必须等待线程执行完毕后才可退出程序。

#### 管理线程的方法

- Start：开启线程

```csharp
//创建一个子线程，并传入子线程的入口方法
var thread = new Thread(ThreadProc);
//开启线程
thread.Start();
```

- Sleep：用于在一个特定的时间暂停线程

```csharp
static void Main(string[] args)
{
    var thread = new Thread(ThreadProc);
    thread.Start();
}

static void ThreadProc()
{
    for (int i = 0; i < 10; i++)
    {
        Console.WriteLine("ThreadProc: {0}", i);
        //暂停线程，两秒后继续执行
        Thread.Sleep(2000);
    }
}
```

- Join：阻塞线程，等待线程结束后执行

```csharp
static void Main(string[] args)
{
    //创建一个子线程，并传入子线程的入口方法
    var thread = new Thread(ThreadProc);
    thread.Start(); //开启线程

    //阻塞主线程，待子线程执行完毕后再执行主线程
    thread.Join();

    //主线程循环输出
    for (int i = 0; i < 4; i++)
    {
        Console.WriteLine("主线程：Main Thread {0}", i);
    }
    Console.ReadLine();
}

static void ThreadProc()
{
    for (int i = 0; i < 10; i++)
    {
        Console.WriteLine("ThreadProc: {0}", i);
    }
}

```

- Abort：销毁线程，通过抛出 ThreadAbortException 在运行时中止线程，这个异常不能被捕获，如果有 finally 块，控制会被送至 finally 块

```csharp
static void Main(string[] args)
{
    var thread = new Thread(ThreadProc);
    thread.Start();
    //暂停主线程两秒
    Thread.Sleep(2000);
    Console.WriteLine("主线程: 终止线程。");
    //通过抛出异常终止线程
    thread.Abort();
    Console.ReadLine();
}

public static void ThreadProc()
{
    try
    {
        for (int i = 0; i < 10; i++)
        {
            Console.WriteLine("ThreadProc: {0}", i);
        }
    }
    catch (ThreadAbortException e)
    {
        // 不能捕捉到异常
        Console.WriteLine("Thread Abort Exception");
    }
    finally
    {
        //这里会输出 “Couldn't catch the Thread Exception”
        Console.WriteLine("Couldn't catch the Thread Exception");
    }
}

```

### 线程池 ThreadPool

- 线程池为应用程序管理一组可重复使用的线程，避免了频繁地创建和销毁线程，在大量并发任务的情况下可以显著降低系统资源开销。
- 线程池在应用程序启动时就会被创建，其中包含一组空闲线程。当需要执行一个任务时，可以从线程池中获取一个空闲线程来执行任务，任务完成后，线程会被放回线程池以供重用
- 线程池 ThreadPool 默认为后台线程，主程序执行完毕后就退出，不管线程是否执行完毕。

#### 队列

- 全局队列
  - 全局队列是线程池的一个队列，用于存储所有等待执行的任务，线程池中的线程都可以访问这个队列，所以在任务多的时候全局队列会存在竞争而消耗资源。
- 本地队列
  - 本地队列是线程池中每个线程的的私有队列，存储线程池中工作线程需要执行的任务，在线程准备执行任务时，先从本地队列获取获取任务。

### C# 任务 Task

- Task 是一个表示异步操作的类，它提供了一种简单、轻量级的方式来创建多线程应用程序

#### 创建启动任务 Task

```csharp
public static void Main(string[] args)
{
    //1. 创建任务并传入开始任务的方法
    var task = new Task(TaskMethod);
    //通过 Start 方法启动任务
    task.Start();

    //2. 通过 Task 的静态方法 Run 创建任务并立即启动
    Task.Run(TaskMethod);

    //3. 通过 TaskFactory 创建任务工厂实例
    //TaskFactory 是一个用于创建和管理任务的工厂类
    var taskFactory = new TaskFactory();
    //通过工厂实例的 StartNew 方法创建并开启任务
    taskFactory.StartNew(TaskMethod);

    //4. 通过 Task 的 Factory 属性的 StartNew 方法创建并开启任务
    var task2 = Task.Factory.StartNew(TaskMethod);
}
```

#### 任务控制

- Wait：等待任务执行完成
- WaitAll：待所有的任务都执行完成
- WaitAny：等待任何一个任务完成就继续向下执行
- ContinueWith：第一个 Task 完成后自动启动下一个 Task，实现 Task 的延续
- CancellationTokenSource：通过 cancellation 的 token 来取消一个 Task

```csharp
public static void Main(string[] args)
{
    var task = new Task(TaskMethod);
    task.Start();

    //上一个 Task 完成后自动启动下一个 Task，实现 Task 的延续
    var continueTask = task.ContinueWith((nextTask) =>
    {
        Console.WriteLine("Task1 Completed...");
    });

    var task2 = Task.Run(TaskRun2);
    //只要有一个任务（task 、task2）完成就继续往下执行
    Task.WaitAny(task, task2);
    //等待参数列表中的所有任务都执行完毕后继续往下执行
    Task.WaitAll(task, continueTask, task2);

    Task.Run(TaskRun3);

    //创建一个 取消令牌源，可以将取消信号传递给任务
    var cts = new CancellationTokenSource();
    //创建并启动任务，将前面创建的取消令牌源的取消令牌 Token 传递给任务内部执行的 LongRunTask 方法
    var task4 = Task.Run(() => LongRunTask(cts.Token));

    //暂停线程，让任务运行一段时间
    Thread.Sleep(3000);
    //取消任务
    cts.Cancel();

    Console.ReadKey();
}

static void TaskMethod()
{
    Console.WriteLine("Task1 start");
    //Delay 方法创建异步操作，在当前线程之外的线程上等待指定的时间间隔
    //Wait 阻塞当前线程，直到前面创建的异步操作完成
    //模拟数据渲染阻塞当前线程3秒
    Task.Delay(3000).Wait();
    Console.WriteLine("Task1 end");
}

static void TaskRun2()
{
    Task.Delay(5000).Wait();
    Console.WriteLine("Task2 start");
    Console.WriteLine("Task2 end");
}

static void TaskRun3()
{
    Console.WriteLine("Task3 start");
    Task.Delay(3000).Wait();
    Console.WriteLine("Task3 end");
}

static void LongRunTask(CancellationToken token)
{
    //通过检查传递的取消令牌来确定是否应该取消任务的执行
    while (true)
    {
        //若 token.IsCancellationRequested 为 true，则任务应该取消
        if (!token.IsCancellationRequested)
        {
            Thread.Sleep(500);
            Console.WriteLine("Task Running...");
        }
        else
        {
            Console.WriteLine("任务取消了");
            break;
        }
    }
}
```

### C# Async、Await

- async 是一个关键字，用于修饰方法，表示该方法是一个异步方法。异步方法可以包含一个或多个 await 表达式，这些表达式指示在进行异步操作时暂停方法的执行，并在异步操作完成后继续执行。
- await 也是一个关键字，用于在异步方法内部等待异步操作的完成。当遇到 await 表达式时，方法的执行会暂停，让出当前线程，直到异步操作完成。一旦异步操作完成，方法会从 await 表达式处恢复执行。

```csharp
public static void Main(string[] args)
{
    Console.WriteLine("Main Thread Start ------");
    MethodAsync();
    SimpleMethod();
    Console.WriteLine("Main Thread Running...");
    Console.ReadKey();
}

// 异步方法，演示异步执行和等待
static async Task MethodAsync()
{
    Console.WriteLine("Task Start ------");
    await Task.Run(() =>
    {
        for (int i = 0; i < 5; i++)
        {
            Console.WriteLine("Async Task First  {0}...", i);
        }
    });
    Console.WriteLine("Task End ------");
}

// 普通方法，演示同步执行
public static void SimpleMethod()
{
    for (int i = 0; i < 5; i++)
    {
        Console.WriteLine("Simple Method {0}...", i);
    }
}
```

### Thread 和 Task 的区别

- Task 是基于线程池的，运行在后台，更轻量级。Thread 通过继承 Thread 类创建，运行在前台，更适合需要更底层控制的情况。
- Task 不会阻塞线程，支持异步操作，不会占用线程资源。Thread 可能会阻塞线程，需要手动管理线程的阻塞和解阻塞。
- Task 是强大的异步编程工具，使用 async/await 实现更清晰的异步操作。Thread 不直接支持异步编程，更偏向于并发执行。
- Task 支持返回值，方便异步操作的结果传递。Thread 不直接支持返回值，需要通过共享变量等手段实现。
- Task 支持构建复杂的异步工作流，可执行后续任务。Thread 不直接提供后续任务的支持。
- Task 支持通过取消令牌取消任务，可用于中止任务。Thread 不支持直接取消，需要手动管理线程的终止。
- Task 提供了更好的异常处理机制，异常可以通过 try/catch ，也可以通过 await 或 Task.Wait() 来捕获。Thread 的错误处理比较困难，因为异常在线程的上下文之外难以捕获，即不可能在父函数中捕获异常。
