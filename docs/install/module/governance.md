## Governance模块
本节介绍`Governance`子模块的详细安装步骤。 `WeEvent`服务的快速安装请参见[快速安装](../quickinstall.html) 。在一台机器上详细安装，和通过快速安装然后把目标路径中的`governance`子目录打包拷贝到这台机器，效果是一样的。

`Governance`为用户提供一个事件治理的`Web`管理端。支持事件治理、区块链节点分析、系统监控预警等。

如果是第一次安装`WeEvent`，参见这里的[系统要求](../environment.html) 。以下安装以`CentOS 7.2`为例。

### 前置条件

- Broker模块

   必选配置，通过`Broker`访问区块链。

   具体安装步骤，请参见[Broker模块安装](./broker.html)。
   
- 数据库 `Governance`通过数据库存储数据。
    必选配置 目前支持H2数据库和Mysql数据库，二选一即可
    - H2数据库  
       默认配置，可切换成Msql数据库,具体切换步骤，请参考[FAQ](https://weeventdoc.readthedocs.io/zh_CN/latest/faq/weevent.html)。
      
       具体使用请参考[H2官网](http://www.h2database.com/html/main.html) 。

    - Mysql数据库    
      推荐安装`Mysql 5.7+`版本。具体安装步骤，安装请参见[Mysql安装](http://dev.mysql.com/downloads/mysql/) 。

- Processor模块
  可选配置。通过`Processor`触发规则引擎。
  具体安装步骤，请参见[Processor模块安装](./processor.html)。


### 获取安装包

从`github`下载安装包[weevent-governance-1.1.0.tar.gz](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.1.0/weevent-governance-1.1.0.tar.gz)，并且解压到`/usr/local/weevent/`下。

```shell
$ cd /usr/local/weevent/
$ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-governance-1.1.0.tar.gz
$ tar -xvf weevent-governance-1.1.0.tar.gz
```

如果`github`下载速度慢，可以尝试[国内下载链接](https://www.fisco.com.cn/cdn/weevent/download/releases/v1.1.0/weevent-governance-1.1.0.tar.gz)。

解压后的目录结构如下

```
$ cd ./weevent-governance-1.1.0
$ tree -L 2
|-- apps
|   `-- weevent-governance-1.1.0.jar
|-- check-service.sh
|-- conf
|   |-- application-prod.properties
|   |-- application.properties
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
|-- lib
```

### 修改配置文件

- 配置端口

  在配置文件`./conf/application-prod.properties`中，`Governance` 的服务端口`server.port` ，默认`7009`。

  ```
  server.port=7009
  ```

- 配置Processor访问路径

    初始化系统，执行脚本`init-governance.sh` ，成功输出如下。否则，用户需要检查配置项是否正常。

    ```shell
    $ ./init-governance.sh
    init governance db success
    ```

- 重置密码的邮件设置

    可选配置。在配置文件`./conf/application-prod.properties`中，修改`mail`中的`host`、`username`、`password` 配置。

    ```ini
	spring.mail.default-encoding= UTF-8
	spring.mail.host= smtp.163.com
	spring.mail.username= mailusername@163.com
	spring.mail.password= mailpwd
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

用户可以通过浏览器访问`http://localhost:8080/weevent-governance/` 默认用户名密码为：`admin/123456` 。显示如下登陆页面说明安装成功。

![Governance-ui.png](../../image/Governance-ui.png)

### 多视图管理

`Governance`支持同时管理多个`WeEvent`服务和区块链网络， 配置界面如下。

![Governance-multi-view.png](../../image/Governance-multi-view.png)


### 其他
推荐安装`Processor`。具体安装步骤，请参见[Processor安装](https://weeventdoc.readthedocs.io/en/latest/install/module/processor.html)。

