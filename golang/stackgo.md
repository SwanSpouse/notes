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

Tiny allocator.

Tiny allocator combines several tiny allocation requests into a single memory block. The resulting memory block is freed when all subobjects are unreachable. The subobjects must be noscan \(don't have pointers\), this ensures that the amount of potentially wasted memory is bounded.

Size of the memory block used for combining \(maxTinySize\) is tunable. Current setting is 16 bytes, which relates to 2x worst case memory wastage \(when all but one subobjects are unreachable\). 8 bytes would result in no wastage at all, but provides less opportunities for combining. 32 bytes provides more opportunities for combining, but can lead to 4x worst case wastage. The best case winning is 8x regardless of block size.

Objects obtained from tiny allocator must not be freed explicitly. So when an object will be freed explicitly, we ensure that its size &gt;= maxTinySize.

SetFinalizer has a special case for objects potentially coming from tiny allocator, it such case it allows to set finalizers for an inner byte of a memory block.

The main targets of tiny allocator are small strings and standalone escaping variables. On a json benchmark the allocator reduces number of allocations by ~12% and reduces heap size by ~20%.



