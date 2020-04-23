## RESTful
`RESTful`作为一种软件架构风格、设计风格，在Web应用上非常流行。基于这个风格设计的软件可以更简洁，`REST` 接口可以直接在浏览器上测试，给开发和测试过程带来很多便利。

`RESTful`都是对`STOMP`协议的一个补充，支持`WeEvent`主题的`CRUD`管理等功能以及事件发布。

### 协议说明

- `HTTP`方法可以使用`GET`和`POST`，并且统一使用`UTF8`字符集。

- 输入参数为`Query String`格式，注意使用`UrlEncode`。

- 应答的`HTTP`状态码是 `200`，输出参数统一是`json`格式，`Content-type: application/json`。

- 调用异常时，返回异常信息`BrokerException`。

  ```json
  {"code": 200202, "message":"the transaction does not correctly executed."}
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
            String topic = "com.weevent.test";
            ResponseEntity<BaseResponse<Boolean>> rsp = rest.exchange("http://localhost:7000/weevent-broker/rest/open?topic={topic}&groupId={groupId}", HttpMethod.GET, null, new ParameterizedTypeReference<BaseResponse<Boolean>>() {
            }, topic, WeEvent.DEFAULT_GROUP_ID);
            System.out.println(rsp.getBody().getData());

            // publish event to topic "com.weevent.test"
            SendResult sendResult = rest.getForEntity("http://localhost:7000/weevent-broker/rest/publish?topic={topic}&groupId={groupId}&content={content}",
                    SendResult.class,
                    topic,
                    WeEvent.DEFAULT_GROUP_ID,
                    "hello WeEvent".getBytes(StandardCharsets.UTF_8)).getBody();
            System.out.println(sendResult);
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
  $ curl "http://localhost:8080/weevent-broker/rest/open?topic=com.weevent.test&groupId=1"
  ```


- 应答

  ```json
  {
      "code": "0",
      "message": "success",
      "data": "true"
  }
  ```


```eval_rst
.. note:: 
    - topic：主题。ascii值在[32,128]之间。支持通配符按层次订阅，因'+'、'#'为通配符的关键字故不能为topic的一部分，详情参见[MQTT通配符](http://public.dhe.ibm.com/software/dw/webservices/ws-mqtt/mqtt-v3r1.html) 。
    - groupId： FISCO-BCOS群组Id。默认的群组1可以不传，其他接口类似。
    - 重复open返回true 。
```
#### 关闭Topic
- 请求

  ```shell
  $ curl "http://localhost:8080/weevent-broker/rest/close?topic=com.weevent.test&groupId=1"
  ```


- 应答

  ```json
  {
      "code": "0",
      "message": "success",
      "data": "true"
  }
  ```

#### 检查Topic是否存在
- 请求

  ```shell
  $ curl "http://localhost:8080/weevent-broker/rest/exist?topic=com.weevent.test&groupId=1"
  ```


- 应答

  ```json
  {
      "code": "0",
      "message": "success",
      "data": "true"
  }
  ```

#### 发布事件
- 请求

  ```shell
  $ curl "http://localhost:8080/weevent-broker/rest/publish?topic=com.weevent.test&groupId=1&content=123456&weevent-format=json"
  ```


- 应答

  ```json
  {
      "topic": "com.weevent.test",
      "eventId": "2cf24dba-59-1124",
      "status": "SUCCESS"
  }
  ```


```eval_rst
.. note::
  - content ：用户自定义数据。需要特别注意content需进行UrlEncode编码，GET方法支持的QueryString最大长度为1024字节。
  - weevent-json:可选参数。用户自定义拓展，以weevent-开头。
  - status：SUCCESS，说明是发布成功，eventId是对应的事件ID。
```
#### 获取Event详情

- 请求

  ```shell
  $ curl "http://localhost:8080/weevent-broker/rest/getEvent?eventId=2cf24dba-59-1124&groupId=1"
  ```

- 应答

  ```json
  {
      "code":0,
      "message":"success",
      "data": {
          "topic": "hello",
          "content": "MTIzNDU2",
          "extensions": {
              "weevent-format": "json"
          },
          "eventId": "2cf24dba-59-1124"
      }
  }
  ```
