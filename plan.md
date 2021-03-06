# 学习计划

## 基础知识相关：

* 1 计算机网络相关的知识：
  * TCP IP 路由, DNS MAC 子网
  * Socket、HTTP FTP等协议
* 2 操作系统相关的知识：
  * 内存管理、分页。调度相关算法。
  * CPU、多核。并发并行。调度算法。
  * Cache, 寄存器相关。
  * PV操作。
* 3 数据结构相关的知识：\(这部分知识主要参照LeetCode来进行复习吧。至少要把所有的Medium层次的题做完\)
  * 数组、堆栈
    * 堆：最大堆，最小堆, golang container中的heap.go就是最小堆的golang实现。
  * 链表等基本操作。
    * 链表的增 改 删 查等操作，原地逆序，双端链表。
  * 树相关。
    * 基本操作，平衡二叉树，字典树。
  * 图相关。
  * dfs bfs
    * 掌握基本的套路就行。
  * 动态规划
* pre4 数学知识：
  * 这里补充一下应该学习一些概率论和线性代数的知识。买了程序员数学。先看看这个。买了，还没看呢。

## 程序设计语言相关：

* 4 程序语言：
  * 4.1 Golang
    * Go源码阅读。
    * Golang内存模型。垃圾回收算法等。
    * Go线程模型。资源调度。
    * golang和Java之间的对比。
    * 语法树、从语言的设计层面来考虑考虑问题。
    * 逃逸分析
    * channel gorountine
  * 4.2 Java
    * Java虚拟机，相关资料 \(2018-08-09\)
      * [https://blogs.oracle.com/poonam/understanding-cms-gc-logs](https://blogs.oracle.com/poonam/understanding-cms-gc-logs)
      * [https://blogs.oracle.com/poonam/understanding-g1-gc-logs](https://blogs.oracle.com/poonam/understanding-g1-gc-logs)
    * Java类加载机制。这部分也算是Java虚拟机里面的。
    * Java中的设计模式。
    * Java源码相关。
  * 4.3 python
    * 用来写一些脚本和科学计算。简单了解
    * 看看python爬虫。
  * 4.4 shell
    * 能够进行一些简单的文本处理和文件操作。
  * 4.5 js
    * 这个也是能够进行简单的增改删查就好了。
  * 4.6 TODO
    * 这个留给新语言。

## 算法相关：

* 5 算法：
  * 5.1 基础算法，这部分和数据结构有重叠。
  * 5.2 机器学习常见算法。分类聚类模型。
  * 5.3 深度学习。应该了解这方面的知识。
  * 5.4 自然语言处理。复习一下概率学角度的自然语言处理。虽然已经过时了。但是还是有一些参考的意义。

## 工程相关：

* 6 工具
  * Git Git的基本操作。
    * 这个可以形成文档。常用命令。项目中常用到的几种场景。
  * Linux 常用命令，脚本
    * grep, awk, 等基本操作
    * Vim 基本操作
    * 查看系统资源，管理系统进程。
    * 编写简单的脚本程序。发个邮件等等。
    * crontab 基本操作。
  * Mysql
    * sql编写练习。
    * sql执行顺序。
    * mysql数据迁移。导入表、切分表、备份表等等。
    * mysql事务
    * mysql数据库的模型
    * 锁\(行锁、表锁、页锁、乐观锁、悲观锁等\)
  * maven
    * maven的常用命令。
    * maven打包项目的基本操作。
    * 定位maven管理的相关问题。
  * 测试相关工具
    * postman
    * JUnit
    * golang test
    * Selenium 操作浏览器进行测试。

## 大数据相关：

* 7 大数据相关
  * 7.1 了解相关的大数据框架、作用、进展、解决了什么问题。
  * 7.2 Redis
    * Redis 常用命令。
    * Redis 基本数据结构以及实现方式和原理。
    * Redis 内存-&gt;磁盘，
    * Redis 集群
    * Redis 源码阅读。
    * Redis 事务的实现。
  * 7.3 Hadoop、Storm、HBase、Kafka基本使用和原理：
    * 了解概念和解决的问题。应用场景。编写简单的相关应用。
    * 对现在的发展情况进行跟进。是否有其替代品。
  * 7.4 推荐系统
    * 常见的推荐系统算法。
    * 设计过的推荐系统总结。架构、各种算法。应用场景。改进的点。
  * 7.5 Docker
    * Docker的一些基本概念。
    * Docker的常用命令。
    * Docker的基本使用。
    * 学习一下Lain，TiDB这些。
  * 7.6 Codis
    * 分布式Redis实现的一种。
  * 7.7 TiDB
    * 了解一下这个，这个项目的人都很牛，而且分享出来的文档质量都非常的高。

## 设计模式：

* 8 设计模式
  * 熟悉相关的设计模式。
  * 编写一些简单的设计模式。画出设计模式的UML图。
  * 设计模式在各大开源项目中的应用。

## 后台开发的基本知识：

* 9 后台开发的基本知识
  * session, cookie的概念，使用，调试。debug问题。
  * Http请求流程。
  * 了解Sprint boot等Java web框架。
  * 搭建一个简单的web应用服务。
  * Orm框架。
  * 微服务、Nginx、Jenkins、Grafana等
  * 后台开发的基本工具。postman, 页面debug问题的方法。

## 前端相关的知识了解：

* 10 前端相关知识：
  * html css以及js， 和一些基本概念。比如说DOM树。
  * 学习使用Node js、Vue等。
  * 了解前端的一些概念。比如什么是React模式。
  * 浏览其相关知识，前端界面渲染。
  * 和后台的一样。搭建一个简单的项目。熟悉前端后台的相关知识。
    * 基本页面的展示。
    * 表格展示，动态表格。表格数据的增减删查。
    * 图表。饼图，各种图的展示。
    * 第三方库的使用。
    * 文件上传、下载。流媒体的播放。mp3、视频播放。
    * 页面动态效果。

## 了解一下现在比较有前景的业务点：

* 区块链技术
* 机器学习

## 后记

* 上面的每个点无先后顺序之分，只是想到了哪个就写哪个上去了。
* 目前考虑到的只有这些，随着学习的进行我再不断的进行完善。

## 参考

* [https://github.com/jwasham/coding-interview-university/blob/master/translations/README-cn.md](https://github.com/jwasham/coding-interview-university/blob/master/translations/README-cn.md)

