# 配置集群



修改zoo.cfg

````properties
tickTime=2000
# 不同的服务设置不同的路径
dataDir=/var/lib/zookeeper/server1
#dataDir=/var/lib/zookeeper/server2
#dataDir=/var/lib/zookeeper/server3


clientPort=2181
initLimit=5
syncLimit=2
server.1=zoo1:2888:3888
server.2=zoo2:2888:3888
server.3=zoo3:2888:3888
````

#### 三个dataDir 分别创建myid文件，里面只保存server的id信息

#### 然后启动各个服务

bin/zkServer.sh start conf/zoo.cfg

bin/zkServer.sh start conf/zoo2.cfg

bin/zkServer.sh start conf/zoo3.cfg

#### 查看集群状态

bin/zkServer.sh status conf/zoo.cfg

bin/zkServer.sh status conf/zoo2.cfg

bin/zkServer.sh status conf/zoo3.cfg