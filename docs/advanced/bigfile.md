## 大文件传输

因为`WeEvent`的事件是会持久化存储在区块链上的，区块链对交易的大小有一定的限制。`WeEvent`现在支持的事件内容最大值为`10K`。这个阈值在大部分场景下是能够满足要求的。`WeEvent`也支持传输`GB`级别的大文件，发布订阅的流程和`API`接口和普通的小事件基本一致。

大文件的数据内容本身不上链，只是通过区块链的`p2p`网络从发布方传输到订阅方，文件传输完毕后将这个动作作为一个事件上链。

### 功能集成
- 集成Java SDK

  大文件传输因为涉及到文件的分片上传和下载，以及可能出现的断点续传，所以该功能只在`Java SDK`中提供。`Java SDK`的集成和使用参见[WeEvent Java SDK](../protocol/weevent-client-sdk.html) 。
  
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
        IWeEventClient client = new IWeEventClient.Builder().brokerUrl("http://localhost:8080/weevent-broker").build();
        
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

### 权限控制

默认情况下，发布方上传的文件，这个群组里的所有节点都可以订阅到。为了能在群组内指定文件的接收方，即只有被授权过的节点才能订阅文件。我们通过在`Topic`主题维度上配置公私钥来解决身份识别和授权认证的问题。

1. 为文件接收方生成账号文件

   通过`FISCO-BCOS`提供的脚本[生成公私钥](https://raw.githubusercontent.com/FISCO-BCOS/console/master/tools/get_account.sh)。

2. 将私钥文件`*.pem`配到接收方的`WeEvent`节点中，将公钥文件`*.public.pem`配置到发送方的`WeEvent`节点中。

3. 重启`WeEvent`节点，后续的文件传输都在授权通道中进行。代码不用做任何改动。

4. 公私钥文件在`./conf/file-transport/*`目录下配置，文件结构参见[公私钥文件结构](https://github.com/WeBankFinTech/WeEvent/tree/master/weevent-broker/src/main/resources/file-transport)。

### 服务状态

- 文件上传事件的区块链验证

```bash
$ curl http://localhost:7000/weevent-broker/file/verify?eventId=4e92cf63-9-101
{
	"code": 0,
	"message": "success",
	"data": {
		"file": {
			"fileId": "96d0011b3c5b43eab79482bb17ffe84b",
			"fileName": "log4j2.xml",
			"fileSize": 1118,
			"fileMd5": "51d695be52981dadf76778b65e5e11fd",
			"topic": "com.weevent.file2",
			"groupId": "1",
			"chunkSize": 1048576,
			"chunkNum": 1,
			"chunkStatus": null,
			"startTime": 1584055853,
			"host": ""
		},
		"plus": {
			"timestamp": 1584055847230,
			"height": 101,
			"txHash": "0x3bc6cd94ec30e969832564896ae121e586c953cd9075a5916e44c686ada8f72e",
			"sender": "0x64fa644d2a694681bd6addd6c5e36cccd8dcdde3"
		}
	}
}
```
```eval_rst
.. note::
    - 输入参数`eventId`为文件传输完毕时的上链事件，输出`file`字段为文件相关信息，`plus`为区块链相关信息，包括交易时间、块高、事件hash、交易账号。
```
- 文件传输的实时状态

```bash
$ curl http://localhost:7000/weevent-broker/file/stats
{
	"code": 0,
	"message": "success",
	"data": {
		"sender": {
			"1": {
				"com.weevent.file": [],
				"com.weevent.file2": [{
					"file": {
						"fileId": "def272b60b4a461a93ec1aa8e7a15215",
						"fileName": "weevent-1.2.0.tar.gz",
						"fileSize": 172172008,
						"fileMd5": "b08ca390265602f483f5013f12462c99",
						"topic": "com.weevent.file2",
						"groupId": "1",
						"chunkSize": 1048576,
						"chunkNum": 165,
						"chunkStatus": null,
						"startTime": 1584055678,
						"host": null						
					},
					"time": "178s",
					"readyChunk": 78,
					"process": "47.27%",
					"speed": "459488.36B/s"
				}]
			}
		},
		"receiver": {
			"1": {}
		}
	}
}
```

输出说明：

`sender`是作为发送方正在传输的文件列表，`receiver`是作为接收方正在传输的文件列表，两个结构相同。数字`1`表示所处的群组号，`com.weevent.file2`为关联的主题。列表中的对象为某个文件传输的实时状态，包括基本表信息、耗时、进度、平均速度等。

