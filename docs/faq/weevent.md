## WeEvent

关于`WeEvent`使用的一些常见问题和答案都会整理归档到这里。

简单的接口调用异常一般从错误码中就可以得到答案[WeEvent错误码](../protocol/errorcode.html) 。有任何意见和建议，欢迎你参与`WeEvent`项目讨论[Issues](https://github.com/WeBankBlockchain/WeEvent/issues) 。

- 怎么部署`WeEvent`和`FISCO-BCOS`？

  建议将`WeEvent`和`FISCO-BCOS`节点部署在同一网段/逻辑区，比如`DMZ`区。

- `WeEvent`的服务访问授权机制是怎么样的？

  `WeEvent`访问权限控制基于`HTTPS` + `IP`白名单，`STOMP`和`MQTT`协议还支持协议定义的账号/密码机制。

- 如何选择各种接入协议，`RESTful`、`JsonRPC`、`STOMP`还是`MQTT`？
  
  - 协议之间的区别
    
    `STOMP`协议是`WeEvent`推荐协议，关于订阅发布的所有特性，以及未来的扩展功能都会支持。
    
    `JsonRPC`协议是对`STOMP`协议标准功能集的一个补充。
    
    `RESTful`主要是给`Web`开发使用，内置的`Governance`模块就有大量使用。
    
    `MQTT`主要面向物联网`IoT`设备的接入。
    
  - Java程序
    
    - 如果是`Spring`服务，有内置的`org.springframework.boot:spring-boot-starter-websocket`支持，推荐使用`STOMP`。
    - 如果是其他`Java`程序，建议使用`WeEvent`提供的`Java SDK`。
    
  - 其他语言
    - 生产者`Producer`建议使用`RESTful`/`JsonRPC` ，简单方便。
    - 消费者`Consumer`因为涉及到事件的持续`Push`，建议使用`STOMP`协议接入。
  
- `WeEvent`服务的高可用性方案是怎么样的？

  `WeEvent`除了订阅之外的功能，都是无状态的请求，通过`Cluster`集群的多实例负载均衡来保证高可用性。订阅功能按是否为长连接分成两类：

    - 使用短链接的协议，比如`JsonRPC`/`RESTful`
        短链接协议都是无状态请求。通过负载均衡路由到任意的服务实例上执行即可。

    - 使用长连接的协议，比如`STOMP`/`MQTT`
        这部分功能基于长连接，订阅上下文以连接为中心，连接关闭则订阅会自动关闭。
        
- `WeEvent`怎么处理发布事件的去重？

  `WeEvent`不处理发布事件的去重。

  当服务超时或者机器宕机时，一般的逻辑是重试，这种策略很容易出现重复请求。但是整个业务系统内，不只在`WeEevnt`服务的边界会出现这种情况，任何一个服务边界都会出现。 建议业务统一处理，比如在事件内容里带一个唯一ID`UUID`，消费的时候使用`UUID`字段来判断该事件是否已经处理过。
```eval_rst
.. important::
    - EventID不是用来去重的，同一事件每成功发布一次，都会生成不同的EventID。
```
- 服务状态检查脚本`check-service.sh`出现`"deploy contract failed"`
  - 检查`WeEvent`到`FISCO-BCOS`的连接及其配置。
  - 检查`FISCO-BCOS`节点是否正常出块。

- `WeEvent`事件内容支持的最大长度是多少？

  事件内容最大长度为10 K字节 `Byte`， 字符集编码为`UTF8`。

- `WeEvent`到区块链`FISCO-BCOS`节点之间的网络故障怎么处理？

  建议为`WeEvent`配置2个属于不同网段的`FISCO-BCOS`节点。这样`WeEvent`服务会随机选用一个可用的节点使用，做到一定程度的网络容灾。

- 怎么使用通配符进行订阅
  `WeEvent`支持通配符按层次订阅，和`MQTT`的定义完全一致。包括层次分隔符"/"，单层通配符"+"，多层通配符"#"。例如，“#”可以订阅到系统内所有的主题。“com/+”可以订阅到主题“com/weevent”和“com/test“，但是无法订阅到“com/weevent/abc”。“com/#”则可以订阅到上述三个主题。详情请参见[MQTT协议](http://public.dhe.ibm.com/software/dw/webservices/ws-mqtt/mqtt-v3r1.html)。
  
- 服务启动时为什么会出现Failed to initialize the client-side SSLContext错误？
  
  ```
  javax.net.ssl.SSLException: Failed to initialize the client-side SSLContext: Input stream not contain valid certificates
  ```
  
  这个问题涉及到`JDK`加密算法的实现。`Oracle JDK`里带了这个算法实现，`Open JDK`直到 1.9版本才有。所以在`CentOS`系统中，如果使用 `Open JDK 1.9`以下版本，`WeEvent`启动时会出现以下异常。请升级`Open JDK`版本到1.9或者使用`Oracle JDK`。
  
    

      
- 如何切换数据库
   
    目前WeEvent默认的是H2数据库，`application-prod.properties`中默认数据库配置如下
   
    ```
    spring.datasource.url=jdbc:h2:./WeEvent_governance
    spring.datasource.driver-class-name=org.h2.Driver
    spring.datasource.username=root
    spring.datasource.password=123456
    ```  
  
    如果要切换成Mysql数据库，改成下面这样，其中`url、username、password` 修改成需要连接的数据库配置。
    
     ```
    spring.datasource.url=jdbc:mysql://127.0.0.1:3306/WeEvent_governance?useUnicode=true&characterEncoding=utf-8&useSSL=false
    spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
    spring.datasource.username=root
    spring.datasource.password=123456
     ```
```eval_rst
.. important::  
    - Mysql数据库要赋予配置账号创建库表的权限。
```
    ```
    >> grant all privileges on *.* to 'root'@'%' identified by '123456';
    >> flush privileges;
    ```
