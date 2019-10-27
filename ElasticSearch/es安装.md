# **2 安装**

## **2.1** **W**indow下安装步骤

因为是基于java开发的，那么安装elastic search之前jdk一定是要装好的。

 

接下来直接从官网上根据操作系统以及版本下载所需要的安装包。

 

下载地址：https://www.elastic.co/cn/downloads

 

 

下载并解压以后就可以使用了，没有听错，就是可以直接使用了，没有听错，开箱即用就是elasticsearch的特点之一。

 

在解压的目录下找到elasticsearch.bat文件双击即可，在浏览器中输入URL地址

 

http://locathost:9200

 

如果出现以一个json格式的数据表示安装成功了。

![img](file:///C:\Users\admi\AppData\Local\Temp\ksohtml4776\wps1.png)![img](file:///C:\Users\admi\AppData\Local\Temp\ksohtml4776\wps2.jpg)

## **2.2 docker 安装**

参考：<https://www.elastic.co/guide/en/elasticsearch/reference/7.4/docker.html>

下载镜像：docker pull docker.elastic.co/elasticsearch/elasticsearch:7.4.1

启动容器运行：docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.4.1

 

## **2.3 centos7安装**

### **2.3****.1** **创建用户并设置密码**

注意：root用户不能启动ES，所以需要创建新用户

使用root登录

创建用户esuser 命令如下：

adduser esuser

设置密码 命令如下：

passwd esuser

我的密码：Guo!123456

### **2.3.2 下载文件**

wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.4.0-linux-x86_64.tar.gz

 

或者使用esuser 用户登录xftp，上传文件

### **2.3.3 使用xshell登录虚拟机，并切换到esuser用户 ，命令如下**

su esuser

### **2.3.4 解压文件，命令如下：**

tar xvzf elasticsearch-7.4.0-linux-x86_64.tar.gz

 

//1.6 授权chown -R esuser elasticsearch-7.3.1

### **2.3.5 启动elasticsearch  进入bin目录，命令如下**

 ./elasticsearch

 

### **2.3.6 查看是否成功访问**

[http://localhost:9200/](http://localhost:9200/_cat/nodes?v)

查看当前节点信息

<http://localhost:9200/_cat/nodes?v>

# **3** **centos7****启动错误解决**

问题出现环境，OS版本：CentOS-7-x86_64-Minimal-1708；ES版本：elasticsearch-6.2.2。

 

## **3****.1****、max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]**

 

每个进程最大同时打开文件数太小，可通过下面2个命令查看当前数量

修改/etc/security/limits.conf文件，增加配置，用户退出后重新登录生效

\*               soft    nofile          65536

\*               hard    nofile          65536

 

## **3****.2** **max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]**

 

　　**修改****/etc/sysctl.conf文件，增加配置vm.max_map_count=262144**

 

**vi****m** **/etc/sysctl.conf**

**sysctl -p**

**执行命令****sysctl -p生效**

## **3****.3** **启动报错信息如下：**

 

**解决：**

**vim /etc/elasticsearch/elasticsearch.yml**

 

**# 修改【#cluster.initial_master_nodes: ["node-1", "node-2"] 】**

**cluster.initial_master_nodes: ["node-1"]**

 

 

## **3.5 正常后访问**

 curl "http://127.0.0.1:9200" 能够正常访问，可是使用外网ip就提示拒绝链接

解决办法：vim config/elasticsearch.yml

 

增加：network.host: 0.0.0.0

 

# **4 Elasticsearch  目录结构如下：**

Ø bin ：脚本文件，包括 ES 启动 & 安装插件等等

Ø config ： elasticsearch.yml（ES 配置文件）、jvm.options（JVM 配置文件）、日志配置文件等等

Ø JDK ： 内置的 JDK，JAVA_VERSION="12.0.1"

Ø lib ： 类库

Ø logs ： 日志文件

Ø modules ： ES 所有模块，包括 X-pack 等

Ø plugins ： ES 已经安装的插件。默认没有插件

Ø data ： ES 启动的时候，会有该目录，用来存储文档数据。该目录可以设置

 

# **5 单机集群多个 ES 实例安装**

单机多个 ES 实例，形成一个 ES 单机伪集群，启动脚本如下：

bin/elasticsearch -E node.name=node01 -E cluster.name=bysocket_es_cluster -E path.data=node01_data -dbin/elasticsearch -E node.name=node02 -E cluster.name=bysocket_es_cluster -E path.data=node02_data -dbin/elasticsearch -E node.name=node03 -E cluster.name=bysocket_es_cluster -E path.data=node03_data -dbin/elasticsearch -E node.name=node04 -E cluster.name=bysocket_es_cluster -E path.data=node04_data -d 

 

命令简单解释如下：

node.name ： ES 节点名称，即实例名cluster.name ： ES 集群名称path.data ： 指定了存储文档数据目录

 

执行完脚本后，需要等一会 ES 启动，也可以查看 logs 看看执行情况。

 

如何关闭集群中的 ES 实例，可以使用简单的命令实现：

ps | grep elasticsearch kill -9 pid

 

# **6 插件概述**

## **6.1 简介**

插件是用来增强 Elasticsearch 功能的方法，分为 核心插件（官方） & 社区插件。

## **6.2 安装及删除**

安装 analysis-icu ICU 分析插件，命令如下：

sudo bin/elasticsearch-plugin install analysis-icu本地安装示例如下：bin/elasticsearch-plugin install file:///root/software/elasticsearch/analysis-smartcn-7.4.0.zip

 

查看已安装的插件，命令如下：

bin/elasticsearch-plugin list  或者curl <http://localhost:9200/_cat/plugins>

 

删除已安装的插件，命令如下：

sudo bin/elasticsearch-plugin remove analysis-icu

 

## **6.3**  **analysis-icu** **插件介绍**

<https://blog.csdn.net/weixin_42257250/article/details/97757295>

## **6.4 分词器简介：**

<https://blog.csdn.net/moshowgame/article/details/99448661>

## **6.5 简体中文插件**

Smart Chinese Analyzer Plugins

中文分词器，听说Elastic Stack 8.0会自带，但是还没release，静候佳音吧。

 

Smart Chinese Analysis插件将Lucene的Smart Chinese分析模块集成到elasticsearch中。

 

提供中文或混合中英文本的分析器。 该分析器使用概率知识来查找简体中文文本的最佳分词。 首先将文本分成句子，然后将每个句子分割成单词。

 

sudo bin/elasticsearch-plugin install analysis-smartcn

 

本地安装：

./elasticsearch-plugin install [file:///root/software/elasticsearch/analysis-smartcn-7.4.0.zip](file://root\software\elasticsearch\analysis-smartcn-7.4.0.zip)

 

查看分词情况：

 

 curl -X POST "localhost:9200/_analyze?pretty" -H 'Content-Type: application/json' -d'{  "tokenizer": "standard",  "text": "中国向日葵"}' 

 

 curl -X POST "localhost:9200/_analyze?pretty" -H 'Content-Type: application/json' -d'{  "tokenizer": "smartcn_tokenizer",  "text": "中国向日葵"}' 

 

curl -X POST "localhost:9200/_analyze?pretty" -H 'Content-Type: application/json' -d'{  "tokenizer": "icu_tokenizer",  "text": "中国向日葵"}' 

 

# **7 RestApi介绍**

参考：<https://www.elastic.co/guide/en/elasticsearch/reference/7.4/rest-apis.html>

 

8 java客户端

<https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/java-docs-index.html>

 

参考：<https://www.cnblogs.com/wangzhuxing/p/9609127.html>