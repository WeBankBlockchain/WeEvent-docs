## Broker模块

本节介绍`WeEvent`的子模块`Broker`的详细安装步骤。快速安装请参见[WeEvent快速安装](../quickinstall.html) 。在一台机器上详细安装，和通过快速安装然后把目标路径中的`broker`子目录打包拷贝到这台机器，效果是一样的。

`Broker`是`WeEvent`的核心子模块，负责事件的发布订阅以及对区块链`FISCO-BCOS`的访问。支持`RESTful`、`JsonRPC`、`STOMP`、`MQTT`多种接入协议，也提供了`Java SDK`。

如果是第一次安装`WeEvent`，参见这里的[系统要求](../environment.html) 。以下安装以`CentOS 7.2`为例。

### 前置条件

- 区块链FISCO-BCOS节点

   必选配置。Broker通过区块链`FISCO-BCOS`持久化数据。

   推荐使用`FISCO-BCOS 2.0`版本。具体安装步骤，请参见[FISCO-BCOS 2.0安装](https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-2.0/docs/installation.html)。

- Redis缓存

  可选配置。

  推荐使用`Redis 5.0+`版本。具体安装步骤，请参见[Redis安装](https://redis.io/download)。

- Zookeeper服务

  可选配置。当使用了`JsonRPC`或者`RESTful`协议的订阅功能起来`Zookeeper`主备切换。

  推荐使用`Zookeeper 3.5.5`版本。具体安装步骤，请参见[Zookeeper安装](http://zookeeper.apache.org/doc/r3.4.13/zookeeperStarted.html)。


### 获取安装包

从`github`下载安装包[weevent-broker-1.0.0.tar.gz](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-broker-1.0.0.tar.gz)，并且解压到`/usr/local/weevent/`下。

``` shell
$ cd /usr/local/weevent/
$ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-broker-1.0.0.tar.gz
$ tar -zxf weevent-broker-1.0.0.tar.gz
```
如果`github`下载速度慢，可以尝试[国内下载链接](https://www.fisco.com.cn/cdn/weevent/download/releases/v1.0.0/weevent-broker-1.0.0.tar.gz)。

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

    修改`nodes=127.0.0.1:20200`配置项，`nodes`为区块链节点`channel`访问入口。

  - 访问节点的证书文件

    2.0版本的证书文件`ca.crt`、`node.crt`、`node.key`放在`./conf/v2`目录下。

    1.3版本的证书文件`ca.crt`、`client.keystore`放在`./conf`目录下。

    证书文件生成及获取请参见[FISCO-BCOS 2.0安装](https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-2.0/docs/installation.html)

- 部署系统合约

  运行脚本`./deploy-topic-control.sh `部署合约。例如:

  ```shell
$ ./deploy-topic-control.sh
  2019-08-19 17:59:31 topic control address in every group:
  1	0xc6fc72f0fe6ebf9881a2103f2829d0e98d020062	[new]
  2	0xd85d3345f8a21f4fd6197c72266ae3e3106e5e1c	[new]
  ```
  
  脚本会检查之前是否部署过合约，重复执行不影响。
  
- 配置Broker监听端口

  可选配置。`./conf/application-prod.properties`中`server.port`配置项，默认监听端口`7000`，根据业务需要配置。
  
- 配置Redis缓存

  可选配置。`./conf/weevent.properties`中`redis.*`配置项 ，配置缓存可以提高`WeEvent`的通知性能。

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

  可选配置。配置文件`./conf/weevent.properties`中`broker.zookeeper.*`配置项。

  ```ini
  # zookeeper服务访问链接 示例：127.0.0.1:8080
  broker.zookeeper.ip=${ip}:${port}
  # zookeeper上数据存储路径
  broker.zookeeper.path=/event_broker
  # 连接zookeeper超时时间 单位：毫秒
  broker.zookeeper.timeout=3000
  ```
  
- 配置IP白名单

  可选配置。`./conf/weevent.properties`中配置了值就会开启白名单验证。

  ```ini
  # 配置ip白名单,多个ip使用分号分割 示例:127.0.0.1;127.0.0.2
  ip.check.white-table=${ip}
  ```

  `ip.check.white-table`默认为空，允许任何客户端`IP`访问。

- 配置STOMP

  可选配置。`./conf/weevent.properties`中`stomp.*`配置项。

  ```ini
  # stomp协议访问用户名
  stomp.user.login=${username}
  # stomp协议访问密码
  stomp.user.passcode=${password}
  # 发送心跳时间间隔 单位:秒
  stomp.heartbeats=30
  ```
  
- 配置`MQTT Broker`

  可选配置。配置文件`./conf/weevent.properties`中`mqtt.*`配置项。

  ```ini
  # 客户端使用MQTT协议访问MQTT Broker端口
  mqtt.broker.port=7001
  # 心跳时间 单位:秒
  mqtt.broker.keepalive=60
  # 客户端使用WebSocket协议访问MQTT Broker链接
  mqtt.websocket.path=/weevent/mqtt
  # 客户端使用WebSocket协议访问MQTT Broker端口
  mqtt.websocket.port=7002
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

将部署好的`Broker`配置到`Nginx`提供对外服务。`Nginx`子模块的安装及详细配置参见[Nginx模块安装及配置](./nginx.html) 。

如需部署其更多实例，将上述步骤安装好的`Broker`目录拷贝到目标位置，启动即可。

