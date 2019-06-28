## Governance模块
`Governance`为用户提供一个事件治理的`Web`管理端。支持事件治理、区块链节点分析、系统监控预警等。

本节介绍`Governance`子模块的详细安装步骤，部署系统之前请确认[系统要求](../environment.html) 。 `WeEvent`服务的快速安装请参见[快速安装](../quickinstall.html) 。

以下安装以`CentOS 7.2`为例。

### 前置条件

- Broker模块

   必选配置，通过`Broker`访问区块链。
   版本1.0.0。具体安装步骤，请参见[Broker模块安装](./broker.html)。

- WeBase模块

  必选配置，通过`WeBase`存取区块链的数据。

  版本为1.0及以上。具体安装步骤，请参见[WeBase安装](https://github.com/WeBankFinTech/WeBase)。

  **注意**：

  - 修改`webase-node-mgr`服务中的`conf/application.yml`文件。

    将`isUseSecurity`配置成`false`，`isDeleteInfo`配置成`false`


- Mysql数据库

  必选配置。`Governance`通过`Mysql`存储统计数据。

  推荐安装`Mysql` 5.7+版本。具体安装步骤，安装请参见[Mysql安装](http://dev.mysql.com/downloads/mysql/) 。


### 获取安装包

下载安装包[weevent-governance安装包](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-governance-1.0.0.tar.gz
)，并且解压到`/usr/local/weevent/`下。

```shell
$ cd /usr/local/weevent/
$ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-governance-1.0.0.tar.gz
$ tar -xvf weevent-governance-1.0.0.tar.gz
```

如果机器无法访问外网`wget`执行失败，可以通过别的方式下载再`rz`上传。

解压后的目录结构如下

```
$ cd ./weevent-governance-1.0.0
$ tree -L 2
|-- apps
|   `-- weevent-governance-1.0.0.jar
|-- check-service.sh
|-- conf
|   |-- application-dev.yml
|   |-- application-prod.yml
|   |-- application.yml
|   |-- log4j2.xml
|   |-- mappers
|   |-- server.p12
|   `-- sso.client.properties
|-- gen-cert-key.sh
|-- governance.sh
|-- html
|   |-- index.html
|   `-- static
`-- init-governance.sh
```

### 修改配置文件

- 配置端口

  在配置文件`./conf/application-prod.yml`中，`Governance` 的服务端口`server:port` ,默认`8082`。

  ```
  server:
    port: ${server_port}
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

- 生成HTTPS证书

  通过`HTTPS`方式访问`Governance`服务，需要配置一个访问证书。服务采用[PKCS12格式](https://tools.ietf.org/html/rfc7292) 。

  安装包里自带了默认证书`./conf/server.p12` ，可以直接使用。用户也可以选择使用包里的`./gen-cert-key.sh`脚本重新生成证书。使用新证书过程如下：

  - 生成证书

    ```shell	
    $ ./gen-cert-key.sh
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
    ```

    根据提示填写证书主题信息和密码，生成的证书`server.p12`已更新到`./weevent-governance-1.0.0/conf/server.p12`。

  - 修改证书配置

    参见`./conf/application.yml`中`ssl`配置项  

    ```ini
      ssl: 
        enabled: true
        key-store: classpath:server.p12
        key-store-password: 123456
        keyStoreType: PKCS12
        keyAlias: weevent
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

  `./governance.sh start`命令会启动进程，并且将进程监控命令`./governance.sh monitor`添加到`crontab`里。

  `./governance.sh stop`命令在进程成功停止后会移除`crontab`监控任务。

- 验证服务

  通过`./check-service.sh` 命令检查服务功能是否正常。

  ```shell
  $ ./check-service.sh
  check governance service
  governance service is ok
  ```

### 加入Nginx反向代理

`WeEvent`服务的所有请求都通过`Nginx`模块接入，`Nginx`模块的安装参见[Nginx模块安装](./nginx.html) 。

如果需要部署多个进程实例，将上述步骤安装好的`Governance `目录打包拷贝到其他机器上，解压启动即可。

`Nginx`配置文件`./conf/conf.d/rs.conf`中，以下为配置2个`Governance`进程的样例。

```nginx
upstream governance_backend{
    server 127.0.0.1:8082 weight=100 max_fails=3;
    server 127.0.0.2:8082 weight=100 max_fails=3;
  
    ip_hash;
    keepalive 1024;
}
```

`Nginx`配置文件`./conf/conf.d/rs.conf`中，以下为配置2个`webase-node-mgr`进程的样例

```nginx
upstream webase_backend{
    server 127.0.0.1:8083 weight=100 max_fails=3;
    server 127.0.0.2:8083 weight=100 max_fails=3;
  
    ip_hash;
    keepalive 1024;
}
```

通过`Nginx`重新加载配置后生效，进入`Nginx`安装路径，执行以下命令。

```
$ ./sbin/nginx -t
.../nginx/conf/nginx.conf syntax is ok
.../nginx/conf/nginx.conf test is successful
$ ./sbin/nginx -s reload
```

用户可以通过浏览器访问http://ip:8080/weevent-governance/。`Governance`页面如下：

![](../../image/Governance-ui.png)



