---
title: 事务隔离与MVCC
toc: true
date: 2021-03-26 09:51:14
tags:
categories:
---


https://blog.csdn.net/weixin_30342639/article/details/107552255

https://blog.csdn.net/qq_44961149/article/details/108420073

卡内基梅隆大学sql92标准txt地址: http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt

## 事务的特点（ACID）
- 原子性：对数据库的一系列的操作，要么都是成功，要么都是失败，不可能出现部分成功或者部分失败的情况；原子性，在 InnoDB 里面是通过 undo log 来实现的，它记录了数据修改之前的值（逻辑日志），一旦发生异常，就可以用 undo log 来实现回滚操作。
- 隔离性：在数据库里面会有很多的 事务同时去操作我们的同一张表或者同一行数据，必然会产生一些并发或者干扰的操作， 那么我们对隔离性的定义，就是这些很多个的事务，对表或者行的并发操作，应该是透明的，互相不干扰的。通过这种方式，我们最终也是保证业务数据的一致性。
- 持久性：我们对数据库的任意的操作，增删改，只要事务提交成功，那么结果就是永久性的，不可能因为我们系统宕机或者重启了数据库的服务器，它又恢复到原来的状态了；持久性是通过 redo log 和 double write 双写缓冲来实现的。
- 一致性：是数据库的完整性约束没有被破坏，事务执行的前后都是合法的数据状态。上面三个特性保证了一致性。

事务的实现原理：
- 事务的原子性是通过 undo log 来实现的
- 事务的持久性是通过 redo log 来实现的
- 事务的隔离性是通过（读写锁+MVCC）来实现的
- 事务的一致性是通过原子性、持久性、隔离性来实现的

隔离性产生的问题，sql92标准中搜索【4.28 SQL-transactions】
- P1 ("Dirty read"): SQL-transaction T1 modifies a row. SQL-transaction T2 then reads that row before T1 performs a COMMIT. If T1 then performs a ROLLBACK, T2 will have read a row that was never committed and that may thus be considered to have never existed.
- P2 ("Non-repeatable read"): SQL-transaction T1 reads a row. SQL-transaction T2 then modifies or deletes that row and performs a COMMIT. If T1 then attempts to reread the row, it may receive the modified value or discover that the row has been deleted.
- P3 ("Phantom"): SQL-transaction T1 reads the set of rows N that satisfy some <search condition>. SQL-transaction T2 then executes SQL-statements that generate one or more rows that satisfy the <search condition> used by SQL-transaction T1. If SQL-transaction T1 then repeats the initial read with the same <search condition>, it obtains a different collection of rows.
- 幻读译：事务T1通过where条件读了N行的集合，事务T2的sql语句生成了一行或多行数据满足事务T1的where条件，事务T1通过原来的where条件再次读取，会读到不同的数据集，像出现幻觉一样。


## 事务的隔离级别
- 读未提交（Read uncommitted）：一个事务还没提交时，它做的变更就能被别的事务看到。
    - RU 隔离级别：不加锁
- 读提交（Read committed）：一个事务提交之后，它做的变更才会被其他事务看到。
    - RC 隔离级别下，普通的 select 都是快照读，使用 MVCC 实现，解决了脏读问题。
    - 加锁的 select 都使用记录锁，因为没有 Gap Lock
    - 除了两种特殊情况——外键约束检查(foreign-key constraint checking)以及重复键检查(duplicate-key checking)时会使用间隙锁封锁区间；所以 RC 会出现幻读的问题。
- 可重复读（Repeatable read）:一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读隔离级别下，未提交变更对其他事务也是不可见的。
    - RR 隔离级别下，普通的 select 使用快照读(snapshot read)，底层使用 MVCC 来实现
    - 加锁的 select(select … in share mode / select … for update)以及更新操作update, delete 等语句使用当前读（current read）（当前读inndb可能会导致幻读问题），底层使用记录锁、或者间隙锁、临键锁。
- 串行化（Serializable ）:顾名思义是对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。所有的 select 语句都会被隐式的转化为 select … in share mode，会和update、delete 互斥

## MySQL锁
官网八锁文档地址：https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html
- Shared and Exclusive Locks：共享锁-排他锁
- Intention Locks：意向锁，可细分为意向共享锁和意向排他锁
- Record Locks：记录锁
- Gap Locks：间隙锁
- Next-Key Locks：临键锁
- Insert Intention Locks：插入意向锁
- AUTO-INC Locks：自增锁
- Predicate Locks for Spatial Indexes：空间索引预测锁

## 参考资料
> - [https://blog.csdn.net/qq_44961149/article/details/108420073](https://blog.csdn.net/qq_44961149/article/details/108420073)
> - []()
