## 大文件传输

因为`WeEvent`的事件是会持久化存储在区块链上的，区块链对交易的大小有一定的限制。`WeEvent`现在支持的事件内容最大值为`10K`。这个阈值在大部分场景下是能够满足要求的。`WeEvent`也支持传输`GB`级别的大文件，发布订阅的流程和`API`接口和普通的小事件基本一致。

大文件的数据内容本身不上链，只是通过区块链的`p2p`网络从发布方传输到订阅方，文件传输完毕后将这个动作作为一个事件上链。

### 功能集成
- 集成Java SDK

  大文件传输因为涉及到文件的分片上传和下载，以及可能出现的断点续传，所以该功能只在`Java SDK`中提供。`Java SDK`的集成和使用参见[WeEvent Java SDK](../protocol/javasdk.html) 。
  
- 和普通事件的发布订阅对比

  发布订阅大文件，和普通事件在流程上没有区别。都是先创建`Topic`主题，然后通过这个主题来发布和订阅。只是在个别的`API`使用上有点不同。

  | 功能          | 普通事件    | 大文件        |
  | ------------- | ----------- | ------------- |
  | 开启/关闭主题 | open/close  | open/close    |
  | 发布          | publish     | publishFile   |
  | 订阅          | subscirbe   | subscirbeFile |
  | 取消订阅      | unSubscribe | unSubscribe   |


### 代码样例

```java
public class Sample {
    public static void main(String[] args) {
    try {
        IWeEventClient client = new IWeEventClient.Builder().brokerUrl("http://localhost:8080/weevent-broker");
        
        String topicName = "com.weevent.file";
        // open 一个"com.weevent.file"的主题,可以重复open
        client.open(topicName);
        
        // 发布文件 "src/main/resources/log4j2.xml"
        SendResult sendResult = client.publishFile("com.weevent.file", new File("src/main/resources/log4j2.xml").getAbsolutePath());
        System.out.println(sendResult);
        
        // 订阅文件，接收到的文件存到"./logs"目录下
        String subscriptionId = client.subscribeFile("com.weevent.file", "./logs", new IWeEventClient.FileListener() {
            @Override
            public void onFile(String subscriptionId, String localFile) {
                Assert.assertFalse(subscriptionId.isEmpty());
                Assert.assertFalse(localFile.isEmpty());

                // 业务可以从localFile里读取到文件内容
            }

            @Override
            public void onException(Throwable e) {

            }
        });
        
        // 取消订阅
        client.unSubscribe(subscriptionId);
    } catch (BrokerException e) {
        e.printStackTrace();
    }
}
```

