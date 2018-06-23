### Heap管理

内置运行时的编程语言通常会抛弃传统的内存分配方式,改由自主管理内存。这样可以完成类似预分配、内存池等操作,以避开频繁地申请释放内存引起的系统调而导致的性能问题。当然,还有一个重要原因是为了更好地配合语言的垃圾回收机制。

Go内部实现也不例外，go runtime接管了所有的内存申请和释放动作。在os上层实现了内存池机制\(源自tcmalloc设计\)。

Go内存池管理的核心数据结构为mHeap。该结构管理从os申请的大块内存，将大块内存切分成多种不同大小的小块，每种小块由数据结构mspan表示。mheap通过数组+链表的方式来维护所有的空闲span。

应用程序在申请内存时一般都是以object为单位。在go runtime内部必须要计算object大小，然后找到合适的mspan大小。从这里面分配出想要的内存，返回给应用程序。

程序在向go runtime申请分配某种object所需内存时，会计算出object占用的内存空间，然后找到最接近的mspan\(因为mspan管理的块大小是按照固定倍数增长的方案。如一个17字节的object需要的块大小应该是24字节，存在轻微的内存浪费\)，将其分配出去。

### 核心数据结构

Go内存管理模块的核心数据结构比较少:

* **mheap**
  ：管理全局的从os申请的虚拟内存空间；
* **mspan**
  ：将mheap按照固定大小切分而成的细粒度的内存区块，每个区块映射了虚拟内存中的若干连续页面，页大小由Go内部定义；
* **mcache**
  ：与线程相关缓存，该结构的存在是为了减少内存分配时的锁操作，优化内存分配性能。
* **mcentral**
  ：集中内存池，线程在本地分配失败后，尝试向mcentral申请，如果mcentral也没有资源，则尝试向mheap分配。

### 数据结构关系图![](/assets/GoHeap数据关系图.png)

1. mheap管理向os申请、释放、组织span；

2. mcentral按照自己管理的块大小将span划分成多个小block，并分配给mcache；

3. mspan是数据的实际存储区域，按照central管理的块规格被切分成小block;

4. mcache管理不同规格\(块大小\)的mspan：规格相同的mspan被链接到同一个链表中。![](/assets/golang数据结构关系图.png)

### 参考

* [https://tracymacding.gitbooks.io/implementation-of-golang/content/](https://tracymacding.gitbooks.io/implementation-of-golang/content/)



