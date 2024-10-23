

<h1 id="BtFLk">1）服务提供者启动流程</h1>
<h2 id="BbnnG">RPC 框架初始化并启动服务器</h2>
![](https://cdn.nlark.com/yuque/0/2024/png/38627688/1728737172626-a3c12e1f-f5a0-45a5-b309-2f05f9046aa3.png)

1. 服务提供者为 springboot 项目，主启动类标注了 `@EnableRpc` 注解，标识开启 Rpc功能。
2. 项目初始化时，会执行`RpcInitBootstrap -> registerBeanDefinitions()`，进行初始化 Rpc 框架；
    1. 获取服务提供者项目的 yml 配置，支持自定义 RPC 配置。
    2. 根据 RPC 配置（包含注册中心的配置）进行初始化。
3. RPC 框架初始化`RpcApplication.init(userRpcConfig)`：
    1. 注册中心初始化：
        1. 通过 rpc 配置获取注册中心类型（当前框架支持 etcd/zookeeper）
          2. 通过注册中心工厂类获取注册中心实例 											`RegistryFactory.getInstance(registryConfig.getRegistry())`
        2. 执行<font style="color:#DF2A3F;">初始化方法</font>`registry.init(registryConfig)`
    2. 创建并注册 Shutdown Hook，JVM 退出时执行注册中心服务注销的工作。
4. 启动 Vert.x 服务器`vertxTcpServer.doStart(rpcConfig.getServerPort())`：
    1. 创建 Vertx 实例
    2. 创建 TCP 服务器
    3. 绑定<font style="color:#DF2A3F;">处理请求的处理器</font>：`server.connectHandler(new TcpServerHandler());`
    4. 启动 TCP 服务器并监听指定端口



<h2 id="eqodq">服务注册</h2>
![](https://cdn.nlark.com/yuque/0/2024/png/38627688/1728738057204-cfc29eda-5090-4d4d-952f-97d0ef45ba76.png)

`RpcProviderBootstrap -> postProcessAfterInitialization()` 每注册一个 Bean 后会执行这个方法：



1. 服务提供者项目在服务的实现类中使用 `@RpcService` 注解，表示为 rpc 服务。
2. 在 `postProcessAfterInitialization()` 方法中，检测这个 Bean 有没有 `@RpcService` 注解。
3. 如果有，就把当前实现类注册到注册中心中。
    1. 获取服务信息：
        1. 该服务实现的接口全类名、版本号：用于注册中心的 key。  
          》》》键名格式：`(全类名):(版本)/(服务地址):(端口)`
    2. 注册服务：
        1. 根据配置获取注册中心实现类
        2. 根据服务的基本信息<font style="color:#DF2A3F;">将服务注册到注册中心</font>`registry.register(serviceMetaInfo);`



<h1 id="wtDek">2）注册中心</h1>
接口设计：

1. init 初始化方法
2. register 服务注册方法
3. unRegister 服务注销
4. serviceDiscovery 获取一个服务的所有节点
5. destory 所有节点下线，释放资源
6. heartbeat 心跳检测（服务端），续期
7. watch 监听（消费端），注册中心服务变动时触发



以 etcd 为例：

<h2 id="hUnQ7">init</h2>
通过 jtcd 的 api 连接到 etcd 服务端，启动心跳检测机制。

<h2 id="QxpHK">register</h2>
1. 创建一个<font style="color:#DF2A3F;">租约Client</font>，可以为服务绑定过期时间，比如30s。
2. 然后通过 KvClient 存入到etcd。
3. 保存到本地缓存的一个 set 中，用于心跳检测。

<h2 id="a7odk">unRegister</h2>
1. 通过 kvClient 在 etcd 中删除。
2. 在本地缓存中删除。



<h2 id="Flzxv">serviceDiscovery</h2>
1. 优先从本地缓存中获取，如果有就直接返回。
2. 然后通过前缀搜索，获取一个服务的所有节点。
3. 解析服务信息：
    1. 获取每一个服务的 key
    2. 调用 watch 方法监听该 key
    3. 收集为一个新 list
4. 更新服务缓存



<h2 id="wQIVg">destory</h2>
1. 删除该服务的所有节点
2. 释放资源



<h2 id="CRqOZ">heartbeat</h2>
使用 Hutool 工具类的定时任务调度器，通过 cron 表达式创建定时任务。

1. 遍历所有节点
2. 判断节点有没有过期，没有过期就续期
3. 然后注册回注册中心



<h2 id="DhRwv">watch</h2>
使用 WatchClient，可以监听键的删除/更新

这里只处理了键被删除的情况：删除本地缓存。





<h1 id="uV1bz">3）服务器的请求处理器（TcpServerHandler）</h1>
为解决 TCP 消息的粘包半包问题，还需要一个 TcpBufferHandlerWrapper 来增强原有的 BufferHandler 对 buffer 的处理能力。（装饰者模式）

<h2 id="yfkf8">TcpBufferHandlerWrapper</h2>
使用了 Vert.x 框架中内置的 <font style="color:#DF2A3F;">RecordParser</font> 完美解决半包粘包，它的作用是：<font style="color:#DF2A3F;">保证下次读取到特定长度的字符。</font>因为消息体的长度是不固定的，所以我设计 RPC 时要通过调整 RecordParser 的固定长度（变长）来解决。 那我的思路是，将读取完整的消息拆分为 2 次：

1）先完整读取请求头信息，由于请求头信息长度是固定的，可以使用 RecordParser保证每次都完整读取。

2）再<font style="color:#DF2A3F;">根据请求头长度信息</font>更改 RecordParser的固定长度，保证完整获取到请求体。



<font style="color:#DF2A3F;">在该类的初始化方法中，是需要把 bufferHandler 当作参数传进来的，当读取到完整的请求体后，用 bufferHandler 处理消息。</font>

<font style="color:#DF2A3F;">所以在 TcpServerHandler 的 handle 方法中，在初始化 TcpBufferHandlerWrapper 时，需要传入一个 BufferHandler。</font>



<h2 id="E4Z8L">BufferHandler 方法</h2>
1. 接收到请求（buffer格式），使用<font style="color:#DF2A3F;">协议消息解码器</font>进行解码，转为 Java 对象 ProtocolMessage<RpcRequest>，获取消息体 RpcRequest。
2. 通过反射机制获取实现类并创建实例，调用方法，得到返回结果。
3. 封装方法执行返回结果对象，封装响应结果对象得到协议消息对象。
4. 对协议消息对象进行编码，并发出响应。



<h1 id="yUNal">4）协议及其编解码器</h1>
<h2 id="rr9en">协议定义</h2>
1. 消息头
    1. 魔数(byte)：保证消息安全性
    2. 版本号(byte)：方便协议升级
    3. 序列化器(byte)：
    4. 消息类型(byte)：请求/响应
    5. 状态(byte)：
    6. 请求id(long)：
    7. 消息体长度(int)：
2. 消息体



<h2 id="zPQRW">协议消息解码编码</h2>
<h3 id="dDHQG">编码</h3>
1. 根据规定顺序将消息头的字段写入缓冲区
2. 获取指定序列化器，将消息体进行序列化
3. 写入消息体长度，写入消息体

<h3 id="NTf8H">解码</h3>
1. 检查魔数是否合法
2. 依次获取指定位置消息头内容。根据消息体长度获取完整消息体
3. 获取解码器，对消息体解码。
4. 根据消息类型封装消息。



<h1 id="oymjm">5）服务消费者启动流程</h1>
消费者在其项目调用到服务的地方，需要注入服务实现类的实例对象，用`@RpcReference` 注解注入。

```java
@Service
public class ExampleServiceImpl {

    // 注入服务
    @RpcReference
    private UserService userService;

    public void test() {
        User user = new User();
        user.setName("vanky");
        User resultUser = userService.getUser(user);
        System.out.println(resultUser.getName());
    }

}
```

服务消费者也需要在 springboot 主启动类中加上 `@EnableRpc` 注解，但是不需要启动服务器。

即`@EnableRpc(needServer = false)`。



spring-boot-starter 中有一个方法，和服务提供者一样，也是在一个 Bean 创建完后执行的方法。

1. 当 Bean (服务消费者的需要用到服务的类）初始化完成后， 执行`RpcConsumerBootstrap -> postProcessAfterInitialization` 方法。
2. 在该方法中，获取对象的所有属性，并遍历所有属性。
3. 检查属性是否带有 `@RpcReference` 注解。
4. 如果有，为该属性生成代理对象。  
  `Object proxyObject = ServiceProxyFactory.getProxy(interfaceClass);`





<h1 id="srSul">6）服务调用过程</h1>
使用到了<font style="color:#DF2A3F;">动态代理 + 工厂</font>。

1. 在服务消费者端注入服务的实现类实例时，用到了动态代理工厂生成代理对象。

```java
public static <T> T getProxy(Class<T> serviceClass){
    if (RpcApplication.getRpcConfig().isMock()){
        return getMockProxy(serviceClass);
    }

    T t = (T) Proxy.newProxyInstance(
            serviceClass.getClassLoader(), // 类加载器
            new Class[]{serviceClass}, // 代理类要实现的接口
            new ServiceProxy());	// 代理对象方法被调用时，ServiceProxy 来处理

    return t;
}
```

2. 调用代理对象的方法时，交给 ServiceProxy (InvocationHandler) 处理。

>  在 Java 的动态代理机制中，当你创建了一个代理对象并调用它的方法时，这些方法调用都会被 `InvocationHandler` <font style="color:#DF2A3F;">拦截并处理</font>。`InvocationHandler` 的作用是定义代理对象如何处理对其方法的调用，而<font style="color:#DF2A3F;">不是直接调用被代理的目标对象</font>。  
>

3. 以下是 ServiceProxy 重新 invoke 方法的逻辑：
    1. 构造一个 RpcRequest：实现的接口名、方法名、参数类型、参数。
    2. 根据配置信息获取注册中心，在注册中心中获取服务列表（一个服务可能有多个节点）。
    3. 根据配置信息获取<font style="color:#DF2A3F;">负载均衡策略</font>，将服务列表传入负载均衡器中，获取到一个服务节点。
    4. 根据配置信息获取<font style="color:#DF2A3F;">重试机制</font>策略，把发送请求并处理响应的操作放入<font style="color:#DF2A3F;">重试机制</font>中执行。

```java
retryStrategy.doRetry(() ->
    // 发送请求并处理响应
    VertxTcpClient.doRequest(rpcRequest, selectedServiceMetaInfo)
);
```

    5. 通过 try-catch 把执行请求的代码段包起来，在 catch 代码块中，获取<font style="color:#DF2A3F;">容错机制</font>策略，当程序出错时，执行容错机制代码。
    6. 最后返回方法调用的结果。



<h1 id="U1Hw0">7）优化</h1>
通过负载均衡、请求重试、容错机制，对系统可用性，服务调用处理过程进行优化。确保系统稳定性和性能。



这三个机制/策略都是通过 <font style="color:#DF2A3F;">SPI + 工厂模式 </font>实现的。相关文章：[https://vankykoo.cn/index.php/308/](https://vankykoo.cn/index.php/308/)

<h2 id="cXaGJ">负载均衡</h2>
好处：通过实现负载均衡将请求分发到多个服务器，即使我们的部分服务器出现故障，系统仍然可以继续提供服务，从而提高了系统的整体可用性和可靠性。负载均衡能够合理分配客户端请求或网络流量到多个服务器，避免单个服务器因负载过重而成为瓶颈，从而提升整体系统性能。

接口方法：

```java
/**
 * 选择服务调用
 * @param requestParams         请求参数
 * @param serviceMetaInfoList   可用服务列表
 * @return
 */
ServiceMetaInfo select(Map<String, Object> requestParams, List<ServiceMetaInfo> serviceMetaInfoList);
```

实现类：

1）轮询（Round Robin）：按照循环的顺序将请求分配给每个服务器，适用于各服务器性能相近的情况。

实现方式：记录轮询编号。

2）随机（Random）：随机选择一个服务器来处理请求，适用于服务器性能相近且负载均匀的情况。

实现方式：随机数取模。

3）<font style="color:#DF2A3F;">一致性 Hash</font>（Consistent Hash）：将服务器和键同时映射到一个虚拟的哈希环上，并且按顺时针的顺序将键分配给距离它最近的服务器。

实现方式：`TreeMap`的`ceilingEntry`方法和`firstEntry`方法。



<h2 id="FQnYZ">重试机制</h2>
接口方法：传入一个可调用的方法，将执行请求的方法在此次调用。

```java
/**
 * 重试
 * @param callable
 * @return
 * @throws Exception
 */
RpcResponse doRetry(Callable<RpcResponse> callable) throws Exception;
```

实现：

1. 不重试
2. 固定重试间隔（Fixed Retry Interval）：在每次重试之间使用固定的时间间隔。

实现方式：通过引入依赖，进行重试。

```xml
<!-- https://github.com/rholder/guava-retrying -->
<dependency>
  <groupId>com.github.rholder</groupId>
  <artifactId>guava-retrying</artifactId>
  <version>2.0.0</version>
</dependency>
```



<h2 id="xHPTv">容错机制</h2>
接口方法：

```java
/**
 * 容错
 * @param context   上下文，用于传递数据
 * @param e         异常
 * @return
 */
RpcResponse doTolerant(Map<String, Object> context, Exception e);
```

实现：

1. Fail-Over 故障转移：一次调用失败后，切换一个其他节点再次进行调用，也算是一种重试。
2. Fail-Safe 静默处理：系统出现部分非重要功能的异常时，直接忽略掉，不做任何处理，就像错误没有发生过一样。
3. Fail-Fast 快速失败：系统出现调用错误时，立刻报错，交给外层调用方处理。



