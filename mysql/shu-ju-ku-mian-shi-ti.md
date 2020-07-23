# 数据库面试题

## MySQL的复制原理以及流程

1. 主：binlog线程——记录下所有改变了数据库数据的语句，放进master上的binlog中；
2. 从：io线程——在使用start slave 之后，负责从master上拉取 binlog 内容，放进 自己的relay log中；
3. 从：sql执行线程——执行relay log中的语句；

## MySQL中varchar与char的区别以及varchar\(50\)中的50代表的涵义

\(1\)、varchar与char的区别

* char是一种固定长度的类型，varchar则是一种可变长度的类型

\(2\)、varchar\(50\)中50的涵义

* 最多存放50个字符，varchar\(50\)和\(200\)存储hello所占空间一样，但后者在排序时会消耗更多内存，因为order by col采用fixed\_length计算col长度

## innodb的事务与日志的实现方式

有多少种日志:

* 错误日志：记录出错信息，也记录一些警告信息或者正确的信息。
* 查询日志：记录所有对数据库请求的信息，不论这些请求是否得到了正确的执行。
* 慢查询日志：设置一个阈值，将运行时间超过该值的所有SQL语句都记录到慢查询的日志文件中。
* 二进制日志：记录对数据库执行更改的所有操作。
* 中继日志：
* 事务日志：

事物的4种隔离级别

* 读未提交\(RU\)
* 读已提交\(RC\)
* 可重复读\(RR\)
* 串行

事务是如何通过日志来实现的

* 事务日志是通过redo和innodb的存储引擎日志缓冲（Innodb log buffer）来实现的，当开始一个事务的时候，会记录该事务的lsn\(log sequence number\)号; 当事务执行时，会往InnoDB存储引擎日志

  的日志缓存里面插入事务日志；当事务提交时，必须将存储引擎的日志缓冲写入磁盘（通过innodb\_flush\_log\_at\_trx\_commit来控制），也就是写数据前，需要先写日志。这种方式称为“预写日志方式”。

## MySQL中InnoDB引擎的行锁是通过加在什么上完成\(或称实现\)的

InnoDB是基于索引来完成行锁

select \* from tab\_with\_index where id = 1 for update; for update 可以根据条件来完成行锁锁定,并且 id 是有索引键的列, 如果 id 不是索引键那么InnoDB将完成表锁,,并发将无从谈起。

