### Golang 内存模型概述

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



