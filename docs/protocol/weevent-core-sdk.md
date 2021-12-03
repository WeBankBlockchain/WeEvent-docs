## Java Core SDK
本节介绍如何将核心的发布订阅功能以`jar`包形式直接集成到业务服务里。

### 前置条件

- 区块链FISCO-BCOS节点

  必选配置。`WeEvent`通过区块链`FISCO-BCOS`持久化数据。

  具体安装步骤，请参见[FISCO-BCOS安装](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/installation.html)。

### 集成SDK

- gradle依赖
```groovy
implement 'com.webank.weevent:weevent-core:1.6.0'
```
- maven依赖
```xml
<dependency>
    <groupId>com.webank.weevent</groupId>
    <artifactId>weevent-core</artifactId>
    <version>1.6.0</version>
</dependency>
```

### API接口
```java
public class FiscoBcosInstance {
    private FiscoBcosDelegate fiscoBcosDelegate;

    public FiscoBcosInstance(FiscoConfig config) throws BrokerException {
        this.fiscoBcosDelegate = new FiscoBcosDelegate();
        this.fiscoBcosDelegate.initProxy(config);
    }

    /**
     * build a producer
     *
     * @return IProducer producer handler
     */
    public IProducer buildProducer() {
        return new FiscoBcosBroker4Producer(this.fiscoBcosDelegate);
    }

    /**
     * build a consumer
     *
     * @return IConsumer consumer handler
     */
    public IConsumer buildConsumer() {
        return new FiscoBcosBroker4Consumer(this.fiscoBcosDelegate);
    }
}
```

### 代码样例

```java
public static void main(String[] args) {
    try {
        String topicName = "com.weevent.test";
        FiscoConfig fiscoConfig = new FiscoConfig();
        // 默认从classpath里读取配置文件fisco.yml
        fiscoConfig.load("");
    
        // 获取FISCO实例
        FiscoBcosInstance fiscoBcosInstance = new FiscoBcosInstance(fiscoConfig);
        
        // 创建生产者
        IProducer iProducer = fiscoBcosInstance.buildProducer();
        iProducer.startProducer();

        // 创建topic
        iProducer.open(topicName, WeEvent.DEFAULT_GROUP_ID);

        // 生产者发布事件
        WeEvent weEvent = new WeEvent(topicName, "hello weevent".getBytes());
        SendResult sendResult = iProducer.publish(weEvent, WeEvent.DEFAULT_GROUP_ID, fiscoConfig.getWeb3sdkTimeout());
        System.out.println(sendResult);
        
        // 创建消费者
        IConsumer iConsumer = fiscoBcosInstance.buildConsumer();
        iConsumer.startConsumer();

        // 消费者订阅消息
        String subscriptionId = iConsumer.subscribe(topicName, WeEvent.DEFAULT_GROUP_ID, WeEvent.OFFSET_LAST, new HashMap<>(), new IConsumer.ConsumerListener() {
            @Override
            public void onEvent(String subscriptionId, WeEvent event) {
                // 回调后的业务处理
            }

            @Override
            public void onException(Throwable e) {
            }
        });

        // 取消订阅
        iConsumer.unSubscribe(subscriptionId);
    } catch (BrokerException e) {
        e.printStackTrace();
    }
}
```

以上样例将发布者和订阅者放在一起只是为了方便示例，业务实际场景一般在不同的进程。
完整的代码请参见[Java Core SDK代码样例](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-core/src/test/java/com/webank/weevent/core/FiscoBcosInstanceTest.java) 。

### 关于配置文件

以上样例中使用到的配置文件`fisco.yml`，内容如下：

```yaml
################ weevent core config ################

version: 2.0
orgId: fisco

timeout: 10000
poolSize: 10
maxPoolSize: 200
keepAliveSeconds: 10

consumerHistoryMergeBlock: 8
consumerIdleTime: 1000


################ fisco bcos sdk config ################
# https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/sdk/java_sdk/configuration.html

#cryptoMaterial:
#  certPath: "conf"
#  caCert: "conf/ca.crt"
#  sslCert: "conf/sdk.crt"
#  sslKey: "conf/sdk.key"
#  enSslCert: "conf/gm/gmensdk.crt"
#  enSslKey: "conf/gm/gmensdk.key"

account:
# ECDSA_TYPE
  accountAddress: "0x64fa644d2a694681bd6addd6c5e36cccd8dcdde3"
#  SM_TYPE
#  accountAddress: "0x4278900c23e4b364ba6202a24682d99be9ff8cbc"
#  accountFileFormat: "pem"
#  accountFilePath: ""
  keyStoreDir: "account"
#  password: ""

#amop:
#  - publicKeys: [ "conf/amop/consumer_public_key_1.pem" ]
#    topicName: "PrivateTopic1"
#  - password: "123456"
#    privateKey: "conf/amop/consumer_private_key.p12"
#    topicName: "PrivateTopic2"


network:
  peers:
    - "127.0.0.1:20200"

threadPool:
  channelProcessorThreadSize: "16"
  maxBlockingQueueSize: "102400"
  receiptProcessorThreadSize: "16"
```

- 将上面文件放入`classpath`下面。通过默认方式`FiscoConfig.load("")`即可加载。

### 初始化部署合约

安装完区块链网络后，需要先初始化部署`WeEvent`内置合约（一般称为`Topic Control`合约）才能使用`WeEvent`。

集成好`weevent-core.Jar`，设置好配置文件`fisco.yml`及其访问节点的证书后。执行`Jar`包里的方法部署合约：

  ```shell
  $ java -classpath "./conf:./lib/*:../lib/*" com.webank.weevent.core.fisco.util.Web3sdkUtils
    2020-06-05 11:08:36 topic control address in every group:
    topic control address in group: 1
            EchoAddress(version=10, address=0x23df89a2893120f686a4aa03b41acf6836d11e5d, isNew=false)
  ```

其中`./conf`为`fisco.yml`所在目录，`./lib`为`weevent-core.Jar`及其依赖所在目录。

这个方法可重入，重复执行时只是简单显示一下数据。屏幕输出`new`为`true`表示合约是本次部署，`false`表示是之前部署的合约。
