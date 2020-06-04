## 大文件传输

因为`WeEvent`的事件是会持久化存储在区块链上的，区块链对交易的大小有一定的限制。`WeEvent`现在支持的事件内容最大值为`10K`。这个阈值在大部分场景下是能够满足要求的。`WeEvent`也支持传输`GB`级别的大文件。

大文件的数据内容本身不上链，只是通过区块链的`p2p`网络从发布方传输到订阅方，文件传输完毕后可以将这个动作作为一个事件上链。

### 功能使用
- 前置条件

    - 区块链FISCO-BCOS节点

        必选配置。`WeEvent`通过区块链`FISCO-BCOS`的P2P网络发布和订阅文件。

        具体安装步骤，请参见[FISCO-BCOS安装](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/installation.html)。

- 集成SDK

    - gradle依赖
    ```groovy
    implement 'com.webank.weevent:weevent-fie:1.3.0'
    ```
    - maven依赖
    ```xml
    <dependency>
        <groupId>com.webank.weevent</groupId>
        <artifactId>weevent-file</artifactId>
        <version>1.3.0</version>
    </dependency>
    ```

- API接口
    ```java
    public interface IWeEventFileClient {
        public static IWeEventFileClient build(String groupId, String filePath, int fileChunkSize, FiscoConfig fiscoConfig) {
            return new WeEventFileClient(groupId, filePath, fileChunkSize, fiscoConfig);
        }
    
        public static IWeEventFileClient build(String groupId, String filePath, String host, int port, String userName, String passWord, String ftpReceivePath, int fileChunkSize, FiscoConfig fiscoConfig) {
            return new WeEventFileClient(groupId, filePath, host, port, userName, passWord, ftpReceivePath, fileChunkSize, fiscoConfig);
        }
    
        /**
         * open transport for sender.
         *
         * @param topic topic name
         */
        void openTransport4Sender(String topic);
    
        /**
         * open transport for authentication sender.
         *
         * @param topic topic name
         * @param publicPem public pem inputstream
         * @throws BrokerException broker exception
         */
        void openTransport4Sender(String topic, InputStream publicPem) throws BrokerException;
    
        /**
         * open transport for authentication sender.
         *
         * @param topic topic name
         * @param publicPem public pem path string
         * @throws BrokerException broker exception
         * @throws IOException IOException
         */
        void openTransport4Sender(String topic, String publicPem) throws BrokerException, IOException;
    
        /**
         * Publish a file to topic.
         * The file's data DO NOT stored in block chain. Yes, it's not persist, may be deleted sometime after subscribe notify.
         *
         * @param topic binding topic
         * @param localFile local file to be send
         * @param overwrite if receiver has this file, overwrite it?
         * @return send result, SendResult.SUCCESS if success, and return SendResult.eventId
         * @throws BrokerException broker exception
         * @throws IOException IOException
         * @throws InterruptedException InterruptedException
         */
        FileChunksMeta publishFile(String topic, String localFile, boolean overwrite) throws BrokerException, IOException, InterruptedException;
    
        /**
         * Interface for event notify callback
         */
        interface EventListener {
            /**
             * Called while new event arrived.
             *
             * @param topic topic name
             * @param fileName file name
             */
            void onEvent(String topic, String fileName);
    
            /**
             * Called while raise exception.
             *
             * @param e the e
             */
            void onException(Throwable e);
        }
    
        /**
         * open transport for receiver.
         *
         * @param topic topic name
         * @param fileListener notify interface
         * @throws BrokerException broker exception
         */
        void openTransport4Receiver(String topic, FileListener fileListener) throws BrokerException;
    
        /**
         * open transport for authentication receiver.
         *
         * @param topic topic name
         * @param fileListener notify interface
         * @param privatePem private key pem inputstream
         * @throws BrokerException broker exception
         */
        void openTransport4Receiver(String topic, FileListener fileListener, InputStream privatePem) throws BrokerException;
    
        /**
         * open transport for authentication receiver.
         *
         * @param topic topic name
         * @param fileListener notify interface
         * @param privatePem private key pem path string
         * @throws IOException IOException
         * @throws BrokerException InterruptedException
         */
        void openTransport4Receiver(String topic, FileListener fileListener, String privatePem) throws IOException, BrokerException;
    
        /**
    
         * Interface for file notify callback
         */
        interface FileListener {
            /**
             * Called while new file arrived.
             *
             * @param topicName topic name
             * @param fileName  file name
             */
            void onFile(String topicName, String fileName);
    
            /**
             * Called while raise exception.
             *
             * @param e the e
             */
            void onException(Throwable e);
        }
    
        /**
         * close transport.
         *
         * @param topic topic name
         */
        void closeTransport(String topic);
    
        /**
         * query transport status.
         *
         * @param topic topic name
         * @return filetransportstatus
         */
        FileTransportStats status(String topic);
    
        /**
         * list received files.
         *
         * @param topic topic name
         * @return filechunksmeta list
         * @throws BrokerException broker exception
         */
        List<FileChunksMeta> listFiles(String topic) throws BrokerException;
    
        /**
         * sign a file transport event.
         *
         * @param fileChunksMeta fileChunksMeta
         * @return send result and eventId
         * @throws BrokerException broker exception
         */
        SendResult sign(FileChunksMeta fileChunksMeta) throws BrokerException;
    
        /**
         * verify a file transport event.
         *
         * @param eventId eventId return by sign
         * @param groupId group id
         * @return file and block information
         * @throws BrokerException broker exception
         */
        FileChunksMetaPlus verify(String eventId, String groupId) throws BrokerException;
    
        /**
         * get DiskFiles.
         * @return DiskFiles
         */
        DiskFiles getDiskFiles();
    }
    
    ```
  

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
            IWeEventFileClient weEventFileClient = IWeEventFileClient.build(groupId, receivePath, fileChunkSize, fiscoConfig);
    
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
  
            // 对文件传输事件上链
            SendResult sendResult = weEventFileClient.sign(fileChunksMeta);
            
            // 验证文件传输事件
            FileChunksMetaPlus fileChunksMetaPlus = weEventFileClient.verify(sendResult.getEventId(), this.groupId);
              
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    ```

### 权限控制

默认情况下，发布方上传的文件，这个群组里的所有节点都可以订阅到。为了能在群组内指定文件的接收方，即只有被授权过的节点才能订阅文件。我们通过在`Topic`主题维度上配置公私钥来解决身份识别和授权认证的问题。

1. 为文件接收方生成公私钥文件

   通过`FISCO-BCOS`提供的脚本[生成公私钥](https://raw.githubusercontent.com/FISCO-BCOS/console/master/tools/get_account.sh)。

2. 生成公私钥文件后，接收方使用私钥文件`*.pem`来开启文件传输通道，发送方使用公钥文件`*.public.pem`来开启文件传输通道。
 
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
        // 构建WeEventFileClient，其中receivePath为接收方收到文件存储的位置
        IWeEventFileClient weEventFileClient = IWeEventFileClient.build(groupId, receivePath, fileChunkSize, fiscoConfig);

        // 接收方加载生成的"*.pem"私钥文件
        PathMatchingResourcePatternResolver resolver4Receiver = new PathMatchingResourcePatternResolver();
        Resource resource4Receiver = resolver4Receiver.getResource("classpath:" + "0x2809a9902e47d6fcaabe6d0183855d9201c93af1.pem");

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
        weEventFileClient.openTransport4Receiver(topicName, fileListener, resource4Receiver.getInputStream());

        // 发送方加载生成的"*.pem"私钥文件
        PathMatchingResourcePatternResolver resolver4Sender = new PathMatchingResourcePatternResolver();
        Resource resource4Sender = resolver4Sender.getResource("classpath:" + "0x2809a9902e47d6fcaabe6d0183855d9201c93af1.public.pem");

        // 发送文件"src/main/resources/ca.crt"
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

- 大文件传输功能支持将FTP服务器上的文件发布出去，接收方收到文件后将文件存储到FTP服务器指定目录中。读写FTP服务器时所用的目录为FTP用户主目录下的相对路径。
1. 使用FTP信息（host、port、username、password等）构造WeEventFileClient。
2. 发布订阅FTP服务器中的文件与普通发布订阅区别只在于所指定的文件目录，请确保FTP服务器中相应目录或文件已存在。

- 代码样例

```java
public static void main(String[] args) {
    String topicName = "com.weevent.file";
    String groupId = "1";
    String receivePath = "./logs";
    int fileChunkSize = 1024 * 1024;
    FiscoConfig fiscoConfig = new FiscoConfig();
    fiscoConfig.load("");

    // FTP连接信息
    String host = "127.0.0.1";
    int port = 21;
    String userName = "ftpuser";
    String passWd = "123456";
    String ftpReceivePath = "./";

    try {
        IWeEventFileClient weEventFileClient = IWeEventFileClient.build(groupId, receivePath, fileChunkSize, fiscoConfig);

        // 当使用ftp信息构造WeEventFileClient时，默认读写FTP服务器中用户主目录下的文件
        IWeEventFileClient weEventFileClient = IWeEventFileClient.build(groupId, receivePath, host, port, userName, passWd, ftpReceivePath, fileChunkSize, fiscoConfig);

        // 发送方读取FTP服务器中用户主目录下./test/build_chain.sh文件发送
        weEventFileClient.openTransport4Sender("com.weevent.file");
        FileChunksMeta fileChunksMeta = weEventFileClient.publishFile(topicName, "./test/build_chain.sh", true);
        System.out.println(fileChunksMeta.toString());

        // 订阅文件接收到文件后，写到FTP服务器用户主目录下的ftpReceivePath目录中
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