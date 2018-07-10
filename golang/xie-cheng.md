### 协程 Corountine

Goroutines exists only in the virtual space of go runtime and not in the OS.

协程对内核透明，也就是系统并不知道有协程的存在，是完全由用户的程序自己调度的，因为是由用户程序自己控制，那么就很难像抢占式调度那样做到强制的CPU控制权切换到其他进程/线程，通常只能进行协作式调度，需要协程自己主动把控制权转让出去后，其它协程才能被执行到。

协程和线程的原理是一样的，当a线程切换到b线程的时候，需要将a线程的相关执行进度压入栈，然后将b线程的执行进度出栈，进入b的执行序列。协程只不过是在应用层实现这一点。

#### golang 协程

A goroutine is lightweight thread of execution.

To invoke a function in a gorountine, use go f\(\). This new goroutine will execute concurrently with the calling one.

协程并不是由操作系统调度的，而且应用程序也没有能力和权限执行cpu调度。怎么解决这个问题？

答案是，协程是基于线程的。内部实现上，维护了一组数据结构和n个线程，真正的执行还是线程，协程执行的代码被扔进一个待执行队列中，有这n个线程从队列中拉出来执行。这就解决了协程的执行问题。

那么协程是怎么切换的呢？

答案是:golang对各种io函数进行了封装，这些封装的函数提供给应用程序使用，而其内部调用了操作系统的异步io函数，当这些异步函数返回busy或bloking时，golang利用这个时机将现有的执行序列压栈，让线程去拉另外一个协程的代码来执行，基本原理就是这样，利用并封装了操作系统的异步函数。包括linux的epoll，select和windows的iocp,event等。

golang的协程是目前各类有协程概念的语言中实现的最完整和成熟的。

#### 协程和线程之间的对比

协程是非抢占式的，这样会导致多任务时间片不能公平分享，所以操作系统废弃了写成改用抢占式的线程来模拟多任务并发。

协程较之于线程内存消耗方面更少。

* gorountine 2KB 线程 8MB
* **A goroutine is created with initial only 2KB of stack size**. Each function in go already has a check if more stack is needed is not and the stack can be copied to another region in memory with twice the original size. This makes goroutine very light on resources.

切换（调度）的开销

* 线程开销涉及模式切换（从用户态到内核态）、PC SP寄存器等的刷新工作。
* gorountine： 只修改 PC/SP/DX 三个寄存器的值。

### 参考

* [http://www.sizeofvoid.net/goroutine-under-the-hood/](http://www.sizeofvoid.net/goroutine-under-the-hood/)
* https://gobyexample.com/goroutines



