## Broker模块

本节介绍`WeEvent`的子模块`Broker`的详细安装步骤。快速安装请参见[WeEvent快速安装](../quickinstall.html) 。在一台机器上详细安装，和通过快速安装然后把目标路径中的`broker`子目录打包拷贝到这台机器，效果是一样的。

如果是第一次安装`WeEvent`，参见这里的[系统要求](../environment.html) 。以下安装以`CentOS 7.2`为例。

### 前置条件

- Zookeeper服务

  必选配置。服务注册和发现会使用到。

  推荐使用`Zookeeper 3.5.5`版本。具体安装步骤，请参见[Zookeeper安装](http://zookeeper.apache.org/doc/r3.4.13/zookeeperStarted.html)。

- 区块链FISCO-BCOS节点

   必选配置。Broker通过区块链`FISCO-BCOS`持久化数据。


### 获取安装包

从`github`下载安装包[weevent-broker-1.2.0.tar.gz](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.2.0/weevent-broker-1.2.0.tar.gz)，并且解压到`/usr/local/weevent/`下。

``` shell
$ cd /usr/local/weevent/
$ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.2.0/weevent-broker-1.2.0.tar.gz
$ tar -zxf weevent-broker-1.2.0.tar.gz
```
如果`github`下载速度慢，可以尝试[国内下载链接](https://www.fisco.com.cn/cdn/weevent/download/releases/v1.2.0/weevent-broker-1.2.0.tar.gz)。

解压后的目录如下：

```
$ cd ./weevent-broker-1.2.0
$ tree  -L 1
|-- apps
|-- broker.sh
|-- check-service.sh
|-- conf
|-- deploy-fabric-topic-control.sh
|-- deploy-topic-control.sh
|-- gen-cert-key.sh
`-- lib
```

### 修改配置文件

- 配置Zookeeper服务

  可选配置。`./conf/application-prod.properties`中`spring.cloud.zookeeper`配置项。
  
  ```ini
  # spring cloud zookeeper
  spring.cloud.zookeeper.enabled=true
  spring.cloud.zookeeper.connect-string=127.0.0.1:2181
  ```
  
- 区块链FISCO-BCOS节点

  - 区块链节点配置文件fisco.properties

    修改`nodes=127.0.0.1:20200`配置项，`nodes`为区块链节点`channel`访问入口。

  - 访问节点的证书文件

    2.0版本的证书文件`ca.crt`、`node.crt`、`node.key`放在`./conf/v2`目录下。

    证书文件生成及获取请参见[FISCO-BCOS 2.0安装](https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-2.0/docs/installation.html)

- 部署系统合约

  运行脚本`./deploy-topic-control.sh `部署合约。例如:

  ```shell
  $ ./deploy-topic-control.sh
  2020-03-06 10:33:37 topic control address in every group:
  topic control address in group: 1
          version: 10     address: 0x23df89a2893120f686a4aa03b41acf6836d11e5d     new: true
  topic control address in group: 2
          version: 10     address: 0x23df89a2893120f686a4aa03b41acf6836d11e5d     new: true
  ```
  
  脚本会检查之前是否部署过合约，重复执行只是显示已有数据。
  
- 配置Broker监听端口

  可选配置。`./conf/application-prod.properties`中`server.port`配置项，默认监听端口`7000`，根据业务需要配置。
  
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
  # 发送心跳时间间隔 单位:秒
  stomp.heartbeats=30
  ```
  
- 配置`MQTT Broker`

  可选配置。配置文件`./conf/weevent.properties`中`mqtt.*`配置项。

  ```ini
  # 客户端使用WebSocket协议访问MQTT Broker端口
  mqtt.broker.port=7001
  # 心跳时间 单位:秒
  mqtt.broker.keepalive=60
  # 客户端使用WebSocket协议访问MQTT Broker链接
  mqtt.websocket.path=/weevent-broker/mqtt
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

  进程启动后，会自动加入集群，同时添加`crontab`监控任务`./broker.sh monitor`。

- 验证服务

  通过`./check-service.sh` 命令检查服务功能是否正常。

  ```shell
  $ ./check-service.sh
  check broker service
  broker service is ok
  ```

