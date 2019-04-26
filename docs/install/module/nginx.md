## Nginx模块

`WeEvent`通过`Nginx`实现负载均衡。

`WeEvent`服务支持主流的`Linux`发行版，如`CentOS`、`Ubuntu` 、`Redhat`。本节介绍`WeEvent`服务`Nginx`模块的详细安装步骤。`WeEvent`服务的快速安装请参见[WeEvent快速安装](../quickinstall.html) 。

以下安装以`CentOS` 7.2为例。

### 安装Nginx

下载安装包[weevent-nginx安装包](https://github.com/WeBankFinTech/WeEvent/releases/download/v0.9.0/weevent-nginx-0.9.0.tar.gz)，解压到`/usr/local/weevent/`下。 

```shell
$ cd /usr/local/weevent/
$ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v0.9.0/weevent-nginx-0.9.0.tar.gz
$ tar -zxf weevent-nginx-0.9.0.tar.gz
```

如果机器无法访问外网`wget`执行失败，可以通过别的方式下载再`rz`上传。

解压后目录结构如下：

```shell
$ cd ./weevent-nginx-0.9.0
$ tree -L 2
.
|-- build-nginx.sh
|-- conf
|   |-- cert.key
|   |-- cert.pem
|   |-- conf.d
|   `-- nginx.conf
|-- nginx.sh
`-- third-packages
    |-- nginx-1.14.2.tar.gz
    `-- pcre-8.20.tar.gz

```

`WeEvent`既支持`HTTP`访问，也支持`HTTPS`访问，可通过配置文件切换。

安装`Nginx`，操作如下

```shell
$ ./build-nginx.sh -p /user/local/nginx
build & install pcre
build & install nginx
nginx install complete!
```

进入目录，进行相关配置进行修改。

#### Nginx配置HTTP访问

- 修改Nginx监听端口

  在配置文件`./conf.d/http.conf`里修改`Nginx`监听端口。其他配置项一般不需要修改。

  ```nginx
  server {
    	#配置nginx的监听端口
      listen          8080;
      server_name     localhost;
    
      ...
  }
  ```

- 修改WeEvent子模块反向代理

  在配置文件`./conf.d/rs.conf` 里设置子模块`Broker`和`Governance`的实例。

  每增加一个实例，在这个配置文件里对应加一行`server`项，比如：

  ```nginx
  upstream broker_backend{
      server 127.0.0.1:8081 weight=100 max_fails=3;
      server 127.0.0.2:8081 weight=100 max_fails=3;
    
      ip_hash;
      keepalive 1024;
  }

  upstream governance_backend{
      server 127.0.0.1:8082 weight=100 max_fails=3;
      server 127.0.0.2:8082 weight=100 max_fails=3;
    
      ip_hash;
      keepalive 1024;
  }

  upstream grafana_backend{
      server 127.0.0.1:3000 weight=100 max_fails=3;
      server 127.0.0.2:3000 weight=100 max_fails=3;
    
      ip_hash;
      keepalive 1024;
  }
  ```

#### Nginx配置HTTPS访问

- 修改Nginx监听端口

  在配置文件`./conf.d/https.conf`里修改`Nginx`监听端口。其他配置项一般不需要修改。

  ```nginx
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

`./nginx.sh start`命令会启动进程，并且将进程监控命令`./broker.sh monitor`添加到`crontab`里。

`./broker.sh stop`命令在进程成功停止后会移除`crontab`监控任务。