```eval_rst
.. note::
  - eventId：事件ID
  - content：事件内容，MTIzNDU2是123456的Base64之后的值。
  - extensions：用户自定义拓展数据。
```
 
```eval_rst
.. important::
   以下管理端接口，业务程序里一般用不到，可以直接安装Goverance模块来使用这部分功能。
```
#### 当前Topic列表
- 请求

  ```shell
  $ curl "http://localhost:8080/weevent-broker/rest/list?pageIndex=1&pageSize=10&groupId=1"
  ```


- 应答

  ```json
  {
      "code":0,
      "message":"success",
      "data": {
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
  }
  ```


```eval_rst
.. note::
  - total：Topic的总数量。
  - pageIndex：表示查询第几页，从0开始 。
  - pageSize：分页大小（0,100），超出这默认每页10个数据。
  - topicInfoList：Topic详细信息列表。
```


#### 查询某个Topic详情
- 请求

  ```shell
  $ curl "http://localhost:8080/weevent-broker/rest/state?topic=com.weevent.test&groupId=1"
  ```


- 应答

  ```json
  {
      "code":0,
      "message":"success",
      "data": {
          "topicName": "com.weevent.test",
          "topicAddress": "0x171befab4c1c7e0d33b5c3bd932ce0112d4caecd",
          "senderAddress": "0x64fa644d2a694681bd6addd6c5e36cccd8dcdde3",
          "createdTimestamp": 1548328570965,
          "sequenceNumber": 9,
          "blockNumber": 2475
      }
  }
  ```


```eval_rst
.. note::
  - topicName ：  事件名称
  - topicAddress ：  Topic区块链上的合约地址
  - createdTimestamp  ：Topic创建的时间
  - sequenceNumber：已发布事件数。
  - blockNumber：最新已发布事件的区块高度。
```  
#### 获取群组列表
- 请求

  ```shell
  $ curl "http://localhost:8080/weevent-broker/admin/listGroup"
  ```


- 应答

  ```json
  {
      "code": 0,
      "message": "success",    
      "data":[
          "1","2"
      ]
  }
  ```

#### 获取 节点ip数组
 - 请求
     ```shell
     $ curl "http://localhost:8080/weevent-broker/admin/listNodes"
     ```

 - 应答

     ```json
    {
        "code": 0,
        "message": "success",
        "data": [
        	"127.0.0.1:7000"
        ]
    }
    ```
```eval_rst
.. note::
    - data：节点ip数组。
```

#### 获取订阅列表 
 - 请求
     ```shell
     $ curl "http://localhost:8080/weevent-broker/admin/listSubscription?nodeInstances=127.0.0.1:7000"
     ```

 - 应答

    ```json
    {
    	"code": 0,
    	"message": "success",
    	"data": {
    		"127.0.0.1:7000": [{
    			"interfaceType": "stomp",
    			"notifiedEventCount": "0",
    			"notifyingEventCount": "0",
    			"notifyTimeStamp": "2020-03-16 15:09:22",
    			"topicName": "com.weevent.test",
    			"subscribeId": "041b2b7b-1ec3-47c9-9cf3-2f40a0a17f5a",
    			"remoteIp": "192.168.0.107",
    			"createTimeStamp": "2020-03-16 15:09:22",
    			"groupId": "1"
    		}]
    	}
    }
    ```
```eval_rst
.. note::
     - interfaceType：监听请求类型 RESTful、JsonRPC、MQTT 、STOMP。
     - notifyingEventCount：待通知事件的数量。
     - notifiedEventCount：已通知事件数量
     - notifyTimeStamp：最近通知事件时间戳。
     - subscribeId：订阅ID
     - topicName ：事件主题。
```
#### 获取版本信息
- 请求
    ```shell
    $ curl "http://localhost:8080/weevent-broker/admin/getVersion"
    ```

- 应答

    ```json
    {
     "code": 0,
     "message":"success",
     "data": 
            {
                "weEventVersion": "1.1.0",
                "gitCommitTimeStamp": "2019-09-16 18:01:23",
                "gitBranch": "master",
                "gitCommitHash": "a5b022b"
            }
    }
    ```
