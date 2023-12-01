# Redis高级

Redis常见用法：

数据共享（分布式Session）、分布式锁、全局ID、计算器、点赞、位统计、购物车、轻量级消息队列、抽奖、点赞、签到、打卡、差集交集并集（用户关注、可能认识的人，推荐模型）、热点新闻、热搜排行榜。



## 一、单线程/多线程（入门）

### 1）Redis4.0之前为什么选择单线程

1. 使用单线程模型是 Redis 的开发和维护更简单，因为单线程模型方便开发和调试；
2. 即使使用单线程模型也并发的处理多客户端的请求，主要使用的是IO多路复用和非阻塞IO；
3. 对于Redis系统来说，主要的性能瓶颈是内存或者网络带宽而并非 CPU。

> **面试：redis是单线程的还是多线程的？**
>
> **回答：**
>
> 这个问题其实并不严谨，Redis是否是单线程还是多线程需要视版本而定。
>
> - 对于Redis的3.x版本（最早版本），也就是大家口口相传的Redis是单线程的。
> - 对于4.x版本，严格意义来说也不是单线程，而是负责处理客户端请求的线程单线程，但是开始加了点多线程的东西（异步删除）。
> - 从6.x版本开始，Redis全面支持多线程。
>
> 所以，当我们说Redis是单线程的，我们主要是指**Redis的网络IO和键值对读写是由一个线程来完成的**。在处理客户端的请求时，包括获取（socket读）、解析、执行、内容返回（socket写）等都由一个顺序串行的主线程处理。但是，其他功能，比如持久化RDB、AOF、异步删除、集群同步数据等都是由额外的线程执行的，因此整个Redis可以看作是多线程的。
>
> 另外，值得一提的是，尽管Redis在4.0之前一直采用单线程模型，但其性能仍然非常高。这主要归功于以下几点：
>
> - Redis基于内存操作，内存的读写速度非常快；
> - Redis使用了简单的数据结构（如HashMap），这些数据结构的查找和操作时间复杂度都是O(1)；
> - Redis使用了IO多路复用技术，可以处理并发连接；
> - 单线程模型避免了频繁的上下文切换。



###2）redis为什么逐渐加入多线程

Redis在4.0版本之前一直采用单线程，主要原因有以下几点：

- 使用单线程模型使Redis的开发和维护更简单
- 虽然使用的是单线程，但也可以并发处理多客户端的请求（IO多路复用和非阻塞IO）
- 对于Redis系统来说，主要的性能瓶颈是内存/网络带宽，而非CPU

然而，随着计算机硬件的发展，多核CPU已经成为常态，**为了更大限度的利用CPU**，Redis开始逐渐加入多线程特性。此外，Redis使用单线程也存在一定的缺点，比如**使用del指令删除大key数据时会造成主线程卡顿**。因此，Redis在4.0版本开始引入了多线程模块，主要是为了解决这个问题。

在Redis 6/7中，Redis全面支持了多线程。这是由于随着硬件性能的提升，**Redis的性能瓶颈主要出现在网络IO上**，就是完全靠单个主线程处理网络请求的速度跟不上底层网络硬件的速度，于是采用多个线程处理网络IO，提高网络请求处理的并行度。但是，Redis的多IO线程只是用来处理网络请求的，对于读写操作命令Redis仍然使用单线程来处理。这是因为，Redis处理请求时，网络处理经常是瓶颈，通过多个IO线程并行处理网络操作，可以提升实例的整体处理性能。而继续使用单线程执行命令操作，则不需要为了保证Lua脚本、事务的原子性而额外开发多线程互斥加锁机制。



### 3）主线程和IO线程怎么协作完成请求处理的

