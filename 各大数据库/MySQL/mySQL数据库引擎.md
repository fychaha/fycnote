# MySQL数据引擎

mysq使用的存储引擎 myisam / innodb/ memory

myisam 存储: 如果表对事务要求不高，同时是以查询和添加为主的，我们考虑使用myisam存储引擎. ,比如 bbs 中的 发帖表，回复表.

INNODB 存储: 对事务要求高，保存的数据都是重要数据，我们建议使用INNODB,比如订单表，账号表.

MyISAM 和 INNODB的区别

\1. 事务安全（MyISAM不支持事务，INNODB支持事务）

\2. 查询和添加速度（MyISAM批量插入速度快）

\3. 支持全文索引（MyISAM支持全文索引，INNODB不支持全文索引）

\4. 锁机制（MyISAM时表锁，innodb是行锁）

\5. 外键 MyISAM 不支持外键， INNODB支持外键. (在PHP开发中，通常不设置外键，通常是在程序中保证数据的一致)

Memory 存储，比如我们数据变化频繁，不需要入库，同时又频繁的查询和修改，我们考虑使用memory, 速度极快. （如果mysql重启的话，数据就不存在了），与redis很相似。

# Myisam注意事项

如果你的数据库的存储引擎是myisam,请一定记住要定时进行碎片整理,myisam引擎在数据删除时，并不会直接删除数据，会在数据库内保留碎片残留，为了能恢复被删除的数据。

举例说明: 

create table test100(id int unsigned ,name varchar(32))engine=myisam;

insert into test100 values(1,’aaaaa’);

insert into test100 values(2,’bbbb’);

insert into test100 values(3,’ccccc’);

我们应该定义对myisam进行整理

optimize table test100;