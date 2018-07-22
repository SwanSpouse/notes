### mcentral

集中内存池，线程在本地分配失败后，尝试向mcentral申请，如果mcentral也没有资源，则尝试向mheap分配。  
全局 cache，mcache 不够用的时候向 mcentral 申请。

mcentral包含在mheap中。

