
## Processor模块

本节介绍`Processor`子模块的详细安装步骤。 `WeEvent`服务的快速安装请参见[快速安装](../quickinstall.html) 。在一台机器上详细安装，和通过快速安装然后把目标路径中的`Processor`子目录打包拷贝到这台机器，效果是一样的。

`Processor`为用户提供时序流分析和时间联动等。如果是第一次安装`WeEvent`，参见这里的[系统要求](../environment.html) 。以下安装以`CentOS 7.2`为例。

### 前置条件


- Broker模块

   必选配置，通过`Broker`访问区块链。

   具体安装步骤，请参见[Broker模块安装](./broker.html)。
- Governance模块

   必选配置，通过`Governance`从`Web`端调用Processor。修改配置文件`./governance/conf/application-prod.properties` ，增加processor对应的ip和端口配置` weevent.processor.url=http://127.0.0.1:7008`。

   具体安装步骤，请参见[Governance模块安装](./overnance.html)。   


- Mysql数据库

  必选配置。`Processor`通过`Mysql`存储数据。

  推荐安装`Mysql 5.6+`版本。具体安装步骤，安装请参见[Mysql安装](http://dev.mysql.com/downloads/mysql/) 。

### 获取安装包

从`github`下载安装包[weevent-processor-1.1.0.tar.gz](https://github.com/WeBankFinTech/WeEvent/releases/download/v1.1.0/weevent-processor-1.1.0.tar.gz)，并且解压到`/usr/local/weevent/`下。

```shell
$ cd /usr/local/weevent/
$ wget https://github.com/WeBankFinTech/WeEvent/releases/download/v1.1.0/weevent-processor-1.1.0.tar.gz
$ tar -xvf weevent-processor-1.1.0.tar.gz
```

如果`github`下载速度慢，可以尝试[国内下载链接](https://www.fisco.com.cn/cdn/weevent/download/releases/v1.1.0/weevent-processor-1.1.0.tar.gz)。

解压后的目录结构如下

```
$ cd ./weevent-processor-1.1.0
$ tree -L 2

```

```
$ cd ./weevent-processor-1.1.0
$ tree -L 2
|-- apps
|   `-- weevent-processor-1.1.0.jar
|-- check-service.sh
|-- conf
|   |-- application-prod.properties
|   |-- application.properties
|   |-- log4j2.xml
|   |-- mappers
|   |-- processor.properties
|-- cep_rule.sql
|-- init-processor.sh
|-- processor.sh
|-- lib
```

### 修改配置文件
- 配置端口

  在配置文件`./conf/application-prod.properties`中，`Processor` 的服务端口`server.port` ，默认`7008`。

   ```
   server.port=7008
   ```

- 配置Mysql数据库
   修改`datasource`中的`url`配置、`username`、`password` 

   ``` 配置数据库连接
   server.port=7008
   spring.datasource.url=jdbc:mysql://127.0.0.1:3306/processor
   spring.datasource.username=root
   spring.datasource.password=123456
   spring.datasource.driver-class-name=com.mysql.jdbc.Driver
   ```

- 在配置文件processor.properties配置Mysql数据库,修改`datasource`中的`url`配置、`username`、`password` 
   ```
   #============================================================================
   # config name and expression
   #============================================================================
   quartz.schedule.name=schedule
   #============================================================================
   # Configure Main Scheduler Properties
   #============================================================================
   org.quartz.scheduler.instanceName=test
   #============================================================================
   # Configure Datasources
   #============================================================================
   org.quartz.dataSource.WeEvent_processor.URL=jdbc:mysql://127.0.0.1:3306/WeEvent_processor
   org.quartz.dataSource.WeEvent_processor.user=xxxx
   org.quartz.dataSource.WeEvent_processor.password=yyyy
   org.quartz.dataSource.WeEvent_processor.maxConnections=30
   org.quartz.dataSource.WeEvent_processor.driver=com.mysql.jdbc.Driver
   #============================================================================
   # Configure JobStore
   #============================================================================
   org.quartz.jobStore.dataSource=WeEvent_processor
   org.quartz.dataSource.weevent_processor.URL=jdbc:mysql://127.0.0.1:3306/WeEvent_processor
   org.quartz.dataSource.weevent_processor.user=root
   org.quartz.dataSource.weevent_processor.password=123456
   org.quartz.dataSource.weevent_processor.maxConnections=30
   #============================================================================
   # Configure ThreadPool Quartz
   #============================================================================
   org.quartz.threadPool.threadCount=20
   org.quartz.threadPool.threadPriority=5
   ```

   - `org.quartz.scheduler.instanceName` 当前Schedule name，用户可以修改
   - `org.quartz.dataSource.weevent_processor.*`  数据库信息的配置
   - `org.quartz.jobStore.dataSource` 配置数据库名称

​    **注意**：数据库要赋予配置账号创建库表的权限。

   ```shell
   mysql
   >> grant all privileges on . to 'test'@'%' identified by '123456';
   >> flush privileges;
   ```

​   初始化系统，执行脚本`init-processor.sh` ，成功输出如下。否则，用户需要检查配置项是否正常。
   
    ```shell
    $ ./init-processor.sh
    init processor db success

    ```

### 服务启停

- 服务启动

  通过`./processor.sh start`命令启动服务，正常启动如下：

```shell
$ ./processor.sh start
  start weevent-processor success (PID=53927)
  add the crontab job success
```

  通过`./processor.sh stop`命令停止服务。

  进程启动后，会自动添加`crontab`监控任务`./processor.sh monitor`。

- 验证服务

   通过`./check-service.sh` 命令检查服务功能是否正常。

   ```shell
      $ ./check-service.sh
      check processor service
      processor service is ok
   ```
### 界面展示
![processor-set1.png](../../image/set1.png)

![processor-set2.png](../../image/set2.png)

![processor-show.png](../../image/show.png)

![processor-list.png](../../image/list.png)



### 命中逻辑说明

- 不支持嵌套查询、连表查询、自带函数查询、ORDER BY（ASC|DESC）


- 文本字段 vs. 数值字段

```
SELECT * FROM Websites WHERE country='CN';
```

- 支持的类型 运算符

| =    | 等于                              |
| ---- | ------------------------------- |
| <>   | 不等于。注释：在 SQL 的一些版本中，该操作符可被写成 != |
| >    | 大于                              |
| <    | 小于                              |
| >=   | 大于等于                            |
| <=   | 小于等于                            |


数字类型

```
temperature=29;
temperature>29;
temperature>=29;
temperature<29;
temperature<=29;
temperature<>29;
```

文本类型

```
SELECT * FROM Websites WHERE facilicty-charater="warning";
```



- AND & OR 运算符

如果第一个条件和第二个条件都成立，则 and 运算符显示一条记录。

如果第一个条件和第二个条件中只要有一个成立，则 or 运算符显示一条记录。

```
SELECT * FROM Websites WHERE facilicty-charater="warning" and temperature > 50;

SELECT * FROM Websites WHERE temperature > 35 or  facilicty-charater="warning" ;
```