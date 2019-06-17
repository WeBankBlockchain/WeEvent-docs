## MQTT
`WeEvent`服务支持`MQTT Broker`功能。任何支持`MQTT`协议的IOT设备及客户端都能连接到`WeEvent`进行消息发布及订阅。

### 协议介绍

- `MQTT`是物联网`IoT`中的主流接入协议，协议具体内容参见[http://mqtt.org/](http://mqtt.org/) 
- `WeEvent`支持`MQTT 3.1.1`
### 配置MQTT服务

 在`Broker`服务中，修改配置文件`./conf/weevent.properties`，然后重新启动服务。

```ini
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

在Broker服务中修改配置文件。`./conf/application.properties`,然后重启服务。

```ini
#https
server.ssl.enabled=true
server.ssl.key-store=classpath:server.p12
server.ssl.key-store-password=123456
server.ssl.keyStoreType=PKCS12
server.ssl.keyAlias=weevent
```

参数说明：

- mqtt.brokerserver.port

  `mqtt`访问端口。

- mqtt.brokerserver.sobacklog

  服务器请求处理线程全满时，用于临时存放已完成`tcp`三次握手请求的队列的最大长度。

- mqtt.brokerserver.sokeepalive

  是否开启连接检测以此判断服务是否可用。

- mqtt.websocketserver.path

  `websocket`访问链接。

- mqtt.websocketserver.port

  `websocket`访问端口。

- mqtt.user.login

  `mqtt`访问用户名，为空则不校验用户名。

- mqtt.user.passcode

  `mqtt`访问用户密码，为空则不校验用户密码。

- server.ssl.enabled

  开启`https`功能。

- server.ssl.key-store

  证书文件路径。安装包里自带了默认证书`./conf/server.p12` ，可以直接使用。用户也可以选择使用包里的`./gen-cert-key.sh`脚本重新生成证书。

- server.ssl.key-store-password

  证书密码。

- server.ssl.keyAlias

  `key`别名。
### 注意事项

- 因区块链必须确保消息成功上链暂不支持QOS-0和QOS-2消息级别。

- 暂不支持断连后会话恢复功能。