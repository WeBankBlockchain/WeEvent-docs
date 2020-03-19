## Gateway模块

`Gateway`模块作为`WeEvent`服务的统一访问入口，提供负载均衡、限流和熔断等功能。

如果是第一次安装`WeEvent`，参见这里的[系统要求](../environment.html) 。以下安装以`CentOS 7.2`为例。

因为区块链使用的加密算法很多`OpenJDK`版本没有提供。所以在各`Java`启动脚本里有设置`JAVA_HOME`变量让用户设置符合要求的`JDK`。

### 前置条件

- Zookeeper服务

  必选配置。服务注册和发现会使用到。

  推荐使用`Zookeeper 3.5.5`版本。具体安装步骤，请参见[Zookeeper安装](http://zookeeper.apache.org/doc/r3.4.13/zookeeperStarted.html)。


### 获取安装包

从`github`下载安装包[weevent-gateway-1.2.0.tar.gz](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.2.0/weevent-gateway-1.2.0.tar.gz)，并且解压到`/usr/local/weevent/`下。

``` shell
$ cd /usr/local/weevent/
$ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.2.0/weevent-gateway-1.2.0.tar.gz
$ tar -zxf weevent-gateway-1.2.0.tar.gz
```
如果`github`下载速度慢，可以尝试[国内下载链接](https://www.fisco.com.cn/cdn/weevent/download/releases/v1.2.0/weevent-gateway-1.2.0.tar.gz)。

解压后的目录如下：

```
$ cd ./weevent-gateway-1.2.0
$ tree  -L 1
|-- apps
|-- gateway.sh
|-- check-service.sh
|-- conf
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
  


更多系统详细配置参见[配置说明](../property.html)

### 服务启停

- 启动服务

  配置`gateway.sh`脚本中`JAVA_HOME`

  通过`./gateway.sh start`命令启动服务，正常启动如下：

  ```shell
  $ ./gateway.sh start
  start weevent-broker success (PID=89059)
  add the crontab job success
  ```

  通过`./gateway.sh stop`命令停止服务。

  进程启动后，同时添加`crontab`监控任务`./gateway.sh monitor`。

