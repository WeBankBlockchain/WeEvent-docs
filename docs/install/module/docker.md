## Docker 模块
### 使用Docker安装WeEvent

操作步骤

- 进入临时目录,获取`DockerFile` ,默认临时目录为`/data/weevent-docker/`

  ```
  $ cd /data/weevent-docker/
  $ wget 
  ```

- 执行命令，具体如下所示

  ```
  $ docker build -t  weevent-fb1.3-0.9.3:v0.9.5 .
  $ docker run --privileged --cap-add SYS_ADMIN -e container=docker -it --name weevent-fb1.3-0.9.3 -p 80:8080  -d  --restart=always store/oracle/serverjre:8
  $ docker exec -it weevent-fb1.3-0.9.3 /bin/bash
  ```

- 启动服务
```
$ ./data/fisco-bcos-nodes/nodes/build/start.sh
$ ./usr/local/weevent/start-all.sh
```

- 验证服务

  ```
  $ cd /usr/local/weevent
  $ ./check-server.sh
  ```

  ​

  ​