#### mcache

与线程相关缓存，该结构的存在是为了减少内存分配时的锁操作，优化内存分配性能。  
per-P cache，可以认为是 local cache。![](/assets/golang数据结构关系图.png)

mcache拥有一个大小为 67 的指针（指针指向 mspan ）数组（\_NumSizeClasses = 67），每个数组元素用来包含特定大小的块。当要分配内存大小时，为 object 在 alloc 数组中选择合适的元素来分配。67 种块大小为 0，8 byte, 16 byte, …

_洺吉：这里的数组的值是怎么分配的呢？最开始的时候是8byte，然后是16byte，到最后4096 byte。_



