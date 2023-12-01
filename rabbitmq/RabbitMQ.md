# RabbitMQ

## 一、设置用户权限

在RabbitMQ中，你可以使用以下命令来设置用户权限：

1. 添加用户：`rabbitmqctl add_user <username> <password>`
2. 删除用户：`rabbitmqctl delete_user <username>`
3. 更改用户密码：`rabbitmqctl change_password <username> <newpassword>`
4. 设置用户标签：`rabbitmqctl set_user_tags <username> <tag>`
5. 创建虚拟主机：`rabbitmqctl add_vhost <vhostpath>`
6. 删除虚拟主机：`rabbitmqctl delete_vhost <vhostpath>`
7. 为用户在某个虚拟主机上设置权限：`rabbitmqctl set_permissions [-p <vhostpath>] <user> <conf> <write> <read>`
8. 清除用户在某个虚拟主机上的权限：`rabbitmqctl clear_permissions [-p <vhostpath>] <username>`

请注意，这些命令需要在RabbitMQ的安装目录下的sbin目录中执行。





## 二、java实现

### 1）消息生产者



```java
public class producer {

    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        //创建一个连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        //通过工厂IP 连接RabbitMQ的队列
        connectionFactory.setHost("192.168.200.138");
        //设置用户名密码
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("123456");
        //创建连接
        Connection connection = connectionFactory.newConnection();
        //获取信道
        Channel channel = connection.createChannel();
        /**
         * 创建一个队列：参数说明
         * 1.队列名称
         * 2.是否需要持久化
         * 3.是否只供一个消费者消费，是否进行消息共享
         * 4.是否自动删除
         * 5.其他参数
         */
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        //发送一个消息
        String message = "hello world!";
        channel.basicPublish("",QUEUE_NAME,null,message.getBytes());

        System.out.println("消息发送成功！");
    }
}
```



> 在Java中创建RabbitMQ队列，你需要调用`queueDeclareNoWait`方法，并传入以下参数：
>
> 1. `queue`：队列名称。
>
> 2. `durable`：队列是否持久化。如果设置为`false`，队列将存储在内存中，服务器挂掉后，队列就会消失；如果设置为`true`，服务器重启后，队列将会重新生成。注意，这只是队列的持久化，并不代表队列中的消息持久化。
>
> 3. `exclusive`：队列是否专属。专属的范围针对的是连接，也就是说，在一个连接下面的多个信道是可见的。对于其他连接是不可见的。连接断开后，该队列会被删除。注意，这里指的是连接断开，而不是信道断开。并且，即使设置成了持久化，队列也会被删除。
>
> 4. `autoDelete`：如果所有消费者都断开连接了，是否自动删除队列。如果还没有消费者从该队列获取过消息或者监听该队列，那么该队列不会删除。只有在有消费者从该队列获取过消息后，该队列才有可能自动删除（当所有消费者都断开连接，不管消息是否获取完）。
>
> 5. ```
>    arguments
>    ```
>
>    ：队列的其他参数。这是一个键值对集合，包括但不限于以下参数：
>
>    - `Message TTL`：消息生存期。
>    - `Auto expire`：队列生存期。
>    - `Max length`：队列可以容纳的消息的最大条数。
>    - `Max length bytes`：队列可以容纳的消息的最大字节数。
>    - `Overflow behaviour`：队列中的消息溢出后如何处理。
>    - `Dead letter exchange`：溢出的消息需要发送到绑定该死信交换机的队列。
>    - `Dead letter routing key`：溢出的消息需要发送到绑定该死信交换机，并且路由键匹配的队列。
>    - `Maximum priority`：最大优先级。
>    - `Lazy mode`：懒人模式。
>    - `Master locator`：集群相关设置。
>
> 请注意，在创建RabbitMQ队列时，一旦声明了参数，就无法更改、添加或删除参数。如果需要更改参数，则必须删除并重新创建该队列。



> 在RabbitMQ中，`basicPublish`方法用于发布消息到指定的交换机。这个方法需要以下四个参数：
>
> 1. `exchange`：交换机名称。当不使用交换机时，传入空串。
> 2. `routingKey`：路由键。发布消息的队列，无论信道绑定哪个队列，最终发布消息的队列都由该字符串指定。
> 3. `props`：消息的配置属性。例如，`MessageProperties.PERSISTENT_TEXT_PLAIN`表示消息持久化。
> 4. `body`：消息数据本体，必须是字节数组。



### 2）消息消费者

```java
public class Consumer {

    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        //创建一个连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("192.168.200.138");
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("123456");

        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();

        DeliverCallback deliverCallback = (var1, var2) -> {
            System.out.println("消息：" + var2.toString());
            System.out.println(new String(var2.getBody()));
        };

        CancelCallback cancelCallback = (message) -> {
            System.out.println("消息获取被中断：" + message);
        };

        channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);
    }

}
```



> 在RabbitMQ中，`basicConsume`方法用于启动一个消费者，并返回服务端生成的消费者标识。这个方法需要以下参数：
>
> 1. `queue`：队列名。
> 2. `autoAck`：是否自动确认消息。如果设置为`true`，接收到传递过来的消息后会自动应答服务器；如果设置为`false`，接收到消息后不会自动应答服务器，需要手动调用。
> 3. `consumerTag`：消费者唯一标识。
> 4. `noLocal`：不消费同一Connection连接生产的消息。
> 5. `exclusive`：是否专属。
> 6. `arguments`：其他参数，这是一个键值对集合。
> 7. `callback`：消费者对象的回调接口。使用接口com.rabbitmq.client.Consumer的实现类com.rabbitmq.client.DefaultConsumer实现自定义消息监听器，接口中有多个不同的方法可以根据自己系统的需要实现。



> 在RabbitMQ中，`DeliverCallback`和`CancelCallback`是两个重要的回调函数。
>
> 1. `DeliverCallback`：这是一个函数式接口，用于处理从RabbitMQ服务器接收到的消息。它有一个方法`handle(String consumerTag, Delivery delivery)`，其中`consumerTag`是消费者标签，`delivery`是消息投递对象。
> 2. `CancelCallback`：这也是一个函数式接口，用于处理消费者被取消的情况。它有一个方法`handle(String consumerTag)`，其中`consumerTag`是被取消的消费者标签。
>
> 这两个回调函数都可以在调用`basicConsume`方法时作为参数传入。





## 三、工作队列

> 消息生产者和消息消费者的代码和上面的一样，就是消费者有多个实例，这些消费者间是竞争关系，轮训消费信息，即你一次我一次，你一次我一次。。。

在RabbitMQ中，工作队列（Work Queue）是一种消息队列模式，它主要用于分发耗时的任务给多个工作者（Worker）。

工作队列的主要特点是：**一个生产者，多个消费者**。生产者将任务作为消息发送到队列中，然后运行在后台的工作者进程就会取出任务并处理。当你运行多个工作者，任务就会在它们之间共享。这种模式在网络应用中非常有用，它可以在短暂的HTTP请求中处理一些复杂的任务。

值得注意的是，**一条消息只能被一个消费者消费，不能被多个消费者重复消费**。默认情况下，RabbitMQ会按顺序将消息发送给每个消费者，**平均每个消费者都会收到同等数量的消息**。这种发送消息的方式叫做**轮询**（round-robin）。

此外，为了防止消息丢失，RabbitMQ提供了消息响应（acknowledgments）。消费者会通过一个ack（响应），告诉RabbitMQ已经收到并处理了某条消息，然后RabbitMQ就会释放并删除这条消息。如果消费者挂掉了，没有发送响应，RabbitMQ就会认为消息没有被完全处理，然后重新发送给其他消费者。这样，即使工作者偶尔挂掉，也不会丢失消息。





## 四、消息应答

### 1）介绍

RabbitMQ的消息应答机制是**为了确保消息不会丢失**。消费者在接收到消息并处理完毕后，会向RabbitMQ发送一个消息应答，告知RabbitMQ该消息已被接收并处理，此时RabbitMQ可以删除该消息。如果消费者在处理消息过程中突然挂掉，而没有发送应答，RabbitMQ会理解为该消息未被完全处理，然后会将其重新分配给另一个消费者进行处理。

RabbitMQ的消息应答机制有两种模式：自动应答和手动应答。

- **自动应答**：这是默认的应答模式。在这种模式下，一旦消息被发送出去，就会立即被标记为已传送成功。但是，如果消费者在接收到消息之前就出现了连接或信道关闭的情况，那么消息就会丢失。
- **手动应答**：在这种模式下，消费者在处理完业务逻辑后，会手动返回一个ack（通知），告诉队列该消息已经被处理完毕，此时队列才会删除该消息。手动应答模式可以通过设置`autoAsk=true`来关闭。

这种机制确保了即使消费者偶尔挂掉也不会丢失任何消息。没有任何消息超时限制；只有当消费者挂掉时，RabbitMQ才会重新投递。即使处理一条消息需要花费很长的时间。



### 2）生产者

```java
public class Task02 {

    public static final String TASK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, false, false, false, null);

        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String message = scanner.next();

            channel.basicPublish("",TASK_QUEUE_NAME,null,message.getBytes("UTF-8"));
            System.out.println("消息发送成功：" + message);
        }
    }
}
```



### 3）消费者

**消费者1**

