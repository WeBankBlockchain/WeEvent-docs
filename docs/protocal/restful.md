## RESTful
`RESTful`作为一种软件架构风格、设计风格，在Web应用上非常流行。基于这个风格设计的软件可以更简洁，`REST` 接口可以直接在浏览器上测试，给开发和测试过程带来很多便利。

`RESTful`都是对`STOMP`协议的一个补充，支持`WeEvent`主题的`CRUD`管理等功能以及事件发布。

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
        System.out.println("This is WeEvent restful sample.");
        try {
            SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
            RestTemplate rest = new RestTemplate(requestFactory);
            // ensure topic exist "com.weevent.test"
            Boolean result = rest.getForEntity("http://localhost:8080/weevent/rest/open?topic={}&groupId={}",
                    Boolean.class,
                    "com.weevent.test",
                    "1").getBody();
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
完整的代码，请参见[RESTful代码样例](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-broker/src/test/java/com/webank/weevent/sample/Rest.java) 。


### 接口说明

`RESTful`接口包括两大类功能：一类是主题`Topic`的`CRUD`管理，包括`open`、`close`、`exist`、`state`、`list`；一类是事件发布`publish`。

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
  
- topic：主题。`ascii`值在`[32,128]`之间。支持通配符按层次订阅，因'+'、'#'为通配符的关键字故不能为topic的一部分，详情参见[MQTT通配符](http://public.dhe.ibm.com/software/dw/webservices/ws-mqtt/mqtt-v3r1.html) 。
  
- groupId： 群组`Id`，`fisco-bcos 2.0+`版本支持多群组功能。2.0以下版本不支持该功能，可以不传，其他接口类似。
  
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
  $ curl http://localhost:8080/weevent/rest/publish?topic=com.weevent.test&groupId=1&content=123456&weevent-format=json
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

  - content ：用户自定义数据。需要特别注意`content`需进行`UrlEncode`编码，`GET`方法支持的`QueryString`最大长度为1024字节。

  - weevent-json:可选参数。用户自定义拓展，以`weevent-`开头。

  - status：`SUCCESS`，说明是发布成功，`eventId`是对应的事件ID。

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
      "extensions":{"weevent-format":"json"},
      "eventId": "2cf24dba-59-1124"
  }
  ```
- 说明
  - eventId：事件ID
  - content：事件内容，`MTIzNDU2`是`123456`的`Base64`之后的值。
  - extensions：用户自定义拓展数据。

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
      "createdTimestamp": 1548328570965,
      "sequenceNumber": 9,
      "blockNumber": 2475
  }
  ```


- 说明
  - topicName ：  事件名称
  - topicAddress ：  `Topic` 区块链上的合约地址
  - createdTimestamp  ：`Topic` 创建的时间
  - sequenceNumber：已发布事件数。
  - blockNumber：最新已发布事件的区块高度。
  
#### 获取群组列表
- 请求

  ```shell
  $ curl http://localhost:8080/weevent/rest/listGroup
  ```


- 应答

  ```json
  ["1","2"]
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
    
#### 获取版本信息
- 请求
    ```shell
    $ curl http://localhost:8080/weevent/admin/getVersion
    ```

- 应答

    ```json
    {
     "code":"0",
     "message":"success",
     "data": 
            {
                "weEventVersion": "1.0.0",
                "gitCommitTimeStamp": "2019-09-16 18:01:23",
                "gitBranch": "master",
                "gitCommitHash": "a5b022b"
            }
    }
    ```
- 说明
    - weEventVersion：WeEvent版本号。
    - gitCommitTimeStamp：WeEvent最近一次提交git的时间。
    - gitBranch：WeEvent构建分支。
    - gitCommitHash：WeEvent最近一次提交git的CommitHash。


#### 获取 节点个数、区块数量、交易数量
- 请求
    ```shell
    $ curl http://localhost:8080/weevent/admin/group/general?groupId=1
    ```

- 应答

    ```json
    {
     "code":"0",
     "message":"success",
     "data": 
            {
                "nodeCount": "4",
                "latestBlock": "100",
                "transactionCount": "5260"
            }
    }
    ```
- 说明
    - nodeCount：节点个数。
    - latestBlock：区块数量。
    - transactionCount：交易数量。

#### 获取 区块链交易列表
- 请求
    ```shell
    $ curl http://localhost:8080/weevent/admin/transaction/transList?groupId=1&pageNumber=1&pageSize=10
    ```

- 应答

    ```json
    {
     "code":"0",
     "message":"success",
     "data": 
            [{
               "blockNumber": 5364,
               "blockTimestamp": "2019-10-15 14:48:01",
               "createTime": null,
               "modifyTime": null,
               "transFrom": "0x64fa644d2a694681bd6addd6c5e36cccd8dcdde3",
               "transHash": "0xb3585cf385a595e5af425d360693e6759d8db5c1a98ebb46277b38c014ec8626",
               "transTo": "0xa40c864c28ee8b07dc2eeab4711e3161fc87e1e2"
            }]
    }
    ```
- 说明
    - blockNumber：区块高度。
    - blockTimestamp 区块交易时间。
    - createTime：创建时间。
    - modifyTime：修改时间。
    - transFrom： 发送者的地址。
    - transHash：交易哈希。
    - transTo： 接收者的地址。
    
   
#### 获取 交易哈希列表
- 请求
    ```shell
    $ curl http://localhost:8080/weevent/admin/block/blockList?groupId=1&pageNumber=1&pageSize=10
    ```

- 应答

    ```json
    {
     "code":"0",
     "message":"success",
     "data": 
            [{
                "blockNumber": 5364,
                "blockTimestamp": "2019-10-15 14:48:01",
                "createTime": null,
                "modifyTime": null,
                "pkHash": "0x382d17374619233978c2f5c8dfc88fea1bb70af52ea824c8ec99982d66b455cd",
                "sealer": "0x3",
                "sealerIndex": 1,
                "transCount": 1
            }]
    }
    ```
- 说明
    - blockNumber：区块高度。
    - blockTimestamp 区块交易时间。
    - createTime：创建时间。
    - modifyTime：修改时间。
    - pkHash：区块哈希。
    - sealer：共识节点序号。
    - sealerIndex：节点序号为index的nodeId。
    - transCount：交易次数。
        
 #### 获取 节点列表
 - 请求
     ```shell
     $ curl http://localhost:8080/weevent/admin/node/nodeList?groupId=1&pageNumber=1&pageSize=10
     ```
 
 - 应答
 
     ```json
     {
      "code":"0",
      "message":"success",
      "data": 
             [{
   
            "nodeId": "543095f2a4a7ec910c4d62fcde2754871c559d375fba9a11aab94cb7c7ae8eef8f55250558a7412d14f11faeb7d31c55cec36746ce5644c749a4674888fe46eb",
            "nodeName": "543095f2a4a7ec910c4d62fcde2754871c559d375fba9a11aab94cb7c7ae8eef8f55250558a7412d14f11faeb7d31c55cec36746ce5644c749a467",
            "pbftView": 23,
            "blockNumber": 5364,
            "createTime": null,
            "modifyTime": null,
            "nodeActive": 1
             }]
     }
     ```
 - 说明
     - nodeId：节点id。
     - nodeName：节点名称。
     - pbftView：PBFT视图。
     - blockNumber：区块高度。
     - nodeActive 运行状态。
     - createTime：创建时间。
     - modifyTime：修改时间。

 #### 获取 节点ip数组
 - 请求
     ```shell
     $ curl http://localhost:8080/weevent/admin/listNodes
     ```
 
 - 应答
 
     ```json
    {
        "data": [
        "10.107.96.107:7000"
        ],
        "code": 0,
        "message": "success"
    }
     ```
 - 说明
     - data：节点ip数组。

#### 获取 节点ip详细信息
 - 请求
     ```shell
     $ curl http://10.107.96.107:8080/weevent/admin/listSubscription?nodeIp=10.107.96.107:7000
     ```
 
 - 应答
 
     ```json
        {
            "data": {
                "10.107.96.107:7000": {
                "5d39d5c1-3aea-48aa-93b2-416624155d0f": {
                "interfaceType": "stomp",
                "notifiedEventCount": "1712",
                "notifyingEventCount": "0",
                "notifyTimeStamp": "2019-11-06 11:28:30",
                "topicName": "com.weevent.rest",
                "subscribeId": "5d39d5c1-3aea-48aa-93b2-416624155d0f",
                "remoteIp": "127.0.0.1",
                "createTimeStamp": "2019-11-05 21:21:57",
                "groupId": "1"
                },
            }
        },
        "code": 0,
        "message": "success"}
     ```
 - 说明
     - data：节点数据的详细信息。