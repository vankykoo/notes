#Seata2.0 + Nacos注册中心 + Nacos配置中心 + MySQL

## 零、Mysql准备

1. 在mysql中创建数据库，导入 \script\server\db 下 `mysql.sql` 的四张表。
2. 在微服务的每个数据库中加上一张 undo_log表

```sql
CREATE TABLE IF NOT EXISTS `undo_log`
(
    `branch_id`     BIGINT       NOT NULL COMMENT 'branch transaction id',
    `xid`           VARCHAR(128) NOT NULL COMMENT 'global transaction id',
    `context`       VARCHAR(128) NOT NULL COMMENT 'undo_log context,such as serialization',
    `rollback_info` LONGBLOB     NOT NULL COMMENT 'rollback info',
    `log_status`    INT(11)      NOT NULL COMMENT '0:normal status,1:defense status',
    `log_created`   DATETIME(6)  NOT NULL COMMENT 'create datetime',
    `log_modified`  DATETIME(6)  NOT NULL COMMENT 'modify datetime',
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
    ) ENGINE = InnoDB AUTO_INCREMENT = 1 DEFAULT CHARSET = utf8mb4 COMMENT ='AT transaction mode undo table';
ALTER TABLE `undo_log` ADD INDEX `ix_log_created` (`log_created`);
```



##一、nacos配置

###1、application.properties

使用nacos作为seata的注册中心和配置中心时，需要对nacos先做一些配置。

**nacos官方文档**：在2.2.1版本后，社区发布版本将移除以文档如下值作为默认值，需要自行填充，否则无法启动节点。

```properties
nacos.core.auth.enabled=true

nacos.core.auth.server.identity.key=nacos
nacos.core.auth.server.identity.value=nacos

# 大于32位即可，最后转化成Base64格式
nacos.core.auth.plugin.nacos.token.secret.key=VGhpc0lzTmFjb3NTZWNyZXRLZXktdmFua3lUaGlzSXNOYWNvc1NlY3JldEtleS12YW5reQo=
```



> `nacos.core.auth.server.identity.key` 和 `nacos.core.auth.server.identity.value` 是 Nacos 服务端的配置属性，用于标识服务器的身份验证信息。这两个参数在Nacos的安全认证机制中起到自定义身份识别的作用。
>
> 具体来说：
>
> - `nacos.core.auth.server.identity.key`：这个属性用来指定请求头中作为服务器身份验证标识的键名。当Nacos集群中的节点之间相互进行通信时（比如服务发现、配置同步等内部通信），通过检查请求头中此键所对应的值来确定请求是否来自于可信的Nacos服务器。
>
> - `nacos.core.auth.server.identity.value`：这个属性是与上述键相对应的值，它是服务器的身份标识字符串，集群内的其他节点在收到请求时会验证请求头中携带的这个键的值是否匹配预设的服务器身份标识。
>
> 通过设置这些属性，可以确保只有携带了正确身份标识的服务器之间的请求才能被接受和处理，从而增强了Nacos集群内部通信的安全性。特别是在开启了权限认证功能的情况下，这种自定义的身份识别机制是非常重要的，它能防止未经授权的服务冒充合法服务器进行操作。

> `nacos.core.auth.plugin.nacos.token.secret.key` 是 Nacos 服务端的另一个重要安全配置属性，用于实现基于令牌（Token）的身份验证机制。具体作用如下：
>
> 在Nacos中，为了加强服务的安全性，可以启用基于Token的认证插件来对客户端请求进行身份验证。当该功能开启后，客户端在访问Nacos时需要携带经过特定算法签名生成的Token。



###2、seata的配置导入nacos中

1. 在 \script\config-center 下的 `config.txt` 中，修改如下几项

```properties
store.mode=db
store.lock.mode=db
store.session.mode=db

store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.cj.jdbc.Driver
store.db.url=jdbc:mysql://localhost:3306/seata?useUnicode=true&rewriteBatchedStatements=true
store.db.user=root
store.db.password=vanky
```



2. 在 \script\config-center\nacos 目录下打开bash，输入以下命令导入nacos

```bash
sh nacos-config.sh -h 127.0.0.1 -p 8848 -g SEATA_GROUP -t nacos命令空间id -u nacos -w nacos
```



## 二、seata配置

### 1、application.yml

```yaml
console:
  user:
    username: seata
    password: seata
    
seata:
  config:
    # support: nacos, consul, apollo, zk, etcd3
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      namespace: public
      group: SEATA_GROUP
      username: nacos
      password: nacos
      context-path:
      data-id: seataServer.properties
  registry:
    # support: nacos, eureka, redis, zk, consul, etcd3, sofa
    type: nacos
    preferred-networks: 30.240.*
    nacos:
      application: seata-server
      server-addr: 127.0.0.1:8848
      group: SEATA_GROUP
      namespace: public
      cluster: default
      username: nacos
      password: nacos
      
  store:
    # support: file 、 db 、 redis 、 raft
    mode: db
    session:
      mode: db
    lock:
      mode: db
    db:
    # 需要和 config.txt 保持一致
      datasource: druid
      db-type: mysql
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/seata?rewriteBatchedStatements=true
      user: root
      password: vanky
      min-conn: 10
      max-conn: 100
      global-table: global_table
      branch-table: branch_table
      lock-table: lock_table
      distributed-lock-table: distributed_lock
      query-limit: 1000
      max-wait: 5000
      
  service:
    vgroupMapping:
      default_tx_group: seata_group
```

其他的按照 application.example.yml 



## 三、微服务 配置

### 1、依赖

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>

<dependency>
  <groupId>io.seata</groupId>
  <artifactId>seata-spring-boot-starter</artifactId>
</dependency>
```



### 2、application.yml

```yaml
seata:
  registry:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      namespace: public
      group: SEATA_GROUP
      application: seata-server
      username: nacos
      password: nacos
  tx-service-group: seata-group
  service:
    vgroup-mapping:
      seata-group: default
  enabled: true
```



###3、注解

@GlobalTransactional

































































































































