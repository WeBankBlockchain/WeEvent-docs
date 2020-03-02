## Java Core SDK
直接将核心的发布订阅功能以`jar`包形式集成到业务服务里。

### 集成SDK

- gradle依赖
```groovy
implement 'com.webank.weevent:weevent-core:1.2.0'
```
- maven依赖
```xml
<dependency>
    <groupId>com.webank.weevent</groupId>
    <artifactId>weevent-core</artifactId>
    <version>1.2.0</version>
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
        // 从classpath里初始化配置fisco.properties
        FiscoConfig fiscoConfig = new FiscoConfig();
        fiscoConfig.load("");
    
        // 获取FISCO实例
        FiscoBcosInstance fiscoBcosInstance = new FiscoBcosInstance(fiscoConfig);
        
        // 创建生产者
        IProducer iProducer = fiscoBcosInstance.buildProducer();
        iProducer.startProducer();

        // 发布事件
        WeEvent weEvent = new WeEvent(“com.weevent.test”, "hello weevent".getBytes());
        SendResult sendResult = iProducer.publish(weEvent, “1”, fiscoConfig.getWeb3sdkTimeout());
        System.out.println(sendResult);
        
        // 创建生产者
        IConsumer iConsumer = fiscoBcosInstance.buildConsumer();
        iConsumer.startConsumer();

		// 订阅消息
        String subscriptionId = iConsumer.subscribe(“com.weevent.test”, “1”, WeEvent.OFFSET_LAST, new HashMap<>(), new IConsumer.ConsumerListener() {
            @Override
            public void onEvent(String subscriptionId, WeEvent event) {
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

完整的代码请参见[Java Core SDK代码样例](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-core/src/test/java/com/webank/weevent/core/FiscoBcosInstanceTest.java) 。

### 关于配置文件

上一段样例中使用到的配置文件`fisco.properties`，内容如下：

```properties
#fisco version
version=2.0
#fisco channel nodes, like 127.0.0.1:20200
nodes=127.0.0.1:20200
orgid=fisco
#god account
account=bcec428d5205abe0f0cc8a734083908d9eb8563e31f943d760786edf42ad67dd
#web3sdk
web3sdk.timeout=10000
web3sdk.core-pool-size=10
web3sdk.max-pool-size=200
web3sdk.keep-alive-seconds=10
#consumer
consumer.idle-time=1000
consumer.history_merge_block=8
```

- 如果是普通Java服务，将上面文件放入`classpath`下面。通过默认方式`FiscoConfig.load("")`即可加载。
- 如果是`Spring`上下服务，将包路径`com.webank.weevent.core.config`加入扫描即可导出已经配置好的`Bean FiscoConfig`。示例如下（"com.webank.weevent.broker"为集成服务的包名）：

```java
@SpringBootApplication(scanBasePackages = {"com.webank.weevent.broker", "com.webank.weevent.core.config"})
```

