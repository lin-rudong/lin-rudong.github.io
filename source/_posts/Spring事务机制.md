---
title: Spring事务机制
date: 2018-10-08 16:09:47
tags:
categories: Spring
---
# 事务机制
## 事务的传播行为
* 事务方法被另一个事务方法调用时，事务如何传播。

1. propagation_required
* 如果当前事务存在，加入当前事务。如果不存在事务，开启一个新的事务。
2. propagation_supports
* 如果当前事务存在，加入当前事务。如果不存在事务，非事务地执行。
3. propagation_mandatory
如果当前事务存在，加入当前事务。如果不存在事务，抛出异常。
4. propagation_requires_new
* 总是开启一个新的事务，并挂起任何存在的事务。内外层事务互相独立。
* 使用JtaTransactionManager作为事务管理器。
5. propagatrion_not_supported
* 总是非事务地执行，并挂起任何存在的事务。
* 使用JtaTransactionManager作为事务管理器。
6. propagation_never
* 总是非事务地执行。如果存在事务，抛出异常。
7. propagation_nested
* 类似propation_required。
* 内层事务依赖于外层事务。外层事务会回滚内层事务的动作。内层事务不会回滚外层事务。

## 隔离级别
* 事务受其他并发事务影响的程度。
1. isolation_default
* 使用后端数据库默认的隔离级别。
2. isolation_read_uncommitted
* 允许读取未提交的数据。最低的隔离级别。
* 会导致脏读、不可重复读和幻读。
3. isolation_read_committed
* 允许读取并发事务已经提交的数据。
* 可以防止脏读，会导致不可重复读和幻读。
4. isolation_repeatable_read
* 会导致幻读。
5. isolation_serializable
* 最高也是最慢的隔离级别。通过锁定事务相关的数据库表来实现。

### 并发事务引起的问题
1. 脏读
* 还没提交的修改被其他事务读取了。事务自身没有原子性。
2. 不可重复读
* 同一事务的多次相同查询得到不同的数据。事务之间没有原子性。
3. 幻读
* 同一事务的多次相同查询记录数不同。事务之间没有原子性。

* 不可重复读和幻读的区别
    1. 不可重复读重点是修改，幻读重点是增加和删除。
    2. 本质原因都是事务之间没有原子性。从控制的角度来看，修改只需锁住满足条件的记录，而增加和删除需要锁住满足条件及其相近的记录。

## 只读事务
* 只读事务可以让数据库进行一些特定的优化。

## 事务超时
* 在特定事件内事务没有执行完毕就会自动回滚。

## 回滚规则
* 默认情况下，事务遇到运行期异常才会回滚，而遇到受查型异常不会回滚。

```
@Transactiona(propagation=Propagation.REQUIRES_NEW,isolation=Isolation.READ_COMMITTED,noRollbackFor={UserAccountException.class},readOnly=true, timeout=3)
public void f(){
    ...
}
```
