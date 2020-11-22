

> [!NOTE]
> - MySQL 中 InnoDB 引擎支持 MVCC  
> - 应对高并发事务, MVCC 比单纯的加行锁更有效, 开销更小  
> - MVCC 在读已提交（Read Committed）和可重复读（Repeatable Read）隔离级别下起作用   
> - MVCC 既可以基于乐观锁又可以基于悲观锁来实现 

## MVCC 描述

> [!NOTE]
> MVCC，处理读写冲突的手段，目的在于提高数据库高并发场景的吞吐性能

MVCC (Multiversion Concurrency Control) 中文全程叫多版本并发控制，是现代数据库（包括 MySQL、Oracle、PostgreSQL 等）引擎实现中常用的**处理读写冲突的手段，**目的在于**提高数据库高并发场景下的吞吐性能。**



**多版本读例子**，下面两个事务 A 和 B 按照如下顺序进行更新和读取操作。

| Transaction A                  | Transaction B                                             |
| :----------------------------- | :-------------------------------------------------------- |
| select x from table; return 10 |                                                           |
| begin transaction              |                                                           |
| Update table set x = 20        |                                                           |
|                                | begin transaction                                         |
|                                | <font color="red">select x from table; return rs1 </font> |
| Commit                         |                                                           |
|                                | <font color="red">select x from table; return rs2</font>  |
|                                | commit                                                    |

在事务 A 提交后，事务 B 读取到 x 的值是什么呢？

事务 B 在不同的隔离级别下，读取的值不一样。

| 事务隔离级别            | 结果                                 |
| ----------------------- | ------------------------------------ |
| 读未提交（RU）          | 两次 x 都是最新值，即 20             |
| 读已提交（RC）          | 第一次读到旧值 10，第二次是最新值 20 |
| 可重复读或串行（RR，S） | 两次都是旧值 10                      |



## InnoDB MVCC 实现原理



参考链接：

[MySQL InnoDB MVCC 机制的原理及实现](https://zhuanlan.zhihu.com/p/64576887)