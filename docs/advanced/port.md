## 服务访问

### 默认端口映射

![](../image/port.png)

### 更多说明

- 业务程序统一从前置的API Gateway接入，通过端口8080访问。

  `RESTful`协议的访问`URL`为 `http://localhost:8080/weevent-broker/rest`。

  `JsonRPC`协议的访问`URL`为 `http://localhost:8080/weevent-broker/jsonrpc`。

  `STOMP`协议基于`WebSocket`的 访问`URL`为`ws://localhost:8080/weevent-broker/stomp`，如果是基于`SockJS`则为`http://localhost:8080/weevent-broker/sockjs`。

  `MQTT`协议基于`WebSocket`的访问`URL`为`ws://localhost:8080/weevent-broker/mqtt`。

- API Gateway也支持HTTPS访问，端口为443。

  客户端无论使用`HTTP`还是`HTTPS`访问，`API Gateway`都是通过`HTTP`来访问后端服务。

- 以上说明全是指默认端口，  所有端口都支持自定义配置

