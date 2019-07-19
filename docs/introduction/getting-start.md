## Getting Start

如果是使用`WeEvent`服务，可以选择`Docker`镜像或者通过`Bash`脚本一键安装。详情参见[WeEvent快速安装]()。

如果是参与`WeEvent`开发或者想要体验未正式发布的特性，流程如下：

- 下载[github源码](https://github.com/WeBankFinTech/WeEvent)

  ```bash
  $ git clone https://github.com/WeBankFinTech/WeEvent.git
  ```

- 运行代码样例

  通过IDE（推荐IDEA）打开项目工程，目录`./weevent-broker/src/test/java/com/webank/weevent/sample`下的样例可以直接运行。
  
- 在weevent-build子模块下编译打包

  ```bash
  $ ./package.sh
  Usage:
      package master: ./package.sh --version 1.0.0
      package tag: ./package.sh --tag v1.0.0 --version 1.0.0
      package local: ./package.sh --tag local --version 1.0.0
  ```

  开发环境包括`git`，`git bash`， `gradle` 4.10，`java` 1.8，`nodejs` 10.16。

- Bash脚本一键安装

  详情参见[WeEvent快速安装]()。


欢迎参与`WeEvent`项目[issues](https://github.com/WeBankFinTech/WeEvent/issues)。

