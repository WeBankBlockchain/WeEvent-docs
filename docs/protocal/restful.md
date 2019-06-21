## RESTful
`RESTful`作为一种软件架构风格、设计风格，在Web应用上非常流行。基于这个风格设计的软件可以更简洁，`REST` 接口可以直接在浏览器上测试，给开发和测试过程带来很多便利。`WeEvent`的所有功能都支持`RESTful`协议访问。

### 协议说明

- `HTTP`方法可以使用`GET`和`POST`，并且统一使用`UTF8`字符集。

- 输入参数为`Query String`格式，注意使用`UrlEncode`。

- 应答的`HTTP`状态码是 `200`，输出参数统一是`json`格式，`Content-type: application/json`。

- 调用异常时，返回异常信息`BrokerException`。

  ```json
  {"code": 200202, "the transaction does not correctly executed."}
  ```

### 代码样例

```java
public class Rest {
    public static void main(String[] args) {
        try {
            SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
            RestTemplate rest = new RestTemplate(requestFactory);

            // ensure topic exist "com.weevent.test"
            Boolean result = rest.getForEntity("http://localhost:8080/weevent/rest/open?topic={}",
                    Boolean.class,
                    "com.weevent.test").getBody();
            System.out.println(result);

            // publish event to topic "com.weevent.test"
            SendResult sendResult = rest.getForEntity("http://localhost:8080/weevent/rest/publish?topic={}&groupId={}&content={}",
                    SendResult.class,
                    "com.weevent.test",
                    "1",
                    "hello weevent".getBytes(StandardCharsets.UTF_8)).getBody();

            System.out.println(sendResult.getStatus());
            System.out.println(sendResult.getEventId());
        } catch (RestClientException e) {
            e.printStackTrace();
        }
    }
}
```
上述代码样例，演示了使用`RESTful`如何创建主题和发布事件。程序执行效果和在浏览器上直接访问下面URL是一样的。

http://localhost:8080/weevent/rest/open?topic=com.weevent.test&groupId=1

http://localhost:8080/weevent/rest/publish?topic=com.weevent.test&groupId=1&content=hello weevent

完整的代码，请参见[RESTful代码样例](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-broker/src/test/java/com/webank/weevent/sample/Rest.java) 。


### 接口说明

`RESTful`接口包括两大类功能：一类是主题`Topic`的`CRUD`管理，包括`open`、`close`、`exist`、`state`、`list`；一类是事件的发布和订阅，包括`publish`、`subscribe`、`unsubscribe` 。

#### 创建Topic
- 请求

  ```shell
  $ curl http://localhost:8080/weevent/rest/open?topic=com.weevent.test&groupId=1
  ```


- 应答

  ```
  true
  ```


- 说明
  `topic`: `Topic`名称，最大长度32字节。`ascii`值在`[32,128]`之间，除了‘+’、‘#’都有效。‘+’、‘#’作为订阅通配符使用。

  `groupId`: 群组`Id`，`fisco-bcos 2.0+`版本支持多群组功能。2.0以下版本不支持该功能，可以不传，其他接口类似。

  重复`open`返回`true` 。

#### 关闭Topic
- 请求

  ```shell
  $ curl http://localhost:8080/weevent/rest/close?topic=com.weevent.test&groupId=1
  ```


- 应答

  ```
  true
  ```

#### 检查Topic是否存在
- 请求

  ```shell
  $ curl http://localhost:8080/weevent/rest/exist?topic=com.weevent.test&groupId=1
  ```


- 应答

  ```
  true
  ```

#### 发布事件
- 请求

  ```shell
  $ curl http://localhost:8080/weevent/rest/publish?topic=com.weevent.test&groupId=1&content=123456&weevent-url=https://github.com/WeBankFinTech/WeEvent
  ```


- 应答

  ```json
  {
      "topic": "com.weevent.test",
      "eventId": "2cf24dba-59-1124",
      "status": "SUCCESS"
  }
  ```


- 说明 

  `content` ：`123456` ，需要特别注意两点：1、`content`的`UrlEncode`编码；2、`GET`方法支持的`QueryString`最大长度为1024字节。

  `weevent-url`:用户自定义拓展，以`weevent-`开头。可选参数。

  status：`SUCCESS`，说明是发布成功，`eventId`是对应的事件ID。

#### 订阅事件
- 请求

  ```shell
  $ curl http://localhost:8080/weevent/rest/subscribe?topic=com.weevent.test&groupId=1&subscriptionId=c8a600c0-61a7-4077-90f6-3aa39fc9cdd5&url=http%3a%2f%2flocalhost%3a8080%2fweevent%2fonsubscribe
  ```


- 应答

  ```
  c8a600c0-61a7-4077-90f6-3aa39fc9cdd5
  ```


