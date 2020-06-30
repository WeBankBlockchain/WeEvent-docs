## 大文件传输

因为`WeEvent`的事件是会持久化存储在区块链上的，区块链对交易的大小有一定的限制。`WeEvent`现在支持的事件内容最大值为`10K`。这个阈值在大部分场景下是能够满足要求的。`WeEvent`也支持传输`GB`级别的大文件。

大文件的数据内容本身不上链，只是通过区块链的`p2p`网络从发布方传输到订阅方，文件传输完毕后可以将这个动作作为一个事件上链。

### 功能集成

大文件传输因为涉及到文件的分片上传和下载，以及可能出现的断点续传，所以该功能只在`Java File SDK`中提供。`Java File SDK`的集成和使用参见[WeEvent File SDK](../protocol/weevent-file-sdk.md)。

### 权限控制

默认情况下，发布方上传的文件，这个群组里的所有节点都可以订阅到。为了能在群组内指定文件的接收方，即只有被授权过的节点才能订阅文件。我们通过在`Topic`主题维度上配置公私钥来解决身份识别和授权认证的问题。

1. 为文件接收方生成公私钥文件

   可以使用IWeEventFileClient.genPemFile接口生成PME格式的公私钥。
   也可以通过`FISCO-BCOS`提供的脚本[生成公私钥](https://raw.githubusercontent.com/FISCO-BCOS/console/master/tools/get_account.sh)。

2. 生成公私钥文件后，接收方使用私钥文件`*.pem`来开启文件传输通道，发送方使用公钥文件`*.public.pem`来开启文件传输通道。
 
- 代码样例

    ```java
    public static void main(String[] args) {       
        try {
            FiscoConfig fiscoConfig = new FiscoConfig();
            fiscoConfig.load("");
        
            // 构建WeEventFileClient，其中receivePath为接收方收到文件存储的位置
            IWeEventFileClient weEventFileClient = IWeEventFileClient.build("1", "./logs", 1024 * 1024, fiscoConfig);
   
            // 接收方加载生成的"*.pem"私钥文件
            PathMatchingResourcePatternResolver resolver4Receiver = new PathMatchingResourcePatternResolver();
            Resource resource4Receiver = resolver4Receiver.getResource("classpath:" + "0x2809a9902e47d6fcaabe6d0183855d9201c93af1.pem");
   
            String topicName = "com.weevent.file";
            // 订阅文件，接收到的文件存到"./logs"目录中
            weEventFileClient.openTransport4Receiver(topicName, new IWeEventFileClient.FileListener() {
                @Override
                public void onFile(String topicName, String fileName) {
                    log.info("+++++++topic name: {}, file name: {}", topicName, fileName);
                }
    
                @Override
                public void onException(Throwable e) {
                    e.printStackTrace();
                }
            }, resource4Receiver.getInputStream());
    
            // 发送方加载生成的"*.pem"私钥文件
            PathMatchingResourcePatternResolver resolver4Sender = new PathMatchingResourcePatternResolver();
            Resource resource4Sender = resolver4Sender.getResource("classpath:" + "0x2809a9902e47d6fcaabe6d0183855d9201c93af1.public.pem");
            weEventFileClient.openTransport4Sender(topicName, resource4Sender.getInputStream());
    
            // handshake time delay for web3sdk
            Thread.sleep(1000*10);
    
            // 发送文件"src/main/resources/log4j2.xml"
            FileChunksMeta fileChunksMeta = weEventFileClient.publishFile(topicName,
                    new File("src/main/resources/log4j2.xml").getAbsolutePath(), true);
    
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    ```

### 读写FTP服务

大文件传输功能支持将FTP服务器上的文件发布出去，接收方收到文件后将文件存储到FTP服务器指定目录中。读写FTP服务器时所用的目录为FTP用户主目录下的相对路径。
1. 使用FTP信息（host、port、username、password等）构造WeEventFileClient。
2. 发布订阅FTP服务器中的文件与普通发布订阅区别只在于所指定的文件目录，请确保FTP服务器中相应目录或文件已存在。

- 代码样例

    ```java
    public static void main(String[] args) {
        try {
            // FTP连接信息，接收文件的根目录./receiver
            FtpInfo ftpInfo = new FtpInfo("127.0.0.1", 21, "ftpuser", "123456", "./receiver");
            // 本地文件缓存./local
            IWeEventFileClient weEventFileClient = IWeEventFileClient.build("1", "./local", ftpInfo, 1024 * 1024, fiscoConfig);
    
            String topicName = "com.weevent.file";
            // 发送FTP服务器上的文件./sender/foo.txt
            weEventFileClient.openTransport4Sender(topicName);
            FileChunksMeta fileChunksMeta = weEventFileClient.publishFile(topicName, "./sender/foo.txt", true);
            System.out.println(fileChunksMeta.toString());
    
            // 订阅文件接收到文件后，写到FTP服务器"./receiver"目录下
            weEventFileClient.openTransport4Receiver(topicName, new IWeEventFileClient.FileListener() {
                @Override
                public void onFile(String topicName, String fileName) {
                    log.info("+++++++topic name: {}, file name: {}", topicName, fileName);
                }
    
                @Override
                public void onException(Throwable e) {
                    e.printStackTrace();
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    ```