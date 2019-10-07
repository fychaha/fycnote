# MySQL压缩版的安装

### step 1: 下载、解压 mysql

略

### step 2: 添加缺省的配置文件

mysql 解压的文件中默认没有两个关键性的东西：`my.ini` 文件和 `data` 文件夹。

在 mysql 的解压目录下的根目录下创建 `my.ini` 文件，并添置一下内容： 

```
# CLIENT SECTION
# ----------------------------------------------------------------------
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306

[mysql]
# 设置mysql客户端默认使用的字符集
default-character-set=utf8

# SERVER SECTION
# ----------------------------------------------------------------------
[mysqld]
# mysql服务端监听/占用的端口
port=3306

# mysql的安装路径
basedir=D:\Program Files\mysql-5.7.24-winx64

# mysql 数据库数据文件所在路径
datadir=D:\Program Files\mysql-5.7.24-winx64\data

# 新建schema或table时默认使用的字符集
character_set_server=utf8

#  新建Table时默认使用的引擎
default-storage-engine=INNODB
```

> 注意，此处的 basedir 和 datadir 要符合实际情况中 MySQL 的解压路径。

### step 3：初始化

> 由于后续要使用到 MySQL 解压目录下的 bin 目录下的各种命令，方便起见，可以新建 MYSQL_HOME 环境变量，并将其下的 bin 目录添加到环境变量 path 中。

在终端中输入，`mysqld --initialize`，回车后可见在 mysql 解压目录中生成了 data 文件夹。

`注意`，在 data 文件夹中，有一个 `.err` 文件，其中的内容包含了（最后一行）初始的 root 用户的密码，类似如下：

> 2019-01-09T08:11:05.817097Z 1 [Note] A temporary password is generated for root@localhost: ?ds:i+)4C/wo

### step 4: 安装/注册

在终端中输入：`mysqld –-install`，安装 mysql 服务。

另外，反向的卸载/删除操作中，使用命令 `mysqld --remove` 。

### step 5：启动

可以通过以下命令启动/停止 mysql 服务端。

```
net start mysql
net stop mysql
```

### step 6: 修改密码

启动 mysql，通过 `mysql -uroot -p` 进行连接/登录（使用默认的初始密码），然后可通过以下命令进行修改 root 用户密码：

```sql
set password for root@localhost=password('新密码');
```

最后输入：`show variables like "character%";` 确保字符集编码是 UTF8 编码。

### step 7:  确认 mysql 编码

mysql 的默认编码是 latin1，可通过下面命令查看确认：

```
show variables like 'character%';
```

如果显示结果中出现 latin1，则通过修改配置文件 my.ini 进行配置：

```
[mysqld]
# 新建schema或table时默认使用的字符集
character_set_server=utf8
```

## 卸载

```
net stop mysql
mysqld --remove
```

删除解压后的文件夹。

删除环境变量。