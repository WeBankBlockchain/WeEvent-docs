## 架构说明

`WeEvent`主要使用[Spring Boot](https://spring.io/projects/spring-boot)框架开发。分为`Broker`和`Governance`两个子模块，`Broker`负责事件发布和订阅以及访问区块链`FISCO-BCOS`，`Governance`提供一个`Web`管理端实现事件治理和流计算。集群模式下使用`Nginx`实现负载均衡。

### 模块说明

- Nginx

  `WeEvent`服务对外统一的访问入口，负责服务请求的负载均衡。

- Broker

  `WeEvent`的事件代理模块，提供核心的事件发布订阅`Pub`/`Sub`和`Topic`管理功能。

  该模块使用`Redis`来缓存事件，使用`Zookeeper`记录当前所有实例的信息。

- Governance

  `WeEvent`的事件治理模块，提供一个`Web`管理端。支持区块链信息浏览、`Topic`事件治理、事件流计算等。

  其中，使用`Mysql`持久化相关数据。

- FISCO-BCOS

    `WeEvent`的事件永久存储在区块链上。推荐使用[FISCO-BCOS 2.1.0](https://github.com/FISCO-BCOS/FISCO-BCOS)，也支持`Fabric 1.4`。

### 架构设计

![](../image/WeEventArchitecture.png)

