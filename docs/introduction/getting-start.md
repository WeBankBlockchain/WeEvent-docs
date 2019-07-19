## Getting Start

如果是使用`WeEvent`服务，可以选择`Docker`镜像或者通过`Bash`脚本一键安装。详情参见[WeEvent快速安装]()。

如果是参与`WeEvent`开发或者想要体验未正式发布的特性，流程如下：

- 下载[github源码](https://github.com/WeBankFinTech/WeEvent)

  ```bash
  $ git clone https://github.com/WeBankFinTech/WeEvent.git
  ```

  通过IDE（推荐IDEA）打开Gradle工程。

- 配置区块链

  支持`FISCO-BCOS` 1.3和2.0。
  
  - FISCO-BCOS 2.0
  
    在配置文件`./weevent-broker/src/main/resources/fisco.properties`里配置：区块链版本`version=2.0`以及节点访问`Channel`端口`nodes`。
  
    请将节点证书文件`ca.crt`、`node.crt`、`node.key`放到目录下`./weevent-broker/src/main/resources/v2`。
  
  - FISCO-BCOS 1.3
  
    在配置文件`./weevent-broker/src/main/resources/fisco.properties`里配置：区块链版本`version=1.3`以及节点访问`Channel`端口`nodes`。
  
    请将节点证书文件`ca.crt`、`client.keystore`放到目录下`./weevent-broker/src/main/resources`。
  
- 部署系统合约

  运行`./weevent-broker/src/main/java/com/webank/weevent/broker/fisco/util/Web3sdkUtils.java`得到合约地址。
  
  将合约地址配置到`./weevent-broker/src/main/resources/fisco.properties#topic-controller.address`。
  
- 运行服务及代码样例

  启动`Broker`服务`./weevent-broker/src/main/java/com/webank/weevent/BrokerApplication.java`。
  
  直接体验各种样例`./weevent-broker/src/test/java/com/webank/weevent/sample`。
  
- 编译打包服务

  ```bash
  $ cd ./weevent-build; ./package.sh
  Usage:
      package master: ./package.sh --version 1.0.0
      package tag: ./package.sh --tag v1.0.0 --version 1.0.0
      package local: ./package.sh --tag local --version 1.0.0
  ```

  编译环境依赖`git`，`git bash`， `gradle` 4.10，`java` 1.8，`nodejs` 10.16。

- Bash脚本一键安装

  详情参见[WeEvent快速安装]()。


欢迎参与`WeEvent`项目[issues](https://github.com/WeBankFinTech/WeEvent/issues)。