接收信息比较快（逻辑处理比较快）

```java
public class Worker03 {

    public static final String TASK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        channel.queueDeclare(TASK_QUEUE_NAME,false,false,false,null);

        DeliverCallback deliverCallback = (var1, var2) -> {
            try {
                Thread.sleep(1000);
                System.out.println("worker03接收消息：" + new String(var2.getBody()));
                channel.basicAck(var2.getEnvelope().getDeliveryTag(),false);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        };

        boolean autoAck = false;
        channel.basicConsume(TASK_QUEUE_NAME,autoAck,deliverCallback,(var1) -> {
            System.out.println(var1 + "取消消息接收回调");
        });

    }

}
```



**消费者2**

消息处理比较慢

```java
public class Worker04 {

    public static final String TASK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        channel.queueDeclare(TASK_QUEUE_NAME,false,false,false,null);

        DeliverCallback deliverCallback = (var1, var2) -> {
            try {
                Thread.sleep(30000);
                System.out.println("worker04接收消息：" + new String(var2.getBody()));
                channel.basicAck(var2.getEnvelope().getDeliveryTag(),false);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        };

        boolean autoAck = false;
        channel.basicConsume(TASK_QUEUE_NAME,autoAck,deliverCallback,(var1) -> {
            System.out.println(var1 + "取消消息接收回调");
        });
    }
}
```



### 4）运行模拟

①消息生产者发送多条信息

