### 调度器所解决的问题:

#### 栈管理：

* 既然每个Goroutine都有自己的栈，那么在创建Goroutine时，就要同时创建对应的栈。 Goroutine在执行时，栈空间会不停增长。栈通常是连续增长的，由于每个进程中的各个线程共享虚拟内存空间，当有多个线程时，就需要为每个线程分配不同起始地址的栈。这就需要在分配栈之前先预估每个线程栈的大小。如果线程数量非常多，就很容易栈溢出。

* 为了解决这个问题，就有了Split Stacks技术： 创建栈时，只分配一块比较小的内存，如果进行某次函数调用导致栈空间不足时，就会在其他地方分配一块新的栈空间。新的空间不需要和老的栈空间连续。函数调用的参数会拷贝到新的栈空间中，接下来的函数执行都在新栈空间中进行。

* Golang的栈管理方式与此类似，但是为了更高的效率，使用了连续栈 （Golang连续栈） 实现方式也是先分配一块固定大小的栈，在栈空间不足时，分配一块更大的栈，并把旧的栈全部拷贝到新栈中。 这样避免了Split Stacks方法可能导致的频繁内存分配和释放。

#### 抢占式调度：

* Goroutine的执行是可以被抢占的。如果一个Goroutine一直占用CPU，长时间没有被调度过， 就会被runtime抢占掉，把CPU时间交给其他Goroutine。

#### 调度器的设计:

* work-stealing算法:
  * 每个P维护一个G队列；
  * 当一个G被创建出来，或者变为可执行状态时，就把他放到P的可执行队列中；
  * 当一个G执行结束时，P会从队列中把该G取出；如果此时P的队列为空，即没有其他G可以执行， 就随机选择另外一个P，从其可执行的G队列中偷取一半。

#### 调度器的实现:

* schedule\(\)与findrunnable\(\)函数

  * schedule\(\)函数首先调用runqget\(\)从当前P的队列中取一个可以执行的G。 如果队列为空，继续调用findrunnable\(\)函数。findrunnable\(\)函数会按照以下顺序来取得G：

    * 调用runqget\(\)从当前P的队列中取G（和schedule\(\)中的调用相同）；
    * 调用globrunqget\(\)从全局队列中取可执行的G；
    * 调用netpoll\(\)取异步调用结束的G，该次调用为非阻塞调用，直接返回；
    * 调用runqsteal\(\)从其他P的队列中“偷”。

  * 如果以上四步都没能获取成功，就继续执行一些低优先级的工作：

  * 如果处于垃圾回收标记阶段，就进行垃圾回收的标记工作；
  * 再次调用globrunqget\(\)从全局队列中取可执行的G；
  * 再次调用netpoll\(\)取异步调用结束的G，该次调用为阻塞调用。

  * 如果还没有获得G，就停止当前M的执行，返回findrunnable\(\)函数开头重新执行。 如果findrunnable\(\)正常返回一个G，shedule\(\)函数会调用execute\(\)函数执行该G。 execute\(\)函数会调用gogo\(\)函数（在汇编源文件asm\_XXX.s中定义，XXX代表系统架构），gogo\(\) 函数会从G.sched结构中恢复出G上次被调度器暂停时的寄存器现场（SP、PC等），然后继续执行。

* 如何进行抢占:

  * runtime在程序启动时，会自动创建一个系统线程，运行sysmon\(\)函数（在proc1.go中定义）。 sysmon\(\)函数在整个程序生命周期中一直执行，负责监视各个Goroutine的状态、判断是否要进行垃圾回收等。

  * sysmon\(\)会调用retake\(\)函数，retake\(\)函数会遍历所有的P，如果一个P处于执行状态， 且已经连续执行了较长时间，就会被抢占。retake\(\)调用preemptone\(\)将P的stackguard0设为stackPreempt\(关于stackguard的详细内容，可以参考 Split Stacks\)，这将导致该P中正在执行的G进行下一次函数调用时， 导致栈空间检查失败。进而触发morestack\(\)（汇编代码，位于asm\_XXX.s中）然后进行一连串的函数调用，主要的调用过程如下：

    ```shell
      morestack()（汇编代码）-> newstack() -> gopreempt_m() -> goschedImpl() -> schedule()
    ```

  * 在goschedImpl\(\)函数中，会通过调用dropg\(\)将G与M解除绑定；再调用globrunqput\(\)将G加入全局runnable队列中。最后调用schedule\(\) 来用为当前P设置新的可执行的G。

### 参考:

* [http://morsmachine.dk/go-scheduler](http://morsmachine.dk/go-scheduler)
* [http://www.sizeofvoid.net/goroutine-under-the-hood/](http://www.sizeofvoid.net/goroutine-under-the-hood/)
* [https://studygolang.com/articles/7852](https://studygolang.com/articles/7852)



