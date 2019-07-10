## 快速安装

快速安装是为了方便用户搭建开发和测试环境，在单台机器上快速部署`WeEvent`服务。提供`Docker`镜像、一键脚本两种安装方式，推荐使用`Docker`镜像。

如果是第一次安装`WeEvent`，参见这里的[系统要求](./environment.html) 。

### Docker安装

- Docker镜像

  `WeEvent`的镜像里包括了`FISCO-BCOS`网络，`WeEvent`的`Broker`和`Governance`服务。

- 安装命令

  ```bash
  
  ```

### 一键安装

为了简化配置，在一键安装时建议将`WeEvent`服务和区块链`FISCO-BCOS`节点安装在同一台机器上。

- 获取安装包

  下载安装包[WeEvent快速安装包](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-1.0.0.tar.gz)，并且解压到`/tmp/` 。

  ```shell
  $ cd /tmp/
  $ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-1.0.0.tar.gz
  $ tar -zxf weevent-1.0.0.tar.gz
  ```

  解压后目录结构如下：

  ```
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
  |-- README.md
  |-- start-all.sh
  |-- stop-all.sh
  |-- third-packages
  |   |-- nginx-1.14.2.tar.gz
  |   `-- pcre-8.20.tar.gz
  `-- uninstall-all.sh
  ```
  
- 修改配置

  默认配置文件`./config.properties`如下：

  ```properties
  # Required module
  # support 2.0 and 1.3
  fisco-bcos.version=2.0
  # FISCO-BCOS node channel, eg: 127.0.0.1:8821;127.0.0.2:8821
  cfisco-bcos.hannel=127.0.0.1:8821
  # FISCO-BCOS's node path
  fisco-bcos.node_path=/data/FISCO-BCOS/127.0.0.1/node0
  
  # Required module
  nginx.port=8080
  
  # Required module
  broker.port=8081
  
  # Optional module
  governance.enable=false
  governance.governance.port=8082
  governance.mysql.ip=127.0.0.1
  governance.mysql.port=3306
  governance.mysql.user=xxx
  governance.mysql.password=yyy
  ```

  配置说明：  

  - fisco-bcos.version

    `FISCO-BCOS`2.0和1.3版本都支持，推荐使用`2.0`及以上版本。

    - fisco-bcos.channel

      区块链节点的`channel`访问入口。配置多个节点时用`;`分割，如`127.0.0.1:8821;127.0.0.2:8821`。

    - fisco-bcos.node_path

      区块链节点的访问证书、私钥存放的位置。值为区块链节点的安装目录。

  - Nginx监听端口`nginx.port`

  - Broker监听端口`broker.port`

  - Governance模块配置

    - `governance.enable`是否安装Governance模块，默认false不安装
    - 监听端口`governance.port`
    - Mysql配置`governance.mysql.*`


- 自动安装

  以安装到目录`/usr/local/weevent/`为例。

  ```shell
  $ ./install-all.sh -p /usr/local/weevent/
  ```

  正常安装后，输出有如下关键字：

  ```
  deploy contract success
  contract_address:0x9392da80a7ae52fdbcd3698111b23f045cf0745c
  broker install success
  build & install pcre
  build & install nginx
  nginx install success
  ```

  目标安装路径`/usr/local/weevent/`的结构如下

  ```
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

    通过`start-all.sh`命令启动所有服务 ，正常启动如下：

    ```shell
    $ ./start-all.sh
    start broker success (PID=3642)
    add the crontab job success
    start nginx success (PID=3643)
    add the crontab job success
    ```

  ​	停止所有服务的命令`./stop-all.sh`。

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
  Please confirm if you remove the WeEvent? [Y/N]Y
  uninstall WeEvent success 
  ```

- 注意事项
  一键安装脚本作为一种简易安装方式，所有子模块都是单实例的。生产环境中建议对`Broker`和`Governance`进行多实例部署。各子模块的部署细节参见[Broker模块部署](./module/broker.html)和[Governance模块部署](./module/governance.html)。
  