![](https://github.com/vankykoo/image/blob/main/011.png?raw=true)

②由于消费者1处理信息较快，消息很快就被消费，并且mq收到应答（ack

③消费者2处理信息较慢，在消息没有被消费时（即mq还未收到应答的时候），我们关闭消费者2，此时消息回到队列中，并由消费者1进行处理。

![](https://github.com/vankykoo/image/blob/main/012.png?raw=true)

![](https://github.com/vankykoo/image/blob/main/013.png?raw=true)



## 五、持久化

### 1）队列持久化

RabbitMQ队列持久化是指将**消息、队列和交换器的数据持久化到磁盘**上，以确保在RabbitMQ服务重启或宕机时数据不会丢失。具体来说，**RabbitMQ的持久化机制包括队列持久化、消息持久化和交换器持久化**。

队列持久化的一个主要目的是确保数据的安全性。在RabbitMQ中，消息通常存储在内存中，以提高消息传递的速度。然而，如果队列没有持久化，一旦RabbitMQ服务器发生故障或者重启，所有未被处理的消息都会丢失。这可能导致数据丢失，对于关键业务应用程序来说是不可接受的。

除了数据安全性，持久化队列还确保消息的可靠性。持久化队列意味着即使在消息进入队列后，但在被消费之前，如果RabbitMQ服务器崩溃，消息也会得到保留。这对于需要确保消息不会丢失的应用程序非常重要，特别是在高可用性和可靠性方面的要求。

除了队列持久化，还可以选择将交换机和消息进行持久化。这意味着消息在发布时也会被存储在磁盘上，以确保在RabbitMQ服务器发生故障时消息的安全性。这提供了更高级别的消息持久性。

如果不进行持久化，可能会发生以下问题：数据丢失：没有持久化的队列和消息会导致在服务器故障或者重启时所有未处理的消息丢失，可能导致数据丢失和消息不可靠性。可用性问题：没有持久化的队列和消息无法在服务器故障后恢复，可能导致应用程序无法正常运行，需要手动处理消息丢失和恢复。

综上所述，持久化是确保消息中间件的可靠性和数据完整性的关键部分。在大多数生产环境中，建议将队列、交换机和消息都进行持久化，以确保消息不会因服务器故障而丢失。但需要权衡性能和可靠性，因为持久化会增加磁盘IO的负载。



**在创建队列时第二个参数设为true即可。**

```java
channel.queueDeclare(QUEUE_NAME, true, false, false, null);
```



### 2）消息持久化

RabbitMQ的消息持久化是为了确保在RabbitMQ服务重启或者崩溃的情况下，消息不会丢失。我们可以将队列（Queue）、交换机（Exchange）和消息（Message）都设置为可持久化的（durable）。这样可以保证绝大部分情况下我们的RabbitMQ消息不会丢失。

具体来说，RabbitMQ的持久化机制包括以下几个方面：

- **队列持久化**：在声明队列时，将durable参数设置为true，这样即使在RabbitMQ崩溃时也能保存队列。但是需要注意的是，如果之前声明的队列不是持久化的，需要把原先队列先删除，或者重新创建一个持久化的队列，否则就会出现错误。
- **消息持久化**：在生产者处，让RabbitMQ将消息持久化至磁盘中。这需要在消息生产者修改代码，添加MessageProperties.PERSISTENT_TEXT_PLAIN属性。但是这个方法并不能保证消息100%不丢失。如果在当消息刚准备存储在磁盘的时候但是还没有存储完，消息还在缓存的一个间隔点。此时并没有真正写入磁盘。如果一旦服务崩溃宕机，消息还是会丢失。
- **交换器持久化**：交换器的持久化也是通过将durable参数设置为true来实现的。

总的来说，RabbitMQ的持久化机制可以大大提高消息的可靠性，但同时也会增加系统的复杂性和降低性能。因此，在实际使用中需要根据具体需求进行权衡。



在发布消息的时候带上参数`MessageProperties.PERSISTENT_TEXT_PLAIN`

```java
channel.basicPublish("",TASK_QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes("UTF-8"));
```



### 3）不公平分发

RabbitMQ的不公平分发（能者多劳）是一种消息分发策略，用于处理消费者处理消息的速度不一致的情况。在某些场景下，如果有两个消费者在处理任务，其中一个消费者处理任务的速度非常快，而另一个消费者处理速度却很慢，这个时候如果还是采用轮询分发的话就会导致处理速度快的消费者大部分时间处于空闲状态，而处理慢的消费者一直在忙碌。这种情况下，轮询分发就不是一个好的选择。

为了避免这种情况，我们可以使用不公平分发。在这种模式下，RabbitMQ会把新的消息分发给空闲的消费者，而不是正在忙碌的消费者。这样就可以确保所有的消费者都能够尽可能地保持忙碌状态。

要实现不公平分发，我们需要在生产者和消费者端（生产者端可以不设置）都设置参数`channel.basicQos(1)`。这个参数的意思是：如果这个任务我还没有处理完或者我还没有应答你，你先别分配给我，我目前只能处理一个任务。然后RabbitMQ就会把该任务分配给没有那么忙的那个空闲消费者。

但是需要注意的是，如果所有的消费者都没有完成手上任务，队列还在不停地添加新任务，队列有可能就会遇到队列被撑满的情况。这个时候就只能添加新的worker或者改变其他存储任务的策略。

总之，RabbitMQ的不公平分发可以有效地提高系统的吞吐量和资源利用率，但同时也需要我们更加细致地管理和监控系统状态。



给消费者信道设置basicQos为1

```java
//实现消息不公平分发
channel.basicQos(1);
```



### 4）预取值

RabbitMQ的预取值是一种流控制机制，用于限制消费者接收的未确认消息的数量。预取值是通过`channel.basicQos()`方法设置的，该值**定义了通道上允许的未确认消息的最大数量**。一旦数量达到配置的数量，RabbitMQ将停止在通道上传递更多消息，除非至少有一个未处理的消息被确认。

例如，如果我们设置预取值为5，那么消费者在任何时候都只会有5条未确认的消息。当消费者处理并确认了一条消息后，RabbitMQ会发送一条新的消息给消费者，以保持通道上的未确认消息数量为5。

这种机制可以有效地防止消费者被大量的未处理消息淹没，特别是在处理速度较慢的消费者面前。同时，预取值也可以用来实现公平分发或者能者多劳的分发策略。

需要注意的是，预取值设置过小可能会导致消费者频繁地进行网络IO操作，从而影响性能。因此，在实际使用中需要根据消费者处理消息的速度和系统的性能要求来合理设置预取值。



也是通过设置basicQos的值来设定预取值

```java
//设置预取值
channel.basicQos(5);
```



## 六、发布确认

RabbitMQ的发布确认机制是为了确保生产者发送的消息被正确地投递到了目标队列。在这个机制中，生产者将信道设置成confirm模式，一旦信道进入confirm模式，所有在该信道上面发布的消息都将会被指派一个唯一的ID（从1开始）。一旦消息被投递到所有匹配的队列之后，broker就会发送一个确认给生产者（包含消息的唯一ID），这就使得生产者知道消息已经正确到达目的队列了。如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出。

发布确认有以下几种策略：

- **单个确认发布**：这是一种简单的确认方式，它是一种同步确认发布的方式，也就是发布一个消息之后只有它被确认发布，后续的消息才能继续发布。这种确认方式有一个最大的缺点就是：发布速度特别的慢，因为如果没有确认发布的消息就会阻塞所有后续消息的发布。
- **批量确认发布**：与单个等待确认消息相比，先发布一批消息然后一起确认可以极大地提高吞吐量。当然这种方式的缺点就是：当发生故障导致发布出现问题时，不知道是哪个消息出现问题了，我们必须将整个批处理保存在内存中，以记录重要的信息而后重新发布消息。当然这种方案仍然是同步的，也一样阻塞消息的发布。
- **异步确认发布**：异步确认虽然编程逻辑比上两个要复杂，但是性价比最高，无论是可靠性还是效率都没得说。他是利用回调函数来达到消息可靠性传递的。

总体来说，RabbitMQ的发布确认机制可以有效地确保生产者发送的消息被正确地投递到目标队列，并且提供了多种策略供我们选择。



### 1）单个确认

```java
//执行完成，发布1000条消息耗时：566 ms
public static void confirmIndividually() throws Exception {
  //单个确认
  Channel channel = RabbitMQUtil.getChannel();

  String uuid = UUID.randomUUID().toString();

  channel.queueDeclare(uuid,false,false,false,null);
  //开启确认发布
  channel.confirmSelect();
  Long begin = System.currentTimeMillis();

  for (int i = 0; i < 1000; i++) {
    String message = "" + i;
    channel.basicPublish("",uuid,null,message.getBytes());
    //每发送一条信息都确认一次
    channel.waitForConfirms();
  }

  Long end = System.currentTimeMillis();

  System.out.println("执行完成，发布1000条消息耗时："+ (end - begin) +" ms");
}
```





### 2）批量确认

```java
//执行完成，发布1000条消息耗时：118 ms
public static void confirmBatch() throws IOException, TimeoutException, InterruptedException {
  Channel channel = RabbitMQUtil.getChannel();

  String uuid = UUID.randomUUID().toString();

  channel.queueDeclare(uuid,false,false,false,null);
  //开启确认发布
  channel.confirmSelect();
  Long begin = System.currentTimeMillis();

  final int CONFIRM_COUNT = 100;

  for (int i = 0; i < 1000; i++) {
    String message = "" + i;
    channel.basicPublish("",uuid,null,message.getBytes());
    //每发送100条消息确认一次
    if(i % CONFIRM_COUNT == 0){
      channel.waitForConfirms();
    }
  }
  Long end = System.currentTimeMillis();

  System.out.println("执行完成，发布1000条消息耗时："+ (end - begin) +" ms");
}
```



### 3）异步批量确认

给信道添加一个确认的监听器

```java
//执行完成，发布1000条消息耗时：12 ms
public static void confirmAsync() throws Exception{
  Channel channel = RabbitMQUtil.getChannel();

  String uuid = UUID.randomUUID().toString();

  channel.queueDeclare(uuid,false,false,false,null);
  //开启确认发布
  channel.confirmSelect();
  //确认接收到消息的回调函数
  ConfirmCallback ackConfirmCallback = (var1, var3) -> {
    //System.out.println("确认消息：" + var1);
  };
  //确认没有接收到消息的回调函数
  ConfirmCallback nackConfirmCallback = (var1, var3) -> {
    System.out.println("未确认消息：" + var1);
  };
  //添加确认监听器
  channel.addConfirmListener(ackConfirmCallback,nackConfirmCallback);
  Long begin = System.currentTimeMillis();

  for (int i = 0; i < 1000; i++) {
    String message = "" + i;
    channel.basicPublish("",uuid,null,message.getBytes());
  }
  Long end = System.currentTimeMillis();

  System.out.println("执行完成，发布1000条消息耗时："+ (end - begin) +" ms");
}
```



#### 处理未确认接收的消息

在RabbitMQ的异步发布确认中，处理未确认的消息通常是通过将未确认的消息放到一个基于内存的能被发布线程访问的队列，比如使用ConcurrentLinkedQueue这个队列在confirm callbacks与发布线程之间进行消息的传递。

具体来说，我们可以在发送消息时，将所有要发送的消息记录下来，例如使用ConcurrentSkipListMap这个线程安全有序的哈希表来存储每个消息及其对应的序号。然后我们可以设置一个消息确认成功的回调函数和一个消息确认失败的回调函数。

**在消息确认成功的回调函数中，我们可以删除已经确认的消息，剩下的就是未确认的消息。**而在消息确认失败的回调函数中，我们可以打印出未确认的消息是哪些。

这样一来，如果有未确认的消息，我们就可以知道是哪些消息没有被确认，并且可以根据需要对这些未确认的消息进行重新发送或者其他处理。这种方法既可以确保消息的可靠性传递，又可以提高系统的吞吐量。但是需要注意的是，这种方法需要更复杂的编程逻辑，并且如果有大量的未确认消息可能会占用较多的内存。



## 七、交换机

![](https://github.com/vankykoo/image/blob/main/014.png?raw=true)

### 1）概念

RabbitMQ的交换机（Exchange）是消息传递模型的核心，它负责接收生产者的消息并将它们推入队列。交换机必须确切知道如何处理收到的消息，是应该把这些消息放到特定队列，还是说把他们到许多队列中，还是说应该丢弃它们。这就由交换机的类型来决定。

RabbitMQ一共有四种交换机：

- **直连交换机（Direct Exchange）**：根据Routing Key（路由键）进行投递到不同队列。
- **扇形交换机（Fanout Exchange）**：采用广播模式，根据绑定的交换机，路由到与之对应的所有队列。
- **主题交换机（Topic Exchange）**：对路由键进行模式匹配后进行投递，符号#表示一个或多个词，*表示一个词。
- **头部交换机（Headers Exchange）**：不处理路由键。而是根据发送的消息内容中的headers属性进行匹配。

总的来说，RabbitMQ的交换机起着承上启下的作用，根据不同业务场景，为我们内置了多种交换机类型。



### 2）扇出交换机

扇形交换机（Fanout Exchange）是RabbitMQ中的一种交换机类型，它将接收到的所有消息广播到所有绑定到它上面的队列。也就是说，如果有n个队列绑定到扇形交换机上，那么扇形交换机就会将消息发送到这n个队列上。

扇形交换机在处理消息时，会忽略Routing Key（路由键），只需将队列绑定到扇形交换机即可接收到发送到该交换机的所有消息。这种特性使得扇形交换机非常适合广播场景，例如日志收集、系统通知等。

在实现步骤中，我们需要新建一个配置类，在配置类中创建两个队列和一个扇形交换机，并将队列绑定到扇形交换机。然后在发送者和接收者类中，分别实现发送消息和接收消息的逻辑。具体的代码实现可以参考相关的RabbitMQ开发文档或者教程。



**消息生产者**

```java
public static void main(String[] args) throws Exception{
  Channel channel = RabbitMQUtil.getChannel();
  //channel.exchangeDeclare(EXCHANGE_NAME,"fanout");

  Scanner scanner = new Scanner(System.in);
  while (scanner.hasNext()){
    String message = scanner.next();
    channel.basicPublish(EXCHANGE_NAME,"",null,message.getBytes("UTF-8"));
    System.out.println("消息发送成功：" + message);
  }
}
```



**消息消费者**

```java
public static void main(String[] args) throws Exception{
  Channel channel = RabbitMQUtil.getChannel();
  //创建一个交换机
  channel.exchangeDeclare(EXCHANGE_NAME,"fanout");
  //创建一个临时队列
  String queueName = channel.queueDeclare().getQueue();
  //参数：1.队列名  2.交换机名  3.路由key
  channel.queueBind(queueName,EXCHANGE_NAME,"");
  System.out.println("ReceiveLogs01等待消息。。。");
  DeliverCallback deliverCallback = (var1,var2) -> {
    System.out.println("ReceiveLogs01接收消息成功：" + new String(var2.getBody()));
  };

  channel.basicConsume(queueName,true,deliverCallback,(var) -> {});
}
```



### 3）直接交换机

直连交换机（Direct Exchange）是RabbitMQ中的一种交换机类型，它根据消息的Routing Key（路由键）进行精确匹配，只有完全匹配的消息会被路由到对应的队列中。

在实现步骤中，我们需要新建一个配置类，在配置类中创建一个队列和一个直连交换机，并将队列绑定到直连交换机。然后在发送者和接收者类中，分别实现发送消息和接收消息的逻辑。

具体来说，**生产者发送消息到交换机时，会指定一个Routing Key。然后交换机会查看所有绑定到自己的队列，只有当队列的Binding Key与消息的Routing Key完全匹配时，消息才会被路由到该队列。**

这种方式非常**适合路由键明确、目标明确的场景**。例如，你可以将日志消息根据严重级别发送到不同的队列，只需要为每个严重级别指定一个Routing Key即可。



消息生产者

```java
public class DirectLogs {

    public static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMQUtil.getChannel();
        //channel.exchangeDeclare(EXCHANGE_NAME,"fanout");

        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String message = scanner.next();
            channel.basicPublish(EXCHANGE_NAME,message,null,message.getBytes("UTF-8"));
            System.out.println("消息发送成功：" + message);
        }
    }

}
```



消息消费者

```java
public class DirectReceiveLogs01 {

    public static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMQUtil.getChannel();
        //创建一个交换机
        channel.exchangeDeclare(EXCHANGE_NAME,"direct");
        //创建一个临时队列
        String queueName = channel.queueDeclare().getQueue();
        //参数：1.队列名  2.交换机名  3.路由key
        //这里一个队列绑定了两个key
        channel.queueBind(queueName,EXCHANGE_NAME,"vanky");
        channel.queueBind(queueName,EXCHANGE_NAME,"koo");
        System.out.println("DirectReceiveLogs01等待消息。。。");
        DeliverCallback deliverCallback = (var1, var2) -> {
            System.out.println("DirectReceiveLogs01接收消息成功：" + new String(var2.getBody()));
        };

        channel.basicConsume(queueName,true,deliverCallback,(var) -> {});
    }

}
```



### 4）主题交换机

主题交换机（Topic Exchange）是RabbitMQ中的一种交换机类型，它的特点是在路由键和绑定键之间有一定的规则。这种交换机其实跟直连交换机流程差不多，但是它可以进行模式匹配，从而实现更复杂的路由策略。

在主题交换机中，路由键通常是由点分隔的一系列单词，如"stock.usd.nyse"、“nyse.vmw”、"quick.orange.rabbit"等。绑定键也必须是这种形式。在队列和交换机进行绑定时，可以使用两种特殊字符来进行模式匹配：

- 星号（*）：用来表示一个单词。
- 井号（#）：用来表示任意数量（零个或多个）单词。

例如，我们可以将一个队列Q1绑定到主题交换机上，绑定键为 “*.orange.*”，那么任何路由键为两个单词，且第二个单词为"orange"的消息都会被路由到队列Q1。

总的来说，主题交换机提供了一种灵活的方式来将生产者的消息路由到一个或多个队列，特别适合于多租户应用、日志记录以及动态路由等场景。

![](https://github.com/vankykoo/image/blob/main/015.png?raw=true)



消息生产者

```java
public class TopicLogs {

    public static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMQUtil.getChannel();
        //channel.exchangeDeclare(EXCHANGE_NAME,"fanout");

        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String message = scanner.next();
            channel.basicPublish(EXCHANGE_NAME,message,null,message.getBytes("UTF-8"));
            System.out.println("消息发送成功：" + message);
        }
    }

}
```



消息消费者1

```java
public class TopicReceiveLogs01 {

    public static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMQUtil.getChannel();

        channel.exchangeDeclare(EXCHANGE_NAME,"topic");

        channel.queueDeclare("Q1",false,false,false,null);
        channel.queueBind("Q1",EXCHANGE_NAME,"*.orange.*");

        DeliverCallback deliverCallback = (var1,message) -> {
            System.out.println("TopicReceiveLogs01接收到消息：" + new String(message.getBody()) + " 路由键：" + message.getEnvelope().getRoutingKey());
        };

        channel.basicConsume("Q1",true,deliverCallback,(var) ->{});
    }

}
```



消息消费者2

```java
public class TopicReceiveLogs02 {

    public static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMQUtil.getChannel();

        channel.exchangeDeclare(EXCHANGE_NAME,"topic");

        channel.queueDeclare("Q2",false,false,false,null);
      //这里匹配的路由键有两种
        channel.queueBind("Q2",EXCHANGE_NAME,"*.*.rabbit");
        channel.queueBind("Q2",EXCHANGE_NAME,"lazy.#");

        DeliverCallback deliverCallback = (var1,message) -> {
            System.out.println("TopicReceiveLogs02接收到消息：" + new String(message.getBody()) + " 路由键：" + message.getEnvelope().getRoutingKey());
        };

        channel.basicConsume("Q2",true,deliverCallback,(var) ->{});
    }

}
```



## 八、死信队列

### 1）介绍

RabbitMQ的死信队列（Dead-Letter-Exchange，简称DLX）是一种特殊的队列，用于处理无法正常消费的消息。当一个队列中的消息变成死信（dead message）后，它会被重新发送到另一个交换机中，这个交换机就是DLX，绑定DLX的队列就称之为死信队列。

消息可能因以下原因变成死信：

1. 消息被拒绝
2. 消息过期
3. 队列达到最大长度

当这个队列中存在死信时，RabbitMQ会自动地将这个消息重新发布到设置的DLX上去，进而被路由到另一个队列，即死信队列。要使用死信队列，只需要在定义队列的时候设置队列参数`x-dead-letter-exchange`指定交换机即可。

例如，你可以设置一个消息的过期时间，如果该消息在过期时间内未被消费，则会被发送到死信队列。同样，如果一个队列的长度超过了设定的最大长度，那么多出来的消息也会被自动发送到死信队列。

总的来说，死信队列是一种有效的处理无法正常消费的消息的机制。

![](https://github.com/vankykoo/image/blob/main/016.png?raw=true)



### 2）消息过期

消息生产者

```java
public class Producer {
    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMQUtil.getChannel();

        AMQP.BasicProperties basicProperties = new AMQP.BasicProperties().builder().expiration("10000").build();

        for (int i = 0; i < 10; i++) {
            String message = "info" + i;
            channel.basicPublish("normal_exchange","normal",basicProperties,message.getBytes());
            System.out.println("生产者发送消息：" + message);
        }
    }
}
```



消息消费者1------普通队列

```java
public class Consumer01 {

    public static final String NORMAL_QUEUE = "normal_queue";
    public static final String NORMAL_EXCHANGE = "normal_exchange";
    public static final String DEAD_QUEUE = "dead_queue";
    public static final String DEAD_EXCHANGE = "dead_exchange";

    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMQUtil.getChannel();
        //创建交换机
        channel.exchangeDeclare(NORMAL_EXCHANGE,"direct");
        channel.exchangeDeclare(DEAD_EXCHANGE,"direct");
        //定义队列参数，给交换机绑定死信队列
        HashMap<String, Object> map = new HashMap<>();
        map.put("x-dead-letter-exchange",DEAD_EXCHANGE);
        map.put("x-dead-letter-routing-key","dead");

        //给交换机绑定死信队列
        channel.queueDeclare(NORMAL_QUEUE,false,false,false,map);

        channel.queueDeclare(DEAD_QUEUE,false,false,false,null);

        //交换机和队列绑定
        channel.queueBind(NORMAL_QUEUE,NORMAL_EXCHANGE,"normal");
        channel.queueBind(DEAD_QUEUE,DEAD_EXCHANGE,"dead");
        System.out.println("Consumer01等待消息。。。");

        DeliverCallback deliverCallback = (var1,var2) -> {
            System.out.println(new String(var2.getBody(),"UTF-8"));
        };

        channel.basicConsume(NORMAL_QUEUE,true,deliverCallback,(var) ->{});

    }

}
```



消息消费者2-------死信队列

```java
public class Consumer02 {

    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMQUtil.getChannel();

        System.out.println("Consumer02等待消息。。。");

        DeliverCallback deliverCallback = (var1,var2) -> {
            System.out.println("Consumer02接收消息：" + new String(var2.getBody(),"UTF-8"));
        };

        channel.basicConsume(DEAD_QUEUE,true,deliverCallback,(var) ->{});
    }
}
```



### 3）队列达到最大长度

```java
public class Consumer01 {

    public static final String NORMAL_QUEUE = "normal_queue";
    public static final String NORMAL_EXCHANGE = "normal_exchange";
    public static final String DEAD_QUEUE = "dead_queue";
    public static final String DEAD_EXCHANGE = "dead_exchange";

    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMQUtil.getChannel();
        //创建交换机
        channel.exchangeDeclare(NORMAL_EXCHANGE,"direct");
        channel.exchangeDeclare(DEAD_EXCHANGE,"direct");
        //定义队列参数
        HashMap<String, Object> map = new HashMap<>();
        map.put("x-dead-letter-exchange",DEAD_EXCHANGE);
        map.put("x-dead-letter-routing-key","dead");
        //设置普通队列的最大长度
        map.put("x-max-length",6);
        //给交换机绑定死信队列
        channel.queueDeclare(NORMAL_QUEUE,false,false,false,map);

        channel.queueDeclare(DEAD_QUEUE,false,false,false,null);

        //交换机和队列绑定
        channel.queueBind(NORMAL_QUEUE,NORMAL_EXCHANGE,"normal");
        channel.queueBind(DEAD_QUEUE,DEAD_EXCHANGE,"dead");
        System.out.println("Consumer01等待消息。。。");

        DeliverCallback deliverCallback = (var1,var2) -> {
            System.out.println(new String(var2.getBody(),"UTF-8"));
        };

        channel.basicConsume(NORMAL_QUEUE,true,deliverCallback,(var) ->{});

    }
}
```





### 4）消息被拒

模拟消息被拒情况

```java
public class Consumer01 {

    public static final String NORMAL_QUEUE = "normal_queue";
    public static final String NORMAL_EXCHANGE = "normal_exchange";
    public static final String DEAD_QUEUE = "dead_queue";
    public static final String DEAD_EXCHANGE = "dead_exchange";

    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMQUtil.getChannel();
        //创建交换机
        channel.exchangeDeclare(NORMAL_EXCHANGE,"direct");
        channel.exchangeDeclare(DEAD_EXCHANGE,"direct");
        //定义队列参数
        HashMap<String, Object> map = new HashMap<>();
        map.put("x-dead-letter-exchange",DEAD_EXCHANGE);
        map.put("x-dead-letter-routing-key","dead");
        //给交换机绑定死信队列
        channel.queueDeclare(NORMAL_QUEUE,false,false,false,map);

        channel.queueDeclare(DEAD_QUEUE,false,false,false,null);

        //交换机和队列绑定
        channel.queueBind(NORMAL_QUEUE,NORMAL_EXCHANGE,"normal");
        channel.queueBind(DEAD_QUEUE,DEAD_EXCHANGE,"dead");
        System.out.println("Consumer01等待消息。。。");

        DeliverCallback deliverCallback = (var1,var2) -> {
            String message = new String(var2.getBody(), "UTF-8");
            if (message.equals("info5")){
                //拒绝消息
                System.out.println("此消息被C1拒绝：" + message);
                //第二个参数表示不放回原队列
                channel.basicReject(var2.getEnvelope().getDeliveryTag(),false);
            }else{
                System.out.println("C1接收消息成功：" + message);
                channel.basicAck(var2.getEnvelope().getDeliveryTag(),false);
            }

        };

        channel.basicConsume(NORMAL_QUEUE,false,deliverCallback,(var) ->{});

    }

}
```





## 九、延迟队列

### 1）介绍

RabbitMQ的延迟队列是一种特殊的队列，可以用来处理需要在某个时间点被处理的消息。在RabbitMQ中，实现延迟队列主要有两种方式：

1. 利用RabbitMQ**自带的消息过期和死信队列机制**实现定时任务。具体来说，可以在发送消息时设置消息的TTL（Time To Live）属性，即消息的生存时间。如果一条消息在TTL指定的时间内没有被消费，那么这条消息就会变成死信。同时，我们可以为队列设置一个特殊的交换机，称为死信交换机。当队列中有消息变成死信时，这些消息会被发送到死信交换机，然后路由到另一个队列，即死信队列。消费者可以从死信队列中获取并处理这些消息。
2. 使用RabbitMQ的rabbitmq_delayed_message_exchange**插件**来实现定时任务。这种方式相对简单，只需要在安装了该插件的RabbitMQ服务器上声明一个类型为x-delayed-message的交换机，并在发送消息时添加一个x-delay属性来指定延迟的时间。

无论采用哪种方式，都可以满足在某个事件发生之后或者之前的指定时间点完成某一项任务的需求。例如：订单在十分钟之内未支付则自动取消、新创建的店铺如果在十天内都没有上传过商品则自动发送消息提醒、账单在一周内未支付则自动结算等等。



![](https://github.com/vankykoo/image/blob/main/017.png?raw=true)



### 2）整合到springboot

依赖

```XML
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.47</version>
  </dependency>
  <dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
  </dependency>
  <dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
  </dependency>
  <dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```



配置类

```properties
spring.rabbitmq.host=192.168.200.138
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=123456
```



### 3）基于死信队列的延迟队列

配置类设置交换机，队列，绑定

```java
@Configuration
public class TtlQueueConfig {

    //普通交换机
    public static final String X_EXCHANGE = "X";
    //死信交换机
    public static final String DEAD_EXCHANGE = "Y";
    //普通队列
    public static final String NORMAL_QUEUE_A = "QA";
    public static final String NORMAL_QUEUE_B = "QB";
    public static final String NORMAL_QUEUE_C = "QC";
    //死信队列
    public static final String DEAD_QUEUE_D = "QD";

    //声明xExchange
    @Bean("xExchange")
    public DirectExchange xExchange(){
        return new DirectExchange(X_EXCHANGE);
    }

    //声明yExchange
    @Bean("yExchange")
    public DirectExchange yExchange(){
        return new DirectExchange(DEAD_EXCHANGE);
    }

    //声明普通队列
    @Bean("queueA")
    public Queue queueA(){
        Map<String,Object> arguments = new HashMap<>();
        arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE);
        arguments.put("x-dead-letter-routing-key","YD");
        arguments.put("x-message-ttl",10000);
        return QueueBuilder.durable(NORMAL_QUEUE_A).withArguments(arguments).build();
    }

    //声明普通队列
    @Bean("queueB")
    public Queue queueB(){
        Map<String,Object> arguments = new HashMap<>();
        arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE);
        arguments.put("x-dead-letter-routing-key","YD");
        arguments.put("x-message-ttl",40000);
        return QueueBuilder.durable(NORMAL_QUEUE_B).withArguments(arguments).build();
    }

    //声明普通队列
    @Bean("queueC")
    public Queue queueC(){
        Map<String,Object> arguments = new HashMap<>();
        arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE);
        arguments.put("x-dead-letter-routing-key","YD");
        return QueueBuilder.durable(NORMAL_QUEUE_C).withArguments(arguments).build();
    }

    //声明私信队列
    @Bean("queueD")
    public Queue queueD(){
        return QueueBuilder.durable(DEAD_QUEUE_D).build();
    }

    //绑定
    @Bean
    public Binding queueABindingX(@Qualifier("queueA")Queue queueA,
                                  @Qualifier("xExchange")DirectExchange xExchange){
        return BindingBuilder.bind(queueA).to(xExchange).with("XA");
    }

    //绑定
    @Bean
    public Binding queueBBindingX(@Qualifier("queueB")Queue queueB,
                                  @Qualifier("xExchange")DirectExchange xExchange){
        return BindingBuilder.bind(queueB).to(xExchange).with("XB");
    }

    //绑定
    @Bean
    public Binding queueCBindingX(@Qualifier("queueC")Queue queueB,
                                  @Qualifier("xExchange")DirectExchange xExchange){
        return BindingBuilder.bind(queueB).to(xExchange).with("XC");
    }

    //绑定
    @Bean
    public Binding queueDBindingX(@Qualifier("queueD")Queue queueD,
                                  @Qualifier("yExchange")DirectExchange yExchange){
        return BindingBuilder.bind(queueD).to(yExchange).with("YD");
    }

}
```



消息消费者

```java
@Component
@Slf4j
public class DeadLetterQueueConsumer {

    @RabbitListener(queues = "QD")
    public void receive(Message message, Channel channel){
        String msg = message.getBody().toString();
        log.info("当前时间：{}，私信队列接收到消息：{}",new Date().toString(),msg);
    }

}
```



controller在网页上发布消息

```java
@Slf4j
@RestController
@RequestMapping("/ttl")
public class SendMsgController {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/sendMsg/{message}")
    public void sendMsg(@PathVariable String message){
        log.info("当前时间：{}，发送消息给QA,QB成功，消息：{}",new Date().toString(),message);
        rabbitTemplate.convertAndSend("X","XA","消息来自ttl为10s的QA：" + message);
        rabbitTemplate.convertAndSend("X","XB","消息来自ttl为40s的QB：" + message);
    }

    @GetMapping("/sendMsgWithTtl/{message}/{ttl}")
    public void sendMsgWithTtl(@PathVariable String message,@PathVariable String ttl){
        log.info("发送消息成功：{}，设置的时长为：{} ms",message,ttl);
        rabbitTemplate.convertAndSend("X","XC",message,msg -> {
            msg.getMessageProperties().setExpiration(ttl);
            return msg;
        });
    }
}
```



#### 缺陷

RabbitMQ基于死信队列实现延迟队列的方式虽然在很多场景下都非常有用，但是也存在一些缺陷：

1. **精度问题**：RabbitMQ的消息TTL是以毫秒为单位的，但实际上，由于RabbitMQ的内部实现，消息的过期检查并不是实时的，这可能会导致实际的延迟时间比设置的TTL稍微长一些。
2. **队列堆积问题**：如果队列中的消息因为某种原因（比如消费者处理能力不足）积压了，那么即使消息已经过期，也不会立即被删除，**只有当消息即将被消费时，才会检查其是否过期**。这可能会导致大量的过期消息在队列中堆积，占用大量的内存。【重点】
3. **延迟时间受限**：RabbitMQ设置消息TTL的最大值为2^32-1毫秒（约49天），这意味着如果你需要更长时间的延迟，就无法使用基于死信队列的延迟队列来实现。
4. **资源占用问题**：如果每个消息的延迟时间都不同，那么可能需要为每个消息创建一个单独的延迟队列，这将会占用大量的资源。

以上就是RabbitMQ基于死信队列实现延迟队列的一些主要缺陷。在使用时需要根据具体业务场景和需求来权衡。



### 4）基于插件实现延迟队列

①去官网下载rabbitmq_delayed_message_exchange插件，

②放到插件目录/usr/lib/rabbitmq/lib/rabbitmq_server-x.x.x/plugins里面

③进入到该目录，执行命令让插件生效

```c
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```



![](https://github.com/vankykoo/image/blob/main/018.png?raw=true)



延迟队列配置类

```java
@Configuration
public class DelayedQueueConfig {

    public static final String DELAYED_QUEUE_NAME = "delayed.queue";
    public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
    public static final String DELAYED_ROUTING_KEY = "delayed.routingkey";

    @Bean("delayedQueue")
    public Queue delayedQueue(){
        return new Queue(DELAYED_QUEUE_NAME);
    }

    @Bean("delayedExchange")
    public CustomExchange delayedExchange(){
        Map<String, Object> arguments = new HashMap<>();
        arguments.put("x-delayed-type","direct");

        return new CustomExchange(DELAYED_EXCHANGE_NAME,"x-delayed-message",true,false,arguments);
    }

    @Bean
    public Binding queueBindingExchange(@Qualifier("delayedQueue") Queue delayedQueue,@Qualifier("delayedExchange") CustomExchange delayedExchange){
        return BindingBuilder.bind(delayedQueue).to(delayedExchange).with(DELAYED_ROUTING_KEY).noargs();
    }
}
```



消息消费者

```java
@Slf4j
@Component
public class DelayQueueConsumer {

    @RabbitListener(queues = DelayedQueueConfig.DELAYED_QUEUE_NAME)
    public void receiveDelayedMessage(Message message){
        String msg = new String(message.getBody());
        log.info("当前时间：{}，收到延迟消息：{}",new Date().toString(),msg);
    }

}
```



消费生产者-----controller

```java
@GetMapping("/sendDelayedMsg/{message}/{delayedTime}")
public void sendDelayedMsg(@PathVariable String message,@PathVariable Integer delayedTime){
  log.info("当前时间：{}，发送消息成功，延时{} ms，消息：{}",new Date().toString(),delayedTime,message);
  rabbitTemplate.convertAndSend(DelayedQueueConfig.DELAYED_EXCHANGE_NAME,DelayedQueueConfig.DELAYED_ROUTING_KEY,message,msg ->{
    msg.getMessageProperties().setDelay(delayedTime);
    return msg;
  });
}
```





##十、发布确认（高级）

###1）介绍

在使用RabbitMQ发送消息时，如果RabbitMQ宕机，确保消息不丢失的方法主要有以下几种：

1. **消息持久化**：在发送消息时，可以将消息标记为持久化，这样即使RabbitMQ宕机，消息也会被保存在磁盘上，当RabbitMQ恢复后，这些消息仍然可以被消费。
2. **发送确认**：RabbitMQ提供了消息确认机制，生产者在发送消息后，可以等待RabbitMQ的确认信息，只有当收到确认信息后，生产者才会认为消息已经成功发送。如果在等待确认信息的过程中RabbitMQ宕机，生产者可以选择重新发送消息。
3. **事务机制**：RabbitMQ还支持事务机制，生产者可以将一系列的发送操作放在一个事务中，只有当所有的发送操作都成功后，事务才会提交。如果在事务提交之前RabbitMQ宕机，那么这个事务中的所有操作都会被回滚，生产者可以选择重新开始一个新的事务。
4. **备份交换机**：在定义交换机时，可以指定一个备份交换机。当主交换机无法处理消息时（例如因为宕机），这些消息会被路由到备份交换机。

以上就是在使用RabbitMQ发送消息时，如何确保RabbitMQ宕机后消息不丢失的一些方法。具体使用哪种方法取决于你的具体需求和场景。



###2）发布确认升级

springboot配置

```properties
spring.rabbitmq.publisher-confirm-type=correlated
spring.rabbitmq.publisher-returns=true
```

> 在Java中，`spring.rabbitmq.publisher-confirm-type`是一个用于配置RabbitMQ发布确认功能的参数。这个参数有三个可选值：`SIMPLE`、`CORRELATED`和`NONE`。
>
> - `SIMPLE`：这种模式下，每次发送消息后，都会等待Broker的确认消息，确认后才会发送下一条消息。这种方式效率较低，但是可以保证消息的可靠性。
> - **`CORRELATED`**：这种模式下，发送消息后**不会立即等待Broker的确认消息，而是可以继续发送下一条消息，当Broker返回确认消息时，会回调用户设定的ConfirmCallback**。这种方式效率较高，同时也可以保证消息的可靠性。
> - `NONE`：这种模式下，发送消息后不会等待Broker的确认消息，也就是所谓的fire-and-forget。这种方式效率最高，但是无法保证消息的可靠性。
>
> 需要注意的是，在早期版本的Spring Boot中（2.2.0.RELEASE版本之前），使用的是`spring.rabbitmq.publisher-confirm`属性来配置发布确认。但在新版本中，该属性已被弃用，改为使用`spring.rabbitmq.publisher-confirm-type`属性。具体使用哪个属性，请根据你使用的Spring Boot版本来决定。如果你正在使用一个较新的版本，那么应该使用`spring.rabbitmq.publisher-confirm-type`属性。



**消息生产者：controller**

```java
@RestController
@Slf4j
@RequestMapping("/confirm")
public class ConfirmController {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/sendMessage/{message}")
    public void sendMessage(@PathVariable String message){
      //设置消息id
        CorrelationData correlationData = new CorrelationData("666");
        CorrelationData correlationData2 = new CorrelationData("777");
	//发送可靠的消息，交换机和队列都存在  
  rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME,ConfirmConfig.CONFIRM_ROUTING_KEY,message,correlationData);

  //发送不可靠信息，发送到不存在的队列中
      rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME, ConfirmConfig.CONFIRM_ROUTING_KEY + "22",message,correlationData2);


        log.info("发送消息成功：{}",message);
    }

}
```



发布确认配置类-----配置队列，交换机，绑定

```java
@Configuration
public class ConfirmConfig {

    public static final String CONFIRM_QUEUE_NAME = "confirm_queue";
    public static final String CONFIRM_EXCHANGE_NAME = "confirm_exchange";
    public static final String CONFIRM_ROUTING_KEY = "confirm_routingkey";

    @Bean("confirmQueue")
    public Queue confirmQueue(){
        return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }

    @Bean("ConfirmExchange")
    public DirectExchange confirmExchange(){
        return new DirectExchange(CONFIRM_EXCHANGE_NAME);
    }

    @Bean
    public Binding queueBindingExchange2(@Qualifier("confirmQueue") Queue queue,
                                        @Qualifier("ConfirmExchange") DirectExchange directExchange){
        return BindingBuilder.bind(queue).to(directExchange).with(CONFIRM_ROUTING_KEY);
    }

}
```



消息回调类-------当消息接收成功/失败时调用

```java
@Slf4j
@Component
public class MyCallback implements RabbitTemplate.ConfirmCallback,RabbitTemplate.ReturnsCallback {

    @Autowired
    private RabbitTemplate rabbitTemplate;
    @PostConstruct
    public void init(){
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setReturnsCallback(this);
    }

    @Override
    public void confirm(CorrelationData correlationData, boolean b, String s) {
        String id = correlationData == null ? "" : correlationData.getId();
        if (b){
            //消息接收确认
            log.info("交换机消息接收成功，消息的id为：{}",id);
        }else {
            log.info("消息接收失败，消息的id为：{}，失败原因：{}",id,s);
        }
    }

    @Override
    public void returnedMessage(ReturnedMessage returnedMessage) {
        log.error("消息发送到队列时出现错误，消息内容：{}，交换机：{}，路由键：{}，原因：{}",
                new String(returnedMessage.getMessage().getBody()),
                returnedMessage.getExchange(),
                returnedMessage.getRoutingKey(),
                returnedMessage.getReplyText());
    }
}
```



消息消费者

```java
@Component
@Slf4j
public class ConfirmConsumer {

    @RabbitListener(queues = ConfirmConfig.CONFIRM_QUEUE_NAME)
    public void confirmConsumer(Message message){
        String msg = new String(message.getBody());
        log.info("接收消息成功：{}",msg);
    }

}
```



### 3）备份交换机

![](https://github.com/vankykoo/image/blob/main/019.png?raw=true)

配置类

```java
@Configuration
public class ConfirmConfig {

    public static final String CONFIRM_QUEUE_NAME = "confirm_queue";
    public static final String CONFIRM_EXCHANGE_NAME = "confirm_exchange";
    public static final String CONFIRM_ROUTING_KEY = "confirm_routingkey";
    //备份交换机
    public static final String BACKUP_EXCHANGE_NAME = "backup_exchange";
    //备份队列
    public static final String BACKUP_QUEUE_NAME = "backup_queue";
    //警告队列
    public static final String WARN_QUEUE_NAME = "warning_queue";

    @Bean("confirmQueue")
    public Queue confirmQueue(){
        return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }

    @Bean("backupQueue")
    public Queue backupQueue(){
        return QueueBuilder.durable(BACKUP_QUEUE_NAME).build();
    }

    @Bean("warningQueue")
    public Queue warningQueue(){
        return QueueBuilder.durable(WARN_QUEUE_NAME).build();
    }

    @Bean("ConfirmExchange")
    public DirectExchange confirmExchange(){
        return ExchangeBuilder.directExchange(CONFIRM_EXCHANGE_NAME)
                .durable(true)
                .withArgument("alternate-exchange",BACKUP_EXCHANGE_NAME)
                .build();
    }

    @Bean("backupExchange")
    public FanoutExchange backupExchange(){
        return new FanoutExchange(BACKUP_EXCHANGE_NAME);
    }

    @Bean
    public Binding queueBindingExchange2(@Qualifier("confirmQueue") Queue queue,
                                        @Qualifier("ConfirmExchange") DirectExchange directExchange){
        return BindingBuilder.bind(queue).to(directExchange).with(CONFIRM_ROUTING_KEY);
    }

    @Bean
    public Binding backupQueueBindingBackupExchange(@Qualifier("backupQueue")Queue queue,
                                                    @Qualifier("backupExchange")FanoutExchange fanoutExchange){
        return BindingBuilder.bind(queue).to(fanoutExchange);
    }

    @Bean
    public Binding warningQueueBindingBackupExchange(@Qualifier("warningQueue")Queue queue,
                                                    @Qualifier("backupExchange")FanoutExchange fanoutExchange){
        return BindingBuilder.bind(queue).to(fanoutExchange);
    }

}
```





## 十一、幂等性问题

在实际的开发项目中，一个对外暴露的接口往往会面临很多次请求。这就需要考虑到一个幂等性问题。
幂等性的概念是：任意多次执行所产生的影响均与一次执行的影响相同，即无论你请求了多少次，对数据库的影响都只能有一次，不能重复处理。例如，以下三条SQL语句中，只有第三条是非幂等的：

- `SELECT col1 FROM tab1 WHERE col2=2`，无论执行多少次都不会改变状态，是天然的幂等。
- `UPDATE tab1 SET col1=1 WHERE col2=2`，无论执行成功多少次状态都是一致的，因此也是幂等操作。
- `UPDATE tab1 SET col1=col1+1 WHERE col2=2`，每次执行的结果都会发生变化，这种不是幂等的。



**在RabbitMQ中，如果我们希望实现幂等性，可以采用以下几种策略：**

- **防重表**：数据库建立唯一性索引，可以保证最终插入数据库的只有一条数据。
- **Token令牌机制**：每次接口请求前先获取一个token，然后再下次请求的时候在请求的header体中加上这个token，后台进行验证，如果验证通过删除token。
- **先查询后判断**：首先通过查询数据库是否存在数据，如果存在证明已经请求过了，直接拒绝该请求。
- **支付缓冲区**：把订单的支付请求都快速地接下来，一个快速接单的缓冲管道。后续使用异步任务处理管道中的数据，过滤掉重复的待支付订单。
- **悲观锁或者乐观锁**：悲观锁可以保证每次for update的时候其他sql无法update数据。乐观锁一般通过version来做乐观锁。
- **分布式锁**：防重表可以使用分布式锁代替，比如Redis

以上就是关于RabbitMQ中幂等性问题的一些解决方案。具体采用哪种方案需要根据你的业务需求来决定。



## 十二、优先级队列

RabbitMQ的优先级队列是从3.5.0版本开始实现的。任何一个队列都可以通过客户端配置参数方式设置一个优先级，但是不能使用策略的方式配置这个参数。当前优先级的最大值为：255，这个值最好在1到10之间。

要声明一个优先级队列，可以使用`x-max-priority`可选队列参数。这个参数应该是1和255之间的正整数，表示队列应支持的最大优先级。例如：

```java
Channel ch = ...;
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-max-priority", 10);
ch.queueDeclare("my-priority-queue", true, false, false, args);
```

然后，发布者可以使用`basic.properties`的优先级字段发布优先级消息。数字越大表示优先级越高。

当消费者阻塞的时候，对具有优先级的消息直接按照优先级排序操作，然后按照优先级在一个一个的发送给消费者。这里需要多个条件 (优先级队列、优先级消息、消费者阻塞、并且server对消费者排序)。

在大多数情况下，您将需要在使用者的手动确认模式下使用`basic.qos`方法，以限制可以随时传递的邮件数量，从而使邮件具有优先级。



消息生产者：

```java
public class producer {

    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        //创建一个连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        //通过工厂IP 连接RabbitMQ的队列
        connectionFactory.setHost("192.168.200.138");
        //设置用户名密码
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("123456");
        //创建连接
        Connection connection = connectionFactory.newConnection();
        //获取信道
        Channel channel = connection.createChannel();
        /**
         * 创建一个队列：参数说明
         * 1.队列名称
         * 2.是否需要持久化
         * 3.是否只供一个消费者消费，是否进行消息共享
         * 4.是否自动删除
         * 5.其他参数
         */
        //设置优先级
        Map<String, Object> map = new HashMap<>();
        //优先级范围设置为0-10
        map.put("x-max-priority",10);
        channel.queueDeclare(QUEUE_NAME,true,false,false,map);

        //发送一个消息
        //String message = "hello world!";
        for (int i = 0; i < 10; i++) {
            String message = "info" + i;
            if (i == 5){
              //当i为5的时候，生产一条带有优先级为5的消息
                AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();
                channel.basicPublish("",QUEUE_NAME,properties,message.getBytes());
            }
            channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
        }


        System.out.println("消息发送成功！");
    }

}
```



## 十三、惰性队列

RabbitMQ从3.6.0版本开始引入了惰性队列（Lazy Queue）的概念。惰性队列会尽可能的**将消息存入磁盘中**，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要**设计目标是能够支持更长的队列**，即支持更多的消息存储。当消费者由于各种各样的原因（比如消费者下线、跌机、或者由于维护而关闭等）致使长时间不能消费消息而造成堆积时，惰性队列就很必要了。

默认情况下，当生产者将消息发送到RabbitMQ的时候，队列中的消息会尽可能地存储在内存之中，这样可以更加快速地将消息发送给消费者。即使是持久化的消息，在被写入磁盘的同时也会在内存中驻留一份备份。当RabbitMQ需要释放内存的时候，会将内存中的消息换页至磁盘中，这个操作会耗费较长的时间，也会阻塞队列的操作，进而无法接收新的消息。

惰性队列会将接收到的消息直接存入文件系统中，而不管是持久化的或者是非持久化的，这样可以减少了内存的消耗，但是会增加I/O的使用，如果消息是持久化的，那么这样的I/O操作不可避免，惰性队列和持久化消息可谓是“最佳拍档”。注意如果惰性队列中存储的是非持久化的消息，内存的使用率会一直很稳定，但是重启之后消息一样会丢失。

队列具备两种模式：**default和lazy**。默认为default模式，在3.6.0之前的版本无需做任何变更。lazy模式即为惰性队列的模式，可以通过调用channel.queueDeclare方法的时候在参数中设置，也可以通过Policy的方式设置。

如果要通过声明的方式改变已有队列的模式，那么只能先删除队列，然后再重新声明一个新的。例如：

```java
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-queue-mode", "lazy");
channel.queueDeclare("myqueue", false, false, false, args);
```

惰性队列和普通队列相比，只有很小的内存开销。这里很难对每种情况给出一个具体的数值，但是我们可以类比一下：当发送1千万条消息，每条消息大小为1KB，并且此时没有任何消费者时，普通队列会消耗1.2GB内存，而惰性队列只消耗1.5MB内存。



## 十四、Rabbitmq集群

### 1）介绍

RabbitMQ集群是一种消息队列中间件产品，它是基于Erlang编写的，Erlang语言天生具备分布式特性（通过同步Erlang集群各节点的magic cookie来实现）。因此，RabbitMQ天然支持Clustering。这使得RabbitMQ本身不需要像ActiveMQ、Kafka那样通过ZooKeeper分别来实现HA方案和保存集群的元数据。集群是保证可靠性的一种方式，同时可以通过水平扩展以达到增加消息吞吐量能力的目的。

RabbitMQ有以下几种集群模式：

1. **普通集群模式**：在这种模式下，RabbitMQ集群同步的只是复制队列，元数据信息的同步，即同步的是数据存储信息；消息的存放只会存储在创建该消息队列的那个节点上。并非在节点上都存储一个完整的数据。在通过非数据所在节点获取数据时，通过元数据信息，路由转发到存储数据节点上，从而得到数据。
2. **镜像集群模式**：与普通集群模式区别主要是消息实体会主动在镜像节点间同步数据，而不是只存储数据元信息。故普通集群模式但凡数据节点挂了，容易造成数据丢失但镜像集群模式可以保证集群只要不全部挂掉，数据就不会丢失。
3. **远程模式**：远程模式：远距离通信和复制，所谓Shovel就是我们可以把消息进行不同数据中心的复制工作，我们可以跨地域的让两个mq集群互联。
4. **多活模式**：这种模式也是实现异地数据复制的主流模式，因为Shovel模式配置比较复杂，所以一般来说实现异地集群都是使用双活或者多活模式来实现的。这种模式需要依赖rabbitmq的federation插件，可以实现继续的可靠AMQP数据通信。

以上就是RabbitMQ集群的一些基本概念和配置方式。在进行操作之前，请确保你已经对相关知识有了充分的了解，并且已经做好了数据备份。



### 2）搭建

在Linux环境下，你可以按照以下步骤来搭建RabbitMQ集群：

1. **环境准备**：你需要准备至少两台服务器，每台服务器上都安装有RabbitMQ和Erlang。

2. **配置Erlang Cookie**：RabbitMQ集群中的每个节点都需要有相同的Erlang Cookie。这个Cookie相当于密钥令牌，集群中的RabbitMQ节点需要通过交换密钥令牌以获得相互认证。你可以在每个节点上设置相同的Erlang Cookie。

3. **配置主机名和IP映射**：在每台服务器的`/etc/hosts`文件中添加所有参与集群的服务器的IP和主机名的映射。

4. **启动RabbitMQ服务**：在每个节点上启动RabbitMQ服务。

5. **创建RabbitMQ集群**：选择一台服务器作为主节点，然后在这台服务器上使用`rabbitmqctl`命令将其他服务器加入到集群中。例如，如果你想要将名为rabbit2和rabbit3的节点加入到rabbit1节点创建的集群中，你可以在rabbit1节点上执行以下命令：

   `rabbitmqctl stop_app`

   `rabbitmqctl reset`

   `rabbitmqctl start_app`

   `rabbitmqctl cluster_status`

   然后在rabbit2和rabbit3节点上分别执行以下命令：

   `rabbitmqctl stop_app`

   `rabbitmqctl reset`

   `rabbitmqctl join_cluster rabbit@rabbit1`

   `rabbitmqctl start_app`

6. **查看集群状态**：你可以在任何一个节点上使用`rabbitmqctl cluster_status`命令来查看集群状态。

以上就是搭建RabbitMQ集群的基本步骤。请注意，在进行操作之前，请确保你已经对相关知识有了充分的了解，并且已经做好了数据备份。如果问题依然存在，建议寻求专业人士的帮助。



### 3）镜像队列

RabbitMQ的镜像队列是一种高可用性解决方案，它通过将一个队列镜像（消息广播）到其他节点的方式来提升消息的高可用性。当主节点宕机，从节点会提升为主节点继续向外提供服务。

每个镜像队列都包含一个**主节点（Leader）和若干个从节点（Follower）**，其中只有主节点向外提供服务（生产消息和消费消息），从节点仅仅接收主节点发送的消息。从节点会准确地按照主节点执行命令的顺序执行动作，所以从节点的状态与主节点应是一致的。

配置镜像队列规则后，新创建的队列，如果队列名称符合规则，则该队列会自动创建为镜像队列。你可以使用策略（Policy）来配置镜像策略，策略使用正则表达式来配置需要应用镜像策略的队列名称，以及在参数中配置镜像队列的具体参数。

在3.8之后的版本中，RabbitMQ推出了Quorum queues来替代镜像队列，在之后的版本中镜像队列将被移除。但是在3.8之前的版本中，RabbitMQ通过镜像队列来提供高可用性。

以上就是RabbitMQ镜像队列的基本概念和配置方式。在进行操作之前，请确保你已经对相关知识有了充分的了解，并且已经做好了数据备份。



配置方法：

![](https://github.com/vankykoo/image/blob/main/020.png?raw=true)



### 4）实现高可用负载均衡

在Linux环境下，你可以按照以下步骤来使用HAProxy和Keepalived实现RabbitMQ的负载均衡：

1. **环境准备**：你需要准备至少两台服务器，每台服务器上都安装有RabbitMQ、Erlang、HAProxy和Keepalived。

2. **配置HAProxy**：在每台机器上配置HAProxy。你需要在`/etc/haproxy/haproxy.cfg`文件中添加RabbitMQ节点的信息。例如，如果你有两个RabbitMQ节点，IP地址分别为192.168.1.1和192.168.1.2，那么你可以添加以下配置：

       listen  rabbitmq_cluster
         bind 0.0.0.0:5672
         mode tcp
         balance roundrobin
         server rabbit1 192.168.1.1:5672 check inter 5000 rise 2 fall 3
         server rabbit2 192.168.1.2:5672 check inter 5000 rise 2 fall 3

3. **启动HAProxy服务**：在每个节点上启动HAProxy服务。

4. **配置Keepalived**：在每台机器上配置Keepalived。你需要在`/etc/keepalived/keepalived.conf`文件中添加虚拟IP地址和其他相关信息。例如，如果你想要设置虚拟IP地址为192.168.1.100，那么你可以添加以下配置：

   ````
   vrrp_instance VI_1 {
       state MASTER
       interface eth0
       virtual_router_id 51
       priority 100
       advert_int 1
       authentication {
           auth_type PASS
           auth_pass 1111
       }
       virtual_ipaddress {
           192.168.1.100
       }
   }
   ````

5. **启动Keepalived服务**：在每个节点上启动Keepalived服务。

6. **验证搭建结果**：最后，你可以使用`ip addr show`命令来查看虚拟IP地址是否已经生效。

以上就是使用HAProxy和Keepalived实现RabbitMQ负载均衡的基本步骤。请注意，在进行操作之前，请确保你已经对相关知识有了充分的了解，并且已经做好了数据备份。如果问题依然存在，建议寻求专业人士的帮助。



### 5）联邦交换机

RabbitMQ的Federation交换机是一种特殊的交换机，它可以在不同的Broker节点或者集群中进行消息的无缝传递。这种机制主要是**为了解决在物理距离较远的RabbitMQ节点之间进行消息传递时可能出现的网络延迟和消息丢失问题**。

Federation插件基于AMQP 0-9-1协议在不同的Broker之间进行通信，并设计成能够容忍不稳定的网络连接情况。一个联邦交换器可以接收来自一个或多个上游（位于其他Broker上的交换器和队列）的消息。**联邦交换器能够将原本发送给上游交换器的消息路由到本地的某个队列中。**

在使用Federation交换机时，你需要配置两个东西：一个或多个定义了到其他节点的联邦连接的上游，以及一个或多个选择交换器/队列并将单个上游或上游集应用于这些对象的策略。

以上就是RabbitMQ Federation交换机的基本概念和使用方式。在进行操作之前，请确保你已经对相关知识有了充分的了解，并且已经做好了数据备份。

![](https://github.com/vankykoo/image/blob/main/021.png?raw=true)



#### 搭建方法

在Linux环境下，你可以按照以下步骤来搭建RabbitMQ的Federation交换机：

1. **环境准备**：你需要准备至少两台服务器，每台服务器上都安装有RabbitMQ和Erlang。
2. **开启Federation插件**：在每台机器上开启federation相关插件。你可以使用命令`rabbitmq-plugins enable rabbitmq_federation`和`rabbitmq-plugins enable rabbitmq_federation_management`来开启插件。
3. **创建交换器和队列**：在一台机器上新建一个队列和一个交换器，并将他们之间进行绑定。你可以使用命令行或者管理后台自行新建。
4. **配置Federation**：
5. 先添加upstream![](https://github.com/vankykoo/image/blob/main/022.png?raw=true)

然后添加policy

![](https://github.com/vankykoo/image/blob/main/023.png?raw=true)



5. **验证搭建结果**：最后，你可以在RabbitMQ的管理界面中查看Federation Status来验证是否搭建成功。

以上就是搭建RabbitMQ的Federation交换机的基本步骤。请注意，在进行操作之前，请确保你已经对相关知识有了充分的了解，并且已经做好了数据备份。





### 6）联邦队列

RabbitMQ的Federation队列是一种特殊的队列，它可以在不同的Broker节点或者集群中进行消息的无缝传递。这种机制主要是**为了解决在物理距离较远的RabbitMQ节点之间进行消息传递时可能出现的网络延迟和消息丢失问题**。

Federation插件基于AMQP 0-9-1协议在不同的Broker之间进行通信，并设计成能够容忍不稳定的网络连接情况。一个联邦队列可以接收来自一个或多个上游（位于其他Broker上的交换器和队列）的消息。联邦队列允许本地消费者接收来自上游队列的消息。

在使用Federation队列时，你需要配置两个东西：一个或多个定义了到其他节点的联邦连接的上游，以及一个或多个选择交换器/队列并将单个上游或上游集应用于这些对象的策略。

以上就是RabbitMQ Federation队列的基本概念和使用方式。在进行操作之前，请确保你已经对相关知识有了充分的了解，并且已经做好了数据备份。![](https://github.com/vankykoo/image/blob/main/024.png?raw=true)



#### 搭建过程

在Linux环境下，你可以按照以下步骤来搭建RabbitMQ的Federation队列：

1. **环境准备**：你需要准备至少两台服务器，每台服务器上都安装有RabbitMQ和Erlang。

2. **开启Federation插件**：在每台机器上开启federation相关插件。你可以使用命令`rabbitmq-plugins enable rabbitmq_federation`和`rabbitmq-plugins enable rabbitmq_federation_management`来开启插件。

3. **创建交换器和队列**：在一台机器上新建一个队列和一个交换器，并将他们之间进行绑定。你可以使用命令行或者管理后台自行新建。

4. **配置Federation**：

   先添加upstream，方法和上面一样

   再添加policy![](https://github.com/vankykoo/image/blob/main/025.png?raw=true)

5. **验证搭建结果**：最后，你可以在RabbitMQ的管理界面中查看Federation Status来验证是否搭建成功。

以上就是搭建RabbitMQ的Federation队列的基本步骤。请注意，在进行操作之前，请确保你已经对相关知识有了充分的了解，并且已经做好了数据备份。如果问题依然存在，建议寻求专业人士的帮助。



### 7）Shovel

RabbitMQ的Shovel插件是一种允许你配置多个shovel（传输工作器），这些工作器可以在RabbitMQ集群启动时自动启动，从一个源（通常是一个队列）中取出消息，然后将其发布到另一个集群的目标（例如交换器、主题等）。Shovel插件基于AMQP协议在broker间进行通信，并被设计成能够容忍不稳定的网络连接。

**以下是使用Shovel插件的步骤：**

1. **开启Shovel插件**：在每台机器上开启federation相关插件。你可以使用命令`rabbitmq-plugins enable rabbitmq_shovel`和`rabbitmq-plugins enable rabbitmq_shovel_management`来开启插件。
2. **创建交换器和队列**：在一台机器上新建一个队列和一个交换器，并将他们之间进行绑定。你可以使用命令行或者管理后台自行新建。
3. **配置Shovel**：![img](https://github.com/vankykoo/image/blob/main/026.png?raw=true)
4. **验证搭建结果**：最后，你可以在RabbitMQ的管理界面中查看Federation Status来验证是否搭建成功[1](https://blog.csdn.net/sanmi8276/article/details/114628605)。

以上就是使用RabbitMQ的Shovel插件的基本步骤。请注意，在进行操作之前，请确保你已经对相关知识有了充分的了解，并且已经做好了数据备份。
