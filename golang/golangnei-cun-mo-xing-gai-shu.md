### Golang 内存模型概述

runtime system通常会抛弃传统的内存分配方式，改为自主管理。这样可以完成类似预分配、内存池等操作，以避开系统调用带来的性能问题。当然，还有一个重要的原因是为了更好地配合垃圾回收。

#### golang内存管理的基本策略

每次从操作系统申请一块大内存（1MB\)，以减少系统调用。

将申请到的大块内存按照特定大小预先切分成小块，构成链表。

为对象分配内存时，只需要从大小合适的链表提取一个小块即可。

回收内存对象时，将该小块内存重新归还原链表，以便复用。

如闲置内存过多，则尝试归还部分内存给操作系统，降低整体开销。

#### Stack, Heap, and Data Segement

There are 3 places memory can be allocated: the stack, the heap, and the data segment.

#### Stack

Typically, a functions parameters and local variables are allocated on the stack.

Each goroutine has its own stack; thus, no synchronization\(e.g., locking\) is necessary.

Goroutine stacks are allocated on the heap. If the stack needs to grow beyond the amount allocated for it, then heap operations \(allocate new, copy old to new, free old\) will occur.

#### Heap

Some memory allocations are put on the heap. Unlike the stack, the heap does not have a single partition of allocated and free regions. Rather, there is a set of of free regions. A data structure must be used to implement this set of free regions. When an item is allocated, it is removed from the free regions. When an item is freed, it is added back to the set of free regions.

Unlike the stack, the heap is not owned by one goroutine, so manipulating the set of free regions in the heap requires synchronization \(e.g., locking\).

#### Data Segement

Memory can also be allocated in the data segment. This is where global variables are stored. The data segement is defined at compile time and therefore does not grow and shrink at runtime.

_洺吉：这部分我在源码中并没有找到对应的内存区域。_

#### reference

* [https://dougrichardson.org/2016/01/23/go-memory-allocations.html](https://dougrichardson.org/2016/01/23/go-memory-allocations.html)



