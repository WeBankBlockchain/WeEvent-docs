## MQTT
`WeEvent`服务支持`MQTT Broker`功能。任何支持`MQTT`协议的IoT设备及客户端都能连接到`WeEvent`进行消息发布及订阅。

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

### 注意事项

- 因区块链必须确保消息成功上链暂不支持QOS-0和QOS-2消息级别。

- 暂不支持断连后会话恢复功能。

### 样例演示

样例演示需依赖`Mosquitto`客户端，请根据链接(`https://mosquitto.org/download/`)进行下载安装。

- IoT设备发布事件

  ```shell
  $ mosquitto_pub -h localhost -p 8083 -u ${user} -P ${password} -t "com.weevent.test" -m "{\"timestamp\":133345566,\"key\":\"temperature\",\"value\":10.0}"
  ```

- IoT设备订阅事件

  ```shell
  $ mosquitto_sub -h localhost -p 8083 -u ${user} -P ${password} -t "com.weevent.test"
  ```