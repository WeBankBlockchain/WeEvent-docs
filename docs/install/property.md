## 配置说明
### Broker 配置

`Broker`服务主要有三类配置，`Spring Boot `进程配置、区块链`FISCO-BCOS`节点配置、`WeEvent`服务配置。

- Spring Boot进程配置

  配置文件`./broker/conf/application-prod.properties`，这个是`Spring Boot`标准配置文件，一般不需要修改。细节请参见[Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#appendix) 。

  | 配置项                           | 默认值                                      | 配置说明        |
  | ----------------------------- | ---------------------------------------- | ----------- |
  | server.port                   | 7000                                     | spring监听端口  |
  | server.servlet.context-path   | /weevent                                 | spring上下文路径 |
  | spring.autoconfigure.exclude  | org.springframework.boot.autoconfigure.<br />security.servlet.SecurityAutoConfiguration | 默认不开启       |
  | spring.security.user.name     | user                                     | 默认不开启       |
  | spring.security.user.password | 123456                                   | 默认不开启       |

  用户访问授权通过用户密码`spring.security.user`配置，对所有的接入协议`RESTFul`、`Json RPC`、`STOMP`、`MQTT`都会生效。

  配置项生效后，`RESTFul`和`Json RPC`需要是使用`http base authorization`访问。`STOMP`访问需要设置`header`项`login`和`passcode`，`MQTT`访问需要设置`username`和`password`。

- 区块链FISCO-BCOS节点配置

  配置文件`./broker/conf/fisco.properties`。

  | 配置项                       | 默认值                | 配置说明                                |
  | ---------------------------- | --------------------- | --------------------------------------- |
  | version                      | 2.0                   | FISCO-BCOS版本，支持2.0和1.3            |
  | orgid                        | fisco                 | 机构名，按机构实际名称填写即可          |
  | nodes                        | 127.0.0.1:30701       | 区块链节点列表，多个地址以`;`分割       |
  | proxy.address                | 0xfff77de6c1a76022... | 1.3版本的proxy系统合约地址              |
  | account                      | bcec428d5205abe0f...  | `WeEvent`执行交易的账号，一般不需要修改 |
  | web3sdk.timeout              | 10000                 | 交易执行超时时间，单位毫秒              |
  | web3sdk.core-pool-size       | 10                    | web3sdk最小线程数                       |
  | web3sdk.max-pool-size        | 1000                  | web3sdk最大线程数                       |
  | web3sdk.keep-alive-seconds   | 10                    | web3sdk线程空闲时间，单位秒             |
  | consumer.idle-time           | 1000                  | 区块链新增块事件检测周期，单位毫秒      |
  | consumer.history_merge_block | 8                     | 事件过滤的区块范围                      |


  区块链节点详细配置，参见[Web3SDK配置文件](https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-2.0/docs/sdk/sdk.html) 。

- WeEvent服务配置

  配置文件`./broker/conf/weevent.properties` 。

  | 配置项                                | 默认值           | 配置说明                                     |
  | ---------------------------------- | ------------- | ---------------------------------------- |
  | ip.check.white-table               |               | IP白名单。多个`IP`地址，以";"分割。<br />默认为空时表示允许任何客户端访问。 |
  | lru.cache.capacity                 | 65536         | 缓存大小，使用LRU策略淘汰                           |
  | restful.subscribe.callback.timeout | 5000          | 事件通知回调的超时时间，单位毫秒                         |
  | broker.zookeeper.ip                |               | zookeeper服务                              |
  | broker.zookeeper.path              | /event_broker | zookeeper数据路径                            |
  | broker.zookeeper.timeout           | 3000          | zookeeper链接超时时间，单位毫秒                     |
  | stomp.heartbeats                   | 30            | stomp心跳间隔，单位秒                            |
  | mqtt.broker.port                   | 7001          | mqtt协议TCP访问端口，默认不开启                      |
  | mqtt.broker.keepalive              | 60            | mqtt连接空闲时间，单位秒                           |
  | mqtt.websocket.path                | /weevent/mqtt | mqtt连接目录                                 |
  | mqtt.websocket.port                | 7002          | mqtt协议web socket访问端口，默认不开启               |

### Governance

配置文件`./governance/conf/application-prod.properties `。

| 配置项                                      | 默认值                                      | 配置说明               |
| ---------------------------------------- | ---------------------------------------- | ------------------ |
| server.port                              | 7009                                     | spring监听端口         |
| spring.datasource.url                    | jdbc:mysql://127.0.0.1:3306/governance?useUnicode=true&characterEncoding=utf-8&useSSL=false | 数据源                |
| spring.datasource.driver-class-name      | org.mariadb.jdbc.Driver                  | 驱动类                |
| spring.datasource.username               | xxxx                                     | 数据库账号用户名           |
| spring.datasource.password               | yyyy                                     | 数据库账号密码            |
| spring.datasource.type                   | org.apache.commons.dbcp2.BasicDataSource | 数据源类型              |
| spring.datasource.dbcp2.max-wait-millis  | 10000                                    | 数据库连接池最长等待时间ms     |
| spring.datasource.dbcp2.min-idle         | 5                                        | 数据库连接池最小空闲         |
| spring.datasource.dbcp2.initial-size     | 5                                        | 数据库连接池初始大小         |
| spring.datasource.dbcp2.validation-query | SELECT 'x'                               | 数据库连接池验证查询         |
| spring.mail.default-encoding             | UTF-8                                    | 编码类型               |
| spring.mail.host                         | smtp.163.com                             | 邮件服务器主机            |
| spring.mail.username                     | mailusername@163.com                     | 邮箱用户名              |
| spring.mail.password                     | mailpwd                                  | 邮箱密码               |
| http.client.max-total                    | 200                                      | http最大请求数          |
| http.client.max-per-route                | 500                                      | http最大请求路由数        |
| http.client.connection-request-timeout   | 3000                                     | http请求读取超时时间ms     |
| http.client.connection-timeout           | 3000                                     | http请求连接超时时间ms     |
| http.client.socket-timeout               | 5000                                     | http请求socket超时时间ms |
| https.client.read-timeout                | 3000                                     | 服务器内部读取其它服务的超时时间ms |
| https.client.connect-timeout             | 3000                                     | 服务器内部连接其它服务的超时时间ms |
| weevent.processor.url                    | http://127.0.0.1:7008                    | 配置processor服务的地址   |
| spring.redis.database                    | 0                                        | redis数据库索引     |
| spring.redis.host                        | 127.0.0.1                                | redis服务访问链接的ip |
| spring.redis.port                        | 6379                                     | redis服务访问端口   |
| spring.redis.password                    |                                          | redis服务访问密码 |
| spring.redis.timeout                     | 5000                                     | redis服务连接超时时间ms   |

### Processor 配置

- 配置文件`./processor/conf/application-prod.properties ` ，`server.port`默认为7008。
- 配置文件`./processor/conf/processor.properties `。

| 配置项                                                 | 默认值                                        | 说明                     |
| ------------------------------------------------------ | --------------------------------------------- | ------------------------ |
| quartz.schedule.name                                   | schedule                                      | quartz标识名称           |
| org.quartz.scheduler.instanceName                      | test                                          | scheduler名称            |
| org.quartz.dataSource.WeEvent_processor.URL            | jdbc:mysql://127.0.0.1:3306/WeEvent_processor | 连接quartz数据库的JDBC串 |
| org.quartz.dataSource.WeEvent_processor.user           | root                                          | 数据库用户名             |
| org.quartz.dataSource.WeEvent_processor.password       | 123456                                        | 数据库密码               |
| org.quartz.dataSource.WeEvent_processor.maxConnections | 30                                            | 最大连接数               |
| org.quartz.dataSource.WeEvent_processor.driver         | org.mariadb.jdbc.Driver                       | 驱动                     |
| org.quartz.jobStore.dataSource                         | WeEvent_processor                             | 数据库名称               |
| org.quartz.threadPool.threadCount                      | 20                                            | 进程数据                 |
| org.quartz.threadPool.threadPriority                   | 5                                             | 进行优先级               |

说明：`org.quartz.dataSource.*`  配置的数据库信息需要和`org.quartz.jobStore.dataSource` 配置的数据源信息一致。

### Nginx 配置说明

- 修改后端子模块路由

  通过配置文件`nginx/conf/conf.d/http_rs.conf`里的`upstream`来增加和移除后端的业务机器。

  ```nginx
  upstream broker_backend{
      server localhost:7000 weight=100 max_fails=3;
      
      ip_hash;
      keepalive 1024;
  }

  upstream broker_mqtt_websocket_backend {
      server localhost:7002 weight=100 max_fails=3;

      ip_hash;
      keepalive 1024;
  }

  upstream governance_backend{
      server localhost:7009 weight=100 max_fails=3;
      
      ip_hash;
      keepalive 1024;
  }
  ```

  特别的，如果使用了基于`TCP`的`MQTT`协议。配置文件为`nginx/conf/conf.d/tcp_rs.conf`。

- 使用TLS加密传输

  `WeEvent`通过`Nginx`实现`TLS`加密传输。如下`./nginx/conf/nginx.conf`的默认配置不支持`TLS`。

  ```nginx
  worker_processes  10;
  pid         logs/nginx.pid;

  events {
  	  use epoll;
      worker_connections  10000;
  }
  worker_rlimit_nofile 10000;

  #support web/restful/jsonrpc/stomp
  http {
      include       mime.types;
      default_type  application/octet-stream;
      keepalive_timeout  65;

      #custom config
      server_tokens           off;
      client_body_temp_path   ./nginx_temp/client_body;
      proxy_temp_path         ./nginx_temp/proxy;
      fastcgi_temp_path       ./nginx_temp/fastcgi;
      uwsgi_temp_path         ./nginx_temp/uwsgi;
      scgi_temp_path          ./nginx_temp/scgi;

      #http conf
      include                 ./conf.d/http_rs.conf;

      #include ./conf.d/https.conf
      include                 ./conf.d/http.conf;
  }

  # support mqtt over tcp
  stream {
      include                 ./conf.d/tcp_rs.conf;
    
      #include ./conf.d/tcp_tls.conf
      include                 ./conf.d/tcp.conf;
  }
  ```

  通过对应替换`include ./conf.d/https.conf`和`include ./conf.d/tcp_tls.conf`来支持`TLS`。


更多`Nginx`配置文件说明，请参见[Nginx配置](https://www.nginx.com/resources/wiki/start/topics/examples/full/) 。
