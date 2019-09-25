# Redis安装 [^centos7]



### 1.下载安装包

```linux
wget http://download.redis.io/releases/redis-2.6.17.tar.gz
```

如果没有wget，先下载wget

```
yum install wget
```

### 2.解压安装包,并安装

解压

``` 
tar -zxvf redis-4.0.9.tar.gz
cd redis-4.0.9
```

安装gcc

```
yum install gcc
```

编译

```
make MALLOC=libc
make install
```

### 3.配置Redis

打开redis.conf,修改以下代码

```
daemonize yes
//允许后台启动
bind xxx.xx.xx.x
//允许连接该redis的地址
requirepass xxxx
//配置登录密码
protected-mode no
//关闭保护模式
```

### 4.启动与关闭Redis

```
redis-server redis.conf
redis-cli -h 127.0.0.1 (-a yourpassword) 
//开启
>ping
pong
//代表开始成功
######################
使用shutdown，然后exit退出
```

### 至此安装成功







 