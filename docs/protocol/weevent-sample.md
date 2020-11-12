## WeEvent-Sample
`WeEvent-Sample`以命令行的形式，提供创建`topic`，发布订阅，查看容量等功能的体验。

### 前置条件

已经安装、部署成功，并启动`WeEvent`服务，`WeEvent`服务[快速安装](../install/quickinstall.md)，

### 下载`WeEvent-Sample`
  
  ```shell
  $ cd /tmp/
  $ git clone https://github.com/WeBankFinTech/WeEvent-Sample.git
  ```

### 配置与构建
- 修改配置

  修改连接broker服务的url地址 (快速安装部署`WeEvent`服务的url)

  ```shell
  $ cd /tmp/WeEvent-Sample
  $ vi /tmp/WeEvent-Sample/src/main/resources/application.properties
  ```

- 构建
  ```shell
  $ chmod +x *.sh gradlew && dos2unix *.sh gradlew
  $ ./build.sh
  ```
  
  构建成功后，输出有如下关键字:
  ```
  begin to build WeEvent-Sample.
  build WeEvent-Sample success.
  ```
  
### 命令行方式使用

- 查看群组`groupId`列表
  ```shell
  $ ./command.sh listGroup
  $ listGroup result: {"code":0,"message":"success","data":["1","2","3"]}
  ```

- 创建`topic`
  
  ```shell
  $ ./command.sh open 1 com.weevent.test
  $ open topic:[com.weevent.test] success.
  ```
  - 命令行参数说明：
  
    - 1 : 代表群组`id`，参考查看群组`groupId`列表返回的群组列表
    - com.weevent.test : `topic`名称
    
- 订阅`topic`
  
  ```shell
  $ ./command.sh subscribe 1 com.weevent.test
  $ subscribe topic:[com.weevent.test] success, subscribeId:9904ac30-2cef-48c7-b72a-fc7c460cc3ed
  ```
  
- 发布事件
  
  说明：发布和订阅存在于不同的客户端，所以这里需要重新开一个新的窗口。
  
  ```shell
  $ cd /tmp/WeEvent-Sample
  $ ./command.sh publish 1 com.weevent.test hello
  $ publish event by topic:[com.weevent.test] success, sendResult:SendResult(status=SUCCESS, topic=com.weevent.test, eventId=317e7c4c-9-42)
  ```
 
   订阅方的客户端会收到发布的事件
   ```shell
   $ received event: WeEvent{topic='com.weevent.test', content.length=5, eventID='317e7c4c-9-42', extensions={weevent-plus={"timestamp":1596794003341,"height":42,"txHash":"0x74c00fe18074a023eb32331737eeef49e28d8c05058392b02f1d0ae114cef45a","sender":"0x64fa644d2a694681bd6addd6c5e36cccd8dcdde3"}}}
   $ event content: hello
   ```
  
  - 命令行参数说明：
    
    - 1 : 代表群组`id`，参考查看群组`groupId`列表返回的群组列表
    - com.weevent.test : `topic`名称
    - hello : 发布事件的内容

- 查看事件详情
  ```shell
  $ ./command.sh getEvent 1 317e7c4c-9-42
  $ getEvent success, event:WeEvent{topic='com.weevent.test', content.length=5, eventID='317e7c4c-9-42', extensions={weevent-plus={"timestamp":1596794003341,"height":42,"txHash":"0x74c00fe18074a023eb32331737eeef49e28d8c05058392b02f1d0ae114cef45a","sender":"0x64fa644d2a694681bd6addd6c5e36cccd8dcdde3"}}}
  ```
   
  - 命令行参数说明：
      
    - 1 : 代表群组`id`，参考查看群组`groupId`列表返回的群组列表
    - 317e7c4c-9-42 : `eventId`，发布事件成功后，会返回该事件的`eventId`
      
- 查看`topic`详情
  ```shell
  $ ./command.sh status 1 com.weevent.test
  $ get topic info success, topicInfo:TopicInfo(topicName=com.weevent.test, topicAddress=null, senderAddress=0x64fa644d2a694681bd6addd6c5e36cccd8dcdde3, createdTimestamp=1596681636529, sequenceNumber=9, blockNumber=42, lastTimestamp=1596794003341, lastSender=null, lastBlock=null)
  ```
 
- 查看节点容量
  ```shell
  $ ./command.sh general 1
  $ general result: {"code":0,"message":"success","data":{"nodeCount":4,"transactionCount":42,"latestBlock":42}}
  ```
  
- 文件传输
    
  说明：发送方和订阅方存在于不同的客户端，所以这里需要重新开一个新的窗口。
    
  ```shell
  $ cd /tmp/WeEvent-Sample
  $ ./command.sh sendFile 1 com.weevent.test build.sh
  $ sendFile success, fileChunksMeta:{"fileId":"53a12e3305b0464da0a737e00516fd49","fileName":"build.sh","fileSize":193,"fileMd5":"bb8a5115bf7e22f041f8f4a56fcac591","topic":"20201111","groupId":"1","overwrite":true,"chunkSize":1048576,"chunkNum":1,"chunkStatus":"AQ==","startTime":1605148408}
  ```
    
  订阅方的客户端会收到发送方的文件
  ```shell
  $ ./command.sh receiveFile 1 com.weevent.test
  $ ./received/build.sh
  ```