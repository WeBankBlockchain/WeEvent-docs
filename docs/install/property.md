## 配置说明
### Broker 配置

主要有三类配置，`Spring Boot `进程配置、区块链`FISCO-BCOS`节点配置、`WeEvent`服务配置。

#### Spring Boot进程配置

配置文件链接[application-prod.properties](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-broker/src/main/resources/application-prod.properties) 。

```ini
#web container
server.port=8081
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
配置文件链接[fisco.properties](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-broker/src/main/resources/fisco.properties) 。
```ini
#fisco
version=2.0
topic-controller.address=1:0x23df89a2893120f686a4aa03b41acf6836d11e5d;
#version=1.3
#topic-controller.address=0xddddd42da68a40784f5f63ada7ead9b36a38d2e3
orgid=fisco
nodes=127.0.0.1:30701
#account
account=bcec428d5205abe0f0cc8a734083908d9eb8563e31f943d760786edf42ad67dd
#web3sdk
web3sdk.timeout=10000
web3sdk.core-pool-size=100
web3sdk.max-pool-size=200
web3sdk.queue-capacity=1000
web3sdk.keep-alive-seconds=60
```

参数说明：

- version

  连接`FISCO-BCOS`节点的版本，支持1.3和2.0两个版本。

- topic-controller.address

  合约地址是`WeEvent`访问区块链访问数据的入口，需要用户在初始化`WeEvent`时部署合约并且修改。`Broker`安装包里带了部署合约的脚本`./deploy-topic-control.sh`。如果是`FISCO-BCOS`2.0，每个群组都有独立的地址，多个群组地址以`;`分割。

- orgid

  机构名，按机构实际名称填写即可。

- nodes

  区块链节点列表，多个地址以`;`分割。

- account

  `WeEvent`执行交易的账号，一般不需要修改。

- web3sdk.*

  `Web3SDK`连接池选项，一般不需要修改。

- 证书文件

  1.3版本的证书文件`ca.crt`和`client.keystore`放在`./conf`目录下。

  2.0版本的证书文件`ca.crt`和`node.crt`、`node.key`放在`./conf/v2`目录下。

区块链节点详细配置，参见[Web3SDK配置文件](https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-2.0/docs/sdk/sdk.html) 。

#### WeEvent服务配置

配置文件链接[weevent.properties](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-broker/src/main/resources/weevent.properties) 。

```ini
#consumer
consumer.idle-time=1000
#ip white list
ip.check.white-table=
#redis server
redis.server.ip=
redis.server.port=
redis.server.password=weevent
lru.cache.capacity=65536
#restful cgi timeout
restful.subscribe.callback.timeout=5000

#zookeeper
broker.zookeeper.ip=127.0.0.1:2181
broker.zookeeper.path=/event_broker
broker.zookeeper.timeout=3000

#user
stomp.user.login=
stomp.user.passcode=
#server heartbeats
stomp.heartbeats=30

