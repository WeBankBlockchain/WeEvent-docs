## Nginx模块

`WeEvent`通过`Nginx`实现负载均衡。

`WeEvent`服务支持主流的`Linux`发行版，如`CentOS`、`Ubuntu` 、`Redhat`。本节介绍`WeEvent`服务`Nginx`模块的详细安装步骤。`WeEvent`服务的快速安装请参见[WeEvent快速安装](../quickinstall.html) 。

以下安装以`CentOS` 7.2为例。

### 安装Nginx

下载安装包[weevent-nginx安装包](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-nginx-1.0.0.tar.gz)，解压到`/usr/local/weevent/`下。 

```shell
$ cd /usr/local/weevent/
$ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.0.0/weevent-nginx-1.0.0.tar.gz
$ tar -zxf weevent-nginx-1.0.0.tar.gz
```

解压后目录结构如下：

```shell
$ cd ./weevent-nginx-1.0.0
$ tree -L 2
.
|-- build-nginx.sh
|-- conf
|   |-- server.crt
|   |-- server.key
|   |-- conf.d
|   `-- nginx.conf
|-- nginx.sh
`-- third-packages
    |-- nginx-1.14.2.tar.gz
    `-- pcre-8.20.tar.gz
```

编译安装`Nginx`，操作如下

```shell
$ ./build-nginx.sh -p /user/local/nginx
build & install pcre
build & install nginx
nginx install complete!
```

### Nginx常见配置

- 修改Nginx监听端口

  在配置文件`./conf/conf.d/http.conf`里修改`Nginx`监听端口。其他配置项一般不需要修改。

  ```nginx
  server {
    	#配置nginx的监听端口
      listen          8080;
      server_name     localhost;
    
      ...
  }
  ```

- 修改WeEvent子模块反向代理

  在配置文件`./conf/conf.d/http_rs.conf` 里设置子模块`Broker`和`Governance`的实例。

  每增加一个实例，在这个配置文件里对应加一行`server`项，比如：

  ```nginx
  upstream broker_backend{
      server 1.1.1.1:8090 weight=100 max_fails=3;
      server 2.2.2.2:8090 weight=100 max_fails=3;
      
      ip_hash;
      keepalive 1024;
  }
  
  upstream broker_mqtt_websocket_backend {
      server 1.1.1.1:8092 weight=100 max_fails=3;
      server 2.2.2.2:8092 weight=100 max_fails=3;
  	ip_hash;
    	keepalive 1024;
  }
  
  upstream governance_backend{
  	server 1.1.1.1:8099 weight=100 max_fails=3;
  	server 2.2.2.2:8099 weight=100 max_fails=3;
  	ip_hash;
   	keepalive 1024;
  }
  ```
  
### Nginx配置TLS访问

- 切换到TLS访问
  
  默认的配置支持`HTTP`和`TCP`访问。通过`./conf/nginx.conf`文件配置`TLS`访问，将
  
  ```nginx
  include                 ./conf.d/http.conf;
  ...
  include                 ./conf.d/tcp.conf;
  ```
  
  对应改成
  
  ```nginx
  include                 ./conf.d/http_rs.conf;
...
  include                 ./conf.d/tcp_rs.conf;
  ```
  
- 修改Nginx监听端口
  
  在配置文件`./conf/conf.d/https.conf`里修改`Nginx`监听端口。其他配置项一般不需要修改。
  
  ```nginx
  nginx
  	server {
  		listen          443 ssl;
  		server_name     localhost;
  		...
  }
  ```
  
- 修改WeEvent子模块反向代理

  和上面`HTTP`访问的修改步骤一致。

### 服务启停

通过`./nginx.sh start`命令启动`Nginx`，正常启动如下：

```shell
$ ./nginx.sh start
start nginx success (PID=3643)
add the crontab job success
```

通过`./nginx.sh stop`停止`Nginx`。

进程启动后，会自动添加`crontab`监控任务`./broker.sh monitor`。

