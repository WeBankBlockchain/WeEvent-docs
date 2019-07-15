## 架构说明

`WeEvent`主要使用[Spring Boot](https://spring.io/projects/spring-boot)框架开发。分为`Broker`和`Governance`两个子模块，`Broker`负责事件发布和订阅以及访问区块链`FISCO-BCOS`，`Governance`提供一个`Web`管理端实现事件治理。整个服务使用`Nginx`实现负载均衡。

### 模块说明

- Nginx

  `WeEvent`服务对外统一的访问入口，负责服务请求的负载均衡。

- Broker

  `WeEvent`的事件代理模块，提供核心的事件发布订阅`Publish`/`Subscribe`和`Topic`管理功能。

  该模块使用`Redis`来缓存事件，使用`Zookeeper`进行可能的主备服务切换。

- Governance

  `WeEvent`的事件治理模块，提供一个`Web`管理端。支持事件治理、区块链节点分析、系统监控预警等。

  其中，使用`Mysql`数据库存储`Topic`的统计数据，根据业务需要使用[WeBase](https://github.com/WeBankFinTech/WeBASE)进行区块链管理。

- FISCO-BCOS

    `WeEvent`的事件永久存储在区块链[FISCO-BCOS](https://github.com/FISCO-BCOS/FISCO-BCOS)上。

### 架构设计

![](../image/WeEventArchitecture.png)

