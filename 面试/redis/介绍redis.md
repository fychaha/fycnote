# redis

### 介绍一下redis

redis是一个开源，基于c语言编写的K-V型的非关系型数据库，s数据即可存储在内存，也可以持久化存储在硬盘。

##### redis支持哪些数据类型？

- String类型：set key value

  string类型是二进制安全的，所以redis的string可以包含任何数据。一个键最大能存储512MB

- Hash(哈希)：hmset key field1 value1 [field2 value2 ],将多个field-value设置入key中 

  redis hash 是一个键值对集合。

  redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

- List（列表）：lpush key value ，在key对应的list头部（左）添加字符串元素。

  rpush key value： 在key对应的尾部（右）添加字符串元素。

  lrem key count value：count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 *count* 。

  count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 *count* 的绝对值。

  count = 0 : 移除表中所有与 VALUE 相等的值。

  llen key：返回对应list的长度

- Set类型：saad key value1[value2 ....]，将一个或多个元素加入某集合。

- zset类型：有序集合，通过score大小来排序。

  zadd key score value ，将一个或多个元素加入某集合

##### 什么是redis持久化？

持久化就是把内存中的数据写到磁盘里去，防止服务器宕机导致内存丢失数据。

redis提供来了两种持久化方式：RDB（默认）和AOF

RDB（redis DataBase）：在指定的时间间隔内将内存中的数据集快照写入磁盘。实际操作是fork一个子进程（因为redis本身是单线程的），现将数据集写入临时文件，写入成功后，再替换磁盘中之前的文件，用二进制压缩存储。

AOF（Append-Only File）：AOF持久化日志的形式记录服务器所处理过的每一个写，删操作（查询没必要），以文本方式记录。

两者的比较：

- aof文件比rbd更新频率高（也就相当于能保证数据丢失的概率更小），优先使用aof还原数据。
- aof比rdb更安全也更大
- rdb性能比aof好。
- 如果两个都配置的话优先加载AOF。（因为数据时最重要的）

### redis有哪些架构模式？

- 单机版

  特点：简单，内存容量有限，处理能力有限，不具备高可用。

- 主从复制

  redis的复制功能允许我们根据一个主redis服务器（master）来创建任意多个该服务器的复制品。而通过复制出来的服务器称为从服务器（slave）。只要主从服务器之间网络连接正常，它们就会有相同的数据。主服务器会一直将自己的更新数据同步给从服务器。

- 哨兵

  redis sentinel是分布式系统中监控主从服务器，并在主服务器下线时自动进行故障转移。有三个特性：

  1. 监控：sentinel会不断地检查你的主服务器和从服务器是否运作正常。
  2. 提醒：当被监控的某个redis服务器出现问题时，sentinel会发送通知。
  3. 自动故障迁移：当一个主服务器故障时，sentinel会开始投票选举从服务器来担任主服务器的功能。

  缺点：

  - 切换期间可能会丢失这段时间的数据
  - 实际上仍是单个主服务器，写的压力依然存在

- 集群（proxy型）：（拜占庭将军算法）其他不了解。

- 集群（直连型）：redis3.0后支持redis-cluster集群，每个节点保存数据和整个集群状态，每个节点都和其他所有节点连接。如果有一半以上节点去ping一个节点没有回应，那么认为这个节点宕机了，然后就去连接该节点的备用（从服务器）节点，要是连备用节点也全挂了的话，那么集群的slot映射也不完整了，此时整个集群进入fail状态不可用。再有，整个集群有超过一半以上master挂掉了，不管还又有没有slave，集群也进入fail状态，不可用。

  

  