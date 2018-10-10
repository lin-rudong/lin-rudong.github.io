---
title: 高性能MySQL
categories: MySQL
abbrlink: 58527
date: 2018-10-08 16:08:42
tags:
---
# MySQL架构
## MySQL逻辑架构
1. 连接/线程处理
2. 查询缓存 解析器 优化器
3. 存储引擎（InnoDB会解析外键定义）

## 并发控制
1. 读写锁，读锁是共享锁，写锁是排他锁。
2. 锁粒度
    1. 表锁
    2. 行级锁
    * 最大程度地支持并发处理，同时也带来了最大的锁开销。

## 事务
* 一组原子性地SQL查询
```
start transaction;
...
commit;
```
* 事务处理系统具备ACID，A原子性，C一致性，I隔离性，D持久性。
    1. 原子性
    * 要么全部提交，要么全部回滚。
    2. 一致性
    * 从一个一致性状态转换到另一个一致性状态。
    3. 隔离性
    * 一个事务的修改在最终提交以前，对其他事务是不可见的。
    4. 持久性
    * 一旦事务提交，所做的修改就会永久保存到数据库中。

### 隔离级别
```
set transaction isolation level read committed;
set session transaction isolation level read committed;
```
1. read uncommitted 未提交读
* 事务中的修改，即使没有提交，对其他事务也都是可见的。
* 脏读：事务可以读取未提交的数据。
* 性能比其他级别不会好太多，却缺乏很多好处，很少使用。

2. read committed 提交读
* 事务只能看见已经提交的事务的修改。
* 大多数数据库的默认隔离级别，MySQL不是。
* 未提交事务就可以读取数据，导致一个事务内2次读取数据可能是不一样的，即保证了可见性却没有原子性。

3. repeatable read 可重复读
* 保证同一个事务中多次读取同样的记录的结果是一致的。
* MySQL默认隔离级别。
* 不能解决幻读。

4. serializable 可串行化
* 最高的隔离级别，强制事务串行执行。
* 每一行数据都加锁，导致大量的超时和锁争用问题，很少使用。

### 死锁
* InnoDB处理死锁的方法，将持有最少行级排他锁的事务进行回滚。

### 事务日志
* 顺序I/O，速度快。

### MySQL中的事务
* 事务型存储引擎：InnoDB和NDB Cluster。
1. 自动提交
* MySQL默认模式，autocommit=1。
```
set autocommit=0;
...
commit;
```