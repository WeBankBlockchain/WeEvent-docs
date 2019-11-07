## 配置Fabric

`WeEvent`支持区块链`Fabric 1.4`。以下安装以`CentOS 7.2`为例。

### 前置条件
- 安装`weevent-broker`, 具体安装步骤[weevent-broker章节](../install/module/broker.html)

  以`weevent-broker`安装到 `/usr/local/weevent/weevent-broker-1.1.0` 目录为例。
  

- 安装`Fabric 1.4`区块链节点(以官网`Fabric Samples`安装为例), 

  具体安装步骤, 请参见[Fabric安装](https://hyperledger-fabric.readthedocs.io/en/latest/install.html)。
  
  以`Fabric 1.4`安装到 `/usr/local/fabric/fabric-sample` 目录为例。

### 修改Fabric的配置文件
- 配置`Fabric`节点的证书文件

  - 配置`WeEvent`访问区块链
  
    ```shell
    $ cd /usr/local/weevent/weevent-broker-1.1.0/
    $ vi ./conf/weevent.properties
    ```

    修改`broker.blockchain.type`配置项为：`broker.blockchain.type=fabric`。

  - 配置证书文件
  
    将`Fabric`目录`/usr/local/fabric/fabric-sample/first-network/crypto-config/`所有文件，
    
    拷贝到`weevent-broker`安装目录下`/conf/fabric/crypto-config/`里。

  - 区块链节点配置文件fabric.properties
  
    ```shell
    $ vi ./conf/fabric/fabric.properties
    ```

    - 修改配置项
    ```
    chain.organizations.user.keyfile=fabric/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/xxx_sk
    chain.peer.address=127.0.0.1:7051
    chain.orderer.address=127.0.0.1:7050
    ```
    
    替换`127.0.0.1`为部署Fabric节点的ip (`peer`端口默认为7051,`orderer`端口默认为7050)
    
- 部署系统合约

  运行脚本`./deploy-fabric-topic-control.sh `部署合约。例如:

  ```shell
  $ ./deploy-fabric-topic-control.sh deploy
  begin deploy topic and topicController contract.
  begin add topic into topicController contract.
  deploy contract success. 
  ```
  
### 重启服务

- 停止服务

   ```shell
   $ ./broker.sh stop
   stop broker success
   remove the crontab job success
   ``` 
 
- 启动服务

   ```shell
   $ ./broker.sh start
   start broker success (PID=89054)
   add the crontab job success
   ```
  
### RESTful请求样例
- 创建Topic,发布事件

  ```shell
  $ curl "http://localhost:8080/weevent/rest/open?topic=com.weevent.test&groupId=mychannel"
  [1] 16414
  $ true
  $ curl "http://localhost:8080/weevent/rest/publish?topic=com.weevent.test&groupId=mychannel&content=123456&weevent-format=json"
  $ {"status":"SUCCESS","topic":"com.weevent.test","eventId":"317e7c4c-301009-6705"}
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
            Boolean result = rest.getForEntity("http://localhost:8080/weevent/rest/open?topic={topic}&groupId={groupId}",
                    Boolean.class,
                    "com.weevent.test",
                    "mychannel").getBody();
            System.out.println(result);
            // publish event to topic "com.weevent.test"
            SendResult sendResult = rest.getForEntity("http://localhost:8080/weevent/rest/publish?topic={topic}&groupId={groupId}&content={content}",
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

