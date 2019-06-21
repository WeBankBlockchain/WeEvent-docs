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

    /**
     * Get the client handler of weevent's broker with default url, http://localhost:8080/weevent.
     *
     * @throws BrokerException broker exception
     */
    public WeEventClient() throws BrokerException {
        buildRpc(defaultJsonRpcUrl);
        buildJms(WeEventConnectionFactory.defaultBrokerUrl, "", "");
    }

    /**
     * Get the client handler of weevent's broker with custom url.
     *
     * @param brokerUrl weevent's broker url, like http://localhost:8080/weevent
     * @throws BrokerException broker exception
     */
    public WeEventClient(String brokerUrl) throws BrokerException {
        validateParam(brokerUrl);
        buildRpc(brokerUrl + "/jsonrpc");
        buildJms(getStompUrl(brokerUrl), "", "");
    }

    /**
     * Get the client handler of weevent's broker custom url and account authorization.
     *
     * @param brokerUrl weevent's broker url, like http://localhost:8080/weevent
     * @param userName account name
     * @param password password
     * @throws BrokerException broker exception
     */
    public WeEventClient(String brokerUrl, String userName, String password) throws BrokerException {
        validateParam(brokerUrl);
        validateUser(userName, password);
        buildRpc(brokerUrl);
        buildJms(getStompUrl(brokerUrl), userName, password);
    }

    /**
     * Publish an event to topic.
     *
     * @param topic topic name
     * @param content topic data
     * @return send result, SendResult.SUCCESS if success, and SendResult.eventId
     * @throws BrokerException broker exception
     */
    public SendResult publish(String topic, byte[] content) throws BrokerException {
        validateParam(topic);
        return this.brokerRpc.publish(topic, content);
    }

    /**
     * Subscribe events from topic.
     *
     * @param topic topic name
     * @param offset, from next event after this offset(an event id), WeEvent.OFFSET_FIRST if from head of queue, WeEvent.OFFSET_LAST if from tail of queue
     * @param listener callback
     * @return subscription Id
     * @throws BrokerException invalid input param
     */
    public String subscribe(String topic, String groupId, String offset, EventListener listener) throws BrokerException {
        try {
            validateParam(topic);
            validateParam(offset);
            TopicSession session = this.connection.createTopicSession(false, Session.AUTO_ACKNOWLEDGE);
            // create topic
            Topic destination = session.createTopic(topic);

            // create subscriber
            ((WeEventTopic) destination).setOffset(offset);
            ((WeEventTopic) destination).setGroupId(groupId);//if not set default 1
            WeEventTopicSubscriber subscriber = (WeEventTopicSubscriber) session.createSubscriber(destination);

            // create listener
            subscriber.setMessageListener(new MessageListener() {
                public void onMessage(Message message) {
                    if (message instanceof BytesMessage) {
                        try {
                            BytesMessage bytesMessage = (BytesMessage) message;
                            ObjectMapper mapper = new ObjectMapper();
                            byte[] body = new byte[(int) bytesMessage.getBodyLength()];
                            bytesMessage.readBytes(body);
                            WeEvent event = mapper.readValue(body, WeEvent.class);
                            listener.onEvent(event);
                        } catch (IOException | JMSException e) {
                            log.error("onMessage exception", e);
                            listener.onException(e);
                        }
                    }
                }
            });

            this.sessionMap.put(subscriber.getSubscriptionId(), session);
            return subscriber.getSubscriptionId();
        } catch (JMSException e) {
            log.error("jms exception", e);
            throw jms2BrokerException(e);
        }
    }


    public String subscribe(String topic, String offset, EventListener listener) throws BrokerException {
        try {
            validateParam(topic);
            validateParam(offset);
            TopicSession session = this.connection.createTopicSession(false, Session.AUTO_ACKNOWLEDGE);
            // create topic
            Topic destination = session.createTopic(topic);

            // create subscriber
            ((WeEventTopic) destination).setOffset(offset);
            WeEventTopicSubscriber subscriber = (WeEventTopicSubscriber) session.createSubscriber(destination);

            // create listener
            subscriber.setMessageListener(new MessageListener() {
                public void onMessage(Message message) {
                    if (message instanceof BytesMessage) {
                        try {
                            BytesMessage bytesMessage = (BytesMessage) message;
                            ObjectMapper mapper = new ObjectMapper();
                            byte[] body = new byte[(int) bytesMessage.getBodyLength()];
                            bytesMessage.readBytes(body);
                            WeEvent event = mapper.readValue(body, WeEvent.class);
                            listener.onEvent(event);
                        } catch (IOException | JMSException e) {
                            log.error("onMessage exception", e);
                            listener.onException(e);
                        }
                    }
                }
            });

            this.sessionMap.put(subscriber.getSubscriptionId(), session);
            return subscriber.getSubscriptionId();
        } catch (JMSException e) {
            log.error("jms exception", e);
            throw jms2BrokerException(e);
        }
    }

    /**
     * Unsubscribe an exist subscription subscribed by subscribe interface.
     * The consumer will no longer receive messages from broker after this.
     *
     * @param subscriptionId invalid input
     * @return success if true
     * @throws BrokerException broker exception
     */
    public boolean unSubscribe(String subscriptionId) throws BrokerException {
        validateParam(subscriptionId);

        if (this.sessionMap.containsKey(subscriptionId)) {
            TopicSession session = this.sessionMap.get(subscriptionId);
            try {
                session.unsubscribe(subscriptionId);
            } catch (JMSException e) {
                log.error("jms exception", e);
                throw jms2BrokerException(e);
            }

            this.sessionMap.remove(subscriptionId);
            return true;
        }

        return false;
    }


    /**
     * Open a topic
     *
     * @param topic topic name
     * @return true if success
     * @throws BrokerException broker exception
     */

    public boolean open(String topic) throws BrokerException {
        validateParam(topic);
        return this.brokerRpc.open(topic);
    }


    /**
     * Close a topic.
     *
     * @param topic topic name
     * @return true if success
     * @throws BrokerException broker exception
     */
    public boolean close(String topic) throws BrokerException {
        validateParam(topic);
        return this.brokerRpc.close(topic);
    }


    /**
     * Check a topic is exist or not.
     *
     * @param topic topic name
     * @return true if exist
     * @throws BrokerException broker exception
     */
    public boolean exist(String topic) throws BrokerException {
        validateParam(topic);
        return this.brokerRpc.exist(topic);
    }


    /**
     * List all topics in weevent's broker.
     *
     * @param pageIndex page index, from 0
     * @param pageSize page size, [10, 100)
     * @return topic list
     * @throws BrokerException broker exception
     */
    public TopicPage list(Integer pageIndex, Integer pageSize) throws BrokerException {
        return this.brokerRpc.list(pageIndex, pageSize);
    }


    /**
     * Get a topic information.
     *
     * @param topic topic name
     * @return topic information
     * @throws BrokerException broker exception
     */
    public TopicInfo state(String topic) throws BrokerException {
        validateParam(topic);
        return this.brokerRpc.state(topic);
    }


    /**
     * Get an event information.
     *
     * @param eventId event id
     * @return weevent
     * @throws BrokerException broker exception
     */
    public WeEvent getEvent(String eventId) throws BrokerException {
        validateParam(eventId);
        return this.brokerRpc.getEvent(eventId);
    }


    /**
     * Publish an event to topic.
     *
     * @param topic topic name
     * @param content topic data
     * @return send result, SendResult.SUCCESS if success, and SendResult.eventId
     * @throws BrokerException broker exception
     */
    public SendResult publish(String topic, String groupId, byte[] content, Map<String, String> extensions) throws BrokerException {
        validateParam(topic);
        validateParam(groupId);
        return this.brokerRpc.publish(topic, groupId, content, extensions);
    }

    /**
     * Publish an event to topic.
     *
     * @param topic topic name
     * @param content topic data
     * @return send result, SendResult.SUCCESS if success, and SendResult.eventId
     * @throws BrokerException broker exception
     */
    public SendResult publish(String topic, byte[] content, Map<String, String> extensions) throws BrokerException {
        validateParam(topic);
        return this.brokerRpc.publish(topic, content, extensions);
    }

    /**
     * Close a topic.
     *
     * @param topic topic name
     * @param groupId which group to close
     * @return true if success
     * @throws BrokerException broker exception
     */
    public boolean close(String topic, String groupId) throws BrokerException {
        validateParam(topic);
        validateParam(groupId);
        return this.brokerRpc.close(topic, groupId);
    }

    /**
     * Check a topic is exist or not.
     *
     * @param topic topic name
     * @param groupId which group to exit
     * @return true if exist
     * @throws BrokerException broker exception
     */
    public boolean exist(String topic, String groupId) throws BrokerException {
        validateParam(topic);
        validateParam(groupId);
        return this.brokerRpc.exist(topic, groupId);
    }


    /**
     * Open a topic.
     *
     * @param topic topic name
     * @param groupId which group to open
     * @return true if success
     * @throws BrokerException broker exception
     */
    public boolean open(String topic, String groupId) throws BrokerException {
        validateParam(topic);
        validateParam(groupId);
        return this.brokerRpc.open(topic, groupId);
    }

    /**
     * List all topics in weevent's broker.
     *
     * @param pageIndex page index, from 0
     * @param pageSize page size, [10, 100)
     * @return topic list
     * @throws BrokerException broker exception
     */
    public TopicPage list(Integer pageIndex, Integer pageSize, String groupId) throws BrokerException {
        validateParam(groupId);
        return this.brokerRpc.list(pageIndex, pageSize, groupId);
    }

    /**
     * Get a topic information.
     *
     * @param topic topic name
     * @return topic information
     * @throws BrokerException broker exception
     */
    public TopicInfo state(String topic, String groupId) throws BrokerException {
        validateParam(topic);
        validateParam(groupId);
        return this.brokerRpc.state(topic, groupId);
    }


    /**
     * Get an event information.
     *
     * @param eventId event id
     * @return weevent
     * @throws BrokerException broker exception
     */
    public WeEvent getEvent(String eventId, String groupId) throws BrokerException {
        validateParam(groupId);
        validateParam(eventId);
        return this.brokerRpc.getEvent(eventId, groupId);
    }

```

### 代码样例
- WeEvent 0.9版本
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

- WeEvent 1.0.0 版本
```java
public static void main(String[] args) {
    try {
        String url = "http://localhost:8080/weevent";
        WeEventClient client = WeEventClient(url);
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
演示了如何通过`Java SDK`发布事件，完整的代码请参见[Java SDK代码样例](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-broker/src/test/java/com/webank/weevent/sample/JavaSDK.java) 。
