#### runtime/malloc.go

![](/assets/memory allocator.png)

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

5. 首先会扫描mspan的free bitmap去寻找是否有未使用的slot。如果存在就进行分配。分配mspan的slot是不需要进行加锁的。

6. 如果mspan中没有空闲的slot，那么就会想mcentral的mspan list申请空闲mspan，mcentral进行内存分配的时候，是需要进行加锁的。

7. 如果mcentral中的mspan list也为空， 那么就会去mheap中申请mspan。
8. 如果mheap同样没有足够的空间，那么就会向操作系统申请一组pages\(至少1MB\)。因为需要和操作系统进行交互，所以分配pages的代价会比较高。

   Sweeping an mspan and freeing objects on it proceeds up a similar hierarchy:

洺吉：清理mspan并释放其中的对象

1. If the mspan is being swept in response to allocation, it is returned to the mcache to satisfy the allocation.

2. Otherwise, if the mspan still has allocated objects in it, it is placed on the mcentral free list for the mspan's size class.

3. Otherwise, if all objects in the mspan are free, the mspan is now "idle", so it is returned to the mheap and no longer has a size class. This may coalesce it with adjacent idle mspans.

4. If an mspan remains idle for long enough, return its pages to the operating system.

5. 清理后的mspan首先会分配给mcache，供mcache进行分配。

6. 否则，如果mspan仍旧存在已分配的对象，它会被放入mcentral的free list中，以便后续继续存放相应大小的class。
7. 否则，如果mspan中的所有对象都已经被清理，那么这个mspan的状态为idle，它会被返回到mheap中，并且不会有class大小的限制，同时也可以和临近的idle mspan合并成一个，共同进行分配。
8. 如果mspan处于idle超过一定时长，就会分返回给操作系统。

Allocating and freeing a large object uses the mheap directly, bypassing the mcache and mcentral.

分配和释放大对象跳过mcache和mcentral，直接在mheap内存上进行分配。

Free object slots in an mspan are zeroed only if mspan.needzero is false. If needzero is true, objects are zeroed as they are allocated. There are various benefits to delaying zeroing this way:

洺吉：仅当mspan.needzero被设置为false的时候，mspan中的Free object slots 才会被初始化为0值；当mspan.needzero为true的时候，它们会在被分配的时候初始化为0值。延迟初始化0值会带来很多好处：

1. Stack frame allocation can avoid zeroing altogether. 
2. It exhibits better temporal locality, since the program is probably about to write to the memory.
3. We don't zero pages that never get reused.

4. 可以避免stack frame分配的时候进行统一清零操作。

5. （没有想好怎么进行翻译）

6. 对于没有使用到的pages，省去了初始化的工作。

```golang
// 分配小于32KB的对象
if size <= maxSmallSize {
      // 对于小于 16 byte 的内存块，mcache 有个专门的内存区域 tiny 用来分配，tiny 是指针，指向开始地址。
      if noscan && size < maxTinySize {
            off := c.tinyoffset
            // Align tiny pointer for required (conservative) alignment.
            if size&7 == 0 {
                  off = round(off, 8)
            } else if size&3 == 0 {
                  off = round(off, 4)
            } else if size&1 == 0 {
                  off = round(off, 2)
            }
            if off+size <= maxTinySize && c.tiny != 0 {
                  // The object fits into existing tiny block.
                  x = unsafe.Pointer(c.tiny + off)
                  c.tinyoffset = off + size
                  c.local_tinyallocs++
                  mp.mallocing = 0
                  releasem(mp)
                  return x
            }
            // Allocate a new maxTinySize block.
            span := c.alloc[tinySpanClass]
            v := nextFreeFast(span)
            if v == 0 {
                  v, _, shouldhelpgc = c.nextFree(tinySpanClass)
            }
            x = unsafe.Pointer(v)
            (*[2]uint64)(x)[0] = 0
            (*[2]uint64)(x)[1] = 0
            // See if we need to replace the existing tiny block with the new one
            // based on amount of remaining free space.
            if size < c.tinyoffset || c.tiny == 0 {
                  c.tiny = uintptr(x)
                  c.tinyoffset = size
            }
            size = maxTinySize
      } else {
      /*
      对于 size 介于 16 ~ 32K byte 的内存分配先计算应该分配的 sizeclass，然后去 mcache 里面 alloc[sizeclass] 申请，如果 mcache.alloc[sizeclass] 不足以申请，则 mcache 向 mcentral 申请，然后再分配。mcentral 给 mcache 分配完之后会判断自己需不需要扩充，如果需要则想 mheap 申请。
      */
            var sizeclass uint8
            if size <= smallSizeMax-8 {
                  sizeclass = size_to_class8[(size+smallSizeDiv-1)/smallSizeDiv]
            } else {
                  sizeclass = size_to_class128[(size-smallSizeMax+largeSizeDiv-1)/largeSizeDiv]
            }
            size = uintptr(class_to_size[sizeclass])
            spc := makeSpanClass(sizeclass, noscan)
            span := c.alloc[spc]
            v := nextFreeFast(span)
            if v == 0 {
                  v, span, shouldhelpgc = c.nextFree(spc)
            }
            x = unsafe.Pointer(v)
            if needzero && span.needzero != 0 {
                  memclrNoHeapPointers(unsafe.Pointer(v), size)
            }
      }
} else {
      // 分配大对象
      var s *mspan
      shouldhelpgc = true
      systemstack(func() {
            s = largeAlloc(size, needzero, noscan)
      })
      s.freeindex = 1
      s.allocCount = 1
      x = unsafe.Pointer(s.base())
      size = s.elemsize
}
```

#### reference

* golang source code 1.10



