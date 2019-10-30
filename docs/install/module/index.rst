详细安装
----------
本章节主要介绍WeEvent模块的详细安装，用户可以根据自身需要，继续探索和学习详细模块，一般在生产环境需实例部署，需要使用该方式安装。
Broker是WeEvent的核心子模块，负责事件的发布订阅以及对区块链FISCO-BCOS的访问。支持RESTful、JsonRPC、STOMP、MQTT多种接入协议，也提供了Java SDK。


Governance为用户提供一个事件治理的Web管理端。支持事件治理、区块链节点分析、系统监控预警等。


Nginx 实现WeEvent服务的负载均衡。服务支持主流的Linux发行版，如CentOS、Ubuntu 、Redhat。


.. toctree::
   :maxdepth: 1

   /install/module/broker
   /install/module/governance
   /install/module/nginx
   /install/module/processor