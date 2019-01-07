### Mysql事务实现

原子性、一致性、持久性通过数据库的redo log和undo log来完成。

隔离性通过锁来实现。

#### redo

在InnoDB存储引擎中，事务日志通过重做\(redo\)日志文件和InnoDB存储引擎的日志缓冲\(InnoDB Log Buffer\)来实现。当开始一个事务时，会记录该事务的一个LSN\(Log Sequence Number, 日志序列号\)；当事务执行时，会往InnoDB存储引擎的日志缓冲里插入事务日志；当事务提交时，必须将InnoDB存储引擎的日志缓冲写入磁盘\(默认的实现，即innodb\_flush\_log\_at\_trx\_commit=1\)。也就是在写数据前，需要先写日志。这种方式称为预写日志方式（Write-Ahead Logging, WAL）。

InnoDB存储引擎通过预写日志的方式来保证事务的完整性。这意味着磁盘上存储的数据页和内存缓冲池中的页是不同步的，对于内存缓冲池中的页的修改，先是写入重做日志文件，然后再写磁盘，因此是一种一步的方式。可以通过命令SHOW ENGINE INNODB STATUS来查看当前磁盘和日志的差距。

#### undo

重做日志记录了事务的行为，可以很好地通过其进行“重做” 。但是事务有时还需要撤销，这时就需要undo。undo与redo正好相反，对于数据库进行修改时，数据库不但会产生redo，而且还会产生一定量的undo，即使你执行的事务或者语句由于某种原因失败了，或者如果你用一条ROLLBACK语句请求回滚，就可以利用这些undo信息将数据回滚到修改之前的样子。与redo不同的是，redo存放在重做日志文件中，undo存放在数据库内部的一个特殊段\(segment\)中，这称为undo段\(undo segment\)，undo段位于共享表空间内。可以通过py\_innodb\_page\_info.py工具，来查看当前共享表空间中undo的数量。

我们通常对undo有这样的误解：undo用于将数据库物理地恢复到执行语句或者事务之前的样子——但事实并非如此。数据库只是逻辑地恢复到原来的样子，所有修改都被逻辑地取消，但是数据结构本身在回滚之后可能大不相同，应为在多用户并发系统中，可能会有数十、数百甚至数千个并发事务。数据库的主要任务就是协调对于数据记录的并发访问。如一个事务在修改当前一个页中的某几条记录，但同时还有别的事务在对同一个页面的另几条记录进行修改。因此，不能将一个页回滚到事务开始的样子，因为这样会影响其他事务正在进行的工作。

#### 事务控制语句

在Mysql命令的默认设置下，事务都是自动提交的，即执行SQL语句后就会马上执行COMMIT操作。因此开始一个事务，必须使用BEGIN,START TRANSACTION，或者执行SET AUTOCOMMIT=0，以禁用当前会话的自动提交。

* START TRANSACTION \| BEGIN ： 显示地开启一个事务。
* COMMIT : COMMIT会提交你的事务，并使得已对数据库做的所有修改成为永久性的。
* ROLLBACK：ROLLBACK会结束你的事务，并撤销正在进行的所有未提交的修改。
* SAVEPOINT identifier: SAVEPOINT允许你在事务中创建一个保存点，一个事物中可以有多个SAVEPOINT。
* RELEASE SAVEPOINT identifier:删除一个事务的保存点，当没有一个保存点执行这条语句时，会抛出一个异常。
* ROLLBACK TO \[SAVEPOINT\] identifier： 把事务滚回到标记点，而不滚回在此标记点之前的任何工作。
* SET TRANSACTION：用来设置事务的隔离级别。

#### reference

* 《Mysql技术内幕：InnoDB存储引擎》



