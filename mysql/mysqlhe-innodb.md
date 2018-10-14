### 数据库和实例

在 MySQL 中，实例和数据库往往都是一一对应的，而我们也无法直接操作数据库，而是要通过数据库实例来操作数据库文件，可以理解为数据库实例是数据库为上层提供的一个专门用于操作的接口。

在 Unix 上，启动一个 MySQL 实例往往会产生两个进程，`mysqld`就是真正的数据库服务守护进程，而`mysqld_safe`是一个用于检查和设置`mysqld`启动的控制程序，它负责监控 MySQL 进程的执行，当`mysqld`发生错误时，`mysqld_safe`会对其状态进行检查并在合适的条件下重启。

### Mysql架构

MySQL 从第一个版本发布到现在已经有了 20 多年的历史，在这么多年的发展和演变中，整个应用的体系结构变得越来越复杂：  
![](/assets/mysql架构图.png)

最上层用于连接、线程处理的部分并不是 MySQL 『发明』的，很多服务都有类似的组成部分；第二层中包含了大多数 MySQL 的核心服务，包括了对 SQL 的解析、分析、优化和缓存等功能，存储过程、触发器和视图都是在这里实现的；而第三层就是 MySQL 中真正负责数据的存储和提取的存储引擎，例如：[InnoDB](https://en.wikipedia.org/wiki/InnoDB)、[MyISAM](https://en.wikipedia.org/wiki/MyISAM)等，文中对存储引擎的介绍都是对 InnoDB 实现的分析。

### reference

* [https://draveness.me/mysql-innodb](https://draveness.me/mysql-innodb)



