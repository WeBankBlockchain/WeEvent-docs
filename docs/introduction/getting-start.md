## Getting Start

### 使用WeEvent服务

选好所需要的版本，一般推荐[最新版本](https://github.com/WeBankFinTech/WeEvent/releases)。选择`Docker`镜像或者通过`Bash`脚本一键安装。详情参见[WeEvent快速安装](../install/quickinstall.html)。

### 参与WeEvent开发

想参与`WeEvent`开发或者体验未正式发布的特性，需要从源码开始，流程如下：

- 下载[github源码](https://github.com/WeBankFinTech/WeEvent)

  ```bash
  $ git clone https://github.com/WeBankFinTech/WeEvent.git
  ```

  通过`IDE`打开`Gradle`工程，推荐`IntelliJ IDEA`。

- 配置区块链

  默认支持`FISCO-BCOS 2.x`，也可以通过配置切换到`Fabric 1.4`。

  - FISCO-BCOS 2.x

    在配置文件`./weevent-broker/src/main/resources/fisco.properties`里配置：

    区块链版本`version=2.0`

    节点访问`channel`端口`nodes=...`

    然后将节点访问证书`ca.crt`、`node.crt`、`node.key`放到目录下`./weevent-broker/src/main/resources/v2/`。

  - Fabric 1.4

    具体内容详见[适配Fabric](https://weeventdoc.readthedocs.io/zh_CN/latest/advanced/fabric.html)。
    

  注意：区块链配置在两个代码模块`weevent-core`和`weevent-broker`里都有涉及到。都需要配置。

- 部署系统合约

  `weevent-core`和`weevent-broker`模块关于区块链的配置相同。

  通过运行`./weevent-core/src/main/java/com/webank/weevent/core/fisco/util/Web3sdkUtils.java`来部署`WeEvent`内置合约。

- 运行服务及代码样例

  启动`Broker`服务`./weevent-broker/src/main/java/com/webank/weevent/broker/BrokerApplication.java`。

  然后体验各种功能样例`./weevent-broker/src/test/java/com/webank/weevent/broker/sample`。

- 编译打包服务

  ```bash
  $ cd ./weevent-build; ./package.sh
  Usage:
      package master: ./package.sh --version 1.1.0
      package tag: ./package.sh --tag v1.1.0 --version 1.1.0
      package local: ./package.sh --tag local --version 1.1.0
  ```

  支持编译`master`最新代码，某个`tag`代码，以及本地的代码。

  编译环境依赖`git`，`git bash`， `gradle 4.10`，`java 1.8`，`nodejs 10.16`。

- 安装包一键安装服务

  详情参见[WeEvent快速安装](../install/quickinstall.html)。


欢迎参与`WeEvent`项目[issues](https://github.com/WeBankFinTech/WeEvent/issues)。

