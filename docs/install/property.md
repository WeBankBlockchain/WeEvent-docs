## 配置说明
### Broker 配置

`Broker`服务主要有三类配置，`Spring Boot `进程配置、区块链`FISCO-BCOS`节点配置、`WeEvent`服务配置。

- Spring Boot进程配置

  配置文件`./broker/conf/application-prod.properties`，这个是`Spring Boot`标准配置文件，一般不需要修改。细节请参见[Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#appendix) 。

  | 配置项                                | 默认值                                                       | 配置说明         |
  | ------------------------------------- | ------------------------------------------------------------ | ---------------- |
  | server.port                           | 7000                                                         | spring监听端口   |
  | server.servlet.context-path           | /weevent                                                     | spring上下文路径 |
  | spring.autoconfigure.exclude          | org.springframework.boot.autoconfigure.<br />security.servlet.SecurityAutoConfiguration | 默认不开启       |
  | spring.security.user.name             | user                                                         | 默认不开启       |
  | spring.security.user.password         | 123456                                                       | 默认不开启       |
  | spring.cloud.zookeeper.enabled        | true                                                         | ZK访问           |
  | spring.cloud.zookeeper.connect-string | 127.0.0.1:2181                                               | ZK节点           |

  用户访问授权通过用户密码`spring.security.user`配置，对所有的接入协议`RESTFul`、`Json RPC`、`STOMP`、`MQTT`都会生效。配置项生效后，`RESTFul`和`Json RPC`需要是使用`http base authorization`访问。`STOMP`访问需要设置`header`项`login`和`passcode`，`MQTT`访问需要设置`username`和`password`。

- 区块链FISCO-BCOS节点配置

  配置文件`./broker/conf/fisco.properties`。

  | 配置项                       | 默认值               | 配置说明                                |
  | ---------------------------- | -------------------- | --------------------------------------- |
  | version                      | 2.0                  | FISCO-BCOS版本，支持2.x                 |
  | orgid                        | fisco                | 机构名，按机构实际名称填写即可          |
  | nodes                        | 127.0.0.1:20200      | 区块链节点列表，多个地址以`;`分割       |
  | account                      | bcec428d5205abe0f... | `WeEvent`执行交易的账号，一般不需要修改 |
  | web3sdk.timeout              | 10000                | 交易执行超时时间，单位毫秒              |
  | web3sdk.core-pool-size       | 10                   | web3sdk最小线程数                       |
  | web3sdk.max-pool-size        | 1000                 | web3sdk最大线程数                       |
  | web3sdk.keep-alive-seconds   | 10                   | web3sdk线程空闲时间，单位秒             |
  | consumer.idle-time           | 1000                 | 区块链新增块事件检测周期，单位毫秒      |
  | consumer.history_merge_block | 8                    | 事件过滤的区块范围                      |
  
  
  区块链节点详细配置，参见[Web3SDK配置文件](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/) 。
  
- WeEvent服务配置

  配置文件`./broker/conf/weevent.properties`。

  | 配置项                | 默认值               | 配置说明                                                     |
  | --------------------- | -------------------- | ------------------------------------------------------------ |
  | ip.check.white-table  |                      | IP白名单。多个`IP`地址，以";"分割。<br />默认为空时表示允许任何客户端访问。 |
  | stomp.heartbeats      | 30                   | stomp心跳间隔，单位秒                                        |
  | mqtt.broker.port      | 7001                 | 使用websocket访问MQTT的端口，默认不开启                      |
  | mqtt.broker.tcp.port  | 7002                 | 使用tcp访问MQTT的端口，默认不开启                            |
  | mqtt.broker.keepalive | 60                   | mqtt连接空闲时间，单位秒                                     |
  | mqtt.websocket.path   | /weevent-broker/mqtt | mqtt连接目录                                                 |

### Governance

配置文件`./governance/conf/application-prod.properties`。

| 配置项                                   | 默认值                                                       | 配置说明                   |
| ---------------------------------------- | ------------------------------------------------------------ | -------------------------- |
| server.port                              | 7009                                                         | spring监听端口             |
| spring.datasource.url                    | jdbc:mysql://127.0.0.1:3306/governance?useUnicode=true&characterEncoding=utf-8&useSSL=false | 数据源                     |
| spring.datasource.driver-class-name      | org.mariadb.jdbc.Driver                                      | 驱动类                     |
| spring.datasource.username               | xxxx                                                         | 数据库账号用户名           |
| spring.datasource.password               | yyyy                                                         | 数据库账号密码             |
| spring.datasource.type                   | org.apache.commons.dbcp2.BasicDataSource                     | 数据源类型                 |
| spring.datasource.dbcp2.max-wait-millis  | 10000                                                        | 数据库连接池最长等待时间ms |
| spring.datasource.dbcp2.min-idle         | 5                                                            | 数据库连接池最小空闲       |
| spring.datasource.dbcp2.initial-size     | 5                                                            | 数据库连接池初始大小       |
| spring.datasource.dbcp2.validation-query | SELECT 'x'                                                   | 数据库连接池验证查询       |
| spring.mail.default-encoding             | UTF-8                                                        | 编码类型                   |
| spring.mail.host                         | smtp.163.com                                                 | 邮件服务器主机             |
| spring.mail.username                     | mailusername@163.com                                         | 邮箱用户名                 |
| spring.mail.password                     | mailpwd                                                      | 邮箱密码                   |

### Processor 配置

- 配置文件`./processor/conf/application-prod.properties ` 。

| 配置项                                     | 默认值               | 说明            |
| --------------------------------------- | ----------------- | ------------- |
| server.port                             | 7008              | 默认端口          |
| spring.datasource.url                   | jdbc:mysql://127.0.0.1:3306/WeEvent_processor | JDBC连接串   |
| spring.datasource.driverClassName       | org.mariadb.jdbc.Driver | 连接驱动 |
| spring.jpa.database                     | mysql                | 类型          |
| spring.datasource.username              | root                 | 数据库用户         |
| spring.datasource.password              | 123456               | 数据库密码        |
   
- 配置文件`./processor/conf/processor.properties `。

| 配置项                                     | 默认值               | 说明            |
| --------------------------------------- | ----------------- | ------------- |
| quartz.schedule.name                    | schedule          | quartz标识名称    |
| org.quartz.scheduler.instanceName       | test              | scheduler名称   |
| org.quartz.dataSource.WeEvent_processor | WeEvent_processor | 连接quartz数据库名称 |
| org.quartz.threadPool.threadCount       | 20                | 进程数据          |
| org.quartz.threadPool.threadPriority    | 5                 | 进行优先级         |

```eval_rst
.. note::
    - `org.quartz.dataSource.*`  配置的数据库信息需要和`org.quartz.jobStore.dataSource` 配置的数据源信息一致。
```