## 多群组

### FISCO-BCOS

`FISCO-BCOS`从2.0版本开始支持群组特性，解决用户隐私问题。1.3版本不支持群组功能。

系统合约[TopicController.sol]([TopicController.sol](https://github.com/WeBankFinTech/WeEvent/blob/master/weevent-broker/contract/TopicController.sol))是`WeEvent`访问区块链数据的入口合约。每个群组有各自独立的系统合约。

- 1.3版本部署合约

  - 部署系统合约

    ```bash
    $ deploy-topic-control.sh
    deploy contract[TopicController] success, address: 0x23df89a2893120f686a4aa03b41acf6836d11e5d
    ```

  - 修改配置

    得到合约地址`0x23df89a2893120f686a4aa03b41acf6836d11e5d`后，修改`broker/conf/fisco.properties`中的配置如

    ```ini
    topic-controller.address=0x23df89a2893120f686a4aa03b41acf6836d11e5d
    ```

    重启`broker`进程使配置生效。

- 2.0版本部署合约

  - 默认群组

    部署`FISCO-BCOS`区块链网络的时候，默认第一个群组的`groupId`为1。

    `WeEvent`快速安装时，一键安装脚本会自动调用脚本`deploy-topic-control.sh`在群组1上部署系统合约。然后自动修改配置文件`broker/conf/fisco.properties`中的配置项如：

    ```ini
    topic-controller.address=1:0x2811b5572d9160281787730ab1298f00a06f33b7
    ```

  - 自定义群组

    一般业务新建自定义群组的时候，都已经有默认群组1。新建群组的操作请参见`FISCO-BCOS`用户手册。

    - 部署合约

      假设用户新建的群组`groupId`为2，在新群组上部署系统合约。

      ```bash
      $ deploy-topic-control.sh 2
      deploy contract[TopicController] success, address: 0x23df89a2893120f686a4aa03b41acf6836d11e5d
      ```

    - 修改配置

      得到合约地址`0x23df89a2893120f686a4aa03b41acf6836d11e5d`后，修改`broker/conf/fisco.properties`中的配置项如：

      ```ini
      topic-controller.address=1:0x2811b5572d9160281787730ab1298f00a06f33b7;2:0x23df89a2893120f686a4aa03b41acf6836d11e5d
      ```

      配置值的格式为：`${groupId1}:${address1};${groupId2}:${address2};`

    - 重启服务

      重启`broker`服务，即可支持在群组2上进行事件的发布和订阅等。


