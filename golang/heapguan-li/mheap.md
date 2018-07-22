### mheap

管理全局的从os申请的虚拟内存空间；  
当 mcentral 也不够用的时候，通过 mheap 向操作系统申请。

所有的 spans 都是通过 mheap\_ 申请，所有申请过的 mspan 都会记录在 allspans。结构体中的 lock 就是用来保证并发安全的。

#### arena

arena 是 Golang 中用于分配内存的连续虚拟地址区域。由 mheap 管理，堆上申请的所有内存都来自 arena。那么如何标志内存可用呢？操作系统的常见做法用两种：一种是用链表将所有的可用内存都串起来；另一种是使用位图来标志内存块是否可用。结合上面一条 spans，内存的布局是下面这样的。

![](/assets/mheap_arena.png)

![](/assets/mheap.png)

