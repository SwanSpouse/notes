# golang gc

## GC的触发条件

主动触发：runtime.GC\(\)

被动触发：

方法mallocgc\(\)中，如果tiny object和small object 都没有从mcache中获取到内存（需要向mcentral申请内存的时候）；或者分配large object的时候。golang认为这是一项很重的工作\(a heavy weight allocation\)。此时变量shouldhelpgc会被置为true，表示需要进行检查，看是否达到了触发gc的条件。

When heap\_live ≥ gc\_trigger, the mark phase will start.

This is also the heap size by which proportional sweeping must be complete.

所以，每次缓存中内存不足或者分配大对象的时候，都要进行一次检查，看是否达到了GC的触发的条件；如果符合条件，就开始进行GC操作。

## GC的三种模式

```go
gcBackgroundMode gcMode = iota // concurrent GC and sweep
gcForceMode                    // stop-the-world GC now, concurrent sweep
gcForceBlockMode               // stop-the-world GC now and STW sweep (forced by user)
```

## 垃圾回收的主要流程

![](../../../.gitbook/assets/golang_gc.png)

关于上图有几点需要说明的是。

* 首先从 root 开始遍历，root 包括全局指针和 goroutine 栈上的指针。
* mark 有两个过程。 1. 从 root 开始遍历，标记为灰色。遍历灰色队列。 2. re-scan 全局指针和栈。因为 mark 和用户程序是并行的，所以在过程 1 的时候可能会有新的对象分配，这个时候就需要通过写屏障（write barrier）记录下来。re-scan 再完成检查一下。 3. Stop The World 有两个过程。 1. 第一个是 GC 将要开始的时候，这个时候主要是一些准备工作，比如 enable write barrier 2. 第二个过程就是上面提到的 re-scan 过程。如果这个时候没有 stw，那么 mark 将无休止。

另外针对上图各个阶段对应 GCPhase 如下：

* Off: \_GCoff
* Stack scan ~ Mark: \_GCmark
* Mark termination: \_GCmarktermination

## reference

* [http://legendtkl.com/2017/04/28/golang-gc/](http://legendtkl.com/2017/04/28/golang-gc/)
* golang source code 1.10