- 说明  

  - topic：事件主题。支持通配符按层次订阅，参见[MQTT通配符](http://public.dhe.ibm.com/software/dw/webservices/ws-mqtt/mqtt-v3r1.html) 。
  - url：事件通知回调`CGI `，当有生产者发布事件时，所有的事件都会通知到这个`url`。 
  - subscriptionId：第一次订阅使用空字符串""。继续上一次订阅，`subscriptionId`是上次订阅ID。
  - 返回值是订阅ID`c8a600c0-61a7-4077-90f6-3aa39fc9cdd5`。

#### 取消订阅


- 请求

  ```shell
  $ curl http://localhost:8080/weevent/rest/unSubscribe?subscriptionId=c8a600c0-61a7-4077-90f6-3aa39fc9cdd5
  ```


- 应答

  ```shell
   true
  ```


- 说明

  - 如果`Broker`没有配置`zookeeper`模块，该接口无法使用。`no broker.zookeeper.ip configuration, can't support CGI subscription` 。
  - subscriptionId：`Subscribe`成功订阅后，返回的订阅ID。

#### 获取Event详情

- 请求

  ```shell
  $ curl http://localhost:8080/weevent/rest/getEvent?eventId=2cf24dba-59-1124&groupId=1
  ```

- 应答

  ```json
  {
      "topic": "hello",
      "content": "MTIzNDU2",
      "eventId": "2cf24dba-59-1124"
  }
  ```
- 说明
  - eventId：事件ID
  - content：事件内容，`MTIzNDU2`是`123456`的`Base64`之后的值。
    ​

**注意** 

 以下管理端接口，业务程序里一般用不到，可以直接安装`Goverance`模块来使用这部分功能。

#### 当前Topic列表
- 请求

  ```shell
  $ curl http://localhost:8080/weevent/rest/list?pageIndex=1&pageSize=10&groupId=1
  ```


- 应答

  ```json
  {
      "total": 50,
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
  ```


- 说明  
  - total：`Topic`的总数量。
  - pageIndex：表示查询第几页，从0开始 。
  - pageSize：分页大小（0,100），超出这默认每页10个数据。
  - topicInfoList：`Topic` 详细信息列表。



#### 查询某个Topic详情
- 请求

  ```shell
  $ curl http://localhost:8080/weevent/rest/state?topic=com.weevent.test&groupId=1
  ```


- 应答

  ```json
  {
      "topicName": "com.weevent.test",
      "topicAddress": "0x171befab4c1c7e0d33b5c3bd932ce0112d4caecd",
      "senderAddress": "0x64fa644d2a694681bd6addd6c5e36cccd8dcdde3",
      "createdTimestamp": 1548328570965
  }
  ```


- 说明
  - topicName ：  事件名称
  - topicAddress ：  `Topic` 区块链上的合约地址
  - createdTimestamp  ：`Topic` 创建的时间

#### 区块链节点信息
- 请求 

  ```shell
  $ curl http://localhost:8080/weevent/admin/blockchaininfo
  ```


- 应答

  ```json
  {
      "blockNumber": 17056,
      "nodeIdList": [
         "7c79f9fb09366f1b872d8c6fffa888754e61e5dcddb8ed8e49a4b294c6d652111a65dfa4163910abe9d08f842e5cb700e3d981f51dd07e460caa29699033b82f"
      ],
      "nodeIpList": [
          "[10.107.105.239:30306]",
          "[10.107.105.239:30305]",
          "[10.107.105.239:30304]"
      ]
  }
  ```
#### 获取订阅列表 
- 请求
    ```shell
    $ curl http://localhost:8080/weevent/admin/listSubscription
    ```

- 应答

    ```json
    {
     "127.0.0.1:8080": {
            "a78d05b7-7e44-4f85-b1d7-395362768af0": {
                "interfaceType": "restful",
                "notifyingEventCount": "0",
                "notifyTimeStamp": "2019-04-09 20:33:15",
                "subscribeId": "a78d05b7-7e44-4f85-b1d7-395362768af0",
                "topicName": "com.webank.test",
                "notifiedEventCount": "0"
            },
            "3da9691d-ec72-4e8d-b17a-679d8a9ea111": {
                "interfaceType": "stomp",
                "notifyingEventCount": "0",
                "notifyTimeStamp": "2019-04-09 20:33:15",
                "subscribeId": "3da9691d-ec72-4e8d-b17a-679d8a9ea111",
                "topicName": "com.webank.test",
                "notifiedEventCount": "0"
            }
        }
    }
    ```
- 说明
    - interfaceType：监听请求类型 `RESTful`、`JsonRPC`、`MQTT` 、`STOMP`。
    - notifyingEventCount：待通知事件的数量。
    - notifiedEventCount：已通知事件数量
    - notifyTimeStamp：最近通知事件时间戳。
    - subscribeId：订阅ID
    - topicName ：事件主题。

