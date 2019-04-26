## 版本历史
### 1.0.0版本

- 支持`FISCO-BCOS` 2.0，通过群组解决数据隐私问题。

- 移除`Mosquitto`的依赖，`MQTT`协议内置处理。

  Coming soon.

### 0.9.0版本

- `Broker`服务：提供事件主题`Topic`的`CRUD`管理，基于事件的发布订阅`Publish`/`Subscribe`等功能。
- 适配多协议：支持`RESTful`、`JsonRPC`、`STMOP`、`MQTT`等4种接入协议。
- `SDK`及`Demo`：一个符合`JMS`规范的客户端jar包，`Java SDK`和多种协议的接入代码样例。
- 高可用性：通过`Zookeeper`实现服务的主备切换，使用`Nginx`只负责集群的负载均衡。
- `Governance`管理端：通过`Web`页面可以查看对应`FISCO-BCOS`区块链及其节点信息，可以管理和维护事件主题`Topic`。
- 效率工具：包括一键打包和安装脚本。用户默认方式安装，就可以体验到`WeEvent`的核心功能。