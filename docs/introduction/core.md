## 核心概念

`WeEvent`是一个基于区块链实现的事件中间件服务，面向用户提供事件发布订阅`Publish/Subscribe`功能。发布到`WeEvent`上的事件永久存储，不可篡改，支持事后跟踪和审计。
生产者`Producer`通过`WeEvent`代理服务发布事件，事件内容会被记录到区块链如`FISCO-BCOS`上，消费者`Consumer`从`WeEvent`订阅事件。订阅成功后，只要生产者发布事件，消费者都会及时得到通知。

![](../image/WeventTopView.png)  

`WeEvent`提供两种部署方案：一种是直接将`Jar`包集成进业务进程，集成过程参见[集成weevent-core.jar](../protocol/weevent-core-sdk.html)；一种是以独立服务部署，业务通过接入这个代理服务访问各种功能，部署过程参见[WeEvent快速安装](../install/quickinstall.html)。两种方式各有优劣，可以自由选择。`WeEvent`独立服务有多种接入方式，生产者`Producer`和消费者`Consumer`可以是后台服务、前端网页、`IoT`设备、甚至单片机。

### 事件（Event）  
事件`Event`可以简单理解成业务层面的一个消息。一般是终端用户或设备触发。

通常一个事件包含四个部分的信息，事件`ID`、内容、可选的自定义属性以及关联的主题。

事件内容对`WeEvent`是透明的，业务可以存放任何数据，例如字符型的`Json`、`XML`，或者二进制的`Protocol Buffer`等。在`Java`中映射为字节数组`byte[]`，完整的`Java`映射类参见[WeEvent.java](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-client/src/main/java/com/webank/weevent/client/WeEvent.java)。

### 主题（Topic）  
业务上一般把数据结构相同，属于同类型的事件归属于同一主题`Topic`。

每个主题`Topic`逻辑上都有彼此独立的队列。主题之间是完全隔离的，发布、存储、通知都不会互相影响。  

### 发布事件（Publish）  
生产者`Producer`往某个主题发布事件，`WeEvent`会将事件永久存储在区块链`FISCO-BCOS`上，不可篡改，支持事后审核。

事件发布成功后`WeEvent`会返回一个事件ID`EventID` ，事件ID代表该事件且具有唯一性。

### 订阅事件（Subscribe）  
消费者`Consumer`订阅某个主题后，当有生产者往该主题发布事件，消费者会及时收到事件通知。

订阅成功时`WeEvent`会返回一个的订阅ID`SubscriptionID`，代表完成一次订阅，订阅ID具有唯一性。

除了实时订阅最新的事件，`WeEvent`也支持从最开始或者从某个历史事件`EventID`之后启动订阅。

### 取消订阅（Unsubscribe）  
消费者通过订阅ID`SubscriptionID` 取消一个订阅后，不会再收到该主题的事件通知。
