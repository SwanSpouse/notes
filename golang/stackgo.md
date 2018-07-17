#### runtime/stakc.go

```go
// Stack describes a Go execution stack.
// The bounds of the stack are exactly [lo, hi),
// with no implicit data structures on either side.
type stack struct {
    lo uintptr
    hi uintptr
}
```

#### global stack list

// Global pool of spans that have free stacks.  Stacks are assigned an order according to size.

// order = log\_2\(size/FixedStack\) There is a free list for each order.

```go
var stackpool [_NumStackOrders]mSpanList
```

stackalloc -&gt; stackpoolalloc\(Allocates a stack from the free pool. Must be called with stackpoolmu held.\)-&gt;

#### malloc.go

Tiny allocator. 小内存分配器

Tiny allocator combines several tiny allocation requests into a single memory block. The resulting memory block is freed when all subobjects are unreachable. The subobjects must be noscan \(don't have pointers\), this ensures that the amount of potentially wasted memory is bounded.

小内存分配器会将很多个小内存分配请求组成一个memory block来进行分配。当memory block中的所有子对象全部成为不可达对象的时候，这个block就会被回收。这样保证被浪费的内存数量是可控的。

Size of the memory block used for combining \(maxTinySize\) is tunable. Current setting is 16 bytes, which relates to 2x worst case memory wastage \(when all but one subobjects are unreachable\). 8 bytes would result in no wastage at all, but provides less opportunities for combining. 32 bytes provides more opportunities for combining, but can lead to 4x worst case wastage. The best case winning is 8x regardless of block size.

memory block 的大小可以通过maxTinySize来设置。当前默认为16bytes，在最坏的情况下将有2x的内存被浪费；当设置为8bytes的时候不会有被浪费的内存，但是内存分配器将不会合并分配请求，每次都直接分配一个memory block。设置为32bytes的时候，内存分配器会合并更多的分配请求，这也将导致更多的内存有可能被浪费。当前效果最好的是设置为8x。

Objects obtained from tiny allocator must not be freed explicitly. So when an object will be freed explicitly, we ensure that its size &gt;= maxTinySize.

从小内存分配器分配到的对象不能被显式的释放。所以当一个对象需被显式的释放的时候，我们需要保证对象内存的大小&gt;= maxTinySize。

洺吉：怎么做叫做freed explicitly

SetFinalizer has a special case for objects potentially coming from tiny allocator, it such case it allows to set finalizers for an inner byte of a memory block.

The main targets of tiny allocator are small strings and standalone escaping variables. On a json benchmark the allocator reduces number of allocations by ~12% and reduces heap size by ~20%.

小内存分配器的主要对象是短字符串和逃逸变量。测试表明，它能够降低12%的堆内存分配请求，减小20%的堆内存大小。







#### reference

* 1.10 golang source code



