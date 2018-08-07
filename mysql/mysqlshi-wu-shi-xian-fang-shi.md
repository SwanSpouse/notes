### Mysql事务实现

原子性、一致性、持久性通过数据库的redo log和undo log来完成。

隔离性通过锁来实现。

redo和undo的作用都可以视为时一种恢复操作，redo恢复提交事务修改的页操作，而undo回滚行记录到某个特定的版本。因此两者记录的内容不同，redo通常是物理日志，记录的是页的物理修改操作。undo是逻辑日志，根据每行记录进行记录。

#### Redo基本概念

Redo log（重做日志）**重做日志用来实现事务的持久性**。由两部分组成：一是内存中的**重做日志缓冲**（redo log buffer），其是易失的；二是**重做日志文件**（redo log file），其是持久的。

Innodb当事务提交时，必须先将该事务的所有日志写入到重做日志文件进行持久化，待事务的commit操作完成才算完成。

innodb中重做日志由redo log和undo log组成。**redo log用来保证事务的持久性，undo log用来帮助事务回滚及MVCC的功能。**redo log基本上时顺序写的，在数据库运行时不需要对redo log的文件进行读取操作。而undo log是需要进行随机读写的。

#### log block {#log-block}

innodb重做日志以512字节进行存储，大小和磁盘扇区大小一样，重做日志缓存、重做日志文件都是以块的方式进行保存的，称之为重做日志块（redo log block），每块的大小为512字节。

log buffer是由log block组成，在内部log buffer好似一个数组。

#### log group {#log-group}

重做日志组，其中有多个重做日志文件。它是一个逻辑上的概念。

log buffer根据一定的规则将内存中的log block刷新到磁盘。  
1.事务提交时  
2.当log buffer中有一般的内存空间被使用  
3.log checkpoint时

#### 恢复 {#恢复}

Innodb在启动时不管上次数据库运行时是否正常关闭，都会尝试进行恢复操作。重做日志时物理日志，因此恢复速度快。

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

* https://blog.csdn.net/qq\_27602093/article/details/77069765







