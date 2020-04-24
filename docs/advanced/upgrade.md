## 服务升级

### Solidity合约升级

`WeEvent`从`1.1.0`版本开始，支持区块链数据的平滑升级，包括`Solidity`合约和`CRUD`表等。升级后的服务支持向后兼容，可以透明的访问历史数据。

每个版本中自带的合约部署脚本`./broker/deploy-topic-control.sh`同时也是区块链数据升级脚本。并且该脚本允许重复执行。

```shell
$ cd ./broker; ./deploy-topic-control.sh
2019-11-08 22:13:07 topic control address in every group:
topic control address in group: 1
	version: 10	address: 0x23df89a2893120f686a4aa03b41acf6836d11e5d	new: true
```

`new`字段`true`表示合约是本次部署的，`false`表示合约已经存在。

### Java程序升级

正常情况下，`Java`程序升级替换对应`Jar`包即可。

遇到数据库结构需要升级时，会在有需要的版本里携带升级脚本。`WeEvent`会始终坚持升级对用户透明的策略。