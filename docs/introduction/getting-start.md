## Getting Start

### 使用WeEvent服务

选好所需要的版本，一般推荐[最新版本](https://github.com/WeBankFinTech/WeEvent/releases)。选择`Docker`镜像或者通过`Bash`脚本一键安装。详情参见[WeEvent快速安装](../install/quickinstall.html)。

### 参与WeEvent开发

想参与`WeEvent`开发或者体验未正式发布的特性，需要从源码开始，流程如下：

- 下载[github源码](https://github.com/WeBankFinTech/WeEvent)

  ```bash
  $ git clone https://github.com/WeBankFinTech/WeEvent.git
  ```

  通过IDE（推荐IDEA）打开工程。

- 配置区块链

  支持`FISCO-BCOS 1.3`和`2.0 ` ，以及`Fabric 1.4`。

  - FISCO-BCOS 2.0

    在配置文件`./weevent-broker/src/main/resources/fisco.properties`里配置：区块链版本`version=2.0`以及节点访问`channel`端口`nodes=`。

    请将节点访问证书文件`ca.crt`、`node.crt`、`node.key`放到目录下`./weevent-broker/src/main/resources/v2`。

  - FISCO-BCOS 1.3

    在配置文件`./weevent-broker/src/main/resources/fisco.properties`里配置：区块链版本`version=1.3`以及节点访问`Channel`端口`nodes=`。

    请将节点访问证书文件`ca.crt`、`client.keystore`放到目录下`./weevent-broker/src/main/resources`。

  - Fabric 1.4
    具体内容详见[Fabric章节](https://weeventdoc.readthedocs.io/zh_CN/latest/advanced/fabric.html)

- 部署系统合约

  通过运行`./weevent-broker/src/main/java/com/webank/weevent/broker/fisco/util/Web3sdkUtils.java`来部署合约。

- 运行服务及代码样例

  启动`Broker`服务`./weevent-broker/src/main/java/com/webank/weevent/BrokerApplication.java`。

  然后体验各种功能样例`./weevent-broker/src/test/java/com/webank/weevent/sample`。

- 编译打包服务

  ```bash
  $ cd ./weevent-build; ./package.sh
  Usage:
      package master: ./package.sh --version 1.1.0
      package tag: ./package.sh --tag v1.1.0 --version 1.1.0
      package local: ./package.sh --tag local --version 1.1.0
  ```

  编译环境依赖`git`，`git bash`， `gradle 4.10`，`java 1.8`，`nodejs 10.16`。

- 安装包一键安装服务

  详情参见[WeEvent快速安装](../install/quickinstall.html)。


欢迎参与`WeEvent`项目[issues](https://github.com/WeBankFinTech/WeEvent/issues)。

