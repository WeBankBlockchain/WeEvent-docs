## Java Client SDK
`WeEvent`支持`RESTful`、`JsonRPC`、`STOMP`、`MQTT`等协议，方便各种语言的接入和适配。

同时为`Java`语言提供了独立的客户端`SDK`。其他语言`SDK`在规划中，欢迎大家贡献代码，[WeEvent代码仓库](https://github.com/WeBankBlockchain/WeEvent) 。

### 集成SDK

- gradle依赖
```groovy
implement 'com.webank.weevent:weevent-client:1.6.0'
```
- maven依赖
```xml
<dependency>
    <groupId>com.webank.weevent</groupId>
    <artifactId>weevent-client</artifactId>
    <version>1.6.0</version>
</dependency>
```

### API接口
```java
public interface IWeEventClient {
    String defaultBrokerUrl = "http://localhost:8080/weevent-broker";

    /**
     * builder class
     */
    class Builder {
        // broker url
        private String brokerUrl = defaultBrokerUrl;
        // group id
        private String groupId = WeEvent.DEFAULT_GROUP_ID;
        // stomp's account&password
        private String userName = "";
        private String password = "";
        // rpc timeout, ms
        private int timeout = 5000;

        public Builder brokerUrl(String brokerUrl) {
            this.brokerUrl = brokerUrl;
            return this;
        }

        public Builder groupId(String groupId) {
            this.groupId = groupId;
            return this;
        }

        public Builder userName(String userName) {
            this.userName = userName;
            return this;
        }

        public Builder password(String password) {
            this.password = password;
            return this;
        }

        public Builder timeout(int timeout) {
            this.timeout = timeout;
            return this;
        }

        public IWeEventClient build() throws BrokerException {
            return new WeEventClient(this.brokerUrl, this.groupId, this.userName, this.password, this.timeout);
        }
    }

    /**
     * Open a topic
     *
     * @param topic topic name
     * @return true if success
     * @throws BrokerException broker exception
     */
    boolean open(String topic) throws BrokerException;

    /**
     * Close a topic.
     *
     * @param topic topic name
     * @return true if success
     * @throws BrokerException broker exception
     */
    boolean close(String topic) throws BrokerException;

    /**
     * Check a topic is exist or not.
     *
     * @param topic topic name
     * @return true if exist
     * @throws BrokerException broker exception
     */
    boolean exist(String topic) throws BrokerException;

    /**
     * Publish an event to topic.
     *
     * @param weEvent WeEvent(String topic, byte[] content, Map extensions)
     * @return send result, SendResult.SUCCESS if success, and SendResult.eventId
     * @throws BrokerException broker exception
     */
    SendResult publish(WeEvent weEvent) throws BrokerException;

    /**
     * Publish an event to topic in asynchronous way.
     *
     * @param weEvent WeEvent(String topic, byte[] content, Map extensions)
     * @return send result, SendResult.SUCCESS if success, and SendResult.eventId
     * @throws BrokerException broker exception
     */
    CompletableFuture<SendResult> publishAsync(WeEvent weEvent) throws BrokerException;

    /**
     * Interface for event notify callback
     */
    interface EventListener {
        /**
         * Called while new event arrived.
         *
         * @param event the event
         */
        void onEvent(WeEvent event);

        /**
         * Called while raise exception.
         *
         * @param e the e
         */
        void onException(Throwable e);
    }

    /**
     * Subscribe events from topic.
     *
     * @param topic topic name
     * @param offset from next event after this offset(an event id), WeEvent.OFFSET_FIRST if from head of queue, WeEvent.OFFSET_LAST if from tail of queue
     * @param extension extension params
     * @param listener notify interface
     * @return subscription Id
     * @throws BrokerException invalid input param
     */
    String subscribe(String topic, String offset, Map<String, String> extension, @NonNull EventListener listener) throws BrokerException;

    /**
     * Subscribe events from topic.
     *
     * @param topics topic list
     * @param offset from next event after this offset(an event id), WeEvent.OFFSET_FIRST if from head of queue, WeEvent.OFFSET_LAST if from tail of queue
     * @param extension extension params
     * @param listener notify interface
     * @return subscription Id
     * @throws BrokerException invalid input param
     */
    String subscribe(String[] topics, String offset, Map<String, String> extension, @NonNull EventListener listener) throws BrokerException;

    /**
     * Unsubscribe an exist subscription subscribed by subscribe interface.
     * The consumer will no longer receive messages from broker after this.
     *
     * @param subscriptionId invalid input
     * @return success if true
     * @throws BrokerException broker exception
     */
    boolean unSubscribe(String subscriptionId) throws BrokerException;

    /**
     * List all topics in WeEvent's broker.
     *
     * @param pageIndex page index, from 0
     * @param pageSize page size, [10, 100)
     * @return topic list
     * @throws BrokerException broker exception
     */
    TopicPage list(Integer pageIndex, Integer pageSize) throws BrokerException;

    /**
     * Get a topic information.
     *
     * @param topic topic name
     * @return topic information
     * @throws BrokerException broker exception
     */
    TopicInfo state(String topic) throws BrokerException;

    /**
     * Get an event information.
     *
     * @param eventId event id
     * @return WeEvent
     * @throws BrokerException broker exception
     */
    WeEvent getEvent(String eventId) throws BrokerException;
}
```

### 代码样例

```java
public static void main(String[] args) {
    try {
        IWeEventClient client = IWeEventClient.builder().brokerUrl("http://localhost:8080/weevent-broker").groupId(WeEvent.DEFAULT_GROUP_ID).build();
        
        String topicName = "com.weevent.test";
        // open 一个"com.weevent.test"的主题
        client.open(topicName);
        
        // 发送hello WeEvent
        WeEvent weEvent = new WeEvent(topicName, "hello WeEvent".getBytes());
        SendResult sendResult = client.publish(weEvent);
        System.out.println(sendResult);
    } catch (BrokerException e) {
        e.printStackTrace();
    }
}
```

完整的代码请参见[Java Client SDK代码样例](https://github.com/WeBankBlockchain/WeEvent/blob/master/weevent-broker/src/test/java/com/webank/weevent/broker/sample/JavaSDK.java) 。
