# volatile

## volatile 型变量的特殊规则

当一个变量被定义为volatile之后，它将具备两种特性，第一是保证此变量对所有线程的可见性，这里的“可见性”是指当一条线程修改了这个变量的值，新值对于其他线程来说是可以立即得知的。而普通变量不能做到这一点，普通变量的值在线程间传递需要通过主内存来完成。例如，线程A修改了一个普通变量的值，然后想主内存进行回写，另外一条线程B在线程A回写完成了之后再从主内存进行读取操作，新变量值才会对线程B可见。

由于volatile变量只能保证可见性，在不符合以下两条规则的运算场景中，我们仍要通过加锁来保证原子性：

* 运算结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值。
* 变量不需要与其他的状态变量共同参与不变约束。

volatile就是一个践行happens-before的关键字。happens-before指的是线程**接收其他线程修改共享变量的消息**与该线程**读取共享变量**的**先后关系**。如果没有happens-before原则，岂不是相当于一个线程读取自己的共享变量副本时，其他线程修改这个变量的消息还没有同步过来？这就是可见性问题。

**volatile变量规则:** 对一个volatile的写，happens-before于任意后续对这个volatile变量的读。

* **线程A写一个volatile变量**，实质上是线程A向接下来要获取这个锁的某个线程发出了（线程A对共享变量修改的）消息。
* **线程B读一个volatile变量**，实质上是线程B接收了之前某个线程发出的（对共享变量所做修改的）消息。
* **线程A写一个volatile变量，随后线程B读这个变量**，这个过程实质上是线程A通过主内存向线程B发送消息。

其实仔细看看volatile的实现方式，实际上就是限制了重排序的范围——加入内存屏障\(Memory Barrier or Memory Fence\)。也即是说，允许指令执行的时间先后顺序在一定范围内发生变化，而这个范围就是根据happens-before原则来规定。内存屏障概括起来有两个功能：

* 使写缓冲区的内容刷新到内存，保证对其他线程/CPU可见
* 禁止读写操作的越过内存屏障进行重排序

## reference

* [https://blog.csdn.net/zdxiq000/article/details/60874848](https://blog.csdn.net/zdxiq000/article/details/60874848) 

