## Governance模块
如果是第一次安装`WeEvent`，参见这里的[系统要求](../environment.html) 。以下安装以`CentOS 7.2`为例。

因为区块链使用的加密算法很多`OpenJDK`版本没有提供。所以在各`Java`启动脚本里有设置`JAVA_HOME`变量让用户设置符合要求的`JDK`。

### 前置条件

- Zookeeper服务

  必选配置。服务注册和发现会使用到。

  推荐使用`Zookeeper 3.5.5`及其以上版本。具体安装步骤，请参见[Zookeeper安装](https://zookeeper.apache.org/doc/r3.5.7/zookeeperStarted.html)。

- Broker模块

   必选配置，通过`Broker`访问区块链。

   具体安装步骤，请参见[Broker模块安装](./broker.html)。

- Mysql数据库

  可选配置。支持`Mysql`存储数据，如果不配置则使用内置的`H2`数据库。如果要使用Mysql数据库，需要做一个

  切换，切换步骤，请参考[FAQ](https://weeventdoc.readthedocs.io/zh_CN/latest/faq/weevent.html)。

  推荐安装`Mysql 5.7+`版本。具体安装步骤，安装请参见[Mysql安装](http://dev.mysql.com/downloads/mysql/) 。
  
- Processor模块
  可选配置。通过`Processor`触发规则引擎。
  具体安装步骤，请参见[Processor模块安装](./processor.html)。


### 获取安装包

从`github`下载安装包[weevent-governance-1.6.0.tar.gz](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.6.0/weevent-governance-1.6.0.tar.gz)，并且解压到`/usr/local/weevent/`下。

```shell
$ cd /usr/local/weevent/
$ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.6.0/weevent-governance-1.6.0.tar.gz
$ tar -xvf weevent-governance-1.6.0.tar.gz
```

如果`github`下载速度慢，可以尝试[国内下载链接](https://osp-1257653870.cos.ap-guangzhou.myqcloud.com/WeEvent/download/releases/v1.6.0/weevent-governance-1.6.0.tar.gz)。

解压后的目录结构如下

```
$ cd ./weevent-governance-1.6.0
$ tree -L 1
.
|-- apps
|-- check-service.sh
|-- conf
|-- governance.sh
|-- html
|-- init-governance.sh
|-- lib
```

### 修改配置文件

- 配置Zookeeper服务

  可选配置。`./conf/application-prod.properties`中`spring.cloud.zookeeper`配置项。
  
  ```ini
  # spring cloud zookeeper
  spring.cloud.zookeeper.enabled=true
  spring.cloud.zookeeper.connect-string=127.0.0.1:2181
  ```
  
- 配置端口

  在配置文件`./conf/application-prod.properties`中，`Governance` 的服务端口`server.port` ，默认`7009`。

  ```
  server.port=7009
  ```

- 区块链FISCO-BCOS节点(复制Broker模块中，配置连接FISCO-BCOS节点的配置文件以及相关证书)

  - 区块链节点配置文件fisco.properties

    修改`nodes=127.0.0.1:20200`配置项，`nodes`为区块链节点`channel`访问入口。

  - 访问节点的证书文件

    2.x版本的证书文件`ca.crt`、`sdk.crt`、`sdk.key`放在`./conf/`目录下。
    
  - governance支持配置多个节点
  
    配置文件`governance.properties` 示例如下，可以体验节点之间传文件等。
  
    ```properties
    nodeAddressList[0]=127.0.0.1:20200
    nodeAddressList[1]=127.0.0.1:20201
    nodeAddressList[2]=127.0.0.1:20202,127.0.0.1:20203
    ```
  
    


### 初始化数据库

执行脚本`init-governance.sh` 初始化数据库，成功输出如下。否则，用户需要检查数据库配置是否正常。

```shell
$ ./init-governance.sh
init governance db success
```

### 服务启停

通过`./governance.sh start`命令启动服务，正常启动如下：

```shell
$ ./governance.sh start
start weevent-governance success (PID=53926)
add the crontab job success
```


通过`./governance.sh stop`命令停止服务。

进程启动后，会自动添加`crontab`监控任务`./governance.sh monitor`。


### 服务访问

服务启动后通过 http://127.0.0.1:8080/weevent-governance/# 访问governance服务，默认用户名为：admin，默认密码为：123456


### 多视图管理

`Governance`支持同时管理多个`WeEvent`服务和区块链网络， 配置界面如下。

![Governance-multi-view.png](../../image/Governance-multi-view.png)


### 其他
推荐安装`Processor`。具体安装步骤，请参见[Processor模块安装](./processor.html)。

