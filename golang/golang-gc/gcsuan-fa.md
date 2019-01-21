### GC算法

#### 三色标记算法 \(Tri-color marking\)

三色标记法是传统 Mark-Sweep 的一个改进，它是一个并发的 GC 算法。

原理如下，

1. 首先创建三个集合：白、灰、黑。
2. 将所有对象放入白色集合中。
3. 然后从根节点开始遍历所有对象（注意这里并不递归遍历），把遍历到的对象从白色集合放入灰色集合。
4. 之后遍历灰色集合，将灰色对象引用的对象从白色集合放入灰色集合，之后将此灰色对象放入黑色集合
5. 重复 4 直到灰色中无任何对象
6. 通过write-barrier检测对象有变化，重复以上操作
7. 收集所有白色对象（垃圾）


这个算法可以实现 "on-the-fly"，也就是在程序执行的同时进行收集，并不需要暂停整个程序。

但是也会有一个缺陷，可能程序中的垃圾产生的速度会大于垃圾收集的速度，这样会导致程序中的垃圾越来越多无法被收集掉。使用这种算法的是 Go 1.5、Go 1.6。

#### 分代收集

分代收集算法基于这样两个共识：

* 长生命周期的对象是不需要频繁扫描的。
* 内存分配存在这么一个事实 “most object die young”。

分代收集也是传统 Mark-Sweep 的一个改进。一般 GC 都会分三代，在 java 中称之为新生代（Young Generation）、年老代（Tenured Generation）和永久代（Permanent Generation）；在 .NET 中称之为第0代、第1代和第2代。

原理如下：

1. 新对象放入第 0 代
2. 当内存用量超过一个较小的阈值时，触发 0 代收集
3. 第 0 代幸存的对象（未被收集）放入第 1 代
4. 只有当内存用量超过一个较高的阈值时，才会触发 1 代收集
5. 2 代同理

因为 0 代中的对象十分少，所以每次收集时遍历都会非常快（比 1 代收集快几个数量级）。只有内存消耗过于大的时候才会触发较慢的 1 代和 2 代收集。

因此，分代收集是目前比较好的垃圾回收方式。使用的语言（平台）有 jvm、.NET 。

#### reference

* [https://lengzzz.com/note/gc-in-golang](https://lengzzz.com/note/gc-in-golang)
* [https://studygolang.com/articles/7366](https://studygolang.com/articles/7366)
* [https://my.oschina.net/lubia/blog/175154](https://my.oschina.net/lubia/blog/175154)
* [http://newhtml.net/v8-garbage-collection/](http://newhtml.net/v8-garbage-collection/)



