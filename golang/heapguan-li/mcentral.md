### mcentral

集中内存池，线程在本地分配失败后，尝试向mcentral申请，如果mcentral也没有资源，则尝试向mheap分配。  
全局 cache，mcache 不够用的时候向 mcentral 申请。

mcentral包含在mheap中。

mcentral和mcache的结构基本一致，都是mspan数组，不过在mcentral多了锁和占位用的padding。

mcache:

```go
mcache      *mcache

type mcache struct {
    ....
    alloc [numSpanClasses]*mspan // spans to allocate from, indexed by spanClass

    stackcache [_NumStackOrders]stackfreelist
}
```

mcentral:

```go
// central free lists for small size classes.
// the padding makes sure that the MCentrals are
// spaced CacheLineSize bytes apart, so that each MCentral.lock
// gets its own cache line.
// central is indexed by spanClass.
central [numSpanClasses]struct {
    mcentral mcentral
    pad      [sys.CacheLineSize - unsafe.Sizeof(mcentral{})%sys.CacheLineSize]byte
}
```

#### 

#### reference

* golang source code 1.10



