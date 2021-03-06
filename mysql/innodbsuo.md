# InnoDB锁

## mysql 乐观锁、悲观锁

![mysql lock](https://s2.ax1x.com/2019/01/30/klek5D.png)

**乐观锁\(Optimistic Locking\):** 指数据对被外界修改保持乐观态度。乐观锁不是数据库自带的，需要自己来进行实现。一般采用version记录机制。

```sql
update task set value = new_value, version = 3 where version = 2;
```

同一时刻多个客户端进行更新的时候只有一个会更新成功。因为where条件只有一个能够符合。

### CAS

乐观锁具体实现细节：主要就是两个步骤：冲突检测和数据更新。还有一种比较典型的就是Compare and Swap\(`CAS`\)。

CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值\(B\)。如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该位置的值。（在 CAS 的一些特殊情况下将仅返回 CAS 是否成功，而不提取当前值。）CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。”这其实和乐观锁的冲突检查+数据更新的原理是一样的。

### ABA问题

CAS看起来很爽，但是会导致“ABA问题”。比如说一个线程one从内存位置V中取出A，这时候另一个线程two也从内存中取出A，并且two进行了一些操作变成了B，然后two又将V位置的数据变成A，这时候线程one进行CAS操作发现内存中仍然是A，然后one操作成功。尽管线程one的CAS操作成功，但是不代表这个过程就是没有问题的。如果链表的头在变化了两次后恢复了原值，但是不代表链表就没有变化。通过版本号可以解决ABA问题。

**悲观锁:** 指数据对被外界修改保持悲观态度。一般采用数据库的锁机制。

* 共享锁【S锁】: 又称读锁，若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。
* ps 加上共享锁之后自己都不能对数据进行修改了吗？

```sql
select * from account where name = "Max" lock in share mode
```

* 排他锁【X锁】: 又称写锁。若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁。这保证了其他事务在T释放A上的锁之前不能再读取和修改A。

```sql
// 会对符合where语句中条件的条目进行加锁。
select * from account where name = "Max" for update;
```

mysql InnoDB引擎默认的修改数据语句，update,delete,insert都会自动给涉及到的数据加上排他锁，select语句默认不会加任何锁类型。

如果加排他锁可以使用select ...for update语句，加共享锁可以使用select ... lock in share mode语句。

## mysql 行锁 表锁

在mysql的InnoDB引擎支持行锁，与Oracle不同，mysql的行锁是通过索引加载的，即是行锁是加在索引响应的行上的，要是对应的SQL语句没有走索引，则会全表扫描，行锁则无法实现，取而代之的是表锁。

* 表锁：不会出现死锁，发生锁冲突几率高，并发低。
* 行锁：会出现死锁，发生锁冲突几率低，并发高。

行锁必须有索引才能实现，否则会自动锁全表，那么就不是行锁了。

两个事务不能锁同一个索引，例如：

```sql
事务A先执行：  
select math from zje where math>60 for update;  

事务B再执行：  
select math from zje where math<60 for update；
```

这样的话，事务B是会阻塞的。如果事务B把 math索引换成其他索引就不会阻塞，但注意，换成其他索引锁住的行不能和math索引锁住的行有重复。

## mysql innoDB锁

InnoDB与MyISAM的最大不同有两点：

* 一是支持事务（TRANSACTION）
* 二是采用了行级锁。行级锁与表级锁本来就有许多不同之处

事务（Transaction）及其ACID属性事务是由一组SQL语句组成的逻辑处理单元，事务具有以下4个属性，通常简称为事务的ACID属性。

* 原子性（Atomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。
* 一致性（Consistent）：在事务开始和完成时，数据都必须保持一致状态。这意味着所有相关的数据规则都必须应用于事务的修改，以保持数据的完整性；事务结束时，所有的内部数据结构（如B树索引或双向链表）也都必须是正确的。
* 隔离性（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。这意味着事务处理过程中的中间状态对外部是不可见的，反之亦然。
* 持久性（Durable）：事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。

## 参考文档

* [http://blog.csdn.net/mysteryhaohao/article/details/51669741](http://blog.csdn.net/mysteryhaohao/article/details/51669741)
* [http://blog.csdn.net/yuwei19840916/article/details/3245107](http://blog.csdn.net/yuwei19840916/article/details/3245107)

