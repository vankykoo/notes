# seata踩坑：Seata启动报nacos 错误 ，错误码 403.

因为seata使用nacos作为配置中心和注册中心，nacos需要修改配置文件：

```properties
nacos.core.auth.enabled=true

nacos.core.auth.server.identity.key=nacos
nacos.core.auth.server.identity.value=nacos

# 大于32位即可，最后转化成Base64格式
nacos.core.auth.plugin.nacos.token.secret.key=VGhpc0lzTmFjb3NTZWNyZXRLZXktdmFua3lUaGlzSXNOYWNvc1NlY3JldEtleS12YW5reQo=
```



所以seata服务注册进入 nacos 时需要填写认证信息。



### 解决办法：

seata 与 nacos 有关配置中都有写 `username` 和 `password`

例如seata的配置文件`application.yml`中

```yaml
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
```









