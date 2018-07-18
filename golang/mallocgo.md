#### runtime/malloc.go

Memory allocator

This was originally based on tcmalloc, but has diverged quite a bit. [http://goog-perftools.sourceforge.net/doc/tcmalloc.html](http://goog-perftools.sourceforge.net/doc/tcmalloc.html)

The main allocator works in runs of pages. Small allocation sizes \(up to and including 32 kB\) are rounded to one of about 70 size classes, each of which has its own free set of objects of exactly that size. Any free page of memory can be split into a set of objects of one size class, which are then managed using a free bitmap.

The allocator's data structures are:

* fixalloc: a free-list allocator for fixed-size off-heap objects, used to manage storage used by the allocator.
* mheap: the malloc heap, managed at page \(8192-byte\) granularity.
* mspan: a run of pages managed by the mheap.
* mcentral: collects all spans of a given size class.
* mcache: a per-P cache of mspans with free space.
* mstats: allocation statistics.

Allocating a small object proceeds up a hierarchy of caches：

洺吉：golang runtime system 对内存实现了分级缓存。给小对象进行内存分配，不同情况下会有不同的代价。

1. Round the size up to one of the small size classes and look in the corresponding mspan in this P's mcache. Scan the mspan's free bitmap to find a free slot. If there is a free slot, allocate it. This can all be done without acquiring a lock.
2. If the mspan has no free slots, obtain a new mspan from the mcentral's list of mspans of the required size class that have free space. Obtaining a whole span amortizes the cost of locking the mcentral.
3. If the mcentral's mspan list is empty, obtain a run of pages from the mheap to use for the mspan.
4. If the mheap is empty or has no page runs large enough, allocate a new group of pages \(at least 1MB\) from the operating system. Allocating a large run of pages amortizes the cost of talking to the operating system.

1. 首先会扫描mspan的free bitmap去寻找是否有未使用的slot。如果存在就进行分配。分配mspan的slot是不需要进行加锁的。
2. 如果mspan中没有空闲的slot，那么就会想mcentral的mspan list申请空闲mspan，mcentral进行内存分配的时候，是需要进行加锁的。
3. 如果mcentral中的mspan list也为空， 那么就会去mheap中申请mspan。
4. 如果mheap同样没有足够的空间，那么就会向操作系统申请一组pages\(至少1MB\)。因为需要和操作系统进行交互，所以分配pages的代价会比较高。

 Sweeping an mspan and freeing objects on it proceeds up a similar hierarchy:

洺吉：清理mspan并释放其中的对象

    1. If the mspan is being swept in response to allocation, it is returned to the mcache to satisfy the allocation.

    2. Otherwise, if the mspan still has allocated objects in it, it is placed on the mcentral free list for the mspan's size class.

    3. Otherwise, if all objects in the mspan are free, the mspan is now "idle", so it is returned to the mheap and no longer has a size class. This may coalesce it with adjacent idle mspans.

    4. If an mspan remains idle for long enough, return its pages to the operating system.

1. 清理后的mspan首先会分配给mcache，供mcache进行分配。
2. 否则，如果mspan仍旧存在已分配的对象，它会被放入mcentral的free list中，以便后续继续存放相应大小的class。
3. 否则，如果mspan中的所有对象都已经被清理，那么这个mspan的状态为idle，它会被返回到mheap中，并且不会有class大小的限制，同时也可以和临近的idle mspan合并成一个，共同进行分配。
4. 如果mspan处于idle超过一定时长，就会分返回给操作系统。

Allocating and freeing a large object uses the mheap directly, bypassing the mcache and mcentral. 

分配和释放大对象跳过mcache和mcentral，直接在mheap内存上进行分配。

Free object slots in an mspan are zeroed only if mspan.needzero is false. If needzero is true, objects are zeroed as they are allocated. There are various benefits to delaying zeroing this way:

洺吉：仅当mspan.needzero被设置为false的时候，mspan中的Free object slots 才会被初始化为0值；当mspan.needzero为true的时候，它们会在被分配的时候初始化为0值。延迟初始化0值会带来很多好处：

1. Stack frame allocation can avoid zeroing altogether. 
2. It exhibits better temporal locality, since the program is probably about to write to the memory.
3. We don't zero pages that never get reused.  

1. 可以避免stack frame分配的时候进行统一清零操作。
2. （没有想好怎么进行翻译）
3. 对于没有使用到的pages，省去了初始化的工作。

























#### reference 

* golang source code 1.10



