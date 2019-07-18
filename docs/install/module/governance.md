## Governance模块
`Governance`为用户提供一个事件治理的`Web`管理端。支持事件治理、区块链节点分析、系统监控预警等。

本节介绍`Governance`子模块的详细安装步骤，部署系统之前请确认[系统要求](../environment.html) 。 `WeEvent`服务的快速安装请参见[快速安装](../quickinstall.html) 。

以下安装以`CentOS 7.2`为例。

### 前置条件

- Broker模块

   必选配置，通过`Broker`访问区块链。

   具体安装步骤，请参见[Broker模块安装](./broker.html)。

- WeBase模块

  必选配置，通过`WeBase`查询区块和交易信息。

  推荐版本1.0.4。具体安装步骤，请参见[WeBase安装](https://webasedoc.readthedocs.io/zh_CN/latest/docs/WeBASE/install.html)。

  - 注意
    - 由于WeEvent和WeBase端口冲突，需修改WeBase一键部署源码包中的`common.properties`文件，配置已有的区块链和服务端口，具体如下。
    ```
        mgr.port=8182
        front.port=8181

        node.p2pPort=30300
        node.channelPort=20200
        node.rpcPort=8545

        if.exist.fisco=yes
    ```
    - 需要修改`webase-node-mgr`服务中的`conf/application.yml`文件。将`isUseSecurity`和`isDeleteInfo`都改成`false`。

- Mysql数据库

  必选配置。`Governance`通过`Mysql`存储统计数据。

  推荐安装`Mysql` 5.7+版本。具体安装步骤，安装请参见[Mysql安装](http://dev.mysql.com/downloads/mysql/) 。


### 获取安装包

下载安装包[weevent-governance-1.0.0.tar.gz](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-governance-1.0.0.tar.gz)，并且解压到`/usr/local/weevent/`下。

```shell
$ cd /usr/local/weevent/
$ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-governance-1.0.0.tar.gz
$ tar -xvf weevent-governance-1.0.0.tar.gz
```

解压后的目录结构如下

```
$ cd ./weevent-governance-1.0.0
$ tree -L 2
|-- apps
|   `-- weevent-governance-1.0.0.jar
|-- check-service.sh
|-- conf
|   |-- application-prod.yml
|   |-- application.yml
|   |-- banner.txt
|   |-- log4j2.xml
|   |-- mappers
|   `-- server.p12
|-- governance.sh
|-- html
|   |-- index.html
|   |-- README
|   `-- static
|-- init-governance.sh
```

### 修改配置文件

- 配置端口

  在配置文件`./conf/application-prod.yml`中，`Governance` 的服务端口`server.port` ，默认`8099`。

  ```
  server:
    port: 8099
  ```


- 配置Mysql数据库

    在配置文件`./conf/application-prod.yml`中，修改`datasource`中的`url`配置、`username`、`password` 。

    ```ini
    spring:  
      datasource:
        url: jdbc:mysql://127.0.0.1:3306/governance?useUnicode=true&characterEncoding=utf-8&useSSL=false
        driver-class-name: org.mariadb.jdbc.Driver
        username: xxxx
        password: yyyy
        type: org.apache.commons.dbcp2.BasicDataSource
    ```
    初始化系统，执行脚本`init-governance.sh` ，成功输出如下。否则，用户需要检查配置项是否正常。

    ```
    $ ./init-governance.sh
    init governance db success
    ```

    **注意**：数据库要赋予该角色通过其他机器进行数据库表增删操作的权限。

- 配置发送邮箱的地址

    在配置文件`./conf/application-prod.yml`中，修改`mail`中的`host`、`username`、`password` 配置。

    ```ini
    mail:
        default-encoding: UTF-8
        # smtp服务器地址 ex: smtp.163.com,smtp.qq.com,smtp.sohu.com
        host: smtp.163.com
        username: mailusername@163.com
        password: mailpwd
    ```

### 服务启停

- 服务启动
  通过`./governance.sh start`命令启动服务，正常启动如下：

  ```shell
  $ ./governance.sh start
  start weevent-governance success (PID=53926)
  add the crontab job success
  ```

  通过`./governance.sh stop`命令停止服务。

  进程启动后，会自动添加`crontab`监控任务`./governance.sh monitor`。

- 验证服务

  通过`./check-service.sh` 命令检查服务功能是否正常。

  ```shell
  $ ./check-service.sh
  check governance service
  governance service is ok
  ```

### 加入Nginx反向代理

将部署好的`Governance`配置到`Nginx`对外提供服务。`Nginx`子模块的安装及详细配置参见[Nginx模块安装及配置](./nginx.html) 。

如果需要部署更多实例，将上述步骤安装好的`Governance `目录拷贝到目标位置，启动即可。

用户可以通过浏览器访问http://localhost:8080/weevent-governance/。`Governance`页面如下：

![](../../image/Governance-ui.png)



