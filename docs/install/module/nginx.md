## Nginx模块

本节介绍`WeEvent`服务`Nginx`模块的详细安装步骤。`WeEvent`服务的快速安装请参见[WeEvent快速安装](../quickinstall.html) 。在一台机器上详细安装，和通过快速安装然后把目标路径中的`nginx`子目录打包拷贝到这台机器，效果是一样的。

`WeEvent`默认使用`Nginx`实现负载均衡。也可以换成`H5`等其他负载均衡服务。端口映射关系详见[服务访问](../../advanced/port.html)。

如果是第一次安装`WeEvent`，参见这里的[系统要求](../environment.html) 。以下安装以`CentOS 7.2`为例。

### 安装Nginx

对`Nginx`版本没有特别的要求。`HTTPS`访问时需要`--with-http_ssl_module`，`MQTT over TCP`访问时需要`--with-stream --with-stream_ssl_module`。如下：

```bash
$ ./nginx -V
nginx version: nginx/1.14.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --with-http_ssl_module --with-stream --with-stream_ssl_module
```

然后将`WeEvent`的[Nginx配置文件](https://github.com/WeBankFinTech/WeEvent/tree/master/weevent-build/modules/nginx/conf)，覆盖`./nginx/conf/`目录即可。另外：快速安装包`weevent-1.0.0.tar.gz`的`./modules/nginx/conf`目录中也包含这些配置文件。

```
$ tree
`-- conf
    |-- conf.d
    |   |-- http.conf
    |   |-- http_rs.conf
    |   |-- https.conf
    |   |-- tcp.conf
    |   |-- tcp_rs.conf
    |   `-- tcp_tls.conf
    |-- nginx.conf
    |-- scgi_params
    |-- server.crt
    |-- server.key
    `-- server.pem
```

### Nginx常见配置

- 修改Nginx监听端口

  在配置文件`./conf/conf.d/http.conf`里修改`Nginx`监听端口。其他配置项一般不需要修改。

  ```nginx
  server {
    	# 配置nginx的监听端口
      listen          8080; 
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
  
  对应修改成
  
  ```nginx
  include                 ./conf.d/https.conf;
  ...
  include                 ./conf.d/tcp_tls.conf;
  ```
- 修改Nginx监听端口
  
  在配置文件`./conf/conf.d/https.conf`里修改`Nginx`监听端口。其他配置项一般不需要修改。
  
  ```nginx
  server {
      listen          443 ssl;
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

通过`./nginx.sh stop`命令停止。

进程启动后，会自动添加`crontab`监控任务`./nginx.sh monitor`。

