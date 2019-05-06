## JsonRPC

相比`RESTful`规范，`JsonRPC`是更符合后台开发习惯的文本协议，`Java`代码映射也更加简单。

`WeEvent`除了少量系统管理功能外，主题的`CRUD`和事件的订阅发布等都支持`JsonRPC`协议访问。

### 协议说明

- `JsonRPC`和`RESTful`实现的功能和参数都相同，只是不同的承载协议，可以自由选择使用。  

- `WeEvent` 使用`JsonRPC 2.0` [协议规范 ](https://www.jsonrpc.org/specification) 。

- `JsonRPC`的传输对象使用`Jackson`库进行序列化。对于字符数组`byte[]`，`Jackson`会先使用`Base64`进行编解码。

- 调用异常返回的信息`BrokerException`如下

  ```json
  {
  	"jsonrpc": "2.0",
  	"id": "1",
  	"error": {
  		"code": -32001,
  		"message": "topic not exist",
  		"data": {
  			"exceptionTypeName": "com.webank.weevent.broker.sdk.BrokerException",
  			"message": "topic not exist"
  		}
  	}
  }
  ```


### 代码样例 

```java
public class JsonRPC {
    public static void main(String[] args) {
        try {
            URL remote = new URL("http://localhost:8080/weevent/jsonrpc");
            // 创建客户端
            JsonRpcHttpClient client = new JsonRpcHttpClient(remote);
            // 实例化rpc对象
            IBrokerRpc rpc = ProxyUtil.createClientProxy(client.getClass().getClassLoader(), IBrokerRpc.class, client);

            // 确认主题存在
            rpc.open("com.weevent.test");

            // 发布事件，主题“com.weevent.test”，事件内容为"hello weevent"
            SendResult sendResult = rpc.publish("com.weevent.test", "hello weevent".getBytes(StandardCharsets.UTF_8));
            System.out.println(sendResult.getStatus());
        } catch (MalformedURLException e) {
            e.printStackTrace();
        } catch (BrokerException e) {
            e.printStackTrace();
        }
    }
}
```

上述样例演示了使用`JsonRPC`如何创建主题和发布事件。完整代码，请参见[JsonRPC代码样例](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-broker/src/test/java/com/webank/weevent/sample/JsonRPC.java) 。

### 接口说明

`JsonRPC`接口包括两大类功能：一类是主题`Topic`的`CRUD`管理，包括`open`、`close`、`exist`、`state`、`list`；一类是事件的发布和订阅，包括`publish`、`subscribe`、`unsubscribe` 。

#### 创建Topic
- 请求
```bash
$ curl -H"Content-Type: application/json" -d'{"id":"1","jsonrpc":"2.0","method":"open","params":{"topic":"com.weevent.test"}}' http://localhost:8080/weevent/jsonrpc
```
- 应答
```json
{
    "jsonrpc": "2.0",
    "id": "1",
    "result": "true"
}
```

- 说明

  topic：主题，`ascii`值在`[32,128]`之间的都为有效字符。
  可以重复`open`，也是返回`true`


#### 关闭Topic

- 请求
```
$ curl -H"Content-Type: application/json" -d'{"id":"1","jsonrpc":"2.0","method":"close","params":{"topic":"com.weevent.test"}}' http://localhost:8080/weevent/jsonrpc
```
- 应答
```json
{
    "jsonrpc": "2.0",
    "id": "1",
    "result": "true"
}
```


#### 检查Topic是否存在
- 请求
```shell
$ curl -H"Content-Type: application/json" -d'{"id":"1","jsonrpc":"2.0","method":"exist","params":{"topic":"com.weevent.test"}}' http://localhost:8080/weevent/jsonrpc
```
- 应答
```json
{
    "jsonrpc": "2.0",
    "id": "1",
    "result": "true"
}
```

#### 发布事件
- 请求
```shell
$ curl -H "Content-Type: application/json" -d'{"id":"1","jsonrpc":"2.0","method":"publish","params":{"topic":"com.weevent.test","content":"MTIzNDU2"}}' http://localhost:8082/weevent/jsonrpc 
```
- 应答
```json
{
    "jsonrpc": "2.0",
    "id": "1",
    "result": {
        "topic": "com.weevent.test",
        "status": "SUCCESS",
        "eventId": "10-123"
    }
}
```
- 说明：
  - content：`MTIzNDU2`是`123456`进行`Base64`编码后的值。
  - result ： 返回字段`result` ，是一个`WeEvent`对象的序列化，可查看WeEvent对象。

       status：“SUCCESS”表示成功。

       eventId：发布成功事件ID。

#### 订阅事件  

- 请求
```shell
$ curl -H"Content-Type: application/json" -d'{"id":"1","jsonrpc":"2.0","method":"subscribe","params":{"topic":"com.weevent.test","subscriptionId":"df68c385-f62d-437f-b32c-669211d51d88","url":"http://localhost/weevent/onsubscribe"}}' http://localhost:8080/weevent/jsonrpc
```
- 应答
```json
{
    "jsonrpc": "2.0",
    "id": "1",
    "result": "df68c385-f62d-437f-b32c-669211d51d88"
}
```

- 说明
  - url：事件通知回调`CGI `，当有生产者发布事件时，所有的事件都会通知到这个`URL`。 
  - subscriptionId：第一次订阅使用空字符串""。继续上一次订阅，`subscriptionId`是上次订阅ID。
  - result：是订阅ID。

#### 取消订阅

- 请求
```shell
$ curl -H"Content-Type: application/json" -d'{"id":"1","jsonrpc":"2.0","method":"unSubscribe","params":{"subscriptionId":"c8a600c0-61a7-4077-90f6-3aa39fc9cdd5"}}' http://localhost:8080/weevent/jsonrpc
```
- 应答
```json
{
    "jsonrpc": "2.0",
    "id": "1",
    "result": "true"
}
```

- 说明
  - 如果`Broker`没有配置`Zookeeper`模块，该接口无法使用。
  - subscriptionId：`Subscribe`成功订阅后，返回的订阅ID。

####  获取Event详情
- 请求
```shell
$ curl -H"Content-Type: application/json" -d '{"id":"1","jsonrpc":"2.0","method":"getEvent","params":{"eventId":"2cf24dba-59-1124"}}' http://localhost:8082/weevent/jsonrpc
```
- 应答
```json
{
    "jsonrpc": "2.0",
    "id": "1",
    "result": {
        "topic": "hello",
        "content": "MTIzNDU2",
        "eventId": "2cf24dba-59-1124"
    }
}
```



**注意**
以下管理端接口，业务程序里一般用不到，可以直接安装`Goverance`模块来使用这部分功能。

#### 当前Topic列表
- 请求
```shell
$ curl -H"Content-Type: application/json" -d'{"id":"1","jsonrpc":"2.0","method":"list","params":{"pageIndex":1，"pageSize":10}}' http://localhost:8080/weevent/jsonrpc
```
- 应答
```json
{
    "jsonrpc": "2.0",
    "id": "1",
    "result": {
        "total": 51,
        "pageIndex": 1,
        "pageSize": 10,
        "topicInfoList": [
            {
                "topicName": "123456",
                "topicAddress": "0x420f853b49838bd3e9466c85a4cc3428c960dde2",
                "senderAddress": "0x64fa644d2a694681bd6addd6c5e36cccd8dcdde3",
                "createdTimestamp": 1548211117753
            }
        ]
    }
}
```

说明

- total：`Topic`的总数量
- pageIndex:：查询第几页，从0开始  
- pageSize： 分页大小（0,100），默认每页10条数据。
- topicInfoList：`Topic` 详细信息列表

#### 查询某个Topic详情
- 请求
```shell
$ curl -H"Content-Type: application/json" -d'{"id":"1","jsonrpc":"2.0","method":"state","params":{"topic":"com.weevent.test"}' http://localhost:8080/weevent/jsonrpc
```
- 应答
```json
{
    "jsonrpc": "2.0",
    "id": "1",
    "result": {
        "topicName": "com.weevent.test",
        "topicAddress": "0x171befab4c1c7e0d33b5c3bd932ce0112d4caecd",
        "senderAddress": "0x64fa644d2a694681bd6addd6c5e36cccd8dcdde3",
        "createdTimestamp": 1548328570965
    }
}
```

返回值说明

- topicAddress： `Topic`区块链上的合约地址。
- createdTimestamp：`Topic`创建的时间。

