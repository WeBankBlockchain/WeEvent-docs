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
- Spring对STOMP的支持需要依赖，以`gradle` 为例：  

  ```groovy
  implementation("org.springframework.boot:spring-boot-starter-websocket")
  ```

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

心跳说明

- `WeEvent`使用单向心跳机制，客户端发送心跳，服务端不发心跳。默认时间间隔为`30s` 。

- 修改心跳方案。

  配置心跳时间间隔：修改配置文件`./broker/conf/weevent.properties`，`stomp.heartbeats=30`。

**第二步：发布事件**

```java
StompHeaders header = new StompHeaders();
header.setDestination("com.weevent.test");
header.set("groupId","1");
header.set("weevent-eventId", "2-1");
header.set("weevent-url","https://github.com/WeBankFinTech/WeEvent")
StompSession.Receiptable receiptable = stompSession.send(header, "hello world, from web socket");
log.info("send result, receipt id: {}", receiptable.getReceiptId());
```

`Topic` 为`com.weevent.test`。用户可以获取到`Receiptable`，并且通过`receiptable.getReceiptId()`，可以获取相应的回执。

`groupId`为群主`Id`，`fisco-bcos 2.0+`版本支持多群主功能，2.0以下版本不支持该功能可以不传。

`weevent-url`为用户自定义拓展默认以`weevent-`开头。可选参数。

**第三步：订阅事件**

```java
StompHeaders header = new StompHeaders();
header.setDestination("com.weevent.test");
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

说明

- `topic`  订阅的主题
- `StompFrameHandler`  ，对`StompFrame`和`StompHeaders`进行处理的方法。
- `groupId` 群主`Id`，`fisco-bcos 2.0+`版本支持多群主功能，2.0以下版本不支持该功能可以不传。


**订阅事件扩展**

- 通过修改配置，进行`header`扩展。
- 配置`eventId`，提高订阅的效率。如果不设置，则默认为取最新内容。

```java
    StompHeaders header = new StompHeaders();
    header.setLogin("root");
    header.setPasscode("123456");
    header.set("eventId","2cf24dba-59-1124");
	header.set("groupId","1");
    header.setDestination(topic);

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


上述样例完整的代码，请参见[STOMP代码样例](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-broker/src/test/java/com/webank/weevent/sample/Stomp.java) 。

#### Spring环境

- 依赖说明
- 代码实现和上面`spring boot`一样

#### 其他语言的适配

各种语言的开源STOMP客户端，参见[https://stomp.github.io/implementations.html](https://stomp.github.io/implementations.html)。

