## 大文件传输

因为`WeEvent`的事件是会持久化存储在区块链上的，区块链对交易的大小有一定的限制。`WeEvent`现在支持的事件内容最大值为`10K`。这个阈值在大部分场景下是能够满足要求的。`WeEvent`也支持传输`GB`级别的大文件，发布订阅的流程和`API`接口和普通的小事件基本一致。

大文件的数据内容本身不上链，只是通过区块链的`p2p`网络从发布方传输到订阅方，文件传输完毕后可以将这个动作作为一个事件上链。

### 功能使用
- 集成Java SDK

  大文件传输因为涉及到文件的分片上传和下载，以及可能出现的断点续传，所以该功能只在`Java SDK`中提供。`Java SDK`的集成和使用参见[WeEvent Java SDK](../protocol/weevent-client-sdk.html) 。
  

- 代码样例

```java
public static void main(String[] args) {
    String topicName = "com.weevent.file";
    String groupId = "1";
    String receivePath = "./logs";
    int fileChunkSize = 1024 * 1024;
    FiscoConfig fiscoConfig = new FiscoConfig();
    fiscoConfig.load("");

    try {
        IWeEventClient client = new IWeEventClient.Builder().brokerUrl("http://localhost:8080/weevent-broker").build();
        IWeEventFileClient weEventFileClient = IWeEventFileClient.build(groupId, receivePath, fileChunkSize, fiscoConfig);

        // open topic
        client.open(topicName);
        
        // 订阅文件，接收到的文件存到"./logs"目录中
        IWeEventFileClient.FileListener fileListener = new IWeEventFileClient.FileListener() {
            @Override
            public void onFile(String topicName, String fileName) {
                log.info("+++++++topic name: {}, file name: {}", topicName, fileName);
            }

            @Override
            public void onException(Throwable e) {
                e.printStackTrace();
            }
        };
        weEventFileClient.openTransport4Receiver(topicName, fileListener);

        // 发布文件"src/main/resources/ca.crt"
        weEventFileClient.openTransport4Sender(topicName);
        FileChunksMeta fileChunksMeta = weEventFileClient.publishFile(topicName,
                new File("src/main/resources/ca.crt").getAbsolutePath(), true);
        System.out.println(fileChunksMeta.toString());
        
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

### 权限控制

默认情况下，发布方上传的文件，这个群组里的所有节点都可以订阅到。为了能在群组内指定文件的接收方，即只有被授权过的节点才能订阅文件。我们通过在`Topic`主题维度上配置公私钥来解决身份识别和授权认证的问题。

1. 为文件接收方生成账号文件

   通过`FISCO-BCOS`提供的脚本[生成公私钥](https://raw.githubusercontent.com/FISCO-BCOS/console/master/tools/get_account.sh)。

2. 接收使用取私钥文件`*.pem`来开启文件传输通道，发送方使用公钥文件`*.public.pem`来开启文件传输通道。
 
- 代码样例

```java
public static void main(String[] args) {
    String topicName = "com.weevent.file";
    String groupId = "1";
    String receivePath = "./logs";
    int fileChunkSize = 1024 * 1024;
    FiscoConfig fiscoConfig = new FiscoConfig();
    fiscoConfig.load("");

    try {
        IWeEventClient client = new IWeEventClient.Builder().brokerUrl("http://localhost:8080/weevent-broker").build();
        IWeEventFileClient weEventFileClient = IWeEventFileClient.build(groupId, receivePath, fileChunkSize, fiscoConfig);

        // open topic
        client.open(topicName);

        // 订阅文件（认证方式），接收到的文件存到"./logs"目录中
        PathMatchingResourcePatternResolver resolver4Receiver = new PathMatchingResourcePatternResolver();
        Resource resource4Receiver = resolver4Receiver.getResource("classpath:" + "0x2809a9902e47d6fcaabe6d0183855d9201c93af1.pem");

        IWeEventFileClient.FileListener fileListener = new IWeEventFileClient.FileListener() {
            @Override
            public void onFile(String topicName, String fileName) {
                log.info("+++++++topic name: {}, file name: {}", topicName, fileName);
            }

            @Override
            public void onException(Throwable e) {
                e.printStackTrace();
            }
        };
        weEventFileClient.openTransport4Receiver(topicName, fileListener, resource4Receiver.getInputStream());

        // 发布(认证方式)文件"src/main/resources/ca.crt"
        PathMatchingResourcePatternResolver resolver4Sender = new PathMatchingResourcePatternResolver();
        Resource resource4Sender = resolver4Sender.getResource("classpath:" + "0x2809a9902e47d6fcaabe6d0183855d9201c93af1.public.pem");

        weEventFileClient.openTransport4Sender(topicName, resource4Sender.getInputStream());

        // handshake time delay for web3sdk
        Thread.sleep(1000*10);

        FileChunksMeta fileChunksMeta = weEventFileClient.publishFile(topicName,
                new File("src/main/resources/ca.crt").getAbsolutePath(), true);

    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

### 读写FTP服务

- 大文件传输功能支持将FTP服务器上的文件发布出去，然后接收方收到文件会存储在FTP服务器上。

- 代码样例

```java
public static void main(String[] args) {
    String topicName = "com.weevent.file";
    String groupId = "1";
    String receivePath = "./logs";
    String host = "127.0.0.1";
    int port = 21;
    String userName = "ftpuser";
    String passWd = "123456";
    String ftpReceivePath = "./";
    int fileChunkSize = 1024 * 1024;
    FiscoConfig fiscoConfig = new FiscoConfig();
    fiscoConfig.load("");

    try {
        IWeEventClient client = new IWeEventClient.Builder().brokerUrl("http://localhost:8080/weevent-broker").build();
        IWeEventFileClient weEventFileClient = IWeEventFileClient.build(groupId, receivePath, fileChunkSize, fiscoConfig);

        // open topic
        client.open(topicName);

        // 使用ftp信息构造WeEventFileClient时，默认读写FTP服务器中的文件
        IWeEventFileClient weEventFileClient = IWeEventFileClient.build(groupId, receivePath, host, port, userName, passWd, ftpReceivePath, fileChunkSize, fiscoConfig);

        // 读取FTP服务器中用户主目录下./test/build_chain.sh文件发送
        weEventFileClient.openTransport4Sender("com.weevent.file");
        FileChunksMeta fileChunksMeta = weEventFileClient.publishFile(topicName, "./test/build_chain.sh", true);
        System.out.println(fileChunksMeta.toString());

        // 订阅文件写到FTP服务器用户主目录下的ftpReceivePath目录中
        IWeEventFileClient.FileListener fileListener = new IWeEventFileClient.FileListener() {
            @Override
            public void onFile(String topicName, String fileName) {
                log.info("+++++++topic name: {}, file name: {}", topicName, fileName);
            }

            @Override
            public void onException(Throwable e) {
                e.printStackTrace();
            }
        };

        weEventFileClient.openTransport4Receiver(topicName, fileListener);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```