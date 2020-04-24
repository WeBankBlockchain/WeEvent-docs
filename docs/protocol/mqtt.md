## MQTT
`WeEvent`服务支持`MQTT Broker`功能。任何支持`MQTT`协议的IoT设备及客户端都能连接到`WeEvent`,并进行消息发布及订阅。

### 协议介绍

- `MQTT`是物联网`IoT`中的主流接入协议，协议具体内容参见[http://mqtt.org/](http://mqtt.org/) 。
- `WeEvent`支持`MQTT 3.1.1`
### 配置MQTT服务

 在`Broker`服务中，修改配置文件`./conf/weevent.properties`，然后重新启动服务。

```ini
#mqtt brokerserver
mqtt.broker.port=7001
mqtt.broker.keepalive=60
mqtt.websocket.path=/weevent-broker/mqtt
```

参数说明：

- mqtt.broker.port

  客户端使用`WebSocket`协议访问`MQTT Broker`。

- mqtt.broker.keepalive

  发送心跳时间。单位为秒。

- mqtt.websocket.path

  客户端使用`WebSocket`协议访问`MQTT Broker`链接。



### 注意事项
```eval_rst
.. important::
    - 区块链必须确保消息成功上链，暂不支持QoS=2的消息级别。
    - 不支持断连后会话恢复功能。
```
### 样例演示

样例演示需依赖`Mosquitto`客户端，请根据链接(`https://mosquitto.org/download/`)进行下载安装。更多客户端参见[MQTT第三方库](https://github.com/mqtt/mqtt.github.io/wiki/libraries)。

- IoT设备发布事件

  发送消息前需创建`topic` (`com.weevent.test`)。详情请参照[创建Topic](./restful.html)

  ```shell
  $ mosquitto_pub -h localhost -p 7000 -q 1 -t "com.weevent.test" -m "{\"timestamp\":133345566,\"key\":\"temperature\",\"value\":10.0}"
  ```

- IoT设备订阅事件

  ```shell
  $ mosquitto_sub -h localhost -p 7000 -q 1 -t "com.weevent.test"
  {"eventId":"317e7c4c-1-24","extensions":{},"topic":"com.weevent.test","content":[123,34,116,105,109,101,115,116,97,109,112,34,58,49,51,51,51,52,53,53,54,54,44,34,107,101,121,34,58,34,116,101,109,112,101,114,97,116,117,114,101,34,44,34,118,97,108,117,101,34,58,49,48,46,48,125]}
  ```