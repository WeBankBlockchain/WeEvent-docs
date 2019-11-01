## 适配Fabric

`WeEvent`对区块链`Fabric 1.4`的支持。以下安装以`CentOS 7.2`为例。

### 前置条件
- 安装`weevent-broker`, 具体安装步骤[weevent-broker安装](../install/module/broker.html)

  以`weevent-broker`安装到 `/usr/local/weevent/weevent-broker-1.1.0` 目录为例。

- 安装`Fabric 1.4`区块链节点(以官网`Fabric Samples`安装为例)

  具体安装步骤, 请参见[Fabric安装](https://hyperledger-fabric.readthedocs.io/en/latest/install.html)。
  
  以`Fabric 1.4`安装到 `/usr/local/fabric/fabric-sample` 目录为例。

### 修改Fabric相关的配置文件
- 添加安装Fabric节点的证书等文件

  - 配置`WeEvent`访问区块链
  
    ```shell
    $ cd /usr/local/weevent/weevent-broker-1.1.0/
    $ vi /conf/weevent.properties
    ```

    修改`broker.blockchain.type`配置项为：`broker.blockchain.type=fabric`。

  - 添加证书文件
  
    将安装Fabric安装后`/usr/local/fabric/fabric-sample/first-network/crypto-config/`目录下的所有文件，
    
    拷贝到`weevent-broker`安装目录下的 `/usr/local/weevent/weevent-broker-1.1.0/conf/fabric/crypto-config/`里。

  - 区块链节点配置文件fabric.properties
  
    ```shell
    $ vi /conf/fabric/fabric.properties
    ```

    修改`chain.organizations.user.keyfile`配置项为：
    
    `/usr/local/weevent/weevent-broker-1.1.0/conf/fabric/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/xxx_sk`
    
    修改`chain.peer.address`配置项，替换`127.0.0.1`为部署Fabric节点的ip (`peer`端口默认为7051)
    
    修改`chain.orderer.address`配置项，替换`127.0.0.1`为部署Fabric节点的ip (`orderer`端口默认为7050)

- 部署系统合约

  运行脚本`./deploy-fabric-topic-control.sh `部署合约。例如:

  ```shell
  $ ./deploy-topic-control.sh deploy
  begin deploy topic and topicController contract.
  begin add topic into topicController contract.
  deploy contract success. 
  ```
  
### 启动服务

  通过`./broker.sh start`命令启动服务，正常启动如下：

  ```shell
  $ ./broker.sh start
  start broker success (PID=89054)
  add the crontab job success
  ```
  
### RESTful请求样例
- 创建Topic,发布事件

  ```shell
  $ curl http://localhost:8080/weevent/rest/open?topic=com.weevent.test&groupId=mychannel
  $ true
  $ curl http://localhost:8080/weevent/rest/publish?topic=com.weevent.test&groupId=mychannel&content=123456&weevent-format=json
  $ {"topic": "com.weevent.test","eventId": "2cf24dba-59-1124","status": "SUCCESS"}
  ```
  
### 代码样例
  
  `API`里所有的`groupId`就是`Fabric`的`channelname`, 以发布事件为例：
    
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
                    "mychannel").getBody();
            System.out.println(result);
            // publish event to topic "com.weevent.test"
            SendResult sendResult = rest.getForEntity("http://localhost:8080/weevent/rest/publish?topic={}&groupId={}&content={}",
                    SendResult.class,
                    "com.weevent.test",
                    "mychannel",
                    "hello weevent".getBytes(StandardCharsets.UTF_8)).getBody();
            System.out.println(sendResult.getStatus());
            System.out.println(sendResult.getEventId());
        } catch (RestClientException e) {
            e.printStackTrace();
        }
    }
}
```

