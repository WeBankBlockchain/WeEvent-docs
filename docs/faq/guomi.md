## 配置`FISCO BCOS`国密

### 前置条件
- 安装国密版`FISCO BCOS` 
  
  具体安装步骤，请参见[部署国密版FISCO BCOS](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/guomi_crypto.html#fisco-bcos)

### 修改`WeEvent`配置文件
- 修改配置项

  ```shell
  $ cd /usr/local/weevent/broker/
  $ vi ./conf/fisco.properties
  ```
  修改`web3sdk.encrypt-type`配置项为：`web3sdk.encrypt-type=SM2_TYPE`。其他安装配置与[快速安装](../install/quickinstall.md)一致。