## WeEvent 介绍
`WeEvent`是一个基于区块链实现的事件中间件，支持`RESTful`、`RPC`、`JsonRPC`、`STOMP`多种方式，`PUB/SUB`是基于`Topic`进行的消息路由。事件发布到`WeEvent`上，将永久存储和不可篡改，并支持事后跟踪和审计。并且有`Governance`管理平台，支持事件治理、区块链节点分析、系统监控预警。


## 特性
- 分布式存储：事件发布到`WeEvent`上，将永久存储且不可篡改，并支持事后跟踪、审计。
- 全链路加密：整个通信链路以RSA，AES加密，保证数据传输的安全。
- 支持多种通信模式：WeEvent支持`RESTful`、`JsonRPC`、`STMOP`、`MQTT`等4种接入协议，以满足用户不同场景下的需求。
- 支持多种设备接入协议：支持设备使用`MQTT`、`HTTPS`协议接入`WeEvent`。
- 可视化操作：支持事件治理、区块链节点分析、系统监控预警。


## 快速入门
- 搭建服务

快速搭建一套WeEvent的服务，请参考[文档](http://)。通过一键部署的WeEvent的服务，用户可以快速体验、开发、测试。

- 体验WeEvent

  在控制台输入如下命令，体验创建主题和发布事件。

  ```shell
  $ curl http://localhost:8080/weevent/rest/open?topic=com.weevent.test
  true
  $ $ curl http://localhost:8080/weevent/rest/publish?topic=com.weevent.test&content=123456
  {
      "topic": "com.weevent.test",
      "eventId": "3263663234646261-1-1065",
      "status": "SUCCESS"
  }
  ```


## 贡献说明
WeEvent爱贡献者！请阅读我们的贡献文档，了解如何贡献代码，并提交你的贡献。

希望在您的帮助下WeEvent继续前进。


## 社区
- 联系我们：weevent@webank.com



