## 快速安装

快速安装是为了方便用户搭建开发和测试环境，在单台机器上快速部署`WeEvent`服务。提供`Docker`镜像、Bash脚本两种安装方式。

如果是第一次安装`WeEvent`，参见这里的[系统要求](./environment.html) 。以下安装过程以`Centos 7.2`为例。

### Docker安装

- 获取镜像

  ```bash
  $ docker pull weevent:1.0.0
  ```

  `WeEvent`的镜像里包括了`FISCO-BCOS`网络，`WeEvent`服务的子模块`Broker`和`Governance`，以及各种依赖。

- 创建一个容器

  ```bash
  $ docker run -d -p 8080:8080 weevent:1.0.0 /bin/bash
  ```

### Bash安装

需要的一些基础工具`yum install wget tree tar dos2unix lsof gcc openssl-devel pcre-devel `。

- 获取安装包

  下载安装包[weevent-1.0.0.tar.gz](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-1.0.0.tar.gz)，并且解压到`/tmp/` 。

  ```shell
  $ cd /tmp/
  $ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-1.0.0.tar.gz
  $ tar -zxf weevent-1.0.0.tar.gz
  ```

  解压后目录结构如下：

  ```shell
  $ cd weevent-1.0.0/ 
  $ tree -L 2
  .
  |-- check-service.sh
  |-- config.properties
  |-- install-all.sh
  |-- modules
  |   |-- broker
  |   |-- governance
  |   `-- nginx
  |-- start-all.sh
  |-- stop-all.sh
  |-- third-packages
  |   `-- nginx-1.14.2.tar.gz
  `-- uninstall-all.sh
  ```
  
- 修改配置

  默认配置文件`./config.properties`如下：

  ```properties
  # Required module
  # support 2.0 and 1.3
  fisco-bcos.version=2.0
  # FISCO-BCOS node channel, eg: 127.0.0.1:20200;127.0.0.2:20200
  fisco-bcos.channel=127.0.0.1:20200
  # The path of FISCO-BCOS 2.0 that contain certificate file ca.crt/node.crt/node.key,
  # OR FISCO-BCOS 1.3 that contain ca.crt/client.keystore
  fisco-bcos.node_path=~/FISCO-BCOS/127.0.0.1/node0/conf
  
  # Required module
  nginx.port=8080
  
  # Required module
  broker.port=8090
  
  # Optional module
  governance.enable=false
  governance.governance.port=9099
  governance.mysql.ip=127.0.0.1
  governance.mysql.port=3306
  governance.mysql.user=xxx
  governance.mysql.password=yyy
  ```
  
  配置说明 :
  
  - 区块链FISCO-BCOS
  
    - fisco-bcos.version
  
      `FISCO-BCOS 2.0`和`1.3`版本都支持，推荐使用`2.0`及以上版本。
  
    - fisco-bcos.channel
  
      区块链节点的`channel`访问入口。配置多个节点时用`;`分割，如`127.0.0.1:8821;127.0.0.2:8821`。
  
    - fisco-bcos.node_path
  
      区块链节点的访问证书、私钥存放目录。`FISCO-BCOS 2.0`的证书文件为`ca.crt`、`node.crt`、`node.key`，`1.3`版本的证书文件为`ca.crt`、`client.keystore`。
      如果`WeEvent`服务和区块链节点不在同一台机器上，需要把证书文件拷贝到`WeEvent`机器的当前目录，修改`fisco-bcos.node_path=./`。
  
  - Nginx监听端口`nginx.port`
  
  - Broker监听端口`broker.port`
  
  - Governance模块配置
  
    - `governance.enable`是否安装`Governance`模块，默认为`false`不安装
    - 监听端口`governance.port`
    - Mysql配置`governance.mysql.*`
  
- 一键安装

  以安装到目录`/usr/local/weevent/`为例。

  ```shell
  $ ./install-all.sh -p /usr/local/weevent/
  ```

  正常安装后，输出有如下关键字:

  ```
  8081 port is okay
  8080 port is okay
  param ok
  install module broker 
  install broker success 
  install module nginx 
  install nginx success 
  ```

  如果安装失败，可以在安装日志`./install.log`中查看更多细节。

  目标安装路径`/usr/local/weevent/`的结构如下

  ```shell
  $ cd /usr/local/weevent/
  $ tree -L 2
  .
  |-- broker					    
  |   |-- apps
  |   |-- broker.sh
  |   |-- check-service.sh
  |   |-- conf
  |   |-- deploy-topic-control.sh
  |   |-- lib  
  |   `-- logs
  |-- check-service.sh				
  |-- nginx					    	
  |   |-- conf
  |   |-- html
  |   |-- logs
  |   |-- nginx.sh
  |   |-- nginx_temp
  |   `-- sbin   
  |-- start-all.sh					
  |-- stop-all.sh				    
  `-- uninstall-all.sh
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

  执行如下脚本，卸载所有服务：

  ```shell
  $ ./uninstall-all.sh
  WeEvent is running, stop it first? [Y/N]Y 
  stop broker success
  remove the crontab job success
  stop nginx success
  remove the crontab job success
  really want to uninstall WeEvent? [Y/N]Y
  uninstall WeEvent success 
  ```

快速安装作为一种简易安装方式，所有子模块都是单实例的。生产环境中建议对`Broker`和`Governance`进行多实例部署。各子模块详细部署参见[Broker模块部署](./module/broker.html)和[Governance模块部署](./module/governance.html)。