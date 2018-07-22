### Golang 内存管理

在这里主要来描述golang是怎样进行内存管理的和垃圾回收的。

go的内存管理参考了tcmalloc（thread-caching malloc\)的思想。思想主要是通过建立全局缓存和线程私有缓存，两级缓存，来提供内存分配的效率。

参照Java垃圾回收的一些概念和步骤，看golang是怎样实现垃圾回收的。同时比较一下两种垃圾回收机制的优劣。

主要解决一下几个问题：

* 运行时内存区域。golang的运行时内存都包括了哪几部分，各自存储着什么样的对象。

  * 暂时可以将golang的运行时内存划分为堆和栈。有个全局变量mheap和一个全局变量stackpool。在创建一个调度器的时候会对全局变量stackpool和全局变量mheap进行初始化。

* 如何给对象分配内存。将对象分配到哪部分内存里。

  * 首先说对象的分配。不是说一些局部变量的分配。对象的分配分为三种：tiny object\(&lt;16B\)、 small object\(\[16B, 32KB\]\)、large object\(&gt; 32KB\)，tiny object直接分配在mcache专门用来存储tiny object的区域；large object 直接分配在堆上，small object会分配在P的mcache中。

* 对象存活判断。这个是垃圾回收的前提。

  * 对象存活的判断，golang使用的是三色标记法。

* 在何时进行垃圾回收。所谓的安全点、安全区。
  * 在golang gc里面进行了介绍。如果进行了一个heavy weight allocation，就会对是否需要进行gc进行一次检查。
* 如何进行垃圾回收。垃圾回收算法。
  * golang采用的是mark sweep来进行垃圾回收。
* 分析gc log，通过命令行查看当前GC的状态和信息。
  * go run -gcflags --params
* 能够根据gc的情况作出针对性的优化 。
* 对照Java和golang，查看各自的垃圾回收的特点，然后进行对照。



