## 架构说明

`WeEvent`主要使用[Spring Boot](https://spring.io/projects/spring-boot)框架开发。既支持单独的`weevent-core.jar `包集成，也支持以独立服务的方式部署。

以独立服务部署的方式中，有`Broker`、`Governance`、`Processor`、`API Gateway`等子服务。

### 子服务简介

- Broker

  `WeEvent`的事件代理模块，提供核心的事件发布订阅`Pub`/`Sub`以及`Topic`管理功能。

  `Broker`就是`weevent-core.jar `的服务化，以支持各种协议接入。

- Governance  

  `WeEvent`的事件治理模块，提供一个`Web`管理端。支持区块链信息浏览、`Topic`事件治理、流计算等。

  使用数据库持久化相关数据，支持`H2`和`Mysql`。

- Processor

    是`Governance`实时流计算功能中，规则引擎的分布式运行容器。
    
- API Gateway

    `WeEvent`服务对外统一的访问入口，负责接入请求的负载均衡、限流、熔断等。
    
- FISCO-BCOS

    `WeEvent`的事件永久存储在区块链上。推荐使用[FISCO-BCOS](https://github.com/FISCO-BCOS/FISCO-BCOS)，也支持`Fabric 1.4`。

### 系统架构

![](../image/WeEventArchitecture.png)

