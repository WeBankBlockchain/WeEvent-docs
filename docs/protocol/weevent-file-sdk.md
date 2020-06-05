## Java File SDK

### 前置条件

- 区块链FISCO-BCOS节点

    必选配置。`WeEvent`通过区块链`FISCO-BCOS`的P2P网络发布和订阅文件。

    具体安装步骤，请参见[FISCO-BCOS安装](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/installation.html)。

### 集成SDK

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

### API接口
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


完整地代码请参见[大文件传输](../advanced/bigfile.md) 。