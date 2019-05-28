## Broker模块

`Broker`是`WeEvent`的核心子模块，负责事件的发布订阅以及对区块链`FISCO-BCOS`的访问。支持`RESTful`、`JsonRPC`、`STOMP`、`MQTT`多种接入协议，也提供了`Java SDK`。

本节介绍`WeEvent`的子模块`Broker`的详细安装步骤，部署系统之前请确认[系统要求](../environment.html) 。快速安装请参见[WeEvent快速安装](../quickinstall.html) 。

以下安装以`CentOS 7.2`为例。

### 前置条件

- 区块链FISCO-BCOS节点

   必选配置。Broker通过区块链`FISCO-BCOS`持久化数据。

   推荐安装`FISCO-BCOS` 2.0版本。具体安装步骤，请参见[FISCO-BCOS 2.0安装](https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-2.0/docs/installation.html)。

- Redis缓存

  可选配置。当多个消费者订阅同一个`Topic`时，`Redis`缓存能及时事件通知。

  推荐安装`Redis`5.0+版本。具体安装步骤，请参见[Redis安装](https://redis.io/download)。

- Zookeeper服务

  可选配置。当用户使用了`JsonRPC`或者`RESTful`的订阅功能时，必须要配置`Zookeeper`服务。

  推荐安装`Zookeeper`3.4+版本。具体安装步骤，请参见[Zookeeper安装](http://zookeeper.apache.org/doc/r3.4.13/zookeeperStarted.html)。


### 获取安装包

下载安装包[weevent-broker安装包](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-broker-1.0.0.tar.gz)，并且解压到`/usr/local/weevent/`下。

``` shell
$ cd /usr/local/weevent/
$ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-broker-1.0.0.tar.gz
$ tar -zxf weevent-broker-1.0.0.tar.gz
```
如果机器无法访问外网`wget`执行失败，可以通过别的方式下载再`rz`上传。

解压后的目录如下：

```
$ cd ./weevent-broker-1.0.0
$ tree  -L 2
|-- apps
|   |-- weevent-client-1.0.0.jar
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
`-- gen-cert-key.sh
```

### 修改配置文件
- 区块链FISCO-BCOS节点

  - 区块链节点配置文件fisco.properties

  - 访问节点的证书文件

    1.3版本的证书文件`ca.crt`和`client.keystore`放在`./conf`目录下。

    2.0版本的证书文件`ca.crt`和`node.crt`、`node.key`放在`./conf/v2`目录下。

    更详细的配置文件说明，请参见[Web3sdk配置说明](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/sdk/sdk.html)。

- 修改合约地址

  - 部署Topic合约  

    运行脚本`./deploy-topic-control.sh`，部署合约并得到合约地址（2.0版本需要在脚本后添加参数，群组ID），例如:

    ```shell
    $ ./deploy-topic-control.sh
    deploy contract[TopicController] success, address: 0xd99253697e61bf19206ceb4704fc9914d0a4116c
    ```

  - 修改配置文件

      在配置文件`./conf/fisco.properties`中，替换为生成的合约地址。例如：

      ```ini
      topic-controller.address=0xd99253697e61bf19206ceb4704fc9914d0a4116c
      ```
      2.0版本每个群组都有自己的合约地址，多个地址用`;`分号分割。格式为`1:0xd99253697e61bf19206ceb4704fc9914d0a4116c;2:0xd99253697e61bf19206ceb4704fc9914d0a4116d`

  -  **注意**

      每部署一个新合约，使用新的合约地址相当于切换了数据视图，原来合约地址关联的数据都无法继续访问（除非切回原合约地址）。  

- 配置Broker监听端口

  配置文件./conf/application-prod.properties中server.port配置项，默认监听端口8081，根据业务需要配置。
- 配置Redis缓存

  可选配置，为了提高`WeEvent`的通知性能。建议修改配置文件`./conf/weevent.properties`中`redis.*`配置项 。

  ```ini
  #redis服务访问链接
  redis.server.ip=${ip}
  #redis服务访问端口
  redis.server.port=${port}
  #redis服务访问密码 为了安全，必须使用密码访问
  redis.server.password=${password}
  #基于redis的broker进程缓存容量，当缓存数据大于这个值时，使用LRU淘汰策略
  lru.cache.capacity=65536
  ```

- 配置Zookeeper服务

  配置文件`./conf/weevent.properties`中`broker.zookeeper.*`配置项。

  ```ini
  #zookeeper服务访问链接 示例：127.0.0.1:8080
  broker.zookeeper.ip=${ip}:${port}
  #zookeeper上数据存储路径
  broker.zookeeper.path=/event_broker
  #连接zookeeper超时时间 单位：毫秒
  broker.zookeeper.timeout=3000
  ```

- 配置IP白名单

  可选配置，为了安全起见，建议修改配置文件`./conf/weevent.properties`中`ip.check.white-table`配置项。

  ```ini
  #配置ip白名单,多个ip使用分号分割 示例:127.0.0.1;127.0.0.2
  ip.check.white-table=${ip}
  ```

  注意：`ip.check.white-table`默认为空，允许任何客户端`IP`访问。

- 配置STOMP

  可选配置，为了安全起见，建议修改配置文件`./conf/weevent.properties`中`stomp.*`配置项。

  ```ini
  #stomp协议访问用户名
  stomp.user.login=${username}
  #stomp协议访问密码
  stomp.user.passcode=${password}
  #发送心跳时间间隔 单位:秒
  stomp.heartbeats=30
  ```
  注意：`login/passcode` 默认为空，表示不校验`Stomp`请求不进行账号和密码校验。`heartbeats为30`表示

- 配置Mosquitto代理

  配置文件`./conf/weevent.properties`中`mqtt.broker.*`配置项。

  ```ini
  #mqtt代理访问链接 示例：tcp://127.0.0.1:1883
  mqtt.broker.url=tcp://${ip}:${port}
  #mqtt代理访问用户名
  mqtt.broker.user=${username}
  #mqtt代理访问密码
  mqtt.broker.password=${password}
  #消息质量0,1,2三种配置 详情见:https://www.hivemq.com/blog/mqtt-essentials-part-6-mqtt-quality-of-service-levels/
  mqtt.broker.qos=2
  #访问超时时间 单位:毫秒
  mqtt.broker.timeout=5000
  ```

- 开启HTTPS功能

  可选配置，为了安全起见，建议通过`HTTPS`方式访问`Broker`服务，需要配置一个访问证书。服务采用[PKCS12格式](https://tools.ietf.org/html/rfc7292) 。

  安装包里自带了默认证书`./conf/server.p12` ，可以直接使用。用户也可以选择使用包里的`./gen-cert-key.sh`脚本重新生成证书。使用新证书过程如下：

  - 生成证书

    ```shell	
    $ cd ./conf
    $ rm server.p12
    $ ../gen-cert-key.sh
    Enter keystore password:  
    Re-enter new password: 
    What is your first and last name?
      [Unknown]:  zhangsan
    What is the name of your organizational unit?
      [Unknown]:  org1       
    What is the name of your organization?
      [Unknown]:  orgname
    What is the name of your City or Locality?
      [Unknown]:  shenzhen
    What is the name of your State or Province?
      [Unknown]:  guangdong
    What is the two-letter country code for this unit?
      [Unknown]:  cn
    Is CN=zhangsan, OU=org1, O=orgname, L=shenzhen, ST=guangdong, C=cn correct?
      [no]:  y
     $ cd ..
    ```

    根据提示填写证书主题信息和证书密码，生成成功会替换当前证书文件`./conf/server.p12`。

  - 修改配置文件

    参见`./conf/application-prod.properties`中`server.ssl.*`配置项。

    ```ini
    #开启https功能
    server.ssl.enabled=true
    #证书文件路径
    server.ssl.key-store=classpath:server.p12
    #证书密码
    server.ssl.key-store-password=123456
    #证书文件类型
    server.ssl.keyStoreType=PKCS12
    #key别名
    server.ssl.keyAlias=weevent
    ```

### 服务启停

- 启动服务

  通过`./broker.sh start`命令启动服务，正常启动如下：

  ```shell
  $ ./broker.sh start
  start broker success (PID=89054)
  add the crontab job success
  ```

  通过`./broker.sh stop`命令停止服务。

  `./broker.sh start`命令会启动进程，并且将进程监控命令`./broker.sh monitor`添加到`crontab`里。

  `./broker.sh stop`命令在进程成功停止后会移除`crontab`监控任务。

- 验证服务

  通过`./check-service.sh` 命令检查服务功能是否正常。

  ```shell
  $ ./check-service.sh
  check broker service
  broker service is ok
  ```

### 加入Nginx反向代理

`WeEvent`服务的所有请求都通过`Nginx`负载均衡接入，`Nginx`子模块的安装参见[Nginx模块安装](./module/nginx.md) 。

如果需要部署多个进程实例，将上述步骤安装好的`Broker`目录打包拷贝到其他机器上，解压启动即可。

`Nginx`配置文件`./conf/conf.d/rs.conf`中， 下面为两个`Broker`进程的样例，然后通过`Nginx`重新加载配置生效。

```nginx
upstream broker_backend{
    server 127.0.0.1:8081 weight=100 max_fails=3;
    server 127.0.0.2:8081 weight=100 max_fails=3;

    ip_hash;
    keepalive 1024;
}
```

`Nginx`重启命令说明
```shell
$ ./nginx -t
$ ./nginx -s reload
```