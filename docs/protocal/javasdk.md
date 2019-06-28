## Java SDK
`WeEvent`支持`RESTful`、`JsonRPC`、`STOMP`、`MQTT`等协议，方便各种语言的接入和适配。

同时为`Java`语言提供了独立的`SDK`。其他语言`SDK`在规划中，欢迎大家贡献代码，[WeEvent代码仓库](https://github.com/WeBankFinTech/WeEvent) 。

### 集成SDK

- gradle依赖
```groovy
implement 'com.webank.weevent:weevent-client:1.0.0'
```
- maven依赖
```xml
<dependency>
    <groupId>com.webank.weevent</groupId>
    <artifactId>weevent-client</artifactId>
    <version>1.0.0</version>
</dependency>
```

### API接口
```java

public interface IWeEventClient {
    /**
     * Get the client handler of weevent's broker with default url, http://localhost:8080/weevent.
     *
     * @throws BrokerException broker exception
     */
    static IWeEventClient build() throws BrokerException {
        return new WeEventClient();
    }

    /**
     * Get the client handler of weevent's broker with custom url.
     *
     * @param brokerUrl weevent's broker url, like http://localhost:8080/weevent
     * @throws BrokerException broker exception
     */
    static IWeEventClient build(String brokerUrl) throws BrokerException {
        return new WeEventClient(brokerUrl);
    }

    /**
     * Get the client handler of weevent's broker custom url and account authorization.
     *
     * @param brokerUrl weevent's broker url, like http://localhost:8080/weevent
     * @param userName account name
     * @param password password
     * @throws BrokerException broker exception
     */
    static IWeEventClient build(String brokerUrl, String userName, String password) throws BrokerException {
        return new WeEventClient(brokerUrl, userName, password);
    }

    /**
     * Publish an event to topic.
     *
     * @param topic topic name
     * @param content topic data
     * @return send result, SendResult.SUCCESS if success, and SendResult.eventId
     * @throws BrokerException broker exception
     */
    SendResult publish(String topic, byte[] content) throws BrokerException;

    /**
     * Subscribe events from topic.
     *
     * @param topic topic name
     * @param offset, from next event after this offset(an event id), WeEvent.OFFSET_FIRST if from head of queue, WeEvent.OFFSET_LAST if from tail of queue
     * @param listener callback
     * @return subscription Id
     * @throws BrokerException invalid input param
     */
    String subscribe(String topic, String offset, WeEventClient.EventListener listener) throws BrokerException;

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
     * List all topics in weevent's broker.
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
     * @return weevent
     * @throws BrokerException broker exception
     */
    WeEvent getEvent(String eventId) throws BrokerException;

    /**
     * Publish an event to topic.
     *
     * @param topic topic name
     * @param content topic data
     * @return send result, SendResult.SUCCESS if success, and SendResult.eventId
     * @throws BrokerException broker exception
     */
    SendResult publish(String topic, String groupId, byte[] content, Map<String, String> extensions) throws BrokerException;

    /**
     * Subscribe events from topic.
     *
     * @param topic topic name
     * @param offset, from next event after this offset(an event id), WeEvent.OFFSET_FIRST if from head of queue, WeEvent.OFFSET_LAST if from tail of queue
     * @param listener callback
     * @return subscription Id
     * @throws BrokerException invalid input param
     */
    String subscribe(String topic, String groupId, String offset, WeEventClient.EventListener listener) throws BrokerException;

    /**
     * Subscribe events from topic.
     *
     * @param topic topic name
     * @param offset, from next event after this offset(an event id), WeEvent.OFFSET_FIRST if from head of queue, WeEvent.OFFSET_LAST if from tail of queue
     * @param listener callback
     * @return subscription Id
     * @throws BrokerException invalid input param
     */
    String subscribe(String topic, String groupId, String offset,String contnueSubScriptionId, WeEventClient.EventListener listener) throws BrokerException;

    /**
     * Publish an event to topic.
     *
     * @param topic topic name
     * @param content topic data
     * @return send result, SendResult.SUCCESS if success, and SendResult.eventId
     * @throws BrokerException broker exception
     */
    SendResult publish(String topic, byte[] content, Map<String, String> extensions) throws BrokerException;

    /**
     * Close a topic.
     *
     * @param topic topic name
     * @param groupId which group to close
     * @return true if success
     * @throws BrokerException broker exception
     */
    boolean close(String topic, String groupId) throws BrokerException;

    /**
     * Check a topic is exist or not.
     *
     * @param topic topic name
     * @param groupId which group to exit
     * @return true if exist
     * @throws BrokerException broker exception
     */
    boolean exist(String topic, String groupId) throws BrokerException;

    /**
     * Open a topic.
     *
     * @param topic topic name
     * @param groupId which group to open
     * @return true if success
     * @throws BrokerException broker exception
     */
    boolean open(String topic, String groupId) throws BrokerException;

    /**
     * List all topics in weevent's broker.
     *
     * @param pageIndex page index, from 0
     * @param pageSize page size, [10, 100)
     * @return topic list
     * @throws BrokerException broker exception
     */
    TopicPage list(Integer pageIndex, Integer pageSize, String groupId) throws BrokerException;

    /**
     * Get a topic information.
     *
     * @param topic topic name
     * @return topic information
     * @throws BrokerException broker exception
     */
    TopicInfo state(String topic, String groupId) throws BrokerException;

    /**
     * Get an event information.
     *
     * @param eventId event id
     * @return weevent
     * @throws BrokerException broker exception
     */
    WeEvent getEvent(String eventId, String groupId) throws BrokerException;

}


```

### 代码样例

- WeEvent 1.0.0 版本
```java
public static void main(String[] args) {
    try {
        String url = "http://localhost:8080/weevent";
         IWeEventClient client =  IWeEventClient.build(url);
        //publish接口的参数分别是主题Topic、事件内容Content
        String groupId = "1";
        //用户自定义拓展必须以weevent-开头，可选参数。
        Map<String, String> extensions = mew HashMap<>();
        extensions.put("weevent-url",https://github.com/WeBankFinTech/WeEvent);
        SendResult sendResult = client.publish("com.weevent.test", groupId, "hello wolrd".getBytes(), extensions);
        System.out.println(sendResult);
    } catch (BrokerException e) {
        e.printStackTrace();
    }
}

```

- WeEvent 1.0.0 版本样例
演示如何通过`Java SDK`发布事件，完整的代码请参见[Java SDK代码样例](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-broker/src/test/java/com/webank/weevent/sample/JavaSDK.java) 。
