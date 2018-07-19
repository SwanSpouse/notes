#### tcmalloc

tcmalloc是Google开源的一个内存管理库，常见的内存分配器还有glibc的ptmalloc和google的jemalloc。相比于ptmalloc，tcmalloc性能更好，特别适用于高并发场景。 作为glibc malloc的替代品。目前已经在chrome、safari等知名软件中运用。

根据官方测试报告，ptmalloc在一台2.8GHz的P4机器上（对于小对象）执行一次malloc及free大约需要300纳秒。而TCMalloc的版本同样的操作大约只需要50纳秒。

#### tcmalloc具体策略

tcmalloc分配的内存主要来自两个地方：全局缓存堆和进程的私有缓存。对于一些小容量的内存申请是用进程的私有缓存，私有缓存不足的时候可以再从全局缓存申请一部分作为私有缓存。对于大容量的内存申请则需要从全局缓存中进行申请。而大小容量的边界就是32k。缓存的组织方式是一个单链表数组，数组的每个元素是一个单链表，链表中的每个元素具有相同的大小。

#### Small Object Allocation

小对象内存分配默认会分配86个不同大小的块，而这些块的大小并没有明确说明，需要查一下源码。每种大小的块的数组的长度都采用使用了才初始化，有点类似于lazy-initialize。

![](/assets/tcmalloc小对象.png)  


#### Big Object Allocation

对于大于32k的内存申请，使用全局内存来分配。全局内存的组织也是单链表数组，数组长度为256，分别对用1 page大小, 2 page大小（1 page=4k）.  
![](/assets/tcmalloc大对象分配.png)

#### Span {#Span}

tcmalloc使用span来管理内存分页，一个span可以包含几个连续分页。span的状态只有未分配、作为大对象分配、作为小对象分配。

![](/assets/tcmalloc_span.png)  


#### go的内存分配

go语言的内存分配并不是和tcmalloc一模一样。

* 局部缓存并不是分配给进程或者线程，而是分配给P（这个还需要说一下go的goroutine实现）
* go的GC是stop the world，并不是每个进程单独进行GC。
* span的管理更有效率

#### reference

* http://legendtkl.com/2015/12/11/go-memory/

* http://goog-perftools.sourceforge.net/doc/tcmalloc.html

* https://github.com/gperftools/gperftools



