### Mysql事务实现

原子性、一致性、持久性通过数据库的redo log和undo log来完成。

隔离性通过锁来实现。

#### 重做日志

InnoDB有buffer pool（简称bp）。bp是数据库页面的缓存，对InnoDB的任何修改操作都会首先在bp的page上进行，然后这样的页面将被标记为dirty并被放到专门的flush list上，后续将由master thread或专门的刷脏线程阶段性的将这些页面写入磁盘（disk or ssd）。这样的好处是避免每次写操作都操作磁盘导致大量的随机IO，阶段性的刷脏可以将多次对页面的修改merge成一次IO操作，同时异步写入也降低了访问的时延。然而，如果在dirty page还未刷入磁盘时，server非正常关闭，这些修改操作将会丢失，如果写入操作正在进行，甚至会由于损坏数据文件导致数据库不可用。为了避免上述问题的发生，Innodb将所有对页面的修改操作写入一个专门的文件，并在数据库启动时从此文件进行恢复操作，这个文件就是redo log file。这样的技术推迟了bp页面的刷新，从而提升了数据库的吞吐，有效的降低了访问时延。带来的问题是额外的写redo log操作的开销（顺序IO，当然很快），以及数据库启动时恢复操作所需的时间。

redo和undo的作用都可以视为时一种恢复操作，redo恢复提交事务修改的页操作，而undo回滚行记录到某个特定的版本。因此两者记录的内容不同，redo通常是物理日志，记录的是页的物理修改操作。undo是逻辑日志，根据每行记录进行记录。

重做日志**用来实现事务的持久性\(Duration\)**。由两部分组成：

* 一是内存中的**重做日志缓冲**（redo log buffer），其是易失的；
* 二是**重做日志文件**（redo log file），其是持久的。

InnoDB是事务的存储引擎，其通过Force Log at Commit 机制实现事务的持久性，即当事务提交commit时，必须先将事务的所有日志写入到重做日志文件进行持久化，待事务COMMIT操作完成才算完成，这里的日志指重做日志，在InnoDB存储引擎中，由两部分组成，即redo log 和undo log. **redo log用来保证事务的持久性，undo log 用来帮主事务回滚及MVCC的功能。**redo log 基本是顺序写的，在数据库运行时不需要对redo log的文件进行读取操作。而undo log 是需要进行随机读写的

#### log block

在InnoDB存储引擎中，重做日志都是以512字节进行存储的，这意味着重做日志缓存、重做日志文件块都是以块block的方式进行保存的，称为重做日志块\(redo log block\)每块的大小512字节。

log buffer是由log block组成，在内部log buffer好似一个数组。

#### log group {#log-group}

重做日志组，其中有多个重做日志文件。它是一个逻辑上的概念。

log buffer根据一定的规则将内存中的log block刷新到磁盘。  
1.事务提交时  
2.当log buffer中有一般的内存空间被使用  
3.log checkpoint时

#### 恢复 {#恢复}

Innodb在启动时不管上次数据库运行时是否正常关闭，都会尝试进行恢复操作。重做日志是物理日志，因此恢复速度快。

例如对于INSERT操作，其记录的时每个页上的变化。对于下面的表：  
`CREATE TABLE t （a int， b int， primary key（a）， key（b））;`

若执行SQL语句：  
`INSERT INTO t SELECT 1,2；`

由于需要对聚集索引页和辅助索引页进行操作，其记录的重做日志大致为：

```sql
page（2,3），offset 32， value 1,2#聚集索引
page（2,4），offset 64，value 2#辅助索引
```

记录的是页的物理修改操作，若插入设计B+树的split，可能会有更多的页需要记录日志。

#### undo基本概念 {#基本概念-1}

重做日志记录了事务的行为，可以很好地通过其对页进行“重做”操作。但是事务有时还需要进行回滚操作，这时就需要undo。

在对数据库进行修改时，innodb不但会产生redo，还会产生一定量的undo。如果用户执行事务或语句由于某种原因失败了，又或者用户用一条ROLLBACK语句请求回滚，就可以利用这些undo信息将数据回滚到修改之前的样子。

用户执行insert 10w条记录，事务会导致分配一个新的段，即表空间会增大，但rollback，事务回滚，表空间大小不会改变。

当innodb回滚时，实际执行先前相反的工作。

INSERT操作，会执行delete。delete操作，会执行INSERT。update操作，会执行相反的update操作。

#### undo 存储管理 {#undo-存储管理}

innodb对undo同样采用段的方式。innodb有rollback segment，每个回滚段记录了1024个undo log segment。

当事务提交时，innodb会做两件事情：  
1.将undo log放入列表中，以供之后的purge操作  
2.判断undo log所在的页是否可以重用，若可以分配给下个事务使用。

#### reference

* [https://blog.csdn.net/qq\_27602093/article/details/77069765](https://blog.csdn.net/qq_27602093/article/details/77069765)



