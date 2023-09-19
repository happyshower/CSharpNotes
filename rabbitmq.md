### RabbitMQ 消息队列

- RabbitMQ 是由 Erlang 语言开发的一个基于 AMQP (Advanced Message Queuing Protocol) 协议的企业级消息队列中间件。可实现队列，订阅/发布，路由，通配符等工作模式。

#### 基本概念

- Server：又称 broker，接受客户端的链接，实现 AMQP 实体服务
- Connection：连接，应用程序与 broker 的网络连接
- Channel：网络信道，几乎所有的操作都在 channel 中进行，Channel 是进行消息读写的通道。客户端可以建立多个 channel，每个 channel 代表一个会话任务。
- VirtualHost：虚拟地址，用于进行逻辑隔离，最上层的消息路由。一个 Virtual Host 里面可以有若干个 Exchange 和 Queue，同一个 Virtual Host 里面不能有相同名称的 Exchange 或 Queue。
- Exchange：交换机，接收消息，根据路由键转单消息到绑定队列
- Routing key： 一个路由规则，虚拟机可用它来确定如何路由一个特定消息。

#### 队列模式

1. **简单模式**

   - 只有一个生产者，一个消费者和队列。
   - 生产者和消费者在发送和接收消息时，只需要指定队列名，而不需要指定发送到那个 Exchange，RabbitMQ 服务器会自动使用 Virtual Host 的默认的 Exchange，type 为 direct。

   ![Alt text](https://img2020.cnblogs.com/blog/630011/202108/630011-20210823224226426-2036472160.png)

   生产者示例：

   ```csharp
   // 创建连接工厂
   var factory = new ConnectionFactory()
   {
       HostName = "localhost",
       Port = 5672,
       UserName = "admin",
       Password = "admin"
   };

   // 创建连接
   var connection = factory.CreateConnection();
   // 创建连接通道
   var channel = connection.CreateModel();

   // 指定队列名
   var queueName = "test";

   // 声明并创建队列
   // - queue：队列名称ID
   // - durable：是否持久化，false 对应不持久化数据，MQ停掉数据就会数据丢失
   // - exclusive：是否队列私有化，false 则代表所有的消费者都可以访问，true 代表只有第一次拥有它的消费者才能一直使用
   // - exclusive：是否自动删除，false 代表连接停掉后不自动删除这个队列
   // - arguments：其他额外参数为null
   channel.QueueDeclare(queue: queueName, durable: false, exclusive: false, autoDelete: false, arguments: null);

   for (int i = 0; i < 100; i++)
   {
       var message = $"The Producer Message【{i + 1}】";
       // 转化为字节流数据
       var body = Encoding.UTF8.GetBytes(message);

       // 发送消息，必须指定 routingKey 且与队列同名
       // - exchange：交换机，在进行发布订阅时才会用到
       // - routingKey：路由key
       // - basicProperties：额外的设置属性
       // - body：要传递的消息字节数组
       channel.BasicPublish(exchange: "", routingKey: queueName, basicProperties: null, body: body);

       Console.WriteLine($"- Sent Message ==> ：{message}");
   }
   ```

   消费者示例：

   ```csharp
   // 指定队列名
   var queueName = "test";
   // 声明队列，指定队列名
   channel.QueueDeclare(queue: queueName, durable: false, exclusive: false, autoDelete: false, arguments: null);

   while (true)
   {
       var consumer = new EventingBasicConsumer(channel);
       consumer.Received += (model, ea) =>
       {
           byte[] body = ea.Body.ToArray();
           var message = Encoding.UTF8.GetString(body);

           Console.WriteLine($"- Receive Message ==> ：{message}");
       };

       // 通过队列名消费指定队列的信息
       channel.BasicConsume( queue: "test", autoAck: true, consumer: consumer);
   }
   ```

2. **工作模式**

   - 工作模式中，不需要设置交换器（RabbitMQ 会使用内部默认交换器进行消息转换），需要指定唯一的消息队列进行消息传递，并且可以有多个消息消费者。
   - 工作队列也称为公平性队列模式，循环分发，RabbitMQ 将按顺序将每条消息发送给下一个消费者，每个消费者将获得相同数量的消息。
   - 适用于繁重并且可以进行拆分处理的业务。

   ![Alt text](https://img2020.cnblogs.com/blog/630011/202108/630011-20210823234032492-1935279120.png)

   ```csharp
   生产者、消费者代码和简单模式相同，但存在多个消费者共同消费同一个队列
   ```

3. **发布订阅模式**

   - 无选择接收消息，一个消息生产者，一个交换机（交换机类型为 fanout ），多个消息队列，多个消费者。
   - 在应用中，只需要将队列绑定到交换机上，发送到交换机的消息都会被转发到与该交换机绑定的所有队列上。
   - 生产者 P 只需把消息发送到交换机 X ，绑定这个交换机的队列都会获得一份一样的数据。
   - 适用于进行相同业务功能处理的场合。
     ![Alt text](https://img2020.cnblogs.com/blog/630011/202108/630011-20210824004518932-1431271545.png)

   生产者示例：

   ```csharp
   // 定义交换机名称
   var exchange = "fanout_exchange";
   // 声明交换机，并指定交换机的类型为 fanout（广播）
   channel.ExchangeDeclare(exchange, type: ExchangeType.Fanout);

   for (int i = 0; i < 100; i++)
   {
     var message = $"The Producer Message【{i + 1}】";
     // 转化为字节流数据
     var body = Encoding.UTF8.GetBytes(message);
     // 发送消息
     channel.BasicPublish(exchange, routingKey: "", basicProperties: null, body: body);

     Console.WriteLine($"- Sent Message ==> ：{message}");
   }
   ```

   消费者示例：

   ```csharp
    var exchange = "fanout_exchange";

    channel.ExchangeDeclare(exchange, type: ExchangeType.Fanout);

    var queueName = channel.QueueDeclare("fanout_queue", false, false, false).QueueName;

    channel.QueueBind(queue: queueName, exchange, routingKey: "");

    Console.WriteLine("开始监听消息...");
    while (true)
    {
        var consumer = new EventingBasicConsumer(channel);

        consumer.Received += (model, ea) =>
        {
            byte[] body = ea.Body.ToArray();
            var message = Encoding.UTF8.GetString(body);

            Console.WriteLine($"- Receive Message ==>：{message}");
        };

        channel.BasicConsume(queue: queueName, autoAck: true, consumer: consumer);
    }
   ```

4. **路由模式**

   - 在发布/订阅模式的基础上，有选择的接收消息，通过 routingKey 路由进行匹配条件是否满足接收消息。

   ![Alt text](https://img2020.cnblogs.com/blog/630011/202108/630011-20210824232341125-726054635.png)

   生产者示例：

   ```csharp
   // 定义交换机名称
   var exchange = "direct_exchange";

   // 声明交换机，并指定交换机的类型为 direct
   channel.ExchangeDeclare(exchange, type: ExchangeType.Direct);

   // 声明队列
   channel.QueueDeclare("error_queue", false, false, false, null);
   channel.QueueDeclare("warn_queue", false, false, false, null);
   channel.QueueDeclare("info_queue", false, false, false, null);

   // 将交换机和队列进行绑定，并根据路由键 routingKey 匹配接收的消息
   channel.QueueBind(queue: "error_queue", exchange: exchange, routingKey: "error");
   channel.QueueBind(queue: "warn_queue", exchange: exchange, routingKey: "warn");
   channel.QueueBind(queue: "info_queue", exchange: exchange, routingKey: "info");

   for (int i = 0; i < 30; i++)
   {
       var message1 = $"RabbitMQ Direct Type Error Message【{i + 1}】";
       var body1 = Encoding.UTF8.GetBytes(message1);

       // 发送消息，将消息保存至路由键 routingKey 一致的队列中
       channel.BasicPublish(exchange, routingKey: "error", basicProperties: null, body: body1);

       Console.WriteLine($"- Sent Error Message ==> ：{message1}");

       var message2 = $"RabbitMQ Direct Type Warn Message【{i + 1}】";
       var body2 = Encoding.UTF8.GetBytes(message2);

       // 发送消息，将消息保存至路由键 routingKey 一致的队列中
       channel.BasicPublish(exchange, routingKey: "warn", basicProperties: null, body: body2);

       Console.WriteLine($"- Sent Warn Message ==> ：{message2}");

       var message3 = $"RabbitMQ Direct Type Info Message【{i + 1}】";
       var body3 = Encoding.UTF8.GetBytes(message3);

       // 发送消息，将消息保存至路由键 routingKey 一致的队列中
       channel.BasicPublish(exchange, routingKey: "info", basicProperties: null, body: body3);

       Console.WriteLine($"- Sent Info Message ==> ：{message3}");
   }
   ```

   消费者示例：

   ```csharp
    Console.WriteLine("开始监听消息...");

    while (true)
    {
        var consumer = new EventingBasicConsumer(channel);

        consumer.Received += (model, ea) =>
        {
            byte[] body = ea.Body.ToArray();
            var message = Encoding.UTF8.GetString(body);

            Console.WriteLine($"- Receive Message ==>：{message}");
        };
        // 根据队列名消费消息，如 "error_queue" 则消费 error_queue 队列的消息
        channel.BasicConsume(queue: "error_queue", autoAck: true, consumer: consumer);
    }
   ```

5. **主题（通配符）模式**

   - topic(主题) 模式跟 routing 路由模式类似，但路由模式是指定固定的路由键 routingKey，而主题模式是可以模糊匹配路由键 routingKey 。
   - topics routingKey 中可以存在两种特殊字符 " * " 与 “#”，用于做模糊匹配，其中 “ * ” 用于匹配一个单词，“ # ” 用于匹配多个单词（可以是零个），没匹配 routingKey 的消息将会被丢弃。

   生产者示例：

   ```csharp
   // 定义交换机名称
   var exchange = "topic_exchange";

   // 声明交换机，并指定交换机的类型为 topic
   channel.ExchangeDeclare(exchange, type: ExchangeType.Topic);

   // 声明队列
   channel.QueueDeclare("error_queue", false, false, false, null);
   channel.QueueDeclare("warn_queue", false, false, false, null);
   channel.QueueDeclare("info_queue", false, false, false, null);

   // 将交换机和队列进行绑定，并根据路由键 routingKey 模糊匹配接受的消息
   channel.QueueBind(queue: "error_queue", exchange: exchange, routingKey: "*.error.*");
   channel.QueueBind(queue: "warn_queue", exchange: exchange, routingKey: "#.log");
   channel.QueueBind(queue: "info_queue", exchange: exchange, routingKey: "*.info.log.#");

   for (int i = 0; i < 30; i++)
   {
       var message1 = $"RabbitMQ Direct Type Error Message【{i + 1}】";
       var body1 = Encoding.UTF8.GetBytes(message1);

       // 发送消息，与队列 routingKey 进行模糊匹配，若一致则将消息放入队列 error_queue 中
       channel.BasicPublish(exchange, routingKey: "test.error.log", basicProperties: null, body: body1);

       Console.WriteLine($"- Sent Error Message ==> ：{message1}");

       var message2 = $"RabbitMQ Direct Type Warn Message【{i + 1}】";
       var body2 = Encoding.UTF8.GetBytes(message2);

       // 发送消息，与队列 routingKey 进行模糊匹配，若一致则将消息放入队列 warn_queue 中
       channel.BasicPublish(exchange, routingKey: "warn.log", basicProperties: null, body: body2);

       Console.WriteLine($"- Sent Warn Message ==> ：{message1}");

       var message3 = $"RabbitMQ Direct Type Info Message【{i + 1}】";
       var body3 = Encoding.UTF8.GetBytes(message3);

       // 发送消息，与队列 routingKey 进行模糊匹配，若一致则将消息放入队列 info_queue 中
       channel.BasicPublish(exchange, routingKey: "test.info.log.key", basicProperties: null, body: body3);

       Console.WriteLine($"- Sent Info Message ==> ：{message3}");
   }
   ```

   消费者示例：

   ```csharp
   Console.WriteLine("开始监听消息...");

   while (true)
   {
       var consumer = new EventingBasicConsumer(channel);

       consumer.Received += (model, ea) =>
       {
           byte[] body = ea.Body.ToArray();
           var message = Encoding.UTF8.GetString(body);

           Console.WriteLine($"- Receive Message ==>：{message}");
       };
       // 根据队列名消费消息，如 "error_queue" 则消费 error_queue 队列的消息
       channel.BasicConsume(queue: "error_queue", autoAck: true, consumer: consumer);
   }
   ```

6. RPC 模式

   - RPC 是指远程过程调用，即两台服务器 A，B，一个应用部署在 A 服务器上，想要调用 B 服务器上应用提供的处理业务，处理完后然后在 A 服务器继续执行下去，把异步的消息以同步的方式执行。

   ![Alt text](https://img2020.cnblogs.com/blog/630011/202108/630011-20210825233438634-771738120.png)

   - RPC 的处理流程：

     - 当客户端启动时，创建一个匿名的回调队列。
     - 客户端为 RPC 请求设置 2 个属性：replyTo：设置回调队列名字；correlationId：标记 request。
     - 请求被发送到 rpc_queue 队列中。
     - RPC 服务器端监听 rpc_queue 队列中的请求，当请求到来时，服务器端会处理并且把带有结果的消息发送给客户端。接收的队列就是 replyTo 设定的回调队列。
     - 客户端监听回调队列，当有消息时，检查 correlationId 属性，如果与 request 中匹配，那就是结果了。

#### 补充

- QueueDeclare 用于声明一个队列

  - 参数解析：

    - durable：是否持久化，为 true 时开启持久化。

    - exclusive：排他队列，只有创建它的连接(connection)能连，创建它的连接关闭，会自动删除队列。

    - autoDelete：被消费后，消费者数量都断开时自动删除队列。

    - arguments：创建队列的参数。

  ```csharp
  // durable: true 开启持久化，即当 rabbitmq 断开连接时，数据不会丢失
  channel.QueueDeclare(queueName, durable: true, exclusive: false, autoDelete: false, arguments: null);

  var properties = channel.CreateBasicProperties();

  properties.Persistent = true; //消息持久化
  ```

- BasicQos 用于多个消费者消费同一个队列时，限制推送消息的数量

  - 参数解析：

    - prefetchSize：每条消息大小，一般设为 0，表示不限制。

    - prefetchCount：限流，告诉 RabbitMQ 不要同时给一个消费者推送多于 N 个消息，消费者会把 N 条消息缓存到本地一条条消费，如果不设，RabbitMQ 会进可能快的把消息推到客户端，导致客户端内存升高。设置合理可以不用频繁从 RabbitMQ 获取能提升消费速度和性能，设的太多的话则会增大本地内存，需要根据机器性能合理设置，官方建议设为 30。

  - global：是否为全局设置。

  - 这些限流设置针对消费者 autoAck：false 时才有效，如果是自动 Ack 的，限流不生效。

- BasicAck：手动进行消息应答

  - 参数解析：

    - deliveryTag：该消息的 index

    - multiple：是否批量.true:将一次性 ack 所有小于 deliveryTag 的消息。

更多方法参数解析 https://www.cnblogs.com/piaolingzxh/p/5448927.html
