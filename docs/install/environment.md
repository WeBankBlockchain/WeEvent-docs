## 系统要求

### 配置说明

| 配置     | 最低配置          | 推荐配置                                                     |
| -------- | ----------------- | ------------------------------------------------------------ |
| CPU      | 2核 1.5GHz        | 4核 2.4GHz                                                   |
| 内存     | 2G                | 4GB                                                          |
| 带宽     | 1M                | 5M                                                           |
| Java     | Java(TM) 1.8      | Java(TM) 1.8                                                 |
| 操作系统 | 能正常运行JVM即可 | 快速安装Bash脚本在以下环境测试通过：<br />CentOS7.2、Ubuntu16.04、RedHat7.4<br />Java服务在以下环境测试通过：<br />CentOS7.2、Ubuntu16.04、RedHat7.4、Windows7 |

注意：在`CentOS`系统中，如果使用 `Open JDK` 1.9以下版本，`WeEvent`启动时会出现以下异常。请升级`Open JDK`版本到1.9或者使用`Oracle JDK`。

```
javax.net.ssl.SSLException: Failed to initialize the client-side SSLContext: Input stream not contain valid certificates
```

### 第三方软件

| 软件名     | 推荐版本 | 是否可选 |
| ---------- | -------- | -------- |
| Nginx      | 1.14.2   | 必选     |
| FISCO-BCOS | 2.0.0    | 必选     |
| Mysql      | 5.6.x    | 可选     |
| Redis      | 5.x.x    | 可选     |
| Zookeeper  | 3.5.5    | 可选     |
| WeBase     | 1.0.x    | 可选     |
