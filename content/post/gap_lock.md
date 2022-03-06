---
title: "记一次死锁现象问题的记录"
date: 2022-03-06
tags: ["MySQL", "lock"]
description : "该文章主要是介绍了 MySQL 中间隙锁"
---
该篇文章主要是从一次死锁现象到分析到原因的记录。
## 现象描述
当一个事务中，当插入了出现了 `Duplicate key Error` 时，但没有提交当前事务；另一个事务使用 `SELECT * FROM table FOR UPDATE` 语句时，会发生等待执行，等待时间后出现错误 `ERROR 1205 (HY000):Lock wait timeout exceeded;`。为什么会出现这种现象呢？


## 现象复现
背景条件: MySQL 5.7.31; 存储引擎：InnoDB; 隔离级别：可重复读。
连上数据库后，通过 `select verion();` 查看当前版本；可以通过命令 `select @@tx_isolation;` 查看当前会话的事务隔离级别。可以通过命令 `show variables like "auto%";` 查看属性 `autocommit` 是否为自动提交。

输入以下建表语句
```SQL
REATE TABLE `user_test` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `create_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `name` varchar(100) DEFAULT NULL,
  `id_card` char(18) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uidx_ic` (`id_card`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8mb4
```
主要注意的是这里有一个唯一键索引 `id_card` 字段。

打开会话 Session A。由于 MySQL 5.7 版本的默认隔离级别是可重复读，事务也是默认自动提交的。所以这里需要使用 `set session autocommit = 0;` ,将当前的事务处于非自动提交状态。输入 SQL 语句
```sql
insert into user_test(id,name, id_card) values(1, "ss", 222222222222222223);
```
这时不会出现错误，当第二次输入时：出现错误 `ERROR 1062 (23000):Duplicate entry ..` ，发生了重复键错误，这时候不要提交当前事务。而进入另一个会话 Session B， 输入 SQL `select * from user_test where id_card = '222222222222222223' for update;`，这时候就会发生等待了，等待一定时间后，发现报一样的错误。这时候复现了上述问题。

为了更好的追踪上述问题，再次在 Session B 中输入 `select * from user_test where id_card = '222222222222222223' for update;`, 这时候再打开其他的 Session， 输入 `show engine innodb status\G` 查看当前 innoDB 的状态，果然在其中发现了蛛丝马迹。
查看
 ```log
 RECORD LOCKS space id 43 page no 4 n bits 72 index uidx_ic of table `demo`.`user_test` trx id 3602 lock_mode X locks rec but not gap waiting
Record lock, heap no 4 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 18; hex 323232323232323232323232323232323233; asc 222222222222222223;;`
 ```
 发生了死锁。

## 查找原因
通过日志信息我们得出了：RECORD LOCKS，锁的模型是 X locks。X lock 搜索数据库官方文档，我们知道它是排他锁(Exclusive Lock)。通过文档 [Locks Set by Different SQL Statements in InnoDB](https://dev.mysql.com/doc/refman/5.7/en/innodb-locks-set.html) 我们可以知道：
在使用 `SELECT ... FOR UPDATE` 命令时，我们会在每条搜索记录上使用一个排他临键锁。然而只有当我们搜索记录只包含有唯一索引的唯一记录时会使用索引记录锁。和这条命令相似的有 `UPDATE ... WHERE ...`, `DELETE FROM ... WHERE ...`。
在使用 `INSERT` 语句时，在插入行会有一个排他锁。这时候是一个页面记录锁，不是临键锁，并不会阻止其他的 Session 在这行记录之前的间隙中插入。在插入之前会设置一个插入间隙意向锁(临键锁的一种)，这个锁标识着不同事物插入不同的间隙位置是不需要等待的。当发生重复键发生时，即拥有相同的间隙位置时，在重复索引记录上会设置一个共享锁。这个共享锁会导致死锁。原因是因为这个啊。

查看 [innodb locking](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html) 后知道会发生死锁的四个因素：

| | 排它锁 | 意向排它锁 | 共享锁 | 意向共享锁|
|:--- |:---|:---|:---|:---|
| 排它锁 | 冲突 |冲突 |冲突 | 冲突 |
|意向排他锁 |冲突 |兼容 | 冲突 | 兼容 |
| 共享锁 | 冲突 | 冲突 | 兼容 | 兼容 |
| 意向共享锁 | 冲突 | 兼容 | 兼容 | 兼容 |

通过整个表，可以我们可以知道，由于 duplicate key 错误会产生一个共享锁, 这时候由于事务还未提交，所以 Session A 一直持有共享锁，而当 Session B 使用 SQL 时持有了排它锁，导致了锁冲突，发生了死锁。
至此我们分析完了这次的原因。

## 优化方案
1. 使用 `INSERT INGORE` , `INSERT..ON DUPLICATE KEY UPDATE` 等不会产生锁的 SQL
2. 避免使用长事务
3. 可以在某些特定的业务场景使用， `READ-COMMIT` 的事务隔离级别。

> 参考文章：https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html