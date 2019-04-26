## 区块链
`WeEvent`使用区块链`FISCO-BCOS`作为数据存储和持久化机制。这里列举的是`WeEvent`访问`FISCO-BCOS`节点可能出现的问题。关于`FISCO-BCOS`更多FAQ，参见[FISCO-BCOS FAQ](https://fisco-bcos-documentation.readthedocs.io/zh_CN/release-1.3/docs/community/FAQ.html) 。
- `FISCO-BCOS`节点推荐的组网方式？

  基于节点容灾备份的考虑，建议每个机构至少部署2个`FISCO-BCOS`节点，分布在不同的机器甚至网段上。例如：3个机构间通过一条链互联，则至少应该部署3 * 2 = 6个区块链节点。


- 怎么查看`FISCO-BCOS`节点状态？

通过`FISCO-BCOS`提供的命令行工具`monitor.sh`

```bash
$ ./monitor.sh 
{"id":67,"jsonrpc":"2.0","result":"0x299"}
{"id":68,"jsonrpc":"2.0","result":"0x1767"}
[2019-03-28 16:15:43]  OK! 0.0.0.0:8545 is working properly: height 0x299 view 0x1767
[2019-03-28 16:15:43]  # log parser min, 2019-03-28 16:14
{"id":67,"jsonrpc":"2.0","result":"0x299"}
{"id":68,"jsonrpc":"2.0","result":"0x1767"}
[2019-03-28 16:15:44]  OK! 0.0.0.0:8546 is working properly: height 0x299 view 0x1767
[2019-03-28 16:15:44]  # log parser min, 2019-03-28 16:14
{"id":67,"jsonrpc":"2.0","result":"0x299"}
{"id":68,"jsonrpc":"2.0","result":"0x1767"}
[2019-03-28 16:15:44]  OK! 0.0.0.0:8547 is working properly: height 0x299 view 0x1767
[2019-03-28 16:15:44]  # log parser min, 2019-03-28 16:14
{"id":67,"jsonrpc":"2.0","result":"0x299"}
{"id":68,"jsonrpc":"2.0","result":"0x1768"}
[2019-03-28 16:15:44]  OK! 0.0.0.0:8548 is working properly: height 0x299 view 0x1768
```

- 为什么会出现`activeConnections isEmpty`错误？

```bash
2019-01-03 14:18:27.507 [pool-6] ERROR ChannelConnections(ChannelConnections.java:161) - activeConnections isEmpty
2019-01-03 14:18:28.061 [pool-6] ERROR Service(Service.java:488) - system error
java.lang.Exception: activeConnections isEmpty
 at org.bcos.channel.handler.ChannelConnections.randomNetworkConnection(ChannelConnections.java:162) ~[web3sdk-v1.2.6.jar:?]
 at org.bcos.channel.client.Service.asyncSendEthereumMessage(Service.java:460) ~[web3sdk-v1.2.6.jar:?]
 at org.bcos.channel.client.Service.sendEthereumMessage(Service.java:290) ~[web3sdk-v1.2.6.jar:?]
 ...
```
`WeEvent`到`FISCO-BCOS`节点的连接有异常，一般是网络不通引起的。

- 节点证书异常

```bash
=====INIT ECDSA KEYPAIR From private key===
2019-03-25 10:27:17.624 ERROR [nioEventLoopGroup-2-1] [ChannelHandler.java:147 org.bcos.channel.handler.ChannelHandler.exceptionCaught] network error 
io.netty.handler.codec.DecoderException: javax.net.ssl.SSLHandshakeException: General SSLEngine problem
	...
	io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:884)
	at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
	at java.lang.Thread.run(Thread.java:745)
```
请检查并且更新证书。

