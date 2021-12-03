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

  配置文件`./broker/conf/fisco.yml`， 与governance依赖的配置文件`fisco.yml`相同。

  改配置文件分为两部分，`weevent core config`和 `fisco bcos sdk config`部分，前者是weevent配置，后者是完全依照[FISCO-BCOS SDK 配置说明](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/sdk/java_sdk/configuration.html) 配置。
  
  ```yaml
  ################ weevent core config ################
  
  version: 2.0
  orgId: fisco
  
  timeout: 10000
  poolSize: 10
  maxPoolSize: 200
  keepAliveSeconds: 10
  
  consumerHistoryMergeBlock: 8
  consumerIdleTime: 1000
  
  
  ################ fisco bcos sdk config ################
  # https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/sdk/java_sdk/configuration.html
  
  #cryptoMaterial:
  #  certPath: "conf"
  #  caCert: "conf/ca.crt"
  #  sslCert: "conf/sdk.crt"
  #  sslKey: "conf/sdk.key"
  #  enSslCert: "conf/gm/gmensdk.crt"
  #  enSslKey: "conf/gm/gmensdk.key"
  
  account:
  # ECDSA_TYPE
    accountAddress: "0x64fa644d2a694681bd6addd6c5e36cccd8dcdde3"
  #  SM_TYPE
  #  accountAddress: "0x4278900c23e4b364ba6202a24682d99be9ff8cbc"
  #  accountFileFormat: "pem"
  #  accountFilePath: ""
    keyStoreDir: "account"
  #  password: ""
  
  #amop:
  #  - publicKeys: [ "conf/amop/consumer_public_key_1.pem" ]
  #    topicName: "PrivateTopic1"
  #  - password: "123456"
  #    privateKey: "conf/amop/consumer_private_key.p12"
  #    topicName: "PrivateTopic2"
  
  
  network:
    peers:
      - "127.0.0.1:20200"
  
  threadPool:
    channelProcessorThreadSize: "16"
    maxBlockingQueueSize: "102400"
    receiptProcessorThreadSize: "16"
  ```
  
```eval_rst
.. note::
   - account.accountAddress为发起交易的账户地址，在`keyStoreDir`目录下存放该账户的私钥，也可以自己重新生成私钥替换，但需保持固定地址，因为topic管理等有权限控制，其他账户无权限。详细逻辑可参考`weevent-core/src/main/java/com/webank/weevent/core/fisco/web3sdk/v2/solc10/solidity`合约实现。
```

  

| 配置项                       | 默认值               | 配置说明                                |
| ---------------------------- | -------------------- | --------------------------------------- |
| version                      | 2.0                  | FISCO-BCOS版本，支持2.x                 |
| orgId                       | fisco                | 机构名，按机构实际名称填写即可          |
| timeout                   | 10000                | 交易执行超时时间，单位毫秒              |
| poolSize | 10                   | 最小线程数                       |
| maxPoolSize | 100                  | 最大线程数                       |
| keepAliveSeconds | 10                   | 线程空闲时间，单位秒             |
| consumerIdleTime | 1000                 | 区块链新增块事件检测周期，单位毫秒      |
| consumerHistoryMergeBlock | 8                    | 事件过滤的区块范围                      |

  


- WeEvent服务配置

  配置文件`./broker/conf/weevent.properties`。

  | 配置项                               | 默认值 | 配置说明                                                     |
  | ------------------------------------ | ------ | ------------------------------------------------------------ |
  | ip.check.white-table                 |        | IP白名单。多个`IP`地址，以";"分割。<br />默认为空时表示允许任何客户端访问。 |
  | block.chain.type                     | fisco  | 区块链类别，fisco或fabric                                    |
  | stomp.heartbeats                     | 30     | stomp心跳间隔，单位秒                                        |
  | mqtt.broker.tcp.port                 | 7001   | 使用tcp访问MQTT的端口，默认不开启                            |
  | mqtt.broker.keepalive                | 60     | mqtt连接空闲时间，单位秒                                     |
  | mqtt.broker.security.ssl             | false  | mqtt是否开启ssl                                              |
  | mqtt.broker.security.ssl.client_auth | false  | mqtt ssl是否开启双向认证                                     |
  | mqtt.broker.security.ssl.ca_cert     |        | ssl ca证书                                                   |
  | mqtt.broker.security.ssl.server_cert |        | 服务端证书                                                   |
  | mqtt.broker.security.ssl.server_key  |        | 服务端私钥                                                   |

### Governance

- 配置文件`./governance/conf/application-prod.properties`。

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
  | spring.datasource.dbcp2.validation-query | SELECT '1'                                                   | 数据库连接池验证查询       |
  
- 配置文件`./governance/conf/governance.properties`

  | 配置项          | 默认值 | 配置说明                                                     |
  | --------------- | ------ | ------------------------------------------------------------ |
  | nodeAddressList |        | 数组，可连接多个不同节点，需要与fisco.yml里配置的节点证书相同 |

### Processor 配置

- 配置文件`./processor/conf/application-prod.properties` 。

  | 配置项                                   | 默认值               | 说明            |
  | --------------------------------------- | ----------------- | ------------- |
  | server.port                             | 7008              | 默认端口          |
  | spring.datasource.url                   | jdbc:mysql://127.0.0.1:3306/WeEvent_processor | JDBC连接串   |
  | spring.datasource.driverClassName       | org.mariadb.jdbc.Driver | 连接驱动 |
  | spring.jpa.database                     | mysql                | 类型          |
  | spring.datasource.username              | root                 | 数据库用户         |
  | spring.datasource.password              | 123456               | 数据库密码        |

- 配置文件`./processor/conf/processor.properties`。

  | 配置项                                   | 默认值               | 说明            |
  | --------------------------------------- | ----------------- | ------------- |
  | quartz.schedule.name                    | schedule          | quartz标识名称    |
  | org.quartz.scheduler.instanceName       | test              | scheduler名称   |
  | org.quartz.dataSource.WeEvent_processor | WeEvent_processor | 连接quartz数据库名称 |
  | org.quartz.threadPool.threadCount       | 20                | 进程数据          |
  | org.quartz.threadPool.threadPriority    | 5                 | 进行优先级         |


```eval_rst
.. note::
   - `org.quartz.dataSource.*`  配置的数据库信息需要和org.quartz.jobStore.dataSource配置的数据源信息一致。
```

### 国密配置

前提条件：区块链为国密版本。

配置方式与[FISCO-BCOS SDK配置](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/sdk/java_sdk/configuration.html)方式一致：

1. 将国密证书放到 `classpath/conf/gm`目录下（fisco-bcos sdk会自动检查链是国密还是非国密版本）。
2. `fisco.yml`中`account.accountAddress`字段改为国密账户地址，并将该账户私钥文件放到 `keyStoreDir`目录下。WeEvent默认已有账户，若使用新账户直接替换账户地址和私钥文件即可。