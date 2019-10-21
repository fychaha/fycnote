## 安装zookeeper

下载镜像站点

```http
https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/
```

## 安装步骤

前提：

安装jdk和设置JAVA_HOME

第一步：解压缩压缩包 

具体命令：tar  xvzf  XXXX.tar.gz

第二步：进入zookeepe目录，修改配置文件把zoo_sample.cfg改名为zoo.cfg 

具体命令 cp zoo_sample.cfg zoo.cfg

第三步：修改数据文件位置dataDir

## 启动

启动服务：

`````bash
windwos  bin/zkServer.cmd
Linux： bin/zkServer.sh start
`````

停止： bin/zkServer.sh stop

备注：查看zookeeper进程 

Jps

ps -ef | grep QuorumPeerMain

启动客户端 查看 bin/zkCli.cmd

使用图形客户端