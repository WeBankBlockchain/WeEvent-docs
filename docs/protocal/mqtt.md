## MQTT
`WeEvent`服务支持`MQTT Broker`功能。任何支持`MQTT`协议的IoT设备及客户端都能连接到`WeEvent`,并进行消息发布及订阅。

### 协议介绍

- `MQTT`是物联网`IoT`中的主流接入协议，协议具体内容参见[http://mqtt.org/](http://mqtt.org/) 。
- `WeEvent`支持`MQTT 3.1.1`
### 配置MQTT服务

 在`Broker`服务中，修改配置文件`./conf/weevent.properties`，然后重新启动服务。

```ini
#mqtt brokerserver
mqtt.broker.port=8083
mqtt.broker.keepalive=60
mqtt.websocket.path=/weevent/mqtt
mqtt.websocket.port=8084
mqtt.user.login=
mqtt.user.passcode=
```

参数说明：

- mqtt.broker.port

  客户端使用`MQTT`协议访问`MQTT Broker`。

- mqtt.brokerserver.keepalive

  发送心跳时间。单位为秒。

- mqtt.websocketserver.path

  客户端使用`WebSocket`协议访问`MQTT Broker`链接。

- mqtt.websocketserver.port

  客户端使用`WebSocket`协议访问`MQTT Broker`。

- mqtt.user.login

  `mqtt`访问用户名，为空则不校验用户名。

- mqtt.user.passcode

  `mqtt`访问用户密码，为空则不校验用户密码。

### 注意事项

- 因区块链必须确保消息成功上链，暂不支持QoS-0和QoS-2消息级别。

- 不支持断连后会话恢复功能。

### 样例演示

样例演示需依赖`Mosquitto`客户端，请根据链接(`https://mosquitto.org/download/`)进行下载安装。

- IoT设备发布事件

  ```shell
  $ mosquitto_pub -h localhost -p 8081 -u ${user} -P ${password} -t "com.weevent.test" -m "{\"timestamp\":133345566,\"key\":\"temperature\",\"value\":10.0}"
  ```

- IoT设备订阅事件

  ```shell
  $ mosquitto_sub -h localhost -p 8081 -u ${user} -P ${password} -t "com.weevent.test"
  ```