## Java JMS SDK
本节介绍如何以符合`JMS`规范，将核心的发布订阅功能以`jar`包形式直接集成到业务服务里。

### 前置条件

- Broker模块

  必选配置，通过`Broker`访问区块链。
  
  具体安装步骤，请参见[Broker模块安装](../install/module/broker.html)。
  
### 集成SDK

- gradle依赖
```groovy
implement 'com.webank.weevent:weevent-jms:1.4.0'
```
- maven依赖
```xml
<dependency>
    <groupId>com.webank.weevent</groupId>
    <artifactId>weevent-jms</artifactId>
    <version>1.4.0</version>
</dependency>
```

### 代码样例

```java
public static void main(String[] args) throws JMSException, InterruptedException {
    private final static String topicName = "com.weevent.test";
    private final static String defaultBrokerUrl = "http://localhost:7000/weevent-broker";
    
    // get topic connection
    TopicConnectionFactory connectionFactory = new WeEventConnectionFactory(defaultBrokerUrl);
    TopicConnection connection = connectionFactory.createTopicConnection();

    // start connection
    connection.start();
    // create session
    TopicSession session = connection.createTopicSession(false, Session.AUTO_ACKNOWLEDGE);

    // create topic
    Topic topic = session.createTopic(topicName);

    // create subscriber
    TopicSubscriber subscriber = session.createSubscriber(topic);
    // create publisher
    TopicPublisher publisher = session.createPublisher(topic);

    // create listener
    subscriber.setMessageListener(message -> {
        BytesMessage msg = (BytesMessage) message;
        try {
            byte[] data = new byte[(int) msg.getBodyLength()];
            msg.readBytes(data);
            System.out.println("received: " + new String(data, StandardCharsets.UTF_8));
        } catch (JMSException e) {
            e.printStackTrace();
        }
    });
        
    // send message
    BytesMessage msg = session.createBytesMessage();
    msg.writeBytes(("hello WeEvent").getBytes(StandardCharsets.UTF_8));
    publisher.publish(msg);

    System.out.print("send done.");
   
    Thread.sleep(3000L);
    connection.close();
}
```

完整的代码请参见[Java JMS SDK代码样例](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-jms/src/test/java/com/webank/weevent/jms/JMSSample.java) 。
