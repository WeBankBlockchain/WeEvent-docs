## 配置说明
### Broker 配置

主要有三类配置，`Spring Boot `进程配置、区块链`FISCO-BCOS`节点配置、`WeEvent`服务配置。

#### Spring Boot进程配置

配置文件链接[application-prod.properties](https://github.com/WeBankFinTech/WeEvent/blob/v0.9.0/src/main/resources/application-prod.properties) 。

```ini
#web container
server.port=8080
server.servlet.context-path=/weevent
#https
server.ssl.enabled=true
server.ssl.key-store=classpath:server.p12
server.ssl.key-store-password=123456
server.ssl.keyStoreType=PKCS12
server.ssl.keyAlias=weevent
#force to utf8
server.tomcat.uri-encoding=UTF-8
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
spring.http.encoding.force=true
spring.messages.encoding=UTF-8
#change not found uri status from 404 to exception
spring.mvc.throw-exception-if-no-handler-found=true
spring.resources.add-mappings=false
#actuator
management.endpoints.web.exposure.include=info
management.endpoint.shutdown.enabled=false
#performance
spring.application.admin.enabled=false
```

以上是`Spring Boot`标准配置文件，一般不需要修改。细节请参见[Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#appendix) 。

#### 区块链`FISCO-BCOS`节点配置

配置文件链接[applicationContext.xml](https://github.com/WeBankFinTech/WeEvent/blob/v0.9.0/src/main/resources/applicationContext.xml) 。

这个就是`FISCO-BCOS`的`Web3SDK`的配置文件，直接从节点上复制过来即可，不用修改。参见[Web3SDK配置文件](https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-1.3/docs/web3sdk/index.html) 。

#### WeEvent服务配置

配置文件链接[weevent.properties](https://github.com/WeBankFinTech/WeEvent/blob/v0.9.0/src/main/resources/weevent.properties) 。

```ini
#fisco
fisco.topic-controller.contract-address=0xddddd42da68a40784f5f63ada7ead9b36a38d2e3
fisco.consumer.idle-time=1000

#ip white list
ip.check.white-table=
#redis server
redis.server.ip=
redis.server.port=
redis.server.password=weevent
#lru.cache.capacity=65536
#restful cgi timeout
restful.subscribe.callback.timeout=5000

#mqtt broker
#mqtt.broker.url=tcp://127.0.0.1:1883
mqtt.broker.user=iot
mqtt.broker.password=123456
mqtt.broker.qos=2
mqtt.broker.timeout=5000
#mosquitto default 20s
mqtt.broker.keep-alive=15

#zookeeper
#broker.zookeeper.ip=127.0.0.1:2181
broker.zookeeper.path=/event_broker
broker.zookeeper.timeout=3000

#user
stomp.user.login=
stomp.user.passcode=
#server heartbeats
stomp.heartbeats=30
```

参数说明：
- 合约地址fisco.topic-controller.contract-address

   合约地址是`WeEvent`访问区块链访问数据的入口，需要用户在初始化`WeEvent`时部署合约并且修改。`Broker`安装包里带了部署合约的脚本`./deploy-topic-control.sh`。


- fisco.consumer.idle-time

    消费者线程中检测区块链新增块事件的周期，默认为1000毫秒。一般不用修改。

- IP白名单ip.check.white-table

   `WeEvent`授权访问的客户端`IP`列表 ，默认为空时表示允许任何客户端访问。

   用户可以配置多个`IP`地址，以";"进行分割。如`ip.check.white-table=127.0.0.1;127.0.0.2`。 

- Redis缓存配置redis.*

   业务场景中如果有多个`Consumer`订阅同一个`Topic`的场景，建议使用`Redis`缓存来加快事件的通知。  

   - redis.server.ip ： `Redis` 服务的IP地址
   - redis.server.port ：`Redis` 服务占用的端口
   - redis.server.password ：`Redis` 服务访问密码
   - lru.cache.capacity：进程中缓存大小，当缓存数据大于这个值时，使用`LRU`策略淘汰。一般不用修改。

- RESTful回调接口的超时时间

   - restful.subscribe.callback.timeout：事件通知回调的超时时间，默认为5000毫秒。一般不用修改。

- MQTT桥接配置mqtt.broker.*

   - mqtt.broker.url：`MQTT`服务的访问地址（一般是指`Mosquitto`的访问地址）。
   - mqtt.broker.user/password：`MQTT`服务的访问权限（没有可以不用填）。

- Zookeeper配置broker.zookeeper.*

   - broker.zookeeper.ip：`Zookeeper`的服务IP列表。
   - broker.zookeeper.path：`WeEvent`的数据存储路径。一般不用修改。
   - broker.zookeeper.timeout：`Zookeeper`的`Session`超时时间。默认为3000毫秒，一般不用修改。

- STOMP协议配置stomp.*

   - stomp.user.login/passcode：建议用户开启`STOMP`协议的账号/密码校验，以增强安全性。默认为空，表示不校验。
   - stomp.heartbeats：配置心跳时间间隔。默认时间间隔30秒，一般不用修改。

### Governance
`Governance`的配置都在文件`application-prod.yml `中，配置文件链接[application-prod.yml](https://github.com/WeBankFinTech/WeEvent-governance/blob/v0.9.0/src/main/resources/application-prod.yml) 。

```nginx
server:
  port: 8082
spring:
  # datasource
  datasource: 
    # mysql的jdbc访问串
    url: jdbc:mysql://127.0.0.1:3306/demo?useUnicode=true&characterEncoding=utf-8&useSSL=false 
    driver-class-name: org.mariadb.jdbc.Driver
    # 数据库用户名
    username: ${username}
    # 数据库密码
    password: ${password}
    type: org.apache.commons.dbcp2.BasicDataSource
    dbcp2:
      max-wait-millis: 10000
      min-idle: 5
      initial-size: 5
      validation-query: SELECT 'x'
  pid:
    fail-on-write-error: true
    file: ./logs/governance.pid

weevent:
  # broker服务的url地址
  url: http://127.0.0.1:8080/weevent
governance:
  influxdb:
    # 如果有配influxdb 填true，如果没有填写false
    enabled: false
    # 用户名
    username: admin
    # 密码
    password: admin
    # influxdb的url
    openurl: http://127.0.0.1:8306
    # 使用的数据库
    database: telegraf
logging:
  config: classpath:log4j2.xml
```
配置说明:
- datasource：数据库的`JDBC`访问串。
- weevent：`broker`服务的访问地址。


### Nginx 配置说明
#### 反向代理映射

这个文件`./conf/conf.d/http.conf `主要配置反向代理的映射，一般不需修改。配置文件链接[http.conf](https://github.com/WeBankFinTech/WeEvent-build/blob/v0.9.0/modules/nginx/conf/conf.d/http.conf) 。

```nginx
$ cat ./conf/conf.d/http.conf 
add_header X-Frame-Options "SAMEORIGIN";

server {
    listen          8888;
    server_name     localhost;

    location / {
        root   html;
        index  index.html index.htm;
    }

    location /weevent/ {
        proxy_pass          http://broker_backend/weevent/;
        
        proxy_set_header    Host $host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version  1.1;
        
        proxy_set_header    Upgrade $http_upgrade;
        proxy_set_header    Connection "upgrade";
    }
    
    location /weevent-governance/ {
        proxy_pass          http://governance_backend/weevent-governance/;
        
        proxy_set_header    Host $host:8080;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version  1.1;
    }
}
```

#### 后端子模块配置

配置文件链接[rs.conf](https://github.com/WeBankFinTech/WeEvent-build/blob/v0.9.0/modules/nginx/conf/conf.d/rs.conf) 。

```shell
$ cat ./conf/conf.d/rs.conf 
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
```
`Nginx`配置文件说明，请参见[Nginx配置](https://www.nginx.com/resources/wiki/start/topics/examples/full/) 。