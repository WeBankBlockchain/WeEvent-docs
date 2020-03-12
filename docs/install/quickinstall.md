## 快速安装

快速安装是为了方便用户搭建开发和测试环境，在单台机器上快速部署`WeEvent`服务。提供`Docker`镜像、Bash脚本两种安装方式。

如果是第一次安装`WeEvent`，参见这里的[系统要求](./environment.html) 。以下安装过程以`Centos 7.2`为例。

### Docker镜像安装

  ```bash
  $ docker pull weevent/weevent:1.2.0; docker run -d -p 8080:8080 weevent/weevent:1.2.0 /root/run.sh
  ```

  `WeEvent`的镜像里包括了`FISCO-BCOS`网络，`WeEvent`服务的各个子模块以及各种依赖。


### Bash安装

需要的一些基础工具`yum install wget tree tar netstat`。

- 获取安装包

  从`github`下载安装包[weevent-1.2.0.tar.gz](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.1.0/weevent-1.2.0.tar.gz)，并且解压到`/tmp/` 。

  ```shell
  $ cd /tmp/
  $ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.2.0/weevent-1.2.0.tar.gz
  $ tar -zxf weevent-1.2.0.tar.gz
  ```

  如果`github`下载速度慢，可以尝试[国内下载链接](https://www.fisco.com.cn/cdn/weevent/download/releases/v1.2.0/weevent-1.2.0.tar.gz)。
解压后目录结构如下：
  
  ```shell
  $ cd weevent-1.2.0/ 
  $ tree -L 2
  .
  ├── bin
  │   ├── start-all.sh
  │   └── stop-all.sh
  ├── config.properties
  ├── install-all.sh
  ├── modules
  │   ├── broker
  │   ├── gateway
  │   ├── governance
  │   ├── lib
  │   └── processor
  ```
  
- 修改配置

  默认配置文件`./config.properties`如下：

  ```properties
  #java jdk environment
  JAVA_HOME=/usr/local/jdk1.8.0_191
  
  # Required module
  # support 2.0
  fisco-bcos.version=2.0
  # FISCO-BCOS node channel, eg: 127.0.0.1:20200;127.0.0.2:20200
  fisco-bcos.channel=127.0.0.1:20200
  # The path of FISCO-BCOS 2.0 that contain certificate file ca.crt/node.crt/node.key,
  # OR FISCO-BCOS 1.3 that contain ca.crt/client.keystore
  fisco-bcos.node_path=~/FISCO-BCOS/127.0.0.1/node0/conf
  
  # Required module
  gateway.port=8080
  zookeeper.connect-string=127.0.0.1:2181
  
  # Required module
  broker.port=7000
  
  # Optional module
  governance.enable=false
  governance.port=7009
  #support both h2 and mysql, default h2
  database.type=h2
  #mysql.ip=127.0.0.1
  #mysql.port=3306
  #mysql.user=xxx
  #mysql.password=yyy
  
  # Optional module
  processor.enable=true
  processor.port=7008
  ```
  
  配置说明 :
  
- JDK环境变量`JAVA_HOME`
  
  - 区块链FISCO-BCOS
  
    - `fisco-bcos.version`
  
      支持`2.0`及其以上版本。
  
    - `fisco-bcos.channel`
  
      区块链节点的`channel`访问入口。配置多个节点时用`;`分割，如`127.0.0.1:20200;127.0.0.2:20200`。
  
    - `fisco-bcos.node_path`
  
      区块链节点的访问证书、私钥存放目录。
      
      `FISCO-BCOS 2.0`的证书文件为`ca.crt`、`node.crt`、`node.key`。如果`WeEvent`服务和区块链节点不在同一台机器上，需要把证书文件拷贝到`WeEvent`所在机器的当前目录，修改`fisco-bcos.node_path=./`。
    
  - Gateway监听端口`gateway.port`
  
  - Broker监听端口`broker.port`
  
  - Governance模块配置
  
    - `governance.enable`是否安装`Governance`模块，默认为`false`不安装
    - 监听端口`governance.port`
    - 默认使用内置的`H2`数据库，也支持`Mysql`配置`mysql.*`
  
  - Proceessor模块配置
  
    - `proceessor.enable`是否安装`Proceessor`模块，默认为`false`不安装
    - 监听端口`processor.port`
  
- 一键安装

  以安装到目录`/usr/local/weevent/`为例。

  ```shell
  $ ./install-all.sh -p /usr/local/weevent/
  ```

  正常安装后，输出有如下关键字:

  ```
  7000 port is okay
  8080 port is okay
  param ok
  install module gateway 
  install gateway success 
  install module broker 
  install broker success 
  ```

  目标安装路径`/usr/local/weevent/`的结构如下

  ```shell
  $ cd /usr/local/weevent/
  $ tree -L 1
  .
  |-- broker
  |-- lib
  |-- gateway
  |-- start-all.sh			    
  `-- stop-all.sh
  ```
  
- 启停服务
  - 启动服务

    在服务安装目录下`/usr/local/weevent`，通过`start-all.sh`命令启动所有服务 ，正常启动如下：

    ```shell
    $ ./start-all.sh
    start broker success (PID=3642)
    add the crontab job success
    start nginx success (PID=3643)
    add the crontab job success
    ```

  - 停止所有服务的命令`./stop-all.sh`。

- 检查是否安装成功

    ```shell
    $ ./check-service.sh
    check broker service 
    broker service is ok
    ```

- 卸载服务

  所有服务停止后，直接删除目录即可。


快速安装作为一种简易安装方式，所有子服务都是单实例的，生产环境中建议多实例部署。各子模块详细部署参见[Broker模块部署](./module/broker.html)和[Governance模块部署](./module/governance.html)。