详细安装
----------
本章节主要介绍WeEvent模块的详细安装，用户可以根据自身需要，继续探索和学习详细模块，一般在生产环境需实例部署，需要使用该方式安装。

Broker是WeEvent的核心子模块，负责事件的发布订阅以及对区块链FISCO-BCOS的访问。

Governance为用户提供一个事件治理的Web管理端。支持事件治理、区块链节点分析、系统监控预警、大文件传输等。

Processor为用户提供时序流分析和事件联动。

Gateway实现接入请求的负载均衡、限流和熔断。

服务支持主流的Linux发行版，如CentOS、Ubuntu 、Redhat。


.. toctree::
   :maxdepth: 1

   /install/module/broker
   /install/module/governance
   /install/module/processor
   /install/module/gateway