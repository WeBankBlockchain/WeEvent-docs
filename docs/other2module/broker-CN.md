## Broker 介绍
`Broker`是一个基于区块链实现的事件中间件，支持`RESTful`、`RPC`、`JsonRPC`、`STOMP`多种方式，`PUB/SUB`是基于`Topic`进行的消息路由。事件发布到`WeEvent`上，将永久存储和不可篡改，并支持事后跟踪和审计。


## 特性
- Broker服务：提供事件`Topic`的`CRUD`管理、事件`Pub/Sub`等功能；
- 适配多协议：`Broker`支持`RESTful`、`JsonRPC`、`STMOP`、`MQTT `4种接入协议；
- Java SDK：提供一个符合`JMS`规范的Jar包，封装了`Broker`服务提供的各种能力；
- 高可用性：完善的主备服务切换和负载均衡能力；
- 丰富样例：各种接入方式的使用代码样例。


## 快速入门
- 安装前置依赖

区块链是`Broker`的前置依赖，用户需要提前安装，具体操作见[文档](https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-1.3/docs/tools/index.html)。

- 搭建服务

快速搭建一套`Broker`的服务，请参考[文档](http://)。通过一键部署的`Broker`的服务，用户可以快速体验、开发、测试。

- 体验订阅

用户可以下载[Client](http://)，体验创建`Topic`、发布`Event`、订阅事件。

## 贡献说明
WeEvent爱贡献者！请阅读我们的贡献[文档](http://)，了解如何贡献代码，并提交你的贡献。

希望在您的帮助下WeEvent继续前进。


## 社区
- 联系我们：weevent@webank.com