```eval_rst
.. note::
    - weEventVersion：WeEvent版本号。
    - gitCommitTimeStamp：WeEvent最近一次提交git的时间。
    - gitBranch：WeEvent构建分支。
    - gitCommitHash：WeEvent最近一次提交git的CommitHash。
```

#### 获取节点个数、区块数量、交易数量
- 请求
    ```shell
    $ curl "http://localhost:8080/weevent-broker/admin/group/general?groupId=1"
    ```

- 应答

    ```json
    {
    	"code": 0,
    	"message": "success",
    	"data": {
    		"nodeCount": "4",
    		"latestBlock": "100",
    		"transactionCount": "5260"
    	}
    }
    ```
```eval_rst
.. note::
    - nodeCount：节点个数。
    - latestBlock：区块数量。
    - transactionCount：交易数量。
```
#### 获取区块链交易列表
- 请求
    ```shell
    $ curl "http://localhost:8080/weevent-broker/admin/transaction/transList?groupId=1&pageNumber=1&pageSize=10"
    ```

- 应答

    ```json
    {
    	"code": 0,
    	"message": "success",
    	"data": {
    		"total": 1,
    		"pageIndex": 1,
    		"pageSize": 10,
    		"pageData": [{
    			"blockNumber": 5364,
    			"blockTimestamp": "2019-10-15 14:48:01",
    			"createTime": null,
    			"modifyTime": null,
    			"transFrom": "0x64fa644d2a694681bd6addd6c5e36cccd8dcdde3",
    			"transHash": "0xb3585cf385a595e5af425d360693e6759d8db5c1a98ebb46277b38c014ec8626",
    			"transTo": "0xa40c864c28ee8b07dc2eeab4711e3161fc87e1e2"
    		}]
    	}
    }
    ```
```eval_rst
.. note::
    - total：当前区块交易总条数。
    - pageIndex：页码。
    - pageSize：页面展示条数。
    - blockNumber：区块高度。
    - blockTimestamp：区块交易时间。
    - createTime：创建时间。
    - modifyTime：修改时间。
    - transFrom： 发送者的地址。
    - transHash：交易哈希。
    - transTo：接收者的地址。
```    

#### 获取交易哈希列表
- 请求
    ```shell
    $ curl "http://localhost:8080/weevent-broker/admin/block/blockList?groupId=1&pageNumber=1&pageSize=10"
    ```

- 应答

    ```json
    {
    	"code": 0,
    	"message": "success",
    	"data": {
    		"total": 1,
    		"pageIndex": 1,
    		"pageSize": 10,
    		"pageData": [{
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
    }
    ```
```eval_rst
.. note::
    - total: 区块总数。
    - pageIndex：页码。
    - pageSize：页面展示条数。
    - blockNumber：区块高度。
    - blockTimestamp：区块交易时间。
    - createTime：创建时间。
    - modifyTime：修改时间。
    - pkHash：区块哈希。
    - sealer：共识节点序号。
    - sealerIndex：节点序号为index的nodeId。
    - transCount：交易次数。
```      
 #### 获取区块链节点列表
 - 请求
     ```shell
     $ curl "http://localhost:8080/weevent-broker/admin/node/nodeList?groupId=1&pageNumber=1&pageSize=10"
     ```

 - 应答

     ```json
     {
     	"code": 0,
     	"message": "success",
     	"data": {
     		"total": 1,
     		"pageIndex": 1,
     		"pageSize": 10,
     		"pageData": [{
     			"nodeId": "543095f2a4a7ec910c4d62fcde2754871c559d375fba9a11aab94cb7c7ae8eef8f55250558a7412d14f11faeb7d31c55cec36746ce5644c749a4674888fe46eb",
     			"nodeName": null,
     			"pbftView": 23,
     			"nodeType": "sealer",
     			"blockNumber": 5364,
     			"createTime": null,
     			"modifyTime": null,
     			"nodeActive": 1
     		}]
     	}
     }
     ```
```eval_rst
.. note::
     - total: 节点总数。
     - pageIndex：页码。
     - pageSize：页面展示条数。
     - nodeId：节点id。
     - nodeName：节点名称。
     - pbftView：PBFT视图。
     - nodeType：节点类型。
     - blockNumber：区块高度。
     - nodeActive：节点运行状态。
     - createTime：创建时间。
     - modifyTime：修改时间。
```