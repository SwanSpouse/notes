# 各版本的golang gc

## golang 的 GC

go 语言在 1.3 以前，使用的是比较蠢的传统 Mark-Sweep 算法。

* 1.3版本以前，golang的垃圾回收算法都非常简陋，然后其性能也广被诟病：go runtime在一定条件下（内存超过阈值或定期如2min），暂停所有任务的执行，进行mark-sweep操作，操作完成后启动所有任务的执行。在内存使用较多的场景下，go程序在进行垃圾回收时会发生非常明显的卡顿现象（Stop The World）。在对响应速度要求较高的后台服务进程中，这种延迟简直是不能忍受的！这个时期国内外很多在生产环境实践go语言的团队都或多或少踩过gc的坑。当时解决这个问题比较常用的方法是尽快控制自动分配内存的内存数量以减少gc负荷，同时采用手动管理内存的方法处理需要大量及高频分配内存的场景。

1.3 版本进行了一下改进，把 Sweep 改为了并行操作。

* 1.3版本开始go team开始对gc性能进行持续的改进和优化，每个新版本的go发布时gc改进都成为大家备受关注的要点。1.3版本中，go runtime分离了mark和sweep操作，和以前一样，也是先暂停所有任务执行并启动mark，mark完成后马上就重新启动被暂停的任务了，而是让sweep任务和普通协程任务一样并行的和其他任务一起执行。如果运行在多核处理器上，go会试图将gc任务放到单独的核心上运行而尽量不影响业务代码的执行。go team自己的说法是减少了50%-70%的暂停时间。

1.4 版本用go语言替换了很多C语言的代码

* 1.4版本（当前最新稳定版本）对gc的性能改动并不多。1.4版本中runtime很多代码取代了原生c语言实现而采用了go语言实现，对gc带来的一大改变是可以是实现精确的gc。c语言实现在gc时无法获取到内存的对象信息，因此无法准确区分普通变量和指针，只能将普通变量当做指针，如果碰巧这个普通变量指向的空间有其他对象，那这个对象就不会被回收。而go语言实现是完全知道对象的类型信息，在标记时只会遍历指针指向的对象，这样就避免了C实现时的堆内存浪费（解决约10-30%）。

1.5 版本进行了较大改进，使用了三色标记算法。go 1.5 在源码中的解释是“非分代的、非移动的、并发的、三色的标记清除垃圾收集器”

* 1.5版本go team对gc又进行了比较大的改进（1.4中已经埋下伏笔如write barrier的引入）,官方的主要目标是减少延迟。go 1.5正在实现的垃圾回收器是“非分代的、非移动的、并发的、三色的标记清除垃圾收集器”。分代算法上文已经提及，是一种比较好的垃圾回收管理策略，然1.5版本中并未考虑实现；我猜测的原因是步子不能迈太大，得逐步改进，go官方也表示会在1.6版本的gc优化中考虑。同时引入了上文介绍的三色标记法，这种方法的mark操作是可以渐进执行的而不需每次都扫描整个内存空间，可以减少stop the world的时间。 由此可以看到，一路走来直到1.5版本，go的垃圾回收性能也是一直在提升，但是相对成熟的垃圾回收系统（如java jvm和javascript v8），go需要优化的路径还很长。
* Since Go version 1.5, the collector is designed so that the stop the world task will take no more than 10 milliseconds out of every 50 milliseconds of execution time.

go 除了标准的三色收集以外，还有一个辅助回收功能，防止垃圾产生过快手机不过来的情况。这部分代码在 runtime.gcAssistAlloc 中。

但是golang 并没有分代收集，所以对于巨量的小对象还是很苦手的，会导致整个 mark 过程十分长，在某些极端情况下，甚至会导致 GC 线程占据 50% 以上的 CPU。

因此，当程序由于高并发等原因造成大量小对象的gc问题时，最好可以使用 sync.Pool 等对象池技术，避免大量小对象加大 GC 压力。

## reference

* [https://lengzzz.com/note/gc-in-golang](https://lengzzz.com/note/gc-in-golang)
* [https://studygolang.com/articles/7366](https://studygolang.com/articles/7366)
* [https://my.oschina.net/lubia/blog/175154](https://my.oschina.net/lubia/blog/175154)
* [http://newhtml.net/v8-garbage-collection/](http://newhtml.net/v8-garbage-collection/)

