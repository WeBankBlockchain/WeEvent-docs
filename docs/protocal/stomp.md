## STOMP
`STOMP`是面向流文本的消息传输协议`Streaming Text Oriented Messaging Protocol`，是 `WebSocket` 通信标准的一部分，属于业务层的控制协议`Wire protocol`，[协议规范](https://stomp.github.io/stomp-specification-1.1.html)。在通常的发布订阅语义之上，它通过 `begin`、 `commit`、`rollback` 事物序列以及`ack`确认机制来提供消息可靠的投递。由于协议简单且易于实现，几乎所有的编程语言都有 STOMP 的客户端实现。

可以通过`STOMP`协议访问`WeEvent`的发布订阅相关功能。

### 协议说明

- 支持STOMP协议的`1.1`、`1.2`版本。暂时不支持消息确认`ACK`和事务`Transaction`语义。 

- 传输协议方面，同时支持`STOMP Over WebSocket`和`STOMP Over SockJS`。

### JavaScript语言
前端面直接访问`WeEvent`，推荐使用开源库[stompjs](https://github.com/stomp-js/stompjs)，该库支持STOMP协议的`1.1`、`1.2`的版本。使用`stompjs` +` sockjs`的组合效果更好。

### Java语言
#### Spring Boot 环境
加入`Spring Boot`的依赖，以`gradle` 为例：  

```groovy
implementation("org.springframework.boot:spring-boot-starter-websocket")
```
`Spring`从4.0开始引入`spring-websocket`模块，支持`STOMP`，建议使用`Spring Boot` 2.0.0以上版本。

#### 代码样例

**第一步：创建链接**

```java
       // standard web socket transport
        WebSocketClient webSocketClient = new StandardWebSocketClient();
        WebSocketStompClient stompClient = new WebSocketStompClient(webSocketClient);

        // MappingJackson2MessageConverter
        stompClient.setMessageConverter(new StringMessageConverter());
        stompClient.setTaskScheduler(taskScheduler); // for heartbeats

        ListenableFuture<StompSession> f = stompClient.connect("ws://localhost:8080/weevent/stomp", getWebsocketSessionHandlerAdapter());

        StompSession stompSession = f.get();
```

- 心跳说明

 `WeEvent`使用单向心跳机制，客户端发送心跳，服务端不发心跳。默认时间间隔为`30s` 。

- 修改心跳方案。

  配置心跳时间间隔：修改配置文件`./broker/conf/weevent.properties`，`stomp.heartbeats=30`。
- 传输协议方面
    `STOMP Over WebSocket`使用[ws://localhost:8080/weevent/stomp](ws://localhost:8080/weevent/stomp)
    `STOMP Over SockJS`使用[ws://localhost:8080/weevent/sockjs](ws://localhost:8080/weevent/sockjs)

**第二步：发布事件**

```java
    StompHeaders header = new StompHeaders();
    header.setDestination("com.weevent.test");
    header.set("groupId","1");
    header.set("weevent-format", "json")
    StompSession.Receiptable receiptable = stompSession.send(header, "{\"hello\":\" wolrd\"}");
    log.info("send result, receipt id: {}", receiptable.getReceiptId());
```

说明：
- `Topic` 为`com.weevent.test`。用户可以获取到`Receiptable`，并且通过`receiptable.getReceiptId()`，可以获取相应的回执。
- `groupId`为群组`Id`，`fisco-bcos 2.0+`版本支持多群组功能，2.0以下版本不支持该功能可以不传。
- `weevent-format`为用户自定义拓展默认以`weevent-`开头。可选参数。

**第三步：订阅事件**

```java
    StompHeaders header = new StompHeaders();
    header.setDestination(topic);
    header.set("eventId","2cf24dba-59-1124");
    header.set("groupId","1");

    StompSession.Subscription subscription = stompSession.subscribe(header, new StompFrameHandler() {
        @Override
        public Type getPayloadType(StompHeaders headers) {
            return String.class;
        }

        @Override
        public void handleFrame(StompHeaders headers, Object payload) {
            logger.info("subscribe handleFrame, header: {} payload: {}", headers, payload);
        }
    });
```

说明：

- `topic`  订阅的主题。支持通配符按层次订阅，参见[MQTT通配符](http://public.dhe.ibm.com/software/dw/webservices/ws-mqtt/mqtt-v3r1.html) 。
- 配置`eventId`，如需要取历史数据，则需要设置。如果不设置，则默认为取最新内容。
- `weevent-format`为用户自定义拓展默认以`weevent-`开头。可选参数。
- `StompFrameHandler`  ，对`StompFrame`和`StompHeaders`进行处理的方法。 

上述样例完整的代码，请参见[STOMP代码样例](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-broker/src/test/java/com/webank/weevent/sample/Stomp.java) 。

#### Spring环境

- 依赖说明

```
implementation("org.springframework:spring-messaging:5.1.2.RELEASE")
implementation("org.springframework:spring-websocket:5.1.2.RELEASE")
```
- 代码实现和上面`spring boot`一样

#### 其他语言的适配

各种语言的开源STOMP客户端，参见[https://stomp.github.io/implementations.html](https://stomp.github.io/implementations.html)。

