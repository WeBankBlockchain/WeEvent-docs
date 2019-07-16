## Broker模块

`Broker`是`WeEvent`的核心子模块，负责事件的发布订阅以及对区块链`FISCO-BCOS`的访问。支持`RESTful`、`JsonRPC`、`STOMP`、`MQTT`多种接入协议，也提供了`Java SDK`。

本节介绍`WeEvent`的子模块`Broker`的详细安装步骤，部署系统之前请确认[系统要求](../environment.html) 。快速安装请参见[WeEvent快速安装](../quickinstall.html) 。

以下安装以`CentOS 7.2`为例。

### 前置条件

- 区块链FISCO-BCOS节点

   必选配置。Broker通过区块链`FISCO-BCOS`持久化数据。

   推荐安装`FISCO-BCOS` 2.0版本。具体安装步骤，请参见[FISCO-BCOS 2.0安装](https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-2.0/docs/installation.html)。

- Redis缓存

  可选配置。

  推荐安装`Redis`5.0+版本。具体安装步骤，请参见[Redis安装](https://redis.io/download)。

- Zookeeper服务

  可选配置。当用户使用了`JsonRPC`或者`RESTful`协议的订阅功能时必选配置。

  推荐安装`Zookeeper`3.4+版本。具体安装步骤，请参见[Zookeeper安装](http://zookeeper.apache.org/doc/r3.4.13/zookeeperStarted.html)。


### 获取安装包

下载安装包[weevent-broker-1.0.0.tar.gz](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-broker-1.0.0.tar.gz)，并且解压到`/usr/local/weevent/`下。

``` shell
$ cd /usr/local/weevent/
$ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-broker-1.0.0.tar.gz
$ tar -zxf weevent-broker-1.0.0.tar.gz
```
解压后的目录如下：

```
$ cd ./weevent-broker-1.0.0
$ tree  -L 2
|-- apps
|   `-- weevent-broker-1.0.0.jar
|-- broker.sh
|-- check-service.sh
|-- conf
|   |-- v2
|   |-- application-prod.properties
|   |-- application.properties
|   |-- banner.txt
|   |-- ca.crt
|   |-- client.keystore
|   |-- fisco.properties
|   |-- log4j2.xml
|   |-- server.p12
|   `-- weevent.properties
|-- deploy-topic-control.sh
```

### 修改配置文件
- 区块链FISCO-BCOS节点

  - 区块链节点配置文件fisco.properties

    修改nodes=127.0.0.1:8501配置项，nodes为区块链节点访问IP及channelport访问端口。

  - 访问节点的证书文件

    2.0版本的证书文件`ca.crt`、`node.crt`、`node.key`放在`./conf/v2`目录下。

    1.3版本的证书文件`ca.crt`、`client.keystore`放在`./conf`目录下。

    证书文件生成及获取请参见[FISCO-BCOS 2.0安装](https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-2.0/docs/installation.html)

- 部署和修改合约地址

  - 部署Topic合约  

    运行脚本`./deploy-topic-control.sh `，部署合约并得到合约地址。例如:

    ```shell
    $ ./deploy-topic-control.sh 1
    deploy contract[TopicController] success, address: 0xd99253697e61bf19206ceb4704fc9914d0a4116c
    ```
    
  - 修改配置文件
  
      在配置文件`./conf/fisco.properties`中，替换为生成的合约地址。例如：
  
    ```ini
  topic-controller.address=1:0xd99253697e61bf19206ceb4704fc9914d0a4116c;
    ```
    
      1.3版本的合约地址设置如下:
  
    ```ini
  topic-controller.address=0xd99253697e61bf19206ceb4704fc9914d0a4116c
    ```
    
    更多关于群组和合约部署的细节，请参考[多群组](../../advanced/group.html)。
    
  -  **注意**
    
        每部署一个新合约，使用新的合约地址相当于切换了数据视图，原来合约地址关联的数据都无法继续访问（除非切回原合约地址）。  
  
- 配置Broker监听端口

  配置文件`./conf/application-prod.properties`中`server.port`配置项，默认监听端口8090，根据业务需要配置。
- 配置Redis缓存

  可选配置，为了提高`WeEvent`的通知性能。建议修改配置文件`./conf/weevent.properties`中`redis.*`配置项 。

  ```ini
  # redis服务访问链接
  redis.server.ip=${ip}
  
  # redis服务访问端口
  redis.server.port=${port}
  
  # redis服务访问密码 为了安全，必须使用密码访问
  redis.server.password=${password}
  
  # 基于redis的broker进程缓存容量，当缓存数据大于这个值时，使用LRU淘汰策略
  lru.cache.capacity=65536
  ```

- 配置Zookeeper服务

  配置文件`./conf/weevent.properties`中`broker.zookeeper.*`配置项。

  ```ini
  # zookeeper服务访问链接 示例：127.0.0.1:8080
  broker.zookeeper.ip=${ip}:${port}
  
  # zookeeper上数据存储路径
  broker.zookeeper.path=/event_broker
  
  # 连接zookeeper超时时间 单位：毫秒
  broker.zookeeper.timeout=3000
  ```

- 配置IP白名单

  可选配置，为了安全起见，建议修改配置文件`./conf/weevent.properties`中`ip.check.white-table`配置项。

  ```ini
  # 配置ip白名单,多个ip使用分号分割 示例:127.0.0.1;127.0.0.2
  ip.check.white-table=${ip}
  ```

  注意：`ip.check.white-table`默认为空，允许任何客户端`IP`访问。

- 配置STOMP

  可选配置，为了安全起见，建议修改配置文件`./conf/weevent.properties`中`stomp.*`配置项。

  ```ini
  # stomp协议访问用户名
  stomp.user.login=${username}
  
  # stomp协议访问密码
  stomp.user.passcode=${password}
  
  # 发送心跳时间间隔 单位:秒
  stomp.heartbeats=30
  ```
  注意：`login/passcode` 默认为空，表示不校验`Stomp`请求不进行账号和密码校验。`heartbeats为30`表示配置心跳时间间隔。默认时间间隔30秒，一般不用修改。

- 配置`MQTT Broker`

  配置文件`./conf/weevent.properties`中`mqtt.*`配置项。

  ```ini
  # 客户端使用MQTT协议访问MQTT Broker端口
  mqtt.broker.port=8091
  
  # 心跳时间 单位:秒
  mqtt.broker.keepalive=60
  
  # 客户端使用WebSocket协议访问MQTT Broker链接
  mqtt.websocket.path=/weevent/mqtt
  
  # 客户端使用WebSocket协议访问MQTT Broker端口
  mqtt.websocket.port=8092
  
  # MQTT Broker访问用户名
  mqtt.user.login=
  
  # MQTT Broker访问密码
  mqtt.user.passcode=
  ```
  

更多系统详细配置参见[配置说明](../property.html)

### 服务启停

- 启动服务

  通过`./broker.sh start`命令启动服务，正常启动如下：

  ```shell
  $ ./broker.sh start
  start broker success (PID=89054)
  add the crontab job success
  ```

  通过`./broker.sh stop`命令停止服务。

  进程启动，会自动添加`crontab`监控任务`./broker.sh monitor`。

- 验证服务

  通过`./check-service.sh` 命令检查服务功能是否正常。

  ```shell
  $ ./check-service.sh
  check broker service
  broker service is ok
  ```

### 加入Nginx反向代理

`WeEvent`服务的所有请求都通过`Nginx`负载均衡接入，`Nginx`子模块的安装及详细配置参见[Nginx模块安装及配置](./nginx.html) 。

如需部署多进程实例，将上述步骤安装好的`Broker`目录打包拷贝到其他机器上，解压启动即可。
