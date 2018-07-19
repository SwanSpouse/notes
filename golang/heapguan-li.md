### Heap管理

内置runtime system的编程语言通常会抛弃传统的内存分配方式,改由自主管理内存。这样可以完成类似预分配、内存池等操作,以避开频繁地申请释放内存引起的系统调而导致的性能问题。当然,还有一个重要原因是为了更好地配合语言的垃圾回收机制。

Go内部实现也不例外，go runtime接管了所有的内存申请和释放动作。在os上层实现了内存池机制\(源自tcmalloc设计\)。

Go内存池管理的核心数据结构为mHeap。该结构管理从os申请的大块内存，将大块内存切分成多种不同大小的小块，每种小块由数据结构mspan表示。mheap通过数组+链表的方式来维护所有的空闲span。

应用程序在申请内存时一般都是以object为单位。在go runtime内部必须要计算object大小，然后找到合适的mspan大小。从这里面分配出想要的内存，返回给应用程序。

程序在向go runtime申请分配某种object所需内存时，会计算出object占用的内存空间，然后找到最接近的mspan\(因为mspan管理的块大小是按照固定倍数增长的方案。如一个17字节的object需要的块大小应该是24字节，存在轻微的内存浪费\)，将其分配出去。

### 核心数据结构

Go内存管理模块的核心数据结构比较少:

#### **mFixAlloc**

runtime/mfixalloc.go![](/assets/mfixAlloc.png)`list`指针上挂的一个链表，这个链表的每个节点是一个固定大小的内存块，cachealloc中的list存储的内存块大小为`sizeof(MCache)`，而spanalloc中的list存储的内存块大小为`sizeof(MSpan)`。`chunk`指针始终挂载的是一个128k大的内存块。 FixAlloc提供了三个API，分别是runtime·FixAlloc\_Init、runtime·FixAlloc\_Alloc和runtime·FixAlloc\_Free。

使用FixAlloc分配MCache和MSpan对象的时候，首先是查找FixAlloc的list链表，如果list不为空，就直接拿一个内存块返回使用; 如果list为空，就把焦点转移到chunk上去，如果128k的chunk内存中有足够的空间，就切割一块内存出来返回使用，如果chunk内存没有剩余内存的话，就从操作系统再申请128k内存替代老的chunk。FixAlloc的固定对象分配逻辑就这么简单，相反释放逻辑更简单了，释放的对象就是直接放到list中，并不会返回给操作系统。当然mcache的个数基本是稳定的，也就是底层线程个数，但span对象就不一定那么稳定了，所以FixAlloc的内存可能增长的因素就是span的对象太多。

_洺吉：我看代码好像chunk的大小是16k的呀。\_FixAllocChunk  = 16 &lt;&lt; 10               // Chunk size for FixAlloc 这不是16K吗？_

#### **mheap**

管理全局的从os申请的虚拟内存空间；  
当 mcentral 也不够用的时候，通过 mheap 向操作系统申请。

#### **mspan**

span 在 tcmalloc 中作为一种管理内存的基本单位而存在。

mspan 用来管理一组组page对象。page就是一个4k大小的内存块而已。span就是将这一个个连续的page给管理起来，注意是连续的page。![](/assets/mspan.png)

npages 表示是此span存储的page的个数，`start`可以看作一个page指针，指向第一个page，有了第一个page当然就可以算出后面的任何一个page的起始地址了，因为span管理的始终是连续的一组page。这里需要注意start的类型是PageID，由此可以看出这个start保存的并不是第一个page的起始地址，而是第一个page的id值。这个id值是如何算出来的呢？其实给每个page算一个id，是非常简单的事情，只要将这个page的的地址除以4096取整\(伪代码：`page_addr>>20`\)即可，当然前提是已经保证好了每个page按4k对齐。是不是觉得很精妙，这样一来每个page都有一个整数id了，并且任何一个内存地址都可以通过移位算出这个地址属于哪个page，这个很重要。

`sizeclass`如果是0的话，就代表这个span是用来分配大对象的，其他值都是分配小对象了。在分配小对象的时候，start字段维护的所有page，最后将会被切分成一个一个的连续内存块，内存块大小当然就是小对象的大小，这些切分出来的内存块将被链接成为一个链表挂在freelist字段上。分配大对象的时候，freelist就没什么用了。size为3的话，那么就它将被分成32byte的小块。因为0,1,2,3分别对应分成0,8byte,16byte,32byte。

#### **mcache**

与线程相关缓存，该结构的存在是为了减少内存分配时的锁操作，优化内存分配性能。  
per-P cache，可以认为是 local cache。![](/assets/golang数据结构关系图.png)

mcache拥有一个大小为 67 的指针（指针指向 mspan ）数组（\_NumSizeClasses = 67），每个数组元素用来包含特定大小的块。当要分配内存大小时，为 object 在 alloc 数组中选择合适的元素来分配。67 种块大小为 0，8 byte, 16 byte, …

#### **mcentral**

集中内存池，线程在本地分配失败后，尝试向mcentral申请，如果mcentral也没有资源，则尝试向mheap分配。  
全局 cache，mcache 不够用的时候向 mcentral 申请。

mcentral包含在mheap中。

#### 数据结构关系图![](/assets/GoHeap数据关系图.png)

1. mheap管理向os申请、释放、组织span；

2. mcentral按照自己管理的块大小将span划分成多个小block，并分配给mcache；

3. mspan是数据的实际存储区域，按照central管理的块规格被切分成小block;

4. mcache管理不同规格\(块大小\)的mspan：规格相同的mspan被链接到同一个链表中。

#### 参考

* [https://tracymacding.gitbooks.io/implementation-of-golang/content/](https://tracymacding.gitbooks.io/implementation-of-golang/content/)
* [http://legendtkl.com/2017/04/02/golang-alloc/](http://legendtkl.com/2017/04/02/golang-alloc/)



