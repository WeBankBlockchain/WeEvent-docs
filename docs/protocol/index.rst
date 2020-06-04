接入说明
========================================= 
WeEvent服务用Java Spring Boot框架实现。
提供了RESTful、JsonRPC、STOMP风格的协议接入，为方便物联网IOT设备的使用，提供了内置MQTT协议的支持。
同时为Java业务接入提供了Java SDK客户端，SDK不依赖Spring框架。

当然也可以不依赖任何服务，直接以jar包将核心的发布订阅功能集成到业务服务里。

.. toctree::
   :maxdepth: 1

   /protocol/weevent-core-sdk
   /protocol/restful
   /protocol/jsonrpc
   /protocol/stomp
   /protocol/mqtt
   /protocol/weevent-client-sdk   
   /protocol/errorcode