#mqtt brokerserver
mqtt.brokerserver.port=8083
mqtt.brokerserver.sobacklog=511
mqtt.brokerserver.sokeepalive=true
mqtt.brokerserver.keepalive=60
mqtt.websocketserver.path=/weevent/mqtt
mqtt.websocketserver.port=8084
mqtt.user.login=
mqtt.user.passcode=
```

参数说明：
- consumer.idle-time

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

- Zookeeper配置broker.zookeeper.*

   - broker.zookeeper.ip：`Zookeeper`的服务IP列表。
   - broker.zookeeper.path：`WeEvent`的数据存储路径。一般不用修改。
   - broker.zookeeper.timeout：`Zookeeper`的`Session`超时时间。默认为3000毫秒，一般不用修改。

- STOMP协议配置stomp.*

   - stomp.user.login/passcode：建议用户开启`STOMP`协议的账号/密码校验，以增强安全性。默认为空，表示不校验。
   - stomp.heartbeats：配置心跳时间间隔。默认时间间隔30秒，一般不用修改。

- MQTT Broker配置mqtt.*

   - mqtt.brokerserver.port：客户端使用`MQTT`协议访问`MQTT Broker`端口。
   - mqtt.brokerserver.sobacklog：服务器请求处理线程全满时，用于临时存放已完成tcp三次握手请求的队列的最大长度。
   - mqtt.brokerserver.sokeepalive：是否开启连接检测以此判断服务是否可用。
   - mqtt.brokerserver.keepalive：是否开启连接检测以此判断服务是否可用。
   - mqtt.websocketserver.path：客户端使用`WebSocket`协议访问`MQTT Broker`链接。
   - mqtt.websocketserver.port：客户端使用`WebSocket`访问`MQTT Broker`端口。
   - mqtt.user.login：`MQTT Broker`访问用户名。
   - mqtt.user.passcode：`MQTT Broker`访问密码。
### Governance

`Governance`的配置都在文件`application-prod.yml `中，配置文件链接[application-prod.yml](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-governance/src/main/resources/application-prod.yml) 。

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
    mail:
        default-encoding: UTF-8
        host: smtp.163.com
        username: mailusername@163.com
        password: mailpwd
  pid:
    fail-on-write-error: true
    file: ./logs/governance.pid
http:
  client:
    max-total: 200
    max-per-route: 500
    connection-request-timeout: 3000
    connection-timeout: 3000
    socket-timeout: 5000
https: 
  read-timeout: 5000
  connect-timeout: 15000     


```
- 配置说明:
    - datasource：数据库的`JDBC`访问字符串。
    - mail.username：用于发送给用户发送邮件的邮件地址。

### Nginx 配置说明
#### 反向代理映射

`./conf/conf.d/http.conf `主要配置反向代理的映射，一般不需修改。配置文件链接[http.conf](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-build/modules/nginx/conf/conf.d/http.conf) 。

```nginx
add_header X-Frame-Options "SAMEORIGIN";

server {
    listen          8080;
    server_name     localhost;

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
        
        proxy_set_header    Host $http_host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version  1.1;
    }
    
    location /webase-node-mgr/ {
        proxy_pass          http://webase_backend/webase-node-mgr/;
        
        proxy_set_header    Host $host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version  1.1;
    }    
}

```

`./conf/conf.d/https.conf `主要配置反向代理的映射，一般不需修改。配置文件链接[https.conf](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-build/modules/nginx/conf/conf.d/https.conf) 。

```shell
add_header X-Frame-Options "SAMEORIGIN";

server {
    listen          443 ssl;
    server_name     localhost;

    ssl_certificate              cert.pem;
    ssl_certificate_key          cert.key;
    ssl_session_cache            shared:SSL:1m;
    ssl_session_timeout          5m;
    ssl_ciphers                  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers    on;

    location /weevent/ {
        proxy_pass          https://broker_backend/weevent/;
        
        proxy_set_header    Host $host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version  1.1;
        
        proxy_set_header    Upgrade $http_upgrade;
        proxy_set_header    Connection "upgrade";
    }
    
    location /weevent-governance/ {
        proxy_pass          https://governance_backend/weevent-governance/;
        
        proxy_set_header    Host $http_host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version  1.1;
    }
    
    location /webase-node-mgr/ {
        proxy_pass          http://webase_backend/webase-node-mgr/;
        
        proxy_set_header    Host $host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version  1.1;
    } 
}

```

#### 后端子模块配置

配置文件链接[rs.conf](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-build/modules/nginx/conf/conf.d/rs.conf) 。

```shell
upstream broker_backend{
    server localhost:8081 weight=100 max_fails=3;
    
    ip_hash;
    keepalive 1024;
}

upstream governance_backend{
    server localhost:8082 weight=100 max_fails=3;
    
    ip_hash;
    keepalive 1024;
}

upstream webase_backend{
    server localhost:8083 weight=100 max_fails=3;
    
    ip_hash;
    keepalive 1024;
}

```
`Nginx`配置文件说明，请参见[Nginx配置](https://www.nginx.com/resources/wiki/start/topics/examples/full/) 。