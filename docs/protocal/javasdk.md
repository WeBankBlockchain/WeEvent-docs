## Java SDK
`WeEvent`支持`RESTful`、`JsonRPC`、`STOMP`、`MQTT`等协议，方便各种语言的接入和适配。

同时为`Java`语言提供了独立的`SDK`。其他语言`SDK`在规划中，欢迎大家贡献代码，[WeEvent代码仓库](https://github.com/WeBankFinTech/WeEvent) 。

### 集成SDK

- gradle依赖
```groovy
implement 'com.webank.weevent:weevent-client:0.9.0'
```
- maven依赖
```xml
<dependency>
    <groupId>com.webank.weevent</groupId>
    <artifactId>weevent-client</artifactId>
    <version>0.9.0</version>
</dependency>
```

### API接口
```java
public class WeEventClient {
    /**
     * Get the client handler of weevent's broker with default url, http://localhost:8080/weevent.
     *
     * @throws BrokerException broker exception
     */
    public WeEventClient() throws BrokerException;

    /**
     * Get the client handler of weevent's broker with custom url.
     *
     * @param brokerUrl weevent's broker url, like http://localhost:8080/weevent
     * @throws BrokerException broker exception
     */
    public WeEventClient(String brokerUrl) throws BrokerException;

    /**
     * Get the client handler of weevent's broker custom url and account authorization.
     *
     * @param brokerUrl weevent's broker url, like http://localhost:8080/weevent
     * @param userName account name
     * @param password password
     * @throws BrokerException broker exception
     */
    public WeEventClient(String brokerUrl, String userName, String password) throws BrokerException;

    /**
     * Publish an event to topic.
     *
     * @param topic topic name
     * @param content topic data
     * @return send result, SendResult.SUCCESS if success, and SendResult.eventId
     * @throws BrokerException broker exception
     */
    public SendResult publish(String topic, byte[] content) throws BrokerException;

    /**
     * Subscribe events from topic.
     *
     * @param topic topic name
     * @param offset, from next event after this offset(an event id), IConsumer.OFFSET_FIRST if from head of queue, IConsumer.OFFSET_LAST if from tail of queue
     * @param listener callback
     * @return subscription Id
     * @throws BrokerException invalid input param
     */
    public String subscribe(String topic, String offset, IConsumer.ConsumerListener listener);

    /**
     * Unsubscribe an exist subscription subscribed by {@link #subscribe(String, String, IConsumer.ConsumerListener)}.
     * The consumer will no longer receive messages from broker after this.
     *
     * @param subscriptionId invalid input
     * @return success if true
     * @throws BrokerException broker exception
     */
    public boolean unSubscribe(String subscriptionId) throws BrokerException;

    // The following is interface for IEventTopic.

    /**
     * Open a topic.
     *
     * @param topic topic name
     * @return true if success
     * @throws BrokerException broker exception
     */
    public boolean open(String topic) throws BrokerException;

    /**
     * Close a topic.
     *
     * @param topic topic name
     * @return true if success
     * @throws BrokerException broker exception
     */
    public boolean close(String topic) throws BrokerException;

    /**
     * Check a topic is exist or not.
     *
     * @param topic topic name
     * @return true if exist
     * @throws BrokerException broker exception
     */
    public boolean exist(String topic) throws BrokerException;
    }

    /**
     * List all topics in weevent's broker.
     *
     * @param pageIndex page index, from 0
     * @param pageSize page size, [10, 100)
     * @return topic list
     * @throws BrokerException broker exception
     */
    public TopicPage list(Integer pageIndex, Integer pageSize);

    /**
     * Get a topic information.
     *
     * @param topic topic name
     * @return topic information
     * @throws BrokerException broker exception
     */
    public TopicInfo state(String topic) throws BrokerException;

    /**
     * Get an event information.
     *
     * @param eventId event id
     * @return weevent
     * @throws BrokerException broker exception
     */
    public WeEvent getEvent(String eventId) throws BrokerException;
}
```

### 代码样例
```java
public static void main(String[] args) {
    try {
        String url = "http://localhost:8080/weevent";
        WeEventClient client = WeEventClient(url);
        //publish接口的参数分别是主题Topic、事件内容Content
        SendResult sendResult = client.publish("com.weevent.test", "hello wolrd".getBytes());
        System.out.println(sendResult);
    } catch (BrokerException e) {
        e.printStackTrace();
    }
}
```
上述样例演示了如何通过`Java SDK`发布事件，完整的代码请参见[Java SDK代码样例](https://github.com/WeBankFinTech/WeEvent/blob/master/src/test/java/com/webank/weevent/sample/JavaSDK.java) 。