![](https://github.com/vankykoo/image/blob/main/072.png?raw=true)

四个阶段：

- 阶段一：服务端和客户端建立Socket连接，并分配处理线程 首先，**主线程**负责接收**建立连接请求**，当有客户端请求和实例建立Socket连接时，主线程会**创建和客户端的连接**，**并把Socket放入全局等待队列中**，紧接着，**主线程通过轮询方法把Socket连接分配给IO线程**。
- 阶设二：IO线程读取并解折请求 主线程一旦把Socket分配给IO线程，就会进入阻塞状态，等待IO线程完成客户端请求的读取和解析，因为有多个IO线程在并行处理，所以，这个过程很快就可以完成。
- 阶段三：主线程执行请求操作 等到IO线程解析完请求，**主线程**还是会以单线程的方式**执行这些命令操作。**
- 阶设四：IO线程回写Socket和主线程清空全局队列 当**主线程执行完请求操作后，会把需要返回的结果写入缓冲区**，然后，**主线程会阻塞等待IO线程**，把这些结果回写到Socket中，并返回给客户端。和IO线程读取和解析请求一样，IO线程回写Socket时，也是有多个线程在并发执行，所以回写Socket的速度也很快。等到IO线程回写Socket完毕，主线程会清空全局队列，等待客户端的后续请求。



### 4）IO多路复用和epoll函数

IO多路复用是一种同步IO模型，它允许单个进程/线程同时处理多个IO请求。当我们调用一定的方法后，内核才对所有监视的文件描述符进行扫描。在我们调用epoll_wait()获取就绪文件描述符时，返回的不是实际的描述符，而是一个代表就绪描述符数量的值。

epoll是Linux下性能最好的多路I/O就绪通知方法。它只告知那些就绪的文件描述符，而且当我们调用epoll_wait()获得就绪文件描述符时，返回的不是实际的描述符，而是一个代表就绪描述符数量的值。你只需要去epoll指定的一个数组中依次取得相应数量的文件描述符即可。

epoll主要有三个基本函数：epoll_create，epoll_ctl，epoll_wait：

- `epoll_create`函数创建一个epoll对象，并返回该对象的描述符。
- `epoll_ctl`函数操作控制epoll对象，主要涉及epoll红黑树上节点的一些操作，比如添加节点，删除节点，修改节点事件。
- `epoll_wait`函数阻塞一段时间并等待事件发生，返回事件集合，也就是获取内核的事件通知。

总的来说，IO多路复用和epoll函数提供了一种高效的方式来处理大量的socket，并且随着socket数量增加，它们的性能不会下降。这使得它们在构建高性能网络服务器时非常有用。这些技术可以帮助服务器在处理大量并发连接时保持高效和稳定。



### 5）redis为什么那么快

Redis之所以快，主要有以下几个原因：

1. **基于内存**：Redis的所有操作都是在内存中完成的，内存的读写速度非常快，这使得Redis具有很高的处理速度。
2. **数据结构简单**：Redis中的数据结构是专门进行设计的，每种数据结构都有一种或多种数据结构来支持。Redis正是依赖这些灵活的数据结构，来提升读取和写入的性能。
3. **单线程**：Redis采用单线程模型，省去了很多上下文切换线程的时间以及CPU消耗，不存在竞争条件，不用去考虑各种锁的问题，不存在加锁释放锁操作，也不会出现死锁而导致的性能消耗。
4. **IO多路复用**：Redis使用基于IO多路复用机制的线程模型，可以处理并发的连接。这使得Redis可以在同一时刻处理多个网络连接请求，从而提高了其处理能力。

总的来说，Redis通过这些设计和实现方式，实现了高效地处理大量并发连接的能力。这使得它在构建高性能网络服务器时非常有用。这些技术可以帮助服务器在处理大量并发连接时保持高效和稳定。





##二、BigKey

### 1）禁用某些命令配置方法

在redis配置文件的【security】配置项中：

例如要禁用keys、flushdb、flushall

![](https://github.com/vankykoo/image/blob/main/073.png?raw=true)



### 2）SCAN命令

引入：

> 在Redis中，虽然`keys *`命令可以用来遍历所有的键，但是在处理大数据时，这并不是一个好的选择。原因有以下几点：
>
> 1. **性能问题**：`keys *`命令会一次性返回所有匹配的键，如果数据库中有大量的键，那么这个命令可能会导致Redis服务器阻塞，从而影响性能。
> 2. **内存问题**：由于`keys *`命令会一次性加载所有的键到内存中，如果键的数量非常大，可能会导致内存不足。
>
> 因此，对于大数据集，我们通常推荐使用`scan`命令进行遍历。`scan`命令是一个基于游标的迭代器，每次被调用后都会返回一个新的游标，用户在下次迭代时需要使用这个新游标作为`scan`命令的参数。这样就可以每次返回少量的元素，避免了一次性加载大量数据带来的问题。



Redis的`scan`命令是一个基于游标的迭代器，每次被调用之后，都会向用户返回一个新的游标。用户在下次迭代时需要使用这个新游标作为`scan`命令的游标参数，以此来延续之前的迭代过程。`scan`返回一个包含两个元素的数组，第一个元素是用于进行下一次迭代的新游标，而第二个元素则是一个数组，这个数组中包含了所有被迭代的元素。如果新游标返回0表示迭代已结束。

`scan`命令的基本语法如下：`SCAN cursor [MATCH pattern] [COUNT count]`

- `cursor` - 游标。
- `pattern` - 匹配的模式。
- `count` - 指定从数据集里返回多少元素，默认值为10。

此外，还有一些与`scan`类似的命令，如：

- `SSCAN`命令用于迭代集合键中的元素。
- `HSCAN`命令用于迭代哈希键中的键值对。
- `ZSCAN`命令用于迭代有序集合中的元素（包括元素成员和元素分值）。

总的来说，Redis提供了`scan`命令，就是用于增量迭代的。这个命令可以每次返回少量的元素，所以这个命令十分适合用来处理大的数据集的迭代，可以用于生产环境。



### 3）bigkey多大才算大

在Redis中，大key并不是指存储在Redis中的某个key的大小超过一定的阈值，而是该key所对应的value过大。一般情况下，我们认为**字符串类型的key的value值超过10KB，就算大key**。对**于set、zset、hash等类型来说，一般数据超过5000条即认为是大key。**

但需要注意的是，在实际业务中，大Key的判定仍然需要根据Redis的实际使用场景、业务场景来进行综合判断。

因此，"多大算big"并没有一个固定的标准，而是需要根据具体情况来判断。



### 4）怎么找出bigkey

在Redis中，我们可以使用以下几种方法来找出大key：

1. **使用Redis自带的命令**：Redis提供了一个`redis-cli --bigkeys`命令，可以对整个数据库中的键值对大小情况进行统计分析。这个命令会输出每种数据类型中最大的bigkey的信息。对于String类型来说，会输出最大bigkey的字节长度，对于集合类型来说，会输出最大bigkey的元素个数。

2. **使用第三方工具**：例如redis-rdb-tools，这个工具在使用过程中会先使用bgsave命令dump一个rdb镜像，然后对这个镜像进行分析。

3. **`MEMORY USAGE ` 命令**：这个命令**可以估计一个key和它的值在RAM中所占用的字节数**。返回的结果是key的值以及为管理该key分配的内存总字节数。对于嵌套数据类型，可以使用选项`SAMPLES`，其中`count`表示抽样的元素个数，默认值为5。当需要抽样所有元素时，使用`SAMPLES 0`。

   基本语法如下：`MEMORY USAGE key [SAMPLES count]`

   例如，在Redis 64位版本V4.0.1和jemalloc做内存分配器的情况下，空字符串可以定义如下：

   ```
   > SET "" ""
   OK
   > MEMORY USAGE ""
   (integer) 51
   ```

   实际数据为空，但是存储时仍然耗费了一些内存，这些内存用于Redis服务器维护内部数据结构。

   随着key和value的增大，内存使用量和key大小基本成线性关系。例如：

   ```
   > SET foo bar
   OK
   > MEMORY USAGE foo
   (integer) 54
   ```

   这个命令在Redis 4.0版本引入，可以帮助我们更深入地了解Redis内部的内存使用情况。

需要注意的是，这些命令可能会阻塞Redis，因此在执行这些命令时需要谨慎操作。在生产环境中，最好在从节点上执行这些命令，以减少对主节点的影响。



### 5）怎么删除bigkey

在Redis中，我们可以使用以下几种方法来删除大key：

1. **使用Redis自带的命令**：Redis提供了一个`redis-cli --bigkeys`命令，可以对整个数据库中的键值对大小情况进行统计分析。这个命令会输出每种数据类型中最大的bigkey的信息。对于String类型来说，会输出最大bigkey的字节长度，对于集合类型来说，会输出最大bigkey的元素个数。
2. **使用第三方工具**：例如redis-rdb-tools，这个工具在使用过程中会先使用bgsave命令dump一个rdb镜像，然后对这个镜像进行分析。
3. **渐进式删除**：这种方法是通过scan命令遍历大key，每次取得少部分元素，对其删除，然后再获取和删除下一批元素。例如，删除大Hashes时，可以通过hscan命令，每次获取500个字段，再用hdel命令，每次删除1个字段。
4. **使用UNLINK命令**：从Redis 4.0版本开始，Redis支持了UNLINK命令。UNLINK命令在所有命名空间中把key删掉，立即返回，不阻塞。后台线程执行真正的释放空间的操作。

需要注意的是，这些命令可能会阻塞Redis，因此在执行这些命令时需要谨慎操作。在生产环境中，最好在从节点上执行这些命令，以减少对主节点的影响。



### 6）BigKey生成调优

Redis的lazy-freeing是一种**延迟释放**或**惰性删除**的机制，当删除键的时候，Redis提供**异步**延时释放key内存的功能，把key释放操作放在后台单独的子线程处理中，从而减少删除大key对Redis主线程的阻塞。

这种机制主要用于解决以下两类问题：

1. 主动删除：与DEL命令对应的删除操作。例如，Redis 4.0引入了<u>UNLINK命令</u>，该命令在逻辑上删除键，然后将实际的内存释放操作交给后台线程。
2. 被动删除：包括过期key的删除和达到maxmemory时的key驱逐淘汰。例如，当Redis内存使用达到maxmemory，并设置有淘汰策略时，在被动淘汰键时，是否采用lazy free机制。

此外，Redis还为FLUSHALL/FLUSHDB命令添加了ASYNC选项，使得在清理整个实例或数据库时，操作都是异步的。

需要注意的是，虽然lazy-freeing可以有效地避免删除大key带来的性能和可用性问题，但如果内存变动不大，可能会导致内存释放不及时，导致Redis内存超过maxmemory的限制。因此，在使用lazy-freeing时需要根据具体情况进行权衡。

在Redis配置文件中可以这样配置：

![](https://github.com/vankykoo/image/blob/main/074.png?raw=true)



#### bigKey如何调优

在Redis中，大key是指某个key的value过大，这可能会导致Redis的性能下降或者崩溃。对于大key的优化，主要有以下几个方面：

1. **找出大key**：我们可以使用Redis自带的`redis-cli --bigkeys`命令或者第三方工具如redis-rdb-tools来找出大key。这些工具可以帮助我们找出数据库中的大key，从而进行针对性的优化。
2. **优化大key**：对于大key的优化，主要是通过拆分和控制key的生命周期来实现。例如，我们可以将一个大的hash或list拆分成多个小的hash或list，每个hash或list存储部分数据。此外，我们还可以通过设置过期时间来控制key的生命周期。
3. **删除大key**：对于需要删除的大key，我们可以使用渐进式删除的方法。例如，我们可以使用`hscan`命令配合`hdel`命令来删除大hash，或者使用`sscan`命令配合`srem`命令来删除大set。此外，从Redis 4.0版本开始，Redis支持了UNLINK命令，该命令在逻辑上删除键，然后将实际的内存释放操作交给后台线程。
4. **使用lazyfree机制**：从Redis 4.0版本开始，Redis引入了lazyfree机制。这种机制允许Redis在后台线程中异步地释放内存。这样，在删除大key时，可以避免阻塞主线程。

总的来说，在处理Redis中的大key问题时，我们需要结合实际情况，通过找出大key、优化大key、删除大key以及使用lazyfree等方法来进行调优。



## 三、缓存双写一致性

###1）介绍

Redis和MySQL缓存双写一致性是指**在使用Redis作为MySQL的缓存时，如何保证两者数据的一致性。**在这种场景下，当数据发生变化时，我们需要同时更新MySQL数据库和Redis缓存。这就涉及到一个问题，即我们应该先更新数据库还是先更新缓存，以及如何处理更新操作失败的情况。

一般来说，有三种常见的策略：

1. **先更新数据库，再更新缓存**：这种策略的问题是如果更新缓存失败，那么缓存中的数据就会是旧的，与数据库中的数据不一致。
2. **先删除缓存，再更新数据库**：这种策略的问题是如果在删除缓存后、更新数据库前，有其他请求查询到了旧数据并将其写回缓存，那么即使数据库更新成功，缓存中的数据也仍然是旧的。
3. **先更新数据库，再删除缓存**：这种策略可以避免上述两种策略的问题。但如果删除缓存失败，则需要有重试机制来确保缓存最终会被删除。

以上三种策略都有可能导致数据不一致的情况，因此在实际应用中，我们需要根据具体情况选择合适的策略，并可能需要结合其他技术（如消息队列）来进一步确保数据的一致性。



### 2）双检加锁策略

问题：redis中没有数据，但是MySQL中有。短时间内大量用户查询redis缓存，由于redis中没有数据，导致大量操作涌入MySQL，然后又大量写回redis，很容易导致**缓存击穿**。

解决：双检加锁策略

> 在Redis中**，双检加锁策略**通常用于实现分布式锁，以解决并发和同步问题。这种策略的基本思路是，当一个客户端尝试获取锁时，它首先会检查锁是否存在，如果不存在，则尝试设置锁；如果存在，则等待一段时间后再次检查。这就是所谓的"双检"：第一次检查是在尝试获取锁之前，第二次检查是在等待一段时间后。
>
> 在Redis中，我们可以使用`SETNX`命令来实现加锁操作。这个命令可以保证在某个key不存在时进行设置，但是当多个线程同时执行`SETNX`命令时，可能会出现多个线程都设置成功的情况，导致死锁。为了解决这个问题，我们可以使用Lua脚本来实现原子性的加锁和解锁操作。
>
> 例如，我们可以定义一个Lua脚本，该脚本首先使用`SETNX`命令尝试设置锁，如果设置成功，则返回1；如果设置失败，则检查当前的锁是否已经过期（通过比较锁的值和当前时间），如果已经过期，则使用`GETSET`命令更新锁，并返回旧的值。然后，在客户端中，我们可以根据Lua脚本的返回值来判断是否成功获取到了锁。
>
> 总的来说，在Redis中实现双检加锁策略需要综合考虑多种因素，并可能需要结合其他技术（如Lua脚本）来确保操作的原子性和数据的一致性。

```java
public User findUserById2(Integer id){
  User user = null;
  String key = CACHE_KEY_USER+id;

  //1 先从redis里面查询，如果有直接返回结果，如果没有再去查询mysql，
  // 第1次查询redis，加锁前
  user = (User) redisTemplate.opsForValue().get(key);
  if(user == null) {
    //2 大厂用，对于高QPS的优化，进来就先加锁，保证一个请求操作，让外面的redis等待一下，避免击穿mysql
    synchronized (UserService.class){
      //第2次查询redis，加锁后
      user = (User) redisTemplate.opsForValue().get(key);
      //3 二次查redis还是null，可以去查mysql了(mysql默认有数据)
      if (user == null) {
        //4 查询mysql拿数据(mysql默认有数据)
        user = userMapper.selectByPrimaryKey(id);
        if (user == null) {
          return null;
        }else{
          //5 mysql里面有数据的，需要回写redis，完成数据一致性的同步工作
          redisTemplate.opsForValue().setIfAbsent(key,user,7L,TimeUnit.DAYS);
        }
      }
    }
  }
  return user;
}
```



###3）缓存一致性的策略

**给缓存设置过期时间，定期清理缓存并回写，是保证最终一致性的解决方案。**

我们可以对存入缓存的数据设置过期时间，所有的**写操作以数据库为准**，对缓存操作只是尽最大努力即可。也就是说如果数据库写成功，缓存更新失败，那么只要到达过期时间，则后面的读请求自然会从数据库中读取新值然后回填缓存，达到一致性，切记，**要以mysql的数据库写入库为准**。



一般来说，有三种常见的策略：

1. **先更新数据库，再更新缓存**：这种策略的问题是如果更新缓存失败，那么缓存中的数据就会是旧的，与数据库中的数据不一致。

2. **先删除缓存，再更新数据库**：这种策略的问题是如果在删除缓存后、更新数据库前，有其他请求查询到了旧数据并将其写回缓存，那么即使数据库更新成功，缓存中的数据也仍然是旧的。

   > 解决方法：延伸双删策略
   >
   > 这是一种常用的策略，用于保持存储和缓存数据的最终一致性。具体来说，当我们需要更新数据库和缓存中的数据时，我们可以先删除缓存中的数据，然后更新数据库中的数据，最后再次删除缓存中的数据。这样做的目的是为了避免在更新数据库和缓存之间的时间窗口内，其他线程可能从缓存中读取到旧的数据。
   >
   > 然而，这种策略并不是完美的，它可能会出现以下问题：
   >
   > 1. **延时时间难以确定**：延时时间需要大于读写缓存的时间，但这个时间很难准确估计，如果设置得过短，可能会导致数据不一致；如果设置得过长，则会增加数据更新的延迟。
   >
   >    > 解决：
   >    >
   >    > **第一种方法：**
   >    >
   >    > 在业务程序运行的时候，统计下线程读数据和写缓存的操作时间，自行评估自己的项目的读数据业务逻辑的耗时，以此为基础来进行估算。然后写数据的休眠时间则在读数据业务逻辑的耗时基础上加百毫秒即可。
   >    >
   >    > 这么做的目的，就是确保读请求结束，写请求可以删除读请求造成的缓存脏数据。
   >    >
   >    >  **第二种方法：**
   >    >
   >    > 新启动一个后台监控程序，比如后面要讲解的WatchDog监控程序，会加时
   >
   > 2. **并发问题**：在高并发环境下，即使使用了延时双删策略，也可能会出现数据不一致的情况。例如，在删除缓存和更新数据库之间，如果有其他线程查询到了旧数据并将其写回缓存，那么即使数据库更新成功，缓存中的数据也仍然是旧的。
   >
   > 3. **Redis和数据库同步时间**：因为Redis和数据库主从节点数据同步不是实时的，所以需要等待一段时间去增强它们的数据一致性。但这个同步时间也是无法确定的。
   >
   > 4. **吞吐量降低：**因为在更新数据时，我们需要先删除缓存，然后更新数据库，最后再次删除缓存。这个过程中涉及到两次缓存删除操作和一次数据库更新操作，相比于单纯的数据库更新操作，会增加额外的开销。特别是当延时时间设置较长时，可能会导致数据更新的延迟增加，从而降低系统的吞吐量。然而，需要注意的是，虽然延时双删策略可能会降低吞吐量，但它可以有效地保证存储和缓存数据的最终一致性。因此，在实际应用中，我们需要根据具体情况进行权衡，选择合适的策略来平衡数据一致性和系统性能。
   >
   >    > **系统吞吐量**是指系统在单位时间内处理请求的数量，是系统的抗压、负载能力，代表一个系统每秒钟能承受的最大用户访问量。系统吞吐量要素包括request对cpu的消耗，外部接口，IO等等紧密关联。一个系统的吞吐量通常由qps（tps）、并发数来决定，每个系统对这两个值都有一个相对极限值，只要某一项达到最大值，系统的吞吐量就上不去了。
   >
   > 因此，在使用延时双删策略时，我们需要根据具体情况进行权衡，并可能需要结合其他技术（如分布式锁）来进一步确保数据的一致性。

3. **先更新数据库，再删除缓存**：这种策略可以避免上述两种策略的问题。但如果删除缓存失败，则需要有重试机制来确保缓存最终会被删除。



**第三条是最优策略：**

**但还是存在问题**：假如缓存删除失败或者来不及，导致请求再次访问redis时缓存命中，读取到的是缓存旧值。

**解决：**

![](https://github.com/vankykoo/image/blob/main/075.png?raw=true)

1 可以把要删除的缓存值或者是要更新的数据库值**暂存到消息队列**中（例如使用Kafka/RabbitMQ等）。

2 当程序没有能够成功地删除缓存值或者是更新数据库值时，可以从消息队列中重新读取这些值，然后再次进行删除或更新。

3 如果能够成功地删除或更新，我们就要把这些值从消息队列中去除，以免重复操作，此时，我们也可以保证数据库和缓存的数据一致了，否则还需要再次进行重试

4 如果重试超过的一定次数后还是没有成功，我们就需要向业务层发送报错信息了，通知运维人员。



> 如果业务层要求必须读取一致性的数据，那么我们就需要在更新数据库时，先在**Redis缓存客户端暂停并发读请求**，等数据库更新完、缓存值删除后，再读取数据，从而保证数据一致性，这是理论可以达到的效果，但实际，**不推荐**，因为真实生产环境中，分布式下很难做到实时一致性，一般都是最终一致性，请大家参考。



![](https://github.com/vankykoo/image/blob/main/076.png?raw=true)



### 4）MySQL主从复制

![](https://github.com/vankykoo/image/blob/main/077.png?raw=true)

MySQL的主从复制将经过如下步骤：

1、当 master 主服务器上的数据发生改变时，则将其改变写入二进制事件日志文件中；

2、salve 从服务器会在一定时间间隔内对 master 主服务器上的二进制日志进行探测，探测其是否发生过改变，

如果探测到 master 主服务器的二进制事件日志发生了改变，则开始一个 I/O Thread 请求 master 二进制事件日志；

3、同时 master 主服务器为每个 I/O Thread 启动一个dump  Thread，用于向其发送二进制事件日志；

4、slave 从服务器将接收到的二进制事件日志保存至自己本地的中继日志文件中；

5、salve 从服务器将启动 SQL Thread 从中继日志中读取二进制日志，在本地重放，使得其数据和主服务器保持一致；

6、最后 I/O Thread 和 SQL Thread 将进入睡眠状态，等待下一次被唤醒；



### 5）canal工作原理

> 是的，我了解阿里的canal。Canal是阿里巴巴旗下的一款开源项目，主要用途是基于MySQL数据库增量日志解析，提供增量数据订阅和消费。它模拟MySQL slave的交互协议，伪装自己为MySQL slave，向MySQL master发送dump协议。然后MySQL master开始推送binary log给slave (即 canal)，canal解析binary log对象。
>
> Canal主要用于以下几种场景：
>
> - 数据库镜像
> - 数据库实时备份
> - 索引构建和实时维护
> - 业务cache刷新
> - 带业务逻辑的增量数据处理
>
> Canal支持MySQL 5.1.x, 5.5.x, 5.6.x, 5.7.x, 8.0.x等版本。在使用过程中，需要注意的是，由于Canal模拟MySQL slave的交互协议，所以在使用Canal时需要确保MySQL的binlog功能已经开启，并且binlog格式需要设置为ROW。
>
> 总的来说，Canal是一个非常强大的工具，可以帮助我们更好地处理数据库的增量数据订阅和消费。

![](https://github.com/vankykoo/image/blob/main/078.png?raw=true)

![](C:\Users\86180\Desktop\picPick\148.png)





## 四、canal使用案例

### 1）MySQL准备

#### 1.开启bin_log写入功能

* MySQL5.7

  在MySQL配置文件my.ini里的mysqld配置项下加入

  ```ini
  log-bin=mysql-bin #开启 binlog
  binlog-format=ROW #选择 ROW 模式
  server_id=1    #配置MySQL replaction需要定义，不要和canal的 slaveId重复
  ```

  **ROW模式** 除了记录sql语句之外，还会记录每个字段的变化情况，能够清楚的记录每行数据的变化历史，但会占用较多的空间。

  **STATEMENT模式 **只记录了sql语句，但是没有记录上下文信息，在进行数据恢复的时候可能会导致数据的丢失情况；

  **MIX模式** 比较灵活的记录，理论上说当遇到了表结构变更的时候，就会记录为statement模式。当遇到了数据更新或者删除情况下就会变为row模式；

* MySQL8好像默认开启了。

  可以用以下命令查看是否开启

  ```mysql
  SHOW VARIABLES LIKE 'log_bin';
  ```



### 2）创建canal用户

* MySQL5.7

  ```mysql
  DROP USER IF EXISTS 'canal'@'%';
  CREATE USER 'canal'@'%' IDENTIFIED BY 'canal';   #创建用户
  GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' IDENTIFIED BY 'canal';  #赋予所有权限 
  FLUSH PRIVILEGES;
   
  SELECT * FROM mysql.user;
  ```

* MySQL8

  ```mysql
  DROP USER IF EXISTS 'canal'@'%';
  CREATE USER 'canal'@'%' IDENTIFIED BY 'canal';  #创建用户
  GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' WITH GRANT OPTION; #赋予所有权限  
  FLUSH PRIVILEGES;
   
  SELECT * FROM mysql.user;
  ```

  ​

### 2）canal准备

①**在linux中安装canal** ，下载地址：https://github.com/alibaba/canal/releases

②在/mycanal路径下安装

③修改/mycanal/conf/example路径下的instance.properties文件

**改成自己电脑的ip和MySQL端口号**

![](https://github.com/vankykoo/image/blob/main/079.png?raw=true)

这里改成自己MySQL用户的账号密码，默认都是canal

![](https://github.com/vankykoo/image/blob/main/080.png?raw=true)



④在/mycanal/bin下输入`./startup.sh`命令启动canal

⑤**确认是否启动成功：**可以在/mycal/logs/canal下查看canal.log；在/mycal/logs/exanple下查看example.log





### 3）java程序

①引入pom依赖

```xml
<dependency>
    <groupId>com.alibaba.otter</groupId>
    <artifactId>canal.client</artifactId>
    <version>1.1.0</version>
</dependency>
```



②写application.properties

```properties
server.port=5555

# ========================alibaba.druid=====================
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
# 记得修改数据库下面是db01
spring.datasource.url=jdbc:mysql://localhost:3306/db01?useUnicode=true&characterEncoding=utf-8&useSSL=false
# 数据库账号密码
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.druid.test-while-idle=false
```



③编写RedisUtils

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

public class RedisUtils
{
    public static final String  REDIS_IP_ADDR = "192.168.xxx.xxx";//Redis所在ip地址
    public static final String  REDIS_pwd = "111111";//redis密码
    public static JedisPool jedisPool;

    static {
        JedisPoolConfig jedisPoolConfig=new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(20);
        jedisPoolConfig.setMaxIdle(10);
      //6379为redis端口，可以修改
        jedisPool=new JedisPool(jedisPoolConfig,REDIS_IP_ADDR,6379,10000,REDIS_pwd);
    }

    public static Jedis getJedis() throws Exception {
        if(null!=jedisPool){
            return jedisPool.getResource();
        }
        throw new Exception("Jedispool is not ok");
    }

}
```



④编写RedisCanalClientExample

```java
import com.alibaba.fastjson.JSONObject;
import com.alibaba.otter.canal.client.CanalConnector;
import com.alibaba.otter.canal.client.CanalConnectors;
import com.alibaba.otter.canal.protocol.CanalEntry.*;
import com.alibaba.otter.canal.protocol.Message;
import com.atguigu.canal.util.RedisUtils;
import redis.clients.jedis.Jedis;
import java.net.InetSocketAddress;
import java.util.List;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

public class RedisCanalClientExample
{
    public static final Integer _60SECONDS = 60;
    public static final String  REDIS_IP_ADDR = "192.168.xxx.xxx";	//redis所在ip

    private static void redisInsert(List<Column> columns)
    {
        JSONObject jsonObject = new JSONObject();
        for (Column column : columns)
        {
            System.out.println(column.getName() + " : " + column.getValue() + "    update=" + column.getUpdated());
            jsonObject.put(column.getName(),column.getValue());
        }
        if(columns.size() > 0)
        {
            try(Jedis jedis = RedisUtils.getJedis())
            {
                jedis.set(columns.get(0).getValue(),jsonObject.toJSONString());
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }


    private static void redisDelete(List<Column> columns)
    {
        JSONObject jsonObject = new JSONObject();
        for (Column column : columns)
        {
            jsonObject.put(column.getName(),column.getValue());
        }
        if(columns.size() > 0)
        {
            try(Jedis jedis = RedisUtils.getJedis())
            {
                jedis.del(columns.get(0).getValue());
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }

    private static void redisUpdate(List<Column> columns)
    {
        JSONObject jsonObject = new JSONObject();
        for (Column column : columns)
        {
            System.out.println(column.getName() + " : " + column.getValue() + "    update=" + column.getUpdated());
            jsonObject.put(column.getName(),column.getValue());
        }
        if(columns.size() > 0)
        {
            try(Jedis jedis = RedisUtils.getJedis())
            {
                jedis.set(columns.get(0).getValue(),jsonObject.toJSONString());
                System.out.println("---------update after: "+jedis.get(columns.get(0).getValue()));
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }

    public static void printEntry(List<Entry> entrys) {
        for (Entry entry : entrys) {
            if (entry.getEntryType() == EntryType.TRANSACTIONBEGIN || entry.getEntryType() == EntryType.TRANSACTIONEND) {
                continue;
            }

            RowChange rowChage = null;
            try {
                //获取变更的row数据
                rowChage = RowChange.parseFrom(entry.getStoreValue());
            } catch (Exception e) {
                throw new RuntimeException("ERROR ## parser of eromanga-event has an error,data:" + entry.toString(),e);
            }
            //获取变动类型
            EventType eventType = rowChage.getEventType();
            System.out.println(String.format("================&gt; binlog[%s:%s] , name[%s,%s] , eventType : %s",
                    entry.getHeader().getLogfileName(), entry.getHeader().getLogfileOffset(),
                    entry.getHeader().getSchemaName(), entry.getHeader().getTableName(), eventType));

            for (RowData rowData : rowChage.getRowDatasList()) {
                if (eventType == EventType.INSERT) {
                    redisInsert(rowData.getAfterColumnsList());
                } else if (eventType == EventType.DELETE) {
                    redisDelete(rowData.getBeforeColumnsList());
                } else {//EventType.UPDATE
                    redisUpdate(rowData.getAfterColumnsList());
                }
            }
        }
    }


    public static void main(String[] args)
    {
        System.out.println("---------O(∩_∩)O哈哈~ initCanal() main方法-----------");

        //=================================
        // 创建链接canal服务端
        CanalConnector connector = CanalConnectors.newSingleConnector(new InetSocketAddress(REDIS_IP_ADDR,
                11111), "example", "", "");
        int batchSize = 1000;
        //空闲空转计数器
        int emptyCount = 0;
        System.out.println("---------------------canal init OK，开始监听mysql变化------");
        try {
            connector.connect();
            //connector.subscribe(".*\\..*");
          //监听数据库的哪个表
            connector.subscribe("db01.t_user");
            connector.rollback();
            int totalEmptyCount = 10 * _60SECONDS;
            while (emptyCount < totalEmptyCount) {
                System.out.println("我是canal，每秒一次正在监听:"+ UUID.randomUUID().toString());
                Message message = connector.getWithoutAck(batchSize); // 获取指定数量的数据
                long batchId = message.getId();
                int size = message.getEntries().size();
                if (batchId == -1 || size == 0) {
                    emptyCount++;
                    try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
                } else {
                    //计数器重新置零
                    emptyCount = 0;
                    printEntry(message.getEntries());
                }
                connector.ack(batchId); // 提交确认
                // connector.rollback(batchId); // 处理失败, 回滚数据
            }
            System.out.println("已经监听了"+totalEmptyCount+"秒，无任何消息，请重启重试......");
        } finally {
            connector.disconnect();
        }
    }
}
```



![](https://github.com/vankykoo/image/blob/main/081.png?raw=true)





## 五、HyperLogLog

### 1）名词介绍

这些都是用来衡量网站或应用性能的常见指标：

- **PV** (Page View)：访问量，即页面浏览量或点击量，衡量网站用户访问的网页数量。在一定统计周期内用户每打开或刷新一个页面就记录1次，多次打开或刷新同一页面则浏览量累计。
- **UV** (Unique Visitor)：独立访客，统计1天内访问某站点的用户数。可以理解成访问某网站的电脑的数量。页面访问人数，同一个账号访问同一个页面两次，UV算1次。
- **DAU** (Daily Active User)：日活跃用户数量。常用于反映网站、互联网应用或网络游戏的运营情况。DAU通常统计一日（统计日）之内，登录或使用了某个产品的用户数（去除重复登录的用户），这与流量统计工具里的访客（UV）概念相似。
- **MAU** (Monthly Active Users)：月活跃用户人数。是在线游戏的一个用户数量统计名词，数量越大意味着玩这款游戏的人越多。



### 2）介绍、需求

* 介绍：

**基数**：数据集，去重复后的真实个数。

**hyperloglog用于统计一个集合中不重复的元素个数。**



* 需求：

很多计数类场景，比如 每日注册 IP 数、每日访问 IP 数、页面实时访问数 PV、访问用户数 UV等。

因为主要的目标高效、巨量地进行计数，所以对存储的数据的内容并不太关心。

也就是说它只能用于统计巨量数量，不太涉及具体的统计对象的内容和精准性。

统计单日一个页面的访问量(PV)，单次访问就算一次。

统计单日一个页面的用户访问量(UV)，即按照用户为维度计算，单个用户一天内多次访问也只算一次。

多个key的合并统计，某个门户网站的所有模块的PV聚合统计就是整个网站的总PV。



### 3）原理及说明

只是进行不重复的基数统计，不是集合也不保存数据，只记录数量而不是具体内容。

有误差，牺牲准确率来换取空间，误差仅仅只是0.82%左右。

> **hyperloglog统计基数原理**
>
> HyperLogLog是一种用于解决基数统计问题的算法，它可以估算一个多重集合中不同元素的数量。基数统计问题的精确计算需要与基数成比例的内存，这对于非常大的数据集来说是不切实际的。概率性基数估计器，如HyperLogLog算法，使用的内存明显少于此，但只能近似基数。
>
> HyperLogLog算法能够估算大于10^9的基数，使用1.5kB的内存，典型的准确率（标准误差）为2%。HyperLogLog是早期LogLog算法的扩展。
>
> HyperLogLog算法的基础是观察到，可以通过计算集合中每个数字二进制表示中前导零的最大数量来估算均匀分布随机数的多重集合的基数。如果观察到的前导零的最大数量为n，则集合中不同元素的数量的估计值为2^n。
>
> 在HyperLogLog算法中，将哈希函数应用于原始多重集合中的每个元素，以获得与原始多重集合具有相同基数的均匀分布随机数集合。然后可以使用上述算法估算这个随机分布集合的基数。
>
> 使用上述算法得到的基数简单估计具有较大的方差。在HyperLogLog算法中，通过将多重集合分割成许多子集，计算每个子集中数字前导零的最大数量，并使用调和平均数将每个子集的这些估计值组合成整个集合的基数估计值，从而最小化方差。
>
> HyperLogLog有三个主要操作：添加新元素到集合（add）、获取集合的基数（count）和获取两个集合的并集（merge）。



> **hyperloglog每个键只需要花费12kb**
>
> HyperLogLog是一种用于解决基数统计问题的算法，它可以估算一个多重集合中不同元素的数量。在Redis中，每个HyperLogLog键只需要花费12KB内存，就可以计算接近2^64个不同元素的基数。
>
> HyperLogLog算法的基础是观察到，可以通过计算集合中每个数字二进制表示中前导零的最大数量来估算均匀分布随机数的多重集合的基数。如果观察到的前导零的最大数量为n，则集合中不同元素的数量的估计值为2^n。
>
> 在HyperLogLog算法中，将哈希函数应用于原始多重集合中的每个元素，以获得与原始多重集合具有相同基数的均匀分布随机数集合。然后可以使用上述算法估算这个随机分布集合的基数。
>
> 使用上述算法得到的基数简单估计具有较大的方差。在HyperLogLog算法中，通过将多重集合分割成许多子集，计算每个子集中数字前导零的最大数量，并使用调和平均数将每个子集的这些估计值组合成整个集合的基数估计值，从而最小化方差。
>
> 因此，尽管HyperLogLog只提供了不精确的去重计数方案（标准误差是0.81%），但其优点在于，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。这就是为什么每个HyperLogLog键在Redis中只需要花费12KB内存。





### 4）使用案例

Service

```java
@Service
@Slf4j
public class HyperLogLogService {

    @Autowired
    private RedisTemplate redisTemplate;

  //模拟访问，使用随机生成ip
    @PostConstruct
    public void initIp(){
        new Thread(() -> {
            String ip = null;
            for(int i = 0; i < 200; i++){
                Random random = new Random();
                ip = random.nextInt(256) + "." +
                        random.nextInt(256) + "." +
                        random.nextInt(256) + "." +
                        random.nextInt(256);
              //加到hyperloglog
                Long hll = redisTemplate.opsForHyperLogLog().add("hll", ip);
                log.info("ip={},该ip访问首页的次数={}",ip,hll);

                try {
                    TimeUnit.SECONDS.sleep(3);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }

        },"t1").start();
    }

    public long uv(){
        return redisTemplate.opsForHyperLogLog().size("hll");
    }

}
```



Controller

```java
@Api(tags = "淘宝亿级UV访问的redis统计方案")
@RestController
@Slf4j
public class HyperLogLogController {
    @Autowired
    private HyperLogLogService hyperLogLogService;

    @GetMapping("/uv")
    @ApiOperation("获取去重后的UV统计量")
    public long uv(){
        return hyperLogLogService.uv();
    }
  
}
```





> `@PostConstruct`是Java EE 5引入的注解，它在依赖注入完成后立即执行，且只会被调用一次，而且在构造函数之后、初始化方法之前运行。这个注解通常被用来进行一些初始化工作。
>
> 例如，在Spring框架中，你可能会在一个使用了`@Component`注解的类中使用`@PostConstruct`注解，如下所示：
>
> ```java 
> @Componentpublic class MyClass {    
>   @PostConstruct    
>   public void init() {        
>     // 这里可以进行一些初始化工作    
>   }
> }
> ```
>
> 在这个例子中，`init`方法会在`MyClass`的实例被Spring创建并完成依赖注入后调用。
>
> 需要注意的是，如果你的类有多个方法使用了`@PostConstruct`注解，那么只有一个方法会被调用，并且具体调用哪个方法是不确定的。因此，一个类中通常只有一个方法使用`@PostConstruct`注解。











## 六、GEO

### 1）介绍

Redis GEO主要用于存储地理位置信息，并对存储的信息进行操作，该功能在Redis 3.2版本新增。Redis GEO操作方法有：

- **geoadd**：添加地理位置的坐标。
- **geopos**：获取地理位置的坐标。
- **geodist**：计算两个位置之间的距离。
- **georadius**：根据用户给定的经纬度坐标来获取指定范围内的地理位置集合。
- **georadiusbymember**：根据储存在位置集合里面的某个地点获取指定范围内的地理位置集合。
- **geohash**：返回一个或多个位置对象的 geohash 值。

例如，我们可以使用`geoadd`命令将一个或多个经度、纬度、位置名称添加到指定的key中。然后，我们可以使用`geopos`命令从给定的key里返回所有指定名称的位置（经度和纬度）。此外，我们还可以使用`geodist`命令返回两个给定位置之间的距离。

此外，我们还可以使用`georadius`和`georadiusbymember`命令来找出位于指定范围内的元素。这两个命令都可以找出位于指定范围内的元素，但是`georadiusbymember`的中心点是由给定的位置元素决定的，而不是使用经度和纬度来决定中心点。

这些功能在处理地理位置相关的问题时非常有用，例如在实现“附近的人”或“附近的商铺”等功能时。但需要注意的是，Redis GEO适合精度不是很高的场景，如果需要更高精度的处理，可能需要使用其他工具或方法。



### 2）使用

Service

```java
@Service
@Slf4j
public class GeoService
{
    public static final String CITY ="city";

    @Autowired
    private RedisTemplate redisTemplate;

    public String geoAdd()
    {
        Map<String, Point> map= new HashMap<>();
        map.put("天安门",new Point(116.403963,39.915119));
        map.put("故宫",new Point(116.403414 ,39.924091));
        map.put("长城" ,new Point(116.024067,40.362639));

        redisTemplate.opsForGeo().add(CITY,map);

        return map.toString();
    }

    public Point position(String member) {
        //获取经纬度坐标
        List<Point> list= this.redisTemplate.opsForGeo().position(CITY,member);
        return list.get(0);
    }


    public String hash(String member) {
        //geohash算法生成的base32编码值
        List<String> list= this.redisTemplate.opsForGeo().hash(CITY,member);
        return list.get(0);
    }


    public Distance distance(String member1, String member2) {
        //获取两个给定位置之间的距离
        Distance distance= this.redisTemplate.opsForGeo().distance(CITY,member1,member2, RedisGeoCommands.DistanceUnit.KILOMETERS);
        return distance;
    }

    public GeoResults radiusByxy() {
        //通过经度，纬度查找附近的,北京王府井位置116.418017,39.914402
        Circle circle = new Circle(116.418017, 39.914402, Metrics.KILOMETERS.getMultiplier());
        //返回50条
        RedisGeoCommands.GeoRadiusCommandArgs args = RedisGeoCommands.GeoRadiusCommandArgs.newGeoRadiusArgs().includeDistance().includeCoordinates().sortAscending().limit(50);
        GeoResults<RedisGeoCommands.GeoLocation<String>> geoResults= this.redisTemplate.opsForGeo().radius(CITY,circle, args);
        return geoResults;
    }

    public GeoResults radiusByMember() {
        //通过地方查找附近
        String member="天安门";
        //返回50条
        RedisGeoCommands.GeoRadiusCommandArgs args = RedisGeoCommands.GeoRadiusCommandArgs.newGeoRadiusArgs().includeDistance().includeCoordinates().sortAscending().limit(50);
        //半径10公里内
        Distance distance=new Distance(10, Metrics.KILOMETERS);
        GeoResults<RedisGeoCommands.GeoLocation<String>> geoResults= this.redisTemplate.opsForGeo().radius(CITY,member, distance,args);
        return geoResults;
    }
}
```

> **radiusByxy()**这段代码是在使用Spring Data Redis的Geo模块来进行地理位置相关的操作。具体来说，这段代码定义了一个名为`radiusByxy`的公共方法，该方法执行以下操作：
>
> 1. 创建一个`Circle`对象，表示一个以经度为113.382656，纬度为23.038821，半径为1公里（`Metrics.KILOMETERS.getMultiplier()`）的圆形区域。
> 2. 定义一个`GeoRadiusCommandArgs`对象，设置了一些参数，包括包含距离、包含坐标、降序排序以及限制结果数量为50。
> 3. 使用`redisTemplate.opsForGeo().radius(CITY,circle,args)`方法查询在上述圆形区域内的地理位置信息，并将结果存储在`GeoResults<RedisGeoCommands.GeoLocation<String>>`对象中。
> 4. 返回查询得到的地理位置信息。
>
> 这段代码的主要作用是查询在指定圆形区域内的地理位置信息，并返回查询结果。这在一些需要进行地理位置查询的场景中非常有用，比如查找附近的餐馆、商店等。



Controller

```java
@Api(tags = "美团地图位置附近的酒店推送GEO")
@RestController
@Slf4j
public class GeoController
{
    @Resource
    private GeoService geoService;

    @ApiOperation("添加坐标geoadd")
    @RequestMapping(value = "/geoadd",method = RequestMethod.GET)
    public String geoAdd()
    {
        return geoService.geoAdd();
    }

    @ApiOperation("获取经纬度坐标geopos")
    @RequestMapping(value = "/geopos",method = RequestMethod.GET)
    public Point position(String member)
    {
        return geoService.position(member);
    }

    @ApiOperation("获取经纬度生成的base32编码值geohash")
    @RequestMapping(value = "/geohash",method = RequestMethod.GET)
    public String hash(String member)
    {
        return geoService.hash(member);
    }

    @ApiOperation("获取两个给定位置之间的距离")
    @RequestMapping(value = "/geodist",method = RequestMethod.GET)
    public Distance distance(String member1, String member2)
    {
        return geoService.distance(member1,member2);
    }

    @ApiOperation("通过经度纬度查找北京王府井附近的")
    @RequestMapping(value = "/georadius",method = RequestMethod.GET)
    public GeoResults radiusByxy()
    {
        return geoService.radiusByxy();
    }

    @ApiOperation("通过地方查找附近,本例写死天安门作为地址")
    @RequestMapping(value = "/georadiusByMember",method = RequestMethod.GET)
    public GeoResults radiusByMember()
    {
        return geoService.radiusByMember();
    }

}
```





## 七、bitmap和布隆过滤器

### 1）bitmap介绍

Bitmap是Redis的一种扩展数据类型，主要用于二值状态统计。它的底层使用的是String的数据结构，而String保存在计算机中的格式是二进制的字节数组，这样Bitmap就充分利用了每个字节的bit位，大大节省了内存开销。

Redis Bitmap提供了以下几个常用命令：

- **SETBIT**：为位数组指定偏移量上的二进制位设置值，偏移量从0开始计数，二进制位的值只能为0或1。
- **GETBIT**：获取指定偏移量上二进制位的值。
- **BITCOUNT**：统计位数组中值为1的二进制位数量。

例如，在记录用户登录行为或签到情况时，我们可以使用Bitmap。我们可以将日期作为key，然后用户id为offset，如果当日活跃过就设置为1。这样就可以方便地统计活跃用户或者用户签到情况。

需要注意的是，由于Bitmap在Redis中并不是一个新的数据类型，其底层是基于Redis的字符串类型实现的。因此，在设置非零初值时，我们不需要挨个设置比特位，只需要给定一个字符串就可以了。

但是，由于Redis中字符串的最大长度是512M，所以Bitmap的offset值也是有上限的，其最大值是：8 * 1024 * 1024 * 512 = 2^32。因此，在使用Bitmap时需要注意这个限制。



### 2）布隆过滤器介绍

布隆过滤器是一种数据结构，非常高效且节省空间，主要用于检查一个元素是否在一个集合中。它的优点是空间效率和查询时间都远超一般的算法，缺点是存在一定的误识别率和删除困难。

布隆过滤器实际上是一个很长的二进制向量和一系列随机映射函数。我们可以通过添加元素到布隆过滤器中，并可以检查一个元素是否可能在布隆过滤器中。**如果布隆过滤器说某个元素可能存在，那么这个元素可能不存在；但是如果布隆过滤器说某个元素不存在，那么这个元素一定不存在。**

在Redis中，布隆过滤器的实现是基于位数组和哈希函数的。我们可以使用`bf.add`命令将一个元素添加到布隆过滤器中，然后使用`bf.exists`命令来检查一个元素是否可能存在于布隆过滤器中。

布隆过滤器有很多实际应用场景，例如网络爬虫中对URL的去重操作，黑名单过滤，垃圾邮件过滤等。但需要注意的是，由于误判率存在，所以在处理需要100%准确性的任务时需要谨慎使用。



**误差来源：**

布隆过滤器的误差主要源于**哈希冲突**。布隆过滤器使用多个哈希函数将输入映射到一个位数组中。当我们查询一个元素是否存在于布隆过滤器中时，我们会使用相同的哈希函数生成相应的哈希值，并检查位数组中对应的位置是否都为1。如果所有位置都为1，那么我们就认为该元素可能存在于布隆过滤器中。

然而，由于哈希函数可能会将不同的输入映射到相同的输出（即哈希冲突），因此可能存在一种情况，即尽管所有查询的位置都为1，但这些位置可能是由其他元素映射过来的。这就是所谓的误判，也就是说，布隆过滤器可能会错误地认为某个不存在的元素其实存在。

需要注意的是，布隆过滤器只会产生假阳性误判，也就是说，它可能会错误地认为某个不存在的元素存在，但如果它说某个元素不存在，那么这个元素肯定不存在。



**为什么不建议删除元素：**

这种情况也造成了布隆过滤器的删除问题，因为布隆过滤器的每一个 bit 并不是独占的，很有可能多个元素共享了某一位。如果我们直接删除这一位的话，会影响其他的元素特性

布隆过滤器可以添加元素，但是不能删除元素。因为删掉元素会导致误判率增加。



### 3）布隆过滤器结合bitmap实操

![](https://github.com/vankykoo/image/blob/main/082.png?raw=true)





##八、缓存预热、缓存雪崩、缓存穿透、缓存击穿

### 1）缓存预热

缓存预热是一种常见的优化策略，主要用于在系统启动或上线后，提前将相关的缓存数据直接加载到缓存系统中。这样做的目的是避免在用户请求时，需要先查询数据库，然后再将数据缓存的问题。通过预热，用户可以直接查询到事先被预热的缓存数据。

实现缓存预热的方法有多种：

1. 手动刷新：可以直接编写一个缓存刷新页面，在系统上线时手动操作。
2. 自动加载：如果数据量不大，可以在项目启动的时候自动进行加载。
3. 定时刷新：设置定时任务，定期刷新缓存。
4. 利用大数据统计用户访问的热点数据，在项目启动时将这些热点数据提前查询并保存到Redis中。

总的来说，缓存预热是一种有效的提高系统性能和用户体验的策略。



### 2）缓存雪崩

缓存雪崩是指**在某一时刻发生大规模的缓存失效，这会导致大量的请求直接打在数据库上，从而引发数据库压力巨大，甚至可能瞬间导致数据库宕机**。这种情况通常发生在缓存中的大量数据同时到达过期时间，而查询的数据量非常大。

解决缓存雪崩的策略有几种：

1. **避免缓存集中失效**：可以通过在原有的失效时间上加上一个随机值，比如1-5分钟随机，这样就避免了因为采用相同的过期时间导致的缓存雪崩。
2. **增加互斥锁**：控制数据库请求，重建缓存。
3. **提高缓存的高可用性**：例如，可以搭建Redis集群，提高Redis的容灾性。
4. **使用熔断机制**：当流量到达一定的阈值时，就直接返回“系统拥挤”之类的提示，防止过多的请求打在数据库上。
5. **多缓存结合**：ehcache本地缓存 + redis 缓存

总的来说，处理缓存雪崩需要对系统进行全方位的优化和调整，包括但不限于提高数据库和缓存服务的容灾能力、优化业务代码、合理设置缓存失效时间等。



### 3）缓存穿透

缓存穿透是指查询一个一定不存在的数据，由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。在流量大时，可能数据库就挂掉了，要是有人利用不存在的key频繁攻击我们的应用，这就是漏洞。

解决缓存穿透的策略有几种：

1. **参数校验**：我们可以对用户id做检验。比如你的合法id是15xxxxxx，以15开头的。如果用户传入了16开头的id，比如：16232323，则参数校验失败，直接把相关请求拦截掉。
2. **使用布隆过滤器**：布隆过滤器的作用是某个 key 不存在，那么就一定不存在，它说某个 key 存在，那么很大可能是存在 (存在一定的误判率)。于是我们可以在缓存之前再加一层布隆过滤器，在查询的时候先去布隆过滤器查询 key 是否存在，如果不存在就直接返回。
3. **缓存空值**：当某个用户id在缓存中查不到，在数据库中也查不到时，也需要将该用户id缓存起来，只不过值是空的。这样后面的请求，再拿相同的用户id发起请求时，就能从缓存中获取空数据，直接返回了，而无需再去查一次数据库。



#### 谷歌GuavaBloomFilter使用案例

![](https://github.com/vankykoo/image/blob/main/083.png?raw=true)

①引依赖

```xml
<!--guava Google 开源的 Guava 中自带的布隆过滤器-->
<dependency>
  <groupId>com.google.guava</groupId>
  <artifactId>guava</artifactId>
  <version>23.0</version>
</dependency>
```

②服务类

```java
public class GuavaBloomFilterService{
    public static final int _1W = 10000;
    //布隆过滤器里预计要插入多少数据
    public static int size = 100 * _1W;
    //误判率,它越小误判的个数也就越少(思考，是不是可以设置的无限小，没有误判岂不更好)
    //fpp the desired false positive probability
    public static double fpp = 0.03;
    // 构建布隆过滤器
    private static BloomFilter<Integer> bloomFilter = BloomFilter.create(Funnels.integerFunnel(), size,fpp);
    public void guavaBloomFilter(){
        //1 先往布隆过滤器里面插入100万的样本数据
        for (int i = 1; i <=size; i++) {
            bloomFilter.put(i);
        }
        //故意取10万个不在过滤器里的值，看看有多少个会被认为在过滤器里
        List<Integer> list = new ArrayList<>(10 * _1W);
        for (int i = size+1; i <= size + (10 *_1W); i++) {
            if (bloomFilter.mightContain(i)) {
                log.info("被误判了:{}",i);
                list.add(i);
            }
        }
        log.info("误判的总数量：:{}",list.size());
    }
}
```



其他应用：

![](https://github.com/vankykoo/image/blob/main/084.png?raw=true)





### 4）缓存击穿

缓存击穿是指一个热点的Key，有大并发集中对其进行访问，突然间这个Key失效了，导致大并发全部打在数据库上，导致数据库压力剧增。这种现象就叫做缓存击穿。

解决缓存击穿的策略有几种：

1. **设置热点key永不过期**：如果业务允许的话，对于热点的key可以设置永不过期的key[1](https://zhuanlan.zhihu.com/p/346651831)。
2. **使用互斥锁**：如果缓存失效的情况，只有拿到锁才可以查询数据库，降低了在同一时刻打在数据库上的请求，防止数据库打死。

总的来说，处理缓存击穿需要对系统进行全方位的优化和调整，包括但不限于提高数据库和缓存服务的容灾能力、优化业务代码、合理设置缓存失效时间等。



#### 差异失效时间预防缓存击穿问题

![](https://github.com/vankykoo/image/blob/main/085.png?raw=true)



service

```java
@Service
@Slf4j
public class JHSTaskService
{
    public  static final String JHS_KEY="jhs";
    public  static final String JHS_KEY_A="jhs:a";
    public  static final String JHS_KEY_B="jhs:b";

    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * 偷个懒不加mybatis了，模拟从数据库读取100件特价商品，用于加载到聚划算的页面中
     * @return
     */
    private List<Product> getProductsFromMysql() {
        List<Product> list=new ArrayList<>();
        for (int i = 1; i <=20; i++) {
            Random rand = new Random();
            int id= rand.nextInt(10000);
            Product obj=new Product((long) id,"product"+i,i,"detail");
            list.add(obj);
        }
        return list;
    }

    //@PostConstruct
  //未使用双缓存版本
    public void initJHS(){
        log.info("启动定时器淘宝聚划算功能模拟.........."+ DateUtil.now());
        new Thread(() -> {
            //模拟定时器，定时把数据库的特价商品，刷新到redis中
            while (true){
                //模拟从数据库读取100件特价商品，用于加载到聚划算的页面中
                List<Product> list=this.getProductsFromMysql();
                //采用redis list数据结构的lpush来实现存储
                this.redisTemplate.delete(JHS_KEY);
                //lpush命令
                this.redisTemplate.opsForList().leftPushAll(JHS_KEY,list);
                //间隔一分钟 执行一遍
                try { TimeUnit.MINUTES.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

                log.info("runJhs定时刷新..............");
            }
        },"t1").start();
    }

  //使用双缓存且设置差异失效时间版本
    @PostConstruct
    public void initJHSAB(){
        log.info("启动AB定时器计划任务淘宝聚划算功能模拟.........."+DateUtil.now());
        new Thread(() -> {
            //模拟定时器，定时把数据库的特价商品，刷新到redis中
            while (true){
                //模拟从数据库读取100件特价商品，用于加载到聚划算的页面中
                List<Product> list=this.getProductsFromMysql();
                //先更新B缓存
                this.redisTemplate.delete(JHS_KEY_B);
                this.redisTemplate.opsForList().leftPushAll(JHS_KEY_B,list);
                this.redisTemplate.expire(JHS_KEY_B,20L,TimeUnit.DAYS);
                //再更新A缓存
                this.redisTemplate.delete(JHS_KEY_A);
                this.redisTemplate.opsForList().leftPushAll(JHS_KEY_A,list);
                this.redisTemplate.expire(JHS_KEY_A,15L,TimeUnit.DAYS);
                //间隔一分钟 执行一遍
                try { TimeUnit.MINUTES.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

                log.info("runJhs定时刷新双缓存AB两层..............");
            }
        },"t1").start();
    }
}
```



controller

```java
@RestController
@Slf4j
@Api(tags = "聚划算商品列表接口")
public class JHSProductController
{
    public  static final String JHS_KEY="jhs";
    public  static final String JHS_KEY_A="jhs:a";
    public  static final String JHS_KEY_B="jhs:b";

    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * 分页查询：在高并发的情况下，只能走redis查询，走db的话必定会把db打垮
     * @param page
     * @param size
     * @return
     */
    @RequestMapping(value = "/pruduct/find",method = RequestMethod.GET)
    @ApiOperation("按照分页和每页显示容量，点击查看")
    public List<Product> find(int page, int size) {
        List<Product> list=null;

        long start = (page - 1) * size;
        long end = start + size - 1;

        try {
            //采用redis list数据结构的lrange命令实现分页查询
            list = this.redisTemplate.opsForList().range(JHS_KEY, start, end);
            if (CollectionUtils.isEmpty(list)) {
                //TODO 走DB查询
            }
            log.info("查询结果：{}", list);
        } catch (Exception ex) {
            //这里的异常，一般是redis瘫痪 ，或 redis网络timeout
            log.error("exception:", ex);
            //TODO 走DB查询
        }

        return list;
    }

    @RequestMapping(value = "/pruduct/findab",method = RequestMethod.GET)
    @ApiOperation("防止热点key突然失效，AB双缓存架构")
    public List<Product> findAB(int page, int size) {
        List<Product> list=null;
        long start = (page - 1) * size;
        long end = start + size - 1;
        try {
            //采用redis list数据结构的lrange命令实现分页查询
            list = this.redisTemplate.opsForList().range(JHS_KEY_A, start, end);
            if (CollectionUtils.isEmpty(list)) {
                log.info("=========A缓存已经失效了，记得人工修补，B缓存自动延续5天");
                //用户先查询缓存A(上面的代码)，如果缓存A查询不到（例如，更新缓存的时候删除了），再查询缓存B
                this.redisTemplate.opsForList().range(JHS_KEY_B, start, end);
                //TODO 走DB查询
            }
            log.info("查询结果：{}", list);
        } catch (Exception ex) {
            //这里的异常，一般是redis瘫痪 ，或 redis网络timeout
            log.error("exception:", ex);
            //TODO 走DB查询
        }
        return list;
    }
}
```



![](https://github.com/vankykoo/image/blob/main/086.png?raw=true)





## 九、分布式锁

### 1）介绍

#### 1.介绍

Redis的分布式锁是一种非常有用的并发控制设备，它能够为多进程提供一种基于共享资源的排他式工作方式。在分布式系统中，一个应用往往会部署在多台机器上（多节点），在有些场景中，为了保证数据不重复，要求在同一时刻，同一任务只在一个节点上运行，即**保证某一方法同一时刻只能被一个线程执行**。这时候就需要使用到分布式锁。

Redis实现分布式锁的核心命令是`SETNX key value`。`SETNX`命令的作用是：如果指定的key不存在，则创建并为其设置值，然后返回状态码1；如果指定的key存在，则直接返回0。如果返回值为1，代表获得该锁；此时其他进程再次尝试创建时，由于key已经存在，则都会返回0 ，代表锁已经被占用。当获得锁的进程处理完成业务后，再通过`del`命令将该key删除，其他进程就可以再次竞争性地进行创建，获得该锁。

然而，在实际应用中，我们通常会使用一些成熟的库来帮助我们更好地实现和管理分布式锁。例如Redission就是一个很好的选择。Redission提供了多种类型的分布式锁，包括可重入锁、公平锁、联锁等。它提供了简单的API来获取锁、释放锁，并且支持超时机制和异步操作。

以下是一个使用Redission实现分布式锁的例子：

```java
// 1.构造redisson实现分布式锁必要的
ConfigConfig config = new Config();
config.useSingleServer().setAddress("redis://127.0.0.1:5379").setPassword("123456").setDatabase(0);
// 2.构造RedissonClient
RedissonClient redissonClient = Redisson.create(config);
// 3.获取锁对象实例
RLock rLock = redissonClient.getLock(lockKey);
try {    
  // 4.尝试获取锁    
  boolean res = rLock.tryLock(waitTimeout, leaseTime, TimeUnit.SECONDS);    
  if (res) {        
    //成功获得锁，在这里处理业务    
  }} catch (Exception e) {    
  throw new RuntimeException("aquire lock fail");
} finally {    
  //无论如何, 最后都要解锁    
  rLock.unlock();
}
```



#### 2.分布式锁需要具备的条件

①独占性  ②高可用  ③防死锁  ④不乱抢  ⑤重入性

> 在分布式系统中，由于系统的并发性和分布式特性，可能会出现多个进程或线程同时操作同一份资源的情况。如果没有合适的控制机制，这可能会导致数据的不一致性或者其他的并发问题。这就是我们需要使用**分布式锁**的原因。
>
> 分布式锁可以确保在分布式环境中，同一时刻只有一个进程或线程能够操作某个资源。这对于保证数据的一致性和防止并发问题非常重要。例如，我们可能需要在一个电商网站上实现一个秒杀功能。在这种情况下，我们需要确保每个商品只能被秒杀一次。如果没有分布式锁，可能会出现多个用户同时秒杀同一个商品的情况，导致库存出现负数。
>
> 此外，分布式锁还可以用于实现一些需要全局同步的功能。例如，在一个游戏中，我们可能需要定期更新全球排行榜。在这种情况下，我们可以使用分布式锁来确保在更新排行榜的过程中，不会有其他的进程或线程来修改排行榜的数据。
>
> 总的来说，分布式锁是一种非常重要的并发控制机制，它可以帮助我们在复杂的分布式环境中保证数据的一致性和系统的正确性。



### 2）案例

#### 1.初始条件

**问题：**

nginx分布式微服务架构，用两台服务器，每台服务器加synchronized或者lock锁，但是还是出现bug（超卖）。

**原因：**

在单机环境下，可以使用synchronized或Lock来实现。但是在分布式系统中，因为竞争的线程可能不在同一个节点上（同一个jvm中），所以需要一个让所有进程都能访问到的锁来实现(比如redis或者zookeeper来构建)不同进程jvm层面的锁就不管用了，那么可以利用第三方的一个组件，来获取锁，未获取到锁，则阻塞当前想要运行的线程

 **代码：**

```java
private Lock lock = new ReentrantLock();

public String sell(){
  String retMessage = "";

  lock.lock();

  try {
    //1.查询库存信息
    String result = redisTemplate.opsForValue().get("inventory001");
    //2.判断库存是否足够
    Integer inventoryNumber = result == null ? 0 : Integer.parseInt(result);
    //3.扣减库存，每次减一个
    if (inventoryNumber > 0){
      redisTemplate.opsForValue().set("inventory001",String.valueOf(--inventoryNumber));
      retMessage = "售出一个商品，库存剩余：" + inventoryNumber;
      System.out.println(retMessage + "\t" + "服务端口号：" + port);
    }else {
      retMessage = "商品卖完了┭┮﹏┭┮";
    }

  }finally {
    lock.unlock();
  }

  return retMessage + "\t" + "服务端口号：" + port;
}
```



####2.版本二

介绍：使用分布式锁，如果没有获取到锁，就递归重试（再调用一遍本方法）

问题：递归容易导致StackOverflow

代码：

```java
public String sell(){
  String retMessage = "";
  String key = "vkRedisLock";
  String uuidValue = UUID.randomUUID() + ":" + Thread.currentThread().getId();

  Boolean flag = redisTemplate.opsForValue().setIfAbsent(key, uuidValue);

  if (!flag){
    //失败
    //暂停20毫秒，递归重试
    try {
      TimeUnit.MILLISECONDS.sleep(20);
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    }
    sell();
  }else{
    //成功
    try {
      //1.查询库存信息
      String result = redisTemplate.opsForValue().get("inventory001");
      //2.判断库存是否足够
      Integer inventoryNumber = result == null ? 0 : Integer.parseInt(result);
      //3.扣减库存，每次减一个
      if (inventoryNumber > 0){
        redisTemplate.opsForValue().set("inventory001",String.valueOf(--inventoryNumber));
        retMessage = "售出一个商品，库存剩余：" + inventoryNumber;
        System.out.println(retMessage + "\t" + "服务端口号：" + port);
      }else {
        retMessage = "商品卖完了┭┮﹏┭┮";
      }

    }finally {
      //释放锁
      redisTemplate.delete(key);
    }
  }

  return retMessage + "\t" + "服务端口号：" + port;
}
```



#### 3.版本三

介绍：使用while循环，没获取到锁就sleep20毫秒，重新调用。

问题：可能还没有释放锁程序宕机，导致锁无法释放

代码：

```java
public String sell(){
  String retMessage = "";
  String key = "vkRedisLock";
  String uuidValue = UUID.randomUUID() + ":" + Thread.currentThread().getId();

  //如果没有获取到锁，就自旋
  while(!redisTemplate.opsForValue().setIfAbsent(key, uuidValue)){
    //休息20毫秒
    try {
      TimeUnit.MILLISECONDS.sleep(20);
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    }
  }

  try {
    //1.查询库存信息
    String result = redisTemplate.opsForValue().get("inventory001");
    //2.判断库存是否足够
    Integer inventoryNumber = result == null ? 0 : Integer.parseInt(result);
    //3.扣减库存，每次减一个
    if (inventoryNumber > 0){
      redisTemplate.opsForValue().set("inventory001",String.valueOf(--inventoryNumber));
      retMessage = "v3.2--售出一个商品，库存剩余：" + inventoryNumber;
      System.out.println(retMessage + "\t" + "服务端口号：" + port);
    }else {
      retMessage = "商品卖完了┭┮﹏┭┮";
    }
  }finally {
    //释放锁
    redisTemplate.delete(key);
  }

  return retMessage + "\t" + "服务端口号：" + port;
}
```





#### 4.版本四

介绍：设置锁释放时间，【加锁和设置释放时间必须在同一行，保证原子性】

问题：在锁自动释放之前还没有执行完程序，锁自动释放，下一个程序获取到了锁，然后还没释放锁时，上一个程序完成，释放了第二个程序的锁。

代码：

```java
public String sell(){
  String retMessage = "";
  String key = "vkRedisLock";
  String uuidValue = UUID.randomUUID() + ":" + Thread.currentThread().getId();

  //如果没有获取到锁，就自旋
  //设置锁自动释放时间
  while(!redisTemplate.opsForValue().setIfAbsent(key, uuidValue,30L,TimeUnit.SECONDS)){
    //休息20毫秒
    try {
      TimeUnit.MILLISECONDS.sleep(20);
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    }
  }

  try {
    //1.查询库存信息
    String result = redisTemplate.opsForValue().get("inventory001");
    //2.判断库存是否足够
    Integer inventoryNumber = result == null ? 0 : Integer.parseInt(result);
    //3.扣减库存，每次减一个
    if (inventoryNumber > 0){
      redisTemplate.opsForValue().set("inventory001",String.valueOf(--inventoryNumber));
      retMessage = "v3.2--售出一个商品，库存剩余：" + inventoryNumber;
      System.out.println(retMessage + "\t" + "服务端口号：" + port);
    }else {
      retMessage = "商品卖完了┭┮﹏┭┮";
    }
  }finally {
    //释放锁
    redisTemplate.delete(key);
  }

  return retMessage + "\t" + "服务端口号：" + port;
}
```



#### 5.版本五

改进：在释放锁的时候判断一下是不是自己的锁

问题：最后判断是不是自己的锁和释放锁的代码不是原子性的。

```java
public String sell(){
  String retMessage = "";
  String key = "vkRedisLock";
  String uuidValue = UUID.randomUUID() + ":" + Thread.currentThread().getId();

  //如果没有获取到锁，就自旋
  //设置锁自动释放时间
  while(!redisTemplate.opsForValue().setIfAbsent(key, uuidValue,30L,TimeUnit.SECONDS)){
    //休息20毫秒
    try {
      TimeUnit.MILLISECONDS.sleep(20);
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    }
  }

  try {
    //1.查询库存信息
    String result = redisTemplate.opsForValue().get("inventory001");
    //2.判断库存是否足够
    Integer inventoryNumber = result == null ? 0 : Integer.parseInt(result);
    //3.扣减库存，每次减一个
    if (inventoryNumber > 0){
      redisTemplate.opsForValue().set("inventory001",String.valueOf(--inventoryNumber));
      retMessage = "v3.2--售出一个商品，库存剩余：" + inventoryNumber;
      System.out.println(retMessage + "\t" + "服务端口号：" + port);
    }else {
      retMessage = "商品卖完了┭┮﹏┭┮";
    }
  }finally {
    //释放锁前，先判断是不是自己创建的锁
    if (uuidValue.equals(redisTemplate.opsForValue().get(key))) {
      redisTemplate.delete(key);
    }
  }

  return retMessage + "\t" + "服务端口号：" + port;
}
```



#### 6.版本六（lua脚本）

改进：使用lua脚本实现锁的判断和释放操作的原子性

问题：不满足可重入性问题，比如如果已经获取到锁，主要业务代码里面需要调用其他方法，正好其他方法也需要获取锁，这时锁被上一层代码占用。

代码：

```java
public String sell(){
  String retMessage = "";
  String key = "vkRedisLock";
  String uuidValue = UUID.randomUUID() + ":" + Thread.currentThread().getId();

  //如果没有获取到锁，就自旋
  //设置锁自动释放时间
  while(!redisTemplate.opsForValue().setIfAbsent(key, uuidValue,30L,TimeUnit.SECONDS)){
    //休息20毫秒
    try {
      TimeUnit.MILLISECONDS.sleep(20);
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    }
  }

  try {
    //1.查询库存信息
    String result = redisTemplate.opsForValue().get("inventory001");
    //2.判断库存是否足够
    Integer inventoryNumber = result == null ? 0 : Integer.parseInt(result);
    //3.扣减库存，每次减一个
    if (inventoryNumber > 0){
      redisTemplate.opsForValue().set("inventory001",String.valueOf(--inventoryNumber));
      retMessage = "v3.2--售出一个商品，库存剩余：" + inventoryNumber;
      System.out.println(retMessage + "\t" + "服务端口号：" + port);
    }else {
      retMessage = "商品卖完了┭┮﹏┭┮";
    }
  }finally {
    //释放锁代码需要具备原子性
    String lua = "if (redis.call('get',KEYS[1]) == ARGV[1]) " +
      "then " +
      "return redis.call('del',KEYS[1]) " +
      "else " +
      "return 0 end";
    redisTemplate.execute(new DefaultRedisScript<>(lua, Boolean.class), Arrays.asList(key),uuidValue);
  }

  return retMessage + "\t" + "服务端口号：" + port;
}
```



#### 7.版本七（可重入锁介绍）

**可重入锁**又名递归锁，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁(前提，锁对象得是同一个对象)，不会因为之前已经获取过还没释放而阻塞。

如果是1个有 **synchronized** 修饰的递归调用方法，程序第2次进入被自己阻塞了岂不是天大的笑话，出现了作茧自缚。

所以Java中**ReentrantLock和synchronized**都是可重入锁，可重入锁的一个优点是可一定程度避免死锁。

> 可重入锁，也被称为递归锁，是一种特殊的锁，它允许同一个线程多次获取同一把锁。具体来说，如果一个线程已经持有了某个锁，那么它可以再次请求获取这个锁，而不会被阻塞。
>
> 这种锁的主要作用是防止死锁。例如，如果一个线程在持有一个锁的同时尝试获取另一个锁，那么可能会发生死锁，因为其他线程可能同时持有这些锁。但是，如果我们使用可重入锁，那么同一个线程可以多次获取同一把锁，从而避免死锁。
>
> 在Java中，synchronized关键字和ReentrantLock类都提供了可重入锁的功能。当一个线程进入由synchronized关键字保护的代码块时，它可以再次进入该代码块而不会被阻塞。同样，当一个线程获取了一个ReentrantLock对象的锁时，它可以再次调用lock方法来重新获取该锁。
>
> 以下是一个使用ReentrantLock的例子：
>
> ```java
> ReentrantLock lock = new ReentrantLock();
> lock.lock();
> try {    
>   // access the resource protected by this lock
> } finally {    
>   lock.unlock();
> }
> ```
>
> 在这个例子中，如果同一个线程再次调用lock.lock()，它将不会被阻塞，并且必须调用相同次数的unlock()方法来释放锁。这就是可重入性的含义。



> 为什么需要可重入锁？
>
> Redis分布式锁实现可重入锁的关键在于，每个锁不仅关联一个键，还需要关联一个客户端标识，通常是UUID。当客户端尝试获取锁时，会同时发送键和客户端标识。如果键不存在，Redis会创建一个新的键，并将客户端标识设置为该键的值，然后返回成功。如果键已经存在，Redis会检查存储的客户端标识是否与发送的客户端标识相同。如果相同，表示是同一个客户端再次请求获取锁，因此可以成功获取，这就实现了可重入。
>
> 同时，为了防止死锁，每个锁都应该有一个自动过期的时间。但是这里需要注意，只有持有锁的客户端才能释放这个锁。也就是说，在释放锁之前，需要检查存储的客户端标识是否与发送的客户端标识相同。



两种可重入锁：

* 隐式锁：synchronized

  指的是可重复可递归调用的锁，在外层使用锁之后，在内层仍然可以使用，并且不发生死锁，这样的锁就叫做可重入锁。简单的来说就是：**在一个synchronized修饰的方法或代码块的内部调用本类的其他synchronized修饰的方法或代码块时，是永远可以得到锁的**。与可重入锁相反，不可重入锁不可递归调用，递归调用就发生死锁。

  > 原理：
  >
  > synchronized是Java中的关键字，用于实现同步机制，它是可重入锁。可重入锁，也被称为递归锁，是指在同一线程内，外层函数获得锁之后，内层递归函数仍然有获取该锁的权限。这种情况下，线程可以直接进入方法，无需等待。这样做的一个主要优点是防止死锁。
  >
  > synchronized的可重入性是由JVM底层通过计数器来实现的。当一个线程请求一个由synchronized修饰的方法时，JVM会检查该方法是否有锁对象关联。如果没有，则创建一个并与之关联，并将计数器设为1；如果已经存在锁对象，则检查持有锁的线程是否就是当前请求的线程，如果是则将计数器加1。当线程退出同步代码块时，计数器会递减，如果计数器为0，则释放该锁。
  >
  > 这就是synchronized作为可重入锁的原理。这种设计使得线程可以在已经持有某个对象锁的情况下，再次申请获取该对象的锁，而不会造成死锁。

* 显式锁：ReentrantLock和Lock



**使用lua脚本实现加锁，解锁：**

* 加锁
  * 先判断是否有锁
  * 返回0：没锁，使用hset获得自己的锁
  * 返回1：有锁，需要判断锁是不是自己的

```lua
if redis.call('exists',KEYS[1]) == 0 or redis.call('hexists',KEYS[1],ARGV[1]) == 1 
then
  	<!--无论有没有锁都执行下面代码-->
	redis.call('hincrby',KEYS[1],ARGV[1],1)
  	redis.call('expire',KEYS[1],ARGV[2])
  	return 1
else
  	return 0
end
```

* 解锁：
  * 判断是否有锁且是否是自己的锁
  * 返回0：根本没有锁，返回nil
  * 返回不是0，说明有锁且是自己的锁，直接调用 hincrby 命令减一，表示解一次锁，直到变为0就可以删除锁了。

```lua
if redis.call('HEXISTS',KEYS[1],ARGV[1]) == 0 
then 
  return nil
elseif
  redis.call('HINCRBY',KEYS[1],ARGV[1],-1) == 0
then
  return redis.call('del',KEYS[1])
else
  return 0
end
```



整合到java代码中：

写一个自研可重入锁，获得锁和解锁用上面lua脚本。

```java
//自研锁
public class RedisDistributedLock implements Lock {

    private StringRedisTemplate redisTemplate;
    private String lockName;
    private String uuidValue;
    private Long expireTime;

    public RedisDistributedLock(StringRedisTemplate redisTemplate, String lockName) {
        this.redisTemplate = redisTemplate;
        this.lockName = lockName;
        this.expireTime = 50L;
        this.uuidValue = UUID.randomUUID() + ":" + Thread.currentThread().getId();
    }

    @Override
    public void lock() {
        tryLock();
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {

    }

    @Override
    public boolean tryLock() {
        try {
            return tryLock(-1L,TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        if (time == -1L){
            String luaScript = "if redis.call('exists',KEYS[1]) == 0 or redis.call('hexists',KEYS[1],ARGV[1]) == 1 " +
                    "then " +
                        "redis.call('hincrby',KEYS[1],ARGV[1],1) " +
                        "redis.call('expire',KEYS[1],ARGV[2]) " +
                        "return 1 " +
                    "else " +
                        "return 0 " +
                    "end";

            while (!redisTemplate.execute(new DefaultRedisScript<>(luaScript, Boolean.class), Arrays.asList(lockName),uuidValue,String.valueOf(expireTime))){
                TimeUnit.MILLISECONDS.sleep(20);
            }
            return true;

        }


        return false;
    }

    @Override
    public void unlock() {

        String luaScript = "if redis.call('HEXISTS',KEYS[1],ARGV[1]) == 0 " +
                "then " +
                    "return nil " +
                "elseif redis.call('HINCRBY',KEYS[1],ARGV[1],-1) == 0 " +
                "then " +
                    "return redis.call('del',KEYS[1]) " +
                "else " +
                    "return 0 " +
                "end";
        Long res = redisTemplate.execute(new DefaultRedisScript<>(luaScript, Long.class), Arrays.asList(lockName), uuidValue);
        if (res == null){
            throw new RuntimeException("这个锁不存在！/(ㄒoㄒ)/~~");
        }
    }

    @Override
    public Condition newCondition() {
        return null;
    }
}
```



版本七service代码

```java
public String sell(){
  String retMessage = "";

  //使用自研可重入锁
  Lock lock = new RedisDistributedLock(redisTemplate,"vkRedisLock");

  lock.lock();

  try {
    //1.查询库存信息
    String result = redisTemplate.opsForValue().get("inventory001");
    //2.判断库存是否足够
    Integer inventoryNumber = result == null ? 0 : Integer.parseInt(result);
    //3.扣减库存，每次减一个
    if (inventoryNumber > 0){
      redisTemplate.opsForValue().set("inventory001",String.valueOf(--inventoryNumber));
      retMessage = "v3.2--售出一个商品，库存剩余：" + inventoryNumber;
      System.out.println(retMessage + "\t" + "服务端口号：" + port);
    }else {
      retMessage = "商品卖完了┭┮﹏┭┮";
    }
  }finally {
    lock.unlock();
  }

  return retMessage + "\t" + "服务端口号：" + port;
}
```



#### 7.1、版本七引入工厂模式

可以达到解耦需求



工厂代码

```java
@Component
public class DistributedLockFactory {

    @Autowired
    private StringRedisTemplate redisTemplate;
    //定义uuid，防止重入时uuid不同
    private String uuid;

    public DistributedLockFactory() {
        this.uuid = UUID.randomUUID().toString();
    }

    public Lock getLockByType(String lockType) {
        if (lockType == null) return null;

        if (lockType.equalsIgnoreCase("REDIS")) {
            return new RedisDistributedLock(redisTemplate, "redisLock",uuid);
        } else if (lockType.equalsIgnoreCase("ZOOKEEPER")) {
            //TODO 返回zookeeper结点
            return null;
        } else if (lockType.equalsIgnoreCase("MYSQL")) {
            //TODO 返回MYSQL 锁
            return null;
        }
        return null;
    }
}
```

使用

```java
@Autowired
private DistributedLockFactory distributedLockFactory;

public String sell(){
  String retMessage = "";

  //使用自研可重入锁
  //Lock lock = new RedisDistributedLock(redisTemplate,"vkRedisLock");
  Lock lock = distributedLockFactory.getLockByType("redis");

  lock.lock();

  try {
    //1.查询库存信息
    String result = redisTemplate.opsForValue().get("inventory001");
    //2.判断库存是否足够
    Integer inventoryNumber = result == null ? 0 : Integer.parseInt(result);
    //3.扣减库存，每次减一个
    //测试重入
    testEntrant();

    if (inventoryNumber > 0){
      redisTemplate.opsForValue().set("inventory001",String.valueOf(--inventoryNumber));
      retMessage = "v3.2--售出一个商品，库存剩余：" + inventoryNumber;
      System.out.println(retMessage + "\t" + "服务端口号：" + port);
    }else {
      retMessage = "商品卖完了┭┮﹏┭┮";
    }
  }finally {
    lock.unlock();
  }

  return retMessage + "\t" + "服务端口号：" + port;
}

private void testEntrant() {
  Lock lock = distributedLockFactory.getLockByType("redis");

  lock.lock();

  try {
    System.out.println("Test");
  }finally {
    lock.unlock();
  }
}
```



#### 8.版本八--自动续期

防止在锁自动释放时间内没有完成业务代码，导致张冠李戴

思路：使用定时任务定时监测是否已经完成业务代码，如果没有完成就用lua脚本自动加时间。

```lua
if redis.call('HEXISTS',KEYS[1],ARGV[1]) == 1
then 
  return redis.call('expire',KEYS[1],ARGV[2])
else
  return 0
end
```



在自研redis锁中，每当创建锁成功就调用一下方法

```java
private void renewExpire() {
  String lua = "if redis.call('HEXISTS',KEYS[1],ARGV[1]) == 1 " +
    "then " +
    "return redis.call('expire',KEYS[1],ARGV[2]) " +
    "else " +
    "return 0 " +
    "end";

  new Timer().schedule(new TimerTask() {
    @Override
    public void run() {
      if (redisTemplate.execute(new DefaultRedisScript<>(lua, Boolean.class), Arrays.asList(lockName),uuidValue,String.valueOf(expireTime))){
        renewExpire();
      }
    }
  },(this.expireTime * 1000) / 3);
}
```



## 十、RedLock

###1）单点部署问题

**"Redis单点部署"是指在一个系统中只部署一个Redis服务器实例**。在这种情况下，所有的客户端请求都会发送到这个服务器。这种部署方式简单易行，但也存在一些问题，比如单点故障、性能瓶颈和数据丢失风险。这就是为什么在生产环境中，我们通常会选择使用Redis的主从复制、哨兵模式或集群模式来提高系统的可用性和性能。

> 如果Redis是单点部署的，可能会带来以下问题：
>
> 1. **单点故障**：如果Redis服务器发生故障，那么所有的请求都会失败，这将导致整个系统无法正常运行。
> 2. **性能瓶颈**：由于所有的请求都发送到同一个Redis服务器，所以在高并发的情况下，可能会出现性能瓶颈。
> 3. **数据丢失风险**：如果Redis服务器发生故障，那么存储在其中的数据可能会丢失。
>
> 解决上述问题的方法包括：
>
> 1. **主从复制**：通过配置主从复制，可以实现数据的备份，同时也可以通过读写分离来提高系统的性能。
> 2. **哨兵模式**：哨兵模式可以用来监控Redis服务器的运行状态，并在主服务器发生故障时自动切换到备用服务器。
> 3. **集群模式**：通过配置Redis集群，可以实现数据的分片存储，从而提高系统的可用性和性能。

![](https://github.com/vankykoo/image/blob/main/087.png?raw=true)



### 2）RedLock、Redisson介绍

RedLock是一种分布式锁算法，由Redis的创建者Salvatore Sanfilippo提出。该算法确保在分布式系统中，即使有多个Redis实例，也只能由一个节点安全地获取锁。

RedLock的工作原理如下：

1. 获取毫秒级别的时间戳，同时设定一个预定的锁定时间。
2. 通过SET命令在每个独立的实例中设置一个带有随机值的键（仅当键不存在时），并为该键设置超时时间。这个过程在所有独立实例中依次进行。如果某个实例宕机，则立即跳过。
3. 如果在给定的超时时间内，成功在大多数实例中设置了锁，那么就获取到了锁。如果没有在规定时间内获取到锁，则认为没有获取到锁。
4. 如果获取到了锁，那么可以在预定的锁定时间内进行操作或者续约锁。如果没有获取到锁或者发生了故障或超时，那么就解锁所有实例并重试。
5. 释放锁时，使用Lua脚本检查键是否具有预期的随机值。如果是，则可以删除键；如果不是，则保留键，因为它可能是一个新的锁。

RedLock提供了良好的保证，并且没有单点故障，因此你可以非常有信心地使用单个锁，并且不会发生死锁。



RedLock是Redis的创建者Salvatore Sanfilippo提出的一种分布式锁算法，用于在分布式系统中实现锁的功能。而Redisson是一个在Java中实现的Redis客户端，它提供了对Redis各种功能的高级封装，包括对RedLock算法的支持。

在Redisson中，你可以通过调用`getRedLock()`方法来获取一个RedLock实例。这个实例可以用来在多个Redis节点之间安全地获取和释放锁。此外，Redisson还提供了其他类型的锁，比如通过`getLock()`方法可以获取一个普通的分布式锁。

总的来说，RedLock和Redisson的关系就是：**RedLock是一种算法，Redisson是实现这个算法的工具。**



### 3）解决方案

使用RedLock算法解决：![](https://github.com/vankykoo/image/blob/main/088.png?raw=true)



#### 版本九

使用Redisson带的锁



①引入pom依赖

```xml
<!--redisson依赖-->
        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson</artifactId>
            <version>3.23.5</version>
        </dependency>
```

②在RedisConfig中添加

```java
@Bean
public Redisson redisson(){
  Config config = new Config();
  //单机版
  config.useSingleServer().setAddress("redis://192.168.200.134:6379").setDatabase(0).setPassword("qaz123");

  return (Redisson) Redisson.create(config);
}
```

③在service中，先从容器中取到Redisson，在方法中用redisson获取锁，并上锁；解锁时需要进行判断，需要满足已上锁且为自己的锁。

```java
@Autowired
private Redisson redisson;
public String sellByRedisson(){
  String retMessage = "";
  //使用redis
  RLock vkRedisLock = redisson.getLock("vkRedisLock");
  vkRedisLock.lock();

  try {
    //1.查询库存信息
    String result = redisTemplate.opsForValue().get("inventory001");
    //2.判断库存是否足够
    Integer inventoryNumber = result == null ? 0 : Integer.parseInt(result);
    //3.扣减库存，每次减一个
    if (inventoryNumber > 0){
      redisTemplate.opsForValue().set("inventory001",String.valueOf(--inventoryNumber));
      retMessage = "v3.2--售出一个商品，库存剩余：" + inventoryNumber;
      System.out.println(retMessage + "\t" + "服务端口号：" + port);
    }else {
      retMessage = "商品卖完了┭┮﹏┭┮";
    }
  } finally {
    //判断是否上锁且为自己的锁
    if (vkRedisLock.isLocked() && vkRedisLock.isHeldByCurrentThread()){
      vkRedisLock.unlock();
    }
  }

  return retMessage + "\t" + "服务端口号：" + port;
}
```



### 4）Redisson锁源码分析

#### 1.加锁逻辑

①通过exists判断，如果锁不存在，就设置值和过期时间，加锁成功。

②通过hexists，如果锁已存在，并且锁的是当前线程，则证明是重入锁，加锁成功。

③如果锁已存在，但锁的不是当前线程，则证明有其他线程持有锁，返回当前锁的过期时间（代表了锁key的剩余生存时间），加锁失败。



#### 2.锁的续期逻辑--看门狗

Redisson的锁续期机制是通过一个被称为"看门狗"的内部服务来实现的。当Redisson实例获取锁之后，如果该实例崩溃，那么这个锁可能会永远处于获取状态。为了避免这种情况，Redisson维护了一个锁看门狗，它会在锁持有者Redisson实例存活时延长锁的过期时间。

默认情况下，锁看门狗的超时时间是30秒，可以通过Config.lockWatchdogTimeout设置进行更改。在获取锁时可以定义leaseTime参数。在指定的时间间隔后，被锁定的锁将自动释放。

因此，即使在Redisson客户端崩溃或由于其他原因导致锁无法以正确的方式释放时，也可以防止锁被无限期地锁定。



#### 3.解锁逻辑

①如果释放锁的线程和已存在锁的线程不是同一个线程，返回nil

②如果释放锁的线程和已存在锁的线程是同一个线程，通过hincrby递减1，先释放一次锁。若剩余次数还大于0，则证明当前锁是重入锁，刷新过期时间。

③若剩余次数小于0，删除key并发布锁释放消息，解锁成功。



###5）多机版Redisson

这个锁的算法实现了多redis实例的情况，相对于单redis节点来说，优点在于 防止了 单节点故障造成整个服务停止运行的情况且在多节点中锁的设计，及多节点同时崩溃等各种意外情况有自己独特的设计方法。

Redisson 分布式锁支持 MultiLock 机制可以将多个锁合并为一个大锁，对一个大锁进行统一的申请加锁以及释放锁。

 

**最低保证分布式锁的有效性及安全性的要求如下：**

1.**互斥** ：任何时刻只能有一个client获取锁

2**.释放死锁** ：即使锁定资源的服务崩溃或者分区，仍然能释放锁

3.**容错性** ：只要多数redis节点（一半以上）在使用，client就可以获取和释放锁

 

**基于故障转移实现的redis主从无法真正实现Redlock:**

因为redis在进行主从复制时是异步完成的，比如在clientA获取锁后，主redis复制数据到从redis过程中崩溃了，导致没有复制到从redis中，然后从redis选举出一个升级为主redis,造成新的主redis没有clientA 设置的锁，这是clientB尝试获取锁，并且能够成功获取锁，导致互斥失效；



①配置

```properties
server.port=9090
spring.application.name=redlock

spring.swagger2.enabled=true

spring.redis.database=0
spring.redis.password=
spring.redis.timeout=3000
spring.redis.mode=single

spring.redis.pool.conn-timeout=3000
spring.redis.pool.so-timeout=3000
spring.redis.pool.size=10

spring.redis.single.address1=192.168.200.134:6381
spring.redis.single.address2=192.168.200.134:6382
spring.redis.single.address3=192.168.200.134:6383
```

②依赖

```xml
<dependency>
  <groupId>org.redisson</groupId>
  <artifactId>redisson</artifactId>
  <version>3.19.1</version>
</dependency>
```

③配置类

CacheConfiguration

```java
@Configuration
@EnableConfigurationProperties(RedisProperties.class)
public class CacheConfiguration {

    @Autowired
    RedisProperties redisProperties;

    @Bean
    RedissonClient redissonClient1() {
        Config config = new Config();
        String node = redisProperties.getSingle().getAddress1();
        node = node.startsWith("redis://") ? node : "redis://" + node;
        SingleServerConfig serverConfig = config.useSingleServer()
                .setAddress(node)
                .setTimeout(redisProperties.getPool().getConnTimeout())
                .setConnectionPoolSize(redisProperties.getPool().getSize())
                .setConnectionMinimumIdleSize(redisProperties.getPool().getMinIdle());
        if (StringUtils.isNotBlank(redisProperties.getPassword())) {
            serverConfig.setPassword(redisProperties.getPassword());
        }
        return Redisson.create(config);
    }

    @Bean
    RedissonClient redissonClient2() {
        Config config = new Config();
        String node = redisProperties.getSingle().getAddress2();
        node = node.startsWith("redis://") ? node : "redis://" + node;
        SingleServerConfig serverConfig = config.useSingleServer()
                .setAddress(node)
                .setTimeout(redisProperties.getPool().getConnTimeout())
                .setConnectionPoolSize(redisProperties.getPool().getSize())
                .setConnectionMinimumIdleSize(redisProperties.getPool().getMinIdle());
        if (StringUtils.isNotBlank(redisProperties.getPassword())) {
            serverConfig.setPassword(redisProperties.getPassword());
        }
        return Redisson.create(config);
    }

    @Bean
    RedissonClient redissonClient3() {
        Config config = new Config();
        String node = redisProperties.getSingle().getAddress3();
        node = node.startsWith("redis://") ? node : "redis://" + node;
        SingleServerConfig serverConfig = config.useSingleServer()
                .setAddress(node)
                .setTimeout(redisProperties.getPool().getConnTimeout())
                .setConnectionPoolSize(redisProperties.getPool().getSize())
                .setConnectionMinimumIdleSize(redisProperties.getPool().getMinIdle());
        if (StringUtils.isNotBlank(redisProperties.getPassword())) {
            serverConfig.setPassword(redisProperties.getPassword());
        }
        return Redisson.create(config);
    }

}
```

RedisPoolProperties

```java
@Data
public class RedisPoolProperties {

    private int maxIdle;

    private int minIdle;

    private int maxActive;

    private int maxWait;

    private int connTimeout;

    private int soTimeout;

    /**
     * 池大小
     */
    private  int size;

}
```

RedisProperties

```java
@ConfigurationProperties(prefix = "spring.redis", ignoreUnknownFields = false)
@Data
public class RedisProperties {

    private int database;

    /**
     * 等待节点回复命令的时间。该时间从命令发送成功时开始计时
     */
    private int timeout;

    private String password;

    private String mode;

    /**
     * 池配置
     */
    private RedisPoolProperties pool;

    /**
     * 单机信息配置
     */
    private RedisSingleProperties single;


}
```

RedisSingleProperties

```java
@Data
public class RedisSingleProperties {
    private  String address1;
    private  String address2;
    private  String address3;
}
```



④方法

这里直接写在controller里了

```java
public static final String LOCK_KEY = "vkRedisLock";

    @Autowired private RedissonClient redissonClient1;
    @Autowired private RedissonClient redissonClient2;
    @Autowired private RedissonClient redissonClient3;


    @GetMapping("/multiLock")
    public String getMultiLock(){
        String uuidValue = UUID.randomUUID() + ":" + Thread.currentThread().getId();

        RLock lock1 = redissonClient1.getLock(LOCK_KEY);
        RLock lock2 = redissonClient2.getLock(LOCK_KEY);
        RLock lock3 = redissonClient3.getLock(LOCK_KEY);

        RedissonMultiLock redissonMultiLock = new RedissonMultiLock(lock1, lock2, lock3);
        redissonMultiLock.lock();

        try {
            log.info("进入业务：" + uuidValue);
            TimeUnit.SECONDS.sleep(30);
            log.info("结束业务：" + uuidValue);
        }catch (Exception e){
            e.printStackTrace();
            log.error("exception:" + e.getCause() + "----" + e.getMessage());
        }finally{
            redissonMultiLock.unlock();
            log.info("锁释放成功,锁：{}",LOCK_KEY );
        }

        return "任务结束,ID：" + uuidValue;
    }
```



## 十一、Redis缓存---缓存淘汰策略

### 1）redis缓存介绍

* 默认内存：maxmemory为0，在64位电脑中的意思是没有限制；在32位的电脑中表示3GB。
* 推荐配置：设置为最大物理内存的四分之三
* 修改方法：在conf文件中修改maxmemory项；或者输入命令`config set maxmemory <byte>` 
* 查看内存使用情况：`info memory` 或者是 `config get maxmemory`



###2）过期键的删除策略

**①立即删除：**

缺点：对CPU不友好，用处理器性能换取存储空间

**②惰性删除：**

数据到达过期时间，不做处理。等下次访问该数据时，如果未过期，返回数据 ；发现已过期，删除，返回不存在。

缺点：对内存不好，用存储空间换取处理器性能

**③定期删除：**

定期删除策略是前两种策略的折中：

定期删除策略每隔一段时间执行一次删除过期键操作并通过限制删除操作执行时长和频率来减少删除操作对CPU时间的影响。

> 周期性轮询redis库中的时效性数据，采用随机抽取的策略，利用过期数据占比的方式控制删除频度 
>
> 特点1：CPU性能占用设置有峰值，检测频度可自定义设置 
>
> 特点2：内存压力不是很大，长期占用内存的冷数据会被持续清理 
>
> 总结：周期性抽查存储空间 （随机抽查，重点抽查）



###3）缓存淘汰策略

Redis的缓存淘汰策略主要有以下几种：

1. **noeviction**：当内存使用超过配置的时候会返回错误，不会驱逐任何键。
2. **allkeys-lru**：加入键的时候，如果过限，首先通过LRU算法驱逐最久没有使用的键。
3. **volatile-lru**：加入键的时候如果过限，首先从设置了过期时间的键集合中驱逐最久没有使用的键。
4. **allkeys-random**：加入键的时候如果过限，从所有key随机删除。
5. **volatile-random**：加入键的时候如果过限，从过期键的集合中随机驱逐。
6. **volatile-ttl**：从配置了过期时间的键中驱逐马上就要过期的键。
7. **volatile-lfu**：从所有配置了过期时间的键中驱逐使用频率最少的键。
8. **allkeys-lfu**：从所有键中驱逐使用频率最少的键。

这些策略在不同场景下有不同的效果。例如，如果你的应用对数据访问具有明显的热点特性，那么LRU或LFU可能是更好的选择。如果你希望在内存满时能够更公平地淘汰数据，那么随机淘汰策略可能更合适。总体来说，并没有一种“最好”的淘汰策略，需要根据具体应用和业务需求来选择。

![](C:\Users\86180\Desktop\picPick\161.png)



> LRU（Least Recently Used）和LFU（Least Frequently Used）都是常见的缓存淘汰算法，它们的主要区别在于淘汰策略的依据：
>
> 1. **LRU**：最近最少使用（Least Recently Used）策略会淘汰最长时间未被访问的数据。这种策略假设如果一个数据在最近一段时间内没有被访问，那么在将来一段时间内也不太可能被访问。因此，当缓存满时，LRU会淘汰最长时间未被访问的数据。
> 2. **LFU**：最不经常使用（Least Frequently Used）策略会淘汰在一段时间内访问次数最少的数据。这种策略假设如果一个数据在最近一段时间内被访问次数较少，那么在将来一段时间内也不太可能被频繁访问。因此，当缓存满时，LFU会淘汰访问次数最少的数据。
>
> 这两种策略各有优劣，适用于不同的场景。**LRU更适合于大部分情况下数据的访问模式都比较稳定，而LFU则更适合于数据的访问模式存在明显的热点特性。**



## 十二、Redis数据结果底层原理

### 1）介绍

![](https://github.com/vankykoo/image/blob/main/089.png?raw=true)

redis每个对象都是一个redisObject结构；**每个键值对都有一个dictEntry，value既不是直接作为字符串存储，也不是作为SDS动态字符串存储，而是存储在redisObject中**![](https://github.com/vankykoo/image/blob/main/090.png?raw=true)

![](https://github.com/vankykoo/image/blob/main/091.png?raw=true)



> redisObject字段含义：
>
> 在Redis中，每一个键值对都可以理解为是一个RedisObject。RedisObject的C源码定义了5个属性：type、encoding、lru、refcount和ptr。下面分别介绍这五个属性：
>
> 1. **type属性**：type主要存储当前值对象的数据类型。对于Redis数据库保存的键值对来说，键一定是一个字符串对象，而值则可以是**字符串对象(String)、列表对象(List)、哈希对象(Hash)、集合对象(Set)和有序集合对象(ZSet)**的其中一种。
> 2. **encoding属性**：encoding存储当前值对象底层编码的实现方式。<u>不同type对象对应不同的编码，每种编码又对应了一种底层数据结构。</u>
> 3. **lru属性**：lru**记录此对象最后一次访问的时间**。当redis内存回收算法设置为volatile-lru或者allkeys-lru时候redis会优先释放最久没有被访问的数据。
> 4. **refcount属性**：用于**共享计数**，类似于jvm的引用计数垃圾回收算法，当refcount为0时，表示没有其它对象引用，可以进行释放此对象。
> 5. **ptr属性**：ptr指针是**指向对象的底层实现数据结构**。



redisObject + Redis数据类型 + Redis 底层三者之间的关系图：

![](https://github.com/vankykoo/image/blob/main/093.png?raw=true)



### 2）总览

![](https://github.com/vankykoo/image/blob/main/094.png?raw=true)



![](https://github.com/vankykoo/image/blob/main/095.png?raw=true)



### 3）String类型

#### 1.三大物理编码方式

①int：里面是long数据类型，（-2^63,2^63-1).【浮点数会转化为字符串值】

②embstr：代表 embstr 格式的SDS ，保存长度小于44字节的字符串

③raw：保存长度大于44字节的字符



> Redis的String类型有三种物理编码类型，分别是：
>
> 1. **INT**：当字符串可以被解析为数字且数字在一定范围内时，Redis会选择INT编码。这种编码方式可以节省存储空间，并且对数字操作有更高的效率。
> 2. **RAW**：当字符串长度超过一定限制（例如44字节）或者字符串无法被解析为数字时，Redis会选择RAW编码。这种编码方式可以存储任意长度的字符串。
> 3. **EMBSTR**：当字符串较短（例如长度小于等于44字节）时，Redis会选择EMBSTR编码。这种编码方式将Redis对象头和SDS（简单动态字符串）连续存储在一起，减少了内存碎片和提高了空间利用率。



![](https://github.com/vankykoo/image/blob/main/096.png?raw=true)









#### 2.SDS简单动态字符串

Redis并没有直接使用C语言传统的字符串表示（以空字符结尾的字符数组，以下简称C字符串），而是自己构建了一种名为简单动态字符串（Simple Dynamic Strings，SDS）的抽象类型，并将SDS用作Redis的默认字符串表示。

SDS的主要特性包括：

1. **二进制安全**：SDS可以保存任何二进制数据，包括’\0’字符。这意味着在SDS中，字符串的长度不再由’\0’字符决定，而是由一个名为len的字段来保存。
2. **获取长度信息的时间复杂度为O(1)**：由于SDS使用len字段来保存字符串长度，因此获取字符串长度的操作只需要读取len字段，时间复杂度为O(1)。
3. **杜绝缓冲区溢出**：SDS在进行字符串修改操作时，会先检查缓冲区空间是否满足修改要求，如果不满足则会自动扩展缓冲区空间。
4. **减少内存重新分配次数**：SDS使用空间预分配和惰性空间释放两种策略来优化内存使用。当SDS需要扩展空间时，除了为保存数据的空间外，可能还会分配额外的未使用空间；当SDS需要缩短空间时，通常并不立即释放多余的空间，而是等待将来可能的再次使用。
5. **兼容部分C字符串函数**：虽然SDS在内部实现上与C字符串有所不同，但在API设计上，SDS仍然保留了对部分C字符串函数的兼容性。

SDS在Redis中被广泛用于各种场景，比如在Redis数据库里面，包含字符串值的键值对在底层都是由SDS实现的。此外，SDS还被用作缓冲区：AOF模块中的AOF缓冲区，以及客户端状态中的输入缓冲区，都是由SDS实现的。

![](https://github.com/vankykoo/image/blob/main/097.png?raw=true)



####3.总结

![](https://github.com/vankykoo/image/blob/main/098.png?raw=true)

![](https://github.com/vankykoo/image/blob/main/099.png?raw=true)

底层逻辑图

![](https://github.com/vankykoo/image/blob/main/100.png?raw=true)



### 4）hash数据类型源码分析

#### 1.介绍

redis6：ziplist + hashtable

> Redis6的Hash数据类型的底层实现可以使用两种数据结构：ziplist（压缩列表）和hashtable（哈希表）。
>
> 1. **ziplist（压缩列表）**：当Hash对象同时满足以下两个条件时，会使用ziplist编码：
>    - 所有键值对的键和值的字符串长度都小于64字节
>    - Hash对象保存的键值对数量小于512个 ziplist是一种特殊编码的双向链表，它不存储指向上一个链表节点和指向下一个链表节点的指针，而是存储上一个节点长度和当前节点长度。这种方式牺牲了部分读写性能，来换取高效的内存空间利用率。
> 2. **hashtable（哈希表）**：当Hash对象无法满足ziplist编码的条件时，会使用hashtable编码。hashtable是Redis中的字典，使用哈希表作为底层实现，一个哈希表里面可以有多个哈希表节点，而每个哈希表节点保存了字典中的一个键值对。Redis中的哈希采用了挂链解决冲突的方式，当有多个key-value键值对的键名key映射值相同时，系统会将这些键值value以单链表的形式保存。

redis7：listpack + hashtable



#### 2.Redis6

【 **ziplist + hashtable**】

**hash-max-ziplist-entries：使用压缩列表保存时哈希集合中的最大元素个数。初始值512**

**hash-max-ziplist-value：使用压缩列表保存时哈希集合中单个元素的最大长度。初始值64**

①哈希对象保存的键值对数量小于512个

②所有的键值对的键和值的字符串长度都小于等于64byte时用ziplist，反之用hashtable

![](https://github.com/vankykoo/image/blob/main/101.png?raw=true)



**底层：**

OBJ_ENCODING_HT 这种编码方式内部才是真正的哈希表结构，或称为字典结构，其可以实现O(1)复杂度的读写操作，因此效率很高。

在 Redis内部，从 OBJ_ENCODING_HT类型到底层真正的散列表数据结构是一层层嵌套下去的

![](https://github.com/vankykoo/image/blob/main/102.png?raw=true)



**ziplist：**

Redis的ziplist（压缩列表）是一种特殊的双向链表，由一系列特殊编码的连续内存块组成。它可以在任意一端进行压入/弹出操作，且该操作的时间复杂度为O(1)。ziplist的结构包括以下几个部分：

- **zlbytes**：记录整个ziplist在内存中占用的字节数。
- **zltail**：表示尾节点距离整个ziplist起始地址之间的字节数，通过这个属性可以快速定位到尾节点。
- **zllen**：记录整个ziplist中存放了多少个节点。
- **entry**：ziplist包含的各个节点，节点的长度由节点保存的内容决定。
- **zlend**：这是一个标志位，用来标志ziplist已经到达了末尾。

每个entry节点包含以下几个部分：

- **previous_entry_length**：记录上一个节点的长度。
- **encoding**：记录content的数据类型（同时通过它还可以知道当前节点的长度，以此可以找到下一个节点）。
- **content**：负责保存节点数据的，这个数据包括字符串或者整数。

总的来说，ziplist是一种内存紧凑、高效的数据结构，被广泛用于Redis中。

![](https://github.com/vankykoo/image/blob/main/103.png?raw=true)

链表在内存中，一般是不连续的，遍历相对比较慢，而ziplist可以很好的解决这个问题。如果知道了当前的起始地址，因为entry是连续的，entry后一定是另一个entry，想知道下一个entry的地址，只要将当前的起始地址加上当前entry总长度。如果还想遍历下一个entry，只要继续同样的操作。

![](https://github.com/vankykoo/image/blob/main/104.png?raw=true)



ziplist缺点：

1. **插入和删除操作的时间复杂度为O(n)**：虽然ziplist可以在两端进行O(1)时间复杂度的压入和弹出操作，但是在列表中间进行插入和删除操作的时间复杂度为O(n)。这是因为ziplist是通过顺序内存布局来实现的，所以在进行中间插入和删除操作时，需要对后续的元素进行移动，这会消耗更多的时间。
2. **列表长度受限**：由于ziplist使用连续的内存块来存储数据，所以当存储的数据量过大时，可能会导致内存分配失败。因此，ziplist不适合用来存储大量数据。
3. **随机访问性能较差**：虽然ziplist可以通过索引来访问元素，但是由于它是一个线性结构，所以随机访问一个元素的时间复杂度为O(n)。这意味着如果你需要频繁地访问列表中间的元素，那么使用ziplist可能会有性能问题。
4. **连锁更新**：压缩列表新增某个元素或修改某个元素时，如果空间不不够，压缩列表占用的内存空间就需要重新分配。而当新插入的元素较大时，可能会导致后续元素的 prevlen 占用空间都发生变化，从而引起「连锁更新」问题，导致每个元素的空间都要重新分配，造成访问压缩列表性能的下降。

总的来说，ziplist在处理小型数据和频繁进行两端操作时表现出色，但在处理大型数据或需要频繁进行中间插入、删除和随机访问操作时可能会遇到性能问题。



> 为什么不用链表，而要创造ziplist？
>
> Redis中的Hash类型在某些情况下会选择使用ziplist（压缩列表）作为底层实现，而不是链表，主要有以下几个原因：
>
> 1. **内存优化**：ziplist是一种紧凑的数据结构，它将所有元素存储在一个**连续的内存区域**中，这样可以节省内存空间，特别适合存储小型数据。
> 2. **提高缓存命中率**：由于ziplist将所有元素存储在连续的内存区域中，因此相比链表，ziplist可以更好地利用CPU缓存，**提高缓存命中率。**
> 3. **提高访问效率**：对于小型数据，ziplist可以通过一次内存访问就获取到所有元素，而链表则需要多次内存访问。因此，在处理小型数据时，ziplist的访问效率更高。
>
> 当然，ziplist也有其局限性，例如当需要频繁地进行中间插入和删除操作时，ziplist的效率就会低于链表。因此，在Redis中，是否使用ziplist还取决于Hash对象的具体使用情况。



#### 3.Redis7

**listpack + hashtable**



listpack介绍：

Listpack是Redis 5.0引入的一种新的数据结构，它是对ziplist（压缩列表）的改进版本，目的是解决ziplist中存在的连锁更新问题。Listpack在存储与结构上都比ziplist要更为节省与精简。

Listpack由以下几部分组成：

- **total_bytes**：占用的总字节数，占用4个字节，每个listpack最多占用4294967295Bytes。
- **num_elem**：listpack中的元素个数，即Entry的个数，占用2个字节。值得注意的是，这并不意味着listpack最多只能存放65535个Entry，当Entry个数大于等于65535时，num_elem被设置为65535，此时如果需要获取元素个数，需要遍历整个listpack。
- **entries**：listpack包含的各个节点。每个节点都包含了三部分内容：**编码、长度、数据**。
- **end**：listpack结束标志，占用1个字节，内容为0xFF。

Listpack中每个节点不再包含前一个节点的长度，而是包含当前节点的长度。这样做的好处是，在插入或删除节点后，不需要更新后续所有节点中保存的前置节点长度值，从而**避免了连锁更新现象**。

![](https://github.com/vankykoo/image/blob/main/105.png?raw=true)



![](https://github.com/vankykoo/image/blob/main/106.png?raw=true)





### 5）list底层源码分析

#### 1.redis6

list用quicklist来存储，quicklist存储了一个双向链表，每个节点都是一个ziplist

![](https://github.com/vankykoo/image/blob/main/107.png?raw=true)



#### 2.redis7

list用quicklist来存储，quicklist存储了一个双向链表，每个节点都是一个**listpack**



### 6）set底层结构源码分析

Redis用**intset或hashtable**存储set，如果元素都是整数类型，就用intset存储。

如果不是整数类型，就用hashtable（数组+链表的存储结构）



### 7）Zset底层结构源码分析

#### 1.redis6

ziplist + skiplist



#### 2.redis7

listpack + skiplist



#### 3.skiplist

skiplist是一种**以空间换取时间**的结构。

由于链表，无法进行二分查找，因此借鉴数据库索引的思想，提取出链表中关键节点（索引），先在关键节点上查找，再进入下层链表查找，提取多层关键节点，就形成了跳跃。

但是由于索引也要占据一定空间的，所以，索引添加的越多，空间占用的越多。

Skip list是一种概率型数据结构，它基于链表的一般概念构建。Skip list使用概率来在原始链表上构建后续的链表层。每个附加的链表层包含更少的元素，但没有新的元素。

**Skip list的主要特性包括：**

1. **高效的搜索和插入**：Skip list允许在有序元素序列中进行平均复杂度的搜索以及插入。这使得它可以获得排序数组（用于搜索）的最佳特性，同时保持允许插入的链表类似的结构。
2. **分层结构**：Skip list是分层构建的。底层是一个普通的有序链表。每个更高的层都充当下面列表的“快速通道”，其中每个元素都出现在下面的列表中，每个连续的子序列跳过比前一个更少的元素。
3. **随机化**：Skip list中被跳过的元素可以通过概率或确定性方式选择，前者更常见。

总体来说，Skip list是一种允许在有序元素序列中进行高效搜索和插入操作的数据结构。

![](https://github.com/vankykoo/image/blob/main/108.png?raw=true)

**Skip list的时间复杂度和空间复杂度如下：**

- **时间复杂度**：Skip list的搜索、插入和删除操作的平均时间复杂度都是O(log n)[1](https://www.geeksforgeeks.org/skip-list/)。这是因为Skip list的结构允许我们在搜索时跳过一些节点，从而减少了需要比较的节点数量[1](https://www.geeksforgeeks.org/skip-list/)。
- **空间复杂度**：Skip list的平均空间复杂度是O(n)。这是因为每个元素平均会出现在log(n)个链表中，所以总的节点数量是n * log(n)，但是由于每个节点可以共享，所以实际的空间复杂度是O(n)。

总的来说，Skip list在处理有序数据时，既能保证较高的操作效率，又能保证较低的空间占用。



**Skip list是一种高效的数据结构，它的优点和缺点如下：**

**优点**：

1. **高效的搜索和插入**：Skip list允许在有序元素序列中进行平均复杂度的搜索以及插入。这使得它可以获得排序数组（用于搜索）的最佳特性，同时保持允许插入的链表类似的结构。
2. **实现简单**：相比于平衡树等数据结构，Skip list的实现更为简单。

**缺点**：

1. **内存占用较大**：由于Skip list需要存储额外的指针信息，因此它的内存占用通常会比链表更大。
2. **最坏情况性能较差**：虽然Skip list在平均情况下具有良好的性能，但在最坏情况下（例如所有元素都在同一层），其性能可能会退化到线性。

总的来说，Skip list是一种在处理有序数据时既能保证较高的操作效率，又能保证较低的空间占用的数据结构。但是，它也存在一些局限性，例如在处理大型数据或需要频繁进行中间插入、删除和随机访问操作时可能会遇到性能问题。



## 十三、IO多路复用

### 1）介绍

Redis利用epoll来实现IO多路复用，将连接信息和事件放到队列中，一次放到文件事件分派器，时间分派器将事件分发给事件处理器。

![](https://github.com/vankykoo/image/blob/main/109.png?raw=true)

Redis 是跑在单线程中的，所有的操作都是按照顺序线性执行的，但是**由于读写操作等待用户输入或输出都是阻塞的**，所以 I/O 操作在一般情况下往往不能直接返回，这会导致某一文件的 I/O 阻塞导致整个进程无法对其它客户提供服务，而 I/O 多路复用就是为了解决这个问题而出现所谓 I/O 多路复用机制，就是说通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或写就绪），能够通知程序进行相应的读写操作。这种机制的使用需要 select 、 poll 、 epoll 来配合。**多个连接共用一个阻塞对象，应用程序只需要在一个阻塞对象上等待，无需阻塞等待所有连接。当某条连接有新的数据可以处理时，操作系统通知应用程序，线程从阻塞状态返回，开始进行业务处理。**

 Redis 服务采用 Reactor 的方式来实现文件事件处理器（每一个网络连接其实都对应一个文件描述符） 

Redis基于Reactor模式开发了网络事件处理器，这个处理器被称为文件事件处理器。它的组成结构为4部分：**多个套接字、IO多路复用程序、文件事件分派器、事件处理器。**

因为**文件事件分派器队列的消费是单线程的**，所以Redis才叫单线程模型。

![](https://github.com/vankykoo/image/blob/main/110.png?raw=true)



###2）同步异步、阻塞非阻塞

同步和异步，阻塞和非阻塞，这四个概念是在描述程序在执行过程中对任务的处理方式和状态。

- **同步**：在发出一个调用时，在没有得到结果之前，该调用就不返回或继续执行后续操作。简单来说，同步就是必须一件一件事做，等前一件做完了才能做下一件事。
- **异步**：当一个异步过程调用发出后，调用者在没有得到结果之前，就可以继续执行后续操作。当这个调用完成后，一般通过状态、通知和回调来通知调用者。对于异步调用，调用的返回并不受调用者控制。
- **阻塞**：阻塞和非阻塞这两个概念与程序（线程）等待消息通知时的状态有关。阻塞调用是指调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。
- **非阻塞**：非阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。在等待的过程中可以做其它事情。

总的来说，同步/异步关注的是消息通知的机制，而阻塞/非阻塞关注的是程序在等待消息通知时的状态。



### 3）阻塞IO（BIO）

阻塞IO是一种输入/输出处理方式，当应用程序发起IO请求后，需要等待或者处理完才能进行下一步操作。如果数据没有准备好，那么应用程序就会停止执行，直到数据准备好并且已经将数据从系统内核复制到应用程序，这个过程中整个程序是被阻塞的。也就是说，**在阻塞IO中，IO操作的完成需要应用程序持续等待，直到操作完成才返回。**

这种模式的一个主要问题是效率低下。因为在等待数据的过程中，CPU什么也做不了，只能空闲等待。当处理大量数据时，CPU大部分时间都在等待数据，从而导致资源利用率低。

但是，在某些情况下，阻塞IO是可以接受的。例如，在客户端-服务器模型中，客户端通常在读取服务器响应之前都是处于空闲状态的，这时候使用阻塞IO并不会造成太大问题。然而，在服务器端，尤其是需要同时处理多个连接请求的情况下，使用阻塞IO就可能导致性能问题，因为服务器需要同时处理多个客户端请求，如果一个请求阻塞了，那么其他请求也会被迫等待。



### 4）非阻塞IO（NIO）

* 在NIO模式中，一切都是非阻塞的：
  * accept()方法是非阻塞的，如果没有客户端连接，就返回无连接标识。
  * read()方法是非阻塞的，如果read()方法读取不到数据就返回空闲中标识，如果读取到数据时只阻塞read()方法读数据的时间
* 在NIO模式中，只有一个线程：
  * 当一个客户端与服务端进行连接，这个socket就会加入到一个数组中，隔一段时间遍历一次，
  * 看这个socket的read()方法能否读到数据，这样一个线程就能处理多个客户端的连接和读取了



**非阻塞IO**是一种IO处理方式，当应用程序发起IO请求后，不需要等待或者处理完就可以进行下一步操作。如果数据没有准备好，那么应用程序就会继续执行，直到数据准备好并且已经将数据从系统内核复制到应用程序，这个过程中整个程序是非阻塞的。也就是说，**在非阻塞IO中，IO操作的完成需要应用程序持续检查，直到操作完成才返回。**

这种模式的一个主要优点是效率高。因为在等待数据的过程中，CPU可以做其它的事情，从而提高了资源利用率。但是，这也意味着**应用程序需要不断地检查IO操作是否完成，这可能会导致CPU使用率过高。**

在实际应用中，非阻塞IO常常和IO多路复用技术结合使用。例如，在服务器端处理多个客户端请求时，可以使用非阻塞IO和select或epoll等IO多路复用技术，使得服务器在等待某个IO操作完成的同时，还可以处理其他的IO请求。



优点：不会阻塞在内核的等待数据过程，每次发起的 I/O 请求可以立即返回，不用阻塞等待，实时性较好。

缺点：轮询将会不断地询问内核，这将占用大量的 CPU 时间，系统资源利用率较低，所以一般 Web 服务器不使用这种 I/O 模型。

结论：让Linux内核搞定上述需求，我们将一批文件描述符通过一次系统调用传给内核由内核层去遍历，才能真正解决这个问题。**IO多路复用应运而生，也即将上述工作直接放进Linux内核，不再两态转换而是直接从内核获得结果，因为内核是非阻塞的。**



### 5）IO多路复用三个方法

`select`、`poll`和`epoll`都是IO多路复用的技术。IO多路复用就是通过一种机制，一个进程可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。但这三种技术在实现上有所不同：

- **select**：它在发起一个IO操作时，会阻塞当前进程，直到有描述符就绪（有数据可读或者可写），或者超时，才返回。一旦返回，进程就可以开始进行IO操作了。然而，select有一个缺点，那就是单个进程能够监视的文件描述符的数量存在最大限制，在Linux上一般为1024。
- **poll**：poll和select非常类似，但是它没有最大文件描述符数量的限制。poll同样提供了非阻塞的IO，并且也能告知哪些描述符已经就绪。
- **epoll**：相比之下，epoll则是Linux特有的IO多路复用技术，它没有描述符数量限制，并且在接收到大量并发连接请求时，比select和poll更加高效。因为在内核中完成了更多的事情，减少了用户空间和内核空间之间的切换。

总的来说，这三种技术各有优劣，具体使用哪一种，取决于你的应用需求和运行环境。



## 微信抢红包

```java
@RestController
public class RedPackageController
{
    public static final String RED_PACKAGE_KEY = "redpackage:";
    public static final String RED_PACKAGE_CONSUME_KEY = "redpackage:consume:";


    @Resource
    private RedisTemplate redisTemplate;

    /**
     * 拆分+发送红包
     * http://localhost:5555/send?totalMoney=100&redPackageNumber=5
     * @param totalMoney
     * @param redPackageNumber
     * @return
     */
    @RequestMapping("/send")
    public String sendRedPackage(int totalMoney,int redPackageNumber)
    {
        //1 拆红包，总金额拆分成多少个红包，每个小红包里面包多少钱
        Integer[] splitRedPackages = splitRedPackage(totalMoney, redPackageNumber);
        //2 红包的全局ID
        String key = RED_PACKAGE_KEY+IdUtil.simpleUUID();
        //3 采用list存储红包并设置过期时间
        redisTemplate.opsForList().leftPushAll(key,splitRedPackages);
        redisTemplate.expire(key,1,TimeUnit.DAYS);
        return key+"\t"+"\t"+ Ints.asList(Arrays.stream(splitRedPackages).mapToInt(Integer::valueOf).toArray());
    }

    /**
     * http://localhost:5555/rob?redPackageKey=上一步的红包UUID&userId=1
     * @param redPackageKey
     * @param userId
     * @return
     */
    @RequestMapping("/rob")
    public String rodRedPackage(String redPackageKey,String userId)
    {
        //1 验证某个用户是否抢过红包
        Object redPackage = redisTemplate.opsForHash().get(RED_PACKAGE_CONSUME_KEY + redPackageKey, userId);
        //2 没有抢过就开抢，否则返回-2表示抢过
        if (redPackage == null) {
            // 2.1 从list里面出队一个红包，抢到了一个
            Object partRedPackage = redisTemplate.opsForList().leftPop(RED_PACKAGE_KEY + redPackageKey);
            if (partRedPackage != null) {
                //2.2 抢到手后，记录进去hash表示谁抢到了多少钱的某一个红包
                redisTemplate.opsForHash().put(RED_PACKAGE_CONSUME_KEY + redPackageKey,userId,partRedPackage);
                System.out.println("用户: "+userId+"\t 抢到多少钱红包: "+partRedPackage);
                //TODO 后续异步进mysql或者RabbitMQ进一步处理
                return String.valueOf(partRedPackage);
            }
            //抢完
            return "errorCode:-1,红包抢完了";
        }
        //3 某个用户抢过了，不可以作弊重新抢
        return "errorCode:-2,   message: "+"\t"+userId+" 用户你已经抢过红包了";
    }

    /**
     * 1 拆完红包总金额+每个小红包金额别太离谱
     * @param totalMoney
     * @param redPackageNumber
     * @return
     */
    private Integer[] splitRedPackage(int totalMoney, int redPackageNumber)
    {
        int useMoney = 0;
        Integer[] redPackageNumbers = new Integer[redPackageNumber];
        Random random = new Random();

        for (int i = 0; i < redPackageNumber; i++)
        {
            if(i == redPackageNumber - 1)
            {
                redPackageNumbers[i] = totalMoney - useMoney;
            }else{
                int avgMoney = (totalMoney - useMoney) * 2 / (redPackageNumber - i);
                redPackageNumbers[i] = 1 + random.nextInt(avgMoney - 1);
            }
            useMoney = useMoney + redPackageNumbers[i];
        }
        return redPackageNumbers;
    }
}
```
